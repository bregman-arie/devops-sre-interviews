# Terraform Mock Interview #3 - Intermediate Level

## Question 1: Terraform Plan Files and CI/CD Workflows

**Question:** In a CI/CD pipeline, you want to separate the planning phase from the apply phase for security and approval workflows. How would you save a Terraform plan to a file and apply it later? What are the benefits and potential risks of this approach?

[📖 Answer](mock_3_answers.md#question-1-terraform-plan-files-and-cicd-workflows)

---

## Question 2: Deterministic Deployments and Plan Analysis

**Question:** Your team needs to ensure that what gets reviewed in a pull request is exactly what gets deployed. How can Terraform plan files help achieve deterministic deployments? How would you integrate plan file generation and analysis into a GitOps workflow?

[📖 Answer](mock_3_answers.md#question-2-deterministic-deployments-and-plan-analysis)

---

## Question 3: Plan File Security and Storage

**Question:** Terraform plan files can contain sensitive information. What security considerations should you keep in mind when working with plan files in CI/CD pipelines? How would you handle plan file storage and cleanup?

[📖 Answer](mock_3_answers.md#question-3-plan-file-security-and-storage)

---

## Question 4: Advanced Planning Strategies

**Question:** Explain the difference between `terraform plan`, `terraform plan -detailed-exitcode`, and `terraform plan -refresh-only`. When would you use each in automation scenarios?

[📖 Answer](mock_3_answers.md#question-4-advanced-planning-strategies)

---

## Question 5: Terraform Refresh and State Synchronization

**Question:** Your infrastructure was modified outside of Terraform. How would you detect these changes and synchronize your state? Compare the approaches of `terraform refresh`, `terraform plan -refresh-only`, and `terraform apply -refresh-only`.

[📖 Answer](mock_3_answers.md#question-5-terraform-refresh-and-state-synchronization)

---

## Question 6: Conditional Resource Creation

**Question:** You need to create resources conditionally based on input variables or environment. Demonstrate how to use `count` and `for_each` with conditional expressions, and explain when each approach is more appropriate.

[📖 Answer](mock_3_answers.md#question-6-conditional-resource-creation)

---

## Question 7: Terraform State Operations

**Question:** Walk through common Terraform state operations: moving resources between states, importing existing infrastructure, and removing resources from state management. Provide examples of when each operation is needed.

[📖 Answer](mock_3_answers.md#question-7-terraform-state-operations)

---

## Question 8: Plan Customization and Filtering

**Question:** In large infrastructures, Terraform plans can be overwhelming. How can you use `-target`, `-replace`, and other planning options to focus on specific resources? What are the risks and best practices?

[📖 Answer](mock_3_answers.md#question-8-plan-customization-and-filtering)

---

## Question 9: Terraform Plan Output Formats

**Question:** Terraform can output plans in different formats. How would you generate machine-readable plan output for integration with other tools? Demonstrate parsing plan output for automated analysis.

[📖 Answer](mock_3_answers.md#question-9-terraform-plan-output-formats)

---

## Question 10: GitOps and Terraform Integration

**Question:** Design a GitOps workflow that uses Terraform plan files for infrastructure changes. How would you handle plan generation, review, approval, and application in a Git-based workflow?

[📖 Answer](mock_3_answers.md#question-10-gitops-and-terraform-integration)

---

**Next:** [View Answers](mock_3_answers.md)

**Time Limit:** 75 minutes  
**Level:** Intermediate  
**Focus Areas:** Plan file management, CI/CD integration, deterministic deployments, advanced planning strategies, and GitOps workflows

**Related:** [Mock Interview #1](mock_1_questions.md) | [Mock Interview #2](mock_2_questions.md) | [Beginner Level](../beginner/) | [Advanced Level](../advanced/)
