# Terraform Mock Interview #3 - Advanced Level - Answer Key (Sensitive Variables & Secrets Management)

## Question 1: Core Approaches to Managing Sensitive Variables
**Answer:**
Strategies:
1. Sensitive Variables in Code:
```hcl
variable "db_password" {
  type      = string
  sensitive = true
}
```
Pros: Masks in CLI output. Cons: Still stored in state if used in resources.
2. Environment Variables (`TF_VAR_db_password`): Inject at runtime; keep out of code. Risk: Shell history/verbosity can leak.
3. Ignored `.tfvars` Files (`secrets.auto.tfvars` in .gitignore): Central override; risk of accidental commit; local-only.
4. External Secret Managers (Vault, AWS Secrets Manager, Azure Key Vault, GCP Secret Manager): Source-of-truth with rotation, audit, dynamic creds.
5. Terraform Cloud/Enterprise Variables (sensitive var storage): Managed encryption; limited to that platform.
6. Indirection via data sources (e.g., retrieving ARNs or references rather than actual secret values).

Comparison Table (Summary):
- Masking: sensitive=true (partial), env vars (depends on CI), external managers (strong control).
- Rotation: manual (vars/files), automatic (external managers).
- Audit: minimal (env/files), rich (Vault/secret managers).

---

## Question 2: Preventing Secret Exposure in Plans and Logs
**Answer:**
Terraform masks sensitive variable values in human-readable plan/apply output. Leaks still possible via:
- Debug logs (`TF_LOG=DEBUG`).
- Third-party provisioner scripts printing values.
- `terraform show -json` may include structure that reveals lengths or derived tags.
- Storing secrets as resource attributes (password fields) → appear (masked) but remain in state.

Mitigations:
- Run with minimal log level; disallow debug in CI.
- Use `sensitive = true` on variables and outputs.
- Avoid echoing secrets in `local-exec` or templates; pass via files with restricted permissions.
- Use dynamic credentials (expire quickly).
- Restrict access to state backend (IAM policies, encryption, RBAC).
- Scan artifacts for high-entropy strings before publishing.

---

## Question 3: Integrating with HashiCorp Vault
**Answer:**
Pattern using Vault provider and dynamic DB creds:
```hcl
provider "vault" {
  address = var.vault_addr
  token   = var.vault_token  # Or use AppRole/JWT auth
}

data "vault_database_creds" "pg" {
  name = "postgres-readwrite"  # Dynamic role
}

resource "aws_ssm_parameter" "db_user" {
  name        = "/app/db/user"
  type        = "String"
  value       = data.vault_database_creds.pg.username
  overwrite   = true
}
resource "aws_ssm_parameter" "db_pass" {
  name        = "/app/db/password"
  type        = "SecureString"
  value       = data.vault_database_creds.pg.password
  overwrite   = true
}
```
State Impact: Dynamic creds rotate; storing in SSM shifts secret persistence away from Terraform state (but parameters still appear as values in plan). Mitigation: Use separate stack or avoid storing password if not required (let application fetch directly from Vault instead of writing to SSM via Terraform).

---

## Question 4: AWS Secrets Manager and Rotation
**Answer:**
Reference secret value:
```hcl
data "aws_secretsmanager_secret_version" "db" {
  secret_id = "prod/database"
}

resource "aws_db_instance" "main" {
  username = var.db_user
  password = data.aws_secretsmanager_secret_version.db.secret_string
  # Avoid forcing recreation by using lifecycle ignore_changes
  lifecycle { ignore_changes = [password] }
}
```
Rotation Strategy:
- DB user password rotates externally; Terraform shouldn't recreate DB.
- Application deployment pipeline retrieves updated secret at runtime (not via Terraform).
Best Practices:
- Use `ignore_changes` for non-critical runtime passwords.
- Use separate module to manage secret metadata, not consumers.
- Avoid embedding secret in immutable resources; prefer ephemeral fetch.

---

## Question 5: Azure Key Vault / GCP Secret Manager Parity
**Answer:**
Azure:
```hcl
data "azurerm_key_vault_secret" "db" {
  name         = "db-password"
  key_vault_id = azurerm_key_vault.main.id
}
```
GCP:
```hcl
data "google_secret_manager_secret_version" "db" {
  secret  = "db-password"
  project = var.project
}
```
Common Abstraction:
- Create a local map aligning fields: `{ value = data.*.secret_string or data.*.value, rotation = ... }`
- Wrap provider-specific logic in modules exporting generic `secret.value`.

---

## Question 6: Minimizing Secret Footprint in State
**Answer:**
Techniques:
- Avoid resources that require storing the secret (let apps pull at runtime).
- Use data sources referencing secret managers instead of variables containing literal secrets.
- Use `lifecycle.ignore_changes` to prevent drift-driven re-plans.
- Provide secret IDs/ARNs rather than values to Terraform-managed resources when possible (e.g., for services that reference secret by ARN).
- Segregate secret-handling into separate workspace with stricter access.
- Employ dynamic credentials (short TTL) so stale state is less valuable.

Example referencing secret by ARN:
```hcl
data "aws_secretsmanager_secret" "db" { name = "prod/database" }
resource "aws_lambda_function" "app" {
  environment { variables = { DB_SECRET_ARN = data.aws_secretsmanager_secret.db.arn } }
}
```
No secret value stored.

---

## Question 7: Policy as Code for Secret Usage
**Answer:**
OPA Rego Example (deny hardcoded high-entropy strings):
```rego
package terraform.secrets

entropy(str) = e { # pseudo entropy function }

violation[msg] {
  input.resource_changes[_].change.after.password = p
  entropy(p) > 4.0
  msg = sprintf("Hardcoded password detected: %s", [p])
}
```
Sentinel (Terraform Cloud) Example Concept:
- Access `tfplan/v2` data; ensure no resource attribute matches regex for hardcoded secret pattern.
- Enforce presence of approved data source addresses for secret retrieval.

Static Analysis: Pre-commit hook scanning `*.tf` and `*.tfvars` for secret regex patterns.

---

## Question 8: Secure Variable Injection in CI/CD
**Answer:**
Pipeline Stage:
1. Fetch ephemeral token from Vault via GitHub Actions OIDC or CI workload identity.
2. Export secrets as masked environment variables (CI secret masking).
3. Run Terraform with no printing of vars: `terraform plan` uses variables implicitly from env (`TF_VAR_db_password`).
4. For plan artifact: ensure redaction script scans plan JSON.
5. Avoid saving secrets in workspace artifacts; only keep plan binary with masked output.
6. Revoke token after pipeline completion.

Key Elements:
- Ephemeral auth method (OIDC, AppRole with short TTL).
- Masking in logs.
- Least privilege Vault policies.
- No persistent `.tfvars` checked in.

---

## Question 9: Secret Rotation Impact Analysis
**Answer:**
Goal: Rotate secret without triggering resource recreation (e.g., RDS instance).

Approach:
- Use external rotation (Secrets Manager / Vault).
- Consumers refer to secret reference (ARN, path) not value.
- Application runtime fetches current secret (SDK call) on startup or periodically.
- Terraform config ignores password changes on underlying resource using `ignore_changes`.

If rotation requires config update (e.g., injecting to ECS task definition environment):
- Update only tasks/services; avoid modifying base infra.
- Use feature flag redeployment or rolling update triggered by external pipeline, not Terraform apply of password literal.

---

## Question 10: Auditing and Compliance of Secrets Use
**Answer:**
Audit Matrix (Sample):
- Event: Secret read (Vault/Secrets Manager) → Log: actor, time, secret path/version.
- Event: Terraform plan/apply → Log: commit SHA, user/service principal, state version ID.
- Event: Secret rotation → Log: old version, new version, initiator.
- Event: Policy violation attempt → Log: rule ID, resource address, decision.
- Access Control Changes → Log IAM diff (who changed secret access policy).

Store logs centrally (e.g., CloudWatch + SIEM). Retain per compliance (e.g., 1 year). Enable alerts on anomalous read volume.

---

## Question 11: Encryption and Backend Selection
**Answer:**
Backends:
- Local: No server-side encryption; rely on disk protections; high leak risk.
- S3 + DynamoDB + KMS: Server-side encryption (SSE-KMS), locking with DynamoDB; versioning for rollback; strong IAM integration.
- Terraform Cloud/Enterprise: Encrypted at rest + RBAC + workspace variable management; less operational overhead.
- Consul: Requires securing cluster (TLS, ACLs); encryption optional; more ops complexity.

Selection Criteria: Encryption at rest, access audit, locking, versioning, operational burden, compliance features.

---

## Question 12: Redaction and Sanitization Strategies
**Answer:**
Incident Response Checklist:
1. Identify Exposure: Which artifact/log contains secret? Determine scope (CI logs, plan JSON, state snapshot).
2. Revoke Secret: Rotate immediately in secret manager / revoke dynamic creds.
3. Purge Artifacts: Delete affected logs or restrict access; create sanitized replacements.
4. Backfill: Re-run plan/apply after rotation ensuring new secret not printed.
5. Retrospective: Add automated scanners to pipeline (entropy detection, regex scanning).
6. Access Review: Enumerate who accessed exposed artifacts; escalate if needed.
7. Documentation: Record timeline, actions, new safeguards.

Redaction Tools: Log rewriting utilities, SIEM purge, Git history rewrite (filter-branch or BFG Repo-Cleaner) for committed `.tfvars` exposures.

---

**Summary:** This answer key covers secure sourcing, masking limits, dynamic credentials, rotation-safe patterns, multi-cloud abstraction, minimizing state secret footprint, policy enforcement, CI injection, audit logging, backend comparison, and incident redaction strategies.

**Related:** [Questions](mock_3_questions.md) | [Advanced Mock #2](mock_2_answers.md) | [Expert Level](../expert/) | [Intermediate Level](../intermediate/) | [Beginner Level](../beginner/)
