# Terraform Mock Interview #3 - Intermediate Level - Answer Key

## Question 1: Terraform Plan Files and CI/CD Workflows

**Answer:**

You save a Terraform plan to a binary file with:

```bash
terraform init
terraform plan -out=plan.tfplan
```

Later you apply exactly what was reviewed:

```bash
terraform apply plan.tfplan
```

This separates the calculation of changes (plan) from execution (apply). The plan encapsulates the diff between current state and desired configuration, including which actions Terraform will take (create, update, destroy).

**Benefits:**
- Determinism: Guarantees the applied changes match the reviewed diff.
- Approval workflows: Security teams or leads can review before any apply.
- Separation of duties: One role generates the plan, another applies it.
- Auditability: Artifact (plan file + JSON rendering) can be archived for compliance.
- Prevents drift from last-minute config edits between plan and apply.

**Risks / Caveats:**
- Staleness: If remote state or real infrastructure changes after plan creation, applying a stale plan may error or act on outdated assumptions.
- Sensitive data: Some attribute values (even when marked sensitive) may appear in the JSON form; treat plan artifacts as sensitive.
- Coupling to provider versions: Plan produced under one provider version must be applied under the same versions (lock with `.terraform.lock.hcl`).
- Storage overhead: Large environments produce large plan files; manage retention.
- Insider tampering: Ensure artifact integrity (checksum/signing) so the plan file isn’t swapped.

**Best Practices:**
1. Generate plan on immutable CI runner.
2. Store plan in a secure artifact store (e.g., encrypted S3 bucket with short TTL).
3. Attach summary (parsed JSON) to PR for human review.
4. On approval, download the exact artifact and run `terraform apply plan.tfplan`.
5. Re-run a lightweight drift check (e.g., `terraform plan -refresh-only`) before apply; abort if unexpected drift occurred.

---

## Question 2: Deterministic Deployments and Plan Analysis

**Answer:**

Deterministic deployments mean the configuration reviewed is the one executed. Terraform plan files plus provider lock files achieve this:

Workflow:
1. Developer pushes branch → CI runs: `terraform fmt -check`, `terraform init`, `terraform validate`.
2. CI generates: `terraform plan -out=plan.tfplan`.
3. CI renders machine-readable diff: `terraform show -json plan.tfplan > plan.json`.
4. A script summarizes actions (create/update/destroy) and comments on the PR.
5. Reviewers approve; plan artifact (plan.tfplan) is stored with commit SHA (immutable).
6. Merge triggers deployment pipeline that fetches the artifact and runs `terraform apply plan.tfplan`.
7. Provider versions are pinned via `.terraform.lock.hcl` to avoid resolution differences.

**GitOps Integration:**
- Git is the source of truth; only merged code is applied.
- No manual `terraform apply` outside pipelines.
- Policy check stage: Use OPA/Conftest against `plan.json` to block destructive changes unless annotated.
- Integrity: Optionally sign `plan.tfplan` (e.g., attach SHA256; verify before apply).

**Key Determinism Elements:**
- Locked provider versions.
- Saved plan artifact.
- Immutable commit references for environment promotion.

---

## Question 3: Plan File Security and Storage

**Answer:**

Plan files can hold resource addresses, proposed values, and sometimes derived secret metadata. Although Terraform obscures values marked `sensitive` in plan output, indirect leakage (names, counts, tags) can still be sensitive.

**Security Considerations:**
- Encryption at rest: Store artifacts in encrypted storage (S3 + SSE, Vault, Artifactory).
- Access control: Restrict download/apply permissions (role-based access in CI).
- Ephemeral retention: Delete plan files after successful apply; short TTL to limit exposure.
- Transmission: Use TLS on artifact APIs; avoid emailing plans.
- Integrity: Hash/sign plan files (SHA256 + Sigstore) to mitigate tampering.
- Logging hygiene: Avoid dumping full JSON plans to public logs; redact sensitive sections.
- Least privilege: CI service account only needs read/write to state backend and limited cloud permissions.

**Storage & Cleanup Strategy:**
1. Artifact saved with naming convention: `plan-${commit_sha}.tfplan`.
2. Metadata recorded: commit SHA, timestamp, environment.
3. Apply pipeline verifies hash before execution.
4. Post-apply job securely deletes artifact (or marks archived for compliance only).
5. Scheduled job cleans unused plans older than retention window (e.g., 7 days).

---

## Question 4: Advanced Planning Strategies

**Answer:**

Command differences:

1. `terraform plan` (default exit codes):
   - Exit 0: Success (changes may or may not exist)
   - Exit 1: Error
   - Not ideal for automation that needs to distinguish “no changes.”

2. `terraform plan -detailed-exitcode`:
   - Exit 0: Success, no changes
   - Exit 1: Error
   - Exit 2: Success, pending changes
   - Use in CI to gate approvals only when changes exist.

3. `terraform plan -refresh-only`:
   - Produces a plan that only refreshes state from real infrastructure without proposing config-driven changes (detects drift). Useful for drift audits.

4. `terraform apply -refresh-only`:
   - Applies a refresh-only plan: updates state file to match reality, never modifies live infra.

**Automation Use Cases:**
- PR pipelines: `plan -detailed-exitcode` to skip noise on no-op changes.
- Nightly drift detection: `plan -refresh-only` + parse JSON for unexpected differences; alert if drift found.
- Controlled drift reconciliation: After review, `apply -refresh-only` to synchronize state safely.

---

## Question 5: Terraform Refresh and State Synchronization

**Answer:**

External changes can create drift (difference between state and real infra). Detection and reconciliation options:

1. Historical `terraform refresh` (legacy): Directly updates state with real resource data without review. Less auditable; avoid in modern workflows.
2. `terraform plan` (default): Refreshes state then compares desired configuration; shows required actions (including drift corrections).
3. `terraform plan -refresh-only`: Shows only drift between state and real infra (ignores desired configuration differences); safe for drift-focused audits.
4. `terraform apply -refresh-only`: Updates state to reflect real infra without altering infrastructure.

**Comparison:**
- Visibility: `plan -refresh-only` offers a diff before altering state; safer than legacy refresh.
- Control: Apply of refresh-only after human approval avoids silent state mutation.
- Full reconciliation: Regular `terraform plan` + `terraform apply` will converge both drift and config changes.

**Recommended Drift Workflow:**
1. Scheduled `terraform plan -refresh-only -out=drift.tfplan`.
2. Parse JSON; if actions appear, open ticket.
3. Human decides: Update config to match reality OR reconcile reality to config.
4. If just metadata drift (e.g., tags added manually) and you accept it, run `terraform apply -refresh-only`.

---

## Question 6: Conditional Resource Creation

**Answer:**

Use `count` for simple optional or repeated resources, `for_each` for collections keyed by stable identifiers. Combine with conditional expressions and dynamic maps.

**Example with `count` (boolean toggle):**
```hcl
variable "enable_backup" { type = bool, default = false }

resource "aws_s3_bucket" "backup" {
  count  = var.enable_backup ? 1 : 0
  bucket = "app-backup-${random_id.suffix.hex}"
  tags = { Purpose = "backup" }
}
```

**Example with `for_each` (conditional membership):**
```hcl
variable "services" {
  type = map(object({
    port     = number
    public   = bool
    replicas = number
  }))
}

# Filter only public services
locals {
  public_services = { for k, v in var.services : k => v if v.public }
}

resource "aws_lb_target_group" "tg" {
  for_each = local.public_services
  name     = "tg-${each.key}"
  port     = each.value.port
  protocol = "HTTP"
}
```

**Conditional with `for_each` using set of strings:**
```hcl
variable "feature_flags" { type = set(string) }

resource "aws_cloudwatch_log_group" "features" {
  for_each = var.feature_flags
  name     = "/app/feature/${each.key}"
  retention_in_days = 30
}
```

**When to choose:**
- `count`: Simple on/off or fixed number; index-based; reordering causes churn.
- `for_each`: Named entities; stable keys prevent destruction during reorder; better for maps/sets.
- Avoid mixing both for the same conceptual collection; pick one pattern for consistency.

---

## Question 7: Terraform State Operations

**Answer:**

State operations manipulate Terraform’s mapping between configuration and real resources. Use them carefully with backups and locking.

1. Moving resources (refactor or rename):
```bash
terraform state mv aws_instance.app module.compute.aws_instance.app
```
Scenario: You introduce a module and shift an existing resource into it without recreation.

2. Renaming addresses (resource name change):
```bash
terraform state mv aws_security_group.old_name aws_security_group.new_name
```
Scenario: Align naming convention; avoid destroy/create cycle.

3. Import existing infrastructure:
```bash
# Add config stub first
resource "aws_s3_bucket" "logs" { bucket = "company-access-logs" }

# Then import
terraform import aws_s3_bucket.logs company-access-logs
```
Scenario: Bring manually created or legacy resources under Terraform management.

4. Remove (stop managing) a resource without deleting cloud object:
```bash
terraform state rm aws_cloudwatch_log_group.legacy
```
Scenario: Hand resource back to manual management or another tool.

5. Inspecting state:
```bash
terraform state list
terraform state show aws_instance.app
```

**Best Practices:**
- Always enable backend locking (e.g., DynamoDB for S3) before state edits.
- Create a backup: copy state file or rely on backend versioning.
- Perform changes in a controlled maintenance window.
- Document state surgery in commit messages or CHANGELOG.

---

## Question 8: Plan Customization and Filtering

**Answer:**

Terraform offers flags to narrow scope or force behaviors. Use sparingly as they can introduce partial convergence.

**`-target`**: Plan/apply limited to specified resource addresses.
```bash
terraform plan -target=module.network
terraform apply -target=aws_vpc.main
```
Use Cases: Bootstrapping prerequisites (VPC before rest), experiments. Risk: Skipping dependent changes may leave graph inconsistent.

**`-replace`** (since Terraform 1.1): Force specific resource to be destroyed and recreated.
```bash
terraform plan -replace=aws_instance.web -out=plan.tfplan
terraform apply plan.tfplan
```
Use Cases: Immutable upgrade, fix drift impossible via in-place update. Risk: Accidental data loss if depends on ephemeral IDs.

**Other Planning Options:**
- `-var` / `-var-file`: Parameterize environments.
- `-parallelism=N`: Control concurrency for rate-limited APIs.
- `-refresh=false`: Skip refresh (rarely recommended; use only for speed in static contexts).
- `-compact-warnings`: Reduce plan noise for CI logs.

**Best Practices:**
- Prefer full graph plans for routine changes.
- Document every use of `-target` or `-replace`.
- After targeted applies, run a full plan to ensure convergence.
- Use modules and clean dependencies rather than frequent targeting.

---

## Question 9: Terraform Plan Output Formats

**Answer:**

Generate machine-readable plan JSON for tooling, policy checks, and diff summaries.

Steps:
```bash
terraform plan -out=plan.tfplan
terraform show -json plan.tfplan > plan.json
```

Sample parsing (actions summary):
```bash
jq -r '.resource_changes[] | "\(.change.actions|join(",")) \(.address)"' plan.json
```
Outputs lines like:
```
create aws_s3_bucket.logs
update aws_security_group.web
delete aws_db_instance.legacy
```

Detect destructive actions:
```bash
jq -e '.resource_changes[] | select(.change.actions[] | contains("delete"))' plan.json >/dev/null && echo "Destruction present" || echo "No destruction"
```

Policy Enforcement (OPA example):
1. Convert plan to JSON.
2. Feed into OPA with Rego rule prohibiting deletes in production unless annotated.

Minimal JSON snippet (structure):
```json
{
  "format_version": "1.2",
  "terraform_version": "1.8.0",
  "resource_changes": [
    {
      "address": "aws_s3_bucket.logs",
      "mode": "managed",
      "type": "aws_s3_bucket",
      "name": "logs",
      "change": {
        "actions": ["create"],
        "before": null,
        "after": { "bucket": "company-access-logs" }
      }
    }
  ]
}
```

Alternative: Use `terraform show -json` directly on state for post-apply audits.

---

## Question 10: GitOps and Terraform Integration

**Answer:**

A GitOps workflow embeds Terraform in a commit-driven promotion pipeline with plan artifacts as approval units.

**High-Level Flow:**
1. Developer pushes feature branch → CI Validation Stage: fmt, init, validate, lint.
2. Plan Stage: `terraform plan -out=plan.tfplan`; generate `plan.json`; summarize diff.
3. PR Decoration: Bot comment with action counts (creates/updates/destroys), any policy violations.
4. Approval: Reviewers approve PR; optional security/policy gates pass.
5. Merge to main triggers Deploy Pipeline:
   - Fetch corresponding `plan.tfplan` (matching commit SHA used in merge).
   - Optional drift check: `terraform plan -refresh-only` (must show no conflicting drift).
   - Apply: `terraform apply plan.tfplan`.
6. Post-Apply:
   - Capture new state version ID; log audit event.
   - Destroy plan artifact (or archive securely for compliance for N days).
7. Promotion (multi-env): Tag or branch (e.g., `release-prod`) triggers same flow in prod folder/workspace with maybe different `*.tfvars` but same commit of code.

**Key Components:**
- Remote state backend with locking (S3 + DynamoDB, or Terraform Cloud) to prevent concurrent applies.
- Provider/version locks committed.
- Plan artifact signing + hash verification.
- Policy checks (OPA, Sentinel) using `plan.json` before approval.
- Observability: Emit metrics (plan size, action counts, apply duration) to monitoring.

**Rollback Strategy:**
- Prefer forward fixes; rollback means reintroducing previous commit and re-planning.
- Keep historical state snapshots (backend versioning) for manual disaster recovery.

**Anti-Patterns to Avoid:**
- Manual console applies bypassing Git.
- Re-generating plan after approval (breaks determinism).
- Applying without verifying provider lock consistency.

---

**Summary:** This answer key covers secure plan artifact workflows, deterministic deployment patterns, plan JSON parsing, conditional resource strategies, targeted planning risks, drift management, and GitOps integration best practices.

**Related:** [Mock Interview #3 Questions](mock_3_questions.md) | [Mock #2 Answers](mock_2_answers.md) | [Advanced Level](../advanced/) | [Beginner Level](../beginner/)
