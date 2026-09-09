# Operations

Neither competing submission has much here — one submission's authors flagged their own
observability gap explicitly, the other has no deployment/DR view at all. For a system whose main
production risk is *silent* AI degradation, that's the wrong thing to skip.

## Observability

Every AI inference is a traced event by architectural decision, not convention
([ADR-010](adrs/ADR-010-every-inference-is-an-event.md)): `capability`, `model_version`, `cost`,
`confidence`, and latency-per-hop are captured on every call, feeding both the verification loop
in [06-verification.md](06-verification.md) and standard operational dashboards. The rest of the
monolith gets conventional request tracing — nothing exotic needed for a system this size.

## SLO & error budget

| Service area | SLO | Owner |
|---|---|---|
| Gate entry (offline-capable) | 99.95% successful validation | Ops lead |
| Cashless spend | 99.9% availability (degrades to queued/offline, never hard-fails) | Ops lead |
| Concierge chat | p95 ≤ 1.8s (see [06-verification.md](06-verification.md)), 99% availability | AI platform owner |
| Welfare alert delivery | ≤ 60s from detection to keeper notification | AI platform owner |

Error budget burn on any AI-specific SLO (not the deterministic core) is reviewed against the
funding gate in [ADR-005](adrs/ADR-005-ai-funding-gate.md) — a capability that keeps burning its
budget is a candidate for the kill criterion, not just an incident to patch repeatedly.

## On-call for a small team

Sized for the team that actually exists here (3-5 engineers), not a platform organization:

- **One rotation**, covering the whole monolith plus the two extracted services — not
  per-service on-call, which would need more engineers than the team has.
- **The deterministic core (gate entry, safety interlocks) pages immediately**; AI-capability
  degradation (a model falling back to its deterministic path) is a next-business-day ticket, not
  a page, because every capability already has a working fallback by design.
- **Quarterly game day**, rotating through four drills so every major failure mode this
  architecture claims to handle actually gets exercised at least once a year, not just designed
  for on paper:

  | Quarter | Drill | Risk exercised |
  |---|---|---|
  | Q1 | Kill the AI Gateway's primary provider; confirm every capability degrades to its stated fallback cleanly | R1 |
  | Q2 | Sever the estate's uplink and hold it past 24h (short of the 72h buffer ceiling); confirm local consumers (siren, gate, cashless) never notice | R7 |
  | Q3 | Force the outbox relay to redeliver a batch of already-processed events; confirm every consumer's duplicate-processing counter moves and no real-world effect double-fires | R13 |
  | Q4 | Feed one enclosure's climate sensor an implausible reading (frozen or drifted value) while its RGB/behavior signal stays normal; confirm cross-confirmation stops the welfare alert from firing on the single bad sensor alone | R11, R12 |

  A drill that has never actually been run is a design claim, not a verified property —
  [ADR-005](adrs/ADR-005-ai-funding-gate.md)'s "a funding gate that never kills anything is
  decoration" applies here too.

## Disaster recovery

| Tier | RTO | RPO | Notes |
|---|---|---|---|
| Edge (estate) | N/A — designed to run standalone | 0 (local store-and-forward buffers 72h) | See [03-edge-and-connectivity.md](03-edge-and-connectivity.md) |
| Cloud monolith | 4h | 15 min (standard DB backup cadence) | Single region at this scale; multi-region is a Phase 3+ conversation if attendance growth justifies it |
| AI Gateway | 1h (fails over to deterministic fallback immediately, full recovery target 1h) | N/A (stateless besides registry, which is backed up with the DB) | |
