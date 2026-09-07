# ADR-004: Capability-based AI gateway + provider indirection

- **Status:** accepted
- **Date:** 2026-09-07
- **Serves:** Judging criterion 4 (dealing with uncertainty in AI tech)

## Context

The judges explicitly ask how we'd handle the best model/provider changing, prices rising, or a
provider shutting down. AI capabilities are called from five different places in the monolith
([04-ai-capabilities/](../04-ai-capabilities/README.md)); wiring each one directly to a specific
provider's SDK means every one of those five capabilities has to be individually updated for
every provider or pricing change.

## Decision

All AI calls go through one extracted AI Gateway. Domain services request a **capability** (a
named business function like `welfare-anomaly-score`), never a model. The gateway resolves the
capability to a versioned bundle of `{model, prompt/config, eval suite}` via a registry, applies
tiered routing (cheap/fast model first, escalate on low confidence), enforces a per-capability
cost budget with 70%/90% alerts and automatic downgrade at 100%, and falls back through a named
secondary provider to a deterministic fallback if both model tiers are unavailable.

## Alternatives considered

| Option | Why not |
|---|---|
| Direct SDK calls to a chosen provider from each domain service | A provider or pricing change means auditing and changing five separate call sites instead of one registry entry; no single place to see or control total AI spend |
| Multi-cloud/multi-provider abstraction built into every domain service | Pushes the same indirection logic (routing, fallback, budget) into five places instead of one, with no operational benefit over centralizing it |
| **Single AI Gateway with capability-based indirection** | — chosen |

## Consequences

- One place to see total AI spend, one place a provider swap or price change is absorbed.
- Adds one network hop for every AI call — acceptable and budgeted for explicitly in
  [ADR-011](ADR-011-latency-budget-visitor-ai.md).
- The gateway itself becomes a single point of failure for all AI capabilities — mitigated by
  every capability having a deterministic fallback that doesn't depend on the gateway being up
  at all for its baseline behavior.

## How we will know this was right

Track: time-to-swap a provider (target: a registry change plus eval run, not a code change,
target ≤ 1 day); whether any cost overrun ever exceeded budget without triggering the 90% alert
first (should be zero).
