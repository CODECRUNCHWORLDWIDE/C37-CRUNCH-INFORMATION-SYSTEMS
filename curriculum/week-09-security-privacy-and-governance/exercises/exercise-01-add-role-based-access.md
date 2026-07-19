# Exercise 1 — Add Role-Based Access to the API

Turn Lecture 1's login and RBAC pattern into a real, running Flask API in front of `crunchcycles`. By the end, `/api/v1/orders` requires a valid login and a permitted role — Week 7's single shared API key is retired.

**Estimated time:** 1.5 hours.

## Setup

```bash
python3 -m venv .venv && source .venv/bin/activate
pip install flask bcrypt pyjwt sqlalchemy psycopg2-binary python-dotenv
echo ".env" >> .gitignore
```

Create `.env` (never committed):

```bash
DATABASE_URL=postgresql://localhost/crunchcycles
JWT_SECRET_KEY=REPLACE_ME_WITH_python3_-c_"import secrets; print(secrets.token_hex(32))"
```

Run the lecture's `seed_users.py` to populate `app_users` with at least: one `sales_rep`, one `sales_manager`, one `finance`, and one `admin` account.

## Tasks

1. **Build `auth.py`** with a `POST /api/v1/login` endpoint that: looks up the user by username, verifies the password with `bcrypt.checkpw`, and — on success — returns a signed JWT containing `sub` (user id), `role`, `employee_id`, and an 8-hour `exp`. On failure, return a **generic** `401` regardless of whether the username or the password was wrong.

2. **Build `rbac.py`** with a `require_auth(roles=None)` decorator (Lecture 1's version is a complete working starting point) that validates the `Authorization: Bearer <token>` header and rejects missing, expired, or invalid tokens with `401`, and wrong-role tokens with `403`.

3. **Protect three routes** in `app.py`:
   - `GET /api/v1/orders` — any of `sales_rep`, `sales_manager`, `finance`, `admin`.
   - `POST /api/v1/orders` — only `sales_rep`, `sales_manager`, `admin` (finance is read-only).
   - `GET /api/v1/app-users` — only `admin`.

4. **Filter by role inside `list_orders()`.** A `sales_rep` should only receive orders for customers in their own region (join `orders → customers` on `region_id`, using `employees.region_id` for the logged-in rep via `g.employee_id`). A `sales_manager`, `finance`, or `admin` sees all regions. Do this in application code for this exercise — Exercise 3's mini-project counterpart adds the database-enforced version with row-level security.

5. **Write a rotation drill.** In a short `ROTATION.md`, write the exact commands you'd run to rotate `JWT_SECRET_KEY` if it leaked — including what happens to tokens issued under the old key (they should become invalid immediately; state that explicitly and why it's the correct trade-off).

## Expected result

```bash
# Login as a sales rep
curl -s -X POST http://localhost:5000/api/v1/login \
  -H "Content-Type: application/json" \
  -d '{"username":"dalvarez","password":"Diego#2024"}'
# → {"token": "eyJ...", "role": "sales_rep"}

TOKEN="eyJ..."   # paste the token above

# Diego (region 1) sees only region-1 customers' orders
curl -s http://localhost:5000/api/v1/orders -H "Authorization: Bearer $TOKEN"

# No token at all → 401
curl -i http://localhost:5000/api/v1/orders
# HTTP/1.1 401 UNAUTHORIZED

# Diego tries the admin-only route → 403
curl -i http://localhost:5000/api/v1/app-users -H "Authorization: Bearer $TOKEN"
# HTTP/1.1 403 FORBIDDEN
```

## Done when

- [ ] Login works with a correct username/password and fails identically (generic `401`) for a wrong username *or* a wrong password.
- [ ] All three protected routes reject a missing/expired/invalid token with `401`.
- [ ] A `sales_rep` gets `403` on the admin-only route; an `admin` succeeds on all three.
- [ ] `list_orders()` returns only the calling rep's region for `sales_rep`, and everything for `sales_manager`/`finance`/`admin`.
- [ ] `.env` is in `.gitignore` and was never staged (`git log --all --full-history -- .env` returns nothing).
- [ ] `ROTATION.md` states the exact rotation steps and confirms old tokens become invalid.
