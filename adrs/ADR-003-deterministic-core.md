# ADR-003: Deterministic core, AI as advisory overlay

- **Status:** accepted
- **Date:** 2026-09-07
- **Serves:** Judging criterion 5 (AI characteristics matching existing architecture); Risk: safety-critical paths (R4, R5 in [10-risks.md](../10-risks.md))

## Context

Several capabilities touch genuinely safety-critical paths: ride safety interlocks, gate entry,
animal welfare alerting, ride maintenance clearance. AI models are probabilistic; a wrong
decision on any of these paths is a safety incident, not a missed optimization. We need one
consistent rule for where AI is allowed to act versus only advise.

## Decision

Safety- and access-critical decisions are always made by deterministic rules, never by a model
directly:

- Ride safety interlocks: hardware/firmware rules, never touch a model.
- Gate entry: signature verification against a public key, never a model.
- Welfare alert triggering: rule-based thresholds fire the immediate local alert; AI adds a
  richer, slower anomaly-scoring layer on top for earlier/subtler detection, but never replaces
  the rule-based trigger.
- Ride maintenance clearance: a model can flag a ride for inspection; only a qualified human ever
  clears it back into service ([ADR-013](ADR-013-advisory-predictive-maintenance.md)).
- Roster generation: the forecast is ML, but the roster itself is produced by a deterministic
  constraint solver that a forecast can never override on minimum safety staffing.

AI is additive everywhere on this list: it can raise more alerts, catch things earlier, or
suggest better inputs — it cannot lower the safety floor the deterministic rule already
guarantees.

## Alternatives considered

| Option | Why not |
|---|---|
| Let models act directly on safety-critical paths where they're confident enough | Removes a human/deterministic check from exactly the paths where a false "all clear" is most expensive; also makes the system's safety properties dependent on model behavior, which is the least stable part of the stack |
| **Deterministic core, AI strictly advisory on safety paths** | — chosen |

## Consequences

- Safety-critical paths are fully testable like conventional software — no non-determinism to
  account for in their verification.
- AI's contribution on these paths is capped at "detect earlier / flag more," never "decide" —
  an intentional ceiling, not a limitation we're trying to work around.

## How we will know this was right

This is not a metric to optimize — it's a standing constraint. Revisit only if a specific
capability's advisory-only design is shown to systematically under-detect something a direct
model decision would have caught, and even then the fix is tightening the rule threshold, not
handing the decision to the model.
