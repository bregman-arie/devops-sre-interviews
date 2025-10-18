# Terraform Mock Interview #5 - Intermediate Level (Safety & Triage Scenarios)

## Question 1: Preventing Accidental Destruction (VPC Recreation Scenario)
**Question:** You run `terraform apply` and Terraform proposes destroying and recreating the entire VPC (plus dependent subnets, gateways, route tables). Walk through your triage process and the safest corrective actions to avoid downtime.
[📖 Answer](mock_5_answers.md#question-1-preventing-accidental-destruction-vpc-recreation-scenario)

---

## Question 2: Handling Unexpected Mass Deletes
**Question:** During a plan you see 120 resources queued for destroy but you only changed a tagging local. How do you investigate and roll back safely?
[📖 Answer](mock_5_answers.md#question-2-handling-unexpected-mass-deletes)

---

## Question 3: Mitigating Provider Schema Changes
**Question:** After upgrading a provider, previously stable resources now show force-replace actions. How do you assess and mitigate impact?
[📖 Answer](mock_5_answers.md#question-3-mitigating-provider-schema-changes)

---

## Question 4: Safeguards Against Production Drifts
**Question:** List guardrails to prevent destructive applies in production (configuration, process, tooling). Which are preventive vs. detective vs. corrective?
[📖 Answer](mock_5_answers.md#question-4-safeguards-against-production-drifts)

---

## Question 5: Emergency Abort Procedures
**Question:** Mid-apply you realize unintended deletions are occurring. What immediate actions can you take to abort or limit blast radius? Include backend and IAM considerations.
[📖 Answer](mock_5_answers.md#question-5-emergency-abort-procedures)

---

## Question 6: Using Lifecycle Blocks for Safety
**Question:** How can lifecycle rules like `prevent_destroy`, `ignore_changes`, and `create_before_destroy` help reduce risk? Provide examples and misuse caveats.
[📖 Answer](mock_5_answers.md#question-6-using-lifecycle-blocks-for-safety)

---

## Question 7: Partial Applies and Consistency
**Question:** What are the risks of using `-target` to avoid wide destruction and how do you ensure later full convergence?
[📖 Answer](mock_5_answers.md#question-7-partial-applies-and-consistency)

---

## Question 8: State Surgery Recovery
**Question:** A mistaken manual state edit removed a critical resource mapping. How do you recover without recreating the resource?
[📖 Answer](mock_5_answers.md#question-8-state-surgery-recovery)

---

## Question 9: Detecting Root Cause of Force Replacement
**Question:** A resource shows `-/+` replacement. Outline steps to pinpoint root cause (diff analysis, schema compare, history review).
[📖 Answer](mock_5_answers.md#question-9-detecting-root-cause-of-force-replacement)

---

## Question 10: Pre-Apply Change Approval Workflow
**Question:** Design an approval workflow that blocks destructive applies unless explicitly authorized. Include plan parsing, thresholds, and escalation.
[📖 Answer](mock_5_answers.md#question-10-pre-apply-change-approval-workflow)

---

**Next:** [View Answers](mock_5_answers.md)

**Time Limit:** 70 minutes  
**Level:** Intermediate  
**Focus Areas:** Safe triage, lifecycle safeguards, destructive change prevention, approval workflows, recovery

**Related:** [Mock Interview #1](mock_1_questions.md) | [Mock #2](mock_2_questions.md) | [Mock #3](mock_3_questions.md) | [Mock #4](mock_4_questions.md) | [Advanced Level](../advanced/) | [Expert Level](../expert/)
