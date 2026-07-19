# Lecture 2 — SQL Meets Python (pandas)

> **Duration:** ~2 hours. **Outcome:** you can pull a query into a pandas DataFrame with `read_sql`, clean and reshape it (`merge`, `pivot_table`, `melt`, `groupby`), and write results back to Postgres with `to_sql` — never touching a spreadsheet as a data store along the way.

SQL is unbeatable at one thing: filtering and summarizing rows that live in a database, using an engine built for exactly that. But some jobs are more naturally Python's: combining data from two different sources (a database query plus a CSV a partner sent you), iterating business logic that's awkward as pure SQL, or handing a result to a plotting or machine-learning library that expects a DataFrame. This lecture is about the **bridge** — moving cleanly between the two without ever "temporarily" dropping into a spreadsheet to do the reshaping by hand.

## 1. The bridge: SQLAlchemy engine

pandas doesn't talk to Postgres directly — it goes through a **SQLAlchemy engine**, a reusable connection object:

```python
import os
from sqlalchemy import create_engine
from dotenv import load_dotenv

load_dotenv()  # reads a .env file into environment variables — never hardcode credentials in the script

DB_URL = os.environ["DATABASE_URL"]  # e.g. postgresql+psycopg2://user:pass@localhost:5432/crunchcycles
engine = create_engine(DB_URL)
```

A `.env` file (never committed — add it to `.gitignore`) holds `DATABASE_URL=postgresql+psycopg2://localhost/crunchcycles`. Every script in this course reads its connection string from the environment, not from a literal string in the code — that's what lets the *same* script run against your laptop, a teammate's laptop, and a production database with zero code changes.

## 2. `read_sql` — a query becomes a DataFrame

```python
import pandas as pd

query = """
    SELECT o.order_id, o.order_date, o.status, c.company_name, r.region_name,
           oi.product_id, oi.quantity, oi.unit_price
    FROM orders o
    JOIN customers c    ON o.customer_id = c.customer_id
    JOIN regions r      ON c.region_id  = r.region_id
    JOIN order_items oi ON oi.order_id  = o.order_id
    WHERE o.status = 'Completed'
"""
orders_df = pd.read_sql(query, engine)
print(orders_df.shape)     # (rows, columns)
print(orders_df.dtypes)    # what pandas inferred for each column
orders_df.head()
```

`read_sql` runs the SQL, then hands you back a `DataFrame` — a 2-D table with labeled columns, index-aware, and a huge standard library of operations. **Do the filtering and joining in SQL, not pandas, when the data already lives in the database** — the database engine has indexes and a query planner; pulling the *entire* `order_items` table into Python and filtering it there is slower and wastes memory. Rule of thumb: **push filters and joins down to SQL; do everything after that — reshaping, iterative logic, combining with non-SQL sources — in pandas.**

## 3. Checking and fixing dtypes

SQL types don't always map to the pandas type you want:

```python
orders_df.dtypes
# order_id         int64
# order_date       object   <- came back as a string/date object, not a real datetime
# status           object
# company_name     object
# region_name      object
# product_id       int64
# quantity         int64
# unit_price      float64

orders_df["order_date"] = pd.to_datetime(orders_df["order_date"])
orders_df["line_total"] = orders_df["quantity"] * orders_df["unit_price"]
```

Two habits that save you real bugs later: (1) always check `.dtypes` right after a `read_sql` — a date that came back as text won't sort or filter the way you expect; (2) compute derived columns (`line_total`) explicitly and name them, the same discipline as `AS` in SQL.

## 4. Handling missing values

Remember `ship_date` is `NULL` for Pending/Cancelled orders — in pandas that becomes `NaT` (Not a Time) or `NaN` (Not a Number):

```python
full_orders = pd.read_sql("SELECT * FROM orders", engine, parse_dates=["order_date", "ship_date"])

full_orders["ship_date"].isna().sum()        # how many orders have no ship date
full_orders[full_orders["ship_date"].isna()] # look at them

# fill vs. drop are different decisions — make it on purpose:
full_orders["is_shipped"] = full_orders["ship_date"].notna()          # a clean boolean flag, keep the NaT
shipped_only = full_orders.dropna(subset=["ship_date"])               # or drop rows missing it entirely
```

`parse_dates=[...]` on `read_sql` is the cleaner way to fix the dtype problem from step 3 — ask pandas to parse the columns as it reads them, instead of fixing them after. **Never silently `fillna(0)` a date or an ID column** — decide row by row whether "missing" means "zero," "unknown," or "exclude this row," and say so in the code.

## 5. `merge` — pandas's join

If you already have two DataFrames (one from SQL, one from a CSV a partner emailed you, say), `merge` is pandas's join:

```python
products_df = pd.read_sql("SELECT product_id, product_name, category FROM products", engine)

enriched = orders_df.merge(products_df, on="product_id", how="left")
enriched[["order_id", "product_name", "category", "quantity", "unit_price"]].head()
```

`how=` maps directly onto SQL join types: `"inner"` (default), `"left"`, `"right"`, `"outer"` (full). `on="product_id"` is the join key — use `left_on=`/`right_on=` when the key has different names in each frame. **If the data already lives in the same database, do the join in SQL** — this is for combining a SQL result with something that *isn't* in the database yet.

## 6. `groupby` — pandas's `GROUP BY`

```python
region_revenue = (
    enriched
    .groupby("region_name")
    .agg(order_count=("order_id", "nunique"), revenue=("unit_price", lambda s: (enriched.loc[s.index, "quantity"] * s).sum()))
    .reset_index()
    .sort_values("revenue", ascending=False)
)
```

That lambda is clunky — which is exactly the lesson: **once line-level math is involved, compute the line total as a column first**, then aggregate the clean column:

```python
enriched["line_total"] = enriched["quantity"] * enriched["unit_price"]

region_revenue = (
    enriched
    .groupby("region_name")
    .agg(order_count=("order_id", "nunique"), revenue=("line_total", "sum"))
    .reset_index()
    .sort_values("revenue", ascending=False)
)
```

`nunique` is pandas's `COUNT(DISTINCT ...)` — same double-counting trap as Lecture 1 applies here: a plain `.size()` or `count()` on line-item-level rows counts line items, not orders.

## 7. `pivot_table` — turning long data into a grid

**Question:** "Give me a grid: rows = region, columns = month, values = revenue" — the shape a chart or a report usually wants.

```python
enriched["order_month"] = pd.to_datetime(enriched["order_date"]).dt.to_period("M").astype(str)

revenue_grid = enriched.pivot_table(
    index="region_name",
    columns="order_month",
    values="line_total",
    aggfunc="sum",
    fill_value=0,
)
revenue_grid
```

`pivot_table` is `GROUP BY` on two keys at once, reshaped so one key becomes rows and the other becomes columns — exactly the "wide" report a stakeholder expects to see, built from "long" (one-row-per-fact) data. `fill_value=0` turns "no orders that month" into an explicit `0` instead of a gap — the pandas equivalent of the `CROSS JOIN`-then-`LEFT JOIN` trick from Lecture 1.

## 8. `melt` — the reverse: wide back to long

If you receive (or produce) a wide grid and need it back in tidy, one-row-per-fact form for further analysis:

```python
tidy = revenue_grid.reset_index().melt(
    id_vars="region_name",
    var_name="order_month",
    value_name="revenue",
)
```

`melt` is the inverse of `pivot_table`. You'll reach for `pivot_table` when *producing* a report for a human, and `melt` when *consuming* a wide file (a lot of exported spreadsheets and legacy reports are wide) and needing it tidy again before you can `groupby` or `merge` it with anything else.

## 9. `to_sql` — writing a DataFrame back to the database

Once you've built a clean result, write it back — as a real table, not a spreadsheet:

```python
region_revenue.to_sql(
    "region_revenue_report",
    engine,
    if_exists="replace",   # or "append" — decide deliberately, see Lecture 3
    index=False,
)
```

`if_exists="replace"` drops and recreates the table every run (fine for a report table you always fully regenerate); `if_exists="append"` adds rows without touching existing ones (fine for an append-only log, wrong for a report you want to be idempotent). `index=False` stops pandas from writing its internal row-number index as a spurious column — you almost always want this off for report tables.

Now anyone with `psql` access — not just people who can run your Python script — can query the same report:

```sql
SELECT * FROM region_revenue_report ORDER BY revenue DESC;
```

That's the whole point of never using a spreadsheet as the store: the output lives in the same queryable, concurrent-safe, access-controlled database as everything else, instead of a `report_v3_FINAL.xlsx` sitting in someone's Downloads folder.

## 10. When to reach for which tool — a quick gut check

| Task | Prefer | Why |
|---|---|---|
| Filtering millions of rows down to thousands | SQL | Indexes + query planner beat a full Python scan |
| Joining tables that already live in the database | SQL | One round trip, the engine optimizes it |
| Combining a SQL result with a CSV/API response | pandas | The other source isn't in the database (yet) |
| Iterative or branching row-by-row logic | pandas | Awkward as a single SQL expression |
| Producing a wide report/chart-ready grid | pandas (`pivot_table`) | That's a display shape, not a storage shape |
| Something two teammates need to query independently | SQL (`to_sql` the result) | A DataFrame in your notebook is invisible to everyone else |

Challenge 1 this week makes you solve one problem both ways and time the difference yourself.

## 11. Check yourself

- Why push filtering into the SQL query instead of pulling the whole table and filtering in pandas?
- What does `parse_dates=[...]` on `read_sql` save you from doing manually?
- What's the pandas equivalent of `GROUP BY` with `COUNT(DISTINCT x)`?
- When would you choose `if_exists="append"` over `"replace"` on `to_sql`?
- In one sentence, what does `pivot_table` do that `groupby` alone doesn't?
- Why does writing a report with `to_sql` beat saving it as an `.xlsx`?

Lecture 3 turns everything so far into a script you (or a teammate, or a cron job) can re-run on demand.

## Further reading

- **pandas — `read_sql`:** <https://pandas.pydata.org/docs/reference/api/pandas.read_sql.html>
- **pandas — `DataFrame.to_sql`:** <https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.to_sql.html>
- **pandas — Merge, join, concatenate:** <https://pandas.pydata.org/docs/user_guide/merging.html>
- **pandas — Reshaping (`pivot_table`, `melt`):** <https://pandas.pydata.org/docs/user_guide/reshaping.html>
- **SQLAlchemy — Engine configuration:** <https://docs.sqlalchemy.org/en/20/core/engines.html>
