# Week 9 — Homework

Extra practice, spaced across the week rather than crammed into one sitting. None of this repeats the exercises or challenges verbatim — new domains and new angles on the same skills, so the judgment generalizes instead of you memorizing one worked example.

**Estimated total time:** 5 hours across the week (roughly 1h/day, Monday–Friday, alongside the day's other work).

## 1. AuthN vs. AuthZ in the wild (45 min)

For each real-world scenario below, state whether it's an authentication failure, an authorization failure, or both — and what specifically should have caught it.

1. A hotel guest uses their key card to enter a room that isn't theirs, because the front desk reissued the card without deactivating the old guest's copy.
2. An employee logs into a company system correctly with their own credentials, then edits another department's payroll records — something their role was never supposed to permit.
3. A phishing email tricks an employee into typing their real password into a fake login page, and the attacker logs in successfully afterward.
4. A hospital's patient portal lets any logged-in patient view any other patient's records by simply changing the ID number in the URL.

Write 2–3 sentences per scenario naming the failure and the specific control (from this week's lectures) that would have prevented it.

## 2. Hash a password by hand, then break your own toy version (1h)

1. Using Python's `bcrypt`, hash the string `"correcthorsebatterystaple"` twice, independently. Confirm the two hashes are different strings even though the input was identical — explain in one sentence why, referencing what bcrypt embeds in its output.
2. Now do the same experiment with plain `hashlib.sha256(b"correcthorsebatterystaple").hexdigest()`, twice. Confirm the two outputs are identical.
3. Write 3–4 sentences explaining, using this exact experiment as your evidence, why the SHA-256 behavior is a security weakness for passwords specifically (rainbow tables, cross-database matching) even though SHA-256 is a perfectly good hash function for other purposes (e.g., verifying a downloaded file hasn't been corrupted).

## 3. Design RLS policies for a new table (1h)

Crunch Cycles is adding a `support_tickets` table:

```sql
CREATE TABLE support_tickets (
    ticket_id     SERIAL PRIMARY KEY,
    customer_id   INTEGER NOT NULL REFERENCES customers(customer_id),
    assigned_to   INTEGER REFERENCES app_users(user_id),
    subject       TEXT NOT NULL,
    status        TEXT NOT NULL,   -- 'open' | 'in_progress' | 'closed'
    created_at    TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

Business rule: a support agent should only see tickets assigned to them; a support *manager* role (new — assume it exists) should see every ticket. Write the `ALTER TABLE ... ENABLE ROW LEVEL SECURITY` statement and both `CREATE POLICY` statements this requires, following Lecture 1's pattern exactly. State what session variable or mechanism the API needs to set for the agent-scoped policy to know *which* agent is asking.

## 4. Classify a table you didn't build (45 min)

Pick any real app you use regularly (a food delivery app, a banking app, a fitness tracker) and, from memory/observation (not by looking at its actual schema, which you don't have), list 8–10 fields you're confident it stores about you. For each, classify: personal data (Y/N), sensitivity tier (none/low/medium/high), and one plausible lawful basis (contract, consent, or legal obligation) for why they collect it. Flag any field where you genuinely can't think of a purpose that justifies collecting it — that's the data-minimization instinct this week is meant to build.

## 5. Retention period research (45 min)

Pick one real regulatory retention requirement relevant to a business (tax records, employment records, medical records — pick whichever is easiest to find authoritative information on in your country) and find the actual minimum retention period required by law. Write 3–4 sentences citing what you found and where, and compare it to the 7-year figure this week's lectures used for Crunch Cycles' financial records — is 7 years conservative, accurate, or too short for what you found?

## 6. One governance policy for something outside work (45 min)

Write a miniature governance policy — owner, retention, one quality check, one lineage sentence — for a personal dataset you actually maintain: a budget spreadsheet, a fitness log, a personal project's data. It should take the same shape as Lecture 3's Crunch Cycles catalog rows, just for something real to you. This is the fastest way to feel why governance isn't corporate box-checking — it's the same discipline that keeps *any* dataset trustworthy over time, including your own.

---

## Submission

Commit your answers as `homework.md` to your portfolio under `c37-week-09/homework/`. Partial answers are fine — the goal is repetitions across new domains, not a perfect score.
