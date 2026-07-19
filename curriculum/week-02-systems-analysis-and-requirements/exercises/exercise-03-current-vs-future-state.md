# Exercise 3 — Current-State vs. Future-State

**Goal:** Diagram the special-order process as it actually happens today (current-state), then diagram how it should happen once the requirements from Exercises 1–2 are built (future-state), and name precisely where the gap between them lives.

**Estimated time:** 90 minutes.

## Setup

No database work in this one — you're diagramming and writing. Create a file `exercise-03.md` for both diagrams and your gap analysis. You may draw the diagrams as Mermaid flowcharts (renders directly on GitHub) or as labeled ASCII swimlanes — either is fine, but the *lanes* (which actor does each step) must be visually distinguishable, not just a flat list of boxes.

## Background — the current state, from this week's elicitation notes

Reconstruct the current process from what you've already learned this week (Lecture 1, and `elicitation_sessions` in the database):

- A customer asks a Store Associate for an item not on the shelf.
- The associate calls the Distribution Center (DC) and waits on hold (15–40 minutes observed).
- The DC clerk checks physical stock and tells the associate a quantity over the phone (no live system lookup on either end).
- The associate writes the order on a sticky note, then transcribes it into the shared spreadsheet.
- The associate tells the customer an estimated date, based on what the DC clerk said verbally.
- If the order is over $500, there is **no formal approval step today** — this is exactly Priya's concern from Lecture 1.
- The warehouse ships the item when it's picked, and updates the spreadsheet's status cell (free text — "ordered", "Ordered", "in transit", "?", or blank, per the document review finding).
- The customer finds out the order shipped only by calling the store, which then calls the DC to check — often re-triggering the same hold-time wait.
- Call center escalations happen when a customer calls corporate directly instead of the store, and the call center has no visibility into the spreadsheet at all.

## Tasks

1. **Diagram the current state.** Use swimlanes for at least these four actors: Customer, Store Associate, Warehouse/DC, Call Center. Show every handoff (phone call, sticky note, spreadsheet edit) as an explicit step — the handoffs are exactly where this process bleeds time and accuracy, so don't collapse them into one box.

2. **Time and count the current state.** For each step, note who does it and (roughly, from the elicitation notes) how long it takes or how error-prone it is. Total it up: how long does one special order take from "customer asks" to "associate can promise a date," today?

3. **Diagram the future state.** Using the requirements you wrote in Exercises 1–2 (or the ones from the lecture worked examples if you skipped ahead), redraw the same process as it *should* work: associate looks up live stock directly, system calculates the estimate, orders over $500 route to a Manager-approval step automatically, status updates flow to the customer and the call center without a phone call.

4. **Name the gap, explicitly.** For each step that disappears or changes between current and future state, write one sentence: what waste, delay, or risk did it remove, and which requirement (by ID, from Exercise 2) is responsible for removing it? A gap analysis that doesn't trace back to a specific requirement is just an opinion.

5. **Flag anything future-state does NOT fix.** Not every problem in the current state is solvable by this system alone (e.g., a customer who insists on calling instead of checking a status page). Name at least one such gap explicitly rather than implying the new system fixes everything.

## Starter (Mermaid swimlane sketch — current state, partial)

```
flowchart TD
    subgraph Customer
        C1[Asks associate for item]
    end
    subgraph "Store Associate"
        A1[Calls DC, holds 15-40 min]
        A2[Writes sticky note]
        A3[Transcribes to shared spreadsheet]
        A4[Tells customer estimated date - verbal, from DC clerk]
    end
    subgraph "Warehouse / DC"
        W1[Clerk checks physical stock, tells associate by phone]
        W2[Picks + ships when reached]
        W3[Updates spreadsheet status - free text]
    end
    subgraph "Call Center"
        CC1[Customer calls corporate instead of store]
        CC2[No visibility into spreadsheet - escalates]
    end
    C1 --> A1 --> W1 --> A2 --> A3 --> A4
    W2 --> W3
    C1 -.-> CC1 --> CC2
```

Continue this for every remaining step, then build the future-state version alongside it.

## Expected outcome

- Two complete swimlane diagrams in `exercise-03.md`, current and future, using the same four actor lanes so they're directly comparable.
- A step-count and rough time estimate for the current state (should land somewhere around 20+ minutes and 5+ handoffs, matching the observation in Lecture 1).
- A gap table: at least 6 rows, each naming a removed/changed step, the waste or risk it caused, and the requirement ID that fixes it.
- At least one explicitly named limitation the new system does not solve.

## Done when…

- [ ] Both diagrams use visually distinct swimlanes per actor, not a single unlabeled flow.
- [ ] Every current-state handoff (phone call, sticky note, spreadsheet edit) appears as its own step.
- [ ] Every gap-table row cites a specific requirement ID, not a vague "the new system helps here."
- [ ] At least one honest limitation is documented — a strong analysis doesn't oversell the fix.

## Stretch

- Add a fifth lane, "System," to the future-state diagram showing exactly which steps are now automated vs. still human-performed — this is the clearest possible answer to "what are we actually buying with this project?"

## Submission

Commit `exercise-03.md` to your portfolio under `c37-week-02/exercise-03/`.
