# Module: Memory (Temporal, Reflective, Spatial)

## What the paper does

See digest §1.2. Three stores: temporal (chronological nodes `{time, location, observation, key}`,
top-k1=5 retrieval over past Δt=24h by cosine similarity), reflective (end-of-day synthesis, up to 5
cited insights), spatial (4-dim POI belief vectors, Kalman-filtered, decaying toward neutral,
k=10-nearest-neighbor imputation for unvisited POIs). Full equations and constants are in the digest
and in `parameters.md` — this file only covers pilot-specific implementation choices.

## Chicago pilot adaptation

- **Retrieval mechanics unchanged in spirit:** keep top-k1=5 over 24h for temporal memory, k=10-NN
  imputation for spatial memory — these are cheap and scale-independent, no reason to shrink them for
  a smaller pilot.
- **Embedding model:** paper doesn't specify which embedding model it uses for similarity search.
  Pilot default: a small, cheap text-embedding model (e.g. an OpenAI/Anthropic-compatible embeddings
  endpoint, or a local sentence-transformer) — this is an infrastructure choice, not a methodology
  choice, and shouldn't need to match the paper.
- **Storage:** given ~300 agents × ~2 weeks × ~5-minute ticks, temporal memory volume per agent is
  small (thousands of nodes at most, likely far fewer since not every tick generates an observation).
  Brute-force cosine similarity over an agent's own memory is almost certainly fast enough without a
  vector index — see `03-architecture.md`.
- **Reflective synthesis cadence:** keep the paper's end-of-day cadence; it's cheap (one LLM call/agent
  /day) and directly implements the mechanism the ablation study (digest §4) shows matters.

## Open questions

- Whether to persist memory across pilot re-runs (for reproducibility / debugging) or treat each run
  as fresh — leaning toward persisting to SQLite so runs are inspectable after the fact.
