# Google Cloud Platform (GCP) - Intermediate Interview Mock #2 - Answer Key

> **Difficulty:** Intermediate  
> **Duration:** ~45 minutes  
> **Goal:** Evaluate deeper understanding of secure architectures, data processing design, networking boundaries, cost/performance tuning, and production operations on GCP.

---

## 🧠 Section 1: Core Questions - Answers

### 1. Explain VPC Service Controls: what problem do they solve, key concepts (service perimeter, access levels), and a common pitfall.

**Problem:** Mitigate data exfiltration risk from Google-managed APIs (BigQuery, Cloud Storage, Pub/Sub, Secret Manager, etc.) even if IAM creds are compromised.

**Core Concepts:**
- **Service Perimeter:** Logical boundary around projects/services; blocks data movement to outside unless explicitly allowed.
- **Access Levels:** Context-based attributes (IP ranges, device policy) required for perimeter ingress (via Access Context Manager).
- **Bridging / Dry Run:** Test mode (dry run) before enforcement; perimeter bridges allow limited cross-perimeter access.

**Mechanism:** Adds an additional control layer beyond IAM; denies requests originating from outside perimeter (e.g., stolen credentials used from unknown IP) even if IAM allows.

**Common Pitfall:** Breaking CI/CD or data processing pipelines because service accounts in external project or Cloud Build are not inside the perimeter; forgetting to add necessary egress rules for legitimate integrations (e.g., Vertex AI training outputs).

### 2. Compare Cloud Run vs GKE for a latency-sensitive API needing sidecars, gRPC, and custom networking controls. Include at least three decision dimensions.

| Dimension | Cloud Run | GKE |
|-----------|-----------|-----|
| **Sidecars** | Not natively multiple containers per revision (workaround: inject logic into single container) | Full multi-container Pod support (Envoy, Istio, collectors) |
| **gRPC** | Supported (HTTP/2) but limited low-level tuning | Native with full control (ingress, load balancer types, mTLS meshes) |
| **Networking** | Basic (VPC connector, egress controls, no full CNI customization) | Advanced (NetworkPolicy, CNI, Pod CIDRs, internal load balancers) |
| **Autoscaling** | Request concurrency-based; cold starts possible | HPA + custom metrics; more predictable warm capacity |
| **Latency Tuning** | Limited knobs (min instances, concurrency) | Pod resource requests/limits, node class, bin packing control |
| **Observability** | Managed logs/metrics tracing; limited DaemonSets | Full ecosystem (Prometheus, OpenTelemetry collector, service mesh) |
| **Operational Overhead** | Minimal | Higher (patching, upgrades, capacity planning) |

**Conclusion:** For strict latency SLAs + sidecar patterns + custom network policy → GKE. For simpler microservice without complex sidecars → Cloud Run.

### 3. Describe BigQuery partitioning vs clustering: when to use each, how they impact cost, and a misuse example.

**Partitioning:** Physically divides table by date/time (ingestion, column, or integer range). Query engine prunes partitions → reduces bytes scanned & improves performance when filters include partition key.

**Clustering:** Organizes data blocks sorted by up to four clustering columns inside partitions (or whole table). Improves performance and cost by limiting block reads for selective predicates.

**When:**
- Use partitioning for time-series / append-only event data.
- Add clustering for frequent filters on high-cardinality columns (user_id, device_id) or for composite ordering (status, region, customer_id).

**Cost Impact:** Less data scanned => lower query cost; partition pruning + clustering block elimination combine.

**Misuse Example:** Partitioning by a high-cardinality string (e.g., user_id) producing thousands of tiny partitions (metadata overhead) while queries still scan broadly; or clustering on a low-cardinality field (e.g., boolean) giving no benefit.

### 4. What are IAM Conditions? Provide two example condition expressions and use cases.

**IAM Conditions:** Attribute-based access control (ABAC) overlay using CEL expressions; refine a binding with context (resource attributes, request time, IP, org tags, etc.).

**Examples:**
1. **Time-Bound Access:**
```json
"condition": {
  "title": "TempSupportAccess",
  "expression": "request.time < timestamp('2025-12-31T23:59:59Z')"
}
```
*Use Case:* Grant short-lived support engineer read access without manual revocation.

2. **Restricted IP Range:**
```json
"condition": {
  "title": "CorpNetworkOnly",
  "expression": "inIpRange(request.ip, '203.0.113.0/24')"
}
```
*Use Case:* Limit sensitive BigQuery dataset access to corporate VPN.

Other patterns: restrict Cloud Storage object access by prefix (`resource.name.startsWith('projects/_/buckets/logs-bkt/objects/eu/')`).

### 5. Compare CMEK vs CSEK (Customer-Managed vs Customer-Supplied Encryption Keys) in GCP: responsibilities, auditability, and when to choose each.

| Aspect | CMEK | CSEK |
|--------|------|------|
| Key Hosting | Cloud KMS | Customer holds raw key material |
| Rotation | Automatic or manual via KMS versions | Customer must rotate & re-encrypt data manually |
| Audit Logs | Full KMS audit (use, rotate, destroy) | Limited (service sees only encrypted key) |
| Risk of Loss | Lower (managed redundantly) | Higher (loss = unrecoverable data) |
| Use Cases | Compliance needing external control, revocation | Rare: extreme regulatory mandates to never expose key material to provider |
| Ease | Easier to integrate | Higher operational burden |

**Summary:** Prefer CMEK for most regulated workloads; CSEK only if policy forbids provider-managed key storage.

### 6. Explain a typical Dataflow (Apache Beam) pipeline architecture ingesting Pub/Sub events to BigQuery with deduplication and late data handling.

**Flow:** Pub/Sub → Dataflow Streaming Job → BigQuery (raw & aggregated tables).

**Key Elements:**
- **Windowing:** Fixed or sliding windows (e.g., 1m) with *event time* semantics.
- **Watermarks:** Estimate event-time completeness; triggers pane emission.
- **Allowed Lateness:** Additional grace period for late events (e.g., 10m) before final pane closes.
- **Deduplication:** Use `PubsubMessage` attributes (messageId) or domain key; maintain state (Set) + TTL to discard duplicates.
- **Dead-Letter:** Side output of parse/validation failures → Pub/Sub `dead-letter` topic + Cloud Storage.
- **Backpressure & Autoscaling:** Streaming Engine + Dataflow autoscaling workers.
- **Schema Evolution:** Use BigQuery schema-compatible field additions.

**Pseudo Beam (Python):**
```python
| pubsub.ReadFromPubSub(topic=...) \
  | beam.Map(parse_json) \
  | beam.WithKeys(lambda e: e['event_id']) \
  | beam.Distinct() \
  | beam.WindowInto(FixedWindows(60), allowed_lateness=minutes(10), trigger=AfterWatermark()) \
  | beam.Map(to_bq_row) \
  | beam.io.WriteToBigQuery(table=..., method='STREAMING_INSERTS')
```

### 7. What is Private Service Connect (PSC)? Contrast it with VPC Peering and Serverless VPC Access.

**Private Service Connect:** Provides private (RFC1918) endpoints to access Google APIs, partner services, or publish your own service privately to consumers without exposing producer network details. Decouples producer & consumer networks (no shared routing domain).

| Feature | PSC | VPC Peering | Serverless VPC Access |
|---------|-----|------------|-----------------------|
| Purpose | Private endpoint to services/APIs | Full mesh network connectivity | Allow serverless (Cloud Run/Functions/App Engine) to reach VPC resources |
| Route Sharing | No (service-level) | Yes (full subnet reachability) | One-way from serverless into VPC |
| Transitivity | N/A (endpoint) | No | N/A |
| Use Case | Expose service to customers securely | Multi-project internal network | Private DB access for serverless |
| Isolation | Strong (endpoint abstraction) | Lower (broad trust) | Limited to egress path |

**Benefit:** Fine-grained exposure without broad network trust required by peering.

### 8. Outline a secure Cloud Build pipeline for container images, covering provenance, vulnerability scanning, least privilege, and promotion strategy.

**Pipeline Stages:**
1. **Source Fetch:** Trigger on Git push / tag.
2. **Build:** Cloud Build uses minimal builder image; build with `docker build` or `kaniko`.
3. **Vulnerability Scan:** Artifact Registry scanning enabled; block if High/Critical unresolved.
4. **Provenance / SLSA:** Enable build provenance (attestations) and store in Artifact Registry; optionally Binary Authorization policy requires attestation.
5. **Tests:** Unit + integration; ephemeral services with Cloud Build machine type.
6. **Signing:** Use Cloud KMS key or Sigstore (cosign) to sign image digest.
7. **Policy Gate:** Binary Authorization enforces signature + no critical vulns.
8. **Promotion:** Tag immutable digest to `:staging` → deploy; after soak + automated tests pass, retag digest to `:prod` (no rebuild).

**Least Privilege:**
- Dedicated Cloud Build service account with only `roles/artifactregistry.writer`, `roles/cloudkms.signerVerifier`, `roles/run.admin` (no broad editor).
- Separate deploy SA per environment.

**Example cloudbuild.yaml (excerpt):**
```yaml
steps:
- name: gcr.io/cloud-builders/docker
  args: ['build','-t','$IMAGE_DIGEST','--label','git_sha=$COMMIT_SHA','.']
- name: gcr.io/cloud-builders/gcloud
  args: ['artifacts','docker','tags','add','$IMAGE_DIGEST','$REPO_URI:$SHORT_SHA']
- name: gcr.io/cloud-builders/gcloud
  args: ['artifacts','docker','vuln','list','$IMAGE_DIGEST']
- name: gcr.io/cloud-builders/gcloud
  args: ['beta','container','binauthz','attestations','create','--artifact-url','$IMAGE_DIGEST','--attestor','secure-attestor']
images: ['$IMAGE_DIGEST']
``` 

**Promotion:** Retag digest rather than rebuilding → ensures reproducibility & traceability.

---

## ⚙️ Section 2: Scenario - Answer

**High-Level Architecture:**

```
Devices → (TLS MQTT/HTTPS) → Global Load Balancer / IoT Core* → Pub/Sub (raw)
                                            (IoT Core deprecated -> custom ingest on Cloud Run/GKE)
                     ↓                                   ↓
              Pub/Sub (raw backlog)           Dead-letter (invalid)
                     ↓
              Dataflow Streaming Pipeline (window, dedupe, enrich)
                     ↓                ↓                     ↓
           BigQuery (recent hot)   Cloud Storage (raw/partitioned)   Feature Extract (BQ → GCS parquet)
                     ↓
         BigQuery (hourly rollup aggregate table)
```

**Components:**
- **Ingestion:** Global External HTTP(S) Load Balancer → regional Cloud Run services (autoscale) handling device auth & protocol translation (or managed IoT gateway replacement). Pub/Sub for durable fan-in with ordered keys per device (ordering key = device_id for intra-device ordering).
- **Buffering & Spikes:** Pub/Sub scales horizontally; backlog observable via metrics; flow control in Dataflow.
- **Processing:** Dataflow streaming job: event-time windows (5m + allowed lateness 10m) produce hourly rollup table via BigQuery MERGE; deduplicate using device_id+timestamp hash; late events update rollups.
- **Storage Tiers:**
  - Hot analytics (30 days): BigQuery partitioned + clustered (partition by ingestion_date, cluster by device_id).
  - Cold raw (1 year): Cloud Storage bucket (partitioned by dt=YYYY-MM-DD/region=us|eu). Lifecycle: Nearline @ 30d, Coldline @ 180d.
- **Replay (48h):** Retain Pub/Sub messages for 48h (subscription retention) OR store raw Cloud Storage objects; to replay, launch temporary Dataflow job reading last 48h GCS paths.
- **ML Feature Export:** Scheduled (Cloud Composer / Cloud Scheduler + Cloud Run Job) daily query last 30 days BigQuery → export parquet to GCS `ml/features/`.
- **Regional Access Control:** Separate projects per region or BigQuery row-level security (RLS) & column-level security to mask PII for non-region teams; VPC Service Controls perimeter around analytics & storage; IAM Conditions restricting IP/location.
- **Exactly-Once Semantics:** Idempotent writes using BigQuery MERGE on composite key (device_id, event_ts); Dataflow dedupe state TTL > max lateness.
- **Monitoring:** Cloud Monitoring dashboards: Pub/Sub backlog, Dataflow watermark lag, BigQuery slot usage; alert if lag > threshold.
- **Cost Optimizations:** Slot reservations only for baseline; autoscale streaming; compressed GCS (gzip/parquet); prune columns; partition & cluster; lifecycle tiers for cold storage.

**Security:**
- Mutual TLS or signed tokens for device auth; Secret Manager for service credentials; CMEK for BigQuery & GCS if compliance; audit logs exported centrally.

### Why This Meets Requirements
- Handles spikes (Pub/Sub + autoscale) & sub-second ingest path.
- Replay via retained raw storage + ephemeral reprocessing.
- Exactly-once via dedupe + MERGE.
- Regional data access control via RLS + VPC SC.
- Cost controlled through tiered storage & partitioning.

---

## 🧩 Section 3: Problem-Solving - Answer

**Goal:** Strong tenant isolation, scalable batch compute, minimal cluster overhead.

### Recommended Design

**Project Structure:**
- `tenant-<ID>-data` projects (per-tenant) containing storage (GCS buckets), BigQuery datasets, and KMS key ring (CMEK per tenant).
- Shared `platform-control` project: identity (Cloud IAM groups), Cloud Build pipelines, central monitoring, Artifact Registry, service catalog.
- Optional `central-logs` project: consolidated audit & access logs.

**Encryption:**
- Each tenant project: KMS key (CMEK) for BigQuery & GCS; disabling a key = immediate revocation of access (data unreadable) fulfilling revocation requirement.

**Data Processing Jobs:**
- Use Cloud Run Jobs or Dataflow for batch; triggered via Cloud Scheduler in control project calling tenant-specific Cloud Run Job (invoker IAM) through Private Service Connect if needed.
- For heavier ETL, per-tenant Dataflow jobs with service account limited to tenant resources only.

**Identity & IAM:**
- Per-tenant service accounts: `tenant-<ID>-sa` with least privilege (e.g., `roles/bigquery.dataEditor` on that tenant dataset only).
- Control plane SA with `roles/run.invoker` across tenants; no direct data access.
- Use IAM Conditions to restrict support staff access by request time & IP.

**Networking:**
- Shared VPC exporting subnets to tenant projects (optionally) or isolate completely; if shared, firewall rules with tight tags; VPC SC perimeter containing all tenant projects to reduce exfiltration vector.

**Observability:**
- Aggregated log sinks (per tenant) exporting only metrics & audit logs to `central-logs` BigQuery for platform analytics; exclude raw PII to minimize exposure.

**Promotion / Deployment:**
- Single artifact built once (Cloud Build) → tagged digest deployed per tenant with environment-specific config (env vars/Secret Manager references).

**Cost Tracking:**
- Billing export uses project boundary; optionally apply consistent labels (`tenant=<ID>`, `component=etl|api`).

**Revocation Flow:**
1. Disable tenant KMS key (immediate cryptographic lock). 
2. Revoke service account access.
3. Archive project (optional) or schedule deletion.

### Trade-offs vs Single Project + Labels Only
| Aspect | Multi-Project | Single Project + Labels |
|--------|---------------|--------------------------|
| Isolation | Strong (IAM, quotas, blast radius) | Weaker (logical only) |
| Compromise Impact | Limited to tenant project | Wider lateral movement risk |
| Key Management | Per-tenant CMEK natural | Complex key scoping logic |
| Operational Overhead | Higher (more projects) | Lower admin overhead |
| Billing Clarity | Native per project | Requires label hygiene |
| Governance Policies | Fine-grained (org/folder inheritance) | Harder to scope exceptions |

**Why Not GKE Multi-namespace Only?** Namespaces share cluster control plane risk; noisy neighbor resource contention; weaker encryption separation.

### Orchestration Simplicity
Cloud Run Jobs avoid cluster scheduling complexity; Dataflow only for heavy data transformations.

---
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
