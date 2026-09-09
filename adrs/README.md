# Architecture Decision Records

| # | Title | Serves | Status |
|---|---|---|---|
| [001](ADR-001-modular-monolith.md) | Modular monolith with selective extraction | Driving characteristic: operability by a small team; Lever: cost-to-serve | accepted |
| [002](ADR-002-edge-store-and-forward.md) | Edge-first store-and-forward over MQTT | NFR: patchy Wi-Fi; Lever: cost-to-serve (C1, C5 telemetry) | accepted |
| [003](ADR-003-deterministic-core.md) | Deterministic core, AI as advisory overlay | Criterion 5 (AI/architecture alignment); Risk: safety-critical paths | accepted |
| [004](ADR-004-ai-gateway-provider-indirection.md) | Capability-based AI gateway + provider indirection | Criterion 4 (dealing with uncertainty) | accepted |
| [005](ADR-005-ai-funding-gate.md) | AI funding gate | Criteria 2 (suitability) + 4 (uncertainty); Lever: all three | accepted |
| [006](ADR-006-confidence-bands-cost-of-error.md) | Confidence bands priced by cost-of-error | Capability: C1; Risk: R4 (alert fatigue) | accepted |
| [007](ADR-007-anonymous-visitor-analytics.md) | Anonymous-by-default visitor analytics | Capability: C2; Capability: C3 | accepted |
| [008](ADR-008-individual-animal-identity.md) | Individual animal identity | Capability: C1 | accepted |
| [009](ADR-009-one-credential-ticket-and-spend.md) | One credential for ticket + cashless spend | Capability: C3 | accepted |
| [010](ADR-010-every-inference-is-an-event.md) | Every AI inference is an event | Criterion 6 (validation & verification) | accepted |
| [011](ADR-011-latency-budget-visitor-ai.md) | Latency budget for visitor-facing AI | Capability: C4 | accepted |
| [012](ADR-012-multilingual-accessible-companion.md) | Multilingual & accessible companion | Capability: C4 | accepted |
| [013](ADR-013-advisory-predictive-maintenance.md) | Advisory predictive maintenance on heritage rides | Capability: C5 | accepted |
| [014](ADR-014-no-agent-platform-at-launch.md) | No multi-step agent platform at launch | Criteria 1 (innovative AI, scoped) + 2 (suitability); Lever: cost-to-serve | accepted |
| [015](ADR-015-internal-outbox-idempotent-consumers.md) | Transactional outbox + idempotent consumers | Criterion 5 (AI/architecture alignment); extends ADR-001 | accepted |
| [016](ADR-016-edge-model-lifecycle-and-sensor-calibration.md) | Edge model lifecycle, golden-set methodology, sensor calibration | Criterion 6 (validation & verification); extends ADR-004, ADR-010 | accepted |

## Traceability

Every ADR above carries a `Serves` line pointing back to a lever, a judging criterion, a risk,
or a capability — checked against the individual ADR files, not just this table. The reverse
direction also holds: every lever in [01-business-case.md](../01-business-case.md), every
capability in [04-ai-capabilities/](../04-ai-capabilities/README.md), and every risk in
[10-risks.md](../10-risks.md) links forward to at least one ADR here. There is no orphaned
decision and no unexplained requirement.

New ADRs should follow [template.md](template.md) and get added to the table above with the
same rigor — a decision without a `Serves` line is a decision without a stated reason to exist.
