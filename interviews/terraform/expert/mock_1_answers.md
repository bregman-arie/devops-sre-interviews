# Terraform - Expert Interview Mock #1 - Answers

---

## 🧠 Section 1: Core Questions - Answers

### 1. Terraform Core Architecture Deep Dive

**Answer:**

Terraform Core's architecture consists of several key components:

**Graph Construction:**
- Terraform builds a directed acyclic graph (DAG) of resources during the planning phase
- Uses depth-first search (DFS) for dependency resolution
- Resource nodes are created from configuration, state, and provider schemas
- Edges represent dependencies (implicit via references, explicit via `depends_on`)

```hcl
# Example showing implicit dependency
resource "aws_instance" "web" {
  subnet_id = aws_subnet.main.id  # Creates implicit dependency
}

resource "aws_subnet" "main" {
  vpc_id = aws_vpc.main.id
}
```

**Dependency Resolution Algorithm:**
- Topological sort using Kahn's algorithm
- Handles transitive dependencies automatically
- Parallel execution based on dependency levels

**RPC Protocol & Provider Communication:**
- Uses gRPC with Protocol Buffers for provider communication
- Plugin protocol versions (4, 5, 6) with different capabilities
- Providers run as separate processes, communicate via stdin/stdout or network

**Circular Dependency Handling:**
```bash
# Terraform detects cycles during graph validation
Error: Cycle: aws_security_group.web, aws_security_group.db
```

**Performance Implications:**
- Graph complexity: O(V + E) where V = vertices, E = edges  
- Memory usage scales with resource count and dependency depth
- Large graphs (>1000 resources) may require parallelism tuning

### 2. Advanced State Management & Concurrency

**Answer:**

**Enterprise State Strategy:**

```hcl
# Backend configuration with encryption
terraform {
  backend "s3" {
    bucket               = "terraform-state-enterprise"
    key                 = "environments/${var.environment}/${var.team}/terraform.tfstate"
    region              = "us-west-2"
    encrypt             = true
    kms_key_id         = "arn:aws:kms:us-west-2:account:key/key-id"
    dynamodb_table     = "terraform-state-locks"
    workspace_key_prefix = "workspaces"
  }
}
```

**Custom State Backend Implementation:**
```go
// Custom backend with enhanced security
type CustomBackend struct {
    client       *storage.Client
    encryption   *encryption.Service
    auditLogger  *audit.Logger
}

func (b *CustomBackend) StateMgr(name string) (statemgr.Full, error) {
    return &CustomStateMgr{
        backend: b,
        name:    name,
        lockID:  generateLockID(),
    }, nil
}
```

**State Isolation Strategy:**
```
├── environments/
│   ├── prod/
│   │   ├── team-a/
│   │   │   ├── networking/
│   │   │   └── applications/
│   │   └── team-b/
│   ├── staging/
│   └── dev/
├── shared-services/
│   ├── security/
│   └── monitoring/
```

**Disaster Recovery Procedures:**
```bash
# State recovery from backups
terraform state pull > current_state.json
aws s3 cp s3://terraform-state-backup/$(date -d "yesterday" +%Y-%m-%d)/terraform.tfstate .
terraform state push terraform.tfstate

# State surgery for corrupted state
terraform state rm aws_instance.corrupted
terraform import aws_instance.recovered i-1234567890abcdef0
```

### 3. Custom Provider Development

**Answer:**

**Provider Framework v6 Implementation:**

```go
// provider.go
package provider

import (
    "context"
    "github.com/hashicorp/terraform-plugin-framework/provider"
    "github.com/hashicorp/terraform-plugin-framework/provider/schema"
)

type customProvider struct {
    version string
}

func (p *customProvider) Configure(ctx context.Context, req provider.ConfigureRequest, resp *provider.ConfigureResponse) {
    var config customProviderModel
    
    diags := req.Config.Get(ctx, &config)
    resp.Diagnostics.Append(diags...)
    
    client := &apiClient{
        baseURL: config.BaseURL.ValueString(),
        token:   config.Token.ValueString(),
    }
    
    resp.DataSourceData = client
    resp.ResourceData = client
}

// resource.go
type resourceExample struct {
    client *apiClient
}

func (r *resourceExample) Create(ctx context.Context, req resource.CreateRequest, resp *resource.CreateResponse) {
    var plan resourceExampleModel
    diags := req.Plan.Get(ctx, &plan)
    resp.Diagnostics.Append(diags...)
    
    // Plan modifiers for computed values
    if plan.ID.IsUnknown() {
        plan.ID = types.StringValue(generateID())
    }
    
    // API call with retry and error handling
    result, err := r.client.CreateResource(ctx, &CreateRequest{
        Name:        plan.Name.ValueString(),
        Description: plan.Description.ValueString(),
    })
    
    if err != nil {
        resp.Diagnostics.AddError("Creation failed", err.Error())
        return
    }
    
    // Update state with computed values
    plan.ID = types.StringValue(result.ID)
    plan.CreatedAt = types.StringValue(result.CreatedAt.String())
    
    diags = resp.State.Set(ctx, plan)
    resp.Diagnostics.Append(diags...)
}
```

**Schema with Validators:**
```go
func (r *resourceExample) Schema(ctx context.Context, req resource.SchemaRequest, resp *resource.SchemaResponse) {
    resp.Schema = schema.Schema{
        Attributes: map[string]schema.Attribute{
            "name": schema.StringAttribute{
                Required: true,
                Validators: []validator.String{
                    stringvalidator.LengthBetween(1, 64),
                    stringvalidator.RegexMatches(
                        regexp.MustCompile(`^[a-zA-Z][a-zA-Z0-9-]*$`),
                        "must start with letter and contain only alphanumeric characters and hyphens",
                    ),
                },
            },
        },
        Blocks: map[string]schema.Block{
            "nested_config": schema.ListNestedBlock{
                NestedObject: schema.NestedBlockObject{
                    Attributes: map[string]schema.Attribute{
                        "enabled": schema.BoolAttribute{Optional: true},
                    },
                },
            },
        },
    }
}
```

### 4. Enterprise Module Ecosystem & Governance

**Answer:**

**Module Registry Architecture:**
```
terraform-modules/
├── aws/
│   ├── vpc/
│   │   ├── versions.tf      # Provider constraints
│   │   ├── main.tf         # Core logic
│   │   ├── variables.tf    # Input validation
│   │   ├── outputs.tf      # Structured outputs
│   │   ├── compliance/     # Policy checks
│   │   │   ├── security.rego
│   │   │   └── cost.rego
│   │   └── tests/          # Automated testing
│   │       ├── basic_test.go
│   │       └── compliance_test.go
```

**Module Versioning & Governance:**
```hcl
# modules/aws/vpc/versions.tf
terraform {
  required_version = ">= 1.5"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

# Semantic versioning with breaking change detection
module "vpc" {
  source  = "company-registry/aws/vpc"
  version = "~> 2.1.0"  # Patch updates only
  
  # Compliance tags enforced by policy
  compliance_tags = {
    "CostCenter"     = var.cost_center
    "DataClassification" = "internal"
    "Compliance"     = "sox,hipaa"
  }
}
```

**Policy as Code Integration:**
```rego
# compliance/security.rego
package terraform.security

deny[msg] {
  resource := input.resource_changes[_]
  resource.type == "aws_s3_bucket"
  not resource.change.after.server_side_encryption_configuration
  
  msg := sprintf("S3 bucket %s must have encryption enabled", [resource.address])
}

# Cost governance policy
deny[msg] {
  resource := input.resource_changes[_]
  resource.type == "aws_instance"
  contains(["m5.24xlarge", "c5.24xlarge"], resource.change.after.instance_type)
  
  msg := sprintf("Instance type %s exceeds cost limits", [resource.change.after.instance_type])
}
```

### 5. Advanced Resource Lifecycle Management

**Answer:**

**Complete Lifecycle Configuration:**
```hcl
resource "aws_db_instance" "main" {
  # ... configuration ...
  
  lifecycle {
    create_before_destroy = true
    prevent_destroy      = true
    
    # Ignore changes to password after initial creation
    ignore_changes = [
      password,
      final_snapshot_identifier,
    ]
    
    # Custom replacement triggers
    replace_triggered_by = [
      aws_db_parameter_group.main.id,
      aws_db_subnet_group.main.id
    ]
  }
  
  # Precondition validation
  lifecycle {
    precondition {
      condition     = var.environment != "prod" || var.backup_retention_period >= 30
      error_message = "Production databases must have backup retention >= 30 days"
    }
  }
}
```

**Zero-Downtime Migration Strategy:**
```hcl
# Phase 1: Create new resource alongside old
resource "aws_rds_cluster" "new" {
  count = var.migration_phase == "create" ? 1 : 0
  
  # New configuration
  engine         = "aurora-mysql"
  engine_version = "8.0.mysql_aurora.3.02.0"
  
  lifecycle {
    create_before_destroy = true
  }
}

# Phase 2: Data migration and DNS switch
resource "aws_route53_record" "database" {
  zone_id = aws_route53_zone.main.zone_id
  name    = "db.internal"
  type    = "CNAME"
  ttl     = 60
  
  records = [
    var.migration_phase == "complete" ? 
      aws_rds_cluster.new[0].endpoint : 
      aws_rds_cluster.old.endpoint
  ]
}

# Phase 3: Remove old resource
resource "aws_rds_cluster" "old" {
  count = var.migration_phase == "complete" ? 0 : 1
  
  # Existing configuration
  lifecycle {
    prevent_destroy = true
  }
}
```

### 6. Terraform Cloud & Enterprise Architecture

**Answer:**

**Multi-Region Deployment Architecture:**
```hcl
# terraform-cloud-config/main.tf
resource "tfe_organization" "enterprise" {
  name  = "company-terraform"
  email = "terraform-admin@company.com"
  
  # Cost estimation settings
  cost_estimation_enabled = true
  
  # Security settings
  two_factor_conformant = true
}

# Workspace organization by team and environment
resource "tfe_workspace" "team_workspaces" {
  for_each = local.workspace_matrix
  
  name              = "${each.value.team}-${each.value.environment}-${each.value.component}"
  organization      = tfe_organization.enterprise.name
  terraform_version = "1.5.7"
  
  # VCS integration
  vcs_repo {
    identifier     = "company/${each.value.team}-infrastructure"
    branch         = each.value.environment == "prod" ? "main" : "develop"
    oauth_token_id = tfe_oauth_client.github.oauth_token_id
  }
  
  # Environment-specific settings
  working_directory = "environments/${each.value.environment}"
  execution_mode    = "remote"
  
  # Workspace-level variables
  dynamic "variable" {
    for_each = each.value.variables
    content {
      key       = variable.value.key
      value     = variable.value.value
      sensitive = variable.value.sensitive
      category  = variable.value.category
    }
  }
}
```

**Policy Sets Configuration:**
```hcl
resource "tfe_policy_set" "security" {
  name         = "security-compliance"
  organization = tfe_organization.enterprise.name
  
  # Apply to all production workspaces
  workspace_ids = [
    for ws in tfe_workspace.team_workspaces : ws.id
    if strcontains(ws.name, "prod")
  ]
  
  # Policy files
  policy_ids = [
    tfe_sentinel_policy.encryption_required.id,
    tfe_sentinel_policy.cost_limits.id,
    tfe_sentinel_policy.compliance_tags.id,
  ]
}

resource "tfe_sentinel_policy" "encryption_required" {
  name         = "encryption-required"
  organization = tfe_organization.enterprise.name
  policy       = file("policies/encryption.sentinel")
  enforce_mode = "hard-mandatory"
}
```

### 7. Performance Optimization & Scalability

**Answer:**

**Parallelism and Performance Tuning:**
```bash
# Terraform configuration for large-scale deployments
export TF_CLI_ARGS_plan="-parallelism=50"
export TF_CLI_ARGS_apply="-parallelism=50"

# Memory optimization
export TF_PLUGIN_CACHE_DIR="$HOME/.terraform.d/plugin-cache"
export TF_LOG_PROVIDER_AWS=DEBUG  # Provider-specific logging
```

```hcl
# terraform.tf - Resource targeting and partial operations
terraform {
  required_version = ">= 1.5"
  
  # Provider caching configuration
  plugin_cache_dir = "/opt/terraform/plugin-cache"
}

# Use data sources efficiently
data "aws_instances" "existing" {
  filter {
    name   = "tag:Environment"
    values = [var.environment]
  }
}

locals {
  # Batch operations for better performance
  instance_batches = chunklist(data.aws_instances.existing.ids, 100)
}
```

**State Splitting Strategy:**
```hcl
# Core infrastructure state
terraform {
  backend "s3" {
    bucket = "terraform-core-state"
    key    = "core/networking/terraform.tfstate"
  }
}

# Application-specific state with remote state data sources
data "terraform_remote_state" "networking" {
  backend = "s3"
  config = {
    bucket = "terraform-core-state"
    key    = "core/networking/terraform.tfstate"
    region = "us-west-2"
  }
}

# Use remote state outputs efficiently
resource "aws_instance" "app" {
  count           = var.instance_count
  subnet_id       = data.terraform_remote_state.networking.outputs.private_subnet_ids[count.index % 3]
  security_groups = [data.terraform_remote_state.networking.outputs.app_security_group_id]
}
```

**Monitoring and Observability:**
```hcl
# Custom provider for Terraform metrics
resource "datadog_monitor" "terraform_performance" {
  name    = "Terraform Apply Duration"
  type    = "metric alert"
  message = "Terraform apply is taking longer than expected"
  
  query = "avg(last_10m):avg:terraform.apply.duration{environment:prod} > 1800"
  
  thresholds = {
    warning  = 1200
    critical = 1800
  }
}
```

### 8. Infrastructure Security & Compliance Integration

**Answer:**

**Comprehensive Security Framework:**
```hcl
# Security scanning integration
resource "null_resource" "security_scan" {
  triggers = {
    config_hash = filemd5("main.tf")
  }
  
  provisioner "local-exec" {
    command = <<EOF
      # Run security scanners
      tfsec .
      checkov -f main.tf --framework terraform
      terrascan scan -i terraform -t aws
      
      # Custom compliance checks
      opa eval -d compliance/ -i plan.json "data.terraform.compliance.violations"
    EOF
  }
}

# Secret management with external providers
data "aws_secretsmanager_secret_version" "db_credentials" {
  secret_id = "prod/database/credentials"
}

locals {
  db_credentials = jsondecode(data.aws_secretsmanager_secret_version.db_credentials.secret_string)
}

resource "aws_db_instance" "secure" {
  # Use managed secrets
  username = local.db_credentials.username
  password = local.db_credentials.password
  
  # Encryption at rest
  encrypted     = true
  kms_key_id   = aws_kms_key.database.arn
  
  # Network security
  db_subnet_group_name   = aws_db_subnet_group.private.name
  vpc_security_group_ids = [aws_security_group.database.id]
  
  # Backup and monitoring
  backup_retention_period   = 30
  backup_window            = "03:00-04:00"
  maintenance_window       = "sun:04:00-sun:05:00"
  monitoring_interval      = 60
  performance_insights_enabled = true
  
  # Compliance tags
  tags = merge(local.compliance_tags, {
    "DataClassification" = "sensitive"
    "BackupRequired"     = "true"
    "EncryptionRequired" = "true"
  })
}
```

**RBAC and Access Control:**
```hcl
# IAM policy for Terraform service accounts
data "aws_iam_policy_document" "terraform_policy" {
  # Least privilege access
  statement {
    effect = "Allow"
    actions = [
      "ec2:Describe*",
      "ec2:CreateTags",
      "ec2:RunInstances",
    ]
    resources = ["*"]
    
    condition {
      test     = "StringEquals"
      variable = "aws:RequestedRegion"
      values   = ["us-west-2", "us-east-1"]
    }
  }
  
  # Prevent privilege escalation
  statement {
    effect = "Deny"
    actions = [
      "iam:CreateRole",
      "iam:AttachRolePolicy",
      "iam:PutRolePolicy"
    ]
    resources = ["*"]
    
    condition {
      test     = "StringNotEquals"
      variable = "aws:PrincipalTag/TerraformManaged"
      values   = ["true"]
    }
  }
}
```

---

## 🚀 Section 2: Practical Scenarios - Answers

### Scenario 1: Multi-Cloud Disaster Recovery

**Answer:**

```hcl
# modules/multi-cloud-dr/main.tf
terraform {
  required_providers {
    aws = {
      source = "hashicorp/aws"
      version = "~> 5.0"
      configuration_aliases = [aws.primary, aws.dr]
    }
    azurerm = {
      source = "hashicorp/azurerm" 
      version = "~> 3.0"
    }
    google = {
      source = "hashicorp/google"
      version = "~> 4.0"
    }
  }
}

# Abstract resource definition
locals {
  infrastructure_spec = {
    compute = {
      instance_type = var.instance_type_map[var.cloud_provider]
      count        = var.instance_count
      image_id     = var.image_map[var.cloud_provider]
    }
    
    network = {
      vpc_cidr          = var.vpc_cidr
      public_subnets    = var.public_subnets
      private_subnets   = var.private_subnets
      availability_zones = var.az_map[var.cloud_provider]
    }
    
    storage = {
      size = var.storage_size
      type = var.storage_type_map[var.cloud_provider]
      encrypted = true
    }
  }
}

# Provider-agnostic module calls
module "aws_infrastructure" {
  source = "./providers/aws"
  count  = var.cloud_provider == "aws" ? 1 : 0
  
  providers = {
    aws = aws.primary
  }
  
  spec = local.infrastructure_spec
}

module "azure_infrastructure" {
  source = "./providers/azure"
  count  = var.cloud_provider == "azure" ? 1 : 0
  
  spec = local.infrastructure_spec
}

module "gcp_infrastructure" {
  source = "./providers/gcp" 
  count  = var.cloud_provider == "gcp" ? 1 : 0
  
  spec = local.infrastructure_spec
}

# Cross-cloud state synchronization
resource "null_resource" "state_sync" {
  count = var.enable_cross_cloud_sync ? 1 : 0
  
  triggers = {
    infrastructure_hash = sha256(jsonencode(local.infrastructure_spec))
  }
  
  provisioner "local-exec" {
    command = <<EOF
      # Sync state to all cloud backends
      terraform state push -force s3://terraform-state-aws/dr.tfstate
      az storage blob upload --account-name tfstateazure --container-name tfstate --name dr.tfstate --file terraform.tfstate
      gsutil cp terraform.tfstate gs://terraform-state-gcp/dr.tfstate
    EOF
  }
}
```

**Automated Failover Implementation:**
```hcl
# failover/main.tf
resource "aws_route53_health_check" "primary" {
  fqdn              = module.primary_infrastructure.load_balancer_dns
  port              = 443
  type              = "HTTPS"
  resource_path     = "/health"
  failure_threshold = "3"
  
  tags = {
    Name = "Primary Infrastructure Health"
  }
}

resource "aws_route53_record" "failover" {
  zone_id = aws_route53_zone.main.zone_id
  name    = "api.example.com"
  type    = "CNAME"
  
  # Primary record
  set_identifier = "primary"
  failover_routing_policy {
    type = "PRIMARY" 
  }
  health_check_id = aws_route53_health_check.primary.id
  ttl             = 60
  records         = [module.primary_infrastructure.load_balancer_dns]
  
  # Secondary record
  dynamic "alias" {
    for_each = var.enable_dr ? [1] : []
    content {
      set_identifier = "secondary"
      failover_routing_policy {
        type = "SECONDARY"
      }
      ttl     = 60
      records = [module.dr_infrastructure.load_balancer_dns]
    }
  }
}
```

### Scenario 2: Terraform State Surgery

**Answer:**

**State Recovery Plan:**

```bash
#!/bin/bash
# state-recovery.sh

set -euo pipefail

# 1. Create backups
echo "Creating state backups..."
terraform state pull > state-backup-$(date +%Y%m%d-%H%M%S).json
aws s3 cp terraform.tfstate s3://terraform-backups/emergency-backup-$(date +%Y%m%d-%H%M%S).tfstate

# 2. Analyze state corruption
echo "Analyzing state corruption..."
terraform plan -detailed-exitcode > plan-analysis.txt 2>&1 || true

# 3. Identify orphaned resources
echo "Identifying orphaned resources..."
terraform state list > current-state-resources.txt

# Get actual AWS resources
aws ec2 describe-instances --query 'Reservations[].Instances[].[InstanceId,Tags[?Key==`Name`].Value|[0]]' --output table > actual-instances.txt
aws s3api list-buckets --query 'Buckets[].Name' --output table > actual-buckets.txt

# 4. State surgery operations
echo "Performing state surgery..."

# Remove resources that don't exist in reality
while IFS= read -r resource; do
    if ! aws ec2 describe-instances --instance-ids "${resource##*.}" &>/dev/null; then
        echo "Removing orphaned resource: $resource"
        terraform state rm "$resource"
    fi
done < <(grep "aws_instance\." current-state-resources.txt)

# Import resources that exist but not in state
aws ec2 describe-instances --filters "Name=tag:Terraform,Values=true" --query 'Reservations[].Instances[].InstanceId' --output text | while read -r instance_id; do
    if ! terraform state show "aws_instance.${instance_id}" &>/dev/null; then
        echo "Importing missing resource: $instance_id"
        terraform import "aws_instance.recovered_${instance_id}" "$instance_id"
    fi
done

# 5. Address moved resources due to refactoring
echo "Handling moved resources..."
terraform state mv 'aws_instance.old_name[0]' 'aws_instance.new_name'
terraform state mv 'module.old_module' 'module.new_module'

# 6. Validate state consistency
echo "Validating state consistency..."
terraform plan -detailed-exitcode

if [ $? -eq 2 ]; then
    echo "Warning: State still has inconsistencies. Manual review required."
    terraform plan > final-plan-review.txt
else
    echo "State recovery completed successfully!"
fi
```

**Advanced State Manipulation:**
```bash
# Complex resource address changes
terraform state mv \
  'aws_instance.web["us-west-2a"]' \
  'module.web_servers.aws_instance.web["us-west-2a"]'

# Batch operations for large numbers of resources
for i in {0..99}; do
    terraform state mv \
      "aws_instance.workers[${i}]" \
      "module.worker_nodes.aws_instance.workers[${i}]"
done

# Import with custom configuration
cat > import-config.tf << 'EOF'
import {
  to = aws_instance.recovered
  id = "i-1234567890abcdef0"
}

resource "aws_instance" "recovered" {
  # Configuration block for import target
  ami           = "ami-0abcdef1234567890"
  instance_type = "t3.micro"
  # ... other required attributes
}
EOF

terraform plan -generate-config-out=generated.tf
```

### Scenario 3: Custom Provider for Legacy API

**Answer:**

**Provider Implementation for SOAP API:**

```go
// client.go - SOAP client wrapper
type SOAPClient struct {
    baseURL    string
    session    *Session
    rateLimiter *rate.Limiter
    mutex      sync.RWMutex
}

type Session struct {
    Token     string
    ExpiresAt time.Time
}

func (c *SOAPClient) authenticate(ctx context.Context) error {
    c.mutex.Lock()
    defer c.mutex.Unlock()
    
    if c.session != nil && time.Now().Before(c.session.ExpiresAt) {
        return nil // Session still valid
    }
    
    // SOAP authentication request
    envelope := &AuthEnvelope{
        Body: AuthBody{
            Login: LoginRequest{
                Username: c.username,
                Password: c.password,
            },
        },
    }
    
    resp, err := c.soapCall(ctx, "login", envelope)
    if err != nil {
        return fmt.Errorf("authentication failed: %w", err)
    }
    
    c.session = &Session{
        Token:     resp.Token,
        ExpiresAt: time.Now().Add(30 * time.Minute),
    }
    
    return nil
}

func (c *SOAPClient) CreateResource(ctx context.Context, req *CreateResourceRequest) (*ResourceResponse, error) {
    // Rate limiting
    if err := c.rateLimiter.Wait(ctx); err != nil {
        return nil, fmt.Errorf("rate limit exceeded: %w", err)
    }
    
    // Ensure authentication
    if err := c.authenticate(ctx); err != nil {
        return nil, err
    }
    
    // Retry logic for unreliable API
    var resp *ResourceResponse
    err := retry.Do(
        func() error {
            var err error
            resp, err = c.createResourceCall(ctx, req)
            if isRetryableError(err) {
                return err
            }
            return retry.Unrecoverable(err)
        },
        retry.Attempts(3),
        retry.Delay(time.Second*5),
        retry.DelayType(retry.BackOffDelay),
    )
    
    return resp, err
}

// Eventual consistency handling
func (c *SOAPClient) WaitForResource(ctx context.Context, id string, timeout time.Duration) (*Resource, error) {
    ctx, cancel := context.WithTimeout(ctx, timeout)
    defer cancel()
    
    ticker := time.NewTicker(30 * time.Second)
    defer ticker.Stop()
    
    for {
        select {
        case <-ctx.Done():
            return nil, fmt.Errorf("timeout waiting for resource %s to be ready", id)
        case <-ticker.C:
            resource, err := c.GetResource(ctx, id)
            if err != nil {
                continue // Keep trying
            }
            
            if resource.Status == "ACTIVE" {
                return resource, nil
            }
        }
    }
}
```

**Terraform Provider Resource Implementation:**
```go
// resource_network_appliance.go
func (r *networkApplianceResource) Create(ctx context.Context, req resource.CreateRequest, resp *resource.CreateResponse) {
    var plan networkApplianceResourceModel
    diags := req.Plan.Get(ctx, &plan)
    resp.Diagnostics.Append(diags...)
    if resp.Diagnostics.HasError() {
        return
    }
    
    // Create resource via SOAP API
    createReq := &CreateResourceRequest{
        Name:        plan.Name.ValueString(),
        Type:        plan.Type.ValueString(),
        Config:      plan.Configuration.ValueString(),
    }
    
    result, err := r.client.CreateResource(ctx, createReq)
    if err != nil {
        resp.Diagnostics.AddError(
            "Error creating network appliance",
            fmt.Sprintf("Could not create network appliance: %s", err),
        )
        return
    }
    
    // Wait for eventual consistency
    finalResource, err := r.client.WaitForResource(ctx, result.ID, 10*time.Minute)
    if err != nil {
        resp.Diagnostics.AddError(
            "Error waiting for network appliance",
            fmt.Sprintf("Network appliance created but not ready: %s", err),
        )
        return
    }
    
    // Update state with computed values
    plan.ID = types.StringValue(finalResource.ID)
    plan.Status = types.StringValue(finalResource.Status)
    plan.CreatedAt = types.StringValue(finalResource.CreatedAt.String())
    
    diags = resp.State.Set(ctx, plan)
    resp.Diagnostics.Append(diags...)
}

// Custom validators for complex constraints
type applianceConfigValidator struct{}

func (v applianceConfigValidator) ValidateString(ctx context.Context, req validator.StringRequest, resp *validator.StringResponse) {
    if req.ConfigValue.IsNull() || req.ConfigValue.IsUnknown() {
        return
    }
    
    config := req.ConfigValue.ValueString()
    
    // Parse and validate SOAP-specific configuration format
    if !isValidSOAPConfig(config) {
        resp.Diagnostics.AddAttributeError(
            req.Path,
            "Invalid Configuration Format",
            "Configuration must be valid SOAP XML format",
        )
    }
}

func (v applianceConfigValidator) Description(ctx context.Context) string {
    return "validates network appliance configuration format"
}

func (v applianceConfigValidator) MarkdownDescription(ctx context.Context) string {
    return "validates network appliance configuration format"
}
```

---

## 🔧 Section 3: Advanced Technical Questions - Answers

### Question A: Terraform Graph Theory

**Answer:**

Terraform uses sophisticated graph algorithms for dependency management:

**Graph Construction Algorithm:**
1. **Node Creation:** Each resource, data source, and module creates nodes
2. **Edge Creation:** Dependencies create directed edges between nodes
3. **Graph Validation:** Cycle detection using DFS-based algorithms

**Topological Sorting:**
- Uses **Kahn's Algorithm** for topological ordering
- Maintains in-degree count for each vertex
- Processes vertices with zero in-degree first
- Time complexity: O(V + E)

**Cycle Detection:**
```go
// Simplified cycle detection algorithm
func detectCycle(graph *Graph) []string {
    visited := make(map[string]bool)
    recStack := make(map[string]bool)
    
    for vertex := range graph.vertices {
        if !visited[vertex] {
            if dfsHasCycle(graph, vertex, visited, recStack) {
                return getCyclePath(graph, vertex, recStack)
            }
        }
    }
    return nil
}
```

**Performance Impact:**
- **Memory Usage:** O(V + E) where V = resources, E = dependencies
- **Plan Time:** Scales linearly with graph size for most operations
- **Apply Time:** Limited by parallelism and provider response times

**Optimization Strategies:**
- Use `-parallelism` flag to control concurrent operations
- Minimize cross-module dependencies
- Consider state splitting for very large configurations

### Question B: Provider Plugin Architecture

**Answer:**

**gRPC Communication Protocol:**
```protobuf
// terraform-plugin-protocol
service Provider {
    rpc GetProviderSchema(GetProviderSchema.Request) returns (GetProviderSchema.Response);
    rpc ValidateProviderConfig(ValidateProviderConfig.Request) returns (ValidateProviderConfig.Response);
    rpc ConfigureProvider(ConfigureProvider.Request) returns (ConfigureProvider.Response);
    rpc ReadResource(ReadResource.Request) returns (ReadResource.Response);
    rpc PlanResourceChange(PlanResourceChange.Request) returns (PlanResourceChange.Response);
    rpc ApplyResourceChange(ApplyResourceChange.Request) returns (ApplyResourceChange.Response);
}
```

**Provider SDK Internals:**
```go
// Provider plugin serving
func main() {
    plugin.Serve(&plugin.ServeOpts{
        ProviderFunc: func() *schema.Provider {
            return &schema.Provider{
                Schema: map[string]*schema.Schema{
                    "endpoint": {
                        Type:        schema.TypeString,
                        Required:    true,
                        Description: "API endpoint URL",
                    },
                },
                ResourcesMap: map[string]*schema.Resource{
                    "example_resource": resourceExample(),
                },
                ConfigureContextFunc: configureProvider,
            }
        },
    })
}
```

**Error Handling & Recovery:**
```go
// Provider crash recovery
type ProviderClient struct {
    client   proto.ProviderClient
    conn     *grpc.ClientConn
    process  *exec.Cmd
    retries  int
    maxRetries int
}

func (p *ProviderClient) handleProviderCrash() error {
    if p.retries >= p.maxRetries {
        return fmt.Errorf("provider crashed %d times, giving up", p.retries)
    }
    
    p.retries++
    log.Printf("Provider crashed, restarting (attempt %d/%d)", p.retries, p.maxRetries)
    
    // Kill existing process
    if p.process != nil {
        p.process.Process.Kill()
    }
    
    // Start new provider process
    return p.startProvider()
}
```

### Question C: State Locking Mechanisms

**Answer:**

**Comparison of Locking Mechanisms:**

| Backend | Consistency | Performance | Reliability | Complexity |
|---------|-------------|-------------|-------------|------------|
| DynamoDB | Strong | High | Very High | Low |
| Consul | Strong | Medium | High | Medium |
| PostgreSQL | Strong | High | High | Medium |
| Azure Blob | Eventual | Medium | High | Low |

**Custom Locking Implementation:**
```go
// custom-lock-backend/lock.go
type CustomLockBackend struct {
    client    *redis.Client
    lockKey   string
    lockValue string
    ttl       time.Duration
}

func (l *CustomLockBackend) Lock(ctx context.Context, info *statemgr.LockInfo) (string, error) {
    lockID := generateLockID()
    
    // Distributed locking with Redis
    success, err := l.client.SetNX(ctx, l.lockKey, lockID, l.ttl).Result()
    if err != nil {
        return "", fmt.Errorf("failed to acquire lock: %w", err)
    }
    
    if !success {
        // Lock is held, get current holder info
        currentHolder, err := l.client.Get(ctx, l.lockKey).Result()
        if err != nil {
            return "", fmt.Errorf("lock held but cannot get holder info: %w", err)
        }
        
        return "", &statemgr.LockError{
            Info: &statemgr.LockInfo{
                ID:        currentHolder,
                Operation: "unknown",
                Who:       "unknown",
                Version:   "unknown",
                Created:   time.Now(),
            },
        }
    }
    
    // Start lock renewal goroutine
    go l.renewLock(ctx, lockID)
    
    return lockID, nil
}

func (l *CustomLockBackend) renewLock(ctx context.Context, lockID string) {
    ticker := time.NewTicker(l.ttl / 3) // Renew at 1/3 of TTL
    defer ticker.Stop()
    
    for {
        select {
        case <-ctx.Done():
            return
        case <-ticker.C:
            // Extend lock TTL
            err := l.client.Expire(ctx, l.lockKey, l.ttl).Err()
            if err != nil {
                log.Printf("Failed to renew lock %s: %v", lockID, err)
                return
            }
        }
    }
}
```

**High-Frequency CI/CD Locking:**
```yaml
# .github/workflows/terraform.yml
jobs:
  terraform:
    runs-on: ubuntu-latest
    steps:
      - name: Acquire Terraform Lock
        id: lock
        run: |
          # Implement exponential backoff for lock acquisition
          attempt=1
          max_attempts=10
          while [ $attempt -le $max_attempts ]; do
            if terraform init -lock-timeout=60s; then
              echo "Lock acquired on attempt $attempt"
              break
            fi
            
            sleep_time=$((2 ** attempt))
            echo "Lock acquisition failed, retrying in ${sleep_time}s (attempt $attempt/$max_attempts)"
            sleep $sleep_time
            attempt=$((attempt + 1))
          done
          
          if [ $attempt -gt $max_attempts ]; then
            echo "Failed to acquire lock after $max_attempts attempts"
            exit 1
          fi
```

### Question D: Terraform Internals & Extensibility

**Answer:**

**Core Execution Phases:**

```go
// terraform/internal/backend/local/backend_apply.go (simplified)
func (b *Local) opApply(ctx context.Context, op *Operation) (*terraform.State, error) {
    // Phase 1: Initialize
    tfCtx, err := b.context(op)
    if err != nil {
        return nil, err
    }
    
    // Phase 2: Validate configuration
    diags := tfCtx.Validate()
    if diags.HasErrors() {
        return nil, diags.Err()
    }
    
    // Phase 3: Create plan
    plan, diags := tfCtx.Plan()
    if diags.HasErrors() {
        return nil, diags.Err()
    }
    
    // Phase 4: Apply changes
    state, diags := tfCtx.Apply()
    if diags.HasErrors() {
        return state, diags.Err()
    }
    
    return state, nil
}
```

**Extending Terraform without Source Modification:**

**1. Custom Provisioners:**
```go
// custom-provisioner/main.go
type customProvisioner struct{}

func (p *customProvisioner) Apply(ctx context.Context, req provisioner.ApplyRequest) error {
    // Custom provisioning logic
    return nil
}

func main() {
    plugin.Serve(&plugin.ServeOpts{
        ProvisionerFunc: func() provisioner.Interface {
            return &customProvisioner{}
        },
    })
}
```

**2. External Data Sources:**
```hcl
data "external" "custom_logic" {
  program = ["python3", "${path.module}/scripts/custom-data.py"]
  
  query = {
    environment = var.environment
    region     = var.region
  }
}

output "custom_result" {
  value = data.external.custom_logic.result
}
```

**3. Terraform Wrapper Pattern:**
```go
// terraform-wrapper/main.go
func main() {
    // Pre-execution hooks
    if err := preExecutionHooks(); err != nil {
        log.Fatal(err)
    }
    
    // Execute Terraform with custom environment
    cmd := exec.Command("terraform", os.Args[1:]...)
    cmd.Env = append(os.Environ(), 
        "TF_VAR_custom_metadata="+getCustomMetadata(),
        "TF_LOG_PROVIDER=DEBUG",
    )
    
    output, err := cmd.CombinedOutput()
    
    // Post-execution hooks
    postExecutionHooks(output, err)
    
    if err != nil {
        os.Exit(1)
    }
}
```

**4. Custom Functions (Terraform 1.8+):**
```hcl
# terraform.tf
terraform {
  required_version = ">= 1.8"
  
  functions {
    custom_transform = {
      params = [value]
      result = provider::custom::transform(value)
    }
  }
}

# Usage in configuration
locals {
  transformed_data = custom_transform(var.input_data)
}
```

---

*This expert-level mock interview covers advanced Terraform concepts including core architecture, enterprise patterns, custom provider development, performance optimization, and complex real-world scenarios that would be encountered by senior infrastructure engineers and platform architects.*
