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

(No decisions have been reversed yet.)
