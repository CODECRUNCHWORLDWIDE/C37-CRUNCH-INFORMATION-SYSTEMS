# OLTP vs. OLAP & Data Warehouses

Crunch Cycles' `crunchcycles` Postgres database, as it stands after Week 9, is good at one job: recording a single fact — one order, one line item, one login — correctly, right now, while a hundred other things might be happening to the same tables at the same instant. This lecture is about the moment that job stops being the only job anyone asks of the database, and why the honest answer to "can't we just run the report against production?" is *no, and here's exactly what breaks if you do*.

## 1. Two workloads, two different jobs

**OLTP — Online Transaction Processing** is what `crunchcycles` does today: many short, simple operations, each touching a few rows, each needing to finish in milliseconds, arriving continuously and unpredictably. A checkout inserting one row into `orders` and a few into `order_items`. An employee's login updating `app_users.last_login_at`. A support rep looking up one customer by `customer_id`. OLTP systems are optimized for **write speed, row-level locking correctness, and point lookups** — an index on `customer_id` that finds one row instantly is exactly what this workload needs.

**OLAP — Online Analytical Processing** is a different job entirely: few, large, complex queries — "total revenue by region by quarter for the last two years" — each scanning and aggregating millions of rows, run by a handful of people (analysts, executives), not thousands of customers. OLAP systems are optimized for **read speed over huge scans, aggregation, and historical range**, not for handling a checkout the instant it happens.

| | OLTP (operational) | OLAP (analytical) |
|---|---|---|
| Typical query | `SELECT * FROM orders WHERE order_id = 482` | `SELECT region, SUM(revenue) FROM ... GROUP BY region, quarter` |
| Rows touched per query | A handful | Millions, potentially |
| Query frequency | Constant, from many users/systems | Occasional, from a handful of analysts |
| Optimized for | Fast, correct writes; row-level locks | Fast, large scans and aggregation |
| Data shape | Normalized (Week 3) — minimal duplication | Denormalized (this week) — duplication traded for speed |
| Data horizon | "Right now" — current state | Historical — years of trend |
| Who queries it | The application, on behalf of customers | Analysts, executives, dashboards |

Crunch Cycles' `crunchcycles` database is, and should remain, a pure OLTP system. It is normalized (Week 3) specifically so that an order's data is never duplicated and never goes stale in two places at once — exactly the property a checkout needs. That same property is what makes it slow and awkward for analytics: answering "revenue by region by quarter" against a normalized OLTP schema means joining `orders` → `order_items` → `products` → `customers` → `regions` every single time, scanning the *entire* `orders` and `order_items` tables to compute a historical aggregate — for every report, every refresh, every dashboard click.

## 2. The question, asked against production, made concrete

Here's "revenue by region by quarter," written the only way it *can* be written against the normalized `crunchcycles` schema — five tables, joined every single time the question is asked:

```sql
SELECT
    r.region_name,
    EXTRACT(YEAR FROM o.order_date)    AS year,
    EXTRACT(QUARTER FROM o.order_date) AS quarter,
    SUM(oi.quantity * oi.unit_price)   AS revenue
FROM orders o
JOIN order_items oi ON o.order_id = oi.order_id
JOIN customers c     ON o.customer_id = c.customer_id
JOIN regions r        ON c.region_id = r.region_id
WHERE o.status = 'Completed'
GROUP BY r.region_name, EXTRACT(YEAR FROM o.order_date), EXTRACT(QUARTER FROM o.order_date)
ORDER BY r.region_name, year, quarter;
```

At Crunch Cycles' current 40 orders this runs instantly — there's nothing to feel yet. Follow the query plan mentally, though, and the future problem is already visible: with no index built for "group by region and quarter," Postgres has to scan `order_items` and `orders` in full, join them row by row against `customers` and `regions`, extract a date part from every single row, *then* aggregate. `EXPLAIN ANALYZE` on this exact query will show a sequential scan today; at 4 million orders instead of 40, that same sequential scan is the difference between a report that returns in milliseconds and one that takes minutes — run by every analyst, every time they refresh a dashboard, on the same tables the checkout flow is trying to write to.

## 3. What actually breaks when you point BI at production

This isn't theoretical caution — it's three concrete failure modes, in increasing order of how bad they get as the business grows:

1. **Lock contention.** A long-running analytical `SELECT` like the one above can hold read locks or consume I/O bandwidth that a checkout's `INSERT` is competing for. At Crunch Cycles' current size this might just mean a slow page load; at real scale it means checkouts timing out because someone's dashboard refreshed at the wrong moment. "Just add an index" helps the *speed* problem but does nothing for problems 2 and 3 below — it's a partial fix to the least important of the three.

2. **The schema fights the question.** OLTP schemas are normalized for write integrity, not shaped around the questions a business asks. "Revenue by region by quarter" requires re-deriving the same five-table join and the same date-bucketing logic every time, in every report, by every analyst — with every risk of one of them getting the join wrong and quietly producing a different number than everyone else's version of "the same" report. Two analysts, two independently-written versions of the query above, two subtly different revenue numbers in two different slide decks, and now a meeting spent debating whose number is right instead of what to do about it.

3. **There's no history.** `orders.status` holds today's status. If a customer's region changed last year, or a product's price changed six months ago, the operational database shows you the *current* value applied retroactively to old rows unless you went out of your way to snapshot it — which OLTP systems, correctly, don't do by default. Run the query above *today* and every historical order gets grouped by each customer's **current** region, not the region they were in when they placed that order. Answering "what was Q1's revenue, valued at Q1's prices, attributed to each customer's Q1 region" is not just slow against an OLTP schema — it may be **impossible**, because the history was never captured. Lecture 2 §5 (slowly changing dimensions) is the warehouse's answer to exactly this gap.

A data warehouse exists to solve exactly these three problems, on purpose, in a separate system.

## 4. Who should even be allowed to run this query

Week 9 built role-based access control and row-level security around `crunchcycles` for a reason: every additional person with a live connection to the operational database is another person who can, deliberately or not, run a query like the one above at the worst possible moment, or see a customer's full order history who has no operational reason to. A warehouse isn't just a performance fix — it's an access-control boundary that fits naturally on top of Week 9's governance work: give analysts read access to `crunchcycles_dw`, never to `crunchcycles` directly, and the two problems (accidental production load, over-broad access to live operational data) are solved by the same architectural decision. This is exactly the kind of design choice that looks like "just for performance" on the surface and turns out to be a security and governance win underneath.

## 5. Row-oriented vs. column-oriented storage (why real OLAP engines are built differently, not just used differently)

Everything above is about *modeling* — what shape the data should take. There's also a physical-storage reason OLTP and OLAP engines diverge, worth understanding even though this course builds the warehouse in plain row-oriented PostgreSQL.

A **row-oriented** database (PostgreSQL, MySQL, the `crunchcycles` OLTP store) physically stores each row's columns together on disk — reading `orders` row 482 means one seek, one read, and every column of that one order comes back together. That's exactly right for `SELECT * FROM orders WHERE order_id = 482`: a point lookup, retrieving all of one row.

A **column-oriented** ("columnar") database (Snowflake, BigQuery, Redshift's storage layer, and Postgres extensions like `cstore_fdw`) physically stores each *column* together instead — every order's `order_date` sits contiguously on disk, separate from every order's `status`, separate from every order's `customer_id`. That's exactly right for `SELECT region, SUM(revenue) FROM ... GROUP BY region`: an aggregate query only touches two or three columns out of a table that might have twenty, and columnar storage lets the engine skip reading the other seventeen entirely, plus compress each column far more effectively (a column of a handful of repeating `region_name` values compresses beautifully; a row of twenty different values, much less so).

| | Row-oriented (OLTP) | Column-oriented (OLAP) |
|---|---|---|
| Best at | Point lookups, single-row reads/writes | Aggregating one or two columns across millions of rows |
| Worst at | Scanning + aggregating a few columns across a huge table | Fetching every column of one specific row |
| Compression | Modest (mixed data types per row) | Strong (uniform data type per column) |
| Example engines | PostgreSQL, MySQL (as used this week) | Snowflake, BigQuery, Redshift |

This course's `crunchcycles_dw` stays in plain row-oriented PostgreSQL, deliberately — the dimensional-modeling skills transfer directly to a columnar engine, and at Crunch Cycles' actual data volume the difference is invisible. The name to recognize, for the day your warehouse *does* outgrow a single row-oriented Postgres instance: that's the point at which "migrate the physical storage engine" becomes worth doing, and it's a storage-layer change, not a schema-design change — everything Lecture 2 teaches about facts, dimensions, and grain applies unchanged on either kind of engine.

## 6. What a data warehouse is

Bill Inmon's classic definition, still the clearest one: a data warehouse is **subject-oriented, integrated, time-variant, and non-volatile**.

- **Subject-oriented** — organized around business concepts (customers, sales, products), not around the applications that produced the data. Crunch Cycles' warehouse is organized around "sales," not around "whatever tables the checkout app happens to have."
- **Integrated** — data from possibly multiple source systems (the OLTP database today; maybe a marketing tool or a support ticketing system later) is combined into one consistent model with one consistent vocabulary. "Customer" means the same thing everywhere in the warehouse, even if two source systems spelled the country differently (Challenge 1, this week).
- **Time-variant** — the warehouse keeps history. A row for January's revenue doesn't get overwritten by February; it accumulates. Some dimensions even track *how an attribute itself changed over time* (slowly changing dimensions, Lecture 2).
- **Non-volatile** — once data lands in the warehouse, it isn't updated in place the way an operational row is; it's appended to or explicitly reprocessed on a schedule, not mutated by a live checkout in real time.

None of that describes `crunchcycles`. All of it describes the second database this week builds: `crunchcycles_dw`.

## 7. Why a genuinely separate store, not just a read replica

A tempting middle ground: keep one schema, just point reports at a **read replica** — a continuously-synced, read-only copy of the OLTP database, a genuinely useful and common piece of infrastructure — instead of building a whole new warehouse. A read replica solves problem #1 from §3 (lock contention: the replica takes the analytical load, the primary keeps serving checkouts undisturbed) but *not* problems #2 and #3 — the replica is still normalized for writes, still has no history beyond what the primary currently holds, and still forces every report to re-derive the same five-table join with the same date-bucketing logic, on a schema shaped for transactions rather than for questions. A read replica is a reasonable *addition* at real scale (and worth knowing the term — you will meet it at a job), but it is not a substitute for a warehouse's different **data model**, which is this week's actual subject, starting with Lecture 2.

There's also a middle layer worth naming, since it sits between "query production directly" and "build a full warehouse": **operational reporting** — a handful of pre-built, cached, or materialized views answering the two or three questions the business asks daily, refreshed on a short cycle (every few minutes), still against the operational schema's shape. It's a reasonable stopgap for a genuinely small set of stable questions. It stops working the moment someone asks a *new* question, because the underlying schema still isn't organized around business concepts — which is exactly the gap a warehouse's dimensional model (Lecture 2) closes permanently, not question by question.

That's why Crunch Cycles gets a second, physically separate Postgres database, `crunchcycles_dw`, holding data **shaped differently on purpose**: denormalized, historical, and organized around business questions instead of transactional integrity.

**How fresh does the warehouse need to be?** Not "as fresh as the checkout," almost always. A dashboard showing yesterday's numbers, refreshed by an ELT job that runs once overnight (the pattern Exercise 2 and the mini-project build), is the right default for a business Crunch Cycles' size — nobody making a quarterly regional-growth decision needs revenue numbers accurate to the second. Reserve near-real-time warehouse refreshes for the rare case where a decision genuinely can't wait until tomorrow morning; building for "instant" everywhere is expensive infrastructure solving a latency problem nobody actually has.

## 8. Warehouse architecture: three layers

Real warehouses — however big the company — tend to converge on the same three-layer shape. Crunch Cycles' warehouse uses it at a scale you can hold in your head:

```
┌─────────────────┐      ┌──────────────────┐      ┌─────────────────────┐
│  Source system   │ ──▶  │   raw / staging    │ ──▶  │  warehouse (marts)   │
│  crunchcycles     │      │  crunchcycles_dw.  │      │  crunchcycles_dw.    │
│  (OLTP, Week 3–9) │      │  raw schema        │      │  warehouse schema    │
│                  │      │                    │      │  star schema         │
└─────────────────┘      └──────────────────┘      └─────────────────────┘
     "what is true            "an untouched copy         "shaped for the
      right now"                of the source,             business question,
                                 timestamped"                fast to query"
```

- **Source** — the operational system(s). Just `crunchcycles` for now; in a bigger company this layer might include a payments provider, a marketing tool, and three other databases.
- **Raw / staging** — a landing zone that holds an as-close-to-untouched copy of the source data, with a `loaded_at` timestamp. Nothing is cleaned or reshaped here — its entire job is to be a faithful, disposable copy you can always re-derive the warehouse from. This is the **E** and the first **L** of ELT.
- **Warehouse (marts)** — the dimensional model: facts and dimensions, built with SQL *from* the raw layer, denormalized and shaped around business questions. This is the **T** — transform happens *inside the destination*, using the destination's own SQL engine, which is what makes this pattern **ELT** rather than Week 7's **ETL**.

## 9. ETL vs. ELT, revisited for a warehouse

Week 7 (Lecture 3) covered ETL: extract, transform in Python/pandas, load an already-clean table. That's still the right pattern for a small, single-purpose feed like the `fx_rates` table.

A warehouse is where **ELT** earns its keep instead. Reasons this pattern fits a warehouse specifically, that didn't apply to a single small feed:

- **Storage is cheap; re-deriving is not.** Landing the raw data first means if you discover a bug in your transform logic next month, you fix the SQL and re-run the transform against data you *already have* — you don't need to re-extract from the source (which might rate-limit you, or might not even have that old data anymore).
- **The raw layer has independent value.** A new report nobody anticipated when the warehouse was designed can often be answered by writing new SQL against the raw layer, without touching the pipeline code at all.
- **SQL is often the right tool for the transform itself.** Joining five tables and computing a fact table's grain is exactly what a relational database engine is built to do well — pushing that work into the destination's own SQL engine, instead of pulling rows into Python to do the same joins by hand, plays to the tool's strength.

You'll build exactly this in Exercise 2: Python does the **E** (extract from `crunchcycles`) and the first **L** (load, untouched, into `raw`); SQL, run *inside* `crunchcycles_dw`, does the **T** (transform `raw` into `warehouse`'s star schema).

## 10. Naming the tools you'll meet outside this course

This week builds everything in plain PostgreSQL, deliberately — the concepts transfer directly to whatever your first job uses. Three names worth recognizing when you see them:

- **Cloud data warehouses** — Snowflake, Google BigQuery, Amazon Redshift. Same dimensional-modeling concepts this week teaches, at a scale of billions of rows, with the compute and storage separated so either can scale independently. Out of scope to install here; the star schema you design this week is the same shape you'd build in any of them.
- **dbt (data build tool)** — the industry-standard way to write the "T" of ELT as version-controlled, testable SQL files instead of one long hand-written script. This week's `warehouse/*.sql` transform files are, deliberately, a hand-rolled miniature of what dbt automates at scale.
- **BI tools** — Metabase, Superset, Looker, Tableau, Power BI. Point-and-click (or lightly coded) layers that sit on top of a warehouse and let non-engineers explore it and build dashboards. This week's dashboard (Lecture 3) is built in plain Python/SQL so you understand exactly what a BI tool is doing underneath its UI — `resources.md` links a free one worth installing once this week is done.

## What's next

Lecture 2 designs the shape of `crunchcycles_dw.warehouse` itself: what a fact table is, what a dimension is, how to choose a fact table's grain without regretting it later, and how to build the whole thing in SQL. Lecture 3 turns that warehouse into KPIs and a dashboard a Crunch Cycles executive can actually trust.
