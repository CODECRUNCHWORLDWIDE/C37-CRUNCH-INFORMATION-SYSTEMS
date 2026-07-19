# Week 5 — Quiz

Fifteen questions. Lectures closed. Aim for 13/15 before starting Week 6. A mix of multiple-choice and short "what would you do here?" questions — the answer key at the bottom explains the *why*, not just the letter.

---

**Q1.** In BPMN, what does a diamond with a `+` inside it represent?

- A) Exclusive gateway — pick exactly one path
- B) Parallel gateway — take all outgoing paths
- C) Inclusive gateway — take one or more paths
- D) An end event

---

**Q2.** A task is drawn with a small gear icon. What does that tell you?

- A) It's a manual task, done by hand with no software
- B) It's a user task — a human does it, software assists
- C) It's a service task — a system does it automatically, no human involved
- D) It's a gateway, not a task at all

---

**Q3.** Why should a parallel gateway that splits the flow into two branches be matched with a parallel gateway (not an exclusive one) where those branches merge back together?

- A) BPMN syntax requires matching shapes for aesthetic reasons only
- B) An exclusive join would proceed as soon as *either* branch arrived, silently changing whether the process actually waits for both — a real behavior difference, not a style choice
- C) It doesn't matter; any gateway type can close any split
- D) Parallel gateways are not allowed to have a matching join at all

---

**Q4.** What is a swimlane for?

- A) Purely visual organization, with no meaning beyond making the diagram look tidy
- B) Marking which tasks are automated vs. manual
- C) Showing who — which role, system, or participant — is responsible for each task
- D) Separating start events from end events

---

**Q5.** A stakeholder says "the process begins." Why is this, by itself, an incomplete start event?

- A) BPMN doesn't have a symbol for start events
- B) A start event needs a named trigger — what actually kicks the process off (a message, a timer, a specific action) — not just a vague description of "beginning"
- C) Every process must have exactly one start event and this sentence implies more than one
- D) It isn't incomplete; this is a perfectly fine start event description

---

**Q6.** Which of these four scoring dimensions from the automation-candidate rubric is the best single predictor of *whether* a task should be automated at all (as opposed to how much value doing so would deliver)?

- A) Frequency / volume
- B) Rule-based-ness — can the decision be written as an unambiguous rule using data you already have
- C) Error cost
- D) How recently the task was last performed

---

**Q7.** CrunchRide's depot repair-or-retire decision scored "never automate" in Lecture 2, despite being high error cost. Why?

- A) Error cost alone should have made it a top automation priority
- B) It requires genuine judgment (a cost/benefit call about a physical asset) that isn't reducible to a rule from available data — automating it risks confidently making a bad, hard-to-reverse call
- C) The depot doesn't have a database connection
- D) Retiring a bike is illegal without a technician present

---

**Q8.** What is "process mining," in the sense Lecture 2 used the term?

- A) Manually interviewing every stakeholder about how they think the process works
- B) Using timestamps and other data already captured by the system (like `opened_ts`/`assigned_ts`) to reconstruct how the process *actually* runs, rather than trusting a description of it
- C) A synonym for BPMN diagramming
- D) A machine learning technique unrelated to SQL

---

**Q9.** In Lecture 3's `assign_ticket` function, the `UPDATE` statement includes `WHERE status = 'open' AND assigned_tech_id IS NULL` — the exact same condition the earlier `SELECT` used to find eligible tickets. What does repeating that condition in the `UPDATE` accomplish?

- A) Nothing; it's redundant and could be safely removed
- B) It makes the operation idempotent — if the ticket was already handled (by this run or a concurrent one) since it was read, the `UPDATE` matches zero rows and the script detects and logs a "skipped," instead of double-assigning
- C) It improves query performance only, with no correctness implications
- D) It prevents SQL injection

---

**Q10.** Why does Lecture 3's `try`/`except` sit inside the loop, around processing one ticket at a time, instead of wrapped once around the whole batch?

- A) Python requires a `try`/`except` inside every `for` loop
- B) So one ticket's failure is logged and isolated, while the rest of the batch still gets processed — instead of one bad row losing you the entire run
- C) It has no effect either way; both are equivalent
- D) To make the code shorter

---

**Q11.** Challenge 2's broken `flag_pickups.py` script used a `SELECT` to check "does a pickup request already exist?" followed by a separate `INSERT`. What class of bug does this pattern create?

- A) A SQL injection vulnerability
- B) A time-of-check to time-of-use (TOCTOU) race condition — two concurrent runs can both see "no existing request" and both insert, producing duplicates
- C) A syntax error that Postgres would reject
- D) An issue only relevant to SQLite, not PostgreSQL

---

**Q12.** What's the most reliable fix for the race condition in Q11?

- A) Add a `time.sleep()` between the `SELECT` and the `INSERT` so they're less likely to overlap
- B) Run the script less often
- C) A database-level `UNIQUE` constraint combined with an atomic `INSERT ... ON CONFLICT DO NOTHING`, so the guarantee lives in the database itself, not in application-level timing
- D) Wrap the whole script in a `try`/`except` so errors are silently ignored

---

**Q13.** Why is a bare `except: pass` (as in the broken Challenge 2 script) considered worse than letting the script crash with a visible error?

- A) It isn't worse — a silent failure is always preferable to a visible one
- B) It hides the failure entirely: no log line, no stack trace, nothing to tell anyone the run didn't do what it appeared to do — the problem surfaces later as an unexplained symptom instead of a diagnosable error
- C) `except: pass` is invalid Python syntax
- D) It only affects performance, not correctness

---

**Q14.** What's the one-sentence tradeoff between scheduling an automation with cron vs. with an in-process scheduler like APScheduler?

- A) Cron is always faster; APScheduler is always slower
- B) Cron needs no long-running process and is simplest for a standalone script; APScheduler keeps a Python process alive, useful when you want richer in-process scheduling logic or you're already running a long-lived service
- C) APScheduler cannot run Python code, only shell commands
- D) There is no meaningful difference; they are interchangeable in every situation

---

**Q15.** A rule-based, high-frequency task scores well on the automation rubric, but the *process itself* is inconsistent — two different dispatchers currently make the same judgment call differently. What should you do?

- A) Automate it immediately; consistency is exactly what automation provides
- B) Automate whichever dispatcher's version is more common, without telling anyone
- C) Fix or explicitly agree on the process first — automating an inconsistent judgment call just locks in and speeds up whichever version you happened to encode, without actually resolving the inconsistency
- D) This situation never actually happens in real organizations

---

## Answer key

<details>
<summary>Reveal after attempting</summary>

1. **B** — a `+` diamond is a parallel gateway: every outgoing path is taken, unconditionally, together.
2. **C** — a gear icon marks a service task: automated, no human involved. (Person icon = user task, hand icon = manual task.)
3. **B** — mismatched split/join types silently change the process's actual behavior (does the merge wait for both branches or fire on the first?). Match your gateway types.
4. **C** — a swimlane shows *who* is responsible for each task — a role, system, or participant. It's organizational information, not decoration.
5. **B** — "the process begins" names no trigger. A real start event says what kicks the process off — a message, a timer, a specific action someone takes.
6. **B** — rule-based-ness is the gate: a task that requires genuine judgment shouldn't be automated regardless of how frequent or costly its errors are. Frequency and error cost tell you how much *value* automating delivers, once rule-based-ness has already said "yes, this is automatable."
7. **B** — the decision requires real judgment about a physical asset's value, not a rule extractable from available data; automating it risks a confident, hard-to-reverse bad call.
8. **B** — process mining (in the lightweight sense used here) means reconstructing what actually happened from the system's own captured timestamps and event data, instead of trusting a stakeholder's description of the process.
9. **B** — repeating the exact eligibility condition in the `UPDATE`'s `WHERE` clause is the idempotency guard: if the row no longer matches (already handled), the update quietly matches zero rows instead of double-processing it.
10. **B** — scoping the `try`/`except` to one ticket at a time gives fault isolation: one bad row is logged and skipped, and the rest of the batch still runs. A single wrapper around the whole function would lose the entire batch to one bad row.
11. **B** — check-then-act with a gap between the check and the act is a textbook time-of-check to time-of-use race condition; two concurrent runs can both pass the check before either has acted.
12. **C** — a `UNIQUE` constraint plus an atomic `INSERT ... ON CONFLICT DO NOTHING` puts the guarantee where only the database can truly enforce it atomically, closing the race window that application-level timing tricks (like A) can't reliably close.
13. **B** — `except: pass` discards the error with no trace, so failures surface later as confusing symptoms (like duplicate pickups) with no diagnostic information about their cause.
14. **B** — cron: no long-running process needed, simplest default for a standalone script. APScheduler: keeps a process alive in exchange for richer in-process scheduling and easier integration into an already-running service.
15. **C** — automating an inconsistent process just encodes and speeds up whichever version you happened to build against, without resolving the underlying disagreement. Fix or explicitly agree on the process first; automate second.

</details>

**Scoring:** 13+ → start Week 6. 10–12 → re-read the lecture sections behind your misses. <10 → re-read all three lectures from the top; process modeling and reliable automation compound directly into every remaining week of this course.
