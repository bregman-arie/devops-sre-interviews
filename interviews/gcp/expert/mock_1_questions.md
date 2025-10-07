# Google Cloud Platform (GCP) - Expert Interview Mock #1

> **Difficulty:** Expert  
> **Duration:** ~75 minutes  
> **Goal:** Evaluate mastery of planet-scale architecture, extreme reliability engineering, advanced security/posture control, and cost-performance governance across heterogeneous GCP workloads.

---

## 🧠 Section 1: Core Questions

1. Design a multi-continent trading & settlement backbone leveraging Spanner + regional low-latency caches while guaranteeing monotonic account balance reads globally. Detail write path, read path, and failure semantics. [📖 Answer](mock_1_answers.md#1-design-a-multi-continent-trading--settlement-backbone-leveraging-spanner--regional-low-latency-caches-while-guaranteeing-monotonic-account-balance-reads-globally-detail-write-path-read-path-and-failure-semantics)
2. Propose a strategy to continuously verify supply chain integrity (build → provenance → deploy → runtime) using Binary Authorization, Artifact Analysis, Cloud Deploy, and runtime drift detection (Config Sync / Policy Controller). How do you enforce revocation? [📖 Answer](mock_1_answers.md#2-propose-a-strategy-to-continuously-verify-supply-chain-integrity-build--provenance--deploy--runtime-using-binary-authorization-artifact-analysis-cloud-deploy-and-runtime-drift-detection-config-sync--policy-controller-how-do-you-enforce-revocation)
3. Engineer a unified, low-latency global pub/sub mesh bridging on-prem high-frequency market feeds with GCP, optimizing for fairness & bounded out-of-order delivery (<20ms skew). Include edge ingestion, compression, and ordering recovery. [📖 Answer](mock_1_answers.md#3-engineer-a-unified-low-latency-global-pubsub-mesh-bridging-on-prem-high-frequency-market-feeds-with-gcp-optimizing-for-fairness--bounded-out-of-order-delivery-20ms-skew-include-edge-ingestion-compression-and-ordering-recovery)
4. Explain an approach to implement adaptive SLOs with dynamic error budget reallocation across 40+ microservices (shared composite product SLO). How do you prevent starvation and budget gaming? [📖 Answer](mock_1_answers.md#4-explain-an-approach-to-implement-adaptive-slos-with-dynamic-error-budget-reallocation-across-40-microservices-shared-composite-product-slo-how-do-you-prevent-starvation-and-budget-gaming)
5. Architect a strict data sovereignty enforcement model spanning 12 jurisdictions with mixed OLTP (Spanner / Cloud SQL) + analytical BigQuery federations. Include policy tagging, encryption domains, and cross-border anonymization workflow. [📖 Answer](mock_1_answers.md#5-architect-a-strict-data-sovereignty-enforcement-model-spanning-12-jurisdictions-with-mixed-oltp-spanner--cloud-sql--analytical-bigquery-federations-include-policy-tagging-encryption-domains-and-cross-border-anonymization-workflow)
6. Design a chaos & resilience verification framework that injects multi-layer faults (network partitions, Spanner leader move, Pub/Sub backlog surge, key revocation) and automatically scores blast radius vs containment objectives. [📖 Answer](mock_1_answers.md#6-design-a-chaos--resilience-verification-framework-that-injects-multi-layer-faults-network-partitions-spanner-leader-move-pubsub-backlog-surge-key-revocation-and-automatically-scores-blast-radius-vs-containment-objectives)
7. Define a cost governance program for petabyte-scale ML + streaming analytics blending committed use discounts, dynamic slot allocation, preemptible fleets, and workload-aware carbon optimization. Provide control loops. [📖 Answer](mock_1_answers.md#7-define-a-cost-governance-program-for-petabyte-scale-ml--streaming-analytics-blending-committed-use-discounts-dynamic-slot-allocation-preemptible-fleets-and-workload-aware-carbon-optimization-provide-control-loops)
8. Construct a defense-in-depth strategy against insider data exfiltration involving BigQuery, GCS, Secret Manager, and Vertex AI notebooks in a regulated environment (PCI + GDPR). [📖 Answer](mock_1_answers.md#8-construct-a-defense-in-depth-strategy-against-insider-data-exfiltration-involving-bigquery-gcs-secret-manager-and-vertex-ai-notebooks-in-a-regulated-environment-pci--gdpr)

---

## ⚙️ Section 2: Scenario

**Scenario:**  
You are the lead platform architect for a real-time global payments network scaling from 25M to 250M daily transactions across North America, EU, LatAm, APAC. Current system (regional silos) causes reconciliation drift and latency spikes during FX settlement windows.

New platform must achieve:
- Consistent account debits/credits globally (<400ms end-to-end P99 including FX conversion)  
- 99.995% availability; zonal + regional failure masking  
- RPO ≤ 5s cross-region; monotonic ledger sequencing  
- Regulated artifact retention (7 years immutable)  
- Dynamic risk scoring inline (<50ms budget)  
- Carbon-aware workload placement (prefer low-carbon region if latency budget available)  

Design the full-stack architecture: consistency model, ledger storage, multi-region replication, risk scoring pipeline, failure domains, feature flag rollout, carbon optimization feedback loop, and compliance logging strategy.

[📖 Answer](mock_1_answers.md#️-section-2-scenario---answer)

---

## 🧩 Section 3: Problem-Solving

**Task:**  
Define an automated hierarchical failover controller for a three-tier topology (edge API → regional processing plane → global settlement core) that: (1) predicts saturation 3–5 minutes ahead (ML), (2) performs partial brownout of non-critical features, (3) promotes standby regions, and (4) validates post-failover invariants (no double debit, sequence gap-free). Provide control flow, signal sources, and rollback criteria.

[📖 Answer](mock_1_answers.md#-section-3-problem-solving---answer)
