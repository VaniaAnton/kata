# ADR-017: No lakehouse / feature store at launch — per-capability dataset ownership instead

- **Status:** accepted
- **Date:** 2026-09-09
- **Serves:** Judging criterion 4 (dealing with uncertainty in AI tech); Judging criterion 5 (AI characteristics matching architecture); extends [ADR-001](ADR-001-modular-monolith.md), [ADR-004](ADR-004-ai-gateway-provider-indirection.md), [ADR-016](ADR-016-edge-model-lifecycle-and-sensor-calibration.md)

## Context

[ADR-014](ADR-014-no-agent-platform-at-launch.md) gave the decision not to build a full
multi-step agent platform an honest trade-off record, with a stated ceiling and a quantified
revisit trigger. The decision not to build a lakehouse / feature store / semantic layer has been
sitting, by contrast, as one sentence in [README.md](../README.md) ("not from a components
list — event bus, edge gateway, lakehouse") and a paragraph about read replicas in
[08-cost-and-payback.md](../08-cost-and-payback.md) — the same kind of decision, held to a
lower standard of rigor. That double standard is a real gap: several claims elsewhere in this
repo quietly assume a governed data layer exists to back them —
[ADR-016](ADR-016-edge-model-lifecycle-and-sensor-calibration.md)'s golden-set leakage control
("explicitly excludes any animal or ride currently contributing training data") and
[06-verification.md](../06-verification.md)'s PSI drift check ("training/eval feature
distribution... vs. its live input distribution") both need *something* to version and compare
against. This ADR says what that something actually is, and why it isn't a lakehouse.

## Decision

No shared lakehouse (medallion architecture), no centralized feature store, no semantic layer at
launch. Instead:

- **Dataset ownership follows module ownership.** Each capability's training/eval data lives as
  a versioned snapshot owned by that capability's module schema
  ([ADR-001](ADR-001-modular-monolith.md)'s schema-per-module rule extends here) — a `golden_set`
  table plus a dated training-extract artifact, not a row in a shared warehouse.
- **The PSI reference distribution is a static artifact attached to the promoted bundle in the
  AI Gateway's registry** ([ADR-004](ADR-004-ai-gateway-provider-indirection.md)), snapshotted at
  the training run that produced that bundle — not computed live against a feature store's
  current state. Comparing live input distribution against *that specific bundle's* training-time
  snapshot is exactly what a PSI check needs; it doesn't need a queryable historical feature
  layer to do it.
- **Feature computation logic is a shared library, called at both training-snapshot time and
  serving time** — the training/serving skew a feature store would prevent architecturally is
  instead prevented by not having two implementations of the same feature logic to drift apart
  in the first place.
- **No cross-capability feature reuse.** If C5's ride-anomaly model and a future capability both
  want "recent vibration trend," each computes it from the shared telemetry event stream
  ([ADR-010](ADR-010-every-inference-is-an-event.md)) independently. This is the actual cost of
  this decision, not a hidden one.

## Alternatives considered

| Option | Why not |
|---|---|
| Full lakehouse (medallion: bronze/silver/gold) + feature store + semantic layer | Solves a feature-reuse-at-scale problem five capabilities on one small team don't have yet; requires data-platform skills and operational capacity ([07-operations.md](../07-operations.md)'s 3-5 engineers) this team doesn't have alongside everything else in [09-roadmap.md](../09-roadmap.md) |
| Shared feature store only, without the full lakehouse | Splits the difference badly — still a new piece of infrastructure to operate, version, and secure, without the full lakehouse's governance benefit, for the same absent reuse problem |
| Let each capability's team hand-roll its own pipeline with no shared convention at all | This is the ungoverned version of the chosen option — no schema-per-module discipline, no shared feature-logic library, so training/serving skew and undocumented golden-set provenance become real risks instead of managed ones |
| **Per-capability, module-owned dataset snapshots + registry-attached training artifacts + shared feature-computation library** | — chosen |

## Consequences

- No cross-capability feature reuse — a genuine ceiling, not a hidden one, matching the honesty
  standard [ADR-014](ADR-014-no-agent-platform-at-launch.md) already set for the equivalent
  agent-platform decision.
- No point-in-time-correct joins across capabilities' historical data — if a future capability
  needs "C1's welfare-anomaly rate as of the same moment as C3's spend data," that join has to be
  built by hand, not pulled from a governed history layer.
- Training/serving skew risk is mitigated by code-sharing discipline, not by infrastructure — a
  real risk if that discipline lapses, and one this ADR names explicitly rather than assuming
  away.
- Golden-set leakage control ([ADR-016](ADR-016-edge-model-lifecycle-and-sensor-calibration.md))
  and PSI drift checks ([06-verification.md](../06-verification.md)) both now have a concrete
  mechanism behind their claims instead of an implied one.

## How we will know this was right

Revisit if either signal shows up: (a) two capabilities are independently computing the same
derived feature from the same underlying telemetry — a concrete, observable duplication, not a
hypothetical one; or (b) the number of AI capabilities grows meaningfully past the current five
and per-capability dataset ownership starts requiring cross-capability coordination as a matter
of routine, not exception. Either is the trigger to invest in a real shared feature layer scoped
to the actual duplication observed — not to build a lakehouse pre-emptively on the belief that
scale is coming.
