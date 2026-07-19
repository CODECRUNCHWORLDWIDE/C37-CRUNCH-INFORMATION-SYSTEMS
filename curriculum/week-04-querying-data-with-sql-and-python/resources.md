# Week 4 — Resources

Curated, free, official. Install steps first, reference docs after.

## Install

**PostgreSQL 16+**

- macOS: `brew install postgresql@16 && brew services start postgresql@16`
- Ubuntu/Debian: `sudo apt install postgresql postgresql-contrib`
- Windows: [postgresql.org/download/windows](https://www.postgresql.org/download/windows/) (EDB installer)
- Verify: `psql --version`

**SQLite (fallback, usually already installed)**

- macOS/Linux: `sqlite3 --version` (ships with the OS on most systems)
- Windows: [sqlite.org/download.html](https://www.sqlite.org/download.html)

**Python 3.10+**

- [python.org/downloads](https://www.python.org/downloads/) if you don't already have it.
- Verify: `python3 --version`

**This week's Python packages**

```bash
python3 -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install pandas sqlalchemy psycopg2-binary python-dotenv
```

- **pandas** — DataFrames, the core data-wrangling library.
- **SQLAlchemy** — the engine/connection layer `pandas.read_sql`/`to_sql` use to talk to Postgres.
- **psycopg2-binary** — the actual PostgreSQL driver SQLAlchemy calls under the hood.
- **python-dotenv** — loads a local `.env` file (your `DATABASE_URL`) into environment variables, so credentials never live in source code.

## SQL — joins, aggregation, CTEs

- **PostgreSQL — Joins between tables (tutorial):** <https://www.postgresql.org/docs/current/tutorial-join.html>
- **PostgreSQL — Join types reference (`SELECT`):** <https://www.postgresql.org/docs/current/sql-select.html#SQL-FROM>
- **PostgreSQL — Aggregate functions:** <https://www.postgresql.org/docs/current/functions-aggregate.html>
- **PostgreSQL — Aggregate expressions (`GROUP BY`/`HAVING`):** <https://www.postgresql.org/docs/current/sql-select.html#SQL-GROUPBY>
- **PostgreSQL — `WITH` queries (CTEs):** <https://www.postgresql.org/docs/current/queries-with.html>
- **PostgreSQL — Date/time functions (`DATE_TRUNC`, `EXTRACT`):** <https://www.postgresql.org/docs/current/functions-datetime.html>
- **SQLite — Join clause:** <https://www.sqlite.org/syntax/join-clause.html>
- **SQLite — Common Table Expressions:** <https://www.sqlite.org/lang_with.html>
- **Use The Index, Luke — how joins actually execute (free book, deep):** <https://use-the-index-luke.com/>

## Python + pandas

- **pandas — 10 minutes to pandas (official quickstart):** <https://pandas.pydata.org/docs/user_guide/10min.html>
- **pandas — `read_sql`:** <https://pandas.pydata.org/docs/reference/api/pandas.read_sql.html>
- **pandas — `DataFrame.to_sql`:** <https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.to_sql.html>
- **pandas — Merge, join, concatenate:** <https://pandas.pydata.org/docs/user_guide/merging.html>
- **pandas — `groupby`:** <https://pandas.pydata.org/docs/user_guide/groupby.html>
- **pandas — Reshaping (`pivot_table`, `melt`):** <https://pandas.pydata.org/docs/user_guide/reshaping.html>
- **pandas — Working with missing data:** <https://pandas.pydata.org/docs/user_guide/missing_data.html>
- **SQLAlchemy — Engine configuration (connection strings):** <https://docs.sqlalchemy.org/en/20/core/engines.html>
- **SQLAlchemy — Using textual SQL with bound parameters:** <https://docs.sqlalchemy.org/en/20/core/tutorial.html#using-textual-sql>

## Building pipelines

- **Python — `argparse` HOWTO:** <https://docs.python.org/3/howto/argparse.html>
- **Python — `logging` HOWTO:** <https://docs.python.org/3/howto/logging.html>
- **python-dotenv — usage:** <https://pypi.org/project/python-dotenv/>
- **crontab.guru — build/verify a cron schedule expression interactively:** <https://crontab.guru/>
- **PostgreSQL — Transactions:** <https://www.postgresql.org/docs/current/tutorial-transactions.html>

## Why not a spreadsheet

- **PostgreSQL — Concurrency control (why a real database handles simultaneous access; a spreadsheet doesn't):** <https://www.postgresql.org/docs/current/mvcc.html>
- If you want the counter-argument fully steelmanned before rejecting it, see [C41 Crunch Excel](../../../C41-CRUNCH-EXCEL/) — spreadsheets are a legitimate, powerful tool for the jobs they're built for (ad hoc, single-user, visual). This course is about the jobs they're *not* built for: shared, queryable, constrained, concurrent data.

## Optional deeper dives

- **"SQL Performance Explained" by Markus Winand (companion to Use The Index, Luke):** <https://sql-performance-explained.com/>
- **pandas — Enhancing performance (vectorization, avoiding row-by-row loops):** <https://pandas.pydata.org/docs/user_guide/enhancingperf.html>
- **The Twelve-Factor App — Config (why credentials live in the environment, not the code):** <https://12factor.net/config>
