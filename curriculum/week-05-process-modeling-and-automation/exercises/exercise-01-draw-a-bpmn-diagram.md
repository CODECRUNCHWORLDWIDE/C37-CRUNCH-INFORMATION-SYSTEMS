# Exercise 1 — Draw a BPMN Diagram for Onboarding

**Goal:** Take a raw, spoken process description — the kind you'll actually be handed on the job — and turn it into a precise BPMN diagram with the right tasks, gateways, events, and swimlanes. This is the same skill Lecture 1 walked you through, applied to a brand-new process you haven't seen modeled yet.

**Estimated time:** 60 minutes.

## The scenario

CrunchRide is hiring more field technicians as the fleet grows. Right now, onboarding a new technician is entirely word-of-mouth and sticky notes — exactly the kind of process this course exists to make precise. You interviewed the Operations Manager, Sofia Nakamura, and here's what she told you, transcribed close to verbatim:

> "Okay so when we hire someone, first thing is HR sends them an offer and once they sign it, that kicks everything off. HR runs the background check — that's required for anyone handling company vehicles, no exceptions — and if it comes back clean, HR notifies IT to set up their accounts: email, the technician app login, building badge. If the background check comes back with a flag, HR has to review it manually and decide whether to still move forward — that's not automatic, someone actually has to look at it and make a call. Assuming it's clean and IT's got the accounts ready, two things happen at the same time: IT ships them their tablet and toolkit, and separately their assigned Field Trainer — that's usually one of our senior techs — schedules a two-day ride-along. The new tech can't actually start solo repair work until BOTH the equipment has arrived AND they've completed the ride-along — we've had bad experiences skipping either one. Once both of those are done, the Field Trainer signs off, and that's when the new tech gets flipped to 'active' in the system and can start getting ticket assignments for real."

## Your task

1. **Identify the participants.** List every lane you'll need (hint: there are at least four — think about who does what: the new technician themselves is one, even though they mostly just... exist as an input to other people's work).

2. **Identify the events.** What's the start event, and what's its trigger? How many end events are there — and is there really only one way this process can end?

3. **Identify every task**, and for each, decide its type (user, manual, or service) and which lane it belongs in. Watch for tasks that are secretly two tasks hiding under one description.

4. **Identify every gateway**, and for each, decide its type (exclusive, parallel, or inclusive). There are at least two gateways in this transcript — one obvious, one easy to miss on a first read.

5. **Draw the full diagram.** Hand-drawn, [bpmn.io](https://demo.bpmn.io/), or a Mermaid flowchart (see Lecture 1's worked example for the syntax) — any is fine. Label every task, event, and gateway condition clearly.

6. **Write a short "modeling notes" section** (4–6 sentences) explaining any judgment calls you made — anywhere the transcript was ambiguous and you had to decide what Sofia probably meant. (There's at least one genuinely ambiguous spot — where exactly does the background-check-flagged path rejoin the main flow, if at all? The transcript doesn't say. State your assumption.)

## Hints

<details>
<summary>On finding the parallel gateway</summary>

Re-read the sentence starting "Assuming it's clean and IT's got the accounts ready, two things happen at the same time." That's your parallel split. What are the two branches? And where do they join back together — what's the condition described for the new tech to be allowed to start solo work?

</details>

<details>
<summary>On the flagged background check</summary>

This is genuinely underspecified in the transcript — Sofia never says what happens if HR's manual review decides *not* to move forward, or what happens if they decide to proceed anyway after a flag. A strong answer states this gap explicitly rather than inventing detail Sofia never gave you. That's a legitimate, valuable thing to flag back to a real stakeholder rather than silently guessing.

</details>

## Done when…

- [ ] Your diagram has at least four lanes, correctly assigned (new technician, HR, IT, Field Trainer at minimum).
- [ ] The start event names its actual trigger (not just "process begins").
- [ ] You've correctly identified and typed both gateways (one exclusive, one parallel) — and drawn a matching join for the parallel split.
- [ ] Every task is labeled with its type (user/manual/service).
- [ ] Your "modeling notes" section explicitly names the ambiguous spot in the transcript and states your assumption.

## Stretch

- Sofia mentioned the background check is "required for anyone handling company vehicles, no exceptions." Add this as an annotation or note on your diagram — BPMN supports text annotations for exactly this kind of business-rule context that doesn't fit neatly into a task or gateway label.
- Redraw the diagram assuming CrunchRide outsources the background check to a third-party vendor. What changes — does a new pool appear? A message flow?

## Submission

Commit your diagram (image, `.bpmn` file, or Mermaid `.md`) plus your modeling notes to your portfolio under `c37-week-05/exercise-01/`.
