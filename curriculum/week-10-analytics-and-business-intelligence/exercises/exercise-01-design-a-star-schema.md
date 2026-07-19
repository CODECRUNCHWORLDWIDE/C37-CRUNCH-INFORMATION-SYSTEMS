# Exercise 1 — Design a Star Schema for Sales

**Goal:** design a star schema for Crunch Cycles sales analytics **from scratch, on paper first** — grain, facts, dimensions, surrogate keys — before comparing your design against Lecture 2's reference schema. The comparison is the point: seeing exactly where your instincts matched the reference and where they didn't is worth more than reading the reference cold.

**Estimated time:** 90 minutes.

## Setup

No install needed — this is a design document, not code yet (Exercise 2 builds it for real). Have Lecture 2 open in a second tab but **don't read past §2 (Facts and dimensions) until you've finished Tasks 1–4 below.** Re-reading the rest early defeats the exercise.

**The business questions your schema must be able to answer**, stated up front (this is how real dimensional modeling starts — from the questions, not from the source tables):

1. "What was total revenue, and gross margin, for each quarter of the last two years?"
2. "Which sales rep generated the most revenue last month?"
3. "Which product category is growing fastest, region by region?"
4. "What's our average order value, and is it trending up or down?"

## Tasks

Produce a single Markdown file, `sales-star-schema-design.md`, with these sections — **written before you re-read Lecture 2 past §2:**

1. **Grain statement.** In one sentence, what does one row of your fact table represent? Justify it against the four questions above — could you answer all four at this grain, or would some require a grain that's too coarse?

2. **Fact table.** List every column you'd put on the fact table, split into two groups: **measures** (numeric, summable — e.g., a price, a quantity) and **degenerate dimensions** (identifiers with business meaning but no attributes worth their own table — e.g., an order number). For each measure, name which of the four business questions it's needed to answer.

3. **Dimension tables.** List each dimension table you'd create, and for each one: its columns, and whether you'd denormalize a related concept into it (like folding region into customer) or keep it as a separate joined table. Justify the denormalize-or-not call for at least one dimension using the star-vs-snowflake trade-off (query speed vs. storage vs. clarity).

4. **Surrogate vs. natural keys.** For one dimension, write out its `PRIMARY KEY` and explain — in your own words, not copied from the lecture — why it's a warehouse-generated surrogate key rather than reusing the source system's ID directly.

5. **Now compare.** Read Lecture 2 §3 onward (the reference `warehouse.dim_date`, `dim_customer`, `dim_product`, `dim_employee`, `fact_order_items` design) and write a short comparison table:

   | Your design | Reference design | Same or different? | If different, which is better and why? |
   |---|---|---|---|

   Include at least 4 rows. It's fine — expected, even — if your grain or dimension choices differ from the reference; the value here is articulating *why*, not matching it exactly.

## Expected result (spot checks)

- Your grain statement is one sentence and specific enough that a stranger could look at any row of your (hypothetical) fact table and know exactly what real-world event it represents.
- Every measure you listed is summable (revenue, quantity, cost — not something like "customer email," which belongs on a dimension).
- Your comparison table has at least one row where your design differed from the reference, with a real justification either way — a comparison table that says "same" four times in a row means you re-read the lecture before finishing Tasks 1–4, which skips the point of the exercise.

## Done when…

- [ ] `sales-star-schema-design.md` has all 5 sections.
- [ ] Sections 1–4 were written **before** re-reading Lecture 2's reference schema (say so explicitly at the top of the file — "written cold, compared after").
- [ ] Your grain statement could answer all four business questions listed in Setup, or you've explicitly noted which one it can't and why.
- [ ] The comparison table (Task 5) has honest, specific reasoning — not just "matched" or "didn't match."

## Stretch

- Design a `dim_date` variant that also supports a **fiscal year** starting April 1 instead of the calendar year (add `fiscal_year` and `fiscal_quarter` columns) — write the `EXTRACT`/`CASE` logic that would compute them from `full_date`.
- Sketch what changes if Crunch Cycles adds a second sales channel (an online store, alongside the existing rep-driven sales) — does `channel` belong as a new dimension, a new column on an existing dimension, or a degenerate dimension on the fact table? Justify your choice.

## Submission

Commit `sales-star-schema-design.md` to your portfolio under `c37-week-10/exercise-01/`.
