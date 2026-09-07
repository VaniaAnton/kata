# ADR-008: Individual animal identity

- **Status:** accepted
- **Date:** 2026-09-07
- **Serves:** Capability: C1 (animal welfare)

## Context

C1's whole premise is per-animal baselines — motion, feeding, posture, social behavior — across
200+ animals in 55 displays and enclosures, many of which house multiple animals. A per-animal
baseline is meaningless without a way to actually attribute a given camera frame or feed-scale
reading to a specific individual; we found this exact gap in a competing submission, whose
reference welfare scenario assumes per-animal data without ever explaining how an individual
animal is identified in a shared enclosure.

## Decision

Tag animals with PIT (passive integrated transponder) or RFID where species and handling
practice allow it, and combine that with enclosure-scoped attribution rules (e.g. per-animal feed
stations where the enclosure design allows it, or statistical disaggregation of shared feed-scale
deltas by observed individual activity, where it doesn't). Species where tagging isn't practical
or humane (e.g. very small or fragile animals, some aquatic species) get enclosure-level
monitoring with an explicit note that per-animal attribution isn't claimed for them — an honest
scope limitation rather than a fabricated per-animal number.

## Alternatives considered

| Option | Why not |
|---|---|
| Assume per-animal baselines work without an identity mechanism (as in a competing submission) | The core welfare-monitoring claim rests on an unsolved step; looks complete on a diagram, isn't buildable as designed |
| Full computer-vision re-identification (no physical tagging) | Higher technical risk and lower reliability for the species mix here than a physical tag, for no clear benefit |
| **PIT/RFID tagging + enclosure-scoped attribution, with honest scope limits where tagging isn't viable** | — chosen |

## Consequences

- Requires a tagging program and process for 200+ animals — a real, non-trivial operational
  task, not a purely digital decision.
- Species without practical tagging get a lower-fidelity (enclosure-level) monitoring tier,
  explicitly documented rather than silently assumed away.

## How we will know this was right

Track the fraction of the animal population with reliable per-animal attribution; if it's
materially below expectation for a species group, that's a signal to invest in a
species-appropriate identification method for that group specifically, not to lower C1's claims
across the board.
