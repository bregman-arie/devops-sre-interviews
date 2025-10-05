# Kubernetes - Beginner Interview Mock #3 - Answer Key

> **Difficulty:** Beginner  
> **Duration:** ~30 minutes  
> **Goal:** Assess your understanding of Kubernetes workloads, resource management, security basics, and operational commands.

---

## 🧠 Section 1: Core Questions - Answers

### 1. What is the difference between a **DaemonSet** and a **Deployment**? When would you use each?

**Answer:**

**Deployment**: Manages replicated applications with a specified number of identical pod replicas
- **Scaling**: Can have 0 to N replicas
- **Scheduling**: Pods distributed across available nodes based on resource availability
- **Updates**: Supports rolling updates and rollbacks
- **Use case**: Stateless applications like web servers, APIs, microservices

**DaemonSet**: Ensures exactly one pod runs on each (or selected) node
- **Scaling**: Automatically scales with cluster size (one pod per node)
- **Scheduling**: Pods scheduled on every node (or nodes matching nodeSelector)
- **Updates**: Supports rolling updates but different strategy than Deployments
- **Use case**: Node-level services like logging agents, monitoring agents, network proxies

**Key Differences:**
- **Pod distribution**: Deployment spreads pods for availability; DaemonSet places one per node
- **Scaling behavior**: Deployment scales by replica count; DaemonSet scales with node count
- **Node affinity**: DaemonSet inherently node-aware; Deployment is node-agnostic

**When to use each:**
- **Deployment**: Application servers, databases, processing services
- **DaemonSet**: Log collection (fluentd), monitoring (node-exporter), networking (kube-proxy), security agents

### 2. Explain what **resource requests** and **resource limits** are and why they're important.

**Answer:**

**Resource Requests**: The amount of CPU and memory guaranteed to a container
- **Scheduler uses this** to decide which node can accommodate the pod
- **Guaranteed allocation**: Node reserves this amount for the container
- **QoS classification**: Affects pod priority during resource pressure

**Resource Limits**: The maximum amount of CPU and memory a container can use
- **Upper boundary**: Container cannot exceed these values
- **Protection**: Prevents one container from consuming all node resources
- **Enforcement**: Kernel enforces memory limits; CPU limits throttle usage

**Example:**
```yaml
resources:
  requests:
    memory: "64Mi"    # Guaranteed 64MB
    cpu: "250m"       # Guaranteed 0.25 CPU cores
  limits:
    memory: "128Mi"   # Maximum 128MB (killed if exceeded)
    cpu: "500m"       # Maximum 0.5 CPU cores (throttled if exceeded)
```

**Why Important:**
- **Scheduling efficiency**: Helps scheduler make optimal placement decisions
- **Resource isolation**: Prevents noisy neighbor problems
- **Cluster stability**: Protects against resource exhaustion
- **Cost optimization**: Better resource utilization and capacity planning
- **QoS guarantees**: Enables different service levels (Guaranteed, Burstable, BestEffort)

**Best Practices:**
- Set requests based on actual usage patterns
- Set limits to prevent resource hogging
- Monitor actual usage to tune values
- Start conservative and adjust based on metrics

### 3. What is a **ServiceAccount** and how does it relate to pod security?

**Answer:**

**ServiceAccount**: An identity for processes running in pods to interact with the Kubernetes API
- **Pod identity**: Every pod runs with a ServiceAccount (default if not specified)
- **API access**: Controls what Kubernetes APIs the pod can access
- **Token-based**: Uses JWT tokens for authentication
- **Namespace-scoped**: ServiceAccounts belong to specific namespaces

**Security Relationship:**
- **Authentication**: Identifies the pod to the API server
- **Authorization**: Combined with RBAC to control permissions
- **Least privilege**: Enables fine-grained access control
- **Token management**: Automatic token mounting and rotation

**How it works:**
1. Pod is assigned a ServiceAccount (default or custom)
2. Kubernetes mounts a token at `/var/run/secrets/kubernetes.io/serviceaccount/`
3. Applications use this token to authenticate API requests
4. RBAC rules determine what actions are allowed

**Example Usage:**
```yaml
# Custom ServiceAccount
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-service-account
---
# Pod using custom ServiceAccount
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  serviceAccountName: my-service-account
  containers:
  - name: app
    image: my-app
```

**RBAC Integration:**
```yaml
# Role defining permissions
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
---
# Bind role to ServiceAccount
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
subjects:
- kind: ServiceAccount
  name: my-service-account
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

### 4. How do **Jobs** and **CronJobs** work in Kubernetes? What are their use cases?

**Answer:**

**Jobs**: Run pods to completion for batch/one-time tasks
- **Completion-based**: Runs until specified number of successful completions
- **Retry logic**: Automatically retries failed pods based on configuration
- **Cleanup**: Can be configured to clean up completed pods
- **Parallelism**: Supports parallel execution of multiple pods

**CronJobs**: Schedule Jobs to run at specific times (like Unix cron)
- **Time-based scheduling**: Uses cron syntax for scheduling
- **Job management**: Creates Job objects at scheduled times
- **History management**: Keeps configurable number of successful/failed jobs
- **Concurrency control**: Can prevent overlapping job executions

**Job Example:**
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: data-migration
spec:
  completions: 1          # Run until 1 successful completion
  parallelism: 3          # Run up to 3 pods in parallel
  backoffLimit: 3         # Retry up to 3 times on failure
  template:
    spec:
      containers:
      - name: migrator
        image: data-migrator:v1
      restartPolicy: Never  # Important: Never or OnFailure only
```

**CronJob Example:**
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: backup-job
spec:
  schedule: "0 2 * * *"   # Daily at 2 AM
  concurrencyPolicy: Forbid
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 1
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: backup-tool:v1
          restartPolicy: OnFailure
```

**Use Cases:**

**Jobs:**
- Database migrations
- Data processing/ETL
- Report generation
- Batch computations
- One-time maintenance tasks

**CronJobs:**
- Regular backups
- Log rotation
- Certificate renewal
- Periodic cleanup tasks
- Scheduled reports
- Health checks

### 5. What are **Ingress** resources and how do they differ from Services?

**Answer:**

**Ingress**: Layer 7 (HTTP/HTTPS) load balancer and router that manages external access to services
- **Protocol**: Works with HTTP/HTTPS traffic only
- **Routing**: Advanced routing based on hostnames, paths, headers
- **SSL/TLS**: Built-in SSL termination and certificate management
- **Single entry point**: One external IP can route to multiple services

**Services**: Layer 4 load balancer that provides stable networking for pods
- **Protocol**: Works with any TCP/UDP traffic
- **Load balancing**: Simple round-robin load balancing
- **Discovery**: Internal service discovery and DNS
- **Types**: ClusterIP, NodePort, LoadBalancer

**Key Differences:**

| Feature | Service | Ingress |
|---------|---------|---------|
| **OSI Layer** | Layer 4 (Transport) | Layer 7 (Application) |
| **Protocols** | TCP, UDP, any | HTTP, HTTPS only |
| **Routing** | Simple load balancing | Path/host-based routing |
| **SSL** | Passthrough only | SSL termination |
| **External Access** | LoadBalancer/NodePort | Ingress Controller |
| **Cost** | One LoadBalancer per service | One LoadBalancer for all services |

**Ingress Example:**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  tls:
  - hosts:
    - myapp.example.com
    secretName: tls-secret
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

**When to use:**
- **Service**: Internal communication, simple external access
- **Ingress**: Multiple web applications, complex routing, SSL management, cost optimization

### 6. How would you view the logs from all containers in a multi-container pod?

**Answer:**

**For all containers in a pod:**
```bash
# View logs from all containers (requires kubectl 1.16+)
kubectl logs <pod-name> --all-containers=true

# Follow logs from all containers
kubectl logs <pod-name> --all-containers=true -f

# Include timestamps
kubectl logs <pod-name> --all-containers=true --timestamps=true

# Previous container instances (if crashed)
kubectl logs <pod-name> --all-containers=true --previous
```

**For specific containers:**
```bash
# List containers in a pod
kubectl describe pod <pod-name> | grep "Container ID"

# View logs from specific container
kubectl logs <pod-name> -c <container-name>

# Follow specific container logs
kubectl logs <pod-name> -c <container-name> -f

# Multiple terminals for different containers
kubectl logs <pod-name> -c container1 -f &
kubectl logs <pod-name> -c container2 -f &
```

**Advanced log viewing:**
```bash
# Last N lines from all containers
kubectl logs <pod-name> --all-containers=true --tail=50

# Logs since specific time
kubectl logs <pod-name> --all-containers=true --since=1h

# Logs with container name prefix
kubectl logs <pod-name> --all-containers=true --prefix=true

# Export logs to file
kubectl logs <pod-name> --all-containers=true > pod-logs.txt
```

**Using kubectl stern (third-party tool):**
```bash
# Install stern for better multi-container log viewing
# stern <pod-name>  # Shows all containers with color coding
```

### 7. What command would you use to create a pod imperatively (without YAML) for quick testing?

**Answer:**

**Basic pod creation:**
```bash
# Create simple pod
kubectl run test-pod --image=nginx

# Create pod with specific port
kubectl run web-pod --image=nginx --port=80

# Create pod with environment variables
kubectl run app-pod --image=busybox --env="VAR1=value1" --env="VAR2=value2"

# Create pod with command
kubectl run debug-pod --image=busybox -- sleep 3600

# Create pod with interactive shell
kubectl run -it debug-pod --image=busybox -- /bin/sh
```

**Advanced options:**
```bash
# Create pod with resource limits
kubectl run resource-pod --image=nginx --requests="cpu=100m,memory=128Mi" --limits="cpu=200m,memory=256Mi"

# Create pod with labels
kubectl run labeled-pod --image=nginx --labels="app=test,env=dev"

# Create pod in specific namespace
kubectl run ns-pod --image=nginx -n development

# Create pod with restart policy
kubectl run job-pod --image=busybox --restart=Never -- echo "Hello World"

# Create pod and expose as service
kubectl run web-pod --image=nginx --port=80 --expose
```

**Dry-run to generate YAML:**
```bash
# Generate YAML without creating the pod
kubectl run my-pod --image=nginx --dry-run=client -o yaml > pod.yaml

# Generate and apply
kubectl run my-pod --image=nginx --dry-run=client -o yaml | kubectl apply -f -

# Generate with additional options
kubectl run complex-pod --image=nginx --port=80 --env="ENV=prod" --labels="app=web" --dry-run=client -o yaml
```

**Quick testing patterns:**
```bash
# Temporary pod for testing (auto-deleted)
kubectl run temp-pod --rm -it --image=busybox -- /bin/sh

# Network debugging pod
kubectl run netshoot --rm -it --image=nicolaka/netshoot -- /bin/bash

# Curl testing pod
kubectl run curl-test --rm -it --image=curlimages/curl -- /bin/sh
```

### 8. What happens during a **rolling update** and how can you monitor its progress?

**Answer:**

**Rolling Update Process:**
1. **Gradual replacement**: New pods created with updated configuration
2. **Health verification**: New pods must pass readiness probes
3. **Traffic switching**: Service gradually routes traffic to new pods
4. **Old pod termination**: Old pods terminated after new ones are ready
5. **Controlled pace**: Respects maxSurge and maxUnavailable settings

**Rolling Update Strategy:**
```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 25%        # Max 25% extra pods during update
      maxUnavailable: 25%   # Max 25% pods can be unavailable
```

**Monitoring Commands:**

**Real-time update status:**
```bash
# Watch rollout status
kubectl rollout status deployment/<deployment-name>

# Watch rollout with timeout
kubectl rollout status deployment/<deployment-name> --timeout=300s

# Continuous monitoring
watch kubectl get pods -l app=<app-label>
```

**Detailed monitoring:**
```bash
# Check deployment status
kubectl get deployment <deployment-name> -o wide

# Watch pod changes in real-time
kubectl get pods -l app=<app-label> -w

# Check ReplicaSet status
kubectl get rs -l app=<app-label>

# View rollout history
kubectl rollout history deployment/<deployment-name>

# Describe deployment for events
kubectl describe deployment <deployment-name>
```

**Update progress indicators:**
```bash
# Deployment status shows:
# - READY: current ready replicas vs desired
# - UP-TO-DATE: replicas with latest template
# - AVAILABLE: available replicas
kubectl get deployment

NAME       READY   UP-TO-DATE   AVAILABLE   AGE
my-app     2/3     2            2           5m
```

**Rollout control:**
```bash
# Pause rollout (stop creating new pods)
kubectl rollout pause deployment/<deployment-name>

# Resume rollout
kubectl rollout resume deployment/<deployment-name>

# Rollback to previous version
kubectl rollout undo deployment/<deployment-name>

# Rollback to specific revision
kubectl rollout undo deployment/<deployment-name> --to-revision=2
```

**Monitoring best practices:**
- Use readiness probes to ensure proper health checking
- Monitor application metrics during updates
- Set appropriate timeouts for rollout status
- Use labels to track pod versions during updates
- Consider using tools like Argo Rollouts for advanced deployment strategies

---

## ⚙️ Section 2: Scenario - Answer

**Scenario:**  
You need to deploy a logging agent that must run on every node in your Kubernetes cluster to collect logs from all pods. The agent should automatically be deployed to new nodes when they join the cluster.  
Explain what Kubernetes resource you would use and walk through the key configuration considerations.

**Answer:**

**Resource Choice: DaemonSet**

A **DaemonSet** is the correct Kubernetes resource for this use case because it ensures exactly one pod runs on each node and automatically deploys to new nodes when they join the cluster.

### **Key Configuration Considerations:**

### **1. Basic DaemonSet Structure**
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: log-collector
  namespace: kube-system  # System namespace for cluster services
  labels:
    app: log-collector
spec:
  selector:
    matchLabels:
      app: log-collector
  template:
    metadata:
      labels:
        app: log-collector
    spec:
      containers:
      - name: log-agent
        image: fluent/fluentd:v1.14
```

### **2. Volume Mounts for Log Access**
```yaml
spec:
  template:
    spec:
      volumes:
      # Access to pod logs
      - name: varlog
        hostPath:
          path: /var/log
      # Access to container logs
      - name: dockercontainers
        hostPath:
          path: /var/lib/docker/containers
      # Access to systemd logs
      - name: varlibdockercontainers
        hostPath:
          path: /run/log/journal
      
      containers:
      - name: log-agent
        volumeMounts:
        - name: varlog
          mountPath: /var/log
          readOnly: true
        - name: dockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
        - name: varlibdockercontainers
          mountPath: /run/log/journal
          readOnly: true
```

### **3. Security Configuration**
```yaml
spec:
  template:
    spec:
      serviceAccountName: log-collector
      securityContext:
        runAsUser: 0  # Root access needed for log files
      
      containers:
      - name: log-agent
        securityContext:
          privileged: true  # Access to host filesystem
```

### **4. Resource Management**
```yaml
containers:
- name: log-agent
  resources:
    requests:
      memory: "100Mi"
      cpu: "100m"
    limits:
      memory: "200Mi"
      cpu: "200m"
```

### **5. Node Selection (if needed)**
```yaml
spec:
  template:
    spec:
      # Only run on nodes with specific labels
      nodeSelector:
        kubernetes.io/os: linux
      
      # Or use node affinity for more complex selection
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: node-type
                operator: NotIn
                values:
                - master
```

### **6. Tolerations for System Nodes**
```yaml
spec:
  template:
    spec:
      # Allow scheduling on master/tainted nodes
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      - key: node-role.kubernetes.io/control-plane
        effect: NoSchedule
      - operator: Exists
        effect: NoExecute
      - operator: Exists
        effect: NoSchedule
```

### **7. Update Strategy**
```yaml
spec:
  updateStrategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1  # Update one node at a time
```

### **Complete Example:**
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: log-collector
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: log-collector
rules:
- apiGroups: [""]
  resources: ["pods", "namespaces"]
  verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: log-collector
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: log-collector
subjects:
- kind: ServiceAccount
  name: log-collector
  namespace: kube-system
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: log-collector
  namespace: kube-system
spec:
  selector:
    matchLabels:
      app: log-collector
  updateStrategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
  template:
    metadata:
      labels:
        app: log-collector
    spec:
      serviceAccountName: log-collector
      tolerations:
      - operator: Exists
        effect: NoSchedule
      - operator: Exists
        effect: NoExecute
      
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: dockercontainers
        hostPath:
          path: /var/lib/docker/containers
      
      containers:
      - name: log-agent
        image: fluent/fluentd:v1.14
        resources:
          requests:
            memory: "100Mi"
            cpu: "100m"
          limits:
            memory: "200Mi"
            cpu: "200m"
        volumeMounts:
        - name: varlog
          mountPath: /var/log
          readOnly: true
        - name: dockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
        env:
        - name: NODE_NAME
          valueFrom:
            fieldRef:
              fieldPath: spec.nodeName
```

### **Operational Benefits:**
- **Automatic scaling**: New nodes automatically get logging agents
- **High availability**: Node failures don't affect other nodes' logging
- **Resource efficiency**: One agent per node, not per pod
- **Consistent coverage**: Ensures no nodes are missed
- **Easy updates**: Rolling updates maintain logging coverage

---

## 🧩 Section 3: Problem-Solving - Answer

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

**Answer:**

### **Issues Identified:**

### **1. Incorrect Restart Policy (Critical)**
**Problem**: `restartPolicy: Always` is invalid for Jobs and causes continuous pod recreation.

**Explanation**: 
- Jobs expect pods to complete and exit
- `Always` restart policy keeps restarting completed containers
- Valid policies for Jobs: `Never` or `OnFailure`

### **2. Missing Job Completion Configuration**
**Problem**: No `completions` or `parallelism` specified.

**Issues**:
- Job doesn't know when it's "complete"
- Defaults may not match expected behavior
- No control over parallel execution

### **3. No Resource Management**
**Problem**: No resource requests/limits specified.

**Risks**:
- Unpredictable resource usage
- Potential node resource exhaustion
- No scheduling guarantees

### **4. No Failure Handling**
**Problem**: No `backoffLimit` specified for failure retry logic.

### **5. No Cleanup Configuration**
**Problem**: Completed pods will accumulate over time.

### **Corrected Configuration:**

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: data-processor
  labels:
    app: data-processor
    batch: data-processing
spec:
  # Job completion configuration
  completions: 1              # Run until 1 successful completion
  parallelism: 1              # Run 1 pod at a time
  backoffLimit: 3             # Retry up to 3 times on failure
  
  # Cleanup configuration
  ttlSecondsAfterFinished: 3600  # Delete job 1 hour after completion
  
  # Pod template
  template:
    metadata:
      labels:
        app: data-processor
    spec:
      # FIXED: Correct restart policy for Jobs
      restartPolicy: Never     # Don't restart containers on completion
      
      containers:
      - name: processor
        image: data-processor:v1
        command: ["python", "process_data.py"]
        
        # Resource management
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "1Gi"
            cpu: "1000m"
        
        # Environment configuration
        env:
        - name: LOG_LEVEL
          value: "INFO"
        - name: BATCH_SIZE
          value: "1000"
        
        # Optional: Volume mounts for data access
        volumeMounts:
        - name: data-volume
          mountPath: /data
          readOnly: true
        - name: output-volume
          mountPath: /output
      
      # Volumes for data processing
      volumes:
      - name: data-volume
        persistentVolumeClaim:
          claimName: input-data-pvc
      - name: output-volume
        persistentVolumeClaim:
          claimName: output-data-pvc
```

### **Alternative Configuration for Different Use Cases:**

### **For Parallel Processing:**
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: parallel-data-processor
spec:
  completions: 10           # Process 10 batches total
  parallelism: 3            # Run 3 pods in parallel
  backoffLimit: 2
  template:
    spec:
      restartPolicy: OnFailure  # Restart failed containers
      containers:
      - name: processor
        image: data-processor:v1
        command: ["python", "process_data.py"]
        env:
        - name: JOB_COMPLETION_INDEX
          valueFrom:
            fieldRef:
              fieldPath: metadata.annotations['batch.kubernetes.io/job-completion-index']
```

### **For Retry on Failure:**
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: resilient-data-processor
spec:
  completions: 1
  backoffLimit: 5             # More retries for unreliable processing
  activeDeadlineSeconds: 3600 # Kill job if running longer than 1 hour
  template:
    spec:
      restartPolicy: OnFailure # Restart container on failure (not pod)
      containers:
      - name: processor
        image: data-processor:v1
        command: ["python", "process_data.py"]
        # Liveness probe to detect hung processes
        livenessProbe:
          exec:
            command:
            - pgrep
            - python
          initialDelaySeconds: 60
          periodSeconds: 30
```

### **Key Fixes Explained:**

1. **Restart Policy**: Changed to `Never` so containers don't restart after successful completion
2. **Completions**: Set to 1 so job completes after one successful run
3. **Parallelism**: Set to 1 for single-threaded processing (adjust as needed)
4. **Backoff Limit**: Added retry logic for failed pods
5. **TTL**: Automatic cleanup of completed jobs
6. **Resources**: Proper resource allocation and limits
7. **Labels**: Better organization and selection
8. **Volumes**: Added for data input/output (common in batch jobs)

### **Best Practices for Jobs:**
- Always use `Never` or `OnFailure` restart policies
- Set appropriate `backoffLimit` for failure tolerance
- Use `activeDeadlineSeconds` to prevent runaway jobs
- Consider `ttlSecondsAfterFinished` for automatic cleanup
- Add resource limits to prevent resource exhaustion
- Use volumes for data persistence and sharing
- Monitor job status with `kubectl get jobs` and `kubectl describe job`
