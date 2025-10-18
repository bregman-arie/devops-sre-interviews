# Terraform Mock Interview #5 - Intermediate Level - Answer Key (Safety & Triage Scenarios)

## Question 1: Preventing Accidental Destruction (VPC Recreation Scenario)
**Answer:**
Triage Steps (ordered):
1. Halt: Do NOT apply. Ensure no auto-apply pipeline will trigger. If in CI with approval step, block immediately.
2. Inspect Plan JSON: `terraform plan -out=plan.tfplan && terraform show -json plan.tfplan > plan.json` then summarize VPC-related actions:
```bash
jq -r '.resource_changes[] | select(.address | contains("aws_vpc")) | .change.actions' plan.json
```
3. Identify Trigger: Compare last committed config vs current diff (git diff). Common causes:
   - CIDR block change (immutable → force replace)
   - Enabling/Disabling IPv6 or DNS options
   - Removing/renaming VPC-level tags required by other resources
   - Provider upgrade altering default attribute forcing recreation
4. Check State vs Config: `terraform state show aws_vpc.main` and confirm attributes changed.
5. Validate External Dependencies: List attached resources (subnets, gateways, NAT, route tables). Recreating VPC cascades deletion of all child resources if they’re in graph.
6. Decide Strategy:
   - If change is non-essential (e.g., tag tweak causing replacement due to bug): Revert the config change.
   - If change is essential (e.g., IPv6 enablement): Plan maintenance window; design migration (create new VPC side-by-side, migrate workloads gradually).
7. Implement Safe Fix:
   - Revert the destructive attribute and re-run plan; expect no destroy.
   - OR isolate VPC migration: Introduce new `aws_vpc` resource with different name (no replacement) and migrate dependent resources incrementally using `terraform state mv`.
8. Add Lifecycle Guard:
```hcl
resource "aws_vpc" "main" {
  cidr_block = var.vpc_cidr
  lifecycle { prevent_destroy = true }
}
```
   - Commit guard; re-run plan; any future destroy must be explicit removal of guard.
9. Document Root Cause: Create incident log; add tests or policy (Sentinel/OPA) to block forced VPC replacements.

Safe Migration Pattern (Blue/Green VPC):
1. Create new VPC (aws_vpc.new).
2. Peer old/new; replicate subnets, route tables, security groups.
3. Migrate workloads (ASG target group updates, RDS snapshot restore).
4. Decommission old VPC after traffic cutover.

Do NOT: Use `-target` to only apply unrelated changes while leaving destructive plan unresolved; this defers risk.

---

## Question 2: Handling Unexpected Mass Deletes
**Answer:**
1. Abort apply. 2. Inspect git diff for removed blocks or changed `for_each` keys. 3. Look for variable value changes altering collections (map key removal → destroy). 4. Compare provider versions; run `terraform providers lock`. 5. Validate state health: `terraform state list`. 6. If due to key renames, reintroduce old keys temporarily and use `terraform state mv` to map to new keys before removing old ones. 7. Add guard rails (`prevent_destroy`) on foundational resources. 8. Re-plan ensuring only expected changes.

---

## Question 3: Mitigating Provider Schema Changes
**Answer:**
1. Read changelog. 2. Use `terraform plan -refresh=false` to see purely config-driven differences vs provider refresh changes. 3. Generate plan with previous provider version (checkout prior lock) to isolate new forced replacements. 4. If replacements unnecessary, pin previous minor version; schedule compatibility refactor. 5. Consider `ignore_changes` for attributes newly enforced but not critical. 6. Incrementally upgrade via staging environment first.

---

## Question 4: Safeguards Against Production Drifts
**Answer:**
Preventive: Code review, policy checks (deny destroy of core resources), lifecycle `prevent_destroy`, version pinning, mandatory plan approvals.
Detective: Scheduled `plan -detailed-exitcode`, drift audit with `plan -refresh-only`, cloud config diff scanning, tagging conformity scans.
Corrective: Automated rollback (reapply last known good commit), state repair procedure, blue/green replacement with controlled migration.

---

## Question 5: Emergency Abort Procedures
**Answer:**
If apply started:
- Ctrl+C (may abort; if mid-destroy, partial state). Immediately run `terraform plan` to assess.
- Enable backend lock (if not locked) to prevent concurrent runs.
- Temporarily revoke IAM permissions for destructive actions (detach delete privileges from CI role).
- Snapshot state file (copy in backend). If partial deletion occurred, manually import surviving resources before next apply.
- Communicate outage risk; freeze deployments.

---

## Question 6: Using Lifecycle Blocks for Safety
**Answer:**
Examples:
```hcl
resource "aws_vpc" "main" {
  cidr_block = var.vpc_cidr
  lifecycle { prevent_destroy = true }
}

resource "aws_db_instance" "primary" {
  # Avoid churn on minor password rotations
  lifecycle { ignore_changes = [password] }
}

resource "aws_launch_template" "app" {
  lifecycle { create_before_destroy = true }  # Zero-downtime replacement
}
```
Misuse: Overusing `ignore_changes` hides drift; `prevent_destroy` may block legitimate teardown; `create_before_destroy` on non-replaceable resources causes errors.

---

## Question 7: Partial Applies and Consistency
**Answer:**
Risks: Skipped dependencies lead to config divergence; future full apply triggers large surprise changes; state may represent partial graph convergence. Mitigation: Document rationale; run full plan after targeted apply; remove reliance on frequent targeting by refactoring modules.

---

## Question 8: State Surgery Recovery
**Answer:**
Steps:
1. Backup current broken state. 2. Identify lost resource (e.g., `aws_vpc.main`). 3. Reconstruct import arguments: `terraform state show` from staging or cloud console values. 4. Re-add resource via `terraform import aws_vpc.main vpc-12345678`. 5. Validate dependent resource references still correct. 6. Commit procedure notes.

---

## Question 9: Detecting Root Cause of Force Replacement
**Answer:**
1. Inspect plan diff for `-/+` on resource. 2. Compare before/after attributes (plan JSON). 3. Check if attribute is ForceNew per provider docs. 4. Check whether variable/local changed value or a computed default changed due to provider upgrade. 5. Validate no hidden whitespace/tag normalization. 6. Decide: revert change, accept replacement with maintenance window, or mitigate via alternative attribute.

---

## Question 10: Pre-Apply Change Approval Workflow
**Answer:**
Flow:
1. CI generates plan + JSON. 2. Parser tallies counts of create/update/destroy/replace. 3. Policy engine thresholds: e.g., >5 destroys OR any root-level VPC destroy → require senior approval label. 4. Slack/Email summary with commit, counts, sensitive resources list. 5. Approver signs off; artifact hash recorded. 6. Deployment job verifies hash match then applies. 7. Post-apply metrics emitted; artifact retired.

Escalation: If threshold exceeded (like VPC replacement), require runbook steps documented and scheduled window.

---

**Summary:** These scenarios emphasize cautious triage, structured analysis of plan diffs, lifecycle protections, controlled migration patterns, and policy-driven safeguards to prevent accidental large-scale destruction.

**Related:** [Questions](mock_5_questions.md) | [Mock #4](mock_4_answers.md) | [Advanced Level](../advanced/) | [Expert Level](../expert/) | [Beginner Level](../beginner/)
