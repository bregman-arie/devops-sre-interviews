# Terraform Mock Interview #6 - Intermediate Level - Answer Key (Lifecycle & Protection Scenarios)

## Question 1: Protecting a Critical RDS Instance
**Answer:**
Evaluation Steps:
1. Confirm why replacement appeared (password change, storage type, engine version, deletion protection toggle, identifier change). Use plan JSON diff.
2. Determine if attribute is ForceNew: consult provider docs; some changes require recreation.
3. Assess business impact (data loss risk, downtime tolerance, snapshot availability).
4. Prefer native AWS protections first (`deletion_protection = true`, automated backups, snapshots) before Terraform lifecycle.

Implementation:
```hcl
resource "aws_db_instance" "primary" {
  identifier         = var.db_identifier
  engine             = "postgres"
  allocated_storage  = 100
  deletion_protection = true
  # ... other attrs
  lifecycle { prevent_destroy = true }
}
```
Safely Handling Future Legitimate Replacement:
- Plan blue/green migration: create new instance with replica or snapshot restore.
- Cut over (promote replica / update DNS / rotate connection string).
- Temporarily remove `prevent_destroy` only after data validated in new instance.
- Destroy old instance after confirmed decommission.
Documentation & Policy:
- Record justification and removal procedure in README or ADR.

---
## Question 2: When `prevent_destroy` Causes Stalled Refactors
**Answer:**
Goal: Rename S3 bucket (requires new bucket).
Strategy:
1. Leave protected bucket; create new bucket with target name.
2. Sync data (S3 batch replication / aws cli sync).
3. Update references (CloudFront origins, apps) to new bucket.
4. Validate integrity; cut traffic; freeze writes on old bucket.
5. Remove `prevent_destroy` guard; apply destruction only when safe.
6. Optionally keep old bucket with read-only policy for rollback window.
Alternative: Avoid rename—tags + aliases (CloudFront domain) may remove need for destructive rename.

---
## Question 3: Choosing Between `ignore_changes` and Data Model Fixes
**Answer:**
Spurious Security Group Diff Cause: AWS reorders rules; Terraform sees order diff.
Decision Criteria:
- If diff is purely cosmetic (ordering) and does not mask semantic changes → `ignore_changes` acceptable.
- If diff hides real drift potential (e.g., ports modified manually) → restructure resource (split rules) rather than ignore changes.
Better Pattern:
```hcl
resource "aws_security_group_rule" "ingress" {
  for_each = var.rules
  type              = "ingress"
  from_port         = each.value.from
  to_port           = each.value.to
  protocol          = each.value.protocol
  security_group_id = aws_security_group.main.id
  cidr_blocks       = each.value.cidr
}
```
Avoid single SG block with many nested rules if ordering noise occurs.
Small Acceptable Ignore Example:
```hcl
resource "aws_security_group" "main" {
  name = var.name
  lifecycle { ignore_changes = [ingress, egress] }
}
```
But pair with per-rule resources for explicit control.

---
## Question 4: Using `create_before_destroy` for Zero-Downtime
**Answer:**
Purpose: Ensure replacement new resource is ready before destroying old.
Mechanics: Terraform creates new resource with different name (if required) or temporary identifier; updates dependencies then destroys old.
Example (Launch Template Option):
```hcl
resource "aws_launch_template" "app" {
  name_prefix = "app-lt-"  # allows new template coexistence
  image_id    = var.new_ami
  lifecycle { create_before_destroy = true }
}
```
Risks:
- Identifier Collision: If `name` fixed, creation fails.
- Capacity Spikes: ASG may temporarily scale using both old/new until refresh complete.
- Cost: Short-lived duplicate resources.
Mitigation: Use prefixes; coordinate scaling steps; define max surge limits via ASG rolling update.

---
## Question 5: Auditing Lifecycle Usage
**Answer:**
Process:
1. Script parse for `lifecycle` blocks (`grep -R "lifecycle"`).
2. Categorize usage (prevent_destroy, ignore_changes, create_before_destroy).
3. Validate justification file (e.g., `lifecycle_justification.md`).
4. Policy check: OPA rule requiring annotation comment above each lifecycle:
```hcl
# lifecycle: reason=protect-rds-production
lifecycle { prevent_destroy = true }
```
5. Quarterly review: list of guards vs recent plan diffs; remove obsolete ones.
Metrics: count of lifecycle blocks; average age; unreferenced justifications flagged.

---
## Question 6: Removing `prevent_destroy` Safely
**Answer:**
Procedure:
1. Snapshot / backup (DB snapshot, AMI, etc.).
2. Ensure new replacement resource exists and validated.
3. Remove lifecycle block in code; commit with justification.
4. Generate plan and have peer review focusing only on intended destroy.
5. Apply during maintenance window.
6. Post-apply verification + rollback path (snapshot) retained until confidence.
7. Optionally re-add guard if resource remains critical.

---
## Question 7: Policy Enforcement for Critical Resources
**Answer:**
OPA Example (pseudo Rego):
```rego
package terraform.prevent_destroy

critical_resources = {"aws_db_instance.primary", "aws_vpc.main"}

violation[msg] {
  r := input.resource_changes[_]
  r.address in critical_resources
  not has_prevent_destroy(r)
  msg = sprintf("Resource %s missing prevent_destroy", [r.address])
}
```
Sentinel: Use `tfplan/v2` to inspect lifecycle settings; deny if absent for tag `critical=true`.
Override: Allow annotation `allow_destroy=true` in commit message or PR label; pipeline passes variable to policy enabling exception.

---
## Question 8: Anti-Patterns in Lifecycle Blocks
**Answer:**
Anti-Patterns & Fixes:
1. Blanket `prevent_destroy` across entire module → blocks legitimate refactors; fix: apply only to true stateful core resources.
2. `ignore_changes` masking configuration drift (e.g., instance_type changes outside Terraform) → fix: manage attribute explicitly or alert on drift.
3. Using `create_before_destroy` where resource uniqueness is enforced (e.g., S3 bucket globally unique) → fix: adopt blue/green naming or manual migration.
4. Guard used instead of proper backup strategy → fix: implement snapshots/backups; treat guard as supplemental.
5. Lifecycle blocks without documented reason → fix: enforce comment/policy requiring rationale.
6. Stale ignore list after provider improvement (no longer noisy) → fix: periodic audit remove obsolete ignores.

---
**Summary:** Lifecycle constructs provide safety and flexibility when applied surgically with clear justification; misuse can hide drift and impede evolution. Policies, audits, and migration patterns maintain balance between protection and agility.

**Related:** [Questions](mock_6_questions.md) | [Mock #5](mock_5_answers.md) | [Advanced Level](../advanced/) | [Beginner Level](../beginner/) | [Expert Level](../expert/)
