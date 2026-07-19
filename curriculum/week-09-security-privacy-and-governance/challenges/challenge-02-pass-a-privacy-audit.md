# Challenge 2 — Redesign a System to Pass a Privacy Audit

Below is a description of **"QuickCart,"** a small e-commerce system built by a team that never took this week's lectures. It works, and it is not compliant with even the basics of the privacy principles from Lecture 2. You are the engineer brought in ahead of a customer's procurement security review, which includes a standard privacy audit checklist. Your job: find every violation, then redesign the schema, code, and policy to pass.

**Estimated time:** 1.5 hours.

## QuickCart, as built today

```sql
CREATE TABLE quickcart_customers (
    id             SERIAL PRIMARY KEY,
    full_name      TEXT,
    email          TEXT,
    password       TEXT,             -- stores what the user typed at signup, verbatim
    phone          TEXT,
    date_of_birth  DATE,             -- collected at signup "in case we need it later"
    newsletter     BOOLEAN DEFAULT TRUE,   -- every new signup starts opted in
    created_at     TIMESTAMP DEFAULT now()
);
```

```python
# api.py — every route
@app.get("/api/customers")
def list_customers():
    # no auth check at all — anyone who finds the URL can call this
    rows = db.execute("SELECT * FROM quickcart_customers").fetchall()
    return jsonify([dict(r) for r in rows])   # includes the password column, in the response

@app.delete("/api/customers/<id>")
def delete_customer(id):
    # a customer asked support to delete their account; support ran this directly:
    db.execute("DELETE FROM quickcart_customers WHERE id = %s", (id,))
    # orders table has customer_id foreign keys pointing at this row — no one checked
```

Other known facts about QuickCart, gathered from the team during handoff:

- The `.env` file with the database password was committed to the public GitHub repo eight months ago, and nobody has rotated the database password since.
- There is no logging of who queried what — if data were breached, there would be no way to know when it started or what was taken.
- Marketing exports the full `quickcart_customers` table (including `date_of_birth` and `phone`) to a spreadsheet every month to run ad campaigns through a third-party tool, with no record of which customers ever agreed to that.
- There is no documented retention period for anything — the founder's stated policy is "we just never delete stuff."

## The audit checklist you must pass

A real (simplified) privacy-audit checklist, item by item:

1. Are passwords stored using a proper one-way hash, never plaintext or reversible encryption?
2. Is personal data readable only by identities with a specific, justified need — enforced by access control, not convention?
3. Is there a distinct, specific, revocable consent record for each separate purpose data is used for (e.g., marketing vs. account operation)?
4. Is unnecessary personal data collection avoided (data minimization)?
5. Can the system fulfill an access request (export everything held about one person) and a rectification request?
6. Can the system fulfill an erasure request without breaking referential integrity or destroying records another law requires it to keep?
7. Is there a documented, enforced retention period for personal data, with an end-of-life mechanism?
8. Are secrets (credentials, keys) excluded from version control, and rotated after any known or suspected exposure?
9. Is there a way to detect and reconstruct what happened in the event of a breach (logging / audit trail)?

## Tasks

1. **Score QuickCart against the checklist**, item by item, in `audit-findings.md` — Pass / Fail / Partial, with the specific evidence from the code or facts above that justifies each score. (You should find it fails or partially fails nearly every item — that's the intended starting state.)

2. **Redesign the schema.** Write a corrected `CREATE TABLE` for `quickcart_customers` that fixes the password storage, removes or properly justifies `date_of_birth`, and adds the consent-tracking columns Lecture 2 describes (separate from any single "I agree" flag).

3. **Redesign the two API routes** shown above so they pass items 2 and 8 of the checklist — proper auth, no password field ever serialized into a response, no secret in source control.

4. **Write the erasure procedure** QuickCart should follow instead of the raw `DELETE` shown above, matching Lecture 2's pseudonymization pattern and explicitly addressing the orphaned-foreign-key problem in the original code.

5. **Write a one-paragraph retention policy** for `quickcart_customers`, with a specific period, justification, and mechanism — replacing "we just never delete stuff."

6. **Re-score your redesign** against the same nine-item checklist in a second table at the bottom of `audit-findings.md`, and for any item that's still not a clean Pass, state honestly what's still missing and what it would take to close it.

## What "great" looks like

- Every "Fail" in the original scoring cites the *exact* line of code or fact that causes it — not a general "this is bad practice."
- The redesigned schema and routes are complete enough to actually run, not pseudocode with gaps.
- The erasure procedure explicitly explains why a hard `DELETE` was wrong here and what it would have broken.
- The re-score is honest — if something genuinely can't be fully fixed without infrastructure this challenge doesn't require you to build (e.g., full audit logging), say so plainly rather than claiming a Pass you haven't earned.

## Deliverable

`audit-findings.md` (both score tables), `quickcart_schema_v2.sql`, `api_v2.py` (the two corrected routes), and `erasure-procedure.md` in your portfolio.
