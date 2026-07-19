# Exercise 2 — Functional vs. Non-Functional

**Goal:** Take a raw list of statements pulled straight from meeting notes, classify each as functional or non-functional, rewrite the vague ones so they're testable, and load the result into the `requirements` table with a MoSCoW priority.

**Estimated time:** 60 minutes.

## Setup

Same database as Exercise 1. Create `exercise-02.sql` for your `INSERT` statements and `exercise-02-notes.md` for your classification/rewrite reasoning.

## The raw list

These are pulled, close to verbatim, from this week's elicitation sessions (see `elicitation_sessions` in the seed data) and the follow-up notes taken during them. Some are already close to testable; most are not. None are yet correctly formatted requirement statements.

1. "Associates need to see warehouse stock without calling."
2. "The lookup needs to be fast."
3. "Only managers can approve orders over $500."
4. "The system needs to log who approved what and when, kept for 3 years."
5. "It should work even during the Saturday rush across all 14 stores."
6. "Customers get notified when their order status changes."
7. "The interface has to be easy for someone with barely any training."
8. "It should integrate with the existing POS system, not be a separate login."
9. "Orders should never just disappear — need a record of every status change."
10. "The system needs to be up basically all the time during store hours."
11. "Associates can cancel an order if it hasn't shipped yet."
12. "Data should be safe — only the right people see payment info."
13. "New stores should be addable without a developer having to change code."
14. "The call center needs to look up any customer's order by phone number or order number."
15. "It has to look professional."
16. "Reports need to be exportable for finance."

## Tasks

1. **Classify each statement** as `functional` or `non-functional` in `exercise-02-notes.md`. For any statement you find genuinely ambiguous (arguably both, or neither until reworded), say so and explain your call — this happens more than a clean textbook example suggests.

2. **Discard true non-requirements.** At least one item on this list is not a requirement at all once you look closely (an opinion with no testable content, like "looks professional," with nothing underneath it that a stakeholder could confirm objectively). Identify it, and in your notes explain what follow-up question (per Lecture 1's laddering) you'd ask to find the real requirement hiding under it, if one exists.

3. **Rewrite every remaining statement** into the requirement template from Lecture 2 §3 — add the number, role, channel, or condition each one is currently missing. Compare your rewrite against the checklist in Lecture 2 §2 (clear, testable, atomic, feasible, traceable, prioritized) before moving on.

4. **Split any non-atomic statement.** At least one item bundles more than one testable idea into a single sentence — find it and split it into separate requirement rows.

5. **Assign MoSCoW priority** to each. You will not have a stakeholder in the room to ask — use the elicitation notes and the stakeholder power/interest map from Lecture 1 to make a defensible call, and write one line justifying each `must`.

6. **Insert every requirement** into the `requirements` table, using the next available `FR-0xx`/`NFR-0xx` IDs and linking `source_stakeholder_id` to whichever stakeholder's session the note came from (check `elicitation_sessions` if you need to trace it back).

## Starter

```sql
-- exercise-02.sql
INSERT INTO requirements (req_id, req_type, statement, priority, source_stakeholder_id, status)
VALUES
('FR-002', 'functional',
 'Store associates shall be able to view current warehouse stock quantity for any SKU without contacting the warehouse by phone.',
 'must', 1, 'draft'),
('NFR-003', 'non-functional',
 'Warehouse stock lookups shall return within 2 seconds for 95% of requests during store hours.',
 'must', 1, 'draft');
-- continue for the remaining 14 statements (after classifying, splitting, and discarding as needed)
```

## Expected outcome

- `exercise-02-notes.md` classifies all 16 raw statements, flags the one you discarded (with reasoning), and flags the one(s) you split.
- `exercise-02.sql` inserts roughly 15–18 rows into `requirements` (16 raw statements, minus 1 discarded, plus at least 1 split into two).
- Every inserted `statement` contains a number, a role, a channel, or a condition — re-check any that don't against Lecture 2 §2's checklist.
- `SELECT req_type, COUNT(*) FROM requirements GROUP BY req_type;` shows a reasonable mix, not 100% one type.

## Done when…

- [ ] Every kept statement is atomic — no row contains an "and" joining two independently testable ideas.
- [ ] Every kept statement is testable — you could describe, in one sentence, exactly how you'd check it passed.
- [ ] MoSCoW priorities aren't all `must` — run the `GROUP BY priority` query from Lecture 2 §4 and sanity-check your own split.
- [ ] The discarded statement and your reasoning are documented, not silently dropped.

## Stretch

- For statement 8 (POS integration), write it as a non-functional *constraint* rather than a functional feature — explain in one sentence why "must integrate with an existing system" is usually a constraint on the solution space, not a behavior of the new system itself.

## Submission

Commit `exercise-02.sql` and `exercise-02-notes.md` to your portfolio under `c37-week-02/exercise-02/`.
