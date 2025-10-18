# Networking Mock Interview #1 - Advanced Level - Answer Key

## Question 1: TCP vs QUIC
**Answer:**
TCP + TLS 1.2/1.3: Separate transport and security layers. Requires TCP 3-way handshake (1 RTT) + TLS handshake (1–2 RTTs) before application data. Head-of-line (HoL) blocking at connection level.

QUIC: UDP-based multiplexed transport with integrated TLS 1.3 handshake; combines transport + crypto → 1 RTT for new, 0 RTT for resume. Independent streams avoid HoL blocking across streams; encryption of most headers increases ossification resistance.

Benefits: Lower connection setup latency, better mobile connection migration (connection ID), improved multiplexing, less ossification.
Trade-offs: More CPU overhead (user-space), middlebox compatibility challenges early on, complex implementation, limited kernel offload.

---
## Question 2: Congestion Control Algorithms
**Answer:**
Reno: AIMD; linear growth, halving on loss; conservative; underutilizes high BDP links.
CUBIC: Window growth based on cubic function of time since last loss; better utilization on high-speed long-distance paths; fairness concerns vs Reno.
BBR: Models bandwidth and RTT to maintain optimal operating point; can reduce queueing delay; potentially unfair to loss-based flows; sensitive to measurement accuracy.
Choose: High BDP → CUBIC/BBR; latency-sensitive low-loss path → BBR; fairness legacy environments → CUBIC.

---
## Question 3: DNS Resolution Path
**Answer:**
Stub resolver → local caching recursive → if cache miss: query root (.), receives NS for TLD → query TLD (e.g., .com) → receives delegation to authoritative → query authoritative → returns A/AAAA + TTL.
Negative caching: NXDOMAIN replies contain SOA; cached per SOA minimum TTL limiting repeated misses.
DNSSEC: Adds RRSIG, DNSKEY, DS chain; recursive validates signatures ensuring integrity; increases response size → EDNS0 usage.

---
## Question 4: Anycast and BGP
**Answer:**
Multiple sites announce same prefix via BGP → routers pick shortest AS path / local pref so client reaches nearest site. Attributes: LocalPref, AS-PATH length, MED, communities influence selection.
Benefits: Latency reduction, DDoS absorption, simple failover (withdraw route).
Risks: Debug complexity (who served request), uneven load distribution, partial outages, BGP hijack impact broad.

---
## Question 5: Mitigating BGP Route Hijacks
**Answer:**
Controls: RPKI origin validation, strict prefix filtering (only expected prefixes), max-prefix limits, BGP communities for propagation control, monitoring (BGPStream, route collectors), anomaly alerting.
Detection: Sudden path change, new origin AS, traffic shift metrics, traceroute divergence.
Response: Contact upstreams, announce more specific (if permissible), leverage RPKI signed ROAs, public comms.

---
## Question 6: Load Balancing Algorithms
**Answer:**
Round-robin: Even distribution; ignores server capacity.
Least-connections: Skews toward lightly loaded servers; good for long-lived sessions.
EWMA latency-based: Uses moving average; adapts to performance; complexity.
Consistent hashing: Session affinity; stable key mapping (CDN, caches); uneven partition risk (requires virtual nodes).
Weighted random: Probabilistic distribution by weight; simple scaling.
Choice: Caches → consistent hashing; stateless short calls → round-robin; variable session length → least-connections; latency-critical dynamic performance → EWMA.

---
## Question 7: TLS 1.3 Features
**Answer:**
Simplified handshake (fewer round trips); removed legacy ciphers (RSA key exchange, static DH); mandatory forward secrecy (ECDHE); 0-RTT for repeat connections (replay risk for non-idempotent requests). Encrypted handshake parts increase privacy.
0-RTT Risks: Replay of early data; mitigations: restrict to idempotent GET, anti-replay windows, reject on sensitive endpoints.

---
## Question 8: Packet Capture Triage
**Answer:**
Method:
1. Establish baseline: latency symptom (high RTT, retransmissions).
2. Inspect TCP handshake timings (SYN→SYN/ACK→ACK).
3. Look at RTT variance, retransmissions, duplicate ACKs.
4. Check window sizes, slow-start progression, out-of-order segments.
5. TLS handshake durations; certificate exchange delays.
6. Application layer gaps (HTTP request vs response start).
Tools: Wireshark I/O Graph, tcptrace, flow export (NetFlow/IPFIX). Key fields: Sequence/ACK numbers, timestamps, SACK blocks, window scaling, MSS.

---
## Question 9: Container / Pod Networking
**Answer:**
Calico Routed Mode: Each node advertises pod CIDRs via BGP to fabric; no encapsulation; efficient routing; needs network reachability.
Overlay (VXLAN): Pods get addresses within overlay; packets encapsulated; reduces need for fabric route changes; adds overhead & MTU constraints.
Operations: CNI adds veth pair, assigns IP from pool, programs iptables policies, updates BGP daemon or overlay mappings.

---
## Question 10: Service Mesh Traffic Flow
**Answer:**
Client app → local sidecar (Envoy) applies retries, circuit breaking, telemetry → mTLS to remote sidecar → remote sidecar enforces policy, distributes to service container.
Observability at sidecars: metrics (latency, success), tracing spans injection, access logs. Policy: RBAC, traffic shifting, fault injection.

---
## Question 11: Zero Trust Networking
**Answer:**
Principles: Verify explicitly (authn/authz each request), least privilege, assume breach, context-aware (user, device posture). Replace flat perimeter with identity agents + microsegmentation + continuous evaluation (short-lived tokens). Reduces lateral movement; complexity added via policy engines and identity distribution.

---
## Question 12: Multi-Region Failover Design
**Answer:**
GSLB uses health probes to update DNS answers (weighted/geographic). Anycast edges drop unreachable site route; traffic shifts to next closest. Pitfalls: Stale DNS due to high TTL, partial failure (brownout) not detected by probes, inconsistent session state (no global replication), negative cache delaying convergence.
Mitigations: Low TTL, multi-metric health, active-active replication, synthetic transaction probes, pre-warm other region capacity.

---
**Summary:** Advanced networking concepts: transport evolution (QUIC), congestion control selection, DNS/BGP mechanics, security controls (TLS 1.3, RPKI), observability, container networking, service mesh, zero trust, and global resilience design.

**Related:** [Questions](mock_1_questions.md) | [Intermediate Level](../intermediate/) | [Beginner Level](../beginner/) | [Expert Level](../expert/)
