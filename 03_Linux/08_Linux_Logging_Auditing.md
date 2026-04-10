# Linux Logging & Auditing - COMPREHENSIVE GUIDE

---
title: "Linux Logging & Auditing"
parent: "[[03_Linux]]"
tags:
  - linux
  - logs
  - auditd
  - syslog
  - forensics
  - evasion
  - journald
  - rsyslog
created: 2026-01-23
updated: 2026-01-24
---

> **Executive Summary**: Logs are the memory of a Linux system - they record every login, command execution, system event, and security incident. For defenders, logs are the primary source of truth for incident response, forensics, and security monitoring. For attackers (OSCP perspective), logs are evidence that must be understood, evaded, or eliminated. Understanding Linux logging is CRITICAL because you need to: analyze logs for reconnaissance, detect failed login attempts, find privilege escalation attempts, cover your tracks after exploitation, and understand what defenders can see. This comprehensive guide covers the complete Linux logging ecosystem - from traditional syslog to modern systemd journald, kernel-level auditd, log file locations, log analysis techniques, covering tracks, and defensive hardening. You'll learn not just where logs are stored, but how to read them, search them efficiently, detect anomalies, understand what gets logged when, and most importantly - how attackers hide their activities and how defenders protect log integrity. With 20+ working examples, hands-on labs, and real OSCP scenarios, you'll master Linux logging from both offensive and defensive perspectives.

---

## Table of Contents

1. [Understanding Linux Logging (Absolute Beginner)](#1-understanding-linux-logging-absolute-beginner)
2. [The Linux Logging Architecture](#2-the-linux-logging-architecture)
3. [Syslog and rsyslog](#3-syslog-and-rsyslog)
4. [systemd journald](#4-systemd-journald)
5. [Important Log Files in /var/log](#5-important-log-files-in-varlog)
6. [Authentication Logs](#6-authentication-logs)
7. [System Logs](#7-system-logs)
8. [Application Logs](#8-application-logs)
9. [Log Rotation with logrotate](#9-log-rotation-with-logrotate)
10. [auditd: Kernel-Level Auditing](#10-auditd-kernel-level-auditing)
11. [Searching and Analyzing Logs](#11-searching-and-analyzing-logs)
12. [Detecting Security Events](#12-detecting-security-events)
13. [Failed Login Detection](#13-failed-login-detection)
14. [Privilege Escalation Detection](#14-privilege-escalation-detection)
15. [Covering Tracks (Anti-Forensics)](#15-covering-tracks-anti-forensics)
16. [Log Integrity and Protection](#16-log-integrity-and-protection)
17. [Centralized Logging](#17-centralized-logging)
18. [Log Analysis Tools](#18-log-analysis-tools)
19. [SIEM Basics](#19-siem-basics)
20. [Hands-On Labs](#20-hands-on-labs)
21. [Interview Questions and Answers](#21-interview-questions-and-answers)
22. [Summary and Next Steps](#22-summary-and-next-steps)

---

## 1. Understanding Linux Logging (Absolute Beginner)

### 1.1 What Are Logs?

**Logs** are files that record events happening on the system.

**Real-world analogy**: Logs are like a security camera's recording - they capture what happened, when, and by whom.

**Why logs matter**:
- **Troubleshooting**: Find out why something broke
- **Security**: Detect intrusions, failed logins, privilege escalation
- **Compliance**: Prove what happened for audits
- **Forensics**: Investigate after a breach

### 1.2 What Gets Logged?

```
┌────────────────────────────────────────┐
│         EVENTS THAT GET LOGGED         │
├────────────────────────────────────────┤
│  AUTHENTICATION                        │
│  - SSH logins (success/failure)        │
│  - sudo commands                       │
│  - su attempts                         │
│  - Password changes                    │
│                                        │
│  SYSTEM EVENTS                         │
│  - Service starts/stops                │
│  - System boot/shutdown                │
│  - Hardware errors                     │
│  - Kernel messages                     │
│                                        │
│  APPLICATIONS                          │
│  - Web server requests                 │
│  - Database queries                    │
│  - Email delivery                      │
│  - Cron job execution                  │
│                                        │
│  SECURITY                              │
│  - Firewall blocks                     │
│  - File access (if auditd configured)  │
│  - Command execution (if logged)       │
│  - Permission denials                  │
└────────────────────────────────────────┘
```

### 1.3 Where Are Logs Stored?

**Primary location**: `/var/log/`

```bash
# List log directory
ls -lah /var/log/

# Common logs:
/var/log/syslog          # General system messages
/var/log/auth.log        # Authentication (Debian/Ubuntu)
/var/log/secure          # Authentication (RedHat/CentOS)
/var/log/kern.log        # Kernel messages
/var/log/dmesg           # Boot messages
/var/log/messages        # General messages (RedHat)
```

### 1.4 Log Format Example

```
Jan 24 10:30:15 server sshd[1234]: Accepted publickey for john from 192.168.1.100 port 54321 ssh2
│              │      │     │                                                                    │
│              │      │     │                                                                    │
Timestamp      Host   Process PID    Message content
```

**Typical log entry components**:
- **Timestamp**: When it happened
- **Hostname**: Which server
- **Process**: Which program logged it
- **PID**: Process ID
- **Message**: What happened

---

## 2. The Linux Logging Architecture

### 2.1 Logging Components

```
┌──────────────────────────────────────────────┐
│       LINUX LOGGING ARCHITECTURE             │
├──────────────────────────────────────────────┤
│                                              │
│  APPLICATION LAYER                           │
│  ┌────────┐  ┌────────┐  ┌────────┐        │
│  │ sshd   │  │ apache │  │ mysql  │        │
│  └───┬────┘  └───┬────┘  └───┬────┘        │
│      │           │           │              │
│      │           │           │              │
│      ▼           ▼           ▼              │
│  ┌──────────────────────────────┐          │
│  │     LOGGING DAEMONS          │          │
│  │  ┌──────────┐  ┌──────────┐ │          │
│  │  │ rsyslog  │  │ journald │ │          │
│  │  └────┬─────┘  └────┬─────┘ │          │
│  └───────┼─────────────┼────────┘          │
│          │             │                    │
│          ▼             ▼                    │
│  ┌────────────┐  ┌──────────────┐         │
│  │ /var/log/* │  │ journald DB  │         │
│  │ (text)     │  │ (binary)     │         │
│  └────────────┘  └──────────────┘         │
│                                              │
│  KERNEL LAYER                                │
│  ┌──────────┐                               │
│  │ auditd   │ ← Kernel syscall monitoring   │
│  └────┬─────┘                               │
│       │                                      │
│       ▼                                      │
│  /var/log/audit/audit.log                   │
│                                              │
└──────────────────────────────────────────────┘
```

### 2.2 Traditional vs Modern Logging

**Traditional (Syslog)**:
- Text-based logs in /var/log/
- rsyslog or syslog-ng daemon
- Simple, human-readable
- Easy to tamper with

**Modern (systemd journald)**:
- Binary logs
- Indexed and structured
- Harder to tamper with
- Better metadata (UID, GID, SELinux context)
- Can coexist with syslog

### 2.3 Log Severity Levels

**Syslog severity** (RFC 5424):

| Level | Name | Description |
|:------|:-----|:------------|
| 0 | Emergency | System unusable |
| 1 | Alert | Immediate action required |
| 2 | Critical | Critical conditions |
| 3 | Error | Error conditions |
| 4 | Warning | Warning conditions |
| 5 | Notice | Normal but significant |
| 6 | Informational | Informational messages |
| 7 | Debug | Debug messages |

### 2.4 Log Facilities

**Syslog facilities** (categories):

| Facility | Description |
|:---------|:------------|
| auth | Authentication/security |
| authpriv | Private authentication |
| cron | Cron daemon |
| daemon | System daemons |
| kern | Kernel messages |
| mail | Mail system |
| user | User-level messages |
| local0-local7 | Custom use |

---

## 3. Syslog and rsyslog

### 3.1 What Is rsyslog?

**rsyslog** (Rocket-fast syslog) is the modern implementation of syslog.

**Features**:
- High performance
- Filtering and forwarding
- Remote logging
- Templates for custom formats
- Database output

### 3.2 rsyslog Configuration

**Main config**: `/etc/rsyslog.conf`

**Basic syntax**:
```
facility.priority    action

# Examples:
auth.info            /var/log/auth.log
kern.*               /var/log/kern.log
*.emerg              :omusrmsg:*
```

**Example config**:
```bash
cat /etc/rsyslog.conf
```

```
# Log auth messages
auth,authpriv.*                 /var/log/auth.log

# All messages except auth
*.*;auth,authpriv.none          /var/log/syslog

# Emergency messages to all users
*.emerg                         :omusrmsg:*

# Kernel messages
kern.*                          /var/log/kern.log

# Mail logs
mail.*                          /var/log/mail.log

# Cron logs
cron.*                          /var/log/cron.log
```

### 3.3 Custom rsyslog Rules

**Example: Log SSH activity separately**

```bash
# Edit config
sudo nano /etc/rsyslog.d/50-ssh.conf
```

```
# Log all sshd messages to separate file
:programname, isequal, "sshd" /var/log/ssh.log
& stop
```

```bash
# Restart rsyslog
sudo systemctl restart rsyslog
```

### 3.4 Remote Logging

**Send logs to remote server**:

```bash
# Edit /etc/rsyslog.conf
sudo nano /etc/rsyslog.conf

# Add at end:
# UDP
*.* @192.168.1.50:514

# TCP (more reliable)
*.* @@192.168.1.50:514
```

**Receive logs on server**:
```bash
# Edit /etc/rsyslog.conf
# Uncomment:
module(load="imudp")
input(type="imudp" port="514")

module(load="imtcp")
input(type="imtcp" port="514")

# Restart
sudo systemctl restart rsyslog
```

### 3.5 Logger Command

**Manually write to syslog**:

```bash
# Basic message
logger "Test message"

# With priority
logger -p auth.info "SSH login from admin"

# With tag
logger -t myapp "Application started"

# View in syslog
tail /var/log/syslog
```

---

## 4. systemd journald

### 4.1 What Is journald?

**journald** is systemd's logging system that stores logs in binary format.

**Advantages**:
- Structured logging (searchable by fields)
- Automatic metadata (UID, GID, PID, command, SELinux context)
- Indexed for fast searching
- Tamper-resistant (binary format)
- Boot logs preserved

**Location**:
- Volatile: `/run/log/journal/` (lost on reboot)
- Persistent: `/var/log/journal/` (preserved)

### 4.2 journalctl Commands

```bash
# View all logs
journalctl

# Latest entries
journalctl -e

# Follow live (like tail -f)
journalctl -f

# Reverse order (newest first)
journalctl -r

# Number of lines
journalctl -n 50

# Current boot only
journalctl -b

# Previous boot
journalctl -b -1

# Specific service
journalctl -u sshd
journalctl -u apache2

# Specific user
journalctl _UID=1000

# Specific process
journalctl _PID=1234

# Kernel messages
journalctl -k
```

### 4.3 Time-Based Filtering

```bash
# Since specific time
journalctl --since "2026-01-24 10:00:00"

# Until specific time
journalctl --until "2026-01-24 12:00:00"

# Time range
journalctl --since "1 hour ago"
journalctl --since "2 days ago"
journalctl --since "yesterday"
journalctl --since "2026-01-24" --until "2026-01-25"

# Today's logs
journalctl --since today

# Last boot
journalctl -b 0
```

### 4.4 Priority Filtering

```bash
# Errors and above
journalctl -p err

# Warning and above
journalctl -p warning

# Info and above
journalctl -p info

# Priority levels: emerg, alert, crit, err, warning, notice, info, debug
```

### 4.5 Field-Based Filtering

```bash
# By executable
journalctl /usr/bin/sshd

# By command
journalctl _COMM=sshd

# By UID
journalctl _UID=0  # Root activities

# By GID
journalctl _GID=33  # www-data

# Multiple filters (AND)
journalctl _SYSTEMD_UNIT=sshd.service _PID=1234

# Output formats
journalctl -o json-pretty
journalctl -o verbose
journalctl -o short
```

### 4.6 Disk Usage and Maintenance

```bash
# Check disk usage
journalctl --disk-usage

# Vacuum (clean old logs)
sudo journalctl --vacuum-time=7d  # Keep 7 days
sudo journalctl --vacuum-size=500M  # Keep 500MB max

# Rotate logs
sudo journalctl --rotate

# Verify integrity
sudo journalctl --verify
```

---

## 5. Important Log Files in /var/log

### 5.1 Authentication and Security

```bash
# Debian/Ubuntu
/var/log/auth.log         # SSH, sudo, su, logins

# RedHat/CentOS
/var/log/secure           # Same as auth.log

# Last logins
/var/log/lastlog          # Binary (use lastlog command)
/var/log/wtmp             # Login records (use last command)
/var/log/btmp             # Failed logins (use lastb command)

# Sudo logs
/var/log/sudo.log         # If configured separately
```

### 5.2 System Logs

```bash
/var/log/syslog           # General system messages (Debian)
/var/log/messages         # General messages (RedHat)
/var/log/kern.log         # Kernel messages
/var/log/dmesg            # Boot messages
/var/log/boot.log         # System boot
```

### 5.3 Application Logs

```bash
# Web servers
/var/log/apache2/access.log
/var/log/apache2/error.log
/var/log/nginx/access.log
/var/log/nginx/error.log

# Databases
/var/log/mysql/error.log
/var/log/postgresql/postgresql-14-main.log

# Mail
/var/log/mail.log
/var/log/mail.err

# FTP
/var/log/vsftpd.log

# Cron
/var/log/cron.log
```

### 5.4 Package Management

```bash
# Debian/Ubuntu
/var/log/dpkg.log         # Package installations
/var/log/apt/history.log  # APT history
/var/log/apt/term.log     # APT terminal output

# RedHat/CentOS
/var/log/yum.log          # YUM package manager
```

---

## 6. Authentication Logs

### 6.1 Reading auth.log

```bash
# View authentication log
sudo tail -f /var/log/auth.log

# Example entries:
# Successful SSH login:
Jan 24 10:30:42 server sshd[1234]: Accepted publickey for john from 192.168.1.100 port 54321 ssh2

# Failed SSH login:
Jan 24 10:31:15 server sshd[1235]: Failed password for invalid user admin from 192.168.1.200 port 45678 ssh2

# Sudo usage:
Jan 24 10:32:00 server sudo:    john : TTY=pts/0 ; PWD=/home/john ; USER=root ; COMMAND=/bin/cat /etc/shadow

# Su attempt:
Jan 24 10:33:05 server su[1236]: pam_unix(su:auth): authentication failure; logname=john uid=1000 euid=0 tty=/dev/pts/0
```

### 6.2 Searching Auth Logs

```bash
# Find all SSH logins
grep "Accepted" /var/log/auth.log

# Find failed SSH attempts
grep "Failed password" /var/log/auth.log

# Find sudo usage
grep "sudo:" /var/log/auth.log

# Find su attempts
grep "su\[" /var/log/auth.log

# Find specific user activity
grep "john" /var/log/auth.log

# Find activity from IP
grep "192.168.1.100" /var/log/auth.log

# Count failed logins
grep -c "Failed password" /var/log/auth.log
```

### 6.3 Last Command (wtmp)

```bash
# Show recent logins
last

# Example output:
# john    pts/0    192.168.1.100   Wed Jan 24 10:30   still logged in
# admin   pts/1    10.10.10.50     Wed Jan 24 09:15 - 09:45  (00:30)

# Last logins for specific user
last john

# Last logins from specific host
last -i 192.168.1.100

# Show reboot history
last reboot

# Full hostnames
last -d
```

### 6.4 Failed Login Attempts (btmp)

```bash
# Show failed login attempts
sudo lastb

# Count failed attempts
sudo lastb | wc -l

# Failed attempts for specific user
sudo lastb admin

# Failed attempts from IP
sudo lastb | grep 192.168.1.200
```

---

## 7. System Logs

### 7.1 Kernel Messages

```bash
# View kernel ring buffer
dmesg

# Follow kernel messages
dmesg -w

# Human-readable timestamps
dmesg -T

# Specific facility
dmesg --facility=kern

# Specific level
dmesg --level=err

# Clear buffer
sudo dmesg -c

# View kern.log
tail /var/log/kern.log
```

### 7.2 Boot Messages

```bash
# Current boot messages
dmesg | less

# Boot log file
cat /var/log/boot.log

# systemd boot messages
journalctl -b

# List all boots
journalctl --list-boots

# Specific boot
journalctl -b -1  # Previous boot
```

### 7.3 System Messages

```bash
# General messages (Debian/Ubuntu)
tail /var/log/syslog

# General messages (RedHat/CentOS)
tail /var/log/messages

# Filter for specific service
grep "sshd" /var/log/syslog

# Filter by date
grep "Jan 24" /var/log/syslog
```

---

## 8. Application Logs

### 8.1 Web Server Logs

**Apache access log**:
```bash
tail -f /var/log/apache2/access.log

# Example entry:
192.168.1.100 - - [24/Jan/2026:10:30:42 +0000] "GET /index.html HTTP/1.1" 200 1234 "-" "Mozilla/5.0"
│            │ │  │                             │                           │   │    │    │
│            │ │  │                             │                           │   │    │    └─ User-Agent
│            │ │  │                             │                           │   │    └─ Referer
│            │ │  │                             │                           │   └─ Size
│            │ │  │                             │                           └─ Status code
│            │ │  │                             └─ Request
│            │ │  └─ Timestamp
│            │ └─ Authenticated user
│            └─ Remote user
└─ IP address
```

**Apache error log**:
```bash
tail -f /var/log/apache2/error.log

# Find PHP errors
grep "PHP" /var/log/apache2/error.log

# Find 404 errors
grep "File does not exist" /var/log/apache2/error.log
```

### 8.2 Database Logs

**MySQL**:
```bash
# Error log
tail /var/log/mysql/error.log

# Query log (if enabled)
tail /var/log/mysql/query.log

# Slow query log
tail /var/log/mysql/slow.log
```

**PostgreSQL**:
```bash
# Main log
tail /var/log/postgresql/postgresql-14-main.log

# Find failed connections
grep "FATAL" /var/log/postgresql/postgresql-14-main.log
```

### 8.3 Mail Logs

```bash
# Mail delivery
tail /var/log/mail.log

# Mail errors
tail /var/log/mail.err

# Find specific email
grep "user@example.com" /var/log/mail.log
```

---

## 9. Log Rotation with logrotate

### 9.1 What Is Log Rotation?

**logrotate** automatically compresses and archives old logs to save disk space.

```
auth.log           (current)
auth.log.1         (yesterday)
auth.log.2.gz      (2 days ago, compressed)
auth.log.3.gz      (3 days ago, compressed)
...
```

### 9.2 logrotate Configuration

**Main config**: `/etc/logrotate.conf`
**Service configs**: `/etc/logrotate.d/`

**Example config**:
```bash
cat /etc/logrotate.d/rsyslog
```

```
/var/log/syslog
/var/log/auth.log
{
    rotate 7           # Keep 7 days
    daily              # Rotate daily
    missingok          # Don't error if missing
    notifempty         # Don't rotate if empty
    delaycompress      # Delay compression for 1 cycle
    compress           # Compress old logs
    postrotate         # Run after rotation
        systemctl reload rsyslog > /dev/null 2>&1
    endscript
}
```

### 9.3 Custom logrotate Rules

```bash
# Create custom config
sudo nano /etc/logrotate.d/myapp
```

```
/var/log/myapp/*.log {
    daily
    rotate 30
    compress
    delaycompress
    notifempty
    missingok
    create 0640 www-data adm
}
```

### 9.4 Manual Rotation

```bash
# Force rotation
sudo logrotate -f /etc/logrotate.conf

# Test without executing
sudo logrotate -d /etc/logrotate.conf

# Verbose mode
sudo logrotate -v /etc/logrotate.conf
```

---

## 10. auditd: Kernel-Level Auditing

### 10.1 What Is auditd?

**auditd** is the Linux Audit Daemon - provides kernel-level auditing of system calls and file access.

**Unlike syslog** (applications choose what to log), **auditd** watches at the kernel level - cannot be bypassed by processes.

**Use cases**:
- Compliance (PCI-DSS, HIPAA)
- Intrusion detection
- Forensics
- File integrity monitoring

### 10.2 Installing auditd

```bash
# Debian/Ubuntu
sudo apt install auditd

# RedHat/CentOS
sudo yum install audit

# Start and enable
sudo systemctl start auditd
sudo systemctl enable auditd
```

### 10.3 Basic auditd Configuration

**Config file**: `/etc/audit/auditd.conf`
**Rules**: `/etc/audit/audit.rules` or `/etc/audit/rules.d/`

**Check status**:
```bash
sudo auditctl -s

# Output shows:
# enabled 1
# rules 10
# ...
```

### 10.4 Creating Audit Rules

**Watch file access**:
```bash
# Watch file reads
sudo auditctl -w /etc/passwd -p r -k passwd_read

# Watch file writes
sudo auditctl -w /etc/shadow -p w -k shadow_write

# Watch file attribute changes
sudo auditctl -w /etc/ssh/sshd_config -p wa -k sshd_config

# Watch directory
sudo auditctl -w /etc/ -p wa -k etc_changes

# Permissions:
# r = read
# w = write
# x = execute
# a = attribute change
```

**Watch system calls**:
```bash
# Monitor all execve (command execution)
sudo auditctl -a always,exit -F arch=b64 -S execve -k command_exec

# Monitor file deletion
sudo auditctl -a always,exit -F arch=b64 -S unlink -S unlinkat -k file_deletion

# Monitor network connections
sudo auditctl -a always,exit -F arch=b64 -S socket -S connect -k network

# Monitor privilege escalation
sudo auditctl -a always,exit -F arch=b64 -S setuid -k setuid
```

### 10.5 Persistent Audit Rules

**Make rules persistent** (survive reboot):

```bash
# Edit rules file
sudo nano /etc/audit/rules.d/audit.rules
```

```bash
# Delete all existing rules
-D

# Buffer size
-b 8192

# Failure mode (0=silent, 1=printk, 2=panic)
-f 1

# Watch /etc/passwd
-w /etc/passwd -p wa -k passwd_changes

# Watch /etc/shadow
-w /etc/shadow -p wa -k shadow_changes

# Watch /etc/sudoers
-w /etc/sudoers -p wa -k sudoers_changes

# Watch sudo commands
-a always,exit -F arch=b64 -S execve -F exe=/usr/bin/sudo -k sudo_usage

# Watch SSH activity
-w /var/log/auth.log -p wa -k auth_log_changes

# Make configuration immutable (cannot be changed without reboot)
-e 2
```

**Load rules**:
```bash
sudo augenrules --load
sudo systemctl restart auditd
```

### 10.6 Searching Audit Logs

```bash
# Search by key
sudo ausearch -k passwd_changes

# Search by file
sudo ausearch -f /etc/shadow

# Search by UID
sudo ausearch -ua 1000

# Search by syscall
sudo ausearch -sc execve

# Search by time
sudo ausearch -ts today
sudo ausearch -ts 10:00:00
sudo ausearch --start today --end now

# Interpret results
sudo ausearch -i -k passwd_changes

# Generate report
sudo aureport

# Authentication report
sudo aureport -au

# File access report
sudo aureport -f

# Executable report
sudo aureport -x
```

---

## 11. Searching and Analyzing Logs

### 11.1 grep for Log Analysis

```bash
# Basic search
grep "pattern" /var/log/syslog

# Case-insensitive
grep -i "error" /var/log/syslog

# Show line numbers
grep -n "Failed" /var/log/auth.log

# Count matches
grep -c "SSH" /var/log/auth.log

# Show context (lines before/after)
grep -A 5 "error" /var/log/syslog  # 5 after
grep -B 5 "error" /var/log/syslog  # 5 before
grep -C 5 "error" /var/log/syslog  # 5 before and after

# Multiple patterns
grep -E "error|warning|fail" /var/log/syslog

# Invert match (exclude)
grep -v "DEBUG" /var/log/app.log

# Recursive search
grep -r "error" /var/log/
```

### 11.2 awk for Log Processing

```bash
# Print specific fields
awk '{print $1, $3, $5}' /var/log/auth.log

# Filter by field value
awk '/Failed password/ {print $1, $2, $3, $11}' /var/log/auth.log

# Count occurrences
awk '/Failed password/ {count++} END {print count}' /var/log/auth.log

# Sum values
awk '{sum+=$10} END {print sum}' /var/log/apache2/access.log
```

### 11.3 sed for Log Manipulation

```bash
# Extract lines between patterns
sed -n '/START/,/END/p' /var/log/syslog

# Delete specific lines
sed '/DEBUG/d' /var/log/app.log

# Replace text
sed 's/old/new/g' /var/log/file.log
```

### 11.4 tail and head

```bash
# Last 10 lines
tail /var/log/syslog

# Last 50 lines
tail -n 50 /var/log/syslog

# Follow live
tail -f /var/log/syslog

# Multiple files
tail -f /var/log/syslog /var/log/auth.log

# First 10 lines
head /var/log/syslog

# First 20 lines
head -n 20 /var/log/syslog
```

---

## 12. Detecting Security Events

### 12.1 Brute Force Detection

```bash
# Count failed SSH attempts per IP
grep "Failed password" /var/log/auth.log | awk '{print $(NF-3)}' | sort | uniq -c | sort -nr

# Find IPs with >10 failed attempts
grep "Failed password" /var/log/auth.log | awk '{print $(NF-3)}' | sort | uniq -c | awk '$1 > 10 {print $2}'

# Time-based failed attempts
grep "Failed password" /var/log/auth.log | grep "$(date +"%b %d")"
```

### 12.2 Successful Intrusion Detection

```bash
# Find accepted logins
grep "Accepted" /var/log/auth.log

# Logins from unusual IPs
grep "Accepted" /var/log/auth.log | awk '{print $(NF-3)}' | sort -u

# Login outside business hours
grep "Accepted" /var/log/auth.log | grep -E " (0[0-6]|2[2-3]):"

# Root logins (should be rare)
grep "Accepted.*root" /var/log/auth.log
```

### 12.3 Sudo Abuse Detection

```bash
# All sudo usage
grep "sudo:" /var/log/auth.log

# Sudo to root
grep "sudo:.*USER=root" /var/log/auth.log

# Failed sudo attempts
grep "sudo:.*NOT in sudoers" /var/log/auth.log

# Specific commands
grep "sudo:.*COMMAND=" /var/log/auth.log | grep -E "(nc|bash|sh|python)"
```

---

## 13. Failed Login Detection

### 13.1 SSH Failed Logins

```bash
# All failed password attempts
grep "Failed password" /var/log/auth.log

# Invalid users
grep "Invalid user" /var/log/auth.log

# Failed attempts with usernames and IPs
grep "Failed password" /var/log/auth.log | awk '{print $(NF-5), $(NF-3)}' | sort | uniq -c

# Top 10 attacked usernames
grep "Failed password" /var/log/auth.log | awk '{print $(NF-5)}' | sort | uniq -c | sort -rn | head -10

# Top 10 attacking IPs
grep "Failed password" /var/log/auth.log | awk '{print $(NF-3)}' | sort | uniq -c | sort -rn | head -10
```

### 13.2 Failed Local Logins

```bash
# Failed console logins
grep "authentication failure" /var/log/auth.log

# Failed su attempts
grep "su\[.*pam_unix.*authentication failure" /var/log/auth.log
```

### 13.3 Account Lockouts

```bash
# Check for locked accounts
grep "account locked" /var/log/auth.log

# Check PAM failures
grep "pam_faillock" /var/log/auth.log
```

---

## 14. Privilege Escalation Detection

### 14.1 SUID Binary Execution

```bash
# If auditd configured to watch SUID
sudo ausearch -sc execve -sv yes | grep -i suid

# Watch SUID file execution
sudo auditctl -a always,exit -F arch=b64 -S execve -F perm=u+s -k suid_exec
```

### 14.2 Capabilities Usage

```bash
# Watch capability changes
sudo auditctl -a always,exit -F arch=b64 -S capset -k capability_changes

# Search capability events
sudo ausearch -k capability_changes
```

### 14.3 Kernel Module Loading

```bash
# Watch module loading
sudo auditctl -w /sbin/insmod -p x -k module_insert
sudo auditctl -w /sbin/modprobe -p x -k module_insert

# Check for loaded modules
grep "insmod\|modprobe" /var/log/syslog
```

---

## 15. Covering Tracks (Anti-Forensics)

### 15.1 Understanding the Risks

**For OSCP**: You need to know these techniques to:
1. Understand what attackers do
2. Detect when someone has covered tracks
3. Know how to avoid detection during authorized pentests

**Ethical note**: Only use during authorized engagements.

### 15.2 Clearing Bash History

```bash
# Disable history for current session
unset HISTFILE

# Clear history
history -c

# Delete history file
rm ~/.bash_history

# Space before command (doesn't save to history)
 ls -la /etc/shadow
#^ Space character

# Set history size to 0
export HISTSIZE=0
export HISTFILESIZE=0

# Prevent specific commands from being logged
export HISTIGNORE="ls*:cat*:pwd*"
```

### 15.3 Clearing Specific Log Entries

```bash
# BAD: Delete entire log (obvious)
sudo rm /var/log/auth.log

# BETTER: Truncate log (still obvious)
sudo cat /dev/null > /var/log/auth.log

# BEST: Remove specific lines with sed
sudo sed -i '/192.168.1.100/d' /var/log/auth.log
sudo sed -i '/Failed password for root/d' /var/log/auth.log

# Remove line range
sudo sed -i '100,200d' /var/log/auth.log
```

### 15.4 Clearing wtmp and btmp

```bash
# Clear last login records
sudo cat /dev/null > /var/log/wtmp

# Clear failed login records
sudo cat /dev/null > /var/log/btmp

# Remove specific entry (harder)
# Would require binary editing tools
```

### 15.5 Clearing journald

```bash
# Clear all journals
sudo journalctl --rotate
sudo journalctl --vacuum-time=1s

# Clear specific unit logs
sudo rm -rf /var/log/journal/MACHINE_ID/
```

### 15.6 Log Tampering Detection

**Defenders look for**:
- Gaps in timestamps
- File modification times
- Missing log entries
- Truncated files
- Service restart messages

**Better evasion**:
- Don't delete, selectively edit
- Preserve timestamps: `touch -r oldfile newfile`
- Edit binary logs carefully (or don't touch them)

---

## 16. Log Integrity and Protection

### 16.1 Immutable Logs

**chattr +a** (append-only):

```bash
# Make log append-only (even root can't modify)
sudo chattr +a /var/log/auth.log

# Now can only append, not modify or delete
echo "test" >> /var/log/auth.log  # Works
cat /dev/null > /var/log/auth.log # Fails

# List attributes
lsattr /var/log/auth.log
# -----a-------e-- /var/log/auth.log

# Remove attribute
sudo chattr -a /var/log/auth.log
```

**chattr +i** (immutable):
```bash
# Make file completely immutable
sudo chattr +i /var/log/critical.log

# Cannot modify, delete, or append
# Must remove attribute first
sudo chattr -i /var/log/critical.log
```

### 16.2 Remote Logging

**Send logs to remote server** (attacker can't delete):

```bash
# rsyslog TCP forwarding
sudo nano /etc/rsyslog.conf

# Add:
*.* @@192.168.1.100:514

# Restart
sudo systemctl restart rsyslog
```

**Receive on log server**:
```bash
# Configure to listen
module(load="imtcp")
input(type="imtcp" port="514")

# Save to separate files per host
$template RemoteLogs,"/var/log/remote/%HOSTNAME%/%PROGRAMNAME%.log"
*.* ?RemoteLogs
& stop
```

### 16.3 Log File Permissions

```bash
# Secure log permissions
sudo chmod 640 /var/log/auth.log
sudo chown root:adm /var/log/auth.log

# Check permissions
ls -l /var/log/auth.log
# -rw-r----- 1 root adm 12345 Jan 24 10:30 /var/log/auth.log

# Audit log directory
find /var/log -type f -perm -004
```

### 16.4 Log Hashing

**Create checksums for integrity verification**:

```bash
# Create hash
sha256sum /var/log/auth.log > /secure/auth.log.sha256

# Verify later
sha256sum -c /secure/auth.log.sha256
```

---

## 17. Centralized Logging

### 17.1 Why Centralize Logs?

**Benefits**:
- Attackers can't delete remote logs
- Correlation across multiple systems
- Easier searching and analysis
- Compliance requirements
- Retention policies

### 17.2 Syslog-based Centralization

**Already covered in section 16.2**

### 17.3 ELK Stack (Elasticsearch, Logstash, Kibana)

**Architecture**:
```
Servers → Logstash → Elasticsearch → Kibana
         (collect)    (store)      (visualize)
```

**Simple setup**:
```bash
# Install Elasticsearch
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
sudo apt install elasticsearch

# Install Logstash
sudo apt install logstash

# Install Kibana
sudo apt install kibana

# Start services
sudo systemctl start elasticsearch logstash kibana
```

### 17.4 Filebeat for Log Shipping

```bash
# Install Filebeat
sudo apt install filebeat

# Configure
sudo nano /etc/filebeat/filebeat.yml
```

```yaml
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/log/auth.log
    - /var/log/syslog

output.elasticsearch:
  hosts: ["192.168.1.100:9200"]
```

```bash
# Start
sudo systemctl start filebeat
```

---

## 18. Log Analysis Tools

### 18.1 logwatch

**Automated log analysis and reporting**:

```bash
# Install
sudo apt install logwatch

# Run for today
sudo logwatch --detail High --range today

# Run for yesterday
sudo logwatch --range yesterday

# Email report
sudo logwatch --mailto admin@example.com

# Specific service
sudo logwatch --service sshd
```

### 18.2 fail2ban

**Automatic IP banning based on log analysis**:

```bash
# Install
sudo apt install fail2ban

# Configure
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo nano /etc/fail2ban/jail.local
```

```ini
[sshd]
enabled = true
port = 22
filter = sshd
logpath = /var/log/auth.log
maxretry = 5
bantime = 3600
```

```bash
# Start
sudo systemctl start fail2ban

# Check status
sudo fail2ban-client status
sudo fail2ban-client status sshd

# Unban IP
sudo fail2ban-client set sshd unbanip 192.168.1.100
```

### 18.3 goaccess (Web Log Analyzer)

```bash
# Install
sudo apt install goaccess

# Analyze Apache logs
goaccess /var/log/apache2/access.log

# Real-time monitoring
goaccess /var/log/apache2/access.log --log-format=COMBINED

# HTML report
goaccess /var/log/apache2/access.log -o report.html
```

### 18.4 splunk (Enterprise SIEM)

**For enterprise environments** - not covered in detail here.

---

## 19. SIEM Basics

### 19.1 What Is a SIEM?

**SIEM (Security Information and Event Management)** centralizes logs from multiple sources for correlation, alerting, and compliance.

**Features**:
- Log collection from multiple sources
- Real-time monitoring
- Correlation rules
- Alerting
- Dashboards
- Forensics

### 19.2 Common SIEM Solutions

**Commercial**:
- Splunk
- QRadar (IBM)
- ArcSight (Micro Focus)
- LogRhythm

**Open Source**:
- ELK Stack (Elasticsearch, Logstash, Kibana)
- Graylog
- Wazuh
- OSSEC

### 19.3 SIEM Use Cases

**Detection scenarios**:
- Brute force attacks (multiple failed logins)
- Privilege escalation (sudo to root)
- Data exfiltration (large outbound transfers)
- Lateral movement (unusual network connections)
- Persistence (new cron jobs, services)

---

## 20. Hands-On Labs

### Lab 1: Log Exploration

**Objective**: Navigate and understand log files.

```bash
# View authentication logs
sudo tail -f /var/log/auth.log

# In another terminal, trigger events:
# SSH login
ssh localhost

# Sudo command
sudo ls /root

# Failed sudo
sudo su -

# Observe logs in first terminal
```

### Lab 2: journalctl Mastery

**Objective**: Search logs with journalctl.

```bash
# View all SSH logs
journalctl -u sshd

# View logs from last hour
journalctl --since "1 hour ago"

# Follow live
journalctl -f

# View errors only
journalctl -p err

# View specific user's actions
journalctl _UID=1000
```

### Lab 3: auditd File Watching

**Objective**: Monitor file access with auditd.

```bash
# Install auditd
sudo apt install auditd

# Watch /etc/passwd
sudo auditctl -w /etc/passwd -p war -k passwd_watch

# Trigger event
cat /etc/passwd
sudo vim /etc/passwd

# Search audit log
sudo ausearch -k passwd_watch
```

### Lab 4: Failed Login Analysis

**Objective**: Detect brute force attempts.

```bash
# Generate failed attempts
for i in {1..10}; do
    ssh baduser@localhost
done

# Analyze
grep "Failed password" /var/log/auth.log | wc -l
grep "Failed password" /var/log/auth.log | awk '{print $(NF-3)}' | sort | uniq -c
```

### Lab 5: Log Rotation Testing

**Objective**: Understand log rotation.

```bash
# Create test log
sudo touch /var/log/test.log

# Create logrotate config
sudo nano /etc/logrotate.d/test
```

```
/var/log/test.log {
    daily
    rotate 3
    compress
    missingok
    notifempty
}
```

```bash
# Test rotation
sudo logrotate -f /etc/logrotate.d/test

# Check results
ls -l /var/log/test.log*
```

---

## 21. Interview Questions and Answers

### Q1: What's the difference between syslog and journald?

**Answer**:
- **syslog/rsyslog**: Text-based logs in /var/log/, human-readable, easy to grep, but easy to tamper with
- **journald**: Binary logs, structured/indexed, harder to tamper, better metadata (UID/GID/SELinux), searchable by fields

Both can coexist - journald can forward to syslog.

---

### Q2: How would you find all failed SSH login attempts from the last week?

**Answer**:
```bash
# Using auth.log
grep "Failed password" /var/log/auth.log* | grep "$(date -d '7 days ago' +%b)"

# Using journalctl
journalctl -u sshd --since "7 days ago" | grep "Failed password"

# Count per IP
grep "Failed password" /var/log/auth.log | awk '{print $(NF-3)}' | sort | uniq -c | sort -rn
```

---

### Q3: Explain what auditd is and when you would use it.

**Answer**:
auditd is kernel-level auditing - it watches system calls and file access at the kernel level, so processes cannot bypass it. Unlike syslog where applications choose what to log, auditd sees everything the kernel sees.

**Use cases**:
- Compliance (PCI-DSS requires file integrity monitoring)
- Watch sensitive files (/etc/shadow, /etc/passwd)
- Track command execution
- Forensics after breach
- Detect privilege escalation

---

### Q4: How do you make a log file append-only to prevent tampering?

**Answer**:
```bash
sudo chattr +a /var/log/auth.log
```

The +a attribute makes the file append-only - even root cannot modify or delete it, only append. To change it back, use `chattr -a`.

---

### Q5: What is log rotation and why is it important?

**Answer**:
Log rotation automatically archives old logs and compresses them to save disk space. Without it, logs would fill up the disk.

**logrotate** handles this - typically:
- Renames auth.log → auth.log.1
- Compresses auth.log.1 → auth.log.2.gz
- Deletes after N days/rotations
- Notifies services to start new log file

---

### Q6: How would you detect if someone has cleared the bash history?

**Answer**:
Signs of cleared history:
- Empty .bash_history despite active session
- .bash_history has recent modification time but few entries
- User has HISTFILE unset or HISTSIZE=0
- Check /home/*/.bash_history sizes: `find /home -name .bash_history -exec wc -l {} \;`

Also check: journalctl for executed commands (if logging enabled), auditd logs for execve, command logs in /var/log/auth.log (sudo commands)

---

### Q7: What logs would you check to investigate a suspected SSH brute force attack?

**Answer**:
1. `/var/log/auth.log` (Debian) or `/var/log/secure` (RedHat)
   - Search for "Failed password"
   - Look for unusual IPs, times, usernames
2. `journalctl -u sshd`
3. `sudo lastb` - failed login attempts
4. `sudo ausearch -m USER_LOGIN` - auditd SSH events
5. Check firewall logs for blocked IPs

Commands:
```bash
grep "Failed password" /var/log/auth.log | awk '{print $(NF-3)}' | sort | uniq -c | sort -rn
```

---

### Q8: How do you set up remote logging and why is it important?

**Answer**:
Remote logging sends logs to a separate server where attackers cannot delete them.

**Setup**:
```bash
# On client (edit /etc/rsyslog.conf):
*.* @@logserver:514

# On server:
# Enable TCP input
module(load="imtcp")
input(type="imtcp" port="514")
```

**Importance**: Attackers often clear local logs. Remote logs survive even if system is compromised.

---

### Q9: What would you look for in logs to detect privilege escalation?

**Answer**:
- Sudo usage: `grep "sudo:" /var/log/auth.log`
- SUID binary execution: auditd events
- User switching: `grep "su\[" /var/log/auth.log`
- Unusual processes as root: `grep "root" /var/log/syslog`
- New cron jobs: `grep "cron" /var/log/syslog`
- Service modifications: `journalctl -u systemd`
- Capabilities changes: `ausearch -sc capset`

---

### Q10: How would you cover your tracks after a pentest (ethically)?

**Answer** (for authorized engagements only):
1. Document what you did BEFORE covering tracks
2. Only clear YOUR activities, not all logs (to preserve evidence of real attacks)
3. Methods:
   - Clear bash history: `history -c; rm ~/.bash_history`
   - Remove specific log lines: `sed -i '/your-ip/d' /var/log/auth.log`
   - Clear wtmp/btmp entries (binary editing)
4. Better: Notify client and let them see your activities in logs as part of report

**Note**: In real pentest, showing log traces is PART of the deliverable - demonstrates detection gaps.

---

## 22. Summary and Next Steps

### 22.1 What We Covered

You've learned:
- ✅ Linux logging architecture
- ✅ syslog and rsyslog configuration
- ✅ systemd journald usage
- ✅ Important log file locations
- ✅ Authentication log analysis
- ✅ System and application logs
- ✅ Log rotation with logrotate
- ✅ auditd kernel-level auditing
- ✅ Log searching and analysis
- ✅ Security event detection
- ✅ Failed login detection
- ✅ Privilege escalation detection
- ✅ Anti-forensics techniques
- ✅ Log integrity protection
- ✅ Centralized logging
- ✅ Log analysis tools
- ✅ SIEM basics
- ✅ 20+ working examples

### 22.2 Critical Skills for OSCP

**For Attack**:
- Understand what gets logged
- Know how to cover tracks (ethical context)
- Avoid triggering fail2ban
- Be aware of auditd

**For Defense**:
- Analyze auth.log for failed logins
- Detect brute force patterns
- Use journalctl effectively
- Monitor sudo usage
- Protect log integrity

### 22.3 Essential Commands

```bash
# View logs
tail -f /var/log/auth.log
journalctl -u sshd -f
sudo ausearch -k key_name

# Search logs
grep "Failed password" /var/log/auth.log
journalctl --since "1 hour ago" -p err

# Last logins
last
sudo lastb

# Audit rules
sudo auditctl -w /etc/passwd -p wa -k passwd
sudo ausearch -k passwd
```

### 22.4 Next Steps

**Practice**:
- Set up centralized logging
- Configure auditd rules
- Analyze real logs
- Practice log analysis on HTB/OSCP labs

**Further Reading**:
- rsyslog documentation
- journald man pages
- auditd best practices
- SIEM concepts

**Next Chapter**: File 09 - Linux Privilege Escalation

---

**You now have comprehensive knowledge of Linux logging and auditing from both offensive and defensive perspectives. Master log analysis and you'll excel at both finding evidence and understanding what defenders see.**

---
*End of Chapter 08 - Linux Logging & Auditing*
