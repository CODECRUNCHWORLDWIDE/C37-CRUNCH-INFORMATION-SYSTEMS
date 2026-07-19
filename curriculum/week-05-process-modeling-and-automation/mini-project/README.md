# Mini-Project — Model, Automate, and Measure the Repair-Ticket Workflow

> Take CrunchRide's damage-to-repair process — modeled in BPMN, scored for automation candidates, and automated in Python against PostgreSQL, with scheduling, logging, and a before/after measure of real time saved. This is the week's capstone: everything from Lectures 1–3 and Exercises 1–3, combined into one complete, defensible deliverable.

**Estimated time:** 2.5–3 hours, best done Saturday after the exercises and challenges are behind you.

A stakeholder doesn't hand you a Python assignment — they hand you a slow, error-prone, manual process and a vague hope that "some of this could probably be automated." Your value is doing exactly what this week taught: model the process honestly, decide *with evidence* what's worth automating, build it so it can be trusted to run unattended, and prove — with real numbers — that it actually helped.

---

## Deliverable

A directory in your portfolio `c37-week-05/mini-project/` containing:

1. **`process-diagram.md`** (or `.bpmn`/image) — the full BPMN diagram of CrunchRide's damage-to-repair process, at least as complete as Lecture 1's worked example, with every lane, task type, gateway type, and event labeled.
2. **`automation-decision-log.md`** — the Lecture 2 rubric applied to every human task in your diagram, with a written verdict for each, backed by at least 3 real SQL queries against `repair_tickets`/`bikes`/`stations`/`technicians` quantifying the actual bottleneck (reuse and extend your Exercise 2 work).
3. **`assign_technicians.py`** — the automation itself, following Lecture 3's pattern: reads eligible tickets, applies the zone + fewest-open-tickets rule, writes the assignment, logs every decision (including skips), wrapped in transactions, with fault isolation so one bad ticket doesn't kill the batch.
4. **`db.py`** and **`requirements.txt`** — supporting files, same as Lecture 3.
5. **A scheduling config** — either a `crontab` snippet in a `schedule.txt` file, or an `apscheduler_runner.py` if you chose the in-process approach. State which you picked and why, in one paragraph.
6. **`before-after-report.md`** — the payoff. Real numbers: manual assignment time (historical, from `completed` tickets) vs. automated assignment time (after running your script against today's backlog), plus the idempotency proof (run it twice, show the second run only produced `skipped` log entries, zero duplicate assignments).

Everything runs against the `crunchride` database seeded in the [week README](../README.md). PostgreSQL required for the automation script (it uses transactions and `RETURNING` the way Lecture 3 does); note if you additionally verified the SQL-only parts against SQLite.

---

## Requirements, in detail

### 1. The diagram

Must include, at minimum:

- At least 4 swimlanes (Rider, App/System, Dispatcher, Field Technician — add Depot as a 5th lane or separate pool, your call, stated explicitly).
- A start event with a named trigger, and multiple end events representing the process's real distinct outcomes (back in service via field repair, back in service via depot repair, retired).
- At least one exclusive gateway (severity decision) and one parallel gateway (the needs-pickup-flag + senior-tech-assignment split), matched with correct joins.
- Every task labeled with its type (user/manual/service).

### 2. The decision log

For **every** human task in your diagram (not just the two Lecture 2 picked), a scored row and a one-line verdict. At least 3 supporting SQL queries, each with its actual output pasted in (not just the query — the *result*, so a reader can see the evidence, not just trust your claim).

### 3. The automation

Must satisfy every property from Lecture 3:

- **Idempotent** — the `WHERE` guard on the `UPDATE`, matching what the `SELECT` used to find eligible tickets.
- **Logged** — every assignment, skip, and error writes a row to `automation_run_log`, in the same transaction as the change it describes.
- **Fault-isolated** — a `try`/`except` around each ticket individually, not around the whole run.
- **Scheduled** — a real cron entry or APScheduler job definition, not just "run it manually when you remember."
- Uses **parameterized queries** throughout — no string-built SQL.

### 4. The before/after report

Must include, with actual numbers from your own run:

- Average and worst-case manual assignment time, from the 5 historical `completed` tickets.
- Average assignment time for tickets your script actually assigned (today's minor-severity backlog).
- The count of tickets processed, skipped, and errored on your first run.
- Proof of idempotency: run the script a second time immediately, and show the `automation_run_log` output from that second run — it should be entirely `skipped` entries with zero new `ok` assignments.
- One paragraph: what would it take to extend this to the major-severity path? (You're not building it — Lecture 2 scored that "not yet" — but you should be able to say, precisely, what's missing: likely a `seniority` column on `technicians` and a clearer rule for what "senior" means.)

---

## Milestones

- **Milestone 1 (45 min):** Diagram complete and reviewed against the Section 1 checklist above.
- **Milestone 2 (30 min):** Decision log complete, all human tasks scored, queries run and pasted in with real output.
- **Milestone 3 (60 min):** `assign_technicians.py` written, tested against the seed backlog, run twice to confirm idempotency.
- **Milestone 4 (30 min):** Scheduling config written, before/after report assembled with real numbers.

---

## Rules

- **This week's data rule applies in full.** Every table involved is SQL (Postgres, SQLite noted as fallback for the SQL-only parts). Nothing in this deliverable should ever be a spreadsheet.
- **The script must actually run** against the seed data and produce the exact assignments the eligibility rule implies (the 3 minor-severity backlog tickets — bikes 101, 108, 114 — get assigned or logged as skipped-with-reason; the 2 major-severity ones are untouched).
- **State every assumption.** If you interpret "same zone" or "fewest open tickets" slightly differently than Lecture 3's exact wording, say so and justify it — consistent, documented judgment calls are fine; silent, undocumented ones are not.

---

## Rubric

| Criterion | Weight | "Great" looks like |
|-----------|------:|---------------------|
| BPMN diagram correctness | 20% | Right notation throughout, matches the real process, correctly typed gateways with matched joins |
| Decision log + evidence | 20% | Every task scored, verdicts backed by real query output, not assertion |
| Automation correctness | 25% | Script runs, produces correct assignments, matches the stated business rule exactly |
| Reliability properties | 20% | Genuinely idempotent (proven, not claimed), logged, fault-isolated, scheduled |
| Before/after report | 15% | Real numbers, honest about what's not yet automated and why |

---

## Reflection (add to `before-after-report.md`, ~200 words)

1. Which part of this pipeline — modeling, scoring, or building — took longest, and did that match what you expected going in?
2. Where did idempotency almost bite you, even in this controlled exercise?
3. If CrunchRide's real dispatcher read your decision log, which verdict would they push back on hardest? What evidence would change your mind?
4. What's the next task you'd automate after this one, and what would you need (a new column, more historical data, a clearer rule) before you could defend automating it?

---

## Why this matters

This is the complete shape of real process-improvement work: understand the process as it truly runs, decide deliberately (not reflexively) what deserves code, build that code so it can be trusted unattended, and prove the value with numbers instead of a demo. Every later week in this course — enterprise systems, integration, cloud deployment, security, analytics — assumes you can do exactly this: take a real organizational process and turn it into a trustworthy system, one honest step at a time.

When done: push, then take the [quiz](../quiz.md) and start [Week 6 — Enterprise Systems](../../week-06-enterprise-systems/).
