# Persona Initialization — Method

How synthetic Chicago personas get generated for the pilot, expanding on `modules/01-persona.md`.

## Approach: weighted resampling from ACS PUMS, not independent marginals

The paper's proprietary survey gives it genuine joint distributions (a real respondent's age, income,
occupation, and household composition all come from the same person, so they're internally
consistent). Naively sampling age/income/occupation/etc. independently from separate marginal
distributions would produce implausible combinations (e.g. a retiree with a full-time student life
stage). Instead:

1. Pull ACS 5-Year PUMS records for the PUMA(s) overlapping the chosen study area (see
   `open-questions.md` for the PUMA↔community-area mapping).
2. Each PUMS record already *is* a real (anonymized, weighted) household/person — sample records
   directly, using the Census-provided person/household weights, rather than fitting and resampling
   from marginal distributions. This preserves real joint structure automatically and is the standard
   technique behind most synthetic-population work (e.g. transportation-planning microsimulation).
3. Assign each sampled person a home location: draw a residential parcel/block within the study area,
   weighted by that block's population (from decennial Census block population counts).
4. For employed/school-age persons, assign a work/school anchor using LEHD LODES home-block→
   work-block flow data, constrained to workplaces that exist within or are commutable from the study
   area — this is what gives the pilot a real commute relationship to validate travel patterns against
   (`evaluation-plan.md`).
5. Layer on the attributes PUMS doesn't cover — Big Five traits and habits/preferences — via the
   heuristics described in `modules/01-persona.md` (population-level Big Five distributions;
   hand-authored habit taxonomy conditioned on demographics). These are explicitly the biggest
   fidelity gap vs. the paper and should be labeled as such in any pilot writeup.

## What this buys us vs. the paper

- Real joint demographic structure (age×income×occupation×household composition) — arguably *more*
  grounded than assuming the paper's own survey-to-simulation pipeline, since PUMS is a large,
  well-vetted public dataset.
- Real commute structure via LODES, rather than a population-density proxy.

## What this doesn't buy us

- No personality/psychographic ground truth (Big Five, habits) — heuristic, not surveyed.
- No behavioral/preference ground truth beyond what can be inferred from static demographics — the
  paper's habits/preferences presumably came from actual survey questions about behavior, which PUMS
  doesn't ask.

## Fairness note

Section 7 (Ethics Statement) and Appendix B of the source paper both flag that persona fidelity
depends on data richness, and that under-observed/marginalized groups get less faithful personas as a
result — worth keeping in mind when picking which community area(s) to model: a lower-income or
historically under-surveyed area will likely have thinner PUMS/LODES coverage relative to a
wealthier one, and that asymmetry should be acknowledged rather than papered over if it shows up.
