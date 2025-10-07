# Linux - Beginner Interview Mock #1 - Answer Key

> **Difficulty:** Beginner  
> **Duration:** ~30 minutes  
> **Goal:** Assess your understanding of fundamental Linux concepts, navigation, process & permissions basics, and common troubleshooting commands.

---

## 🧠 Section 1: Core Questions - Answers

### 1. What is Linux and how does it differ from other operating systems?

**Answer:** Linux is an open-source, Unix-like operating system kernel developed by Linus Torvalds. Key differences:
- **Open Source**: Source code is freely available and can be modified
- **Multi-user & Multi-tasking**: Supports multiple users and processes concurrently
- **Security Model**: Strong permission model + discretionary access control (DAC)
- **Stability**: Known for long uptimes and predictable performance
- **Ecosystem Variety**: Many distributions (Ubuntu, CentOS/Alma, Debian, Arch, Fedora)
- **Customization & Automation**: Shell scripting and modular components

### 2. What is a shell in Linux? Name a few different shell types.

**Answer:** A shell is a command interpreter that provides a user interface (CLI) to interact with the OS by executing commands (it mediates between user and kernel via system calls). Common shells:
- **bash** – Widely used default on many distros
- **zsh** – Extended features (completion, globbing)
- **fish** – User-friendly, autosuggestions
- **dash** – Lightweight POSIX shell (often /bin/sh)
- **csh / tcsh** – C-like syntax variants
- **ksh** – Korn shell (scripting features)

### 3. Explain the Linux file system hierarchy. What are some important directories?

**Answer:** Based on the FHS (Filesystem Hierarchy Standard):
- `/` Root of entire filesystem tree
- `/bin` Essential user binaries (cat, ls, cp)
- `/sbin` System admin binaries
- `/etc` System-wide configuration
- `/lib`, `/lib64` Shared libraries for binaries in /bin & /sbin
- `/usr` Read-mostly userland programs & libraries
- `/var` Variable data (logs, spool, caches)
- `/tmp` Temporary files (often tmpfs, world-writable)
- `/home` User home directories
- `/opt` Add-on / 3rd-party packages
- `/proc` Virtual procfs (kernel & process info)
- `/sys` sysfs (device & kernel info)
- `/dev` Device nodes
- `/boot` Kernel + bootloader files
- `/run` Volatile runtime state

### 4. What is the difference between `ls -l` and `ls -la`?

**Answer:**
- `ls -l`: Long listing excluding hidden files (those starting with `.`)
- `ls -la`: Long listing including hidden entries plus `.` (current) and `..` (parent)

### 5. How do you create a new directory in Linux?

**Answer:** Use `mkdir`:
```bash
mkdir new_dir              # single
mkdir dir1 dir2 dir3       # multiple
mkdir -p a/b/c             # create parents as needed
mkdir -m 750 secure_dir    # set mode at creation
```

### 6. What command would you use to change file permissions in Linux?

**Answer:** `chmod` (symbolic or numeric):
```bash
chmod u+x script.sh              # add execute for user
chmod g-w file                   # remove write for group
chmod u=rw,g=r,o= file.conf      # explicit set
chmod 755 script.sh              # rwx r-x r-x
chmod 644 notes.txt              # rw- r-- r--
```
Permission bits: r=4, w=2, x=1 (sum per class).

### 7. Explain what a process is in Linux. How can you view running processes?

**Answer:** A process is an executing program instance with its own PID, memory space, file descriptors, and scheduling context. Inspection tools:
```bash
ps aux            # snapshot of all
top / htop        # dynamic view
pstree -p         # hierarchical view
pgrep <name>      # find PIDs by pattern
ps -u $USER       # user processes
```

### 8. What is the difference between `cp` and `mv` commands?

**Answer:**
- `cp` duplicates files/directories (source remains)
- `mv` renames or relocates (source removed) – same filesystem = metadata relink; cross-filesystem = copy + delete
```bash
cp fileA fileB
cp -r dirA dirB
mv oldname newname
mv file /target/path/
```

### 9. How do you search for files in Linux?

**Answer:**
```bash
find /path -name 'file.txt'      # by name
find /var/log -type f -size +200M
locate sshd_config               # DB-based (updatedb)
which python                     # executable in PATH
find . -perm 600 -user root
```

### 10. What is the purpose of the `grep` command? Give an example.

**Answer:** `grep` filters input by regex/pattern.
```bash
grep error /var/log/syslog
grep -i fail app.log
ps aux | grep '[n]ginx'          # avoid matching grep
grep -rn "TODO" src/
```

---

## ⚙️ Section 2: Scenario - Answer

**Scenario Recap:** Script `backup.sh` fails with `Permission denied` when run, then doesn’t execute via cron.

**Troubleshooting Steps:**
1. Verify permissions & shebang:
```bash
ls -l backup.sh
head -1 backup.sh               # should be like: #!/bin/bash
chmod +x backup.sh
```
2. Check file format (no CRLF): `file backup.sh` or `sed -n l backup.sh` (if `^M` present: `dos2unix backup.sh`).
3. Run with explicit shell: `bash backup.sh` to isolate exec bit vs logic errors.
4. Ensure path correctness; cron runs with minimal `PATH`.
5. Cron diagnostics:
   - `crontab -l`
   - Add logging: `0 2 * * * /path/to/backup.sh >> /var/log/backup.log 2>&1`
   - Inspect `/var/log/syslog` or `journalctl -u cron` depending on distro.
6. Add absolute paths inside script (`/usr/bin/tar`, `/bin/date`).
7. Export required environment or source profile inside script if using variables.
8. Add `set -euo pipefail` for safer failure handling and explicit exit codes.

**Common Root Causes:** Missing execute bit, wrong shebang, Windows line endings, relative paths, insufficient permissions on parent directory, relying on interactive-only environment variables.

**Fix Summary:** Correct shebang, add execute bit, ensure UNIX line endings, use absolute paths, add logging, verify cron environment.

---

## 🧩 Section 3: Problem-Solving - Answer

**Task Recap:** `/var` at 95% usage; identify and remediate safely.

### Priority Command Sequence
```bash
df -h /var                               # confirm filesystem & usage
sudo du -xhd1 /var                       # high-level breakdown
sudo du -xh /var | sort -h | tail -20    # largest items (human readable)
sudo find /var/log -type f -size +200M -printf '%s %p\n' | sort -nr | head
sudo lsof +L1                            # deleted-but-open files still consuming space
```

### Common Space Hogs
- Unrotated or verbose logs
- Package caches (`/var/cache/apt`, yum/dnf caches)
- Container images & layers (`/var/lib/docker` / `/var/lib/containers`)
- Core dumps
- Stale build artifacts or temporary extraction dirs

### Remediation Examples
```bash
sudo journalctl --vacuum-size=500M
sudo truncate -s 0 /var/log/huge.log
sudo apt-get clean      # or 'dnf clean all'
find /var/tmp -type f -mtime +7 -delete
# Remove old cores after analysis
dsudo find /var -type f -name 'core.*' -delete
```
If a deleted large file is held open: restart the owning process from `lsof +L1` output.

### Safety Considerations
- Avoid deleting under `/var/lib` blindly (databases, state directories)
- Prefer compression over deletion for compliance-critical logs
- Validate backups before purging archives
- Keep at least recent rotated logs for incident investigation

### Preventive Measures
- Configure `logrotate` with compression & retention limits
- Set journald caps (`/etc/systemd/journald.conf`: `SystemMaxUse=`)
- Add disk usage monitoring & alerts (e.g., Prometheus 80% warn / 90% critical)
- Limit debug logging in production
- Scheduled cleanup script with retention policy

### Sample `logrotate` Snippet
```conf
/var/log/app/*.log {
  daily
  rotate 14
  compress
  missingok
  notifempty
  create 640 appuser adm
  postrotate
    systemctl reload app.service >/dev/null 2>&1 || true
  endscript
}
```

---

**Previous:** [Back to Questions](mock_1_questions.md)
