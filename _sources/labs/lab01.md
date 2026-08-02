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

# Lab 1 — Module 1: Cloud Concepts

**Week 1 · AWS Cloud Foundations · Module 1 · STATUS: COMPLETED**
This module is **already done** — you may have skipped it; re-confirm the quiz score screenshot.

## Module summary (what the Skill Builder module covers)
- Cloud computing fundamentals, deployment models (public/private/hybrid),
- Economics: CapEx → OpEx, total cost of ownership (TCO),
- Characteristics of cloud computing (on-demand, elasticity, pay-as-you-go).

## Hands-on activity (lightweight)
```bash
# Estimate a TCO shift: 1 server moved to AWS
# On-prem cost: $2,400/yr hardware (3yr) + $800/yr power/rack = $10,400 3yr
# AWS cost (ap-southeast-1 t3.small, 3yr reserved): ~$3,100
python3 - <<'PY'
onprem = (2400*3 + 800*3) / 3      # per year
aws    = 3100 / 3                  # per year (reserved)
print(f"annual on-prem ~ ${onprem:,.0f}")
print(f"annual aws    ~ ${aws:,.0f}")
print(f"3yr saving     ~ ${onprem*3 - aws*3:,.0f}")
PY
```

## Artifact
`screenshot` of your AWS Skill Builder Module 1 quiz result (≥ 80%) + the TCO estimate above
saved to `lab1-tco.txt`.

## Checkpoint

```{dropdown} Q1. Name two characteristics of cloud computing that make it cheaper than owning servers.
The cloud is **on-demand** (no waste when idle) and **pay-as-you-grow** (no 3-year hardware bet).
```
```{dropdown} Q2. "CapEx vs OpEx" — why does a CFO prefer OpEx for a web startup?
Capital spend must be **justified upfront over years** (hard for a startup with no assets);
OpEx is monthly and matches revenue — no large upfront cash outlay.
```
```{dropdown} Q3. Shared tenancy vs dedicated tenancy — which is cheaper and when do you pick dedicated?
Shared is cheaper (you share the host). Pick dedicated only for **license mobility** or **regulatory**
isolation where sharing is prohibited.
```
```{dropdown} Q4. 60% of cloud spend is estimated waste — give one classic cause.
Idle or **oversized** resources (e.g., a t2.2xlarge left on 24×7 doing nothing, or unattached EBS).
```
```{dropdown} Q5. TCO calculator compares AWS to on-prem — name one assumption it always asks for.
The **number of users/transactions** (throughput) — without workload sizing you can't model cost.
```

## Sources
- AWS TCO Calculator: https://aws.amazon.com/tco-calculator/
- NIST cloud definition: https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-145.pdf
- Module 1 transcript: AWS Skill Builder "AWS Cloud Practitioner Essentials"
