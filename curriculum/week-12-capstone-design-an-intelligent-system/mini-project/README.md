# Mini-Project — Deliver a Complete Intelligent Information System

> Design, build the core of, cost, and defend a complete intelligent information system for a real organization — requirements, SQL data model, process automation, integrations, cloud deployment, security and governance, a BI dashboard, and an AI feature, packaged with an architecture diagram, a cost estimate, and a delivery plan you present and defend.

**Estimated time:** 8–10 hours across the week, on top of the exercises and challenges (which are this project's first draft, not separate work).

This is the course's final deliverable. Every week from 1 through 11 built one piece of this; this project is where you prove the pieces compose into one working, coherent, defensible system for an organization you chose yourself. You are not building a toy — you are producing exactly the artifact a real small organization would receive from a systems consultant: a working core, a clear-eyed cost and risk picture, and a proposal document that survives being forwarded to someone who wasn't in the room.

## What you already have

If you completed Exercises 1–3 and at least one Challenge, you already have:

- A scoped organization, strategic purpose sentence, and in/out-of-scope boundary (Exercise 1).
- A full six-layer architecture diagram with justified trade-offs (Exercise 2).
- A phased delivery plan, a Python TCO model, and a risk register (Exercise 3).
- At minimum one stress-test — a defense under hostile questions or a re-architecture under a new constraint (Challenge 1 or 2).

This project's job is to turn that design into **working evidence** for the data-foundation-through-analytics layers, plus a real (if modest) AI feature, and then wrap all of it in the presentation artifact from Lecture 3.

## Requirements

### 1. Requirements & scope (carry forward from Exercise 1)

- The organization description, strategic purpose sentence, and in/out-of-scope table, refined if your thinking changed while building.

### 2. SQL data model (build this — don't just diagram it)

- A normalized PostgreSQL (or SQLite) schema for your in-scope entities: `CREATE TABLE` statements with primary keys, foreign keys, and at least two `CHECK`/`NOT NULL` constraints that encode a real business rule (e.g., a status column restricted to valid values).
- Realistic seed data — at minimum 20–30 rows across your core tables, enough to make every query below return non-trivial results. Generate it with a Python script if hand-writing 20+ `INSERT`s is tedious; do not use a spreadsheet at any point in this pipeline.
- Three to five SQL queries that answer real questions this organization would ask (mirroring the "business questions" pattern from Week 1) — e.g., "which members haven't been active in 60 days," "what's this month's revenue by category."

### 3. Process automation (build one workflow)

- One concrete manual process from your Exercise 1 scope, automated as a runnable Python script — e.g., a reminder job, a weekly summary email/report generator, a data-quality check that flags bad rows. It must read from your real schema, not mock data.

### 4. Integration (design, and build if in scope)

- If your Exercise 1 scope includes an external system (payment processor, calendar, email service), show at minimum a designed integration point (what API, what data crosses the boundary, authenticated how) — a working call against a free-tier/sandbox API is a strong bonus, not a requirement, if the real credential setup is out of reach this week.

### 5. Cloud deployment plan

- Your chosen hosting approach (PaaS/IaaS/SaaS, named provider) with justification against Week 8's decision framework, and — if feasible in your remaining time — an actual deployment of at least the database and a thin API layer to a free/low-cost tier. If you don't deploy live, the plan must be concrete enough that someone else could execute it from your document alone.

### 6. Security & governance

- A roles table (who can do what) and, for PostgreSQL, at least one working `CREATE POLICY` row-level security rule enforcing it — not just a described intention.
- A one-page governance note: data ownership, retention period for each sensitive table, and one sentence on what personal data this system holds and what protects it (Week 9 pattern).

### 7. BI dashboard

- A star-schema-shaped set of 2–4 aggregate SQL queries (or a small warehouse if you built one in Week 10's pattern) answering your Exercise 2 "one KPI this system must surface." A rendered dashboard (a simple HTML page, a notebook with charts, or a tool like Metabase) is preferred; a well-labeled query + results table is an acceptable minimum.

### 8. AI feature

- One AI/ML feature tied directly to your strategic purpose sentence — a simple classifier, a rule-based or scikit-learn scoring model, or a well-scoped LLM-based feature (e.g., summarizing free-text notes, a Q&A assistant grounded in your own data) — built in Python, reading real data from your schema, with a stated human-in-the-loop check per Week 11's pattern.

### 9. Packaging (the presentation layer, per Lecture 3)

- `PROPOSAL.md` — the one-page executive summary from Lecture 3, section 6: problem, purpose, cost, phased value, top risks, the ask.
- `architecture-diagram` (from Exercise 2, refined if needed).
- `delivery-plan.md` (from Exercise 3, refined if needed).

## Deliverable structure

```
c37-week-12/mini-project/
├── PROPOSAL.md                  # one-page executive summary
├── requirements.md              # scope + strategic purpose (from Ex. 1)
├── architecture-diagram.md      # (or .svg/.png) full six-layer diagram
├── delivery-plan.md             # phases + risk register (from Ex. 3)
├── schema/
│   ├── schema.sql                # CREATE TABLE + constraints
│   ├── seed_data.py               # generates realistic seed rows
│   └── business_queries.sql      # 3-5 real business-question queries
├── automation/
│   └── automated_workflow.py     # the one automated process
├── security/
│   ├── roles_and_rls.sql          # roles + RLS policy
│   └── governance_note.md        # ownership, retention, PII note
├── analytics/
│   └── dashboard_queries.sql     # or a notebook/HTML dashboard
├── ai_feature/
│   └── ai_feature.py              # the AI/ML feature, human-in-the-loop noted
└── cost/
    └── tco_model.py               # from Exercise 3, final numbers
```

## Milestones

- **Milestone 1 (2h):** Finalize requirements + schema + seed data + business queries. This is your foundation layer — everything else depends on it being right (Lecture 1).
- **Milestone 2 (2h):** Build the automated workflow and the security/RLS layer.
- **Milestone 3 (2h):** Build the analytics queries/dashboard and the AI feature.
- **Milestone 4 (1.5h):** Write the deployment plan (and deploy, if time allows) and the governance note.
- **Milestone 5 (1.5h):** Write `PROPOSAL.md` using Lecture 3's structure; do a full dry-run defense against your own Challenge 1 questions before you consider it done.

## Rubric

| Criterion | Weight | "Great" looks like |
|-----------|------:|--------------------|
| Requirements clarity & scope discipline | 10% | Specific organization, measurable "better," honest in/out-of-scope boundary |
| Data model quality | 20% | Normalized, constrained, seeded with realistic data, answers real business questions |
| Process automation | 10% | A real manual process replaced by a runnable script reading real data |
| Security & governance | 15% | Working RLS (or equivalent) plus a real governance note, not a description of intent |
| Analytics | 10% | KPI-driven queries/dashboard tied to the strategic purpose |
| AI feature | 15% | Grounded in real data, tied to the strategic purpose, with a stated human-in-the-loop check |
| Cost & delivery plan | 10% | Python TCO model with sourced/assumed numbers, phased plan with independently valuable phases |
| Presentation (`PROPOSAL.md`) | 10% | One page, follows Lecture 3's structure, survives being forwarded without you present |

## Reflection (`REFLECTION.md`, ~250 words)

1. Which layer of your system was hardest to get right, and why — was it technical difficulty or a genuine ambiguity in what the organization actually needs?
2. If you ran Challenge 1's hostile review against this final version, what's the one question you're least prepared to answer?
3. Across all 12 weeks, which single skill from an earlier week (a specific SQL pattern, a security concept, a cloud decision) turned out to matter most in this capstone?
4. If this organization actually funded and deployed this system, what's the very first thing you'd build in Phase 2 — the thing this week's scope deliberately left out?

## Why this matters

Every real information-systems job eventually asks for exactly this: not a perfect system, but a *coherent, honest, defensible* one — scoped realistically, costed honestly, secured by default, and able to survive a skeptical question in the room. That is what eleven weeks of this course built toward, one layer at a time, and it is what you're proving here, for an organization you chose, on data you modeled yourself, without ever reaching for a spreadsheet.

When done: push your full `c37-week-12/` folder, take the [quiz](../quiz.md), and — if you're presenting live — walk through `PROPOSAL.md` out loud to someone who wasn't part of this course, and see if it holds.
