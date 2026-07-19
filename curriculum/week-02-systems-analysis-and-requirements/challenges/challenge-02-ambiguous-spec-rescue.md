# Challenge 2 — Ambiguous Spec Rescue

**Time:** ~60 minutes. **Difficulty:** Medium. **No single right answer.**

## The scenario

A junior developer on Grace Kim's IT team was handed exactly one line, pulled from an old email thread, and told "build this for the next sprint":

> "The system should let store staff quickly check on special orders and notify customers when their stuff arrives."

That's it. No stakeholder list, no acceptance criteria, no priority. This is precisely the kind of spec that produces a working demo nobody actually wanted — because it technically satisfies the sentence while missing almost everything a real analysis would have surfaced. Your job is to do the analysis that should have happened *before* that line was ever written, using only what you already know about Meridian from this week (the seeded stakeholders, elicitation notes, and everything the lectures covered) plus reasonable, explicitly-stated assumptions.

## Your task

Produce `challenge-02.md` containing all of the following.

### 1. Interrogate the one-liner

Break the sentence into its separate hidden requirements — there are at least four distinct ideas buried in it ("check on special orders," "quickly," "notify customers," "when their stuff arrives"). For each, list the specific ambiguity: what question would you have asked in an interview (per Lecture 1's laddering) if you'd been in the room when this was written?

### 2. Identify who's missing

The sentence mentions "store staff" and "customers" but this week's elicitation surfaced at least three more legitimate stakeholders with a stake in this exact feature. Name them and, for each, say in one sentence what requirement they'd add or constrain that the one-liner completely omits. (Hint: re-read `elicitation_sessions` — someone's entire job is fielding calls about exactly this, and nobody consulted them.)

### 3. Rewrite it as a full requirement set

Produce **at least 5 requirement statements** (mix of functional and non-functional, per Lecture 2's template), each with a MoSCoW priority and a one-line justification for the priority. At least one must directly address something the original sentence didn't mention at all but that a competent analysis would catch (e.g., what happens when the "notify" channel fails — a bounced email, a disconnected phone).

### 4. Model it as a use case

Write one use case (per Lecture 3's format — main flow, at least one alternate flow, at least one exception flow) for the core interaction the sentence implies. Choose the primary actor deliberately and defend the choice in one sentence — it's not as obvious as it first looks.

### 5. Write the "Won't" list

Name at least 2 things a stakeholder might reasonably assume this feature includes, that you are explicitly **not** building in this pass — and why. This is the MoSCoW "Won't" discipline from Lecture 2, applied to protect the sprint from silent scope creep.

## Constraints

- You may not simply say "this is too vague, send it back" — that's a correct instinct for a real project, but the exercise is to demonstrate you *can* extract a workable spec from a bad one, because in the real world you will sometimes have to.
- Every assumption you make must be flagged as an assumption, in a dedicated section, not buried in prose as if it were a confirmed fact.
- Reuse the Meridian stakeholders and elicitation notes already in your database rather than inventing a new scenario — the whole point is applying this week's real material, not writing fiction from scratch.

## Hints

<details>
<summary>On "quickly" (the vaguest word in the sentence)</summary>

"Quickly" could mean fast to load, few steps to reach, or fast to *learn* for someone who's never used the system. Those three produce different requirements (a performance NFR, a usability NFR about click-count, and a training-related NFR respectively) — and a strong answer picks the one(s) actually supported by this week's evidence (Lecture 1's observation: 22 minutes and multiple handoffs for one order) rather than guessing at random.

</details>

<details>
<summary>On the missing stakeholder</summary>

Tom Alvarez's entire elicitation note is about exactly this feature, and he isn't mentioned in the one-liner at all. What would he need that "store staff" and "customers" alone would never surface?

</details>

## How success is judged

| Signal | Weak answer | Strong answer |
|--------|-------------|----------------|
| Interrogation | Accepts the sentence at face value | Surfaces the 4+ hidden ambiguities explicitly |
| Missing stakeholders | Only staff + customers | Finds the call-center (and other) gaps |
| Requirement quality | Restates the sentence with "shall" added | New, testable content the original never specified |
| Use case | Happy path only | Includes a real alternate and exception flow |
| Scope discipline | No "Won't" list, or a vague one | Specific, defensible exclusions |

## Submission

Commit `challenge-02.md` to your portfolio under `c37-week-02/challenge-02/`.
