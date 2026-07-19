# Lecture 3 — Building Reliable Automation

> **Duration:** ~2 hours. **Outcome:** You can write a Python script that reads and writes a PostgreSQL store, runs on a schedule, logs every decision it makes, survives a crash or bad row without dying, and can be run twice in a row without doing double the work.

Lecture 2 decided *what* to automate: assigning a technician to a minor-severity repair ticket. This lecture builds it — not a demo script that only works when nothing goes wrong, but something you could actually hand to an operations team and trust to run unattended at 2am.

## 1. What "reliable" means here, concretely

A script that works once, on your laptop, while you watch it, is a prototype. A script that's **reliable** has four properties, and this lecture builds all four, in order:

1. **Idempotent** — running it twice (or ten times) in a row produces the same end state as running it once. Crashes happen mid-run; retries happen; cron can double-fire. The script must not care.
2. **Logged** — every decision it makes (and every decision it *skips*, and every error it hits) is written somewhere a human can audit later, without needing to have been watching the terminal live.
3. **Fault-isolated** — one bad row (a ticket with an unexpected value, a transient network blip) doesn't take down the whole batch. The script processes what it can and reports what it couldn't.
4. **Scheduled** — it runs on its own, on a cadence someone chose deliberately, not "whenever I remember to run it by hand."

## 2. The business rule, precisely

Before any code: the automation rule Lecture 2 approved, written as an unambiguous sentence — this is the translation step from Lecture 1/2's diagram into something you can actually implement.

> For every ticket where `status = 'open'`, `severity = 'minor'`, and `assigned_tech_id IS NULL`: find the **active** technician whose `zone` matches the bike's `current_station`'s `zone`, who currently has the **fewest open tickets** assigned (and is under their `max_open_tickets` cap). Assign that technician, set `status = 'assigned'` and `assigned_ts = now()`. If no eligible technician exists in that zone, leave the ticket alone and log why.

Notice this sentence resolves every ambiguity a vaguer version would have hidden: what "nearby" means (zone match), what "available" means (active + under their cap), and what happens on failure (leave it, don't guess, log it).

## 3. Project layout

```
crunchride_automation/
├── assign_technicians.py     # the automation itself
├── db.py                     # connection helper, one place to change credentials
├── .env                      # DB_HOST, DB_NAME, DB_USER, DB_PASSWORD (never committed)
└── requirements.txt          # psycopg2-binary, python-dotenv, APScheduler, tenacity
```

```
# requirements.txt
psycopg2-binary==2.9.9
python-dotenv==1.0.1
APScheduler==3.10.4
tenacity==8.2.3
```

```python
# db.py
import os
import psycopg2
from dotenv import load_dotenv

load_dotenv()

def get_connection():
    """One place to change how we connect. Everything else imports this."""
    return psycopg2.connect(
        host=os.environ["DB_HOST"],
        dbname=os.environ["DB_NAME"],
        user=os.environ["DB_USER"],
        password=os.environ["DB_PASSWORD"],
    )
```

Keeping credentials in `.env` (never in the script, never committed to git) and connection logic in one function is not ceremony — it's the difference between changing a password in one place versus hunting through every script that touches the database.

## 4. The core script, built up piece by piece

### Step 1 — logging, set up first, used everywhere

Set up structured logging *before* writing any business logic. A `print()` statement is invisible the moment nobody is watching the terminal; a log line with a timestamp, a level, and enough context survives to be read the next morning.

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s [%(levelname)s] %(message)s",
    handlers=[
        logging.FileHandler("automation.log"),
        logging.StreamHandler(),   # also print to terminal when run interactively
    ],
)
log = logging.getLogger("assign_technicians")
```

### Step 2 — the read: find eligible tickets

```python
def fetch_unassigned_minor_tickets(conn):
    """Tickets waiting for a minor-path automatic assignment."""
    with conn.cursor() as cur:
        cur.execute("""
            SELECT rt.ticket_id, s.zone
            FROM repair_tickets rt
            JOIN bikes b     ON b.bike_id = rt.bike_id
            JOIN stations s  ON s.station_id = b.current_station_id
            WHERE rt.status = 'open'
              AND rt.severity = 'minor'
              AND rt.assigned_tech_id IS NULL
            ORDER BY rt.opened_ts ASC
        """)
        return cur.fetchall()   # list of (ticket_id, zone)
```

`ORDER BY opened_ts ASC` matters: it means the oldest-waiting ticket gets assigned first — a small fairness decision worth making deliberately, not by accident of whatever order Postgres happens to return rows.

### Step 3 — the decision: pick a technician

```python
def find_best_technician(conn, zone):
    """Active tech in this zone, fewest open assignments, under their cap.
    Returns tech_id, or None if nobody is eligible right now."""
    with conn.cursor() as cur:
        cur.execute("""
            SELECT t.tech_id
            FROM technicians t
            LEFT JOIN repair_tickets rt
                   ON rt.assigned_tech_id = t.tech_id
                  AND rt.status IN ('assigned', 'in_progress')
            WHERE t.zone = %s
              AND t.active = TRUE
            GROUP BY t.tech_id, t.max_open_tickets
            HAVING COUNT(rt.ticket_id) < t.max_open_tickets
            ORDER BY COUNT(rt.ticket_id) ASC, t.tech_id ASC
            LIMIT 1
        """, (zone,))
        row = cur.fetchone()
        return row[0] if row else None
```

Always pass `zone` as a **parameter** (`%s` + a tuple), never with an f-string. String-building SQL from user- or process-supplied values is a SQL-injection risk even in an internal tool, and parameterized queries are exactly as easy to write.

### Step 4 — the write: assign, and log it in the same transaction

```python
def assign_ticket(conn, ticket_id, tech_id):
    """Write the assignment AND the audit log entry atomically."""
    with conn.cursor() as cur:
        cur.execute("""
            UPDATE repair_tickets
            SET status = 'assigned', assigned_tech_id = %s, assigned_ts = now()
            WHERE ticket_id = %s
              AND status = 'open'
              AND assigned_tech_id IS NULL
            RETURNING ticket_id
        """, (tech_id, ticket_id))
        updated = cur.fetchone()

        if updated is None:
            # Someone/something else already claimed this ticket since we read it.
            log_step(cur, ticket_id, "assign_technician", "skipped",
                      "Ticket no longer open/unassigned — already handled.")
            return False

        log_step(cur, ticket_id, "assign_technician", "ok",
                  f"Assigned to tech_id={tech_id}")
        return True


def log_step(cur, ticket_id, step, status, detail):
    cur.execute("""
        INSERT INTO automation_run_log (ticket_id, step, status, detail)
        VALUES (%s, %s, %s, %s)
    """, (ticket_id, step, status, detail))
```

Two things worth pausing on:

1. **`RETURNING ticket_id` combined with the same `WHERE` guard the read used** is the idempotency mechanism. If this exact `UPDATE` runs twice — a retry, a double cron fire, whatever — the second run's `WHERE status = 'open' AND assigned_tech_id IS NULL` matches **zero rows**, because the first run already changed `status`. `updated` comes back `None`, the script logs "skipped," and nothing bad happens. The database's own `WHERE` clause *is* the guard — you don't need a separate "have I done this before?" check.
2. **The assignment `UPDATE` and the log `INSERT` happen inside the same transaction** (see Step 5) — either both commit or neither does. A log entry that says "assigned" for an assignment that got rolled back would be worse than no log at all.

### Step 5 — tying it together with a transaction per ticket, and retrying transient failures

```python
from tenacity import retry, stop_after_attempt, wait_exponential, retry_if_exception_type
import psycopg2

@retry(
    retry=retry_if_exception_type(psycopg2.OperationalError),  # only retry connection-level hiccups
    stop=stop_after_attempt(3),
    wait=wait_exponential(multiplier=1, min=1, max=8),
    reraise=True,
)
def process_one_ticket(conn, ticket_id, zone):
    """One ticket, one transaction. Errors here don't touch other tickets."""
    tech_id = find_best_technician(conn, zone)
    if tech_id is None:
        with conn.cursor() as cur:
            log_step(cur, ticket_id, "assign_technician", "skipped",
                      f"No eligible active technician in zone '{zone}'.")
        conn.commit()
        return

    assigned = assign_ticket(conn, ticket_id, tech_id)
    conn.commit()
    return assigned


def run():
    log.info("Automation run starting.")
    processed, skipped, errored = 0, 0, 0

    with get_connection() as conn:
        tickets = fetch_unassigned_minor_tickets(conn)
        log.info(f"Found {len(tickets)} unassigned minor ticket(s) to process.")

        for ticket_id, zone in tickets:
            try:
                result = process_one_ticket(conn, ticket_id, zone)
                if result:
                    processed += 1
                else:
                    skipped += 1
            except Exception as exc:
                # One bad ticket must not stop the batch.
                conn.rollback()
                errored += 1
                log.error(f"ticket_id={ticket_id} failed: {exc}", exc_info=True)

    log.info(f"Run complete. processed={processed} skipped={skipped} errored={errored}")


if __name__ == "__main__":
    run()
```

The `try`/`except` **inside the loop, around one ticket** — not around the whole `run()` function — is the fault-isolation property. If ticket 47 has some data problem that raises an exception, tickets 48–51 still get processed; ticket 47's failure gets logged with a full stack trace (`exc_info=True`) and the run keeps going. A single `try` wrapped around the entire function would mean one bad row loses you the whole batch — exactly the failure mode a "reliable" automation is supposed to prevent.

`@retry` is scoped narrowly: it only retries `psycopg2.OperationalError` (connection drops, the kind of thing that's often transient and safe to retry), not every possible exception. Retrying a bug in your own business logic three times with exponential backoff just delays the failure and spams the log — only retry things that are plausibly *transient*.

## 5. Scheduling it

Two legitimate options; pick based on how the rest of your infrastructure works.

### Option A — cron (simplest; no long-running process)

```bash
# crontab -e
# Run every 15 minutes, log cron's own stdout/stderr separately from the script's own log file
*/15 * * * * cd /opt/crunchride_automation && /usr/bin/python3 assign_technicians.py >> cron.log 2>&1
```

Cron is dead simple, survives reboots, and needs nothing extra running — the OS wakes the script up, it runs, it exits. The tradeoff: no built-in retry-the-whole-run-later, no in-process state, and debugging "why didn't it run" means checking cron's own logs, which is its own small skill.

### Option B — APScheduler (an in-process Python scheduler)

```python
from apscheduler.schedulers.blocking import BlockingScheduler

scheduler = BlockingScheduler()
scheduler.add_job(run, "interval", minutes=15, id="assign_technicians")

if __name__ == "__main__":
    log.info("Scheduler starting — running every 15 minutes.")
    scheduler.start()
```

APScheduler keeps a Python process alive and schedules jobs from inside it — useful when you want richer scheduling logic (skip a run if the last one is still going, add jobs dynamically, coordinate multiple related jobs) or you're already running a long-lived Python service (a web app, a worker process) and don't want a second, separate scheduling mechanism. The tradeoff: now you have a process to keep alive, monitored, and restarted if it crashes — something like `systemd` or a process manager, which cron doesn't need.

**For this course:** cron is the right default for a standalone script like this one — simpler, and it's what you'll meet most often maintaining someone else's infrastructure. Use APScheduler when the automation is one job among several inside a larger running application.

## 6. Measuring before/after — the payoff

The whole point was consistency and speed. Prove it, the same way Lecture 2 quantified the problem:

```sql
-- BEFORE: manual assignment times (from the historical, manually-assigned tickets)
SELECT ROUND(AVG(EXTRACT(EPOCH FROM (assigned_ts - opened_ts)) / 60.0), 1) AS avg_minutes_manual
FROM repair_tickets
WHERE status IN ('completed')
  AND assigned_ts IS NOT NULL;

-- AFTER: run the script, then check how fast today's backlog got assigned
SELECT ROUND(AVG(EXTRACT(EPOCH FROM (assigned_ts - opened_ts)) / 60.0), 1) AS avg_minutes_automated
FROM repair_tickets
WHERE assigned_ts > (SELECT MIN(run_ts) FROM automation_run_log);

-- The audit trail itself — proof of exactly what the automation did and when
SELECT run_ts, ticket_id, step, status, detail
FROM automation_run_log
ORDER BY run_ts DESC;
```

Run `assign_technicians.py` once against this week's seed backlog and check `automation_run_log` — you should see the three minor-severity open tickets (bikes 101, 108, 114) each get an `ok` or `skipped` row, and the two major-severity tickets (bikes 105, 110) untouched, exactly matching the rule from Section 2. Run it a **second** time immediately after — the log should fill with `skipped` entries, not duplicate `ok` assignments. That second run *is* your idempotency test.

## 7. Check yourself

- What makes the `assign_ticket` function idempotent? Point to the exact line that does the work.
- Why does the `try`/`except` sit inside the per-ticket loop instead of around the whole `run()` function?
- Why does `@retry` only catch `psycopg2.OperationalError`, not a bare `except Exception`?
- What's the one-sentence tradeoff between cron and APScheduler?
- Why does the assignment `UPDATE` and the `automation_run_log` `INSERT` need to be in the same transaction?
- If you ran this script twice in a row, what would you expect to see in `automation_run_log` the second time, and why is that the correct, desired behavior?

If those are automatic, you're ready for the exercises — a smaller three-step version first, then this exact pattern applied end to end in the mini-project.

## Further reading

- **`psycopg2` documentation:** <https://www.psycopg.org/docs/>
- **PostgreSQL — Transactions:** <https://www.postgresql.org/docs/current/tutorial-transactions.html>
- **`tenacity` — retrying library docs:** <https://tenacity.readthedocs.io/>
- **APScheduler documentation:** <https://apscheduler.readthedocs.io/>
- **Python `logging` HOWTO (official docs):** <https://docs.python.org/3/howto/logging.html>
- **crontab.guru — build/verify cron schedule expressions:** <https://crontab.guru/>
