# Terraform Mock Interview #3 - Advanced Level (Sensitive Variables & Secrets Management)

## Question 1: Core Approaches to Managing Sensitive Variables
**Question:** Enumerate the main strategies to manage sensitive variables (passwords, API keys) in Terraform. Compare using `sensitive = true` on variables, environment variables (`TF_VAR_*`), `.tfvars` files excluded from VCS, and external secret managers.
[📖 Answer](mock_3_answers.md#question-1-core-approaches-to-managing-sensitive-variables)

---

## Question 2: Preventing Secret Exposure in Plans and Logs
**Question:** How does Terraform attempt to mask sensitive values, and in what scenarios can secrets still leak? Detail mitigations for CI/CD pipelines.
[📖 Answer](mock_3_answers.md#question-2-preventing-secret-exposure-in-plans-and-logs)

---

## Question 3: Integrating with HashiCorp Vault
**Question:** Show a pattern for retrieving dynamic database credentials from Vault for use in Terraform-managed infrastructure without writing the password to state permanently.
[📖 Answer](mock_3_answers.md#question-3-integrating-with-hashicorp-vault)

---

## Question 4: AWS Secrets Manager and Rotation
**Question:** Demonstrate how to reference an AWS Secrets Manager secret in Terraform and update dependent resources when the secret rotates. What are best practices to avoid unnecessary resource recreation?
[📖 Answer](mock_3_answers.md#question-4-aws-secrets-manager-and-rotation)

---

## Question 5: Azure Key Vault / GCP Secret Manager Parity
**Question:** Compare patterns for retrieving secrets from Azure Key Vault and Google Secret Manager. What common abstractions can you build to keep multi-cloud code consistent?
[📖 Answer](mock_3_answers.md#question-5-azure-key-vault--gcp-secret-manager-parity)

---

## Question 6: Minimizing Secret Footprint in State
**Question:** What techniques reduce or eliminate secret values stored in the Terraform state file? Include resource choices, data sources, and indirection approaches.
[📖 Answer](mock_3_answers.md#question-6-minimizing-secret-footprint-in-state)

---

## Question 7: Policy as Code for Secret Usage
**Question:** How would you enforce that no hardcoded secrets appear in Terraform code using Sentinel/OPA or static analysis? Provide a high-level policy example.
[📖 Answer](mock_3_answers.md#question-7-policy-as-code-for-secret-usage)

---

## Question 8: Secure Variable Injection in CI/CD
**Question:** Design a pipeline stage to inject secrets into Terraform runs securely. Cover vault authentication, ephemeral tokens, masking, and artifact handling.
[📖 Answer](mock_3_answers.md#question-8-secure-variable-injection-in-cicd)

---

## Question 9: Secret Rotation Impact Analysis
**Question:** A DB password rotates monthly. Explain how to prevent cascading redeployments of dependent services while ensuring they receive the updated credential.
[📖 Answer](mock_3_answers.md#question-9-secret-rotation-impact-analysis)

---

## Question 10: Auditing and Compliance of Secrets Use
**Question:** What data and events should be logged/audited around Terraform secret handling for compliance (SOC2, HIPAA)? Provide a minimal audit matrix.
[📖 Answer](mock_3_answers.md#question-10-auditing-and-compliance-of-secrets-use)

---

## Question 11: Encryption and Backend Selection
**Question:** Compare security implications of different backends (local, S3+DynamoDB+KMS, Terraform Cloud, Consul) for storing state that may include secrets.
[📖 Answer](mock_3_answers.md#question-11-encryption-and-backend-selection)

---

## Question 12: Redaction and Sanitization Strategies
**Question:** How can you redact secrets from historical logs, plan artifacts, or debug outputs after accidental exposure? Outline an incident response checklist.
[📖 Answer](mock_3_answers.md#question-12-redaction-and-sanitization-strategies)

---

**Next:** [View Answers](mock_3_answers.md)

**Time Limit:** 80 minutes  
**Level:** Advanced  
**Focus Areas:** Secret sourcing, masking, rotation, compliance, backend security, CI/CD practices

**Related:** [Mock Interview #1](mock_1_questions.md) | [Mock Interview #2](mock_2_questions.md) | [Expert Level](../expert/) | [Intermediate Level](../intermediate/) | [Beginner Level](../beginner/)
