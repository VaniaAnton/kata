# ADR-002: Edge-first store-and-forward over MQTT

- **Status:** accepted
- **Date:** 2026-09-07
- **Serves:** NFR: patchy Wi-Fi (from brief); Lever: cost-to-serve (via C1, C5 telemetry)

## Context

The brief states Wi-Fi coverage on the estate is patchy and that there's budget for
MQTT-capable hardware. Every sensor (enclosure monitors, ride telemetry, edge CV nodes) needs a
way to get data to the cloud that survives connectivity gaps without losing data or blocking
local operation.

## Decision

Every device publishes to a local, on-site MQTT broker (deployed as an HA pair), not directly to
the cloud. The broker persists messages to disk at QoS 1, sized for 72 hours of buffer, and
forwards to cloud Telemetry Ingest opportunistically whenever uplink is available. Cloud ingest
deduplicates by `(device_id, seq)` so a retried batch after a reconnect never double-counts.
Local consumers (safety-alert siren, gate device) read directly off the local broker and never
wait on a cloud round-trip.

## Alternatives considered

| Option | Why not |
|---|---|
| Direct device-to-cloud uplink, retry on failure | No local buffering means data loss during any outage longer than the retry window, and local consumers (safety alerts) would depend on cloud reachability, which the brief says is unreliable |
| Cloud-side buffering only (accept device drops, backfill later) | Devices themselves would need to buffer anyway to avoid data loss, which is exactly what a local broker does, but without the local-consumer benefit |
| **Local MQTT broker with store-and-forward** | — chosen |

## Consequences

- Requires on-site broker hardware and its own HA/monitoring, a small addition to operational
  surface, offset by being the thing that makes everything else in
  [03-edge-and-connectivity.md](../03-edge-and-connectivity.md) possible.
- 72h buffer is a deliberate choice, not unlimited — a multi-day outage beyond that would lose
  data, flagged as risk R7 in [10-risks.md](../10-risks.md).

## How we will know this was right

Track buffer utilization in production; if outages regularly approach the 72h ceiling, that's a
signal to either extend the buffer or invest in a secondary uplink (cellular), not a signal the
architecture is wrong.
