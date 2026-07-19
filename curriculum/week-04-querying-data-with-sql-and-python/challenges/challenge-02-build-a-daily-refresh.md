# Challenge 2 — Build a Script That Refreshes a Report Daily

**Time:** ~60 minutes. **Difficulty:** Medium-High.

## The scenario

Exercise 3 gave you a script that takes `--start`/`--end` and writes a region-revenue report. That's a good report engine, but "someone runs it by hand when they remember" isn't a real reporting pipeline — it's a chore waiting to be forgotten. Your job now is to make it something `cron` (or any scheduler) can run **unattended, every night, forever**, without a human watching.

## Your task

Extend Exercise 3's `region_report.py` (or start fresh, reusing its `extract`/`transform`/`load` functions) into `daily_refresh.py` that:

1. **Defaults to "yesterday through today"** when run with no arguments — `cron` won't pass `--start`/`--end` by hand every night, so the script needs a sensible default window, computed from the current date, not hardcoded.
2. **Still accepts an explicit `--start`/`--end`** for manual backfills or testing — don't lose Exercise 3's flexibility, just add a default on top of it.
3. **Appends to a running history table** (`daily_region_revenue`) instead of replacing a single-snapshot table — each day's run should add that day's numbers without disturbing previous days'. Use the delete-then-insert pattern from Lecture 3 §3 so re-running the same day twice doesn't duplicate rows.
4. **Logs every run** — start time, date range used, row counts extracted and written, and total elapsed time — using the `logging` module, not `print`.
5. **Exits with a non-zero status code on failure** (a bad date range, a database connection error, an empty extract that shouldn't be empty) so a scheduler can detect and alert on failed runs, not just silently skip a day.
6. **Never hardcodes today's date as a string** — compute it with `datetime.date.today()` (or accept an injectable "as of" date for testing, which is the more advanced version of the same idea — see stretch).

## Write it up

In `challenge-02.md`, answer:

1. **What is your default date window, exactly, and why that choice?** ("Yesterday" avoids partial-day data if orders trickle in during the current day — say whether that reasoning applies here and why.)
2. **Walk through what happens if the script is run twice for the same day** — trace it step by step and confirm no duplicate rows land in `daily_region_revenue`.
3. **Sketch the cron line** you'd use to run this at 2 AM daily (you don't need a live server — write the crontab syntax and explain each field).
4. **What's still missing** for this to be production-grade? (Alerting on failure, retry logic, monitoring, secrets management — name at least two gaps honestly; a good engineer knows what they *didn't* build.)

## Constraints

- The script must be safe to run **twice in a row for the same day** with no duplicate or doubled data — this is the whole point, verify it by actually running it twice and querying the table.
- No interactive prompts — a script `cron` runs unattended cannot pause for input.
- Connection details still come from environment variables (`.env` locally; real env vars in a scheduled context) — never hardcoded.

## Hints

<details>
<summary>On computing "yesterday" without hardcoding</summary>

```python
from datetime import date, timedelta

def default_window(as_of: date = None) -> tuple[str, str]:
    as_of = as_of or date.today()
    start = as_of - timedelta(days=1)
    return start.isoformat(), as_of.isoformat()
```

Taking `as_of` as an optional parameter (defaulting to `date.today()`) instead of calling `date.today()` deep inside the function is what makes this testable — a test can pass a fixed date and get a deterministic, repeatable result, instead of the test's answer changing depending on what day it happens to run.

</details>

<details>
<summary>On the crontab line</summary>

`cron`'s five fields are minute, hour, day-of-month, month, day-of-week. "Every day at 2 AM" is:

```
0 2 * * * cd /path/to/project && /path/to/.venv/bin/python daily_refresh.py >> /path/to/logs/refresh.log 2>&1
```

Note the absolute paths — `cron` runs with a minimal environment, not your interactive shell's `PATH`, so relying on a bare `python` or a relative path is a classic first-cron-job mistake.

</details>

## How success is judged

| Signal | Weak answer | Strong answer |
|--------|-------------|----------------|
| Idempotency | Re-running duplicates rows, or the writer didn't test it | Explicitly tested twice; write-up traces exactly why it's safe |
| Defaults | Hardcodes a date string somewhere | Computes the window from `date.today()`, injectable for tests |
| Failure handling | Errors crash with an unreadable traceback and exit 0 | Clear log message, non-zero exit code, `cron`-detectable |
| Honesty | Claims the script is "production ready" | Names real, specific gaps (alerting, retries, secrets) |

## Submission

Commit `daily_refresh.py` and `challenge-02.md` to your portfolio under `c37-week-04/challenge-02/`.
