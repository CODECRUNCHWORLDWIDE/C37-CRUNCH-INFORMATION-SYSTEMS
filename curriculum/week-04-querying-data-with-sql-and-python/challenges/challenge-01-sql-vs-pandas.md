# Challenge 1 — Solve One Problem Two Ways and Compare

**Time:** ~60 minutes. **Difficulty:** Medium.

## The scenario

Maria Chen (Sales Director) wants a "customer health" list: **every customer's total completed revenue, total order count, and their single most recent order date** — sorted worst-to-first by revenue, so she can see who's gone quiet. This is exactly the kind of question where SQL and pandas can *both* get the right answer, but the path there — and the code you'd want to maintain a year from now — looks different.

## Your task

Solve it **twice**, and write up the comparison in `challenge-01.md`.

### Solution A — pure SQL

One query, using joins, `GROUP BY`, and whatever aggregate functions you need to also get "most recent order date" per customer (hint: `MAX()` is an aggregate too, not just `SUM`/`COUNT`/`AVG`). Include customers with **zero** completed orders — they should show `0` revenue, `0` orders, and a `NULL` most-recent-date, not be missing from the list entirely. Think back to Lecture 1 §2–3 for the join type that keeps them.

### Solution B — pandas

Pull the minimum raw data needed with `read_sql` (a simpler query than Solution A — you're pushing the aggregation logic into Python instead), then use `groupby`/`agg`/`merge` to build the identical result.

### The comparison

For both solutions, answer in `challenge-01.md`:

1. **Line count and readability** — paste both. Which is shorter? Which would a new teammate understand faster with zero explanation?
2. **Where did the "customers with zero orders" requirement bite you?** Which solution handled it more naturally, and why?
3. **Performance intuition** — you don't need to benchmark microseconds, but reason about it: which one makes the database do more work, and which makes Python do more work? At what data size (thousands of customers? millions?) would that difference start to matter?
4. **Which one would you actually ship**, and why? It's fine — expected, even — to say "it depends," but say specifically what it depends on.

## Constraints

- Both solutions must produce the **same result** — same customers, same numbers, same row count. Diff them (row by row, or with a quick `pd.testing.assert_frame_equal` after loading Solution A's output into a DataFrame too) to prove it.
- Solution A is SQL only — no pandas.
- Solution B must do its aggregation in pandas — don't just write the same `GROUP BY` SQL and call `read_sql` on it; that's Solution A with extra steps.

## Hints

<details>
<summary>On including zero-order customers (Solution A)</summary>

You need a `LEFT JOIN` from `customers` outward to `orders`, filtered to `status = 'Completed'` — but be careful *where* that filter goes. Put `AND o.status = 'Completed'` in the `WHERE` clause and you've accidentally turned your `LEFT JOIN` back into an inner join (a customer with only Pending/Cancelled orders gets filtered out entirely, instead of showing `0`). Put the status condition in the `ON` clause instead, so it's evaluated as part of the join, not after it:

```sql
FROM customers c
LEFT JOIN orders o ON o.customer_id = c.customer_id AND o.status = 'Completed'
```

This is a classic, subtle bug — `WHERE` vs. `ON` placement on an outer join changes the result. Worth writing a sentence about in your comparison.

</details>

<details>
<summary>On "most recent order date" for a zero-order customer (Solution B)</summary>

`groupby("customer_id")["order_date"].max()` on an empty group won't produce a row at all — you'll need to `merge` your aggregated result back onto the *full* customer list (`how="left"`) to get zero-order customers to appear with `NaT`/`0`, the pandas mirror of the SQL `LEFT JOIN` issue above.

</details>

## How success is judged

| Signal | Weak answer | Strong answer |
|--------|-------------|----------------|
| Correctness | One solution silently drops zero-order customers | Both solutions include every customer, verified to match |
| Comparison depth | "SQL is faster" with no reasoning | Specific reasoning about where each engine's work happens and when that matters |
| Bug awareness | Doesn't notice the `WHERE`-vs-`ON` trap | Names it explicitly, in either solution |
| Decision | No recommendation, or a coin flip | A stated recommendation with a concrete "it depends on X" |

## Submission

Commit `challenge-01.md` plus both solutions (`solution_a.sql`, `solution_b.py`) to your portfolio under `c37-week-04/challenge-01/`.
