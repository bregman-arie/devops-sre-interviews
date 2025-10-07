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
