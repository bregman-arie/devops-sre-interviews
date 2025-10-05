# Kubernetes - Beginner Interview Mock #4 - Answer Key

> **Difficulty:** Beginner  
> **Duration:** ~30 minutes  
> **Goal:** Assess your understanding of Kubernetes troubleshooting, networking fundamentals, workload management, and cluster operations.

---

## 🧠 Section 1: Core Questions - Answers

### 1. What is the difference between **kubectl apply** and **kubectl create**? When would you use each?

**Answer:**

**kubectl create**: Imperative command that creates resources from scratch
- **One-time operation**: Creates new resources only
- **Fails if exists**: Returns error if resource already exists
- **No tracking**: Doesn't track configuration changes
- **Direct creation**: Creates exactly what's specified

**kubectl apply**: Declarative command that creates or updates resources
- **Idempotent**: Can be run multiple times safely
- **Creates or updates**: Creates new resources or updates existing ones
- **Configuration tracking**: Tracks last-applied-configuration as annotation
- **Three-way merge**: Compares current state, desired state, and last applied config

**Examples:**
```bash
# CREATE - fails if pod already exists
kubectl create -f pod.yaml

# CREATE - imperative creation
kubectl create deployment nginx --image=nginx

# APPLY - creates or updates
kubectl apply -f deployment.yaml

# APPLY - can be run repeatedly
kubectl apply -f manifests/
```

**When to use each:**

**Use kubectl create:**
- One-time resource creation
- Quick testing and prototyping
- When you want to ensure resource doesn't exist
- Imperative operations (create secret, configmap, etc.)

**Use kubectl apply:**
- Production deployments
- GitOps workflows
- Configuration management
- When you need to update existing resources
- Continuous deployment pipelines

**Key Differences:**
- **Error handling**: create fails on existing resources, apply updates them
- **State management**: apply tracks configuration history
- **Workflow**: create is imperative, apply is declarative
- **Production use**: apply is preferred for production environments

### 2. Explain what **Init Containers** are and provide a use case where they would be helpful.

**Answer:**

**Init Containers**: Special containers that run and complete before the main application containers start
- **Sequential execution**: Run one at a time in specified order
- **Must complete successfully**: Main containers won't start if init containers fail
- **Same environment**: Share volumes, network, and security context with main containers
- **Temporary**: Don't run alongside main containers

**Key Characteristics:**
- Run to completion before main containers
- If any init container fails, the pod restarts
- Support all container features (resources, security, volumes)
- Always run on pod restart

**Use Case Example: Database Migration**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-app
spec:
  initContainers:
  # Wait for database to be ready
  - name: wait-for-db
    image: busybox:1.35
    command: ['sh', '-c']
    args:
    - until nslookup db-service.default.svc.cluster.local; 
      do echo waiting for db-service; sleep 2; done
  
  # Run database migrations
  - name: db-migration
    image: myapp:latest
    command: ['python', 'manage.py', 'migrate']
    env:
    - name: DATABASE_URL
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: url
  
  containers:
  - name: web-app
    image: myapp:latest
    ports:
    - containerPort: 8080
```

**Common Use Cases:**

1. **Service Dependencies**: Wait for dependent services to be ready
2. **Database Setup**: Run migrations or schema initialization
3. **Configuration Setup**: Download configuration files or certificates
4. **Security Setup**: Set up security contexts or fetch secrets
5. **Data Preparation**: Seed databases or prepare initial data
6. **Network Setup**: Configure networking or register with service discovery

**More Examples:**
```bash
# Wait for service availability
- name: wait-for-redis
  image: redis:alpine
  command: ['redis-cli', '-h', 'redis-service', 'ping']

# Download configuration
- name: config-downloader
  image: curlimages/curl
  command: ['curl', '-o', '/shared/config.json', 'https://config-server/app-config']
  volumeMounts:
  - name: shared-data
    mountPath: /shared

# Set permissions
- name: fix-permissions
  image: busybox
  command: ['chmod', '755', '/app/data']
  volumeMounts:
  - name: app-data
    mountPath: /app/data
```

### 3. What is **DNS** in Kubernetes and how do services communicate with each other?

**Answer:**

**Kubernetes DNS**: Built-in DNS service that provides name resolution for services and pods
- **Automatic**: Every service gets a DNS name automatically
- **Cluster-wide**: Available to all pods in the cluster
- **Service discovery**: Enables services to find each other by name
- **Implementation**: Usually CoreDNS (kube-dns in older versions)

**DNS Naming Convention:**
```
<service-name>.<namespace>.svc.cluster.local
```

**Service Communication Examples:**

**Same Namespace:**
```bash
# Full FQDN
curl http://web-service.default.svc.cluster.local

# Short name (same namespace)
curl http://web-service

# With port
curl http://web-service:80
```

**Different Namespace:**
```bash
# Must include namespace
curl http://database-service.production.svc.cluster.local

# Or with namespace only
curl http://database-service.production
```

**DNS Resolution Hierarchy:**
1. **Service name only**: `web-service` (same namespace)
2. **Service + namespace**: `web-service.production`
3. **Full FQDN**: `web-service.production.svc.cluster.local`

**Pod DNS Names:**
```
<pod-ip-with-dashes>.<namespace>.pod.cluster.local
# Example: 10-244-1-5.default.pod.cluster.local
```

**Testing DNS Resolution:**
```bash
# Test from inside a pod
kubectl exec -it test-pod -- nslookup web-service

# Test cross-namespace
kubectl exec -it test-pod -- nslookup db-service.production

# Test full FQDN  
kubectl exec -it test-pod -- nslookup web-service.default.svc.cluster.local

# Check DNS configuration
kubectl exec -it test-pod -- cat /etc/resolv.conf
```

**DNS Configuration in Pod:**
```bash
# Automatic DNS configuration
nameserver 10.96.0.10        # Cluster DNS service IP
search default.svc.cluster.local svc.cluster.local cluster.local
options ndots:5
```

**Common DNS Issues and Troubleshooting:**
```bash
# Check CoreDNS pods
kubectl get pods -n kube-system -l k8s-app=kube-dns

# Check DNS service
kubectl get svc -n kube-system kube-dns

# Test DNS from troubleshooting pod
kubectl run dns-test --rm -it --image=busybox -- nslookup kubernetes.default
```

### 4. How do you **drain a node** and why would you need to do this?

**Answer:**

**Node Draining**: Process of safely evicting all pods from a node before maintenance or removal
- **Graceful eviction**: Pods are terminated gracefully with proper shutdown
- **Rescheduling**: Pods are automatically rescheduled to other nodes
- **Cordoning**: Node is marked as unschedulable during the process

**Command Syntax:**
```bash
# Basic drain command
kubectl drain <node-name>

# Drain with common options
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data --force

# Drain with timeout
kubectl drain <node-name> --timeout=300s --ignore-daemonsets
```

**Why Drain Nodes:**

1. **Node Maintenance**: OS updates, kernel patches, hardware maintenance
2. **Node Replacement**: Replacing faulty or outdated nodes
3. **Cluster Scaling**: Removing nodes to scale down the cluster
4. **Resource Rebalancing**: Moving workloads for better resource distribution
5. **Security Updates**: Applying security patches that require reboots
6. **Hardware Upgrades**: Upgrading RAM, CPU, or storage

**Drain Process Steps:**

1. **Cordon the node**: Mark as unschedulable
2. **Evict pods**: Gracefully terminate and reschedule pods
3. **Wait for completion**: Ensure all pods are moved
4. **Perform maintenance**: Safe to work on the node
5. **Uncordon**: Make node schedulable again (if keeping the node)

**Common Drain Options:**
```bash
# Ignore DaemonSet pods (they can't be moved)
kubectl drain node1 --ignore-daemonsets

# Delete pods with emptyDir volumes
kubectl drain node1 --delete-emptydir-data

# Force deletion of standalone pods
kubectl drain node1 --force

# Set timeout for drain operation
kubectl drain node1 --timeout=600s

# Dry run to see what would be drained
kubectl drain node1 --dry-run=client
```

**Handling Different Pod Types:**

```bash
# DaemonSet pods - use --ignore-daemonsets
kubectl drain node1 --ignore-daemonsets

# Pods with local storage - use --delete-emptydir-data
kubectl drain node1 --delete-emptydir-data

# Standalone pods (not managed by controller) - use --force
kubectl drain node1 --force

# All together
kubectl drain node1 --ignore-daemonsets --delete-emptydir-data --force
```

**After Maintenance:**
```bash
# Make node schedulable again
kubectl uncordon <node-name>

# Verify node status
kubectl get nodes
kubectl describe node <node-name>
```

**Monitoring Drain Progress:**
```bash
# Watch pods being evicted
kubectl get pods --all-namespaces -o wide --watch

# Check node status
kubectl describe node <node-name>

# View events
kubectl get events --sort-by='.lastTimestamp'
```

### 5. What are **Taints and Tolerations** and how do they affect pod scheduling?

**Answer:**

**Taints and Tolerations**: Mechanism to control which pods can be scheduled on which nodes

**Taints**: Applied to nodes to repel pods that don't have matching tolerations
- **Node-level**: Set on nodes to restrict pod placement
- **Three effects**: NoSchedule, PreferNoSchedule, NoExecute

**Tolerations**: Applied to pods to allow scheduling on tainted nodes
- **Pod-level**: Set on pods to "tolerate" specific taints
- **Matching**: Must match taint key, value, and effect

**Taint Effects:**
- **NoSchedule**: Pods without tolerations won't be scheduled
- **PreferNoSchedule**: Scheduler tries to avoid placing pods (soft)
- **NoExecute**: Existing pods without tolerations are evicted

**Adding Taints to Nodes:**
```bash
# Add taint with NoSchedule effect
kubectl taint nodes node1 key=value:NoSchedule

# Add taint for dedicated workloads
kubectl taint nodes gpu-node1 dedicated=gpu:NoSchedule

# Add taint with NoExecute (evicts existing pods)
kubectl taint nodes node1 maintenance=true:NoExecute

# Remove taint (note the minus sign)
kubectl taint nodes node1 key=value:NoSchedule-
```

**Pod Tolerations:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: tolerant-pod
spec:
  tolerations:
  # Exact match toleration
  - key: "dedicated"
    operator: "Equal"
    value: "gpu"
    effect: "NoSchedule"
  
  # Tolerate any value for key
  - key: "maintenance"
    operator: "Exists"
    effect: "NoExecute"
    tolerationSeconds: 300  # Tolerate for 5 minutes
  
  containers:
  - name: app
    image: nginx
```

**Common Use Cases:**

**1. Dedicated Nodes:**
```bash
# Taint GPU nodes for ML workloads
kubectl taint nodes gpu-node dedicated=ml:NoSchedule
```

```yaml
# ML pod with toleration
tolerations:
- key: "dedicated"
  value: "ml"
  effect: "NoSchedule"
```

**2. Master Node Protection:**
```bash
# Default master taint
kubectl taint nodes master-node node-role.kubernetes.io/master:NoSchedule
```

**3. Node Maintenance:**
```bash
# Prevent new pods during maintenance
kubectl taint nodes node1 maintenance=true:NoSchedule
```

**Toleration Operators:**
- **Equal**: Key, value, and effect must match exactly
- **Exists**: Only key and effect must match (ignores value)

**Examples:**
```yaml
tolerations:
# Exact match
- key: "node-type"
  operator: "Equal" 
  value: "gpu"
  effect: "NoSchedule"

# Key exists (any value)
- key: "maintenance"
  operator: "Exists"
  effect: "NoExecute"

# Tolerate all taints on a key
- key: "dedicated"
  operator: "Exists"

# Tolerate everything (use carefully!)
- operator: "Exists"
```

**Checking Taints:**
```bash
# View node taints
kubectl describe node <node-name> | grep -A5 Taints

# Get node taints in output
kubectl get nodes -o custom-columns=NAME:.metadata.name,TAINTS:.spec.taints
```

### 6. How would you check which **image version** is currently running in a deployment?

**Answer:**

**Multiple Methods to Check Image Versions:**

**1. Basic Deployment Info:**
```bash
# Get deployment with image info
kubectl get deployment <deployment-name> -o wide

# More detailed deployment info
kubectl describe deployment <deployment-name>
```

**2. Direct Image Query:**
```bash
# Get just the image field
kubectl get deployment <deployment-name> -o jsonpath='{.spec.template.spec.containers[*].image}'

# With container names and images
kubectl get deployment <deployment-name> -o jsonpath='{range .spec.template.spec.containers[*]}{.name}{": "}{.image}{"\n"}{end}'
```

**3. Pod-Level Verification:**
```bash
# Check running pods' images
kubectl get pods -l app=<app-label> -o wide

# Get images from all pods of a deployment
kubectl get pods -l app=<app-label> -o jsonpath='{range .items[*]}{.metadata.name}{": "}{.spec.containers[*].image}{"\n"}{end}'
```

**4. Comprehensive Status Check:**
```bash
# Rollout status with image info
kubectl rollout status deployment/<deployment-name>

# Deployment history with images
kubectl rollout history deployment/<deployment-name>

# Specific revision details
kubectl rollout history deployment/<deployment-name> --revision=2
```

**5. YAML/JSON Output:**
```bash
# Full deployment YAML
kubectl get deployment <deployment-name> -o yaml | grep -A5 -B5 image

# JSON output for parsing
kubectl get deployment <deployment-name> -o json | jq '.spec.template.spec.containers[].image'
```

**6. Multiple Containers:**
```bash
# For deployments with multiple containers
kubectl get deployment <deployment-name> -o jsonpath='{.spec.template.spec.containers[*].name}' && echo
kubectl get deployment <deployment-name> -o jsonpath='{.spec.template.spec.containers[*].image}' && echo

# Better formatted for multiple containers
kubectl get deployment <deployment-name> -o jsonpath='{range .spec.template.spec.containers[*]}{.name}{" -> "}{.image}{"\n"}{end}'
```

**7. ReplicaSet Level Check:**
```bash
# Check ReplicaSets (shows current and previous images)
kubectl get rs -l app=<app-label> -o wide

# Get current ReplicaSet image
kubectl get rs -l app=<app-label> -o jsonpath='{.items[0].spec.template.spec.containers[*].image}'
```

**8. Custom Output Formats:**
```bash
# Custom columns for better readability
kubectl get deployments -o custom-columns="NAME:.metadata.name,IMAGE:.spec.template.spec.containers[*].image,REPLICAS:.spec.replicas"

# With ready status
kubectl get deployments -o custom-columns="NAME:.metadata.name,IMAGE:.spec.template.spec.containers[*].image,READY:.status.readyReplicas,TOTAL:.spec.replicas"
```

**9. Watch for Changes:**
```bash
# Monitor image changes during updates
kubectl get deployment <deployment-name> -o wide --watch

# Watch pods during rolling update
kubectl get pods -l app=<app-label> -o wide --watch
```

**10. Script for Multiple Deployments:**
```bash
# Check all deployments in namespace
kubectl get deployments -o jsonpath='{range .items[*]}{.metadata.name}{": "}{.spec.template.spec.containers[*].image}{"\n"}{end}'

# All deployments across all namespaces
kubectl get deployments --all-namespaces -o jsonpath='{range .items[*]}{.metadata.namespace}{"/"}{.metadata.name}{": "}{.spec.template.spec.containers[*].image}{"\n"}{end}'
```

### 7. What's the difference between **kubectl delete** and **kubectl patch** for updating resources?

**Answer:**

**kubectl delete**: Removes resources entirely from the cluster
- **Destructive operation**: Completely removes the resource
- **Recreates from scratch**: If recreated, gets new UID and loses state
- **Immediate**: Resource is marked for deletion immediately
- **Use case**: Remove unwanted resources, clean up, complete replacement

**kubectl patch**: Modifies specific fields of existing resources
- **Non-destructive**: Updates only specified fields
- **Preserves state**: Maintains resource UID and existing state
- **Selective updates**: Changes only the fields you specify
- **Use case**: Update configuration, scale resources, modify labels

**Examples:**

**Delete Operations:**
```bash
# Delete a deployment (removes everything)
kubectl delete deployment web-app

# Delete and recreate from file
kubectl delete -f deployment.yaml
kubectl apply -f deployment.yaml

# Delete multiple resources
kubectl delete pods,services -l app=web-app

# Delete with grace period
kubectl delete pod my-pod --grace-period=30
```

**Patch Operations:**
```bash
# Scale deployment
kubectl patch deployment web-app -p '{"spec":{"replicas":5}}'

# Update image
kubectl patch deployment web-app -p '{"spec":{"template":{"spec":{"containers":[{"name":"web-container","image":"myapp:v2"}]}}}}'

# Add labels
kubectl patch deployment web-app -p '{"metadata":{"labels":{"version":"v2"}}}'

# Update resource limits
kubectl patch deployment web-app -p '{"spec":{"template":{"spec":{"containers":[{"name":"web-container","resources":{"limits":{"memory":"1Gi"}}}]}}}}'
```

**Patch Types:**

**1. Strategic Merge Patch (default):**
```bash
kubectl patch deployment web-app -p '{"spec":{"replicas":3}}'
```

**2. JSON Merge Patch:**
```bash
kubectl patch deployment web-app --type='merge' -p '{"spec":{"replicas":3}}'
```

**3. JSON Patch:**
```bash
kubectl patch deployment web-app --type='json' -p='[{"op": "replace", "path": "/spec/replicas", "value": 3}]'
```

**Key Differences:**

| Aspect | kubectl delete | kubectl patch |
|--------|---------------|---------------|
| **Operation** | Remove resource | Modify resource |
| **Resource UID** | New UID on recreate | Preserves UID |
| **State** | Loses all state | Maintains state |
| **Downtime** | Causes downtime | Usually no downtime |
| **Rollback** | Manual recreation | Can be reverted |
| **Scope** | Entire resource | Specific fields |

**When to Use Each:**

**Use kubectl delete when:**
- Removing unwanted resources
- Complete replacement needed
- Cleaning up test resources
- Resource is corrupted and needs fresh start

**Use kubectl patch when:**
- Scaling applications
- Updating configuration
- Modifying labels or annotations
- Quick configuration changes
- Maintaining resource identity

**Alternative Update Methods:**
```bash
# kubectl edit (interactive)
kubectl edit deployment web-app

# kubectl set (specific updates)
kubectl set image deployment/web-app web-container=myapp:v2

# kubectl scale (replica changes)
kubectl scale deployment web-app --replicas=5

# kubectl apply (declarative)
kubectl apply -f updated-deployment.yaml
```

### 8. Explain what happens when you run **kubectl get pods -o wide** vs **kubectl get pods**.

**Answer:**

**kubectl get pods**: Shows basic pod information in compact format
- **Standard columns**: NAME, READY, STATUS, RESTARTS, AGE
- **Compact view**: Minimal information for quick overview
- **Default output**: Most commonly used format

**kubectl get pods -o wide**: Shows extended pod information with additional columns
- **Additional columns**: IP, NODE, NOMINATED NODE, READINESS GATES
- **Detailed view**: More comprehensive information about pod placement
- **Network info**: Shows pod IPs and node assignments

**Output Comparison:**

**Standard Output:**
```bash
$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
web-app-7d4d8c8f-abc12   1/1     Running   0          5m
web-app-7d4d8c8f-def34   1/1     Running   1          5m
db-pod                   1/1     Running   0          10m
```

**Wide Output:**
```bash
$ kubectl get pods -o wide
NAME                     READY   STATUS    RESTARTS   AGE   IP           NODE     NOMINATED NODE   READINESS GATES
web-app-7d4d8c8f-abc12   1/1     Running   0          5m    10.244.1.5   worker1  <none>           <none>
web-app-7d4d8c8f-def34   1/1     Running   1          5m    10.244.2.3   worker2  <none>           <none>
db-pod                   1/1     Running   0          10m   10.244.1.7   worker1  <none>           <none>
```

**Additional Information in Wide Output:**

**1. IP Address:**
- Shows the internal cluster IP of each pod
- Useful for networking troubleshooting
- Helps identify pod communication issues

**2. NODE:**
- Shows which worker node the pod is running on
- Important for debugging node-specific issues
- Helps with resource distribution analysis

**3. NOMINATED NODE:**
- Shows node where pod is scheduled but not yet running
- Usually shows `<none>` for running pods
- Useful during scheduling conflicts

**4. READINESS GATES:**
- Shows additional readiness conditions
- Usually `<none>` unless custom readiness gates are configured
- Advanced feature for complex readiness logic

**Use Cases for Each:**

**Use `kubectl get pods` when:**
- Quick status check
- Monitoring pod health
- Checking restart counts
- General overview of pod states

**Use `kubectl get pods -o wide` when:**
- Debugging networking issues
- Checking pod distribution across nodes
- Investigating node-specific problems
- Planning node maintenance
- Troubleshooting service connectivity

**Other Useful Output Formats:**
```bash
# YAML output (full resource definition)
kubectl get pods <pod-name> -o yaml

# JSON output (for programmatic parsing)
kubectl get pods <pod-name> -o json

# Custom columns
kubectl get pods -o custom-columns="NAME:.metadata.name,STATUS:.status.phase,NODE:.spec.nodeName,IP:.status.podIP"

# Show labels
kubectl get pods --show-labels

# Specific label columns
kubectl get pods -L app,version

# No headers
kubectl get pods --no-headers

# Sort by age
kubectl get pods --sort-by=.metadata.creationTimestamp
```

**Filtering and Selection:**
```bash
# Combine wide output with selectors
kubectl get pods -l app=web -o wide

# Specific namespace with wide output
kubectl get pods -n production -o wide

# All namespaces with wide output
kubectl get pods --all-namespaces -o wide

# Watch mode with wide output
kubectl get pods -o wide --watch
```

---

## ⚙️ Section 2: Scenario - Answer

**Scenario:**  
Your team has deployed a web application, but some users report intermittent connection timeouts. You notice that sometimes the application works fine, but other times it's unreachable. The pods are showing as "Running" but you suspect there might be networking or readiness issues.  
Walk through your systematic troubleshooting approach to identify and resolve this intermittent connectivity problem.

**Answer:**

This intermittent connectivity issue suggests problems with readiness probes, service endpoints, or load balancing. Here's a systematic troubleshooting approach:

### **Phase 1: Initial Assessment**

**1. Check Overall Pod and Service Status:**
```bash
# Check pod status and distribution
kubectl get pods -l app=web-app -o wide

# Check service and endpoints
kubectl get service web-service -o wide
kubectl get endpoints web-service

# Check recent events
kubectl get events --sort-by='.lastTimestamp' | grep web-app
```

**2. Analyze Pod Health:**
```bash
# Check detailed pod status
kubectl describe pods -l app=web-app

# Look for readiness probe failures
kubectl describe pod <pod-name> | grep -A10 -B5 "Readiness"

# Check restart patterns
kubectl get pods -l app=web-app --watch
```

### **Phase 2: Service and Endpoint Investigation**

**3. Verify Service Configuration:**
```bash
# Check service selector matches pod labels
kubectl describe service web-service

# Verify service endpoints are populated
kubectl describe endpoints web-service

# Check if some pods are missing from endpoints
kubectl get pods -l app=web-app --show-labels
```

**4. Test Service Connectivity:**
```bash
# Test service DNS resolution
kubectl run test-pod --rm -it --image=busybox -- nslookup web-service

# Test direct service connection
kubectl run test-pod --rm -it --image=curlimages/curl -- curl -v web-service:80

# Test individual pod IPs
kubectl run test-pod --rm -it --image=curlimages/curl -- curl -v <pod-ip>:8080
```

### **Phase 3: Deep Dive into Readiness Issues**

**5. Analyze Readiness Probe Configuration:**
```bash
# Check deployment readiness probe settings
kubectl get deployment web-app -o yaml | grep -A15 readinessProbe

# Check if readiness probes are failing
kubectl describe pod <pod-name> | grep -A5 "Readiness probe failed"
```

**6. Test Application Health Endpoints:**
```bash
# Test readiness endpoint directly
kubectl exec -it <pod-name> -- curl localhost:8080/health

# Check application logs for health check errors
kubectl logs <pod-name> | grep -i "health\|ready\|probe"

# Monitor readiness probe attempts
kubectl logs <pod-name> --previous | grep -i probe
```

### **Phase 4: Network and Load Balancing Analysis**

**7. Check Load Balancing Behavior:**
```bash
# Test multiple requests to see distribution
for i in {1..10}; do 
  kubectl run test-$i --rm -it --image=curlimages/curl -- curl -s web-service:80/hostname
done

# Check if all pods are receiving traffic
kubectl logs -l app=web-app --tail=50 | grep -E "GET|POST|request"
```

**8. Investigate Network Issues:**
```bash
# Check network policies
kubectl get networkpolicy --all-namespaces

# Test pod-to-pod communication
kubectl exec -it <pod1> -- curl <pod2-ip>:8080

# Check DNS resolution consistency
kubectl exec -it <pod-name> -- cat /etc/resolv.conf
```

### **Phase 5: Root Cause Analysis**

**Common Issues and Solutions:**

**Issue 1: Readiness Probe Misconfiguration**
```bash
# Check current probe settings
kubectl get deployment web-app -o jsonpath='{.spec.template.spec.containers[0].readinessProbe}'
```

**Likely problems:**
- `initialDelaySeconds` too short
- `periodSeconds` too frequent  
- `timeoutSeconds` too short
- Wrong health check endpoint

**Fix:**
```yaml
readinessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 30    # Increase startup time
  periodSeconds: 10          # Check every 10 seconds
  timeoutSeconds: 5          # 5 second timeout
  failureThreshold: 3        # 3 failures = not ready
  successThreshold: 1        # 1 success = ready
```

**Issue 2: Application Startup Race Condition**
```bash
# Check if app starts before dependencies
kubectl logs <pod-name> | head -20

# Check dependency services
kubectl get pods -l app=database
kubectl get service database-service
```

**Fix with Init Container:**
```yaml
initContainers:
- name: wait-for-db
  image: busybox:1.35
  command: ['sh', '-c']
  args:
  - until nslookup database-service.default.svc.cluster.local; 
    do echo waiting for database; sleep 2; done
```

**Issue 3: Resource Constraints**
```bash
# Check resource usage
kubectl top pods -l app=web-app

# Check resource limits
kubectl describe pods -l app=web-app | grep -A5 Limits
```

**Fix:**
```yaml
resources:
  requests:
    memory: "256Mi"
    cpu: "250m"
  limits:
    memory: "512Mi" 
    cpu: "500m"
```

### **Phase 6: Monitoring and Validation**

**9. Implement Monitoring:**
```bash
# Monitor service endpoints continuously
watch kubectl get endpoints web-service

# Monitor pod readiness
watch kubectl get pods -l app=web-app

# Test connectivity continuously
while true; do curl -s web-service:80 || echo "FAIL"; sleep 1; done
```

**10. Validate Fix:**
```bash
# Apply updated configuration
kubectl apply -f fixed-deployment.yaml

# Monitor rollout
kubectl rollout status deployment/web-app

# Test sustained connectivity
for i in {1..100}; do
  curl -s web-service:80 > /dev/null || echo "Request $i failed"
  sleep 0.1
done
```

### **Prevention Strategies:**

1. **Proper Health Checks:**
   - Implement meaningful readiness endpoints
   - Set appropriate probe timings
   - Test health checks during deployment

2. **Gradual Rollouts:**
   - Use rolling update strategy
   - Set `maxUnavailable: 0` for zero downtime
   - Monitor during deployments

3. **Resource Planning:**
   - Set appropriate resource requests/limits
   - Monitor resource usage patterns
   - Use horizontal pod autoscaling

4. **Dependency Management:**
   - Use init containers for dependencies
   - Implement proper startup ordering
   - Add circuit breakers in application code

---

## 🧩 Section 3: Problem-Solving - Answer

**Task:**  
A developer deployed the following configuration but the application isn't starting properly. The pods start but immediately crash with "connection refused" errors when trying to connect to the database and cache. The database and cache services exist and are working. What's the most likely issue and how would you fix it using Init Containers?

**Answer:**

### **Root Cause Analysis:**

**Primary Issue: Race Condition During Startup**

The application is trying to connect to database and cache services immediately upon startup, but there's no guarantee that:
1. The DNS names are resolvable yet
2. The target services are fully ready to accept connections
3. The network routes are established

This creates a **startup race condition** where the application fails before the dependencies are available.

**Why This Happens:**
- Kubernetes starts pods asynchronously
- Service DNS registration takes time
- Services might exist but pods behind them aren't ready
- Application doesn't implement retry logic for initial connections

### **Solution: Use Init Containers for Dependency Checking**

**Fixed Configuration:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  labels:
    app: web-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      # Init containers run before main containers
      initContainers:
      
      # Wait for database service to be resolvable and ready
      - name: wait-for-database
        image: busybox:1.35
        command: ['sh', '-c']
        args:
        - |
          echo "Waiting for database service..."
          until nslookup db-service.default.svc.cluster.local; do
            echo "Database service not found, waiting..."
            sleep 2
          done
          echo "Database service found, checking connectivity..."
          until nc -z db-service 5432; do
            echo "Database not accepting connections, waiting..."
            sleep 2
          done
          echo "Database is ready!"

      # Wait for cache service to be resolvable and ready  
      - name: wait-for-cache
        image: busybox:1.35
        command: ['sh', '-c']
        args:
        - |
          echo "Waiting for cache service..."
          until nslookup redis-service.default.svc.cluster.local; do
            echo "Cache service not found, waiting..."
            sleep 2
          done
          echo "Cache service found, checking connectivity..."
          until nc -z redis-service 6379; do
            echo "Cache not accepting connections, waiting..."
            sleep 2
          done
          echo "Cache is ready!"

      # Main application container
      containers:
      - name: web-container
        image: myapp:latest
        ports:
        - containerPort: 8080
        env:
        - name: DATABASE_HOST
          value: "db-service"
        - name: CACHE_HOST  
          value: "redis-service"
        
        # Add readiness probe to ensure app is ready before receiving traffic
        readinessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 3
        
        # Add liveness probe for restart if app becomes unresponsive
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        
        # Resource limits to prevent resource issues
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"

---
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  selector:
    app: web-app
  ports:
  - port: 80
    targetPort: 8080
  type: ClusterIP
```

### **Alternative Init Container Approaches:**

**1. Combined Dependency Check:**
```yaml
initContainers:
- name: wait-for-dependencies
  image: busybox:1.35
  command: ['sh', '-c']
  args:
  - |
    # Function to wait for service
    wait_for_service() {
      local service=$1
      local port=$2
      echo "Waiting for $service:$port..."
      until nc -z $service $port; do
        echo "$service not ready, waiting..."
        sleep 2
      done
      echo "$service is ready!"
    }
    
    # Wait for all dependencies
    wait_for_service db-service 5432
    wait_for_service redis-service 6379
    echo "All dependencies ready!"
```

**2. Using Specific Client Tools:**
```yaml
initContainers:
# Database readiness check with actual client
- name: check-database
  image: postgres:13-alpine
  command: ['sh', '-c']
  args:
  - |
    until pg_isready -h db-service -p 5432 -U postgres; do
      echo "Waiting for database..."
      sleep 2
    done
    echo "Database is ready!"

# Redis readiness check with redis client
- name: check-redis
  image: redis:6-alpine
  command: ['sh', '-c']
  args:
  - |
    until redis-cli -h redis-service ping; do
      echo "Waiting for Redis..."
      sleep 2
    done
    echo "Redis is ready!"
```

**3. Advanced Dependency Check with Retries:**
```yaml
initContainers:
- name: wait-for-dependencies
  image: curlimages/curl:7.85.0
  command: ['sh', '-c']
  args:
  - |
    MAX_RETRIES=30
    RETRY_INTERVAL=5
    
    check_service() {
      local service=$1
      local port=$2
      local retries=0
      
      while [ $retries -lt $MAX_RETRIES ]; do
        if nc -z $service $port; then
          echo "$service:$port is ready!"
          return 0
        fi
        echo "Attempt $((retries+1))/$MAX_RETRIES: $service:$port not ready"
        sleep $RETRY_INTERVAL
        retries=$((retries+1))
      done
      
      echo "ERROR: $service:$port not ready after $MAX_RETRIES attempts"
      exit 1
    }
    
    check_service db-service 5432
    check_service redis-service 6379
    echo "All services ready!"
```

### **Additional Improvements:**

**1. Application-Level Health Check:**
```yaml
# Add health endpoint to your application
readinessProbe:
  httpGet:
    path: /ready  # Endpoint that checks all dependencies
    port: 8080
  initialDelaySeconds: 15
  periodSeconds: 5
```

**2. Graceful Degradation:**
```yaml
env:
- name: DB_MAX_RETRIES
  value: "5"
- name: DB_RETRY_INTERVAL
  value: "2"
- name: CACHE_TIMEOUT
  value: "30"
```

**3. Resource Quotas for Dependencies:**
```yaml
resources:
  requests:
    memory: "64Mi"    # Minimal resources for init containers
    cpu: "50m"
  limits:
    memory: "128Mi"
    cpu: "100m"
```

### **Verification Steps:**

```bash
# Deploy and monitor init container progress
kubectl apply -f fixed-deployment.yaml

# Watch init containers complete
kubectl get pods -w

# Check init container logs
kubectl logs <pod-name> -c wait-for-database
kubectl logs <pod-name> -c wait-for-cache

# Verify main container starts successfully
kubectl logs <pod-name> -c web-container

# Test application connectivity
kubectl port-forward service/web-service 8080:80
curl http://localhost:8080/health
```

This solution ensures that the application only starts after all its dependencies are available and ready to accept connections, eliminating the race condition that was causing the crashes.
