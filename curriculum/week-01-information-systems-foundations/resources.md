# Week 1 — Resources

Free, public, no signup unless noted. Read the "required" set; treat the rest as reference you dip into when a specific question comes up.

## Install first

This is the first week of C37 that needs a database — if you've already installed one for [C33 Crunch SQL](../../../C33-CRUNCH-SQL/) or another course, skip ahead.

- **PostgreSQL 16+** — this course's primary engine:
  <https://www.postgresql.org/download/> · macOS: [Postgres.app](https://postgresapp.com/) is the easiest. Linux: `sudo apt install postgresql` / `sudo dnf install postgresql-server`. Windows: the EDB installer.
- **SQLite 3.35+** — the zero-setup fallback; ships on macOS and most Linux already: <https://www.sqlite.org/download.html>. Check with `sqlite3 --version`.
- **A GUI (optional, later)** — [DBeaver](https://dbeaver.io/) (free, both engines) or [pgAdmin](https://www.pgadmin.org/) (Postgres). This week, stay in the terminal — typing `INSERT` statements by hand is part of the point.

## Required reading (this week's core)

- **NIST — "information system" (glossary definition):** <https://csrc.nist.gov/glossary/term/information_system>
  *Why: the formal, standards-body definition behind Lecture 1's working definition.*
- **Wikipedia — Information system (five-component framework):** <https://en.wikipedia.org/wiki/Information_system>
  *Why: the classic hardware/software split this course's "technology" category simplifies — good context for Lecture 2 §0.*
- **Wikipedia — DIKW pyramid:** <https://en.wikipedia.org/wiki/DIKW_pyramid>
  *Why: the canonical treatment of data→information→knowledge→wisdom behind Lecture 3 §1 — read before the mini-project's Section 3.*
- **Investopedia — Porter's Value Chain:** <https://www.investopedia.com/terms/v/valuechain.asp>
  *Why: the primary-vs-support activity model behind Lecture 3 §2, with more industry examples than the lecture has room for.*

## Reference (keep in tabs)

- **Wikipedia — Systems theory:** <https://en.wikipedia.org/wiki/Systems_theory>
  *Why: the deeper background on the input/process/output/feedback loop, beyond what Lecture 1 §3 covers.*
- **Wikipedia — Effectiveness (efficiency vs. effectiveness):** <https://en.wikipedia.org/wiki/Effectiveness>
  *Why: more worked examples of the two-axis distinction from Lecture 3 §3.*
- **PostgreSQL — "Populating a database":** <https://www.postgresql.org/docs/current/populate.html>
  *Why: the official reference for `INSERT`, in case anything in the exercises trips you up.*
- **SQLite — `INSERT` syntax:** <https://www.sqlite.org/lang_insert.html>
  *Why: same, for the SQLite engine.*
- **PostgreSQL — `SELECT` reference:** <https://www.postgresql.org/docs/current/sql-select.html>
  *Why: you'll be writing `WHERE` clauses and simple filters all week; bookmark this now, you'll use it constantly through Week 4.*

## On the anti-pattern this course avoids

- **Wikipedia — Single point of failure:** <https://en.wikipedia.org/wiki/Single_point_of_failure>
  *Why: the formal name for exactly the risk an uncontrolled shared spreadsheet creates (Lecture 2 §4) — worth understanding as a general systems-reliability concept, not just a database preference.*

## Practice beyond Riverbend

- **PostgreSQL Exercises** — free, browser-based, graded `SELECT` drills, if you want more raw SQL reps beyond this week's register work: <https://pgexercises.com/>
  *Why: this week keeps SQL intentionally light (just `CREATE TABLE`/`INSERT`/`SELECT`/`WHERE`) since the real skill is classification, not query syntax — this is where to go if you want more query practice on the side.*
