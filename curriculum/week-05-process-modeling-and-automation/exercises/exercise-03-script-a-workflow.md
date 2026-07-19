# Exercise 3 — Script a 3-Step Workflow in Python

**Goal:** Build a small, complete automation — find, act, log — before Lecture 3's full technician-assignment script. This one is deliberately simpler (no gateway logic, no scheduling, no retry decorator) so you can get the idempotency pattern into your hands without juggling everything at once.

**Estimated time:** 90 minutes.

## The scenario

CrunchRide bikes are supposed to get a preventive service check every 90 days, regardless of whether a rider ever reports a problem. Right now, nobody tracks this — it's tribal knowledge. Your job: write a script that finds bikes overdue for service and automatically opens a repair ticket for them, so they enter the same workflow as a rider-reported issue.

## The 3-step workflow

1. **Find** — query `bikes` for every row where `status = 'in_service'` and `last_service_date` is more than 90 days before today, **and** that doesn't already have an open preventive ticket (the idempotency check — see below).
2. **Act** — for each overdue bike found, `INSERT` a new row into `repair_tickets` with `severity = 'minor'`, `reported_by = 'field_audit'`, and `issue_notes = 'Preventive maintenance — 90-day service overdue.'`.
3. **Log** — write one row to `automation_run_log` per ticket created (or per bike skipped because a ticket already exists), same as Lecture 3's pattern.

## Why idempotency is the whole point of this exercise

If you run this script twice without a guard, you'll open **two** duplicate tickets for the same overdue bike on the second run — nothing in a plain `INSERT` stops that. Before you write a line of Python, answer this in your own words in `exercise-03.md`: **what SQL condition would you add to Step 1's query so a bike that already has an open preventive ticket doesn't get picked up again?** Write the condition, then use it.

<details>
<summary>Hint if you're stuck</summary>

You need a way to tell "already has an open ticket from this automation" apart from "has no ticket at all" or "has a ticket, but it's already `completed`." A `NOT EXISTS` subquery against `repair_tickets`, checking for a row with the same `bike_id`, `reported_by = 'field_audit'`, and `status` still `open` or `assigned`, is one clean way to write it.

</details>

## Starter

```python
# preventive_maintenance.py
import logging
from datetime import date, timedelta
from db import get_connection   # reuse Lecture 3's db.py connection helper

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
log = logging.getLogger("preventive_maintenance")

SERVICE_INTERVAL_DAYS = 90


def find_overdue_bikes(conn):
    """TODO: return bike_ids for in-service bikes overdue for service
    that do NOT already have an open preventive ticket."""
    raise NotImplementedError


def open_preventive_ticket(conn, bike_id):
    """TODO: INSERT a new repair_ticket, then log the action.
    Both in the same transaction."""
    raise NotImplementedError


def run():
    log.info("Preventive maintenance sweep starting.")
    # TODO: fetch overdue bikes, open a ticket for each, commit, log a summary


if __name__ == "__main__":
    run()
```

Fill in the three `TODO`s. Reuse `db.py` and the logging pattern from Lecture 3 — don't reinvent them.

## Tasks

1. Implement `find_overdue_bikes` with the `NOT EXISTS` (or equivalent) idempotency guard.
2. Implement `open_preventive_ticket` — the `INSERT` and the `automation_run_log` write, same transaction.
3. Implement `run()` — loop over overdue bikes, call `open_preventive_ticket` for each, log a final summary line (`"Sweep complete. tickets_opened=N"`).
4. **Run it once.** Check `repair_tickets` for new rows with `reported_by = 'field_audit'` and the preventive-maintenance note.
5. **Run it a second time, immediately.** Confirm — by querying `repair_tickets` — that **no duplicate tickets** were created. Paste the before/after row counts into `exercise-03.md` as your proof.

## Expected result (spot checks)

- On the seed data, bikes with `last_service_date` more than 90 days before **2026-07-18** (today, per the week README) are overdue. Check the seed `INSERT` values in the week README against that date — several bikes qualify (e.g., bike 102's `2026-03-11`, bike 106's `2026-02-08`, bike 109's `2026-01-29`). Compute the exact list yourself rather than trusting this hint blindly — the "90 days ago" boundary is a good thing to get precisely right in SQL (`CURRENT_DATE - INTERVAL '90 days'`), not eyeball.
- After the first run, `repair_tickets` should have one new `open` row per overdue bike.
- After the second run, the row count should be **identical** to after the first run.

## Done when…

- [ ] `preventive_maintenance.py` runs without error against your `crunchride` database.
- [ ] Running it twice in a row produces zero duplicate tickets — verified by an actual row-count query, not just "it looked fine."
- [ ] `exercise-03.md` states your idempotency condition in plain English before showing the SQL, and includes the before/after row counts proving it worked.
- [ ] Every ticket the script opens has a matching `automation_run_log` row.

## Stretch

- Add a `--dry-run` flag (check `sys.argv`) that logs what it *would* do without actually writing to the database — a common, valuable safety feature for any automation that mutates data.
- What happens if `last_service_date` is `NULL` for some future bike? Add a guard or a comment explaining your script's behavior in that case (this is exactly the kind of edge case Challenge 2 later this week will make you hunt for on purpose).

## Submission

Commit `preventive_maintenance.py` and `exercise-03.md` to your portfolio under `c37-week-05/exercise-03/`.
