# Week 10 — Homework

Five problems, ~5 hours total, spread across the week. These reinforce the lectures with a mix of schema design, SQL, a written explanation, and small extensions to code you've already built. Commit each.

All problems assume the `crunchcycles` and `crunchcycles_dw` Postgres databases from the [README](./README.md), unless a problem says otherwise.

---

## Problem 1 — Grain audit (30 min)

For each of the following four hypothetical fact tables, state the grain in one sentence, and identify **one specific business question** it could answer and **one specific business question** it could NOT answer without changing the grain:

1. `fact_daily_regional_sales` — one row per region, per day.
2. `fact_order_items` — one row per order line item (this week's actual grain).
3. `fact_orders` — one row per order (a coarser alternative this week rejected).
4. `fact_order_status_changes` — one row per time an order's status changed (a finer, event-based alternative not built this week).

**Deliver** `grain-audit.md`: 4 entries, each with grain statement, one answerable question, one unanswerable-at-this-grain question.

---

## Problem 2 — Build `dim_region` and compare (60 min)

Lecture 2 denormalized region directly into `dim_customer` and `dim_employee` rather than building a separate `dim_region` table (a star, not a snowflake). Build the snowflake alternative:

```sql
CREATE TABLE warehouse.dim_region (
    region_key   SERIAL PRIMARY KEY,
    region_id    INTEGER NOT NULL UNIQUE,
    region_name  TEXT NOT NULL
);
```

Add a `region_key` foreign key to `dim_customer` and `dim_employee` in place of the denormalized `region_name` text column, and rewrite Lecture 2 §7's "revenue by region by quarter" query against your new snowflake version.

**Deliver** `dim_region.sql` (the new DDL) and `snowflake-comparison.md`: your rewritten query, and 3–4 sentences comparing it to the star-schema original — count the joins in each, and state which version you'd actually choose for Crunch Cycles' scale and why (this should agree with Lecture 2 §3's trade-off table, argued in your own words).

---

## Problem 3 — Explain ELT to a non-technical stakeholder (30 min)

In `elt-writeup.md`, write **no more than 350 words**, in plain language a Crunch Cycles sales director (not an engineer) could follow, explaining:

1. What the warehouse is and why it's a separate database from the one the checkout uses.
2. What "the ELT job runs every night" means in practice — what happens if it fails one night, and what happens the next time it runs successfully.
3. Why last month's revenue number in the dashboard won't change retroactively just because a customer's region got reclassified this month (tie this to Lecture 2 §5's Type 1 vs. Type 2 distinction, explained without using the term "slowly changing dimension" — translate it).

No SQL, no code — the deliverable is whether a genuinely non-technical reader would come away with an accurate mental model.

---

## Problem 4 — A fourth KPI, self-derived (60 min)

Lecture 3 §3 defined six KPIs. Derive a **seventh**, following §2's discipline exactly: state a plausible Crunch Cycles business goal *first*, in one sentence, that none of the six lecture KPIs directly serve — then derive and define a new KPI from it, and write the SQL.

Good starting goals to pick from (or invent your own, if it's genuinely goal-first, not metric-first): "identify which products are being discounted most heavily," "understand seasonality in order volume," "flag sales reps whose average deal size is shrinking."

**Deliver** `kpi-07-definition.md` (goal, KPI name, the "20% move" test, precise formula) and `kpi-07-query.sql`, run and validated against your warehouse.

---

## Problem 5 — Rebuild one Lecture 3 chart as a bar chart AND a line chart (60 min)

Take the monthly revenue trend query from Lecture 3 §6. Render it two ways in one script, `chart-comparison.py`: once as a bar chart, once as a line chart, both zero-based, both saved to disk.

In a short comment block at the top of the file (3–5 sentences), state which chart type you think communicates the month-over-month trend more honestly for this specific data, and why — there's a real, defensible answer here (bar charts emphasize discrete period comparisons; line charts emphasize continuous trend/slope — which framing fits "is this growing or shrinking" better is worth reasoning through, not just asserting).

**Deliver** `chart-comparison.py` and both `.png` outputs.

---

## Time budget

| Problem | Time |
|--------:|-----:|
| 1 | 30 min |
| 2 | 60 min |
| 3 | 30 min |
| 4 | 60 min |
| 5 | 60 min |
| **Total** | **~4 h** |

After homework, take the [quiz](./quiz.md) and ship the [mini-project](./mini-project/README.md).
