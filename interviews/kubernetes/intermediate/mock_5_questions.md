# Kubernetes - Intermediate Interview Mock #5

> **Difficulty:** Intermediate  
> **Duration:** ~45 minutes  
> **Goal:** Assess your understanding of Kubernetes cluster operations, performance optimization, cost management, and production readiness.

---

## 🧠 Section 1: Core Questions

1. How do you **optimize cluster resource utilization** and manage costs effectively? [📖 Answer](mock_5_answers.md#1-how-do-you-optimize-cluster-resource-utilization-and-manage-costs-effectively)
2. What strategies do you use for **cluster upgrades** while maintaining service availability? [📖 Answer](mock_5_answers.md#2-what-strategies-do-you-use-for-cluster-upgrades-while-maintaining-service-availability)
3. Explain **multi-cluster management** patterns and when you'd use each approach. [📖 Answer](mock_5_answers.md#3-explain-multi-cluster-management-patterns-and-when-youd-use-each-approach)
4. How do you implement **disaster recovery** and **high availability** for Kubernetes clusters? [📖 Answer](mock_5_answers.md#4-how-do-you-implement-disaster-recovery-and-high-availability-for-kubernetes-clusters)
5. What are the best practices for **performance tuning** Kubernetes clusters and applications? [📖 Answer](mock_5_answers.md#5-what-are-the-best-practices-for-performance-tuning-kubernetes-clusters-and-applications)
6. How do you handle **cluster security hardening** and compliance requirements? [📖 Answer](mock_5_answers.md#6-how-do-you-handle-cluster-security-hardening-and-compliance-requirements)
7. Explain **capacity planning** methodologies for Kubernetes environments. [📖 Answer](mock_5_answers.md#7-explain-capacity-planning-methodologies-for-kubernetes-environments)
8. How do you troubleshoot **performance bottlenecks** in production Kubernetes clusters? [📖 Answer](mock_5_answers.md#8-how-do-you-troubleshoot-performance-bottlenecks-in-production-kubernetes-clusters)

---

## ⚙️ Section 2: Scenario

**Scenario:**  
Your organization operates a large-scale Kubernetes platform serving multiple business units. The platform has grown organically and now faces several operational challenges:

- **Cluster sprawl**: 15+ clusters across different cloud providers and regions
- **Cost concerns**: Cloud bills have tripled in the last year 
- **Performance issues**: Applications experiencing latency during peak hours
- **Operational overhead**: Manual processes for updates, monitoring, and incident response
- **Compliance requirements**: New regulations requiring audit trails and data governance
- **Scaling challenges**: Some workloads require rapid scaling beyond current capacity

Design a comprehensive cluster optimization and governance strategy that addresses operational efficiency, cost management, performance, and compliance requirements while maintaining high availability and developer productivity.

[📖 Answer](mock_5_answers.md#️-section-2-scenario---answer)

---

## 🧩 Section 3: Problem-Solving

**Task:**  
Your production Kubernetes cluster is experiencing performance degradation and cost overruns. The monitoring dashboard shows concerning metrics, and the finance team is asking for immediate cost reduction while maintaining service quality.

**Current Cluster Metrics:**

```yaml
# Cluster Overview
Nodes: 20 (mixed instance types)
Total CPU: 320 cores
Total Memory: 1.2TB
Average CPU Utilization: 25%
Average Memory Utilization: 45%
Monthly Cost: $35,000

# Resource Allocation Issues
Over-provisioned Workloads: 60%
Under-provisioned Workloads: 15%
Properly-sized Workloads: 25%

# Performance Metrics
P95 Response Time: 800ms (target: 200ms)
Error Rate: 0.5% (target: 0.1%)
Pod Startup Time: 45s (target: 15s)

# Cost Breakdown
Compute: 70% ($24,500)
Storage: 15% ($5,250)  
Network: 10% ($3,500)
Other: 5% ($1,750)
```

**Sample Workload Configuration:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-service
spec:
  replicas: 10
  template:
    spec:
      containers:
      - name: api
        image: api-service:v1.2.3
        resources:
          requests:
            cpu: 1000m      # Often sits at 200m usage
            memory: 2Gi     # Often sits at 800Mi usage
          limits:
            cpu: 2000m
            memory: 4Gi
        env:
        - name: LOG_LEVEL
          value: DEBUG     # Excessive logging in production
        readinessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30  # Slow startup
          periodSeconds: 10
      nodeSelector:
        instance-type: m5.2xlarge  # Expensive instances

---
# Multiple similar services with similar over-provisioning
apiVersion: apps/v1
kind: Deployment
metadata:
  name: worker-service
spec:
  replicas: 8
  template:
    spec:
      containers:
      - name: worker  
        image: worker-service:v1.1.0
        resources:
          requests:
            cpu: 800m       # Actually uses 150m
            memory: 1.5Gi   # Actually uses 600Mi
          limits:
            cpu: 1600m
            memory: 3Gi
```

**Identified Issues:**
1. Massive over-provisioning of CPU and memory resources
2. Expensive instance types not matched to workload requirements
3. Poor application performance despite over-provisioning
4. No autoscaling configured for variable workloads
5. Inefficient container images and startup times
6. No cost visibility or chargeback mechanisms
7. Storage volumes provisioned but not optimized

Provide a comprehensive optimization plan that reduces costs by at least 40% while improving application performance and maintaining reliability.

[📖 Answer](mock_5_answers.md#-section-3-problem-solving---answer)
