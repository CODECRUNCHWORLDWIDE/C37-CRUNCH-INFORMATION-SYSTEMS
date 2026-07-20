# Authentication & Access Control — AuthN vs AuthZ, RBAC, Row-Level Security, and Secrets

Week 7's Crunch Cycles API had exactly one security check: a shared string, `sk_warehouse_9f2a`, sent in a header. Anyone with that string could do anything the API could do — read every customer's email, every employee's salary if you'd exposed it, place or cancel any order. That is not access control. It's a single lock on a building with one key photocopied for everyone who works there, and no record of who used it to walk through which door. This lecture builds the real thing: people log in as *themselves*, their role determines what they can touch, and the database — not just your Python code — refuses to hand over rows they shouldn't see.

## Authentication vs. authorization — two different questions

These two words get used interchangeably in casual conversation and that habit causes real bugs. Keep them separate:

- **Authentication (AuthN)** — *"Who are you?"* Proving identity. A username + password check, a fingerprint, a signed token. The output of authentication is an identity: "this request is from user 14, Diego Alvarez."
- **Authorization (AuthZ)** — *"What are you allowed to do?"* Given a known identity, deciding whether this specific action on this specific resource is permitted. The output is a yes/no: "user 14 may `GET /orders?region=1` but may not `GET /orders?region=2`."

A system can authenticate perfectly and still be wide open — if every logged-in user can do everything, you've built a guest book, not access control. Both layers are required, and they're built differently: authentication happens once, at login, and produces a token; authorization happens on *every* request, checking that token's identity against the specific thing being requested.

## Never store a plaintext password — hashing, not encryption

A password column should never contain the password. Not encrypted, not obfuscated — **hashed**, with a specific kind of hash function designed for this one job.

**Why not encryption?** Encryption is reversible by design — if you can decrypt it, so can anyone who steals your encryption key, which usually sits right next to the encrypted data. You never need to recover a user's original password; you only need to check whether a login attempt *matches* it. That's a one-way problem, and hashing is the one-way tool.

**Why not a general-purpose hash like SHA-256?** SHA-256 is *fast* — billions of hashes per second on commodity hardware. Fast is exactly wrong for passwords: if a password database leaks, an attacker with a GPU can try billions of guesses per second against a fast hash. Password-hashing algorithms (**bcrypt**, **scrypt**, **argon2**) are deliberately slow and tunable, so that same attack takes years instead of hours, while a legitimate login (one hash, once) is imperceptibly slower for the real user.

```python
import bcrypt

def hash_password(plain: str) -> str:
    # bcrypt generates its own random salt and embeds it in the output —
    # you never manage salts yourself.
    return bcrypt.hashpw(plain.encode(), bcrypt.gensalt()).decode()

def verify_password(plain: str, stored_hash: str) -> bool:
    return bcrypt.checkpw(plain.encode(), stored_hash.encode())

# seeding a Crunch Cycles user — never write the plaintext to a .sql file
pw_hash = hash_password("Tr41lheadB!kes2024")
print(pw_hash)
# $2b$12$KIXQ...   <- this, and only this, goes in app_users.password_hash
```

Every stored hash looks different even for the same password, because of the random salt baked in — that's what stops an attacker from precomputing "the hash of `password123`" once and matching it against every user in every leaked database at once (a **rainbow table** attack).

## Seeding real users with roles

Crunch Cycles needs five roles this week, matched to the org chart from Week 4's `employees` table:

| Role | Who | Can do |
|---|---|---|
| `sales_rep` | Diego, Sarah, Tom, Elena, Kenji, Priya, Lucas | See and manage orders/customers **in their own region only** |
| `sales_manager` | Maria (Sales Director) | See and manage orders/customers in **all** regions |
| `finance` | A new Finance hire | Read-only access to orders and revenue figures, across all regions, no ability to edit |
| `support` | A new Support hire | Read customer contact info to answer tickets; cannot see order financials |
| `admin` | You, running the system | Full access, including managing `app_users` itself |

```python
# seed_users.py — run once against crunchcycles
import bcrypt, os
from sqlalchemy import create_engine, text

engine = create_engine(os.environ["DATABASE_URL"])

users = [
    ("mchen",  "maria.chen@crunchcycles.io",  "SalesLead#24",  "sales_manager", 1),
    ("dalvarez","diego.alvarez@crunchcycles.io","Diego#2024",   "sales_rep",     2),
    ("skim",   "sarah.kim@crunchcycles.io",    "SarahK!99",     "sales_rep",     3),
    ("finance1","priya.iyer@crunchcycles.io",  "FinOps#24",     "finance",       None),
    ("support1","noah.reyes@crunchcycles.io",  "Support#24",    "support",       None),
    ("admin",  "admin@crunchcycles.io",        "ChangeMeNow!1", "admin",         None),
]

with engine.begin() as conn:
    for username, email, plain, role, emp_id in users:
        h = bcrypt.hashpw(plain.encode(), bcrypt.gensalt()).decode()
        conn.execute(text("""
            INSERT INTO app_users (username, email, password_hash, role, employee_id)
            VALUES (:u, :e, :h, :r, :emp)
            ON CONFLICT (username) DO NOTHING
        """), {"u": username, "e": email, "h": h, "r": role, "emp": emp_id})

print("seeded", len(users), "users")
```

Notice `admin`'s password is deliberately a placeholder you're meant to rotate immediately — the same pattern real systems use for a first-run superuser account, and the same pattern that causes breaches when nobody rotates it. Flag that instinct now; you'll act on it for real in Exercise 1.

## Building the login endpoint and issuing a session token

Login exchanges a username/password for a **JWT (JSON Web Token)** — a signed, tamper-evident token carrying the user's identity and role, which the client sends on every subsequent request instead of re-sending the password.

```python
# auth.py
import os, jwt, bcrypt
from datetime import datetime, timedelta, timezone
from flask import Blueprint, request, jsonify, abort
from sqlalchemy import create_engine, text

SECRET_KEY = os.environ["JWT_SECRET_KEY"]   # long, random, from .env — never hardcoded
engine = create_engine(os.environ["DATABASE_URL"])
auth_bp = Blueprint("auth", __name__)

@auth_bp.post("/api/v1/login")
def login():
    body = request.get_json(force=True)
    username, password = body.get("username"), body.get("password")
    if not username or not password:
        abort(400, description="username and password are required")

    with engine.connect() as conn:
        user = conn.execute(
            text("SELECT * FROM app_users WHERE username = :u AND is_active = TRUE"),
            {"u": username},
        ).mappings().first()

    # Same generic error whether the username doesn't exist or the password is wrong —
    # never reveal which one failed. That distinction is a gift to an attacker.
    if user is None or not bcrypt.checkpw(password.encode(), user["password_hash"].encode()):
        abort(401, description="invalid username or password")

    payload = {
        "sub": user["user_id"],
        "role": user["role"],
        "employee_id": user["employee_id"],
        "exp": datetime.now(timezone.utc) + timedelta(hours=8),
    }
    token = jwt.encode(payload, SECRET_KEY, algorithm="HS256")

    with engine.begin() as conn:
        conn.execute(
            text("UPDATE app_users SET last_login_at = now() WHERE user_id = :id"),
            {"id": user["user_id"]},
        )

    return jsonify({"token": token, "role": user["role"]}), 200
```

Two details worth internalizing because both are common real breaches:

1. **The same error for "no such user" and "wrong password."** A distinguishable error (`404 user not found` vs `401 wrong password`) lets an attacker enumerate valid usernames one guess at a time — an *account enumeration* vulnerability. One generic `401` closes it.
2. **The token expires (`exp`).** An 8-hour session token limits how long a stolen token is useful. Compare that to Week 7's API key, which worked forever until someone manually noticed and revoked it.

## Enforcing roles in the API — RBAC at the application layer

With login working, every protected route checks the token and the role it carries:

```python
# rbac.py
import os, jwt
from functools import wraps
from flask import request, abort, g

SECRET_KEY = os.environ["JWT_SECRET_KEY"]

def require_auth(roles=None):
    """Decorator: require a valid JWT, and optionally restrict to specific roles."""
    def decorator(fn):
        @wraps(fn)
        def wrapper(*args, **kwargs):
            header = request.headers.get("Authorization", "")
            token = header.removeprefix("Bearer ").strip()
            if not token:
                abort(401, description="missing bearer token")
            try:
                payload = jwt.decode(token, SECRET_KEY, algorithms=["HS256"])
            except jwt.ExpiredSignatureError:
                abort(401, description="token expired, please log in again")
            except jwt.InvalidTokenError:
                abort(401, description="invalid token")

            if roles is not None and payload["role"] not in roles:
                abort(403, description=f"role '{payload['role']}' cannot access this resource")

            g.user_id = payload["sub"]
            g.role = payload["role"]
            g.employee_id = payload["employee_id"]
            return fn(*args, **kwargs)
        return wrapper
    return decorator
```

```python
# app.py — protected routes
@app.get("/api/v1/orders")
@require_auth(roles=["sales_rep", "sales_manager", "finance", "admin"])
def list_orders():
    ...

@app.delete("/api/v1/app-users/<int:user_id>")
@require_auth(roles=["admin"])
def deactivate_user(user_id):
    ...
```

This is **RBAC (role-based access control)**: permissions attach to a *role*, not to an individual user, so adding a new sales rep is "assign the `sales_rep` role" — one line — not "figure out which of forty permission flags to copy from another user." It also gives you exactly one place to audit: read the `roles=[...]` list on every route and you know the entire access policy of the API.

## Least privilege — the principle underneath every decision above

**Least privilege**: every identity — human or service — gets the minimum access it needs to do its job, no more. It sounds obvious and is routinely violated because "just give them admin, it's easier" is faster on day one. The cost shows up later: a phished support account with admin rights can leak the whole customer table, when it only ever needed read access to names and emails. Every role table above is least privilege applied concretely — `support` cannot see revenue because support's job never requires it, not because anyone distrusts the support team personally.

This applies to service accounts too, not just people. Week 7's Flask app connects to Postgres — with what privileges? If that database user can `DROP TABLE`, then any SQL-injection bug or leaked `DATABASE_URL` becomes a catastrophe instead of a bounded incident. A dedicated `crunchcycles_api` Postgres role with only the grants the API actually needs is least privilege for machines.

## Row-level security — access control the database enforces itself

RBAC in Flask decides *which endpoints* a role can call. It does **not**, by itself, stop a bug from leaking rows — if `list_orders()` forgets to filter by region, every sales rep sees every region's orders regardless of the role check, because the role check only gated the endpoint, not the rows it returned. **Row-level security (RLS)**, a PostgreSQL feature, closes that gap by enforcing row visibility *inside the database*, so it holds even when application code has a bug.

```sql
-- 1. Turn RLS on for the table.
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;

-- 2. Create a Postgres role per application role, and have the API connect as one of them.
CREATE ROLE sales_rep_role NOLOGIN;
CREATE ROLE sales_manager_role NOLOGIN;
CREATE ROLE finance_role NOLOGIN;

GRANT SELECT ON orders, order_items, customers TO sales_rep_role, sales_manager_role, finance_role;
GRANT INSERT, UPDATE ON orders TO sales_rep_role, sales_manager_role;   -- finance is read-only

-- 3. A policy: sales reps only see orders for customers in their own region.
--    current_setting('app.current_region') is set per-connection by the API after login (below).
CREATE POLICY sales_rep_region_only ON orders
    FOR SELECT
    TO sales_rep_role
    USING (
        customer_id IN (
            SELECT customer_id FROM customers
            WHERE region_id = current_setting('app.current_region')::INTEGER
        )
    );

-- 4. Sales managers and finance see everything — a permissive policy with no filter.
CREATE POLICY manager_and_finance_see_all ON orders
    FOR SELECT
    TO sales_manager_role, finance_role
    USING (TRUE);
```

The API sets the session variable right after verifying the JWT, before running any query for that request:

```python
def set_rls_context(conn, role: str, region_id: int | None):
    pg_role = {"sales_rep": "sales_rep_role", "sales_manager": "sales_manager_role",
               "finance": "finance_role"}.get(role, "admin")
    conn.execute(text(f"SET ROLE {pg_role}"))
    if region_id is not None:
        conn.execute(text("SET app.current_region = :r"), {"r": region_id})
```

Now, even if a future engineer writes `SELECT * FROM orders` with no `WHERE` clause at all, a sales rep's database connection physically cannot see another region's rows — the policy is evaluated by Postgres itself on every query, regardless of what SQL the application sent. This is defense in depth: RBAC stops the wrong role from calling an endpoint; RLS stops the wrong rows from coming back even if RBAC or application logic has a bug.

```mermaid
flowchart TD
  A["Client sends username and password"] --> B["AuthN - check bcrypt hash"]
  B --> C["API issues JWT with identity and role"]
  C --> D["Client sends Bearer token on request"]
  D --> E["RBAC require auth checks role"]
  E -->|role denied| F["403 Forbidden"]
  E -->|role allowed| G["SET ROLE and app.current_region in Postgres"]
  G --> H["RLS policy filters rows by region"]
  H --> I["Only permitted rows returned"]
```
*Two independent layers of defense: RBAC gates the endpoint, RLS gates the rows.*

## Secret management — the thing that leaks the whole system if you get it wrong

Every credential this lecture introduced — `JWT_SECRET_KEY`, `DATABASE_URL` with its password, the API keys from Week 7 — is a **secret**: a value that, if exposed, lets someone impersonate your system or read your data directly. Three rules, non-negotiable:

1. **Never commit a secret to Git.** Not even once, not even in a "temporary" test file — Git remembers every commit forever, and a `git log -p` or a public GitHub search finds it years later even after you delete it in a later commit. Keep secrets in a `.env` file, and put `.env` in `.gitignore` **before** the file exists.

```bash
# .gitignore
.env
*.pem
*.key
```

```bash
# .env — loaded by python-dotenv, never committed
DATABASE_URL=postgresql://crunchcycles_api:S3cur3Pass@localhost/crunchcycles
JWT_SECRET_KEY=a1f9c2e8b7d4...   # generate with: python3 -c "import secrets; print(secrets.token_hex(32))"
```

```python
# app.py, top of file
from dotenv import load_dotenv
load_dotenv()   # reads .env into os.environ before anything else runs
```

2. **Rotate a secret the moment it might be exposed** — a leaked `.env` in a support ticket, a key pasted into a public Slack channel, an ex-employee's offboarding. Rotation means: generate a new value, deploy it, confirm the new value works, *then* invalidate the old one. Doing it in the wrong order causes an outage; skipping it entirely leaves a working credential in the hands of someone who shouldn't have it, indefinitely.

3. **Different secrets per environment.** Your local `.env`, a staging deployment, and Week 8's production deployment should never share a database password or JWT secret. If your laptop is compromised, that blast radius should stop at your laptop's own database, not extend to production.

For a solo or small-team project, `.env` + `.gitignore` is a legitimate, real answer. At larger scale, dedicated secrets managers (AWS Secrets Manager, HashiCorp Vault, Google Secret Manager) add rotation automation, access auditing (who read this secret, and when), and the ability to revoke one credential without redeploying every service that uses it — mentioned here so the term isn't a mystery when you meet it on a team; not required for this week's exercises.

## What's next

You can now prove who someone is and enforce what they can touch, at both the application and database layers. Lecture 2 turns to the data itself: once you know *who* can see a customer's email address, the next question is what you're legally and ethically obligated to do with that email address in the first place.
