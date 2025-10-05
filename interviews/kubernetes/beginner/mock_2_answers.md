# Kubernetes - Beginner Interview Mock #2 - Answer Key

> **Difficulty:** Beginner  
> **Duration:** ~30 minutes  
> **Goal:** Assess your understanding of Kubernetes networking, storage, configuration, and troubleshooting fundamentals.

---

## 🧠 Section 1: Core Questions - Answers

### 1. What is the difference between **ClusterIP**, **NodePort**, and **LoadBalancer** service types?

**Answer:** These are the three main service types for exposing applications:

- **ClusterIP** (default): Exposes the service on a cluster-internal IP. Only accessible from within the cluster. Used for internal communication between services.

- **NodePort**: Exposes the service on each node's IP at a static port (30000-32767 range). Makes the service accessible from outside the cluster via `<NodeIP>:<NodePort>`. Automatically creates a ClusterIP service as well.

- **LoadBalancer**: Exposes the service externally using a cloud provider's load balancer. Creates NodePort and ClusterIP services automatically. Provides a single external IP address for accessing the service.

**Use cases:**
- ClusterIP: Internal microservices communication
- NodePort: Development/testing external access
- LoadBalancer: Production external access in cloud environments

### 2. Explain what **ConfigMaps** and **Secrets** are used for and how they differ.

**Answer:**

**ConfigMaps**: Store non-confidential configuration data in key-value pairs. Used for:
- Application configuration files
- Environment variables
- Command-line arguments
- Any non-sensitive configuration data

**Secrets**: Store sensitive information like passwords, tokens, and keys. Features:
- Base64 encoded (not encrypted by default)
- Can be encrypted at rest in etcd
- More access controls and audit logging
- Memory-mounted (not written to disk)

**Key Differences:**
- **Security**: Secrets have additional security measures
- **Storage**: Secrets are stored in tmpfs on nodes
- **Access**: Secrets have more restrictive RBAC controls
- **Encoding**: Secrets are base64 encoded, ConfigMaps are plain text
- **Use case**: Secrets for sensitive data, ConfigMaps for regular config

### 3. What is the role of **etcd** in a Kubernetes cluster?

**Answer:** etcd is a distributed key-value store that serves as Kubernetes' backing store for all cluster data:

**Functions:**
- **Cluster state storage**: Stores all Kubernetes objects (pods, services, deployments, etc.)
- **Configuration data**: Holds cluster configuration and metadata
- **Service discovery**: Maintains service endpoints and DNS records
- **Distributed coordination**: Provides consistency across cluster nodes
- **Watch API**: Enables real-time notifications of state changes

**Characteristics:**
- Highly available and consistent (uses Raft consensus algorithm)
- Only the API server communicates directly with etcd
- Critical for cluster operation - if etcd fails, cluster becomes read-only
- Backup and disaster recovery essential for production clusters

### 4. How do **liveness probes** and **readiness probes** work, and why are they important?

**Answer:**

**Liveness Probes**: Determine if a container is running properly
- **Action**: If fails, kubelet kills and restarts the container
- **Use case**: Detect deadlocked applications that appear running but are unresponsive
- **Example**: HTTP GET to `/healthz` endpoint

**Readiness Probes**: Determine if a container is ready to serve traffic
- **Action**: If fails, removes pod from service endpoints (no traffic routing)
- **Use case**: Prevent traffic to pods still starting up or temporarily overloaded
- **Example**: Check database connection before serving requests

**Why Important:**
- **High availability**: Automatic recovery from failures
- **Zero-downtime deployments**: Traffic only goes to ready pods
- **Better user experience**: Prevents requests to failing instances
- **Operational reliability**: Reduces manual intervention

**Probe Types:**
- HTTP GET requests
- TCP socket connections  
- Command execution

### 5. What is a **PersistentVolume** and **PersistentVolumeClaim**? How do they relate to each other?

**Answer:**

**PersistentVolume (PV)**: A piece of storage in the cluster provisioned by an administrator or dynamically using Storage Classes.
- Cluster-level resource
- Independent lifecycle from pods
- Represents actual storage (NFS, cloud disks, etc.)

**PersistentVolumeClaim (PVC)**: A request for storage by a user/pod.
- Namespace-scoped resource
- Specifies size, access modes, and storage class
- Acts like a "voucher" that pods can use

**Relationship:**
1. **Binding**: PVC finds and binds to a suitable PV
2. **Usage**: Pods reference PVC in volume mounts
3. **Lifecycle**: PVC protects PV from deletion while in use

**Flow:**
```
Storage Admin creates PV → User creates PVC → PVC binds to PV → Pod uses PVC → Data persists beyond pod lifecycle
```

**Benefits:**
- Decouples storage provisioning from consumption
- Enables data persistence across pod restarts
- Provides storage abstraction for developers

### 6. How would you check the resource usage (CPU and memory) of nodes and pods?

**Answer:** Multiple approaches depending on what metrics server is available:

**Using kubectl top (requires metrics-server):**
```bash
# Check node resource usage
kubectl top nodes

# Check pod resource usage (all namespaces)
kubectl top pods --all-namespaces

# Check specific namespace
kubectl top pods -n production

# Sort by CPU usage
kubectl top pods --sort-by=cpu

# Sort by memory usage  
kubectl top pods --sort-by=memory
```

**Using describe commands:**
```bash
# Node resource allocation and usage
kubectl describe node <node-name>

# Pod resource requests/limits
kubectl describe pod <pod-name>
```

**Additional monitoring options:**
```bash
# Check resource requests/limits in deployments
kubectl get deployment <name> -o yaml | grep -A 10 resources

# View resource quotas
kubectl get resourcequota --all-namespaces

# Check if metrics-server is running
kubectl get pods -n kube-system | grep metrics-server
```

### 7. What command would you use to get detailed information about a specific pod including events?

**Answer:**

**Primary command:**
```bash
kubectl describe pod <pod-name>
```

**With namespace:**
```bash
kubectl describe pod <pod-name> -n <namespace>
```

**The describe command provides:**
- Pod metadata and labels
- Container specifications and status
- Volume mounts and persistent volume claims
- Network information (IP addresses, ports)
- Resource requests and limits
- Container state and restart counts
- **Recent events** (most important for troubleshooting)

**Additional commands for more specific information:**
```bash
# Get events separately, sorted by time
kubectl get events --sort-by='.lastTimestamp' --field-selector involvedObject.name=<pod-name>

# Get pod details in YAML format
kubectl get pod <pod-name> -o yaml

# Get pod status and basic info
kubectl get pod <pod-name> -o wide
```

### 8. What is the purpose of **labels** and **selectors** in Kubernetes?

**Answer:**

**Labels**: Key-value pairs attached to Kubernetes objects for identification and organization.
- **Format**: `key=value` (e.g., `app=nginx`, `environment=production`)
- **Flexible**: Can add multiple labels to any object
- **Mutable**: Can be added, modified, or removed after creation

**Selectors**: Used to filter and select objects based on their labels.
- **Equality-based**: `app=nginx`, `environment!=development`
- **Set-based**: `environment in (production,staging)`, `tier notin (frontend)`

**Use Cases:**
- **Service discovery**: Services use selectors to find target pods
- **Deployments**: ReplicaSets use selectors to manage pods
- **Resource grouping**: Organize resources by application, team, environment
- **Operational tasks**: Bulk operations on labeled resources

**Examples:**
```bash
# Get pods with specific label
kubectl get pods -l app=nginx

# Get pods with multiple label conditions
kubectl get pods -l 'app=nginx,environment=production'

# Delete resources by label
kubectl delete pods -l app=nginx
```

**Best Practices:**
- Use consistent labeling conventions
- Include app name, version, component, environment
- Use labels for automation and monitoring tools

---

## ⚙️ Section 2: Scenario - Answer

**Scenario:**  
Your application pods are running but users report they cannot access the application. The pods show as "Running" but the service endpoints show as "Not Ready".  
Describe the troubleshooting steps you would take to identify and resolve this issue.

**Answer:** This is a classic readiness probe issue. Here's a systematic troubleshooting approach:

### 1. **Verify Service and Endpoints**
```bash
# Check service configuration
kubectl get service <service-name> -o wide

# Check service endpoints
kubectl get endpoints <service-name>

# Describe service for detailed info
kubectl describe service <service-name>
```

### 2. **Check Pod Status and Labels**
```bash
# Verify pods are running and have correct labels
kubectl get pods -l <service-selector-labels> -o wide

# Check if pod labels match service selector
kubectl describe service <service-name> | grep Selector
kubectl get pods --show-labels
```

### 3. **Examine Readiness Probes**
```bash
# Check pod configuration for readiness probes
kubectl describe pod <pod-name>

# Look for readiness probe failures in events
kubectl get events --field-selector involvedObject.name=<pod-name>
```

### 4. **Test Application Health**
```bash
# Test if application is actually responding inside the pod
kubectl exec -it <pod-name> -- curl localhost:<port>/health

# Check application logs for errors
kubectl logs <pod-name>

# Test network connectivity
kubectl exec -it <pod-name> -- netstat -tlnp
```

### 5. **Check Readiness Probe Configuration**
```bash
# Get deployment/pod YAML to examine probe settings
kubectl get deployment <deployment-name> -o yaml
```

### **Common Root Causes and Solutions:**

**1. Readiness probe failing:**
- **Cause**: Application takes longer to start than probe allows
- **Solution**: Increase `initialDelaySeconds` and `periodSeconds`

**2. Wrong probe endpoint:**
- **Cause**: Readiness probe checking wrong path/port
- **Solution**: Update probe to correct health check endpoint

**3. Application not fully initialized:**
- **Cause**: App appears running but dependencies aren't ready
- **Solution**: Improve application startup logic or readiness check

**4. Label mismatch:**
- **Cause**: Service selector doesn't match pod labels
- **Solution**: Fix labels or selector to match

**5. Port misconfiguration:**
- **Cause**: Service targeting wrong container port
- **Solution**: Verify `targetPort` matches container `containerPort`

### **Example Fix for Readiness Probe:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  template:
    spec:
      containers:
      - name: app
        image: myapp:v1
        ports:
        - containerPort: 8080
        readinessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30  # Wait 30s before first check
          periodSeconds: 10        # Check every 10s
          timeoutSeconds: 5        # 5s timeout per check
          failureThreshold: 3      # 3 failures = not ready
```

---

## 🧩 Section 3: Problem-Solving - Answer

**Task:**  
A developer has created this Deployment but is experiencing issues:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - name: web-container
        image: nginx:1.20
        ports:
        - containerPort: 80
        env:
        - name: DATABASE_URL
          value: "postgresql://user:password@db-host:5432/mydb"
```

Identify at least 3 problems with this Deployment and explain how you would fix them. What Kubernetes best practices are being violated?

**Answer:**

### **Problems Identified:**

### **1. Missing Selector (Critical Error)**
**Problem**: No `selector` field in deployment spec - this will cause deployment creation to fail.

**Fix**:
```yaml
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-app  # Must match template labels
  template:
    # ... rest of template
```

### **2. Hardcoded Sensitive Information**
**Problem**: Database password exposed in plain text in environment variable.

**Security Risk**: Credentials visible in deployment YAML, pod specs, and logs.

**Fix**: Use Secrets for sensitive data:
```yaml
# Create secret first
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
data:
  database-url: <base64-encoded-connection-string>
---
# Reference in deployment
spec:
  template:
    spec:
      containers:
      - name: web-container
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: database-url
```

### **3. No Resource Requests/Limits**
**Problem**: No CPU/memory resource management specified.

**Issues**: 
- Pods can consume unlimited resources
- No scheduling guarantees
- Risk of resource contention

**Fix**:
```yaml
spec:
  template:
    spec:
      containers:
      - name: web-container
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
```

### **4. No Health Checks**
**Problem**: Missing liveness and readiness probes.

**Issues**:
- No automatic recovery from failures
- Traffic sent to unready pods
- Poor deployment reliability

**Fix**:
```yaml
spec:
  template:
    spec:
      containers:
      - name: web-container
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
```

### **5. Missing Deployment Strategy**
**Problem**: No rolling update strategy defined.

**Fix**:
```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
```

### **Complete Fixed Deployment:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  labels:
    app: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-app
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - name: web-container
        image: nginx:1.20
        ports:
        - containerPort: 80
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: database-url
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
```

### **Best Practices Violated:**
1. **Security**: Hardcoded credentials
2. **Resource Management**: No resource constraints
3. **Reliability**: No health checks
4. **Maintainability**: Missing required selector
5. **Deployment Safety**: No deployment strategy
6. **Observability**: No proper labeling strategy
