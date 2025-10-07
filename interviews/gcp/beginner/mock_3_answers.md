# Google Cloud Platform (GCP) - Beginner Interview Mock #3 - Answer Key

> **Difficulty:** Beginner  
> **Duration:** ~30 minutes  
> **Goal:** Assess practical understanding of beginner-level GCP serverless, networking basics, security, cost optimization, and multi-project organization.

---

## 🧠 Section 1: Core Questions - Answers

### 1. Compare Cloud Run, App Engine, and Cloud Functions. When would you choose each?

**Cloud Functions (FaaS):**
- Single-purpose functions triggered by events (HTTP, Pub/Sub, Storage)
- Auto-scales per request, scales to zero
- Minimal configuration; short-running logic
- Best for: Webhooks, lightweight event processing, glue code

**App Engine (Standard / Flexible - PaaS):**
- Deploy application code (no container required in Standard)
- Built-in traffic splitting, versions, scaling controls
- Standard: sandboxed runtimes, fast scale, scales to zero (some languages)
- Flexible: custom runtimes (Docker), longer startup, doesn’t scale to zero
- Best for: Monolithic web apps & APIs needing managed platform features

**Cloud Run (Serverless containers):**
- Runs any container (HTTP request/Cloud Events)
- Scales to zero; per-request concurrency configurable
- More portability (built from any language/framework)
- Supports CPU during request; optional min instances for warm start
- Best for: Containerized microservices, API backends, background workers (via Pub/Sub / Scheduler triggers)

**Selection Summary:**
- Need simplest event handler: Cloud Functions
- Need full app with versions/gradual rollout: App Engine
- Need container flexibility or non-supported runtime: Cloud Run
- Migrating existing container: Cloud Run
- Very granular billing & scale-to-zero for containerized API: Cloud Run

### 2. Explain Cloud Storage classes (Standard, Nearline, Coldline, Archive) and lifecycle management. Give an example policy.

**Classes:**
- **Standard:** Frequent access, low latency, multi-region/region; active content, websites
- **Nearline (~30-day min):** Infrequent (access < 1/month); backups, monthly reports
- **Coldline (~90-day min):** Very infrequent (quarterly); DR data, archives needing occasional access
- **Archive (~365-day min):** Long-term preservation; compliance, rarely accessed logs

All classes share same API & millisecond access (no restore delay). Differences are mainly storage vs access costs & minimum storage durations.

**Lifecycle Management:** Automates transitions and deletions based on object age / versions.

**Example Policy (JSON concept):**
```
If age >= 30 days → set storage class Nearline
If age >= 120 days → set storage class Coldline
If age >= 365 days → set storage class Archive
If age >= 730 days → delete
```
Applied via bucket lifecycle rules to reduce long-term cost automatically.

### 3. What is a Shared VPC and why use it? How does it differ from VPC Peering?

**Shared VPC:**
- Central (host) project provides VPC subnets to service projects
- Service project resources attach directly to host project networks
- Centralized network/security management (firewall, routes, Cloud NAT)
- Simplifies IAM & segmentation (teams isolated by project but share network)

**Benefits:** Centralized egress control, fewer duplicated networks, consistent security, easier auditing.

**VPC Peering:**
- Connects two independent VPC networks (private RFC1918 reachability)
- No transitive routing; each side manages own firewall & subnets

**Difference:** Shared VPC = single VPC spanned across projects (tight integration). Peering = two separate VPCs connected (loose coupling). Use Shared VPC for org-wide standard networking; use Peering for isolated/business-unit or third-party integration.

### 4. How do you securely store and access application secrets in GCP? Compare Secret Manager vs environment variables.

**Secret Manager:**
- Managed storage for secrets with versioning & IAM-controlled access
- Automatic replication, audit logs, CMEK support
- Access via API, gcloud, libraries; integrates with Cloud Run, GKE, Functions
- Rotation workflows via Pub/Sub triggers

**Environment Variables Only:**
- Stored in service configuration / deployment metadata
- Harder to rotate centrally; risk of accidental logging or exposure in history
- No version tracking; must redeploy to update

**Best Practice:** Store sensitive values in Secret Manager; inject at runtime (direct fetch) or mount (GKE). Use least privilege IAM (e.g., secretAccessor). Log access via Cloud Audit Logs. Use labels & rotation schedule.

### 5. What is Cloud Scheduler and how can it integrate with other GCP services? Provide a simple use case.

**Cloud Scheduler:**
- Fully managed cron service (HTTP, Pub/Sub, App Engine target)
- Supports retries, time zones, authentication (OIDC)

**Integrations:**
- Trigger Cloud Run/Functions via authenticated HTTP
- Publish Pub/Sub messages to start pipelines (Dataflow, Workflows)
- Kick off App Engine tasks

**Use Case:** Nightly (02:00 UTC) job triggers an HTTP endpoint on Cloud Run that generates daily usage reports, writes CSV to Cloud Storage, and publishes a Pub/Sub message notifying a Slack notification function.

### 6. What are Spot (Preemptible) VMs? How do they help reduce cost and what are their limitations?

**Spot / Preemptible VMs:** Deeply discounted Compute Engine instances (up to ~70–90% cheaper) that can be terminated by Google at any time (and always after 24h max for classic preemptible; Spot can be reclaimed anytime without fixed max). 

**Benefits:** Lower cost for fault-tolerant, stateless, batch, CI, big data, rendering workloads.

**Limitations:**
- Short / unpredictable lifetime (must handle termination notice)
- Not suitable for critical stateful services without checkpointing
- Capacity not guaranteed (may fail to start)
- Must architect for graceful interruption (use shutdown scripts or detect termination signal)

### 7. What is Cloud NAT? When do you need it and what problem does it solve?

**Cloud NAT (Network Address Translation):**
- Provides outbound internet access for resources with private (internal) IPs in a VPC (e.g., GCE VMs without external IPs, GKE nodes, Cloud Run VPC connectors)
- Maintains private-only inbound surface (improved security)

**Need It When:**
- Instances/containers need to call external APIs, download packages, reach public services, but should not have public IPs

**Solves:** Public egress without assigning/maintaining external IPs; simplifies security posture and IP management; supports high availability via regional managed service.

### 8. List four cost optimization techniques for beginner GCP workloads and briefly explain each.

1. **Right-sizing & Autoscaling:** Use autoscaling (GCE MIG / Cloud Run concurrency) to match demand; avoid over-provisioned instances.
2. **Idle Resource Cleanup:** Turn off unused VMs, delete unattached persistent disks / old images / stale snapshots; schedule dev environment shutdowns.
3. **Appropriate Storage Classes & Lifecycle:** Transition older objects to cheaper classes and delete obsolete logs/backups automatically.
4. **Spot / Committed Use Discounts:** Use Spot for stateless/batch; apply committed use discounts for predictable baseline workloads.
5. *(Bonus)* **Cloud Monitoring Budgets & Alerts:** Early detection of anomalies prevents runaway cost.

---

## ⚙️ Section 2: Scenario - Answer

**Goal:** Event-driven, low-maintenance image processing.

**Proposed Architecture:**
1. **Ingestion:** Cloud Run (public HTTPS) receives uploads; validates MIME/type & size.
2. **Storage:** Original images stored in Cloud Storage bucket (regional). Signed URLs optional for direct browser upload (bypassing service compute).
3. **Trigger:** Cloud Storage Object Finalize event → Pub/Sub → Cloud Function (or Cloud Run job) for processing (decouples spikes, easier retries).
4. **Processing:** Function uses ImageMagick (layer/container) to generate thumbnails; writes outputs to `thumbnails/` prefix in same or separate bucket.
5. **Metadata:** Write metadata (filename, checksum, sizes, timestamp) to Firestore (simple) or BigQuery (if analytics needed). For beginner: Firestore.
6. **Notification:** Publish completion message to Pub/Sub; separate Cloud Function posts to Slack webhook (secret in Secret Manager).
7. **Security:**
   - Service accounts: upload service (write object), processor (read original, write thumbnail, write Firestore), notifier (access secret + Pub/Sub subscribe).
   - Secret Manager for Slack webhook + API keys.
   - Bucket IAM: Principle of least privilege; optional uniform bucket-level access.
8. **Scaling:** Fully serverless (Cloud Run + Functions + Pub/Sub) auto-scales; backlog smoothing via Pub/Sub.
9. **Monitoring:** Log-based metric on processing failures; alert if error ratio > X%; Cloud Monitoring dashboard for function latency & Pub/Sub backlog.
10. **Cost Controls:** Lifecycle: delete original after 90 days (if not needed) or move to Nearline at 30 days; budgets with 50/80/100% alerts.

**Flow Diagram Concept:**
```
User → Cloud Run (Upload API) → Cloud Storage (original)
                           ⬇ (event)
                     Pub/Sub Topic → Cloud Function (thumbnail)
                            ⬇                      ⬇
                       Firestore (metadata)   Cloud Storage (thumb)
                            ⬇
                     Pub/Sub (complete) → Function (Slack notify)
```

**Why This Works (Beginner Friendly):** No servers to patch, components isolated, easy incremental enhancement (add Cloud CDN later for delivery).

---

## 🧩 Section 3: Problem-Solving - Answer

**Objective:** Simple, organized multi-project foundation.

**Project Layout:**
- `org-shared-infra` (Shared VPC host: networking, NAT, logging sink targets)
- `app-dev`, `app-staging`, `app-prod` (service projects attaching to Shared VPC)
- (Optional) `sec-ops` for org-level logging exports / security tooling

**Networking:**
- Shared VPC subnets (per region) defined in host project
- Central Cloud Load Balancer (HTTPS) fronting Cloud Run services (prod/staging)
- Cloud NAT in host for egress (GKE/Cloud Run with VPC connector if needed)

**Compute / Runtime:**
- Cloud Run for containerized API (one service per environment)
- Cloud Scheduler (in each env project) → Pub/Sub → Cloud Run job for scheduled tasks (e.g., daily cleanup/report)

**IAM Strategy:**
- Folder-level groups: `dev-engineers` (dev project editor limited), `ops` (viewer + monitoring), `security` (logs viewer, secret manager admin in shared infra)
- Least privilege service accounts: `api-runner`, `scheduler-job`, `pubsub-worker`
- Deny broad primitive roles in prod (no project editor for humans)

**Secrets:**
- Secret Manager in each env project (scoped secrets) + replication
- CI deploy SA granted `secretAccessor` only

**Logging & Monitoring:**
- Aggregated sink: All projects → central BigQuery dataset (cost & audit) in `org-shared-infra`
- Dashboards per environment; alerts on error rate/latency & budget thresholds

**Cost Allocation:**
- Enforce labeling: `env`, `service`, `team`
- Budgets: per project (50/80/100%), org roll-up; export billing to BigQuery for cost views filtered by labels

**Security Basics:**
- Org policy: restrict external IPs (use Cloud NAT) except load balancer
- Basic WAF (Cloud Armor) policy on HTTPS LB
- Enforce CMEK optional later (deferred for beginner)

**Deployment Flow:**
1. Dev merges → Cloud Build (or GitHub Actions) builds container → Artifact Registry
2. Deploy to `app-dev` Cloud Run; smoke tests
3. Promote image SHA to staging then prod using manual approval

**Scaling & Resilience:**
- Min instances: 0 in dev, 1 in prod for warm start
- Concurrency tuned (e.g., 80) for efficient cost/performance
- Pub/Sub backlog alert if > N messages for M minutes

**Summary Diagram Concept:**
```
               org-shared-infra (Shared VPC, NAT, LB, Logs Sink)
                 ├── Subnets / Firewall / Cloud NAT
                 └── HTTPS LB → Cloud Run (prod/staging backends)
                       ↑
    app-dev  app-staging  app-prod (attach to Shared VPC; Cloud Run, Scheduler, Pub/Sub)
```

**Why This Fits Beginner Needs:** Clear separation, centralized governance, minimal services, scalable foundation for growth without early complexity.

---
