# Exercise 1 — Answer 10 Business Questions with Joins

**Goal:** work through every join type from Lecture 1 — `INNER`, `LEFT` (twice, as an anti-join), `FULL OUTER`, self-join, `CROSS`, plus `GROUP BY`/`HAVING`/`WITH` — each attached to a real question about Crunch Cycles.

**Estimated time:** 90 minutes.

## Setup

Confirm the schema is loaded:

```sql
SELECT COUNT(*) FROM orders;       -- must print 40
SELECT COUNT(*) FROM order_items;  -- must print 54
```

Create `solutions.sql` and answer each question under a `-- Q<n>: <question>` comment.

## The 10 questions

**Q1. INNER JOIN.** List every order's `order_id`, the customer's `company_name`, and the rep's full name, sorted by `order_id`.
*Expected: 40 rows — every order has a valid customer and employee (enforced by the foreign keys).*

**Q2. LEFT JOIN anti-join.** Which customers have **never** placed an order (any status)?
*Expected: 1 row — Fjord Cycling Supply (customer 18, signed up Jan 2024, no orders since).*

**Q3. LEFT JOIN anti-join.** Which sales reps have **zero** orders on record?
*Expected: 2 rows — Maria Chen (the Sales Director — expected, she doesn't carry a book of business) and Priya Nair (a Sales Rep — a real staffing question worth flagging).*

**Q4. FULL OUTER JOIN.** Join `customers` to `orders` on `customer_id`, keeping unmatched rows from **both** sides. How many total rows come back, and which row(s) are unmatched?
*Expected: 41 rows — 40 matched (one per order) plus 1 unmatched customer (Fjord, from Q2). Zero unmatched orders, because every order's `customer_id` is guaranteed valid by the foreign key. **SQLite note:** there's no native `FULL OUTER JOIN` — emulate it with `LEFT JOIN ... UNION ... LEFT JOIN` (swap table order on the second half) and dedupe.*

**Q5. Self-join.** List every employee next to their manager's name (use `LEFT JOIN`, not `INNER` — think about why before you write it).
*Expected: 8 rows. Maria Chen's `manager` column is `NULL` (she has no manager) — an `INNER JOIN` here would silently drop her row entirely.*

**Q6. CROSS JOIN.** Build a grid of every `(region, month)` pair for Jan–Jun 2024 — one row per combination, whether or not anything sold that month in that region.
*Expected: 24 rows (4 regions × 6 months). This is the "fill the gaps" pattern from Lecture 1 §5 — you'll reuse it in Exercise 2's `pivot_table`.*

**Q7. GROUP BY aggregation.** For **Completed** orders only, show each region's order count (`COUNT(DISTINCT order_id)`, not a plain `COUNT(*)` — think about why) and total revenue (`quantity * unit_price` from `order_items`), ranked highest revenue first.
*Expected 4 rows, in this order: North America — 15 orders, $18,797.00 revenue; Europe — 9 orders, $13,287.00; APAC — 5 orders, $10,619.00; LATAM — 6 orders, $9,416.00. (Grand total: $52,119.00 — a useful checksum.)*

**Q8. HAVING.** Which customers placed **more than 2** completed orders? Show `company_name` and their order count, highest first.
*Expected: 7 customers — Trailhead Bikes (4), Pacific Wheels (4), Northern Gear Co (3), Rocky Mountain Riders (3), Alpine Sport Supply (3), Le Velo Parisien (3), Tokyo Pedal Works (3).*

**Q9. CTE.** Using `WITH`, compute total revenue from **Completed** orders per calendar month for Jan–Jun 2024, one row per month.
*Expected 6 rows: Jan $14,165.00, Feb $12,098.00, Mar $10,476.00, Apr $8,813.00, May $3,734.00, Jun $2,833.00. (Notice revenue is trending down month over month — that's a real signal worth flagging in a report, not a data error. Keep this number; the mini-project asks you to investigate it.)*

**Q10. LEFT JOIN anti-join.** Which products have **never** appeared in an order (any status)?
*Expected: 1 row — Crunch Trail 50 (product 12), which is also the one product marked `discontinued = TRUE`. Consistent, and worth a one-line note in a real report: was it discontinued *because* it never sold, or did it never sell *because* it was discontinued? The data alone can't tell you — say so.*

## Expected result (spot checks)

| Q | Row count | Key number to verify |
|---|---:|---|
| 1 | 40 | — |
| 2 | 1 | Fjord Cycling Supply |
| 3 | 2 | Maria Chen, Priya Nair |
| 4 | 41 | 1 unmatched (customer side) |
| 5 | 8 | Maria's manager is `NULL` |
| 6 | 24 | 4 × 6 |
| 7 | 4 | North America highest at $18,797.00 |
| 8 | 7 | Trailhead Bikes and Pacific Wheels tied at 4 |
| 9 | 6 | Total across months = $52,119.00 |
| 10 | 1 | Crunch Trail 50 |

## Done when…

- [ ] `solutions.sql` has all 10 queries under `-- Q<n>` comments.
- [ ] Every row count in your output matches the table above.
- [ ] Q4 and Q5 use `LEFT`/`FULL OUTER`, not `INNER` — and you can say in one sentence why an `INNER` would be wrong there.
- [ ] Q7 uses `COUNT(DISTINCT order_id)`, not `COUNT(*)` — and you can explain what `COUNT(*)` would have counted instead (line items, not orders).
- [ ] Q9 is written with `WITH`, not a bare subquery.

## Stretch

- Rewrite Q2 and Q10 using `NOT EXISTS` instead of `LEFT JOIN ... WHERE ... IS NULL`. Confirm you get the same rows.
- Extend Q9 to also show each month's change from the prior month (a hint: `LAG(revenue) OVER (ORDER BY month)` — Week 10 covers window functions properly; this is a preview).
- Add an 11th query: for each region, the single highest-revenue **order** (not customer) — you'll need a subquery or CTE plus `ORDER BY ... LIMIT 1` per region, or a window function if you attempted the stretch above.

## Submission

Commit `solutions.sql` to your portfolio under `c37-week-04/exercise-01/`.
