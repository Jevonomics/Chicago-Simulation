# Module: Social

## What the paper does

See digest §3 and Appendix A.1.2. Weighted social network; each directed edge `b_{u,v} ∈ R^3` over
`{affinity, trust, familiarity}`, initialized from demographic similarity + relationships, updated
continuously from interaction sentiment (positive/neutral/negative). **Face-to-face:** co-located
agents; partner chosen with probability `p_v = b_{u,v} / Σ b_{u,v'}`; capped at one partner per
30-minute tick. **Online:** triggered when social-need satisfaction drops below threshold; agent
proactively picks whom to contact and modality (face-to-face vs. online) via a single LLM call.
Message generation reflects personality, past interactions, and persona-derived topic constraints.

## Chicago pilot adaptation

- **Initial network seeding is the hard part at pilot scale.** The paper says beliefs init from
  "demographic similarity and relationships" but doesn't fully specify the graph structure (how many
  contacts per agent, how household/coworker/neighbor relationships are established). For the pilot,
  plan to seed a small number of structural relationship types explicitly rather than leaving it
  implicit: household members (from the ACS-PUMS household composition already in the persona),
  coworkers (agents sharing a work-anchor POI or same workplace category/location), and a handful of
  proximity-based "neighbor" ties (agents with nearby home anchors) — then layer affinity/trust/
  familiarity beliefs on top of that structural graph, closer in spirit to the paper's "demographic
  similarity and relationships" than a fully random graph would be.
- **Scale is favorable here:** at ~300 agents, a synthetic social graph with household + coworker +
  neighbor ties is easy to construct and fully inspectable — a good pilot advantage over the paper's
  1,000-agent setting where relationship structure is presumably harder to hand-verify.
- **Everything else (interaction mechanics, belief update, message generation) copied as-is** — no
  Chicago-specific change needed to the mechanics themselves.

## Open questions

- Exact rule for how many initial contacts/relationship ties to seed per agent, and how strongly to
  weight household vs. coworker vs. neighbor ties in the initial belief values — needs a first-pass
  default before implementation starts.
