# Week 2 — Resources

Curated, free, official. You don't need to read all of this to finish the week — it's here for when a lecture references a standard or technique and you want the primary source.

## Standards & bodies of knowledge

- **ISO/IEC/IEEE 29148:2018 — Systems and software engineering, requirements engineering.** The formal international standard behind the "clear, testable, atomic, feasible, traceable, prioritized" qualities taught in Lecture 2. <https://www.iso.org/standard/72089.html>
- **IIBA — Business Analysis Body of Knowledge (BABOK), Elicitation & Collaboration + Requirements Analysis & Design Definition chapters.** The professional reference for everything in Lecture 1 and 2. Free overview and guide available at <https://www.iiba.org/business-analysis-body-of-knowledge/>
- **IEEE 830 (superseded by 29148, still widely referenced) — Recommended Practice for Software Requirements Specifications.** Historical but still the template many real SRS documents descend from.

## Elicitation & interviewing

- **IIBA — Elicitation techniques overview** (interviews, observation, document analysis, workshops, surveys): <https://www.iiba.org/business-analysis-body-of-knowledge/>
- **The Five Whys (Toyota Production System origin, widely adapted for requirements laddering):** a short, practical technique reference — search "Five Whys root cause analysis" for the classic Toyota-originated explanation any manufacturing/lean-methodology source covers well.

## Prioritization

- **DSDM Consortium — the original MoSCoW method (Must/Should/Could/Won't):** <https://www.agilebusiness.org/dsdm-project-framework/moscow-prioritisation.html>

## Use cases

- **Alistair Cockburn — "Writing Effective Use Cases."** The canonical reference for use-case structure (actors, main/alternate/exception flows), used almost unchanged in Lecture 3. <https://alistair.cockburn.us/get-the-must-have-book-writing-effective-use-cases/>
- **UML use-case diagram notation reference** (for when you want to draw the actor/use-case bubble diagrams alongside the text spec): any current UML 2.x reference covers this; the OMG's own UML specification is the primary source at <https://www.omg.org/spec/UML/>

## User stories & acceptance criteria

- **Bill Wake — the original INVEST criteria for user stories:** <https://xp123.com/articles/invest-in-good-stories-and-smart-tasks/>
- **Mike Cohn — "User Stories Applied" concepts, story splitting patterns:** <https://www.mountaingoatsoftware.com/agile/user-stories>
- **Cucumber / Gherkin — the Given/When/Then syntax reference used for acceptance criteria:** <https://cucumber.io/docs/gherkin/reference/>

## Diagramming (current-state / future-state, use-case flows)

- **Mermaid — flowchart and sequence diagram syntax** (renders natively in GitHub markdown, used in Exercise 3): <https://mermaid.js.org/intro/>
- **BPMN quick reference** (you'll go deeper on this in Week 5 — Process Modeling & Automation, but the swimlane concept in Exercise 3 is a BPMN idea): <https://www.bpmn.org/>

## This course's tools (installed in Week 1, used again this week)

- **PostgreSQL — `CREATE TABLE`, `INSERT`, `JOIN` reference:** <https://www.postgresql.org/docs/current/sql-createtable.html> and <https://www.postgresql.org/docs/current/sql-insert.html>
- **PostgreSQL download:** <https://www.postgresql.org/download/>
- **SQLite download (zero-setup fallback):** <https://www.sqlite.org/download.html>
- **`psql` cheat sheet** (connecting, `\d`, `\dt`): <https://www.postgresql.org/docs/current/app-psql.html>

## Why we don't use a spreadsheet here

If you're ever tempted to track a real requirements register in Excel or Google Sheets instead of SQL, revisit this course's data rule: a spreadsheet has no enforced structure (a status column can hold `"ordered"`, `"Ordered"`, `"?"`, and blank all at once — exactly what broke Meridian's current process), no real concurrency control (multiple people editing the same cell silently overwrite each other), and no query language for the kind of traceability check Lecture 2 §5 relies on. A `requirements` table with a `CHECK` constraint and a foreign key to `stakeholders` cannot silently drift the way a shared spreadsheet tab does. Spreadsheets have their place — it's [C41 Crunch Excel](../../../C41-CRUNCH-EXCEL/), not here.
