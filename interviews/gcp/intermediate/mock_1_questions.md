# Google Cloud Platform (GCP) - Intermediate Interview Mock #1

> **Difficulty:** Intermediate  
> **Duration:** ~45 minutes  
> **Goal:** Assess your understanding of GCP architecture patterns, multi-region deployments, advanced networking, and production-ready solutions.

---

## 🧠 Section 1: Core Questions

1. Explain the different types of Cloud Load Balancers in GCP. When would you use each type? [📖 Answer](mock_1_answers.md#1-explain-the-different-types-of-cloud-load-balancers-in-gcp-when-would-you-use-each-type)
2. What is Cloud Armor and how does it integrate with load balancing? Describe its security features. [📖 Answer](mock_1_answers.md#2-what-is-cloud-armor-and-how-does-it-integrate-with-load-balancing-describe-its-security-features)
3. How do you implement Blue-Green deployments in GCP? Compare different approaches using various services. [📖 Answer](mock_1_answers.md#3-how-do-you-implement-blue-green-deployments-in-gcp-compare-different-approaches-using-various-services)
4. Explain VPC peering vs VPN vs Interconnect. When would you choose each connectivity option? [📖 Answer](mock_1_answers.md#4-explain-vpc-peering-vs-vpn-vs-interconnect-when-would-you-choose-each-connectivity-option)
5. What are the different database replication options in Cloud SQL? How do you handle failover scenarios? [📖 Answer](mock_1_answers.md#5-what-are-the-different-database-replication-options-in-cloud-sql-how-do-you-handle-failover-scenarios)
6. How do you implement autoscaling for different GCP services? Discuss metrics and policies. [📖 Answer](mock_1_answers.md#6-how-do-you-implement-autoscaling-for-different-gcp-services-discuss-metrics-and-policies)
7. Explain Workload Identity in GKE. How does it improve security compared to traditional service account keys? [📖 Answer](mock_1_answers.md#7-explain-workload-identity-in-gke-how-does-it-improve-security-compared-to-traditional-service-account-keys)
8. What are the best practices for organizing resources using labels, and how do they help with cost management? [📖 Answer](mock_1_answers.md#8-what-are-the-best-practices-for-organizing-resources-using-labels-and-how-do-they-help-with-cost-management)

---

## ⚙️ Section 2: Scenario

**Scenario:**  
Your company runs an e-commerce platform that has grown from a single region to requiring global presence. You currently have the application running in `us-central1` serving North American customers. Now you need to expand to serve European customers with low latency while maintaining high availability and data compliance (GDPR). The application consists of:

- Web frontend (React SPA)
- API backend (Node.js on GKE)  
- PostgreSQL database
- Redis cache
- File storage for product images

Design a multi-region architecture that includes:
- Geographic traffic routing
- Database replication strategy
- Failover mechanisms
- Data residency compliance
- Cost optimization considerations

[📖 Answer](mock_1_answers.md#️-section-2-scenario---answer)

---

## 🧩 Section 3: Problem-Solving

**Task:**  
Design a multi-region service with automated failover for a mission-critical financial application. The requirements are:

**Availability Requirements:**
- 99.9% uptime SLA (43 minutes downtime per month)
- RTO (Recovery Time Objective): < 5 minutes
- RPO (Recovery Point Objective): < 1 minute data loss

**Technical Requirements:**
- Primary region: `us-east1`
- Secondary region: `us-west1` 
- Database must maintain ACID compliance
- Real-time transaction processing
- Audit logging for compliance

**Constraints:**
- Budget conscious - avoid unnecessary duplication
- Automated failover (no manual intervention)
- Health checks and monitoring
- Rollback capability

Design the architecture including:
1. Service deployment strategy across regions
2. Database replication and failover approach
3. Load balancing and traffic routing
4. Monitoring and alerting setup
5. Failover triggers and automation
6. Testing strategy for disaster recovery

[📖 Answer](mock_1_answers.md#-section-3-problem-solving---answer)

---

## 🎯 Additional Topics to Review

- Cloud Spanner for global consistency
- Traffic Director for service mesh
- Cloud Endpoints and API Gateway
- Private Google Access and Private Service Connect
- Cloud NAT for outbound connectivity  
- Cloud CDN optimization strategies
- Shared VPC and network security
- Cloud KMS for encryption key management
- Binary Authorization for container security
- Organization policies and constraints

---

[🏠 Back to Main](../../../README.md) | [📋 All GCP Interviews](../../README.md)
