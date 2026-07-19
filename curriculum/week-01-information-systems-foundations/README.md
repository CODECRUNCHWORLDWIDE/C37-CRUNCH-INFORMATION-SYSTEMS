# Week 1 — Information Systems Foundations

> **Goal:** by Sunday you can look at any organization — a coffee roaster, a hospital wing, your own side hustle — and map it as an information system: the people who do the work, the processes they follow, the data those processes produce and consume, and the technology that carries it all — with the flows between them precise enough that a stranger could read your map and understand how the business actually runs.

Welcome to **C37 · Crunch Information Systems**. Before you can design, secure, or automate a system, you have to be able to *see* one — and most people can't. Ask someone what "the system" is at their job and they'll point at a piece of software. That's wrong, or at least incomplete: the software is one component of something bigger. An information system is the whole loop — a person does a task, following a process, that touches data, carried by technology — and it existed in some form long before computers did. A ship's captain's log, a library's card catalog, a restaurant's order pad and kitchen bell are all information systems. Computers made them faster and more reliable; they didn't invent the concept.

This week gives you the vocabulary and the mental model the rest of this course builds on. Every later week — requirements (Week 2), data modeling (Week 3), automation (Week 5), cloud (Week 8), AI (Week 11) — is really just going deeper into one corner of the four-part picture you draw this week: **people, process, data, technology.**

We work all week against one running case: **Riverbend Coffee Roasters**, a small company that roasts coffee, sells drinks and bags over a retail counter, ships wholesale orders to cafés and grocers, and runs a home-delivery subscription. It's deliberately not a tech company — most organizations aren't — so you learn to see the information system *inside* an ordinary business, not just the software layer bolted on top of it. Per this course's data rule, we track Riverbend's system map itself — every person, process, data item, and piece of technology, and how they connect — in **SQL tables**, never a spreadsheet. That's the first lesson in disguise: a shared list that three people edit in Excel *is* the kind of no-audit-trail, single-point-of-failure problem this course exists to teach you to avoid, and you'll feel the difference by using a real, constrained, queryable store from day one.

## Learning objectives

By the end of this week, you will be able to:

- **Define** what an information system is — formally, as an input→process→output→feedback loop with a boundary and an environment — and explain why "information system" is not a synonym for "software" or "computer."
- **Name and classify** the four components of any information system — **people, process, data, technology** — and correctly sort ambiguous real-world items into the right bucket.
- **Distinguish** data from information from knowledge (the first three layers of the DIKW pyramid), and explain why a pile of data with no process around it creates no value.
- **Trace** a flow of information end-to-end through an organization — who touches it, what process transforms it, what technology carries it, and what decision it ultimately supports.
- **Diagnose** a broken or informal information system (paper, sticky notes, an uncontrolled shared spreadsheet) and name precisely which component or flow is missing, and what business risk that creates.
- **Model** a real organization's people/process/data/technology components and their flows in SQL tables — the same discipline you'll use for requirements (Week 2) and formal data models (Week 3).

## Prerequisites

- You can run commands in a terminal (open one, type a command, read the output).
- **No** prior systems, database, or business-analysis knowledge. That's what this course is for.
- PostgreSQL 16+ **or** SQLite 3.35+ installed. Both are free. Install steps are in [`resources.md`](./resources.md). If you install only one, install **PostgreSQL** — it's the primary engine for this course; SQLite is the zero-setup fallback. This is the first week you'll need a database, so budget time for install if you haven't done it for another course.
- Basic comfort typing `CREATE TABLE`, `INSERT INTO`, and `SELECT` is helpful but **not required** — this week teaches you just enough SQL to store and retrieve a structured list. If you want the fuller grounding first, [C33 Crunch SQL](../../../C33-CRUNCH-SQL/) Week 1 is a great companion, not a prerequisite.

## Set up the information-systems register (do this first)

Everything this week reads and writes to two small tables: `is_components` (the people, processes, data, and technology that make up a system) and `is_flows` (how they connect). Create them once, seeded with a first pass at Riverbend Coffee Roasters — you'll extend it across the week's lectures, exercises, and challenges.

**PostgreSQL:**

```bash
createdb riverbend
psql riverbend
```

**SQLite:**

```bash
sqlite3 riverbend.db
```

Paste this into the shell (works unchanged on both engines):

```sql
CREATE TABLE is_components (
    component_id  INTEGER PRIMARY KEY,
    category      TEXT NOT NULL,   -- 'people' | 'process' | 'data' | 'technology'
    name          TEXT NOT NULL,
    description   TEXT NOT NULL,
    department    TEXT NOT NULL
);

INSERT INTO is_components VALUES
(1,'people','Elena Vasquez','Head Roaster — plans and runs every roast batch','Production'),
(2,'people','Devon Park','Barista Lead — runs the retail counter and café staff','Retail'),
(3,'people','Ruth Okafor','Wholesale Account Manager — takes and manages café/grocer orders','Wholesale'),
(4,'people','Sam Higgins','Warehouse Packer — picks, packs, and ships every outbound order','Fulfillment'),
(5,'people','Nora Chen','Owner / General Manager — watches the numbers, makes the calls','Leadership'),
(6,'process','Roast Scheduling','Turn wholesale, retail, and subscription demand into a daily roast plan','Production'),
(7,'process','Order Intake','Capture a wholesale order from a café or grocer by phone, email, or portal','Wholesale'),
(8,'process','POS Checkout','Ring up a retail drink or bag of beans at the counter','Retail'),
(9,'process','Pick-Pack-Ship','Pull, box, and label a wholesale or subscription order for carrier pickup','Fulfillment'),
(10,'process','Subscription Renewal','Auto-charge and re-ship a recurring home-delivery order','Digital'),
(11,'data','Green Coffee Inventory','Unroasted beans on hand, by origin and quantity','Production'),
(12,'data','Roast Batch Log','Time/temperature/weight-loss record for every batch roasted','Production'),
(13,'data','Customer Order','A wholesale or subscription order — items, quantities, ship-to address','Wholesale'),
(14,'data','POS Transaction','An itemized retail sale — items, timestamp, payment method','Retail'),
(15,'data','Delivery Manifest','Carrier, tracking number, and contents for one outbound shipment','Fulfillment'),
(16,'technology','Roast Profile Controller','Software that drives the roaster''s heat/airflow curve','Production'),
(17,'technology','Square POS Terminal','Card-swipe register at the café counter','Retail'),
(18,'technology','Wholesale Order Portal','Web form wholesale customers use to place or repeat an order','Wholesale'),
(19,'technology','Warehouse Scanner','Barcode scanner that confirms each picked item matches the order','Fulfillment'),
(20,'technology','Subscription Billing Platform','Recurring-charge engine for home-delivery customers','Digital');

CREATE TABLE is_flows (
    flow_id                INTEGER PRIMARY KEY,
    source_component_id    INTEGER NOT NULL REFERENCES is_components(component_id),
    target_component_id    INTEGER NOT NULL REFERENCES is_components(component_id),
    data_exchanged         TEXT NOT NULL,
    frequency               TEXT NOT NULL,   -- 'real-time' | 'daily' | 'weekly' | 'on-demand'
    notes                   TEXT
);

INSERT INTO is_flows VALUES
(1,3,7,'wholesale order details called or emailed in','daily',NULL),
(2,18,7,'submitted order form','real-time',NULL),
(3,7,13,'validated order record','real-time','Order Intake writes the confirmed order'),
(4,13,6,'confirmed order quantities','daily',NULL),
(5,11,6,'on-hand bean stock by origin','daily',NULL),
(6,6,1,'today''s roast batch plan','daily',NULL),
(7,1,16,'roast curve settings for the batch','on-demand',NULL),
(8,16,12,'time/temp/weight-loss readings','real-time',NULL),
(9,12,11,'beans consumed, deducted from stock','daily',NULL),
(10,2,17,'rings up drink or bag sale','real-time',NULL),
(11,17,14,'itemized sale record','real-time',NULL),
(12,14,5,'daily retail sales summary','daily',NULL),
(13,13,9,'items + ship-to address to fulfill','daily',NULL),
(14,4,19,'scans each picked item','real-time',NULL),
(15,19,15,'confirmed contents + tracking number','real-time',NULL),
(16,9,15,'packed order ready for carrier pickup','daily',NULL),
(17,20,10,'due-for-renewal customer list','weekly',NULL),
(18,10,13,'auto-generated recurring order','weekly',NULL),
(19,15,5,'on-time ship rate','weekly',NULL);
```

Sanity check — this should print `20`:

```sql
SELECT COUNT(*) FROM is_components;
```

Notice what the seed does **not** yet contain: a Finance/Support department, a data flow back from the customer, and a few other gaps. Those are there **on purpose** — you'll find and fill them across this week's exercises and challenges. A "complete" system map on day one would defeat the point of the week.

## Weekly schedule

The schedule below adds up to approximately **28 hours** (the course's full-time pace). Treat it as a target.

| Day | Focus | Lectures | Exercises | Challenges | Quiz/Read | Homework | Mini-Project | Daily Total |
|-----------|------------------------------------------------|---------:|----------:|-----------:|----------:|---------:|-------------:|------------:|
| Monday | Install + seed; what an information system *is* | 2h | 1h | 0h | 0.5h | 1h | 0h | 4.5h |
| Tuesday | People, process, data, technology | 2h | 1.5h | 0h | 0.5h | 1h | 0h | 5h |
| Wednesday | DIKW, value, and mapping flows | 2h | 1.5h | 1h | 0.5h | 1h | 0h | 6h |
| Thursday | Diagnosing broken systems; challenges | 0h | 1.5h | 1h | 0.5h | 1h | 1h | 5h |
| Friday | Modeling growth; catch-up | 0h | 0h | 1h | 0.5h | 1h | 1.5h | 4h |
| Saturday | Mini-project (map a real business) | 0h | 0h | 0h | 0h | 0h | 2.5h | 2.5h |
| Sunday | Quiz + review | 0h | 0h | 0h | 1h | 0h | 0h | 1h |
| **Total** | | **6h** | **5.5h** | **3h** | **3.5h** | **5h** | **5h** | **28h** |

## How to navigate this week

Work top to bottom. Each piece assumes the ones above it.

| # | File | What's inside | ~Time |
|--:|------|---------------|------:|
| 1 | [lecture-notes/01-what-is-an-information-system.md](./lecture-notes/01-what-is-an-information-system.md) | Definition, input/process/output/feedback, boundary vs. environment, IS vs. IT | 2h |
| 2 | [lecture-notes/02-people-process-data-technology.md](./lecture-notes/02-people-process-data-technology.md) | The four components in depth, classifying ambiguous items, the SQL register | 2h |
| 3 | [lecture-notes/03-how-information-systems-create-value.md](./lecture-notes/03-how-information-systems-create-value.md) | DIKW pyramid, value chain, efficiency vs. effectiveness, value destruction | 2h |
| 4 | [exercises/exercise-01-classify-the-components.md](./exercises/exercise-01-classify-the-components.md) | Sort real-world items into people/process/data/technology; extend the register | 1h |
| 5 | [exercises/exercise-02-map-the-data-flows.md](./exercises/exercise-02-map-the-data-flows.md) | Insert and query `is_flows`; find components with no connections | 1.5h |
| 6 | [exercises/exercise-03-data-information-knowledge-wisdom.md](./exercises/exercise-03-data-information-knowledge-wisdom.md) | Turn raw POS data into information, knowledge, and a decision | 1.5h |
| 7 | [challenges/challenge-01-diagnose-the-before-state.md](./challenges/challenge-01-diagnose-the-before-state.md) | Find what was missing in Riverbend's pre-register, paper-and-spreadsheet days | 1h |
| 8 | [challenges/challenge-02-model-a-new-location.md](./challenges/challenge-02-model-a-new-location.md) | Add a second café location — decide what's shared vs. duplicated | 1h |
| 9 | [mini-project/README.md](./mini-project/README.md) | Map a real business as a full information-systems register | 2.5h |
| 10 | [homework.md](./homework.md) | Extra practice, spaced out | 5h |
| 11 | [quiz.md](./quiz.md) | 14 self-check questions + answer key | 1h |
| 12 | [resources.md](./resources.md) | Official docs + the few links worth your time | — |

## By the end of this week you can…

- Explain what an information system is without pointing at a screen.
- Sort any real-world item — a person, a habit, a form, a gadget — into people, process, data, or technology, and defend the call when it's genuinely ambiguous.
- Trace one piece of information from where it's created to the decision it eventually supports.
- Spot the specific missing component or flow that turns "we have a process for that" into a business incident.
- Read and extend a small SQL register — the same discipline requirements (Week 2) and data models (Week 3) build on.

## Up next

[Week 2 — Systems Analysis & Requirements](../week-02-systems-analysis-and-requirements/) — once you can see a system, the next skill is turning a vague organizational pain point into a requirements spec someone can actually build against.

---

*Part of the Code Crunch Worldwide open curriculum · GPL-3.0 · If you find errors, please open an issue or PR.*
