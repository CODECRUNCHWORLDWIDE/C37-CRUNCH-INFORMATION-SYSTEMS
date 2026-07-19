# Mini-Project — A Complete Requirements Specification

> Produce a full requirements specification for a real need: a stakeholder map, a prioritized set of functional and non-functional requirements, and a set of use cases with acceptance criteria — precise enough that a developer who has never talked to a single stakeholder could build against it correctly.

**Estimated time:** 2.5–3 hours, best done Saturday after the exercises and challenges.

This is the week's capstone, and it's the actual job. Nobody hires a systems analyst to write elegant `SELECT` statements — they hire one to walk into a mess of competing opinions and hand back a document the whole team can build from without a follow-up meeting every day. You'll do that once, completely, end to end.

---

## Choose your subject

Pick **one**:

- **Option A — Continue Meridian Outfitters.** Take the special-order system further than the week's exercises did: extend beyond order placement into a second real workflow at Meridian (e.g., "process a return on a special order," or "reassign an order between stores when a customer's plans change"). Use the existing `stakeholders` table and add elicitation sessions of your own invention, grounded in what you already know about these eight people.
- **Option B — A real need of your own.** An organization you're part of (a job, a club, a family business, a nonprofit) has a real, messy process held together the same way Meridian's was — a spreadsheet, a group chat, someone's memory. Pick a real one. It's more work to gather the stakeholder detail yourself, and it's the more valuable version of this project if you have access to real people to (informally) ask.

State your choice and your one-sentence reason for it at the top of your `README.md` deliverable.

---

## Deliverable

A directory in your portfolio `c37-week-02/mini-project/` containing:

1. **`stakeholder-map.md`** — every stakeholder you've identified, their power/interest classification, their primary goal, and (if Option A) the relevant rows inserted into `stakeholders`/`elicitation_sessions`; (if Option B) a short summary of how you gathered each person's input.
2. **`requirements.sql`** — `INSERT` statements populating `requirements` with **at least 10 requirements** (a real mix of functional and non-functional), each with a MoSCoW priority and a `source_stakeholder_id`.
3. **`use-cases.md`** — **at least 3 full use cases** (main flow, at least one alternate flow, at least one exception flow each) covering the most important behaviors your requirements imply.
4. **`user-stories.sql`** — `INSERT` statements populating `user_stories` with stories derived from your use cases, each with Given/When/Then acceptance criteria and `linked_req_id` set.
5. **`report.md`** — a one-page executive summary: the problem in plain English, the top 3 "must" requirements, and what you deliberately put on the "Won't" list and why.

Everything SQL-based runs against your `meridian_sos` database (or a renamed copy if you chose Option B) — never a spreadsheet, per this course's data rule.

---

## Requirements for the requirements (yes, really)

Every deliverable in this project is graded against the standards this week actually taught — not vibes:

- **Stakeholder map:** at least 5 distinct stakeholders, correctly power/interest classified, with at least one from each quadrant of the grid (Lecture 1 §2) represented or explicitly noted as absent and why.
- **Requirements:** every one passes the 6-point checklist from Lecture 2 §2 (clear, testable, atomic, feasible, traceable, prioritized). At least 3 must be non-functional, covering at least 2 different NFR categories (performance, availability, security, usability, scalability, compliance, maintainability).
- **Use cases:** each has a named primary actor, a numbered main success scenario, at least one alternate flow, and at least one exception flow — an unfinished use case (happy path only) does not meet the bar this week set.
- **User stories:** each passes INVEST (state it explicitly for at least 3 of them in `report.md`), and each has at least 2 acceptance criteria including a non-happy-path case.
- **Traceability:** you can run this query and get a sensible, complete result for every story:

```sql
SELECT us.story_id, us.wants, r.statement AS traces_to, s.name AS originally_from
FROM user_stories us
JOIN requirements r ON r.req_id = us.linked_req_id
JOIN stakeholders s ON s.stakeholder_id = r.source_stakeholder_id;
```

---

## Milestones

- **Milestone 1 (45 min):** Choose your subject, finalize the stakeholder map, run at least 2 elicitation "sessions" (real or, for Option A, plausibly reconstructed from what you know of these characters) and log them.
- **Milestone 2 (45 min):** Write and load all 10+ requirements, checked against the 6-point quality list and MoSCoW-prioritized.
- **Milestone 3 (45 min):** Write the 3 use cases — main, alternate, exception flows for each.
- **Milestone 4 (30–45 min):** Derive user stories from the use cases, write acceptance criteria, load and link them, write `report.md`.

---

## Rubric

| Criterion | Weight | "Great" looks like |
|-----------|------:|--------------------|
| Stakeholder mapping | 15% | 5+ stakeholders, correctly classified, quadrant coverage justified |
| Requirement quality | 30% | All pass the 6-point checklist; real FR/NFR mix; sensible MoSCoW spread (not all "must") |
| Use cases | 20% | 3 complete use cases with real alternate/exception flows, not happy-path-only |
| User stories + AC | 20% | INVEST-passing, testable Given/When/Then criteria, correctly linked |
| Executive report | 15% | A stakeholder who has never seen the detail could read `report.md` and understand the plan in 2 minutes |

---

## Reflection (add to `report.md`, ~200 words)

1. Which requirement took the most iterations to make truly testable, and what was wrong with your first draft?
2. Where did two stakeholders (real or Meridian's) genuinely disagree, and how did you resolve or document it?
3. What's on your "Won't" list, and what would change your mind about building it later?
4. Now that you've fully specified *what* this system must do, what do you expect to be hardest about designing its *data model* next week?

---

## Why this matters

This is the artifact that separates "I have an idea for an app" from "I can hand a development team something they can actually build correctly the first time." Everything in Week 3 onward — the ER model, the schema, the API design — depends on getting this right first. A beautifully normalized database built against the wrong requirements is still the wrong system.

When done: push, then take the [quiz](../quiz.md) and get ready for Week 3 — Data Modeling & Databases, where these requirements become an ER diagram and a real PostgreSQL schema.
