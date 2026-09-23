# 🐚 Linux Shell Tools and Shortcuts

This lab introduces package management, the shared course directory, shell aliases, file links, and commands for reviewing activity and reading long output.

---

## Table of Contents

- [1. Packages and Build Tools](#1-packages-and-build-tools)
- [2. The Shared `/public` Directory](#2-the-shared-public-directory)
- [3. Shell Aliases](#3-shell-aliases)
- [4. Links: Symbolic and Hard](#4-links-symbolic-and-hard)
- [5. Reviewing Commands, Files, and Logins](#5-reviewing-commands-files-and-logins)
- [6. Quick Reference Card](#6-quick-reference-card)

---

## 1. Packages and Build Tools

Linux distributions install software through a **package manager**. Debian and Ubuntu use **APT** (Advanced Package Tool), which downloads packages and their dependencies from configured repositories.

> **Student machines:** Students do not have `sudo`. The commands that begin with `sudo` are included for reference or for systems where you have administrator access. The lab image should already contain the tools needed for the exercises.

### Essential APT Commands

```bash
sudo apt update                    # Refresh the package list
sudo apt upgrade                   # Upgrade installed packages
sudo apt install <package-name>    # Install a package
sudo apt remove <package-name>     # Remove a package but keep its configuration
sudo apt purge <package-name>      # Remove a package and its configuration
```

`apt update` does **not** install updates. It downloads current package information. Run it before `apt upgrade` or `apt install` so APT knows which versions are available.

### Repositories

Repositories are servers that provide packages. APT reads their addresses from configuration files:

```bash
cat /etc/apt/sources.list
ls /etc/apt/sources.list.d/
cat /etc/apt/sources.list.d/ubuntu.sources
```

After an administrator changes repository configuration, they must refresh APT's package list:

```bash
sudo apt update
```

### `build-essential`

`build-essential` is a meta-package: installing it installs a standard C/C++ build environment, including `gcc`, `g++`, `make`, and C library development headers.

```bash
sudo apt install build-essential

gcc --version
g++ --version
make --version
```

[⬆ Back to top](#table-of-contents)

---

## 2. The Shared `/public` Directory

`/public` contains shared course material. You can read and copy its contents, but should work on copies in your own home directory.

```bash
cd /public
ls -la
```

### Useful Course Locations

| Path | Contents |
|---|---|
| `/public/prednasky/` | Lecture materials |
| `/public/priklady/` | Examples and practice material |
| `/public/seminare/` | Seminar materials |
| `/public/ucebnove/` | Educational resources |
| `/public/zadania/` | Assignments |
| `/public/testovaci_adresar/` | Safe directory for experimenting |
| `/public/zaciatocnik.txt` | Introductory text file |

### Work Conveniently with `/public`

Create a symbolic link in your home directory:

```bash
ln -s /public ~/public
cd ~/public
```

Or create a temporary alias for the current terminal session:

```bash
alias exercises='cd /public/priklady'
exercises
```

Copy a directory before changing it:

```bash
cp -r /public/priklady ~/my-practice
```

[⬆ Back to top](#table-of-contents)

---

## 3. Shell Aliases

An **alias** is a short replacement for a command. It saves typing, but it is only text substitution by the shell; aliases are not a replacement for learning the underlying command.

### Create, Inspect, and Remove Aliases

Aliases created in a terminal exist only until that terminal session ends.

```bash
alias ll='ls -lah'
alias ..='cd ..'

alias                              # Show all aliases
alias ll                           # Show one alias definition
unalias ll                         # Remove one alias
```

Useful examples:

```bash
alias home='cd ~'
alias rm='rm -i'                   # Ask before removing files
alias cp='cp -i'                   # Ask before overwriting files
alias mv='mv -i'                   # Ask before overwriting files
alias diskspace='df -h'
```

### Make an Alias Persistent

For Bash, add aliases to `~/.bashrc`. This is your own configuration file, so it does not need `sudo`.

```bash
nano ~/.bashrc
```

For example, add these lines at the end of the file:

```bash
alias ll='ls -lah'
alias exercises='cd /public/priklady'
```

Load the updated file into the current shell:

```bash
source ~/.bashrc
```

> **Tip:** Run `type <command>` when a command behaves unexpectedly. It tells you whether the name is an alias, a shell builtin, or an external program.

[⬆ Back to top](#table-of-contents)

---

## 4. Links: Symbolic and Hard

Links give one file more than one name or location. Linux has two important kinds.

| Type | Command | What It Points To | Key Limitation |
|---|---|---|---|
| **Symbolic link** (symlink) | `ln -s target link` | A path to another file or directory | Breaks if the target is moved or deleted |
| **Hard link** | `ln target link` | The same file data (inode) | Cannot span filesystems and normally cannot link directories |

### Symbolic Links

Symbolic links work like shortcuts. They can point to files or directories, even on another filesystem.

```bash
ln -s /public ~/public
ln -s /path/to/original my-link
ls -l my-link
readlink my-link
readlink -f my-link                # Resolve to an absolute path
```

`ls -l` identifies a symbolic link with `l` at the beginning of its permissions and displays its target after `->`:

```text
lrwxrwxrwx 1 user user 24 Oct 07 10:30 my-link -> /path/to/original
```

### Hard Links

A hard link is another directory entry for exactly the same file data. Removing one name does not remove the data while another hard link still exists.

```bash
ln report.txt report-backup.txt
ls -li report.txt report-backup.txt # Same inode number means the same file data
```

### Remove a Link Safely

```bash
rm my-link
# or
unlink my-link
```

Do **not** add a trailing slash when removing a symlink to a directory: `rm my-link/` follows the link and may affect the target directory instead.

[⬆ Back to top](#table-of-contents)

---

## 5. Reviewing Commands, Files, and Logins

Three commands help you inspect past commands, long text output, and login records.

### Command History

```bash
history                           # Show command history
history 20                        # Show the last 20 entries
history | grep git                # Search history
history -d 523                    # Delete entry number 523
history -c                        # Clear current shell history
```

You can rerun commands without retyping them:

```bash
!!                                # Repeat the previous command
!523                              # Run command number 523
!git                              # Run the latest command starting with "git"
```

Press `Ctrl+R` to search your history interactively. Bash usually saves history in `~/.bash_history` when the shell exits.

### Read Long Output with `less`

`less` shows one screen at a time, which makes it safer and more useful than `cat` for long files or command output.

```bash
less /public/zaciatocnik.txt
history | less
ls -la /usr/bin | less
less -N /etc/passwd               # Show line numbers
less -i file.txt                  # Case-insensitive searching
```

| Key in `less` | Action |
|---|---|
| `Space` / `f` | Next page |
| `b` | Previous page |
| `g` / `G` | Beginning / end of file |
| `/pattern` / `?pattern` | Search forward / backward |
| `n` / `N` | Next / previous match |
| `q` | Quit |

### Login Records with `last`

`last` reads the system login database (`/var/log/wtmp`) and reports users, terminals, source addresses, session times, and reboots.

```bash
last                              # Recent login records
last -n 10                        # Ten most recent records
last "$USER"                      # Your own login history
last reboot                       # System reboot records
last -F                           # Full date and time
last -s -7days                    # Records since seven days ago
```

Related commands:

```bash
who                               # Currently logged-in users
w                                 # Logged-in users and what they are running
lastlog                           # Last login for each account
```

[⬆ Back to top](#table-of-contents)

---

## 6. Quick Reference Card

| Command | Flag or Example | Tricky Detail |
|---|---|---|
| `apt` | `apt update` | Refreshes package information only; it does **not** upgrade installed packages. |
| `apt` | `apt remove` vs `apt purge` | `remove` keeps package configuration; `purge` removes it too. |
| `alias` | `alias name='command'` | The quotes matter when the command contains spaces or special characters. Aliases disappear when the shell closes unless placed in `~/.bashrc`. |
| `source` | `source ~/.bashrc` | Reloads shell configuration into the current terminal; opening a new terminal does this automatically. |
| `ln` | `ln -s target link` | Creates a symbolic link. Without `-s`, `ln` creates a hard link instead. |
| `ln` | `ls -li file-a file-b` | Matching inode numbers reveal hard links to the same file data. |
| `readlink` | `readlink -f link` | Resolves a symlink to its absolute target path. |
| `rm` | `rm link`, not `rm link/` | The trailing slash can follow a directory symlink rather than remove the link itself. |
| `history` | `!!`, `!123`, `!prefix` | Repeats the previous command, command number, or latest matching command. Check first: these run immediately. |
| `less` | `less -N file` | Shows line numbers; search with `/text`, then use `n` and `N` to move between matches. |
| `last` | `last -n 10` | Limits output to the ten most recent login records. |
| `last` | `last -s -7days` | Shows records since seven days ago; use `-F` for full timestamps. |

[⬆ Back to top](#table-of-contents)