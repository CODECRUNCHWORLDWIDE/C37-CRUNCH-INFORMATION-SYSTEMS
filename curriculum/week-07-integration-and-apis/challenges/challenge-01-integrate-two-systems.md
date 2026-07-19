# Challenge 1 — Integrate Two Systems

**Time:** ~90 minutes. **Difficulty:** Medium–Hard. **No single right answer.**

## The scenario

Crunch Cycles' operations lead pulls you aside: "We need our warehouse system and our shipping carrier to actually talk to Crunch Cycles' order database. Right now someone re-types tracking numbers into a spreadsheet by hand every afternoon, and it's already caused two shipped orders to sit marked 'Pending' for a week." You're asked to design — not fully build — the integration.

This is the job you'll actually be asked to do more often than "write the code": look at two real systems, figure out what data needs to move between them, in which direction, how often, and what happens when it fails.

## Your task

Pick **one** of these two integration scenarios (or propose your own, with instructor/self sign-off that it's comparably real):

**Option A — Shipping carrier integration.** When an order's items are packed, Crunch Cycles needs to: (1) request a shipping label from a carrier's API, (2) receive a tracking number back, (3) later receive a webhook when the package is delivered, and (4) update `orders.status` and `orders.ship_date` accordingly.

**Option B — Accounting system integration.** Every night, Crunch Cycles needs to export completed orders (with line items, customer, and totals) to an accounting package so finance doesn't re-key invoices by hand. The accounting system exposes a REST API for creating invoices, but does **not** support webhooks — it can only be polled or pushed to.

Write `integration-design.md` covering:

1. **The data flow diagram (as text/ASCII or a described flow).** What data moves, in which direction, triggered by what? Be specific: is this event-driven (a webhook fires) or scheduled (a nightly job runs)? Justify the choice against Lecture 3's batch-vs-streaming framework.

2. **Point-to-point or hub?** Given that this is (for now) the *only* external integration Crunch Cycles has, which pattern do you pick, and — separately — at what point (how many integrations) would you reconsider?

3. **The contract.** For whichever direction Crunch Cycles is the API *consumer*, sketch the 2–3 calls you'd make (method, rough path, what you send/receive) against the third-party's API — you're free to invent realistic-looking endpoints since you don't have real carrier/accounting docs, but they should look like something you researched, not something arbitrary. If Crunch Cycles is the API *provider* in this flow (e.g., receiving the shipping webhook), sketch that endpoint the way you did in Exercise 3.

4. **Failure modes — list at least four, specific to this integration** (not the generic "the network could be down"). For each: what actually breaks in the Crunch Cycles data if it happens, and what's your mitigation? Example failure mode for Option A: "carrier's webhook for a delivered package never arrives (delivery confirmed on their side, but their webhook infra hiccups) — order stays `Shipped` forever. Mitigation: a nightly reconciliation job that polls the carrier's `GET /shipments/{id}` for any shipment still `Shipped` after N days, instead of relying on the webhook alone."

5. **What happens on day one vs. steady state.** The first time this integration runs, is there a backlog of existing orders/shipments to reconcile, or does it only need to handle *new* activity going forward? How do you handle the gap?

## Constraints

- You are not required to write working code for this challenge — this is a design document. But every design decision must be **specific to Crunch Cycles' actual schema** (reference real column names from `orders`, `order_items`, etc.) — a design that could apply to any company with any database is too generic to be useful.
- At least one failure mode must involve **duplicate delivery** of the same event (a webhook or a scheduled job firing twice) — tie your mitigation back to Lecture 3's idempotency techniques by name.

## Hints

<details>
<summary>On choosing batch vs. streaming for Option B</summary>

Ask: does finance need an invoice the instant an order completes, or is "by 9am the next morning" completely fine? For most companies the honest answer is the latter — a nightly batch export is simpler, easier to debug (you can re-run last night's export by hand if something looked wrong), and matches how accounting teams actually work (in daily/weekly batches, not real-time). Resist the pull toward "real-time" when the business doesn't need it — it's the more expensive choice in engineering complexity, not the more impressive one.

</details>

<details>
<summary>On Option A's reconciliation job</summary>

This is the single most common resilience pattern in webhook-based integrations, and it's worth internalizing: **never rely on a webhook alone for anything that matters.** Always pair it with a periodic "sweep" job that polls for anything that should have updated via webhook but didn't, and catches it. The webhook makes things fast; the sweep makes things eventually *correct* even when the webhook is lost.

</details>

## How success is judged

| Signal | Weak answer | Strong answer |
|--------|-------------|----------------|
| Specificity | Generic "the systems will sync data" | Names real columns, real endpoints, real triggers |
| Batch/streaming choice | Picks one without justification | Applies Lecture 3's "does latency actually matter here" test |
| Failure modes | Lists textbook failures ("network down") | Lists failures specific to *this* integration and *this* schema |
| Idempotency | Mentioned as a buzzword | Tied to a specific mechanism (natural key, idempotency key, or event log) for a specific failure mode |
| Day-one vs. steady-state | Not addressed | Explicitly handles the backlog/bootstrap problem |

## Submission

Commit `integration-design.md` to your portfolio under `c37-week-07/challenge-01/`.
