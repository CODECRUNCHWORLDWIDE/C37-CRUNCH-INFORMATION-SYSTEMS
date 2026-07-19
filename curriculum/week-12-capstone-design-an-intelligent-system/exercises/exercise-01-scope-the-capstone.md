# Exercise 1 — Scope the Capstone Organization and System

**Estimated time:** 1.5 hours

Every system you've built this course had its organization chosen for you (Crunch Cycles). Your capstone doesn't. This exercise forces the scoping decision that Lecture 2 says is the single biggest predictor of a strategically sound system: a specific organization, a specific pain, and an honest boundary around what you will and won't build.

## Part A — Pick your organization (15 min)

Choose one real organization. In order of preference:

1. **An organization you actually have access to** — your employer, a family business, a club, a nonprofit, a school department, a local shop whose owner you can talk to for 20 minutes.
2. **A fallback, if nothing above is available** — pick one of these three and treat it as real (research it lightly, e.g. its public website, so your requirements aren't invented from nothing):
   - A small independent bookstore with 2 locations, no current inventory system beyond a paper log.
   - A neighborhood veterinary clinic scheduling appointments by phone and a wall calendar.
   - A youth sports league tracking rosters, game schedules, and volunteer coaches on a shared spreadsheet.

Write one paragraph in `01-organization.md`: what the organization does, roughly how many people work there, and what their current information-handling looks like today (spreadsheet? paper? nothing?). This grounds everything that follows — you cannot design a system for an organization you haven't described.

## Part B — The strategic purpose sentence (20 min)

Using Lecture 2's template, write:

> "This system helps **[organization]** do **[specific thing]** better by **[specific mechanism]**."

Then answer, in 3–4 sentences each, in the same file:

1. **What is the current pain, concretely?** Not "they need a database" — what specific, recurring problem does the current (spreadsheet/paper/nothing) approach cause? Give a real or realistic example.
2. **Who feels that pain first?** Name the role (not a person) — the volunteer coordinator, the shop owner, the front-desk scheduler.
3. **What does "better" mean, measurably?** A number or a concrete before/after, not "more efficient." ("Currently takes 3 days to compile a monthly report by hand; should take under 5 minutes.")

## Part C — Scope boundary (30 min)

List, in two columns in `02-scope.md`:

- **In scope for this capstone** — the entities, workflows, and layers (from Lecture 1's six-layer stack) you will actually design and build evidence for this week.
- **Explicitly out of scope** — real things this organization needs eventually, that you are deliberately not doing this week, and one sentence why.

Example shape (don't copy verbatim — this is illustrative, your organization is different):

```
IN SCOPE
- Customer/member data model (SQL schema)
- One automated workflow (e.g., appointment reminders)
- Basic role-based access (staff vs. admin)
- One dashboard KPI
- One AI feature tied to the strategic purpose sentence

OUT OF SCOPE (this week)
- Payment processing integration — real PCI compliance work is a
  project on its own; we'll note it as a Phase 5 in the delivery plan
- Mobile native app — a responsive web view covers the MVP; native
  app is a future phase, not a capstone requirement
- Multi-location inventory sync — this org has one location; design
  the schema so it *could* extend, but don't build the sync job
```

A capstone that tries to scope in everything is a capstone that ships nothing. Being explicit about what's deliberately excluded — and why — is itself graded; it's evidence you understand the difference between "everything this organization could ever want" and "what a phased, deliverable system covers first."

## Part D — Data you'll need (25 min)

List the core entities your system will need to track (you'll formalize this into a schema in the mini-project, but name them now):

1. For each entity, name it and give 3–5 attributes you already know it needs.
2. Mark which entities already exist *somewhere* (a spreadsheet, a paper form) versus which are new.
3. Note **one** entity where you genuinely don't know the right structure yet — every real project has at least one. Write the specific question you'd need to ask a stakeholder to resolve it (this is a Week 2 requirements-analysis skill, applied here).

## Deliverable

A folder `c37-week-12/exercise-01/` containing:

- `01-organization.md` — the organization description + strategic purpose sentence + Part B answers
- `02-scope.md` — the in/out-of-scope table
- `03-entities.md` — Part D's entity list with the one open question flagged

## Expected outcome

A one-page-equivalent, honest scoping document that names a real pain for a real (or realistic) organization, states measurably what "better" means, and draws an explicit, defensible line around what this week's system will and won't cover. This is the input to Exercise 2 — you cannot draw a coherent architecture for a scope that doesn't exist yet in writing.

Next: [Exercise 2 — Draft the architecture](exercise-02-draft-the-architecture.md).
