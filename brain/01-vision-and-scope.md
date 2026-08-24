# Vision & Scope

## What CitySim is, in one paragraph

CitySim simulates a city's population as LLM-driven agents, each with a survey-derived persona,
evolving beliefs about places, decaying physiological/social needs, monthly-revised long-term goals,
and layered memory (temporal/reflective/spatial). Each simulated day, agents recursively plan a
schedule (mandatory blocks → medium-priority blocks → value-driven leisure blocks), pick where to go
via a belief-weighted gravity model, pick how to get there via an LLM vehicle choice, and interact
socially through an evolving belief-weighted contact network. The paper validates the result against
real Japanese time-use surveys, mobility data, POI popularity, well-being surveys, and crowd density —
at up to 1,000 agents (with a scalability test to 10^6 query-only agents).

## What "lightweight pilot" means for ChicagoSim

We are explicitly **not** trying to replicate the paper's scale or its access to proprietary data
(a custom Japan survey, a proprietary city-scale mobility dataset, a proprietary well-being survey,
smartphone location panels). Those don't exist for us. The pilot's job is to prove the *mechanism*
— recursive value-driven planning, belief-driven place/vehicle choice, decaying needs, evolving
goals — produces recognizably human, Chicago-plausible behavior at small scale, using only public data.

Concretely, "lightweight" means:

- **Population:** low hundreds of agents (target ~200–500), not thousands.
- **Geography:** 1–3 contiguous Chicago community areas (see `data/chicago-data-sources.md`), not
  the full city. A neighborhood pair with a real commute relationship (a residential area + a job-dense
  area, e.g. a South/West Side neighborhood ↔ the Loop) is more interesting than one isolated area.
- **Duration:** simulate on the order of 1–2 weeks of agent-days, not two months.
- **Model cost:** default to a cheap/fast LLM for agent cognition (e.g. a mini-tier model or a local
  open-weight model); reserve a stronger model for the LLM-as-judge evaluation only, mirroring the
  paper's GPT-4o-mini-agents / GPT-4o-judge split.
- **Validation:** reuse public, free equivalents of the paper's ground truth (ATUS/CMAP for time-use
  and mobility, Google/Yelp for POI popularity) and skip validations that require data we can't get
  (proprietary well-being survey, smartphone crowd panels) — see `evaluation-plan.md` for the mapping
  and what's marked as a stretch goal vs. core.
- **Module fidelity, not module count:** we keep all of the paper's cognitive modules (persona, memory,
  beliefs, needs, goals, perception/dispatcher, planning, place selection, vehicle selection, social) —
  cutting a module would defeat the point of testing the mechanism — but we simplify each module's
  internals where the paper's version assumes scale or data we don't have (e.g. fewer memory nodes
  retrieved, smaller candidate sets, simpler embedding similarity). Details per-module in `modules/`.

## Explicit non-goals for the pilot

- Not building a general-purpose ABM platform (AgentSociety already exists for that; the paper's
  authors *ran CitySim inside AgentSociety*). We may borrow ideas but aren't reimplementing that
  platform layer from scratch unless a lighter existing tool doesn't fit — see `open-questions.md`.
- Not attempting the 10^3–10^6 scalability benchmark from Appendix C.1.
- Not attempting the well-being classification study (Section 4.5) unless a suitable public survey
  substitute is found — currently marked stretch.
- Not attempting true smartphone-panel crowd-density validation (Section 4.6) — Divvy trip data is
  used as a free directional proxy instead, explicitly weaker ground truth.

## Success criteria for the pilot

A pilot is "working" if, for the chosen Chicago area(s):

1. Simulated time-use-by-activity-category for a synthetic population resembles ATUS patterns for
   comparable demographics (visual/qualitative match is enough for a pilot; no formal statistical test required).
2. Simulated hourly travel volume shows a real weekday commute double-peak and a flatter weekend
   pattern, directionally matching CMAP/CTA patterns.
3. An ablation (remove beliefs, remove needs, remove recursive planning) visibly degrades
   plausibility under an LLM-judge rubric, replicating the paper's Table 4 finding that these modules
   matter — this is the cheapest way to confirm the mechanism is actually doing something, and doesn't
   depend on us matching the paper's absolute numbers.

If those three hold at ~300 agents over ~1 week, the pilot has done its job and a scale-up decision
can be made deliberately, not by default.
