# Linux Beginner Mock Interview #2 - Answers

## Answer 1
**What is the difference between absolute and relative paths in Linux?**

**Absolute Path:**
- Starts from the root directory (/)
- Provides the complete path from root to the target
- Always begins with a forward slash
- Example: `/home/user/documents/file.txt`

**Relative Path:**
- Starts from the current working directory
- Does not begin with a forward slash
- Uses special symbols:
  - `.` (current directory)
  - `..` (parent directory)
  - `~` (home directory)

**Examples:**
```bash
# Absolute paths
cd /home/user/documents
ls /etc/passwd
cp /var/log/syslog /tmp/

# Relative paths
cd documents          # from current directory
ls ../downloads       # parent directory then downloads
cp ./file.txt ~/      # current dir file to home
cd ../../             # go up two levels
```

## Answer 2
**How do you check disk space usage in Linux?**

Several commands are available:

**`df` command (disk free):**
```bash
# Show disk space for all mounted filesystems
df

# Human-readable format
df -h

# Show specific filesystem
df -h /home

# Show inodes usage
df -i
```

**`du` command (disk usage):**
```bash
# Show directory size
du /path/to/directory

# Human-readable format
du -h /path/to/directory

# Show summary (total size only)
du -sh /path/to/directory

# Show sizes of all subdirectories
du -h --max-depth=1 /path/to/directory

# Sort by size
du -sh * | sort -hr
```

**Other useful commands:**
```bash
# Check available space in current directory
df -h .

# Find largest directories
du -sh /home/* | sort -hr | head -10
```

## Answer 3
**What is a symbolic link (symlink) and how do you create one?**

A **symbolic link** is a file that points to another file or directory. It's like a shortcut that references the target file/directory by its path.

**Types of links:**
- **Symbolic (soft) link**: Points to the path of target file
- **Hard link**: Points directly to the inode of target file

**Creating symbolic links:**
```bash
# Create symlink to a file
ln -s /path/to/original/file /path/to/symlink

# Create symlink to a directory
ln -s /path/to/original/directory /path/to/symlink

# Create symlink in current directory
ln -s /path/to/original/file linkname

# Example
ln -s /home/user/documents/important.txt ~/desktop/important-link.txt
```

**Identifying symlinks:**
```bash
# List with details (symlinks show -> target)
ls -la

# Example output:
# lrwxrwxrwx 1 user user 25 Oct  6 10:00 linkname -> /path/to/original/file
```

**Benefits:**
- Can link across filesystems
- Can link to non-existent files
- Smaller file size
- If original is deleted, link becomes broken

## Answer 4
**Explain the purpose of the `sudo` command. When would you use it?**

`sudo` (Super User Do) allows authorized users to execute commands as another user, typically the root user.

**Purpose:**
- Execute commands with elevated privileges
- Perform system administration tasks
- Access restricted files and directories
- Install/remove software
- Modify system configurations

**When to use sudo:**
```bash
# Install packages
sudo apt install package-name

# Edit system files
sudo nano /etc/hosts

# Change file ownership
sudo chown user:group filename

# Restart system services
sudo systemctl restart nginx

# Mount filesystems
sudo mount /dev/sdb1 /mnt/usb

# View system logs
sudo tail -f /var/log/syslog

# Change to root user
sudo su -
```

**Configuration:**
- Users must be in the `sudo` group or listed in `/etc/sudoers`
- Password required (user's own password, not root's)
- Activities logged for security auditing

**Best practices:**
- Use `sudo` for specific commands rather than `sudo su -`
- Regularly review sudo permissions
- Use `sudo -l` to see what commands you can run

## Answer 5
**What are environment variables? How do you display and set them?**

**Environment variables** are key-value pairs that store system and user information, accessible to programs and processes.

**Common environment variables:**
- `HOME` - User's home directory
- `PATH` - Directories to search for executables
- `USER` - Current username
- `SHELL` - Default shell
- `PWD` - Current working directory

**Display environment variables:**
```bash
# Show all environment variables
env
printenv

# Show specific variable
echo $HOME
echo $PATH
printenv USER

# Show all variables (including shell variables)
set
```

**Set environment variables:**
```bash
# Temporary (current session only)
export MYVAR="value"
export PATH=$PATH:/new/directory

# Permanent (add to ~/.bashrc or ~/.profile)
echo 'export MYVAR="value"' >> ~/.bashrc
source ~/.bashrc

# System-wide (add to /etc/environment)
sudo echo 'MYVAR="value"' >> /etc/environment
```

**Examples:**
```bash
# Set custom variable
export DATABASE_URL="postgresql://localhost:5432/mydb"

# Add directory to PATH
export PATH=$PATH:$HOME/bin

# Use variable in commands
echo "Welcome $USER to $HOSTNAME"
cd $HOME/documents
```

## Answer 6
**How do you compress and decompress files in Linux?**

Several compression tools are available:

**tar (tape archive):**
```bash
# Create compressed archive
tar -czf archive.tar.gz directory/
tar -cjf archive.tar.bz2 directory/

# Extract compressed archive
tar -xzf archive.tar.gz
tar -xjf archive.tar.bz2

# List contents without extracting
tar -tzf archive.tar.gz

# Extract to specific directory
tar -xzf archive.tar.gz -C /path/to/destination/
```

**gzip/gunzip:**
```bash
# Compress file (replaces original)
gzip filename

# Decompress file
gunzip filename.gz

# Keep original file while compressing
gzip -k filename

# Compress multiple files
gzip file1 file2 file3
```

**zip/unzip:**
```bash
# Create zip archive
zip archive.zip file1 file2 directory/
zip -r archive.zip directory/

# Extract zip archive
unzip archive.zip

# Extract to specific directory
unzip archive.zip -d /path/to/destination/

# List contents
unzip -l archive.zip
```

**Common flags:**
- `-c` or `--create`: create archive
- `-x` or `--extract`: extract files
- `-z`: use gzip compression
- `-j`: use bzip2 compression
- `-f`: specify filename
- `-v`: verbose output
- `-r`: recursive (for directories)

## Answer 7
**What is the difference between `cat`, `less`, and `more` commands?**

All three commands are used to display file contents, but they work differently:

**`cat` (concatenate):**
- Displays entire file content at once
- Scrolls to the end immediately
- Good for small files
- Can concatenate multiple files

```bash
# Display file content
cat filename.txt

# Display multiple files
cat file1.txt file2.txt

# Display with line numbers
cat -n filename.txt

# Display non-printing characters
cat -A filename.txt
```

**`less` (improved `more`):**
- Displays content page by page
- Allows scrolling up and down
- Doesn't load entire file into memory
- More features and better for large files

```bash
# View file with less
less filename.txt

# Navigation in less:
# Space or f: next page
# b: previous page
# q: quit
# /pattern: search forward
# ?pattern: search backward
# G: go to end
# g: go to beginning
```

**`more` (original pager):**
- Displays content page by page
- Only allows forward scrolling
- Loads entire file into memory
- Basic functionality

```bash
# View file with more
more filename.txt

# Navigation in more:
# Space: next page
# Enter: next line
# q: quit
# /pattern: search
```

**When to use which:**
- **`cat`**: Small files, scripting, concatenation
- **`less`**: Large files, interactive viewing, searching
- **`more`**: Basic paging (less preferred over less)

## Answer 8
**How do you kill a running process in Linux?**

Several methods and commands are available:

**`kill` command:**
```bash
# Kill process by PID (graceful termination)
kill PID

# Force kill (immediate termination)
kill -9 PID
kill -KILL PID

# Send specific signal
kill -TERM PID    # Terminate (default)
kill -HUP PID     # Hang up
kill -USR1 PID    # User signal 1
```

**`killall` command:**
```bash
# Kill all processes by name
killall process_name

# Force kill by name
killall -9 process_name

# Kill processes for specific user
killall -u username process_name
```

**`pkill` command:**
```bash
# Kill processes by name pattern
pkill process_name

# Kill with pattern matching
pkill -f "python script.py"

# Kill processes by user
pkill -u username
```

**Finding process to kill:**
```bash
# Find process ID
ps aux | grep process_name
pgrep process_name

# Interactive process manager
top
htop

# Example workflow:
ps aux | grep firefox
kill 1234

# Or in one command:
pkill firefox
```

**Signal types:**
- **SIGTERM (15)**: Graceful termination (default)
- **SIGKILL (9)**: Force kill (cannot be ignored)
- **SIGHUP (1)**: Reload configuration
- **SIGSTOP (19)**: Pause process
- **SIGCONT (18)**: Resume process

## Answer 9
**What is a cron job and how do you create one?**

A **cron job** is a scheduled task that runs automatically at specified times/intervals on Unix-like systems.

**Cron service:**
- `cron` daemon runs in background
- Reads crontab files for scheduled tasks
- Executes commands at specified times

**Crontab format:**
```
* * * * * command
│ │ │ │ │
│ │ │ │ └─ Day of week (0-7, Sunday=0 or 7)
│ │ │ └─── Month (1-12)
│ │ └───── Day of month (1-31)
│ └─────── Hour (0-23)
└───────── Minute (0-59)
```

**Managing crontabs:**
```bash
# Edit current user's crontab
crontab -e

# List current user's cron jobs
crontab -l

# Remove all cron jobs
crontab -r

# Edit another user's crontab (requires sudo)
sudo crontab -e -u username
```

**Example cron jobs:**
```bash
# Run every minute
* * * * * /path/to/script.sh

# Run every day at 2:30 AM
30 2 * * * /home/user/backup.sh

# Run every Monday at 9 AM
0 9 * * 1 /usr/bin/weekly-report.py

# Run every 15 minutes
*/15 * * * * /path/to/monitor.sh

# Run on the 1st day of every month at midnight
0 0 1 * * /home/user/monthly-cleanup.sh

# Run Monday to Friday at 8 AM
0 8 * * 1-5 /path/to/workday-script.sh
```

**Special shortcuts:**
```bash
@reboot /path/to/startup-script.sh     # Run at boot
@daily /path/to/daily-task.sh          # Run once per day
@weekly /path/to/weekly-task.sh        # Run once per week
@monthly /path/to/monthly-task.sh      # Run once per month
@yearly /path/to/yearly-task.sh        # Run once per year
```

**Best practices:**
- Use absolute paths for commands and scripts
- Redirect output to log files
- Test scripts before scheduling
- Set appropriate environment variables if needed

## Answer 10
**Explain file ownership in Linux. How do you change file ownership?**

Linux file ownership consists of three components:
- **User (owner)**: The user who owns the file
- **Group**: The group that owns the file  
- **Others**: Everyone else on the system

**Viewing ownership:**
```bash
# Long listing shows ownership
ls -l filename
# Output: -rw-r--r-- 1 user group 1024 Oct 6 10:00 filename
#                    │    │     │
#                    │    │     └─ Group owner
#                    │    └─────── User owner
#                    └──────────── Link count

# Show ownership for directory contents
ls -la /path/to/directory/

# Show numeric user/group IDs
ls -ln filename
```

**Changing ownership with `chown`:**
```bash
# Change user owner only
sudo chown newuser filename

# Change user and group owner
sudo chown newuser:newgroup filename
sudo chown newuser.newgroup filename

# Change group only (keep same user)
sudo chown :newgroup filename

# Recursive change for directories
sudo chown -R user:group directory/

# Examples
sudo chown alice:developers project.txt
sudo chown -R www-data:www-data /var/www/html/
```

**Changing group ownership with `chgrp`:**
```bash
# Change group owner
sudo chgrp newgroup filename

# Recursive change
sudo chgrp -R developers /home/shared/

# Example
sudo chgrp staff document.txt
```

**Finding user/group information:**
```bash
# Show current user
whoami
id

# Show user's groups
groups
groups username

# Show all users
cat /etc/passwd

# Show all groups
cat /etc/group

# Find files owned by specific user
find /path -user username

# Find files owned by specific group
find /path -group groupname
```

**Special considerations:**
- Only root can change user ownership
- Users can change group ownership to groups they belong to
- Some files require root ownership for security (e.g., `/etc/passwd`)

---

**Previous:** [Back to Questions](mock_2_questions.md)
