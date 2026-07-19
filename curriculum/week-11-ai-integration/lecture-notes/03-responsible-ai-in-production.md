# Responsible AI in Production

You now know how to wire a predictive model and an LLM into Crunch Cycles. This lecture is about the part that separates a feature you can defend from one that gets your system into the news for the wrong reason: what happens *around* the model call. Four things: who's allowed to act on the output, how you know if the output is any good, whether the feature treats people fairly, and how you keep it correct — or shut it off — for as long as it's live.

## Human-in-the-loop: three levels of autonomy

"Human in the loop" isn't a single switch. It's a spectrum, and picking the wrong point on it is the most common way a well-built model or LLM causes real harm despite working exactly as designed. Three useful levels:

| Level | What the human does | What the system does | Example at Crunch Cycles |
|---|---|---|---|
| **Advisory** | Sees the AI output alongside other information; free to ignore it | Surfaces a signal, takes no action | The risk score shown on a rep's order list — a number they can glance at or ignore |
| **Review-and-approve** | Must explicitly confirm before anything happens | Prepares a specific action, but does not execute it until approved | A "contact this customer about a delay" draft the copilot writes, that a rep edits and sends themselves |
| **Autonomous with override** | Can intervene, but the system acts by default | Executes automatically unless a human stops it | (Not built this week — see below) |

Lecture 1's error-tolerance question directly determines which level is appropriate. **Advisory** is right when a wrong signal costs almost nothing to ignore — the order-risk score is advisory: a rep who never looks at it has lost nothing beyond the visibility the feature was meant to add, and a false "high risk" flag costs two minutes of a rep glancing at an order that was fine. **Review-and-approve** is right when the AI's job is to save a human time on a task the human was going to do anyway, but where the *content* of the action matters — the copilot can draft a message to a customer, but a person reads it, corrects anything wrong, and hits send. **Autonomous with override** is right only when errors are cheap, frequent enough that a human reviewing every one isn't feasible, and reversible — think automatically re-sending a failed email notification, not cancelling an order or approving a refund. Neither of this week's two features reaches that bar, and that's a deliberate design choice, not an oversight: an order-risk score and a database Q&A answer both have consequences serious enough, and infrequent enough, that a human staying in the loop costs almost nothing and buys real protection.

### Building review-and-approve into the schema

Notice that `order_risk_scores` isn't just a log of predictions — it has `reviewed_by_user_id`, `review_decision`, and `reviewed_at` columns. That's not incidental. **Human-in-the-loop has to be a first-class part of your data model, not an afterthought bolted onto the UI.** If the review decision isn't stored, you can't prove a human looked at it, you can't measure how often reps agree with the model, and you can't build the evaluation feedback loop that tells you whether the model is actually helping.

```python
@app.route("/api/v1/orders/<int:order_id>/risk/review", methods=["POST"])
def review_risk_score(order_id):
    # `current_user` comes from Week 9's RBAC — only sales_rep, sales_manager, or admin may review
    if current_user.role not in ("sales_rep", "sales_manager", "admin"):
        return jsonify({"error": "not permitted"}), 403

    decision = request.json.get("decision")
    if decision not in ("proceed", "contact_customer", "expedite", "hold"):
        return jsonify({"error": "invalid decision"}), 400

    with engine.begin() as conn:
        conn.execute(text("""
            UPDATE order_risk_scores
            SET reviewed_by_user_id = :user_id, review_decision = :decision, reviewed_at = now()
            WHERE order_id = :order_id AND score_id = (
                SELECT score_id FROM order_risk_scores
                WHERE order_id = :order_id ORDER BY scored_at DESC LIMIT 1
            )
        """), {"user_id": current_user.user_id, "decision": decision, "order_id": order_id})

    return jsonify({"status": "recorded"})
```

The model never touches `review_decision` — only a human, acting through an authenticated, RBAC-checked endpoint, can write to it. That's the review-and-approve pattern made concrete: the model proposes, the schema records that a specific person disposed of the proposal, and nothing downstream treats the model's output as final on its own.

## Evaluating an AI feature

"It looked right when I tried it a few times" is not an evaluation. It's an anecdote. A real evaluation is a repeatable measurement against a fixed test set, and it looks different for a classifier than for an LLM feature — because they fail in different ways.

### Evaluating the predictive model: precision, recall, and calibration

You already saw `classification_report` and `roc_auc_score` in Lecture 2. Here's what those numbers actually mean for the order-risk model, and why you need more than one of them:

- **Precision** — of the orders the model flagged "high risk," what fraction actually shipped late? Low precision means reps waste time chasing orders that were fine — this is the cost of *false positives*.
- **Recall** — of the orders that actually shipped late, what fraction did the model flag? Low recall means the feature is missing the problems it was built to catch — this is the cost of *false negatives*.
- **Calibration** — when the model says "70% risk," do roughly 70% of orders at that score actually ship late? A model can rank orders correctly (good ROC-AUC) while its actual probability numbers are wrong — which matters if you ever show that percentage to a human and expect them to trust it as a number, not just a ranking.

There is no universal "good enough" threshold for these — it depends on what a false positive costs (a rep's two minutes) versus what a false negative costs (a support ticket and a possible reorder) at Crunch Cycles specifically. Deciding that trade-off is a product decision, informed by these numbers, not a purely technical one. Challenge 2 has you build this evaluation for real, including checking it against a held-out time period the model never saw during training — the only honest way to estimate how it'll perform going forward.

### Evaluating the LLM copilot: a golden set and a groundedness check

An LLM feature doesn't produce a single number you can threshold — it produces free text. You evaluate it against a **golden set**: a fixed list of realistic questions paired with the answer you'd consider correct, checked by hand against the actual database state.

```python
golden_set = [
    {
        "question": "What's the status of order 101?",
        "must_mention": ["101"],           # the answer must reference the specific order
        "must_not_claim": ["shipped"],      # order 101 is still open per the seed data — a wrong claim here is a real failure
    },
    {
        "question": "What's the status of order 999999?",
        "must_mention": ["no", "not found", "doesn't exist", "don't have"],  # a nonexistent order must be reported as such, never fabricated
    },
]

def check_groundedness(answer: str, case: dict) -> bool:
    text_lower = answer.lower()
    mentions_ok = any(term.lower() in text_lower for term in case.get("must_mention", []))
    no_wrong_claims = not any(term.lower() in text_lower for term in case.get("must_not_claim", []))
    return mentions_ok and no_wrong_claims
```

The second case in that golden set matters as much as the first: a good grounded-answer evaluation always includes questions about things that **don't exist** in the database. A copilot that confidently answers "order 999999 shipped on time" when no such order exists has failed the single most important test of a grounded system — it invented a fact instead of reporting that it had none. Build your golden set to specifically probe for this failure mode, not just to confirm the happy path works. Challenge 2 has you extend this into a real evaluation harness.

## Bias: accuracy and fairness are not the same thing

A model can be accurate on average and still unfair to a specific group. For Crunch Cycles' order-risk model, ask: **does the model's error rate differ by region, or by customer size?** If APAC orders are flagged "high risk" far more often than the base rate justifies — maybe because APAC has fewer historical orders in the training data, and the model is simply less confident there — reps in that region start ignoring the flag entirely, which defeats the feature for exactly the customers it might help most.

This isn't hypothetical for Crunch Cycles specifically: the training query in Lecture 2 includes `region_name` as a feature. If one region is under-represented in the training data (fewer past orders to learn from), the model's error rate for that region will typically be *higher*, not just noisier — worth checking directly, not assuming away.

```python
# after scoring the held-out test set (X_test, y_test, probs from Lecture 2)
eval_df = X_test.copy()
eval_df["actual_late"] = y_test.values
eval_df["predicted_risk"] = probs

for region, group in eval_df.groupby("region_name"):
    predicted_high = (group["predicted_risk"] >= 0.6)
    if predicted_high.sum() == 0:
        continue
    precision = (group.loc[predicted_high, "actual_late"]).mean()
    print(f"{region}: n={len(group)}, flagged={predicted_high.sum()}, precision={precision:.2f}")
```

If precision for one region is meaningfully worse than the others, that's a concrete, measurable fairness gap — not a vague worry, a number you can put in the responsible-AI note and act on (collect more data for that region, adjust the threshold per-region, or flag the region-specific uncertainty in the UI). The fix is never "don't measure it." An unmeasured bias doesn't go away; it just goes unnoticed until a customer or a regulator notices it for you.

For the LLM copilot, the equivalent risk is different in shape: a system prompt or retrieval design that works well for simple lookups ("what's the status of order X") but degrades badly for the kinds of questions certain roles ask more often — for example, if support staff tend to ask more open-ended, multi-step questions than sales reps do, and the tool-use retrieval design was only tested against simple ones, support staff get a worse experience from the same feature. Test your golden set with questions representative of every role that will actually use the copilot, not just the easiest ones to write.

## Monitoring and governing an AI feature over its lifetime

A model that was accurate on the day you shipped it is not guaranteed to stay accurate. Customer behavior changes, product mix changes, shipping partners change — this is called **drift**, and the only way to catch it is to keep measuring after launch, not just before.

### What to log, always

Both features already write everything needed for monitoring, because Lecture 2 built the logging in from the start rather than adding it later:

- `order_risk_scores` — every prediction, its model version, and (once a human reviews it) the human's decision. Compare `risk_band` against the order's *actual* outcome once it ships, on a rolling basis, and watch precision/recall over time, not just once at launch.
- `ai_query_log` — every question, what was retrieved, what was answered, and (once staff give feedback) whether it was helpful. A rising rate of `was_helpful = false`, or a spike in questions the retrieval tools can't answer, is your drift signal for the LLM feature.

### A minimal governance note

Every AI feature in production needs a short, standing document — not a thick policy binder, a one-page note anyone on the team can find. At minimum:

- **Owner** — who gets paged, who approves changes to the prompt or the model.
- **Intended use** — the exact scope this was built and evaluated for (e.g., "advisory risk signal for domestic B2B orders under $50,000" — not "predicts everything about every order").
- **Known limitations** — from your bias check and your evaluation: where does it perform worse, and does the UI communicate that?
- **Retraining / re-prompting cadence** — how often the model gets retrained on fresh data, or the system prompt gets revisited, and what triggers an off-cycle update (a measured accuracy drop, a new product category, a schema change).
- **Kill switch** — the specific, tested mechanism for turning the feature off and falling back to the manual process it replaced, and who has authority to pull it.

That last item is not optional. If you can't turn a feature off in minutes when it starts behaving badly — a config flag, an environment variable, a feature toggle in the database — you don't actually control it, you're just hoping it keeps working. A one-line change, like gating the `/api/v1/ask` route behind an `AI_COPILOT_ENABLED` flag checked at the top of the request, is often all it takes:

```python
import os

@app.route("/api/v1/ask", methods=["POST"])
def ask_copilot():
    if os.getenv("AI_COPILOT_ENABLED", "true").lower() != "true":
        return jsonify({"error": "The AI copilot is temporarily unavailable. Please ask a data analyst directly."}), 503
    # ... normal grounded-answer flow
```

Cheap to write, easy to forget, and the single most important line of code in the whole feature the day something goes wrong.

## Summary

- Human-in-the-loop is a spectrum — advisory, review-and-approve, autonomous-with-override — and the right level is determined by what a wrong output costs, per Lecture 1's error-tolerance question, not by how good the model tests out to be.
- Build the human's decision into your schema, not just your UI, so you can prove review happened and measure whether humans agree with the model over time.
- Evaluate a classifier with precision, recall, and calibration on a held-out test set; evaluate an LLM feature against a golden set that specifically includes cases the system should refuse to answer, not just cases it should get right.
- Check for bias by measuring error rates across meaningfully different groups (region, customer segment, staff role) — an unmeasured fairness gap doesn't disappear, it just surfaces later and worse.
- Log every prediction and every answer from day one, define a retraining/re-prompting cadence and what triggers it, and build and test a kill switch before you need it, not after.
