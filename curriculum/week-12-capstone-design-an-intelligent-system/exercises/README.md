# Week 12 — Exercises

Three exercises, done in order, that build the capstone piece by piece: scope it, diagram it, cost and phase it. Unlike earlier weeks, these are not drilled against a shared seed table — you choose a **real organization** in Exercise 1, and every exercise after that builds on your own choice.

1. **[Exercise 1 — Scope the capstone](exercise-01-scope-the-capstone.md)** — pick a real organization, write its strategic purpose sentence, and define what's in and out of scope. ~1.5h
2. **[Exercise 2 — Draft the architecture](exercise-02-draft-the-architecture.md)** — draw the full six-layer system diagram for your organization. ~2h
3. **[Exercise 3 — Build a delivery plan](exercise-03-build-a-delivery-plan.md)** — phase the build, model TCO in Python, and write the risk register. ~2h

## Before you start

- You've completed all three lectures this week and, ideally, the prior 11 weeks (or you're comfortable with normalized schemas, REST APIs, cloud deployment basics, RBAC/RLS, a BI warehouse, and one AI/ML pattern).
- You have Python 3.10+ and PostgreSQL 16+ (or SQLite) available — Exercise 3's cost model runs in Python; if your scoped system needs a schema sketch, sketch it in SQL, never a spreadsheet.
- Pick a **real** organization now, before Exercise 1: your employer, a family business, a club or nonprofit you're part of, your school, or a local business whose owner will talk to you for 20 minutes. It does not need to be large — small, real, and specific beats large and hypothetical every time. If nothing comes to mind, Exercise 1 offers three ready-made fallback organizations.

## Suggested workflow

- Work the three exercises in order — Exercise 2's diagram depends on Exercise 1's scope, and Exercise 3's plan depends on Exercise 2's architecture.
- Keep a single working folder, e.g. `c37-week-12/`, with one subfolder per exercise. You'll carry this same folder into the mini-project — these exercises are not throwaway drills, they are the mini-project's first draft.
- Write in plain Markdown and SQL/Python files, not a slide deck yet — Lecture 3's presentation structure comes after the content is settled, in the mini-project.
