# Mini-Project — Model a Small Enterprise & Trace Order-to-Cash

> Model the core entities and master-data relationships of a small enterprise — customers, products, orders, suppliers — in PostgreSQL, and document how one end-to-end process (order-to-cash) flows across them. This is the week's capstone: proof that you can see through the ERP/CRM acronyms to the actual data model and process underneath.

**Estimated time:** 3 hours, best done Saturday after the exercises and challenges.

You've spent this week extending Crunch Cycles with suppliers, purchase orders, and a CRM pipeline. This mini-project asks you to do the harder, more valuable version of that skill: model a **new** small enterprise from a plain-English brief, on your own, and then prove your model works by tracing a real process through it with SQL — exactly what a junior data/systems analyst is asked to do in their first few months on a real team.

---

## The brief

**Northstar Outfitters** is a small outdoor-gear retailer (think: tents, backpacks, boots) that sells both to individual consumers online and to specialty outdoor shops wholesale. They've hired you to design the data model for their new internal system, replacing a tangle of spreadsheets. Here's what you know from talking to their team:

- They have **customers** — some are individual consumers, some are wholesale retail shops. Every customer has a name, contact info, and a `customer_type` ('Consumer' or 'Wholesale'). Wholesale customers get a negotiated `discount_pct`; consumers don't (their discount is always 0).
- They have a **product catalog** — each product has a name, a category (Tents, Backpacks, Footwear, Apparel, Accessories), a retail price, and a current on-hand quantity they track loosely today (and want tracked properly going forward).
- They buy finished goods from **suppliers** — some suppliers make one product line, some make several. Each supplier has a name, country, and a typical lead time.
- Customers place **orders**, each with one or more line items (product + quantity + price actually charged — which may differ from the current catalog price if it was a promotion).
- Northstar issues **purchase orders** to suppliers to restock inventory — each with one or more line items (product + quantity + unit cost).
- They also want to track their **sales pipeline** for wholesale deals specifically — a wholesale prospect goes through stages before becoming a recurring wholesale customer with actual orders.

## Deliverable

A directory in your portfolio `c37-week-06/mini-project/` containing:

1. `schema.sql` — every `CREATE TABLE` statement, with primary keys, foreign keys, and at least three meaningful `CHECK`/`NOT NULL` constraints (e.g., `customer_type IN ('Consumer','Wholesale')`, `discount_pct >= 0 AND discount_pct <= 1`).
2. `seed.sql` — realistic seed data: **at least 8 customers** (mix of Consumer and Wholesale), **at least 10 products** across at least 3 categories, **at least 4 suppliers**, **at least 15 orders** with order items, **at least 5 purchase orders** with items, and **at least 5 pipeline entries** for wholesale prospects (some won, some lost, some still open).
3. `erd.md` — a text-based entity-relationship description: every entity, its master/transactional classification (per Lecture 2's test), and every relationship with its cardinality (one-to-many, many-to-many). ASCII or Markdown is fine — no drawing tool required, though you may link an image if you made one.
4. `trace-order-to-cash.sql` — a set of queries that trace order-to-cash for Northstar specifically, mirroring [Exercise 3](../exercises/exercise-03-trace-order-to-cash.md)'s pattern: pipeline → won deal → order → fulfillment.
5. `report.md` — see "Report requirements" below.

---

## Requirements

### Schema requirements

- Every table must be classified in `erd.md` as master or transactional, using Lecture 2's test — and you must be able to defend the classification for every single one, including any edge cases (this brief has at least one, the pipeline table — think about why, same as `crm_opportunities` this week).
- At least one **many-to-many** relationship must exist and be modeled with a proper junction table (hint: which product can come from which supplier — same pattern as this week's `product_suppliers`).
- Every foreign key must actually be enforced (`FOREIGN KEY ... REFERENCES ...`), not just implied by column naming.
- Wholesale-specific business rules must be enforced at the database level where possible: a `CHECK` constraint that a Consumer customer's `discount_pct` really is 0, not just documentation saying it should be.

### Seed data requirements

- Include at least **one wholesale customer with zero orders but an open pipeline entry** — deliberately mirroring this week's Fjord Cycling Supply / opportunity 2 pattern, because that gap (pipeline interest, no transaction yet) is a real and common state, not an edge case to avoid.
- Include at least **one cancelled or lost pipeline entry** and **one cancelled purchase order** — process breaks are normal, and your model has to represent them, not just the happy path.
- Include at least **one product sourced from two different suppliers** (a real many-to-many row, not just a lonely junction-table entry).

### Trace requirements

Your `trace-order-to-cash.sql` must include, at minimum:

1. Every wholesale pipeline entry that became a real order (forecast vs. actual value, same pattern as this week's Exercise 3 Task 4).
2. Every wholesale customer with open pipeline activity but zero completed orders.
3. Total revenue this quarter, split by `customer_type` (Consumer vs. Wholesale) — a query a Northstar exec would actually ask for.
4. One query of your own invention that answers a business question the brief implies but doesn't explicitly ask — state the question as a SQL comment above the query.

---

## Milestones

- **Milestone 1 (45 min):** Design and write `schema.sql`. Get every `CREATE TABLE` to actually run without error before writing a single row of seed data.
- **Milestone 2 (45 min):** Write and load `seed.sql`, hitting every minimum-row-count requirement above. Run sanity-check `COUNT(*)`s on every table.
- **Milestone 3 (30 min):** Write `erd.md` — classify every table, document every relationship and cardinality.
- **Milestone 4 (45 min):** Write and run `trace-order-to-cash.sql`, all four required queries plus your own.
- **Milestone 5 (15 min):** Write `report.md`.

---

## Report requirements (`report.md`, ~350–500 words)

1. **Model summary.** One paragraph: what entities you modeled, and the one design decision you're most confident about.
2. **The hardest call.** One paragraph: which table or relationship was genuinely ambiguous to design (master vs. transactional, or how to model the many-to-many), and how you resolved it.
3. **What order-to-cash revealed.** One paragraph: after running your trace queries, what did you learn about Northstar's business that wasn't obvious from the brief alone? (e.g., "wholesale pipeline conversion looks weak," "one supplier is a single point of failure for three products.")
4. **What you'd add next.** One paragraph: if Northstar asked you to extend this next month, what table or process would you add first (invoices? inventory movements? a returns process?) and why that one before the others.

---

## Rubric

| Criterion | Weight | "Great" looks like |
|-----------|------:|--------------------|
| Schema correctness | 25% | All tables, keys, constraints run cleanly; many-to-many modeled correctly |
| Seed data realism | 20% | Meets every minimum; includes the deliberate edge cases (zero-order pipeline customer, cancelled PO, dual-sourced product) |
| ERD & classification | 20% | Every entity classified with a defensible reason; every relationship's cardinality stated correctly |
| Trace queries | 20% | All 4 required queries correct and return sensible results; the invented query answers a real, stated business question |
| Report | 15% | All four paragraphs present, specific to *your* model — not generic restatement of the lectures |

---

## Why this matters

This is the actual first project a junior analyst or systems person gets handed at a real company: "here's roughly how the business works, go build the data model and prove it answers our questions." Nobody hands you a finished ERD. You extract the entities from a messy verbal brief (exactly like this one), you decide the edge cases, and you prove the model works by running the queries a real stakeholder would ask for. Keep this project — Week 7 (Integration & APIs) will have you connect a second, independent system to it, which only works cleanly if this week's model has a genuine single source of truth for each entity.

When done: push, then take the [quiz](../quiz.md) and continue to [Week 7 — Integration & APIs](../../week-07-integration-and-apis/).
