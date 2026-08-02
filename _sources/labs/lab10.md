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

# Lab 10 — Module 10: Incident Response

**Week 10 · AWS Cloud Operations · Module 10 · Incident Response**
Complete AWS Skill Builder "**Incident response on AWS**" + activity.

## Module summary
- **NIST IR lifecycle** applied to AWS: Identify → Contain → Eradicate → Recover → Lessons Learned.
- **IAM break-glass** procedures, **quarantine SGs**, **isolate** via SCPs / OU moves.
- **CloudTrail + CloudWatch** for IR evidence and automated **event-driven containment**
  (EventBridge rule → Lambda quarantine).

## Hands-on activity — write the runbook table
```bash
# 10.1  Fill in the IR playbook actions (one per phase, no AWS call)
cat > /tmp/gg-ir-runbook.tsv <<'TSV'
Phase	Action in AWS	Who
Identify	List CloudTrail events for suspicious IP/user	IAM auditor
Contain	Detach SG/attach quarantine SG/NACL	SecOps lead
Eradic	ate	Delete leaked IAM key + force password reset	IAM admin
Recover	Re-launch from AMIs / restore from backups	Ops lead
Lessons	L	Rename CloudTrail trail + enable data events on sensitive S3	IAM auditor
TSV

# 10.2  Break-glass procedure (concept)
echo "Break-glass: MFA-delete disabled on a pre-created 'breakglass' IAM user with AdministratorAccess, password stored encrypted."
```

## Artifact
`/tmp/gg-ir-runbook.tsv` (filled in) + screenshot of a **CloudTrail event** for an
unauthorized `iam:CreateAccessKey`, plus the Module 10 quiz (≥ 80%).

## Checkpoint

`````{dropdown} Q1. IR phase "Contain" on AWS — name two network-level actions.
(1) Replace the compromised instance's **security group** with a *quarantine SG*
(deny-all except a forensically controlled IP), (2) update the **subnet NACL** to
block egress to the internet.
````{dropdown} Q2. Why disable IAM access keys found in public GitHub *before* rotating them?
So the leaked key, if still working, cannot be used during the window between
detection and rotation — **revoke-then-rotate**; also rotate the **credentials of
the user it was stolen from**.
```
``{dropdown} Q3. EventBridge rule: `events Put` on `CreateNetworkInterface` → Lambda quarantine — what does this buy you?
**Automated, near-real-time containment**: any new ENI not from an approved ASG triggers a Lambda that
moves the instance into a quarantine SG — no on-call sleep needed for the first 60s of an attack.
```
`````{dropdown} Q4. Forensic image of an EC2 — which AWS action preserves evidence without changing the disk?
**Create an EBS snapshot** of the running volume (consistent) **and** note the volume is attached —
**do not detach** (detach changes the running system). Then mount the *snapshot* read-only for analysis.
``//``
`````{dropdown} Q5. Your "lessons learned" recommends enabling data events on S3 — but Lab 9 warned they cost. Conflict?
No — enable data events **only on the breach-sensitive prefix** (`s3://gg-payments/*`), not all buckets,
so cost stays bounded while coverage closes the forensic gap.
`````

## Sources
- AWS Incident Response Guide: https://docs.aws.amazon.com/whitepapers/latest/aws-best-practices-incident-response/
- AWS Security Hub: https://docs.aws.amazon.com/securityhub/latest/userguide/
- CloudTrail: https://docs.aws.amazon.com/awscloudtrail/latest/userguide/
- NIST SP 800-61 (Computer Security Incident Handling Guide)
- Module 10: AWS Skill Builder "Incident response on AWS"
