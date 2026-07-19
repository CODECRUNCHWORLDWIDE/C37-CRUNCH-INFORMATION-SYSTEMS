# Week 11 Exercises

Three exercises, meant to be done in order — each builds on the previous one's output.

| # | Exercise | Builds | Depends on |
|---|----------|--------|------------|
| 1 | [exercise-01-scope-an-ai-feature.md](./exercise-01-scope-an-ai-feature.md) | A written scoping decision for five candidate features, using Lecture 1's four-question framework | Lecture 1 |
| 2 | [exercise-02-call-a-model-api.md](./exercise-02-call-a-model-api.md) | A working Anthropic API call, then a trained + persisted order-risk classifier | Lecture 2, an `ANTHROPIC_API_KEY` |
| 3 | [exercise-03-ground-ai-in-your-data.md](./exercise-03-ground-ai-in-your-data.md) | A grounded Q&A function that retrieves real `crunchcycles` rows before calling Claude | Exercise 2, Lecture 2 |

## How to work these

- Use the `crunchcycles` database and `.venv` you set up in this week's [README](../README.md#setup). Every exercise assumes both exist.
- Write real, runnable code — a Python file or a Jupyter cell you actually execute, not pseudocode. Print your outputs. If a query returns zero rows, that's information — say why, don't skip it.
- Each exercise has an **Expected outcome** section. Match it before moving on; these build on each other and a broken Exercise 2 will produce a broken Exercise 3.
- **Do not commit your `.env` file or your API key anywhere**, including in the Python file you turn in for a review. If a snippet needs to show the key exists, reference `os.getenv("ANTHROPIC_API_KEY")`, never the literal value.
- Keep every SQL query parameterized (`:name` placeholders via SQLAlchemy, never an f-string building SQL text) — this is not optional practice, it's the same injection discipline from Week 7's API and Week 9's security work, and it matters even more here because some of this week's inputs are ultimately shaped by an LLM's tool call.
