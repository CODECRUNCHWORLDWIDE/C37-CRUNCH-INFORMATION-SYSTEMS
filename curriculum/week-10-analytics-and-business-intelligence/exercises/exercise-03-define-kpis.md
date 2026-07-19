# Exercise 3 — Define KPIs for a Business Goal

**Goal:** starting from a stated business goal — not from "what's easy to query" — derive three KPIs, define each precisely enough to write unambiguous SQL, then write and run that SQL against your populated `crunchcycles_dw` warehouse from Exercise 2.

**Estimated time:** 90 minutes.

## Setup

Your `crunchcycles_dw.warehouse` schema must be populated (Exercise 2 complete, `fact_order_items` has 54 rows). Re-read Lecture 3 §1–3 before starting — you're about to do, for a new goal, exactly what those sections modeled for Crunch Cycles' existing KPIs.

**The business goal, assigned:**

> "Crunch Cycles' leadership has noticed that revenue growth over the last two quarters has come almost entirely from existing customers reordering, not from new customer acquisition. They want to understand: is this a strength (a loyal customer base) or a risk (a stalling top of funnel) — and which regions and sales reps are most and least dependent on repeat business."

## Tasks

**1. Derive three KPIs from the goal.** In `kpi-definitions.md`, for each of three KPIs:

- **Name** the KPI.
- **State the business question it answers**, in one sentence, tracing directly back to the goal above (not a generic definition — *this* goal, specifically).
- **Apply the test from Lecture 3 §1**: if this number moved 20% in either direction, what would Crunch Cycles' leadership actually do differently? Write the answer — if you can't state a concrete action, pick a different KPI.
- **Write the precise formula**: exact numerator, exact denominator, exact filters (which `order_status` values count, what date range, what grouping) — precise enough that two different people implementing it from your definition would get the identical number.

At minimum, one of your three KPIs must be a **rate** (like repeat customer rate), not a raw count — re-read Lecture 3 §4 on why a raw cumulative count doesn't answer "is this a strength or a risk" the way a rate does.

**2. Write and run the SQL for each KPI**, against `crunchcycles_dw.warehouse`. Save each query, with a comment above it stating which KPI it computes, in `kpi-queries.sql`. Run all three and capture their actual output.

**3. Answer the business question**, in `findings.md` (~200 words): given your three KPIs' actual computed values against Crunch Cycles' current data, is the reordering-heavy growth pattern a strength or a risk? Which regions or reps (if your KPIs surface this) look most exposed if new-customer acquisition doesn't pick up? Be specific — cite the actual numbers your queries returned, not a general impression.

## Expected result (spot checks)

- All three KPI definitions in `kpi-definitions.md` pass the "20% move" test explicitly — each one names a concrete action leadership would take.
- At least one KPI is a rate/percentage, with an explicitly stated denominator.
- `kpi-queries.sql` runs without error against a freshly Exercise-2-loaded `crunchcycles_dw` and every query's `WHERE` clause explicitly states which `order_status` values it includes (per Lecture 3 §3's "every revenue KPI filters explicitly" rule).
- `findings.md` references actual numbers from your query output, not hypothetical ones.

## Done when…

- [ ] `kpi-definitions.md`, `kpi-queries.sql`, and `findings.md` all exist and are internally consistent (the SQL actually computes what the definitions say it computes).
- [ ] Every KPI definition traces back to the stated business goal in one sentence — a reader unfamiliar with the goal could match each KPI to the part of the goal it addresses.
- [ ] `findings.md` gives a real, numbers-backed answer to "strength or risk," not a hedge.

## Stretch

- Add a fourth KPI: new-customer revenue vs. repeat-customer revenue, split by month, to show whether the *mix* is shifting over time (not just a single repeat rate snapshot) — this needs a `CASE WHEN` on whether a customer's *first* completed order falls in the same month as the one being measured.
- For one KPI, write the query **two ways** — once against the warehouse star schema, once against the raw `crunchcycles` operational tables directly with the same joins pre-Week-10 style — and time both with `EXPLAIN ANALYZE`. Report the difference and connect it back to Lecture 1's argument for why a warehouse exists.

## Submission

Commit `kpi-definitions.md`, `kpi-queries.sql`, and `findings.md` to your portfolio under `c37-week-10/exercise-03/`.
