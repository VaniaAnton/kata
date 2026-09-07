# ADR-010: Every AI inference is an event

- **Status:** accepted
- **Date:** 2026-09-07
- **Serves:** Judging criterion 6 (validation & verification)

## Context

A competing submission explicitly admits it has zero ADRs on observability — a real gap for a
system whose main production risk (an AI capability silently degrading) is exactly the kind of
failure that doesn't show up as an outage or an error, only as slowly-wrong numbers. Observability
for AI-specific behavior needs to be designed in, not bolted on as generic APM after the fact.

## Decision

Every AI inference call, regardless of which capability or which module makes it, emits a
structured event through the AI Gateway carrying `capability`, `model_version`, `cost`,
`confidence`, and per-hop latency. These events feed both real-time operational dashboards
([07-operations.md](../07-operations.md)) and the verification loop
([06-verification.md](../06-verification.md)) that joins predictions against delayed ground
truth to compute real production precision/recall, not just offline eval numbers.

## Alternatives considered

| Option | Why not |
|---|---|
| Generic request tracing (treat AI calls like any other API call) | Misses the fields that actually matter for catching AI-specific degradation — confidence and model version aren't things generic APM tracing captures by default |
| Observability added per-capability, as each team gets to it | This is effectively what both competing submissions have — inconsistent or entirely absent, and the gap that we're specifically avoiding here |
| **Every inference is a structured event by architectural rule, enforced at the gateway** | — chosen |

## Consequences

- The AI Gateway becomes the natural enforcement point for this rule — since all AI calls
  already route through it (per [ADR-004](ADR-004-ai-gateway-provider-indirection.md)), there's
  no capability that can accidentally skip instrumentation.
- Slightly more data volume than plain request tracing — acceptable and budgeted within the
  observability cost line in [08-cost-and-payback.md](../08-cost-and-payback.md).

## How we will know this was right

Every automatic rollback in production (see [06-verification.md](../06-verification.md)) should
be traceable back to a specific inference-event signal, not discovered first through a user
complaint or a manual audit.
