# Kubernetes - Intermediate Interview Mock #2 - Answers

> **Difficulty:** Intermediate  
> **Duration:** ~45 minutes  
> **Goal:** Assess your understanding of Kubernetes networking, security, monitoring, and advanced troubleshooting techniques.

---

## 🧠 Section 1: Core Questions - Answers

### 1. Explain Kubernetes networking model and how pod-to-pod communication works across nodes

**Answer:**

Kubernetes follows a flat networking model with several key principles:

**Core Networking Principles:**
- Every pod gets its own IP address
- All pods can communicate with each other without NAT
- Nodes can communicate with all pods without NAT
- No port mapping required

**Pod-to-Pod Communication Across Nodes:**

1. **Same Node Communication:**
   - Pods share the same network namespace as the node
   - Communication happens through the node's virtual ethernet bridge (cbr0)
   - Direct layer 2 communication

2. **Cross-Node Communication:**
   - Each node has a unique pod CIDR range
   - Container Network Interface (CNI) plugins handle routing
   - Popular CNI plugins: Flannel, Calico, Weave, Cilium

**Network Flow Example:**
```
Pod A (10.244.1.10) → Node A → CNI → Network → CNI → Node B → Pod B (10.244.2.15)
```

**CNI Plugin Responsibilities:**
- IP address allocation to pods
- Network route configuration
- Cross-node connectivity setup
- Network policy enforcement (some plugins)

### 2. What are Network Policies and how do they enhance cluster security? Provide examples

**Answer:**

Network Policies are Kubernetes resources that control traffic flow at the IP address or port level for pods.

**Key Benefits:**
- **Microsegmentation**: Isolate workloads at network level
- **Zero-trust networking**: Default deny, explicit allow
- **Compliance**: Meet regulatory requirements
- **Attack surface reduction**: Limit lateral movement

**Example 1: Default Deny All Policy**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: production
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

**Example 2: Allow Specific Communication**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: web-to-api-policy
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: web-frontend
  policyTypes:
  - Egress
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: api-backend
    ports:
    - protocol: TCP
      port: 8080
  - to: []  # Allow DNS
    ports:
    - protocol: UDP
      port: 53
```

**Example 3: Cross-Namespace Communication**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: database-access
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
      podSelector:
        matchLabels:
          role: api
    ports:
    - protocol: TCP
      port: 5432
```

### 3. How do RBAC (Role-Based Access Control) components work together? Explain Roles vs ClusterRoles

**Answer:**

RBAC in Kubernetes consists of four main components working together:

**Core Components:**
1. **Role/ClusterRole**: Define what can be done
2. **RoleBinding/ClusterRoleBinding**: Define who can do it
3. **ServiceAccount**: Identity for pods
4. **User/Group**: Identity for humans

**Roles vs ClusterRoles:**

| Aspect | Role | ClusterRole |
|--------|------|-------------|
| **Scope** | Namespace-specific | Cluster-wide |
| **Resources** | Namespaced resources only | All resources including cluster-scoped |
| **Use Cases** | Team isolation, app-specific permissions | Admin tasks, cross-namespace operations |

**Example Role:**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: development
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods", "pods/log"]
  verbs: ["get", "list", "watch"]
```

**Example ClusterRole:**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: node-manager
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["get", "list", "watch", "patch"]
- apiGroups: ["metrics.k8s.io"]
  resources: ["nodes", "pods"]
  verbs: ["get", "list"]
```

**RoleBinding Example:**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: dev-team-binding
  namespace: development
subjects:
- kind: User
  name: alice
  apiGroup: rbac.authorization.k8s.io
- kind: ServiceAccount
  name: dev-sa
  namespace: development
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

### 4. What are Admission Controllers and how do they impact cluster security and governance?

**Answer:**

Admission Controllers are plugins that intercept requests to the Kubernetes API server before objects are persisted, enabling validation and mutation of resources.

**Types of Admission Controllers:**
1. **Validating**: Approve or reject requests
2. **Mutating**: Modify objects before storage
3. **Mutating + Validating**: Both capabilities

**Security and Governance Impact:**

**Built-in Admission Controllers:**
- **PodSecurityPolicy**: Enforce security policies on pods
- **ResourceQuota**: Limit resource consumption
- **LimitRanger**: Set default/max resource limits
- **ServiceAccount**: Auto-inject service account tokens
- **ImagePolicyWebhook**: Validate container images

**Example: Pod Security Standards**
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: restricted-namespace
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/warn: restricted
```

**Custom Admission Controller (Webhook):**
```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingAdmissionWebhook
metadata:
  name: security-policy-webhook
webhooks:
- name: validate-security.company.com
  clientConfig:
    service:
      name: security-webhook
      namespace: security-system
      path: /validate
  rules:
  - operations: ["CREATE", "UPDATE"]
    apiGroups: ["apps"]
    apiVersions: ["v1"]
    resources: ["deployments"]
  admissionReviewVersions: ["v1", "v1beta1"]
```

**Common Use Cases:**
- Image vulnerability scanning
- Resource limit enforcement
- Compliance policy validation
- Network policy injection
- Secret encryption requirements

### 5. Explain Service Mesh concepts and how they integrate with Kubernetes (Istio/Linkerd)

**Answer:**

A Service Mesh is a dedicated infrastructure layer for handling service-to-service communication with features like traffic management, security, and observability.

**Core Concepts:**

**1. Data Plane vs Control Plane:**
- **Data Plane**: Sidecar proxies (Envoy in Istio)
- **Control Plane**: Management and configuration components

**2. Sidecar Pattern:**
- Proxy container injected alongside application container
- Intercepts all network communication
- Transparent to application code

**Istio Architecture:**
```yaml
# Istio components
- istiod (Control Plane)
  - Pilot: Traffic management
  - Citadel: Security (mTLS, RBAC)
  - Galley: Configuration validation
- Envoy Proxy (Data Plane)
- Istio Gateway (Edge proxy)
```

**Example Istio Configuration:**

**Traffic Management:**
```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews-route
spec:
  http:
  - match:
    - headers:
        end-user:
          exact: jason
    route:
    - destination:
        host: reviews
        subset: v2
  - route:
    - destination:
        host: reviews
        subset: v1
---
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: reviews-destination
spec:
  host: reviews
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
```

**Security (mTLS):**
```yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: production
spec:
  mtls:
    mode: STRICT
```

**Benefits:**
- **Traffic Management**: Load balancing, circuit breaking, retries
- **Security**: Automatic mTLS, policy enforcement
- **Observability**: Distributed tracing, metrics, logs
- **Resilience**: Fault injection, timeouts

### 6. How do you implement monitoring and observability in Kubernetes clusters?

**Answer:**

Comprehensive monitoring requires the "Three Pillars of Observability": Metrics, Logs, and Traces.

**Architecture Overview:**
```
Applications → Metrics/Logs/Traces → Collection → Storage → Visualization/Alerting
```

**1. Metrics Collection:**

**Prometheus Stack:**
```yaml
# ServiceMonitor for automatic scraping
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: app-metrics
spec:
  selector:
    matchLabels:
      app: web-application
  endpoints:
  - port: metrics
    interval: 30s
    path: /metrics
```

**Key Metrics to Monitor:**
- **Cluster**: CPU, memory, disk, network
- **Nodes**: Resource utilization, kubelet metrics
- **Pods**: Container metrics, application metrics
- **Control Plane**: API server, etcd, scheduler

**2. Logging Strategy:**

**Centralized Logging with ELK/EFK:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: fluentd-config
data:
  fluent.conf: |
    <source>
      @type tail
      path /var/log/containers/*.log
      pos_file /var/log/fluentd-containers.log.pos
      tag kubernetes.*
      read_from_head true
      <parse>
        @type json
        time_format %Y-%m-%dT%H:%M:%S.%NZ
      </parse>
    </source>
    <match kubernetes.**>
      @type elasticsearch
      host elasticsearch.logging.svc.cluster.local
      port 9200
      index_name kubernetes
    </match>
```

**3. Distributed Tracing:**

**Jaeger Integration:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jaeger-collector
spec:
  template:
    spec:
      containers:
      - name: jaeger-collector
        image: jaegertracing/jaeger-collector
        env:
        - name: SPAN_STORAGE_TYPE
          value: elasticsearch
```

**4. Alerting Rules:**
```yaml
groups:
- name: kubernetes
  rules:
  - alert: PodCrashLooping
    expr: rate(kube_pod_container_status_restarts_total[15m]) > 0
    for: 5m
    annotations:
      summary: Pod {{ $labels.pod }} is crash looping
      
  - alert: NodeNotReady
    expr: kube_node_status_condition{condition="Ready",status="true"} == 0
    for: 5m
    annotations:
      summary: Node {{ $labels.node }} is not ready
```

### 7. What are Taints and Tolerations and how do they affect pod scheduling?

**Answer:**

Taints and Tolerations work together to ensure pods are not scheduled onto inappropriate nodes.

**Concepts:**
- **Taint**: Applied to nodes, repels pods without matching tolerations
- **Toleration**: Applied to pods, allows scheduling on tainted nodes

**Taint Effects:**
1. **NoSchedule**: New pods won't be scheduled
2. **PreferNoSchedule**: Soft constraint, avoid if possible
3. **NoExecute**: Evict existing pods without tolerations

**Example Scenarios:**

**1. Dedicated Nodes for Specific Workloads:**
```bash
# Taint node for GPU workloads
kubectl taint nodes gpu-node-1 workload=gpu:NoSchedule
```

```yaml
# Pod with toleration for GPU nodes
apiVersion: v1
kind: Pod
metadata:
  name: gpu-workload
spec:
  tolerations:
  - key: workload
    operator: Equal
    value: gpu
    effect: NoSchedule
  containers:
  - name: gpu-app
    image: tensorflow/tensorflow:latest-gpu
```

**2. Node Maintenance:**
```bash
# Taint node to prevent new pods during maintenance
kubectl taint nodes worker-1 maintenance=true:NoSchedule
```

**3. Node with Issues:**
```bash
# Cordon and taint problematic node
kubectl cordon worker-2
kubectl taint nodes worker-2 node-problem=disk-full:NoExecute
```

**4. Master Node Protection:**
```yaml
# Typical master node taint
tolerations:
- key: node-role.kubernetes.io/master
  operator: Exists
  effect: NoSchedule
```

**Advanced Toleration Examples:**
```yaml
tolerations:
# Exact match
- key: "key1"
  operator: "Equal"
  value: "value1"
  effect: "NoSchedule"
  
# Key exists (any value)
- key: "key2"
  operator: "Exists"
  effect: "NoExecute"
  
# Tolerate for specific time
- key: "key3"
  operator: "Equal"
  value: "value3"
  effect: "NoExecute"
  tolerationSeconds: 3600
```

### 8. How do you troubleshoot network connectivity issues between pods and services?

**Answer:**

Systematic approach to network troubleshooting in Kubernetes:

**1. Initial Assessment:**

```bash
# Check pod status and events
kubectl get pods -o wide
kubectl describe pod <pod-name>

# Check service endpoints
kubectl get endpoints <service-name>
kubectl describe service <service-name>
```

**2. DNS Resolution Testing:**
```bash
# Test DNS from within a pod
kubectl exec -it <pod-name> -- nslookup <service-name>
kubectl exec -it <pod-name> -- dig <service-name>.<namespace>.svc.cluster.local

# Check CoreDNS
kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl logs -n kube-system -l k8s-app=kube-dns
```

**3. Network Connectivity Tests:**
```bash
# Test connectivity to service IP
kubectl exec -it <pod-name> -- telnet <service-ip> <port>

# Test direct pod-to-pod connectivity
kubectl exec -it <pod-name> -- ping <target-pod-ip>

# Test with curl
kubectl exec -it <pod-name> -- curl -v http://<service-name>:<port>
```

**4. Network Policy Investigation:**
```bash
# List network policies
kubectl get networkpolicy -A

# Check if policies are blocking traffic
kubectl describe networkpolicy <policy-name>

# Temporarily remove policies for testing
kubectl delete networkpolicy <policy-name>
```

**5. CNI Plugin Debugging:**
```bash
# Check CNI plugin logs
kubectl logs -n kube-system -l app=<cni-plugin>

# Verify node network configuration
kubectl get nodes -o wide
ip route show  # On nodes
```

**6. Service Mesh Troubleshooting (if applicable):**
```bash
# Check Istio sidecar injection
kubectl get pods -o jsonpath='{.items[*].spec.containers[*].name}'

# Istio proxy logs
kubectl logs <pod-name> -c istio-proxy

# Check Istio configuration
istioctl proxy-config cluster <pod-name>
istioctl analyze
```

**7. Common Tools for Network Debugging:**
```yaml
# Debug pod with network tools
apiVersion: v1
kind: Pod
metadata:
  name: network-debug
spec:
  containers:
  - name: debug
    image: nicolaka/netshoot
    command: ["/bin/bash"]
    args: ["-c", "while true; do ping localhost; sleep 60;done"]
```

**8. Packet Capture:**
```bash
# Capture traffic on node
sudo tcpdump -i any host <pod-ip>

# Inside debug container
tcpdump -i eth0 -w /tmp/capture.pcap
```

---

## ⚙️ Section 2: Scenario - Answer

**Multi-Tenant Kubernetes Security and Governance Strategy**

### 1. Namespace Isolation Setup

```yaml
# Team namespaces with labels
apiVersion: v1
kind: Namespace
metadata:
  name: team-frontend
  labels:
    team: frontend
    security-level: standard
    pod-security.kubernetes.io/enforce: baseline
---
apiVersion: v1
kind: Namespace
metadata:
  name: team-backend
  labels:
    team: backend
    security-level: restricted
    pod-security.kubernetes.io/enforce: restricted
---
apiVersion: v1
kind: Namespace
metadata:
  name: team-data
  labels:
    team: data
    security-level: high
    pod-security.kubernetes.io/enforce: restricted
```

### 2. Network Isolation

```yaml
# Default deny-all policy per namespace
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: team-frontend
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
---
# Allow frontend to backend communication
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: frontend-to-backend
  namespace: team-backend
spec:
  podSelector:
    matchLabels:
      tier: api
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          team: frontend
    ports:
    - protocol: TCP
      port: 8080
---
# Allow backend to data communication
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-to-data
  namespace: team-data
spec:
  podSelector:
    matchLabels:
      tier: database
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          team: backend
      podSelector:
        matchLabels:
          role: data-access
    ports:
    - protocol: TCP
      port: 5432
```

### 3. RBAC Configuration

```yaml
# Team-specific ClusterRoles
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: team-developer
rules:
# Basic pod operations
- apiGroups: [""]
  resources: ["pods", "pods/log", "pods/exec"]
  verbs: ["get", "list", "watch", "create", "delete"]
# Deployment management
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
# Service and ingress
- apiGroups: [""]
  resources: ["services"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
---
# Team lead role (additional permissions)
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: team-lead
rules:
- apiGroups: [""]
  resources: ["secrets", "configmaps"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
- apiGroups: ["networking.k8s.io"]
  resources: ["networkpolicies"]
  verbs: ["get", "list", "watch"]
---
# RoleBindings per team
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: frontend-team-binding
  namespace: team-frontend
subjects:
- kind: User
  name: alice@company.com
  apiGroup: rbac.authorization.k8s.io
- kind: User
  name: bob@company.com
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: team-developer
  apiGroup: rbac.authorization.k8s.io
```

### 4. Resource Quotas and Limits

```yaml
# Namespace resource quotas
apiVersion: v1
kind: ResourceQuota
metadata:
  name: team-frontend-quota
  namespace: team-frontend
spec:
  hard:
    requests.cpu: "4"
    requests.memory: 8Gi
    limits.cpu: "8"
    limits.memory: 16Gi
    pods: "10"
    services: "5"
    secrets: "10"
    configmaps: "10"
    persistentvolumeclaims: "4"
---
# Limit ranges for default values
apiVersion: v1
kind: LimitRange
metadata:
  name: team-frontend-limits
  namespace: team-frontend
spec:
  limits:
  - default:
      cpu: 100m
      memory: 128Mi
    defaultRequest:
      cpu: 50m
      memory: 64Mi
    type: Container
  - max:
      cpu: 2
      memory: 4Gi
    min:
      cpu: 10m
      memory: 16Mi
    type: Container
```

### 5. Security Policies and Scanning

```yaml
# Pod Security Standards via admission controller
apiVersion: v1
kind: ConfigMap
metadata:
  name: security-policies
  namespace: policy-system
data:
  policy.yaml: |
    apiVersion: config.gatekeeper.sh/v1alpha1
    kind: Config
    metadata:
      name: config
      namespace: gatekeeper-system
    spec:
      match:
        - excludedNamespaces: ["kube-system", "kube-public"]
          processes: ["*"]
      validation:
        traces:
          - user:
              kind:
                group: "user"
                version: "v1"
                kind: "User"
---
# OPA Gatekeeper constraint for security
apiVersion: templates.gatekeeper.sh/v1beta1
kind: ConstraintTemplate
metadata:
  name: k8srequiredsecuritycontext
spec:
  crd:
    spec:
      names:
        kind: K8sRequiredSecurityContext
      validation:
        openAPIV3Schema:
          type: object
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8srequiredsecuritycontext
        
        violation[{"msg": msg}] {
          container := input.review.object.spec.containers[_]
          not container.securityContext.runAsNonRoot
          msg := "Containers must run as non-root user"
        }
```

### 6. Monitoring and Audit Configuration

```yaml
# Service monitors for each team
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: team-frontend-monitor
  namespace: team-frontend
spec:
  selector:
    matchLabels:
      monitoring: enabled
  endpoints:
  - port: metrics
    interval: 30s
---
# Audit policy for compliance
apiVersion: audit.k8s.io/v1
kind: Policy
rules:
# Log all secret access
- level: RequestResponse
  resources:
  - group: ""
    resources: ["secrets"]
# Log security-related changes
- level: RequestResponse
  namespaces: ["team-data", "team-backend"]
  verbs: ["create", "update", "patch", "delete"]
  resources:
  - group: ""
    resources: ["pods", "services"]
  - group: "apps"
    resources: ["deployments"]
---
# Alert rules for security violations
groups:
- name: security-alerts
  rules:
  - alert: UnauthorizedSecretAccess
    expr: increase(apiserver_audit_total{verb="get",objectRef_resource="secrets"}[5m]) > 0
    labels:
      severity: warning
    annotations:
      summary: Unauthorized secret access detected
      
  - alert: PodSecurityViolation
    expr: increase(gatekeeper_violations_total[5m]) > 0
    labels:
      severity: critical
    annotations:
      summary: Pod security policy violation
```

---

## 🧩 Section 3: Problem-Solving - Answer

**Root Cause Analysis and Solution:**

### Issues Identified:

1. **Network Policy Mismatch**: Policy allows traffic from `backend` namespace, but app is in `frontend` namespace
2. **Missing Frontend Namespace Labels**: Frontend namespace needs proper labeling
3. **Istio Configuration**: Missing Istio-specific configurations for service mesh
4. **Cross-namespace Service Discovery**: Potential DNS resolution issues

### Corrected Configuration:

**1. Fix Namespace Labels:**
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: frontend
  labels:
    name: frontend
    istio-injection: enabled
---
apiVersion: v1
kind: Namespace
metadata:
  name: database
  labels:
    name: database
    istio-injection: enabled
```

**2. Updated Network Policy:**
```yaml
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
  # Allow from frontend namespace (corrected)
  - from:
    - namespaceSelector:
        matchLabels:
          name: frontend
    ports:
    - protocol: TCP
      port: 5432
  # Also allow from backend namespace (if needed)
  - from:
    - namespaceSelector:
        matchLabels:
          name: backend
    ports:
    - protocol: TCP
      port: 5432
```

**3. Add Egress Network Policy for Frontend:**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: frontend-egress-policy
  namespace: frontend
spec:
  podSelector:
    matchLabels:
      app: web-app
  policyTypes:
  - Egress
  egress:
  # Allow communication to database
  - to:
    - namespaceSelector:
        matchLabels:
          name: database
    ports:
    - protocol: TCP
      port: 5432
  # Allow DNS resolution
  - to: []
    ports:
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 53
  # Allow external traffic (if needed)
  - to: []
    ports:
    - protocol: TCP
      port: 80
    - protocol: TCP
      port: 443
```

**4. Istio Configuration for Service Mesh:**

**Destination Rule:**
```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: postgres-destination
  namespace: database
spec:
  host: postgres-service.database.svc.cluster.local
  trafficPolicy:
    tls:
      mode: ISTIO_MUTUAL  # Enable mTLS
```

**Service Entry (if needed for external services):**
```yaml
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: external-db
  namespace: database
spec:
  hosts:
  - postgres-service.database.svc.cluster.local
  ports:
  - number: 5432
    name: postgres
    protocol: TCP
  resolution: DNS
  location: MESH_EXTERNAL
```

**Authorization Policy for Istio:**
```yaml
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: database-access-policy
  namespace: database
spec:
  selector:
    matchLabels:
      app: postgres
  rules:
  - from:
    - source:
        namespaces: ["frontend"]
        principals: ["cluster.local/ns/frontend/sa/default"]
  - to:
    - operation:
        ports: ["5432"]
```

**5. Updated Application Deployment with Istio Annotations:**
```yaml
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
      annotations:
        sidecar.istio.io/inject: "true"
    spec:
      containers:
      - name: app
        image: myapp:latest
        env:
        - name: DB_HOST
          value: postgres-service.database.svc.cluster.local
        - name: DB_PORT
          value: "5432"
        # Add readiness probe for better service mesh integration
        readinessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5
```

**6. Troubleshooting Commands:**

```bash
# Verify network policy effectiveness
kubectl exec -it <web-app-pod> -n frontend -- telnet postgres-service.database.svc.cluster.local 5432

# Check Istio sidecar injection
kubectl get pods -n frontend -o jsonpath='{.items[*].spec.containers[*].name}'

# Verify Istio mTLS configuration
istioctl authn tls-check <web-app-pod>.frontend postgres-service.database.svc.cluster.local

# Check network policy logs (if supported by CNI)
kubectl logs -n kube-system -l app=<cni-plugin> | grep -i deny

# Test DNS resolution
kubectl exec -it <web-app-pod> -n frontend -- nslookup postgres-service.database.svc.cluster.local
```

**7. Monitoring and Validation:**

```yaml
# Add monitoring for database connections
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: database-monitor
  namespace: database
spec:
  selector:
    matchLabels:
      app: postgres
  endpoints:
  - port: metrics
    path: /metrics
---
# Alert for connection failures
groups:
- name: database-connectivity
  rules:
  - alert: DatabaseConnectionFailure
    expr: rate(postgres_connection_errors_total[5m]) > 0.1
    for: 2m
    annotations:
      summary: High database connection error rate
```

This comprehensive solution addresses network isolation, service mesh integration, and provides proper monitoring for the multi-tenant environment.
