# Terraform Mock Interview #3 - Beginner Level (Core Language Constructs)

## Question 1: Variables vs Locals vs Outputs
**Question:** What are the differences between Terraform variables, local values (`locals`), and outputs? Provide use cases, lifecycle, visibility, and examples showing how they interact in a module.
[📖 Answer](mock_3_answers.md#question-1-variables-vs-locals-vs-outputs)

---

## Question 2: Input Variable Types and Validation
**Question:** Show examples of declaring string, number, map, object, and list variables with validation rules. When should you use validation vs. descriptive documentation only?
[📖 Answer](mock_3_answers.md#question-2-input-variable-types-and-validation)

---

## Question 3: Default Values and Overriding
**Question:** How are variable defaults overridden using CLI flags, environment variables, and `.tfvars` files? Which order of precedence applies?
[📖 Answer](mock_3_answers.md#question-3-default-values-and-overriding)

---

## Question 4: Organizing Variables and Outputs in Modules
**Question:** Provide a recommended file layout for a reusable module with variables, locals, resources, and outputs. Explain why separation aids readability.
[📖 Answer](mock_3_answers.md#question-4-organizing-variables-and-outputs-in-modules)

---

## Question 5: Sensitive Outputs
**Question:** How do you mark outputs as sensitive? What effect does that have in CLI output and downstream modules?
[📖 Answer](mock_3_answers.md#question-5-sensitive-outputs)

---

## Question 6: Passing Module Outputs Between Modules
**Question:** Show an example of chaining module outputs from a network module into an application module. What happens if the output changes type?
[📖 Answer](mock_3_answers.md#question-6-passing-module-outputs-between-modules)

---

## Question 7: Using Locals for DRY Patterns
**Question:** Provide 3 examples where locals reduce duplication (naming convention, common tags, computed lists). Why not turn every repeated value into a local?
[📖 Answer](mock_3_answers.md#question-7-using-locals-for-dry-patterns)

---

## Question 8: Refactoring Variables into Locals
**Question:** When should something start as a variable and later become a local (or vice versa)? Describe refactoring signals.
[📖 Answer](mock_3_answers.md#question-8-refactoring-variables-into-locals)

---

## Question 9: Output Consumption in Root Module
**Question:** How do you reference output values after `terraform apply` for scripting or integration? Provide both CLI and automation examples.
[📖 Answer](mock_3_answers.md#question-9-output-consumption-in-root-module)

---

## Question 10: Common Pitfalls
**Question:** List common mistakes with variables, locals, and outputs (e.g., circular local references, overexposed outputs, ignoring sensitive marking) and fixes.
[📖 Answer](mock_3_answers.md#question-10-common-pitfalls)

---

**Next:** [View Answers](mock_3_answers.md)

**Time Limit:** 55 minutes  
**Level:** Beginner  
**Focus Areas:** Variables, locals, outputs, module wiring, DRY patterns

**Related:** [Mock Interview #1](mock_1_questions.md) | [Mock Interview #2](mock_2_questions.md) | [Intermediate Level](../intermediate/) | [Advanced Level](../advanced/) | [Expert Level](../expert/)
