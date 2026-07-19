# Designing REST APIs — Resources, Verbs, Status Codes, Versioning, and Auth

Every information system eventually needs to talk to another one. Crunch Cycles' warehouse system needs to know when an order is placed. A mobile app needs to show a customer their order history. A partner's accounting software needs nightly revenue totals. You could hand every one of these systems direct SQL access to your Postgres database — and you would regret it within a month: someone runs an unindexed query that locks a table, someone else depends on a column name you need to rename, and now you can never change your schema without breaking three other teams.

An **API** (Application Programming Interface) is the fix: a deliberate, versioned, documented contract that sits between your database and everyone who needs data from it. Change the schema all you want behind the API — as long as the contract holds, nothing outside breaks. This lecture is about designing that contract well, using **REST** (Representational State Transfer), the dominant style for APIs exposed over HTTP.

## Why integration is where systems succeed or fail

A system with beautiful features that can't talk to payroll, the warehouse, or the bank is a system nobody uses at scale — someone re-types the numbers by hand instead, and re-typing is where the real damage happens: typos, missed transactions, no audit trail. Conversely, a system with modest features but a clean, reliable API earns trust fast, because other teams can build on it without fear.

The failure mode is almost never "the API doesn't exist." It's usually one of:

- **Undocumented contracts.** The API "works" but nobody wrote down what a 404 means versus a 200 with an empty list, so every consumer guesses differently.
- **No versioning.** A field gets renamed, every downstream integration breaks at once, on a Friday.
- **No idempotency.** A network blip causes a client to retry a "charge the customer" call, and the customer is charged twice.
- **Silent failures.** A webhook fails to deliver and nobody notices for three weeks, because nothing was watching.

Every lecture this week is aimed at one of those four failure modes. This one is aimed at the first two.

## REST in one paragraph

REST models everything as a **resource** — a noun, addressed by a URL (`/orders/482`) — and everything you can do to it as one of a small set of **HTTP verbs** (`GET`, `POST`, `PUT`/`PATCH`, `DELETE`). The server is **stateless**: each request carries everything needed to understand it (including who's asking), so any server behind a load balancer can handle any request. Responses use standard **HTTP status codes** to say what happened, and a predictable body format (almost always JSON today) to carry the data.

That's it. REST is not a protocol or a library — it's a set of conventions. The value is entirely in how consistently you apply them.

## Naming resources: nouns, not verbs

The single most common REST mistake is putting a verb in the URL. Compare:

| Bad (verb in URL) | Good (noun + HTTP verb) |
|---|---|
| `POST /createOrder` | `POST /orders` |
| `GET /getOrderById?id=482` | `GET /orders/482` |
| `POST /cancelOrder/482` | `DELETE /orders/482` (or `PATCH /orders/482` with `{"status":"cancelled"}`) |
| `POST /listCustomersByRegion` | `GET /customers?region=Europe` |

The URL names a **thing**; the HTTP verb names the **action**. Once you internalize that split, most of your API's shape falls out for free.

**Designing the Crunch Cycles resources.** Walking the schema from Week 3, the natural top-level resources are the entities customers actually think in terms of:

```
/customers
/customers/{customer_id}
/customers/{customer_id}/orders

/products
/products/{product_id}

/orders
/orders/{order_id}
/orders/{order_id}/items
```

Notice `orders` nested under `customers` — `GET /customers/42/orders` reads naturally as "this customer's orders." But `orders` also exists at the top level, because "give me all orders shipped yesterday across every customer" is a real, common question that has nothing to do with one customer. **Nesting is for context, not ownership** — don't force everything into a hierarchy just because the database has foreign keys.

## The HTTP verbs and what they promise

| Verb | Meaning | Idempotent? | Safe (no side effects)? | Crunch Cycles example |
|---|---|---|---|---|
| `GET` | Read a resource | Yes | Yes | `GET /orders/482` |
| `POST` | Create a new resource (server assigns the ID) | **No** | No | `POST /orders` |
| `PUT` | Replace a resource entirely | Yes | No | `PUT /customers/42` (send the *whole* record) |
| `PATCH` | Partially update a resource | Usually treated as idempotent | No | `PATCH /orders/482` `{"status":"shipped"}` |
| `DELETE` | Remove a resource | Yes | No | `DELETE /orders/482` |

**Idempotent** means calling it once has the same effect as calling it five times. This matters enormously for integration, because networks fail: a client sends a request, the response is lost on the way back, and the client — not knowing if it worked — retries. If the operation is idempotent, retrying is safe. If it's not (like `POST`, which creates a *new* resource every time), a naive retry creates duplicate orders. You'll build the fix for this — idempotency keys — in Lecture 3.

A subtlety worth remembering: `GET` must never have side effects. If checking an order's status also happens to decrement inventory, that's a serious design bug — a monitoring tool, a browser prefetch, or a retried request could trigger it accidentally.

## Status codes: the vocabulary of the response

A REST API's status code is not decoration — it's the first thing a well-behaved client reads to decide what to do next. Learn this set cold:

| Code | Name | When to use it |
|---|---|---|
| `200` | OK | A `GET`, `PUT`, or `PATCH` succeeded and there's a body to return. |
| `201` | Created | A `POST` succeeded and created a new resource. Include a `Location` header pointing at it. |
| `204` | No Content | The request succeeded and there's nothing to return (a common `DELETE` response). |
| `400` | Bad Request | The request is malformed — missing a required field, wrong data type. |
| `401` | Unauthorized | No valid credentials were provided at all. |
| `403` | Forbidden | Valid credentials were provided, but they don't grant access to this resource. |
| `404` | Not Found | The resource doesn't exist (or you don't want to reveal that it does — see below). |
| `409` | Conflict | The request conflicts with the resource's current state (e.g., cancelling an already-shipped order). |
| `422` | Unprocessable Entity | The request is well-formed JSON but fails business validation (e.g., `quantity: -5`). |
| `429` | Too Many Requests | The caller is being rate-limited. Include a `Retry-After` header. |
| `500` | Internal Server Error | Something broke on your side. Never leak a stack trace in the body. |
| `503` | Service Unavailable | You're up but temporarily can't serve requests (maintenance, overload). |

A frequent judgment call: `401` vs. `403` vs. `404`. If a customer requests `GET /orders/999` and order 999 belongs to a *different* customer, do you return `403` (it exists, you can't see it) or `404` (pretend it doesn't exist)? Most production APIs choose `404` here deliberately — it leaks no information about what IDs exist, which matters for security. State this choice explicitly in your API docs; don't leave it to be discovered by trial and error.

## Request and response shape

Pick a consistent envelope and never deviate. A reasonable Crunch Cycles convention:

```json
// GET /orders/482 → 200 OK
{
  "order_id": 482,
  "customer_id": 42,
  "status": "shipped",
  "order_date": "2024-11-03",
  "ship_date": "2024-11-06",
  "items": [
    {"product_id": 3, "quantity": 1, "unit_price": 1899.00}
  ]
}
```

```json
// POST /orders with a validation error → 422 Unprocessable Entity
{
  "error": "validation_failed",
  "message": "quantity must be a positive integer",
  "field": "items[0].quantity"
}
```

Two rules that save every consumer of your API real pain:

1. **Errors have a consistent shape too.** A client should be able to write one error-handling code path for your whole API, not one per endpoint.
2. **Dates and numbers are unambiguous.** Use ISO 8601 (`2024-11-03`, or `2024-11-03T14:30:00Z` with timezone for timestamps) and plain JSON numbers for money — never a locale-formatted string like `"$1,899.00"`.

## Pagination, filtering, and sorting — in the URL, not the body

`GET` requests don't carry a body by convention (many servers reject one). Everything that shapes *which* rows come back — filters, sort order, pagination — goes in the query string:

```
GET /orders?status=shipped&region=Europe&sort=-order_date&limit=25&offset=50
```

- `limit`/`offset` — the same pagination you used in SQL in Week 1, exposed at the API level. Cap `limit` server-side (say, 100) so nobody accidentally asks for a million rows in one call.
- `-order_date` — a leading `-` for descending, a common lightweight sort convention.
- Always return a `total_count` (or a `next` link) somewhere in the response so the client knows when to stop paging — you'll rely on this in Lecture 2 when you're the one *consuming* a paginated API.

## Versioning: how you change your mind later

You will need to change this API. A field will need renaming, a resource will need restructuring. Versioning is how you do that without breaking every existing consumer on the day you ship the change.

**URL path versioning** (`/api/v1/orders`, `/api/v2/orders`) is the most common approach because it's visible, cacheable, and impossible to get wrong by accident — you can see the version in every log line and every browser tab. The alternative, a custom header (`Accept: application/vnd.crunchcycles.v2+json`), is more "correct" REST purism but harder to debug and easy for consumers to forget. **This course uses URL path versioning** — start every route with `/api/v1/…` from day one, even before you have a `v2`, so the pattern is already in place when you need it.

The rule that makes versioning bearable: **never make a breaking change inside a version.** Adding a new optional field is fine in `v1`. Renaming or removing a field, changing a type, or changing what a status code means is a `v2`. Consumers on `v1` should never wake up to a surprise.

## Authentication: who's allowed to ask?

Three approaches you'll meet, in order of how much this course expects you to build yourself:

- **API keys** — a long random string the caller sends in a header (`Authorization: Bearer sk_live_abc123...` or a custom `X-API-Key` header). Simple to implement, simple to revoke, good enough for server-to-server integration where you control both ends. **This is what you build in this week's exercises and mini-project.**
- **OAuth2** — the standard for *delegated* access, where a user grants a third-party app limited access to their account without sharing their password (this is how "Sign in with Google" works, and how you'd let a Crunch Cycles customer connect their own accounting software). You'll consume OAuth2-protected APIs in Lecture 2; building an OAuth2 *provider* is out of scope for this course.
- **JWTs (JSON Web Tokens)** — a signed, self-contained token that encodes who the caller is and what they're allowed to do, without the server needing to look anything up. Common for authenticating your own front-end to your own API. Mentioned here so the term isn't a mystery; not required this week.

For the Crunch Cycles API, an API key is the right tool: warehouse systems, accounting software, and internal dashboards are all server-to-server callers you provision directly, not end users clicking "log in."

## Building the skeleton in Flask

Here's the shape of the API you'll extend in the exercises and mini-project — deliberately minimal, showing the pattern rather than every route:

```python
# app.py
from flask import Flask, jsonify, request, abort
from sqlalchemy import create_engine, text
import os

app = Flask(__name__)
engine = create_engine(os.environ["DATABASE_URL"])  # e.g. postgresql://localhost/crunchcycles

API_KEYS = {"sk_warehouse_9f2a", "sk_accounting_7c1b"}  # in real life: a table, hashed

def require_api_key():
    key = request.headers.get("Authorization", "").removeprefix("Bearer ").strip()
    if key not in API_KEYS:
        abort(401, description="missing or invalid API key")

@app.get("/api/v1/orders/<int:order_id>")
def get_order(order_id):
    require_api_key()
    with engine.connect() as conn:
        order = conn.execute(
            text("SELECT * FROM orders WHERE order_id = :id"), {"id": order_id}
        ).mappings().first()
    if order is None:
        abort(404, description=f"order {order_id} not found")
    return jsonify(dict(order)), 200

@app.get("/api/v1/orders")
def list_orders():
    require_api_key()
    limit = min(int(request.args.get("limit", 25)), 100)   # cap it
    offset = int(request.args.get("offset", 0))
    status = request.args.get("status")

    sql = "SELECT * FROM orders"
    params = {}
    if status:
        sql += " WHERE status = :status"
        params["status"] = status
    sql += " ORDER BY order_id LIMIT :limit OFFSET :offset"
    params.update(limit=limit, offset=offset)

    with engine.connect() as conn:
        rows = conn.execute(text(sql), params).mappings().all()
    return jsonify([dict(r) for r in rows]), 200

@app.errorhandler(401)
@app.errorhandler(404)
@app.errorhandler(422)
def handle_error(e):
    return jsonify({"error": e.name.lower().replace(" ", "_"), "message": e.description}), e.code

if __name__ == "__main__":
    app.run(debug=True, port=5000)
```

Run it and test with `curl`:

```bash
export DATABASE_URL=postgresql://localhost/crunchcycles
python3 app.py

# in another terminal:
curl -i http://localhost:5000/api/v1/orders/482 \
  -H "Authorization: Bearer sk_warehouse_9f2a"

curl -i http://localhost:5000/api/v1/orders \
  -H "Authorization: Bearer sk_warehouse_9f2a"   # 401 — no key
```

`curl -i` shows the response headers *and* status line — get in the habit of checking the status code first, body second. It's the fastest way to tell "it worked," "I did something wrong," and "the server broke" apart at a glance.

## Documenting the contract

An API without documentation is an API only its author can use. At minimum, for every endpoint, write down: the method and path, required headers, query parameters, a sample request, a sample success response with its status code, and every error status code it can return and why. Exercise 1 has you produce exactly this document for a new resource — treat it as seriously as the code, because for an API, **the documentation *is* the product** other teams build against.

## What's next

You've designed the *serving* side — the Crunch Cycles API other systems will call. Lecture 2 flips the direction: you become the caller, consuming someone else's API, and the receiver of someone else's events via webhooks.
