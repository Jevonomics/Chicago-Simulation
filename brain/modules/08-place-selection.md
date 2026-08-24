# Module: Place Selection (Belief-Weighted Gravity Model)

## What the paper does

See digest §2.2. Two steps. **Macro:** LLM picks stay-local vs. travel-farther given intention,
schedule, emotional state, area visit history, and the top-10 nearby areas ranked by distance and
popularity. **Micro:** intention extraction narrows to POI type(s) and a feasible range (factoring
age/schedule/weather/traffic), producing up to 200 candidate POIs; final choice uses the belief-
weighted gravity model:

```
p_ij = [(b_j+ε)/D_ij^(1+γ(b_j−0.5))] / Σ_k [(b_k+ε)/D_ik^(1+γ(b_k−0.5))]
```

`γ=2.0`, `ε=10⁻³`, `b_j` = mean of the agent's 4-dim belief vector for POI `j`. Home/work use fixed
persona addresses, no selection.

## Chicago pilot adaptation

- **Formula copied verbatim, including γ and ε** — this is a pure spatial-choice model, not tied to
  Japan-specific data; no reason to retune for a pilot.
- **"Areas" definition for Chicago:** the paper's "area" concept (macro-level, ranked by distance and
  popularity) maps naturally onto Chicago's **77 official Community Areas** (a real, well-known,
  city-published unit — see `data/chicago-data-sources.md`) rather than an arbitrary grid, since the
  study area is itself defined in terms of community areas (`01-vision-and-scope.md`). "Popularity"
  per area for the macro step can be approximated from POI density/rating aggregates within each area.
- **Candidate POI cap:** the paper's "up to 200 candidates" is a global-city-scale number; for a
  1–3-community-area pilot the actual candidate pool per activity type may be well under 200 already —
  keep 200 as an upper cap, not a target, and don't artificially inflate the candidate set to hit it.
- **Distance `D_ij`:** use real network distance (via the OSM street graph, `03-architecture.md`)
  rather than straight-line distance where feasible — the paper doesn't specify which it uses, and
  network distance is more realistic for a walkable-urban-neighborhood pilot where street grid detours
  matter (e.g. river/highway barriers common in Chicago community areas).

## Open questions

- Straight-line vs. network distance for `D_ij` — leaning network distance, not yet finalized; see
  `open-questions.md`.
- Exact popularity proxy for macro-area ranking — candidates: POI count, aggregate Google/Yelp rating
  volume, or Divvy station activity as a footfall proxy.
