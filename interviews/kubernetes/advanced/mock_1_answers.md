# Kubernetes - Advanced Interview Mock #1 - Answers

---

## 🧠 Section 1: Core Questions - Answers

### 1. Describe how the Kubernetes scheduler selects a node for a Pod

**Phases (Scheduler Framework):** QueueSort → PreFilter → Filter → (Preemption if needed) → Score → NormalizeScore → Reserve → Permit → PreBind → Bind → PostBind / Unreserve on failure.

1. Unschedulable Pod enters queue (Priority & Fairness influences ordering).  
2. Filter phase eliminates nodes failing predicates (NodeUnschedulable, Taints/Tolerations, resource fit, NodeAffinity, PodAffinity/AntiAffinity, topology spread).  
3. If no nodes remain, preemption may try to evict lower-priority pods.  
4. Score phase ranks remaining nodes (LeastRequested, BalancedAllocation, ImageLocality, topology, custom plugins).  
5. Highest score wins; Bind updates `spec.nodeName`.  
6. Kubelet handles actual placement.

**PodTopologySpread vs (Anti)Affinity:**  
- (Anti)Affinity: Flexible but computationally expensive; label-based; can enforce co/anti-location.  
- PodTopologySpread: Declarative skew limits across topology domains (zone, hostname) with `maxSkew`; easier to express even distribution, better scaling.  
- Use topology spread for balanced spread; anti-affinity for strict separation (e.g., single replica per node).

### 2. Contrast DaemonSet, StatefulSet, Deployment

| Aspect | Deployment | StatefulSet | DaemonSet |
|--------|------------|-------------|-----------|
| Identity | Ephemeral | Stable ordinal + network ID | Node-linked |
| Storage | Usually shared / ephemeral | Persistent per replica (PVC templates) | Usually host resources / node agents |
| Rolling Update | Parallel w/ surge | Ordered (default) | Rolling (updateStrategy) |
| Use Cases | Stateless web/API | Databases, Kafka, ZooKeeper | Log shippers, CNI, monitoring agents |
| Scaling Semantics | Adjust replicas freely | Maintains identity on scale | Matches node count |

### 3. Explain Pod resource management

- **Requests:** Scheduler reservation; basis for bin-packing.  
- **Limits:** Enforced ceilings (CPU = throttling via CFS quota; Memory = OOMKill).  
- **QoS Classes:** Guaranteed (req=limit for all containers), Burstable (partial), BestEffort (none). Eviction priority: BestEffort → Burstable → Guaranteed.
- **CPU Throttling:** If CFS quota < actual demand; avoid extremely low limits causing latency.  
**Mitigation:** Right-size requests via profiling/VPA recommend; avoid limit for CPU when latency sensitive (or set >= request near demand); isolate noisy neighbors with LimitRanges + node labeling.

### 4. How do HPA, VPA, Cluster Autoscaler interact

- **HPA:** Scales pods horizontally based on metrics (CPU/memory/custom).  
- **VPA:** Recommends or adjusts container requests/limits (may restart pods in Auto mode).  
- **Cluster Autoscaler (CA):** Adds/removes nodes when pending pods can't schedule or nodes underutilized.  
**Interaction:** VPA right-sizes requests → HPA decisions become more accurate → CA provisions nodes if aggregate requests exceed capacity.  
**Anti-Patterns:** Running VPA in Auto mode on same workload HPA scales by CPU (changing request modifies utilization %); using tiny requests to force overpacking → throttling; large `maxReplicas` with no CA headroom.

### 5. Outline a secure baseline for running workloads

Layers: 
1. Pod Security Admission: Enforce `restricted` in prod (no privilege escalation, runAsNonRoot).  
2. NetworkPolicies: Default deny; allow namespace/pod scoped ingress & minimal egress (DNS, dependencies).  
3. Image Governance: Signed images (cosign), pinned digests, vulnerability threshold gates.  
4. Runtime Restrictions: seccomp=RuntimeDefault, drop ALL capabilities except required, read-only root FS, no hostPath, no privileged, no hostNetwork unless justified.  
5. RBAC: Least privilege service accounts; namespace isolation.  
6. Supply Chain: SBOM + provenance attestation; scanning in CI; admission enforcement.

### 6. Discuss etcd health, backup, and performance best practices

**Health:** Low latency disk (SSD), dedicated network path, correct quorum sizing (odd: 3 or 5).  
**Backups:** Regular `etcdctl snapshot save`; store off-cluster; validate restore; version compatibility checks.  
**Performance Tuning:** Defrag routinely (`etcdctl defrag`), compaction retention (reduce revisions), avoid very large objects (>1MB), limit CRD explosion.  
**Diagnosis of Slow API:** Check metrics: `apiserver_request_duration_seconds` high? Inspect `etcd_request_duration_seconds`, WAL fsync latency, backend commit duration; analyze goroutine dumps for blocking; network RTT to etcd.  
Enable audit sampling to ensure not bottlenecked by logging.

### 7. What are CSI dynamic provisioning workflows

- User creates PVC referencing a StorageClass.  
- External provisioner sees Pending claim → calls `CreateVolume` on CSI driver.  
- PV object created & bound to PVC.  
- If `volumeBindingMode: WaitForFirstConsumer`, binding is deferred until Pod scheduled → scheduler can consider topology (zones), preventing cross-zone volume issues.  
Supports expansion (if `allowVolumeExpansion: true`) and snapshots via snapshot CRDs.

### 8. Explain how to extend Kubernetes using CRDs + Controllers / Operators

**CRD:** Adds new resource type.  
**Controller Loop:** Watch desired (CR) + observed (child objects) → reconcile toward desired.  
**Idempotency:** Desired spec → deterministic manifests; patch/apply minimal diff.  
**Finalizers:** Block deletion until external cleanup done; remove when successful.  
**Status Conditions:** Communicate phases (Progressing, Ready, Degraded) with reason + observedGeneration.  
**Patterns:** Level-triggered, backoff on transient errors, event recording, metrics (reconcile duration, errors).

---

## ⚙️ Section 2: Scenario - Answer

### Challenges Mapped
1. Latency variability = noisy batch resource contention.  
2. PVC binding failures = zonal capacity / storage topology mismatch.  
3. Drift issues = imperative changes outside Git.  
4. API timeouts = high cardinality lists (CRDs) + inefficient clients.

### Plan
**Resource / Scheduling Isolation:**
- Separate node pools: `api` (taint `workload=latency:NoSchedule`) & `batch`.  
- API workloads: Guaranteed QoS (requests=limits) or elevated CPU shares; topology spread across zones.  
- Batch: Lower PriorityClass; allow preemption by API pods.  
- Apply PodTopologySpreadConstraints instead of broad anti-affinity for even distribution.

**Storage Topology & Capacity:**
- Use `WaitForFirstConsumer` StorageClasses to avoid pre-binding wrong zone.  
- Monitor per-zone PVC binding latency & free capacity metrics.  
- Add proactive capacity alerting and periodic defrag for cloud block storage fragmentation (where relevant).  
- For Kafka: StatefulSet with volume affinity labels; ensure balanced broker placement across zones.

**GitOps + Policy:**
- Argo CD / Flux for declarative sync; drift detection alerts.  
- Policy Layer: Kyverno/Gatekeeper for mandatory labels (owner, tier), enforce no `latest` tags.  
- Admission webhooks latency budgets (<2s) + fail-open for non-critical.

**API / Controller Performance:**
- Enable watch caching for CRDs; limit large list operations by pagination & label selectors.  
- Consolidate high-churn CRDs; avoid per-entity custom resources where ConfigMap suffices.  
- Increase controller concurrency for slow reconcilers after profiling.  
- Turn on metrics sampling dashboards: apiserver inflight requests, workqueue depth, etcd backend commit duration.

**Outcome:** Stable latency for APIs, fewer PVC binding failures, auditable drift-free config, improved API responsiveness.

---

## 🧩 Section 3: Problem-Solving - Answer

### 1. Critique

**Reliability / Scalability:**
- No readiness/liveness/startup probes → blind rollout & slow failure detection.
- No resource limits → risk of memory explosion; requests very low vs actual might cause throttling.
- No HPA → fixed capacity for variable batch load.

**Performance / Scheduling:**
- No anti-affinity; all pods might land on one node impacting throughput.  
- No topology spread; potential AZ imbalance.  
- Batch workload not identified; could interfere with latency-sensitive apps.

**Security:**
- No securityContext (runs as root, full capabilities).  
- Image tag `latest` (non-repeatable deployment).  
- No NetworkPolicy → unrestricted communication.

**Storage:**
- PVC assumed exists; no indication of access mode or binding mode; risk of single-zone PV causing scheduling failures.  
- Single shared PVC might not be RWX capable depending on class (e.g., block).  

**Governance / Observability:**
- Missing labels (team, tier, version).  
- No termination message policy or structured logging annotation.

### 2. Improved Manifest

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: reporting
  labels:
    pod-security.kubernetes.io/enforce: baseline
    team: analytics
---
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: batch-low
value: 1000
preemptionPolicy: PreemptLowerPriority
globalDefault: false
description: Low priority batch jobs.
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: reports
  namespace: reporting
  labels:
    app: reports
    app.kubernetes.io/version: 2.1.0
    workload: batch
spec:
  replicas: 3
  revisionHistoryLimit: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  selector:
    matchLabels:
      app: reports
  template:
    metadata:
      labels:
        app: reports
        workload: batch
    spec:
      priorityClassName: batch-low
      securityContext:
        runAsNonRoot: true
        fsGroup: 2000
      containers:
      - name: worker
        image: registry.local/reports@sha256:1234567890abcdefdeadbeefcafefade
        imagePullPolicy: IfNotPresent
        command: ["python", "worker.py"]
        env:
        - name: MODE
          value: batch
        resources:
          requests:
            cpu: 300m
            memory: 512Mi
          limits:
            cpu: 1000m
            memory: 1Gi
        securityContext:
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
          capabilities:
            drop: ["ALL"]
        readinessProbe:
          exec:
            command: ["/bin/sh", "-c", "test -f /data/READY"]
          initialDelaySeconds: 10
          periodSeconds: 10
        livenessProbe:
          httpGet:
            path: /healthz
            port: 8080
          initialDelaySeconds: 60
          periodSeconds: 30
        startupProbe:
          httpGet:
            path: /startupz
            port: 8080
          failureThreshold: 30
          periodSeconds: 5
        volumeMounts:
        - name: data
          mountPath: /data
        - name: tmp
          mountPath: /tmp
        terminationMessagePolicy: FallbackToLogsOnError
      volumes:
      - name: data
        persistentVolumeClaim:
          claimName: reports-cache
      - name: tmp
        emptyDir: {}
      topologySpreadConstraints:
      - maxSkew: 1
        topologyKey: kubernetes.io/hostname
        whenUnsatisfiable: DoNotSchedule
        labelSelector:
          matchLabels:
            app: reports
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchLabels:
                  app: reports
              topologyKey: kubernetes.io/hostname
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: reports-hpa
  namespace: reporting
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: reports
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
      - type: Percent
        value: 100
        periodSeconds: 60
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 20
        periodSeconds: 60
---
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: reports-pdb
  namespace: reporting
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: reports
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: reports-default-deny
  namespace: reporting
spec:
  podSelector: {}
  policyTypes: [Ingress, Egress]
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: reports-egress-allow-dns
  namespace: reporting
spec:
  podSelector:
    matchLabels:
      app: reports
  policyTypes: [Egress]
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          kube-system: "true"
    ports:
    - port: 53
      protocol: UDP
---
# Optional: StorageClass reminder (if PVC binding issues)
# kind: StorageClass
# allowVolumeExpansion: true
# volumeBindingMode: WaitForFirstConsumer
```

### 3. Rationale Summary
- Probes → Safe rollout & faster detection of wedged jobs.  
- Resource limits/requests → Predictable scheduling + cost control.  
- Image digest → Immutable deployments & provenance.  
- Security context & capability drop → Minimizes container breakout risk.  
- Topology spread + anti-affinity → Even distribution; resilience.  
- HPA → Elastic capacity for bursty batch load.  
- PDB → Maintains availability during node drains.  
- NetworkPolicies → Zero-trust baseline (default deny + explicit DNS).  
- PriorityClass → Allows preemption by more critical tiers elsewhere.  
- Namespace + labels → Isolation + governance + policy hooks.  
- StorageClass (comment) → Guides correct binding behavior across zones.

Collectively these changes improve stability, security, scalability, and operational safety for the batch workload.
