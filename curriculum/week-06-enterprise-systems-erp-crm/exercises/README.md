# Week 6 — Exercises

Three guided exercises, ~1.5 hours each. The first two are pen-and-paper/prose reasoning exercises (map modules, classify data) — the enterprise-systems skill is judgment as much as SQL. The third is hands-on SQL against the schema you loaded in the [week README](../README.md).

1. **[Exercise 1 — Map ERP modules](exercise-01-map-erp-modules.md)** — match ERP/CRM modules to a real company's departments and the questions each answers.
2. **[Exercise 2 — Identify master data](exercise-02-identify-master-data.md)** — classify tables and individual columns from this week's schema (and a new scenario) as master or transactional data.
3. **[Exercise 3 — Trace order-to-cash](exercise-03-trace-order-to-cash.md)** — write the SQL that follows five real Crunch Cycles deals from CRM opportunity through to shipped order.

## Before you start

- You've completed all three lectures.
- You ran the setup SQL in the [week README](../README.md) and your sanity-check counts match (`5`, `14`, `7`, `13`, `7`).
- You have a shell open: `psql crunchcycles` (PostgreSQL) or `sqlite3 crunchcycles.db` (SQLite).

## Suggested workflow

- Exercises 1 and 2 go in a plain Markdown file — write your reasoning, not just a final answer. The grading (see each exercise) rewards *why*, not just *what*.
- Exercise 3: write each query before running it, then check the output against the "Expected" note.
- Save your work as you go: `exercise-01.md`, `exercise-02.md`, `exercise-03.sql`. You'll commit them.

## Note on the two engines

Exercise 3's SQL is written to run unchanged on PostgreSQL and SQLite. Date subtraction (`received_date - po_date`) works natively in Postgres; on SQLite use `julianday(received_date) - julianday(po_date)` instead — the exercise flags this where it comes up.
