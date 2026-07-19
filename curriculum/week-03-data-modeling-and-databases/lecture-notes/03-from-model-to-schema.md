# Lecture 3 — From Model to SQL Schema

> **Duration:** ~2 hours. **Outcome:** You can translate a normalized ER model directly into working PostgreSQL DDL — choosing correct types, writing every constraint from Lecture 2, indexing foreign keys, and handling `ON DELETE` behavior deliberately — and you can explain, precisely, why this belongs in a database and not a spreadsheet.

The ER diagram from Lecture 1 and the normalization rules from Lecture 2 exist to produce one artifact: a set of `CREATE TABLE` statements that PostgreSQL will run without complaint and that will *refuse* to store bad data. This lecture writes that DDL for the full CrunchRide model, decision by decision, so every line you write in the mini-project is a choice you understand — not boilerplate you copied.

## 1. Choosing types

PostgreSQL has more type precision available than most beginners use. Defaulting to `TEXT` for everything and `NUMERIC` for nothing is how money bugs and timezone bugs get into production.

| Kind of data | Use | Not | Why |
|---|---|---|---|
| Surrogate primary key | `SERIAL` (or `GENERATED ALWAYS AS IDENTITY` in modern Postgres) | manually-managed integers | auto-increments, guarantees uniqueness for you |
| Short identifiers/codes | `TEXT` or `VARCHAR(n)` | — | Postgres `TEXT` has no performance penalty over `VARCHAR(n)`; use a length cap only if the business rule genuinely caps it |
| Money / prices | `NUMERIC(10,2)` | `FLOAT`/`REAL` | floating point cannot represent money exactly — `0.1 + 0.2` is a real bug waiting to happen in a billing table |
| Whole counts | `INTEGER` | `TEXT` | lets the database enforce numeric `CHECK`s and sort correctly |
| Yes/no facts | `BOOLEAN` | `INTEGER` (0/1) or `TEXT` ('Y'/'N') | self-documenting, and Postgres understands `TRUE`/`FALSE` natively |
| Calendar date only | `DATE` | `TIMESTAMP` | a birth date or "day" has no meaningful time-of-day component |
| A specific instant | `TIMESTAMPTZ` | `TIMESTAMP` (no zone) | **always** store instants with a time zone; a ride's `start_time` recorded without one is ambiguous the moment your users or servers span more than one time zone |
| A fixed small set of labels | `TEXT` + `CHECK (col IN (...))`, or a lookup table | free-text `TEXT` with no constraint | typos like `'Actve'` silently corrupt filters and reports |

**A note on `ENUM` types:** PostgreSQL supports a native `CREATE TYPE ... AS ENUM (...)`. It's tidy, but altering an enum later (adding a value) is more awkward than adding a row to a lookup table or loosening a `CHECK`. For CrunchRide we use a `CHECK` constraint for `bike.status` — simple, visible in the table definition itself, and painless to change later with `ALTER TABLE ... DROP CONSTRAINT` / `ADD CONSTRAINT`.

## 2. The full CrunchRide schema

Building bottom-up — tables with no foreign keys first, so every `REFERENCES` target already exists when we need it.

```sql
-- Independent lookup tables first: nothing else exists yet for them to reference.

CREATE TABLE membership_plans (
    plan_id             SERIAL PRIMARY KEY,
    name                TEXT NOT NULL UNIQUE,
    monthly_price       NUMERIC(6,2) NOT NULL CHECK (monthly_price >= 0),
    ride_minutes_incl   INTEGER NOT NULL CHECK (ride_minutes_incl >= 0)
);

CREATE TABLE stations (
    station_id   SERIAL PRIMARY KEY,
    name         TEXT NOT NULL,
    address      TEXT NOT NULL,
    capacity     INTEGER NOT NULL CHECK (capacity > 0)
);

CREATE TABLE mechanics (
    mechanic_id  SERIAL PRIMARY KEY,
    first_name   TEXT NOT NULL,
    last_name    TEXT NOT NULL,
    hired_date   DATE NOT NULL DEFAULT CURRENT_DATE
);

-- Riders reference membership_plans, so it comes after that table.

CREATE TABLE riders (
    rider_id     SERIAL PRIMARY KEY,
    first_name   TEXT NOT NULL,
    last_name    TEXT NOT NULL,
    email        TEXT NOT NULL UNIQUE,
    plan_id      INTEGER NOT NULL REFERENCES membership_plans(plan_id),
    signup_date  DATE NOT NULL DEFAULT CURRENT_DATE
);

-- Bikes reference stations (where currently docked).

CREATE TABLE bikes (
    bike_id      SERIAL PRIMARY KEY,
    model        TEXT NOT NULL,
    status       TEXT NOT NULL DEFAULT 'in_service'
                 CHECK (status IN ('in_service', 'in_maintenance', 'retired')),
    station_id   INTEGER REFERENCES stations(station_id)
                 -- NULL allowed: a bike mid-ride is not docked anywhere
);

-- Rides reference riders, bikes, and stations TWICE (start and end).

CREATE TABLE rides (
    ride_id          SERIAL PRIMARY KEY,
    rider_id         INTEGER NOT NULL REFERENCES riders(rider_id),
    bike_id          INTEGER NOT NULL REFERENCES bikes(bike_id),
    start_station_id INTEGER NOT NULL REFERENCES stations(station_id),
    end_station_id   INTEGER REFERENCES stations(station_id),   -- NULL until the ride ends
    start_time       TIMESTAMPTZ NOT NULL DEFAULT now(),
    end_time         TIMESTAMPTZ,
    CHECK (end_time IS NULL OR end_time > start_time)
);

-- Maintenance records reference bikes and mechanics.

CREATE TABLE maintenance_records (
    record_id     SERIAL PRIMARY KEY,
    bike_id       INTEGER NOT NULL REFERENCES bikes(bike_id),
    mechanic_id   INTEGER NOT NULL REFERENCES mechanics(mechanic_id),
    service_date  DATE NOT NULL DEFAULT CURRENT_DATE,
    description   TEXT NOT NULL,
    cost          NUMERIC(8,2) NOT NULL CHECK (cost >= 0)
);
```

Read that top to bottom and notice the ordering isn't arbitrary: **PostgreSQL will refuse to create a table with a `REFERENCES` clause pointing at a table that doesn't exist yet.** Order your `CREATE TABLE` statements so every foreign key target is already defined — a direct, practical consequence of the relationships you mapped in Lecture 1.

## 3. Two foreign keys to the same table — `rides.start_station_id` / `end_station_id`

This is the DDL version of the "two relationships to Station" callout from Lecture 1. Both columns reference `stations(station_id)`, and that's completely legal — a foreign key constraint doesn't care that another column in the same table references the same target; each `REFERENCES` is independent. What it *does* require is that you name them clearly (`start_station_id`, not `station_id_1`) — the column name is the only thing telling a future reader which relationship is which.

## 4. `ON DELETE` behavior — deciding what happens when the referenced row disappears

Every foreign key needs a deliberate answer to: *"what happens to this row if the row it points to gets deleted?"* Leaving it unspecified silently defaults to `ON DELETE NO ACTION`, which **blocks the delete entirely** if any row still references it. That's often correct, but not always — decide, don't default by accident.

```sql
-- If a mechanic leaves the company, keep their maintenance history
-- but don't allow deleting a mechanic who still has records — force
-- a deliberate reassignment first.
mechanic_id INTEGER NOT NULL REFERENCES mechanics(mechanic_id)
    -- ON DELETE RESTRICT is the (implicit) default and correct here

-- If a station is decommissioned, a bike currently docked there should
-- become "unassigned" rather than block the station's deletion.
station_id INTEGER REFERENCES stations(station_id) ON DELETE SET NULL

-- If a membership plan is discontinued and genuinely has zero riders left
-- on it, cascading isn't needed — but if you wanted "delete the plan,
-- and everyone on it too" you'd write ON DELETE CASCADE. We do NOT want
-- that here: deleting a plan should never silently delete riders.
plan_id INTEGER NOT NULL REFERENCES membership_plans(plan_id)
    -- left as RESTRICT (default): force reassigning riders before deleting a plan
```

| Option | Behavior | When it's right |
|---|---|---|
| `RESTRICT` / `NO ACTION` (default) | Blocks the delete if any row references it | The default for anything where an accidental cascade would destroy unrelated history (deleting a rider should never be allowed to silently delete their ride history) |
| `CASCADE` | Deletes the referencing rows too | Genuine parent/child ownership where the child has no meaning without the parent (deleting an `order` should cascade to its `order_lines`) |
| `SET NULL` | Sets the FK column to `NULL` | The relationship is optional and the referencing row still means something without it (a bike's `station_id` when the station is decommissioned) |

**Never pick `CASCADE` reflexively because it "makes the error go away."** A cascading delete that silently wipes out a table you didn't think about is one of the most common ways real production data gets destroyed. Every `ON DELETE` clause should be a sentence you could say out loud and defend.

## 5. Indexes — foreign keys don't get one for free

PostgreSQL automatically indexes primary keys and `UNIQUE` columns, but **not** foreign key columns. Every foreign key you add is a column you will eventually filter or join on (`WHERE rider_id = ...`, `JOIN ... ON bike_id = ...`), so add an index explicitly:

```sql
CREATE INDEX idx_rides_rider_id  ON rides(rider_id);
CREATE INDEX idx_rides_bike_id   ON rides(bike_id);
CREATE INDEX idx_rides_start_station_id ON rides(start_station_id);
CREATE INDEX idx_maintenance_bike_id ON maintenance_records(bike_id);
```

This isn't premature optimization — it's the standard baseline for any foreign key you expect to query by, which in a normalized schema is nearly all of them (the whole point of splitting tables is that you'll be joining them back together constantly).

## 6. Seeding and sanity-checking

Once the tables exist, insert a handful of rows and check the constraints actually bite:

```sql
INSERT INTO membership_plans (name, monthly_price, ride_minutes_incl) VALUES
('Basic', 9.99, 60), ('Plus', 19.99, 180), ('Unlimited', 39.99, 999999);

INSERT INTO stations (name, address, capacity) VALUES
('Downtown Hub', '100 Main St', 20),
('Riverside', '55 River Rd', 12);

-- This should FAIL — proves the CHECK constraint works:
INSERT INTO membership_plans (name, monthly_price, ride_minutes_incl)
VALUES ('Broken', -5.00, 60);
-- ERROR: new row for relation "membership_plans" violates check constraint
```

**Deliberately trying to break your own constraints is not optional — it's how you find out whether they actually work**, versus just looking correct on paper. Do this for every constraint in the mini-project: try to insert a row that should be rejected, and confirm it is.

## 7. SQLite fallback notes

If you're working in SQLite instead of Postgres, most of this DDL is unchanged — a few differences to know:

- SQLite has no `SERIAL`; use `INTEGER PRIMARY KEY` (which auto-increments by itself in SQLite specifically because it *is* the rowid).
- SQLite does not enforce foreign keys **unless you turn it on per connection**: run `PRAGMA foreign_keys = ON;` at the start of every session, or your `REFERENCES` clauses are silently decorative.
- SQLite has no `TIMESTAMPTZ`; store instants as `TEXT` in ISO-8601 UTC (`'2026-07-18T14:30:00Z'`) and be disciplined about always writing UTC.
- SQLite's type system is famously loose ("type affinity," not strict typing) — a `CHECK` constraint still works and is worth leaning on more heavily than usual to compensate.

## 8. Why this is not a spreadsheet

You now have the full, concrete list to answer that question precisely instead of vaguely:

| What the schema gives you | What a spreadsheet cannot do |
|---|---|
| `FOREIGN KEY` | Guarantee a `rides.bike_id` always refers to a real bike — a spreadsheet cell can hold any text, referencing a row that was deleted, renamed, or never existed |
| `CHECK` | Reject a negative `monthly_price` or an `end_time` before `start_time` **at write time**, for every writer, forever |
| `UNIQUE` | Guarantee no two riders ever share an email — a spreadsheet can't stop a copy-paste duplicate |
| Transactions | Insert a `ride` and update `bikes.station_id` together, atomically — either both happen or neither does; a spreadsheet has no concept of "these five cell edits must succeed or fail together" |
| Concurrent access | Ten people can safely insert rides at the same second without corrupting each other's edits or silently overwriting a colleague's row — spreadsheet "who's editing this cell right now" conflicts are a known, painful failure mode at any real scale |
| A queryable, indexed store | Ask "how many rides started at Downtown Hub last month" in milliseconds against millions of rows — a spreadsheet formula scanning that many rows becomes unusable |

That's the paragraph this week's learning objectives asked you to be able to say out loud to a non-technical stakeholder: **a spreadsheet is a great tool for one person to look at a small amount of data by eye. A database is the tool for many people and programs to write, read, and trust a growing amount of data together, with rules the software enforces instead of rules people remember.** For anything CrunchRide-shaped — many riders, many stations, constant concurrent writes, rules that must never be violated — that's not a style preference, it's the only architecture that survives contact with real usage.

## 9. Check yourself

- Why must `membership_plans` be created before `riders` in the DDL script?
- Why does `bikes.station_id` allow `NULL` while `rides.rider_id` does not?
- Walk through what `ON DELETE CASCADE` vs. `ON DELETE RESTRICT` vs. `ON DELETE SET NULL` would each do if you deleted a station that still has bikes docked at it. Which did we choose for CrunchRide, and why?
- Why do foreign key columns need an explicit index in PostgreSQL, when primary keys don't?
- Name two SQLite differences you'd have to account for if you built this schema there instead of Postgres.
- In one paragraph, in your own words (not copied from Section 8), explain to a non-technical founder why their idea to "just track rides in a shared spreadsheet" won't hold up.

That closes the lecture sequence for the week. Move to the exercises to put every piece of this — ER diagrams, normalization, and DDL — into your own hands before the challenges and mini-project ask you to do it on new, unseen domains.

## Further reading

- **PostgreSQL — Data types:** <https://www.postgresql.org/docs/current/datatype.html>
- **PostgreSQL — `CREATE TABLE` reference:** <https://www.postgresql.org/docs/current/sql-createtable.html>
- **PostgreSQL — Foreign Keys chapter:** <https://www.postgresql.org/docs/current/ddl-constraints.html#DDL-CONSTRAINTS-FK>
- **PostgreSQL — Indexes overview:** <https://www.postgresql.org/docs/current/indexes.html>
- **SQLite — Foreign key support (and why you must enable it):** <https://www.sqlite.org/foreignkeys.html>
