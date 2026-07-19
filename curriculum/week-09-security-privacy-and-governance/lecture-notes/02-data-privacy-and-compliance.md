# Data Privacy & Compliance — Principles, Personal Data, Consent, and What Regulation Actually Requires

Lecture 1 secured *access* — who can reach which rows. This lecture is about the rows themselves: `customers.email`, `customers.contact_name`, `employees.email`. Those columns describe real people who never agreed to have their information cross-examined by whoever happens to have database credentials. Privacy law — the EU's **GDPR** (General Data Protection Regulation) is the clearest and most widely copied version, but California's CCPA, Brazil's LGPD, and dozens of others share its shape — turns that ethical obligation into specific, checkable requirements. This lecture teaches the principles in a form you can actually implement, using Crunch Cycles' own schema as the worked example.

**A scope note, stated plainly:** this lecture teaches the *engineering* implications of privacy principles that are genuinely useful regardless of which jurisdiction you operate in. It is not legal advice, and "does GDPR apply to my specific company" is a question for an actual lawyer, not a course README. What follows is what a competent engineer should be able to build once a lawyer says "yes, we need this" — and, often, what you should build anyway, because "we don't sell people's data to strangers who ask nicely" is a good default independent of any statute.

## What counts as personal data

**Personal data** is any information that relates to an identified or identifiable natural person. That's a wider net than people expect on first read:

- **Obviously personal:** name, email, phone number, home address, government ID number.
- **Personal because it's linkable:** an IP address, a device ID, a customer ID that maps 1:1 to a named person in another table — none of these look like "someone's name," but each one lets you re-identify a specific person, which is all the definition requires.
- **Sensitive personal data** (a stricter subcategory in most frameworks): health data, racial or ethnic origin, religious belief, sexual orientation, biometric data, precise geolocation history. This category typically requires a higher bar to collect and process at all, and extra protection once you do.

Walking the Crunch Cycles schema: `customers.contact_name`, `customers.email`, `customers.city`, `customers.country` are personal data — they identify a real human contact at a business customer. `employees.email`, `employees.first_name`/`last_name` are personal data about your own staff (privacy law protects employees too, not just customers). `products`, `regions`, and `order_items.unit_price` are not personal data — nothing there identifies a person. `orders.customer_id` is a bridge: not personal data on its own, but it's the key that lets anyone with `orders` and `customers` join the two and produce personal data, which matters when you're deciding what a report or an export is allowed to include.

## The core principles, made concrete

GDPR names seven principles; five of them are the ones that actually change how you build a system. Each gets a direct Crunch Cycles translation.

| Principle | What it means | Crunch Cycles, concretely |
|---|---|---|
| **Lawful basis** | You need a specific, identifiable legal reason to process someone's personal data — it can't just be "we felt like it." | Storing a customer's email to fulfill their order is lawful basis **"contract"** — you need it to do the thing they asked for. Emailing them a marketing newsletter is a *different* processing activity needing its own basis: **consent**. |
| **Purpose limitation** | Data collected for one stated purpose can't silently be repurposed for another. | The `customers.email` collected to send order confirmations cannot, without a new lawful basis, be sold to a third-party ad network. |
| **Data minimization** | Collect only what you actually need for the stated purpose — not "might be useful someday." | Crunch Cycles' checkout form should not ask for a customer's date of birth; nothing in the order-fulfillment purpose needs it. If sales analytics wants birth-date-based segmentation later, that is a new purpose requiring a new, explicit justification — not a field added "just in case." |
| **Storage limitation** | Don't keep personal data longer than the purpose requires. | An order's financial record has a legitimate long retention (tax law, in most jurisdictions, requires 5–7 years). A customer's *marketing consent preferences* for a newsletter they never subscribed to has no such justification for indefinite storage — this is Lecture 3's retention policy, applied at the column level. |
| **Accuracy** | Personal data should be correct and kept up to date, with a way for the subject to fix it. | A "right to rectification" isn't abstract — it's a concrete `PATCH /customers/42` endpoint your API from Week 7 already has the shape for; Lecture 1's RBAC decides who's allowed to call it. |

## Lawful basis, specifically

GDPR names six lawful bases; three cover almost everything a system like Crunch Cycles does:

- **Contract** — processing necessary to perform a contract with the person (fulfilling an order, sending a shipping confirmation). No separate consent checkbox needed; the transaction itself is the basis.
- **Consent** — the person affirmatively opted in, for a specific stated purpose, and can withdraw it as easily as they gave it. This is the basis for marketing email, not for order fulfillment.
- **Legal obligation** — you're required to keep or report the data by another law (tax records, employment records).

The practical engineering consequence: **don't let one checkbox cover two different purposes.** A signup form with a single "I agree to the Terms" checkbox that silently also opts someone into marketing email conflates contract-basis processing (creating the account) with consent-basis processing (marketing) — and makes consent meaningless because it wasn't a real, specific choice. Model them as separate facts:

```sql
ALTER TABLE customers
    ADD COLUMN marketing_consent      BOOLEAN NOT NULL DEFAULT FALSE,
    ADD COLUMN marketing_consent_date TIMESTAMPTZ,
    ADD COLUMN consent_source         TEXT;   -- e.g. 'signup_form_v3', 'sales_call_2024-11-02'

-- Recording consent is an explicit, timestamped action, not a default:
UPDATE customers
SET marketing_consent = TRUE,
    marketing_consent_date = now(),
    consent_source = 'signup_form_v3'
WHERE customer_id = 42;
```

Two details that show up in every real audit: consent must be **as easy to withdraw as to give** (a working "unsubscribe," not a support ticket you never answer), and you must be able to **prove when and how** consent was given if challenged — which is exactly what `marketing_consent_date` and `consent_source` are for. "We assumed everyone wants our newsletter" is not consent; it's the absence of it.

## Data subject rights — what a person can ask you for, and how you fulfill it

Privacy regulation gives the person the data is about ("the data subject") specific, actionable rights. Each maps to something your system must be able to actually *do*, not just a policy paragraph:

| Right | What the person can ask for | How Crunch Cycles fulfills it |
|---|---|---|
| **Access** | "Send me a copy of everything you hold about me." | A query joining every table that references `customer_id`, exported as JSON or CSV — the export script in Exercise 2. |
| **Rectification** | "This information is wrong; fix it." | `PATCH /customers/42` — already in your Week 7 API's shape, gated by Lecture 1's RBAC so only the customer (or an authorized employee) can call it. |
| **Erasure** ("right to be forgotten") | "Delete my personal data." | Not always a literal `DELETE` — see the conflict below. |
| **Portability** | "Give me my data in a format I can take to a competitor." | The same export as Access, in a structured, machine-readable format (JSON/CSV, not a PDF screenshot). |
| **Restriction / objection** | "Stop processing my data for [purpose] while we sort out a dispute," or "stop using it for marketing specifically." | A `marketing_consent = FALSE` flag that the send job checks before every campaign; a `processing_restricted` flag that read paths must respect. |

## The erasure conflict — and why pseudonymization is usually the right answer

Erasure sounds simple until it collides with storage limitation's mirror image: **you're often legally required to keep the data**, not delete it. A customer who orders bikes and then requests erasure still generated real invoices, and tax law in most jurisdictions requires you to retain financial records for years. Literally deleting `customers` row 42 would also orphan every `orders` row referencing it — breaking the very financial records you're required to keep.

The standard resolution is **pseudonymization**: replace the identifying fields with irreversible placeholders while preserving the non-personal facts (the transaction happened, for this amount, on this date) that other obligations require you to keep.

```sql
-- Fulfilling an erasure request for customer 42 while preserving order history integrity
UPDATE customers
SET contact_name = 'ERASED-' || customer_id,
    email         = 'erased-' || customer_id || '@deleted.invalid',
    city           = NULL,
    marketing_consent = FALSE,
    marketing_consent_date = NULL,
    consent_source = NULL
WHERE customer_id = 42;

-- orders and order_items rows are untouched — the financial record survives,
-- but it no longer identifies a real person by name or contactable email.
```

Document this decision explicitly wherever your privacy policy or governance doc addresses erasure: "we pseudonymize rather than hard-delete records subject to a legal retention obligation, and hard-delete everything else." That single sentence, decided in advance, is the difference between a calm, defensible response to an erasure request and a scramble the day one actually arrives.

## Breach notification — the clock that starts the moment you find out

If personal data is exposed — a stolen laptop with an unencrypted database export, a misconfigured cloud storage bucket, a SQL injection that dumped `customers` — GDPR requires notifying the relevant regulator within **72 hours** of becoming aware of the breach, and notifying affected individuals "without undue delay" if the breach poses a high risk to them. Two engineering consequences follow directly from that deadline:

1. **You need to be able to detect a breach at all** — logging who accessed what, and alerting on anomalies (a service account suddenly running `SELECT * FROM customers` with no `LIMIT`, at 3 a.m., is exactly the kind of signal that turns "we never knew" into "we caught it in six hours"). This is precisely what row-level security and query logging from Lecture 1 give you for free, beyond just blocking bad access — they also make it observable.
2. **You need to know what data was actually exposed**, fast — which is only possible if you already know, ahead of time, what personal data lives in which table and column. That inventory is Lecture 3's data classification and governance work, and it is not optional busywork: it is the thing that turns a 72-hour deadline from impossible into merely urgent.

## GDPR vs. other frameworks, briefly

You will meet other names in the wild — know roughly what distinguishes them, without needing depth on any but GDPR here:

- **CCPA/CPRA (California)** — similar spirit (access, deletion, opt-out of "sale" of data), narrower in some ways, broader in others (e.g., an explicit right to opt out of data *sales*, a concept GDPR doesn't name the same way).
- **LGPD (Brazil)**, **PIPEDA (Canada)**, **POPIA (South Africa)** — each closely modeled on GDPR's structure, because GDPR became the de facto template regulators reach for.
- **HIPAA (US, healthcare)** — a different, sector-specific regime for health data specifically, with its own separate rules on top of general privacy law — mentioned so you don't confuse "we did a GDPR-style privacy review" with "we are HIPAA compliant," which are not the same claim.

The practical takeaway: build the *capabilities* (classify personal data, honor access/erasure/rectification requests, minimize what you collect, document lawful basis, detect and report breaches) once, well, and generically. Which specific law requires which specific capability, for which specific users, is a legal question — but a system that already has all five capabilities is compliant with nearly anything a lawyer later tells you applies.

## What's next

You now know what personal data *is* and what you owe the people it describes. Lecture 3 turns that into standing policy: who owns each table, how long each column is actually kept, how you'd measure whether the data is even trustworthy, and how you'd trace a number in a report back to the system it came from.
