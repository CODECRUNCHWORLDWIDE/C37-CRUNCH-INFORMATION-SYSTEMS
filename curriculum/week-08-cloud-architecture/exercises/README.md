# Week 8 — Exercises

Three exercises. Work them in order — Exercise 2's deployed database is what Exercise 3's cost estimate is modeled on, and both feed the mini-project.

## How to work these

- Each exercise has a **Setup**, a set of **Tasks**, an **Expected result** to check yourself against, and a **Done when** checklist.
- Exercise 2 requires a free account on a cloud provider (Render, Neon, Railway, or Supabase all work — the instructions use Render Postgres as the primary example and note the equivalent step for the others). No credit card is required for any of these free tiers as of this writing; if a provider you pick asks for one, use a different provider from the list.
- Per this course's data rule, Exercise 3's cost estimate is a **Python script**, never a spreadsheet — the whole point is that "what if traffic 10x's" becomes a parameter change, not a rebuilt spreadsheet.
- Write real, runnable artifacts — a `.md` file, a `.py` script, a real deployed database — committed to your portfolio.

## Exercises

| # | File | Builds | Time |
|--:|------|--------|-----:|
| 1 | [exercise-01-pick-a-cloud-model.md](./exercise-01-pick-a-cloud-model.md) | A justified IaaS/PaaS/SaaS classification for 10 real systems | 1h |
| 2 | [exercise-02-deploy-a-database.md](./exercise-02-deploy-a-database.md) | A live managed PostgreSQL instance loaded with the `crunchcycles` schema | 1h |
| 3 | [exercise-03-estimate-monthly-cost.md](./exercise-03-estimate-monthly-cost.md) | A Python cost model for Crunch Cycles' cloud deployment | 1h |

## Submission

Commit each exercise's deliverable to your portfolio under `c37-week-08/exercise-0N/`. For Exercise 2, keep your database connection string and password **out** of Git — commit a `.env.example` with placeholder values instead, and add your real `.env` to `.gitignore` before your first commit this week.
