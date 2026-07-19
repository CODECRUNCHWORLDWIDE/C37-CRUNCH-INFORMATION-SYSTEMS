# Challenge 1 — Design Human-in-the-Loop for a Risky Decision

**Goal:** design and implement a full review-and-approve workflow for a decision meaningfully riskier than the order-risk score — one where getting the autonomy level wrong has a real cost — and defend every design choice against Lecture 3's framework.

## The scenario

Crunch Cycles' finance team wants AI help with a decision Exercise 1 already flagged as dangerous to automate: **recommending a credit limit for new B2B accounts requesting net-30 payment terms.** Unlike the order-risk score (advisory, cheap to be wrong about), an incorrect credit decision is either a lost sale (limit set too low) or real financial exposure (limit set too high and the customer defaults). This is exactly the shape of decision Lecture 3 says needs review-and-approve, not advisory, and definitely not autonomous.

You are going to build the reviewed version — not the model itself in full production quality, a plausible one is fine — but the **workflow around it**, which is the actual point of this challenge.

## Requirements

1. **A model or a rule-based scoring function** that takes a prospective customer's available signals — you decide which columns from `customers`, `orders`, and `order_items` are legitimate inputs — and outputs a recommended credit limit and a confidence level. It's fine if this is simpler than the order-risk model (even a documented heuristic is acceptable) — the workflow is what's graded, not the sophistication of the number it's built around.
2. **A new table**, `credit_limit_recommendations`, that records: the customer, the recommended limit, the confidence level, the model/heuristic version, timestamps, and — critically — `reviewed_by_user_id`, `approved_limit` (which may differ from the recommendation), and `reviewed_at`. Design the exact columns yourself; justify each one in a sentence.
3. **An API endpoint** (extending your Week 7/9 Flask app) that:
   - Only a `finance` or `admin` role (per Week 9's RBAC) can call to *approve* a recommendation.
   - Returns a `403` for any other role attempting to approve — and returns the pending recommendation (read-only) for roles that should be able to see it but not approve it. Decide who that is and justify it.
   - **Never** applies the recommended limit directly to any account-level table without a row appearing first in `reviewed_by_user_id` / `approved_limit`. Write a short comment in the code explaining why this ordering is enforced, not just implemented.
4. **A written design note** (a few paragraphs) answering:
   - Where on Lecture 3's three-level spectrum does this feature sit, and why is that the right level — argue it from the actual cost of a wrong decision, not from "credit limits are just risky in general."
   - What would have to be true about this feature's track record before it would be reasonable to consider moving it toward more autonomy (e.g., "autonomous below some threshold, review-required above it")? Be specific about what evidence you'd want to see, referencing the kind of metrics from Lecture 3's evaluation section.
   - What's the very first thing that should happen if finance reports that approved credit limits are correlating with a rising default rate three months from now? Name the specific table you'd query and the specific action (retrain, adjust threshold, kill switch) that finding would justify.

## Hints

- This is deliberately more open-ended than the exercises. If you're stuck on what counts as a "legitimate input" for requirement 1, think about what a human financial analyst would actually be allowed to consider — and notice that some tempting columns (e.g., anything correlated with a customer's home country in a way that functions as a proxy for demographic characteristics) might be inputs you should explicitly justify excluding, even if they'd technically improve the model's accuracy.
- For requirement 3, look back at Lecture 3's `review_risk_score` endpoint — the ordering discipline ("only a human write ever sets the final value") is the exact pattern you're extending, just with real money attached this time.
- Don't over-build the model. A team that spends all its time perfecting the credit-scoring math and none of it on the review workflow has, ironically, reproduced the exact mistake this challenge is testing for.

## Expected outcome

A working `credit_limit_recommendations` table, a Flask endpoint enforcing RBAC on the approval step, a demonstration (a short script or a couple of `curl` calls) showing that a non-finance role is rejected and a finance role can approve, and a written design note that references specific numbers and specific tables rather than general principles. A reviewer should be able to trace, from your code alone, that no credit limit ever takes effect without a specific human's ID attached to the decision.
