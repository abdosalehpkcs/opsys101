# 💻 More of Linux Commands

# Process and Job Control

This section covers process management and job control in Linux. You'll learn how to view running processes and control background/foreground tasks.

---

## Understanding Processes and Jobs

### What is a Process?

A **process** is a running instance of a program. Every command you execute becomes a process. Each process has:
- **PID (Process ID)** - Unique identifier number
- **Parent process** - The process that started it
- **State** - Running, sleeping, stopped, etc.
- **Resources** - CPU, memory usage

![alt text](image.png)

Examples of processes:
- Your terminal shell (bash)
- A text editor (nano, vim)
- A running script or program
- Background services (web server, database)

### What is a Job?

A **job** is a process that was started by your current shell session. Jobs are specific to your terminal - they're the commands you run in that terminal.

Key differences:
- **Process** = Any running program on the system
- **Job** = A process started from YOUR current terminal

---

## The `ps` Command

### What is `ps`?

The `ps` (process status) command displays information about running processes.

### Basic Usage

Show processes in current terminal:
```bash
ps
```

Show all your processes:
```bash
ps -u username
ps -u $(whoami)
```

Show all processes on the system:
```bash
ps aux
```

Show processes in tree format (shows parent-child relationships):
```bash
ps -ef --forest
ps auxf
```

### Understanding `ps aux` Output

```
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.0  12020  4524 ?        Ss   Oct02   0:00 sshd: /usr/sbin/sshd -D [listener] 0 of 10-
root         348  0.0  0.0  14828  6944 ?        Ss   11:51   0:00 sshd: root@pts/0
root         358  0.0  0.0   5016  3148 pts/0    Ss   11:51   0:00  \_ -bash
root         374  0.0  0.0   8280  3064 pts/0    R+   12:09   0:00      \_ ps auxf
```

Column meanings:
- **USER** - Who owns the process
- **PID** - Process ID
- **%CPU** - CPU usage percentage
- **%MEM** - Memory usage percentage
- **VSZ** - Virtual memory size
- **RSS** - Physical memory (RAM) used
- **TTY** - Terminal associated (? means no terminal)
- **STAT** - Process state (R=running, S=sleeping, T=stopped, Z=zombie)
- **START** - When the process started
- **TIME** - CPU time used
- **COMMAND** - The command that started the process

### Common `ps` Options

Find specific process:
```bash
ps aux | grep bash
```

Monitor processes in real-time:
```bash
top
```

### Practical Examples

```bash
# Find all Python processes
ps aux | grep bash

# Find processes using most CPU
ps aux --sort=-%cpu | head -10

# Find processes using most memory
ps aux --sort=-%mem | head -10
```

---

## The `jobs` Command

### What is `jobs`?

The `jobs` command lists all jobs (processes) running in the background or stopped in your current terminal session.

### Basic Usage

List all jobs:
```bash
jobs
```

List with process IDs:
```bash
jobs -l
```

List only running jobs:
```bash
jobs -r
```

List only stopped jobs:
```bash
jobs -s
```

### Understanding `jobs` Output

```
[1]+  Running                 sleep 100 &
[2]-  Stopped                 nano file.txt
[3]   Running                 python3 script.py &
```

Breaking it down:
- **[1]** - Job number
- **+** - Current job (most recent)
- **-** - Previous job
- **Running/Stopped** - Job state
- **sleep 100 &** - The command

---

## Starting Jobs in Background

### Running Commands in Background

Add `&` at the end to run in background:
```bash
sleep 100 &
```

When you do this, you'll see:
```
[1] 5678
```
- **[1]** - Job number
- **5678** - Process ID (PID)

### Example Workflow

```bash
# Start a long-running process in background
sleep 10000 &

# Continue working while it runs
ls
cd /public

# Check if it's still running
jobs
```

---

## The `fg` Command (Foreground)

### What is `fg`?

The `fg` command brings a background or stopped job to the foreground, making it the active process in your terminal.

### Basic Usage

Bring most recent job to foreground:
```bash
fg
```

Bring specific job to foreground:
```bash
fg %1        # Bring job [1] to foreground
fg %2        # Bring job [2] to foreground
```

### Practical Example

```bash
# Start a process in background
sleep 100 &
[1] 5678

# Continue working...
ls
cd ~

# Bring it to foreground
fg %1
# Now sleep is in foreground

# Press Ctrl+Z to stop it
# Press Ctrl+C to terminate it
```

---

## The `bg` Command (Background)

### What is `bg`?

The `bg` command resumes a stopped job and runs it in the background.

### Basic Usage

Resume most recent stopped job in background:
```bash
bg
```

Resume specific job in background:
```bash
bg %1        # Resume job [1] in background
bg %2        # Resume job [2] in background
```

## Complete Practical Scenarios

### Scenario 1: Managing Jobs with `sleep` and `vim`

This scenario shows how to work with multiple jobs, switching between foreground and background.

**Step-by-step walkthrough:**

```bash
# Step 1: Start a sleep command in the background
sleep 300 &
[1] 12345

# Step 2: Start vim to edit a file
vim /public/zaciatocnik.txt
# You're now editing in vim

# Step 3: While in vim, you need to check something in terminal
# Press Ctrl+Z to stop vim and return to shell
^Z
[2]+  Stopped                 vim /public/zaciatocnik.txt

# Step 4: Check your jobs
jobs
[1]-  Running                 sleep 300 &
[2]+  Stopped                 vim /public/zaciatocnik.txt

# Step 5: Start another sleep in background
sleep 400 &
[3] 12346

# Step 6: Check jobs again
jobs
[1]   Running                 sleep 300 &
[2]+  Stopped                 vim /public/zaciatocnik.txt
[3]-  Running                 sleep 400 &

# Step 7: Resume vim in foreground to continue editing
fg %2
# You're back in vim, continue editing

# Step 8: Stop vim again to do more terminal work
# Press Ctrl+Z
^Z
[2]+  Stopped                 vim /public/zaciatocnik.txt

# Step 9: Check remaining time on first sleep (just for demonstration)
jobs
[1]   Running                 sleep 300 &
[2]+  Stopped                 vim /public/zaciatocnik.txt
[3]-  Running                 sleep 400 &

# Step 10: Kill the first sleep as you don't need it
kill %1
[1]   Terminated              sleep 300

# Step 11: Resume vim to finish your work
fg %2
# Save and quit vim with :wq

# Step 12: Check remaining jobs
jobs
[3]+  Running                 sleep 400 &

# Step 13: Bring sleep to foreground to see it count (optional)
fg %3
# Now you see it running in foreground
# Press Ctrl+C to terminate it
^C

# All jobs completed!
jobs
# (empty)
```

**What you learned:**
- Background jobs keep running while you do other tasks
- Stopped jobs (like vim) stay in memory but don't execute
- You can switch between jobs using `fg` and `%jobnumber`
- `Ctrl+Z` stops a foreground job and returns you to shell
- You can kill jobs you no longer need

---

### Scenario 2: Understanding Processes Across Two Terminals

This scenario demonstrates how processes and jobs differ when working with multiple terminals.

**Terminal 1:**

```bash
# Step 1: Start a long sleep in background
sleep 1000 &
[1] 23456

# Step 2: Start vim
vim /public/zaciatocnik.txt
# Press Ctrl+Z to stop it
^Z
[2]+  Stopped                 vim /public/zaciatocnik.txt

# Step 3: Check YOUR jobs in THIS terminal
jobs
[1]-  Running                 sleep 1000 &
[2]+  Stopped                 vim /public/zaciatocnik.txt

# Step 4: Check ALL processes (including other terminals)
ps aux | grep sleep
root          50  0.0  0.0   3124   752 pts/0    S    12:36   0:00 sleep 1000
root          54  0.0  0.0   3956  1284 pts/0    S+   12:37   0:00 grep --color=auto sleep


# Step 5: Resume vim in background (this won't work well, but let's try)
bg %2
[2]+ vim /public/zaciatocnik.txt &
# Vim needs user interaction, so this is not practical
# But the command works for non-interactive programs

# Step 6: Bring vim back to foreground
fg %2
# Save and quit with :wq
```

**Terminal 2 (open this in parallel):**

```bash
# Step 1: Start a different sleep in background
sleep 500 &
[1] 23789

# Step 2: Check jobs in THIS terminal
jobs
[1]+  Running                 sleep 500 &
# Notice: You DON'T see the jobs from Terminal 1

# Step 3: But you CAN see all processes
ps aux | grep sleep
root          50  0.0  0.0   3124   752 pts/0    S    12:36   0:00 sleep 1000
root          57  0.0  0.0   3124   808 pts/1    S    12:40   0:00 sleep 1000
root          61  0.0  0.0   3956  1340 pts/1    S+   12:41   0:00 grep --color=auto sleep
# You see BOTH sleep processes!

# Step 4: Check all processes from your user
    PID TTY          TIME CMD
      1 ?        00:00:00 sshd
      7 ?        00:00:00 sshd
     17 pts/0    00:00:00 bash
     28 ?        00:00:00 sshd
     38 pts/1    00:00:00 bash
     23457 pts/0    00:00:00 vim  # Vim from Terminal 1
     50 pts/0    00:00:00 sleep # Sleep from Terminal 1
     57 pts/1    00:00:00 sleep # Sleep from Terminal 2
     62 pts/1    00:00:00 ps

# Step 5: You can kill processes from other terminals using PID
kill 23457    # Kill the sleep from Terminal 1
# The process is killed, even though it's not YOUR job
```

**Key observations:**

1. **Jobs are terminal-specific:**
   - Terminal 1 only sees its own jobs with `jobs` command
   - Terminal 2 only sees its own jobs with `jobs` command

2. **Processes are system-wide:**
   - Both terminals can see all processes with `ps`
   - You can kill processes from any terminal using PID

3. **Job numbers vs PID:**
   - Job numbers `[1], [2]` are terminal-specific
   - PIDs like `23456` are unique across the entire system

**Back to Terminal 1:**

```bash
# Your sleep was killed from Terminal 2!
jobs
[1]-  Terminated              sleep 1000
[2]+  Stopped                 vim /public/zaciatocnik.txt

# Clean up
fg %2
# Quit vim
```

---

# Command Piping, Substitution, and Redirection

Learn how to chain commands, use command output, and control where data goes.

---

## Installing Fun Tools

```bash
sudo apt update
sudo apt install cowsay fortune-mod
```

**What are these?**
- `fortune` - Displays random quotes and sayings
- `cowsay` - Makes an ASCII cow say your text

Test them:
```bash
fortune
cowsay "Hello!"
fortune | cowsay
```

---

## Command Piping (`|`)

**Piping** connects output of one command to input of another.

### Syntax
```bash
command1 | command2
```

### Examples

```bash
# Fortune from a cow
fortune | cowsay

# Different animals
fortune | cowsay -f tux
fortune | cowsay -f dragon

# Count files in directory
ls | wc -l

# Search command history
history | grep "apt"

# Sort and show first 5
ls -l | sort | head -5
```

---

## Command Substitution (`$()`)

**Substitution** uses command output as part of another command.

### Syntax
```bash
$(command)
```

### Examples

```bash
# Cow says who you are
cowsay "Hello $(whoami)"

# Cow tells the date
cowsay "Today is $(date +%A)"

# Show file count
echo "You have $(ls | wc -l) files here"

# Backup with date
cp /public/zaciatocnik.txt /public/zaciatocnik_$(date +%Y%m%d).txt

# Personalized message
echo "Welcome $(whoami)! Current directory: $(pwd)"
```

---

## Command Redirection

**Redirection** controls where input/output goes.

![alt text](image-1.png)

### Output Redirection

**Overwrite (`>`):**
```bash
echo "Hello" > file.txt
fortune > quote.txt
```

**Append (`>>`):**
```bash
echo "World" >> file.txt
fortune >> quotes.txt
```

### Input Redirection (`<`)
```bash
wc -l < file.txt
cowsay < quote.txt
```

### Error Redirection
```bash
# Hide errors
command 2> /dev/null

# Save errors
ls /fake 2> errors.txt
```

### Advanced Redirection

| Command | Action |
|---------|--------|
| `command > file` | Redirect stdout to a file |
| `command 2> file` | Redirect stderr to a file |
| `command > file 2> file2` | Redirect stdout to one file and stderr to another file |
| `command > file 2>&1` | Redirect stdout and stderr to the same file |
| `command > /dev/null` | Discard stdout |
| `command 2> /dev/null` | Discard stderr |
| `command > /dev/null 2>&1` | Discard both stdout and stderr |


### Examples

```bash
# Save cow wisdom
fortune | cowsay > wisdom.txt

# Daily quotes collection
fortune >> daily_quotes.txt
fortune >> daily_quotes.txt
cat daily_quotes.txt

# Create greeting file
echo "Welcome $(whoami)!" | cowsay > greeting.txt
cat greeting.txt

# Save and display (tee)
fortune | tee saved_fortune.txt | cowsay
```

---

## Combining All Three

### Example 1: Daily Message
```bash
echo "=== $(date +%A) ===" > daily.txt
fortune | cowsay >> daily.txt
cat daily.txt
```

### Example 2: System Info
```bash
echo "User: $(whoami)" > info.txt
echo "Files: $(ls | wc -l)" >> info.txt
echo "Processes: $(ps aux | grep $(whoami) | wc -l)" >> info.txt
cat info.txt | cowsay
```

### Example 3: Smart Greeting
```bash
cowsay "Hello $(whoami)! You have $(ls ~ | wc -l) files in home directory"
```

### Example 4: Fortune Logger
```bash
echo "$(date): $(fortune)" >> fortune_log.txt
tail -5 fortune_log.txt | cowsay
```

---

## Quick Reference

| Symbol | Meaning | Example |
|--------|---------|---------|
| `\|` | Pipe output to next command | `fortune \| cowsay` |
| `$()` | Use command output | `echo $(date)` |
| `>` | Save output (overwrite) | `echo "hi" > file.txt` |
| `>>` | Save output (append) | `echo "hi" >> file.txt` |
| `<` | Read input from file | `cowsay < file.txt` |
| `2>` | Redirect errors | `command 2> errors.txt` |
| `2>/dev/null` | Hide errors | `find / -name file 2>/dev/null` |

---

## Practice 10mins

1. Make a cow say the current date
2. Count files in `/public/` with `ls | wc -l`
3. Save 3 fortunes to one file using `>>`
4. Create: `echo "User: $(whoami), Files: $(ls | wc -l)" | cowsay`
5. Search your history for "cd" and count results: `history | grep cd | wc -l`
