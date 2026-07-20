# Lecture 2 — Cloud Building Blocks

> **Duration:** ~2 hours. **Outcome:** You can name and reason about the four primitives every cloud architecture is assembled from — compute, managed databases, object storage, and networking — choose sensible options for each for a real system, and draw an architecture diagram of Crunch Cycles that a teammate could deploy from without asking you a single question.

Lecture 1 settled *which model* to rent from. This lecture is about *what you're actually renting* — the small set of building blocks that every cloud provider offers under different brand names, and that every architecture, from a solo side project to a bank's core ledger, is built out of. Learn these four primitives once and you can read any provider's documentation, because AWS's "EC2 + RDS + S3 + VPC" and Google Cloud's "Compute Engine + Cloud SQL + Cloud Storage + VPC" and Render's "Web Service + Postgres + Disk + private networking" are the same four ideas with different labels.

## 1. Compute — where your code runs

Compute is the CPU/RAM/execution environment that runs your application. There are three shapes, from most control to least:

### Virtual machines (VMs)

A full, isolated operating system, sized by vCPU and RAM (e.g., "2 vCPU, 4 GB RAM"). You install and run anything, but you're responsible for the whole software stack above the OS. This is the IaaS compute shape from Lecture 1.

```
Example sizing (illustrative, provider pricing varies):
  1 vCPU / 1 GB RAM   — small script runners, low-traffic side projects
  2 vCPU / 4 GB RAM   — a small production API under light-to-moderate load
  4 vCPU / 16 GB RAM  — a database server, or a compute-heavy job
```

### Containers

Your application, packaged with its exact runtime and dependencies into a portable image (typically via **Docker**), run by a platform that handles the OS and scheduling for you. This is the PaaS compute shape — you write a `Dockerfile` (or the platform infers one from your `requirements.txt`), push it, and the platform runs N copies of it behind a load balancer.

```dockerfile
# A minimal Dockerfile for the Crunch Cycles Flask API from Week 7
FROM python:3.11-slim

WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

# Gunicorn, not Flask's dev server — the dev server is single-threaded
# and explicitly says "do not use in production" in its own docs.
CMD ["gunicorn", "--bind", "0.0.0.0:8080", "--workers", "3", "app:app"]
```

The container is the unit most PaaS platforms scale by: "3 instances" means three running copies of this exact image, each getting its own slice of CPU/RAM, with the platform's load balancer spreading traffic across them.

### Serverless functions

Code that runs only in response to an event (an HTTP request, a queued message, a schedule) and that you are billed for **only while it executes** — no idle cost, but also a cold-start delay and a hard execution-time limit. Good for spiky, event-driven work (resizing an uploaded product photo, processing a webhook payload from Week 7); a poor fit for an always-on API expecting low, consistent latency, because the first request after idle time pays a cold-start tax.

**Which one for Crunch Cycles' API?** A container on a PaaS platform. It's an always-on API with steady traffic — serverless's cold starts would hurt latency for no benefit, and a container gets you 90% of a VM's flexibility with none of the OS-patching burden.

## 2. Managed databases — where your data lives

A **managed database** is a database engine (PostgreSQL, MySQL, etc.) that the provider runs, patches, and backs up for you — you get a connection string, not SSH access to the machine it runs on.

**What "managed" buys you that self-hosting on a VM doesn't, out of the box:**

- **Automated backups** — nightly snapshots, often with **point-in-time recovery** (restore to any minute in the last N days, not just last night).
- **Patching** — security patches to the database engine applied without you doing anything (usually during a scheduled maintenance window).
- **High availability options** — a **standby replica** in a different data center that the platform can promote to primary automatically if the primary dies (more in Lecture 3).
- **Read replicas** — read-only copies you can point reporting/analytics queries at, so a heavy `GROUP BY` report (Week 4) doesn't slow down the live order-taking traffic hitting the primary.
- **Connection pooling** — a layer (e.g., PgBouncer) that multiplexes many application connections onto fewer real database connections, because Postgres connections are expensive and a naive app opening one per request will exhaust the database's connection limit under load.

For Crunch Cycles, this means: `crunchcycles` moves from "a Postgres process on your laptop" to a managed instance — Render Postgres, Neon, Supabase, or AWS RDS are all reasonable free/cheap starting points — and you connect to it with the same `psql` and `psycopg2` you already know. The SQL doesn't change. What changes is who's responsible for keeping the engine itself alive, patched, and backed up.

```bash
# Locally, Week 3-7: connecting to your own Postgres
psql crunchcycles

# Week 8: connecting to a managed instance — same tool, a remote connection string
psql "postgresql://cc_admin:••••••••@dpg-xxxxx.oregon-postgres.render.com/crunchcycles"
```

**Why not just run Postgres yourself on a cheap VM?** You can — it's valid IaaS — but you've now signed up to be the DBA: write and test a backup script, verify restores actually work (an untested backup is not a backup), apply security patches on your own schedule, and build failover yourself if the VM's disk dies at 2 a.m. A managed database gives you all of that, tested by a team whose full-time job is exactly this, usually for a few dollars a month at small scale. This is Lecture 1's SaaS-for-commodity-work logic applied one layer down the stack.

## 3. Object storage — where your files live

**Object storage** holds unstructured files — images, PDFs, CSV exports, database backup archives — as **objects** (a blob of bytes plus metadata) addressed by a key, not as rows in a database and not as a mounted filesystem. The canonical example is AWS **S3** (Simple Storage Service); every provider has an equivalent (Google Cloud Storage, Azure Blob Storage, Cloudflare R2).

**Object storage vs. a database column vs. a disk:**

| | Object storage | Database (`BYTEA`/`TEXT`) | Block storage (a disk) |
|---|---|---|---|
| Good for | Large files: images, PDFs, exports, backups | Structured, queryable data | The OS and app files a running server needs |
| Scales to | Effectively unlimited, per-object | Bloats your DB and slows backups/replication | Fixed size, attached to one machine |
| Access pattern | Fetch by key over HTTP(S) | `SELECT` with `WHERE`/`JOIN` | Mounted filesystem paths |

**Where Crunch Cycles would use it:** generated PDF invoices for orders, a nightly `pg_dump` backup archive kept in addition to the managed database's built-in backups, exported CSV/Parquet reports for finance, or product images referenced by `products.image_url`. The rule of thumb from Week 3's normalization lectures still applies here at a system level: **don't stuff large binary blobs into Postgres rows** — store the file in object storage and keep a URL/key column in the table pointing at it.

```sql
-- crunchcycles: reference an object-storage key, don't store the file itself
ALTER TABLE products ADD COLUMN image_key TEXT;
-- e.g. 'products/images/product-7-hybrid-city.jpg', fetched from the bucket at request time
```

## 4. Networking — how it's all reachable, and how it's protected

Four concepts show up in nearly every architecture diagram:

- **DNS** — translates a human name (`api.crunchcycles.com`) to an IP address. You buy/manage a domain and point a DNS record at your PaaS platform or load balancer.
- **TLS / HTTPS** — encrypts traffic between the client and your service, and proves to the client they're really talking to you (via a certificate). PaaS platforms typically provision and renew free TLS certificates (via **Let's Encrypt**) automatically — one more thing you'd have to script yourself on raw IaaS.
- **Load balancer** — sits in front of multiple app instances and spreads incoming requests across them, and stops sending traffic to any instance that fails a **health check** (a periodic "are you still alive?" request, usually to a `/health` endpoint your app exposes). This is what makes horizontal scaling (Lecture 3) actually work — without a load balancer, "3 instances" is just three separate, unreachable copies of your app.
- **Private networking / VPC** (Virtual Private Cloud) — an isolated network segment where your app and database talk to each other *without* going over the public internet. Your database should almost never have a public IP that accepts connections from anywhere; it should sit in a private network reachable only from your app instances (and maybe your own IP, for admin work).

```sql
-- add a health-check endpoint the load balancer polls
-- (Flask, extending the Week 7 app.py)
@app.route("/health")
def health():
    return {"status": "ok"}, 200
```

## 5. Assembling the blocks: a Crunch Cycles architecture diagram

Putting all four primitives together for the system you'll actually deploy this week:

```
                         ┌─────────────────────┐
      HTTPS requests     │        DNS           │
   ───────────────────►  │ api.crunchcycles.com  │
                         └──────────┬───────────┘
                                    │ TLS-terminated HTTPS
                                    ▼
                         ┌─────────────────────┐
                         │   Load balancer /     │
                         │   platform ingress     │
                         │   (health-checked)     │
                         └─────┬────────┬────────┘
                               │        │
                    ┌──────────┘        └──────────┐
                    ▼                               ▼
          ┌───────────────────┐           ┌───────────────────┐
          │  App instance #1    │           │  App instance #2    │
          │  (container, Flask  │           │  (container, Flask  │
          │   + Gunicorn)        │           │   + Gunicorn)        │
          └──────────┬─────────┘           └──────────┬─────────┘
                     │                                  │
                     │      private network (VPC)       │
                     └────────────────┬─────────────────┘
                                       ▼
                         ┌─────────────────────────┐
                         │  Managed PostgreSQL        │
                         │  crunchcycles               │
                         │  (primary + standby)        │
                         └──────────┬──────────────────┘
                                    │ nightly export
                                    ▼
                         ┌─────────────────────────┐
                         │  Object storage             │
                         │  (invoice PDFs, DB backups)  │
                         └─────────────────────────┘
```

Every box in that diagram is one of the four primitives from this lecture: compute (the app instances), managed database (Postgres primary + standby), object storage (backups/invoices), and networking (DNS, load balancer, private network). This is the diagram you'll redraw and extend for your own mini-project deployment.

```mermaid
flowchart TD
    DNS["DNS api.crunchcycles.com"] --> LB["Load balancer health checked"]
    LB --> App1["App instance 1 container"]
    LB --> App2["App instance 2 container"]
    App1 --> DB["Managed PostgreSQL primary and standby"]
    App2 --> DB
    DB --> Obj["Object storage invoices and backups"]
```
*Request flow from DNS through the load balancer to app instances, the managed database, and object storage.*

## 6. Check yourself

- Name the three compute shapes and, for each, one workload it fits well and one it fits poorly.
- Why shouldn't a product's large image files be stored as `BYTEA` columns in the `products` table? What should happen instead?
- What does a load balancer's health check actually do, and why does horizontal scaling depend on it?
- Why should Crunch Cycles' managed database sit in a private network instead of accepting connections from any public IP?
- List the automated capabilities a managed database gives you that a self-hosted Postgres on a bare VM does not have by default.

Lecture 3 builds on these four primitives to ask the next question: how many of them do you need, how are they arranged to survive failure, and what does it actually cost.

## Further reading

- **AWS — What Is Cloud Computing? (compute, storage, networking overview):** <https://aws.amazon.com/what-is-cloud-computing/>
- **Docker — Get Started, "What is a container?":** <https://docs.docker.com/get-started/docker-overview/>
- **Render — Docs, Private Networking:** <https://render.com/docs/private-network>
- **PostgreSQL — High Availability, Load Balancing, and Replication:** <https://www.postgresql.org/docs/current/high-availability.html>
- **Let's Encrypt — How It Works:** <https://letsencrypt.org/how-it-works/>
