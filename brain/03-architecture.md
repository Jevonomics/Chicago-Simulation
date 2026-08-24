# Architecture (Proposed)

Status: **proposed defaults, not locked** — nothing below has been implemented yet. Treat this as the
straw-man to react to, not a commitment. Revise freely; log any change in `decisions-log.md`.

## Why not just run the original stack

The paper runs CitySim as a set of modules on top of **AgentSociety**, a large-scale multi-agent
simulation platform (Piao et al., 2025) — itself a substantial research system built for distributed,
1M-agent-scale simulation. For a several-hundred-agent pilot, adopting that whole platform is
almost certainly more infrastructure than the pilot needs. Default plan: implement the modules
directly against a much simpler loop, and only reach for something like AgentSociety/Mesa if the
pilot proves out and needs to scale. Open question: whether AgentSociety's code is actually available
to borrow from — see `open-questions.md`.

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
   weeks — see `01-vision-and-scope.md`).
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

## What's intentionally deferred

- Distributed/multi-process execution — a few hundred agents at 5-minute ticks over ~2 weeks is small
  enough to run single-machine, likely single-process with async LLM calls for concurrency.
- A general plugin/module framework — build the specific modules the methodology needs; don't build a
  generic "module system" the paper doesn't ask for.
- A frontend/visualization app — evaluation outputs (charts, heatmaps) are enough for a pilot; a live
  dashboard is a stretch goal at best.

See `roadmap.md` for the phased build order this implies.
