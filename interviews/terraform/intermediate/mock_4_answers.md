# Terraform Mock Interview #4 - Intermediate Level - Answer Key (Dependency Handling)

## Question 1: Implicit vs Explicit Dependencies
**Answer:**
Terraform constructs a dependency graph implicitly from interpolation references. If resource A references an attribute of resource B, Terraform adds an edge B → A. Most dependencies should be implicit.

Implicit example:
```hcl
resource "aws_subnet" "app" {
  vpc_id = aws_vpc.main.id  # Implicit dependency on aws_vpc.main
}

resource "aws_instance" "web" {
  subnet_id = aws_subnet.app.id  # Chain continues
}
```
Explicit dependencies with `depends_on` are needed when:
- No direct attribute reference exists (e.g., provisioners referencing external files produced by another resource).
- Module output ordering (rare; prefer outputs/inputs references).
- Resources depend on side-effects of `null_resource` or external provisioning.

Explicit example:
```hcl
resource "aws_iam_role" "app" { /* ... */ }
resource "aws_iam_role_policy_attachment" "app_attach" { /* ... */ }

resource "aws_ecs_task_definition" "task" {
  # No direct reference to the attachment, but must wait for policy
  depends_on = [aws_iam_role_policy_attachment.app_attach]
}
```
Common Misuse Patterns:
- Adding `depends_on` “just in case” (unnecessary edges slow planning).
- Using it to serialize unrelated resources (causes artificial bottlenecks).
- Attempting to fix race conditions that stem from external systems (better to model real dependencies using data sources or outputs).

Rule of Thumb: If you can reference an attribute, do that—skip `depends_on`.

---

## Question 2: Module-Level Dependencies
**Answer:**
Three techniques:
1. Input/Output wiring (preferred):
```hcl
module "network" { /* ... */ }
module "compute" {
  vpc_id = module.network.vpc_id  # Implicit dependency via output
}
```
2. Explicit `depends_on` (avoid unless output cannot express dependency):
```hcl
module "compute" {
  source = "./modules/compute"
  depends_on = [module.network]
}
```
3. Remote state data source (cross-stack):
```hcl
data "terraform_remote_state" "network" {
  backend = "s3"
  config = { bucket = "tf-state", key = "network/terraform.tfstate", region = "us-east-1" }
}
module "compute" { vpc_id = data.terraform_remote_state.network.outputs.vpc_id }
```
Preferred: Outputs → inputs. Explicit module `depends_on` only when no output reference exists (e.g., you rely on side-effect like IAM replication delay—still dubious). Remote state is for separate stacks; introduces weaker coupling and potential drift.

---

## Question 3: Avoiding Circular Dependencies
**Answer:**
Cycles arise when two resources/modules reference each other’s attributes directly or through outputs (A needs B; B needs A).

Typical Scenario: Network module outputs a security group ID; app module wants to inject rules that reference application LB which is in app module and then network module expects those rules before output.

Refactoring Strategies:
- Split shared concerns: Extract a “base networking” module containing VPC & subnets; security rules that depend on app resources live in app module referencing base outputs.
- Invert dependency: Pass required inputs downward rather than reaching upward.
- Use data sources: If only an ID is needed, fetch existing resource via data source to break direct output → input cycle.
- Decompose modules further so edges become acyclic.

Example fix:
```hcl
# network_base module outputs vpc_id
module "network_base" { /* ... */ }

# app module depends on vpc_id only
module "app" {
  vpc_id = module.network_base.vpc_id
}

# Optional post-processing module for SG rules referencing app LB
module "network_app_rules" {
  vpc_id    = module.network_base.vpc_id
  lb_arn    = module.app.lb_arn
}
```
Now edges: network_base → app; network_base → network_app_rules; app → network_app_rules; no cycles.

---

## Question 4: Data Sources vs Outputs
**Answer:**
Module outputs:
- Strong coupling; ensures evaluation order.
- Lower risk of drift; values reflect config-managed resources.
- Faster if values already in state.
Data sources:
- Looser coupling; allow referencing pre-existing infra.
- Potential drift: External changes might alter data source results without config updates.
- May incur extra API calls → longer plan time.

Choose outputs when referencing resources under Terraform control in same stack. Choose data sources for external/pre-existing resources or cross-workspace references where remote state is not available.

---

## Question 5: Cross-Workspace / Cross-Environment Dependencies
**Answer:**
Use remote state to pull outputs:
```hcl
data "terraform_remote_state" "network" {
  backend = "s3"
  config = {
    bucket = "tf-state"
    key    = "prod/network/terraform.tfstate"
    region = "us-east-1"
  }
}

resource "aws_subnet" "extra" {
  vpc_id = data.terraform_remote_state.network.outputs.vpc_id
}
```
Trade-offs:
- Pros: Decouples lifecycle; independent applies; clear boundaries.
- Cons: Consistency lag (no shared transaction), risk applying with stale remote state, need ACLs to read remote state.
- Risks: If remote state file moves or is locked, downstream fails; version mismatch of output schema.
Mitigations: Version/tag outputs, add validation (locals checking shape), implement state integrity monitoring.

---

## Question 6: Using `terraform graph` for Debugging
**Answer:**
Generate DOT graph:
```bash
terraform graph > graph.dot
```
Visualize:
```bash
dot -Tpng graph.dot -o graph.png
```
Filter for a chain (grep resource addresses):
```bash
grep -E "aws_vpc.main|aws_subnet.app|aws_instance.web" graph.dot
```
Interpretation: Nodes = resources/data/modules; edges = dependencies. Helps detect unintended edges from attribute references or `depends_on`. If a resource appears earlier than expected, verify no accidental interpolation.

---

## Question 7: Targeted Operations and Partial Graphs
**Answer:**
`-target` prunes graph to selected nodes and their dependencies; skips unrelated pending changes. Repeated use can accumulate drift because skipped resources never converge.

Acceptable Uses:
- Initial bootstrap (create network before referencing it in other stacks).
- Emergency remediation (force fix a single broken resource).

Risks:
- Leaves configuration partially applied.
- Hidden future failures when full apply eventually runs.
- Harder audits; action history fragmented.

Mitigation: Always follow targeted apply with a full plan; document rationale.

---

## Question 8: Forced Recreation with `-replace`
**Answer:**
`-replace=addr` marks the resource for destroy+create even if diff is empty. Dependent resources may also be tainted if they rely on attributes that change (e.g., new instance_id).

Example:
```bash
terraform plan -replace=aws_instance.web -out=plan.tfplan
terraform apply plan.tfplan
```
Cascade: Replaced instance changes IP; downstream DNS record gets updated. If DNS depends on ephemeral attribute, may cause record recreation.

Mitigation:
- Use stable attributes or allocate static resources (e.g., Elastic IP) to reduce cascades.
- Inspect plan JSON to see other actions triggered.
- Consider resource-specific in-place changes if possible.

---

## Question 9: Null Resources and Triggers
**Answer:**
Use `null_resource` for orchestration when Terraform lacks native modeling (e.g., running a script after infra creation). Triggers create implicit dependencies based on values.

Safe Pattern:
```hcl
resource "null_resource" "migrations" {
  triggers = { build_id = var.build_id }
  provisioner "local-exec" { command = "./run_migrations.sh" }
  depends_on = [aws_rds_cluster.db]
}
```
Anti-Pattern:
- Using null_resource to serialize unrelated resources (e.g., delaying EC2 creation with sleep).
- Overusing as a generic pipeline step replacement; better handled in CI/CD outside Terraform.

Remediation: Replace null resources with external continuous delivery steps or proper resource references.

---

## Question 10: Optimizing Large Dependency Graphs
**Answer:**
Techniques:
- Segmentation: Split monolithic config into independently versioned stacks (network, data, app) with remote state boundaries.
- Module granularity: Avoid overly nested trivial modules; keep modules cohesive.
- Reduce dynamic oversized locals and data sources to shrink plan CPU time.
- Parallelism tuning: `-parallelism` lowered for rate-limited APIs; increased for small resource compute heavy applies.
- Caching: Use provider features (e.g., AWS request throttling backoff) and minimal data sources.
- State hygiene: Remove obsolete resources from state to reduce graph size.

Metrics: Track plan duration, node count, memory usage; optimize offending modules.

---

## Question 11: Conditional Dependencies with `count` / `for_each`
**Answer:**
`count` indexes shift when list order changes—can cause destroy/recreate cascades even if semantic identity unchanged. `for_each` keys are stable, preserving identity if map keys stay constant.

Pitfall Example (`count` reorder):
```hcl
variable "subnets" { type = list(string) }
resource "aws_subnet" "app" {
  count  = length(var.subnets)
  cidr_block = var.subnets[count.index]
}
# Reordering var.subnets triggers recreation of all.
```
Stable Example (`for_each`):
```hcl
variable "subnets" { type = set(string) }
resource "aws_subnet" "app" {
  for_each = var.subnets
  cidr_block = each.key
}
# Adding/removing subnets only affects deltas.
```
Conditional with `for_each`:
```hcl
locals { feature_flags = { for k, v in var.flags : k => v if v.enable } }
resource "aws_cloudwatch_log_group" "feature" {
  for_each = local.feature_flags
  name     = "/feature/${each.key}"
}
```

---

## Question 12: Dependency Hygiene & Anti-Patterns
**Answer:**
Common Anti-Patterns:
1. Overuse of `depends_on` → Slows plan, hides real graph intent. Fix: Remove; rely on attribute references.
2. Hidden timing dependencies (assume eventual consistency) → Race conditions (e.g., IAM propagation). Fix: Use retry logic in provisioners or provider features; avoid artificial null_resource delays.
3. Mixing infra + app provisioning in one apply → Churn & large graphs. Fix: Separate infra stack from app deployment pipeline.
4. Cross-stack tight coupling (chain of remote states) → Fragile promotion. Fix: Define minimal, versioned contract outputs.
5. Data sources for internal resources already in same config → Redundant API calls. Fix: Use direct references/outputs.
6. Imperative scripting in provisioners to create infra (e.g., shell creating S3 buckets) → Terraform unaware of resources. Fix: Model resources natively.
7. Large variable-driven dynamic rendering for minor differences → Complexity. Fix: Simplify module parameters; split modules.

Remediation involves pruning artificial edges, clarifying boundaries, and enforcing module contracts.

---

**Summary:** These answers cover modeling dependencies, avoiding cycles, safe explicit dependencies, graph debugging, targeted operation risks, forced replacement behavior, null resource usage, conditional identity stability, and optimization strategies.

**Related:** [Questions](mock_4_questions.md) | [Intermediate Mock #3](mock_3_answers.md) | [Advanced Level](../advanced/) | [Expert Level](../expert/)
