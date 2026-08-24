# Agent-Cognition LLM Cost Estimate & Model Choice

Informs the "Agent-cognition LLM" open question in `parameters.md` / `open-questions.md`. Covers
agent-cognition calls only (the ~35 LLM calls/agent/simulated-day driving persona/memory/beliefs/
needs/goals/planning/place/vehicle/social) — not the judge-LLM evaluation runs or embedding calls,
which are a separate, smaller line item.

## Call budget (assumptions)

Per agent per simulated day, assuming the perception/dispatcher pre-filter from
`modules/06-perception-dispatcher.md` is implemented (LLM fires at real decision points, not every
5-minute tick — without that filter, multiply everything below by ~10-15x):

| Call type | Calls/agent/day | Avg input tok | Avg output tok |
|---|---|---|---|
| Needs init (daily) | 1 | 400 | 100 |
| Mandatory + medium-priority planning | 2 | 750 | 375 |
| Value-driven leisure block (N=3 candidates, 1 call/block) | 3 | 700 | 350 |
| Perception/dispatch + outcome→needs update at activity transitions | 12 | 500 | 150 |
| Place selection (macro + micro) + vehicle selection, ~3.5 trips/day × 3 calls | 10.5 | 633 | 150 |
| Belief/observation update per POI visited | 3.5 | 500 | 200 |
| Social (partner select + message gen), population-averaged | 1.5 | 500 | 150 |
| End-of-day reflection synthesis | 1 | 1,200 | 400 |
| Long-term goal revision (monthly, amortized) | 0.03 | 900 | 300 |
| **Total** | **~35 calls** | **~20,400 in** | **~6,600 out** |

~27,000 tokens/agent/day. Scenarios use the pilot's own population/duration range from
`01-vision-and-scope.md`: Low = 200 agents × 7 days (1,400 agent-days), Mid = 350 × 10 (3,500),
High = 500 × 14 (7,000).

## Pricing checked (2026-08-24, from provider docs, not aggregator sites)

| Model | Tier | Input $/M | Output $/M | Source |
|---|---|---|---|---|
| Grok 4.3 | xAI "Economical" (<200k ctx) | $1.25 | $2.50 | docs.x.ai |
| Grok Build 0.1 | xAI "Lightweight" (<200k ctx) | $1.00 | $2.00 | docs.x.ai |
| *Grok 3 Mini* | *retired/aliased into 4.3 as of Aug 2026* | *n/a* | *n/a* | docs.x.ai (May 2026 retirement notice) |
| Llama 3.1 8B | DeepInfra | ~$0.05 | ~$0.08 | DeepInfra pricing (search-verified range $0.02–0.06/M) |
| Llama 3.3 70B | Groq (cheapest first-party) | $0.59 | $0.79 | search-verified |
| Llama 3.3 70B | Fireworks (flat) | $0.90 | $0.90 | search-verified |

Note: **Grok (xAI's model) and Groq (an inference-hardware host serving open-weight models) are
different companies** — easy to conflate given the name.

## Cost by scenario

| Scenario | Agent-days | Grok 4.3 | Grok Build 0.1 | OSS tiered (below) | OSS cheap-tier-only |
|---|---|---|---|---|---|
| Low (200×7) | 1,400 | ~$59 | ~$47 | **~$17** | ~$2 |
| Mid (350×10) | 3,500 | ~$147 | ~$118 | **~$44** | ~$5 |
| High (500×14) | 7,000 | ~$294 | ~$235 | **~$87** | ~$11 |

**OSS tiered** = route high-frequency/low-stakes calls (needs init, perception/dispatch, outcome
updates, vehicle selection, social) to a cheap 8B-class model, and route the calls the paper's own
ablation study (`02-citysim-methodology-digest.md` §4, Appendix C.5) and belief-backbone study
(Appendix C.4) show matter most for plausibility — planning, place selection, belief/observation
generation, reflection — to a stronger 70B-class model. **OSS cheap-tier-only** = everything on the
8B-class model; cheapest but carries the exact risk the paper's own backbone study flags (smaller
LLaMA-2-class models had markedly higher belief-estimation error than GPT-4o/GPT-4o-mini/Qwen-14B,
worst on Entertainment-category POIs) — not recommended as a default, useful mainly for cheap
pipeline shakeout runs.

## Recommendation

**Don't size the pilot by dollar cost — every option here is affordable (max ~$294 for the priciest
combination).** Money isn't the binding constraint; statistical/mechanistic adequacy of the results
is. That changes how to think about "Low would give minimal results":

- **Population** mostly buys statistical smoothness in the aggregate comparisons (time-use-by-age-
  group bars, travel-volume curves) and a richer social graph. More agents = less noisy charts.
- **Duration** mostly buys mechanism depth: at least 2 weekly cycles (~14 days) gives a real
  weekday-vs-weekend split instead of a single noisy 5-vs-2-day sample, and gives beliefs/reflective
  memory enough repeat visits to show non-trivial dynamics rather than staying near their neutral
  init values. A 7-day run barely exercises belief formation or reflection accumulation at all.

So the actual "Low gives minimal results" risk is concentrated in **duration**, not population or
dollars. Concrete plan:

1. **Shakeout run first, regardless of final scale**: ~20-30 agents × 2-3 days on the cheap OSS tier
   (cost: low single-digit dollars, effectively free) to catch pipeline bugs before committing to a
   real run. This is the actual reason to start cheap — not to save money on the real run, but to
   avoid burning a full-scale run on a broken pipeline.
2. **Real pilot run**: lean toward the fuller end of the range on both axes — something like
   **350-500 agents × 14 days** — rather than the Low bucket, since duration in particular is needed
   to satisfy the pilot's own success criteria (`01-vision-and-scope.md`: weekday/weekend travel
   pattern, ablation study quality). At OSS-tiered pricing this is ~$44-87 total, still trivial.
3. **Model routing**: use the OSS tiered strategy above rather than either extreme (all-Grok or
   all-cheap-OSS) — it's ~3.4x cheaper than Grok 4.3 while explicitly protecting the belief/planning
   calls the paper's own experiments show are quality-sensitive, and ~30x cheaper than Grok while
   being materially safer than cheap-tier-only.
4. Self-hosting (vLLM/Ollama on local or rented GPU) is a viable zero-marginal-cost alternative to
   hosted OSS APIs if GPU access exists, but adds real infra work (batched/concurrent serving for
   up to ~500 agents × 35 calls/day); at these dollar amounts, hosted APIs are simpler and the
   savings from self-hosting aren't worth the setup cost for a pilot.

This is a recommendation, not yet a locked decision — see `open-questions.md` for how to close it out.
