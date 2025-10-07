# Google Cloud Platform (GCP) - Intermediate Interview Mock #1 - Answer Key

> **Difficulty:** Intermediate  
> **Duration:** ~45 minutes  
> **Goal:** Assess your understanding of GCP architecture patterns, multi-region deployments, advanced networking, and production-ready solutions.

---

## 🧠 Section 1: Core Questions - Answers

### 1. Explain the different types of Cloud Load Balancers in GCP. When would you use each type?

**Answer:**

**Global Load Balancers:**

**1. Global External HTTP(S) Load Balancer:**
- **Layer**: 7 (Application layer)
- **Protocol**: HTTP, HTTPS
- **Scope**: Global (multi-region)
- **Use Cases**: Web applications, APIs, content serving
- **Features**: URL-based routing, SSL termination, Cloud CDN integration
- **Backend Services**: Instance groups, NEGs, Cloud Storage buckets

**2. Global External TCP Proxy:**
- **Layer**: 4 (Transport layer) 
- **Protocol**: TCP
- **Scope**: Global
- **Use Cases**: Non-HTTP TCP traffic that needs global distribution
- **Features**: SSL termination for TCP, connection pooling
- **Limitations**: Limited to specific ports

**3. Global External SSL Proxy:**
- **Layer**: 4 with SSL termination
- **Protocol**: SSL/TLS
- **Scope**: Global
- **Use Cases**: SSL traffic that needs global load balancing
- **Features**: SSL offloading, connection multiplexing

**Regional Load Balancers:**

**4. Regional External HTTP(S) Load Balancer:**
- **Layer**: 7
- **Protocol**: HTTP, HTTPS
- **Scope**: Single region
- **Use Cases**: Regional applications, cost optimization
- **Features**: Lower latency, regional traffic distribution

**5. External TCP/UDP Network Load Balancer:**
- **Layer**: 4
- **Protocol**: TCP, UDP
- **Scope**: Regional
- **Use Cases**: High performance, low latency applications
- **Features**: Pass-through load balancing, preserves source IP

**6. Internal TCP/UDP Load Balancer:**
- **Layer**: 4
- **Protocol**: TCP, UDP
- **Scope**: Regional, internal traffic only
- **Use Cases**: Microservices, database load balancing
- **Features**: Private load balancing within VPC

**7. Internal HTTP(S) Load Balancer:**
- **Layer**: 7
- **Protocol**: HTTP, HTTPS
- **Scope**: Regional, internal traffic only
- **Use Cases**: Internal APIs, microservices with advanced routing
- **Features**: Content-based routing within VPC

### 2. What is Cloud Armor and how does it integrate with load balancing? Describe its security features.

**Answer:**

**Cloud Armor:**
- Web Application Firewall (WAF) and DDoS protection service
- Integrates with Global External HTTP(S) Load Balancers
- Edge security service that filters malicious traffic before it reaches backends

**Integration with Load Balancing:**
- **Attachment Point**: Security policies attached to backend services or backend buckets
- **Processing Order**: Cloud Armor → Load Balancer → Backend
- **Global Reach**: Leverages Google's global edge network (Point of Presence)
- **Performance**: Minimal latency impact with edge filtering

**Security Features:**

**1. DDoS Protection:**
- **Layer 3/4**: Network and transport layer protection (always on)
- **Layer 7**: Application layer DDoS protection
- **Adaptive Protection**: ML-based detection of application-layer attacks
- **Rate Limiting**: Request rate controls per client IP

**2. WAF Rules:**
- **Pre-configured Rules**: OWASP Top 10 protection
- **Custom Rules**: IP allowlists/denylists, geographic restrictions
- **Signature-based Detection**: SQL injection, XSS, RCE prevention
- **Bot Management**: Distinguish between good and malicious bots

**3. Security Policies:**
```yaml
# Example Cloud Armor rule
rules:
- priority: 1000
  match:
    versionedExpr: SRC_IPS_V1
    config:
      srcIpRanges: ["192.0.2.0/24"]
  action: "allow"
- priority: 2000
  match:
    expr:
      expression: 'origin.region_code == "CN"'
  action: "deny-403"
```

**4. Advanced Features:**
- **Logging**: Detailed security event logging
- **Monitoring**: Integration with Cloud Operations
- **Custom Error Pages**: Branded error responses
- **Preview Mode**: Test rules without blocking traffic

### 3. How do you implement Blue-Green deployments in GCP? Compare different approaches using various services.

**Answer:**

**Blue-Green Deployment Concept:**
- Two identical production environments (Blue = current, Green = new version)
- Traffic switches from Blue to Green after validation
- Enables zero-downtime deployments and quick rollbacks

**GCP Implementation Approaches:**

**1. Global Load Balancer + Instance Groups:**
```bash
# Create new instance group (Green)
gcloud compute instance-groups managed create app-green \
    --template=app-template-v2 --size=3 --zone=us-central1-a

# Update backend service to split traffic
gcloud compute backend-services update app-backend \
    --global --backend group=app-blue,balancing-mode=RATE,max-rate=70 \
    --backend group=app-green,balancing-mode=RATE,max-rate=30
```

**2. App Engine Traffic Splitting:**
```yaml
# app.yaml for new version
service: default
version: green
runtime: python39

# Deploy and split traffic
gcloud app deploy --version=green --no-promote
gcloud app services set-traffic default --splits=blue=70,green=30
gcloud app services set-traffic default --splits=green=100  # Full cutover
```

**3. GKE with Ingress Controller:**
```yaml
# Blue-Green with Kubernetes Services
apiVersion: v1
kind: Service
metadata:
  name: app-blue
spec:
  selector:
    app: myapp
    version: blue
---
apiVersion: v1
kind: Service
metadata:
  name: app-green
spec:
  selector:
    app: myapp
    version: green
```

**4. Cloud Run with Traffic Allocation:**
```bash
# Deploy new revision without traffic
gcloud run deploy myapp --image=gcr.io/project/app:v2 \
    --no-traffic --tag=green

# Split traffic gradually
gcloud run services update-traffic myapp \
    --to-revisions=blue=70,green=30
```

**Comparison of Approaches:**

| Service | Pros | Cons | Best For |
|---------|------|------|----------|
| **Compute Engine** | Full control, cost-effective | Manual management, complex setup | Custom applications |
| **App Engine** | Managed, built-in traffic splitting | Platform limitations | Web applications |
| **GKE** | Kubernetes-native, flexible | Complexity, requires K8s knowledge | Containerized apps |
| **Cloud Run** | Serverless, automatic scaling | Stateless only | Microservices |

### 4. Explain VPC peering vs VPN vs Interconnect. When would you choose each connectivity option?

**Answer:**

**VPC Peering:**
- **What**: Direct network connection between two VPCs
- **Scope**: Within Google Cloud (same or different projects/organizations)
- **Routing**: Private RFC 1918 IP addresses
- **Bandwidth**: High bandwidth, low latency
- **Cost**: No additional charges for traffic between peered VPCs

**Use Cases:**
- Multi-project architectures
- Shared services (databases, monitoring)
- Cross-organization collaboration within GCP

**Limitations:**
- No transitive peering
- No on-premises connectivity
- Subnet IP ranges cannot overlap

**Cloud VPN:**
- **What**: IPsec tunnel over internet to connect on-premises to GCP
- **Types**: Classic VPN (deprecated), HA VPN (99.99% SLA)
- **Bandwidth**: Up to 3 Gbps per tunnel
- **Cost**: Moderate ($0.05/hour + egress charges)

**Use Cases:**
- Hybrid cloud connectivity
- Backup connectivity for Interconnect
- Low-to-moderate bandwidth requirements
- Quick setup for testing/development

**Cloud Interconnect:**

**Dedicated Interconnect:**
- **What**: Physical connection between on-premises and Google
- **Bandwidth**: 10 Gbps or 100 Gbps
- **Latency**: Ultra-low latency
- **Cost**: High (port fees + egress charges)

**Partner Interconnect:**
- **What**: Connection through Google Cloud Partner
- **Bandwidth**: 50 Mbps to 50 Gbps
- **Setup**: Faster than Dedicated Interconnect
- **Cost**: Variable based on partner

**Decision Matrix:**

| Requirement | VPC Peering | Cloud VPN | Dedicated Interconnect | Partner Interconnect |
|-------------|-------------|-----------|----------------------|---------------------|
| **GCP-to-GCP** | ✅ Best | ❌ | ❌ | ❌ |
| **Hybrid Cloud** | ❌ | ✅ Good | ✅ Best | ✅ Good |
| **High Bandwidth** | ✅ | ❌ | ✅ | ✅ |
| **Low Latency** | ✅ | ❌ | ✅ | ✅ |
| **Quick Setup** | ✅ | ✅ | ❌ | ⚠️ |
| **Cost Effective** | ✅ | ✅ | ❌ | ⚠️ |

### 5. What are the different database replication options in Cloud SQL? How do you handle failover scenarios?

**Answer:**

**Cloud SQL Replication Options:**

**1. Read Replicas:**
- **Purpose**: Scale read operations, reduce latency
- **Replication**: Asynchronous
- **Location**: Same region, cross-region, or cross-zone
- **Use Cases**: Read-heavy workloads, reporting, analytics

**Configuration:**
```bash
gcloud sql instances create read-replica-1 \
    --master-instance-name=master-instance \
    --region=us-west1 \
    --replica-type=READ
```

**2. High Availability (Regional Persistent Disks):**
- **Purpose**: Automatic failover within region
- **Setup**: Primary instance with standby in different zone
- **Failover Time**: 60-120 seconds
- **Data Loss**: Minimal (synchronous replication)

**3. Cross-Region Failover Replica:**
- **Purpose**: Disaster recovery across regions
- **Replication**: Asynchronous (potential data loss)
- **Failover**: Manual promotion to primary
- **Use Cases**: Business continuity, compliance

**Failover Scenarios and Handling:**

**Scenario 1: Zone Failure (HA Configuration)**
```bash
# Enable HA during instance creation
gcloud sql instances create prod-db \
    --availability-type=REGIONAL \
    --backup-start-time=03:00
```
- **Automatic Failover**: 60-120 seconds
- **Application Impact**: Brief connection interruption
- **Data Loss**: None (synchronous replication)

**Scenario 2: Region Failure (Cross-Region Replica)**
```bash
# Create cross-region replica
gcloud sql instances create dr-replica \
    --master-instance-name=prod-db \
    --region=us-west1 \
    --replica-type=FAILOVER

# Promote replica during disaster
gcloud sql instances promote-replica dr-replica
```

**Best Practices for Failover:**
- **Connection Pooling**: Use connection pools with retry logic
- **Health Checks**: Monitor database connectivity and performance
- **Automated Scripts**: Prepare failover automation scripts
- **Regular Testing**: Practice failover procedures regularly
- **Monitoring**: Set up alerts for replication lag and failures

**Application-Level Considerations:**
```python
# Python example with retry logic
import time
from sqlalchemy import create_engine
from sqlalchemy.exc import OperationalError

def get_db_connection():
    primary_url = "postgresql://user:pass@primary-ip:5432/db"
    replica_url = "postgresql://user:pass@replica-ip:5432/db"
    
    try:
        engine = create_engine(primary_url)
        engine.connect()
        return engine
    except OperationalError:
        # Fallback to read replica
        return create_engine(replica_url)
```

### 6. How do you implement autoscaling for different GCP services? Discuss metrics and policies.

**Answer:**

**Compute Engine Autoscaling:**

**Managed Instance Groups (MIG):**
```bash
# Create autoscaler
gcloud compute autoscalers create web-autoscaler \
    --target-instance-group=web-group \
    --min-num-replicas=2 \
    --max-num-replicas=10 \
    --target-cpu-utilization=0.7 \
    --cool-down-period=90s
```

**Scaling Metrics:**
- **CPU Utilization**: Most common, good for CPU-bound workloads
- **HTTP Load Balancing Utilization**: Requests per second per instance
- **Custom Metrics**: Application-specific metrics via Cloud Monitoring
- **Multiple Metrics**: Combine different metrics for better decisions

**GKE Autoscaling:**

**Horizontal Pod Autoscaler (HPA):**
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: app-deployment
  minReplicas: 2
  maxReplicas: 50
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
```

**Cluster Autoscaler:**
```yaml
# Node pool with autoscaling
gcloud container node-pools create scalable-pool \
    --cluster=my-cluster \
    --enable-autoscaling \
    --min-nodes=1 \
    --max-nodes=10 \
    --machine-type=n1-standard-2
```

**App Engine Autoscaling:**
```yaml
# app.yaml
automatic_scaling:
  min_instances: 2
  max_instances: 20
  target_cpu_utilization: 0.6
  target_throughput_utilization: 0.7
  max_concurrent_requests: 50
  min_pending_latency: 30ms
  max_pending_latency: 100ms
```

**Cloud Run Autoscaling:**
```yaml
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  annotations:
    autoscaling.knative.dev/minScale: "1"
    autoscaling.knative.dev/maxScale: "100"
    autoscaling.knative.dev/target: "70"
spec:
  template:
    metadata:
      annotations:
        autoscaling.knative.dev/target: "70"
```

**Cloud Functions Autoscaling:**
- **Automatic**: Scales based on incoming requests
- **Max Instances**: Limit concurrent executions
- **Configuration**: Set max instances to control costs

**Best Practices:**
- **Gradual Scaling**: Avoid aggressive scaling policies
- **Predictive Scaling**: Use scheduled scaling for predictable load
- **Cost Optimization**: Set appropriate max limits
- **Monitoring**: Track scaling events and performance impact
- **Testing**: Load test autoscaling behavior before production

### 7. Explain Workload Identity in GKE. How does it improve security compared to traditional service account keys?

**Answer:**

**Workload Identity:**
- Secure way to access Google Cloud services from GKE pods
- Eliminates need to store service account keys in clusters
- Maps Kubernetes Service Accounts to Google Service Accounts

**Traditional Approach (Less Secure):**
```yaml
# Bad practice: Service account key as secret
apiVersion: v1
kind: Secret
metadata:
  name: gcp-key
type: Opaque
data:
  key.json: <base64-encoded-key-file>
```

**Workload Identity Setup:**
```bash
# Enable Workload Identity on cluster
gcloud container clusters update CLUSTER_NAME \
    --workload-pool=PROJECT_ID.svc.id.goog

# Create Kubernetes service account
kubectl create serviceaccount KSA_NAME

# Create Google service account
gcloud iam service-accounts create GSA_NAME

# Bind the accounts
gcloud iam service-accounts add-iam-policy-binding \
    GSA_NAME@PROJECT_ID.iam.gserviceaccount.com \
    --role roles/iam.workloadIdentityUser \
    --member "serviceAccount:PROJECT_ID.svc.id.goog[NAMESPACE/KSA_NAME]"

# Annotate the Kubernetes service account
kubectl annotate serviceaccount KSA_NAME \
    iam.gke.io/gcp-service-account=GSA_NAME@PROJECT_ID.iam.gserviceaccount.com
```

**Pod Configuration:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: workload-identity-test
spec:
  serviceAccountName: KSA_NAME  # Uses Workload Identity
  containers:
  - image: google/cloud-sdk:slim
    name: workload-identity-test
    command: ["sleep", "3600"]
```

**Security Improvements:**

**1. No Key Management:**
- **Traditional**: Keys stored in secrets, potential exposure
- **Workload Identity**: No keys stored in cluster
- **Benefit**: Eliminates key rotation and secret management

**2. Automatic Credential Rotation:**
- **Traditional**: Manual key rotation required
- **Workload Identity**: Google handles token lifecycle
- **Benefit**: Always fresh, short-lived tokens

**3. Fine-grained Access Control:**
- **Traditional**: Service account key has all permissions
- **Workload Identity**: Specific pod-to-GSA mapping
- **Benefit**: Principle of least privilege per workload

**4. Audit and Monitoring:**
- **Traditional**: Difficult to track key usage
- **Workload Identity**: Full audit trail in Cloud Logging
- **Benefit**: Better security observability

**5. Cross-project Access:**
```bash
# Grant cross-project access
gcloud projects add-iam-policy-binding TARGET_PROJECT_ID \
    --member "serviceAccount:GSA_NAME@SOURCE_PROJECT_ID.iam.gserviceaccount.com" \
    --role roles/storage.viewer
```

**Migration Strategy:**
1. Enable Workload Identity on existing cluster
2. Create GSA and KSA mappings
3. Update pod specifications gradually
4. Remove service account key secrets
5. Validate access and audit logs

### 8. What are the best practices for organizing resources using labels, and how do they help with cost management?

**Answer:**

**GCP Labels:**
- Key-value pairs attached to resources
- Used for organization, automation, and cost tracking
- Inherited by some resources (e.g., Compute Engine instances from templates)

**Labeling Best Practices:**

**1. Consistent Naming Convention:**
```bash
# Recommended label structure
environment: prod|staging|dev
team: frontend|backend|data|ops
application: web-app|api|database
cost-center: engineering|marketing|sales
owner: team-email@company.com
```

**2. Hierarchical Organization:**
```bash
# Example labels for a web application
gcloud compute instances create web-server-1 \
    --labels=environment=prod,team=frontend,app=ecommerce,owner=web-team,cost-center=engineering
```

**3. Automated Labeling:**
```yaml
# Deployment Manager template with labels
resources:
- name: web-instance
  type: compute.v1.instance
  properties:
    labels:
      environment: {{ env.environment }}
      team: {{ env.team }}
      created-by: deployment-manager
```

**Cost Management Benefits:**

**1. Cost Allocation:**
```bash
# BigQuery query for cost by team
SELECT
  labels.key AS label_key,
  labels.value AS label_value,
  SUM(cost) AS total_cost
FROM `project.dataset.gcp_billing_export_v1_XXXXX`
CROSS JOIN UNNEST(labels) AS labels
WHERE labels.key = 'team'
GROUP BY labels.key, labels.value
ORDER BY total_cost DESC
```

**2. Budget Controls:**
```yaml
# Budget with label filters
displayName: "Frontend Team Budget"
amount:
  specifiedAmount:
    units: "1000"
budgetFilter:
  projects: ["projects/my-project"]
  labels:
    - key: "team"
      value: "frontend"
```

**3. Resource Lifecycle Management:**
```bash
# Delete resources by environment
gcloud compute instances list --filter="labels.environment=staging" \
    --format="value(name,zone)" | \
while read name zone; do
    gcloud compute instances delete $name --zone=$zone --quiet
done
```

**4. Chargeback Reporting:**
- **Departmental Costs**: Allocate cloud costs to business units
- **Project Tracking**: Track costs per project or application
- **Resource Optimization**: Identify underutilized resources by team

**Advanced Labeling Strategies:**

**5. Automation Tags:**
```bash
# Labels for automation
backup: daily|weekly|none
monitoring: critical|standard|minimal
auto-scaling: enabled|disabled
```

**6. Compliance and Governance:**
```bash
# Compliance labels
data-classification: public|internal|confidential
retention-policy: 7-years|3-years|1-year
compliance-scope: pci|sox|gdpr
```

**7. Policy Enforcement:**
```yaml
# Organization Policy requiring labels
constraints:
- constraint: "constraints/compute.requireOsLogin"
  listPolicy:
    enforcedValue: true
- constraint: "constraints/gcp.resourceLocations"
  listPolicy:
    allowedValues: ["us-central1", "us-east1"]
```

**Monitoring and Alerting on Labels:**
```yaml
# Alert policy based on labels
conditions:
- displayName: "High Cost Resources"
  conditionThreshold:
    filter: 'resource.type="gce_instance" AND resource.label.environment="prod"'
    comparison: COMPARISON_GT
    thresholdValue: 100
```

---

## ⚙️ Section 2: Scenario - Answer

**Multi-Region E-commerce Architecture Design:**

**1. Geographic Traffic Routing:**

**Global Load Balancer Setup:**
```bash
# Create global HTTP(S) load balancer
gcloud compute url-maps create global-lb \
    --default-service=us-backend-service

# Add path matcher for regions
gcloud compute url-maps add-path-matcher global-lb \
    --path-matcher-name=region-matcher \
    --default-service=us-backend-service \
    --backend-service-path-rule="/*=us-backend-service"
```

**Traffic Distribution Strategy:**
- **Primary Regions**: 
  - `us-central1` for North America
  - `europe-west1` for Europe (GDPR compliance)
- **Load Balancer**: Global External HTTP(S) LB with geo-routing
- **CDN**: Cloud CDN for static assets (images, CSS, JS)

**2. Database Replication Strategy:**

**PostgreSQL Cross-Region Setup:**
```bash
# Primary database in us-central1
gcloud sql instances create ecommerce-db-us \
    --database-version=POSTGRES_13 \
    --region=us-central1 \
    --availability-type=REGIONAL \
    --backup-start-time=03:00

# Read replica in europe-west1
gcloud sql instances create ecommerce-db-eu-replica \
    --master-instance-name=ecommerce-db-us \
    --region=europe-west1 \
    --replica-type=READ
```

**Data Strategy:**
- **User Data**: Region-specific databases for GDPR compliance
- **Product Catalog**: Global replication with eventual consistency
- **Orders/Transactions**: Process in user's region, sync globally
- **Analytics**: Aggregate in BigQuery with regional exports

**3. Failover Mechanisms:**

**Application Layer:**
```yaml
# GKE deployment with anti-affinity
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ecommerce-api
spec:
  replicas: 6
  template:
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values: ["ecommerce-api"]
            topologyKey: kubernetes.io/hostname
```

**Database Failover:**
- **Regional HA**: Automatic failover within region (60-120s)
- **Cross-Region**: Manual promotion of read replica during disasters
- **Application Logic**: Circuit breakers and retry mechanisms

**4. Data Residency Compliance (GDPR):**

**Architecture Pattern:**
```
EU Users → europe-west1 LB → EU GKE Cluster → EU Database
US Users → us-central1 LB → US GKE Cluster → US Database
```

**Data Isolation:**
- **Customer Data**: Stored only in customer's region
- **Shared Data**: Product catalog, pricing (non-personal data)
- **Data Transfer**: Encrypted, logged, with consent tracking

**5. Cost Optimization:**

**Strategies:**
- **Preemptible Instances**: Use for batch processing workloads
- **Committed Use Discounts**: 1-year commit for stable workloads
- **Regional Resources**: Use regional LB where global isn't needed
- **Auto-scaling**: Scale down during low-traffic periods
- **Storage Classes**: Archive old data to Nearline/Coldline storage

**Resource Sharing:**
- **Shared Services**: Monitoring, logging in primary region
- **Cross-Region**: Only for critical services requiring failover
- **Development**: Single region for dev/staging environments

---

## 🧩 Section 3: Problem-Solving - Answer

**Multi-Region Financial Application with Automated Failover:**

**1. Service Deployment Strategy:**

**Primary Region (us-east1) Architecture:**
```yaml
# Production deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: financial-app-primary
  namespace: production
spec:
  replicas: 6
  template:
    spec:
      containers:
      - name: app
        image: gcr.io/project/financial-app:v1.2.3
        resources:
          requests:
            cpu: 500m
            memory: 1Gi
          limits:
            cpu: 1000m
            memory: 2Gi
        readinessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 60
          periodSeconds: 30
```

**Secondary Region (us-west1) - Standby:**
- **Cold Standby**: Minimal instances running (2 replicas)
- **Quick Scale**: Pre-configured auto-scaling for rapid expansion
- **Image Sync**: Automatic image replication across regions

**2. Database Replication and Failover:**

**Cloud SQL PostgreSQL with Synchronous Replication:**
```bash
# Primary instance with HA
gcloud sql instances create financial-db-primary \
    --database-version=POSTGRES_13 \
    --region=us-east1 \
    --availability-type=REGIONAL \
    --storage-type=SSD \
    --storage-size=1TB \
    --backup-start-time=02:00 \
    --maintenance-window-day=SUN \
    --maintenance-window-hour=03

# Cross-region failover replica
gcloud sql instances create financial-db-failover \
    --master-instance-name=financial-db-primary \
    --region=us-west1 \
    --replica-type=FAILOVER \
    --availability-type=REGIONAL
```

**Alternative: Cloud Spanner (for stronger consistency):**
```bash
# Multi-region Spanner instance
gcloud spanner instances create financial-spanner \
    --config=nam-eur-asia1 \
    --description="Financial transactions DB" \
    --nodes=3
```

**3. Load Balancing and Traffic Routing:**

**Global Load Balancer Configuration:**
```bash
# Health check
gcloud compute health-checks create http financial-health-check \
    --port=8080 \
    --request-path=/health \
    --check-interval=10s \
    --timeout=5s \
    --healthy-threshold=2 \
    --unhealthy-threshold=3

# Backend services
gcloud compute backend-services create financial-backend-us-east \
    --protocol=HTTP \
    --health-checks=financial-health-check \
    --global

gcloud compute backend-services create financial-backend-us-west \
    --protocol=HTTP \
    --health-checks=financial-health-check \
    --global

# URL map with failover
gcloud compute url-maps create financial-lb \
    --default-service=financial-backend-us-east
```

**4. Monitoring and Alerting Setup:**

**Comprehensive Monitoring Stack:**
```yaml
# SLI/SLO Configuration
apiVersion: v1
kind: ConfigMap
metadata:
  name: slo-config
data:
  availability_slo: "99.9"  # 43 minutes/month
  latency_slo_p95: "500ms"
  error_rate_slo: "0.1%"
```

**Critical Alerts:**
```bash
# Availability alert
gcloud alpha monitoring policies create \
    --policy-from-file=availability-policy.yaml

# Database failover alert
gcloud alpha monitoring policies create \
    --policy-from-file=db-failover-policy.yaml
```

**5. Failover Triggers and Automation:**

**Automated Failover Logic:**
```python
# Cloud Function for automated failover
import googleapiclient.discovery
from google.cloud import monitoring_v3

def check_primary_health():
    """Check if primary region is healthy"""
    # Monitor multiple signals:
    # - Load balancer health checks
    # - Database connectivity 
    # - Application error rates
    # - Network connectivity
    
    health_score = calculate_health_score()
    if health_score < FAILOVER_THRESHOLD:
        trigger_failover()

def trigger_failover():
    """Execute automated failover"""
    # 1. Promote database replica
    promote_database_replica()
    
    # 2. Scale up secondary region
    scale_secondary_region()
    
    # 3. Update load balancer
    update_load_balancer_backend()
    
    # 4. Notify operations team
    send_failover_notification()
```

**Failover Decision Matrix:**
```yaml
triggers:
  - name: "Database Unavailable"
    threshold: "Connection failures > 5 in 2 minutes"
    action: "Immediate failover"
  
  - name: "High Error Rate"
    threshold: "Error rate > 5% for 3 minutes"
    action: "Investigate then failover"
  
  - name: "Region Network Issues"
    threshold: "Network latency > 2s for 5 minutes"
    action: "Gradual traffic shift"
```

**6. Testing Strategy for Disaster Recovery:**

**Chaos Engineering Approach:**
```bash
# Scheduled disaster recovery tests
# 1. Database failover test (monthly)
gcloud sql instances promote-replica financial-db-failover --async
# Verify application connectivity and data consistency

# 2. Region failure simulation (quarterly)
# Block traffic to primary region, verify secondary takes over

# 3. Full disaster recovery drill (bi-annually)
# Complete failover and failback procedures
```

**Automated Testing:**
```yaml
# Kubernetes CronJob for DR testing
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: dr-test
spec:
  schedule: "0 2 1 * *"  # Monthly at 2 AM
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: dr-test
            image: gcr.io/project/dr-test:latest
            command:
            - "/bin/sh"
            - "-c"
            - "run-dr-tests.sh"
          restartPolicy: OnFailure
```

**Rollback Procedures:**
1. **Health Verification**: Confirm primary region is healthy
2. **Database Sync**: Ensure data consistency between regions
3. **Gradual Traffic Shift**: Move traffic back in 25% increments
4. **Monitoring**: Watch for issues during rollback
5. **Cleanup**: Reset standby configuration

**SLA Compliance Metrics:**
- **RTO Achievement**: < 5 minutes (target: 3 minutes average)
- **RPO Achievement**: < 1 minute data loss (using synchronous replication)
- **Availability**: 99.9% uptime (measured across all regions)
- **Performance**: Maintain < 500ms P95 latency during failover
