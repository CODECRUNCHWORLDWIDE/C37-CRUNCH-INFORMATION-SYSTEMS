# Mini-Project — A Data Warehouse, an ELT Job, and a BI Dashboard

> Build the analytics layer this week's lectures argued Crunch Cycles needs: a star-schema warehouse in its own PostgreSQL database, an idempotent ELT job that keeps it fresh from the operational store, and a dashboard delivering 4–6 KPIs that honestly answer real questions about the business. This is the week's capstone — Lectures 1–3 and all three exercises, combined into one working system.

**Estimated time:** 2.5–3 hours, best done Saturday after the exercises and challenges.

By now you can explain why OLTP and OLAP need separate stores, design a star schema with a deliberate grain, build an idempotent ELT job, and tell a real KPI apart from a vanity metric. A real analytics engineer's sprint is some mix of all four, on the same ticket, for a stakeholder who's going to make a decision based on what you ship. This project asks you to build that ticket, start to finish, for Crunch Cycles.

---

## Deliverable

A directory in your portfolio, `c37-week-10/mini-project/`, containing:

1. `schema.sql` — every `CREATE TABLE` for `crunchcycles_dw`'s `raw` and `warehouse` schemas (Lecture 2's star schema, exactly or with your own justified refinements from Exercise 1).
2. `elt_warehouse_load.py` — the ELT job (built out from Exercise 2) loading `raw` from `crunchcycles` and transforming into `warehouse`.
3. `dashboard.py` — a Python script that queries the warehouse and produces both printed KPI values and at least 3 saved chart images.
4. `KPIs.md` — a definition of every KPI on your dashboard (name, business question, exact formula, filters) — the same rigor as Exercise 3's `kpi-definitions.md`, for the KPIs you actually shipped.
5. `run-log.md` — a transcript proving the system works: two consecutive runs of the ELT job with row counts proving idempotency, and the dashboard's printed output.
6. `README.md` — how to set up and run everything, written so a stranger could get it running in under 15 minutes.

Everything runs against your `crunchcycles` operational database from Weeks 3–9 and a new `crunchcycles_dw` warehouse database you create for this project.

---

## Part 1 — The warehouse

Build `schema.sql` covering, at minimum:

- `raw.regions`, `raw.employees`, `raw.customers`, `raw.products`, `raw.orders`, `raw.order_items` — untouched mirrors of the operational tables, each with a `loaded_at` timestamp.
- `warehouse.dim_date` — generated, covering at least 2023–2026, with year/quarter/month/day-of-week/is_weekend.
- `warehouse.dim_customer`, `warehouse.dim_product`, `warehouse.dim_employee` — denormalized, surrogate-keyed, per Lecture 2.
- `warehouse.fact_order_items` — grain: one row per order line item, with a stated grain comment above the `CREATE TABLE`, degenerate dimensions (`order_id`, `order_status`) on the fact row, and a `UNIQUE (order_id, product_id)` constraint enabling the upsert load.

You may reuse Lecture 2's reference DDL directly, or ship the refined design from your Exercise 1 comparison — either is acceptable as long as `KPIs.md`'s queries actually run against whatever you built.

---

## Part 2 — The ELT job

Extend Exercise 2's `elt_warehouse_load.py` into the version that ships with this project:

- **Extract + Load** every operational table into `raw`, idempotently (full-replace each run, per Exercise 2 Task 2).
- **Transform** dimensions with a full rebuild each run (`TRUNCATE ... RESTART IDENTITY CASCADE` + `INSERT`), in the correct order relative to the fact load.
- **Transform** the fact table with an `ON CONFLICT (order_id, product_id) DO UPDATE` upsert — re-running the job must never change `fact_order_items`'s row count if the source data hasn't changed, and must correctly update a row whose source `status` changed since the last run.
- **Log every run** — start, per-table row counts, outcome — in the style Week 7 Lecture 3 established for the `fx_rates` job.
- **Let genuine failures raise loudly.** A missing source table or a broken database connection should crash the script with a clear error, not silently produce an empty warehouse.

---

## Part 3 — The dashboard

Build `dashboard.py` delivering **4 to 6 KPIs**, each meeting every one of these bars (Lecture 3, throughout):

- Tied to a stated business goal, named explicitly in `KPIs.md` (reuse or adapt Lecture 3 §2's example goals, or state your own).
- Passes the "would a 20% move change what someone does" test.
- At least **two** of your KPIs must be rates or margins (like gross margin % or repeat customer rate), not raw counts — Lecture 3 §4's vanity-metric warning applies directly here.
- At least **one** KPI must show a trend over time (month-over-month or quarter-over-quarter), not a single snapshot number — per Lecture 3 §5's "never a cherry-picked window" rule.
- Every revenue-based KPI explicitly filters `order_status = 'Completed'`, stated in a comment above the query.
- Every chart's y-axis starts at zero (or explicitly annotates why it doesn't, if you have a specific, defensible reason to zoom) — no silent truncation.

`dashboard.py` must run standalone (`python3 dashboard.py`) and produce: printed KPI values to stdout, and at least 3 `.png` chart files saved to disk.

---

## Milestones

- **Milestone 1 (45 min):** `schema.sql` created and run against a fresh `crunchcycles_dw`; `raw` tables populated by a first pass of the ELT job.
- **Milestone 2 (60 min):** Dimensions and fact table transforms complete; two consecutive ELT runs produce identical row counts (idempotency proven).
- **Milestone 3 (45 min):** 4–6 KPIs defined in `KPIs.md`, queries written and validated against the populated warehouse.
- **Milestone 4 (30 min):** `dashboard.py` complete, charts generated, `run-log.md` and `README.md` written, clean end-to-end run captured.

---

## Rules

- **The ELT job must be genuinely re-runnable.** If running it twice produces a different row count in any `warehouse.*` table than running it once, the project isn't done — this is the same non-negotiable bar Week 7's mini-project set for its ETL job.
- **Every KPI must trace to a stated goal.** A KPI in `KPIs.md` with no goal sentence above it doesn't count toward the 4–6 required.
- **No cumulative-only charts.** If a chart shows a running total, a matching non-cumulative (rate or period) chart must accompany it — Challenge 2's lesson, enforced here.
- **No secrets committed.** Database URLs/credentials live in a `.env` file, `.gitignore`'d.

---

## Rubric

| Criterion | Weight | "Great" looks like |
|-----------|------:|---------------------|
| Star schema design | 20% | Correct grain stated explicitly; surrogate keys; sensible denormalization; `dim_date` correctly generated |
| ELT correctness & idempotency | 25% | Raw and warehouse layers cleanly separated; job is genuinely re-runnable, proven with row counts in `run-log.md` |
| KPI quality | 20% | Every KPI passes the "20% move" test; at least 2 rates, at least 1 trend; filters stated explicitly |
| Dashboard honesty | 15% | Zero-based axes, no bare cumulative charts, trends shown with real time context |
| Error handling & logging | 10% | ELT job logs every run and fails loudly on genuine errors |
| Documentation | 10% | `KPIs.md` and `README.md` let a stranger understand and run the whole system in under 15 minutes |

---

## Reflection (`notes.md`, ~250 words)

1. Which of your KPIs was hardest to define precisely enough to trust — where did "obviously what we mean" turn out to hide an ambiguous filter or denominator once you tried to write the exact SQL?
2. If Crunch Cycles' data volume grew 1,000x, which part of this week's design (the ELT job's full-dimension-rebuild pattern, the single-database warehouse, the hand-written SQL transforms) would you change first, and what would you reach for instead (name a real tool from `resources.md` if relevant)?
3. Look back at your dashboard with Lecture 3 §6's five trust questions. Pick the KPI you're least confident survives all five, and say honestly why.
4. Now that Crunch Cycles has a governed, secured system (Week 9) and an honest analytics layer (this week), what's the first AI-assisted feature you'd want to build on top of it — and what about this week's warehouse specifically would make that feature possible that wouldn't have been, querying the operational database directly?

---

## Why this matters

This is the shape of a real analytics engineer's sprint: model the data around the business, not the source system; move it reliably, provably, on a schedule; and hand decision-makers numbers they can actually trust, including the ones that don't look great. Every skill from this week shows up here at once — grain, star schema, ELT idempotency, KPI discipline, dashboard honesty — because in production they're never separable. Keep this project; Week 11 builds AI-assisted features on top of exactly this warehouse.

When done: push, then take the [quiz](../quiz.md) and start [Week 11 — AI integration](../../week-11-ai-integration/).
