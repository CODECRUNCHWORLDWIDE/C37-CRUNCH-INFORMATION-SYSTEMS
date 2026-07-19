# Week 7 — Homework

Five problems, ~5 hours total, spread across the week. These reinforce the lectures with a mix of hands-on API work, a written explanation, and small extensions to code you've already built. Commit each.

All problems assume the `crunchcycles` Postgres database and the Python/Flask environment from the [README](./README.md), unless a problem says otherwise.

---

## Problem 1 — Status code audit (45 min)

Take the Flask API skeleton from Lecture 1 (`app.py`) and extend it to cover 8 distinct real-world scenarios, each returning the *correct* status code:

1. `GET /api/v1/orders/999999` where the order doesn't exist → what code?
2. `GET /api/v1/orders` with no `Authorization` header → what code?
3. `GET /api/v1/orders` with a well-formed but wrong API key → what code?
4. `POST /api/v1/orders` with a `customer_id` that doesn't exist → what code (and why, vs. option 5)?
5. `POST /api/v1/orders` with a malformed JSON body (not valid JSON at all) → what code?
6. `POST /api/v1/orders` that succeeds → what code, and what header should accompany it?
7. `DELETE /api/v1/orders/{id}` on an order that's already `Cancelled` → what code (think about `409` vs `400`)?
8. `GET /api/v1/orders?limit=999999` (requesting way more than your server-side cap) → does this error, or silently cap? Which is better API design, and why?

**Deliver** `status-code-audit.md`: for each of the 8, state the code you chose, and one sentence justifying it against Lecture 1's status-code table.

---

## Problem 2 — Consume a real public API you haven't used yet (75 min)

Pick a free, no-signup-required public API you have **not** used this week (not Frankfurter, not JSONPlaceholder). Good options: Open-Meteo (weather, <https://open-meteo.com>), the REST Countries API (<https://restcountries.com>), or PokéAPI (<https://pokeapi.co>) — pick whichever sounds least boring.

Write `explore_new_api.py` that:

1. Makes a `GET` request with a `timeout` and `raise_for_status()`.
2. Loads the result into a pandas DataFrame.
3. Answers one real question with it (e.g., "which of these 10 cities will be warmest tomorrow" for Open-Meteo, or "which countries share a border with Brazil" for REST Countries).

**Deliver** `explore_new_api.py` and `findings.md` (the question, the answer, and one thing that surprised you about the API's response shape or pagination — even "there was no pagination and I had to check the docs to be sure" counts).

---

## Problem 3 — Explain idempotency without code (30 min)

In `idempotency-writeup.md`, answer in prose (no more than 400 words total):

1. In your own words, what does "idempotent" mean? Give one HTTP verb that's naturally idempotent and one that isn't, and explain why the difference matters when a network request fails partway through.
2. Describe a real (or realistic) situation — not from this course — where a **lack** of idempotency caused a visible problem (a duplicate charge, a duplicate order, a doubled email). If you can't think of a real one, invent a plausible one and explain exactly where the missing safeguard should have been.
3. Explain the difference between the three idempotency techniques from Lecture 3 (natural-key upsert, client-generated idempotency key, event-log dedup) in one sentence each, and say which one fits "processing a webhook" best and why.

---

## Problem 4 — Rate-limit yourself, on purpose (45 min)

Write `self_rate_limiter.py` that wraps calls to Frankfurter in a client-side rate limiter — even though Frankfurter doesn't strictly enforce one, practice being a *good citizen* API consumer:

1. Write a function that allows at most 1 call per 2 seconds (track the last call time; `time.sleep()` if called again too soon).
2. Fire off 5 requests for 5 different currency pairs in a loop and print a timestamp before each call, proving they're spaced out.
3. In a 3-sentence comment at the top of the file, explain why a *considerate* API consumer self-throttles even against an API with no visible rate limit — think about what happens if every one of a public API's users assumes "no visible limit" means "call it as fast as possible."

**Deliver** `self_rate_limiter.py` with its printed timestamps in a comment or a `run-output.txt`.

---

## Problem 5 — Extend the webhook receiver (75 min)

Take `webhook_receiver.py` from Exercise 3 and add a second, realistic event type: `order.refund_requested`, which should:

1. Look up the order; if it isn't currently `Completed`, respond `409 Conflict` (a refund can't be requested on an order that was never completed) — **and do not** record the event as processed in this case, since it wasn't successfully handled. *(Think carefully: should a rejected event be retried by the sender, or not? Justify your choice in your writeup.)*
2. If the order is `Completed`, insert a new row into a `refund_requests` table you create (`refund_id SERIAL`, `order_id`, `requested_at`, `status DEFAULT 'pending'`), record the webhook event as processed, and respond `200`.
3. Prove, the same way Exercise 3 did, that resending the same `event_id` after a **successful** refund request is a safe no-op, and that resending after a **rejected** (409) one is *not* silently swallowed — it should still be evaluated fresh each time, since you chose not to record it as processed.

**Deliver** the updated `webhook_receiver.py`, the new table's `CREATE TABLE`, and `refund-flow-proof.md` with your test transcript and one paragraph justifying your choice in step 1.

---

## Time budget

| Problem | Time |
|--------:|-----:|
| 1 | 45 min |
| 2 | 75 min |
| 3 | 30 min |
| 4 | 45 min |
| 5 | 75 min |
| **Total** | **~4.5 h** |

After homework, take the [quiz](./quiz.md) and ship the [mini-project](./mini-project/README.md).
