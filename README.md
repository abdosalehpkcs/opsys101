# Installing Tools

This section covers the essential tools needed for working with the Linux command line and compiling software from source.

## Installation Commands

```bash
sudo apt update
sudo apt install build-essential
```

## Understanding the Commands

### What is APT?

**APT (Advanced Package Tool)** is a package management system used in Debian-based Linux distributions such as Ubuntu, Debian, and Linux Mint. It provides a simple way to manage software on your system through the command line.

APT helps you perform the following tasks:

- Install new software packages
- Update existing packages to newer versions
- Remove unwanted packages from your system
- Automatically manage dependencies between packages

### Common APT Commands

| Command | Description |
|---------|-------------|
| `sudo apt update` | Refreshes the package list from repositories |
| `sudo apt upgrade` | Upgrades all installed packages to their latest versions |
| `sudo apt install <package>` | Installs a specific package |
| `sudo apt remove <package>` | Removes a package from the system |

**Note:** The `sudo` prefix is required because installing, updating, or removing software requires administrator privileges. The `sudo` command (SuperUser DO) allows you to execute commands with elevated permissions.

### APT Repositories

#### What are Repositories?

Repositories are servers that store software packages. APT downloads packages from these repositories.

#### Repository Locations

Ubuntu has two formats for storing repository information:

**Old format (still supported):**
- `/etc/apt/sources.list` - Traditional single file with all repositories

**New format (Ubuntu 22.04+):**
- `/etc/apt/sources.list.d/ubuntu.sources` - Modern DEB822 format

The new format is more structured and easier to read, but both formats work.

#### Basic Repository Commands

View repositories (old format):
```bash
cat /etc/apt/sources.list
```

View repositories (new format):
```bash
cat /etc/apt/sources.list.d/ubuntu.sources
```

Edit repository list (new format):
```bash
sudo vim /etc/apt/sources.list.d/ubuntu.sources
```

Update package information from repositories:
```bash
sudo apt update
```

**Note:** Always run `sudo apt update` after modifying repositories to refresh the package list.

### What is build-essential?

**build-essential** is a meta-package that bundles together all the essential tools needed for compiling and building software from source code in Linux.

### Components Included

When you install `build-essential`, you get:

- **gcc** - GNU C Compiler for compiling C programs
- **g++** - GNU C++ Compiler for compiling C++ programs
- **make** - Build automation tool that controls the compilation process
- **libc-dev** - C standard library development files and headers
- Additional libraries and utilities required for software development

### Why You Need build-essential

You'll need these tools when you want to:

- Compile programs written in C or C++
- Build software from source code
- Install packages that require compilation during installation
- Develop your own software in C/C++
- Work with many programming tools and frameworks that need to compile native extensions

## Verification

After installation, verify that the tools are installed correctly:

```bash
gcc --version
g++ --version
make --version
```

Each command should display version information, confirming successful installation.

# Public Directory Guide

The `/public/` directory contains shared resources, examples, and materials for students. All students have read access to this directory.

## Accessing the Public Directory

Navigate to the public directory:
```bash
cd /public/
```

List contents:
```bash
ls -la /public/
```

## Directory Structure

```
/public/
├── ls (script)
├── prednasky/
├── priklady/
├── samples/
├── seminare/
├── testovaci_adresar/
├── ucebnove/
├── zaciatocnik.txt
└── zadania/
```

## Contents Description

### Files

**`ls`** (executable script)
- Custom script for demonstrating command behavior
- Usage: Run with `./ls` or `/public/ls`

**`zaciatocnik.txt`**
- Beginner's guide text file
- Contains introductory materials for new students

### Directories

**`prednasky/`** (Lectures)
- Contains lecture materials and presentations

**`priklady/`** (Examples)
- Practice examples and sample code

**`samples/`**
- Additional sample files and demonstrations

**`seminare/`** (Seminars)
- Seminar materials and exercises

**`testovaci_adresar/`** (Testing Directory)
- Practice directory for testing commands

**`ucebnove/`** (Educational Materials)
- Core educational content

**`zadania/`** (Assignments)
- Course assignments and tasks
- Problem sets and exercises




## Tips

1. Create a symbolic link to `/public/` in your home directory for quick access:
   ```bash
   ln -s /public ~/public
   ```

2. Bookmark frequently used directories:
   ```bash
   alias exercises='cd /public/priklady'
   alias assignments='cd /public/zadania'
   ```

3. Copy entire directories for offline practice:
   ```bash
   cp -r /public/priklady ~/my-practice/
   ```


# Aliases and Links

This section covers two powerful features in Linux: aliases for creating command shortcuts and links for creating file references.

## Part 1: Aliases

### What is an Alias?

An alias is a shortcut for a command or series of commands. It saves typing and makes complex commands easier to remember.

### Creating Temporary Aliases

Temporary aliases only exist in your current terminal session:

```bash
alias ll='ls -lah'
alias update='sudo apt update && sudo apt upgrade'
alias ..='cd ..'
```

### Viewing Aliases

See all defined aliases:
```bash
alias
```

See a specific alias:
```bash
alias ll
```

### Removing Aliases

Remove a temporary alias:
```bash
unalias ll
```

### Practical Alias Examples

```bash
# Navigation
alias home='cd ~'

# Safety aliases (ask before overwrite/delete)
alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'

# System monitoring
alias ports='netstat -tulanp'
alias meminfo='free -h'
alias diskspace='df -h'

# Quick edits
alias bashrc='vim ~/.bashrc'
alias reload='source ~/.bashrc'
```

### Making Aliases Permanent

To make aliases permanent, add them to your shell configuration file.

Edit `.bashrc` (for Bash shell):
```bash
sudo vim ~/.bashrc
```

Add your aliases at the end of the file:
```bash
# My custom aliases
alias ll='ls -lah'
alias update='sudo apt update && sudo apt upgrade'
alias gs='git status'
alias gp='git push'
alias python='python3'
```

Save and apply changes:
```bash
source ~/.bashrc
```


## Part 2: Links

### What are Links?

Links are references to files or directories. They allow multiple names or locations to point to the same data.

### Types of Links

Linux has two types of links:

1. **Symbolic Links (Soft Links)** - Like shortcuts, point to a path
2. **Hard Links** - Direct references to file data on disk

### Symbolic Links (Symlinks)

Symbolic links are like shortcuts in Windows - they point to another file or directory.

**Create a symbolic link:**
```bash
ln -s /path/to/original /path/to/link
```

**Example:**
```bash
ln -s ~/Documents/project ~/Desktop/project-link
ln -s /etc/nginx/sites-available/mysite /etc/nginx/sites-enabled/mysite
```

**Characteristics:**
- Can link to directories
- Can link across different file systems
- If original is deleted, the link breaks
- Shows the path it points to

### Hard Links

Hard links are additional names for the same file data on disk.

**Create a hard link:**
```bash
ln /path/to/original /path/to/linkname
```

**Example:**
```bash
ln ~/Documents/important.txt ~/Backup/important.txt
```

**Characteristics:**
- Cannot link directories (with exceptions)
- Must be on the same file system
- If original is deleted, data still exists through other hard links
- All hard links are equal - no "original"

### Viewing Links

Check if a file is a link:
```bash
ls -lh
```

Symbolic links show with `->`:
```
lrwxrwxrwx 1 user user 24 Oct 07 10:30 mylink -> /path/to/original
```

See where a symbolic link points:
```bash
readlink mylink
readlink -f mylink  # Show absolute path
```

### Removing Links

Remove a link (doesn't affect the original file):
```bash
rm linkname
unlink linkname
```

**Warning:** Be careful with trailing slashes when removing directory symlinks:
```bash
rm mylink      # Correct - removes the link
rm mylink/     # Wrong - might affect linked directory contents
```


# History, Less, and Last Commands

This section covers three essential commands for viewing information in Linux: `history` for command history, `less` for reading files, and `last` for login records.

---

## The `history` Command

### What is `history`?

The `history` command shows a list of previously executed commands in your current shell session. Linux stores your command history to help you:
- Recall previous commands
- Re-execute commands without retyping
- Review what you've done

### Basic Usage

View all command history:
```bash
history
```

View last N commands:
```bash
history 10      # Show last 10 commands
history 20      # Show last 20 commands
```

### Useful `history` Options

Search history for a specific term:
```bash
history | grep "apt"
history | grep "git"
```

Clear all history:
```bash
history -c
```

Delete a specific history entry:
```bash
history -d 523    # Delete command number 523
```

### Using History for Command Execution

Re-run the last command:
```bash
!!
```

Re-run command number N:
```bash
!523              # Run command number 523 from history
```

Re-run the last command that started with specific text:
```bash
!apt              # Run last command starting with "apt"
!git              # Run last command starting with "git"
```

Search history interactively:
```bash
Ctrl + R          # Press Ctrl+R, then type to search
```

### History File Location

Your command history is stored in:
```bash
~/.bash_history
```

View history file directly:
```bash
cat ~/.bash_history
```

### Practical Examples

```bash
# View your recent work
history 20

# Find all git commands you used
history | grep git

# Find when you last used apt
history | grep apt | tail -5

# Re-execute your last sudo command
!sudo

# Clear history before leaving (privacy)
history -c
```

---

## The `less` Command

### What is `less`?

`less` is a file viewer that allows you to read files page by page. Unlike `cat`, which dumps the entire file at once, `less` is perfect for viewing large files.

**Why "less"?** It's named as a play on the older `more` command - "less is more"!

### Basic Usage

View a file:
```bash
less filename.txt
less /var/log/syslog
less /public/zaciatocnik.txt
```

### Navigation in `less`

| Key | Action |
|-----|--------|
| `Space` or `Page Down` | Move forward one page |
| `b` or `Page Up` | Move backward one page |
| `Enter` or `Down Arrow` | Move forward one line |
| `Up Arrow` | Move backward one line |
| `g` | Go to beginning of file |
| `G` | Go to end of file |
| `/pattern` | Search forward for pattern |
| `?pattern` | Search backward for pattern |
| `n` | Go to next search result |
| `N` | Go to previous search result |
| `q` | Quit less |

### Useful `less` Options

View with line numbers:
```bash
less -N filename.txt
```

Case-insensitive search:
```bash
less -i filename.txt
```

Don't clear screen on exit:
```bash
less -X filename.txt
```

View multiple files:
```bash
less file1.txt file2.txt
# Use :n for next file, :p for previous file
```

### Practical Examples

```bash
# Read a long configuration file
less /etc/apt/sources.list

# View system logs
less /var/log/syslog

# Search for errors in a log file
less /var/log/syslog
# Then press / and type "error" to search

# View command output with less
history | less
ps aux | less
ls -la /usr/bin | less

# Read documentation
man ls | less    # Actually, man uses less by default
```

### Combining Commands with `less`

Pipe output to less for easier reading:
```bash
cat large-file.txt | less
history | less
df -h | less
```

---

## The `last` Command

### What is `last`?

The `last` command shows a history of user logins and system reboots. It reads from the `/var/log/wtmp` file to display:
- Who logged in
- When they logged in
- How long they were logged in
- Where they logged in from (terminal or IP address)

### Basic Usage

View all recent logins:
```bash
last
```

View last N logins:
```bash
last -n 10        # Show last 10 login records
last -10          # Shorter syntax
```

### Useful `last` Options

Show logins for a specific user:
```bash
last username
last root
last student
```

Show logins from a specific terminal:
```bash
last tty1
last pts/0
```

Show only successful logins (exclude failed attempts):
```bash
last
```

Show with full date and time:
```bash
last -F
```

Show system reboots:
```bash
last reboot
```

Show system shutdown events:
```bash
last -x shutdown
```

Limit output by date:
```bash
last -s yesterday     # Since yesterday
last -s -2days        # Last 2 days
last -t 20251006      # Until Oct 6, 2025
```

### Understanding the Output

```
student  pts/0    192.168.1.100   Tue Oct 7 09:15   still logged in
student  pts/1    192.168.1.100   Tue Oct 7 08:30 - 09:00  (00:30)
root     tty1                      Mon Oct 6 18:00 - 22:00  (04:00)
reboot   system boot               Mon Oct 6 17:58 - 22:30  (04:32)
```

Breaking down a line:
- **student** - Username
- **pts/0** - Terminal/pseudo-terminal
- **192.168.1.100** - Login source (IP or local)
- **Tue Oct 7 09:15** - Login time
- **still logged in** or **09:00** - Logout time or current status
- **(00:30)** - Duration of session

### Practical Examples

```bash
# Check who is currently logged in
last -n 5

# Check your own login history
last $USER
last $(whoami)

# Find all root logins
last root

# Check when the system was last rebooted
last reboot | head -5

# Check logins from the last week
last -s -7days

# Find login attempts from specific IP
last | grep "192.168.1.100"

# Check how long the system has been up
last reboot | head -1
```

### Related Commands

View currently logged-in users:
```bash
who
w
```

View failed login attempts:
```bash
lastb           # Requires root/sudo
sudo lastb
```

View last login time for all users:
```bash
lastlog
```

---

## Combining These Commands

**Example 1:** Review what commands you ran during your last session
```bash
last -1              # See when you logged in
history | less       # Browse your command history
```

**Example 2:** Search for specific commands in history and read details
```bash
history | grep "apt" | less
```

**Example 3:** Check system logs for your login time
```bash
last $(whoami) -n 5
less /var/log/auth.log
```

**Example 4:** Create a report of your activities
```bash
echo "My last login:" > report.txt
last $(whoami) -n 1 >> report.txt
echo "\nRecent commands:" >> report.txt
history 20 >> report.txt
less report.txt
```

---

## Practice Exercises

1. View your last 15 commands and find how many times you used `ls`
2. Open `/etc/passwd` with `less` and search for your username
3. Find out when you last logged into the system
4. Use `history` to re-execute a command from 5 commands ago
5. Check if the system was rebooted in the last 7 days
6. Combine `history` and `grep` to find all commands containing "sudo"
7. Read a long file with `less` and practice navigation (search, jump to end, etc.)

---

## Quick Reference

| Command | Purpose | Example |
|---------|---------|---------|
| `history` | Show command history | `history 20` |
| `history \| grep term` | Search history | `history \| grep git` |
| `!!` | Repeat last command | `!!` |
| `!N` | Run command N from history | `!523` |
| `less file` | View file with pagination | `less log.txt` |
| `/pattern` in less | Search in less | `/error` |
| `q` in less | Quit less | `q` |
| `last` | Show login history | `last -n 10` |
| `last user` | Show user's logins | `last root` |
| `last reboot` | Show system reboots | `last reboot` |
