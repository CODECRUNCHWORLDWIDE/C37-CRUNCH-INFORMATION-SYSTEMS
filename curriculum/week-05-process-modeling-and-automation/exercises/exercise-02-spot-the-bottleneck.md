# Exercise 2 — Spot the Bottleneck

**Goal:** Score every human task in the CrunchRide damage-to-repair process (from Lecture 1) as an automation candidate using the Lecture 2 rubric, then **prove or disprove your instinct about where the real bottleneck is** using actual SQL queries against `repair_tickets` — not a guess.

**Estimated time:** 90 minutes.

## Setup

You already have the `crunchride` database from the [week README](../README.md), including the 5 historical `completed` tickets and 5 current `open` backlog tickets. That's your data source for this whole exercise. Create a file `exercise-02.sql` for your queries and `exercise-02.md` for your written scoring and answers.

## Part A — Score every task (30 min)

Using the ten human tasks identified in Lecture 1's worked example (reproduced below for convenience), score each on the four Lecture 2 dimensions (rule-based-ness, frequency, error cost, stability — each High/Medium/Low) and write a one-line verdict: **Automate now**, **Automate later (name the blocker)**, or **Never automate**.

1. Flag problem on ride end (Rider)
2. Read ticket, judge severity (Dispatcher)
3. Assign nearby tech — minor path (Dispatcher)
4. Flag bike needs-pickup (Dispatcher)
5. Assign senior tech — major path (Dispatcher)
6. Inspect bike (Field Technician)
7. Repair on the spot (Field Technician)
8. Transport to depot (Field Technician)
9. Depot repair-or-retire decision (Depot)

*(Task 1, "flag problem on ride end," belongs to the Rider — think carefully about whether "automating" it even makes sense. What would that even mean? A task performed by a customer, not an employee, is a special case worth a sentence of its own.)*

Put your scoring in a table in `exercise-02.md`. You don't need to match Lecture 2's answer exactly (some of these have real room for judgment) — you need a rubric-backed, defensible verdict for each.

## Part B — Quantify the real bottleneck (45 min)

Before you look at the data, write down **one sentence predicting where you think the bottleneck is** — which step, which zone, which severity — based only on reading the process. Then write and run these queries and see if the data agrees with you.

1. **Time-to-assign, per historical ticket.** Compute `hours_to_assign` (opened → assigned) for every `completed` ticket. Which single ticket took longest? By how much did it beat/miss your prediction?

2. **The overall average and worst-case.** One summary query: average and max `hours_to_assign` across all completed tickets.

3. **Time-to-complete after assignment.** Compute `hours_to_complete` (assigned → completed) for the same tickets. Is *assignment* the slow part, or is the actual repair work the slow part? This changes what you'd automate.

4. **Today's backlog by zone.** How many `open`, unassigned tickets exist right now in each zone? (Join `repair_tickets` → `bikes` → `stations`.)

5. **Today's backlog by severity.** Split the same backlog by `severity`. How many of the backlog tickets are even eligible for the minor-path automation Lecture 3 builds?

## Part C — Write the decision log (15 min)

Using the Lecture 2 "DECISION LOG" format as a template, write two entries in `exercise-02.md`:

- One task you're confidently automating, with the evidence from Part B backing it up.
- One task you're confidently **not** automating, with the reason stated in a sentence a skeptical dispatcher would accept (not just "it's hard").

## Expected result (spot checks)

- Part B, Query 2: on the seed data, average `hours_to_assign` across the 5 completed tickets should land somewhere around 4–5 hours, with the worst case well over 24 hours (ticket for bike 103's battery issue).
- Part B, Query 5: exactly 3 of the 5 backlog tickets are `severity = 'minor'` (bikes 101, 108, 114) — these are the ones eligible for Lecture 3's automation. The other 2 (bikes 105, 110) are `major` and stay with the dispatcher for now.

## Done when…

- [ ] `exercise-02.md` has a completed scoring table for all 9 dispatcher/technician/depot tasks.
- [ ] `exercise-02.sql` has all 5 numbered queries from Part B, each under a comment.
- [ ] Your written prediction (before running queries) is recorded, and you explicitly say whether the data confirmed or contradicted it.
- [ ] Two decision-log entries are written, matching the Lecture 2 format.

## Stretch

- Write a query that computes, per **technician**, their average `hours_to_complete` across tickets they've handled — a first pass at "is one tech consistently faster/slower," which is itself a legitimate (if politically delicate) automation-adjacent question worth knowing how to ask.

## Submission

Commit `exercise-02.sql` and `exercise-02.md` to your portfolio under `c37-week-05/exercise-02/`.
