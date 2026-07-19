# Week 9 — Security, Privacy & Governance

> **Goal:** by Sunday you can log a stranger into the Crunch Cycles system, prove they are who they claim to be, and guarantee — at the database, not just in application code — that they can only see and touch what their role permits; you can point at every personal-data column in the schema and say what protects it and why; and you can hand a stakeholder a one-page policy that says who owns the data, how long it's kept, and what happens if someone attacks it.

Welcome back to **C37 · Crunch Information Systems**. Week 7 gave Crunch Cycles an API. Week 8 put that API on a real server, reachable from anywhere on the internet. That is exactly the moment a system stops being a toy and starts being a target. Right now, anyone who finds the URL from Week 8's deployment can call `GET /api/v1/orders` and read every customer's name, email, and order history — the API key check from Week 7 was a start, but one shared key with no roles, no audit trail, and no database-level enforcement is not access control, it's a locked door with the key taped to it.

This week fixes that, in three layers. **Authentication and access control** (Lecture 1) answers "who are you, and what are you allowed to do" — with real password hashing, roles, and PostgreSQL row-level security that holds even if application code has a bug. **Data privacy and compliance** (Lecture 2) answers "what personal data are we holding, and what do we owe the people it belongs to" — the GDPR-shaped questions every system with a `customers` table eventually has to answer, whether or not it ever serves an EU resident. **Data governance** (Lecture 3) answers the boring question that prevents the expensive one: who owns this table, how long do we keep it, and can we trust what's in it. By Saturday's mini-project you'll have hardened the whole Crunch Cycles system and written the governance policy a real auditor would ask to see first.

## Learning objectives

By the end of this week, you will be able to:

- **Distinguish** authentication from authorization, and implement both for a real API: hashed-password login, role-based access control (RBAC), and PostgreSQL row-level security (RLS) enforced at the database layer.
- **Apply** the principle of least privilege to a schema and a running application — naming exactly what each role can and cannot do, and why.
- **Manage secrets safely** — environment variables, `.env` files kept out of Git, hashed (never plaintext) credentials, and rotating a leaked key without downtime.
- **Classify personal data** at the column level, distinguish PII from sensitive PII, and protect it with masking, encryption-in-transit, and access restrictions matched to that classification.
- **Reason about privacy regulation** (GDPR-style principles) in concrete, implementable terms: lawful basis, consent, data minimization, and the data subject rights (access, rectification, erasure, portability) a system must be able to fulfill.
- **Define and write** a data governance policy: ownership, retention, quality, and lineage — the four questions every table in a production system should have answered before it ships.
- **Threat-model a system** using a structured method (STRIDE), rank the risks that matter, and propose mitigations sized to the actual threat, not to anxiety.

## Prerequisites

- Weeks 1–8 of this course, or equivalent comfort with: a normalized PostgreSQL schema (Week 3), SQL joins/aggregation (Week 4), a Flask REST API with API-key auth (Week 7), and a system deployed somewhere reachable (Week 8).
- Python 3.10+ with `pip` working, and the PostgreSQL 16+ `crunchcycles` database from Weeks 3–7 (schema reproduced below if you need it fresh).
- Comfort running `psql` commands and reading a `CREATE TABLE` statement. No prior security, cryptography, or legal background required — this week teaches the concepts you need, not a law degree.

## Setup

**1. Confirm your database.** If your `crunchcycles` Postgres database from earlier weeks is still around, you're set. If not, recreate the Week 4 schema (regions, employees, customers, products, orders, order_items) from that week's README, then continue below.

**2. Add this week's table — application users.** This week introduces real login: people (not just a shared API key) authenticate as themselves, and their role determines what they can do.

```sql
CREATE TABLE app_users (
    user_id        SERIAL PRIMARY KEY,
    username       TEXT NOT NULL UNIQUE,
    email          TEXT NOT NULL UNIQUE,
    password_hash  TEXT NOT NULL,               -- bcrypt hash — NEVER plaintext, ever
    role           TEXT NOT NULL,               -- 'sales_rep' | 'sales_manager' | 'finance' | 'support' | 'admin'
    employee_id    INTEGER,                     -- links to employees.emp_id; NULL for non-employee accounts
    is_active      BOOLEAN NOT NULL DEFAULT TRUE,
    last_login_at  TIMESTAMPTZ,
    created_at     TIMESTAMPTZ NOT NULL DEFAULT now(),
    FOREIGN KEY (employee_id) REFERENCES employees(emp_id),
    CHECK (role IN ('sales_rep','sales_manager','finance','support','admin'))
);
```

Never `INSERT` a password directly in SQL — a plaintext value in a `.sql` file defeats the entire point. Seed users with a Python script instead, which is also how you'll do it in Lecture 1 and Exercise 1:

```bash
python3 -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install flask flask-cors bcrypt pyjwt sqlalchemy psycopg2-binary python-dotenv
```

- **bcrypt** — the password-hashing library you'll use; never `hashlib.md5`/`sha256` alone for passwords (Lecture 1 explains exactly why).
- **pyjwt** — issues and verifies the signed session tokens your API will use after login, replacing Week 7's single shared API key.
- **python-dotenv** — loads secrets from a local `.env` file that is **never committed**. Run `echo ".env" >> .gitignore` right now, before you create the file.

**3. Sanity-check the install.**

```bash
python3 -c "import bcrypt, jwt; print('ok')"
```

## Weekly schedule

Adds up to the course's full-time target of **~28 hours**.

| Day | Focus | Lectures | Exercises | Challenges | Quiz/Read | Homework | Mini-Project | Daily Total |
|-----------|-------------------------------------------|---------:|----------:|-----------:|----------:|---------:|-------------:|------------:|
| Monday | Authentication, RBAC, row-level security | 2h | 1.5h | 0h | 0.5h | 1h | 0h | 5h |
| Tuesday | Secrets management; build the login API | 0h | 1h | 0h | 0.5h | 1h | 0h | 2.5h |
| Wednesday | Data privacy, GDPR principles, consent | 2h | 1.5h | 0h | 0.5h | 1h | 0h | 5h |
| Thursday | Classify + protect personal data | 0h | 1h | 1h | 0.5h | 1h | 1h | 4.5h |
| Friday | Governance: ownership, retention, lineage | 2h | 0h | 2h | 0.5h | 1h | 1.5h | 7h |
| Saturday | Mini-project (harden the system + policy) | 0h | 0h | 0h | 0h | 0h | 2.5h | 2.5h |
| Sunday | Quiz + review | 0h | 0h | 0h | 1h | 0h | 0h | 1h |
| **Total** | | **6h** | **5h** | **3h** | **3.5h** | **5h** | **5h** | **~28h** |

## How to navigate this week

Work top to bottom. Each piece assumes the ones above it.

| # | File | What's inside | ~Time |
|--:|------|---------------|------:|
| 1 | [lecture-notes/01-access-control-and-authn.md](./lecture-notes/01-access-control-and-authn.md) | AuthN vs. authZ, password hashing, RBAC, PostgreSQL row-level security, secret management | 2h |
| 2 | [lecture-notes/02-data-privacy-and-compliance.md](./lecture-notes/02-data-privacy-and-compliance.md) | Privacy principles, personal-data handling, consent, data subject rights, what regulation actually requires | 2h |
| 3 | [lecture-notes/03-data-governance.md](./lecture-notes/03-data-governance.md) | Ownership, retention, quality, and lineage — the policies that keep data trustworthy over time | 2h |
| 4 | [exercises/exercise-01-add-role-based-access.md](./exercises/exercise-01-add-role-based-access.md) | Add hashed-password login and RBAC to the Crunch Cycles API | 1.5h |
| 5 | [exercises/exercise-02-classify-personal-data.md](./exercises/exercise-02-classify-personal-data.md) | Build a column-level data classification and mask sensitive fields for low-privilege roles | 1.5h |
| 6 | [exercises/exercise-03-write-a-retention-policy.md](./exercises/exercise-03-write-a-retention-policy.md) | Write and implement a real data retention policy for three tables | 1h |
| 7 | [challenges/challenge-01-threat-model-the-system.md](./challenges/challenge-01-threat-model-the-system.md) | Threat-model the whole Crunch Cycles system with STRIDE and rank the risks | 1.5h |
| 8 | [challenges/challenge-02-pass-a-privacy-audit.md](./challenges/challenge-02-pass-a-privacy-audit.md) | Redesign a deliberately non-compliant system to pass a privacy audit checklist | 1.5h |
| 9 | [mini-project/README.md](./mini-project/README.md) | Harden Crunch Cycles: authentication, RBAC + RLS, protected PII, one-page governance policy | 2.5h |
| 10 | [homework.md](./homework.md) | Extra practice, spaced out | 5h |
| 11 | [quiz.md](./quiz.md) | 15 self-check questions + answer key | 1h |
| 12 | [resources.md](./resources.md) | Official docs, standards, and tools to install | — |

## By the end of this week you can…

- Build a login flow with hashed passwords and signed session tokens, and explain in one sentence why a plaintext password column is a resignation-worthy mistake.
- Design roles with least privilege, enforce them at both the application layer (RBAC) and the database layer (row-level security), and explain why you need both, not either.
- Look at any table and say, precisely, which columns are personal data, what protects each one, and what a data subject is legally owed regarding it.
- Write a governance policy — ownership, retention, quality, lineage — that a new engineer or an external auditor could read and understand in five minutes.
- Threat-model a real system, separate the risks worth mitigating now from the ones worth accepting, and defend that ranking out loud.

## Up next

[Week 10 — Analytics & business intelligence](../week-10-analytics-and-business-intelligence/) — now that the system is secure and governed, we build the warehouse and dashboards that turn its data into decisions.

---

*Part of the Code Crunch Worldwide open curriculum · GPL-3.0 · If you find errors, please open an issue or PR.*
