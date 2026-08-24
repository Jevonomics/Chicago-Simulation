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
  platform layer from scratch — `03-architecture.md` resolves this as a stated default (build our own
  simplified loop), not an open question.
- Not attempting the 10^3–10^6 scalability benchmark from Appendix C.1.
- Not attempting the well-being classification study (Section 4.5) unless a suitable public survey
  substitute is found — currently marked stretch.
- Not attempting true smartphone-panel crowd-density validation (Section 4.6) — Divvy trip data is
  used as a free directional proxy instead, explicitly weaker ground truth.

## Pre-flight check: structured-output reliability

Before committing to the full 200–500 agent / 1–2 week run, run a small test batch — ~20–30 agents,
1–2 simulated days — and track the **structured-output parse failure rate per module**: the fraction
of LLM calls whose response fails to parse against the expected schema (malformed JSON, missing
required fields, an out-of-range value). This is distinct from the behavioral-quality success criteria
below — it's about whether the plumbing is reliable enough to trust before spending the pilot's
population/duration budget on it.

**Threshold:** if any single module shows a **>5% parse failure rate** in the test batch, treat that
as a blocker — add a retry wrapper (re-prompt on parse failure, capped at N attempts) or revise that
module's prompt/schema before proceeding to the full run. Don't average the rate across modules; one
unreliable module can silently corrupt downstream state even if every other module is fine. The most
likely candidates for higher failure rates are the calls returning the most complex structured output
— the multi-dimensional belief/observation call (`modules/03-beliefs.md`) and the N=3-candidate
value-driven leisure-planning call (`modules/07-planning.md`) — worth checking first.

This test batch is also where the calls/agent-day budget check in `cost-and-budget.md` should first be
measured against reality, and where the runtime signal in that file (>100 calls/agent-day → the
perception gate isn't working as intended) should first be checked. Both checks belong in the same
pre-flight pass, before the full run, not discovered partway through it.

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

## Calibration vs. validation: don't fit and check against the same data

Some of the methodology's parameters are pilot-specific free choices, not values given by the source
paper — most notably the **needs decay rates α_n** (`modules/04-needs.md`, `parameters.md`), which the
paper specifies only as a functional form, never numerically. (The gravity model's distance-decay `γ`
is *not* in this category — it's copied verbatim from the paper as a fixed constant, per
`parameters.md`; it isn't ours to fit.) Where a pilot-specific parameter genuinely does need tuning
against data, that tuning has to be kept separate from validation, or success criteria 1–2 above become
circular — a parameter fit to match ATUS will trivially "validate" against ATUS.

Concretely: if α_n is tuned against the Appendix-C.3-style archetype need-trajectory shapes (as
`modules/04-needs.md` currently proposes), the time-use and travel-pattern checks in
`evaluation-plan.md` should be run against an **independent** source not used in that tuning — e.g.
calibrate against archetype trajectories, then validate against ATUS/CMAP, not the reverse and not
both from the same source. This should become an explicit step in `evaluation-plan.md`'s execution
order rather than staying an implicit assumption.
