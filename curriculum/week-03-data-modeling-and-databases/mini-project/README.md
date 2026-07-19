# Mini-Project — The Full CrunchRide Schema

> Design and build a complete, normalized PostgreSQL schema for CrunchRide: an ER diagram, full DDL with keys and constraints, seed data proving it all works, and a short written note on why this system belongs in SQL — not a spreadsheet. This is the week's capstone: everything the lectures explained and the exercises drilled, applied once, end to end, on the real running case study.

**Estimated time:** 2.5–3 hours, best done Saturday after the exercises and challenges.

## The brief (playing the role of "requirements")

You've been modeling CrunchRide in pieces all week — Lecture 1 gave you the entities and the ER diagram, Lecture 2 gave you the normalization theory, Lecture 3 walked through the DDL for the core tables. This mini-project asks you to build the **complete** system, including two pieces the lectures deliberately left as an exercise for you: **membership billing history** and **the many-to-many "favorite stations" feature** riders have been asking for. Treat the requirements below the way you'd treat a stakeholder's ask in a real job — precise enough to build from, but yours to model.

CrunchRide needs to track, in addition to everything already covered in the lectures:

1. **Billing history.** Each month, a rider is charged their plan's `monthly_price`. CrunchRide needs a record of every charge: which rider, which month, how much, and whether it was paid. (A rider can have failed or pending charges, not just paid ones — model a `status`.)
2. **Favorite stations.** Riders can mark stations as favorites (for quick access in the app). A rider can favorite many stations; a station can be favorited by many riders. There's no extra fact attached to a "favorite" beyond the pairing itself existing.

## Deliverable

A directory in your portfolio `c37-week-03/mini-project/` containing:

1. **`er-diagram.md`** (or an image) — the complete CrunchRide ER diagram, all entities from the lectures **plus** `billing_charges` and the favorites junction, with cardinality marked at both ends of every relationship.
2. **`schema.sql`** — full `CREATE TABLE` DDL for every table, in dependency order, with primary keys, foreign keys (each with a stated, justified `ON DELETE` behavior), `CHECK` constraints for every real business rule you can identify, and indexes on every foreign key.
3. **`seed.sql`** — enough realistic seed data to prove the schema works: at minimum 3 membership plans, 4 stations, 6 bikes, 5 riders, 8 rides (including at least 2 still in progress — `end_time IS NULL`), 3 maintenance records, 4 billing charges (mixed statuses), and at least 6 favorite-station pairings.
4. **`verification.sql`** — a set of queries that prove your constraints work: at least 4 `INSERT` statements that should each **fail** against a specific constraint (with a comment stating which constraint and why), immediately followed by the corrected version that succeeds.
5. **`why-sql-not-spreadsheet.md`** — a ~250-word note, written as if for a non-technical CrunchRide co-founder who suggested "can't we just track this in Google Sheets," explaining specifically (not generically) why this system's data needs to live in a constrained SQL database. Ground it in **CrunchRide's own data**, not the generic argument from Lecture 3 Section 8 — e.g., what specifically breaks if two people edit a spreadsheet row for the same bike at the same moment, using an actual CrunchRide scenario (two ops staff both trying to dock the same bike at two different stations, say).

## Milestones

Pace yourself; don't try to do all of this in one sitting.

- **Milestone 1 (30 min):** Extend the ER diagram — add `billing_charges` and the favorites junction to the diagram from Lecture 1, get the cardinality right for both new pieces.
- **Milestone 2 (60 min):** Write the full `schema.sql` — every table, key, constraint, and index, in the correct dependency order.
- **Milestone 3 (30 min):** Write and run `seed.sql`. Fix any constraint violations your seed data trips over — that's normal and expected, not a sign you did something wrong.
- **Milestone 4 (30 min):** Write `verification.sql` — deliberately break each constraint, confirm PostgreSQL rejects it, then insert the corrected version.
- **Milestone 5 (20 min):** Write `why-sql-not-spreadsheet.md`.

## Design requirements — read before you start

- **Billing charges** need a `status` column limited to a small, fixed set of values (`CHECK ... IN (...)`, following Lecture 3's pattern for `bikes.status`) — pick the set (e.g., `'pending'`, `'paid'`, `'failed'`) and justify it wouldn't be better as a lookup table (hint: it's small and stable — this is a legitimate `CHECK`-over-lookup-table call, the same judgment Lecture 3 Section 1 made for `bikes.status`).
- **The favorites junction table** needs a **composite primary key** on `(rider_id, station_id)` — not a surrogate `favorite_id`. Think about why: what would a surrogate key let happen that shouldn't be possible? (Hint: could a rider favorite the same station twice with a surrogate key design, and is that actually a problem worth preventing at the schema level?)
- **At least one `CHECK` constraint must relate two columns of the same row** to each other (like Lecture 3's `rides` table checking `end_time > start_time`) — a business rule a simple "column must be positive" check can't express.
- **Every foreign key needs a stated `ON DELETE` decision** — don't leave any at the silent default without at least a comment confirming that's the deliberate choice, not an oversight.
- **Seed data must include at least one row designed to test a boundary** — e.g., a ride where `end_time` is exactly equal to `start_time` plus one second (should this be allowed? your `CHECK` constraint's `>` vs `>=` choice determines the answer — make sure your seed data and your constraint agree).

## Rubric

| Criterion | Weight | "Great" looks like |
|---|---:|---|
| ER diagram completeness | 20% | Every entity from the lectures plus both new pieces; cardinality correct and marked at both ends |
| Schema correctness | 30% | `schema.sql` runs cleanly top to bottom; every table has a PK; every relationship from the diagram has a matching FK; the favorites table has the correct composite PK |
| Constraint depth | 20% | Real `CHECK` constraints tied to actual business rules (not decorative); at least one cross-column check; every `ON DELETE` is a stated decision |
| Verification rigor | 15% | `verification.sql` actually proves each constraint works — fail case and success case, both run and confirmed |
| The spreadsheet note | 15% | Specific to CrunchRide's real data and a real concurrent-editing scenario, not a restatement of the lecture's generic argument |

## Reflection (add to `why-sql-not-spreadsheet.md`, ~150 words)

1. Which table in your schema took the most back-and-forth to get right, and what changed between your first draft and the final version?
2. Where did writing the `CHECK` constraints reveal a business rule the plain-English brief never stated explicitly?
3. If CrunchRide added electric bikes with battery-charge tracking next quarter, what's the smallest schema change that would support it without breaking anything that already exists?

## Why this matters

This schema is the foundation every later week in this course builds on. Week 4 writes SQL and pandas pipelines against exactly this shape of data. Week 6 extends it toward enterprise master-data patterns. Week 10's analytics dashboards assume a normalized, trustworthy store underneath them — the same argument you just wrote in `why-sql-not-spreadsheet.md`, at production scale. Get the model right now, in one afternoon, and it pays back every week after this one.

When done: push your work, then take the [quiz](../quiz.md) and move on to Week 4 — Querying Data with SQL + Python.
