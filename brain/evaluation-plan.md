# Evaluation Plan

Maps each of the paper's validation experiments (digest §4, Section 4/Appendix C of the source paper)
to a Chicago-pilot equivalent, and marks each as **core** (part of the pilot's success criteria per
`01-vision-and-scope.md`) or **stretch** (nice to have, not required to call the pilot successful).

| Paper experiment | Chicago pilot equivalent | Status |
|---|---|---|
| §4.1 Macro time-use vs. Japan national time-use survey, by age group | Simulated time-use-by-category vs. ATUS (national/regional demographic breakdown, no Chicago-specific ATUS exists) | **Core** |
| §4.2 Pairwise human preference (LLM-judge: Naturalness/Coherence/Plausibility) | Same method, unchanged — LLM-judge pairwise comparison of generated daily routines/dialogue, against an ablated or simpler baseline agent built for comparison | **Core** (also doubles as the ablation-study delivery mechanism) |
| §4.3 Travel patterns (avg. travels/hour, weekday vs. weekend) vs. proprietary mobility dataset | Vs. CMAP My Daily Travel Survey mode/time-of-day data, and/or CTA ridership totals as a coarse cross-check | **Core** |
| §4.4 POI popularity prediction (Spearman correlation) vs. Google Maps ratings | Vs. Google Places / Yelp ratings + review-count for POIs in the study area(s) | **Stretch** — doable, but lower priority than time-use/travel-pattern checks for a first pilot |
| §4.5 Well-being prediction (5-class, macro-F1 vs. GBDT) vs. proprietary survey | No adequate public individual-level equivalent found — see `data/chicago-data-sources.md` | **Skip** unless a suitable public survey substitute turns up |
| §4.6 Crowd density heatmaps vs. smartphone location data | Vs. Divvy trip data as a directional proxy (bike-only, weaker ground truth, explicitly caveated) | **Stretch** |
| Appendix C.1 Scalability (10³–10⁶ agents) | Not attempted — pilot is single-scale (~300 agents) by design | **Out of scope** |
| Appendix C.2 Human-likeness Likert scoring across Activity/Dialogue/Mobility/Event Reaction | Same method at pilot scale — GPT-tier judge scores a sample of generated outputs per domain | **Core** |
| Appendix C.3 Needs-evolution archetype case studies | Directly replicated — a handful of named Chicago archetypes (office worker, CPS student, shift-work nurse/CTA employee, freelancer, retiree), inspect need trajectories over a day | **Core** — cheapest, highest-signal integration test; do this first, before full-population runs |
| Appendix C.4 Belief-estimation MAE across LLM backbones | Only relevant if comparing multiple agent-cognition LLMs — worth running once if cost allows, to justify the pilot's chosen model | **Stretch** |
| Appendix C.5 Ablation study (remove belief / recursive planning / long-term goal / needs / persona) | Directly replicated — same ablation set, scored via the same LLM-judge Likert method as Appendix C.2 | **Core** — this is the pilot's primary evidence that "the mechanism does something," per `01-vision-and-scope.md`'s success criteria |

## Suggested execution order

1. **Archetype needs-trajectory sanity check** (C.3 equivalent) — smallest possible test, no full
   population needed, catches basic needs-module bugs cheaply before scaling up.
2. **Full-population short run** (core time-use + travel-pattern checks) — the population-scale
   integration test.
3. **Ablation study** (C.5 equivalent, scored via C.2's LLM-judge method) — the pilot's headline
   result per the success criteria in `01-vision-and-scope.md`.
4. Stretch items (POI popularity, crowd density proxy, belief-estimation-by-backbone) only after 1–3
   are working and if budget/time remain.

## Honesty notes for any pilot writeup

- ATUS is national/regional, not Chicago-specific — say so, don't imply a tighter match than exists.
- Divvy-based crowd-density comparison is bike-mode-only and not a true population crowd measure — a
  much weaker proxy than the paper's smartphone panel; present it as directional only.
- Skipping the well-being study isn't a methodology gap in ChicagoSim — it's a data-availability gap.
  State it as such rather than silently omitting it.
