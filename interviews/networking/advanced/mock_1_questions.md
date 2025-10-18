# Networking Mock Interview #1 - Advanced Level

## Question 1: TCP vs QUIC
**Question:** Contrast the TCP + TLS handshake with QUIC's connection establishment. What performance and operational benefits does QUIC provide and what trade-offs exist?
[📖 Answer](mock_1_answers.md#question-1-tcp-vs-quic)

---
## Question 2: Congestion Control Algorithms
**Question:** Explain Reno, CUBIC, and BBR congestion control differences and how they affect throughput and latency under varying network conditions.
[📖 Answer](mock_1_answers.md#question-2-congestion-control-algorithms)

---
## Question 3: DNS Resolution Path
**Question:** Walk through a full recursive DNS resolution (stub → resolver → root → TLD → authoritative). Where do negative caching and DNSSEC fit?
[📖 Answer](mock_1_answers.md#question-3-dns-resolution-path)

---
## Question 4: Anycast and BGP
**Question:** How does Anycast work for global services (e.g., DNS, CDN)? What BGP attributes influence route selection and what are operational risks?
[📖 Answer](mock_1_answers.md#question-4-anycast-and-bgp)

---
## Question 5: Mitigating BGP Route Hijacks
**Question:** List technical controls to reduce impact of BGP hijacks (RPKI, prefix filtering, max-prefix limits, monitoring). How do you detect and respond?
[📖 Answer](mock_1_answers.md#question-5-mitigating-bgp-route-hijacks)

---
## Question 6: Load Balancing Algorithms
**Question:** Compare round-robin, least-connections, EWMA latency-based, consistent hashing, and weighted random. When choose each?
[📖 Answer](mock_1_answers.md#question-6-load-balancing-algorithms)

---
## Question 7: TLS 1.3 Features
**Question:** What improvements did TLS 1.3 introduce over 1.2 (handshake steps, cipher suites, PFS)? How do 0-RTT risks manifest?
[📖 Answer](mock_1_answers.md#question-7-tls-13-features)

---
## Question 8: Packet Capture Triage
**Question:** Outline a methodical approach to diagnosing high latency using packet captures and flow metrics. Which key fields do you inspect first?
[📖 Answer](mock_1_answers.md#question-8-packet-capture-triage)

---
## Question 9: Container / Pod Networking
**Question:** Explain how a typical CNI plugin (e.g., Calico) programs routes for pods. Contrast overlay vs routed (BGP) modes.
[📖 Answer](mock_1_answers.md#question-9-container--pod-networking)

---
## Question 10: Service Mesh Traffic Flow
**Question:** Describe request path with sidecars (client → envoy → mTLS → remote envoy → service). Where are retries, circuit breaking, and telemetry applied?
[📖 Answer](mock_1_answers.md#question-10-service-mesh-traffic-flow)

---
## Question 11: Zero Trust Networking
**Question:** Define core Zero Trust principles (identity, context, continuous verification). How do they alter traditional perimeter design?
[📖 Answer](mock_1_answers.md#question-11-zero-trust-networking)

---
## Question 12: Multi-Region Failover Design
**Question:** How do GSLB/DNS steering, health probes, and anycast interact to enable fast failover? What pitfalls cause brownouts instead of clean shifts?
[📖 Answer](mock_1_answers.md#question-12-multi-region-failover-design)

---
**Next:** [View Answers](mock_1_answers.md)

**Time Limit:** 70 minutes  
**Level:** Advanced  
**Focus Areas:** Transport protocols, DNS/BGP, security, observability, container & service mesh networking, resilience

**Related:** [Beginner Level](../beginner/) | [Intermediate Level](../intermediate/) | [Expert Level](../expert/)
