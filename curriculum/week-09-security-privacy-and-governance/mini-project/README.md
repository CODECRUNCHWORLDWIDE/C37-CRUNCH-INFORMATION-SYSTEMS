# Mini-Project — Harden Crunch Cycles

> Take the Crunch Cycles system from Weeks 3–8 — schema, API, deployment — and harden it: real authentication, role-based *and* row-level access control, protected personal data, and a one-page governance policy an auditor could read in five minutes and trust.

**Estimated time:** 2.5–3 hours, best done Saturday after the exercises and challenges.

This is the week's capstone, and it's deliberately assembly, not new material — every piece was built once already this week, in Lecture 1, Lecture 2, Lecture 3, and Exercises 1–3. The skill this project tests is whether you can bring authentication, access control, privacy protection, and governance together into one coherent, working system, the way a real security-hardening sprint would, rather than as three disconnected homework answers.

---

## Deliverable

A directory in your portfolio `c37-week-09/mini-project/` containing:

1. `app.py`, `auth.py`, `rbac.py` — a working Flask API with login (Exercise 1) and RBAC on every route.
2. `enable_rls.sql` — row-level security enabled on `orders` and `customers`, with policies for `sales_rep` (own region only), `sales_manager` (all regions), and `finance` (all regions, read-only).
3. `column_classification.sql` — the populated classification table from Exercise 2, plus the `customers_masked` view and its grants.
4. `governance-policy.md` — **exactly one page**, covering ownership, retention, quality, and lineage for `customers`, `employees`, `orders`/`order_items`, and `app_users`.
5. `threat-summary.md` — a condensed version of Challenge 1's threat model: your top 5 risks and their mitigations, in table form, no more than half a page.
6. `demo.md` — a short script (a sequence of `curl` commands and their expected output) proving the system works end to end, for someone who wasn't in the room while you built it.

---

## Requirements

### 1. Authentication (from Lecture 1 / Exercise 1)

- `POST /api/v1/login` issues a JWT on valid credentials, generic `401` on invalid ones.
- Passwords are bcrypt-hashed; no plaintext password ever appears in a table, a log, or a response.
- Every protected route rejects a missing, expired, or invalid token.

### 2. Role-based access control

- At minimum four roles enforced: `sales_rep`, `sales_manager`, `finance`, `admin`.
- Each route's `@require_auth(roles=[...])` list matches the permission table from Lecture 1 — a `finance` user cannot successfully `POST /api/v1/orders`.

### 3. Row-level security (this is the requirement Exercise 1 explicitly deferred — build it now)

- `ALTER TABLE orders ENABLE ROW LEVEL SECURITY;` with a policy restricting `sales_rep_role` to their own region, using `current_setting('app.current_region')`.
- The API sets the Postgres session role and `app.current_region` **after** verifying the JWT and **before** running the query — show this in `app.py`.
- Prove it: a `sales_rep` querying with `SELECT * FROM orders` (no `WHERE` clause at all) from a `psql` session connected as their Postgres role still only sees their region. Include this proof in `demo.md`.

### 4. Personal data protection

- The `column_classification` table is populated for `customers`, `employees`, and `app_users`.
- `customers_masked` exists; `support_role` (create it if you haven't) can query it but is denied on the raw `customers` table.
- No API response, at any role, ever includes `password_hash`.

### 5. Governance policy (one page — this is a hard constraint, not a suggestion)

For each of `customers`, `employees`, `orders`/`order_items`, and `app_users`, state in `governance-policy.md`:
- **Owner** (a role, not a person's name).
- **Retention period + end-of-life mechanism** (delete, archive, or pseudonymize — pick one and say which).
- **One quality check** you'd run to know the table is trustworthy, with the actual SQL.
- **Lineage**, one sentence: where the data originates and where it's consumed.

### 6. Threat summary

- Your top 5 risks from Challenge 1 (or freshly derived if you haven't done Challenge 1 yet), each with a one-line mitigation, in a table.

---

## Milestones

- **Milestone 1 (45 min):** Login + RBAC wired into all routes; `demo.md`'s login and 401/403 checks pass.
- **Milestone 2 (45 min):** Row-level security enabled and proven with the `psql`-session test.
- **Milestone 3 (45 min):** Classification table, masking view, and grants in place; confirm `support_role` is denied on raw `customers`.
- **Milestone 4 (45 min):** Write `governance-policy.md` and `threat-summary.md` — hold yourself to the one-page limit; cutting content down to what matters *is* the exercise.

---

## Rubric

| Criterion | Weight | "Great" looks like |
|-----------|------:|--------------------|
| Authentication correctness | 20% | Hashed passwords, generic login errors, token expiry all verified working |
| RBAC + RLS enforcement | 25% | Every route matches its role table; RLS holds even against an unfiltered `SELECT *` |
| Personal data protection | 20% | Classification is complete and defensible; masking view genuinely denies raw access |
| Governance policy | 20% | Exactly one page; all four required elements present per table; specific, not generic |
| Threat summary + demo | 15% | Top-5 risks are specific and scored; `demo.md` runs cleanly for someone who wasn't there |

---

## Reflection (`notes.md`, ~200 words)

1. Which layer — RBAC or RLS — caught a mistake the other one would have missed? Give the specific scenario.
2. What was hardest to keep to one page in the governance policy, and what did you cut?
3. Of your threat model's "accepted risks," which one would you revisit first if Crunch Cycles tripled in size?
4. One thing you'd do differently if you were hardening a system that already had real users and couldn't have any downtime during the migration.

---

## Why this matters

Every control this week exists because a real system without it eventually fails an audit, a breach, or both — usually at the worst possible moment, with a customer or a regulator watching. This mini-project is the shape of a real security-hardening sprint: not one clever fix, but authentication, access control, data protection, and governance working together, documented well enough that someone who joins the team next month can trust it without re-deriving it from the code.

When done: push, then take the [quiz](../quiz.md) and start [Week 10 — Analytics & business intelligence](../../week-10-analytics-and-business-intelligence/).
