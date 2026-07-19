# Lecture 1 — Modeling Processes with BPMN

> **Duration:** ~2 hours. **Outcome:** You can read and draw a BPMN diagram with tasks, gateways, events, and swimlanes — precisely enough that two different people modeling the same real process converge on the same diagram.

## 1. Why model a process before you touch code

Every automation failure in this course's later weeks traces back to the same root cause: somebody wrote code against a process that existed only in their head, and their head was wrong about at least one branch. "The dispatcher assigns the closest available tech" sounds simple until you ask: closest by what measure? What if none are available? What if the bike can't be moved? A sentence hides all of that. A diagram can't.

**BPMN** (Business Process Model and Notation) is a small, standardized vocabulary — maintained by the Object Management Group — for drawing exactly what happens in a process: who does what, in what order, under which conditions. It's standardized on purpose. A flowchart you invent yourself is readable by you; a BPMN diagram is readable by anyone who has spent an hour learning the same eight symbols, on any team, in any company, in any tool. That portability is the entire point — it's why BPMN, not a personal flowchart style, is what you'll see in real requirements documents, real consulting decks, and real process-improvement projects.

You don't need special software to use it. Every diagram in this lecture is drawn two ways: as a description in English, and as a [Mermaid](https://mermaid.js.org/) flowchart (renders directly in GitHub/most Markdown viewers, no install). If you prefer a dedicated BPMN tool, [bpmn.io](https://demo.bpmn.io/) is free, runs in the browser, and uses the exact shapes below.

## 2. The core elements

BPMN has dozens of symbols for exhaustive edge cases. You need about eight of them for 95% of real work. Learn these cold.

### Events — circles

An **event** is something that *happens* — it has no duration and does no work itself.

| Symbol | Name | Meaning |
|--------|------|---------|
| Thin circle | **Start event** | Where the process begins — a trigger. Exactly one per "happy path" start, though a process can have several alternative starts. |
| Thick/double circle | **End event** | Where a path of the process terminates. A process can (and often should) have *multiple* end events — "repaired," "sent to depot," "retired" are three different valid endings. |
| Circle with an envelope | **Message event** | Something arrives from outside the process (a form submission, an email, an API webhook). |
| Circle with a clock | **Timer event** | A wait for a specific time or duration ("every night at 2am," "3 days after opened"). |

A process with no explicit end event, or with a start event that isn't actually a trigger ("the process begins" is not a trigger — "rider submits a damage report" is), is the single most common beginner mistake. Name the trigger.

### Activities — rounded rectangles

An **activity** (BPMN calls the smallest unit a **task**) is work that takes time and produces a result.

| Task type | Icon | Meaning |
|-----------|------|---------|
| **User task** | small person icon | A human does this, usually with software helping (filling a form, reviewing a ticket) |
| **Manual task** | small hand icon | A human does this with *no* software involved (physically inspecting a bike) |
| **Service task** | small gear icon | A system does this automatically, no human involved (an API call, a scheduled script) |
| **Script task** | small script icon | Automated logic runs inline in the process engine (rare outside dedicated BPM software — for this course, treat it the same as a service task) |

The task-type icon is not decoration — it's the single most useful piece of information in the whole diagram once you get to Lecture 2, because "service task" and "automated already" are the same thing, and every remaining **user task** or **manual task** is a candidate you have to evaluate.

### Gateways — diamonds

A **gateway** is a decision point or a fork/join in the flow. Getting these three right is the part beginners fumble most.

| Symbol | Name | Meaning |
|--------|------|---------|
| Diamond with an ✕ | **Exclusive gateway (XOR)** | Exactly **one** path is taken, based on a condition. "Is severity major or minor?" → one branch, never both. |
| Diamond with a `+` | **Parallel gateway (AND)** | **All** outgoing paths happen, simultaneously, no condition. "Notify the warehouse AND block the bike from being unlocked" — both happen, always, together. |
| Diamond with a circle | **Inclusive gateway (OR)** | **One or more** paths happen, each independently evaluated. Rarer — use it only when "some subset of these, depending on conditions" is genuinely what happens. |

Every gateway that **splits** the flow into branches should have a matching gateway of the **same type** downstream where the branches **merge** back together, unless a branch ends the process entirely. A parallel split that merges with an exclusive join (or vice versa) is a classic modeling bug — it silently changes the process's actual behavior (does the merge wait for both branches, or does it fire on the first one to arrive?).

### Pools and swimlanes — the rectangle grid

A **pool** represents a whole participant (an organization, a system). A **swimlane** (or just "lane") inside a pool represents one role or system component. Every task in the diagram sits in exactly one lane — the lane says **who is responsible** for that piece of work. This is the feature most hand-drawn flowcharts skip, and it's the one that makes BPMN actually useful for organizational analysis: you can look at a diagram and instantly see whose workload is heavy, where work crosses a boundary (a **handoff**, always a risk point), and which lane is entirely automated (all service tasks) versus entirely human.

### Sequence flow vs. message flow — the arrows

A solid arrow (**sequence flow**) shows order *within* one pool — "this happens, then that happens." A dashed arrow (**message flow**) shows communication *between* pools — one participant sending something to another. If your whole diagram lives in one pool with several lanes (the common case for one company's internal process), you'll use sequence flow almost exclusively; message flow shows up when modeling interactions *between* separate organizations (CrunchRide and a third-party bike manufacturer, say).

## 3. Reading order and legend, at a glance

| Shape | Meaning |
|-------|---------|
| ○ thin circle | start event |
| ◎ thick circle | end event |
| ▭ rounded rectangle | task (icon says what kind) |
| ◇ with ✕ | exclusive gateway — pick one path |
| ◇ with + | parallel gateway — do all paths |
| → solid arrow | sequence flow (order, within a pool) |
| ⇢ dashed arrow | message flow (between pools) |
| horizontal band | swimlane — who's responsible |

## 4. Worked example — CrunchRide's damage-to-repair process

Here is the process **as it actually runs today** at CrunchRide, described the way you'd hear it if you interviewed the dispatcher (Priya) and a field technician (Marcus). This is deliberately written the way a stakeholder actually talks — a little repetitive, a little vague in one spot on purpose, because turning exactly this kind of raw description into a precise diagram is the actual skill.

> "A rider ends their ride and if something's wrong with the bike, they flag it in the app — that locks the bike so nobody else can grab it. That creates a ticket in our system automatically. I look at the ticket, read the notes, and decide if it's minor or major. If it's minor, I just find whichever tech is in that zone and has room on their plate and assign it to them — they go inspect it wherever it's parked. If it's major, two things happen at once: I flag the bike as needs-pickup so the warehouse team knows to send a van for it, and I also assign a senior tech to go do the inspection first, because we don't want to move a bike that just needs a five-minute fix. Once a tech inspects it, one of two things happens: either they can fix it on the spot — they do, mark it fixed, and the bike goes back in service — or they can't, and it goes to the depot, where someone decides whether it's worth repairing there or if it should just be retired. If it's retired, that's the end. If it's repaired at the depot, it goes back into service too."

### Step 1 — find the participants (pools/lanes)

Four lanes, all inside one pool ("CrunchRide"):

- **Rider** — reports the problem, does nothing else in this process
- **App/System** — the automated parts: locking the bike, creating the ticket
- **Dispatcher** — reads the ticket, decides severity, assigns
- **Field Technician** — inspects, repairs or escalates

(A fifth lane, **Depot**, appears only on the major/unrepairable branch — some diagrams add it as a lane, others as a separate pool since it's a physically different facility with different staff. Either is defensible; state which you chose.)

### Step 2 — find the events

- **Start:** "Rider flags a problem on ride end" (message event — it's a trigger from outside the automated system, the rider's own action)
- **Ends:** three of them — "Bike back in service (field-repaired)," "Bike back in service (depot-repaired)," "Bike retired"

### Step 3 — find the tasks and their types

| Task | Lane | Type |
|------|------|------|
| Flag problem on ride end | Rider | user task |
| Lock bike / create ticket | App/System | service task |
| Read ticket, judge severity | Dispatcher | user task |
| Assign tech (minor path) | Dispatcher | user task |
| Flag bike needs-pickup | Dispatcher | user task |
| Assign senior tech (major path) | Dispatcher | user task |
| Inspect bike | Field Technician | manual task |
| Repair on the spot | Field Technician | manual task |
| Transport to depot | Field Technician (or Depot) | manual task |
| Depot repair-or-retire decision | Depot | user task |

### Step 4 — find the gateways

- **Exclusive:** "minor or major?" after the dispatcher reads the ticket
- **Parallel:** on the major path — "flag needs-pickup" AND "assign senior tech" happen together, not one-then-the-other
- **Exclusive:** "fixable on the spot?" after inspection
- **Exclusive:** at the depot — "repair or retire?"

### Step 5 — draw it

```mermaid
flowchart TD
    subgraph Rider
        A((Rider ends ride,<br/>flags problem)):::start
    end
    subgraph System["App / System"]
        B[Lock bike,<br/>create ticket]
    end
    subgraph Dispatcher
        C[Read ticket,<br/>judge severity]
        D{Severity?}
        E[Assign nearby<br/>tech - minor]
        F[Flag needs-pickup]
        G[Assign senior tech]
    end
    subgraph Tech["Field Technician"]
        H[Inspect bike]
        I{Fixable<br/>on the spot?}
        J[Repair on the spot]
        K[Transport to depot]
    end
    subgraph Depot
        L{Repair or retire?}
        M[Repair at depot]
    end

    A --> B --> C --> D
    D -- minor --> E --> H
    D -- major --> F
    D -- major --> G
    F --> H
    G --> H
    H --> I
    I -- yes --> J --> N1((Back in service)):::end
    I -- no --> K --> L
    L -- repair --> M --> N2((Back in service)):::end
    L -- retire --> N3((Retired)):::end

    classDef start fill:#2b6,stroke:#1a4;
    classDef end fill:#c33,stroke:#911;
```

Read it lane by lane, then read it top-to-bottom by following one path at a time (trace the minor path start to finish, then the major/repair path, then the major/retire path). That's the whole discipline: **one branch at a time**, never try to hold the whole diagram in your head while drawing it.

## 5. Common modeling mistakes (and how to catch them)

- **Missing end events.** If a lane has a task with no arrow leading anywhere, the process doesn't actually end there — you forgot to draw the rest, or you forgot an end event. Every path must terminate.
- **A gateway with only one outgoing path.** That's not a gateway, that's just a task with a label. Delete it, or add the second real branch.
- **Mismatched split/join types.** A parallel split (both branches always happen) that later joins with an exclusive gateway (proceed as soon as *either* branch arrives) silently changes the process — does the "assign senior tech" step actually have to *finish* before inspection can start, or can it happen in parallel? Pin this down explicitly; don't leave it implied.
- **Putting a task in the wrong lane.** "Assign senior tech" belongs to the Dispatcher, not the Field Technician, even though it's *about* the technician. The lane answers "who does this work," not "who does this work affect."
- **A task that's secretly two tasks.** "Inspect and repair" hides a decision (fixable on the spot?) inside a single box. If a gateway could plausibly sit in the middle of your task's name, split it.
- **No trigger on the start event.** "Process begins" is not an event. Name the thing that actually kicks it off — a message, a timer, a person's action.

## 6. Check yourself

- What's the difference between a user task and a manual task? Why does that distinction matter later, in Lecture 2?
- When would you use a parallel gateway instead of an exclusive gateway? Give a CrunchRide example that isn't from this lecture.
- Why should a split gateway's matching join be the same type?
- What's a swimlane for, and why does putting the wrong actor in a lane matter, beyond just being "neater"?
- In the worked example, why are there three end events instead of one?
- What's the difference between a sequence flow and a message flow?

If those are automatic, Lecture 2 uses this exact diagram to decide which of its ten tasks should become code.

## Further reading

- **OMG BPMN 2.0 specification (the source of truth):** <https://www.omg.org/spec/BPMN/2.0/>
- **bpmn.io — free browser BPMN editor, same notation as this lecture:** <https://demo.bpmn.io/>
- **Camunda's BPMN symbol reference (excellent visual glossary):** <https://camunda.com/bpmn/reference/>
- **Mermaid flowchart syntax (used for every diagram above):** <https://mermaid.js.org/syntax/flowchart.html>
