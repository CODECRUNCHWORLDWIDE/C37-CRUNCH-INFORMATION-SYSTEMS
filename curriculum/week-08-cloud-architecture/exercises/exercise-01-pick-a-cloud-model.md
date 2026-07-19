# Exercise 1 — Pick a Cloud Model

**Goal:** Turn Lecture 1's IaaS/PaaS/SaaS framework into a reflex — read a real system description and place it on the spectrum in under a minute, with a one-sentence justification that would survive a code-review comment.

**Estimated time:** 1 hour.

## Setup

Create a file `cloud-model-picks.md`. For each system below, write:

1. **Model:** IaaS, PaaS, or SaaS (pick the single best fit — some are genuinely close calls; say which second choice you rejected and why, if it's close).
2. **Justification (1–2 sentences):** grounded in the shared-responsibility reasoning from Lecture 1 — what would you be responsible for, and why does that make sense (or not) for this scenario?

## Tasks — classify these 10 systems

1. **Crunch Cycles' Flask REST API** (the app itself, from Week 7), deployed by a two-person dev team with no dedicated ops staff.
2. **The `crunchcycles` PostgreSQL database** — same team, same constraints.
3. **A payment page that charges a customer's credit card** at checkout.
4. **A research lab's one-off script** that trains a machine learning model for 6 hours on a GPU, then never runs again.
5. **A bank's core transaction ledger**, subject to strict regulatory control over exactly which physical region and hardware processes the data.
6. **The team's internal chat and video calls.**
7. **A marketing team's monthly email newsletter** to 40,000 subscribers.
8. **A legacy internal tool** that only runs on a specific old Linux kernel version no modern platform supports out of the box.
9. **A startup's brand-new MVP web app**, built by a solo founder who wants to spend all available time on product, not infrastructure.
10. **A company's applicant-tracking system for hiring** — a workflow (post a job, collect applications, schedule interviews) that dozens of existing products already do well.

## Expected result (spot checks)

- #1 and #9 → **PaaS.** Both are custom application code owned by a small team that wants to avoid ops work — the textbook PaaS case from Lecture 1.
- #3 and #10 → **SaaS.** Neither is the business's core differentiator; both are solved problems (Stripe/a payments processor; an existing ATS product) worth buying rather than building.
- #5 and #8 → **IaaS** (or arguably on-prem for #5, if the regulation is strict enough — a defensible answer either way, as long as you justify it). Both have a stated, concrete reason a higher-level platform can't satisfy: regulatory control over physical infrastructure, and a kernel version no PaaS supports.
- #4 → **IaaS**, usually via a specialized GPU instance — a good example of a workload that doesn't fit standard PaaS compute shapes cleanly (see Lecture 2's compute section).

## Done when…

- [ ] `cloud-model-picks.md` has all 10 systems classified with a stated model and a justification.
- [ ] At least 2 of your justifications explicitly reference the shared-responsibility table from Lecture 1 (who manages what).
- [ ] For any system where you considered two models seriously (#5 and #8 are the likeliest candidates), you named the runner-up and said why you rejected it.
- [ ] Your answers for #1/#9 and #3/#10 match the reasoning pattern in "Expected result" even if your exact wording differs.

## Stretch

- Add an 11th system of your own choosing — something from your own life or a company you know — and classify it the same way.
- For #2 (the `crunchcycles` database), write one sentence on what would have to change about Crunch Cycles' situation for IaaS (self-hosted Postgres) to become the *better* choice instead of PaaS.

## Submission

Commit `cloud-model-picks.md` to your portfolio under `c37-week-08/exercise-01/`.
