# C3 — On-site Spend

## Problem

Both competing submissions in this kata put "revenue per visitor" in their stated goals, and
neither of them has a single component for on-site spend anywhere in their architecture — no
POS, no F&B, no retail. That's the gap this capability fills, and it's arguably the single most
directly attributable revenue lever in the whole system: unlike attendance growth (which depends
on capacity and marketing beyond this architecture's control), a visitor's on-site spend is
something the software can influence in the moment.

## Approach

- **One credential for ticket and spend** ([ADR-009](../adrs/ADR-009-one-credential-ticket-and-spend.md)):
  the same signed token used for gate entry carries a cashless balance, so there's no second app
  or card to forget, and it works offline (validated locally, reconciled when connectivity
  returns).
- **F&B and retail pre-order**: browse and order from a queue or an enclosure, skip the line at
  pickup. Every theme-park operator's data shows queue abandonment is a major tax on food/retail
  revenue; removing the queue removes the tax directly, no AI required for this part.
- **Contextual offers (the AI part)**: a lightweight recommendation model suggests an offer based
  on time of day, weather, current location in the park, and remaining time before closing —
  e.g. a discounted drink bundle offered near closing when a family is already near the exit
  café. Kept deliberately simple (classical ML on tabular features, not generative) because the
  failure mode of a bad recommendation here is "ignored," not "harmful."

Deterministic fallback: cashless payment and pre-order work with zero personalization — the
static menu and standard pricing are always available; contextual offers are a pure upside layer
that can be switched off per [ADR-005](../adrs/ADR-005-ai-funding-gate.md) without breaking
anything else.

## Serves

- **Lever:** ↑ spend / guest
- **ADRs:** [ADR-009](../adrs/ADR-009-one-credential-ticket-and-spend.md), [ADR-007](../adrs/ADR-007-anonymous-visitor-analytics.md) (offers are cohort-based, not identity-based beyond the opt-in loyalty tier)
- **Kill criterion:** see [04-ai-capabilities/README.md](README.md)
