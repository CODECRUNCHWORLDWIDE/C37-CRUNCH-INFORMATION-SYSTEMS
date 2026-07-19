# Exercise 3 — Estimate a System's Monthly Cost

**Goal:** Build a small, reusable Python cost model for Crunch Cycles' cloud deployment — real enough to answer "what does this cost today, and what would it cost at 10x traffic?" without touching a spreadsheet.

**Estimated time:** 1 hour.

**Data rule reminder:** this is a calculation over a handful of numeric line items, exactly the kind of thing people reach for Excel to do. Don't. Per this course's data rule, model it in Python — the payoff is that "what if we 10x?" becomes changing a few function arguments and re-running the script, not rebuilding a grid of cells.

## Setup

Start from Lecture 3's `cloud_cost_model.py`. Create your own copy, `crunchcycles_cost.py`, and adapt it with **real pricing** pulled from your Exercise 2 provider's published pricing page (or, if you're on a free tier, the provider's *paid* tier pricing — free tiers usually don't have per-unit rates you can extrapolate from). Cite the pricing page URL in a comment at the top of your script.

## Tasks

1. **Look up real unit prices** for: compute (per-instance-hour or per-instance-month), database storage (per GB-month), data egress (per GB), and backup storage (per GB-month) from your provider's pricing page. Write each one into your script as a named constant with a comment citing where it came from.

2. **Model the current Crunch Cycles deployment** — 1–2 small app instances, the database size from Exercise 2, a realistic (even if estimated) monthly egress figure for a small course-project-scale API, and whatever backup retention your provider defaults to. Compute the total.

3. **Model a 10x-growth scenario.** Don't just multiply everything by 10 uniformly — reason about which line items actually scale with traffic and which don't, the way Lecture 3's example does:
   - Compute: does traffic 10x mean you need 10x the instances, or does each instance handle more load first (vertical headroom) before you add more?
   - Database storage: does data volume grow linearly with *traffic*, or with *time* (more orders accumulate regardless of read traffic)? State your assumption.
   - Egress: this one usually *does* scale close to linearly with traffic — say why.
   - Backups: usually track storage, not traffic.

4. **Print both estimates**, broken out by line item (not just a single total), and write 3–4 sentences in `cost-writeup.md`:
   - Which line item dominates the total today?
   - Which line item grows fastest under your 10x scenario, and does that match your intuition from Lecture 3's discussion of egress?
   - If you had to cut this hypothetical bill by 25%, which line item would you attack first, and what specific change would you make?

## Expected result

- `crunchcycles_cost.py` runs with `python3 crunchcycles_cost.py` and prints two breakdowns (current, 10x) each showing compute/storage/egress/backups separately and a total.
- The 10x scenario's total should **not** simply be 10× the current total — if it is, revisit task 3; that's the sign every line item was scaled uniformly instead of reasoned about individually.
- `cost-writeup.md` names a specific dominant line item and a specific proposed cut, not a vague "reduce costs" statement.

## Done when…

- [ ] Real (or realistically sourced) unit prices are used, cited with a source comment.
- [ ] The script prints a line-item breakdown for both scenarios, not just totals.
- [ ] The 10x scenario reflects deliberate per-line-item reasoning, documented in code comments.
- [ ] `cost-writeup.md` answers all three questions in Task 4 with specific numbers, not generalities.

## Stretch

- Add a third scenario: "10x growth, but you also fix a cross-region egress mistake" (Lecture 3, Section 4) — model the egress rate dropping because app and database now share a region, and quantify the savings.
- Parameterize the script to take the growth multiplier as a command-line argument (`sys.argv` or `argparse`) so you can run `python3 crunchcycles_cost.py --growth 5` for any multiplier, not just 10x.

## Submission

Commit `crunchcycles_cost.py` and `cost-writeup.md` to your portfolio under `c37-week-08/exercise-03/`.
