# Google Cloud Platform (GCP) - Expert Interview Mock #2

> **Difficulty:** Expert  
> **Duration:** ~75 minutes  
> **Goal:** Stress deep multi-domain architecture judgment: isolation, data + AI fusion, migration strategy, zero trust, cost/carbon governance, and production control systems.

---

## 🧠 Section 1: Core Questions

1. Engineer a global multi-tenant SaaS on GCP ensuring hard tenant isolation with minimal duplication. Define org / folder / project layout, per-tenant encryption, runtime isolation strategy (GKE multi-cluster vs namespace hardening vs Cloud Run), shared vs dedicated data plane (Spanner / BigQuery), tenant cost attribution, and lateral movement prevention. [📖 Answer](mock_2_answers.md#1-engineer-a-global-multi-tenant-saas-on-gcp-ensuring-hard-tenant-isolation-with-minimal-duplication-define-org--folder--project-layout-per-tenant-encryption-runtime-isolation-strategy-gke-multi-cluster-vs-namespace-hardening-vs-cloud-run-shared-vs-dedicated-data-plane-spanner--bigquery-tenant-cost-attribution-and-lateral-movement-prevention)
2. Design a unified low-latency analytics fabric combining Bigtable (hot time-series), BigQuery (ad hoc + federated), and Dataplex governance to serve <2s freshness dashboards + <300ms point lookups while tolerating schema evolution. Describe ingestion, storage tiers, freshness pipelines, and schema drift controls. [📖 Answer](mock_2_answers.md#2-design-a-unified-low-latency-analytics-fabric-combining-bigtable-hot-time-series-bigquery-ad-hoc--federated-and-dataplex-governance-to-serve-2s-freshness-dashboards--300ms-point-lookups-while-tolerating-schema-evolution-describe-ingestion-storage-tiers-freshness-pipelines-and-schema-drift-controls)
3. Outline a zero-downtime migration path from a regional Cloud SQL Postgres monolith to globally consistent Spanner for write-heavy workflows. Include dual-write / backfill strategy, consistency validation, cutover guardrails, error handling, and rollback trigger points. [📖 Answer](mock_2_answers.md#3-outline-a-zero-downtime-migration-path-from-a-regional-cloud-sql-postgres-monolith-to-globally-consistent-spanner-for-write-heavy-workflows-include-dual-write--backfill-strategy-consistency-validation-cutover-guardrails-error-handling-and-rollback-trigger-points)
4. Architect automated ephemeral least-privilege credential issuance for CI/CD pipelines spanning multiple projects using Workload Identity Federation, Cloud Build, Artifact Registry, Cloud Deploy, and IAM Conditions. Detail token lifecycle, scope reduction, revocation, and anomalous usage detection. [📖 Answer](mock_2_answers.md#4-architect-automated-ephemeral-least-privilege-credential-issuance-for-cicd-pipelines-spanning-multiple-projects-using-workload-identity-federation-cloud-build-artifact-registry-cloud-deploy-and-iam-conditions-detail-token-lifecycle-scope-reduction-revocation-and-anomalous-usage-detection)
5. Optimize a real-time ensemble ML inference platform (mix of transformer + gradient boosted models) on Vertex AI to meet P99 <150ms globally while reducing GPU hours by 40%. Provide architecture: model graph execution, dynamic batching, hardware tiering, multi-region routing, and A/B governance. [📖 Answer](mock_2_answers.md#5-optimize-a-real-time-ensemble-ml-inference-platform-mix-of-transformer--gradient-boosted-models-on-vertex-ai-to-meet-p99-150ms-globally-while-reducing-gpu-hours-by-40-provide-architecture-model-graph-execution-dynamic-batching-hardware-tiering-multi-region-routing-and-ab-governance)
6. Implement organization-wide egress control + DLP for 5k developers across hybrid environments. Include VPC Service Controls, hierarchical firewall policy, Cloud NAT strategy, DNS control plane (Cloud DNS + policy), Private Service Connect patterns, just-in-time exceptions, and audit pipeline. [📖 Answer](mock_2_answers.md#6-implement-organization-wide-egress-control--dlp-for-5k-developers-across-hybrid-environments-include-vpc-service-controls-hierarchical-firewall-policy-cloud-nat-strategy-dns-control-plane-cloud-dns--policy-private-service-connect-patterns-just-in-time-exceptions-and-audit-pipeline)
7. Define an observability + early anomaly detection framework to surface gray failures (partial degradation) across Spanner, GKE, and Pub/Sub before user-visible SLO burn. Include multi-signal correlation, adaptive trace sampling, streaming ML detectors, and remediation hooks. [📖 Answer](mock_2_answers.md#7-define-an-observability--early-anomaly-detection-framework-to-surface-gray-failures-partial-degradation-across-spanner-gke-and-pubsub-before-user-visible-slo-burn-include-multi-signal-correlation-adaptive-trace-sampling-streaming-ml-detectors-and-remediation-hooks)
8. Propose a cost + carbon aware capacity planning mechanism for 3-year growth of streaming + ML workloads. Cover forecasting inputs, reservation (CUDs / slots) acquisition cadence, model error bounding, carbon intensity weighting, and variance-driven re-optimization loop. [📖 Answer](mock_2_answers.md#8-propose-a-cost--carbon-aware-capacity-planning-mechanism-for-3-year-growth-of-streaming--ml-workloads-cover-forecasting-inputs-reservation-cuds--slots-acquisition-cadence-model-error-bounding-carbon-intensity-weighting-and-variance-driven-re-optimization-loop)

---

## ⚙️ Section 2: Scenario

**Scenario:**  
You inherit a multi-region e-commerce + personalization platform (NA, EU, APAC) currently suffering from: (a) inventory mismatch (eventual store updates lag 3–5 minutes), (b) high fraud/card testing bursts at edges, (c) overspent GPU inference budget, (d) privacy restrictions for EU behavioral data, and (e) noisy on-call due to alert fatigue.

New target state must achieve:
- <1s global inventory update propagation (99th)  
- Real-time fraud scoring inline (<60ms added) with adaptive throttling  
- 35% GPU cost reduction w/out latency regressions  
- Strict data residency & differential privacy for EU training exports  
- Alert reduction (≥50%) while improving MTTR  

Design end-to-end: event ingestion, authoritative inventory store, personalization inference routing, fraud pipeline, data residency boundaries, cost/carbon feedback loops, and observability/alerting overhaul.

[📖 Answer](mock_2_answers.md#️-section-2-scenario---answer)

---

## 🧩 Section 3: Problem-Solving

**Task:**  
Design an autonomous progressive global feature rollout controller for high-traffic APIs that: (1) computes real-time risk score (error, latency, saturation, fraud anomalies), (2) shifts traffic region-by-region with exponential backoff on uncertainty, (3) auto-pauses & bisects blast radius, (4) produces causal telemetry slices (e.g., only EU mobile Android spike), and (5) reverts deterministically when guardrail regression persists >N windows. Provide control loop, signals, state machine, and rollback invariants.

[📖 Answer](mock_2_answers.md#-section-3-problem-solving---answer)
