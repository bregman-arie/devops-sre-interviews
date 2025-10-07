# Google Cloud Platform (GCP) - Beginner Interview Mock #5 - Answer Key

> **Difficulty:** Beginner  
> **Duration:** ~30 minutes  
> **Goal:** Assess understanding of governance basics, data store selection, access patterns, resiliency concepts, and operational practices in GCP.

---

## 🧠 Section 1: Core Questions - Answers

### 1. What is the Organization Policy Service in GCP? Give two example policies and why they are useful.

**Organization Policy Service:** Lets you enforce guardrails (constraints) across the resource hierarchy (org → folder → project). Prevents misconfiguration and enforces compliance before resources are created.

**Examples:**
1. Restrict VM external IPs (`constraints/compute.vmExternalIpAccess`) → Helps reduce attack surface by forcing use of Cloud NAT / private networking.
2. Allowed regions (`constraints/gcp.resourceLocations`) → Ensures data residency / compliance.
Other common: Disable service account key creation, restrict public bucket access.

### 2. Explain the difference between resource labels and network tags. When do you use each?

**Labels:** Key/value pairs attached to many resource types (e.g., `env=prod`, `team=payments`). Used for cost reporting, filtering, automation, and identifying ownership. Appear in billing exports.

**Network Tags:** Simple strings on Compute Engine instances (and used indirectly with GKE nodes). Used exclusively for applying firewall rules & sometimes routes. Not for billing.

**Usage:** Use labels for governance, cost, and search; use network tags to target firewall rules or load balancer backend selection.

### 3. Compare Cloud SQL, Firestore, and BigQuery for a simple web application storing user profiles, transactions, and analytics events. Which would you use for each data type and why?

| Data Type | Best Service | Reason |
|-----------|--------------|--------|
| User profiles (moderate reads/writes, relational fields) | Firestore OR Cloud SQL (depends) | Firestore for flexible schema & real-time sync; Cloud SQL if strong relational joins needed |
| Financial / transactional records | Cloud SQL | ACID transactions, strong relational integrity |
| Analytics event stream (large append-only) | BigQuery | Scalable analytical queries, partitioning, low ops |

**Summary:** Cloud SQL for relational transactional integrity; Firestore for real-time app state; BigQuery for large-scale analytical queries.

### 4. What is a Cloud Run Job and how does it differ from a Cloud Run Service? Provide an example use case.

**Cloud Run Service:** Long-lived HTTP endpoint or event consumer; handles concurrent requests; scales based on traffic; always expects request/response or event.

**Cloud Run Job:** Runs a container to completion (batch/one-off). No HTTP server required. Executes tasks like ETL, exports, maintenance.

**Differences:**
- Invocation: Job run vs HTTP request
- Lifecycle: Finite (Job) vs continuous (Service)
- Concurrency: Jobs can use multiple tasks (parallelism) but each task runs to completion.

**Use Case:** Nightly data aggregation or cleanup script that processes records and exits.

### 5. How would you set up a basic alert for elevated HTTP 5xx error rates in a Cloud Run application? List the high‑level steps.

1. Deploy service with structured logging (includes status codes).
2. In Cloud Monitoring: create metric (built-in `run.googleapis.com/request_count` filtered by status >=500) or define log-based metric if custom.
3. Create alerting policy: condition = ratio (5xx / total) > threshold (e.g., 2%) for 5 minutes.
4. Add notification channels (email / Slack / PagerDuty).
5. (Optional) Add uptime check to detect full outage.

### 6. Define RPO and RTO. Give a beginner-friendly example using Cloud SQL backups.

**RPO (Recovery Point Objective):** Max acceptable data loss measured in time. If automated backups + PITR allow restoring to any point in last 24h with 5‑minute binlog granularity, RPO ≈ 5 minutes.

**RTO (Recovery Time Objective):** Max acceptable downtime to restore service. If your tested restore of Cloud SQL takes 20 minutes to provision + redirect app, RTO = 20 minutes.

**Example:** Nightly full backup + binary logs: You can roll forward to failure point (RPO small). Restoration and cutover takes 15–20 minutes (RTO 15–20m).

### 7. Explain signed URLs for Cloud Storage. When would you use them instead of making a bucket public or granting IAM roles?

**Signed URL:** Time-limited URL granting anyone (no auth) access to an object (GET/PUT) using a cryptographic signature by a service account.

**Use Cases:** Temporary controlled download/upload for users without GCP identities (e.g., user uploads avatar directly to bucket). Avoids broad bucket public access and avoids provisioning per-user IAM.

**Benefits:** Principle of least privilege, expiration, revocable by rotating key, no proxy server bandwidth needed when using direct upload pattern.

### 8. What is Error Reporting in Google Cloud Operations Suite and how does it help during incident response?

**Error Reporting:** Automatically aggregates and de-duplicates application exceptions (stack traces) from logs. Shows counts, first seen, last seen, and new error detection.

**Helps By:**
- Rapidly highlighting new regressions
- Reducing noise vs raw logs
- Alerting on new error types
- Linking stack traces to source (when integrated)
- Prioritizing fix effort by frequency

---

## ⚙️ Section 2: Scenario - Answer

**Goals:** Global low-latency reads, real-time collaboration, analytics, nightly export, minimal ops.

**Architecture:**
1. **Authentication:** Firebase Auth (easy integration + tokens for front-end calls).
2. **Primary Data Store:** Firestore (multi-region) for notes (documents: user/collection/note). Real-time sync & offline support.
3. **Transactions:** If minimal, embed in Firestore; if strong relational logic emerges, add Cloud SQL later (deferred now for simplicity).
4. **Analytics Events:** Client/front-end sends usage events to Pub/Sub → Dataflow (optional) or simple Cloud Run service → BigQuery partitioned table (DATE ingestion).
5. **Global Performance:** Firestore multi-region + optionally Cloud CDN for static assets (front-end hosting via Cloud Storage + HTTPS Load Balancer + CDN).
6. **Real-Time Updates:** Firestore listeners stream changes instantly to clients.
7. **Nightly Export:** Cloud Scheduler (00:30 UTC) → Pub/Sub → Cloud Run Job fetches BigQuery query results (daily summary) + Firestore metadata → writes CSV to Cloud Storage `exports/` (Nearline transition after 30d lifecycle rule).
8. **Security:** IAM: least privilege service accounts (`note-api-sa`, `analytics-writer-sa`, `export-job-sa`). Firestore security rules enforce per-user note access. Secret Manager for API keys (e.g., third-party integration).
9. **Monitoring:** Cloud Monitoring dashboard: request latency (Cloud Run), Firestore read/write rates, Pub/Sub backlog. Log-based metric for export job failures.
10. **Cost Controls:** Labels (`env`, `service`), budget alerts (50/80/100%), BigQuery partition pruning, lifecycle rules (Nearline at 30d, Archive at 365d) for exports.

**Flow Concept:**
```
Users ↔ Front-End (CDN) ↔ Cloud Run (API) ↔ Firestore (notes)
                            ↘ Pub/Sub (events) → BigQuery (analytics)
Scheduler → Pub/Sub → Cloud Run Job (export) → Cloud Storage (CSV)
```

**Why Beginner-Friendly:** Fully managed serverless components; incremental complexity; avoids early relational overhead.

---

## 🧩 Section 3: Problem-Solving - Answer

**Objective:** Simple daily ingestion with reliability & low cost.

**Services & Flow:**
1. **Scheduling:** Cloud Scheduler (cron: `30 0 * * *`) → Pub/Sub topic `daily-fetch`.
2. **Fetch & Store Raw:** Cloud Run Job (container with fetch script) subscribed via push-trigger Cloud Function or direct Pub/Sub push to Cloud Run service that internally triggers Job. Job calls external REST API, stores raw JSON in Cloud Storage bucket `raw/dt=YYYY-MM-DD/`.
3. **Transform:** Same job (second step) or separate Cloud Run Job parses JSON, extracts curated fields, writes newline-delimited JSON to temporary Cloud Storage path (`staging/`). Then loads into BigQuery partitioned table (`api_events` partition on ingestion date) using BigQuery load API.
4. **History Retention:** Bucket lifecycle: transition raw older than 30 days to Nearline, delete after 90 days. BigQuery table TTL 90 days (partition expiration) for cost control.
5. **Failure Handling:**
   - Pub/Sub retry (if using push) or manual rerun of Job.
   - Job writes success/fail status to log; log-based metric triggers alert if failure count ≥1.
   - Idempotency: Use date-based object naming; on rerun, overwrite or check existence.
6. **Alerting:** Cloud Monitoring alert on log-based metric `daily_ingest_failures` if >0 between 00:30–01:00 UTC + no corresponding success metric.
7. **Secrets:** External API key stored in Secret Manager; injected at runtime.
8. **Cost Optimization:** Single daily batch (avoid per-event streaming cost), partitioned BigQuery table (scans limited by date), compressed JSON (gzip) in storage.

**Flow Diagram:**
```
Cloud Scheduler → Pub/Sub → Cloud Run Job → External API
                                   ↓
                              Cloud Storage (raw)
                                   ↓ transform
                              Cloud Storage (staging) → BigQuery (partitioned)
                                   ↓ logs
                              Monitoring Alert
```

**Why This Works:** Minimal moving parts, recoverable, cost-aware (batch, partitioning, lifecycle policies), easy to extend.

---
