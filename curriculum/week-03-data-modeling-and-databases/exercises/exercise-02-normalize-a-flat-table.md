# Exercise 2 — Normalize a Flat Table to 3NF

**Goal:** Take a genuinely messy, redundant flat table and walk it through 1NF → 2NF → 3NF by hand, naming the specific anomaly each step removes — the exact procedure from Lecture 2, run on data you haven't seen before.

**Estimated time:** 1.5 hours.

## Setup

Load this flat table into your scratch database exactly as given — resist the urge to "fix" it before you start, the redundancy is the point:

```sql
CREATE TABLE orders_flat (
    order_id          INTEGER,
    order_date        DATE,
    customer_name     TEXT,
    customer_email    TEXT,
    customer_city     TEXT,
    customer_state    TEXT,
    product_sku       TEXT,
    product_name      TEXT,
    product_category  TEXT,
    unit_price        NUMERIC,
    quantity          INTEGER,
    sales_rep_name    TEXT,
    sales_rep_region  TEXT
);

INSERT INTO orders_flat VALUES
(5001, '2026-01-08', 'Grace Hopper', 'grace@example.com', 'Austin', 'TX', 'SKU-01', 'Wireless Mouse',   'Electronics', 24.99, 2, 'Don Draper', 'South'),
(5001, '2026-01-08', 'Grace Hopper', 'grace@example.com', 'Austin', 'TX', 'SKU-04', 'USB-C Cable',      'Electronics', 9.99,  1, 'Don Draper', 'South'),
(5002, '2026-01-09', 'Ada Lovelace', 'ada@example.com',   'Seattle','WA', 'SKU-01', 'Wireless Mouse',   'Electronics', 24.99, 1, 'Peggy Olson','West'),
(5003, '2026-01-10', 'Ada Lovelace', 'ada@example.com',   'Seattle','WA', 'SKU-07', 'Standing Desk',    'Furniture',   299.00,1, 'Peggy Olson','West'),
(5004, '2026-01-11', 'Alan Turing',  'alan@example.com',  'Austin', 'TX', 'SKU-04', 'USB-C Cable',      'Electronics', 9.99,  3, 'Don Draper', 'South'),
(5005, '2026-01-12', 'Grace Hopper', 'grace@example.com', 'Austin', 'TX', 'SKU-07', 'Standing Desk',    'Furniture',   299.00,1, 'Don Draper', 'South');
```

Confirm it loaded: `SELECT COUNT(*) FROM orders_flat;` should print `6`.

## Tasks

Write your work in `normalization.md`, one section per step.

### Step 1 — Find the redundancy and name the anomalies

Before touching the design, look at the data as it sits:

1. Which facts repeat across multiple rows that shouldn't have to? List at least three (e.g., "Grace Hopper's email appears on rows for order 5001 and 5005").
2. For each repeated fact, write a concrete **update anomaly** scenario: if that fact changed, what would go wrong? (e.g., "if Grace changes her email, we'd have to update it in 2 places — miss one and her account has two contradictory emails on file.")
3. Write one concrete **insertion anomaly** scenario: what fact can you *not* record using this table as it stands? (Think about what happens before a customer's first order.)
4. Write one concrete **deletion anomaly** scenario: what fact would be silently lost if you deleted a row?

### Step 2 — 1NF check

Is `orders_flat` already in 1NF? Check: does every column hold a single atomic value? Is every row uniquely identifiable? State your answer and reasoning. *(Hint: look closely at what `order_id` alone does and doesn't uniquely identify.)*

### Step 3 — Identify functional dependencies

List the functional dependencies you can find in this table, in the form `A → B`, the way Lecture 2 Section 2 did for the same domain shape. You should find dependencies rooted in at least three different "determiners" (i.e., not everything depends on the same column). Be specific — `customer_name → customer_email` is a real FD here; so is `product_sku → unit_price` (arguably — think about whether price is really a product-level fact or could vary by order, and state your assumption).

### Step 4 — Normalize to 2NF

What's the primary key of `orders_flat` once you fix the 1NF issue from Step 2? Given that key, which columns depend on only *part* of it? Split those out. Show the resulting table shapes (column lists, not full DDL yet).

### Step 5 — Normalize to 3NF

Of the tables from Step 4, which non-key columns depend on *another non-key column* rather than directly on their table's key (transitive dependencies)? Split those out too. Show your final set of table shapes — you should land on **4 tables**.

### Step 6 — Write the DDL

Translate your final 3NF design into real `CREATE TABLE` statements with primary keys and foreign keys (full constraint syntax is Exercise 3's focus, but at minimum get the keys right here). Run them. Then write `INSERT` statements that load the same 6 orders' worth of data into your new normalized tables and confirm the row counts make sense (e.g., how many distinct customers should you end up with, out of 6 order rows?).

## Done when…

- [ ] `normalization.md` names at least 3 update-anomaly scenarios, 1 insertion anomaly, and 1 deletion anomaly, each concrete and specific to this data (not generic definitions copied from the lecture).
- [ ] Functional dependencies are listed explicitly, `A → B` form, rooted in at least 3 different columns.
- [ ] You land on exactly 4 normalized tables, and can state in one sentence per table what its primary key is and why nothing in it violates 2NF or 3NF.
- [ ] The DDL runs without error and the seed data loads.
- [ ] You can answer: "if Grace Hopper's email changes, how many rows now need to be updated in your normalized design?" (The answer should be a much smaller number than in the flat table — say what it is.)

## Stretch

- Is `unit_price` really a pure product-level fact, or could the same SKU legitimately sell at different prices on different orders (a discount, a price change over time)? If you think it's order-line-level, not product-level, redo Step 5's split with that assumption instead and explain how the table shapes change.
- Write one query against your normalized tables that reconstructs something close to the original flat view (a `JOIN` across all 4 tables) — proof that normalizing didn't lose any information, just reorganized it.

## Submission

Commit `normalization.md` and your `.sql` file (DDL + inserts) to your portfolio under `c37-week-03/exercise-02/`.
