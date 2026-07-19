# Week 6 — Enterprise Systems (ERP/CRM)

> **Goal:** by Sunday you can explain, in plain language a CFO and a warehouse manager would both nod along to, what an ERP system and a CRM system actually *do*, why "master data" is the single most expensive concept in enterprise software to get wrong, and how one business event — a customer buying a bike — ripples across sales, inventory, procurement, and finance as a single traceable process. You'll model the entities that make this possible in PostgreSQL and prove the trace with real queries.

Welcome back to **C37 · Crunch Information Systems**. Week 3 you built a normalized schema. Week 4 you learned to query it. Week 5 you modeled and automated a manual process end to end. This week those threads meet the world they were built for: **the enterprise system**. Every mid-size-or-larger organization runs its sales, inventory, purchasing, HR, and accounting through a small number of large, integrated platforms — an ERP (Enterprise Resource Planning) system on the operational/financial side, a CRM (Customer Relationship Management) system on the sales/relationship side. Neither is magic. Both are, underneath the vendor branding, a set of tables holding master data (customers, products, suppliers, employees — the *nouns* of the business) and transactional data (orders, purchase orders, invoices, opportunities — the *verbs*), wired together by foreign keys and business processes. Get the master data wrong — two customer records for the same company, a product price that disagrees between the warehouse system and the invoice system — and every report built on top of it quietly lies. This week you learn to see through the acronyms to the data model underneath, and you extend **Crunch Cycles** (the bike company from Weeks 3–4) with a supplier side and a sales pipeline so it looks, for the first time, like a real small enterprise.

## Learning objectives

By the end of this week, you will be able to:

- **Explain** the purpose and major modules of ERP and CRM systems, and articulate *why* integrating them is the hard, expensive part of enterprise software — harder than any single module.
- **Define** master data and distinguish it from transactional data, and explain why a single source of truth for master data matters across an entire business.
- **Trace** an end-to-end enterprise process — order-to-cash and procure-to-pay — across the module/system boundaries it actually crosses in a real company.
- **Model** core enterprise entities and master-data relationships in SQL: customers, products, suppliers, and the transactions that connect them.
- **Weigh** build vs. buy for enterprise software, and price out the real, often-hidden cost of customizing a bought system.

## Prerequisites

- Weeks 1–4 of this course (information systems foundations, requirements, data modeling, SQL/pandas querying). Week 5 (process modeling) is helpful context but not required — this week re-introduces the process-tracing skill from scratch.
- PostgreSQL 16+ running locally (SQLite fallback noted where it matters). Install steps: [`resources.md`](./resources.md).
- The Week 4 **Crunch Cycles** schema loaded (`regions`, `employees`, `customers`, `products`, `orders`, `order_items`). If you don't have it, the full seed is in [Week 4's README](../week-04-querying-data-with-sql-and-python/README.md#setup-load-the-week-3-schema) — load that first, then come back here.
- Comfort with `JOIN`, `GROUP BY`, and multi-table `SELECT` from Week 4. This week adds tables; it doesn't add new SQL syntax you haven't seen.

## Setup: extend Crunch Cycles into a small enterprise

Crunch Cycles as built in Week 4 only has a **sales** side: customers place orders, orders contain products. A real enterprise also has a **procurement** side (where the products *came from*) and a **CRM** side (the pipeline of deals *before* they become orders). This week you add both — three new master/transactional structures layered onto the schema you already have.

Run this against your existing `crunchcycles` database (or `crunchcycles.db` on SQLite):

```sql
-- ── Suppliers: new master-data entity ──────────────────────────────────────
CREATE TABLE suppliers (
    supplier_id     INTEGER PRIMARY KEY,
    supplier_name   TEXT NOT NULL,
    contact_name    TEXT NOT NULL,
    email           TEXT NOT NULL,
    country         TEXT NOT NULL,
    lead_time_days  INTEGER NOT NULL,          -- typical days from PO to delivery
    is_active       BOOLEAN NOT NULL DEFAULT TRUE
);

-- ── Which supplier makes which product, at what cost (many-to-many) ────────
CREATE TABLE product_suppliers (
    product_id      INTEGER NOT NULL,
    supplier_id     INTEGER NOT NULL,
    supplier_sku    TEXT NOT NULL,              -- the supplier's own part number
    unit_cost       NUMERIC NOT NULL,
    is_preferred    BOOLEAN NOT NULL DEFAULT FALSE,
    PRIMARY KEY (product_id, supplier_id),
    FOREIGN KEY (product_id)  REFERENCES products(product_id),
    FOREIGN KEY (supplier_id) REFERENCES suppliers(supplier_id)
);

-- ── Purchase orders: the procure-to-pay transactional spine ────────────────
CREATE TABLE purchase_orders (
    po_id           INTEGER PRIMARY KEY,
    supplier_id     INTEGER NOT NULL,
    employee_id     INTEGER NOT NULL,           -- buyer who approved it
    po_date         DATE NOT NULL,
    expected_date   DATE NOT NULL,
    received_date   DATE,                       -- NULL = not yet received
    status          TEXT NOT NULL,               -- 'Open' | 'Received' | 'Cancelled'
    FOREIGN KEY (supplier_id) REFERENCES suppliers(supplier_id),
    FOREIGN KEY (employee_id) REFERENCES employees(emp_id)
);

CREATE TABLE purchase_order_items (
    po_id           INTEGER NOT NULL,
    product_id      INTEGER NOT NULL,
    quantity        INTEGER NOT NULL,
    unit_cost       NUMERIC NOT NULL,
    PRIMARY KEY (po_id, product_id),
    FOREIGN KEY (po_id)      REFERENCES purchase_orders(po_id),
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);

-- ── CRM opportunities: the pipeline that precedes a sales order ────────────
CREATE TABLE crm_opportunities (
    opportunity_id  INTEGER PRIMARY KEY,
    customer_id     INTEGER NOT NULL,
    employee_id     INTEGER NOT NULL,           -- owning sales rep
    stage           TEXT NOT NULL,               -- 'Prospecting'|'Qualified'|'Proposal'|'Won'|'Lost'
    est_value       NUMERIC NOT NULL,
    opened_date     DATE NOT NULL,
    closed_date     DATE,                        -- NULL = still open
    won_order_id    INTEGER,                     -- FK to orders once won; NULL until then
    FOREIGN KEY (customer_id)   REFERENCES customers(customer_id),
    FOREIGN KEY (employee_id)   REFERENCES employees(emp_id),
    FOREIGN KEY (won_order_id)  REFERENCES orders(order_id)
);

INSERT INTO suppliers VALUES
(1,'Taiwan Cycle Works','Amy Lin','amy@taiwancycleworks.tw','Taiwan',30,TRUE),
(2,'Rhine Frame GmbH','Bernd Koch','bernd@rhineframe.de','Germany',21,TRUE),
(3,'Pacific Alloy Co','Trang Nguyen','trang@pacificalloy.vn','Vietnam',35,TRUE),
(4,'Iron Gate Components','Sam Rios','sam@irongatecomp.com','USA',14,TRUE),
(5,'Nordic Rubber Co','Ingrid Solvang','ingrid@nordicrubber.se','Sweden',18,TRUE);

INSERT INTO product_suppliers VALUES
(1,1,'TCW-MTB100',540.00,TRUE),
(1,3,'PAC-MTB100',555.00,FALSE),          -- backup supplier, slightly pricier
(2,1,'TCW-MTB250',890.00,TRUE),
(3,2,'RFG-ROAD-PRO',1120.00,TRUE),
(4,2,'RFG-ROAD-ELITE',1680.00,TRUE),
(5,4,'IGC-HYBRID-C',390.00,TRUE),
(6,4,'IGC-FOLD-CITY',470.00,TRUE),
(7,3,'PAC-KIDS20',165.00,TRUE),
(8,5,'NRC-HELM-PRO',38.00,TRUE),
(8,4,'IGC-HELM-PRO',41.00,FALSE),          -- backup supplier
(9,5,'NRC-HELM-LITE',19.00,TRUE),
(10,4,'IGC-PANNIER',58.00,TRUE),
(11,5,'NRC-REPAIR',14.00,TRUE),
(12,1,'TCW-MTB50',330.00,TRUE);

INSERT INTO purchase_orders VALUES
(1,1,1,'2024-01-03','2024-02-02','2024-01-30','Received'),
(2,2,1,'2024-01-10','2024-01-31','2024-02-05','Received'),   -- arrived late
(3,4,1,'2024-02-01','2024-02-15','2024-02-14','Received'),
(4,5,1,'2024-02-10','2024-02-28',NULL,'Open'),
(5,3,1,'2024-03-01','2024-04-05',NULL,'Open'),
(6,1,1,'2024-03-15','2024-04-14','2024-04-10','Received'),
(7,2,1,'2024-04-01','2024-04-22',NULL,'Cancelled');           -- cancelled, never received

INSERT INTO purchase_order_items VALUES
(1,1,50,540.00),(1,2,20,890.00),
(2,3,15,1120.00),(2,4,10,1680.00),
(3,5,40,390.00),(3,10,60,58.00),
(4,8,100,38.00),(4,9,100,19.00),(4,11,150,14.00),
(5,7,80,165.00),
(6,1,60,540.00),(6,2,25,890.00),
(7,3,10,1120.00);

INSERT INTO crm_opportunities VALUES
(1,17,2,'Won',1698.00,'2024-02-20','2024-03-10',24),
(2,18,5,'Proposal',2198.00,'2024-05-01',NULL,NULL),           -- Fjord: master record exists, zero orders, live pipeline
(3,12,7,'Qualified',1499.00,'2024-04-10',NULL,NULL),          -- Priya (emp 7): pipeline activity, zero closed orders
(4,11,6,'Won',1899.00,'2024-01-20','2024-02-10',14),
(5,9,6,'Lost',2799.00,'2024-04-01','2024-04-20',NULL),
(6,6,5,'Won',799.00,'2024-01-05','2024-01-20',6),
(7,2,2,'Prospecting',3000.00,'2024-06-01',NULL,NULL);
```

Sanity check — these should print `5`, `14`, `7`, `13`, `7`:

```sql
SELECT COUNT(*) FROM suppliers;
SELECT COUNT(*) FROM product_suppliers;
SELECT COUNT(*) FROM purchase_orders;
SELECT COUNT(*) FROM purchase_order_items;
SELECT COUNT(*) FROM crm_opportunities;
```

Notice what's deliberately in there: PO 7 is **cancelled** — a purchase order that never delivers a thing, exactly like the cancelled sales orders from Week 4. Product 1 and product 8 each have **two suppliers**, one preferred and one backup — a real many-to-many master-data relationship. Opportunity 2 (Fjord Cycling Supply) is a live deal for a customer who, per Week 4, has **never placed an order** — the pipeline and the ledger are two different truths about the same customer. Opportunity 3 belongs to Priya (employee 7), who per Week 4 has **zero completed orders** — she's working a deal that hasn't closed yet. These aren't bugs; they're exactly the gap between CRM (what we're pursuing) and ERP (what actually happened) that this week is about.

## Weekly schedule

Adds up to the course's full-time target of **~28 hours**.

| Day | Focus | Lectures | Exercises | Challenges | Quiz/Read | Homework | Mini-Project | Daily Total |
|-----------|--------------------------------------------|---------:|----------:|-----------:|----------:|---------:|-------------:|------------:|
| Monday | ERP & CRM modules, why integration is hard | 2h | 1.5h | 0h | 0.5h | 1h | 0h | 5h |
| Tuesday | Master data vs. transactional data | 2h | 1.5h | 0h | 0.5h | 1h | 0h | 5h |
| Wednesday | Order-to-cash and procure-to-pay traced | 2h | 1.5h | 1h | 0.5h | 1h | 0h | 6h |
| Thursday | Build vs. buy; TCO of customization | 0h | 1h | 1.5h | 0.5h | 1h | 1h | 5h |
| Friday | Reconciling duplicate master data | 0h | 0.5h | 1.5h | 0.5h | 1h | 1.5h | 5h |
| Saturday | Mini-project (model + trace a process) | 0h | 0h | 0h | 0h | 0h | 3h | 3h |
| Sunday | Quiz + review | 0h | 0h | 0h | 1h | 0h | 0h | 1h |
| **Total** | | **6h** | **6h** | **4h** | **3.5h** | **5h** | **5.5h** | **~30h** |

## How to navigate this week

Work top to bottom. Each piece assumes the ones above it.

| # | File | What's inside | ~Time |
|--:|------|---------------|------:|
| 1 | [lecture-notes/01-erp-and-crm-explained.md](./lecture-notes/01-erp-and-crm-explained.md) | What ERP and CRM systems do, their modules, and why integrating them is the hard part | 2h |
| 2 | [lecture-notes/02-master-data-and-single-source-of-truth.md](./lecture-notes/02-master-data-and-single-source-of-truth.md) | Master vs. transactional data, data ownership, cost of duplication | 2h |
| 3 | [lecture-notes/03-end-to-end-enterprise-processes.md](./lecture-notes/03-end-to-end-enterprise-processes.md) | Order-to-cash and procure-to-pay traced across module boundaries | 2h |
| 4 | [exercises/exercise-01-map-erp-modules.md](./exercises/exercise-01-map-erp-modules.md) | Map ERP modules to a company's departments | 1.5h |
| 5 | [exercises/exercise-02-identify-master-data.md](./exercises/exercise-02-identify-master-data.md) | Classify tables and columns as master vs. transactional | 1.5h |
| 6 | [exercises/exercise-03-trace-order-to-cash.md](./exercises/exercise-03-trace-order-to-cash.md) | Trace order-to-cash across the schema with SQL | 1.5h |
| 7 | [challenges/challenge-01-build-vs-buy.md](./challenges/challenge-01-build-vs-buy.md) | Argue build vs. buy for a mid-size firm, with a priced-out TCO model | 2h |
| 8 | [challenges/challenge-02-reconcile-duplicate-masters.md](./challenges/challenge-02-reconcile-duplicate-masters.md) | Design and script a fix for duplicated customer master data | 2h |
| 9 | [mini-project/README.md](./mini-project/README.md) | Model core enterprise entities + document order-to-cash end to end | 3h |
| 10 | [homework.md](./homework.md) | Extra practice, spaced across the week | 5h |
| 11 | [quiz.md](./quiz.md) | 15 self-check questions + answer key | 1h |
| 12 | [resources.md](./resources.md) | Official docs, open-source ERPs to explore, MDM reading | — |

## By the end of this week you can…

- Point at any table in a business's database and say, correctly, whether it's master data or transactional data — and explain why the distinction changes how you'd design, own, and govern it.
- Name the ERP modules (sales, inventory, procurement, finance/GL, HR) and CRM's role alongside them, and map each to the department that owns it.
- Trace order-to-cash and procure-to-pay as concrete SQL joins across real tables, not just boxes on a diagram.
- Read a "build vs. buy" enterprise-software decision and evaluate it on total cost of ownership, not sticker price.
- Spot duplicated or conflicting master data and design a reconciliation strategy that doesn't just delete the "obviously wrong" row and hope.

## Up next

[Week 7 — Integration & APIs](../week-07-integration-and-apis/) — now that you can see the tables underneath ERP and CRM, next week you connect systems that don't share a database at all: REST APIs, webhooks, and ETL/ELT jobs.

---

*Part of the Code Crunch Worldwide open curriculum · GPL-3.0 · If you find errors, please open an issue or PR.*
