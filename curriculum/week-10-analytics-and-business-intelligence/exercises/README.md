# Week 10 — Exercises

Three exercises, each building on the previous one. Work them in order — Exercise 2 loads the star schema Exercise 1 designs, and Exercise 3 queries the warehouse Exercise 2 populates.

## How to work these

- Each exercise has a **Setup**, a set of **Tasks**, an **Expected result** to check yourself against, and a **Done when** checklist.
- Write real, runnable SQL and Python — files committed to your portfolio, not just pasted into a scratch buffer.
- All three run against your `crunchcycles` (operational) and `crunchcycles_dw` (warehouse) Postgres databases from this week's [README setup](../README.md#setup).
- If a task says "expected," that's a checkpoint — if your output doesn't match, something upstream is off; debug before moving on.

## Exercises

| # | File | Builds | Time |
|--:|------|--------|-----:|
| 1 | [exercise-01-design-a-star-schema.md](./exercise-01-design-a-star-schema.md) | A documented star-schema design for a second Crunch Cycles subject area | 1.5h |
| 2 | [exercise-02-build-an-elt-job.md](./exercise-02-build-an-elt-job.md) | An idempotent Python + SQL ELT job loading `crunchcycles_dw` | 2h |
| 3 | [exercise-03-define-kpis.md](./exercise-03-define-kpis.md) | KPI definitions and SQL, derived from a stated business goal | 1.5h |

## Submission

Commit each exercise's deliverable to your portfolio under `c37-week-10/exercise-0N/`. Keep your working `.env` file (database URLs, credentials) **out** of Git — add it to `.gitignore` before your first commit this week.
