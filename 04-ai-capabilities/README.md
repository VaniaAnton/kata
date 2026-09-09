# AI Capabilities

Every capability below exists because it serves one of the three levers in
[01-business-case.md](../01-business-case.md), not because it's a fashionable AI use case. Cost and
effect figures are derived in [08-cost-and-payback.md](../08-cost-and-payback.md); this table is
the summary.

| Capability | Lever | Cost (Yr1, order of magnitude) | Expected effect | Kill criterion |
|---|---|---|---|---|
| [C1 — Animal welfare](C1-animal-welfare.md) | ↓ cost-to-serve | ~€140k build + ~€35k/yr run | -15% acute vet cost, earlier disease detection | If false-negative rate on held-out welfare incidents doesn't beat the existing keeper-observation baseline after 6 months, revert to keeper-only monitoring. Separately gated on individual-attribution accuracy ≥ 0.95 and weekly live-audit consistency ([06-verification.md](../06-verification.md)) — a breach there triggers recalibration, not full reversion |
| [C2 — Visitor flow](C2-visitor-flow.md) | ↑ attendance / ↓ cost-to-serve | ~€90k build + ~€20k/yr run | Staffing cost scales at ~2.5x rather than linear 3x at full attendance | If forecast MAPE > naive baseline ("same weekday last year × seasonal factor") for 2 consecutive quarters, drop the model and keep the naive baseline. Pricing guidance is gated separately: any corridor-bound violation freezes auto-apply immediately, and it must not underperform static pricing over a rolling month ([06-verification.md](../06-verification.md)) |
| [C3 — On-site spend](C3-onsite-spend.md) | ↑ spend/guest | ~€110k build + ~€25k/yr run | +20-25% uplift on-site spend/visit | If pre-order conversion doesn't beat the queue-based baseline within one season, cut contextual offers and keep cashless-only |
| [C4 — Guest companion](C4-guest-companion.md) | ↑ attendance / ↑ spend | ~€150k build + ~€60k/yr run (LLM inference) | Measurable return-visit uplift + companion-attributed spend | If AI Task Success Rate < 80% or safety-refusal rate < 100% after 2 evaluation cycles, freeze rollout and stay in FAQ-only mode |
| [C5 — Ride maintenance](C5-ride-maintenance.md) | ↓ cost-to-serve | ~€100k build + ~€20k/yr run | -30% unplanned ride-downtime events | If flagged-ride precision < 60% (too many false alarms fatigue inspectors) after one season, revert to fixed inspection schedule |

Full arithmetic, hardware BOM, and cloud run-rate: [08-cost-and-payback.md](../08-cost-and-payback.md).
Funding-gate mechanism that enforces the kill criteria above: [ADR-005](../adrs/ADR-005-ai-funding-gate.md).
