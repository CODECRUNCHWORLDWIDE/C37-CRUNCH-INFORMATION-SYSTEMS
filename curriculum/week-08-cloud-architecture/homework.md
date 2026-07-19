# Week 8 — Homework

Extra practice, spaced across the week rather than crammed into one sitting. None of this repeats the exercises or challenges verbatim — it's new scenarios and new angles on the same skills, so the reasoning generalizes past this one course project.

**Estimated total time:** 5 hours across the week (roughly 1h/day, Monday–Friday, alongside the day's other work).

---

## 1. Classify five more systems (45 min)

Using the same format as Exercise 1 (model + 1–2 sentence justification), classify:

1. A university's student information system (grades, enrollment, transcripts) — built decades ago on a mainframe, still running, nobody wants to touch it.
2. A three-person startup's error-tracking and logging tool (think: something like Sentry).
3. A logistics company's real-time GPS truck-tracking dashboard, built in-house because it's the company's core competitive advantage.
4. A small nonprofit's donor-management and email-outreach tool.
5. A video game studio's matchmaking servers, which need very specific low-latency networking control most PaaS platforms don't expose.

For each, note one thing about the *scenario* (not the technology) that pushed you toward your answer — team size, whether it's core business value, regulatory/latency requirements, or an existing legacy constraint.

## 2. Redundancy audit of a real system you use (45 min)

Pick any app or website you personally use regularly (a bank, a streaming service, a small SaaS tool — anything). Without any inside knowledge, reason from the outside:

1. Does it seem to have redundancy? (Has it ever gone down for you, and if so, how did it recover — instantly, or with a visible outage window?)
2. Guess, and justify your guess, at roughly what availability tier (99%, 99.9%, 99.99%) it's likely targeting, based on how critical its function is and how much downtime you've personally observed.
3. If you were designing it from scratch, what's the one redundancy investment (multi-zone app, database standby, etc.) you'd prioritize first, and why that one?

Write your answer as `redundancy-audit.md`, 300–400 words.

## 3. Read a real pricing page and compute a scenario (60 min)

Pick one cloud provider (AWS, Google Cloud, Azure, Render, Railway, or Fly.io) and read its **actual, current pricing page** for compute and managed Postgres. Then, in a Python script `pricing-scenario.py`, compute the monthly cost of a hypothetical deployment: 3 app instances of a size you choose, one managed Postgres instance with 50 GB storage, and 200 GB of monthly egress. Print the total, broken down by line item, with each unit price cited in a comment.

## 4. Vertical vs. horizontal, applied to a non-web example (45 min)

Lecture 3 focused on a web API. Pick a **different** kind of workload — a nightly batch job that processes 10 million rows of data (imagine Week 4's ETL pipeline at real scale), or a video-encoding service that converts uploaded files. For your chosen workload, answer in `scaling-tradeoffs.md`:

1. Does this workload scale horizontally, vertically, both, or neither easily? Why?
2. Is it stateless in the sense Lecture 3 requires for horizontal scaling? If not, what would need to change?
3. Would a load balancer even make sense for this workload, or does it need a different pattern (e.g., a job queue distributing work to multiple workers)? Explain your reasoning.

## 5. Diagnose a wasteful bill from a written description (45 min)

Unlike Challenge 2, you're not given a table this time — just a paragraph. Read it, then write out (in prose, no SQL required this time) every cost trap from Lecture 3 you can identify:

> "Our system runs 4 production app instances 24/7, sized for our Black Friday peak traffic, all year round. We also have a staging environment that's an exact copy of production, always on, that only gets used during the two weeks before a release. Our database backups have been running nightly since launch 14 months ago, and we've never set a retention policy, so we're storing all 420 of them. Last month we noticed our monthly bill had a `$340` line item for a load balancer attached to a service we decommissioned six months ago. Our app servers are in `us-west-2`; our database, for historical reasons nobody quite remembers, is in `ap-southeast-1`."

Write `bill-diagnosis.md` naming each specific waste pattern (there are at least four) and, for each, the specific fix from this week's lectures.

## 6. Design one thing you'd need before trusting this with real money (45 min)

Crunch Cycles' mini-project deployment takes real orders in the story of the course, but the actual database has no protection against, say, someone finding the connection string and reading every customer's data. In 200–300 words (`before-real-money.md`), name the **one** thing you'd add to this week's deployment before you'd trust it with a paying customer's real credit card and address — and be specific about *why* this week's architecture doesn't already cover it. (You're not expected to build it — Week 9 covers security, privacy, and governance in depth. This is a prediction exercise: what gap do you already see?)

---

## Time budget

| Problem | Time |
|--------:|----:|
| 1 | 45 min |
| 2 | 45 min |
| 3 | 60 min |
| 4 | 45 min |
| 5 | 45 min |
| 6 | 45 min |
| **Total** | **~4.75 h** |

After homework, take the [quiz](./quiz.md) and ship the [mini-project](./mini-project/README.md).
