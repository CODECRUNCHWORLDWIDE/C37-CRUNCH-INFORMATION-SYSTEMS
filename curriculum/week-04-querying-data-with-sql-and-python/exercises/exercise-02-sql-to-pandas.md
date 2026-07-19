# Exercise 2 — Load a Query into pandas and Reshape It

**Goal:** pull a joined query into a DataFrame, fix its dtypes, compute a derived column, then reshape it two ways — `groupby` (matches Exercise 1 Q7) and `pivot_table` (matches Exercise 1 Q6's grid).

**Estimated time:** 90 minutes.

## Setup

```bash
python3 -m venv .venv && source .venv/bin/activate
pip install pandas sqlalchemy psycopg2-binary python-dotenv
```

Create a `.env` file (not committed) with `DATABASE_URL=postgresql+psycopg2://localhost/crunchcycles` (adjust user/host as needed for your setup). Create `explore.py` for your work.

## Starter code

```python
import os
import pandas as pd
from sqlalchemy import create_engine
from dotenv import load_dotenv

load_dotenv()
engine = create_engine(os.environ["DATABASE_URL"])

query = """
    SELECT o.order_id, o.order_date, r.region_name, oi.quantity, oi.unit_price
    FROM orders o
    JOIN customers c    ON o.customer_id = c.customer_id
    JOIN regions r      ON c.region_id  = r.region_id
    JOIN order_items oi ON oi.order_id  = o.order_id
    WHERE o.status = 'Completed'
"""
df = pd.read_sql(query, engine)
```

## Tasks

1. **Check dtypes.** Print `df.dtypes`. Is `order_date` a real datetime, or a string/object? Fix it with `pd.to_datetime` (or re-run `read_sql` with `parse_dates=["order_date"]` — try both, note which is fewer lines).
   *Expected: `df.shape` is `(49, 5)` — 49 line items across the 35 Completed orders (14 of those orders have 2 line items, 21 have 1 — that's `14*2 + 21*1 = 49`). The full `order_items` table has 54 rows total (Exercise 1); this query already filtered out the 5 line items belonging to Pending/Cancelled orders before the data ever reached pandas — a good check that your filter is happening in SQL, not by accident in Python.*

2. **Compute `line_total`.** Add a column `line_total = quantity * unit_price`. Confirm `df["line_total"].sum()` equals the grand total revenue from Exercise 1 Q7.
   *Expected: `52119.00` (allow for float rounding — compare with `round(df["line_total"].sum(), 2)`).*

3. **Add `order_month`.** Add a column `order_month = df["order_date"].dt.to_period("M").astype(str)` (values like `"2024-01"`).

4. **`groupby` — reproduce Exercise 1 Q7.** Group by `region_name`, aggregate `order_count = ("order_id", "nunique")` and `revenue = ("line_total", "sum")`, sort by revenue descending.
   *Expected: same 4 rows and numbers as Exercise 1 Q7 — North America $18,797.00 / 15 orders, Europe $13,287.00 / 9, APAC $10,619.00 / 5, LATAM $9,416.00 / 6. If your numbers don't match Exercise 1's, you have a bug — go find it before continuing.*

5. **`pivot_table` — reproduce Exercise 1 Q6's grid, with real values.** Build `index="region_name", columns="order_month", values="line_total", aggfunc="sum", fill_value=0`.
   *Expected grid (rows = region, columns = Jan–Jun 2024):*

   | region | 2024-01 | 2024-02 | 2024-03 | 2024-04 | 2024-05 | 2024-06 |
   |---|---:|---:|---:|---:|---:|---:|
   | North America | 5154 | 3584 | 3480 | 2847 | 899 | 2833 |
   | Europe | 2396 | 3647 | 2698 | 3948 | 598 | 0 |
   | APAC | 2799 | 1934 | 4298 | 0 | 1588 | 0 |
   | LATAM | 3816 | 2933 | 0 | 2018 | 649 | 0 |

   Every column and every row should sum to the same region/month totals you already verified in Exercise 1 Q7 and Q9 — a built-in double-check.

6. **`melt` it back.** Take the pivoted grid from Task 5, `reset_index()`, then `melt(id_vars="region_name", var_name="order_month", value_name="revenue")`. Confirm `revenue.sum()` still equals `52119.00` and the row count is `24` (4 regions × 6 months) — matching Exercise 1 Q6's `CROSS JOIN` grid exactly, but now derived from real data instead of hypothetical combinations.

7. **`merge` in the product name.** Separately, `pd.read_sql("SELECT product_id, product_name, category FROM products", engine)`, then re-run your original query with `oi.product_id` included, and `merge` the product info in on `product_id`. Which `category` had the highest total `line_total`?
   *Expected: "Mountain Bike" — check your merged-and-grouped result to confirm the number.*

## Done when…

- [ ] `df["order_date"]` is a real `datetime64` dtype, not `object`.
- [ ] `df["line_total"].sum()` rounds to `52119.00`.
- [ ] Your `groupby` result matches Exercise 1 Q7 exactly (row order, counts, and revenue).
- [ ] Your `pivot_table` matches the grid above exactly, including the `0`s (not `NaN` — `fill_value=0` is doing its job).
- [ ] Your `melt`ed frame has exactly 24 rows and sums back to `52119.00`.

## Stretch

- Instead of `fill_value=0`, drop it and see what appears in the empty cells (`NaN`). In one sentence, when would `NaN` (missing) be more honest than `0` (a fact) for a business report?
- Write the pivoted grid to a CSV with `df.to_csv("region_month_revenue.csv", index=True)` and open it — this is what a stakeholder-ready export looks like, produced with zero manual copy-paste.
- Compute the same `groupby` region/revenue result using **only** SQL (a single query with `JOIN` + `GROUP BY`), and compare the code length and readability to the pandas version. You'll do this properly in Challenge 1.

## Submission

Commit `explore.py` to your portfolio under `c37-week-04/exercise-02/`.
