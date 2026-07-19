# Mini-Project — A Repeatable SQL-to-Pandas Reporting Pipeline

> Build one script, `crunch_report.py`, that queries PostgreSQL, transforms the result in pandas, and writes a clean, multi-part sales report — parameterized by date range, safe to re-run, and with **zero** spreadsheet anywhere in the loop.

**Estimated time:** 2.5–3 hours, best done Saturday after the exercises and challenges.

This is the week's capstone, and it's the shape of real analytics-engineering work: Maria Chen (Sales Director) doesn't want a one-off number — she wants a report she can ask for again next month, with different dates, and trust that it's correct every time. You have every piece already: joins and aggregation (Exercise 1), the SQL-to-pandas bridge (Exercise 2), parameterization (Exercise 3), and the idempotent-pipeline pattern (Lecture 3 + Challenge 2). This project assembles them into one real deliverable.

---

## The brief

Maria has three standing questions she asks every reporting period:

1. **"How did each region do?"** — orders and revenue by region, for the period.
2. **"Who are our top customers this period?"** — top 10 customers by revenue, with their order count.
3. **"What's selling?"** — revenue by product category, for the period.

She wants **one command** that answers all three, for whatever date range she names, writing results somewhere she (and anyone else with database access) can query directly — not a file she has to ask you to re-open and re-run by hand.

---

## Deliverable

A directory in your portfolio `c37-week-04/mini-project/` containing:

1. **`crunch_report.py`** — the pipeline script (see Requirements below).
2. **`report.md`** — a short written report: for **Q1 2024** (`2024-01-01` to `2024-04-01`), the actual numbers your script produced for all three questions, presented as readable tables, plus 2–3 sentences of interpretation per question (not just the numbers — what do they *mean*?).
3. **`notes.md`** — a short reflection (see the end).

Everything runs against the `crunchcycles` schema from the [week README](../README.md#setup-load-the-week-3-schema).

---

## Requirements

### 1. Structure: extract → transform → load, per question

Three logical extracts (or one shared extract you slice three ways — your call, document which and why), three transforms, three loads. Reuse the `get_engine()` pattern from the exercises.

### 2. Parameterized by date range

```bash
python crunch_report.py --start 2024-01-01 --end 2024-04-01
```

`--start` (inclusive) and `--end` (exclusive) control every query. No hardcoded dates anywhere in the script. Reject `end <= start` with a clear error before touching the database (Exercise 3, Task 6).

### 3. All three questions, correctly joined

- **Regional summary** — `region_name`, `order_count` (`COUNT(DISTINCT order_id)`), `revenue`, sorted by revenue descending. **Completed orders only.**
- **Top 10 customers** — `company_name`, `order_count`, `revenue`, sorted by revenue descending, limited to the top 10. **Completed orders only.**
- **Category breakdown** — `category`, `revenue`, `units_sold` (`SUM(quantity)`), sorted by revenue descending. **Completed orders only.** (This one needs `products` joined in for `category` — reuse the `merge` pattern from Exercise 2 Task 7, or do it in SQL — your choice, but be consistent with your Challenge 1 reasoning about when to push work into SQL vs. pandas.)

### 4. Written back to the database, not just printed

Each of the three results is written with `to_sql` into its own table: `report_regional_summary`, `report_top_customers`, `report_category_breakdown`. Use `if_exists="replace"` — each run should fully regenerate all three, not accumulate.

### 5. Idempotent and safe to re-run

Running the script twice in a row with the same `--start`/`--end` must leave the three tables in exactly the same state, not doubled. Prove it: run it twice, `SELECT COUNT(*)` from each table both times, show the counts are identical in your `notes.md`.

### 6. Logged, not silent

Use `logging` (Lecture 3 §5), not bare `print`, for at least: the date range being processed, and the row count written to each of the three tables.

### 7. `report.md` — the human-readable deliverable

For the Q1 2024 run specifically, present all three results as markdown tables in `report.md`, each followed by 2–3 sentences answering: *what does this number mean for the business, and is there anything here worth flagging?* (There is — you found it in Exercise 1 Q9. Say what it is and what you'd ask Maria to investigate.)

---

## Verify your numbers against the exercises

Your pipeline's regional summary for the full 6-month window (`--start 2024-01-01 --end 2024-07-01`) must match Exercise 1 Q7 / Exercise 2 Task 4 exactly: North America $18,797.00 / 15 orders, Europe $13,287.00 / 9, APAC $10,619.00 / 5, LATAM $9,416.00 / 6. If it doesn't, you have a bug — find it before writing `report.md`. This cross-check is not optional; a report that "looks plausible" but doesn't match numbers you've already independently verified is exactly the kind of silent error a real reporting pipeline must never ship.

---

## Milestones

- **Milestone 1 (45 min):** `extract`/`transform`/`load` for the regional summary alone, parameterized, writing to `report_regional_summary`. Verify against Exercise 1 Q7.
- **Milestone 2 (45 min):** Add the top-10-customers question, same pattern, own table.
- **Milestone 3 (45 min):** Add the category breakdown (the one needing `products` joined in), own table.
- **Milestone 4 (30–45 min):** Idempotency proof, logging pass, then run for Q1 2024 and write `report.md`.

---

## Rules

- **No spreadsheet anywhere** — not as intermediate storage, not as the final deliverable. `report.md` is markdown; the machine-readable output lives in Postgres tables.
- **All three report tables must exist after one run** and be independently queryable with plain SQL — don't make them require re-running the Python script to inspect.
- **Every number in `report.md` must be reproducible** by re-running `crunch_report.py --start 2024-01-01 --end 2024-04-01` — no hand-edited numbers.
- **State your SQL-vs-pandas choice** for the category breakdown explicitly in `notes.md`, consistent with your reasoning in Challenge 1.

---

## Rubric

| Criterion | Weight | "Great" looks like |
|-----------|------:|--------------------|
| Correctness | 30% | All three reports match the cross-check numbers; Q1 numbers in `report.md` are internally consistent |
| Idempotency | 20% | Verified re-run produces identical row counts, shown in `notes.md` |
| Structure | 20% | Clean `extract`/`transform`/`load` separation; no SQL built with f-string interpolation of user input |
| Parameterization | 15% | `--start`/`--end` fully control the report; bad ranges rejected clearly |
| Communication | 15% | `report.md` reads like something you'd actually hand to Maria — numbers plus meaning |

---

## Reflection (`notes.md`, ~250 words)

1. Which of the three reports needed the most joins, and did that change how you split work between SQL and pandas?
2. Show your idempotency proof (the two run's row counts) and explain in one sentence why they match.
3. What did you flag from the revenue-decline pattern (Exercise 1 Q9), and what would you actually ask Maria or investigate next, if this were real data?
4. If Maria asked for this report **every single day** instead of on demand, what would you change about the script? (Foreshadows Challenge 2 and Week 5's process automation.)

---

## Stretch goals

- Add a fourth report: month-over-month revenue trend (reuse the CTE pattern from Exercise 1 Q9), written to `report_monthly_trend`.
- Add a `--format` flag that, alongside the `to_sql` writes, also exports each report as a CSV (`report.to_csv(...)`) for someone who wants a file to attach to an email — note in `notes.md` why this CSV is an *export*, not the system of record, and how that's different from "using a spreadsheet as a database."
- Wrap the whole pipeline in a single `try`/`except` that logs a failure and exits non-zero (Challenge 2's pattern) — the mini-project script should already be "cron-safe" going into next week.

---

## Why this matters

This is the pipeline shape you'll use for the rest of the course and, honestly, for most of a data/analytics career: pull from a real store with SQL, shape it in pandas, hand back something durable and re-runnable. Week 5 automates the *process* around a report like this one; Week 10 (Analytics & BI) builds on exactly this pattern at warehouse scale. Keep `crunch_report.py` — you'll extend it, not replace it.

When done: push, then take the [quiz](../quiz.md) and start [Week 5 — Process modeling & automation](../../week-05-process-modeling-and-automation/).
