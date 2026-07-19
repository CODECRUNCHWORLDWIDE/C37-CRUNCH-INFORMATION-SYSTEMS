# Challenge 1 — Threat-Model the System and Rank Risks

Threat-model the full Crunch Cycles system as it exists after Weeks 7–9: a Flask REST API (Week 7), deployed on a cloud server reachable from the internet (Week 8), backed by a PostgreSQL database with RBAC and row-level security (Week 9), holding real customer, employee, and financial data. Not every possible attack deserves equal attention — the actual skill this challenge grades is **prioritization**: separating the handful of risks that genuinely matter from the long tail that doesn't, this week.

**Estimated time:** 1.5 hours.

## Background: STRIDE

**STRIDE** is a structured way to brainstorm threats by category, so you don't just list the three attacks you happen to have read about recently:

| Letter | Threat category | Plain-English question |
|---|---|---|
| **S** | Spoofing | Can someone pretend to be a user or system they're not? |
| **T** | Tampering | Can someone modify data or code without authorization? |
| **R** | Repudiation | Can someone deny having done something, with no evidence to contradict them? |
| **I** | Information disclosure | Can someone see data they shouldn't? |
| **D** | Denial of service | Can someone make the system unavailable to legitimate users? |
| **E** | Elevation of privilege | Can someone gain more access than they were granted? |

Each is a lens, not a checklist to fill mechanically — the goal is to walk each major component of the system (API, database, login flow, deployment) through all six lenses and see what surfaces.

## Tasks

1. **Draw or list the system's assets and trust boundaries.** At minimum: the Flask API (Week 7/8), the Postgres database, the `app_users` table specifically, the JWT signing key, the `.env` file on the server, and the network boundary between "the public internet" and "your server." A trust boundary is any point where data crosses from a less-trusted zone to a more-trusted one (e.g., an HTTP request arriving from the internet, about to touch your database).

2. **Apply STRIDE to at least four components** (the login endpoint, the `/api/v1/orders` endpoint, the database connection itself, and the deployment/server from Week 8). For each component × category combination that produces a real, specific threat (not every combination will), write one row:

   | Component | STRIDE category | Specific scenario | Likelihood (1–5) | Impact (1–5) | Risk score | Mitigation |
   |---|---|---|---|---|---|---|

   Example row, to calibrate specificity — yours should be this concrete, not vaguer:

   | `/api/v1/login` | Spoofing | An attacker with a leaked username list runs a credential-stuffing attack (trying passwords from other breaches) against real usernames | 4 | 4 | 16 | Rate-limit login attempts per IP and per username; alert on 10+ failures in 5 minutes |

   Aim for **10–15 total rows** across the four components. Reuse the categories that genuinely apply per component — not every component needs all six.

3. **Score and rank.** Use `Risk score = Likelihood × Impact` (1–25 scale). Sort your table by risk score descending.

4. **Write a "top 5, this sprint" section.** Of everything you found, pick the five highest-scored, genuinely actionable risks and write one sentence each on the specific mitigation you'd implement first and why it's the highest-leverage fix available — cite this week's lectures (RLS, hashed passwords, secrets management, RBAC) where they directly apply.

5. **Write an "accepted risk, for now" section — at least 2 items.** Not everything gets fixed this sprint. Pick two real risks from your table that you're deliberately *not* addressing yet, and state why: too low-likelihood for a system this size, too expensive relative to the impact, or genuinely out of scope for a course project (e.g., "a nation-state actor targeting our Flask app specifically" is a real STRIDE-derivable threat and a genuinely irrational thing to spend a week defending against for a small business's order system). Naming what you're *not* doing, and why, is as much a sign of security judgment as the fixes themselves.

## What "great" looks like

- Scenarios are specific enough that someone reading only your table could reproduce the attack in their head — not "hacker steals data," but the exact request, exact weakness, exact consequence.
- Likelihood and impact scores are individually defensible, not just picked to make the ranking come out a certain way.
- At least one row references a control from **this week's lectures specifically** (bcrypt hashing, RLS, JWT expiry, `.env`/secrets hygiene) as either the mitigation already in place or the mitigation you're proposing.
- The "accepted risk" section shows genuine judgment about proportion, not laziness — a one-line "not doing this because it's low value for the cost" is fine when it's true and stated honestly.

## Deliverable

`threat-model.md` in your portfolio with: the asset/trust-boundary list, the full STRIDE table (10–15 rows, ranked), the top-5 mitigation plan, and the accepted-risk section.
