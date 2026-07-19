# Exercise 3 — Parameterize a Report by Date Range

**Goal:** turn the region-revenue query from Exercises 1–2 into a small script that takes a date range on the command line, so "run the Q1 report" and "run the report for just March" are the same script, different arguments — no hand-editing SQL.

**Estimated time:** 90 minutes.

## Setup

You'll extend the `.env`-based connection pattern from Exercise 2. Create `region_report.py`.

## Starter code

```python
import os
import argparse
import pandas as pd
from sqlalchemy import create_engine, text
from dotenv import load_dotenv


def get_engine():
    load_dotenv()
    return create_engine(os.environ["DATABASE_URL"])


def extract(engine, start_date: str, end_date: str) -> pd.DataFrame:
    query = text("""
        SELECT o.order_id, o.order_date, r.region_name, oi.quantity, oi.unit_price
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
    raw["line_total"] = raw["quantity"] * raw["unit_price"]
    return (
        raw.groupby("region_name")
        .agg(order_count=("order_id", "nunique"), revenue=("line_total", "sum"))
        .reset_index()
        .sort_values("revenue", ascending=False)
    )


# your job: fill in main() below
```

## Tasks

1. **Wire up `argparse`.** Add `--start` and `--end` as required string arguments (`YYYY-MM-DD`). Write `main()` so it calls `extract`, then `transform`, then prints the result with `print(report.to_string(index=False))`.

2. **Test with the full window.** Run:

   ```bash
   python region_report.py --start 2024-01-01 --end 2024-07-01
   ```

   *Expected: the same 4 rows as Exercise 1 Q7 / Exercise 2 Task 4 — North America $18,797.00 / 15, Europe $13,287.00 / 9, APAC $10,619.00 / 5, LATAM $9,416.00 / 6.*

3. **Test with a narrower window — Q1 only.** Run:

   ```bash
   python region_report.py --start 2024-01-01 --end 2024-04-01
   ```

   *Expected: revenue should be higher for the first three months than any single quarter after it (recall from Exercise 1 Q9 that Jan+Feb+Mar = $14,165 + $12,098 + $10,476 = $36,739 combined, vs. the full 6-month total of $52,119). Compute each region's Q1-only revenue and sanity-check it's less than that region's full-6-month number from Task 2, but still a meaningful share of it.*

4. **Test an edge case — a month with no LATAM sales.** Run the report for **June only** (`--start 2024-06-01 --end 2024-07-01`).
   *Expected: only regions with June activity appear. From Exercise 2's pivot grid, June had revenue only in North America ($2,833). Does your report show 1 row or 4? If your `groupby` only shows regions that appear in the filtered data, that's correct — but write one sentence in a comment explaining why Europe, APAC, and LATAM don't show up as `$0` rows instead of not showing up at all (hint: nothing in `raw` ever mentions them for that date range — there's no row to group).*

5. **Add `--table` and write it.** Add an optional `--table` argument (default `"region_revenue_report"`), and a `load()` function using `to_sql(..., if_exists="replace", index=False)`. Confirm with `psql`/`sqlite3` that the table exists and holds the right numbers after running the script.

6. **Guard against a bad range.** What happens right now if someone runs `--start 2024-07-01 --end 2024-01-01` (end before start)? Add a check in `main()` that raises a clear `ValueError` if `end <= start`, before ever querying the database.

## Expected result (spot checks)

| Run | Rows | Top region | Top revenue |
|---|---:|---|---:|
| Full 6 months (`2024-01-01`→`2024-07-01`) | 4 | North America | $18,797.00 |
| Q1 only (`2024-01-01`→`2024-04-01`) | 4 | North America | $12,218.00 *(Jan+Feb+Mar for NA — recompute from Exercise 2's pivot grid: 5154+3584+3480)* |
| June only (`2024-06-01`→`2024-07-01`) | 1 | North America | $2,833.00 |

## Done when…

- [ ] The script runs from the command line with `--start`/`--end` and no other edits.
- [ ] All three spot-check runs above match.
- [ ] Running the same command twice in a row does **not** double the numbers in the `region_revenue_report` table (idempotency — see Lecture 3 §3).
- [ ] The end-before-start case raises a clear error instead of silently returning an empty or wrong report.
- [ ] No f-string ever builds SQL text with `start`/`end` spliced directly in — only bound `:params`.

## Stretch

- Add a `--status` argument (default `"Completed"`) so the same script can report on `Pending` or `Cancelled` orders too, without touching the query.
- Log each run with the `logging` module (Lecture 3 §5) instead of `print` — include the row count extracted and the row count written.
- Wrap `extract` + `transform` + `load` in a `run(start, end, table)` function with a `try`/`except` that logs a clear error and exits with a non-zero status code on failure — this is what makes the script safe to hand to `cron` in Challenge 2.

## Submission

Commit `region_report.py` to your portfolio under `c37-week-04/exercise-03/`.
