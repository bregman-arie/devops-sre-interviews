````markdown
# Google Cloud Platform (GCP) - Intermediate Interview Mock #2 - Answer Key

> **Difficulty:** Intermediate  
> **Duration:** ~45 minutes  
> **Goal:** Evaluate ability to design resilient, secure, cost-aware GCP workloads involving networking, data, CI/CD, and observability.

---

## 🧠 Section 1: Core Questions - Answers

### 1. Compare the different Cloud Load Balancing tiers (Standard vs Premium) and when to choose one over the other.
**Answer:**
| Aspect | Premium Tier (Global) | Standard Tier (Regional) |
|--------|-----------------------|---------------------------|
| Scope | Global anycast LB | Regional (per region) |
| Latency | Uses Google's backbone | Public internet routing |
| Failover | Cross-region | None (regional only) |
| Use Cases | Global SaaS, low latency | Dev/test, cost-sensitive regional apps |
| Features | Global routing, advanced CDN integration | Basic load balancing |
| Pricing | Higher egress but optimized path | Lower base cost |
Choose Premium for multi-region HA, global users, better latency. Use Standard for internal tools, regional-only workloads, staging environments.

### 2. How does Cloud NAT differ from using external IPs on instances? What limits or scaling considerations exist?
**Answer:** Cloud NAT provides outbound internet access for private instances without external IPs. Benefits: improved security (no exposed IP), centralized logging, scaling managed. Considerations:
- NAT IP pool limits concurrent connections per IP (scale by adding IPs)
- Idle timeout (default 2 min TCP; can tune)
- Port exhaustion risk under high ephemeral outbound bursts
- Regional construct; configure per region
- Logs via VPC Flow Logs + NAT logging (for auditing)
Use external IPs only if inbound access required or very high NAT port pressure per single VM.

### 3. Explain Artifact Registry vs Container Registry. Why migrate? What security and governance improvements does it enable?
**Answer:** Artifact Registry (AR) is next-gen, multi-format (Docker, Maven, npm), regional + fine-grained IAM, VPC-SC supported, repository-level separation. Container Registry (GCR) is legacy (global buckets, less granular). Migrate to AR for:
- Per-repo IAM + conditional policies
- CMEK encryption
- Private endpoints + VPC SC
- Vulnerability scanning integration
- Reduced blast radius (scoped repos) and lifecycle policies.

### 4. Describe an end-to-end CI/CD pipeline on GCP for a containerized microservice (build, security scanning, deployment, verification).
**Answer:**
1. Source: Cloud Source Repos / GitHub → trigger Cloud Build.
2. Build: Cloud Build steps (lint, unit tests, build image, SBOM generation).
3. Scan: Container Analysis / Artifact Registry vulnerability scan + policy check (Binary Authorization gate).
4. Store: Push signed image (cosign) to Artifact Registry.
5. Deploy: Cloud Deploy or GitOps (Config Sync / Argo) to GKE using progressive rollouts (canary 5% → 25% → 100%).
6. Verification: Cloud Monitoring SLO / metrics check + error budget guard; automated rollback if latency/error threshold exceeded.
7. Observability: Structured logs, trace instrumentation (OpenTelemetry), build provenance (SLSA level improvement).
8. Policy: Enforce only signed images via Binary Authorization.

### 5. How would you secure inter-service communication for a multi-service GKE deployment (mTLS, identity, policies)?
**Answer:**
- Service Mesh (Anthos Service Mesh / Istio) for automatic mTLS (certificate rotation via Citadel)
- Workload Identity for mapping KSA → GSA (no static keys)
- Peer + request auth policies (JWT claims / SPIFFE IDs)
- Network Policies (Calico) for L3/L4 segmentation
- Authorization: RBAC + mesh authorization policies (ALLOW least privileges)
- Egress control: egress gateways + DNS policy
- Secret management: Secret Manager CSI driver with KMS-backed encryption.

### 6. Compare Cloud SQL read replicas, cross-region replicas, and Cloud Spanner for multi-region reads. Trade-offs?
**Answer:**
| Feature | Cloud SQL Read Replica | Cross-Region Replica | Cloud Spanner |
|---------|-----------------------|----------------------|---------------|
| Consistency | Async eventually | Async higher latency | Strong (global) or bounded staleness |
| Failover | Manual / HA primary only | Manual promotion | Automatic leader election |
| Scale | Limited vertical & read scale | Adds geo latency reads | Horizontal partitioning |
| Use Case | Local read scaling | DR + regional reads | Global OLTP, high concurrency |
| Complexity | Low | Moderate | High (schema + design) |
Choose read replicas for intra-region scale; cross-region for DR + limited geo reads; Spanner when needing global strong consistency & horizontal scale.

### 7. What strategies exist to reduce BigQuery cost while sustaining performance (storage + query optimizations)?
**Answer:**
- Partition tables (ingestion/time) + clustering on high-selectivity columns
- Use column pruning (SELECT needed columns only)
- Materialized views or precomputed aggregates
- BI Engine for dashboard acceleration
- Table decorators (@) for point-in-time queries
- Set query bytes limit; monitor with INFORMATION_SCHEMA.JOBS
- Storage: tier cold data to long-term reduced cost ( >90 days )
- Use streaming only when needed; batch loads cheaper
- Use `EXPLAIN` / dry runs for planning
- Denormalize carefully; avoid repeated nested blow-ups.

### 8. How do you design an observability stack on GCP that supports SLO-based alerting and cost attribution?
**Answer:**
Components:
- Metrics: Cloud Monitoring custom metrics (latency distributions, saturation), export to BigQuery for cost attribution join.
- Logs: Cloud Logging with log-based metrics + retention tiers (hot vs archive). Use sinks to BigQuery (aggregated) & GCS (raw).
- Tracing: Cloud Trace with sampling tuned (e.g., 1% baseline + tail-based for high latency).
- Profiling: Cloud Profiler for CPU/memory hotspots.
- Error Reporting for unhandled exceptions.
- SLOs: Define latency + availability SLOs (burn rate alerts 2h/6h windows). Use error budget burn multi-window alerts.
- Cost Attribution: Use labels + billing export to BigQuery; join with custom usage metrics to compute per-request cost.
- Dashboards: Golden Signals + error budget trend + cost per SLI unit.
- Governance: IAM boundaries & VPC SC for telemetry projects.

---

## ⚙️ Section 2: Scenario - Answer

**Goal Alignment:** Improve EU latency, add rate limiting, protect backend, safe deploys, implement zero-trust identity.

### Plan Overview
1. Multi-region rollout (add `europe-west1` GKE regional cluster)
2. Global external HTTPS LB (Premium) with traffic steering using latency-based + geo policy
3. mTLS + identity (Anthos Service Mesh + Workload Identity)
4. Rate limiting + abuse protection (Cloud Armor + Envoy local + Redis token bucket)
5. Progressive delivery with automated verification
6. Observability + error budget guardrails

### Detailed Steps
- Networking: Use global external Application Load Balancer → backend services referencing NEG (GKE Ingress ASM). Enable Cloud CDN for static & caching idempotent GETs.
- Latency: Warm edge caches, configure outlier ejection & connection pooling, enable HTTP/2 & gzip/brotli.
- Rate Limiting: Cloud Armor per-client rules (customer API key/IP/service account). Fine-grained dynamic limits via Envoy filter + Redis (Memorystore) storing counters (sliding window or leaky bucket).
- Identity & Zero-Trust: Mesh-issued SPIFFE IDs + policy: service A → service B only with valid JWT claim (audience scoping). Disable plaintext pod-to-pod.
- Deployments: Canary via Cloud Deploy pipeline; 5% traffic mirrored or dark launched; automated rollback if p95 latency > threshold or error rate > baseline + 3σ.
- Resilience: Regional failover by LB backend health checks; session stickiness minimized; idempotent request design.
- Data Layer: If stateful (payments), keep primary in original region with async replica to EU for read-mostly endpoints; evaluate Spanner for future multi-region consistency.
- Observability: SLO 99.9% latency (p95 < X ms). Multi-window burn alerts (fast: 5m, slow: 1h). Per-customer dashboards using labels.

### Tools
- Anthos Service Mesh, Cloud Armor, Memorystore, Cloud Deploy, Cloud Trace, Cloud Profiler, Binary Authorization.

### Risk Mitigation
- Pre-warm caches & image layers
- Load test with synthetic EU clients
- Staged DNS TTL reduction before traffic shift
- Backout: traffic weighting revert + cluster autoscaler scale down.

---

## 🧩 Section 3: Problem-Solving - Answer

**Objectives:** Cost ↓, robust schema evolution, data quality, hot/cold separation.

### Issues Identified
- Full table scans (no partitioning/clustering)
- Schema drift causing load failures
- Mixed query workloads on same table
- Streaming ingestion (higher cost) when batch sufficient

### Refactor Strategy
1. Ingestion
   - Land raw JSON in Cloud Storage (partitioned by dt=YYYY/MM/DD) → Dataflow (batch) → normalized staging tables.
   - Use schema registry (Pub/Sub schema or custom metadata) + versioning.
2. BigQuery Modeling
   - Raw (immutable), Staging (normalized), Curated (analytics-friendly). Apply partitioning (ingestion_time or date field) + clustering on high cardinal columns (customer_id, region).
   - Introduce materialized views for frequent aggregates.
3. Schema Evolution
   - Use `ALLOW_FIELD_ADDITION` load jobs; maintain compatibility tests in CI.
   - Apply Dataflow schema mapping layer (unknown fields → JSON blob column for later promotion).
4. Cost Optimization
   - Convert streaming to hourly batch loads (GCS → LOAD jobs)
   - Archive cold partitions (>90d) with long-term storage pricing; separate historical tables
   - Implement row-level partition pruning in queries via views enforcing date filter
   - Use slot reservations (flex slots) for peak hours only.
5. Data Quality
   - Data validation (Great Expectations / custom Dataflow side output). Store failed records separately.
   - Build log-based metrics for load success rate & schema drift events.
6. Governance
   - Labels: dataset=raw|staging|curated, env=prod, owner=team
   - Central BigQuery usage dashboard (bytes scanned vs budget)

### Outcome Metrics
- 50–70% query cost reduction (partition pruning + materialized views)
- Reduced load failures (schema mediation layer)
- Faster analyst queries (clustered curated tables)
- Predictable cost via scheduled batch & reservations.

---

**Previous:** [Back to Questions](mock_2_questions.md)
````
