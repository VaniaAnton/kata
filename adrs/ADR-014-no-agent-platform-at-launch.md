# ADR-014: No multi-step agent platform at launch — grounded LLM + typed tools behind the existing gateway

- **Status:** accepted
- **Date:** 2026-09-08
- **Serves:** Judging criterion 1 (innovative use of AI, scoped honestly); Judging criterion 2 (suitability under constraints); Lever: cost-to-serve (engineering capacity)

## Context

[01-business-case.md](../01-business-case.md) states, in one line under "Not building," that we
are not shipping an agentic multi-step AI orchestration layer at launch. That line deserves its
own trade-off record rather than living as an aside, because it is the one place our small-team
thesis ([ADR-001](ADR-001-modular-monolith.md)) most directly collides with what a judge steeped
in production AI-agent platforms will expect to see: a reusable agent layer, a typed tool
registry shared across capabilities, and a two-tier (long-term + working) memory architecture.

Today, C4's actual need is narrower: answer a visitor's question, optionally call one live data
tool (queue times, cashless balance), and cite a source
([C4-guest-companion.md](../04-ai-capabilities/C4-guest-companion.md)). No capability in
[04-ai-capabilities/](../04-ai-capabilities/README.md) currently needs a chain of more than one
tool call, and none needs state carried across a session.

## Decision

C4 ships as a grounded LLM with typed tool access, requested through the same capability
contract every other AI call uses ([ADR-004](ADR-004-ai-gateway-provider-indirection.md)) — not
as a standalone agent runtime with its own registry, orchestrator, or shared memory store. Tool
access is scoped per-request (retrieval lookup, live queue/balance read), least-privilege, and
stateless between turns beyond the conversation itself.

We are explicitly not building: a cross-capability tool registry, a long-term/working memory
tier shared across capabilities, multi-step autonomous task planning, or an agent-population
lifecycle (versioning/rollback at the *agent* level, distinct from the model-bundle rollback
[ADR-010](ADR-010-every-inference-is-an-event.md) already gives us).

## Alternatives considered

| Option | Why not |
|---|---|
| Full agent platform at launch: typed tool registry shared across capabilities, long-term + working memory tiers, multi-step orchestration | Solves a scale-of-agent-population problem we don't have — one capability, one tool-call depth of one. Building shared agent infrastructure ahead of a second consumer is speculative platform work competing for the same 3-5 engineers already carrying five capabilities across [09-roadmap.md](../09-roadmap.md)'s three phases |
| Per-capability ad-hoc tool wiring outside the AI Gateway | Reinvents the provider/lifecycle/cost problem [ADR-004](ADR-004-ai-gateway-provider-indirection.md) already solves once, for no gain — we'd still need eval, rollback, and cost tracking per tool call, just duplicated instead of centralized |
| **Grounded LLM + typed tool access via the existing capability contract, no standalone agent runtime** | — chosen |

## Consequences

- We give up real capability: no multi-step autonomous task composition (e.g. a companion that
  both checks a ride's maintenance-hold status *and* rebooks a visitor's slot in one flow), and
  no reusable memory substrate a second agent-shaped capability could sit on without rebuilding
  one. This is a genuine ceiling, not a hidden one — say so plainly if a judge asks.
- We avoid a second operational surface: agent lifecycle management, cross-capability tool-misuse
  guardrails, and a memory store all carry their own on-call and security surface that
  [07-operations.md](../07-operations.md)'s 3-5 person rotation is not sized for today.
- This is a bet that "one narrow, well-governed tool-user" beats "a platform for an agent
  population of one" at this scale — a reviewer coming from large-scale agent-platform practice
  will disagree with the framing by default; naming that disagreement here is the point of this
  ADR existing.

## How we will know this was right

Track the fraction of concierge queries a single grounded-retrieval-plus-one-tool-call cannot
satisfy (visitor asks for something requiring two dependent actions, e.g. "book me a table and
alert me when the queue clears"). If that fraction exceeds roughly 15% sustained over a season,
that is the signal to build a minimal orchestration layer scoped to the actual demonstrated
chains — not to build a general agent platform pre-emptively.
