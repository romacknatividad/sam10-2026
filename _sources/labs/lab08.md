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

# Lab 8 — Module 8: Monitoring & Observability

**Week 8 · AWS Cloud Operations · Module 8 · Monitoring & Observability**
Complete AWS Skill Builder "**Monitoring, Logging, and Metrics on AWS**" (Operations module).

## Module summary
- **CloudWatch Metrics** (time-series: CPU, network, custom), **Alarms**, **Dashboards**.
- **CloudWatch Logs** → **Insights** (SQL-like queries), subscription filters.
- **X-Ray** (distributed tracing), **Service Lens**, **Contributor Insights**.
- What the **AWS SLA** does and doesn't cover (it's **not** a monitoring substitute).

## Hands-on activity — build a sample dashboard JSON
```bash
# 8.1  A CloudWatch dashboard definition (the shape the console renders)
cat > /tmp/gg-dashboard.json <<'JSON'
{
  "widgets": [
    {"type":"metric","x":0,"y":0,"w":12,"h":6,
     "properties":{"metrics":[["AWS/EC2","CPUUtilization","InstanceId","i-1234"]],
                    "view":"timeSeries","region":"us-east-1","title":"Web CPU %"}},
    {"type":"log","x":12,"y":0,"w":12,"h":6,
     "properties":{"region":"us-east-1","title":"5xx errors",
                    "query":"fields @timestamp, @message | filter status='503' | sort @timestamp desc | limit 20"}}
  ]
}
JSON

# 8.2  Validate JSON shape (no AWS call)
python3 -c "import json; d=json.load(open('/tmp/gg-dashboard.json')); assert len(d['widgets'])==2; print('dashboard parse OK')"

# 8.3  Signal vs noise (exam concept)
echo "Good alarm: CPU >80% over 3 consecutive 1-min periods. Bad alarm: 1 raw spike."
```

## Artifact
`/tmp/gg-dashboard.json` + screenshot of a **sample CloudWatch dashboard** you build in the console
(from the free tier), plus the Module 8 quiz (≥ 80%).

## Checkpoint

````{dropdown} Q1. CloudWatch metric datum granularity defaults to 60 s, but some metrics come at 1 s — name two that do.
**EC2 CPUCreditBalance/CPUCreditUsage** (per-second on T2/T3) and **Lambda** metrics (per-invocation →
1 s or finer) — both are high-resolution-capable.
```
`````{dropdown} Q2. A CloudWatch alarm on `CPUUtilization` fires but the server is actually fine — likely cause?
A **noisy-neighbor or burst-balance** issue, or the alarm uses a single 1-min sample
(no `EvaluationPeriods` > 1) → fix the **alarm math / period / threshold**.
`````
```{dropdown} Q3. CloudWatch Logs `subscription filter` to Lambda — what problem does it solve?
You ship **live log events** to a Lambda (for alerting / redaction / fan-out to a SIEM) **without
polling**, decoupling log producers from consumers.
```
`````{dropdown} Q4. Metric math: `SEARCH('{AWS/EC2,InstanceId} MetricName=\"CPUUtilization\"', 'Average', 60)` — what's the purpose?
`SEARCH` finds the metric **across many instances automatically** (no hard-coded Id) → you can
build an alarm that fires if **any** web server exceeds threshold, then alarm on the `MAX`.
`````
```{dropdown} Q5. X-Ray adds latency overhead — when do you leave it on?
Leave **sampling** on (1 req/sec per instance default) in production; **full-trace** only when
debugging the specific call path (then turn it off to recover performance).
```

## Sources
- Amazon CloudWatch: https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/
- CloudWatch Logs insights: https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/AnalyzingLogData.html
- AWS X-Ray: https://docs.aws.amazon.com/xray/latest/dev/
- Module 8: AWS Skill Builder "Monitoring, Logging, and Metrics on AWS"
