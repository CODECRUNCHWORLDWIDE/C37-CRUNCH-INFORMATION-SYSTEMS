# Exercise 1 — Write User Stories

**Goal:** Take one raw feature request and turn it into a small, INVEST-passing backlog of user stories with Given/When/Then acceptance criteria, stored in the `user_stories` table.

**Estimated time:** 60 minutes.

## Setup

Confirm your `meridian_sos` database is seeded (see the [week README](../README.md)):

```sql
SELECT COUNT(*) FROM stakeholders;   -- must print 8
```

Create a file `exercise-01.sql` for your `INSERT` statements and a file `exercise-01-notes.md` for your written work (INVEST check, splitting rationale).

## The feature request

This is the actual sentence Sofia Nakamura (Regional Ops Director) wrote in an email to IT:

> "Can we get some kind of order tracking thing so customers and the call center aren't in the dark, and so managers can see how each store is doing on this?"

Treat this exactly like a raw stakeholder request from Lecture 1 — it names at least three different actors, bundles several distinct needs, and proposes no specifics. Your job is not to guess what Sofia "really wants" from thin air — it's to apply what Lecture 1 and 3 taught you: identify the actors bundled inside it, and split it into small, valuable, testable stories.

## Tasks

1. **List every actor implied by the sentence.** There are at least three. For each, name the actor role (not a person's name — a role, per Lecture 3 §1).

2. **Write 5–8 user stories** in `As a [actor], I want [goal], so that [benefit]` form, one per distinct need you can identify inside the sentence. Do not invent needs the sentence doesn't support — if you're unsure whether a need is really there, note it as an open question in `exercise-01-notes.md` instead of guessing.

3. **Run each story through INVEST** in `exercise-01-notes.md` — one line per story is enough, e.g. "US-003 passes INVEST; Small because it's a single view, one actor." If a story fails a letter, split it and show the split.

4. **Write at least 2 Given/When/Then acceptance criteria** for each story — at minimum, a happy path and one edge case (an empty result, a permission boundary, a system-down case).

5. **Insert every story** into the `user_stories` table with a unique `story_id` (`US-0xx`, continuing from any IDs already used this week), a MoSCoW `priority`, and `status = 'draft'`. Leave `linked_req_id` NULL for now — that traceability link happens once you've written matching requirements in Exercise 2.

## Starter

```sql
-- exercise-01.sql
INSERT INTO user_stories (story_id, actor, wants, benefit, priority, acceptance_criteria, linked_req_id, status)
VALUES
('US-010', 'Customer',
 'to check my special order status online without calling the store',
 'I know when to expect my item without waiting on hold',
 'must',
 'Given an order exists with status "in transit", When the customer looks it up by order number, Then the system shows "in transit" and the estimated delivery date.
Given an order number that does not exist, When the customer looks it up, Then the system shows a clear "order not found" message, not an error page.',
 NULL, 'draft');
-- continue for the remaining actors/stories you identified
```

## Expected outcome

- `exercise-01.sql` inserts 5–8 rows into `user_stories`, covering at least the Customer, Call Center, and Store Manager actors.
- `SELECT COUNT(*) FROM user_stories;` returns 5–8 (plus any from other exercises you've already run).
- Each story is independently understandable — someone who never saw Sofia's email could read `US-010` alone and know exactly what to build and how to check it's done.
- `exercise-01-notes.md` shows your actor list, your INVEST check per story, and any story you split (with the before/after).

## Done when…

- [ ] Every actor implied by the sentence is represented by at least one story.
- [ ] No single story tries to cover more than one actor's need (that's the #1 INVEST failure in this exercise).
- [ ] Every story has at least 2 acceptance criteria, including one non-happy-path case.
- [ ] `exercise-01-notes.md` explicitly states any assumption you made about what Sofia meant.

## Stretch

- Write UC-level detail (main flow + one alternate flow, per Lecture 3 §2) for just the Store Manager's "see how each store is doing" story — it's the one most likely to actually need full use-case treatment, since it may involve aggregating across stores and roles.

## Submission

Commit `exercise-01.sql` and `exercise-01-notes.md` to your portfolio under `c37-week-02/exercise-01/`.
