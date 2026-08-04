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

# MCO1 — AWS Infrastructure Deployment

**Major Course Output 1** · Weeks 1–6 · Weight: 40% of Part V grade

## Learning outcomes

By the end of this MCO you will be able to:
1. Design a secure, multi-tier cloud architecture on AWS.
2. Deploy a VPC with public and private subnets across two Availability Zones.
3. Launch EC2 instances, attach EBS volumes, and configure security groups.
4. Provision an S3 bucket for static assets and an RDS instance for a database.
5. Document the architecture with a diagram and a configuration matrix.
6. Prove the environment is functional (end-to-end HTTP), secure (least-privilege IAM), and properly configured (tags, naming conventions).

---

## Step-by-step instruction guide

### Step 1 — Architecture design (Week 3, after CF Module 4)

**Task:** Draw your target architecture on paper or in a tool (draw.io, Excalidraw, or a hand sketch).

Your architecture must include:
- 1 VPC with a CIDR block (e.g., `10.0.0.0/16`).
- 2 public subnets (one per AZ) for the web/ALB tier.
- 2 private subnets (one per AZ) for the app/DB tier.
- 1 Internet Gateway attached to the VPC.
- 1 NAT Gateway in a public subnet (for private subnet outbound access).
- Route tables: public subnets → IGW; private subnets → NAT.
- At least 1 EC2 instance in a public subnet (web server).
- 1 RDS instance in a private subnet (database).
- 1 S3 bucket for static assets.

**Deliverable:** `mco1-architecture.pdf` or `mco1-architecture.png` — a labelled diagram showing all components and their connections.

**Template (dummy content — replace with your real values):**

```
┌─────────────────────────────────────────────────────────────┐
│                     VPC 10.0.0.0/16                        │
│                                                             │
│  ┌─────────────────────┐    ┌─────────────────────────┐   │
│  │  Public Subnet AZ-a │    │  Public Subnet AZ-b     │   │
│  │  10.0.1.0/24        │    │  10.0.2.0/24            │   │
│  │                     │    │                         │   │
│  │  ┌───────────────┐  │    │  ┌───────────────────┐  │   │
│  │  │ EC2 Web Server │  │    │  │ EC2 Web Server     │  │   │
│  │  │ (t3.micro)     │  │    │  │ (t3.micro)         │  │   │
│  │  └───────────────┘  │    │  └───────────────────┘  │   │
│  │  ┌───────────────┐  │    │  ┌───────────────────┐  │   │
│  │  │ NAT Gateway    │  │    │  │ NAT Gateway        │  │   │
│  │  └───────────────┘  │    │  └───────────────────┘  │   │
│  └─────────────────────┘    └─────────────────────────┘   │
│                                                             │
│  ┌─────────────────────┐    ┌─────────────────────────┐   │
│  │ Private Subnet AZ-a │    │ Private Subnet AZ-b     │   │
│  │ 10.0.3.0/24         │    │ 10.0.4.0/24             │   │
│  │                     │    │                         │   │
│  │  ┌───────────────┐  │    │  ┌───────────────────┐  │   │
│  │  │ RDS PostgreSQL │  │    │  │ RDS PostgreSQL     │  │   │
│  │  │ (db.t3.micro)  │  │    │  │ (db.t3.micro)      │  │   │
│  │  └───────────────┘  │    │  └───────────────────┘  │   │
│  └─────────────────────┘    └─────────────────────────┘   │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ S3 Bucket: gg-mco1-static-assets                    │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

---

### Step 2 — Create the VPC and networking (Week 4, after CF Module 5)

**Task:** Use the AWS Console (or AWS CLI if you prefer) to create the VPC and all networking components.

**Detailed instructions:**

1. Go to the AWS Console → VPC Dashboard.
2. Click **Create VPC**:
   - Name tag: `gg-mco1-vpc`
   - IPv4 CIDR block: `10.0.0.0/16`
   - Click **Create VPC**
3. Create 2 public subnets:
   - **Subnet 1**: VPC = `gg-mco1-vpc`, CIDR = `10.0.1.0/24`, AZ = `ap-southeast-1a`
     - Name tag: `gg-mco1-public-a`
     - Auto-assign public IPv4: **Enable**
   - **Subnet 2**: VPC = `gg-mco1-vpc`, CIDR = `10.0.2.0/24`, AZ = `ap-southeast-1b`
     - Name tag: `gg-mco1-public-b`
     - Auto-assign public IPv4: **Enable**
4. Create 2 private subnets:
   - **Subnet 3**: VPC = `gg-mco1-vpc`, CIDR = `10.0.3.0/24`, AZ = `ap-southeast-1a`
     - Name tag: `gg-mco1-private-a`
   - **Subnet 4**: VPC = `gg-mco1-vpc`, CIDR = `10.0.4.0/24`, AZ = `ap-southeast-1b`
     - Name tag: `gg-mco1-private-b`
5. Create an Internet Gateway:
   - Name tag: `gg-mco1-igw`
   - Attach to VPC: `gg-mco1-vpc`
6. Create route tables:
   - **Public RT**: VPC = `gg-mco1-vpc`
     - Route: `0.0.0.0/0` → `igw-ggmco1`
     - Associate with: `gg-mco1-public-a` and `gg-mco1-public-b`
   - **Private RT**: VPC = `gg-mco1-vpc`
     - Route: `0.0.0.0/0` → NAT Gateway (create one in Step 3)
     - Associate with: `gg-mco1-private-a` and `gg-mco1-private-b`

**Verify:** From the VPC Dashboard, confirm all 4 subnets, 1 IGW, 2 route tables, and 1 NAT Gateway are listed and correctly associated.

**Screenshot:** Save a screenshot of the VPC Dashboard showing all components. Save as `mco1-vpc-diagram.png`.

---

### Step 3 — Create a NAT Gateway (Week 4)

**Task:** Create a NAT Gateway in a public subnet so private instances can reach the internet (for updates).

**Detailed instructions:**

1. Go to VPC → NAT Gateways → **Create NAT Gateway**.
2. Subnet: select `gg-mco1-public-a`.
3. Elastic IP: allocate a new Elastic IP (or use an existing one).
4. Click **Create NAT Gateway**.
5. Wait for the status to become **available** (takes ~2 minutes).
6. Update the **Private RT** route table:
   - Add route: `0.0.0.0/0` → `nat-ggmco1` (the NAT Gateway you just created).

**Verify:** The private route table now has a route to the NAT Gateway.

**Screenshot:** Save as `mco1-nat-diagram.png`.

---

### Step 4 — Create security groups (Week 4)

**Task:** Create security groups for the web tier and the database tier.

**Detailed instructions:**

1. Go to EC2 → Security Groups → **Create Security Group**.

**Web Tier SG (`gg-mco1-web-sg`):**
- VPC: `gg-mco1-vpc`
- Name: `gg-mco1-web-sg`
- Description: "Allow HTTP, HTTPS, and SSH from anywhere"
- Inbound rules:
  - Type: HTTP, Protocol: TCP, Port: 80, Source: `0.0.0.0/0`
  - Type: HTTPS, Protocol: TCP, Port: 443, Source: `0.0.0.0/0`
  - Type: SSH, Protocol: TCP, Port: 22, Source: `Your-IP/32` (restrict to your IP)
- Outbound rules: All traffic (default)

**Database Tier SG (`gg-mco1-db-sg`):**
- VPC: `gg-mco1-vpc`
- Name: `gg-mco1-db-sg`
- Description: "Allow PostgreSQL from web tier only"
- Inbound rules:
  - Type: PostgreSQL, Protocol: TCP, Port: 5432, Source: `gg-mco1-web-sg` (reference the web SG by ID)
- Outbound rules: All traffic (default)

**Verify:** Both SGs are listed in EC2 → Security Groups with correct inbound rules.

**Screenshot:** Save as `mco1-sgs.png`.

---

### Step 5 — Launch EC2 instances (Week 5, after CF Module 4)

**Task:** Launch two EC2 instances — one web server in the public subnet, and optionally one in the private subnet.

**Detailed instructions:**

1. Go to EC2 → Instances → **Launch Instance**.

**Web Server 1 (public):**
- Name: `gg-mco1-web-a`
- AMI: Amazon Linux 2023 (or latest Amazon Linux 2)
- Instance type: `t3.micro` (Free Tier eligible)
- Key pair: select your existing key pair (or create a new one named `gg-mco1-key`)
- Network settings:
  - VPC: `gg-mco1-vpc`
  - Subnet: `gg-mco1-public-a`
  - Auto-assign public IP: **Enable**
  - Security group: select `gg-mco1-web-sg`
- Storage: default (8 GB gp3)
- Click **Launch Instance**

**Web Server 2 (public):**
- Same settings as above, but:
  - Name: `gg-mco1-web-b`
  - Subnet: `gg-mco1-public-b`

**Verify:** Both instances are in **running** state. Note their public IP addresses.

**Screenshot:** Save as `mco1-ec2-instances.png`.

---

### Step 6 — Configure the web server (Week 5)

**Task:** SSH into the web server and install nginx, then prove it serves a page.

**Detailed instructions:**

1. SSH into Web Server 1:
   ```bash
   ssh -i gg-mco1-key.pem ec2-user@<WEB_SERVER_1_PUBLIC_IP>
   ```
2. Install nginx:
   ```bash
   sudo dnf install -y nginx
   ```
3. Create a simple index page:
   ```bash
   sudo bash -c 'cat > /usr/share/nginx/html/index.html <<EOF
   <html>
   <head><title>GreenGrid MCO1</title></head>
   <body>
   <h1>GreenGrid Infrastructure — MCO1</h1>
   <p>Server: gg-mco1-web-a</p>
   <p>Status: operational</p>
   </body>
   </html>'
   ```
4. Start nginx:
   ```bash
   sudo systemctl enable --now nginx
   ```
5. Test from your local browser:
   ```
   http://<WEB_SERVER_1_PUBLIC_IP>/
   ```
   You should see the "GreenGrid Infrastructure — MCO1" page.

**Repeat** for Web Server 2 (optional, for multi-AZ proof).

**Verify:** The page loads in your browser. The security group allows HTTP (port 80) from `0.0.0.0/0`.

**Screenshot:** Save the browser view as `mco1-web-page.png`.

---

### Step 7 — Create an S3 bucket (Week 5, after CF Module 4)

**Task:** Create an S3 bucket for static assets and upload a test file.

**Detailed instructions:**

1. Go to S3 → Create bucket:
   - Bucket name: `gg-mco1-static-assets-<your-account-id>` (must be globally unique)
   - Region: `ap-southeast-1`
   - Block Public Access: **uncheck** "Block all public access" (for static website hosting)
   - Bucket versioning: **Enable**
   - Click **Create bucket**
2. Upload a test file:
   - Click on the bucket → **Upload** → **Add files**
   - Upload a file named `index.html` with content:
     ```html
     <h1>GreenGrid Static Assets</h1>
     <p>This file is stored in S3 for the MCO1 deployment.</p>
     ```
3. Make the file publicly readable (for testing only — in production you'd use CloudFront or a presigned URL):
   - Select the file → **Actions** → **Make public**
4. Get the object URL and test it in your browser.

**Verify:** The `index.html` file is accessible via the S3 object URL.

**Screenshot:** Save as `mco1-s3-bucket.png`.

---

### Step 8 — Provision an RDS instance (Week 6, after CF Module 6)

**Task:** Create a PostgreSQL RDS instance in the private subnet.

**Detailed instructions:**

1. Go to RDS → Databases → **Create database**.
2. Choose:
   - Database creation method: **Standard create**
   - Engine: **PostgreSQL**
   - Version: latest PostgreSQL 15 or 16
   - Templates: **Free tier**
3. Settings:
   - DB instance identifier: `gg-mco1-db`
   - Master username: `admin`
   - Master password: `Mco1Demo!2026` (use a strong password in practice)
   - Confirm password: same
4. Instance size: `db.t3.micro` (Free Tier)
5. Storage: 20 GB, GP3
6. Connectivity:
   - VPC: `gg-mco1-vpc`
   - Public access: **No** (this is a private database)
   - Subnet group: create a new DB subnet group `gg-mco1-db-subnets` with both private subnets
   - Security group: select `gg-mco1-db-sg`
   - Database port: 5432
7. Click **Create database**.
8. Wait for the status to become **available** (~5 minutes).

**Verify:** The RDS instance is in the private subnet and has status "available".

**Screenshot:** Save as `mco1-rds.png`.

---

### Step 9 — Tag all resources (Week 6)

**Task:** Apply consistent tags to every resource you created.

**Required tags for every resource:**

| Key | Value |
| --- | --- |
| `Project` | `SAM10-MCO1` |
| `Owner` | `<your-name>` |
| `Environment` | `mco1-demo` |
| `CostCenter` | `CCS-AUF` |

**How to tag:**
- EC2: select instance → **Actions** → **Instance settings** → **Add/Edit tags**
- RDS: select DB → **Modify** → **Add tags**
- S3: bucket → **Properties** → **Tags**
- VPC/Subnets: VPC Dashboard → select resource → **Tags** tab → **Manage tags**

**Verify:** All resources have the 4 required tags.

**Screenshot:** Save as `mco1-tags.png`.

---

### Step 10 — Prove the environment is functional, secure, and properly configured (Week 6)

**Task:** Run a series of verification commands and document the results.

**Functional proof:**
```bash
# 1. Web server responds
curl -sS -o /dev/null -w "HTTP %{http_code} in %{time_total}s\n" http://<WEB_SERVER_1_PUBLIC_IP>/
# Expected: HTTP 200 in <1s

# 2. S3 bucket is accessible
aws s3 ls s3://gg-mco1-static-assets-<account-id>/
# Expected: lists index.html

# 3. RDS is reachable from the web server
ssh -i gg-mco1-key.pem ec2-user@<WEB_SERVER_1_PUBLIC_IP>
sudo dnf install -y postgresql
psql -h <RDS_ENDPOINT> -U admin -d postgres -c "SELECT version();"
# Expected: PostgreSQL version string
```

**Security proof:**
```bash
# 1. Verify SGs only allow expected ports
aws ec2 describe-security-groups --group-ids <web-sg-id> <db-sg-id> \
  --query 'SecurityGroups[*].{Name:GroupName,Inbound:IpPermissions}'
# Expected: web-sg allows 80,443,22; db-sg allows 5432 only from web-sg

# 2. Verify RDS is not publicly accessible
aws rds describe-db-instances --db-instance-identifier gg-mco1-db \
  --query 'DBInstances[0].{PubliclyAccessible:PubliclyAccessible,Endpoint:Endpoint.Address}'
# Expected: PubliclyAccessible = false
```

**Configuration proof:**
```bash
# 1. Verify all resources have required tags
aws ec2 describe-tags --filters "Name=value,Values=SAM10-MCO1" \
  --query 'Tags[*].{ResourceId:ResourceId,Key:Key,Value:Value}'
# Expected: all resource IDs listed with Project=SAM10-MCO1
```

**Deliverable:** Save all verification outputs to `mco1-verification.txt`.

---

## MCO1 Documentation Template

### MCO1 Report Template

Replace the dummy content below with your real values.

```markdown
# MCO1 — AWS Infrastructure Deployment Report

## Student Information
- **Name:** Jane Doe
- **Student ID:** 12345678
- **Course:** SAM10 Systems Administration and Management | College of Computer Science | Angeles University Foundation | R.L.Natividad
- **Date:** 2026-08-04

## 1. Architecture Diagram
![Architecture Diagram](mco1-architecture.png)

## 2. Component Inventory

| Component | AWS Resource | ID/Name | Region | AZ |
| --- | --- | --- | --- | --- |
| VPC | gg-mco1-vpc | vpc-0abc123 | ap-southeast-1 | — |
| Public Subnet A | gg-mco1-public-a | subnet-0def456 | ap-southeast-1a | AZ-a |
| Public Subnet B | gg-mco1-public-b | subnet-0ghi789 | ap-southeast-1b | AZ-b |
| Private Subnet A | gg-mco1-private-a | subnet-0jkl012 | ap-southeast-1a | AZ-a |
| Private Subnet B | gg-mco1-private-b | subnet-0mno345 | ap-southeast-1b | AZ-b |
| Internet Gateway | gg-mco1-igw | igw-0pqr678 | ap-southeast-1 | — |
| NAT Gateway | gg-mco1-nat | nat-0stu901 | ap-southeast-1a | AZ-a |
| Web SG | gg-mco1-web-sg | sg-0vwx234 | — | — |
| DB SG | gg-mco1-db-sg | sg-0yza567 | — | — |
| Web Server A | gg-mco1-web-a | i-0bcd890 | ap-southeast-1a | AZ-a |
| Web Server B | gg-mco1-web-b | i-0efg123 | ap-southeast-1b | AZ-b |
| RDS PostgreSQL | gg-mco1-db | gg-mco1-db.xxxxx.ap-southeast-1.rds.amazonaws.com | ap-southeast-1 | Multi-AZ |
| S3 Bucket | gg-mco1-static-assets | gg-mco1-static-assets-123456789 | ap-southeast-1 | — |

## 3. Security Configuration

### Security Group Rules
**Web Tier SG (`gg-mco1-web-sg`):**
| Type | Protocol | Port | Source | Purpose |
| --- | --- | --- | --- | --- |
| HTTP | TCP | 80 | 0.0.0.0/0 | Allow web traffic |
| HTTPS | TCP | 443 | 0.0.0.0/0 | Allow secure web traffic |
| SSH | TCP | 22 | <your-IP>/32 | Admin access only |

**DB Tier SG (`gg-mco1-db-sg`):**
| Type | Protocol | Port | Source | Purpose |
| --- | --- | --- | --- | --- |
| PostgreSQL | TCP | 5432 | sg-0vwx234 (web-sg) | DB access from web tier only |

### Least-Privilege Justification
The web tier SG allows HTTP/HTTPS from anywhere because it serves public web traffic. SSH is restricted to the student's IP address only. The DB tier SG allows PostgreSQL only from the web tier SG — no direct internet access to the database.

## 4. Functional Verification

### Web Server
- URL: http://<WEB_SERVER_1_PUBLIC_IP>/
- Status: HTTP 200 OK
- Response time: < 1 second
- Screenshot: mco1-web-page.png

### S3 Bucket
- Bucket name: gg-mco1-static-assets-<account-id>
- Test file: index.html (uploaded and publicly accessible)
- Versioning: Enabled

### RDS Database
- Endpoint: gg-mco1-db.xxxxx.ap-southeast-1.rds.amazonaws.com
- Engine: PostgreSQL 15/16
- Publicly accessible: No (private subnet)
- Connection test: `psql -h <endpoint> -U admin -d postgres -c "SELECT 1;"` → returns `1`

## 5. Tagging Compliance

All resources tagged with:
- Project: SAM10-MCO1
- Owner: Jane Doe
- Environment: mco1-demo
- CostCenter: CCS-AUF

Tag compliance screenshot: mco1-tags.png

## 6. Lessons Learned

1. What went well: (describe)
2. What was challenging: (describe)
3. What you would improve: (describe)
4. Security considerations for production: (describe)

## 7. Sources
- AWS VPC User Guide: https://docs.aws.amazon.com/vpc/latest/userguide/
- AWS EC2 User Guide: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/
- AWS RDS User Guide: https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/
- AWS S3 User Guide: https://docs.aws.amazon.com/AmazonS3/latest/userguide/
```

---

## Checkpoint Questions

```{dropdown} Q1. Why does the private subnet need a NAT Gateway?
**Answer.** Instances in private subnets have no direct internet route (no IGW). The NAT Gateway allows them to initiate outbound connections (e.g., for OS updates, package installs) while remaining unreachable from the internet inbound.
```
```{dropdown} Q2. Why restrict SSH to your IP (`<your-IP>/32`) instead of `0.0.0.0/0`?
**Answer.** SSH on port 22 is a prime target for brute-force attacks. Restricting to your IP means only you can SSH in — no one else on the internet can attempt login.
```
```{dropdown} Q3. Why is the RDS instance in a private subnet and not publicly accessible?
**Answer.** Databases contain sensitive data. A public RDS instance is exposed to the internet and is a common attack vector. Keeping it private means only resources within the VPC (the web tier SG) can reach it.
```
```{dropdown} Q4. What is the difference between a route table and a security group?
**Answer.** A **route table** controls *where traffic can go* (network-level routing — e.g., 0.0.0.0/0 → IGW). A **security group** controls *who can reach a resource* (instance-level firewall — e.g., allow port 80 from 0.0.0.0/0). They operate at different layers.
```
```{dropdown} Q5. Why enable S3 bucket versioning?
**Answer.** Versioning protects against accidental deletion or overwrites. If an object is deleted or modified, previous versions are retained and can be restored — providing an additional safety net beyond the 11 nines of S3 durability.
```

---

## Sources
- AWS Well-Architected Framework — Security Pillar: https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/
- AWS VPC User Guide: https://docs.aws.amazon.com/vpc/latest/userguide/
- AWS EC2 User Guide: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/
- AWS RDS User Guide: https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/
- AWS S3 User Guide: https://docs.aws.amazon.com/AmazonS3/latest/userguide/
- AWS Security Best Practices: https://docs.aws.amazon.com/prescriptive-guidance/latest/security-best-practices/
- Module 3 (Security), Module 4 (Compute & Storage), Module 5 (Networking), Module 6 (Databases) — AWS Skill Builder "AWS Cloud Practitioner Essentials"

<footer class="sam10-footer" style="margin-top:3em;text-align:center;color:#666;font-size:0.85em">
  <hr>
  <span>SAM10 Systems Administration and Management | College of Computer Science | Angeles University Foundation | R.L.Natividad</span> &mdash;
  <span>Systems Administration and Maintenance</span> &mdash;
  <span>Undergraduate course companion</span>
</footer>
