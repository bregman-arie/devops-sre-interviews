# Terraform Mock Interview #3 - Beginner Level - Answer Key (Variables, Locals, Outputs)

## Question 1: Variables vs Locals vs Outputs
**Answer:**
Concept Summary:
- Input Variables: Externalized parameters provided to a module (via parent module, CLI, env vars, tfvars). Change → may alter plan. Public interface.
- Locals: Internal computed values derived from variables, data sources, or other locals. Not overridable externally. Pure convenience/DRY.
- Outputs: Exported values from a module for consumption by parent modules or external tooling. Represent the module's contract.

Lifecycle & Visibility:
| Construct  | Defined In | Set By | Overridable? | Appears in State | Purpose |
|-----------|------------|--------|--------------|------------------|---------|
| Variable  | variables.tf | Caller/CLI/env | Yes | Only if referenced by resources | Parameterize module |
| Local     | locals.tf | Module logic | No | If referenced by resources | Compute/reduce duplication |
| Output    | outputs.tf | Module config | No (value expression) | Yes | Expose results/IDs |

Example:
```hcl
variable "environment" { type = string }
variable "name" { type = string }

locals {
  app_prefix  = "${var.name}-${var.environment}"
  common_tags = {
    Environment = var.environment
    ManagedBy   = "terraform"
  }
}

resource "aws_s3_bucket" "logs" {
  bucket = "${local.app_prefix}-logs"
  tags   = local.common_tags
}

output "bucket_name" { value = aws_s3_bucket.logs.id }
```
Use Cases:
- Variable: Provide environment name ("dev", "prod").
- Local: Generate consistent prefix & tag map.
- Output: Share created bucket name with parent for later referencing.

---

## Question 2: Input Variable Types and Validation
**Answer:**
Examples:
```hcl
variable "replicas" {
  type        = number
  default     = 2
  validation {
    condition     = var.replicas >= 1 && var.replicas <= 10
    error_message = "replicas must be between 1 and 10"
  }
}

variable "allowed_cidrs" {
  type        = list(string)
  default     = ["10.0.0.0/24"]
  validation {
    condition     = alltrue([for c in var.allowed_cidrs : can(regex("^[0-9.]+/\\d{1,2}$", c))])
    error_message = "All CIDRs must be valid"
  }
}

variable "instance_config" {
  type = object({
    type  = string
    cpu   = number
    memory = number
  })
  default = { type = "t3.micro", cpu = 1, memory = 1024 }
}

variable "tags" { type = map(string) }
```
Validation vs Documentation:
- Use validation for hard constraints (range, format).
- Use description for soft guidance (naming conventions) when enforcement not critical.

---

## Question 3: Default Values and Overriding
**Answer:**
Precedence (highest wins):
1. CLI flags: `terraform apply -var="name=web"`
2. `.tfvars` or `-var-file=custom.tfvars`
3. Env vars: `TF_VAR_name=web`
4. Variable default in code
5. Prompt (if no default and not supplied)

Example overrides:
```bash
TF_VAR_environment=prod terraform plan
terraform plan -var-file=prod.tfvars
terraform apply -var="replicas=5"
```

---

## Question 4: Organizing Variables and Outputs in Modules
**Answer:**
Recommended layout:
```
modules/app/
  variables.tf   # Input interface
  locals.tf      # Computed values
  main.tf        # Resources
  outputs.tf     # Exported contract
```
Benefits: Separation clarifies boundaries, eases review, reduces merge conflicts, improves readability.

---

## Question 5: Sensitive Outputs
**Answer:**
Mark outputs sensitive to mask in CLI:
```hcl
output "db_password" {
  value     = aws_db_instance.main.password
  sensitive = true
}
```
Effects: Value hidden in `terraform apply` and `terraform output`; available to parent as opaque; still stored in state (protect backend access).

---

## Question 6: Passing Module Outputs Between Modules
**Answer:**
```hcl
module "network" {
  source = "./modules/network"
  cidr_block = "10.0.0.0/16"
}

module "app" {
  source  = "./modules/app"
  vpc_id  = module.network.vpc_id
}
```
If output type changes (string → object), downstream module evaluation fails with type mismatch; fix by updating consumer module variable type and references.

---

## Question 7: Using Locals for DRY Patterns
**Answer:**
Examples:
1. Naming:
```hcl
locals { base_name = "${var.project}-${var.environment}" }
```
2. Common Tags:
```hcl
locals { tags = { Project = var.project, Env = var.environment } }
```
3. Computed List:
```hcl
locals { zones = [for n in var.az_count : "${var.region}${n}"] }
```
Why not every repeated value? Overuse of locals creates indirection; simple literals can stay inline for clarity.

---

## Question 8: Refactoring Variables into Locals
**Answer:**
Signals to convert variable → local:
- Value never overridden externally.
- Derived entirely from other variables.
Signals to convert local → variable:
- Stakeholders request configurability.
- Reuse requires different values across environments.
Refactor: Move definition, update references, adjust documentation.

---

## Question 9: Output Consumption in Root Module
**Answer:**
After apply:
```bash
terraform output bucket_name
terraform output -json > outputs.json
jq -r '.bucket_name.value' outputs.json
```
Automation (script):
```bash
BUCKET=$(terraform output -raw bucket_name)
aws s3 ls "s3://$BUCKET"
```
In CI: Export via step output; pass to subsequent deployment tasks.

---

## Question 10: Common Pitfalls
**Answer:**
Pitfalls & Fixes:
1. Missing variable types → implicit any; add explicit `type` for validation.
2. Circular local references → ensure locals only reference previously defined values; combine into one locals block.
3. Leaking secrets via outputs → mark `sensitive = true` or avoid outputting secret values.
4. Overexposed outputs (exporting internal resource details) → limit outputs to contract essentials.
5. Hardcoding environment names → use variable with defaults instead of string literals.
6. Massive locals blocks mixing unrelated logic → split logically or convert to module.
7. Using outputs instead of data sources for external infra → outputs only for same stack; use data sources for external.

---

**Summary:** Variables accept external input, locals shape internal logic, outputs publish results. Proper organization and cautious exposure enhance maintainability and security.

**Related:** [Questions](mock_3_questions.md) | [Beginner Mock #2](mock_2_answers.md) | [Intermediate Level](../intermediate/) | [Advanced Level](../advanced/) | [Expert Level](../expert/)
