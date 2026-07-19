# Exercise 2 — Call a Model/LLM API from the System

**Goal:** make your first real call to the Anthropic API from Python, then train, evaluate, and persist the order-risk classifier from Lecture 2 against your own `crunchcycles` data.

## Part A — a working Anthropic API call

Before building anything, confirm your API key and client are wired up correctly.

1. Confirm `ANTHROPIC_API_KEY` is set (from this week's [README setup](../README.md#setup)):

   ```bash
   python3 -c "import os; from dotenv import load_dotenv; load_dotenv(); print('set' if os.getenv('ANTHROPIC_API_KEY') else 'MISSING')"
   ```

2. Write `test_claude.py`:

   ```python
   import os
   from dotenv import load_dotenv
   import anthropic

   load_dotenv()
   client = anthropic.Anthropic()

   response = client.messages.create(
       model="claude-opus-4-8",
       max_tokens=256,
       messages=[{
           "role": "user",
           "content": "In one sentence, explain why an information system should never let an LLM write raw SQL.",
       }],
   )

   for block in response.content:
       if block.type == "text":
           print(block.text)

   print("\n--- usage ---")
   print(f"input tokens:  {response.usage.input_tokens}")
   print(f"output tokens: {response.usage.output_tokens}")
   ```

3. Run it. You should get a real, on-topic answer, plus a token count. Note the token counts — this is the number that determines cost, and you'll use it again in Challenge 2's evaluation.

## Part B — train the order-risk classifier

Using the feature-extraction query and training code from [Lecture 2](../lecture-notes/02-integrating-models-and-llms.md#pattern-1-a-predictive-model-trained-on-your-own-sql-data), build `train_risk_model.py` that:

1. Pulls features and the `is_late` label from `crunchcycles` with the feature query.
2. Prints the shape of the resulting DataFrame and the label balance (`df["is_late"].mean()`).
3. Splits into train/test with `stratify=y` (this matters — without it, an unlucky split can leave your test set with almost no positive examples, and every metric downstream becomes meaningless).
4. Trains the `LogisticRegression` pipeline exactly as shown in Lecture 2.
5. Prints `classification_report` and `roc_auc_score` on the test set.
6. Saves the model bundle with `joblib.dump({"model": model, "version": "order-risk-v1", "trained_at": <today's date>}, "order_risk_model.joblib")`.

## Part C — serve one prediction

Write a small script (not the full Flask app yet — that's the mini-project) that loads the saved model and scores a single real order from your database:

```python
import joblib
import pandas as pd
from sqlalchemy import create_engine, text

engine = create_engine("postgresql://localhost/crunchcycles")
bundle = joblib.load("order_risk_model.joblib")

# pick any real order_id from your database
order_id = 101

query = text("""
    SELECT r.region_name,
           (o.order_date - c.signup_date) AS customer_tenure_days,
           COUNT(oi.product_id)            AS line_item_count,
           SUM(oi.quantity)                AS total_units,
           SUM(oi.quantity * oi.unit_price) AS order_value,
           COUNT(DISTINCT p.category)      AS distinct_categories
    FROM orders o
    JOIN customers c ON c.customer_id = o.customer_id
    JOIN regions r ON r.region_id = c.region_id
    JOIN order_items oi ON oi.order_id = o.order_id
    JOIN products p ON p.product_id = oi.product_id
    WHERE o.order_id = :order_id
    GROUP BY r.region_name, o.order_date, c.signup_date
""")

with engine.connect() as conn:
    row = conn.execute(query, {"order_id": order_id}).mappings().first()

features = pd.DataFrame([dict(row)])
risk_score = float(bundle["model"].predict_proba(features)[0, 1])
print(f"Order {order_id}: risk_score={risk_score:.3f}")
```

## Expected outcome

- Part A prints a real Claude-generated sentence and a nonzero token count. If you get an `anthropic.AuthenticationError`, your key isn't loaded — recheck `.env` and that `load_dotenv()` runs before the client is constructed.
- Part B prints a label balance (probably somewhere between 5% and 30% "late," depending on your seed data — if it's exactly 0% or 100%, your `is_late` CASE expression or your data has a problem, not your model), a classification report, and an ROC-AUC score above 0.5 (0.5 means "no better than a coin flip" — if you're at or below that, something in feature construction or the train/test split is broken; revisit before moving on).
- Part C prints a single risk score between 0 and 1 for a real order. Try it against two or three different order IDs and sanity-check: does a large, first-time-customer order score higher than a small order from a long-tenured customer? It won't be perfect, but the direction should make intuitive sense.
