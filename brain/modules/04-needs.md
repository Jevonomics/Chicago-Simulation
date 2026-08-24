# Module: Needs

## What the paper does

See digest §1.4. Four needs (hunger, energy, safety, social) ∈ [0,1], LLM-initialized daily, decaying
continuously (`s_n(t) = max(0, s_n(t−Δt) − α_n·Δt)`), LLM-updated after activities/events, prioritized
`hunger > safety > energy > social` with thresholds `T_hunger=0.3, T_energy=0.3, T_safety=0.2,
T_social=0.2`. Interruption logic: a higher-priority need crossing its threshold can break an ongoing
plan. Ablation shows this module matters most for Activity and Event Reaction realism.

## Chicago pilot adaptation

- **Keep thresholds and priority order identical to the paper** — there's no principled reason to
  retune these for Chicago, and matching them keeps the pilot a faithful test of the mechanism rather
  than a new mechanism.
- **Decay rates (`α_n`):** the paper doesn't publish numeric values for these — only the functional
  form. Pilot needs to choose values that produce plausible within-day dynamics (e.g. hunger crossing
  threshold roughly every 4–6 hours for a typical adult). Plan: hand-tune against the qualitative
  check in Appendix C.3 (five archetype agents' need trajectories over a day, digest §4) as the target
  shape, rather than against any numeric ground truth — none exists publicly. Record chosen values in
  `parameters.md` once set, with the reasoning.
- **Archetype validation:** deliberately replicate the paper's Appendix C.3 case study — simulate a
  small number of named archetypes (e.g. office worker, student, shift worker, retiree, freelancer)
  and sanity-check their need trajectories look distinct and plausible before trusting the same needs
  module across the full synthetic population. Cheap, high-signal, good first integration test.

## Open questions

- Decay-rate values — currently unset, tracked in `open-questions.md` until first tuning pass.
