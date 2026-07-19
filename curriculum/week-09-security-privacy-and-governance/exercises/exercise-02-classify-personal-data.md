# Exercise 2 — Classify and Protect Personal Data Fields

Walk every table in `crunchcycles` column by column, decide what's personal data and how sensitive it is, then build a database view that masks the sensitive columns for roles that don't need to see them.

**Estimated time:** 1.5 hours.

## Setup

Use your `crunchcycles` database with `regions`, `employees`, `customers`, `products`, `orders`, `order_items`, and `app_users`.

## Tasks

1. **Build the classification table.** Create and populate a `column_classification` table, one row per column in `customers`, `employees`, and `app_users` (17–20 rows total):

```sql
CREATE TABLE column_classification (
    table_name       TEXT NOT NULL,
    column_name       TEXT NOT NULL,
    is_personal_data   BOOLEAN NOT NULL,
    sensitivity        TEXT NOT NULL,     -- 'none' | 'low' | 'medium' | 'high'
    protection_method  TEXT NOT NULL,     -- 'none' | 'access_control_only' | 'mask_for_low_privilege' | 'never_expose_raw'
    PRIMARY KEY (table_name, column_name)
);
```

For each column, decide: is it personal data (Lecture 2's definition)? What sensitivity tier? What protection does that tier require? A `password_hash` is personal-data-adjacent and `high` sensitivity even though it's already hashed — a leaked hash still enables offline cracking attempts, so it should never appear in any query result an application returns to a client, ever, hashed or not.

2. **Build a masking view for `customers`.** Support and low-trust roles need contact names to route a ticket, but not the full email address. Create a view:

```sql
CREATE VIEW customers_masked AS
SELECT
    customer_id,
    company_name,
    contact_name,
    -- show only the domain, not the mailbox: "j***@trailheadbikes.com"
    LEFT(email, 1) || '***@' || SPLIT_PART(email, '@', 2) AS email_masked,
    city,
    country,
    region_id
FROM customers;
```

3. **Grant deliberately.** `support_role` gets `SELECT` on `customers_masked` only, never on `customers` directly. `finance_role` and `sales_manager_role` get `SELECT` on the real `customers` table — they have a legitimate need for the full email.

```sql
REVOKE ALL ON customers FROM support_role;
GRANT SELECT ON customers_masked TO support_role;
GRANT SELECT ON customers TO finance_role, sales_manager_role;
```

4. **Write a one-paragraph justification** for each `high`-sensitivity column, in `justifications.md`, explaining specifically what harm exposure would cause and why the chosen protection method matches that harm — not a generic "PII is bad" sentence per row.

## Expected result

```sql
-- Querying as support_role (via SET ROLE support_role;)
SELECT * FROM customers_masked LIMIT 1;
--  customer_id | company_name    | contact_name | email_masked          | city   | country | region_id
--  1           | Trailhead Bikes | Jen Ortiz    | j***@trailheadbikes.com | Denver | USA     | 1

-- support_role querying the raw table directly should fail:
SELECT email FROM customers LIMIT 1;
-- ERROR: permission denied for table customers
```

## Done when

- [ ] `column_classification` has a row for every column in `customers`, `employees`, and `app_users`, with a defensible `sensitivity` tier for each.
- [ ] `customers_masked` exists and masks the email mailbox while preserving the domain.
- [ ] `support_role` can query `customers_masked` but is denied on raw `customers`.
- [ ] `justifications.md` explains each `high`-sensitivity column's specific risk and matching protection — no copy-pasted boilerplate across rows.
- [ ] `password_hash` is classified `high` sensitivity with `protection_method = 'never_expose_raw'`, and you can point to the line in your API code that guarantees it's never included in a JSON response.
