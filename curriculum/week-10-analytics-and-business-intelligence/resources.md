# Week 10 — Resources

Free, public, no signup unless noted. Read the "required" set; treat the rest as reference you dip into when a specific question comes up.

## Install first

- **PostgreSQL 16+** — you already have this from earlier weeks. Confirm: `psql --version`.
- **Python 3.10+** with `pip` — also already installed. Confirm: `python3 --version`.
- **pandas / SQLAlchemy / psycopg2-binary** — the ELT toolkit reused from Weeks 4 and 7: `pip install pandas sqlalchemy psycopg2-binary`
- **matplotlib** — new this week, renders the dashboard's charts: `pip install matplotlib`
- **python-dotenv** — for loading database URLs from a local `.env` file instead of hard-coding them: `pip install python-dotenv`

## Required reading (this week's core)

- **Kimball Group — Dimensional Modeling Techniques:** <https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/>
  *Why: the free, canonical reference for facts, dimensions, grain, and star schemas — written by the person who invented the technique this week teaches.*
- **Kimball Group — Star Schema: Fact and Dimension Tables:** <https://www.kimballgroup.com/2003/01/design-tip-42-combining-periodic-and-accumulating-snapshots/> (and browse the adjacent "Design Tips" archive)
  *Why: short, practitioner-written notes on real modeling decisions — grain, degenerate dimensions, and conformed dimensions among them.*
- **PostgreSQL — `GENERATE_SERIES`:** <https://www.postgresql.org/docs/current/functions-srf.html>
  *Why: the exact reference for the function this week's `dim_date` generation relies on.*
- **PostgreSQL — Window Functions:** <https://www.postgresql.org/docs/current/tutorial-window.html>
  *Why: `LAG()`, used in the month-over-month growth KPI, is a window function — this is the official tutorial if the concept is new.*

## Reference (keep in tabs)

- **PostgreSQL — `INSERT ... ON CONFLICT`:** <https://www.postgresql.org/docs/current/sql-insert.html#SQL-ON-CONFLICT>
  *Why: the exact upsert syntax used in this week's fact-table ELT load — the same pattern Week 7 introduced, applied here to a warehouse.*
- **PostgreSQL — `TRUNCATE`:** <https://www.postgresql.org/docs/current/sql-truncate.html>
  *Why: exact semantics of `RESTART IDENTITY` and `CASCADE`, used in the dimension-rebuild step of the ELT job.*
- **pandas — `read_sql` / `to_sql`:** <https://pandas.pydata.org/docs/reference/api/pandas.read_sql.html>
  *Why: the exact reference for the Extract/Load steps of this week's ELT job, reused from Week 4/7.*
- **matplotlib — pyplot tutorial:** <https://matplotlib.org/stable/tutorials/pyplot.html>
  *Why: everything needed to build this week's dashboard charts, in official-docs form.*

## On data warehousing concepts specifically

- **Martin Fowler — Data Warehouse:** <https://martinfowler.com/bliki/DataWarehouse.html>
  *Why: a short, clear industry explanation of what a warehouse is and isn't, from a widely-respected software-architecture writer.*
- **Google Cloud — What is ETL?** <https://cloud.google.com/learn/what-is-etl>
  *Why: a vendor-written but even-handed explainer of ETL vs. ELT, useful alongside Week 7 Lecture 3 and this week's Lecture 1.*
- **AWS — What's the Difference Between OLTP and OLAP?** <https://aws.amazon.com/compare/the-difference-between-olap-and-oltp/>
  *Why: another vendor's framing of the same OLTP/OLAP distinction — reading two independent explanations of the same idea sharpens it.*

## On KPIs and dashboards

- **Avinash Kaushik — Web Analytics: An Hour a Day (excerpt on vanity metrics):** search "Avinash Kaushik vanity metrics" — his blog Occam's Razor (<https://www.kaushik.net/avinash/>) covers this extensively and freely.
  *Why: one of the most-cited writers on separating actionable metrics from vanity metrics — directly informs Lecture 3 §4.*
- **Edward Tufte — The Visual Display of Quantitative Information (principles summarized in many free talks/interviews):** search "Tufte chart junk" and "Tufte truncated axis."
  *Why: the foundational thinking behind "never truncate a y-axis without saying so" — the book itself isn't free, but Tufte's core principles are widely and freely summarized and referenced.*
- **Stephen Few — Dashboard Design: Beyond Meters, Gauges, and Traffic Lights** (free PDF, various analytics-community mirrors — search the title): 
  *Why: a practitioner's concrete critique of common dashboard anti-patterns, going well beyond what this lecture covers.*

## Free BI tools worth trying (optional this week, referenced in Lecture 1 §7)

- **Metabase** (open source, free, self-hosted or free-tier cloud): <https://www.metabase.com/>
  *Why: the fastest way to point a real BI tool at `crunchcycles_dw` and see what it automates on top of the SQL you wrote by hand this week — install it after finishing the mini-project, not instead of it.*
- **Apache Superset** (open source, free, self-hosted): <https://superset.apache.org/>
  *Why: a heavier, more customizable open-source alternative to Metabase, used at real companies — worth knowing by name even if you don't install it this week.*

## Practice beyond this week's exercises

- **Kimball Group — "The Data Warehouse Toolkit" case studies (article excerpts):** <https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/>
  *Why: real dimensional-modeling case studies (retail, subscriptions, more) to practice grain and fact/dimension decisions against domains other than Crunch Cycles' bike sales.*
- **Mode Analytics — SQL Tutorial (Analytics-focused sections):** <https://mode.com/sql-tutorial/>
  *Why: extra practice writing analytical SQL (window functions, cohort-style queries) beyond what this week's exercises require.*

## Glossary

| Term | Definition |
|------|------------|
| **OLTP** | Online Transaction Processing — a system optimized for many small, fast, concurrent writes and point lookups (e.g., `crunchcycles`). |
| **OLAP** | Online Analytical Processing — a system optimized for large, complex aggregate queries over historical data. |
| **Data warehouse** | A subject-oriented, integrated, time-variant, non-volatile store built for analytics, separate from operational systems. |
| **Fact table** | A table whose rows are measurable events, holding numeric measures and foreign keys to dimensions. |
| **Dimension table** | A table holding descriptive context (who, what, when, where) that a fact is analyzed by. |
| **Grain** | The precise definition of what one row of a fact table represents. |
| **Star schema** | A dimensional model with one fact table joined directly to denormalized dimension tables. |
| **Snowflake schema** | A dimensional model where dimensions are further normalized into sub-tables. |
| **Surrogate key** | A warehouse-generated key (e.g., `SERIAL`), independent of any source system's ID, used as a dimension's primary key. |
| **Natural key** | An identifier from the source system (e.g., `customer_id`), preserved in a dimension table but not used as its primary key. |
| **Degenerate dimension** | An identifier with business meaning (e.g., an order number) stored directly on the fact table, with no dedicated dimension table. |
| **Conformed dimension** | A dimension (like `dim_date`) shared consistently across multiple fact tables, so its meaning never differs by report. |
| **Slowly changing dimension (SCD)** | The pattern for handling a dimension attribute that changes over time — Type 1 overwrites; Type 2 versions with new rows. |
| **ELT** | Extract-Load-Transform — raw data lands in the destination first; transformation happens afterward, inside the destination, usually with SQL. |
| **Raw / staging layer** | An untouched (or near-untouched) landing copy of source data inside the warehouse, used as the input to transforms. |
| **KPI (Key Performance Indicator)** | A metric deliberately chosen because it's tied to a business goal and would change a decision if it moved. |
| **Vanity metric** | A number that goes up almost regardless of business health, feels good, and drives no decisions — often a bare cumulative total. |
| **Cumulative total** | A running sum over time; mathematically incapable of decreasing if the underlying values are non-negative, which can hide a slowdown. |
| **AOV (Average Order Value)** | Total revenue divided by the number of distinct orders (not line items) in a period. |
| **Gross margin %** | `(Revenue − Cost) / Revenue`, expressed as a percentage — profitability, not just sales volume. |

---

*Broken link? Open an issue or PR.*
