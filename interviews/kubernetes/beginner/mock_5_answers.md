# Kubernetes - Beginner Interview Mock #5 - Answer Key

> **Difficulty:** Beginner  
> **Duration:** ~30 minutes  
> **Goal:** Assess your understanding of Kubernetes security basics, pod lifecycle management, debugging techniques, and cluster administration fundamentals.

---

## 🧠 Section 1: Core Questions - Answers

### 1. What is **RBAC** (Role-Based Access Control) in Kubernetes and why is it important?

**Answer:**

**RBAC (Role-Based Access Control)**: Security mechanism that regulates access to Kubernetes resources based on the roles assigned to users, groups, or service accounts.

**Core Components:**
- **Subjects**: Users, Groups, or ServiceAccounts
- **Roles/ClusterRoles**: Define what actions are allowed
- **RoleBindings/ClusterRoleBindings**: Connect subjects to roles

**Key Concepts:**
```yaml
# Role - namespace-scoped
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: development
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]

---
# RoleBinding - assigns role to subject
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: development
subjects:
- kind: User
  name: jane
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

**Why RBAC is Important:**

1. **Security**: Principle of least privilege - users get only necessary permissions
2. **Compliance**: Meets regulatory requirements for access control
3. **Auditing**: Track who can do what in the cluster
4. **Multi-tenancy**: Safe resource sharing between teams
5. **Operational safety**: Prevents accidental resource deletion/modification

**Common RBAC Patterns:**
- **Developer role**: Read-only access to applications
- **DevOps role**: Full access to deployments and services
- **Admin role**: Cluster-wide administrative access
- **Service account roles**: Application-specific permissions

### 2. Explain the difference between **StatefulSet** and **Deployment**. When would you use each?

**Answer:**

**StatefulSet vs Deployment** - Different workload controllers for different application types:

**Deployment**:
- **Stateless applications**: No persistent identity or storage requirements
- **Interchangeable pods**: Any pod can handle any request
- **Rolling updates**: Pods updated in parallel
- **Random naming**: Pods get random suffixes (app-123abc-xyz)
- **No ordering**: No guarantees about startup/shutdown order

**StatefulSet**:
- **Stateful applications**: Require persistent identity and ordered deployment
- **Unique identity**: Each pod has stable, predictable name (app-0, app-1, app-2)
- **Ordered operations**: Sequential startup, updates, and termination
- **Persistent storage**: Each pod can have its own PersistentVolume
- **Stable network**: Predictable DNS names for each pod

**Example StatefulSet:**
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: database
spec:
  serviceName: "database-headless"
  replicas: 3
  selector:
    matchLabels:
      app: database
  template:
    metadata:
      labels:
        app: database
    spec:
      containers:
      - name: db
        image: postgres:13
        volumeMounts:
        - name: data
          mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 10Gi
```

**When to use each:**

**Use Deployment for:**
- Web servers (nginx, Apache)
- Microservices and APIs
- Worker processes
- Load balancers
- Any stateless application

**Use StatefulSet for:**
- Databases (PostgreSQL, MySQL, MongoDB)
- Message brokers (Kafka, RabbitMQ)
- Distributed systems (Elasticsearch, Cassandra)
- Applications requiring stable identity or ordered startup

### 3. What are **environment variables** and **volumes** in Kubernetes? How do they differ in terms of configuration management?

**Answer:**

**Environment Variables** and **Volumes** are different methods for providing configuration and data to containers:

**Environment Variables:**
- **Simple key-value pairs** injected into container processes
- **Available at runtime** through standard ENV mechanism
- **Immutable** once pod starts (require pod restart to change)
- **Limited size** (typically small values)

**Volumes:**
- **File system mounts** that provide access to data and configuration files
- **Dynamic updates** possible for some volume types
- **Large data support** (files, directories, entire filesystems)
- **Persistent or ephemeral** depending on volume type

**Configuration Management Differences:**

**Environment Variables Example:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: myapp:latest
    env:
    - name: DATABASE_URL
      value: "postgresql://db:5432/myapp"
    - name: LOG_LEVEL
      value: "INFO"
    - name: API_KEY
      valueFrom:
        secretKeyRef:
          name: app-secret
          key: api-key
```

**Volumes Example:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: myapp:latest
    volumeMounts:
    - name: config
      mountPath: /etc/config
    - name: data
      mountPath: /var/data
  volumes:
  - name: config
    configMap:
      name: app-config
  - name: data
    persistentVolumeClaim:
      claimName: app-data-pvc
```

**Key Differences:**

| Aspect | Environment Variables | Volumes |
|--------|----------------------|---------|
| **Data Type** | Simple strings | Files, directories |
| **Size Limit** | Small (few KB) | Large (GBs, TBs) |
| **Updates** | Require pod restart | Can be dynamic |
| **Security** | Visible in process list | File-level permissions |
| **Use Cases** | Simple config, flags | Complex config, data persistence |
| **Performance** | Fastest access | File I/O dependent |

**Best Practices:**
- **Environment variables**: Simple configuration, feature flags, connection strings
- **ConfigMaps as volumes**: Complex configuration files that may need updates
- **Secrets as volumes**: Sensitive files with proper permissions
- **PersistentVolumes**: Application data that must survive pod restarts

### 4. How do **HorizontalPodAutoscaler (HPA)** work and what metrics can trigger scaling?

**Answer:**

**HorizontalPodAutoscaler (HPA)**: Kubernetes controller that automatically scales the number of pod replicas based on observed metrics.

**How HPA Works:**

1. **Monitoring**: Continuously monitors specified metrics
2. **Calculation**: Compares current metrics with target values
3. **Decision**: Determines if scaling up/down is needed
4. **Action**: Adjusts replica count in target resource (Deployment/StatefulSet)
5. **Cooldown**: Waits for stabilization before next scaling action

**HPA Configuration Example:**
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
        periodSeconds: 15
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 10
        periodSeconds: 60
```

**Metrics that can trigger scaling:**

**1. Resource Metrics (Built-in):**
- **CPU utilization**: Percentage of requested CPU
- **Memory utilization**: Percentage of requested memory

**2. Custom Metrics:**
- **Application metrics**: Request rate, queue length, response time
- **Business metrics**: Orders per second, active users
- **Infrastructure metrics**: Database connections, cache hit ratio

**3. External Metrics:**
- **Cloud provider metrics**: AWS CloudWatch, GCP Monitoring
- **Third-party services**: Datadog, New Relic, Prometheus

**Scaling Algorithm:**
```
desiredReplicas = ceil[currentReplicas * (currentMetricValue / desiredMetricValue)]
```

**Key Behaviors:**
- **Scale-up**: Can be aggressive (double pods quickly)
- **Scale-down**: Conservative (gradual reduction to avoid thrashing)
- **Stabilization windows**: Prevents rapid scaling fluctuations
- **Multiple metrics**: Uses the largest scaling recommendation

**Requirements:**
- Resource requests must be defined in pods
- Metrics server must be installed in cluster
- Target resource must be scalable (Deployment, ReplicaSet, StatefulSet)

### 5. What is the purpose of **annotations** vs **labels** in Kubernetes resources?

**Answer:**

**Labels** and **Annotations** are both metadata mechanisms, but serve different purposes:

**Labels:**
- **Selectable metadata** used for grouping and selecting resources
- **Queryable**: Used by selectors for services, deployments, etc.
- **Limited character set**: Alphanumeric, dashes, underscores, dots
- **Size limit**: 63 characters for values
- **Operational**: Used by Kubernetes controllers for decisions

**Annotations:**
- **Non-selectable metadata** for storing arbitrary information
- **Not queryable**: Cannot be used in selectors
- **Flexible format**: Any UTF-8 string
- **Larger size**: Can store more detailed information
- **Informational**: Used by tools, users, and external systems

**Labels Example:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  labels:
    app: web-app                    # Application name
    version: v1.2.3                 # Version identifier
    tier: frontend                  # Application tier
    environment: production         # Environment
    component: web-server           # Component type
spec:
  selector:
    matchLabels:
      app: web-app                  # Used by deployment to find pods
      version: v1.2.3
  template:
    metadata:
      labels:
        app: web-app                # Pod labels for selection
        version: v1.2.3
        tier: frontend
```

**Annotations Example:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  annotations:
    kubernetes.io/change-cause: "Update to version 1.2.3 for bug fixes"
    deployment.kubernetes.io/revision: "5"
    contact.email: "team@company.com"
    documentation.url: "https://docs.company.com/web-app"
    monitoring.alert-rules: |
      - alert: HighCPU
        expr: cpu_usage > 80%
        for: 5m
    build.info: |
      {
        "git_commit": "abc123def456",
        "build_date": "2023-10-06T10:30:00Z",
        "jenkins_build": "456"
      }
```

**Common Use Cases:**

**Labels:**
- Service selectors: `selector: {app: web-app}`
- Deployment targeting: `matchLabels: {app: web-app}`
- Node selection: `nodeSelector: {hardware: ssd}`
- Resource filtering: `kubectl get pods -l environment=production`
- Grouping related resources

**Annotations:**
- Build information (commit hash, build number)
- Contact information (team email, documentation)
- Tool configuration (monitoring, ingress controllers)
- Change tracking (kubectl apply stores last-applied-configuration)
- External system integration

**Key Differences:**

| Aspect | Labels | Annotations |
|--------|--------|-------------|
| **Purpose** | Selection & grouping | Information storage |
| **Queryable** | Yes (selectors) | No |
| **Size limit** | 63 chars (value) | 256 KB (total) |
| **Format** | Restricted charset | Any UTF-8 |
| **Used by** | Kubernetes controllers | Tools & humans |
| **Examples** | app, version, tier | build info, contacts |

### 6. How would you **copy files** to and from a running pod?

**Answer:**

Use **kubectl cp** command to copy files between local machine and running pods:

**Basic Syntax:**
```bash
# Copy FROM pod TO local
kubectl cp <pod-name>:<source-path> <destination-path> -n <namespace>

# Copy FROM local TO pod
kubectl cp <source-path> <pod-name>:<destination-path> -n <namespace>
```

**Detailed Examples:**

**1. Copy from Pod to Local:**
```bash
# Copy single file from pod
kubectl cp web-app-pod:/app/config.json ./config.json

# Copy entire directory from pod
kubectl cp web-app-pod:/app/logs ./local-logs/

# Specify namespace
kubectl cp web-app-pod:/var/log/app.log ./app.log -n production

# Multi-container pod - specify container
kubectl cp web-app-pod:/app/data.db ./backup.db -c app-container
```

**2. Copy from Local to Pod:**
```bash
# Copy single file to pod
kubectl cp ./config.json web-app-pod:/app/config.json

# Copy directory to pod
kubectl cp ./static-files web-app-pod:/app/static/

# Copy with namespace and container specification
kubectl cp ./update.zip web-app-pod:/tmp/update.zip -n production -c main-container
```

**3. Advanced Usage:**

**Copy with compression (for large files):**
```bash
# Create archive and copy
tar czf - /local/directory | kubectl exec -i pod-name -- tar xzf - -C /remote/directory

# Copy and extract
kubectl exec pod-name -- tar czf - /remote/directory | tar xzf - -C /local/directory
```

**Copy between pods:**
```bash
# Via local intermediate step
kubectl cp source-pod:/app/data.json ./temp-data.json
kubectl cp ./temp-data.json dest-pod:/app/data.json

# Direct transfer using tar
kubectl exec source-pod -- tar czf - /app/data | kubectl exec -i dest-pod -- tar xzf - -C /app/
```

**4. Practical Use Cases:**

**Configuration Updates:**
```bash
# Update config file
kubectl cp ./new-config.yaml app-pod:/etc/config/config.yaml
kubectl exec app-pod -- pkill -HUP main-process  # Reload config
```

**Log Collection:**
```bash
# Collect logs for analysis
kubectl cp app-pod:/var/log/application.log ./logs/app-$(date +%Y%m%d).log
```

**Database Backup:**
```bash
# Export database and copy out
kubectl exec postgres-pod -- pg_dump mydb > /tmp/backup.sql
kubectl cp postgres-pod:/tmp/backup.sql ./backup-$(date +%Y%m%d).sql
```

**Important Notes:**
- **Pod must be running**: cp only works with running pods
- **Container specification**: Use `-c` flag for multi-container pods
- **Permissions**: Ensure proper file permissions in destination
- **Large files**: Consider using tar with compression for efficiency
- **Security**: Be careful copying sensitive data
- **Alternative**: Consider using volumes or init containers for regular file transfers

### 7. What command would you use to **port-forward** to access a pod locally for debugging?

**Answer:**

Use **kubectl port-forward** to create a secure tunnel from local machine to a pod:

**Basic Syntax:**
```bash
kubectl port-forward <pod-name> <local-port>:<pod-port> -n <namespace>
```

**Common Port-Forward Examples:**

**1. Basic Port Forwarding:**
```bash
# Forward local port 8080 to pod port 80
kubectl port-forward web-app-pod 8080:80

# Forward to same port (8080 -> 8080)
kubectl port-forward web-app-pod 8080

# Access via: http://localhost:8080
```

**2. Database Access:**
```bash
# PostgreSQL database access
kubectl port-forward postgres-pod 5432:5432 -n database

# Then connect locally:
# psql -h localhost -p 5432 -U username database_name

# MySQL with different local port
kubectl port-forward mysql-pod 3307:3306
# mysql -h localhost -P 3307 -u root -p
```

**3. Multiple Ports:**
```bash
# Forward multiple ports simultaneously
kubectl port-forward app-pod 8080:80 8443:443

# Access HTTP: http://localhost:8080
# Access HTTPS: https://localhost:8443
```

**4. Service Port Forwarding:**
```bash
# Forward to service instead of specific pod
kubectl port-forward service/web-service 8080:80

# This automatically selects an available pod from the service
```

**5. Deployment Port Forwarding:**
```bash
# Forward to deployment (selects random pod)
kubectl port-forward deployment/web-app 8080:80
```

**Advanced Options:**

**Bind to Specific Interface:**
```bash
# Bind to all interfaces (allow external access)
kubectl port-forward --address 0.0.0.0 web-app-pod 8080:80

# Bind to specific IP
kubectl port-forward --address 192.168.1.100 web-app-pod 8080:80
```

**Background Process:**
```bash
# Run port-forward in background
kubectl port-forward web-app-pod 8080:80 &
PF_PID=$!

# Later, kill the port-forward
kill $PF_PID
```

**Real-World Debugging Scenarios:**

**1. Debug Application Issues:**
```bash
# Access application directly
kubectl port-forward app-pod 8080:8080
curl http://localhost:8080/health

# Check metrics endpoint
kubectl port-forward monitoring-pod 9090:9090
# Open http://localhost:9090/metrics in browser
```

**2. Database Debugging:**
```bash
# Connect to database for queries
kubectl port-forward db-pod 5432:5432 &

# Run debugging queries
psql -h localhost -p 5432 -c "SELECT * FROM users LIMIT 5;"
```

**3. Log Analysis Tools:**
```bash
# Access Elasticsearch for log analysis
kubectl port-forward elasticsearch-pod 9200:9200

# Query logs
curl "http://localhost:9200/_search?q=error"
```

**Security Considerations:**
- **Local only**: By default, only accessible from localhost
- **Temporary**: Connection ends when command is stopped
- **Authentication**: Inherits your kubectl authentication
- **Secure tunnel**: Traffic is encrypted through Kubernetes API

**Troubleshooting:**
```bash
# Check if pod is running and port is correct
kubectl get pod web-app-pod -o wide
kubectl describe pod web-app-pod

# Test if port is actually listening in pod
kubectl exec web-app-pod -- netstat -tlnp

# Check for port conflicts locally
netstat -tlnp | grep 8080
lsof -i :8080
```

### 8. Explain what **multi-container pods** are and provide a common use case pattern.

**Answer:**

**Multi-container pods**: Pods that run multiple containers sharing the same network namespace, storage volumes, and lifecycle.

**Key Characteristics:**
- **Shared network**: All containers use same IP and port space
- **Shared storage**: Can mount same volumes
- **Atomic unit**: Scheduled, started, and stopped together
- **Tight coupling**: Designed for containers that work closely together
- **Same node**: All containers always run on same worker node

**Container Patterns:**

**1. Sidecar Pattern** (Most Common):
Helper container that enhances or extends main application container.

**Example: Log Collection Sidecar**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-app-with-logging
spec:
  containers:
  # Main application container
  - name: web-app
    image: nginx:latest
    ports:
    - containerPort: 80
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log/nginx
  
  # Sidecar container for log shipping
  - name: log-shipper
    image: fluent/fluent-bit:latest
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log/nginx
      readOnly: true
    - name: fluent-bit-config
      mountPath: /fluent-bit/etc
  
  volumes:
  - name: shared-logs
    emptyDir: {}
  - name: fluent-bit-config
    configMap:
      name: fluent-bit-config
```

**2. Ambassador Pattern:**
Proxy container that brokers connections for main container.

**Example: Database Proxy**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-with-db-proxy
spec:
  containers:
  # Main application
  - name: app
    image: myapp:latest
    env:
    - name: DATABASE_URL
      value: "localhost:3306"  # Connect to local proxy
  
  # Database proxy/ambassador
  - name: db-proxy
    image: mysql-proxy:latest
    ports:
    - containerPort: 3306
    env:
    - name: UPSTREAM_DB
      value: "mysql-cluster.database.svc.cluster.local:3306"
```

**3. Adapter Pattern:**
Transforms output of main container to match expected format.

**Example: Metrics Adapter**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-with-metrics-adapter
spec:
  containers:
  # Main application (legacy format)
  - name: legacy-app
    image: legacy-app:1.0
    ports:
    - containerPort: 8080
  
  # Metrics adapter (converts to Prometheus format)
  - name: metrics-adapter
    image: metrics-adapter:latest
    ports:
    - containerPort: 9090
    env:
    - name: SOURCE_URL
      value: "http://localhost:8080/stats"
```

**Real-World Use Cases:**

**1. Web Server with SSL Termination:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-with-ssl
spec:
  containers:
  - name: web-app
    image: myapp:latest
    ports:
    - containerPort: 8080
  - name: ssl-proxy
    image: nginx:latest
    ports:
    - containerPort: 443
    volumeMounts:
    - name: ssl-certs
      mountPath: /etc/ssl/certs
  volumes:
  - name: ssl-certs
    secret:
      secretName: ssl-certificates
```

**2. Microservice with Service Mesh:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: service-with-istio
  annotations:
    sidecar.istio.io/inject: "true"
spec:
  containers:
  - name: microservice
    image: my-service:v1.0
    ports:
    - containerPort: 8080
  # Istio automatically injects envoy proxy sidecar
```

**Benefits:**
- **Separation of concerns**: Each container has single responsibility
- **Reusability**: Sidecar containers can be reused across applications
- **Technology diversity**: Different containers can use different languages/tools
- **Simplified networking**: Containers communicate via localhost
- **Shared resources**: Efficient resource sharing

**When to Use Multi-Container Pods:**
- Logging and monitoring sidecars
- Service mesh proxies (Istio, Linkerd)
- Configuration synchronization
- File or data transformation
- Protocol adaptation
- Security scanning

**When NOT to Use:**
- Loosely coupled services (use separate pods)
- Services that can scale independently
- Different resource requirements
- Different restart policies needed

---

## ⚙️ Section 2: Scenario - Answer

**Scenario:**  
Your company is deploying a new application that requires different access permissions for different teams. The development team needs read-only access to pods and services in the `development` namespace, while the operations team needs full access to all resources in both `development` and `production` namespaces.  
Design and explain the RBAC configuration needed to implement this security model.

**Answer:**

Here's a comprehensive RBAC configuration to implement the described security model:

**1. Create Namespaces:**
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: development
---
apiVersion: v1
kind: Namespace
metadata:
  name: production
```

**2. Development Team - Read-Only Access:**

**Create Role for Development Namespace:**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: development
  name: dev-team-reader
rules:
# Pods - read-only access
- apiGroups: [""]
  resources: ["pods", "pods/log", "pods/status"]
  verbs: ["get", "list", "watch"]
# Services - read-only access  
- apiGroups: [""]
  resources: ["services", "endpoints"]
  verbs: ["get", "list", "watch"]
# Additional useful resources for developers
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets"]
  verbs: ["get", "list", "watch"]
- apiGroups: [""]
  resources: ["configmaps", "secrets"]
  verbs: ["get", "list"]
```

**Create RoleBinding for Development Team:**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: dev-team-binding
  namespace: development
subjects:
# Add individual developers
- kind: User
  name: dev-user-1
  apiGroup: rbac.authorization.k8s.io
- kind: User
  name: dev-user-2
  apiGroup: rbac.authorization.k8s.io
# Or use a group (recommended)
- kind: Group
  name: developers
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: dev-team-reader
  apiGroup: rbac.authorization.k8s.io
```

**3. Operations Team - Full Access:**

**Create ClusterRole for Operations Team:**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: ops-team-full-access
rules:
# Full access to all core resources
- apiGroups: [""]
  resources: ["*"]
  verbs: ["*"]
# Full access to apps resources
- apiGroups: ["apps"]
  resources: ["*"]
  verbs: ["*"]
# Access to networking resources
- apiGroups: ["networking.k8s.io"]
  resources: ["*"]
  verbs: ["*"]
# Access to RBAC resources (for user management)
- apiGroups: ["rbac.authorization.k8s.io"]
  resources: ["*"]
  verbs: ["*"]
# Access to storage resources
- apiGroups: ["storage.k8s.io"]
  resources: ["*"]
  verbs: ["*"]
```

**Create RoleBindings for Specific Namespaces:**
```yaml
# Operations access to development namespace
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: ops-team-dev-binding
  namespace: development
subjects:
- kind: Group
  name: operations
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: ops-team-full-access
  apiGroup: rbac.authorization.k8s.io

---
# Operations access to production namespace  
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: ops-team-prod-binding
  namespace: production
subjects:
- kind: Group
  name: operations
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: ops-team-full-access
  apiGroup: rbac.authorization.k8s.io
```

**4. Optional: Create Groups and Service Accounts:**

**Create Groups (configured in identity provider):**
```yaml
# This would be configured in your identity provider (AD, LDAP, etc.)
# Groups: developers, operations
```

**Alternative: Service Accounts for CI/CD:**
```yaml
# Service account for development CI/CD
apiVersion: v1
kind: ServiceAccount
metadata:
  name: dev-ci-sa
  namespace: development

---
# Bind CI service account to dev role
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: dev-ci-binding
  namespace: development
subjects:
- kind: ServiceAccount
  name: dev-ci-sa
  namespace: development
roleRef:
  kind: Role
  name: dev-team-reader
  apiGroup: rbac.authorization.k8s.io
```

**5. Testing the Configuration:**

**Test Development Team Access:**
```bash
# Test as development user
kubectl auth can-i get pods --namespace=development --as=user:dev-user-1
# Should return: yes

kubectl auth can-i delete pods --namespace=development --as=user:dev-user-1  
# Should return: no

kubectl auth can-i get pods --namespace=production --as=user:dev-user-1
# Should return: no
```

**Test Operations Team Access:**
```bash
# Test as operations user  
kubectl auth can-i "*" "*" --namespace=development --as=user:ops-user-1
# Should return: yes

kubectl auth can-i "*" "*" --namespace=production --as=user:ops-user-1
# Should return: yes
```

**6. Security Best Practices:**

**Principle of Least Privilege:**
```yaml
# Give developers only what they need for troubleshooting
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: development
  name: dev-team-troubleshoot
rules:
- apiGroups: [""]
  resources: ["pods/exec", "pods/portforward"]
  verbs: ["create"]
- apiGroups: [""]
  resources: ["pods/log"]
  verbs: ["get", "list"]
```

**Regular Access Review:**
```bash
# Audit current permissions
kubectl auth can-i --list --as=user:dev-user-1
kubectl auth can-i --list --as=user:ops-user-1

# Review role bindings
kubectl get rolebindings --all-namespaces
kubectl get clusterrolebindings
```

This RBAC configuration provides:
- **Secure separation**: Teams can only access appropriate resources
- **Scalable**: Easy to add new users to existing groups
- **Auditable**: Clear permission structure for compliance
- **Flexible**: Can be extended for additional teams or requirements

---

## 🧩 Section 3: Problem-Solving - Answer

**Task:**  
A team deployed a StatefulSet for a database, but they're experiencing issues:

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: database
spec:
  replicas: 3
  serviceName: database-service
  selector:
    matchLabels:
      app: database
  template:
    metadata:
      labels:
        app: database
    spec:
      containers:
      - name: db
        image: postgres:13
        ports:
        - containerPort: 5432
        env:
        - name: POSTGRES_DB
          value: myapp
        - name: POSTGRES_USER
          value: admin
        - name: POSTGRES_PASSWORD
          value: secretpassword
```

The database pods keep restarting and losing data between restarts. Users report that database connections are unreliable. Identify the problems and provide a corrected configuration that addresses data persistence, security, and service discovery issues.

**Answer:**

**Problems Identified:**

1. **No data persistence** - Missing PersistentVolumeClaims
2. **Security issues** - Hardcoded passwords in plain text
3. **Missing service discovery** - No headless service for StatefulSet
4. **No health checks** - Missing readiness/liveness probes
5. **No resource limits** - Could cause OOM kills
6. **Missing PostgreSQL initialization** - No proper DB initialization

**Corrected Configuration:**

**1. Create Secrets for Sensitive Data:**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: database-credentials
type: Opaque
data:
  # Base64 encoded values
  # postgres-password: <base64-encoded-password>
  # postgres-user: <base64-encoded-username>  
  postgres-password: cG9zdGdyZXMtcGFzc3dvcmQtaGVyZQ==
  postgres-user: cG9zdGdyZXMtdXNlcg==
  postgres-db: bXlhcHAtZGI=
```

**2. Create Headless Service for StatefulSet:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: database-service
  labels:
    app: database
spec:
  clusterIP: None  # Headless service
  selector:
    app: database
  ports:
  - port: 5432
    targetPort: 5432
    protocol: TCP
    name: postgresql
```

**3. Create Regular Service for External Access:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: database-client-service
  labels:
    app: database
spec:
  type: ClusterIP
  selector:
    app: database
  ports:
  - port: 5432
    targetPort: 5432
    protocol: TCP
    name: postgresql
```

**4. Corrected StatefulSet with Persistence:**
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: database
  labels:
    app: database
spec:
  serviceName: database-service  # References headless service
  replicas: 3
  selector:
    matchLabels:
      app: database
  template:
    metadata:
      labels:
        app: database
    spec:
      containers:
      - name: db
        image: postgres:13
        ports:
        - containerPort: 5432
          name: postgresql
        
        # Environment variables from secrets
        env:
        - name: POSTGRES_DB
          valueFrom:
            secretKeyRef:
              name: database-credentials
              key: postgres-db
        - name: POSTGRES_USER
          valueFrom:
            secretKeyRef:
              name: database-credentials
              key: postgres-user
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: database-credentials
              key: postgres-password
        - name: PGDATA
          value: /var/lib/postgresql/data/pgdata
        
        # Resource limits and requests
        resources:
          limits:
            cpu: "1"
            memory: "1Gi"
          requests:
            cpu: "500m"
            memory: "512Mi"
        
        # Persistent volume mount
        volumeMounts:
        - name: database-storage
          mountPath: /var/lib/postgresql/data
          subPath: postgres
        
        # Health checks
        livenessProbe:
          exec:
            command:
            - /bin/sh
            - -c
            - exec pg_isready -U "$POSTGRES_USER" -d "$POSTGRES_DB" -h 127.0.0.1 -p 5432
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          successThreshold: 1
          failureThreshold: 6
        
        readinessProbe:
          exec:
            command:
            - /bin/sh
            - -c
            - exec pg_isready -U "$POSTGRES_USER" -d "$POSTGRES_DB" -h 127.0.0.1 -p 5432
          initialDelaySeconds: 5
          periodSeconds: 10
          timeoutSeconds: 5
          successThreshold: 1
          failureThreshold: 3
        
        # Security context
        securityContext:
          runAsUser: 999
          runAsGroup: 999
          fsGroup: 999
          runAsNonRoot: true
  
  # Persistent Volume Claim Templates
  volumeClaimTemplates:
  - metadata:
      name: database-storage
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: "fast-ssd"  # Use appropriate storage class
      resources:
        requests:
          storage: 20Gi
```

**5. Optional: PostgreSQL Configuration ConfigMap:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: postgres-config
data:
  postgresql.conf: |
    # PostgreSQL configuration
    max_connections = 200
    shared_buffers = 256MB
    effective_cache_size = 1GB
    maintenance_work_mem = 64MB
    checkpoint_completion_target = 0.7
    wal_buffers = 16MB
    default_statistics_target = 100
    random_page_cost = 1.1
    effective_io_concurrency = 200
    work_mem = 4MB
    min_wal_size = 1GB
    max_wal_size = 4GB
```

**6. Enhanced StatefulSet with ConfigMap:**
```yaml
# Add to the StatefulSet spec.template.spec
        volumeMounts:
        - name: database-storage
          mountPath: /var/lib/postgresql/data
          subPath: postgres
        - name: postgres-config
          mountPath: /etc/postgresql/postgresql.conf
          subPath: postgresql.conf
      
      volumes:
      - name: postgres-config
        configMap:
          name: postgres-config
```

**7. Network Policy for Security (Optional):**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: database-network-policy
spec:
  podSelector:
    matchLabels:
      app: database
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          access: database  # Only pods with this label can connect
    ports:
    - protocol: TCP
      port: 5432
  egress:
  - {} # Allow all egress for now (restrict as needed)
```

**Key Fixes Implemented:**

1. **Data Persistence**: 
   - Added `volumeClaimTemplates` for persistent storage
   - Each pod gets its own PVC (database-0, database-1, database-2)

2. **Security Improvements**:
   - Moved secrets to Kubernetes Secret resource
   - Added security context with non-root user
   - Optional network policy for access control

3. **Service Discovery**:
   - Added headless service for StatefulSet
   - Added regular service for client connections
   - Predictable pod DNS names: `database-0.database-service.default.svc.cluster.local`

4. **Health Monitoring**:
   - Added liveness probes to restart unhealthy containers
   - Added readiness probes to control traffic routing

5. **Resource Management**:
   - Added CPU and memory limits/requests
   - Prevents resource starvation and OOM kills

6. **Operational Improvements**:
   - Proper PGDATA configuration
   - ConfigMap for PostgreSQL configuration
   - Better labeling and organization

**Verification Commands:**
```bash
# Check StatefulSet status
kubectl get statefulset database

# Check persistent volumes
kubectl get pvc

# Check pod connectivity
kubectl exec -it database-0 -- psql -U admin -d myapp -c "SELECT 1;"

# Test service discovery
kubectl exec -it database-0 -- nslookup database-service
```

This corrected configuration addresses all the identified issues and provides a production-ready PostgreSQL StatefulSet deployment.
