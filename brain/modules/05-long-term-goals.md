# Module: Long-Term Goals

## What the paper does

See digest §1.5. Monthly (or major-life-event-triggered) LLM-driven goal formation/revision, grounded
in Maslow's hierarchy, conditioned on persona + financial status + social contacts + recent activity +
current goals, plus three computed signals: need-fulfillment index, financial-stress flag
(`income < 0.9 × expenses`), social-isolation flag (<3 unique contacts/7 days), and an "interest"
signal (share of last-30-days-visited POIs with satisfaction belief > 0.5). Ablation shows this
module's benefit is localized to Dialogue and Mobility, and accrues over multiple days/months — a
short simulation window will under-show its value.

## Chicago pilot adaptation

- **Cadence mismatch with pilot duration:** the paper's "monthly" revision cadence barely fires within
  a 1–2 week pilot. Two options: (a) keep monthly cadence and accept that goal *revision* mostly won't
  be observed, only *initial* goal formation — cheapest, most faithful; (b) shorten the revision
  cadence for the pilot (e.g. weekly) purely to observe the mechanism working within a short run, and
  document this explicitly as a deliberate deviation from the paper, not a hidden one. Leaning (a) for
  the initial pilot with (b) as a fallback if goal-revision behavior specifically needs to be
  demonstrated — see `open-questions.md`.
- **Financial data:** income comes from the ACS-PUMS-sourced persona (`modules/01-persona.md`).
  Expenses aren't in PUMS directly; plan to approximate via a simple cost-of-living heuristic (e.g.
  BLS Consumer Expenditure Survey regional averages scaled by household size) rather than modeling
  real spending — flagged as an approximation, not measured behavior.
- **Everything else (signals, prompt structure) copied as-is** — these are cheap-to-compute, don't
  depend on scale or Japan-specific data.

## Open questions

- Monthly vs. shortened cadence for the pilot — see above and `open-questions.md`.
