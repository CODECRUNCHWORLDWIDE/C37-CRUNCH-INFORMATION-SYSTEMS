# C37 · Crunch Information Systems

> A free, open-source 12-week course on designing intelligent information systems with data, automation, cloud, and AI — systems analysis, data modeling, integration, and governance for real organizations.

[![License: GPL v3](https://img.shields.io/badge/License-GPL%20v3-blue.svg)](LICENSE)
[![PostgreSQL · Python](https://img.shields.io/badge/stack-PostgreSQL_·_Python-2463EB.svg)](#stack)
[![Built in the open](https://img.shields.io/badge/built-in%20the%20open-2463EB.svg)](https://github.com/CODECRUNCHWORLDWIDE)

C37 is a Tier-1 foundation course, sized long (12 weeks) because it goes all the way — from "what is an information system?" to designing, integrating, securing, and governing the systems a whole business runs on. It ties together the skills that other tracks teach in isolation: you learn to analyze a real organization, model its data, automate its processes, host it in the cloud, and layer analytics and AI on top — done right. It pairs naturally with [C33 Crunch SQL](../C33-CRUNCH-SQL/), [C27 Crunch Data](../C27-CRUNCH-DATA/), and [C5 Crunch AI & Data Science](../C5-CRUNCH-AI-DATA-SCIENCE/) — this course makes you the person who designs the system, not just one component of it.

---

## Pathway summary

- **Full-time:** 12 weeks · ~28 hrs/week · ~336 hours
- **Working-professional pace:** 6 months · ~14 hrs/week
- **Evening pace:** 12 months · ~7 hrs/week

See [`SYLLABUS.md`](SYLLABUS.md).

---

## What you will be able to do at the end of 12 weeks

- **See the whole system:** describe any organization as people + process + data + technology, and map how information flows and creates value.
- **Analyze and gather requirements:** interview stakeholders, write use cases and user stories, and turn a messy real-world need into a clear, testable specification.
- **Model data properly:** design an ER model, normalize a schema, and implement it in SQL (PostgreSQL) — never in a spreadsheet.
- **Query and wrangle at will:** answer business questions with SQL joins/aggregation and Python (pandas), and build repeatable data pipelines instead of manual copy-paste.
- **Model and automate processes:** diagram a workflow in BPMN and automate it end to end with code — removing the manual steps that break at scale.
- **Understand enterprise systems:** reason about ERP/CRM concepts, master data, and why integration — not features — is where real systems live or die.
- **Integrate everything:** design REST APIs, consume third-party services, handle webhooks, and build ETL/ELT integrations between systems.
- **Architect for the cloud:** choose IaaS/PaaS/SaaS deliberately, deploy a system, and reason about availability, scaling, and cost.
- **Secure, govern, and comply:** apply access control, data privacy, and governance so the system holds up to audit and attack.
- **Deliver analytics and AI:** build BI dashboards on a real warehouse and embed AI/ML into a system as a decision-support feature — responsibly.
- **Design and defend a full system:** deliver an end-to-end intelligent information system for a real organization — the capstone.

---

## Curriculum (12 weeks)

| Week | Topic | You leave able to… |
|------|-------|--------------------|
| 1 | [Information systems foundations](curriculum/week-01-information-systems-foundations/) | Map any organization as people, process, data, and technology. |
| 2 | [Systems analysis & requirements](curriculum/week-02-systems-analysis-and-requirements/) | Turn a real-world need into a clear, testable specification. |
| 3 | [Data modeling & databases](curriculum/week-03-data-modeling-and-databases/) | Design an ER model and implement a normalized schema in SQL. |
| 4 | [Querying data with SQL & Python](curriculum/week-04-querying-data-with-sql-and-python/) | Answer business questions with SQL joins and pandas pipelines. |
| 5 | [Process modeling & automation](curriculum/week-05-process-modeling-and-automation/) | Diagram a workflow in BPMN and automate it end to end. |
| 6 | [Enterprise systems (ERP/CRM)](curriculum/week-06-enterprise-systems-erp-crm/) | Reason about ERP/CRM, master data, and enterprise data flows. |
| 7 | [Integration & APIs](curriculum/week-07-integration-and-apis/) | Design REST APIs and build integrations between systems. |
| 8 | [Cloud architecture](curriculum/week-08-cloud-architecture/) | Choose a cloud model and deploy a system that scales. |
| 9 | [Security, privacy & governance](curriculum/week-09-security-privacy-and-governance/) | Secure and govern a system so it survives audit and attack. |
| 10 | [Analytics & business intelligence](curriculum/week-10-analytics-and-business-intelligence/) | Build a warehouse and BI dashboards from real data. |
| 11 | [AI integration](curriculum/week-11-ai-integration/) | Embed AI/ML into a system as responsible decision support. |
| 12 | [Capstone — design an intelligent system](curriculum/week-12-capstone-design-an-intelligent-system/) | Design, defend, and cost a full intelligent information system. |

---

## How to navigate a week

Every week folder holds the same structure:

- **`README.md`** — the week overview + how the pieces fit + the week's goal.
- **`lecture-notes/`** — 3 lectures (~2 hrs each), the conceptual core.
- **`exercises/`** — 3 short, guided reps on a real system or dataset.
- **`challenges/`** — 2 open-ended problems with no single right answer.
- **`mini-project/`** — one build that ties the week together.
- **`homework.md`**, **`quiz.md`**, **`resources.md`** — practice, self-check, and further reading.

---

## Stack

PostgreSQL (16+) as the primary data store, with SQLite for zero-setup practice, and Python 3 (with pandas) for wrangling, automation, and integration. Data always lives in SQL or Python — **never in a spreadsheet used as a database.** (Spreadsheets are taught only in [C41 Crunch Excel](../C41-CRUNCH-EXCEL/).) You'll also use `psql`, `git`, HTTP/REST tooling, and a free cloud tier for deployment. Everything is free and runs on macOS, Linux, and Windows.

---

*Part of the Code Crunch Worldwide open curriculum · GPL-3.0 · [Browse all courses](https://codecrunchglobal.vercel.app/courses)*
