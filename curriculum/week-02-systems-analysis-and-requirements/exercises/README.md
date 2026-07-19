# Week 2 Exercises

Three guided exercises, done in order. Each builds directly on the `meridian_sos` database you seeded in the [week README](../README.md) — stakeholders and elicitation notes are already there; you'll be adding to `requirements` and `user_stories` yourself.

## How to work these

1. Confirm your database is seeded: `SELECT COUNT(*) FROM stakeholders;` should return `8`.
2. Work each exercise in order — Exercise 2 assumes the classification skill from the lectures, Exercise 3 assumes you can already write a testable requirement.
3. Each exercise tells you exactly what file(s) to produce and what "done" looks like. Keep your SQL in a `.sql` file per exercise so your `requirements`/`user_stories` inserts are reproducible.
4. There's no single correct answer for the wording of a requirement or story — there is a correct *standard* (testable, atomic, clear) that your answer must meet. Check your work against the checklists in Lectures 2 and 3, not against a hidden key.

## What's here

| # | File | Skill drilled | ~Time |
|--:|------|---------------|------:|
| 1 | [exercise-01-write-user-stories.md](./exercise-01-write-user-stories.md) | Converting a feature request into a backlog of user stories with acceptance criteria | 1h |
| 2 | [exercise-02-functional-vs-nonfunctional.md](./exercise-02-functional-vs-nonfunctional.md) | Classifying and rewriting raw meeting notes as FR/NFR, prioritized with MoSCoW | 1h |
| 3 | [exercise-03-current-vs-future-state.md](./exercise-03-current-vs-future-state.md) | Diagramming current-state vs. future-state and naming the gap | 1.5h |

When all three are done, move on to the [challenges](../challenges/).
