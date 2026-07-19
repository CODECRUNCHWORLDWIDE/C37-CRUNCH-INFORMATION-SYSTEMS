# Week 2 — Quiz

Fourteen questions. Lectures closed. Aim for 12/14 before starting Week 3. A mix of multiple-choice and short "what would you do" questions — the answer key explains the *why*, not just the letter.

---

**Q1.** On the power/interest grid, Jordan Blake (Store Associate: low power, high interest) belongs in which quadrant, and what does that mean for how you treat her input?

- A) Monitor — light touch, her input matters least
- B) Keep informed — she uses the system daily and her detail is often the richest in the analysis
- C) Manage closely — she's the executive sponsor
- D) Keep satisfied — inform her but don't ask for input

---

**Q2.** Which pair of elicitation techniques best triangulates a finding — confirming what a stakeholder *says* against what actually happens?

- A) Two separate interviews with the same person
- B) A survey sent to the same person twice
- C) An interview and an observation/job-shadowing session
- D) Reading the requirements document twice

---

**Q3.** A stakeholder says "make it faster." What is the correct next move?

- A) Write the requirement "the system shall be fast."
- B) Ask them to approve a mockup so you can move forward
- C) Ladder it with follow-up questions ("walk me through the last time it felt slow") until you reach something specific and measurable
- D) Assume they mean page-load time and proceed

---

**Q4.** Why is a workaround (a personal spreadsheet, a sticky note, a side text-message thread) valuable evidence in elicitation?

- A) It isn't — workarounds are a distraction from the real requirements.
- B) It proves the stakeholder is not following instructions.
- C) It's proof a real need exists that the official process doesn't meet.
- D) It shows the stakeholder needs more training.

---

**Q5.** Which statement is a **non-functional** requirement?

- A) "The system shall let a customer look up their order status by order number."
- B) "The system shall send a confirmation email when an order is placed."
- C) "Order-status lookups shall return within 2 seconds for 95% of requests during store hours."
- D) "The system shall let a Store Manager approve an order."

---

**Q6.** "The system shall let associates place orders and notify customers and log every approval" fails which quality of a good requirement?

- A) Testable
- B) Atomic — it bundles three separately testable ideas into one statement
- C) Feasible
- D) Traceable

---

**Q7.** In MoSCoW, what is the purpose of an explicit **"Won't"** entry, as opposed to just leaving an idea off the list?

- A) It has no real purpose — silence is equivalent.
- B) It tells stakeholders the idea was heard and deliberately deferred, not forgotten, which prevents repeated re-litigation.
- C) It means the idea is banned from ever being built.
- D) It's only used when a stakeholder is difficult.

---

**Q8.** If a requirements list comes back roughly 80% "Must," what's the most likely problem?

- A) The team simply has an unusually critical project.
- B) Prioritization hasn't actually happened — everything has been relabeled "Must" without a real tradeoff conversation.
- C) MoSCoW doesn't apply to this kind of project.
- D) The stakeholders are all high-power.

---

**Q9.** In a use case, what is the difference between an **alternate flow** and an **exception flow**?

- A) They're the same thing with different names.
- B) An alternate flow is a legitimate, expected variation (e.g., insufficient stock); an exception flow is a failure (e.g., a system is down).
- C) Alternate flows only apply to secondary actors.
- D) Exception flows are optional; alternate flows are mandatory.

---

**Q10.** A use case that documents only the main success scenario, with no alternate or exception flows, is:

- A) Complete and ready to build from
- B) Unfinished — the alternate and exception flows are often where the real requirements (like an approval threshold) hide
- C) Better than one with too many flows
- D) Only acceptable for simple systems

---

**Q11.** Which user story best passes the **Valuable** letter of INVEST?

- A) "As a developer, I want a database table for orders."
- B) "As a Store Associate, I want to see live warehouse stock, so that I can tell the customer immediately whether an item is available."
- C) "As the system, I want to log data."
- D) "As a QA tester, I want a test environment."

---

**Q12.** Given/When/Then acceptance criteria exist to:

- A) Replace the user story entirely
- B) Make "done" an observable, testable fact instead of a matter of opinion
- C) Document only the happy path
- D) Give the story a title

---

**Q13.** Why does this course store the stakeholder map, requirements, and user stories in SQL tables instead of a shared spreadsheet?

- A) SQL is required by law for business documents.
- B) Spreadsheets can't store text.
- C) A shared spreadsheet is exactly the single-copy, no-audit-trail, silently-diverging artifact this course's data rule is designed to avoid — a queryable, constrained store scales and traces correctly instead.
- D) SQL is faster to type than a spreadsheet.

---

**Q14.** In a current-state vs. future-state diagram, why does it matter to show every handoff (a phone call, a sticky note, a spreadsheet edit) as its own explicit step rather than collapsing them into one box?

- A) It doesn't matter — fewer boxes is always clearer.
- B) Handoffs are exactly where time, accuracy, and trust bleed out of a process — collapsing them hides the waste the redesign is supposed to remove.
- C) Diagrams require a minimum number of boxes to be valid.
- D) Only the future-state diagram needs detail; the current-state one can be a summary.

---

## Answer key

<details>
<summary>Reveal after attempting</summary>

1. **B** — low power / high interest is "keep informed." Jordan uses the system daily; her ground-level detail is usually the richest data in the whole analysis, even though she has no organizational power.
2. **C** — an interview tells you what a stakeholder *says* happens; an observation shows you what *actually* happens. Two interviews or two surveys don't cross-check each other the same way.
3. **C** — laddering. Writing "the system shall be fast" (A) just relabels the vague word instead of resolving it; assuming a meaning (D) risks building the wrong thing confidently.
4. **C** — a workaround is proof a real, unmet need exists. Dismissing it (A) or treating it as a training gap (D) throws away exactly the evidence you need.
5. **C** — a quantified performance constraint that applies across a feature is the textbook non-functional requirement. A, B, and D each describe a specific system behavior — functional.
6. **B** — "place orders and notify customers and log approvals" is three testable ideas in one requirement ID; if only one part fails, you can't tell which. Split it.
7. **B** — writing "Won't" explicitly closes the loop for the stakeholder who suggested the idea, which is what actually stops it from resurfacing in every later meeting.
8. **B** — if nearly everything is "Must," the team hasn't made the hard calls MoSCoW exists to force; go back and re-test each item against "the system fails its purpose without this."
9. **B** — alternate flows are expected branches within the design (like a stock shortfall); exception flows are failures the system has to handle gracefully (like an unreachable warehouse system).
10. **B** — a use case with only the happy path is unfinished. Real requirements (like Priya's $500 approval rule) frequently live in the alternate/exception flows, not the main scenario.
11. **B** — it names a real actor, a concrete goal, and a genuine benefit. A, C, and D describe technical tasks or system-internal actions with no real external actor who benefits — a classic "fake user story."
12. **B** — Given/When/Then turns "done" into an observable pass/fail check instead of a subjective judgment call.
13. **C** — the exact failure mode this week is teaching you to avoid: a shared spreadsheet with no constraints, no audit trail, and multiple silently-diverging copies. SQL enforces structure and lets you query traceability directly.
14. **B** — the phone calls, sticky notes, and spreadsheet edits *are* the waste. Collapsing them into a single box in the diagram hides exactly what the redesign needs to remove.

</details>

**Scoring:** 12+ → start Week 3. 9–11 → re-read the lecture sections behind your misses. <9 → re-read all three lectures from the top; requirements discipline compounds every week from here on.
