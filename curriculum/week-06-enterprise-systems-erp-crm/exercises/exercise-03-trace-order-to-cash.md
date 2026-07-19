# Exercise 3 — Trace Order-to-Cash Across Systems

**Goal:** Turn Lecture 3's order-to-cash chain into real, working SQL against this week's schema — following specific deals from CRM opportunity through to a shipped ERP order, and finding the process breaks a real ops team would want flagged.

**Estimated time:** 60–90 minutes.

## Setup

Confirm your tables are loaded (see the [week README](../README.md)):

```sql
SELECT COUNT(*) FROM crm_opportunities;   -- must print 7
SELECT COUNT(*) FROM orders;              -- must print 40 (from Week 4)
```

Create a file `solutions.sql` and put each answer under a `-- Task N` comment.

## Tasks

1. **Every opportunity, plain.** Select `opportunity_id`, `customer_id`, `stage`, `est_value` for all 7 opportunities, ordered by `opportunity_id`. *(Expected: 7 rows — your baseline to check the rest against.)*

2. **Won deals, with the customer's name.** Join `crm_opportunities` to `customers` and select the customer's `company_name` alongside each **Won** opportunity's `stage` and `est_value`. *(Expected: 3 rows.)*

3. **Won deals, matched to their real order.** Join `crm_opportunities` (where `stage = 'Won'`) to `orders` on `won_order_id = order_id`. Select `opportunity_id`, `company_name` (via a further join to `customers`), `order_id`, `order_date`, `ship_date`. *(Expected: 3 rows — every Won deal in this seed does have a matching order.)*

4. **Forecast vs. actual.** Extend Task 3: also join to `order_items` and compute `SUM(quantity * unit_price)` as `actual_value`, grouped appropriately. Select `opportunity_id`, `est_value`, `actual_value`, and a computed column `variance` = `actual_value - est_value`. *(Expected: 3 rows. Note which opportunity's forecast was closest to reality.)*

5. **The process break query.** Select any opportunity where `stage = 'Won'` but `won_order_id IS NULL` — a deal marked won with no matching sales order created yet. *(Expected: 0 rows in this clean seed — write the query anyway; you'll reuse it verbatim on real, messier data one day.)*

6. **Open pipeline, no purchase yet.** Select every opportunity that is **not** Won and **not** Lost (i.e., still actively open: Prospecting, Qualified, or Proposal), with the customer's `company_name` and whether that customer has **ever** placed a real order (`EXISTS` against `orders`). *(Expected: 3 rows — Fjord, Mumbai Mountain Bikes, and Northern Gear Co. Two of the three should show "has never ordered.")*

7. **Rep pipeline vs. rep closed sales.** For each employee who owns at least one opportunity, compute two numbers side by side: the count of opportunities they own, and the count of **distinct orders** they've actually closed (`orders.employee_id`). *(Hint: two separate aggregates, joined or computed with subqueries — watch out for double-counting if you try to do it in one `GROUP BY` with a naive join.)*

8. **Priya's story, in one query.** Employee 7 (Priya Nair) has one CRM opportunity and, per Week 4, zero completed orders. Write a single query that proves both facts in one result row: her name, her opportunity count, and her completed-order count. *(This is the exact query you'd hand a sales manager asking "is Priya actually working, or just not closing?")*

9. **Supplier side: lead time performance.** For every **received** purchase order, compute `actual_lead_time_days` (`received_date - po_date`) and compare it to the supplier's quoted `lead_time_days`. Select `po_id`, `supplier_name`, `actual_lead_time_days`, `lead_time_days`, and a computed `on_time` boolean (`actual_lead_time_days <= lead_time_days`). *(PostgreSQL: plain date subtraction. SQLite: wrap both sides in `julianday(...)`.)* *(Expected: 4 rows — POs 1, 2, 3, 6 are Received.)*

10. **Everything still in motion.** One final query: every `purchase_orders` row that is **not** `Received` (i.e., `Open` or `Cancelled`), with the supplier name and how many days it's been since the `po_date` (use `CURRENT_DATE - po_date`, or hardcode `'2024-06-01'` as "today" for a deterministic answer). *(Expected: 3 rows.)*

## Expected result (spot checks)

- Task 2 → 3 rows (Desert Trail Cycles, Seoul Cycle Hub, Le Velo Parisien).
- Task 5 → 0 rows.
- Task 6 → 3 rows, two of which have never placed a real order.
- Task 9 → 4 rows; PO 2 should show `on_time = FALSE`.

## Done when…

- [ ] All 10 queries run without error on your engine and are saved with `-- Task N` comments.
- [ ] Task 6 correctly identifies which open-pipeline customers have never ordered.
- [ ] Task 8 returns exactly one row for Priya with both counts visible.
- [ ] You can explain, out loud, why Task 5 returning 0 rows here doesn't mean the query is useless in a real, larger dataset.

## Stretch

Write one more query: total **estimated pipeline value** currently open (`Prospecting` + `Qualified` + `Proposal` stages, summed), compared to total **actual revenue** from Won deals this quarter. What story do the two numbers tell together that neither tells alone?

## Submission

Commit `solutions.sql` to your portfolio under `c37-week-06/exercise-03/`.
