# Challenge 1 — Automate a Real Process You Know

**Time:** ~90 minutes. **Difficulty:** Medium. **No single right answer.**

## The point of this challenge

Every exercise this week used CrunchRide's data, already loaded, already clean. Real process-improvement work never starts that tidy. This challenge strips that scaffolding away: you pick a real, manual, multi-step process you actually deal with — something at your job, your school, a club you run, even a household chore with more than one person involved — model it, and automate one real step of it.

## Your task

### Part 1 — Pick and describe the process (15 min)

Pick a process that:

- Has **at least 3 distinct steps** done by **at least 2 different people or roles** (or one person wearing two hats — "student" and "instructor," say).
- Currently runs on **memory, paper, chat messages, or a spreadsheet** — something this course would call out as fragile.
- You can describe accurately, because you've actually watched or done it, not guessed at.

Examples that have worked well for past students: a club's event-signup-to-check-in process; a small business's customer-order-to-fulfillment process; a household's bill-splitting-and-payment-tracking process; a school club's member-dues process. Pick something real to you — the specificity is what makes the modeling exercise honest.

Write a short (150–250 word) description of the process in your own words, the same way Sofia described onboarding in Exercise 1 — plain language, as if explaining it to a coworker.

### Part 2 — Model it in BPMN (25 min)

Diagram the full process: swimlanes for every participant, events, tasks (typed), and gateways. Same standard as Lecture 1 and Exercise 1 — a stranger should be able to read your diagram and understand exactly how the process works without you in the room to explain it.

### Part 3 — Score and pick one task to automate (15 min)

Using the Lecture 2 rubric, score at least 3 of the human tasks in your diagram. Pick the **one** that scores best as an automation candidate. Write a short decision-log entry (Lecture 2's format) explaining why.

### Part 4 — Build it (30 min)

Automate that one task. It does **not** need to be a full production system — it needs to demonstrate the pattern:

1. A small SQL schema (1–3 tables) representing the data the task needs. **SQL, not a spreadsheet** — this is non-negotiable per this course's data rule, even if the "real" version of your process currently lives in a spreadsheet today. That's exactly the gap you're closing.
2. A short Python script (20–60 lines is plenty) that reads the relevant rows, applies your rule, writes the result, and logs what it did.
3. At least a basic idempotency guard — running it twice shouldn't duplicate work. It doesn't need Lecture 3's full retry/scheduling machinery, but the core "safe to re-run" property is required, not optional.

## Constraints

- The automated task must be genuinely **rule-based** — if you find yourself writing a rule that's really just "guess what a human would decide," that's a sign you picked the wrong task. Go back to Part 3 and pick a different one.
- Keep the schema small. Three tables and 10–20 rows of realistic sample data is plenty to prove the pattern works — this challenge rewards a clean, complete small example over a sprawling, half-finished big one.

## How success is judged

| Signal | Weak answer | Strong answer |
|--------|-------------|----------------|
| Process description | Vague, hard to picture | Specific enough a stranger could model it |
| BPMN diagram | Missing lanes or gateway types wrong | Correct notation throughout, matches the description |
| Automation candidate scoring | One task picked with no reasoning | Rubric applied to 3+ tasks, clear reasoning for the pick |
| The build | Doesn't run, or duplicates work on a second run | Runs cleanly, idempotent, logs its actions |
| SQL, not a spreadsheet | Schema improvised, no real thought given | Schema reflects real entities/relationships from the process |

## Submission

Commit your process description, BPMN diagram, decision log, SQL schema, and Python script to your portfolio under `c37-week-05/challenge-01/`.
