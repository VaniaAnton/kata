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
  generation, so patchy Wi-Fi mid-park doesn't strand someone who already got their plan.

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
