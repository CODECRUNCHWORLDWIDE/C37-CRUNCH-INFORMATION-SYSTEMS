# Week 4 — Homework

Extra practice, spaced across the week. Do these **after** the matching lecture, not all at once at the end — the spacing is what makes the join/aggregation instincts stick. Everything runs against the `crunchcycles` schema from the [week README](./README.md#setup-load-the-week-3-schema).

**Estimated time:** ~5 hours total, spread across the week.

---

## Set A — Joins (after Lecture 1 / Exercise 1)

Work in a `homework.sql` file, one query per task under a `-- A<n>` comment.

**A1.** List every **Pending** order with the customer's `company_name`, the rep's name, and the order date. *(Expected: 3 rows — orders 8, 19, 33.)*

**A2.** List every **Cancelled** order the same way. *(Expected: 2 rows — orders 13, 25.)*

**A3.** For each region, count **distinct customers who have ever ordered** (any status) vs. the region's **total customer count**. You'll need two aggregations joined or unioned together — a `LEFT JOIN` from `regions` outward, or two subqueries. *(Hint: every region has 100% of its customers with at least one order except Europe — Fjord Cycling Supply never ordered.)*

**A4.** List each product alongside the **total quantity ever sold** (any order status) and the **total quantity sold in Completed orders only** — two separate columns, side by side, so a difference between them is visible at a glance. *(Think about what two different `SUM(CASE WHEN ...)` expressions, or two joins with different filters, would look like.)*

**A5.** Self-join challenge: for every pair of employees who share the same `region_id` (excluding an employee paired with themselves), list both names once per pair. *(Hint: `e1.emp_id < e2.emp_id` in the join condition avoids both the self-pair and listing each pair twice.)*

**A6.** Which employees have **never** managed anyone (i.e., no other employee has their `emp_id` as `manager_id`)? Use a `LEFT JOIN` anti-pattern on the self-join. *(Expected: everyone except Maria — she's the only manager on record.)*

---

## Set B — Aggregation & CTEs (after Lecture 1 / Exercise 1)

**B1.** For **Completed** orders, compute each employee's total revenue and order count, sorted highest revenue first. Which rep brought in the most revenue?

**B2.** Using `HAVING`, find any region where **average order value** (`revenue / order_count`, computed per region) for Completed orders exceeds $1,300. *(Careful: you can't `HAVING SUM(...)/COUNT(...) > 1300` directly in every engine's syntax the same way — try it, and if it doesn't work as written, compute it in a CTE first, then filter in the outer query.)*

**B3.** Using `WITH`, build a two-step query: first compute each customer's total Completed revenue, then classify each customer as `'VIP'` (revenue > $3,000), `'Regular'` (revenue between $500 and $3,000), or `'New/Low'` (< $500 or no orders) using a `CASE` expression in the outer query. Count how many customers fall in each tier.

---

## Set C — SQL to pandas (after Lecture 2 / Exercise 2)

Work in `homework.py`.

**C1.** Read the full `products` table into a DataFrame. Add a column `margin_pct = (unit_price - unit_cost) / unit_price * 100`, rounded to 1 decimal. Which product has the **highest** margin percentage? Which has the **lowest**?

**C2.** Read Completed order line items joined with `products` (category included). Build a `pivot_table`: rows = `category`, columns = `region_name`, values = `line_total` (`quantity * unit_price`), `aggfunc="sum"`, `fill_value=0`. Which `(category, region)` cell has the single highest value?

**C3.** Using the same joined DataFrame from C2, compute total revenue per category with `groupby`, then **separately** write the exact same aggregation as one SQL query. Confirm the two match. Note in a comment which version you found easier to write and why.

---

## Set D — Pipelines (after Lecture 3 / Exercise 3)

**D1.** Take Exercise 3's `region_report.py` and add a `--min-revenue` argument that filters the **output** (not the underlying query) to only regions above that revenue threshold, using pandas (`report[report["revenue"] >= args.min_revenue]`), after the `transform` step. Test it with a threshold that excludes at least one region.

**D2.** Add a second report function, `top_products(engine, start_date, end_date, limit=5)`, that returns the top N products by revenue for a date range, reusing the `extract`-style pattern. Wire it into the same script behind a `--report {regional,products}` argument so one script can produce either report on demand.

**D3.** Write one paragraph: if this script's database connection failed halfway through `load()` — after writing `report_regional_summary` but before finishing `report_top_customers` — what state would the database be left in, and is that a problem? What would you change (hint: a single transaction wrapping all the `to_sql` calls) to make partial failure safer?

---

## Submission

Commit `homework.sql` and `homework.py` to your portfolio under `c37-week-04/homework/`, with brief comments answering the "which/what" questions inline (A3, A6, B1, C1, C2, D3).
