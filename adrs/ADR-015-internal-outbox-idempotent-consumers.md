# ADR-015: Transactional outbox + idempotent consumers for cross-module effects

- **Status:** accepted
- **Date:** 2026-09-08
- **Serves:** Judging criterion 5 (AI/architecture characteristics alignment); extends [ADR-001](ADR-001-modular-monolith.md)

## Context

[ADR-001](ADR-001-modular-monolith.md) states that cross-module effects needing to survive a
module being temporarily degraded "go through an internal event log (outbox pattern), not a
message broker" — but doesn't specify the mechanics. Left unspecified, "outbox pattern" is a
name, not a guarantee. Concretely: a welfare alert (C1) that also needs to notify staffing (C2)
cannot be allowed to succeed in one module and silently vanish before reaching the other because
a process crashed between two separate writes.

## Decision

Every cross-module side-effect that isn't a synchronous in-process call is written to an outbox
table in the **same database transaction** as the domain write that caused it — so the effect
either commits with its cause or not at all, never separately. A single in-process relay process
polls unpublished outbox rows in insertion order and dispatches them to subscribing modules via
an in-process event dispatcher (not a broker — [ADR-001](ADR-001-modular-monolith.md) already
rejected broker overhead for calls inside one deployable), with **at-least-once** delivery.

Every consumer is written to be idempotent, keyed on `(aggregate_id, event_id)`: re-applying the
same event twice must be a safe no-op. This mirrors the `(device_id, seq)` dedup already used at
the edge ingest boundary ([ADR-002](ADR-002-edge-store-and-forward.md)) — the same discipline,
applied one layer up. Ordering is guaranteed only per-aggregate; no consumer here needs ordering
across independent aggregates (a C1 welfare event never needs to interleave, in strict order,
with a C3 spend event).

The relay persists its own read cursor (last-processed outbox row id) **transactionally**,
committed alongside marking the row dispatched — so a relay crash mid-poll resumes from the last
committed cursor on restart rather than re-scanning from the beginning or losing its place; any
row it happens to redeliver on resume is exactly the at-least-once case idempotent consumers
already handle, not a new failure mode. Published rows are retained for 30 days (covering audit
and incident-debugging needs) then purged by a scheduled job — the table is not allowed to grow
unbounded at 15,000 visitors/day of event volume.

## Alternatives considered

| Option | Why not |
|---|---|
| Direct synchronous in-process calls only, no event log | A request touching three modules either fails atomically (defeats the point of having module boundaries at all) or partially fails silently, with no record of what succeeded — exactly the failure mode an outbox exists to remove |
| Dual write: write the domain row, then separately publish an event | The classic dual-write bug — a crash between the two writes silently drops the event. The welfare-alert and cashless-spend paths cannot tolerate a silently dropped event |
| External broker even for in-process cross-module calls | [ADR-001](ADR-001-modular-monolith.md) already rejected broker operational overhead here; nothing about idempotency changes that trade-off |
| **Transactional outbox, at-least-once, idempotent consumers keyed by `(aggregate_id, event_id)`** | — chosen |

## Consequences

- Adds one outbox table and one relay process to operate — a small addition, still in-process,
  no new deployable — in exchange for removing an entire class of "module A wrote the row,
  module B never found out" bugs.
- Every new cross-module integration must be written idempotently from day one — a standing
  discipline cost, but far cheaper than debugging a silently dropped event in production.
- At-least-once means occasional duplicate delivery is expected and normal. Duplicate-processing
  rate should be a monitored health signal, not an alert-worthy error — a rate of exactly zero
  for months is itself a signal the retry path has never actually been exercised.

## How we will know this was right

Track: outbox-to-relay lag (should stay in the low seconds under normal load); duplicate-event
rate per consumer (expected low but nonzero); and zero production incidents traced to a
cross-module effect that was written but never delivered.
