# Lecture 1 — Eliciting Requirements

> **Duration:** ~2 hours. **Outcome:** You can map a project's stakeholders by power and interest, choose the right elicitation technique for each one, ladder a vague complaint down to a specific testable need in a live conversation, and record what you learned in a queryable requirements register instead of a shared spreadsheet.

Every broken software project you've ever heard of — the redesign nobody wanted, the app that solved the wrong problem, the six-month build that got cancelled — has the same root cause more often than any other: **nobody wrote down the real requirement, because nobody found it.** This lecture is about finding it.

## 1. What a requirement actually is

A **requirement** is a statement of what a system must do or a quality it must have, written precisely enough that you could later check — objectively, without asking the person who wrote it — whether the finished system satisfies it.

That definition rules out most of what stakeholders say in the first meeting. "Make it faster." "It should be easy to use." "We need better reporting." None of these are requirements yet. They're **symptoms** — real pain, pointing at a real problem, but not yet specific enough to build against. Your job as an analyst is to turn a symptom into a requirement without inventing needs the stakeholder didn't actually have. That's a narrow path: ask too few questions and you build the wrong thing precisely; ask too many leading questions and you build *your* idea of the right thing, dressed up as theirs.

## 2. Stakeholder mapping — who do you even talk to?

A **stakeholder** is anyone who affects, is affected by, or has a legitimate interest in the system — not just the person who requested it. Before you interview anyone, map the field. The standard tool is the **power/interest grid**:

| | **Low interest** | **High interest** |
|---|---|---|
| **High power** | *Keep satisfied* — inform them, don't overwhelm them | *Manage closely* — your key interviews, your sponsors |
| **Low power** | *Monitor* — light touch, watch for change | *Keep informed* — they'll use the system daily; their detail is gold |

At Meridian Outfitters, the special-order problem touches at least eight people (seeded in `stakeholders` — see the [week README](../README.md)):

```sql
SELECT name, role, power_level, interest_level FROM stakeholders ORDER BY power_level DESC, interest_level DESC;
```

Read the grid off that query:

- **High power / high interest ("manage closely"):** Dana Reyes (Store Manager) and Sofia Nakamura (Regional Ops Director, the executive sponsor). You interview these two first and in depth — and you take their opinions as *important*, not as *final*. Power doesn't mean their stated need is the correct one; it means their buy-in decides whether the project survives.
- **Low power / high interest ("keep informed"):** Jordan Blake (Store Associate) and Tom Alvarez (Call Center Lead). These are the people who will use the system every single day and never got asked in a typical bad project. Their detail is usually the richest in the whole analysis — they know exactly where the process actually breaks, because they're the ones improvising a fix for it right now.
- **Medium power / high interest:** Marcus Webb (Warehouse) and Grace Kim (IT). They control constraints — what the DC can realistically report, what systems can realistically integrate — that will kill an otherwise-good requirement if you don't surface them early.
- **High power / medium interest:** Priya Anand (Finance/Loss Prevention). She isn't watching the project day to day, but she has veto power over anything touching money, and finding that out in week 10 instead of week 2 is how requirements churn.

**A common beginner mistake:** interviewing only the person who called the meeting (usually high-power) and treating that as "gathering requirements." A requirement elicited from one stakeholder, especially the most powerful one, is a *request*, not yet a requirement — it hasn't been checked against the people who will actually live with the system.

## 3. Elicitation techniques — and when to use each

| Technique | Best for | Weakness |
|---|---|---|
| **Interview** | Deep, individual understanding of goals, pain, constraints | Slow (one person at a time); people describe what they *think* they do, not always what they *actually* do |
| **Observation / job shadowing** | Seeing the real process, including workarounds nobody would think to mention | Time-consuming; presence can change behavior (people act more "correctly" when watched) |
| **Document review** | Fast baseline — existing forms, tickets, the current spreadsheet, error logs | Documents describe the *stated* process, which often diverges from the real one |
| **Survey / questionnaire** | Reaching many stakeholders (e.g. customers) cheaply | Shallow; can't follow up on an interesting answer |
| **Workshop / JAD session** | Resolving disagreement between stakeholders in the room together | Requires a skilled facilitator or the loudest voice wins by default |

No single technique is sufficient alone — this is **triangulation**: confirm what you heard in an interview against what you saw in observation and what the documents say. At Meridian, three techniques already converged on the same finding without anyone planning it that way:

- **Interview** (Dana): associates wait 15–40 minutes on hold with the DC before they can promise a customer a date.
- **Observation** (Jordan): a single special order took 3 phone calls, a sticky note, and 2 spreadsheet edits — 22 minutes, matching Dana's complaint from the inside.
- **Document review** (the spreadsheet itself): order status is a free-text cell with values `"ordered"`, `"Ordered"`, `"in transit"`, `"?"`, and blank — which explains *why* nobody trusts it enough to skip the phone call in the first place.

Three techniques, one root cause. That convergence is what lets you write a requirement with confidence instead of a guess.

## 4. Interview technique — getting past "make it faster"

A first-meeting answer like "make it faster" is not a lie or laziness — it's the stakeholder handing you a *symptom* and trusting you to find the disease. Your tool for that is **laddering**: a chain of specific, non-leading follow-up questions that walks a vague statement down to something measurable.

### The Five Whys, applied

```
Dana:  "The special order process needs to be faster."
You:   "Walk me through the last one that felt slow. What happened, step by step?"
Dana:  "A customer wanted a jacket we didn't have. Jordan called the DC, was on
        hold 25 minutes, then had to call back because the line dropped."
You:   "Why does confirming stock require a phone call at all?"
Dana:  "Because we can't see the DC's actual inventory — only what our own
        store's system shows."
You:   "If Jordan could see DC stock directly, would the call go away?"
Dana:  "Yes — she'd just need to know it's genuinely available before quoting
        a date."
You:   "So the real ask is: an associate can see live DC stock, in the store
        system, without calling anyone. Is that the need, or is there more?"
Dana:  "That's it. And then the customer needs to know when it actually ships,
        without calling us either."
```

Five exchanges took "make it faster" to two testable candidate requirements:

- Store associates can view current DC stock levels without a phone call.
- Customers can check order status without calling the store.

Neither of those is a finished requirement yet — "current," "check," and "without calling" all still need precision (how current? within what time? through what channel?). That precision work is Lecture 2. What laddering gets you is the *right target* to make precise — the actual need, not the first sentence someone said in a hallway.

### Rules for the interview itself

- **Ask open questions first, closed questions last.** "Tell me about the last time this went wrong" surfaces things you didn't know to ask about. "Do you need a mobile app?" presupposes a solution and will get you a yes/no about *your* idea, not their need.
- **Ask for a specific recent example, not a generality.** "What usually happens" invites a sanitized, idealized answer. "Tell me about last Tuesday" gets you the real, messy process — including the workaround.
- **Treat every workaround as a requirement in disguise.** If someone has invented a sticky-note system, a personal spreadsheet, or a "call Marcus's cell directly" back channel, that workaround is proof a real need exists and the official system doesn't meet it. Write it down verbatim.
- **Mirror back what you heard, in your own words, before moving on.** "So if I have this right: the block isn't placing the order, it's not knowing if it'll actually ship on time — did I get that right?" This catches your own misunderstanding while it's cheap to fix.
- **Don't let power dictate truth.** If the VP's account of the process contradicts the associate who does it every day, that contradiction is data, not a problem to smooth over — write both down and go find out which one (or both, partially) is right.
- **Never let the stakeholder design the solution for you in the interview** — "we need a mobile app" is a solution, not a need. Follow it with "what would that let you do that you can't do today?" and you'll usually find the real requirement sitting one layer under the proposed feature.

## 5. Recording what you learn — in SQL, not a spreadsheet

This is the exact moment most teams reach for a shared Excel file — a "requirements tracker" tab that becomes the very same kind of single-copy, no-audit-trail, silently-diverging artifact you're trying to replace at Meridian. Don't. This course's data rule applies here as much as anywhere: elicitation notes are structured, queryable records, and they belong in SQL.

You already have the `elicitation_sessions` table (seeded in the [week README](../README.md)). After every interview, observation, or document review, insert a row:

```sql
INSERT INTO elicitation_sessions (session_id, stakeholder_id, session_date, method, summary)
VALUES (7, 5, '2026-06-05', 'interview',
        'Sofia confirmed the KPI she cares about is fulfillment time (order
         placed to customer notified) and complaint volume, not headcount.');
```

And because it's SQL, you can immediately ask questions a spreadsheet makes you eyeball by hand:

```sql
-- Who has NOT been interviewed yet, ranked by how much it matters that we get to them?
SELECT s.name, s.role, s.power_level, s.interest_level
FROM stakeholders s
LEFT JOIN elicitation_sessions e ON e.stakeholder_id = s.stakeholder_id
WHERE e.session_id IS NULL;

-- Which findings came from more than one technique (triangulated, higher confidence)?
SELECT stakeholder_id, method, summary FROM elicitation_sessions ORDER BY session_date;
```

That first query is exactly the kind of check a power/interest grid on paper quietly lets you skip — in a live project with 30+ stakeholders instead of 8, you will not eyeball that gap correctly by hand.

## 6. Common elicitation pitfalls

- **Leading questions.** "Don't you think a dashboard would help?" gets you agreement, not a requirement. Ask what problem exists before proposing any form of solution.
- **Single-source bias.** One glowing interview with the sponsor is not "requirements gathered." Triangulate against at least one other technique before you trust a finding.
- **Solutioneering too early.** Stakeholders (and analysts) love to jump to "we need an app for that." Every proposed solution deserves the follow-up: *what need does that meet, specifically?*
- **Skipping the low-power, high-interest people.** Jordan Blake, the associate, had never been asked anything before this project — and her 2-hour shadow session is the richest data point in the whole set. The org chart is not a proxy for who knows the truth.
- **Scope creep starting in elicitation, not construction.** Every "while we're at it, could it also…" from an interview is a new candidate requirement, not an automatic yes. Log it, don't promise it — prioritization is Lecture 2's job, not the interview's.

## 7. Check yourself

- Why is "make it faster" not yet a requirement, even though it's honest and important?
- Name two techniques you'd use to confirm an interview finding, and why one technique alone isn't enough.
- Where does Jordan Blake sit on the power/interest grid, and why does that matter for how you treat her input?
- What does it mean to "ladder" a vague complaint, and what's the danger of stopping too early?
- Why is a workaround (a personal spreadsheet, a sticky note) evidence of a requirement rather than a distraction from one?
- Why does this course store elicitation notes in SQL tables instead of a shared spreadsheet — what specific failure mode does that avoid?

If those are automatic, Lecture 2 turns the laddered, triangulated findings from this lecture into requirement statements precise enough to build — and to test.

## Further reading

- **IIBA BABOK (Business Analysis Body of Knowledge) — Elicitation & Collaboration:** <https://www.iiba.org/business-analysis-body-of-knowledge/>
- **ISO/IEC/IEEE 29148:2018 — Systems and software engineering, requirements engineering:** <https://www.iso.org/standard/72089.html>
- **PostgreSQL — `INSERT`:** <https://www.postgresql.org/docs/current/sql-insert.html>
- **PostgreSQL — `JOIN` basics (for the "who hasn't been interviewed" query):** <https://www.postgresql.org/docs/current/queries-table-expressions.html>
