# Challenge 1 — Model a Real Domain from an Interview Transcript

**Time:** ~90 minutes. **Difficulty:** Medium–Hard. **No single right answer.**

## The scenario

Real requirements don't arrive as a tidy paragraph like Exercise 1's library brief. They arrive as a transcript of someone talking about their business — full of asides, contradictions, things they clearly haven't thought through yet, and details they think are obvious but aren't written down anywhere. Your job is to build a clean ER model out of that noise, and — just as importantly — to write down the open questions you'd need to ask before you'd trust the model enough to build on it.

This is the single most valuable skill this week teaches. Nobody hands a systems designer a normalized brief. They hand you forty-five minutes of a business owner talking, and you have to leave that room with a model good enough to start building.

## The transcript

This is Dr. Amara Osei, owner of a small veterinary clinic, talking to you during a discovery interview:

> "So basically we're a small animal clinic — dogs, cats, the occasional rabbit, one time a guy brought in a bearded dragon. We've got three vets on staff right now, me included, and two vet techs who assist. Clients bring their pets in for appointments — could be a checkup, a vaccination, surgery, whatever. Each appointment is with one specific vet, obviously, and it's for one pet — sometimes a client has multiple pets and books back-to-back appointments for each one, but that's just multiple appointment records, not one appointment for two pets, if that makes sense.
>
> Every pet has an owner — well, actually, sometimes a pet has two owners, like a married couple, and either one can bring the pet in or pick up prescriptions, and honestly this has caused problems before because our old system only let us put one phone number on file and if it's the other spouse's number we don't recognize it when they call. So that's something I really want fixed.
>
> We track medical records per pet — vaccinations, past diagnoses, medications they're on. Every appointment generates a record: what was the reason for the visit, what was the vet's diagnosis, what treatment or prescription came out of it. Prescriptions... a pet can be on multiple prescriptions at once, refills need to be tracked, and some meds need a specific dosage that changes over time as the pet gets older or heavier, which is annoying because our current spreadsheet just has one 'dosage' column that people just overwrite every time, so we've lost history on that more than once.
>
> Oh — and we also do boarding. If a client goes on vacation, they can board their pet with us for a stretch of days. That's totally separate from appointments, it's not a vet visit necessarily, though sometimes we do give meds during a boarding stay, which I guess counts as an appointment too? I'm honestly not sure how that should work, you tell me.
>
> Billing is its own headache. Every appointment has a cost, boarding has a per-day rate, and clients either pay on the spot or we invoice them monthly if they're on our subscription wellness plan — which is a thing some clients pay for, it covers a certain number of checkups a year at a discount. Not everyone's on it though, so I don't want you assuming every client has a plan."

## Your task

Produce `vet-clinic-model.md` with three sections:

### 1. Entities and relationships

Extract the ER model, same rigor as Exercise 1: entities, attributes, primary keys, and every relationship's cardinality read from both ends. You should find real many-to-many relationships here (at minimum: pets and owners) that the library brief didn't force you to confront as directly.

### 2. Open questions

List **at least 5** concrete, specific questions you would need to ask Dr. Osei before you'd be confident building this schema. A weak open question is vague ("what about billing?"). A strong one is precise and shows you already have a candidate answer in mind that you want confirmed or corrected — for example: *"You mentioned meds given during a boarding stay 'might count as an appointment' — should a boarding stay be able to reference zero-or-more appointments that happened during it, or should medication-during-boarding be its own separate record type? I'd lean toward the first because it reuses the appointment→prescription relationship you already described, but want your read."*

At minimum, address: the two-owners-per-pet situation, the boarding/appointment overlap, the dosage-history problem, and the wellness-plan-is-optional detail — but don't stop at exactly four; find at least one more the transcript is quietly ambiguous about.

### 3. Assumptions you made anyway

For anything you couldn't reasonably ask about (this is a written exercise, not a live interview), state the assumption you made to move forward, and flag it explicitly as an assumption — not a confirmed fact. This is exactly the discipline Challenge 1 in Week 1 (ambiguous business questions, if you took C33) and every real client conversation demands: ship a defensible interpretation, but never hide that it *is* an interpretation.

## Hints

<details>
<summary>On owners (multiple owners per pet)</summary>

This is a textbook many-to-many: a pet can have multiple owners, and a person can own multiple pets (or none, if they're only an authorized pickup contact — is that the same thing as an "owner"? That's exactly the kind of question worth asking). The relationship itself might carry attributes — e.g., "primary contact" flag, or a phone number specific to that person — which is your cue (same test as Lecture 1) for whether the junction needs its own identity beyond two foreign keys.

</details>

<details>
<summary>On the dosage-history problem</summary>

Dr. Osei explicitly told you the *current* system's flaw: overwriting one `dosage` column loses history. The fix is a modeling one — dosage isn't a fact about the *prescription* (a fixed thing), it's a fact about the prescription **at a point in time**. That's a strong hint toward a separate entity: something like `dosage_changes` or `prescription_adjustments`, each row an immutable historical fact, rather than a mutable column. This is worth a sentence in your write-up: recognizing "this should be a history of records, not a column that gets overwritten" is a modeling instinct worth having permanently.

</details>

## How success is judged

| Signal | Weak answer | Strong answer |
|---|---|---|
| Noise-filtering | Models every aside literally, including irrelevant ones (the bearded dragon) | Extracts the entities and rules that matter, ignores color commentary |
| Cardinality | States 1:N/M:N without justification | Justifies each from what Dr. Osei actually said, quoting or paraphrasing the relevant line |
| Open questions | Vague, generic questions | Specific, shows a candidate answer already in mind, tied to an exact ambiguity in the transcript |
| Assumption discipline | Silently picks an interpretation | Explicitly flags every assumption as an assumption |
| The dosage insight | Treats dosage as a plain column | Recognizes it needs to be a history, not a mutable field |

## Submission

Commit `vet-clinic-model.md` (plus a diagram, if you draw one) to your portfolio under `c37-week-03/challenge-01/`.
