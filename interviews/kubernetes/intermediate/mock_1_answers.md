# Kubernetes - Intermediate Interview Mock #1 - Answers

---

## 🧠 Section 1: Core Questions - Answers

### 1. How do you **check for resource availability** in a cluster? Include both current usage and capacity planning.

**Answer:**

To check resource availability in a Kubernetes cluster, you should examine both current usage and future capacity needs:

**Current Usage Assessment:**
```bash
# Check node resource utilization
kubectl top nodes

# Check pod resource usage
kubectl top pods --all-namespaces

# Get detailed node information including allocatable resources
kubectl describe nodes

# Check resource requests and limits across all pods
kubectl get pods --all-namespaces -o=custom-columns="NAMESPACE:.metadata.namespace,NAME:.metadata.name,CPU-REQ:.spec.containers[*].resources.requests.cpu,MEM-REQ:.spec.containers[*].resources.requests.memory"

# View cluster resource summary
kubectl cluster-info dump | grep -A 10 -B 10 "allocatable\|capacity"
```

**Capacity Planning Tools:**
- **Metrics Server**: For real-time resource metrics
- **Prometheus + Grafana**: For historical data and trends
- **Vertical Pod Autoscaler**: To understand right-sizing needs
- **Resource Quotas**: To track namespace-level consumption

**Key Metrics to Monitor:**
- CPU and memory requests vs limits vs actual usage
- Node pressure (disk, memory, PID)
- Pod scheduling failures due to resource constraints
- Resource fragmentation across nodes

---

### 2. Explain the difference between **ResourceQuota** and **LimitRange**. When would you use each?

**Answer:**

**ResourceQuota:**
- Sets **aggregate** resource limits at the **namespace level**
- Controls total resource consumption across all objects in a namespace
- Prevents any namespace from consuming more than its fair share

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: namespace-quota
spec:
  hard:
    requests.cpu: "4"
    requests.memory: 8Gi
    limits.cpu: "8"
    limits.memory: 16Gi
    pods: "10"
    persistentvolumeclaims: "5"
```

**LimitRange:**
- Sets **individual** resource limits at the **object level** (pods, containers, PVCs)
- Enforces minimum/maximum resource requests and limits per object
- Provides default values when not specified

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: pod-limit-range
spec:
  limits:
  - default:
      cpu: 200m
      memory: 256Mi
    defaultRequest:
      cpu: 100m
      memory: 128Mi
    max:
      cpu: 500m
      memory: 1Gi
    min:
      cpu: 50m
      memory: 64Mi
    type: Container
```

**When to use:**
- **ResourceQuota**: Multi-tenant clusters, cost control, preventing resource hogging
- **LimitRange**: Enforcing standards, preventing resource waste, setting defaults

---

### 3. What is **Pod Disruption Budget (PDB)** and how does it help during cluster maintenance?

**Answer:**

**Pod Disruption Budget** ensures application availability during voluntary disruptions (node maintenance, cluster upgrades, etc.).

**Key Concepts:**
- **Voluntary disruptions**: Planned events like node drains, deployments
- **Involuntary disruptions**: Hardware failures, kernel panics
- PDB only protects against voluntary disruptions

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: web-app-pdb
spec:
  minAvailable: 2  # or maxUnavailable: 1
  selector:
    matchLabels:
      app: web-app
```

**Benefits during maintenance:**
1. **Prevents service downtime**: Ensures minimum replicas stay running
2. **Coordinates with cluster operations**: `kubectl drain` respects PDBs
3. **Enables safe rolling updates**: Controls pod eviction rate
4. **Supports compliance requirements**: Maintains SLA guarantees

**Best Practices:**
- Set `minAvailable` to maintain service capacity
- Use `maxUnavailable` for flexible scaling scenarios
- Align PDB with deployment replica counts
- Consider dependencies between services

---

### 4. How do **Node Affinity** and **Pod Affinity/Anti-Affinity** work? Provide examples of when you'd use each.

**Answer:**

**Node Affinity** - Controls which nodes pods can be scheduled on:

```yaml
# Example: Schedule pods only on SSD-equipped nodes
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disk-type
            operator: In
            values: ["ssd"]
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        preference:
          matchExpressions:
          - key: zone
            operator: In
            values: ["us-west-1a"]
```

**Pod Affinity/Anti-Affinity** - Controls pod placement relative to other pods:

```yaml
# Anti-affinity: Spread replicas across nodes
spec:
  affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: app
            operator: In
            values: ["web-app"]
        topologyKey: kubernetes.io/hostname

# Affinity: Co-locate web app with cache
spec:
  affinity:
    podAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchExpressions:
            - key: app
              operator: In
              values: ["redis-cache"]
          topologyKey: kubernetes.io/hostname
```

**Use Cases:**
- **Node Affinity**: Hardware requirements (GPU, SSD), compliance zones
- **Pod Anti-Affinity**: High availability, fault tolerance
- **Pod Affinity**: Performance optimization, data locality

---

### 5. Explain **Horizontal Pod Autoscaler (HPA)** vs **Vertical Pod Autoscaler (VPA)** vs **Cluster Autoscaler**. How do they work together?

**Answer:**

**Horizontal Pod Autoscaler (HPA):**
- Scales the **number of pod replicas** based on metrics
- Reacts to load by adding/removing pods
- Works with CPU, memory, and custom metrics

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

**Vertical Pod Autoscaler (VPA):**
- Adjusts **resource requests and limits** per pod
- Right-sizes containers based on historical usage
- Can work in recommendation, auto, or initial modes

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: web-app-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-app
  updatePolicy:
    updateMode: "Auto"
```

**Cluster Autoscaler:**
- Adds/removes **nodes** based on pod scheduling needs
- Responds to unschedulable pods by scaling nodes
- Removes underutilized nodes to save costs

**How they work together:**
1. **VPA** right-sizes resource requests
2. **HPA** scales replicas based on load
3. **Cluster Autoscaler** adds nodes when HPA can't schedule new pods
4. All three optimize for both performance and cost

**Considerations:**
- Don't use HPA and VPA on the same metric simultaneously
- Set appropriate resource requests for HPA to work effectively
- Configure node pools properly for Cluster Autoscaler

---

### 6. What are **Custom Resource Definitions (CRDs)** and how do they extend Kubernetes functionality?

**Answer:**

**Custom Resource Definitions** allow you to extend Kubernetes API with your own resource types.

**Core Concepts:**
- **CRD**: Defines the schema and validation for custom resources
- **Custom Resource (CR)**: Instances of your CRD
- **Controllers/Operators**: Logic to act on custom resources

**Example CRD:**
```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: databases.stable.example.com
spec:
  group: stable.example.com
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              engine:
                type: string
                enum: ["mysql", "postgres"]
              size:
                type: string
              replicas:
                type: integer
                minimum: 1
  scope: Namespaced
  names:
    plural: databases
    singular: database
    kind: Database
```

**Custom Resource Instance:**
```yaml
apiVersion: stable.example.com/v1
kind: Database
metadata:
  name: my-db
spec:
  engine: postgres
  size: large
  replicas: 3
```

**Benefits:**
- **Domain-specific abstractions**: Model business logic in Kubernetes
- **Operator pattern**: Automate complex operational tasks
- **GitOps integration**: Manage custom resources declaratively
- **Ecosystem integration**: Build on Kubernetes primitives

**Common Use Cases:**
- Database operators (PostgreSQL, MySQL)
- Application frameworks (Istio, Prometheus)
- CI/CD pipelines (Tekton, Argo)
- Infrastructure management (Crossplane)

---

### 7. How do you perform a **rolling update** with zero downtime? What strategies can you use?

**Answer:**

**Rolling Update Strategy:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 4
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1      # Allow 1 pod to be unavailable
      maxSurge: 1            # Allow 1 extra pod during update
  template:
    spec:
      containers:
      - name: app
        image: myapp:v2.0
        readinessProbe:       # Critical for zero downtime
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
```

**Zero Downtime Requirements:**
1. **Proper Health Checks**: Readiness probes prevent traffic to unhealthy pods
2. **Graceful Shutdown**: Handle SIGTERM signals properly
3. **Service Configuration**: Use services for load balancing
4. **Rolling Update Parameters**: Configure surge and unavailable correctly

**Advanced Strategies:**

**Blue-Green Deployment:**
```bash
# Deploy to green environment
kubectl apply -f green-deployment.yaml

# Switch traffic atomically
kubectl patch service web-service -p '{"spec":{"selector":{"version":"green"}}}'

# Clean up blue environment
kubectl delete deployment web-app-blue
```

**Canary Deployment:**
```yaml
# Canary deployment with traffic splitting
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: web-app
spec:
  strategy:
    canary:
      steps:
      - setWeight: 10      # 10% traffic to new version
      - pause: {duration: 300s}
      - setWeight: 50
      - pause: {duration: 300s}
      - setWeight: 100
```

**Best Practices:**
- Always use readiness probes
- Set appropriate terminationGracePeriodSeconds
- Monitor deployment progress with `kubectl rollout status`
- Have rollback strategy: `kubectl rollout undo`

---

### 8. Explain **Ingress Controllers** and how they differ from **LoadBalancer** services.

**Answer:**

**LoadBalancer Service:**
- Layer 4 (TCP/UDP) load balancing
- One external IP per service
- Cloud provider specific
- Limited routing capabilities

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  type: LoadBalancer
  selector:
    app: web-app
  ports:
  - port: 80
    targetPort: 8080
  # Results in cloud LB with external IP
```

**Ingress + Ingress Controller:**
- Layer 7 (HTTP/HTTPS) routing
- Single entry point for multiple services
- Advanced routing (path, host, headers)
- TLS termination, authentication, rate limiting

```yaml
# Ingress resource
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    cert-manager.io/cluster-issuer: letsencrypt
spec:
  tls:
  - hosts:
    - myapp.example.com
    secretName: myapp-tls
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
```

**Popular Ingress Controllers:**
- **NGINX Ingress**: Most common, feature-rich
- **Traefik**: Dynamic configuration, service mesh integration
- **Ambassador**: API Gateway features
- **AWS ALB**: Native AWS integration
- **Istio Gateway**: Service mesh integration

**Key Differences:**

| Aspect | LoadBalancer | Ingress |
|--------|--------------|---------|
| Layer | L4 (TCP/UDP) | L7 (HTTP/HTTPS) |
| Cost | One LB per service | One LB for all services |
| Routing | Port-based only | Path, host, header-based |
| Features | Basic | SSL, auth, rate limiting |
| Protocol | Any | HTTP/HTTPS primarily |

**When to use:**
- **LoadBalancer**: Non-HTTP protocols, simple setups, database access
- **Ingress**: Web applications, microservices, complex routing needs

---

## ⚙️ Section 2: Scenario - Answer

**Scenario Analysis:**

The e-commerce application faces several challenges:
1. Resource constraints causing pod evictions
2. Uneven resource distribution across nodes  
3. Need for traffic spike handling
4. High availability requirements

**Comprehensive Solution:**

### 1. Resource Allocation Strategy

```yaml
# Frontend Deployment with proper resource management
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: frontend
        image: ecommerce/frontend:v1.2
        resources:
          requests:
            cpu: 200m
            memory: 256Mi
          limits:
            cpu: 500m
            memory: 512Mi
        readinessProbe:
          httpGet:
            path: /health
            port: 3000
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchLabels:
                app: frontend
            topologyKey: kubernetes.io/hostname
```

### 2. Scaling Strategy

```yaml
# HPA for handling traffic spikes
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: frontend-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: frontend
  minReplicas: 3
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
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
        value: 10
        periodSeconds: 60
```

### 3. Pod Scheduling Optimization

```yaml
# Resource quotas and limits
apiVersion: v1
kind: ResourceQuota
metadata:
  name: ecommerce-quota
  namespace: ecommerce
spec:
  hard:
    requests.cpu: "10"
    requests.memory: 20Gi
    limits.cpu: "20" 
    limits.memory: 40Gi

---
apiVersion: v1
kind: LimitRange
metadata:
  name: ecommerce-limits
  namespace: ecommerce
spec:
  limits:
  - default:
      cpu: 500m
      memory: 512Mi
    defaultRequest:
      cpu: 100m
      memory: 128Mi
    type: Container
```

### 4. High Availability Configuration

```yaml
# Pod Disruption Budget
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: frontend-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: frontend

---
# Node affinity for zone distribution
spec:
  template:
    spec:
      affinity:
        nodeAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            preference:
              matchExpressions:
              - key: topology.kubernetes.io/zone
                operator: In
                values: ["us-west-2a", "us-west-2b", "us-west-2c"]
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchLabels:
                app: frontend
            topologyKey: topology.kubernetes.io/zone
```

### 5. Monitoring and Alerting

```bash
# Monitor resource usage
kubectl top nodes
kubectl top pods -n ecommerce

# Set up alerts for:
# - High resource utilization (>85%)
# - Pod evictions 
# - HPA scaling events
# - Node pressure conditions
```

**Expected Outcomes:**
- Predictable resource allocation prevents OOMKills
- Even pod distribution across nodes and zones
- Automatic scaling handles traffic spikes
- High availability during maintenance windows
- Cost optimization through right-sizing

---

## 🧩 Section 3: Problem-Solving - Answer

**Issues Identified:**

1. **Poor Pod Distribution**: No anti-affinity rules
2. **No Auto-scaling**: Fixed replicas can't handle traffic spikes  
3. **Lack of Resilience**: No PDB for maintenance scenarios
4. **Inadequate Resource Management**: Only requests, no limits
5. **Missing Load Balancing**: No session affinity configuration

**Improved Configuration:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 6
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
        version: v1
    spec:
      containers:
      - name: web
        image: nginx:latest
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:              # Added resource limits
            cpu: 200m
            memory: 256Mi
        ports:
        - containerPort: 80
        readinessProbe:        # Added health checks
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 10
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 15
          periodSeconds: 20
      # Pod distribution strategy
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchLabels:
                app: web-app
            topologyKey: kubernetes.io/hostname
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchLabels:
                  app: web-app  
              topologyKey: topology.kubernetes.io/zone
      # Node selection preferences
      nodeAffinity:
        preferredDuringSchedulingIgnoredDuringExecution:
        - weight: 50
          preference:
            matchExpressions:
            - key: node-type
              operator: In
              values: ["worker"]

---
# Enhanced service with session affinity
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  selector:
    app: web-app
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
  sessionAffinity: ClientIP    # Added session affinity
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 10800

---
# Horizontal Pod Autoscaler for traffic spikes
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-app
  minReplicas: 6
  maxReplicas: 15
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory  
      target:
        type: Utilization
        averageUtilization: 80

---
# Pod Disruption Budget for maintenance resilience  
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: web-app-pdb
spec:
  minAvailable: 4           # Ensure 4 pods remain during disruptions
  selector:
    matchLabels:
      app: web-app

---
# Enhanced Ingress with load balancing annotations
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
  annotations:
    nginx.ingress.kubernetes.io/load-balance: "round_robin"
    nginx.ingress.kubernetes.io/upstream-hash-by: "$request_uri"
    nginx.ingress.kubernetes.io/rate-limit: "100"
    nginx.ingress.kubernetes.io/rate-limit-window: "1m"
spec:
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
```

**Key Improvements:**

1. **Pod Distribution**: 
   - Required anti-affinity spreads pods across nodes
   - Preferred anti-affinity spreads across zones
   - Node affinity preferences for worker nodes

2. **Scaling & Performance**:
   - HPA handles traffic spikes automatically
   - Resource limits prevent resource hogging
   - Readiness probes ensure traffic only goes to ready pods

3. **Resilience**:
   - PDB ensures 4 pods minimum during maintenance
   - Health checks detect and replace failed pods
   - Session affinity improves user experience

4. **Load Balancing**:
   - Round-robin distribution at ingress level
   - Rate limiting prevents abuse
   - URI-based hashing for consistency

**Monitoring Commands:**
```bash
# Verify pod distribution
kubectl get pods -o wide -l app=web-app

# Check HPA status  
kubectl get hpa web-app-hpa

# Monitor during scaling
kubectl top pods -l app=web-app

# Test PDB during maintenance
kubectl drain <node-name> --ignore-daemonsets
```

This configuration addresses all the identified issues while maintaining high availability and performance.
