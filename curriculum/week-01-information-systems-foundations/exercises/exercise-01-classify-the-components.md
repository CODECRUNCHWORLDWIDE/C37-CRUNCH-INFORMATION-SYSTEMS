# Exercise 1 — Classify the Components

> **Time:** ~1 hour. **Uses:** Lecture 1, Lecture 2. Works against the `is_components` table from the [week README](../README.md).

## Part A — Classify without the database (15 min)

For each item below, write down which of the four categories it belongs to — **people, process, data, technology** — and one sentence defending your answer. A few of these are deliberately ambiguous; that's the point.

1. The barcode printed on a bag of Riverbend coffee.
2. "Confirm the customer's ship-to address before charging the card."
3. Devon Park, the Barista Lead.
4. The shared spreadsheet Riverbend used for wholesale orders before this week's register existed.
5. The fact that a specific customer, Maria Ibarra, has an active subscription.
6. A card reader plugged into the POS terminal.
7. "Whenever an order sits unconfirmed for more than 24 hours, call the customer."
8. The Wholesale department as a whole.
9. Nora Chen's decision to approve a large one-time wholesale discount.
10. The green-coffee supplier Riverbend buys from.

**Expected difficulty:** items 4, 8, and 9 should make you stop and think. Write your reasoning, not just a one-word answer, for those three specifically.

## Part B — Verify against the seed (10 min)

Run these against your `riverbend` database and confirm the counts match:

```sql
SELECT category, COUNT(*) FROM is_components WHERE category = 'people' GROUP BY category;
SELECT category, COUNT(*) FROM is_components WHERE category = 'process' GROUP BY category;
SELECT category, COUNT(*) FROM is_components WHERE category = 'data' GROUP BY category;
SELECT category, COUNT(*) FROM is_components WHERE category = 'technology' GROUP BY category;
```

**Expected:** 5 for each category, 20 total.

*(If `GROUP BY` is unfamiliar, that's fine — you can also just run `SELECT COUNT(*) FROM is_components WHERE category = 'people';` four separate times, once per category, and get the same answer. Either way works; `GROUP BY` is a Week 3/4 topic, not required here.)*

## Part C — Extend the register (30 min)

Riverbend just hired a bookkeeper and started a lightweight customer-service function that didn't exist in the original seed. Add **five new components** to `is_components` — at minimum one `people` row and one `process` row, spread department-appropriately:

1. A `people` row for a bookkeeper (`Finance` department) who reconciles daily sales and pays suppliers.
2. A `process` row for how a customer complaint gets logged and resolved (`Support` department).
3. A `data` row for the record that process produces — a complaint ticket, with what it's about and its resolution (`Support` department).
4. A `technology` row for whatever tool the bookkeeper or support person actually uses day to day (your choice — a spreadsheet is a *legal* answer here, but if you pick it, write one sentence in your notes explaining the risk, per Lecture 2 §4).
5. One more component of any category that you think Riverbend is realistically missing. Justify it in one sentence.

Write all five as `INSERT` statements. Use `component_id` values `21`–`25` so you don't collide with the seed.

**Deliverable:** your five `INSERT` statements in `week01-solutions.sql`, plus your Part A answers and Part C item-5 justification in `notes.md`.

## Check yourself

- Run `SELECT * FROM is_components WHERE department = 'Support';` — you should see at least the two rows you just added.
- Run `SELECT COUNT(*) FROM is_components;` — should now read **25**.
- For item 4 in Part A: did you classify the spreadsheet's *existence* as technology, and the informal habits around editing it as an unmodeled (missing) process? If you classified the whole thing as one blob, go back and split it — that split is exactly what Lecture 2 §7 warns you to do.
