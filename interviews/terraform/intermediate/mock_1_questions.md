# Terraform Mock Interview #1 - Intermediate Level

## Question 1: Remote State Management

**Question:** What are the benefits of using remote state in Terraform? How would you configure an S3 backend for storing Terraform state, and what additional considerations should you keep in mind for team collaboration?

---

## Question 2: Terraform Modules

**Question:** What are Terraform modules and how do they promote code reusability? Can you walk through how you would create a custom module for an AWS VPC and how you would use it in multiple environments?

---

## Question 3: Terraform Workspaces

**Question:** Explain Terraform workspaces and their use cases. How would you use workspaces to manage multiple environments (dev, staging, prod) and what are the limitations of this approach?

---

## Question 4: Data Sources and Lifecycle Rules

**Question:** What are data sources in Terraform and when would you use them? Additionally, explain the different lifecycle rules available in Terraform and provide examples of when you might use `create_before_destroy` and `prevent_destroy`.

---

## Question 5: Terraform Import and State Management

**Question:** You have existing infrastructure that was created manually, and you need to bring it under Terraform management. How would you approach this using `terraform import`? What challenges might you face and how would you handle state file conflicts?

---

## Question 6: Advanced Variable Handling

**Question:** Explain the different variable types in Terraform (list, map, object, tuple). How would you use conditional expressions and the `for` expression to dynamically create resources based on input variables?

---

## Question 7: Terraform Functions and Expressions

**Question:** Describe some commonly used Terraform functions and provide examples. How would you use functions like `lookup()`, `merge()`, `templatefile()`, and `length()` in real-world scenarios?

---

## Question 8: State Locking and Concurrent Operations

**Question:** What is state locking in Terraform and why is it important? How would you implement state locking with different backends, and what happens when a lock is not properly released?

---

## Question 9: Terraform Plan Files and Targeted Operations

**Question:** What are Terraform plan files and when would you use them? Additionally, explain how targeted applies work (`terraform apply -target`) and when this might be necessary or dangerous.

---

## Question 10: Error Handling and Debugging

**Question:** You're working on a complex Terraform configuration with multiple modules and providers. A `terraform apply` fails with dependency issues and resource conflicts. Walk through your debugging methodology, including log analysis, state inspection, and resolution strategies.

---

**Time Limit:** 60 minutes  
**Level:** Intermediate  
**Focus Areas:** Remote state, modules, workspaces, advanced configurations, debugging, and team collaboration workflows
