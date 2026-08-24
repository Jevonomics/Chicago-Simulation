# Module: Perception / Dispatcher

## What the paper does

See digest §1.6. At each tick, receives an environment observation, decides whether the agent should
react at all, and if so, asks the LLM to pick the most appropriate module (from a list of available
modules with functional descriptions) to handle the situation; a dispatcher then invokes it with the
agent's inferred needs, a short explanation, and required parameters. This is the agent's top-level
control/routing loop — everything else in Section 3 is a "module" this one can select.

## Chicago pilot adaptation

- **Keep the two-stage structure (react-or-not, then select-module)** — it's cheap (well short of a
  full planning call) and is what keeps the agent from re-planning or re-socializing every single
  5-minute tick, which matters even more at pilot LLM-cost budgets than at paper scale.
- **Module registry:** the set of invokable modules mirrors the paper's own list — planning (recursive
  + value-driven leisure fill), place selection, vehicle selection, social interaction, belief
  update/reflection triggers. No new modules invented for the pilot; this file's job is routing, not
  capability.
- **Cost control lever:** since this fires every tick for every agent, it's the single highest-volume
  LLM call in the whole system. Worth considering a cheap non-LLM pre-filter (e.g. "no action needed"
  short-circuit when the agent is mid-activity and nothing salient changed) before falling back to an
  LLM call — the paper doesn't specify this, but it's consistent with the paper's own framing of this
  step as a lightweight react/don't-react gate, not full reasoning.

## Open questions

- Whether a rule-based pre-filter (see above) is faithful enough to the paper's intent or drifts too
  far from "modeling the mechanism" — worth a comment in `decisions-log.md` once decided either way.
