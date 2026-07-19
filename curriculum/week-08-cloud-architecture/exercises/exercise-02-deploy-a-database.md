# Exercise 2 — Deploy a Database

**Goal:** Move `crunchcycles` off your laptop and onto a real managed PostgreSQL instance in the cloud — the first concrete step toward deploying the whole system in the mini-project.

**Estimated time:** 1 hour.

## Setup

Pick **one** free managed-Postgres provider. Any of these work; the steps below use **Render Postgres** as the walkthrough, with the equivalent step noted for the alternatives:

- **Render** (<https://render.com>) — a free Postgres instance, expires after 30 days on the free plan (fine for this course; note the expiry so it doesn't surprise you).
- **Neon** (<https://neon.tech>) — free tier with no time-boxed expiry, serverless Postgres, generous for a course project.
- **Supabase** (<https://supabase.com>) — free tier, Postgres plus extras you won't need this week.
- **Railway** (<https://railway.app>) — free trial credit, Postgres as a one-click service.

You do **not** need a credit card for any of these at the free tier described. If a provider asks for one, pick a different provider from the list.

## Tasks

1. **Create an account** on your chosen provider (free tier).

2. **Provision a managed PostgreSQL instance.** On Render: New → PostgreSQL → name it `crunchcycles`, pick the free plan, pick a region close to you (note the region — you'll need it to match your app's region in the mini-project, per Lecture 3's egress warning). On Neon/Supabase/Railway: the equivalent "new Postgres project/database" flow.

3. **Get the connection string.** It looks like:
   ```
   postgresql://cc_admin:XXXXXXXX@dpg-xxxxxxxxxxxx-a.oregon-postgres.render.com/crunchcycles
   ```
   Copy it somewhere safe — you will **not** commit it to Git.

4. **Connect with `psql`** from your terminal, using the connection string directly:
   ```bash
   psql "postgresql://cc_admin:XXXXXXXX@dpg-xxxxxxxxxxxx-a.oregon-postgres.render.com/crunchcycles"
   ```
   If this connects and drops you into a `crunchcycles=>` prompt, the network path and credentials both work.

5. **Load the schema.** Paste (or `\i schema.sql`) the `CREATE TABLE` statements for at least the core Crunch Cycles tables from Weeks 3–4: `regions`, `employees`, `customers`, `products`, `orders`, `order_items`. (Reuse your existing `schema.sql` from earlier weeks if you kept one; otherwise reconstruct it from the Week 3/4 READMEs.)

6. **Load a small amount of seed data** — either the full Week 3/4 seed, or a trimmed version (at minimum: 3 regions, 5 employees, 5 customers, 5 products, 10 orders, 15 order items) — enough to run real queries against.

7. **Verify with a sanity query:**
   ```sql
   SELECT COUNT(*) FROM orders;
   ```

8. **Store the connection string safely.** Create a `.env` file (do **not** commit it):
   ```bash
   # .env  (add this filename to .gitignore before your first commit)
   DATABASE_URL=postgresql://cc_admin:XXXXXXXX@dpg-xxxxxxxxxxxx-a.oregon-postgres.render.com/crunchcycles
   ```
   And commit a `.env.example` with placeholder values instead, so the shape is documented without leaking the secret:
   ```bash
   # .env.example  (safe to commit)
   DATABASE_URL=postgresql://username:password@host/dbname
   ```

9. **Confirm from Python**, using the same `psycopg2`/SQLAlchemy pattern from Week 4, pointed at the remote instance instead of `localhost`:
   ```python
   import os
   from sqlalchemy import create_engine, text

   engine = create_engine(os.environ["DATABASE_URL"])
   with engine.connect() as conn:
       result = conn.execute(text("SELECT COUNT(*) FROM orders"))
       print("orders:", result.scalar())
   ```

## Expected result

- `psql` connects to the remote instance without error.
- `SELECT COUNT(*) FROM orders;` returns a nonzero count matching however much seed data you loaded.
- The Python script prints the same count, confirming the app-side connection path works too — this is exactly the connection your deployed Flask app will use in the mini-project.
- `git status` shows `.env` is **not** tracked, and `.env.example` **is**.

## Done when…

- [ ] You have a live managed PostgreSQL instance with at least the six core Crunch Cycles tables and seed data loaded.
- [ ] You connected successfully both via `psql` and via a Python script.
- [ ] `.env` is in `.gitignore`; `.env.example` is committed with placeholder values.
- [ ] You wrote one sentence in `notes.md` on which provider you picked and why (free-tier limits, region, familiarity — any honest reason counts).

## Stretch

- Check whether your provider's free tier includes automated backups or point-in-time recovery, and note what you found — most free tiers trim this first, which is worth knowing before you'd trust it with anything beyond a course project.
- If your provider supports it, try connecting from a second location (a friend's machine, a cloud shell) to confirm the database really is reachable over the public internet and not just from your one machine — then think about whether that's actually what you want in production (Lecture 2's private-networking point).

## Submission

Commit `notes.md` and `.env.example` (never the real `.env`) to your portfolio under `c37-week-08/exercise-02/`.
