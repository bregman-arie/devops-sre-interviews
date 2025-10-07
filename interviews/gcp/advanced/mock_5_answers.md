# Google Cloud Platform (GCP) - Advanced Interview Mock #5 - Answer Key

> **Difficulty:** Advanced  
> **Duration:** ~60 minutes  
> **Goal:** Challenge depth in cross-cloud portability, governance automation, real-time ML/analytics fusion, resilience economics, and security.

---

## 🧠 Section 1: Core Questions - Answers

### 1. Design a multi-cloud (GCP + AWS) active/active API layer using Global External HTTP(S) Load Balancer + Cloud Interconnect + AWS GWLB. How do you route, fail, and observe without duplicating business logic?

**Pattern:** Primary ingress = GCP Global LB (anycast) with backends: GCP services (NEG: Cloud Run / GKE) + external NEG referencing AWS ALB via Interconnect (VLAN attachments + Cloud Router). Weighted routing (initial 80/20) + health checks. Optionally reciprocal AWS Route53 latency LB pointing to GCP via CloudFront custom origin for bilateral resilience.

**State Handling:** Stateless JWT or opaque session token referencing Redis Enterprise multi-region cluster; avoid per-cloud session stickiness. Idempotency keys ensure safe retries during partial failover.

**Failover:** Automated weight shift when health check + synthetic transaction probes + error budget burn threshold triggered. Use slow start (gradual ramp) to protect cold region. DNS TTL kept low (≤60s) for AWS entrypoint fallback.

**Observability:** Shared OpenTelemetry collector exporting to Cloud Logging + CloudWatch via OTLP; unified trace IDs propagate cross-cloud. SLO computation centralized in BigQuery (billing + latency + error metrics joined). Unified error taxonomy mapping.

### 2. Propose a blueprint for end-to-end ML model governance (data → features → model artifact → deployment → drift retirement) using Dataplex, Data Catalog, Vertex AI, Artifact Registry, and Binary Authorization.

1. **Data Layer:** Raw → curated zones in Dataplex; assets tagged (sensitivity, retention, domain). Schema + lineage auto-registered in Data Catalog.
2. **Feature Engineering:** Dataflow / Spark-on-Dataproc writes to Vertex AI Feature Store (offline BQ + online); feature definitions version-controlled.
3. **Training:** Vertex AI Pipelines orchestrate training; metadata (hyperparams, dataset snapshot hash) logged to ML Metadata Store.
4. **Artifact Signing:** Trained model container (or model file) stored in Artifact Registry; SBOM + vulnerability scan; cosign sign digest.
5. **Policy Gate:** Binary Authorization attestor requires provenance + fairness metrics + drift baseline present before deploy.
6. **Deployment:** Vertex AI Endpoint / Cloud Run inference container; canary traffic splitting.
7. **Monitoring:** Vertex drift & data skew monitors; threshold breach triggers Pipeline retraining or rollback to previous model version.
8. **Retirement:** Stale models archived (immutable GCS bucket) + revocation of deploy approval attestation.

### 3. Engineer a GCP-native approach to dynamic per-tenant network isolation for thousands of tenants sharing common microservices (avoid VPC explosion). Which primitives enable scalable isolation & policy?

**Approach:** Single Shared VPC (or limited set) + hierarchical firewall policies + service-to-service auth (mTLS + Identity-Aware Proxy / JWT) + per-tenant identity claims. Use **Gateway APIs + Envoy filters** to enforce tenant-level egress policies. For data isolation: row/column level security + CMEK key-per-tenant.

**Primitives:**
- Cloud Armor (tenant-based rate shaping via header)  
- Anthos Service Mesh / mTLS identities (SPIFFE)  
- Firewall policy + network tags for broad segmentation tiers (premium vs standard)  
- IAM Conditions (restrict SA usage, time-bound)  
- VPC SC for sensitive multi-tenant analytics perimeter  
- Tenant context token introspection for fine-grained authz  

### 4. Compare Cloud Run Jobs vs Dataflow vs Batch (GKE Autopilot + Kueue) for large-scale periodic backfills with variable data skew and strict cost ceilings. Provide a decision guide.

| Criteria | Cloud Run Jobs | Dataflow | GKE Batch (Kueue) |
|---------|----------------|----------|-------------------|
| Ops Overhead | Very low | Low (managed) | Medium |
| Parallelism Control | Limited tasks | Dynamic workers | Fine-grained pods |
| Skew Handling | Manual partition logic | Built-in autoscaling + dynamic splitting | Custom schedulers / preemption |
| Cost Optimization | Scale to zero | Streaming + flexRS + preemptibles | Spot / preemptible pools | 
| Stateful / Complex DAG | Limited | Possible (Beam DAG) | Arbitrary (Argo / custom) |
| Data Vol (10+ TB) | Possible but manual chunking | Natural fit | Fit if custom libs needed |
**Guide:** Use Dataflow for large ETL/backfill with skew; Cloud Run Jobs for lightweight, moderate parallel chunk tasks; Batch GKE when custom runtime/sidecars, GPU, or tight pod-level scheduling needed.

### 5. Devise a strategy to continuously minimize tail latency (P99) for a polyglot microservice environment (Go, Java, Python) using eBPF profiling, adaptive concurrency, and autoscaling policies.

**Loop:**
1. eBPF continuous profiling (CPU, off-CPU, syscall) via profiling agent → BigQuery aggregated flamegraph metrics.  
2. Identify top P99 contributors → emit optimization hints (lock contention, GC pauses).  
3. Adaptive Concurrency Control (token bucket / gradient controller) reduces in-flight requests when queue latency rising.  
4. Autoscaling (HPA / Cloud Run concurrency) uses composite metric: weighted (CPU, latency, queue depth).  
5. Canary performance guard: new build deployed with synthetic latency regression test; rollback if P99 > baseline + threshold.  
6. Periodic regression ML model forecasts rising tail; preemptively add capacity.  

### 6. Explain how to implement real-time least‑privilege reduction (“permission decay”) for service accounts based on access graph usage trends. Which data sources & automation loops?

**Data Sources:** Cloud Audit Logs (Admin + Data Access), IAM Recommender export, Access Approval logs, policy metadata.

**Loop:**
1. Build access graph (edges: SA → permission).  
2. Compute usage frequency + recency (sliding window).  
3. Score risk (high privilege + low usage).  
4. Propose removal PR (Config Sync repo) → require human approval for critical roles.  
5. After grace period, apply; monitor error spikes; auto-rollback if anomaly.  
6. Continuous weekly cycle; emergency escalate if new unused high-risk privilege appears.  

### 7. Architect a frictionless cross-region data portability layer for BigQuery + GCS enabling selective lawful export & erasure requests (GDPR) with auditable irreversibility.

**Flow:** Data subject ID index (tokenized) maps to dataset partitions. Export pipeline (Dataflow) queries only partitions referencing token → generates export bundle (parquet + manifest hash) → signs + stores ephemeral in secure GCS link (time-bound URL). Erasure = soft tombstone + background redaction in BigQuery (MERGE updating status) + GCS object deletion (versioned, locked retention exempt). Audit log records hash; append-only ledger (Cloud Spanner) stores irreversible proof of action. Tokenization ensures minimal direct PII scanning cost.

### 8. Provide a holistic cost + carbon optimization framework factoring workload SLO tiers, CI pipeline cadence, data retention, and hardware accelerator scheduling (TPUs/GPUs).

**Framework Layers:**
- **Classification:** Tag workloads with SLO tier (Critical, Standard, Best-Effort) + carbon flexibility score.  
- **Scheduler Policies:** Critical pinned to lowest latency region; best-effort ML training deferred to low-carbon window.  
- **Accelerator Pooling:** Central queue for GPU/TPU jobs; preemptible accelerators for non-critical; right-size epochs via early stopping metrics.  
- **CI Cadence Optimization:** Adaptive pipeline frequency (skip builds if code diff doc-only or below risk threshold).  
- **Retention:** Tier logs/data (Hot → Nearline → Coldline → Archive) via lifecycle rules; enforce TTL budgets.  
- **Control Loop Metrics:** Carbon intensity API, slot utilization, accelerator idle %, cost per SLO tier, build success delta, error budget status.  
- **Automation:** Daily optimization job adjusts reservations, min instances, accelerator queue weights; generates report with savings & SLO impact.  

---

## ⚙️ Section 2: Scenario - Answer

**Unified Event + Analytics + ML Platform:**

```
Sources → Pub/Sub (core events) & Datastream (CDC) → Normalizer (Cloud Run / Dataflow) → Hot Stream Processor (Dataflow streaming)
   │                                                           │
   │                                                           ├→ Feature Store (online + offline)
   │                                                           ├→ BigQuery Hot (incremental micro-partitions)
   │                                                           └→ Cache Invalidation Bus (Pub/Sub)
Batch Backfill (BigQuery historic) → Reconciliation Job → Gold KPIs (Materialized Views)
Model Training Pipelines (Vertex AI) ← Offline Features (BQ) / Drift Signals
Active/Active Scoring (Cloud Run regional) → Risk Scores Topic → Downstream APIs
```

**Key Aspects:**
- **Freshness:** Streaming micro-batches produce provisional aggregates; nightly reconciliation merges corrections (idempotent MERGE in BigQuery).  
- **Dual Path Reconciliation:** Hot path writes delta, warm job recalculates windowed metrics, cold batch ensures historical accuracy.  
- **Feature Lifecycle:** Real-time features (velocity, recency) + offline features combined at inference time; drift monitors feed retrain triggers.  
- **Chargeback:** Labels (project, team, tenant) enforced via deployment admission; billing export aggregated to BigQuery; Looker dashboards allocate cost per tenant usage events.  
- **Replay:** Bronze/raw events retained 90 days (partitioned); versioned transform DAG runs deterministic replays (content-addressed code artifact).  
- **Governance:** Dataplex domain zoning; policy tags; Binary Authorization for scoring images; lineage edges (event → feature → model → score).  
- **Carbon-Aware:** Non-critical retrain + reconciliation scheduled when carbon intensity low; dynamic slot scaling.  
- **Cost Reduction:** Materialized incremental views; compaction of small files; autoscaling Dataflow & Cloud Run; flex slots for spike windows.  

---

## 🧩 Section 3: Problem-Solving - Answer

**SLO Guard Controller Architecture:**

```
Cloud Monitoring (SLO, burn) + Cloud Deploy (revisions) + Audit Logs + Cost Export → BigQuery (metrics warehouse)
                                                            │
                                                     Feature Builder (Cloud Run)
                                                            │
                                                    Vertex AI Model (risk score)
                                                            │
                                                    Decision Engine (Cloud Functions)
                                                            │
                                      Actions: Pause Deploy | Slow Roll | Budget Loan | Rollback
```

**Decision Rules:**
1. If fast burn rate > threshold & new deploy in last 15m → rollback revision.  
2. If slow burn trending high but no recent deploy → allocate temporary budget loan (tracked) & open optimization ticket.  
3. If risk score (deploy complexity × prior incident density × saturation forecast) > policy AND error budget < buffer → block rollout; require manual override.  
4. Budget loans expire automatically (auto-recovery) and reduce next sprint’s reliability buffer (visible in planning dashboard).  

**Rollback Safeguards:** Multi-step confirmation for cross-region rollback; store pre-action snapshot (traffic weights, revision map).  

**Outputs:** Daily report: burn deltas, blocked deploys, borrowed budget ledger, projected SLO debt cost.  

---
