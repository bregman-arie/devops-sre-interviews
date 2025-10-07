# Google Cloud Platform (GCP) - Advanced Interview Mock #5

> **Difficulty:** Advanced  
> **Duration:** ~60 minutes  
> **Goal:** Challenge depth in cross-cloud portability, governance automation, real-time ML/analytics fusion, resilience economics, and security at scale.

---

## 🧠 Section 1: Core Questions

1. Design a multi-cloud (GCP + AWS) active/active API layer using Global External HTTP(S) Load Balancer + Cloud Interconnect + AWS GWLB. How do you route, fail, and observe without duplicating business logic? [📖 Answer](mock_5_answers.md#1-design-a-multi-cloud-gcp--aws-activeactive-api-layer-using-global-external-https-load-balancer--cloud-interconnect--aws-gwlb-how-do-you-route-fail-and-observe-without-duplicating-business-logic)
2. Propose a blueprint for end-to-end ML model governance (data → features → model artifact → deployment → drift retirement) using Dataplex, Data Catalog, Vertex AI, Artifact Registry, and Binary Authorization. [📖 Answer](mock_5_answers.md#2-propose-a-blueprint-for-end-to-end-ml-model-governance-data--features--model-artifact--deployment--drift-retirement-using-dataplex-data-catalog-vertex-ai-artifact-registry-and-binary-authorization)
3. Engineer a GCP-native approach to dynamic per-tenant network isolation for thousands of tenants sharing common microservices (avoid VPC explosion). Which primitives enable scalable isolation & policy? [📖 Answer](mock_5_answers.md#3-engineer-a-gcp-native-approach-to-dynamic-per-tenant-network-isolation-for-thousands-of-tenants-sharing-common-microservices-avoid-vpc-explosion-which-primitives-enable-scalable-isolation--policy)
4. Compare Cloud Run Jobs vs Dataflow vs Batch (GKE Autopilot + Kueue) for large-scale periodic backfills with variable data skew and strict cost ceilings. Provide a decision guide. [📖 Answer](mock_5_answers.md#4-compare-cloud-run-jobs-vs-dataflow-vs-batch-gke-autopilot--kueue-for-large-scale-periodic-backfills-with-variable-data-skew-and-strict-cost-ceilings-provide-a-decision-guide)
5. Devise a strategy to continuously minimize tail latency (P99) for a polyglot microservice environment (Go, Java, Python) using eBPF profiling, adaptive concurrency, and autoscaling policies. [📖 Answer](mock_5_answers.md#5-devise-a-strategy-to-continuously-minimize-tail-latency-p99-for-a-polyglot-microservice-environment-go-java-python-using-ebpf-profiling-adaptive-concurrency-and-autoscaling-policies)
6. Explain how to implement real-time least‑privilege reduction (“permission decay”) for service accounts based on access graph usage trends. Which data sources & automation loops? [📖 Answer](mock_5_answers.md#6-explain-how-to-implement-real-time-least-privilege-reduction-permission-decay-for-service-accounts-based-on-access-graph-usage-trends-which-data-sources--automation-loops)
7. Architect a frictionless cross-region data portability layer for BigQuery + GCS enabling selective lawful export & erasure requests (GDPR) with auditable irreversibility. [📖 Answer](mock_5_answers.md#7-architect-a-frictionless-cross-region-data-portability-layer-for-bigquery--gcs-enabling-selective-lawful-export--erasure-requests-gdpr-with-auditable-irreversibility)
8. Provide a holistic cost + carbon optimization framework factoring workload SLO tiers, CI pipeline cadence, data retention, and hardware accelerator scheduling (TPUs/GPUs). [📖 Answer](mock_5_answers.md#8-provide-a-holistic-cost--carbon-optimization-framework-factoring-workload-slo-tiers-ci-pipeline-cadence-data-retention-and-hardware-accelerator-scheduling-tpusgpus)

---

## ⚙️ Section 2: Scenario

**Scenario:**  
You’re asked to converge three legacy stacks into a unified event-driven architecture:
- A batch ETL (daily) building data marts in BigQuery
- A real-time event processor (custom VMs) producing partial aggregates
- A model serving tier (single-region) scoring fraud events with stale features

New objectives:
- Sub‑60s end-to-end freshness for KPI dashboards
- Real-time feature computation + batch reconciliation (no double counting)
- Multi-region active/active scoring with drift detection & rollback
- Per-tenant usage metering & chargeback
- Auditable lineage & reproducible historical recompute
- 20% cost reduction & carbon-aware scheduling for non-latency-critical jobs

Design the unified platform using GCP services: ingestion, processing layers (hot + warm + cold), storage zones, model feature lifecycle, chargeback tagging, replay path, governance controls.

[📖 Answer](mock_5_answers.md#️-section-2-scenario---answer)

---

## 🧩 Section 3: Problem-Solving

**Task:**  
Engineer an automated “SLO Guard” controller that: (1) aggregates service SLO + burn rate + deployment metadata, (2) halts risky rollouts, (3) allocates temporary error budget loans, (4) back-propagates cost of reliability debt to planning dashboards. Provide architecture, data flows, decision rules, and rollback safeguards.

[📖 Answer](mock_5_answers.md#-section-3-problem-solving---answer)
