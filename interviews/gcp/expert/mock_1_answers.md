# Google Cloud Platform (GCP) - Expert Interview Mock #1 - Answer Key

> **Difficulty:** Expert  
> **Duration:** ~75 minutes  
> **Goal:** Evaluate mastery of planet-scale architecture, extreme reliability engineering, advanced security, and governance.

---

## 🧠 Section 1: Core Questions - Answers

### 1. Design a multi-continent trading & settlement backbone leveraging Spanner + regional low-latency caches while guaranteeing monotonic account balance reads globally. Detail write path, read path, and failure semantics.

**Write Path:**
1. API ingress (Global HTTPS LB + Cloud Armor) routes to nearest region.  
2. Transaction service validates idempotency key, reserves sequence via Spanner monotonically increasing ledger table (commit timestamp ordering).  
3. Writes batched (mutations) → Spanner (multi-region config, e.g., `nam-eur-asia1`) achieving external consistency.  
4. Change Streams emit mutation events to Pub/Sub for cache invalidation + downstream analytics.  

**Read Path:**
- Regional in-memory cache (Redis Enterprise / Memorystore) stores derived balances (materialized from last N ledger deltas). Reads attach a **read-timestamp ≥ last seen** ensuring monotonicity: client passes `X-Last-Read-Timestamp`; service performs a Spanner stale read at `max(client_ts, now - staleness_window)` or fresh strong read if needed; updates client token.  
- Cache updated asynchronously; on cache miss, service issues Spanner strong read.  

**Failure Semantics:**
- Regional cache loss → fallback to strong Spanner reads (higher latency).  
- Spanner region unavailability masked by multi-region quorum (continued writes).  
- Client monotonic guarantee preserved via timestamp token; if token > current bound (due to staleness), force strong read.  
- Partition detection triggers temporary suspension of speculative batching to reduce conflict retries.  

### 2. Propose a strategy to continuously verify supply chain integrity (build → provenance → deploy → runtime) using Binary Authorization, Artifact Analysis, Cloud Deploy, and runtime drift detection (Config Sync / Policy Controller). How do you enforce revocation?

**Chain:** Source (signed commits) → Cloud Build (provenance attestations) → Artifact Registry (scanned, SBOM) → Binary Authorization checks (attestors: vulnerability_scan_passed, provenance_verified, policy_conformant) → Cloud Deploy progressive rollout → Config Sync enforces runtime manifest baseline → Policy Controller forbids unreferenced images.  
**Continuous Verification:** Scheduled re-scan of active digests; compare declared SBOM vs live processes (eBPF runtime scan).  
**Revocation:** Invalidate attestor public key or push revocation policy; Binary Authorization denies future rollouts; Config Sync garbage collects non-compliant workloads; emergency Cloud Deploy rollback pipeline to last good digest.  

### 3. Engineer a unified, low-latency global pub/sub mesh bridging on-prem high-frequency market feeds with GCP, optimizing for fairness & bounded out-of-order delivery (<20ms skew). Include edge ingestion, compression, and ordering recovery.

**Edge Ingestion:** Co-located PoPs run lightweight collectors (UDP multicast capture) → compress (LZ4) + frame messages → TLS tunnel to nearest Cloud Run/gRPC ingest.  
**Transport:** Regional Pub/Sub topics per asset class with ordering keys (instrument_id). Cross-region replication via Dataflow pipeline that preserves publish timestamp ordering (buffer + watermark).  
**Fairness:** Weighted fair queueing at edge; apply sequence tokens per instrument; backlog smoothing via adaptive batch size (latency target).  
**Ordering Recovery:** Consumer library detects gap (seq_n+2 arrives) → temporary hold buffer (≤20ms) waiting for missing; if missing persists, emit synthetic fill event flagged for reconciliation.  
**Monitoring:** Skew histogram, gap frequency, per-instrument lag; anomalies publish to ops channel.  

### 4. Explain an approach to implement adaptive SLOs with dynamic error budget reallocation across 40+ microservices (shared composite product SLO). How do you prevent starvation and budget gaming?

**Composite:** Product availability derived from dependency graph (AND probability model). Each service has base SLO + adjustable budget multiplier.  
**Dynamic Allocation:** Hourly recompute contribution to overall burn (dependency sensitivity via error propagation analysis). Services with low recent burn donate portion of unused budget to constrained critical path services.  
**Anti-Gaming:** Minimum floor (e.g., 70% of base budget retained); anomaly detection on sudden traffic shaping or artificial error suppression. Independent synthetic checks ensure reported metrics reflect user experience. Governance committee reviews monthly drift.  
**Tooling:** Custom controller reading Monitoring APIs; writes updated budgets to config repo → triggers alert recalibration.  

### 5. Architect a strict data sovereignty enforcement model spanning 12 jurisdictions with mixed OLTP (Spanner / Cloud SQL) + analytical BigQuery federations. Include policy tagging, encryption domains, and cross-border anonymization workflow.

**Model:**
- Per-jurisdiction project grouping (folder) + Org Policy restricting resource locations.  
- KMS key rings per jurisdiction; data assets tagged with policy tags (region_code, sensitivity).  
- Spanner instances regional or multi-region limited to jurisdiction; Cloud SQL localized.  
- BigQuery: raw regional datasets; anonymization pipeline (Dataflow + DLP) outputs pseudonymized set; only DP aggregates exported to global dataset.  
- Access mediated by Data Catalog policy tags + IAM Conditions (device + region).  
- Cross-border job requests require Access Approval.  

### 6. Design a chaos & resilience verification framework that injects multi-layer faults (network partitions, Spanner leader move, Pub/Sub backlog surge, key revocation) and automatically scores blast radius vs containment objectives.

**Framework:**
- Scenario catalog (YAML) with fault primitives + expected KPIs (latency delta < X, error rate < Y).  
- Orchestrator (Cloud Workflows + Cloud Functions) triggers faults (e.g., firewall rule to simulate partition, `gcloud spanner databases ddl update` to force leader move, flood publisher for backlog, disable KMS key version).  
- Observability harness collects time-window metrics pre/during/post.  
- Scoring engine (BigQuery ML) computes deviation; flags regression if containment SLO exceeded.  
- Results stored with lineage; automatic Jira ticket if score < threshold.  

### 7. Define a cost governance program for petabyte-scale ML + streaming analytics blending committed use discounts, dynamic slot allocation, preemptible fleets, and workload-aware carbon optimization. Provide control loops.

**Controls:**
- **Baseline Commit:** Purchase moderate committed use discounts (Spanner, GCE, BigQuery slots) sized to P50 usage.  
- **Elastic Layer:** Flex slots + preemptible GCE for burst training; autoscaler monitors queue depth & model urgency.  
- **Carbon Scheduling:** Non-urgent training deferred to low-carbon intensity regions (Carbon Intensity API) if added latency < SLA.  
- **Feedback Loops:**
  - Slot Utilization Loop: If avg utilization < 60% for 24h → reduce reservation; > 85% → acquire flex.  
  - Preemptible Loss Loop: Track interruption rate; if > threshold shift fraction to regular VMs.  
  - Model Retrain Loop: Dynamic cadence based on feature drift metric.  
  - Carbon Loop: Re-route queued batch jobs when carbon score difference > defined margin & latency slack ≥ threshold.  

### 8. Construct a defense-in-depth strategy against insider data exfiltration involving BigQuery, GCS, Secret Manager, and Vertex AI notebooks in a regulated environment (PCI + GDPR).

**Controls:**
1. VPC Service Controls perimeters across analytic + secret projects.  
2. Column + row-level security, policy tags; deny SELECT * for sensitive datasets via custom query analyzer (Dataform/Proxy).  
3. Just-in-time elevation workflow with Access Approval audit.  
4. Notebook Execution Policy: Vertex AI notebooks ephemeral; idle auto-shutdown; no outbound internet (private endpoints only).  
5. DLP scanning & anomaly detection on large query result exports; export size quotas.  
6. CMEK + key usage anomaly alerts; break-glass keys separate audit log.  
7. Secret Manager least-privilege + alert on unusual enumerate patterns.  
8. Honeytoken fake datasets & credentials to trip exfiltration early.  

---

## ⚙️ Section 2: Scenario - Answer

**Global Payments Platform Architecture:**

```
Clients → Global LB (HTTP/2, mTLS) → Regional API (Cloud Run/GKE) → Spanner (multi-region) ← Change Streams → Risk Stream (Pub/Sub → Dataflow → Vertex AI Real-time) 
                                      |                           
                                      └─ Ledger Cache (Redis Enterprise regional)  

FX Rates Feed → Pub/Sub (regional) → Spanner reference tables (TTL versioned)
```

**Consistency Model:** Spanner external consistency ensures single global ledger sequence. Derived balances cached with timestamp tokens ensuring monotonic reads.  
**Replication:** Multi-region Spanner (3x read replicas) + ledger change streams → regional caches + BigQuery near-real-time ingestion for reconciliation.  
**Risk Scoring (<50ms):** Lightweight feature extraction (recent txn count, geo velocity) from Redis + static model in Vertex AI Prediction (or in-process TF Lite). Pre-warmed model instances.  
**Failure Domains:** Zonal failure masked by Spanner + multi-zone service; regional failure triggers traffic shift (LB health) + warm standby expansion.  
**Feature Flags:** Cloud Deploy + Config service controlling risk model version & FX conversion formulas; canary region first.  
**Carbon Optimization:** Non-critical analytic batch (reconciliation, model retrain) scheduled in region with lowest real-time carbon index if latency slack available.  
**Compliance Logging:** Immutable append bucket (Object Lock) for ledger snapshots + Cloud Audit Logs export (WORM). Hash-chained daily ledger digests stored in external archive for tamper detection.  
**Scalability:** Horizontal Cloud Run services; Spanner compute autoscaling; Dataflow streaming shards keyed by account region.  
**RPO/RTO Goals:** Spanner meets RPO ≤5s; global LB + warm secondary region meets RTO < 2m (automated runbook + health gating).  

---

## 🧩 Section 3: Problem-Solving - Answer

**Hierarchical Failover Controller:**

```
Signals: Metrics (latency, error rate, queue depth), Spanner commit latency, Pub/Sub backlog, Carbon score, Saturation forecasts (Vertex AI model)
Controller Tiers: Edge → Regional → Global
```

**Flow:**
1. **Predict:** ML model (gradient boosting) consumes last 10m features (P95 latency trend, CPU headroom, backlog slope) to predict saturation probability > threshold.  
2. **Brownout Stage:** Disable / degrade non-critical endpoints (e.g., detailed history) via config flag; reduce model inference enrichment (light mode).  
3. **Partial Promotion:** Scale standby region services; start replicating hot cache keys; pre-warm inference containers.  
4. **Failover Trigger:** If primary region health composite < policy (multi-metric: error rate, commit latency, packet loss) for N windows, shift LB weighting gradually (25% → 50% → 100%).  
5. **Invariant Validation:** Post-shift: verify ledger sequence gap-free (Spanner change stream continuity), double-debit detector (idempotency keys uniqueness), risk score latency within SLO.  
6. **Rollback Criteria:** If invariants fail or latency regression > X%, revert traffic weighting; issue incident escalation.  
7. **Learning Loop:** Store feature/outcome pair back into training dataset for model recalibration.  

**Automation Components:** Cloud Monitoring metrics → Pub/Sub → Cloud Function (feature builder) → Vertex AI prediction endpoint → Cloud Workflows orchestrating gating actions (Config changes, Cloud Deploy rollout, LB weight update).  
**Blast Radius Scoring:** Compare error / latency deltas vs baseline; store in BigQuery; periodic report ranks scenario resilience.  

---
