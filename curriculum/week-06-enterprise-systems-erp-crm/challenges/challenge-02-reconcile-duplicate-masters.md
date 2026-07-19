# Challenge 2 — Design a Fix for Duplicated Master Data

**Time:** ~2 hours. **Difficulty:** Medium-hard. **No single right answer.**

## The scenario

Crunch Cycles' sales team has been entering customers directly, without ever checking if the company already exists in the system — exactly the failure mode Lecture 2 described. Run this against your database first:

```sql
INSERT INTO customers VALUES
(19,'Trailhead Bikes Inc','Jen Ortiz','jen.ortiz@trailheadbikes.com','Denver','USA',1,'2023-11-02'),
(20,'Le Vélo Parisien','Claire D.','claire.dubois@levelo.fr','Paris','France',2,'2023-08-14'),
(21,'Nordic Cycle Club Ltd','E. Solberg','erik.solberg@nordiccycle.no','Oslo','Norway',2,'2022-12-01');
```

These are near-duplicates of existing rows: customer 1 (`Trailhead Bikes`), customer 6 (`Le Velo Parisien`), and customer 7 (`Nordic Cycle Club`). Each new row has a slightly different name, a different (but plausibly-the-same-person) contact, and a different signup date — because it was re-entered by someone who didn't know the original existed. Worse: some of Crunch Cycles' **existing orders** may reference either the original or the duplicate `customer_id`, depending on which row the sales rep who took the order happened to pick. Simulate that:

```sql
-- One "new" order gets attributed to the duplicate customer 19, not the original customer 1
INSERT INTO orders VALUES
(41,19,2,'2024-06-15','2024-06-19','Completed');
INSERT INTO order_items VALUES
(41,2,1,1499.00);
```

## Your task

You are the data person asked to clean this up **without losing any history** — every order, ever placed, by either the original or duplicate row, has to still be attributable to the *real* company after your fix.

1. **Detect.** Write a SQL query that would have caught this automatically — flag pairs of `customers` rows that are probably the same company. You won't have exact-string-match duplicates in a real dataset (that's what makes this hard); use a looser signal: same `city` + `country` + a name that shares a recognizable root, or a matching email domain. Document exactly what heuristic you used and its false-positive/false-negative risk.
2. **Decide the survivor.** For each duplicate pair, decide which row survives as the canonical record (the "golden record") and which is retired. Write down your rule (e.g., "earliest `signup_date` wins," "the row with the most complete data wins") and apply it consistently — don't decide case by case with no stated rule.
3. **Reassign.** Write the SQL that repoints every `orders.customer_id` (and anything else that references the retired customer_id) from the retired row to the surviving row. **Do this inside a transaction** (`BEGIN` / `COMMIT`) and verify row counts before and after.
4. **Retire, don't delete.** Don't `DELETE` the retired customer rows outright — add a way to mark them merged (e.g., add a `merged_into_customer_id` column to `customers`, nullable, pointing at the survivor) so anyone who looks up the old row later can still find where its history went. Write the `ALTER TABLE` and the `UPDATE` that sets it.
5. **Verify.** Write a query proving that after your fix, (a) every order that used to point at a retired customer now points at the survivor, and (b) `SELECT company_name, COUNT(*) FROM customers WHERE merged_into_customer_id IS NULL GROUP BY company_name HAVING COUNT(*) > 1` returns zero rows for the pairs you fixed.
6. **Write it up.** In `writeup.md` (400–500 words): what you did, why you chose the survivor rule you chose, and — critically — **what process change** (not just a one-time SQL fix) would prevent this from happening again. A one-time cleanup without a process fix just means you'll be back here in six months.

## Constraints

- All detection, decision logic, and the fix itself in SQL. This is exactly what Lecture 2's "master data quality checks" section modeled — extend that pattern, don't invent a new tool.
- **No data loss.** Every order that existed before your fix must be attributable to a real customer after it. If your fix would orphan an order, your fix is wrong.
- State every assumption your "survivor" rule depends on, explicitly, in `writeup.md`.

## Hints

<details>
<summary>On detecting near-duplicates without exact string match</summary>

A reasonable heuristic: `same city AND same country AND (LOWER(company_name) LIKE LOWER(other.company_name) || '%' OR LOWER(other.company_name) LIKE LOWER(company_name) || '%')` — i.e., one name is a prefix of the other, ignoring case ("Trailhead Bikes" is a prefix of "Trailhead Bikes Inc"). This won't catch every real-world case (it wouldn't catch "Le Vélo Parisien" vs. "Le Velo Parisien" — the accent breaks a naive prefix match!) — which is exactly why you're asked to document the heuristic's limits, not pretend it's perfect.

</details>

<details>
<summary>On the transaction step</summary>

```sql
BEGIN;

UPDATE orders
SET customer_id = 1              -- survivor
WHERE customer_id = 19;           -- retired

-- verify the count moved as expected before committing
SELECT COUNT(*) FROM orders WHERE customer_id = 19;   -- should now be 0
SELECT COUNT(*) FROM orders WHERE customer_id = 1;    -- should be original count + 1

COMMIT;   -- or ROLLBACK if either check looks wrong
```

</details>

## How success is judged

| Signal | Weak answer | Strong answer |
|---|---|---|
| Detection | Manually names the 3 pairs with no reusable query | A `SELECT` that would flag these (and similar future) pairs automatically |
| Survivor rule | Ad hoc, inconsistent choice per pair | One stated rule, applied consistently to all 3 pairs |
| Data safety | Deletes rows, or a fix that could orphan an order | Retired rows kept and tagged; a transaction with verification before commit |
| Verification | Assumes the fix worked | Runs and shows the actual before/after count queries |
| Process fix | Only proposes cleaning up again later | Names a concrete prevention mechanism (e.g., a uniqueness check before insert, a required "search first" step in the sales workflow) |

## Submission

Commit `challenge-02/fix.sql` and `challenge-02/writeup.md` to your portfolio under `c37-week-06/challenge-02/`.
