# Google Cloud Platform (GCP) - Beginner Interview Mock #4

> **Difficulty:** Beginner  
> **Duration:** ~30 minutes  
> **Goal:** Assess foundational knowledge around networking basics, load balancing, DNS, container delivery, infrastructure as code, database reliability, caching, and logging/auditing in GCP.

---

## 🧠 Section 1: Core Questions

1. What are the main types of Google Cloud load balancers (HTTP(S), SSL Proxy, TCP Proxy, Network, Internal) and when would you use an HTTP(S) load balancer vs a Network Load Balancer? [📖 Answer](mock_4_answers.md#1-what-are-the-main-types-of-google-cloud-load-balancers-https-ssl-proxy-tcp-proxy-network-internal-and-when-would-you-use-an-https-load-balancer-vs-a-network-load-balancer)
2. Explain the basic purpose of Cloud DNS. How does it integrate with global load balancing? [📖 Answer](mock_4_answers.md#2-explain-the-basic-purpose-of-cloud-dns-how-does-it-integrate-with-global-load-balancing)
3. Compare Artifact Registry and Container Registry. Why should new projects prefer Artifact Registry? [📖 Answer](mock_4_answers.md#3-compare-artifact-registry-and-container-registry-why-should-new-projects-prefer-artifact-registry)
4. What is Terraform and why would a team adopt it for GCP resource management? Give a simple example workflow. [📖 Answer](mock_4_answers.md#4-what-is-terraform-and-why-would-a-team-adopt-it-for-gcp-resource-management-give-a-simple-example-workflow)
5. How do you enable high availability and backups for Cloud SQL at a beginner level? [📖 Answer](mock_4_answers.md#5-how-do-you-enable-high-availability-and-backups-for-cloud-sql-at-a-beginner-level)
6. What is Cloud CDN and when does it help? Mention required components to enable it for a static site. [📖 Answer](mock_4_answers.md#6-what-is-cloud-cdn-and-when-does-it-help-mention-required-components-to-enable-it-for-a-static-site)
7. What are Cloud Audit Logs (Admin, Data Access, System Event, Policy) and why are they important? [📖 Answer](mock_4_answers.md#7-what-are-cloud-audit-logs-admin-data-access-system-event-policy-and-why-are-they-important)
8. List four beginner-friendly logging & observability best practices in GCP. [📖 Answer](mock_4_answers.md#8-list-four-beginner-friendly-logging--observability-best-practices-in-gcp)

---

## ⚙️ Section 2: Scenario

**Scenario:**  
You have a small containerized REST API (public) and a scheduled daily data aggregation job. Both are currently run manually on a single VM. You want to modernize using managed GCP services with minimal ops. Requirements: auto-scaling for the API, versioned deployments, secrets stored securely, scheduled job runs daily at 01:00 UTC, and basic monitoring + cost awareness.

Describe the target architecture using GCP services. Include: runtime platforms, image storage, secret handling, scheduling, monitoring, and networking.

[📖 Answer](mock_4_answers.md#️-section-2-scenario---answer)

---

## 🧩 Section 3: Problem-Solving

**Task:**  
Design a lightweight logging and audit baseline for a new 3-project setup (dev, staging, prod). Goals:
- Central visibility of errors and access activity
- Retain detailed logs for 30 days, summaries for 180 days
- Alert on API error spike (HTTP 5xx >2% for 5 minutes)
- Minimize cost and complexity

Which GCP services, export targets, filters, sinks, and IAM roles would you use? Provide a concise plan.

[📖 Answer](mock_4_answers.md#-section-3-problem-solving---answer)
