# Open Questions

Unresolved decisions. When one gets answered, move the resolution into `decisions-log.md` (with
today's date) and delete it from this list — don't let this file accumulate stale entries.

## Scope-defining (blocks meaningful progress until answered)

- **Agent-cognition LLM and judge LLM choice** — paper uses GPT-4o-mini (agents) / GPT-4o (judge).
  `cost-estimate.md` recommends an OSS tiered strategy (cheap 8B-class model for high-frequency
  low-stakes calls, 70B-class model for belief/planning/place-selection/reflection calls) over a
  single-model choice like Grok — ~3.4x cheaper than an all-Grok-4.3 approach while protecting the
  calls the paper's own ablation/backbone studies show are quality-sensitive. Not yet locked in —
  still need to pick specific models/hosts (candidates: Llama 3.1 8B via DeepInfra for cheap tier,
  Llama 3.3 70B via Groq/Fireworks for strong tier) and a judge LLM. Affects `parameters.md`,
  `03-architecture.md`.

## Methodology adaptation choices (can be decided as each module is built)

- **Long-term goal revision cadence:** keep the paper's monthly cadence (goal *formation* observed,
  revision mostly not, within a 1–2 week pilot) or shorten to weekly purely to observe revision
  behavior within pilot duration? See `modules/05-long-term-goals.md`.
- **Rideshare as a distinct vehicle mode?** Very salient in Chicago, absent from the paper's vehicle
  set. Current lean: fold into "car" for pilot simplicity. See `modules/09-vehicle-selection.md`.
- **Distance metric for the gravity model (`D_ij`):** network distance (via OSM street graph) vs.
  straight-line. Current lean: network distance. See `modules/08-place-selection.md`.
- **Social network seeding rule:** how many initial contacts per agent, and how to weight
  household/coworker/neighbor ties in initial belief values. See `modules/10-social.md`.
- **Need decay rates (α_n):** unset; needs a first tuning pass against the archetype-trajectory sanity
  check (`evaluation-plan.md` step 1). See `modules/04-needs.md`, `parameters.md`.
- **Habit taxonomy source:** hand-authored vocabulary (current lean) vs. mined from a public lifestyle
  survey. See `modules/01-persona.md`.

## Infrastructure (not methodology, but needs an answer before implementation)

- **Pilot-phase LLM budget ceiling** — `cost-and-budget.md` uses **$100 of a $2,500 total project
  budget** as a placeholder to work the calls/agent-day × agent-count × days check. Needs confirming
  (or replacing) as a real number before that check's conclusions can be treated as final.
- **Agent-cognition LLM choice, sharpened by the budget check:** `cost-and-budget.md` shows that on
  Grok's mini/economical tier specifically, a $100 ceiling only comfortably covers the low end of the
  pilot's population/duration range (~200 agents × 7 days) — the fuller-scope lean in `parameters.md`
  (350–500 agents × 14 days) requires the cheaper OSS tiered-routing approach from `cost-estimate.md`
  to fit the same ceiling. This needs to be decided deliberately (model choice ↔ scope are coupled),
  not discovered mid-run.
- **Embedding model for memory similarity search** — any cheap/available option works; not yet chosen.
- **Weather/temperature data pipeline** (NOAA API vs. pre-pulled table) for vehicle-selection context.
- **Google Places / Yelp API quota needs** at the pilot's expected POI volume — check before treating
  either as a hard dependency (`data/chicago-data-sources.md`).
