# Week 2 — Systems Analysis & Requirements

> **Goal:** by Sunday you can take a real, messy request — the kind stakeholders actually say out loud, like "just make it faster" — and turn it into a stakeholder map, a prioritized set of functional and non-functional requirements, and a handful of use cases with acceptance criteria a developer could build against without asking you a single clarifying question.

Welcome to Week 2 of **C37 · Crunch Information Systems**. Week 1 taught you to see an organization as people, process, data, and technology. This week you learn the discipline that turns a vague organizational pain point into something buildable: **systems analysis**. This is the highest-leverage skill in this entire course — more projects fail from bad requirements than from bad code. A system built perfectly to the wrong spec is still wrong.

We work all week against one running scenario: **Meridian Outfitters**, a 14-store outdoor gear retailer whose "special order" process — what happens when a customer wants an item the store doesn't stock — is held together by phone calls, sticky notes, and a shared spreadsheet that three different people edit at once. It is exactly the kind of mess real analysts get handed. You'll interview its stakeholders (scripted, so everyone works from the same material), write requirements against it, and model its behavior with use cases and user stories. Per this course's data rule, we track the requirements register itself — stakeholders, elicitation notes, requirements, user stories — in **SQL tables**, never a spreadsheet. That's not just a rule for this course; it's the professional practice. A requirements register that lives in a shared Excel file is exactly the kind of single-point-of-failure, no-audit-trail, everyone-has-a-different-copy problem this course exists to teach you to avoid.

## Learning objectives

By the end of this week, you will be able to:

- **Identify stakeholders** and elicit real needs through interviews, observation, and document review — and know which technique fits which situation.
- **Ladder** a vague complaint ("make it faster") down to a specific, testable need using follow-up questions and the Five Whys.
- **Write functional and non-functional requirements** that are clear, unambiguous, testable, atomic, and prioritized with MoSCoW.
- **Model system behavior** with actors, use cases, user stories, and acceptance criteria a QA person (or you, later) could actually verify.
- **Diagram current-state vs. future-state** processes and name precisely where the gap — the waste, the delay, the risk — lives.
- **Recognize and manage** scope creep, ambiguity, and conflicting stakeholder goals before they become a broken build.

## Prerequisites

- Week 1 of this course (seeing an organization as people/process/data/technology).
- Comfort with basic SQL `CREATE TABLE`, `INSERT`, and `SELECT` — Week 1 of [C33 Crunch SQL](../../../C33-CRUNCH-SQL/) or equivalent. You are not writing complex queries this week, just storing and retrieving structured notes.
- PostgreSQL 16+ **or** SQLite 3.35+ installed (see [`resources.md`](./resources.md) if you skipped this in Week 1).
- No prior business-analyst or requirements experience assumed.

## Set up the requirements register (do this first)

Everything this week reads and writes to one small set of tables: `stakeholders`, `elicitation_sessions`, `requirements`, and `user_stories`. Create them once, seeded with the first pass of real analysis work already done on the Meridian Outfitters special-order problem — you'll extend it as the week goes on.

**PostgreSQL:**

```bash
createdb meridian_sos
psql meridian_sos
```

**SQLite:**

```bash
sqlite3 meridian_sos.db
```

Paste this into the shell (works unchanged on both engines):

```sql
CREATE TABLE stakeholders (
    stakeholder_id  INTEGER PRIMARY KEY,
    name            TEXT NOT NULL,
    role            TEXT NOT NULL,
    department      TEXT NOT NULL,
    power_level     TEXT NOT NULL,   -- 'low' | 'medium' | 'high'
    interest_level  TEXT NOT NULL,   -- 'low' | 'medium' | 'high'
    primary_goal    TEXT NOT NULL,
    notes           TEXT
);

INSERT INTO stakeholders VALUES
(1,'Dana Reyes','Store Manager, Riverside','Store Operations','high','high','Serve the customer fast without adding steps to my team''s day','Runs the busiest store; loudest voice in every meeting'),
(2,'Marcus Webb','Warehouse Fulfillment Lead','Distribution Center','medium','high','Accurate stock counts and enough lead time to pick correctly','Owns the 40k sq ft DC that ships to all 14 stores'),
(3,'Priya Anand','VP Finance & Loss Prevention','Finance','high','medium','No unapproved discounts, full audit trail on every order','Signs off on any process touching money or inventory shrink'),
(4,'Jordan Blake','Store Associate, Riverside','Store Operations','low','high','A simple way to place and track an order without three phone calls','Does the work today; never asked for input yet'),
(5,'Sofia Nakamura','Regional Ops Director','Executive','high','medium','Fulfillment time down, complaint volume down, cost flat','Executive sponsor; controls the budget for this project'),
(6,'Tom Alvarez','Customer Service Lead, Call Center','Customer Service','medium','high','Ability to answer "where is my order" without transferring the caller','Fields ~40 escalation calls a week about special orders'),
(7,'Grace Kim','IT Systems Analyst','IT','medium','high','One system of record, integrated with POS and inventory, not a bolt-on','Will inherit whatever gets built'),
(8,'Elena Cho','Customer (voice-of-customer, via survey)','External','low','high','Know when my item will arrive without calling the store','Represented via aggregated survey data, not a direct interview');

CREATE TABLE elicitation_sessions (
    session_id      INTEGER PRIMARY KEY,
    stakeholder_id  INTEGER NOT NULL REFERENCES stakeholders(stakeholder_id),
    session_date    DATE NOT NULL,
    method          TEXT NOT NULL,   -- 'interview' | 'observation' | 'document review' | 'survey'
    summary         TEXT NOT NULL
);

INSERT INTO elicitation_sessions VALUES
(1,1,'2026-06-01','interview','Dana wants special orders "faster" — laddered down to: associates currently wait 15-40 min on hold with the DC to confirm stock before they can even promise a date to the customer.'),
(2,2,'2026-06-02','interview','Marcus says stockouts get promised anyway because stores do not see live DC inventory; he finds out an item is short only when he tries to pick it, after the customer was already told a date.'),
(3,4,'2026-06-02','observation','Shadowed Jordan for 2 hours: watched one special order take 3 phone calls, 1 sticky note, and 2 separate edits to the shared spreadsheet by different people, 22 minutes total for a single order.'),
(4,3,'2026-06-03','interview','Priya requires any order over $500 to have a documented approval step; current spreadsheet has no approval trail at all — anyone can edit any cell.'),
(5,6,'2026-06-04','interview','Tom says the #1 call center complaint is "nobody can tell me where my order is" — call center has no visibility into the spreadsheet at all today.'),
(6,7,'2026-06-04','document review','Reviewed the current shared spreadsheet: 6 tabs, no data validation, order status is a free-text cell (values seen: "ordered", "Ordered", "in transit", "?", blank).');

CREATE TABLE requirements (
    req_id          TEXT PRIMARY KEY,   -- e.g. 'FR-001', 'NFR-001'
    req_type        TEXT NOT NULL,      -- 'functional' | 'non-functional'
    statement       TEXT NOT NULL,
    priority        TEXT NOT NULL,      -- MoSCoW: 'must' | 'should' | 'could' | 'wont'
    source_stakeholder_id INTEGER REFERENCES stakeholders(stakeholder_id),
    status          TEXT NOT NULL DEFAULT 'draft'   -- 'draft' | 'approved' | 'rejected'
);

CREATE TABLE user_stories (
    story_id            TEXT PRIMARY KEY,   -- e.g. 'US-001'
    actor               TEXT NOT NULL,
    wants               TEXT NOT NULL,
    benefit             TEXT NOT NULL,
    priority            TEXT NOT NULL,
    acceptance_criteria TEXT NOT NULL,
    linked_req_id       TEXT REFERENCES requirements(req_id),
    status              TEXT NOT NULL DEFAULT 'draft'
);
```

Sanity check — this should print `8`:

```sql
SELECT COUNT(*) FROM stakeholders;
```

`requirements` and `user_stories` start empty — you populate them yourself across this week's lectures, exercises, and challenges. That's the point: you're doing the analyst's job, not reading someone else's answers.

## Weekly schedule

Approximately **28 hours**, matching the course's full-time pace.

| Day | Focus | Lectures | Exercises | Challenges | Quiz/Read | Homework | Mini-Project | Daily Total |
|-----------|------------------------------------------|---------:|----------:|-----------:|----------:|---------:|-------------:|------------:|
| Monday | Setup; stakeholder mapping + elicitation | 2h | 1h | 0h | 0.5h | 1h | 0h | 4.5h |
| Tuesday | Writing requirements that hold | 2h | 1.5h | 0h | 0.5h | 1h | 0h | 5h |
| Wednesday | Use cases & user stories | 2h | 1.5h | 1h | 0.5h | 1h | 0h | 6h |
| Thursday | Current vs. future state; challenges | 0h | 1.5h | 1h | 0.5h | 1h | 1h | 5h |
| Friday | Conflicting stakeholders; spec rescue | 0h | 0h | 1h | 0.5h | 1h | 1.5h | 4h |
| Saturday | Mini-project (requirements spec) | 0h | 0h | 0h | 0h | 0h | 2.5h | 2.5h |
| Sunday | Quiz + review | 0h | 0h | 0h | 1h | 0h | 0h | 1h |
| **Total** | | **6h** | **5.5h** | **3h** | **3.5h** | **5h** | **5h** | **28h** |

## How to navigate this week

Work top to bottom. Each piece assumes the ones above it.

| # | File | What's inside | ~Time |
|--:|------|---------------|------:|
| 1 | [lecture-notes/01-eliciting-requirements.md](./lecture-notes/01-eliciting-requirements.md) | Stakeholder mapping, elicitation techniques, interview laddering, the requirements register in SQL | 2h |
| 2 | [lecture-notes/02-writing-requirements-that-hold.md](./lecture-notes/02-writing-requirements-that-hold.md) | Functional vs. non-functional, qualities of a good requirement, MoSCoW prioritization | 2h |
| 3 | [lecture-notes/03-use-cases-and-user-stories.md](./lecture-notes/03-use-cases-and-user-stories.md) | Actors, use case specs, user stories, INVEST, Given/When/Then acceptance criteria | 2h |
| 4 | [exercises/exercise-01-write-user-stories.md](./exercises/exercise-01-write-user-stories.md) | Convert a feature request into a backlog of user stories | 1h |
| 5 | [exercises/exercise-02-functional-vs-nonfunctional.md](./exercises/exercise-02-functional-vs-nonfunctional.md) | Sort raw meeting notes into FR/NFR, prioritize, store in SQL | 1h |
| 6 | [exercises/exercise-03-current-vs-future-state.md](./exercises/exercise-03-current-vs-future-state.md) | Diagram current-state vs. future-state for the special-order process | 1.5h |
| 7 | [challenges/challenge-01-conflicting-stakeholders.md](./challenges/challenge-01-conflicting-stakeholders.md) | Reconcile Dana, Marcus, and Priya's conflicting goals into one spec | 1h |
| 8 | [challenges/challenge-02-ambiguous-spec-rescue.md](./challenges/challenge-02-ambiguous-spec-rescue.md) | Rewrite a one-line spec into a full testable requirement set | 1h |
| 9 | [mini-project/README.md](./mini-project/README.md) | Full requirements specification for a real (or the Meridian) need | 2.5h |
| 10 | [homework.md](./homework.md) | Extra practice, including a real interview of your own | 5h |
| 11 | [quiz.md](./quiz.md) | 14 self-check questions + answer key | 1h |
| 12 | [resources.md](./resources.md) | Standards, templates, and the few links worth your time | — |

## By the end of this week you can…

- Map a project's stakeholders by power and interest, and know who to interview first.
- Ladder a vague ask down to a specific, measurable need in a live conversation.
- Write a requirement that a stranger could test pass/fail without asking you what you meant.
- Model a feature as a use case *and* as a set of user stories, and know when to reach for which.
- Spot scope creep and conflicting goals before they become a blown deadline.

## Up next

Week 3 — Data Modeling & Databases — once you know exactly *what* the system must do, you design the ER model and schema that will hold its data.

---

*Part of the Code Crunch Worldwide open curriculum · GPL-3.0 · If you find errors, please open an issue or PR.*
