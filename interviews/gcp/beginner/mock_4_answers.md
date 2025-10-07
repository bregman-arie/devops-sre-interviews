# Google Cloud Platform (GCP) - Beginner Interview Mock #4 - Answer Key

> **Difficulty:** Beginner  
> **Duration:** ~30 minutes  
> **Goal:** Assess foundational knowledge around networking basics, load balancing, DNS, container delivery, infrastructure as code, database reliability, caching, and logging/auditing in GCP.

---

## 🧠 Section 1: Core Questions - Answers

### 1. What are the main types of Google Cloud load balancers (HTTP(S), SSL Proxy, TCP Proxy, Network, Internal) and when would you use an HTTP(S) load balancer vs a Network Load Balancer?

**Global External:**
- **HTTP(S) Load Balancer**: Layer 7, global anycast IP, intelligent routing, URL-based routing, Cloud CDN integration, managed SSL certs.
- **SSL Proxy / TCP Proxy**: Layer 4/7 hybrid for non-HTTP encrypted (SSL Proxy) or raw TCP (TCP Proxy) with global front end.

**Regional External:**
- **Network Load Balancer (L4)**: Pass-through, regional, good for non-HTTP protocols needing low latency.

**Internal:**
- **Internal HTTP(S) / Internal TCP/UDP / Internal passthrough NLB**: For internal-only service exposure inside a VPC / hybrid.

**Choose HTTP(S) LB When:**
- Need global distribution, URL path / host-based routing, Cloud CDN, managed certs, single global IP.

**Choose Network LB When:**
- Need simple regional L4 forwarding for non-HTTP protocols (e.g., custom TCP), minimal features, ultra-low latency.

### 2. Explain the basic purpose of Cloud DNS. How does it integrate with global load balancing?

**Cloud DNS:** Managed, scalable, low-latency authoritative DNS service. Host public or private zones, create A / AAAA / CNAME / TXT / MX / etc. records.

**Integration:** You map a DNS name (e.g., `api.example.com`) via an A/AAAA record to the global anycast IP of an HTTP(S) Load Balancer. Users resolve through Cloud DNS to nearest edge POP; load balancer routes to healthy backends globally. DNS TTL kept moderate (e.g., 300s) for agility while caching.

### 3. Compare Artifact Registry and Container Registry. Why should new projects prefer Artifact Registry?

**Container Registry (gcr.io):** Legacy service, regional/multi-regional buckets storing images; limited artifact types, fewer fine-grained features.

**Artifact Registry:** Modern unified repository supporting containers, language packages (Maven, npm, Python, etc.), per-repo IAM, VPC SC integration, CMEK, better vulnerability scanning integration.

**Prefer Artifact Registry Because:** More secure granularity, multi-format support, future-focused, integrated scanning & policy, regional control, deprecation path for Container Registry.

### 4. What is Terraform and why would a team adopt it for GCP resource management? Give a simple example workflow.

**Terraform:** Open-source IaC tool using declarative configuration to provision cloud resources reproducibly. Uses the GCP provider to create/update/destroy resources.

**Reasons to Adopt:**
- Versioned infrastructure in Git (audit & rollback)
- Consistency across environments (dev/staging/prod)
- Review changes (plan) before applying
- Reduce manual console drift & misconfiguration

**Basic Workflow:**
1. Write `main.tf` (e.g., define a `google_storage_bucket`).
2. `terraform init` (downloads providers).
3. `terraform plan` (shows proposed changes).
4. Code review + merge.
5. CI runs `terraform apply` (applies desired state).
6. Store state in remote backend (e.g., Google Cloud Storage) for team collaboration.

### 5. How do you enable high availability and backups for Cloud SQL at a beginner level?

**High Availability:**
- Select HA (regional) configuration (creates primary + standby in different zones, automatic failover).
- Use private IP + authorized networks/firewalls.

**Backups:**
- Enable automated daily backups + point-in-time recovery (binary logs) for MySQL/Postgres.
- Test restore periodically.

**Other Basics:** Set maintenance window, use strong passwords / IAM DB authentication if supported, monitor storage usage to avoid outages.

### 6. What is Cloud CDN and when does it help? Mention required components to enable it for a static site.

**Cloud CDN:** Global edge caching for HTTP(S) content behind Google’s network; reduces latency and origin load.

**Helps When:**
- Static or cacheable assets (images, JS, CSS) served globally
- Reducing egress cost & improving time to first byte

**Requirements for Static Site:**
1. Backend bucket or service (e.g., Cloud Storage bucket or Cloud Run service)
2. Global external HTTP(S) Load Balancer
3. Enable Cloud CDN on backend
4. (Optional) Cloud DNS mapping + managed SSL cert
5. Appropriate cache-control headers on objects

### 7. What are Cloud Audit Logs (Admin, Data Access, System Event, Policy) and why are they important?

**Log Types:**
- **Admin Activity Logs:** Management operations (always on, no charge for ingestion)
- **Data Access Logs:** Reads/writes to data plane (configurable, may cost)
- **System Event Logs:** Google system actions (e.g., infrastructure maintenance)
- **Policy Denied Logs:** Requests denied by Org Policy / IAM

**Importance:** For security investigations, compliance (who changed what & when), anomaly detection, change tracking, incident response root cause. They create accountability and help satisfy audit frameworks.

### 8. List four beginner-friendly logging & observability best practices in GCP.

1. **Structured Logging:** Emit JSON logs (severity, trace ID) for easier filtering & metrics.
2. **Log-based Metrics:** Create metrics (e.g., error_count) to power alerts instead of scanning raw logs.
3. **Retention Policies:** Export long-term logs to cheaper storage (BigQuery / Cloud Storage) and keep high-detail logs short-lived.
4. **Dashboards & Alerts Early:** Create minimal Cloud Monitoring dashboard + error/latency SLIs with alerts; iterate later.
5. *(Bonus)* **Label Consistency:** Include `env`, `service`, `version` fields for correlation.

---

## ⚙️ Section 2: Scenario - Answer

**Goal:** Modernize manual VM-based API + batch job into managed, auto-scaling, low-ops stack.

**Architecture:**
1. **Container Build:** Source in GitHub → Cloud Build builds container → push to Artifact Registry.
2. **Runtime (API):** Deploy container to Cloud Run (public HTTP). Configure min instances = 0 dev / 1 prod, concurrency (e.g., 80) for efficiency.
3. **Scheduled Job:** Cloud Scheduler (cron 0 1 * * *) → Pub/Sub topic → Cloud Run Job or Cloud Function (pulls latest image / executes aggregation logic).
4. **Secrets:** Secret Manager for DB credentials / API keys; Cloud Run configured with secret environment variables or runtime access using service account with `secretAccessor`.
5. **Networking:** Global HTTPS Load Balancer optional if custom domain + Cloud CDN needed; otherwise direct Cloud Run domain + Managed SSL. Private DB access (Cloud SQL) via Cloud SQL connector.
6. **Database (if needed):** Cloud SQL (single zone to start, HA later) with automated backups + PITR.
7. **Monitoring & Logging:** Cloud Monitoring dashboards (latency, req/s, errors). Log-based metric: count severity>=ERROR for alert (5xx >2% for 5m). Uptime check on `/health` endpoint.
8. **Cost Controls:** Budget with 50/80/100% alerts; lifecycle on storage; minimal idle instances.
9. **Deployment:** Staged rollout: deploy new revision with percentage traffic (e.g., 10% canary) then promote to 100%.

**Flow Concept:**
```
Git Push → Cloud Build → Artifact Registry → Cloud Run (API)
                               ↑                 ↓ logs/metrics
Cloud Scheduler → Pub/Sub → Cloud Run Job (batch) → Cloud SQL / Storage
Secret Manager → (access by service accounts)
```

**Why This Meets Requirements:** Serverless auto-scaling, versioned revisions, secure secret storage, reliable scheduling, basic monitoring, cost visibility.

---

## 🧩 Section 3: Problem-Solving - Answer

**Objective:** Centralize visibility & retain necessary audit data cost-effectively.

**Projects:** `app-dev`, `app-staging`, `app-prod` (separate). Optional shared logging project if scaling later.

**Services Used:**
- Cloud Logging (per project) + Log Sinks
- Cloud Monitoring (dashboards, alerting policies)
- BigQuery (summary / analytics retention 180 days)
- Cloud Storage (raw detailed log export 30→180 day tiered retention optional)

**Retention Strategy:**
1. Keep default project logging retention (≈30 days) for detailed troubleshooting.
2. Create aggregated log sink (include all three projects) with inclusion filter for summarized fields (e.g., only structured API request logs map-reduced) → BigQuery dataset (`central_logs`) partitioned by day, TTL 180 days.
3. Optional second sink for only Audit Logs (Admin + Data Access) → BigQuery (same dataset) to correlate changes.

**Filters (Examples):**
```
resource.type="cloud_run_revision" AND severity>="ERROR"
logName:"activity" (for Admin Activity logs)
```

**Alerting:**
- Create log-based metric: `api_5xx_count` (filter severity>=ERROR AND httpRequest.status>=500)
- Create counter metric for total requests (`httpRequest.status:*`)
- Alert policy: condition (api_5xx_count / total_requests) > 0.02 for 5 min (using MQL or ratio metric)
- Notification channel: email / Slack webhook.

**IAM Roles:**
- Project level: developers = `roles/logging.viewer`; ops = `roles/monitoring.editor` + `roles/logging.viewer`; security = `roles/logging.privateLogViewer` (to view Data Access/Audit)
- BigQuery dataset: grant read to security/ops group only; deny broad dev access.

**Cost Minimization:**
- Export only necessary fields (use log sinks with field restrictions later if needed)
- Partitioned BigQuery tables reduce scanned bytes
- Avoid exporting verbose DEBUG logs

**Summary Plan:** Use per-project detailed logs for near-term debugging, aggregated curated subset + audit logs centrally for medium-term compliance & trend analysis, with alerting built on log-based metrics rather than constant querying.

---
