# Kubernetes - Beginner Interview Mock #1 - Answer Key

> **Difficulty:** Beginner  
> **Duration:** ~30 minutes  
> **Goal:** Assess your understanding of basic Kubernetes architecture, concepts, and common commands.

---

## 🧠 Section 1: Core Questions - Answers

### 1. What is Kubernetes, and what problems does it solve compared to running containers manually?

**Answer:** Kubernetes is an open-source container orchestration platform that automates the deployment, scaling, and management of containerized applications. Problems it solves:
- **High Availability**: Automatically restarts failed containers and reschedules them on healthy nodes
- **Scaling**: Automatically scales applications based on demand (CPU, memory, custom metrics)
- **Load Distribution**: Distributes traffic across multiple container instances
- **Service Discovery**: Provides built-in DNS and service discovery mechanisms
- **Rolling Updates**: Enables zero-downtime deployments with rollback capabilities
- **Resource Management**: Efficiently allocates compute resources across the cluster
- **Self-healing**: Automatically replaces failed containers and nodes

### 2. Explain the difference between a **Pod**, a **Deployment**, and a **Service**.

**Answer:**
- **Pod**: The smallest deployable unit in Kubernetes. Contains one or more tightly coupled containers that share networking and storage. Pods are ephemeral and typically not created directly.
- **Deployment**: A higher-level abstraction that manages ReplicaSets and provides declarative updates to Pods. It ensures a specified number of Pod replicas are running and handles rolling updates/rollbacks.
- **Service**: An abstraction that defines a logical set of Pods and provides stable networking (IP address and DNS name) to access them. It acts as a load balancer for Pod traffic.

### 3. What is the role of the **kubelet** and **kube-apiserver**?

**Answer:**
- **kubelet**: The primary node agent that runs on each worker node. It communicates with the API server, manages Pod lifecycles, monitors container health, and reports node status back to the control plane.
- **kube-apiserver**: The central management component that exposes the Kubernetes API. It processes REST operations, validates requests, updates etcd, and serves as the communication hub for all cluster components.

### 4. What is a **Namespace**, and why would you use it?

**Answer:** A Namespace provides a way to divide cluster resources between multiple users or teams. Use cases:
- **Multi-tenancy**: Isolate resources for different teams/projects
- **Environment separation**: dev, staging, production environments
- **Resource quotas**: Apply limits and quotas per namespace
- **Access control**: Implement RBAC policies per namespace
- **Naming scope**: Prevent resource name conflicts

### 5. What's the difference between **ReplicaSet** and **ReplicationController**?

**Answer:**
- **ReplicationController**: Legacy resource that ensures a specified number of Pod replicas. Uses equality-based selectors only (e.g., `env=production`)
- **ReplicaSet**: Newer, more flexible resource that also ensures Pod replicas but supports set-based selectors (e.g., `env in (production, staging)`). It's the underlying resource managed by Deployments.

### 6. How do you scale a Deployment in Kubernetes?

**Answer:** Multiple ways to scale:
```bash
# Imperative scaling
kubectl scale deployment myapp --replicas=5

# Edit the deployment
kubectl edit deployment myapp

# Patch the deployment
kubectl patch deployment myapp -p '{"spec":{"replicas":5}}'

# Declarative - update YAML file and apply
kubectl apply -f deployment.yaml
```

### 7. What command would you use to view all Pods running in all namespaces?

**Answer:**
```bash
kubectl get pods --all-namespaces
# or
kubectl get pods -A
```

### 8. What happens when a Pod crashes — how does Kubernetes handle it?

**Answer:** Kubernetes implements self-healing through controllers:
- **ReplicaSet/Deployment**: Automatically creates a new Pod to maintain desired replica count
- **kubelet**: Restarts containers within a Pod based on the restart policy (`Always`, `OnFailure`, `Never`)
- **Node failure**: Scheduler reschedules Pods to healthy nodes
- **Health checks**: Liveness and readiness probes determine Pod health and traffic eligibility

---

## ⚙️ Section 2: Scenario - Answer

**Scenario:**  
You deployed an application to your cluster, but it's stuck in `CrashLoopBackOff`.  
Walk through how you'd troubleshoot this issue using `kubectl` commands.  
What steps would you take to identify the root cause and fix it?

**Answer:** Here's a systematic troubleshooting approach:

### 1. **Check Pod Status and Basic Info**
```bash
# Get basic pod information
kubectl get pods
kubectl get pod <pod-name> -o wide

# Get detailed pod description
kubectl describe pod <pod-name>
```

### 2. **Check Pod Events**
```bash
# Look at recent events (included in describe output)
kubectl get events --sort-by='.lastTimestamp' | grep <pod-name>
```

### 3. **Examine Container Logs**
```bash
# Check current logs
kubectl logs <pod-name> 

# Check previous container logs (before crash)
kubectl logs <pod-name> --previous

# For multi-container pods
kubectl logs <pod-name> -c <container-name>

# Follow logs in real-time
kubectl logs <pod-name> -f
```

### 4. **Check Resource Constraints**
```bash
# Check if resource limits are causing issues
kubectl describe node <node-name>
kubectl top pod <pod-name>
```

### 5. **Interactive Debugging**
```bash
# Execute commands in the container (if it stays running long enough)
kubectl exec -it <pod-name> -- /bin/bash

# Debug with a temporary container
kubectl debug <pod-name> -it --image=busybox
```

### **Common Root Causes and Solutions:**
- **Missing dependencies**: Check application configuration and required services
- **Resource limits**: Increase CPU/memory limits or requests
- **Environment variables**: Verify required env vars are set correctly
- **Volume mounts**: Check if persistent volumes are accessible
- **Image issues**: Verify image exists and is accessible
- **Port conflicts**: Ensure ports are correctly configured
- **Health check failures**: Review liveness/readiness probe configurations

---

## 🧩 Section 3: Problem-Solving - Answer

**Task:**  
A team member created this YAML and applied it:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 8080
```

The team member notices they can't access the nginx server from outside the cluster. What's wrong with this Pod definition, and how would you fix it? What additional Kubernetes resources would you need to create?

**Answer:**

### **Issues with the Pod Definition:**

1. **Wrong Port**: Nginx runs on port 80 by default, but the YAML specifies port 8080
2. **No Service**: A Pod alone cannot be accessed from outside the cluster - you need a Service
3. **Direct Pod Creation**: Better to use a Deployment for production workloads

### **Fixed Pod Definition:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80  # Nginx default port
```

### **Required Service to Enable External Access:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080  # For NodePort service
  type: NodePort  # or LoadBalancer for cloud environments
```

### **Better Approach - Using Deployment:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
  type: LoadBalancer  # or ClusterIP, NodePort depending on requirements
```

### **Access Methods:**
- **ClusterIP**: Internal cluster access only
- **NodePort**: Access via `<NodeIP>:<NodePort>`
- **LoadBalancer**: External load balancer (cloud providers)
- **Ingress**: HTTP/HTTPS routing with domain names (requires Ingress Controller)
