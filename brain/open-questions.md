# Open Questions

Unresolved decisions. When one gets answered, move the resolution into `decisions-log.md` (with
today's date) and delete it from this list — don't let this file accumulate stale entries.

## Scope-defining (blocks meaningful progress until answered)

- **Which Chicago community area(s) are the pilot's study area?** Needs a real commute relationship
  (residential ↔ job-dense pairing suggested in `data/chicago-data-sources.md`) and reasonably good
  ACS PUMS / LODES data coverage. This choice cascades into almost everything else (POI pull, PUMA
  crosswalk, LODES filter, network graph extent) — should be resolved first.
- **PUMA ↔ Community Area crosswalk** for whichever area(s) get chosen — PUMAs are coarser than
  community areas, so persona sampling needs a defined filter/weighting approach once the area is set.
- **Agent-cognition LLM and judge LLM choice** — paper uses GPT-4o-mini (agents) / GPT-4o (judge).
  Pilot needs its own pick, favoring cost given "lightweight." Affects `parameters.md`,
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
- **Perception/dispatcher pre-filter:** is a cheap rule-based "no action needed" short-circuit before
  the LLM routing call faithful to the paper's intent, or does it drift too far from the mechanism
  being tested? See `modules/06-perception-dispatcher.md`.

## Infrastructure (not methodology, but needs an answer before implementation)

- **Is AgentSociety's code available to build on, or fully build our own loop?** The paper runs on top
  of AgentSociety (Piao et al., 2025); worth checking whether that codebase is public and adoptable
  before committing to `03-architecture.md`'s "build our own simple loop" default.
- **Embedding model for memory similarity search** — any cheap/available option works; not yet chosen.
- **Weather/temperature data pipeline** (NOAA API vs. pre-pulled table) for vehicle-selection context.
- **Google Places / Yelp API quota needs** at the pilot's expected POI volume — check before treating
  either as a hard dependency (`data/chicago-data-sources.md`).
