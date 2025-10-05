# Kubernetes - Intermediate Interview Mock #1

> **Difficulty:** Intermediate  
> **Duration:** ~45 minutes  
> **Goal:** Assess your understanding of Kubernetes resource management, cluster operations, advanced workloads, and troubleshooting skills.

---

## 🧠 Section 1: Core Questions

1. How do you **check for resource availability** in a cluster? Include both current usage and capacity planning. [📖 Answer](mock_1_answers.md#1-how-do-you-check-for-resource-availability-in-a-cluster-include-both-current-usage-and-capacity-planning)
2. Explain the difference between **ResourceQuota** and **LimitRange**. When would you use each? [📖 Answer](mock_1_answers.md#2-explain-the-difference-between-resourcequota-and-limitrange-when-would-you-use-each)
3. What is **Pod Disruption Budget (PDB)** and how does it help during cluster maintenance? [📖 Answer](mock_1_answers.md#3-what-is-pod-disruption-budget-pdb-and-how-does-it-help-during-cluster-maintenance)
4. How do **Node Affinity** and **Pod Affinity/Anti-Affinity** work? Provide examples of when you'd use each. [📖 Answer](mock_1_answers.md#4-how-do-node-affinity-and-pod-affinityanti-affinity-work-provide-examples-of-when-youd-use-each)
5. Explain **Horizontal Pod Autoscaler (HPA)** vs **Vertical Pod Autoscaler (VPA)** vs **Cluster Autoscaler**. How do they work together? [📖 Answer](mock_1_answers.md#5-explain-horizontal-pod-autoscaler-hpa-vs-vertical-pod-autoscaler-vpa-vs-cluster-autoscaler-how-do-they-work-together)
6. What are **Custom Resource Definitions (CRDs)** and how do they extend Kubernetes functionality? [📖 Answer](mock_1_answers.md#6-what-are-custom-resource-definitions-crds-and-how-do-they-extend-kubernetes-functionality)
7. How do you perform a **rolling update** with zero downtime? What strategies can you use? [📖 Answer](mock_1_answers.md#7-how-do-you-perform-a-rolling-update-with-zero-downtime-what-strategies-can-you-use)
8. Explain **Ingress Controllers** and how they differ from **LoadBalancer** services. [📖 Answer](mock_1_answers.md#8-explain-ingress-controllers-and-how-they-differ-from-loadbalancer-services)

---

## ⚙️ Section 2: Scenario

**Scenario:**  
Your e-commerce application is experiencing performance issues during peak hours. The application consists of a web frontend, API backend, and Redis cache. You've noticed that some pods are being killed due to resource constraints, while other nodes have available capacity. The application also needs to handle traffic spikes during flash sales.  

Design a comprehensive resource management strategy that includes:
- Resource allocation and limits
- Scaling strategies  
- Pod scheduling optimization
- High availability considerations

[📖 Answer](mock_1_answers.md#️-section-2-scenario---answer)

---

## 🧩 Section 3: Problem-Solving

**Task:**  
Your team deployed the following configuration, but they're experiencing issues with uneven load distribution and pods frequently being scheduled on the same nodes:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 6
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - name: web
        image: nginx:latest
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  selector:
    app: web-app
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
spec:
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
```

The issues reported are:
1. All pods sometimes get scheduled on 2-3 nodes instead of spreading across the cluster
2. During traffic spikes, response times increase significantly
3. When nodes go down for maintenance, too many pods are affected at once
4. Resource utilization is inconsistent across nodes

Identify the problems and provide an improved configuration that addresses pod distribution, scaling, and resilience issues.

[📖 Answer](mock_1_answers.md#-section-3-problem-solving---answer)
