# Google Cloud Platform (GCP) - Advanced Interview Mock #6 - Answer Key

> **Difficulty:** Advanced  
> **Focus:** Multi-region data/ML platform modernization, governance automation, cost + performance control loops.

---

## 🧠 Section 1: Core Questions - Answers

### 1. Design a multi-region transactional + analytical dual-store pattern using Spanner (OLTP) and BigQuery (OLAP) with <5s replication lag for derived aggregates. Detail change capture, transformation, idempotency, schema evolution, and consistency guarantees to consumers.

**Pattern:** Spanner is source of truth; Change Streams → Dataflow streaming pipeline → BigQuery raw change table → transformation (exactly-once) → curated aggregate tables.  
**Lag Target:** Parallel partitions by table + commit timestamp watermark; Dataflow uses streaming engine + autoscaling to maintain <5s P95 end-to-end ingest.  
**Idempotency:** Each change event includes Spanner commit timestamp + primary key; BigQuery raw table uses MERGE with de-dup on (primary_key, commit_ts). Aggregations built incrementally using windowed upserts (BigQuery SQL MERGE or Dataform).  
**Schema Evolution:** Schema registry (Cloud Source repo) + automated diff script; additive columns propagate from Spanner DDL events; pipeline interprets unknown columns as JSON payload until formalized. Backfill job (batch Dataflow) fills historical values.  
**Consumer Consistency:** Publish a “synchronized watermark” table storing last fully processed commit_ts per table; downstream dashboards only query rows where commit_ts ≤ watermark.  
**Failure Handling:** Dead-letter for transformation errors (invalid enum etc.) with periodic replay after rule update.  

### 2. Engineer a secure cross-environment (dev→prod) artifact promotion pipeline enforcing provenance, vulnerability gating, policy-as-code, and staged rollout. Use: Cloud Build, Artifact Registry, Binary Authorization, Cloud Deploy, and Config Sync.

**Flow:**  
1. Dev push → Cloud Build: build container + SBOM → sign provenance (SLSA attestation) + vulnerability scan.  
2. Artifact Registry (dev repo) stores image digest; promotion request triggers policy check (OPA/Rego via Cloud Build step).  
3. Promotion pipeline copies digest to staging/prod repos only if: vulnerabilities ≤ severity threshold, provenance attestation valid, license policy satisfied.  
4. Binary Authorization policy in prod requires attestors: (provenance_ok, vuln_scan_passed, policy_conformant).  
5. Cloud Deploy pipeline stages: canary (5%), shadow, 25%, 100% with metric & error budget gates; automatic abort on regression.  
6. Config Sync enforces deployment manifests & disallows drift (admission webhook denies out-of-band updates).  
7. Rollback: On gate fail Cloud Deploy reverts to previous digest; Binary Auth denies further rollout of failed digest.  

### 3. Propose an approach for adaptive data retention & tiering (hot vs warm vs cold vs purge) across GCS, BigQuery, Bigtable, and Archive Storage balancing SLOs, compliance (7-year finance), and cost/carbon goals.

**Classification:** Tag datasets with access frequency, compliance class, latency SLO.  
**Tiers:**  
- Hot: Bigtable (sub-50ms), BigQuery streaming tables (recent 7–30d).  
- Warm: BigQuery partitioned (monthly partitions older than 30d, cluster for scan reduction).  
- Cold: GCS Nearline / Coldline parquet; external tables for ad-hoc retrieval.  
- Archive: GCS Archive (immutable) + hash manifest for retention compliance (WORM).  
**Automation:** Scheduler job reads usage metrics (BigQuery `INFORMATION_SCHEMA.JOBS`, GCS object access logs) → moves older partitions to externalized GCS + updates metadata view.  
**Purging:** At retention expiry generate cryptographic erase manifest; store audit record (BigQuery log table) + verify no orphan indexes remain.  
**Carbon:** Defer non-urgent tier migrations to low-carbon windows; aggregator groups moves to reduce churn.  

### 4. Design a global secrets + encryption strategy combining CMEK, Cloud KMS, External Key Manager, and Secret Manager with per-service blast radius minimization and automated key rotation + revocation drills.

**Key Hierarchy:** Root external HSM (EKM) for master; per-domain KMS key rings per region; per-service crypto keys (envelope) with rotation ≤90d.  
**Secret Storage:** Secret Manager with versioned secrets; access via service-specific service accounts (Workload Identity).  
**Blast Radius:** Scoped keys (one service logical boundary) + key usage alerting (Cloud Logging metric).  
**Rotation:** Scheduler triggers key version creation; re-encryption job rotates data; old version disabled after verification window.  
**Revocation Drill:** Quarterly simulate compromise: disable key version; verify fallback path (cached decrypt fails gracefully).  
**Access Governance:** IAM Conditions (device, time) for human break-glass; Access Approval for highly sensitive key usage.  

### 5. Architect a near-real-time ML feature store (Vertex AI Feature Store + Bigtable + Pub/Sub) enabling online <20ms fetch and offline parity for training reproducibility. Include update fan-out, skew detection, and point-in-time correctness validation.

**Ingestion:** Source events → Pub/Sub topics per domain → Dataflow enrich & compute derived features → (a) write to Bigtable serving (row key entity_id#feature_group), (b) publish to Feature Store ingestion API (offline store in BigQuery).  
**Latency:** Bigtable single-row read P95 <10ms; client library caches frequent entity groups.  
**Parity:** Snapshotted offline dataset (time travel using ingestion timestamp) used for training; point-in-time join service prevents feature leakage by using commit watermark.  
**Skew Detection:** Compare online vs offline feature distributions (KS test) daily; anomaly events stored.  
**Validation:** Feature definitions versioned; pipeline asserts schema & transformation hash stable; mismatches trigger quarantine.  
**Backfill:** Historical recompute batch writes both layers with same event_time; idempotent via (entity, feature, event_time) uniqueness.  

### 6. Design a compliance logging & lineage platform integrating Cloud Audit Logs, Data Catalog, Dataplex, and BigQuery to enable regulated queries: “Show all principals who accessed PII field X in last 30d across region Y datasets.”

**Ingestion:** Export Data Access Audit Logs (BigQuery, GCS) to central BigQuery sink.  
**Metadata Graph:** Data Catalog + Dataplex store asset metadata & policy tags (PII classification). ETL job builds mapping: field → dataset → region.  
**Query Engine:** Materialized view joining (audit_log.principal, resource_name, timestamp) with field classification expansion. Access to table with PII field X implies potential field exposure; for fine-grained logs (row/column ACL) include filter context.  
**Optimization:** Partition logs by date + cluster by principal to keep <5s query.  
**UI:** Portal form compiles parameterized query hitting curated view.  
**Retention:** Logs WORM stored 1y hot + moved to GCS parquet for long-term.  

### 7. Explain a strategy for proactive cost guardrails that dynamically throttle or reschedule workloads (Dataflow, Vertex AI training, BigQuery queries) when hitting budget anomaly thresholds while preserving critical SLO workloads.

**Detection:** Streaming BigQuery ML anomaly model on real-time billing export (project,label).  
**Classification:** Workloads labeled with criticality tier (T1 must run, T2 deferrable, T3 opportunistic).  
**Actions:**  
- If anomaly in T3: pause Dataflow backfills (drain) & defer training jobs (queue).  
- If anomaly in T2: reduce BigQuery concurrency slots; move batch to low-carbon window.  
- Never throttle T1 unless safety override (hard quota).  
**Mechanism:** Orchestrator writes quota adjustments (Reservation API update, Vertex training scheduling service).  
**Feedback:** Evaluate action effectiveness (cost delta) and revert if minimal impact.  

### 8. Engineer a low-latency inter-region API mesh using Global Load Balancing + Traffic Director + gRPC with adaptive concurrency, circuit breaking, and eBPF-driven latency instrumentation to reduce P99 tail by 25%.

**Topology:** Global External HTTPS LB → regional gateways (Envoy/Traffic Director managed) → service mesh (sidecars / proxyless gRPC).  
**Latency Reduction Tactics:**  
- Adaptive concurrency (token bucket sized by observed queue time).  
- Per-endpoint circuit breaking (pending request & connection limits).  
- Cross-region failover with jittered retry budgets (hedging for P95 > threshold).  
- eBPF (Cilium / BCC) capture of TCP RTT & syscall latency feeding metrics to adjust concurrency window.  
- Prioritize warm connection pools; gRPC keepalive tuned (ping interval < idle timeout).  
**Observability:** Distributed tracing with trace sampling boost when latency anomaly detected; correlation with network layer eBPF spans.  
**Result:** Controlled concurrency & faster outlier detection shrinks tail latency variance.  

---

## ⚙️ Section 2: Scenario - Answer

**Target Architecture (High-Level):**  
Spanner (multi-region) central OLTP; Change Streams → Dataflow streaming unify replication to BigQuery (curated + raw) and Pub/Sub feature topics. Feature pipeline writes Bigtable serving + Vertex Feature Store offline. Secrets & keys under centralized Crypto Controller (rotation orchestrator). Cost & carbon controller orchestrates BigQuery reservations & training schedule shift. Compliance mesh indexes lineage + audit.  

**Migration (Cloud SQL → Spanner):** Parallel dual-write phase + Dataflow backfill. Read shadow & hash diff until convergence. Cutover via feature flag after replication lag stable <2s.  

**Ingestion Layers:**  
API events → Pub/Sub (ordered keys) → Dataflow: branch to (a) OLTP write service (idempotent), (b) feature compute, (c) analytics raw ingest. Batch historical import uses GCS staged parquet processed by Dataflow (backfill mode).  

**Feature Freshness <10s:** Streaming features computed within Dataflow (sliding windows) → Bigtable write (<200ms) + offline append (BigQuery) simultaneously. Watermark service publishes synchronization pointer for consistent training snapshots.  

**Analytics Lag <30s:** Streaming write API to BigQuery for near real-time ingestion; micro-batch rollups every 30s via scheduled materialized views.  

**Secrets & Crypto Control Plane:** Key Orchestrator (Cloud Run) enumerates keys, rotates per schedule, validates decrypt tests, updates secret references; publishes rotation events consumed by services for hot reload. Compromise drill automation disables key version & triggers chaos test.  

**Cost & Carbon Feedback:** Reservation manager monitors slot utilization; shrinks or grows flex slots; schedules non-critical ML training in low-carbon region/time window (external carbon API). GPU/slot contention predictor (ARIMA + queue depth) triggers preemptive scaling or deferral.  

**Governance & Lineage Mesh:** Dataplex tasks register datasets; Data Catalog tagging pipeline associates sensitivity + owner; lineage graph built (Dataflow instrumentation) stored in BigQuery graph table; portal queries via parameterized views.  

**Operational Resilience:** Chaos suite injects: Spanner region failover, Dataflow worker loss, Bigtable hotspotting. Controller validates SLO invariants (feature freshness, replication lag). Multi-region failover runbooks codified as workflows.  

**KPIs:** Replication lag P95, feature freshness P95, cost per processed event, carbon-adjusted compute utilization, secret rotation SLA adherence.  

---

## 🧩 Section 3: Problem-Solving - Answer

**Adaptive Data & Compute Orchestrator**

**Signals:** BigQuery slot utilization, reservation commitments, pending job queue length, Dataflow backlog growth rate, Vertex AI training job queue, GPU occupancy, feature store update lag, carbon intensity forecast, cost anomaly score.  

**Prediction Model:** Sliding-window features (last 5 intervals) + exogenous calendar (hour of day, day-of-week), queue depth derivative, backlog acceleration. Model: Gradient Boosted Trees (fast inference) forecasting contention probability for next 5 / 10 minutes.  

**Control Loop Stages:**  
1. Collect & Normalize (every 1m)  
2. Predict contention probabilities  
3. Optimize scheduling (solve: minimize Cost + Penalty(SLO risk) + λ*Carbon while constraints: critical SLO freshness, max deferral window)  
4. Apply actions (adjust BigQuery slot reservations, pause / slow Dataflow non-critical pipelines, queue Vertex training, throttle feature updates if lag < slack budget)  
5. Monitor effect vs expected (closed-loop)  
6. Log decision (inputs, predicted risk, action set, expected savings) to BigQuery (immutable audit)  
7. Rollback if deviation from target > threshold (e.g., feature lag overshoot)  

**Decision Policy:** Priority tiers: Tier1 (latency-critical / regulatory), Tier2 (batch analytics near-term), Tier3 (exploratory / training). Linear programming chooses deferrals from lowest priority upward until predicted contention below target utilization (e.g., 75%).  

**Safeguards:** Hard floors for BigQuery slots & GPU capacity for Tier1; max consecutive deferral count per job; carbon optimization only applied if predicted SLO impact < small epsilon.  

**Explainability:** Store feature vector & SHAP explanation per decision; dashboard visualizes “top contributors to deferral.”  

**Rollback Mechanics:** If post-action metrics diverge >2σ from forecast after 2 intervals, revert slot allocations & resume paused jobs sequentially (avoid thundering herd).  

**Outcome:** Smoothed resource contention, lowered tail latency impact, measurable cost & carbon improvements with auditable governance layer.  

---
