# Challenge 2 — Make It Idempotent

**Time:** ~60 minutes. **Difficulty:** Medium-Hard. **Answer key at the end — read it only after you've tried.**

## The scenario

A teammate wrote a script for CrunchRide's warehouse team: whenever a repair ticket is marked `severity = 'major'`, the warehouse needs to send a van to pick the bike up. The script checks for major tickets that don't have a pickup request yet, and creates one. It "worked" in their five-minute manual test. It shipped to a cron job running every 5 minutes. Three weeks later, the warehouse team is furious — some bikes have **two, even three** vans show up for the same pickup, wasting driver time and confusing everyone. Your job: find every way this script is broken, not just the first one you spot, and fix it.

## The schema (as shipped)

```sql
CREATE TABLE pickup_requests (
    request_id   SERIAL PRIMARY KEY,
    ticket_id    INTEGER NOT NULL REFERENCES repair_tickets(ticket_id),
    requested_ts TIMESTAMP NOT NULL DEFAULT now(),
    status       TEXT NOT NULL DEFAULT 'pending'   -- 'pending' | 'fulfilled'
);
```

## The script (as shipped)

```python
# flag_pickups.py — DO NOT COPY THIS, IT'S THE BROKEN VERSION
import psycopg2
from db import get_connection

def run():
    conn = get_connection()
    cur = conn.cursor()

    cur.execute("""
        SELECT ticket_id FROM repair_tickets WHERE severity = 'major'
    """)
    major_tickets = cur.fetchall()

    for (ticket_id,) in major_tickets:
        try:
            cur.execute("""
                SELECT request_id FROM pickup_requests WHERE ticket_id = %s
            """, (ticket_id,))
            existing = cur.fetchone()

            if not existing:
                cur.execute("""
                    INSERT INTO pickup_requests (ticket_id) VALUES (%s)
                """, (ticket_id,))
                conn.commit()
                print(f"Requested pickup for ticket {ticket_id}")
        except:
            pass

    cur.close()
    conn.close()

if __name__ == "__main__":
    run()
```

## Your task

### Part 1 — Diagnose (20 min)

Don't fix anything yet. Read the script line by line and list **every** distinct way it can misbehave. There are at least **four** separate problems here — some cause duplicate pickup requests, at least one hides failures entirely. For each one, write:

- What the bug is (point to the line).
- A concrete scenario that triggers it (e.g., "if the cron job's previous run is still finishing when the next one starts...").
- What actually goes wrong as a result.

Two of the four are genuinely subtle — don't stop as soon as you find the obvious one.

<details>
<summary>Hint if you're stuck after finding one or two</summary>

Ask yourself: what does the schema itself allow, regardless of what the Python code checks? What happens if two copies of this script run at the *exact* same moment (overlapping cron, a manual run while the scheduled one is also firing)? What happens to a ticket the warehouse already picked up and marked `fulfilled` — does the script leave it alone? What does `except: pass` actually do to your ability to find out any of this happened?

</details>

### Part 2 — Fix it (30 min)

Rewrite both the schema and the script so that:

1. **Running the script any number of times, back to back, produces at most one `pending` or `fulfilled` pickup request per ticket** — enforced by the **database itself**, not just application-level logic (application-level checks alone can't survive true concurrency — Part 1 should have shown you why).
2. **A ticket whose pickup is already `fulfilled` is left alone** and not re-flagged.
3. **No exception is ever silently swallowed.** Every failure gets logged with enough detail to diagnose it later.
4. The check-then-insert pattern is replaced with something that's safe even if two copies of the script run at the exact same instant.

### Part 3 — Prove it (10 min)

Show your fix actually works:

- Run the fixed script twice in a row against a `major` ticket with no existing pickup request. Query `pickup_requests` and show only **one** row exists.
- Manually mark a request `fulfilled`, then run the script again. Show no new row was created for that ticket.

## Answer key — the four problems

<details>
<summary>Reveal only after you've written your own diagnosis</summary>

1. **Check-then-insert race condition (the main bug).** `SELECT ... WHERE ticket_id = %s` followed by a separate `INSERT` is two round-trips with a gap between them. If two runs of the script (overlapping cron, a manual run colliding with the scheduled one) both execute the `SELECT` before either has done its `INSERT`, both see "no existing request" and both insert — two rows for one ticket. This is a classic **TOCTOU (time-of-check to time-of-use)** bug, and it's exactly what the warehouse team is experiencing.

2. **No database-level uniqueness constraint.** Even setting aside the race, nothing in the schema *prevents* two `pickup_requests` rows for the same `ticket_id`. Application code checking "does one exist?" is a courtesy, not a guarantee — only a `UNIQUE` constraint (or equivalent) makes duplication actually impossible, at the one layer (the database) that can enforce it atomically.

3. **No `fulfilled` filter.** The `SELECT ticket_id FROM repair_tickets WHERE severity = 'major'` query re-selects **every** major ticket, every run, forever — including ones the warehouse already picked up. The only thing stopping a re-flag today is bug #1's flawed existence check; fix that check without also filtering on `status != 'fulfilled'` (or scoping the existence check to `status = 'pending'`) and you'd start re-requesting pickups for tickets that are already done.

4. **The bare `except: pass`.** Any error — a dropped connection, a constraint violation, a typo — is silently discarded. The script prints nothing, logs nothing, and `run()` returns normally as if everything succeeded. Nobody would ever know it failed until, like here, someone downstream notices the *symptom* (duplicate vans) with zero information about the *cause*. This is arguably worse than the duplication bug: it guarantees that whatever you fix next will fail silently again someday.

### A fixed version

```sql
ALTER TABLE pickup_requests
    ADD CONSTRAINT uq_pickup_per_ticket UNIQUE (ticket_id);
```

```python
# flag_pickups.py — fixed
import logging
import psycopg2
from db import get_connection

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
log = logging.getLogger("flag_pickups")

def run():
    with get_connection() as conn:
        with conn.cursor() as cur:
            cur.execute("""
                SELECT ticket_id FROM repair_tickets WHERE severity = 'major'
            """)
            major_tickets = cur.fetchall()

        requested, skipped, errored = 0, 0, 0
        for (ticket_id,) in major_tickets:
            try:
                with conn.cursor() as cur:
                    # ON CONFLICT DO NOTHING relies on the UNIQUE constraint above —
                    # this INSERT is now atomic: either it creates exactly one row,
                    # or it silently no-ops if one already exists. No race window.
                    cur.execute("""
                        INSERT INTO pickup_requests (ticket_id)
                        VALUES (%s)
                        ON CONFLICT (ticket_id) DO NOTHING
                        RETURNING request_id
                    """, (ticket_id,))
                    inserted = cur.fetchone()
                conn.commit()

                if inserted:
                    requested += 1
                    log.info(f"Requested pickup for ticket {ticket_id}")
                else:
                    skipped += 1
            except Exception as exc:
                conn.rollback()
                errored += 1
                log.error(f"ticket_id={ticket_id} failed: {exc}", exc_info=True)

        log.info(f"Run complete. requested={requested} skipped={skipped} errored={errored}")

if __name__ == "__main__":
    run()
```

Note this fixed version still doesn't filter out `fulfilled` tickets from the *source* query — with the `UNIQUE` constraint and `ON CONFLICT DO NOTHING` in place, a `fulfilled` ticket's pickup row already exists, so the `INSERT` correctly no-ops (`skipped`, not a duplicate). That's a case where fixing the deeper problem (bug #1/#2) also incidentally covers bug #3 — but a stronger answer still adds an explicit `WHERE` filter for clarity and to avoid the wasted round-trip of attempting an insert you know will conflict.

</details>

## Submission

Commit your Part 1 diagnosis, fixed schema, fixed script, and Part 3 proof (query output showing no duplicates) to your portfolio under `c37-week-05/challenge-02/`.
