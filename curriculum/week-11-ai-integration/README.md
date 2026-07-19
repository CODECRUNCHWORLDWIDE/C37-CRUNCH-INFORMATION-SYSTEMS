# Week 11 — AI Integration

> **Goal:** by Sunday, Crunch Cycles has two AI features you can defend in front of a skeptical engineering manager: an order-risk model that flags likely-late orders for a human to triage, and a staff Q&A copilot that answers questions about customers and orders by retrieving real rows from the `crunchcycles` database and citing them — never by guessing. You can explain, in one paragraph, why each feature is scoped the way it is, what happens when the model or the LLM API is wrong, and who is accountable for the final decision.

Welcome back to **C37 · Crunch Information Systems**. For ten weeks you've built Crunch Cycles into a real system: a normalized PostgreSQL schema (Week 3), the SQL and pandas skills to query it (Week 4), automated processes on top of it (Week 5), an enterprise-shaped data model (Week 6), a REST API exposing it to the outside world (Week 7), a cloud deployment (Week 8), and authentication, RBAC, and a governance policy protecting it (Week 9). Every one of those weeks made the system *correct* and *safe*. This week asks a different question: where, if anywhere, does the system get *smarter* — and how do you add that intelligence without giving up the correctness and safety you just spent nine weeks building?

AI integration has a bad reputation in production systems, and it's earned. Teams bolt a chatbot onto a database, skip the part where they check whether the chatbot's answers are actually true, and ship a feature that confidently tells a customer the wrong shipping date. This week is about not doing that. You'll learn a concrete framework for deciding whether a feature is even a good candidate for AI (Lecture 1), how to wire a classical predictive model *and* a large language model into Crunch Cycles as real system components — called via API, grounded in your own SQL data, with latency and cost accounted for (Lecture 2) — and how to wrap both in the guardrails a production AI feature actually needs: a human in the loop for anything consequential, an evaluation that catches regressions before customers do, and a governance note that says who owns the feature and what happens when it's wrong (Lecture 3). By Saturday's mini-project, Crunch Cycles will have an AI decision-support feature — model or LLM, your choice — that a human reviews before anything happens, that you've evaluated against a real test set, and that comes with the one-page responsible-AI note a real audit would ask for first.

## Learning objectives

By the end of this week, you will be able to:

- **Identify** where AI genuinely adds value in an information system — and, just as importantly, name the shapes of feature where it does not — using a repeatable four-question framework (value, data, error tolerance, latency/cost) instead of "AI would be cool here."
- **Integrate** a classical predictive model (trained with scikit-learn on data pulled from PostgreSQL via pandas) as a feature in a running system, served through an API endpoint.
- **Integrate** a large language model (Claude, via the Anthropic API) as a feature, calling it the way a real backend calls any external service — with a client library, a model ID, and explicit error handling, not a browser tab.
- **Ground** an LLM's answers in your system's own data by retrieving rows from PostgreSQL first and passing them into the prompt (or exposing them as a tool the model calls) — so the model answers from your facts, not its training data or its imagination.
- **Design** human-in-the-loop decision support: distinguish a feature that *recommends* from a feature that *acts*, and build the UI/workflow seam where a human reviews an AI output before anything consequential happens.
- **Evaluate** an AI feature with a method that matches what it is — precision/recall/calibration for a classifier, a grounded-answer check against a golden Q&A set for an LLM feature — instead of "it looked right when I tried it."
- **Recognize and mitigate bias** in a predictive feature trained on your own operational data, and explain the difference between a model that is *inaccurate* and one that is *unfair*.
- **Monitor and govern** an AI feature over its lifetime: log every prediction and answer, track drift, define a retraining or re-prompting trigger, and — critically — build the kill switch that turns the feature off and falls back to the manual process it augmented.

## Prerequisites

- Weeks 1–9 of this course, or equivalent comfort with: a normalized PostgreSQL schema (Week 3), SQL joins/aggregation and pandas (Week 4), a Flask REST API (Week 7), a deployed system (Week 8), and role-based access control with the `app_users` table (Week 9).
- Python 3.10+ with `pip` working, and the PostgreSQL 16+ `crunchcycles` database from earlier weeks (this week's setup below reproduces the tables you need if you don't have them).
- An **Anthropic API key**. Create one at [console.anthropic.com](https://console.anthropic.com) if you don't have one — Lecture 2 and Exercise 2 use it. There's a free trial credit; nothing this week requires more than a few cents of usage if you follow the exercises as written.
- Comfort with basic `pandas` (`read_sql`, `DataFrame` operations) from Week 4. No prior machine learning or LLM experience assumed — this week teaches both from first principles.

## Setup

**1. Confirm your database.** If your `crunchcycles` PostgreSQL database from Weeks 3–9 is still around, you're set. If not, recreate the core schema (`regions`, `employees`, `customers`, `products`, `orders`, `order_items`) from [Week 4's README](../week-04-querying-data-with-sql-and-python/README.md#setup-load-the-week-3-schema), then the `app_users` table from [Week 9's README](../week-09-security-privacy-and-governance/README.md#setup), then continue below.

**2. Add this week's tables.** AI features need something to ground themselves in and somewhere to log what they did. Run this against `crunchcycles`:

```sql
-- Customer support interactions — grounding context for the Q&A copilot
CREATE TABLE support_tickets (
    ticket_id      SERIAL PRIMARY KEY,
    customer_id    INTEGER NOT NULL REFERENCES customers(customer_id),
    order_id       INTEGER REFERENCES orders(order_id),   -- NULL if not order-related
    subject        TEXT NOT NULL,
    body           TEXT NOT NULL,
    status         TEXT NOT NULL DEFAULT 'open',           -- 'open' | 'in_progress' | 'closed'
    created_at     TIMESTAMPTZ NOT NULL DEFAULT now(),
    closed_at      TIMESTAMPTZ
);

-- Output of the order-risk predictive model + the human review that follows it
CREATE TABLE order_risk_scores (
    score_id             SERIAL PRIMARY KEY,
    order_id             INTEGER NOT NULL REFERENCES orders(order_id),
    model_version        TEXT NOT NULL,
    risk_score           NUMERIC NOT NULL,      -- 0.0-1.0, predicted probability of a late/problem order
    risk_band            TEXT NOT NULL,         -- 'low' | 'medium' | 'high'
    scored_at            TIMESTAMPTZ NOT NULL DEFAULT now(),
    reviewed_by_user_id  INTEGER REFERENCES app_users(user_id),
    review_decision      TEXT,                  -- 'proceed' | 'contact_customer' | 'expedite' | 'hold'
    reviewed_at          TIMESTAMPTZ
);

-- Every question asked of the LLM copilot, what it retrieved, what it answered, and staff feedback
CREATE TABLE ai_query_log (
    query_id           SERIAL PRIMARY KEY,
    asked_by_user_id    INTEGER REFERENCES app_users(user_id),
    question            TEXT NOT NULL,
    retrieved_context    JSONB,      -- the SQL rows that grounded the answer
    answer               TEXT,
    was_helpful          BOOLEAN,    -- staff feedback, filled in after the fact
    asked_at             TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- A little seed data so the model has something to learn from and the copilot has something to answer
INSERT INTO support_tickets (customer_id, order_id, subject, body, status, created_at) VALUES
(1, 101, 'Where is my order?', 'It has been 9 days and I have not received a shipping notice.', 'open', now() - interval '2 days'),
(3, 104, 'Wrong item received', 'I ordered the carbon frame and received the aluminum one.', 'in_progress', now() - interval '5 days'),
(2, NULL, 'Bulk pricing question', 'Do you offer a discount for orders over 20 units?', 'closed', now() - interval '20 days');
```

**3. Install this week's Python dependencies.**

```bash
python3 -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install pandas scikit-learn sqlalchemy psycopg2-binary flask anthropic python-dotenv joblib
```

- **scikit-learn** — trains and serves the order-risk classifier (Lecture 2, Exercise 1).
- **anthropic** — the official Python SDK for calling Claude (Lecture 2, Exercise 2/3). Never call the API with raw `requests` when an official SDK exists.
- **python-dotenv** — loads `ANTHROPIC_API_KEY` from a local `.env` file that is never committed. Run `echo ".env" >> .gitignore` before creating it, exactly as you did for database secrets in Week 9.
- **joblib** — persists the trained scikit-learn model to disk so the API can load it without retraining on every request.

Set your key:

```bash
echo "ANTHROPIC_API_KEY=sk-ant-..." >> .env
```

## Weekly schedule

The schedule below adds up to approximately **28 hours** (the course's full-time pace).

| Day | Focus | Lectures | Exercises | Challenges | Quiz/Read | Homework | Mini-Project | Daily Total |
|-----------|------------------------------------------------|---------:|----------:|-----------:|----------:|---------:|-------------:|------------:|
| Monday | Where AI fits; scope a feature | 2h | 1h | 0h | 0.5h | 1h | 0h | 4.5h |
| Tuesday | Integrating a predictive model via API | 2h | 1.5h | 0h | 0.5h | 1h | 0h | 5h |
| Wednesday | Integrating an LLM; grounding in SQL data | 2h | 1.5h | 1h | 0.5h | 1h | 0h | 6h |
| Thursday | Human-in-the-loop design | 0h | 1h | 1.5h | 0.5h | 1h | 1h | 5h |
| Friday | Evaluation, bias, monitoring, governance | 0h | 0h | 1h | 0.5h | 1h | 1.5h | 4h |
| Saturday | Mini-project (ship the AI feature) | 0h | 0h | 0h | 0h | 0h | 2.5h | 2.5h |
| Sunday | Quiz + review | 0h | 0h | 0h | 1h | 0h | 0h | 1h |
| **Total** | | **6h** | **5h** | **3.5h** | **3.5h** | **5h** | **5h** | **28h** |

## How to navigate this week

Work top to bottom. Each piece assumes the ones above it.

| # | File | What's inside | ~Time |
|--:|------|---------------|------:|
| 1 | [lecture-notes/01-where-ai-fits-in-a-system.md](./lecture-notes/01-where-ai-fits-in-a-system.md) | Framing AI as a component; a four-question framework for scoping AI features; where AI does *not* belong | 2h |
| 2 | [lecture-notes/02-integrating-models-and-llms.md](./lecture-notes/02-integrating-models-and-llms.md) | Calling a scikit-learn model and Claude via API; grounding via retrieval; latency, cost, and failure handling | 2h |
| 3 | [lecture-notes/03-responsible-ai-in-production.md](./lecture-notes/03-responsible-ai-in-production.md) | Human-in-the-loop design; evaluation methods; bias; monitoring; governing an AI feature's lifecycle | 2h |
| 4 | [exercises/exercise-01-scope-an-ai-feature.md](./exercises/exercise-01-scope-an-ai-feature.md) | Apply the four-question framework to five candidate features for Crunch Cycles | 1h |
| 5 | [exercises/exercise-02-call-a-model-api.md](./exercises/exercise-02-call-a-model-api.md) | Call the Anthropic API directly; then train and serve the order-risk classifier | 1.5h |
| 6 | [exercises/exercise-03-ground-ai-in-your-data.md](./exercises/exercise-03-ground-ai-in-your-data.md) | Build the retrieval step and ground an LLM answer in real `crunchcycles` rows | 1.5h |
| 7 | [challenges/challenge-01-human-in-the-loop-design.md](./challenges/challenge-01-human-in-the-loop-design.md) | Design the review workflow for a risky, hard-to-reverse decision | 1.5h |
| 8 | [challenges/challenge-02-evaluate-an-ai-feature.md](./challenges/challenge-02-evaluate-an-ai-feature.md) | Build a real evaluation harness for both the classifier and the copilot | 1.5h |
| 9 | [mini-project/README.md](./mini-project/README.md) | Ship one AI decision-support feature end to end: API integration, grounding, HITL, evaluation, responsible-AI note | 2.5h |
| 10 | [homework.md](./homework.md) | Extra practice tying the week together | 5h |
| 11 | [quiz.md](./quiz.md) | 15 self-check questions + answer key | 1h |
| 12 | [resources.md](./resources.md) | Official docs, tools to install, and the few links worth your time | — |

## By the end of this week you can…

- Look at a proposed "let's add AI to X" feature request and say, concretely, whether it's a good candidate — and if not, why not.
- Wire a trained model and a hosted LLM into a running system as ordinary components: called via API, with timeouts, retries, and a fallback when they're unavailable.
- Ground an LLM's answer in your own database instead of letting it guess, and prove — with a citation back to real rows — that it did.
- Draw the exact line between "the AI recommends" and "the AI acts," and explain why that line has to be drawn on purpose, not discovered in an incident postmortem.
- Evaluate an AI feature the way you'd evaluate anything else in a production system: with numbers, a test set, and a monitoring plan — not a gut feeling.

## Up next

Week 12 — Capstone: design, cost, and defend a complete intelligent information system for an organization of your choosing, pulling together everything from data modeling through AI integration into one system you can present.

---

*Part of the Code Crunch Worldwide open curriculum · GPL-3.0 · If you find errors, please open an issue or PR.*
