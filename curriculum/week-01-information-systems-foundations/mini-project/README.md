# Mini-Project — Map a Real Business as an Information System

> Pick a real organization — your job, a family business, a student club, a well-known company you can research from the outside — and build a complete `is_components`/`is_flows` register for it, the same discipline you've used on Riverbend all week. This is the week's capstone: proof you can *see* a system that nobody has already diagrammed for you.

**Estimated time:** 2.5–3 hours, best done Saturday after the exercises and challenges.

## Why a real business, not Riverbend

Every exercise and challenge this week gave you a system that was already 90% mapped — you extended and diagnosed it. That's the right way to *learn* the vocabulary, but it hides the actual hard part of this skill: **going from nothing**. A stakeholder never hands you a finished `is_components` table. They hand you a business that runs, somehow, and your job is to look at it and produce the map yourself. That's what this project tests.

## Choosing your organization

Pick something you have real visibility into — you'll produce much better work modeling a business you actually understand than one you're guessing about from a press release. Good choices, roughly easiest to hardest:

- **Your own job** (any role — retail, food service, an office, healthcare, anything with more than one person and more than one step).
- **A family business** or a business you've worked at in the past.
- **A student organization or club** — smaller, but still has all four components if you look closely (an officer is people, "how we plan an event" is process, a membership list is data, a group chat or shared doc is technology).
- **A well-known company**, researched from the outside (their public "how it works" pages, app, or your own experience as a customer) — hardest, because you can't interview anyone, so be honest in your report about what you had to infer versus what you know for certain.

If you genuinely can't think of one, model **your own daily routine as a one-person information system** — it still has all four components (you're the people; getting ready, commuting, working are processes; your calendar and to-do list are data; your phone and apps are technology) and is a legitimate, if smaller, submission.

## Deliverable

A directory in your portfolio `c37-week-01/mini-project/` containing:

1. **`register.sql`** — a full, runnable `CREATE TABLE` + `INSERT` script for your organization, using the exact same `is_components` and `is_flows` schema from the [week README](../README.md) (you may add columns if your organization genuinely needs them — document any addition in `report.md`).
2. **`report.md`** — a written report, structured as the sections below.
3. **`diagram`** *(optional but recommended)* — a hand-drawn or digital diagram of your components and flows (a photo of a whiteboard/napkin sketch is completely fine; this isn't a design course). If you skip the diagram, your `report.md` needs to describe the shape of the system clearly enough in words that someone could redraw it.

## Minimum scope

- **At least 12 components total, with at least 2 in each of the four categories** (people, process, data, technology). More is fine if your organization genuinely has more moving parts — don't pad it artificially.
- **At least 10 flows** connecting them. Every component should participate in at least one flow — a component with zero flows either doesn't belong in your map, or you haven't finished tracing how it connects (see Exercise 2 Part C's dead-end check; run the same query against your own register).

## `report.md` structure

### 1. The organization (2–3 sentences)

What it is, what it does, and — critically — **where you drew your system's boundary** (Lecture 1 §4). You cannot map "the whole business" in 2.5 hours; say explicitly what's in scope and what you deliberately left out.

### 2. The four components (one short paragraph each)

For people, process, data, and technology: what's actually there, and was anything genuinely hard to classify? If so, which item, and how did you resolve it using the checklist from Lecture 2 §6?

### 3. Three flows worth explaining

Pick your three most important flows and, for each, apply Lecture 3 §4's approach: what value does this flow create, and what would concretely go wrong if it silently stopped?

### 4. Value chain placement

Using Lecture 3 §2, classify at least three of your processes as primary or support activities, and name which specific value-chain stage each belongs to.

### 5. The weakest link

Identify the single component or flow in your map that you'd fix first if you were hired to improve this organization's information system — and why. Use the gap → consequence → impact → cost → value pattern from Lecture 3 §5. This is the single most important paragraph in your report — it's the difference between "I made a diagram" and "I did an analysis."

### 6. Reflection (150–250 words)

What surprised you about mapping a real system versus working with Riverbend's already-built one? Was anything about your organization harder to classify than anything in this week's lectures prepared you for?

## Grading rubric (self-check before you call it done)

- [ ] `register.sql` runs cleanly against a fresh PostgreSQL or SQLite database with no errors.
- [ ] At least 12 components, 2+ per category, at least 10 flows.
- [ ] No dead-end components (or any left are explicitly justified in `report.md`).
- [ ] Section 5 ("weakest link") names a *specific* component or flow, not a vague complaint — and traces a real consequence, not just "it's inefficient."
- [ ] You could hand `report.md` to someone who has never seen your organization and they'd understand how it runs.

## Up next

[Week 2 — Systems Analysis & Requirements](../../week-02-systems-analysis-and-requirements/) takes the "weakest link" you just identified and teaches you the discipline for turning it into a requirements spec someone could actually build against.
