
# Find & Grep

##### Table of Contents

##### The find Command

**Basics**
- [Command Structure & Syntax](#command-structure--syntax)
- [Basic Examples (Simple Searches)](#basic-examples-simple-searches)
  - [Finding by Name](#finding-by-name)
  - [Finding by Type](#finding-by-type)
  - [Combining Name and Type](#combining-name-and-type)
  - [Using -print (Explicit Output)](#using--print-explicit-output)

**Size-Based Searches**
- [Size-Based Searches](#size-based-searches)
  - [Introduction to -size Option](#introduction-to--size-option)
  - [Size Units](#size-units)
  - [Size Operators](#size-operators)
  - [Greater Than or Equal / Less Than or Equal](#greater-than-or-equal--less-than-or-equal)
  - [Size Range: Finding Files Within a Range](#size-range-finding-files-within-a-range)
  - [Default Operator: -and (Implicit)](#default-operator--and-implicit)

**Logic Operations**
- [Combining Conditions with Logic Operators](#combining-conditions-with-logic-operators)
  - [Available Logic Operators](#available-logic-operators)
  - [Simple AND Examples](#simple-and-examples)
  - [Simple OR Examples](#simple-or-examples)
  - [CRITICAL: Operator Precedence](#critical-operator-precedence)
  - [What Applies First: -and or -or?](#what-applies-first--and-or--or)
  - [Using Parentheses to Control Order](#using-parentheses-to-control-order)
  - [Complex Example: Multiple Conditions with OR](#complex-example-multiple-conditions-with-or)
  - [Common Mistakes](#common-mistakes-logic)
  - [Summary: Operator Precedence Rules](#summary-operator-precedence-rules)

**Actions and Performance**
- [Actions on Found Files](#actions-on-found-files)
  - [What are Actions?](#what-are-actions)
  - [The -exec Action](#the--exec-action)
  - [Understanding the {} Placeholder](#understanding-the--placeholder)
  - [The Problem with \; - Performance Issue](#the-problem-with---performance-issue)
  - [The Solution: Using + for Batch Processing](#the-solution-using--for-batch-processing)
  - [Comparison: \; vs +](#comparison--vs-)
  - [When to Use \; vs +](#when-to-use--vs-)
  - [Best Practices: Avoiding -exec When Possible](#best-practices-avoiding--exec-when-possible)
  - [Common Mistakes](#common-mistakes-exec)
  - [Summary: Performance Guidelines](#summary-performance-guidelines)

**Edge Cases and Tricky Examples**
- [Tricky Path Specifications](#tricky-path-specifications)
- [Tricky Name Patterns](#tricky-name-patterns)
- [Tricky Type Tests](#tricky-type-tests)
- [Tricky Size Tests](#tricky-size-tests)
- [Tricky Logic Operations](#tricky-logic-operations)
- [Tricky -exec Examples](#tricky--exec-examples)
- [Tricky -delete Cases](#tricky--delete-cases)
- [Tricky -prune Cases](#tricky--prune-cases)
- [Special Edge Cases](#special-edge-cases)
- [Mind-Bending Combined Examples](#mind-bending-combined-examples)
- [Summary: Common Pitfalls](#summary-common-pitfalls)

##### The grep Command

**Basics**
- [Command Structure & Syntax](#grep-command-structure--syntax)
- [Basic Usage and Common Options](#grep-basic-usage-and-common-options)
  - [Searching in Files](#searching-in-files)
  - [Case Sensitivity](#case-sensitivity)
  - [Line Numbers and Counting](#line-numbers-and-counting)
  - [Inverted Matching](#inverted-matching)

**Word Boundaries and Pattern Matching**
- [Word Boundaries and Exact Matching](#word-boundaries-and-exact-matching)
  - [Pattern Contains vs Exact Word](#pattern-contains-vs-exact-word)
  - [Using -w Flag](#using--w-flag)
  - [Manual Word Boundaries with \< and \>](#manual-word-boundaries-with--and-)

**Regular Expressions - Character Classes**
- [Regular Expressions - Character Classes](#regular-expressions---character-classes)
  - [Wildcard Matching with .*](#wildcard-matching-with-)
  - [Character Ranges [A-Z], [a-z], [0-9]](#character-ranges-a-z-a-z-0-9)
  - [Combining Character Classes](#combining-character-classes)

**POSIX Character Classes**
- [POSIX Character Classes](#posix-character-classes)
  - [Using [[:alnum:]]](#using-alnum)
  - [Other POSIX Classes](#other-posix-classes)

**Character Ranges and ASCII**
- [Character Ranges and ASCII Considerations](#character-ranges-and-ascii-considerations)
  - [Invalid Ranges [a-Z]](#invalid-ranges-a-z)
  - [Valid but Unexpected [A-z]](#valid-but-unexpected-a-z)
  - [ASCII Table Explanation](#ascii-table-explanation)

**Quantifiers**
- [Quantifiers](#quantifiers)
  - [Exact Count {n}](#exact-count-n)
  - [Range Count {n,m}](#range-count-nm)
  - [Other Quantifiers: *, +, ?](#other-quantifiers---)

**Extended Regular Expressions**
- [Extended Regular Expressions (ERE)](#extended-regular-expressions-ere)
  - [Using -E Flag](#using--e-flag)
  - [Alternation with |](#alternation-with-)
  - [Differences Between BRE and ERE](#differences-between-bre-and-ere)

**Multiple Patterns**
- [Multiple Patterns](#multiple-patterns)
  - [Using -e Flag](#using--e-flag-multiple)
  - [Combining Multiple Patterns](#combining-multiple-patterns)

**Practical Examples**
- [Practical Examples and Use Cases](#practical-examples-and-use-cases)

**Edge Cases and Tricky Examples**
- [Common Pitfalls and Tricky Cases](#common-pitfalls-and-tricky-cases-grep)

---

##### Find Overview
##### Command Structure & Syntax

The `find` command is used to search for files and directories in a directory hierarchy based on various criteria.

##### Basic Anatomy

```
find [path] [options] [tests] [actions]
```

##### Components Explained

| Component | Description | Example |
|-----------|-------------|---------|
| **path** | Starting directory for the search | `/public`, `.`, `/home/user` |
| **options** | Modify how find operates | `-maxdepth 2`, `-mindepth 1` |
| **tests** | Criteria to match files/directories | `-name`, `-type`, `-size` |
| **actions** | What to do with matched files | `-print`, `-exec`, `-delete` |

##### Simple Structure Example

```bash
find /public -type f -name "*.txt"
```

Breaking this down:
- **find** - the command
- **/public** - path (search in /public directory)
- **-type f** - test (only files, not directories)
- **-name "*.txt"** - test (name matches pattern)
- **(implicit -print)** - action (display results)

##### Important Notes

1. **Path is required** - If omitted, find uses current directory (`.`)
   ```bash
   find . -name "test.txt"  # Search in current directory
   ```

2. **Tests can be combined** - Multiple criteria work together
   ```bash
   find /public -type f -name "*.log" -size +1M
   ```

3. **Default action is -print** - If no action specified, find prints matches
   ```bash
   find /public -name "*.txt"        # Prints results automatically
   find /public -name "*.txt" -print # Same thing, explicit
   ```

4. **Order matters for performance** - Put most restrictive tests first
   ```bash
   # By default (optimization level 1), find reorders tests:
   # - Name-based tests like -name are performed first
   # - Then -type tests
   # - Then tests requiring file stats (like -size)
   
   # You write this:
   find /public -type f -name "*.txt" -size +1b
   
   # But find may internally optimize it to check -name first,
   # then -type, then -size (which requires reading file stats)
   
   # You can see optimization with -D opt flag:
   find -D opt /public -type f -name "*.txt"
   ```


   
---

##### Basic Examples (Simple Searches)

This section introduces basic `find` usage with simple, practical examples.

##### Finding by Name

The `-name` option searches for files matching a pattern.

```bash
# Find a specific file by exact name
$ find /public -name "kw5.txt"
/public/ucebnove/kw5.txt

# Find files starting with "kw"
$ find /public -name "kw*"
/public/ucebnove/kw5.txt
/public/ucebnove/kw7.txt
/public/ucebnove/kw9.txt

# Find files ending with .txt
$ find /public -name "*.txt"

# Find files containing "kw" anywhere in the name
$ find /public -name "*kw*"
```

**Important notes about wildcards:**
- Always quote patterns with wildcards (`"*.txt"` not `*.txt`)
- Without quotes, your shell expands the pattern before `find` sees it
- `-name` only matches the filename, not the full path

##### Finding by Type

The `-type` option filters results by file type.

```bash
# Find only regular files
$ find /public -type f

# Find only directories
$ find /public -type d

# Find only symbolic links
$ find /public -type l
```

**Common type options:**
- `f` - regular file
- `d` - directory
- `l` - symbolic link
- `c` - character device
- `b` - block device

##### Combining Name and Type

You can combine multiple tests - all must be true for a match.

```bash
# Find files (not directories) starting with "kw"
$ find /public -type f -name "kw*"
/public/ucebnove/kw5.txt
/public/ucebnove/kw7.txt
/public/ucebnove/kw9.txt

# Find directories with "historia" in their name
$ find /public -type d -name "*historia*"
/public/ucebnove/historia
```

##### Default Operator: `-and` (Implicit)

**CRITICAL CONCEPT**: When you put tests next to each other without an operator, `find` assumes `-and` between them.

```bash
# These are EQUIVALENT:
$ find /public -type f -name "kw*" -size +70c
$ find /public -type f -and -name "kw*" -and -size +70c

# All three conditions must be true:
# 1. Must be a file (-type f)
# 2. AND name must match kw*
# 3. AND size must be greater than 70 bytes
```

**The implicit operator is `-and`** - this means ALL tests must be true for a file to match.

##### Using -print (Explicit Output)

By default, `find` prints results. You can make it explicit:

```bash
# These are equivalent:
$ find /public -type f -name "kw*"
$ find /public -type f -name "kw*" -print

# -print is implied when no other action is specified
```

**When to use `-print` explicitly:**
- When combining with other actions
- For clarity in complex commands
- When learning `find` syntax

---

##### Size-Based Searches

##### Introduction to `-size` Option

The `-size` option allows you to search for files based on their size. You can specify different units and use operators to find files larger, smaller, or exactly matching a certain size.

##### Size Units

| Unit | Description | Example |
|------|-------------|---------|
| `c` | Bytes (characters) | `-size 100c` = exactly 100 bytes |
| `k` | Kilobytes (1024 bytes) | `-size 5k` = exactly 5 KB |
| `M` | Megabytes (1024 KB) | `-size 2M` = exactly 2 MB |
| `G` | Gigabytes (1024 MB) | `-size 1G` = exactly 1 GB |
| `b` | Blocks (512 bytes) | `-size 10b` = exactly 10 blocks |

**Note**: If no unit is specified, blocks (512 bytes) are assumed.

##### Size Operators

| Operator | Meaning | Example | Description |
|----------|---------|---------|-------------|
| `+` | Greater than | `-size +50c` | Files larger than 50 bytes |
| `-` | Less than | `-size -100c` | Files smaller than 100 bytes |
| (none) | Exactly | `-size 70c` | Files exactly 70 bytes |

**Important**: The size is rounded UP to the next unit!
- `-size -1M` matches files from 0 to 1,048,575 bytes (NOT files less than 1MB)
- `-size +1M` matches files 1,048,576 bytes and larger

##### Basic Size Examples

```bash
# Find all files larger than 70 bytes
$ find /public -type f -size +70c

# Find all files smaller than 50 bytes
$ find /public -type f -size -50c

# Find all files exactly 100 bytes
$ find /public -type f -size 100c
```


##### Size Range: Finding Files Within a Range

To find files within a size range, you need to combine conditions:

```bash
# Find files between 20 and 30 bytes (exclusive of endpoints)
# Means: larger than 20 AND smaller than 30
$ find /public -type f -size +20c -size -30c

# Breaking it down:
# -size +20c     → files > 20 bytes
# (implicit -and)
# -size -30c     → files < 30 bytes
# Result: files where 20 < size < 30
```

```bash
# Find files that are 30 bytes or larger, up to but not including 50 bytes
$ find /public -type f -size +29c -size -50c

# Greater than or equal to 50 bytes (>= 50)
# Trick: +49c means "greater than 49", which includes 50 and above
$ find /public -type f -size +49c

# Less than or equal to 100 bytes (<= 100)
# Trick: -101c means "less than 101", which includes 100 and below
$ find /public -type f -size -101c

# Equal to 70 bytes (= 70)
$ find /public -type f -size 70c
```

Rule of thumb:

- For >= N, use -size +$(N-1)c
- For <= N, use -size -$(N+1)c
- For = N, use -size Nc


##### Practice Examples

Based on the images you provided:

```bash
# Example 1: Find all files named "kw*" larger than 70 bytes
$ find /public -type f -name "kw*" -size +70c -print
/public/ucebnove/kw5.txt
/public/ucebnove/kw9.txt

# Example 2: Find files in specific size range
# Files larger than 20 bytes but smaller than 30 bytes
$ find /public -type f -size +20c -size -30c
```

##### Key Takeaways

1. **Size units**: `c` (bytes), `k` (KB), `M` (MB), `G` (GB)
2. **Operators**: `+` (greater), `-` (less), no operator (exact)
3. **Default operator**: When tests are side-by-side, `-and` is assumed (ALL must be true)
4. **Size ranges**: Combine two `-size` tests with implicit `-and`
5. **Rounding**: Sizes are rounded UP to the next unit

---

##### Logic Operations

##### Combining Conditions with Logic Operators

##### Available Logic Operators

| Operator | Meaning | Description |
|----------|---------|-------------|
| `-and` (or `-a`) | Logical AND | Both conditions must be true |
| `-or` (or `-o`) | Logical OR | At least one condition must be true |
| `!` (or `-not`) | Logical NOT | Negates the condition |
| `\( \)` | Parentheses | Groups conditions to control evaluation order |

**Note**: When no operator is specified between tests, `-and` is assumed (implicit).

##### Simple AND Examples

```bash
# Explicit -and
$ find /public -type f -and -name "kw*" -and -size +70c

# Implicit -and (same as above)
$ find /public -type f -name "kw*" -size +70c

# Both mean: files that are regular files AND named kw* AND larger than 70 bytes
```

##### Simple OR Examples

```bash
# Find files with .txt OR .log extension
$ find /public -type f -name "*.txt" -or -name "*.log"

# Find files named kw5.txt OR kw7.txt
$ find /public -name "kw5.txt" -or -name "kw7.txt"
```

##### CRITICAL: Operator Precedence

**`-and` has HIGHER precedence than `-or`**

This is extremely important when combining both operators!

```bash
# Example: This may NOT work as you expect!
$ find /public -name "*.txt" -or -name "*.log" -type f

# How find interprets it (due to precedence):
$ find /public -name "*.txt" -or \( -name "*.log" -and -type f \)

# This means:
# - Match anything named *.txt (could be files OR directories!)
# - OR match things named *.log that are files

# To make BOTH be files, use explicit parentheses (covered below)
```

##### What Applies First: `-and` or `-or`?

**Answer: `-and` applies first!**

Think of it like multiplication and addition in math:
- In math: `2 + 3 × 4 = 2 + 12 = 14` (multiplication first)
- In find: `-or` and `-and` work similarly (`-and` binds tighter)

```bash
# Expression without parentheses:
$ find /public -name "*.txt" -or -name "*.log" -and -type f

# Is evaluated as (implicit grouping):
$ find /public -name "*.txt" -or \( -name "*.log" -and -type f \)

# Step-by-step evaluation:
# 1. First: -name "*.log" -and -type f (AND has higher precedence)
# 2. Then: -name "*.txt" -or (result from step 1)
```

##### Using Parentheses to Control Order

To control the order of evaluation, use parentheses `\( \)`:

**Important**: You MUST escape parentheses with backslash `\(` and `\)` or quote them!

```bash
# Wrong - shell will interpret parentheses:
$ find /public ( -name "*.txt" -or -name "*.log" ) -type f  # ERROR!

# Correct - escaped parentheses:
$ find /public \( -name "*.txt" -or -name "*.log" \) -and -type f

# This means:
# 1. First: evaluate -name "*.txt" -or -name "*.log"
# 2. Then: take result AND -type f
# Result: files that are (*.txt OR *.log) AND regular files
```

##### Complex Example: Multiple Conditions with OR

Let's break down this complex command:

```bash
$ find /public -type f \( \( -size +20c -and -size -30c \) -or -size +100c \)
```

**Breaking it down step by step:**

1. **Innermost parentheses**: `\( -size +20c -and -size -30c \)`
   - Files between 20 and 30 bytes (20 < size < 30)

2. **Outer OR**: `\( ... \) -or -size +100c`
   - Files from step 1 OR files larger than 100 bytes

3. **Final AND with -type f**: The entire expression must be true AND it must be a file

**What this finds:**
- Regular files that are either:
  - Between 20 and 30 bytes, OR
  - Larger than 100 bytes

##### Visual Representation

```
find /public -type f \( \( -size +20c -and -size -30c \) -or -size +100c \)
                 │                    │              │               │
                 │                    └──────────────┘               │
                 │                         Group 1                   │
                 │                    (20 < size < 30)               │
                 │                                                   │
                 │    ┌──────────────────────────────────────────────┘
                 │    │              Group 1 OR size > 100
                 │    └──────────────────────────────────────────────┐
                 │                                                   │
                 └───────────────────────────────────────────────────┘
                        Must be a file AND (Group 1 OR size > 100)
```

##### Practice Examples

##### Example 1: Files in size ranges
```bash
# Find files that are:
# - Between 30 and 50 bytes (30 < size < 50), OR
# - Larger than 100 bytes

$ find /public -type f \( -size +30c -size -50c \) -or -size +100c
```

Wait, this has a problem! Let me fix it:

```bash
# Correct version with proper grouping:
$ find /public -type f \( \( -size +30c -and -size -50c \) -or -size +100c \)
```

##### Example 2: Multiple name patterns
```bash
# Find files that are:
# - Named kw* OR named *.txt
# - AND are regular files

$ find /public \( -name "kw*" -or -name "*.txt" \) -and -type f
```

##### Example 3: Complex with NOT
```bash
# Find files that are:
# - Larger than 50 bytes
# - NOT named *.log

$ find /public -type f -size +50c ! -name "*.log"
```

##### Common Mistakes

##### Mistake 1: Forgetting parentheses precedence
```bash
# WRONG - probably not what you want:
$ find /public -name "*.txt" -or -name "*.log" -type f

# This matches:
# - Anything named *.txt (files OR directories)
# - OR files named *.log

# CORRECT - if you want both to be files:
$ find /public \( -name "*.txt" -or -name "*.log" \) -and -type f
```

##### Mistake 2: Not escaping parentheses
```bash
# WRONG - syntax error:
$ find /public ( -name "*.txt" -or -name "*.log" ) -type f

# CORRECT:
$ find /public \( -name "*.txt" -or -name "*.log" \) -type f
```

##### Summary: Operator Precedence Rules

1. **Parentheses `\( \)`** - Highest precedence (evaluated first)
2. **`-not` or `!`** - Second highest
3. **`-and` or `-a`** - Third (higher than `-or`)
4. **`-or` or `-o`** - Lowest precedence (evaluated last)

**Remember**: When in doubt, use parentheses to make your intent clear!

##### Quick Reference

```bash
# AND (all must be true) - implicit
find /path -test1 -test2 -test3

# OR (at least one must be true) - explicit
find /path -test1 -or -test2

# Complex with parentheses
find /path \( -test1 -or -test2 \) -and -test3

# NOT (negation)
find /path ! -test1
find /path -not -test1
```

---

##### Actions on Found Files

##### What are Actions?

Actions tell `find` what to do with the files it finds. The default action is `-print` (display the filename).

##### Common Actions

| Action | Description |
|--------|-------------|
| `-print` | Print the file path (default) |
| `-print0` | Print with null character separator (safer for special filenames) |
| `-ls` | Print file details like `ls -l` |
| `-delete` | Delete the found files |
| `-exec command \;` | Execute command for each file (one at a time) |
| `-exec command {} +` | Execute command with multiple files at once (batch mode) |

##### The `-exec` Action

The `-exec` action allows you to run any command on the files found by `find`.

##### Basic Syntax

```bash
find [path] [tests] -exec command {} \;
```

**Components:**
- `command` - Any shell command you want to run
- `{}` - Placeholder that gets replaced with the filename
- `\;` - Marks the end of the `-exec` command (must be escaped!)

##### Simple Example

```bash
# Run 'ls -la' on each file found
$ find /public -type f -name "kw*" -exec ls -la {} \;

# This will execute:
# ls -la /public/ucebnove/kw5.txt
# ls -la /public/ucebnove/kw7.txt
# ls -la /public/ucebnove/kw9.txt
```

##### The Problem with `\;` - Performance Issue

**CRITICAL PERFORMANCE PROBLEM**: Using `-exec command {} \;` creates a NEW PROCESS for EVERY file found!

```bash
# BAD - Creates one process per file (very slow with many files!)
$ find /public -type f -name "kw*" -exec ls -la {} \;

# If find matches 1000 files:
# - Creates 1000 separate processes
# - Each process startup has overhead
# - This exhausts system resources!
```

**Why is this a problem?**
1. **Process creation overhead** - Starting a new process takes time and resources
2. **System resource exhaustion** - Too many processes can slow down or crash the system
3. **Slow execution** - With thousands of files, this becomes extremely slow

##### The Solution: Using `+` for Batch Processing

**RECOMMENDED**: Use `-exec command {} +` to process multiple files in one command!

```bash
# GOOD - Batch processing (much faster!)
$ find /public -type f -name "kw*" -exec ls -la {} +

# If find matches 1000 files, it might execute:
# ls -la file1 file2 file3 ... file50    (process 1)
# ls -la file51 file52 ... file100       (process 2)
# ... and so on
# Instead of 1000 processes, maybe only 20!
```

**How `+` works:**
- Collects multiple filenames
- Passes them all to ONE command invocation
- Repeats until all files are processed
- Much more efficient!

##### Comparison: `\;` vs `+`

```bash
# Using \; (SLOW - one process per file)
$ find /public -type f -exec ls -l {} \;
# Executes:
# ls -l /public/file1.txt
# ls -l /public/file2.txt
# ls -l /public/file3.txt
# (3 separate processes)

# Using + (FAST - batch processing)
$ find /public -type f -exec ls -l {} +
# Executes:
# ls -l /public/file1.txt /public/file2.txt /public/file3.txt
# (1 process with multiple arguments)
```

##### Visual Example: Process Count

```
With \; (one file per process):
┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐
│Process 1│  │Process 2│  │Process 3│  │Process N│  ... (1000 processes!)
│ls file1 │  │ls file2 │  │ls file3 │  │ls fileN │
└─────────┘  └─────────┘  └─────────┘  └─────────┘

With + (batch processing):
┌──────────────────────────────┐  ┌──────────────────────────────┐
│       Process 1               │  │       Process 2              │  ... (20 processes)
│ ls file1 file2 ... file50     │  │ ls file51 file52 ... file100 │
└──────────────────────────────┘  └──────────────────────────────┘
```

##### When to Use `\;` vs `+`

##### Use `\;` when:
- You need to process each file individually
- The command doesn't support multiple arguments
- You need different behavior for each file


##### Use `+` when:
- The command can handle multiple files
- Performance matters (almost always!)
- You have many files to process

```bash
# Example: Delete multiple files efficiently
$ find /public -name "*.tmp" -exec rm {} +

# Example: Change permissions on many files
$ find /public -name "*.sh" -exec chmod +x {} +
```

##### Real-World Performance Example

```bash
# Scenario: You have 200,000 files to process

# BAD APPROACH - Using \;
$ time find /public -type f -exec basename {} \;
# Result: ~35 minutes (creates 200,000 processes!)

# GOOD APPROACH - Using + with -printf
$ time find /public -type f -printf '%f\n'
# Result: ~30 seconds (no external processes!)
```

##### Best Practices: Avoiding `-exec` When Possible

Sometimes you don't need `-exec` at all!

```bash
# Instead of this:
$ find /public -name "*.txt" -exec basename {} \;

# Use find's built-in options:
$ find /public -name "*.txt" -printf '%f\n'
# Much faster! No external processes needed.
```

##### Set Limits to Avoid System Exhaustion

If you MUST use `\;`, consider limiting the number of results:

```bash
# Limit to first 100 files
$ find /public -name "kw*" -exec ls -l {} \; | head -100

# Or use -maxdepth to limit search depth
$ find /public -maxdepth 2 -name "kw*" -exec ls -l {} \;
```

##### Practice Examples

##### Example 1: Find and list files efficiently
```bash
# Find all .txt files and list them with details (EFFICIENT)
$ find /public -type f -name "*.txt" -exec ls -lh {} +
```

##### Example 2: Find and delete temporary files
```bash
# Find and delete .tmp files (EFFICIENT)
$ find /public -type f -name "*.tmp" -exec rm {} +

# Or use the built-in -delete action (even better!)
$ find /public -type f -name "*.tmp" -delete
```

##### Example 3: Find files and change permissions
```bash
# Make all .sh files executable (EFFICIENT)
$ find /public -type f -name "*.sh" -exec chmod +x {} +
```

##### Example 4: Complex example with size conditions
```bash
# Find files in certain size range and list them
$ find /public -type f \( \( -size +20c -size -30c \) -or -size +100c \) -exec ls -lh {} +
```

##### Common Mistakes

##### Mistake 1: Forgetting to escape the semicolon
```bash
# WRONG - shell interprets ; as command separator
$ find /public -name "*.txt" -exec ls -l {} ;

# CORRECT
$ find /public -name "*.txt" -exec ls -l {} \;
```

##### Mistake 2: Using \; with many files
```bash
# INEFFICIENT - creates too many processes
$ find /large_directory -type f -exec command {} \;

# EFFICIENT - batch processing
$ find /large_directory -type f -exec command {} +
```

##### Mistake 3: Forgetting the `{}` placeholder
```bash
# WRONG - command doesn't know which file to process
$ find /public -name "*.txt" -exec ls -l \;

# CORRECT
$ find /public -name "*.txt" -exec ls -l {} \;
```

##### Summary: Performance Guidelines

1. **Prefer `+` over `\;`** - Almost always faster
2. **Use built-in options** - `-printf`, `-delete` instead of `-exec`
3. **Limit results** - Use `-maxdepth`, specific tests to reduce matches
4. **Test first** - Use `-print` to see what will be affected before using `-exec` or `-delete`
5. **Consider alternatives** - Sometimes `xargs` or `while read` loops are better

##### Quick Reference

```bash
# Print results (default)
find /path -name "pattern"

# Execute command per file (SLOW)
find /path -name "pattern" -exec command {} \;

# Execute command in batches (FAST)
find /path -name "pattern" -exec command {} +

# Use built-in actions when possible (FASTEST)
find /path -name "pattern" -delete
find /path -name "pattern" -printf '%f\n'
```

---


##### Edge Cases with `find`

##### 1. Tricky Path Specifications

##### Case 1: Current Directory vs Absolute Path

```bash
# These look similar but behave differently:
$ find . -name "*.txt"
./file.txt
./subdir/file.txt

$ find /home/user -name "*.txt"
/home/user/file.txt
/home/user/subdir/file.txt

# Key difference: Output paths are relative vs absolute
```

##### Case 2: Multiple Starting Points

```bash
# You can search multiple directories at once:
$ find /public /tmp /home -name "kw*"

# This searches in ALL three directories
# Useful for finding files across different locations
```

---

##### 2. Tricky Name Patterns

##### Case 1: Hidden Files (Starting with .)

```bash
# Does NOT find hidden files by default in some contexts
$ find /public -name "*.txt"

# To explicitly find hidden .txt files:
$ find /public -name ".*.txt"

# To find ALL files including hidden:
$ find /public -name ".*" -o -name "*"
```

##### Case 2: Case Sensitivity

```bash
# Case-sensitive (default)
$ find /public -name "README.txt"
# Finds: README.txt
# Does NOT find: readme.txt, ReadMe.txt

# Case-insensitive
$ find /public -iname "README.txt"
# Finds: README.txt, readme.txt, ReadMe.txt, README.TXT
```

##### Case 3: Wildcards in Unexpected Places

```bash
# Wildcard at the beginning (finds files ENDING with .txt)
$ find /public -name "*.txt"

# Wildcard in the middle
$ find /public -name "kw*5.txt"
# Finds: kw5.txt, kw12345.txt, kw_test_5.txt

# Multiple wildcards
$ find /public -name "*kw*txt*"
# Very broad! Matches: test_kw_file.txt.backup
```

##### Case 4: The Quote Problem

```bash
# WRONG - shell expands * before find sees it
$ find /public -name *.txt
# If current directory has file.txt, this becomes:
# find /public -name file.txt
# Only finds files named exactly "file.txt"!

# CORRECT - quotes protect the wildcard
$ find /public -name "*.txt"
$ find /public -name '*.txt'
```

---

##### 3. Tricky Type Tests

##### Case 1: Broken Symbolic Links

```bash
# Regular -type l finds symbolic links
$ find /public -type l

# But what about BROKEN symbolic links?
# -type l shows all symlinks (working or broken)
$ find /public -type l

# To find ONLY broken symlinks:
$ find /public -type l ! -readable
# Or:
$ find /public -type l ! -exec test -e {} \; -print
```

##### Case 2: Type with OR - A Common Mistake

```bash
# WRONG - This doesn't work as expected!
$ find /public -type f -or -type d -name "*.txt"
# Due to precedence, this means:
# (-type f) OR (-type d AND -name "*.txt")
# Finds ALL files, plus directories named *.txt

# CORRECT - Use parentheses:
$ find /public \( -type f -or -type d \) -name "*.txt"
```

##### Case 3: Multiple Types

```bash
# GNU find allows comma-separated types:
$ find /public -type f,l
# Finds regular files AND symbolic links

# Equivalent to:
$ find /public \( -type f -or -type l \)
```

---

##### 4. Tricky Size Tests

##### Case 1: Size Rounding Surprise

```bash
# Files are rounded UP to the next unit!
$ find /public -size 1k

# This finds files from 1 byte to 1024 bytes
# NOT files that are exactly 1024 bytes!

# A file of 1025 bytes is -size 2k
# A file of 1 byte is -size 1k
```

##### Case 2: Empty Files

```bash
# Empty files have size 0
$ find /public -type f -size 0

# But this also works:
$ find /public -type f -empty

# -empty works for directories too (empty directories)
$ find /public -type d -empty
```

##### Case 3: The Negative Size Trap

```bash
# This does NOT find files smaller than 1MB!
$ find /public -size -1M
# Finds files from 0 to 1,048,575 bytes (0 to 1MB - 1 byte)

# This finds files 1MB and larger:
$ find /public -size +1M
# Finds files 1,048,576 bytes and up
```

##### Case 4: Combining Size Ranges - Order Matters?

```bash
# These are equivalent (order doesn't matter with AND):
$ find /public -size +20c -size -30c
$ find /public -size -30c -size +20c

# Both find: 20 < size < 30 bytes
```

##### Case 5: Size with Different Units

```bash
# Be careful mixing units!
$ find /public -size +1k -size -1M
# Finds: 1KB < size < 1MB

# This is NOT the same as:
$ find /public -size +1024c -size -1048576c
# Due to rounding, results can differ slightly!
```

---

##### 5. Tricky Logic Operations

##### Case 1: Implicit AND Can Surprise You

```bash
# This finds .txt files that are also directories (impossible!)
$ find /public -name "*.txt" -type d
# Returns nothing - files can't be both .txt AND directories

# Be careful with your logic!
```

##### Case 2: The OR Trap with -print

```bash
# This prints EVERYTHING!
$ find /public -name "*.txt" -or -name "*.log" -print
# Why? -print only applies to the second part!
# Interpreted as: (-name "*.txt") OR (-name "*.log" AND -print)

# CORRECT - put -print at the end:
$ find /public \( -name "*.txt" -or -name "*.log" \) -print
# Or rely on implicit -print (don't specify it at all)
```

##### Case 3: Double Negative Confusion

```bash
# Find files that are NOT .txt
$ find /public ! -name "*.txt"

# Find files that are NOT .txt AND NOT .log
$ find /public ! -name "*.txt" ! -name "*.log"

# But this is confusing - equivalent using De Morgan's law:
$ find /public ! \( -name "*.txt" -or -name "*.log" \)
```

##### Case 4: The Precedence Nightmare

```bash
# What does this find?
$ find /public -name "*.txt" -or -name "*.log" -type f -size +100c

# Due to precedence (-and before -or):
# (-name "*.txt") OR (-name "*.log" AND -type f AND -size +100c)

# Finds:
# - ANY .txt file (could be directory!)
# - .log FILES larger than 100 bytes

# Probably NOT what you want!
```

##### Case 5: Empty OR - A Weird Edge Case

```bash
# This always matches!
$ find /public -name "*.txt" -or -true
# Because -true is always true, so the OR is always satisfied

# This never matches:
$ find /public -name "*.txt" -and -false
```

---

##### 6. Tricky -exec Examples

##### Case 1: The Semicolon Escape Variants

```bash
# All of these work:
$ find /public -name "*.txt" -exec ls {} \;
$ find /public -name "*.txt" -exec ls {} ';'
$ find /public -name "*.txt" -exec ls {} ";"

# Why? They all protect ; from the shell
```

##### Case 2: Shell Commands in -exec

```bash
# WRONG - This doesn't work!
$ find /public -name "*.txt" -exec echo "Found: {}" >> log.txt \;
# The >> is interpreted by find, not the shell!

# CORRECT - Use sh -c:
$ find /public -name "*.txt" -exec sh -c 'echo "Found: $1" >> log.txt' _ {} \;
# The _ is a placeholder for $0, {} becomes $1
```

##### Case 3: Multiple Commands in -exec

```bash
# WRONG - Can't chain commands directly:
$ find /public -name "*.txt" -exec ls {} && cat {} \;

# CORRECT - Use sh -c:
$ find /public -name "*.txt" -exec sh -c 'ls "$1" && cat "$1"' _ {} \;
```

##### Case 4: Using {} with +, Position Restriction

```bash
# CORRECT - {} at the end:
$ find /public -name "*.txt" -exec grep "error" {} +

# WRONG - {} not at the end:
$ find /public -name "*.txt" -exec grep "error" {} + file.log
# ERROR! With +, {} must be immediately before +
```

##### Case 5: Exit Status Affects Results

```bash
# -exec returns true only if command exits with status 0
$ find /public -name "*.txt" -exec grep -q "TODO" {} \; -print

# Prints only files where grep found "TODO"
# If grep doesn't find "TODO", -exec returns false, -print doesn't execute!
```

##### Case 6: The -exec with -or Trap

```bash
# WRONG - What gets executed?
$ find /public -name "*.txt" -or -name "*.log" -exec rm {} \;

# Due to precedence:
# (-name "*.txt") OR (-name "*.log" AND -exec rm)
# Only .log files are deleted! .txt files just match and print!

# CORRECT:
$ find /public \( -name "*.txt" -or -name "*.log" \) -exec rm {} \;
```

---

##### 7. Tricky -delete Cases

##### Case 1: -delete Implies -depth

```bash
# When using -delete, find automatically uses depth-first traversal
$ find /public -name "*.tmp" -delete

# This is important! Otherwise, find might try to delete a directory
# before deleting its contents (which would fail)
```

##### Case 2: -delete with OR is Dangerous

```bash
# DANGEROUS! What gets deleted?
$ find /public -name "*.txt" -or -delete

# Due to precedence: (-name "*.txt") OR (-delete)
# Deletes EVERYTHING that doesn't match *.txt!

# ALWAYS test with -print first:
$ find /public -name "*.txt" -or -name "*.log" -print
# Then add -delete:
$ find /public \( -name "*.txt" -or -name "*.log" \) -delete
```

##### Case 3: -delete Cannot Be Combined with +

```bash
# WRONG - -delete doesn't support batch mode:
$ find /public -name "*.tmp" -delete +  # ERROR!

# CORRECT - -delete works file by file:
$ find /public -name "*.tmp" -delete
```

---

##### 8. Tricky -prune Cases

##### Case 1: Excluding Directories

```bash
# Skip the .git directory and everything in it:
$ find /public -name ".git" -prune -or -name "*.txt" -print

# Breakdown:
# -name ".git" -prune  → If it's .git, prune (don't descend) and return true
# -or                   → OR
# -name "*.txt" -print  → Print .txt files
```

##### Case 2: -prune Doesn't Work with -depth

```bash
# -prune has no effect when -depth is used!
$ find /public -depth -name ".git" -prune -or -print
# The -prune does nothing! -depth visits children first.
```

##### Case 3: Multiple -prune Conditions

```bash
# Skip multiple directories:
$ find /public \( -name ".git" -or -name "node_modules" -or -name ".cache" \) -prune -or -name "*.js" -print
```

---

##### 9. Special Edge Cases

##### Case 1: Finding Files Modified "Today"

```bash
# Files modified in the last 24 hours:
$ find /public -mtime 0

# Files modified between 24-48 hours ago:
$ find /public -mtime 1

# -mtime uses 24-hour periods, and fractional parts are ignored!
```

##### Case 2: Empty -name Pattern

```bash
# What happens here?
$ find /public -name ""
# Finds nothing - empty pattern matches nothing
```

##### Case 3: Root Directory /

```bash
# Finding in root requires special care:
$ find / -name "*.txt"
# This searches EVERYTHING - very slow!
# Might also hit permission errors

# Better:
$ find / -name "*.txt" 2>/dev/null
# Suppresses permission denied errors
```

##### Case 4: Files with Spaces in Names

```bash
# File: "my file.txt"
# This breaks with spaces:
$ find /public -name "*.txt" -exec rm {} \;  # Works fine actually

# But this breaks:
$ for f in $(find /public -name "*.txt"); do rm $f; done
# WRONG! Treats "my" and "file.txt" as separate

# CORRECT:
$ find /public -name "*.txt" -print0 | xargs -0 rm
# -print0 uses null separator, -0 reads null-separated input
```

##### Case 5: Maximum Depth Limit

```bash
# Search only in the specified directory (not subdirectories):
$ find /public -maxdepth 1 -name "*.txt"

# Search only in subdirectories (exclude the start point itself):
$ find /public -mindepth 1 -name "*.txt"

# Specific depth only:
$ find /public -mindepth 2 -maxdepth 2 -name "*.txt"
# Only searches exactly 2 levels deep
```

---

##### 10. Mind-Bending Combined Examples

##### Example 1: The Everything Paradox

```bash
# This matches everything:
$ find /public -name "*"

# But this matches nothing:
$ find /public -name "" 
```

##### Example 2: Complex Size Logic

```bash
# Files that are:
# - (Between 1KB and 2KB) OR (Exactly 5KB) OR (Greater than 10KB)
$ find /public -type f \( \( -size +1k -size -2k \) -or -size 5k -or -size +10k \)
```

##### Example 3: Permission Paradox

```bash
# Files that are readable but not writable by owner:
$ find /public -type f -perm -u=r ! -perm -u=w

# Files that are executable by anyone:
$ find /public -type f -perm /a=x
```

##### Example 4: Time-Based Complex Query

```bash
# Files modified in the last 7 days but not in the last 24 hours:
$ find /public -type f -mtime -7 ! -mtime 0
```

##### Example 5: The Ultimate Complex Find

```bash
# Find files that are:
# - Regular files OR symbolic links
# - Named *.txt OR *.log
# - (Size between 10KB-50KB) OR (larger than 1MB)
# - Not in .git directories
# - Modified in the last 30 days
# - Execute ls on them in batch mode

$ find /public \
    \( -type f -or -type l \) \
    \( -name "*.txt" -or -name "*.log" \) \
    \( \( -size +10k -size -50k \) -or -size +1M \) \
    ! -path "*/.git/*" \
    -mtime -30 \
    -exec ls -lh {} +
```

---

##### Summary: Common Pitfalls

1. **Forgetting quotes** around wildcards
2. **Ignoring operator precedence** (-and before -or)
3. **Using -delete or -exec without testing** with -print first
4. **Not escaping semicolons** in -exec
5. **Mixing up size rounding** semantics
6. **Forgetting {} position rules** with +
7. **Not using parentheses** for complex logic
8. **Assuming left-to-right evaluation** (precedence matters!)
9. **Using -prune with -depth** (doesn't work)
10. **Not handling filenames with spaces** properly

**Golden Rule**: When in doubt, test with `-print` first, then add your action!

---

# Grep
## Command Structure & Syntax

The `grep` command (Global Regular Expression Print) is used to search for text patterns in files or input streams.

### Basic Anatomy

```
grep [options] pattern file1 [file2 ...]
```

### Components Explained

| Component | Description | Example |
|-----------|-------------|---------|
| **grep** | The command itself | `grep` |
| **options** | Flags that modify behavior | `-i`, `-n`, `-c`, `-v`, `-r` |
| **pattern** | The text or regex pattern to search for | `"error"`, `"system"`, `"\<s.*\>"` |
| **file(s)** | File(s) to search in (optional - can use stdin) | `logfile.txt`, `*.txt` |

### Simple Structure Example

```bash
grep "error" logfile.txt
```

Breaking this down:
- **grep** - the command
- **"error"** - pattern to search for
- **logfile.txt** - file to search in

### Important Notes

1. **Pattern quoting is recommended** - Always quote patterns to avoid shell interpretation
   ```bash
   grep "error" file.txt        # Good
   grep error file.txt          # Works, but risky with special characters
   ```

2. **Multiple files can be searched** - grep will search all specified files
   ```bash
   grep "error" file1.txt file2.txt file3.txt
   grep "error" *.txt           # Search all .txt files
   ```

3. **stdin input is supported** - grep can read from pipes
   ```bash
   cat file.txt | grep "error"
   ps aux | grep "apache"
   ```

4. **Pattern is a regular expression by default** - grep uses basic regular expressions (BRE)
   ```bash
   grep "err.*" file.txt        # Matches "error", "err123", etc.
   ```

---

## Basic Usage and Common Options

This section covers the fundamental options you'll use most frequently with grep.

### Searching in Files

#### Search for a Word in a Single File

```bash
# Find lines containing "udrziava" in zaciatocnik.txt
$ grep "udrziava" /public/zaciatocnik.txt
- OS udrziava pre kazdy proces jednu strukturu "user".

# The matching lines are printed to stdout
```

#### Search in Multiple Files

```bash
# Search for "brucho" in all .txt files
$ grep "brucho" /public/samples/*.txt
zamestnanci.txt:brucho ceo  23000

# Notice: filename is prefixed to each matching line
```

#### Search in All Files in a Directory

```bash
# Search for "TODO" in all files in a directory
$ grep "TODO" /path/to/directory/*
/path/to/directory/script.sh:# TODO: Fix this bug
/path/to/directory/README.md:TODO: Add documentation
```

#### Recursive Search

```bash
# Search recursively in directory and all subdirectories
$ grep -r "error" /var/log/
# Or use -R (follows symbolic links)
$ grep -R "error" /var/log/
```

---

### Case Sensitivity

By default, grep is **case-sensitive**. Use `-i` for case-insensitive search.

#### Case-Sensitive Search (Default)

```bash
# Find "error" (lowercase only)
$ grep "error" logfile.txt
error in line 42
system error

# Does NOT match: "Error", "ERROR", "ErRoR"
```

#### Case-Insensitive Search with `-i`

```bash
# Find "error" regardless of case
$ grep -i "error" logfile.txt
Error: Connection failed
error in line 42
System ERROR occurred
ErRoR in parsing

# Matches: error, Error, ERROR, ErRoR, etc.
```

**When to use `-i`:**
- Searching log files where case varies
- User input searches
- When you want all variations of a word

---

### Line Numbers and Counting

#### Show Line Numbers with `-n`

The `-n` option displays the line number where the match was found.

```bash
# Show line numbers for matches
$ grep -n "error" logfile.txt
5:Connection error
12:System error occurred
28:fatal error in module

# Format: line_number:matching_line
```

#### Combining `-n` with Other Options

```bash
# Case-insensitive search with line numbers
$ grep -in "error" logfile.txt
5:Connection Error
12:system error occurred
28:Fatal ERROR in module
```

#### Count Matching Lines with `-c`

The `-c` option counts how many lines contain the pattern (not how many matches).

```bash
# Count lines containing "error"
$ grep -c "error" logfile.txt
3

# Just shows the count, not the lines themselves
```

#### Count in Multiple Files

```bash
# Count matches in multiple files
$ grep -c "error" *.txt
file1.txt:5
file2.txt:12
file3.txt:0
```

**Important distinction:**
- `-c` counts **lines** that contain the pattern
- If a line has the pattern twice, it still counts as 1

---

### Inverted Matching

The `-v` option inverts the match - it shows lines that do **NOT** match the pattern.

#### Basic Inverted Match

```bash
# Show all lines that do NOT contain "error"
$ grep -v "error" logfile.txt
System started successfully
Processing request
Connection established
Task completed

# All lines WITHOUT "error" are displayed
```

#### Practical Use Cases for `-v`

```bash
# Remove comment lines from a config file
$ grep -v "^#" config.conf

# Show all processes except grep itself
$ ps aux | grep "apache" | grep -v "grep"

# Find files that don't have "test" in their name
$ ls | grep -v "test"
```

#### Combining `-v` with Other Options

```bash
# Show line numbers of lines NOT containing "error"
$ grep -vn "error" logfile.txt
1:System started
3:Processing request
4:Connection established

# Case-insensitive inverted match
$ grep -vi "error" logfile.txt
```

---

## Combining Options

You can combine multiple options together:

```bash
# Case-insensitive search with line numbers
$ grep -in "error" logfile.txt

# Recursive, case-insensitive search with line numbers
$ grep -rin "TODO" /path/to/project/

# Count matches recursively, case-insensitive
$ grep -ric "error" /var/log/

# Inverted match with line numbers
$ grep -vn "^#" config.conf
```

---

## Summary of Basic Options

| Option | Description | Example |
|--------|-------------|---------|
| `-i` | Case-insensitive search | `grep -i "error" file.txt` |
| `-n` | Show line numbers | `grep -n "error" file.txt` |
| `-c` | Count matching lines | `grep -c "error" file.txt` |
| `-v` | Invert match (show non-matching lines) | `grep -v "error" file.txt` |
| `-r` or `-R` | Recursive search | `grep -r "error" /path/` |
| `-l` | Show only filenames with matches | `grep -l "error" *.txt` |
| `-h` | Suppress filename prefix | `grep -h "error" *.txt` |
| `-w` | Match whole words only | `grep -w "error" file.txt` |

---

## Practice Examples

```bash
# Example 1: Find all errors in log file with line numbers
$ grep -n "error" /var/log/bootstrap.log

# Example 2: Count how many times "warning" appears (case-insensitive)
$ grep -ic "warning" /var/log/bootstrap.log

# Example 3: Find all non-empty, non-comment lines in a config file
$ grep -v "^#" /var/log/bootstrap.log | grep -v "^$"

# Example 4: List all files containing "error" (just filenames)
$ grep -l "error" /var/log/*.log
```

---

## Word Boundaries and Exact Matching

When searching for text, you often need to distinguish between:
- Finding a pattern **anywhere** in a word (contains)
- Finding an **exact word** only (whole word match)

### Pattern Contains vs Exact Word

#### Finding Pattern Anywhere (Default Behavior)

By default, grep finds the pattern **anywhere** it appears, even as part of another word.

```bash
# Find "system" anywhere it appears
$ grep "system" /public/zaciatocnik.txt
filesystem
system
subsystem
systematic

# This matches:
# - "system" as a standalone word
# - "system" as part of "filesystem"
# - "system" as part of "subsystem"
# - "system" as part of "systematic"
```

**Key point:** The pattern "system" matches whenever those characters appear in sequence, regardless of what's around them.

---

### Using -w Flag

The `-w` flag tells grep to match only **whole words** - the pattern must be surrounded by non-word characters.

#### What is a Word Boundary?

A word boundary is the position between:
- A word character (`[A-Za-z0-9_]`)
- A non-word character (space, punctuation, newline, etc.)

#### Exact Word Match with `-w`

```bash
# Find exact word "system" only
$ grep -w "system" /public/zaciatocnik.txt
system

# This matches:
# - "system" as a standalone word
# This does NOT match:
# - "filesystem" (system is part of a larger word)
# - "subsystem" (system is part of a larger word)
# - "systematic" (system is part of a larger word)
```

#### Comparison: With and Without `-w`

```bash
# WITHOUT -w (contains)
$ grep "system" /public/zaciatocnik.txt
filesystem
system
subsystem
systematic

# WITH -w (exact word)
$ grep -w "system" /public/zaciatocnik.txt
system
```

---

### Manual Word Boundaries with \< and \>

Instead of using `-w`, you can manually specify word boundaries using regex anchors:
- `\<` - Matches the start of a word
- `\>` - Matches the end of a word

#### Both Boundaries: \<pattern\>

```bash
# Equivalent to -w flag
$ grep "\<system\>" /public/zaciatocnik.txt
system

# Same as: grep -w "system" /public/zaciatocnik.txt
```

**This matches:**
- `system` (word by itself)

**This does NOT match:**
- `filesystem` (no word boundary before 'system')
- `subsystem` (no word boundary before 'system')
- `systematic` (no word boundary after 'system')

---

#### One-Sided Boundary: Start Only

Using only the start boundary `\<` matches words that **start with** the pattern.

```bash
# Find words that START with "system"
$ grep "\<system" /public/zaciatocnik.txt
system
systematic

# This matches:
# - "system" (starts with system, ends with system)
# - "systematic" (starts with system, continues)

# This does NOT match:
# - "filesystem" (doesn't start with system)
# - "subsystem" (doesn't start with system)
```

#### One-Sided Boundary: End Only

Using only the end boundary `\>` matches words that **end with** the pattern.

```bash
# Find words that END with "system"
$ grep "system\>" /public/zaciatocnik.txt
filesystem
system
subsystem

# This matches:
# - "system" (ends with system)
# - "filesystem" (ends with system)
# - "subsystem" (ends with system)

# This does NOT match:
# - "systematic" (doesn't end with system)
```

---

### Visual Comparison

```
Pattern: "system"

Text: "The filesystem and subsystem are systematic"

grep "system"
      ↓↓↓↓↓↓        ↓↓↓↓↓↓          ↓↓↓↓↓↓
     filesystem    subsystem       systematic
     (matches)     (matches)       (matches)

grep -w "system" or grep "\<system\>"
     No match      No match        No match
     (part of      (part of        (part of
      word)         word)           word)

grep "\<system"
     No match      No match        ↓↓↓↓↓↓
                                  systematic
                                  (starts with system)

grep "system\>"
      ↓↓↓↓↓↓        ↓↓↓↓↓↓          No match
     filesystem    subsystem
     (ends with    (ends with
      system)       system)
```

---

### When to Use Each Method

| Method | Use Case | Example |
|--------|----------|---------|
| `grep "pattern"` | Find pattern anywhere | `grep "system" /public/zaciatocnik.txt` |
| `grep -w "pattern"` | Find exact word only | `grep -w "system" /public/zaciatocnik.txt` |
| `grep "\<pattern\>"` | Find exact word (manual) | `grep "\<system\>" /public/zaciatocnik.txt` |
| `grep "\<pattern"` | Find words starting with pattern | `grep "\<system" /public/zaciatocnik.txt` |
| `grep "pattern\>"` | Find words ending with pattern | `grep "system\>" /public/zaciatocnik.txt` |

---

### Practice Examples

```bash
# Example 1: Find exact word "sa" (not "subor" which contains "sa")
$ grep "\<sa\>" /public/zaciatocnik.txt

# Example 2: Find words starting with "s"
$ grep "\<s" /public/zaciatocnik.txt

# Example 3: Find words ending with "or"
$ grep "or\>" /public/zaciatocnik.txt

# Example 4: Find exact word "system" case-insensitive
$ grep -iw "system" /public/zaciatocnik.txt

# Example 5: Find exact word with line numbers
$ grep -nw "system" /public/zaciatocnik.txt
```

---

### Common Mistakes

#### Mistake 1: Forgetting the Backslash

```bash
# WRONG - < and > are shell redirections!
$ grep <system> /public/zaciatocnik.txt   # ERROR!

# CORRECT - Escape with backslash
$ grep "\<system\>" /public/zaciatocnik.txt
```

#### Mistake 2: Mixing -w with \< \>

```bash
# REDUNDANT - Don't combine them
$ grep -w "\<system\>" /public/zaciatocnik.txt

# Choose one:
$ grep -w "system" /public/zaciatocnik.txt    # Simpler
$ grep "\<system\>" /public/zaciatocnik.txt   # More explicit
```

#### Mistake 3: Not Quoting the Pattern

```bash
# RISKY - Shell might interpret the backslashes
$ grep \<system\> /public/zaciatocnik.txt

# CORRECT - Always quote patterns with special characters
$ grep "\<system\>" /public/zaciatocnik.txt
```

---

### Key Takeaways

1. **Default grep** finds patterns anywhere in the text
2. **`-w` flag** matches whole words only (simplest method)
3. **`\<` and `\>`** are regex anchors for word boundaries
4. **`\<pattern`** matches words starting with pattern
5. **`pattern\>`** matches words ending with pattern
6. **`\<pattern\>`** is equivalent to `-w pattern`
7. **Always quote** patterns with special characters

---

## Regular Expressions - Character Classes

Character classes allow you to match specific sets of characters. Wildcards and character ranges creates flexible search patterns.

---

## Wildcard Matching with .*

The combination of `.` (any character) and `*` (zero or more times) creates a powerful wildcard pattern.

### Understanding the Components

- `.` - Matches **any single character** (except newline)
- `*` - Matches **zero or more** of the preceding element
- `.*` - Together, matches **any sequence of characters** (including empty string)

### Finding Words Starting with a Letter

```bash
# Find everything that STARTS with "s"
$ grep "\<s.*\>" /public/zaciatocnik.txt

# Breaking it down:
# \<      - Word boundary (start of word)
# s       - Must start with letter 's'
# .*      - Followed by any characters (or nothing)
# \>      - Word boundary (end of word)

# This matches:
# - "s" (just the letter s)
# - "sa"
# - "system"
# - "subor"
# - "systematic"
# - Any word starting with 's'
```

**How it works:**
1. `\<` ensures we're at the start of a word
2. `s` matches the letter 's'
3. `.*` matches everything after 's' until...
4. `\>` marks the end of the word

---

## Character Ranges [A-Z], [a-z], [0-9]

Character ranges allow you to match specific sets of characters. Use square brackets `[]` to define a range.

### Uppercase Letters Only: [A-Z]

```bash
# Find words starting with "s" followed by UPPERCASE letters only
$ grep "\<s[A-Z]*\>" /public/zaciatocnik.txt

# Breaking it down:
# \<      - Word boundary (start of word)
# s       - Must start with letter 's'
# [A-Z]   - Any uppercase letter (A, B, C, ..., Z)
# *       - Zero or more times
# \>      - Word boundary (end of word)

# This matches:
# - "s" (no uppercase letters after)
# - "sA", "sZ", "sABC"
# This does NOT match:
# - "system" (has lowercase letters)
# - "s123" (has numbers)
# - "s_test" (has underscore and lowercase)
```

### Lowercase Letters Only: [a-z]

```bash
# Find words starting with "s" followed by LOWERCASE letters only
$ grep "\<s[a-z]*\>" /public/zaciatocnik.txt

# Breaking it down:
# \<      - Word boundary (start of word)
# s       - Must start with letter 's'
# [a-z]   - Any lowercase letter (a, b, c, ..., z)
# *       - Zero or more times
# \>      - Word boundary (end of word)

# This matches:
# - "s" (no lowercase letters after)
# - "sa", "system", "subor", "systematic"
# This does NOT match:
# - "sSystem" (has uppercase letters)
# - "s123" (has numbers)
```

---

## Combining Character Classes

You can combine multiple character ranges within a single bracket expression.

### Any Letter (Uppercase or Lowercase): [A-Za-z]

```bash
# Find words starting with "s" followed by any letters
$ grep "\<s[A-Za-z]*\>" /public/zaciatocnik.txt

# Breaking it down:
# \<        - Word boundary (start of word)
# s         - Must start with letter 's'
# [A-Za-z]  - Any uppercase OR lowercase letter
# *         - Zero or more times
# \>        - Word boundary (end of word)

# This matches:
# - "s", "sa", "System", "subor", "sYsTeM"
# This does NOT match:
# - "s123" (has numbers)
# - "s_test" (has underscore)
```

### Letters and Numbers: [A-Za-z0-9]

```bash
# Find words starting with "s" followed by letters or numbers
$ grep "\<s[A-Za-z0-9]*\>" /public/zaciatocnik.txt

# Breaking it down:
# \<          - Word boundary (start of word)
# s           - Must start with letter 's'
# [A-Za-z0-9] - Any letter (upper/lower) OR digit (0-9)
# *           - Zero or more times
# \>          - Word boundary (end of word)

# This matches:
# - "s", "sa", "s123", "system42", "s1a2b3"
# This does NOT match:
# - "s_test" (has underscore)
# - "s-file" (has hyphen)
```

---

## Character Range Syntax

| Pattern | Matches | Example | Description |
|---------|---------|---------|-------------|
| `[A-Z]` | Uppercase letters | A, B, C, ..., Z | Single uppercase letter |
| `[a-z]` | Lowercase letters | a, b, c, ..., z | Single lowercase letter |
| `[0-9]` | Digits | 0, 1, 2, ..., 9 | Single digit |
| `[A-Za-z]` | Any letter | a, B, z, M | Uppercase OR lowercase |
| `[A-Za-z0-9]` | Alphanumeric | a, Z, 5, 8 | Letters OR digits |
| `[aeiou]` | Specific chars | a, e, i, o, u | Any of these vowels |

---

## The Quantifier: *

The `*` quantifier means "zero or more" of the preceding element.

```bash
# Pattern breakdown for: "\<s[a-z]*\>"

\<s[a-z]*\>
  │  │  │
  │  │  └─ * means: zero or more lowercase letters
  │  └──── [a-z] means: any lowercase letter
  └─────── \<s means: word starting with 's'

# Matches:
# s      (zero lowercase letters after s)
# sa     (one lowercase letter: a)
# sub    (two lowercase letters: u, b)
# system (five lowercase letters: y, s, t, e, m)
```

---

## Step-by-Step Pattern Evolution

Let's see how patterns become more restrictive or permissive:

### 1. Most Permissive: Any Characters

```bash
$ grep "\<s.*\>" /public/zaciatocnik.txt
# Matches: s, sa, s123, s_test, sYsTeM, etc.
# (Any word starting with 's')
```

### 2. Only Uppercase After 's'

```bash
$ grep "\<s[A-Z]*\>" /public/zaciatocnik.txt
# Matches: s, sA, sABC
# Does NOT match: system (has lowercase)
```

### 3. Only Lowercase After 's'

```bash
$ grep "\<s[a-z]*\>" /public/zaciatocnik.txt
# Matches: s, sa, system, subor
# Does NOT match: System (has uppercase)
```

### 4. Any Letters (Upper or Lower)

```bash
$ grep "\<s[A-Za-z]*\>" /public/zaciatocnik.txt
# Matches: s, sa, System, subor, sYsTeM
# Does NOT match: s123 (has numbers)
```

### 5. Letters and Numbers

```bash
$ grep "\<s[A-Za-z0-9]*\>" /public/zaciatocnik.txt
# Matches: s, sa, System, s123, system42
# Does NOT match: s_test (has underscore)
```

---

## Visual Examples

```
Word: "system"

Pattern: \<s.*\>
         ↓↓↓↓↓↓
         system  ✓ MATCH (any characters)

Pattern: \<s[A-Z]*\>
         s ystem  ✗ NO MATCH (has lowercase)

Pattern: \<s[a-z]*\>
         ↓↓↓↓↓↓
         system  ✓ MATCH (all lowercase)

Pattern: \<s[A-Za-z]*\>
         ↓↓↓↓↓↓
         system  ✓ MATCH (all letters)

Pattern: \<s[A-Za-z0-9]*\>
         ↓↓↓↓↓↓
         system  ✓ MATCH (letters/numbers allowed)
```

---

## Practice Examples

```bash
# Example 1: Find words starting with 's', only lowercase letters
$ grep "\<s[a-z]*\>" /public/zaciatocnik.txt

# Example 2: Find words starting with 's', any letters
$ grep "\<s[A-Za-z]*\>" /public/zaciatocnik.txt

# Example 3: Find words starting with 's', letters and digits
$ grep "\<s[A-Za-z0-9]*\>" /public/zaciatocnik.txt

# Example 4: Find words starting with any lowercase vowel
$ grep "\<[aeiou]" /public/zaciatocnik.txt

# Example 5: Find words ending with any digit
$ grep "[0-9]\>" /public/zaciatocnik.txt
```

---

## Common Mistakes

### Mistake 1: Forgetting the Quantifier

```bash
# WRONG - Matches only "sa", "sb", "sc", etc. (exactly 2 characters)
$ grep "\<s[a-z]\>" /public/zaciatocnik.txt

# CORRECT - Matches "s", "sa", "system", etc. (1 or more characters)
$ grep "\<s[a-z]*\>" /public/zaciatocnik.txt
```

### Mistake 2: Using Spaces in Character Class

```bash
# WRONG - Space is treated as a literal space character
$ grep "\<s[A-Z a-z]*\>" /public/zaciatocnik.txt

# CORRECT - No spaces in the character class
$ grep "\<s[A-Za-z]*\>" /public/zaciatocnik.txt
```

### Mistake 3: Wrong Range Order

```bash
# WRONG - This is covered in the next section (ASCII issues)
$ grep "\<s[a-Z]*\>" /public/zaciatocnik.txt

# CORRECT - Always use proper order: A-Z or a-z
$ grep "\<s[A-Za-z]*\>" /public/zaciatocnik.txt
```

---

## Key Takeaways

1. **`.`** matches any single character
2. **`*`** means zero or more of the preceding element
3. **`.*`** matches any sequence of characters
4. **`[A-Z]`** matches uppercase letters
5. **`[a-z]`** matches lowercase letters
6. **`[0-9]`** matches digits
7. **`[A-Za-z]`** matches any letter
8. **`[A-Za-z0-9]`** matches alphanumeric characters
9. Character classes must be inside square brackets `[]`
10. Always use `*` after the character class for "zero or more"

---


## POSIX Character Classes

POSIX character classes provide a portable and readable way to match common character sets. They are especially useful because they work consistently across different locales and systems.

---

### Why Use POSIX Character Classes?

### Problems with Traditional Ranges

```bash
# Traditional approach - can be verbose
$ grep "\<s[A-Za-z0-9]*\>" /public/zaciatocnik.txt

# POSIX approach - cleaner and more portable
$ grep "\<s[[:alnum:]]*\>" /public/zaciatocnik.txt
```

**Advantages of POSIX classes:**
1. **More readable** - `[[:alnum:]]` is clearer than `[A-Za-z0-9]`
2. **Locale-aware** - Handles international characters properly
3. **Less error-prone** - No need to remember multiple ranges
4. **Portable** - Works the same across different Unix systems

---

### Using [[:alnum:]]

The `[[:alnum:]]` class matches **alphanumeric characters** (letters and digits).

### Basic Usage

```bash
# Find words starting with "s" followed by alphanumeric characters
$ grep "\<s[[:alnum:]]*\>" /public/zaciatocnik.txt

# Breaking it down:
# \<          - Word boundary (start of word)
# s           - Must start with letter 's'
# [[:alnum:]] - Any letter (A-Z, a-z) OR digit (0-9)
# *           - Zero or more times
# \>          - Word boundary (end of word)

# This matches:
# - "s", "sa", "system", "s123", "system42"
# This does NOT match:
# - "s_test" (underscore is not alphanumeric)
# - "s-file" (hyphen is not alphanumeric)
```

### Comparison: [A-Za-z0-9] vs [[:alnum:]]

```bash
# These are equivalent:
$ grep "\<s[A-Za-z0-9]*\>" /public/zaciatocnik.txt
$ grep "\<s[[:alnum:]]*\>" /public/zaciatocnik.txt

# But [[:alnum:]] is:
# - Shorter and more readable
# - Works with international characters (like é, ñ, ü)
```

---

### Other POSIX Classes

### Complete List of POSIX Character Classes

| POSIX Class | Equivalent | Matches | Example |
|-------------|------------|---------|---------|
| `[[:alnum:]]` | `[A-Za-z0-9]` | Letters and digits | a, Z, 5, 9 |
| `[[:alpha:]]` | `[A-Za-z]` | Letters only | a, B, z, M |
| `[[:digit:]]` | `[0-9]` | Digits only | 0, 1, 2, 9 |
| `[[:lower:]]` | `[a-z]` | Lowercase letters | a, b, z |
| `[[:upper:]]` | `[A-Z]` | Uppercase letters | A, B, Z |
| `[[:space:]]` | `[ \t\n\r\f\v]` | Whitespace chars | space, tab, newline |
| `[[:punct:]]` | Punctuation | Punctuation marks | ., !, ?, ; |
| `[[:blank:]]` | `[ \t]` | Space and tab only | space, tab |
| `[[:print:]]` | Printable | Printable characters | a, 1, !, (not control chars) |
| `[[:graph:]]` | Visible | Visible characters | a, 1, ! (not space) |
| `[[:xdigit:]]` | `[0-9A-Fa-f]` | Hex digits | 0-9, A-F, a-f |

---

### Practical Examples

### Example 1: Alphabetic Characters Only

```bash
# Using traditional range
$ grep "\<s[A-Za-z]*\>" /public/zaciatocnik.txt

# Using POSIX class (equivalent, but cleaner)
$ grep "\<s[[:alpha:]]*\>" /public/zaciatocnik.txt

# Matches: s, sa, system, subor
# Does NOT match: s123, s_test
```

### Example 2: Digits Only

```bash
# Find words starting with "s" followed by digits only
$ grep "\<s[[:digit:]]*\>" /public/zaciatocnik.txt

# Matches: s, s1, s123, s9999
# Does NOT match: system, s1a2, s_test
```

### Example 3: Lowercase Letters Only

```bash
# Using traditional range
$ grep "\<s[a-z]*\>" /public/zaciatocnik.txt

# Using POSIX class (equivalent)
$ grep "\<s[[:lower:]]*\>" /public/zaciatocnik.txt

# Matches: s, sa, system, subor
# Does NOT match: System, SYSTEM
```

### Example 4: Uppercase Letters Only

```bash
# Using traditional range
$ grep "\<s[A-Z]*\>" /public/zaciatocnik.txt

# Using POSIX class (equivalent)
$ grep "\<s[[:upper:]]*\>" /public/zaciatocnik.txt

# Matches: s, sA, sABC
# Does NOT match: system, sa
```

---

### POSIX Class Syntax

### Important: Double Brackets Required!

```bash
# WRONG - Single brackets don't work
$ grep "\<s[:alnum:]*\>" /public/zaciatocnik.txt   # ERROR!

# CORRECT - Must use double brackets [[ ]]
$ grep "\<s[[:alnum:]]*\>" /public/zaciatocnik.txt
```

**Why double brackets?**
- Outer brackets `[]` define the character class
- Inner brackets `[: :]` define the POSIX class name
- Format: `[[:classname:]]`

---

### Combining POSIX Classes

You can combine multiple POSIX classes within a single character class.

### Letters or Digits

```bash
# Letters OR digits
$ grep "\<s[[:alpha:][:digit:]]*\>" /public/zaciatocnik.txt

# Same as [[:alnum:]]
$ grep "\<s[[:alnum:]]*\>" /public/zaciatocnik.txt
```

### Letters, Digits, or Underscore

```bash
# Alphanumeric OR underscore
$ grep "\<s[[:alnum:]_]*\>" /public/zaciatocnik.txt

# Matches: s, sa, system, s123, s_test
# Does NOT match: s-file (hyphen not included)
```

### Letters or Punctuation

```bash
# Letters OR punctuation marks
$ grep "[[:alpha:][:punct:]]" /public/zaciatocnik.txt

# Matches lines containing letters or punctuation
```

---

### When to Use POSIX vs Traditional Ranges

### Use POSIX Classes When:

✅ You want **readable, self-documenting** patterns
```bash
# Clear what it does
grep "[[:digit:]]" /public/zaciatocnik.txt
```

✅ You need **international character** support
```bash
# Handles é, ñ, ü correctly with locale
grep "[[:alpha:]]" /public/zaciatocnik.txt
```

✅ You want **portable** code across systems
```bash
# Works the same everywhere
grep "[[:alnum:]]" /public/zaciatocnik.txt
```

### Use Traditional Ranges When:

✅ You need **specific characters** only
```bash
# Only ASCII letters, no international chars
grep "[A-Za-z]" /public/zaciatocnik.txt
```

✅ You want **explicit control** over the range
```bash
# Only lowercase h through m
grep "[h-m]" /public/zaciatocnik.txt
```

---

### Visual Comparison

```
Pattern: \<s[A-Za-z0-9]*\>
Pattern: \<s[[:alnum:]]*\>      (EQUIVALENT)
         ↓↓↓↓↓↓↓
         system42  ✓ MATCH

Pattern: \<s[A-Za-z]*\>
Pattern: \<s[[:alpha:]]*\>      (EQUIVALENT)
         ↓↓↓↓↓↓
         system  ✓ MATCH
         system42  ✗ NO MATCH (has digits)

Pattern: \<s[0-9]*\>
Pattern: \<s[[:digit:]]*\>      (EQUIVALENT)
         s123  ✓ MATCH
         system  ✗ NO MATCH (has letters)

Pattern: \<s[a-z]*\>
Pattern: \<s[[:lower:]]*\>      (EQUIVALENT)
         ↓↓↓↓↓↓
         system  ✓ MATCH
         System  ✗ NO MATCH (has uppercase)
```

---

## Practice Examples

```bash
# Example 1: Words starting with "s", alphanumeric only
$ grep "\<s[[:alnum:]]*\>" /public/zaciatocnik.txt

# Example 2: Words starting with "s", letters only
$ grep "\<s[[:alpha:]]*\>" /public/zaciatocnik.txt

# Example 3: Words starting with "s", lowercase only
$ grep "\<s[[:lower:]]*\>" /public/zaciatocnik.txt

# Example 4: Lines containing any digit
$ grep "[[:digit:]]" /public/zaciatocnik.txt

# Example 5: Words starting with any lowercase letter
$ grep "\<[[:lower:]]" /public/zaciatocnik.txt

# Example 6: Lines containing punctuation
$ grep "[[:punct:]]" /public/zaciatocnik.txt
```

---

## Common Mistakes

### Mistake 1: Single Brackets

```bash
# WRONG - Missing outer brackets
$ grep "\<s[:alnum:]*\>" /public/zaciatocnik.txt

# CORRECT - Double brackets required
$ grep "\<s[[:alnum:]]*\>" /public/zaciatocnik.txt
```

### Mistake 2: Typo in Class Name

```bash
# WRONG - "alnum" not "alphanum"
$ grep "\<s[[:alphanum:]]*\>" /public/zaciatocnik.txt

# CORRECT
$ grep "\<s[[:alnum:]]*\>" /public/zaciatocnik.txt
```

### Mistake 3: Forgetting Quantifier

```bash
# WRONG - Matches only 2-character words (s + one alnum)
$ grep "\<s[[:alnum:]]\>" /public/zaciatocnik.txt

# CORRECT - Zero or more alphanumeric chars
$ grep "\<s[[:alnum:]]*\>" /public/zaciatocnik.txt
```

### Mistake 4: Spaces in POSIX Class

```bash
# WRONG - Spaces break the POSIX class
$ grep "\<s[[:alnum:] ]*\>" /public/zaciatocnik.txt

# CORRECT - If you want space, add it outside the POSIX class
$ grep "\<s[[:alnum:] ]*\>" /public/zaciatocnik.txt
# Or use [[:space:]]
$ grep "\<s[[:alnum:][:space:]]*\>" /public/zaciatocnik.txt
```

---

## Key Takeaways

1. **POSIX classes** use double brackets: `[[:classname:]]`
2. **`[[:alnum:]]`** = letters and digits (A-Z, a-z, 0-9)
3. **`[[:alpha:]]`** = letters only (A-Z, a-z)
4. **`[[:digit:]]`** = digits only (0-9)
5. **`[[:lower:]]`** = lowercase letters (a-z)
6. **`[[:upper:]]`** = uppercase letters (A-Z)
7. POSIX classes are **more readable** than traditional ranges
8. POSIX classes are **locale-aware** (handle international chars)
9. POSIX classes are **portable** across Unix systems
10. Can **combine** multiple POSIX classes: `[[:alpha:][:digit:]]`

---

## Character Ranges and ASCII Considerations

Understanding how character ranges work in grep requires knowledge of ASCII values.

---

## How Character Ranges Work

Character ranges in grep are based on **ASCII values** (or the current locale's collating sequence). The range `[a-z]` means "all characters between 'a' and 'z' in ASCII order."

### ASCII Table Reference

Here's a simplified ASCII table showing the relevant characters:

```
Decimal | Character | Type
--------|-----------|------------
  48    |    0      | Digit
  49    |    1      | Digit
  ...   |   ...     | ...
  57    |    9      | Digit
  58    |    :      | Punctuation
  59    |    ;      | Punctuation
  ...   |   ...     | ...
  64    |    @      | Punctuation
  65    |    A      | Uppercase
  66    |    B      | Uppercase
  ...   |   ...     | ...
  90    |    Z      | Uppercase
  91    |    [      | Punctuation
  92    |    \      | Punctuation
  93    |    ]      | Punctuation
  94    |    ^      | Punctuation
  95    |    _      | Punctuation
  96    |    `      | Punctuation
  97    |    a      | Lowercase
  98    |    b      | Lowercase
  ...   |   ...     | ...
 122    |    z      | Lowercase
```

**Key observation:** 
- Uppercase letters (A-Z): ASCII 65-90
- Lowercase letters (a-z): ASCII 97-122
- **There are 6 punctuation characters between Z and a:** `[ \ ] ^ _ \``

---

## Invalid Ranges: [a-Z]

### Why [a-Z] is Invalid

```bash
# INVALID RANGE - This will cause an error or unexpected behavior
$ grep "\<s[a-Z]*\>" /public/zaciatocnik.txt

# Error message (on most systems):
# grep: Invalid range end
```

**Why it's invalid:**
- `a` has ASCII value 97
- `Z` has ASCII value 90
- The range goes from 97 to 90, which is **backwards**!
- You can't have a range where the start value is greater than the end value

### ASCII Comparison

```
[a-Z] means: ASCII 97 to ASCII 90
             ↓              ↓
             a              Z
             
This is INVALID because 97 > 90
```

---

## Valid but Unexpected: [A-z]

### Why [A-z] is Confusing

```bash
# VALID but includes unexpected characters!
$ grep "\<s[A-z]*\>" /public/zaciatocnik.txt

# This works, but matches MORE than just letters!
```

**What [A-z] actually matches:**
- All uppercase letters: A-Z (ASCII 65-90)
- Six punctuation characters: `[ \ ] ^ _ \`` (ASCII 91-96)
- All lowercase letters: a-z (ASCII 97-122)

### ASCII Breakdown

```
[A-z] means: ASCII 65 to ASCII 122

Includes:
A B C ... Z    (uppercase letters - ASCII 65-90)
[ \ ] ^ _ `    (punctuation - ASCII 91-96)  ← UNEXPECTED!
a b c ... z    (lowercase letters - ASCII 97-122)
```

### Demonstration

```bash
# This matches words with underscores!
$ grep "\<s[A-z]*\>" /public/zaciatocnik.txt

# If the file contains: "s_test", "s[bracket", "s^caret"
# They will ALL match because [, ], ^, _, \, ` are in the range A-z
```

### Visual Representation

```
What you might THINK [A-z] matches:
ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz

What [A-z] ACTUALLY matches:
ABCDEFGHIJKLMNOPQRSTUVWXYZ[\]^_`abcdefghijklmnopqrstuvwxyz
                           ↑↑↑↑↑↑
                     These 6 punctuation chars!
```

---

## The ASCII Table Explanation

### Complete Character Order

Understanding the full ASCII sequence helps avoid confusion:

```
Position | ASCII | Character | In Range?
---------|-------|-----------|----------
  ...    |  ...  |   ...     |
   64    |  @    | @         | NO
   65    |  A    | A         | YES [A-z]
   66    |  B    | B         | YES [A-z]
  ...    | ...   | ...       | YES [A-z]
   90    |  Z    | Z         | YES [A-z]
   91    |  [    | [         | YES [A-z] ← Surprise!
   92    |  \    | \         | YES [A-z] ← Surprise!
   93    |  ]    | ]         | YES [A-z] ← Surprise!
   94    |  ^    | ^         | YES [A-z] ← Surprise!
   95    |  _    | _         | YES [A-z] ← Surprise!
   96    |  `    | `         | YES [A-z] ← Surprise!
   97    |  a    | a         | YES [A-z]
   98    |  b    | b         | YES [A-z]
  ...    | ...   | ...       | YES [A-z]
  122    |  z    | z         | YES [A-z]
  123    |  {    | {         | NO
```

### Why This Matters

When you write `[A-z]`, grep interprets it as:
1. Start at ASCII 65 (A)
2. End at ASCII 122 (z)
3. Include **everything** between them

This includes the 6 punctuation marks that sit between uppercase and lowercase letters in the ASCII table.

---

## Correct Ways to Match All Letters

### Method 1: Use Two Ranges (Recommended)

```bash
# Correct: Explicitly specify both ranges
$ grep "\<s[A-Za-z]*\>" /public/zaciatocnik.txt

# Matches: sA, sZ, sa, sz, sSystem, system
# Does NOT match: s[test, s_file, s^caret
```

### Method 2: Use POSIX Classes (Best Practice)

```bash
# Best: Use POSIX class for letters
$ grep "\<s[[:alpha:]]*\>" /public/zaciatocnik.txt

# Cleaner, more readable, and locale-aware
```

### Method 3: Case-Insensitive Flag

```bash
# Alternative: Use -i flag with one range
$ grep -i "\<s[a-z]*\>" /public/zaciatocnik.txt

# The -i flag makes the search case-insensitive
```

---

## Common Range Patterns

### Valid Ranges

| Range | ASCII Values | Matches | Example |
|-------|--------------|---------|---------|
| `[A-Z]` | 65-90 | Uppercase only | A, B, Z |
| `[a-z]` | 97-122 | Lowercase only | a, b, z |
| `[0-9]` | 48-57 | Digits only | 0, 5, 9 |
| `[A-Za-z]` | 65-90, 97-122 | Letters only | A, z, M |
| `[a-zA-Z]` | Same as above | Letters only | a, Z, m |
| `[0-9A-Za-z]` | 48-57, 65-90, 97-122 | Alphanumeric | 5, A, z |

### Invalid Ranges

| Range | Why Invalid | Fix |
|-------|-------------|-----|
| `[a-Z]` | Start > End (97 > 90) | Use `[A-Za-z]` |
| `[z-a]` | Start > End (122 > 97) | Use `[a-z]` |
| `[9-0]` | Start > End (57 > 48) | Use `[0-9]` |
| `[Z-A]` | Start > End (90 > 65) | Use `[A-Z]` |

### Unexpected Ranges (Valid but Surprising)

| Range | ASCII Values | Unexpected Characters | Fix |
|-------|--------------|----------------------|-----|
| `[A-z]` | 65-122 | `[ \ ] ^ _ \`` | Use `[A-Za-z]` |
| `[0-z]` | 48-122 | Many punctuation marks | Use `[0-9A-Za-z]` |
| `[@-Z]` | 64-90 | `@` character | Use `[A-Z]` |

---

## Testing Character Ranges

### How to Test What a Range Matches

```bash
# Create a test file with various characters
$ echo "ABC XYZ abc xyz 123 ___ [[[" > test.txt

# Test [A-z] range
$ grep "[A-z]" test.txt
# Output shows which characters match

# Test [A-Za-z] range
$ grep "[A-Za-z]" test.txt
# Compare the output
```

---

## Practice Examples

```bash
# Example 1: Correct - Only letters
$ grep "\<s[A-Za-z]*\>" /public/zaciatocnik.txt

# Example 2: Incorrect - Backwards range (will fail)
$ grep "\<s[a-Z]*\>" /public/zaciatocnik.txt

# Example 3: Unexpected - Includes punctuation
$ grep "\<s[A-z]*\>" /public/zaciatocnik.txt

# Example 4: Best practice - Use POSIX class
$ grep "\<s[[:alpha:]]*\>" /public/zaciatocnik.txt

# Example 5: Valid ranges for specific cases
$ grep "\<s[a-z]*\>" /public/zaciatocnik.txt    # Only lowercase
$ grep "\<s[A-Z]*\>" /public/zaciatocnik.txt    # Only uppercase
$ grep "\<s[0-9]*\>" /public/zaciatocnik.txt    # Only digits
```

---

## Visual Comparison

```
Text: "system", "s_test", "s[bracket"

Pattern: [a-Z]
Result: ERROR (invalid range)

Pattern: [A-z]
         ↓↓↓↓↓↓  ↓↓↓↓↓↓   ↓↓↓↓↓↓↓↓↓↓
         system  s_test   s[bracket
         ✓       ✓        ✓
         (All match due to _, [)

Pattern: [A-Za-z]
         ↓↓↓↓↓↓  s_test   s[bracket
         system  ✗        ✗
         ✓       (has _)  (has [)
```

---

## Common Mistakes

### Mistake 1: Using [a-Z] Instead of [A-Za-z]

```bash
# WRONG - Invalid range
$ grep "\<s[a-Z]*\>" /public/zaciatocnik.txt

# CORRECT
$ grep "\<s[A-Za-z]*\>" /public/zaciatocnik.txt
```

### Mistake 2: Using [A-z] and Expecting Only Letters

```bash
# WRONG - Includes punctuation
$ grep "\<s[A-z]*\>" /public/zaciatocnik.txt

# CORRECT - Only letters
$ grep "\<s[A-Za-z]*\>" /public/zaciatocnik.txt
```

### Mistake 3: Forgetting ASCII Order

```bash
# WRONG - Thinking ASCII is alphabetical
# Student thinks: a comes before Z alphabetically, so [a-Z] should work

# CORRECT - ASCII is numeric
# ASCII values: A=65, Z=90, a=97, z=122
# So use: [A-Za-z]
```

---

## Quantifiers

Quantifiers control how many times a pattern should match. They specify the number of occurrences of the preceding element.

---

## Basic Quantifiers Overview

| Quantifier | Meaning | Example | Matches |
|------------|---------|---------|---------|
| `*` | Zero or more | `a*` | "", "a", "aa", "aaa" |
| `+` | One or more (ERE) | `a+` | "a", "aa", "aaa" |
| `?` | Zero or one (ERE) | `a?` | "", "a" |
| `{n}` | Exactly n times | `a\{3\}` | "aaa" |
| `{n,}` | At least n times | `a\{2,\}` | "aa", "aaa", "aaaa" |
| `{n,m}` | Between n and m times | `a\{2,4\}` | "aa", "aaa", "aaaa" |

**Note:** In Basic Regular Expressions (BRE), braces must be escaped: `\{` and `\}`. In Extended Regular Expressions (ERE with `-E`), use unescaped braces.

---

## Exact Count: {n}

The `{n}` quantifier matches **exactly n occurrences** of the preceding element.

### Basic Syntax (BRE - Requires Escaping)

```bash
# Find words that consist of exactly "sa" (the letters s followed by a)
$ grep "\<sa\{1\}\>" /public/zaciatocnik.txt

# Breaking it down:
# \<      - Word boundary (start of word)
# s       - Letter 's'
# a       - Letter 'a'
# \{1\}   - Exactly 1 time
# \>      - Word boundary (end of word)

# This matches:
# - "sa" (exactly one 'a' after 's')
# This does NOT match:
# - "saa" (has two 'a's)
# - "s" (no 'a')
# - "system" (doesn't end after the 'a')
```

### Why \{1\} Seems Redundant

```bash
# These are equivalent:
$ grep "\<sa\{1\}\>" /public/zaciatocnik.txt
$ grep "\<sa\>" /public/zaciatocnik.txt

# Both match exactly "sa" as a word
# The \{1\} explicitly says "one time" but 'a' already means "one 'a'"
```

### Practical Use: Multiple Characters

```bash
# Find words with exactly 2 'a's after 's'
$ grep "\<sa\{2\}\>" /public/zaciatocnik.txt

# Matches: "saa"
# Does NOT match: "sa", "saaa"

# Find words starting with 's' followed by exactly 3 lowercase letters
$ grep "\<s[a-z]\{3\}\>" /public/zaciatocnik.txt

# Matches: "s" + exactly 3 lowercase letters
# Examples: "syst" (if it exists as a word), "subr"
# Does NOT match: "s", "sa", "system" (has more than 3)
```

---

## Range Count: {n,m}

The `{n,m}` quantifier matches **between n and m occurrences** (inclusive).

### Basic Syntax

```bash
# Find words that consist of 1-2 letters
$ grep "\<[A-Za-z]\{1,2\}\>" /public/zaciatocnik.txt

# Breaking it down:
# \<          - Word boundary (start of word)
# [A-Za-z]    - Any letter (uppercase or lowercase)
# \{1,2\}     - Between 1 and 2 times (inclusive)
# \>          - Word boundary (end of word)

# This matches:
# - "a" (1 letter)
# - "is" (2 letters)
# - "or" (2 letters)
# This does NOT match:
# - "and" (3 letters)
# - "system" (6 letters)
```

### More Examples

```bash
# Find words starting with 's' followed by 2-4 lowercase letters
$ grep "\<s[a-z]\{2,4\}\>" /public/zaciatocnik.txt

# Matches:
# - "s" + 2 letters: "sub"
# - "s" + 3 letters: "sys", "sam"
# - "s" + 4 letters: "subor" (if only 4 letters after s)
# Does NOT match:
# - "s" + 1 letter: "sa"
# - "s" + 5+ letters: "system" (too many)
```

### Minimum Only: {n,}

```bash
# Find words starting with 's' followed by at least 2 lowercase letters
$ grep "\<s[a-z]\{2,\}\>" /public/zaciatocnik.txt

# Matches:
# - "s" + 2+ letters: "sub", "system", "subor", etc.
# Does NOT match:
# - "s" + 0-1 letters: "s", "sa"
```

---

## Other Quantifiers: *, +, ?

### The * Quantifier (Zero or More)

We've been using this throughout the previous sections.

```bash
# Find words starting with 's' followed by zero or more lowercase letters
$ grep "\<s[a-z]*\>" /public/zaciatocnik.txt

# Matches:
# - "s" (zero lowercase letters after)
# - "sa" (one lowercase letter)
# - "system" (multiple lowercase letters)
```

**Key point:** `*` means "zero or more", so the pattern can match even without the repeated element.

### The + Quantifier (One or More) - ERE Only

The `+` quantifier requires **at least one** occurrence. It's only available with Extended Regular Expressions.

```bash
# BASIC REGEX (BRE) - Doesn't support +
$ grep "\<s[a-z]+\>" /public/zaciatocnik.txt
# This WON'T work correctly in BRE!

# EXTENDED REGEX (ERE) - Use -E flag
$ grep -E "\<s[a-z]+\>" /public/zaciatocnik.txt

# Matches:
# - "sa" (one or more lowercase letters)
# - "system" (one or more lowercase letters)
# Does NOT match:
# - "s" (zero lowercase letters)
```

### The ? Quantifier (Zero or One) - ERE Only

The `?` quantifier means "optional" - zero or one occurrence.

```bash
# EXTENDED REGEX (ERE) - Use -E flag
$ grep -E "\<s[a-z]?\>" /public/zaciatocnik.txt

# Matches:
# - "s" (zero lowercase letters)
# - "sa" (one lowercase letter)
# Does NOT match:
# - "system" (more than one lowercase letter after s)
```

---

## Comparing Quantifiers

### Visual Comparison

```
Pattern: s[a-z]*     (zero or more)
Matches: s, sa, system, subor

Pattern: s[a-z]+     (one or more, ERE)
Matches: sa, system, subor
Does NOT match: s

Pattern: s[a-z]?     (zero or one, ERE)
Matches: s, sa
Does NOT match: system, subor

Pattern: s[a-z]\{1\}   (exactly 1)
Matches: sa
Does NOT match: s, system

Pattern: s[a-z]\{2\}   (exactly 2)
Matches: s + exactly 2 letters
Does NOT match: s, sa, system (too many)

Pattern: s[a-z]\{2,4\} (between 2 and 4)
Matches: s + 2, 3, or 4 letters
Does NOT match: s, sa, system (if more than 4)

Pattern: s[a-z]\{2,\}  (at least 2)
Matches: s + 2 or more letters
Does NOT match: s, sa
```

---

## Escaping in BRE vs ERE

### Basic Regular Expressions (BRE) - Default

```bash
# In BRE, braces MUST be escaped
$ grep "\<s[a-z]\{2,4\}\>" /public/zaciatocnik.txt

# Braces with backslashes: \{ \}
```

### Extended Regular Expressions (ERE) - Use -E

```bash
# In ERE, braces should NOT be escaped
$ grep -E "\<s[a-z]{2,4}\>" /public/zaciatocnik.txt

# Braces without backslashes: { }
# Also available: + and ? quantifiers
```

**Common mistake:** Mixing BRE and ERE syntax!

---

## Practical Examples

### Example 1: Find Two-Letter Words

```bash
# Words that are exactly 2 letters
$ grep "\<[A-Za-z]\{2\}\>" /public/zaciatocnik.txt

# Matches: "is", "or", "to", "at"
```

### Example 2: Find Three-Letter Words

```bash
# Words that are exactly 3 letters
$ grep "\<[A-Za-z]\{3\}\>" /public/zaciatocnik.txt

# Matches: "and", "the", "for"
```

### Example 3: Find Short Words (1-3 letters)

```bash
# Words with 1 to 3 letters
$ grep "\<[A-Za-z]\{1,3\}\>" /public/zaciatocnik.txt

# Matches: "a", "is", "and", "the"
```

### Example 4: Find Words with Repeated Characters

```bash
# Find words with at least 2 consecutive 'o's
$ grep "o\{2,\}" /public/zaciatocnik.txt

# Matches words containing: "oo", "ooo", etc.
# Examples: "book", "food", "cool"
```

### Example 5: Using ERE with +

```bash
# Find words starting with 's' followed by at least one letter
$ grep -E "\<s[a-z]+\>" /public/zaciatocnik.txt

# Same as: grep "\<s[a-z]\{1,\}\>" /public/zaciatocnik.txt
```

---

## Quantifiers with Character Classes

You can apply quantifiers to entire character classes.

```bash
# Exactly 3 digits
$ grep "[0-9]\{3\}" /public/zaciatocnik.txt

# 2-4 alphanumeric characters
$ grep "[A-Za-z0-9]\{2,4\}" /public/zaciatocnik.txt

# At least 5 lowercase letters
$ grep "[a-z]\{5,\}" /public/zaciatocnik.txt
```

---

## Common Mistakes

### Mistake 1: Forgetting to Escape Braces in BRE

```bash
# WRONG - Unescaped braces in basic regex
$ grep "\<s[a-z]{2,4}\>" /public/zaciatocnik.txt

# CORRECT - Escape the braces
$ grep "\<s[a-z]\{2,4\}\>" /public/zaciatocnik.txt

# OR use -E for extended regex
$ grep -E "\<s[a-z]{2,4}\>" /public/zaciatocnik.txt
```

### Mistake 2: Using + or ? Without -E

```bash
# WRONG - + doesn't work in basic regex
$ grep "\<s[a-z]+\>" /public/zaciatocnik.txt

# CORRECT - Use -E flag
$ grep -E "\<s[a-z]+\>" /public/zaciatocnik.txt
```

### Mistake 3: Confusing {n} with {n,}

```bash
# {2} means EXACTLY 2
$ grep "\<s[a-z]\{2\}\>" /public/zaciatocnik.txt
# Matches only: 2 letters after s

# {2,} means AT LEAST 2
$ grep "\<s[a-z]\{2,\}\>" /public/zaciatocnik.txt
# Matches: 2 or more letters after s
```

### Mistake 4: Wrong Range Order in {n,m}

```bash
# WRONG - End is less than start
$ grep "\<[a-z]\{5,2\}\>" /public/zaciatocnik.txt

# CORRECT - Start must be ≤ End
$ grep "\<[a-z]\{2,5\}\>" /public/zaciatocnik.txt
```

---

## Summary Table

| Quantifier | BRE Syntax | ERE Syntax | Matches |
|------------|------------|------------|---------|
| Zero or more | `*` | `*` | 0, 1, 2, 3, ... |
| One or more | Not available | `+` | 1, 2, 3, ... |
| Zero or one | Not available | `?` | 0, 1 |
| Exactly n | `\{n\}` | `{n}` | n |
| At least n | `\{n,\}` | `{n,}` | n, n+1, n+2, ... |
| Between n and m | `\{n,m\}` | `{n,m}` | n, n+1, ..., m |

---

## Key Takeaways

1. **`*`** means zero or more (available in BRE and ERE)
2. **`+`** means one or more (ERE only, requires `-E`)
3. **`?`** means zero or one (ERE only, requires `-E`)
4. **`\{n\}`** means exactly n times (BRE - escape braces)
5. **`{n}`** means exactly n times (ERE - no escape)
6. **`\{n,m\}`** means between n and m times (BRE)
7. **`{n,m}`** means between n and m times (ERE)
8. **`\{n,\}`** means at least n times (BRE)
9. In BRE (default), **braces must be escaped**: `\{` and `\}`
10. In ERE (with `-E`), **braces are not escaped**: `{` and `}`

---

## Extended Regular Expressions (ERE)

Extended Regular Expressions (ERE) provide additional features and simplified syntax compared to Basic Regular Expressions (BRE). The `-E` flag enables ERE mode in grep.

---

## Using -E Flag

The `-E` flag (or `--extended-regexp`) tells grep to interpret patterns as Extended Regular Expressions.

### Basic Syntax

```bash
# Basic format
grep -E "pattern" file

# Or use the longer form
grep --extended-regexp "pattern" file
```

### Why Use -E?

**Benefits of ERE:**
1. **No need to escape special characters** like `{`, `}`, `(`, `)`, `|`, `+`, `?`
2. **Alternation operator `|`** works without escaping
3. **Cleaner, more readable** patterns
4. **Additional quantifiers** `+` and `?` are available

---

## Alternation with |

The pipe `|` operator means "OR" - it matches either the pattern on the left OR the pattern on the right.

### Basic Alternation

```bash
# Find lines containing "system" OR "subor"
$ grep -E "\<(system|subor)\>" /public/zaciatocnik.txt

# Breaking it down:
# \<          - Word boundary (start of word)
# (...)       - Grouping (no escape needed in ERE)
# system      - First pattern
# |           - OR operator
# subor       - Second pattern
# \>          - Word boundary (end of word)

# This matches:
# - Lines containing the exact word "system"
# - Lines containing the exact word "subor"
```

### Why Use Parentheses?

Parentheses group the alternatives so the word boundaries apply to both options.

```bash
# WITH parentheses (CORRECT)
$ grep -E "\<(system|subor)\>" /public/zaciatocnik.txt
# Matches: "system" as a word OR "subor" as a word

# WITHOUT parentheses (DIFFERENT MEANING)
$ grep -E "\<system|subor\>" /public/zaciatocnik.txt
# Matches: "\<system" (word starting with "system")
#       OR "subor\>" (word ending with "subor")
# This is probably NOT what you want!
```

### Multiple Alternatives

```bash
# Find "system" OR "subor" OR "sa"
$ grep -E "\<(system|subor|sa)\>" /public/zaciatocnik.txt

# You can have as many alternatives as needed
# Separated by | within the parentheses
```

---

## Differences Between BRE and ERE

Understanding when to use escapes is crucial when switching between BRE and ERE.

### Special Characters: Escape Requirements

| Character | BRE (default) | ERE (with -E) | Purpose |
|-----------|---------------|---------------|---------|
| `(` `)` | `\(` `\)` (escaped) | `(` `)` (unescaped) | Grouping |
| `{` `}` | `\{` `\}` (escaped) | `{` `}` (unescaped) | Quantifiers |
| `\|` | `\|` (escaped) | `|` (unescaped) | Alternation (OR) |
| `+` | Not available | `+` (unescaped) | One or more |
| `?` | Not available | `?` (unescaped) | Zero or one |

### Same Pattern in BRE vs ERE

#### Example 1: Alternation

```bash
# BRE - Need to escape |, (, )
$ grep "\<\(system\|subor\)\>" /public/zaciatocnik.txt

# ERE - No escapes needed
$ grep -E "\<(system|subor)\>" /public/zaciatocnik.txt
```

#### Example 2: Quantifiers with Grouping

```bash
# BRE - Escape braces and parentheses
$ grep "\<s\(a\|y\)\{2,3\}\>" /public/zaciatocnik.txt

# ERE - Clean syntax
$ grep -E "\<s(a|y){2,3}\>" /public/zaciatocnik.txt
```

---

## Practical Examples with -E

### Example 1: Simple Alternation

```bash
# Find exact words "system" or "subor"
$ grep -E "\<(system|subor)\>" /public/zaciatocnik.txt

# Much cleaner than BRE:
$ grep "\<\(system\|subor\)\>" /public/zaciatocnik.txt
```

### Example 2: Multiple Word Search

```bash
# Find lines with "sa" or "system" or "subor"
$ grep -E "\<(sa|system|subor)\>" /public/zaciatocnik.txt

# Matches any line containing any of these words
```

### Example 3: Pattern Variations

```bash
# Find "system" or "systematic"
$ grep -E "\<system(atic)?\>" /public/zaciatocnik.txt

# Breaking it down:
# \<system    - Word must start with "system"
# (atic)?     - Optionally followed by "atic"
# \>          - Word boundary (end)

# Matches: "system" OR "systematic"
```

### Example 4: Using + Quantifier

```bash
# Find words starting with 's' followed by one or more lowercase letters
$ grep -E "\<s[a-z]+\>" /public/zaciatocnik.txt

# The + means "one or more"
# Matches: "sa", "system", "subor"
# Does NOT match: "s" (needs at least one letter after)
```

### Example 5: Complex Pattern

```bash
# Find words starting with 's' or 'a', followed by 2-4 letters
$ grep -E "\<(s|a)[a-z]{2,4}\>" /public/zaciatocnik.txt

# Matches:
# - Words starting with 's': "sys", "subor" (if 2-4 letters after s)
# - Words starting with 'a': "and", "also" (if 2-4 letters after a)
```

---

## Remove Special Characters Effect (ERE Enabled)

One of the main benefits of ERE is that you don't need to escape metacharacters.

### In BRE (Lots of Backslashes)

```bash
# BRE requires escaping many special characters
$ grep "\<\(system\|subor\)\>" /public/zaciatocnik.txt
       ↑↑      ↑↑      ↑↑
       Escaped characters for grouping and alternation
```

### In ERE (Clean Syntax)

```bash
# ERE removes the need for most escapes
$ grep -E "\<(system|subor)\>" /public/zaciatocnik.txt
          No escapes needed!
```

**Visual Comparison:**

```
BRE:  \<\(system\|subor\)\>
      ↓  Remove escapes
ERE:  \<(system|subor)\>
```

---

## When to Use BRE vs ERE

### Use BRE (default grep) When:

✅ **Simple patterns** without special operators
```bash
grep "system" /public/zaciatocnik.txt
grep "\<system\>" /public/zaciatocnik.txt
```

✅ **Portable scripts** (BRE is POSIX standard)

✅ **You're used to the syntax** and don't need advanced features

### Use ERE (grep -E) When:

✅ **Using alternation** (`|`) frequently
```bash
grep -E "(system|subor)" /public/zaciatocnik.txt
```

✅ **Complex patterns** with grouping and alternation
```bash
grep -E "\<s(a|y|u){2,4}\>" /public/zaciatocnik.txt
```

✅ **Need `+` or `?` quantifiers**
```bash
grep -E "\<s[a-z]+\>" /public/zaciatocnik.txt
```

✅ **Want cleaner, more readable** patterns

---

## Comparison Table: BRE vs ERE

| Feature | BRE Example | ERE Example | Notes |
|---------|-------------|-------------|-------|
| Alternation | `\(a\|b\)` | `(a\|b)` or `(a|b)` | ERE is cleaner |
| Grouping | `\(abc\)` | `(abc)` | ERE doesn't escape parens |
| Quantifiers | `a\{2,4\}` | `a{2,4}` | ERE doesn't escape braces |
| One or more | N/A | `a+` | Only in ERE |
| Zero or one | N/A | `a?` | Only in ERE |
| Word boundary | `\<word\>` | `\<word\>` | Same in both |
| Character class | `[a-z]` | `[a-z]` | Same in both |

---

## Multiple Pattern Matching: Alternative to -E

Sometimes you want to match multiple separate patterns without alternation.

### Using Multiple -e Flags

```bash
# Find lines matching "subor" OR "system" (as separate searches)
$ grep -E -e "\<subor\>" -e "\<system\>" /public/zaciatocnik.txt

# This is equivalent to:
$ grep -E "\<(subor|system)\>" /public/zaciatocnik.txt

# But -e can be useful when patterns are complex or numerous
```

---

## Practice Examples

```bash
# Example 1: Find "system" or "subor" (ERE)
$ grep -E "\<(system|subor)\>" /public/zaciatocnik.txt

# Example 2: Same in BRE (for comparison)
$ grep "\<\(system\|subor\)\>" /public/zaciatocnik.txt

# Example 3: Find words starting with 's' or 'a'
$ grep -E "\<(s|a)[a-z]*\>" /public/zaciatocnik.txt

# Example 4: Find "sa" repeated 2-3 times
$ grep -E "(sa){2,3}" /public/zaciatocnik.txt

# Example 5: Optional suffix
$ grep -E "\<system(atic)?\>" /public/zaciatocnik.txt
```

---

## Common Mistakes

### Mistake 1: Using Escapes in ERE

```bash
# WRONG - Too many escapes in ERE mode
$ grep -E "\<\(system\|subor\)\>" /public/zaciatocnik.txt

# CORRECT - No escapes needed
$ grep -E "\<(system|subor)\>" /public/zaciatocnik.txt
```

### Mistake 2: Not Using Escapes in BRE

```bash
# WRONG - Missing escapes in BRE mode
$ grep "\<(system|subor)\>" /public/zaciatocnik.txt

# CORRECT - Escape special characters
$ grep "\<\(system\|subor\)\>" /public/zaciatocnik.txt
```

### Mistake 3: Forgetting Parentheses with Alternation

```bash
# WRONG - Word boundaries don't apply correctly
$ grep -E "\<system|subor\>" /public/zaciatocnik.txt

# CORRECT - Group the alternatives
$ grep -E "\<(system|subor)\>" /public/zaciatocnik.txt
```

### Mistake 4: Using + or ? in BRE

```bash
# WRONG - + doesn't work in BRE
$ grep "\<s[a-z]+\>" /public/zaciatocnik.txt

# CORRECT - Use -E flag
$ grep -E "\<s[a-z]+\>" /public/zaciatocnik.txt

# OR use equivalent BRE syntax
$ grep "\<s[a-z]\{1,\}\>" /public/zaciatocnik.txt
```

---

## Key Takeaways

1. **`-E` flag enables Extended Regular Expressions** (ERE)
2. **ERE has cleaner syntax** - fewer backslashes needed
3. **Alternation `|`** works without escaping in ERE
4. **Parentheses `( )`** don't need escaping in ERE
5. **Braces `{ }`** don't need escaping in ERE
6. **`+` quantifier** (one or more) only works in ERE
7. **`?` quantifier** (zero or one) only works in ERE
8. **Always group alternatives** with parentheses for correct word boundaries
9. **BRE is default** - requires escaping `\(`, `\|`, `\)`, `\{`, `\}`
10. **Use `-E` for complex patterns** with alternation and grouping

---

## Quick Reference: ERE Operators

```bash
# Alternation
grep -E "(pattern1|pattern2)" file

# Grouping
grep -E "(abc)+" file

# One or more
grep -E "a+" file

# Zero or one
grep -E "a?" file

# Quantifiers (no escapes)
grep -E "a{2,4}" file

# Combined example
grep -E "\<(s|a)[a-z]{2,4}\>" file
```

---

## Multiple Patterns

Sometimes you need to search for multiple different patterns in the same grep command. The `-e` flag allows you to specify multiple patterns, and grep will match lines containing ANY of them.

---

## Using -e Flag

The `-e` option allows you to specify multiple search patterns. Each pattern is treated as a separate search criterion.

### Basic Syntax

```bash
# Search for multiple patterns
grep -e "pattern1" -e "pattern2" -e "pattern3" file

# Can be combined with other options
grep -E -e "pattern1" -e "pattern2" file
```

### How -e Works

When you use multiple `-e` options:
- grep searches for **ANY** of the patterns (logical OR)
- A line matches if it contains **at least one** of the patterns
- The patterns are evaluated independently

---

## Simple Multiple Pattern Example

### Without Word Boundaries

```bash
# Find lines containing "system" OR "subor"
$ grep -e "system" -e "subor" /public/zaciatocnik.txt

# This matches:
# - Lines containing "system" anywhere
# - Lines containing "subor" anywhere
# - Lines containing both
```

### With Word Boundaries

```bash
# Find exact words "system" OR "subor"
$ grep -e "\<system\>" -e "\<subor\>" /public/zaciatocnik.txt

# This matches:
# - Lines with the exact word "system"
# - Lines with the exact word "subor"
# - Lines with both exact words
```

---

## Combining -E and -e Flags

You can use `-E` (Extended Regular Expressions) together with `-e` (multiple patterns) for powerful searches.

### Example from Your Notes

```bash
# Search for exact words "subor" OR "system" using ERE
$ grep -E -e "\<subor\>" -e "\<system\>" /public/zaciatocnik.txt

# Breaking it down:
# -E             - Use Extended Regular Expressions
# -e "\<subor\>" - First pattern: exact word "subor"
# -e "\<system\>" - Second pattern: exact word "system"
```

**This is equivalent to:**

```bash
# Using alternation instead of multiple -e flags
$ grep -E "\<(subor|system)\>" /public/zaciatocnik.txt
```

---

## When to Use -e vs Alternation |

Both methods achieve similar results, but each has its use cases.

### Using -e Flag

**Advantages:**
- Clear separation of patterns
- Easier to read with many patterns
- Can mix simple and complex patterns
- Good for dynamically generated patterns in scripts

```bash
# Clear and readable with multiple patterns
$ grep -E -e "\<subor\>" -e "\<system\>" -e "\<sa\>" /public/zaciatocnik.txt
```

### Using Alternation |

**Advantages:**
- More compact for 2-3 patterns
- Single pattern string
- Better for related alternatives

```bash
# Compact for a few alternatives
$ grep -E "\<(subor|system|sa)\>" /public/zaciatocnik.txt
```

### Comparison

```bash
# Method 1: Multiple -e flags
$ grep -E -e "\<subor\>" -e "\<system\>" /public/zaciatocnik.txt

# Method 2: Alternation with |
$ grep -E "\<(subor|system)\>" /public/zaciatocnik.txt

# Both produce the same results!
```

---

## Combining Multiple Patterns

### Multiple Patterns with Different Options

```bash
# Find lines with "system" (case-sensitive) OR "ERROR" (exact word)
$ grep -e "system" -e "\<ERROR\>" /public/zaciatocnik.txt

# First pattern: matches "system" anywhere (part of word OK)
# Second pattern: matches "ERROR" as exact word only
```

### Multiple Complex Patterns

```bash
# Find words starting with 's' OR ending with 'or'
$ grep -E -e "\<s[a-z]*" -e "[a-z]*or\>" /public/zaciatocnik.txt

# First pattern: words starting with 's'
# Second pattern: words ending with 'or'
```

---

## Practical Examples

### Example 1: Search for Multiple Exact Words

```bash
# Find lines containing "system", "subor", or "sa" as exact words
$ grep -E -e "\<system\>" -e "\<subor\>" -e "\<sa\>" /public/zaciatocnik.txt

# Matches lines with any of these exact words
```

### Example 2: Different Pattern Types

```bash
# Find lines with:
# - Exact word "system"
# - OR anything starting with "sub"
$ grep -E -e "\<system\>" -e "\<sub[a-z]*" /public/zaciatocnik.txt
```

### Example 3: Case-Insensitive with Multiple Patterns

```bash
# Find "system" or "subor" (case-insensitive)
$ grep -iE -e "\<system\>" -e "\<subor\>" /public/zaciatocnik.txt

# -i makes the search case-insensitive
# Matches: system, System, SYSTEM, subor, Subor, SUBOR
```

### Example 4: Multiple Patterns with Line Numbers

```bash
# Show line numbers for matches
$ grep -nE -e "\<system\>" -e "\<subor\>" /public/zaciatocnik.txt

# Output includes line numbers
```

### Example 5: Count Lines Matching Any Pattern

```bash
# Count lines containing any of the patterns
$ grep -cE -e "\<system\>" -e "\<subor\>" /public/zaciatocnik.txt

# Returns the count of matching lines
```

---

## Patterns from a File: -f Flag

Instead of using multiple `-e` flags, you can store patterns in a file and use `-f`.

### Using -f Flag

```bash
# Create a file with patterns (one per line)
$ cat patterns.txt
system
subor
sa

# Use -f to read patterns from file
$ grep -f patterns.txt /public/zaciatocnik.txt

# This is equivalent to:
$ grep -e "system" -e "subor" -e "sa" /public/zaciatocnik.txt
```

**Note:** Patterns in the file are treated as basic regular expressions unless you use `-E`.

```bash
# With word boundaries in pattern file
$ cat patterns.txt
\<system\>
\<subor\>
\<sa\>

# Use with -E for better syntax
$ grep -Ef patterns.txt /public/zaciatocnik.txt
```

---

## Combining Options

You can combine `-e` with many other grep options.

### Multiple Patterns with Various Options

```bash
# Case-insensitive, recursive, with line numbers
$ grep -rinE -e "\<system\>" -e "\<subor\>" /path/to/directory/

# Inverted match (lines NOT containing any pattern)
$ grep -vE -e "\<system\>" -e "\<subor\>" /public/zaciatocnik.txt

# Count matches, case-insensitive
$ grep -ciE -e "\<system\>" -e "\<subor\>" /public/zaciatocnik.txt

# Show only matching filenames
$ grep -lE -e "\<system\>" -e "\<subor\>" *.txt
```

---

## Visual Comparison

### Single Pattern

```
grep "system" file
     ↓
Searches for: system
```

### Multiple Patterns with -e

```
grep -e "system" -e "subor" -e "sa" file
       ↓           ↓          ↓
Searches for: system OR subor OR sa
```

### Alternation with |

```
grep -E "(system|subor|sa)" file
         ↓       ↓      ↓
Searches for: system OR subor OR sa
```

---

## Common Mistakes

### Mistake 1: Forgetting -e with Multiple Patterns

```bash
# WRONG - Only searches for "system", ignores "subor"
$ grep "system" "subor" /public/zaciatocnik.txt

# CORRECT - Use -e for each pattern
$ grep -e "system" -e "subor" /public/zaciatocnik.txt
```

### Mistake 2: Mixing -e with Alternation Syntax

```bash
# WRONG - Confusing syntax
$ grep -e "(system|subor)" /public/zaciatocnik.txt

# CORRECT - Choose one approach:
$ grep -e "system" -e "subor" /public/zaciatocnik.txt
# OR
$ grep -E "(system|subor)" /public/zaciatocnik.txt
```

### Mistake 3: Not Using Quotes

```bash
# WRONG - Shell may interpret special characters
$ grep -e \<system\> -e \<subor\> /public/zaciatocnik.txt

# CORRECT - Quote patterns
$ grep -e "\<system\>" -e "\<subor\>" /public/zaciatocnik.txt
```

### Mistake 4: Using -f with Wrong Pattern Format

```bash
# If patterns.txt contains: (system|subor)
# And you run:
$ grep -f patterns.txt /public/zaciatocnik.txt
# It will search for literal string "(system|subor)" in BRE mode!

# CORRECT - Use -E with -f for ERE patterns
$ grep -Ef patterns.txt /public/zaciatocnik.txt
```

---

## Decision Guide: Which Method to Use?

| Scenario | Recommended Method | Example |
|----------|-------------------|---------|
| 2-3 simple patterns | Alternation `\|` | `grep -E "(a\|b\|c)"` |
| Many patterns (5+) | Multiple `-e` | `grep -e "a" -e "b" -e "c"` |
| Dynamic patterns in scripts | Multiple `-e` | Loop to build `-e` args |
| Patterns stored in file | `-f` flag | `grep -f patterns.txt` |
| Mix of simple and complex | Multiple `-e` | Each pattern separate |
| Very long pattern list | `-f` flag | Easier to manage |

---

## Key Takeaways

1. **`-e` flag** allows multiple patterns in one grep command
2. **Each `-e`** specifies one pattern to search for
3. **Lines match if they contain ANY pattern** (logical OR)
4. **`-e` can be combined with `-E`** for Extended Regular Expressions
5. **`-e` vs `|` alternation** - both work, choose based on use case
6. **Multiple `-e` flags** are clearer for many patterns
7. **`|` alternation** is more compact for 2-3 patterns
8. **`-f` flag** reads patterns from a file (one per line)
9. **Always quote patterns** to protect special characters
10. **Combine with other options** like `-i`, `-n`, `-c`, `-r`

---


# grep - Practical Examples and Common Pitfalls

## Practical Examples and Use Cases

This section demonstrates real-world scenarios where grep is commonly used, along with common mistakes and tricky cases to avoid.

---

## Log File Analysis

### Finding Errors in Logs

```bash
# Find all error messages
$ grep -i "error" /var/log/bootstrap.log

# Find errors with line numbers
$ grep -in "error" /var/log/bootstrap.log

# Count how many error lines
$ grep -ic "error" /var/log/bootstrap.log

# Find errors but exclude warnings
$ grep -i "error" /var/log/bootstrap.log | grep -iv "warning"
```

### Finding Multiple Log Levels

```bash
# Find ERROR or FATAL messages
$ grep -E "(ERROR|FATAL)" /var/log/bootstrap.log

# Or using -e flag
$ grep -e "ERROR" -e "FATAL" /var/log/bootstrap.log

# Case-insensitive search for error, warning, or critical
$ grep -iE "(error|warning|critical)" /var/log/bootstrap.log
```

### Time-Based Log Search

```bash
# Find logs from a specific hour
$ grep "2024-10-14 15:" /var/log/bootstrap.log

# Find logs from today (if date format is known)
$ grep "$(date +%Y-%m-%d)" /var/log/bootstrap.log
```

---

## Searching Source Code

### Finding Function Definitions

```bash
# Find function definitions in Python files
$ grep -rn "^def " /path/to/project/

# Find class definitions
$ grep -rn "^class " /path/to/project/

# Find TODO comments
$ grep -rn "TODO" /path/to/project/ --include="*.py"
```

### Finding Specific Patterns in Code

```bash
# Find all files containing a specific function call
$ grep -rl "connect_database" /path/to/project/

# Find lines with both "import" and "requests"
$ grep "import" /path/to/project/*.py | grep "requests"

# Find variable assignments
$ grep -E "^\s*[a-zA-Z_][a-zA-Z0-9_]*\s*=" file.py
```

---

## Configuration File Management

### Finding Active Configuration

```bash
# Show non-comment, non-empty lines from config
$ grep -v "^#" /etc/config.conf | grep -v "^$"

# Or in one command
$ grep -Ev "^#|^$" /etc/config.conf

# Find specific settings
$ grep "^[^#]*database" /etc/config.conf
```

### Searching Multiple Config Files

```bash
# Find which config file contains a setting
$ grep -l "max_connections" /etc/*.conf

# Search with context (show 2 lines before and after)
$ grep -A 2 -B 2 "max_connections" /etc/config.conf
```

---

## Process Management

### Finding Running Processes

```bash
# Find Apache processes
$ ps aux | grep "apache"

# Find Apache processes, exclude grep itself
$ ps aux | grep "apache" | grep -v "grep"

# Better way: use pgrep
$ pgrep -f apache

# Find processes by multiple names
$ ps aux | grep -E "(apache|nginx|mysql)"
```

---

## Network and System Information

### Searching Network Connections

```bash
# Find established connections
$ netstat -an | grep "ESTABLISHED"

# Find connections on port 80
$ netstat -an | grep ":80"

# Find listening ports
$ netstat -tuln | grep "LISTEN"
```

### Searching System Information

```bash
# Find specific hardware info
$ lsusb | grep -i "keyboard"

# Find mounted filesystems
$ mount | grep "^/dev"

# Find specific package installed
$ dpkg -l | grep "python"
```

---

## Text Processing and Filtering

### Extracting Specific Data

```bash
# Extract email addresses (simple pattern)
$ grep -Eo "[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}" file.txt

# Extract IP addresses
$ grep -Eo "([0-9]{1,3}\.){3}[0-9]{1,3}" file.txt

# Extract URLs
$ grep -Eo "https?://[a-zA-Z0-9./?=_%:-]*" file.txt
```

### Filtering Command Output

```bash
# Show only directories from ls
$ ls -la | grep "^d"

# Show only files modified in October
$ ls -la | grep "Oct"

# Filter environment variables
$ env | grep -i "path"
```

---

## Working with /public/zaciatocnik.txt

### Real Examples from Your File

```bash
# Find all words starting with 's'
$ grep "\<s" /public/zaciatocnik.txt

# Find exact word "system"
$ grep -w "system" /public/zaciatocnik.txt

# Find "system" or "subor"
$ grep -E "\<(system|subor)\>" /public/zaciatocnik.txt

# Find words with 2-4 letters
$ grep -E "\<[a-z]{2,4}\>" /public/zaciatocnik.txt

# Count lines containing "s"
$ grep -c "s" /public/zaciatocnik.txt

# Show line numbers for matches
$ grep -n "system" /public/zaciatocnik.txt
```

---

## Common Pitfalls and Tricky Cases

### Pitfall 1: Not Quoting Patterns

**Problem:** Shell interprets special characters before grep sees them.

```bash
# WRONG - Shell expands *
$ grep *.txt file
# If current directory has "a.txt", this becomes: grep a.txt file

# CORRECT - Quote the pattern
$ grep "*.txt" file
$ grep '*.txt' file
```

### Pitfall 2: Regex vs Literal Strings

**Problem:** Special characters in regex have different meanings.

```bash
# WRONG - '.' matches any character, not literal dot
$ grep "192.168.1.1" /etc/hosts
# This matches "192.168.1.1" but also "192x168y1z1"

# CORRECT - Escape the dots for literal match
$ grep "192\.168\.1\.1" /etc/hosts

# OR use -F for literal/fixed string search
$ grep -F "192.168.1.1" /etc/hosts
```

### Pitfall 3: Forgetting Word Boundaries

**Problem:** Pattern matches as part of larger words.

```bash
# WRONG - Matches "system", "filesystem", "subsystem"
$ grep "system" /public/zaciatocnik.txt

# CORRECT - Use word boundaries for exact word
$ grep "\<system\>" /public/zaciatocnik.txt
# OR
$ grep -w "system" /public/zaciatocnik.txt
```

### Pitfall 4: Case Sensitivity Issues

**Problem:** Missing matches due to case differences.

```bash
# WRONG - Misses "ERROR", "Error", "error"
$ grep "error" logfile.txt

# CORRECT - Use -i for case-insensitive
$ grep -i "error" logfile.txt
```

### Pitfall 5: Greedy vs Non-Greedy Matching

**Problem:** `.*` matches as much as possible (greedy).

```bash
# Text: "<tag>content</tag> more <tag>content</tag>"

# WRONG - Matches from first < to last >
$ echo "<tag>content</tag> more <tag>content</tag>" | grep -o "<.*>"
# Output: <tag>content</tag> more <tag>content</tag>

# Note: grep doesn't support non-greedy quantifiers
# You need other tools like perl or awk for non-greedy matching
```

### Pitfall 6: Special Characters in File Names

**Problem:** Spaces or special characters in file names cause issues.

```bash
# WRONG - Shell splits on spaces
$ grep "pattern" my file.txt
# This searches for "pattern" in files named "my" and "file.txt"

# CORRECT - Quote the filename
$ grep "pattern" "my file.txt"
```

### Pitfall 7: Empty Pattern

**Problem:** Empty pattern matches everything (or causes confusion).

```bash
# WRONG - Empty pattern
$ grep "" file.txt
# Matches every line!

# CORRECT - Be explicit about what you want
$ grep "." file.txt  # Non-empty lines
$ cat file.txt       # Just show the file
```

### Pitfall 8: Incorrect Escape Sequences

**Problem:** Mixing BRE and ERE syntax.

```bash
# WRONG - Using ERE syntax without -E
$ grep "\<s(a|y)\>" /public/zaciatocnik.txt
# Doesn't work! Need -E for | without escaping

# CORRECT - Use -E for ERE
$ grep -E "\<s(a|y)\>" /public/zaciatocnik.txt

# OR use BRE with escaped characters
$ grep "\<s\(a\|y\)\>" /public/zaciatocnik.txt
```

### Pitfall 9: Character Range Mistakes

**Problem:** Invalid or unexpected character ranges.

```bash
# WRONG - Invalid range (a > Z in ASCII)
$ grep "[a-Z]" /public/zaciatocnik.txt
# Error or unexpected behavior

# WRONG - Includes unexpected punctuation
$ grep "[A-z]" /public/zaciatocnik.txt
# Matches: A-Z, [\]^_`, a-z (6 punctuation chars!)

# CORRECT - Use proper ranges
$ grep "[A-Za-z]" /public/zaciatocnik.txt
# OR use POSIX class
$ grep "[[:alpha:]]" /public/zaciatocnik.txt
```

### Pitfall 10: Forgetting Anchors

**Problem:** Pattern matches anywhere in the line.

```bash
# WRONG - Matches lines where "error" appears anywhere
$ grep "error" logfile.txt
# Matches: "error message", "no errors", "terror"

# CORRECT - Use anchors for specific positions
$ grep "^error" logfile.txt      # Lines starting with "error"
$ grep "error$" logfile.txt      # Lines ending with "error"
$ grep "^error$" logfile.txt     # Lines containing only "error"
```

### Pitfall 11: Multiple grep vs Single Complex Pattern

**Problem:** Inefficiency with multiple grep calls.

```bash
# LESS EFFICIENT - Multiple greps in pipeline
$ grep "error" file.txt | grep "database" | grep -v "warning"

# MORE EFFICIENT - Combine patterns
$ grep "error" file.txt | grep "database" | grep -v "warning"
# This is actually fine for clarity

# ALTERNATIVE - Single complex pattern (may be less readable)
$ grep -E "^(?=.*error)(?=.*database)(?!.*warning)" file.txt
# Note: grep doesn't support lookahead, this won't work!
# Multiple pipes are often the right approach
```

### Pitfall 12: Not Using -r for Directory Search

**Problem:** Forgetting to search recursively.

```bash
# WRONG - Only searches files in current directory
$ grep "TODO" *.py
# Misses files in subdirectories

# CORRECT - Use -r for recursive search
$ grep -r "TODO" . --include="*.py"
```

### Pitfall 13: Binary Files in Output

**Problem:** grep tries to search binary files.

```bash
# WRONG - May produce garbage output or warnings
$ grep -r "pattern" /path/

# CORRECT - Skip binary files
$ grep -rI "pattern" /path/
# -I skips binary files

# OR explicitly handle binary files
$ grep -r "pattern" /path/ --binary-files=without-match
```

### Pitfall 14: Performance with Large Files

**Problem:** Searching very large files takes too long.

```bash
# SLOW - grep reads entire file
$ grep "pattern" huge_file.log

# FASTER - Stop after first match
$ grep -m 1 "pattern" huge_file.log

# FASTER - Use more specific patterns
$ grep "^2024-10-14.*ERROR" huge_file.log
# More specific pattern = fewer matches processed
```

### Pitfall 15: Inverted Match Confusion

**Problem:** `-v` doesn't work as expected with multiple patterns.

```bash
# WRONG - This doesn't exclude both patterns
$ grep -v "error" file.txt | grep -v "warning" file.txt
# The second grep searches its own input, not the file!

# CORRECT - Pipeline properly
$ grep -v "error" file.txt | grep -v "warning"

# OR use ERE
$ grep -Ev "(error|warning)" file.txt
```

---

## Best Practices

### 1. Always Quote Your Patterns

```bash
# Good practice
$ grep "pattern" file
$ grep 'pattern' file
```

### 2. Use -E for Complex Patterns

```bash
# Readable with -E
$ grep -E "\<(word1|word2|word3)\>" file

# Harder to read without -E
$ grep "\<\(word1\|word2\|word3\)\>" file
```

### 3. Use POSIX Character Classes

```bash
# Better (portable, locale-aware)
$ grep "[[:alnum:]]" file

# Works but less portable
$ grep "[A-Za-z0-9]" file
```

### 4. Use -F for Literal Strings

```bash
# When searching for literal dots, brackets, etc.
$ grep -F "192.168.1.1" file
$ grep -F "value[0]" file
```

### 5. Combine Options for Efficiency

```bash
# Good combination of useful flags
$ grep -rinE "pattern" /path/
# -r: recursive
# -i: case-insensitive  
# -n: line numbers
# -E: extended regex
```

### 6. Test Patterns Before Destructive Operations

```bash
# Always test first
$ grep "pattern" file

# Then use in pipeline
$ grep "pattern" file | xargs rm
```

### 7. Use Context Options for Debugging

```bash
# Show context around matches
$ grep -C 3 "error" logfile.txt  # 3 lines before and after
$ grep -A 5 "error" logfile.txt  # 5 lines after
$ grep -B 2 "error" logfile.txt  # 2 lines before
```

---

## Quick Troubleshooting Guide

| Problem | Likely Cause | Solution |
|---------|--------------|----------|
| No matches found | Case sensitivity | Use `-i` flag |
| Too many matches | Pattern too broad | Add word boundaries `\<` `\>` |
| Special chars not working | Missing quotes | Quote the pattern |
| Wrong syntax error | BRE vs ERE confusion | Use `-E` or escape properly |
| Binary file match | Searching binary files | Use `-I` to skip |
| Slow performance | Large files/broad pattern | Use `-m` to limit matches |
| Pattern matches too much | Greedy matching | Be more specific with pattern |
| Can't find in subdirectories | Not recursive | Use `-r` flag |

---

## Summary: Common grep Mistakes

1. **Not quoting patterns** - Shell interprets special characters
2. **Treating regex as literal** - `.` `*` `[]` have special meanings
3. **Forgetting word boundaries** - Matches partial words
4. **Case sensitivity** - Missing uppercase/lowercase variants
5. **Invalid character ranges** - `[a-Z]` is wrong, `[A-z]` includes punctuation
6. **Mixing BRE and ERE** - Know when to use `-E`
7. **Not escaping metacharacters** - In BRE, must escape `\(` `\{` `\|`
8. **Forgetting anchors** - `^` for start, `$` for end
9. **Binary files in output** - Use `-I` to skip
10. **Inefficient patterns** - Be as specific as possible

---

## Key Takeaways

1. **Always quote patterns** to protect from shell interpretation
2. **Use `-E`** for complex patterns with alternation and grouping
3. **Use `-i`** when case doesn't matter
4. **Use word boundaries** `\<` `\>` or `-w` for exact word matches
5. **Use POSIX classes** `[[:alpha:]]` for portability
6. **Use `-F`** for literal string searches (no regex)
7. **Test patterns** before using in destructive operations
8. **Use context options** `-A`, `-B`, `-C` for debugging
9. **Know your regex mode** - BRE (default) vs ERE (`-E`)
10. **Remember ASCII order** - uppercase before lowercase

---

## Practical Examples and Use Cases

This section demonstrates real-world scenarios where grep is commonly used, along with common mistakes and tricky cases to avoid.

---

## Log File Analysis

### Finding Errors in Logs

```bash
# Find all error messages in bootstrap log
$ grep -i "error" /var/log/bootstrap.log

# Find errors with line numbers
$ grep -in "error" /var/log/bootstrap.log

# Count how many error lines
$ grep -ic "error" /var/log/bootstrap.log

# Find errors but exclude warnings
$ grep -i "error" /var/log/bootstrap.log | grep -iv "warning"
```

### Finding Multiple Log Levels

```bash
# Find ERROR or FATAL messages
$ grep -E "(ERROR|FATAL)" /var/log/bootstrap.log

# Or using -e flag
$ grep -e "ERROR" -e "FATAL" /var/log/bootstrap.log

# Case-insensitive search for error, warning, or critical
$ grep -iE "(error|warning|critical)" /var/log/bootstrap.log
```

### Finding Specific Events

```bash
# Find all "started" events
$ grep "started" /var/log/bootstrap.log

# Find "failed" events with context (2 lines before and after)
$ grep -A 2 -B 2 "failed" /var/log/bootstrap.log

# Find lines with timestamps (if log has timestamps)
$ grep "[0-9]\{2\}:[0-9]\{2\}:[0-9]\{2\}" /var/log/bootstrap.log
```

---

## Working with /public/zaciatocnik.txt

### Finding Words and Patterns

```bash
# Find all words starting with 's'
$ grep "\<s" /public/zaciatocnik.txt

# Find exact word "system"
$ grep -w "system" /public/zaciatocnik.txt

# Find "system" or "subor"
$ grep -E "\<(system|subor)\>" /public/zaciatocnik.txt

# Find words with 2-4 letters
$ grep -E "\<[a-z]{2,4}\>" /public/zaciatocnik.txt

# Count lines containing "s"
$ grep -c "s" /public/zaciatocnik.txt

# Show line numbers for matches
$ grep -n "system" /public/zaciatocnik.txt
```

### Pattern Variations

```bash
# Find words starting with 's' followed by lowercase letters
$ grep "\<s[a-z]*\>" /public/zaciatocnik.txt

# Find words starting with 's', exactly 2 letters after
$ grep "\<s[a-z]\{2\}\>" /public/zaciatocnik.txt

# Find words with alphanumeric characters
$ grep "\<s[[:alnum:]]*\>" /public/zaciatocnik.txt

# Case-insensitive search
$ grep -i "system" /public/zaciatocnik.txt
```

---

## Process and System Commands

### Finding Running Processes

The idea: Filter process lists to find specific programs.

```bash
# Find Apache processes (concept)
$ ps aux | grep "apache"

# Find processes, exclude grep itself
$ ps aux | grep "apache" | grep -v "grep"

# Find processes by multiple names
$ ps aux | grep -E "(apache|nginx|mysql)"
```

### Filtering System Information

The idea: Extract specific information from system commands.

```bash
# Find specific hardware (concept)
$ lsusb | grep -i "keyboard"

# Find mounted filesystems (concept)
$ mount | grep "^/dev"

# Filter environment variables
$ env | grep -i "path"
```

---

## Text Processing Concepts

### Extracting Patterns

The idea: Use grep to find specific text patterns.

```bash
# Extract lines with email-like patterns (concept)
$ grep -E "[a-zA-Z0-9]+@[a-zA-Z0-9]+\.[a-z]+" /public/zaciatocnik.txt

# Extract lines with numbers
$ grep "[0-9]" /public/zaciatocnik.txt

# Extract lines starting with uppercase
$ grep "^[A-Z]" /public/zaciatocnik.txt
```

### Filtering Output

The idea: Show only relevant lines from command output.

```bash
# Show only directories from ls
$ ls -la | grep "^d"

# Filter specific entries
$ ls -la | grep "txt"

# Show environment PATH
$ env | grep "PATH"
```

---

## Common Pitfalls and Tricky Cases

### Pitfall 1: Not Quoting Patterns

**Problem:** Shell interprets special characters before grep sees them.

```bash
# WRONG - Shell expands *
$ grep *.txt /public/zaciatocnik.txt
# If current directory has "a.txt", this becomes: grep a.txt /public/zaciatocnik.txt

# CORRECT - Quote the pattern
$ grep "*.txt" /public/zaciatocnik.txt
$ grep '*.txt' /public/zaciatocnik.txt
```

**Key idea:** Always quote patterns to protect them from the shell.

---

### Pitfall 2: Regex vs Literal Strings

**Problem:** Special characters in regex have different meanings.

```bash
# WRONG - '.' matches any character, not literal dot
$ grep "file.txt" /public/zaciatocnik.txt
# This matches "file.txt" but also "fileAtxt", "file_txt"

# CORRECT - Escape the dot for literal match
$ grep "file\.txt" /public/zaciatocnik.txt

# OR use -F for literal/fixed string search
$ grep -F "file.txt" /public/zaciatocnik.txt
```

**Key idea:** Remember that `.` `*` `[]` have special meanings in regex.

---

### Pitfall 3: Forgetting Word Boundaries

**Problem:** Pattern matches as part of larger words.

```bash
# WRONG - Matches "system", "filesystem", "subsystem"
$ grep "system" /public/zaciatocnik.txt

# CORRECT - Use word boundaries for exact word
$ grep "\<system\>" /public/zaciatocnik.txt
# OR
$ grep -w "system" /public/zaciatocnik.txt
```

**Key idea:** Use `\<` and `\>` or `-w` for whole word matching.

---

### Pitfall 4: Case Sensitivity Issues

**Problem:** Missing matches due to case differences.

```bash
# WRONG - Misses "ERROR", "Error", "error"
$ grep "error" /var/log/bootstrap.log

# CORRECT - Use -i for case-insensitive
$ grep -i "error" /var/log/bootstrap.log
```

**Key idea:** Use `-i` when you want all case variations.

---

### Pitfall 5: Greedy Matching

**Problem:** `.*` matches as much as possible (greedy).

```bash
# Text: "<tag>content</tag> more <tag>content</tag>"

# WRONG - Matches from first < to last >
$ echo "<tag>content</tag> more <tag>content</tag>" | grep -o "<.*>"
# Output: <tag>content</tag> more <tag>content</tag>
```

**Key idea:** `.*` is greedy - it matches the longest possible string.

---

### Pitfall 6: Special Characters in File Names

**Problem:** Spaces or special characters in file names cause issues.

```bash
# WRONG - Shell splits on spaces
$ grep "pattern" my file.txt
# This searches for "pattern" in files named "my" and "file.txt"

# CORRECT - Quote the filename
$ grep "pattern" "my file.txt"
```

**Key idea:** Quote file names with spaces or special characters.

---

### Pitfall 7: Empty Pattern

**Problem:** Empty pattern matches everything.

```bash
# WRONG - Empty pattern
$ grep "" /public/zaciatocnik.txt
# Matches every line!

# CORRECT - Be explicit
$ grep "." /public/zaciatocnik.txt  # Non-empty lines
```

**Key idea:** Empty patterns match all lines.

---

### Pitfall 8: Incorrect Escape Sequences

**Problem:** Mixing BRE and ERE syntax.

```bash
# WRONG - Using ERE syntax without -E
$ grep "\<s(a|y)\>" /public/zaciatocnik.txt
# Doesn't work! Need -E for | without escaping

# CORRECT - Use -E for ERE
$ grep -E "\<s(a|y)\>" /public/zaciatocnik.txt

# OR use BRE with escaped characters
$ grep "\<s\(a\|y\)\>" /public/zaciatocnik.txt
```

**Key idea:** Know when you need `-E` and when to escape special chars.

---

### Pitfall 9: Character Range Mistakes

**Problem:** Invalid or unexpected character ranges.

```bash
# WRONG - Invalid range (a > Z in ASCII)
$ grep "[a-Z]" /public/zaciatocnik.txt
# Error or unexpected behavior

# WRONG - Includes unexpected punctuation
$ grep "[A-z]" /public/zaciatocnik.txt
# Matches: A-Z, [\]^_`, a-z (6 punctuation chars!)

# CORRECT - Use proper ranges
$ grep "[A-Za-z]" /public/zaciatocnik.txt
# OR use POSIX class
$ grep "[[:alpha:]]" /public/zaciatocnik.txt
```

**Key idea:** `[A-z]` includes punctuation; use `[A-Za-z]` or `[[:alpha:]]`.

---

### Pitfall 10: Forgetting Anchors

**Problem:** Pattern matches anywhere in the line.

```bash
# WRONG - Matches lines where "error" appears anywhere
$ grep "error" /var/log/bootstrap.log
# Matches: "error message", "no errors", "terror"

# CORRECT - Use anchors for specific positions
$ grep "^error" /var/log/bootstrap.log      # Lines starting with "error"
$ grep "error$" /var/log/bootstrap.log      # Lines ending with "error"
$ grep "^error$" /var/log/bootstrap.log     # Lines containing only "error"
```

**Key idea:** Use `^` for line start, `$` for line end.

---

### Pitfall 11: Multiple grep Inefficiency

**Problem:** Using multiple grep commands when one would work.

```bash
# LESS EFFICIENT - Multiple greps in pipeline
$ grep "error" /var/log/bootstrap.log | grep "failed"

# MORE EFFICIENT - Single pattern with multiple requirements
$ grep "error.*failed\|failed.*error" /var/log/bootstrap.log
```

**Key idea:** Sometimes multiple greps are fine for readability; optimize if needed.

---

### Pitfall 12: Not Using -r for Directory Search

**Problem:** Forgetting to search recursively.

```bash
# WRONG - Only searches specified files
$ grep "TODO" /public/*
# Misses files in subdirectories

# CORRECT - Use -r for recursive search
$ grep -r "TODO" /public/
```

**Key idea:** Use `-r` to search all files in a directory tree.

---

### Pitfall 13: Binary Files in Output

**Problem:** grep tries to search binary files.

```bash
# WRONG - May produce garbage output or warnings
$ grep -r "pattern" /public/

# CORRECT - Skip binary files
$ grep -rI "pattern" /public/
# -I skips binary files
```

**Key idea:** Use `-I` to ignore binary files.

---

### Pitfall 14: Performance with Large Files

**Problem:** Searching very large files takes too long.

```bash
# SLOW - grep reads entire file
$ grep "pattern" /var/log/bootstrap.log

# FASTER - Stop after first match
$ grep -m 1 "pattern" /var/log/bootstrap.log

# FASTER - Use more specific patterns
$ grep "^ERROR" /var/log/bootstrap.log
# More specific = fewer false matches
```

**Key idea:** Use `-m` to limit matches; be more specific with patterns.

---

### Pitfall 15: Inverted Match Confusion

**Problem:** `-v` doesn't work as expected with pipes.

```bash
# WRONG - Second grep operates on first grep's output, not the file
$ grep -v "error" /var/log/bootstrap.log | grep -v "warning" /var/log/bootstrap.log

# CORRECT - Pipeline properly
$ grep -v "error" /var/log/bootstrap.log | grep -v "warning"

# OR use ERE
$ grep -Ev "(error|warning)" /var/log/bootstrap.log
```

**Key idea:** In pipelines, each grep operates on the previous command's output.

---

## Best Practices

### 1. Always Quote Your Patterns

```bash
# Good practice
$ grep "pattern" file
$ grep 'pattern' file
```

### 2. Use -E for Complex Patterns

```bash
# Readable with -E
$ grep -E "\<(word1|word2|word3)\>" /public/zaciatocnik.txt

# Harder to read without -E
$ grep "\<\(word1\|word2\|word3\)\>" /public/zaciatocnik.txt
```

### 3. Use POSIX Character Classes

```bash
# Better (portable, locale-aware)
$ grep "[[:alnum:]]" /public/zaciatocnik.txt

# Works but less portable
$ grep "[A-Za-z0-9]" /public/zaciatocnik.txt
```

### 4. Use -F for Literal Strings

```bash
# When searching for literal dots, brackets, etc.
$ grep -F "192.168.1.1" /var/log/bootstrap.log
$ grep -F "value[0]" /public/zaciatocnik.txt
```

### 5. Combine Options for Efficiency

```bash
# Good combination of useful flags
$ grep -rinE "pattern" /public/
# -r: recursive
# -i: case-insensitive  
# -n: line numbers
# -E: extended regex
```

### 6. Use Context Options for Debugging

```bash
# Show context around matches
$ grep -C 3 "error" /var/log/bootstrap.log  # 3 lines before and after
$ grep -A 5 "error" /var/log/bootstrap.log  # 5 lines after
$ grep -B 2 "error" /var/log/bootstrap.log  # 2 lines before
```

---

## Quick Troubleshooting Guide

| Problem | Likely Cause | Solution |
|---------|--------------|----------|
| No matches found | Case sensitivity | Use `-i` flag |
| Too many matches | Pattern too broad | Add word boundaries `\<` `\>` |
| Special chars not working | Missing quotes | Quote the pattern |
| Wrong syntax error | BRE vs ERE confusion | Use `-E` or escape properly |
| Binary file match | Searching binary files | Use `-I` to skip |
| Slow performance | Large files/broad pattern | Use `-m` to limit matches |
| Pattern matches too much | Greedy matching | Be more specific with pattern |
| Can't find in subdirectories | Not recursive | Use `-r` flag |

---

## Summary: Common grep Mistakes

1. **Not quoting patterns** - Shell interprets special characters
2. **Treating regex as literal** - `.` `*` `[]` have special meanings
3. **Forgetting word boundaries** - Matches partial words
4. **Case sensitivity** - Missing uppercase/lowercase variants
5. **Invalid character ranges** - `[a-Z]` is wrong, `[A-z]` includes punctuation
6. **Mixing BRE and ERE** - Know when to use `-E`
7. **Not escaping metacharacters** - In BRE, must escape `\(` `\{` `\|`
8. **Forgetting anchors** - `^` for start, `$` for end
9. **Binary files in output** - Use `-I` to skip
10. **Inefficient patterns** - Be as specific as possible

---

## Key Takeaways

1. **Always quote patterns** to protect from shell interpretation
2. **Use `-E`** for complex patterns with alternation and grouping
3. **Use `-i`** when case doesn't matter
4. **Use word boundaries** `\<` `\>` or `-w` for exact word matches
5. **Use POSIX classes** `[[:alpha:]]` for portability
6. **Use `-F`** for literal string searches (no regex)
7. **Use context options** `-A`, `-B`, `-C` for debugging
8. **Know your regex mode** - BRE (default) vs ERE (`-E`)
9. **Remember ASCII order** - uppercase before lowercase
10. **Use `-I`** to skip binary files in recursive searches

---