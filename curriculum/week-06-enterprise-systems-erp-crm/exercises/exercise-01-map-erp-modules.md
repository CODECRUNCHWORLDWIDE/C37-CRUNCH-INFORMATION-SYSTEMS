# Exercise 1 — Map ERP Modules to a Company's Departments

**Goal:** Get comfortable connecting the abstract module list from Lecture 1 to a real organization's actual departments and the day-to-day questions each one asks. This is the skill you'll use the moment you walk into any company and try to figure out "what system handles what" here.

**Estimated time:** 45–60 minutes.

## Setup

No database needed for this one — grab a text file and Lecture 1's module table.

## Part A — Map Crunch Cycles

Crunch Cycles (this course's running example) has grown to about 80 employees, organized into these departments: **Sales**, **Warehouse/Fulfillment**, **Purchasing**, **Accounting**, **HR**, **Customer Support**.

For each department below, write:
1. Which ERP module (or CRM) it primarily lives in.
2. One question from Lecture 1's module table (or a new one you invent) that department would ask that system daily.
3. Which table(s) *from this week's schema* that module would touch.

| Department | Module | Daily question | Tables it touches |
|---|---|---|---|
| Sales | ? | ? | ? |
| Warehouse/Fulfillment | ? | ? | ? |
| Purchasing | ? | ? | ? |
| Accounting | ? | ? | ? |
| HR | ? | ? | ? |
| Customer Support | ? | ? | ? |

Fill in every cell. For **Accounting** and **HR**, be honest that this week's schema doesn't have dedicated tables for their full scope (no `invoices`, no `payroll`) — say what's missing, in one sentence each.

## Part B — A messier real company

Now do the same exercise for a company that doesn't map as cleanly: **a software-as-a-service (SaaS) company** that sells subscriptions, not physical products. It has: **Sales**, **Customer Success**, **Engineering**, **Finance**, **HR**.

1. Does this company need a **Procurement** module in the same way Crunch Cycles does? Why or why not — think about what a SaaS company actually buys physically vs. what it "buys" (cloud infrastructure, software licenses).
2. Does it need an **Inventory** module at all? If not, what module (if any) replaces the concept of "what do we have to sell/deliver"?
3. Where does **Customer Success** fit — is it CRM, a support-ticket system, or something ERP doesn't traditionally cover at all? Argue your answer in 2-3 sentences.

## Part C — Reflection (150 words)

In your own words: why do off-the-shelf ERP systems ship with dozens of modules most companies never turn on? What's the trade-off between buying a system with modules you don't need vs. building only what you need yourself? (You'll argue this properly in [Challenge 1](../challenges/challenge-01-build-vs-buy.md) — this is just the warm-up thought.)

## Done when…

- [ ] Every cell in Part A's table is filled, including the two "what's missing" sentences.
- [ ] Part B answers all three questions with a stated reason, not just a yes/no.
- [ ] Part C reflection is written and is genuinely your own reasoning, not a restatement of the lecture.

## Stretch

Pick one real, named ERP or CRM product you can find public documentation for (SAP S/4HANA, Oracle NetSuite, Microsoft Dynamics 365, Odoo, Salesforce, HubSpot — all have public feature/module pages). Find their module list and compare it to Lecture 1's table. What did they call something differently? What module did they have that Lecture 1 didn't mention?

## Submission

Commit `exercise-01.md` (or your equivalent) to your portfolio under `c37-week-06/exercise-01/`.
