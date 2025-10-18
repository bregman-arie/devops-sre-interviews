# Terraform - Beginner Interview Mock #2

> **Difficulty:** Beginner  
> **Duration:** ~35 minutes  
> **Goal:** Test understanding of Terraform basics, resource management, and common workflows.

---

## 🧠 Section 1: Core Concepts

1. What is **Infrastructure as Code (IaC)** and what problems does Terraform solve? [📖 Answer](mock_2_answers.md#1-what-is-infrastructure-as-code-iac-and-what-problems-does-terraform-solve)
2. Explain the difference between **declarative** and **imperative** approaches. Which one does Terraform use? [📖 Answer](mock_2_answers.md#2-explain-the-difference-between-declarative-and-imperative-approaches-which-one-does-terraform-use)
3. What are the main components of a Terraform configuration file? [📖 Answer](mock_2_answers.md#3-what-are-the-main-components-of-a-terraform-configuration-file)
4. What is a **Terraform provider** and can you name a few examples? [📖 Answer](mock_2_answers.md#4-what-is-a-terraform-provider-and-can-you-name-a-few-examples)

---

## 🔧 Section 2: Terraform Workflow

5. Walk through the basic Terraform workflow commands. [📖 Answer](mock_2_answers.md#5-walk-through-the-basic-terraform-workflow-commands)
6. What does `terraform plan` do and why is it important? [📖 Answer](mock_2_answers.md#6-what-does-terraform-plan-do-and-why-is-it-important)
7. What's the difference between `terraform apply` and `terraform apply -auto-approve`? [📖 Answer](mock_2_answers.md#7-whats-the-difference-between-terraform-apply-and-terraform-apply--auto-approve)
8. How do you destroy resources created by Terraform? [📖 Answer](mock_2_answers.md#8-how-do-you-destroy-resources-created-by-terraform)

---

## 📁 Section 3: State Management

9. What is **Terraform state** and why is it important? [📖 Answer](mock_2_answers.md#9-what-is-terraform-state-and-why-is-it-important)
10. Where is the Terraform state stored by default, and what are the alternatives? [📖 Answer](mock_2_answers.md#10-where-is-the-terraform-state-stored-by-default-and-what-are-the-alternatives)
11. What happens if you accidentally delete the state file? [📖 Answer](mock_2_answers.md#11-what-happens-if-you-accidentally-delete-the-state-file)
12. What is **state locking** and why do you need it? [📖 Answer](mock_2_answers.md#12-what-is-state-locking-and-why-do-you-need-it)
13. What's the purpose of `.terraform.lock.hcl` file and should it be committed to version control? [📖 Answer](mock_2_answers.md#13-whats-the-purpose-of-terraformlockhcl-file-and-should-it-be-committed-to-version-control)

---

## 🔄 Section 4: Variables and Outputs

14. How do you define and use **variables** in Terraform? [📖 Answer](mock_2_answers.md#14-how-do-you-define-and-use-variables-in-terraform)
15. What are the different ways to assign values to Terraform variables? [📖 Answer](mock_2_answers.md#15-what-are-the-different-ways-to-assign-values-to-terraform-variables)
16. What are **outputs** in Terraform and when would you use them? [📖 Answer](mock_2_answers.md#16-what-are-outputs-in-terraform-and-when-would-you-use-them)

---

## 🏗️ Section 5: Practical Scenarios

17. You need to create an AWS S3 bucket. Write a basic Terraform configuration. [📖 Answer](mock_2_answers.md#17-you-need-to-create-an-aws-s3-bucket-write-a-basic-terraform-configuration)
18. How would you create multiple similar resources (e.g., 3 EC2 instances)? [📖 Answer](mock_2_answers.md#18-how-would-you-create-multiple-similar-resources-eg-3-ec2-instances)
19. Your `terraform apply` failed. What steps would you take to troubleshoot? [📖 Answer](mock_2_answers.md#19-your-terraform-apply-failed-what-steps-would-you-take-to-troubleshoot)
20. How can you save a Terraform plan to a file and apply it later? What are the benefits of this approach? [📖 Answer](mock_2_answers.md#20-how-can-you-save-a-terraform-plan-to-a-file-and-apply-it-later-what-are-the-benefits-of-this-approach)

---

## 📚 Follow-up Resources

- [Terraform Getting Started Guide](https://learn.hashicorp.com/terraform/getting-started/install)
- [Terraform AWS Provider Documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
- [Terraform Best Practices](https://www.terraform-best-practices.com/)
