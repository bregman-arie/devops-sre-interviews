# Kubernetes - Advanced Interview Mock #1

> **Difficulty:** Advanced  
> **Duration:** ~55 minutes  
> **Goal:** Assess depth in production-grade architecture, reliability engineering, security hardening, performance tuning, and extensibility.

---

## 🧠 Section 1: Core Questions

1. Describe how the Kubernetes **scheduler** selects a node for a Pod. Include Filter vs Score, preemption, and how PodTopologySpread differs from (anti)affinity. [📖 Answer](mock_1_answers.md#1-describe-how-the-kubernetes-scheduler-selects-a-node-for-a-pod)
2. Contrast **DaemonSet**, **StatefulSet**, and **Deployment** for operational guarantees, identity, storage, and rollout semantics. Provide real-world use cases. [📖 Answer](mock_1_answers.md#2-contrast-daemonset-statefulset-and-deployment)
3. Explain **Pod resource management**: requests vs limits, QoS classes, CPU throttling (CFS), and strategies to mitigate noisy neighbor issues. [📖 Answer](mock_1_answers.md#3-explain-pod-resource-management)
4. How do **Horizontal Pod Autoscaler**, **Vertical Pod Autoscaler**, and **Cluster Autoscaler** interact? What are anti-patterns when combining them? [📖 Answer](mock_1_answers.md#4-how-do-horizontal-pod-autoscaler-vertical-pod-autoscaler-and-cluster-autoscaler-interact)
5. Outline a secure baseline for running workloads: Pod Security Admission, NetworkPolicies, image governance, runtime restrictions (seccomp, capabilities, read-only FS). [📖 Answer](mock_1_answers.md#5-outline-a-secure-baseline-for-running-workloads)
6. Discuss **etcd** health, backup, and performance best practices. How would you diagnose slow API server responses suspected to be etcd-related? [📖 Answer](mock_1_answers.md#6-discuss-etcd-health-backup-and-performance-best-practices)
7. What are **CSI** dynamic provisioning workflows and how do `WaitForFirstConsumer` StorageClasses improve scheduling decisions? [📖 Answer](mock_1_answers.md#7-what-are-csi-dynamic-provisioning-workflows)
8. Explain how to extend Kubernetes using **CRDs + Controllers / Operators**. Include reconciliation patterns, finalizers, status conditions, and idempotency. [📖 Answer](mock_1_answers.md#8-explain-how-to-extend-kubernetes-using-crds--controllers--operators)

---

## ⚙️ Section 2: Scenario

**Scenario:**  
You operate a multi-team cluster. Workloads include latency-sensitive APIs, batch jobs, and a Kafka platform. Teams complain about: (1) unpredictable API latency during batch spikes, (2) failed PVC bindings in one availability zone, (3) difficulty auditing configuration drift, (4) occasional timeouts when listing large numbers of custom resources.

Design an improvement plan covering:
- Scheduling and resource isolation (affinity, topology spread, QoS)
- Storage topology & capacity planning (zonal imbalance)
- GitOps + policy enforcement for drift control
- API / controller performance tuning (cache, watch efficiency)

[📖 Answer](mock_1_answers.md#️-section-2-scenario---answer)

---

## 🧩 Section 3: Problem-Solving

**Task:**  
Given the following manifest, identify issues and produce an improved configuration.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: reports
spec:
  replicas: 3
  selector:
    matchLabels:
      app: reports
  template:
    metadata:
      labels:
        app: reports
    spec:
      containers:
      - name: worker
        image: registry.local/reports:latest
        command: ["python", "worker.py"]
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
        env:
        - name: MODE
          value: batch
        volumeMounts:
        - name: data
          mountPath: /data
      volumes:
      - name: data
        persistentVolumeClaim:
          claimName: reports-cache
```

Problems may involve: rollout safety, scaling, missing probes, storage assumptions, security posture, scheduling inefficiency, image governance, and cost control.

Provide:  
1. A categorized critique  
2. An improved manifest implementing: probes, limits, topology spread, anti-affinity, securityContext, pinned image digest, HPA, PDB, NetworkPolicy, StorageClass hint (if needed)  
3. Rationale for major changes

[📖 Answer](mock_1_answers.md#-section-3-problem-solving---answer)
