# Week 6 — Homework

Five problems, ~5 hours total, spread across the week. All SQL runs against the schema you loaded in the [README](./README.md) unless a problem says otherwise. Data tooling rule applies here too: every calculation happens in SQL or Python — no spreadsheets, even for the "sounds like a report" problems.

---

## Problem 1 — Module inventory for a real company (45 min)

Pick a real company you know something about — an employer, a family business, a company you've been a customer of. In `problem1.md`:

1. List every department/function it has (Sales, Ops, Finance, whatever applies).
2. For each, name the ERP/CRM module (from Lecture 1) it would map to, and one specific question that department asks daily that a system like that would need to answer.
3. Note one place where, from what you can observe as an outsider or employee, that company's systems visibly **don't** talk to each other well (a form you had to fill out twice, a support rep who couldn't see your order history, etc.). This is the "integration is the hard part" lesson made real.

---

## Problem 2 — Master data audit (60 min)

Run these three master-data-quality queries (adapted from Lecture 2) against the full Crunch Cycles schema, including this week's additions, and report the results in `problem2.md` with one sentence of interpretation each:

```sql
-- 1. Duplicate check across BOTH customers and suppliers
SELECT company_name, COUNT(*) FROM customers GROUP BY company_name HAVING COUNT(*) > 1;
SELECT supplier_name, COUNT(*) FROM suppliers GROUP BY supplier_name HAVING COUNT(*) > 1;

-- 2. Completeness: products with no preferred supplier
SELECT p.product_id, p.product_name
FROM products p
LEFT JOIN product_suppliers ps ON ps.product_id = p.product_id AND ps.is_preferred = TRUE
WHERE ps.product_id IS NULL;

-- 3. Consistency: orders referencing a customer whose region_id doesn't match any row in regions
SELECT o.order_id, c.customer_id, c.region_id
FROM orders o
JOIN customers c ON c.customer_id = o.customer_id
LEFT JOIN regions r ON r.region_id = c.region_id
WHERE r.region_id IS NULL;
```

All three should currently return zero rows (this week's seed data is clean) — that's expected. Write one sentence per query explaining, in your own words, *what kind* of real-world problem it would have caught if it hadn't been clean.

---

## Problem 3 — Order-to-cash and procure-to-pay, side by side (75 min)

In `problem3.sql`, write:

1. A query showing, for each **region**, total sales revenue this year (`orders`/`order_items`) and total procurement spend this year (`purchase_orders`/`purchase_order_items` — note: purchase orders aren't tied to a region in this schema; join through the buying `employee_id`'s `region_id` instead, and handle the one employee, Maria, with a `NULL` region explicitly).
2. A single combined query (`UNION ALL` with a `flow` column labeled `'sales'` or `'purchase'`) that lists every dollar amount moving in either direction, sorted by date, so you can eyeball cash flow direction over time.
3. In `problem3.md`, two sentences: which region generates the most net revenue (sales minus attributable procurement), and one caveat about why this "net revenue by region" number is a rough approximation, not a real P&L (hint: think about what costs this schema doesn't capture — payroll, overhead, shipping).

---

## Problem 4 — CRM pipeline health check (60 min)

Northstar-style thinking, applied to Crunch Cycles' own pipeline. In `problem4.sql`:

1. Compute the **win rate** — `COUNT(Won) / COUNT(Won + Lost)` — excluding still-open deals, as a percentage.
2. Compute **average days to close** for Won and Lost deals separately (`closed_date - opened_date`).
3. List every rep with at least one opportunity, their win rate individually, sorted worst to best.
4. In `problem4.md`, one paragraph: based only on these numbers, would you flag any single rep for a coaching conversation? What additional data (not in this schema) would you want before actually having that conversation — this is a chance to practice not over-trusting a small, incomplete dataset.

---

## Problem 5 — Design a data-ownership policy (60 min)

No SQL for this one. In `problem5.md`, write a short (400–500 word) internal policy document for Crunch Cycles, modeled on Lecture 2 section 4's ownership table, covering:

1. Who owns `customers` (which fields, if you want to split billing vs. relationship ownership the way the lecture described).
2. Who owns `products` (pricing vs. specs, if those should be split).
3. Who owns `suppliers`.
4. One **rule** for how conflicts get resolved when two teams disagree about a master-data value (e.g., an escalation path, a designated tie-breaker role).
5. One **process control** that would have prevented Challenge 2's duplicate-customer problem from happening in the first place, that you'd actually propose rolling out.

---

## Time budget

| Problem | Time |
|--------:|----:|
| 1 | 45 min |
| 2 | 60 min |
| 3 | 75 min |
| 4 | 60 min |
| 5 | 60 min |
| **Total** | **~5 h** |

After homework, take the [quiz](./quiz.md) and ship the [mini-project](./mini-project/README.md).
