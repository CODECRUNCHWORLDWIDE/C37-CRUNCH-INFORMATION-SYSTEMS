# Challenge 2 — Model a New Location

> **Time:** ~1 hour. **Uses:** everything so far, plus a genuine judgment call that foreshadows data modeling in Week 3.

## The scenario

Riverbend is opening a second café location across town — same company, same roastery and warehouse, but a second retail counter with its own staff and its own POS terminal. Nora needs the information-systems register updated to reflect the new location **before** it opens, not after something breaks because nobody thought it through.

## Part A — Shared vs. duplicated (25 min)

This is the actual challenge: for each component below, decide whether the new location **shares** the existing component (one row serves both locations) or needs its **own, duplicated** component (a second row, distinct from the original). Write one sentence of justification for each — "obviously" is not a justification.

1. Elena Vasquez, Head Roaster.
2. The Roast Profile Controller.
3. Green Coffee Inventory.
4. A Barista Lead (currently just Devon Park, at the original location).
5. A Square POS Terminal.
6. POS Transaction (the data).
7. Roast Batch Log.
8. Nora Chen, Owner/GM.

**Hint for the hard ones:** ask "does this thing exist once, physically, in the world — or does opening a second physical location necessarily create a second physical instance of it?" A roaster (and the person running it) is one central facility serving both counters. A card reader is bolted to a specific counter and cannot be in two places.

## Part B — Model it (25 min)

Based on your Part A answers, write the `INSERT` statements to add the new location's **genuinely duplicated** components (not the shared ones — don't re-insert Elena or the roaster) plus the flows connecting them into the existing system. At minimum:

1. The new location's Barista Lead (people) and POS Terminal (technology).
2. A `department` value that distinguishes the new location's rows from the original — e.g. `Retail-Downtown` vs. `Retail-Riverside`, or similar. This matters: if both locations' POS Transactions share one identical department label, you've made it impossible to later query "how is the new location doing on its own," which defeats half the point of opening it.
3. The flow(s) connecting the new POS Terminal to a POS Transaction data component — decide (and justify in `notes.md`) whether the new location's transactions belong in the *same* `POS Transaction` row-type or need their own — this is the shared-vs-duplicated question applied to data specifically, not just to technology.
4. A flow showing how the new location's retail sales eventually reach Nora Chen for visibility — should it be a new flow, or does it reuse the *pattern* of existing flow `12` (POS Transaction → Nora) with a new source? Justify your choice.

## Part C — Name the risk if you get it wrong (10 min)

Pick **one** component from Part A that you called "shared" and write two to three sentences describing what would actually go wrong if someone on the team incorrectly duplicated it instead (created a second, separate row when one shared row was correct). Then do the reverse: pick one you called "duplicated" and describe what goes wrong if someone incorrectly shares it.

## Deliverable

`notes.md` with your Part A table (component, shared/duplicated, one-sentence justification) and Part C write-up. `week01-solutions.sql` with your Part B `INSERT` statements.

## Check yourself

- Is your reasoning in Part A based on "is this one physical/logical thing or two," or did you default to "duplicate everything to be safe"? Over-duplicating is a real cost (double data entry, numbers that drift apart) — it's not automatically the safe choice.
- Did your new `department` values (Part B.2) actually let you write a query that isolates just the new location's data? Test it: `SELECT * FROM is_components WHERE department LIKE '%Downtown%';` (or whatever naming you chose) should return only new-location rows.
- This exact shared-vs-duplicated judgment call is the core question Week 3 (data modeling) formalizes with primary keys and foreign keys — you've just done it by hand a week early.
