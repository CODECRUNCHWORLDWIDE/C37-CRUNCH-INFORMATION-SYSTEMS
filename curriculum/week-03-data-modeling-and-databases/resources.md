# Week 3 — Resources

Curated, not exhaustive. Everything here is free. Install what you need before Monday; read the rest as you go, referenced from the lectures.

## Install first

### PostgreSQL 16+ (primary engine)

- **macOS:** `brew install postgresql@16 && brew services start postgresql@16`
- **Windows:** [postgresql.org/download/windows](https://www.postgresql.org/download/windows/) — use the EDB installer, includes `psql` and pgAdmin.
- **Linux (Debian/Ubuntu):** `sudo apt install postgresql postgresql-contrib`
- **Verify:** `psql --version` should print `16.x` or higher.
- **Official install docs:** <https://www.postgresql.org/download/>

### SQLite 3.35+ (fallback, zero setup)

- **macOS/Linux:** usually preinstalled — check with `sqlite3 --version`. If missing: `brew install sqlite3` (macOS) or `sudo apt install sqlite3` (Linux).
- **Windows:** download the precompiled binary from <https://www.sqlite.org/download.html>.
- Remember Lecture 3's warning: run `PRAGMA foreign_keys = ON;` at the start of every SQLite session, or foreign keys are silently unenforced.

### A diagramming tool (optional but recommended)

- **[dbdiagram.io](https://dbdiagram.io)** — free, browser-based, text-to-diagram (DBML syntax), exports PNG/PDF. The fastest way to turn an ER model into a shareable diagram. No account required for basic use.
- **[draw.io](https://www.drawio.com)** (also at app.diagrams.net) — free, general-purpose diagramming with a crow's-foot ER shape library, works offline as a desktop app too.
- Pen and paper, photographed, is completely acceptable for this week's exercises — don't let tooling become a blocker.

## Official documentation (the primary source — read these over any blog post)

- **PostgreSQL — Data Definition Language (DDL) chapter:** <https://www.postgresql.org/docs/current/ddl.html>
- **PostgreSQL — `CREATE TABLE` full reference:** <https://www.postgresql.org/docs/current/sql-createtable.html>
- **PostgreSQL — Constraints (`CHECK`, `NOT NULL`, `UNIQUE`, `FOREIGN KEY`):** <https://www.postgresql.org/docs/current/ddl-constraints.html>
- **PostgreSQL — Data types:** <https://www.postgresql.org/docs/current/datatype.html>
- **PostgreSQL — Indexes overview:** <https://www.postgresql.org/docs/current/indexes.html>
- **PostgreSQL — Date/Time types (`DATE` vs. `TIMESTAMP` vs. `TIMESTAMPTZ`):** <https://www.postgresql.org/docs/current/datatype-datetime.html>
- **SQLite — Foreign key support (read this before assuming your FKs are enforced):** <https://www.sqlite.org/foreignkeys.html>
- **SQLite — Data types (type affinity, the looser type system):** <https://www.sqlite.org/datatype3.html>

## Normalization theory

- **"A Simple Guide to Five Normal Forms in Relational Database Theory" — William Kent, 1983.** Still the clearest plain-English explanation of 1NF–5NF ever written, and it's free: <https://www.bkent.net/Doc/simple5.htm>
- **PostgreSQL wiki — "Database Normalization":** <https://wiki.postgresql.org/wiki/Database_normalization> — a shorter, Postgres-flavored companion to Kent's paper.

## ER modeling notation

- **"Crow's Foot Notation" reference (Vertabelo Academy):** <https://academy.vertabelo.com/blog/crow-s-foot-notation/> — the clearest visual reference for the symbol pairs used in Lecture 1.
- **DBML language reference** (the text format dbdiagram.io uses): <https://dbml.dbdiagram.io/docs/> — worth skimming if you want to describe schemas as text instead of dragging boxes.

## From this course

- [C33 · Crunch SQL](../../../C33-CRUNCH-SQL/) — if `SELECT`, `WHERE`, and basic query syntax feel shaky, Weeks 1–2 of that course are the ideal companion before or alongside this week.
- This week's own [Week 1 — Information Systems Foundations](../week-01-information-systems-foundations/) and [Week 2 — Systems Analysis + Requirements](../week-02-systems-analysis-and-requirements/) — the "why are we modeling this system at all" and "what did the stakeholder actually ask for" context this week's schema work assumes.

## A note on AI-assisted modeling tools

Several tools now offer "describe your app, get a schema" AI generation. They can be a fast way to get a **first draft** to react to — but the actual skill this week teaches (reading ambiguity, choosing keys deliberately, naming the specific anomaly a constraint prevents) is exactly the judgment an AI-generated schema hasn't exercised for you. Use such tools, if you use them at all, the way you'd use a rough sketch from a junior colleague: a starting point to critique and correct, never a design to trust unreviewed. Challenge 1 this week (modeling from a noisy interview transcript) is deliberately hard to shortcut this way — the noise-filtering judgment is the point.
