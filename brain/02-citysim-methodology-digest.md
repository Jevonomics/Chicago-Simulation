# CitySim Methodology — Granular Digest

Source: Bougie & Watanabe, "CitySim: Modeling Urban Behaviors and City Dynamics with Large-Scale
LLM-Driven Agent Simulation," arXiv:2506.21805v1, 26 Jun 2025 (Woven by Toyota). This is a complete
technical distillation — every module, equation, and numeric parameter used in the paper — so this
project never needs to re-open the PDF to recall how the original works. Chicago-specific adaptation
decisions live in `modules/*.md`, not here; this file is a faithful record of the source.

## Simulation loop (Algorithm 1, Appendix B.1)

```
For each day:
  For each agent:
    plan_day()
    For each time step (5-minute ticks):
      perceive()
      action ← decide_action()
      if action.requires_move:
        poi ← select_POI()
        vehicle ← select_vehicle(poi)
        move(poi, vehicle)
      else:
        execute(action)
      reflect()   # beliefs, goals, needs, habits
```

Timestep = 5 minutes, matching time-use-survey reporting resolution. Random seeds fixed for
reproducibility. Framework runs on top of the AgentSociety platform (Piao et al., 2025).

## 1. Cognitive state representation

### 1.1 Persona module

Questionnaire-derived, fixed at agent creation:

- **Demographic attributes:** name, age, gender, occupation, income, hobbies, education, household
  composition, life stage. These modulate activity space (children attend school; retirees prefer
  daytime leisure) and shape daily-routine patterns.
- **Spatial anchors:** home location, work/school location.
- **Psychographic traits:** Big Five (Openness, Conscientiousness, Extraversion, Agreeableness,
  Neuroticism), each on a discretized **1–3 scale** (1=low, 2=medium, 3=high).
- **Habits and preferences:** empirically-derived activity preferences, habits (e.g. "early riser"),
  leisure patterns.

Paper's population: personas sampled from a **proprietary Japan survey**, cross-checked against
Japanese census/lifestyle-survey distributions. Home/work assigned by Japanese population density
via OpenStreetMap data, ensuring realistic spatial distribution and feasible commutes.

### 1.2 Memory module

Three components, motivated by the factual/emotional split in human memory (LaBar & Cabeza, 2006):

**Temporal memory** — chronological stream of memory *nodes*. Each node = `{time, location,
observation, key}`. `key` filters observations at retrieval time. New entries appended on every
simulation step, action, or reflection. Retrieval: top **k₁ = 5** entries from the past **Δt = 24
hours**, ranked by cosine embedding similarity.

**Reflective memory** — the agent's thoughts/attitudes about events in temporal memory. Each entry
links to one or more temporal nodes. Synthesized at end of each day: recent temporal entries are
compiled and given to the LLM, which is prompted to generate salient high-level questions answerable
from that memory (aimed at uncovering habits, preferences, evolving beliefs). For each question,
relevant entries are retrieved by semantic + temporal proximity, and the LLM extracts **up to 5
insights**, each cited with explicit evidence references, e.g.:

> "The agent prefers evening study sessions (evidence: 8, 13, 22, 45)."

Both raw observations and prior reflections are eligible as evidence — reflections can recursively
build on reflections. All insights are stored back into reflective memory.

**Spatial memory** — beliefs `b_i ∈ R^4` about POIs, one scalar per dimension in
`{price, atmosphere, satisfaction, convenience}`.
- Initialized to **0.5** (neutral) for any unvisited POI.
- If a *similar* POI has been visited, initialize instead via embedding-based similarity:
  `b_i ← E_{j∈N(i)}[b_j]`, where `N(i)` = similar POIs retrieved from spatial memory (nearest-neighbor
  imputation over **k = 10** most similar visited locations by embedding distance).
- **Update rule (Kalman filter,** Appendix eq. 2**):** on visiting POI `i` at time `t` with observed
  outcome `o_i^(t)`:
  ```
  K^(t)     = σ_b^(t-1) / (σ_b^(t-1) + σ_o)
  b_i^(t)   = K^(t) · o_i^(t) + (1 − K^(t)) · b_i^(t-1)
  σ_b^(t)   = (1 − K^(t)) · σ_b^(t-1)
  ```
  with `σ_b^(0) = 0.25` (initial belief uncertainty), `σ_o = 0.2` (observation noise). Uncertainty
  `σ_b` is retrieved alongside `b_i` and used to guide POI selection (more uncertain beliefs presumably
  invite exploration, though the paper doesn't specify an explicit exploration bonus formula beyond
  this).
- **Decay:** after each simulated day, each dimension `d` decays toward neutral:
  `b_{i,d} ← (1−λ)·b_{i,d} + λ·b_{0,d}`, with `b_0 = 0.5` (neutral) and **λ = 0.03**.

### 1.3 Belief module

Triggered every time an agent visits a place. The LLM is prompted with visit-specific context
(persona, current activity, emotional state, POI description) to generate a subjective, multi-dimensional
observation `o_i^(t)` with reasoning. That observation is merged into the prior belief `b_i^(t-1)` via
the Kalman update above. This is the module that turns raw visits into durable, reasoned preference.

### 1.4 Needs module

Four tracked needs: **hunger, energy, safety, social connection**, each `∈ [0,1]`.

- **Initialization:** at the start of each day, an LLM call conditioned on demographic + temporal
  context sets `s₀ = {hunger, energy, safety, social} ∈ [0,1]^4`.
- **Decay:** continuous over the day: `s_n(t) = max(0, s_n(t−Δt) − α_n·Δt)` (decay rate `α_n` per need,
  values not given numerically in the paper beyond the mechanism).
- **Updates:** after each activity or significant event, the LLM evaluates the outcome and updates
  scores given agent experience, context, and current state.
- **Priority / interruption:** needs are prioritized **hunger > safety > energy > social** (confirmed
  in both main text and appendix, though the main text's prose lists "hungry > safe > tired > social" —
  same order, "tired" = energy). `dominant_need = argmin_n {s_n ≤ T_n}` (the least-satisfied need that
  has crossed its threshold). A higher-priority need crossing threshold can interrupt an ongoing plan
  and force reprioritization. The dominant need is written into the persona module as plain text so
  downstream LLM calls can condition on it directly.
- **Thresholds (Appendix A):** `T_hunger = 0.3`, `T_energy = 0.3`, `T_safety = 0.2`, `T_social = 0.2`.

### 1.5 Long-term goal module

Models formation/revision of high-level aspirations, grounded in Maslow's Hierarchy of Needs (Huitt,
2007). Goals are revisited **monthly**, or immediately after a major life event (e.g. employment
change). The LLM is queried with: persona, financial status, social contacts, recent activities,
current goals, plus three computed signals:

- **Need fulfillment index:** proportion of the day during which needs *exceed* their thresholds
  (i.e. are satisfied).
- **Financial stress flag:** `income < 0.9 × expenses`.
- **Social isolation flag:** fewer than 3 unique social contacts in the last 7 days.
- **Interest signal:** proportion of recently-visited POIs (last 30 days, set `V`) whose satisfaction
  belief exceeds 0.5:
  `interest = (1/|V|) · Σ_{i∈V} 𝟙[b_i^sat > 0.5]`.

Given this composed context `c`, a structured prompt conditions the LLM to generate coherent
short-term (few-weeks) and long-term goals: `g_t^1, …, g_t^M ~ LLM(p_goal | θ, c)`. These goals feed
back into the planning module as inputs for value-driven planning.

### 1.6 Perception module

At each timestep, receives an environment observation (location, presence of friends, etc.) and
decides whether the agent should react at all. If so, it enumerates the available modules (each with
a functional description) and asks the LLM to pick the most appropriate one for the situation. A
**dispatcher** then invokes that module (planning, social interaction, etc.) using the agent's
inferred needs, a short explanation, and required parameters. This is effectively the agent's
top-level control loop / router.

## 2. Mobility behaviors

### 2.1 Planning module — recursive, value-driven daily scheduling

Two-step recursive decomposition into time **blocks** (min granularity = 5 minutes, matching the
timestep), each block carrying a start time, duration, and activity/intention:

1. **Mandatory block assignment.** Starting from an empty day, assign fixed, non-negotiable
   activities (sleep, work, medical appointments) from persona + occupation + needs. If an activity
   doesn't fill its interval, the block subdivides.
2. **Medium-priority recursive filling.** Remaining `[EMPTY]` blocks are recursively filled with
   medium-priority tasks: meals, hygiene, essential errands.

Some blocks still remain unfilled. Per Maslow's hierarchy, these are reserved for **leisure or
long-term-goal activities** (hobbies, socializing, exploration) and are filled **at execution time**
(not pre-planned) via **value-driven planning**:

- For each empty block, generate **N = 3** candidate activities (max duration = the block length)
  that might improve enjoyment/satisfaction or fulfill a need/goal.
- "Evaluate" each candidate by having the LLM imagine the resulting desire/need state after taking
  that action.
- Select the candidate expected to best fulfill the agent's intrinsic desires.
- Implemented as a **single structured LLM call with internal (chain-of-thought-style) reasoning
  steps** — not N separate LLM calls.

### 2.2 Place selection module — belief-aware gravity model

Extends AgentSociety's gravity model with belief-weighting. Home/work activities use the persona's
fixed addresses directly (no selection needed).

**Step 1 — Macro-level area selection.** LLM chooses whether to stay local or travel farther, given:
intention, schedule, emotional state, area visit history, and popular nearby areas (ranked by distance
and popularity). Considers the **top 10 nearby areas**.

**Step 2 — Micro-level POI selection**, within the chosen area:
- *Intention extraction:* determine required POI type(s) (café, park, …) and adjust feasible search
  range using internal factors (age, schedule) and environmental factors (weather, traffic), producing
  a candidate set — up to **200 candidate POIs**, ranked by relevance and proximity.
- *Belief-weighted gravity model* (Eq. 1): for candidate POI `i`,
  ```
  p_ij = [(b_j + ε) / D_ij^(1 + γ(b_j − 0.5))] / Σ_k [(b_k + ε) / D_ik^(1 + γ(b_k − 0.5))]
  ```
  where `b_j` = belief-based attractiveness of location `j` (the **mean of the agent's 4-dimensional
  belief vector** for that POI, per Appendix A), `D_ij` = distance, `γ` = distance-decay strength,
  `ε` = small constant for numerical stability. Effectively: higher belief flattens the distance-decay
  exponent (agent will travel farther for a place it believes in); if no belief exists for `j`, it's
  estimated from similar POIs in spatial memory (same imputation as §1.2).
  **Parameters:** `γ = 2.0`, `ε = 10⁻³`.

### 2.3 Vehicle selection module

Given trip distance `d`, time of day `t`, month `m`, weather `w`, temperature `T`, persona `p`, and
available vehicle set `V`, a structured prompt queries the LLM:
`LLM(p_v | d, t, m, w, T, p, θ, V)` → selected vehicle `v*` **plus a brief justification**, which is
stored in reflective memory. This approximates
`v* = argmax_{v∈V} U(d,t,m,w,T,p,θ,V)` where `U(·)` is an implicit utility function realized purely
through the LLM's reasoning (no explicit utility formula is specified/learned).

**Available vehicles:** walk, bicycle, car, bus, train.

## 3. Social behaviors

Foundation: a weighted social network where every edge carries the *believing* agent's evolving
social beliefs about the other party. Agent `u` maintains `b_{u,v} ∈ R^3` per contact `v`, over
`{affinity, trust, familiarity}`. Beliefs initialize at simulation start from demographic similarity
and stated relationships, then update continuously from interaction outcomes (positive/neutral/negative
sentiment extracted post-interaction, each dimension nudged accordingly — no explicit formula given,
analogous in spirit to the Kalman-style belief update used for places).

**Face-to-face interactions:** occur when agents are co-located. Partner `v` is chosen with probability
proportional to current belief score: `p_v = b_{u,v} / Σ_{v'∈V} b_{u,v'}`, where `V` = eligible
co-located agents. **Limited to one partner per 30-minute tick** to prevent excessive socializing.
Message generation reflects personality traits, past interactions, and persona-derived topic
constraints (single LLM call).

**Online interactions:** simulate remote contact (phone/messaging). Triggered when an agent's social
satisfaction score drops below threshold; during leisure time it proactively evaluates whom to
contact and whether to go face-to-face or online, in a single LLM call that returns both the modality
and the target agent in structured form.

## 4. Experimental setup (Section 4 + Appendix A)

- Platform: AgentSociety (Piao et al., 2025). Agent LLM: **GPT-4o-mini** (unless noted). Judge LLM
  for pairwise/Likert evaluations: **GPT-4o**. Population: **1,000 agents**, Tokyo metropolitan area.
  Big Five discretized 1–3. Home/work assigned via Japanese population density from OSM.
- Baselines compared against: GeAn (Generative Agents, Park et al. 2023a/b), AGA — "Affordable
  Generative Agents" (Yu et al. 2024), HumanoidAgent (Wang et al. 2023), MobileCity (Ye et al. 2025),
  AgentSociety (Piao et al. 2025).

### Results summary (for reference, not targets we need to hit)

- **4.1 Macro time-use:** two months of simulated daily activity, mapped to Japan's 2021 national
  time-use-survey categories, aggregated by age group. Close match to ground truth (Figure 1).
- **4.2 Pairwise human preference:** 15 trials/method; outputs normalized via Llama-3.1-70B to remove
  stylistic bias; GPT-4o judges pairs on **Naturalness, Coherence, Plausibility**. CitySim wins most
  often (Figure 2 win-rate matrix, e.g. CitySim beats GeAn 0.85, AgentSociety 0.60).
- **4.3 Travel patterns:** avg. agent travels/hour, weekday vs weekend, vs a proprietary city-scale
  dataset. CitySim best matches commute-peak timing/amplitude and weekend leisure shape (Figure 3).
- **4.4 POI popularity prediction:** Shibuya, Tokyo. Ground truth = Google Maps ratings; simulated =
  visit counts over a simulated month. Compared via Spearman correlation against SocietyAgent, across
  categories Food/Entertainment/Shopping/Transportation/Services (Figure 4). CitySim shows a **positive
  bias toward well-known/branded POIs** (documented limitation).
- **4.5 Well-being prediction:** proprietary 1,200-response Japan well-being survey, 5-class target.
  Agents seeded with matching personas, simulate 3 weeks, answer the same survey via memory. Macro-F1:
  GeAn 0.19, AGA 0.20, HumanoidAgent 0.22, MobileCity 0.21, AgentSociety 0.28, **CitySim 0.36**, GBDT
  baseline (trained directly on real activity/location data) 0.45 — best but not competitive with a
  supervised baseline that sees real behavioral data.
- **4.6 Crowd density:** Shibuya heatmaps from aggregated agent visit counts vs. smartphone-location
  ground truth. Matches central/commercial-street densities well; **underestimates small-street
  crowding**, attributed to the belief-gravity model's bias toward prominent POIs.
- **Appendix C.1 Scalability:** mean time/simulation-step at 10³/10⁴/10⁵/10⁶ agents (1:999
  set:fetch query ratio) — grows sub-linearly-ish, e.g. 9.0ms → 0.183s from 10³ to 10⁶ agents; broadly
  comparable to AgentSociety's own numbers.
- **Appendix C.2 Human-likeness (Likert 1–5, GPT-4o judge)** across Activity/Dialogue/Mobility/Event
  Reaction, 20,000 outputs/baseline. CitySim leads on all four (e.g. Activity 4.37 vs AgentSociety's
  4.02, GeAn's 3.11).
- **Appendix C.3 Needs evolution case studies:** 5 archetype agents (office worker, HS student,
  night-shift nurse, freelance designer, retired senior) show distinct, plausible need trajectories
  across a day (Figure 7) — a strong, cheap qualitative sanity check worth replicating.
- **Appendix C.4 Belief estimation accuracy:** MAE of predicted vs. ground-truth belief for held-out
  POIs, across 5 categories (Restaurants, Parks, Shops, Transport, Entertainment) and 10 LLM backbones.
  Larger models (GPT-4o) generalize beliefs best; smaller LLaMA-2 models worst, especially Entertainment.
- **Appendix C.5 Ablation study (Table 4):** removing each module drops human-likeness. Persona removal
  hurts broadest (all 4 metrics drop sharply — agents become homogenized). Needs removal hurts Activity
  and Event Reaction most (can't interrupt/reprioritize). Recursive planning removal hurts Activity and
  Mobility most (schedules become rigid). Belief removal hurts Activity and Event Reaction. Long-term
  goal removal has the most localized effect (Dialogue, Mobility) — consistency benefits accrue over
  multiple days, so a short ablation window under-states its value.

## 5. Stated limitations (Section 6 / Appendix B, as written by the authors)

- Reproducibility limited — some experiment data is proprietary/non-public.
- Inherits LLM cultural/gender/socioeconomic biases; occasional hallucination in POI appraisal,
  especially for recent/less-known places.
- Quality is bounded by the underlying LLM's reasoning strengths/weaknesses.
- Many interacting modules make it hard to isolate individual effects (hence the ablation study, which
  the authors say only "partially" addresses this).
- Circular evaluation risk: GPT-4o judges GPT-4o-mini-driven agents — same-family LLM-as-judge bias
  toward its own stylistic outputs is explicitly acknowledged as a limitation, not fully mitigated.
- Underestimates small-street crowd density due to belief-gravity's popularity bias.
- Self-esteem / self-actualization needs are not modeled — only the four lower-tier needs are tracked.
- Doesn't model weather/crowding/transport-delay effects on real-time appraisal (only on vehicle
  choice's input features).
- Persona fidelity depends on having rich survey data — degrades for cold-start / under-observed
  populations, an explicitly named equity concern (Section 7, Ethics Statement) directly relevant to
  choosing which Chicago community area(s) to model, and how to source personas fairly. See
  `data/persona-initialization.md`.
