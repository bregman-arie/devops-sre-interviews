# Monitoring & Observability - Beginner Interview Mock #1

> **Difficulty:** Beginner  
> **Duration:** ~30 minutes  
> **Goal:** Assess foundational understanding of monitoring vs logging vs observability, core telemetry concepts, and baseline metric selection.

---

## 🧠 Section 1: Core Questions

1. What’s the difference between monitoring, logging, and observability? [📖 Answer](mock_1_answers.md#1-whats-the-difference-between-monitoring-logging-and-observability)
2. How do you design an effective monitoring strategy for a distributed system? [📖 Answer](mock_1_answers.md#2-how-do-you-design-an-effective-monitoring-strategy-for-a-distributed-system)
3. Explain black-box vs. white-box monitoring. [📖 Answer](mock_1_answers.md#3-explain-black-box-vs-white-box-monitoring)
4. What are the key metrics you’d monitor for infrastructure and applications? [📖 Answer](mock_1_answers.md#4-what-are-the-key-metrics-youd-monitor-for-infrastructure-and-applications)
5. What’s the difference between proactive and reactive monitoring? [📖 Answer](mock_1_answers.md#5-whats-the-difference-between-proactive-and-reactive-monitoring)

---

## ⚙️ Section 2: Scenario

**Scenario:**  
You are onboarding a new microservices application composed of: an API gateway, 5 backend services, a PostgreSQL database, Redis cache, and a message queue. There is no existing telemetry. Incidents have been missed because engineers only check logs after customer complaints.

Design an initial monitoring rollout plan covering: telemetry priorities (logs/metrics/traces), instrumentation approach, SLO candidates, alerting philosophy (avoid spam), dashboard grouping, and quick wins for meantime-to-detect reduction.

[📖 Answer](mock_1_answers.md#️-section-2-scenario---answer)

---

## 🧩 Section 3: Problem-Solving

**Task:**  
Propose a minimal Service Level Indicator (SLI) + alerting specification for the API gateway that differentiates transient blips from real outages while staying under an alert budget of 4 pages/week. Include: SLIs, calculation windows, burn-rate alert tiers, and suppression logic (e.g., deployments).

[📖 Answer](mock_1_answers.md#-section-3-problem-solving---answer)

---

## 💡 Additional Questions (Time Permitting)

1. What is cardinality and why does it matter in metrics design? [📖 Answer](mock_1_answers.md#1-what-is-cardinality-and-why-does-it-matter-in-metrics-design)
2. What are structured logs and why are they useful? [📖 Answer](mock_1_answers.md#2-what-are-structured-logs-and-why-are-they-useful)
3. What is distributed tracing and what problems does it solve? [📖 Answer](mock_1_answers.md#3-what-is-distributed-tracing-and-what-problems-does-it-solve)
4. How do you prevent alert fatigue? [📖 Answer](mock_1_answers.md#4-how-do-you-prevent-alert-fatigue)

---

> **Next Steps:** Practice by instrumenting a demo service with metrics, logs, and traces (e.g., Prometheus + OpenTelemetry + Loki/Elastic) and define a simple SLO.
