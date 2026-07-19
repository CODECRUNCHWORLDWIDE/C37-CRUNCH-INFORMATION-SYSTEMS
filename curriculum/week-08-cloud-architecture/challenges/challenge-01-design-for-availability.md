# Challenge 1 — Design for Availability

**Time:** ~90 minutes. **Difficulty:** Medium-hard. **No single right answer.**

## The scenario

Crunch Cycles shipped its cloud deployment (Exercise 2, and soon the mini-project) the simple way: one app instance, one database instance, one zone. It works. Then your VP of Engineering asks a question that has ended more "it works on my machine" launches than any bug ever has:

> "What happens to Crunch Cycles if the data center our app is running in has a bad afternoon?"

Your job: answer that question honestly for the **current** (single-zone) architecture, then redesign the system so the honest answer to "what happens if one zone dies" becomes "customers might notice a few seconds of elevated latency during failover, and nothing else."

## Your task

Write `challenge-01.md` containing three sections.

### Section 1 — Failure-mode walkthrough of the current architecture

Walk through, step by step, exactly what happens if Zone A (where everything currently runs) becomes unreachable:

1. What happens to in-flight requests?
2. What happens to new requests?
3. What happens to the database — is data lost, or just unreachable? (These are different failures with different fixes — be precise.)
4. How would the team even *find out* it happened, in this current setup? (Be honest if the answer is "a customer complains.")
5. How long would recovery realistically take, and what manual steps would someone have to perform?

### Section 2 — The redundant redesign

Redesign the architecture using Lecture 3's redundancy pattern (multi-zone app instances behind a health-checked load balancer, database primary + standby with automatic failover) as your baseline, but make it **specific to Crunch Cycles**, not a generic copy of the lecture diagram:

1. Draw the redesigned architecture (ASCII or a [Mermaid](https://mermaid.js.org/) diagram — either is fine, both render in GitHub).
2. Explicitly identify **every remaining single point of failure** in your redesign, if any — DNS, a shared object-storage bucket, a third-party payment API dependency, etc. A strong answer doesn't claim zero SPOFs; it names the ones that remain and explains why they're accepted (cost, complexity, or genuinely out of your control) rather than pretending they don't exist.
3. Confirm your app tier is actually stateless under this design — if Week 7's Flask app has any in-memory state (check: does it use Flask sessions stored server-side? A local cache? An in-process rate limiter?), say what you'd change to make horizontal failover safe.

### Section 3 — Re-run the failure-mode walkthrough

Repeat Section 1's five questions against your **redesigned** architecture. The contrast between the two walkthroughs — same five questions, very different answers — is the actual deliverable. If your redesign's answers to questions 3 and 5 don't meaningfully improve over Section 1, the redesign isn't done yet.

## Constraints

- Ground every claim in a specific mechanism (a health check, a promoted standby, a load balancer's retry behavior) — "it would just handle it" is not an answer.
- You do not need to actually deploy the redundant version for this challenge (the mini-project deploys a real system, single-zone is fine there given free-tier constraints) — this is a design and reasoning exercise.
- State any assumption about your specific cloud provider's automatic-failover behavior — providers differ in how automatic vs. manual database failover is, and a strong answer says which behavior it's assuming and why.

## Hints

<details>
<summary>On "how would the team find out" (Section 1, Q4)</summary>

If your honest answer is "a customer complains," that's not a failure on your part — it's an accurate diagnosis of the current setup, and naming it plainly is exactly what a strong answer does. The fix (mentioned but not required to build here) is a health check plus an alerting rule that pages someone the moment the check starts failing, not the moment a customer notices.

</details>

<details>
<summary>On remaining single points of failure (Section 2, Q2)</summary>

DNS is a classic one — even a perfectly redundant app and database are unreachable if the DNS provider itself is down. So is a payment processor (Stripe) — if it's down, checkout is down, and no amount of Crunch Cycles' own redundancy fixes that. Naming these honestly, rather than only listing the ones you fixed, is what separates a strong answer from a shallow one.

</details>

## How success is judged

| Signal | Weak answer | Strong answer |
|--------|-------------|----------------|
| Failure-mode precision | "The app would go down" | Distinguishes in-flight vs. new requests, data loss vs. unreachability, detection vs. recovery |
| Redesign specificity | Redraws the lecture's generic diagram unchanged | Names Crunch Cycles' actual components and states provider-specific assumptions |
| SPOF honesty | Claims the redesign has zero remaining single points of failure | Names the ones that remain (DNS, third-party dependencies) and justifies accepting them |
| Statelessness check | Assumed without checking | Actually inspects whether Week 7's Flask app has server-side state and says what would need to change |
| Before/after contrast | Two disconnected sections | Section 3 visibly answers the same five questions as Section 1, showing the improvement |

## Submission

Commit `challenge-01.md` to your portfolio under `c37-week-08/challenge-01/`.
