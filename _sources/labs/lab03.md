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

# Lab 3 — Module 3: Security

**Week 3 · AWS Cloud Foundations · Module 3 · Security**
> **Prerequisite:** Module 1 + 2 (done).
Complete AWS Skill Builder "**Security in the AWS Cloud**" (Cloud Practitioner Essentials Module 3).

## Module summary
- The AWS shared security **responsibility** boundary (customer = data/identity; AWS = infrastructure).
- **AWS IAM**: users, groups, roles, policies (the "policy evaluation" order: deny wins).
- **Shared resources, MFA**, and the **AWS root account** (never use daily).

## Hands-on activity (lightweight)
```bash
# 3.1  Check you do NOT use the root account daily (best practice)
# (In the real AWS console: IAM → Users → root → "did I sign in via root?" check)

# 3.2  Policy evaluation order (Deny always wins — prove it in words)
# An explicit Allow + explicit Deny -> DENY.
# A default Deny + no matching statement -> DENY.
# Write the order:
echo "1. Default DENY (implicit) -> 2. Allow -> 3. Explicit DENY (overrides)"

# 3.3  MFA reminder (no AWS call needed)
echo "Root MFA: REQUIRED | IAM-user MFA: REQUIRED for privileged access"
```

## Artifact
`lab3-security-posture.md`: a **filled-in security posture table** you screenshot from the console
(IAM → account settings → root last sign-in + MFA status), followed by the policy-evaluation order
above.

```markdown
## GreenGrid Security Posture

| Control | Status |
| --- | --- |
| Root account never used for IAM tasks | ✓ (last root sign-in: never) |
| MFA on root | enabled |
| IAM users with console access | 0 (use role-based access) |
| Permissions boundary in place | yes (PowerUser boundary) |

## IAM policy evaluation order
Default DENY → Allow → **Explicit DENY wins**.
```

## Checkpoint

```{dropdown} Q1. "Shared responsibility" — you secure WHAT and AWS secures WHAT?
You (customer) secure: data, identity/IAM, OS, apps, network config you build.
AWS secures: physical datacenters, hypervisor, regions, managed-service infrastructure.
```
```{dropdown} Q2. Why is an IAM *role* (not a user) the right way to give an EC2 instance permission?
Roles provide **temporary, auto-rotated credentials** via STS — no long-lived keys on the host.
```
```{dropdown} Q3. Policy evaluation: a statement `Allow s3:*` AND `Deny s3:DeleteBucket` exist. Can you delete the bucket?
**No.** Explicit **Deny always wins**, regardless of any Allow.
```
```{dropdown} Q4. Your root access key was created for a script and never rotated (2 years). What's the risk + fix?
Risk: if leaked, an attacker has **full account takeover** with no boundary.
Fix: **delete the root access key**; use an IAM role with least-privilege for the script.
```
```{dropdown} Q5. IAM policies vs resource-based policies (S3 bucket policy) — one difference in who can attach.
IAM policies attach to a **principal** (user/role/group); resource-based policies attach to the
**resource itself** (bucket, queue) and name the principal.
```

## Sources
- AWS IAM User Guide: https://docs.aws.amazon.com/IAM/latest/UserGuide/
- IAM policy evaluation logic: https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html
- AWS Account best practices: https://docs.aws.amazon.com/accounts/latest/default.html
- Module 3 transcript: AWS Skill Builder "AWS Cloud Practitioner Essentials"
