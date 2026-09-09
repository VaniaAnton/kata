# ADR-016: Edge model lifecycle, golden-set methodology, and sensor calibration

- **Status:** accepted
- **Date:** 2026-09-09
- **Serves:** Judging criterion 6 (validation & verification); extends [ADR-004](ADR-004-ai-gateway-provider-indirection.md), [ADR-010](ADR-010-every-inference-is-an-event.md)

## Context

[06-verification.md](../06-verification.md) specifies real thresholds for cloud-side inference —
but every one of them is written as if the model runs in the cloud. C1's welfare cameras and
C5's ride sensors run inference **on-site, at the edge**
([02-architecture.md](../02-architecture.md) container view), and three questions go unanswered
for that tier specifically: how is an edge model version deployed and rolled back without the
gateway's cloud registry in the loop; what is the held-out "golden set" behind a claim like
"≥0.90 recall" actually made of and how is it kept honest over time; and how are the physical
sensors themselves (not the models reading them) kept calibrated, rather than only caught
after-the-fact by cross-confirmation ([R11](../10-risks.md), [R12](../10-risks.md)).

## Decision

**Edge model versioning.** Edge bundles are versioned entries in the same capability registry the
AI Gateway uses for cloud models ([ADR-004](ADR-004-ai-gateway-provider-indirection.md)) — a
`capability` like `welfare-anomaly-score-edge` resolves to a bundle just like its cloud
counterpart. New bundles are pushed opportunistically over the existing MQTT store-and-forward
channel ([ADR-002](ADR-002-edge-store-and-forward.md)). Versioning happens per **monitored
device** (a specific camera or ride-sensor group), not per GPU compute node — the 4 shared
inference nodes in [08-cost-and-payback.md](../08-cost-and-payback.md)'s BOM run whichever bundle
each device is currently assigned, so a device-level canary works regardless of how few physical
GPU nodes back it.

New bundles are staged as a **canary to at least 3 devices or 10% of the relevant device
population, whichever is greater** (so C1's ~20 cameras canary to at least 3, not a fractional
node) for a burn-in of **7 days or 200 scored local inferences per canary device, whichever is
longer** — deliberately longer in wall-clock time than the cloud Shadow window
([05-ai-platform.md](../05-ai-platform.md)), because edge devices see far fewer scoreable events
per day than cloud traffic does. Exactly like cloud Shadow, **a canary device's candidate bundle
is scored against live local input but its output never reaches a keeper, vet, or inspector, and
never fires an alert or feeds [ADR-006](ADR-006-confidence-bands-cost-of-error.md)'s
confidence-band actions** during burn-in — the rule-based/prior-bundle path stays authoritative
for that device the whole time. This is what keeps a not-yet-proven edge bundle from ever being
the thing a real welfare or safety decision was made on, honoring the same cost(FN) asymmetry
[ADR-006](ADR-006-confidence-bands-cost-of-error.md) already established for this domain — a
live A/B test on welfare-critical signals is exactly what this canary design avoids being.

Every edge device keeps its previous bundle on-device; a failed local fitness check (or loss of
contact with **Telemetry Ingest**, the already-extracted edge-facing service from
[ADR-001](ADR-001-modular-monolith.md), past a set window) triggers automatic revert to that
prior bundle without waiting for connectivity to the cloud registry. A canary window that closes
with zero scoreable anomalies on its devices is logged as **inconclusive, not passed** — it
extends automatically for one more window rather than silently counting "no signal" as "no
problem," given how rare the incidents this system exists to catch actually are.

**Golden-set methodology.** The held-out set behind every welfare/maintenance fitness threshold
is keeper-confirmed (a vet's actual diagnosis, an inspector's actual finding) — never
model-labeled — refreshed quarterly with a fixed minimum of new confirmed incidents per
species-group / ride-type, and explicitly excludes any animal or ride currently contributing
training data to avoid the set going stale by silent leakage as re-tagging and retraining happen
over time. This set is owned and versioned per capability, not in a shared feature store — see
[ADR-017](ADR-017-no-lakehouse-feature-store-at-launch.md) for why, and for what backs this claim
concretely.

**Sensor calibration.** Every MQTT-connected sensor (enclosure climate/water-chemistry, ride
vibration/cycle) is calibrated and certified at installation, then **recalibrated on a fixed
schedule** (climate/water sensors: monthly; ride vibration sensors: aligned with each heritage
ride's existing physical inspection cadence, piggybacking on a visit that already has to happen).
This is a scheduled, proactive check — distinct from and in addition to the reactive
cross-sensor-confirmation already in place for R11/R12, which catches drift *between*
calibration cycles, not instead of them.

## Alternatives considered

| Option | Why not |
|---|---|
| Push edge model updates the same way as cloud (all-or-nothing, no canary) | An edge bundle that fails silently on-device (different lighting/camera angle distribution than the training set) has no cloud eval gate to catch it before it's live on every node |
| Canary that acts/alerts live during burn-in (a real rollout, not shadow-equivalent) | Would put an unproven bundle's output directly into a welfare or safety decision during the exact window it's least trusted — violates the cost(FN) asymmetry [ADR-006](ADR-006-confidence-bands-cost-of-error.md) established for this domain |
| Golden-set curated once at launch, reused indefinitely | Goes stale exactly as the population and rides age and change — a "0.90 recall" claim measured against a 3-year-old golden set is not the same claim as one measured against a current one |
| Calibration purely reactive (cross-sensor confirmation only, as originally specified) | Catches a drifted sensor only once it disagrees with another signal — a slow, correlated drift across sensor types (e.g. a whole enclosure's climate control degrading gradually) can pass cross-confirmation for a long time before it's caught |
| **Canary edge rollout + on-device revert; quarterly keeper-confirmed golden set with leakage control; scheduled proactive calibration alongside reactive cross-confirmation** | — chosen |

## Consequences

- Edge nodes need enough local storage/compute to hold two bundle versions and run a local
  fitness check — a small addition to the edge hardware spec in
  [08-cost-and-payback.md](../08-cost-and-payback.md), not a new tier of infrastructure.
- Quarterly golden-set refresh is a standing keeper/inspector time cost, not a one-time project —
  the same trade-off [ADR-008](ADR-008-individual-animal-identity.md) already accepted for
  tagging: real operational work behind a claim that would otherwise be unverifiable.
- Scheduled sensor recalibration adds a maintenance-calendar line item; piggybacking ride-sensor
  recalibration onto existing heritage-inspection visits keeps this from being a second
  disruption to the historic structures.

## How we will know this was right

Track: edge canary-stage failure rate (a healthy sign if nonzero — it means the canary is doing
its job before a bad bundle reaches the full fleet); golden-set size and refresh cadence actually
held to quarterly, not silently skipped under delivery pressure; and whether any welfare/ride
incident is ever traced back to a sensor that was overdue for its scheduled recalibration — that
specific failure mode should approach zero given the schedule above.
