# Exercise 3 — Data, Information, Knowledge, Wisdom

> **Time:** ~1.5 hours. **Uses:** Lecture 3 §1–§3. Works against `is_components`; you'll also write plain-English answers.

Lecture 3 argued that raw data is nearly worthless until a process climbs it up to a decision. This exercise makes you do that climb yourself, twice — once guided, once on your own — and forces you to be precise about which value-chain activity and which efficiency/effectiveness axis you're talking about.

## Part A — Climb the ladder (guided) (30 min)

Here are three raw **data** facts from Riverbend's POS Transaction log for one Tuesday:

- `09:14, 1 flat white, $5.25, card`
- `09:16, 1 flat white, $5.25, card`
- `09:19, 1 flat white, $5.25, cash`

For this set of facts, write out all four DIKW layers, matching Lecture 3 §1's table format:

1. **Data:** (already given above — just restate it as one combined observation)
2. **Information:** aggregate the three rows into something a human would actually say out loud. (Hint: what do all three transactions have in common, in a five-minute window?)
3. **Knowledge:** *why* might that pattern be happening? You're allowed to hypothesize — state your hypothesis clearly as a hypothesis, not a fact.
4. **Wisdom:** what specific, concrete action would Devon Park or Nora Chen actually take because of it? (A real decision — "monitor it" is not a decision, "keep two flat-white shots pre-pulled between 9:00–9:30" is.)

## Part B — Climb the ladder (on your own) (30 min)

Pick **one** of these two raw-data starting points and repeat the full four-layer climb:

**Option 1:** Over the last three wholesale orders, two different cafés both requested a delivery date change after their order was already confirmed.

**Option 2:** Of Riverbend's last 40 subscription renewals (flow `17`/`18` in your seed), 6 failed to charge on the first attempt.

Write your own four-layer table. For the **wisdom** layer, use the "gap → consequence → impact → cost → value" pattern from Lecture 3 §5 to phrase your recommended action as a one-paragraph argument, not just a bullet.

## Part C — Value chain + efficiency/effectiveness tagging (30 min)

For your Part B scenario:

1. Which value-chain activity does it sit in — primary or support, and which specific one (inbound logistics, operations, outbound logistics, marketing & sales, service, or an infrastructure/support function)? Justify in one sentence, referencing Lecture 3 §2's table.
2. Is the underlying problem primarily an **efficiency** problem, an **effectiveness** problem, or both? Justify with the specific definitions from Lecture 3 §3 — don't just assert it.
3. Query your register for one piece of supporting evidence — e.g., if you picked Option 1, find the process and technology components involved:

```sql
SELECT category, name, department
FROM is_components
WHERE department IN ('Wholesale', 'Digital')
ORDER BY category, name;
```

Adjust the `department` filter to match whichever option you picked, and reference the returned components by name in your write-up.

## Deliverable

- `notes.md` — your Part A table, Part B table + wisdom paragraph, and Part C answers (1–2).
- `week01-solutions.sql` — your Part C query (adjusted for your chosen option).

## Check yourself

- In Part A, is your "information" layer just a rephrasing of the data, or does it actually add context/comparison? If it's just a rephrase, redo it — that's the single most common mistake at this layer.
- In Part B, does your "wisdom" layer name a specific action with a specific trigger, or is it vague ("investigate further")? Vague wisdom isn't wisdom yet — tighten it.
- Could you defend your value-chain classification (Part C.1) to someone who disagreed with you? If not, you may have picked the activity that *sounds* right rather than the one that's actually right — reread Lecture 3 §2's table and try again.
