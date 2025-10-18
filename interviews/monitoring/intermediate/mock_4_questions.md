# Monitoring & Observability - Intermediate Interview Mock #4

> **Difficulty:** Intermediate  
> **Duration:** ~50 minutes  
> **Goal:** Assess ability to instrument distributed services, troubleshoot latency, design actionable alerts, and validate telemetry coverage across environments.

---

## 🧠 Section 1: Core Questions

1. How do you monitor a microservices / Kubernetes-based system end-to-end (layers: cluster, platform, service, request, user journey)?  
2. Explain how distributed tracing (Jaeger / OpenTelemetry) complements metrics and logs. Include when traces are superior and when they’re redundant.  
3. How do you troubleshoot high latency in a distributed system (systematic workflow: confirm symptom -> isolate span -> identify resource contention vs network vs dependency)?  
4. How do you visualize service dependencies (service map strategies: span analysis, topology graph building from ingress logs, dependency tracing DAGs)?  
5. What’s your process for defining SLIs, SLOs, and SLAs for a new feature (steps: user journey decomposition, candidate SLIs, feasibility, baseline, target negotiation, error budget policy)?  
6. How do you instrument applications for metrics and traces (libraries vs manual, semantic conventions, RED/USE patterns, exemplars)?  
7. How do you handle log retention and cost optimization (tiered storage, field whitelisting, dynamic sampling, structuring for compression)?  
8. Describe a monitoring pipeline you’ve built end-to-end (collection -> transport -> storage -> query -> alerting -> visualization -> governance).  
9. How would you ensure monitoring systems themselves are reliable (telemetry health SLIs: ingestion lag, scrape success %, rule eval latency, alert delivery success)?  
10. Strategies to verify monitoring covers all critical production paths (coverage matrix, synthetic canaries, dependency diff audits, deployment hooks).  

---

## ⚙️ Section 2: Scenario

**Scenario:**  
A payments microservice stack (API gateway -> auth -> payments -> ledger -> notification) shows sporadic 95th percentile latency spikes from 300ms to 3s. CPU, memory, and pod restarts are normal. Tracing exists but only on two services (gateway, payments). Logs are verbose but unstructured. Dashboard focuses on averages. Alerts fire for p95 > 1s but flap during traffic ramp.

Your tasks:  
- Outline a latency investigation plan distinguishing symptom vs cause signals.  
- Propose additional instrumentation changes (span attributes, custom metrics).  
- Suggest a refined alert strategy (multi-window + percentile + saturation correlation).  
- Recommend dependency visualization improvements to uncover hidden database or queue contention.  
- Provide a path to structured logging minimizing added cardinality.  

---

## 🧩 Section 3: Problem-Solving

1. High CPU on a critical node: Enumerate a triage flow (verify node vs pod, kubelet health, throttling, noisy neighbor, scheduling pressure).  
2. Diagnose "no data" alerts in Prometheus: List differential checks (rule evaluation errors, scrape error metrics, target down, service discovery gap, relabel drop, retention expiration).  
3. Flapping alert remediation: Provide 5 stabilization tactics (hysteresis, multi-window burn, aggregation, inhibition during deploy, dynamic thresholds).  
4. Slow service with normal CPU/memory: What next? (Check latency breakdown: network RTT, GC pauses, I/O wait, lock contention, external dependency, DNS).  
5. Coverage validation: Draft a telemetry coverage checklist for a new critical path (SLI mapped spans, error taxonomy, saturation metrics, synthetic journey, alert dry-run).  
6. Instrumentation plan: Convert an uninstrumented Go service to Prometheus + OpenTelemetry with minimal cardinality (metrics names + exemplar use).  
7. Service dependency mapping: Outline an automated job that builds a daily graph from trace data + ingress logs highlighting new dependencies for review.  
8. Actionable alert template: Provide label + annotation schema ensuring ownership, runbook, severity rationale, last deploy metadata.  
9. Resource vs dependency latency disambiguation: Show how combining histograms + span tags can distinguish CPU saturation vs downstream slowness.  
10. Continuous improvement loop: Define a monthly observability review process (inputs: pager stats, false positives, missed incidents, coverage gaps; outputs: backlog items).  

Optional: Add a PromQL pseudo-rule showing a multi-window latency burn alert referencing error budget consumption.
