# Exercise 2 — Map the Data Flows

> **Time:** ~1.5 hours. **Uses:** Lecture 1 §3, Lecture 3 §4. Works against `is_components` and `is_flows`. Do Exercise 1 first — you need components `21`–`25` to exist before you can connect flows to them.

A system map that's just a list of components is only half the picture — Lecture 3 was explicit that most of the value (and most of the risk) lives in the **connections**, not the parts. This exercise makes you build and query those connections directly.

## Part A — Read the existing flows (15 min)

Run this against your seeded data (it's a `JOIN`, a preview of Week 3 — read the output, don't worry about writing your own yet):

```sql
SELECT f.flow_id, s.name AS source, t.name AS target, f.data_exchanged, f.frequency
FROM is_flows f
JOIN is_components s ON s.component_id = f.source_component_id
JOIN is_components t ON t.component_id = f.target_component_id
ORDER BY f.flow_id;
```

Read all 19 rows top to bottom. For three of them, write one sentence each explaining, in plain English, what would go wrong at Riverbend if that specific flow simply stopped happening. (Lecture 3 §4 walked through flow `9` as a worked example — pick three *different* ones.)

## Part B — Add the missing flows (45 min)

Your Exercise 1 additions (components `21`–`25`) are currently **disconnected** — they exist in `is_components` but nothing in `is_flows` references them yet. A component with no flow in or out is either genuinely isolated (rare, and worth double-checking) or it means you haven't finished mapping the system.

Write `INSERT` statements connecting your new components into the existing system with **at least four new flows**. Use `flow_id` values `20`–`23` or higher. For each flow, decide a `frequency` (`real-time`, `daily`, `weekly`, or `on-demand`) and write a short `data_exchanged` description. At minimum, include:

1. A flow **into** your complaint-logging process from somewhere a complaint would actually originate (hint: which existing component would a customer complaint usually follow — Retail's POS Checkout? Subscription Renewal? Pick one and justify it).
2. A flow **from** your complaint process **to** your complaint-ticket data component (the process writes the record — same pattern as flow `3`, Order Intake → Customer Order, in the seed).
3. A flow from your bookkeeper (people) to whatever technology they use.
4. A flow from your complaint-ticket data **to** Nora Chen (people) — she needs visibility into support issues, the same way she already gets visibility into sales (flow `12`) and shipping (flow `19`).

## Part C — Find the dead ends (30 min)

A "dead end" component is one that never appears as a `source_component_id` **or** a `target_component_id` in `is_flows`. Before you had SQL, you'd have to eyeball two long lists and compare them by hand — tedious and error-prone. Do exactly that by hand once, for practice, then check your work with a query:

**By hand:** list every `component_id` from `1` to `25`. Cross off every id that appears anywhere in `is_flows`, in either column. Whatever's left is a dead end.

**By query** (uses `NOT IN`, a small step beyond what you've used so far — read it slowly, it's just "give me components whose id never shows up in either flow column"):

```sql
SELECT component_id, category, name
FROM is_components
WHERE component_id NOT IN (SELECT source_component_id FROM is_flows)
  AND component_id NOT IN (SELECT target_component_id FROM is_flows)
ORDER BY component_id;
```

**Expected:** your hand count and your query result should match exactly. If they don't, you missed a flow somewhere in Part B or your hand-count — go find the discrepancy before moving on; don't just trust the query blindly.

For every dead end left standing, either (a) add the flow that connects it — go back and do that now — or (b) write one sentence in `notes.md` defending why it's genuinely, correctly isolated (this should be rare).

## Deliverable

- `week01-solutions.sql` — your Part B `INSERT` statements, appended after Exercise 1's.
- `notes.md` — your three Part A sentences, and any Part C dead-end justifications.

## Check yourself

- After Part B, run the Part C query again. Ideally the only rows left (if any) are ones you can genuinely justify.
- Which existing seeded component has the *most* incoming flows? What does that tell you about how central it is to the system? *(Hint: look back at the Part A output.)*
- What's the difference between a flow that's missing because nobody's built it yet, versus a flow that's missing because it genuinely shouldn't exist? Give one Riverbend example of each.
