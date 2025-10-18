# Terraform - Beginner Interview Mock #2 - Answers

> **Difficulty:** Beginner  
> **Duration:** ~35 minutes  
> **Goal:** Test understanding of Terraform basics, resource management, and common workflows.

---

## 🧠 Section 1: Core Concepts

### 1. What is Infrastructure as Code (IaC) and what problems does Terraform solve?

**Answer:**
Infrastructure as Code (IaC) is the practice of managing and provisioning computing infrastructure through machine-readable configuration files rather than through manual processes or interactive configuration tools.

**Problems Terraform solves:**
- **Manual errors:** Eliminates human errors in infrastructure provisioning
- **Inconsistency:** Ensures identical infrastructure across environments
- **Lack of version control:** Infrastructure changes can be tracked and versioned
- **Poor documentation:** Configuration files serve as living documentation
- **Slow provisioning:** Automates and speeds up infrastructure deployment
- **Difficult scaling:** Easy to replicate infrastructure patterns
- **Vendor lock-in:** Works across multiple cloud providers

---

### 2. Explain the difference between declarative and imperative approaches. Which one does Terraform use?

**Answer:**
**Imperative approach:**
- Describes **how** to achieve the desired state
- Step-by-step instructions
- Example: "Create a server, then install nginx, then configure firewall"

**Declarative approach:**
- Describes **what** the desired state should be
- System figures out how to achieve it
- Example: "I want a server running nginx with specific firewall rules"

**Terraform uses the declarative approach.** You define the desired end state of your infrastructure, and Terraform determines the necessary steps to achieve that state.

**Benefits of declarative:**
- More predictable outcomes
- Easier to understand and maintain
- Idempotent operations (can run multiple times safely)
- Better for complex infrastructures

---

### 3. What are the main components of a Terraform configuration file?

**Answer:**
**Main components:**

1. **Provider blocks:** Define which providers to use
   ```hcl
   provider "aws" {
     region = "us-west-2"
   }
   ```

2. **Resource blocks:** Define infrastructure resources
   ```hcl
   resource "aws_instance" "web" {
     ami           = "ami-12345678"
     instance_type = "t2.micro"
   }
   ```

3. **Variable blocks:** Define input parameters
   ```hcl
   variable "instance_type" {
     type = string
     default = "t2.micro"
   }
   ```

4. **Output blocks:** Export values
   ```hcl
   output "instance_ip" {
     value = aws_instance.web.public_ip
   }
   ```

5. **Data blocks:** Query existing resources
   ```hcl
   data "aws_ami" "latest" {
     most_recent = true
     owners      = ["amazon"]
   }
   ```

---

### 4. What is a Terraform provider and can you name a few examples?

**Answer:**
A **Terraform provider** is a plugin that enables Terraform to interact with APIs of cloud providers, SaaS providers, and other services. Providers are responsible for understanding API interactions and exposing resources that can be managed.

**Popular providers:**
- **AWS:** Amazon Web Services (EC2, S3, RDS, etc.)
- **Azure:** Microsoft Azure (Virtual Machines, Storage Accounts, etc.)
- **Google Cloud:** Google Cloud Platform (Compute Engine, Cloud Storage, etc.)
- **Kubernetes:** Kubernetes cluster resources
- **Docker:** Container management
- **GitHub:** Repository and organization management
- **Vault:** HashiCorp Vault for secrets management
- **Random:** Generate random values
- **Local:** Local files and command execution

**Provider configuration example:**
```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```

---

## 🔧 Section 2: Terraform Workflow

### 5. Walk through the basic Terraform workflow commands.

**Answer:**
The basic Terraform workflow consists of four main commands:

**1. `terraform init`**
- Initializes a Terraform working directory
- Downloads required providers
- Sets up backend configuration
- Creates `.terraform` directory

**2. `terraform plan`**
- Creates an execution plan
- Shows what actions Terraform will take
- Compares current state with desired state
- Does not make any changes

**3. `terraform apply`**
- Executes the actions proposed in the plan
- Creates, updates, or destroys resources
- Updates the state file
- Requires confirmation unless `-auto-approve` is used

**4. `terraform destroy`**
- Destroys all resources managed by Terraform
- Useful for cleaning up environments
- Requires confirmation

**Complete workflow example:**
```bash
# Initialize
terraform init

# Plan changes
terraform plan

# Apply changes
terraform apply

# Destroy when done (optional)
terraform destroy
```

---

### 6. What does `terraform plan` do and why is it important?

**Answer:**
`terraform plan` creates an execution plan by:

**What it does:**
- Reads current state file
- Compares with desired configuration
- Queries current state of resources from cloud provider
- Determines what actions are needed (create, update, delete)
- Shows a preview of changes without executing them

**Why it's important:**
- **Safety:** Preview changes before applying them
- **Validation:** Ensures configuration is valid
- **Cost estimation:** Understand resource impact
- **Team review:** Share plans for approval before execution
- **Debugging:** Identify configuration issues early
- **Change management:** Document what will be modified

**Example output:**
```
Plan: 2 to add, 1 to change, 0 to destroy.
```

**Best practices:**
- Always run `terraform plan` before `terraform apply`
- Save plans to files for review: `terraform plan -out=tfplan`
- Review plans carefully in production environments

---

### 7. What's the difference between `terraform apply` and `terraform apply -auto-approve`?

**Answer:**

**`terraform apply` (without flags):**
- Shows the execution plan
- Waits for user confirmation
- User must type "yes" to proceed
- Safer for production environments
- Allows last-minute review

**`terraform apply -auto-approve`:**
- Skips the confirmation prompt
- Automatically applies changes
- Useful for automation and CI/CD pipelines
- Faster but potentially dangerous

**When to use each:**

**Use `terraform apply`:**
- Production environments
- Manual operations
- When you want to review changes one more time
- Learning and development

**Use `terraform apply -auto-approve`:**
- CI/CD pipelines
- Automated deployments
- Development environments
- When you're confident about the changes

**Example:**
```bash
# Interactive (safer)
terraform apply

# Automated (faster)
terraform apply -auto-approve
```

---

### 8. How do you destroy resources created by Terraform?

**Answer:**
**Primary method: `terraform destroy`**
```bash
terraform destroy
```

**Other methods:**

1. **Auto-approve destroy:**
   ```bash
   terraform destroy -auto-approve
   ```

2. **Destroy specific resources:**
   ```bash
   terraform destroy -target=aws_instance.example
   ```

3. **Remove from configuration and apply:**
   - Delete resource from `.tf` files
   - Run `terraform apply`

**What happens during destroy:**
- Terraform reads the state file
- Plans destruction of all managed resources
- Shows what will be destroyed
- Requires confirmation (unless auto-approved)
- Removes resources in reverse dependency order

**Important considerations:**
- **Data loss:** Some resources (databases, storage) may lose data
- **Dependencies:** Terraform handles resource dependencies automatically
- **State file:** Resources are removed from state after successful destruction
- **Partial failures:** Some resources might fail to destroy and need manual cleanup

**Best practices:**
- Always backup important data before destroying
- Use `terraform plan -destroy` to preview what will be destroyed
- Consider using lifecycle rules to prevent accidental destruction

---

## 📁 Section 3: State Management

### 9. What is Terraform state and why is it important?

**Answer:**
**Terraform state** is a file that maps your Terraform configuration to real-world resources and tracks metadata about your infrastructure.

**What it contains:**
- Resource mappings (configuration → real resources)
- Resource metadata and attributes
- Dependency information
- Provider configuration details

**Why it's important:**

1. **Resource tracking:** Knows which resources Terraform manages
2. **Performance:** Caches resource attributes for large infrastructures
3. **Collaboration:** Provides consistent view across team members
4. **Change detection:** Compares desired vs. current state
5. **Dependency management:** Tracks resource relationships
6. **Metadata storage:** Stores resource configuration details

**State file format:**
- JSON format (though you shouldn't edit manually)
- Default name: `terraform.tfstate`
- Contains sensitive information

**Example state content:**
```json
{
  "version": 4,
  "terraform_version": "1.5.0",
  "resources": [
    {
      "type": "aws_instance",
      "name": "example",
      "instances": [...]
    }
  ]
}
```

---

### 10. Where is the Terraform state stored by default, and what are the alternatives?

**Answer:**
**Default storage:**
- **Local file:** `terraform.tfstate` in the working directory
- Suitable for individual development
- Not recommended for team collaboration

**Remote storage alternatives (backends):**

1. **AWS S3:**
   ```hcl
   terraform {
     backend "s3" {
       bucket = "my-terraform-state"
       key    = "prod/terraform.tfstate"
       region = "us-west-2"
     }
   }
   ```

2. **Azure Storage:**
   ```hcl
   terraform {
     backend "azurerm" {
       resource_group_name  = "StorageAccount-ResourceGroup"
       storage_account_name = "tfstate"
       container_name       = "tfstate"
       key                  = "prod.terraform.tfstate"
     }
   }
   ```

3. **Google Cloud Storage:**
   ```hcl
   terraform {
     backend "gcs" {
       bucket = "tf-state-bucket"
       prefix = "terraform/state"
     }
   }
   ```

4. **Terraform Cloud:**
   ```hcl
   terraform {
     backend "remote" {
       organization = "my-org"
       workspaces {
         name = "my-workspace"
       }
     }
   }
   ```

**Best practices:**
- Use remote backends for team collaboration
- Enable versioning on remote storage
- Implement state locking
- Never commit state files to version control

---

### 11. What happens if you accidentally delete the state file?

**Answer:**
**Immediate consequences:**
- Terraform loses track of managed resources
- `terraform plan` will show all resources as new (to be created)
- Risk of creating duplicate resources
- Cannot manage existing infrastructure

**Recovery options:**

1. **Restore from backup:**
   ```bash
   # If you have a backup
   cp terraform.tfstate.backup terraform.tfstate
   ```

2. **Import existing resources:**
   ```bash
   # Import resources one by one
   terraform import aws_instance.example i-1234567890abcdef0
   ```

3. **Remote backend recovery:**
   ```bash
   # If using remote backend with versioning
   terraform init
   # Remote backends often have versioning/backup features
   ```

4. **Recreate state (last resort):**
   - Document all existing resources
   - Import them systematically
   - Very time-consuming for large infrastructures

**Prevention strategies:**
- Use remote backends with versioning
- Regular state file backups
- State locking to prevent concurrent modifications
- Team access controls
- Never manually edit state files

**Example import process:**
```bash
# List resources that need importing
terraform plan

# Import each resource
terraform import aws_instance.web i-1234567890abcdef0
terraform import aws_security_group.web sg-0123456789abcdef0

# Verify state is correct
terraform plan
```

---

### 12. What is state locking and why do you need it?

**Answer:**
**State locking** is a mechanism that prevents multiple Terraform operations from running simultaneously on the same state file, avoiding corruption and conflicts.

**Why you need it:**
- **Prevent corruption:** Multiple concurrent operations can corrupt state
- **Avoid conflicts:** Prevents conflicting changes from different team members
- **Data integrity:** Ensures consistent state updates
- **Team collaboration:** Safe for multiple developers working on same infrastructure

**How it works:**
1. Terraform acquires a lock before state operations
2. Other operations wait or fail if lock is held
3. Lock is released after operation completes
4. Automatic timeout prevents permanent locks

**Backends with locking support:**
- **AWS S3 + DynamoDB:** Uses DynamoDB table for locking
- **Azure Storage:** Built-in blob leasing
- **Google Cloud Storage:** Built-in object locking
- **Terraform Cloud:** Built-in locking
- **Consul:** Uses Consul's locking mechanism

**S3 + DynamoDB example:**
```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-west-2"
    dynamodb_table = "terraform-state-lock"
    encrypt        = true
  }
}
```

**Lock management commands:**
```bash
# Force unlock if lock is stuck
terraform force-unlock LOCK_ID

# Check current lock status
terraform show
```

---

### 13. What's the purpose of `.terraform.lock.hcl` file and should it be committed to version control?

**Answer:**
**Purpose of `.terraform.lock.hcl`:**

The `.terraform.lock.hcl` file is a **dependency lock file** that records the exact provider versions and checksums used in your Terraform configuration.

**What it does:**
- **Version pinning:** Records specific provider versions used
- **Checksum verification:** Stores cryptographic checksums for security
- **Reproducible builds:** Ensures consistent provider versions across environments
- **Supply chain security:** Protects against malicious provider versions

**What it contains:**
```hcl
provider "registry.terraform.io/hashicorp/aws" {
  version     = "5.23.1"
  constraints = "~> 5.0"
  hashes = [
    "h1:abc123...",
    "zh:def456...",
  ]
}
```

**Should it be committed to version control?**
**YES, absolutely!** 

**Reasons to commit:**
- **Team consistency:** All team members use same provider versions
- **Reproducible deployments:** Same versions in all environments
- **Security:** Protects against supply chain attacks
- **Debugging:** Easier to troubleshoot version-related issues
- **CI/CD reliability:** Consistent builds in pipelines

**When is it created/updated:**
- Created during `terraform init`
- Updated when provider versions change
- Updated with `terraform init -upgrade`

**Best practices:**
```bash
# Initial setup - creates lock file
terraform init

# Commit the lock file
git add .terraform.lock.hcl
git commit -m "Add provider lock file"

# Upgrade providers when needed
terraform init -upgrade
git add .terraform.lock.hcl
git commit -m "Update provider versions"
```

**Don't ignore in `.gitignore`:**
```bash
# ❌ Wrong - don't ignore lock file
.terraform.lock.hcl

# ✅ Correct - ignore only .terraform directory
.terraform/
```

---

## 🔄 Section 4: Variables and Outputs

### 14. How do you define and use variables in Terraform?

**Answer:**
**Variable definition:**
```hcl
variable "instance_type" {
  description = "Type of EC2 instance"
  type        = string
  default     = "t2.micro"
}

variable "availability_zones" {
  description = "List of availability zones"
  type        = list(string)
  default     = ["us-west-2a", "us-west-2b"]
}

variable "tags" {
  description = "Tags to apply to resources"
  type        = map(string)
  default = {
    Environment = "dev"
    Project     = "demo"
  }
}
```

**Variable types:**
- `string` - Text values
- `number` - Numeric values
- `bool` - True/false values
- `list(type)` - Ordered list
- `map(type)` - Key-value pairs
- `object({...})` - Complex structures

**Using variables in configuration:**
```hcl
resource "aws_instance" "example" {
  ami           = "ami-12345678"
  instance_type = var.instance_type
  
  availability_zone = var.availability_zones[0]
  
  tags = var.tags
}
```

**Variable validation:**
```hcl
variable "instance_type" {
  type = string
  
  validation {
    condition     = contains(["t2.micro", "t2.small", "t2.medium"], var.instance_type)
    error_message = "Instance type must be t2.micro, t2.small, or t2.medium."
  }
}
```

**Sensitive variables:**
```hcl
variable "db_password" {
  description = "Database password"
  type        = string
  sensitive   = true
}
```

---

### 15. What are the different ways to assign values to Terraform variables?

**Answer:**
**Methods to assign variable values (in order of precedence):**

**1. Command line flags (`-var`):**
```bash
terraform apply -var="instance_type=t2.large" -var="region=us-east-1"
```

**2. Variable files (`-var-file`):**
```bash
terraform apply -var-file="production.tfvars"
```

**3. `terraform.tfvars` file (auto-loaded):**
```hcl
# terraform.tfvars
instance_type = "t2.medium"
region        = "us-west-2"
tags = {
  Environment = "production"
  Team        = "devops"
}
```

**4. `*.auto.tfvars` files (auto-loaded):**
```hcl
# production.auto.tfvars
instance_type = "t2.large"
```

**5. Environment variables (`TF_VAR_`):**
```bash
export TF_VAR_instance_type="t2.small"
export TF_VAR_region="us-east-1"
terraform apply
```

**6. Default values in variable definition:**
```hcl
variable "instance_type" {
  type    = string
  default = "t2.micro"
}
```

**7. Interactive prompts:**
- Terraform prompts for undefined variables
- Not recommended for automation

**Variable file formats:**
```hcl
# HCL format (.tfvars)
instance_type = "t2.medium"
tags = {
  Name = "web-server"
}

# JSON format (.tfvars.json)
{
  "instance_type": "t2.medium",
  "tags": {
    "Name": "web-server"
  }
}
```

**Best practices:**
- Use `.tfvars` files for environment-specific values
- Use environment variables for sensitive data
- Don't commit sensitive `.tfvars` files to version control
- Use descriptive variable names and descriptions

---

### 16. What are outputs in Terraform and when would you use them?

**Answer:**
**Terraform outputs** are a way to extract and display information about your infrastructure after Terraform creates it.

**Basic syntax:**
```hcl
output "instance_ip" {
  description = "Public IP address of the instance"
  value       = aws_instance.web.public_ip
}

output "instance_url" {
  description = "URL to access the web server"
  value       = "http://${aws_instance.web.public_ip}:80"
}
```

**When to use outputs:**

1. **Display important information:**
   ```hcl
   output "database_endpoint" {
     value = aws_db_instance.main.endpoint
   }
   ```

2. **Module integration:**
   ```hcl
   # In a VPC module
   output "vpc_id" {
     value = aws_vpc.main.id
   }
   
   # Using the module
   module "vpc" {
     source = "./modules/vpc"
   }
   
   resource "aws_instance" "web" {
     vpc_security_group_ids = [module.vpc.security_group_id]
   }
   ```

3. **CI/CD pipeline integration:**
   ```bash
   # Get output value in scripts
   INSTANCE_IP=$(terraform output -raw instance_ip)
   curl http://$INSTANCE_IP/health
   ```

4. **Documentation and visibility:**
   ```hcl
   output "important_urls" {
     description = "Important URLs for this deployment"
     value = {
       application = "https://${aws_lb.main.dns_name}"
       monitoring  = "https://monitoring.${var.domain}"
       logs       = "https://logs.${var.domain}"
     }
   }
   ```

**Output attributes:**
```hcl
output "example" {
  description = "Human-readable description"
  value       = aws_instance.web.public_ip
  sensitive   = true  # Hide value from CLI output
}
```

**Viewing outputs:**
```bash
# Show all outputs
terraform output

# Show specific output
terraform output instance_ip

# Get raw value (no quotes)
terraform output -raw instance_ip

# JSON format
terraform output -json
```

---

## 🏗️ Section 5: Practical Scenarios

### 17. You need to create an AWS S3 bucket. Write a basic Terraform configuration.

**Answer:**
**Basic S3 bucket configuration:**

```hcl
# Configure the AWS Provider
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
  required_version = ">= 1.0"
}

# Configure AWS provider
provider "aws" {
  region = var.aws_region
}

# Variable for AWS region
variable "aws_region" {
  description = "AWS region"
  type        = string
  default     = "us-west-2"
}

# Variable for bucket name
variable "bucket_name" {
  description = "Name of the S3 bucket"
  type        = string
}

# Create S3 bucket
resource "aws_s3_bucket" "main" {
  bucket = var.bucket_name
  
  tags = {
    Name        = var.bucket_name
    Environment = "dev"
    ManagedBy   = "terraform"
  }
}

# Configure bucket versioning
resource "aws_s3_bucket_versioning" "main" {
  bucket = aws_s3_bucket.main.id
  versioning_configuration {
    status = "Enabled"
  }
}

# Configure bucket encryption
resource "aws_s3_bucket_server_side_encryption_configuration" "main" {
  bucket = aws_s3_bucket.main.id

  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}

# Block public access
resource "aws_s3_bucket_public_access_block" "main" {
  bucket = aws_s3_bucket.main.id

  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

# Output bucket information
output "bucket_name" {
  description = "Name of the created S3 bucket"
  value       = aws_s3_bucket.main.id
}

output "bucket_arn" {
  description = "ARN of the created S3 bucket"
  value       = aws_s3_bucket.main.arn
}

output "bucket_domain_name" {
  description = "Domain name of the S3 bucket"
  value       = aws_s3_bucket.main.bucket_domain_name
}
```

**Variables file (terraform.tfvars):**
```hcl
aws_region  = "us-west-2"
bucket_name = "my-terraform-bucket-unique-name-123"
```

**Deployment commands:**
```bash
# Initialize Terraform
terraform init

# Plan the deployment
terraform plan

# Apply the configuration
terraform apply

# Check outputs
terraform output
```

---

### 18. How would you create multiple similar resources (e.g., 3 EC2 instances)?

**Answer:**
**Method 1: Using `count`**

```hcl
variable "instance_count" {
  description = "Number of instances to create"
  type        = number
  default     = 3
}

resource "aws_instance" "web" {
  count = var.instance_count
  
  ami           = "ami-0c02fb55956c7d316"
  instance_type = "t2.micro"
  
  tags = {
    Name = "web-server-${count.index + 1}"
    Environment = "dev"
  }
}

# Output all instance IPs
output "instance_ips" {
  description = "Public IP addresses of all instances"
  value       = aws_instance.web[*].public_ip
}
```

**Method 2: Using `for_each` with list**

```hcl
variable "instance_names" {
  description = "Names for the instances"
  type        = list(string)
  default     = ["web-1", "web-2", "web-3"]
}

resource "aws_instance" "web" {
  for_each = toset(var.instance_names)
  
  ami           = "ami-0c02fb55956c7d316"
  instance_type = "t2.micro"
  
  tags = {
    Name = each.value
    Environment = "dev"
  }
}

# Output instance IPs with names
output "instance_details" {
  description = "Instance details"
  value = {
    for name, instance in aws_instance.web : name => {
      id        = instance.id
      public_ip = instance.public_ip
    }
  }
}
```

**Method 3: Using `for_each` with map (most flexible)**

```hcl
variable "instances" {
  description = "Map of instance configurations"
  type = map(object({
    instance_type = string
    availability_zone = string
  }))
  default = {
    web-1 = {
      instance_type     = "t2.micro"
      availability_zone = "us-west-2a"
    }
    web-2 = {
      instance_type     = "t2.small"
      availability_zone = "us-west-2b"
    }
    web-3 = {
      instance_type     = "t2.micro"
      availability_zone = "us-west-2c"
    }
  }
}

resource "aws_instance" "web" {
  for_each = var.instances
  
  ami               = "ami-0c02fb55956c7d316"
  instance_type     = each.value.instance_type
  availability_zone = each.value.availability_zone
  
  tags = {
    Name = each.key
    Environment = "dev"
  }
}
```

**When to use each method:**
- **`count`:** Simple duplication with numeric indexing
- **`for_each` with list:** Named instances with simple configuration
- **`for_each` with map:** Complex configurations per instance

**Best practices:**
- Prefer `for_each` over `count` for better resource management
- Use descriptive names in `for_each` keys
- Consider using modules for complex resource patterns

---

### 19. Your `terraform apply` failed. What steps would you take to troubleshoot?

**Answer:**
**Systematic troubleshooting approach:**

**1. Read the error message carefully:**
```bash
terraform apply
# Look for specific error details, resource names, and error codes
```

**2. Check Terraform and provider versions:**
```bash
terraform version
# Ensure compatibility between Terraform and provider versions
```

**3. Validate configuration syntax:**
```bash
terraform validate
# Check for syntax errors in configuration files
```

**4. Format and check configuration:**
```bash
terraform fmt -check
# Ensure proper formatting
```

**5. Review the plan:**
```bash
terraform plan
# See if the plan reveals issues before applying
```

**6. Enable detailed logging:**
```bash
export TF_LOG=DEBUG
terraform apply
# Get detailed logs for debugging
```

**Common issues and solutions:**

**Authentication/Permission errors:**
```bash
# Check AWS credentials
aws sts get-caller-identity

# Verify permissions
aws iam get-user
```

**Resource conflicts:**
```bash
# Check if resources already exist
terraform import aws_instance.example i-1234567890abcdef0
```

**State lock issues:**
```bash
# Check for stuck locks
terraform force-unlock LOCK_ID
```

**Provider/version issues:**
```bash
# Reinitialize providers
terraform init -upgrade

# Lock provider versions
terraform providers lock
```

**Network/connectivity issues:**
```bash
# Test connectivity
curl -I https://registry.terraform.io/

# Check firewall/proxy settings
```

**Resource quota/limits:**
- Check cloud provider quotas
- Verify resource limits in the region
- Use different instance types or regions

**State corruption:**
```bash
# Check state file integrity
terraform show

# Restore from backup if needed
cp terraform.tfstate.backup terraform.tfstate
```

**Debugging steps:**
1. Isolate the problem (comment out resources)
2. Apply resources incrementally
3. Check provider documentation
4. Search for similar issues online
5. Use `-target` to apply specific resources

**Example debugging session:**
```bash
# Step 1: Enable logging
export TF_LOG=DEBUG

# Step 2: Try with target
terraform apply -target=aws_security_group.web

# Step 3: Check state
terraform state list

# Step 4: Refresh state
terraform refresh

# Step 5: Import if needed
terraform import aws_instance.web i-1234567890abcdef0
```

**Prevention strategies:**
- Always run `terraform plan` first
- Use version constraints for providers
- Implement proper CI/CD validation
- Regular state backups
- Use remote state with locking
- Test in development environments first

---

### 20. How can you save a Terraform plan to a file and apply it later? What are the benefits of this approach?

**Answer:**

You can save a Terraform plan to a file using the `-out` flag and apply it later using the plan file:

**Saving a plan:**
```bash
terraform plan -out=planfile.tfplan
```

**Applying the saved plan:**
```bash
terraform apply planfile.tfplan
```

**Benefits of this approach:**

1. **CI/CD approval workflows:** Separate planning from execution phases for security reviews
2. **Deterministic deployments:** Ensures exactly what was reviewed gets deployed
3. **Team collaboration:** Share plans for review before execution  
4. **Audit trail:** Document what changes will be made
5. **Risk reduction:** Prevents drift between plan and apply phases
6. **Automation-friendly:** Perfect for automated pipelines where human approval is needed

**Common workflow example:**
```bash
# Step 1: Create and save plan
terraform plan -out=production.tfplan

# Step 2: Review the plan file (manual approval step)
terraform show production.tfplan

# Step 3: Apply the exact plan (no surprises)
terraform apply production.tfplan
```

**Important notes:**
- Plan files are binary and environment-specific
- Should not be stored in version control
- Have expiration considerations for long approval workflows
- Contain sensitive data - handle securely

This approach is especially useful for CI/CD approval workflows or ensuring deterministic deployments where you need to guarantee that what gets reviewed is exactly what gets deployed.

---

**Return to:** [Questions](mock_2_questions.md)

**Related:** [Mock Interview #1](mock_1_questions.md) | [Intermediate Level](../intermediate/)

**Level:** Beginner  
**Total Questions:** 20  
**Estimated Time:** 40 minutes
