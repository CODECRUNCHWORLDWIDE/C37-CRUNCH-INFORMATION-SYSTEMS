# Lecture 3 — Repeatable Data Pipelines

> **Duration:** ~2 hours. **Outcome:** you can turn a one-off analysis script into a parameterized, re-runnable, idempotent pipeline — one command produces the same report every time, for any date range, with no manual copy-paste.

Every analysis you've written this week so far is a script you run once, read the output, and close. That's fine for exploration — it's not fine for a report someone depends on weekly. The difference between "a script that worked once" and "a pipeline" is three properties: it takes **parameters** instead of hardcoded values, it is **idempotent** (running it twice produces the same result, not duplicated data), and it is **structured** so each step (extract, transform, load) can be tested and reasoned about on its own. This lecture builds one, end to end, for Crunch Cycles' regional revenue report.

## 1. Extract, Transform, Load (ETL) — the shape every pipeline has

- **Extract** — pull raw data from the source (here: a SQL query against `crunchcycles`).
- **Transform** — clean, reshape, compute (here: pandas — dtypes, `line_total`, `pivot_table`).
- **Load** — write the result somewhere durable and queryable (here: `to_sql` into a report table).

Keep these as three distinct functions, even in a small script. It makes each one testable alone, and it's the vocabulary you'll need in Week 7 (Integration & APIs) when the source and destination are different systems entirely.

```python
# report.py
import os
import argparse
from datetime import date

import pandas as pd
from sqlalchemy import create_engine, text
from dotenv import load_dotenv


def get_engine():
    load_dotenv()
    return create_engine(os.environ["DATABASE_URL"])


def extract(engine, start_date: str, end_date: str) -> pd.DataFrame:
    """Pull completed order line items in [start_date, end_date)."""
    query = text("""
        SELECT o.order_id, o.order_date, r.region_name,
               oi.quantity, oi.unit_price
        FROM orders o
        JOIN customers c    ON o.customer_id = c.customer_id
        JOIN regions r      ON c.region_id  = r.region_id
        JOIN order_items oi ON oi.order_id  = o.order_id
        WHERE o.status = 'Completed'
          AND o.order_date >= :start_date
          AND o.order_date <  :end_date
    """)
    return pd.read_sql(query, engine, params={"start_date": start_date, "end_date": end_date})


def transform(raw: pd.DataFrame) -> pd.DataFrame:
    """Compute region-level revenue and order counts."""
    raw["line_total"] = raw["quantity"] * raw["unit_price"]
    report = (
        raw.groupby("region_name")
        .agg(order_count=("order_id", "nunique"), revenue=("line_total", "sum"))
        .reset_index()
        .sort_values("revenue", ascending=False)
    )
    return report


def load(report: pd.DataFrame, engine, table_name: str = "region_revenue_report"):
    """Replace the report table with this run's result — safe to re-run."""
    report.to_sql(table_name, engine, if_exists="replace", index=False)
```

## 2. Parameterize with `argparse` — no more hand-editing the script

Hardcoding `'2024-01-01'` inside the query means editing the source file every time someone wants a different range. Take it as a command-line argument instead:

```python
def main():
    parser = argparse.ArgumentParser(description="Regenerate the region revenue report.")
    parser.add_argument("--start", required=True, help="Start date (inclusive), YYYY-MM-DD")
    parser.add_argument("--end", required=True, help="End date (exclusive), YYYY-MM-DD")
    parser.add_argument("--table", default="region_revenue_report", help="Destination table name")
    args = parser.parse_args()

    engine = get_engine()
    raw = extract(engine, args.start, args.end)
    report = transform(raw)
    load(report, engine, args.table)

    print(f"Wrote {len(report)} rows to {args.table} for [{args.start}, {args.end})")
    print(report.to_string(index=False))


if __name__ == "__main__":
    main()
```

Now the same file answers any date range, run from a terminal:

```bash
python report.py --start 2024-01-01 --end 2024-04-01   # Q1
python report.py --start 2024-04-01 --end 2024-07-01   # Q2
```

**Never build the SQL query with an f-string on user-controlled input** — `f"... WHERE order_date >= '{start}'"` is a SQL-injection door, even on a script only you run today (today's convenience script is tomorrow's shared tool). Always use `text(...)` with named `:params` and pass `params={...}` — SQLAlchemy escapes them safely, the same discipline as a parameterized query in any language.

## 3. Idempotency — running it twice must not duplicate anything

An **idempotent** operation produces the same end state no matter how many times you run it. `if_exists="replace"` in `load()` gives you this for free for a *report* table you always fully regenerate: run the script five times with the same arguments, you get the same table each time, not five times the rows.

The trap is `if_exists="append"` used carelessly — if `load()` appended instead, running the same date range twice would double every number. Append is correct only for genuinely incremental data (a log of pipeline runs, an audit trail) where each run really does add new facts that didn't exist before. For a **report**, always ask: *if I run this again right now with the same inputs, will the table end up correct?* If the answer depends on remembering not to run it twice, it isn't a pipeline yet — it's a trap.

For an append-only case where you still need idempotency (e.g., loading each day's orders into a `daily_snapshot` table exactly once), the standard pattern is delete-then-insert for the specific partition you're reloading:

```python
def load_incremental(day_df: pd.DataFrame, engine, run_date: str, table_name: str = "daily_snapshot"):
    with engine.begin() as conn:
        conn.execute(text(f"DELETE FROM {table_name} WHERE snapshot_date = :d"), {"d": run_date})
        day_df.to_sql(table_name, conn, if_exists="append", index=False)
```

Deleting that day's rows first, inside the same transaction (`engine.begin()`), then inserting — so a re-run for the same day replaces cleanly instead of piling up duplicates, and if either step fails, the transaction rolls back instead of leaving the table half-updated.

## 4. Structure over a single monolithic script

Notice `extract`, `transform`, and `load` are separate functions, each doing one job and returning a value the next step consumes. This isn't ceremony — it's what lets you:

- **Unit test `transform` alone** with a small hand-built DataFrame, no database needed.
- **Swap the source** later (a different query, a CSV, an API) by rewriting only `extract`.
- **Swap the destination** later (a different table, a file, an email) by rewriting only `load`.
- **Read the `main()` function top to bottom** and understand the whole pipeline in four lines, because the details are named and tucked away.

This is the same "single responsibility" instinct behind normalizing a database schema (Week 3) — one piece of logic owns one job, and the pieces compose.

## 5. Logging instead of silent runs

A script a teammate (or a cron job) runs unattended needs to say what it did, especially when something goes wrong:

```python
import logging

logging.basicConfig(level=logging.INFO, format="%(asctime)s %(levelname)s %(message)s")
logger = logging.getLogger(__name__)

def extract(engine, start_date, end_date):
    logger.info("Extracting orders from %s to %s", start_date, end_date)
    df = pd.read_sql(query, engine, params={"start_date": start_date, "end_date": end_date})
    logger.info("Extracted %d rows", len(df))
    return df
```

`print()` is fine for a script you watch interactively; `logging` is what you want the moment a script might run unattended (a cron job, a scheduled task) — timestamps, severity levels, and the ability to redirect output to a file without changing the code.

## 6. Failing loudly instead of silently

If `extract()` returns zero rows, is that "no sales that quarter" (fine) or "the connection string pointed at the wrong database" (not fine)? A pipeline should distinguish the two on purpose:

```python
def extract(engine, start_date, end_date):
    df = pd.read_sql(query, engine, params={"start_date": start_date, "end_date": end_date})
    if df.empty:
        logger.warning("No completed orders found for [%s, %s) — check the date range and DB connection", start_date, end_date)
    return df
```

A pipeline that swallows an empty or malformed result and writes an empty report anyway is worse than one that crashes — a crash gets noticed; a silently wrong report gets emailed to a VP.

## 7. What "repeatable" buys you

- **Onboarding:** a new teammate runs one command instead of following a five-step tribal-knowledge checklist.
- **Auditability:** the exact query and transform logic are in version control, not in someone's memory of "the steps I did last time."
- **Scheduling:** once a script takes parameters and is idempotent, handing it to `cron` (Challenge 2) or a scheduler is trivial — the hard design work is already done.
- **Trust:** a stakeholder who's seen the same report regenerate identically from the same inputs, twice, starts trusting the number.

## 8. Check yourself

- What are the three stages of ETL, and what does each one own?
- Why is `f"WHERE order_date >= '{start}'"` dangerous even in a script only you run?
- What makes `if_exists="replace"` idempotent and `if_exists="append"` (used carelessly) not?
- Describe the delete-then-insert pattern for idempotent incremental loads, in one sentence.
- Why separate `extract`/`transform`/`load` into different functions instead of one long script?
- What's the difference between a script failing loudly and failing silently — and why does silent failure matter more for a report?

Exercise 3 has you build exactly this pipeline shape for the Crunch Cycles data; the mini-project extends it into the full report.

## Further reading

- **Python — `argparse` tutorial:** <https://docs.python.org/3/howto/argparse.html>
- **Python — `logging` HOWTO:** <https://docs.python.org/3/howto/logging.html>
- **SQLAlchemy — Using textual SQL (`text()`, bound parameters):** <https://docs.sqlalchemy.org/en/20/core/tutorial.html#using-textual-sql>
- **python-dotenv — README:** <https://pypi.org/project/python-dotenv/>
