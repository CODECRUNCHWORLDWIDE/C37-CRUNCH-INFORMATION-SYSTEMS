# Mini-Project — Integrate Two Systems: a REST API and an ETL Pipeline

> Build the two halves of real-world integration for Crunch Cycles: a small REST API that exposes the Postgres store to other systems, and an ETL job that pulls transformed data from a third-party API into that same store — with error handling and idempotency throughout. This is the week's capstone: everything from Lectures 1–3 and all three exercises, combined into one working system.

**Estimated time:** 2.5–3 hours, best done Saturday after the exercises and challenges.

By now you can design a REST endpoint, call someone else's API defensively, receive a webhook safely, and write an idempotent ETL job. A real integration engineer's day is some mix of all four, usually on the same ticket. This project asks you to build a small but complete slice of that: **Crunch Cycles exposing itself to the world, and Crunch Cycles pulling from the world** — both directions, both done right.

---

## Deliverable

A directory in your portfolio, `c37-week-07/mini-project/`, containing:

1. `api.py` — a Flask REST API exposing at least the endpoints listed below.
2. `etl_fx_pipeline.py` — an idempotent ETL job pulling exchange rates from a public API into Postgres.
3. `API.md` — documentation of every endpoint (method, path, auth, request/response, status codes) — the same rigor as Exercise 1's spec, but for the API you actually built.
4. `run-log.md` — a transcript proving the system works: `curl` calls against your API with their real responses, and two consecutive runs of the ETL job with their row counts.
5. `README.md` — how to set up and run everything (dependencies, environment variables, commands), written so a stranger could get it running in under 10 minutes.

Everything runs against your `crunchcycles` Postgres database from Weeks 3–6, plus one new table you'll create for the FX data.

---

## Part 1 — The REST API

Build `api.py` exposing, at minimum:

| Method | Path | Purpose |
|---|---|---|
| `GET` | `/api/v1/customers` | List customers, paginated (`limit`/`offset`), filterable by `region_id` |
| `GET` | `/api/v1/customers/{customer_id}` | Get one customer; `404` if not found |
| `GET` | `/api/v1/orders` | List orders, paginated, filterable by `status` |
| `GET` | `/api/v1/orders/{order_id}` | Get one order **with its line items** (join `orders` + `order_items` + `products`) |
| `POST` | `/api/v1/orders` | Create a new order with at least one line item |
| `GET` | `/api/v1/fx-rates/latest` | Return the most recent exchange rates loaded by your ETL job (Part 2) |

Requirements, tying together every lecture:

- **Auth.** Every endpoint requires an API key via `Authorization: Bearer <key>` (Lecture 1). Return `401` for a missing/invalid key.
- **Status codes.** `200`/`201`/`400`/`401`/`404`/`422` all appear somewhere and are used correctly — not just `200` and `500` for everything.
- **Validation.** `POST /api/v1/orders` must validate: `customer_id` exists (else `404` or `422` — pick one and justify it in `API.md`), at least one line item is present, every `quantity` is a positive integer, every `product_id` exists. Return `422` with a clear error message for a validation failure — not a raw database error or a `500`.
- **Idempotency key on order creation.** Accept an optional `Idempotency-Key` header on `POST /api/v1/orders`. If the same key is sent twice, return the **original** order (with its original `201` — or `200`, your call, documented) instead of creating a duplicate order. *(This is the client-generated idempotency key pattern from Lecture 3, applied to the one non-idempotent verb in your API.)*
- **Pagination.** `limit` capped server-side at 100; both list endpoints support it.

---

## Part 2 — The ETL Pipeline

Build `etl_fx_pipeline.py`, extending the pattern from Lecture 3:

- **Extract** the latest exchange rates for at least 4 currencies from a free public API (Frankfurter, as in the lectures, or another no-signup API you can justify).
- **Transform** into a tidy table matching a schema you design and document in `API.md` or a short `SCHEMA.md` — reuse or extend the `fx_rates` table from Lecture 3.
- **Load** idempotently — running the job twice in a row must not change the row count the second time. Prove this in `run-log.md`.
- **Handle failure.** Wrap the extract call with a timeout and `raise_for_status()`; add retry-with-backoff for transient failures (429/5xx/connection errors) per Lecture 2; let genuine, non-retryable failures (like a malformed API response) raise loudly with a clear log message rather than fail silently.
- **Log every run** — start, rows extracted, rows loaded, outcome — in the style of Lecture 3's `etl_fx_rates.py`.
- Wire `GET /api/v1/fx-rates/latest` (from Part 1) to read from the table this job populates, so the API and the ETL job are provably part of the same system, not two disconnected demos.

---

## Milestones

- **Milestone 1 (60 min):** `GET` endpoints for customers and orders working, with auth and pagination. Test each with `curl`.
- **Milestone 2 (45 min):** `POST /api/v1/orders` with full validation and the idempotency-key behavior. Prove duplicate submission doesn't duplicate the order.
- **Milestone 3 (45 min):** `etl_fx_pipeline.py` extracting, transforming, and loading idempotently, with retry and logging.
- **Milestone 4 (30 min):** Wire `GET /api/v1/fx-rates/latest`, write `API.md` and `run-log.md`, do a clean end-to-end run and capture the transcript.

---

## Rules

- **No unauthenticated endpoints** except a `GET /api/v1/health` you may optionally add for your own testing convenience — document if you add it.
- **Every status code you use must be correct**, not just "close enough" — re-check against Lecture 1's table before you commit.
- **The ETL job must be re-runnable.** If running it twice produces a different row count than running it once, the project isn't done.
- **No secrets committed.** API keys (yours and Crunch Cycles') live in a `.env` file, `.gitignore`'d.

---

## Rubric

| Criterion | Weight | "Great" looks like |
|-----------|------:|---------------------|
| API correctness | 25% | Every endpoint returns the right shape and the right status code in success and failure cases |
| Auth & validation | 20% | No endpoint is reachable without a key; `POST` validates thoroughly and returns `422` with a useful message |
| Idempotency (API) | 15% | Duplicate `POST` with the same `Idempotency-Key` provably does not create a second order |
| ETL correctness | 20% | Extract/transform/load are cleanly separated; the job is genuinely re-runnable, proven with row counts |
| Error handling | 10% | Timeouts, `raise_for_status`, and retry-with-backoff are present and used only where appropriate |
| Documentation | 10% | `API.md` and `README.md` let a stranger run the whole system in under 10 minutes |

---

## Reflection (`notes.md`, ~250 words)

1. Where did you have to make a judgment call the spec didn't fully pin down (e.g., `404` vs `422` for a missing `customer_id`)? What did you decide, and why?
2. What's the weakest point in your system's resilience right now — the thing most likely to break in production that your time budget didn't let you harden?
3. If Crunch Cycles added a second third-party integration next month, would you keep this point-to-point, or start building toward a hub? What would tip the decision?
4. Which idempotency technique (natural-key upsert, idempotency key, or event-log dedup) did you find hardest to get right, and what tripped you up?

---

## Why this matters

This is the shape of a real integration engineer's sprint: expose an internal system safely to the outside, and pull the outside world in without letting a bad network turn into bad data. Every skill from this week shows up here at once — resource design, auth, validation, pagination, retries, and idempotency — because in production they're never separable. Keep this project; Week 8 builds a reporting layer on top of exactly this data.

When done: push, then take the [quiz](../quiz.md) and start [Week 8 — Reporting, analytics & dashboards](../../week-08-reporting-analytics-and-dashboards/).
