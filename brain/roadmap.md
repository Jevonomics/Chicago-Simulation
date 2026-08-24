# Roadmap

Phased build plan. Update the status section after every work session — this should reflect reality,
not aspiration.

## Phase 0 — Brain (current)

Build and maintain the markdown knowledge base in this directory: methodology digest, architecture
proposal, per-module adaptation plans, data-source mapping, parameters, evaluation plan. No code yet.

**Status: in progress.** Initial pass complete (2026-08-24): index, vision/scope, methodology digest,
architecture proposal, all 10 module docs, 2 data docs, parameters table, evaluation plan, decisions
log, open questions. Scope-defining open questions (study area, PUMA crosswalk, LLM choice) still
unresolved — see `open-questions.md`.

## Phase 1 — Resolve scope-defining decisions

- Pick the study area (community area pair with a real commute relationship).
- Resolve the PUMA↔community-area crosswalk for that area.
- Pick agent-cognition and judge LLMs.
- Check whether AgentSociety's codebase is available to build on.

Exit criterion: `open-questions.md`'s "Scope-defining" section is empty.

## Phase 2 — World + population layer

`data/data-gathering-checklist.md` is the actionable sequencing for this phase (and the tail end of
Phase 1) — account setup, what's blocked on the study-area choice vs. what isn't, and the PUMA↔
community-area crosswalk task.

- Pull OSM street graph, POIs, CTA GTFS for the study area.
- Build the ACS PUMS → LODES → synthetic-persona pipeline (`data/persona-initialization.md`).
- Sanity-check: does the sampled population's demographic profile look like the real study area?

Exit criterion: a persisted population of personas, each with home/work anchors, ready to drive
agents.

## Phase 3 — Cognitive modules, smallest-first

Build order follows the evaluation plan's execution order (`evaluation-plan.md`), since it front-loads
the cheapest, highest-signal test:

1. Needs module + a handful of named archetype agents (no full population, no other modules needed
   yet) → replicate the Appendix C.3 needs-trajectory sanity check.
2. Persona, memory (temporal/reflective/spatial), beliefs modules.
3. Perception/dispatcher, planning (recursive + value-driven), place selection, vehicle selection.
4. Long-term goals, social.

Exit criterion: a single agent can run the full daily loop (Algorithm 1) end-to-end for one simulated
day without falling back to any module.

## Phase 4 — Full-population short run

- Run the full ~200–500-agent population for the target 1–2 simulated weeks.
- Produce core evaluation outputs: time-use-by-category vs. ATUS, hourly travel volume vs. CMAP/CTA.

Exit criterion: `evaluation-plan.md`'s "core" rows for time-use and travel patterns produce a result,
whether or not it looks good — the point of a pilot is finding out.

## Phase 5 — Ablation study (headline result)

- Re-run with belief / recursive-planning / long-term-goal / needs / persona modules individually
  removed, score via the same LLM-judge Likert method as the paper's Appendix C.2/C.5.

Exit criterion: `01-vision-and-scope.md`'s third success criterion (ablation visibly degrades
plausibility) is either confirmed or refuted with evidence.

## Phase 6 — Stretch goals, if time/budget allow

POI popularity prediction, Divvy-based crowd-density proxy, belief-estimation-by-backbone comparison.
Not required for the pilot to be considered successful.

---

## Current status (update this section each session)

**As of 2026-08-24:** Phase 0 in progress. No code exists. Next action: resolve Phase 1's
scope-defining open questions, starting with study-area selection.
