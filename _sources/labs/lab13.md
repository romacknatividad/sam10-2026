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

# Lab 13 — Module 13: Deployment Strategies

**Week 13 · AWS Cloud Operations · Module 13 · Deployment Strategies**
Complete AWS Skill Builder "**DevOps: Deployment strategies**" (or "CI/CD on AWS").
+ activity.

## Module summary
- **Deployment patterns**: In-place, Blue/Green, Canary, Linear (CodeDeploy).
- **CodeDeploy** with **ALB** target groups for traffic shifting.
- **CloudFront functions / Lambda@Edge** for client-side canary.
- **Rollback criteria** + health checks (ELB / custom).

## Hands-on activity — design the canary plan
```bash
# 13.1  Canary plan written out (no AWS call)
cat > /tmp/gg-deploy-plan.md <<'MD'
## Orders service — canary deploy plan (CodeDeploy + ALB)

1. Pre: ALB target group `orders-blue` (current) + `orders-green` (new).
2. Step 1 (10%): shift 10% of traffic to green; watch 5-min window.
3. Metrics: 5xx < 1% AND p95 latency < 200 ms AND target-HTTP-5xx < 1.
4. Step 2 (50%): after 15 min stable, shift 50%.
5. Step 3 (100%): after 15 min stable, deregister `orders-blue`.
6. Rollback: on any failure, **immediate 100% back to blue** + page on-call.

RTO goal: full deploy ≤ 20 min; rollback ≤ 30 s.
MD
python3 -c "p='/tmp/gg-deploy-plan.md'; assert 'rollback' in open(p).read().lower(); print('deploy plan has rollback:', True)"

# 13.2  Blue/Green vs Canary (concept)
echo "Blue/Green: all-or-nothing switch (fast, but big blast radius)."
echo "Canary: gradual (small blast radius, but longer to full)."
```

## Artifact
`/tmp/gg-deploy-plan.md` (the canary plan with rollback) + screenshot of a **CloudFront deployment**
config or CodeDeploy group, plus the Module 13 quiz (≥ 80%).

## Checkpoint

``{dropdown} Q1. Blue/Green cutover on an ALB — how do you "switch traffic" in one API call?
Change the **ALB listener rule's `forward` action** to point at the green target group
(`modify-listener` with `--default-actions`) — traffic flips instantly to green.
`````{dropdown} Q2. Canary 10% → 50% → 100% — what AWS feature automates these steps?
**AWS CodeDeploy** with an **ALB target group** and a `Canary10Percent5Minutes` / `Canary10Percent15Minutes`
**traffic routing config** — it shifts traffic without you scripting each step.
```
`````{dropdown} Q3. A canary step shows 5xx rising but it's below threshold — do you roll back?
**No** — you **halt** the promotion and watch; rollback triggers on the *configured* threshold breach.
Below threshold → keep monitoring the canary cohort.
`````
``{dropdown} Q4. Lambda canary on traffic — how do you do a safe 101% (test 100% then full)?
Use **alias routing** (`lambda update-alias --routing-config`) to send 100% to the NEW version while
keeping the OLD as a fallback weight for 30 min — if errors spike, **shift weight back to $LATEST**.
````{dropdown} Q5. "Flashing back" to blue after rollback — what must the data layer support?
**Backward-compatible schema** + idempotent writes: rolling back code that wrote a *new column*
must not crash the old code reading the table.
````

## Sources
- AWS CodeDeploy: https://docs.aws.amazon.com/codedeploy/latest/userguide/
- AWS CodePipeline: https://docs.aws.amazon.com/codepipeline/latest/userguide/
- AWS Lambda alias routing: https://docs.aws.amazon.com/lambda/latest/dg/configuration-aliases.html
- AWS Well-Architected — Deployments (reliability + change management)
- Module 13: AWS Skill Builder "DevOps: Deployment strategies"
