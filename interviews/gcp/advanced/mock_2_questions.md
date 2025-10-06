# Google Cloud Platform (GCP) - Advanced Interview Mock #2

> **Difficulty:** Advanced  
> **Duration:** ~60 minutes  
> **Goal:** Assess your expertise in designing resilient, highly available systems on GCP with advanced architectural patterns and enterprise-grade reliability requirements.

---

## 🧠 Section 1: Core Questions

1. Design a highly available and fault-tolerant architecture in GCP for a mission-critical application. Consider multi-region deployment, disaster recovery, and RTO/RPO requirements. [📖 Answer](mock_2_answers.md#1-design-a-highly-available-and-fault-tolerant-architecture-in-gcp-for-a-mission-critical-application-consider-multi-region-deployment-disaster-recovery-and-rtorpo-requirements)
2. Explain how you would implement chaos engineering practices in GCP to test system resilience. What tools and strategies would you use? [📖 Answer](mock_2_answers.md#2-explain-how-you-would-implement-chaos-engineering-practices-in-gcp-to-test-system-resilience-what-tools-and-strategies-would-you-use)
3. How do you design for graceful degradation and circuit breaker patterns in a microservices architecture on GKE? [📖 Answer](mock_2_answers.md#3-how-do-you-design-for-graceful-degradation-and-circuit-breaker-patterns-in-a-microservices-architecture-on-gke)
4. Implement a comprehensive observability strategy using Cloud Monitoring, Logging, and Trace. How do you handle alert fatigue and ensure actionable insights? [📖 Answer](mock_2_answers.md#4-implement-a-comprehensive-observability-strategy-using-cloud-monitoring-logging-and-trace-how-do-you-handle-alert-fatigue-and-ensure-actionable-insights)
5. Design a global database strategy using Cloud Spanner with read replicas and Cloud SQL for a financial services application with strict consistency requirements. [📖 Answer](mock_2_answers.md#5-design-a-global-database-strategy-using-cloud-spanner-with-read-replicas-and-cloud-sql-for-a-financial-services-application-with-strict-consistency-requirements)
6. How would you architect a blue-green deployment strategy for a stateful application with zero downtime requirements? [📖 Answer](mock_2_answers.md#6-how-would-you-architect-a-blue-green-deployment-strategy-for-a-stateful-application-with-zero-downtime-requirements)
7. Explain how to implement automated backup and disaster recovery testing for critical GCP workloads across multiple services. [📖 Answer](mock_2_answers.md#7-explain-how-to-implement-automated-backup-and-disaster-recovery-testing-for-critical-gcp-workloads-across-multiple-services)
8. Design a cross-region failover mechanism with automated traffic routing and health checking for a global e-commerce platform. [📖 Answer](mock_2_answers.md#8-design-a-cross-region-failover-mechanism-with-automated-traffic-routing-and-health-checking-for-a-global-e-commerce-platform)

---

## ⚙️ Section 2: Scenario

**Scenario:**  
You're architecting a real-time stock trading platform that must handle 1 million transactions per second during peak hours. The system serves global markets (NYSE, NASDAQ, LSE, TSE) and must comply with financial regulations in each region. The platform consists of:

- Order matching engine (ultra-low latency required)
- Market data distribution system
- Risk management and compliance engine
- User portfolio and account management
- Real-time analytics and reporting dashboard

**Critical Requirements:**
- **Availability**: 99.99% uptime (52 minutes downtime per year)
- **Latency**: <1ms for order processing, <10ms for market data
- **Scalability**: Handle 10x traffic spikes during market events
- **Compliance**: Data residency, audit trails, real-time fraud detection
- **Disaster Recovery**: RTO < 30 seconds, RPO < 1 second

Design a comprehensive GCP architecture that addresses:
1. Multi-region deployment strategy with active-active failover
2. Data consistency and replication across regions
3. Network optimization for ultra-low latency
4. Automated scaling and capacity management
5. Security and compliance framework
6. Monitoring and incident response procedures

[📖 Answer](mock_2_answers.md#️-section-2-scenario---answer)

---

## 🧩 Section 3: Problem-Solving

**Task:**  
Your production system experienced a cascading failure where a database connection pool exhaustion in one service caused a domino effect across 15 microservices, resulting in a 45-minute outage. The failure happened during Black Friday peak traffic.

**Failure Timeline:**
1. Payment service database connections maxed out (initial trigger)
2. Payment timeouts caused inventory service to retry aggressively
3. Inventory service overwhelmed the product catalog service
4. Frontend services started failing health checks
5. Load balancer removed all healthy instances
6. Complete system unavailability for 45 minutes

**Your Task:**
Design a comprehensive resilience strategy to prevent similar cascading failures:

1. **Circuit Breaker Implementation**: How would you implement circuit breakers across services?
2. **Bulkhead Pattern**: Design resource isolation strategies
3. **Timeout and Retry Policies**: Define appropriate timeout hierarchies
4. **Rate Limiting**: Implement service-level and global rate limiting
5. **Health Check Strategy**: Design meaningful health checks that prevent false positives
6. **Observability**: What metrics and alerts would detect this earlier?
7. **Testing**: How would you continuously test these resilience patterns?

**Bonus**: Implement this using specific GCP services and provide Infrastructure as Code examples.

[📖 Answer](mock_2_answers.md#-section-3-problem-solving---answer)

---

## 🎯 Section 4: Technical Deep Dive

**Question:**  
Explain how you would implement a sophisticated autoscaling strategy for a machine learning inference service that has the following characteristics:

- Model inference takes 2-5 seconds per request
- Traffic patterns are highly unpredictable (can spike 50x in minutes)
- Cold start time for new instances is 3-4 minutes (model loading)
- Cost optimization is critical (GPU instances are expensive)
- SLA requires 95th percentile response time < 8 seconds

Design an autoscaling solution that considers:
1. **Predictive Scaling**: Using historical patterns and ML-based forecasting
2. **Multi-tier Architecture**: Different instance types for different workloads
3. **Pre-warming Strategies**: Keeping warm instances for traffic spikes
4. **Cost Optimization**: Balancing performance vs. cost
5. **Monitoring and Tuning**: Continuous optimization of scaling policies

[📖 Answer](mock_2_answers.md#-section-4-technical-deep-dive---answer)

---

*Good luck! Remember to think about real-world trade-offs, operational complexity, and business requirements in your answers.*
