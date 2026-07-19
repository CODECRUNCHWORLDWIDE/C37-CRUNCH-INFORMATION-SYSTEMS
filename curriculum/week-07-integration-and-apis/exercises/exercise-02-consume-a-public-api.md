# Exercise 2 — Consume a Public API

**Goal:** Pull real data from a public API into your `crunchcycles` Postgres database using the extract/transform/load pattern from Lecture 3 — with proper timeouts, error handling, and pagination.

**Estimated time:** 90 minutes.

## Setup

Confirm your virtualenv from the week README is active and `requests`, `pandas`, and `sqlalchemy` are installed. You'll use **two** public APIs in this exercise — Frankfurter (no pagination, used in the lectures) and JSONPlaceholder (paginated-in-spirit, used here to practice looping through pages).

Create the destination table:

```sql
CREATE TABLE fx_rates_history (
    rate_date      DATE    NOT NULL,
    base_currency  TEXT    NOT NULL,
    quote_currency TEXT    NOT NULL,
    rate           NUMERIC NOT NULL,
    fetched_at     TIMESTAMP NOT NULL DEFAULT now(),
    PRIMARY KEY (rate_date, base_currency, quote_currency)
);
```

## Tasks

Write a single script, `pull_fx_history.py`, that:

1. **Extracts a date range, not just "latest."** Frankfurter supports a range query:

   ```
   GET https://api.frankfurter.dev/v1/2024-10-01..2024-10-31?base=USD&symbols=EUR,GBP,JPY
   ```

   This returns one `rates` object **per date** in the range. Write an `extract(start_date, end_date, base, symbols)` function that calls this endpoint once (it's not paginated — one call gets the whole range) and returns the raw JSON. Use a `timeout=10` and call `raise_for_status()`.

2. **Transforms into tidy rows.** The response shape is `{"rates": {"2024-10-01": {"EUR": 0.92, ...}, "2024-10-02": {...}, ...}}` — one date per key, not one flat list. Write a `transform(raw)` function using pandas that turns this into a tidy DataFrame with columns `rate_date`, `base_currency`, `quote_currency`, `rate` — one row per (date, currency) pair. *(Hint: loop the outer dict, build a list of row-dicts, then `pd.DataFrame(rows)` — same shape as Lecture 3's `transform`, just one more level of nesting to unwrap.)*

3. **Loads idempotently.** Write a `load(df)` function using the `ON CONFLICT ... DO UPDATE` upsert pattern from Lecture 3, targeting `fx_rates_history`.

4. **Handles a real failure.** Temporarily change the URL to something that 404s (e.g., misspell `frankfurter`) and confirm your script logs a clear error and exits non-zero instead of crashing with an unhandled traceback or, worse, silently writing nothing and reporting success. Then fix the URL back.

5. **Proves idempotency.** Run the script twice in a row for the same date range. Run:

   ```sql
   SELECT COUNT(*) FROM fx_rates_history;
   ```

   before the second run and after. They must be equal.

## Practicing pagination (a second, smaller script)

Frankfurter has no pagination to practice on, so do this part against JSONPlaceholder, which returns all 100 "posts" in one call but is still worth looping defensively — write `fetch_posts_paginated.py`:

```python
import requests

def fetch_all_posts(page_size=10):
    posts, page = [], 1
    while True:
        resp = requests.get(
            "https://jsonplaceholder.typicode.com/posts",
            params={"_page": page, "_limit": page_size},
            timeout=10,
        )
        resp.raise_for_status()
        batch = resp.json()
        if not batch:
            break
        posts.extend(batch)
        page += 1
    return posts
```

Run it, print `len(posts)` (expect **100**), and print the `title` of the 37th post. This is the offset/page-number pagination pattern from Lecture 2 — practice recognizing "empty page = stop" as the loop's exit condition, since not every API tells you the total count up front.

## Expected result (spot checks)

- `fx_rates_history` has `31 * 3 = 93` rows after loading October 2024 for `EUR,GBP,JPY` (31 days × 3 currencies). Adjust if you pick a different month/currency count — the point is the row count matches your math exactly.
- Running `pull_fx_history.py` twice does **not** change the row count the second time.
- `fetch_all_posts()` returns exactly 100 posts.

## Done when…

- [ ] `pull_fx_history.py` has separate `extract`, `transform`, `load` functions (not one giant script).
- [ ] Every `requests` call has a `timeout` and a `raise_for_status()` (or equivalent explicit status check).
- [ ] You can point to the exact line that makes the load idempotent, and explain it in one sentence.
- [ ] `fetch_posts_paginated.py` correctly stops when a page comes back empty, not on a hard-coded page count.

## Stretch

- Add basic logging (`logging.info`) to `pull_fx_history.py` showing row counts extracted, transformed, and loaded — match the style from Lecture 3's `etl_fx_rates.py`.
- Write one SQL query against `fx_rates_history` that shows how EUR/USD moved day-over-day across your date range (a simple `LAG()` window function works — see if Week 6's material or `resources.md` jogs your memory).

## Submission

Commit `pull_fx_history.py`, `fetch_posts_paginated.py`, and a short `notes.md` (your row-count checks) to your portfolio under `c37-week-07/exercise-02/`.
