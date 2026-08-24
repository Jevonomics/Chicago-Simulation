# Architecture (Proposed)

Status: **proposed defaults, not locked** — nothing below has been implemented yet. Treat this as the
straw-man to react to, not a commitment. Revise freely; log any change in `decisions-log.md`.

## Why not just run the original stack

The paper runs CitySim as a set of modules on top of **AgentSociety**, a large-scale multi-agent
simulation platform (Piao et al., 2025) — itself a substantial research system built for distributed,
1M-agent-scale simulation.

**Decision: build ChicagoSim's own simplified loop; do not adopt the AgentSociety platform.** At the
pilot's scale (low hundreds of agents, single machine, 1–2 simulated weeks), AgentSociety's
distributed-platform machinery is more infrastructure than the pilot needs, and adopting it would be a
substantial engineering detour before any of the methodology itself gets tested. This is a resolved
default for the pilot phase, not a permanent stance — revisit only if the pilot succeeds and a
scale-up is deliberately chosen (see `roadmap.md` Phase 6). Whether AgentSociety's code is even
available to reference at that point is a question worth asking then, not now.

## Proposed system layers

1. **World layer** — a graph-structured representation of the chosen Chicago area(s): street network
   + POIs + transit stops, built from OpenStreetMap + CTA GTFS (see `data/chicago-data-sources.md`).
   Proposed tooling: `osmnx` for the street graph, `geopandas`/`shapely` for spatial joins, a flat POI
   table (id, name, category, lat/lon, address, source price/rating if available).
2. **Population layer** — synthetic persona generation from ACS PUMS microdata + LODES commute flows
   (see `data/persona-initialization.md`), producing one persona record per agent, each anchored to a
   home location and (if employed/school-age) a work/school location within the study area.
3. **Cognitive layer** — one process/object per agent implementing the modules in `modules/`: persona
   (static), memory (temporal/reflective/spatial stores), beliefs, needs, long-term goals, perception/
   dispatcher, planning, place selection, vehicle selection, social. This is the direct analogue of
   CitySim's Section 3.
4. **Simulation loop** — the driver from the paper's Algorithm 1 (`02-citysim-methodology-digest.md`
   §"Simulation loop"), at 5-minute ticks, run for the pilot's target duration (default: 1–2 simulated
   weeks — see `01-vision-and-scope.md`). **Implements the resolved interpretation documented in that
   section's "Resolved ambiguity" note**: `perceive()` at every tick is a cheap, non-LLM check (state
   comparisons, threshold checks, co-location checks — no model call), and the LLM dispatcher only
   fires when that check flags a reaction is warranted; `reflect()` runs once per agent per day, not
   per tick. A builder implementing this layer from this document alone should not reintroduce an LLM
   call inside the raw per-tick loop by default — see `cost-and-budget.md` for the expected
   calls/agent-day this produces and the threshold that signals the gate isn't working as intended.
5. **LLM layer** — a thin client wrapping whichever model(s) are chosen for (a) agent cognition
   (cheap/fast) and (b) judge evaluation (stronger), with prompt templates per module call, and
   response caching where safe (e.g. deterministic parts of persona init) to control cost.
6. **Persistence layer** — memory stores need to survive across ticks/days and support similarity
   retrieval (temporal memory's top-k1 cosine search, spatial memory's k-nearest-POI imputation).
   Proposed: SQLite (or DuckDB) for structured state + a small embedding index (e.g. `sqlite-vec`,
   or just brute-force cosine over a few hundred agents' worth of memory — at this scale a vector DB
   is probably unnecessary complexity).
7. **Evaluation layer** — scripts that consume simulation output and produce the comparisons defined
   in `evaluation-plan.md` (time-use charts, travel-volume charts, ablation runs, LLM-judge scoring).

## Language/stack proposal

Python end-to-end: the geospatial tooling (`osmnx`, `geopandas`), the ABM space (Mesa exists as a
prior-art reference), and LLM SDKs are all strongest there, and the paper's own ecosystem
(AgentSociety, MobileCity) is Python. No strong reason to deviate.

## Cost & call-volume budget

Under the resolved simulation-loop interpretation above, expected LLM call volume is roughly
**40–60 calls/agent/simulated-day** — see `cost-and-budget.md` for the itemized worked estimate, the
target-model budget ceiling, and the calls/agent-day × agent-count × days check that should be run
*before* committing to a population/duration combination, not after. If actual implementation measures
meaningfully above **~100 calls/agent-day**, that's a signal the perception gate isn't suppressing
per-tick calls as intended and should be investigated before scaling up population or duration.

## What's intentionally deferred

- Distributed/multi-process execution — a few hundred agents at 5-minute ticks over ~2 weeks is small
  enough to run single-machine, likely single-process with async LLM calls for concurrency.
- A general plugin/module framework — build the specific modules the methodology needs; don't build a
  generic "module system" the paper doesn't ask for.
- A frontend/visualization app — evaluation outputs (charts, heatmaps) are enough for a pilot; a live
  dashboard is a stretch goal at best.

See `roadmap.md` for the phased build order this implies.
