# Challenge 1 — Build vs. Buy for a Mid-Size Firm

**Time:** ~2 hours. **Difficulty:** Medium-hard. **No single right answer.**

## The scenario

Crunch Cycles has grown to 80 employees and just hit the wall described in Lecture 1: sales, warehouse, and purchasing each run on a different tool (a shared spreadsheet, a standalone POS, and email, respectively), and nothing agrees with anything else. The CEO wants a real ERP. Two options have landed on the table:

**Option A — Buy (a mid-market ERP, e.g. NetSuite/Dynamics-class product):**
- License: **$125/user/month**, for 35 named users who'd actually touch the system (not all 80 staff need seats).
- Implementation (a consulting partner configures it to Crunch Cycles' processes): **$180,000 one-time**, paid in Year 1.
- Customization for Crunch Cycles' specific bike-industry quirks (variant/size matrices, warranty tracking): **$60,000 one-time** in Year 1, plus **$15,000/year** in ongoing customization maintenance (every vendor upgrade risks breaking a customization, and someone has to fix it).
- Annual support/maintenance fee: **18% of the license cost**, billed yearly on top of the license.
- Internal staff time to administer it: **0.5 FTE** (a systems administrator), starting Year 1. Assume a fully-loaded cost of **$95,000/year** for that role.

**Option B — Build (in-house, using exactly the tooling from this course — PostgreSQL + Python):**
- Two engineers for 9 months to build a first working version covering sales, inventory, and procurement (no HR/payroll — Crunch Cycles keeps using its existing payroll vendor either way). Fully-loaded cost per engineer: **$160,000/year**.
- Ongoing: **1.5 FTE** engineers to maintain and extend it indefinitely, same fully-loaded rate.
- No license fee, no implementation fee, no vendor customization fee — but budget **$25,000/year** for cloud infrastructure (database hosting, backups, monitoring) that Option A's vendor would otherwise include.
- **Risk line, don't skip it:** budget a one-time **$40,000** "unknown unknowns" contingency in Year 1 — building in-house almost always uncovers requirements nobody wrote down (this is a real, well-documented pattern in build-vs-buy literature, not a made-up number for this exercise).

## Your task

**Do not use a spreadsheet.** Model both options' cost over a **5-year horizon** in Python (pandas), producing a real, run-a-script-and-get-numbers comparison — this is precisely the kind of "looks like a finance workflow" task the course's data rule exists for. A spreadsheet hides its own formulas from anyone else; a script is auditable, re-runnable, and diffable.

1. Write a Python script `tco_model.py` that builds a small DataFrame (or two) with one row per year (Year 1–5) and columns for each cost component, for each option. Use `pandas`.
2. Compute a `total_cost` column per year per option, and a `cumulative_cost` running total.
3. Print (or write to a CSV via `to_csv`, which is fine as an *output artifact* — the modeling itself must not live in a spreadsheet) a clean year-by-year comparison table.
4. Identify the **breakeven point**, if any — the year in which one option's cumulative cost overtakes the other's. State it as a sentence, not just a number in a table.
5. Write `recommendation.md` (400–600 words): your recommendation, the assumptions you'd want to challenge if this were real (staffing estimates, especially), and one scenario under which your recommendation would flip (e.g., "if Crunch Cycles planned to grow to 300 employees in 3 years, my answer would change because...").

## Starter shape

```python
import pandas as pd

years = list(range(1, 6))

buy = pd.DataFrame({
    "year": years,
    "license":        [125 * 12 * 35] * 5,
    "implementation":  [180_000, 0, 0, 0, 0],
    "customization":   [60_000, 15_000, 15_000, 15_000, 15_000],
    "support_pct":     [0.18] * 5,
    "admin_staff":     [95_000 * 0.5] * 5,
})
# Next: compute buy["support_fee"] = buy["license"] * buy["support_pct"]
# Next: compute buy["total_cost"] = sum of the relevant columns
# Next: compute buy["cumulative_cost"] = buy["total_cost"].cumsum()

build = pd.DataFrame({
    "year": years,
    # Next: model 2 engineers x 9 months in Year 1, then 1.5 FTE ongoing
    # Next: infrastructure, contingency (Year 1 only)
})
# Next: same total_cost / cumulative_cost treatment

# Next: merge buy and build on "year", print or save a comparison table
# Next: find the year (if any) where build's cumulative_cost first drops below buy's
```

## Constraints

- All modeling in Python/pandas. No Excel, no Google Sheets, not even as scratch work you "translate" into the script afterward.
- Show your work: every number in `recommendation.md` must trace back to a line in `tco_model.py`'s output, not a number you eyeballed.
- Take a side. "It depends" is not a recommendation — state what you'd tell the CEO **today**, given the numbers, while still naming the conditions that would change your mind.

## Hints

<details>
<summary>On the Year 1 build cost</summary>

Two engineers for 9 months isn't a full year of cost — it's 9/12 of their annual rate, for two people, in Year 1 only. Don't forget Year 1 build also needs *some* ongoing-maintenance overlap if the build finishes partway through the year and needs to start being maintained — state your assumption about exactly when in Year 1 the system goes live and maintenance costs start.

</details>

<details>
<summary>On why 18% support and $15k customization maintenance compound</summary>

Both of these are recurring, not one-time — model them for **all 5 years**, not just Year 1. This is exactly the kind of cost that makes "buy" look cheap in a one-year sales pitch and expensive over a 5-year horizon; a big part of this challenge is noticing that.

</details>

## How success is judged

| Signal | Weak answer | Strong answer |
|---|---|---|
| Tooling discipline | Any spreadsheet in the workflow | 100% Python/pandas, reproducible from the script alone |
| Completeness | Misses a recurring cost line (support %, customization maintenance) | Every cost component from the scenario appears, correctly recurring or one-time |
| Breakeven | No breakeven analysis, or a vague "buy is more expensive" | States the specific year (or "no breakeven within 5 years") with the number to back it |
| Recommendation | Restates the numbers with no opinion | Takes a clear position **and** names the condition that would flip it |
| Assumption honesty | Treats every input as certain | Flags which inputs (staffing estimates especially) are the shakiest and why |

## Submission

Commit `challenge-01/tco_model.py`, its CSV output, and `recommendation.md` to your portfolio under `c37-week-06/challenge-01/`.
