# Module: Persona

## What the paper does

See `02-citysim-methodology-digest.md` §1.1. Fixed-at-creation record: demographics (name, age,
gender, occupation, income, hobbies, education, household composition, life stage), spatial anchors
(home, work/school), Big Five on a 1–3 scale, and empirically-derived habits/preferences. Sourced from
a proprietary Japan survey validated against census/lifestyle-survey distributions; home/work assigned
via OSM population density.

## Chicago pilot adaptation

- **Source distribution:** ACS 5-year PUMS microdata for the Chicago PUMAs covering the study area(s)
  — gives real joint distributions of age/sex/income/education/household composition/occupation
  industry, so sampled personas aren't independent-marginal draws (which would produce implausible
  combinations, e.g. a 12-year-old with a management occupation). See `data/persona-initialization.md`
  for the exact sampling method.
- **Home/work anchors:** home drawn from the study area's residential parcels/census blocks weighted
  by population; work location drawn using **LEHD LODES** origin-destination flows so home→work pairs
  reflect real commute patterns instead of nearest-job assignment.
- **Big Five:** paper doesn't say how these are elicited beyond "questionnaire-derived." No public
  Chicago-specific personality survey exists at this granularity — plan is to sample from published
  population-level Big Five distributions (not Chicago-specific, that's an accepted approximation) and
  discretize to 1–3, matching the paper's scale exactly so downstream prompts are unaffected.
- **Habits/preferences:** the paper is vague on exact elicitation here too. Pilot plan: derive a small
  fixed vocabulary of habits (early riser / night owl, home cook / eats out, transit-preferring /
  car-preferring, etc.) and assign each stochastically conditioned on the persona's demographics,
  documented explicitly as heuristic (not survey-derived) — flag this honestly in any write-up, since
  it's the biggest fidelity gap vs. the paper in this module.
- **Scale:** ~200–500 personas total for the pilot, see `01-vision-and-scope.md`.

## Open questions

- Exact PUMA-to-community-area crosswalk needed once the study area is chosen — see `open-questions.md`.
- Whether to hand-author a small habit taxonomy or attempt to mine it from a public lifestyle survey
  (e.g. a Pew or Gallup public dataset) — currently leaning hand-authored for pilot simplicity.
