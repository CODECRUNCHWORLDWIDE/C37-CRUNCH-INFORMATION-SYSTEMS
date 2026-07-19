# Exercise 2 — Build an ELT Job into the Warehouse

**Goal:** build `elt_warehouse_load.py`, a real, idempotent ELT job that extracts every table from `crunchcycles` (operational), loads it untouched into `crunchcycles_dw.raw`, then runs SQL transforms *inside* the warehouse to populate `crunchcycles_dw.warehouse`'s star schema from Lecture 2. Run it twice in a row and prove nothing duplicates.

**Estimated time:** 2 hours.

## Setup

Confirm both databases exist and are reachable (from this week's [README setup](../README.md#setup)):

```bash
psql crunchcycles -c "SELECT COUNT(*) FROM orders;"        # expect 40
psql crunchcycles_dw -c "\dn"                                # expect raw, warehouse
```

Create the `warehouse` schema tables from Lecture 2 (`dim_date`, `dim_customer`, `dim_product`, `dim_employee`, `fact_order_items`) in `crunchcycles_dw` now, if you haven't already — this exercise populates them, it doesn't define them.

## Tasks

**1. Create the `raw` landing tables.** One per operational table you need, each with a `loaded_at` timestamp, mirroring the source structure (no transformation yet — that's the whole point of `raw`):

```sql
CREATE TABLE raw.customers (
    customer_id   INTEGER, company_name TEXT, contact_name TEXT, email TEXT,
    city TEXT, country TEXT, region_id INTEGER, signup_date DATE,
    loaded_at     TIMESTAMP NOT NULL DEFAULT now()
);
CREATE TABLE raw.regions   (region_id INTEGER, region_name TEXT, loaded_at TIMESTAMP NOT NULL DEFAULT now());
CREATE TABLE raw.employees (employee_id INTEGER, first_name TEXT, last_name TEXT, email TEXT,
                             title TEXT, region_id INTEGER, hire_date DATE, manager_id INTEGER,
                             loaded_at TIMESTAMP NOT NULL DEFAULT now());
CREATE TABLE raw.products  (product_id INTEGER, product_name TEXT, category TEXT,
                             unit_price NUMERIC, unit_cost NUMERIC, discontinued BOOLEAN,
                             loaded_at TIMESTAMP NOT NULL DEFAULT now());
CREATE TABLE raw.orders    (order_id INTEGER, customer_id INTEGER, employee_id INTEGER,
                             order_date DATE, ship_date DATE, status TEXT,
                             loaded_at TIMESTAMP NOT NULL DEFAULT now());
CREATE TABLE raw.order_items (order_id INTEGER, product_id INTEGER, quantity INTEGER,
                                unit_price NUMERIC, loaded_at TIMESTAMP NOT NULL DEFAULT now());
```

**2. Write the Extract + Load step (Python).** Pull each operational table into a DataFrame with `pandas.read_sql`, then write it to `raw` — but not with a plain append: each run should **replace** the raw landing tables' contents (raw is a mirror of "current source state," not an accumulating history) rather than append duplicates every run:

```python
# elt_warehouse_load.py — Part 1: Extract + Load into raw
import logging
import pandas as pd
from sqlalchemy import create_engine, text

logging.basicConfig(level=logging.INFO, format="%(asctime)s %(levelname)s %(message)s")
log = logging.getLogger("elt_warehouse_load")

oltp_engine = create_engine("postgresql://localhost/crunchcycles")
dw_engine   = create_engine("postgresql://localhost/crunchcycles_dw")

SOURCE_TABLES = ["regions", "employees", "customers", "products", "orders", "order_items"]

def extract_and_load_raw():
    for table in SOURCE_TABLES:
        df = pd.read_sql(f"SELECT * FROM {table}", oltp_engine)
        df.to_sql(table, dw_engine, schema="raw", if_exists="replace", index=False)
        log.info("Loaded raw.%s — %d rows", table, len(df))
```

`if_exists="replace"` is the idempotency mechanism for the raw layer specifically — every run fully replaces `raw`'s contents with the operational store's *current* state, so running it twice in a row leaves `raw` identical either way (the source data hasn't changed between your two runs). This is different from the fact table's idempotency mechanism in Task 4 — note that difference; it's intentional and Lecture 1 §5 explains why raw's job is to be a disposable mirror, not an accumulating history.

**3. Write the Transform step, in SQL, run from Python.** This is the "T" of ELT happening *inside the destination*. Truncate-and-reload each dimension (dimensions are small; full reload each run is simpler and correct at this scale):

```python
# Part 2: Transform raw -> warehouse (dimensions)
DIM_CUSTOMER_SQL = """
TRUNCATE warehouse.dim_customer RESTART IDENTITY CASCADE;
INSERT INTO warehouse.dim_customer
    (customer_id, company_name, contact_name, city, country, region_name, signup_date)
SELECT c.customer_id, c.company_name, c.contact_name, c.city, c.country, r.region_name, c.signup_date
FROM raw.customers c
JOIN raw.regions r ON c.region_id = r.region_id;
"""

DIM_PRODUCT_SQL = """
TRUNCATE warehouse.dim_product RESTART IDENTITY CASCADE;
INSERT INTO warehouse.dim_product
    (product_id, product_name, category, current_price, unit_cost, discontinued)
SELECT product_id, product_name, category, unit_price, unit_cost, discontinued
FROM raw.products;
"""

DIM_EMPLOYEE_SQL = """
TRUNCATE warehouse.dim_employee RESTART IDENTITY CASCADE;
INSERT INTO warehouse.dim_employee
    (employee_id, full_name, title, region_name, hire_date)
SELECT e.employee_id, e.first_name || ' ' || e.last_name, e.title, r.region_name, e.hire_date
FROM raw.employees e
LEFT JOIN raw.regions r ON e.region_id = r.region_id;   -- LEFT: HQ roles have NULL region_id
"""

def transform_dimensions():
    with dw_engine.begin() as conn:
        for label, sql in [("dim_customer", DIM_CUSTOMER_SQL),
                            ("dim_product", DIM_PRODUCT_SQL),
                            ("dim_employee", DIM_EMPLOYEE_SQL)]:
            conn.execute(text(sql))
            log.info("Rebuilt warehouse.%s", label)
```

`TRUNCATE ... RESTART IDENTITY CASCADE` before each `INSERT` is the dimensions' idempotency mechanism: full rebuild, every run, from `raw`'s current state — running twice in a row leaves the same rows with the same (reset) surrogate keys. `RESTART IDENTITY` resets the `SERIAL` counter so surrogate keys don't climb forever across runs; `CASCADE` is required because `fact_order_items` has foreign keys pointing at these dimension tables — think carefully about *ordering*: dimensions must be rebuilt (and the fact table reloaded) together, in the same transaction or same script run, never dimensions today and fact next week, or foreign keys will point at surrogate keys that no longer mean the same thing.

**4. Write the fact table transform — this one uses upsert, not truncate-and-reload.** Unlike the dimensions, `fact_order_items` should grow incrementally as new orders come in, without regenerating surrogate keys the fact table already references:

```python
FACT_ORDER_ITEMS_SQL = """
INSERT INTO warehouse.fact_order_items
    (order_id, product_id, order_status, date_key, customer_key, employee_key, product_key,
     quantity, unit_price, unit_cost, extended_price, extended_cost, gross_profit)
SELECT
    o.order_id,
    oi.product_id,
    o.status,
    CAST(TO_CHAR(o.order_date, 'YYYYMMDD') AS INTEGER)   AS date_key,
    dc.customer_key,
    de.employee_key,
    dp.product_key,
    oi.quantity,
    oi.unit_price,
    dp.unit_cost,
    oi.quantity * oi.unit_price                          AS extended_price,
    oi.quantity * dp.unit_cost                            AS extended_cost,
    (oi.quantity * oi.unit_price) - (oi.quantity * dp.unit_cost) AS gross_profit
FROM raw.orders o
JOIN raw.order_items oi ON o.order_id = oi.order_id
JOIN warehouse.dim_customer dc ON o.customer_id = dc.customer_id
JOIN warehouse.dim_employee de ON o.employee_id = de.employee_id
JOIN warehouse.dim_product  dp ON oi.product_id = dp.product_id
ON CONFLICT (order_id, product_id) DO UPDATE SET
    order_status   = EXCLUDED.order_status,     -- status can change (e.g. Pending -> Completed)
    quantity       = EXCLUDED.quantity,
    unit_price     = EXCLUDED.unit_price,
    unit_cost      = EXCLUDED.unit_cost,
    extended_price = EXCLUDED.extended_price,
    extended_cost  = EXCLUDED.extended_cost,
    gross_profit   = EXCLUDED.gross_profit;
"""

def transform_fact():
    with dw_engine.begin() as conn:
        result = conn.execute(text(FACT_ORDER_ITEMS_SQL))
        log.info("Upserted fact_order_items")

def run():
    log.info("ELT run starting")
    extract_and_load_raw()
    transform_dimensions()      # must run BEFORE the fact load — fact FKs depend on fresh surrogate keys
    transform_fact()
    log.info("ELT run complete")

if __name__ == "__main__":
    run()
```

`ON CONFLICT (order_id, product_id) DO UPDATE` is exactly Week 7's idempotency pattern, applied to the fact table: an order that was `Pending` last run and is `Completed` this run updates in place instead of creating a duplicate fact row. Note the ordering dependency called out in the comment — because dimensions get `TRUNCATE ... RESTART IDENTITY`'d and rebuilt every run, the fact transform *must* run after, in the same script execution, using the surrogate keys freshly assigned this run — never a stale set from a previous run.

## Expected result (spot checks)

```bash
python3 elt_warehouse_load.py
psql crunchcycles_dw -c "SELECT COUNT(*) FROM warehouse.fact_order_items;"   # expect 54
psql crunchcycles_dw -c "SELECT COUNT(*) FROM warehouse.dim_customer;"       # expect 18

python3 elt_warehouse_load.py   # run it again, immediately
psql crunchcycles_dw -c "SELECT COUNT(*) FROM warehouse.fact_order_items;"   # still 54 — not 108
psql crunchcycles_dw -c "SELECT COUNT(*) FROM warehouse.dim_customer;"       # still 18
```

If either count changed between the two runs, your idempotency mechanism has a gap — go back to Task 3 or 4 and find where a run isn't cleanly reproducible.

## Done when…

- [ ] `elt_warehouse_load.py` runs end to end with one command and logs each step's row counts.
- [ ] Running it twice in a row produces **identical** row counts in every `warehouse.*` table both times.
- [ ] `dim_employee`'s `LEFT JOIN` correctly gives HQ-role employees (no `region_id`) a `NULL region_name` instead of dropping them from the dimension entirely.
- [ ] Every fact row's `extended_price`/`extended_cost`/`gross_profit` are internally consistent (`extended_price - extended_cost = gross_profit` for every row — spot-check with a `SELECT` that flags any row where it isn't).

## Stretch

- Add a `raw_load_log` table in `crunchcycles_dw` that records, per run: a timestamp, which tables were loaded, and their row counts — a lightweight version of the observability Week 7 Lecture 3 taught for the `fx_rates` ETL job.
- Change one order's `status` in `crunchcycles` from `Pending` to `Completed` (pick one of orders 8, 19, or 33), re-run the ELT job, and confirm `warehouse.fact_order_items` reflects the new status for that row without creating a duplicate — proof the upsert path in Task 4 handles a genuinely changed source row, not just a re-run of unchanged data.

## Submission

Commit `elt_warehouse_load.py` and the `raw`/`warehouse` `CREATE TABLE` statements to your portfolio under `c37-week-10/exercise-02/`.
