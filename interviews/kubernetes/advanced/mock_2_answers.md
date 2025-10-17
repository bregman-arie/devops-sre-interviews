# Kubernetes - Advanced Interview Mock #2 - Answer Key

> **Difficulty:** Advanced  
> **Focus:** Systematic ingress / networking incident isolation.

---

## You’ve deployed an Nginx ingress controller, but some requests hang indefinitely while others succeed. What steps would you take to debug and isolate the cause?

### 1. Characterize & Scope the Symptom
1. Identify pattern: specific paths, hosts, methods, payload sizes, client geos, or only long‑lived (e.g., websockets / chunked) requests?  
2. Compare success vs hang request headers (Accept-Encoding, Connection, Upgrade) and response codes when they do finish.  
3. Determine layer of stall: DNS resolution? TCP handshake? TLS handshake? First byte? Mid‑stream? Use `curl -v --trace-time` and browser HAR.  

### 2. Quick Divergence Tests
| Layer | Test | Purpose |
|-------|------|---------|
| Client → Service | `curl --resolve host:443:<ingress_lb_ip>` | Bypass DNS path issues |
| Direct Pod | `kubectl port-forward` or NodePort access | Bypass ingress entirely |
| Service ClusterIP | `kubectl exec busybox -- curl -v http://svc:port` | Validate kube-proxy / endpoints |
| Nginx Pod localhost upstream | `kubectl exec <nginx-pod> -- curl -v http://127.0.0.1:<upstream-port>` | Distinguish ingress vs upstream app |

### 3. Ingress Controller & Config Inspection
1. Check Nginx ingress controller logs for `upstream timed out (110: Connection timed out)` vs `client timed out` vs `upstream prematurely closed`.  
2. Run: `kubectl exec <nginx-pod> -- nginx -T | grep -A3 <problematic-host-or-path>` to inspect generated server/location blocks (mismatched regex precedence, missing `proxy_read_timeout`).  
3. List ingress objects to detect overlapping host/path rules; confirm only one matches: `kubectl get ingress -A`.  
4. Verify annotations: timeouts (`nginx.ingress.kubernetes.io/proxy-read-timeout`), buffering, websocket (`proxy-http-version`), large body (`proxy-body-size`). Misconfiguration may affect only certain routes.  
5. Inspect ConfigMap used by controller (global timeouts, keepalive).  

### 4. Service & Endpoint Consistency
1. `kubectl describe svc <svc>`: confirm port names, types, and sessionAffinity not skewing traffic.  
2. `kubectl get endpoints <svc> -o wide`: ensure all backend pods have Ready endpoints; partial readiness can make only some connections hang (SYN sent to draining pod / failing IP).  
3. If using readiness gates or sidecars (e.g., service mesh), verify they report Ready quickly; mis-synced readiness leads to traffic to initializing pods.  

### 5. Network Path & kube-proxy Mode
1. Determine kube-proxy mode: iptables vs IPVS. Inconsistent programming or stale rules can blackhole flows.  
2. For IPVS: `ipvsadm -Ln` inside node (daemonset toolbox) to confirm endpoints sync.  
3. Check CNI plugin logs (Calico, Cilium, etc.); policy drops may affect only larger or long-lived connections.  
4. Validate MTU: hanging often from PMTU blackhole. Compare `curl --max-time 5 -o /dev/null --limit-rate 1M` vs normal; capture with `tcpdump` on ingress pod + node interface to see retransmissions.  

### 6. TLS & Connection Lifecycle
1. For HTTPS, run `openssl s_client -connect host:443 -servername host -msg -tlsextdebug` to see if stall is pre or post handshake.  
2. Check if HTTP/2 only requests hang: disable via annotation `nginx.ingress.kubernetes.io/backend-protocol` tests.  
3. Look at keepalive / connection reuse; exhaustion of upstream keepalive pool can manifest as partial hangs.  

### 7. Application Behavior
1. Examine upstream pod logs for slow handlers, blocking I/O, thread pool exhaustion; correlate timestamps with stalled request IDs (add request ID header).  
2. Confirm liveness/readiness probes not causing restarts mid-stream (aborted connections).  
3. Large bodies or streaming responses: ensure `proxy-buffering` setting appropriate; buffering mismatch can cause partial flush waits.  

### 8. Resource Pressure & Node Signals
1. Check node dmesg / kubelet logs for OOMKills; if upstream pods OOM mid-response, client hangs awaiting data.  
2. Verify no ephemeral storage pressure (Nginx buffering on disk).  
3. Confirm CPU throttling not severe: `kubectl top pods` + `cgroupfs` metrics; heavy throttling can delay Nginx event loop.  

### 9. Observability Artifacts
1. Enable Nginx access logs (if disabled) with upstream response time: `"$request" $status $request_time $upstream_response_time`.  
2. Add temporary span / trace headers to watch path in distributed tracing backend (OpenTelemetry).  
3. Packet capture sample: `kubectl exec <nginx-pod> -- tcpdump -nn -s96 -c50 tcp port 80` to observe half-open or retransmitted sessions.  

### 10. Narrowing Hypotheses (Examples)
| Symptom | Likely Causes | Action |
|---------|---------------|--------|
| Only large uploads hang | body size limit, client body temp storage issue | Set `proxy-body-size`, verify disk, disable buffering for streams |
| WebSockets hang, REST fine | Missing `Upgrade` pass or timeout | Add `nginx.ingress.kubernetes.io/proxy-read-timeout: "3600"` |
| Random subset hang | One backend pod unhealthy / network policy partial | Remove pod from Service; inspect readiness; check `kubectl describe endpoints` |
| Only HTTP/2 | ALPN / flow control bug | Force HTTP/1.1 for test; upgrade controller version |
| Only after ~60s | `proxy_read_timeout` too low or upstream never responds | Increase timeout; profile app latency |

### 11. Remediation & Hardening
1. Set explicit sane timeouts: connect/read/send.  
2. Enforce readiness gating; use probes reflective of real dependency health.  
3. Add NetworkPolicies permitting only required paths; detect drops early via flow logs.  
4. Implement structured access + error logging; export metrics (`nginx_ingress_controller_request_duration_seconds_bucket`).  
5. Version pin Nginx ingress image; audit change history for regression.  

### 12. Postmortem Checklist
Collect timeline, affected % traffic, root cause classification (config / infra / app), detection gap, and preventive actions (dashboards, runbooks, synthetic tests).  

**Outcome:** A systematic top-down (L7→L4→cluster internals) approach isolates whether the hang originates in ingress configuration, upstream application behavior, network path inconsistency, or resource pressure—minimizing guesswork.

---
