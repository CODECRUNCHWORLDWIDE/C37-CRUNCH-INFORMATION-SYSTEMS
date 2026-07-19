# Week 8 — Cloud Architecture

> **Goal:** by Sunday, Crunch Cycles' Week 7 API and database are running on a real, free cloud tier — reachable by a real URL, backed by a real managed PostgreSQL instance — and you can defend, on paper, why it's built the way it is: which cloud service model you chose and why, what would survive one data center having a bad day, how it would scale to 10x traffic, and what it would cost to get there.

Welcome back to **C37 · Crunch Information Systems**. For seven weeks Crunch Cycles has lived somewhere between your laptop and a course exercise — correct, queryable, even API-accessible over `localhost`, but reachable by exactly one person: you. This week that changes. You'll decide, deliberately rather than by reflex, how much of the technology stack to rent versus run yourself (IaaS, PaaS, or SaaS); learn the small set of building blocks — compute, managed databases, object storage, networking — that every cloud architecture on Earth is assembled from; and then reason about the two things that turn a deployed system into a *trustworthy* one: whether it survives failure, and whether anyone can afford to keep it running. By Saturday, Crunch Cycles will be live on the internet, with a diagram, a scaling plan, and a real cost estimate to show for it.

This week also introduces the last new discipline of the "systems" half of this course. Weeks 1–7 were about modeling, storing, processing, and connecting data. Week 8 is about *where it runs* — a question every one of those weeks quietly assumed away. Weeks 9–12 build directly on top of a deployed system: you can't secure, monitor, or govern something that only exists on your laptop.

## Learning objectives

By the end of this week, you will be able to:

- **Distinguish** IaaS, PaaS, and SaaS using the shared-responsibility model, and choose deliberately among them for a given system and team instead of defaulting to whatever's familiar.
- **Explain** the four building blocks every cloud architecture is assembled from — compute, managed databases, object storage, and networking — and pick sensible options for each.
- **Reason** about availability in concrete downtime-per-year terms, design redundancy that eliminates a single point of failure, and distinguish horizontal from vertical scaling — including why databases scale differently than stateless app tiers.
- **Deploy** a working system — an API plus a PostgreSQL database — to a free cloud tier, verified end to end from outside your own network.
- **Estimate and control** cloud cost: read a bill by line item, name the common runaway-cost traps, and project cost at 10x growth using code, never a spreadsheet.

## Prerequisites

- Weeks 1–7 of this course, especially Week 7's Flask REST API and the `crunchcycles` PostgreSQL schema (`regions`, `employees`, `customers`, `products`, `orders`, `order_items`, plus Week 6/7's `suppliers`, `purchase_orders`, `crm_opportunities`, `fx_rates` if you built them). If your Week 7 app isn't in a Git repository yet, put it in one before Monday — this week's deployment steps assume it.
- Python 3.10+ with `pip` working, and comfort running the Week 7 Flask app locally (`python3 app.py`) and querying `crunchcycles` with `psql`.
- A free account (created this week, no cost) on one cloud PaaS platform (Render, Railway, or Fly.io) and one managed-Postgres provider — see [`resources.md`](./resources.md) for links. No credit card is required for the free tiers used this week.
- Comfort with `git push`, environment variables, and reading a terminal error message closely — deployment surfaces small configuration mistakes fast, and debugging them is half the point of the week.

## Setup

No new database schema this week — you're taking the schema and data you already have from Weeks 3–7 and moving it to a managed cloud instance (Exercise 2 walks through this precisely). If you don't have a working local copy anymore, the full seed lives in [Week 4's README](../week-04-querying-data-with-sql-and-python/README.md) and Week 6's enterprise extensions live in [Week 6's README](../week-06-enterprise-systems-erp-crm/README.md) — load those first, then start this week's exercises.

Before Monday, confirm your local environment is ready:

```bash
python3 -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install flask gunicorn requests pandas sqlalchemy psycopg2-binary python-dotenv
```

- **`gunicorn`** is new this week — it's the production WSGI server you'll actually deploy with; Flask's built-in dev server explicitly warns against production use.
- Everything else carries over from Week 7's setup.

## Weekly schedule

Adds up to the course's full-time target of **~28 hours**.

| Day | Focus | Lectures | Exercises | Challenges | Quiz/Read | Homework | Mini-Project | Daily Total |
|-----------|--------------------------------------------|---------:|----------:|-----------:|----------:|---------:|-------------:|------------:|
| Monday | IaaS/PaaS/SaaS; choosing a model | 2h | 1h | 0h | 0.5h | 1h | 0h | 4.5h |
| Tuesday | Cloud building blocks; architecture diagram | 2h | 1h | 0h | 0.5h | 1h | 0h | 4.5h |
| Wednesday | Deploy the database; availability & scaling | 2h | 1h | 0h | 0.5h | 1h | 0h | 4.5h |
| Thursday | Cost reading & estimation; deploy the app | 0h | 1h | 1.5h | 0.5h | 1h | 1h | 5h |
| Friday | Availability design + cloud-bill challenges | 0h | 0h | 1.5h | 0.5h | 1h | 1.5h | 4.5h |
| Saturday | Mini-project (deploy + diagram + cost) | 0h | 0h | 0h | 0h | 0h | 2.5h | 2.5h |
| Sunday | Quiz + review | 0h | 0h | 0h | 1h | 0h | 0h | 1h |
| **Total** | | **6h** | **4h** | **3h** | **3.5h** | **5h** | **5h** | **~26.5h** |

## How to navigate this week

Work top to bottom. Each piece assumes the ones above it.

| # | File | What's inside | ~Time |
|--:|------|---------------|------:|
| 1 | [lecture-notes/01-iaas-paas-saas.md](./lecture-notes/01-iaas-paas-saas.md) | The cloud service models, the shared-responsibility line, and choosing deliberately for Crunch Cycles | 2h |
| 2 | [lecture-notes/02-cloud-building-blocks.md](./lecture-notes/02-cloud-building-blocks.md) | Compute, managed databases, object storage, networking — and a Crunch Cycles architecture diagram | 2h |
| 3 | [lecture-notes/03-availability-scaling-and-cost.md](./lecture-notes/03-availability-scaling-and-cost.md) | Redundancy, horizontal vs. vertical scaling, and reading (and modeling) a cloud bill | 2h |
| 4 | [exercises/exercise-01-pick-a-cloud-model.md](./exercises/exercise-01-pick-a-cloud-model.md) | Classify 10 real systems as IaaS/PaaS/SaaS, justified | 1h |
| 5 | [exercises/exercise-02-deploy-a-database.md](./exercises/exercise-02-deploy-a-database.md) | Deploy a real managed PostgreSQL instance and load Crunch Cycles into it | 1h |
| 6 | [exercises/exercise-03-estimate-monthly-cost.md](./exercises/exercise-03-estimate-monthly-cost.md) | Build a Python cost model for the deployment, current and at 10x | 1h |
| 7 | [challenges/challenge-01-design-for-availability.md](./challenges/challenge-01-design-for-availability.md) | Redesign Crunch Cycles to survive one availability zone failing entirely | 1.5h |
| 8 | [challenges/challenge-02-cut-a-cloud-bill.md](./challenges/challenge-02-cut-a-cloud-bill.md) | Find and cut 50%+ of a wasteful cloud bill using SQL analysis | 1.5h |
| 9 | [mini-project/README.md](./mini-project/README.md) | Deploy the real system, deliver a diagram, a scaling plan, and a 10x cost estimate | 2.5h |
| 10 | [homework.md](./homework.md) | Extra practice, spaced across the week | 5h |
| 11 | [quiz.md](./quiz.md) | 14 self-check questions + answer key | 1h |
| 12 | [resources.md](./resources.md) | Official docs, provider pricing pages, diagramming tools, glossary | — |

## By the end of this week you can…

- Look at any system and place it deliberately on the IaaS/PaaS/SaaS spectrum, defending the choice with the shared-responsibility model instead of habit.
- Draw an accurate architecture diagram of a real deployed system using the four cloud building blocks, not a generic template.
- Walk through, precisely, what happens to a system when one data center fails — and design redundancy that makes the honest answer boring.
- Explain why an app tier scales horizontally with ease while a relational database does not, and choose the right lever for each.
- Read a cloud bill line by line, name the traps that inflate it without adding value, and project cost at 10x growth in code you can actually re-run.

## Up next

[Week 9 — Security, Privacy & Governance](../week-09-security-privacy-and-governance/) — now that Crunch Cycles is live on the internet and reachable by anyone with the URL, next week you make sure only the *right* anyones can do anything with it.

---

*Part of the Code Crunch Worldwide open curriculum · GPL-3.0 · If you find errors, please open an issue or PR.*
