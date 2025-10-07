# Google Cloud Platform (GCP) - Beginner Interview Mock #5

> **Difficulty:** Beginner  
> **Duration:** ~30 minutes  
> **Goal:** Assess understanding of governance basics, data store selection, access patterns, resiliency concepts, and operational practices in GCP.

---

## 🧠 Section 1: Core Questions

1. What is the Organization Policy Service in GCP? Give two example policies and why they are useful. [📖 Answer](mock_5_answers.md#1-what-is-the-organization-policy-service-in-gcp-give-two-example-policies-and-why-they-are-useful)
2. Explain the difference between resource labels and network tags. When do you use each? [📖 Answer](mock_5_answers.md#2-explain-the-difference-between-resource-labels-and-network-tags-when-do-you-use-each)
3. Compare Cloud SQL, Firestore, and BigQuery for a simple web application storing user profiles, transactions, and analytics events. Which would you use for each data type and why? [📖 Answer](mock_5_answers.md#3-compare-cloud-sql-firestore-and-bigquery-for-a-simple-web-application-storing-user-profiles-transactions-and-analytics-events-which-would-you-use-for-each-data-type-and-why)
4. What is a Cloud Run Job and how does it differ from a Cloud Run Service? Provide an example use case. [📖 Answer](mock_5_answers.md#4-what-is-a-cloud-run-job-and-how-does-it-differ-from-a-cloud-run-service-provide-an-example-use-case)
5. How would you set up a basic alert for elevated HTTP 5xx error rates in a Cloud Run application? List the high‑level steps. [📖 Answer](mock_5_answers.md#5-how-would-you-set-up-a-basic-alert-for-elevated-http-5xx-error-rates-in-a-cloud-run-application-list-the-high-level-steps)
6. Define RPO and RTO. Give a beginner-friendly example using Cloud SQL backups. [📖 Answer](mock_5_answers.md#6-define-rpo-and-rto-give-a-beginner-friendly-example-using-cloud-sql-backups)
7. Explain signed URLs for Cloud Storage. When would you use them instead of making a bucket public or granting IAM roles? [📖 Answer](mock_5_answers.md#7-explain-signed-urls-for-cloud-storage-when-would-you-use-them-instead-of-making-a-bucket-public-or-granting-iam-roles)
8. What is Error Reporting in Google Cloud Operations Suite and how does it help during incident response? [📖 Answer](mock_5_answers.md#8-what-is-error-reporting-in-google-cloud-operations-suite-and-how-does-it-help-during-incident-response)

---

## ⚙️ Section 2: Scenario

**Scenario:**  
You operate a multi-tenant SaaS note‑taking application. Reads are global, writes moderate, and you need fast access worldwide with low ops burden. Features: real-time note updates, user authentication, analytics on feature usage, and export-to-CSV job run nightly. Provide an architecture using beginner-appropriate GCP services. Address: data storage choices, global performance, real-time sync, analytics, scheduled export, cost & security basics.

[📖 Answer](mock_5_answers.md#️-section-2-scenario---answer)

---

## 🧩 Section 3: Problem-Solving

**Task:**  
Design a minimal, cost-conscious daily external-API ingestion pipeline that: (1) calls a third-party REST API at 00:30 UTC, (2) stores raw JSON, (3) transforms and loads curated fields into BigQuery, (4) alerts on failures, (5) keeps history for 90 days. List services, flow, and basic failure handling.

[📖 Answer](mock_5_answers.md#-section-3-problem-solving---answer)
