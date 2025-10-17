# Google Cloud Platform (GCP) - Advanced Interview Mock #6

> **Difficulty:** Advanced  
> **Duration:** ~60 minutes  
> **Goal:** Evaluate architectural judgment in resilient multi-region data systems, adaptive security, AI integration efficiency, and governance automation.

---

## 🧠 Section 1: Core Questions

1. Design a multi-region transactional + analytical dual-store pattern using Spanner (OLTP) and BigQuery (OLAP) with <5s replication lag for derived aggregates. Detail change capture, transformation, idempotency, schema evolution, and consistency guarantees to consumers. [📖 Answer](mock_6_answers.md#1-design-a-multi-region-transactional--analytical-dual-store-pattern-using-spanner-oltp-and-bigquery-olap-with-5s-replication-lag-for-derived-aggregates-detail-change-capture-transformation-idempotency-schema-evolution-and-consistency-guarantees-to-consumers)
2. Engineer a secure cross-environment (dev→prod) artifact promotion pipeline enforcing provenance, vulnerability gating, policy-as-code, and staged rollout. Use: Cloud Build, Artifact Registry, Binary Authorization, Cloud Deploy, and Config Sync. [📖 Answer](mock_6_answers.md#2-engineer-a-secure-cross-environment-devprod-artifact-promotion-pipeline-enforcing-provenance-vulnerability-gating-policy-as-code-and-staged-rollout-use-cloud-build-artifact-registry-binary-authorization-cloud-deploy-and-config-sync)
3. Propose an approach for adaptive data retention & tiering (hot vs warm vs cold vs purge) across GCS, BigQuery, Bigtable, and Archive Storage balancing SLOs, compliance (7-year finance), and cost/carbon goals. [📖 Answer](mock_6_answers.md#3-propose-an-approach-for-adaptive-data-retention--tiering-hot-vs-warm-vs-cold-vs-purge-across-gcs-bigquery-bigtable-and-archive-storage-balancing-slos-compliance-7-year-finance-and-costcarbon-goals)
4. Design a global secrets + encryption strategy combining CMEK, Cloud KMS, External Key Manager, and Secret Manager with per-service blast radius minimization and automated key rotation + revocation drills. [📖 Answer](mock_6_answers.md#4-design-a-global-secrets--encryption-strategy-combining-cmek-cloud-kms-external-key-manager-and-secret-manager-with-per-service-blast-radius-minimization-and-automated-key-rotation--revocation-drills)
5. Architect a near-real-time ML feature store (Vertex AI Feature Store + Bigtable + Pub/Sub) enabling online <20ms fetch and offline parity for training reproducibility. Include update fan-out, skew detection, and point-in-time correctness validation. [📖 Answer](mock_6_answers.md#5-architect-a-near-real-time-ml-feature-store-vertex-ai-feature-store--bigtable--pubsub-enabling-online-20ms-fetch-and-offline-parity-for-training-reproducibility-include-update-fan-out-skew-detection-and-point-in-time-correctness-validation)
6. Design a compliance logging & lineage platform integrating Cloud Audit Logs, Data Catalog, Dataplex, and BigQuery to enable regulated queries: “Show all principals who accessed PII field X in last 30d across region Y datasets.” [📖 Answer](mock_6_answers.md#6-design-a-compliance-logging--lineage-platform-integrating-cloud-audit-logs-data-catalog-dataplex-and-bigquery-to-enable-regulated-queries-show-all-principals-who-accessed-pii-field-x-in-last-30d-across-region-y-datasets)
7. Explain a strategy for proactive cost guardrails that dynamically throttle or reschedule workloads (Dataflow, Vertex AI training, BigQuery queries) when hitting budget anomaly thresholds while preserving critical SLO workloads. [📖 Answer](mock_6_answers.md#7-explain-a-strategy-for-proactive-cost-guardrails-that-dynamically-throttle-or-reschedule-workloads-dataflow-vertex-ai-training-bigquery-queries-when-hitting-budget-anomaly-thresholds-while-preserving-critical-slo-workloads)
8. Engineer a low-latency inter-region API mesh using Global Load Balancing + Traffic Director + gRPC with adaptive concurrency, circuit breaking, and eBPF-driven latency instrumentation to reduce P99 tail by 25%. [📖 Answer](mock_6_answers.md#8-engineer-a-low-latency-inter-region-api-mesh-using-global-load-balancing--traffic-director--grpc-with-adaptive-concurrency-circuit-breaking-and-ebpf-driven-latency-instrumentation-to-reduce-p99-tail-by-25)

---

## ⚙️ Section 2: Scenario

**Scenario:**  
Your analytics + transactional ecosystem has grown organically:
- OLTP: Multiple Cloud SQL Postgres instances per region; ad-hoc Dataflow jobs replicate to BigQuery with 2–15 minute lag.
- Feature Engineering: Scattered custom Redis caches + cron backfills for ML features.
- Security: Manual key rotation; secrets duplicated across repos.
- Costs: BigQuery slot over-provisioned (50% idle at night); Vertex AI training bursts collide with Dataflow peak windows.
- Observability: No global view tying data lineage → access logs → cost centers.

Strategic objectives (next 12 months):
- Consolidate OLTP → single global Spanner + governed change streams.
- Feature freshness target: <10s end-to-end for high-value features.
- Reduce analytics lag to <30s; unify batch + streaming ingestion.
- Implement adaptive key & secret lifecycle + blast radius segmentation.
- Achieve 25% analytics cost reduction & 15% carbon-intensity improvement.
- Provide compliance portal answering cross-dataset access lineage queries in <5s.

Design a target platform architecture: ingestion layers, dual-write / migration, feature pipeline, lineage + governance mesh, cost/carbon feedback loops, secret/crypto control plane, and operational resilience (failover + chaos testing approach).

[📖 Answer](mock_6_answers.md#️-section-2-scenario---answer)

---

## 🧩 Section 3: Problem-Solving

**Task:**  
Define an autonomous “Adaptive Data & Compute Orchestrator” that: (1) predicts slot / GPU contention 5–10 minutes ahead, (2) re-prioritizes Dataflow & BigQuery workloads, (3) injects dynamic feature store backpressure, (4) enforces per-domain cost SLOs, and (5) produces explainable decision logs for audit.

Provide: control loop stages, signals, prediction model features, decision policy (optimization objective), safe-guard rails, and rollback mechanics.

[📖 Answer](mock_6_answers.md#-section-3-problem-solving---answer)
