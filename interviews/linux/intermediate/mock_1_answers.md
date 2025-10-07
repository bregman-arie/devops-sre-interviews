````markdown
# Linux - Intermediate Interview Mock #1 - Answer Key

> **Difficulty:** Intermediate  
> **Duration:** ~45 minutes  
> **Goal:** Assess understanding of process control, performance analysis, networking, storage, security primitives, and automation in Linux.

---

## 🧠 Section 1: Core Questions - Answers

### 1. How do Linux process states (e.g., `R`, `S`, `D`, `Z`, `T`) differ and how would you investigate a process stuck in `D` state?
**Answer:**
- `R` (Running/Runnable): On CPU or ready in run queue
- `S` (Interruptible Sleep): Waiting for an event (signal can wake)
- `D` (Uninterruptible Sleep): Usually waiting on blocking I/O (cannot be killed until syscall returns)
- `Z` (Zombie): Process finished, parent hasn't reaped (defunct)
- `T` (Stopped/Traced): Stopped by signal or being debugged

A process stuck in `D` often indicates slow/broken storage or unresponsive device. Investigation:
```bash
ps -o pid,stat,comm,wchan -p <PID>      # wchan shows sleeping kernel function
cat /proc/<PID>/stack                   # kernel stack trace (needs perms)
strace -p <PID>                         # might hang if truly blocked
lsof -p <PID>                           # file descriptors
iotop -p <PID>                          # I/O usage
# Kernel messages
dmesg | tail
```
If storage latency: check underlying block device (`iostat -x 1`, `pidstat -d 1`). If NFS: look for `nfs` waits, network issues. Cannot SIGKILL; must resolve underlying I/O.

### 2. Explain the difference between a hard link and a symbolic link at the inode level. When can a hard link NOT be created?
**Answer:**
- Hard link: Additional directory entry pointing to same inode. Increases link count. Same filesystem only. File persists until link count zero.
- Symbolic link: New inode containing path to target. Can span filesystems or point to missing target.
Cannot create hard link:
- Across different filesystems
- To a directory (without special privileges; restricted to avoid loops)
- To special pseudo files (some virtual FS constraints)
- If lacking permissions in parent directory

### 3. Describe how Linux capabilities differ from full root privileges. Give three practical examples.
**Answer:** Capabilities split root's all-powerful privilege into fine-grained units applied per process or file (ambient, inheritable, permitted, effective sets). Examples:
- `CAP_NET_BIND_SERVICE`: Bind to ports <1024 without full root
- `CAP_SYS_TIME`: Adjust system clock
- `CAP_SYS_ADMIN`: Broad admin ops (mount, pivot_root) – keep minimal
- `CAP_CHOWN`: Change file ownership
- `CAP_DAC_OVERRIDE`: Bypass file permission checks
Use case: Drop all but required caps in containers to reduce blast radius.

### 4. How does the Linux page cache improve performance, and how can you measure page cache effectiveness?
**Answer:** Page cache stores file-backed pages in memory to avoid disk reads. Improves throughput & latency. Measurement:
```bash
grep -E 'Cached|Buffers|MemFree' /proc/meminfo
vmstat 1                 # si/so (swap in/out), cache behavior
sar -B 1 5              # page faults
perf stat -e page-faults,minor-faults,major-faults <cmd>
iostat -x 1             # reduced read IO if cache effective
```
High major faults or high disk read rate for frequently reused data may indicate ineffective caching. `vfs_cache_pressure` tuning affects reclaim behavior.

### 5. Compare cgroups v1 vs v2 and explain how resource limits (CPU, memory) are enforced.
**Answer:**
- v1: Multiple hierarchies (controllers mounted separately); complexity & edge cases.
- v2: Unified hierarchy, consistent semantics, improved delegation, pressure stall info (PSI).
Enforcement:
- CPU: `cpu.max` (quota/period), `cpu.weight` (proportional share)
- Memory: `memory.max`, `memory.high` (throttling), `memory.min` (protection).
Kernel scheduler enforces CPU throttling; OOM killer targets processes exceeding memory controls.

### 6. What are namespaces and how do they underpin container isolation? List the main namespace types.
**Answer:** Namespaces wrap global system resources to provide isolated views per process group. Types:
- `PID` (process IDs)  
- `NET` (network stack: interfaces, routes, iptables)  
- `UTS` (hostname, domain)  
- `IPC` (System V, POSIX IPC)  
- `MNT` (mount points)  
- `USER` (UID/GID mapping)  
- `CGROUP` (cgroup root view)  
- `TIME` (monotonic & boottime)  
Provide isolation building blocks for containers when combined with cgroups + seccomp + capabilities.

### 7. How would you trace why a process is performing excessive disk I/O? Provide a structured approach and key tools.
**Answer:**
1. Identify candidate: `iotop -oPa`, `pidstat -d 1`.
2. Determine access pattern: `strace -p PID -e trace=read,write,openat` (sample), `lsof -p PID`.
3. Use eBPF/bcc tools:
```bash
biolatency 1
biosnoop
fileslower 1
opensnoop
```
4. Check filesystem & device stats: `iostat -x 1`, `df -h`, `mount` options.
5. Correlate with memory: Are page cache misses driving I/O? (`perf stat`, `vmstat`).
6. Look at contention: `dstat`, queue depth (`iostat r_await, w_await`).
7. Optimize: batching writes, adjusting readahead (`blockdev --getra /dev/sdX`), caching layer.

### 8. Explain the difference between ephemeral, buffered, and direct I/O in Linux.
**Answer:**
- Ephemeral (tmpfs / RAM-backed): Data in memory only (e.g., `/dev/shm`); disappears on reboot.
- Buffered I/O: Goes through page cache; kernel may delay writes (write-back). Improves small read/write performance.
- Direct I/O: Bypasses page cache (`O_DIRECT`); application manages alignment & caching. Useful for databases avoiding double buffering.
Trade-offs: Direct reduces cache pollution; buffered simpler. Choose based on workload access pattern.

---

## ⚙️ Section 2: Scenario - Answer

**Scenario Recap:** Java service has CPU spikes, high `%steal`, latency issues, network storage.

### Structured Triage
| Dimension | Evidence | Tools | Next Step |
|-----------|----------|-------|-----------|
| Host CPU Contention | High `%steal` means hypervisor preemption | `top`, `vmstat 1`, `sar -u`, cloud metrics | Migrate VM / choose larger dedicated instance / reduce noisy neighbors |
| Application Hot Paths | High user CPU, specific threads busy | `perf top`, `jstack`, `async-profiler` | Optimize code / reduce lock contention |
| I/O Latency | High iowait, storage RTT | `iostat -x 1`, `blktrace`, `biolatency` | Cache layer, provision faster storage |
| GC Pressure | Long GC pauses, heap churn | `jstat -gcutil`, GC logs | Tune heap sizes, use G1/ZGC, reduce allocations |
| Scheduling Delays | Run queue length > cores | `vmstat`, `pidstat -u 1` | Scale horizontally, rebalance threads |
| Network Bottleneck | Slow remote storage path | `nstat`, `ss -tmi`, storage metrics | Local SSD cache, warm page cache |

### Step Order
1. Capture snapshot (cpu/mem/iostat/perf) during spike.
2. Confirm steal/time vs user/sys share (`mpstat -P ALL 1`).
3. If `%steal` > ~5–10% sustained: address host placement first.
4. Profile JVM (`async-profiler -e cpu -d 30 <PID>`). Correlate with GC logs.
5. If I/O bound: measure read/write latency distribution (`biolatency 1`). Add local caching (e.g., page cache priming or Redis/FS cache layer).
6. Implement autoscaling based on latency or queue depth.

### Mitigations
- Reduce noisy neighbor: dedicated tenancy or larger instance type.
- Enable JVM ergonomics: proper heap (avoid frequent young gen GCs).
- Use connection pooling & async I/O to reduce blocking.
- Introduce metrics (exporter) for `cpu.steal`, GC pause, 99p latency -> alerting.

---

## 🧩 Section 3: Problem-Solving - Answer

**Task Recap:** Recurrent `/var` growth from core dumps + logs.

### (a) Detect Growth Early
- Deploy disk usage exporter (node_exporter) + alert at 70/80/90%
- Track top directories: periodic `du -x -d2 /var` to log/trend
- Use `systemd` service with `InaccessiblePaths=` for apps not needing broad write access

### (b) Enforce Retention
- Configure `logrotate` with size + time rotation + compression
- Journald: set `SystemMaxUse=` and `SystemMaxFileSize=`
- Remove old archives beyond N days: cron or systemd timer

### (c) Prevent Uncontrolled Core Dumps
```bash
# Limit size (shell)
ulimit -c 0                 # disable
# System-wide (/etc/security/limits.conf)
* soft core 0
# Configure core pattern to central dir with naming
echo '|/usr/lib/systemd/systemd-coredump %P %u %g %s %t %e' > /proc/sys/kernel/core_pattern
```
Or point to a restricted directory with quota.

### (d) On-Call Runbook
1. Check free space: `df -h /var`
2. Identify largest dirs: `sudo du -xhd1 /var | sort -h`
3. List new large files in last 24h: `find /var -xdev -type f -mtime -1 -size +50M -printf '%s %p\n' | sort -nr | head`
4. Core dump triage: `coredumpctl list | tail` -> analyze first then prune
5. Log sanity: ensure rotation working: `logrotate -d /etc/logrotate.conf`
6. If emergency: compress infrequently accessed large text logs instead of delete

### Prevent Recurrence
- Add CI check ensuring apps set `ulimit -c 0` unless debugging
- Instrument app logging level via config service (avoid debug in prod)
- Quota-managed partition for volatile dumps separate from `/var`

### Example Minimal logrotate Rule
```conf
/var/log/app/*.log {
  daily
  rotate 7
  size 50M
  compress
  missingok
  notifempty
  copytruncate
}
```

---

**Previous:** [Back to Questions](mock_1_questions.md)
````
