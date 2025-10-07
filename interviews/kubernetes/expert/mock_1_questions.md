# Kubernetes - Expert Interview Mock #1

> **Difficulty:** Expert  
> **Duration:** ~60 minutes  
> **Goal:** Evaluate deep expertise in control plane internals, scalability, security, networking, multi-cluster architecture, and production operations.

---

## 🧠 Section 1: Core Questions

1. Walk through the full lifecycle of a Pod creation request: from `kubectl apply` to the Pod running on a node. Include the roles of the API server, admission chain (mutating + validating), etcd, controllers, and scheduler. [📖 Answer](mock_1_answers.md#1-walk-through-the-full-lifecycle-of-a-pod-creation-request)
2. Compare Kubernetes networking data paths using: (a) kube-proxy iptables mode, (b) kube-proxy IPVS mode, and (c) eBPF-based dataplanes like Cilium. Discuss performance, observability, and operational trade-offs. [📖 Answer](mock_1_answers.md#2-compare-kubernetes-networking-data-paths)
3. Design a multi-tenant security model for a shared regulated cluster. Cover: Namespaces, RBAC, NetworkPolicies, Pod Security Admission levels, seccomp/AppArmor, RuntimeClass, image provenance, and policy engines (Kyverno vs OPA Gatekeeper). [📖 Answer](mock_1_answers.md#3-design-a-multi-tenant-security-model)
4. Explain the CSI architecture (provisioner, attacher, node plugin, snapshotter). Contrast ephemeral vs persistent volumes; discuss dynamic provisioning, expansion, snapshots, RWX scaling patterns (e.g., CephFS, EFS, NFS), and performance tuning strategies. [📖 Answer](mock_1_answers.md#4-explain-the-csi-architecture)
5. Outline the Operator (controller) reconcile loop design. How do you ensure idempotency, handle partial failure, exponential backoff, finalizers, and status conditions for progressive delivery? [📖 Answer](mock_1_answers.md#5-outline-the-operator-reconcile-loop-design)
6. Compare multi-cluster strategies: Cluster API + GitOps, service mesh (Istio multi-primary), Kubernetes Federation (KubeFed), and external DNS + Global Load Balancers. Discuss service discovery, failover, latency, and complexity trade-offs. [📖 Answer](mock_1_answers.md#6-compare-multi-cluster-strategies)
7. What are key performance + scalability tuning levers for large (>5k node / >250k Pod) clusters? Include: API server flags, etcd compaction/defrag, watch reduction, CRD explosion management, controller concurrency, admission latency, and server-side apply impacts. [📖 Answer](mock_1_answers.md#7-what-are-key-performance--scalability-tuning-levers)
8. Explain the Kubernetes scheduling extension surfaces: scheduler profiles, framework plugin phases, scheduling queues, preemption, and extenders. When would you build a score vs filter vs reserve plugin? Provide an example use case (e.g., GPU fragmentation avoidance). [📖 Answer](mock_1_answers.md#8-explain-the-kubernetes-scheduling-extension-surfaces)

---

## ⚙️ Section 2: Scenario

**Scenario:**  
You run a hybrid (on-prem + cloud) Kubernetes platform for a fintech company. Ultra low-latency trading microservices (p99 < 5ms) share clusters with bursty batch risk analytics jobs. Recently:
- Trading workloads see latency spikes during batch job fan-outs
- Node CPU throttling and last-level cache contention increase
- Horizontal scaling adds pods but tail latency doesn’t improve
- Compliance requires immutable audit of scheduling + config changes

Design a solution covering:
- Workload isolation (resource classes, node pools, topology)
- Scheduling & QoS strategies to protect latency-sensitive services
- Runtime and kernel-level tuning (CNI, CPU pinning, hugepages, IRQ isolation)
- Autoscaling strategy (HPA + VPA + Cluster Autoscaler + batch queueing)
- Multi-cluster or cell-based design considerations
- Observability + continuous latency SLO enforcement

[📖 Answer](mock_1_answers.md#️-section-2-scenario---answer)

---

## 🧩 Section 3: Problem-Solving

**Task:**  
Your platform team deployed the following manifest for a critical payments API. Incidents report intermittent latency spikes, pod restarts, and security audit findings.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: payments-api
spec:
  replicas: 6
  selector:
    matchLabels:
      app: payments-api
  template:
    metadata:
      labels:
        app: payments-api
    spec:
      hostNetwork: true
      containers:
      - name: api
        image: registry.local/payments/api:latest
        ports:
        - containerPort: 8443
        securityContext:
          privileged: true
        resources:
          requests:
            cpu: 250m
            memory: 256Mi
        volumeMounts:
        - name: certs
          mountPath: /etc/certs
        - name: socket
          mountPath: /var/run/app
      volumes:
      - name: certs
        hostPath:
          path: /etc/ssl
      - name: socket
        emptyDir: {}
---
apiVersion: v1
kind: Service
metadata:
  name: payments-api
spec:
  selector:
    app: payments-api
  ports:
  - port: 443
    targetPort: 8443
  type: NodePort
```

Issues to identify may include: security hardening gaps, scheduling/affinity problems, poor resource isolation, lack of network policy, no rollout safety, etc.

Provide: 
1. A detailed critique (grouped by category)  
2. An improved multi-document manifest implementing: security context hardening, PodSecurity admission compliance, controlled egress, PDB, HPA, anti-affinity, topology spread, NetworkPolicy, proper Service exposure (Ingress or Gateway), resource limits + SLO-aware probes, image pinning, and supply chain integrity hints.  
3. Brief explanation of how each change mitigates risk or improves reliability.

[📖 Answer](mock_1_answers.md#-section-3-problem-solving---answer)
