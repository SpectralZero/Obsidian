# Flow Control & Loops: The Engine of Automation

---
title: "Flow Control & Loops"
parent: "[[03b_Bash_Scripting]]"
tags:
  - bash
  - loops
  - for-loop
  - while-loop
  - until-loop
  - automation
  - iteration
created: 2026-01-23
updated: 2026-01-23
difficulty: Intermediate
length: Long
---

> **Executive Summary**: 
> Logic (`if`) allows a script to make a decision once. Loops allow it to do it a million times. Loops are the difference between checking one IP and scanning the entire internet. This chapter masters the three engines of iteration: `for`, `while`, and `until`. We will explore advanced file reading techniques, infinite monitoring loops, and C-style iteration for performance.

---

## Table of Contents

1. [The For Loop: Iterating Lists](#1-the-for-loop-iterating-lists)
    - [1.1 List Iteration (The Standard)](#11-list-iteration-the-standard)
    - [1.2 Range Iteration (`{1..10}`)](#12-range-iteration-110)
    - [1.3 Globbing (File Iteration)](#13-globbing-file-iteration)
2. [The C-Style For Loop](#2-the-c-style-for-loop)
    - [2.1 Syntax and Math](#21-syntax-and-math)
    - [2.2 Performance implications](#22-performance-implications)
3. [The While Loop: Condition Based](#3-the-while-loop-condition-based)
    - [3.1 Standard Logic](#31-standard-logic)
    - [3.2 The Infinite Loop (Daemons)](#32-the-infinite-loop-daemons)
    - [3.3 Reading Files Line-by-Line](#33-reading-files-line-by-line)
4. [The Until Loop: The Inverse](#4-the-until-loop-the-inverse)
    - [4.1 Waiting for a Condition](#41-waiting-for-a-condition)
5. [Loop Control: Break and Continue](#5-loop-control-break-and-continue)
    - [5.1 `break` (Escape)](#51-break-escape)
    - [5.2 `continue` (Skip)](#52-continue-skip)
6. [Practical Lab: The Ping Sweeper](#6-practical-lab-the-ping-sweeper)
7. [Critical Analysis & Checkpoints](#7-critical-analysis--checkpoints)

---

## 1. The For Loop: Iterating Lists

The most common loop. "Do this for every item in this pile."

### 1.1 List Iteration (The Standard)
Iterates over space-separated items.

```bash
for IP in 10.0.0.1 10.0.0.2 10.0.0.3; do
    echo "Scanning $IP..."
    nmap -p 80 $IP
done
```

**Using Arrays**:
```bash
TARGETS=("web" "db" "mail")
for target in "${TARGETS[@]}"; do
    echo "Hacking $target"
done
```

### 1.2 Range Iteration (`{1..10}`)
Bash brace expansion creates lists of numbers.

```bash
# Scan last octet 1 to 254
for i in {1..254}; do
    echo "192.168.1.$i"
done
```
**Zero Padding**: `{01..10}` produces `01 02 ... 10`.

### 1.3 Globbing (File Iteration)
**NEVER** parse `ls`. Use Globs (`*`).

```bash
# Process all .pcap files
for file in *.pcap; do
    # Handle case where no files exist
    [[ -e "$file" ]] || break 
    echo "Analyzing $file"
    tshark -r "$file"
done
```

---

## 2. The C-Style For Loop

If you come from C/Java/Python, this logic is familiar. It uses the `(( ))` math context.

### 2.1 Syntax and Math
```bash
# for (( INIT; CONDITION; STEP ))
for (( i=0; i<10; i++ )); do
    echo "Count: $i"
done
```

### 2.2 Performance implications
It is slightly faster for math-heavy operations but less readable for text processing. Use it when you need an index counter.

---

## 3. The While Loop: Condition Based

Runs *while* a command returns Success (0).

### 3.1 Standard Logic
```bash
COUNT=0
while [[ $COUNT -lt 5 ]]; do
    echo "Count: $COUNT"
    (( COUNT++ ))
done
```

### 3.2 The Infinite Loop (Daemons)
Useful for monitoring or persistence.
```bash
while true; do
    nc -l -p 4444 -e /bin/bash
    echo "Shell died. Restarting..."
    sleep 1
done
```
*Note*: `true` is a command that always exits with 0. `:` is a shorter synonym (`while :; do`).

### 3.3 Reading Files Line-by-Line
The most robust way to read a file. It handles spaces safely.

```bash
while read -r line; do
    echo "Processing: $line"
done < targets.txt
```
- `-r`: Prevents backslash escaping (treats `\` as a character).
- `< targets.txt`: Redirects the file into the loop's STDIN.

---

## 4. The Until Loop: The Inverse

Runs *until* a command returns Success (0). (Runs while it fails).

### 4.1 Waiting for a Condition
Wait for a host to come online.
```bash
echo "Waiting for 10.10.10.10..."
until ping -c 1 10.10.10.10 >/dev/null 2>&1; do
    sleep 1
done
echo "Host is UP!"
```

---

## 5. Loop Control: Break and Continue

### 5.1 `break` (Escape)
Exits the loop immediately.
```bash
for ip in {1..254}; do
    if ping -c 1 192.168.1.$ip; then
        echo "Found live host!"
        break # Stop scanning after finding one
    fi
done
```

### 5.2 `continue` (Skip)
Skips the rest of the current iteration and jumps to the next.
```bash
for file in *; do
    if [[ "$file" == *.txt ]]; then
        continue # Skip text files
    fi
    echo "Processing binary: $file"
done
```

---

## 6. Practical Lab: The Ping Sweeper

**Objective**: Write a script that scans a Class C subnet (`192.168.1.x`), identifies live hosts, and logs them.

**Script: `sweeper.sh`**
```bash
#!/bin/bash

SUBNET=$1

if [[ -z "$SUBNET" ]]; then
    echo "Usage: $0 <192.168.1>"
    exit 1
fi

echo "[*] Sweeping ${SUBNET}.0/24..."

for i in {1..254}; do
    IP="${SUBNET}.${i}"
    
    # Ping with timeout (W) and count (c)
    # Run in background (&) for speed
    (
        if ping -c 1 -W 1 $IP >/dev/null 2>&1; then
            echo "[+] $IP is ALIVE"
        fi
    ) &
done

# Wait for all background pings to finish
wait
echo "[*] Scan complete."
```

**Optimization**: The `( ) &` runs the check in a subshell in the background. This makes the scanner parallel (fast) instead of sequential (slow).

---

## 7. Critical Analysis & Checkpoints

### Common Pitfalls
1.  **Reading Output of a Pipe**:
    ```bash
    cat file | while read line; do VAR=$line; done
    echo $VAR
    ```
    **Bug**: The loop runs in a Subshell because of the pipe `|`. Variable assignments inside die when the loop ends. `$VAR` will be empty.
    **Fix**: Use redirection `< file` or Process Substitution `< <(cat file)`.

2.  **Infinite Loops**: Always include a `sleep` in monitoring loops to avoid 100% CPU usage.

### Checkpoint Questions
1.  **Q**: What is the difference between `while` and `until`?
    -   *A*: `while` runs as long as the condition is TRUE (0). `until` runs as long as the condition is FALSE (Non-zero).
2.  **Q**: How do you loop through files with spaces in their names?
    -   *A*: Use `for file in *; do ... done`. Do NOT use `for file in $(ls)`.

### Final Thoughts
Loops define the scale of your impact. A hacker who manually checks 10 servers is limited by time. A hacker who loops checks 10,000 servers in the same time.

---
*End of Chapter 05*
