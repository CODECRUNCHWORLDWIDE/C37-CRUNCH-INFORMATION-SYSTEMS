# Exercise 1 — Design an API

**Goal:** Design and document a small REST API for a new Crunch Cycles resource — `returns` — before writing a line of code. A good API design is a document another developer could implement without asking you a single follow-up question.

**Estimated time:** 90 minutes.

## Setup

No install needed for this one — it's a design document, not code. Re-read Lecture 1's sections on resource naming, verbs, status codes, and versioning before you start; you'll apply every one of them.

**The scenario:** Crunch Cycles wants to let customers initiate a product return. A return references an existing order, has a reason, a status that moves through a small lifecycle, and eventually a refund amount. You're designing the API surface a future returns-processing team (and, later, a customer-facing web form) will build against.

## Tasks

Produce a single Markdown file, `returns-api-spec.md`, with these sections:

1. **Resource list.** What are the resource(s)? At minimum you need `returns`. Decide: does it nest under `orders` (`/orders/{id}/returns`) or stand alone at the top level (`/returns`) or both? Justify your choice using Lecture 1's "nesting is for context, not ownership" rule.

2. **Endpoint table.** For every endpoint, document:

   | Method | Path | Purpose | Auth required? |
   |---|---|---|---|
   | ... | ... | ... | ... |

   At minimum, cover: list returns, get one return, create a return, and update a return's status (e.g., moving it from `requested` → `approved` → `refunded`). Decide whether status updates use `PUT` or `PATCH`, and say why.

3. **Request/response examples.** For the "create a return" endpoint, write out:
   - A sample request body (JSON).
   - The success response, with its exact status code.
   - **Three** different error responses with their status codes: one for a missing required field, one for referencing an order that doesn't exist, and one for trying to return an order that was never delivered (a business-rule violation, not a malformed request — pick the right code and explain the difference from the missing-field case).

4. **Status lifecycle.** A return moves through states (you choose the exact set, e.g. `requested → approved → refunded` or `requested → rejected`). Document: what HTTP call moves a return from one state to the next, and what happens (what status code, what body) if someone tries an invalid transition (e.g., moving `refunded` back to `requested`).

5. **Pagination and filtering.** Design the query parameters for `GET /returns` — at minimum, filtering by `status` and by `order_id`, plus `limit`/`offset`. Show one example URL.

6. **Versioning and auth.** State explicitly: what's the URL prefix? What auth mechanism, and what header does the caller send? (Reuse the API-key pattern from Lecture 1 unless you have a specific reason to deviate — state the reason if you do.)

## Expected result (spot checks)

- Your endpoint table has at least 4 rows and every row has a method, path, purpose, and auth column filled in — no blanks.
- Your three error examples use three **different** status codes, and each one's code is defensible against the status-code table in Lecture 1 (not just "some 4xx").
- Your status lifecycle section explicitly names the illegal transition and its response code (this should be `409 Conflict`, not `400` — explain why in one sentence).

## Done when…

- [ ] `returns-api-spec.md` has all 6 sections.
- [ ] Every endpoint uses a noun in the path and the correct HTTP verb for the action (re-check against Lecture 1's "verb in the URL" table if you're unsure).
- [ ] A teammate who has never seen this scenario could implement the `POST /returns` endpoint from your spec alone, with no follow-up questions.
- [ ] You've stated your auth mechanism and versioning prefix explicitly, not left implicit.

## Stretch

- Add an `idempotency_key` field to the "create a return" request body and document how the server should behave if the same key is sent twice (tie this back to Lecture 3's idempotency-key pattern).
- Sketch what a `GET /returns/{id}` response looks like once a return has multiple line items (a return doesn't have to be for the whole order) — how does your resource shape change?

## Submission

Commit `returns-api-spec.md` to your portfolio under `c37-week-07/exercise-01/`.
