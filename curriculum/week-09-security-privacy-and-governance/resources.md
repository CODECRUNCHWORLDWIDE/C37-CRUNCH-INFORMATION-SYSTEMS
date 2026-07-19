# Week 9 — Resources

Curated, not exhaustive. Everything here is free. Install what you need before Monday; read the rest as you go, referenced from the lectures.

## Install first

### Python security libraries

```bash
pip install bcrypt pyjwt python-dotenv
```

- **Verify:** `python3 -c "import bcrypt, jwt; print('ok')"`
- **bcrypt PyPI page + docs:** <https://pypi.org/project/bcrypt/>
- **PyJWT docs:** <https://pyjwt.readthedocs.io/>
- **python-dotenv docs:** <https://pypi.org/project/python-dotenv/>

### PostgreSQL 16+ (already installed from earlier weeks)

- **Row-level security reference:** <https://www.postgresql.org/docs/current/ddl-rowsecurity.html> — read this before Lecture 1's RLS section if any part of `CREATE POLICY` feels unfamiliar.
- **`GRANT` reference:** <https://www.postgresql.org/docs/current/sql-grant.html>
- **`CREATE ROLE` reference:** <https://www.postgresql.org/docs/current/sql-createrole.html>

## Official documentation and standards (the primary sources — read these over any blog post)

### Access control and authentication

- **OWASP Authentication Cheat Sheet:** <https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html> — the industry-standard checklist for building login systems correctly.
- **OWASP Password Storage Cheat Sheet:** <https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html> — why bcrypt/argon2/scrypt, and current recommended work-factor settings.
- **OWASP Top 10 (the standard list of the most critical web application security risks):** <https://owasp.org/www-project-top-ten/> — skim this once; "Broken Access Control" is consistently #1 or #2 and is exactly what this week's RBAC/RLS work defends against.
- **NIST Digital Identity Guidelines (SP 800-63B), password policy section:** <https://pages.nist.gov/800-63-3/sp800-63b.html> — the authoritative source for modern password-policy guidance (it explicitly argues against forced periodic password rotation and arbitrary complexity rules, which surprises people who learned older advice).

### Privacy and data protection

- **GDPR full text (official EU source):** <https://gdpr-info.eu/> — searchable, article-by-article; Articles 5 (principles), 6 (lawful basis), 15–22 (data subject rights), and 33 (breach notification) are the ones this week's lecture draws from directly.
- **ICO (UK regulator) "Guide to the UK GDPR":** <https://ico.org.uk/for-organisations/uk-gdpr-guidance-and-resources/> — the most readable plain-English regulator guidance available, written for practitioners rather than lawyers.
- **CCPA/CPRA official California AG page:** <https://oag.ca.gov/privacy/ccpa> — for comparing GDPR's shape against the US's most prominent state-level equivalent.

### Data governance

- **DAMA-DMBOK (Data Management Body of Knowledge) summary** — the standard reference framework data governance practices are drawn from; search "DAMA DMBOK data governance" for accessible summaries if the full book isn't available to you.
- **OpenLineage project (open standard for data lineage):** <https://openlineage.io/> — worth a skim to see what automated lineage tracking looks like at a scale beyond one Markdown diagram.

### Threat modeling

- **Microsoft's STRIDE overview:** <https://learn.microsoft.com/en-us/azure/security/develop/threat-modeling-tool-threats#stride-model> — the original source of the STRIDE framework used in Challenge 1.
- **OWASP Threat Modeling guide:** <https://owasp.org/www-community/Threat_Modeling> — broader context on when and how to threat-model, beyond just STRIDE.

## From this course

- [Week 1 — Information Systems Foundations](../week-01-information-systems-foundations/) and [Week 3 — Data Modeling & Databases](../week-03-data-modeling-and-databases/) — the schema and constraint fundamentals this week's RLS policies and classification tables build directly on top of.
- [Week 7 — Integration & APIs](../week-07-integration-and-apis/) — the Flask API and API-key auth this week's login flow and RBAC decorators replace and extend.
- [Week 8 — Cloud Architecture](../week-08-cloud-architecture/) — the deployed, internet-reachable server that makes this week's controls non-optional rather than academic.
- [C33 · Crunch SQL](../../../C33-CRUNCH-SQL/) — if `GRANT`, roles, or `CREATE POLICY` syntax feels unfamiliar at the SQL level, that course's earlier weeks are the ideal companion.

## A note on AI-assisted security work

AI tools can draft a plausible-looking auth flow, RLS policy, or privacy policy in seconds — and a plausible-looking security control is exactly the dangerous kind, because a subtle mistake (a policy that doesn't actually restrict what you think it does, a "generic error" that still leaks timing information) often *looks* correct until someone tests it adversarially. Use such tools, if at all, to draft a first pass — then test every control the way Exercise 1 and the mini-project require: log in as the *lowest*-privileged role you built and try, deliberately, to see or do something that role shouldn't be able to. Security controls you haven't tried to break are controls you don't actually know work.
