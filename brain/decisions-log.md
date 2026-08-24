# Decisions Log

Append-only. Each entry: date, decision, why, what it affects. Don't rewrite history — if a decision
is later reversed, add a new entry saying so and link back to the original.

---

## 2026-08-24 — Brain established; pilot framed as mechanism test, not scale/data replication

**Decision:** ChicagoSim will faithfully keep every cognitive/behavioral module from CitySim (persona,
memory, beliefs, needs, long-term goals, perception/dispatcher, planning, place selection, vehicle
selection, social) but scale everything else down — population (~200–500 agents), geography (1–3
Chicago community areas), duration (1–2 simulated weeks), and validation scope (skip well-being study
and true crowd-density validation; both require data we don't have access to).

**Why:** The paper's core contribution is the *mechanism* (recursive value-driven planning +
belief-driven place/vehicle choice + decaying needs + evolving goals), not its scale or its Japan-
specific data. A pilot that keeps the mechanism intact but shrinks scale/data-scope tests the thing
that actually matters, without pretending to have data (proprietary surveys, smartphone panels) that
doesn't exist for Chicago.

**Affects:** `01-vision-and-scope.md` (primary), all of `modules/*.md`, `evaluation-plan.md`.

---

## 2026-08-24 — Persona sourcing: ACS PUMS weighted resampling, not independent-marginal sampling

**Decision:** Synthetic personas will be built by resampling real ACS 5-Year PUMS records (using
Census person/household weights) rather than sampling age/income/occupation/etc. independently from
separate marginal distributions.

**Why:** Independent-marginal sampling produces internally-inconsistent personas (e.g. implausible
age/occupation/income combinations). PUMS records preserve real joint structure because each record
is a real (anonymized) person. This is standard practice in synthetic-population/microsimulation work.

**Affects:** `modules/01-persona.md`, `data/persona-initialization.md`.

---

## 2026-08-24 — Home/work anchors via LEHD LODES, not density-proxy assignment

**Decision:** Work/school location anchors will be assigned using LEHD LODES home-block→work-block
commute-flow data, rather than the paper's own approach of density-proxy assignment via OSM population
density.

**Why:** LODES captures actual observed commute structure for the whole US, which is a strictly better
signal than a density proxy where it's available — and it is available for Chicago. Using it also
gives the travel-pattern validation (`evaluation-plan.md`) a realistic commute relationship to
reproduce.

**Affects:** `modules/01-persona.md`, `data/persona-initialization.md`, `data/chicago-data-sources.md`.

---

## 2026-08-24 — Study area unit: Chicago's 77 official Community Areas

**Decision:** The place-selection module's "macro-level area" concept, and the pilot's geographic
scope generally, will use Chicago's official Community Area boundaries rather than an arbitrary grid
or radius.

**Why:** Community Areas are a real, city-published, well-known unit that Chicago data (Census
crosswalks, city open data, commute flows) is commonly organized around — using them keeps the pilot
grounded in units a Chicago audience would recognize, and avoids inventing an arbitrary spatial
partition the paper doesn't require.

**Affects:** `data/chicago-data-sources.md`, `modules/08-place-selection.md`, `01-vision-and-scope.md`.

---

## 2026-08-24 — Correction: resolved simulation-loop call-frequency contradiction; added budget/reliability scaffolding

**What was wrong:** `02-citysim-methodology-digest.md`'s Algorithm 1 pseudocode, as transcribed from
the paper's Appendix B.1, nested `perceive()`, `decide_action()`, and `reflect()` all inside the
per-5-minute-tick loop — literally read, that's an LLM-scale call every tick, all day, for every
agent. This directly contradicted two other parts of the same digest: §1.2 (Memory module), which
states reflective memory is synthesized once per day, not per tick; and §1.6 (Perception module),
which describes the paper's own two-tier design (a cheap react/don't-react check, then an LLM
dispatch only if warranted) — a gate the pseudocode's structure didn't visually capture. Left
unresolved, a builder implementing the pseudocode literally would over-generate LLM calls by roughly
10-20x versus what the paper's own mechanism implies, and the brain would have been asserting three
mutually-inconsistent things without flagging it.

**Decision:**
1. Corrected the Algorithm 1 pseudocode in `02-citysim-methodology-digest.md` so `reflect()` sits
   outside the timestep loop (once/agent/day) and `decide_action()` is gated behind a cheap non-LLM
   `perceive()` check, and added a "Resolved ambiguity" note documenting the three conflicting
   descriptions and the adopted interpretation explicitly, rather than leaving it implicit.
2. `03-architecture.md`'s Simulation loop layer (layer 4) now states this resolved interpretation
   directly and warns a builder reading only that doc not to reintroduce per-tick LLM calls.
3. Added `cost-and-budget.md`: a worked ~41-calls/agent-day estimate under the resolved
   interpretation (within the paper-implied ~40–60/day range), a >100-calls/agent-day runtime signal
   that the gate isn't working, a stated Grok-mini-tier budget-ceiling framing ($100 of a $2,500 total
   pilot budget, placeholder pending confirmation), and — critically — the calls × agents × days check
   run *against* the pilot's proposed 200–500 agent / 1–2 week scope, which surfaced a real coupling:
   on Grok's mini tier alone, $100 only comfortably covers ~200 agents × 7 days, not the fuller
   350–500 × 14 lean already recorded in `parameters.md` (that lean was costed against OSS tiered
   routing, not Grok). This doesn't resolve the model-choice question — it sharpens the existing open
   item in `open-questions.md` with a concrete dollar consequence.
4. `03-architecture.md`'s AgentSociety question is now resolved as a stated default rather than left
   open: build ChicagoSim's own simplified loop, do not adopt the AgentSociety platform, revisit only
   if the pilot succeeds and scale-up is deliberately chosen. Removed the corresponding entry from
   `open-questions.md`.
5. Removed `modules/06-perception-dispatcher.md`'s open question about whether a rule-based pre-filter
   is faithful to the paper's intent — resolved by the same "Resolved ambiguity" note: yes, the
   two-tier gate is what the paper's own §3.1.6 describes, not a deviation from it.
6. Added a pre-flight structured-output reliability gate to `01-vision-and-scope.md` (>5% parse
   failure rate in any module, measured on a 20-30 agent / 1-2 day test batch, blocks proceeding to
   the full run) and a calibrate/validate separation note (needs decay rates α_n, the one genuinely
   unspecified pilot-side free parameter, must be tuned against a different data source than the one
   used to validate against — corrected the request's premise that the gravity model's γ is also a
   free parameter; it's copied verbatim from the paper as a fixed constant, not fit by us).

**Why:** A "brain" that contains an internal contradiction on something as consequential as LLM call
volume (a 10-20x cost/throughput swing) is worse than useless — it actively misleads whoever
implements from it. Resolving it required picking an explicit interpretation and saying so, not
averaging the conflicting descriptions or leaving it for an implementer to guess.

**Affects:** `02-citysim-methodology-digest.md`, `03-architecture.md`, `01-vision-and-scope.md`,
`cost-and-budget.md` (new), `open-questions.md`, `INDEX.md`. No scope decisions (population, geography,
module count, duration) were changed — this was a correctness/scaffolding pass only.

---

(No decisions have been reversed yet.)
