# ADR-011: Latency budget for visitor-facing AI

- **Status:** accepted
- **Date:** 2026-09-07
- **Serves:** Capability: C4 (guest companion)

## Context

Both competing submissions define latency NFRs for ticketing and analytics (e.g. ticketing p95
≤ 500ms) but neither defines one for the concierge/companion chat path — which is the AI
experience a visitor actually interacts with directly and notices being slow. An unstated
"should feel fast" isn't a design constraint; it's a hope.

## Decision

Define an explicit, hop-by-hop latency budget for the companion's request path — retrieval
(400ms), model inference (900ms, tiered routing cheap-first), guardrail/groundedness check
(300ms, run in parallel with partial rendering where possible), render (200ms) — summing to a
p95 target of ≤ 1.8s. Full table in [06-verification.md](../06-verification.md).

## Alternatives considered

| Option | Why not |
|---|---|
| No explicit budget, monitor "user-perceived speed" qualitatively after launch | Not actionable at design time — nobody knows which hop to optimize when the whole thing feels slow, and it's not testable in CI |
| Single end-to-end target only (e.g. "p95 ≤ 2s") with no per-hop breakdown | Better than nothing, but doesn't tell an engineer where the budget is being spent when the target is missed |
| **Explicit hop-by-hop budget summing to a target p95** | — chosen |

## Consequences

- Constrains model choice in the AI Gateway's tiered routing — the primary model tier must fit
  inside the 900ms inference budget, which rules out some larger/slower models as the default
  tier (they remain available as an escalation path for genuinely hard queries).
- Gives the verification loop a concrete, per-hop signal to alert on, rather than only an
  aggregate that doesn't say which layer regressed.

## How we will know this was right

Track p95 by hop in production; if the render or retrieval hops are consistently the bottleneck
rather than inference, that's a signal the budget split itself needs rebalancing, not that the
overall 1.8s target is wrong.
