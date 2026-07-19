# Week 11 — Resources

Curated, not exhaustive. Official docs first, then the few extra links worth your time.

## Install / accounts

- **Anthropic API key** — [console.anthropic.com](https://console.anthropic.com). Free trial credit is available; this week's exercises use only a few cents of usage if followed as written. Never commit the key — it goes in a local `.env` file, gitignored, exactly as you handled database secrets in Week 9.
- **Python packages for this week:**

  ```bash
  pip install pandas scikit-learn sqlalchemy psycopg2-binary flask anthropic python-dotenv joblib
  ```

- If you're on SQLite instead of PostgreSQL for local experimentation, `sqlite3` ships with Python — no separate install — but this week's SQL (window-free, standard joins/aggregates) runs unchanged on either engine, matching every prior week's approach.

## Official documentation

- **Anthropic API reference** — [platform.claude.com/docs](https://platform.claude.com/docs) — the canonical source for the Messages API, tool use, and model IDs. If anything in this week's lectures looks out of date by the time you read it (model names, pricing), this is the source of truth, not this README.
- **Anthropic Python SDK** — [github.com/anthropics/anthropic-sdk-python](https://github.com/anthropics/anthropic-sdk-python) — the `anthropic` package used all week. Read the README for the full client API beyond what this week's lectures cover (streaming, async, batching).
- **Tool use guide** — [platform.claude.com/docs/en/agents-and-tools/tool-use/overview](https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview) — the full reference for the tool-definition schema and the tool-use loop behind Exercise 3.
- **scikit-learn: Logistic Regression** — [scikit-learn.org/stable/modules/linear_model.html#logistic-regression](https://scikit-learn.org/stable/modules/linear_model.html) — the classifier used for the order-risk model. Also worth skimming: the `Pipeline` and `ColumnTransformer` docs, since they're what make the feature-preprocessing step reusable between training and serving.
- **scikit-learn: model evaluation** — [scikit-learn.org/stable/modules/model_evaluation.html](https://scikit-learn.org/stable/modules/model_evaluation.html) — precision, recall, ROC-AUC, and calibration, covered in more depth than this week's lectures had room for.
- **pandas: `read_sql`** — [pandas.pydata.org/docs/reference/api/pandas.read_sql.html](https://pandas.pydata.org/docs/) — the bridge between SQLAlchemy and DataFrames used throughout Lecture 2. Same tool you used in Week 4.
- **Anthropic: pricing** — [platform.claude.com/docs/en/about-claude/pricing](https://platform.claude.com/docs/en/about-claude/pricing) — current per-token pricing, needed for Homework 4's cost estimate. Prices and model names change over time — always check this page rather than trusting a cached number.

## Concepts worth reading beyond this week

- **Google's "Rules of Machine Learning"** (Martin Zinkevich) — a widely-cited, practically-minded checklist for when and how to bring ML into a production system, written by a Google engineer. Rule #1 ("don't be afraid to launch a product without machine learning") is Lecture 1's four-question framework in one sentence, from a different author, years earlier.
- **NIST AI Risk Management Framework** — a public-sector reference for the same governance ideas covered in Lecture 3 (bias, monitoring, accountability), if you want a more formal treatment than this course's practitioner-level pass.
- **The concept of "label leakage"** — worth a deliberate search beyond this week's brief mention in Lecture 2; it is one of the most common ways a first machine learning project quietly produces misleadingly good test results.

## Tools you already have from earlier weeks

- **PostgreSQL 16+ / `psql`** — Week 3.
- **SQLAlchemy** — Week 4 (this week's `text()` + parameterized-query pattern is identical to Week 4's).
- **Flask** — Week 7 (this week's routes extend that same application).
- **The `app_users` table and RBAC pattern** — Week 9 (every human-in-the-loop endpoint this week reuses it directly).

If any of those feel rusty, this is a good week to skim back rather than push forward confused — everything in Weeks 11 assumes they're solid.
