# Week 12 — Capstone — Design an Intelligent System

> **Goal:** by the end of this week you can take a real organization, scope a system it actually needs, design and build the core of that system across every layer this course covered — data, process, integration, cloud, security, analytics, and AI — cost it honestly, phase its delivery, and defend every major decision in it against a skeptical, unscripted question.

Welcome to the final week of **C37 · Crunch Information Systems**. Eleven weeks ago you learned that an information system is people, process, data, and technology working together. Since then, one week at a time, you built every piece of a real system on the running **Crunch Cycles** example: a normalized schema, SQL and Python querying, an automated workflow, ERP/CRM-shaped enterprise data, a REST API, a cloud deployment, real access control, a BI warehouse, and an AI feature. Each of those weeks handed you a finished piece. This week hands you nothing finished — it hands you a real organization of your own choosing and asks you to build the whole thing yourself, then stand behind it.

This is not a review week. It's the week the other eleven were building toward: proof that you can do the whole job, not just each part of it, for a system nobody handed you pre-scoped. You'll pick a real organization (your employer, a family business, a nonprofit, or one of three ready-made fallbacks), scope it, architect it end to end, cost and phase its delivery, build its data foundation and a working slice of the layers above it, and present and defend the result — including, in the challenges, under deliberately hostile questioning and under a sudden budget or scale shock.

## Learning objectives

By the end of this week, you will be able to:

- **Analyze** a real organization and specify concrete, measurable requirements for an end-to-end information system, including an explicit, defensible boundary around what's in and out of scope.
- **Design** the data model, process automation, integrations, cloud architecture, and security together as one coherent system — able to trace one record through every layer without an unexplained gap.
- **Justify** build-vs-buy, tooling, and AI choices against real cost (modeled in Python, never a spreadsheet) and organizational fit, not just technical preference.
- **Deliver** working analytics and an AI feature grounded in the system's own real data, with a stated human-in-the-loop check for when the AI is wrong.
- **Present and defend** the full design and its trade-offs to a mixed audience, using a structured four-part defense for every contested decision, and adapt the whole plan under a changed budget or scale constraint.

## Prerequisites

- Weeks 1–11 of this course, or equivalent comfort with: requirements analysis, a normalized PostgreSQL schema, SQL + pandas querying, workflow automation in Python, ERP/CRM-style enterprise data modeling, a REST API, cloud deployment (PaaS/IaaS/SaaS trade-offs), RBAC + row-level security, a BI warehouse and dashboard, and one AI/ML decision-support feature.
- Python 3.10+ and PostgreSQL 16+ (SQLite fallback) working locally — this week's cost model, seed data, automation, and AI feature all run in Python/SQL, per this course's data rule: **no spreadsheet is ever used as a data store or cost model, even for a small organization's finance-shaped questions.**
- A real organization you can scope this week — see [`exercises/exercise-01-scope-the-capstone.md`](./exercises/exercise-01-scope-the-capstone.md) for how to pick one, including three ready-made fallbacks if nothing comes to mind.

## Setup

No new seed database this week — you're building your **own** schema for your own chosen organization, from scratch, starting in Exercise 1. If you want a quick sanity check that your tooling still works before diving in:

```bash
python3 -c "import pandas, sklearn; print('python stack ok')"
psql --version   # or: sqlite3 --version
```

## Weekly schedule

Adds up to the course's full-time target of **~28 hours**, weighted toward the mini-project since it *is* the capstone deliverable.

| Day | Focus | Lectures | Exercises | Challenges | Quiz/Read | Homework | Mini-Project | Daily Total |
|-----------|-------------------------------------------|---------:|----------:|-----------:|----------:|---------:|-------------:|------------:|
| Monday | Composing the six-layer system | 2h | 1.5h | 0h | 0.5h | 1h | 0h | 5h |
| Tuesday | IT strategy, phasing, TCO, risk | 2h | 1.5h | 0h | 0.5h | 1h | 0h | 5h |
| Wednesday | Presenting & defending a design | 2h | 2h | 0h | 0.5h | 1h | 0h | 5.5h |
| Thursday | Stress-test the design | 0h | 0h | 1.5h | 0.5h | 1h | 2h | 5h |
| Friday | Re-architect under new constraints | 0h | 0h | 1.5h | 0.5h | 1h | 3h | 6h |
| Saturday | Mini-project build + package | 0h | 0h | 0h | 0h | 0h | 4h | 4h |
| Sunday | Quiz, dry-run defense, ship | 0h | 0h | 0h | 1h | 0h | 1h | 2h |
| **Total** | | **6h** | **5h** | **3h** | **3.5h** | **5h** | **10h** | **~32.5h** |

*(Slightly over the usual 28h — this is the capstone; budget extra time Saturday/Sunday if you're on the standard pace.)*

## How to navigate this week

Work top to bottom. Each piece assumes the ones above it — the exercises build your capstone piece by piece, the challenges stress-test it, and the mini-project is where it all becomes a working, defensible deliverable.

| # | File | What's inside | ~Time |
|--:|------|---------------|------:|
| 1 | [lecture-notes/01-putting-the-system-together.md](./lecture-notes/01-putting-the-system-together.md) | The six-layer stack, a full Crunch Cycles worked example, the four seams that break real systems, build-vs-buy by layer | 2h |
| 2 | [lecture-notes/02-it-strategy-and-delivery.md](./lecture-notes/02-it-strategy-and-delivery.md) | Strategic alignment, phased delivery, a Python TCO model, risk registers, matching system complexity to org maturity | 2h |
| 3 | [lecture-notes/03-presenting-and-defending-a-design.md](./lecture-notes/03-presenting-and-defending-a-design.md) | Audience-aware presenting, top-down structure, diagram craft, the four-part trade-off defense, the one-page proposal | 2h |
| 4 | [exercises/exercise-01-scope-the-capstone.md](./exercises/exercise-01-scope-the-capstone.md) | Pick a real organization, write its strategic purpose sentence, set an in/out-of-scope boundary | 1.5h |
| 5 | [exercises/exercise-02-draft-the-architecture.md](./exercises/exercise-02-draft-the-architecture.md) | Draw the full six-layer architecture diagram and justify three key decisions | 2h |
| 6 | [exercises/exercise-03-build-a-delivery-plan.md](./exercises/exercise-03-build-a-delivery-plan.md) | Phase the build, model TCO in Python, write a risk register, write the operating plan | 2h |
| 7 | [challenges/challenge-01-defend-under-pressure.md](./challenges/challenge-01-defend-under-pressure.md) | A five-round hostile design review across four stakeholder personas | 1.5h |
| 8 | [challenges/challenge-02-re-architect-on-new-constraints.md](./challenges/challenge-02-re-architect-on-new-constraints.md) | Re-shape the plan under a 60% budget cut or a 20x scale shock | 1.5h |
| 9 | [mini-project/README.md](./mini-project/README.md) | Deliver the full system: schema, automation, security, analytics, AI feature, cost model, and a defended proposal | 10h |
| 10 | [homework.md](./homework.md) | Audit a real-world system, a TCO sensitivity analysis, diagram critique, build-vs-buy memo, a live practice defense | 5h |
| 11 | [quiz.md](./quiz.md) | 14 self-check questions + answer key | 1h |
| 12 | [resources.md](./resources.md) | Official docs, standards, diagramming and cost tools, and a closing glossary | — |

## By the end of this week you can…

- Take a real organization with no existing system and produce a scoped, measurable, honestly-bounded set of requirements for one.
- Draw and defend a full six-layer architecture — data through AI — that a stranger can trace one record through without hitting an unexplained gap.
- Model total cost of ownership in Python, phase a delivery plan by risk, and write a risk register you'd actually stand behind in a review.
- Build a working data foundation, a real security policy, a KPI-driven analytics layer, and an AI feature grounded in your own real data — with a human-in-the-loop check stated explicitly.
- Present a system top-down to a mixed audience, defend every contested decision in four parts, and reshape the whole plan under a sudden budget cut or scale shock without starting over.

## Up next

There isn't one — this is Week 12, the last week of **C37 · Crunch Information Systems**. If you've completed all 12 weeks' mini-projects, you've analyzed an organization, modeled and stored its data in SQL, automated its processes, integrated and deployed its systems in the cloud, governed and secured them, layered analytics and AI on top, and — this week — designed and defended a complete intelligent information system end to end. From here: ship your capstone, add it to your portfolio, and consider [C33 · Crunch SQL](../../../C33-CRUNCH-SQL/) if you want to go deeper on the query layer specifically, or explore the rest of the Code Crunch Worldwide catalog.

---

*Part of the Code Crunch Worldwide open curriculum · GPL-3.0 · If you find errors, please open an issue or PR.*
