# Mini-Project — Ship an AI Decision-Support Feature

## Brief

Ship **one** AI decision-support feature into the Crunch Cycles system you've been building since Week 3 — either the order-risk model or the Q&A copilot from this week (or, if you completed Challenge 1, the credit-limit recommendation workflow). "Ship" means: it's a real endpoint on your Flask API, it's grounded in real `crunchcycles` data, a human reviews its output before anything consequential happens, you have evaluation numbers proving it works, and you have a one-page note a real engineering manager or auditor would accept as evidence you built this responsibly.

This is the week's synthesis. Every piece — the API integration from Lecture 2, the human-in-the-loop schema and endpoint from Lecture 3, the evaluation from Challenge 2 — comes together into one feature you could plausibly hand to a real team.

## Requirements

Your submission must include all five of the following, for **one** chosen feature:

### 1. API integration

- A real Flask route (extending your Week 7/8 API) that calls either your trained scikit-learn model or the Anthropic API — not a stub, not a hardcoded response.
- If you chose the LLM path: real grounding via retrieval or tool use, per Lecture 2 — no ungrounded LLM call that could plausibly hallucinate a fact about a real customer or order.
- If you chose the model path: the model is loaded once at process startup, not retrained or reloaded per request.
- Real error handling: a network failure, a missing order, or a rate limit returns a sensible response, not a stack trace.

### 2. Grounding in the PostgreSQL data

- Every fact the feature surfaces traces back to an actual `SELECT` against `crunchcycles` that you can show a reviewer. If you're using the LLM path, this means the retrieval step (direct query or tool-use call) is real and testable independent of the LLM call.

### 3. Human-in-the-loop wrapper

- A schema that records a human review decision (reuse `order_risk_scores`, extend it, or build the analogous table for your chosen feature) — `reviewed_by_user_id`, a decision field, and a timestamp, at minimum.
- An RBAC-protected endpoint (per Week 9's `app_users` roles) where a human records that decision — and only that endpoint can record it.
- A design note (a few sentences is fine) stating which of Lecture 3's three autonomy levels this feature sits at, and why.

### 4. An evaluation

- If you shipped the risk model: precision, recall, and ROC-AUC on a held-out test set (time-based, per Challenge 2, not just random), plus the region-level breakdown.
- If you shipped the copilot: a golden set of at least six questions (including at least two "should refuse" cases) with a pass/fail result for each.
- Real numbers, computed by running real code against your real database — not estimated or described in prose.

### 5. A responsible-AI note

One page, in Markdown, covering:

- **Owner** — who (a role, if not a name) is responsible for this feature.
- **Intended use** — the specific scope you built and evaluated this for.
- **Known limitations** — at least one concrete limitation surfaced by your evaluation (a region with weaker precision, a class of question the copilot handles poorly, a category of order your feature wasn't tested against).
- **Kill switch** — the actual mechanism (a config flag, an environment variable, a feature-flag row in a table) and confirmation that you tested it — turn the feature off, verify the system falls back cleanly to the prior manual behavior, turn it back on.

## Deliverables

- Working code: the Flask route(s), the model training script or the grounding/tool-use functions, the review-decision endpoint, and the SQL for any new or extended tables — organized however your project has organized code all term.
- The evaluation script(s) and their output (paste the actual printed numbers into your write-up, don't just describe them).
- `RESPONSIBLE-AI-NOTE.md` — the one-pager from requirement 5.
- A short `README.md` for this mini-project (2–3 paragraphs) explaining what you shipped, how to run it locally against your `crunchcycles` database, and what you'd build next if you had another week.

## Stretch goals

Pick zero or more, time permitting:

- **Ship both features**, with a shared `ai_query_log`-style logging discipline across them, and a combined responsible-AI note covering both.
- **Add a second LLM-backed feature**: a "draft a follow-up email" tool that takes a high-risk order and drafts (never sends) a customer-facing message referencing the real order data, gated behind the same review-and-approve pattern as Challenge 1's credit-limit workflow.
- **Simulate drift detection**: seed a batch of new orders with a deliberately different pattern (e.g., a new region, or a shift in typical order size) into `crunchcycles`, re-run your evaluation, and show — with real numbers — that your monitoring would have caught the model's performance degrading on the new pattern.
- **Wire the kill switch into the RBAC system** so only an `admin` role can flip it, and log who flipped it and when, the same discipline Week 9 applied to every other sensitive action in the system.
- **A/B the LLM system prompt**: run your golden set against two different system-prompt wordings and report which one produces a higher pass rate — a small, real taste of the prompt-iteration cycle a production LLM feature lives under.

## Grading rubric (self-check before you call it done)

- [ ] The feature is scoped per Lecture 1's framework, and you can state its four-question scoring in one sentence each.
- [ ] Nothing the feature surfaces is ungrounded — every fact traces to a real query.
- [ ] A human, identified by `user_id` through Week 9's RBAC, is the only thing that can finalize a consequential decision.
- [ ] The evaluation numbers are real, computed, and printed — not estimated.
- [ ] The kill switch exists, was tested, and is documented in the responsible-AI note.
- [ ] You could hand this to a new teammate and they'd understand what it does, why it's safe, and how to turn it off.
