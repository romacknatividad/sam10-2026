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

# Lab 5 — Module 5: Networking & Content Delivery

**Week 5 · AWS Cloud Foundations · Module 5 · Networking & Content Delivery**
Complete AWS Skill Builder Module 5 + activity.

## Module summary
- **VPC** as your private slice of AWS; **subnets** (public = IGW route, private = NAT),
  **route tables**, **network ACLs** (stateless, subnet layer) vs **security groups** (stateful, ENI layer).
- **VPC peering / Transit Gateway**, **Direct Connect**, and **VPN CloudHub**.
- **Route 53** (DNS + health checks), **CloudFront** (CDN), **ELB** families (ALB/NLB/GLB).

## Hands-on activity
```bash
# 5.1  Draw the rule that distinguishes NACL (statful/stateless) from SG (stateful/stateless)
cat > lab5-net-comparison.tsv <<'TSV'
Control	Stateful?	Layer	Direction handled	Typical use
Security Group	Yes	ENI (instance)	Returns only if request returned	App port allow
Network ACL	No	Subnet	Both inbound + outbound explicitly	Egress-restrict subnet
TSV

# 5.2  VPC CIDR sizing exercise (no AWS call)
python3 - <<'PY'
# One VPC 10.0.0.0/16 -> 65k addresses. Need 2 public + 3 private subnets per AZ x 2 AZs.
az = 2; subnets_per_az = 5
total = az * subnets_per_az          # 10 subnets
cidr_per_subnet = 32 - 16 - 4        # /20 = 4096 hosts each -> room to grow
print(f"subnets needed: {total}, use /{cidr_per_subnet} per subnet (4096 hosts)")
PY
```

## Artifact
`lab5-net-comparison.tsv` + a sketch in `lab5-vpc.png` (screenshot of a hand-drawn or whiteboard
VPC with 2 AZs, public/private subnets, IGW, NAT) showing the **statefulness** of SGs vs
**statelessness** of NACLs.

## Checkpoint

```{dropdown} Q1. A packet leaves a subnet → NACL allows outbound → SG allows inbound. Return packet — which device decides?
**Both.** NAC**L is stateless** (return must pass an explicit outbound rule); the **SG is stateful**
(return is auto-allowed if the request was allowed out).
```
```{dropdown} Q2. Public subnet vs private subnet — what single route-table line tells them apart?
Public has a route to an **Internet Gateway** (`0.0.0.0/0 → igw-xxx`); private has that default
routed to a **NAT Gateway** instead (or nothing, for fully isolated subnets).
```
``{dropdown} Q3. CloudFront vs Route 53 — which is the DNS, which is the CDN?
**Route 53** = DNS (names, health checks, failover). **CloudFront** = CDN (edge caches, lowest-latency.
```
``{dropdown} Q4. ALB vs NLB — one key difference in what they route on?
ALB = **Layer 7** (HTTP/HTTPS, Host/path routing); NLB = **Layer 4** (TCP/UDP, extreme throughput,
preserves source IP by default).
```
````{dropdown} Q5. '10.0.0.0/16' VPC — how many /24 subnets can you carve from it?
A /16 has 2^(24−16) = 2^8 = **256** /24 subnets (each /24 = 251 usable IPs in AWS).
````

## Sources
- Amazon VPC User Guide: https://docs.aws.amazon.com/vpc/latest/userguide/
- Route 53: https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/
- CloudFront: https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/
- Elastic Load Balancing: https://docs.aws.amazon.com/elasticloadbalancing/latest/
- Module 5 transcript: AWS Skill Builder "AWS Cloud Practitioner Essentials"
