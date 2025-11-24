# Sed – Stream Editing Basics

## What Sed Is
Sed is a stream editor. It reads text line by line, executes commands, and writes modified output. It processes input non‑interactively and is suited for large text transformations.

## How Sed Works
1. Read one line.
2. Apply script commands to that line.
3. Print the result.
4. Continue until end of file.

## When to Use Sed
- Bulk text modification.
- Automated transformations.
- Line‑based editing.
- Pattern‑driven changes.

## Syntax
```
sed OPTIONS... [SCRIPT] [INPUTFILE...]
```

## Options
- `-n` suppress automatic printing.
- `-i` edit file in place.
- `-f file.sed` read script from file.
- `-e 'SCRIPT'` specify script on command line.

## Script Structure
A script combines an address selector with a command.

### Addresses
- `1` first line.
- `3` third line.
- `4,8` lines 4 through 8.
- `5~10` line 5 and every tenth after.
- `$` last line.

### Commands
- `p` print.
- `=` print line number.
- `n` read next line.
- `i` insert before line.
- `a` append after line.
- `c` replace entire line.
- `d` delete.
- `s/pattern/replacement/[flags]` substitute text.

## Examples with Explanations

### Missing script
```
sed sedinput.txt
```
Sed requires a script; without one it errors.

### Empty script
```
sed '' sedinput.txt
```
No commands; sed prints each line unchanged.

### Suppress printing
```
sed -n '' sedinput.txt
```
`-n` disables output; without commands nothing prints.

### Explicit print
```
sed -n 'p' sedinput.txt
```
`p` prints each line; `-n` prevents duplicates.

### Print line numbers
```
sed -n '=' sedinput.txt
```
Prints only line numbers.

### Line numbers + content
```
sed -n '=;p' sedinput.txt
```
Outputs number then content for each line.

### Insert before each line
```
sed 'i vkladam pred riadok' sedinput.txt
```
Adds the text before every line.

### Append after each line
```
sed 'a vkladam za riadok' sedinput.txt
```
Adds text after each line.

### Append after line 3
```
sed '3a vkladam za treti riadok' sedinput.txt
```
Targets only line 3.

### Replace line 3
```
sed '3c vkladam namiesto tretieho riadka' sedinput.txt
```
Line 3 is replaced entirely.

### Delete line 3
```
sed '3d' sedinput.txt
```
Line 3 removed from output.

### Substitute first occurrence
```
sed 's/hello/world/' sedinput.txt
```
Replaces first match per line.

### Create script file
```
echo 's/hello/world/g' > mojscript.sed
```

### Run script from file
```
sed -f mojscript.sed sedinput.txt
```

### Global substitution
```
sed 's/hello/world/g' sedinput.txt
```
Replaces all matches per line.


# Sed Practical Tasks – list.txt Editing

Before starting, copy the source file to your home directory and work only with the copy:

```bash
cp /public/samples/list.txt ~/list.txt
cp ~/list.txt ~/list.bkp
```

Always keep a backup before applying any sed command that modifies content.

![alt text](image.png)

This is a letter that Jozko Mrkvicka once wrote to his girlfriend Zuzka Blazkova. But then Jozko and Zuzka broke up, and now this absolute genius decided to have a new girlfriend and figured the best way to handle it was to just recycle the same love letter. Because apparently, why write something new when you can just do a find-and-replace on the old one? Help him (yes, despite the obvious moral red flags here) rewriting the letter according to the following tasks:

---

## Task 1 – Delete lines 7 through 10
```bash
sed '7,10d' list.txt
```
Deletes lines 7–10. All other lines are printed.

---

## Task 2 – Replace “Zuzka” with “Lucka” only on line 3
```bash
sed '3s/Zuzka/Lucka/g' list.txt
```
Address 3 limits substitution to line 3. `g` replaces every occurrence on that line only.

---

## Task 3 – Replace only the second occurrence of “Zuzka” on line 3
```bash
sed '3s/Zuzka/Lucka/2' list.txt
```
`/2` replaces only the second match of the pattern on line 3.

---

## Task 4 – Replace “Zuzka” with uppercase “ZUZKA”
```bash
sed 's/Zuzka/\U&/g' list.txt
```
`&` inserts the matched text. `\U` transforms it to uppercase.

---

## Task 5 – Replace “Zuzka” with “zUZKA” (mixed case)
```bash
sed 's/\(Z\)\(uzka\)/\L\1\U\2/g' list.txt
```
`\L` lowercases group 1. `\U` uppercases group 2.

---

## Task 6 – Replace whole-word “mozno” with “nemozno”
```bash
sed 's/\<[Mm]ozno\>/nemozno/g' list.txt
```
Word boundaries ensure only full words match. Prevents altering “Nemozno”.

---

## Task 7 – Print only the poem (remove odd-numbered lines)
```bash
sed -n 'n;7,16p' list.txt
```
`n` skips one line (odd), `p` prints lines 7–16 (even).

---

## Task 8 – Print poem but remove every even-numbered line
```bash
sed -n '7,16p;n' list.txt
```
`p` prints targeted range; `n` skips the next line to remove even lines.

---

## Task 9 – Replace “Zuzka” with “Lucka” in backup file
```bash
sed -i 's/Zuzka/Lucka/g' list.bkp
```
In‑place editing (`-i`). Works on backup file only.

---

## Task 10 – Print poem with line numbers
```bash
sed -n '=;7,16p' list.txt   | sed "N;s/\n/ /g;"   | sed -n '4,12p'
```
1. Prints numbers and lines.  
2. Joins number + text into one line.  
3. Extracts the formatted poem region.


