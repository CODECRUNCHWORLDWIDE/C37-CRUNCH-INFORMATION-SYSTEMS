# Week 6 — Quiz

Fifteen questions. Lectures closed. Aim for 12/15 before starting Week 7. A mix of multiple-choice and short reasoning — the answer key at the bottom explains the *why*, not just the letter.

---

**Q1.** What is the primary purpose of an ERP system?

- A) To manage a company's social media presence
- B) To run the operational and financial backbone of a company — inventory, purchasing, finance, and related resources — as one shared system
- C) To replace all human decision-making with automation
- D) To store only customer contact information

---

**Q2.** Which of these is a CRM concern rather than an ERP concern?

- A) Inventory stock levels
- B) A sales opportunity's pipeline stage before it becomes an order
- C) A supplier's purchase-order lead time
- D) The general ledger's account balances

---

**Q3.** According to the lectures, why is *integrating* ERP and CRM harder than building either one alone?

- A) It isn't harder — buying both from the same vendor solves it automatically.
- B) The two systems disagree about timing, ownership, and whether two records represent the same real-world entity, and someone has to resolve that continuously.
- C) CRM systems can't technically connect to a database.
- D) ERP systems don't support APIs.

---

**Q4.** Which is the correct one-sentence test for whether a table is master data?

- A) If it has more than 100 rows, it's master data.
- B) If it was created more than a year ago, it's master data.
- C) If it would still make sense to keep after deleting every transaction, it's master data.
- D) If a manager can see it in a dashboard, it's master data.

---

**Q5.** In this week's schema, `purchase_order_items` is:

- A) Master data — it describes what a supplier is capable of producing
- B) Transactional data — each row exists only inside one specific purchase event
- C) Neither — it's a lookup table
- D) The same thing as `product_suppliers`

---

**Q6.** A company has two `customers` rows for the same real company, entered independently by two reps. What does Lecture 2 call the underlying problem class?

- A) Referential integrity violation
- B) Master-data duplication
- C) A normalization failure (1NF violation)
- D) A transactional data conflict

---

**Q7.** Why does Lecture 2 recommend flagging an inactive supplier (`is_active = FALSE`) instead of deleting its row?

- A) Deleting is technically impossible in PostgreSQL.
- B) Deleting the row would break every historical purchase order that references it via foreign key, destroying real transaction history.
- C) Flagging is faster to type than `DELETE`.
- D) There's no real difference; either approach is fine.

---

**Q8.** "Single source of truth" for a piece of master data means:

- A) Only one system in the company may ever store that data, in any form.
- B) There is a pre-agreed, authoritative answer for which system wins when two systems disagree about that data.
- C) The data never changes once entered.
- D) Every employee must manually re-enter the data into every system that uses it.

---

**Q9.** Which best describes **data ownership** as covered in Lecture 2?

- A) A purely technical database permissions setting
- B) An organizational assignment of accountability for a piece of master data's accuracy to a specific role or team
- C) Legal ownership of the company's servers
- D) The department that pays for the software license

---

**Q10.** Order-to-cash, in the correct order, is:

- A) Invoice → Sales order → Opportunity → Shipment → Payment
- B) Opportunity/Lead → Sales order → Fulfillment/Shipment → Invoice → Payment
- C) Shipment → Opportunity → Payment → Invoice → Sales order
- D) Sales order → Payment → Opportunity → Shipment → Invoice

---

**Q11.** In this week's data, why don't a Won opportunity's `est_value` and its matched order's actual summed value usually match exactly?

- A) It's a data-entry bug that should always be fixed to match exactly.
- B) `est_value` is a forecast made before the deal closed; the actual order reflects the real, finalized product mix and quantities.
- C) `est_value` is always double the real order value by design.
- D) Orders and opportunities are unrelated and coincidentally similar.

---

**Q12.** What is a "three-way match" in procure-to-pay?

- A) Comparing three different suppliers' prices before choosing one
- B) Verifying that the purchase order, the goods receipt, and the vendor's invoice all agree before payment is approved
- C) Matching an order to exactly three product line items
- D) A CRM feature that matches leads to sales reps

---

**Q13.** A "Won" CRM opportunity with `won_order_id IS NULL` should be treated as:

- A) Normal and expected — nothing to investigate
- B) A likely process break worth investigating — the deal is marked closed in CRM but hasn't yet produced a real ERP transaction
- C) Proof that the CRM system is broken and should be replaced
- D) Evidence the customer never actually bought anything

---

**Q14.** In the build-vs-buy framing from Challenge 1, why can a "buy" option that looks cheaper in Year 1 end up more expensive over a 5-year horizon?

- A) License costs always double every year automatically.
- B) Recurring costs — support fees (often a percentage of license), and ongoing customization maintenance — compound annually and are easy to under-weight against a single big Year 1 implementation number.
- C) Buy options never include support at all.
- D) Building in-house is always cheaper in every scenario, so the comparison is pointless.

---

**Q15.** `product_suppliers` (product ↔ supplier, many-to-many, with an `is_preferred` flag) exists in this week's schema mainly to model:

- A) That a single product can be sourced from more than one supplier, with one designated as preferred and others as backups
- B) That a supplier can only ever produce one product
- C) The shipping address for each order
- D) The commission a sales rep earns per product

---

## Answer key

<details>
<summary>Reveal after attempting</summary>

1. **B** — an ERP is the shared operational/financial backbone: inventory, purchasing, finance, and related resource tracking, as one integrated system.
2. **B** — a pipeline stage before a deal closes is CRM's territory; the other three (inventory, PO lead time, GL balances) are all ERP-module concerns.
3. **B** — integration is hard because of disagreement about timing, ownership, and entity matching between systems — not a technical API limitation.
4. **C** — the practical test from Lecture 2: master data would still make sense to keep if every transaction were deleted tomorrow.
5. **B** — `purchase_order_items` rows exist only as part of one specific purchase event; they're transactional, referencing master data (`product_id`) but not describing an enduring entity themselves.
6. **B** — this is exactly what Lecture 2 calls master-data duplication: two rows for one real-world entity, created independently.
7. **B** — deleting would violate foreign-key history; every purchase order that references the supplier would lose a valid link to real past transactions. Flagging preserves history while signaling the current status.
8. **B** — SSOT is about having a pre-agreed authoritative answer when systems disagree, not a restriction on which systems may store a copy.
9. **B** — data ownership, per Lecture 2, is an organizational accountability assignment — a role or team responsible for a table's/column's accuracy — not a technical permission setting.
10. **B** — Opportunity/Lead → Sales order → Fulfillment/Shipment → Invoice → Payment is the order-to-cash chain from Lecture 3.
11. **B** — the opportunity's estimated value is a pre-close forecast; the order reflects the actual, finalized deal, so some variance is normal and expected, not a bug.
12. **B** — a three-way match compares the PO, the goods receipt, and the vendor invoice before approving payment — the core procure-to-pay financial control.
13. **B** — this specific gap (Won in CRM, nothing yet in ERP) is exactly the process break Lecture 3 flags as worth investigating, and exactly what Week 7's integration work exists to close.
14. **B** — recurring costs (percentage-based support fees, ongoing customization maintenance) compound every year and are easy to underweight against one large, visible Year 1 number — this is the whole point of doing a multi-year TCO model instead of comparing sticker prices.
15. **A** — `product_suppliers` models the real many-to-many relationship between products and the suppliers who can produce them, with a preferred/backup distinction.

</details>

**Scoring:** 12+ → start Week 7. 9–11 → re-read the lecture sections behind your misses. <9 → re-read all three lectures from the top; this week's concepts (master data, SSOT, process tracing) are the backbone of Weeks 7–10.
