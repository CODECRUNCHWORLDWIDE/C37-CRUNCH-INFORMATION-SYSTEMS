# Week 5 — Homework

Extra practice, spaced across the week, tying together BPMN modeling, automation-candidate scoring, and reliable Python automation. Do these after the exercises, alongside or after the challenges — they're independent of each other and independent of the mini-project, so any order works.

## 1. Model a second CrunchRide process (BPMN practice, ~90 min)

CrunchRide also has a **station rebalancing** process: some stations fill up with bikes (nobody can dock) while others run empty (nobody can rent). Every evening, an operations coordinator looks at dock counts across all 6 stations and decides which bikes need to be moved where, then radios two rebalancing drivers with instructions.

1. Write your own 150–200 word "interview transcript" describing this process, in the coordinator's voice, the way Lecture 1's dispatcher interview and Exercise 1's Sofia interview are written. Include at least one gateway-worthy decision point (e.g., "if a station is over 90% full, it's flagged for pickup; under 10% full, it's flagged for drop-off") and one point where two things happen at once.
2. Draw the full BPMN diagram from your own transcript.
3. Score at least 2 of its tasks using the Lecture 2 rubric. Is "decide which bikes go where" a good automation candidate? Defend your answer — this one is genuinely debatable, and a strong answer engages with the debate rather than picking a side too quickly.

## 2. Extend the bottleneck analysis (SQL practice, ~45 min)

Using `repair_tickets`, write queries to answer:

1. Which **technician** (from the historical `completed` tickets) has the fastest average `hours_to_complete` (assigned → completed)? Which is slowest?
2. Group completed tickets by `severity` and compute average `hours_to_assign` for each group separately. Does severity correlate with how fast the dispatcher historically responded — and is that the direction you'd expect (major tickets assigned *faster*, one hopes)?
3. Write a query that would flag any ticket that's been `open` for more than 4 hours **right now**, as of whenever you run the query (hint: compare `opened_ts` to `now()`/`CURRENT_TIMESTAMP` directly for still-open tickets, rather than computing against `assigned_ts` which doesn't exist yet for them).

## 3. Extend the automation (Python practice, ~90 min)

Take Lecture 3's `assign_technicians.py` and add **one** of the following (pick one, don't do all three unless you want the extra practice):

- **A summary email/Slack message stub.** After a run completes, build a summary string ("Assigned 3 tickets, skipped 1, 0 errors — see automation_run_log for detail") and print it in a clearly-labeled block. You don't need real email/Slack credentials — a function `send_notification(message: str)` that just logs the message it *would* send is a legitimate, common pattern for a first pass (swap in a real integration later without touching the calling code).
- **A `--dry-run` flag.** Reading command-line args (`sys.argv` or the `argparse` module), add a mode that runs the full eligibility + technician-selection logic and logs what it *would* do, without executing the `UPDATE`. This is one of the highest-value features you can add to any automation that mutates data — never underestimate how often "let me see what this would do first" saves someone from a bad surprise.
- **A max-runtime guard.** Add a check so the script logs a warning and stops picking up new tickets (finishing whatever's in flight) if the run has been going for longer than, say, 60 seconds — a real safeguard against an automation silently running far longer than expected because something upstream (a growing backlog, a slow query) changed.

Write 3–4 sentences in `homework.md` explaining what you built and why you picked it.

## 4. Read one real BPMN diagram from outside this course (~20 min)

Search for a real, published BPMN diagram from a company or open-source project (a university's admissions process, an open-source project's release process, a government agency's permit-approval process — plenty are public). Find one, and answer:

1. Does it use swimlanes correctly? Any place you'd have modeled it differently?
2. Find one exclusive gateway and one parallel gateway in it (if it doesn't have both, that's worth noting too — not every real process needs a parallel gateway).
3. Is there anywhere the diagram is ambiguous or looks like it's hiding a judgment call inside a single task box, the way Lecture 1 warned against?

## Submission

Homework isn't formally graded, but push what you produce to `c37-week-05/homework/` in your portfolio — future-you (and a real employer looking at your portfolio) will want the evidence that you did more than the minimum.
