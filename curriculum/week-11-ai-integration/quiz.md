# Week 11 — Quiz

Fifteen questions. Lectures closed. Aim for 12/15 before starting Week 12. The answer key explains the *why*, not just the letter.

---

**Q1.** Per Lecture 1's four-question framework, which question should you answer *first*, before the other three?

- A) Latency/cost
- B) Error tolerance
- C) Value — does the manual process this replaces actually hurt?
- D) Data — do you have labeled outcomes or retrievable facts?

---

**Q2.** A proposed feature would auto-approve customer refunds under $200 based on an LLM reading the ticket text, with no human review. Per Lecture 1, why does this fail the framework regardless of how accurate the LLM tests out to be?

- A) LLMs cannot read plain text
- B) The action is irreversible and the failure mode (money out the door) is expensive, so the error-tolerance question fails independent of accuracy
- C) Refund amounts can't be represented in a database
- D) $200 is too large a dollar amount for any AI feature

---

**Q3.** In the order-risk model's feature query, why is `ship_date` dropped from the feature set after being used to compute the `is_late` label?

- A) `ship_date` is not a valid PostgreSQL column type
- B) Using it as a feature would leak the outcome into the inputs — the model would effectively see its own answer during training
- C) `ship_date` is always NULL
- D) scikit-learn cannot process date columns

---

**Q4.** `class_weight="balanced"` is passed to `LogisticRegression` in Lecture 2. What problem does it address?

- A) It speeds up training
- B) It compensates for a class-imbalanced label, so the model doesn't just learn to always predict the majority class
- C) It normalizes numeric features to the same scale
- D) It prevents the model from using categorical features

---

**Q5.** Why does Lecture 2 load the trained model file **once**, at Flask app startup, rather than reloading it inside the request handler?

- A) `joblib.load` only works once per process
- B) Reloading from disk on every request adds unnecessary latency and I/O for a model that doesn't change between retrains
- C) Flask does not allow global variables
- D) The model file is deleted after the first load

---

**Q6.** What does "grounding" an LLM answer mean, in this week's sense?

- A) Turning off extended thinking
- B) Retrieving real data from your own system and providing it to the model before generation, so the answer is based on retrieved facts rather than the model's training data or invention
- C) Lowering the model's temperature parameter to zero
- D) Running the model on local hardware instead of via API

---

**Q7.** In the tool-use grounding pattern, who or what ever constructs and executes the raw SQL query?

- A) Claude writes and executes the SQL directly
- B) The user's browser
- C) Your own application code (`execute_tool`), using parameterized queries — Claude only supplies structured tool-call arguments, never SQL text
- D) The PostgreSQL query planner infers it automatically

---

**Q8.** Why does Exercise 3 specifically require testing a question about a **nonexistent** order (`order 999999`)?

- A) To test the database's error-logging system
- B) Because verifying the system correctly refuses to answer when there's no supporting data is the most important test of a grounded system — an LLM that fabricates a status for a nonexistent order has failed the core promise of grounding
- C) To measure API latency under error conditions
- D) Nonexistent order IDs cause the Anthropic API to return a 500 error

---

**Q9.** Per Lecture 3, which of the three human-in-the-loop levels is the order-risk score, as built this week?

- A) Autonomous with override
- B) Review-and-approve
- C) Advisory
- D) Fully automated

---

**Q10.** Why does `order_risk_scores` include `reviewed_by_user_id`, `review_decision`, and `reviewed_at` as actual table columns, rather than just showing the risk score in a UI and leaving review undocumented?

- A) PostgreSQL requires every table to have at least six columns
- B) So a human's review decision is recorded as first-class data — provable, auditable, and usable later to measure whether staff agree with the model
- C) To satisfy a foreign key constraint on `orders`
- D) It has no functional purpose beyond documentation

---

**Q11.** A classifier has high ROC-AUC (it ranks late orders above on-time ones well) but poor calibration (when it says "70% risk," the true late rate at that score is actually 40%). What does this tell you?

- A) The model is useless and should be discarded
- B) ROC-AUC and calibration measure different things — the model may be good at *ranking* orders by relative risk while its actual probability numbers are not trustworthy as stated percentages
- C) The model needs more training data and nothing else
- D) This combination is impossible — ROC-AUC and calibration always move together

---

**Q12.** Why does Challenge 2 recommend a **time-based** train/test split over a random split for evaluating the order-risk model before deployment?

- A) Time-based splits are always faster to compute
- B) A random split can let information from the future leak into training, making test metrics look better than the model will actually perform once deployed on genuinely new, future orders
- C) PostgreSQL requires data to be queried in date order
- D) scikit-learn does not support random splits on this kind of data

---

**Q13.** A bias check finds that the order-risk model's precision for APAC orders is 35%, versus 70%+ for other regions, when flagging "high risk." What does this most directly suggest as a next step, per Lecture 3?

- A) Nothing — precision differences across groups are expected and not worth investigating
- B) Immediately shut down the entire feature for all regions
- C) Investigate why (e.g., APAC may be under-represented in training data), and consider region-specific handling (more data, an adjusted threshold, or surfacing the uncertainty) rather than treating the flag as equally reliable everywhere
- D) Retrain the model using only APAC data and discard the rest

---

**Q14.** What is the *specific* purpose of the "kill switch" concept from Lecture 3, as illustrated by the `AI_COPILOT_ENABLED` environment-variable check?

- A) To permanently delete the trained model
- B) To provide a tested, fast mechanism for disabling an AI feature and falling back to the manual process it replaced, without requiring a code deploy
- C) To automatically retrain the model every night
- D) To block a specific user from ever accessing the system again

---

**Q15.** Which of the following best distinguishes evaluating a *classifier* feature from evaluating an *LLM* feature, per this week's material?

- A) Classifiers cannot be evaluated at all; only LLMs can
- B) A classifier is evaluated with numeric metrics (precision, recall, calibration) against a held-out test set with known labels; an LLM feature is evaluated against a golden set of questions with expected behaviors, including cases it should explicitly refuse to answer
- C) Both are evaluated identically, using ROC-AUC
- D) LLM features cannot be evaluated because their outputs are free text

---

## Answer key

<details>
<summary>Reveal after attempting</summary>

1. **C** — Value comes first. If the manual process doesn't genuinely hurt (isn't slow, error-prone, or bottlenecked), there's no reason to pay the cost of building AI around it, regardless of how the other three questions score.
2. **B** — The framework's error-tolerance question is about the *cost and reversibility* of a wrong output, not the model's accuracy. An irreversible financial action fails this regardless of accuracy.
3. **B** — `ship_date` is used to compute the label (`is_late`) and then must be excluded from the features, or the model would be training on (a version of) its own answer — label leakage.
4. **B** — It compensates for class imbalance, so the model doesn't just default to predicting the majority class ("not late") for everything and call that high accuracy.
5. **B** — Reloading a model file from disk on every request adds unnecessary latency and I/O for a model that only changes when explicitly retrained.
6. **B** — Grounding means retrieving real facts from your own system before generation, so the model's answer is based on what was retrieved, not on training data or invention.
7. **C** — Your own application code executes parameterized queries. Claude only ever supplies structured tool-call arguments (JSON), never raw SQL text, by design.
8. **B** — Testing the refusal case for a nonexistent order is the most important groundedness test — it directly checks whether the system fabricates facts when it has none, which is the central risk grounding is meant to prevent.
9. **C** — Advisory. A rep can see the score or ignore it; nothing happens automatically, and a wrong flag costs almost nothing.
10. **B** — Recording review as data (not just UI) makes it provable, auditable, and usable to measure agreement between staff and the model over time — a prerequisite for real monitoring.
11. **B** — ROC-AUC measures ranking quality; calibration measures whether the stated probabilities are trustworthy as percentages. A model can be good at one and weak at the other.
12. **B** — A random split can leak future information into training, inflating test metrics relative to genuine forward-looking performance once deployed.
13. **C** — The right response to a measured fairness gap is to investigate the cause and consider a targeted fix (more data, adjusted threshold, surfaced uncertainty) — not to ignore it, and not to overreact by discarding the feature or the other regions' data entirely.
14. **B** — The kill switch's specific purpose is a fast, tested way to disable the feature and fall back to the prior manual process, without needing a code deploy — critical when something is actively going wrong.
15. **B** — Classifiers get numeric metrics against known labels; LLM features get evaluated against a golden set of expected behaviors, explicitly including cases where the correct behavior is to refuse.

</details>

**Scoring:** 12+ → start Week 12. 9–11 → re-read the lecture sections behind your misses. <9 → re-read all three lectures from the top before continuing; this week's concepts are load-bearing for the capstone.
