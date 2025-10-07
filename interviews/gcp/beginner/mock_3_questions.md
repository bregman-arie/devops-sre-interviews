# Google Cloud Platform (GCP) - Beginner Interview Mock #3

> **Difficulty:** Beginner  
> **Duration:** ~30 minutes  
> **Goal:** Assess practical understanding of beginner-level GCP serverless, networking basics, security, cost optimization, and multi-project organization.

---

## 🧠 Section 1: Core Questions

1. Compare Cloud Run, App Engine, and Cloud Functions. When would you choose each? [📖 Answer](mock_3_answers.md#1-compare-cloud-run-app-engine-and-cloud-functions-when-would-you-choose-each)
2. Explain Cloud Storage classes (Standard, Nearline, Coldline, Archive) and lifecycle management. Give an example policy. [📖 Answer](mock_3_answers.md#2-explain-cloud-storage-classes-standard-nearline-coldline-archive-and-lifecycle-management-give-an-example-policy)
3. What is a Shared VPC and why use it? How does it differ from VPC Peering? [📖 Answer](mock_3_answers.md#3-what-is-a-shared-vpc-and-why-use-it-how-does-it-differ-from-vpc-peering)
4. How do you securely store and access application secrets in GCP? Compare Secret Manager vs environment variables. [📖 Answer](mock_3_answers.md#4-how-do-you-securely-store-and-access-application-secrets-in-gcp-compare-secret-manager-vs-environment-variables)
5. What is Cloud Scheduler and how can it integrate with other GCP services? Provide a simple use case. [📖 Answer](mock_3_answers.md#5-what-is-cloud-scheduler-and-how-can-it-integrate-with-other-gcp-services-provide-a-simple-use-case)
6. What are Spot (Preemptible) VMs? How do they help reduce cost and what are their limitations? [📖 Answer](mock_3_answers.md#6-what-are-spot-preemptible-vms-how-do-they-help-reduce-cost-and-what-are-their-limitations)
7. What is Cloud NAT? When do you need it and what problem does it solve? [📖 Answer](mock_3_answers.md#7-what-is-cloud-nat-when-do-you-need-it-and-what-problem-does-it-solve)
8. List four cost optimization techniques for beginner GCP workloads and briefly explain each. [📖 Answer](mock_3_answers.md#8-list-four-cost-optimization-techniques-for-beginner-gcp-workloads-and-briefly-explain-each)

---

## ⚙️ Section 2: Scenario

**Scenario:**  
You need to build a lightweight image processing workflow: users upload images through a simple HTTPS endpoint, the platform generates thumbnails, stores metadata, and notifies a Slack channel when processing completes. You want minimal ops overhead, fast iteration, and cost efficiency.

Describe the architecture using GCP services. Cover: ingestion, processing, storage, notification, scaling, and security (IAM + secrets).

[📖 Answer](mock_3_answers.md#️-section-2-scenario---answer)

---

## 🧩 Section 3: Problem-Solving

**Task:**  
Design a basic multi-project layout for a small startup adopting GCP with these needs:
- Separate dev, staging, prod application environments
- Central shared networking (single ingress) and logging
- Simple cost allocation & budgeting
- Minimal manual secret handling
- Ability to deploy a containerized API and a scheduled job

Which projects would you create, which services would you use, and how would networking, IAM, and cost controls be structured? Provide a concise architecture outline.

[📖 Answer](mock_3_answers.md#-section-3-problem-solving---answer)
