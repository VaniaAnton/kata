# C4 — Guest Companion

## Problem

A grounded, chat-based companion helps visitors navigate a 40-ride, 55-enclosure estate, answer
questions, and get personalized suggestions — the return-visit and in-context-spend driver. Both
competing submissions built something similar; where we differ is treating **latency,
language, and accessibility as architectural requirements**, not follow-on polish.

## Approach

- **Retrieval-grounded**, not fine-tuned: answers are generated from a knowledge base of estate
  content (ride status, opening hours, safety rules, enclosure info) plus live data (current
  queue times from C2's forecast, current cashless balance). Every factual claim must cite a
  source record or a live call; if it can't, the companion says "let me check with staff" instead
  of guessing.
- **Multilingual and accessible by design** ([ADR-012](../adrs/ADR-012-multilingual-accessible-companion.md)):
  language is a parameter of the capability contract itself (see
  [05-ai-platform.md](../05-ai-platform.md)), not a separate product to bolt on later, and every
  eval set is run per-language, not just in English. This is a gap in both competing submissions
  — a language-based visitor product with no i18n/a11y treatment at all.
- **Offline-tolerant**: the day's itinerary and a map are cached to the visitor's device on
  generation, so patchy Wi-Fi mid-park doesn't strand someone who already got their plan. This
  caching, and the companion's UI generally, is tested on a representative low-end device under a
  simulated patchy-Wi-Fi network profile — not just budgeted for on paper in
  [ADR-011](../adrs/ADR-011-latency-budget-visitor-ai.md)'s hop table — as part of the same CI
  gate as the rest of C4's fitness functions ([06-verification.md](../06-verification.md)).
- **Stateless by design, personalized through data it's actually given**: the companion carries
  no memory of its own across turns beyond the current conversation or across visits
  ([ADR-014](../adrs/ADR-014-no-agent-platform-at-launch.md)). Return-visit personalization comes
  from looking up the visitor's **opt-in loyalty record** ([ADR-007](../adrs/ADR-007-anonymous-visitor-analytics.md))
  at conversation start, not from the companion recalling anything itself — the "return-visit
  uplift" this capability is funded against ([08-cost-and-payback.md](../08-cost-and-payback.md))
  is a data-lookup effect, not a memory-architecture one.

## Accessibility as a functional requirement, not polish

WCAG 2.2 AA is an acceptance criterion for the companion's interface, checked by an automated
axe-core scan in CI with a **0 critical/serious violations** gate — the same fitness-function
mechanism every other AI capability is held to (see the table in
[06-verification.md](../06-verification.md)), not a manual audit that happens once before launch
and drifts afterward. Concretely: full keyboard navigation (no chat action reachable only by
mouse/touch), screen-reader-announced message updates (not silent DOM changes), and color
contrast that holds under the confidence-band visual treatment used elsewhere in the guest UI.
Full rationale in [ADR-012](../adrs/ADR-012-multilingual-accessible-companion.md).

## Guardrails / grounding

Safety-relevant facts (ride restrictions, allergen info, opening hours) are never generated —
they're inserted verbatim from structured records, with generation only used for phrasing around
them. This removes the highest-stakes hallucination risk from the highest-exposure surface.

## Latency budget

Visitor-facing chat has an explicit hop budget (retrieval → inference → guardrail check →
render) rather than an unstated "should feel fast" — see
[06-verification.md](../06-verification.md) and [ADR-011](../adrs/ADR-011-latency-budget-visitor-ai.md).

## Serves

- **Lever:** ↑ attendance (return-visit driver), ↑ spend/guest (in-context suggestions, shares C3's offer engine)
- **ADRs:** [ADR-011](../adrs/ADR-011-latency-budget-visitor-ai.md), [ADR-012](../adrs/ADR-012-multilingual-accessible-companion.md), [ADR-004](../adrs/ADR-004-ai-gateway-provider-indirection.md)
- **Kill criterion:** see [04-ai-capabilities/README.md](README.md)
