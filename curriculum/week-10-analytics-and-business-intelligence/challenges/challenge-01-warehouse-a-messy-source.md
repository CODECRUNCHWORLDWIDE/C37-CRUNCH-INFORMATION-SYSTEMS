# Challenge 1 — Warehouse a Messy Operational Source

**Time:** ~90 minutes. **Difficulty:** Hard. **Code required.**

## The scenario

Crunch Cycles just acquired a small regional distributor and inherited its legacy CRM export — a flat table of "customers" nobody has cleaned since it was exported from a system that's now been shut off. Lecture 1 §3 called a warehouse **integrated**: data from multiple sources combined into one consistent model. This challenge is that word, made real — and messy, the way a real acquisition's data always is.

Load this into `crunchcycles_dw.raw` exactly as given — don't clean it on the way in; `raw` mirrors the source untouched, per Lecture 1 §5:

```sql
CREATE TABLE raw.legacy_crm_customers (
    row_id          SERIAL PRIMARY KEY,
    company_name    TEXT,
    contact_name    TEXT,
    email           TEXT,
    city            TEXT,
    country_raw     TEXT,           -- inconsistent free-text country field, on purpose
    signup_date_raw TEXT,           -- inconsistent date formats, on purpose — NOT a DATE column
    loaded_at       TIMESTAMP NOT NULL DEFAULT now()
);

INSERT INTO raw.legacy_crm_customers
    (company_name, contact_name, email, city, country_raw, signup_date_raw)
VALUES
('Trailhead Bikes',      'Jen Ortiz',      'JEN@trailheadbikes.com',   'Denver',    'USA',            '01/10/2021'),
('Trailhead Bikes Inc.', 'Jennifer Ortiz', 'jen@trailheadbikes.com',  'Denver',    'U.S.A.',         '2021-01-10'),
('Northern Gear Co',     'Mark Bell',      'mark@northerngear.ca',    'Toronto',   'Canada',         '05/22/2020'),
('Alpine Sport Supply',  'Hans Weber',     'hans@alpinesport.de',     'Munich',    'Germany',        '19-07-2018'),
('Le Velo Parisien',     ' Claire Dubois', 'claire@levelo.fr',        'Paris ',    'France',         '2020/09/08'),
('nordic cycle club',    'Erik Solberg',   'erik@nordiccycle.no',     'Oslo',      'Norway',         '25.06.2021'),
('Sydney Spokes',        'Amy Clarke',     'amy@sydneyspokes.au',     'Sydney',    'Australia',      '2020-12-01'),
('Sydney Spokes Pty',    'Amy J. Clarke',  'amy@sydneyspokes.au',     'Sydney',    'AU',             '01-12-2020'),
('Mumbai Mountain Bikes','Ravi Shah',      'ravi@mumbaimtb.in',       'Mumbai',    'India',          '02/09/2023'),
('Harborview Cycles',    'Tom Reyes',      'tom@harborviewcycles.com','Vancouver', 'Canada',         '11/14/2023'),
('Kiwi Trail Traders',   'Nia Watts',      NULL,                       'Auckland',  'New Zealand',    '2024-02-01'),
('Kiwi Trail Traders',   'Nia Watts',      'nia@kiwitrail.co.nz',     'Auckland',  'NZ',             '2024-02-01'),
('Desert Trail Cycles',  'Nina Cole',      'nina@deserttrail.com',    'Phoenix',   'United States',  '19/09/2023');
```

Notice, deliberately: **three kinds of mess**, matching three real acquisition-data problems:

1. **Duplicate customers, spelled differently** — "Trailhead Bikes" (already in your `dim_customer` from Exercise 2) appears again as "Trailhead Bikes Inc.", with a slightly different contact name but the *same* email once you normalize case. "Sydney Spokes" similarly duplicates. "Kiwi Trail Traders" duplicates *within this file itself*, once with a `NULL` email.
2. **Inconsistent country spellings** — `USA`, `U.S.A.`, `United States` are the same country, three ways; `Canada` is consistent (a control, so you don't over-correct); `AU`/`Australia` and `NZ`/`New Zealand` are abbreviation-vs-full-name pairs.
3. **Inconsistent date formats** — `MM/DD/YYYY`, `YYYY-MM-DD`, `DD-MM-YYYY`, `YYYY/MM/DD`, and `DD.MM.YYYY` all appear, several of them genuinely ambiguous (`01/10/2021` — January 10th, or October 1st?) without an external clue.

## Your task

Write `clean_and_load_legacy_customers.py` (or a well-commented `.sql` file, your call) plus `cleaning-notes.md` documenting every decision. Specifically:

**1. Normalize country names.** Build a mapping (a `CASE` expression or a small lookup table) that resolves every `country_raw` value in the sample to one canonical name matching `warehouse.dim_customer.country`'s existing values (`USA`, `Canada`, `Germany`, `France`, `Norway`, `Australia`, `India`, `New Zealand`). State, in `cleaning-notes.md`, how you'd handle a country spelling you *haven't* seen before and don't have a mapping for — silently guess, silently drop the row, or flag it for a human? Justify your choice.

**2. Resolve the ambiguous dates.** For each of the five date formats present, state which one you assumed and why. For the genuinely ambiguous case (`01/10/2021` for Trailhead Bikes) — cross-reference: the same customer's *other* row gives an unambiguous `2021-01-10`. Use that to resolve the ambiguous one, and say explicitly in your notes that this is exactly the kind of judgment call an automated pipeline usually **can't** make safely without a human-reviewed rule, and why you were able to here (a second, unambiguous data point for the same real-world entity).

**3. Deduplicate.** Decide a matching rule (exact email match after lowercasing/trimming is the strongest signal here; company-name-plus-city is a weaker fallback for the `NULL`-email Kiwi Trail Traders row) and merge each duplicate group into **one** clean row. Document your matching rule precisely enough that someone could apply it to next month's export without asking you what "duplicate" meant.

**4. Load only the genuinely new customers** into `warehouse.dim_customer` — the ones that don't already match an existing customer by your Task 3 rule (Harborview Cycles, Kiwi Trail Traders, and Desert Trail Cycles's duplicate check against the *existing* `dim_customer` — is Desert Trail Cycles already there from Exercise 2? Check before assuming). For customers that **do** already exist (Trailhead Bikes, Sydney Spokes), do **not** insert a second row — but do note in `cleaning-notes.md` whether any of the legacy record's fields (e.g., a more complete contact name) should have updated the existing warehouse row, and whether that's a Type 1 or Type 2 change per Lecture 2 §5.

## Expected result (spot checks)

- `SELECT COUNT(*) FROM warehouse.dim_customer WHERE company_name ILIKE '%trailhead%'` returns **1**, not 2, after your load.
- `SELECT DISTINCT country FROM warehouse.dim_customer` shows no `U.S.A.`, `USA`/`United States` duplication, no `AU`/`Australia` duplication — every country appears exactly once in its canonical form.
- The two "Kiwi Trail Traders" rows collapsed into one, with the email preserved from whichever row had one (not silently lost by an unlucky merge).

## Done when…

- [ ] `clean_and_load_legacy_customers.py` (or `.sql`) runs and loads only genuinely new customers.
- [ ] `cleaning-notes.md` documents your country-mapping rule, your date-resolution reasoning for the ambiguous case, and your deduplication rule — precisely enough for someone else to apply the same rules to a new batch of legacy data.
- [ ] Re-running your script a second time does not create a second copy of the newly-loaded customers either — the idempotency lesson from Exercise 2 applies here too.
- [ ] You explicitly stated what your pipeline does with a country spelling or date format it's never seen before (Task 1) — "I didn't think about that" is not an acceptable answer for this bullet.

## Why this matters

Every real company's "second data source" looks exactly like this — not clean, not consistent with your existing keys, and full of duplicates that are obvious to a human eye and invisible to a naive `INSERT`. The skill this challenge tests isn't SQL syntax; it's the judgment to notice the mess, make a defensible call about it, and write down the call so the next person (or the next month's data) doesn't have to guess what you did.

## Submission

Commit `clean_and_load_legacy_customers.py` (or `.sql`) and `cleaning-notes.md` to your portfolio under `c37-week-10/challenge-01/`.
