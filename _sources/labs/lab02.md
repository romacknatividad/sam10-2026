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

# Lab 2 — Module 2: Technology Fundamentals

**Week 2 · AWS Cloud Foundations · Module 2 · STATUS: COMPLETED**

## Module summary
- AWS global infrastructure: **Regions** (geographic) and **Availability Zones**
  (AZs = one or more isolated data centers in a Region).
- Shared responsibility model.
- Core services: IAM, EC2, S3, VPC, RDS at a high level.

## Hands-on activity
```bash
# Inspect the shared-responsibility model in one sentence each:
echo "YOU manage identity (IAM)"        # Identity
echo "AWS manages the hypervisor"        # Infrastructure
# Show the AZ count of the nearest region (requires no AWS calls here — just recall):
# ap-southeast-1 = 3 AZs; us-east-1 = 6 AZs
python3 -c "print('Choose N>=2 AZs in one region for fault isolation.')"
```

## Artifact
Screenshot of your Skill Builder Module 2 quiz (≥ 80%) + a **filled-in one-line table**
saved to `lab2-shared-resp.txt`:

| Layer | Responsibility | Who? |
| --- | --- | --- |
| IAM / data | config, identity, OS | Customer |
| Hypervisor / regions | physical host, facilities | AWS |
| Network & gateway | VPC routing, SG | Shared |

## Checkpoint

```{dropdown} Q1. How many Availability Zones make up an AWS Region at minimum?
**Minimum 2** AZs (production regions all have ≥2; many have 3). Each AZ is one+ data centers.
```
```{dropdown} Q2. In the shared responsibility model, who patches the guest OS inside an EC2 instance?
**You, the customer.** AWS patches the hypervisor/facilities; you patch your OS/apps — that's the
classic boundary.
```
```{dropdown} Q3. Region vs Availability Zone — which do you choose if you want the lowest latency to your users?
A **Region**, then within it **multiple AZs** for fault isolation; Route 53 latency-based routing
then sends each user to the nearest region.
```
```{dropdown} Q4. What does "99.99% availability" actually mean in a year?
≈ **52.6 minutes** of allowable downtime per year (365×24×0.0001 ≈ 52.6 min).
```
````{dropdown} Q5. If one AZ in ap-southeast-1 fails, does S3 in that region also fail?
**No** — S3 is regional and replicates objects across AZs; one AZ failing keeps S3 **available**
(other AZs serve it). Object *durability* (11 nines) is separate from AZ availability.
````

## Sources
- AWS Global Infrastructure: https://aws.amazon.com/about-aws/global-infrastructure/
- Shared responsibility model: https://aws.amazon.com/compliance/shared-responsibility-model/
- Module 2 transcript: AWS Skill Builder "AWS Cloud Practitioner Essentials"
