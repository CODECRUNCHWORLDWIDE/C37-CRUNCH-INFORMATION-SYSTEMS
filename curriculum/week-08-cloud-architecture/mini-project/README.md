# Mini-Project — Deploy Crunch Cycles to the Cloud

> Take the Week 7 Flask REST API and its PostgreSQL database off your laptop and onto a real, free cloud tier — reachable at a real URL, backed by a real managed database — then deliver the three artifacts a stakeholder would actually ask for: an architecture diagram, a scaling plan, and a cost estimate for 10x growth.

**Estimated time:** 2.5–3 hours, best done Saturday after the exercises and challenges.

This is the week's capstone, and it's different from every prior mini-project in one important way: it doesn't end at your keyboard. A deployed system is a claim about the world — "this is reachable, right now, by anyone with the URL" — and either it is or it isn't. You'll find out this week whether Crunch Cycles' Week 7 API and Week 3–7 database actually work end to end outside the safety of `localhost`, which is exactly the gap between "it works on my machine" and "it's a real information system."

---

## Deliverable

A directory in your portfolio `c37-week-08/mini-project/` containing:

1. **A live deployment** — a real URL where the Crunch Cycles API responds (e.g., `https://crunch-cycles-api.onrender.com/health`), and a real managed PostgreSQL instance it's connected to. If your provider's free tier later expires or sleeps the service, that's fine — capture proof it worked (see `deployment-proof.md` below).
2. `architecture-diagram.md` — a Mermaid or ASCII diagram of what you actually deployed (not the lecture's generic diagram — yours, with your provider's real component names), plus 3–5 sentences walking through the request path from a client to a database row and back.
3. `scaling-plan.md` — a written plan for how this specific deployment would need to change to handle **10x today's traffic**, using Lecture 3's horizontal/vertical framework: what scales first, what's the bottleneck, and at what point (roughly) would you need to change *architecture*, not just turn a dial.
4. `cost-estimate.py` — a Python script (not a spreadsheet — see the data rule below) that estimates current monthly cost and 10x-growth monthly cost, broken down by line item, building on Exercise 3.
5. `deployment-proof.md` — evidence the deployment worked: the live URL, a `curl` command and its output hitting a real endpoint, and a screenshot or terminal output of a query run against the live database showing real Crunch Cycles data.

**Data rule reminder:** the cost estimate is Python, never a spreadsheet — this course never uses Excel as a data or modeling tool, and a cost model you can re-run with a changed parameter is strictly more useful than a static grid of cells.

---

## Steps

Work roughly in this order — each step depends on the one before it.

### 1. Prepare the code for deployment (30 min)

- If you don't already have the Week 7 Flask app in a Git repository, put it in one now. A cloud PaaS platform deploys **from a Git repo**, not from a zip file you upload by hand.
- Add a `requirements.txt` if you don't have one (`pip freeze > requirements.txt`, then trim it to what you actually import).
- Add the `/health` endpoint from Lecture 2, Section 4, if your Week 7 app doesn't already have one — the platform's load balancer needs something to poll.
- Make sure the app reads its database connection from an environment variable (`os.environ["DATABASE_URL"]`), never a hard-coded `localhost` string — this is what lets the exact same code run locally and in the cloud.

### 2. Deploy the database (30 min)

- Reuse your Exercise 2 managed PostgreSQL instance, or provision a fresh one on the same provider you'll use for the app.
- **Put it in the same region you'll deploy the app to** — Lecture 3's cross-region-egress warning applies to your own deployment now, not just the challenge's fictional bill.
- Confirm the schema and seed data are loaded (reuse Exercise 2's work if it's the same instance).

### 3. Deploy the app (45 min)

- On your chosen PaaS platform (Render, Railway, Fly.io — pick one), create a new web service pointed at your Git repo.
- Set the `DATABASE_URL` environment variable to your managed database's connection string, using the platform's environment-variable UI — **never** commit it into the repo.
- Set the start command to run via a production WSGI server, not Flask's dev server:
  ```
  gunicorn --bind 0.0.0.0:$PORT app:app
  ```
  (Most platforms inject `$PORT` for you — bind to it, don't hard-code a port number.)
- Deploy, and watch the build/deploy logs for errors. The most common first-deploy failures: a missing dependency in `requirements.txt`, a hard-coded `localhost` connection string, or binding to the wrong port.

### 4. Verify it actually works (20 min)

- Hit `/health` from your own browser or `curl`, from a network that isn't the machine you developed on if you can (a phone on cellular data is a good, easy check) — this confirms it's really public, not just reachable from your home network by coincidence.
- Hit at least one real data endpoint (e.g., a route that lists orders or products) and confirm it returns real rows from the deployed database, not an error.
- Capture this in `deployment-proof.md`.

### 5. Diagram, scaling plan, cost estimate (60–75 min)

- Draw `architecture-diagram.md` from what you actually built, not from memory of the lecture diagram — name your real provider, your real region, your real instance types.
- Write `scaling-plan.md` reasoning through 10x traffic using Lecture 3's horizontal/vertical framework and the statelessness check from Challenge 1.
- Adapt Exercise 3's cost model into `cost-estimate.py`, using your actual provider's pricing for your actual instance choices.

---

## Requirements checklist

- [ ] The app is deployed and reachable at a real public URL.
- [ ] The database is a managed cloud instance, not a local Postgres tunneled through something.
- [ ] The app connects to the database via an environment variable, never a hard-coded connection string.
- [ ] `.env` (or equivalent secrets file) is **not** committed to Git; a `.env.example` with placeholders is.
- [ ] `architecture-diagram.md` reflects your real deployment, with a request-path walkthrough.
- [ ] `scaling-plan.md` names a specific first bottleneck and a specific architectural change threshold, not a vague "we'd scale it."
- [ ] `cost-estimate.py` runs and prints both current and 10x-growth totals, broken down by line item.
- [ ] `deployment-proof.md` shows a real `curl`/browser hit and a real database query result.

---

## Rubric

| Criterion | Weight | "Great" looks like |
|-----------|------:|--------------------|
| Working deployment | 30% | App and database are both real, cloud-hosted, and connected to each other |
| Architecture diagram accuracy | 15% | Diagram matches what was actually built, not a generic template |
| Scaling plan reasoning | 20% | Correctly identifies app tier as horizontally scalable, database as the harder case; names a concrete bottleneck |
| Cost estimate rigor | 20% | Real or realistically sourced pricing; 10x scenario reasons per-line-item, not a uniform multiply |
| Security hygiene | 10% | No secrets committed; `DATABASE_URL` via environment variable |
| Proof of work | 5% | `deployment-proof.md` shows real, verifiable evidence |

---

## Stretch goals

- Add a second app instance and confirm the platform load-balances between them — check your platform's dashboard for per-instance metrics to prove traffic is actually being split, not just configured to be.
- Provision a read replica of your database (if your provider's free tier allows it) and point one report-style query at it instead of the primary, demonstrating Lecture 3's read-scaling pattern for real.
- Set up a basic uptime monitor (many free options exist — UptimeRobot and similar) pinging your `/health` endpoint, and let it run for 24 hours; report what it observed.
- Extend `cost-estimate.py` to accept a growth multiplier as a command-line argument, so it isn't hard-coded to exactly 10x.

## Reflection (`notes.md`, ~200 words)

1. What broke the first time you tried to deploy, and what actually fixed it? (Something almost always breaks — a missing env var, a wrong port, a forgotten dependency. Naming it is more useful than pretending it went smoothly.)
2. Which of Lecture 3's cost traps did you personally have to think about or avoid while setting this up?
3. If Crunch Cycles' real traffic hit 10x tomorrow, which single change from your scaling plan would you make *first*, and why that one before the others?
4. What would you need to add to this deployment before you'd trust it with a paying customer's real order? (Foreshadows Week 9 — security, privacy, and governance.)

---

## Why this matters

Every system in this course, up through Week 7, was correct-but-unreachable — right answers that lived only on your laptop. This week it becomes a real, addressable thing on the internet, with a real bill attached to it and a real failure mode if a data center has a bad day. That gap — between "the code is correct" and "the system is running, reachable, resilient, and affordable" — is most of what the word "deployment" means in a real job, and it's the gap this course has been building toward since Week 1's information-systems diagram.

When done: push, then take the [quiz](../quiz.md) and continue to Week 9 — security, privacy, and governance.
