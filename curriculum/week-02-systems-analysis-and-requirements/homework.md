# Week 2 — Homework

Extra practice, spaced across the week, tying together elicitation, requirement writing, and use-case/user-story modeling. Unlike the exercises, most of this asks you to go find real input rather than working only from the seeded Meridian data — that's deliberate; elicitation is a skill you can only build by actually talking to someone.

**Estimated time:** ~5 hours across the week.

## Part 1 — Interview a real person about a real annoyance (90 min)

Find someone — a coworker, a family member, a friend running a small side business, a club you're in — who has a process they find genuinely annoying: something manual, repetitive, or error-prone that they currently do by memory, sticky notes, text messages, or (very likely) a spreadsheet.

1. Conduct a short interview (15–20 minutes is enough). Use the laddering technique from Lecture 1 — start with their vague complaint, ask for a specific recent example, and follow up until you hit something concrete and testable.
2. Write up your **elicitation notes** as you would insert them into `elicitation_sessions` — stakeholder, date, method, summary — even if you're keeping them in a plain text file instead of an actual database this time.
3. Identify at least one **workaround** they've invented (a personal spreadsheet, a sticky note, a group chat used as a tracking system) and write one sentence on what real need it's evidence of.

Deliverable: `homework/part1-interview-notes.md`.

## Part 2 — Write 5 requirements from that interview (60 min)

From what you learned in Part 1:

1. Write at least 5 requirement statements (mix of FR and NFR) using the Lecture 2 template, each passing the 6-point quality checklist.
2. Assign MoSCoW priorities.
3. For each, write one sentence tracing it back to the specific thing your interviewee said (traceability, without a database this time — just cite the quote).

Deliverable: `homework/part2-requirements.md`.

## Part 3 — Rewrite 10 vague requirements (45 min)

Below are 10 vague statements pulled from real (composited, anonymized) project kickoffs. For each, apply Lecture 2's rewrite discipline — identify what's missing (a number, a role, a channel, a condition) and rewrite it as a testable requirement. You don't need a real system context for these; the point is pure rewriting fluency.

1. "The dashboard needs to load fast."
2. "Only admins should be able to delete stuff."
3. "The app should work on phones too."
4. "We need better error messages."
5. "The system should scale."
6. "Search needs to actually find what people are looking for."
7. "There should be some kind of backup."
8. "The onboarding should be smoother."
9. "We need to know who changed what."
10. "It should handle a lot of users."

Deliverable: `homework/part3-rewrites.md`, one rewritten requirement per line, each with the number/role/channel/condition you added called out.

## Part 4 — Two user stories with acceptance criteria, from Part 1 (45 min)

Convert two of your Part 2 requirements into user stories (`As a / I want / so that`), run each through INVEST, and write at least 2 Given/When/Then acceptance criteria per story.

Deliverable: `homework/part4-user-stories.md`.

## Part 5 — Use case sketch (60 min)

Pick the single most complex interaction from your Part 1 interview — the one with the most "what if" branches — and write a full use case for it (primary actor, main success scenario, at least one alternate flow, at least one exception flow), per Lecture 3's format.

Deliverable: `homework/part5-use-case.md`.

## Done when…

- [ ] Part 1 shows real laddering — your notes show the vague opening statement *and* the specific thing it turned into, not just the final answer.
- [ ] Part 2's requirements each trace to a specific quote from the interview.
- [ ] Part 3's 10 rewrites each add at least one of: a number, a role, a channel, a condition.
- [ ] Part 4's stories both pass INVEST, explicitly checked.
- [ ] Part 5's use case has all three flow types — main, alternate, exception — not just the happy path.

## Submission

Commit the `homework/` folder to your portfolio under `c37-week-02/homework/`.
