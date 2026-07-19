# Lecture 2 — Finding What to Automate

> **Duration:** ~2 hours. **Outcome:** Given a BPMN diagram, you can score every human task as an automation candidate using a real rubric, back that score up with actual data queried from the SQL store, and explain — in one sentence a skeptical dispatcher would accept — why you're automating this step and not that one.

Drawing the process in Lecture 1 was the easy half. The hard half, and the one that separates a junior analyst from a senior one, is deciding **what to do about it**. Automating the wrong step wastes engineering time and can make the process *worse* — a bad decision made instantly by a computer is still a bad decision, just faster and harder to catch. This lecture gives you a repeatable way to decide.

## 1. The temptation to skip straight to code

Ten human tasks sat in the CrunchRide diagram from Lecture 1. A junior developer, handed that diagram, often reaches for the keyboard and starts automating the first one they understand well enough to code. That's backwards. The question isn't "can I automate this?" — with enough code, you can automate almost anything. The question is **"should I, and in what order?"**

There's a classic warning in process work, usually phrased as: *automating a broken process just lets you make mistakes faster.* If the dispatcher's severity judgment is inconsistent between two people, automating "assign whoever the dispatcher would have picked" doesn't fix the inconsistency — it locks it in and removes the one human who might have caught an obviously wrong call. **Fix or accept the process first. Automate second.**

## 2. A scoring rubric for automation candidates

For every **user task** or **manual task** in the diagram (service tasks are already automated — skip them), score four dimensions, each roughly high/medium/low:

| Dimension | Question | High score means |
|-----------|----------|-------------------|
| **Rule-based-ness** | Can the decision be written as an unambiguous `IF`/`THEN`, using data you already have? | Yes, entirely — no judgment call, no "it depends," no missing information |
| **Frequency / volume** | How often does this happen, and at what scale? | Happens constantly, or in bursts that overwhelm a human |
| **Error cost** | What happens when a human gets it wrong today? | Errors are costly, frequent, or hard to catch downstream |
| **Stability** | Does the *way* this task is done change often? | The rule has been the same for a long time and isn't about to change |

A task scores well as an automation candidate when it's **high on rule-based-ness and stability**, and the ROI is worth it when it's also **high on frequency or error cost**. A task that's high-frequency but genuinely requires judgment (not just "hasn't been written down as a rule yet") is not a good candidate — not this quarter, maybe not ever.

### Scoring the CrunchRide tasks

| Task | Rule-based? | Frequency | Error cost | Stability | Verdict |
|------|:-----------:|:---------:|:----------:|:---------:|---------|
| Read ticket, judge severity | Medium — "major" has real signal words but also judgment | High | Medium | Medium | **Not yet.** Leave to the human; revisit once you have a year of labeled tickets to check a rule against. |
| Assign nearby tech (minor path) | **High** — "active tech in the bike's zone with the fewest open tickets" is a complete, checkable rule | High | Medium (wrong assignment = delay, not danger) | High — the assignment policy hasn't changed in a year | **Automate first.** This is this week's build. |
| Flag needs-pickup | High — pure data write, no judgment | Medium | Low | High | **Automate** — trivial, do it as a side effect of the severity gateway once that exists. |
| Assign senior tech (major path) | Medium-High — "most senior *active* tech in zone" is nearly a rule, but "senior" isn't a column yet | Low (major tickets are rarer) | High (a bad assignment on a major issue is costly) | Medium | **Candidate for next quarter** — needs a `seniority` column added to `technicians` first; not ready today. |
| Inspect bike | Low — requires physically looking at a bike | Low | — | — | **Never.** Not a software problem. |
| Repair on the spot | Low — physical work | Low | — | — | **Never.** |
| Transport to depot | Low — physical work, though *scheduling* the transport could be automated later | Medium | Low | High | **Partial** — the physical task, no; a service task that auto-creates a "pickup needed" work order the moment a tech marks "not fixable," yes. Worth a note for later, not this week's build. |
| Depot repair-or-retire decision | Low — cost/benefit judgment, genuinely needs a person | Low | High (an over-eager auto-retire destroys a working asset) | — | **Never automate the decision itself.** You *could* automate surfacing the data that informs it (bike's age, repair history, cost-to-date) — that's a dashboard, not this week's script. |

Two tasks come out as clear, high-confidence "yes" for this week: **assigning the minor-path technician**, and the tiny **needs-pickup flag** that rides along with it. Everything else is either a physical task no software touches, or a judgment call that would be reckless to hand to a rule today. That's a real, defensible decision — and notice it took evidence and a rubric, not a hunch.

## 3. Don't guess the bottleneck — query it

"The dispatcher is slow" is a complaint. "The median time from ticket opened to technician assigned is 4 hours 51 minutes, and one ticket sat for over 3 days" is a finding. You have the data — every historical ticket in `repair_tickets` has `opened_ts` and `assigned_ts`. Use it.

```sql
-- How long did MANUAL assignment actually take, historically?
SELECT
    ticket_id,
    opened_ts,
    assigned_ts,
    ROUND(EXTRACT(EPOCH FROM (assigned_ts - opened_ts)) / 3600.0, 2) AS hours_to_assign
FROM repair_tickets
WHERE status IN ('completed')          -- these were manually assigned historically
ORDER BY hours_to_assign DESC;
```

```sql
-- Summary: median and worst-case time-to-assign
SELECT
    ROUND(AVG(EXTRACT(EPOCH FROM (assigned_ts - opened_ts)) / 3600.0), 2) AS avg_hours,
    ROUND(MAX(EXTRACT(EPOCH FROM (assigned_ts - opened_ts)) / 3600.0), 2) AS worst_hours,
    COUNT(*) AS tickets
FROM repair_tickets
WHERE assigned_ts IS NOT NULL;
```

Run this against the seed data and you'll see assignment times ranging from under an hour to over four — on a five-row sample. That spread *is* the bottleneck: it's not that the dispatcher is always slow, it's that assignment time is **unpredictable**, which is its own kind of failure. A rider whose bike sits flagged for four hours has a worse experience than one whose ticket takes 20 minutes even if the "average" looks fine. Automation's real promise here isn't just "faster," it's **consistent** — every ticket assigned in seconds, every time, whether the dispatcher is at their desk or not.

```sql
-- Right now: how big is the backlog sitting unassigned?
SELECT COUNT(*) AS open_unassigned
FROM repair_tickets
WHERE status = 'open' AND assigned_tech_id IS NULL;

-- Which zones are worst hit?
SELECT s.zone, COUNT(*) AS open_tickets
FROM repair_tickets rt
JOIN bikes b ON b.bike_id = rt.bike_id
JOIN stations s ON s.station_id = b.current_station_id
WHERE rt.status = 'open' AND rt.assigned_tech_id IS NULL
GROUP BY s.zone
ORDER BY open_tickets DESC;
```

This kind of query — reconstructing what actually happened from timestamps already sitting in your tables — is a lightweight form of **process mining**: using the system's own event log (here, `opened_ts`/`assigned_ts`/`completed_ts` on `repair_tickets`) to measure the real process, instead of trusting what people say it is in an interview. It's the SQL-and-Python-native way to do this analysis — no spreadsheet pivot table required, and every query is reusable the next time you want the same measurement.

## 4. Writing down the decision, not just making it

A rubric score in your head convinces nobody. Write the decision down where a skeptical stakeholder (or your future self, six months later, wondering why this task isn't automated) can see the reasoning:

```
DECISION LOG — CrunchRide repair-ticket automation, Week 5

Task: Assign nearby technician (minor-severity path)
Score: rule-based HIGH · frequency HIGH · error cost MEDIUM · stability HIGH
Decision: AUTOMATE. Rule = active tech, same zone as the bike's current
station, fewest currently-open tickets, under their max_open_tickets cap.
Evidence: median manual assignment time = ~2.8h, worst case 3+ days on
backlog buildup; zone imbalance visible in the open-ticket-by-zone query.

Task: Judge ticket severity (minor vs. major)
Score: rule-based MEDIUM · frequency HIGH · error cost MEDIUM · stability MEDIUM
Decision: DO NOT AUTOMATE (yet). "Major" mixes objective signals (keywords
like "brake," "battery") with judgment about rider safety. Revisit after
6 months of labeled data — a supervised classifier might earn its way in,
but a hand-written keyword rule risks silently missing a real safety issue.
```

This is the artifact you'll produce in Exercise 2 and again, for real, in the mini-project. Notice the second entry: **saying no, with a reason, is a legitimate and valuable output of this whole exercise.** Not automating something is a decision, not a failure to finish.

## 5. From decision to build order

Once you have a scored list, order the build by (a) confidence in the "automate" verdict and (b) blast radius if you're wrong. This week's build order:

1. **Needs-pickup flag** — trivial, zero judgment, safe even if buggy (worst case: a bike gets flagged that didn't strictly need it, a human notices and unflags it).
2. **Technician assignment** — the real target. Higher value, still fully rule-based, but a wrong assignment costs someone a wasted trip — worth the extra care Lecture 3 spends on logging and error handling.
3. Everything scored "not yet" stays a human task, on purpose, with the reasoning on record.

That ordering — cheap-and-safe first, valuable-but-needs-care second, judgment-calls never — is the shape almost every real automation backlog should take.

## 6. Check yourself

- Why is "can I automate this?" the wrong first question?
- Name the four dimensions of the scoring rubric. Which two matter most for *whether* to automate at all, and which two matter for *how much value* it delivers?
- Why did the depot repair-or-retire decision score "never automate," even though it's high error cost?
- What's process mining, in one sentence, and what columns in `repair_tickets` let you do a lightweight version of it?
- Why is "the dispatcher is slow" a weaker statement than a query showing the actual hours-to-assign distribution?
- What should you do with a task that scores "not yet"? Just leave it alone silently, or something else?

If those are automatic, Lecture 3 builds the real thing: the technician-assignment automation, made trustworthy enough to run unattended.

## Further reading

- **Van der Aalst, "Process Mining: Data Science in Action" (overview article, free):** <https://www.processmining.org/>
- **Camunda — "What is Business Process Automation?" (accessible overview, vendor-neutral concepts):** <https://camunda.com/best-practices/business-process-automation/>
- **PostgreSQL — date/time functions (`EXTRACT`, `AGE`, interval math you'll use constantly for this kind of analysis):** <https://www.postgresql.org/docs/current/functions-datetime.html>
- **Martin Fowler, "InfrastructureAsCode" and general writing on automating processes deliberately, not reflexively:** <https://martinfowler.com/bliki/>
