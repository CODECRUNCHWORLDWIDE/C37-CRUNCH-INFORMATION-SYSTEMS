# Week 10 — Quiz

Fifteen questions. Lectures closed. Aim for 13/15 before starting Week 11. A mix of multiple-choice and short "what happens here?" — the answer key at the bottom explains the *why*, not just the letter.

---

**Q1.** Which best describes why running analytics queries directly against `crunchcycles` (the OLTP database) is a bad long-term plan?

- A) PostgreSQL cannot run `GROUP BY` queries at all
- B) OLTP schemas are normalized for write integrity and lack history; large analytical scans can also contend with live transactional writes
- C) Analytics queries are illegal to run against a production database
- D) OLAP and OLTP use entirely different SQL syntax

---

**Q2.** A data warehouse, per Inmon's classic definition, is subject-oriented, integrated, time-variant, and:

- A) Volatile — updated in place constantly, like an OLTP table
- B) Non-volatile — data is appended/reprocessed, not mutated in place by live transactions
- C) Schema-less — no fixed table structure
- D) Single-source — it may only ever hold data from one system

---

**Q3.** In this week's ELT pattern, where does the **transform** step happen?

- A) In Python, before any data is loaded anywhere (this is ETL, not ELT)
- B) Inside the destination warehouse, using SQL, after raw data has already landed
- C) On the client's browser
- D) Transform never happens in ELT — only extract and load

---

**Q4.** A fact table's **grain** is:

- A) The database engine it's stored in
- B) A precise statement of what one row of the fact table represents
- C) The number of columns in the table
- D) A synonym for "primary key"

---

**Q5.** Crunch Cycles' `fact_order_items` is built at the grain of one row per order line item rather than one row per order. What's lost if you had instead chosen one row per order (aggregated)?

- A) Nothing — it's always safe to disaggregate later
- B) The ability to answer "revenue by product," since a multi-item order would be a single row with no per-product breakdown
- C) The ability to compute total revenue at all
- D) The primary key would no longer be unique

---

**Q6.** In a star schema, why does this week's `dim_customer` fold `region_name` directly into the table instead of joining to a separate `dim_region` table?

- A) PostgreSQL doesn't support joins between dimension tables
- B) It denormalizes on purpose — fewer joins, faster and simpler queries for analysts, at the cost of some repeated data
- C) `dim_region` is a reserved table name in PostgreSQL
- D) Region data changes too often to have its own table

---

**Q7.** Why does every dimension table use a warehouse-generated **surrogate key** instead of reusing the operational system's ID (e.g., `customer_id`) as the primary key?

- A) Surrogate keys are required by the SQL standard for all analytical tables
- B) It decouples the warehouse from a single source's ID scheme, supports multiple source systems cleanly, and is required to support Type 2 slowly changing dimensions
- C) Natural keys can't be integers
- D) It makes the warehouse smaller

---

**Q8.** A **Type 1** slowly changing dimension update:

- A) Inserts a new versioned row and keeps the old one, preserving history
- B) Overwrites the existing dimension row in place, losing the prior value
- C) Is only used for fact tables, never dimensions
- D) Requires a separate `dim_date` table

---

**Q9.** Why does `warehouse.fact_order_items` store `unit_price` and `unit_cost` directly on the fact row, instead of always joining to `dim_product` for the current price?

- A) `dim_product` doesn't have a `unit_cost` column
- B) A product's price can change over time; the fact table must preserve the price actually charged at the time of that historical sale, not today's price
- C) It's faster to store redundant data even when it's wrong
- D) Fact tables are not allowed to reference dimension tables at all

---

**Q10.** In the ELT job's dimension-transform step, why is `TRUNCATE ... RESTART IDENTITY CASCADE` followed by a full `INSERT` used, rather than an incremental upsert like the fact table uses?

- A) `TRUNCATE` is faster in all cases, so it's always preferred
- B) Dimensions are small; a full rebuild each run from `raw`'s current state is simple and correct, and `RESTART IDENTITY` prevents surrogate keys from climbing forever across runs
- C) Upserts are not supported in PostgreSQL
- D) It's required because `raw` tables use `if_exists="append"`

---

**Q11.** `ON CONFLICT (order_id, product_id) DO UPDATE` on `fact_order_items` correctly handles which real scenario?

- A) A customer changing their email address
- B) An order whose `status` changes from `Pending` to `Completed` between two ELT runs — the existing fact row updates instead of a duplicate being inserted
- C) A brand-new product being added to the catalog
- D) The `dim_date` table running out of future dates

---

**Q12.** Which of these passes the "would a 20% move change what someone does" test for a real KPI?

- A) Total number of rows in `dim_customer`, all-time
- B) Repeat customer rate this quarter
- C) Number of columns in the `orders` table
- D) The warehouse database's disk size in megabytes

---

**Q13.** A chart shows **cumulative** total revenue climbing smoothly upward every single month, presented as proof of "strong growth." What's the core problem with this, even if every number in it is computed correctly?

- A) Cumulative sums are always mathematically wrong
- B) A cumulative sum of non-negative values can never decrease, so the chart cannot reveal a slowdown or decline in the underlying period-by-period rate — it looks the same whether growth is accelerating or stalling
- C) `matplotlib` cannot render cumulative data accurately
- D) There is no problem — cumulative charts are always the correct choice

---

**Q14.** For `AOV = SUM(extended_price) / COUNT(DISTINCT order_id)` against a fact table with a line-item grain, why must the denominator use `COUNT(DISTINCT order_id)` rather than `COUNT(*)`?

- A) `COUNT(*)` doesn't work in PostgreSQL
- B) The fact table's grain is one row per line item, so `COUNT(*)` counts line items, not orders — a multi-item order would inflate the row count and silently deflate AOV
- C) `COUNT(DISTINCT ...)` is always faster
- D) `order_id` cannot appear in a `COUNT` at all

---

**Q15.** `dim_customer` contains every customer who ever signed up, including some who never placed an order. A dashboard's "Total Customers" KPI counts every row in `dim_customer`. What's the specific problem with using this as a proxy for "the business is healthy and growing"?

- A) There is no problem — more rows in `dim_customer` always means more health
- B) It conflates signups with actual buying customers, using the wrong population/denominator — the same "wrong denominator" trap that repeat customer rate's definition was built specifically to avoid
- C) `dim_customer` cannot be queried with `COUNT`
- D) The problem is purely a matter of chart color choice, not the underlying number

---

## Answer key

<details>
<summary>Reveal after attempting</summary>

1. **B** — OLTP schemas are normalized for correctness/write-speed and hold current state only; large analytical scans can also lock or compete with live transactional writes, which is why a separate analytical store exists.
2. **B** — non-volatile means warehouse data is loaded/appended or explicitly reprocessed on a schedule, not mutated in place the way a live operational row is.
3. **B** — that's the defining difference from ETL: ELT loads raw data into the destination first, then transforms it there using the destination's own engine (SQL), rather than transforming in-flight before loading.
4. **B** — grain is the precise, one-sentence statement of what a single fact row represents; every other design decision (which measures, which dimensions) follows from it.
5. **B** — aggregating to one row per order collapses per-product detail; you could no longer answer "revenue by product" without re-deriving from a finer source, which is exactly why this week chose the finer, line-item grain instead.
6. **B** — this is deliberate denormalization: a star schema trades some repeated data for fewer joins, which is faster and simpler for the analysts and dashboards actually querying the warehouse.
7. **B** — surrogate keys decouple the warehouse from any one source's ID numbering (useful if a second source system is added later) and are required to support Type 2 SCDs, where the same real-world entity needs multiple warehouse rows.
8. **B** — Type 1 overwrites in place, losing the previous value; Type 2 is the pattern that preserves history by inserting a new versioned row instead.
9. **B** — prices change over time in the source system; the fact table must preserve what was actually charged at the moment of each historical sale, or historical revenue would silently restate itself every time today's price changes.
10. **B** — dimensions are small enough that a full rebuild each run is simple and correct; `RESTART IDENTITY` keeps surrogate key values from growing unbounded across repeated runs.
11. **B** — the natural key `(order_id, product_id)` lets the upsert detect "this line item already exists" and update its status/values in place instead of inserting a duplicate row when the source order's status changes between runs.
12. **B** — repeat customer rate is a rate tied to a real business concern (customer retention) that would prompt a concrete response if it moved sharply; the others are counts with no inherent connection to a decision anyone would make.
13. **B** — a cumulative (running total) chart of non-negative values is mathematically guaranteed to never decrease, so it cannot visually distinguish steady growth from a slowdown or a plateau in the underlying month-by-month rate.
14. **B** — at a line-item grain, `COUNT(*)` counts rows (line items), not distinct orders; a multi-item order would be counted multiple times, inflating the denominator and understating AOV.
15. **B** — using every `dim_customer` row (including never-purchased signups) as "total customers" mixes two different populations together and doesn't measure actual buying/retention health — the same wrong-denominator problem the repeat-customer-rate KPI is explicitly designed to avoid.

</details>

**Scoring:** 13+ → start Week 11. 10–12 → re-read the lecture sections behind your misses, especially grain and idempotency. <10 → re-read all three lectures from the top; this week's dimensional-modeling concepts underpin every later analytics or AI feature you'll build on this data.
