# Lecture 1 — ERP & CRM Explained

Every organization past a certain size — usually somewhere around 20-50 employees, sometimes sooner — hits the same wall: the spreadsheets and small standalone tools that got them started stop being able to answer basic questions. "How much inventory do we actually have?" takes three phone calls. "What did we sell to this customer last year?" depends on whose laptop you ask. "Are we paying this supplier twice?" nobody knows. The answer the software industry converged on, starting in the 1990s and still true today, is two large, integrated systems: an **ERP** and a **CRM**. This lecture is about what they actually are underneath the marketing — because once you see them as databases with workflows on top, the rest of this week (and a fair amount of real IT-department work) stops being mysterious.

## 1. ERP: Enterprise Resource Planning

**ERP** is a single integrated system (or tightly-connected set of systems) that runs the *operational and financial backbone* of a company: what do we have, what did we spend, what do we owe, what are we owed. The word "resource" is doing real work in the name — an ERP tracks resources: inventory, cash, people-hours, equipment, raw materials.

An ERP is not one thing; it's a set of **modules**, each covering one operational area, sharing one underlying database (or a tightly integrated set of them):

| Module | What it tracks | Typical questions it answers |
|---|---|---|
| **Sales / Order Management** | Sales orders, pricing, fulfillment status | "What did customer X order, and has it shipped?" |
| **Inventory / Warehouse** | Stock levels, locations, movements | "How many units of product Y do we have, and where?" |
| **Procurement / Purchasing** | Purchase orders, supplier terms, receiving | "What have we ordered from supplier Z, and when does it arrive?" |
| **Manufacturing / Production** | Bills of materials, work orders, capacity | "Can we build 500 more units by Friday?" |
| **Finance / General Ledger (GL)** | Every transaction as a debit/credit, accounts | "What's our cash position? Are we profitable this quarter?" |
| **Human Resources / Payroll** | Employees, compensation, benefits, time-off | "What's our total payroll cost by department?" |

You've already built the seed of two of these. Crunch Cycles' `orders`/`order_items` from Week 4 *is* a slice of a Sales module. The `suppliers`/`purchase_orders`/`purchase_order_items` you loaded this week *is* a slice of a Procurement module. Real ERP vendors (SAP, Oracle NetSuite, Microsoft Dynamics, Odoo, ERPNext) sell software that does the same thing you just did — tables, foreign keys, business rules — at a scale and polish that took decades and thousands of engineers to build.

**Why one system, not six separate ones?** Because the modules constantly need each other's data *right now*. Sales needs to know current inventory before promising a ship date. Procurement needs to know sales forecasts before ordering more stock. Finance needs every sale and every purchase to post correctly to the ledger, automatically, the moment they happen — not at month-end when someone remembers to type them in. An ERP's core value proposition is: **one shared, live, transactionally consistent database, instead of six departments emailing each other spreadsheets.**

## 2. CRM: Customer Relationship Management

**CRM** is the system that runs everything *before* and *around* a sale: who are our prospects, what deals are in progress, what's the history of every conversation with a customer, what's our support ticket backlog. Where ERP asks "what happened," CRM asks "what's the relationship, and what might happen next."

CRM's core entities:

| Entity | What it represents | Example |
|---|---|---|
| **Lead** | An unqualified potential customer | "Someone downloaded our catalog and gave an email" |
| **Contact / Account** | A qualified person / company we're tracking | "Fjord Cycling Supply, contact: Oda Berg" |
| **Opportunity / Deal** | A specific potential sale, with a stage and a value | "Fjord — 40 units, Proposal stage, $2,198 estimated" |
| **Activity** | A logged interaction | "Called Oda on May 3rd, sent follow-up proposal" |
| **Case / Ticket** | A support or service issue | "Fjord reported a defective helmet buckle" |

Salesforce, HubSpot, and Microsoft Dynamics 365 (which, notably, also sells ERP — the line blurs at the top of the market) are the household names here. The `crm_opportunities` table you loaded this week is a deliberately minimal version of what a real CRM's `Opportunity` object looks like: a customer, an owning rep, a stage, an estimated value, and — critically — a nullable link to the sales order it becomes *if and when it's won*.

## 3. Where ERP ends and CRM begins

The boundary is genuinely blurry in vendor marketing, but there's a clean way to think about it: **CRM owns the sales *process*; ERP owns the sales *record*.**

- Before a deal closes, it lives in CRM as an `Opportunity` with a `stage` — Prospecting, Qualified, Proposal, Won, Lost. It's a forecast, a piece of *intent*.
- The moment a deal is won, it becomes a fact: a sales order, with real product lines, a real ship date, a real invoice. That fact belongs in ERP.

Query the data you loaded this week and you'll see the boundary directly:

```sql
-- Opportunities that never became an order at all (Lost, or still open)
SELECT opportunity_id, customer_id, stage, est_value
FROM crm_opportunities
WHERE won_order_id IS NULL;
```

```sql
-- Opportunities that DID become an order — CRM intent, matched to ERP fact
SELECT o.opportunity_id, o.stage, o.est_value, ord.order_id, ord.order_date
FROM crm_opportunities o
JOIN orders ord ON ord.order_id = o.won_order_id;
```

Run the second query and you'll notice `est_value` doesn't always equal the order's actual total (compare opportunity 1's `1698.00` against what order 24's `order_items` actually sum to) — a forecast is an estimate, made before the real product mix and quantities were finalized. That gap between CRM's *forecast* and ERP's *actual* is exactly why companies watch both numbers, and why sales forecasting is notoriously imprecise even with perfect software.

## 4. Why integration is the hard part

Here's the sentence every enterprise-software veteran will tell you, in one form or another: **buying the modules is easy; making them agree with each other is the entire job.**

Concretely, integration is hard for reasons that show up over and over:

1. **Different systems, different "customer."** The CRM's `Account` for "Fjord Cycling Supply" and the ERP's `customer` record for the same company were very likely created independently, by different people, at different times, possibly with a typo in one and not the other. Are they the same entity? A computer can't tell without a deliberate matching key (this is exactly next lecture's topic: master data).
2. **Different systems, different timing.** CRM says a deal is "Won" the instant a salesperson clicks a button. ERP doesn't know the order exists until someone (or some integration job) actually creates the sales order. There's a window — seconds, sometimes days — where the two systems disagree about reality.
3. **Different systems, different owners.** The sales team configures and trusts the CRM. The finance/ops team configures and trusts the ERP. Neither wants the other team's system to be the "real" source of a fact that affects their own reporting. This is an organizational problem as much as a technical one, and it's often the harder half to solve.
4. **The integration itself is more software to build and maintain.** Whether it's a nightly batch job, a real-time webhook, or a vendor-built connector (Salesforce ↔ NetSuite is a whole cottage industry), someone has to write it, test it, and fix it when either side changes its schema. You'll build exactly this kind of integration in Week 7.

## 5. A mental model to keep

Strip away the vendor names and dashboards, and every ERP or CRM is:

- A **relational database** (yes — the same kind you've been building all course) holding master data and transactional data.
- A set of **business rules and workflows** enforced in application code on top of that database (you can't ship an order with no inventory; you can't mark an opportunity "Won" without a customer).
- A **user interface** so non-technical staff can read and write that database without writing SQL.
- Increasingly, an **API** so other systems (including each other) can read and write it too.

You already know how to build the first layer — you did it this week and in Week 3. The rest of this course (automation in Week 5, integration in Week 7, security in Week 9) is about the layers on top. When a $40,000/seat enterprise software quote crosses your desk one day, you'll be able to ask the question that actually matters: *what tables, what rules, and what does the integration to our other systems really cost?* — instead of just trusting the sales deck.

## 6. Check yourself

Before moving to Lecture 2, make sure you can answer these without looking back:

- Name three ERP modules and one business question each answers.
- What is the practical difference between a CRM `Opportunity` and an ERP `Order`?
- Why is "we bought the same vendor's ERP and CRM" not a guarantee that integration is easy?
- In your own words, why is one shared database more valuable to a company than six departments each running their own tool?

## Further reading

Continue to [Lecture 2 — Master Data & the Single Source of Truth](./02-master-data-and-single-source-of-truth.md), where you'll dig into exactly what makes a "customer" record trustworthy — or not — across every system that touches it.
