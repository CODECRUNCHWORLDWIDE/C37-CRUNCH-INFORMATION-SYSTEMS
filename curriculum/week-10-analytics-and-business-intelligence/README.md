# Week 10 — Analytics & Business Intelligence

> **Goal:** by Sunday you can explain, precisely, why Crunch Cycles' operational database is the wrong place to run a "revenue by region by quarter" query; you can design and build a star-schema data warehouse that answers that query in milliseconds; you can write an idempotent ELT job that keeps it fresh from the operational store; and you can hand a stakeholder a dashboard of KPIs that tells the truth, including the parts of the truth that don't look great in a slide deck.

Welcome back to **C37 · Crunch Information Systems**. Weeks 1–9 built Crunch Cycles into a real system: a normalized schema (Week 3), joins and pipelines to query it (Week 4), an API and ELT integrations (Week 7), a cloud deployment (Week 8), and access control with a governance policy (Week 9). That system is good at exactly one job: recording one order, one customer, one login at a time, correctly, right now. Ask it "what was our gross margin by region for each of the last six quarters" and it will either take forever (scanning and joining millions of rows nobody indexed for that question) or get in the way of the checkout flow trying to answer it (locking rows that a live order needs). This week you stop asking the operational database questions it was never built to answer, and instead build the system that was: a **data warehouse**.

By Saturday you will have a second Postgres database — `crunchcycles_dw` — holding a dimensional model (facts and dimensions, in a star schema) fed by a repeatable ELT job from the operational store, and a small dashboard delivering KPIs a Crunch Cycles executive would actually trust, because you can show your work behind every number.

## Learning objectives

By the end of this week, you will be able to:

- **Distinguish** OLTP (operational, transactional) systems from OLAP (analytical) systems and data warehouses — what each is optimized for, and why running analytics against the OLTP database directly eventually breaks both the analytics and the operations.
- **Model** analytical data with **facts** and **dimensions**, choose a fact table's **grain** deliberately, and build a **star schema** in SQL — including surrogate keys, a conformed `dim_date`, and a first working understanding of slowly changing dimensions.
- **Build an ELT pipeline** that extracts from the operational store, lands raw data in a landing zone, and transforms it *inside the warehouse* with SQL — the "load-then-transform" pattern, and how it differs from Week 7's ETL.
- **Define KPIs** that are tied to an actual business goal, distinguish them from vanity metrics that look good but decide nothing, and compute them correctly against a star schema.
- **Build a BI dashboard** — a small, honest, reproducible report — that answers real questions about Crunch Cycles' business without hiding an inconvenient trend behind a flattering chart type.

## Prerequisites

- Weeks 1–9 of this course, or equivalent comfort with: a normalized PostgreSQL schema (Week 3), SQL joins/aggregation/CTEs and `pandas`/SQLAlchemy pipelines (Week 4), and an ELT job with idempotent upserts (Week 7).
- Python 3.10+ with `pip` working, and the PostgreSQL 16+ `crunchcycles` database from earlier weeks (schema reproduced below if you need it fresh).
- Comfort running `psql`, reading a `CREATE TABLE` statement, and writing a `GROUP BY`/`HAVING` query. No prior data-warehousing, BI-tool, or "big data" background required — this week teaches the concepts from first principles.

## Setup

**1. Confirm your operational database.** If your `crunchcycles` Postgres database from earlier weeks is still around, you're set. If not, recreate the Week 4 schema (`regions`, `employees`, `customers`, `products`, `orders`, `order_items`) from that week's README, then continue below.

**2. Create the warehouse — a genuinely separate database.**

```bash
createdb crunchcycles_dw
psql crunchcycles_dw
```

```sql
CREATE SCHEMA IF NOT EXISTS raw;        -- landing zone: raw copies from the operational store
CREATE SCHEMA IF NOT EXISTS warehouse;  -- the star schema: facts and dimensions
```

Two separate Postgres **databases** (not just schemas) is deliberate, not overkill — Lecture 1 explains exactly why a warehouse belongs on its own store, physically, even at Crunch Cycles' small scale. `raw` and `warehouse` as two separate **schemas** inside `crunchcycles_dw` is the ELT landing-zone pattern this week teaches: extract and load first, transform second, inside the destination.

**3. Install the Python side (adds one package to Week 7's toolkit).**

```bash
python3 -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install pandas sqlalchemy psycopg2-binary python-dotenv matplotlib
```

`matplotlib` is new this week — it renders the charts your dashboard script produces. Everything else you've already used since Week 4.

**4. Sanity-check both connections.**

```bash
psql crunchcycles -c "SELECT COUNT(*) FROM orders;"       -- expect 40
psql crunchcycles_dw -c "\dn"                              -- expect raw, warehouse (+ public)
```

## Weekly schedule

Adds up to the course's full-time target of **~28 hours**.

| Day | Focus | Lectures | Exercises | Challenges | Quiz/Read | Homework | Mini-Project | Daily Total |
|-----------|--------------------------------------------------|---------:|----------:|-----------:|----------:|---------:|-------------:|------------:|
| Monday | OLTP vs. OLAP, why warehouses exist | 2h | 1h | 0h | 0.5h | 1h | 0h | 4.5h |
| Tuesday | Dimensional modeling: facts, dims, grain | 2h | 1.5h | 0h | 0.5h | 1h | 0h | 5h |
| Wednesday | Build the star schema in SQL | 0h | 1.5h | 1h | 0.5h | 1h | 0h | 4h |
| Thursday | ELT job: raw → warehouse, idempotent | 0h | 1.5h | 1h | 0.5h | 1h | 1h | 5h |
| Friday | KPIs, vanity metrics, honest dashboards | 2h | 1h | 1h | 0.5h | 1h | 1.5h | 7h |
| Saturday | Mini-project (warehouse + ELT + dashboard) | 0h | 0h | 0h | 0h | 0h | 2.5h | 2.5h |
| Sunday | Quiz + review | 0h | 0h | 0h | 1h | 0h | 0h | 1h |
| **Total** | | **4h** | **5.5h** | **3h** | **3.5h** | **5h** | **5h** | **~29h** |

## How to navigate this week

Work top to bottom. Each piece assumes the ones above it.

| # | File | What's inside | ~Time |
|--:|------|---------------|------:|
| 1 | [lecture-notes/01-oltp-vs-olap-and-warehouses.md](./lecture-notes/01-oltp-vs-olap-and-warehouses.md) | OLTP vs. OLAP workloads, why a shared store breaks both, warehouse architecture | 2h |
| 2 | [lecture-notes/02-dimensional-modeling.md](./lecture-notes/02-dimensional-modeling.md) | Facts, dimensions, grain, star vs. snowflake, surrogate keys, slowly changing dimensions | 2h |
| 3 | [lecture-notes/03-kpis-and-dashboards.md](./lecture-notes/03-kpis-and-dashboards.md) | Choosing real KPIs, spotting vanity metrics, building a dashboard people can trust | 2h |
| 4 | [exercises/exercise-01-design-a-star-schema.md](./exercises/exercise-01-design-a-star-schema.md) | Design and document a star schema for Crunch Cycles sales | 1.5h |
| 5 | [exercises/exercise-02-build-an-elt-job.md](./exercises/exercise-02-build-an-elt-job.md) | Build the raw → warehouse ELT job, idempotent | 2h |
| 6 | [exercises/exercise-03-define-kpis.md](./exercises/exercise-03-define-kpis.md) | Define and compute KPIs tied to a stated business goal | 1.5h |
| 7 | [challenges/challenge-01-warehouse-a-messy-source.md](./challenges/challenge-01-warehouse-a-messy-source.md) | Clean and warehouse a deliberately messy legacy export | 1.5h |
| 8 | [challenges/challenge-02-expose-a-misleading-metric.md](./challenges/challenge-02-expose-a-misleading-metric.md) | Find and fix a dashboard metric that's technically correct and practically misleading | 1.5h |
| 9 | [mini-project/README.md](./mini-project/README.md) | Build the warehouse, the ELT job, and a 4–6 KPI dashboard | 2.5h |
| 10 | [homework.md](./homework.md) | Extra practice, spaced out | 5h |
| 11 | [quiz.md](./quiz.md) | 15 self-check questions + answer key | 1h |
| 12 | [resources.md](./resources.md) | Official docs, standards, and tools to install | — |

## By the end of this week you can…

- Explain, to a non-technical stakeholder, why "just query the production database for the report" is a bad idea — and what breaks when someone tries it anyway.
- Design a star schema from scratch: pick a grain, name the facts and dimensions, and defend both choices out loud.
- Write SQL that builds `dim_date`, conformed dimensions, and a fact table from an operational schema — with surrogate keys, not natural keys, as the join targets.
- Build an ELT job that lands raw data untouched, then transforms it inside the warehouse with SQL, and re-run it without duplicating a single row.
- Define a KPI in one sentence that ties it to a business goal, and tell it apart from a vanity metric that just goes up and to the right.
- Build a dashboard that shows a trend honestly — including a declining one — instead of picking the chart type that hides it.

## Up next

[Week 11 — AI integration](../week-11-ai-integration/) — now that Crunch Cycles' data tells the truth reliably, we add AI as responsible decision support on top of it.

---

*Part of the Code Crunch Worldwide open curriculum · GPL-3.0 · If you find errors, please open an issue or PR.*
