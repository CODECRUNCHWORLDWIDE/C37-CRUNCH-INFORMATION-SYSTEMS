# Exercise 3 — Ground an LLM Answer in SQL-Stored Data

**Goal:** build the full retrieval-then-generation loop from Lecture 2, including the tool-use exchange that lets Claude request data instead of you guessing what to retrieve — and prove, with a concrete test, that the model refuses to answer when the data doesn't support an answer.

## Background

By the end of this exercise you'll have a function, `ask_crunchcycles(question: str) -> str`, that a staff member could plausibly type a real question into and trust the answer. "Trust" here has a precise meaning: every fact in the answer traces back to a row your code actually retrieved from `crunchcycles`, and if no such row exists, the function says so instead of guessing.

## Task

### Step 1 — implement the two tools

Using [Lecture 2's tool-use pattern](../lecture-notes/02-integrating-models-and-llms.md#grounding-with-tool-use-letting-claude-request-the-retrieval) as your starting point, implement both `lookup_order` and `find_customers_by_region_and_min_spend` as real, tested Python functions against your `crunchcycles` database. Confirm each one works standalone before wiring up the model — call `execute_tool(engine, "lookup_order", {"order_id": 101})` directly and check the output against `psql`.

### Step 2 — drive the tool-use loop to completion

Claude's first response to an open-ended question won't be the final answer — it'll be a request to call a tool. Your code has to execute that tool, send the result back, and loop until Claude produces a text answer instead of another tool request:

```python
def ask_crunchcycles(client, engine, question: str) -> str:
    messages = [{"role": "user", "content": question}]
    system_prompt = (
        "You are a support assistant for Crunch Cycles staff. You have tools to look up "
        "real order and customer data. Use them before answering any question about "
        "specific orders, customers, or spend. If a tool returns no data, or the question "
        "asks about something outside what your tools can look up, say so explicitly — "
        "never guess, estimate, or invent an order status, date, or dollar amount."
    )

    while True:
        response = client.messages.create(
            model="claude-opus-4-8",
            max_tokens=1024,
            system=system_prompt,
            tools=tools,
            messages=messages,
        )

        if response.stop_reason != "tool_use":
            return next(b.text for b in response.content if b.type == "text")

        messages.append({"role": "assistant", "content": response.content})
        tool_results = []
        for block in response.content:
            if block.type == "tool_use":
                result = execute_tool(engine, block.name, block.input)
                tool_results.append({
                    "type": "tool_result",
                    "tool_use_id": block.id,
                    "content": str(result),
                })
        messages.append({"role": "user", "content": tool_results})
```

### Step 3 — log every question

Every call to `ask_crunchcycles` should write a row to `ai_query_log` — the question, the retrieved context (you'll need to capture the tool results as you go, not just the final text), and the answer. This is not optional polish; it's what makes Lecture 3's monitoring and Challenge 2's evaluation possible. Extend the function to build up a list of everything retrieved and insert it as `retrieved_context` (cast to JSON) alongside the question and answer.

### Step 4 — test three questions, one of which should fail gracefully

Run all three of these through `ask_crunchcycles` and read the actual output for each:

1. `"What's the status of order 101?"` — should retrieve and correctly report the real status from your seed data.
2. `"Which customers in North America have spent more than $1,000 since the start of the year?"` — should use the second tool and return real company names, not a guess.
3. `"What's the status of order 999999?"` — this order does not exist. **This is the important test.** Read Lecture 2's grounding section again if you're not sure why: does the model correctly report that it found no such order, or does it produce a plausible-sounding fabricated status anyway?

## Expected outcome

- Questions 1 and 2 return specific, correct facts that you can independently verify against `psql` — company names, real statuses, real dollar amounts.
- Question 3 returns an explicit "no such order" answer, **not** a fabricated status. If it fabricates one, that's a real failure of your system prompt or tool design, not an acceptable quirk — go back and tighten the system prompt's instruction about what to do when a tool returns no data, and re-test until it reliably refuses.
- `ai_query_log` has three new rows after running the test, each with a non-null `question`, `retrieved_context`, and `answer`.
- You can explain, in one sentence, why `execute_tool` — not the model — is the only thing in this system that ever constructs a SQL query.
