# Kubernetes - Beginner Interview Mock #5

> **Difficulty:** Beginner  
> **Duration:** ~30 minutes  
> **Goal:** Assess your understanding of Kubernetes security basics, pod lifecycle management, debugging techniques, and cluster administration fundamentals.

---

## 🧠 Section 1: Core Questions

1. What is **RBAC** (Role-Based Access Control) in Kubernetes and why is it important? [📖 Answer](mock_5_answers.md#1-what-is-rbac-role-based-access-control-in-kubernetes-and-why-is-it-important)
2. Explain the difference between **StatefulSet** and **Deployment**. When would you use each? [📖 Answer](mock_5_answers.md#2-explain-the-difference-between-statefulset-and-deployment-when-would-you-use-each)
3. What are **environment variables** and **volumes** in Kubernetes? How do they differ in terms of configuration management? [📖 Answer](mock_5_answers.md#3-what-are-environment-variables-and-volumes-in-kubernetes-how-do-they-differ-in-terms-of-configuration-management)
4. How do **HorizontalPodAutoscaler (HPA)** work and what metrics can trigger scaling? [📖 Answer](mock_5_answers.md#4-how-do-horizontalpodautoscaler-hpa-work-and-what-metrics-can-trigger-scaling)
5. What is the purpose of **annotations** vs **labels** in Kubernetes resources? [📖 Answer](mock_5_answers.md#5-what-is-the-purpose-of-annotations-vs-labels-in-kubernetes-resources)
6. How would you **copy files** to and from a running pod? [📖 Answer](mock_5_answers.md#6-how-would-you-copy-files-to-and-from-a-running-pod)
7. What command would you use to **port-forward** to access a pod locally for debugging? [📖 Answer](mock_5_answers.md#7-what-command-would-you-use-to-port-forward-to-access-a-pod-locally-for-debugging)
8. Explain what **multi-container pods** are and provide a common use case pattern. [📖 Answer](mock_5_answers.md#8-explain-what-multi-container-pods-are-and-provide-a-common-use-case-pattern)

---

## ⚙️ Section 2: Scenario

**Scenario:**  
Your company is deploying a new application that requires different access permissions for different teams. The development team needs read-only access to pods and services in the `development` namespace, while the operations team needs full access to all resources in both `development` and `production` namespaces.  
Design and explain the RBAC configuration needed to implement this security model.

[📖 Answer](mock_5_answers.md#️-section-2-scenario---answer)

---

## 🧩 Section 3: Problem-Solving

**Task:**  
A team deployed a StatefulSet for a database, but they're experiencing issues:

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: database
spec:
  replicas: 3
  serviceName: database-service
  selector:
    matchLabels:
      app: database
  template:
    metadata:
      labels:
        app: database
    spec:
      containers:
      - name: db
        image: postgres:13
        ports:
        - containerPort: 5432
        env:
        - name: POSTGRES_DB
          value: myapp
        - name: POSTGRES_USER
          value: admin
        - name: POSTGRES_PASSWORD
          value: secretpassword
```

The database pods keep restarting and losing data between restarts. Users report that database connections are unreliable. Identify the problems and provide a corrected configuration that addresses data persistence, security, and service discovery issues.

[📖 Answer](mock_5_answers.md#-section-3-problem-solving---answer)
