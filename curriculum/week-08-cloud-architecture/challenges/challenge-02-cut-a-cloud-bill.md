# Challenge 2 — Cut a Cloud Bill

**Time:** ~90 minutes. **Difficulty:** Medium-hard. **Target: cut the bill by 50% or more without reducing production capability.**

## The scenario

A sister team at Crunch Cycles — not yours, a different product line that also runs on the cloud — has been live for eight months and nobody has looked closely at the bill since launch week. It's grown to **$2,542/month** and the CFO wants to know why a system this size costs that much. You've been asked to find out, using nothing but the itemized line items and Lecture 3's cost-trap list as your guide.

This is deliberately an **analysis** challenge, not a provisioning one: the "cloud bill" is a table of data. You'll load it into a real database and use SQL to find the waste — the same skill from Weeks 1–4, aimed at a new kind of table.

## Setup

Load this into PostgreSQL or SQLite (works unchanged on both):

```sql
CREATE TABLE cloud_bill_line_items (
    line_item_id    INTEGER PRIMARY KEY,
    resource_name   TEXT    NOT NULL,
    resource_type   TEXT    NOT NULL,   -- 'compute' | 'db_storage' | 'egress' | 'backup' | 'load_balancer' | 'unattached_disk'
    environment     TEXT    NOT NULL,   -- 'production' | 'staging' | 'dev'
    region          TEXT    NOT NULL,
    monthly_cost    NUMERIC NOT NULL,
    avg_cpu_pct     NUMERIC,            -- NULL where not applicable (e.g. storage/egress lines)
    last_active_at  DATE,               -- NULL = still actively used; a date = when it was last touched
    notes           TEXT
);

INSERT INTO cloud_bill_line_items VALUES
(1, 'api-prod-1',            'compute',         'production', 'us-east-1', 145.00, 62, NULL,         'Primary production API instance'),
(2, 'api-prod-2',            'compute',         'production', 'us-east-1', 145.00, 58, NULL,         'Second production API instance, load-balanced with #1'),
(3, 'api-staging',           'compute',         'staging',    'us-east-1', 290.00,  4, NULL,         'Staging environment — same instance size as production, runs 24/7'),
(4, 'api-dev-old',           'compute',         'dev',        'us-east-1', 145.00,  1, '2024-11-02', 'Dev instance from a project that shipped 3 months ago'),
(5, 'db-prod',               'db_storage',      'production', 'us-east-1', 180.00, NULL, NULL,       '220 GB production database'),
(6, 'db-staging',            'db_storage',      'staging',    'us-east-1', 180.00, NULL, NULL,       '210 GB staging database — nearly the same size as production'),
(7, 'db-analytics-replica',  'compute',         'production', 'eu-west-1', 145.00, 71, NULL,         'Read replica for reporting, in a DIFFERENT region than db-prod'),
(8, 'egress-prod-to-users',  'egress',          'production', 'us-east-1', 210.00, NULL, NULL,       'Normal API response traffic to end users'),
(9, 'egress-prod-to-replica','egress',          'production', 'us-east-1', 410.00, NULL, NULL,       'Continuous replication traffic from db-prod (us-east-1) to db-analytics-replica (eu-west-1)'),
(10,'backup-prod-daily',     'backup',          'production', 'us-east-1',  35.00, NULL, NULL,       '30-day rolling backup retention, as intended'),
(11,'backup-prod-legacy',    'backup',          'production', 'us-east-1', 260.00, NULL, NULL,       '18 months of accumulated snapshots, retention policy never configured'),
(12,'lb-prod',               'load_balancer',   'production', 'us-east-1',  25.00, NULL, NULL,       'Production load balancer, in active use'),
(13,'lb-old-service-a',      'load_balancer',   'production', 'us-east-1',  25.00, NULL, '2024-08-15','Load balancer for a service decommissioned in August'),
(14,'lb-old-service-b',      'load_balancer',   'production', 'us-east-1',  25.00, NULL, '2024-09-01','Load balancer for a service decommissioned in September'),
(15,'disk-orphan-1',         'unattached_disk', 'production', 'us-east-1',  40.00, NULL, '2024-10-10', 'Disk volume left behind after its instance was terminated'),
(16,'disk-orphan-2',         'unattached_disk', 'dev',        'us-east-1',  22.00, NULL, '2024-07-22', 'Another orphaned disk from an old dev instance'),
(17,'log-storage-unbounded', 'db_storage',      'production', 'us-east-1', 260.00, NULL, NULL,       'Application logs, no retention/expiry policy ever set, growing every day');
```

Sanity check — this should print `2542.00`:

```sql
SELECT SUM(monthly_cost) FROM cloud_bill_line_items;
```

## Your task

Produce `challenge-02.md` with three sections, backed by the SQL and Python you actually ran.

### Section 1 — Find the waste (SQL)

Write and run SQL queries — not eyeballing the table — to identify **every** category of waste from Lecture 3, Section 4's trap list that's present in this bill. For each finding, show the query and its result. At minimum, find:

1. Resources that are clearly **unused or idle** (hint: `last_active_at IS NOT NULL`, or a suspiciously low `avg_cpu_pct` on a compute line).
2. A **staging or dev environment sized like production** running 24/7 when its utilization doesn't justify it.
3. **Unattached/orphaned disks** — cost with nothing using them.
4. **Cross-region traffic** costing money that same-region traffic wouldn't (tie two specific line items together, using their `notes` and `region` columns, to show the causal link).
5. **Backup/log retention that was never bounded**, ballooning past what a sane policy would keep.

### Section 2 — Propose specific cuts

For each finding in Section 1, propose one specific, concrete change (not "reduce costs" — an actual action: "delete resource X," "resize Y from A to B," "move Z to the same region as W," "set a 30-day retention policy on log storage"). Note anywhere you're trading off something real (e.g., a smaller staging environment might mean staging tests run slower, or take longer to load-test at production scale) — a strong answer acknowledges the tradeoff instead of pretending every cut is free.

### Section 3 — Recompute the bill in Python

Write `recompute_bill.py` that starts from the original line items (hard-code them as a list of dicts or read them back from the table via `psycopg2`/`sqlite3` — your choice) and applies your Section 2 cuts programmatically — remove deleted resources, adjust costs for resized/moved ones — then prints the new total and the percentage reduction.

```python
# recompute_bill.py — sketch of the shape; fill in your actual cuts
original_total = 2542.00

# Example structure — replace with YOUR specific findings and numbers
cuts = [
    {"line_item": "lb-old-service-a", "action": "delete — decommissioned service", "savings": 25.00},
    {"line_item": "lb-old-service-b", "action": "delete — decommissioned service", "savings": 25.00},
    # ... your remaining cuts, each with a one-line justification
]

total_savings = sum(cut["savings"] for cut in cuts)
new_total = original_total - total_savings
pct_reduction = (total_savings / original_total) * 100

print(f"Original total:   ${original_total:,.2f}")
print(f"Total savings:    ${total_savings:,.2f}")
print(f"New total:        ${new_total:,.2f}")
print(f"Reduction:        {pct_reduction:.1f}%")
```

## Constraints

- **Every cut must be justified by a specific finding from Section 1** — no cut without a query result backing it.
- Do not cut anything actually load-bearing for production without a stated replacement (e.g., you cannot just delete `db-prod`).
- Target at least a **50% reduction** in total monthly cost. If you can't hit 50% without touching production capacity, say explicitly which remaining costs you consider justified and why 50% isn't achievable without a real tradeoff — that's a legitimate answer too, as long as it's argued, not just asserted.

## How success is judged

| Signal | Weak answer | Strong answer |
|--------|-------------|----------------|
| Evidence | Cuts based on eyeballing the table | Every cut backed by a SQL query and its result |
| Specificity | "Reduce staging costs" | "Resize `api-staging` from the production instance size to a size matched to its 4% CPU utilization" |
| Cross-region reasoning | Missed entirely | Explicitly connects `db-analytics-replica`'s region to the `egress-prod-to-replica` line item's cost |
| Tradeoff honesty | Claims every cut is free | States what's given up, if anything, for at least one cut |
| Verification | Recomputed total stated but not derived from code | `recompute_bill.py` actually runs and prints a total matching the write-up |

## Submission

Commit `challenge-02.md` and `recompute_bill.py` to your portfolio under `c37-week-08/challenge-02/`.
