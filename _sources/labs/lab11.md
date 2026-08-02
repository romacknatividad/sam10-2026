---
jupytext:
  text_representation:
    extension: .md
    format_name: myst
kernelspec:
  display_name: Python 3
  language: python
  name: python3
---

# Lab 11 — Module 11: Backup & Disaster Recovery

**Week 11 · AWS Cloud Operations · Module 11 · Backup & DR**
Complete AWS Skill Builder "**Backup and restore on AWS**" + activity.

## Module summary
- **RPO / RTO** definitions and how each service delivers them.
- **AWS Backup** (central policy across RDS/EFS/EBS/DynamoDB) vs service-native snapshots.
- **Cross-region copy** for DR + **point-in-time restore (PITR)**.
- **DR strategies**: Backup-and-restore (cheap, slow), Pilot Light, Warm Standby, Multi-site (fast,
  expensive).

## Hands-on activity — build the RPO/RTO table
```bash
# 11.1  Map service → native RPO/RTO (no AWS call)
cat > /tmp/gg-rpo-rto.tsv <<'TSV'
Service	RPO (default)	RTO (typical)	Mechanism
RDS	Continuous (1s)	Minutes	PITR / automated snapshots
DynamoDB	Global table / PITR	~5 min	~minutes	PITR / backup-restore
EBS	Snapshot (1x/day)	Hours	ebs snapshots (manual)
S3	11 nines durability (versioning)	N/A	Cross-region replication (CRR)
EFS	Continuous	~minutes	Built-in backup + lifecycle
TSV
python3 -c "import csv; rows=list(csv.DictReader(open('/tmp/gg-rpo-rto.tsv'),delimiter='\t')); print('rows:',len(rows)); print('services:',[r['Service'] for r in rows])"

# 11.2  DR strategy decision (scenario)
echo "Orders microservice (need <5min RPO, <1hr RTO, global) -> Multi-region active-active (DynamoDB global + ALB + Route53)"
```

## Artifact
`/tmp/gg-rpo-rto.tsv` + a 2-cell **decision table** for the "Orders microservice" DR strategy
strategy chosen + justification, plus the Module 11 quiz (≥ 80%).

## Checkpoint

`````{dropdown} Q1. RPO=5min, RTO=1hr, workload is global — which DR strategy from Lab 14?
**Warm standby** (or Multi-region if budget high): a **hot standby in another region** lets you
meet RTO < 1 h and RPO ~5 min, cheaper than fully-active-active.
```
````{dropdown} Q2. Why does an Amazon Data Lifecycle Manager (DLM) policy on EBS beat cron `create-snapshot`?
DLM **runs server-side on a managed schedule, tags snapshots, and deletes them by retention** — cron
needs an instance to stay alive and your own rotation code.
`````
``{dropdown} Q3. You restore an RDS from a snapshot — does encryption-at-rest key carry over?
Yes: a snapshot is **encrypted with the same KMS key** as the source; restoring to the same region
re-uses that key (no extra permission).
```
``{dropdown} Q4. S3 CRR vs S3 Glacier for long-term compliance — one difference for a 7-year retention?
CRR **keeps objects accessible/available** across regions (still S3 Standard); Glacier moves
them to **archival** (you must restore 1–5 min before a retrieval) — Glacier is cheaper long-term
for 7-year.
```
````{dropdown} Q5. "Backup and restore" DR strategy — what's the hidden RTO risk at 2 AM?
If backups are in **cold storage (Glacier Deep Archive)** you restore takes **hours**, so your 1-hr RTO
SLA is blown — the plan must declare which bucket/policy backs which RTO tier.
````

## Sources
- AWS Backup User Guide: https://docs.aws.amazon.com/backup/latest/devguide/
- Amazon RDS point-in-time recovery: https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/
- AWS Disaster Recovery options: https://docs.aws.amazon.com/whitepapers/latest/aws-disaster-recovery-workloads-on-aws/welcome.html
- AWS Well-Architected — Reliability pillar (DR chapter)
- Module 11: AWS Skill Builder "Backup and restore on AWS"
