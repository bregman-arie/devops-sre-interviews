# Monitoring & Observability - Advanced Interview Mock #3

> **Difficulty:** Advanced  
> **Duration:** ~60 minutes  
> **Goal:** Evaluate ability to architect resilient, cost-efficient, adaptive observability platforms at scale.

---

## 🧠 Section 1: Core Questions

1. End-to-End Observability Pipeline Design: Architect collection → buffering → enrichment → storage → query → correlation → alerting; justify technology choices (Prometheus, Mimir/VictoriaMetrics, OTLP, ClickHouse, Kafka).  
2. Scaling Distributed Tracing: Discuss sampling strategies (head, tail, adaptive, reservoir) and dynamic adjustment based on error budget burn & anomaly detection.  
3. Telemetry SLOs: Define SLIs for observability system health (e.g., span ingestion latency <5s, metric scrape success >99.9%, alert delivery <30s, log indexing freshness <2m).  
4. Dependency Graph Automation: Methods to continuously build/update service topology and detect orphaned or shadow dependencies (trace mining, eBPF flow capture, graph diff).  
5. Cardinality Governance: Implement a policy engine preventing unsafe label explosions (admission controller + linting + per-label cardinality budgets).  
6. Multi-Region Observability: Strategies for federation vs sharding; query fan-out, remote write, aggregation accuracy, failure isolation.  
7. Cost Optimization at Scale: Downsampling, retention tiering, query acceleration indices, compression trade-offs, cold storage rehydration patterns.  
8. Alert Quality Scoring: Framework to quantify alert value (MTTA reduction, false positive ratio, unique signal contribution) and prune low-scoring alerts.  
9. Distributed Latency Root Cause Automation: Using trace span anomaly detection + service dependency graph to auto-suggest suspect services.  
10. Secure & Compliant Telemetry: Approaches for PII scrubbing, schema contracts, field-level encryption, audit trails, tenant isolation.  

---

## ⚙️ Section 2: Scenario

**Scenario:**  
You inherit an observability stack ingesting: ~15M metrics samples/sec, 120K spans/sec, 2TB/day logs. Engineers report slow dashboard loads (>8s), missed alerts (delivery delays), and runaway storage costs. Some services hardcode trace exporters; sampling fixed at 10%. Remote Prometheus instances push to a central Thanos/Mimir setup. Alertmanager cluster occasionally partitions causing duplicate pages.

Pain Points:  
- Query slowness due to high churn + unoptimized label sets.  
- Inconsistent trace sampling causing sparse latency root cause data.  
- Metrics retention uniform (30d) despite varied usage patterns.  
- Lack of telemetry SLOs → outages in pipeline unnoticed until after incidents.  
- Alert duplication and silence conflicts during partitions.  

Task: Propose a phased modernization plan (0-30 / 30-60 / 60-120 days) improving performance, reliability, and cost while minimizing risk.

---

## 🧩 Section 3: Problem-Solving

1. Draft Telemetry SLOs: Provide 4 SLIs + targets + error budget policy and associated burn alerts.  
2. Sampling Controller: Design an adaptive sampling service (inputs: span error rate, tail latency percentiles, throughput; outputs: updated sampling config via OTLP).  
3. Label Optimization: Provide a procedure/tooling workflow to detect high-cardinality offenders (top-N series growth, label entropy) and remediate safely.  
4. Multi-Window Alert Delivery SLA: Ensure alert delivery reliability with redundancy (queue + retry + circuit breaker + idempotent dedupe).  
5. Storage Tiering Plan: Map data classes (SLI metrics, diagnostic metrics, raw spans, aggregated spans, hot logs, cold logs) to retention & access patterns.  
6. Partition Resilience: Design Alertmanager HA architecture (mesh gossip hardening, sharding, quorum-based dedupe) and a fallback path.  
7. Automated Coverage Audit: Outline a job that cross-references deployed services vs presence of baseline telemetry (metrics/traces/logs) and outputs a gap report.  
8. Latency Anomaly Triage: Define a pipeline turning span outliers into candidate root cause ranking (features: error ratio shift, critical path inflation, resource tag correlation).  
9. Incident Drill: Construct a game-day validating telemetry SLOs and alert pipeline (synthetic load, induced trace sampling shift, forced scrape error).  
10. Risk Register: List top 5 modernization risks + mitigations (data loss, cardinality explosion, sampling misconfiguration, alert storm, migration rollback).  

Optional: Provide pseudo YAML/JSON for an adaptive sampling policy update event.
