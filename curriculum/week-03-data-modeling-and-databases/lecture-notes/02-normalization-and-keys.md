# Lecture 2 — Normalization & Keys

> **Duration:** ~2 hours. **Outcome:** You can define primary/foreign keys precisely, name the three classic update anomalies, walk a flat table through 1NF → 2NF → 3NF → BCNF step by step, and state — with a real reason — when denormalizing on purpose is the right call instead of a mistake.

Lecture 1 gave you a diagram. This lecture gives you the *theory* that tells you the diagram is actually correct — not just plausible-looking, but provably free of a specific, named category of bug. That theory is **normalization**, and it rests on two ideas you need cold: **keys** and **functional dependencies**.

## 1. Keys, precisely

You used "primary key" informally in Lecture 1. Here's the precise version, because the exact wording is what tells you whether a constraint is doing its job.

**Primary key (PK):** a column (or minimal set of columns) such that:

1. **Uniqueness** — no two rows ever have the same value.
2. **Non-null** — every row has a value (you can't identify a row by "unknown").
3. **Minimality** — if it's more than one column, *every* column is needed for uniqueness; dropping any one of them would allow duplicates.
4. **Stability** — it shouldn't need to change once assigned (this is the practical argument for surrogate keys from Lecture 1).

**Candidate key:** any column or column-set that *could* serve as the primary key (satisfies uniqueness + non-null). A table can have several candidate keys; you pick one as PK and the rest become `UNIQUE` constraints. Example: `Rider` has two candidate keys — `rider_id` (surrogate) and `email` (natural, assuming emails are unique and required). You'd make `rider_id` the PK and put a `UNIQUE` constraint on `email`.

**Foreign key (FK):** a column (or column-set) in one table whose values must match a primary key (or unique key) value in another table, or be `NULL` if the relationship is optional. A foreign key is how a relationship from your ER diagram actually gets *enforced* by the database instead of just hoped for.

**Composite key:** a primary or foreign key made of more than one column. Junction tables (resolving M:N relationships) very often have a composite primary key made of the two foreign keys they join — you'll build one of these in the mini-project.

### Why the database should enforce this, not your application code

You could, in theory, just be careful in your Python/JavaScript code and never insert a `ride` with a `bike_id` that doesn't exist. In practice: multiple developers touch the code over years, one bug slips through code review, one bulk-import script skips validation "just this once," and now you have orphaned rows nobody notices until a report breaks. A `FOREIGN KEY` constraint makes that class of bug **impossible at the database layer**, regardless of which application, script, or careless analyst touches the table next. This is the core argument for a real database over a spreadsheet: **a spreadsheet has no way to say "this value must exist in that other sheet, or the cell must be empty" and enforce it on every single edit, forever, no matter who's editing.**

## 2. Functional dependencies — the idea normalization is built on

A **functional dependency** (FD), written `A → B`, means: *"given a value of A, there is only ever one possible value of B."* Read it "A determines B."

Examples from a flat, unnormalized `orders_flat` table (the kind you'll normalize in Exercise 2):

- `order_id → order_date` (each order has exactly one date)
- `customer_id → customer_email` (each customer has exactly one email — a customer *fact*, not an order fact)
- `product_sku → product_name` (each SKU has exactly one name)
- `order_id, product_sku → quantity` (quantity depends on *both* — which order, which line item within it)

The whole discipline of normalization is: **find every column, figure out what it truly depends on, then organize your tables so every non-key column depends on the *whole* primary key, and nothing but the key.** That sentence — "the key, the whole key, and nothing but the key" — is the entire idea of 3NF in eleven words. Everything below just makes it rigorous and gives you a repeatable procedure.

## 3. Why normalize — the three anomalies

Un-normalized (flat, redundant) tables cause three specific, nameable bugs. Normalization exists to eliminate them, not as an academic exercise.

Picture a flat `orders_flat` table where every row repeats the customer's email and city:

| order_id | customer_name | customer_email | customer_city | product_sku | quantity |
|---|---|---|---|---|---|
| 1001 | Grace Hopper | grace@ex.com | Austin | SKU-01 | 2 |
| 1001 | Grace Hopper | grace@ex.com | Austin | SKU-04 | 1 |
| 1002 | Ada Lovelace | ada@ex.com | Seattle | SKU-01 | 3 |

**Insertion anomaly:** you can't record a new customer until they place an order — there's no table for "customer" independent of "order," so a customer with zero orders simply cannot exist in this design.

**Update anomaly:** Grace changes her email. You must find and update *every row* that mentions her (here, 2 rows; in a real system, thousands). Miss one and the same customer now has two different emails on file — a direct contradiction with no way to tell which is correct.

**Deletion anomaly:** Ada cancels her only order. Delete that row and you've silently lost the fact that Ada is a customer at all — her email and city vanish along with the order, even though "Ada is a customer" and "Ada placed order 1002" are two independent facts that shouldn't be tied together.

Every step of normalization below is aimed at one or more of these three anomalies. When you're deciding whether a table needs to be split further, ask: *"which of the three anomalies would this redundancy cause?"* — if you can't name one, you may not need to split it further (more on that in the BCNF section, and in Challenge 2 on deliberate denormalization).

## 4. First Normal Form (1NF)

**Rule:** every column holds a single, atomic value (no repeating groups, no comma-separated lists, no arrays-as-strings) — and every row is uniquely identifiable.

A common 1NF violation:

```
| rider_id | name         | favorite_stations        |
|----------|--------------|---------------------------|
| 1        | Grace Hopper | "Station A, Station C"    |
```

`favorite_stations` packs multiple values into one cell. You can't index it, can't `JOIN` on it cleanly, can't ask "which riders favorite Station C" without string parsing. The 1NF fix is a separate table:

```
riders(rider_id, name)
rider_favorite_stations(rider_id, station_id)   -- one row per (rider, station) pair
```

This is exactly the junction-table pattern from Lecture 1's many-to-many discussion — 1NF and "resolve M:N with a junction table" are the same instinct applied at different levels.

## 5. Second Normal Form (2NF)

**Rule:** must already be in 1NF, **and** every non-key column must depend on the **whole** primary key — not just part of it. 2NF only becomes a meaningful question when the primary key is **composite** (more than one column); if your PK is a single surrogate column, you're automatically in 2NF once you're in 1NF.

Take an order-line table with a composite key `(order_id, product_sku)`:

```
order_lines(order_id, product_sku, quantity, product_name, product_category)
```

- `quantity` depends on **both** `order_id` and `product_sku` — correct, it's a genuine order-line fact ("how many of *this* product on *this* order").
- `product_name` and `product_category` depend **only** on `product_sku` — they'd be identical for SKU-01 on every order it ever appears on. This is a **partial dependency** and it violates 2NF.

The fix: pull `product_name`/`product_category` into their own `products` table, keyed by `product_sku` alone.

```
products(sku PK, name, category)
order_lines(order_id, sku, quantity)   -- FK sku → products.sku
```

Now `order_lines` has no columns that depend on only *part* of its composite key — `quantity` genuinely needs both `order_id` and `sku` to make sense.

## 6. Third Normal Form (3NF)

**Rule:** must already be in 2NF, **and** no non-key column may depend on another **non-key** column (no *transitive* dependencies). Every non-key column must depend on the key **directly**, not through another non-key column.

Back to the flat orders example — imagine the table also carried `sales_rep_name` and `sales_rep_region`:

```
orders(order_id, customer_id, customer_email, sales_rep_id, sales_rep_name, sales_rep_region)
```

`customer_email` depends on `customer_id`, not on `order_id` directly. `sales_rep_region` depends on `sales_rep_name`/`sales_rep_id`, not on `order_id` directly. These are **transitive dependencies**: `order_id → customer_id → customer_email`. The middleman (`customer_id`) is a non-key column standing between the actual key and the fact.

The 3NF fix separates each independently-varying fact into its own table:

```
customers(customer_id PK, email)
sales_reps(sales_rep_id PK, name, region)
orders(order_id PK, customer_id FK, sales_rep_id FK)
```

Now `orders` only holds facts that are truly about the order itself (which customer, which rep, when) — everything about the customer lives with the customer, everything about the rep lives with the rep. This is precisely the design that removes all three anomalies from Section 3: you can insert a customer with no orders, updating an email touches exactly one row, and deleting an order never touches customer data.

**"The key, the whole key, and nothing but the key"** — that's 1NF (atomic values, a key exists) + 2NF (whole key) + 3NF (nothing but the key) in one sentence. Most real-world schemas stop at 3NF; it removes essentially all the practically damaging redundancy for the effort involved.

## 7. Boyce-Codd Normal Form (BCNF)

**Rule:** a stricter version of 3NF for a specific edge case — a table can satisfy 3NF and still have an anomaly if it has **multiple overlapping candidate keys** and a non-candidate-key column determines part of a candidate key.

The classic textbook example: a table `(student, course, instructor)` where each course is taught by exactly one instructor, but each instructor teaches only one course (a small school). Here `instructor → course` is a real FD, but `instructor` isn't a candidate key on its own (the actual key is `student, course`). This passes 3NF (no non-key column depends transitively on another non-key column — `course` isn't a "non-key column" here, it's part of the key) but still allows the anomaly: if you delete the last student in a course, you lose the fact that the instructor teaches that course.

**In practice:** BCNF violations are rare in typical business schemas — they only show up with overlapping composite candidate keys, which is unusual outside academic scheduling and a few other niche domains. Know it exists, know the general shape (multiple overlapping keys, a non-key-column-that's-really-a-partial-key determining part of another key), and don't lose sleep over hand-checking every table against it. 3NF is where 95% of real schema design work happens.

## 8. When to stop normalizing on purpose

Normalization removes redundancy, and redundancy is what causes anomalies — but normalization is not free. Every extra table is an extra `JOIN` a query needs, and every `JOIN` costs performance and reading effort. Two situations where **staying at 3NF (or even deliberately backing off it) is the right call**, not a mistake:

1. **Read-heavy reporting tables.** If a dashboard runs a query that joins six tables every time a user loads a page, and the underlying data changes rarely, storing a precomputed, redundant summary column (or table) can be the right trade — as long as you have a clear plan for keeping it in sync (a trigger, a scheduled job) and you document *why* it's there.
2. **Genuinely fixed, tiny reference data.** A `country_code` column that never needs a lookup table because the values are a stable, small, universally known set (ISO country codes) — adding a `countries` table with two columns buys you almost nothing and costs a join on every query that touches it.

The discipline is: **normalize by default, denormalize by name.** Every deliberate denormalization should be a documented decision with a stated reason ("we accept this redundancy because X, and we keep it correct by Y") — not something a developer backed into because joins felt annoying. Challenge 2 this week has you write exactly that kind of justification.

## 9. Constraints — turning the model into enforcement

Once your tables are normalized, PostgreSQL gives you the vocabulary to make the rules real:

```sql
CREATE TABLE riders (
    rider_id     SERIAL PRIMARY KEY,
    email        TEXT NOT NULL UNIQUE,
    plan_id      INTEGER NOT NULL REFERENCES membership_plans(plan_id),
    signup_date  DATE NOT NULL DEFAULT CURRENT_DATE,
    CHECK (signup_date <= CURRENT_DATE)
);
```

| Constraint | What it enforces | Which anomaly / rule it backs |
|---|---|---|
| `PRIMARY KEY` | uniqueness + not null on the identifying column(s) | the definition of a key, Section 1 |
| `NOT NULL` | a column can never be left empty | data completeness the business actually requires |
| `UNIQUE` | a non-key column (or set) never repeats | candidate keys that aren't the chosen PK |
| `FOREIGN KEY ... REFERENCES` | a value must match a real row elsewhere, or be `NULL` | the relationships from your ER diagram, Lecture 1 |
| `CHECK (expression)` | any boolean rule about a row's own values | business rules ("price must be positive," "end_time > start_time") |
| `DEFAULT` | fills a value automatically when omitted | reduces `NULL`s that shouldn't really be optional |

Lecture 3 goes deep on writing all of these against the CrunchRide model — types, `ON DELETE` behavior, indexes — turning this week's diagram into a schema PostgreSQL will actually run.

## 10. Check yourself

- Define "functional dependency" in your own words, using an example that isn't from this lecture.
- Name the three classic anomalies. For each, describe a concrete scenario (can reuse `orders_flat`) where it bites.
- A table has PK `(order_id, product_sku)` and a column `warehouse_location` that depends only on `product_sku`. Which normal form does this violate, and what's the fix?
- Why is "the key, the whole key, and nothing but the key" a reasonable one-line summary of 1NF+2NF+3NF together?
- Give one legitimate business reason to denormalize on purpose, and state what you'd need to do to keep the redundant data correct.
- Why does a `FOREIGN KEY` constraint matter even if your application code is "always careful"?

Next: [Lecture 3 — From Model to SQL Schema](./03-from-model-to-schema.md), where the CrunchRide ER diagram and this normalization theory turn into real, runnable `CREATE TABLE` statements.

## Further reading

- **PostgreSQL — Constraints:** <https://www.postgresql.org/docs/current/ddl-constraints.html>
- **PostgreSQL — Foreign keys and `ON DELETE`/`ON UPDATE` actions:** <https://www.postgresql.org/docs/current/ddl-constraints.html#DDL-CONSTRAINTS-FK>
- **"A Simple Guide to Five Normal Forms in Relational Database Theory" (Kent, classic 1983 paper, still the clearest explanation):** <https://www.bkent.net/Doc/simple5.htm>
