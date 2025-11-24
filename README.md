# AWK – Introduction to Text Processing

## What Is AWK

AWK is a programming language for processing structured text.  
Used for logs, CSV, reports, and other column‑oriented data.  
Name comes from: **Aho**, **Weinberger**, **Kernighan**.

## What AWK Does

- Process files line by line  
- Filter and extract data  
- Transform text  
- Generate reports  
- Perform calculations  
- Pattern matching  

## How AWK Works

1. Reads input record by record (default: each line)  
2. Splits each line into fields (`$1`, `$2`, …)  
3. Applies selection pattern  
4. Executes action when pattern matches  
5. Continues until end of file  

General syntax:
```
awk 'pattern { action }' input-file > output-file
```

AWK uses:
- **records** → lines  
- **fields** → whitespace-separated values  

## Basic Printing Examples

Print entire line:
```
awk '{ print $0 }' file.txt
```

Print first field:
```
awk '{ print $1 }' file.txt
```

Print specific fields:
```
awk '{ print $1, $3 }' file.txt
```

Print with line numbers:
```
awk '{ print NR, $0 }' file.txt
```

Print number of fields:
```
awk '{ print NF }' file.txt
```

Print last field:
```
awk '{ print $NF }' file.txt
```

## BEGIN Block

```
awk 'BEGIN { print "zoznam zamestnancov:" } { print }' zamestnanci.txt
```

BEGIN: runs once at start.  
Main: runs for every line.

Example:
```
awk 'BEGIN { print "zoznam zamestnancov:" }      { print "zamestnanec: " $1 }' zamestnanci.txt
```

Full flow:
```
awk 'BEGIN { print "zoznam zamestnancov:" }      { print "zamestnanec: " $1 }      END { print "koniec" }' zamestnanci.txt
```

Escaped quote example:
```
awk 'BEGIN {print "zoznam: "}      {print "employee first name: " $1}      END {print "that'''s all"}' zamestnanci.txt
```

## Pattern Matching

Match lines starting with `z`:
```
awk '/^z/' zamestnanci.txt
```

Negated:
```
awk '!/^z/' zamestnanci.txt
```

Pattern with action:
```
awk '/^z/ { print $1 ":" $NF }' zamestnanci.txt
```

## Numeric Comparison

```
awk '$NF > 1000 { print NR, $0 }' zamestnanci.txt
```

## Average Calculation

```
awk '{ sum += $NF } END { print "Priemerny plat = " sum/NR }' zamestnanci.txt
```

## Multiple Patterns (OR)

```
awk '/^z|^b/ { print $1 ":" $NF }' zamestnanci.txt
```


## Command Components

AWK command pattern:

```bash
awk [options] 'selection_criteria { action }' input-file > output-file
```

Component | Description | Example
---|---|---
`options` | Flags such as field separator | `-F':'`
`selection_criteria` | Pattern or condition to match | `$3 > 25`
`{ action }` | What to do when the pattern matches | `{ print $1 }`
`input-file` | File to process | `data.txt`
`> output-file` | Optional redirection of output | `> results.txt`

## Special Built‑in Variables

### Field Variables

Variable | Meaning | Example (line: `John 25 USA`)
---|---|---
`$0` | Entire current record (whole line) | `John 25 USA`
`$1` | First field | `John`
`$2` | Second field | `25`
`$3` | Third field | `USA`
`$n` | nth field | Any field number

### Record and Field Metadata

These variables control how AWK interprets and formats data:

- `NR` – current record number (line counter).  
  Example: `awk '{ print NR, $0 }' file` adds a line number.
- `NF` – number of fields in the current record.  
  Example: `awk '{ print NF }' file` prints field count per line; `$NF` is the last field.
- `FS` – input field separator (how AWK splits fields). Default: any whitespace.  
  Example: `awk -F':' '{ print $1 }' /etc/passwd` uses colon as separator.
- `OFS` – output field separator between fields in `print`. Default: single space.  
  Example: `awk 'BEGIN { OFS = ";" } { print $1, $2 }' file` joins fields with `;`.
- `RS` – input record separator (how records are split). Default: newline.
- `ORS` – output record separator added after each `print`. Default: newline.

Together, `$0`, `$1`, `$2`, …, `$NF`, `NR`, `NF`, `FS`, `OFS`, `RS`, and `ORS` form the core building blocks for reading, selecting, and rewriting text with AWK.
