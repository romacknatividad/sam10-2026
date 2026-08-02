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

# Lab 4 — Module 4: Compute & Storage

**Week 4 · AWS Cloud Foundations · Module 4 · Compute & Storage**
Complete AWS Skill Builder Module 4 + the activity below.

## Module summary
- **Compute:** EC2 instance types/families, AMI, auto scaling, load balancing.
- **Storage:** S3 (object, 99.999999999% durability), EBS (block, same-AZ), EFS (file, shared).
- Choosing by workload: object for content, block for boot/OS, file for shared POSIX.

## Hands-on activity
```bash
# 4.1  Map workload → storage (the exam table)
cat > lab4-storage-matrix.tsv <<'TSV'
Workload	Store	Why
website assets	S3	HTTP-served, versioned
boot volume of EC2	EBS	gp3 / io2
shared /var/log across 3 web servers	EFS	NFS, multi-AZ mount
database index (low latency, fixed size)	EBS	io2 Block Express
TSV

# 4.2  EC2 instance-family families (know the letter → workload)
echo "t = burstable | m = general | c = compute | r = memory | p/g = GPU | x = high-mem"

# 4.3  Spot vs On-Demand risk (no call, just concept)
python3 - <<'PY'
instances = {"On-Demand":0, "Reserved":0, "Spot":0}
# Spot: cheap, can be reclaimed with 2-min notice.
# Best fit: stateless, checkpointable workloads (batch, CI).
print("Spot interruption notice window = 2 minutes")
PY
```

## Artifact
`lab4-matrix.tsv` (the table above, you may extend it) + screenshot of Skill Builder quiz (≥ 80%).

## Checkpoint

```{dropdown} Q1. Which store is *durably* shared across many EC2 instances and accessed via NFS?
**EFS** (regional, multi-AZ mount targets, 99.99% availability); S3 is object (no POSIX),
EBS is single-AZ block.
```
```{dropdown} Q2. S3 "11 nines" durability but only "99.99% availability" — what's the practical difference?
**Durability** = won't lose an object (across AZs); **availability** = the service responds — you
can have a transient 503 read while the object is permanently safe.
```
`````{dropdown} Q3. t3.small vs m6i.large — which to pick for a steady CPU web server and why?
**m6i.large** (sustained CPU); t3.* are *burstable* — CPU credits deplete and throttle a steady load.
```
```{dropdown} Q4. Spot instance reclaimed with 2-min notice — what two AWS mechanisms help you survive it?
(1) **Spot interruption notice** (a CloudWatch event you can catch to checkpoint/exit),
(2) **Auto Scaling + multiple instance-purchase-options** so the ASG relaunches On-Demand.
```
````{dropdown} Q5. EBS `gp3` lets you set IOPS *independently* of size — what's the cost benefit?
You can have a **small volume (low $/GB-month) with provisioned performance ($/IOPS-month)**
separately — cheaper than `gp2` where IOPS scaled only with size.
````

## Sources
- Amazon EC2 instance families: https://aws.amazon.com/ec2/instance-types/
- Amazon S3 storage classes & durability: https://aws.amazon.com/s3/storage-classes/
- Amazon EBS volume types: https://aws.amazon.com/ebs/volume-types/
- Amazon EFS: https://aws.amazon.com/efs/
- Module 4 transcript: AWS Skill Builder "AWS Cloud Practitioner Essentials"
