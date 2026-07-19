# KPIs & Dashboards — Metrics People Actually Trust

You now have a warehouse (Lecture 2) that can answer almost any question about Crunch Cycles' sales fast. That capability creates a new risk: it is now *easy* to compute a hundred different numbers, and easy is not the same as useful. This lecture is about choosing the handful of numbers that actually matter, computing them correctly, and presenting them in a way that survives someone asking "wait, is that actually true?" — because someone always eventually does, and the answer needs to be yes.

## 1. What makes something a KPI, and not just a number

A **metric** is anything measurable: total rows in `orders`, average characters in a `company_name`, page-load time. A **KPI (Key Performance Indicator)** is a small, deliberately chosen subset of metrics that a business has decided to track *because it's tied to a goal* — and, critically, because someone would **change what they do** based on the number moving.

The test that separates a real KPI from a metric masquerading as one:

> **If this number went up 20% next month, would anyone at Crunch Cycles do anything differently? If it went down 20%, would anyone act?**

If the honest answer is "no, we'd just note it," it's a metric, not a KPI — interesting, maybe, but not key. "Number of rows in the `customers` table" fails this test (nobody changes strategy because row count went up). "Repeat customer rate" passes it (a sales director who sees this falling starts asking why retention is broken, and does something about it).

## 2. Tie every KPI to a stated business goal, first

The single biggest mistake in dashboard design is picking metrics because they're *available* rather than because they answer a *question someone actually asked*. Work backward, always: state the business goal in one sentence, then derive the KPI from it — never the reverse.

| Business goal (stated first) | KPI that measures progress toward it |
|---|---|
| "Grow revenue in underperforming regions" | Revenue by region, quarter over quarter |
| "Improve profitability, not just top-line sales" | Gross margin % |
| "Reduce dependence on one-time buyers" | Repeat customer rate |
| "Make sure no single rep is a single point of failure" | Revenue per sales rep |
| "Understand if bigger deals or more deals are driving growth" | Average order value (AOV) alongside order count |

Exercise 3 has you do this derivation yourself, from a stated goal, before writing a single query — the same discipline as the table above.

## 3. Six KPIs for Crunch Cycles, defined precisely

A KPI definition that isn't precise enough to write unambiguous SQL from isn't actually defined yet. Each of these states: the exact formula, which rows count, and the business question it answers — against `crunchcycles_dw.warehouse` from Lecture 2.

**1. Total Revenue** — *"How much did we sell?"*

```sql
SELECT SUM(extended_price) AS total_revenue
FROM warehouse.fact_order_items
WHERE order_status = 'Completed';
```

Only `Completed` orders count — a `Pending` order hasn't happened yet, and a `Cancelled` order explicitly didn't. Every KPI in this lecture that touches revenue filters on `order_status = 'Completed'` for the same reason; skipping this filter is the single most common way a warehouse KPI silently overstates the business.

**2. Gross Margin %** — *"How profitable were those sales, not just how big?"*

```sql
SELECT
    SUM(extended_price)                                   AS revenue,
    SUM(extended_cost)                                    AS cost,
    ROUND(100.0 * (SUM(extended_price) - SUM(extended_cost)) / NULLIF(SUM(extended_price), 0), 1) AS gross_margin_pct
FROM warehouse.fact_order_items
WHERE order_status = 'Completed';
```

Revenue alone can grow while margin quietly erodes (deep discounting, a shift toward low-margin accessories) — the reason this KPI exists *separately* from Total Revenue, not as an afterthought. `NULLIF(..., 0)` guards the division against a zero-revenue period producing a divide-by-zero error instead of a clean `NULL`.

**3. Average Order Value (AOV)** — *"Are deals getting bigger or smaller?"*

```sql
SELECT
    ROUND(SUM(extended_price) / COUNT(DISTINCT order_id), 2) AS aov
FROM warehouse.fact_order_items
WHERE order_status = 'Completed';
```

`COUNT(DISTINCT order_id)`, not `COUNT(*)` — the fact table's grain is *line items*, and a single order can have several. Dividing revenue by the row count instead of the distinct order count silently deflates AOV for every multi-item order — a grain mistake exactly like the one Lecture 2 warned about, made concrete.

**4. Revenue per Sales Rep** — *"Is growth broad-based, or is one person carrying the whole team?"*

```sql
SELECT
    e.full_name,
    e.title,
    SUM(f.extended_price) AS revenue,
    COUNT(DISTINCT f.order_id) AS orders
FROM warehouse.fact_order_items f
JOIN warehouse.dim_employee e ON f.employee_key = e.employee_key
WHERE f.order_status = 'Completed'
GROUP BY e.full_name, e.title
ORDER BY revenue DESC;
```

A single join, straight from the star schema — this is the query Lecture 2's design exists to make effortless.

**5. Repeat Customer Rate** — *"Are we building a business, or constantly replacing one-time buyers?"*

```sql
WITH customer_order_counts AS (
    SELECT customer_key, COUNT(DISTINCT order_id) AS completed_orders
    FROM warehouse.fact_order_items
    WHERE order_status = 'Completed'
    GROUP BY customer_key
)
SELECT
    COUNT(*) FILTER (WHERE completed_orders >= 2)                          AS repeat_customers,
    COUNT(*)                                                               AS customers_with_any_order,
    ROUND(100.0 * COUNT(*) FILTER (WHERE completed_orders >= 2) / COUNT(*), 1) AS repeat_rate_pct
FROM customer_order_counts;
```

The denominator is customers **with at least one completed order** — not every row in `dim_customer` (which includes customers who never bought anything, and would understate the rate against the wrong base) and not "all-time signups" (a vanity-metric trap covered next).

**6. Month-over-Month Revenue Growth** — *"Is the trend actually improving, or did one big quarter make everything look fine?"*

```sql
WITH monthly_revenue AS (
    SELECT d.year, d.month, SUM(f.extended_price) AS revenue
    FROM warehouse.fact_order_items f
    JOIN warehouse.dim_date d ON f.date_key = d.date_key
    WHERE f.order_status = 'Completed'
    GROUP BY d.year, d.month
)
SELECT
    year, month, revenue,
    LAG(revenue) OVER (ORDER BY year, month) AS prior_month_revenue,
    ROUND(100.0 * (revenue - LAG(revenue) OVER (ORDER BY year, month))
          / NULLIF(LAG(revenue) OVER (ORDER BY year, month), 0), 1) AS mom_growth_pct
FROM monthly_revenue
ORDER BY year, month;
```

This KPI is the direct antidote to §4 below — it forces the *rate of change*, month by month, into view instead of letting a single cumulative total hide a slowdown.

## 4. Vanity metrics — the trap that looks like progress

A **vanity metric** is a number that goes up almost no matter what, tells a flattering story, and drives zero decisions — the opposite of §1's test, dressed up to look like a real KPI.

**Classic Crunch Cycles example: "Total Customers Signed Up, All-Time."** This number can *only* go up — signups accumulate, they never subtract when a customer stops buying. A chart of it is, by mathematical necessity, always trending up and to the right, which makes it feel like proof of a healthy business. It answers no operational question: it doesn't say how many of those customers bought *anything last quarter*, doesn't say how many are actively repeat buyers (KPI #5 above), and doesn't say whether the *rate* of new signups is accelerating or has quietly stalled while the cumulative total keeps climbing on the back of customers signed up years ago.

```sql
-- Vanity: always goes up, answers nothing about current health
SELECT COUNT(*) AS total_customers_ever FROM warehouse.dim_customer;

-- Actionable: the same underlying data, asked the right question
SELECT COUNT(DISTINCT customer_key) AS active_customers_this_quarter
FROM warehouse.fact_order_items f
JOIN warehouse.dim_date d ON f.date_key = d.date_key
WHERE f.order_status = 'Completed'
  AND d.year = 2024 AND d.quarter = 2;
```

The tell, every time: **a cumulative total, presented without a time-boxed rate alongside it.** "Total signups" (cumulative) tells you nothing "signups this month, compared to last month" (a rate) doesn't tell you better. Challenge 2 hands you a dashboard built entirely around this exact trap and asks you to find and fix it with a real example.

## 5. Building a dashboard people actually trust

A dashboard is not "put some charts on a page" — it's a small set of promises: *this number is computed the same way every time, from the same definition, and you can see the query behind it.* Three principles, each answering a specific way dashboards go wrong in practice:

**Show the rate, not just the total.** Every cumulative total in this week's dashboard is paired with a time-boxed rate (KPI #6's pattern) — never a cumulative line chart standing alone, for exactly the reason §4 demonstrated.

**Show enough history for context, never a cherry-picked window.** A single number ("Revenue: $45,231 this month") means nothing without a baseline. Was last month $40,000 or $60,000? A dashboard that shows one month at a time, chosen by whoever's presenting, is trivially easy to make look good by picking the flattering month. This week's dashboard always plots at least six months of trend, not a single snapshot.

**Never truncate the y-axis without saying so.** Starting a bar chart's y-axis at 90 instead of 0 makes a 2% change look like a 40% change. If you must zoom in on a range to make a real (small) change visible, label the axis explicitly and say so in the caption — the honest version of "make small changes visible" is annotation, not a hidden axis trick.

**Build the dashboard in code, so it's provably the same query every time.** This week's dashboard is a Python script, `dashboard.py`, that queries `crunchcycles_dw` directly and renders both numbers and charts — not numbers manually copied out of a query result into a slide. Re-running the script re-derives every number from the same SQL, which means the dashboard *cannot* drift from the warehouse it's reporting on.

```python
# dashboard.py — structure (Exercise 3 / mini-project build this out fully)
import matplotlib
matplotlib.use("Agg")  # render to file, no display needed
import matplotlib.pyplot as plt
import pandas as pd
from sqlalchemy import create_engine

engine = create_engine("postgresql://localhost/crunchcycles_dw")

def kpi_total_revenue() -> float:
    return pd.read_sql(
        "SELECT SUM(extended_price) AS revenue FROM warehouse.fact_order_items "
        "WHERE order_status = 'Completed'", engine
    )["revenue"].iloc[0]

def kpi_monthly_revenue_trend() -> pd.DataFrame:
    return pd.read_sql("""
        SELECT d.year, d.month, SUM(f.extended_price) AS revenue
        FROM warehouse.fact_order_items f
        JOIN warehouse.dim_date d ON f.date_key = d.date_key
        WHERE f.order_status = 'Completed'
        GROUP BY d.year, d.month
        ORDER BY d.year, d.month
    """, engine)

def render_trend_chart(df: pd.DataFrame, path: str):
    fig, ax = plt.subplots(figsize=(8, 4))
    labels = [f"{int(y)}-{int(m):02d}" for y, m in zip(df["year"], df["month"])]
    ax.plot(labels, df["revenue"], marker="o")
    ax.set_ylim(bottom=0)                 # never truncate — start at zero, always
    ax.set_title("Monthly Revenue (Completed Orders)")
    ax.set_ylabel("Revenue (USD)")
    plt.xticks(rotation=45, ha="right")
    fig.tight_layout()
    fig.savefig(path, dpi=120)
    plt.close(fig)

if __name__ == "__main__":
    print(f"Total revenue: ${kpi_total_revenue():,.2f}")
    render_trend_chart(kpi_monthly_revenue_trend(), "monthly_revenue.png")
    print("Saved monthly_revenue.png")
```

`ax.set_ylim(bottom=0)` is not decoration — it's the code-level enforcement of "never truncate the y-axis" from above. Bake the honesty rule into the chart function itself, so nobody building on top of `dashboard.py` later has to remember to apply it by hand.

## 6. Reading a KPI dashboard critically — the questions to ask before trusting one

Whether you built the dashboard or you're the one reading someone else's, run every number through:

1. **What's the exact filter?** ("Revenue" — including cancelled orders, or not? This week: always `Completed` only, stated explicitly.)
2. **What's the denominator?** (Repeat rate — of *all* customers ever, or of customers who bought at all? Getting this wrong silently changes the answer, as §3's KPI #5 showed.)
3. **Is this a rate or a cumulative total?** (§4's whole point.)
4. **What time window, and was it chosen before or after seeing the result?** A window picked *after* seeing which range looks best is not analysis — it's storytelling with a SQL `WHERE` clause.
5. **Could I re-derive this number myself from the query, right now?** If the answer is "I'd have to ask whoever built the dashboard," the dashboard has a trust problem, no matter how good it looks.

## What's next

Exercise 3 has you derive KPIs from a stated business goal, the way §2 modeled. Challenge 2 hands you a real misleading dashboard built on §4's exact trap, for you to diagnose and fix. The mini-project combines the full week: the warehouse (Lecture 2), an idempotent ELT job (Exercise 2), and a dashboard of 4–6 KPIs built the way §5 describes.
