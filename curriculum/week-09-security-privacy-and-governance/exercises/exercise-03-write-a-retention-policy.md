# Exercise 3 — Write a Data Retention Policy

Write a real retention policy for three Crunch Cycles tables, then implement the mechanism each policy describes as runnable SQL. A policy that isn't backed by a script that actually enforces it is a wish, not a policy.

**Estimated time:** 1 hour.

## Setup

Use `crunchcycles` with the `data_catalog` table from Lecture 3 (create it if you skipped ahead — the `CREATE TABLE` + sample rows are in that lecture).

## Tasks

1. **Write `retention-policy.md`** covering exactly these three tables, one section each, with this structure per table:
   - **Retention period** — a specific duration, not "as needed."
   - **Justification** — the concrete reason (legal, business, or risk-based) behind that specific duration.
   - **End-of-life mechanism** — exactly one of: hard delete, archive, or pseudonymize. State which.
   - **Trigger** — what condition starts the clock (e.g., "3 years after `MAX(order_date)` for that customer," not "eventually").

   Cover:
   - `orders` / `order_items` (financial records)
   - `customers` (contact/personal data)
   - `app_users` (deactivated staff accounts)

2. **Implement each mechanism as a runnable SQL script**, `enforce_retention.sql`, with one clearly commented section per table. Use Lecture 3's pseudonymization query as your `customers` starting point; write the `orders` archival logic as a query that *identifies* rows eligible for archival (you're not required to actually build a cold-storage target this week — `SELECT` the candidates and explain in a comment where they'd go) and the `app_users` logic as a real `DELETE`.

3. **Add a dry-run mode.** Every script that deletes or mutates data should be runnable in a mode that reports what *would* happen without doing it — a `SELECT COUNT(*)` / `SELECT *` version of each `UPDATE`/`DELETE`'s `WHERE` clause, run and reviewed before the real version ever executes.

## Expected result

```sql
-- Dry run: how many customers would be pseudonymized right now?
SELECT COUNT(*) FROM customers
WHERE customer_id NOT IN (
    SELECT DISTINCT customer_id FROM orders WHERE order_date > now() - interval '3 years'
)
AND contact_name NOT LIKE 'ERASED-%';
--  count
-- -------
--      1      -- (customer 18, Fjord Cycling Supply, who has never ordered)
```

## Done when

- [ ] `retention-policy.md` has all three tables, each with retention period, justification, mechanism, and trigger stated explicitly.
- [ ] `enforce_retention.sql` has a working, commented section per table.
- [ ] Every destructive statement has a dry-run counterpart you ran and checked first.
- [ ] The `orders`/`order_items` retention period is at least as long as your `justifications.md` from Exercise 2 assumes for any tax/financial reasoning you referenced there.
- [ ] No script silently deletes a row that another retention rule requires you to keep (e.g., pseudonymizing a customer never deletes their `orders` rows).
