# Week 7 — Integration & APIs

> **Goal:** by Sunday you can design a REST API other teams can build against without asking you a single question, pull data from a real third-party API into your database with proper error and rate-limit handling, receive and process a webhook safely, and ship a small ETL job that moves data between two systems idempotently — meaning you can run it twice by accident and nothing breaks.

Welcome back to **C37 · Crunch Information Systems**. For six weeks you've built Crunch Cycles as a closed world: one schema, one database, queries and reports that never leave your laptop. Real information systems are never that isolated. Crunch Cycles needs to know today's EUR/USD rate to report revenue in one currency. It needs to tell a shipping partner when an order is ready. It needs to hear back from a payment processor the instant a charge succeeds — not five minutes later when someone happens to refresh a dashboard. None of that happens inside your database. It happens over the network, through APIs, in both directions.

This is the week the "systems" in "information systems" stops being one system. You'll design a REST API that exposes the Crunch Cycles database to the outside world, consume a real public API from the outside world into your database, receive webhooks — the reverse of calling an API, where someone else calls *you* — and wire it all together into an ETL job that is boring, predictable, and safe to re-run. Integration is unglamorous by design. When it's done well nobody notices; when it's done badly, it's 2 a.m. and someone's phone is ringing.

## Learning objectives

By the end of this week, you will be able to:

- **Explain** why integration, not features, is where enterprise systems succeed or fail — and give a concrete example of an integration failure and what it cost.
- **Design and document** a REST API with resources, HTTP verbs, status codes, versioning, and authentication — a spec another developer could implement without talking to you.
- **Consume** a third-party API with Python's `requests`, handling pagination, rate limits (429s, `Retry-After`, backoff), and transient errors gracefully.
- **Receive and process** webhooks — verify the sender, respond fast, avoid double-processing the same event, and act on it asynchronously.
- **Build an ETL/ELT job** that extracts data from one system, transforms it, and loads it into PostgreSQL — with logging, retries, and idempotency so re-running it is always safe.
- **Choose deliberately** between batch and streaming, point-to-point and hub-based integration, and explain the trade-off out loud.

## Prerequisites

- Weeks 1–6 of this course, or equivalent comfort with: normalized schemas, SQL joins/aggregation, moving data between SQL and pandas, and process/workflow thinking.
- Python 3.10+ with `pip` working, and the PostgreSQL 16+ `crunchcycles` database from Weeks 3–6 (schema reproduced below if you need it fresh).
- Comfort making an HTTP request and reading a status code — if `curl -i https://example.com` is unfamiliar, skim the "HTTP basics" link in [`resources.md`](./resources.md) first.
- No prior API or webhook experience required. That is what this week teaches.

## Setup

**1. Confirm your database.** If your `crunchcycles` Postgres database from Week 3/4 is still around, you're set. If not, recreate it:

```bash
createdb crunchcycles
psql crunchcycles -f setup/crunchcycles_schema.sql   # or paste the CREATE TABLE + INSERT block from Week 4's README
```

This week reuses `regions`, `employees`, `customers`, `products`, `orders`, and `order_items` unchanged, and adds one new table, `fx_rates`, which you'll create in Lecture 3.

**2. Install the Python side.**

```bash
python3 -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install flask requests pandas sqlalchemy psycopg2-binary python-dotenv
```

- **Flask** — the web framework you'll use to build the REST API and the webhook receiver. Small, explicit, no magic — good for *learning* what a web framework does before you reach for something heavier.
- **requests** — the HTTP client you'll use to call other people's APIs.
- **pandas / SQLAlchemy / psycopg2-binary** — same ETL toolkit from Week 4.

**3. Sanity-check Flask.**

```bash
python3 -c "import flask; print(flask.__version__)"
```

Any 2.x or 3.x version is fine.

## Weekly schedule

Adds up to the course's full-time target of **~28 hours**.

| Day | Focus | Lectures | Exercises | Challenges | Quiz/Read | Homework | Mini-Project | Daily Total |
|-----------|--------------------------------------------|---------:|----------:|-----------:|----------:|---------:|-------------:|------------:|
| Monday | Designing REST APIs: resources, verbs, codes | 2h | 1.5h | 0h | 0.5h | 1h | 0h | 5h |
| Tuesday | Auth, versioning; build the API skeleton | 0h | 1h | 0h | 0.5h | 1h | 0h | 2.5h |
| Wednesday | Consuming APIs: pagination, rate limits | 2h | 1.5h | 0h | 0.5h | 1h | 0h | 5h |
| Thursday | Webhooks: receive, verify, dedupe | 2h | 1.5h | 1h | 0.5h | 1h | 1h | 7h |
| Friday | ETL/ELT patterns; idempotent pipelines | 2h | 0h | 2h | 0.5h | 1h | 1.5h | 7h |
| Saturday | Mini-project (API + ETL integration) | 0h | 0h | 0h | 0h | 0h | 2.5h | 2.5h |
| Sunday | Quiz + review | 0h | 0h | 0h | 1h | 0h | 0h | 1h |
| **Total** | | **6h** | **5.5h** | **3h** | **3.5h** | **5h** | **5h** | **~28h** |

## How to navigate this week

Work top to bottom. Each piece assumes the ones above it.

| # | File | What's inside | ~Time |
|--:|------|---------------|------:|
| 1 | [lecture-notes/01-designing-rest-apis.md](./lecture-notes/01-designing-rest-apis.md) | Resources, verbs, status codes, versioning, auth — designing the Crunch Cycles API | 2h |
| 2 | [lecture-notes/02-consuming-apis-and-webhooks.md](./lecture-notes/02-consuming-apis-and-webhooks.md) | Calling third-party APIs, pagination, rate limits, retries; receiving webhooks safely | 2h |
| 3 | [lecture-notes/03-etl-and-integration-patterns.md](./lecture-notes/03-etl-and-integration-patterns.md) | Batch vs. streaming, point-to-point vs. hub, building an idempotent ETL job | 2h |
| 4 | [exercises/exercise-01-design-an-api.md](./exercises/exercise-01-design-an-api.md) | Design and document a small REST API | 1.5h |
| 5 | [exercises/exercise-02-consume-a-public-api.md](./exercises/exercise-02-consume-a-public-api.md) | Pull real data from a public API into SQL | 1.5h |
| 6 | [exercises/exercise-03-handle-a-webhook.md](./exercises/exercise-03-handle-a-webhook.md) | Build a Flask endpoint that processes an incoming webhook | 1.5h |
| 7 | [challenges/challenge-01-integrate-two-systems.md](./challenges/challenge-01-integrate-two-systems.md) | Design an integration between two real-world systems | 1.5h |
| 8 | [challenges/challenge-02-make-integration-resilient.md](./challenges/challenge-02-make-integration-resilient.md) | Add retries, backoff, and dedup to a fragile ETL job | 1.5h |
| 9 | [mini-project/README.md](./mini-project/README.md) | Expose a REST API over Postgres + build an ETL job from a public API | 2.5h |
| 10 | [homework.md](./homework.md) | Extra practice, spaced out | 5h |
| 11 | [quiz.md](./quiz.md) | 15 self-check questions + answer key | 1h |
| 12 | [resources.md](./resources.md) | Official docs, tools to install, API design references | — |

## By the end of this week you can…

- Look at a business process and sketch the REST resources and endpoints it needs, with correct verbs and status codes, before writing a line of code.
- Call a real third-party API defensively — handle pagination, respect rate limits, and recover from transient failures without human intervention.
- Build a webhook receiver that verifies its caller, responds fast, and never processes the same event twice.
- Write an ETL job you can re-run at 3 a.m. after a crash and trust it will leave the data exactly as correct as if it had run once.
- Explain, in a design review, why "just call the API again" is not a resilience strategy on its own.

## Up next

[Week 8 — Cloud Architecture](../week-08-cloud-architecture/) — now that data flows in and out of Crunch Cycles automatically, next week the whole system moves off your laptop and onto a real, scalable, cost-aware cloud deployment.

---

*Part of the Code Crunch Worldwide open curriculum · GPL-3.0 · If you find errors, please open an issue or PR.*
