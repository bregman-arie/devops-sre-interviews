# Google Cloud Platform (GCP) - Intermediate Interview Mock #2

> **Difficulty:** Intermediate  
> **Duration:** ~45 minutes  
> **Goal:** Evaluate deeper understanding of secure architectures, data processing design, networking boundaries, cost/performance tuning, and production operations on GCP.

---

## 🧠 Section 1: Core Questions

1. Explain VPC Service Controls: what problem do they solve, key concepts (service perimeter, access levels), and a common pitfall. [📖 Answer](mock_2_answers.md#1-explain-vpc-service-controls-what-problem-do-they-solve-key-concepts-service-perimeter-access-levels-and-a-common-pitfall)
2. Compare Cloud Run vs GKE for a latency-sensitive API needing sidecars, gRPC, and custom networking controls. Include at least three decision dimensions. [📖 Answer](mock_2_answers.md#2-compare-cloud-run-vs-gke-for-a-latency-sensitive-api-needing-sidecars-grpc-and-custom-networking-controls-include-at-least-three-decision-dimensions)
3. Describe BigQuery partitioning vs clustering: when to use each, how they impact cost, and a misuse example. [📖 Answer](mock_2_answers.md#3-describe-bigquery-partitioning-vs-clustering-when-to-use-each-how-they-impact-cost-and-a-misuse-example)
4. What are IAM Conditions? Provide two example condition expressions and use cases. [📖 Answer](mock_2_answers.md#4-what-are-iam-conditions-provide-two-example-condition-expressions-and-use-cases)
5. Compare CMEK vs CSEK (Customer-Managed vs Customer-Supplied Encryption Keys) in GCP: responsibilities, auditability, and when to choose each. [📖 Answer](mock_2_answers.md#5-compare-cmek-vs-csek-customer-managed-vs-customer-supplied-encryption-keys-in-gcp-responsibilities-auditability-and-when-to-choose-each)
6. Explain a typical Dataflow (Apache Beam) pipeline architecture ingesting Pub/Sub events to BigQuery with deduplication and late data handling. [📖 Answer](mock_2_answers.md#6-explain-a-typical-dataflow-apache-beam-pipeline-architecture-ingesting-pubsub-events-to-bigquery-with-deduplication-and-late-data-handling)
7. What is Private Service Connect (PSC)? Contrast it with VPC Peering and Serverless VPC Access. [📖 Answer](mock_2_answers.md#7-what-is-private-service-connect-psc-contrast-it-with-vpc-peering-and-serverless-vpc-access)
8. Outline a secure Cloud Build pipeline for container images, covering provenance, vulnerability scanning, least privilege, and promotion strategy. [📖 Answer](mock_2_answers.md#8-outline-a-secure-cloud-build-pipeline-for-container-images-covering-provenance-vulnerability-scanning-least-privilege-and-promotion-strategy)

---

## ⚙️ Section 2: Scenario

**Scenario:**  
You are designing a data ingestion platform for near real-time analytics of IoT sensor metrics (temperature, voltage, status flags) from 250K devices globally. Requirements:
- Sub-second ingestion fan-in (spikes 5× during firmware rollout)
- Exactly-once semantics for aggregated hourly rollups
- Cold storage retention of raw data for 1 year (cost optimized)
- Regionally restricted operations team access (EU team only sees EU devices' PII fields)
- ML feature export (recent 30 days) for model training
- Ability to replay last 48 hours if a processing bug is discovered

Design the architecture using GCP services. Address ingestion, buffering, processing, storage tiers, access control, replay strategy, and cost optimizations.

[📖 Answer](mock_2_answers.md#️-section-2-scenario---answer)

---

## 🧩 Section 3: Problem-Solving

**Task:**  
You must implement a multi-tenant SaaS isolation model for customer-specific data processing jobs. Constraints:
- Each tenant gets logically isolated data path; strong identity boundary
- Some shared control-plane services (authn, metrics)
- Need per-tenant encryption key separation & revocation capability
- Batch compute jobs run daily with variable resources
- Minimize cluster management overhead; avoid building custom schedulers

Propose a design (projects, services, encryption, IAM, job orchestration) and explain trade-offs vs a single-project, label-only segregation approach.

[📖 Answer](mock_2_answers.md#-section-3-problem-solving---answer)
````markdown
# Google Cloud Platform (GCP) - Intermediate Interview Mock #2

> **Difficulty:** Intermediate  
> **Duration:** ~45 minutes  
> **Goal:** Evaluate ability to design resilient, secure, cost-aware GCP workloads involving networking, data, CI/CD, and observability.

---

## 🧠 Section 1: Core Questions

1. Compare the different Cloud Load Balancing tiers (Standard vs Premium) and when to choose one over the other. [📖 Answer](mock_2_answers.md#1-compare-the-different-cloud-load-balancing-tiers-standard-vs-premium-and-when-to-choose-one-over-the-other)
2. How does Cloud NAT differ from using external IPs on instances? What limits or scaling considerations exist? [📖 Answer](mock_2_answers.md#2-how-does-cloud-nat-differ-from-using-external-ips-on-instances-what-limits-or-scaling-considerations-exist)
3. Explain Artifact Registry vs Container Registry. Why migrate? What security and governance improvements does it enable? [📖 Answer](mock_2_answers.md#3-explain-artifact-registry-vs-container-registry-why-migrate-what-security-and-governance-improvements-does-it-enable)
4. Describe an end-to-end CI/CD pipeline on GCP for a containerized microservice (build, security scanning, deployment, verification). [📖 Answer](mock_2_answers.md#4-describe-an-end-to-end-cicd-pipeline-on-gcp-for-a-containerized-microservice-build-security-scanning-deployment-verification)
5. How would you secure inter-service communication for a multi-service GKE deployment (mTLS, identity, policies)? [📖 Answer](mock_2_answers.md#5-how-would-you-secure-inter-service-communication-for-a-multi-service-gke-deployment-mtls-identity-policies)
6. Compare Cloud SQL read replicas, cross-region replicas, and Cloud Spanner for multi-region reads. Trade-offs? [📖 Answer](mock_2_answers.md#6-compare-cloud-sql-read-replicas-cross-region-replicas-and-cloud-spanner-for-multi-region-reads-trade-offs)
7. What strategies exist to reduce BigQuery cost while sustaining performance (storage + query optimizations)? [📖 Answer](mock_2_answers.md#7-what-strategies-exist-to-reduce-bigquery-cost-while-sustaining-performance-storage--query-optimizations)
8. How do you design an observability stack on GCP that supports SLO-based alerting and cost attribution? [📖 Answer](mock_2_answers.md#8-how-do-you-design-an-observability-stack-on-gcp-that-supports-slo-based-alerting-and-cost-attribution)

---

## ⚙️ Section 2: Scenario

**Scenario:**  
You operate a payments API on GKE (regional cluster) fronted by an external HTTPS load balancer. Traffic is increasing globally, and you must: (a) reduce p95 latency for EU customers, (b) implement per-customer rate limiting, (c) protect backend from abusive bursts, (d) deploy new versions with <1% error budget burn risk, (e) add zero-trust style service identity. Provide an architectural enhancement plan.

[📖 Answer](mock_2_answers.md#️-section-2-scenario---answer)

---

## 🧩 Section 3: Problem-Solving

**Task:**  
A data ingestion pipeline loads semi-structured JSON into BigQuery hourly. Costs have grown 3× month-over-month. Analysts run ad-hoc queries scanning full tables. Load jobs occasionally fail due to schema drift (new nested fields). Design a refactor plan addressing: cost reduction, schema evolution handling, data quality validation, and separation of hot vs cold data.

[📖 Answer](mock_2_answers.md#-section-3-problem-solving---answer)
````
