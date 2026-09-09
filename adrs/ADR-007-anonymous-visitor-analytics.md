# ADR-007: Anonymous-by-default visitor analytics

- **Status:** accepted
- **Date:** 2026-09-07
- **Serves:** Capability: C2 (visitor flow); Capability: C3 (on-site spend, cohort-based offers)

## Context

C2 needs footfall counts by area to forecast demand and staffing; C3 needs enough context to
make contextual offers relevant. Neither actually needs to know who a specific visitor is, and
identity tracking (face recognition, MAC-address tracking, cross-visit re-identification) carries
privacy risk and regulatory exposure disproportionate to the benefit for these two use cases.
Concretely, this is GDPR **Art. 5(1)(c) data minimisation** — don't collect identity data a
capability doesn't need — and **Art. 17 right to erasure** is far simpler to honor when the
default data shape has no identity to erase in the first place.

## Decision

Footfall and area-popularity counting is anonymous by construction — devices emit counts, not
identities, with no face recognition, no MAC-address tracking, and no re-identification across
visits. On-site spend offers (C3) are cohort/context-based (time of day, location, weather), not
identity-based, except for visitors who explicitly opt in to a loyalty tier for personalized
history-based offers.

## Alternatives considered

| Option | Why not |
|---|---|
| Face recognition or MAC-tracking for higher-fidelity per-visitor analytics | Materially higher privacy/regulatory risk for a benefit (marginally better forecasts/offers) that cohort-level data already captures adequately |
| **Anonymous counting by default, identity only on explicit opt-in** | — chosen |

## Consequences

- Some personalization quality is left on the table relative to full identity tracking — an
  acceptable trade given C2 and C3 don't need per-visitor identity to hit their targets in
  [04-ai-capabilities/README.md](../04-ai-capabilities/README.md).
- Opt-in loyalty data is the only place personal visitor data accumulates, simplifying the
  privacy/compliance surface to one well-defined path instead of every analytics pipeline —
  one erasure path to build and test against Art. 17, not one per analytics feature.

## How we will know this was right

If C3's offer-conversion targets aren't met with cohort-level data alone, that's a signal to
grow the opt-in loyalty tier's appeal (better rewards for opting in), not a signal to add
tracking without consent.
