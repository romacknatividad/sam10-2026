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

# Lab 7 — Module 7: Infrastructure as Code (AWS Cloud Operations)

**Week 7 · AWS Cloud Operations · Module 7 · Infrastructure as Code**
Complete AWS Skill Builder "**Getting Started with AWS CloudFormation**" + activity.
(CloudFormation is the primary IaC taught; Terraform is an alternate discussed in lecture ch09.)

## Module summary
- **IaC** = declarative, version-controlled infrastructure (vs. point-and-click).
- **CloudFormation** resources, parameters, mappings, outputs, change sets.
- **Drift detection** and **stack lifecycle** (create / update / delete).
- **SAM / CDK** as higher-level IaC options (overview; not required to write).

## Hands-on activity — build a stack from this book
```bash
# 7.1  Write a tiny template: index.html served from S3 (no AWS CLI run here — file saved)
cat > /tmp/gg-static.json <<'JSON'
{
  "Parameters": {"BucketName":{"Type":"String"}},
  "Resources": {
    "WebBucket": {"Type":"AWS::S3::Bucket",
      "Properties": {"BucketName":{"Ref":"BucketName"},"AccessControl":"PublicReadWrite",
        "WebsiteConfiguration":{"IndexDocument":"index.html"}}
    }
  },
  "Outputs": {"SiteUrl":{"Value":{"Fn::GetAtt":["WebBucket","WebsiteURL"]}}}
}
JSON

# 7.2  Validate the template syntax (local check — no IAM needed)
python3 - <<'PY'
import json
t = json.load(open("/tmp/gg-static.json"))      # parse -> valid JSON
assert "Resources" in t and "WebBucket" in t["Resources"]
assert t["Resources"]["WebBucket"]["Type"] == "AWS::S3::Bucket"
print("template parse OK: key fields present")
PY

# 7.3  Change set thought experiment (the exam concept)
echo "Update adds 1 property -> create-change-set -> REVIEW shows Add(1)/Modify(0)/Remove(0) -> execute-change-set"
```

## Artifact
`/tmp/gg-static.json` + screenshot of the **change-set review** page you'll create in the console
(showing the 1-Add plan), plus the Module 7 quiz (≥ 80%).

## Checkpoint

````{dropdown} Q1. Why does CloudFormation create a stack resource in `CREATE_IN_PROGRESS` then `CREATE_COMPLETE`?
Because CFN provisions the resource **asynchronously** and waits to confirm it reached a stable,
healthy state before marking the stack complete (and before depending resources can attach to it).
````
``{dropdown} Q2. 'Update stack' changes a property that forces replacement — what happens?
CFN performs a **delete + create**, updates dependents, and keeps the old physical id only if
retention is set — otherwise the resource is **replaced**.
```
```{dropdown} Q3. Drift detection — why run it after a manual console change?
To find the **drift between your template (source of truth) and reality**, so you can
re-import the manual change into the template (or roll it back) instead of losing it.
```
`````{dropdown} Q4. Parameters vs Mappings vs Conditions in one sentence each.
Parameters = input from the user at launch; Mappings = static lookup table (e.g., AMI per region);
Conditions = branch logic (create X only if Y).
``//``
````{dropdown} Q5. Why store templates in git + CodePipeline rather than launching once from the console?
So every environment (dev/stage/prod) is **identical** and every change is **auditable + reversible**;
the console launch is a one-off and impossible to diff/patch programmatically.
````

## Sources
- AWS CloudFormation User Guide: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/
- CloudFormation Guard: https://github.com/aws-cloudformation/cloudformation-guard
- AWS CDK: https://docs.aws.amazon.com/cdk/latest/guide/
- Module 7: AWS Skill Builder "Infrastructure as Code" learning path
