# Terraform Mock Interview #4 - Intermediate Level (Dependency Handling Focus)

## Question 1: Implicit vs Explicit Dependencies
**Question:** Explain how Terraform builds implicit dependencies between resources. When is it necessary to use `depends_on` for explicit dependencies, and what are common misuse patterns?
[📖 Answer](mock_4_answers.md#question-1-implicit-vs-explicit-dependencies)

---

## Question 2: Module-Level Dependencies
**Question:** How do you express dependency ordering between modules? Show three techniques and explain which is preferred and why.
[📖 Answer](mock_4_answers.md#question-2-module-level-dependencies)

---

## Question 3: Avoiding Circular Dependencies
**Question:** What causes circular dependencies in Terraform? Provide strategies to refactor a configuration that produces a cycle between networking and application modules.
[📖 Answer](mock_4_answers.md#question-3-avoiding-circular-dependencies)

---

## Question 4: Data Sources vs Outputs
**Question:** Compare using module outputs vs provider data sources to obtain IDs/attributes. How does each choice affect the dependency graph, plan time, and potential for drift?
[📖 Answer](mock_4_answers.md#question-4-data-sources-vs-outputs)

---

## Question 5: Cross-Workspace / Cross-Environment Dependencies
**Question:** How can you reference infrastructure from another Terraform workspace or stack? Show an example using remote state and discuss the trade-offs and risks.
[📖 Answer](mock_4_answers.md#question-5-cross-workspace--cross-environment-dependencies)

---

## Question 6: Using `terraform graph` for Debugging
**Question:** How do you generate and interpret the Terraform dependency graph? Demonstrate how to isolate a problematic dependency chain visually.
[📖 Answer](mock_4_answers.md#question-6-using-terraform-graph-for-debugging)

---

## Question 7: Targeted Operations and Partial Graphs
**Question:** What impact does `terraform plan -target` (or apply with `-target`) have on dependency resolution? When is it acceptable and what are the risks of repeated targeted applies?
[📖 Answer](mock_4_answers.md#question-7-targeted-operations-and-partial-graphs)

---

## Question 8: Forced Recreation with `-replace`
**Question:** How does `-replace` interact with the dependency graph? Provide an example where forcing recreation cascades to dependent resources and explain mitigation.
[📖 Answer](mock_4_answers.md#question-8-forced-recreation-with--replace)

---

## Question 9: Null Resources and Triggers
**Question:** When should `null_resource` be used to model dependencies? Show a safe pattern and an anti-pattern involving external scripts and timing dependencies.
[📖 Answer](mock_4_answers.md#question-9-null-resources-and-triggers)

---

## Question 10: Optimizing Large Dependency Graphs
**Question:** What techniques reduce plan/apply time in large configurations (hundreds of resources)? Cover graph segmentation, remote state design, module boundaries, and provider throttling.
[📖 Answer](mock_4_answers.md#question-10-optimizing-large-dependency-graphs)

---

## Question 11: Conditional Dependencies with `count` / `for_each`
**Question:** How do conditional creation constructs (`count`, `for_each`) affect dependency resolution? Provide examples showing pitfalls with index shifts and stable keys.
[📖 Answer](mock_4_answers.md#question-11-conditional-dependencies-with-count--for_each)

---

## Question 12: Dependency Hygiene & Anti-Patterns
**Question:** List common dependency anti-patterns in Terraform and how to remediate each (e.g., overuse of `depends_on`, hidden timing dependencies, mixing provisioning and infrastructure logic).
[📖 Answer](mock_4_answers.md#question-12-dependency-hygiene--anti-patterns)

---

**Next:** [View Answers](mock_4_answers.md)

**Time Limit:** 70 minutes  
**Level:** Intermediate  
**Focus Areas:** Dependency modelling, explicit vs implicit dependencies, graph optimization, safe targeting, cross-stack references

**Related:** [Mock Interview #1](mock_1_questions.md) | [Mock Interview #2](mock_2_questions.md) | [Mock Interview #3](mock_3_questions.md) | [Advanced Level](../advanced/) | [Expert Level](../expert/)
