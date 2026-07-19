# Week 4 — Quiz

Fourteen questions. Lectures closed. Aim for 12/14 before starting Week 5. The answer key at the bottom explains the *why*, not just the letter.

---

**Q1.** You want "every customer, whether or not they've ever ordered." Which join keeps that guarantee?

- A) `INNER JOIN`
- B) `LEFT JOIN` from `customers` to `orders`
- C) `CROSS JOIN`
- D) A self-join

---

**Q2.** `LEFT JOIN orders o ON o.customer_id = c.customer_id WHERE o.order_id IS NULL` returns:

- A) Every customer, matched or not
- B) Only customers with at least one order
- C) Only customers with **zero** orders
- D) Every order with no customer

---

**Q3.** Why does almost nobody write `RIGHT JOIN` in practice?

- A) It's not valid SQL in PostgreSQL
- B) Swapping the table order and using `LEFT JOIN` produces the identical result and reads more consistently
- C) `RIGHT JOIN` is slower than `LEFT JOIN`
- D) `RIGHT JOIN` can't be combined with `WHERE`

---

**Q4.** `SELECT COUNT(*) FROM orders o JOIN order_items oi ON oi.order_id = o.order_id;` on the Week 4 seed data returns **54**, not 40. Why?

- A) It's a bug in the join condition
- B) `order_items` has more than one row for some orders, so the join produces one row per **line item**, not per order
- C) `COUNT(*)` always double-counts
- D) `orders` actually has 54 rows

---

**Q5.** What's the difference between `WHERE` and `HAVING`?

- A) They're interchangeable
- B) `WHERE` filters rows before grouping; `HAVING` filters groups after aggregation, and can reference aggregate functions
- C) `HAVING` runs before `GROUP BY`; `WHERE` runs after
- D) `WHERE` only works with `JOIN`; `HAVING` only works with `GROUP BY`

---

**Q6.** In a `LEFT JOIN`, placing a filter like `AND o.status = 'Completed'` in the `ON` clause instead of the `WHERE` clause matters because:

- A) There's no difference — both produce identical results
- B) Putting it in `WHERE` can silently turn the outer join back into an inner join, dropping rows the `LEFT JOIN` was supposed to keep
- C) `ON` only accepts equality conditions
- D) `WHERE` is evaluated before the join, so it can't reference joined columns at all

---

**Q7.** What is a Common Table Expression (`WITH ... AS (...)`) mainly used for?

- A) Making a query run faster in every case
- B) Naming an intermediate result so multi-step logic reads top-to-bottom instead of nesting subqueries
- C) Creating a permanent table
- D) Replacing `GROUP BY`

---

**Q8.** `pd.read_sql(query, engine, parse_dates=["order_date"])` does what, compared to a plain `pd.read_sql(query, engine)`?

- A) Filters out rows with missing dates
- B) Converts the `order_date` column to a proper `datetime64` dtype during the read, instead of leaving it as `object`/string
- C) Sorts the result by `order_date`
- D) Has no effect — pandas always infers date columns automatically

---

**Q9.** You want a report table to always reflect exactly this run's data, with no leftover rows from a previous run. Which `to_sql` argument gives you that?

- A) `if_exists="append"`
- B) `if_exists="fail"`
- C) `if_exists="replace"`
- D) `index=True`

---

**Q10.** In `df1.merge(df2, on="product_id", how="left")`, `how="left"` behaves most like which SQL join?

- A) `INNER JOIN`
- B) `LEFT JOIN`
- C) `CROSS JOIN`
- D) `FULL OUTER JOIN`

---

**Q11.** What does `pivot_table(index="region_name", columns="order_month", values="revenue", aggfunc="sum", fill_value=0)` produce that a plain `groupby(["region_name", "order_month"])["revenue"].sum()` doesn't, on its own?

- A) A different total sum
- B) A wide grid with one row per region and one column per month, with explicit `0`s for empty combinations, instead of a long list of (region, month) pairs
- C) It removes duplicate rows
- D) It sorts the data chronologically

---

**Q12.** Why is `query = f"SELECT * FROM orders WHERE order_date >= '{start_date}'"` dangerous, even in a script only you run?

- A) f-strings are slower than `.format()`
- B) It opens a SQL-injection door and breaks if `start_date` ever contains a quote — today's private script often becomes tomorrow's shared tool
- C) SQLAlchemy doesn't support f-strings
- D) It's not actually dangerous — the danger only applies to web applications

---

**Q13.** A script is idempotent when:

- A) It runs faster on the second execution
- B) Running it multiple times with the same inputs leaves the system in the same end state as running it once
- C) It never writes to the database
- D) It always produces different output each time, by design

---

**Q14.** You need to combine a SQL query result with a CSV a partner emailed you, then hand the result to a plotting library. Where should the join happen, and why?

- A) In SQL — always do every join in SQL regardless of data source
- B) In pandas, with `merge` — the CSV isn't in the database, and pandas is what both results need to pass through anyway
- C) It can't be done — data from two different sources can never be combined
- D) Export the SQL result to a spreadsheet and combine there

---

## Answer key

<details>
<summary>Reveal after attempting</summary>

1. **B** — `LEFT JOIN` keeps every row from `customers` regardless of a match in `orders`, `NULL`-padding where there's no order. `INNER JOIN` would drop customers with zero orders entirely — the opposite of the requirement.
2. **C** — this is the anti-join pattern: the `LEFT JOIN` pads unmatched customers with `NULL` in `order_id`; filtering for that `NULL` isolates exactly the customers with no match — zero orders.
3. **B** — `RIGHT JOIN` is legal and works, but the community convention is to always write `LEFT JOIN` (swapping table order if needed) so every join in a codebase reads the same way.
4. **B** — joining `order_items` (which can have 2 rows for one order) makes each joined row represent one line item, not one order; `COUNT(*)` counts whatever a row represents, which here is line items (54 of them, from 40 orders, some with 2 items each).
5. **B** — `WHERE` runs before `GROUP BY`, so it can only see individual row values; `HAVING` runs after grouping, so it can reference `COUNT()`, `SUM()`, etc.
6. **B** — this is the subtle bug: `WHERE o.status = 'Completed'` runs after the join and removes any row where that condition is false or `NULL` — including a customer's otherwise-`NULL`-padded outer-join row — effectively cancelling the `LEFT JOIN`'s purpose. Putting the condition in `ON` evaluates it as part of the join itself, preserving unmatched left rows.
7. **B** — a CTE's main value is readability: naming a step lets you build multi-stage logic that reads top-to-bottom. (It is *not* guaranteed to be faster — Postgres may or may not inline it — and it does not create a permanent table.)
8. **B** — `parse_dates` tells pandas to parse those columns as real datetimes during the read, instead of you having to call `pd.to_datetime` afterward.
9. **C** — `if_exists="replace"` drops and recreates the table each run, so the table always reflects exactly the current run's data — no accumulation from previous runs.
10. **B** — `how="left"` keeps every row from the left DataFrame regardless of a match, `NaN`-padding unmatched columns — the same behavior as SQL's `LEFT JOIN`.
11. **B** — `pivot_table` reshapes the same aggregated numbers into a wide grid (one row per region, one column per month) with `fill_value=0` making "no data that month" an explicit zero — `groupby` alone gives you the same numbers but in long, one-row-per-combination form.
12. **B** — building SQL text by splicing a variable directly into the string is a SQL-injection vulnerability and breaks on unescaped special characters, regardless of whether the script is "just for you" today — always use bound parameters (`text(...)` + `params={...}`) instead.
13. **B** — that's the definition: same inputs, same end state, no matter how many times you run it. A script that doubles a report's rows on a second run is not idempotent.
14. **B** — the CSV doesn't live in the database, so SQL can't join it; pandas' `merge` is the natural place to combine a SQL result with data from any other source, and the combined DataFrame is also what a plotting library expects.

</details>

**Scoring:** 12+ → start Week 5. 9–11 → re-read the lecture sections behind your misses. <9 → re-read all three lectures from the top before continuing; this week's concepts are the backbone of everything from here through Week 12.
