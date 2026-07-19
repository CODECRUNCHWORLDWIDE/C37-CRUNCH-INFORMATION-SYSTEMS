# Week 5 — Process Modeling & Automation

> **Goal:** by Sunday you can take a real, messy, multi-person workflow, draw it precisely in BPMN with swimlanes, gateways, and events, decide *which* steps are worth automating and which aren't, and ship a Python script — scheduled, logged, retried, and safe to re-run — that automates the worthwhile steps against the PostgreSQL store.

Welcome back to **C37 · Crunch Information Systems**. Week 3 gave CrunchRide a normalized schema. Week 4 taught you to pull answers out of it with SQL and pandas. This week the direction flips: instead of just *reading* the data, your code starts *acting* on it. Every real organization runs on processes — a customer reports a problem, a person reviews it, someone gets assigned, work happens, the record gets closed. Right now at CrunchRide, that process runs on phone calls, memory, and someone eyeballing a list. Your job this week is to first **draw the process honestly** — not the process from the training manual, the process that actually happens — and then **automate the parts of it that are mechanical, rule-based, and safe to hand to a computer**, while leaving the parts that need judgment to a human.

We stay with **CrunchRide**, the bike-share company from Weeks 3–4, and add one new slice of its business: what happens when a bike gets damaged. A rider ends a ride and flags a problem. Right now, a dispatcher manually reads the flag, decides how bad it is, and calls around to find a free technician near the bike — by memory, some days by guesswork. It's slow, it's inconsistent, and on a busy day tickets sit unassigned for hours. You'll model that process in BPMN, find the parts of it worth automating, and build the automation: a Python script that reads open repair tickets from Postgres, assigns them to the right technician automatically, logs every decision, runs on a schedule, and can be re-run safely if it fails partway through.

## Learning objectives

By the end of this week, you will be able to:

- **Model** a business process in BPMN — tasks, gateways (exclusive, parallel, inclusive), events (start, end, intermediate, timer, message), and swimlanes/pools — as a shared, unambiguous language for "how work actually flows."
- **Identify**, inside a diagrammed process, which steps are manual, repetitive, error-prone, or a bottleneck — and score each one as an automation candidate using frequency, rule-based-ness, error cost, and stability.
- **Automate** a multi-step workflow in Python that reads from and writes to the SQL store, replacing a manual, judgment-free step with deterministic code.
- **Add** scheduling (cron or an in-process scheduler), structured logging, and error handling so the automation is something a business could actually trust to run unattended.
- **Design for idempotency** — make an automation safe to re-run after a crash or a retry, without double-processing or corrupting data.
- **Measure** the before/after impact of automating a process: time saved, backlog cleared, errors removed — with real queries against real timestamps, not a guess.

## Prerequisites

- **Weeks 1–4** of this course, or equivalent comfort with information-systems thinking, requirements, ER modeling, and querying with SQL + pandas.
- Comfortable writing basic Python: functions, `try`/`except`, loops, and reading a library's docs. [C1 · Code Crunch Convos — Python Bootcamp](../../../C1-Code-Crunch-Convos/) is the ideal companion if Python itself is new.
- PostgreSQL 16+ installed (primary engine). SQLite 3.35+ as a fallback for the SQL portions — the Python automation script in Lecture 3 assumes Postgres, since it uses features (like `RETURNING` and advisory-style guard clauses) you'll want in production; a SQLite note is included where the two diverge.
- Python 3.10+ with `pip` working. Install steps and the exact packages you need are in [`resources.md`](./resources.md).
- A way to view/edit a BPMN-style diagram — pen and paper is genuinely fine for every exercise here; the free [bpmn.io](https://demo.bpmn.io/) editor or Mermaid (rendered right in this README, no install) work well if you want something shareable.

## Set up this week's database (do this first)

This week extends the CrunchRide database with the tables for its repair workflow: `stations`, `bikes`, `technicians`, `repair_tickets`, and `automation_run_log`. If you still have last week's `crunchride` database, reuse it — these `CREATE TABLE` statements are new and don't touch anything from Week 3/4. If you're starting fresh, create the database first.

**PostgreSQL:**

```bash
createdb crunchride     # skip if it already exists from Week 3/4
psql crunchride
```

```sql
CREATE TABLE stations (
    station_id     INTEGER PRIMARY KEY,
    name           TEXT NOT NULL,
    zone           TEXT NOT NULL,        -- 'North' | 'Central' | 'South'
    dock_capacity  INTEGER NOT NULL
);

INSERT INTO stations VALUES
(1,'Harbor Point','North',18),
(2,'Old Mill Square','North',14),
(3,'City Hall Plaza','Central',24),
(4,'University Commons','Central',20),
(5,'Riverside Walk','South',16),
(6,'Sunset Terminal','South',12);

CREATE TABLE bikes (
    bike_id            INTEGER PRIMARY KEY,
    model              TEXT NOT NULL,     -- 'classic' | 'e-bike'
    current_station_id INTEGER REFERENCES stations(station_id),
    status             TEXT NOT NULL DEFAULT 'in_service',  -- 'in_service' | 'in_repair' | 'retired'
    last_service_date  DATE NOT NULL
);

INSERT INTO bikes VALUES
(101,'classic',1,'in_service','2026-04-02'),
(102,'classic',1,'in_service','2026-03-11'),
(103,'e-bike',2,'in_service','2026-05-20'),
(104,'classic',2,'in_repair','2026-01-15'),
(105,'e-bike',3,'in_service','2026-06-01'),
(106,'classic',3,'in_service','2026-02-08'),
(107,'classic',3,'in_service','2026-05-30'),
(108,'e-bike',4,'in_service','2026-04-19'),
(109,'classic',4,'in_service','2026-01-29'),
(110,'e-bike',5,'in_service','2026-05-05'),
(111,'classic',5,'in_service','2026-03-22'),
(112,'classic',5,'in_repair','2026-02-14'),
(113,'e-bike',6,'in_service','2026-06-10'),
(114,'classic',6,'in_service','2026-04-27');
```

```sql
CREATE TABLE technicians (
    tech_id           INTEGER PRIMARY KEY,
    name              TEXT NOT NULL,
    zone              TEXT NOT NULL,       -- home zone, matches stations.zone
    active            BOOLEAN NOT NULL DEFAULT TRUE,
    max_open_tickets  INTEGER NOT NULL DEFAULT 3
);

INSERT INTO technicians VALUES
(1,'Marcus Webb','North',TRUE,3),
(2,'Priya Anand','Central',TRUE,4),
(3,'Jordan Blake','Central',TRUE,3),
(4,'Sofia Nakamura','South',TRUE,3),
(5,'Tom Alvarez','North',FALSE,3);   -- inactive: out this week, must never be auto-assigned

CREATE TABLE repair_tickets (
    ticket_id       SERIAL PRIMARY KEY,
    bike_id         INTEGER NOT NULL REFERENCES bikes(bike_id),
    opened_ts       TIMESTAMP NOT NULL,
    reported_by     TEXT NOT NULL,        -- 'rider_report' | 'field_audit'
    issue_notes     TEXT NOT NULL,
    severity        TEXT NOT NULL,        -- 'minor' | 'major'
    status          TEXT NOT NULL DEFAULT 'open',   -- 'open' | 'assigned' | 'in_progress' | 'completed' | 'cannot_repair'
    assigned_tech_id INTEGER REFERENCES technicians(tech_id),
    assigned_ts     TIMESTAMP,
    completed_ts    TIMESTAMP
);

-- Historical, already-completed tickets — these were assigned MANUALLY by a dispatcher.
-- You'll use their timestamps in Lecture 2/3 to measure how slow "manual" really was.
INSERT INTO repair_tickets
(bike_id, opened_ts, reported_by, issue_notes, severity, status, assigned_tech_id, assigned_ts, completed_ts) VALUES
(104,'2026-06-01 08:15','rider_report','Rear brake not engaging','major','completed',1,'2026-06-01 12:40','2026-06-01 16:05'),
(112,'2026-06-03 09:02','rider_report','Flat tire, front','minor','completed',4,'2026-06-03 14:55','2026-06-03 15:30'),
(107,'2026-06-05 07:40','field_audit','Chain slipping under load','minor','completed',2,'2026-06-05 11:10','2026-06-05 12:00'),
(109,'2026-06-06 10:20','rider_report','Seat post loose','minor','completed',3,'2026-06-06 13:05','2026-06-06 13:20'),
(103,'2026-06-08 06:55','field_audit','Battery not charging','major','completed',1,'2026-06-08 15:30','2026-06-09 09:10');

-- Today's backlog — OPEN, unassigned. This is what your automation will process.
INSERT INTO repair_tickets
(bike_id, opened_ts, reported_by, issue_notes, severity, status) VALUES
(101,'2026-07-18 07:05','rider_report','Gears skipping on hills','minor','open'),
(105,'2026-07-18 07:40','rider_report','Display panel blank','major','open'),
(108,'2026-07-18 08:10','field_audit','Headlight not working','minor','open'),
(110,'2026-07-18 08:30','rider_report','Loud grinding noise, rear wheel','major','open'),
(114,'2026-07-18 09:00','rider_report','Kickstand bent, won''t hold bike up','minor','open');

CREATE TABLE automation_run_log (
    log_id      SERIAL PRIMARY KEY,
    run_ts      TIMESTAMP NOT NULL DEFAULT now(),
    ticket_id   INTEGER REFERENCES repair_tickets(ticket_id),
    step        TEXT NOT NULL,          -- e.g. 'assign_technician'
    status      TEXT NOT NULL,          -- 'ok' | 'skipped' | 'error'
    detail      TEXT
);
```

**SQLite** (for the SQL-only lectures/exercises — the Lecture 3 automation script targets Postgres):

```bash
sqlite3 crunchride.db
```

Paste the same statements with two swaps: `SERIAL PRIMARY KEY` → `INTEGER PRIMARY KEY AUTOINCREMENT`, and drop `now()` in favor of `CURRENT_TIMESTAMP`.

Sanity check — this should print `10` (5 historical + 5 backlog):

```sql
SELECT COUNT(*) FROM repair_tickets;
```

And this should print `5` — today's open, unassigned backlog, exactly what a dispatcher is staring at right now:

```sql
SELECT COUNT(*) FROM repair_tickets WHERE status = 'open' AND assigned_tech_id IS NULL;
```

## Weekly schedule

Approximately **28 hours**, matching the course's full-time pace.

| Day | Focus | Lectures | Exercises | Challenges | Quiz/Read | Homework | Mini-Project | Daily Total |
|-----------|------------------------------------------------|---------:|----------:|-----------:|----------:|---------:|-------------:|------------:|
| Monday | Setup; BPMN notation + the CrunchRide process | 2h | 1h | 0h | 0.5h | 1h | 0h | 4.5h |
| Tuesday | Draw the full BPMN diagram | 0h | 1.5h | 0h | 0.5h | 1h | 0h | 3h |
| Wednesday | Finding what to automate; scoring candidates | 2h | 1.5h | 0h | 0.5h | 1h | 0h | 5h |
| Thursday | Bottleneck analysis; script a 3-step workflow | 0h | 1.5h | 1h | 0.5h | 1h | 1h | 5h |
| Friday | Reliable automation: scheduling, logging, retries | 2h | 0h | 1.5h | 0.5h | 1h | 1.5h | 6.5h |
| Saturday | Mini-project (model + automate + measure) | 0h | 0h | 0h | 0h | 0h | 2.5h | 2.5h |
| Sunday | Quiz + review | 0h | 0h | 0h | 1h | 0h | 0h | 1h |
| **Total** | | **4h** | **4.5h** | **2.5h** | **3.5h** | **5h** | **5h** | **27.5h** |

## How to navigate this week

Work top to bottom. Each piece assumes the ones above it.

| # | File | What's inside | ~Time |
|--:|------|---------------|------:|
| 1 | [lecture-notes/01-modeling-processes-with-bpmn.md](./lecture-notes/01-modeling-processes-with-bpmn.md) | BPMN tasks, gateways, events, swimlanes — modeling CrunchRide's damage-to-repair process | 2h |
| 2 | [lecture-notes/02-finding-what-to-automate.md](./lecture-notes/02-finding-what-to-automate.md) | Scoring automation candidates; using the ticket timestamps as an event log to find real bottlenecks | 2h |
| 3 | [lecture-notes/03-building-reliable-automation.md](./lecture-notes/03-building-reliable-automation.md) | A full Python automation script: scheduling, structured logging, retries, idempotency, against Postgres | 2h |
| 4 | [exercises/exercise-01-draw-a-bpmn-diagram.md](./exercises/exercise-01-draw-a-bpmn-diagram.md) | Draw a BPMN diagram for new-technician onboarding, from a raw interview transcript | 1h |
| 5 | [exercises/exercise-02-spot-the-bottleneck.md](./exercises/exercise-02-spot-the-bottleneck.md) | Annotate a process for automation candidates; query the real bottleneck in SQL | 1.5h |
| 6 | [exercises/exercise-03-script-a-workflow.md](./exercises/exercise-03-script-a-workflow.md) | Script a 3-step workflow in Python: find, act, log — idempotently | 1.5h |
| 7 | [challenges/challenge-01-automate-a-real-process.md](./challenges/challenge-01-automate-a-real-process.md) | Model and automate one step of a real manual process from your own life or work | 1.5h |
| 8 | [challenges/challenge-02-make-it-idempotent.md](./challenges/challenge-02-make-it-idempotent.md) | Diagnose and fix a fragile automation that breaks when re-run | 1h |
| 9 | [mini-project/README.md](./mini-project/README.md) | Model, automate, schedule, log, and measure the full ticket-assignment workflow | 2.5h |
| 10 | [homework.md](./homework.md) | Extra BPMN diagrams, more automation candidates, extending the script | 5h |
| 11 | [quiz.md](./quiz.md) | 15 self-check questions + answer key | 1h |
| 12 | [resources.md](./resources.md) | BPMN references, free tools, and the Python packages you need | — |

## By the end of this week you can…

- Sit in a process-review meeting and sketch a correct BPMN diagram — swimlanes, gateways, and all — before it ends.
- Look at a diagrammed process and say, with reasons, exactly which steps are worth automating this quarter and which aren't.
- Write a Python script that reads and writes a SQL store on a schedule, survives a crash mid-run, and can be re-run without making a mess.
- Prove, with real queries against real timestamps, that an automation actually saved time — not just assert it.

## Up next

[Week 6 — Enterprise Systems](../week-06-enterprise-systems/) — now that CrunchRide has a modeled, partly automated process, we zoom out to how ERP/CRM systems manage master data and data flows across an entire organization.

---

*Part of the Code Crunch Worldwide open curriculum · GPL-3.0 · If you find errors, please open an issue or PR.*
