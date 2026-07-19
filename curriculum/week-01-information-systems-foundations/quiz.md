# Week 1 — Quiz

Fourteen questions. Lectures closed. Aim for 12/14 before starting Week 2. A mix of multiple-choice and short "what would you do" questions — the answer key at the bottom explains the *why*, not just the letter.

---

**Q1.** Which of these is the most accurate definition of an information system?

- A) A piece of software that stores business data
- B) A computer network connecting an organization's departments
- C) An organized combination of people, process, data, and technology that collects, processes, stores, and distributes information to support decisions
- D) Any database used by more than one person

---

**Q2.** A ship captain's paper log book, kept before any computers existed, is:

- A) Not a real information system, because it has no technology component
- B) A real information system — technology doesn't have to mean "computer"
- C) Only a process, not a full system
- D) Only data, not a full system

---

**Q3.** In the input→process→output→feedback loop, what does the **feedback** step do?

- A) Nothing — it's optional and most systems don't have one
- B) Sends the output back out to the customer a second time
- C) Routes some of the output back in to adjust future behavior of the system
- D) Deletes bad output before it reaches anyone

---

**Q4.** Riverbend's system boundary is drawn to include its staff, processes, data, and technology, but **not** its green-coffee suppliers or shipping carriers. Those excluded items are part of the system's:

- A) Feedback loop
- B) Environment
- C) Technology component
- D) Value chain

---

**Q5.** A wholesale order keeps getting double-booked against beans that are already promised elsewhere. Riverbend's first instinct is to buy a more expensive order-management app. Based on Lecture 1 §5, what should you check *before* recommending that?

- A) Nothing — better software always fixes ordering problems
- B) Whether the real gap is a missing process/data check, not a technology limitation
- C) Whether the app has a nicer user interface
- D) Whether competitors use the same app

---

**Q6.** Which of these is correctly classified as **process**, not people or technology?

- A) "Elena Vasquez"
- B) "The Roast Profile Controller"
- C) "The sequence of steps that happens whenever a wholesale order arrives"
- D) "The green coffee currently in the warehouse"

---

**Q7.** Why is an uncontrolled, shared spreadsheet used as a system of record specifically flagged as a problem in this course, rather than just "one option among several" for technology?

- A) Spreadsheets can't store text, only numbers
- B) It's technically not "technology" at all
- C) It typically has no access control, no audit trail, and no enforced structure — inviting the exact single-point-of-failure problems this course exists to avoid
- D) Spreadsheets are always slower than a database, in every case

---

**Q8.** A team collects a huge volume of POS transaction data every day but never summarizes, compares, or acts on it. According to the DIKW pyramid, what do they actually have?

- A) Information
- B) Knowledge
- C) Wisdom
- D) Data — and nothing further up the ladder yet

---

**Q9.** "Sales are up 12% in June compared to May" is an example of which DIKW layer?

- A) Data
- B) Information
- C) Knowledge
- D) Wisdom

---

**Q10.** In Porter's value chain, which of these is a **support** activity rather than a **primary** one?

- A) Roasting the coffee (operations)
- B) Shipping a wholesale order (outbound logistics)
- C) Maintaining the company's information-systems register (infrastructure)
- D) Taking a wholesale order (marketing & sales)

---

**Q11.** A process completes very quickly but frequently produces the *wrong* result (e.g., confirming a delivery date the warehouse can't hit). This process is:

- A) Efficient and effective
- B) Efficient but not effective
- C) Effective but not efficient
- D) Neither efficient nor effective — you can't tell speed and correctness apart from this description

---

**Q12.** In the roast-to-inventory example (Lecture 3 §4), the eventual missed delivery was caused by:

- A) A single broken component that should have been immediately obvious
- B) A missing **flow** between two components, whose absence only became visible days later
- C) Elena Vasquez making a scheduling mistake
- D) A hardware failure in the Roast Profile Controller

---

**Q13.** You're asked to justify fixing a missing flow to a non-technical stakeholder. Which structure does Lecture 3 §5 recommend?

- A) A list of every table and column involved
- B) Gap → consequence → impact → cost → value, in plain language
- C) A full ER diagram
- D) A comparison of three competing software vendors

---

**Q14.** A component in your `is_components` table never appears as a `source_component_id` or `target_component_id` in `is_flows`. What does Exercise 2 call this, and what should you do about it?

- A) A "root node" — leave it, every system needs at least one
- B) A "dead end" — either connect it with a real flow, or explicitly justify why it's genuinely isolated
- C) An "orphan key" — delete it immediately, it's always a mistake
- D) A "primary component" — it's fine, most components have zero flows

---

## Answer key

**Q1 — C.** The definition names all four components and the actual purpose (supporting decisions) — A and D describe only the technology piece, and B is a subset of technology, not the whole system.

**Q2 — B.** Lecture 1 opens with exactly this example. Technology means hardware/software broadly — paper and a ledger count. The point is that information systems predate computers.

**Q3 — C.** Feedback is output routed back in to adjust future behavior — it's what lets a system self-correct instead of repeating the same behavior regardless of results.

**Q4 — B.** Anything outside the system's boundary that still affects it is the environment. Suppliers and carriers are outside Riverbend's boundary but very much shape how it runs.

**Q5 — B.** Lecture 1 §5 is explicit: treating every IS problem as an IT problem is a costly, common mistake. Diagnose which component (process, data, people, or technology) actually broke before reaching for a technology fix.

**Q6 — C.** A repeatable sequence of steps is a process. "Elena Vasquez" is people, "The Roast Profile Controller" is technology, "the green coffee in the warehouse" is data (inventory).

**Q7 — C.** Lecture 2 §4 names the specific risks precisely: no access control, no audit trail, no enforced structure, and no protection against two people overwriting each other's edits.

**Q8 — D.** Data doesn't automatically become information — Lecture 3 §1 is explicit that climbing each layer of the pyramid requires a process (aggregation, comparison, interpretation). Collecting without processing leaves you at the bottom layer.

**Q9 — B.** It's data given context — a comparison over time, not just a raw fact and not yet an explanation of *why* (which would be knowledge) or a decision (wisdom).

**Q10 — C.** Infrastructure — which includes maintaining a system-of-record register — is a support activity: it doesn't directly create the product/service the customer pays for, but it makes the primary activities possible.

**Q11 — B.** Speed (efficiency) and correctness (effectiveness) are different axes. A fast process that produces wrong results is efficient but not effective — and Lecture 3 §3 flags this specific combination as more dangerous than either failure alone, because it looks like success on a speed metric.

**Q12 — B.** The worked example in Lecture 3 §4 traces the failure to a single missing flow (batch consumption not flowing back into inventory), not to any one broken component — and the consequence didn't show up until days or weeks later, which is exactly why it's hard to catch by inspecting components alone.

**Q13 — B.** Gap → consequence → impact → cost → value is the exact pattern Lecture 3 §5 gives you, and it's the pattern used again in Week 2 for requirements justifications and Week 9 for governance cases.

**Q14 — B.** Exercise 2 Part C calls this a "dead end" and asks you to either connect it with a genuine flow or explicitly justify, in writing, why it's correctly isolated — silently leaving one unexamined is the mistake to avoid.
