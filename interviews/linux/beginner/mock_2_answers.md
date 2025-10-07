````markdown
# Linux - Beginner Interview Mock #2 - Answer Key

> **Difficulty:** Beginner  
> **Duration:** ~30 minutes  
> **Goal:** Assess file navigation, system resource inspection, scheduling, compression, and permissions basics.

---

## 🧠 Section 1: Core Questions - Answers

### 1. What is the difference between absolute and relative paths in Linux?
**Answer:**
- **Absolute**: Begins with `/`, full path from root (e.g. `/etc/hosts`). Independent of current directory.
- **Relative**: Interpreted from current working directory (no leading `/`). Uses `.` (current), `..` (parent), `~` (home in shells).
```bash
cd /var/log            # absolute
cd ../tmp              # relative (up one then into tmp)
ls ../../              # two levels up
```

### 2. How do you check disk space usage in Linux?
**Answer:**
```bash
df -h                  # mounted filesystem usage
df -i                  # inode usage
du -sh *               # sizes in current dir
du -xhd1 /var          # top level under /var (same FS)
du -sh /home/* | sort -h | tail   # biggest home dirs
```

### 3. What is a symbolic link (symlink) and how do you create one?
**Answer:** A symbolic (soft) link is a pointer referencing a path; a hard link points to the same inode. Create:
```bash
ln -s /real/path/app config_app
ls -l config_app   # shows -> target
```
Broken if target removed; unlike hard links, can cross filesystems.

### 4. Explain the purpose of the `sudo` command. When would you use it?
**Answer:** Executes a command as another user (default root) with policy defined in `/etc/sudoers` (via `visudo`). Used for privileged actions: package install, service mgmt, editing system config, managing users, mounting devices. Logged for audit. Principle: least privilege & run single commands (`sudo -l` to list allowed commands).

### 5. What are environment variables? How do you display and set them?
**Answer:** Key-value pairs inherited by processes. View: `printenv`, `env`, `echo $VAR`. Set (session): `export KEY=value`. Persist (user): append to `~/.bashrc` or `~/.profile`. System-wide: `/etc/environment` (simple KEY=VALUE) or `/etc/profile.d/*.sh`.
```bash
export PATH="$PATH:$HOME/bin"
echo 'export EDITOR=vim' >> ~/.bashrc && . ~/.bashrc
```

### 6. How do you compress and decompress files in Linux?
**Answer:**
```bash
tar -czf data.tar.gz dir/      # create gzip tar
tar -xzf data.tar.gz           # extract
gzip file && gunzip file.gz    # single file
zip -r archive.zip dir/ && unzip archive.zip
```
Flags: `c` create, `x` extract, `t` list, `z` gzip, `j` bzip2, `J` xz, `f` file name.

### 7. What is the difference between `cat`, `less`, and `more` commands?
**Answer:**
- `cat`: dumps file(s) sequentially; good for small files or piping.
- `less`: pager; bidirectional navigation, search, efficient for large files.
- `more`: older pager; mostly forward only. Prefer `less`.

### 8. How do you kill a running process in Linux?
**Answer:** Send a signal via PID or name:
```bash
ps aux | grep app
kill <PID>              # SIGTERM (15)
kill -9 <PID>           # SIGKILL (force)
pkill appname           # by name
killall appname         # all exact matches
lsof -p <PID>           # inspect before killing
```
Prefer graceful (TERM) → escalate (KILL) if needed.

### 9. What is a cron job and how do you create one?
**Answer:** Scheduled command defined in a crontab. Edit with `crontab -e`. Format: `m h dom mon dow cmd`. Examples:
```bash
0 2 * * * /usr/local/bin/backup.sh >> /var/log/backup.log 2>&1
*/10 * * * * /usr/local/bin/health_check.sh
```
List: `crontab -l`. System-wide: `/etc/crontab`, `/etc/cron.d/`.

### 10. Explain file ownership in Linux. How do you change file ownership?
**Answer:** Each file has user + group owner controlling permission triads. View: `ls -l`. Change with `chown user:group file` (root required to change user). Group-only: `chown :group file` or `chgrp group file`. Recursive: `chown -R`.

---

## ⚙️ Section 2: Scenario - Answer

**Scenario Recap:** Cron cleanup script works manually but not via cron.

**Diagnosis Path:**
1. List cron: `crontab -l` – confirm entry syntax & schedule.
2. Redirect output: `... /path/cleanup.sh >> /var/log/cleanup.log 2>&1`.
3. Check environment differences: cron has minimal `PATH`. Add: `PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/bin` at top of script.
4. Ensure script has shebang & exec perms: `head -1 cleanup.sh` (`#!/bin/bash`), `chmod 750 cleanup.sh`.
5. Use absolute paths inside script (`/usr/bin/find` vs `find`).
6. Confirm expected user: maybe job needs root for directory ownership; if system-wide place in `/etc/cron.daily/` with correct perms.
7. Check logs: `/var/log/syslog` (Debian) or `journalctl -u cron` / `journalctl -t CRON`.
8. If using `run-parts`, file name must match allowed regex (no dots). Rename if necessary.

**Fix Summary:** Add env exports, absolute paths, logging, correct permissions, validate by temporarily scheduling every minute then revert.

---

## 🧩 Section 3: Problem-Solving - Answer

**Task Recap:** Remove only broken symlinks safely; prevent recurrence.

### (a) Count broken symlinks
```bash
find /opt/app/releases -xtype l | wc -l
```
`-xtype l` matches symlinks whose targets do NOT exist.

### (b) Review then remove
```bash
find /opt/app/releases -xtype l -print | head    # spot check
find /opt/app/releases -xtype l -delete          # remove broken only
```
Safer dry run: omit `-delete` first.

### (c) Prevent recurrence
- Deployment script: cleanup old release dirs (retain N) before creating new symlink
- Atomic symlink update (`ln -s new release_tmp && mv -Tf release_tmp current`)
- CI/CD validation: ensure target dir exists before linking
- Monitoring: alert if broken symlink count exceeds threshold

### Example retention snippet
```bash
KEEP=5
cd /opt/app/releases
ls -1dt release-* | tail -n +$((KEEP+1)) | xargs -r rm -rf
```

---

**Previous:** [Back to Questions](mock_2_questions.md)
````
