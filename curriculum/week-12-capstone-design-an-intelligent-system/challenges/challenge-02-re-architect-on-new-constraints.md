# Challenge 2 — Re-architect After a Budget or Scale Change

**Depends on:** Exercises 1–3 (your organization, architecture, and delivery plan).

No real system is designed once and left alone. Budgets get cut. Organizations grow faster than anyone modeled. A system designed for 50 users that suddenly has 5,000 doesn't just need "more server" — it often needs different architectural decisions entirely. This challenge tests whether you can **reshape** an existing design under a new constraint, which is a different and harder skill than designing from a blank page.

## Pick one scenario (not both)

### Scenario A — The 60% budget cut

Your organization tells you: "We love the plan, but the board approved 40% of the budget you asked for. Same problem, same organization, far less money. What do you cut, what do you simplify, and what do you keep?"

Work through, in `challenge-02/budget-cut-memo.md`:

1. **Re-run your Exercise 3 TCO model** with `hourly_rate` and `monthly_cloud` both reduced (or `build_hours` reduced, if you're cutting scope instead of rate) until the 3-year TCO fits 40% of your original number. Show the before/after output.
2. **Which phase(s) from Exercise 3 do you cut entirely**, and which do you simplify? For each cut/simplified phase, state what value the organization loses and whether that's acceptable given the strategic purpose sentence from Exercise 1.
3. **Which layer of Lecture 1's six-layer stack takes the biggest hit?** Almost always this is analytics (5) or AI (6) — the highest layers, built last, are also the first to go under pressure. Confirm or challenge that pattern for your specific system: is there a cheaper layer you'd cut first instead, and why?
4. **Re-draw (or annotate) your Exercise 2 diagram** showing what's removed. A design that survives a budget cut gracefully removes boxes without breaking the boxes that remain — if cutting the AI layer breaks your dashboard, that's a coupling problem in the *original* design, and you should say so.
5. **State the one thing you refuse to cut**, and defend it in Lecture 3's four-part structure — every real budget negotiation has a line the designer should hold, and "we cut everything asked" is not a defensible answer either.

### Scenario B — The 20x scale shock

Your organization tells you: "A local news story went viral and interest in [organization] just exploded. What was 50 active users is now realistically 1,000+, starting next month, and it might not be temporary."

Work through, in `challenge-02/scale-shock-memo.md`:

1. **Identify the layer that breaks first.** Using your Exercise 2 diagram, walk through which component hits a real limit first at 20x load — usually the OLTP database's write capacity, the ETL job's runtime window (Week 5/7 pattern), or a rate-limited third-party API. Name the specific bottleneck, not "the database might be slow."
2. **Propose the fix for that layer**, referencing Week 8's scaling concepts: does it need vertical scaling (bigger instance), horizontal scaling (read replicas, connection pooling), a caching layer, or an architectural change (e.g., moving a synchronous process to an async queue)? Justify the choice against cost — Week 8 (and Lecture 2) taught you that the cheapest fix that solves the actual bottleneck beats the most sophisticated one.
3. **Re-run your Exercise 3 TCO model** with `monthly_cloud` scaled up realistically (not just multiplied by 20 — cloud costs rarely scale linearly; note where economies of scale or step-function pricing changes apply) and `annual_growth_pct` raised. Show the new 3-year TCO.
4. **Does your security model (Week 9 pattern) still hold at this scale?** More users usually means more roles, more edge cases in row-level security, and a higher cost of a mistake. State one thing about your access control design that gets harder at 20x, and how you'd handle it.
5. **What breaks in your delivery plan's timeline?** If Phase 3 (analytics) was scheduled for month 4, does the scale shock force you to move some hardening work earlier? Re-order your Exercise 3 phase table and justify the new order.

## Deliverable

One memo (`budget-cut-memo.md` or `scale-shock-memo.md`, matching your chosen scenario), 1–2 pages, containing all five numbered items above with concrete before/after numbers from your re-run TCO model.

## What "great" looks like

- The memo **reshapes** your existing Exercise 1–3 work — it references specific phases, layers, and numbers from your own prior deliverables, not a generic "here's what you'd do in general" answer.
- The TCO before/after is a real re-run of your Python model with changed inputs, not a hand-waved percentage.
- You correctly identify which layer breaks first (Scenario B) or gets cut first (Scenario A) using Lecture 1's dependency logic — the answer should follow from your architecture, not from a generic rule of thumb.
- You hold at least one line you refuse to compromise, and defend it — showing judgment, not just compliance with the new constraint.
