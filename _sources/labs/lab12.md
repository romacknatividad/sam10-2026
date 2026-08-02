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

# Lab 12 — Module 12: Cost Optimization

**Week 12 · AWS Cloud Operations · Module 12 · Cost Optimization**
Complete AWS Skill Builder "**Cost optimization on AWS**" + activity.

## Module summary
- **TCO pillars**: right-size, choose the right purchase option, delete unused resources.
- **Savings Plans** (1–3 yr, compute) vs **Reserved Instances** (specific instance-family).
- **Tagging + Cost Explorer** + **Budgets** for visibility and anomaly detection.
- **Compute Optimizer** recommendations vs manual `top`-style metrics.

## Hands-on activity — optimize a mock bill
```bash
# 12.1  Spot + Reserved math (no AWS call)
python3 - <<'PY'
# 10 x t3.medium On-Demand $0.0416/hr * 730h = $30.37/mo each -> $303.7 total
ondemand = 10 * 0.0416 * 730
# Same with Compute Savings Plan 3yr (~$0.025/hr)
sp       = 10 * 0.025 * 730
# Spot at 70% discount (risk modeled separately)
spot     = ondemand * 0.30
print(f"On-Demand : ${ondemand:,.2f}/mo")
print(f"SavePlan  : ${sp:,.2f}/mo  (save ~${ondemand-sp:,.0f})")
print(f"Spot (70%): ${spot:,.2f}/mo (risk: 2-min kill)")
PY

# 12.2  Budget alarm (concept)
echo "Budget: monthly $500, alert at 50/80/100% via SNS, action stops dev resources at 100%"

# 12.3  Unused-resource audit checklist
cat > /tmp/gg-cost-audit.tsv <<'TSV'
Check	Tool
Idle EC2 (CPU<10% 14d)	Compute Optimizer / CloudWatch
Unattached EBS volumes	EC2 → Volumes (status=available)
Underutilized ELB		CloudWatch RequestCount<1/day
Idle Elastic IP		EC2 → Elastic IPs (not attached)
Old EBS snapshots		DRS / Backup reports
TSV
```

## Artifact
`/tmp/gg-cost-audit.tsv` + screenshot of a **Cost Explorer** monthly-running-cost graph with a
**budget alert** created, plus the Module 12 quiz (≥ 80%).

## Checkpoint

`````{dropdown} Q1. Compute Savings Plan vs Reserved Instance — which buys flexibility?
**Compute Savings Plan**: applies to *any* EC2 + Fargate usage (no family lock-in). An **RI**
locks you to an instance *family/family-size*; cheaper but less flexible.
```
``{dropdown} Q2. Spot interrupted with 2-min notice — what's the safe workload?
**Stateless, checkpointable** work (batch transform, CI builds, HPC jobs that checkpoint to S3).
Never use Spot for a stateful primary DB without a multi-AZ design.
```
``{dropdown} Q3. Cost Explorer group-by `TAG:Owner` returns blank rows — likely cause?
A resource with **no tags** (or the tag key differs by case) — you can't attribute untagged spend
to a cost center → enforce a **default-tagging baseline** at launch.
```
````{dropdown} Q4. `aws ce get-reservation-coverage` returns 68% — what does that mean for your RI?
Only 68% of your eligible usage is covered by **reserved** spend → 32% runs On-Demand (overspend).
Add more RIs/Savings Plans or right-size the reservation.
````
````{dropdown} Q5. Idle EIP costs but it IS attached to a stopped instance — is it free?
**No**: an EIP attached to a **stopped NAT/instance** still charges (hours of association), but
**detached** EIPs are free — the rule is "attached and running" must hold, else detach.
````

## Sources
- AWS Economics Center: https://aws.amazon.com/economics/
- AWS Cost Explorer: https://docs.aws.amazon.com/cost-reports/latest/user/get-started/
- AWS Budgets: https://docs.aws.amazon.com/cost-management/aws-budgets/
- Compute Optimizer: https://docs.aws.amazon.com/compute-optimizer/latest/ug/
- Well-Architected — Cost Optimization pillar
- Module 12: AWS Skill Builder "Cost optimization on AWS"
