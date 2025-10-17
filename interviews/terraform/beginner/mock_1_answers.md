# Terraform Mock Interview #1 - Beginner Level - Answers

## Question 1: What is Terraform?

**Answer:** 
Terraform is an open-source Infrastructure as Code (IaC) tool developed by HashiCorp. It allows you to define, provision, and manage infrastructure resources across multiple cloud providers and services using declarative configuration files written in HashiCorp Configuration Language (HCL).

**Problems it solves:**
- **Manual infrastructure management:** Eliminates error-prone manual provisioning
- **Inconsistency:** Ensures consistent infrastructure across environments
- **Scalability:** Easily replicate and scale infrastructure
- **Documentation:** Infrastructure configuration serves as documentation
- **Version control:** Track changes to infrastructure over time

---

## Question 2: Infrastructure as Code (IaC)

**Answer:**
Infrastructure as Code (IaC) is the practice of managing and provisioning infrastructure through machine-readable configuration files, rather than through manual processes or interactive configuration tools.

**Main benefits:**
- **Consistency:** Same configuration produces identical infrastructure
- **Version Control:** Track changes, rollback if needed
- **Automation:** Reduce manual errors and time
- **Scalability:** Easy to replicate across environments
- **Documentation:** Code serves as living documentation
- **Cost Management:** Better resource tracking and optimization
- **Collaboration:** Teams can review and collaborate on infrastructure changes

---

## Question 3: Terraform Basics

**Answer:**
A Terraform provider is a plugin that enables Terraform to interact with APIs of various services and platforms. Providers are responsible for understanding API interactions and exposing resources.

**Popular providers:**
- **AWS:** Amazon Web Services resources
- **Azure:** Microsoft Azure resources
- **Google Cloud:** Google Cloud Platform resources
- **Kubernetes:** Kubernetes cluster resources
- **Docker:** Docker containers and images
- **GitHub:** GitHub repositories and settings
- **Vault:** HashiCorp Vault secrets management
- **Random:** Generate random values
- **Local:** Local files and commands

---

## Question 4: Terraform Configuration

**Answer:**
A `.tf` file contains Terraform configuration written in HCL (HashiCorp Configuration Language). 

**Main components:**
- **Provider blocks:** Define which providers to use
- **Resource blocks:** Define infrastructure resources to create
- **Variable blocks:** Define input variables
- **Output blocks:** Define output values
- **Data blocks:** Query existing infrastructure
- **Module blocks:** Call reusable modules
- **Terraform block:** Configure Terraform settings

**Example:**
```hcl
provider "aws" {
  region = "us-west-2"
}

resource "aws_instance" "example" {
  ami           = "ami-12345678"
  instance_type = "t2.micro"
}
```

---

## Question 5: Terraform State

**Answer:**
Terraform state is a file that maps real-world resources to your configuration and tracks metadata. It's crucial for Terraform to know which resources it manages and their current state.

**Why it's important:**
- **Resource tracking:** Maps configuration to real resources
- **Performance:** Caches resource attributes for large infrastructures
- **Collaboration:** Ensures team members have consistent view
- **Metadata:** Stores resource dependencies and metadata

**Default storage:** 
- Stored locally in `terraform.tfstate` file
- Should be stored remotely (S3, Azure Storage, etc.) for team collaboration
- Never commit state files to version control

---

## Question 6: Basic Terraform Commands

**Answer:**
The three main Terraform commands for deployment:

1. **`terraform init`**
   - Initializes a Terraform working directory
   - Downloads required providers
   - Sets up backend for state storage

2. **`terraform plan`**
   - Creates an execution plan
   - Shows what actions Terraform will take
   - Doesn't make any changes, just previews

3. **`terraform apply`**
   - Applies the changes required to reach desired state
   - Executes the plan to create/modify/destroy resources
   - Requires confirmation unless `-auto-approve` is used

---

## Question 7: Resource Dependencies

**Answer:**
Terraform automatically determines dependencies between resources by analyzing resource references in the configuration.

**Types of dependencies:**
- **Implicit dependencies:** Terraform detects automatically when one resource references another
- **Explicit dependencies:** Manually defined using `depends_on` argument

**Example:**
```hcl
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}

resource "aws_subnet" "public" {
  vpc_id     = aws_vpc.main.id  # Implicit dependency
  cidr_block = "10.0.1.0/24"
}

resource "aws_instance" "web" {
  subnet_id = aws_subnet.public.id
  depends_on = [aws_vpc.main]  # Explicit dependency
}
```

---

## Question 8: Variables in Terraform

**Answer:**
Variables make Terraform configurations flexible and reusable.

**Defining variables:**
```hcl
variable "instance_type" {
  description = "Type of EC2 instance"
  type        = string
  default     = "t2.micro"
}
```

**Using variables:**
```hcl
resource "aws_instance" "example" {
  instance_type = var.instance_type
}
```

**Ways to set variable values:**
- **terraform.tfvars file:** `instance_type = "t3.small"`
- **Command line:** `terraform apply -var="instance_type=t3.small"`
- **Environment variables:** `TF_VAR_instance_type=t3.small`
- **Default values:** In variable definition
- **Interactive prompt:** Terraform will ask if no value provided

---

## Question 9: Outputs

**Answer:**
Terraform outputs are used to extract and display information about your infrastructure after deployment.

**Uses:**
- Display important information (IP addresses, URLs)
- Pass data between modules
- Share information with other tools
- Provide values to calling modules

**Example:**
```hcl
output "instance_ip" {
  description = "Public IP of the instance"
  value       = aws_instance.web.public_ip
}

output "vpc_id" {
  value = aws_vpc.main.id
  sensitive = false  # Set to true for sensitive data
}
```

**Viewing outputs:**
- After `terraform apply`
- Using `terraform output` command

---

## Question 10: Basic Troubleshooting

**Answer:**
**Basic troubleshooting steps:**

1. **Check the error message:** Read the complete error output carefully
2. **Verify configuration syntax:** Ensure HCL syntax is correct
3. **Check provider authentication:** Ensure credentials are properly configured
4. **Validate configuration:** Run `terraform validate`
5. **Check state file:** Verify state file isn't corrupted
6. **Review plan:** Run `terraform plan` to see what Terraform intends to do
7. **Check resource limits:** Ensure you haven't hit provider limits
8. **Verify permissions:** Ensure adequate permissions for resource creation
9. **Check dependencies:** Ensure all dependencies are properly defined
10. **Use debug logging:** Set `TF_LOG=DEBUG` for detailed logs

**Common issues:**
- Authentication errors
- Resource naming conflicts  
- Insufficient permissions
- Resource limits exceeded
- Syntax errors in configuration
