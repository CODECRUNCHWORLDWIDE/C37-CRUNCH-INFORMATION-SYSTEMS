# Exercise 2 — Identify Master vs. Transactional Data

**Goal:** Practice the classification test from Lecture 2 — "if I deleted every transaction tomorrow, would this row still make sense to keep?" — until it's automatic, on both this week's schema and a schema you've never seen.

**Estimated time:** 45–60 minutes.

## Setup

You need Lecture 2 open for reference. Part B uses a brand-new schema described in prose — no need to create it, just reason about it.

## Part A — Classify every table in this week's schema

Fill in the table. For each one, give the one-sentence "why" — don't just write Master/Transactional.

| Table | Master or Transactional | Why (one sentence) |
|---|---|---|
| `regions` | ? | ? |
| `employees` | ? | ? |
| `customers` | ? | ? |
| `products` | ? | ? |
| `orders` | ? | ? |
| `order_items` | ? | ? |
| `suppliers` | ? | ? |
| `product_suppliers` | ? | ? |
| `purchase_orders` | ? | ? |
| `purchase_order_items` | ? | ? |
| `crm_opportunities` | ? | ? |

`product_suppliers` is the trickiest row here — it's a many-to-many *relationship* between two master tables. Argue for your classification; there's a defensible case for either answer as long as you explain your reasoning (see Lecture 2's note on `crm_opportunities` being a similar edge case).

## Part B — Column-level classification within a single table

Not every column in a "transactional" table is itself transactional, and not every column in a "master" table is purely descriptive. Consider Crunch Cycles' `orders` table:

```sql
CREATE TABLE orders (
    order_id      INTEGER PRIMARY KEY,
    customer_id   INTEGER NOT NULL,
    employee_id   INTEGER NOT NULL,
    order_date    DATE NOT NULL,
    ship_date     DATE,
    status        TEXT NOT NULL,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id),
    FOREIGN KEY (employee_id) REFERENCES employees(emp_id)
);
```

For each column, say whether it's:
- **(a)** transactional data native to this row, or
- **(b)** a foreign key *pointing at* master data (the column itself isn't master data — it's a reference to it).

1. `order_id`
2. `customer_id`
3. `employee_id`
4. `order_date`
5. `ship_date`
6. `status`

## Part C — A new scenario: a hospital's patient-scheduling system

You're handed this (simplified, invented) schema for a clinic's scheduling system. No need to build it — just read it.

```
patients(patient_id, first_name, last_name, date_of_birth, insurance_provider, primary_physician_id)
physicians(physician_id, first_name, last_name, specialty, npi_number)
appointment_types(type_id, type_name, default_duration_minutes, billing_code)
appointments(appointment_id, patient_id, physician_id, type_id, scheduled_at, status, checked_in_at)
```

1. Classify each of the four tables as master or transactional, with your one-sentence reason.
2. In `appointments`, which columns are foreign keys pointing at master data, and which are transactional data native to the appointment event itself?
3. Name **one master-data quality problem** this clinic could plausibly have (think about what Lecture 2 called duplication, completeness, and consistency) and write the one query (in prose or actual SQL — your choice) you'd run to detect it.

## Done when…

- [ ] Part A's table is fully filled with a real "why" for each row, not a repeated boilerplate sentence.
- [ ] Part B correctly separates the two foreign keys from the four native columns.
- [ ] Part C classifies all four hospital tables and names a specific, plausible data-quality problem (not a generic "data could be wrong").

## Stretch

`employees.manager_id` (from Week 4's schema) is a foreign key that points back into the *same* master table it lives in — a self-referencing relationship. Is `manager_id` itself master data, transactional data, or something else? Write two or three sentences defending your answer.

## Submission

Commit `exercise-02.md` to your portfolio under `c37-week-06/exercise-02/`.
