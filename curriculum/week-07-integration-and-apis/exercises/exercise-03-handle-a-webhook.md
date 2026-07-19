# Exercise 3 — Handle a Webhook

**Goal:** Build a Flask endpoint that receives a simulated payment webhook, verifies its signature, updates an order's status in Postgres, and never processes the same event twice — even if it arrives three times.

**Estimated time:** 90 minutes.

## Setup

You'll play **both roles** in this exercise: the "payment processor" sending webhooks (a small Python script you write) and Crunch Cycles receiving them (a Flask app). This mirrors how you'd test against a real provider's sandbox before going live.

Create the dedup table from Lecture 2:

```sql
CREATE TABLE processed_webhook_events (
    event_id    TEXT PRIMARY KEY,
    event_type  TEXT NOT NULL,
    received_at TIMESTAMP NOT NULL DEFAULT now()
);
```

Confirm your `orders` table has a `status` column you can update (it does, from Week 3/4 — values like `'Pending'`, `'Completed'`, `'Cancelled'`).

## Tasks

**1. Build the receiver.** In `webhook_receiver.py`, write a Flask app with a `POST /webhooks/payments` route that:

- Reads the raw request body and an `X-Signature` header.
- Recomputes the expected HMAC-SHA256 signature using a shared secret (`WEBHOOK_SECRET = "whsec_test_shared_secret"`) and compares with `hmac.compare_digest`.
- Returns `401` if the signature doesn't match.
- Parses the JSON body, expecting `{"id": "<event_id>", "type": "order.paid", "data": {"order_id": <int>}}`.
- Checks `processed_webhook_events` for the `id` — if already present, returns `200` with a body noting it's a duplicate, **without** touching `orders`.
- If new: updates `orders.status = 'Completed'` for the given `order_id`, inserts the event into `processed_webhook_events`, and returns `200`.
- Returns `404` if the referenced `order_id` doesn't exist in `orders`.

**2. Build the sender.** In `send_webhook.py`, write a script that:

- Constructs a JSON payload with a unique `id` (use `uuid.uuid4()`), `type: "order.paid"`, and a real `order_id` from your `orders` table.
- Computes the HMAC signature the same way the receiver expects.
- `POST`s it to `http://localhost:5000/webhooks/payments` with the `X-Signature` header set.
- Prints the response status code and body.

**3. Prove the happy path.** Start the receiver (`python3 webhook_receiver.py`), run the sender once, and confirm:
   - The response is `200`.
   - `SELECT status FROM orders WHERE order_id = <the one you used>;` now shows `'Completed'`.

**4. Prove the duplicate-safety.** Run the **exact same** sender invocation again (same `event_id` — don't regenerate the UUID; hard-code it for this step, or capture it from the first run) and confirm:
   - The response is still `200`, but the body says something like `"duplicate, ignored"`.
   - The order's `status` and `processed_webhook_events` row count are **unchanged** from step 3 — the second call had zero effect beyond the log.

**5. Prove the signature check works.** Modify `send_webhook.py` (or write a quick one-off) to send a request with a wrong or missing `X-Signature` header. Confirm you get `401`, and confirm `orders.status` is **not** touched — an unverified request must never reach your update logic.

## Expected result (spot checks)

- First send: `200`, order status flips to `Completed`.
- Second send (same event ID): `200`, order status **unchanged**, no new row in `processed_webhook_events`.
- Bad signature: `401`, no database change at all.

## Done when…

- [ ] `webhook_receiver.py` verifies the signature **before** parsing or acting on the body.
- [ ] Sending the same event twice is provably a no-op the second time (you checked the row count, not just "it seemed fine").
- [ ] A bad signature never reaches the part of the code that updates `orders`.
- [ ] You can explain, in one sentence, why the dedup check needs to be backed by the database's `PRIMARY KEY` constraint (with `ON CONFLICT DO NOTHING` on the insert) rather than just an `if already_processed(...)` check in Python — think about what happens if two identical requests arrive at nearly the same instant.

## Stretch

- Time how long your handler takes to respond (`time.time()` before/after in `send_webhook.py`). If a real payment processor times out receivers slower than ~10–30 seconds, is your handler comfortably under that, or are you doing anything in the request path that shouldn't block the response? Write one sentence on what you'd move to a background job if `mark_order_paid` also had to send a confirmation email.
- Add a second event type, `order.refunded`, that sets `status = 'Cancelled'` — practice branching on `event_type` cleanly instead of hard-coding one behavior.

## Submission

Commit `webhook_receiver.py`, `send_webhook.py`, and a short `proof.md` (your three spot-check results, pasted from the terminal) to your portfolio under `c37-week-07/exercise-03/`.
