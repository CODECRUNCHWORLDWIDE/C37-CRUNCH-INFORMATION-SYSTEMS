# Week 5 — Exercises

Three exercises, meant to be done in order. Each builds a skill the mini-project needs at full scale.

| # | Exercise | Skill drilled | Time |
|---|----------|----------------|-----:|
| 1 | [exercise-01-draw-a-bpmn-diagram.md](./exercise-01-draw-a-bpmn-diagram.md) | Turning a raw, spoken process description into a precise BPMN diagram | 1h |
| 2 | [exercise-02-spot-the-bottleneck.md](./exercise-02-spot-the-bottleneck.md) | Scoring automation candidates and confirming the bottleneck with SQL, not a guess | 1.5h |
| 3 | [exercise-03-script-a-workflow.md](./exercise-03-script-a-workflow.md) | Writing a small, idempotent, logged Python automation against Postgres | 1.5h |

## How to work them

- Use the `crunchride` database you set up in the [week README](../README.md). All three exercises read or write it.
- Exercise 1 is diagram-only — hand-drawn, [bpmn.io](https://demo.bpmn.io/), or a Mermaid flowchart in a `.md` file all count. What matters is correctness (right symbols, right lanes, right gateway types), not the tool.
- Exercises 2 and 3 mix SQL and prose — keep your SQL in a `.sql` file and your written answers in a `.md` file per exercise, same convention as every other week in this course.
- Do them in order — Exercise 3's script reuses the exact eligibility logic Exercise 2 makes you derive by hand first.

## Submission

Each exercise's file says exactly what to commit and where in your portfolio it goes (`c37-week-05/exercise-0N/`). Push after each one — don't wait until Friday to discover Exercise 1's diagram had a gateway-type mistake that would've been a two-minute fix on Monday.
