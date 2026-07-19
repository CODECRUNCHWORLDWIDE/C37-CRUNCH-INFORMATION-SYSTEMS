# Week 3 — Homework

Extra practice, spaced across the week rather than crammed into one sitting. None of this repeats the exercises or challenges verbatim — it's new domains and new angles on the same skills, so you feel the modeling process generalize instead of memorizing one worked example.

**Estimated total time:** 5 hours across the week (roughly 1h/day, Monday–Friday, alongside the day's other work).

## 1. Model a domain from a one-line prompt (1h)

Real requirements are sometimes even thinner than a transcript — sometimes it's one sentence and a follow-up conversation you have to imagine yourself asking. Pick **two** of the following one-liners and, for each, write a short (half-page) ER model: entities, attributes, primary keys, relationships with cardinality.

- "We're building a system for a gym: members sign up for classes, instructors teach classes, and we need to track attendance."
- "We run a food-delivery app: customers order from restaurants, drivers deliver the orders, and each order has multiple items from a restaurant's menu."
- "We're a university: students enroll in courses, courses are taught by professors in specific terms, and students get a grade per course per term."
- "We manage a apartment building: units are rented to tenants, tenants pay rent monthly, and maintenance requests get logged against a unit."

For each, explicitly note **one assumption you had to make** because the one-liner didn't specify it (e.g., for the university prompt: can a student retake a course in a later term, and does that need to be a distinct enrollment row from the first attempt?).

## 2. Find the normal form violation (1h)

For each table below, state which normal form it violates (1NF, 2NF, or 3NF) and why, following the exact reasoning style from Lecture 2.

**Table A:**
```
employee_projects(employee_id, project_id, employee_name, employee_dept, hours_logged)
-- PK: (employee_id, project_id)
```

**Table B:**
```
students(student_id, name, phone_numbers)
-- phone_numbers holds values like "555-1234, 555-5678"
```

**Table C:**
```
invoices(invoice_id, client_id, client_address, invoice_date, total_amount)
-- PK: invoice_id
```
*(Hint: this one is subtler than A or B — think carefully about whether `client_address` genuinely depends only on `invoice_id`, or transitively through `client_id`.)*

Write your answer for each as: *"Table X violates [normal form] because [column] depends on [what it actually depends on] rather than [the whole/only key]. Fix: [one-sentence description of the split]."*

## 3. Practice crow's-foot notation cold (45 min)

Without looking at any of this week's lectures, draw crow's-foot relationships (just the notation, small sketches, no full ER diagrams needed) for these five plain-English sentences:

1. "Every invoice belongs to exactly one client. A client can have many invoices, or none yet."
2. "Every course has exactly one required textbook. A textbook can be required by more than one course."
3. "A driver's license can belong to at most one person, and a person has at most one driver's license." *(This is the rare 1:1 case — think about why.)*
4. "A playlist contains many songs, and a song can appear on many playlists."
5. "Every shipment has exactly one destination address. A destination address might receive many shipments over time, or none."

Check your answers against Lecture 1, Section 4's symbol table once you've drawn all five, not before.

## 4. Constraint design drill (1h)

For each business rule below, write the exact PostgreSQL constraint (as a fragment you'd put in a `CREATE TABLE`) that enforces it. Assume reasonable column names.

1. "An employee's salary must be positive."
2. "A discount percentage must be between 0 and 100, inclusive."
3. "An order's ship date can't be before its order date."
4. "A username must be unique across the whole `users` table, case-insensitively." *(Harder — think about whether a plain `UNIQUE` on a `TEXT` column achieves case-insensitivity, and if not, what would. A `CHECK` combined with storing a normalized `LOWER(username)` in a generated column, or a `UNIQUE` index on `LOWER(username)`, are both defensible — pick one and show it.)*
5. "A subscription's `status` column can only ever be `'active'`, `'paused'`, or `'cancelled'`."

## 5. Write and defend one ON DELETE decision (45 min)

Pick any foreign key from your Exercise 3 library schema or your mini-project CrunchRide schema. Write a short paragraph (4–6 sentences) arguing for `RESTRICT`, `CASCADE`, or `SET NULL` as if defending the choice in a code review comment — cite a concrete scenario (a specific delete that would happen in the real system) that shows why your choice, and not one of the other two, is correct.

## 6. Read one real production schema (30 min)

Pick any open-source project you're even mildly curious about that ships a SQL schema file (search GitHub for `filename:schema.sql` or check a framework's migration files — Django, Rails, and most Postgres-backed open-source apps have one). Skim it for 20 minutes and answer, in 3–4 sentences: What's one constraint or design choice you recognize from this week's lectures? What's one thing in it you don't yet understand, and would want to ask about?

---

## Submission

Commit your answers as `homework.md` to your portfolio under `c37-week-03/homework/`. Partial answers are fine — the goal is repetitions across new domains, not a perfect score.
