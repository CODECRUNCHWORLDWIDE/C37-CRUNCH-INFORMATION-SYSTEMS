# Week 12 — Quiz

Fourteen questions. Lectures closed. Aim for 12/14 before considering the capstone complete. A mix of multiple-choice and short "what's the right call here" — the answer key at the bottom explains the *why*, not just the letter.

---

**Q1.** In the six-layer stack from Lecture 1, why does the AI layer sit at the top, built last?

- A) AI is the least important layer
- B) It depends on every layer below being correct — a broken data foundation produces confidently wrong AI output
- C) Regulations require AI to be added after launch
- D) AI features are always the cheapest to build, so they're saved for last as a bonus

---

**Q2.** Which of these is a genuine example of "seam 2: security boundary duplication" from Lecture 1?

- A) The dashboard enforces its own separate `if role == 'admin'` check instead of relying on the database's row-level security
- B) The ETL job runs nightly instead of in real time
- C) The cloud bill grows as usage grows
- D) A new column is added to the `orders` table

---

**Q3.** Per Lecture 1's build-vs-buy guidance, which layer should you almost always **build** rather than buy?

- A) Cloud hosting platform
- B) Payment processing
- C) The data foundation / schema
- D) Authentication primitives

---

**Q4.** A strategic purpose sentence, per Lecture 2, must include:

- A) A list of every technology used
- B) A specific audience, a specific pain, and a measurable improvement
- C) The full cost estimate
- D) The names of every stakeholder

---

**Q5.** Why does Lecture 2 recommend shipping the AI feature in the last phase, not the first?

- A) AI is always the least useful feature
- B) It's the least certain to work and most dependent on lower layers being solid — betting the project on it first is the highest-risk order
- C) Regulations require it
- D) Stakeholders never want AI features

---

**Q6.** In the `tco_model.py` pattern from Lecture 2, what does the function typically reveal that a build-cost-only estimate hides?

- A) The build cost is always the largest number
- B) For a small system, 3-year recurring cloud + support cost often exceeds the one-time build cost
- C) TCO models are inaccurate and should be avoided
- D) Cloud costs never grow over time

---

**Q7.** A risk register entry should be ranked primarily by:

- A) Alphabetical order of the risk name
- B) Likelihood × impact
- C) How recently the risk was discovered
- D) Whether the risk involves AI

---

**Q8.** Per Lecture 2, a system that is technically correct but strategically wrong for the organization most often fails because:

- A) The code has bugs
- B) It doesn't match the organization's actual size, budget, or technical maturity to operate it
- C) It uses PostgreSQL instead of a NoSQL database
- D) The team didn't use enough AI

---

**Q9.** Per Lecture 3, why should you present top-down even though you designed bottom-up?

- A) Stakeholders want to see the schema first
- B) Nobody wants to hear about `CREATE TABLE` statements before they understand why the system exists
- C) Top-down presentations are legally required
- D) There's no actual difference in how to present vs. how to design

---

**Q10.** Which is the strongest fourth part of a defended trade-off, per Lecture 3's four-part structure?

- A) Restating the decision a second time, more firmly
- B) Naming what would actually change your mind — the condition under which you'd choose differently
- C) Pointing out that the reviewer isn't a technical expert
- D) Citing a competitor who made the same choice

---

**Q11.** When you're asked a question in a defense that you genuinely don't know the answer to, Lecture 3 recommends:

- A) Give a confident guess so you don't look unprepared
- B) Say "I don't have that number — here's my rough bound, my method to confirm it, and when I'll follow up"
- C) Change the subject to a question you do know
- D) Say the question is out of scope and decline to answer

---

**Q12.** Why does a good architecture diagram, per Lecture 3, label boxes with nouns like "Customer order history" instead of "orders JOIN order_items"?

- A) SQL syntax isn't allowed in diagrams
- B) A stakeholder-facing diagram should be readable by a non-technical audience; the implementation detail belongs in a technical appendix
- C) JOIN syntax is deprecated
- D) It makes the diagram shorter

---

**Q13.** In Challenge 2's budget-cut scenario, Lecture 1's dependency logic predicts that under a severe cut, the layers most likely to be cut first are:

- A) The data foundation and security layers, because they're the most expensive
- B) The analytics and AI layers, because they're built last and depend on the lower layers rather than being depended upon
- C) All layers equally, since cuts are always proportional
- D) Only the cloud hosting layer

---

**Q14.** The one-page `PROPOSAL.md` structure from Lecture 3 exists primarily because:

- A) It's shorter to write than a full slide deck
- B) Real decisions often get made after the presenter leaves the room, by someone who forwards the document to a person who wasn't there
- C) Stakeholders never read documents longer than one page, as a rule
- D) It replaces the need for an architecture diagram entirely

---

## Answer key

<details>
<summary>Reveal after attempting</summary>

1. **B** — every layer above depends on the ones below being correct; an AI feature built on a bad data foundation produces confident nonsense, so it's built and shipped last, once the foundation is proven.
2. **A** — two independent copies of the same access-control rule (one in the dashboard, one in the database) will eventually drift apart; the fix is enforcing the rule in exactly one place.
3. **C** — the data foundation encodes your organization's *specific* business rules; a generic product can't do that for you, so it's almost always built, not bought.
4. **B** — "This system helps [org] do [specific thing] better by [specific mechanism]" requires a named audience, a concrete pain, and a measurable "better," not a technology list.
5. **B** — AI is the highest-risk, least-proven layer and depends on everything below it; shipping it first bets the whole project on the shakiest piece.
6. **B** — the TCO model usually reveals that for a small system, 3-year cloud + support costs exceed the one-time build cost, which is the number stakeholders actually need for budget approval.
7. **B** — likelihood × impact ranks risks by how much they actually matter, not by recency or alphabetization.
8. **B** — a system the organization can't operate (wrong size/budget/skill match) fails strategically even with flawless code; correctness alone isn't strategic fit.
9. **B** — presenting bottom-up buries the "why does this exist" question under implementation detail nobody asked for yet; top-down leads with purpose, then drops into detail only as questions require.
10. **B** — stating the condition that would change your decision is what separates genuine trade-off understanding from having simply picked a side and rationalized it.
11. **B** — a bounded estimate plus a stated method and follow-up commitment is honest and still useful; a bluff destroys credibility for every other answer you give.
12. **B** — stakeholder-facing diagrams should use nouns the audience recognizes; raw SQL/column-level detail belongs in a technical appendix, not the main diagram.
13. **B** — per the layer dependency logic, analytics and AI are built last and depended *on* rather than depended *upon*, making them the layers a budget cut removes first without breaking what's below them.
14. **B** — the written summary is what survives being forwarded to someone who wasn't in the room; the slide deck (or the presenter's live narration) doesn't travel the same way.

</details>

**Scoring:** 12+ → your capstone reasoning is solid; finish the mini-project with confidence. 9–11 → re-read the lecture sections behind your misses before your defense. <9 → re-read all three lectures from the top; these concepts are the ones a hostile reviewer (Challenge 1) will test hardest.
