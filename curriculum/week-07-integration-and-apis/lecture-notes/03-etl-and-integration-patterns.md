# ETL & Integration Patterns — Batch vs. Streaming, Point-to-Point vs. Hub

You now know how to expose an API (Lecture 1) and how to consume one, including reacting to webhooks (Lecture 2). This lecture is about the wiring pattern that turns those individual skills into a system: how data actually moves between Crunch Cycles and the outside world, on a schedule, safely, and in a shape everyone can reason about.

## ETL vs. ELT

Both acronyms describe the same three steps in different order:

- **Extract** — pull raw data out of a source system (a third-party API, another database, a file drop).
- **Transform** — clean it, reshape it, join it with other data, compute derived fields.
- **Load** — write the result into the destination system.

**ETL** transforms *before* loading — the data lands in the destination already clean. This is the pattern this course uses: pull exchange rates from Frankfurter, compute what you need in pandas, then write a tidy `fx_rates` table to Postgres. It's simpler to reason about and matches a small team without a dedicated data-warehouse layer.

**ELT** loads the raw data first, then transforms it *inside* the destination (often with SQL views or a tool like dbt) — common at larger companies where storage is cheap, the raw source-of-truth copy has independent value, and many different transformations get built on top of the same raw landing zone over time. You'll recognize ELT if you ever see a "raw" schema and a "staging" schema in the same warehouse feeding a "marts" schema of clean, ready-to-query tables.

For Crunch Cycles' scale, **ETL** is the right default: transform in Python/pandas where you have full programming-language power, then load a small, clean, purpose-built table. Know that ELT exists and why bigger organizations reach for it — you may meet it at a job even if you don't build it here.

## Batch vs. streaming

**Batch** integration runs on a schedule — every hour, every night at 2 a.m., every Monday morning — and processes everything that changed since last time in one pass. It's simple, easy to debug (you can just re-run it), and the right default for anything that doesn't need to be instant: nightly exchange-rate refreshes, weekly supplier catalog syncs, end-of-day reconciliation reports.

**Streaming** (or **event-driven**) integration reacts the moment something happens — a webhook firing when a payment clears is streaming integration, even though the mechanism (a single HTTP `POST`) is simple. True high-volume streaming (Kafka, Kinesis) is built for continuous, high-throughput event flow and is out of scope for this course, but the *concept* — react to an event instead of waiting for the next scheduled check — is exactly what your webhook receiver from Lecture 2 already does.

The deciding question: **does the business actually need this data faster than the next scheduled batch would provide it?** Exchange rates for financial reporting: batch, once a day, is fine — nobody needs FX rates to the minute for a quarterly report. A customer's payment confirming: streaming, because a customer staring at a spinner for five minutes waiting for the next hourly batch job is a bad experience and, at some order volume, a lost sale. Don't reach for streaming by default — it's more infrastructure, more failure modes, and more to monitor. Use it where the latency genuinely matters.

## Point-to-point vs. hub-and-spoke

Imagine Crunch Cycles eventually integrates with five systems: a payment processor, a shipping carrier, an accounting package, a marketing email tool, and a returns/warranty vendor.

**Point-to-point** means each system talks directly to each other system it needs data from — five systems can mean up to ten separate direct connections (n(n-1)/2), each with its own auth, its own retry logic, its own quirks to remember. It's the natural starting shape (you build the first integration point-to-point almost by default) and it's fine for two or three integrations. It stops being fine past that: every new system is now a new one-off connection to build and maintain, and nobody has a single place to see "everything Crunch Cycles is integrated with."

**Hub-and-spoke** (also called an integration layer, or in bigger shops, an "iPaaS" — integration platform as a service) puts one central system in the middle. Every other system talks *only* to the hub; the hub handles translating, routing, and retrying to each spoke. Adding a sixth system means one new connection to the hub, not five new point-to-point links. The Crunch Cycles REST API from Lecture 1, once other systems (warehouse, accounting) start calling it instead of hitting Postgres directly, *is* the beginning of a hub — every consumer goes through one contract instead of touching the database five different ways.

| | Point-to-point | Hub-and-spoke |
|---|---|---|
| Connections for n systems | Up to n(n−1)/2 | n |
| Good for | 2–3 integrations, moving fast | 4+ integrations, long-term maintainability |
| Single point of failure? | No (but many small points) | Yes — the hub must be reliable |
| Who owns the contract | Every pair, separately | The hub, once |

Neither is "correct" in isolation — point-to-point is the right call for a two-system integration you need working this week; hub-and-spoke is the right call once you're the fourth or fifth integration deep and copy-pasted retry logic is showing up in every script.

## Building the ETL job: FX rates into Postgres

Putting Lecture 2's API-consuming code and this lecture's transform/load pattern together — a real, runnable job:

**1. Create the destination table.**

```sql
CREATE TABLE fx_rates (
    rate_date     DATE    NOT NULL,
    base_currency TEXT    NOT NULL,
    quote_currency TEXT   NOT NULL,
    rate          NUMERIC NOT NULL,
    fetched_at    TIMESTAMP NOT NULL DEFAULT now(),
    PRIMARY KEY (rate_date, base_currency, quote_currency)
);
```

The composite primary key `(rate_date, base_currency, quote_currency)` is the load-side idempotency mechanism — see below.

**2. Extract, transform, load.**

```python
# etl_fx_rates.py
import logging
import requests
import pandas as pd
from sqlalchemy import create_engine, text
from datetime import date

logging.basicConfig(level=logging.INFO, format="%(asctime)s %(levelname)s %(message)s")
log = logging.getLogger("etl_fx_rates")

engine = create_engine("postgresql://localhost/crunchcycles")
SYMBOLS = ["EUR", "JPY", "GBP", "BRL", "CAD"]


def extract(base="USD", symbols=SYMBOLS) -> dict:
    """Pull today's rates from Frankfurter. Raises on failure — caller decides what to do."""
    resp = requests.get(
        "https://api.frankfurter.dev/v1/latest",
        params={"base": base, "symbols": ",".join(symbols)},
        timeout=10,
    )
    resp.raise_for_status()
    return resp.json()


def transform(raw: dict) -> pd.DataFrame:
    """Reshape Frankfurter's {currency: rate} dict into tidy rows."""
    rows = [
        {
            "rate_date": raw["date"],
            "base_currency": raw["base"],
            "quote_currency": currency,
            "rate": rate,
        }
        for currency, rate in raw["rates"].items()
    ]
    df = pd.DataFrame(rows)
    df["rate_date"] = pd.to_datetime(df["rate_date"]).dt.date
    df["rate"] = df["rate"].astype(float)
    return df


def load(df: pd.DataFrame):
    """Upsert — safe to run twice for the same date without creating duplicates."""
    with engine.begin() as conn:
        for _, row in df.iterrows():
            conn.execute(
                text("""
                    INSERT INTO fx_rates (rate_date, base_currency, quote_currency, rate)
                    VALUES (:rate_date, :base_currency, :quote_currency, :rate)
                    ON CONFLICT (rate_date, base_currency, quote_currency)
                    DO UPDATE SET rate = EXCLUDED.rate, fetched_at = now()
                """),
                row.to_dict(),
            )


def run():
    log.info("ETL run starting for %s", date.today())
    try:
        raw = extract()
    except requests.exceptions.RequestException as e:
        log.error("Extract failed: %s", e)
        raise  # let the scheduler / caller know this run failed — don't silently swallow it

    df = transform(raw)
    log.info("Transformed %d rate rows for %s", len(df), df["rate_date"].iloc[0])

    load(df)
    log.info("Load complete — %d rows upserted into fx_rates", len(df))


if __name__ == "__main__":
    run()
```

Run it:

```bash
python3 etl_fx_rates.py
```

```
2024-11-08 09:00:01 INFO ETL run starting for 2024-11-08
2024-11-08 09:00:01 INFO Transformed 5 rate rows for 2024-11-08
2024-11-08 09:00:02 INFO Load complete — 5 rows upserted into fx_rates
```

Run it **again**, right now, with no other changes:

```bash
python3 etl_fx_rates.py
```

Same log lines, same 5 rows — but this time `ON CONFLICT ... DO UPDATE` matched the existing `(rate_date, base_currency, quote_currency)` rows and updated them in place instead of inserting duplicates. `SELECT COUNT(*) FROM fx_rates` after two runs equals `SELECT COUNT(*) FROM fx_rates` after one run. **That is what idempotent means, made concrete** — you can prove it with a row count.

## Why idempotency is the whole game

Every failure mode in integration — a network timeout mid-request, a scheduler that fires the same job twice because of a misconfigured cron, a webhook retried because your `200` got lost on the way back — resolves to the same question: *what happens if this runs twice?* If the honest answer is "duplicate rows" or "double-charged customer," the system is fragile no matter how well everything else is built. If the answer is "nothing changes the second time," the system tolerates the messiness of real networks by design.

Three idempotency techniques, in increasing order of how much they lean on the database rather than application logic:

1. **Natural keys + upsert** — what `fx_rates` uses above. If the data has a unique real-world identity (this currency pair, on this date), make that the primary key and always `ON CONFLICT DO UPDATE`/`DO NOTHING`.
2. **Idempotency keys** — for operations with no natural key (like "create an order"), the *client* generates a unique key and sends it with the request; the server records which keys it has already handled and returns the original result on a repeat, instead of creating a second order. Stripe popularized this pattern for payment APIs — worth reading their write-up (linked in `resources.md`).
3. **Event IDs + a processed-log table** — what the webhook receiver in Lecture 2 uses. The sender assigns each event a unique ID; you keep a table of IDs you've already acted on and check it before doing anything with a side effect.

## Error handling and observability

An ETL job that fails silently is worse than one that fails loudly — at least the loud failure gets fixed. Minimum bar for anything you schedule to run unattended:

- **Log every run's start, row counts, and outcome** (as `etl_fx_rates.py` does above) — when someone asks "did yesterday's FX sync run?", the answer should be a log search, not a guess.
- **Let real failures propagate** (`raise`, don't `except: pass`) so a scheduler or monitoring tool can alert on them. Swallowing an exception to "keep the job green" just moves the failure somewhere harder to find.
- **Distinguish retryable from fatal.** A `429` or a dropped connection: retry with backoff (Lecture 2). A `400` because the API's contract changed underneath you: fail loudly and page a human — retrying won't fix a permanently broken request.
- **Consider a dead-letter table** for rows that fail transform/load even after retries — write them to a `failed_records` table with the error message instead of losing them, so nothing silently vanishes.

## Scheduling

A script that only runs when you remember to run it isn't automation. The simplest scheduler, and a perfectly good one for a job like this:

```bash
# crontab -e
# Run the FX sync every day at 9:00 AM server time
0 9 * * * cd /path/to/project && /path/to/.venv/bin/python etl_fx_rates.py >> logs/etl_fx_rates.log 2>&1
```

For anything more complex — jobs with dependencies on each other, retries with visibility, a UI showing run history — tools like **Apache Airflow** or **Dagster** exist and are worth knowing by name, but a `cron` entry plus good logging is the right amount of infrastructure for a job like this one. Don't reach for a workflow orchestrator to run one Python script once a day.

## What's next

You have every piece: designing an API (Lecture 1), consuming one and handling webhooks (Lecture 2), and wiring extract/transform/load into an idempotent, schedulable job (this lecture). The exercises turn each piece into hands-on practice; the mini-project asks you to combine all three into one working integration, exactly as it would look at a real company.
