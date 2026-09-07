# C2 — Visitor Flow Forecasting & Staffing

## Problem

Growing to 15,000 visitors/day only works if staffing and ride/enclosure capacity grow with
demand rather than linearly with a worst-case assumption. Naive 3x staffing at 3x attendance
would erase most of the cost-to-serve improvement the rest of this plan is fighting for; naive
flat staffing would collapse service quality and undo the attendance gain itself.

## Approach

Two outputs, kept deliberately separate because they have different risk profiles:

1. **Demand forecast (ML)** — footfall history, calendar, weather, pre-sold tickets, and known
   events, predicting visitor counts 7 days out at 30-minute resolution with a prediction
   interval, retrained weekly. This is genuinely probabilistic and gets the full verification
   treatment in [06-verification.md](../06-verification.md).
2. **Roster generation (deterministic solver, not ML)** — a constraint solver takes the forecast
   as input and produces shift rosters respecting legal break rules, skill requirements, and
   *minimum safety staffing floors that the forecast can never override*. We deliberately keep
   this off the ML path: a roster is a hard-constraint optimization problem, not a
   pattern-recognition one, and a wrong roster is an operational incident, not a "low confidence"
   result to shrug off.

Secondary output: the same demand forecast feeds **family-bundle pricing guidance** — proposed
price adjustments within Countess-set min/max/max-Δ-per-day corridors, auto-applied inside the
corridor and requiring sign-off outside it. Price never varies by individual identity, only by
time/date cohort.

Deterministic fallback: if the forecast is unavailable or fails its fitness gate, rostering
reverts to last season's same-weekday pattern — the naive baseline the model has to beat
(below), not a shutdown.

## Serves

- **Lever:** ↑ attendance (capacity enabler), ↓ cost-to-serve (staffing efficiency)
- **ADRs:** [ADR-003](../adrs/ADR-003-deterministic-core.md) (solver stays deterministic), [ADR-005](../adrs/ADR-005-ai-funding-gate.md)
- **Kill criterion:** forecast MAPE must beat the naive baseline ("same weekday last year ×
  seasonal factor") — see [04-ai-capabilities/README.md](README.md)
