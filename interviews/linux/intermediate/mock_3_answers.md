````markdown
# Linux - Intermediate Interview Mock #3 - Answer Key

> **Difficulty:** Intermediate  
> **Duration:** ~45 minutes  
> **Goal:** Evaluate skills in storage performance, kernel observability, networking resiliency, security controls, and systematic troubleshooting.

---

## 🧠 Section 1: Core Questions - Answers

### 1. Explain how Linux handles interrupts vs softirqs vs workqueues. How can excessive softirq time impact application latency?
- Hardware Interrupt (IRQ): CPU stops current execution, jumps to Interrupt Service Routine (ISR); should be minimal work.
- SoftIRQ: Deferred bottom-half processing (e.g., NET_RX, TIMER) executed either in return-from-interrupt context or by `ksoftirqd/N` kernel threads when load high.
- Workqueue: Generic deferral to kernel threads (process context) allowing sleeping operations.
Excessive softirq (esp NET_RX) consumes CPU prior to user tasks, raising run queue latency; `top` shows high `%si`; `perf top` shows `net_rx_action`. Mitigation: Tune NIC IRQ affinity (spread across cores), increase GRO/RPS, mitigate packet storm, apply backpressure higher in stack.

### 2. Describe EXT4 journaling modes (ordered, writeback, data=journal) and their performance/durability trade-offs.
- ordered (default): Data blocks flushed to disk before journal metadata commit. Prevents stale metadata pointing to uninitialized garbage; good balance.
- writeback: Metadata may commit before related data; risk of old data exposure after crash; slightly faster for some write-heavy workloads.
- data=journal: Both data + metadata journaled; strongest consistency, highest write amplification / latency.
Choice: Use `ordered` usually; `data=journal` for small, critical transactional workloads; avoid `writeback` unless acceptable risk & throughput critical.

### 3. How do control groups (cgroups) memory limits interact with the Linux OOM killer? Outline the decision path when a container hits its memory limit.
- Each cgroup tracks usage; when `memory.max` exceeded, kernel attempts reclaim within group (page cache reclaim, slab reclaim).  
- If cannot reclaim and still over limit → cgroup-level OOM triggered (v2).  
- OOM killer selects victim within cgroup using badness heuristic (RSS, oom_score_adj).  
- If global pressure and no reclaim across groups → global OOM.  
Decision: (1) Charge page → over limit? (2) Reclaim attempt (LRU). (3) Memory.high throttling (if set). (4) OOM kill in-group. (5) Emit OOM event to `memory.events`.  
Mitigation: Right-size limits, set `memory.min` for critical services, monitor `memory.events` counters.

### 4. Compare iptables / nftables pipeline concepts. Why might a migration to nftables improve performance or manageability?
- iptables: Multiple tables (filter, nat, mangle, raw, security); linear list evaluation per chain; duplication for IPv4/IPv6; expensive rule updates (global rebuild).
- nftables: Unified framework; rules organized in tables/chains with set/maps; bytecode VM executed by kernel; atomic incremental updates; native IPv4/IPv6 handling; dynamic sets (efficient for large IP lists). Performance gains from reduced duplication + maps vs linear scan.
Migration benefits: Lower latency with large rule sets, simpler maintenance, transactional updates, reduced code complexity.

### 5. Walk through diagnosing high system CPU (%sy) with low user CPU: list likely causes and targeted tools for narrowing (scheduler contention, context switching, lock contention, syscall storm).
1. Confirm distribution: `perf stat -a -- sleep 5` (context switches, migrations).  
2. Syscall storm: `strace -c -p <PID>`; `perf top` shows syscalls (e.g., `read`, `futex`).  
3. Lock contention: `perf lock record & report`; `echo 1 > /proc/lock_stat` (if enabled).  
4. Scheduler issues: High voluntary/involuntary context switches (`pidstat -w 1`); run queue length (`vmstat`, `uptime`).  
5. Interrupt pressure: `%si` or `%hi` in `mpstat -P ALL 1`; drill with `perf top` into IRQ handlers.  
6. TLB shootdowns / page faults: `perf stat -e tlb_flush,dTLB-load-misses`.  
Mitigations depend: batch syscalls, reduce lock scope, pin IRQs, adjust thread counts, use pooling.

### 6. Explain TCP retransmission vs TCP reordering vs application retries; how can you distinguish these in Linux metrics/logs without full packet capture?
- Retransmission: Sender resends unacked segment (loss). Metrics: `netstat -s` (segments retransmitted), increase in `TCPSegRetrans`.  
- Reordering: Packets arrive out of order; may trigger fast retransmit heuristics but not actual loss. Metrics: `nstat | grep TcpExtReord*`.  
- Application retry: App layer resends request (e.g., HTTP client timeout) even if TCP was fine; network counters may not show abnormal retransmits.  
Distinguish: High app latency + increased HTTP 499/504 but stable retransmit counters → app retry pattern. Retransmits + drops in `ss` cwnd shrink → actual transport loss. Reordering indicated by rising `TcpExtReord*` without drop in throughput.

### 7. How would you profile a sporadic thread lock contention issue inside a multi-threaded service using only standard + eBPF tools?
1. Detect spikes: Monitor latency + `pidstat -w` voluntary cs.  
2. Sample stack traces: `perf record -F 99 -g -p <PID> -- sleep 30`; analyze `perf report` for `futex_wait` hotspots.  
3. eBPF tools: `offcputime -p <PID> 10` (time blocked off-CPU), `funclatency` around suspected lock functions, `argdist` to count contended paths.  
4. Correlate with goroutine dump / thread dump (language-specific).  
5. Identify heavy lock: find symbol with high off-CPU + call-site frequency.  
Mitigation: Reduce critical section, sharding, switch to lock-free (atomic, RWLock), reduce goroutine fan-out.

### 8. Describe the difference between page faults, minor faults, major faults, and why a surge in major faults might not always mean memory pressure.
- Page fault: Access to a page not currently mapped.  
- Minor fault: Page present in memory (e.g., shared, copy-on-write) but not mapped to process; resolved without I/O.  
- Major fault: Requires disk I/O to load page.  
Surge not always memory pressure: Could be intentional cold start warming (loading large binary libs), sequential scan of new dataset (expected one-time cost), container just started pulling pages from image layers. Real pressure signs: swap activity, reclaim scans, elevated `kswapd` CPU, throttling via PSI (`cat /proc/pressure/memory`).

---

## ⚙️ Section 2: Scenario - Answer

### Categorization Strategy
| Category | Signals | Discriminators | Commands |
|----------|---------|----------------|----------|
| (a) Goroutine Oversubscription | High runnable goroutines vs GOMAXPROCS | `GODEBUG=schedtrace` output shows long run queues | `curl :6060/debug/pprof/goroutine?debug=2` (if enabled) |
| (b) Lock Contention | `perf top` shows `futex`, high voluntary cs | Off-CPU time in futex wait | `offcputime`, `perf lock` |
| (c) GC Pauses | Latency aligns with GC cycles | `GODEBUG=gctrace=1` lines show stop-the-world durations | Enable gctrace temporarily |
| (d) Kernel Contention / NUMA | Run queue length high on subset cores | `numastat`, imbalance across CPUs | `mpstat -P ALL 1`, `schedstat` |

### Stepwise Investigation
1. Snapshot scheduling: `GODEBUG=schedtrace=1000,scheddetail=1` (short window) → inspect run queue length vs GOMAXPROCS.  
2. Off-CPU profiling: `offcputime -p <PID> 30 > offcpu.txt` → aggregate futex waits by symbol.  
3. GC correlation: Enable `gctrace` in staging or short prod window; check if pauses coincide with latency spike timestamps.  
4. Heap profile: `go tool pprof http://host:6060/debug/pprof/heap` to ensure not memory churn causing GC pressure.  
5. NUMA / core imbalance: `taskset -pc <PID>`; verify threads distributed; if network IRQs pinned to same cores as runtime threads, reassign via `/proc/irq/*/smp_affinity`.

### Likely Root Cause Pattern
Batch ingestion increases goroutines performing JSON decode + compression; shared map (cache) causes serialized access; goroutines block on a mutex → futex waits → voluntary context switches → latency for request-serving goroutines.

### Immediate Mitigations
- Reduce parallelism: Cap ingestion worker pool size (bounded channel).  
- Increase cache granularity: Shard map by key hash to reduce lock scope.  
- Separate batch job to another node / schedule off-peak.  
- Temporarily raise GOMAXPROCS only if CPUs underutilized (monitor for diminished returns).  
- Pin noisy batch to cgroup with lower CPU weight.

### Longer-Term Improvements
- Replace global map with sync.Map or RWMutex + partition; evaluate lock-free ring buffer for queue.  
- Preparse / precompress JSON offline; store binary format.  
- Introduce backpressure: ingestion acknowledges only when downstream queue ready.  
- Enable pprof permanently with auth; integrate continuous profiles (Parca / Pyroscope).  
- Add SLO-based autoscaling tied to queue depth & latency, not raw CPU.

### Validation Metrics
- Reduced futex wait time (off-CPU report).  
- Lower voluntary context switches per second.  
- Stable p99 during ingestion window.  
- GC pause cumulative time < target (e.g., <1% of wall time).

---

## 🧩 Section 3: Problem-Solving - Answer

### Metrics & Commands
| Metric | Command Source | Collection Window |
|--------|----------------|-------------------|
| CPU run queue length | `vmstat 1 60` (avg), or `sar -q` | Nightly baseline | 
| Disk latency dist | eBPF `biolatency 60` | Nightly baseline |
| Network retransmits | `nstat -az | grep -E 'Tcp(Ext)?Retrans'` | Nightly |
| Network reordering | `nstat -az | grep TcpExtReord` | Nightly |
| Load vs cores | `uptime`, `mpstat 1 10` | Nightly |
| Deployment metadata | CI injects build hash | On deploy |

### Aggregation
- For each 60s sampling window compute: mean, P95, max for run queue; P50/P95/P99 for block latency; delta retransmissions vs prior night.  
- Use exponentially weighted moving average (EWMA) baseline or rolling 7-day median.

### Regression Detection (Simple)
```
if current_metric > baseline * 1.2 and abs(current_metric - baseline) > threshold_min:
    flag regression
```
Fallback to median absolute deviation (MAD) if variance high.

### Storage Layout
Directory `/var/lib/perf_guard/YYYY-MM-DD/summary.json`. Retain 30 days. Size small (<10KB/day). Optionally ship to central store.

### Alert Workflow
1. Nightly cron/systemd timer runs collector script post low-traffic window.  
2. Script loads last 7 baselines, computes stats & regression flags.  
3. On regression: emit structured log + push to webhook (e.g., Alertmanager) with deployment hash & suspected metric.  
4. CI/CD annotates deployments with link to last baseline vs current diff.

### Privacy / Overhead
- Only aggregates, no raw I/O traces or packet payload.  
- eBPF tool run limited to 60s; low overhead; fallback to iostat if eBPF unavailable.

### Sample JSON Record
```json
{
  "host": "app-07",
  "date": "2025-10-07",
  "deploy_hash": "abc1234",
  "cpu_runq": {"mean": 1.4, "p95": 2.8, "max": 3},
  "cpu_runq_regression": false,
  "disk_latency_ms": {"p50": 1.2, "p95": 3.9, "p99": 7.5, "regression": true, "baseline_p95": 2.9},
  "net": {"retrans_delta": 150, "retrans_regression": false, "reorder_delta": 2},
  "alerts": ["disk_latency_p95_regression"],
  "baseline_window_days": 7
}
```

### Minimal Collector Skeleton (Pseudo)
```bash
#!/usr/bin/env bash
set -euo pipefail
OUT=/var/lib/perf_guard/$(date +%F); mkdir -p "$OUT"
RUNQ=$(vmstat 1 60 | awk 'NR>2 {sum+=$1; if($1>max)max=$1; a[NR]=$1} END {n=NR-2; asort(a); p95=a[int(n*0.95)]; print sum/n, p95, max}')
# ... gather others; compute baseline via jq ...
```

### Benefits
- Fast detection of performance drift linked to releases.  
- Low storage footprint.  
- Extensible (add memory PSI later).

---

**Previous:** [Back to Questions](mock_3_questions.md)
````
