# Kubernetes - Intermediate Interview Mock #2

> **Difficulty:** Intermediate  
> **Duration:** ~45 minutes  
> **Goal:** Assess your understanding of Kubernetes networking, security, monitoring, and advanced troubleshooting techniques.

---

## 🧠 Section 1: Core Questions

1. Explain **Kubernetes networking model** and how pod-to-pod communication works across nodes. [📖 Answer](mock_2_answers.md#1-explain-kubernetes-networking-model-and-how-pod-to-pod-communication-works-across-nodes)
2. What are **Network Policies** and how do they enhance cluster security? Provide examples. [📖 Answer](mock_2_answers.md#2-what-are-network-policies-and-how-do-they-enhance-cluster-security-provide-examples)
3. How do **RBAC (Role-Based Access Control)** components work together? Explain Roles vs ClusterRoles. [📖 Answer](mock_2_answers.md#3-how-do-rbac-role-based-access-control-components-work-together-explain-roles-vs-clusterroles)
4. What are **Admission Controllers** and how do they impact cluster security and governance? [📖 Answer](mock_2_answers.md#4-what-are-admission-controllers-and-how-do-they-impact-cluster-security-and-governance)
5. Explain **Service Mesh** concepts and how they integrate with Kubernetes (Istio/Linkerd). [📖 Answer](mock_2_answers.md#5-explain-service-mesh-concepts-and-how-they-integrate-with-kubernetes-istiolinkerd)
6. How do you implement **monitoring and observability** in Kubernetes clusters? [📖 Answer](mock_2_answers.md#6-how-do-you-implement-monitoring-and-observability-in-kubernetes-clusters)
7. What are **Taints and Tolerations** and how do they affect pod scheduling? [📖 Answer](mock_2_answers.md#7-what-are-taints-and-tolerations-and-how-do-they-affect-pod-scheduling)
8. How do you troubleshoot **network connectivity issues** between pods and services? [📖 Answer](mock_2_answers.md#8-how-do-you-troubleshoot-network-connectivity-issues-between-pods-and-services)

---

## ⚙️ Section 2: Scenario

**Scenario:**  
Your company is migrating to a multi-tenant Kubernetes cluster where different teams (frontend, backend, data) need isolated environments. Security is a major concern - teams should not be able to access each other's resources or network traffic. Additionally, you need to implement proper monitoring and ensure compliance with organizational security policies.  

Design a comprehensive security and governance strategy that includes:
- Network isolation between teams
- RBAC configuration for different user roles  
- Resource access controls and quotas
- Security scanning and policy enforcement
- Monitoring and audit logging

[📖 Answer](mock_2_answers.md#️-section-2-scenario---answer)

---

## 🧩 Section 3: Problem-Solving

**Task:**  
A team reports that their application pods cannot communicate with the database service, but the database pods are running fine. The infrastructure uses a service mesh (Istio) and has strict network policies in place. You've been asked to troubleshoot and fix the connectivity issue.

**Current Configuration:**

```yaml
# Database Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres-db
  namespace: database
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
        version: v1
    spec:
      containers:
      - name: postgres
        image: postgres:13
        ports:
        - containerPort: 5432
        env:
        - name: POSTGRES_DB
          value: appdb
        - name: POSTGRES_USER
          value: dbuser
        - name: POSTGRES_PASSWORD
          value: dbpass

---
# Database Service  
apiVersion: v1
kind: Service
metadata:
  name: postgres-service
  namespace: database
spec:
  selector:
    app: postgres
  ports:
  - port: 5432
    targetPort: 5432
  type: ClusterIP

---
# Application Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  namespace: frontend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
        version: v1
    spec:
      containers:
      - name: app
        image: myapp:latest
        env:
        - name: DB_HOST
          value: postgres-service.database.svc.cluster.local
        - name: DB_PORT
          value: "5432"

---
# Network Policy
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: database-policy
  namespace: database
spec:
  podSelector:
    matchLabels:
      app: postgres
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: backend
    ports:
    - protocol: TCP
      port: 5432
```

**Reported Issues:**
1. Web application cannot connect to database
2. Connection timeouts when trying to reach postgres-service
3. Istio sidecar injection is enabled for both namespaces
4. Network policies are blocking unexpected traffic

Identify the connectivity problems and provide a corrected configuration with proper security controls.

[📖 Answer](mock_2_answers.md#-section-3-problem-solving---answer)
