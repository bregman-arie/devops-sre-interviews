# Terraform Mock Interview #1 - Intermediate Level - Answers

## Question 1: Remote State Management

**Answer:**

**Benefits of remote state:**
- **Team collaboration:** Multiple team members can access the same state
- **State locking:** Prevents concurrent modifications
- **Security:** State can be encrypted and access-controlled
- **Backup and versioning:** Cloud providers offer backup and versioning
- **Consistency:** Single source of truth for infrastructure state

**S3 Backend Configuration:**
```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state-bucket"
    key            = "project/terraform.tfstate"
    region         = "us-west-2"
    encrypt        = true
    dynamodb_table = "terraform-state-lock"
    
    # Optional: Use role assumption
    role_arn = "arn:aws:iam::123456789012:role/TerraformStateRole"
  }
}
```

**Additional considerations:**
- **DynamoDB table for locking:** Create table with LockID as partition key
- **Bucket versioning:** Enable versioning on S3 bucket
- **Encryption:** Use KMS keys for encryption at rest
- **Access controls:** Use IAM policies to restrict access
- **Cross-region replication:** For disaster recovery
- **State file structure:** Organize by environment/project

---

## Question 2: Terraform Modules

**Answer:**

**Terraform modules** are containers for multiple resources that are used together. They promote:
- **Code reusability:** Write once, use multiple times
- **Consistency:** Standardized resource configurations
- **Maintainability:** Centralized updates
- **Abstraction:** Hide complexity behind simple interfaces

**Creating a VPC module:**

**File structure:**
```
modules/vpc/
├── main.tf
├── variables.tf
├── outputs.tf
└── versions.tf
```

**modules/vpc/main.tf:**
```hcl
resource "aws_vpc" "main" {
  cidr_block           = var.cidr_block
  enable_dns_hostnames = var.enable_dns_hostnames
  enable_dns_support   = var.enable_dns_support

  tags = merge(var.tags, {
    Name = var.name
  })
}

resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id

  tags = merge(var.tags, {
    Name = "${var.name}-igw"
  })
}

resource "aws_subnet" "public" {
  count = length(var.public_subnet_cidrs)

  vpc_id                  = aws_vpc.main.id
  cidr_block              = var.public_subnet_cidrs[count.index]
  availability_zone       = var.availability_zones[count.index]
  map_public_ip_on_launch = true

  tags = merge(var.tags, {
    Name = "${var.name}-public-${count.index + 1}"
    Type = "Public"
  })
}
```

**modules/vpc/variables.tf:**
```hcl
variable "name" {
  description = "Name prefix for VPC resources"
  type        = string
}

variable "cidr_block" {
  description = "CIDR block for VPC"
  type        = string
  default     = "10.0.0.0/16"
}

variable "public_subnet_cidrs" {
  description = "CIDR blocks for public subnets"
  type        = list(string)
  default     = ["10.0.1.0/24", "10.0.2.0/24"]
}

variable "availability_zones" {
  description = "Availability zones"
  type        = list(string)
}

variable "tags" {
  description = "Tags to apply to resources"
  type        = map(string)
  default     = {}
}
```

**Using the module:**
```hcl
module "dev_vpc" {
  source = "./modules/vpc"
  
  name               = "dev-environment"
  cidr_block         = "10.0.0.0/16"
  public_subnet_cidrs = ["10.0.1.0/24", "10.0.2.0/24"]
  availability_zones = ["us-west-2a", "us-west-2b"]
  
  tags = {
    Environment = "development"
    Project     = "my-app"
  }
}

module "prod_vpc" {
  source = "./modules/vpc"
  
  name               = "prod-environment"
  cidr_block         = "10.1.0.0/16"
  public_subnet_cidrs = ["10.1.1.0/24", "10.1.2.0/24"]
  availability_zones = ["us-west-2a", "us-west-2b"]
  
  tags = {
    Environment = "production"
    Project     = "my-app"
  }
}
```

---

## Question 3: Terraform Workspaces

**Answer:**

**Terraform workspaces** allow you to manage multiple environments with the same configuration by maintaining separate state files.

**Use cases:**
- Multiple environments (dev, staging, prod)
- Feature branch deployments
- Testing different configurations
- Multi-tenant deployments

**Managing environments with workspaces:**

```bash
# Create and switch to dev workspace
terraform workspace new dev
terraform workspace select dev

# Create and switch to staging workspace  
terraform workspace new staging
terraform workspace select staging

# Create and switch to prod workspace
terraform workspace new prod
terraform workspace select prod

# List workspaces
terraform workspace list

# Show current workspace
terraform workspace show
```

**Configuration using workspace:**
```hcl
locals {
  env = terraform.workspace
  
  instance_counts = {
    dev     = 1
    staging = 2
    prod    = 5
  }
  
  instance_types = {
    dev     = "t3.micro"
    staging = "t3.small"
    prod    = "t3.medium"
  }
}

resource "aws_instance" "web" {
  count = local.instance_counts[local.env]
  
  ami           = "ami-12345678"
  instance_type = local.instance_types[local.env]
  
  tags = {
    Name        = "web-${local.env}-${count.index + 1}"
    Environment = local.env
  }
}
```

**Limitations:**
- All workspaces share the same backend configuration
- Can become complex with many environments
- Limited isolation between environments
- State files are in same location (different keys)
- Not suitable for completely different infrastructure patterns

**Better alternatives for complex scenarios:**
- Separate Terraform configurations per environment
- Directory structure with environment folders
- Different backends per environment

---

## Question 4: Data Sources and Lifecycle Rules

**Answer:**

**Data sources** allow Terraform to fetch information about existing resources that are not managed by the current Terraform configuration.

**When to use data sources:**
- Reference existing VPCs, subnets, AMIs
- Get information about resources managed by other Terraform configurations
- Fetch dynamic values (latest AMI, availability zones)
- Reference resources created outside Terraform

**Examples:**
```hcl
# Get latest Amazon Linux 2 AMI
data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
}

# Reference existing VPC
data "aws_vpc" "existing" {
  filter {
    name   = "tag:Name"
    values = ["production-vpc"]
  }
}

# Get availability zones
data "aws_availability_zones" "available" {
  state = "available"
}

# Use data sources
resource "aws_instance" "web" {
  ami               = data.aws_ami.amazon_linux.id
  availability_zone = data.aws_availability_zones.available.names[0]
  subnet_id         = data.aws_vpc.existing.id
}
```

**Lifecycle Rules:**

**1. create_before_destroy:**
```hcl
resource "aws_launch_configuration" "web" {
  name_prefix     = "web-config-"
  image_id        = "ami-12345678"
  instance_type   = "t3.micro"

  lifecycle {
    create_before_destroy = true
  }
}
```
**Use case:** When resource recreation might cause downtime (launch configurations, security groups referenced by other resources)

**2. prevent_destroy:**
```hcl
resource "aws_s3_bucket" "important_data" {
  bucket = "my-critical-data-bucket"

  lifecycle {
    prevent_destroy = true
  }
}
```
**Use case:** Protect critical resources like databases, data buckets from accidental deletion

**3. ignore_changes:**
```hcl
resource "aws_instance" "web" {
  ami           = "ami-12345678"
  instance_type = "t3.micro"

  lifecycle {
    ignore_changes = [
      ami,
      user_data
    ]
  }
}
```
**Use case:** When external systems modify resources or when you want to prevent certain attributes from being updated

---

## Question 5: Terraform Import and State Management

**Answer:**

**Terraform Import Process:**

**1. Identify existing resources:**
```bash
# List existing AWS resources
aws ec2 describe-instances
aws s3 ls
```

**2. Create Terraform configuration:**
```hcl
resource "aws_instance" "existing_server" {
  # Configuration will be filled after import
  ami           = "ami-12345678"
  instance_type = "t3.micro"
  
  tags = {
    Name = "existing-server"
  }
}
```

**3. Import the resource:**
```bash
terraform import aws_instance.existing_server i-1234567890abcdef0
```

**4. Run terraform plan to see differences:**
```bash
terraform plan
```

**5. Update configuration to match current state:**
```hcl
resource "aws_instance" "existing_server" {
  ami                    = "ami-12345678"
  instance_type          = "t3.micro"
  vpc_security_group_ids = ["sg-12345678"]
  subnet_id              = "subnet-12345678"
  
  tags = {
    Name = "existing-server"
  }
}
```

**Challenges and Solutions:**

**Challenge 1: Complex resource dependencies**
- **Solution:** Import resources in dependency order (VPC → Subnets → Security Groups → Instances)

**Challenge 2: Missing or incomplete configuration**
- **Solution:** Use `terraform show` to see current state and match configuration

**Challenge 3: Resource naming conflicts**
- **Solution:** Use `terraform state mv` to rename resources in state

**Challenge 4: Large number of resources**
- **Solution:** Use tools like `terraforming` or `former2` to generate configurations

**State conflict handling:**
```bash
# Remove resource from state without destroying
terraform state rm aws_instance.problematic

# Move resource to different name
terraform state mv aws_instance.old_name aws_instance.new_name

# Replace resource in state
terraform state replace-provider hashicorp/aws registry.terraform.io/hashicorp/aws

# Show current state
terraform state list
terraform state show aws_instance.existing_server
```

---

## Question 6: Advanced Variable Handling

**Answer:**

**Variable Types:**

**1. List:**
```hcl
variable "availability_zones" {
  type    = list(string)
  default = ["us-west-2a", "us-west-2b", "us-west-2c"]
}

# Usage
resource "aws_subnet" "public" {
  count             = length(var.availability_zones)
  availability_zone = var.availability_zones[count.index]
}
```

**2. Map:**
```hcl
variable "instance_types" {
  type = map(string)
  default = {
    dev  = "t3.micro"
    prod = "t3.large"
  }
}

# Usage
resource "aws_instance" "web" {
  instance_type = var.instance_types[terraform.workspace]
}
```

**3. Object:**
```hcl
variable "database_config" {
  type = object({
    engine         = string
    engine_version = string
    instance_class = string
    allocated_storage = number
    multi_az       = bool
  })
  
  default = {
    engine         = "mysql"
    engine_version = "8.0"
    instance_class = "db.t3.micro"
    allocated_storage = 20
    multi_az       = false
  }
}

# Usage
resource "aws_db_instance" "main" {
  engine            = var.database_config.engine
  engine_version    = var.database_config.engine_version
  instance_class    = var.database_config.instance_class
  allocated_storage = var.database_config.allocated_storage
  multi_az          = var.database_config.multi_az
}
```

**4. Tuple:**
```hcl
variable "server_config" {
  type = tuple([string, number, bool])
  default = ["web-server", 8080, true]
}
```

**Conditional Expressions:**
```hcl
variable "environment" {
  type = string
}

locals {
  instance_type = var.environment == "prod" ? "t3.large" : "t3.micro"
  
  # Multiple conditions
  storage_size = var.environment == "prod" ? 100 : var.environment == "staging" ? 50 : 20
}

resource "aws_instance" "web" {
  instance_type = local.instance_type
  
  # Conditional resource creation
  count = var.environment == "prod" ? 3 : 1
}
```

**For Expressions:**
```hcl
variable "users" {
  type = list(object({
    name = string
    role = string
  }))
  
  default = [
    { name = "alice", role = "admin" },
    { name = "bob", role = "developer" },
    { name = "charlie", role = "admin" }
  ]
}

locals {
  # Create list of admin users
  admin_users = [for user in var.users : user.name if user.role == "admin"]
  
  # Create map of users by role
  users_by_role = {
    for user in var.users : user.name => user.role
  }
  
  # Create IAM policies based on users
  admin_policies = {
    for user in var.users : user.name => {
      policy_name = "${user.name}-admin-policy"
      policy_arn  = "arn:aws:iam::aws:policy/AdministratorAccess"
    } if user.role == "admin"
  }
}

# Dynamic resource creation
resource "aws_iam_user" "users" {
  for_each = { for user in var.users : user.name => user }
  name     = each.key
  
  tags = {
    Role = each.value.role
  }
}
```

---

## Question 7: Terraform Functions and Expressions

**Answer:**

**Commonly Used Functions:**

**1. lookup() - Safe map lookups:**
```hcl
variable "instance_types" {
  type = map(string)
  default = {
    dev  = "t3.micro"
    prod = "t3.large"
  }
}

locals {
  # lookup(map, key, default_value)
  instance_type = lookup(var.instance_types, terraform.workspace, "t3.small")
}
```

**2. merge() - Combine maps:**
```hcl
variable "common_tags" {
  type = map(string)
  default = {
    Project = "my-app"
    Owner   = "devops-team"
  }
}

variable "environment_tags" {
  type = map(string)
  default = {
    Environment = "production"
    CostCenter  = "engineering"
  }
}

resource "aws_instance" "web" {
  ami           = "ami-12345678"
  instance_type = "t3.micro"
  
  # Merge multiple tag maps
  tags = merge(
    var.common_tags,
    var.environment_tags,
    {
      Name = "web-server"
      Type = "application"
    }
  )
}
```

**3. templatefile() - Dynamic file content:**
```hcl
# user-data.tpl
#!/bin/bash
yum update -y
yum install -y ${package_name}
echo "Environment: ${environment}" > /etc/environment
echo "Database URL: ${database_url}" >> /etc/environment

%{ for port in ports ~}
echo "Opening port ${port}"
firewall-cmd --permanent --add-port=${port}/tcp
%{ endfor ~}

firewall-cmd --reload
```

```hcl
resource "aws_instance" "web" {
  ami           = "ami-12345678"
  instance_type = "t3.micro"
  
  user_data = templatefile("${path.module}/user-data.tpl", {
    package_name = "httpd"
    environment  = "production"
    database_url = aws_db_instance.main.endpoint
    ports        = [80, 443, 8080]
  })
}
```

**4. length() - Get collection size:**
```hcl
variable "subnets" {
  type = list(string)
  default = ["subnet-1", "subnet-2", "subnet-3"]
}

resource "aws_instance" "web" {
  count = length(var.subnets)
  
  subnet_id = var.subnets[count.index]
  
  tags = {
    Name = "web-${count.index + 1}-of-${length(var.subnets)}"
  }
}
```

**5. Additional useful functions:**
```hcl
locals {
  # String manipulation
  uppercase_name = upper("my-resource")
  lowercase_name = lower("MY-RESOURCE")
  trimmed_name   = trimspace("  my-resource  ")
  
  # File functions
  public_key     = file("~/.ssh/id_rsa.pub")
  config_json    = jsonencode(var.config_object)
  parsed_config  = jsondecode(file("config.json"))
  
  # Date/time
  current_time = timestamp()
  
  # Networking
  vpc_cidr_blocks = cidrsubnets("10.0.0.0/16", 8, 8, 8)  # Creates /24 subnets
  
  # Collections
  unique_values = distinct(["a", "b", "a", "c"])  # ["a", "b", "c"]
  sorted_values = sort(["c", "a", "b"])           # ["a", "b", "c"]
  
  # Type conversion
  string_number = tostring(123)
  number_string = tonumber("123")
  set_to_list   = tolist(toset(["a", "b", "a"]))  # ["a", "b"]
}
```

**Real-world example combining multiple functions:**
```hcl
variable "environments" {
  type = map(object({
    instance_count = number
    instance_type  = string
    subnets        = list(string)
  }))
  
  default = {
    dev = {
      instance_count = 1
      instance_type  = "t3.micro"
      subnets        = ["subnet-dev1"]
    }
    prod = {
      instance_count = 3
      instance_type  = "t3.large"
      subnets        = ["subnet-prod1", "subnet-prod2", "subnet-prod3"]
    }
  }
}

locals {
  current_env = lookup(var.environments, terraform.workspace, var.environments["dev"])
  
  # Create instance configurations
  instances = flatten([
    for i in range(local.current_env.instance_count) : {
      name      = "web-${terraform.workspace}-${i + 1}"
      subnet_id = local.current_env.subnets[i % length(local.current_env.subnets)]
      type      = local.current_env.instance_type
    }
  ])
}
```

---

## Question 8: State Locking and Concurrent Operations

**Answer:**

**State Locking** prevents multiple Terraform operations from running simultaneously on the same state file, which could cause corruption or conflicts.

**Why state locking is important:**
- **Prevents corruption:** Multiple writers could corrupt state file
- **Ensures consistency:** Only one operation modifies infrastructure at a time
- **Avoids conflicts:** Prevents race conditions during apply operations
- **Maintains integrity:** Ensures state accurately reflects infrastructure

**Implementation with different backends:**

**1. S3 + DynamoDB (AWS):**
```hcl
terraform {
  backend "s3" {
    bucket         = "terraform-state-bucket"
    key            = "project/terraform.tfstate"
    region         = "us-west-2"
    encrypt        = true
    dynamodb_table = "terraform-state-lock"  # Required for locking
  }
}
```

**DynamoDB table setup:**
```hcl
resource "aws_dynamodb_table" "terraform_locks" {
  name           = "terraform-state-lock"
  billing_mode   = "PAY_PER_REQUEST"
  hash_key       = "LockID"

  attribute {
    name = "LockID"
    type = "S"
  }

  tags = {
    Name = "Terraform State Lock Table"
  }
}
```

**2. Azure Storage (Azure):**
```hcl
terraform {
  backend "azurerm" {
    storage_account_name = "mystorageaccount"
    container_name       = "tfstate"
    key                  = "project.terraform.tfstate"
    
    # Automatic locking is built-in with Azure Storage
  }
}
```

**3. Google Cloud Storage (GCP):**
```hcl
terraform {
  backend "gcs" {
    bucket = "tf-state-bucket"
    prefix = "project/terraform/state"
    
    # Automatic locking is built-in with GCS
  }
}
```

**Lock lifecycle:**
```bash
# Normal operation
terraform plan   # Acquires read lock
terraform apply  # Acquires write lock

# Lock information
terraform force-unlock <lock_id>  # Emergency unlock
```

**When locks are not properly released:**

**Common causes:**
- Terraform process killed unexpectedly
- Network interruption during operation
- System crash or reboot
- CI/CD pipeline failure

**Resolution steps:**
```bash
# 1. Check if any Terraform processes are running
ps aux | grep terraform

# 2. Verify no other team members are running operations

# 3. Check the lock information
terraform plan  # Will show lock details

# 4. Force unlock if necessary (DANGEROUS - use with caution)
terraform force-unlock 12345678-1234-1234-1234-123456789012

# 5. Verify state integrity after unlock
terraform plan
```

**Best practices for state locking:**
- Always use backends that support locking in team environments
- Never force unlock unless you're certain no operations are running
- Implement proper CI/CD pipelines with lock timeout handling
- Monitor lock duration for stuck operations
- Use separate state files for different components to reduce lock contention

**Lock timeout configuration:**
```bash
# Set timeout for lock acquisition
terraform apply -lock-timeout=10m
```

**Handling locks in CI/CD:**
```yaml
# GitHub Actions example
- name: Terraform Apply
  run: |
    terraform apply \
      -auto-approve \
      -lock-timeout=15m \
      -input=false
  timeout-minutes: 20
```

---

## Question 9: Terraform Plan Files and Targeted Operations

**Answer:**

**Terraform Plan Files:**

Plan files save the execution plan for later use, ensuring consistency between planning and applying changes.

**Creating and using plan files:**
```bash
# Create a plan file
terraform plan -out=tfplan

# Apply the saved plan
terraform apply tfplan

# Review plan file contents (human-readable)
terraform show tfplan

# Convert plan to JSON for programmatic analysis
terraform show -json tfplan > plan.json
```

**When to use plan files:**
- **CI/CD pipelines:** Plan in one stage, apply in another
- **Approval workflows:** Generate plan for review before apply
- **Complex changes:** Ensure exact same changes are applied
- **Audit requirements:** Maintain record of planned changes

**CI/CD Pipeline example:**
```yaml
# Stage 1: Plan
plan_stage:
  script:
    - terraform init
    - terraform plan -out=tfplan
  artifacts:
    paths:
      - tfplan

# Stage 2: Apply (after approval)
apply_stage:
  script:
    - terraform apply tfplan
  when: manual
  dependencies:
    - plan_stage
```

**Targeted Operations:**

Targeted applies allow you to apply changes to specific resources instead of the entire configuration.

**Syntax:**
```bash
# Target specific resource
terraform apply -target=aws_instance.web

# Target multiple resources
terraform apply -target=aws_instance.web -target=aws_security_group.web

# Target module
terraform apply -target=module.vpc

# Target resource within module
terraform apply -target=module.vpc.aws_vpc.main

# Plan with targets
terraform plan -target=aws_instance.web
```

**When targeted operations are necessary:**
- **Dependency issues:** Break circular dependencies
- **Partial deployments:** Deploy specific components first
- **Emergency fixes:** Apply critical fixes without full deployment
- **Resource recovery:** Recreate specific failed resources
- **Testing:** Test changes to specific resources

**When targeted operations are dangerous:**
- **Breaking dependencies:** May leave infrastructure in inconsistent state
- **Incomplete deployments:** Other resources might be out of sync
- **State drift:** Can cause configuration drift over time

**Example scenarios:**

**1. Dependency resolution:**
```bash
# If there's a circular dependency, target the base resource first
terraform apply -target=aws_security_group.database
terraform apply  # Then apply everything else
```

**2. Emergency database fix:**
```bash
# Critical database issue needs immediate fix
terraform apply -target=aws_db_instance.main -var="backup_retention_period=30"
```

**3. Module-specific deployment:**
```bash
# Deploy only the networking module
terraform apply -target=module.networking

# Then deploy application infrastructure
terraform apply -target=module.application
```

**Best practices:**
- Use targeted operations sparingly and with caution
- Always run full `terraform plan` after targeted operations
- Document why targeted operations were used
- Follow up with full apply when possible
- Consider using separate configurations instead of frequent targeting

**Combining plan files with targeting:**
```bash
# Create targeted plan
terraform plan -target=aws_instance.web -out=web-only.tfplan

# Review targeted plan
terraform show web-only.tfplan

# Apply targeted plan
terraform apply web-only.tfplan
```

---

## Question 10: Error Handling and Debugging

**Answer:**

**Systematic Debugging Methodology:**

**1. Initial Assessment:**
```bash
# Enable debug logging
export TF_LOG=DEBUG
export TF_LOG_PATH=terraform-debug.log

# Run with verbose output
terraform apply -no-color 2>&1 | tee apply.log
```

**2. Error Analysis:**

**Common error patterns and solutions:**

**A. Dependency Issues:**
```
Error: Cycle: aws_security_group.web, aws_security_group.db
```

**Solution:**
```bash
# Check resource dependencies
terraform graph | dot -Tpng > dependency-graph.png

# Break circular dependencies by using data sources or separate resources
```

**B. Resource Conflicts:**
```
Error: resource already exists
```

**Solution:**
```bash
# Check if resource exists in state
terraform state list

# Import existing resource or remove from configuration
terraform import aws_instance.web i-1234567890abcdef0

# Or remove conflicting resource
terraform state rm aws_instance.web
```

**C. Provider Issues:**
```
Error: authentication failed
```

**Solution:**
```bash
# Verify credentials
aws sts get-caller-identity

# Check provider configuration
terraform providers

# Re-initialize with updated providers
terraform init -upgrade
```

**3. State Inspection:**
```bash
# List all resources in state
terraform state list

# Show detailed resource information
terraform state show aws_instance.web

# Pull current state
terraform state pull > current-state.json

# Check for state drift
terraform plan -detailed-exitcode
```

**4. Configuration Validation:**
```bash
# Validate syntax and configuration
terraform validate

# Format and check style
terraform fmt -check -diff

# Check for unused variables
grep -r "var\." . --include="*.tf" | grep -v "variable "
```

**5. Advanced Debugging Techniques:**

**A. Module Debugging:**
```bash
# Debug specific module
terraform plan -target=module.vpc

# Check module sources and versions
terraform get -update

# Validate module configuration
cd modules/vpc && terraform validate
```

**B. Provider Debugging:**
```bash
# Test provider authentication separately
terraform console
> data.aws_caller_identity.current

# Check provider version constraints
terraform version

# Use specific provider versions
terraform init -upgrade
```

**C. Network and API Issues:**
```bash
# Test API connectivity
curl -s https://ec2.amazonaws.com/

# Check rate limiting
grep -i "rate" terraform-debug.log

# Retry with delays
terraform apply -parallelism=1
```

**6. Resolution Strategies:**

**Strategy 1: Incremental Recovery**
```bash
# Start with minimal configuration
terraform apply -target=aws_vpc.main
terraform apply -target=module.networking
terraform apply  # Full apply
```

**Strategy 2: State Surgery**
```bash
# Backup state first
cp terraform.tfstate terraform.tfstate.backup

# Remove problematic resources
terraform state rm aws_instance.problematic

# Re-import or recreate
terraform import aws_instance.problematic i-1234567890abcdef0
```

**Strategy 3: Fresh Start (Last Resort)**
```bash
# Export resource information
terraform state pull > old-state.json

# Initialize clean state
rm terraform.tfstate*
terraform init

# Import critical resources
terraform import aws_vpc.main vpc-12345678
terraform import aws_instance.web i-1234567890abcdef0
```

**7. Prevention and Monitoring:**

**Pre-apply Checks:**
```bash
#!/bin/bash
# pre-apply-check.sh

echo "Running pre-apply checks..."

# Validate configuration
terraform validate || exit 1

# Check for drift
terraform plan -detailed-exitcode
if [ $? -eq 2 ]; then
    echo "Drift detected, review changes carefully"
fi

# Check state lock
terraform force-unlock -help >/dev/null 2>&1 || echo "No locks detected"

echo "Pre-apply checks completed"
```

**Post-apply Verification:**
```bash
#!/bin/bash
# post-apply-verify.sh

echo "Running post-apply verification..."

# Verify no pending changes
terraform plan -detailed-exitcode
if [ $? -ne 0 ]; then
    echo "WARNING: Configuration drift detected after apply"
fi

# Test critical resources
aws ec2 describe-instances --instance-ids $(terraform output -raw instance_id)

echo "Post-apply verification completed"
```

**Common Resolution Patterns:**
- **Timeout errors:** Increase timeouts or reduce parallelism
- **Capacity errors:** Check quotas and limits
- **Permission errors:** Verify IAM policies and assume roles
- **Network errors:** Check security groups and NACLs
- **State corruption:** Restore from backup and re-import resources
