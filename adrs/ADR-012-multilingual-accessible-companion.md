# ADR-012: Multilingual & accessible companion

- **Status:** accepted
- **Date:** 2026-09-07
- **Serves:** Capability: C4 (guest companion)

## Context

The estate is a tourist attraction growing toward 15,000 visitors/day; a language-based
concierge product that only works well in one language, or that isn't usable with assistive
technology, silently excludes a real fraction of the attendance-growth target this whole
architecture is built to serve. Neither competing submission treats this as more than a single
NFR checkbox line.

## Decision

Language is a first-class parameter of the capability contract itself
([05-ai-platform.md](../05-ai-platform.md)): a request for `concierge-answer` specifies a
language, and the AI Gateway resolves it to a model/prompt bundle that has been validated —
with its own eval suite — for that specific language, not just assumed to generalize from an
English-only eval pass. Accessibility (screen-reader compatibility, keyboard navigation, WCAG
conformance for the chat UI) is an acceptance criterion for the companion's interface, checked
the same way any other functional requirement is checked, not treated as later polish.

## Alternatives considered

| Option | Why not |
|---|---|
| Ship English-only at launch, add languages/accessibility later based on demand | "Later" rarely arrives for non-core features once a system ships, and it treats a real fraction of the attendance-growth target as a nice-to-have rather than a requirement the business case already justifies |
| Rely on the underlying model's general multilingual capability with no per-language evaluation | Model multilingual quality varies significantly by language and by task; shipping without per-language eval is shipping unverified quality in every language except the one that happened to get tested |
| **Language as a capability parameter with per-language eval; accessibility as an acceptance criterion** | — chosen |

## Consequences

- Every new supported language requires its own eval-suite pass before going live — more upfront
  work per language than "just turn it on," in exchange for not shipping unverified quality.
- Accessibility requirements are testable (automated WCAG checks) and go in the same CI gate as
  the rest of C4's fitness functions in [06-verification.md](../06-verification.md).

## How we will know this was right

Track companion usage and satisfaction by language; if non-English usage is materially lower
than the estate's actual visitor-language mix, that's a signal the per-language eval bar itself
needs raising, not that this decision was unnecessary.
