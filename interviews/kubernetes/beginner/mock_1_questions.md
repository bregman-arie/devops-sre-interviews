# Kubernetes - Beginner Interview Mock #1

> **Difficulty:** Beginner  
> **Duration:** ~30 minutes  
> **Goal:** Assess your understanding of basic Kubernetes architecture, concepts, and common commands.

---

## 🧠 Section 1: Core Questions

1. What is Kubernetes, and what problems does it solve compared to running containers manually? [📖 Answer](mock_1_answers.md#1-what-is-kubernetes-and-what-problems-does-it-solve-compared-to-running-containers-manually)
2. Explain the difference between a **Pod**, a **Deployment**, and a **Service**. [📖 Answer](mock_1_answers.md#2-explain-the-difference-between-a-pod-a-deployment-and-a-service)
3. What is the role of the **kubelet** and **kube-apiserver**? [📖 Answer](mock_1_answers.md#3-what-is-the-role-of-the-kubelet-and-kube-apiserver)
4. What is a **Namespace**, and why would you use it? [📖 Answer](mock_1_answers.md#4-what-is-a-namespace-and-why-would-you-use-it)
5. What's the difference between **ReplicaSet** and **ReplicationController**? [📖 Answer](mock_1_answers.md#5-whats-the-difference-between-replicaset-and-replicationcontroller)
6. How do you scale a Deployment in Kubernetes? [📖 Answer](mock_1_answers.md#6-how-do-you-scale-a-deployment-in-kubernetes)
7. What command would you use to view all Pods running in all namespaces? [📖 Answer](mock_1_answers.md#7-what-command-would-you-use-to-view-all-pods-running-in-all-namespaces)
8. What happens when a Pod crashes — how does Kubernetes handle it? [📖 Answer](mock_1_answers.md#8-what-happens-when-a-pod-crashes--how-does-kubernetes-handle-it)

---

## ⚙️ Section 2: Scenario

**Scenario:**  
You deployed an application to your cluster, but it's stuck in `CrashLoopBackOff`.  
Walk through how you'd troubleshoot this issue using `kubectl` commands.  
What steps would you take to identify the root cause and fix it?

[📖 Answer](mock_1_answers.md#️-section-2-scenario---answer)

---

## 🧩 Section 3: Problem-Solving

**Task:**  
A team member created this YAML and applied it:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 8080
```

The team member notices they can't access the nginx server from outside the cluster. What's wrong with this Pod definition, and how would you fix it? What additional Kubernetes resources would you need to create?

[📖 Answer](mock_1_answers.md#-section-3-problem-solving---answer)
