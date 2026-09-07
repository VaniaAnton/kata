# AI Platform

The AI Gateway is the one place every AI capability's provider risk, cost, and evaluation
lifecycle gets managed — extracted from the monolith specifically because it needs to evolve
faster than the rest of the system ([ADR-004](adrs/ADR-004-ai-gateway-provider-indirection.md)).

## Capability contract

Domain services never call a model or a provider directly. They request a **capability** (e.g.
`welfare-anomaly-score`, `concierge-answer`, `spend-offer-rank`) and the gateway resolves that to
a versioned bundle of `{model, prompt/config, eval suite}`. Swapping a provider or a model
version means publishing a new bundle version that passes the capability's eval suite — it never
means touching the calling code. This is also where the language parameter for C4 lives: `{
capability: concierge-answer, language: de }` resolves to a bundle validated for German, not just
whatever the English eval suite happened to pass.

## Provider indirection & budgets

- **Per-capability budget** with cost telemetry; alerts at 70%/90% of budget, automatic downgrade
  to a cheaper tier at 100% rather than a silent cost overrun.
- **Tiered routing**: cheap/fast model as default, escalate to a stronger model only when the
  cheap tier's own confidence signal is low.
- **Named secondary provider** for every capability, shadow-tested periodically so "switch
  providers" is a config change validated ahead of time, not a scramble during an outage.
- **Deterministic fallback always exists** below the model tier — every capability in
  [04-ai-capabilities/](04-ai-capabilities/README.md) states what it does with zero AI available.

This directly answers "what happens if the best model today isn't the best tomorrow" and "what
happens if our provider shuts down or changes prices": neither event touches calling code, both
are config/registry changes gated by the same eval suite every other model change goes through.

## Diagram — AI capability pipeline

```mermaid
flowchart LR
  Service[Domain Service] -->|request capability| Gateway[AI Gateway 🤖]
  Gateway --> Router[Model Router]
  Router --> Primary[Primary Model]
  Router -.fallback.-> Secondary[Secondary Model]
  Router -.fallback.-> Deterministic[Deterministic Fallback]
  Registry[(Prompt / Model Registry)] --> Router
  Gateway --> CostMonitor[Cost Monitor]
  Gateway --> Trace[(Trace: capability, model_version, cost, confidence)]

  subgraph Lifecycle[Promotion Lifecycle]
    direction LR
    Candidate --> Shadow --> Production --> Retired
  end
  Registry --- Lifecycle
```
