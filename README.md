# Lab 4: Search Party

Find files and search their contents with Linux `find` and `grep`, using the course files in `/public`.

---

## Table of Contents

- [1. Introduction to `find` and `grep`](#1-introduction-to-find-and-grep)
- [2. Structure of a `find` Command](#2-structure-of-a-find-command)
- [3. Names, Types, and Search Depth](#3-names-types-and-search-depth)
- [4. Size and Logical Conditions](#4-size-and-logical-conditions)
- [5. Access and Modification Times](#5-access-and-modification-times)
- [6. Count Results and Process Each File](#6-count-results-and-process-each-file)
- [7. Structure of a `grep` Command](#7-structure-of-a-grep-command)
- [8. Anchors, Case, and Whole Words](#8-anchors-case-and-whole-words)
- [9. Repetition and Character Classes](#9-repetition-and-character-classes)
- [10. Sorting, Counting, and Input](#10-sorting-counting-and-input)
- [11. Sources](#11-sources)

---

## 1. Introduction to `find` and `grep`

Linux provides two useful tools for different kinds of searching: **`find` locates files and directories**, while **`grep` searches text**.

**`find`** searches directories and subdirectories by name, type, size, modification time, and other properties. Use it to locate files or pass matching files to another command. These tests examine file properties, not contents.

**`grep`** selects lines matching a text pattern. Use it to find words, match a line's structure, or filter another command's output. It reads files or stdin and prints whole matching lines by default.

| What you need | Tool |
|---|---|
| Locate a file by its name or properties. | `find` |
| Search inside a file for text or a pattern. | `grep` |
| Search the contents of files selected by name or properties. | Use `find` to select the files, then `grep` to search their contents. |

In short: **use `find` to choose which files, and `grep` to choose which lines**.

[Back to top](#table-of-contents)

---

## 2. Structure of a `find` Command

```text
find [starting-path ...] [tests] [actions]
```

**Brackets mark optional parts**; `...` means multiple starting paths are allowed. Do not type the brackets or ellipsis. Find visits each entry in the directory tree and evaluates the tests and actions.

| Part | Required? | What happens if omitted? |
|---|---|---|
| `find` | Yes: the command name. | There is no find command to run. |
| `[starting-path ...]` | Optional in GNU find. | Search `.` (the current directory). A starting path can name a file or directory. |
| `[tests]` | Optional. | No selection tests exclude entries, so every visited entry can be selected. |
| `[actions]` | Optional. | In the simple searches shown here, print selected paths with the default `-print`. |

Each chosen test still needs its argument: `-name` needs a pattern, `-type` a type letter, and `-size` a size. Tests and actions form one expression and can be mixed, as in `-print -exec`.

### Default Action and Alternatives

An **action** says what to do with a match. **The default is `-print`: show each matching pathname on its own line.** Ordinary name/type/size searches therefore print results without an explicit action.

| Action | What it does |
|---|---|
| `-print` | Print the pathname followed by a newline. This is the default. |
| `-exec command {} \;` | Run a command such as `wc`, `head`, or `tail` for each match. `{}` is the pathname; `\;` ends the action. |
| `-ok command {} \;` | Like `-exec`, but ask for confirmation before each command is run. |
| `-ls` | Show a detailed listing for each matching entry, including permissions, owner, and size. |
| `-print0` | Separate pathnames with a NUL character instead of a newline, for passing names safely to a tool that understands this format. |
| `-delete` | Actually delete matching files or empty directories. This is not a preview and is not used in this lesson. |

`-exec` disables automatic printing. Use `-print -exec tail -n 1 {} \;` to print each pathname, then run tail on that file. The command may also produce its own output.

**Check matches before changing files.** `-delete` removes entries; `-exec` can change files if its command does. Our examples use read-only commands.

### Read the Complete Command

```bash
find /public/ucebnove/ -type f -name 'kw*' -print
# Search this tree, select regular files whose names begin with kw, and print their paths.
```

| Part | What it does |
|---|---|
| `/public/ucebnove/` | Starting directory; subdirectories are searched too. |
| `-type f` | Test whether an entry is a regular file. |
| `-name 'kw*'` | Test the final name, not the file's contents. |
| `-print` | Print the pathname of a selected entry. |

Adjacent tests mean **AND**. In a simple search with no explicit action, `find` supplies `-print`. GNU find uses the current directory if the starting path is omitted.

```bash
cd /public/
find                                   # GNU find: search the current tree and print entries.
find . -true -print                    # Same idea, with explicit path and an always-true test.
find ucebnove/                         # Search only this subtree relative to /public.
```

Put starting paths **before** tests: use `find /dev -type l`, not `find -type l /dev`.

[Back to top](#table-of-contents)

---

## 3. Names, Types, and Search Depth

### Inspect an Entry Before Assuming Its Type

```bash
cd /public/
ls ./ucebnove/kw                       # List the entry, or its contents if it is a directory.
ls -l ./ucebnove/kw                    # Add permissions, owner, size, and other details.
ls -la ./ucebnove/kw                   # Also include hidden entries when listing a directory.
cat ucebnove/kw2.txt                   # Display the contents of this named file.
file ucebnove/kw2.txt                  # Inspect its detected content/type.
file ucebnove/kw10.txt                 # Inspect this file without relying on its extension.
file ucebnove/kw                       # The extensionless name does not establish its type.
```

`ls -l directory` lists its contents; `ls -ld directory` shows the directory itself. `ll` is usually an alias for a detailed listing; its definition depends on the shell setup.

### Match Names and Types

```bash
cd /public/ucebnove/
find -type f -name 'kw3.txt'            # Exact filename, restricted to regular files.
find -type d                          # Directories, including the starting directory when it matches.
find -type l                          # Symbolic links, not regular files.
find -name 'kw*'                       # All entry types whose final names begin with kw.
find -type f -name 'kw*'               # Only regular files with those names.
find -type d -name 'kw*'               # Only directories with those names.
find -type f -name '*.history'         # Regular files ending in .history, including within year directories.
find /dev -type l -print               # List links under /dev; do not follow their targets by default.
find /public -name '[0-9]*'            # Names beginning with a digit, followed by any characters.
```

In a filename pattern, `*` means any sequence of characters. `[0-9]*` means a digit followed by anything, not "digits only." Quotes prevent Bash from expanding a pattern before find receives it.

| Pattern | Explanation |
|---|---|
| `-name 'kw'` | Exact final name; not kw2.txt or kw10.txt. |
| `-name 'kw*'` | A name beginning with kw. |
| `-name kw*` | Unquoted: Bash may replace it with matching names in the current directory before find starts. |
| `-name '.history'` | Exactly the name .history, not every file with that suffix. |
| `-name '\.history'` or `-name \.history` | Escaping the dot is unnecessary in this filename pattern; it is already literal. |
| `-name '*.history'` or `-name \*.history` | Both protect the asterisk from Bash and select the .history suffix. Quotes are clearer. |

Find searches hidden entries as well as visible ones. Its default link behavior is not to follow symbolic links. A filename containing "symlink" is not proof of the entry's actual type; `-type l` checks metadata.

### Limit Depth

```bash
cd /public/
find ucebnove/ -maxdepth 1              # Visit ucebnove itself and its immediate children only.
ls -l ucebnove/                        # Compare with a detailed directory listing.
```

Depth 0 is the starting path; its children are depth 1, and their children are depth 2. **Without `-maxdepth`, there is no maximum depth limit.** Find visits all reachable subdirectories, subject to permissions, and does not follow symbolic links by default. This includes `historia`, its year directories, and deeper entries.

**`-mindepth` controls where tests and actions begin; its default is 0.** `-mindepth 1` skips evaluating the starting path; `-mindepth 2` also skips its immediate children. Find still passes through these directories to reach deeper entries.

Combine **`-mindepth 1 -maxdepth 1`** to evaluate only immediate children. Put both options after the starting path and before the tests.

[Back to top](#table-of-contents)

---

## 4. Size and Logical Conditions

### Size in Bytes

```bash
cd /public/
find ucebnove/ -name 'kw*' -type f -size +100c
# Name begins with kw AND regular file AND more than 100 bytes.
find ucebnove/ -name 'kw*' -type f -size +160c
# Same selection with the threshold raised to more than 160 bytes.
find ucebnove/ -name 'kw*' -type f -size +200c
# More than 200 bytes; increasing the threshold cannot add smaller files.
```

| Test | Meaning |
|---|---|
| `-size 100c` | Exactly 100 bytes. |
| `-size +100c` | Strictly more than 100 bytes. |
| `c` suffix | Bytes, not letters or displayed characters. |

Do not use `-size >100c`: Bash interprets `>` as output redirection, potentially creating or truncating a file named `100c` before find reports a missing size argument.

### AND and OR

```bash
cd /public/
find -type f -and -name 'kw*'           # Both tests must match; -and can be omitted here.
find -type f -or -name 'kw*'            # Any regular file OR any entry named kw*, including directories.
find -type f -and \( -name 'kw*' -or -size +100c \)
# Regular file AND (name begins kw OR size exceeds 100 bytes).
find -type f -and '(' -name 'kw*' -or -size 100c ')'
# Quoted parentheses also work; this version tests exactly 100 bytes.
find ucebnove/ -name 'kw*' -and \( -type f -or -size +160c \)
# Name begins kw AND (regular file OR more than 160 bytes). Not the same grouping as above.
```

GNU `-and` / `-or` can also be written `-a` / `-o`. AND binds more tightly than OR. Parentheses make the intended grouping explicit. Each parenthesis must reach find as a separate argument: escape it or quote it.

| Syntax | Explanation |
|---|---|
| Bare `( ... )` | Bash treats parentheses as shell syntax, not find arguments. |
| Quoting the entire expression | Turns several tests into one argument; quote the parentheses and patterns individually. |
| Combining `'('` with `\)` | Valid: both methods pass literal parentheses. Consistent spelling is easier to read. |
| `")"A` | Passes `)A`, not a closing parenthesis. The trailing A is a typo. |

[Back to top](#table-of-contents)

---

## 5. Access and Modification Times

```bash
find /public/ucebnove/ -type f -mtime -7
# File data modified less than 7 complete 24-hour periods ago.
find /public/ucebnove/ -type f -mtime 7
# Last changed 7 full days ago: the file has not yet reached 8 days since its last change.
find /public/ucebnove/ -type f -atime -7
# Accessed less than 7 days ago: a different timestamp from modification.
find /public/ucebnove/ -type f -atime 7
# Last accessed 7 full days ago, but not yet 8 days ago.
find /public/ucebnove/ -type f -name '*.history' -mtime -14 -print
# Combine regular-file, filename, and modification-age tests.
```

Find counts complete 24-hour periods from when it starts. `-mtime 5`, `6`, `7`, or `14` selects the corresponding one-day age range. A minus sign, as in `-mtime -7`, selects ages below that number. These are not calendar-day or midnight boundaries.

### A Small Date Example

Suppose you run find on **7 October 2026 at 12:00** (all times below are UTC). `-mtime 7` means **7 whole days have passed since the last content change, but 8 whole days have not passed yet**.

| Last content change | Time since the change | Matches `-mtime 7`? |
|---|---|---|
| 30 September at 18:00 | 6 days and 18 hours | No: not yet 7 days. |
| 30 September at 12:00 | Exactly 7 days | Yes. |
| 29 September at 18:00 | 7 days and 18 hours | Yes: still less than 8 days. |
| 29 September at 12:00 | Exactly 8 days | No: already 8 days. |

**`-mtime 7` does not mean "during the last 7 days."** That is `-mtime -7`, which selects changes less than 7 days ago and excludes a change exactly 7 days ago. The same date calculation applies to `-atime 7`, but it checks the last access time instead of the last content-change time.

A filename's date does not determine its timestamps; copying or restoring files can change them. Access-time updates depend on filesystem settings, so `-atime` cannot reliably show who read a file.

The files `grezo25092024.history` and `morhac26092024.history` are under `/public/ucebnove/historia/2024/`.

[Back to top](#table-of-contents)

---

## 6. Count Results and Process Each File

### Count File Contents with `wc`

```bash
wc /public/ucebnove/kw8.txt             # Newline count, word count, byte count, then filename.
wc -l /public/ucebnove/kw9.txt          # Count newline characters in this file.
wc -w /public/ucebnove/historia/2024/grezo25092024.history
# Count whitespace-separated words in the file.
wc -c /public/ucebnove/historia/2024/grezo25092024.history
# Count bytes, which need not equal characters in multibyte text.
wc -l /public/ucebnove/kw9.txt /public/ucebnove/kw10.txt
# Count each named file and print a combined total.
```

Plain `wc` reports **newlines, words, and bytes**, in that order. `wc -l` does not count a final line without a newline. Bytes are not necessarily characters; `wc -m` counts characters instead.

### Count or Select Printed Pathnames

```bash
find /public/ucebnove/ -type f -name 'kw*' -size +160c | wc -l
# Count find's output lines, not lines inside the selected files.
find /public -name '[0-9]*' | wc -l
# Count printed entries whose names begin with digits; this includes matching directories.
find /public/ucebnove/ -type f -name '*.history' -mtime -7 | tail -n 1
# Select the last pathname emitted by find, not the newest file or the last line inside every file.
```

Counting pathnames this way assumes no newline characters in names. Find does not sort by modification time. Tail defaults to ten lines; `tail -1` is an older spelling of `tail -n 1`.

### Run a Command on Each Found File

```bash
find /public/ucebnove/ -type f -name 'kw*' -size +100c -exec wc {} \;
# Run wc once for each selected file; {} supplies that file's pathname.
find /public/ucebnove/ -type f -name 'kw*' -size +160c -exec wc -l {} \;
# Count newlines in each selected file, rather than counting find's path output.
find /public/ucebnove/ -type f -name 'kw*' -size +100c -exec tail -n 1 {} \;
# Print the last line of each selected file.
find /public/ucebnove/ -type f -name '*.history' -mtime -7 -print -exec tail -n 3 {} \;
# Print each pathname, then the last three lines of that file.
find /public/ucebnove/ -type f -name '*.history' -print -exec head -n 1 {} \;
# Print each pathname, then its first line instead.
tail -n 1 /public/ucebnove/kw8.txt /public/ucebnove/kw7.txt
# Read these two explicit files; tail normally labels multiple-file output with headers.
```

| Part | Role |
|---|---|
| `-exec` | Introduce a command to run for a match. |
| `wc`, `tail`, or `head` | The external command receiving the file argument. |
| `{}` | Placeholder replaced with the current pathname as one argument. |
| `\;` | Terminate the find action; the backslash stops Bash consuming the semicolon. |

#### Reading Each Output Row

Output of `find /public/ucebnove/ -type f -name 'kw*' -size +100c -exec wc {} \;`:

```text
	7  10 142 /public/ucebnove/kw8.txt
	8  18 160 /public/ucebnove/kw7.txt
 14  30 259 /public/ucebnove/kw9.txt
	7  20 177 /public/ucebnove/kw10.txt
 14  25 241 /public/ucebnove/kw5.txt
```

**Each row comes from one wc execution on one file:** newlines, words, bytes, then pathname.

| File | Newlines (`wc -l`) | Words (`wc -w`) | Bytes (`wc -c`) |
|---|---|---|---|
| `/public/ucebnove/kw8.txt` | 7 | 10 | 142 |
| `/public/ucebnove/kw7.txt` | 8 | 18 | 160 |
| `/public/ucebnove/kw9.txt` | 14 | 30 | 259 |
| `/public/ucebnove/kw10.txt` | 7 | 20 | 177 |
| `/public/ucebnove/kw5.txt` | 14 | 25 | 241 |

For the first file, `{}` becomes `/public/ucebnove/kw8.txt`, so find runs `wc /public/ucebnove/kw8.txt`. It repeats this **per file, not per line**.

All five files pass the tests: regular files, names starting with `kw`, and sizes above 100 bytes. The first number is a newline count, **not a row number or PID**.

There is no `total` row because each wc invocation receives one file. Wc prints both counts and pathname; find's traversal determines their order, not a name or size sort.

Adding another `-print` prints the pathname again. Head and tail read file contents; they do not execute them.

| Incorrect or fragile syntax | Why it matters |
|---|---|
| `-exec wc`, `-exec wc {}`, `-exec tail -n 1` | Incomplete action: the shown forms lack a terminator, and some also lack the file argument. |
| `-exec wc {} ;` | Bash consumes the semicolon, leaving find without its action terminator. |
| `-exec {tail -n 1}` | Braces are not a way to enclose the command here. Only `{}` has the pathname-placeholder role. |
| `wc -l find ...` | Does not run find first; wc receives those words as its own operands/options. |
| `tail -n 1 $(find ...)`, backtick spelling, `wc -l $(find ...)` | Captured path text is split and expanded by Bash. Spaces, newlines, wildcard characters, many results, or no results can break the intended operation. Use the shown `-exec ... {} \;` form. |
| `tail -n 1` alone | Reads stdin; it does not remember the previous find result. |
| `cat /public/ucebnove/historia/*` | The glob includes year directories. Cat does not recursively read their files. |

`/public` also contains PDFs and other binary files. Use head/tail only on the intended text files, as in the `.history` and `kw*` searches above.

[Back to top](#table-of-contents)

---

## 7. Structure of a `grep` Command

```text
grep [options] 'pattern' [file ...]
```

Here, brackets mean optional and `...` allows multiple files. The pattern is required in this form; without file operands, grep reads stdin. Options such as `-e` and `-f` can supply patterns differently, as explained below.

| Part | Role |
|---|---|
| `grep` | Search input text for a pattern. |
| Options | Change pattern interpretation or selection/output behavior. |
| `'pattern'` | What to match; quotes protect it from Bash. |
| File operands | Files whose contents are searched. |
| No input files | Read standard input from a pipe, redirection, or keyboard. |

Grep is case-sensitive and prints **whole matching lines** by default. It searches text without executing it.

### `grep` vs `grep -E`: BRE and ERE

A **regular expression (regex)** describes text using characters, boundaries, classes, and repetition. **BRE** and **ERE** are two ways to write these patterns:

| Command | Pattern syntax | What that means |
|---|---|---|
| `grep` | **BRE: Basic Regular Expressions** | The default syntax. Some operators need a backslash to acquire their special meaning. |
| `grep -E` | **ERE: Extended Regular Expressions** | Enables operators such as `?`, `+`, grouping parentheses, and repetition braces without those extra backslashes. |

`-E` changes **pattern syntax**, not the input files. It does not imply `-i` (ignore case), `-w` (whole words), or `-o` (only matched text). GNU grep supports the same matching capabilities in BRE and ERE, but the syntax differs.

| Purpose | BRE with `grep` | ERE with `grep -E` |
|---|---|---|
| One arbitrary character | `.` | `.` |
| Start / end anchors | `^` / `$` | `^` / `$` |
| Zero or more repetitions | `*` | `*` |
| Zero or one repetition | `\?` | `?` |
| One or more repetitions | `\+` | `+` |
| Exactly ten repetitions | `\{10\}` | `{10}` |
| Group an expression | `\(` and `\)` | `(` and `)` |
| Choose either alternative | `\|` | `|` |

These are operators in a pattern, not complete commands. The backslashed BRE forms `\?`, `\+`, and `\|` are GNU extensions. Letter classes such as `[[:alpha:]]` work in both modes.

Compare these patterns:

- `grep 'I?'` searches for the literal text `I?`. With `grep -E 'I?'`, the I becomes optional; the pattern can match an empty string and therefore select every line.
- `grep -w '[[:alpha:]]\{10\}'` and `grep -Ew '[[:alpha:]]{10}'` express the same whole-word letter-count condition. The complete commands using `/public/zaciatocnik.txt` appear in section 9.

Single quotes protect patterns from **Bash**, not from regex interpretation. Use the syntax for your chosen mode. Plain grep works for simple anchors; `-E` makes repetition patterns easier to read.

### Search File Contents

```bash
grep reset /public/ucebnove/historia/2021/vojtko15102021.history
# Search for the substring reset: pattern first, file second.
grep -w find /public/ucebnove/historia/2021/vojtko08102021.history
# Require find to be a whole word.
grep '&' /public/ucebnove/historia/2021/vojtko15102021.history
# Search for ampersands; quotes prevent Bash treating & as a background operator.
```

Put the pattern before the filename; reversing them searches for the pathname as text in the wrong file. `grep cd /public/` supplies a directory instead of a text file. Bare `grep` lacks a pattern; `grep '^1'` has a pattern but waits for stdin. Ctrl+D at an empty line ends typed input; Ctrl+C interrupts.

`grep -f /public/zaciatocnik.txt` reads **patterns from the file**, one per line, then waits for input to search. `grep /public/zaciatocnik.txt` instead treats the pathname as the pattern and waits for stdin. Neither means "search this file."

Grep status is 0 when a line is selected, 1 when none is selected, and 2 for an error in GNU grep. Lab3 explains checking the last command's status immediately. No output alone does not distinguish no matches from an error whose message was redirected.

[Back to top](#table-of-contents)

---

## 8. Anchors, Case, and Whole Words

### Beginning and End of a Line

```bash
grep '^1' /public/zaciatocnik.txt        # Select lines beginning with digit 1.
grep '\.$' /public/zaciatocnik.txt       # Select lines ending in a literal dot.
grep '.$' /public/zaciatocnik.txt        # Select any nonempty line: . can match its final character.
grep 'd$' /public/zaciatocnik.txt        # Select lines ending in lowercase d.
grep '5$' /public/zaciatocnik.txt        # Select lines ending in digit 5.
grep '$' /public/zaciatocnik.txt         # Every line has an end; selects empty lines too.
grep -E '^[0-9]' /public/zaciatocnik.txt # Select lines beginning with a digit.
grep -E '[0-9]$' /public/zaciatocnik.txt # Select lines ending with a digit.
grep -E '[a-zA-Z]$' /public/zaciatocnik.txt
# Select a line ending in a character from the written letter ranges.
```

`^` anchors the beginning and `$` anchors the end. `.` matches one character; `\.` means a literal dot. A trailing space is still a character, so a line with a dot followed by spaces does not end with that dot. `-E` does not change these basic anchor/dot meanings.

### Case and Inverted Selection

```bash
grep 'h$' /public/ucebnove/historia/2021/vojtko15102021.history
# Lines ending in lowercase h.
grep 'H$' /public/ucebnove/historia/2021/vojtko15102021.history
# Lines ending in uppercase H.
grep -i 'H$' /public/ucebnove/historia/2021/vojtko15102021.history
# Ignore case: either ending can match.
grep -v 'h$' /public/ucebnove/historia/2021/vojtko15102021.history
# Invert selection: lines NOT ending in lowercase h.
grep -ivE 'a' /public/zaciatocnik.txt
# Exclude lines containing a or A. -E is unnecessary for this single ordinary letter, but valid.
```

`-v` excludes matching **lines**, not individual characters. `ss$` requires two lowercase s characters at the end.

### Whole Words

```bash
grep -w 'help' /public/ucebnove/historia/2021/vojtko15102021.history
# Select help as a whole word, not just part of a longer word.
grep -w 'hel' /public/ucebnove/historia/2021/vojtko15102021.history
# A different whole word: does not match the beginning of help.
grep -w 'test' /public/ucebnove/historia/2021/vojtko15102021.history
# Search for the whole word test.
grep -w 'find' /public/ucebnove/historia/2021/vojtko30092021.history
# Search the file for the whole word find.
grep -w 'h$' /public/ucebnove/historia/2021/vojtko15102021.history
# Require a final h that also forms a whole word; not any longer word ending in h.
```

GNU grep's word characters are letters, digits, and underscore. Punctuation can provide a boundary. GNU `\<` / `\>` mark the beginning/end of a word; `\b` marks a word boundary. They are regex extensions, not Bash operators.

[Back to top](#table-of-contents)

---

## 9. Repetition and Character Classes

### Basic and Extended Regular Expressions

The first two commands use BRE and ERE to match the same word length. The third changes the length and prints only matching words.

```bash
grep -w '[[:alpha:]]\{10\}' /public/zaciatocnik.txt
# BRE: whole words of exactly ten letters; braces are escaped.
grep -Ew '[[:alpha:]]{10}' /public/zaciatocnik.txt
# ERE: the same letter-count idea without escaping braces.
grep -oEw '[[:alpha:]]{21}' /public/zaciatocnik.txt
# Print only each matching 21-letter word, rather than its entire line.
```

`-w` supplies whole-word matching; `-o` prints nonempty matching parts separately. Without word boundaries, a ten-letter pattern can match part of a longer word. Without `-o`, several matching words on one line still produce one selected line.

### Character Classes

| Pattern | Meaning / correction |
|---|---|
| `[[:alpha:]]` | One letter according to the current locale; both bracket pairs and both colons are required. |
| `[a-zA-Z]` | One character from the written ranges; range interpretation can depend on locale. |
| `[a-Z]` | A reversed range in ASCII ordering; can be rejected rather than meaning all letters. |
| `[A-z]` | Includes punctuation between uppercase Z and lowercase a in ASCII ordering. Not a clean letter class. |
| `[:alpha]`, `[:alpha]{9,}` | Not the POSIX letter class; the malformed brackets describe literal listed characters instead. |
| `\w` | GNU grep word character: letters, digits, or underscore, not letters only. |
| `.` | Any one character; `.{10}` is not specifically ten letters or a word. |

The optional `ascii` utility displays character codes. `man ascii` (often `man 7 ascii`) explains the punctuation between Z and a. Regex matching still follows the server's locale rules.

### Exact Counts, Minimums, and Ranges

```bash
grep -Ew '[a-zA-Z]{5}' /public/zaciatocnik.txt
# Exactly 5 characters from these ranges, forming a whole word.
grep -Ew '[a-zA-Z]{15}' /public/zaciatocnik.txt
# Exactly 15 instead.
grep -Ew '[a-zA-Z]{15,}' /public/zaciatocnik.txt
# At least 15; the missing upper number means no specified maximum.
grep -Ew '[a-zA-Z]{16,17}' /public/zaciatocnik.txt
# Between 16 and 17 inclusive.
grep -Ew '[a-zA-Z]{,10}' /public/zaciatocnik.txt
# GNU ERE extension: at most 10 repetitions, including the possibility of zero.
grep -oEw '[[:alpha:]]{19}' /public/zaciatocnik.txt
# Print each whole 19-letter match, not its surrounding line.
grep -oEw '[[:alpha:]]{20,}' /public/zaciatocnik.txt
# Print whole-word matches of at least 20 letters.
```

The same rules apply to other exact counts (`{8}`, `{9}`, `{10}`, `{21}`) and minimums (`{9,}`, `{10,}`, `{13,}`, `{16,}`, `{18,}`). Replace n in `{n}` with a number. Patterns allowing zero repetitions can match empty strings and select unexpected lines.

### Optional and Repeated Suffixes

```bash
grep 'I' /public/zaciatocnik.txt         # Literal uppercase I anywhere in the line.
grep 'I?' /public/zaciatocnik.txt        # BRE: ? here is literal, not an optional-I operator.
grep -E 'I?' /public/zaciatocnik.txt     # ERE: zero or one I; an empty match can select EVERY line.
grep -Ew 'informaci[a-zA-Z]?' /public/zaciatocnik.txt
# Whole-word informaci followed by zero or one range-matched letter.
grep -E 'informac[a-zA-Z]*' /public/zaciatocnik.txt
# Prefix informac followed by zero or more letters; no whole-word requirement.
grep -E 'informac[a-zA-Z]+' /public/zaciatocnik.txt
# Prefix informac followed by one or more letters.
grep -E 'informaci[a-zA-Z]? o' /public/zaciatocnik.txt
# Then require a literal space and o after the optional suffix.
grep -iE '\<i{1,3}\>' /public/zaciatocnik.txt
# Whole word i, ii, or iii, ignoring case.
grep -ivE '\<i{1,3}\>' /public/zaciatocnik.txt
# Exclude lines containing one of those whole-word matches.
```

The prefixes `informac` and `informaci` differ by a literal i. Adding `-w` changes substring matching into whole-word matching. With a multiword pattern, `-w` checks the outer boundaries of the entire match, not an independent boundary around every token.

`.set` requires one character before set; `.*set` allows any number, including none. In BRE, `.?set` contains a literal `?`, not an optional-character operator. ERE `?I` puts a quantifier before its item; use `I?` for optional I.

`\w\{10\}` matches ten word characters, possibly inside a longer word. `\b\w\{10\}\b` adds GNU word boundaries. `\b\w{10\}\b` mixes escaping styles; use consistent BRE or ERE syntax.

[Back to top](#table-of-contents)

---

## 10. Sorting, Counting, and Input

### Sort Matching Lines

Here, grep reads **one file** and sends its matching lines to sort. Sort orders **those lines of text**, not filenames, file sizes, or modification times. The `2021` and other digits in the input pathname do not affect the order.

**By default, sort uses ascending text order.** It compares whole lines from left to right, like sorting words rather than calculating numeric values. The current **locale** (language and character-order settings) determines how spaces, uppercase/lowercase letters, and accented characters compare. Digits are treated as text unless you request numeric sorting. Lines do not need to contain numbers.

```bash
grep -w 'find' /public/ucebnove/historia/2021/vojtko30092021.history | sort
# Ascending text order: compare whole matching lines, not their source filename.
grep -w 'find' /public/ucebnove/historia/2021/vojtko30092021.history | sort -r
# Descending text order: reverse the default comparison.
grep -w 'find' /public/ucebnove/historia/2021/vojtko30092021.history | sort -ri
# Descending text order, ignoring nonprinting characters during comparison.
grep -w 'find' /public/ucebnove/historia/2021/vojtko30092021.history | sort -rn
# Descending numeric order: use a number at the start of each line; no leading number means 0.
```

| Option | What it changes |
|---|---|
| No option | Compare complete lines as text, in ascending order. |
| `-r` | Reverse the comparison order. |
| `-i` | Ignore nonprinting characters when comparing; it does not remove them from the output or ignore letter case. |
| `-n` | Compare the numeric value at the beginning of each line, allowing leading spaces. It does not search for numbers later in the line. |

**What if there are no numbers?** Plain sort still works: it sorts the text. With `-n`, a line without a leading number has numeric value **0**; the line is not discarded. GNU sort breaks equal numeric values by comparing the whole lines as text. `-r` reverses that tie-break too, so `sort -rn` on lines with no leading numbers falls back to descending text order.

Use `sort -rn` when the matching lines begin with numbers you want to compare. If they begin with command names, use plain sort or `sort -r`. Numbers inside filenames mentioned later in a line are not numeric sort keys.

### Count Selected Lines Versus Matched Words

```bash
grep -Ew '[a-zA-Z]{20,}' /public/zaciatocnik.txt | wc
# Count newlines, whitespace-separated words, and bytes in the FULL selected lines.
grep -Ew '[[:alpha:]]{21}' /public/zaciatocnik.txt | wc
# Same kind of measurement, with an exactly-21-letter selection condition.
grep -oEw '[[:alpha:]]{21}' /public/zaciatocnik.txt
# Inspect only matching words instead; the whole-line output above can contain other words too.
```

`> wc` replaces a **file named wc**; `>> wc` appends to it. Neither invokes the wc program. Lab3 explains redirection; `>>>` is invalid syntax.

### Read the Course Text from Standard Input

```bash
grep najfrekventovanejsimi /public/zaciatocnik.txt
# Grep opens the named file itself.
grep najfrekventovanejsimi < /public/zaciatocnik.txt
# Bash opens the same file and connects it to grep's stdin.
```

`najfrekventovanejsimi` has 21 ASCII letters. `najfrekventovanejsmi` is missing an i, so a literal search distinguishes them. Lab3 covers here-documents and here-strings.

[Back to top](#table-of-contents)

---

## 11. Sources

- [GNU find manual](https://man7.org/linux/man-pages/man1/find.1.html): name/type tests, time buckets, expression precedence, and `-exec`.
- [GNU grep matching control](https://www.gnu.org/software/grep/manual/html_node/Matching-Control.html): `-e`, `-f`, case, whole words, and inverted selection.
- [GNU grep: basic vs extended regular expressions](https://www.gnu.org/software/grep/manual/html_node/Basic-vs-Extended.html): the syntax differences between BRE and ERE.
- [GNU grep regular-expression structure](https://www.gnu.org/software/grep/manual/html_node/Fundamental-Structure.html): repetition, empty matches, and GNU interval syntax.
- [GNU sort manual](https://www.gnu.org/software/coreutils/manual/html_node/sort-invocation.html): `-r`, `-i`, and `-n`.
- [GNU Coreutils manual](https://www.gnu.org/software/coreutils/manual/coreutils.html): `wc`, `head`, `tail`, and file operands.
- [GNU Bash manual](https://www.gnu.org/software/bash/manual/bash.html): quoting, expansion, and redirection before command execution.

[Back to top](#table-of-contents)