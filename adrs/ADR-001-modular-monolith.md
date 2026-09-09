# ADR-001: Modular monolith with selective extraction

- **Status:** accepted
- **Date:** 2026-09-07
- **Serves:** Driving characteristic: operability by a small team; Lever: cost-to-serve (via lower operational overhead)

## Context

Both competing kata submissions decompose the estate into 11-13 independently deployed
services/quanta, each with its own database. That buys independent scalability and team
autonomy. This estate is run by a 3-5 person engineering team at 15,000 visitors/day — a real
but modest scale, not a hyperscale problem. Independent scalability per business domain is a
solution to a problem this team doesn't have; the problem this team does have is keeping 11-13
deployment pipelines, schemas, and on-call surfaces alive without a platform engineering
organization to do it.

## Decision

Ship one deployable core ("the monolith"), internally organized into modules along business-domain
seams (Ticketing & Access, On-site Spend, Animal Welfare, Visitor Flow, Guest Companion, Ride
Maintenance) with one shared database — but **not one shared schema**: each module owns its own
tables inside that database, namespaced per module, with no cross-module foreign keys. A module
that needs another module's data either calls it in-process (its actual API, not its tables
directly) or reads a projection fed by the outbox
([ADR-015](ADR-015-internal-outbox-idempotent-consumers.md)) — never a direct cross-schema join.
This is what makes "extract a module later" a schema migration plus a network boundary, not a
rewrite: the ownership boundary already exists at the data layer, one database instance is a
deployment convenience, not a design coupling. Modules communicate in-process; where a
cross-module effect needs to survive a module being temporarily degraded, it goes through an
internal outbox event log, not an external message broker — mechanics (transactional write,
at-least-once delivery, idempotent consumers) are specified in
[ADR-015](ADR-015-internal-outbox-idempotent-consumers.md).

Extract a component out of the monolith only when it has a genuinely different non-functional
profile the monolith can't satisfy. At launch, exactly two things qualify: the **AI Gateway**
(different deployment cadence, provider-latency-bound scaling) and **Telemetry Ingest** (must
survive the estate's connectivity outages independently of the core app's uptime).

## Alternatives considered

| Option | Why not |
|---|---|
| Microservices / architectural quanta per business domain (both competitors' choice) | Buys independent scaling and team autonomy this team doesn't need; costs 11-13 deployment pipelines, 11-13 places a cross-domain saga can partially fail, and an operational model that assumes a platform team |
| Single undifferentiated monolith with no internal module boundaries | Cheaper to start, but makes it hard to later extract anything (e.g. the AI Gateway) cleanly if churn concentrates there, and makes ownership fuzzy even within a small team |
| **Modular monolith + selective extraction** | — chosen |

## Consequences

- One deployment pipeline, one on-call rotation, one schema migration process — matches the
  team's actual size (see [07-operations.md](../07-operations.md)).
- Independent scaling of, say, C1 welfare processing vs. C3 spend processing isn't possible
  without a later extraction — acceptable, since neither is anywhere near a scale where that
  matters at 15,000 visitors/day.
- The AI Gateway and Telemetry Ingest are extracted for independent deploy cadence and scaling
  profile, **not** independent failure domain — both still depend on the same primary DB
  instance today; see [ADR-004](ADR-004-ai-gateway-provider-indirection.md) for the specific
  mitigation and the honest limit of that mitigation.
- If the estate's growth trajectory continues well past the 3-year, 15,000/day horizon, this
  decision should be revisited — see "How we will know this was right."

## How we will know this was right

Revisit if: (a) attendance growth continues meaningfully past 15,000/day and a specific module
starts dominating compute/ops load disproportionately, or (b) the team grows past ~8 engineers
and module ownership boundaries start causing real coordination friction. Either signal is a
prompt to extract that specific module, not to re-decompose the whole system preemptively.
