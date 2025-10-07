# Google Cloud Platform (GCP) - Intermediate Interview Mock #4 - Answer Key

> **Difficulty:** Intermediate  
> **Duration:** ~45 minutes  
> **Goal:** Assess design skill across resilient data, secure exposure, scalable analytics, cost and operational safeguards.

---

## 🧠 Section 1: Core Questions - Answers

### 1. Your team needs to deploy a highly available PostgreSQL database on GCP. Which GCP service would you use, and how would you design for HA and automatic failover?

**Service:** Cloud SQL for PostgreSQL with **REGIONAL (HA)** configuration.

**Design Elements:**
- Regional HA → primary + synchronous standby in different zones (automatic failover ~60–120s).  
- Private IP + no public exposure; access via Cloud SQL Auth Proxy or Cloud SQL connectors (Cloud Run / GKE).  
- Automated backups + PITR (binary logs) → set backup window off-peak.  
- Read replicas (async) for heavy read/reporting; separate from HA standby.  
- Connection pooling (Cloud SQL built-in PgBouncer, or Cloud Run sidecar alternative) to reduce churn.  
- Metrics + alerts: replication lag, CPU, storage utilization, failed connections, failover events.  
- IAM: dedicated service accounts; no broad `cloudsql.admin` to applications.  
- Maintenance window pinned, review release notes before apply.  

### 2. Contrast Cloud SQL HA (REGIONAL) with running PostgreSQL on a GKE StatefulSet + Patroni. What trade-offs drive the choice?

| Aspect | Cloud SQL HA | GKE + Patroni |
|--------|--------------|---------------|
| Operations | Managed patching, backups | You manage versioning, patching, backup, tuning |
| Failover | Built-in (synchronous standby) | Patroni automation; more tuning knobs |
| Observability | Integrated metrics | Must aggregate (Prometheus), custom alerting |
| Custom Extensions | Limited subset supported | Full control (install any extension) |
| Scaling | Vertical (limited), read replicas | Horizontal more flexible (sharding), but complex |
| Networking | Simplified (private IP) | Full CNI control, may need load balancer/virtual IP |
| Cost | Higher for managed premium | Possibly cheaper infra but higher ops toil |
| Compliance | Built-in encryption, logging | You must enforce/combine primitives |

**Choose Cloud SQL** for speed, reliability with small ops team. **Choose Patroni** when needing exotic extensions, large-scale sharding, or custom failover logic.

### 3. How would you implement automated cross-region disaster recovery (RPO < 5 min) for a Cloud SQL PostgreSQL instance?

1. Primary: Regional HA in primary region.  
2. Create cross-region read replica (async).  
3. Monitor replication lag (alert if > 300s).  
4. Store automated backups + PITR logs in multi-region or replicate (default GCS durability).  
5. Automate promotion: Cloud Function triggered by incident decision (manual OR orchestrated) calling `gcloud sql instances promote-replica`.  
6. External endpoint abstraction: Use environment variable / secret for DB host, or service discovery layer; rotate to promoted instance.  
7. After promotion, rebuild former primary as replica once region healthy.  
8. If stricter RPO needed (<1–2s) evaluate Spanner or logical replication with more frequent commit discipline.  

### 4. Explain strategies to minimize lock contention and connection pressure on Cloud SQL under spiky microservice load.

- **Connection Pooling:** Use Cloud SQL PgBouncer or Cloud Run concurrency tuning to reuse connections.  
- **Reduce Chatty Patterns:** Batch writes; use COPY for bulk loads.  
- **Optimize Transactions:** Keep them short; avoid user think time inside transactions.  
- **Appropriate Indexing:** Prevent full table scans causing long-held locks.  
- **Hot Partition Mitigation:** Time bucket tables; avoid single counter rows.  
- **Read Scaling:** Move read-only queries to replicas + cache layer (Redis).  
- **Backoff & Circuit Breakers:** Prevent thundering herd reopens on transient failures.  
- **Prepared Statements / Plan Caching:** Reduce planning overhead.  

### 5. Describe a secure pattern for exposing an internal REST API to selected external partners using GCP networking/security features.

**Pattern:**
1. Internal service runs on internal HTTP(S) Load Balancer (or Cloud Run with internal ingress).  
2. Expose via **Private Service Connect (PSC) endpoint** per partner project for least privilege; partners connect through private IP.  
3. Layer **Cloud Armor** policy (geo/IP allowlists, rate limits) if external LB required.  
4. Mutual TLS (mTLS) or signed JWT (Workload Identity Federation) for auth.  
5. VPC Service Controls perimeter around backend data services.  
6. Audit Logging + anomaly detection on unusual access.  
7. Optional: Apigee or API Gateway for quota, key management, versioning.  

### 6. How do you design a cost-efficient BigQuery data model for semi-structured event data while enabling low-latency product analytics?

- **Schema:** Use nested & repeated fields for related attributes (reduces joins).  
- **Partition:** Ingestion or event_date (daily).  
- **Cluster:** High-cardinality filter columns (user_id, feature_flag).  
- **Shallow Wide Tables:** Denormalize; avoid snowflake joins for analytics.  
- **Materialized Views:** For top dashboards (rolling 1d/7d metrics) to reduce repeated full scans.  
- **Streaming vs Batch:** Use streaming inserts for real-time (<5 min) + periodic batch compaction.  
- **Query Optimization:** LIMIT early, select explicit columns, avoid SELECT *.  
- **Tiering:** Export cold (>180d) to GCS parquet if infrequently queried.  
- **Slot Management:** Use autoscaling or reservations sized to median workload; schedule heavy transformations off-peak.  

### 7. What layered controls prevent secret exfiltration from a compromised Cloud Run service account? Provide at least six controls.

1. **Least Privilege SA:** Only `secretAccessor` on specific secrets, not project-wide.  
2. **Secret Separation:** Different secrets per environment/service (blast radius).  
3. **VPC Egress Controls:** Serverless VPC connector + Cloud NAT route allowlist; restrict arbitrary exfil endpoints.  
4. **Outbound Firewall / Cloud Armor Egress (where applicable):** Limit to approved domains using egress proxy.  
5. **Secret TTL / Rotation:** Short-lived credentials (rotate keys automatically).  
6. **Access Transparency / Audit Logs:** Alert on anomalous secret access volume.  
7. **Runtime Detection:** Cloud Logging anomaly detection & Cloud IDS (if using VPC).  
8. **Token Audience Restrictions (OIDC):** Limit API tokens use outside intended service.  

### 8. Outline an approach to progressive feature rollout + rollback across 20 Cloud Run services with dependency chains.

1. **Version Tagging:** Immutable image digests; semantic version + commit SHA.  
2. **Dependency Graph:** Topologically order services (foundation libs → auth → user → billing → API gateway).  
3. **Automated Canary:** Each service: deploy new revision with 5–10% traffic + synthetic tests & SLO guardrails.  
4. **Orchestrated Promotion:** Pipeline promotes only after upstream dependencies stable for N minutes (no error budget burn).  
5. **Config Flagging:** Use runtime config (Firestore / Config service) for feature flags decoupled from deployment.  
6. **Rollback:** Instant revert by shifting traffic to previous revision; flags revert for half-complete features.  
7. **Coordinated Observability:** Correlate revision label in logs/traces; latency & error diff dashboards.  
8. **Freeze on Burn:** Multi-window burn alerts halt automation.  

---

## ⚙️ Section 2: Scenario - Answer

**Target Architecture Overview:**

```
Cloud Run (microservices) → Cloud SQL (Postgres REGIONAL) ← PgBouncer (managed) 
         │                       │
         ├─ Pub/Sub (event bus) ─┤
         └─ Dataflow (stream enrich) → BigQuery (partitioned + clustered) → Materialized Views
Secrets: Secret Manager | Metrics/Logs/Trace: Cloud Operations | Backups: Automated + PITR
```

**Components & Justification:**

| Concern | Design | Rationale |
|---------|--------|-----------|
| DB HA | Cloud SQL REGIONAL + read replica (optional) | Automatic zone failover; quick to implement |
| Connection Pressure | Built-in PgBouncer + Cloud Run concurrency tuning | Reduces connection explosions |
| Analytics Freshness | Pub/Sub streaming events → Dataflow → BigQuery (≤5m) | Real-time dashboards without heavy DB reads |
| Legacy Cron ETL | Replaced by scheduled Cloud Run Job / Dataform (or Composer later) | Managed scheduling, retries |
| Secrets | Secret Manager + least-priv SA | Eliminates embedding secrets in images |
| Schema Migration | Liquibase/Flyway run in Cloud Build deploy step (pre-deploy) with dry-run + rollback SQL | Controlled, auditable pipeline |
| Observability | Structured logs, trace sampling 10%, custom metrics (db_pool_wait, queue_lag) | Faster triage |
| Egress Security | Private IP DB, VPC SC around analytics project, restrict outbound via egress rules | Limits data exfiltration |
| Cost Control | Auto scale to zero on low-traffic services; BigQuery partition pruning; lifecycle on GCS exports | Prevents runaway spend |

**Migration Strategy:**
1. Enable replication → switch to REGIONAL instance.  
2. Introduce PgBouncer → test load.  
3. Emit domain events (change data capture via triggers or application-level publishing).  
4. Gradually shift analytics queries from OLTP to BigQuery materialized views.  
5. Decommission cron VM after parity validated.  

**Observability Enhancements:** SLO dashboards (availability, latency, error). Backpressure alerts (queue lag, connection saturation). Billing anomaly job.  

---

## 🧩 Section 3: Problem-Solving - Answer

**Blue/Green PostgreSQL Release Strategy:**

**Goal:** Safe engine version upgrade or major schema change with rollback.

### Sequence
1. **Prep:**
   - Create new Cloud SQL instance (GREEN) from latest PITR backup or replica clone; apply schema migrations in forward-only mode.  
   - Set read-only flag for validation queries only.  
2. **Sync:**
   - If using logical replication: enable publication on BLUE; GREEN subscribes until replication lag ~0.  
   - Verify row counts & checksums (sample tables).  
3. **Pre-Cut Validation:**
   - Run shadow read queries & synthetic transactions against GREEN (read-only).  
   - Compare performance metrics (latency, query plans).  
4. **Cutover (≤60s):**
   - Quiesce writes: application enters brief maintenance draining (reject new write requests with retry-after).  
   - Stop logical replication; final WAL apply; flip connection string (secret rotation or env var update) to GREEN.  
   - Lift maintenance; closely monitor error & latency metrics.  
5. **Post-Cut Validation (5–10 min):**
   - Dual-write optional (for critical tables) via feature flag writing to BLUE (now read-only) and GREEN to detect divergence (should be zero).  
   - If mismatch or errors > threshold, rollback.  
6. **Finalize:**
   - Decommission BLUE or keep as hot fallback (read-only) for fixed window (e.g., 24h).  
   - Snapshot & destroy after retention policy.  

### Rollback (Instant)
- Rotate connection secret back to BLUE; disable writes to GREEN (to avoid divergent fork).  
- Investigate migration issue; remediate; rerun process.  

### GCP Features / Tooling
- Cloud SQL clones / replicas, logical replication (if needed).  
- Secret Manager versioned connection strings (promote/demote via version alias).  
- Cloud Build pipeline for migration step gating (manual approval).  
- Cloud Monitoring alert policies (error rate spike, connection failure, replication lag).  
- Audit Logs for DDL statements.  

### Risk Mitigations
- Shadow testing reduces runtime surprises.  
- Dual-write (brief) detects logic regressions early.  
- Secret version rollback is atomic.  
- Logical replication reduces downtime vs full backup restore.  

---
