# Challenge 2 — Make Integration Resilient

**Time:** ~90 minutes. **Difficulty:** Hard. **Code required.**

## The scenario

Below is a real (deliberately fragile) ETL job, `fragile_fx_sync.py`, that someone on the Crunch Cycles team wrote in a hurry. It works — on a good day, with a fast network, run exactly once. Your job is to find every way it breaks and fix each one, proving the fix with a test you can actually run.

```python
# fragile_fx_sync.py — DO NOT COPY AS-IS. This is the "before" version.
import requests
import pandas as pd
from sqlalchemy import create_engine

engine = create_engine("postgresql://localhost/crunchcycles")

def run():
    resp = requests.get(
        "https://api.frankfurter.dev/v1/latest",
        params={"base": "USD", "symbols": "EUR,JPY,GBP,BRL"},
    )
    data = resp.json()
    rows = [
        {"rate_date": data["date"], "base_currency": "USD",
         "quote_currency": k, "rate": v}
        for k, v in data["rates"].items()
    ]
    df = pd.DataFrame(rows)
    df.to_sql("fx_rates", engine, if_exists="append", index=False)
    print("done")

run()
```

## Your task

Write `resilient_fx_sync.py` — a fixed version — plus `resilience-notes.md` documenting **every** bug you found and how you fixed it. At minimum, find and fix these four (there are more if you look closely):

1. **No timeout, no status check.** If Frankfurter is slow or down, this hangs indefinitely or silently treats an error page as data. Fix it using the patterns from Lecture 2.

2. **`if_exists="append"` has no idempotency.** Run the original script twice and count rows in `fx_rates` before/after — prove to yourself it duplicates data. Fix it using the `ON CONFLICT` upsert pattern from Lecture 3 (you'll need to switch from `df.to_sql` to a manual `INSERT ... ON CONFLICT` loop, or another mechanism you can justify — `to_sql` alone cannot express an upsert).

3. **No retry on transient failure.** Simulate a transient failure (temporarily point the URL at a nonexistent host, or use a tool like `unplug your wifi for 5 seconds` if you want to get physical about it) and confirm the original script just crashes. Add retry-with-backoff from Lecture 2, but only for the *retryable* status codes/exceptions — prove you're not retrying a `400`-class error that will never succeed.

4. **No logging, no failure signal for automation.** If this were in a `cron` job, how would anyone know it failed at 3 a.m.? Add logging (per Lecture 3's style) and make sure a genuine failure `raise`s instead of getting swallowed — `print("done")` at the end currently runs even conceptually adjacent to failure paths that don't exist yet; make sure your fixed version only logs success when the load genuinely succeeded.

## Prove each fix

For each of the four bugs, `resilience-notes.md` must include:

- **What breaks** in the original (a specific, reproducible scenario — "run it twice," "point it at a bad host," etc.)
- **What you changed** (a short code excerpt or a clear description).
- **Proof it's fixed** — a command you ran and its actual output (row counts, log lines, exit codes). "I added retry logic" without evidence it retries and eventually succeeds (or eventually gives up and raises, when appropriate) doesn't count.

## Constraints

- Your fixed version must still be a single, runnable script — no framework overhead beyond what Lecture 3 already uses (`requests`, `pandas`, `sqlalchemy`, `logging`).
- Do not simply wrap the whole `run()` function in one bare `try/except: pass` — that "fixes" every symptom by hiding it, which is worse than the original. Every fix should address the *specific* failure mode, and genuine unrecoverable failures must still surface loudly.

## Hints

<details>
<summary>On proving the duplicate-row bug</summary>

```sql
SELECT COUNT(*) FROM fx_rates;
-- run fragile_fx_sync.py
SELECT COUNT(*) FROM fx_rates;
-- run it again
SELECT COUNT(*) FROM fx_rates;
```

If the count goes up by the same amount both times, you've reproduced the bug. After your fix, the second and third counts must be **equal** — that's your proof.

</details>

<details>
<summary>On simulating a bad host without breaking your real internet</summary>

```python
BAD_URL = "https://api.frankfurter-typo-on-purpose.dev/v1/latest"
```

A misspelled hostname fails DNS resolution immediately and reliably — a clean, repeatable way to trigger `requests.exceptions.ConnectionError` without touching your actual network.

</details>

## How success is judged

| Signal | Weak answer | Strong answer |
|--------|-------------|----------------|
| Coverage | Fixes 1–2 of the 4 bugs | Fixes all 4, plus notes any extra ones found |
| Proof | Claims a fix works | Shows the actual before/after command output |
| Idempotency fix | Just adds `if_exists="replace"` (still not idempotent, just differently wrong — wipes history) | Correctly upserts on the natural key |
| Retry logic | Retries everything, including `400`s | Retries only transient failures; fails fast on permanent ones |
| Restraint | Wraps everything in a silent `try/except` | Fixes each failure mode specifically; real failures still surface |

## Submission

Commit `resilient_fx_sync.py` and `resilience-notes.md` to your portfolio under `c37-week-07/challenge-02/`.
