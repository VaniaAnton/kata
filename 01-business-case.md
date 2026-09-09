# Business Case

The brief is explicit that the family's carnivorous-plant collection gets sold if the estate
doesn't work out financially. That makes this a P&L problem wearing a theme-park costume. Every
architectural choice in this repo is justified against one of three levers below — if a proposed
AI capability doesn't move one of them by more than it costs to run, it doesn't get built.

## Baseline (illustrative — calibrate against the Countess's real books before committing budget)

We don't have the estate's actual financials, so the numbers below are a transparent, labeled
model, not a claim of fact. Every figure has a source column so it can be swapped for real data
in week one of delivery without touching the reasoning.

| Metric | Value | Basis |
|---|---|---|
| Operating days/year | 200 | Assumption: seasonal operation (roughly March–November), closed in deep winter — consistent with animal welfare needs and heritage-building conservation |
| Current visitors/day (avg) | 5,000 | From brief |
| Current visits/year | 1,000,000 | 5,000 × 200 |
| Target visitors/day (Yr3, avg) | 15,000 | From brief |
| Target visits/year (Yr3) | 3,000,000 | 15,000 × 200 |
| Blended revenue/visit today | ~€32 (€22 ticket + €10 on-site) | Illustrative — replace with real admission price and POS data |
| Current annual revenue | ~€32M | 1,000,000 × €32 |
| Animal-care operating cost | ~€2.2M/yr | Illustrative, 55 enclosures — keepers, feed, vet, utilities |
| Heritage-ride maintenance cost | ~€1.6M/yr, of which ~€0.5M/yr is *unplanned* repair | Illustrative, 40 rides — ~60 unplanned-downtime events/yr at ~€8k average cost (lost throughput + expedited repair) |
| Park operations labor | ~€4M/yr | Illustrative, scaled for 5,000 visitors/day |

## Three levers

We deliberately do **not** claim AI triples attendance on its own — tripling attendance is
mostly a capacity, marketing, and pricing story that belongs to the Countess's business
strategy. AI's honest job is narrower and more defensible: **remove the operational
bottlenecks that would otherwise cap growth, and capture spend/cost improvements that are
directly attributable to software.**

| Lever | AI's actual contribution | Why AI, specifically | Target |
|---|---|---|---|
| ↑ Attendance | Capacity enabler: staffing that scales with demand (C2) and ride uptime that doesn't collapse under 3x load (C5), so growth isn't throttled by service quality | Without accurate demand forecasting, a 3x visitor increase either means 3x fixed staffing cost (unaffordable) or degraded experience (queues, understaffed enclosures) that caps return visits | Sustain <5% queue-abandonment rate through Yr3 attendance growth |
| ↑ Spend / guest | Directly attributable: on-site spend (C3) via cashless friction removal, F&B pre-order, and contextual offers | This is the lever both competing submissions left unbuilt — POS/F&B has no owner in either of their architectures despite being in their own stated goals | +20–25% uplift on the on-site component of spend/visit |
| ↓ Cost-to-serve | Directly attributable: fewer emergency vet callouts via earlier anomaly detection (C1), fewer unplanned ride-downtime events (C5), rosters sized to forecast rather than worst-case (C2) | These are the estate's largest controllable cost lines, and each has a "detect earlier / staff smarter" AI angle with a clean cost-of-error trade-off | -15% acute veterinary cost; -30% unplanned ride-downtime events; staffing cost scaling at ~2.5x rather than linear 3x at full attendance |

See [04-ai-capabilities/](04-ai-capabilities/README.md) for how each capability maps to these
targets, and [08-cost-and-payback.md](08-cost-and-payback.md) for the arithmetic.

## Assumptions (stated honestly)

We are not repeating the mistake we found in a competing submission, where a multi-million-euro
revenue figure was derived from a ticket price the same document labeled "Unspecified." Every
number above is flagged as illustrative, and the ones that matter most have an explicit
sensitivity note here.

| Assumption | Source / why we believe it | If we're wrong |
|---|---|---|
| 200 operating days/year | Typical for European heritage/animal attractions with winter closures | If the estate is open year-round, per-day cost targets in C2 need re-deriving, but the per-visit levers are unaffected |
| €32 blended revenue/visit | Placeholder — no real data available | This is the single most load-bearing number in this document; every payback figure in 08-cost-and-payback.md scales linearly with it. Replace first. |
| Attendance growth is capacity-limited, not demand-limited | Brief states hope/expectation of growth, implying demand exists; our job is not to lose it to poor service | If demand itself is the bottleneck (not capacity), the attendance lever shifts from C2/C5 to C4 (guest companion, return-visit driver) and marketing — architecture is unaffected either way since C4 already exists |
| Unplanned ride downtime costs ~€8k/event | Illustrative — lost throughput + expedited heritage-appropriate repair | If real cost is much higher (heritage rides can require specialist restoration), C5's payback improves; if much lower, C5 may not clear the funding gate (see ADR-005) |

## Not building (and why)

- **No autonomous vehicles / physical robots.** Neither the brief nor our three levers require
  one — it's a capability in search of a use case, and it was the single biggest capex item in a
  competing submission's cost model with no attached payback.
- **No agentic multi-step AI orchestration layer at launch.** A grounded LLM with tool access
  (C4) covers the guest-facing need; an orchestration platform is infrastructure for a scale of
  agent population we don't have yet. Revisit if C4 usage data shows demand for more autonomous
  multi-step tasks — full trade-off, including what we give up, in
  [ADR-014](adrs/ADR-014-no-agent-platform-at-launch.md).
- **No dynamic per-person pricing.** Family-bundle pricing (part of C2) stays cohort-level and
  policy-bounded — per-person pricing is a fairness and PR risk with no lever justification.
