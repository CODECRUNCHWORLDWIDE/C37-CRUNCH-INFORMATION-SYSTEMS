# Week 12 — Resources

Free, public, no signup unless noted. Read the "required" set; treat the rest as reference you dip into when a specific part of the capstone needs it. This week draws on every prior week's toolchain at once — the links below are grouped by the layer they support, mirroring Lecture 1's stack.

## Install / confirm first

You should already have these from earlier weeks; confirm they're current before the mini-project.

- **PostgreSQL 16+** — <https://www.postgresql.org/download/> — the primary engine for your schema (Week 3).
- **Python 3.10+** with `pip` — for the TCO model, seed data generation, automation script, and AI feature.
- **A cloud account on at least one free tier** — for the deployment plan (and live deploy if you attempt it): [Render](https://render.com/), [Fly.io](https://fly.io/), [Railway](https://railway.app/), or a cloud provider's free tier (AWS/GCP/Azure all offer one — Week 8 covers the trade-offs).

```bash
python3 -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install flask flask-cors sqlalchemy psycopg2-binary python-dotenv pandas scikit-learn
```

## Required reading (this week's core)

- **PostgreSQL — Row Security Policies:** <https://www.postgresql.org/docs/current/ddl-rowsecurity.html>
  *Why: your mini-project's security layer needs at least one real `CREATE POLICY` rule, not just a description.*
- **The Twelve-Factor App:** <https://12factor.net/>
  *Why: the closest thing to an industry-standard checklist for "is this system actually deployable and operable," which is exactly what Lecture 2's operating plan and Exercise 3 are testing for.*
- **Google's "Site Reliability Engineering" book, Ch. 1 (free online):** <https://sre.google/sre-book/introduction/>
  *Why: the clearest free treatment of what "operating a system after launch" really costs — grounds Lecture 2's TCO "maintain" bucket in real practice.*
- **A16Z / general primer on "build vs. buy":** search for current, reputable engineering-blog treatments of build-vs-buy decision frameworks (e.g., major cloud providers' architecture blogs) — use as a sanity check against Lecture 1 section 5's per-layer guidance, not a replacement for it.

## Architecture & diagramming

- **draw.io / diagrams.net** (free, no signup for local use): <https://app.diagrams.net/>
  *Why: a fast, free tool for Exercise 2's architecture diagram if you'd rather not do ASCII art; exports directly to SVG/PNG for embedding in Markdown.*
- **Mermaid (diagram-as-code, renders in many Markdown viewers including GitHub):** <https://mermaid.js.org/>
  *Why: lets you version-control your architecture diagram as text, which fits this course's "no spreadsheets, reproducible artifacts" ethic — a `mermaid` fenced code block renders directly in many Markdown viewers.*
- **C4 Model (a standard vocabulary for system diagrams — Context, Container, Component, Code):** <https://c4model.com/>
  *Why: if you want a proven diagramming standard instead of inventing your own conventions; its "Container" level maps closely to this week's six-layer stack.*

## Cost modeling

- **AWS Pricing Calculator:** <https://calculator.aws/>
- **Render pricing:** <https://render.com/pricing>
- **Fly.io pricing:** <https://fly.io/docs/about/pricing/>
  *Why: ground your Exercise 3 / mini-project TCO model's `monthly_cloud` input in a real, current number instead of a guess — cite which page you used.*

## Presenting & defending

- **"Presenting to Win" — general public-speaking-for-technical-work advice** (search for current, reputable treatments; avoid outdated or paywalled sources).
- **The Pyramid Principle (Barbara Minto) — summarized freely in many public write-ups**
  *Why: the classic "conclusion first, then supporting detail" structure behind Lecture 3's top-down presentation order; useful background, not required reading.*

## AI feature building blocks

- **scikit-learn — Getting Started:** <https://scikit-learn.org/stable/getting_started.html>
  *Why: if your AI feature is a classifier or scoring model (e.g., churn/no-show risk), this is the fastest path to something real and explainable.*
- **Anthropic / OpenAI API docs** (use whichever the rest of this course's Week 11 material referenced): check your Week 11 lecture notes for the exact provider and pattern used, and reuse it here rather than introducing a new one for the first time in the capstone.

## Glossary

| Term | Definition |
|------|------------|
| **TCO (total cost of ownership)** | Build cost plus all recurring run and maintain costs over a defined horizon (this course uses 3 years). |
| **Phased delivery** | Splitting a system into increments, each independently valuable, ordered by risk. |
| **Risk register** | A ranked table of what could go wrong, its likelihood × impact, its mitigation, and its owner. |
| **Build vs. buy** | The decision to custom-build a capability versus purchasing/using an existing product or service for it. |
| **Seam** | A boundary between two layers or components where integration commonly breaks (schema drift, duplicated security rules, batch/real-time mismatch, cost blind spots). |
| **Strategic alignment** | Whether a system feature actually serves the organization's stated goal, versus existing because it was interesting to build. |
| **C4 model** | A standard four-level vocabulary (Context, Container, Component, Code) for architecture diagrams. |
| **Human-in-the-loop** | A design pattern where a human reviews or approves an AI/ML output before it affects a real decision or person. |
| **Executive summary** | A one-page document summarizing problem, purpose, cost, phased value, risk, and the ask — designed to survive being read without the author present. |

---

*Broken link? Open an issue or PR.*
