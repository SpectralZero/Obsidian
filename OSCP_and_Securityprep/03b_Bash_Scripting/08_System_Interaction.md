# System Interaction: Processes, Signals, and Jobs

---
title: "System Interaction"
parent: "[[03b_Bash_Scripting]]"
tags:
  - bash
  - processes
  - signals
  - jobs
  - traps
  - cron
  - scheduling
created: 2026-01-23
updated: 2026-01-23
difficulty: Advanced
length: Long
---

> **Executive Summary**: 
> A script doesn't run in a vacuum. It shares resources with the OS. Advanced scripting requires managing **Child Processes** (running tasks in parallel), handling **Signals** (what happens when the user hits Ctrl+C?), and scheduling tasks for later execution. This chapter bridges the gap between simple automation and robust system programming.

---

## Table of Contents

1. [Process Management](#1-process-management)
    - [1.1 Background Jobs (`&`)](#11-background-jobs-)
    - [1.2 Job Control (`jobs`, `fg`, `bg`)](#12-job-control-jobs-fg-bg)
    - [1.3 The `wait` Command](#13-the-wait-command)
2. [Signal Handling (Traps)](#2-signal-handling-traps)
    - [2.1 Signals (`SIGINT`, `SIGTERM`)](#21-signals-sigint-sigterm)
    - [2.2 The `trap` Command](#22-the-trap-command)
    - [2.3 Cleanup Patterns](#23-cleanup-patterns)
3. [Scheduling & Persistence](#3-scheduling--persistence)
    - [3.1 Cron (Recurring)](#31-cron-recurring)
    - [3.2 At (One-Time)](#32-at-one-time)
    - [3.3 Systemd Timers](#33-systemd-timers)
4. [Filesystem Interaction](#4-filesystem-interaction)
    - [4.1 Temporary Files (`mktemp`)](#41-temporary-files-mktemp)
    - [4.2 Locking (Mutex)](#42-locking-mutex)
5. [Practical Lab: The Resilient Monitor](#5-practical-lab-the-resilient-monitor)
6. [Critical Analysis & Checkpoints](#6-critical-analysis--checkpoints)

---

## 1. Process Management

Bash allows you to multi-task.

### 1.1 Background Jobs (`&`)
Append `&` to any command to run it asynchronously.
```bash
ping -c 10 8.8.8.8 &
echo "Ping started with PID $!"
```
- `$!`: The PID of the last background command.

### 1.2 Job Control (`jobs`, `fg`, `bg`)
- `jobs`: List background jobs.
- `fg %1`: Bring Job 1 to Foreground.
- `bg %1`: Resume Job 1 in Background (after Ctrl+Z).
- `disown`: Detach job from shell (keeps running if terminal closes).

### 1.3 The `wait` Command
Crucial for parallel scripts. Stops the script until background jobs finish.
```bash
for ip in 10.0.0.{1..10}; do
    nmap -p 80 $ip &
done
wait # Wait for ALL 10 scans to finish
echo "Scan Complete"
```

---

## 2. Signal Handling (Traps)

What happens if the user kills your script halfway through? Does it leave garbage files in `/tmp`?

### 2.1 Signals (`SIGINT`, `SIGTERM`)
- **SIGINT (2)**: Ctrl+C.
- **SIGTERM (15)**: `kill [pid]`. Polite termination.
- **SIGKILL (9)**: `kill -9 [pid]`. Immediate death. **Cannot be trapped**.
- **EXIT (0)**: Pseudo-signal when script finishes normally.

### 2.2 The `trap` Command
Register a function to run when a signal is received.
```bash
cleanup() {
    echo "Caught Ctrl+C. Cleaning up..."
    rm -f /tmp/tempfile
    exit 1
}

trap cleanup SIGINT SIGTERM
```

### 2.3 Cleanup Patterns
Best practice: Trap `EXIT`. This covers normal finish AND errors.
```bash
temp_file=$(mktemp)
trap "rm -f $temp_file" EXIT

echo "Working..."
# Even if I exit 1 here, the trap runs.
```

---

## 3. Scheduling & Persistence

### 3.1 Cron (Recurring)
Edit via `crontab -e`.
- Syntax: `m h dom mon dow command`.
- `*/5 * * * * /path/to/script.sh`: Run every 5 minutes.
- **Hacker Tip**: Look for writable scripts in `/etc/cron.d/`.

### 3.2 At (One-Time)
Run a command once in the future.
```bash
echo "rm -rf /tmp/evidence" | at now + 5 minutes
```

### 3.3 Systemd Timers
The modern replacement for Cron.
- Precision (seconds).
- Logging (journalctl).
- Dependencies (wait for network).

---

## 4. Filesystem Interaction

### 4.1 Temporary Files (`mktemp`)
Don't use hardcoded paths like `/tmp/file`. It's a security risk (Symlink Race Condition).
```bash
TEMP=$(mktemp)
echo "Data" > $TEMP
```

### 4.2 Locking (Mutex)
Prevent two instances of your script running at once.
```bash
LOCKFILE="/tmp/myscript.lock"
if mkdir "$LOCKFILE" 2>/dev/null; then
    trap "rmdir '$LOCKFILE'" EXIT
else
    echo "Script already running."
    exit 1
fi
```
*Why mkdir?* Because `mkdir` is atomic. Creating a file isn't always.

---

## 5. Practical Lab: The Resilient Monitor

**Objective**: Write a script that pings a target in parallel, cleans up on exit, and prevents double execution.

**Script: `monitor.sh`**
```bash
#!/bin/bash
set -u

LOCKDIR="/tmp/monitor.lock"
LOGFILE="monitor.log"

# 1. Mutex
if ! mkdir "$LOCKDIR" 2>/dev/null; then
    echo "Already running." >&2
    exit 1
fi

# 2. Cleanup Function
cleanup() {
    echo "[*] Cleaning up..."
    # Kill all child jobs of this script
    jobs -p | xargs -r kill
    rmdir "$LOCKDIR"
}

# 3. Trap Signals
trap cleanup EXIT SIGINT SIGTERM

echo "[*] Starting monitoring (Ctrl+C to stop)..."

# 4. Parallel Work
targets=("8.8.8.8" "1.1.1.1" "127.0.0.1")

for target in "${targets[@]}"; do
    (
        while true; do
            if ping -c 1 $target &>/dev/null; then
                echo "$(date): $target UP" >> $LOGFILE
            else
                echo "$(date): $target DOWN" >> $LOGFILE
            fi
            sleep 5
        done
    ) &
done

# 5. Wait forever (until Ctrl+C)
wait
```

---

## 6. Critical Analysis & Checkpoints

### Checkpoint Questions
1.  **Q**: Why use `mkdir` for locking instead of `touch`?
    -   *A*: `mkdir` is an atomic operation at the kernel level. `touch` (check existence, then create) has a race condition window between the check and the create.
2.  **Q**: What happens to background jobs when the parent script exits?
    -   *A*: They might become "Orphaned" (owned by init) or receive SIGHUP. It depends on shell options (`shopt -s huponexit`). Best practice is to kill them explicitly in a trap.
3.  **Q**: Can you trap `kill -9`?
    -   *A*: No. The kernel terminates the process immediately without notifying it.

### Final Thoughts
Robust scripts are polite. They clean up their mess, they don't hog the CPU, and they handle interruptions gracefully. This distinguishes a "hack" from a "tool".

---
*End of Chapter 08*
