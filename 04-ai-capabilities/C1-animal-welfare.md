# C1 — Animal Welfare Monitoring

## Problem

200+ animals across 55 displays & enclosures, including aquatic, land-based, and venomous
species. Keepers can't watch every enclosure continuously; by the time a sick animal is visibly
distressed to a human observer, veterinary cost and welfare harm are both already higher than if
it had been caught early. The brief specifically calls out tracking health, eating
patterns/volume, and — separately — the jumping piranha population count.

## Approach

Per-animal (not per-enclosure) baselines, built from:

- **Edge computer vision**: activity level, feeding behavior, posture, social distance from
  RGB/thermal cameras — inference runs on-site, only derived features and short flagged clips
  leave the estate, not raw video (bandwidth and privacy both benefit).
- **Enclosure sensors**: feed-scale deltas, water chemistry (pH, temperature, turbidity for
  aquatic species), climate.
- **Individual identity** ([ADR-008](../adrs/ADR-008-individual-animal-identity.md)): PIT/RFID
  tagging plus enclosure-scoped attribution, so a shared feed scale or a multi-animal enclosure
  doesn't collapse into an unusable enclosure-wide average. This is a gap we found in a
  competing submission — their per-animal baselines had no mechanism to actually identify which
  animal is which in a shared display.

Piranha population counting is a narrower, separate pipeline: detect→track→count across 2-3
camera angles (including one above the waterline for jumpers), reported as a daily estimate with
a confidence interval, not a bare number — validated monthly against a manual keeper count.

Deterministic fallback: keeper-performed daily visual rounds continue regardless — this
capability adds an earlier-warning layer, it never replaces the keeper.

## Confidence bands priced by cost-of-error

Rather than picking thresholds like "0.9 = auto-act" out of habit, we derive them from the
actual monetary and welfare asymmetry: a false positive costs a keeper a wasted enclosure check
(~€50-100 of time); a false negative risks a treatable condition progressing to an emergency vet
callout (~€800-2,000) or animal loss (unbounded, both financially and reputationally). That
asymmetry — roughly 15-25:1 — is what sets the sensitivity of the alert threshold, not
intuition. Full reasoning in [ADR-006](../adrs/ADR-006-confidence-bands-cost-of-error.md).

| Band | Action |
|---|---|
| High confidence anomaly | Immediate keeper/vet notification, logged as a welfare event |
| Medium confidence | Added to next scheduled keeper round with a flagged clip attached |
| Low confidence | Logged only, sampled weekly for false-negative audit |

## Serves

- **Lever:** ↓ cost-to-serve
- **ADRs:** [ADR-006](../adrs/ADR-006-confidence-bands-cost-of-error.md), [ADR-008](../adrs/ADR-008-individual-animal-identity.md), [ADR-003](../adrs/ADR-003-deterministic-core.md)
- **Kill criterion:** see [04-ai-capabilities/README.md](README.md)
