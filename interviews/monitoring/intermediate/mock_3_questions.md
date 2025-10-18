# Monitoring & Observability - Intermediate Interview Mock #3

> **Difficulty:** Intermediate  
> **Duration:** ~45 minutes  
> **Goal:** Assess ability to select meaningful metrics, architect multi-layer observability, handle scale/cardinality, and design SLO-based alerting.

---

## 🧠 Section 1: Core Questions

1. Which metrics do you monitor on Kubernetes clusters? (Break down by control plane, node health, workloads, and platform add-ons.)  
2. Which metrics do you monitor on EC2 instances or other VM-based workloads? (Include system, application, and agent-level.)  
3. Which metrics do you monitor for databases or queues (e.g., PostgreSQL, RabbitMQ)? (Highlight saturation, throughput, errors, durability.)  
4. Explain how you’d use Prometheus + Grafana, Datadog, ELK, or CloudWatch to build an observability stack. (Contrast strengths, deployment models, and integration paths.)  
5. How do you handle high cardinality metrics in Prometheus? (Discuss label hygiene, relabeling, recording rules, and aggregation.)  
6. How would you alert on SLO breaches instead of raw thresholds? (Explain error budgets, burn rate alerts, multi-window multi-burn strategies.)  
7. How do you avoid alert fatigue? (Cover deduplication, grouping, inhibition, noise audits, pruning, and aligning alerts to user impact/SLOs.)  
8. How would you design alerting for a multi-environment setup (staging vs prod)? (Different severities, filtered routing, gating pre-prod alerts, shadow alerts, canary-based promotion.)  
9. What’s the difference between symptom-based and cause-based alerts? (Define each, trade-offs for MTTR vs precision, layering strategy.)  
10. How do you ensure alerts are actionable and meaningful? (Context enrichment: tags, recent deploy info, runbook links, ownership, suggested remediation.)  
11. Give an example of how you’d use Alertmanager or similar systems (routing tree, grouping labels, inhibition rules, silences for maintenance, multi-channel escalation).  

---

## ⚙️ Section 2: Scenario

**Scenario:**  
You’ve joined a company running a SaaS analytics platform composed of:  
- Ingress NGINX controllers terminating TLS for ~2000 rps peak traffic.  
- ~40 microservices (HTTP + gRPC) on Kubernetes (EKS/GKE).  
- A PostgreSQL primary + 2 replicas, Redis for caching, and RabbitMQ for async workloads.  
- Prometheus + Alertmanager self-managed, Grafana dashboards, Loki for logs, Tempo for traces (early adoption), plus a legacy ELK stack still used by one team.  

Current pain points:  
1. High-cardinality metrics (per-user, per-feature labels) creating Prometheus churn and slow queries.  
2. Alert fatigue (2000 pages/month; on-call ignores some alerts).  
3. Staging generates noisy alerts during load tests, masking emerging production issues.  
4. SLOs exist for signup and dashboard load, but alerts are threshold-based (HTTP 5xx > 2%) causing flapping during deploys.  
5. Costs rising: metrics TSDB size + log retention (30 days hot, rarely queried).  

Your task: Outline a remediation & uplift plan covering:  
- Telemetry rationalization (metrics, logs, traces) and cardinality control.  
- Multi-window burn rate SLO alert design for key user journeys.  
- Environment-aware alert routing & suppression strategy (staging vs prod vs canary).  
- Ownership & runbook enrichment to make alerts actionable.  
- Cost optimization (tiered log storage, recording rules, downsampling, remote-write).  
- Migration path from legacy ELK to Loki/Tempo/OpenTelemetry without halting feature delivery.  

Deliverables (discuss, not implement code):  
1. Prioritized 30/60/90 day roadmap.  
2. Example improved metric taxonomy (base, derived, SLI).  
3. Alertmanager routing + inhibition high-level design.  
4. Risk mitigation for migration phases.  

Be ready to justify trade-offs (build vs buy, push vs pull, sampling strategies, cost vs fidelity).

---

## 🧩 Section 3: Problem-Solving

Work through the following practical design prompts:

1. Design: Propose 3 SLIs and target SLOs for the “Dashboard Load” journey (ingress -> API gateway -> aggregation service -> Postgres). Include measurement method and potential sources of error.  
2. Burn Rate Alerts: Specify two multi-window alert rules (fast + slow) for an availability SLO of 99.9% (30-day window) using PromQL style pseudocode. Explain why the chosen burn factors help early detection without noise.  
3. Cardinality Reduction: Given a metric http_request_duration_seconds{user_id, feature_flag, build_sha, pod, method, status}, list which labels to keep, aggregate, or move to trace/log context; propose a recording rule that produces p95 latency per service while preserving query speed.  
4. Alert Actionability: Draft a minimal alert template including annotations (runbook, recent deploy diff link, owning team, severity rationale) and labels enabling grouping & inhibition.  
5. Alertmanager Design: Sketch a routing tree (text) for: critical prod -> PagerDuty; warning prod -> Slack #alerts; staging -> Slack #staging-alerts only; inhibit downstream service 5xx alerts if upstream latency/availability alert is firing.  
6. Cost Optimization: Provide 3 tactics to shrink Prometheus TSDB by >30% without losing SLI accuracy; include rationale and risk.  
7. Tracing Sampling: Recommend a sampling approach (e.g., head + tail hybrid) for high-volume services, describing when traces are retained fully (errors, high latency) and how sampling impacts metric exemplars.  
8. Migration Plan: Outline steps to shift teams from ELK to Loki/Tempo with dual-write, success criteria, and a deprecation schedule.  
9. Runbook Quality: List 5 checklist items every production alert runbook must contain to reduce MTTR.  
10. Testing Alerts: Describe a safe method to load-test new alert rules (synthetic traffic, replay, rule evaluation dry-run) before enabling paging routes.  

Optional Extension (if time allows): Provide an example PrometheusRule YAML snippet (pseudo) showing a burn rate alert + inhibition label usage.
