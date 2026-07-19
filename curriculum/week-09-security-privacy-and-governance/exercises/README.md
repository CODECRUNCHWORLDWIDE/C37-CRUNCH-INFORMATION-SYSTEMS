# Week 9 — Exercises

Three exercises, each building on the previous one. Work them in order — Exercise 2 assumes the login API from Exercise 1, and Exercise 3 assumes the classification you build in Exercise 2.

## How to work these

- Each exercise has a **Setup**, a set of **Tasks**, an **Expected result** to check yourself against, and a **Done when** checklist.
- Write real, runnable code — Python, SQL, and one Markdown policy document, committed to your portfolio.
- Continue using the `crunchcycles` Postgres database and the `app_users` table from this week's [README setup](../README.md#setup). Keep your `.env` file out of Git — check `.gitignore` before your first commit this week.
- If a task says "expected," that's a checkpoint — if your output doesn't match, something upstream is off; debug before moving on.

## Exercises

| # | File | Builds | Time |
|--:|------|--------|-----:|
| 1 | [exercise-01-add-role-based-access.md](./exercise-01-add-role-based-access.md) | A working login endpoint + RBAC-protected Crunch Cycles API | 1.5h |
| 2 | [exercise-02-classify-personal-data.md](./exercise-02-classify-personal-data.md) | A column-level data classification + a masking view for low-privilege roles | 1.5h |
| 3 | [exercise-03-write-a-retention-policy.md](./exercise-03-write-a-retention-policy.md) | A written retention policy + the SQL that implements it | 1h |

## Submission

Commit each exercise's deliverable to your portfolio under `c37-week-09/exercise-0N/`. Never commit a real password, JWT secret, or `.env` file — double-check `git status` before every commit this week specifically.
