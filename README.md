# 💻 Linux Command Line Fundamentals

A practical, streamlined guide to the Linux terminal: navigation, file inspection, permissions, and essential commands.

---

## Table of Contents

- [1. Introduction: CLI vs Desktop](#1-introduction-cli-vs-desktop)
- [2. Navigation and File Paths](#2-navigation-and-file-paths)
- [3. Listing Directory Contents (`ls`)](#3-listing-directory-contents-ls)
- [4. Anatomy of `ls -l` and Linux File Types](#4-anatomy-of-ls--l-and-linux-file-types)
- [5. Permissions and Ownership](#5-permissions-and-ownership)
- [6. Creating and Inspecting Files](#6-creating-and-inspecting-files)
- [7. Getting Help (`help` vs `man`)](#7-getting-help-help-vs-man)
- [8. Quick Reference Card (Tricky Flags)](#8-quick-reference-card-tricky-flags)

---

## 1. Introduction: CLI vs Desktop

An operating system manages CPU, memory, storage, and networking. While graphical interfaces (GUIs) translate clicks into commands, a **command-line shell** allows you to interact directly with the system using typed text instructions.

- When you type a command, the shell runs the corresponding program, prints its output, and displays a **prompt** waiting for the next instruction.
- The power of the CLI comes from composability: combining simple programs to automate complex workflows with minimal effort.

[⬆ Back to top](#table-of-contents)

---

## 2. Navigation and File Paths

The Linux filesystem is organized as a single hierarchical tree starting at the root directory (`/`).

### Finding Where You Are

- `pwd` (**p**rint **w**orking **d**irectory): displays the full path of your current location.
- `echo $HOME`: your home directory path (e.g. `/home/username` or `/root`).

### Absolute vs Relative Paths

| Path Type | Starts with | Description | Example |
|---|---|---|---|
| **Absolute** | `/` | Defined from the root directory; identical regardless of where you are. | `/public/ucebnove/file.txt` |
| **Relative** | (no `/`) | Defined relative to your current working directory. | `ucebnove/file.txt` or `../../public` |

### Special Path Shortcuts

- `.` — Current directory.
- `..` — Parent directory (one level up).
- `~` — Current user's home directory (`~/notes.txt` = `/home/username/notes.txt`).
- `-` — Previous working directory (used with `cd -` to jump back).

### Navigating with `cd`

```bash
cd /public/ucebnove     # Absolute path navigation
cd ucebnove             # Relative path navigation
cd ..                   # Move up one level
cd ../..                # Move up two levels
cd                      # Jump directly to your home directory (same as cd ~)
cd -                    # Jump to previous directory
```

[⬆ Back to top](#table-of-contents)

---

## 3. Listing Directory Contents (`ls`)

The `ls` command displays files and folders. Adding flags unlocks detailed information and sorting.

```bash
ls                      # List visible files and folders
ls -a                   # Include hidden files (names starting with a dot '.')
ls -l                   # Long format (detailed list)
ls -lh                  # Long format with human-readable file sizes (e.g., 4K, 25M)
ls -la                  # Long format including hidden files
ls -latr                # Long format, all files, sorted by time, reversed (newest at bottom)
```

**Useful `ls` flags breakdown:**
- `-a` / `-A`: `-a` shows all (including `.` and `..`); `-A` shows all except `.` and `..`.
- `-l`: Detailed listing with permissions, ownership, size, and date.
- `-h`: Human-readable units (`K`, `M`, `G`) instead of raw byte counts (use with `-l`).
- `-t`: Sort files by modification time (newest first).
- `-r`: Reverse whichever sort order is active.
- `-R`: Recursive listing of all subdirectories.
- `-d`: List directory entries themselves, not their contents (e.g. `ls -ld dirname`).

[⬆ Back to top](#table-of-contents)

---

## 4. Anatomy of `ls -l` and Linux File Types

Running `ls -l` displays an informative line for each file:

```text
drwxr-xr-x  2 xsaleh users 4096 Sep 24 10:30 my_folder
-rw-r--r--  1 xsaleh users 1024 Sep 24 10:25 notes.txt
││└┬─┘└┬─┘  │   │      │     │        │          └── Name
││ │   └── Others permissions (r-x = read, execute)
││ └────── Group permissions    (r-x = read, execute)
│└──────── Owner permissions    (rwx = read, write, execute)
│          │    │      │     │        └── Last modification timestamp
│          │    │      │     └── Size in bytes
│          │    │      └── Group owner
│          │    └── User owner
│          └── Number of hard links
└── File Type Indicator
```

### File Type Indicators (First Character)

In Linux, **"everything is a file"** — from regular files to hardware devices.

| Character | Type | Meaning |
|:---:|---|---|
| `-` | Regular file | Text documents, binary executables, images. |
| `d` | Directory | A special file containing a list of other files. |
| `l` | Symbolic link | A shortcut pointing to another target path (`link -> target`). |
| `c` | Character device | Stream-oriented hardware (`/dev/tty`, terminal, keyboard). |
| `b` | Block device | Block-oriented storage device (`/dev/sda`, hard drive, USB). |
| `s` | Socket | Inter-process network communication endpoint. |
| `p` | Named pipe (FIFO) | Inter-process unidirectional data stream. |

### Identifying Files and Creating Links

```bash
# Check true file type by examining content signatures
file notes.txt          # Output: notes.txt: ASCII text
file image.png          # Output: image.png: PNG image data, 800 x 600

# Create a symbolic link (shortcut)
ln -s /path/to/target link_name

# Verify symlink
ls -l link_name         # Output: lrwxrwxrwx ... link_name -> /path/to/target
```

[⬆ Back to top](#table-of-contents)

---

## 5. Permissions and Ownership

Every file and directory in Linux has an **Owner** (`u`), a **Group** (`g`), and **Others** (`o`).

### Permission Triplet

Each group of three characters represents `rwx`:

| Symbol | Permission | Value (Octal) | Effect on Files | Effect on Directories |
|:---:|---|:---:|---|---|
| `r` | Read | **4** | View file contents | List directory contents (`ls`) |
| `w` | Write | **2** | Modify/delete file contents | Create/delete files in directory |
| `x` | Execute | **1** | Run as a program / script | Enter directory (`cd`) and access files |
| `-` | None | **0** | No permission granted | No permission granted |

**Calculation Example:**
- `rwx` = $4 + 2 + 1 = 7$ (full access)
- `rw-` = $4 + 2 + 0 = 6$ (read and write)
- `r-x` = $4 + 0 + 1 = 5$ (read and execute)
- `r--` = $4 + 0 + 0 = 4$ (read only)

### Modifying Permissions (`chmod`)

```bash
# Octal / Numeric Mode
chmod 755 script.sh     # rwxr-xr-x (owner: all, group/others: read+execute)
chmod 644 file.txt      # rw-r--r-- (owner: read+write, group/others: read-only)
chmod 600 id_ed25519    # rw------- (owner only: private key requirement)
chmod 700 ~/.ssh        # rwx------ (owner only: directory requirement)

# Symbolic Mode
chmod +x script.sh      # Add execute permission for everyone
chmod u+w file.txt      # Add write permission for owner
chmod g-w file.txt      # Remove write permission from group
chmod o-rwx secret.txt  # Strip all permissions from others
chmod -R 755 folder/    # Apply permissions recursively to folder and contents
```

### Modifying Ownership (`chown` / `chgrp`)

```bash
chown alice file.txt            # Change user owner to alice
chown alice:developers file.txt # Change user to alice AND group to developers
chgrp developers file.txt       # Change group owner only
chown -R alice:developers dir/  # Change ownership recursively
```

[⬆ Back to top](#table-of-contents)

---

## 6. Creating and Inspecting Files

### Creating Files

```bash
touch empty.txt                 # Create empty file or update timestamp of existing file
echo "Hello World" > new.txt    # Create file with text (overwrites existing content)
echo "Next line" >> new.txt     # Append text to existing file without overwriting
nano file.txt                   # Open terminal text editor to write and save
```

### Viewing File Contents

```bash
# Small files
cat file.txt                    # Print entire file to screen
cat -n file.txt                 # Print with line numbers

# Large files / Paging
less file.txt                   # Interactive viewer (does not flood the terminal)
# Navigation in less:
#   [Space] / [f]  - Page down
#   [b]            - Page up
#   /pattern       - Search forward
#   ?pattern       - Search backward
#   n / N          - Next / Previous search match
#   g / G          - Jump to beginning / end of file
#   q              - Quit viewer

# Inspecting first and last lines
head -n 10 file.txt             # First 10 lines
tail -n 10 file.txt             # Last 10 lines
tail -f app.log                 # Follow log in real-time as new lines are appended

# Binary / Hexadecimal Inspection
hexdump -C binary.dat           # Canonical hex + ASCII dump side-by-side
```

[⬆ Back to top](#table-of-contents)

---

## 7. Getting Help (`help` vs `man`)

Linux commands fall into two categories: **built-in shell commands** and **external executable binaries**.

```bash
type cd                         # Output: cd is a shell builtin
type ls                         # Output: ls is aliased to `ls --color=auto` / /bin/ls
```

### Which Help Command to Use?

- **`help <command>`** — Used for **shell builtins** (e.g. `help cd`, `help echo`, `help pwd`).
- **`man <command>`** — Used for **external programs & system calls** (e.g. `man ls`, `man ssh`, `man chmod`).
- **`<command> --help` / `-h`** — Quick flag reference provided by most modern utilities.

### Man Page Sections

Manual pages are organized into numbered sections:

```bash
man 1 passwd                    # Section 1: User command (/usr/bin/passwd)
man 5 passwd                    # Section 5: Configuration file format (/etc/passwd)
```

[⬆ Back to top](#table-of-contents)

---

## 8. Quick Reference Card (Tricky Flags)

A fast cheat sheet with subtle and essential flags often tested or encountered in labs:

| Command | Flag / Example | Tricky Detail / Why It Matters |
|---|---|---|
| **`ls`** | `ls -d */` | Lists **only directories** themselves, without listing their contents. |
| | `ls -A` | Lists all files including hidden ones, **omitting `.` and `..`** (unlike `-a`). |
| | `ls -latr` | Sorts by modification time, reversed — **most recently modified files appear at the very bottom** near your prompt. |
| | `ls -lSh` | Sorts by **file size** (largest first) with human-readable units. |
| **`cd`** | `cd -` | Switches to the **previous directory** and prints its path. |
| | `cd` or `cd ~` | Both return to your `$HOME` directory without arguments. |
| **`chmod`** | `chmod 600 <key>` | Read/write for owner only. **Required by OpenSSH** for private keys (`~/.ssh/id_*`); SSH rejects keys with looser permissions. |
| | `chmod 700 ~/.ssh` | `rwx` for owner only. **Required by OpenSSH** for the `.ssh` directory. |
| | `chmod -R` | Capital **`-R`** applies recursively (lower-case `-r` is invalid or refers to remove in other tools). |
| **`chown`** | `chown :group file` | Notice the leading colon `:` — changes **only the group** without needing `chgrp` or specifying user. |
| | `chown user: file` | Colon with no group name sets the group to the user's **primary group** automatically. |
| **`cat`** | `cat -A` / `cat -vET` | Shows **hidden/special characters** (`$` for line endings, `^I` for tabs) — great for debugging script formatting. |
| | `cat -n` | Numbers all output lines. |
| **`tail`** | `tail -f file` | **Follows** live file growth (press `Ctrl+C` to stop). |
| | `tail -n +2 file` | Plus sign `+2` means output starting from **line 2 to the end** (skips header line). |
| **`touch`** | `touch -c file` | Only updates timestamp; **does not create** the file if it does not already exist. |
| **`ln`** | `ln -s target link` | Without **`-s`**, it creates a **hard link** (shares same inode; cannot span across different filesystems or link directories). |
| **`file`** | `file -b file` | **Brief mode** — outputs file type description only, suppressing filename. |
| | `file -i file` | Outputs **MIME type** string (e.g. `text/plain; charset=utf-8`). |
| **`man`** | `man -k <keyword>` | Same as `apropos` — searches short descriptions and man page names for keyword. |

[⬆ Back to top](#table-of-contents)
