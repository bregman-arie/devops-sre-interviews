# Google Cloud Platform (GCP) - Intermediate Interview Mock #4

> **Difficulty:** Intermediate  
> **Duration:** ~45 minutes  
> **Goal:** Assess your ability to design resilient data platforms, secure network boundaries, optimize cost/performance trade-offs, and apply operational excellence patterns on GCP.

---

## 🧠 Section 1: Core Questions

1. Your team needs to deploy a highly available PostgreSQL database on GCP. Which GCP service would you use, and how would you design for HA and automatic failover? [📖 Answer](mock_4_answers.md#1-your-team-needs-to-deploy-a-highly-available-postgresql-database-on-gcp-which-gcp-service-would-you-use-and-how-would-you-design-for-ha-and-automatic-failover)
2. Contrast Cloud SQL HA (REGIONAL) with running PostgreSQL on a GKE StatefulSet + Patroni. What trade-offs drive the choice? [📖 Answer](mock_4_answers.md#2-contrast-cloud-sql-ha-regional-with-running-postgresql-on-a-gke-statefulset--patroni-what-trade-offs-drive-the-choice)
3. How would you implement automated cross-region disaster recovery (RPO < 5 min) for a Cloud SQL PostgreSQL instance? [📖 Answer](mock_4_answers.md#3-how-would-you-implement-automated-cross-region-disaster-recovery-rpo--5-min-for-a-cloud-sql-postgresql-instance)
4. Explain strategies to minimize lock contention and connection pressure on Cloud SQL under spiky microservice load. [📖 Answer](mock_4_answers.md#4-explain-strategies-to-minimize-lock-contention-and-connection-pressure-on-cloud-sql-under-spiky-microservice-load)
5. Describe a secure pattern for exposing an internal REST API to selected external partners using GCP networking/security features. [📖 Answer](mock_4_answers.md#5-describe-a-secure-pattern-for-exposing-an-internal-rest-api-to-selected-external-partners-using-gcp-networkingsecurity-features)
6. How do you design a cost-efficient BigQuery data model for semi-structured event data while enabling low-latency product analytics? [📖 Answer](mock_4_answers.md#6-how-do-you-design-a-cost-efficient-bigquery-data-model-for-semi-structured-event-data-while-enabling-low-latency-product-analytics)
7. What layered controls prevent secret exfiltration from a compromised Cloud Run service account? Provide at least six controls. [📖 Answer](mock_4_answers.md#7-what-layered-controls-prevent-secret-exfiltration-from-a-compromised-cloud-run-service-account-provide-at-least-six-controls)
8. Outline an approach to progressive feature rollout + rollback across 20 Cloud Run services with dependency chains. [📖 Answer](mock_4_answers.md#8-outline-an-approach-to-progressive-feature-rollout--rollback-across-20-cloud-run-services-with-dependency-chains)

---

## ⚙️ Section 2: Scenario

**Scenario:**  
You are modernizing the data persistence layer of a SaaS platform. Current state: a single-zone Cloud SQL Postgres, cron-based ETL scripts on a VM, and ad-hoc exports to CSV for analytics. Target goals:
- Zero single-zone database dependency (zone outage survivable)  
- Lower p95 API latency (currently high due to connection storms)  
- Near real-time analytics (sub-5 min freshness) with cost control  
- Security: restricted egress & least-privilege identities  
- Automated schema migration & rollback path  

Design a target architecture using managed GCP services. Cover: database layer, connection management, analytics ingestion, secret management, migration strategy, and observability. Provide justifications.

[📖 Answer](mock_4_answers.md#️-section-2-scenario---answer)

---

## 🧩 Section 3: Problem-Solving

**Task:**  
Design a blue/green release and failback strategy for the PostgreSQL tier behind a mission‑critical payment API (Cloud Run). Constraints:
- Max 60s client visible disruption  
- Must validate schema + data correctness before full cutover  
- Support instant rollback without data divergence risk  
- Acceptable temporary dual-write overhead <= 10% CPU  
- Change window: off‑peak 15 minutes  

Propose the sequence (prep → sync → cut → validate → finalize / rollback) and the GCP features / tooling you rely on.

[📖 Answer](mock_4_answers.md#-section-3-problem-solving---answer)
