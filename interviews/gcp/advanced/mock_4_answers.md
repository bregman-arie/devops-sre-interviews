# Google Cloud Platform (GCP) - Advanced Interview Mock #4 - Answer Key

> **Difficulty:** Advanced  
> **Duration:** ~60 minutes  
> **Goal:** Evaluate mastery of cross-boundary networking, high-throughput data & ML pipelines, reliability automation, and security operations.

---

## 🧠 Section 1: Core Questions - Answers

### 1. Architect a hybrid multi-cloud ingress layer using Global External HTTP(S) Load Balancer + Cloud Armor + Cloud CDN that can fail traffic to an alternate cloud while preserving client affinity. What mechanisms enable health detection and state continuity?

**Pattern:** Global LB (anycast IP) → backend services (primary GCP, standby external origin via external NEG / hybrid connectivity). Cloud Armor for WAF + rate limiting; Cloud CDN for edge caching static assets.

**Health & Failover:** Multi-region health checks (HTTP/gRPC) + synthetic probe Cloud Functions verifying deeper dependencies. External NEG referencing Hybrid Connectivity (Cloud Interconnect / VPN) to alternate cloud ingress. If primary backend 5xx/unhealthy threshold crossed, LB shifts weighting to secondary.

**State Continuity:** Stateless JWT sessions; sticky-affinity cookie only for short-lived flows. Shared global session/feature flags in Spanner or Memorystore replicated. Client-affinity after failover maintained via consistent hashing token (e.g., user ID mod region set). Edge cache keys include version to prevent mixing partially upgraded assets.

### 2. Explain how to build a low-latency feature store architecture supporting both streaming (≤2s) and historical batch backfill at 10+ TB/day using GCP services.

**Streaming Path:** Pub/Sub (events) → Dataflow streaming (stateful aggregation, TTL windows) → Vertex AI Feature Store (online store) + Bigtable (low-latency) for hot feature reads.

**Batch Backfill:** BigQuery (raw partitioned) → Dataflow/BigQuery SQL transform → bulk import to Feature Store offline store (BigQuery) + parallel Bigtable writes. Use checkpointed export shards.

**Convergence:** Periodic reconciliation job compares streaming state vs batch recompute; anomalies trigger state repair.

**Scalability:** Autoscaling Dataflow with Streaming Engine; per-feature family separate pipelines to isolate lag. Schema registry ensures forward-compatible field additions.

### 3. Design a unified policy enforcement model using Organization Policy, IAM Conditions, Policy Controller, and VPC Service Controls to prevent unauthorized copying of sensitive BigQuery datasets to external projects. Provide enforcement + detection flows.

**Enforcement Layers:**
1. Org Policy: Restrict resource locations + disable cross-project dataset copy for tagged projects (combined with custom Cloud Function guard if needed).  
2. VPC SC: Perimeter around sensitive projects stops data exfil via API to outside perimeter.  
3. IAM Conditions: Grant `bigquery.dataViewer` only if request.ip in corporate ranges & request.time < expiry.  
4. Policy Controller: Deny creation of datasets without required policy tags (sensitivity=confidential).  

**Detection:** Audit logs (BigQuery Data Access) exported to BigQuery; scheduled query detects large export jobs or COPY to non-whitelisted project; Pub/Sub alert → Cloud Function triggers manual approval workflow for exception.

### 4. Compare Dataflow vs Flink-on-GKE vs Spark (Dataproc) for a pipeline requiring complex event-time joins, low watermark lag, and exactly-once semantics at 500K events/sec. Provide a selection rationale.

| Aspect | Dataflow | Flink on GKE | Spark (Structured Streaming) |
|--------|----------|--------------|------------------------------|
| Ops Overhead | Low (managed) | High (cluster mgmt) | Medium (Dataproc ephemeral) |
| Event-Time Joins | Strong (Beam API) | Strong | Moderate (joins cost, micro-batch) |
| Latency | Seconds to sub-second | Sub-second (tuned) | Micro-batch (higher) |
| Exactly-Once | Provided (idempotent sinks) | Provided (stateful) | Achievable with checkpointing |
| Scaling | Autoscaling workers | Manual / custom | Manual or autoscaling limited |
| Ecosystem | Beam portability | Rich Flink features | Spark ML integration |
**Rationale:** Dataflow preferred for managed scaling + watermark accuracy; Flink if requiring exotic window operators / custom state; Spark if heavy inline ML libs needed and latency tolerance higher.

### 5. Outline a blueprint for proactive SRE automation that uses burn-rate signals, change risk scores, and predictive saturation models to auto-throttle deployments across 30+ services. Which metrics and algorithms power this?

**Metrics:** Error budget burn (fast/slow windows), latency percentiles, CPU/memory headroom, queue backlog slope, recent deploy frequency, incident count, anomaly score.

**Algorithms:**
- Burn-rate thresholds (multi-window).  
- Time-series forecasting (Prophet) for saturation; if predicted breach < horizon, throttle.  
- Change risk scoring: logistic regression on past deploy metadata (lines changed, service criticality, on-call load).  
- Anomaly detection: Isolation Forest on combined latency/error features.

**Automation:** Cloud Deploy gating: if risk score + burn-rate exceed policy, shift to slower rollout or freeze. Notification with override (break-glass) logged.

### 6. Describe a secure analytical enclave pattern for regulated data (PCI + HIPAA) enabling ephemeral querying via isolated Vertex AI notebook sessions with auditable, hermetic execution.

**Pattern:** Request triggers provisioning of ephemeral Vertex AI Workbench instance inside restricted subnet (no public egress) + VPC SC perimeter. Access via IAP + short-lived IAM binding. Notebook environment built from signed container (Binary Authorization enforced). All storage mount read-only except designated scratch GCS bucket (CMEK). Session logs & executed cells captured to BigQuery audit dataset. Idle timeout (15m) auto-shutdown; secret access broker issues time-bound tokens. Data egress policies enforced by Org Policy & Firewall egress deny except internal APIs.

### 7. Engineer a dynamic cost-to-reliability optimization loop that tunes Cloud Run min instances, GKE node pool autoscaling parameters, and BigQuery slot reservations daily. What signals and control actions are applied?

**Signals:** P95 cold start latency, request burst factor, node pending pods %, BigQuery slot utilization %, query queue wait time, SLO error budget, cost per workload vs target.

**Control Actions:**
- Increase min instances for services breaching latency SLO due to cold starts; decrease where spare > threshold.  
- Adjust GKE node pool min/max; shift workloads to spot/preemptible pools if error budget healthy.  
- Reallocate slot reservations (ingest vs ad-hoc) based on prior day queue delays; purchase/release flex slots.  
- Produce daily diff report + apply via Cloud Scheduler + Cloud Functions automation (with rollback if SLA regression).  

### 8. Provide a hardened pattern for secret lifecycle & rotation across multi-env (dev/staging/prod) using Secret Manager, CMEK, workload identity, and just-in-time ephemeral access.

**Pattern:**
1. Separate projects per env; CMEK keys per env (key ring).  
2. Secrets versioned; rotation Cloud Scheduler → Cloud Run Job creates new version, validates canary retrieval, then promotes label `current`.  
3. Workload Identity (GKE / Cloud Run) grants minimal `secretAccessor` to specific secret only.  
4. Just-in-time ephemeral token service issues short-lived signed JWT enabling secret fetch for limited period.  
5. Access logs stream to BigQuery; anomaly detection on unusual pattern triggers forced rotation (disable old version).  
6. Replicate only non-prod sanitized secrets; production secrets isolated (no shared SAs).  
7. Alert if secret version age > rotation policy or if unreferenced secrets linger > TTL.  

---

## ⚙️ Section 2: Scenario - Answer

**Risk Scoring Modernization Architecture:**

```
Kafka (on-prem) → Datastream / MirrorMaker → Pub/Sub (ingest) ← App Events
       File Drops → GCS Landing → Dataflow File Ingest (batch micro-batch) → Pub/Sub (normalized)
                                     │
                               Streaming Dataflow (feature agg, joins, TTL state)
                                     │               │
                                     │               └→ Vertex AI Feature Store (online)
                                     │
                                     └→ BigQuery Bronze → Silver → Gold (audit lineage)
                                                 │            │
                                      Backfill Dataflow   Risk Score Recompute Job
                                                 │
                                      Vertex AI Prediction (online scoring API)
```

**Key Points:**
- Normalization layer produces canonical event envelope (schema-registry enforced).  
- Streaming aggregation updates incremental risk factors (velocity, recent anomalies).  
- Online scoring service (Cloud Run/GKE) pulls features from Feature Store + low-latency Bigtable (optional) for hot counters; responses cached (sub-second).  
- Replay: Bronze layer immutable + event timestamp partitioning; new scoring logic reprocess via versioned pipeline (Dataflow) writing alternative score column until verified.  
- Governance: Dataplex lineage, policy tags, signed scoring model metadata; reproducibility via model artifact hashing (Binary Authorization).  
- Resilience: Multi-region Pub/Sub topics; Dataflow jobs with drain/failover templates; scoring endpoints behind global LB with weighted region shift.  
- Cost: Tiered storage (Bronze short TTL to Nearline after 14d), selective feature retention, flex slots for burst transform windows.  

---

## 🧩 Section 3: Problem-Solving - Answer

**Automated Credential Compromise Response Pipeline:**

```
Secret Access Logs → Log Sink (Pub/Sub) → Cloud Function (Detector)
                                  │
                             BigQuery (historical baseline)
                                  │
                         Vertex AI Anomaly Model (score)
                                  │ (high score)
                       Cloud Workflows (Response Orchestrator)
          ┌──────────────┬───────────────┬────────────────┬────────────────┐
      Rotate Secret  Invalidate Tokens  Quarantine SA  Forensic Bundle
          │                 │                │               │
  Cloud Run Job → New Version   IAM policy drop  Org Policy tag  Export logs + key history
```

**Detection Logic:**
- Features: access frequency delta, time-of-day deviation, geo/IP novelty, sequential enumeration of secret names, spike in denied attempts.
- If anomaly score > threshold & corroborated by second signal (e.g., IAM unusual SA impersonation), mark incident.

**Response Steps:**
1. Create new secret version; health check canary retrieval.  
2. Update label `current`; disable old version; notify dependent services via Pub/Sub.  
3. Force token revocation: revoke service account external access (disable key / remove binding).  
4. Quarantine: add deny condition IAM binding for compromised principal.  
5. Forensics: snapshot logs, export audit to GCS WORM bucket, hash manifest.  
6. Rollback: If downstream failure rate > threshold, re-enable prior version temporarily (flagged).  

**Safeguards:** Two-person approval (Access Approval) for destructive revocations except in auto-critical severity; backoff & retry on rotation errors; limit rotation bursts with rate controls.  

---
