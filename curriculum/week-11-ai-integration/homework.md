# Week 11 — Homework

Extra practice, spaced across the week, tying together scoping, integration, grounding, and responsible-AI thinking. Do these after the exercises; they assume your `crunchcycles` database, trained risk model, and `ask_crunchcycles` function all exist and work.

## 1. Score five more feature ideas (30 min)

Using Lecture 1's four-question framework, score these five additional Crunch Cycles feature ideas the same way Exercise 1 had you score the first five. Write one verdict and one justification sentence each:

1. "Predict which products are likely to go out of stock next month, based on order velocity."
2. "Have the LLM copilot draft responses to `support_tickets` for a human to review and send."
3. "Auto-flag `support_tickets` with 'urgent' language for faster routing."
4. "Let the LLM decide which sales rep gets assigned to a new inbound lead."
5. "Summarize a customer's full order history into a one-paragraph 'account brief' for a rep's first call."

## 2. Feature engineering practice (45 min)

The order-risk model in Lecture 2 used five features (`region_name`, `customer_tenure_days`, `line_item_count`, `total_units`, `order_value`, `distinct_categories`). Propose **three additional features** you could compute from the existing `crunchcycles` schema (`regions`, `employees`, `customers`, `products`, `orders`, `order_items`) that might help predict late shipping, and write the SQL to compute each one. For each, explain in a sentence why you think it might correlate with lateness — and flag if you're not sure it would (uncertain hypotheses are fine to include; guessing wrong and saying so is part of real feature engineering).

## 3. Read a real refusal (30 min)

Send `ask_crunchcycles` (or a simplified version) five deliberately unanswerable or out-of-scope questions of your own devising — not the ones from Exercise 3 or Challenge 2. Examples of the kind of question to write: something about a table you never exposed a tool for, something that requires comparing data across two systems you don't have, something ambiguously worded. For each, record: did the system correctly decline, or did it produce something confidently wrong? If it declined, was the decline message actually useful to the person asking (does it tell them what to do instead), or just a flat "I don't know"?

## 4. Cost estimate (30 min)

Using the token counts you printed in Exercise 2 Part A, estimate: if Crunch Cycles' 40 sales reps each ask the copilot an average of 5 questions per day, roughly how many tokens does that consume per month, and — checking current Claude pricing (see [resources.md](./resources.md)) — roughly what would that cost? Is that number small enough that cost isn't a meaningful constraint on this feature, or large enough that it should factor into your scoping decision? Show your arithmetic.

## 5. Write the postmortem you hope you never need (45 min)

Imagine, six months after shipping, the order-risk model's precision has quietly dropped from 70% to 40% — reps are now ignoring the "high risk" flag because it's wrong more often than not, and a genuinely late shipment to a major account slipped through uncaught as a result. Write a one-page incident postmortem: what data would you pull first to confirm the drop (name the actual table and query), what's the most likely cause (a few plausible hypotheses — did the product mix shift? did a new region launch with too little training data? did a shipping partner change?), and what's the very first action you'd take — including whether pulling the kill switch is warranted before you've even diagnosed the cause.

## 6. Bias audit on your own model (45 min)

Rerun Lecture 3's region-level precision breakdown on your own trained model from Exercise 2, but split by a **different** dimension this time: customer tenure (e.g., "new" customers signed up in the last 90 days vs. everyone else) instead of region. Do new customers get flagged "high risk" more or less often than the base rate, and does the model's precision differ meaningfully for that group? Write two or three sentences on what you find and whether it changes how you'd communicate the risk score's reliability to a rep working with a brand-new account.
