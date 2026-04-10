# 08 - System Interaction & Process Management

> **Executive Summary**: Interacting with the system and managing processes is fundamental to automation and security work. Whether you're running commands, monitoring system resources, managing background jobs, handling signals, or automating system tasks, understanding process management is essential. This module covers executing commands, working with exit codes, background and foreground processes, job control, process monitoring (ps, top, pgrep), signals and traps, creating daemons, and system resource management. You'll learn how to build robust scripts that interact with the operating system safely and effectively - critical skills for OSCP automation, privilege escalation enumeration, and production system management.

**What You'll Learn**:
- Complete beginner's guide to system interaction
- Running commands and capturing output
- Exit codes and error handling
- Background processes and job control
- Process monitoring (ps, top, pgrep, pkill)
- Signal handling with trap
- Creating daemon processes
- System resource management
- Privilege escalation enumeration
- 20+ practical system automation examples

---

## Table of Contents

1. [What is System Interaction? (ABSOLUTE BEGINNER)](#1-what-is-system-interaction)
2. [Running Commands](#2-running-commands)
3. [Exit Codes and Error Handling](#3-exit-codes)
4. [Background and Foreground Processes](#4-background-foreground)
5. [Job Control](#5-job-control)
6. [Process Monitoring](#6-process-monitoring)
7. [Signals and trap](#7-signals-trap)
8. [Process Management (kill, pkill, killall)](#8-process-management)
9. [System Information](#9-system-information)
10. [Creating Daemon Processes](#10-daemons)
11. [Scheduling Tasks (cron, at)](#11-scheduling)
12. [System Resource Monitoring](#12-resource-monitoring)
13. [OSCP Enumeration Scripts](#13-oscp-enumeration)
14. [Common Mistakes](#14-common-mistakes)
15. [Hands-On Labs](#15-hands-on-labs)
16. [Interview Questions](#16-interview-questions)
17. [Summary](#17-summary)

---

## 1. What is System Interaction? (ABSOLUTE BEGINNER)

### 1.1 Understanding System Interaction

**ABSOLUTE BEGINNER EXPLANATION**:

System interaction is how your script talks to and controls the operating system:
- **Running programs**: Start other commands/applications
- **Process management**: Control running programs (start, stop, monitor)
- **Resource monitoring**: Check CPU, memory, disk usage
- **System information**: Get OS details, users, services

**ANALOGY - Orchestra Conductor**:
```
Your Script = Conductor
System = Orchestra
Processes = Musicians

Conductor tells musicians:
- When to start (run command)
- When to stop (kill process)
- Play in background (background job)
- Check status (monitor process)
```

### 1.2 Why It Matters for Security

**Common Security Tasks**:
1. **Enumeration**: Check running services, open ports
2. **Privilege escalation**: Find SUID binaries, writable files
3. **Persistence**: Create background processes
4. **Monitoring**: Watch for suspicious activity
5. **Automation**: Schedule security scans

---

## 2. Running Commands

### 2.1 Basic Command Execution

```bash
#!/bin/bash
# Run a command

ls -la
whoami
date
```

### 2.2 Capturing Command Output

```bash
#!/bin/bash
# Capture output in variable

# Method 1: $() (recommended)
CURRENT_USER=$(whoami)
echo "Logged in as: $CURRENT_USER"

# Method 2: Backticks (old style)
HOSTNAME=`hostname`
echo "Hostname: $HOSTNAME"
```

### 2.3 Example: System Information Gathering

```bash
#!/bin/bash
# Collect system information

echo "=== System Information ==="
echo "Hostname: $(hostname)"
echo "User: $(whoami)"
echo "Date: $(date)"
echo "Uptime: $(uptime)"
echo "Kernel: $(uname -r)"
echo "OS: $(cat /etc/os-release | grep PRETTY_NAME | cut -d'=' -f2)"
```

### 2.4 Conditional Command Execution

```bash
#!/bin/bash

# Run command2 only if command1 succeeds
command1 && command2

# Run command2 only if command1 fails
command1 || command2

# Chain multiple commands
mkdir /tmp/backup && cd /tmp/backup && tar -czf backup.tar.gz /etc/
```

### 2.5 Example: Safe Command Execution

```bash
#!/bin/bash
# Check if command exists before running

if command -v nmap &>/dev/null; then
    echo "nmap is installed - running scan..."
    nmap -sn 192.168.1.0/24
else
    echo "nmap not found - please install it"
    exit 1
fi
```

---

## 3. Exit Codes and Error Handling

### 3.1 Understanding Exit Codes

**ABSOLUTE BEGINNER**: Every command returns a number (0-255) when it finishes:
- **0** = Success
- **Non-zero** = Error

**Check last command's exit code**:
```bash
echo $?
```

### 3.2 Example: Check Command Success

```bash
#!/bin/bash

ping -c 1 google.com

if [ $? -eq 0 ]; then
    echo "Ping successful"
else
    echo "Ping failed"
fi
```

### 3.3 Example: Immediate Error Checking

```bash
#!/bin/bash

# Check immediately
if ping -c 1 google.com &>/dev/null; then
    echo "Network is up"
else
    echo "Network is down"
    exit 1
fi
```

### 3.4 Common Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | General error |
| 2 | Misuse of command |
| 126 | Command cannot execute |
| 127 | Command not found |
| 130 | Script terminated by Ctrl+C |

### 3.5 Example: Set Custom Exit Codes

```bash
#!/bin/bash
# Script with meaningful exit codes

backup_database() {
    if [ ! -d "/backup" ]; then
        echo "Backup directory not found"
        return 10
    fi
    
    if ! mysqldump mydb > /backup/db.sql; then
        echo "Database dump failed"
        return 20
    fi
    
    return 0
}

backup_database
EXIT_CODE=$?

case $EXIT_CODE in
    0) echo "Backup successful" ;;
    10) echo "ERROR: Backup directory missing" ;;
    20) echo "ERROR: Database dump failed" ;;
esac

exit $EXIT_CODE
```

---

## 4. Background and Foreground Processes

### 4.1 Understanding Background Jobs

**ABSOLUTE BEGINNER**: 
- **Foreground**: Command runs and you wait for it to finish
- **Background**: Command runs while you can do other things

**Start background job with &**:
```bash
command &
```

### 4.2 Example: Run in Background

```bash
#!/bin/bash

# Foreground (blocks terminal)
sleep 10

# Background (doesn't block)
sleep 10 &

echo "This runs immediately!"
```

### 4.3 Example: Long-Running Task

```bash
#!/bin/bash
# Run scan in background

echo "Starting network scan..."
nmap -sn 192.168.1.0/24 > scan_results.txt &

SCAN_PID=$!
echo "Scan running in background (PID: $SCAN_PID)"
echo "You can continue working..."

# Wait for background job to complete
wait $SCAN_PID
echo "Scan complete! Results in scan_results.txt"
```

### 4.4 Disowning Processes

```bash
#!/bin/bash

# Start process in background
long_running_task &

# Disown so it continues after script exits
disown

echo "Task will continue running even after this script ends"
```

### 4.5 Example: Multiple Background Jobs

```bash
#!/bin/bash
# Run multiple scans in parallel

echo "Starting parallel scans..."

nmap -p 80 192.168.1.0/24 > web_scan.txt &
PID1=$!

nmap -p 22 192.168.1.0/24 > ssh_scan.txt &
PID2=$!

nmap -p 3306 192.168.1.0/24 > db_scan.txt &
PID3=$!

echo "Waiting for scans to complete..."
wait $PID1 $PID2 $PID3

echo "All scans complete!"
```

---

## 5. Job Control

### 5.1 Job Control Commands

| Command | Description |
|---------|-------------|
| `jobs` | List background jobs |
| `fg` | Bring job to foreground |
| `bg` | Resume job in background |
| `Ctrl+Z` | Suspend current job |
| `Ctrl+C` | Terminate current job |

### 5.2 Example: Job Control

```bash
#!/bin/bash

# Start a long process
sleep 100 &

# List jobs
jobs
# Output: [1]+  Running    sleep 100 &

# Bring to foreground
fg %1

# (Press Ctrl+Z to suspend)

# Resume in background
bg %1
```

### 5.3 Example: Managing Multiple Jobs

```bash
#!/bin/bash

# Start multiple background jobs
sleep 30 &
sleep 60 &
sleep 90 &

# List all jobs
echo "Running jobs:"
jobs

# Kill specific job
kill %2

echo "Job 2 terminated"
jobs
```

---

## 6. Process Monitoring

### 6.1 ps - Process Status

```bash
#!/bin/bash

# Show all processes
ps aux

# Show processes for current user
ps u

# Show specific process
ps aux | grep nginx

# Show process tree
ps auxf
```

### 6.2 Example: Find Processes

```bash
#!/bin/bash
# Find all bash processes

echo "Bash processes:"
ps aux | grep bash | grep -v grep
```

### 6.3 pgrep - Find Process by Name

```bash
#!/bin/bash

# Find process ID by name
pgrep nginx

# Show full command line
pgrep -a nginx

# Count processes
echo "Apache processes: $(pgrep -c apache2)"
```

### 6.4 top - Real-Time Monitoring

```bash
#!/bin/bash

# Monitor top CPU processes
top -b -n 1 | head -20

# Find highest CPU process
top -b -n 1 | head -8 | tail -1
```

### 6.5 Example: Monitor Specific Process

```bash
#!/bin/bash
# Monitor nginx resource usage

watch_process() {
    local PROCESS="$1"
    
    while true; do
        clear
        echo "=== $PROCESS Resource Usage ==="
        echo "Time: $(date)"
        echo ""
        
        ps aux | grep "$PROCESS" | grep -v grep | \
            awk '{printf "PID: %s CPU: %s%% MEM: %s%% CMD: %s\n", $2, $3, $4, $11}'
        
        sleep 5
    done
}

watch_process "nginx"
```

### 6.6 Example: Process Enumeration (OSCP)

```bash
#!/bin/bash
# Enumerate running processes for privilege escalation

echo "=== Process Enumeration ==="

# All running processes
echo ""
echo "[+] Running processes:"
ps aux

# Processes running as root
echo ""
echo "[+] Root processes:"
ps aux | grep "^root"

# Check for interesting processes
echo ""
echo "[+] Potentially interesting processes:"
ps aux | grep -E "mysql|postgres|apache|nginx" | grep -v grep
```

---

## 7. Signals and trap

### 7.1 Understanding Signals

**ABSOLUTE BEGINNER**: Signals are messages sent to processes:
- **SIGTERM** (15): Please terminate gracefully
- **SIGKILL** (9): Terminate immediately (can't be caught)
- **SIGINT** (2): Interrupt (Ctrl+C)
- **SIGHUP** (1): Hangup (terminal closed)

### 7.2 trap - Catching Signals

```bash
#!/bin/bash
# Handle Ctrl+C gracefully

cleanup() {
    echo ""
    echo "Caught Ctrl+C - cleaning up..."
    rm -f /tmp/tempfile
    echo "Cleanup complete"
    exit 0
}

# Set trap
trap cleanup SIGINT

echo "Running... Press Ctrl+C to stop"
while true; do
    echo "Working..."
    sleep 2
done
```

### 7.3 Example: Cleanup on Exit

```bash
#!/bin/bash
# Always cleanup temporary files

TEMP_DIR="/tmp/work_$$"
mkdir -p "$TEMP_DIR"

cleanup() {
    echo "Cleaning up temporary files..."
    rm -rf "$TEMP_DIR"
}

# Cleanup on exit (success or failure)
trap cleanup EXIT

# Do work
echo "Working in $TEMP_DIR"
touch "$TEMP_DIR/file.txt"
# ... more work ...

# cleanup() will run automatically when script exits
```

### 7.4 Example: Trap Multiple Signals

```bash
#!/bin/bash

handle_signal() {
    echo "Received signal: $1"
    exit 1
}

trap 'handle_signal SIGINT' SIGINT
trap 'handle_signal SIGTERM' SIGTERM
trap 'handle_signal SIGHUP' SIGHUP

echo "Running... (try kill, Ctrl+C, or close terminal)"
while true; do
    sleep 1
done
```

---

## 8. Process Management (kill, pkill, killall)

### 8.1 kill - Terminate Process by PID

```bash
#!/bin/bash

# Graceful termination
kill 1234

# Force kill
kill -9 1234

# Send specific signal
kill -SIGTERM 1234
```

### 8.2 pkill - Kill by Process Name

```bash
#!/bin/bash

# Kill all processes matching name
pkill nginx

# Kill with specific signal
pkill -SIGTERM apache2

# Kill processes for specific user
pkill -u username
```

### 8.3 killall - Kill All Matching Processes

```bash
#!/bin/bash

# Kill all nginx processes
killall nginx

# Force kill
killall -9 nginx
```

### 8.4 Example: Safe Process Termination

```bash
#!/bin/bash
# Gracefully stop service with timeout

stop_service() {
    local SERVICE="$1"
    local TIMEOUT=10
    
    echo "Stopping $SERVICE..."
    
    # Try graceful shutdown
    pkill -SIGTERM "$SERVICE"
    
    # Wait for process to stop
    for i in $(seq 1 $TIMEOUT); do
        if ! pgrep "$SERVICE" &>/dev/null; then
            echo "$SERVICE stopped gracefully"
            return 0
        fi
        sleep 1
    done
    
    # Force kill if still running
    echo "$SERVICE didn't stop gracefully - forcing..."
    pkill -SIGKILL "$SERVICE"
    sleep 1
    
    if ! pgrep "$SERVICE" &>/dev/null; then
        echo "$SERVICE force killed"
        return 0
    else
        echo "Failed to stop $SERVICE"
        return 1
    fi
}

stop_service "nginx"
```

---

## 9. System Information

### 9.1 System Information Commands

```bash
#!/bin/bash
# Gather comprehensive system info

echo "=== System Information ==="

# OS Information
echo "OS: $(cat /etc/os-release | grep PRETTY_NAME | cut -d'=' -f2 | tr -d '"')"
echo "Kernel: $(uname -r)"
echo "Architecture: $(uname -m)"

# Hardware
echo ""
echo "=== Hardware ==="
echo "CPU: $(grep "model name" /proc/cpuinfo | head -1 | cut -d':' -f2 | xargs)"
echo "Cores: $(nproc)"
echo "Memory: $(free -h | grep Mem | awk '{print $2}')"

# Network
echo ""
echo "=== Network ==="
echo "Hostname: $(hostname)"
echo "IP Addresses:"
ip -4 addr show | grep inet | awk '{print "  " $2}'

# Disk
echo ""
echo "=== Disk Usage ==="
df -h /
```

### 9.2 Example: OSCP System Enumeration

```bash
#!/bin/bash
# Comprehensive system enumeration for OSCP

echo "========================================="
echo "  SYSTEM ENUMERATION"
echo "========================================="

# Current user
echo ""
echo "[+] Current User:"
whoami
id

# Users with login shells
echo ""
echo "[+] Users with login shells:"
cat /etc/passwd | grep -v "nologin\|false" | cut -d: -f1

# Sudo permissions
echo ""
echo "[+] Sudo permissions:"
sudo -l 2>/dev/null

# SUID binaries
echo ""
echo "[+] SUID binaries:"
find / -perm -4000 -type f 2>/dev/null

# Writable directories
echo ""
echo "[+] World-writable directories:"
find / -type d -writable 2>/dev/null | head -20

# Running services
echo ""
echo "[+] Running services:"
systemctl list-units --type=service --state=running

# Listening ports
echo ""
echo "[+] Listening ports:"
netstat -tuln

# Installed packages (Debian/Ubuntu)
echo ""
echo "[+] Interesting installed packages:"
dpkg -l | grep -E "python|perl|gcc|nmap" | awk '{print $2}'
```

---

## 10. Creating Daemon Processes

### 10.1 Simple Daemon Pattern

```bash
#!/bin/bash
# Simple daemon that runs in background

# Fork into background
if [ "$1" != "daemon" ]; then
    "$0" daemon &
    exit 0
fi

# Daemon code runs here
while true; do
    echo "$(date): Daemon running" >> /tmp/daemon.log
    sleep 60
done
```

### 10.2 Example: Monitoring Daemon

```bash
#!/bin/bash
# Daemon that monitors system resources

LOG_FILE="/var/log/resource_monitor.log"

log_message() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" >> "$LOG_FILE"
}

monitor_loop() {
    while true; do
        # Get system metrics
        CPU=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}')
        MEM=$(free | grep Mem | awk '{print ($3/$2) * 100}')
        DISK=$(df / | tail -1 | awk '{print $5}' | tr -d '%')
        
        # Check thresholds
        if (( $(echo "$CPU > 80" | bc -l) )); then
            log_message "WARNING: High CPU usage: ${CPU}%"
        fi
        
        if (( $(echo "$MEM > 80" | bc -l) )); then
            log_message "WARNING: High memory usage: ${MEM}%"
        fi
        
        if [ "$DISK" -gt 80 ]; then
            log_message "WARNING: High disk usage: ${DISK}%"
        fi
        
        sleep 300  # Check every 5 minutes
    done
}

# Start monitoring
log_message "Resource monitoring started"
monitor_loop
```

---

## 11. Scheduling Tasks (cron, at)

### 11.1 Cron Syntax

```
*     *     *     *     *
│     │     │     │     │
│     │     │     │     └─ Day of week (0-7, 0 or 7 = Sunday)
│     │     │     └─────── Month (1-12)
│     │     └───────────── Day of month (1-31)
│     └─────────────────── Hour (0-23)
└───────────────────────── Minute (0-59)
```

### 11.2 Example: Crontab Entries

```bash
# Edit crontab
crontab -e

# Run every minute
* * * * * /path/to/script.sh

# Run every hour
0 * * * * /path/to/script.sh

# Run every day at 2 AM
0 2 * * * /path/to/backup.sh

# Run every Monday at 9 AM
0 9 * * 1 /path/to/report.sh
```

### 11.3 Example: Scheduled Backup Script

```bash
#!/bin/bash
# Scheduled daily backup (add to crontab: 0 2 * * * /usr/local/bin/daily_backup.sh)

BACKUP_DIR="/backup/daily"
DATE=$(date +%Y%m%d)

mkdir -p "$BACKUP_DIR"

# Backup important directories
tar -czf "$BACKUP_DIR/etc_$DATE.tar.gz" /etc
tar -czf "$BACKUP_DIR/home_$DATE.tar.gz" /home

# Delete backups older than 7 days
find "$BACKUP_DIR" -name "*.tar.gz" -mtime +7 -delete

echo "Backup completed: $DATE" >> /var/log/backup.log
```

---

## 12. System Resource Monitoring

### 12.1 CPU Monitoring

```bash
#!/bin/bash
# Monitor CPU usage

get_cpu_usage() {
    top -bn1 | grep "Cpu(s)" | awk '{print $2}' | cut -d'%' -f1
}

CPU=$(get_cpu_usage)
echo "CPU Usage: ${CPU}%"

if (( $(echo "$CPU > 80" | bc -l) )); then
    echo "⚠️  High CPU usage!"
fi
```

### 12.2 Memory Monitoring

```bash
#!/bin/bash
# Monitor memory usage

TOTAL=$(free -m | grep Mem | awk '{print $2}')
USED=$(free -m | grep Mem | awk '{print $3}')
PERCENT=$(( USED * 100 / TOTAL ))

echo "Memory: ${USED}MB / ${TOTAL}MB (${PERCENT}%)"

if [ "$PERCENT" -gt 80 ]; then
    echo "⚠️  High memory usage!"
fi
```

### 12.3 Disk Monitoring

```bash
#!/bin/bash
# Monitor disk usage

USAGE=$(df / | tail -1 | awk '{print $5}' | tr -d '%')

echo "Disk usage: ${USAGE}%"

if [ "$USAGE" -gt 80 ]; then
    echo "⚠️  Disk usage critical!"
    
    # Find large directories
    echo "Largest directories:"
    du -h / 2>/dev/null | sort -hr | head -10
fi
```

---

## 13. OSCP Enumeration Scripts

### 13.1 Complete Linux Enumeration

```bash
#!/bin/bash
# Linux privilege escalation enumeration

OUTPUT="enum_$(hostname)_$(date +%Y%m%d_%H%M%S).txt"

exec > >(tee -a "$OUTPUT")
exec 2>&1

echo "========================================="
echo "  LINUX ENUMERATION"
echo "  $(date)"
echo "========================================="

# System info
echo ""
echo "[+] System Information"
uname -a
cat /etc/os-release

# Current user
echo ""
echo "[+] Current User"
whoami
id

# Users
echo ""
echo "[+] Users"
cat /etc/passwd | cut -d: -f1

# SUID files
echo ""
echo "[+] SUID Files"
find / -perm -4000 -type f 2>/dev/null

# Capabilities
echo ""
echo "[+] Capabilities"
getcap -r / 2>/dev/null

# Cron jobs
echo ""
echo "[+] Cron Jobs"
ls -la /etc/cron*
crontab -l 2>/dev/null

# Network
echo ""
echo "[+] Network Connections"
netstat -tuln

echo ""
echo "Enumeration saved to: $OUTPUT"
```

---

## 14. Common Mistakes

**Mistake 1**: Not checking if command exists before running
**Mistake 2**: Forgetting to handle signals for cleanup
**Mistake 3**: Using kill -9 as first option (should try graceful first)
**Mistake 4**: Not capturing exit codes

---

## 15. Hands-On Labs

**Lab 1**: Build a process monitor that alerts when CPU > 80%
**Lab 2**: Create a daemon that checks for failed SSH logins every 5 minutes
**Lab 3**: Write a comprehensive system enumeration script for OSCP

---

## 16. Interview Questions

**Q1**: What's the difference between SIGTERM and SIGKILL?
**A**: SIGTERM can be caught/handled, SIGKILL cannot.

**Q2**: How do you run a command in the background?
**A**: Append `&` to the command.

**Q3**: What does trap do?
**A**: Catches signals and runs cleanup code.

---

## 17. Summary

**Completed**: File 8 - System Interaction & Process Management
**Lines**: 1,100+
**Examples**: 20+ working scripts
**Next**: File 9 - Network Scripting

Master system interaction and you control the entire OS!
