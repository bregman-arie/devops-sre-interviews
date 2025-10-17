# Terraform Mock Interview #2 - Intermediate Level - Answer Key

## Question 1: Dynamic Blocks and Complex Resource Configuration

**Answer:**

Dynamic blocks allow you to dynamically construct repeatable nested blocks within resources based on complex values.

**Example: Security Group with Dynamic Rules**

```hcl
variable "security_rules" {
  description = "List of security group rules"
  type = list(object({
    type        = string
    from_port   = number
    to_port     = number
    protocol    = string
    cidr_blocks = list(string)
    description = string
  }))
  default = [
    {
      type        = "ingress"
      from_port   = 80
      to_port     = 80
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
      description = "HTTP traffic"
    },
    {
      type        = "ingress"
      from_port   = 443
      to_port     = 443
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
      description = "HTTPS traffic"
    },
    {
      type        = "egress"
      from_port   = 0
      to_port     = 0
      protocol    = "-1"
      cidr_blocks = ["0.0.0.0/0"]
      description = "All outbound traffic"
    }
  ]
}

resource "aws_security_group" "web_sg" {
  name_prefix = "web-sg"
  vpc_id      = var.vpc_id

  dynamic "ingress" {
    for_each = [for rule in var.security_rules : rule if rule.type == "ingress"]
    content {
      from_port   = ingress.value.from_port
      to_port     = ingress.value.to_port
      protocol    = ingress.value.protocol
      cidr_blocks = ingress.value.cidr_blocks
      description = ingress.value.description
    }
  }

  dynamic "egress" {
    for_each = [for rule in var.security_rules : rule if rule.type == "egress"]
    content {
      from_port   = egress.value.from_port
      to_port     = egress.value.to_port
      protocol    = egress.value.protocol
      cidr_blocks = egress.value.cidr_blocks
      description = egress.value.description
    }
  }

  tags = {
    Name = "web-security-group"
  }
}
```

**Benefits:**
- **Flexibility**: Easily modify rules without changing resource structure
- **DRY principle**: Avoid code duplication
- **Variable-driven**: Rules can come from variables, locals, or data sources
- **Maintainability**: Centralized rule management

**Drawbacks:**
- **Complexity**: Can make code harder to read
- **State management**: Complex changes may require resource recreation
- **Debugging**: More difficult to trace issues
- **Overuse**: Can lead to overly abstract configurations

---

## Question 2: Terraform Providers and Version Constraints

**Answer:**

Provider version constraints ensure compatibility and predictable behavior across environments.

**Version Constraint Operators:**

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"  # Pessimistic constraint
    }
    kubernetes = {
      source  = "hashicorp/kubernetes"
      version = ">= 2.10"  # Minimum version
    }
    helm = {
      source  = "hashicorp/helm"
      version = "= 2.9.0"  # Exact version
    }
  }
  required_version = ">= 1.5"
}
```

**Version Operators Explained:**

- **`= 1.2.3`**: Exact version only
- **`>= 1.2.3`**: Minimum version (allows any higher version)
- **`<= 1.2.3`**: Maximum version (allows any lower version)
- **`~> 1.2.3`**: Pessimistic constraint (allows 1.2.x but not 1.3.x)
- **`~> 1.2`**: Allows 1.x but not 2.x

**Team Environment Best Practices:**

1. **Lock file usage:**
```bash
# Generate lock file
terraform init

# Commit .terraform.lock.hcl to version control
git add .terraform.lock.hcl
git commit -m "Add provider lock file"
```

2. **Provider upgrade process:**
```bash
# Check for available updates
terraform providers

# Upgrade specific provider
terraform init -upgrade

# Upgrade all providers
terraform init -upgrade
```

3. **Team workflow:**
```hcl
# Use conservative version constraints
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"  # Allows 5.x but not 6.x
    }
  }
}
```

---

## Question 3: Local Values and Complex Data Transformation

**Answer:**

Local values allow complex data processing and standardization across configurations.

**Example: Application Configuration Processing**

```hcl
variable "applications" {
  description = "Application configurations"
  type = list(object({
    name        = string
    environment = string
    port        = number
    replicas    = number
    cpu         = string
    memory      = string
    type        = string
  }))
  default = [
    {
      name        = "web-api"
      environment = "production"
      port        = 8080
      replicas    = 3
      cpu         = "500m"
      memory      = "1Gi"
      type        = "web"
    },
    {
      name        = "worker"
      environment = "production"
      port        = 0
      replicas    = 2
      cpu         = "200m"
      memory      = "512Mi"
      type        = "background"
    }
  ]
}

locals {
  # Standardized naming convention
  app_configs = {
    for app in var.applications : app.name => {
      # Standardized resource naming
      resource_name = "${var.project_name}-${app.name}-${app.environment}"
      
      # Standardized labels/tags
      labels = {
        app         = app.name
        environment = app.environment
        type        = app.type
        managed_by  = "terraform"
        project     = var.project_name
      }
      
      # Resource sizing based on environment and type
      resources = {
        cpu = app.environment == "production" ? app.cpu : "100m"
        memory = app.environment == "production" ? app.memory : "256Mi"
      }
      
      # Port configuration
      service_port = app.port > 0 ? app.port : null
      
      # Replica calculation
      replica_count = app.environment == "production" ? app.replicas : 1
      
      # Environment-specific configuration
      config = {
        log_level = app.environment == "production" ? "info" : "debug"
        metrics   = app.environment == "production" ? true : false
      }
    }
  }

  # Group applications by type
  web_apps = {
    for name, config in local.app_configs : 
    name => config if var.applications[index(var.applications.*.name, name)].type == "web"
  }

  # Generate common tags for all resources
  common_tags = {
    Project     = var.project_name
    Environment = var.environment
    ManagedBy   = "terraform"
    CreatedAt   = formatdate("YYYY-MM-DD", timestamp())
  }
}

# Usage in resources
resource "kubernetes_deployment" "apps" {
  for_each = local.app_configs

  metadata {
    name   = each.value.resource_name
    labels = each.value.labels
  }

  spec {
    replicas = each.value.replica_count

    selector {
      match_labels = {
        app = each.key
      }
    }

    template {
      metadata {
        labels = each.value.labels
      }

      spec {
        container {
          name  = each.key
          image = "${var.image_registry}/${each.key}:${var.image_tag}"

          resources {
            requests = each.value.resources
            limits   = each.value.resources
          }

          dynamic "port" {
            for_each = each.value.service_port != null ? [each.value.service_port] : []
            content {
              container_port = port.value
            }
          }

          env {
            name  = "LOG_LEVEL"
            value = each.value.config.log_level
          }

          env {
            name  = "ENABLE_METRICS"
            value = tostring(each.value.config.metrics)
          }
        }
      }
    }
  }
}
```

---

## Question 4: Count vs for_each - When and Why

**Answer:**

**Count vs for_each Comparison:**

| Feature | count | for_each |
|---------|-------|----------|
| **Index Type** | Numeric (0, 1, 2...) | String keys |
| **State Identity** | By index position | By key name |
| **Reordering** | Causes recreation | Safe |
| **Conditional** | Easy with ternary | Requires map manipulation |
| **Best For** | Simple duplication | Complex configurations |

**When to use Count:**

```hcl
# Simple resource duplication
variable "instance_count" {
  description = "Number of instances to create"
  default     = 3
}

resource "aws_instance" "web" {
  count         = var.instance_count
  ami           = "ami-12345678"
  instance_type = "t3.micro"

  tags = {
    Name = "web-server-${count.index + 1}"
  }
}

# Conditional resource creation
variable "create_backup" {
  description = "Whether to create backup resources"
  type        = bool
  default     = false
}

resource "aws_s3_bucket" "backup" {
  count  = var.create_backup ? 1 : 0
  bucket = "my-app-backup-${random_id.bucket.hex}"
}
```

**When to use for_each:**

```hcl
# Complex configurations with unique identities
variable "environments" {
  description = "Environment configurations"
  type = map(object({
    instance_type = string
    min_size      = number
    max_size      = number
    subnets       = list(string)
  }))
  default = {
    dev = {
      instance_type = "t3.micro"
      min_size      = 1
      max_size      = 2
      subnets       = ["subnet-dev1", "subnet-dev2"]
    }
    prod = {
      instance_type = "t3.large"
      min_size      = 2
      max_size      = 10
      subnets       = ["subnet-prod1", "subnet-prod2"]
    }
  }
}

resource "aws_autoscaling_group" "app" {
  for_each = var.environments

  name             = "app-${each.key}"
  min_size         = each.value.min_size
  max_size         = each.value.max_size
  vpc_zone_identifier = each.value.subnets

  launch_template {
    id      = aws_launch_template.app[each.key].id
    version = "$Latest"
  }

  tag {
    key                 = "Name"
    value               = "app-${each.key}"
    propagate_at_launch = true
  }

  tag {
    key                 = "Environment"
    value               = each.key
    propagate_at_launch = true
  }
}
```

**Migration from Count to for_each:**

```hcl
# Original count-based configuration
resource "aws_instance" "web" {
  count         = 3
  ami           = "ami-12345678"
  instance_type = "t3.micro"
}

# Migration steps:
# 1. Remove resources from state
terraform state rm 'aws_instance.web[0]'
terraform state rm 'aws_instance.web[1]'
terraform state rm 'aws_instance.web[2]'

# 2. Change to for_each
resource "aws_instance" "web" {
  for_each = toset(["web-1", "web-2", "web-3"])
  
  ami           = "ami-12345678"
  instance_type = "t3.micro"
  
  tags = {
    Name = each.value
  }
}

# 3. Import existing resources
terraform import 'aws_instance.web["web-1"]' i-1234567890abcdef0
terraform import 'aws_instance.web["web-2"]' i-0987654321fedcba0
terraform import 'aws_instance.web["web-3"]' i-abcdef1234567890f
```

---

## Question 5: Terraform Provisioners and Null Resources

**Answer:**

**Provisioners are generally discouraged because:**
- **Dependency management**: Break Terraform's dependency graph
- **State issues**: Don't track remote changes
- **Error handling**: Limited rollback capabilities
- **Cloud-init alternatives**: Better native solutions exist

**When provisioners might be appropriate:**
- **Legacy systems**: No API available
- **Temporary solutions**: During migration periods
- **Local operations**: File generation, local commands

**Example: Null Resource with local-exec**

```hcl
# Generate kubeconfig after cluster creation
resource "null_resource" "kubeconfig" {
  depends_on = [aws_eks_cluster.main]

  triggers = {
    cluster_endpoint = aws_eks_cluster.main.endpoint
    cluster_name     = aws_eks_cluster.main.name
  }

  provisioner "local-exec" {
    command = <<-EOT
      aws eks update-kubeconfig \
        --region ${var.aws_region} \
        --name ${aws_eks_cluster.main.name} \
        --kubeconfig ./kubeconfig
    EOT
  }

  provisioner "local-exec" {
    when    = destroy
    command = "rm -f ./kubeconfig"
  }
}

# Deploy Kubernetes manifests
resource "null_resource" "kubernetes_manifests" {
  depends_on = [null_resource.kubeconfig]

  triggers = {
    manifests_hash = filemd5("${path.module}/k8s-manifests.yaml")
  }

  provisioner "local-exec" {
    command = <<-EOT
      kubectl apply \
        --kubeconfig ./kubeconfig \
        -f ${path.module}/k8s-manifests.yaml
    EOT
  }
}
```

**Better Alternatives:**

1. **Use dedicated providers:**
```hcl
# Instead of provisioners for Kubernetes
provider "kubernetes" {
  host                   = aws_eks_cluster.main.endpoint
  cluster_ca_certificate = base64decode(aws_eks_cluster.main.certificate_authority[0].data)
  exec {
    api_version = "client.authentication.k8s.io/v1beta1"
    command     = "aws"
    args = ["eks", "get-token", "--cluster-name", aws_eks_cluster.main.name]
  }
}

resource "kubernetes_deployment" "app" {
  # Native Kubernetes resource management
}
```

2. **Use cloud-init:**
```hcl
# Instead of remote-exec provisioners
data "template_cloudinit_config" "config" {
  gzip          = true
  base64_encode = true

  part {
    content_type = "text/cloud-config"
    content = templatefile("${path.module}/cloud-init.yaml", {
      database_url = aws_db_instance.main.endpoint
      api_key     = var.api_key
    })
  }
}

resource "aws_instance" "app" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t3.micro"
  user_data     = data.template_cloudinit_config.config.rendered
}
```

3. **Use configuration management:**
```hcl
# Trigger external configuration management
resource "null_resource" "ansible_playbook" {
  triggers = {
    instance_ids = join(",", aws_instance.web[*].id)
  }

  provisioner "local-exec" {
    command = <<-EOT
      ansible-playbook \
        -i ${join(",", aws_instance.web[*].public_ip)}, \
        --private-key ${var.private_key_path} \
        site.yml
    EOT
  }
}
```

---

## Question 6: Custom Provider Configuration and Aliases

**Answer:**

Provider aliases enable multiple configurations of the same provider within a single Terraform configuration.

**Multi-Region AWS Example:**

```hcl
# Provider configurations with aliases
provider "aws" {
  region = "us-east-1"
  alias  = "primary"
  
  default_tags {
    tags = {
      ManagedBy = "terraform"
      Region    = "us-east-1"
    }
  }
}

provider "aws" {
  region = "us-west-2"
  alias  = "secondary"
  
  default_tags {
    tags = {
      ManagedBy = "terraform"
      Region    = "us-west-2"
    }
  }
}

provider "aws" {
  region = "eu-west-1"
  alias  = "europe"
  
  default_tags {
    tags = {
      ManagedBy = "terraform"
      Region    = "eu-west-1"
    }
  }
}

# Resources using specific providers
resource "aws_s3_bucket" "primary_backup" {
  provider = aws.primary
  bucket   = "app-backup-primary-${random_id.bucket.hex}"
}

resource "aws_s3_bucket" "secondary_backup" {
  provider = aws.secondary  
  bucket   = "app-backup-secondary-${random_id.bucket.hex}"
}

# Cross-region VPC peering
resource "aws_vpc_peering_connection" "cross_region" {
  provider    = aws.primary
  vpc_id      = aws_vpc.primary.id
  peer_vpc_id = aws_vpc.secondary.id
  peer_region = "us-west-2"
  
  tags = {
    Name = "primary-to-secondary-peering"
  }
}

resource "aws_vpc_peering_connection_accepter" "cross_region" {
  provider                  = aws.secondary
  vpc_peering_connection_id = aws_vpc_peering_connection.cross_region.id
  auto_accept               = true
  
  tags = {
    Name = "primary-to-secondary-peering-accepter"
  }
}
```

**Multi-Account Configuration:**

```hcl
# Different AWS accounts
provider "aws" {
  region  = "us-east-1"
  alias   = "production"
  profile = "prod-account"
  
  assume_role {
    role_arn = "arn:aws:iam::111111111111:role/TerraformRole"
  }
}

provider "aws" {
  region  = "us-east-1"  
  alias   = "staging"
  profile = "staging-account"
  
  assume_role {
    role_arn = "arn:aws:iam::222222222222:role/TerraformRole"
  }
}

provider "aws" {
  region  = "us-east-1"
  alias   = "development"
  profile = "dev-account"
  
  assume_role {
    role_arn = "arn:aws:iam::333333333333:role/TerraformRole"
  }
}

# Environment-specific resources
resource "aws_vpc" "main" {
  for_each = {
    prod    = { provider = "production", cidr = "10.0.0.0/16" }
    staging = { provider = "staging", cidr = "10.1.0.0/16" }
    dev     = { provider = "development", cidr = "10.2.0.0/16" }
  }
  
  provider   = aws[each.value.provider]
  cidr_block = each.value.cidr
  
  tags = {
    Name        = "${each.key}-vpc"
    Environment = each.key
  }
}
```

**Authentication Management:**

1. **AWS CLI Profiles:**
```bash
# ~/.aws/config
[profile prod-account]
region = us-east-1
output = json

[profile staging-account]  
region = us-east-1
output = json

[profile dev-account]
region = us-east-1
output = json

# ~/.aws/credentials
[prod-account]
aws_access_key_id = AKIA...
aws_secret_access_key = ...

[staging-account]
aws_access_key_id = AKIA...
aws_secret_access_key = ...

[dev-account]
aws_access_key_id = AKIA...
aws_secret_access_key = ...
```

2. **IAM Role Assumption:**
```hcl
# Cross-account role assumption
data "aws_caller_identity" "current" {}

provider "aws" {
  alias  = "target_account"
  region = var.target_region
  
  assume_role {
    role_arn     = "arn:aws:iam::${var.target_account_id}:role/${var.cross_account_role}"
    session_name = "terraform-${data.aws_caller_identity.current.account_id}"
  }
}
```

3. **Module Usage with Providers:**
```hcl
# modules/networking/main.tf
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

# Root configuration
module "production_network" {
  source = "./modules/networking"
  
  providers = {
    aws = aws.production
  }
  
  environment = "production"
  vpc_cidr    = "10.0.0.0/16"
}

module "staging_network" {
  source = "./modules/networking"
  
  providers = {
    aws = aws.staging
  }
  
  environment = "staging"
  vpc_cidr    = "10.1.0.0/16"
}
```

---

## Question 7: Terraform Backend Migration

**Answer:**

Backend migration requires careful planning to avoid state loss or corruption.

**Migration Process from Local to S3:**

1. **Backup Current State:**
```bash
# Create backup of local state
cp terraform.tfstate terraform.tfstate.backup
cp terraform.tfstate.backup terraform.tfstate.backup.$(date +%Y%m%d_%H%M%S)
```

2. **Prepare S3 Backend Resources:**
```hcl
# backend-setup.tf (separate configuration)
resource "aws_s3_bucket" "terraform_state" {
  bucket = "my-terraform-state-${random_id.bucket_suffix.hex}"
}

resource "aws_s3_bucket_versioning" "terraform_state" {
  bucket = aws_s3_bucket.terraform_state.id
  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_encryption" "terraform_state" {
  bucket = aws_s3_bucket.terraform_state.id

  server_side_encryption_configuration {
    rule {
      apply_server_side_encryption_by_default {
        sse_algorithm = "AES256"
      }
    }
  }
}

resource "aws_s3_bucket_public_access_block" "terraform_state" {
  bucket = aws_s3_bucket.terraform_state.id

  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

resource "aws_dynamodb_table" "terraform_locks" {
  name           = "terraform-state-locks"
  billing_mode   = "PAY_PER_REQUEST"
  hash_key       = "LockID"

  attribute {
    name = "LockID"
    type = "S"
  }
}

resource "random_id" "bucket_suffix" {
  byte_length = 8
}
```

3. **Update Backend Configuration:**
```hcl
# Add to main Terraform configuration
terraform {
  backend "s3" {
    bucket         = "my-terraform-state-bucket"
    key            = "terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-state-locks"
    encrypt        = true
  }
}
```

4. **Perform Migration:**
```bash
# Initialize new backend (will prompt for migration)
terraform init

# Terraform will ask: "Do you want to copy existing state to the new backend?"
# Answer: yes

# Verify migration
terraform plan  # Should show no changes if successful
```

5. **Validation Steps:**
```bash
# Verify state is in S3
aws s3 ls s3://my-terraform-state-bucket/

# Verify lock table
aws dynamodb scan --table-name terraform-state-locks

# Test locking mechanism
terraform plan  # Should acquire and release lock

# Verify state integrity
terraform refresh
terraform plan  # Should show no changes
```

6. **Team Migration Checklist:**
```bash
# Each team member should:
# 1. Pull latest changes
git pull origin main

# 2. Remove local state files
rm -f terraform.tfstate*
rm -rf .terraform/

# 3. Initialize with new backend
terraform init

# 4. Verify configuration
terraform plan
```

**Backend Configuration Best Practices:**

```hcl
# Use separate backend configuration file
# backend.hcl
bucket         = "company-terraform-state"
key            = "environments/production/terraform.tfstate"
region         = "us-east-1"
dynamodb_table = "terraform-state-locks"
encrypt        = true
role_arn       = "arn:aws:iam::ACCOUNT:role/TerraformBackendRole"

# Initialize with backend config file
# terraform init -backend-config=backend.hcl
```

---

## Question 8: Resource Drift Detection and Management

**Answer:**

Configuration drift occurs when infrastructure changes outside of Terraform's control.

**Drift Detection Methods:**

1. **Regular Plan Checks:**
```bash
# Schedule regular drift detection
#!/bin/bash
# drift-check.sh

terraform plan -detailed-exitcode -out=tfplan

EXIT_CODE=$?
if [ $EXIT_CODE -eq 1 ]; then
    echo "Error running terraform plan"
    exit 1
elif [ $EXIT_CODE -eq 2 ]; then
    echo "Drift detected! Changes needed:"
    terraform show tfplan
    
    # Send notification (Slack, email, etc.)
    curl -X POST -H 'Content-type: application/json' \
        --data '{"text":"Terraform drift detected in production!"}' \
        $SLACK_WEBHOOK_URL
fi

rm -f tfplan
```

2. **Automated Drift Detection:**
```yaml
# .github/workflows/drift-detection.yml
name: Terraform Drift Detection

on:
  schedule:
    - cron: '0 9 * * 1-5'  # Weekdays at 9 AM
  workflow_dispatch:

jobs:
  drift-detection:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Terraform
      uses: hashicorp/setup-terraform@v2
      with:
        terraform_version: 1.5.0
    
    - name: Terraform Init
      run: terraform init
    
    - name: Terraform Plan
      id: plan
      run: terraform plan -detailed-exitcode
      continue-on-error: true
    
    - name: Comment Drift Detection Results
      if: steps.plan.outputs.exitcode == 2
      uses: actions/github-script@v6
      with:
        script: |
          github.rest.issues.createComment({
            issue_number: context.issue.number,
            owner: context.repo.owner,
            repo: context.repo.repo,
            body: '⚠️ **Terraform Drift Detected!** Infrastructure has changed outside of Terraform.'
          })
```

3. **Drift Resolution Strategies:**

**Option 1: Import existing changes**
```bash
# Identify drifted resources
terraform plan

# Import manually created resources
terraform import aws_instance.web i-1234567890abcdef0

# Update configuration to match current state
terraform plan  # Should show no changes
```

**Option 2: Revert to desired state**
```bash
# Apply Terraform configuration to fix drift
terraform apply

# For force replacement of drifted resources
terraform apply -replace=aws_instance.web
```

**Option 3: Lifecycle management for acceptable drift**
```hcl
resource "aws_instance" "web" {
  ami           = "ami-12345678"
  instance_type = "t3.micro"
  
  # Ignore changes to tags (might be modified by other tools)
  lifecycle {
    ignore_changes = [
      tags,
      user_data,
    ]
  }
}

# Prevent accidental destruction
resource "aws_db_instance" "main" {
  # ... configuration ...
  
  lifecycle {
    prevent_destroy = true
  }
}
```

4. **Monitoring Tools Integration:**

**Terraform Cloud/Enterprise:**
```hcl
# terraform cloud configuration supports drift detection
terraform {
  cloud {
    organization = "my-org"
    
    workspaces {
      name = "production"
    }
  }
}
```

**Custom Monitoring Script:**
```python
#!/usr/bin/env python3
import subprocess
import json
import boto3

def check_terraform_drift():
    """Check for Terraform configuration drift."""
    try:
        # Run terraform plan with JSON output
        result = subprocess.run([
            'terraform', 'plan', '-json', '-detailed-exitcode'
        ], capture_output=True, text=True)
        
        if result.returncode == 2:
            # Parse plan output for specific changes
            plan_lines = result.stdout.strip().split('\n')
            changes = []
            
            for line in plan_lines:
                try:
                    log_entry = json.loads(line)
                    if log_entry.get('type') == 'resource_drift':
                        changes.append(log_entry)
                except json.JSONDecodeError:
                    continue
            
            if changes:
                send_drift_alert(changes)
                
        return result.returncode
        
    except subprocess.CalledProcessError as e:
        print(f"Error running terraform plan: {e}")
        return 1

def send_drift_alert(changes):
    """Send drift alert to monitoring system."""
    # Send to CloudWatch, Datadog, etc.
    pass

if __name__ == "__main__":
    exit_code = check_terraform_drift()
    exit(exit_code)
```

**Prevention Strategies:**

1. **IAM Policies:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": [
        "ec2:TerminateInstances",
        "ec2:StopInstances"
      ],
      "Resource": "*",
      "Condition": {
        "StringNotEquals": {
          "aws:PrincipalArn": "arn:aws:iam::ACCOUNT:role/TerraformRole"
        }
      }
    }
  ]
}
```

2. **Resource Tagging:**
```hcl
locals {
  common_tags = {
    ManagedBy   = "terraform"
    Project     = var.project_name
    Environment = var.environment
    DoNotModify = "true"
  }
}

resource "aws_instance" "web" {
  # ... configuration ...
  
  tags = merge(local.common_tags, {
    Name = "web-server"
  })
}
```

---

## Question 9: Terraform Testing and Validation

**Answer:**

Comprehensive testing ensures reliable infrastructure deployments.

**Testing Pyramid for Infrastructure:**

1. **Static Analysis (Unit Tests):**

**Terraform Validate:**
```bash
# Basic syntax validation
terraform validate

# Format check
terraform fmt -check=true -diff=true

# Custom validation script
#!/bin/bash
echo "Running Terraform validation..."
terraform validate

echo "Checking Terraform formatting..."
terraform fmt -check=true

echo "Running security scan..."
tfsec .

echo "Running compliance check..."
checkov -d . --framework terraform
```

**TFLint Configuration:**
```hcl
# .tflint.hcl
plugin "terraform" {
  enabled = true
  preset  = "recommended"
}

plugin "aws" {
  enabled = true
  version = "0.21.0"
  source  = "github.com/terraform-linters/tflint-ruleset-aws"
}

rule "terraform_unused_declarations" {
  enabled = true
}

rule "terraform_typed_variables" {
  enabled = true
}
```

2. **Module Testing with Terratest:**

```go
// test/terraform_aws_example_test.go
package test

import (
    "testing"
    "github.com/gruntwork-io/terratest/modules/terraform"
    "github.com/gruntwork-io/terratest/modules/aws"
    "github.com/stretchr/testify/assert"
)

func TestTerraformAwsExample(t *testing.T) {
    t.Parallel()

    // Expected values
    expectedName := "example-vpc"
    expectedRegion := "us-east-1"

    // Terraform options
    terraformOptions := terraform.WithDefaultRetryableErrors(t, &terraform.Options{
        TerraformDir: "../examples/vpc",
        
        Vars: map[string]interface{}{
            "vpc_name": expectedName,
            "region":   expectedRegion,
        },
        
        EnvVars: map[string]string{
            "AWS_DEFAULT_REGION": expectedRegion,
        },
    })

    // Clean up resources after test
    defer terraform.Destroy(t, terraformOptions)

    // Deploy infrastructure
    terraform.InitAndApply(t, terraformOptions)

    // Validate outputs
    vpcId := terraform.Output(t, terraformOptions, "vpc_id")
    assert.NotEmpty(t, vpcId)

    // Validate AWS resources
    vpc := aws.GetVpcById(t, vpcId, expectedRegion)
    assert.Equal(t, expectedName, aws.GetTagsForVpc(t, vpcId, expectedRegion)["Name"])
    assert.Equal(t, "10.0.0.0/16", *vpc.CidrBlock)
}

func TestTerraformModuleExample(t *testing.T) {
    t.Parallel()

    terraformOptions := &terraform.Options{
        TerraformDir: "../modules/web-server",
        Vars: map[string]interface{}{
            "instance_type": "t3.micro",
            "server_name":   "test-server",
        },
    }

    defer terraform.Destroy(t, terraformOptions)
    terraform.InitAndApply(t, terraformOptions)

    // Test that server is accessible
    publicIp := terraform.Output(t, terraformOptions, "public_ip")
    url := fmt.Sprintf("http://%s:8080", publicIp)
    
    http_helper.HttpGetWithRetry(t, url, nil, 200, "Hello, World!", 30, 5*time.Second)
}
```

3. **Integration Testing:**

```yaml
# .github/workflows/terraform-test.yml
name: Terraform Test

on:
  pull_request:
    branches: [ main ]
  push:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        terraform-version: [1.4.0, 1.5.0]
        
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Terraform
      uses: hashicorp/setup-terraform@v2
      with:
        terraform_version: ${{ matrix.terraform-version }}
    
    - name: Terraform Format Check
      run: terraform fmt -check -recursive
    
    - name: Terraform Init
      run: terraform init
    
    - name: Terraform Validate
      run: terraform validate
    
    - name: Run TFSec
      uses: aquasecurity/tfsec-action@v1.0.0
      with:
        soft_fail: true
    
    - name: Run Checkov
      uses: bridgecrewio/checkov-action@master
      with:
        directory: .
        framework: terraform
        soft_fail: true
    
    - name: Setup Go
      uses: actions/setup-go@v3
      with:
        go-version: 1.19
    
    - name: Run Terratest
      run: |
        cd test
        go mod download
        go test -v -timeout 30m
      env:
        AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
        AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

4. **Contract Testing:**

```hcl
# modules/web-server/tests/main.tf
module "test" {
  source = "../"
  
  instance_type = "t3.micro"
  server_name   = "contract-test"
  vpc_id        = "vpc-12345"  # Mock VPC for testing
}

# Test required outputs exist
output "instance_id" {
  description = "Test that instance_id output exists"
  value       = module.test.instance_id
}

output "public_ip" {
  description = "Test that public_ip output exists"  
  value       = module.test.public_ip
}

# Test variable validation
variable "invalid_instance_type" {
  description = "Test validation with invalid instance type"
  type        = string
  default     = "invalid.type"
  
  validation {
    condition = can(regex("^t[3-4]\\.", var.invalid_instance_type))
    error_message = "Instance type must be t3 or t4 family."
  }
}
```

5. **Policy as Code Testing:**

```hcl
# Open Policy Agent (OPA) policies
# policies/terraform.rego
package terraform.analysis

deny[reason] {
  resource := input.planned_values.root_module.resources[_]
  resource.type == "aws_instance"
  resource.values.instance_type == "t3.large"
  reason := "Large instances not allowed in development"
}

deny[reason] {
  resource := input.planned_values.root_module.resources[_]
  resource.type == "aws_s3_bucket"
  not resource.values.server_side_encryption_configuration
  reason := "S3 buckets must have encryption enabled"
}
```

```bash
# Test with OPA
terraform plan -out=tfplan
terraform show -json tfplan > plan.json
opa eval -d policies/ -i plan.json "data.terraform.analysis.deny[x]"
```

**Testing Best Practices:**

1. **Test Structure:**
```
project/
├── modules/
│   ├── vpc/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   │   └── tests/
│   │       ├── main.tf
│   │       └── terraform_test.go
│   └── web-server/
├── environments/
│   ├── dev/
│   ├── staging/
│   └── prod/
├── test/
│   ├── integration_test.go
│   └── e2e_test.go
└── policies/
    └── security.rego
```

2. **Continuous Validation:**
```bash
# Makefile
.PHONY: test validate fmt security

validate:
	terraform validate
	terraform fmt -check=true

security:
	tfsec .
	checkov -d . --framework terraform

test: validate security
	cd test && go test -v -timeout 30m

fmt:
	terraform fmt -recursive

all: fmt validate security test
```

---

## Question 10: Performance Optimization and Large Infrastructure

**Answer:**

Large Terraform configurations require optimization strategies for maintainability and performance.

**Performance Challenges:**
- **State size**: Large state files slow operations
- **API rate limits**: Provider API throttling  
- **Plan time**: Complex dependency graphs
- **Resource drift**: Many resources to check
- **Team coordination**: Multiple developers, state conflicts

**Optimization Strategies:**

1. **State Separation and Workspace Management:**

```hcl
# Separate state files by environment and component
# environments/prod/networking/main.tf
terraform {
  backend "s3" {
    bucket = "terraform-state-prod"
    key    = "networking/terraform.tfstate"
    region = "us-east-1"
  }
}

# environments/prod/compute/main.tf  
terraform {
  backend "s3" {
    bucket = "terraform-state-prod"
    key    = "compute/terraform.tfstate" 
    region = "us-east-1"
  }
}

# environments/prod/databases/main.tf
terraform {
  backend "s3" {
    bucket = "terraform-state-prod"
    key    = "databases/terraform.tfstate"
    region = "us-east-1" 
  }
}
```

2. **Resource Targeting and Parallel Execution:**

```bash
# Target specific resources during development
terraform apply -target=module.networking
terraform apply -target=aws_instance.web[0]

# Increase parallelism (default is 10)
terraform apply -parallelism=20

# Refresh only changed resources
terraform apply -refresh=false
```

3. **Module Structure for Large Infrastructure:**

```hcl
# Root configuration with loose coupling
# main.tf
module "networking" {
  source = "./modules/networking"
  
  vpc_cidr = var.vpc_cidr
  region   = var.region
}

module "compute" {
  source = "./modules/compute"
  
  vpc_id     = module.networking.vpc_id
  subnet_ids = module.networking.private_subnet_ids
  
  depends_on = [module.networking]
}

module "databases" {
  source = "./modules/databases"
  
  vpc_id               = module.networking.vpc_id
  database_subnet_ids  = module.networking.database_subnet_ids
  
  depends_on = [module.networking]
}

# Use data sources to reduce dependencies
data "aws_vpc" "main" {
  filter {
    name   = "tag:Name"
    values = ["main-vpc"]
  }
}

module "application" {
  source = "./modules/application"
  
  vpc_id = data.aws_vpc.main.id  # No direct dependency
}
```

4. **Conditional Resource Creation:**

```hcl
# Avoid creating unnecessary resources
variable "create_monitoring" {
  description = "Create monitoring resources"
  type        = bool
  default     = false
}

resource "aws_cloudwatch_dashboard" "main" {
  count = var.create_monitoring ? 1 : 0
  
  dashboard_name = "infrastructure-monitoring"
  # ... configuration
}

# Dynamic configuration based on environment
locals {
  is_production = var.environment == "production"
  
  instance_config = {
    type  = local.is_production ? "t3.large" : "t3.micro"
    count = local.is_production ? 3 : 1
  }
}

resource "aws_instance" "web" {
  count = local.instance_config.count
  
  instance_type = local.instance_config.type
  # ... configuration
}
```

5. **Terraform Configuration Optimization:**

```hcl
# Use for_each instead of count for better performance
variable "environments" {
  type = map(object({
    instance_count = number
    instance_type  = string
  }))
}

# Better: for_each with map
resource "aws_instance" "app" {
  for_each = var.environments
  
  count         = each.value.instance_count
  instance_type = each.value.instance_type
  
  tags = {
    Environment = each.key
    Name        = "${each.key}-app-${count.index}"
  }
}

# Optimize data source usage
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"] # Canonical
  
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"]
  }
}

locals {
  # Cache AMI ID to avoid repeated lookups
  ubuntu_ami_id = data.aws_ami.ubuntu.id
}
```

6. **CI/CD Pipeline Optimization:**

```yaml
# .github/workflows/terraform-deploy.yml
name: Terraform Deploy

on:
  push:
    branches: [main]
    paths: 
      - 'terraform/**'
      - '.github/workflows/terraform-deploy.yml'

jobs:
  plan:
    runs-on: ubuntu-latest
    outputs:
      changed-dirs: ${{ steps.changes.outputs.all_changed_files }}
      
    steps:
    - uses: actions/checkout@v3
      with:
        fetch-depth: 2
        
    - name: Get changed files
      id: changes
      uses: tj-actions/changed-files@v35
      with:
        files: |
          terraform/**/*.tf
          terraform/**/*.tfvars
        dir_names: true
        dir_names_max_depth: 3
        
    - name: Plan changed directories
      run: |
        for dir in ${{ steps.changes.outputs.all_changed_files }}; do
          echo "Planning $dir"
          cd $dir
          terraform init
          terraform plan -out=tfplan
          cd -
        done

  apply:
    needs: plan
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    strategy:
      matrix:
        directory: ${{ fromJSON(needs.plan.outputs.changed-dirs) }}
      max-parallel: 3  # Limit concurrent applies
      
    steps:
    - name: Apply Terraform
      run: |
        cd ${{ matrix.directory }}
        terraform init
        terraform apply -auto-approve tfplan
```

7. **State Management Best Practices:**

```hcl
# Terraform configuration
terraform {
  required_version = ">= 1.5"
  
  backend "s3" {
    bucket         = "terraform-state-bucket"
    key            = "path/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-locks"
    encrypt        = true
    
    # Optimize state operations
    workspace_key_prefix = "workspaces"
  }
}

# Use remote state data sources for cross-stack references
data "terraform_remote_state" "networking" {
  backend = "s3"
  
  config = {
    bucket = "terraform-state-bucket"
    key    = "networking/terraform.tfstate"
    region = "us-east-1"
  }
}

# Reference outputs without direct module dependencies
locals {
  vpc_id = data.terraform_remote_state.networking.outputs.vpc_id
}
```

8. **Monitoring and Alerting:**

```hcl
# CloudWatch alarms for Terraform operations
resource "aws_cloudwatch_log_group" "terraform" {
  name              = "/aws/terraform/operations"
  retention_in_days = 30
}

resource "aws_cloudwatch_metric_alarm" "terraform_errors" {
  alarm_name          = "terraform-errors"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = "2"
  metric_name         = "ErrorCount"
  namespace           = "Terraform/Operations"
  period              = "300"
  statistic           = "Sum"
  threshold           = "0"
  alarm_description   = "This metric monitors terraform errors"
  
  alarm_actions = [aws_sns_topic.alerts.arn]
}
```

**Performance Monitoring:**

```bash
#!/bin/bash
# terraform-performance.sh

echo "Terraform Performance Analysis"
echo "=============================="

# Measure plan time
echo "Running terraform plan..."
time terraform plan -detailed-exitcode > /dev/null

# State file size
echo "State file size:"
ls -lh terraform.tfstate

# Resource count
echo "Resource count:"
terraform state list | wc -l

# Provider count
echo "Providers in use:"
terraform providers

# Module count
echo "Modules in use:"  
terraform state list | grep "module\." | cut -d. -f1-2 | sort -u | wc -l
```

**Recommended Structure for Large Infrastructure:**

```
infrastructure/
├── global/
│   ├── iam/
│   ├── route53/
│   └── cloudtrail/
├── environments/
│   ├── production/
│   │   ├── networking/
│   │   ├── compute/
│   │   ├── databases/
│   │   ├── monitoring/
│   │   └── security/
│   ├── staging/
│   └── development/
├── modules/
│   ├── vpc/
│   ├── eks/
│   ├── rds/
│   └── monitoring/
└── policies/
    ├── security/
    └── compliance/
```

This approach enables:
- **Parallel development**: Teams can work on different components
- **Targeted deployments**: Deploy only changed components
- **Reduced blast radius**: Failures affect smaller scope
- **Better performance**: Smaller state files and faster operations
- **Easier troubleshooting**: Isolated configurations

---

## 📚 Additional Resources

- **Terraform Documentation**: [terraform.io/docs](https://terraform.io/docs)
- **Terraform Best Practices**: [terraform.io/docs/cloud/guides/recommended-practices](https://terraform.io/docs/cloud/guides/recommended-practices)
- **Terratest**: [terratest.gruntwork.io](https://terratest.gruntwork.io)
- **TFLint**: [github.com/terraform-linters/tflint](https://github.com/terraform-linters/tflint)
- **Checkov**: [checkov.io](https://checkov.io)
- **HashiCorp Learn**: [learn.hashicorp.com/terraform](https://learn.hashicorp.com/terraform)

---

**Next Steps:**
1. Practice with real-world scenarios
2. Set up automated testing pipelines  
3. Implement infrastructure as code best practices
4. Explore advanced Terraform features and providers
