# Week 3 — Quiz

Fifteen questions. Lectures closed. Aim for 12/15 before starting Week 4. A mix of multiple-choice and short "what's wrong with this design" questions — the answer key at the bottom explains the *why*, not just the letter.

---

**Q1.** In ER modeling, what is the key test for whether a noun in a business description should become its own entity?

- A) Whether it appears more than once in the sentence
- B) Whether the business needs to track many of them individually, each with its own attributes
- C) Whether it's capitalized in the brief
- D) Whether a stakeholder mentioned it first

---

**Q2.** CrunchRide's `rides` table has both `start_station_id` and `end_station_id`, both referencing `stations`. Why is this legal and correct, not a design mistake?

- A) It isn't legal — a table can only have one foreign key to the same target table
- B) Each column represents a distinct relationship (where a ride started vs. ended), and a foreign key constraint doesn't care that another column references the same target
- C) PostgreSQL automatically merges them into one relationship
- D) `end_station_id` should actually reference `bikes`, not `stations`

---

**Q3.** Which of these is the most complete, correct definition of a primary key?

- A) A column that is indexed
- B) Any column with mostly-unique values
- C) A column (or minimal column set) that is unique, never null, minimal, and ideally stable
- D) The first column listed in a `CREATE TABLE` statement

---

**Q4.** A junction table resolving a many-to-many relationship between `students` and `courses`, with no extra attributes of its own, should typically have its primary key be:

- A) A new surrogate `enrollment_id`
- B) The composite `(student_id, course_id)`
- C) `student_id` alone
- D) No primary key is needed for a junction table

---

**Q5.** What specifically does a `FOREIGN KEY` constraint guarantee that careful application code alone cannot?

- A) Faster queries
- B) That the referenced value always exists (or is `NULL`, if allowed), enforced at the database layer for every writer — app code, scripts, bulk imports — forever, regardless of bugs in any one of them
- C) That the column is always `NOT NULL`
- D) Automatic indexing of the column

---

**Q6.** A table `order_lines(order_id, sku, quantity, product_name)` has composite primary key `(order_id, sku)`. `product_name` depends only on `sku`. This table violates:

- A) 1NF
- B) 2NF
- C) 3NF only, not 2NF
- D) No normal form — this is fine

---

**Q7.** A table `orders(order_id, customer_id, customer_email)` has primary key `order_id`. `customer_email` depends on `customer_id`, which is itself just a non-key column in this table (not part of the key). This is:

- A) A partial dependency, violating 2NF
- B) A transitive dependency, violating 3NF
- C) Perfectly normalized
- D) A 1NF violation

---

**Q8.** "The key, the whole key, and nothing but the key" is a one-line summary of which combination of normal forms?

- A) 1NF only
- B) 2NF only
- C) BCNF only
- D) 1NF + 2NF + 3NF together

---

**Q9.** Which of the three classic anomalies does this scenario describe? *"A flat table has no way to record a new customer's information until that customer places their first order."*

- A) Update anomaly
- B) Insertion anomaly
- C) Deletion anomaly
- D) Not an anomaly — this is expected behavior

---

**Q10.** In PostgreSQL, which type should you use for a `price` column, and why?

- A) `FLOAT`, because it's fast
- B) `TEXT`, because it preserves formatting like `$19.99`
- C) `NUMERIC(10,2)`, because floating-point types cannot represent decimal money values exactly
- D) `INTEGER`, because prices should be whole numbers

---

**Q11.** You delete a `stations` row that still has bikes docked at it (`bikes.station_id` references it). With the default foreign key behavior (no `ON DELETE` clause specified), what happens?

- A) The bikes are deleted too
- B) The bikes' `station_id` is set to `NULL`
- C) The delete is rejected — PostgreSQL blocks it because referencing rows still exist
- D) PostgreSQL silently ignores the constraint

---

**Q12.** Why does a foreign key column need an index added explicitly in PostgreSQL?

- A) It doesn't — PostgreSQL indexes every foreign key automatically
- B) PostgreSQL automatically indexes primary keys and `UNIQUE` columns, but not foreign key columns, even though they're queried and joined on constantly
- C) Indexes only work on primary keys
- D) Foreign keys are never queried, so an index would be wasted

---

**Q13.** A team wants to add a `plan_name` column directly on `riders`, copied from `membership_plans.name`, to avoid a join. Per Lecture 2's denormalization guidance, what's the single most important thing missing before this should be approved?

- A) Nothing — more columns are always better for read performance
- B) A specific, measured reason the join is actually a performance problem, plus a stated mechanism for keeping the copy in sync if the plan is ever renamed
- C) A `NOT NULL` constraint on the new column
- D) Renaming the column to something shorter

---

**Q14.** SQLite has a well-known gotcha regarding foreign keys. What is it?

- A) SQLite doesn't support the `REFERENCES` keyword at all
- B) Foreign key enforcement must be turned on per connection with `PRAGMA foreign_keys = ON;` — otherwise `REFERENCES` clauses are silently decorative
- C) SQLite requires foreign keys to be `UNIQUE` in the parent table
- D) SQLite only supports foreign keys between tables in different files

---

**Q15.** A `CHECK (end_time IS NULL OR end_time > start_time)` constraint on a `rides` table is doing what, specifically?

- A) Making `end_time` a required column
- B) Enforcing a rule about the relationship *between two columns of the same row* — a ride's end must be after its start, but only checked once the ride has actually ended
- C) Ensuring `end_time` and `start_time` are stored in the same time zone
- D) Automatically calculating `end_time` from `start_time`

---

## Answer key

<details>
<summary>Reveal after attempting</summary>

1. **B** — the test is whether the business tracks many individually-identified instances with their own attributes, not surface features like capitalization or mention count.
2. **B** — a foreign key constraint is per-column; two columns in the same table can each independently reference the same target table to represent two distinct relationships (start vs. end).
3. **C** — the full definition requires uniqueness, non-null, minimality (for composite keys), and ideally stability. "Indexed" and "mostly unique" are both incomplete or wrong.
4. **B** — with no attributes of its own, the natural composite key `(student_id, course_id)` both identifies the row and directly enforces "no duplicate enrollment," which a surrogate key would not do automatically.
5. **B** — the database enforces this for every writer, always, regardless of which code path or bug touches the table — the core argument for enforcing rules at the schema level, not just in application logic.
6. **B** — `product_name` depends on only part of the composite key (`sku` alone), which is precisely the 2NF partial-dependency violation.
7. **B** — `customer_email` depends on `customer_id`, a non-key column, rather than directly on `order_id` — a transitive dependency, a 3NF violation.
8. **D** — 1NF gives you a key that exists at all, 2NF requires depending on the *whole* key, 3NF requires depending on *nothing but* the key. Together: "the key, the whole key, and nothing but the key."
9. **B** — an insertion anomaly: you cannot record a fact (a new customer exists) independently of an unrelated fact (that customer has placed an order).
10. **C** — `NUMERIC(10,2)` stores exact decimal values; `FLOAT`/`REAL` use binary floating point, which cannot represent many decimal fractions (like money) exactly, leading to rounding errors.
11. **C** — the implicit default is `RESTRICT`/`NO ACTION`, which blocks the delete rather than silently cascading or nulling anything.
12. **B** — PostgreSQL automatically indexes primary keys and columns with a `UNIQUE` constraint, but a plain `REFERENCES` foreign key column gets no automatic index — you add it yourself.
13. **B** — per Lecture 2 Section 8 and Challenge 2's rubric, a real denormalization justification names the specific, measured pain and a concrete mechanism for keeping the redundant copy correct — not just "it'd be more convenient."
14. **B** — SQLite requires `PRAGMA foreign_keys = ON;` per connection; without it, `REFERENCES` clauses are parsed but not enforced, which silently defeats the whole point of writing them.
15. **B** — this is a cross-column `CHECK`: it relates two columns of the same row to each other, and the `end_time IS NULL OR ...` clause correctly allows an in-progress ride (no end time yet) to pass the check.

</details>

**Scoring:** 12+ → start Week 4. 9–11 → re-read the lecture sections behind your misses, especially normal-form definitions if that's where you lost points. <9 → re-read all three lectures from the top; this week's concepts are the foundation every later week in the course assumes you have solid.
