# Module: Planning (Recursive, Value-Driven)

## What the paper does

See digest §2.1 and Appendix A.1.1. Two-step recursive decomposition of the day into 5-minute-granular
blocks: (1) mandatory block assignment (sleep, work, appointments) from persona/occupation/needs, with
subdivision if an activity doesn't fill its interval; (2) medium-priority recursive filling of
remaining `[EMPTY]` blocks (meals, hygiene, errands). Leftover empty blocks are filled **at execution
time**, not pre-planned, via value-driven planning: generate N=3 candidate leisure/goal-directed
activities per block, imagine each one's resulting need/desire state, pick the best — one structured
LLM call with internal reasoning. Ablation shows removing recursive planning hurts Activity and
Mobility scores most.

## Chicago pilot adaptation

- **Keep N=3 candidates and the two-mandatory/medium-priority-then-value-driven structure exactly as
  specified** — this is the mechanism most central to the paper's contribution over prior work (digest
  intro: "agents often plan activities in a fixed sequential manner" is explicitly the gap CitySim
  claims to close), so faithfulness here matters more than almost anywhere else in the pilot.
- **Occupation/schedule inputs:** mandatory-block content (work hours, school hours) is driven by the
  persona's occupation/life-stage fields (from ACS PUMS, `modules/01-persona.md`). For students, use
  typical Chicago Public Schools hours as a default schedule template; for standard employment, use
  typical local work-hour norms unless the persona's occupation implies otherwise (e.g. shift work for
  healthcare/service occupations, matching the paper's own night-shift-nurse archetype in Appendix
  C.3).
- **Execution-time leisure fill is the expensive part cost-wise** (one structured LLM call per empty
  block per agent per day) — at pilot scale (~300 agents × maybe 2–4 empty blocks/day × ~10–14 days)
  this is on the order of a few thousand to ~10k such calls total, which should be affordable on a
  mini-tier model; worth tracking actual cost once implemented (`roadmap.md`).

## Open questions

- Chicago-specific default schedule templates (school hours, typical shift patterns per occupation
  category) still need to be assembled — see `open-questions.md`.
