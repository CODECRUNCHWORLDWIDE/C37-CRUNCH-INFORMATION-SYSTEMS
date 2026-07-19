# Exercises — Week 3

Three exercises, meant to be done in order. Each builds the specific hand-skill the lectures explained: drawing an ER diagram from scratch, normalizing a flat table step by step, and writing real `CREATE TABLE` DDL with constraints. Together they're the full pipeline — model, normalize, implement — that the mini-project asks you to run end to end on CrunchRide.

## How to work them

- Do them **in order** — Exercise 3's DDL depends on the normalized design Exercise 2 teaches, and both lean on the ER-diagram thinking from Exercise 1.
- Read the relevant lecture section again before starting if you're unsure — these are meant to be worked with the lecture open, not from memory alone.
- Every exercise has a "Done when…" checklist. Don't move on until you can tick every box honestly.
- Keep your work — the mini-project and Week 4 both build on the modeling habits (not the literal files) from this set.

## What's here

| # | File | Skill drilled | ~Time |
|--:|------|---------------|------:|
| 1 | [exercise-01-draw-an-er-diagram.md](./exercise-01-draw-an-er-diagram.md) | Extracting entities/attributes/relationships from plain English; crow's-foot cardinality | 1h |
| 2 | [exercise-02-normalize-a-flat-table.md](./exercise-02-normalize-a-flat-table.md) | Spotting anomalies; normalizing 1NF → 2NF → 3NF step by step | 1.5h |
| 3 | [exercise-03-write-ddl.md](./exercise-03-write-ddl.md) | Translating a normalized design into real PostgreSQL `CREATE TABLE` statements with constraints | 1.5h |

## Submission

Commit each exercise's output to your portfolio under `c37-week-03/exercise-0N/` as you finish it, per that exercise's own submission note.
