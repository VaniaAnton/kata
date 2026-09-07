# ADR-009: One credential for ticket + cashless spend

- **Status:** accepted
- **Date:** 2026-09-07
- **Serves:** Capability: C3 (on-site spend)

## Context

C3's premise is removing friction from on-site spend. A separate payment app or card, on top of
the entry ticket, re-introduces exactly the friction (another thing to carry, another login,
another thing that fails when Wi-Fi is patchy) the capability exists to remove.

## Decision

The same signed credential used for gate entry (physical wristband/card or app-held token) also
carries a cashless spend balance. Both ticket validation and spend authorization are verified
locally against a cached public key / balance, working fully offline; transactions queue for
cloud reconciliation when connectivity returns, consistent with the degraded ladder in
[03-edge-and-connectivity.md](../03-edge-and-connectivity.md).

## Alternatives considered

| Option | Why not |
|---|---|
| Separate app-only payment method alongside the physical ticket | Adds a second thing to carry/remember, and depends on the visitor's phone and connectivity — directly works against the patchy-Wi-Fi constraint and the friction C3 is trying to remove |
| **One credential for both ticket and cashless spend** | — chosen |

## Consequences

- Slightly more complex credential (carries both a ticket claim and a spend balance) than a pure
  ticket-only token, but avoids a second issuance/onboarding flow entirely.
- Loss of the physical credential loses both entry and spend access simultaneously — mitigated
  by the same signed-credential-reissue process either way would have needed.

## How we will know this was right

Track pre-order and cashless adoption rate; if adoption lags because visitors find checkout
friction elsewhere (not credential-related), that points to a different bottleneck in C3, not
this decision.
