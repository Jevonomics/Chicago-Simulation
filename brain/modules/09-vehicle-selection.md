# Module: Vehicle Selection

## What the paper does

See digest §2.3. Given distance, time of day, month, weather, temperature, persona, and available
vehicle set `V = {walk, bicycle, car, bus, train}`, a structured LLM prompt selects a vehicle plus a
brief justification (stored in reflective memory), approximating utility maximization purely through
LLM reasoning — no explicit learned utility function.

## Chicago pilot adaptation

- **Vehicle set:** keep `{walk, bicycle, car, bus, train}` — all five map directly onto real Chicago
  options (CTA bus, CTA 'L' train, Divvy/personal bike, walk, personal car/rideshare-as-car-proxy).
  Whether to add "rideshare" as a distinct sixth option (very salient in Chicago, less so in the
  paper's Tokyo setting) is an open question — leaning toward folding it into "car" for pilot
  simplicity rather than adding a mode the paper doesn't have, but worth revisiting.
- **Weather/temperature inputs:** the paper uses simulated weather/temperature by month; for a
  pilot bounded to real historical or near-term dates, actual **NWS/NOAA Chicago weather data** for the
  simulated date range can be used instead of a synthetic weather model — strictly more realistic and
  free to obtain, no reason not to use real data here.
- **Transit realism:** vehicle choice should be constrained by what's actually reachable — e.g. "train"
  shouldn't be selectable for an OD pair with no CTA rail access nearby. Plan to feed the LLM real
  CTA GTFS stop proximity (bus stop / 'L' station within walking distance of origin and destination) as
  part of the prompt context so choices stay grounded, rather than letting the LLM hallucinate transit
  availability.

## Open questions

- Rideshare as a distinct mode — see above.
- How to source per-timestep weather/temperature cleanly (NOAA API vs. a pre-pulled table for the
  simulated date range) — infrastructure detail, not a methodology question.
