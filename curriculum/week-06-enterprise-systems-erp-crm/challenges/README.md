# Week 6 — Challenges

Two open-ended problems. Unlike the exercises, these have **no single right answer** — the skill is judgment and defensible reasoning, backed by real numbers you compute in SQL and Python, never a spreadsheet. Do them after the three exercises.

1. **[Challenge 1 — Build vs. buy](challenge-01-build-vs-buy.md)** — a mid-size firm is deciding whether to buy an ERP or build one; you model the real 5-year cost of each path in Python and argue a recommendation. *(~2 hours.)*
2. **[Challenge 2 — Reconcile duplicate masters](challenge-02-reconcile-duplicate-masters.md)** — Crunch Cycles' `customers` table has accumulated duplicates the way real companies' do; you design and script a reconciliation in SQL. *(~2 hours.)*

## How these are judged

There's no answer key with row counts. Instead, each challenge tells you what a *strong* submission looks like. You're being graded on:

- **Correctness of reasoning** — can you justify the numbers and decisions you produced?
- **Explicit assumptions** — did you write down every assumption your model or fix depends on?
- **Data-tooling discipline** — every calculation lives in SQL and/or Python (pandas). No spreadsheet, ever, even as scratch work — that's the whole point of this course's data rule, and it's especially relevant this week because "build vs. buy" and "data cleanup" are exactly the workflows people are tempted to reach for Excel on.

Keep your work in `challenge-01/` and `challenge-02/` directories with your code **and** your written reasoning. The reasoning is the point.
