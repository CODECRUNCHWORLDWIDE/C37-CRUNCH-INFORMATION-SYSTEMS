# Lecture 3 — Availability, Scaling & Cost

> **Duration:** ~2 hours. **Outcome:** You can reason about uptime targets in concrete downtime-per-year terms, design redundancy that survives a single failure, choose deliberately between horizontal and vertical scaling, and read a cloud bill closely enough to catch the handful of mistakes that quietly turn a $20/month hobby project into a $2,000 surprise.

The four building blocks from Lecture 2 don't automatically add up to a system that stays up or a bill you can predict. Two identical-looking architecture diagrams — one app instance talking to one database, versus two app instances across two zones talking to a primary-with-standby database — can differ by orders of magnitude in both reliability and cost, and the difference is invisible until the day a data center has a bad afternoon or a `LIMIT`-less query runs against 50 million rows. This lecture is about making that difference a deliberate choice instead of an accident you discover during an outage or an invoice.

## 1. Availability, in numbers you can act on

**Availability** is the fraction of time a system is usable, usually expressed as a percentage of "nines." The gap between adjacent tiers looks small on paper and is enormous in practice:

| Availability | Downtime per year | Downtime per month | Downtime per week |
|---|---|---|---|
| 99% ("two nines") | 3.65 days | 7.3 hours | 1.68 hours |
| 99.9% ("three nines") | 8.76 hours | 43.8 minutes | 10.1 minutes |
| 99.95% | 4.38 hours | 21.9 minutes | 5.04 minutes |
| 99.99% ("four nines") | 52.6 minutes | 4.38 minutes | 1.01 minutes |
| 99.999% ("five nines") | 5.26 minutes | 26.3 seconds | 6.05 seconds |

A free-tier PaaS platform or a single un-redundant VM realistically gets you somewhere around 99–99.5% on a good year — plenty for a course project, a portfolio demo, or an early-stage MVP where a few hours of downtime a month is annoying but not catastrophic. A payment system or a hospital's patient system needs 99.99%+ and an architecture (and budget) to match. **The first design decision is picking the tier the business actually needs** — chasing five nines for a system that can tolerate 99% wastes money and engineering time that should go into the product.

## 2. Redundancy — surviving one thing dying

**Redundancy** means no single failure takes the whole system down. The unit of failure to design around, at minimum, is the **availability zone (AZ)** — an isolated data center within a cloud region, with its own power, cooling, and networking. Zones fail (power outages, network partitions, hardware faults) more often than most people assume; regions (a geographic area made of several zones) fail far less often but it does happen.

**Single point of failure (SPOF):** any one component whose failure takes down the whole system. In the "before" architecture most projects start with — one app instance, one database, one zone — nearly *everything* is a SPOF:

```
BEFORE (no redundancy — 3 single points of failure):

  [1 app instance] ──► [1 Postgres instance]
       Zone A                Zone A

  If the app process crashes: total outage.
  If the database goes down: total outage.
  If Zone A has a bad day: total outage — nothing to fail over to.
```

```
AFTER (redundant across zones):

  [App instance #1]     [App instance #2]
       Zone A                 Zone B
            \                  /
             \                /
          [Load balancer, health-checked]
                     │
        ┌────────────┴────────────┐
        ▼                          ▼
  [Postgres PRIMARY]      [Postgres STANDBY]
       Zone A                   Zone B
   (replicates continuously to standby;
    standby promotes to primary automatically
    if the primary or Zone A becomes unreachable)
```

Two changes did the work: **at least two app instances in at least two zones** behind a health-checked load balancer (if one instance or zone dies, the other keeps serving — the load balancer stops routing to the dead one within seconds), and **a database standby in a second zone** that the platform can promote automatically. Neither change requires new *kinds* of building blocks — it's the same compute and managed-database primitives from Lecture 2, just provisioned with more than one of each, deliberately spread across zones.

**A subtlety that trips people up:** redundant app instances only help if the app is **stateless** — if any instance can handle any request without needing data that only exists in that instance's memory or local disk. If Crunch Cycles' app stored a shopping cart in a local file or an in-process variable, killing that instance would lose the cart even though "the system" stayed up. The fix is what you've already been doing all course: put state in the database (or, for ephemeral session data, a shared cache like Redis), never on the instance itself. **Statelessness is what makes horizontal scaling and failover possible at all** — it's a design discipline, not a checkbox you tick at deploy time.

## 3. Horizontal vs. vertical scaling

When load grows, you have two directions to add capacity:

**Vertical scaling** ("scale up"): give the *same* instance more CPU/RAM — move from a 1 vCPU/1 GB instance to a 4 vCPU/8 GB instance.

- Simple — no architecture changes, no statelessness requirement.
- Has a ceiling — eventually you run out of bigger machines to rent, or the cost curve gets steep.
- Usually requires a restart (downtime) to resize.
- Does nothing for availability — you still have exactly one instance, exactly one SPOF, just a bigger one.

**Horizontal scaling** ("scale out"): add *more* instances of the same size, spreading load across them via a load balancer.

- Requires statelessness (Section 2) — every instance must be interchangeable.
- Scales further in principle — add instances roughly as demand grows, with no hard ceiling.
- Directly improves availability as a side effect — more instances means more redundancy, not just more throughput.
- The app tier scales horizontally easily; the database tier does not, or not the same way.

**Why the database is the hard part.** You can put ten identical, stateless app containers behind a load balancer with little thought — they don't need to agree with each other about anything. A database is exactly the opposite: every instance needs to see the *same* consistent data, so "just add more Postgres instances" doesn't trivially work. The real tools for scaling a relational database are narrower and each solve a different problem:

- **Read replicas** — extra read-only copies that serve `SELECT` traffic (reports, dashboards), while all writes still go to one primary. This scales *read* load, not write load, and it's exactly what you'd point Week 4's reporting pipeline at so heavy analytical queries stop competing with live order-taking for the same connections.
- **Connection pooling** (PgBouncer, etc.) — doesn't add capacity, but lets many more app-instance connections share a fixed, smaller number of real database connections, which is usually the actual bottleneck before raw query throughput is.
- **Vertical scaling the database** — often the first and simplest lever: a bigger managed-database instance, same architecture, no new failure modes.
- **Sharding** (splitting data across multiple independent databases by some key, e.g. by region) — a last resort for extreme scale, and a substantial complexity jump (cross-shard queries and joins get hard) that most systems, including Crunch Cycles at any realistic size, will never need.

**For Crunch Cycles specifically:** scale the app tier horizontally (it's already stateless — Flask + Postgres, no server-side sessions) and scale the database tier vertically first, adding a read replica only once reporting queries measurably compete with order-taking traffic. Reach for sharding never, until there's overwhelming evidence every simpler lever is exhausted.

## 4. Reading a cloud bill

A cloud bill is a sum of small, usage-metered line items, and the ones that bite are rarely the obvious "how big is my server" line:

| Line item | Billed by | The trap |
|---|---|---|
| Compute (app instances) | Instance-hours × size | Leaving a staging environment sized like production running 24/7 when it's only used during business hours |
| Database storage | GB stored × time | Old, unpruned log tables or forgotten large exports left in the database |
| **Data egress** | GB transferred **out** of the provider's network | The single most common surprise bill — traffic *into* the cloud is usually free; traffic *out* (API responses to users, cross-region traffic between your own app and database, backup downloads) is billed, often per GB, and it's easy to not notice until the invoice arrives |
| Backups / snapshots | GB stored × time, retained | Snapshots nobody deletes, accumulating for months |
| Idle / unattached resources | Instance-hours or GB, regardless of use | A disk volume left behind after its instance was deleted; a load balancer nobody removed after decommissioning a service |
| Auto-scaling without a cap | Instance-hours × however many instances got spun up | A traffic spike (legitimate or a bot/attack) that auto-scales to 50 instances with no maximum set, and nobody notices until the bill |

**Cross-region egress deserves its own callout** because it's the trap that looks like a completely reasonable architecture decision until you see the bill: an app instance in one region talking to a database in a *different* region pays egress on every single query's response, continuously, forever. The fix isn't a special cost-optimization technique — it's the same private-networking discipline from Lecture 2: **keep your app and database in the same region**, ideally the same VPC, so that traffic between them never crosses a metered boundary at all.

## 5. Estimating cost — in code, not a spreadsheet

Per this course's data rule, cost modeling is not a job for a spreadsheet — it's a small, reusable calculation, and Python makes the "what if traffic 10x's" question a one-line change instead of a maze of copy-pasted cells.

```python
# cloud_cost_model.py
# A minimal, transparent monthly cost estimate for Crunch Cycles' deployment.
# Figures are illustrative — pull real numbers from your provider's pricing page
# for the actual estimate you'll build in Exercise 3 and the mini-project.

def estimate_monthly_cost(
    app_instances: int,
    app_instance_hourly_rate: float,
    db_gb_storage: float,
    db_storage_rate_per_gb: float,
    egress_gb: float,
    egress_rate_per_gb: float,
    backup_gb: float,
    backup_rate_per_gb: float,
) -> dict:
    hours_per_month = 730  # the standard cloud-billing month approximation

    compute_cost = app_instances * app_instance_hourly_rate * hours_per_month
    storage_cost = db_gb_storage * db_storage_rate_per_gb
    egress_cost = egress_gb * egress_rate_per_gb
    backup_cost = backup_gb * backup_rate_per_gb

    total = compute_cost + storage_cost + egress_cost + backup_cost

    return {
        "compute": round(compute_cost, 2),
        "db_storage": round(storage_cost, 2),
        "egress": round(egress_cost, 2),
        "backups": round(backup_cost, 2),
        "total": round(total, 2),
    }


current = estimate_monthly_cost(
    app_instances=2,
    app_instance_hourly_rate=0.02,   # e.g. a small container instance
    db_gb_storage=10,
    db_storage_rate_per_gb=0.25,
    egress_gb=50,
    egress_rate_per_gb=0.09,
    backup_gb=10,
    backup_rate_per_gb=0.10,
)
print("Current:", current)

# The 10x-growth question is now a parameter change, not a rebuilt spreadsheet.
growth_10x = estimate_monthly_cost(
    app_instances=6,             # horizontal scaling of a stateless tier
    app_instance_hourly_rate=0.02,
    db_gb_storage=60,            # data grows roughly with usage, not linearly with traffic
    db_storage_rate_per_gb=0.25,
    egress_gb=500,               # egress tends to scale closely with traffic
    egress_rate_per_gb=0.09,
    backup_gb=60,
    backup_rate_per_gb=0.10,
)
print("At 10x growth:", growth_10x)
```

Running this prints each cost component separately — which is the point. A bill that's one opaque total tells you nothing about where to cut; a bill decomposed by line item, computed in code you can re-run with different assumptions, tells you immediately that (in this illustrative model) egress and compute dominate, and that the database storage line barely moves the total even at 10x. That's exactly the kind of insight Challenge 2 and the mini-project will ask you to act on.

## 6. Check yourself

- What's the difference between 99% and 99.9% availability, expressed in hours of downtime per month? Why does that gap matter for a system taking customer orders?
- Why does horizontal scaling require your application to be stateless? What breaks if it isn't?
- Name one thing that scales the app tier horizontally with almost no extra thought, and explain why the database tier doesn't get the same free lunch.
- What is data egress, and why does keeping your app and database in the same region matter for cost, not just latency?
- List three concrete ways a cloud bill can quietly balloon that have nothing to do with legitimate customer traffic growth.
- Why does this course model cloud costs in Python instead of a spreadsheet, and what does that buy you when someone asks "what if we 10x our traffic?"

This closes out the lecture content for the week. Exercises 1–3 put the model choice, the deployment, and the cost estimate into your hands directly; the challenges push into failure-mode design and cost-cutting under a real (if fictional) wasteful bill; the mini-project asks you to actually deploy Crunch Cycles and defend the resulting architecture, scaling plan, and cost estimate as a package.

## Further reading

- **AWS — Well-Architected Framework, Reliability Pillar:** <https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/welcome.html>
- **Google Cloud — Availability Zones and Regions:** <https://cloud.google.com/compute/docs/regions-zones>
- **The Twelve-Factor App — "Processes" (statelessness) and "Concurrency" (horizontal scaling):** <https://12factor.net/processes> · <https://12factor.net/concurrency>
- **PostgreSQL — Connection Pooling:** <https://wiki.postgresql.org/wiki/PgBouncer>
- **AWS — Understanding Data Transfer Costs:** <https://aws.amazon.com/blogs/architecture/overview-of-data-transfer-costs-for-common-architectures/>
