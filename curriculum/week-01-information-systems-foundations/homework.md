# Week 1 — Homework

Five problems, ~5 hours total, spread across the week. These reinforce the lectures with a mix of installation, classification drills, register work in SQL, a short essay, and a warm-up build. Commit each.

Unless a problem says otherwise, work against the `riverbend` database from the [week README](./README.md).

---

## Problem 1 — Set up both engines (45 min)

Get **both** PostgreSQL and SQLite working, so you can feel the difference firsthand — you'll need at least one working for every remaining week of this course.

1. Install PostgreSQL 16+ and SQLite 3.35+ (see [`resources.md`](./resources.md)).
2. Create the `riverbend` database in Postgres and `riverbend.db` in SQLite, and load the seed from the README into **both**.
3. Run `SELECT COUNT(*) FROM is_components;` in each — both must return `20`.

**Deliver** `setup.md` with: the version of each engine (`SELECT version();` in Postgres, `SELECT sqlite_version();` in SQLite), and one sentence on a difference you noticed connecting to each.

---

## Problem 2 — Twenty classification calls (60 min)

Classify each item below as **people**, **process**, **data**, or **technology**. Write your answer plus one clause of justification for each — not just a bare letter. Put these in `classify.md`, numbered to match.

1. A hospital nurse's shift-change checklist.
2. A patient's vital-signs reading, taken at 3:14 PM.
3. The blood-pressure cuff itself.
4. The radiology technician who took an X-ray.
5. A bank's fraud-detection algorithm.
6. The specific transaction it just flagged as suspicious.
7. "Call the customer if a charge is flagged, before reversing it."
8. An airline's gate agent.
9. A boarding pass barcode.
10. "Scan every boarding pass before allowing passengers onto the jet bridge."
11. A restaurant's walk-in cooler thermometer log.
12. The decision a head chef makes to 86 a menu item.
13. A school's student attendance record for one class period.
14. The bell schedule a school runs on.
15. A rideshare driver's phone, running the driver app.
16. The GPS coordinates the app sends every few seconds.
17. A warehouse's pallet-racking system (the physical shelving).
18. The inventory count of items on a specific pallet.
19. "When stock on any SKU drops below 10 units, automatically reorder."
20. The vendor who fulfills that automatic reorder.

---

## Problem 3 — Extend the register with a real gap (60 min)

Riverbend's seed has no **Finance** function beyond Nora eyeballing sales summaries — no bookkeeping, no supplier-invoice tracking, no payroll. Design and add it:

1. At least 2 `people` rows, 2 `process` rows, 2 `data` rows, and 1 `technology` row for a realistic Finance function (bookkeeper, payroll process, supplier invoice, payroll register, accounting software — your call on specifics).
2. At least 4 flows connecting your new Finance components to each other **and** to at least one existing component from the original seed (e.g., a flow from POS Transaction into a daily-sales-reconciliation process).
3. Run the dead-end query from Exercise 2 Part C against your updated register and confirm none of your new rows are orphaned.

**Deliver** `finance-register.sql` with all your `INSERT` statements and a short comment above each explaining what it represents.

---

## Problem 4 — A system you use daily (45 min)

Pick one information system you personally interact with at least weekly — a banking app, a school portal, a food-delivery app, a gym's check-in kiosk, anything real. In `daily-system.md` (300–450 words):

1. Name its people, process, data, and technology components as you understand them from the outside (you won't have perfect visibility — say where you're guessing).
2. Walk one piece of its data through the DIKW pyramid (Lecture 3 §1) to a decision either you or the organization actually makes because of it.
3. Identify one flow you *wish* existed but doesn't (something that would make the system genuinely better if it were connected) — using the gap → consequence → impact → cost → value pattern from Lecture 3 §5.

---

## Problem 5 — Build a second business from scratch (90 min)

Before you tackle the mini-project's real business (which is higher-stakes and needs your best work), get a rep in on a **hypothetical** one. Invent a small business — different from Riverbend and different from whatever you're planning for the mini-project. A few ideas if you're stuck: a food truck, a small gym, a dog-walking service, a used-bookstore, a bike-repair shop.

1. Write a 3–4 sentence description of what it does (mirror the level of detail in the week README's Riverbend description).
2. Build a full `is_components`/`is_flows` register: at least 10 components (2+ per category) and 8 flows.
3. Run the dead-end check. Fix anything orphaned.

**Deliver** `practice-business.sql` with the full schema + seed, runnable end to end against a fresh database.

---

## Wrap-up

You now have four working registers by the end of this week: Riverbend (seeded + your extensions), your Problem 3 Finance addition, your Problem 5 practice business, and — coming Saturday — your real mini-project business. That repetition is the actual curriculum; the vocabulary sticks through use, not through re-reading the lectures.
