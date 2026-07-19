# Challenge 2 — Build an Evaluation for an AI Feature

**Goal:** build a real, runnable evaluation harness for both of this week's AI features — one that produces the specific numbers Lecture 3 named, computed against your actual data, not described in prose.

## Part A — evaluate the order-risk classifier on a time-based holdout

Exercise 2's `train_test_split` used a **random** split. That's a reasonable first pass, but it has a subtle flaw for a model that will run in production going forward: a random split can leak information from the future into training (an order from last week can end up in the training set while an order from two months ago ends up in the test set), which makes your test metrics look better than the model will actually perform once deployed. A **time-based holdout** — train on everything before a cutoff date, test on everything after — is the honest way to estimate forward-looking performance.

1. Rewrite the train/test split from Exercise 2 to be time-based: train on orders with `order_date` before some cutoff (pick a date that leaves you a reasonable-sized test set — print both set sizes and adjust if one is too small), test on everything after.
2. Retrain the model on this split and report `classification_report` and `roc_auc_score` again. Compare the numbers to your random-split results from Exercise 2 — are they meaningfully different? Write one or two sentences on what you observe and why a time-based split might differ from a random one for this specific label.
3. **Compute the region-level precision breakdown** from Lecture 3's bias-check code, on this time-based test set. Report precision per region at your chosen risk threshold, and the count of orders each region contributed to the test set. If any region has too few orders to compute a meaningful precision number, say so explicitly rather than reporting a misleading number from three data points.
4. Pick a `risk_band` threshold (Lecture 2 used 0.6/0.3 as a starting example) and justify it in a sentence: given your precision and recall at that threshold, is it the right trade-off for Crunch Cycles, where a false positive costs a rep a couple of minutes and a false negative costs a support ticket and possible reorder?

## Part B — build a golden-set evaluation for the Q&A copilot

Extend Exercise 3's three-question test into a real golden set of **at least eight questions**, covering:

- At least two questions with a clear, verifiable correct answer (like Exercise 3's Question 1).
- At least two questions that require the aggregation tool (like Exercise 3's Question 2).
- At least two "should refuse" questions — nonexistent orders, nonexistent customers, or questions genuinely outside what your tools can look up (e.g., asking about a table you didn't expose a tool for).
- At least two questions of your own design, chosen because you think they're likely to break the system in an interesting way — say why you chose them.

Write a script that runs every question in your golden set through `ask_crunchcycles`, applies a pass/fail check per question (extend `check_groundedness` from Lecture 3, or write your own per-question checks — some questions may need a custom check rather than a generic keyword match), and prints a summary: how many passed, which ones failed, and — for each failure — the actual answer the model gave versus what should have happened.

## Part C — write up the results

A short report (a Markdown file is fine) covering:

- The order-risk model's precision/recall/ROC-AUC on the time-based holdout, the region-level precision table, and your threshold justification from Part A.
- The golden-set pass rate for the copilot, and a specific discussion of every failure — what went wrong, and what you'd change (system prompt wording, an additional tool, a stricter refusal instruction) to fix it.
- One paragraph: if you had to ship only one of these two features this week and hold the other back, which would you ship first, and why — argue from the evaluation numbers you just produced, not from general preference.

## Hints

- If your `crunchcycles` seed data is small (it will be, unless you've been adding rows across the term), a time-based holdout might leave you with very few test examples in some regions. That's a real, reportable finding, not a bug in your evaluation — say so in Part A rather than hiding it.
- For Part B's "questions you think will break it" — good candidates include questions that combine two conditions your tools weren't designed for together, ambiguous phrasing that could refer to more than one thing, or a question about a real order that belongs to a *different* customer than the one asking (if your system has any notion of "asking on behalf of," test whether it leaks). Trying to break your own feature before a user does is a core evaluation skill, not just an exercise requirement.

## Expected outcome

Two runnable evaluation scripts (or one combined script with two clear sections) that print real numbers when you run them against your database — not hypothetical numbers in a write-up. A Markdown report that a non-technical stakeholder could read and understand what's working, what isn't, and what you'd fix first. If every single question in your golden set passes on the first try, go back and add harder questions — a golden set with a 100% pass rate on the first attempt usually means it isn't testing anything hard enough to be useful.
