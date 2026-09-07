# C5 — Ride Maintenance (advisory)

## Problem

40 rides, recently passed safety inspection, explicitly 18th-century in origin. Both competing
submissions noticed this is a natural predictive-maintenance case and both declined to design it
— one deferred it outright, the other gave it a single line. We think it's one of the clearer
cost-to-serve wins in the whole brief: unplanned downtime on a heritage ride is expensive (lost
throughput plus expedited, heritage-appropriate repair) and directly threatens the safety story
the brief foregrounds ("recently passed inspection... asbestos, broken glass and garden gnomes
were removed").

## Approach

Retrofit-only, non-invasive sensing: vibration, temperature, and cycle-count sensors added
externally to moving parts, feeding the same MQTT edge pipeline as everything else — no
modification to the historic structures themselves, which the heritage status rules out.

A model flags anomalous vibration/cycle patterns against each ride's own baseline (rides differ
too much mechanically for a shared model to be meaningful) and raises an inspection
recommendation with a confidence score and the specific anomalous signal attached, so an
inspector isn't handed a bare "something's wrong."

## Human clears the ride

The model **never** takes a ride out of service and never puts one back in service. It only ever
adds an inspection request to the maintenance queue. A qualified inspector always makes the
actual call — this mirrors the same deterministic-core principle used for animal welfare and
safety interlocks ([ADR-003](../adrs/ADR-003-deterministic-core.md)), applied here because a
false "all clear" on a ride is a safety incident, not a missed business opportunity.

Deterministic fallback: the existing fixed inspection schedule continues unchanged regardless —
this capability only ever adds unscheduled inspection requests on top, it never removes a
scheduled one.

## Serves

- **Lever:** ↓ cost-to-serve
- **ADRs:** [ADR-013](../adrs/ADR-013-advisory-predictive-maintenance.md), [ADR-003](../adrs/ADR-003-deterministic-core.md)
- **Kill criterion:** see [04-ai-capabilities/README.md](README.md)
