# Terraform Mock Interview #2 - Intermediate Level

## Question 1: Dynamic Blocks and Complex Resource Configuration

**Question:** Explain dynamic blocks in Terraform and provide an example of how you would use them to create multiple security group rules based on a variable input. What are the benefits and potential drawbacks of using dynamic blocks?

---

## Question 2: Terraform Providers and Version Constraints

**Question:** You're working on a project that needs to use specific versions of providers for compatibility. How would you configure provider version constraints, and what's the difference between `~>`, `>=`, and `=` version operators? How do you handle provider upgrades in a team environment?

---

## Question 3: Local Values and Complex Data Transformation

**Question:** Demonstrate how to use local values to transform and process complex data structures. Given a list of application configurations, show how you would use locals to generate standardized resource naming, tags, and configurations for multiple environments.

---

## Question 4: Count vs for_each - When and Why

**Question:** Compare and contrast `count` and `for_each` meta-arguments in Terraform. Provide scenarios where each is more appropriate, and explain what happens when you need to change from one to the other in an existing configuration.

---

## Question 5: Terraform Provisioners and Null Resources

**Question:** When would you use Terraform provisioners, and why are they generally discouraged? Provide an example of using a null resource with local-exec provisioner, and explain alternative approaches to achieve the same result.

---

## Question 6: Custom Provider Configuration and Aliases

**Question:** You need to deploy resources across multiple AWS regions and accounts within the same Terraform configuration. How would you configure multiple provider aliases, and how would you manage authentication for different accounts?

---

## Question 7: Terraform Backend Migration

**Question:** Your team needs to migrate Terraform state from local storage to a remote S3 backend without losing existing infrastructure. Walk through the migration process, including backup strategies and validation steps.

---

## Question 8: Resource Drift Detection and Management

**Question:** Explain how to detect and handle configuration drift in Terraform. What tools and techniques would you use to identify when infrastructure has been modified outside of Terraform, and how would you resolve conflicts?

---

## Question 9: Terraform Testing and Validation

**Question:** Describe different approaches to testing Terraform configurations. How would you implement automated testing for modules, validate configurations before apply, and ensure compliance with organizational standards?

---

## Question 10: Performance Optimization and Large Infrastructure

**Question:** Your Terraform configuration manages thousands of resources and takes too long to plan/apply. What strategies would you implement to optimize performance, and how would you structure the configuration for better maintainability?

---

**Time Limit:** 60 minutes  
**Level:** Intermediate  
**Focus Areas:** Advanced resource management, provider configuration, state management, testing, performance optimization, and operational best practices
