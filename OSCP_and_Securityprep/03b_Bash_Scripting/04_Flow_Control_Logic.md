# Flow Control & Logic: Making Decisions

---
title: "Flow Control & Logic"
parent: "[[03b_Bash_Scripting]]"
tags:
  - bash
  - logic
  - if-else
  - case
  - test-operators
  - regex
  - exit-codes
created: 2026-01-23
updated: 2026-01-23
difficulty: Intermediate
length: Long
---

> **Executive Summary**: 
> A script that executes commands linearly is fragile. Real tools need to make decisions: "Is the server up? If yes, scan it. If no, skip it." "Does the file exist? If not, create it."
>
> This chapter covers the logic engine of Bash. We will dissect the difference between the legacy `[ ]` test command and the modern `[[ ]]` keyword (which supports Regex). We will master Exit Codes (`$?`) to chain commands safely and use `case` statements to build interactive menus.

---

## Table of Contents

1. [Exit Status: The Boolean of Bash](#1-exit-status-the-boolean-of-bash)
    - [1.1 Success (0) vs Failure (Non-Zero)](#11-success-0-vs-failure-non-zero)
    - [1.2 Chaining Logic (`&&`, `||`)](#12-chaining-logic---)
2. [The Test Commands: `[` vs `[[`](#2-the-test-commands--vs-)
    - [2.1 The Legacy `[` (test)](#21-the-legacy--test)
    - [2.2 The Modern `[[` (Keyword)](#22-the-modern--keyword)
    - [2.3 Comparison Operators (Integers vs Strings)](#23-comparison-operators-integers-vs-strings)
3. [If-Else Statements](#3-if-else-statements)
    - [3.1 Syntax Structure](#31-syntax-structure)
    - [3.2 File & Directory Tests](#32-file--directory-tests)
4. [Advanced Logic: Regex Matching](#4-advanced-logic-regex-matching)
    - [4.1 The `=~` Operator](#41-the--operator)
    - [4.2 Capturing Groups (`BASH_REMATCH`)](#42-capturing-groups-bash_rematch)
5. [Case Statements: Clean Menus](#5-case-statements-clean-menus)
    - [5.1 Syntax & Pattern Matching](#51-syntax--pattern-matching)
    - [5.2 Interactive Tools](#52-interactive-tools)
6. [Practical Lab: The Logic Gate](#6-practical-lab-the-logic-gate)
7. [Critical Analysis & Checkpoints](#7-critical-analysis--checkpoints)

---

## 1. Exit Status: The Boolean of Bash

In Python, `True` and `False` exist. In Bash, there is only the **Exit Status** of the last command.

### 1.1 Success (0) vs Failure (Non-Zero)
Every command returns a number between 0 and 255.
- **0**: Success ("True").
- **1-255**: Error ("False").

```bash
ping -c 1 8.8.8.8 > /dev/null
echo $?
# Output: 0 (If internet works)

ls /nonexistent_directory
echo $?
# Output: 2 (No such file)
```

### 1.2 Chaining Logic (`&&`, `||`)
You don't always need `if`.
- **`&&` (AND)**: Run the second command ONLY IF the first succeeded (0).
- **`||` (OR)**: Run the second command ONLY IF the first failed (Non-Zero).

```bash
# Update logic: Only upgrade if update succeeds
sudo apt update && sudo apt upgrade -y

# Fallback logic: Try pinging A, if fail, ping B
ping -c 1 10.0.0.1 || ping -c 1 10.0.0.2
```

---

## 2. The Test Commands: `[` vs `[[`

Bash has two ways to "Test" a condition.

### 2.1 The Legacy `[` (test)
`[` is actually a command (`/usr/bin/[`). It expects arguments.
- **Quoting**: Essential. `[ $VAR == "foo" ]` fails if VAR is empty. You MUST use `[ "$VAR" == "foo" ]`.
- **Logic**: Uses `-a` (AND) and `-o` (OR).

### 2.2 The Modern `[[` (Keyword)
`[[` is a shell keyword. It is safer, faster, and smarter.
- **Quoting**: Not required (mostly). `[[ $VAR == foo ]]` works even if empty.
- **Logic**: Uses `&&` and `||`.
- **Wildcards**: Supports pattern matching (`[[ $FILE == *.jpg ]]`).

**Recommendation**: Always use `[[ ]]` unless you need portability to ancient `sh` shells.

### 2.3 Comparison Operators (Integers vs Strings)
Bash gets confused easily. Be explicit.

| Type | Operator | Meaning | Example |
| :--- | :--- | :--- | :--- |
| **Integer** | `-eq` | Equal | `[[ $A -eq $B ]]` |
| **Integer** | `-ne` | Not Equal | `[[ $A -ne $B ]]` |
| **Integer** | `-gt` / `-lt` | Greater / Less | `[[ $A -gt 10 ]]` |
| **String** | `==` (or `=`) | Equal | `[[ "$S" == "admin" ]]` |
| **String** | `!=` | Not Equal | `[[ "$S" != "root" ]]` |
| **String** | `-z` | Empty (Zero length) | `[[ -z "$VAR" ]]` |
| **String** | `-n` | Not Empty | `[[ -n "$VAR" ]]` |

---

## 3. If-Else Statements

### 3.1 Syntax Structure
```bash
if [[ condition ]]; then
    # code
elif [[ other_condition ]]; then
    # code
else
    # code
fi
```
*Note*: The `;` after `condition` is required if `then` is on the same line.

### 3.2 File & Directory Tests
Critical for scripts that read/write data.
- **`-e file`**: Exists.
- **`-f file`**: Is a regular file (not a dir).
- **`-d file`**: Is a directory.
- **`-s file`**: Not empty (Size > 0).
- **`-w file`**: Writable (by me).
- **`-x file`**: Executable.

```bash
if [[ -f "/etc/passwd" ]]; then
    echo "Found passwd file."
fi
```

---

## 4. Advanced Logic: Regex Matching

Bash 3.0+ supports Regex inside `[[ ]]`.

### 4.1 The `=~` Operator
```bash
IP="192.168.1.50"
if [[ $IP =~ ^[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}$ ]]; then
    echo "Valid IP format"
else
    echo "Invalid IP"
fi
```
**Tip**: Don't quote the Regex pattern.

### 4.2 Capturing Groups (`BASH_REMATCH`)
You can extract data using parentheses.
```bash
EMAIL="admin@target.com"
REGEX="^(.+)@(.*)$"

if [[ $EMAIL =~ $REGEX ]]; then
    echo "User: ${BASH_REMATCH[1]}"  # admin
    echo "Domain: ${BASH_REMATCH[2]}" # target.com
fi
```

---

## 5. Case Statements: Clean Menus

Use `case` instead of massive `if-elif-elif` blocks when checking a single variable against multiple patterns.

### 5.1 Syntax & Pattern Matching
```bash
case "$VAR" in
    start|START)
        echo "Starting..."
        ;;
    stop|STOP)
        echo "Stopping..."
        ;;
    *)  # Wildcard (Default)
        echo "Usage: $0 {start|stop}"
        exit 1
        ;;
esac
```

### 5.2 Interactive Tools
```bash
echo "Select Target:"
echo "1) Web Server"
echo "2) DB Server"
read CHOICE

case $CHOICE in
    1) TARGET="10.10.10.10" ;;
    2) TARGET="10.10.10.20" ;;
    *) echo "Invalid"; exit 1 ;;
esac
```

---

## 6. Practical Lab: The Logic Gate

**Objective**: Write a script that checks if a user is root. If yes, check if a target IP is up. If up, check if Port 80 is open.

**Script: `logic_lab.sh`**
```bash
#!/bin/bash

TARGET=${1:-8.8.8.8}

# 1. Check Root
if [[ $EUID -ne 0 ]]; then
    echo "[-] Error: Run as root."
    exit 1
fi

echo "[+] Running as root."

# 2. Check Host Up (Using && logic)
ping -c 1 -W 1 $TARGET >/dev/null && echo "[+] Host $TARGET is UP." || {
    echo "[-] Host $TARGET is DOWN."
    exit 1
}

# 3. Check Port 80 (Using Regex logic for variety)
# Using nmap output parsing
OUTPUT=$(nmap -p 80 $TARGET -Pn)

if [[ $OUTPUT =~ "80/tcp open" ]]; then
    echo "[+] Port 80 is OPEN."
else
    echo "[-] Port 80 is CLOSED."
fi
```

---

## 7. Critical Analysis & Checkpoints

### Common Pitfalls
1.  **Integer vs String**: `[[ 10 > 2 ]]` works (String comparison, "1" vs "2"). `[[ 10 -gt 2 ]]` works (Math). But `[[ 10 > 2 ]]` sorts alphabetically, so 10 comes before 2? No, but be careful. Always use `-gt` for numbers.
2.  **Unbound Variables**: If `$VAR` is unset, `if [ $VAR == "foo" ]` becomes `if [ == "foo" ]` -> Syntax Error. Use `[[ ]]` or quotes `"$VAR"`.

### Checkpoint Questions
1.  **Q**: What is the exit code of `true`?
    -   *A*: 0.
2.  **Q**: How do you check if a string does NOT match a regex?
    -   *A*: `if [[ ! $VAR =~ regex ]]`.
3.  **Q**: What happens if you forget `;;` in a case statement?
    -   *A*: Syntax Error. It is required to terminate the block.

### Final Thoughts
Logic is what separates a script from a macro. By handling errors, validating input, and using regex, your scripts become robust tools that can handle the chaos of a real network assessment.

---
*End of Chapter 04*
