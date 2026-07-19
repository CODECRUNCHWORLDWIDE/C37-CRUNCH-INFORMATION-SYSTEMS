# Exercise 3 — Write CREATE TABLE DDL with Constraints

**Goal:** Turn the library ER diagram from Exercise 1 into full, runnable PostgreSQL DDL — every table, every primary/foreign key, and at least one `CHECK` constraint per table that enforces a real business rule from the brief, not a decorative one.

**Estimated time:** 1.5 hours.

## Setup

Have your `er-diagram.md` from Exercise 1 open — you're implementing that design now, not designing from scratch. If your Exercise 1 model differed from the "expected" shape below in a defensible way (e.g., you modeled individual copies), implement *your* design; just make sure it's internally consistent.

Create a fresh scratch database or schema so this doesn't collide with Exercise 2's tables:

```bash
createdb library      # or: psql crunchride -c 'CREATE SCHEMA library;'
psql library
```

## Tasks

Write all DDL in a single file, `library-schema.sql`, structured in dependency order (tables with no foreign keys first) exactly as Lecture 3 modeled for CrunchRide.

1. **`categories`** — `category_id` PK, `name` (`NOT NULL UNIQUE`), `shelf_location` (`NOT NULL`).

2. **`books`** — `book_id` PK, `title` (`NOT NULL`), `author` (`NOT NULL`), `isbn` (`NOT NULL UNIQUE` — a 13-character `CHECK` on length is a nice touch), `copies_owned` (`NOT NULL`, `CHECK (copies_owned > 0)`), `category_id` FK → `categories`.

3. **`members`** — `member_id` PK, `name` (`NOT NULL`), `email` (`NOT NULL UNIQUE`), `membership_start` (`NOT NULL DEFAULT CURRENT_DATE`).

4. **`loans`** — `loan_id` PK, `member_id` FK → `members` (`NOT NULL`), `book_id` FK → `books` (`NOT NULL`), `checkout_date` (`NOT NULL DEFAULT CURRENT_DATE`), `due_date` (`NOT NULL` — think about whether this should be computed or stored explicitly, and write a `CHECK` relating it to `checkout_date`), `return_date` (nullable — a loan not yet returned), with a `CHECK` that `return_date`, if present, is not before `checkout_date`.

5. **`holds`** — `hold_id` PK, `member_id` FK → `members` (`NOT NULL`), `book_id` FK → `books` (`NOT NULL`), `hold_date` (`NOT NULL DEFAULT CURRENT_DATE`).

For **every** foreign key you write, decide and state (as a SQL comment right above the column) what `ON DELETE` behavior it should have, using Lecture 3 Section 4's decision table (`RESTRICT`, `CASCADE`, or `SET NULL`) — and *why*. At least one table should use something other than the default `RESTRICT`, with a justified reason.

Add indexes on every foreign key column, following Lecture 3 Section 5.

## Verify your own constraints

This is the part beginners skip and shouldn't: for **each** constraint you wrote, write one `INSERT` (in a separate scratch file, `constraint-tests.sql`) designed to violate it, run it, and confirm PostgreSQL rejects it with the error you expected. Then write one valid `INSERT` per table that succeeds. Comment each test with what you expect to happen (`-- should FAIL: isbn too short` / `-- should SUCCEED`).

Example shape:

```sql
-- should FAIL: copies_owned must be positive
INSERT INTO books (title, author, isbn, copies_owned, category_id)
VALUES ('Test Book', 'Test Author', '9780000000000', 0, 1);

-- should SUCCEED
INSERT INTO books (title, author, isbn, copies_owned, category_id)
VALUES ('Test Book', 'Test Author', '9780000000000', 3, 1);
```

## Done when…

- [ ] `library-schema.sql` creates all 5 tables in an order PostgreSQL accepts without a "relation does not exist" error.
- [ ] Every table has a primary key; every foreign key names an explicit, justified `ON DELETE` behavior in a comment.
- [ ] At least one `CHECK` constraint per table enforces a real rule from the Exercise 1 brief (not a placeholder like `CHECK (true)`).
- [ ] Every foreign key column has an index.
- [ ] `constraint-tests.sql` has at least one FAIL-expected and one SUCCEED-expected `INSERT` per table, and you've actually run them and confirmed the outcome matches the comment.

## Stretch

- Add a `CHECK` (or, if you want to go further than this week requires, a trigger — not expected, just mentioned) enforcing "a loan can't be created for a book with zero available copies" — think hard about whether a plain `CHECK` constraint (which can only see the row being inserted, not the rest of the table) can actually express this rule, and write a sentence explaining what you found. *(This is a genuinely hard constraint to express in pure DDL — discovering why is the point of the stretch, not solving it perfectly.)*
- Rewrite the schema for SQLite, applying every fallback note from Lecture 3 Section 7 (`INTEGER PRIMARY KEY`, `PRAGMA foreign_keys = ON`, `TEXT` for dates). Confirm it loads and the same constraint tests behave the same way.

## Submission

Commit `library-schema.sql` and `constraint-tests.sql` to your portfolio under `c37-week-03/exercise-03/`.
