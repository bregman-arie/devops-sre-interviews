# Kubernetes - Beginner Interview Mock #4

> **Difficulty:** Beginner  
> **Duration:** ~30 minutes  
> **Goal:** Assess your understanding of Kubernetes troubleshooting, networking fundamentals, workload management, and cluster operations.

---

## 🧠 Section 1: Core Questions

1. What is the difference between **kubectl apply** and **kubectl create**? When would you use each? [📖 Answer](mock_4_answers.md#1-what-is-the-difference-between-kubectl-apply-and-kubectl-create-when-would-you-use-each)
2. Explain what **Init Containers** are and provide a use case where they would be helpful. [📖 Answer](mock_4_answers.md#2-explain-what-init-containers-are-and-provide-a-use-case-where-they-would-be-helpful)
3. What is **DNS** in Kubernetes and how do services communicate with each other? [📖 Answer](mock_4_answers.md#3-what-is-dns-in-kubernetes-and-how-do-services-communicate-with-each-other)
4. How do you **drain a node** and why would you need to do this? [📖 Answer](mock_4_answers.md#4-how-do-you-drain-a-node-and-why-would-you-need-to-do-this)
5. What are **Taints and Tolerations** and how do they affect pod scheduling? [📖 Answer](mock_4_answers.md#5-what-are-taints-and-tolerations-and-how-do-they-affect-pod-scheduling)
6. How would you check which **image version** is currently running in a deployment? [📖 Answer](mock_4_answers.md#6-how-would-you-check-which-image-version-is-currently-running-in-a-deployment)
7. What's the difference between **kubectl delete** and **kubectl patch** for updating resources? [📖 Answer](mock_4_answers.md#7-whats-the-difference-between-kubectl-delete-and-kubectl-patch-for-updating-resources)
8. Explain what happens when you run **kubectl get pods -o wide** vs **kubectl get pods**. [📖 Answer](mock_4_answers.md#8-explain-what-happens-when-you-run-kubectl-get-pods--o-wide-vs-kubectl-get-pods)

---

## ⚙️ Section 2: Scenario

**Scenario:**  
Your team has deployed a web application, but some users report intermittent connection timeouts. You notice that sometimes the application works fine, but other times it's unreachable. The pods are showing as "Running" but you suspect there might be networking or readiness issues.  
Walk through your systematic troubleshooting approach to identify and resolve this intermittent connectivity problem.

[📖 Answer](mock_4_answers.md#️-section-2-scenario---answer)

---

## 🧩 Section 3: Problem-Solving

**Task:**  
A developer deployed the following configuration but the application isn't starting properly:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - name: web-container
        image: myapp:latest
        ports:
        - containerPort: 8080
        env:
        - name: DATABASE_HOST
          value: "db-service"
        - name: CACHE_HOST  
          value: "redis-service"
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
    targetPort: 8080
  type: ClusterIP
```

The pods start but immediately crash with "connection refused" errors when trying to connect to the database and cache. The database and cache services exist and are working. What's the most likely issue and how would you fix it using Init Containers?

[📖 Answer](mock_4_answers.md#-section-3-problem-solving---answer)
