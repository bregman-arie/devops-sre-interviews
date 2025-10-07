# Google Cloud Platform (GCP) - Intermediate Interview Mock #3 - Answer Key

> **Difficulty:** Intermediate  
> **Duration:** ~45 minutes  
> **Goal:** Assess architectural decision making across compute options, security boundaries, scalability, data handling, and operational excellence on GCP.

---

## 🧠 Section 1: Core Questions - Answers

### 1. You’re designing a scalable web application on Google Cloud Platform. How would you decide between Cloud Run, GKE, and Compute Engine for deploying your application, and what are the trade-offs of each?

| Aspect | Cloud Run | GKE | Compute Engine |
|--------|-----------|-----|----------------|
| Ops Overhead | Very low (fully managed) | Medium/High (cluster lifecycle, patching) | Highest (OS, scaling, patching) |
| Scaling | Per-request concurrency; scale to zero | Pod + node autoscaling (HPA, VPA, CA) | Manual or MIG autoscaling |
| Startup / Cold | Possible cold starts; mitigation via min instances | Warm pods if capacity reserved | Always warm (VMs) |
| Flexibility | Single container per revision (no native sidecars) | Full pod model (sidecars, mesh) | Anything (daemon, stateful) |
| Networking Control | Basic (VPC connector, egress settings) | Advanced (NetworkPolicy, CNI, service mesh) | Full (sysctl, custom routing) |
| Stateful / Daemons | Not suited | StatefulSets available | Native (disks, custom) |
| Cost Low Traffic | Best (scale to zero) | Idle cluster cost | Always paying for VMs |
| Advanced Scheduling | Limited | Rich (affinity, tolerations) | N/A (per VM) |
| Use Cases | Stateless APIs, web, gRPC, event handlers | Complex microservices, sidecars, custom networking | Legacy apps, specialized runtimes, long-lived jobs |

**Decision Heuristic:** Start with Cloud Run unless you need (a) sidecars/mesh, (b) tight latency tuning, (c) complex networking/security policies → then GKE. Choose Compute Engine for legacy or specialized workloads requiring OS-level control or persistent processes unsuitable for container orchestration.

### 2. When would Cloud Tasks be preferable to Pub/Sub or Eventarc for asynchronous workloads? Provide two concrete examples.

**Cloud Tasks Strengths:** Per-task scheduling, rate limiting, per-queue retry/TTL, ordered delivery within a task queue, guaranteed exactly-once execution of HTTP target (idempotency still recommended). Good for controlling downstream pressure.

**Example 1:** Throttled outbound API calls (e.g., calling a third-party API limited to 50 req/s) — Cloud Tasks enforces dispatch rate.
**Example 2:** User-initiated background actions (email confirmation, invoice generation) needing per-task retry with exponential backoff and visibility into individual task status.

**Pub/Sub Better:** High-throughput fan-out, streaming pipelines, broadcast to many subscribers.
**Eventarc Better:** Event routing from first-party sources (Cloud Storage object finalize → target) without custom publishers.

### 3. Compare Cloud SQL + Redis cache vs Cloud Spanner for a multi-region read-heavy SaaS product (focus: consistency, scaling, cost, ops).

| Dimension | Cloud SQL + Redis | Cloud Spanner |
|-----------|-------------------|---------------|
| Consistency | Primary-write + async replicas; eventual cross-region | Global strongly consistent (TrueTime) |
| Read Scaling | Read replicas per region + cache | Horizontal partitioning built-in |
| Write Scaling | Single-region primary (vertical scale) | Multi-region writes (with latency trade-offs) |
| Failover | Promotion required (may have lag) | Automatic within multi-region config |
| Caching Need | Often essential (Redis/Memorystore) | Often optional for many workloads |
| Schema | Relational (standard) | Relational (subset + interleaving) |
| Ops | Moderate (patching, failover drills, replica tuning) | Lower day-2 ops; capacity planning via nodes |
| Cost | Lower for small/medium scale | Higher baseline; efficient at large scale |

**Guideline:** Use Cloud SQL + cache until (a) global writes, (b) unscalable primary, or (c) strong multi-region consistency required → then consider Spanner.

### 4. How do you enforce and monitor SLOs (availability & latency) for a Cloud Run based microservice? Mention metrics, alerting strategy, and error budget usage.

**SLO Definition:** 99.5% availability, p95 latency < 400ms (over 30 days). 
**Metrics:**
- Availability = 1 - (5xx requests / total requests) using `run.googleapis.com/request_count` label filter.
- Latency: custom distribution metric via OpenTelemetry export or Cloud Trace sample aggregated to p95.
**Alerting:** Multi-window multi-burn alerts: fast-burn (2h window > 5% budget burn), slow-burn (24h > 25% of period). Separate latency alert if p95 > threshold 5m.
**Error Budget:** If burn rate > target, freeze feature rollouts & prioritize reliability (e.g., investigate spikes, optimize cold start, tune concurrency).
**Dashboards:** RED (Rate, Errors, Duration) + deployment markers.

### 5. Explain strategies to minimize cold starts in Cloud Run and their trade-offs.

| Strategy | Benefit | Trade-off |
|----------|---------|-----------|
| Min Instances (e.g., 1–2) | Warm container always ready | Fixed baseline cost |
| Smaller Image / Fast Start | Faster spin-up | Engineering effort to slim image |
| Increase Concurrency | Fewer instances needed → fewer cold spins | Potential higher tail latency under load |
| Region Selection (closer to users) | Lower network + startup impact | May increase multi-region complexity |
| Avoid Heavy Init (lazy load) | Shortens cold path | Complexity managing deferred errors |
| Use Shared Libraries Layer Cache | Faster bootstrap | Build pipeline complexity |

### 6. What security controls would you layer to prevent data exfiltration from a BigQuery dataset containing regulated data? (At least five controls.)

1. **VPC Service Controls** perimeter around analytics & storage projects.  
2. **Column / Row-Level Security** (mask PII, restrict tenant visibility).  
3. **CMEK** with tightly scoped KMS key IAM (rotate & monitor usage).  
4. **IAM Least Privilege** (separate `dataViewer` vs `queryJobUser`; no broad `owner`).  
5. **Audit Log Export + Alerting** on large extract jobs / unusual egress bytes.  
6. **Query Result Size Quotas** / restrict external query destinations.  
7. **Disable External Copy / Cloud Storage Export** for non-privileged roles.  
8. **Private Service Connect** for private API access (avoid public endpoints).  

### 7. Describe a pattern for safe multi-tenant per-customer encryption in GCP using CMEK and key rotation without service downtime.

**Pattern:**
1. Each tenant has a dedicated KMS key (key ring scoped per region).  
2. Application stores only key references (not material).  
3. Data encryption handled by BigQuery / GCS / Spanner with CMEK pointer.  
4. Rotation: create new key version → update resource to point to new version (many services auto-re-encrypt lazily) → monitor access logs → after validation disable old version.  
5. Revocation: disable key (cryptographic lock) achieves immediate denial.  
6. Access: per-tenant service account granted `roles/cloudkms.cryptoKeyEncrypterDecrypter` only on that key.  
7. Observability: export KMS audit logs; anomaly detection on unusual decrypt count.  

### 8. How would you design an automated cost anomaly detection workflow using native GCP tools?

**Flow:** Billing Export → BigQuery → Scheduled Query / Looker Studio anomaly detection → Pub/Sub → Notification.

**Steps:**
1. Enable daily (or hourly) detailed billing export to BigQuery.  
2. Create partitioned table; scheduled query computes expected spend (e.g., EWMA or % deviation vs 7-day rolling avg by service & project).  
3. Flag rows where actual > threshold (e.g., > 3σ or > 40% deviation & > $X absolute).  
4. Write anomalies into `cost_anomalies` table.  
5. Scheduled Cloud Function / Cloud Run job queries anomalies for last hour → publishes formatted message to Pub/Sub → Cloud Run notifier sends Slack/email.  
6. (Optional) Trigger automated remediation (stop idle MIG) if tagged environment=dev.  
7. Dashboard in Looker Studio for historical anomaly trend & MTTR tracking.  

---

## ⚙️ Section 2: Scenario - Answer

**Recommendation:** Start on Cloud Run (simplicity, low ops) with architecture prepared for later selective GKE adoption if advanced networking/sidecars emerge.

**Target Platform Components:**
- **Compute:** Each stateless service → Cloud Run (per service). Batch / scheduled jobs → Cloud Run Jobs + Cloud Scheduler. Future: If service requires sidecar (e.g., Envoy / OpenTelemetry Collector) migrate that service to a small GKE cluster (hybrid). 
- **Relational Store:** Cloud SQL (PostgreSQL) with HA (regional) + read replica planned in second region for future expansion. Add Memorystore (Redis) only after profiling (avoid premature caching).  
- **Analytics Events:** Pub/Sub → Dataflow (or initially Cloud Run consumer) → BigQuery partitioned (daily) + aggregated tables.  
- **Secrets:** Secret Manager + per-service service account with `secretAccessor`; no environment embedding of secrets at build time.  
- **Networking:** Global HTTPS Load Balancer fronting Cloud Run services (custom domain, Cloud Armor basic rules, optional Cloud CDN for static assets). Private egress to DB via Cloud SQL connector (no public DB IP).  
- **Config & Deploy:** Cloud Build pipeline (build → scan → sign → deploy with progressive traffic splitting). Use Cloud Run revision traffic percentages (10% canary → 50% → 100%). Rollback = revert traffic to previous stable revision instantly.  
- **Observability:**
  - Logs: Structured (JSON) with trace IDs; log-based metrics for error rates.  
  - Tracing: Cloud Trace (sampling 10% baseline; raise during incident).  
  - Metrics: RED dashboard (requests, errors, duration), DB connection saturation, Pub/Sub backlog.  
  - SLOs stored in a config repo; burn-rate alerts (multi-window) via Monitoring.  
- **Multi-Region Readiness:**
  - Phase 1: Keep single write region; add read replica + global load balancer latency steering for read-only endpoints (e.g., catalog).  
  - Phase 2 (6 months): Evaluate multi-primary (maybe Spanner) if write latency or failover RTO become constraints.  
  - Store user-generated static assets in dual-region GCS bucket to reduce latency before full multi-region writes.  
- **Security:** Org Policy: restrict external IPs, enforce CMEK on BigQuery if needed later; Cloud Armor basic OWASP; IAM least privilege (no primitive roles). VPC Service Controls perimeter around analytics when Dataflow added.  
- **Governance:** Labels: `service`, `env`, `owner`, `cost-center`; billing export + anomaly detection scheduled query.  

**Why Not GKE Now?** Adds cluster admin load (patching, node pools, autoscaling tuning) without immediate need for pod-level complexity. **Why Not Compute Engine?** Higher toil (OS patching, scaling) for purely stateless services. Hybrid allowed later service-by-service.  

**Rollout Safety:** Progressive traffic splitting + automated smoke tests hitting health & synthetic journeys; error budget policy halts promotions if burn > threshold.  

---

## 🧩 Section 3: Problem-Solving - Answer

**Webhook Processing Architecture:**

```
3rd-Party Provider → HTTPS LB → Cloud Run Ingest (auth, basic validation) → Cloud Tasks (queue per provider)
                                         ↓ (idempotency check cache)
                                   Pub/Sub (fan-out events)
                                         ↓
         Fraud Scoring (Cloud Run)  Ledger Writer (Cloud Run / Cloud SQL)  Notification (Cloud Run)
```

**Ingestion & Ordering:** Cloud Run ingest service extracts `transaction_id`; writes idempotency key (transaction_id + hash) to Memorystore (Redis) with short TTL; enqueues Cloud Task (queue configured with max dispatch rate & retry policy). Ordering preserved per queue if single worker concurrency=1 per `transaction_id` bucket (or create sharded queues by hash). For large scale requiring strict per key ordering, alternative: Pub/Sub with ordering keys then a Dataflow / subscriber pulling sequentially per key; Cloud Tasks chosen here for fine-grained retry control.

**Durability & Retry:** Cloud Tasks persists payload; exponential backoff policy with max attempts; permanently failed tasks published to a dead-letter Pub/Sub topic for manual inspection.

**Idempotency Storage:** Redis SETNX (or Cloud SQL table) ensures duplicates short-circuited; ledger writer uses database unique constraint on `transaction_id` to guarantee at-least-once upstream becomes exactly-once effect.

**Fan-out:** After core processing, publish canonical event to Pub/Sub topic `payment.processed`; subscribers (fraud scoring, notification) are independent → resilience & decoupling.

**Failure Handling:**
- Business validation failure: log structured event (type=validation_failed), do not retry.
- Transient dependency failure: rely on Cloud Tasks retry (HTTP 5xx) + alert if retry count high.
- Downstream ledger DB latency: circuit breaker (fast fail, queue length metric triggers scale-out of worker revision). 

**Monitoring & Alerting:**
- Metrics: queue depth (Cloud Tasks), task age (95th), ingestion request latency, duplicate drop count, ledger commit failures, Pub/Sub ack latency. 
- Alerts: queue age > X mins (backlog), duplicate rate anomaly, error rate > 2% 5m. 
- Tracing: propagate trace header from ingest through worker & fan-out. 
- Audit: Cloud Logging for each state transition (RECEIVED, DEDUP_SKIPPED, PROCESSED, DLQ). 

**Scaling:** Cloud Run workers autoscale on request concurrency from Cloud Tasks; set max concurrent requests low (e.g., 10) if CPU-bound signature verification. For burst absorption, keep Cloud Tasks queue as shock absorber (protect DB). 

**Security:** mTLS or signed shared secret from provider; WAF (Cloud Armor) for generic abuse; secret rotation stored in Secret Manager; principle of least privilege for worker SA (no broad project roles). 

**Why This Works:** Absorbs spikes (queue), preserves ordering per key, ensures idempotency, isolates failures via fan-out, and enables operational visibility & rapid remediation.

---
