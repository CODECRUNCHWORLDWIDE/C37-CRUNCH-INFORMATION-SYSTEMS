# Challenge 2 — Expose a Misleading Metric

**Time:** ~90 minutes. **Difficulty:** Hard. **Code + critique required.**

## The scenario

Below is `growth_dashboard.py`, a real (deliberately misleading) dashboard someone on the Crunch Cycles leadership team's analytics contractor built and presented last quarter. Every number in it is **computed correctly** — there is no SQL bug, no wrong `JOIN`, no off-by-one. And it is still misleading, in the specific way Lecture 3 §4 and §6 warned about. Your job is to find exactly how, prove it with the underlying data, and rebuild the honest version.

```python
# growth_dashboard.py — DO NOT COPY AS-IS. This is the "before" version.
import matplotlib
matplotlib.use("Agg")
import matplotlib.pyplot as plt
import pandas as pd
from sqlalchemy import create_engine

engine = create_engine("postgresql://localhost/crunchcycles_dw")

def total_revenue_to_date():
    return pd.read_sql("""
        SELECT d.year, d.month,
               SUM(SUM(f.extended_price)) OVER (ORDER BY d.year, d.month) AS cumulative_revenue
        FROM warehouse.fact_order_items f
        JOIN warehouse.dim_date d ON f.date_key = d.date_key
        WHERE f.order_status = 'Completed'
        GROUP BY d.year, d.month
        ORDER BY d.year, d.month
    """, engine)

def total_customers_to_date():
    return pd.read_sql("""
        SELECT signup_date, COUNT(*) OVER (ORDER BY signup_date) AS cumulative_customers
        FROM warehouse.dim_customer
        ORDER BY signup_date
    """, engine)

def render(df, ycol, title, path):
    fig, ax = plt.subplots(figsize=(8, 4))
    ax.plot(range(len(df)), df[ycol], marker="o", color="green")
    ax.set_title(title)
    fig.tight_layout()
    fig.savefig(path, dpi=120)
    plt.close(fig)

if __name__ == "__main__":
    rev = total_revenue_to_date()
    render(rev, "cumulative_revenue", "Crunch Cycles Growth: Cumulative Revenue", "revenue_growth.png")
    print(f"Cumulative revenue at end of period: ${rev['cumulative_revenue'].iloc[-1]:,.2f}")

    cust = total_customers_to_date()
    render(cust, "cumulative_customers", "Crunch Cycles Growth: Total Customers", "customer_growth.png")
    print(f"Total customers to date: {cust['cumulative_customers'].iloc[-1]}")
```

Run it. Both charts go up and to the right, smoothly, the whole way — because a cumulative sum of non-negative monthly values is *mathematically incapable* of going down. That's the trap.

## Your task

Write `honest_dashboard.py` — a fixed version — plus `metric-critique.md` documenting the problem and your fix. At minimum:

**1. Reproduce why the charts are misleading, with data, not just assertion.** Query the **non-cumulative** monthly revenue (Lecture 3 §3 KPI #6's pattern) and the **non-cumulative** monthly new-signup count. Show, with actual numbers from your warehouse, at least one month where the real month-over-month trend was flat or declining — a fact the cumulative chart above visually hides. If your specific dataset doesn't have a declining month, show the month-over-month **growth rate** slowing (a smaller increase than the prior month) — the same hiding effect, less dramatic but still real: a cumulative chart makes deceleration look identical to steady acceleration.

**2. Rebuild both charts the honest way**, per Lecture 3 §5: non-cumulative, monthly bars or lines (not a running total), y-axis starting at zero, a clear title stating the actual time unit ("Monthly Revenue," not "Growth"). Both must be in `honest_dashboard.py`, runnable end to end.

**3. Explain, in `metric-critique.md`, exactly which of Lecture 3's dashboard-trust questions (§6) the original `growth_dashboard.py` fails, and why each one specifically applies:**
   - Is this a rate or a cumulative total?
   - What's the exact filter, and is it stated?
   - Could a reader re-derive this number themselves from the chart alone?

**4. Propose the actual KPI leadership should see instead**, tying back to Lecture 3 §2's discipline of deriving a KPI from a stated goal — state what goal ("sustainable growth," "no revenue cliff hiding behind a total," etc.) the honest version actually serves, and why the cumulative chart, despite superficially answering "are we growing," doesn't serve that goal at all.

## A harder twist — find the second problem

There's a second, subtler issue in `total_customers_to_date()` beyond "it's cumulative." Look again at what `dim_customer` actually contains after Exercise 2 and Challenge 1: does *every* row represent a customer who ever placed an order, or does the dimension also include customers who signed up and never bought anything? If "total customers" includes never-purchased signups, what does that do to a leadership team reading this chart as a proxy for "the business is growing"? Name this second problem explicitly in `metric-critique.md` — it's the same "wrong denominator/wrong population" trap Lecture 3 §3's KPI #5 (repeat customer rate) was built specifically to avoid, showing up here in a different disguise.

## How success is judged

| Signal | Weak answer | Strong answer |
|--------|-------------|----------------|
| Diagnosis | "The chart looks too smooth" | Specific month(s) named, with real numbers, where the non-cumulative trend tells a different story than the cumulative one |
| Fix | Just changes the line color or adds gridlines | Rebuilds as a genuinely non-cumulative metric with a zero-based axis, per Lecture 3 §5 |
| Second problem | Missed entirely | Correctly identifies that `dim_customer`'s population includes non-purchasing signups, and states what that does to the metric's meaning |
| Critique depth | Says "it's misleading" | Answers all three of Lecture 3 §6's trust questions specifically against this dashboard, with reasoning |
| Proposed KPI | Restates the same cumulative metric with a new label | A genuinely different, goal-tied metric (e.g., active customers this quarter, or MoM revenue growth %) that would actually change what leadership does |

## Submission

Commit `honest_dashboard.py` and `metric-critique.md` to your portfolio under `c37-week-10/challenge-02/`.
