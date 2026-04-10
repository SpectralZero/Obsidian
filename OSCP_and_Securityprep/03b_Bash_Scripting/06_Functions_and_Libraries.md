# Functions & Libraries: Modular Scripting

---
title: "Functions & Libraries"
parent: "[[03b_Bash_Scripting]]"
tags:
  - bash
  - functions
  - scope
  - libraries
  - modularity
  - local-variables
created: 2026-01-23
updated: 2026-01-23
difficulty: Intermediate
length: Long
---

> **Executive Summary**: 
> Copy-pasting code is the root of all evil. If you use a block of code twice, put it in a function. Functions allow you to organize logic, limit variable scope, and create reusable libraries. This chapter covers the syntax nuances (`function name` vs `name()`), the critical concept of **Scope**, handling arguments, and managing return values in a language that doesn't really support them.

---

## Table of Contents

1. [Defining Functions](#1-defining-functions)
    - [1.1 Syntax Styles (Standard vs Legacy)](#11-syntax-styles-standard-vs-legacy)
    - [1.2 Where to Define](#12-where-to-define)
2. [Arguments & Parameters](#2-arguments--parameters)
    - [2.1 Passing Data](#21-passing-data)
    - [2.2 Shift and Argument Parsing](#22-shift-and-argument-parsing)
3. [Variable Scope: The Silent Killer](#3-variable-scope-the-silent-killer)
    - [3.1 Global by Default](#31-global-by-default)
    - [3.2 The `local` Keyword](#32-the-local-keyword)
4. [Return Values](#4-return-values)
    - [4.1 Exit Status (`return`)](#41-exit-status-return)
    - [4.2 Capturing Output (`echo`)](#42-capturing-output-echo)
5. [Creating Libraries](#5-creating-libraries)
    - [5.1 Sourcing Files](#51-sourcing-files)
    - [5.2 Guard Clauses](#52-guard-clauses)
6. [Practical Lab: The Logger Library](#6-practical-lab-the-logger-library)
7. [Critical Analysis & Checkpoints](#7-critical-analysis--checkpoints)

---

## 1. Defining Functions

### 1.1 Syntax Styles (Standard vs Legacy)

**Style 1: The C-Style (Portable & Recommended)**
```bash
my_function() {
    # Code here
    echo "Hello"
}
```
This works in `sh`, `dash`, `bash`, and `zsh`. It is the POSIX standard.

**Style 2: The Bash Keyword**
```bash
function my_function {
    # Code here
}
```
This is Bash-specific. It is readable but less portable.

**Style 3: The Hybrid (Avoid)**
`function my_function() { ... }`. Redundant. Don't use.

### 1.2 Where to Define
Functions must be defined **before** they are called. Bash reads the file top-to-bottom.
**Best Practice**: Put all functions at the top of the file, or in a separate library file.

---

## 2. Arguments & Parameters

Functions act like mini-scripts. They have their own `$1`, `$2`, `$@`.

### 2.1 Passing Data
```bash
greet() {
    echo "Hello, $1!"
}

greet "Alice"
# Output: Hello, Alice!
```
*Note*: The function's `$1` effectively shadows the script's global `$1`. Inside `greet`, `$1` is "Alice", not whatever argument was passed to the main script.

### 2.2 Shift and Argument Parsing
Use `shift` to process variable numbers of arguments.

```bash
scan_ports() {
    HOST=$1
    shift # Remove HOST from argument list
    PORTS=("$@") # Remaining args are ports
    
    echo "Scanning $HOST..."
    for port in "${PORTS[@]}"; do
        echo "Check $port"
    done
}

scan_ports 192.168.1.1 80 443 22
```

---

## 3. Variable Scope: The Silent Killer

This is the most common bug in Bash scripting.

### 3.1 Global by Default
```bash
my_func() {
    # No 'local' keyword
    result="I changed it"
}

result="Original"
my_func
echo $result
# Output: I changed it
```
The function overwrote the global variable. If `result` was a loop counter `i`, this would break your script.

### 3.2 The `local` Keyword
```bash
my_func() {
    local result="Safe change"
    echo "Inside: $result"
}

result="Original"
my_func
echo "Outside: $result"
# Output:
# Inside: Safe change
# Outside: Original
```
**Rule**: Always use `local` for variables defined inside a function.

---

## 4. Return Values

Bash functions do not return data like Python (`return data`). They return **Status Codes**.

### 4.1 Exit Status (`return`)
Returns an integer (0-255). 0 is Success.
```bash
check_root() {
    if [[ $EUID -eq 0 ]]; then
        return 0
    else
        return 1
    fi
}

if check_root; then
    echo "I am root"
fi
```

### 4.2 Capturing Output (`echo`)
To return data (strings, lists), you must `echo` it to STDOUT and capture it.
```bash
get_ip() {
    hostname -I | awk '{print $1}'
}

# Capture
MY_IP=$(get_ip)
echo "My IP is $MY_IP"
```
**Warning**: Anything you `echo` inside the function becomes part of the "Return Value". Do not `echo "Debug info"` if you intend to capture the output. Use `echo "Debug" >&2` (STDERR) instead.

---

## 5. Creating Libraries

Don't rewrite your logging logic in every script.

### 5.1 Sourcing Files
Create `utils.sh`.
```bash
# utils.sh
log() {
    echo "[$(date +%T)] $1"
}
```

In `main.sh`:
```bash
source ./utils.sh
# OR
. ./utils.sh

log "Starting scan..."
```

### 5.2 Guard Clauses
Prevent a library from being sourced twice (like C header guards).
```bash
[[ -n "${UTILS_LOADED:-}" ]] && return
readonly UTILS_LOADED=1
```

---

## 6. Practical Lab: The Logger Library

**Objective**: Create a reusable logging library with colors and levels.

**File 1: `logging.lib`**
```bash
#!/bin/bash

# Color Constants
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[0;33m'
NC='\033[0m' # No Color

log_info() {
    echo -e "${GREEN}[INFO]${NC} $1"
}

log_warn() {
    echo -e "${YELLOW}[WARN]${NC} $1" >&2
}

log_error() {
    echo -e "${RED}[ERROR]${NC} $1" >&2
}

check_dependency() {
    local cmd=$1
    if ! command -v $cmd &> /dev/null; then
        log_error "$cmd could not be found."
        return 1
    fi
    log_info "$cmd found."
    return 0
}
```

**File 2: `tool.sh`**
```bash
#!/bin/bash
source ./logging.lib

main() {
    log_info "Starting Tool..."
    
    if ! check_dependency "nmap"; then
        exit 1
    fi
    
    log_warn "This might take a while..."
    # Fake work
    sleep 1
    log_info "Done."
}

main
```

---

## 7. Critical Analysis & Checkpoints

### Checkpoint Questions
1.  **Q**: What happens if I use `return` outside a function?
    -   *A*: It errors out or behaves like `exit` (depending on shell). `return` is specific to functions and sourced scripts.
2.  **Q**: Can I pass an array to a function?
    -   *A*: Not directly. You pass the *elements* (`${ARR[@]}`) and rebuild the array inside (`local new_arr=("$@")`).
3.  **Q**: How do I print debug messages inside a function that returns a value?
    -   *A*: Print to STDERR (`echo "debug" >&2`). STDERR is not captured by `$()`.

### Final Thoughts
Functions transform a "Script" (linear list) into an "Application" (modular logic). By building a personal library of functions (logging, networking, parsing), you can assemble powerful tools in minutes.

---
*End of Chapter 06*
