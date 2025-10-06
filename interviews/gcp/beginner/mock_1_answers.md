# Google Cloud Platform (GCP) - Beginner Interview Mock #1 - Answer Key

> **Difficulty:** Beginner  
> **Duration:** ~30 minutes  
> **Goal:** Assess your understanding of basic Google Cloud Platform concepts, core services, and fundamental GCP architecture principles.

---

## 🧠 Section 1: Core Questions - Answers

### 1. What is Google Cloud Platform (GCP), and what are its main advantages over traditional on-premises infrastructure?

**Answer:** Google Cloud Platform (GCP) is Google's suite of cloud computing services that runs on the same infrastructure that Google uses internally for its end-user products. Main advantages:
- **Global Infrastructure**: Leverages Google's massive global network and data centers
- **Cost Efficiency**: Pay-as-you-go model, no upfront hardware costs
- **Scalability**: Automatic scaling based on demand
- **Reliability**: Built-in redundancy and Google's SLA guarantees
- **Security**: Google's security model with encryption at rest and in transit
- **Innovation**: Access to Google's cutting-edge AI/ML services and technologies
- **Managed Services**: Reduced operational overhead with fully managed services
- **Performance**: Google's high-performance network and infrastructure

### 2. Explain the GCP hierarchy: Organization, Folders, Projects, and Resources. How does this structure help with resource management?

**Answer:** GCP uses a hierarchical structure for resource organization:

- **Organization**: Root node representing your company/domain (e.g., company.com)
- **Folders**: Optional organizational units to group projects (e.g., by department, environment)
- **Projects**: Fundamental organizing entity where resources are created and billed
- **Resources**: Individual GCP services (VMs, databases, storage buckets, etc.)

**Benefits:**
- **Policy Inheritance**: IAM policies applied at higher levels inherit down
- **Billing Management**: Organize billing by business units or projects
- **Resource Organization**: Logical grouping for different environments (dev/staging/prod)
- **Access Control**: Granular permissions at different organizational levels
- **Resource Quotas**: Apply limits and quotas at various levels

### 3. What are the core compute services in GCP? Describe Compute Engine, App Engine, and Cloud Functions.

**Answer:**

**Compute Engine (IaaS):**
- Virtual machines running in Google's data centers
- Full control over OS, applications, and configuration
- Custom machine types and preemptible instances
- Use case: Traditional applications, custom environments, lift-and-shift migrations

**App Engine (PaaS):**
- Fully managed serverless platform for applications
- Auto-scaling, load balancing, and health monitoring
- Supports multiple languages (Python, Java, Node.js, Go, etc.)
- Use case: Web applications, APIs, microservices without infrastructure management

**Cloud Functions (FaaS):**
- Event-driven serverless compute platform
- Execute code in response to events (HTTP requests, Cloud Storage changes, Pub/Sub messages)
- Pay only for actual execution time
- Use case: Event processing, webhooks, data processing, IoT backends

### 4. What storage options does GCP provide? Explain Cloud Storage, Cloud SQL, and Firestore.

**Answer:**

**Cloud Storage (Object Storage):**
- Scalable object storage for unstructured data
- Multiple storage classes: Standard, Nearline, Coldline, Archive
- Global accessibility with strong consistency
- Use case: Website assets, backup, data archiving, data lakes

**Cloud SQL (Managed Relational Database):**
- Fully managed MySQL, PostgreSQL, and SQL Server
- Automatic backups, patch management, and high availability
- Read replicas and point-in-time recovery
- Use case: Traditional applications, transactional data, structured data

**Firestore (NoSQL Document Database):**
- Serverless, real-time NoSQL document database
- Auto-scaling with strong consistency
- Offline support and real-time synchronization
- Use case: Mobile apps, web apps, real-time applications, flexible data models

### 5. What is a VPC (Virtual Private Cloud) in GCP, and how does it differ from the default network?

**Answer:**

**VPC (Virtual Private Cloud):**
- Virtualized network within GCP that provides isolated environment
- Global resource spanning all GCP regions
- Custom IP address ranges, subnets, and routing
- Firewall rules and network security controls

**Default Network vs Custom VPC:**

**Default Network:**
- Pre-created in every new project
- One subnet per region with predefined IP ranges (10.128.0.0/20, etc.)
- Default firewall rules allowing internal communication and some external access
- Good for getting started quickly

**Custom VPC:**
- User-defined IP ranges and subnet structure
- Custom firewall rules and routing
- Better security and network isolation
- Required for production environments and complex architectures
- More control over network topology and security policies

### 6. Explain GCP regions and zones. Why is this important for application deployment?

**Answer:**

**Regions:**
- Independent geographic areas (e.g., us-central1, europe-west1)
- Multiple zones within each region
- Isolated from failures in other regions

**Zones:**
- Isolated locations within a region (e.g., us-central1-a, us-central1-b)
- Single failure domain
- Low-latency connections between zones in same region

**Importance for deployment:**
- **High Availability**: Deploy across multiple zones to avoid single points of failure
- **Disaster Recovery**: Use multiple regions for geographic redundancy
- **Performance**: Choose regions close to users for lower latency
- **Compliance**: Some regulations require data to stay in specific geographic areas
- **Cost Optimization**: Pricing may vary between regions

### 7. What is IAM (Identity and Access Management) in GCP? What are the three main components?

**Answer:** IAM in GCP controls who (identity) has what access (role) to which resources. The three main components:

**1. Members (Who):**
- Google Accounts (individual users)
- Service Accounts (applications/VMs)
- Google Groups (collection of users)
- Google Workspace domains
- Cloud Identity domains

**2. Roles (What permissions):**
- **Primitive Roles**: Basic roles (Owner, Editor, Viewer) - broad permissions
- **Predefined Roles**: Curated roles for specific GCP services (e.g., Compute Admin)
- **Custom Roles**: User-defined roles with specific permissions

**3. Resources (Where):**
- GCP resources where permissions apply
- Can be applied at organization, folder, project, or individual resource level

**Policy Binding:** Connects members to roles for specific resources

### 8. What is Google Cloud Console, and what alternative methods exist to interact with GCP services?

**Answer:**

**Google Cloud Console:**
- Web-based graphical user interface for managing GCP resources
- Accessible from any browser at console.cloud.google.com
- Provides dashboards, monitoring, resource management, and billing information
- Best for: Initial exploration, monitoring, and occasional administrative tasks

**Alternative Methods:**

**Cloud SDK/gcloud CLI:**
- Command-line interface for scripting and automation
- Local installation or Cloud Shell
- Best for: Automation, CI/CD pipelines, power users

**Cloud Shell:**
- Browser-based shell environment with pre-installed tools
- Includes gcloud CLI, kubectl, terraform, etc.
- 5GB persistent disk storage
- Best for: Quick tasks without local setup

**Client Libraries:**
- SDKs for various programming languages (Python, Java, Node.js, etc.)
- Best for: Application development and integration

**REST APIs:**
- Direct HTTP API calls to GCP services
- Best for: Custom integrations and applications

---

## ⚙️ Section 2: Scenario - Answer

**Migration Approach:**

**1. Assessment and Planning:**
- Assess current application architecture and dependencies
- Identify data migration requirements
- Plan for minimal downtime during migration

**2. GCP Services Selection:**
- **Web Application**: 
  - **Option A**: Compute Engine VM (lift-and-shift approach)
  - **Option B**: App Engine (managed platform, minimal code changes)
- **Database**: Cloud SQL for MySQL (managed, compatible replacement)
- **Networking**: Custom VPC for security isolation
- **Load Balancing**: Cloud Load Balancer for high availability

**3. Migration Steps:**
- Set up Cloud SQL instance and migrate database
- Deploy application to Compute Engine or App Engine
- Configure VPC and firewall rules
- Set up Cloud DNS for domain management
- Implement SSL certificates using Google-managed certificates
- Configure monitoring and logging with Cloud Operations

**4. Security Considerations:**
- Use IAM for access control
- Configure firewall rules to restrict access
- Enable Cloud SQL private IP
- Implement SSL/TLS encryption
- Set up Cloud Armor for DDoS protection

---

## 🧩 Section 3: Problem-Solving - Answer

**GCP E-commerce Architecture:**

**Frontend (Static Website):**
- **Service**: Cloud Storage + Cloud CDN
- **Implementation**: Host static files in Cloud Storage bucket with CDN for global delivery
- **Benefits**: Cost-effective, global caching, automatic scaling

**Backend API:**
- **Service**: App Engine or Cloud Run
- **Implementation**: Deploy RESTful API as containerized service
- **Benefits**: Auto-scaling, managed infrastructure, regional deployment

**Database:**
- **Service**: Cloud SQL (PostgreSQL) for transactional data
- **Implementation**: Multi-zone deployment with read replicas
- **Benefits**: ACID compliance, managed backups, high availability

**File Storage (Product Images):**
- **Service**: Cloud Storage with Cloud CDN
- **Implementation**: Different buckets for different content types
- **Benefits**: Global distribution, automatic image optimization

**Global Access:**
- **Service**: Cloud Load Balancer with Cloud CDN
- **Implementation**: Global load balancer routing traffic to nearest region
- **Benefits**: Low latency, automatic failover

**Additional Components:**
- **Security**: Cloud Armor for DDoS protection, WAF rules
- **Monitoring**: Cloud Operations Suite for logging and monitoring
- **Networking**: Custom VPC with private subnets
- **Authentication**: Firebase Auth or Cloud Identity
- **Search**: Cloud Search API for product search functionality

**Architecture Diagram Concept:**
```
Internet → Cloud CDN → Load Balancer → App Engine (API)
                    ↓                      ↓
              Cloud Storage          Cloud SQL
              (Static files)        (Database)
```

**High Availability:**
- Multi-zone Cloud SQL deployment
- App Engine automatic scaling
- Load balancer health checks
- Cloud Storage 99.999% availability

**Security Measures:**
- VPC firewall rules
- Cloud IAM for service access
- Cloud SQL private IP
- SSL certificates
- Cloud Armor protection
- Regular security scanning

---

[🏠 Back to Main](../../../README.md) | [📋 All GCP Interviews](../../README.md)
