# Week 1 — Exercises

Three guided exercises, ~45–90 min each. **Type every SQL statement yourself** — don't copy-paste. Reading a schema and writing rows into it are different skills, and only writing sticks.

1. **[Exercise 1 — Classify the components](exercise-01-classify-the-components.md)** — sort real-world items into people/process/data/technology; extend Riverbend's register.
2. **[Exercise 2 — Map the data flows](exercise-02-map-the-data-flows.md)** — insert and query `is_flows`; find components with no connections.
3. **[Exercise 3 — Data, information, knowledge, wisdom](exercise-03-data-information-knowledge-wisdom.md)** — climb the DIKW pyramid from a raw POS row to a real decision.

## Before you start

- You've completed all three lectures.
- You ran the seed from the [week README](../README.md) and `SELECT COUNT(*) FROM is_components;` returns **20**.
- You have a shell open: `psql riverbend` (Postgres) or `sqlite3 riverbend.db` (SQLite).

## Suggested workflow

- Open the exercise file beside your terminal.
- For each task, **write your SQL before running it**, then run it and check the output against the "Expected" note.
- If a classification call feels ambiguous, that's not a bug in the exercise — write one sentence explaining your reasoning. Ambiguity you can defend is the actual skill this week is building.
- Save your work in a `week01-solutions.sql` file (one statement per task, with the task number in a `-- comment`) plus a short `notes.md` for any written-answer tasks. You'll want both for the mini-project.

## Note on the two engines

Every task works on both PostgreSQL and SQLite. Nothing this week is engine-specific — you're only using `CREATE TABLE`, `INSERT`, `SELECT`, and `WHERE`, all of which are identical on both. If you're on SQLite, run `.headers on` and `.mode column` first so output is readable.
