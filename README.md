# 💻 Linux Command Line Fundamentals

## How Does the Shell Compare to a Desktop Interface?

An operating system like Windows, Linux, or Mac OS is a special kind of program. It controls the computer's processor, hard drive, and network connection, but its most important job is to run other programs.

Since human beings aren't digital, they need an interface to interact with the operating system. The most common one these days is a graphical file explorer, which translates clicks and double-clicks into commands to open files and run programs. Before computers had graphical displays, though, people typed instructions into a program called a **command-line shell**. Each time a command is entered, the shell runs some other programs, prints their output in human-readable form, and then displays a *prompt* to signal that it's ready to accept the next command. (Its name comes from the notion that it's the "outer shell" of the computer.)

Typing commands instead of clicking and dragging may seem clumsy at first, but as you will see, once you start spelling out what you want the computer to do, you can combine old commands to create new ones and automate repetitive operations with just a few keystrokes.

## Where Am I?

The **filesystem** manages files and directories (or folders). Each is identified by an **absolute path** that shows how to reach it from the filesystem's **root directory**: `/home/public` is the directory `public` in the directory `home`, while `/home/public/kw2.txt` is a file `kw2.txt` in that directory, and `/` on its own is the root directory.

To find out where you are in the filesystem, run the command `pwd` (short for "**p**rint **w**orking **d**irectory"). This prints the absolute path of your **current working directory**, which is where the shell runs commands and looks for files by default.



# Linux Navigation and Home Directory Guide

## 🏠 What is the Home Directory?

The **home directory** is your personal workspace in the Linux filesystem. It's where your personal files, configurations, and user-specific data are stored. Think of it as your "user folder" - similar to "My Documents" in Windows, but much more central to the Linux experience.

**Characteristics of Home Directory:**
- Contains your personal files and folders
- Houses configuration files (dotfiles like `.bashrc`, `.profile`)
- Default location when you first log in
- Usually located at `/home/username` (where `username` is your login name)
- For root user, it's typically `/root`

## 🧭 Ways to Navigate to Home Directory

There are several methods to get to your home directory from anywhere in the filesystem:

### Method 1: Using `cd` with no arguments
```bash
cd
```
Simply typing `cd` with no parameters takes you directly home.

### Method 2: Using the tilde (`~`) symbol
```bash
cd ~
```
The tilde (`~`) is a shortcut that represents your home directory.

### Method 3: Using the `$HOME` environment variable
```bash
cd $HOME
```
The `$HOME` variable stores the path to your home directory.

### Method 4: Using the absolute path
```bash
cd /home/username
# or for root user:
cd /root
```
Replace `username` with your actual username.

### Method 5: Using `cd` with tilde and username
```bash
cd ~username
```
This takes you to a specific user's home directory (useful when you have permissions).

## 🔍 Verifying Your Location

After navigating to your home directory, verify your location:

```bash
pwd
```

You should see output like:
- `/home/your_username` (for regular users)
- `/root` (for root user)

## 📂 Exploring Home Directory Contents

Once in your home directory, explore what's there:

```bash
# List visible files and directories
ls

# List all files including hidden ones
ls -la

# List with detailed information
ls -l
```

**Common contents you might find:**
- **Documents**, **Downloads**, **Pictures** - Standard user folders
- **.bashrc** - Bash shell configuration
- **.profile** - User environment settings
- **.ssh/** - SSH keys and configuration

## Pro Tips

**Quick navigation tricks:**
```bash
# Go home from anywhere
cd

# Go to previous directory
cd -

# Go up one level, then home
cd ../.. && cd ~

# List home directory contents from anywhere
ls ~
```

**Environment variables related to home:**
```bash
# Display your home directory path
echo $HOME

# Display current user
echo $USER

# Display current working directory
echo $PWD
```

The home directory is your starting point and safe harbor in the Linux filesystem - master these navigation methods and you'll always find your way back!



## How Can I Identify Files and Directories?

`pwd` tells you where you are. To find out what's there, type `ls` (which is short for "**l**i**s**ting") and press the enter key. On its own, `ls` lists the contents of your current directory (the one displayed by `pwd`). If you add the names of some files, `ls` will list them, and if you add the names of directories, it will list their contents.

### Command Structure: Commands, Options, and Arguments

Understanding Linux commands follows a simple pattern: **command + options + arguments**

```bash
# Basic command
ls

# Command with options (switches)
ls -l -a

# Combined options (shorthand)
ls -la

# Command with arguments (directories to list)
ls /public /public/ucebnove

# Command with options and arguments
ls -latr /public
```

### Practical Example

Let's explore different ways to use the `ls` command with real examples:

**Basic listing:**
```bash
ls /public/ucebnove
```
Shows the contents of the `/public/ucebnove` directory in simple format.

**Detailed listing:**
```bash
ls -l /public/ucebnove
```
Shows detailed information including permissions, ownership, size, and modification dates.

**Show all files (including hidden):**
```bash
ls -la /public/ucebnove
```
Displays all files, including hidden ones that start with a dot (.).

**Reverse time-sorted listing:**
```bash
ls -latr /public/ucebnove
```
- `-l`: Long format (detailed)
- `-a`: All files (including hidden)
- `-t`: Sort by modification time
- `-r`: Reverse order (oldest first)

### Distinguishing Files from Directories

When using `ls -l`, the first character of each line tells you the file type:

```bash
drwxr-xr-x  2 user group 4096 Sep 24 10:30 directory_name
-rw-r--r--  1 user group 1024 Sep 24 10:25 file_name.txt
```

**File Type Indicators:**
- `d` - Directory
- `-` - Regular file
- `l` - Symbolic link
- `c` - Character device
- `b` - Block device
- `s` - Socket
- `p` - Named pipe (FIFO)

## Everything is a File in Linux

In Linux philosophy, **everything is a file** - a fundamental concept that makes the system elegant and consistent:

**What does "everything is a file" mean?**
- **File = sequence of bytes**
- **Sequence of bytes ≠ always a file** (but can be treated as one)

**Types of "files" in Linux:**
- **Regular files** - Documents, images, executables
- **Directories** - Special files that contain lists of other files
- **Devices** - Hardware components (hard drives, keyboards, etc.)
- **Symbolic links** - Shortcuts pointing to other files
- **Sockets** - Communication endpoints for processes
- **Named pipes** - Allow communication between processes

### Understanding the Full `ls -l` Output

Let's break down what each part means:

```bash
drwxr-xr-x  2 user group 4096 Sep 24 10:30 directory_name
│││││││││   │  │    │     │    │           │
│││││││││   │  │    │     │    │           └── File/directory name
│││││││││   │  │    │     │    └── Modification time
│││││││││   │  │    │     └── File size (in bytes)
│││││││││   │  │    └── Group owner
│││││││││   │  └── User owner
│││││││││   └── Number of hard links
│└┴┴┴┴┴┴┴── File permissions (rwx for user, group, others)
└── File type indicator
```

### Practical Examples

**Identifying different file types in your home directory:**

```bash
# List all files with details
ls -la ~

# Example output explanation:
drwxr-xr-x  25 xsaleh users  4096 Sep 24 10:30 .
drwxr-xr-x   3 root root   4096 Sep 20 09:15 ..
-rw-r--r--   1 xsaleh users   220 Sep 20 09:15 .bash_logout
-rw-r--r--   1 xsaleh users  3771 Sep 20 09:15 .bashrc
drwx------   2 xsaleh users  4096 Sep 24 08:45 .ssh
-rw-rw-r--   1 xsaleh users  1024 Sep 24 10:25 document.txt
lrwxrwxrwx   1 xsaleh users    12 Sep 24 10:30 shortcut -> /path/to/file
```

**What this tells us:**
- `.` and `..` are special directories (current and parent)
- `.bash_logout`, `.bashrc`, `document.txt` are regular files
- `.ssh` is a directory (note the `d` and different permissions)
- `shortcut` is a symbolic link (note the `l` and arrow showing target)

### Quick File Type Check

**Using `file` command for detailed information:**
```bash
file filename
# Example outputs:
# document.txt: ASCII text
# image.jpg: JPEG image data
# script.sh: Bourne-Again shell script, ASCII text executable
```

Understanding these file type indicators helps you navigate the filesystem more effectively and understand what you're working with at a glance.

## 🔐 Linux Permissions and Ownership

### Understanding the Permission System

Let's break down this example line from `ls -l`:
```bash
drwxr-xr-x  2 user group 4096 Sep 24 10:30 directory_name
```

**Detailed breakdown:**
```bash
drwxr-xr-x  2 user group 4096 Sep 24 10:30 directory_name
│││││││││   │  │    │     │    │           │
│││││││││   │  │    │     │    │           └── File/directory name
│││││││││   │  │    │     │    └── Last modified date/time
│││││││││   │  │    │     └── Size in bytes (4096 = 4KB)
│││││││││   │  │    └── Group owner
│││││││││   │  └── User owner (file owner)
│││││││││   └── Hard link count
│└┬─┘└┬─┘└┬─┘── Permissions for: owner, group, others
│ │   │   └── Others (everyone else): r-x (read, execute)
│ │   └── Group: r-x (read, execute)
│ └── Owner: rwx (read, write, execute)
└── File type: d (directory)
```

### Permission Types

**Basic permissions:**
- **r (read)** = 4: View file contents or list directory contents
- **w (write)** = 2: Modify file contents or create/delete files in directory
- **x (execute)** = 1: Run file as program or enter directory

**Permission calculation:**
- rwx = 4+2+1 = 7 (full permissions)
- r-x = 4+0+1 = 5 (read and execute)
- r-- = 4+0+0 = 4 (read only)
- --- = 0+0+0 = 0 (no permissions)

### Changing File Permissions

**Using `chmod` command:**

```bash
# Numeric method (octal notation)
chmod 755 filename          # rwxr-xr-x
chmod 644 filename          # rw-r--r--
chmod 600 filename          # rw-------
chmod 777 filename          # rwxrwxrwx (full access - be careful!)

# Symbolic method
chmod u+x filename          # Add execute for user
chmod g-w filename          # Remove write for group
chmod o+r filename          # Add read for others
chmod a+r filename          # Add read for all (user, group, others)
chmod u=rw,g=r,o=r filename # Set specific permissions
```

**Common permission combinations:**
- `755` - Directories (owner: full, others: read/execute)
- `644` - Regular files (owner: read/write, others: read-only)
- `600` - Private files (owner: read/write, others: no access)
- `700` - Private directories (owner: full access, others: no access)

### Changing Ownership

**Changing user owner with `chown`:**
```bash
# Change user owner
sudo chown newuser filename
sudo chown newuser directory/

# Change user and group
sudo chown newuser:newgroup filename
sudo chown newuser:newgroup directory/

# Recursive change (for directories and contents)
sudo chown -R newuser:newgroup directory/
```

**Changing group owner with `chgrp`:**
```bash
# Change group owner
sudo chgrp newgroup filename
sudo chgrp newgroup directory/

# Recursive change
sudo chgrp -R newgroup directory/
```

### User and Group Management

**Creating a new user:**
```bash
# Create user with home directory
sudo useradd -m username

# Create user with specific shell and home directory
sudo useradd -m -s /bin/bash username

# Set password for the new user
sudo passwd username
```

**Creating and managing groups:**
```bash
# Create a new group
sudo groupadd groupname

# Add existing user to a group
sudo usermod -aG groupname username

# Add user to multiple groups
sudo usermod -aG group1,group2,group3 username

# Remove user from a group
sudo gpasswd -d username groupname
```

**Adding user to sudo group:**
```bash
# Add user to sudo group (Ubuntu/Debian)
sudo usermod -aG sudo username

# Verify user's groups
groups username
# or check current user's groups
groups
```

### Practical Examples - Check at home/dorm/car at any time, "Examples are not related to each other".

**Example 1: Creating a new user with sudo access**
```bash
# Create new user
sudo useradd -m -s /bin/bash xsalehdoe

# Set password
sudo passwd xsalehdoe

# Add to sudo group
sudo usermod -aG sudo xsalehdoe

# Verify the user can use sudo
sudo -u xsalehdoe sudo whoami
# Should output: root
```


**Example 2: Setting up a shared directory**
```bash
# Create a shared directory
sudo mkdir /shared/projects

# Create a group for project team
sudo groupadd projectteam

# Add users to the group
sudo usermod -aG projectteam alice
sudo usermod -aG projectteam bob

# Change ownership and permissions
sudo chown :projectteam /shared/projects
sudo chmod 775 /shared/projects

# Result: Group members can read, write, and execute
```

### Checking Permissions and Ownership

**Useful commands for verification:**
```bash
# Check current user
whoami

# Check user's groups
groups

# Check specific user's groups
groups username
```

Understanding permissions and ownership is crucial for Linux security and collaboration. Always use the principle of least privilege - give only the minimum permissions necessary for the task.

# Linux File Paths: Absolute vs Relative

## Understanding File Paths

An absolute path is like a latitude and longitude: it has the same value no matter where you are. A **relative path**, on the other hand, specifies a location starting from where you are: it's like saying "20 kilometers north".

The shell decides if a path is absolute or relative by looking at its first character: If it begins with `/`, it is absolute. If it does *not* begin with `/`, it is relative.

## Path Types

### Absolute Paths
**Always start with `/` - same meaning everywhere:**
```bash
/public/ucebnove/file.txt    # Always refers to the same file
```

### Relative Paths  
**Start from your current location:**
```bash
ucebnove/file.txt           # Only works if you're in /public/
```

## Examples with `/public/`

### From Root Directory (`/`)
```bash
# Absolute: /public/ucebnove
# Relative: public/ucebnove
ls /public/ucebnove         # Works from anywhere
ls public/ucebnove          # Only works from root (/)
```

### From Home Directory (`/home/username`)
```bash
# Absolute: /public/ucebnove  
# Relative: ../../public/ucebnove
ls /public/ucebnove         # Works from anywhere
ls ../../public/ucebnove    # Go up 2 levels, then down to public
```

### From `/public/` Directory
```bash
# Absolute: /public/ucebnove
# Relative: ucebnove
ls /public/ucebnove         # Full path
ls ucebnove                 # Just the subdirectory name
```

### From `/public/ucebnove/` Directory
```bash
# Absolute: /public/
# Relative: ../
ls /public/                 # Full path to parent
ls ../                      # Go up one level
```

## Special Symbols

```bash
.       # Current directory
..      # Parent directory  
~       # Home directory
-       # Previous directory (with cd)
```

## Quick Rules

- **Absolute paths**: Start with `/` - work from anywhere
- **Relative paths**: No starting `/` - depend on current location
- **Use absolute** for scripts and documentation
- **Use relative** for interactive navigation

---

# Essential Linux Commands and Concepts

## Basic Output Commands

### `echo` - Display Text
```bash
echo "Hello World"              # Print text
echo $HOME                      # Print environment variable
echo "Text" > file.txt          # Write text to file
echo "More text" >> file.txt    # Append text to file
```

## Getting Help

### `help` - Built-in Shell Help
```bash
help                            # List all built-in commands
help cd                         # Help for cd command (built-ins only)
help echo                       # Help for echo command
```

### `man` - Manual Pages
```bash
man ls                          # Show manual for ls command
man man                         # Manual about manual pages
man 5 passwd                    # Section 5 of passwd manual
```

### `help` vs `man`
- **`help`**: For shell built-in commands (cd, echo, etc.)
- **`man`**: For external programs and system functions
- Use `type command_name` to check if it's built-in or external

## Manual Pages Explained

**What are manual pages?**
Manual pages (man pages) are documentation files stored in `/usr/share/man/` that explain how commands work.

**Navigating in manual pages:**
```bash
man ls                          # Open ls manual
# Navigation keys:
# Space    - Next page
# b        - Previous page  
# /text    - Search for "text"
# n        - Next search result
# N        - Previous search result
# q        - Quit
```

## File Viewing Commands

### `cat` - Display File Contents
```bash
cat file.txt                    # Display entire file
cat file1.txt file2.txt         # Display multiple files
cat > newfile.txt               # Create file (type content, Ctrl+D to save)
```

### `less` - Page Through Files
```bash
less file.txt                   # View file page by page
less +/search file.txt          # Open and search for "search"
# Same navigation as man pages (Space, b, /text, q)
```

### `hexdump` - View Binary Data
```bash
hexdump file.txt                # Show hex representation
hexdump -C file.txt             # Canonical format (hex + ASCII)
hexdump -x file.txt             # Two-byte hex display
```

## Creating Files

### Regular Files
```bash
touch filename.txt              # Create empty file
echo "content" > file.txt       # Create file with content
cat > file.txt                  # Create file interactively
nano file.txt                   # Create and edit with nano
```

### Hidden Files
```bash
touch .hidden_file              # Hidden file (starts with .)
echo "secret" > .config         # Hidden file with content
ls -la                          # Show hidden files (with -a flag)
```

**Hidden files in Linux:**
- Start with a dot (`.`)
- Not shown in regular `ls` command
- Used for configuration files and user settings
- Examples: `.bashrc`, `.profile`, `.ssh/`

## 🔗 Symbolic Links

**What is a symbolic link?**
A symbolic link (symlink) is a special file that points to another file or directory - like a shortcut.

```bash
ln -s /path/to/original linkname     # Create symbolic link
ln -s /public/ucebnove/ pub          # Create shortcut to ucebnove
ls -l pub                            # Shows: pub -> /public/ucebnove
```

**Characteristics:**
- Shows as `l` in file type (first character of `ls -l`)
- Arrow (`->`) shows what it points to
- If original is deleted, link becomes "broken"
- Can point to files or directories

## Device File Types (c and b)

### Character Devices (`c`)
**Stream-oriented devices** that handle data one character at a time:
```bash
ls -l /dev/tty                  # Terminal device
# Output: crw-rw-rw- 1 root tty 5, 0 Sep 24 10:30 /dev/tty
```
**Examples:** keyboard, mouse, serial ports, terminals

### Block Devices (`b`)
**Block-oriented devices** that handle data in fixed-size blocks:
```bash
ls -l /dev/sda                  # Hard disk device
# Output: brw-rw---- 1 root disk 8, 0 Sep 24 10:30 /dev/sda
```
**Examples:** hard drives, USB drives, CD-ROMs, SSDs

### Device Files Location
```bash
ls -l /dev/                     # List all device files
ls -l /dev/ | grep "^c"         # Show only character devices
ls -l /dev/ | grep "^b"         # Show only block devices
```

**Key Differences:**
- **Character devices**: Data flows like a stream (keyboard input)
- **Block devices**: Data stored in addressable blocks (hard drive sectors)
- Both appear in `/dev/` directory
- System uses them to communicate with hardware

## Quick Reference

```bash
# Get help
help command_name               # For built-ins
man command_name                # For external commands

# View files  
cat file.txt                    # Show entire file
less file.txt                   # Page through file

# Create files
touch file.txt                  # Empty file
touch .hidden                   # Hidden file
echo "text" > file.txt          # File with content

# Symbolic links
ln -s /path/to/target linkname  # Create symlink

# Check file types
ls -l                          # See file type indicators
file filename                  # Detailed file information
```

