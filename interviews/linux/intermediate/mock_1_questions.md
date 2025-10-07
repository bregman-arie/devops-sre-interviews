````markdown
# Linux - Intermediate Interview Mock #1

> **Difficulty:** Intermediate  
> **Duration:** ~45 minutes  
> **Goal:** Assess understanding of process control, performance analysis, networking, storage, security primitives, and automation in Linux.

---

## 🧠 Section 1: Core Questions

1. How do Linux process states (e.g., `R`, `S`, `D`, `Z`, `T`) differ and how would you investigate a process stuck in `D` state? [📖 Answer](mock_1_answers.md#1-how-do-linux-process-states-eg-r-s-d-z-t-differ-and-how-would-you-investigate-a-process-stuck-in-d-state)
2. Explain the difference between a **hard link** and a **symbolic link** at the inode level. When can a hard link NOT be created? [📖 Answer](mock_1_answers.md#2-explain-the-difference-between-a-hard-link-and-a-symbolic-link-at-the-inode-level-when-can-a-hard-link-not-be-created)
3. Describe how **Linux capabilities** differ from full root privileges. Give three practical examples. [📖 Answer](mock_1_answers.md#3-describe-how-linux-capabilities-differ-from-full-root-privileges-give-three-practical-examples)
4. How does the Linux **page cache** improve performance, and how can you measure page cache effectiveness? [📖 Answer](mock_1_answers.md#4-how-does-the-linux-page-cache-improve-performance-and-how-can-you-measure-page-cache-effectiveness)
5. Compare **cgroups v1 vs v2** and explain how resource limits (CPU, memory) are enforced. [📖 Answer](mock_1_answers.md#5-compare-cgroups-v1-vs-v2-and-explain-how-resource-limits-cpu-memory-are-enforced)
6. What are **namespaces** and how do they underpin container isolation? List the main namespace types. [📖 Answer](mock_1_answers.md#6-what-are-namespaces-and-how-do-they-underpin-container-isolation-list-the-main-namespace-types)
7. How would you trace why a process is performing excessive disk I/O? Provide a structured approach and key tools. [📖 Answer](mock_1_answers.md#7-how-would-you-trace-why-a-process-is-performing-excessive-disk-io-provide-a-structured-approach-and-key-tools)
8. Explain the difference between **ephemeral**, **buffered**, and **direct** I/O in Linux. [📖 Answer](mock_1_answers.md#8-explain-the-difference-between-ephemeral-buffered-and-direct-io-in-linux)

---

## ⚙️ Section 2: Scenario

**Scenario:**  
A Java service intermittently spikes CPU and latency. `top` shows high `%steal` on a VM. At the same time, users report slow API responses and increased 95th percentile latency. Disk is on network-attached storage. Provide a structured triage plan that separates: host contention vs application inefficiency vs I/O bottleneck vs GC tuning issues.

[📖 Answer](mock_1_answers.md#️-section-2-scenario---answer)

---

## 🧩 Section 3: Problem-Solving

**Task:**  
You receive a report: "Some deployments slowly fill `/var` with large core dumps and logs. After cleanup, it happens again." Design a preventative approach that (a) detects growth before critical levels, (b) enforces retention, (c) prevents uncontrolled core file generation, (d) documents remediation steps for on-call.

[📖 Answer](mock_1_answers.md#-section-3-problem-solving---answer)

---

**Next:** [View Answers](mock_1_answers.md)
````
