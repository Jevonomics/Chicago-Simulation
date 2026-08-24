# Cost & Call-Volume Budget

This file exists for one purpose: run the calls/agent-day × agent-count × days budget check **before**
committing to a population/duration combination, not after. It's a companion to `cost-estimate.md`
(which compares Grok vs. OSS-hosted model options broadly and recommends OSS tiered routing) — this
file specifically works the numbers under a stated Grok-mini-tier target and a stated dollar ceiling,
since that's the concrete scenario that needs checking against the pilot's proposed scope before
anyone starts a real run. Where the two files' numbers differ slightly, see the reconciliation note at
the end — they're not in conflict, just different levels of granularity on the same gated design.

## Worked calls/agent-day estimate (resolved interpretation)

Per `02-citysim-methodology-digest.md`'s "Resolved ambiguity" note, the perception/dispatch gate means
the LLM fires at real decision points, not every 5-minute tick. Itemized:

| Call type | Calls/agent/day |
|---|---|
| Needs init (daily) | 1 |
| Mandatory + medium-priority planning | 2 |
| Value-driven leisure block planning (N=3 candidates, 1 call/block) | 3.5 |
| Perception-triggered dispatch + outcome/needs update at activity transitions | 15 |
| Trip-level place selection (macro + micro) + vehicle selection, ~4 trips/day × 3 calls | 12 |
| Belief/observation update per POI visited | 4 |
| Social interaction (partner select + message gen), population-averaged | 2 |
| End-of-day reflection synthesis | 1 |
| Long-term goal revision (monthly, amortized) | ~0.03 |
| **Total** | **~41 calls/agent/day** |

That lands in the ~40–60 calls/agent-day range the paper's own described mechanism implies once the
perception gate is applied correctly. `cost-estimate.md`'s earlier ~35/agent-day figure used slightly
coarser activity-transition counting under the same gated assumption — same order of magnitude, not a
contradiction (see reconciliation note below).

**Token estimate:** ~23,900 input / ~7,700 output tokens/agent/day (~31,600 total), scaled from
`cost-estimate.md`'s per-call-type token assumptions.

**Runtime check:** if a real implementation measures meaningfully above **~100 calls/agent-day**,
that's a signal the perception gate isn't suppressing per-tick calls as intended — investigate before
scaling population or duration further. This should be one of the first things checked in the small
test batch described in `01-vision-and-scope.md`'s pre-flight reliability check.

## Budget ceiling and target model for this check

- **Target LLM for this budget check: Grok, mini/economical tier** (Grok 4.3's "Economical" tier per
  `cost-estimate.md` — the closest currently-available equivalent to what used to be sold as
  "Grok 3 Mini," which is retired as of Aug 2026). $1.25/M input, $2.50/M output.
- **Pilot-phase budget ceiling: $100** of a **$2,500 total project budget**. This is a placeholder
  figure to confirm, not a measured constraint — flagged for confirmation in `open-questions.md`.

At ~31,600 tokens/agent-day: cost ≈ (23,900 × $1.25 + 7,700 × $2.50) / 1,000,000 ≈ **$0.049/agent-day**
on Grok's mini/economical tier.

## Budget check against the pilot's proposed scope — run *before* committing

$100 ÷ $0.049/agent-day ≈ **2,000 agent-days** affordable on Grok mini-tier alone.

| Population × duration | Agent-days | Cost on Grok mini-tier | Fits $100 ceiling? |
|---|---|---|---|
| 200 agents × 7 days (low end, `01-vision-and-scope.md`) | 1,400 | ~$69 | Yes |
| ~300 agents × 7 days (the success-criteria example in `01-vision-and-scope.md`) | 2,100 | ~$103 | Right at the edge |
| 350 agents × 10 days | 3,500 | ~$172 | No — 1.7x over |
| 500 agents × 14 days (high end, `01-vision-and-scope.md`) | 7,000 | ~$345 | No — 3.5x over |
| 350–500 agents × 14 days (the fuller-scope lean recommended in `parameters.md`, based on OSS pricing) | 4,900–7,000 | ~$240–$345 | No — 2.4-3.5x over |

**This is exactly the kind of finding this check is supposed to surface before a run starts, not
during one:** on Grok's mini/economical tier specifically, the $100 ceiling only comfortably covers
the low end of the pilot's proposed range (~200 agents × 7 days). The fuller-scope lean already
recorded in `parameters.md` (350–500 agents × 14 days) was costed against OSS tiered routing
(~$0.0124/agent-day blended, per `cost-estimate.md`), where the same ceiling covers ~8,065 agent-days
— comfortably including even the 500×14 case (~$87). **Model choice and scope are coupled: which one
you commit to constrains the other.** This file does not resolve that choice — it's restating and
sharpening the existing open item in `open-questions.md` (Agent-cognition LLM) and `parameters.md`,
now with the concrete dollar consequence attached, so it gets decided deliberately rather than
discovered mid-run.

## Reconciliation with `cost-estimate.md`

`cost-estimate.md`'s ~35-calls/agent-day figure and this file's ~41-calls/agent-day figure both assume
the same resolved perception-gate interpretation; they differ only in how finely activity-transition
and trip counts were itemized (12 vs. 15 dispatch calls, 3.5 vs. 4 trips/day). Both are estimates, not
measurements — the actual figure should be measured from the pre-flight test batch in
`01-vision-and-scope.md` and used to replace both once real data exists. Neither implies per-tick LLM
calls; both are an order of magnitude below the ~10-20x-larger literal-pseudocode reading that
`02-citysim-methodology-digest.md`'s "Resolved ambiguity" note rules out.
