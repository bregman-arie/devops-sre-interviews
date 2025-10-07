# Google Cloud Platform (GCP) - Intermediate Interview Mock #3

> **Difficulty:** Intermediate  
> **Duration:** ~45 minutes  
> **Goal:** Assess architectural decision making across compute options, security boundaries, scalability, data handling, and operational excellence on GCP.

---

## 🧠 Section 1: Core Questions

1. You’re designing a scalable web application on Google Cloud Platform. How would you decide between Cloud Run, GKE, and Compute Engine for deploying your application, and what are the trade-offs of each? [📖 Answer](mock_3_answers.md#1-youre-designing-a-scalable-web-application-on-google-cloud-platform-how-would-you-decide-between-cloud-run-gke-and-compute-engine-for-deploying-your-application-and-what-are-the-trade-offs-of-each)
2. When would Cloud Tasks be preferable to Pub/Sub or Eventarc for asynchronous workloads? Provide two concrete examples. [📖 Answer](mock_3_answers.md#2-when-would-cloud-tasks-be-preferable-to-pubsub-or-eventarc-for-asynchronous-workloads-provide-two-concrete-examples)
3. Compare Cloud SQL + Redis cache vs Cloud Spanner for a multi-region read-heavy SaaS product (focus: consistency, scaling, cost, ops). [📖 Answer](mock_3_answers.md#3-compare-cloud-sql--redis-cache-vs-cloud-spanner-for-a-multi-region-read-heavy-saas-product-focus-consistency-scaling-cost-ops)
4. How do you enforce and monitor SLOs (availability & latency) for a Cloud Run based microservice? Mention metrics, alerting strategy, and error budget usage. [📖 Answer](mock_3_answers.md#4-how-do-you-enforce-and-monitor-slos-availability--latency-for-a-cloud-run-based-microservice-mention-metrics-alerting-strategy-and-error-budget-usage)
5. Explain strategies to minimize cold starts in Cloud Run and their trade-offs. [📖 Answer](mock_3_answers.md#5-explain-strategies-to-minimize-cold-starts-in-cloud-run-and-their-trade-offs)
6. What security controls would you layer to prevent data exfiltration from a BigQuery dataset containing regulated data? (At least five controls.) [📖 Answer](mock_3_answers.md#6-what-security-controls-would-you-layer-to-prevent-data-exfiltration-from-a-bigquery-dataset-containing-regulated-data-at-least-five-controls)
7. Describe a pattern for safe multi-tenant per-customer encryption in GCP using CMEK and key rotation without service downtime. [📖 Answer](mock_3_answers.md#7-describe-a-pattern-for-safe-multi-tenant-per-customer-encryption-in-gcp-using-cmek-and-key-rotation-without-service-downtime)
8. How would you design an automated cost anomaly detection workflow using native GCP tools? [📖 Answer](mock_3_answers.md#8-how-would-you-design-an-automated-cost-anomaly-detection-workflow-using-native-gcp-tools)

---

## ⚙️ Section 2: Scenario

**Scenario:**  
You are leading a migration from a monolithic application (single regional GCE VM stack) to a microservices architecture. Target state: 12 stateless services, a transactional relational store, analytics event pipeline, and scheduled batch processors. Requirements:
- Can expand to second region within 6 months (read traffic first, eventual write locality)  
- Deployment safety (progressive rollout + fast rollback)  
- Consistent identity and secret handling  
- Minimal ops burden for platform team of 3  
- Observability (structured logs, trace sampling, RED metrics)  

Design the target platform using GCP services (compute, networking, deployment, secrets, data, observability). Justify Cloud Run vs GKE vs hybrid for this case. Include rollout and multi-region readiness approach.

[📖 Answer](mock_3_answers.md#️-section-2-scenario---answer)

---

## 🧩 Section 3: Problem-Solving

**Task:**  
You need a resilient architecture for processing critical webhook callbacks from third-party payment providers. Characteristics: bursty (20× spikes), strict ordering per transaction id, retries with exponential backoff, idempotency, protection against duplicate delivery, audit logging, and downstream fan-out (fraud scoring + ledger writer + notification). Design a solution with GCP services covering ingestion, ordering, durability, processing, idempotency storage, failure handling, and monitoring.

[📖 Answer](mock_3_answers.md#-section-3-problem-solving---answer)
