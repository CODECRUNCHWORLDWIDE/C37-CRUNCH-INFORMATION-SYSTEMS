# Exercise 3 — Build a Phased Delivery and Cost Plan

**Estimated time:** 2 hours

**Depends on:** Exercise 2's architecture diagram and layer breakdown.

Lecture 2 argued that a beautiful architecture nobody can afford or operate is a failed project. This exercise makes that concrete: phase your Exercise 2 architecture into shippable increments, model its total cost of ownership in Python (never a spreadsheet — this course's data rule applies to cost models exactly as much as to customer records), and write a risk register you'd actually stand behind in a review.

## Part A — Phase the build (30 min)

Using Lecture 2's five-phase pattern (Discovery → MVP data/workflow → Integration/access → Analytics → AI) as a starting template, map **your** Exercise 2 layers onto phases specific to your organization. Not every system needs exactly five phases — merge or split where it makes sense for your scope — but every phase must pass this test: **if this phase shipped and nothing after it did, would the organization still be better off than before?**

Write this as a table in `08-phases.md`:

| Phase | What ships | Layers touched | Duration (weeks) | Independently valuable? (how) |
|---|---|---|---|---|
| 0 | ... | ... | ... | ... |
| 1 | ... | ... | ... | ... |
| ... | | | | |

## Part B — Model total cost of ownership in Python (45 min)

Adapt Lecture 2's `tco_model.py` to your own system's numbers. Start from the lecture's function and change the inputs to match your Exercise 1 scope and Exercise 2 architecture — don't invent numbers from nothing; ground each input in something (a real cloud pricing page, a real hourly-rate benchmark, or a stated, reasonable assumption you write down).

```python
# your_tco_model.py — adapt the Lecture 2 function to YOUR capstone system
# Ground every number in a source or a stated assumption — don't guess silently.

def tco(build_hours, hourly_rate, monthly_cloud, monthly_support_hours,
        years=3, annual_growth_pct=0.15):
    build_cost = build_hours * hourly_rate
    months = years * 12
    cloud_total = 0.0
    monthly_bill = monthly_cloud
    for month in range(months):
        if month > 0 and month % 12 == 0:
            monthly_bill *= (1 + annual_growth_pct)
        cloud_total += monthly_bill
    support_total = monthly_support_hours * hourly_rate * months
    total = build_cost + cloud_total + support_total
    return {
        "build_cost": round(build_cost, 2),
        "cloud_total_3yr": round(cloud_total, 2),
        "support_total_3yr": round(support_total, 2),
        "tco_3yr": round(total, 2),
        "tco_monthly_avg": round(total / months, 2),
    }

# TODO (you fill these in, with a one-line source/assumption comment each):
result = tco(
    build_hours=___,             # e.g., from your Part A phase durations
    hourly_rate=___,             # e.g., a stated blended-rate assumption
    monthly_cloud=___,           # e.g., from a real PaaS pricing page
    monthly_support_hours=___,   # e.g., an assumption tied to system complexity
)
for k, v in result.items():
    print(f"{k:20s}: {v:,}")
```

Run it, and in the same file or a comment block, add **two scenarios**: your best-guess numbers, and a "budget-constrained" scenario (halve `monthly_cloud` and `hourly_rate` — what has to change about your architecture or phasing to survive that cut?). This second run is direct preparation for Challenge 2.

## Part C — Write the risk register (30 min)

Using Lecture 2's risk register format, list **at least five** risks specific to your organization's system — not generic ones copy-pasted from the lecture. Rank by likelihood × impact and give each a mitigation and an owner (a role, not necessarily you).

| Risk | Likelihood | Impact | Mitigation | Owner |
|---|---|---|---|---|
| ... | ... | ... | ... | ... |

At least one risk must come from each of: **data** (migration, quality, loss), **people** (single point of failure, adoption resistance), and **AI/analytics** (wrong prediction, stale data) — these three categories cover the failure modes real systems hit most often, and a register missing one is a register with a blind spot.

## Part D — The operating plan (15 min)

Lecture 2 said a system nobody can operate after launch is a strategy failure even if the code is perfect. Write 4–6 sentences in `10-operating-plan.md` answering: who runs this system after you're gone (a role, e.g. "the office manager, with a written runbook" — not "nobody, it just works")? What's the plan when something breaks at 6pm on a Friday? What's the minimum technical skill the organization needs on staff or on call?

## Deliverable

A folder `c37-week-12/exercise-03/` containing:

- `08-phases.md` — the phase table
- `your_tco_model.py` — the runnable cost model, with both scenarios and their printed output captured in a comment or a paired `tco-output.txt`
- `09-risk-register.md` — at least five risks, ranked, with mitigations
- `10-operating-plan.md` — the operating plan

## Expected outcome

A phased, costed, risk-aware delivery plan for your specific organization that you could hand to a real stakeholder — every number traceable to a source or a stated assumption, every phase independently valuable, every top risk named with an owner. This, plus Exercises 1 and 2, is the substance of your mini-project; the mini-project mainly adds the schema, the working code, and the presentation.

Next: take what you've built into the [mini-project](../mini-project/README.md), or stress-test it first in [Challenge 1 — Defend under pressure](../challenges/challenge-01-defend-under-pressure.md).
