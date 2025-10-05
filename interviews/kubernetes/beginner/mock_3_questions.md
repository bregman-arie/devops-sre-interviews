# Kubernetes - Beginner Interview Mock #3

> **Difficulty:** Beginner  
> **Duration:** ~30 minutes  
> **Goal:** Assess your understanding of Kubernetes workloads, resource management, security basics, and operational commands.

---

## 🧠 Section 1: Core Questions

1. What is the difference between a **DaemonSet** and a **Deployment**? When would you use each? [📖 Answer](mock_3_answers.md#1-what-is-the-difference-between-a-daemonset-and-a-deployment-when-would-you-use-each)
2. Explain what **resource requests** and **resource limits** are and why they're important. [📖 Answer](mock_3_answers.md#2-explain-what-resource-requests-and-resource-limits-are-and-why-theyre-important)
3. What is a **ServiceAccount** and how does it relate to pod security? [📖 Answer](mock_3_answers.md#3-what-is-a-serviceaccount-and-how-does-it-relate-to-pod-security)
4. How do **Jobs** and **CronJobs** work in Kubernetes? What are their use cases? [📖 Answer](mock_3_answers.md#4-how-do-jobs-and-cronjobs-work-in-kubernetes-what-are-their-use-cases)
5. What are **Ingress** resources and how do they differ from Services? [📖 Answer](mock_3_answers.md#5-what-are-ingress-resources-and-how-do-they-differ-from-services)
6. How would you view the logs from all containers in a multi-container pod? [📖 Answer](mock_3_answers.md#6-how-would-you-view-the-logs-from-all-containers-in-a-multi-container-pod)
7. What command would you use to create a pod imperatively (without YAML) for quick testing? [📖 Answer](mock_3_answers.md#7-what-command-would-you-use-to-create-a-pod-imperatively-without-yaml-for-quick-testing)
8. What happens during a **rolling update** and how can you monitor its progress? [📖 Answer](mock_3_answers.md#8-what-happens-during-a-rolling-update-and-how-can-you-monitor-its-progress)

---

## ⚙️ Section 2: Scenario

**Scenario:**  
You need to deploy a logging agent that must run on every node in your Kubernetes cluster to collect logs from all pods. The agent should automatically be deployed to new nodes when they join the cluster.  
Explain what Kubernetes resource you would use and walk through the key configuration considerations.

[📖 Answer](mock_3_answers.md#️-section-2-scenario---answer)

---

## 🧩 Section 3: Problem-Solving

**Task:**  
A team is trying to run a batch job that processes data files, but they're having issues with the following Job configuration:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: data-processor
spec:
  template:
    spec:
      containers:
      - name: processor
        image: data-processor:v1
        command: ["python", "process_data.py"]
      restartPolicy: Always
```

The job keeps running indefinitely and creating new pods even after the processing completes successfully. Identify the issues and provide a corrected configuration that includes proper resource management and failure handling.

[📖 Answer](mock_3_answers.md#-section-3-problem-solving---answer)
