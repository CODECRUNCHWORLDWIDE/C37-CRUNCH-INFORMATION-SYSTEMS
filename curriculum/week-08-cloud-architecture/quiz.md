# Week 8 — Quiz

Fourteen questions. Lectures closed. Aim for 12/14 before starting Week 9. A mix of multiple-choice and short "what's wrong here" questions — the answer key at the bottom explains the *why*, not just the letter.

---

**Q1.** On **PaaS**, which of the following are you still responsible for managing? (Select the best single answer.)

- A) The operating system and its security patches
- B) The physical servers and virtualization layer
- C) Your application code and its data
- D) Nothing — PaaS manages everything

---

**Q2.** Under the shared-responsibility model, who is responsible for a database password accidentally committed to a public GitHub repository, if the database is hosted on a fully managed PaaS platform?

- A) The cloud provider — they manage the database
- B) Nobody — it's an unavoidable risk of using the cloud
- C) You — the provider secures the infrastructure below your line, but access control and secrets are always yours
- D) GitHub, for allowing the commit

---

**Q3.** A two-person startup wants to ship a standard CRUD web application as fast as possible and has no dedicated ops staff. Which cloud model best fits, and why?

- A) IaaS — full control is always best
- B) PaaS — they get OS/runtime management for free and can focus on the product
- C) SaaS — they should buy a finished app instead of building one
- D) On-premises — cheapest long-term

---

**Q4.** A company decides to use Stripe for payments instead of building its own card-processing system. This is an example of:

- A) IaaS, because Stripe runs on servers
- B) SaaS, because they're renting a finished capability that isn't their core business
- C) PaaS, because Stripe has an API
- D) A mistake — they should always build payment processing in-house for control

---

**Q5.** Which of these is **object storage** best suited for?

- A) A `products` table's `price` column
- B) A generated PDF invoice for a customer order
- C) The rows of the `orders` table
- D) An in-memory session cache

---

**Q6.** Why shouldn't a managed production database typically accept connections from any public IP address?

- A) Managed databases don't support public connections at all
- B) It should sit in a private network, reachable only from the app instances (and admin access) that need it — reducing the attack surface
- C) Public IPs are always more expensive
- D) `psql` can't connect to a public IP

---

**Q7.** A load balancer's health check fails for one of three app instances behind it. What happens?

- A) The whole service goes down until someone intervenes
- B) The load balancer stops routing new traffic to the failing instance and continues serving from the other two
- C) The health check automatically restarts the entire database
- D) Nothing — health checks are informational only

---

**Q8.** A system targets 99.9% availability. Roughly how much downtime per month does that allow?

- A) About 7 hours
- B) About 44 minutes
- C) About 4 minutes
- D) About 26 seconds

---

**Q9.** Why does horizontal scaling require the application to be stateless?

- A) It doesn't — any application can be horizontally scaled
- B) Stateless apps are always faster
- C) If any instance can die or be added without losing data that only existed on that one instance, requests can be safely spread across many instances
- D) Stateless is required only for databases, not applications

---

**Q10.** Why is scaling a relational database horizontally (adding more instances) fundamentally harder than scaling a stateless app tier horizontally?

- A) It isn't harder — the exact same load-balancer approach works
- B) Every database instance must see the same consistent data, unlike interchangeable stateless app instances
- C) Databases cannot be scaled at all
- D) Managed databases don't allow more than one instance ever

---

**Q11.** A team's app runs in `us-east-1` and its database runs in `eu-west-1`. What cost problem does this create, beyond added latency?

- A) None — cross-region traffic within your own account is always free
- B) Continuous data-transfer (egress) charges for every query's traffic crossing the region boundary
- C) The database will refuse the connection
- D) It only affects backup costs, not live traffic

---

**Q12.** A `LIMIT`-less staging environment, sized identically to production and left running 24/7 despite being used two weeks a year, appears on a cloud bill. What Lecture 3 cost trap is this?

- A) Data egress
- B) An idle/over-provisioned resource running when it doesn't need to
- C) Sharding overhead
- D) A necessary and unavoidable cost of having a staging environment at all

---

**Q13.** Per this course's data rule, how should a team model "what would our cloud bill be at 10x traffic"?

- A) In a shared Excel spreadsheet, since it's a financial calculation
- B) In Python (or SQL for stored line items), so line items can be broken out and the growth scenario is a parameter change, not a rebuilt grid of cells
- C) It can't be estimated in advance
- D) By asking the cloud provider's sales team and trusting their number verbatim

---

**Q14.** In a 10x-growth cost projection, why shouldn't every line item simply be multiplied by 10 uniformly?

- A) Cloud providers apply a fixed 10x discount at scale
- B) Different line items scale with different drivers — e.g., egress tends to scale with traffic, while database storage often grows more with data accumulated over time than with read traffic — so a uniform multiply overstates some costs and understates others
- C) Costs never actually change with more traffic
- D) Compute is always the only cost that changes

---

## Answer key

<details>
<summary>Reveal after attempting</summary>

1. **C** — on PaaS, the provider manages OS, runtime, and servers; your code and your data are always yours to manage, on every model including SaaS's data-within-the-tool.
2. **C** — the shared-responsibility model puts access control and secrets on your side of the line at every service model, even fully managed ones. The provider secures the vault; you're still responsible for not leaving the door open.
3. **B** — PaaS is the textbook fit for a small team that wants to ship product, not manage servers: OS/runtime/patching are the provider's job, letting the team focus entirely on application code.
4. **B** — SaaS: Stripe is a rented, finished capability (payment processing) that isn't this company's core differentiator, and building it in-house would add compliance burden (PCI-DSS) for no competitive gain.
5. **B** — a generated PDF is a large, unstructured file — exactly what object storage is designed for. Structured rows belong in the database; session caches belong in an in-memory store, not object storage.
6. **B** — a private network limits who can even attempt a connection, shrinking the attack surface dramatically versus accepting connections from any IP on the internet.
7. **B** — that's the entire point of a health-checked load balancer: it detects the failing instance and routes around it, keeping the service available from the remaining healthy instances.
8. **B** — 99.9% allows about 43.8 minutes of downtime per month (8.76 hours/year ÷ 12, roughly).
9. **C** — statelessness means no instance holds data the others don't also have access to (it's all in the shared database/cache), so any instance can safely handle any request, live, or die without losing anything unique.
10. **B** — every database instance must agree on the same data at all times; app instances are interchangeable and don't need to coordinate with each other the same way, which is what makes them trivial to add or remove.
11. **B** — cross-region traffic between your own app and database is billed as data egress on essentially every request/response, continuously — not a one-time cost, a recurring one that scales with traffic.
12. **B** — an over-provisioned resource (sized like production) running continuously despite minimal real use is exactly the idle/over-provisioned trap from Lecture 3.
13. **B** — Python or SQL, per the course's data rule — never a spreadsheet — because it makes "what if traffic 10x's" a parameter change instead of a maze of copy-pasted cells, and keeps each line item auditable.
14. **B** — different cost drivers (traffic-driven egress vs. time-driven storage accumulation, for example) scale at different rates, so a uniform 10x multiply is a lazy approximation that misrepresents where the real cost growth will actually come from.

</details>

**Scoring:** 12+ → start Week 9. 9–11 → re-read the lecture sections behind your misses. <9 → re-read all three lectures from the top; deployment, availability, and cost reasoning compound directly into Week 9's security work.
