# ADR-005: AI funding gate

- **Status:** accepted
- **Date:** 2026-09-07
- **Serves:** Judging criteria 2 (suitability) and 4 (uncertainty); Lever: all three (this is the mechanism that enforces lever-alignment)

## Context

It's easy for an architecture exercise to accumulate AI capabilities because they're interesting,
not because they pay for themselves. Both competing submissions include capabilities (an
autonomous shuttle, a full agentic orchestration layer, a social-marketing pipeline) with no
attached cost-benefit reasoning at all. We wanted a mechanism that forces the opposite by
construction.

## Decision

No AI capability is built without, up front: (a) an estimated cost (build + Yr1 run,
[08-cost-and-payback.md](../08-cost-and-payback.md)), (b) an estimated effect tied to one of the
three levers in [01-business-case.md](../01-business-case.md)), and (c) an explicit **kill
criterion** — the measurable condition under which the capability gets turned off or reverted to
its deterministic fallback. Kill criteria aren't a launch-time formality; they're re-checked
continuously in production against the fitness functions in
[06-verification.md](../06-verification.md), not just assessed once before build.

## Alternatives considered

| Option | Why not |
|---|---|
| Build all five identified capabilities up front, evaluate later | This is what both competing submissions effectively do — several of their capabilities (autonomous shuttle, social-marketing AI) have no stated payback at all, and by the time a capability is live and staffed, "kill it" becomes organizationally much harder than "don't build it yet" |
| Central AI steering committee approves capabilities case-by-case | Adds a bureaucratic layer this small team doesn't have people for; the funding gate achieves the same discipline as a lightweight, repeatable check instead of a standing committee |
| **Funding gate: payback + kill criterion required before build, re-checked continuously after** | — chosen |

## Consequences

- Some plausible AI use cases (autonomous shuttle, agentic orchestration, social-marketing AI —
  see "Not building" in [01-business-case.md](../01-business-case.md)) are explicitly out of
  scope, not because they're bad ideas, but because no lever justifies their cost at this scale.
- C4 (guest companion), the capability with the least certain payback, is sequenced last in
  [09-roadmap.md](../09-roadmap.md) specifically so the gate's discipline shows up in delivery
  order, not just in prose.

## How we will know this was right

Every capability's kill criterion should actually be exercised at least once in a two-year
window — either triggering a real revert, or being reviewed and consciously kept despite being
close to the line. A funding gate that never actually kills anything is decoration, not a
mechanism.
