# Kubernetes - Beginner Interview Mock #2

> **Difficulty:** Beginner  
> **Duration:** ~30 minutes  
> **Goal:** Assess your understanding of Kubernetes networking, storage, configuration, and troubleshooting fundamentals.

---

## 🧠 Section 1: Core Questions

1. What is the difference between **ClusterIP**, **NodePort**, and **LoadBalancer** service types? [📖 Answer](mock_2_answers.md#1-what-is-the-difference-between-clusterip-nodeport-and-loadbalancer-service-types)
2. Explain what **ConfigMaps** and **Secrets** are used for and how they differ. [📖 Answer](mock_2_answers.md#2-explain-what-configmaps-and-secrets-are-used-for-and-how-they-differ)
3. What is the role of **etcd** in a Kubernetes cluster? [📖 Answer](mock_2_answers.md#3-what-is-the-role-of-etcd-in-a-kubernetes-cluster)
4. How do **liveness probes** and **readiness probes** work, and why are they important? [📖 Answer](mock_2_answers.md#4-how-do-liveness-probes-and-readiness-probes-work-and-why-are-they-important)
5. What is a **PersistentVolume** and **PersistentVolumeClaim**? How do they relate to each other? [📖 Answer](mock_2_answers.md#5-what-is-a-persistentvolume-and-persistentvolumeclaim-how-do-they-relate-to-each-other)
6. How would you check the resource usage (CPU and memory) of nodes and pods? [📖 Answer](mock_2_answers.md#6-how-would-you-check-the-resource-usage-cpu-and-memory-of-nodes-and-pods)
7. What command would you use to get detailed information about a specific pod including events? [📖 Answer](mock_2_answers.md#7-what-command-would-you-use-to-get-detailed-information-about-a-specific-pod-including-events)
8. What is the purpose of **labels** and **selectors** in Kubernetes? [📖 Answer](mock_2_answers.md#8-what-is-the-purpose-of-labels-and-selectors-in-kubernetes)

---

## ⚙️ Section 2: Scenario

**Scenario:**  
Your application pods are running but users report they cannot access the application. The pods show as "Running" but the service endpoints show as "Not Ready".  
Describe the troubleshooting steps you would take to identify and resolve this issue.

[📖 Answer](mock_2_answers.md#️-section-2-scenario---answer)

---

## 🧩 Section 3: Problem-Solving

**Task:**  
A developer has created this Deployment but is experiencing issues:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - name: web-container
        image: nginx:1.20
        ports:
        - containerPort: 80
        env:
        - name: DATABASE_URL
          value: "postgresql://user:password@db-host:5432/mydb"
```

Identify at least 3 problems with this Deployment and explain how you would fix them. What Kubernetes best practices are being violated?

[📖 Answer](mock_2_answers.md#-section-3-problem-solving---answer)
