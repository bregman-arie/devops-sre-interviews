# Google Cloud Platform (GCP) - Advanced Interview Mock #4

> **Difficulty:** Advanced  
> **Duration:** ~60 minutes  
> **Goal:** Evaluate mastery of cross-boundary networking, high-throughput data & ML pipelines, reliability automation, and sophisticated security/compliance operations.

---

## 🧠 Section 1: Core Questions

1. Architect a hybrid multi-cloud ingress layer using Global External HTTP(S) Load Balancer + Cloud Armor + Cloud CDN that can fail traffic to an alternate cloud while preserving client affinity. What mechanisms enable health detection and state continuity? [📖 Answer](mock_4_answers.md#1-architect-a-hybrid-multi-cloud-ingress-layer-using-global-external-https-load-balancer--cloud-armor--cloud-cdn-that-can-fail-traffic-to-an-alternate-cloud-while-preserving-client-affinity-what-mechanisms-enable-health-detection-and-state-continuity)
2. Explain how to build a low-latency feature store architecture supporting both streaming (≤2s lag) and historical batch backfill at 10+ TB/day using GCP services. [📖 Answer](mock_4_answers.md#2-explain-how-to-build-a-low-latency-feature-store-architecture-supporting-both-streaming-2s-lag-and-historical-batch-backfill-at-10-tbday-using-gcp-services)
3. Design a unified policy enforcement model using Organization Policy, IAM Conditions, Policy Controller, and VPC Service Controls to prevent unauthorized copying of sensitive BigQuery datasets to external projects. Provide enforcement + detection flows. [📖 Answer](mock_4_answers.md#3-design-a-unified-policy-enforcement-model-using-organization-policy-iam-conditions-policy-controller-and-vpc-service-controls-to-prevent-unauthorized-copying-of-sensitive-bigquery-datasets-to-external-projects-provide-enforcement--detection-flows)
4. Compare Dataflow vs Flink-on-GKE vs Spark (Dataproc) for a pipeline requiring complex event-time joins, low watermark lag, and exactly-once semantics at 500K events/sec. Provide a selection rationale. [📖 Answer](mock_4_answers.md#4-compare-dataflow-vs-flink-on-gke-vs-spark-dataproc-for-a-pipeline-requiring-complex-event-time-joins-low-watermark-lag-and-exactly-once-semantics-at-500k-eventssec-provide-a-selection-rationale)
5. Outline a blueprint for proactive SRE automation that uses burn-rate signals, change risk scores, and predictive saturation models to auto-throttle deployments across 30+ services. Which metrics and algorithms power this? [📖 Answer](mock_4_answers.md#5-outline-a-blueprint-for-proactive-sre-automation-that-uses-burn-rate-signals-change-risk-scores-and-predictive-saturation-models-to-auto-throttle-deployments-across-30-services-which-metrics-and-algorithms-power-this)
6. Describe a secure analytical enclave pattern for regulated data (PCI + HIPAA) enabling ephemeral querying via isolated Vertex AI notebook sessions with auditable, hermetic execution. [📖 Answer](mock_4_answers.md#6-describe-a-secure-analytical-enclave-pattern-for-regulated-data-pci--hipaa-enabling-ephemeral-querying-via-isolated-vertex-ai-notebook-sessions-with-auditable-hermetic-execution)
7. Engineer a dynamic cost-to-reliability optimization loop that tunes Cloud Run min instances, GKE node pool autoscaling parameters, and BigQuery slot reservations daily. What signals and control actions are applied? [📖 Answer](mock_4_answers.md#7-engineer-a-dynamic-cost-to-reliability-optimization-loop-that-tunes-cloud-run-min-instances-gke-node-pool-autoscaling-parameters-and-bigquery-slot-reservations-daily-what-signals-and-control-actions-are-applied)
8. Provide a hardened pattern for secret lifecycle & rotation across multi-env (dev/staging/prod) using Secret Manager, CMEK, workload identity, and just-in-time ephemeral access. [📖 Answer](mock_4_answers.md#8-provide-a-hardened-pattern-for-secret-lifecycle--rotation-across-multi-env-devstagingprod-using-secret-manager-cmek-workload-identity-and-just-in-time-ephemeral-access)

---

## ⚙️ Section 2: Scenario

**Scenario:**  
You are overhauling a legacy risk scoring system that currently runs hourly batch jobs, writes scores to a relational DB, and feeds nightly dashboards. New goals:
- Sub‑10s propagation of new events to updated risk scores  
- Hybrid input: Kafka (on-prem), Pub/Sub (cloud), file micro-batches  
- Strict lineage & reproducibility (regulator audits)  
- Rollback capability for scoring logic (deterministic replays)  
- Cost ceiling: 30% reduction vs current daily batch cluster  
- Multi-region resilience (failover < 3 min)  

Design the new architecture with: ingestion adapters, stateful processing, feature materialization, scoring service, rollback/replay pipeline, governance, and cost controls. Highlight key GCP services per function.

[📖 Answer](mock_4_answers.md#️-section-2-scenario---answer)

---

## 🧩 Section 3: Problem-Solving

**Task:**  
Create an automated credential compromise response pipeline that: detects anomalous Secret Manager access patterns, confirms via secondary signals, rotates affected secrets within 5 minutes, invalidates active sessions, and produces a forensic bundle. Provide event flow, services, decision logic, and rollback safeguards.

[📖 Answer](mock_4_answers.md#-section-3-problem-solving---answer)
