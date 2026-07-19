# Lecture 1 — IaaS, PaaS, SaaS: Choosing the Model

> **Duration:** ~2 hours. **Outcome:** You can place any system on the IaaS/PaaS/SaaS spectrum, explain exactly which layers you'd be responsible for at each level using the shared-responsibility model, and make — and defend — a deliberate choice of cloud model for a real system instead of defaulting to whatever's trendy.

Every week so far, Crunch Cycles has lived on your laptop. Week 7 gave it a REST API and an ETL pipeline, but "deploy" meant `python3 app.py` in a terminal you were staring at. That's not a system anyone else can use — the moment you close the laptop, close the lid, or lose wifi, Crunch Cycles goes dark. This week we fix that by putting the system on **someone else's computer, on purpose, with a plan.** Before you provision anything, you need to answer one question that most tutorials skip: *how much of the stack do you want to run yourself, and how much do you want to rent already running?* That question has a name — the cloud service model — and getting it wrong is expensive in two different directions: too much control costs you weeks of undifferentiated ops work; too little control means you can't fix the thing that's on fire.

## 1. The stack, layer by layer

Any running piece of software sits on a stack of layers, from the physical building down to the code you actually wrote:

```
Application       ← your code: the Crunch Cycles Flask app, its routes, its business logic
Data              ← your database: crunchcycles Postgres, its rows
Runtime           ← the language interpreter/VM: Python 3.11, the WSGI server
Middleware        ← the pieces that glue runtime to OS: containers, process managers
Operating System  ← Linux, patched and configured
Virtualization    ← the hypervisor slicing physical machines into virtual ones
Servers           ← physical compute — racks of machines
Storage           ← physical disks
Networking        ← physical cables, switches, routers, the data center's uplink
```

**On-premises** ("on-prem") means *you* own and run every layer — you bought the servers, you're in the building, you're the one who gets paged when a disk dies at 3 a.m. Nobody starting a project in 2026 does this for a system like Crunch Cycles; it's the baseline everything else is defined against.

**Cloud computing** means a provider (AWS, Google Cloud, Microsoft Azure, and dozens of smaller ones) owns the bottom layers and rents you access to the top ones — over the network, billed by usage instead of by purchase. The three service models differ in exactly **where the line is drawn** between "the provider runs this" and "you run this."

## 2. The three models

### IaaS — Infrastructure as a Service

You rent the bottom layers — servers, storage, networking, and virtualization — as raw, general-purpose building blocks. The provider hands you a virtual machine with an IP address and a blank Linux install; everything above the OS is yours to configure.

**Examples:** AWS EC2, Google Compute Engine, Azure Virtual Machines, DigitalOcean Droplets, Linode.

**You manage:** OS patching, runtime installation (Python, its version, its packages), your web server (Gunicorn, nginx), your database engine, backups, security updates, scaling — literally everything except the physical hardware and the hypervisor.

**Provider manages:** physical servers, storage hardware, networking hardware, virtualization, power, cooling, physical security.

**When it fits:** you need control most PaaS platforms don't give you — a specific OS kernel version, a custom networking topology, a non-mainstream language runtime, or workloads (GPU clusters, specialized databases) no PaaS packages nicely. It also fits when you already have a dedicated ops/platform team whose job *is* running servers — the "undifferentiated heavy lifting" isn't wasted effort for them, it's the job.

### PaaS — Platform as a Service

You rent a *platform* that already has the OS, runtime, and middleware configured — you hand over your application code (or a container) and a Git push, and the platform builds it, runs it, load-balances it, and restarts it when it crashes. You stop thinking about servers entirely and start thinking about your app.

**Examples:** Render, Railway, Heroku, Fly.io, AWS Elastic Beanstalk, Google App Engine.

**You manage:** your application code, your data, your environment configuration (env vars, secrets), and choices *within* the platform's model (how many instances, what size).

**Provider manages:** OS, runtime, middleware, servers, storage, virtualization, networking, and usually TLS certificates, load balancing, and zero-downtime deploys out of the box.

**When it fits:** the overwhelming majority of small-to-mid web applications and APIs — including Crunch Cycles. You want to ship features, not patch kernels. A two-person team on Render can run a production API with the same operational rigor (automatic HTTPS, health-checked restarts, log aggregation) that used to take a dedicated ops engineer to hand-roll on raw VMs.

### SaaS — Software as a Service

You rent a finished *application*, full stop. You don't write code against it beyond configuration and, sometimes, an API; you log in and use it.

**Examples:** Salesforce (the CRM concept from Week 6's `crm_opportunities` table, as an off-the-shelf product), Gmail, Slack, Stripe (payments), SendGrid (transactional email), GitHub.

**You manage:** your data *within* the tool, your configuration, your users.

**Provider manages:** literally everything else, including the application code itself.

**When it fits:** whenever the thing you need is a solved problem that isn't your competitive advantage. Crunch Cycles doesn't need to build its own email-sending infrastructure (SMTP servers, deliverability reputation, bounce handling) — that's SendGrid's entire job, and doing it well is genuinely hard. Building your own would be a worse version of a thing you can rent for a few dollars a month. This is the "build vs. buy" instinct from Week 6's ERP/CRM lecture, generalized: **SaaS is "buy" applied to entire categories of software you don't need to own.**

## 3. The shared-responsibility model

Cloud providers describe this trade-off formally as the **shared-responsibility model**: security and operation of *everything below your chosen line* is the provider's job; everything *above* the line, including whether you configured it correctly, is yours — even on a fully managed platform. A misconfigured database password or an S3 bucket left publicly readable is **your** fault under any model, IaaS through SaaS; the provider secures the vault, you're still responsible for not leaving the door open.

| Layer | On-prem | IaaS | PaaS | SaaS |
|---|:---:|:---:|:---:|:---:|
| Application code & logic | You | You | You | Provider |
| Data | You | You | You | You (in their store) |
| Access control / config | You | You | You | You |
| Runtime (language, libraries) | You | You | Provider | Provider |
| OS patching | You | You | Provider | Provider |
| Servers / virtualization | You | Provider | Provider | Provider |
| Physical security & power | You | Provider | Provider | Provider |

Read the table as a diagonal: as you move right, the line separating "you" from "provider" slides down the stack. Nothing on the right side of the line is your problem; everything on the left side still is, no matter how "managed" the marketing calls it.

## 4. A decision framework, not a default

New teams tend to pick a model by reflex — "everyone uses AWS EC2" or "just throw it on Heroku" — instead of by reasoning. Ask these four questions instead:

1. **How much undifferentiated ops work can the team absorb, and does anyone want to?** A solo founder or small dev team almost always wants PaaS — every hour spent patching a kernel is an hour not spent on the product. A platform team of five whose entire job is infrastructure can extract real value from IaaS control.
2. **Does the workload need something the higher-level model doesn't offer?** GPU-heavy ML training, a legacy application needing a specific OS version, or extreme cost optimization at massive scale often forces IaaS. Most CRUD APIs and web apps don't.
3. **Is this capability core to what makes the business valuable, or a solved commodity problem?** If competitors don't win or lose on "how well we run our own email server," rent it (SaaS). If the thing *is* the product — Crunch Cycles' order-management logic — build and own it.
4. **What's the compliance and control requirement?** Some regulated workloads (certain government, health, or financial-data workloads) require control down to specific layers that only IaaS — or on-prem — provides. Most startups building a first product don't have this constraint yet; don't borrow tomorrow's requirements today.

## 5. Placing Crunch Cycles on the spectrum

Walk through the actual system built in Weeks 3–7 and place each piece:

| Piece | Model | Why |
|---|---|---|
| The Flask REST API (Week 7) | **PaaS** (Render, Railway, Fly.io) | It's a standard web app; nobody at Crunch Cycles wants to own OS patching for a two-person team. Deploy from Git, get HTTPS and restarts for free. |
| The `crunchcycles` PostgreSQL database | **PaaS** — a managed database service (Render Postgres, Neon, Supabase, AWS RDS) | Automated backups, point-in-time recovery, and patching without anyone at Crunch Cycles becoming a full-time DBA. Running Postgres yourself on a bare IaaS VM is a common early-stage mistake — you inherit backup strategy, failover, and security patching with zero of it being your differentiator. |
| Sending order-confirmation emails | **SaaS** (SendGrid, Postmark, AWS SES) | Deliverability is a hard, unglamorous, solved problem. Buy it. |
| Taking customer payments | **SaaS** (Stripe) | Handling raw card numbers yourself invites a compliance burden (PCI-DSS) that a two-person team should never take on voluntarily. |
| Internal team chat & docs | **SaaS** (Slack, Notion, Google Workspace) | Zero competitive advantage in hosting your own chat server. |
| A future GPU-heavy demand-forecasting job (foreshadowing Week 11) | **IaaS** or a specialized managed ML platform | Standard PaaS platforms don't package GPU compute well; you may need lower-level control here — but even then, most teams reach for a managed ML platform (still closer to PaaS) before raw IaaS. |

Notice the pattern: **almost nothing about Crunch Cycles' actual application logic belongs on IaaS.** The team's job is order management, inventory, and reporting — not running Linux servers. That's true for most of the systems you'll build in this course and in your career. IaaS is the exception you reach for when you have a specific reason, not the default you start from.

## 6. Check yourself

- In one sentence, what's the difference between what you manage on IaaS versus PaaS?
- Why is "running your own Postgres on a bare VM" often a mistake for a small team, even though it's technically IaaS territory?
- Give an example of a capability that's a strong SaaS candidate for almost any company, and explain why in terms of "core to the business vs. solved commodity problem."
- Under the shared-responsibility model, who is responsible for a database password accidentally committed to a public GitHub repo — on IaaS? On PaaS? On SaaS? Why does the answer not change?
- Name one legitimate reason a team might choose IaaS over PaaS for a brand-new project in 2026.

If those are automatic, Lecture 2 goes one level deeper — the actual building blocks (compute, managed databases, object storage, networking) you assemble a system from, regardless of which model you pick.

## Further reading

- **AWS — Shared Responsibility Model:** <https://aws.amazon.com/compliance/shared-responsibility-model/>
- **NIST — The NIST Definition of Cloud Computing (SP 800-145):** <https://nvlpubs.nist.gov/nistpubs/legacy/sp/nistspecialpublication800-145.pdf>
- **Render — Docs, "What is Render?":** <https://render.com/docs>
- **Martin Fowler — "PaaS vs IaaS" style comparisons and the broader "undifferentiated heavy lifting" framing:** <https://martinfowler.com/bliki/>
