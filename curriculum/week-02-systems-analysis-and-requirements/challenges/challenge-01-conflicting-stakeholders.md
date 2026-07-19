# Challenge 1 — Conflicting Stakeholders

**Time:** ~60 minutes. **Difficulty:** Medium-hard. **No single right answer.**

## The scenario

Three of Meridian's stakeholders have genuinely incompatible goals for the same feature — the approval step for high-value special orders — and all three are right, from where they're standing. This is not a misunderstanding you can clear up with better communication. It's a real tradeoff, and someone (you) has to design a requirement that a reasonable person on all three sides could live with, then say plainly what each side gives up.

**Dana Reyes, Store Manager:** "My team is already doing three phone calls per order. If you add an approval step on top of that, associates will just avoid offering special orders at all rather than deal with the friction — and we lose the sale to a competitor who says yes on the spot."

**Marcus Webb, Warehouse Fulfillment Lead:** "We've had two incidents this year where an associate promised same-week delivery on an order the DC couldn't actually fulfill that fast, and one where a $1,200 order was placed with zero visibility until it showed up in picking. I need a check before an order over some threshold goes anywhere near my dock."

**Priya Anand, VP Finance & Loss Prevention:** "Any order over $500 needs a named, logged approval — full stop. That's not a preference, it's how our loss-prevention audit works, and I will not sign off on a process that skips it. Ideally anything over $1,000 needs two signatures."

Read carefully: Dana wants *speed and low friction*. Marcus wants *a check before commitment reaches the warehouse*. Priya wants *a mandatory, logged, possibly-dual-signature approval on money*. These are not the same requirement wearing different words — they pull in different directions, and a requirement written to fully satisfy one will visibly violate another.

## Your task

Produce `challenge-01.md` containing:

1. **Name the actual conflict in one paragraph.** Not "they disagree" — specifically, which goal trades off against which other goal, and why can't all three be maximized at once? (Hint: friction and control are usually in direct tension — every additional check reduces speed, by definition.)

2. **Propose a reconciled requirement set** (2–4 requirement statements, in the Lecture 2 template, with priorities) that gives each stakeholder *most* of what they need, even if none gets everything. Your statements must be testable, per the Lecture 2 checklist — "reasonable approval process" is not a requirement, it's a restatement of the problem.

3. **State explicitly what each stakeholder gives up.** For Dana: how much added friction does your design actually add, in concrete terms (an extra screen? a wait for someone else's action? how long, realistically)? For Marcus: does his check happen before or after the customer commitment — and if after, how do you address the two incidents he cited? For Priya: does your design meet her literal ask (a signature over $500, two over $1,000), or a documented, defensible alternative — and if it's an alternative, why would she accept it?

4. **Write the one-paragraph memo you'd actually send the three of them.** This is the real deliverable of a conflict like this — a short, specific message that shows each person their concern was heard and explains the tradeoff in one read, not a 10-page document nobody opens.

## Constraints

- You may not simply pick one stakeholder's position and call the others "wrong." All three have a legitimate, evidence-backed concern (Dana's phone-call friction is documented in Lecture 1's observation session; Marcus cited two specific incidents; Priya's audit requirement is a compliance constraint, not a preference).
- Your reconciled requirements must be things a small engineering team could actually build in a matter of weeks, not a hand-wave ("an AI will figure out the right threshold dynamically").
- At least one of your requirements should use a **threshold** (a specific dollar amount) as the condition — decide where it goes, and justify the number with something more than "it felt right."

## Hints

<details>
<summary>On breaking the false binary (approval vs. no approval)</summary>

The real design space isn't "approve every order" vs. "approve none" — it's *when* the check happens and *how much friction it adds*. A single-tap approval that a Store Manager can do from their own device in under 15 seconds, triggered automatically and only above a threshold, is a very different amount of friction than a phone call to Finance. Consider whether the check can happen in parallel with something else the associate is already doing, instead of blocking the whole flow.

</details>

<details>
<summary>On the dual-threshold structure Priya actually asked for</summary>

Priya's ask ("over $500 = one signature, over $1,000 = two") is already close to a testable requirement — it just needs the template applied: who signs, through what channel, within what time, and what happens to the order while it waits. Don't discard her literal ask just because it's the "strict" one; strict and testable are not opposites.

</details>

## How success is judged

| Signal | Weak answer | Strong answer |
|--------|-------------|----------------|
| Conflict framing | "They just need to communicate better" | Names the specific tradeoff (speed vs. control) precisely |
| Requirement quality | Vague compromise ("reasonable approval") | Testable, atomic, threshold-based statements |
| Honesty about tradeoffs | Implies everyone gets everything | States plainly what each stakeholder gives up |
| Traceability | No connection to this week's evidence | Cites Dana's observed friction, Marcus's incidents, Priya's audit rule specifically |
| Communication | Jargon-heavy memo | One paragraph a busy VP would actually read and feel heard by |

## Submission

Commit `challenge-01.md` to your portfolio under `c37-week-02/challenge-01/`.
