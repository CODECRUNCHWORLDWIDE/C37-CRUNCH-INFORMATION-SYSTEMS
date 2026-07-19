# Week 7 — Quiz

Fifteen questions. Lectures closed. Aim for 13/15 before starting Week 8. A mix of multiple-choice and short "what happens here?" — the answer key at the bottom explains the *why*, not just the letter.

---

**Q1.** In REST, the correct way to design an endpoint that cancels order 482 is:

- A) `POST /cancelOrder/482`
- B) `GET /orders/482/cancel`
- C) `DELETE /orders/482` or `PATCH /orders/482` with a status change
- D) `POST /orders/482/actions/cancel-this-order-please`

---

**Q2.** Which HTTP verb is **not** idempotent — calling it twice can create two different results instead of the same one?

- A) `GET`
- B) `PUT`
- C) `POST`
- D) `DELETE`

---

**Q3.** A client requests `GET /orders/482`, which belongs to a different customer than the one making the request. Many production APIs deliberately return which status code, and why?

- A) `403 Forbidden`, because it's the technically correct code
- B) `404 Not Found`, so as not to reveal that an order with that ID exists at all
- C) `401 Unauthorized`, because the customer needs to log in again
- D) `400 Bad Request`, because the ID is malformed

---

**Q4.** `POST /api/v1/orders` is missing a required `customer_id` field entirely. What's the correct status code?

- A) `500 Internal Server Error`
- B) `404 Not Found`
- C) `400 Bad Request` (malformed request — a required field is absent)
- D) `200 OK` with an error message in the body

---

**Q5.** Why does this course recommend **URL path versioning** (`/api/v1/...`) over a custom `Accept` header for versioning?

- A) It's the only method HTTP supports
- B) It's visible in every log line and URL, and harder to get wrong by accident
- C) Header-based versioning is deprecated by the HTTP spec
- D) URL versioning doesn't require a version number at all

---

**Q6.** Why must every `requests.get(...)` call in production code include an explicit `timeout`?

- A) It's required by the `requests` library or it throws immediately
- B) Without one, a hung connection can block the call indefinitely, stalling the whole pipeline
- C) It's only needed for `POST` requests, not `GET`
- D) It has no effect — timeouts are configured server-side only

---

**Q7.** A third-party API returns `429 Too Many Requests` with a `Retry-After: 30` header. The correct response is:

- A) Immediately retry as fast as possible — every failed call costs momentum
- B) Give up and never call this API again
- C) Wait at least the indicated time (and generally back off further on repeated failures) before retrying
- D) Switch to `POST` instead of `GET`, which has no rate limit

---

**Q8.** Which of these errors should generally **not** be retried automatically?

- A) `503 Service Unavailable`
- B) A `requests.exceptions.ConnectionError`
- C) `429 Too Many Requests`
- D) `400 Bad Request` — the request itself is malformed and retrying won't fix it

---

**Q9.** What is the primary reason a webhook receiver should verify a signature (e.g., HMAC) before acting on the payload?

- A) It's required by the HTTP specification for all `POST` requests
- B) Anyone on the internet can `POST` to a public URL and claim to be the real sender
- C) It makes the JSON parse faster
- D) It's only needed if the payload contains money-related data

---

**Q10.** A webhook receiver takes 45 seconds to fully process an incoming event (including sending a confirmation email) before responding. What's the likely consequence?

- A) None — the sender will wait as long as needed
- B) The sender may time out and consider the delivery failed, triggering a retry — possibly processing the same event twice
- C) The event will be silently dropped with no retry
- D) The receiver will automatically batch the next event

---

**Q11.** In `ON CONFLICT (rate_date, base_currency, quote_currency) DO UPDATE SET rate = EXCLUDED.rate`, what does `EXCLUDED` refer to?

- A) The existing row already in the table before this statement ran
- B) The row that *would have been* inserted, had there not been a conflict — i.e., the new incoming values
- C) A special table of rows that failed validation
- D) The primary key columns only

---

**Q12.** Which statement about ETL vs. ELT is correct?

- A) ETL transforms data before loading it; ELT loads raw data first and transforms it inside the destination
- B) ELT is always faster than ETL
- C) ETL and ELT are the same thing, just different names
- D) ELT never needs a transform step at all

---

**Q13.** An ETL job runs successfully, then is accidentally run a second time by a misconfigured cron entry. In a properly idempotent job, the second run should:

- A) Fail loudly with an error, since it already ran today
- B) Insert a second copy of every row
- C) Produce the exact same end state in the destination table as running it once — no row-count change
- D) Delete all previously loaded data and start fresh

---

**Q14.** For n systems all needing to exchange data with each other, point-to-point integration can require up to how many separate connections, compared to hub-and-spoke?

- A) Point-to-point: n(n−1)/2 connections; hub-and-spoke: n connections
- B) They require the same number of connections either way
- C) Point-to-point always requires fewer connections
- D) Hub-and-spoke requires n² connections

---

**Q15.** Why is "never rely on a webhook alone for anything that matters" good practice, and what's the standard mitigation?

- A) Webhooks are inherently insecure and should never be used
- B) Webhook delivery can be lost due to network issues on either side; pair it with a periodic reconciliation/polling job that catches anything the webhook missed
- C) Webhooks are slower than polling, so polling should always be preferred
- D) There is no mitigation — this is an unsolvable problem

---

## Answer key

<details>
<summary>Reveal after attempting</summary>

1. **C** — REST resources are nouns; the verb belongs to HTTP, not the URL. `DELETE` or a `PATCH` status-change both correctly express "cancel" without a verb baked into the path.
2. **C** — `POST` creates a new resource each time by design; calling it twice with the same body typically creates two resources. `GET`/`PUT`/`DELETE` are idempotent — repeating them leaves the system in the same state as doing it once.
3. **B** — many APIs choose `404` over `403` here deliberately, to avoid leaking whether a resource with that ID exists at all — a small but real information-security consideration.
4. **C** — `400 Bad Request` for a structurally malformed/incomplete request. (Note: some APIs use `422` for this specific case — the important thing is picking one convention and being consistent and documented, as Lecture 1 stresses.)
5. **B** — URL path versioning is visible in logs, URLs, and browser tabs, and near-impossible to omit by accident, unlike a header a client can simply forget to set.
6. **B** — without a timeout, a slow or hung server can block the calling code indefinitely, and one bad connection can stall an entire pipeline.
7. **C** — respect `Retry-After` (or back off exponentially if absent) rather than hammering the API immediately, which only makes the underlying problem worse for you and everyone else calling it.
8. **D** — a `400` means the request itself is wrong; retrying the identical malformed request will produce the identical `400` every time. Only transient failures (5xx, 429, connection errors) are worth retrying.
9. **B** — a public webhook URL is, by definition, reachable by anyone; the signature is the only proof the request actually came from the claimed sender and wasn't forged or tampered with in transit.
10. **B** — most webhook senders enforce a response timeout and treat "no timely response" as a failed delivery, which they retry — meaning a slow handler both risks timing out *and* risks having to handle the same event again.
11. **B** — `EXCLUDED` is Postgres's name, inside an `ON CONFLICT` clause, for the row values that were about to be inserted before the conflict was detected — i.e., the new data from this run.
12. **A** — this is the core distinction: ETL transforms in flight before writing to the destination; ELT lands the raw data first and transforms it afterward, typically inside the destination system.
13. **C** — that is the definition of an idempotent job in practice: the end state after running it N times is identical to the end state after running it once.
14. **A** — point-to-point connections scale roughly with the square of the number of systems (n(n−1)/2 in the worst case), while a hub-and-spoke design scales linearly (one connection per system, to the hub).
15. **B** — webhook delivery is not guaranteed (networks fail on either end); pairing it with a scheduled reconciliation/polling sweep catches anything the webhook silently missed, trading a little latency for real correctness.

</details>

**Scoring:** 13+ → start Week 8. 10–12 → re-read the lecture sections behind your misses, especially status codes and idempotency. <10 → re-read all three lectures from the top; this week's concepts are the foundation for every later integration you'll build.
