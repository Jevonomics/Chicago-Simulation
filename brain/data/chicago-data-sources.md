# Chicago Data Sources

Maps every data need identified in the methodology digest and module files to a real, freely
available Chicago/US data source. This is the pilot's substitute for the paper's proprietary Japan
survey, proprietary city-scale mobility dataset, and smartphone-panel crowd data.

## Study area definition

- **Chicago Community Areas (77 total)** — the city's official, long-standing neighborhood-grouping
  boundaries, published by the Chicago Data Portal ("Boundaries - Community Areas" dataset). This is
  the natural unit for `01-vision-and-scope.md`'s "1–3 community areas" scope and for the place-
  selection module's "macro-level area" concept (`modules/08-place-selection.md`).
- Suggested pilot candidates (not yet chosen — see `open-questions.md`): a residential area with a
  real commute relationship to a job-dense area, e.g. a South/West Side residential community area
  paired with the Loop (job-dense, transit-hub, dense POI coverage) — mirrors the paper's own
  home/work spatial realism goal and gives the travel-pattern validation something real to show.
  Named candidates for the residential side, to be checked against actual LODES flow volume once
  pulled (see `data-gathering-checklist.md`) rather than picked on demographic plausibility alone:
  - **Pilsen / Lower West Side** — Pink Line + 18th St, moderate size (~35k), has enough internal POI
    density to be interesting even before counting the Loop pairing.
  - **North Lawndale** — Pink Line, well-documented West Side community area, strong residential/
    job-dense contrast with the Loop.
  - **Austin** — Green/Blue Line access, largest West Side community area by population, real
    commute-flow volume to the Loop worth confirming via LODES.

## Population / persona data

- **ACS 5-Year PUMS (Public Use Microdata Sample)**, via the Census Bureau API or IPUMS USA. Gives
  individual/household-level microdata (age, sex, income, education, occupation, household
  composition) at the PUMA level — PUMAs are larger than community areas, so a crosswalk/filter is
  needed (`open-questions.md`). This is the direct substitute for the paper's proprietary Japan survey
  (`modules/01-persona.md`).
- **LEHD LODES (LEHD Origin-Destination Employment Statistics)**, Census Bureau — census-block-level
  home→work commute flow data for the whole US. Substitute for the paper's OSM-population-density-based
  home/work assignment; arguably better since it's actual observed commute structure, not a density
  proxy.

## Street network, POIs, buildings

- **OpenStreetMap** (via `osmnx`/Overpass) — street graph, building footprints, POI points with
  `amenity`/`shop`/`leisure` tags. Same source the paper itself uses for spatial structure. Free,
  well-covered for Chicago.
- **Chicago Data Portal** (data.cityofchicago.org) — supplementary POI-adjacent datasets: Business
  Licenses (active business locations/categories), Building Footprints, Boundaries (community areas,
  wards, census tracts).

## Transit

- **CTA GTFS feed** (transitfeeds.com / CTA developer site) — bus/rail routes, stops, schedules. Used
  by the vehicle-selection module to ground transit availability in reality
  (`modules/09-vehicle-selection.md`).
- **CTA ridership data** (Chicago Data Portal: "CTA - Ridership - Daily Boarding Totals",
  "CTA - Ridership - 'L' Station Entries") — freely published daily/monthly ridership, usable as a
  coarse real-world check on simulated transit-mode volume, though not agent-level ground truth.

## Mobility / travel-pattern ground truth

- **CMAP (Chicago Metropolitan Agency for Planning) — My Daily Travel Survey / Travel Tracker Survey**
  — regional household travel survey with trip purpose, mode, time-of-day. This is the closest public
  equivalent to the paper's proprietary city-scale mobility dataset (§4.3) — primary candidate for
  the hourly-travel-volume validation in `evaluation-plan.md`.
- **Divvy trip data** (published historical CSVs, divvybikes.com/system-data) — free, agent-level bike
  trip records (start/end station, timestamps) for actual Chicago cycling. Useful both as bike-mode
  ground truth and as a rough spatial-flow/footfall proxy where better data isn't available (e.g. for
  the macro-area "popularity" input in place selection).

## Time-use ground truth

- **American Time Use Survey (ATUS)**, BLS — the direct US analogue of the Japanese national time-use
  survey the paper validates against (§4.1). No Chicago-specific breakout exists; regional (Midwest
  urban) or national demographic-group breakdowns are the best available proxy — an accepted
  approximation, same spirit as the paper's own age-group aggregation.

## POI popularity / ratings ground truth

- **Google Places API** and/or **Yelp Fusion API** — POI category, price level, rating, review count.
  Substitute for the paper's Google Maps ratings ground truth (§4.4). Also feeds POI descriptions for
  the belief module (`modules/03-beliefs.md`). Note: both have usage/rate limits and terms of service
  to respect at whatever POI volume the pilot needs — check quota needs before relying on either as a
  hard dependency.

## Explicitly unavailable / stretch-goal data

- **Well-being survey** (paper §4.5, 1,200 Japan responses) — no free direct equivalent. Possible
  distant proxies: Healthy Chicago Survey / Chicago Health Atlas public indicators (community-area-
  level, not individual-level — not enough to replicate the paper's per-respondent methodology).
  Marked **stretch/likely-skip** in `01-vision-and-scope.md`.
- **Smartphone-panel crowd density** (paper §4.6) — no free equivalent at that granularity. Divvy trip
  data is the closest free proxy for spatial flow, but it's bike-only and not a true crowd-density
  measure. Marked **stretch, weaker ground truth accepted** in `01-vision-and-scope.md`.
