# ADR-006: Confidence bands priced by cost-of-error

- **Status:** accepted
- **Date:** 2026-09-07
- **Serves:** Capability: C1 (animal welfare); Risk: R4 (alert fatigue) in [10-risks.md](../10-risks.md)

## Context

Human-in-the-loop confidence thresholds ("high confidence = auto-act, medium = review, low =
discard") appear in both competing submissions and in most AI system designs generally — but
almost always with the specific numeric cutoffs (0.7, 0.9) presented as given, with no stated
reasoning for why those particular numbers. That's a threshold chosen by convention, not
argument, and it's indefensible if a judge asks "why 0.9 and not 0.8."

## Decision

Derive confidence-band cutoffs from the actual monetary and welfare cost asymmetry between a
false positive and a false negative for each capability, rather than picking round numbers. For
C1 specifically: a false positive costs a keeper a wasted enclosure check (~€50-100); a false
negative risks an emergency vet callout (~€800-2,000) or worse. That ~15-25:1 asymmetry is the
input to setting the sensitivity of the alert threshold — the more expensive false negatives are
relative to false positives, the more the threshold shifts toward over-alerting rather than
under-alerting.

## Alternatives considered

| Option | Why not |
|---|---|
| Fixed, convention-based thresholds (e.g. 0.7/0.9) applied uniformly | Indefensible under questioning ("why 0.9?"), and wrong by construction whenever a capability's error costs are actually asymmetric, which they usually are |
| Let each capability owner set thresholds by intuition/experience | Better than nothing, but not repeatable, auditable, or comparable across capabilities |
| **Thresholds derived from cost(FP) vs. cost(FN)** | — chosen |

## Consequences

- Requires estimating cost(FP) and cost(FN) explicitly for each capability that uses this
  pattern — a small extra step at design time, in exchange for a threshold that survives being
  questioned.
- The weekly false-negative audit on discarded low-confidence events (see
  [C1](../04-ai-capabilities/C1-animal-welfare.md)) exists specifically to catch cases where the
  cost estimates themselves were wrong, and feed back into recalibrating the threshold.

## How we will know this was right

Track actual keeper-reported alert fatigue (subjective, but real) against the false-positive
rate the threshold implies; if fatigue is high despite the "correct" threshold, the cost(FP)
estimate was probably too low and needs revisiting, not the mechanism itself.
