# Terraform - Advanced Interview Mock #2

> **Difficulty:** Advanced  
> **Duration:** ~60 minutes  
> **Goal:** Assess deep understanding of Terraform enterprise patterns, state management, complex module design, and production deployment strategies.

---

## 🧠 Section 1: Core Questions

1. How would you implement a **zero-downtime blue-green deployment** strategy using Terraform with AWS Application Load Balancer target groups? Include considerations for database migrations and rollback procedures. [📖 Answer](mock_2_answers.md#1-how-would-you-implement-a-zero-downtime-blue-green-deployment-strategy-using-terraform-with-aws-application-load-balancer-target-groups-include-considerations-for-database-migrations-and-rollback-procedures)

2. Design a **multi-region disaster recovery** setup using Terraform that can automatically failover a stateful application between AWS regions. Address state file replication, cross-region networking, and data synchronization. [📖 Answer](mock_2_answers.md#2-design-a-multi-region-disaster-recovery-setup-using-terraform-that-can-automatically-failover-a-stateful-application-between-aws-regions-address-state-file-replication-cross-region-networking-and-data-synchronization)

3. Explain how to implement **dynamic module composition** where module behavior changes based on input maps or complex data structures. Provide examples using `for_each`, `count`, and conditional expressions. [📖 Answer](mock_2_answers.md#3-explain-how-to-implement-dynamic-module-composition-where-module-behavior-changes-based-on-input-maps-or-complex-data-structures-provide-examples-using-for_each-count-and-conditional-expressions)

4. How would you implement **custom providers** for integrating with proprietary APIs? Walk through the provider development lifecycle and publishing to the Terraform Registry. [📖 Answer](mock_2_answers.md#4-how-would-you-implement-custom-providers-for-integrating-with-proprietary-apis-walk-through-the-provider-development-lifecycle-and-publishing-to-the-terraform-registry)

5. Design a **state migration strategy** for moving from local state to remote backend while maintaining multiple environments and ensuring zero data loss. Include workspace migration patterns. [📖 Answer](mock_2_answers.md#5-design-a-state-migration-strategy-for-moving-from-local-state-to-remote-backend-while-maintaining-multiple-environments-and-ensuring-zero-data-loss-include-workspace-migration-patterns)

6. Implement **policy-as-code** using Terraform Sentinel or OPA to enforce organizational standards around resource tagging, cost controls, and security compliance. [📖 Answer](mock_2_answers.md#6-implement-policy-as-code-using-terraform-sentinel-or-opa-to-enforce-organizational-standards-around-resource-tagging-cost-controls-and-security-compliance)

7. How do you handle **circular dependencies** in complex Terraform configurations? Provide strategies for refactoring and prevention. [📖 Answer](mock_2_answers.md#7-how-do-you-handle-circular-dependencies-in-complex-terraform-configurations-provide-strategies-for-refactoring-and-prevention)

8. Design a **GitOps workflow** with Terraform that implements proper approval processes, automated testing, and progressive deployment across environments using tools like Atlantis or Terraform Cloud. [📖 Answer](mock_2_answers.md#8-design-a-gitops-workflow-with-terraform-that-implements-proper-approval-processes-automated-testing-and-progressive-deployment-across-environments-using-tools-like-atlantis-or-terraform-cloud)

---

## ⚙️ Section 2: Scenario

**Scenario:**  
Your organization runs a microservices platform across multiple AWS accounts (dev, staging, prod) with strict compliance requirements. You need to implement a Terraform solution that:

- Deploys identical infrastructure patterns across accounts with environment-specific configurations
- Enforces security policies (encryption, network isolation, access controls)
- Implements cost allocation and resource optimization
- Supports both planned deployments and emergency rollbacks
- Maintains audit trails and compliance reporting
- Handles secrets management and credential rotation

The platform includes: EKS clusters, RDS instances, ElastiCache, S3 buckets, Lambda functions, and API Gateway endpoints. Design the complete Terraform architecture including module structure, state management, CI/CD integration, and operational procedures.

[📖 Answer](mock_2_answers.md#️-section-2-scenario---answer)

---

## 🧩 Section 3: Problem-Solving

**Task:**  
During a critical production deployment, you discover that Terraform is trying to recreate a production database (force replacement) due to a seemingly minor configuration change. The database contains 500GB of data and serves live traffic.

Analyze the situation and provide:
1. Immediate steps to prevent data loss
2. Root cause analysis methodology
3. Safe remediation strategy
4. Long-term prevention measures
5. Team communication plan

Consider aspects like state manipulation, import strategies, resource lifecycle rules, and testing approaches.

[📖 Answer](mock_2_answers.md#-section-3-problem-solving---answer)

---

## 🔧 Section 4: Code Review

**Code Review Task:**  
Review the following Terraform module and identify security, performance, and maintainability issues. Provide specific recommendations for improvement.

```hcl
variable "instances" {
  type = list(string)
  default = ["web-1", "web-2", "web-3"]
}

resource "aws_instance" "web" {
  count                  = length(var.instances)
  ami                   = "ami-12345678"
  instance_type         = "t3.micro"
  key_name              = "my-key"
  vpc_security_group_ids = [aws_security_group.web.id]
  
  tags = {
    Name = var.instances[count.index]
  }
}

resource "aws_security_group" "web" {
  name = "web-sg"
  
  ingress {
    from_port   = 0
    to_port     = 65535
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  egress {
    from_port   = 0
    to_port     = 65535
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

output "instance_ips" {
  value = aws_instance.web.*.public_ip
}
```

[📖 Answer](mock_2_answers.md#-section-4-code-review---answer)

---

**Next:** [View Answers](mock_2_answers.md)
