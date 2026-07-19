# Week 4 — Exercises

Three exercises, done in order. Each builds on the last: joins first (pure SQL), then bringing a result into pandas, then wrapping the whole thing into a parameterized script. All three run against the `crunchcycles` schema from the [week README](../README.md#setup-load-the-week-3-schema) — load it once before you start.

## How to work these

1. **Set up once.** Confirm `SELECT COUNT(*) FROM orders;` returns `40` before you start Exercise 1.
2. **Work in order.** Exercise 2 assumes the query patterns from Exercise 1; Exercise 3 assumes the pandas basics from Exercise 2.
3. **Save your work.** Exercise 1 → a `.sql` file. Exercises 2–3 → `.py` scripts you can re-run.
4. **Check the "Expected outcome"** under each task before moving on — if your number or shape doesn't match, re-read the relevant lecture section before continuing; don't push forward on a wrong foundation.
5. **Use a real editor**, not just the `psql`/`sqlite3` shell, for anything longer than two lines — you'll be reading and revising these queries as you go.

## Files

| # | Exercise | Skill drilled | ~Time |
|--:|----------|---------------|------:|
| 1 | [exercise-01-answer-with-joins.md](./exercise-01-answer-with-joins.md) | Every join type, answering 10 real business questions | 1.5h |
| 2 | [exercise-02-sql-to-pandas.md](./exercise-02-sql-to-pandas.md) | `read_sql`, dtype cleanup, `merge`, `pivot_table` | 1.5h |
| 3 | [exercise-03-parameterize-a-report.md](./exercise-03-parameterize-a-report.md) | `argparse`, parameterized queries, `to_sql` | 1.5h |

When all three are done, move on to the [challenges](../challenges/README.md).
