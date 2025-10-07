````markdown
# Linux - Intermediate Interview Mock #2 - Answer Key

> **Difficulty:** Intermediate  
> **Duration:** ~45 minutes  
> **Goal:** Assess deeper knowledge of service management, filesystems & storage, networking diagnostics, security hardening, performance observability, and troubleshooting methodology.

---

## 🧠 Section 1: Core Questions - Answers

### 1. Walk through the Linux boot process from firmware/UEFI to a running multi-user system. Where can it fail and how would you triage each stage?
**Stages:**
1. Firmware/UEFI initializes hardware, runs POST → loads bootloader from EFI partition (or MBR in legacy).  
2. Bootloader (GRUB/systemd-boot) loads kernel + initramfs, passes cmdline.  
3. Kernel decompresses, sets up paging, detects hardware, mounts early root (initramfs).  
4. `init` (PID 1; systemd) launched → mounts real root, switches root.  
5. systemd targets & units started (basic.target → multi-user.target → graphical.target).  
6. Services + sockets + timers activated → login shells or display manager.

**Failure Points & Triage:**
- Firmware: No POST / beep codes → hardware diag (vendor tools, IPMI logs).
- Bootloader: Stuck at GRUB menu → inspect config (`grub.cfg`), kernel path, root UUID mismatch.
- Kernel panic early: Use previous known-good entry; add `systemd.unit=rescue.target` or `rd.break` for dracut shell; check missing drivers for storage (initramfs rebuild via `dracut -f`).
- Root FS mount fail: Check UUID vs `lsblk -f`; emergency shell; verify LVM activation (`lvm vgchange -ay`).
- systemd service hang: `systemctl list-jobs`, `systemd-analyze blame | critical-chain`.
- Slow boot: `systemd-analyze time`, `systemd-analyze plot > boot.svg`.

### 2. Compare systemd unit types (service, socket, timer, path, target, mount). Give a practical use-case for each.
- Service: Run a daemon or one-shot task. Example: `nginx.service`.
- Socket: Socket activation to defer service start until connection. Example: `docker.socket`.
- Timer: Cron replacement with calendar/monotonic triggers. Example: log cleanup daily.
- Path: Trigger service when file/dir changes. Example: re-index on new file arrival.
- Target: Logical sync point/grouping (like runlevel). Example: `multi-user.target`.
- Mount: Managed mount point; ensures ordering, auto-restart if lost. Example: network share mount with dependencies.

### 3. Explain how LVM abstracts physical storage. Contrast linear, striped, and mirrored logical volumes; include when you'd choose each.
- Physical Volumes (PVs) → Volume Group (VG) (pool) → Logical Volumes (LVs). Extents map logical to physical.
- Linear: Concatenate extents sequentially. Use: simplicity, heterogeneous disks, predictable capacity growth.
- Striped: Distribute extents round-robin (RAID0-like). Use: throughput for sequential or parallel I/O (databases, logs). Risk: one disk failure loses LV.
- Mirrored: Duplicate extents (RAID1-like). Use: redundancy for critical data. Higher write amplification.
- Can layer with RAID hardware or dm-crypt. Choose based on performance vs resilience tradeoffs.

### 4. A filesystem shows high %util and long await in iostat, but application throughput is low. Provide a structured investigation plan.
1. Confirm device saturation: `iostat -x 1` (queue `avgqu-sz`, `svctm`, `await`).  
2. Check I/O size & pattern: `blktrace` / bcc `biosnoop`, `biolatency`.  
3. Distinguish read vs write pressure: `pidstat -d 1`, `iotop -oPa`.  
4. Filesystem overhead: Check mount options (`mount | grep <fs>`), metadata ops (`perf stat -e ext4:*` or `filetop`).  
5. Queue depth tuning: `cat /sys/block/<dev>/queue/nr_requests`, scheduler (`mq-deadline`, `none`, `bfq`).  
6. Latency source: Cloud EBS credits? (`cloud vendor metrics`).  
7. Eliminate page cache churn: `vmstat 1`, major faults, readahead (`blockdev --getra`).  
8. Mitigations: Increase instance/storage class, stripe, cache layer (Redis), batch small I/O, enlarge readahead for sequential reads.

### 5. Contrast SELinux and AppArmor models. How would you quickly determine whether an access denial was caused by SELinux policy?
- SELinux: Label-based (contexts: user:role:type:level). Policy allows explicit type interactions. Mandatory access control via TE + MLS/MCS.
- AppArmor: Path-based profiles; simpler, less granular with rename/hardlink edge cases.
- Detection: `ausearch -m avc -ts recent` or `journalctl -t setroubleshoot`; look for `AVC denied`. Or `dmesg | grep -i denied`. Use `sealert -a /var/log/audit/audit.log` for explanation. `getenforce` to see mode.

### 6. Explain ephemeral port exhaustion: causes, detection, and mitigation at OS + application layers.
- Causes: Too many outbound connections in TIME_WAIT/CLOSE_WAIT; short-lived bursts; low ephemeral range.
- Detection: `ss -s`, `ss -ant '( sport >= 32768 )' | wc -l`, high TIME_WAIT counts; failures: `EADDRNOTAVAIL`.
- OS Mitigation: Enlarge range (`sysctl net.ipv4.ip_local_port_range="2000 65000"`), enable reuse (`net.ipv4.tcp_tw_reuse=1`), tune `tcp_fin_timeout` cautiously.
- App Mitigation: Connection pooling, keep-alive reuse, async multiplexing (HTTP/2, gRPC), reduce churn.
- Monitoring: Alert when active ephemeral ports > 80% of range.

### 7. A process shows a slow memory leak over days but RSS is stable while resident + swap grows. How do you differentiate: heap leak vs file descriptor leak vs page cache pressure vs kernel slab growth?
- Heap leak: RSS grows; confirm via `pmap`, `smem`, `malloc` profiling (`jemalloc heap profile`). Here RSS stable → unlikely.
- FD leak: `ls /proc/<PID>/fd | wc -l`; monitor growth; large number of mapped files may raise page cache usage indirectly.
- Page cache pressure: System free drops, Cached rises, process RSS steady. Check `vmstat`, `/proc/meminfo`, file access pattern. Dropping caches temporarily (`echo 1 > /proc/sys/vm/drop_caches`) (diagnostic only).
- Kernel slab growth: `slabtop` shows rising slabs (e.g., dentry, inode). Memory used not attributed to process RSS. Investigate workloads causing many fs objects. Use `perf kmem` for advanced tracing.
- Also check cgroup memory stats (`memory.current`, `memory.stat`).

### 8. Describe when you'd use each: ss, ethtool, nstat, tcpdump, perf record -g, bcc/eBPF (tcpconnect, biolatency). Provide one insight each tool gives that's hard to get elsewhere.
- `ss`: Fast socket state summary; per-state counts (TIME_WAIT surge) → quick saturation view.
- `ethtool`: NIC offload, driver, ring sizes; detect disabled GRO causing small packet flood.
- `nstat`: Kernel network stack counters; reveals retransmits vs out-of-order without full pcaps.
- `tcpdump`: Precise packet timing & payload (if allowed); handshake anomalies & MTU issues.
- `perf record -g`: CPU sample call stacks; identify kernel vs user hotspots causing network stalls.
- `bcc/eBPF tcpconnect`: Live attach to show new TCP connections (app churn) without packet capture.
- `bcc/eBPF biolatency`: Block device latency distribution (P50..P99) quickly surfaces long-tail spikes.

---

## ⚙️ Section 2: Scenario - Answer

### Goal
Disambiguate hardware/storage fault vs application inefficiency vs OS cache tuning.

### Step 1: Capture Snapshot During Event
```bash
vmstat 1 5
iostat -x 1 5
journalctl -k -n 200 --no-pager
smartctl -a /dev/nvme1n1   # if permitted
nvme smart-log /dev/nvme1n1
pidstat -d 1 5
sudo opensnoop -n python  # if eBPF tools installed
```

### Step 2: Classify
| Signal | Hardware Fault? | App Pattern? | Cache/Readahead? |
|--------|-----------------|--------------|------------------|
| Kernel `I/O error` messages | Strong | Weak | Weak |
| Increasing media errors (SMART) | Strong | Weak | Weak |
| High small sync reads of same files | Weak | Strong | Strong |
| Low page cache hit ratio | Weak | Moderate | Strong |
| Latency improves after cache warm | Weak | Strong | Strong |

### Step 3: Hardware vs Transient Queue
- Check SMART/NVMe log for media errors, temperature throttling.  
- If errors persistent & counter increments → cordon host, fail over, replace volume.  
- If transient: Inspect queue depth `cat /sys/block/nvme1n1/queue/nr_requests` and scheduler (`none` for NVMe). Potential PCIe saturation: `lspci -vv` ASPM errors.

### Step 4: Application Inefficiency
- Confirm repeated full file reads each request: trace open/read count via `strace -f -tt -e openat,read -p <PID>` sample.  
- Mitigation: Preload templates into memory (singleton), enable application-level caching, reduce JSON size (precompiled objects), consider memcached/Redis.

### Step 5: OS Cache / Readahead
- Check readahead: `blockdev --getra /dev/nvme1n1`; adjust: `blockdev --setra 4096 /dev/nvme1n1` for large sequential template scans.  
- Monitor page cache effectiveness: `perf stat -e page-faults,minor-faults,major-faults -p <PID>`; high major faults → cache misses.  
- If many small random reads: consider aggregating files, using mmap, or keeping hot data in tmpfs.

### Short-Term Mitigations
- If hardware errors: switch traffic off host, snapshot & replace disk.
- Enable app layer caching of templates.
- Increase readahead experimentally; measure 95p latency delta.
- Throttle heavy batch jobs hitting same device concurrently.

### Long-Term Mitigations
- Add synthetic I/O health checks (fio read probe) + SMART monitoring + alerting on media error delta.
- Implement warm-cache init step (touch all templates at deploy).
- Observability: Export block latency histogram (eBPF) & page cache hit metrics.
- Architectural: Move static templates to object storage + CDN; load once at boot.

### Decision Points
- If SMART critical warnings: replace.  
- If no hardware faults & caching removes latency → fix app.  
- If latency persists even with cached working set → deeper kernel/device path analysis (`perf record -g`, `blkstat`).

---

## 🧩 Section 3: Problem-Solving - Answer

### Objectives
Low overhead, privacy-aware, correlated forensic capture for 30 min window.

### Artifacts to Collect (Sampled / Bounded)
| Category | Command / Source | Frequency | Notes |
|----------|------------------|-----------|-------|
| CPU & Load | `uptime`, `pidstat -u 1 10` | On trigger | Snapshot threads |
| Memory | `vmstat 1 10`, `smem -rt` | On trigger | Top consumers |
| I/O | `iostat -x 1 10`, `biolatency 5` | On trigger | Distribution not raw data |
| Network | `ss -tni`, `nstat` diff | Start + 5m | Counts not payload |
| Processes | `ps -eo pid,ppid,comm,%cpu,%mem,args --sort=-%cpu | head -50` | Every 5m | Truncate args |
| Open Files | `lsof -nP -l -p <hot PIDs>` | Optional | Filter by whitelisted procs |
| Kernel Events | `dmesg --ctime | tail -200` | On trigger | Ring buffer tail |
| Slabs | `slabtop -o -s c -n 1` | 10m | Growth patterns |
| Cgroups | `/sys/fs/cgroup/*/memory.current` | 1m | Container contexts |

### Privacy Safeguards
- Redact command args with regex (e.g., tokens) via wrapper script.
- No packet payload capture; network limited to counters & metadata (no `tcpdump` unless escalated).
- Whitelist process list fields; truncate after N chars.
- Store artifacts locally root-owned with `0600` perms; encrypt at rest if centralized.

### Retention & Rotation
- Directory: `/var/lib/forensics/<ISO8601>/`  
- Compress after capture (`tar --lz4`).  
- systemd tmpfiles rule to prune > 7 days or beyond size quota (e.g., 500MB).  
- Optional shipping: fluent-bit sidecar shipping only aggregated metrics.

### Automation (systemd Example)
1. Alert triggers (Prometheus Alertmanager webhook) -> writes flag file `/run/forensic.trigger`.  
2. `path` unit watches file creation → starts `forensic-capture@<timestamp>.service`.  
3. Service runs bounded script (max 2 min runtime) gathering snapshots + starts a timer for periodic deltas (30 min).  
4. After 30 min, timer stops; service compacts & labels bundle with hostID, incidentID.

### Sample Service Snippets
`/usr/local/bin/forensic_snapshot.sh` (pseudo):
```bash
#!/usr/bin/env bash
set -euo pipefail
OUT="$1"; mkdir -p "$OUT"
(redact() { sed -E 's/(password|token|secret)=[^ ]+/\1=REDACTED/g'; };
  date --iso-8601=seconds > "$OUT/meta.time";
  uptime > "$OUT/uptime.txt";
  ps -eo pid,ppid,comm,%cpu,%mem,args | head -200 | redact > "$OUT/ps.txt";
  vmstat 1 5 > "$OUT/vmstat.txt";
  iostat -x 1 3 > "$OUT/iostat.txt";
  ss -tni > "$OUT/ss.txt";
  nstat -az > "$OUT/nstat.txt";
  dmesg | tail -200 > "$OUT/dmesg_tail.txt";
  slabtop -o -s c -n 1 > "$OUT/slab.txt" )
```
Systemd timer ensures additional snapshots every 5 minutes into same dir until stop.

### Correlation
- Include tags: `hostname`, `incidentID`, `UTC timestamp`, `kernel version` in `meta.json`.
- Use consistent naming to allow later centralized ingestion (e.g., `forensic_<host>_<incident>_<seq>.tar.lz4`).

### Why This Works
- Focus on metadata vs payload reduces privacy risk.
- Sampling over 30 min shows trends (growth vs spike).
- systemd path+timer pattern avoids always-on overhead.
- eBPF tools optional, gated by capability detection.

---

**Previous:** [Back to Questions](mock_2_questions.md)
````
