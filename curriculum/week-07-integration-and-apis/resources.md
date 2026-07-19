# Week 7 — Resources

Free, public, no signup unless noted. Read the "required" set; treat the rest as reference you dip into when a specific question comes up.

## Install first

- **Python 3.10+** with `pip` — you already have this from earlier weeks. Confirm: `python3 --version`.
- **Flask** — the web framework for the API and webhook receiver this week: `pip install flask` · docs: <https://flask.palletsprojects.com/>
- **requests** — the HTTP client for consuming APIs: `pip install requests` · docs: <https://requests.readthedocs.io/>
- **pandas / SQLAlchemy / psycopg2-binary** — same ETL toolkit from Week 4, reused: `pip install pandas sqlalchemy psycopg2-binary`
- **python-dotenv** — for loading API keys/secrets from a local `.env` file instead of hard-coding them: `pip install python-dotenv`
- **ngrok (optional, for real third-party webhook testing)** — exposes `localhost` at a temporary public URL: <https://ngrok.com/download>. Not required for this week's exercises (you simulate both sender and receiver yourself), but essential the first time you integrate with a real external webhook sender.
- **`curl`** — you likely already have it (`curl --version`); it's the fastest way to poke an API by hand without writing a script.

## Required reading (this week's core)

- **MDN — HTTP response status codes:** <https://developer.mozilla.org/en-US/docs/Web/HTTP/Status>
  *Why: the canonical, complete reference for every status code — bookmark this, you'll use it every week from now on.*
- **MDN — An overview of HTTP:** <https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview>
  *Why: if HTTP verbs, headers, or the request/response cycle feel fuzzy, this is the clearest short primer.*
- **Stripe — Idempotent Requests:** <https://stripe.com/docs/api/idempotent_requests>
  *Why: the industry-standard explanation of the idempotency-key pattern, from the company that popularized it for payment APIs — directly informs Lecture 3 and the mini-project.*
- **Flask — Quickstart:** <https://flask.palletsprojects.com/en/latest/quickstart/>
  *Why: everything you need to build the routes in this week's lectures and exercises, in official-docs form.*

## Reference (keep in tabs)

- **Frankfurter API docs** (the free exchange-rate API used throughout this week): <https://www.frankfurter.dev/>
  *Why: exact parameter names, date-range syntax, and available currencies for the ETL lectures/exercises.*
- **JSONPlaceholder** (fake REST API for pagination/CRUD practice): <https://jsonplaceholder.typicode.com/>
  *Why: a safe, free sandbox to practice consuming and paginating through an API with no rate limits or auth to worry about.*
- **requests — Advanced usage (timeouts, sessions, retries):** <https://requests.readthedocs.io/en/latest/user/advanced/>
  *Why: the official reference for `timeout`, `Session`, and the built-in `HTTPAdapter` retry configuration.*
- **PostgreSQL — `INSERT ... ON CONFLICT`:** <https://www.postgresql.org/docs/current/sql-insert.html#SQL-ON-CONFLICT>
  *Why: the exact syntax reference for the upsert pattern used in every ETL example this week.*
- **Microsoft REST API Guidelines:** <https://github.com/microsoft/api-guidelines>
  *Why: a thorough, opinionated, freely-readable real-world API design guide — useful when your own design choices (Exercise 1, the mini-project) need a second opinion.*
- **Stripe API Reference** (read as a design example, not for the payments features): <https://stripe.com/docs/api>
  *Why: widely considered one of the best-documented REST APIs in the industry — read a few pages purely to study *how* they document an endpoint.*

## On webhooks specifically

- **Svix — Webhooks: The Missing Guide:** <https://www.svix.com/guides/webhooks-101/webhooks-guide/>
  *Why: a free, vendor-written but even-handed deep dive into webhook design, signature verification, retries, and delivery guarantees.*
- **Stripe — Webhook signatures:** <https://stripe.com/docs/webhooks/signatures>
  *Why: shows exactly how a real production system computes and verifies HMAC signatures — the pattern Lecture 2's receiver is modeled on.*

## Practice beyond this week's exercises

- **Public APIs (curated list, free tier heavy):** <https://github.com/public-apis/public-apis>
  *Why: hundreds of free APIs to practice "consuming a public API" against, once you've exhausted Frankfurter and JSONPlaceholder — used in Homework Problem 2.*
- **httpbin** (an API that exists purely to test HTTP clients — echoes headers, simulates status codes, delays, etc.): <https://httpbin.org/>
  *Why: `httpbin.org/status/429` or `httpbin.org/delay/5` let you deliberately trigger the exact failure conditions Lecture 2's retry code is built to handle, without needing a flaky real API.*

## Deeper background (optional this week)

- **Roy Fielding's dissertation, Chapter 5 — "Representational State Transfer (REST)"** — the original source of the term, from the person who coined it: <https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm>
  *Why: dense but foundational; read it once you're comfortable with the practical patterns, to understand where they came from.*
- **"Enterprise Integration Patterns" (Hohpe & Woolf) — summary site:** <https://www.enterpriseintegrationpatterns.com/>
  *Why: the classic catalog of integration patterns (point-to-point, hub, message queues, and more) — the book itself isn't free, but the site's pattern summaries are, and they go well beyond what this week covers.*

## Glossary

| Term | Definition |
|------|------------|
| **REST** | An architectural style for APIs built on resources (nouns), HTTP verbs, statelessness, and standard status codes. |
| **Resource** | A noun-addressed entity in an API (e.g., `/orders/482`) — the thing a URL points at. |
| **Idempotent** | Calling an operation once has the same end effect as calling it many times (`GET`, `PUT`, `DELETE` are; plain `POST` is not). |
| **Status code** | The 3-digit HTTP response code stating what happened (`200` OK, `404` Not Found, `429` Too Many Requests, etc.). |
| **API versioning** | A strategy (e.g., `/api/v1/`) for changing an API's contract without breaking existing consumers. |
| **API key** | A secret string a caller sends to authenticate server-to-server requests. |
| **OAuth2** | A standard for *delegated* access — a user grants a third-party limited access without sharing their password. |
| **Pagination** | Returning results in pages instead of all at once; offset/limit or cursor-based are the two common styles. |
| **Rate limit** | A cap on how often a client may call an API in a given time window, enforced via `429` responses. |
| **Exponential backoff** | Waiting progressively longer between retries (1s, 2s, 4s, …) after a failure, to avoid overwhelming a struggling server. |
| **Webhook** | An HTTP `POST` a service sends *to you* the instant an event happens, instead of you polling for it. |
| **HMAC signature** | A cryptographic hash of a payload plus a shared secret, used to verify a webhook (or any request) genuinely came from the claimed sender and wasn't tampered with. |
| **ETL / ELT** | Extract-Transform-Load (transform before loading) vs. Extract-Load-Transform (transform after loading, inside the destination). |
| **Upsert** | An insert that updates the existing row instead of failing or duplicating, when a matching key already exists (`ON CONFLICT ... DO UPDATE`). |
| **Idempotency key** | A unique value a *client* generates and sends with a non-idempotent request (like `POST`), so the server can detect and safely ignore an accidental retry. |
| **Point-to-point integration** | Each system connects directly to each other system it needs, with no central intermediary. |
| **Hub-and-spoke integration** | Every system connects to one central hub, which routes and translates between them. |
| **Dead-letter table/queue** | A place failed records are written instead of being silently dropped, so nothing vanishes without a trace. |

---

*Broken link? Open an issue or PR.*
