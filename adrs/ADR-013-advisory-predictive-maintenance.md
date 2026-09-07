# ADR-013: Advisory predictive maintenance on heritage rides

- **Status:** accepted
- **Date:** 2026-09-07
- **Serves:** Capability: C5 (ride maintenance)

## Context

40 rides, explicitly 18th-century and recently passed safety inspection after remediation work.
Both competing submissions identified predictive maintenance as a plausible AI use case for the
rides and both declined to design it — one deferred it, the other gave it a single line. We
think it's a clean cost-to-serve win (see [08-cost-and-payback.md](../08-cost-and-payback.md))
as long as it's built advisory-only, consistent with the deterministic-core principle in
[ADR-003](ADR-003-deterministic-core.md).

## Decision

Retrofit non-invasive sensors (vibration, temperature, cycle count) externally onto moving parts
— no modification to the historic structures, which their heritage status rules out — feeding the
same MQTT edge pipeline as every other telemetry source. A model flags anomalous patterns against
each ride's own baseline (rides differ mechanically too much for a shared model) and raises an
inspection recommendation with the specific anomalous signal attached. The model never takes a
ride out of service and never clears one back into service; a qualified inspector always makes
that call.

## Alternatives considered

| Option | Why not |
|---|---|
| Decline the use case entirely (as both competitors did) | Leaves a clear cost-to-serve win and a natural fit for the brief's heritage/safety framing unaddressed |
| Fully automated ride shutdown/clearance based on model confidence | A false "all clear" is a safety incident on 18th-century equipment — unacceptable risk for the cost-to-serve benefit gained; violates the deterministic-core principle in [ADR-003](ADR-003-deterministic-core.md) |
| **Advisory-only: model flags, human always clears** | — chosen |

## Consequences

- Caps the achievable cost reduction versus a fully automated system — an intentional ceiling,
  not an oversight, matching [ADR-003](ADR-003-deterministic-core.md)'s reasoning.
- Inspector trust in the flagging system is the main adoption risk; addressed by the precision
  threshold and fallback in [06-verification.md](../06-verification.md) and the kill criterion in
  [04-ai-capabilities/README.md](../04-ai-capabilities/README.md).

## How we will know this was right

Track unplanned-downtime events and inspector-reported false-alarm fatigue together; both need
to move in the right direction, not just downtime — a system that reduces downtime but burns out
inspector trust in the flags has failed its own kill criterion.
