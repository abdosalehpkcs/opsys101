# Lab 5: Script Lab

Write, run, and read Bash scripts. Sections 1 to 5 explain script files, built-in variables, the shebang, variables, and how to run a script. Nine example scripts then cover pipelines, loops, quoting, renaming files, arguments, arrays, paths, word splitting, and `source` and `exec`.

The sample outputs come from a test user named `student`. Your user name, process IDs, dates, and file sizes will differ.

---

## Table of Contents

- [1. What Is a Bash Script](#1-what-is-a-bash-script)
- [2. Built-in Variables](#2-built-in-variables)
- [3. The Shebang: Fixed vs Portable](#3-the-shebang-fixed-vs-portable)
- [4. Defining Variables](#4-defining-variables)
- [5. Running a Script](#5-running-a-script)
- [6. Script 1: Count Directory Entries (`1.sh`)](#6-script-1-count-directory-entries-1sh)
- [7. Script 2: Two Kinds of `for` Loop (`2.sh`)](#7-script-2-two-kinds-of-for-loop-2sh)
- [8. Script 3: Quoting and the `while` Loop (`3.sh`)](#8-script-3-quoting-and-the-while-loop-3sh)
- [9. Script 4: Rename Files to Lowercase (`4.sh`)](#9-script-4-rename-files-to-lowercase-4sh)
- [10. Script 5: Arguments, `case`, and `shift` (`5.sh`)](#10-script-5-arguments-case-and-shift-5sh)
- [11. Script 6: Arrays (`6.sh`)](#11-script-6-arrays-6sh)
- [12. Script 7: Split a Path with `dirname` and `basename` (`7.sh`)](#12-script-7-split-a-path-with-dirname-and-basename-7sh)
- [13. Script 8: Word Splitting and `IFS` (`8.sh`)](#13-script-8-word-splitting-and-ifs-8sh)
- [14. Script 9: `source`, `.`, and `exec` (`9.sh`)](#14-script-9-source--and-exec-9sh)
- [15. Sources](#15-sources)

---

## 1. What Is a Bash Script

**Bash** is the program that reads your commands at the prompt and runs them. A **Bash script** is a plain text file that contains those same commands. Bash reads the file and runs the commands one by one, top to bottom, exactly as if you typed them.

Use a script when you want to:

- repeat a task without retyping it,
- get the same result every time, with no typing mistakes,
- share a procedure with someone else.

A script file has three kinds of lines:

```bash
#!/bin/bash
# Greet the current user.
echo "Hello, $USER"
```

| Line | Part | Role |
|---|---|---|
| `#!/bin/bash` | Shebang | Names the program that runs the file. Must be the very first line. |
| `# Greet the current user.` | Comment | A note for humans. Bash ignores everything from `#` to the end of the line. |
| `echo "Hello, $USER"` | Command | The actual work. Anything you can type at the prompt can go here. |

Blank lines are ignored. There is no compile step: you save the file and run it.

The `.sh` ending is **only a naming convention**. Linux does not look at the name; it uses the file's execute permission and first line to decide how to run it.

[Back to top](#table-of-contents)

---

## 2. Built-in Variables

Bash fills some variables for you, so a script can use them without defining them.

| Variable | Holds | Set by |
|---|---|---|
| `$0` | The script name, exactly as you typed it to run the script. | Bash |
| `$1`, `$2`, ... | Positional parameters: the 1st, 2nd, ... argument after the script name. Use `${10}` from the 10th on. | Bash |
| `$#` | The number of arguments. | Bash |
| `$@` | All arguments. Written as `"$@"`, each argument stays a separate word. | Bash |
| `$?` | The exit status of the last command: `0` means success, anything else means failure. | Bash |
| `$$` | The process ID (PID) of the shell running the script. | Bash |
| `$USER` | Your login name. | Your login session (environment) |
| `$HOME` | Your home directory. | Your login session (environment) |
| `$PWD` | The current working directory. | Bash, updated by every `cd` |

### One Script That Prints Them All

Save this as `vars.sh` in a practice directory, such as `~/lab5`:

```bash
#!/bin/bash
echo "Script name      \$0: $0"
echo "First argument   \$1: $1"
echo "Second argument  \$2: $2"
echo "Argument count   \$#: $#"
echo "All arguments    \$@: $@"
ls /no/such/dir 2>/dev/null
echo "Last exit status \$?: $?"
echo "Script PID       \$\$: $$"
echo "Current user  \$USER: $USER"
echo "Home dir      \$HOME: $HOME"
echo "Working dir    \$PWD: $PWD"
```

A backslash before `$` prints the dollar sign itself, so `\$0` shows the label `$0` and the following `$0` shows the value. Script 3 covers this kind of escaping in detail.

Run it with three arguments. [Section 5](#5-running-a-script) explains `chmod +x` and `./`.

```bash
cd ~/lab5
chmod +x vars.sh
./vars.sh apple "green pear" kiwi
```

Output:

```text
Script name      $0: ./vars.sh
First argument   $1: apple
Second argument  $2: green pear
Argument count   $#: 3
All arguments    $@: apple green pear kiwi
Last exit status $?: 2
Script PID       $$: 25
Current user  $USER: student
Home dir      $HOME: /home/student
Working dir    $PWD: /home/student/lab5
```

**Reading the output:**

- `"green pear"` is one argument because of the quotes. That is why `$2` is `green pear` and `$#` is `3`, not `4`.
- `$?` is `2` because `ls` failed: the directory does not exist. `2>/dev/null` only hides the error message (Lab 3 covers redirection). `$?` changes after every command, so read it immediately.
- `$$` is a different number on every run, because every run is a new process.
- `$PWD` is the directory you ran the script from, not the directory where the script file is stored.

[Back to top](#table-of-contents)

---

## 3. The Shebang: Fixed vs Portable

The shebang is `#!` followed by the path of the program that should run the script. When you run `./vars.sh apple`, the kernel reads this first line and actually starts:

```text
/bin/bash ./vars.sh apple
```

There are two common ways to write it:

```bash
#!/bin/bash
# Fixed: always run the program at exactly /bin/bash.
```

```bash
#!/usr/bin/env bash
# Portable: run env, which searches the directories in PATH and starts the first bash it finds.
```

**Why use fixed:** it is predictable. A changed `PATH`, or a fake `bash` placed in a directory listed in `PATH`, cannot change which program runs. Most Linux distributions have `/bin/bash`.

**Why use portable:** the same file runs wherever Bash is installed, and you can choose a newer Bash just by putting it first in `PATH`.

[Back to top](#table-of-contents)

---

## 4. Defining Variables

```bash
name=Alice
# Store the text Alice in the variable name.
echo "$name"
# Read the variable: put $ before its name. Prints: Alice
echo "${name}"
# Same thing, written with braces. Prints: Alice
```

The braces matter when letters, digits, or `_` follow the variable. Bash keeps reading the name until it reaches a character that cannot be part of a name:

```bash
echo "$name_file"
# Bash reads the name as name_file, a variable that does not exist. Prints an empty line.
echo "${name}_file"
# The braces end the name after name, then _file is added. Prints: Alice_file
```

**There must be no spaces around `=`.**

### Is `name=Alice` the Same as `name="Alice"`?

**Yes.** Bash removes the quotes before it stores the value, so the variable holds the same five characters in both cases. Single quotes give the same result too:

```bash
#!/bin/bash
name=Alice
echo "$name"
name="Alice"
echo "$name"
name='Alice'
echo "$name"
name=alice
echo "$name"
```

Output:

```text
Alice
Alice
Alice
alice
```

The last value is different only because of the lowercase a. Bash is case-sensitive: `alice` is not `Alice`, and `$Name` is a different variable from `$name`.

Quotes become necessary when the value contains spaces or special characters:

```bash
name="Alice Smith"
echo "$name"
# Alice Smith
```

### Common Mistakes

| Incorrect or fragile syntax | What happens |
|---|---|
| `name=Alice Smith` | Bash sets `name=Alice` only while it runs a command called `Smith`, then reports `Smith: command not found`. Afterwards `name` is still not set. |
| `name = Alice` | Bash runs a command called `name` with the arguments `=` and `Alice`, and reports `name: command not found`. |

**Naming rules:** use letters, digits, and `_`, and do not start with a digit. Use lowercase names for your own variables so you never overwrite system variables such as `USER`, `HOME`, or `PATH`.

[Back to top](#table-of-contents)

---

## 5. Running a Script

There are two ways to run a script:

```bash
./vars.sh apple
# The system runs the file itself: it needs execute permission and uses the shebang.
bash vars.sh apple
# You start bash and tell it which file to read: only read permission is needed, and the shebang is ignored.
```

Both ways start a **new process**: the script gets its own PID, and any variables it sets disappear when it ends.

### Add Execute Permission with `chmod +x`

A new file is not executable. Running it directly fails:

```bash
ls -l vars.sh
# -rw-r--r-- 1 student student 369 Oct 21 10:38 vars.sh
./vars.sh
# ./vars.sh: Permission denied  (exit status 126)
```

Add execute permission, then check again:

```bash
chmod +x vars.sh
ls -l vars.sh
# -rwxr-xr-x 1 student student 369 Oct 21 10:38 vars.sh
./vars.sh
```

| Command | Effect |
|---|---|
| `chmod +x file` | Add execute for owner, group, and others (except what your `umask` blocks). |
| `chmod u+x file` | Add execute for the owner (`u` = user) only. |
| `chmod -x file` | Remove execute permission again. |

In `-rwxr-xr-x`, each `x` is one execute permission: owner `rwx`, group `r-x`, others `r-x`.

[Back to top](#table-of-contents)

---

## 6. Script 1: Count Directory Entries (`1.sh`)

**Core idea:** a script is a file of commands. This one holds a single pipeline that counts the entries in the current directory.

```bash
#!/bin/bash

ls | wc -l
```

Output in a directory that contains `1.sh`, `a.txt`, `b.txt`, `my notes.txt`, the directory `photos`, and the hidden file `.hidden`:

```text
5
```

The count includes `1.sh` itself and the directory `photos`, but not `.hidden`.

### Anatomy

| Part | What it does |
|---|---|
| `#!/bin/bash` | Line 1, the shebang: run this file with `/bin/bash`. |
| (blank line) | Ignored. It only separates the header from the commands. |
| `ls` | List the names in the current directory. Hidden names (starting with `.`) are skipped. |
| `\|` | Pipe: connect the output of `ls` to the input of `wc`. Both programs run at the same time. |
| `wc -l` | Count the lines it receives, and print that number. |

[Back to top](#table-of-contents)

---

## 7. Script 2: Two Kinds of `for` Loop (`2.sh`)

**Core idea:** a `for` loop repeats commands. The first loop goes through a list of words; the second counts with a number, using syntax borrowed from C.

```bash
#!/bin/bash

for d in /public/prednasky /public/ucebnove /public/priklady
do
	echo -n $d
	ls $d | wc -l
done

#podobne ako v C
#     init  podm  vyraz
for (( i=0 ; i<5 ; i++ ))
do
    echo "opakovanie $i"
done
```

Output (the three counts depend on what the directories contain):

```text
/public/prednasky3
/public/ucebnove2
/public/priklady1
opakovanie 0
opakovanie 1
opakovanie 2
opakovanie 3
opakovanie 4
```

### Anatomy: Loop 1, a List of Words

| Part | What it does |
|---|---|
| `for d in A B C` | Run the body once for each word after `in`. Each time, `d` holds the next word. |
| `do ... done` | Mark the start and end of the loop body. |
| `echo -n $d` | Print the directory name. `-n` means **no newline** at the end. |
| `ls $d \| wc -l` | Count the entries in that directory, as in Script 1. |

Because `-n` removes the newline, each count is printed right after its name, on the same line, for example `/public/prednasky3`. Write `echo -n "$d "` to put a space between them.

### Anatomy: Loop 2, a C-Style Counter

This loop copies the shape of C's `for (i = 0; i < 5; i++)`. `(( ... ))` is Bash's arithmetic syntax, so it is still Bash, not C: there are no types, and variables inside `(( ))` need no `$`. Only Bash supports this form: `sh` rejects it with `Syntax error: Bad for loop variable`.

| Part | What it does |
|---|---|
| `i=0` | **init**: runs once, before the first pass. |
| `i<5` | **condition**: checked before every pass. The loop stops when it is false. |
| `i++` | **update**: adds 1 to `i` after every pass. |
| `echo "opakovanie $i"` | The body: print the current value of `i`. |

[Back to top](#table-of-contents)

---

## 8. Script 3: Quoting and the `while` Loop (`3.sh`)

**Core idea:** quoting decides what a command receives. The same two variables print in five different ways, inside a `while` loop that counts down.

```bash
#!/bin/bash

#veta=ahoj svet
veta="ahoj svet"
pocet=3

while [ $pocet -gt 0 ]; do
	echo $pocet $veta
	echo $pocet	$veta
	echo "$pocet	$veta"
	echo '$pocet	$veta'
	echo \$pocet	\$veta
	echo ""
	((pocet--))
done
```

Output of the first pass:

```text
3 ahoj svet
3 ahoj svet
3	ahoj svet
$pocet	$veta
$pocet $veta

```

The same block repeats with `2` and then `1`.

### Anatomy: Variables

| Part | What it does |
|---|---|
| `#veta=ahoj svet` | Commented out because it is wrong: without quotes, Bash runs `svet` as a command (`svet: command not found`) and `veta` stays empty. See [section 4](#4-defining-variables). |
| `veta="ahoj svet"` | Store text that contains a space. The quotes make it one value. |
| `pocet=3` | Store `3`. It is used as a number in the test and in `(( ))`. |

### Anatomy: The `while` Loop

| Part | What it does |
|---|---|
| `while CONDITION; do ... done` | Repeat the body as long as the condition succeeds. The condition is checked before every pass. |
| `[ $pocet -gt 0 ]` | The test command `[`: true while `pocet` is greater than `0`. The spaces after `[` and before `]` are required. |
| `((pocet--))` | Arithmetic: subtract 1 from `pocet`. Without it, the condition never becomes false and the loop never ends. |

The body runs three times, with `pocet` equal to `3`, `2`, and `1`. Then `pocet` is `0`, `0 -gt 0` is false, and the loop ends.

Numbers are compared with letter operators inside `[ ]`:

| Operator | Meaning |
|---|---|
| `-eq` | equal |
| `-ne` | not equal |
| `-gt` | greater than |
| `-ge` | greater than or equal |
| `-lt` | less than |
| `-le` | less than or equal |

Do not write `[ $pocet > 0 ]`: inside `[ ]`, `>` is still a redirection, so this creates a file named `0`.

### Anatomy: The Five `echo` Lines

The 2nd to 5th `echo` lines separate the two variables with a **tab**, shown here as `→`. `echo` prints its arguments with one space between them, so any unquoted spaces or tabs collapse into one space.

| Line | What `echo` receives | Printed |
|---|---|---|
| `echo $pocet $veta` | 3 words: `3`, `ahoj`, `svet` | `3 ahoj svet` |
| `echo $pocet→$veta` | The same 3 words. The unquoted tab only separates them. | `3 ahoj svet` |
| `echo "$pocet→$veta"` | 1 argument: variables expanded, tab kept | `3→ahoj svet` |
| `echo '$pocet→$veta'` | 1 argument: nothing expanded | `$pocet→$veta` |
| `echo \$pocet→\$veta` | 2 words: `\` stops each `$`, the unquoted tab separates them | `$pocet $veta` |
| `echo ""` | 1 empty argument | an empty line |

| Quoting | Is `$name` replaced by its value? | Are spaces and tabs kept? |
|---|---|---|
| None | Yes | No: the text is split into words. |
| `"double"` | Yes | Yes |
| `'single'` | No | Yes |
| `\` before a character | No, for that one character only | It protects that one character only. |

[Back to top](#table-of-contents)

---

## 9. Script 4: Rename Files to Lowercase (`4.sh`)

**Core idea:** a pattern picks the files, a pipeline builds each new name, and `$( )` puts that name into the `mv` command.

```bash
#!/bin/bash

for f in [A-Z]*; do
	echo "$f" | tr 'A-Z' 'a-z'
	mv -i "$f" "$(echo "$f" | tr A-Z a-z)"
	#mv -i "$f" "`echo "$f" | tr A-Z a-z`"
done
```

**This script renames real files.** Run it only in an empty practice directory.

Output in a directory that contains `4.sh`, the directory `Docs`, and the files `My File.txt`, `Notes.TXT`, `README`, `lower.txt`, and `photo.JPG`:

```text
docs
my file.txt
notes.txt
readme
```

`ls` afterwards shows `4.sh`, `docs`, `lower.txt`, `my file.txt`, `notes.txt`, `photo.JPG`, and `readme`. `photo.JPG` keeps its name: only the first character is checked, and `p` is already lowercase.

### Anatomy

| Part | What it does |
|---|---|
| `for f in [A-Z]*; do ... done` | Run the body once for each name that matches the pattern. Each time, `f` holds the current name. |
| `[A-Z]*` | Pattern: names whose **first** character is an uppercase letter, followed by anything (`*`). Bash replaces it with the matching names before the loop starts. Directories match too. |
| `echo "$f" \| tr 'A-Z' 'a-z'` | Print the new, lowercase name, so you can see what is about to be renamed. |
| `tr 'A-Z' 'a-z'` | Translate characters: each uppercase letter becomes its lowercase partner. `tr` works on single characters, not words. |
| `mv -i OLD NEW` | Rename `OLD` to `NEW`. `-i` asks before replacing a file that already has the name `NEW`. |
| `"$(echo "$f" \| tr A-Z a-z)"` | Command substitution: run the pipeline and use its output as `NEW`. |
| `"$f"` | The quotes keep a name with spaces, such as `My File.txt`, as one argument. |
| `` #mv -i "$f" "`...`" `` | The same line written with backticks, the older form of `$( )`. It is commented out: prefer `$( )`, which is easier to read and nest. |

### Useful `tr` and `mv` Options

```bash
tr 'a-z' 'A-Z'    # Lowercase to uppercase.
tr -d '0-9'       # Delete every digit.
tr -s ' '         # Squeeze runs of spaces into one space.
mv -n old new     # Rename, but never overwrite an existing file named new.
mv -v old new     # Rename and print what was done.
```

[Back to top](#table-of-contents)

---

## 10. Script 5: Arguments, `case`, and `shift` (`5.sh`)

**Core idea:** a script reads its arguments one at a time. `case` decides what the current argument `$1` is, and `shift` removes it, so the next argument becomes `$1`. This is one common way for a script to handle options such as `-h`.

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
            echo "Neznamy prepinac \"$1\""
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

Output for different arguments:

```text
$ ./5.sh
Help: ./5.sh arg1 arg2 arg3 ... argN

$ ./5.sh -h
Argumenty: -h
Help: ./5.sh arg1 arg2 arg3 ... argN

$ ./5.sh -x
Argumenty: -x
Neznamy prepinac "-x"

$ ./5.sh abc cba file1
Argumenty: abc cba file1
pokracujem dalej
pokracujem dalej

$ ./5.sh -d file1 file2
Argumenty: -d file1 file2
Dalsie argumenty: file1 file2

$ ./5.sh abc -D x y
Argumenty: abc -D x y
pokracujem dalej
Dalsie argumenty: x y

$ ./5.sh file1 -d x
Argumenty: file1 -d x
```

### Anatomy

| Part | What it does |
|---|---|
| `help="Help: $0 ..."` | A usage message. `$0` is the name the script was run with, so the message always shows the right name. |
| `if [ "$#" == "0" ]; then ... fi` | No arguments: print the help and stop. `==` compares **text** here; the usual test for numbers is `[ "$#" -eq 0 ]`. |
| `exit 1` / `exit 0` | Stop the script immediately with this exit status: `0` means success, anything else means failure. |
| `unset debug` | Delete the variable `debug`, so it starts as not set. |
| `echo "Argumenty: $@"` | Print all arguments. |
| `#echo $1 ; shift ; echo $1` | A commented-out demo of `shift`: it would print the first argument, then the second. |
| `while (( "$#" ))` | Loop while arguments remain. `(( ))` is true when the number is not `0`. |
| `case "$1" in ... esac` | Compare `$1` with each pattern, top to bottom. The first match runs; `;;` ends that branch. |
| `shift` (before `done`) | Remove `$1`. The next argument becomes `$1`, and `$#` goes down by 1. |
| `[[ -v debug ]]` | True if `debug` is set, even when its value is empty. |

### The `case` Patterns

| Pattern | Matches | What happens |
|---|---|---|
| `-d\|-D)` | `-d` or `-D`. `\|` means OR. | Set `debug`, `shift` the option away, and `break`: leave the loop. The arguments after it are kept. |
| `abc\|cba)` | Exactly `abc` or `cba`. | Print a message, then continue with the next argument. |
| `-h)` | `-h` | Print the help and exit with `0`. |
| `-*)` | Any other word that starts with `-`. | Unknown option: print an error and exit with `1`. |
| `*)` | Anything else. | The first ordinary argument: `break`, and stop reading options. |

The patterns are globs, like `[A-Z]*` in Script 4, not regular expressions. Their **order matters**: `-*` also matches `-d` and `-h`, so the specific patterns must come first.

The `-d|-D` branch calls `shift` itself because `break` leaves the loop before the `shift` at the end of the loop body.

### How `shift` Moves the Arguments

For `./5.sh -d file1 file2`:

```text
before shift:   $1=-d     $2=file1   $3=file2   $#=3
after shift:    $1=file1  $2=file2              $#=2
```

`-d` is gone, so `$@` is now `file1 file2`, which is what `Dalsie argumenty:` prints.

### A Flag Without a Value

`debug=''` sets the variable to an empty value. That is enough: the script only checks whether `debug` exists, not what it contains.

| State | `[[ -v debug ]]` | `[[ -n $debug ]]` |
|---|---|---|
| After `unset debug` | false | false |
| After `debug=''` | true | false: the value is empty |

### Two Runs to Read Closely

- `./5.sh abc -D x y`: `abc` prints a message, then `-D` works like `-d`. The loop ends, and the last line prints the rest: `x y`.
- `./5.sh file1 -d x`: `file1` matches `*` first, so the loop ends at once. `-d` is never read, `debug` stays unset, and the last line prints nothing.

[Back to top](#table-of-contents)

---

## 11. Script 6: Arrays (`6.sh`)

**Core idea:** an array stores a list of values in one variable. Each value, called an element, can contain spaces, as long as you expand the array as `"${name[@]}"`.

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

A space after `#!` is allowed: `#! /bin/bash` works the same as `#!/bin/bash`.

Output:

```text
jeden
tri
3
15
6
tri styri pat sest cele sedem

9
jeden dva tri styri pat sest cele sedem osem
7
jeden dva tri styri pat sest cele sedem osem
jeden
dva
tri
styri
pat
sest cele sedem

```

### Anatomy: Array Syntax

The array has six elements, numbered from 0:

| Index | 0 | 1 | 2 | 3 | 4 | 5 |
|---|---|---|---|---|---|---|
| Element | `jeden` | `dva` | `tri` | `styri` | `pat` | `sest cele sedem` |

| Syntax | What it gives |
|---|---|
| `name=(a b "c d")` | Create an array. Spaces separate elements; quotes keep `"c d"` as one element. |
| `${name[i]}` | The element at index `i`. The braces are required: `$zoznam[2]` prints `jeden[2]`. |
| `$name` | Without an index: element `0`. |
| `${#name[i]}` | The length in characters of element `i`. |
| `${name[@]}` | All elements. |
| `${#name[@]}` | The number of elements. |
| `${name[@]:start:count}` | A slice: `count` elements, starting at index `start`. |
| `"${name[@]}"` | All elements, each kept as **exactly one** word. Without the quotes, elements that contain spaces are split. |

### Line by Line

| Line | Output | Why |
|---|---|---|
| `echo $zoznam` | `jeden` | No index means element `0`. |
| `echo ${zoznam[2]}` | `tri` | Index `2` is the third element, because counting starts at `0`. |
| `echo ${#zoznam[2]}` | `3` | `tri` has 3 characters. |
| `echo ${#zoznam[5]}` | `15` | `sest cele sedem` has 15 characters, including the two spaces. |
| `echo ${#zoznam[@]}` | `6` | The array has 6 elements. |
| `echo ${zoznam[@]:2:4}` | `tri styri pat sest cele sedem` | Start at index 2 and take 4 elements: `tri`, `styri`, `pat`, `sest cele sedem`. |
| `zoznam1=(...)` | nothing | Builds a new array, but the script never prints it. See below. |
| `zoznam2=(${zoznam[@]} osem)`, then `echo ${#zoznam2[@]}` | `9` | **Unquoted**: `sest cele sedem` is split into 3 elements, so 5 + 3 + 1 (`osem`) = 9. |
| `zoznam2=("${zoznam[@]}" osem)`, then `echo ${#zoznam2[@]}` | `7` | **Quoted**: the 6 elements stay intact, plus `osem` = 7. |
| `echo ${zoznam2[@]}` (both times) | the same text | `echo` joins everything with spaces, so it hides the difference. Only the counts `9` and `7` reveal it. |
| `for prvok in "${zoznam[@]}"` | 6 lines | One pass per element. `sest cele sedem` stays on one line. |

### The `zoznam1` Line

```bash
zoznam1=(${zoznam[@]:0:6} sedem ${zoznam[$((7-${#zoznam[@]}))]})
```

The outer `( )` creates a new array from the words inside it. There are three parts:

```text
zoznam1=(  ${zoznam[@]:0:6}   sedem   ${zoznam[$((7-${#zoznam[@]}))]}  )
           \____ part 1 ___/  part 2  \____________ part 3 ___________/
```

#### Part 1: `${zoznam[@]:0:6}`

A slice: start at index `0` and take `6` elements. The array has exactly 6 elements, so this is all of them.

It is **not quoted**, so Bash splits every element at spaces. `sest cele sedem` becomes three separate words:

```text
jeden dva tri styri pat sest cele sedem      # 8 words
```

#### Part 2: `sedem`

Plain text: one more word, `sedem`.

#### Part 3: `${zoznam[$((7-${#zoznam[@]}))]}`

This is three constructs nested inside each other. Spaces are allowed inside it, which makes the layers easier to see:

```bash
${zoznam[ $(( 7 - ${#zoznam[@]} )) ]}
```

`$((` opens one construct and `))` closes it. The braces inside, `${#zoznam[@]}`, are a different construct.

| Brackets | Name | What it does | Example |
|---|---|---|---|
| `${ }` | Parameter expansion | Insert a variable's value. | `${#zoznam[@]}` → `6` |
| `(( ))` | Arithmetic command | Calculate, but insert **nothing**. Used for `i++` and `pocet--`. | `((pocet--))` |
| `$(( ))` | Arithmetic expansion | Calculate, and insert the **result** into the line. | `$((7-6))` → `1` |
| `$( )` | Command substitution | Run a command and insert its output (Script 4). | `$(echo "$f" \| tr A-Z a-z)` |

The `$` in front means "put the result here", just as `$name` puts a variable's value here.

Bash works from the **inside out**:

```text
${zoznam[$((7-${#zoznam[@]}))]}
${zoznam[$((7-6))]}             # ${#zoznam[@]} is the number of elements: 6
${zoznam[1]}                    # $((7-6)) calculates 7 - 6 = 1
dva                             # element 1 is dva
```

The index inside `[ ]` is already calculated as arithmetic, so `${zoznam[7-${#zoznam[@]}]}` gives the same result without `$(( ))`.

#### The Result

Bash puts the three parts together, 8 + 1 + 1 words, and each word becomes one element:

| Index | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 |
|---|---|---|---|---|---|---|---|---|---|---|
| Element | `jeden` | `dva` | `tri` | `styri` | `pat` | `sest` | `cele` | `sedem` | `sedem` | `dva` |
| From | part 1 | part 1 | part 1 | part 1 | part 1 | part 1 | part 1 | part 1 | part 2 | part 3 |

`zoznam1` has **10** elements. With quotes, `zoznam1=("${zoznam[@]:0:6}" sedem "${zoznam[$((7-${#zoznam[@]}))]}")` keeps `sest cele sedem` as one element and gives 8.

The commented line `#zoznam=(${zoznam[@]} osem)` would replace `zoznam` itself. To add an element to the end of an existing array, write `zoznam+=(osem)`.

[Back to top](#table-of-contents)

---

## 12. Script 7: Split a Path with `dirname` and `basename` (`7.sh`)

**Core idea:** `dirname` and `basename` cut a path into its directory part and its last name. Nesting them with `$( )` walks up the path. A path with spaces needs quotes at **every** level.

```bash
#! /bin/bash

#cesta="/public/ucebnove/seminar _1/vim.txt"
cesta=/public/ucebnove/seminar_1/vim.txt

echo $(dirname $cesta)
echo $(basename $cesta)
echo $(dirname $(dirname $cesta))
echo $(basename $(dirname $cesta))
```

Output:

```text
/public/ucebnove/seminar_1
vim.txt
/public/ucebnove
seminar_1
```

### Anatomy

```text
/public/ucebnove/seminar_1/vim.txt
\_______ dirname ________/ \_____/
                           basename
```

| Part | What it does |
|---|---|
| `dirname PATH` | Print the path without its last component: everything before the last `/`. |
| `basename PATH` | Print only the last component: everything after the last `/`. |
| `$( )` | Command substitution: run the command and insert its output (Script 4). |
| `$(dirname $(dirname $cesta))` | Nested: the inner `$( )` runs first, and its output becomes the input of the outer one. |

Both commands only cut text. The path does not need to exist.

### Line by Line

| Line | Output | Why |
|---|---|---|
| `echo $(dirname $cesta)` | `/public/ucebnove/seminar_1` | Remove the last component, `vim.txt`. |
| `echo $(basename $cesta)` | `vim.txt` | Keep only the last component. |
| `echo $(dirname $(dirname $cesta))` | `/public/ucebnove` | The inner `dirname` gives `/public/ucebnove/seminar_1`. The outer one removes `seminar_1`. |
| `echo $(basename $(dirname $cesta))` | `seminar_1` | The inner `dirname` gives `/public/ucebnove/seminar_1`. `basename` keeps its last component: the name of the directory that contains the file. |

### A Path with a Space

The commented-out line has a space in the directory name: `seminar _1`. Switch the two `cesta=` lines and run the script unchanged:

```text
/public/ucebnove _1
seminar
/public .
ucebnove
```

Every line is wrong. Without quotes, `$cesta` splits into two arguments: `/public/ucebnove/seminar` and `_1/vim.txt`.

| Line | Output | What went wrong |
|---|---|---|
| `echo $(dirname $cesta)` | `/public/ucebnove _1` | `dirname` prints one line per argument: `/public/ucebnove` and `_1`. The unquoted `$( )` splits them again, and `echo` prints them on one line with a space between them. |
| `echo $(basename $cesta)` | `seminar` | `basename` treats a second argument as a **suffix to remove**, not as another path. |
| `echo $(dirname $(dirname $cesta))` | `/public .` | The inner output, `/public/ucebnove` and `_1`, splits again. `dirname _1` gives `.`, meaning "the current directory". |
| `echo $(basename $(dirname $cesta))` | `ucebnove` | `basename /public/ucebnove _1`: the `_1` is again taken as a suffix. |

Quote the variable and every `$( )`:

```bash
cesta="/public/ucebnove/seminar _1/vim.txt"

echo "$(dirname "$cesta")"
echo "$(basename "$cesta")"
echo "$(dirname "$(dirname "$cesta")")"
echo "$(basename "$(dirname "$cesta")")"
```

Output:

```text
/public/ucebnove/seminar _1
vim.txt
/public/ucebnove
seminar _1
```

Quotes inside `$( )` do not clash with the quotes outside it: each `$( )` starts a new quoting level. Quoting only the outside is not enough: `"$(dirname $cesta)"` still splits `$cesta`.

### Useful Forms

```bash
dirname vim.txt               # .   (no directory part: the current directory)
basename /public/ucebnove/    # ucebnove   (a trailing / is ignored)
basename vim.txt .txt         # vim   (remove the suffix .txt)
```

[Back to top](#table-of-contents)

---

## 13. Script 8: Word Splitting and `IFS` (`8.sh`)

**Core idea:** Bash splits the output of an unquoted `$( )` into separate words, using the characters in `IFS` (Internal Field Separator). By default `IFS` holds a space, a tab, **and a newline**, so a `for` loop gets **words**. Set `IFS` to a newline only, and the loop gets whole **lines**.

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

Output in a directory that contains `8.sh`, `a.txt`, and `b.txt`:

```text
total 12
-rwxr-xr-x 1 student student 141 Oct 21 10:00 8.sh
-rw-r--r-- 1 student student   6 Oct 21 10:00 a.txt
-rw-r--r-- 1 student student   3 Oct 21 10:00 b.txt

total
12
-rwxr-xr-x
1
student
student
141
Oct
21
10:00
8.sh
-rw-r--r--
1
student
student
6
Oct
21
10:00
a.txt

total 12
-rwxr-xr-x 1 student student 141 Oct 21 10:00 8.sh
-rw-r--r-- 1 student student   6 Oct 21 10:00 a.txt
```

### Anatomy

| Part | What it does |
|---|---|
| `ls -l` | Long listing, one entry per line. The first line, `total 12`, is the disk space used by the entries, not a file. |
| `ls -l \| head -3` | Keep only the first 3 lines: the `total` line and the first two entries. |
| `for f in $(ls -l \| head -3)` | Run the command, split its output by `IFS`, and run the body once for each piece. |
| `echo $f` | Print the current piece. |
| `IFS='` + newline + `'` | The quotes contain **only a newline**, so `IFS` becomes a newline. `IFS=$'\n'` is a more readable way to write the same thing. |

### Loop 1: Default `IFS`

The output of `ls -l | head -3` is split at every space and newline. In this example the loop runs 20 times, once per word, so `total` and `12` arrive as separate items and the structure of each line is lost.

### Loop 2: `IFS` Is a Newline

The same output is now split only at newlines. The loop runs 3 times, once per line. Spaces are no longer separators, so `echo $f` receives the whole line as one argument and even the extra spaces before `6` are kept.

The new `IFS` stays in effect until the script ends, so every later unquoted expansion is also split only at newlines.

### Reading Lines Without Changing `IFS`

A `while read` loop reads one line per pass and leaves `IFS` unchanged for the rest of the script:

```bash
ls -l | head -3 | while IFS= read -r line; do
	echo "$line"
done
```

`IFS=` here applies only to `read`. `-r` keeps backslashes as they are.

[Back to top](#table-of-contents)

---

## 14. Script 9: `source`, `.`, and `exec` (`9.sh`)

**Core idea:** the script runs itself two more times, in two different ways. `source` runs it **inside** the current shell, so its variables stay. `exec` **replaces** the current shell with a new run, so nothing after `exec` runs.

```bash
#!/bin/bash

run=0

if [ $# -ne 1 ]; then
	echo "start"
	source $0 prvykrat
	#. $0 prvykrat
	echo "$var1"
	echo ""

	exec $0 druhykrat
	echo "$var2"
elif [ $1 == prvykrat ]; then
	var1=prvykrat
	echo "$var1 zo source"
elif [ $1 == druhykrat ]; then
	var2=druhykrat
	echo "$var2 z exec"
fi
```

Run it as `./9.sh`, after `chmod +x 9.sh`:

```text
start
prvykrat zo source
prvykrat

druhykrat z exec
```

With `bash 9.sh`, `$0` is `9.sh` without `./`. `exec` then searches `PATH` for it and fails with `exec: 9.sh: not found`.

### `source`, `.`, and `exec` Compared

- **`source file`** runs the lines of the file **in your current shell**, as if you typed them yourself. No new process starts. Variables it sets, and a `cd` it does, are still there afterwards.
- **`. file`** is exactly the same as `source file`. `.` is the original short name that every POSIX shell knows; `source` is the easier-to-read name that Bash adds.
- **`exec command`** **replaces** the current shell with the command. The process ID stays the same, but the old script is gone: the lines after `exec` never run.
- For comparison, **`./file`** runs the file in a **new** process. When it ends, your script continues, but the variables it set are gone.

| | `./child.sh` | `source child.sh` or `. child.sh` | `exec ./child.sh` |
|---|---|---|---|
| Runs in | A new process | The current shell | The current process, which it takes over |
| Variables it sets stay for the caller | No | Yes | There is no caller left |
| The caller continues afterwards | Yes | Yes | No |

### See It with `$$`

`$$` prints the process ID. `child.sh` prints its PID and sets a variable `x`:

```bash
#!/bin/bash
echo "  child.sh PID: $$"
x=set_in_child
```

`pid.sh` runs it in all three ways:

```bash
#!/bin/bash
echo "pid.sh PID: $$"
./child.sh;      echo "after ./child.sh:  x=[$x]"
source child.sh; echo "after source:      x=[$x]"
unset x
. ./child.sh;    echo "after . :          x=[$x]"
exec ./child.sh
echo "never printed"
```

Output:

```text
pid.sh PID: 19
  child.sh PID: 20
after ./child.sh:  x=[]
  child.sh PID: 19
after source:      x=[set_in_child]
  child.sh PID: 19
after . :          x=[set_in_child]
  child.sh PID: 19
```

- `./child.sh` has a different PID, and `x` is empty afterwards.
- `source` and `.` keep the same PID, and `x` stays set.
- `exec` keeps the same PID, and `never printed` is never printed.

### Step by Step Through `9.sh`

| Step | Line | What happens |
|---|---|---|
| 1 | `if [ $# -ne 1 ]` | `./9.sh` has no arguments, so `$#` is `0`: the first branch runs and prints `start`. |
| 2 | `source $0 prvykrat` | Run `9.sh` again in this shell, with `$1` set to `prvykrat`. That run takes the second branch: it sets `var1` and prints `prvykrat zo source`. |
| 3 | `echo "$var1"` | Prints `prvykrat`: `var1` was set in this shell, so it still exists. |
| 4 | `echo ""` | An empty line. |
| 5 | `exec $0 druhykrat` | Replace this shell with a new run of `9.sh`, with `$1` set to `druhykrat`. That run takes the third branch and prints `druhykrat z exec`. |
| 6 | `echo "$var2"` | **Never runs**: the script containing this line was replaced in step 5. |

The commented line `#. $0 prvykrat` does the same as step 2, written with `.`. The variable `run` is never used.

The check `[ $# -ne 1 ]` stops the script from calling itself forever: both inner runs get exactly one argument, so they skip the first branch.

[Back to top](#table-of-contents)

---

## 15. Sources

- [GNU Bash: Shell Scripts](https://www.gnu.org/software/bash/manual/html_node/Shell-Scripts.html): how Bash runs a script file and the `#!` line.
- [GNU Bash: Special Parameters](https://www.gnu.org/software/bash/manual/html_node/Special-Parameters.html): `$0`, `$#`, `$@`, `$?`, and `$$`.
- [GNU Bash: Shell Parameters](https://www.gnu.org/software/bash/manual/html_node/Shell-Parameters.html): variable assignment and positional parameters.
- [GNU Bash: Looping Constructs](https://www.gnu.org/software/bash/manual/html_node/Looping-Constructs.html): `for` loops over a list and the C-style `for (( ))` form.
- [GNU Bash: Quoting](https://www.gnu.org/software/bash/manual/html_node/Quoting.html): no quotes, double quotes, single quotes, and the backslash.
- [GNU Bash: Conditional Expressions](https://www.gnu.org/software/bash/manual/html_node/Bash-Conditional-Expressions.html): numeric tests such as `-gt` and `-eq`.
- [GNU Bash: Filename Expansion](https://www.gnu.org/software/bash/manual/html_node/Filename-Expansion.html): patterns such as `[A-Z]*`.
- [GNU Bash: Command Substitution](https://www.gnu.org/software/bash/manual/html_node/Command-Substitution.html): `$( )` and backticks.
- [GNU Bash: Arrays](https://www.gnu.org/software/bash/manual/html_node/Arrays.html): creating arrays, indexes, `${name[@]}`, and `${#name[@]}`.
- [GNU Bash: Shell Parameter Expansion](https://www.gnu.org/software/bash/manual/html_node/Shell-Parameter-Expansion.html): lengths with `${#...}` and slices with `${name[@]:start:count}`.
- [GNU Bash: Arithmetic Expansion](https://www.gnu.org/software/bash/manual/html_node/Arithmetic-Expansion.html): `$(( ))`.
- [GNU Bash: Word Splitting](https://www.gnu.org/software/bash/manual/html_node/Word-Splitting.html): how `IFS` splits unquoted expansions into words.
- [GNU Bash: Conditional Constructs](https://www.gnu.org/software/bash/manual/html_node/Conditional-Constructs.html): `case` and `[[ ]]`.
- [GNU Bash: Bourne Shell Builtins](https://www.gnu.org/software/bash/manual/html_node/Bourne-Shell-Builtins.html): `.`, `exec`, `exit`, `shift`, and `unset`.
- [GNU Bash: Bash Builtins](https://www.gnu.org/software/bash/manual/html_node/Bash-Builtins.html): `source`.
- [GNU Bash: Pipelines](https://www.gnu.org/software/bash/manual/html_node/Pipelines.html): connecting commands with `|`.
- [execve(2)](https://man7.org/linux/man-pages/man2/execve.2.html): how the kernel runs interpreter scripts that start with `#!`.
- [env(1)](https://man7.org/linux/man-pages/man1/env.1.html): running a program found through `PATH`.
- [chmod(1)](https://man7.org/linux/man-pages/man1/chmod.1.html): adding and removing permissions.
- [tr(1)](https://man7.org/linux/man-pages/man1/tr.1.html): translating, deleting, and squeezing characters.
- [mv(1)](https://man7.org/linux/man-pages/man1/mv.1.html): renaming, with `-i`, `-n`, and `-v`.
- [dirname(1)](https://man7.org/linux/man-pages/man1/dirname.1.html): removing the last component of a path.
- [basename(1)](https://man7.org/linux/man-pages/man1/basename.1.html): keeping the last component, and removing a suffix.
- [head(1)](https://man7.org/linux/man-pages/man1/head.1.html): printing the first lines of input.
- [GNU Coreutils: wc](https://www.gnu.org/software/coreutils/manual/html_node/wc-invocation.html): counting lines, words, and bytes.

[Back to top](#table-of-contents)
