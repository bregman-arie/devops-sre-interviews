# Terraform Mock Interview #6 - Intermediate Level (Lifecycle & Protection Scenarios)

## Question 1: Protecting a Critical RDS Instance
**Scenario Question:** A teammate proposes adding `lifecycle { prevent_destroy = true }` to an RDS instance after a near miss where a plan showed a replacement. Walk through how you evaluate whether to add it, implement it safely, and handle legitimate replacement needs later.
[📖 Answer](mock_6_answers.md#question-1-protecting-a-critical-rds-instance)

---

## Question 2: When `prevent_destroy` Causes Stalled Refactors
**Scenario Question:** You need to migrate an S3 bucket to a new naming standard but it is guarded by `prevent_destroy`. How do you proceed without breaking data access while respecting the guard?
[📖 Answer](mock_6_answers.md#question-2-when-prevent_destroy-causes-stalled-refactors)

---

## Question 3: Choosing Between `ignore_changes` and Data Model Fixes
**Scenario Question:** A security group keeps showing spurious diffs because AWS normalizes rule ordering. Should you use `ignore_changes` or restructure configuration? Detail decision criteria.
[📖 Answer](mock_6_answers.md#question-3-choosing-between-ignore_changes-and-data-model-fixes)

---

## Question 4: Using `create_before_destroy` for Zero-Downtime
**Scenario Question:** You must rotate an auto-scaling group launch template AMI with zero downtime. Explain how `create_before_destroy` works and risks if dependencies (e.g., unique names) aren't adjusted.
[📖 Answer](mock_6_answers.md#question-4-using-create_before_destroy-for-zero-downtime)

---

## Question 5: Auditing Lifecycle Usage
**Scenario Question:** Your repo has many lifecycle blocks. Design a lightweight audit process to ensure they are justified and not masking drift.
[📖 Answer](mock_6_answers.md#question-5-auditing-lifecycle-usage)

---

## Question 6: Removing `prevent_destroy` Safely
**Scenario Question:** You need to legitimately replace a protected VPC. Outline a safe procedure to temporarily remove the guard, migrate, and re-establish protection.
[📖 Answer](mock_6_answers.md#question-6-removing-prevent_destroy-safely)

---

## Question 7: Policy Enforcement for Critical Resources
**Scenario Question:** How would you enforce (Sentinel/OPA) that certain resources must always have `prevent_destroy` applied unless an approved override label is present?
[📖 Answer](mock_6_answers.md#question-7-policy-enforcement-for-critical-resources)

---

## Question 8: Anti-Patterns in Lifecycle Blocks
**Scenario Question:** List common anti-patterns with lifecycle usage (`prevent_destroy`, `ignore_changes`, `create_before_destroy`) and remediation steps.
[📖 Answer](mock_6_answers.md#question-8-anti-patterns-in-lifecycle-blocks)

---

**Next:** [View Answers](mock_6_answers.md)

**Time Limit:** 60 minutes  
**Level:** Intermediate  
**Focus Areas:** Lifecycle safeguards, controlled replacements, guard removal, policy enforcement, anti-pattern remediation

**Related:** [Mock #1](mock_1_questions.md) | [Mock #5](mock_5_questions.md) | [Advanced Level](../advanced/) | [Beginner Level](../beginner/) | [Expert Level](../expert/)
