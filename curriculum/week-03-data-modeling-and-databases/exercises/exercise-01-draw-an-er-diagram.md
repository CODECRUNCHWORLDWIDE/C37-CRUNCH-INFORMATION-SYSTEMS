# Exercise 1 — Draw an ER Diagram for a Library

**Goal:** Run the modeling process from Lecture 1 on a brand-new domain, start to finish, without CrunchRide as a crutch. By the end, extracting entities and cardinality from a plain-English brief feels like a repeatable procedure, not a one-off trick.

**Estimated time:** 1 hour.

## The brief

Here is the domain, described the way a library director would actually describe it to you:

> "We run a small public library. We have books — each book has a title, an author, an ISBN, and a number of copies we own. Members join the library with a name, email, and a membership start date. A member can borrow a copy of a book; we call that a loan — we need to know which member borrowed which book, when it was checked out, and when it's due back (always 21 days later), and when it actually came back (if it has). A book can be borrowed by many different members over its life, and a member can have many loans over time, but at any given moment we only have as many copies checked out as we physically own. We also want to track which books are currently 'on hold' for a member waiting for a copy to free up — a hold has a member, a book, and the date the hold was placed. Finally, each book belongs to exactly one category (Fiction, Non-Fiction, Reference, Children's), and each category just has a name and a shelf location."

## Tasks

Work through the process from Lecture 1, Section 6, in order. Write your answers in `er-diagram.md`.

1. **Circle the nouns.** List every candidate entity you find in the brief. For each one, state whether you accepted or rejected it as an entity, and why — one sentence per noun is enough. *(You should end up with 5 entities. If you have significantly more or fewer, re-read the brief.)*

2. **List attributes.** For each accepted entity, list its attributes, and mark which one you're choosing as the primary key (natural or surrogate — state which, and why, for at least two of the five).

3. **Classify every relationship.** For each pair of entities that relate to each other, state the cardinality (1:1, 1:N, or M:N) in a sentence read from *both* ends, the way Lecture 1 modeled `Station — Bike`. You should find at least 5 relationships.

4. **Resolve the many-to-many.** At least one relationship in this brief is genuinely many-to-many. Identify it, and decide: does it need to become its own entity (because there's a fact attached to the pairing), or is a plain junction table with no extra attributes enough? Justify your answer using the same test from Lecture 1 ("is there a fact worth recording about the pairing itself?").

5. **Draw the diagram.** Using crow's-foot notation (ASCII, hand-drawn and photographed, or a tool like dbdiagram.io — see [resources.md](../resources.md)), draw every entity as a box with its attributes and primary key marked, every relationship as a line, and cardinality symbols at **both** ends of every line.

## Hints

<details>
<summary>On the "copies" detail</summary>

Re-read the sentence about copies carefully: *"at any given moment we only have as many copies checked out as we physically own."* This is a **business rule**, not necessarily a new entity — you don't need a `Copy` entity unless you decide to track *individual physical copies* separately (copy #1 of "Dune" vs copy #2). A simpler, equally valid model treats `copies_owned` as an attribute of `Book` and enforces the "can't loan more than we own" rule as application logic or a more advanced constraint (not required for this exercise — just don't let the sentence trick you into inventing an entity you don't need). If you *do* want to model individual copies, that's also a defensible, slightly more detailed answer — either is acceptable as long as you state which choice you made and why.

</details>

<details>
<summary>On "Hold" vs. "Loan"</summary>

These look similar (member + book + a date) but they mean different things — a hold is a *request* for a future loan, not a loan itself. Should they be the same entity with a status flag, or two separate entities? Either is defensible; state your reasoning. A hint toward one answer: what happens to a hold once the loan actually happens — does the row transform, or does a new row get created and the old one closed out?

</details>

## Done when…

- [ ] `er-diagram.md` lists all 5 entities with your accept/reject reasoning for at least 3 rejected or borderline nouns.
- [ ] Every entity has a clearly marked primary key.
- [ ] Every relationship states cardinality read from both ends (not just "1:N" with no explanation).
- [ ] The many-to-many relationship is explicitly identified and your junction-vs-own-entity decision is justified in one or two sentences.
- [ ] A diagram exists (ASCII, photo, or tool export) with cardinality symbols on every line at both ends.

## Stretch

- Add a `Fine` concept: a member who returns a book late owes a fine calculated from days-late × a daily rate. Where does this fit — a new entity, or attributes on `Loan`? Justify it.
- What changes about your model if the library has **multiple branches**, and a copy physically lives at one specific branch?

## Submission

Commit `er-diagram.md` (plus your diagram image, if hand-drawn) to your portfolio under `c37-week-03/exercise-01/`.
