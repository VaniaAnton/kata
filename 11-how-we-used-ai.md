# How We Used AI (kata theme: AI-Assisted Software Architecture)

Both competing submissions in this kata are about AI *inside the product* — animal cameras, LLM
concierges, demand forecasters. Neither says anything about how AI was used to do the
architecture work itself, even though that's the kata's actual stated theme. We're not
confident that gap costs points, but leaving it unaddressed felt dishonest given how much AI
assistance actually went into this document, so here's the plain account.

## What AI actually did here

- **Competitive research**: reading both competing submissions in full (README, requirements,
  HLD, all ADRs, all diagrams) and extracting their architecture, tech choices, AI use cases, and
  — most usefully — their gaps. This is what surfaced the on-site-spend gap that became the basis
  for [C3](04-ai-capabilities/C3-onsite-spend.md), and the observability/latency-budget gaps that
  shaped [06-verification.md](06-verification.md) and [07-operations.md](07-operations.md).
- **Scaffolding**: the file structure, the ADR template, and the initial skeleton of every
  document with placeholder markers — a mechanical step that would otherwise have eaten a day of
  the nine available.
- **Drafting**: the prose, numbers, and diagrams in this repo were AI-drafted from the plan above,
  then reviewed by the team before submission.

## Where it got things wrong, and how that was caught

- The first pass at the business-case numbers leaned on a specific ticket price pulled from
  nowhere in particular, the same mistake flagged as a weakness in one of the competing
  submissions. It was caught by asking "where does that number come from" and rewriting it as an
  explicit, labeled assumption with an "if wrong" column instead — see
  [01-business-case.md](01-business-case.md).
- Diagram content was initially left as bare placeholders rather than actual diagrams in an
  earlier scaffolding pass — a reminder that AI-generated structure without AI-generated (and
  human-reviewed) substance is not a finished document, just a table of contents.

## What stayed human

The three-lever framing itself, the decision to treat this as "deliberately small architecture"
rather than matching the competing submissions' scope, and every judgment call about what to
build vs. explicitly not build — those are decisions, not facts an AI system can look up or
verify, and they're the part of this submission we'd stand behind in a room with the judges.
