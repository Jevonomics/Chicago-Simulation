# Module: Beliefs

## What the paper does

See digest §1.3 (and §1.2 spatial memory for the storage/update math). Every POI visit triggers an
LLM-generated subjective observation (persona + activity + emotional state + POI description as
context), which is merged into the prior 4-dim belief via the Kalman update (`σ_b^(0)=0.25`,
`σ_o=0.2`, decay `λ=0.03`). This is the module the ablation study shows matters most for Activity and
Event Reaction human-likeness (digest §4, Appendix C.5).

## Chicago pilot adaptation

- **Keep the mechanics as-is** — the Kalman filter and decay are cheap, dimension-agnostic math; no
  reason to simplify for a smaller pilot. Copy the constants verbatim (`parameters.md`).
- **POI description context:** the paper's "description of the POI" input needs a real source. Pilot
  plan: pull whatever's available per-POI from OSM tags + optionally a Google Places/Yelp lookup
  (category, price level if listed, rating if listed) — see `data/chicago-data-sources.md`. Where no
  rich description exists (common for OSM-only POIs), fall back to category + name only; the LLM
  observation will be less grounded for those, which is an accepted pilot limitation worth noting in
  eval writeups (echoes the paper's own admission of hallucination risk for "recent or less-known POIs").
- **Belief priors for chains/franchises:** the paper notes CitySim shows a positive popularity bias
  toward well-known/branded POIs (digest §4.4, §4.6). Worth deliberately watching for the same effect
  in Chicago output (e.g. agents over-visiting a well-known chain vs. a real but obscure local spot) —
  this is a good qualitative check to include in `evaluation-plan.md`, not something to try to "fix"
  preemptively.

## Open questions

- None currently beyond the shared POI-data-quality question tracked in `data/chicago-data-sources.md`.
