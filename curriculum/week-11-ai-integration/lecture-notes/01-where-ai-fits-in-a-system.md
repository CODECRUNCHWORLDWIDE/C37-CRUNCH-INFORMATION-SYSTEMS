# Where AI Fits in a System

By the end of Week 9, Crunch Cycles was a real information system: a normalized database, a REST API, a cloud deployment, authentication and row-level security. Nothing about that system was "smart" — every response it gave was a deterministic function of a query and the rows that matched it. That is not a criticism. Determinism is a feature. When a customer asks "what's my order total," you want the same answer every time, computed the same way, traceable back to the rows that produced it.

This week you're going to add something non-deterministic to that system: a predictive model, a large language model, or both. Before you write a line of integration code, you need a clear-eyed answer to a harder question than "can we add AI here" — which is almost always yes — and that question is **should we, and exactly what should it do.**

## AI as a system component, not a system replacement

The single biggest framing mistake teams make is thinking of "adding AI" as adding a new system, when it should be adding a new **component** to the system you already have — one with the same obligations as every other component: a defined input, a defined output, a failure mode, an owner, and a cost.

Think back to Week 7's REST API. A `GET /api/v1/orders/{id}` endpoint has:

- **Input**: an order ID, a caller's auth token.
- **Output**: a JSON object, or a 404, or a 403.
- **Failure mode**: the database is down, the ID doesn't exist, the caller lacks permission.
- **Owner**: whoever's on call when it breaks.
- **Cost**: a few milliseconds of Postgres query time.

An AI feature needs the exact same profile, and — this is the part teams skip — it needs it *written down before you build it*, not discovered afterward. Here's the order-risk model from this week, specified the same way:

- **Input**: an `order_id`, and the rows in `orders`, `order_items`, `customers`, and `products` needed to compute its features.
- **Output**: a `risk_score` between 0 and 1, and a `risk_band` (`low` / `medium` / `high`).
- **Failure mode**: the model file fails to load, a feature is missing (e.g. a brand-new customer with no order history), the score is stale (the order changed after scoring).
- **Owner**: the person who trained it and who gets paged when its accuracy drifts.
- **Cost**: milliseconds to score, hours to retrain, and the ongoing cost of the training pipeline.

If you can't fill in all five of those for a proposed AI feature, you are not ready to build it — you're still ideating, and that's fine, but say so out loud rather than starting to write API-integration code.

## The four-question framework

Not every AI feature is worth building, and treating "the model would be smart" as sufficient justification is how teams end up with an unmaintained inference endpoint nobody trusts six months later. Before scoping any AI feature, answer four questions, in this order. If the answer to any of the first three is a clear "no," stop — the feature isn't ready, no matter how good the demo looks.

### 1. Value — does the manual process actually hurt?

AI is expensive to build and expensive to keep correct. It's only worth that cost when the thing it replaces or augments is genuinely costly: slow, error-prone, or bottlenecked on a scarce human. "Sales reps currently guess which of their 40 open orders are at risk of shipping late, and they guess wrong about a third of the time, and late orders cost us support tickets and reorders" is real value. "It would be neat if the dashboard had a chatbot" is not — it's a feature in search of a problem.

Ask: *if this manual process disappeared tomorrow with no replacement, would anyone notice, and how much would it cost the business?* If the honest answer is "not much," the feature doesn't clear the bar.

### 2. Data — do you have the facts a model or LLM needs?

A predictive model needs **labeled historical outcomes** — you can't predict "will this order ship late" without a column somewhere that records whether *past* orders shipped late. An LLM feature needs **retrievable, structured facts** to ground itself in — you can't answer "what's the status of order 4521" correctly without a `SELECT` that returns order 4521's actual status.

This is where Weeks 3–9 pay off directly: Crunch Cycles has a normalized schema with real history in it. If you were starting from nothing — no orders table, no ship-date history — you would need to build that data pipeline *first*, and that's a Week 3–4 problem, not a Week 11 problem. AI integration is the last step in the data lifecycle, not a substitute for the earlier ones.

### 3. Error tolerance — what happens when it's wrong?

Every model and every LLM will be wrong sometimes. The question that determines whether a feature is safe to build is: **what happens when it is?** Distinguish two shapes of consequence:

- **Reversible and low-stakes**: the risk model flags an order as "high risk" and it ships on time anyway. A rep spent two minutes checking on it for nothing. Annoying, cheap, recoverable.
- **Irreversible or high-stakes**: the system auto-cancels an order because a model scored it as fraud, and it wasn't. A real customer loses a real order, and undoing that costs more than the two minutes you saved.

The fix is almost never "make the model better" — models are never perfect — it's **matching the feature's autonomy to its error tolerance.** A feature with irreversible consequences needs a human in the loop, full stop, regardless of how accurate the model tests out to be. Lecture 3 is entirely about this line.

### 4. Latency and cost — does the budget fit the use case?

A classical model scored offline in a nightly batch job can take minutes; a risk score shown live on a rep's dashboard needs to return in well under a second. An LLM call typically costs somewhere between a fraction of a cent and a few cents and takes anywhere from under a second to several seconds depending on the model and how much it has to generate — fine for an on-demand staff Q&A tool, unacceptable if you tried to put it in the hot path of checkout at 500 requests per second. Know your latency and cost budget *before* you pick where in the request path the AI call goes, not after a load test surprises you.

## Applying the framework to five Crunch Cycles proposals

Here's how the framework plays out against feature requests you might plausibly get for Crunch Cycles. Work through these before Exercise 1, where you'll do this scoring yourself.

| Proposed feature | Value | Data | Error tolerance | Latency/cost | Verdict |
|---|---|---|---|---|---|
| **Order-risk scoring**: flag orders likely to ship late so a rep can follow up | High — late orders generate support tickets and reorders | High — years of `orders`/`order_items` history with real ship dates | Good — a false flag costs a rep two minutes; a missed flag is no worse than today | Cheap — batch-scored nightly, read from a table | **Build it.** This is this week's model feature. |
| **Staff Q&A copilot**: answer "what's the status of order X" / "which APAC customers haven't ordered in 90 days" in plain English | High — staff currently write ad hoc SQL or ping a data analyst | High — the whole `crunchcycles` schema is retrievable | Good, **if grounded** — an ungrounded answer that invents a shipping date is actively harmful; a grounded one that says "I don't have that information" is safe | Moderate — a few seconds and a few cents per question is fine for an internal tool used dozens of times a day | **Build it — grounding is not optional.** This is this week's LLM feature. |
| **Auto-approve refunds under $200 via an LLM reading the ticket text** | Moderate — saves support time | Moderate — ticket text is available, but "should this be refunded" is a judgment call, not a retrievable fact | **Bad** — an incorrect auto-approval is real money out the door, and it's not easily reversed once a refund posts | Feature isn't the bottleneck — the risk is | **Don't build the autonomous version.** A *draft-a-refund-recommendation-for-a-human-to-approve* version passes; a *the-LLM-clicks-approve* version doesn't, no matter the model's accuracy. |
| **Fully autonomous inventory reordering** — the system decides purchase quantities and places supplier orders with no review | High potential value | Moderate — you have sales history, but demand forecasting has real uncertainty and supplier lead times add risk | **Bad** — a bad autonomous reorder ties up real capital and warehouse space for weeks; wrong is expensive and slow to unwind | Not the constraint | **Don't build the autonomous version.** A recommendation queue a purchasing manager reviews weekly is the shape that clears error tolerance. |
| **Replace sales reps with a chatbot for enterprise accounts** | Framed as "value" but is actually a relationship-quality risk in a B2B business where reps carry account context | Weak — the thing that makes a good enterprise sales conversation isn't in any table | **Bad** — losing a six-figure account because a chatbot mishandled a negotiation is not a two-minute mistake | Not the constraint | **Don't build it.** This is AI applied to a problem shape — relationship management, judgment, negotiation — that AI is currently a poor fit for, independent of any model's benchmark scores. |

Notice the pattern: the two features Crunch Cycles is actually building this week both pass all four questions *and* their failure mode is cheap. The three rejected features aren't rejected because "AI can't do it" in some absolute sense — they're rejected because their failure mode is expensive and irreversible, and no accuracy number fixes that. That's the judgment this lecture is training: **scope the feature to match what a wrong answer costs, not to what the model can technically attempt.**

## What "adding AI" looks like in the Crunch Cycles architecture

Concretely, here is where each of this week's two features sits in the system you've already built:

```
                         ┌─────────────────────────┐
   Browser/staff client ─▶  Flask REST API (Wk 7/8) │
                         │  + RBAC / RLS (Wk 9)     │
                         └─────────┬────────┬───────┘
                                   │        │
                    ┌──────────────┘        └──────────────┐
                    ▼                                       ▼
       ┌────────────────────────┐            ┌────────────────────────────┐
       │ /api/v1/orders/{id}/risk│            │ /api/v1/ask                │
       │  reads order_risk_scores│            │  1. retrieve rows from      │
       │  (written nightly by a  │            │     crunchcycles (SQL)      │
       │  batch training/scoring │            │  2. call Claude with the    │
       │  job — Lecture 2)        │            │     retrieved rows as        │
       └────────────────────────┘            │     grounding (Anthropic API)│
                                               │  3. log to ai_query_log      │
                                               └────────────────────────────┘
```

Both features are **new endpoints on the API you already built**, backed by **new tables in the database you already normalized**, protected by **the RBAC you already wrote in Week 9**. AI integration, done well, doesn't introduce a parallel system — it extends the one you have, with the same discipline about inputs, outputs, and ownership you've applied to every component since Week 1.

## Summary

- Treat an AI feature as a system component with the same five properties as any other: input, output, failure mode, owner, cost.
- Score every proposed AI feature against four questions, in order: value, data, error tolerance, latency/cost. Stop if the first three don't clear the bar.
- The determining factor for whether autonomy is safe is almost never "how accurate is the model" — it's "what does a wrong answer cost, and can it be undone." Match the feature's autonomy to that cost, not to the model's benchmark score.
- Some feature shapes — irreversible financial actions, high-stakes judgment calls, relationship-dependent work — are currently poor fits for AI independent of any specific model's quality. Recognizing that shape up front saves you from building something you'll have to walk back.
- The two features you're building this week — order-risk scoring and a grounded Q&A copilot — both passed this framework on purpose. You'll re-derive that scoring yourself in Exercise 1.
