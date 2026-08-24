# Parameters — Single Source of Truth

Every numeric knob, split into **paper values** (copy verbatim — no principled reason to change) and
**pilot-specific values** (not specified by the paper, need pilot-side decisions). When a value
changes, edit it here and note why in `decisions-log.md`.

## Copied verbatim from the paper

| Parameter | Value | Module | Source |
|---|---|---|---|
| Simulation timestep | 5 minutes | Simulation loop | digest §"Simulation loop" |
| Big Five scale | 1–3 (discretized) | Persona | digest §1.1 |
| Temporal memory retrieval window (Δt) | 24 hours | Memory | digest §1.2 |
| Temporal memory top-k (k₁) | 5 | Memory | digest §1.2 |
| Reflective memory insights/day | up to 5, cited | Memory | digest §1.2 |
| Spatial belief dimensions | 4: price, atmosphere, satisfaction, convenience | Memory / Beliefs | digest §1.2 |
| Spatial belief init (unvisited, no similar POI) | 0.5 (neutral) | Memory / Beliefs | digest §1.2 |
| Spatial belief imputation k-NN (k) | 10 | Memory / Beliefs | digest §1.2 |
| Belief decay rate (λ) | 0.03 | Memory / Beliefs | digest §1.2, Appendix A |
| Kalman initial belief uncertainty (σ_b⁽⁰⁾) | 0.25 | Beliefs | Appendix A eq. 2 |
| Kalman observation noise (σ_o) | 0.2 | Beliefs | Appendix A eq. 2 |
| Needs tracked | hunger, energy, safety, social | Needs | digest §1.4 |
| Need priority order | hunger > safety > energy > social | Needs | digest §1.4, Appendix A |
| Need threshold — hunger (T_hunger) | 0.3 | Needs | Appendix A |
| Need threshold — energy (T_energy) | 0.3 | Needs | Appendix A |
| Need threshold — safety (T_safety) | 0.2 | Needs | Appendix A |
| Need threshold — social (T_social) | 0.2 | Needs | Appendix A |
| Long-term goal revision cadence | monthly (or major life event) | Long-term Goals | digest §1.5 |
| Social isolation threshold | <3 unique contacts / 7 days | Long-term Goals | digest §1.5 |
| Financial stress condition | income < 0.9 × expenses | Long-term Goals | digest §1.5 |
| Interest window | last 30 days visited POIs, satisfaction belief > 0.5 | Long-term Goals | digest §1.5 |
| Planning block minimum granularity | 5 minutes | Planning | digest §2.1 |
| Value-driven leisure candidates (N) | 3 | Planning | digest §2.1, Appendix A |
| Macro-area candidates considered | top 10 nearby areas | Place Selection | digest §2.2, Appendix A |
| Micro POI candidate cap | up to 200 | Place Selection | digest §2.2, Appendix A |
| Gravity model distance decay (γ) | 2.0 | Place Selection | digest §2.2 eq. 1, Appendix A |
| Gravity model stability constant (ε) | 10⁻³ | Place Selection | digest §2.2 eq. 1, Appendix A |
| Vehicle set | walk, bicycle, car, bus, train | Vehicle Selection | digest §2.3, Appendix A |
| Social belief dimensions | affinity, trust, familiarity | Social | digest §3 |
| Face-to-face partner cap | 1 partner / 30-minute tick | Social | digest §3, Appendix A |

## Pilot-specific — not given numerically by the paper, need our own defaults

| Parameter | Status | Notes |
|---|---|---|
| Need decay rates (α_n, one per need) | **unset** | Paper gives functional form only. Tune against Appendix C.3 archetype-trajectory shapes. See `modules/04-needs.md`. |
| Population size | **proposed: 200–500 agents** | See `01-vision-and-scope.md`. |
| Simulated duration | **proposed: 1–2 weeks** | See `01-vision-and-scope.md`. |
| Study area | **unset** | Which community area(s) — see `open-questions.md`. |
| Agent-cognition LLM | **unset** | Cheap/mini-tier target, exact model TBD — see `open-questions.md`. |
| Judge LLM | **unset** | Stronger-tier target for evaluation only — see `open-questions.md`. |
| Embedding model (memory similarity) | **unset** | Infrastructure choice, not methodology — see `modules/02-memory.md`. |
| Long-term goal revision cadence for pilot | **proposed: keep monthly (default)** | See `modules/05-long-term-goals.md` for the weekly-fallback option. |
| Distance metric for gravity model (D_ij) | **proposed: network distance via OSM graph** | See `modules/08-place-selection.md`. |
| Social network seeding rule (contacts/agent, tie weighting) | **unset** | See `modules/10-social.md`. |
