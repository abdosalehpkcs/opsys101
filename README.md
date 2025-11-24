# Bash scripting

## Introduction

Bash is a command interpreter on Unix-like systems. A Bash script is a plain text file with executable commands processed in sequence. Purpose: automate tasks, enforce repeatability, reduce manual error.

## Built-in Variables
Core variables used without prior definition:

- $0 holds script name.
- $1, $2, … hold positional parameters.
- $# holds count of parameters.
- $@ expands to all parameters.
- $? holds exit status of last command.
- $\$ holds PID of current script.
- $USER holds current user.
- $HOME holds user home directory.
- $PWD holds current working directory.

### Example

```bash
#!/bin/bash
echo "Script name: $0"
echo "First argument: $1"
echo "Argument count: $#"
echo "Exit status of last command: $?"
```
###  Interpreter Definition (Shebang)
The first line determines which interpreter runs the script.

Fixed path:
```bash
#!/bin/bash
```

Portable path:
```bash
#!/usr/bin/env bash
```

### Comments
Used to document intent. Bash ignores them.

```bash 
# This is a comment
```

### Variable Definition
Variables store values without spaces around the equals sign.
```bash
name="Alice"
count=5
path="/usr/local/bin"
```
Use a variable by prefixing with `$`:
```bash
echo "$name"
```

###  Executing a Script
Run a script directly when it has execute permission and a proper interpreter line.
```bash
./script.sh
```
Run via interpreter without execute permission:
```bash
bash script.sh
```

###  Adding Execute Permissions
Make the script executable using:
```bash
chmod +x script.sh
```

### Key Takeaways
- Shebang defines interpreter.
- Execution requires correct permissions.
- Positional variables allow input-driven behavior.
- Exit codes define success or failure paths.

# Script Explanation: 1.sh

## Script Content
```bash
#!/bin/bash
ls | wc -l
```

## Function
Counts the number of directory entries in the current working directory.

## Breakdown
- `ls` lists files and directories in the current directory.
- `|` pipes the output of `ls` into the next command.
- `wc -l` counts the number of lines received from standard input.

## Key Takeaways
- `wc -l` is a direct method to count items line-by-line.

# Script Explanation: 2.sh

## Script Content
```bash
#!/bin/bash

for d in /public/prednasky /public/ucebnove /public/priklady
do
    echo -n $d
    ls $d | wc -l
done

# podobne ako v C
#     init  podm  vyraz
for (( i=0 ; i<5 ; i++ ))
do
    echo "opakovanie $i"
done
```

## Function
Performs two loops. First loop iterates through fixed directories and prints each directory name with the count of its entries. Second loop uses C‑style syntax to repeat a message five times.

## Breakdown

### Loop 1: Directory Loop
- `for d in ...` assigns each listed path to variable `d`.
- `echo -n $d` prints directory name without newline.
- `ls $d | wc -l` counts entries inside each directory.

### Loop 2: C‑Style Loop
- `for (( i=0 ; i<5 ; i++ ))` uses initializer, condition, and increment.
- `echo "opakovanie $i"` outputs iteration index.

## Key Takeaways
- Bash supports space‑separated item lists in `for`.
- `-n` suppresses newline in `echo`.
- Piping directory listings into `wc -l` produces entry counts.
- C‑style loop is suited for numeric sequences.

# Script Explanation: 3.sh

## Script Content
```bash
#!/bin/bash

#veta=ahoj svet
veta="ahoj svet"
pocet=3

while [ $pocet -gt 0 ]; do
    echo $pocet $veta
    echo $pocet    $veta
    echo "$pocet   $veta"
    echo '$pocet   $veta'
    echo \$pocet  \$veta
    echo ""
    ((pocet--))
done
```

## Variable Definition
Variables are assigned without spaces around `=`:
```bash
veta="ahoj svet"
pocet=3
```
- `veta="ahoj svet"` stores a string with spaces because it is quoted.
- `pocet=3` stores a numeric value.

## Quoting
### Unquoted
```bash
echo $pocet $veta
```
Performs word splitting. Multiple spaces collapse to one.

### Double Quotes
```bash
echo "$pocet   $veta"
```
- Variables expand.
- Embedded spacing is preserved.

### Single Quotes
```bash
echo '$pocet   $veta'
```
- No variable expansion.
- Exact literal output is printed.

### Escaped Characters
```bash
echo \$pocet \$veta
```
- Backslash escapes the dollar sign.
- Displays `$pocet` and `$veta` literally.

## While Loop and Numeric Tests
```bash
while [ $pocet -gt 0 ]; do
```
`[` is the test command. Numeric operators:
- `-gt` greater than
- `-lt` less than
- `-eq` equal
- `-ne` not equal
- `-ge` greater or equal
- `-le` less or equal

## Decrement
```bash
((pocet--))
```
Arithmetic context reduces the value by one.

## Key Takeaways
- Quoting determines expansion and spacing behavior.
- Numeric comparisons require test operators such as `-gt` and `-lt`.
- Escaping allows literal printing of special characters.
- While loop repeats until numeric condition fails.


# Script Explanation: 4.sh (Rename Uppercase Files to Lowercase)

## Script Content

```bash
#!/bin/bash

for f in [A-Z]*; do
    echo "$f" | tr 'A-Z' 'a-z'
    mv -i "$f" "$(echo "$f" | tr A-Z a-z)"
    #mv -i "$f" "`echo "$f" | tr A-Z a-z`"
done
```

## What the Script Does

- Loops over all files whose names start with an uppercase letter (`[A-Z]*`).
- Prints a lowercase version of each name.
- Renames each file from its original (possibly uppercase) name to a lowercase name.

---

## `for f in [A-Z]*`

- `[A-Z]*` is a shell pattern (glob).
- It matches every file in the current directory whose name:
  - starts with a letter between `A` and `Z`
  - followed by zero or more characters (`*`).
- In each iteration, the variable `f` holds one matched filename.

This is a `for` loop over a pattern, not a hard-coded list.

---

## `tr` Command

`tr` reads from standard input and writes to standard output, translating or deleting characters.

### Basic usage in the script

```bash
echo "$f" | tr 'A-Z' 'a-z'
```

- `echo "$f"` prints the filename.
- `|` pipes it into `tr`.
- `tr 'A-Z' 'a-z'` maps each uppercase letter to the matching lowercase letter.

Result: the filename is shown in lowercase.

### Common `tr` variants and use cases

1. **Uppercase to lowercase**
   ```bash
   tr 'A-Z' 'a-z'
   ```

2. **Lowercase to uppercase**
   ```bash
   tr 'a-z' 'A-Z'
   ```

3. **Delete characters with `-d`**
   ```bash
   tr -d '0-9'     # remove digits
   ```

4. **Squeeze repeats with `-s`**
   ```bash
   tr -s ' '       # convert multiple spaces to a single space
   ```

5. **Complement set with `-c`**
   ```bash
   tr -cd '0-9'    # keep only digits, delete everything else
   ```

`tr` works on single characters, not strings. It is useful for:
- case conversion
- cleanup of input (removing unwanted characters)
- normalising whitespace
- simple filters before further processing

---

## `mv` Command and Renaming

### Usage in the script

```bash
mv -i "$f" "$(echo "$f" | tr A-Z a-z)"
```

- `mv` moves or renames files and directories.
- Here it is used only inside the same directory.
- So this is a rename: old name → new name.

#### `-i` option (interactive)
- `-i` asks before overwrite if the target name already exists.
- This protects from data loss:
  - If the lowercase name already exists, `mv` will prompt you.

### Other useful `mv` options

- `-n` : do not overwrite existing files (no prompt, just skip).
- `-v` : verbose, prints what is being moved or renamed.
- `-f` : force, overwrite without asking.

### Renaming files and directories with `mv`

**File rename:**
```bash
mv oldname.txt newname.txt
```

**Directory rename:**
```bash
mv olddir newdir
```

`mv` does not care if the source is a file or directory. If both are in the same parent directory, this is just a rename.

**Move to another directory:**
```bash
mv file.txt /path/to/dir/
mv mydir /path/to/dir/
```

---

## `cp` Command (for comparison)

`cp` copies files and directories. Often used together with `mv` in scripts.

### Important `cp` options

- `-r` or `-R` : recursive copy of directories.
- `-a` : archive mode (preserves permissions, timestamps, etc.; equivalent to `-dR --preserve=all` in many systems).
- `-p` : preserve file attributes (mode, ownership, timestamps).
- `-i` : interactive, ask before overwrite.
- `-u` : copy only when the source is newer than the destination.
- `-v` : verbose, show progress.

Examples:

```bash
cp file1 file2
cp -i file1 file2       # ask before overwriting file2
cp -r dir1 dir2         # copy directory recursively
cp -a dir1 backup_dir   # archive copy
```

---

## Command Substitution: `$( )` vs `` ` ` ``

In the script:

```bash
mv -i "$f" "$(echo "$f" | tr A-Z a-z)"
#mv -i "$f" "`echo "$f" | tr A-Z a-z`"
```

- `$(command)` and `` `command` `` both run a command and substitute its output into the line.
- `$( )` is the modern, clearer form:
  - Easier to read.
  - Easier to nest.
- Backticks are older and harder to read, especially inside other quotes.

The script uses `$( ... )` and comments the backtick version as an alternative.

---

## Key Takeaways

- `for f in [A-Z]*` loops over files matching a pattern.
- `tr` is used to transform characters (here: uppercase to lowercase).
- `mv` can rename both files and directories; `-i` protects against overwrite.
- `cp` copies, `mv` moves or renames; both have important safety flags (`-i`, `-n`, `-v`).
- Use `$( )` for command substitution instead of backticks for clearer scripts.


# Script Explanation: 5.sh (Argument Parsing and Flags)

## Script Content

```bash
#!/bin/bash

help="Help: $0 arg1 arg2 arg3 ... argN"

if [ "$#" == "0" ]; then
    echo "$help"
    exit 1
fi

unset debug
echo "Argumenty: $@"
#echo $1 ; shift ; echo $1

while (( "$#" )); do
    case "$1" in
        -d|-D)
            debug=''
            shift
            break
            ;;
        abc|cba)
            echo "pokracujem dalej"
            ;;
        -h)
            echo "$help"
            exit 0
            ;;
        -*)
            echo "Neznamy prepinac "$1""
            exit 1
            ;;
        *)
            break
            ;;
    esac
    shift
done
if [[ -v debug ]]; then echo "Dalsie argumenty: $@"; fi
```

---

## Help Text and `$0`

```bash
help="Help: $0 arg1 arg2 arg3 ... argN"
```

- `help` is a variable holding a usage message.
- `$0` expands to the name with which the script was invoked (e.g. `./5.sh` or `5.sh`).
- The help string is later printed to show how to call the script.

---

## Checking Argument Count: `"$#"` and `if [ ... ]`

```bash
if [ "$#" == "0" ]; then
    echo "$help"
    exit 1
fi
```

- `$#` is the number of positional parameters (arguments) passed to the script.
- Quoting `"$#"` prevents issues if it is empty (defensive habit).
- `[ ... ]` is the test command.

String comparison here uses `==` inside `[` `]`. For numeric comparison you should use `-eq`, `-gt`, etc.

If there are **zero arguments**:
- print the help text,
- exit with status `1` (non‑zero → error / incorrect usage).

---

## `unset debug` and Flag by Variable Existence

```bash
unset debug
```

- `unset` removes a variable from the environment.
- After `unset debug`, the variable `debug` is **not set at all**.

Later, the script uses this variable as a flag: if the user passes `-d` or `-D`, the script sets `debug` and checks for its existence at the end.

This pattern uses:
- variable **presence** instead of value to represent a boolean flag.

---

## Printing All Arguments: `$@`

```bash
echo "Argumenty: $@"
```

- `$@` expands to all positional parameters as separate words.
- When quoted as `"$@"`, each argument stays a separate word; here it is unquoted inside a double‑quoted string, which is still usually safe for simple demonstration.

---

## The `while (( "$#" ))` Loop

```bash
while (( "$#" )); do
    ...
done
```

- `(( ... ))` is arithmetic evaluation.
- Inside arithmetic context, `"$#"` is treated as a number (argument count).
- As long as the result is **non‑zero**, the condition is true.
- This is a compact idiom meaning: “loop while there are arguments left”.

Each turn, the loop will usually process `$1` (the first argument) and then `shift` to move to the next.

---

## `case` Statement and Patterns

```bash
case "$1" in
    -d|-D)
        debug=''
        shift
        break
        ;;
    abc|cba)
        echo "pokracujem dalej"
        ;;
    -h)
        echo "$help"
        exit 0
        ;;
    -*)
        echo "Neznamy prepinac "$1""
        exit 1
        ;;
    *)
        break
        ;;
esac
```

### General form

```bash
case WORD in
    PATTERN1)
        commands
        ;;
    PATTERN2|PATTERN3)
        commands
        ;;
    *)
        default
        ;;
esac
```

- `WORD` here is `$1`, the current argument.
- Each `PATTERN` uses shell wildcard patterns (not regular expressions).

### Patterns in this script

1. **`-d|-D`**

   ```bash
   -d|-D)
       debug=''
       shift
       break
       ;;
   ```

   - Matches `-d` **or** `-D`.
   - `debug=''` sets the debug flag by creating the variable.
   - `shift` discards this option from the argument list:
     - `$2` becomes `$1`, `$3` becomes `$2`, and so on.
     - `"$#"` decreases by 1.
   - `break` exits the `while` loop early after finding this option.

2. **`abc|cba`**

   ```bash
   abc|cba)
       echo "pokracujem dalej"
       ;;
   ```

   - Matches the literal strings `abc` or `cba`.
   - Prints a message and continues to the next argument (no break).

3. **`-h` (help option)**

   ```bash
   -h)
       echo "$help"
       exit 0
       ;;
   ```

   - Typical help flag.
   - Prints usage and exits normally (`0` → success).

4. **`-*` (unknown option)**

   ```bash
   -*)
       echo "Neznamy prepinac "$1""
       exit 1
       ;;
   ```

   - Matches anything that **starts with `-`** but did not match earlier cases.
   - Treats it as an unknown switch and exits with error.

5. **`*` (default)**

   ```bash
   *)
       break
       ;;
   ```

   - Matches anything (fallback).
   - Breaks out of the loop when the next argument is not an option.

This `case` block is a basic argument parser that:
- recognizes known options (`-d`, `-D`, `abc`, `cba`, `-h`),
- rejects unknown options starting with `-`,
- stops parsing when a non‑option argument is encountered.

---

## `shift` and Argument Consumption

Inside the loop:

- `shift` (unconditional at the end of the loop body) removes the current `$1` and shifts all remaining arguments left.
- After `shift`, the next argument becomes `$1`.

The `-d|-D` branch also calls `shift` inside the case and then `break`s, so that the outer `shift` at the end of the loop does not run for that branch (because the loop is already exited).

This use of `shift` allows step‑by‑step processing of arguments.

---

## Testing Variable Existence: `[[ -v debug ]]`

```bash
if [[ -v debug ]]; then echo "Dalsie argumenty: $@"; fi
```

- `[[ ... ]]` is Bash’s extended test syntax.
- `-v varname` returns true if the variable is set (even if empty).
- Here: `[[ -v debug ]]` is true if `debug` exists.

So:
- If the user passed `-d` or `-D`, `debug` was set and not unset again.
- In that case, the script prints:

  ```bash
  Dalsie argumenty: $@
  ```

  showing the remaining (non‑option) arguments.

This is a typical “debug mode” flag using variable existence.

---

## Important New Concepts In This Script

- **Usage / help string** built with `$0` to show correct invocation.
- **Argument count check** using `$#` and `[ ... ]`.
- **Error vs success exit codes**: `exit 1` for error, `exit 0` for normal termination.
- **`unset` and `[[ -v VAR ]]`** to model boolean flags based on variable presence.
- **`while (( "$#" ))`** as an idiom for looping while arguments remain.
- **`case` / `esac`** with multiple patterns (`-d|-D`, `abc|cba`, `-*`, `*`) for option parsing.
- **`shift`** to consume arguments one by one.
- **Separation of options and remaining arguments**, with a final check on the debug flag controlling extra output.


# Script Explanation: 6.sh (Arrays in Bash)

## Bash Arrays – Basics

Bash supports indexed arrays. Index starts at 0.

### Definition

```bash
# Space-separated elements
zoznam=(jeden dva tri styri pat "sest cele sedem")
```

- Parentheses `(...)` define an array.
- Elements are separated by spaces.
- Quotes keep spaces inside a single element:
  - `"sest cele sedem"` is **one** array element, not three.

### Accessing Elements

```bash
${zoznam[0]}   # first element
${zoznam[1]}   # second element
${zoznam[5]}   # sixth element
```

If you omit the index, Bash treats index 0 by default:
```bash
$zoznam        # same as ${zoznam[0]}
```

### All Elements

```bash
${zoznam[@]}   # all elements as separate words
${zoznam[*]}   # all elements, joined into one word if quoted as "${zoznam[*]}"
```

### Array Length

```bash
${#zoznam[@]}  # count of elements in array
${#zoznam[2]}  # character length of single element at index 2
```

- `#` before the name means “length”.
- With `[index]` → length of that element.
- With `[@]` or `[*]` → number of elements.

### Array Slicing

```bash
${zoznam[@]:start:count}
```

- `start` is starting index.
- `count` is how many elements to take.

Example:
```bash
${zoznam[@]:2:4}   # elements from index 2, total 4 elements
```

### Copying / Extending Arrays

You can build new arrays from existing ones:

```bash
new=(${old[@]} extra)
new=("${old[@]}" "extra with space")
```

- `("${old[@]}")` preserves elements exactly.
- Unquoted `(${old[@]})` re-splits elements on spaces.

---

## Script Content

```bash
#! /bin/bash

zoznam=(jeden dva tri styri pat "sest cele sedem")

echo $zoznam
echo ${zoznam[2]}
echo ${#zoznam[2]}
echo ${#zoznam[5]}
echo ${#zoznam[@]}

echo ${zoznam[@]:2:4}

zoznam1=(${zoznam[@]:0:6} sedem ${zoznam[$((7-${#zoznam[@]}))]})

echo ""

#zoznam=(${zoznam[@]} osem)
zoznam2=(${zoznam[@]} osem)
echo ${#zoznam2[@]}
echo ${zoznam2[@]}

zoznam2=("${zoznam[@]}" osem)
echo ${#zoznam2[@]}
echo ${zoznam2[@]}

for prvok in "${zoznam[@]}"; do
    echo "$prvok"
done

echo ""
```

---

## Step-by-Step Explanation

### 1 Initial Array

```bash
zoznam=(jeden dva tri styri pat "sest cele sedem")
```

Elements:
- `zoznam[0] = "jeden"`
- `zoznam[1] = "dva"`
- `zoznam[2] = "tri"`
- `zoznam[3] = "styri"`
- `zoznam[4] = "pat"`
- `zoznam[5] = "sest cele sedem"`

### 2 Basic Prints

```bash
echo $zoznam
```
- Prints element 0 (`jeden`), because `$zoznam` → `${zoznam[0]}`.

```bash
echo ${zoznam[2]}       # tri
echo ${#zoznam[2]}      # number of characters in "tri"  (3)
echo ${#zoznam[5]}      # number of characters in "sest cele sedem"
echo ${#zoznam[@]}      # number of elements in array (6)
```

### 3 Slicing

```bash
echo ${zoznam[@]:2:4}
```

- Start at index 2: `tri`
- Take 4 elements: `tri styri pat "sest cele sedem"`

So output is the 4 elements starting from `tri`.

### 4 Building `zoznam1`

```bash
zoznam1=(${zoznam[@]:0:6} sedem ${zoznam[$((7-${#zoznam[@]}))]})
```

Pieces:

1. `${zoznam[@]:0:6}`  
   - slice from index 0, length 6 → all current elements of `zoznam`.

2. Literal `sedem`.

3. `${zoznam[$((7-${#zoznam[@]}))]}`  
   - `${#zoznam[@]}` is the number of elements, here `6`.
   - `7 - 6 = 1`.
   - So this is `${zoznam[1]}` → `dva`.

So `zoznam1` becomes something like:
```bash
zoznam1=(jeden dva tri styri pat "sest cele sedem" sedem dva)
```

Note: elements get re-split because the expansion `(${zoznam[@]:0:6} ...)` is unquoted. The element with spaces (`"sest cele sedem"`) may be split into several words in `zoznam1`, depending on IFS. This line mainly demonstrates index math and slicing.

### 5 Extending Array Without Quotes: `zoznam2`

```bash
zoznam2=(${zoznam[@]} osem)
echo ${#zoznam2[@]}
echo ${zoznam2[@]}
```

- `(${zoznam[@]} osem)` expands all elements of `zoznam`, then adds `osem`.
- Because the expansion is unquoted, each element is subject to word splitting.
- The element `"sest cele sedem"` is split into three elements: `sest`, `cele`, `sedem`.
- `zoznam2` therefore has **more elements** than `zoznam` + 1.

### 6 Extending Array With Quotes: `zoznam2` again

```bash
zoznam2=("${zoznam[@]}" osem)
echo ${#zoznam2[@]}
echo ${zoznam2[@]}
```

- `("${zoznam[@]}" osem)` preserves each element exactly.
- `"sest cele sedem"` stays one element.
- `zoznam2` now has exactly original count + 1:
  - same as `zoznam`, plus `osem` as the last element.

This contrast shows why you use `"${array[@]}"` when you want to keep elements with spaces intact.

### 7 Looping Over Array Elements

```bash
for prvok in "${zoznam[@]}"; do
    echo "$prvok"
done
```

- Loops over each element in `zoznam`.
- Quotes around `"${zoznam[@]}"` ensure:
  - each element is treated as a separate word,
  - elements containing spaces are not split.
- Each element is printed on its own line.

### 8 Final Blank Line

```bash
echo ""
```

Just prints an empty line for clean output formatting.

---

## Key Takeaways

- Arrays in Bash are zero-indexed and defined with `( ... )`.
- `"sest cele sedem"` is a **single** element because of quotes.
- `${#array[@]}` gives the number of elements.
- `${#array[index]}` gives length of one element in characters.
- `${array[@]:start:count}` slices out a portion of the array.
- Unquoted `(${array[@]})` can split elements on spaces.
- Quoted `"${array[@]}"` preserves each element exactly.
- Use `for prvok in "${array[@]}"` to safely iterate over all elements, including those with spaces.


# Script Explanation: 7.sh (dirname, basename, quoting, paths with spaces)

## Script Content

```bash
#! /bin/bash

#cesta="/public/ucebnove/seminar _1/vim.txt"
cesta=/public/ucebnove/seminar_1/vim.txt

echo $(dirname $cesta)
echo $(basename $cesta)
echo $(dirname $(dirname $cesta))
echo $(basename $(dirname $cesta))
```

---

## Two Path Variants

### Path without spaces
```bash
cesta=/public/ucebnove/seminar_1/vim.txt
```

### Path with spaces (commented in script)
```bash
cesta="/public/ucebnove/seminar _1/vim.txt"
```

- Spaces in paths require quoting.
- Without quotes, word splitting occurs and dirname/basename receive wrong tokens.

---

## `dirname` and `basename`

These utilities process filesystem paths.

### `dirname`
- Returns the directory part of a path.
- Removes the last component.

Example:
```
dirname /a/b/c.txt → /a/b
```

### `basename`
- Returns the final component (file or directory name).

Example:
```
basename /a/b/c.txt → c.txt
basename /a/b/ → b
```

Both rely on correct quoting when paths contain spaces.

---

## Demonstrating Correct Quoting

### Without spaces – works even unquoted:
```bash
echo $(dirname $cesta)
echo $(basename $cesta)
echo $(dirname $(dirname $cesta))
echo $(basename $(dirname $cesta))
```

### With spaces – must use quotes:
```bash
cesta="/public/ucebnove/seminar _1/vim.txt"

echo "$(dirname "$cesta")"
echo "$(basename "$cesta")"
echo "$(dirname "$(dirname "$cesta")")"
echo "$(basename "$(dirname "$cesta")")"
```

Explanation:
- Quotes preserve the full path as a single argument.
- Nested command substitution also needs quoting inside each level.

---

## Step-by-Step Examples

Assume:  
```
cesta="/public/ucebnove/seminar _1/vim.txt"
```

### 1 `dirname "$cesta"`

Input:
```
/public/ucebnove/seminar _1/vim.txt
```

Output:
```
/public/ucebnove/seminar _1
```

### 2 `basename "$cesta"`

Output:
```
vim.txt
```

### 3 `dirname "$(dirname "$cesta")"`

First `dirname "$cesta"` → `/public/ucebnove/seminar _1`

Second `dirname "/public/ucebnove/seminar _1"` → `/public/ucebnove`

### 4 `basename "$(dirname "$cesta")"`

`dirname "$cesta"` → `/public/ucebnove/seminar _1`  
`basename "/public/ucebnove/seminar _1"` → `seminar _1`

---

## Additional Examples Using Quotes

### Example A: Nested levels
```bash
echo "$(dirname "$(dirname "$(dirname "$cesta")")")"
```

### Example B: Extract filename without extension
```bash
filename="$(basename "$cesta")"
echo "${filename%.*}"
```

### Example C: Extract only extension
```bash
echo "${filename##*.}"
```

### Example D: Build a new path
```bash
dir="$(dirname "$cesta")"
base="$(basename "$cesta")"
echo "$dir/new_$base"
```

---

## Key Points

- Always quote variables containing paths: `"$var"`.
- `dirname` strips the last path component.
- `basename` extracts the final component.
- Nested path operations require quoting at each substitution level.

# Script Explanation: 8.sh (IFS, Word Splitting, Command Substitution)

## Script Content

```bash
#! /bin/bash

ls -l

echo ""

for f in $(ls -l | head -3); do
    echo $f
done

echo ""

IFS='
'
for f in $(ls -l | head -3); do
    echo $f
done
```

---

## Purpose of the Script

The script demonstrates:
- How command substitution `$( ... )` interacts with word splitting.
- How the shell’s `IFS` (Internal Field Separator) influences splitting.
- Why parsing `ls` output is unsafe without controlling `IFS`.
- Differences between default splitting (splits on spaces, tabs, newlines) and splitting only by newline.

---

## First Command: Listing Files

```bash
ls -l
```

Displays directory contents in long format:
- permissions  
- number of links  
- owner  
- group  
- size  
- date  
- filename  

This is shown first so the user can compare with later loop output.

---

## Loop Without Modifying IFS (Default Splitting)

```bash
for f in $(ls -l | head -3); do
    echo $f
done
```

### What happens here

1. `$(ls -l | head -3)` produces **multiple lines** of output:
   ```
   total 10
   -rw-r--r-- 1 user group ...
   drwxr-xr-x 2 user group ...
   ```

2. The shell applies **word splitting** using the default `IFS`:
   - space  
   - tab  
   - newline

3. Therefore **every word** becomes a separate loop item.

### Example of splitting

Line:
```
-rw-r--r-- 1 user group 1234 Jan 10 file.txt
```

Splits into:
```
-rw-r--r--
1
user
group
1234
Jan
10
file.txt
```

The loop prints each token independently.

### Result

The loop does **not** iterate by line.  
It iterates by **words**, which destroys structure.

---

## Changing IFS to Preserve Line Structure

```bash
IFS='
'
for f in $(ls -l | head -3); do
    echo $f
done
```

### What changes

The new `IFS` is set to a **single newline**.

Default was: space, tab, newline  
New value: **newline only**

### Effect on splitting

- Splitting no longer occurs on spaces or tabs.
- Each line remains intact.
- The loop now iterates one **full line** at a time.

### Correct behavior

Lines such as:
```
-rw-r--r-- 1 user group 1234 Jan 10 file.txt
```

Stay **whole**, because spaces no longer split elements.

### Output

Now each printed `$f` is one complete line from `ls -l | head -3`.

---

## Why This Matters

### Parsing `ls` output is unsafe
- Filenames with spaces break scripts.
- Filenames with tabs break scripts.
- Filenames with newlines break everything.

This script demonstrates the issue clearly.

### Proper approaches (not shown in script)
- Use `find -print0` with null delimiters.
- Use `while IFS= read -r line` loops.
- Avoid parsing `ls` entirely.

---

## Key Takeaways

- Command substitution passes output through word splitting.
- Without modifying `IFS`, splitting uses space, tab, and newline.
- Setting `IFS` to newline allows line-accurate iteration.
- Quoting and field separation are critical in shell scripting.
- This script highlights the difference between **word-based** and **line-based** iteration.


