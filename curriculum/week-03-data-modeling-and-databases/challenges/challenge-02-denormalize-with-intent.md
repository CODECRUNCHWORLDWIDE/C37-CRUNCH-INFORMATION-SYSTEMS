# Challenge 2 — Denormalize with Intent

**Time:** ~60 minutes. **Difficulty:** Medium. **You are arguing against the last two lectures — on purpose.**

## The scenario

Lecture 2 spent two hours making the case for normalization, and Section 8 of that lecture made a narrower, harder point: **sometimes staying fully normalized is the wrong engineering call, and denormalizing on purpose — with a name, a reason, and a maintenance plan — is the better trade.** This challenge asks you to find one real place in the CrunchRide schema where that's true, and make the case rigorously enough that a skeptical senior engineer (who just spent all week teaching you to normalize) would approve the pull request.

**The bar is high on purpose.** "It's more convenient" is not a justification — convenience is what normalization *costs* you, and you already knew that going in. A real justification names a specific, measurable pain (a query that's too slow, a report that's too complex, a join depth that's become a maintenance burden) and shows the denormalized alternative solves it without silently reintroducing one of the three anomalies from Lecture 2.

## Your task

Pick **one** of these two CrunchRide denormalization candidates (or propose your own, if you have a better one from the mini-project schema — check with the reasoning below either way):

**Option A — a `rides` summary column.** CrunchRide's ops team wants a live dashboard showing "total rides this month per station." As designed, that's a `JOIN` across `rides` and `stations`, grouped and filtered by date, run every time the dashboard loads. Should `stations` carry a denormalized `rides_started_this_month` counter column, updated incrementally, instead of computing it fresh every time?

**Option B — a `riders.plan_name` copy.** Every query that needs to show a rider's plan name in a report currently joins `riders` to `membership_plans`. A junior developer proposes adding a `plan_name TEXT` column directly on `riders`, copied from `membership_plans.name` at signup time, "to avoid the join." Should you do this?

Write `denormalization-case.md` covering:

1. **The redundancy you're introducing.** State exactly what data now exists in two places, and under what conditions it could become inconsistent.
2. **The specific pain it solves.** Not "joins are annoying in general" — name the actual query, its frequency, and why the join cost is a real problem (or, if you conclude it *isn't* a real problem, say so — see Section 3).
3. **The anomaly risk, and your mitigation.** Which of the three classic anomalies (insertion, update, deletion) does this redundancy risk reintroducing? What specific mechanism — a trigger, an application-layer rule, a scheduled job, an explicit "this field is derived, never edit directly" comment — keeps the redundant copy correct? A denormalization with no stated mitigation is just a bug you haven't found yet.
4. **Your verdict.** Recommend "do it" or "don't" for your chosen option, and defend it. **Note:** for Option B specifically, a strong answer may well conclude the join genuinely isn't expensive enough to justify the risk — plan names rarely change, but the join cost of `riders JOIN membership_plans` on a 3-row lookup table is essentially free, and "avoid the join" is a weaker reason than it sounds. If you conclude "don't denormalize," that is a completely valid, and arguably the *more* senior, answer — as long as you argue it from the same rigor, not from just restating the normalization dogma.

## Hints

<details>
<summary>On telling a real justification from a fake one</summary>

A real justification for Option A sounds like: *"This dashboard query runs on every page load for an ops team of 40 people, joins two tables and aggregates over what will be millions of ride rows within a year, and the aggregation cost grows with total ride history even though the dashboard only cares about *this month*. A running counter, incremented by a trigger on `rides` insert, turns an O(rows) aggregate into an O(1) column read." That's specific, measurable, and names the exact mechanism.*

A fake justification sounds like: *"Joins are slow, so let's just store the count."* No numbers, no query, no comparison — just a vibe. Don't write the second kind.

</details>

<details>
<summary>On the mitigation mechanism</summary>

For Option A, a trigger is the natural fix — every `INSERT` into `rides` fires a trigger that increments the relevant station's counter (and a `ride` that gets deleted or reassigned needs the reverse). Sketch (pseudocode is fine, full trigger syntax isn't required this week) what that trigger would need to do, and be honest about the new failure mode it introduces: **the counter can now drift from reality if the trigger is ever bypassed** (a bulk import that skips triggers, a manual `UPDATE`) — which is exactly the kind of cost a "just add a column" instinct tends to hide.

</details>

## How success is judged

| Signal | Weak answer | Strong answer |
|---|---|---|
| Specificity | "It'll be faster" | Names the actual query, its frequency, and what grows unboundedly without the change |
| Anomaly honesty | Ignores the redundancy risk | Names which anomaly is now possible and exactly how |
| Mitigation | None, or "just be careful" | A concrete mechanism (trigger, job, comment convention) that keeps the copy correct |
| Willingness to say no | Denormalizes because the challenge title suggested you should | Reaches "don't do it" when the evidence doesn't support the change, and says so plainly |

## Submission

Commit `denormalization-case.md` to your portfolio under `c37-week-03/challenge-02/`.
