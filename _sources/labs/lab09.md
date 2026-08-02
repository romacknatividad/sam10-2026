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

# Lab 9 — Module 9: Logging & Auditing

**Week 9 · AWS Cloud Operations · Module 9 · Logging & Auditing**
Complete AWS Skill Builder "**Audit logging with AWS CloudTrail**" + activity.

## Module summary
- **CloudTrail** records account activity (API calls, who did what, where).
- **Management events** vs **Data events** (S3 object-level, Lambda invoke).
- **CloudWatch Logs integration** (send trails to a log group for retention/query).
- **AWS Config** (configuration changes + conformance packs) vs CloudTrail (events).

## Hands-on activity — inspect a trail policy locally
```bash
# 9.1  The trail must log multi-region + management + data events
# Write the JSON you would attach as a trail policy (validate locally):
cat > /tmp/gg-trail.json <<'JSON'
{
  "Version": "2012-10-17",
  "Statement": [
    {"Effect": "Allow", "Action": ["s3:GetBucketAcl","s3:PutBucketAcl"],
     "Resource": "arn:aws:s3:::gg-cloudtrail-*/"},
    {"Effect": "Allow", "Action": ["s3:GetBucketLocation"], "Resource": "arn:aws:s3:::*"}
  ]
}
JSON
python3 -c "import json; json.load(open('/tmp/gg-trail.json')); print('trail JSON parse OK')"

# 9.2  Log-retention plan (concept — no call needed)
echo "CloudTrail logs in S3 -> lifecycle: transition to IA after 30d, Glacier after 90d, delete after 7y"
```

## Artifact
`/tmp/gg-trail.json` + screenshot of a **CloudTrail event history** filtered to a `ConsoleLogin`
event (showing the source IP + result), plus the Module 9 quiz (≥ 80%).

## Checkpoint

``{dropdown} Q1. CloudTrail is on by default but logging to where? Where must you enable logging?
The default trail logs to a **new S3 bucket you name** and only the **region where you create it**;
you must enable **multi-region** to capture `ap-southeast-1` etc. if your account is `us-east-1`.
```
````{dropdown} Q2. One sentence difference: CloudTrail vs AWS Config.
CloudTrail logs **who did what when** (point-in-time events); AWS Config records **what
configurations changed** over time + drift + conformance.
````
``{dropdown} Q3. You delete an S3 bucket that held last 90 days of CloudTrail logs (no replication). What's lost?
**90 days of audit history** → a deleted object leaves a record of the delete event, but **before
that there are no logs** for other buckets' activity that wasn't replicated elsewhere.
```
````{dropdown} Q4. Data events on S3 are billed per 10,000 requests — why not leave them ON everywhere?
Because high-traffic buckets (e.g., your static website) generate **billions** of GetObject data
events → enable **only for sensitive prefixes** (e.g., `s3://gg-financial/*`).
````
``{dropdown} Q5. `eventCategory` in the S3 object log — what two values help a forensic?
`Management` events (the API call that **deleted/mutated** the object) + **object-level data events**
(let you see the Get/Put that led up to the event, with the source IP + user agent).
`````

## Sources
- AWS CloudTrail User Guide: https://docs.aws.amazon.com/awscloudtrail/latest/userguide/
- AWS Config: https://docs.aws.amazon.com/config/latest/developerguide/
- AWS Config conformance packs: https://docs.aws.amazon.com/config/latest/developerguide/
- Module 9: AWS Skill Builder "Audit logging with AWS CloudTrail"
