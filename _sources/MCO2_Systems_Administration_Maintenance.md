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

# MCO2 — Systems Administration and Maintenance

**Major Course Output 2** · Weeks 7–15 · Weight: 60% of Part V grade

## Learning outcomes

By the end of this MCO you will be able to:
1. Implement monitoring, logging, and alerting for an AWS infrastructure.
2. Configure automated backups and prove a restore operation.
3. Apply security hardening measures (IAM least-privilege, SSH hardening, patching).
4. Build basic automation (Infrastructure as Code, scheduled tasks).
5. Troubleshoot common system issues using logs, metrics, and process inspection.
6. Present a professional operations report with evidence and recommendations.

---

## Step-by-step instruction guide

### Step 1 — Set up monitoring with CloudWatch (Week 8, CO Module 8)

**Task:** Create CloudWatch dashboards, alarms, and log groups for your MCO1 infrastructure.

**Detailed instructions:**

1. **Create a custom metric for CPU utilization:**
   - Go to CloudWatch → Metrics → **All metrics** → **Custom Namespaces** → **Create metric**.
   - Namespace: `GreenGrid/MCO2`
   - Metric name: `CPUUtilization`
   - Dimensions: `InstanceId` = `<your-web-instance-id>`
   - Value: the current CPU % (you can push this manually or use the CloudWatch agent).

2. **Create a CloudWatch dashboard:**
   - Go to CloudWatch → Dashboards → **Create dashboard**.
   - Name: `gg-mco2-dashboard`
   - Add widgets:
     - **Line chart**: `GreenGrid/MCO2` → `CPUUtilization` (last 24 hours)
     - **Line chart**: `AWS/EC2` → `CPUUtilization` for your instance
     - **Number widget**: `AWS/EC2` → `StatusCheckFailed` (instance status checks)
     - **Log widget**: query your CloudWatch Logs group for errors

3. **Create an alarm for high CPU:**
   - Go to CloudWatch → Alarms → **Create alarm**.
   - Select metric: `CPUUtilization` for your instance.
   - Condition: `Average > 80%` for 3 consecutive periods of 5 minutes.
   - Actions: notify your SNS topic (create one if you don't have it).
   - Name: `gg-mco2-high-cpu`

4. **Create a CloudWatch Log Group for nginx access logs:**
   - Go to CloudWatch → Logs → **Create log group**.
   - Log group name: `/gg/mco2/nginx-access`
   - On your web server, install the CloudWatch agent and configure it to ship `/var/log/nginx/access.log` to this log group.

**Verify:** The dashboard shows CPU metrics. The alarm is in `OK` state (CPU is low). The log group receives log entries.

**Screenshots:** Save as:
- `mco2-cw-dashboard.png` (dashboard view)
- `mco2-cw-alarm.png` (alarm configuration)
- `mco2-cw-logs.png` (log group with entries)

---

### Step 2 — Set up logging and auditing (Week 9, CO Module 9)

**Task:** Enable CloudTrail for your account and configure log retention.

**Detailed instructions:**

1. **Create a CloudTrail:**
   - Go to CloudTrail → Trails → **Create trail**.
   - Name: `gg-mco2-trail`
   - Storage location: **Create a new S3 bucket** named `gg-mco2-cloudtrail-logs-<account-id>`
   - Enable **Log file validation** (ensures log integrity)
   - Enable **CloudWatch Logs integration**:
     - Create a new log group: `/aws/cloudtrail/gg-mco2`
     - Use a new IAM role: `gg-mco2-cloudtrail-role`
   - Apply the trail to **all regions**
   - Enable **data events** for S3 (log all S3 object-level API calls)

2. **Verify the trail is working:**
   - Go to CloudTrail → Events → **Event history**.
   - Filter by event name: `ConsoleLogin`
   - You should see your own login events.

3. **Set up log retention:**
   - Go to CloudWatch → Log groups → `/aws/cloudtrail/gg-mco2`
   - Set retention to **90 days** (or your organization's policy).
   - Set retention for `/gg/mco2/nginx-access` to **30 days**.

**Verify:** CloudTrail is recording events in all regions. Log groups have retention policies set.

**Screenshots:** Save as `mco2-cloudtrail.png` and `mco2-log-retention.png`.

---

### Step 3 — Implement backup and disaster recovery (Week 11, CO Module 11)

**Task:** Create automated backups for your RDS instance and EBS volumes, then prove a restore works.

**Detailed instructions:**

1. **Enable RDS automated backups:**
   - Go to RDS → Databases → select `gg-mco1-db` → **Modify**.
   - Backup retention period: **7 days**
   - Preferred backup window: **03:00-04:00 UTC** (off-peak hours)
   - Enable **Performance Insights** (optional, for the report)
   - Click **Continue** → **Modify DB**

2. **Create a manual RDS snapshot:**
   - Go to RDS → Databases → select `gg-mco1-db` → **Actions** → **Take snapshot**.
   - Snapshot name: `gg-mco1-db-backup-$(date +%F)`
   - Wait for the snapshot status to become **available**.

3. **Create an EBS snapshot for the web server's root volume:**
   - Go to EC2 → Volumes → select the volume attached to `gg-mco1-web-a`.
   - **Actions** → **Create snapshot**.
   - Description: `gg-mco1-web-a-root-backup-$(date +%F)`
   - Wait for the snapshot to complete.

4. **Prove a restore works:**
   - **RDS restore:** Go to RDS → Snapshots → select your manual snapshot → **Actions** → **Restore snapshot**.
     - New DB instance identifier: `gg-mco1-db-restore-test`
     - Use the same DB instance class and settings.
     - Wait for the restored DB to become **available**.
     - Connect to the restored DB and run: `SELECT 1;` — should return `1`.
     - **Delete the restored DB** after verification (to avoid charges).
   - **EBS restore:** Go to EC2 → Snapshots → select your snapshot → **Actions** → **Create volume**.
     - Create the volume in the same AZ.
     - Attach it to a test instance as a secondary volume.
     - Mount and verify the files are intact.

**Verify:** RDS PITR works (you can restore to any point within the 7-day window). EBS snapshot restores correctly.

**Screenshots:** Save as:
- `mco2-rds-snapshot.png`
- `mco2-rds-restore.png`
- `mco2-ebs-snapshot.png`

---

### Step 4 — Apply security hardening (Week 10, CO Module 10 + Week 3 CF Module 3)

**Task:** Harden the EC2 instances and IAM configuration.

**Detailed instructions:**

1. **SSH hardening on web servers:**
   ```bash
   ssh -i gg-mco1-key.pem ec2-user@<WEB_SERVER_1_PUBLIC_IP>
   
   # Change SSH to non-standard port
   sudo sed -i 's/^#Port 22/Port 2222/' /etc/ssh/sshd_config
   
   # Disable password authentication
   sudo sed -i 's/^#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config
   
   # Disable root login
   sudo sed -i 's/^#PermitRootLogin prohibit-password/PermitRootLogin no/' /etc/ssh/sshd_config
   
   # Restart SSH
   sudo systemctl restart sshd
   
   # Verify SSH is listening on port 2222
   sudo ss -tlnp | grep :2222
   ```

2. **Install and configure the CloudWatch agent:**
   ```bash
   sudo dnf install -y amazon-cloudwatch-agent
   
   # Create the agent config
   sudo bash -c 'cat > /opt/aws/amazon-cloudwatch-agent/bin/config.json <<EOF
   {
     "metrics": {
       "append_dimensions": {
         "InstanceId": "${aws:InstanceId}"
       },
       "metrics_collected": {
         "cpu": { "measurement": ["cpu_usage_idle"], "metrics_collection_interval": 60 },
         "mem": { "measurement": ["mem_used_percent"], "metrics_collection_interval": 60 },
         "disk": { "measurement": ["used_percent"], "resources": ["/"] }
       }
     },
     "logs": {
       "logs_collected": {
         "files": {
           "collect_list": [
             {
               "file_path": "/var/log/nginx/access.log",
               "log_group_name": "/gg/mco2/nginx-access",
               "log_stream_name": "${aws:InstanceId}"
             }
           ]
         }
       }
     }
   }
   EOF'
   
   sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
     -a fetch-config -m ec2 -s -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json
   ```

3. **Apply OS patches:**
   ```bash
   sudo dnf update -y
   sudo dnf install -y yum-plugin-security
   sudo yum --security updateinfo list security
   sudo yum update -y --security
   ```

4. **Verify IAM least-privilege for the EC2 instance:**
   - Go to IAM → Roles → find the role attached to your EC2 instance.
   - Verify the policy is scoped to only the actions needed (S3 read/write for static assets, CloudWatch put metrics, etc.).
   - **Do NOT** use `AdministratorAccess` — use a custom policy with least privilege.

**Verify:** SSH works on port 2222 only. CloudWatch agent is shipping metrics and logs. OS is patched. IAM role has least-privilege policies.

**Screenshots:** Save as `mco2-ssh-hardening.png`, `mco2-cw-agent.png`, `mco2-patches.png`, `mco2-iam-role.png`.

---

### Step 5 — Implement basic automation (Week 13, CO Module 13 + Week 7 CO Module 7)

**Task:** Create an AWS Systems Manager Automation document or a simple Lambda function for a routine task.

**Detailed instructions (Option A — SSM Automation):**

1. Go to Systems Manager → Automation → **Create automation**.
2. Choose **Quick setup** → **AWS-ApplyPatchBaseline** (or create a custom document).
3. Configure:
   - Name: `gg-mco2-patch-automation`
   - Targets: select your web server instances by tag (`Project=SAM10-MCO1`)
   - Schedule: create a **Maintenance Window** that runs every Sunday at 02:00 UTC
4. Create the maintenance window:
   - Go to Systems Manager → Maintenance Windows → **Create maintenance window**.
   - Name: `gg-mco2-weekly-patch`
   - Schedule: `cron(0 2 ? * SUN *)` (every Sunday at 02:00 UTC)
   - Duration: 2 hours
   - Cutoff: 1 hour
   - Target: select your web server instances
   - Task: `AWS-ApplyPatchBaseline`

**Detailed instructions (Option B — Lambda function for log rotation):**

1. Go to Lambda → **Create function**.
2. Name: `gg-mco2-log-rotation`
3. Runtime: Python 3.12
4. Role: create a new role with basic Lambda permissions + CloudWatch Logs permissions.
5. Add the following code:
   ```python
   import boto3
   import datetime
   
   logs = boto3.client('logs')
   
   def lambda_handler(event, context):
       # List log groups older than 30 days and set retention to 30 days
       response = logs.describe_log_groups()
       for group in response['logGroups']:
           log_group = group['logGroupName']
           retention = group.get('retentionInDays', 0)
           if retention == 0 or retention > 30:
               logs.put_retention_policy(
                   logGroupName=log_group,
                   retentionInDays=30
               )
               print(f"Set retention for {log_group} to 30 days")
       return {'statusCode': 200, 'body': 'Log retention policies applied'}
   ```
6. Create a CloudWatch Events rule to trigger the Lambda every day at 00:00 UTC:
   - Rule name: `gg-mco2-daily-log-retention`
   - Schedule expression: `cron(0 0 ? * * *)`
   - Target: `gg-mco2-log-rotation` Lambda function

**Verify:** The automation runs on schedule. Check CloudWatch Logs for execution results.

**Screenshots:** Save as `mco2-ssm-automation.png` or `mco2-lambda.png`.

---

### Step 6 — Troubleshoot a common issue (Week 10, CO Module 10)

**Task:** Simulate a common issue and troubleshoot it using CloudWatch, logs, and process inspection.

**Scenario:** The web server is returning 502 Bad Gateway.

**Troubleshooting steps:**

1. **Check the ALB target group health:**
   - Go to EC2 → Target Groups → select your TG → **Health checks** tab.
   - Are the targets healthy? If not, note the reason (e.g., "request timed out", "unhealthy").

2. **Check nginx logs on the web server:**
   ```bash
   ssh -i gg-mco1-key.pem -p 2222 ec2-user@<WEB_SERVER_1_PUBLIC_IP>
   sudo tail -50 /var/log/nginx/error.log
   ```
   Look for errors like `connect() failed`, `permission denied`, or `no route to host`.

3. **Check if nginx is running:**
   ```bash
   sudo systemctl status nginx
   ```
   If it's not running, try to start it:
   ```bash
   sudo systemctl start nginx
   sudo systemctl status nginx
   ```

4. **Check system resources:**
   ```bash
   top -bn1 | head -10
   df -h /
   free -m
   ```
   Is CPU at 100%? Is disk full? Is memory exhausted?

5. **Check the security group:**
   - Is the ALB security group allowing traffic on port 80 from `0.0.0.0/0`?
   - Is the web server's security group allowing traffic on port 80 from the ALB's security group?

6. **Document the root cause and fix:**
   - Write a brief description of what caused the 502 and how you fixed it.

**Deliverable:** `mco2-troubleshooting.md` — a written incident report with:
- Timeline of events
- Symptoms observed
- Root cause identified
- Fix applied
- Prevention measures for the future

---

### Step 7 — Prepare the operations report (Week 15)

**Task:** Compile all evidence into a professional operations report.

**Detailed instructions:**

1. Gather all screenshots and outputs from Steps 1–6.
2. Organize them into the report template below.
3. Write a summary paragraph for each section describing what you did and what you learned.
4. Include a "Recommendations for Production" section with at least 3 actionable recommendations.

---

## MCO2 Documentation Template

### MCO2 Operations Report Template

Replace the dummy content below with your real values.

```markdown
# MCO2 — Systems Administration and Maintenance Report

## Student Information
- **Name:** Jane Doe
- **Student ID:** 12345678
- **Course:** SAM10 Systems Administration and Management | College of Computer Science | Angeles University Foundation | R.L.Natividad
- **Date:** 2026-08-04
- **Infrastructure:** AWS (ap-southeast-1)
- **Reference Architecture:** MCO1 deployment (gg-mco1-vpc, gg-mco1-web-a, gg-mco1-web-b, gg-mco1-db)

## 1. Monitoring & Observability

### 1.1 CloudWatch Dashboard
![Dashboard](mco2-cw-dashboard.png)

**Description:** The dashboard tracks CPU utilization, memory usage, disk usage, and instance status checks for all MCO1 resources. The CPU alarm (`gg-mco2-high-cpu`) fires when average CPU exceeds 80% for 15 minutes.

### 1.2 Key Metrics (sample data)
| Metric | Current Value | Threshold | Status |
| --- | --- | --- | --- |
| CPUUtilization (web-a) | 12% | 80% | OK |
| CPUUtilization (web-b) | 8% | 80% | OK |
| MemoryUtilization (web-a) | 45% | 90% | OK |
| DiskUtilization (/) | 32% | 85% | OK |
| StatusCheckFailed | 0 | 0 | OK |

### 1.3 CloudWatch Logs
![Logs](mco2-cw-logs.png)

**Description:** Nginx access logs are shipped to CloudWatch Logs group `/gg/mco2/nginx-access`. Log retention is set to 30 days.

## 2. Logging & Auditing

### 2.1 CloudTrail Configuration
![CloudTrail](mco2-cloudtrail.png)

**Description:** CloudTrail `gg-mco2-trail` is enabled for all regions with log file validation. Management events and S3 data events are recorded. Logs are stored in S3 bucket `gg-mco2-cloudtrail-logs-<account-id>` with lifecycle policy transitioning to Glacier after 90 days.

### 2.2 Log Retention
| Log Group | Retention |
| --- | --- |
| `/aws/cloudtrail/gg-mco2` | 90 days |
| `/gg/mco2/nginx-access` | 30 days |

![Log Retention](mco2-log-retention.png)

## 3. Backup & Disaster Recovery

### 3.1 Backup Strategy
| Resource | Backup Method | Frequency | Retention | RPO | RTO |
| --- | --- | --- | --- | --- | --- |
| RDS PostgreSQL | Automated snapshots + PITR | Daily + continuous | 7 days | ~1 second | ~5 minutes |
| RDS (manual snapshot) | Manual snapshot | Weekly | 30 days | Point-in-time | ~10 minutes |
| EBS (web root volume) | Snapshot | Weekly | 30 days | 1 week | ~30 minutes |
| S3 (static assets) | Versioning + CRR | Continuous | Forever | 0 | ~minutes |

### 3.2 Restore Test Results
![RDS Restore](mco2-rds-restore.png)

**RDS Restore Test:** Manual snapshot `gg-mco1-db-backup-2026-08-04` was restored to a test instance `gg-mco1-db-restore-test`. Connection test `SELECT 1;` returned `1`. Test instance deleted after verification. **Result: PASS.**

**EBS Restore Test:** Snapshot `gg-mco1-web-a-root-backup-2026-08-04` was restored to a new volume and attached to a test instance. Files verified intact. **Result: PASS.**

## 4. Security Hardening

### 4.1 SSH Hardening
| Setting | Before | After |
| --- | --- | --- |
| SSH Port | 22 | 2222 |
| Password Authentication | yes | no |
| Root Login | prohibit-password | no |

### 4.2 OS Patching
- Last patch date: 2026-08-04
- Security patches applied: 3 (CVE-2026-xxxx)
- Reboot required: no

### 4.3 IAM Least Privilege
![IAM Role](mco2-iam-role.png)

**EC2 instance role:** `gg-mco1-ec2-role` with a custom policy allowing only:
- `s3:GetObject`, `s3:PutObject` on `gg-mco1-static-assets-*`
- `cloudwatch:PutMetricData` on `GreenGrid/MCO2` namespace
- `logs:CreateLogStream`, `logs:PutLogEvents` on `/gg/mco2/*`

## 5. Automation

### 5.1 Weekly Patching Maintenance Window
![SSM Automation](mco2-ssm-automation.png)

**Description:** A maintenance window `gg-mco2-weekly-patch` runs every Sunday at 02:00 UTC. It applies security patches to all web server instances tagged `Project=SAM10-MCO1`. The window has a 2-hour duration and a 1-hour cutoff.

### 5.2 Log Retention Automation (optional)
A Lambda function `gg-mco2-log-rotation` runs daily at 00:00 UTC to enforce 30-day log retention across all CloudWatch log groups.

## 6. Troubleshooting Report

### 6.1 Incident: 502 Bad Gateway on Web Server

**Timeline:**
| Time | Event |
| --- | --- |
| 14:30 | Monitoring alarm fired: 5xx errors detected |
| 14:31 | SSHed into web server on port 2222 |
| 14:32 | Checked nginx status: `inactive (dead)` |
| 14:33 | Checked nginx error log: `bind() to 0.0.0.0:80 failed (98: Address already in use)` |
| 14:34 | Killed stale nginx process: `sudo kill -9 $(pgrep nginx)` |
| 14:35 | Restarted nginx: `sudo systemctl start nginx` |
| 14:36 | Verified: `curl -sS -o /dev/null -w "%{http_code}" http://localhost/` → `200` |
| 14:37 | Verified ALB health check: target healthy |

**Root Cause:** A previous nginx process was not properly terminated, leaving the port 80 socket in `TIME_WAIT` state. The new nginx instance could not bind to port 80.

**Fix:** Killed the stale process and restarted nginx.

**Prevention:** Add a pre-start script to the systemd unit that kills any stale nginx processes before starting:
```ini
[Service]
ExecStartPre=/bin/bash -c 'pkill -9 nginx || true'
```

## 7. Recommendations for Production

1. **Enable Multi-AZ for RDS** — currently running as single-AZ. Multi-AZ provides automatic failover with ~60 second RTO.
2. **Implement AWS Config conformance packs** — to continuously audit resource configurations against security baselines.
3. **Add a WAF (Web Application Firewall)** — in front of the ALB to protect against common web attacks (SQL injection, XSS).
4. **Implement Infrastructure as Code** — all resources should be defined in CloudFormation templates stored in git, not created manually via the console.
5. **Set up Cost Anomaly Detection** — to alert on unexpected spend spikes.

## 8. Sources
- AWS CloudWatch User Guide: https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/
- AWS CloudTrail User Guide: https://docs.aws.amazon.com/awscloudtrail/latest/userguide/
- AWS Backup User Guide: https://docs.aws.amazon.com/backup/latest/devguide/
- AWS Systems Manager User Guide: https://docs.aws.amazon.com/systems-manager/latest/userguide/
- AWS Well-Architected — Operational Excellence pillar: https://docs.aws.amazon.com/wellarchitected/latest/
- Module 8 (Monitoring), Module 9 (Logging), Module 10 (Incident Response), Module 11 (Backup/DR), Module 13 (Deployment), Module 15 (Automation) — AWS Skill Builder
```

---

## Checkpoint Questions

```{dropdown} Q1. What is the difference between a CloudWatch alarm and a CloudWatch dashboard?
**Answer.** A **dashboard** is a visual view of metrics (graphs, numbers). An **alarm** is an automated action triggered when a metric crosses a threshold (e.g., send an SNS notification, trigger a Lambda).
```
```{dropdown} Q2. Why enable CloudTrail log file validation?
**Answer.** Log file validation adds an SHA-256 hash to each log file, allowing you to **prove** that the logs haven't been tampered with — critical for forensic and compliance purposes.
```
```{dropdown} Q3. RDS automated backup retention is set to 7 days — what does this mean for PITR?
**Answer.** You can restore the database to **any point within the last 7 days** (second-level granularity). Beyond 7 days, only manual snapshots are available.
```
```{dropdown} Q4. Your EC2 instance role has `s3:PutObject` on the static assets bucket. Can it also `s3:DeleteObject`?
**Answer.** No — unless the policy explicitly includes `s3:DeleteObject`. IAM policies are **explicit allow** — if an action isn't listed, it's denied by default.
```
```{dropdown} Q5. A maintenance window runs for 2 hours but the patching takes 3 hours. What happens?
**Answer.** The maintenance window **cuts off** at the 2-hour mark (or the 1-hour cutoff, whichever comes first). Any instances still being patched are **aborted** and must be retried in the next window.
```

---

## Sources
- AWS CloudWatch User Guide: https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/
- AWS CloudTrail User Guide: https://docs.aws.amazon.com/awscloudtrail/latest/userguide/
- AWS Backup User Guide: https://docs.aws.amazon.com/backup/latest/devguide/
- AWS Systems Manager User Guide: https://docs.aws.amazon.com/systems-manager/latest/userguide/
- AWS Well-Architected Framework: https://aws.amazon.com/architecture/well-architected/
- AWS Security Best Practices: https://docs.aws.amazon.com/prescriptive-guidance/latest/security-best-practices/
- Module 8–15 — AWS Skill Builder "AWS Cloud Operations" learning path

<footer class="sam10-footer" style="margin-top:3em;text-align:center;color:#666;font-size:0.85em">
  <hr>
  <span>SAM10 Systems Administration and Management | College of Computer Science | Angeles University Foundation | R.L.Natividad</span> &mdash;
  <span>Systems Administration and Maintenance</span> &mdash;
  <span>Undergraduate course companion</span>
</footer>
