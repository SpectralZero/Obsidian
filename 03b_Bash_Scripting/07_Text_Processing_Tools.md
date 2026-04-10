# Text Processing Tools: Grep, Sed, Awk

---
title: "Text Processing Tools"
parent: "[[03b_Bash_Scripting]]"
tags:
  - bash
  - grep
  - sed
  - awk
  - regex
  - parsing
created: 2026-01-23
updated: 2026-01-23
difficulty: Advanced
length: Long
---

> **Executive Summary**: 
> A shell script is essentially a text processing engine. Nmap outputs text. Log files are text. Configuration files are text. The ability to filter, transform, and extract data from these streams is the core skill of a Linux power user. This chapter covers the "Holy Trinity" of text processing: `grep` (Search), `sed` (Edit), and `awk` (Program). We will also cover essential helpers like `cut`, `sort`, `uniq`, and `tr`.

---

## Table of Contents

1. [Grep: The Search Engine](#1-grep-the-search-engine)
    - [1.1 Basic Searching](#11-basic-searching)
    - [1.2 Regular Expressions (Regex)](#12-regular-expressions-regex)
    - [1.3 Context and Inversion](#13-context-and-inversion)
2. [Sed: The Stream Editor](#2-sed-the-stream-editor)
    - [2.1 Substitution (`s///`)](#21-substitution-s)
    - [2.2 Deletion and Insertion](#22-deletion-and-insertion)
    - [2.3 In-Place Editing (`-i`)](#23-in-place-editing--i)
3. [Awk: The Programmable Filter](#3-awk-the-programmable-filter)
    - [3.1 Columns and Fields](#31-columns-and-fields)
    - [3.2 Logic and Math](#32-logic-and-math)
    - [3.3 Reporting](#33-reporting)
4. [Helper Tools: Cut, Sort, Uniq, Tr](#4-helper-tools-cut-sort-uniq-tr)
    - [4.1 Cut (Simple Slicing)](#41-cut-simple-slicing)
    - [4.2 Sort and Uniq (Aggregation)](#42-sort-and-uniq-aggregation)
    - [4.3 Tr (Character Translation)](#43-tr-character-translation)
5. [Practical Lab: Log Analysis Pipeline](#5-practical-lab-log-analysis-pipeline)
6. [Critical Analysis & Checkpoints](#6-critical-analysis--checkpoints)

---

## 1. Grep: The Search Engine

Global Regular Expression Print.

### 1.1 Basic Searching
- `grep "error" file.log`: Find lines containing "error".
- `grep -i "Error" file.log`: Case insensitive.
- `grep -r "password" .`: Recursive search (current dir and subdirs).
- `grep -l "main" *.c`: Show filenames only, not matching lines.

### 1.2 Regular Expressions (Regex)
Use `-E` (Extended Regex) for power.
- `grep -E "192\.168\.[0-9]{1,3}\.[0-9]{1,3}" access.log`
- `^`: Start of line.
- `$`: End of line.
- `|`: OR (`grep -E "error|warning" log`).

### 1.3 Context and Inversion
- `grep -v "success" log`: Show everything EXCEPT success.
- `grep -A 2 "Failed" log`: Show match AND 2 lines **After**.
- `grep -B 2 "Failed" log`: Show match AND 2 lines **Before**.
- `grep -C 2 "Failed" log`: Context (Both).

---

## 2. Sed: The Stream Editor

Sed modifies the stream line by line.

### 2.1 Substitution (`s///`)
The most common use.
```bash
echo "Hello World" | sed 's/World/Bash/'
# Output: Hello Bash
```
- `s/find/replace/g`: Global replace (all occurrences in line).
- `s/find/replace/I`: Case insensitive.

**Capture Groups**:
```bash
echo "User: Alice" | sed -E 's/User: (.*)/Username is \1/'
# Output: Username is Alice
```

### 2.2 Deletion and Insertion
- `sed '/password/d' config.txt`: Delete lines containing "password".
- `sed '1d' file`: Delete first line (header).
- `sed '$d' file`: Delete last line.

### 2.3 In-Place Editing (`-i`)
**Warning**: This modifies the file on disk.
```bash
sed -i 's/PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config
```
*Tip*: Use `-i.bak` to create a backup first.

---

## 3. Awk: The Programmable Filter

Awk is a full scripting language for structured text.

### 3.1 Columns and Fields
By default, Awk splits by whitespace.
- `$0`: The whole line.
- `$1`: First column.
- `$NF`: Last column.

```bash
# Print Users (Col 1) from /etc/passwd (Delimited by :)
awk -F ":" '{print $1}' /etc/passwd
```

### 3.2 Logic and Math
Count total size of files.
```bash
ls -l | awk '{sum += $5} END {print sum}'
```
Filter based on value:
```bash
# Print process IDs using > 10% CPU
ps aux | awk '$3 > 10.0 {print $2, $11}'
```

### 3.3 Reporting
```bash
awk -F: '{printf "User %-10s has UID %d\n", $1, $3}' /etc/passwd
```

---

## 4. Helper Tools: Cut, Sort, Uniq, Tr

### 4.1 Cut (Simple Slicing)
Faster/Simpler than Awk for single chars.
- `cut -c 1-10`: First 10 chars.
- `cut -d "," -f 2`: 2nd field of CSV.

### 4.2 Sort and Uniq (Aggregation)
`uniq` only works on **adjacent** lines, so you MUST sort first.
```bash
# Top 10 IPs
cat access.log | awk '{print $1}' | sort | uniq -c | sort -nr | head -n 10
```

### 4.3 Tr (Character Translation)
- `tr 'a-z' 'A-Z'`: Uppercase.
- `tr -d '\r'`: Delete CR (Windows newlines).
- `tr -s ' '`: Squeeze multiple spaces into one.

---

## 5. Practical Lab: Log Analysis Pipeline

**Scenario**: You have an Nginx log. You want to extract all IPs that accessed the `/admin` panel and check their GeoIP (simulated).

**Log format**: `IP - - [Date] "METHOD URL..." CODE SIZE`

**Script**:
```bash
#!/bin/bash

LOG_FILE="access.log"

echo "Extracting Admin Access..."

# 1. Grep for /admin
# 2. Awk to get IP (Col 1) and Code (Col 9)
# 3. Filter only 200 OK
grep "/admin" "$LOG_FILE" \
    | awk '$9 == 200 {print $1}' \
    | sort \
    | uniq -c \
    | sort -nr \
    | while read count ip; do
        echo "IP: $ip (Requests: $count)"
        # Simulated GeoIP
        # geoiplookup $ip
      done
```

---

## 6. Critical Analysis & Checkpoints

### Comparison
| Tool | Strength | Weakness | Use For |
| :--- | :--- | :--- | :--- |
| **Grep** | Searching | Cannot edit | Finding needles. |
| **Sed** | Editing | Bad at math/columns | Replacements. |
| **Awk** | Fields/Math | Complex syntax | CSVs, Reports. |

### Checkpoint Questions
1.  **Q**: How do you delete empty lines in a file with sed?
    -   *A*: `sed '/^$/d' file`.
2.  **Q**: How do you print line 10 of a file?
    -   *A*: `sed -n '10p' file` or `head -n 10 file | tail -n 1`.
3.  **Q**: How do you find a string starting with "Error" followed by any 3 digits?
    -   *A*: `grep -E "Error[0-9]{3}"`.

### Final Thoughts
These tools are older than most users, but they remain the fastest way to manipulate data. A regex-fu master is a dangerous hacker.

---
*End of Chapter 07*
