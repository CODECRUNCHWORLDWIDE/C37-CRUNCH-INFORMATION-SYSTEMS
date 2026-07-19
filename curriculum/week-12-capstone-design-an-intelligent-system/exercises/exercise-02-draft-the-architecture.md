# Exercise 2 — Draft the Full System Architecture Diagram

**Estimated time:** 2 hours

**Depends on:** Exercise 1's scope document.

Lecture 1 gave you a six-layer stack and the Crunch Cycles example wired through it. This exercise makes you build the same trace for your own organization: one diagram, one entity flowing through every layer, labeled well enough that a stranger — or a hostile reviewer in Challenge 1 — could follow it without you narrating.

## Part A — List the layer contents (30 min)

In `04-layers.md`, for **your** scoped system (from Exercise 1), fill in each layer concretely. Don't skip a layer just because Exercise 1 marked something out of scope — note "N/A this phase" and say why, so the omission is a decision, not a gap.

```
1. DATA FOUNDATION
   Core tables: ...
   Primary keys / relationships: ...

2. INTEGRATION & PROCESS
   Automated workflow(s): ...
   External systems touched (if any): payment processor, calendar, email, none...

3. CLOUD & DEPLOYMENT
   Hosting choice (PaaS/IaaS/SaaS — justify against Week 8's decision framework): ...
   Expected scale (rough number of users/records — order of magnitude is fine): ...

4. SECURITY & GOVERNANCE
   Roles: ...
   What's protected and how (RLS? app-level checks?): ...

5. ANALYTICS
   The one KPI this system must surface: ...
   Refresh cadence (real-time? nightly? weekly?) and why that's good enough: ...

6. AI / DECISION SUPPORT
   The one AI feature, and the specific decision it improves: ...
   What happens when it's wrong (Week 11's human-in-the-loop pattern): ...
```

## Part B — Draw the diagram (45 min)

Draw a single diagram — ASCII art in a fenced code block, or a tool of your choice exported to an image/SVG and embedded — that traces **one concrete record** (e.g., one appointment, one order, one member sign-up) from entry to decision. It must show, at minimum:

- Where data enters the system (a person, a form, an external system).
- The data foundation (name the core tables/entities, not every column).
- Any process/integration step data passes through.
- The security boundary (draw it as an explicit box or dotted line — Lecture 3, section 3).
- The analytics and/or AI layer, and what it produces.
- Where the output lands back with a human who makes a decision.

Use Lecture 3's rule: **label boxes with nouns a stakeholder recognizes**, not raw table/column names. "Member sign-up" not `INSERT INTO members`. Keep a short legend if you use abbreviations.

A minimal ASCII starting shape (expand this — it is intentionally too sparse to submit as-is):

```
[Front-desk staff] --(new appt form)--> [API / app layer]
                                              |
                                    (writes, auth-checked)
                                              v
                                   +--------------------+
                                   |   PostgreSQL (OLTP) |
                                   |   RLS: staff/admin  |
                                   +----------+---------+
                                              |
                                    (nightly sync job)
                                              v
                                   +--------------------+
                                   |  Reporting warehouse |
                                   +---+--------------+---+
                                       |              |
                                       v              v
                                [Dashboard]    [No-show risk score]
                                       |              |
                                       v              v
                                [Manager decision] [Reminder call priority]
```

## Part C — Justify three choices (30 min)

Pick **three** decisions from your diagram and write a short justification for each, using Lecture 3's four-part trade-off structure (decision, rejected alternative, real trade-off, what would change your mind). Good candidates: your hosting choice, your database engine, whether the AI feature is build-vs-buy, your security enforcement point (app-level vs. database RLS).

## Part D — Stress-test with a walk-through (15 min)

Pick one specific scenario and trace it through your own diagram, step by step, in prose: e.g., "A new member signs up on a Tuesday evening. Walk through exactly what touches their data, in order, until the manager's Monday-morning dashboard reflects them." If a step is unclear or missing from your diagram, fix the diagram — this walk-through is how you catch Lecture 1's "seams" (schema drift, duplicated security checks, batch/real-time mismatch, cost blind spots) before someone else does.

## Deliverable

A folder `c37-week-12/exercise-02/` containing:

- `04-layers.md` — the six-layer breakdown
- `05-architecture-diagram.md` (or `.svg`/`.png` alongside a short caption) — the diagram
- `06-justifications.md` — three four-part trade-off justifications
- `07-walkthrough.md` — the one-scenario trace

## Expected outcome

A diagram and supporting notes specific enough that Exercise 3 can cost it layer by layer, and specific enough that a reviewer in Challenge 1 has real surface area to push on — not three boxes and an arrow labeled "magic."

Next: [Exercise 3 — Build a delivery plan](exercise-03-build-a-delivery-plan.md).
