# Week 8 — Resources

Free, public, no signup unless noted. Read the "required" set; treat the rest as reference you dip into when a specific question comes up.

## Install / sign up first

- **A free PaaS account** — pick one: [Render](https://render.com), [Railway](https://railway.app), or [Fly.io](https://fly.io). No credit card needed at the free tiers used this week.
- **A free managed-Postgres account** — Render Postgres (bundled with a Render account), [Neon](https://neon.tech), or [Supabase](https://supabase.com) — pick whichever pairs with your PaaS choice above, or use one for both.
- **Docker (optional but recommended)** — lets you build and test the container locally before deploying: <https://docs.docker.com/get-docker/>
- **`gunicorn`** — the production WSGI server for the Flask app: `pip install gunicorn` (added to `requirements.txt`).

## Required reading (this week's core)

- **AWS — Shared Responsibility Model:** <https://aws.amazon.com/compliance/shared-responsibility-model/>
  *Why: the canonical statement of the IaaS/PaaS/SaaS responsibility split, from the largest provider in the market.*
- **The Twelve-Factor App:** <https://12factor.net/>
  *Why: the short, foundational document behind almost every PaaS platform's assumptions — read at least "III. Config," "VI. Processes" (statelessness), and "VIII. Concurrency" (horizontal scaling).*
- **Render — Docs, "What is Render?" and "Web Services":** <https://render.com/docs> · <https://render.com/docs/web-services>
  *Why: the concrete, hands-on version of Lecture 1's PaaS section, for the platform this week's walkthroughs use.*
- **PostgreSQL — High Availability, Load Balancing, and Replication (overview section):** <https://www.postgresql.org/docs/current/high-availability.html>
  *Why: the official explanation of primary/standby replication behind Lecture 3's redundancy pattern.*

## Reference (keep in tabs)

- **AWS — What Is Cloud Computing?:** <https://aws.amazon.com/what-is-cloud-computing/>
  *Why: a clear, vendor-written but broadly applicable overview of the compute/storage/networking building blocks from Lecture 2.*
- **Google Cloud — Regions and Zones:** <https://cloud.google.com/compute/docs/regions-zones>
  *Why: the clearest official explanation of the region/zone distinction that Lecture 3's redundancy design depends on.*
- **Docker — "What is a container?":** <https://docs.docker.com/get-started/docker-overview/>
  *Why: the concept behind the `Dockerfile` in Lecture 2, if you haven't used containers before.*
- **Let's Encrypt — How It Works:** <https://letsencrypt.org/how-it-works/>
  *Why: what's actually happening when your PaaS platform hands you free, auto-renewing HTTPS.*
- **AWS — Understanding Data Transfer (Egress) Costs:** <https://aws.amazon.com/blogs/architecture/overview-of-data-transfer-costs-for-common-architectures/>
  *Why: the deepest free treatment of the single most commonly underestimated cloud cost line item.*
- **PgBouncer — Wiki:** <https://wiki.postgresql.org/wiki/PgBouncer>
  *Why: the connection-pooling tool referenced in Lecture 3's database-scaling section.*

## Pricing pages (for Exercise 3, Challenge 2, and the mini-project)

- **Render — Pricing:** <https://render.com/pricing>
- **Railway — Pricing:** <https://railway.app/pricing>
- **Fly.io — Pricing:** <https://fly.io/docs/about/pricing/>
- **Neon — Pricing:** <https://neon.tech/pricing>
- **AWS Pricing Calculator** (useful even if you don't deploy to AWS this week, as a reference for realistic unit prices): <https://calculator.aws/>

## Diagramming

- **Mermaid — Live Editor** (no install, renders diagrams from text, and GitHub renders Mermaid natively in Markdown): <https://mermaid.live/>
  *Why: this is how to turn `architecture-diagram.md`'s diagram into something that actually renders as a picture on GitHub, not just ASCII art.*

## Glossary

| Term | Definition |
|------|------------|
| **IaaS** | Infrastructure as a Service — you rent raw compute/storage/networking (e.g., a VM) and manage everything above the OS yourself. |
| **PaaS** | Platform as a Service — you rent a platform that manages OS/runtime/servers; you deploy application code. |
| **SaaS** | Software as a Service — you rent a finished application; you manage only your data and configuration within it. |
| **Shared-responsibility model** | The formal split between what the cloud provider secures/operates and what you do, at any service model. |
| **Compute** | CPU/RAM execution environment: a VM, a container, or a serverless function. |
| **Managed database** | A database engine run, patched, and backed up by the provider; you get a connection string, not the underlying machine. |
| **Object storage** | Storage for unstructured files (images, PDFs, backups) addressed by key, not rows or a filesystem path — e.g., AWS S3. |
| **VPC / private network** | An isolated network segment where internal traffic (e.g., app-to-database) doesn't traverse the public internet. |
| **Load balancer** | Distributes incoming traffic across multiple app instances and routes around any instance failing its health check. |
| **Health check** | A periodic request confirming an instance is alive and able to serve traffic. |
| **Availability zone (AZ)** | An isolated data center within a cloud region, with independent power/cooling/networking. |
| **Region** | A geographic area made of multiple availability zones. |
| **Single point of failure (SPOF)** | Any one component whose failure takes down the whole system. |
| **Redundancy** | Having more than one of a critical component so a single failure doesn't cause an outage. |
| **Horizontal scaling** | Adding more instances of the same size to handle more load. |
| **Vertical scaling** | Increasing the size (CPU/RAM) of an existing instance. |
| **Statelessness** | An application design where no instance holds data the others lack — required for safe horizontal scaling and failover. |
| **Read replica** | A read-only copy of a database used to offload read traffic from the primary. |
| **Data egress** | Data transferred out of a cloud provider's network, typically billed per GB — the most commonly underestimated cost line item. |
| **Connection pooling** | Multiplexing many application connections onto fewer real database connections (e.g., via PgBouncer). |

---

*Broken link? Open an issue or PR.*
