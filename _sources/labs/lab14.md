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

# Lab 14 — Module 14: Well-Architected Reviews

**Week 14 · AWS Cloud Operations · Module 14 · Well-Architected Reviews**
Complete AWS Skill Builder "**AWS Well-Architected Framework**" + activity.

## Module summary
- The **six pillars**: Operational Excellence, Security, Reliability, Performance Efficiency,
  Cost Optimization, and **Sustainability** (2023 addition).
- **Well-Architected Tool** in the console: define a workload, answer the pillar questions,
  get a **risk** + **best-practice** review.
- **Improvement plan** with prioritized **best-practice names** (not just "it's risky").
- **Lenses** (Serverless, IoT, HPC) extend the base questions for domain-specific workloads.

## Hands-on activity — fill in the pillar scorecard
```bash
# 14.1  Six pillars + one question you'd ask in a review
cat > /tmp/gg-wa-scores.tsv <<'TSV'
Pillar	Due diligence question (you ask the team)
Operational Excellence	"Do we have runbooks + a post-mortem culture?"
Security	                "Are IAM least-privilege + MFA enforced everywhere?"
Reliability	            "Do we test failover (not just Multi-AZ)?"
Performance Efficiency	"Do we right-size and use auto-scaling, not over-provision?"
Cost Optimization	        "Can we tag + budget + stop idle resources?"
Sustainability	        "Can we right-size to fewer hosts / use Graviton?"
TSV
python3 -c "import csv; rows=list(csv.DictReader(open('/tmp/gg-wa-scores.tsv'),delimiter='\t')); print('pillars:',len(rows)); print([r['Pillar'] for r in rows])"
```

## Artifact
`/tmp/gg-wa-scores.tsv` + screenshot of a **Well-Architected workload review** (you define a
workload and capture at least one **HIGH**-risk finding), plus the Module 14 quiz (≥ 80%).

## Checkpoint

`````{dropdown} Q1. Well-Architected has 6 pillars — which is **not** one of them?
(A) Fault Tolerance  (B) Security  (C) Operational Excellence  (D) Sustainability
**Answer: A** — "Fault Tolerance" is a **subtopic** under **Reliability**; the six named pillars are
OpEx, Security, Reliability, Performance, Cost, Sustainability.
```
`````{dropdown} Q2. "Risk: HIGH, question: do you use IAM groups for access?" — remediation direction?
Move to **IAM roles + SSO/permission sets** (no long-lived users, no shared groups); groups are
fine for *permission boundaries* but the finding means users are being granted access directly.
`````
`````{dropdown} Q3. A workload review says the DB has "no Multi-AZ / no read replica" — Risk?
**HIGH Reliability** finding (single point of failure) — fix by enabling Multi-AZ, then test a
manual failover (see Lab 12 DR testing).
`````
``{dropdown} Q4. Serverless lens vs base — one extra question you'd get?
"What is the **concurrency reservation / burst** behavior of your Lambda?" — the base lens won't
ask about Lambda cold-starts, provisioned concurrency, or function-level IAM roles.
```
````{dropdown} Q5. A finding says "Sustainability: hosts are 80% utilized" — is that good or bad?
**Bad.** Underutilized servers waste **embodied carbon + electricity**; right-size (fewer, bigger)
or move to Serverless to improve **PUE** and utilization → that *is* the sustainability lever.
````

## Sources
- AWS Well-Architected Framework: https://aws.amazon.com/architecture/well-architected/
- AWS Well-Architected Tool: https://docs.aws.amazon.com/wellarchitected/latest/ug/
- Well-Architected Sustainability pillar: https://d1.aws/static/WhitePaper/
- AWS Well-Architected Lenses page: https://aws.amazon.com/architecture/well-architected/lenses/
- Module 14: AWS Skill Builder "AWS Well-Architected Framework"
