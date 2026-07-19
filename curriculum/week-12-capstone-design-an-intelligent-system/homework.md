# Week 12 — Homework

Five problems, ~5 hours total, that reinforce this week's lectures with a mix of analysis, writing, and one hands-on Python task. Unlike most weeks, these don't add new build work on top of the mini-project — they make you look sideways and backward at systems you didn't design, which is a different and necessary skill from designing your own.

---

## Problem 1 — Audit a system you use (60 min)

Pick any real information system you interact with regularly (your bank's app, a university's course portal, a food delivery app, your workplace's internal tool). Without any inside access — just as a user — map it against Lecture 1's six-layer stack as best you can infer:

1. What can you infer about its data foundation from what it lets you enter and see?
2. What automated processes are visible to you (confirmation emails, status changes, reminders)?
3. Any evidence of its cloud/scale behavior (does it slow down at peak times, show maintenance windows)?
4. What security/access behavior have you observed (login flows, permission errors, session timeouts)?
5. What analytics does it clearly show you (your own dashboard, spending summary, order history)?
6. Any AI/decision-support feature (recommendations, fraud flags, predicted delivery time)?

**Deliver** `homework-01-audit.md`, one paragraph per layer, plus a closing paragraph: which layer seems weakest from the outside, and what evidence makes you think so?

---

## Problem 2 — TCO sensitivity analysis (60 min)

Take your mini-project's `tco_model.py` (or the Lecture 2 version if you haven't finalized your own) and run it across a **range** of one input, in Python, to see how sensitive your 3-year TCO is to that one assumption.

```python
# homework-02-sensitivity.py
# Vary hourly_rate from 40 to 120 in steps of 10, holding everything else fixed.
# Print the resulting tco_3yr for each, and identify roughly how much the
# total swings (in dollars and in percent) across that range.
```

**Deliver** the script and its output, plus 3–4 sentences: which single input in your TCO model has the biggest effect on the 3-year total, and does that match your intuition about where the real cost risk is (build cost vs. recurring cloud cost vs. support hours)?

---

## Problem 3 — Rewrite a bad architecture diagram (45 min)

Find (or recall) an architecture diagram you've seen that violates Lecture 3's rules — too much implementation detail, unclear flow direction, no security/cost boundary shown, or jargon a stakeholder wouldn't understand. (If you can't find a real one, deliberately draw a bad one yourself first.) Then redraw it correctly, applying Lecture 3 section 3's rules.

**Deliver** `homework-03-diagrams.md` with both versions (bad and fixed) and 3 sentences on what specifically you changed and why each change helps a specific audience from Lecture 3's table.

---

## Problem 4 — Build-vs-buy decision memo (60 min)

Pick one layer from your mini-project system where you have NOT yet firmly decided build vs. buy (or reconsider one you did decide, playing devil's advocate against your own choice). Write a full build-vs-buy memo using Lecture 3's four-part trade-off structure, but go one step further: research one real, specific product or service you'd be buying (name it, note its actual pricing tier if published) rather than a generic category.

**Deliver** `homework-04-build-vs-buy.md`, roughly 300–400 words, citing the real product/service by name.

---

## Problem 5 — Practice defense, out loud (75 min)

Recruit one person — a classmate, a friend, a family member, anyone willing to spend 20 minutes — and actually present your mini-project's `PROPOSAL.md` to them out loud, in under 5 minutes, then let them ask you 3 questions freely (don't feed them Challenge 1's script). If nobody is available, record yourself presenting it and play it back as your own critical audience.

**Deliver** `homework-05-defense-notes.md`: the 3 questions you were actually asked, how you answered each, and — most importantly — one thing that was harder to explain out loud than it looked on paper. That gap between written and spoken clarity is exactly what Lecture 3 is training, and it's the single most reliable predictor of how your real capstone defense will go.

---

## Time budget

| Problem | Time |
|--------:|----:|
| 1 | 60 min |
| 2 | 60 min |
| 3 | 45 min |
| 4 | 60 min |
| 5 | 75 min |
| **Total** | **~5h** |

After homework, take the [quiz](./quiz.md) and finish the [mini-project](./mini-project/README.md) if you haven't already — this is the last week.
