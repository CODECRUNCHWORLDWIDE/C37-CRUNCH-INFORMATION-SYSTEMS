# Week 4 — Querying Data with SQL & Python

> **Goal:** by Sunday you can take any business question — "which region grew fastest last quarter," "which reps have zero sales," "what did we sell in March" — and answer it two ways: a single well-formed SQL query, or a short, re-runnable Python script that pulls the data into pandas, reshapes it, and writes a clean report. No copy-pasting numbers into a spreadsheet, ever.

Welcome back to **C37 · Crunch Information Systems**. In Week 3 you designed and built a normalized schema for **Crunch Cycles**, a small bike company with regions, employees, customers, products, orders, and order line items. A schema by itself doesn't answer anything — it just stores facts correctly. This week you learn to *ask it questions*: joins that pull related facts together, `GROUP BY`/`HAVING` that summarize them, CTEs that structure multi-step logic, and pandas that picks up where SQL's row-and-column model stops being the most natural tool. By Friday you'll turn a one-off analysis into a script anyone on the team can re-run.

If you don't already have the Week 3 schema loaded, the exact `CREATE TABLE` + seed data are reproduced below — copy-paste once and every lecture, exercise, challenge, and the mini-project this week runs against it.

## Learning objectives

By the end of this week, you will be able to:

- **Join** tables with `INNER`, `LEFT`, `RIGHT`, `FULL OUTER`, `CROSS`, and self-joins — and know which one a given business question calls for.
- **Aggregate** with `COUNT`, `SUM`, `AVG`, `MIN`, `MAX`, `GROUP BY`, and filter groups with `HAVING` (as opposed to filtering rows with `WHERE`).
- **Structure** multi-step logic with Common Table Expressions (`WITH`), including chained and recursive-adjacent patterns.
- **Move data between SQL and Python** with `pandas.read_sql` and `DataFrame.to_sql`, via SQLAlchemy.
- **Clean and reshape** data in pandas — dtypes, missing values, `merge`, `pivot_table`, `melt`, `groupby` — without ever treating a spreadsheet as the system of record.
- **Parameterize** a query or report by variables (like a date range) instead of hand-editing SQL every time you run it.
- **Build a repeatable script** — extract, transform, load, in that order — that regenerates a report from source data on demand.
- **Choose deliberately** between "do this in SQL" and "do this in pandas," and explain the trade-off out loud.

## Prerequisites

- Week 1–3 of this course, or equivalent comfort with: what an information system is, basic requirements/use-case thinking, and a normalized relational schema.
- Comfort with `SELECT`, `WHERE`, `ORDER BY` — [C33 Crunch SQL Week 1](../../../C33-CRUNCH-SQL/curriculum/week-01-relational-model-and-select/) covers this if you need a refresher; it is not required, but the muscle memory helps.
- Python 3.10+ installed, with `pip` working. We install `pandas`, `SQLAlchemy`, and `psycopg2-binary` below.
- PostgreSQL 16+ running locally (SQLite fallback noted where behavior differs). Install steps are in [`resources.md`](./resources.md).

## Setup: load the Week 3 schema

**PostgreSQL:**

```bash
createdb crunchcycles
psql crunchcycles
```

**SQLite (fallback):**

```bash
sqlite3 crunchcycles.db
```

Paste this into the shell (works unchanged on both engines — SQLite ignores nothing here):

```sql
CREATE TABLE regions (
    region_id     INTEGER PRIMARY KEY,
    region_name   TEXT NOT NULL
);

CREATE TABLE employees (
    emp_id        INTEGER PRIMARY KEY,
    first_name    TEXT NOT NULL,
    last_name     TEXT NOT NULL,
    email         TEXT NOT NULL,
    title         TEXT NOT NULL,
    region_id     INTEGER,              -- NULL for HQ roles with no single region
    hire_date     DATE NOT NULL,
    manager_id    INTEGER,              -- NULL for the top of the org
    FOREIGN KEY (region_id) REFERENCES regions(region_id)
);

CREATE TABLE customers (
    customer_id   INTEGER PRIMARY KEY,
    company_name  TEXT NOT NULL,
    contact_name  TEXT NOT NULL,
    email         TEXT NOT NULL,
    city          TEXT NOT NULL,
    country       TEXT NOT NULL,
    region_id     INTEGER NOT NULL,
    signup_date   DATE NOT NULL,
    FOREIGN KEY (region_id) REFERENCES regions(region_id)
);

CREATE TABLE products (
    product_id    INTEGER PRIMARY KEY,
    product_name  TEXT NOT NULL,
    category      TEXT NOT NULL,
    unit_price    NUMERIC NOT NULL,
    unit_cost     NUMERIC NOT NULL,
    discontinued  BOOLEAN NOT NULL DEFAULT FALSE
);

CREATE TABLE orders (
    order_id      INTEGER PRIMARY KEY,
    customer_id   INTEGER NOT NULL,
    employee_id   INTEGER NOT NULL,
    order_date    DATE NOT NULL,
    ship_date     DATE,                 -- NULL = not shipped yet
    status        TEXT NOT NULL,        -- 'Completed' | 'Pending' | 'Cancelled'
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id),
    FOREIGN KEY (employee_id) REFERENCES employees(emp_id)
);

CREATE TABLE order_items (
    order_id      INTEGER NOT NULL,
    product_id    INTEGER NOT NULL,
    quantity      INTEGER NOT NULL,
    unit_price    NUMERIC NOT NULL,     -- price actually charged (may differ from products.unit_price)
    PRIMARY KEY (order_id, product_id),
    FOREIGN KEY (order_id) REFERENCES orders(order_id),
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);

INSERT INTO regions VALUES
(1,'North America'),(2,'Europe'),(3,'APAC'),(4,'LATAM');

INSERT INTO employees VALUES
(1,'Maria','Chen','maria.chen@crunchcycles.io','Sales Director',NULL,'2018-03-01',NULL),
(2,'Diego','Alvarez','diego.alvarez@crunchcycles.io','Sales Rep',1,'2019-06-10',1),
(3,'Sarah','Kim','sarah.kim@crunchcycles.io','Sales Rep',1,'2020-01-15',1),
(4,'Tom','Fischer','tom.fischer@crunchcycles.io','Sales Rep',2,'2019-09-23',1),
(5,'Elena','Rossi','elena.rossi@crunchcycles.io','Sales Rep',2,'2021-04-05',1),
(6,'Kenji','Watanabe','kenji.watanabe@crunchcycles.io','Sales Rep',3,'2020-11-30',1),
(7,'Priya','Nair','priya.nair@crunchcycles.io','Sales Rep',3,'2022-02-14',1),
(8,'Lucas','Silva','lucas.silva@crunchcycles.io','Sales Rep',4,'2022-08-01',1);

INSERT INTO customers VALUES
(1,'Trailhead Bikes','Jen Ortiz','jen@trailheadbikes.com','Denver','USA',1,'2021-01-10'),
(2,'Northern Gear Co','Mark Bell','mark@northerngear.ca','Toronto','Canada',1,'2020-05-22'),
(3,'Pacific Wheels','Dana Wu','dana@pacificwheels.com','Seattle','USA',1,'2022-03-14'),
(4,'Rocky Mountain Riders','Paul Green','paul@rmriders.ca','Calgary','Canada',1,'2019-11-02'),
(5,'Alpine Sport Supply','Hans Weber','hans@alpinesport.de','Munich','Germany',2,'2018-07-19'),
(6,'Le Velo Parisien','Claire Dubois','claire@levelo.fr','Paris','France',2,'2020-09-08'),
(7,'Nordic Cycle Club','Erik Solberg','erik@nordiccycle.no','Oslo','Norway',2,'2021-06-25'),
(8,'Iberia Bike Traders','Marta Ruiz','marta@iberiabike.es','Madrid','Spain',2,'2022-01-30'),
(9,'Tokyo Pedal Works','Ken Sato','ken@tokyopedal.jp','Tokyo','Japan',3,'2019-04-11'),
(10,'Sydney Spokes','Amy Clarke','amy@sydneyspokes.au','Sydney','Australia',3,'2020-12-01'),
(11,'Seoul Cycle Hub','Ji-ho Park','jiho@seoulcycle.kr','Seoul','South Korea',3,'2021-08-17'),
(12,'Mumbai Mountain Bikes','Ravi Shah','ravi@mumbaimtb.in','Mumbai','India',3,'2023-02-09'),
(13,'Sao Paulo Ciclismo','Bruna Alves','bruna@spciclismo.br','Sao Paulo','Brazil',4,'2020-03-03'),
(14,'Andes Bike Co','Diego Fuentes','diego@andesbike.cl','Santiago','Chile',4,'2021-10-14'),
(15,'Bogota Bike Collective','Laura Diaz','laura@bogotabike.co','Bogota','Colombia',4,'2022-05-27'),
(16,'Lima Rueda Libre','Carlos Vega','carlos@limarueda.pe','Lima','Peru',4,'2023-07-06'),
(17,'Desert Trail Cycles','Nina Cole','nina@deserttrail.com','Phoenix','USA',1,'2023-09-19'),
(18,'Fjord Cycling Supply','Oda Berg','oda@fjordcycling.no','Bergen','Norway',2,'2024-01-05');

INSERT INTO products VALUES
(1,'Crunch Trail 100','Mountain Bike',899.00,540.00,FALSE),
(2,'Crunch Trail 250','Mountain Bike',1499.00,890.00,FALSE),
(3,'Crunch Road Pro','Road Bike',1899.00,1120.00,FALSE),
(4,'Crunch Road Elite','Road Bike',2799.00,1680.00,FALSE),
(5,'Crunch Commuter','Hybrid Bike',649.00,390.00,FALSE),
(6,'Crunch City Fold','Folding Bike',799.00,470.00,FALSE),
(7,'Crunch Kids 20','Kids Bike',299.00,165.00,FALSE),
(8,'Crunch Helmet Pro','Accessory',89.00,38.00,FALSE),
(9,'Crunch Helmet Lite','Accessory',49.00,19.00,FALSE),
(10,'Crunch Pannier Set','Accessory',119.00,58.00,FALSE),
(11,'Crunch Repair Kit','Accessory',35.00,14.00,FALSE),
(12,'Crunch Trail 50','Mountain Bike',549.00,330.00,TRUE);

INSERT INTO orders VALUES
(1,1,2,'2024-01-05','2024-01-08','Completed'),
(2,2,2,'2024-01-09','2024-01-12','Completed'),
(3,3,3,'2024-01-11','2024-01-15','Completed'),
(4,4,2,'2024-01-15','2024-01-18','Completed'),
(5,5,4,'2024-01-18','2024-01-22','Completed'),
(6,6,5,'2024-01-20','2024-01-25','Completed'),
(7,9,6,'2024-01-22','2024-01-26','Completed'),
(8,10,6,'2024-01-25',NULL,'Pending'),
(9,13,8,'2024-01-28','2024-02-02','Completed'),
(10,14,8,'2024-01-30','2024-02-03','Completed'),
(11,1,2,'2024-02-03','2024-02-06','Completed'),
(12,7,5,'2024-02-05','2024-02-09','Completed'),
(13,8,4,'2024-02-08',NULL,'Cancelled'),
(14,11,6,'2024-02-10','2024-02-14','Completed'),
(15,15,8,'2024-02-12','2024-02-16','Completed'),
(16,3,3,'2024-02-15','2024-02-19','Completed'),
(17,5,4,'2024-02-18','2024-02-22','Completed'),
(18,2,2,'2024-02-20','2024-02-24','Completed'),
(19,12,6,'2024-02-22',NULL,'Pending'),
(20,16,8,'2024-02-25','2024-03-01','Completed'),
(21,4,2,'2024-03-02','2024-03-06','Completed'),
(22,6,5,'2024-03-05','2024-03-09','Completed'),
(23,9,6,'2024-03-08','2024-03-12','Completed'),
(24,17,2,'2024-03-10','2024-03-14','Completed'),
(25,13,8,'2024-03-13',NULL,'Cancelled'),
(26,1,3,'2024-03-16','2024-03-20','Completed'),
(27,7,4,'2024-03-19','2024-03-23','Completed'),
(28,10,6,'2024-03-22','2024-03-26','Completed'),
(29,3,2,'2024-04-01','2024-04-05','Completed'),
(30,5,4,'2024-04-03','2024-04-07','Completed'),
(31,2,3,'2024-04-06','2024-04-10','Completed'),
(32,8,5,'2024-04-09','2024-04-13','Completed'),
(33,11,6,'2024-04-12',NULL,'Pending'),
(34,14,8,'2024-04-15','2024-04-19','Completed'),
(35,4,2,'2024-05-02','2024-05-06','Completed'),
(36,6,4,'2024-05-05','2024-05-09','Completed'),
(37,9,6,'2024-05-08','2024-05-12','Completed'),
(38,15,8,'2024-05-15','2024-05-19','Completed'),
(39,1,2,'2024-06-01','2024-06-05','Completed'),
(40,3,3,'2024-06-10','2024-06-14','Completed');

INSERT INTO order_items VALUES
(1,1,1,899.00),(1,8,1,89.00),
(2,3,1,1899.00),
(3,5,2,649.00),
(4,1,1,899.00),(4,11,2,35.00),
(5,6,1,799.00),
(6,2,1,1499.00),(6,9,2,49.00),
(7,4,1,2799.00),
(8,7,3,299.00),
(9,3,1,1899.00),(9,10,1,119.00),
(10,1,2,899.00),
(11,5,1,649.00),(11,8,1,89.00),
(12,6,1,799.00),
(13,2,1,1499.00),
(14,3,1,1899.00),(14,11,1,35.00),
(15,7,4,299.00),
(16,1,1,899.00),
(17,4,1,2799.00),(17,9,1,49.00),
(18,5,3,649.00),
(19,6,1,799.00),
(20,2,1,1499.00),(20,10,2,119.00),
(21,1,2,850.00),
(22,3,1,1899.00),
(23,4,1,2799.00),
(24,7,2,299.00),(24,8,2,89.00),
(25,5,1,649.00),
(26,1,1,899.00),(26,11,3,35.00),
(27,6,1,799.00),
(28,2,1,1499.00),
(29,3,1,1899.00),(29,9,1,49.00),
(30,5,2,649.00),
(31,1,1,899.00),
(32,4,1,2650.00),
(33,6,1,799.00),
(34,3,1,1899.00),(34,10,1,119.00),
(35,1,1,899.00),
(36,7,2,299.00),
(37,2,1,1499.00),(37,8,1,89.00),
(38,5,1,649.00),
(39,1,1,899.00),(39,11,1,35.00),
(40,3,1,1899.00);
```

Sanity check — this should print `40` and `54`:

```sql
SELECT COUNT(*) FROM orders;
SELECT COUNT(*) FROM order_items;
```

Notice what's deliberately in there: order 13 and 25 are **Cancelled** (no `ship_date`); orders 8, 19, and 33 are **Pending** (no `ship_date` yet, not cancelled); employees 1 (Maria, the director) and 7 (Priya) have **zero orders**; customer 18 (Fjord) has **never ordered**. Every one of those gaps is there on purpose — joins and aggregates behave differently around missing rows, and that's exactly what this week teaches you to reason about.

## Install the Python side

```bash
python3 -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install pandas sqlalchemy psycopg2-binary python-dotenv
```

We use **SQLAlchemy** as the bridge between Postgres and pandas — `pandas.read_sql` and `DataFrame.to_sql` both take a SQLAlchemy engine. Details and exact connection strings are in [`resources.md`](./resources.md).

## Weekly schedule

Adds up to the course's full-time target of **~28 hours**.

| Day | Focus | Lectures | Exercises | Challenges | Quiz/Read | Homework | Mini-Project | Daily Total |
|-----------|--------------------------------------------|---------:|----------:|-----------:|----------:|---------:|-------------:|------------:|
| Monday | Joins as answers to business questions | 2h | 1.5h | 0h | 0.5h | 1h | 0h | 5h |
| Tuesday | Aggregation, `GROUP BY`/`HAVING`, CTEs | 2h | 1.5h | 0h | 0.5h | 1h | 0h | 5h |
| Wednesday | SQL meets pandas: `read_sql`/`to_sql` | 2h | 1.5h | 1h | 0.5h | 1h | 0h | 6h |
| Thursday | Cleaning, reshaping, merging in pandas | 0h | 1.5h | 1h | 0.5h | 1h | 1h | 5h |
| Friday | Repeatable pipelines; parameterization | 2h | 1h | 0h | 0.5h | 1h | 1.5h | 6h |
| Saturday | Mini-project (SQL-to-pandas pipeline) | 0h | 0h | 0h | 0h | 0h | 2.5h | 2.5h |
| Sunday | Quiz + review | 0h | 0h | 0h | 1h | 0h | 0h | 1h |
| **Total** | | **6h** | **7h** | **2h** | **3.5h** | **5h** | **5h** | **~28h** |

## How to navigate this week

Work top to bottom. Each piece assumes the ones above it.

| # | File | What's inside | ~Time |
|--:|------|---------------|------:|
| 1 | [lecture-notes/01-sql-for-business-questions.md](./lecture-notes/01-sql-for-business-questions.md) | Every join type, `GROUP BY`/`HAVING`, CTEs — framed as questions a manager would ask | 2h |
| 2 | [lecture-notes/02-sql-meets-python.md](./lecture-notes/02-sql-meets-python.md) | `read_sql`/`to_sql`, DataFrame cleaning, `merge`, `pivot_table`, `melt` | 2h |
| 3 | [lecture-notes/03-repeatable-pipelines.md](./lecture-notes/03-repeatable-pipelines.md) | Turning a script into a parameterized, re-runnable, idempotent pipeline | 2h |
| 4 | [exercises/exercise-01-answer-with-joins.md](./exercises/exercise-01-answer-with-joins.md) | 10 business questions, answered with joins | 1.5h |
| 5 | [exercises/exercise-02-sql-to-pandas.md](./exercises/exercise-02-sql-to-pandas.md) | Load a query into pandas and reshape it | 1.5h |
| 6 | [exercises/exercise-03-parameterize-a-report.md](./exercises/exercise-03-parameterize-a-report.md) | Parameterize a report by date range | 1.5h |
| 7 | [challenges/challenge-01-sql-vs-pandas.md](./challenges/challenge-01-sql-vs-pandas.md) | Solve one problem two ways and compare | 1h |
| 8 | [challenges/challenge-02-build-a-daily-refresh.md](./challenges/challenge-02-build-a-daily-refresh.md) | Build a script that refreshes a report daily | 1h |
| 9 | [mini-project/README.md](./mini-project/README.md) | Build a repeatable SQL-to-pandas reporting pipeline | 2.5h |
| 10 | [homework.md](./homework.md) | Extra practice, spaced out | 5h |
| 11 | [quiz.md](./quiz.md) | 14 self-check questions + answer key | 1h |
| 12 | [resources.md](./resources.md) | Official docs + install steps | — |

## By the end of this week you can…

- Pick the right join for a question in one read, without trial and error.
- Tell `WHERE` and `HAVING` apart on sight, and know why order matters.
- Move a result set from Postgres into a DataFrame and back without losing types.
- Reshape messy query output into a clean report with `merge`/`pivot_table`/`melt`.
- Write a script that takes a date range as an argument and regenerates the same report every time — no manual re-typing, no spreadsheet in the loop.

## Up next

[Week 5 — Process modeling & automation](../week-05-process-modeling-and-automation/) — now that you can pull any answer out of the data, we automate the workflow that produces it.

---

*Part of the Code Crunch Worldwide open curriculum · GPL-3.0 · If you find errors, please open an issue or PR.*
