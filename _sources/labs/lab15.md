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

# Lab 15 — Module 15: Automation & DevOps

**Week 15 · AWS Cloud Operations · Module 15 · Automation & DevOps**
Complete AWS Skill Builder "**DevOps on AWS**" + activity. This is the final
lab — it ties together everything from the semester.

## Module summary
- **CI/CD**: CodeCommit → CodeBuild → CodeDeploy → CodePipeline.
- **Infrastructure as Code** (CloudFormation / CDK / Terraform) — the "one source of truth."
- **GitOps**: repo as the system-of-record; any change = PR → pipeline → deploy.
- **Observability + runbooks** as code (not wiki pages that rot).

## Hands-on activity — design a CI/CD pipeline diagram
```bash
# 15.1  Draw the pipeline as a TSV (no AWS call — design exercise)
cat > /tmp/gg-pipeline.tsv <<'TSV'
Stage	Tool	Input	Output
Source	CodeCommit	PR merged to main	Source artifact
Build	CodeBuild	Source artifact	Docker image pushed to ECR
Test	CodeBuild (pytest)	Docker image	Test pass/fail
Deploy	CodeDeploy + ALB	Docker image	Green hosts live
Verify	CloudWatch alarm	Green hosts	Rollback if 5xx > threshold
TSV
python3 -c "import csv; rows=list(csv.DictReader(open('/tmp/gg-pipeline.tsv'),delimiter='\t')); print('stages:',len(rows)); [print(r['Stage'],'->',r['Tool']) for r in rows]"

# 15.2  GitOps principle (concept)
echo "GitOps: the git repo is the SINGLE SOURCE OF TRUTH. Any infra change = PR."
echo "No console clicks. No manual ssh. Every change is auditable + reversible."
```

## Artifact
`/tmp/gg-pipeline.tsv` + a **one-page DevOps maturity self-assessment** you write:

```markdown
## GreenGrid DevOps Maturity

| Practice	| Current	| Target (semester end)
| --- | --- | --- |
| IaC for all infra	| partial (CloudFormation templates exist) | 100% — every resource in a template
| CI/CD pipeline	| manual deploy from console | CodePipeline with auto-rollback
| Monitoring	| CloudWatch dashboards exist | Alarms auto-trigger runbooks
| Runbooks	| markdown files in repo | executable runbooks (Lambda + SSM)
| GitOps	| some infra in git	| all infra in git, PR-gated
```

## Checkpoint

````{dropdown} Q1. "GitOps" — what is the single source of truth?
The **git repository**. Infrastructure state is declared in templates stored in git;
the pipeline reconciles the live AWS state to match the git state.
````
````{dropdown} Q2. CodeBuild vs CodeDeploy — which builds, which deploys?
**CodeBuild** compiles/tests the artifact (produces the Docker image or zip).
**CodeDeploy** takes the artifact and **deploys it to the target** (EC2, Lambda, ECS).
````
``{dropdown} Q3. A pipeline has no rollback step — what's the risk?
A bad deploy stays live with **no automatic revert**; the blast radius is the full
production user base until a human manually rolls back (minutes to hours of downtime).
```
````{dropdown} Q4. Terraform vs CloudFormation — one trade-off?
Terraform is **cloud-agnostic** (multi-cloud) and has a richer module ecosystem;
CloudFormation is **AWS-native**, integrates deeper (StackSets, Drift Detection),
and is the default for AWS-only shops.
````
````{dropdown} Q5. "Runbook as code" vs "runbook as wiki" — one advantage?
A code runbook is **executable and testable** (you can run it in CI to prove it still
works); a wiki page rots silently and no one trusts it at 3 AM during an incident.
````

## Sources
- AWS CodePipeline: https://docs.aws.amazon.com/codepipeline/latest/userguide/
- AWS CodeBuild: https://docs.aws.amazon.com/codebuild/latest/userguide/
- AWS CodeDeploy: https://docs.aws.amazon.com/codedeploy/latest/userguide/
- GitOps pattern: https://www.gitops.com/
- AWS DevOps blog: https://aws.amazon.com/blogs/devops/
- Module 15: AWS Skill Builder "DevOps on AWS"
