# Lecture 2 — Writing Requirements That Hold

> **Duration:** ~2 hours. **Outcome:** You can classify a requirement as functional or non-functional, rewrite a vague statement into one that's clear, testable, atomic, and feasible, and prioritize a full requirements list with MoSCoW — storing all of it in the `requirements` SQL table so it can be queried, filtered, and traced, not just read top to bottom.

Lecture 1 got you from "make it faster" to two candidate needs. This lecture is about writing those needs down so precisely that they *hold* — meaning six months from now, when the system is built, anyone (not just you) can look at the requirement and the finished feature and say, unambiguously, pass or fail.

## 1. Functional vs. non-functional requirements

A **functional requirement (FR)** describes something the system *does* — a behavior, a feature, an output given an input.

> FR-001: The system shall let a store associate look up an item's current warehouse stock quantity by SKU.

A **non-functional requirement (NFR)** describes a *quality* the system must have while doing what it does — how fast, how available, how secure, how usable, how much load it can take. NFRs are just as buildable and testable as FRs, but they're easy to skip because nobody demos "the system was available 99.5% of the time" the way they demo a button.

> NFR-001: Warehouse stock lookups shall return a result within 2 seconds for 95% of requests during store hours (7am–9pm local).

**The test that separates them:** if you can point at a single feature or action and say "the system did X," it's functional. If you're describing a constraint that applies *across* many features — speed, uptime, who's allowed to see it, how many people can use it at once — it's non-functional.

### The common NFR categories

| Category | What it constrains | Meridian example |
|---|---|---|
| **Performance** | Speed, response time, throughput | Stock lookup returns in <2s for 95% of requests |
| **Availability** | Uptime, when the system must be up | System available 99.5% during store hours, 95% overnight (maintenance window allowed) |
| **Usability** | How easily the intended user succeeds | A store associate with 10 minutes of training can place a special order without help |
| **Security** | Who can do/see what, how access is controlled | Only Store Managers and above can approve orders over $500 |
| **Scalability** | Behavior as load grows | System supports all 14 stores placing orders simultaneously during a Saturday peak without degrading lookup speed |
| **Compliance** | Legal, regulatory, or audit requirements | Every status change is logged with user, timestamp, and old/new value, retained 3 years |
| **Maintainability** | Cost/effort to change or extend later | A new store can be added to the system without a code change |

A system with perfect functional requirements and zero non-functional requirements is exactly how you get a demo that works flawlessly on a laptop and falls over the first Saturday it's used by 14 stores at once. Both categories are mandatory; NFRs are just quieter about it.

## 2. The qualities of a good requirement

Six qualities separate a requirement that *holds* from one that quietly rots into an argument later. Run every requirement you write through this checklist.

1. **Clear / unambiguous.** One reasonable reading, not several. "The system should be secure" has as many readings as there are people in the room. "Only authenticated users with the `store_manager` role or higher can approve orders over $500" has one.
2. **Testable / verifiable.** You can point at the finished system and say pass or fail, with no argument. "Fast" is not testable. "Returns in under 2 seconds for 95% of requests" is — you can literally write the test.
3. **Atomic.** One requirement, one testable statement. "The system shall let associates place orders and notify customers and log approvals" is three requirements wearing one ID — if one part fails, which part failed? Split it.
4. **Feasible.** Achievable with the time, budget, and technology actually available. "99.999% uptime" sounds impressive and is wildly out of proportion for a 14-store retail chain's internal tool — feasibility isn't cynicism, it's honesty about tradeoffs.
5. **Traceable.** Linked back to the stakeholder need (and, later, forward to the design, code, and test that satisfy it) so you can answer "why does this requirement exist?" and "who asked for this?" without guessing. That's exactly why `requirements.source_stakeholder_id` exists in this week's schema.
6. **Prioritized.** Not everything is equally important, and pretending otherwise (an unprioritized wall of 80 "requirements") guarantees the wrong ones get built first when time runs short. That's MoSCoW, section 4 below.

### Rewriting a vague requirement — worked example

Raw meeting note from Dana: *"The new system needs to be user-friendly for associates."*

Run it through the checklist:

- **Clear?** No — "user-friendly" means something different to every reader.
- **Testable?** No — there's no observable pass/fail condition.
- **Atomic?** Unclear until it's specific — "user-friendly" might secretly be five separate requirements (fast to learn, few clicks, readable on the small POS screen, forgiving of mistakes, works without training).

Laddering (Lecture 1's tool, reused here) turns "user-friendly" into something specific: *what, concretely, would make Jordan (the associate who's actually never been asked) call this easy?* Interview follow-up reveals: she wants to place an order in fewer steps than the current phone-call process, on the same small screen she already uses for POS, without needing training beyond a single walkthrough.

Rewritten:

> NFR-002: A store associate who has completed the one-time system walkthrough shall be able to place a special order in 4 or fewer screen steps, on the existing 10-inch POS terminal, without contacting the warehouse or another store.

Every vague word is gone and replaced with something you could put a stopwatch and a click-counter on. That's the transformation this whole lecture teaches.

### More before/after pairs

| Vague (rejected) | Testable (accepted) |
|---|---|
| "The system should have good reporting." | "A Store Manager can generate a report of all open special orders older than 5 business days, filterable by store, in under 10 seconds." |
| "It needs to be reliable." | "The system shall have 99.5% uptime measured monthly, excluding a published maintenance window." |
| "Customers should be notified." | "The system shall send an SMS or email to the customer within 15 minutes of an order status changing to 'ready for pickup'." |
| "Only the right people should see sensitive data." | "Only users with role `finance` or `admin` may view an order's payment or discount-approval history." |

Notice the pattern: every rewrite adds a **number, a role, a channel, or a condition**. If your requirement has none of those four, it probably isn't testable yet.

## 3. A requirement statement template

Use a consistent shape so every requirement is parseable at a glance:

> **[ID]** The [system / role] shall [testable behavior or quality] [under what condition / by when / how measured].

```sql
INSERT INTO requirements (req_id, req_type, statement, priority, source_stakeholder_id, status)
VALUES
('FR-001', 'functional',
 'The system shall let a store associate look up current warehouse stock quantity by SKU without contacting the warehouse.',
 'must', 1, 'draft'),
('NFR-001', 'non-functional',
 'Warehouse stock lookups shall return a result within 2 seconds for 95% of requests during store hours (7am-9pm local).',
 'must', 2, 'draft');
```

Give every requirement a stable ID (`FR-###` / `NFR-###`) the moment you write it, even in draft. IDs are how you'll trace a requirement to a use case, a test, and eventually a line of code in later weeks — renumbering later breaks every link you've built.

## 4. Prioritizing with MoSCoW

Not every "must-have" is one. **MoSCoW** forces the conversation stakeholders avoid — what actually ships first if time runs out:

| Category | Meaning | Meridian example |
|---|---|---|
| **Must** | The system fails its purpose without this. Non-negotiable for launch. | Associates can see live warehouse stock (the #1 root cause from Lecture 1) |
| **Should** | Important, real value, but the system still functions without it at launch. | Automatic SMS notification when order status changes |
| **Could** | Desirable if time and budget allow; genuinely optional. | A store-to-store transfer option in addition to store-to-warehouse |
| **Won't** (this time) | Explicitly out of scope for *this* release — written down so it stops being re-litigated every meeting. | Predictive stock forecasting / auto-reorder |

The "Won't" category is the most underused and most valuable. Writing "Won't: predictive forecasting (candidate for a future phase)" does something a silent omission doesn't: it tells every stakeholder in the room, in writing, that the idea was heard and deliberately deferred — not forgotten, not rejected out of ignorance. That single sentence prevents more scope-creep arguments than any other technique in this lecture.

**A rule of thumb for the split:** aim for roughly 60% Must, 20% Should, 20% Could on a first pass — if 80% of your list is "Must," you haven't prioritized, you've just relabeled everything.

Query the register to sanity-check your own prioritization discipline:

```sql
SELECT priority, COUNT(*) AS n
FROM requirements
GROUP BY priority
ORDER BY CASE priority WHEN 'must' THEN 1 WHEN 'should' THEN 2 WHEN 'could' THEN 3 ELSE 4 END;
```

If that query comes back almost entirely `must`, go back and re-examine each one against the definition above — "the system fails its purpose without this" is a high bar, deliberately.

## 5. Traceability — why `source_stakeholder_id` matters

Every requirement should answer "who needs this, and why?" without a follow-up meeting. That's what the foreign key to `stakeholders` is for:

```sql
-- Every requirement, with who asked for it and why it matters to them
SELECT r.req_id, r.statement, r.priority, s.name AS requested_by, s.primary_goal
FROM requirements r
JOIN stakeholders s ON s.stakeholder_id = r.source_stakeholder_id
ORDER BY r.priority;
```

This matters for a very practical reason: when a requirement gets challenged in month 4 ("why are we building this again?"), you want an answer that's a query, not a memory.

## 6. Check yourself

- What's the one-sentence test that separates a functional requirement from a non-functional one?
- Rewrite "the system should load quickly" into a testable NFR. What number and condition did you add?
- Why is "the system shall let associates place orders and notify customers" a bad, non-atomic requirement? Split it.
- What does "feasible" add to a requirement that "testable" alone doesn't catch?
- Why does writing "Won't" explicitly matter more than just leaving something off the list?
- If your requirements list is 90% "Must," what's most likely wrong, and how do you fix it?

If those are automatic, Lecture 3 takes these requirement statements and turns the *behavioral* ones into use cases and user stories a development team can build sprint by sprint.

## Further reading

- **ISO/IEC/IEEE 29148:2018 — Requirements characteristics (the "clear, testable, atomic…" list, formalized):** <https://www.iso.org/standard/72089.html>
- **IIBA BABOK — Requirements Analysis & Design Definition:** <https://www.iiba.org/business-analysis-body-of-knowledge/>
- **DSDM Consortium — the original MoSCoW method:** <https://www.agilebusiness.org/dsdm-project-framework/moscow-prioritisation.html>
- **PostgreSQL — `GROUP BY` and aggregates:** <https://www.postgresql.org/docs/current/tutorial-agg.html>
