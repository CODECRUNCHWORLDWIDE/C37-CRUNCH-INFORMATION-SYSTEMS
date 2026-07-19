# Week 3 — Data Modeling & Databases

> **Goal:** by Sunday you can take a real domain — described in messy, ambiguous English — and turn it into a normalized PostgreSQL schema with real primary keys, foreign keys, and constraints: an information system's foundation, built once and trusted forever.

Welcome back to **C37 · Crunch Information Systems**. Week 1 taught you to see an organization as people, process, data, and technology. Week 2 taught you to turn stakeholder needs into a requirements spec and use cases. This week you build the thing every one of those requirements eventually has to stand on: **the data model**. Get the model wrong and every feature you build on top of it — every report, every API, every dashboard — inherits the wrong shape. Get it right and the rest of the course (querying in Week 4, enterprise data flows in Week 6, integration in Week 7, analytics in Week 10) has solid ground to build on.

We use one running case study all week: **CrunchRide**, a fictional bike-share company operating docking stations across a city. Riders sign up for membership plans, borrow bikes from stations, ride, and return them — sometimes to a different station, sometimes needing a repair first. It's a small domain, but it has every modeling problem you'll meet in a real job: one-to-many relationships (a station has many bikes), many-to-many relationships (a rider can take many rides, a bike is ridden by many riders), optional attributes, business rules that need to be enforced by the database — not just by good intentions — and a redundant flat table crying out to be normalized.

By the reader-tracked lessons below you'll draw the ER diagram, learn the theory of why normal forms exist, and translate the whole thing into `CREATE TABLE` statements PostgreSQL will actually accept — with the constraints that keep bad data out **before** it ever lands in a table.

## Learning objectives

By the end of this week, you will be able to:

- **Model** a domain as entities, attributes, and relationships, and draw it using crow's-foot ER notation — including one-to-one, one-to-many, and many-to-many cardinality.
- **Choose** primary keys (natural vs. surrogate) and foreign keys, and explain what each constraint (`NOT NULL`, `UNIQUE`, `CHECK`, `FOREIGN KEY`, `ON DELETE`) actually protects against.
- **Normalize** a flat, redundant table step by step from 1NF through BCNF, naming the specific anomaly (insertion, update, or deletion) each step removes.
- **Recognize** when denormalizing on purpose is the right engineering trade-off — and how to document that decision so it doesn't look like an accident six months later.
- **Translate** an ER model into working PostgreSQL DDL — correct types, constraints, indexes on foreign keys — with a SQLite fallback for anything platform-specific.
- **Explain**, in one paragraph you could say to a non-technical stakeholder, why a shared, constrained SQL database beats a spreadsheet as the system of record for anything more than a handful of people or rows.

## Prerequisites

- **Week 1 and Week 2** of this course, or equivalent comfort with "what is an information system" and "what is a requirement."
- Comfortable reading and writing basic `SELECT` statements. [C33 Crunch SQL](../../../C33-CRUNCH-SQL/), Weeks 1–2, is the ideal companion if SQL itself is new — this week assumes you can read a `CREATE TABLE` statement, not that you're a query expert.
- PostgreSQL 16+ installed (primary engine). SQLite 3.35+ as a fallback if you can't install Postgres. Install steps are in [`resources.md`](./resources.md).
- A way to draw diagrams — pen and paper is genuinely fine for the exercises; [dbdiagram.io](https://dbdiagram.io) or draw.io work well if you want something shareable.

## Set up a scratch database (do this first)

You don't need a seed dataset loaded yet — this week you're building schemas from scratch. Just confirm your engine works:

**PostgreSQL:**

```bash
createdb crunchride
psql crunchride
```

```sql
SELECT version();     -- confirm you're connected
```

**SQLite:**

```bash
sqlite3 crunchride.db
```

```sql
SELECT sqlite_version();
```

Keep this database open — every lecture and exercise this week has you run real `CREATE TABLE` statements against it. Feel free to `DROP TABLE` and start over as often as you like; that's the whole point of a scratch database.

## Weekly schedule

The schedule below adds up to approximately **28 hours** (the course's full-time pace). Treat it as a target, not a stopwatch.

| Day | Focus | Lectures | Exercises | Challenges | Quiz/Read | Homework | Mini-Project | Daily Total |
|-----------|------------------------------------------------|---------:|----------:|-----------:|----------:|---------:|-------------:|------------:|
| Monday | Entities, attributes, relationships, cardinality | 2h | 1h | 0h | 0.5h | 1h | 0h | 4.5h |
| Tuesday | Draw the CrunchRide ER diagram | 0h | 1.5h | 0h | 0.5h | 1h | 0h | 3h |
| Wednesday | Normalization theory, 1NF–BCNF, keys | 2h | 1.5h | 0h | 0.5h | 1h | 0h | 5h |
| Thursday | Normalize a flat table; from model to DDL | 2h | 1h | 1h | 0.5h | 1h | 1h | 6.5h |
| Friday | Write real DDL with constraints; challenges | 0h | 1h | 1.5h | 0.5h | 1h | 1.5h | 5.5h |
| Saturday | Mini-project (schema, DDL, seed data) | 0h | 0h | 0h | 0h | 0h | 2.5h | 2.5h |
| Sunday | Quiz + review | 0h | 0h | 0h | 1h | 0h | 0h | 1h |
| **Total** | | **6h** | **6h** | **2.5h** | **3.5h** | **5h** | **5h** | **28h** |

## How to navigate this week

Work top to bottom. Each piece assumes the ones above it.

| # | File | What's inside | ~Time |
|--:|------|---------------|------:|
| 1 | [lecture-notes/01-entity-relationship-modeling.md](./lecture-notes/01-entity-relationship-modeling.md) | Entities, attributes, relationships, crow's-foot cardinality, building the CrunchRide ER diagram | 2h |
| 2 | [lecture-notes/02-normalization-and-keys.md](./lecture-notes/02-normalization-and-keys.md) | Primary/foreign keys, functional dependencies, 1NF → BCNF, when to denormalize | 2h |
| 3 | [lecture-notes/03-from-model-to-schema.md](./lecture-notes/03-from-model-to-schema.md) | Translating the ER model into PostgreSQL DDL — types, constraints, indexes, why not a spreadsheet | 2h |
| 4 | [exercises/exercise-01-draw-an-er-diagram.md](./exercises/exercise-01-draw-an-er-diagram.md) | Draw an ER diagram for a public library | 1h |
| 5 | [exercises/exercise-02-normalize-a-flat-table.md](./exercises/exercise-02-normalize-a-flat-table.md) | Normalize a messy flat orders table to 3NF | 1.5h |
| 6 | [exercises/exercise-03-write-ddl.md](./exercises/exercise-03-write-ddl.md) | Write `CREATE TABLE` DDL with real constraints | 1.5h |
| 7 | [challenges/challenge-01-model-a-real-domain.md](./challenges/challenge-01-model-a-real-domain.md) | Model a veterinary clinic from a raw interview transcript | 1.5h |
| 8 | [challenges/challenge-02-denormalize-with-intent.md](./challenges/challenge-02-denormalize-with-intent.md) | Justify one deliberate denormalization and prove it's necessary | 1h |
| 9 | [mini-project/README.md](./mini-project/README.md) | Design and build the full CrunchRide PostgreSQL schema: ER diagram, DDL, seed data | 2.5h |
| 10 | [homework.md](./homework.md) | Extra modeling and normalization practice, spaced out | 5h |
| 11 | [quiz.md](./quiz.md) | 15 self-check questions + answer key | 1h |
| 12 | [resources.md](./resources.md) | Official docs, diagramming tools, and the few links worth your time | — |

## By the end of this week you can…

- Sit down with a stakeholder, listen to them describe their business in plain English, and sketch a correct ER diagram before the meeting ends.
- Look at a flat table and name exactly which anomaly (insertion, update, deletion) each redundant column will eventually cause.
- Write `CREATE TABLE` DDL with primary keys, foreign keys, `CHECK` constraints, and sensible types — not as boilerplate you copy, but as decisions you can defend.
- Tell a stakeholder, in one paragraph, why their "just put it in a spreadsheet" instinct will not survive contact with more than one simultaneous user.

## Up next

Week 4 — Querying Data with SQL + Python: now that the schema exists and holds real, constrained data, we pull answers out of it — joins, aggregation, and a repeatable pandas pipeline on top.

---

*Part of the Code Crunch Worldwide open curriculum · GPL-3.0 · If you find errors, please open an issue or PR.*
