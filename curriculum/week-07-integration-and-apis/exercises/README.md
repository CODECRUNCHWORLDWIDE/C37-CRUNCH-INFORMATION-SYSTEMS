# Week 7 — Exercises

Three exercises, each building on the previous lecture. Work them in order — Exercise 2 assumes the Python/Flask setup from Exercise 1's environment, and Exercise 3 assumes the database table Exercise 2 loaded.

## How to work these

- Each exercise has a **Setup**, a set of **Tasks**, an **Expected result** to check yourself against, and a **Done when** checklist.
- Write real, runnable code — a `.py` or `.md` file per exercise, committed to your portfolio.
- These use two free, no-signup public APIs: **Frankfurter** (exchange rates, from the lectures) and **JSONPlaceholder** (a fake REST API for practicing pagination and CRUD calls, at <https://jsonplaceholder.typicode.com>). Neither requires an account or an API key.
- If a task says "expected," that's a checkpoint — if your output doesn't match, something upstream is off; debug before moving on.

## Exercises

| # | File | Builds | Time |
|--:|------|--------|-----:|
| 1 | [exercise-01-design-an-api.md](./exercise-01-design-an-api.md) | A documented REST API spec for a new Crunch Cycles resource | 1.5h |
| 2 | [exercise-02-consume-a-public-api.md](./exercise-02-consume-a-public-api.md) | A Python script pulling real API data into Postgres | 1.5h |
| 3 | [exercise-03-handle-a-webhook.md](./exercise-03-handle-a-webhook.md) | A Flask endpoint that safely processes an incoming webhook | 1.5h |

## Submission

Commit each exercise's deliverable to your portfolio under `c37-week-07/exercise-0N/`. Keep your working `.env` file (API keys, secrets) **out** of Git — add it to `.gitignore` before your first commit this week.
