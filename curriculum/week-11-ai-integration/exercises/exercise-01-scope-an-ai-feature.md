# Exercise 1 — Scope an AI Feature

**Goal:** apply Lecture 1's four-question framework to five candidate features, in writing, and reach a build/don't-build verdict for each with a one-sentence justification.

## Background

Crunch Cycles' product manager has come to you with five feature requests, pulled from actual conversations with sales, support, and finance. Your job is not to build any of them yet — it's to score each one and write down whether it's ready to build, and if not, what would have to change first.

## Task

For each of the five candidates below, answer Lecture 1's four questions — **Value**, **Data**, **Error tolerance**, **Latency/cost** — in your own words, using specifics from the `crunchcycles` schema where you can (table and column names, not vague references). End each with one of three verdicts:

- **Build** — clears all four questions as specified.
- **Build a narrower version** — the idea has real value, but the *autonomous* version fails error tolerance; specify what changes to make it safe (usually: add a human-in-the-loop step).
- **Don't build** — fails value or data availability outright; say what would need to be true for that to change.

### The five candidates

1. **"Predict which customers are about to churn"** — flag customers who haven't ordered in a while and are unlikely to order again, so a rep can proactively reach out.
2. **"Auto-generate the weekly sales summary email"** — an LLM reads last week's `orders`/`order_items` and drafts a summary email that goes out to the sales team every Monday morning with no review.
3. **"Let customers chat with an AI to check their own order status"** — a customer-facing (not staff-facing) version of this week's Q&A copilot, answering questions from people outside the company.
4. **"Recommend a maximum credit limit for new B2B accounts"** — a model suggests a dollar figure finance uses when deciding whether to extend net-30 terms to a new customer.
5. **"Automatically write and post a product description when a new item is added to `products`"** — saves the catalog team from writing descriptions by hand.

## Starter

Write your answers in a Markdown file (or directly in your notes) with this structure per candidate:

```markdown
### Candidate N — <name>

- **Value:** ...
- **Data:** ...
- **Error tolerance:** ...
- **Latency/cost:** ...
- **Verdict:** Build / Build a narrower version / Don't build — <one sentence why>
```

## Hints

- For Candidate 1, think about what "churn" would need to mean as a SQL-computable label — you did something very similar to this for `is_late` in Lecture 2. What column combination would define "hasn't ordered in a while and is unlikely to order again"? Is the second half of that ("unlikely to order again") even knowable from data you have, or is it closer to a guess dressed up as a prediction?
- For Candidate 2, separate the *drafting* task (an LLM writing prose from real numbers — fine) from the *sending with no review* part (an irreversible external communication with zero human check — this is where it fails).
- For Candidate 3 the data and value might genuinely be there, but reread the "Error tolerance" section of Lecture 2's grounding discussion: what's different about who's asking the question, and what happens if a *customer-facing* ungrounded or exploited answer leaks another customer's order data?
- For Candidate 4, this is the closest analog to the refund-approval example that failed the framework in Lecture 1 — read that row again before answering.
- For Candidate 5, weigh this one earnestly — of the five, it's the one most likely to genuinely clear all four questions as an advisory/review-and-approve feature. Say why, specifically.

## Expected outcome

A short written scoping document, five sections, each with a specific verdict and a justification that references the actual `crunchcycles` schema — not a generic AI-ethics essay. If your verdict is "don't build" or "build a narrower version" for a candidate, you should be able to point to exactly which of the four questions it fails and what would need to be different for that answer to change. Compare notes with a classmate or study partner if you have one — reasonable people can disagree on Candidates 1 and 5 in particular, as long as the reasoning is concrete.
