````markdown
# Linux - Intermediate Interview Mock #2

> **Difficulty:** Intermediate  
> **Duration:** ~45 minutes  
> **Goal:** Assess deeper knowledge of service management, filesystems & storage, networking diagnostics, security hardening, performance observability, and troubleshooting methodology.

---

## 🧠 Section 1: Core Questions

1. Walk through the **Linux boot process** from firmware/UEFI to a running multi-user system. Where can it fail and how would you triage each stage? [📖 Answer](mock_2_answers.md#1-walk-through-the-linux-boot-process-from-firmwareuefi-to-a-running-multi-user-system-where-can-it-fail-and-how-would-you-triage-each-stage)
2. Compare **systemd unit types** (service, socket, timer, path, target, mount). Give a practical use-case for each. [📖 Answer](mock_2_answers.md#2-compare-systemd-unit-types-service-socket-timer-path-target-mount-give-a-practical-use-case-for-each)
3. Explain how **LVM** abstracts physical storage. Contrast linear, striped, and mirrored logical volumes; include when you'd choose each. [📖 Answer](mock_2_answers.md#3-explain-how-lvm-abstracts-physical-storage-contrast-linear-striped-and-mirrored-logical-volumes-include-when-youd-choose-each)
4. A filesystem shows high `%util` and long `await` in `iostat`, but application throughput is low. Provide a structured investigation plan. [📖 Answer](mock_2_answers.md#4-a-filesystem-shows-high-util-and-long-await-in-iostat-but-application-throughput-is-low-provide-a-structured-investigation-plan)
5. Contrast **SELinux** and **AppArmor** models. How would you quickly determine whether an access denial was caused by SELinux policy? [📖 Answer](mock_2_answers.md#5-contrast-selinux-and-apparmor-models-how-would-you-quickly-determine-whether-an-access-denial-was-caused-by-selinux-policy)
6. Explain **ephemeral port exhaustion**: causes, detection, and mitigation at OS + application layers. [📖 Answer](mock_2_answers.md#6-explain-ephemeral-port-exhaustion-causes-detection-and-mitigation-at-os--application-layers)
7. A process shows a slow **memory leak** over days but RSS is stable while `resident + swap` grows. How do you differentiate: heap leak vs file descriptor leak vs page cache pressure vs kernel slab growth? [📖 Answer](mock_2_answers.md#7-a-process-shows-a-slow-memory-leak-over-days-but-rss-is-stable-while-resident--swap-grows-how-do-you-differentiate-heap-leak-vs-file-descriptor-leak-vs-page-cache-pressure-vs-kernel-slab-growth)
8. Describe when you'd use each: `ss`, `ethtool`, `nstat`, `tcpdump`, `perf record -g`, `bcc/eBPF (tcpconnect, biolatency)`. Provide one insight each tool gives that's hard to get elsewhere. [📖 Answer](mock_2_answers.md#8-describe-when-youd-use-each-ss-ethtool-nstat-tcpdump-perf-record--g-bccebpf-tcpconnect-biolatency-provide-one-insight-each-tool-gives-thats-hard-to-get-elsewhere)

---

## ⚙️ Section 2: Scenario

**Scenario:**  
A Python API service occasionally returns timeouts. Symptoms during an event:
- `vmstat 1` shows spikes in `wa` (I/O wait)
- `dmesg` logs intermittent `blk_update_request: I/O error` on one device
- `iostat -x` shows high `svctm` and increasing `await` for `/dev/nvme1n1`
- Application logs show increased latency on endpoints involving reading large JSON templates from disk each request.

Outline a triage & mitigation plan that cleanly separates: (a) failing hardware vs transient kernel/device queue issues vs (b) inefficient application I/O pattern vs (c) OS caching / readahead tuning opportunity. Provide concrete commands, decision points, and short-term + long-term mitigations.

[📖 Answer](mock_2_answers.md#️-section-2-scenario---answer)

---

## 🧩 Section 3: Problem-Solving

**Task:**  
You must implement a **post-incident forensic data capture playbook** for production Linux hosts that: (1) minimizes performance impact, (2) preserves volatile evidence for 30 minutes after alert trigger, (3) avoids collecting sensitive payload data, (4) supports correlation across hosts. List the artifacts & commands, rotation / retention method, privacy safeguards, and an automation approach (systemd + timer / sidecar / eBPF).

[📖 Answer](mock_2_answers.md#-section-3-problem-solving---answer)

---

**Next:** [View Answers](mock_2_answers.md)
````
