# Data Gathering — Action Checklist

Practical "what do I actually go do" companion to `chicago-data-sources.md` (which maps *what* source
covers *what* need) and `persona-initialization.md` (which explains the persona-sampling *method*).
This file is about sequencing and access — account setup, what's blocked on what, and the one real
data-engineering task in the pipeline. Cost/usage-tier sizing for the paid-capable APIs (Google
Places, Yelp) is deliberately deferred here, per the current focus — just get accounts provisioned so
nothing blocks later; size usage when it's actually time to pull POIs.

## Step 0 — study area (blocks most of Tier D below)

Almost everything in Tier D can't start until this is picked. `chicago-data-sources.md` now lists
three named candidates (Pilsen/Lower West Side, North Lawndale, Austin) paired with the Loop — pick
one to unblock work, and treat it as provisional until checked against real LODES commute-flow volume
once that data's pulled (cheap to verify, not worth blocking on upfront). See the question at the end
of this session's chat, or `open-questions.md`, for where this gets confirmed.

## Environment setup (do once)

Python packages covering everything below: `geopandas`, `shapely`, `osmnx`, `pandas`, `requests`,
`census` (Census API wrapper) or an IPUMS API client, `pygris` (Census TIGER/Line boundary puller),
`gtfs-kit` or `partridge` (GTFS parsing). All pip-installable, no paid dependencies.

## Tier A — no signup needed, start today regardless of study area

| Source | What | Where |
|---|---|---|
| OpenStreetMap | Street graph, POIs, building footprints | Geofabrik regional `.osm.pbf` extract (download once, clip locally) rather than hammering the public Overpass API repeatedly via `osmnx` |
| Chicago Community Area boundaries | Study-area polygon | Chicago Data Portal, "Boundaries - Community Areas" — direct GeoJSON/shapefile |
| Census PUMA boundaries | For the PUMA↔community-area crosswalk (see below) | Census TIGER/Line, 2020 PUMAs — direct shapefile, or via `pygris` |
| Census tract/block boundaries + population | For population-weighting the crosswalk | Census TIGER/Line + decennial Census — direct, or via `pygris` |
| CTA GTFS static feed | Bus/rail routes, stops, schedules | transitchicago.com/developers — direct zip download, no key needed for the static feed (only GTFS-realtime needs a registered key, and we don't need that) |
| CTA ridership data | Daily boardings, 'L' station entries | Chicago Data Portal — direct CSV |
| Divvy trip history | Agent-level bike trips | divvybikes.com/system-data — direct CSV/zip, historical months |
| CMAP My Daily Travel Survey | Regional household travel data (trip purpose, mode, time-of-day) | CMAP Data Hub (datahub.cmap.illinois.gov) — public-use dataset, direct download, confirmed no account/request process needed |
| ATUS public-use files | US time-use survey (validates against §4.1's methodology) | bls.gov/tus — direct download. **Heads up:** it's a multi-table relational dataset (Respondent, Activity, Roster, Activity Summary files, plus the ATUS-CPS linking file for demographics) — budget time to merge, not just download |
| LEHD LODES OD flow files | Home→work commute flows | lehd.ces.census.gov/data — gzipped CSV by state, direct download |

## Tier B — free account/key, ~5-10 min each, worth doing today

| Source | Signup | Notes |
|---|---|---|
| Census API key | api.census.gov/data/key_signup.html | Instant email. Only needed if pulling ACS PUMS via the API rather than bulk flat files — the raw PUMS person/household CSVs are also downloadable in bulk without a key if you'd rather skip this |
| IPUMS USA account | ipums.org | Free; alternative to raw Census PUMS flat files — gives a browser extract-builder and harmonized variable names, easier to work with for a one-off pilot pull. Extract requests typically process quickly, not instant |
| Socrata/Chicago Data Portal app token | data.cityofchicago.org (account settings) | Optional — only matters if pulling many datasets via the SODA API repeatedly; one-off direct downloads don't need it |
| NOAA CDO token | ncdc.noaa.gov/cdo-web/token | Instant via email. Needed for historical daily weather by date (vehicle-selection module context); NWS's no-key API only covers current/forecast, not historical |

## Tier C — needs an account now, cost sizing deferred

| Source | Signup | Notes |
|---|---|---|
| Google Places API | Google Cloud Console — requires a billing profile on file even for free-tier usage | Feeds POI descriptions (belief module) and popularity ground truth (evaluation-plan.md, stretch). Get the key now; don't size call volume yet |
| Yelp Fusion API | yelp.com/developers — free developer account | Same purpose as Google Places, often used as a cross-check/fallback for POI coverage gaps |

## Tier D — needs the study area decided first

- **ACS PUMS extract**, filtered to whichever PUMA(s) overlap the chosen community area(s).
- **LODES flows**, filtered/aggregated to census blocks within or commuting to the study area.
- **OSM street graph + POIs**, clipped from the Tier-A regional extract to the study-area polygon(s).
- **Google Places / Yelp POI pulls**, scoped to the study area (this is also where actual call volume,
  and therefore cost, gets sized — deferred until here on purpose).
- **CMAP travel-survey trips**, filtered to the relevant geography once the Tier-A dataset is in hand.

## The one real data-engineering task: PUMA ↔ Community Area crosswalk

No official crosswalk exists — PUMAs are Census-drawn and don't align with Chicago's community area
boundaries, and PUMAs are generally larger, so one PUMA can cover several community areas or a
fraction of one. Two ways to handle it, in order of how much precision the pilot actually needs:

1. **Pilot-pragmatic (recommended default):** identify whichever PUMA(s) overlap the chosen community
   area(s) via a spatial overlay (`geopandas.overlay` of the PUMA and community-area shapefiles from
   Tier A), and sample personas from that PUMA directly — accepting that the synthetic population
   represents a somewhat larger area than the strict community-area polygon. Document this as an
   accepted approximation (same spirit as the paper's own age-group aggregation approximations,
   `02-citysim-methodology-digest.md` §4.1). Fast, one spatial join, no extra weighting logic.
2. **More precise (only if the pilot later needs it):** population-weight the overlap using Census
   tract/block population counts within the PUMA∩community-area intersection, so sampled PUMS records
   are drawn proportionally to where people actually live inside the study area rather than uniformly
   across the whole PUMA. More faithful, meaningfully more engineering — not worth it for a first pilot
   pass per `01-vision-and-scope.md`'s "lightweight" framing.

## Suggested order of operations

1. Confirm the study area (Step 0) — the single highest-leverage decision, everything else in Tier D
   depends on it.
2. Set up the Python environment and knock out all of Tier B + Tier C signups in parallel — ~30-45 min
   total, none of it blocks on the study-area decision.
3. Pull Tier A data — most of it is study-area-independent and can start immediately; the OSM regional
   extract and boundary files are worth grabbing first since Step 4 needs them.
4. Build the PUMA↔Community Area crosswalk (pilot-pragmatic version).
5. Pull Tier D data now that the area and crosswalk exist.
6. Feed into the persona-sampling pipeline (`persona-initialization.md`) — this is `roadmap.md`
   Phase 2's exit criterion: a persisted population of personas with home/work anchors.

This maps directly onto `roadmap.md` Phases 1–2 — update that file's status section as steps complete.
