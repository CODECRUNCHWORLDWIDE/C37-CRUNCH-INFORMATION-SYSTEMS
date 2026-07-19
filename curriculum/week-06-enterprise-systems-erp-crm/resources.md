# Week 6 — Resources

Free, public, no signup unless noted. Read the "required" set; treat the rest as reference you dip into when a specific question comes up.

## Install first (if not already set up)

- **PostgreSQL 16+** — this week's primary engine, same as every prior week: <https://www.postgresql.org/download/> · macOS: [Postgres.app](https://postgresapp.com/). Linux: `sudo apt install postgresql` / `sudo dnf install postgresql-server`. Windows: the EDB installer.
- **SQLite 3.35+** — the zero-setup fallback: <https://www.sqlite.org/download.html>. Check with `sqlite3 --version`.
- **Python 3.10+ with pandas** — needed for Challenge 1's TCO model:
  ```bash
  python3 -m venv .venv && source .venv/bin/activate
  pip install pandas
  ```

## Explore a real open-source ERP (free, no purchase needed)

You don't need to buy $125,000 software to see what a real ERP's module structure looks like — these are free and open-source, and browsing their docs/demo makes Lecture 1's module table concrete:

- **Odoo (Community Edition)** — arguably the most widely deployed open-source ERP; browse its module list: <https://www.odoo.com/page/erp-apps> · docs: <https://www.odoo.com/documentation>
  *Why: see a real, current module catalog — Sales, Inventory, Purchase, Accounting, CRM — mapped to actual screens.*
- **ERPNext** — another full open-source ERP, MIT-licensed, with a public demo: <https://frappe.io/erpnext> · docs: <https://docs.frappe.io/erpnext>
  *Why: a second, differently-organized module structure to compare against Odoo's — module boundaries aren't universal, they're a design choice.*
- **SuiteCRM** — open-source CRM (a fork of the SugarCRM codebase), if you want to see a CRM's object model (Leads, Opportunities, Accounts) up close: <https://suitecrm.com/> · docs: <https://docs.suitecrm.com/>
  *Why: compare its `Opportunity` object fields to this week's minimal `crm_opportunities` table — see what a mature CRM adds.*

## Required reading (this week's core)

- **Salesforce, "What Is CRM?"** — vendor-authored but a clear, accurate plain-language explainer of CRM's core objects: <https://www.salesforce.com/crm/what-is-crm/>
  *Why: the Lead → Opportunity → Account/Contact model explained by the company that popularized modern CRM.*
- **Oracle NetSuite, "What Is ERP?"** — a vendor-neutral-enough overview of ERP modules and history: <https://www.netsuite.com/portal/resource/articles/erp/what-is-erp.shtml>
  *Why: covers the module breakdown from Lecture 1 with more history and context than we had room for.*
- **Gartner, "Master Data Management (MDM)" glossary entry:** <https://www.gartner.com/en/information-technology/glossary/master-data-management-mdm>
  *Why: the industry-standard short definition of MDM — useful vocabulary for Lecture 2 and Challenge 2.*
- **Investopedia, "Order to Cash (O2C)":** <https://www.investopedia.com/terms/o/order-to-cash.asp>
  *Why: a finance-audience explanation of O2C that complements Lecture 3's systems-audience one.*
- **Investopedia, "Procure-to-Pay (P2P)":** <https://www.investopedia.com/terms/p/procure-to-pay.asp>
  *Why: same, for P2P — including the "three-way match" control from Lecture 3.*

## Reference (keep in tabs)

- **PostgreSQL — "Constraints":** <https://www.postgresql.org/docs/current/ddl-constraints.html>
  *Why: `CHECK`, `NOT NULL`, `FOREIGN KEY` — you'll lean on all three in the mini-project's schema requirements.*
- **PostgreSQL — "Transactions":** <https://www.postgresql.org/docs/current/tutorial-transactions.html>
  *Why: `BEGIN`/`COMMIT`/`ROLLBACK` — required for Challenge 2's safe master-data reassignment.*
- **pandas — "Intro to data structures":** <https://pandas.pydata.org/docs/user_guide/dsintro.html>
  *Why: DataFrame basics for Challenge 1's TCO model, if pandas from Week 4 needs a refresher.*
- **pandas — `cumsum` reference:** <https://pandas.pydata.org/docs/reference/api/pandas.Series.cumsum.html>
  *Why: exactly the function you need for the cumulative-cost columns in Challenge 1.*

## Deeper background (optional this week)

- **Wikipedia, "Enterprise resource planning" — History section:** <https://en.wikipedia.org/wiki/Enterprise_resource_planning>
  *Why: how ERP evolved from 1960s MRP (Material Requirements Planning) through today — useful context for why the module list looks the way it does.*
- **"The Data Warehouse Toolkit" (Kimball & Ross)** — not free, but the standard reference on modeling master vs. transactional (fact/dimension) data at scale; your library likely has it.
  *Why: the concepts in Lecture 2 (master vs. transactional) are the direct ancestor of "dimension vs. fact" tables, which you'll meet properly in Week 10 (Analytics & BI).*
- **MuleSoft, "What Is Enterprise Application Integration?"** — vendor-authored, but a clear primer on the integration-is-hard theme from Lecture 1: <https://www.mulesoft.com/resources/esb/what-enterprise-application-integration-eai>
  *Why: foreshadows Week 7's integration/API work with the enterprise-scale version of the same problem.*

## Glossary

| Term | Definition |
|------|------------|
| **ERP** | Enterprise Resource Planning — an integrated system running a company's operational/financial backbone (inventory, purchasing, finance, and related resources). |
| **CRM** | Customer Relationship Management — a system tracking the sales pipeline and customer relationship history, before and around a sale. |
| **Master data** | Data describing enduring business entities (customers, products, suppliers, employees) that exist independent of any single transaction. |
| **Transactional data** | Data describing events tied to a point in time, referencing master data (orders, purchase orders, invoices). |
| **Single source of truth (SSOT)** | The pre-agreed authoritative system/record for a piece of data when multiple systems could otherwise disagree. |
| **Data ownership** | The organizational assignment of accountability for a piece of master data's accuracy to a specific role or team. |
| **MDM (Master Data Management)** | The discipline (and often dedicated software) for governing, matching, and mastering an organization's master data across systems. |
| **Order-to-cash (O2C)** | The end-to-end sales process: opportunity → sales order → fulfillment → invoice → payment. |
| **Procure-to-pay (P2P)** | The end-to-end purchasing process: requisition → purchase order → goods receipt → vendor invoice → payment. |
| **Three-way match** | Verifying the PO, goods receipt, and vendor invoice all agree before a payment is approved. |
| **Golden record** | The single, authoritative, deduplicated version of a master-data entity after reconciliation. |
| **Total cost of ownership (TCO)** | The full multi-year cost of a system, including license/build, implementation, support, maintenance, and staffing — not just the sticker price. |

---

*Broken link? Open an issue or PR.*
