# Google Cloud Platform (GCP) - Advanced Interview Mock #3

> **Difficulty:** Advanced  
> **Duration:** ~60 minutes  
> **Theme:** Ultra Low-Latency, High Availability & Fault Tolerance at Global Scale
> **Goal:** Evaluate ability to design multi-region, latency-optimized, failure-resilient systems with measurable SLOs and cost/performance trade‑offs.

---

## 🧠 Section 1: Core Questions

1. Design a globally distributed, highly available, fault-tolerant, and ultra low-latency architecture on GCP for a mission‑critical API (p99 < 40ms globally; multi-region; RTO 2m; RPO < 5s). Describe compute, data, routing, caching, observability, and failure isolation strategies. [📖 Answer](mock_3_answers.md#1-design-a-globally-distributed-highly-available-fault-tolerant-and-ultra-low-latency-architecture-on-gcp-for-a-missioncritical-api-p99--40ms-globally-multi-region-rto-2m-rpo--5s-describe-compute-data-routing-caching-observability-and-failure-isolation-strategies)
2. Explain techniques to reduce tail latency (p99/p99.9) in GCP-based microservices. Include hedged requests, adaptive concurrency, priority queues, and load shedding. [📖 Answer](mock_3_answers.md#2-explain-techniques-to-reduce-tail-latency-p99p999-in-gcp-based-microservices-include-hedged-requests-adaptive-concurrency-priority-queues-and-load-shedding)
3. Compare Cloud Spanner, Bigtable, Memorystore, and AlloyDB for sub-10ms read/write use cases. Provide a decision matrix for hybrid usage in a latency-sensitive platform. [📖 Answer](mock_3_answers.md#3-compare-cloud-spanner-bigtable-memorystore-and-alloydb-for-sub-10ms-readwrite-use-cases-provide-a-decision-matrix-for-hybrid-usage-in-a-latency-sensitive-platform)
4. How would you design multi-region data replication that balances strong consistency for critical writes and eventual consistency for derived views—while preventing cross-region write hotspots? [📖 Answer](mock_3_answers.md#4-how-would-you-design-multi-region-data-replication-that-balances-strong-consistency-for-critical-writes-and-eventual-consistency-for-derived-viewswhile-preventing-cross-region-write-hotspots)
5. Describe a layered caching strategy (edge → regional → service → in-process) for latency SLIs. How do you measure cache effectiveness and avoid stale data risk? [📖 Answer](mock_3_answers.md#5-describe-a-layered-caching-strategy-edge--regional--service--in-process-for-latency-slis-how-do-you-measure-cache-effectiveness-and-avoid-stale-data-risk)
6. Outline an observability design focused on latency: metrics, high-cardinality labeling, tracing sampling strategies, and real-time anomaly detection (predictive). [📖 Answer](mock_3_answers.md#6-outline-an-observability-design-focused-on-latency-metrics-high-cardinality-labeling-tracing-sampling-strategies-and-real-time-anomaly-detection-predictive)
7. How would you run controlled latency experiments (A/B or shadow traffic) to validate optimizations without impacting production SLOs? [📖 Answer](mock_3_answers.md#7-how-would-you-run-controlled-latency-experiments-ab-or-shadow-traffic-to-validate-optimizations-without-impacting-production-slos)
8. Detail a failure containment model: fault domains, blast-radius reduction, regional isolation, and progressive degradation under overload—mapped to concrete GCP services. [📖 Answer](mock_3_answers.md#8-detail-a-failure-containment-model-fault-domains-blast-radius-reduction-regional-isolation-and-progressive-degradation-under-overloadmapped-to-concrete-gcp-services)

---

## ⚙️ Section 2: Scenario

**Scenario:**  
You must design a real-time global multiplayer gaming backend (matchmaking + state synchronization + leaderboards) with:
- **Latency Targets:** p95 < 30ms regional; p99 < 45ms cross-region relay
- **Traffic:** 5 million concurrent players; spikes 3× within 2 minutes
- **Consistency:** Strong for transactional inventory/currency; eventual for presence/status; monotonic for leaderboard updates
- **Availability:** 99.99% (≤ 52m downtime/year)
- **Resilience:** Survive a full regional outage with < 120s impact and no player data loss
- **Security:** Prevent cheating/exploit replay; cryptographic session integrity; minimal data exfiltration risk

Design an end-to-end GCP architecture addressing:
1. Matchmaking latency and proximity routing
2. Real-time state fan-out (Pub/Sub vs WebSockets vs UDP gateways)
3. Data tier composition (Spanner, Bigtable, Memorystore, Firestore?)
4. Leaderboard freshness vs cost trade-offs
5. Cheat detection ML inference path (Vertex AI / custom accelerators)
6. Disaster recovery + session continuity
7. Observability: per-player latency heatmaps + tail spike early warning
8. Cost controls while keeping SLO headroom

[📖 Answer](mock_3_answers.md#️-section-2-scenario---answer)

---

## 🧩 Section 3: Problem-Solving

**Task:**  
Current production system shows intermittent p99 latency spikes (40ms → 400ms) during regional GC pauses and cascading retries. You have limited one-week remediation window before a launch event.

Provide a prioritized remediation plan including:
1. Immediate triage + instrumentation gaps
2. Quick-win latency reducers (config / infra tuning)
3. Medium-term structural fixes (protocol, caching, data model)
4. Tail-latency regression guardrails (CI/CD & canaries)
5. Risk matrix with ROI rationale

[📖 Answer](mock_3_answers.md#-section-3-problem-solving---answer)

---

## 🎯 Section 4: Technical Deep Dive

**Question:**  
Design a hybrid adaptive concurrency + predictive autoscaling system on GKE for an API with: bursty traffic, bimodal latency distribution, heterogeneous request cost, and expensive cold starts. Include:
1. Real-time concurrency controller algorithm
2. Queueing + early shedding logic
3. Forecast-driven pre-scaling (Vertex AI)
4. Feedback loops for error budget usage
5. Validation & continuous tuning

[📖 Answer](mock_3_answers.md#-section-4-technical-deep-dive---answer)

---

## 📚 Additional Topics to Review
- Tail latency literature (hedged requests, redundancy, queue theory)
- Spanner leader placement + split design for latency
- Bigtable hotspot avoidance & adaptive read patterns
- Adaptive concurrency control (Envoy / custom token bucket)
- Advanced Cloud Armor rate policies for prioritization
- Workload partitioning & traffic classification strategies
- Load test methodology (baseline vs steady-state vs surge)
- SLO error budget–driven release gating

---

[⬅ Back to Main](../../../README.md) | [📋 All GCP Interviews](../../README.md)
