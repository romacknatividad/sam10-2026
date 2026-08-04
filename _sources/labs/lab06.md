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

# Lab 6 — Module 6: Databases & Analytics

**Week 6 · AWS Cloud Foundations · Module 6 · Databases & Analytics**
Complete AWS Skill Builder Module 6 + activity. This closes the Foundations half.

## Module summary
- Managed relational: **RDS** (MySQL/PostgreSQL/MariaDB/Oracle/SQL Server) — automated
  backups, Multi-AZ, read replicas.
- Managed NoSQL: **DynamoDB** (key-value/document), DAX (accelerator), on-demand vs provisioned.
- Analytics: **Redshift** (data warehouse), **EMR** (Hadoop/Spark), ** Athena** (ad-hoc SQL on S3).

## Hands-on activity
```bash
# 6.1  Match the engine to the use case (write-only; no provisioning needed)
cat > lab6-db-picks.tsv <<'TSV'
Use case	Service	Why
user sessions + shopping cart	DynamoDB	key-value, single-digit ms
relational app (JOINs)	RDS PostgreSQL	SQL, ACID
data warehouse reporting	Redshift	OLAP, columnar
ad-hoc SQL on S3 logs	Athena	pay-per-query, no cluster
real-time recommendations	DynamoDB + DAX	<10ms reads, in-memory cache
TSV

# 6.2  RDS backup types (name them)
# Automated snapshots (daily, kept by retention) + Manual snapshots (until deleted)
# Point-in-time restore (PITR): seconds RPO within retention window.
echo "RDS automated = time-based backup + transaction log shipping (continuous PITR)"

# 6.3  DynamoDB billing: on-demand vs provisioned math
python3 - <<'PY'
# on-demand: $1.25/million WCU, ideal for spiky/unknown traffic
# provisioned: $0.0065/WCU-hour, ideal if you reserve (Compute Savings Plan)
writes_expected_per_month = 5_000_000
print(f"on-demand writes $ = {writes_expected_per_month/1_000_000*1.25:.2f}/mo")
print(f"provisioned 50 WCU $ = {50*0.0065*24*30:.2f}/mo (no traffic assumed)")
PY
```

## Artifact
`lab6-db-picks.tsv` + screenshot of the Module 6 quiz (≥ 80%).

## Checkpoint

``{dropdown} Q1. RDS "automated backup retention" window — what two values matter?
The **backup retention period** (1–35 days) and the **preferred window** (when the daily
backup runs). Together they define your PITR window.
```
```{dropdown} Q2. DynamoDB on-demand vs provisioned — which protects you from a sudden traffic spike?
**On-demand** (no capacity planning; you pay per request and scale automatically). Provisioned
without auto-scaling → 400-throttled until you raise the limit.
```
````{dropdown} Q3. Redshift vs Athena — both do SQL on large data. Two differences?
- Redshift = **provisioned cluster** (you manage nodes); Athena = **serverless** (pay per query).
- Redshift = loaded via COPY from S3 (expensive ingest); Athena = queries S3 **in place**.
````
``//``{dropdown} Q4. RDS Multi-AZ is for HA, not read scaling — what gives read-scaling reads?
A **Read Replica** (same engine). Multiple replicas take the read load off the primary.
``//``
``{dropdown} Q5. S3 Select vs Athena — both filter S3. Scope difference?
**S3 Select** returns rows from **one object** via an API call (cheap, one file).
**Athena** runs SQL across **all objects in a table** (S3 prefix) and returns columns/aggregates.
`````

## Sources
- Amazon RDS: https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/
- Amazon DynamoDB: https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/
- Amazon Redshift: https://docs.aws.amazon.com/redshift/latest/mgmt/welcome.html
- Amazon Athena: https://docs.aws.amazon.com/athena/latest/ug/
- Module 6 transcript: AWS Skill Builder "AWS Cloud Practitioner Essentials"
