# Kubernetes - Expert Interview Mock #1 - Answers

---

## 🧠 Section 1: Core Questions - Answers

### 1. Walk through the full lifecycle of a Pod creation request

**High-Level Flow:**
`kubectl apply` → Client builds manifest → REST call to kube-apiserver → AuthN → AuthZ (RBAC / ABAC / Webhook) → Admission (Mutating → Validating) → Object persisted (etcd via API server) → Controllers & Scheduler react via watches → Scheduler assigns Node (bind) → Kubelet on target Node pulls PodSpec via watch → Volume plugins attach/mount → CNI allocates IP + programs networking → Container runtime (CRI) creates sandbox + containers → Readiness gates/probes pass → Service endpoints updated → Pod becomes Ready.

**Detailed Phases:**
1. Client-side: kubectl may do server-side apply; diff/patch logic stored as managedFields for SSA conflict detection.
2. API Front Door: Authentication (x509, OIDC, webhook). Failure stops request.
3. Authorization: RBAC evaluation (verb, group, resource, namespace). Deny → 403.
4. Admission Chain:
   - MutatingAdmission: Adds sidecars (Istio), injects defaults (PSPs replaced by Pod Security Admission levels), image pull secrets, resource defaults.
   - ValidatingAdmission: Enforces policies (OPA Gatekeeper/Kyverno), disallows privileged fields.
5. Storage: kube-apiserver writes object to etcd (primary key: /registry/pods/<ns>/<name>). Write acknowledged only after quorum commit (Raft). ResourceVersion assigned.
6. Watches: Controllers (ReplicaSet, Deployment) and custom controllers maintain informers (local caches + delta FIFO). Scheduler caches unscheduled pods.
7. Scheduling Cycle: Filter phase prunes nodes (resources, taints, NodeAffinity, PodAffinity/AntiAffinity, topology spread). Score phase ranks (e.g., LeastAllocated, ImageLocality). Reserve/Permit/PreBind plugin extension points. Binding API updates pod.spec.nodeName.
8. Kubelet: Watches assigned pods; creates Pod sandbox via CRI (containerd / CRI-O); pulls images; sets up cgroups, namespaces.
9. Storage: CSI Node plugin mounts volumes (NodePublishVolume); for dynamic provisioning the external-provisioner already created PVC/PV before scheduling if Bound.
10. Networking: CNI ADD assigns Pod IP; kube-proxy or eBPF dataplane programs service VIPs; NetworkPolicy engine (Calico, Cilium) attaches ACLs.
11. Lifecycle: Init containers run sequentially; app containers start; readiness/liveness probes feed kubelet → apiserver status updates.
12. Endpoint Propagation: Endpoints/EndpointSlice controllers add Pod IP to Service; kube-proxy updates rules; load balancers adjust.

**Key Reliability Concepts:** idempotent reconcile, eventual consistency with watch caches, optimistic concurrency via ResourceVersion.

---

### 2. Compare Kubernetes networking data paths

| Aspect | kube-proxy iptables | kube-proxy IPVS | eBPF (e.g., Cilium) |
|--------|--------------------|-----------------|---------------------|
| Programming Model | Large linear chains | Kernel hash tables | eBPF maps + dynamic programs |
| Scale Behavior | Rule explosion O(N services * ports) | Efficient at 10k+ services | Highly scalable; per-packet programmable |
| Latency | DNAT per hop; traversal overhead | Lower than iptables | Lowest; single-pass, XDP capable |
| Update Cost | Full chain rewrites | Incremental | Incremental, JIT optimized |
| Observability | Limited (conntrack + tcpdump) | Similar | Rich (flow logs, latency histograms) |
| Features | Basic ClusterIP/NodePort | Same + session persistence | L7 aware (if Envoy integration), NetworkPolicy, encryption, service mesh-lite |
| NetworkPolicy | Implemented by separate engine | Same | Native eBPF enforcement (no iptables dependency) |
| Overhead | Higher CPU under churn | Moderate | Lowest under high churn |
| Complexity | Low | Medium | Higher (kernel/eBPF knowledge) |

**Trade-offs:** eBPF improves performance & visibility but raises operational complexity and kernel dependency constraints. IPVS better than iptables for large service counts. Iptables still simplest for small clusters.

---

### 3. Design a multi-tenant security model

**Layers:**
1. Namespaces per tenant/environment (dev/stage/prod separated) + hierarchical logical grouping via labels.
2. RBAC: Principle of least privilege; separate CRUD on CRDs vs core; restrict wildcard verbs/resources.
3. Pod Security Admission: Enforce Baseline in dev, Restricted in prod (no privileged, no host namespaces, controlled capabilities, mandatory seccompProfile, runAsNonRoot).
4. NetworkPolicies: Default deny ingress & egress; explicit namespaceSelector + podSelector allowlists; restrict egress to DNS + required services; layered per tier.
5. Runtime Hardening: Drop ALL capabilities except needed (e.g., NET_BIND_SERVICE); read-only root FS; seccomp (RuntimeDefault), AppArmor profiles; disallow hostPath; use RuntimeClass for gVisor or Kata for untrusted workloads.
6. Image Security: Only pull from signed registry; enforce cosign signatures via Kyverno/Gatekeeper; disallow :latest tags; vulnerability scanning (Trivy/Clair) gate.
7. Policy Engine: Kyverno for mutation + policy authoring simplicity; Gatekeeper for complex Rego logic; choose one primary to reduce overlap; central policy bundle versioned (GitOps).
8. Supply Chain: SBOM attestation (CycloneDX), provenance attestations (SLSA), admission verifying transparency log.
9. Audit: Enable audit policy with segregated sinks; immutable storage (WORM) for compliance.
10. Resource Quotas + LimitRanges to prevent noisy neighbor abuse.

---

### 4. Explain the CSI architecture

**Components:**
- External-Provisioner: Watches PVCs -> triggers CreateVolume via CSI controller plugin.
- External-Attacher: Handles ControllerPublish/UnpublishVolume for block volumes.
- External-Snapshotter: Manages VolumeSnapshot & VolumeSnapshotContent objects.
- External-Resizer: Handles expansion requests.
- Node Plugin (DaemonSet): NodePublishVolume mounts; NodeStageVolume prepares staging path.
- Controller Plugin (Deployment/StatefulSet): Control-plane operations (Create/Delete, ControllerExpand, Snapshot).

**Ephemeral vs Persistent:** Ephemeral inline volumes live only for Pod lifecycle; no reclaim. Persistent volumes bound via PVC objects; support reclaim policies (Retain/Delete/Recycle).

**Dynamic Provisioning:** StorageClass defines provisioner + parameters (fstype, iops). PVC triggers asynchronous provisioning; scheduler waits for bound claim if `WaitForFirstConsumer` improving topology alignment.

**Snapshots:** CSI snapshot API creates CRDs; used for restore workflows & backups (incremental for some backends).

**RWX Scaling:** Use distributed filesystems (CephFS, EFS, NFS, Portworx shared vols). Performance tuning: mount options (noatime, rsize/wsize), NodeLocal DNS to reduce latency, volume topology awareness.

**Performance Tuning:**
- Enable volume expansion & ephemeral local SSD for write-heavy caches.
- Use separate StorageClasses for latency tiers.
- Monitor pvc_fs_inodes_free, volume IOPS, throttling from cloud provider.

---

### 5. Outline the Operator reconcile loop design

**Principles:** Declarative desired vs observed state; idempotent; level-triggered.

**Loop Steps:**
1. Fetch CR instance from cache; deep copy.
2. Validate spec (syntactic + semantic) early; set Condition=Invalid if needed.
3. Derive desired child objects (Deployments, Services, Secrets) as pure functions.
4. Compare actual (Lister cache) vs desired; create/update/patch/delete minimal diff.
5. Handle partial failures with exponential backoff (rate-limiter workqueue).
6. Status Update: Set Conditions (Available, Progressing, Degraded) with lastTransitionTime + observedGeneration.
7. Finalizers: Add on create to ensure cleanup (e.g., external cloud resource) before deletion; remove only after success.
8. Requeue triggers: Spec changes (resourceVersion), periodic resync (safety), external event channels.

**Idempotency:** All mutations keyed deterministically; use controller-runtime client Patch with merge strategy; avoid storing transient state in annotations unless versioned.

**Error Handling:** Distinguish transient (retry) vs terminal (mark Condition). Use structured events.

**Progressive Delivery:** Canary fields -> create parallel Deployment (weight shift) -> update status.

---

### 6. Compare multi-cluster strategies

| Strategy | Strengths | Weaknesses | Use Case |
|----------|-----------|-----------|----------|
| Cluster API + GitOps | Reproducible lifecycle, infra as code | Operational learning curve | Standardized platform provisioning |
| Service Mesh Multi-Primary (Istio) | Cross-cluster mTLS, locality-aware routing, traffic shifting | Complexity, control plane overhead | Resilient multi-region failover & traffic control |
| KubeFed (Federation) | Federated APIs & propagation policies | Limited adoption, complexity | Need central governance of object distribution |
| Global DNS + LB (Route53, GSLB) | Simple, decoupled, proven | Coarse routing, health granularity | Active-active region routing, low complexity |

**Decision Axes:** Failure isolation, latency routing, consistency model, cognitive load, tooling ecosystem alignment.

---

### 7. What are key performance & scalability tuning levers

- API Server: `--max-mutating-requests-inflight`, `--max-requests-inflight`, enable Priority & Fairness; optimize admission webhooks latency (timeoutSeconds < 5, failurePolicy=Fail for critical, Ignore otherwise). Shard webhooks by object type.
- Etcd: Proper compaction cadence (e.g., every 5m), defragment off-peak, SSD + low latency network, separate etcd clusters for control-plane vs high-churn CRDs if needed (or minimize CRD cardinality instead).
- Watch Reduction: Prefer watch over list+poll; use efficient field/label selectors; avoid high-cardinality labels.
- CRD Explosion: Aggregate status objects; avoid per-entity CRDs if >100k; consider packing arrays into a single CR when appropriate.
- Controller Concurrency: Tune workers (controller-manager flags) to match API throughput; avoid thundering herds with jitter.
- Scheduler: Increase parallelism (`--parallelism`), enable PodTopologySpread instead of custom anti-affinity when possible.
- Server-Side Apply: Manage managedFields size; prune large field sets (e.g., endpoint slices) from apply sources; use PATCH for high-churn.
- Resource Quotas: Keep count quotas low cardinality to reduce admission load.
- Network: Use eBPF for lower per-packet cost; enable conntrack tuning for high flows.
- Node OS: Use CRI with tuned cgroups v2; disable unnecessary kernel modules.

Monitoring: apiserver_request_duration_seconds, etcd_disk_wal_fsync_duration_seconds, goroutine counts, workqueue depth, admission webhook latency histograms.

---

### 8. Explain the Kubernetes scheduling extension surfaces

**Scheduler Framework Phases:** QueueSort → PreFilter → Filter → PostFilter → Score → Reserve → Permit → PreBind → Bind → PostBind → Unreserve.

**Plugins:**
- Filter: Reject nodes lacking GPU, memory, topology constraints.
- Score: Rank nodes to minimize fragmentation (e.g., pack vs spread).
- Reserve: Claim resources temporarily before binding (transaction-like).
- Permit: Gate binding on external approval (batch gating, gang scheduling).
- Extender: Legacy HTTP extender for custom logic (less preferred now).

**When to Choose:**
- Filter plugin if you must exclude nodes early (capability enforcement).
- Score plugin for placement optimization (e.g., GPU memory defrag, NUMA locality).
- Reserve for temporary resource holds before expensive external allocation.
- Permit for gang scheduling (ML jobs all-or-nothing start) or quota fairness.

**GPU Fragmentation Example:** Filter nodes lacking required GPU type & memory; Score nodes to prefer ones with largest contiguous free slice; Reserve to mark slice usage; Permit to batch multi-pod job start; fallback if deadline exceeded.

---

## ⚙️ Section 2: Scenario - Answer

**Goals:** Protect ultra-low-latency trading from noisy neighbors, ensure deterministic CPU/cache behavior, preserve auditability, and enable elastic batch throughput.

### 1. Workload Isolation
- Dedicated node pools: `trading` (pinned cores, high clock, low NUMA distance) vs `batch` (general purpose, cheaper). Use taints `dedicated=trading:NoSchedule` + tolerations.
- CPU Manager static policy on trading nodes + guaranteed QoS (requests=limits) for trading pods to avoid throttling.
- HugePages for certain latency-sensitive memory paths if applicable.
- TopologyManager `single-numa-node` for predictable memory locality.

### 2. Scheduling & QoS Strategies
- Trading pods: Guaranteed QoS, explicit CPU sets (cpuset cgroup allocation via static policy), priorityClass high (but not system-cluster-critical). Batch: BestEffort or Burstable with lower priority; enable pod preemption (preemptionPolicy=PreemptLowerPriority) to evict batch under contention.
- Use PodTopologySpread across zones & hosts for resilience.
- Optionally deploy custom scheduler plugin to penalize cache contention (LLC occupancy metrics via NodeFeatureDiscovery + scoring).

### 3. Runtime & Kernel Tuning
- Isolate IRQ affinity away from trading cores; configure RPS/XPS for NIC queues.
- Use eBPF dataplane (Cilium) for lower latency; enable pod-to-pod encryption only on batch nodes if compliance permits to reduce overhead where critical.
- Disable powersave (governor=performance) on trading nodes.
- Enable `--proc/sys/net/core/somaxconn` tuning if high connection rates; tune TCP small queues.
- Consider DPDK/AF_XDP user-space networking if extreme latency path.

### 4. Autoscaling Strategy
- Trading: HPA on custom latency SLO metric (p99 < 5ms) + floor of replicas; VPA in recommendation mode only (avoid restarts).
- Batch: Queue-backed (KEDA or custom controller) scaling off job backlog; scale to zero allowed.
- Cluster Autoscaler: Separate node groups with min capacity for trading (no scale-to-zero) and elastic for batch.
- Use Priority & Fairness to ensure API server not saturated by batch job surges.

### 5. Multi-Cluster / Cell Design
- Cell-based partitioning: Independent clusters per region/cell for blast radius containment. Global ingress / DNS performs latency-based routing.
- Trading cluster kept lean (fewer CRDs, fewer webhooks). Batch clusters handle heavy CRD/operator footprint.

### 6. Observability & SLO Enforcement
- Collect eBPF-based continuous profiling + per-pod TCP latency histograms.
- RED + USE metrics dashboards segregated per workload class.
- Alert on: p99 latency > 5ms 3 out of 5 minutes, CPU throttling > 1%, runqueue length per core > threshold, cache miss surge.
- Admission webhook to reject trading pod spec changes lacking change ticket annotation (immutable infra guardrails).
- Tamper-evident audit log sink with hash chaining.

**Outcome:** Trading latency shielded via dedicated, deterministically scheduled resources; batch elasticity preserved without cache contention; compliance & audit reinforced.

---

## 🧩 Section 3: Problem-Solving - Answer

### 1. Critique (Grouped)

**Security:**
- `privileged: true` unnecessary; host escape risk.
- `hostNetwork: true` broad exposure, bypasses CNI NetworkPolicies.
- `hostPath` to `/etc/ssl` risks exfil + mutation of host certs.
- No `runAsNonRoot`, `readOnlyRootFilesystem`, capabilities drop, seccomp profile.
- Image tag `latest` prevents provenance & rollback certainty.
- No NetworkPolicy (open east-west & egress).

**Reliability / Performance:**
- No resource limits (risk of noisy neighbor & throttling unpredictability).
- Requests too low for critical payment API; may cause CPU starvation.
- Missing readiness/liveness probes; traffic may route before warm-up.
- No PodDisruptionBudget; maintenance could evict too many pods.
- No anti-affinity / topology spread; failure domain concentration.

**Deployment Hygiene:**
- No rollout strategy tuning; default might cause sudden capacity drops.
- No HPA for scaling.
- `hostNetwork` with NodePort redundant; using proper Service/Ingress cleaner.

**Observability / Compliance:**
- No annotations/labels for traceability (commit, build provenance, owner, SLO tier).
- No securityContext for volumes.

**Supply Chain:**
- No signature / digest pinning.
- Broad privilege blocks enforcement of Pod Security restricted profile.

### 2. Improved Manifest

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: payments
  labels:
    pod-security.kubernetes.io/enforce: restricted
    app.kubernetes.io/part-of: payments-platform
---
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: payments-critical
value: 900000
preemptionPolicy: PreemptLowerPriority
globalDefault: false
description: High priority for critical payments traffic.
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: payments-api
  namespace: payments
  labels:
    app: payments-api
    app.kubernetes.io/name: payments-api
    app.kubernetes.io/version: 1.4.3
    app.kubernetes.io/component: api
    security-tier: restricted
spec:
  replicas: 6
  revisionHistoryLimit: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  selector:
    matchLabels:
      app: payments-api
  template:
    metadata:
      labels:
        app: payments-api
        topology.spread/pod: payments-api
      annotations:
        checksum/config: "<configmap-hash>"
        image.integrity/signature: cosign-v1
    spec:
      priorityClassName: payments-critical
      enableServiceLinks: false
      dnsPolicy: ClusterFirst
      securityContext:
        fsGroup: 2000
        runAsNonRoot: true
        runAsUser: 1000
        seccompProfile:
          type: RuntimeDefault
      containers:
      - name: api
        image: registry.local/payments/api@sha256:abcdef1234567890deadbeefcafebabe
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 8443
          name: https
        resources:
          requests:
            cpu: 500m
            memory: 512Mi
          limits:
            cpu: 1000m
            memory: 1Gi
        securityContext:
          allowPrivilegeEscalation: false
          privileged: false
          readOnlyRootFilesystem: true
          capabilities:
            drop: ["ALL"]
          seccompProfile:
            type: RuntimeDefault
        readinessProbe:
          httpGet:
            path: /healthz/ready
            port: https
            scheme: HTTPS
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 2
          failureThreshold: 3
        livenessProbe:
          httpGet:
            path: /healthz/live
            port: https
            scheme: HTTPS
          initialDelaySeconds: 20
          periodSeconds: 10
          timeoutSeconds: 2
          failureThreshold: 3
        startupProbe:
          httpGet:
            path: /healthz/startup
            port: https
            scheme: HTTPS
          periodSeconds: 5
          failureThreshold: 12
        envFrom:
        - secretRef:
            name: payments-api-env
        volumeMounts:
        - name: tls
          mountPath: /etc/tls
          readOnly: true
        - name: runtime-socket
          mountPath: /var/run/app
        - name: tmp
          mountPath: /tmp
        terminationMessagePolicy: FallbackToLogsOnError
      volumes:
      - name: tls
        secret:
          secretName: payments-api-tls
      - name: runtime-socket
        emptyDir: { medium: Memory }
      - name: tmp
        emptyDir: {}
      topologySpreadConstraints:
      - maxSkew: 1
        topologyKey: kubernetes.io/hostname
        whenUnsatisfiable: DoNotSchedule
        labelSelector:
          matchLabels:
            app: payments-api
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchLabels:
                app: payments-api
            topologyKey: kubernetes.io/hostname
      tolerations:
      - key: dedicated
        operator: Equal
        value: payments
        effect: NoSchedule
      imagePullSecrets:
      - name: registry-creds
---
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: payments-api-pdb
  namespace: payments
spec:
  minAvailable: 5
  selector:
    matchLabels:
      app: payments-api
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: payments-api-hpa
  namespace: payments
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: payments-api
  minReplicas: 6
  maxReplicas: 12
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 60
  - type: Resource
    resource:
      name: memory
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
apiVersion: v1
kind: Service
metadata:
  name: payments-api
  namespace: payments
  labels:
    app: payments-api
spec:
  selector:
    app: payments-api
  ports:
  - port: 443
    targetPort: https
    name: https
  type: ClusterIP
  sessionAffinity: None
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: payments-api-restrict
  namespace: payments
spec:
  podSelector:
    matchLabels:
      app: payments-api
  policyTypes: [Ingress, Egress]
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          access: payments-clients
    - podSelector:
        matchLabels:
          app: payments-gateway
    ports:
    - port: 443
      protocol: TCP
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          name: observability
    - podSelector:
        matchLabels:
          app: db
    ports:
    - port: 5432
      protocol: TCP
  - to:
    - namespaceSelector:
        matchLabels:
          kube-system: "true"
    ports:
    - port: 53
      protocol: UDP
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: payments-api
  namespace: payments
  annotations:
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
    nginx.ingress.kubernetes.io/proxy-body-size: "4m"
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  tls:
  - hosts:
    - payments.example.com
    secretName: payments-api-public-tls
  rules:
  - host: payments.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: payments-api
            port:
              number: 443
```

### 3. Rationale Summary
- Dropped privileged/hostNetwork/hostPath → Reduces attack surface & enforces NetworkPolicy.
- Seccomp, non-root, read-only FS, cap drop → Defense-in-depth; meets restricted profile.
- Resource requests/limits tuned → Deterministic performance; avoids CPU starvation.
- Probes (startup/readiness/liveness) → Reliable rollout gating & failure detection.
- Topology spread + anti-affinity → High availability; reduces correlated failure.
- PDB → Maintains service during node drains.
- HPA with conservative behavior → Smooth scaling; protects latency.
- NetworkPolicy → Zero-trust networking; restricts egress to required services.
- Digest-pinned image + signature annotation → Supply chain integrity & rollback confidence.
- PriorityClass + toleration → Scheduling guarantees on dedicated nodes.
- Ingress (TLS) vs NodePort → Standardized edge, centralized cert management.
- Namespaced + pod-security labels → Automatic enforcement of restricted defaults.

Collectively these changes harden security, stabilize latency, enable safe scaling, and improve operational governance.
```
