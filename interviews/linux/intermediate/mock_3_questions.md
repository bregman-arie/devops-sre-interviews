````markdown
# Linux - Intermediate Interview Mock #3

> **Difficulty:** Intermediate  
> **Duration:** ~45 minutes  
> **Goal:** Evaluate skills in storage performance, kernel observability, networking resiliency, security controls, and systematic troubleshooting.

---

## 🧠 Section 1: Core Questions

1. Explain how Linux handles **interrupts vs softirqs vs workqueues**. How can excessive softirq time impact application latency? [📖 Answer](mock_3_answers.md#1-explain-how-linux-handles-interrupts-vs-softirqs-vs-workqueues-how-can-excessive-softirq-time-impact-application-latency)
2. Describe **EXT4 journaling modes** (`ordered`, `writeback`, `data=journal`) and their performance/durability trade-offs. [📖 Answer](mock_3_answers.md#2-describe-ext4-journaling-modes-ordered-writeback-datajournal-and-their-performancedurability-trade-offs)
3. How do **control groups (cgroups) memory limits** interact with the Linux OOM killer? Outline the decision path when a container hits its memory limit. [📖 Answer](mock_3_answers.md#3-how-do-control-groups-cgroups-memory-limits-interact-with-the-linux-oom-killer-outline-the-decision-path-when-a-container-hits-its-memory-limit)
4. Compare **iptables / nftables** pipeline concepts. Why might a migration to nftables improve performance or manageability? [📖 Answer](mock_3_answers.md#4-compare-iptables--nftables-pipeline-concepts-why-might-a-migration-to-nftables-improve-performance-or-manageability)
5. Walk through diagnosing **high system CPU (%sy)** with low user CPU: list likely causes and targeted tools for narrowing (scheduler contention, context switching, lock contention, syscall storm). [📖 Answer](mock_3_answers.md#5-walk-through-diagnosing-high-system-cpu-sy-with-low-user-cpu-list-likely-causes-and-targeted-tools-for-narrowing-scheduler-contention-context-switching-lock-contention-syscall-storm)
6. Explain **TCP retransmission vs TCP reordering vs application retries**; how can you distinguish these in Linux metrics/logs without full packet capture? [📖 Answer](mock_3_answers.md#6-explain-tcp-retransmission-vs-tcp-reordering-vs-application-retries-how-can-you-distinguish-these-in-linux-metricslogs-without-full-packet-capture)
7. How would you profile a **sporadic thread lock contention** issue inside a multi-threaded service using only standard + eBPF tools? [📖 Answer](mock_3_answers.md#7-how-would-you-profile-a-sporadic-thread-lock-contention-issue-inside-a-multi-threaded-service-using-only-standard--ebpf-tools)
8. Describe the difference between **page faults, minor faults, major faults**, and why a surge in major faults might not always mean memory pressure. [📖 Answer](mock_3_answers.md#8-describe-the-difference-between-page-faults-minor-faults-major-faults-and-why-a-surge-in-major-faults-might-not-always-mean-memory-pressure)

---

## ⚙️ Section 2: Scenario

**Scenario:**  
A containerized Go API intermittently spikes tail latency (p99) during nightly batch ingestion. Symptoms:
- Node shows rising `context switches/sec` and elevated `%sys` while `%idle` remains >30%
- `pidstat -w` shows high voluntary context switches for the Go process
- Disk (`iostat -x`) normal; network stable
- `perf top` during spike indicates heavy time in `futex` and `sched` functions
- Batch job loads many JSON blobs from S3, decompresses, and writes processed artifacts to local disk then to a queue

Provide a structured approach to isolate the root cause categories: (a) goroutine oversubscription / scheduler thrash, (b) lock contention on shared maps, (c) GC pause amplification, (d) kernel-level contention (run queue / NUMA). Recommend immediate mitigations and longer-term code / runtime improvements.

[📖 Answer](mock_3_answers.md#️-section-2-scenario---answer)

---

## 🧩 Section 3: Problem-Solving

**Task:**  
Design a **lightweight recurring performance regression guard** for Linux application hosts that (1) baseline key resource metrics nightly, (2) detects statistically significant regressions (>20% deviation) in CPU latency (run queue), disk latency, and network retransmissions, (3) stores only compact summaries (no raw samples), (4) integrates with CI/CD deployment metadata to flag suspect releases.

Describe: metrics collected + commands, aggregation technique, statistical method (simple), storage layout, and alert workflow. Provide an example compact JSON record.

[📖 Answer](mock_3_answers.md#-section-3-problem-solving---answer)

---

**Next:** [View Answers](mock_3_answers.md)
````
