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

## Round two: adversarial self-review

A day after the first draft, we ran the same submission back through AI — this time in a
reviewer role, not an author role: read every file, adopt the perspective of the three named
judges' actual professional focus areas (quality/AI-V&V, AI agent platforms/production ML,
DDD/EDA/ADR rigor), and find what each would push back on. This is a different task from
drafting and it caught a different class of problem.

What it found, specifically:

- **Claims not backed by the file they pointed to.** [ADR-012](adrs/ADR-012-multilingual-accessible-companion.md)
  stated that accessibility was "checked in the same CI gate as the rest of C4's fitness
  functions in 06-verification.md" — but that table had no such row. [10-risks.md](10-risks.md)'s
  R4 and R5 both named a specific live-production check ("weekly false-negative audit,"
  "individual attribution... explicitly designed") that [06-verification.md](06-verification.md)
  didn't actually measure anywhere. Same failure shape both times: a promise made in one document,
  never made concrete in the document it pointed to.
- **A real trade-off recorded as a throwaway line instead of a decision.** The choice not to
  build a multi-step agent platform for C4 lived as one bullet under "Not building" in
  [01-business-case.md](01-business-case.md) — true, but not argued, and the single point most
  likely to draw a direct challenge from a judge whose own book is about production agent
  platforms. It's now [ADR-014](adrs/ADR-014-no-agent-platform-at-launch.md), with the trade-off
  named honestly, including what we give up.
- **An operational claim that had never actually been exercised.** [07-operations.md](07-operations.md)'s
  game day tested exactly one of the risks in [10-risks.md](10-risks.md) (a provider outage) and
  called the rest "designed for" — later fixed to rotate through four drills covering
  connectivity loss, duplicate-event delivery, and sensor cross-confirmation as well.

None of this changed the architecture's shape. All of it changed whether the document's claims
about the architecture would survive being checked against each other — which, for a submission
judged by people who ask exactly those kinds of cross-checking questions, is the part that
actually gets tested in the room.

## What stayed human

The three-lever framing itself, the decision to treat this as "deliberately small architecture"
rather than matching the competing submissions' scope, and every judgment call about what to
build vs. explicitly not build — those are decisions, not facts an AI system can look up or
verify, and they're the part of this submission we'd stand behind in a room with the judges.
