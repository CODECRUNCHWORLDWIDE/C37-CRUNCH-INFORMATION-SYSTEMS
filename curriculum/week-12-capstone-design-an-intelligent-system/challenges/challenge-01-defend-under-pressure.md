# Challenge 1 — Defend the Design Against a Hostile Review

**Depends on:** Exercises 1–3 (your organization, architecture, and delivery plan).

Real design reviews are not friendly. A stakeholder with budget authority, a skeptical engineer, or an auditor will find your weakest point and push on it — not out of hostility for its own sake, but because that's their job: don't approve or fund something that hasn't been stress-tested. This challenge simulates that review. There is no partial credit for getting defensive; there is full credit for using Lecture 3's four-part trade-off structure and, where the reviewer is actually right, saying so and adjusting.

## The setup

You're presenting your capstone design (from Exercises 1–3) to a review panel. The panel asks the questions below, **in the persona given**. Answer every question, in writing, as if you were speaking directly to that reviewer. Use Lecture 3's four-part structure (state the decision → state the honest rejected alternative → state the real trade-off in *their* terms → state what would change your mind) for every answer where it applies.

## Round 1 — The budget-focused executive

> "You're asking me to approve a 3-year TCO of $[your number]. Convince me this is cheaper than just hiring someone to keep doing this by hand, or buying an off-the-shelf tool. Why build custom at all?"

Answer this using your actual Exercise 3 TCO model. If a real off-the-shelf tool exists for part of your system, name it and compare honestly — don't strawman it as worse than it is.

## Round 2 — The skeptical engineer

> "Walk me through what happens when your nightly ETL job fails at 2am and nobody notices until the Monday dashboard is wrong. What's your actual failure mode, not the happy path?"

Answer by tracing a concrete failure through your Exercise 2 diagram: what breaks first, what's the blast radius, how would anyone find out, and what's the fix. If your design doesn't currently answer this (most first-draft designs don't), say what you'd add — a monitoring/alerting step, a retry policy — and where in your phased plan (Exercise 3) it belongs.

## Round 3 — The privacy-minded board member

> "You said this AI feature uses customer data to make predictions about them. What happens when it's wrong about a real person, and who's accountable when that happens?"

Answer using Week 9's and Week 11's human-in-the-loop pattern: state explicitly whether a human reviews the AI output before it affects a real person, and if not, why that's an acceptable risk for *this specific* decision (Lecture 2's risk register should already have this named — cite it).

## Round 4 — The "why not simpler" challenger

> "This whole thing sounds like a lot of infrastructure for an organization this size. Why isn't this just three formulas in a spreadsheet and a shared calendar?"

This is the sharpest question in the set, because for a genuinely small organization, it might be partly right. Answer honestly: name the specific thing your design does that a spreadsheet structurally cannot (concurrent multi-user writes without corruption, enforced relationships instead of typo-prone free text, a security boundary, a queryable history) — and if any part of your Exercise 1 scope really *would* be better served by a simpler tool, say so and cut it. Defending a design also means knowing what not to defend.

## Round 5 — Your choice

Write **one more hostile question**, in a persona of your choosing, that specifically targets the weakest point in *your own* design — the part you'd least want a sharp reviewer to find. Then answer it. (If you can't think of a weak point, that itself is a finding: reread your Exercise 2 and 3 work looking for the thing you glossed over.)

## Deliverable

`challenge-01/defense-memo.md` with all five rounds answered, each in Lecture 3's four-part structure where applicable, running roughly 1–2 pages total. For any round where the reviewer's challenge genuinely changes your design, note the change and where it lands in your Exercise 3 phase plan.

## What "great" looks like

- Every answer is grounded in your actual Exercise 1–3 numbers and diagram — no invented figures introduced for the first time here.
- At least one round results in an honest, stated change to the design, not just a justification of the original.
- Round 4's answer draws a real, structural line between what a database-backed system does and what a spreadsheet cannot — vague appeals to "professionalism" don't count.
- The tone throughout is confident, not defensive: acknowledging a weak point and having a plan for it reads stronger than insisting nothing is wrong.
