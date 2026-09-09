# Cost & Payback

One competing submission has zero currency figures anywhere. The other has real numbers, but
derives its headline revenue figure from a ticket price its own document labels "Unspecified."
We'd rather show a smaller, honestly-labeled model than a bigger, unverifiable one — every
number below is illustrative and meant to be replaced with the Countess's real figures before a
euro is committed (see the assumptions table in
[01-business-case.md](01-business-case.md)).

## Hardware BOM (illustrative, Phase 1+2 combined)

| Item | Qty | Unit cost (illustrative) | Total |
|---|---|---|---|
| MQTT-capable enclosure sensors (feed scale, water chemistry, climate) | 55 | €250 | €13,750 |
| RGB/thermal edge cameras (welfare + ride monitoring) | 20 | €700 | €14,000 |
| Ride vibration/cycle sensors (retrofit, non-invasive) | 40 | €300 | €12,000 |
| Edge inference nodes (on-site GPU) | 4 | €1,200 | €4,800 |
| Local MQTT broker (HA pair) | 1 | €2,000 | €2,000 |
| Gate readers (offline-capable) | 8 | €600 | €4,800 |
| **Total hardware (one-time)** | | | **~€51,000** |

## Read scaling without CQRS

Rejecting CQRS/ES ([02-architecture.md](02-architecture.md)'s worksheet) doesn't mean every read
hits the same write path. The Countess's P&L dashboards, C2's batch forecast training reads, and
operational reporting all read from a **standard read replica** of the primary DB — ordinary
database replication, eventually consistent by seconds, not an event-sourced projection. This is
enough separation to stop analytics/ML batch reads from contending with real-time cashless writes
during peak attendance, without taking on CQRS/ES's operational overhead (projections, replay
tooling, schema evolution) for a read-scaling problem this size doesn't actually have yet. Revisit
if replica lag or write contention shows up as a real production signal, not pre-emptively.

This replica is the one stated exception to [ADR-001](adrs/ADR-001-modular-monolith.md)'s
schema-per-module rule: it reads across module schemas because it is a read-only reporting
consumer, not a module, and nothing downstream of it writes back or makes a transactional
decision off its results. It stays that — a dashboard and a training-data source, not a second
way for one module's code to read another module's tables.

## Cloud run-rate

| Item | Monthly (illustrative) |
|---|---|
| Core monolith hosting (compute + DB) | €800 |
| Telemetry ingest + storage | €400 |
| AI Gateway inference spend (tiered routing, all 5 capabilities) | €1,800 → scales toward €4,500 at Yr3 attendance |
| Observability / tracing | €300 |
| **Total (Yr1, blended)** | **~€3,300/mo → ~€40k/yr** |

At the illustrative €32M/yr baseline revenue from
[01-business-case.md](01-business-case.md), full AI run-rate at Yr3 scale (~€54k/yr cloud +
~€35k/yr build amortization across capabilities) stays well under 1% of revenue — the ceiling we
use as a sanity check, not a hard target.

## Cost per inference (illustrative)

A monthly line item ("€1,800 → €4,500") hides whether any single capability is quietly expensive
per use. Splitting that AI Gateway line by capability and rough volume:

| Capability | Illustrative cost per unit | Basis |
|---|---|---|
| C1 Welfare | ~€0.004 per anomaly-scoring event | Edge-first — most scoring runs on-site for free; only escalations reach the paid gateway tier |
| C2 Forecast | ~€1.50 per daily forecast run | Batch job, not called per visitor interaction |
| C3 Offers | ~€0.006 per contextual-offer decision | High volume, cheap classical (non-generative) model |
| C4 Companion | ~€0.09 per conversation | Highest per-unit cost of the five — LLM inference, tiered routing cheap-first |
| C5 Maintenance | ~€0.003 per anomaly-scoring reading | Edge-first, same pattern as C1 |

These are a back-of-envelope split of the AI Gateway line by relative capability volume and model
tier, not measured yet — flagged illustrative like every other figure in this file. The real
version is a one-line query away once there's production data: every inference already carries a
`cost` field by architectural rule ([ADR-010](adrs/ADR-010-every-inference-is-an-event.md)), so
this table becomes a query against that event stream on day one of Phase 2, not a new
instrumentation project.

## Payback per capability

| Capability | Cost (build + Yr1 run) | Effect (Yr1, illustrative) | Payback | Kill criterion |
|---|---|---|---|---|
| C1 Animal welfare | ~€175k | -15% of ~€2.2M vet/care cost ≈ €330k/yr | < 1 year | See [04-ai-capabilities/README.md](04-ai-capabilities/README.md) |
| C2 Visitor flow & staffing | ~€110k | Staffing cost avoided vs. linear scaling: at full Yr3 attendance, ~€2M/yr avoided | < 1 year at scale; ramps with attendance growth | See [04-ai-capabilities/README.md](04-ai-capabilities/README.md) |
| C3 On-site spend | ~€135k | +20-25% of €10 on-site spend × 1M visits/yr ≈ €2-2.5M/yr | < 1 year | See [04-ai-capabilities/README.md](04-ai-capabilities/README.md) |
| C4 Guest companion | ~€210k (highest run-rate — LLM inference) | Return-visit uplift, hard to isolate cleanly — smallest, least certain payback in this set | 1-2 years, contingent on return-visit data once live | See [04-ai-capabilities/README.md](04-ai-capabilities/README.md) |
| C5 Ride maintenance | ~€120k | -30% of ~€500k unplanned-repair cost ≈ €150k/yr | < 1 year | See [04-ai-capabilities/README.md](04-ai-capabilities/README.md) |

C4 is deliberately the least confident number here — we say so plainly rather than force-fitting
a payback figure to make five capabilities look uniformly justified. It ships in Phase 3 for
exactly that reason (see [09-roadmap.md](09-roadmap.md)) — funded by the payback the earlier
phases have already proven, not funded on faith.
