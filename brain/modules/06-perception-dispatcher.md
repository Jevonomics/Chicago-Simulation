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
- **Resolved: the react-or-not check itself is a cheap, non-LLM pre-filter, not an LLM call.** This
  was previously described below as an open question ("worth considering," "the paper doesn't specify
  this") — it's now resolved, per `02-citysim-methodology-digest.md`'s "Resolved ambiguity" note under
  the Algorithm 1 pseudocode: the paper's own §3.1.6 describes exactly this two-tier design (a
  react/don't-react check, then an LLM dispatch only if warranted), so a rule-based pre-filter (has the
  agent arrived somewhere, has a needs threshold crossed, is a co-located agent available, has the
  current block ended) is not a deviation from the paper's intent — it's the correct reading of it.
  Only `decide_action()` (module selection) is an LLM call, and only when the pre-filter flags a
  reaction is warranted. See `cost-and-budget.md` for the resulting calls/agent-day estimate.
