# Monitoring & Observability - Beginner Interview Mock #1 Answers

## 1. What’s the difference between monitoring, logging, and observability?
**Monitoring:** The practice of collecting and visualizing predefined telemetry (metrics, events) to track system health against expectations (dashboards & alerts).  
**Logging:** Time-stamped, often structured records of discrete events or state changes emitted by components; raw, high-detail context.  
**Observability:** A system property: given telemetry (metrics, logs, traces), you can infer internal state and answer new questions without changing code. Observability emphasizes rich, correlated signals enabling unknown-unknown investigation.  
**Summary:** Monitoring = watching known signals; logging = granular event data; observability = ability to explore & explain.

## 2. How do you design an effective monitoring strategy for a distributed system?
1. Define user-centric SLOs first (e.g., request latency, success rate).  
2. Map services → critical paths → SLIs (availability, latency, errors, saturation).  
3. Instrument standardized metrics (RED / USE patterns).  
4. Capture structured, contextual logs (correlation IDs, trace IDs).  
5. Implement distributed tracing for causal latency breakdown.  
6. Control cardinality (labels: avoid user_id, use bounded sets).  
7. Tier alerting: fast burn (severe), slow burn (persistent), paging only for SLO impact.  
8. Provide focused dashboards: (a) service overview, (b) dependency map, (c) SLO health.  
9. Automate instrumentation standards (SDKs / sidecars).  
10. Continuously review noisy alerts; couple with on-call retro feedback.

## 3. Explain black-box vs. white-box monitoring.
- **Black-box:** External perspective: probes, synthetic tests, or clients simulate usage. Detects user-impacting failures regardless of internal state (e.g., HTTP 200 + latency). Useful for SLA verification.  
- **White-box:** Internal instrumentation: metrics/traces/logs inside the application (queue depth, heap, DB connections). Enables root cause analysis and proactive capacity management.  
Best practice: Combine both—black-box for correctness, white-box for diagnosis.

## 4. What are the key metrics you’d monitor for infrastructure and applications?
Use common frameworks:  
- **RED (Requests, Errors, Duration)** for user-facing services.  
- **USE (Utilization, Saturation, Errors)** for resources (CPU, memory, disk IO, network, thread pools).  
Examples: request rate, p50/p95/p99 latency, error ratio, CPU %, memory working set, GC pause time, queue length, cache hit rate, DB connection usage, disk latency, network egress, saturation signals (load avg vs cores).  
Add business SLIs (e.g., checkout success, messages processed/min).  
Exclude vanity metrics; prefer bounded, actionable ones.

## 5. What’s the difference between proactive and reactive monitoring?
- **Reactive:** Detects and responds after a threshold breach or failure (alerts triggered on symptoms).  
- **Proactive:** Anticipates future issues via trends, anomaly detection, capacity forecasts, synthetic tests, and early degradation signals (e.g., rising error ratio, growing queue).  
Effective systems blend both: reactive for immediate incidents; proactive for prevention & planning (autoscaling, capacity reservation).

## ⚙️ Section 2: Scenario - Answer
Priorities: protect user experience, reduce meantime-to-detect (MTTD), bootstrap minimal telemetry quickly.  
1. Telemetry phases: Phase 1 metrics (RED + infra USE), Phase 2 tracing, Phase 3 enriched logs (JSON).  
2. Instrument: shared middleware adds trace + request metrics; sidecar exporter (Prometheus/OpenTelemetry).  
3. Candidate SLOs: availability (>=99.9%), latency (p95 < 400ms), cache hit ratio (>90%), queue processing lag (<2s).  
4. Alerting: page only on sustained SLO burn (fast burn 5m, slow burn 1h). Non-paging: high error spike (warn), saturation >80% sustained 15m.  
5. Dashboards: (a) Aggregated request path; (b) Dependency health (DB/Redis/queue); (c) Release comparison (pre/post deploy latency).  
6. Quick wins: synthetic uptime check; add global trace ID to logs; implement structured error logging; set automatic scaling thresholds based on CPU & queue depth.  
7. MTTR reduction: correlate alerts with traces; add runbooks linked in dashboard.

## 🧩 Section 3: Problem-Solving - Answer
SLI Set (API Gateway):  
- Availability SLI: successful requests / total (HTTP 2xx/3xx minus known client errors).  
- Latency SLI: p95 request duration (exclude queued background endpoints).  
- Error SLI: 5xx rate proportion.  
Alert Policy (multi burn-rate):  
1. Fast burn: 14.4× error budget consumption over 5m → page (severe outage).  
2. Medium burn: 6× over 30m → page (ongoing degradation).  
3. Slow burn: 3× over 2h → non-paging ticket (capacity/planning).  
Latency page: p95 > 2× SLO for 15m and error budget burn >2× concurrently.  
Suppression: during deploy window with feature flag rollouts unless fast burn triggers. Auto-silence if <2 consecutive evaluation periods remain above threshold (flapping control).  
Alert budget governance: review weekly; remove any alert without resolved action past 3 weeks.

## 💡 Additional Questions (Time Permitting) - Answers
1. Cardinality: number of unique label combinations; high cardinality increases memory/query cost and hampers performance. Control via aggregation & label allowlists.  
2. Structured logs: machine-parseable (JSON) with consistent keys; improves queryability, correlation, and reduces parsing ambiguity.  
3. Distributed tracing: attaches spans across service boundaries; solves opaque latency and dependency mapping; aids root cause in microservices.  
4. Prevent alert fatigue: set SLO-based alerts, multi-window burn rates, deduplicate related symptoms, rotate maintenance reviews, include context (runbooks), enforce alert budget.

> Practice Idea: Instrument a demo app with OpenTelemetry SDK generating metrics + traces; push logs to a structured store and iterate on an SLO dashboard.
