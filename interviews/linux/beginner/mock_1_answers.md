# Linux Beginner Mock Interview #1 - Answers

## Answer 1
**What is Linux and how does it differ from other operating systems?**

Linux is an open-source, Unix-like operating system kernel developed by Linus Torvalds. Key differences:
- **Open Source**: Source code is freely available and can be modified
- **Multi-user and Multi-tasking**: Supports multiple users simultaneously
- **Security**: Built-in security features and permission systems
- **Stability**: Known for reliability and uptime
- **Cost**: Free to use and distribute
- **Customization**: Highly customizable with many distributions available

## Answer 2
**What is a shell in Linux? Name a few different shell types.**

A shell is a command-line interface that allows users to interact with the operating system by executing commands. It acts as an intermediary between the user and the kernel.

Common shell types:
- **Bash** (Bourne Again Shell) - Most common default shell
- **Zsh** (Z Shell) - Extended Bourne shell with improvements
- **Fish** (Friendly Interactive Shell) - User-friendly with syntax highlighting
- **Csh** (C Shell) - C-like syntax
- **Tcsh** - Enhanced C shell
- **Dash** - Lightweight POSIX-compliant shell

## Answer 3
**Explain the Linux file system hierarchy. What are some important directories?**

Linux follows the Filesystem Hierarchy Standard (FHS):

- **/** - Root directory, top of the hierarchy
- **/bin** - Essential user binaries/commands
- **/boot** - Boot loader files and kernel
- **/dev** - Device files
- **/etc** - System configuration files
- **/home** - User home directories
- **/lib** - Essential shared libraries
- **/media** - Removable media mount points
- **/mnt** - Temporary mount points
- **/opt** - Optional application packages
- **/proc** - Virtual filesystem with process information
- **/root** - Root user's home directory
- **/sbin** - System administration binaries
- **/tmp** - Temporary files
- **/usr** - User programs and data
- **/var** - Variable data (logs, databases, etc.)

## Answer 4
**What is the difference between `ls -l` and `ls -la`?**

- **`ls -l`**: Lists files in long format showing detailed information (permissions, ownership, size, date) but excludes hidden files (those starting with a dot)
- **`ls -la`**: Lists all files including hidden files (the 'a' flag includes files starting with '.') in long format

Example output difference:
```bash
$ ls -l
total 8
-rw-r--r-- 1 user user 1024 Oct  6 10:00 file.txt
drwxr-xr-x 2 user user 4096 Oct  6 09:30 directory/

$ ls -la
total 12
drwxr-xr-x 3 user user 4096 Oct  6 10:00 .
drwxr-xr-x 5 user user 4096 Oct  6 09:00 ..
-rw-r--r-- 1 user user   20 Oct  6 08:00 .hiddenfile
-rw-r--r-- 1 user user 1024 Oct  6 10:00 file.txt
drwxr-xr-x 2 user user 4096 Oct  6 09:30 directory/
```

## Answer 5
**How do you create a new directory in Linux?**

Use the `mkdir` command:

```bash
# Create a single directory
mkdir new_directory

# Create multiple directories
mkdir dir1 dir2 dir3

# Create nested directories (parent directories created if they don't exist)
mkdir -p path/to/new/directory

# Create directory with specific permissions
mkdir -m 755 new_directory
```

## Answer 6
**What command would you use to change file permissions in Linux?**

Use the `chmod` command:

**Symbolic method:**
```bash
# Add execute permission for owner
chmod u+x filename

# Remove write permission for group
chmod g-w filename

# Set read and write for owner, read for group and others
chmod u=rw,g=r,o=r filename
```

**Numeric method:**
```bash
# Set permissions to 755 (owner: rwx, group: rx, others: rx)
chmod 755 filename

# Set permissions to 644 (owner: rw, group: r, others: r)
chmod 644 filename
```

Permission values:
- r (read) = 4
- w (write) = 2  
- x (execute) = 1

## Answer 7
**Explain what a process is in Linux. How can you view running processes?**

A **process** is a running instance of a program. Each process has:
- Unique Process ID (PID)
- Parent Process ID (PPID)
- Memory space
- File descriptors
- Environment variables

Commands to view processes:
```bash
# Show processes for current user
ps

# Show all processes with detailed info
ps aux

# Real-time process viewer
top

# Enhanced version of top
htop

# Show process tree
pstree

# Show processes for specific user
ps -u username
```

## Answer 8
**What is the difference between `cp` and `mv` commands?**

- **`cp` (copy)**: Creates a duplicate of the file/directory at the destination, original remains
- **`mv` (move)**: Moves/renames the file/directory, original is removed from source location

```bash
# Copy file (original remains)
cp source.txt destination.txt
cp source.txt /path/to/destination/

# Move file (original is moved/removed from source)
mv source.txt destination.txt
mv source.txt /path/to/destination/

# Copy directory recursively
cp -r source_dir destination_dir

# Move directory
mv source_dir destination_dir
```

## Answer 9
**How do you search for files in Linux?**

Several commands can be used:

**`find` command:**
```bash
# Find files by name
find /path -name "filename"

# Find files by pattern
find /path -name "*.txt"

# Find directories
find /path -type d -name "dirname"

# Find by size
find /path -size +100M

# Find by permissions
find /path -perm 644
```

**`locate` command:**
```bash
# Fast search using database
locate filename

# Update database first
sudo updatedb
locate filename
```

**`which` command:**
```bash
# Find executable in PATH
which python
which ls
```

## Answer 10
**What is the purpose of the `grep` command? Give an example.**

`grep` (Global Regular Expression Print) searches for patterns in files or input streams.

**Purpose:**
- Search for specific text patterns
- Filter output from other commands
- Use regular expressions for complex searches

**Examples:**
```bash
# Search for word in file
grep "error" logfile.txt

# Case-insensitive search
grep -i "ERROR" logfile.txt

# Show line numbers
grep -n "pattern" file.txt

# Recursive search in directories
grep -r "function" /path/to/code/

# Search multiple files
grep "pattern" *.txt

# Invert match (show lines that don't match)
grep -v "exclude" file.txt

# Use with pipe
ps aux | grep "python"
cat file.txt | grep "search_term"

# Count matches
grep -c "pattern" file.txt
```

---

**Previous:** [Back to Questions](mock_1_questions.md)
