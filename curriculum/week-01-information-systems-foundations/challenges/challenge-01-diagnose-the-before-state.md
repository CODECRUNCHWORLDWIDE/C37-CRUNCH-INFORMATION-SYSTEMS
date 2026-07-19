# Challenge 1 — Diagnose the Before-State

> **Time:** ~1 hour. **Uses:** everything so far. This is the week's first real diagnostic exercise — messier and less guided on purpose.

## The scenario

Before the `is_components`/`is_flows` register you've been building all week existed, Riverbend ran its wholesale side like this (this is the company's actual "before" state — read it once, carefully):

> Ruth takes wholesale orders by phone and writes them into a shared Google Sheet called `Wholesale Orders 2026` — six tabs, one per quarter-ish, no consistent format. Anyone with the link can edit any cell. There's a `Status` column that different people fill in differently: values seen include `ordered`, `Ordered`, `in transit`, `?`, and blank.
>
> Elena doesn't look at the sheet. Every morning, Ruth tells her by Slack message roughly how many bags of what she needs to roast that day, based on Ruth's own mental tally of open orders — not the sheet, because the sheet is "too messy to read quickly."
>
> Nobody checks the sheet against actual bean stock before Ruth promises a delivery date to a customer. Two weeks ago, Ruth promised two different cafés delivery of the same rare micro-lot within the same week — there was only enough of that lot for one order. Nobody noticed until Elena went to pull the beans for the second order and there weren't any left. The second café's order slipped four days. Nora found out about the whole thing from the customer's angry email, not from anyone at Riverbend.
>
> Sam ships whatever Elena hands him, with no record beyond a shipping label — Riverbend has no way to look up, after the fact, what was in a given shipment or when it went out, unless Sam happens to remember.

## Part A — Name what's actually missing (25 min)

Do **not** write "communication was bad" or "they needed better software" — those are both true and both useless, because they don't point at a fix. Instead, for each of the four numbered failures below, name the **specific missing component or missing flow** using the vocabulary from this week's lectures (a specific row that should exist in `is_components`, or a specific row that should exist in `is_flows`, that doesn't).

1. Elena plans roasts from a mental tally Ruth reports by Slack, not from the actual order data.
2. Nobody checks bean stock against promised delivery dates before a promise is made.
3. There's no single source of truth for an order's status — the same real-world state gets written four different ways.
4. There's no record of what was actually in a shipment after the fact.

For each, write one sentence in the shape: *"The missing [component/flow] is ___, which would have prevented this by ___."*

## Part B — Trace the double-booking to its root cause (15 min)

Failure #2 above (the double-booked micro-lot) is the most expensive one — it cost Riverbend a customer relationship and nearly cost them the account. Using the gap → consequence → impact → cost → value pattern from Lecture 3 §5, write a four-to-five sentence argument explaining:

- **Gap:** which specific missing flow (from Part A) let this happen.
- **Consequence:** what happened because that flow was missing.
- **Impact:** the actual business cost (a slipped order, a furious customer, reputational risk with a wholesale account).
- **Fix + value:** what adding the missing flow would cost to build (roughly — it's fine to say "a day of work") versus what it prevents going forward.

## Part C — Fix it in the register (20 min)

The `Wholesale Orders 2026` spreadsheet is exactly the anti-pattern Lecture 2 §4 named. Write the `INSERT` statements that properly model what *should* exist instead — using your already-extended register (components `1`–`25` from Exercises 1–2) as the base. At minimum:

1. If it doesn't already exist in your register, add a flow from `Customer Order` (component `13`) directly into `Roast Scheduling` (component `6`) that's driven by the actual order data — not a Slack message. *(Check first: does flow `4` in the original seed already cover this? If so, explain in `notes.md` why the before-state scenario violates what that flow should be doing, rather than adding a duplicate.)*
2. Add a flow that checks `Green Coffee Inventory` (component `11`) **before** an order is confirmed, not after — this is the specific missing check that let the double-booking happen. You may need to reason about whether this requires a new **process** component (e.g., "Stock Check") in addition to a new flow — if so, add both, with justification.
3. Add whatever data component records what a shipment actually contained, tied back to Sam Higgins and the Warehouse Scanner, if it isn't already adequately covered by `Delivery Manifest` (component `15`) — check the existing seed before assuming it's missing.

## Deliverable

`notes.md` with your Part A and Part B write-ups, plus `week01-solutions.sql` with your Part C `INSERT` statements (use `component_id`/`flow_id` values that don't collide with anything you've already added).

## Check yourself

- Did you catch that some of what Part C asks for might *already exist* in the original seed (flow `4`, flow `5`, component `15`)? Part of this challenge is not over-fixing something that isn't actually broken — go back and check the seed data before assuming a gap.
- Is your Part B argument specific enough that a stakeholder unfamiliar with the technical details could read it and understand exactly what to approve?
- Would your Part C fix have actually prevented the specific double-booking described, or does it just generally "improve things"? Be precise.
