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
