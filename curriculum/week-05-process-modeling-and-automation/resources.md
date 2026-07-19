# Week 5 — Resources

Curated, not exhaustive. Everything below is free. Install what you need before Monday; the rest is reference material to return to as questions come up during the week.

## Install first

| Tool | Why you need it | Get it |
|------|------------------|--------|
| **PostgreSQL 16+** | Primary database engine for this week's automation | <https://www.postgresql.org/download/> — `brew install postgresql@16` (macOS), `apt install postgresql` (Debian/Ubuntu), or the Windows installer |
| **Python 3.10+** | The automation scripts in Lecture 3 and beyond | <https://www.python.org/downloads/> (most systems already have it — check with `python3 --version`) |
| **pip packages** | `psycopg2-binary`, `python-dotenv`, `APScheduler`, `tenacity` | `pip install -r requirements.txt` using the file from [Lecture 3](./lecture-notes/03-building-reliable-automation.md#3-project-layout) |

## BPMN — notation and tools

- **OMG BPMN 2.0 specification (the source of truth for every symbol):** <https://www.omg.org/spec/BPMN/2.0/>
- **Camunda's BPMN symbol reference (the best free visual glossary — bookmark this one):** <https://camunda.com/bpmn/reference/>
- **bpmn.io — free, no-install, browser-based BPMN editor, standard notation:** <https://demo.bpmn.io/>
- **draw.io (diagrams.net) — general diagramming, has a BPMN shape library, works fully offline:** <https://www.drawio.com/>
- **Mermaid flowchart syntax — used for every diagram in this week's lectures, renders inline in Markdown/GitHub:** <https://mermaid.js.org/syntax/flowchart.html>

## Process mining and automation-candidate thinking

- **processmining.org — accessible overview of process mining as a discipline (Van der Aalst's community site):** <https://www.processmining.org/>
- **Camunda, "What is Business Process Automation?" (vendor-neutral concepts, worth the 10-minute read):** <https://camunda.com/best-practices/business-process-automation/>

## Python for reliable automation

- **`psycopg2` documentation (the Postgres driver used all week):** <https://www.psycopg.org/docs/>
- **PostgreSQL — Transactions (commit/rollback, the mechanism behind idempotent writes):** <https://www.postgresql.org/docs/current/tutorial-transactions.html>
- **PostgreSQL — `INSERT ... ON CONFLICT` (the upsert syntax used in Challenge 2's fix):** <https://www.postgresql.org/docs/current/sql-insert.html#SQL-ON-CONFLICT>
- **PostgreSQL — date/time functions (`EXTRACT`, `now()`, interval math):** <https://www.postgresql.org/docs/current/functions-datetime.html>
- **Python `logging` HOWTO (official docs — read this once, use it forever):** <https://docs.python.org/3/howto/logging.html>
- **`tenacity` — retry library docs, used for the transient-failure retry decorator:** <https://tenacity.readthedocs.io/>
- **`python-dotenv` — loading credentials from a `.env` file, never hardcoded:** <https://pypi.org/project/python-dotenv/>

## Scheduling

- **APScheduler documentation:** <https://apscheduler.readthedocs.io/>
- **crontab.guru — build and verify cron schedule expressions interactively:** <https://crontab.guru/>
- **`man 5 crontab`** — run this in any terminal for the canonical crontab format reference; always available, no internet required.

## Related weeks in this course

- **[C37 Week 3 — Data Modeling & Databases](../week-03-data-modeling-and-databases/)** — where the CrunchRide schema and normalization discipline this week's tables follow were established.
- **[C37 Week 4 — Querying Data with SQL + Python](../week-04-querying-data-with-sql-and-python/)** — the SQL + pandas fluency this week's queries assume.
- **[C33 Crunch SQL](../../../C33-CRUNCH-SQL/)** — a full standalone SQL course if `EXTRACT`, joins, or transactions feel shaky.
- **[C1 · Code Crunch Convos — Python Bootcamp](../../../C1-Code-Crunch-Convos/)** — a full standalone Python course if functions, `try`/`except`, or reading library docs feel shaky.
