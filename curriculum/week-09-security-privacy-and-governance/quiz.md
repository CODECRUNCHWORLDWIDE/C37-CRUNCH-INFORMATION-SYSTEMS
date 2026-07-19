# Week 9 — Quiz

Fifteen questions. Lectures closed. Aim for 12/15 before starting Week 10. A mix of multiple-choice and short "what's wrong with this design" questions — the answer key at the bottom explains the *why*, not just the letter.

---

**Q1.** A login form correctly identifies a user and confirms their password. A moment later, that same user tries to `DELETE /api/v1/orders/482`, an action their role doesn't permit. Which concept governs the second check?

- A) Authentication
- B) Authorization
- C) Encryption
- D) Data minimization

---

**Q2.** Why is `bcrypt` (or `argon2`/`scrypt`) preferred over `SHA-256` specifically for hashing passwords?

- A) `SHA-256` isn't a real hash function
- B) `bcrypt` is reversible, which is required for password verification
- C) `bcrypt` is deliberately slow and tunable, which makes brute-force guessing against a leaked database far more expensive; `SHA-256` is fast, which helps an attacker, not you
- D) `SHA-256` cannot process strings, only files

---

**Q3.** A login endpoint returns `404 user not found` for a nonexistent username and `401 wrong password` for an existing username with an incorrect password. What's the security problem with this design?

- A) There is no problem — this is more informative for the user
- B) It allows an attacker to enumerate which usernames are valid, one guess at a time, by watching which status code comes back
- C) `404` should never be used for authentication endpoints
- D) The endpoint should return `500` instead of `401`

---

**Q4.** What does role-based access control (RBAC) enforced only in application code (e.g., a Flask decorator) fail to protect against, that row-level security (RLS) at the database layer does protect against?

- A) RBAC in application code offers no protection at all
- B) A bug in the application's query logic (e.g., a missing `WHERE` clause) that would otherwise leak rows regardless of the role check on the endpoint
- C) RLS is strictly worse than RBAC and should never be used
- D) RBAC and RLS solve exactly the same problem, so only one is ever needed

---

**Q5.** In PostgreSQL row-level security, a policy is defined `USING (customer_id IN (SELECT customer_id FROM customers WHERE region_id = current_setting('app.current_region')::INTEGER))`. What must the application do for this policy to correctly scope a specific sales rep's queries?

- A) Nothing — Postgres infers the region automatically from the JWT
- B) Set the `app.current_region` session variable to that rep's region, after verifying their identity and before running their query
- C) Include the region in every `WHERE` clause manually, in addition to the policy
- D) Grant the rep `SUPERUSER` so the policy can read their session

---

**Q6.** What is the principle of least privilege?

- A) Every user should have the same access level to simplify administration
- B) Every identity — human or service — should have the minimum access required to do its job, no more
- C) Administrators should have unrestricted access to make troubleshooting easier
- D) Access should be granted generously and revoked only after an incident

---

**Q7.** Which of these is the correct way to handle a database credential (`DATABASE_URL`) in a project?

- A) Hardcode it directly in `app.py` so it's easy to find
- B) Commit it to a private GitHub repo — private repos are safe to store secrets in
- C) Store it in a `.env` file that is listed in `.gitignore` before the file is ever created, and never commit it
- D) Email it to teammates who need it, for convenience

---

**Q8.** A company's `.env` file, containing a database password, was accidentally committed to a public repository and then removed in a later commit. Is the credential still exposed?

- A) No — deleting the file in a later commit removes it completely
- B) Yes — Git retains every prior commit's content in its history, so the credential remains recoverable (e.g., via `git log -p`) until it is rotated, regardless of later deletions
- C) Only if someone forks the repository before the deletion
- D) No, because GitHub automatically scrubs deleted secrets from history

---

**Q9.** Under the GDPR-style privacy principle of "purpose limitation," which of these is a violation?

- A) Collecting a customer's email to send an order confirmation, and later using that same email to send unrelated marketing without new consent
- B) Using a customer's shipping address to ship their order
- C) Retaining an order's financial details for a legally required period
- D) Correcting a customer's address after they report it's wrong

---

**Q10.** A customer requests erasure of their personal data ("right to be forgotten"), but the company is legally required to retain financial records of their past transactions for tax purposes. What is the standard resolution?

- A) Refuse the erasure request entirely and do nothing
- B) Hard-delete the customer row, breaking referential integrity in the orders table
- C) Pseudonymize the identifying personal fields while preserving the non-personal transaction facts the legal retention obligation requires
- D) Ignore the tax retention requirement and delete everything

---

**Q11.** What is the difference between "consent" and "contract" as a lawful basis for processing personal data?

- A) They are the same thing under a different name
- B) Contract-basis processing is necessary to fulfill an agreement the person entered into (e.g., shipping their order) and needs no separate opt-in; consent-basis processing (e.g., marketing email) requires an affirmative, specific, revocable opt-in
- C) Consent is only required for European citizens; contract applies everywhere else
- D) Contract basis expires after 30 days; consent basis does not

---

**Q12.** A signup form has one checkbox: "I agree to the Terms of Service," which silently also opts the user into marketing email. What privacy principle does this violate?

- A) Data minimization
- B) Storage limitation
- C) The requirement that consent be specific and separate per purpose — bundling an unrelated purpose (marketing) into an unrelated required action (agreeing to terms) doesn't constitute real consent for the marketing use
- D) Nothing — one checkbox is sufficient as long as it exists

---

**Q13.** In data governance terms, what does "data lineage" describe?

- A) Who is allowed to query a table
- B) How long a table's data is retained before deletion
- C) The path data takes from its origin, through transformations, to where it's ultimately consumed — e.g., an external API, through an ETL job, into a database table, into a report
- D) The physical hardware a database runs on

---

**Q14.** A data catalog row states a table's `owner_role` as `finance`. What does that ownership designation primarily establish?

- A) That only people in finance are allowed to ever query the table
- B) That finance is accountable for the table's accuracy, its access policy, and its lifecycle decisions — the "who do I ask" answer, not a technical access restriction by itself
- C) That the table was created by someone in the finance department
- D) That the table contains only financial data and nothing else

---

**Q15.** In STRIDE threat modeling, an attacker modifying an order's `unit_price` in transit before it reaches the database, without authorization, falls under which category?

- A) Spoofing
- B) Tampering
- C) Repudiation
- D) Denial of service

---

## Answer key

<details>
<summary>Reveal after attempting</summary>

1. **B** — authentication already confirmed identity at login; the second check, deciding whether that identified user's role permits this specific action, is authorization.
2. **C** — bcrypt's deliberate slowness is the point: it makes offline brute-force attacks against a leaked hash database expensive, while a legitimate one-time login check stays fast enough to be unnoticeable.
3. **B** — a distinguishable response for "no such user" vs. "wrong password" enables username enumeration; a single generic error for both closes that leak.
4. **B** — RBAC at the application layer only gates which endpoints a role can call; it does nothing if the endpoint's own query logic has a bug that returns rows it shouldn't. RLS enforces the row filter inside the database itself, so it holds regardless of application bugs.
5. **B** — the policy reads a Postgres session variable (`app.current_region`) that the application must explicitly set for that connection, after authenticating the user and before running their query.
6. **B** — least privilege means the minimum access necessary for the job, applied to every identity, not "everyone gets the same access" or "give admins everything."
7. **C** — a `.env` file excluded via `.gitignore` before it's created is the standard safe pattern; "private repo" and "email it" are both still real exposure paths, and hardcoding is worse than either.
8. **B** — Git preserves full history; a later "delete" commit does not erase the credential from earlier commits, which remain fully recoverable until the credential itself is rotated.
9. **A** — using data collected for one purpose (order confirmation) for an unrelated purpose (marketing) without a new lawful basis is exactly what purpose limitation prohibits.
10. **C** — pseudonymization preserves the required financial facts while removing the personal identifiers, resolving the conflict between erasure and legal retention without breaking referential integrity or violating either obligation.
11. **B** — contract-basis processing flows directly from an agreement's necessary fulfillment and needs no separate consent; consent-basis processing (like marketing) requires its own specific, revocable opt-in.
12. **C** — bundling an unrelated purpose into a required checkbox means the person never made a real, specific choice about the marketing use — which fails the requirement that consent be purpose-specific.
13. **C** — lineage traces data's path from origin through transformation to final consumption, answering "where did this number come from" and "what breaks if the source changes."
14. **B** — ownership is an accountability designation (who to ask, who decides lifecycle changes), not by itself a technical access-control mechanism — that's a separate RBAC/RLS grant, even though the two are usually aligned.
15. **B** — unauthorized modification of data (here, a price, in transit or in storage) is the defining scenario of the Tampering category.

</details>

**Scoring:** 12+ → start Week 10. 9–11 → re-read the lecture sections behind your misses, especially the RBAC-vs-RLS and consent-vs-contract distinctions if that's where you lost points. <9 → re-read all three lectures from the top; this week's controls are the ones a real production system is judged on first.
