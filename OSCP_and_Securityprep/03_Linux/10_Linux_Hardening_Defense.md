# Linux Hardening & Defense: Building an Impenetrable Fortress - COMPREHENSIVE GUIDE

---
title: "Linux Hardening & Defense"
parent: "[[03_Linux]]"
tags:
  - linux
  - hardening
  - selinux
  - apparmor
  - defense
  - blue-team
  - security
  - compliance
  - cis-benchmarks
created: 2026-01-23
updated: 2026-01-24
---

> **Executive Summary**: A default Linux installation is like a house with unlocked doors and windows open. It's designed for convenience and compatibility, not security. Linux Hardening is the systematic process of securing a Linux system by reducing its attack surface, implementing security controls, and following industry best practices. This involves removing unnecessary software, locking down permissions, enforcing mandatory access controls, configuring firewalls, implementing intrusion detection, and maintaining continuous monitoring.
>
> This comprehensive guide covers everything from basic hardening steps to advanced enterprise security controls. You'll learn how to transform a vulnerable default installation into a hardened fortress that can withstand sophisticated attacks. We'll cover CIS Benchmarks, SELinux/AppArmor, kernel hardening, network security, file integrity monitoring, log analysis, compliance frameworks, and much more.
>
> Whether you're a system administrator responsible for production servers, a security engineer implementing defense-in-depth, or a penetration tester who needs to understand defensive measures, this guide provides the complete blueprint for Linux security hardening. Understanding both attack (File 09) and defense (this guide) makes you a complete security professional.
>
> **Why This Matters**: According to Verizon's Data Breach Investigations Report, over 80% of breaches involve exploitation of known vulnerabilities that could have been prevented through proper system hardening. The difference between a secure infrastructure and a compromised one often comes down to following systematic hardening procedures. This guide will teach you to think like a defender and build systems that resist even determined attackers.

---

## Table of Contents

1. [Introduction: Defense-in-Depth Philosophy](#1-introduction-defense-in-depth-philosophy)
2. [Pre-Hardening Assessment](#2-pre-hardening-assessment)
3. [Attack Surface Reduction](#3-attack-surface-reduction)
4. [User Account Security](#4-user-account-security)
5. [SSH Hardening](#5-ssh-hardening)
6. [Filesystem Hardening](#6-filesystem-hardening)
7. [Kernel Hardening](#7-kernel-hardening)
8. [Mandatory Access Control (MAC)](#8-mandatory-access-control-mac)
9. [Network Hardening](#9-network-hardening)
10. [Firewall Configuration](#10-firewall-configuration)
11. [Intrusion Detection and Prevention](#11-intrusion-detection-and-prevention)
12. [File Integrity Monitoring](#12-file-integrity-monitoring)
13. [Log Management and Monitoring](#13-log-management-and-monitoring)
14. [Patch Management](#14-patch-management)
15. [Service Hardening](#15-service-hardening)
16. [Container Security](#16-container-security)
17. [Compliance and Benchmarks](#17-compliance-and-benchmarks)
18. [Security Auditing Tools](#18-security-auditing-tools)
19. [Incident Response Preparation](#19-incident-response-preparation)
20. [Hands-On Labs](#20-hands-on-labs)
21. [Interview Questions and Answers](#21-interview-questions-and-answers)
22. [Summary and Checklist](#22-summary-and-checklist)

---

## 1. Introduction: Defense-in-Depth Philosophy

### 1.1 What is System Hardening?

**System Hardening** is the process of securing a system by reducing its attack surface and implementing security controls to protect against unauthorized access and attacks.

**Real-World Analogy**: Think of hardening like fortifying a castle:
- **Default Linux** = Medieval village with no walls (easy to attack)
- **Hardened Linux** = Fortress with walls, moats, guards, and restricted access (difficult to breach)

```
┌─────────────────────────────────────────────────────────────┐
│              DEFAULT vs HARDENED LINUX                      │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Default Installation (Vulnerable):                         │
│  ┌───────────────────────────────────┐                    │
│  │ ✗ Root login via SSH enabled      │                    │
│  │ ✗ Password authentication allowed  │                    │
│  │ ✗ Unnecessary services running     │                    │
│  │ ✗ World-writable files exist       │                    │
│  │ ✗ No firewall configured          │                    │
│  │ ✗ No intrusion detection          │                    │
│  │ ✗ Default passwords unchanged      │                    │
│  │ ✗ Old unpatched kernel            │                    │
│  │ Result: Compromised in minutes     │                    │
│  └───────────────────────────────────┘                    │
│                                                             │
│  Hardened Installation (Secure):                            │
│  ┌───────────────────────────────────┐                    │
│  │ ✓ Root login disabled             │                    │
│  │ ✓ Key-based authentication only    │                    │
│  │ ✓ Minimal services running         │                    │
│  │ ✓ Strict file permissions          │                    │
│  │ ✓ Firewall with default deny       │                    │
│  │ ✓ IDS/IPS monitoring traffic       │                    │
│  │ ✓ Strong password policies         │                    │
│  │ ✓ Kernel patched and hardened      │                    │
│  │ ✓ SELinux/AppArmor enforcing       │                    │
│  │ Result: Resists sophisticated attacks│                  │
│  └───────────────────────────────────┘                    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 1.2 Defense-in-Depth

Don't rely on a single security control. Layer multiple defenses.

```
┌─────────────────────────────────────────────────────────────┐
│              DEFENSE-IN-DEPTH LAYERS                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Layer 7: Policies and Procedures                          │
│    ↓ (Security policies, user training, incident response) │
│  Layer 6: Data Security                                     │
│    ↓ (Encryption, DLP, backups)                            │
│  Layer 5: Application Security                              │
│    ↓ (Input validation, secure coding, WAF)                │
│  Layer 4: Host Security                                     │
│    ↓ (Hardening, AV, HIDS, patching) ← THIS GUIDE          │
│  Layer 3: Network Security                                  │
│    ↓ (Firewalls, IPS, segmentation)                        │
│  Layer 2: Perimeter Security                                │
│    ↓ (Firewalls, VPN, DMZ)                                 │
│  Layer 1: Physical Security                                 │
│    (Locked server rooms, access controls)                   │
│                                                             │
│  Attacker must breach ALL layers to succeed                 │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**Principle**: If one layer fails, others still provide protection.

### 1.3 The Hardening Mindset

**Attack (Red Team)**: "How can I exploit this system?"
**Defense (Blue Team)**: "How can I make exploitation impossible or extremely difficult?"

**Key Principles**:
1. **Least Privilege**: Grant minimum necessary permissions
2. **Defense-in-Depth**: Multiple layers of security
3. **Fail Secure**: If something breaks, fail to a secure state
4. **Separation of Duties**: No single user has complete control
5. **Minimize Attack Surface**: Remove unnecessary components
6. **Audit Everything**: Log and monitor all security-relevant events

### 1.4 Hardening Process Overview

```
1. Assessment     → Baseline current security posture
2. Planning       → Define security requirements
3. Implementation → Apply hardening measures
4. Testing        → Verify controls work
5. Documentation  → Record all changes
6. Monitoring     → Continuous security monitoring
7. Review         → Regular security audits
8. Update         → Patch and improve continuously
```

---

## 2. Pre-Hardening Assessment

### 2.1 Inventory and Discovery

Before hardening, know what you're protecting.

```bash
# System information
hostnamectl
uname -a
cat /etc/os-release
lsb_release -a

# Installed packages
dpkg -l | wc -l                    # Debian/Ubuntu
rpm -qa | wc -l                    # RedHat/CentOS
apt list --installed               # Detailed list
yum list installed                 # RHEL

# Running services
systemctl list-units --type=service --state=running
systemctl list-unit-files --type=service --state=enabled

# Open ports
ss -tunlp
netstat -tunlp
lsof -i

# User accounts
cat /etc/passwd | wc -l
awk -F: '($3 >= 1000) {print $1}' /etc/passwd  # Real users

# Scheduled tasks
crontab -l
ls -la /etc/cron*
systemctl list-timers

# Installed compilers (security risk on production)
which gcc g++ cc
dpkg -l | grep compiler

# World-writable files (security risk)
find / -xdev -type f -perm -0002 2>/dev/null | wc -l

# SUID/SGID files
find / -xdev -type f -perm -4000 2>/dev/null | wc -l  # SUID
find / -xdev -type f -perm -2000 2>/dev/null | wc -l  # SGID

# Kernel modules
lsmod | wc -l
```

### 2.2 Baseline Security Audit

```bash
# Quick security check script
#!/bin/bash

echo "=== SECURITY BASELINE ASSESSMENT ==="
echo ""

echo "[*] System Information"
echo "Hostname: $(hostname)"
echo "OS: $(cat /etc/os-release | grep PRETTY_NAME | cut -d'"' -f2)"
echo "Kernel: $(uname -r)"
echo ""

echo "[*] User Accounts"
echo "Total accounts: $(cat /etc/passwd | wc -l)"
echo "UID 0 accounts: $(awk -F: '($3 == 0) {print $1}' /etc/passwd)"
echo "Users with empty passwords: $(awk -F: '($2 == "") {print $1}' /etc/shadow 2>/dev/null)"
echo ""

echo "[*] SSH Configuration"
grep "^PermitRootLogin" /etc/ssh/sshd_config
grep "^PasswordAuthentication" /etc/ssh/sshd_config
grep "^Port" /etc/ssh/sshd_config
echo ""

echo "[*] Firewall Status"
systemctl status ufw 2>/dev/null || systemctl status firewalld 2>/dev/null
echo ""

echo "[*] SELinux/AppArmor Status"
getenforce 2>/dev/null || aa-status 2>/dev/null
echo ""

echo "[*] Services Running"
echo "Total running services: $(systemctl list-units --type=service --state=running | wc -l)"
echo ""

echo "[*] Open Ports"
ss -tunlp | grep LISTEN
echo ""

echo "[*] World-Writable Files"
echo "Count: $(find / -xdev -type f -perm -0002 2>/dev/null | wc -l)"
echo ""

echo "[*] SUID Binaries"
echo "Count: $(find / -xdev -type f -perm -4000 2>/dev/null | wc -l)"
echo ""

echo "=== ASSESSMENT COMPLETE ==="
```

### 2.3 Using Lynis for Baseline

**Lynis** is the industry-standard security auditing tool.

```bash
# Install Lynis
apt-get install lynis        # Debian/Ubuntu
yum install lynis            # RHEL/CentOS
# Or download latest
git clone https://github.com/CISOfy/lynis

# Run full audit
sudo lynis audit system

# Run and save report
sudo lynis audit system --log-file /var/log/lynis-initial.log

# Check your score
cat /var/log/lynis.log | grep "Hardening index"
# Example output: Hardening index : 57 [############        ]
# Goal: Get above 80
```

**Lynis Audit Areas**:
- Boot and services
- Kernel
- Memory and processes
- Users, groups, and authentication
- Shells
- File systems
- Storage
- NFS
- Software: name services, malware, file integrity
- Network
- Printers and spools
- Software: e-mail and messaging
- Software: firewalls
- Software: webserver
- SSH
- SNMP
- Databases
- LDAP services
- Software: PHP
- Squid
- Logging and files
- Insecure services
- Banners and identification
- Scheduled tasks
- Accounting
- Time and synchronization
- Cryptography
- Virtualization
- Security frameworks
- Software: file integrity
- Software: malware scanner

### 2.4 CIS Benchmark Assessment

```bash
# Download CIS-CAT (Center for Internet Security Configuration Assessment Tool)
# Free version available at: https://www.cisecurity.org/

# Or use manual CIS benchmark PDFs
# Available at: https://www.cisecurity.org/cis-benchmarks/

# Key CIS Benchmark areas for Linux:
# 1. Initial Setup
# 2. Services
# 3. Network Configuration
# 4. Logging and Auditing
# 5. Access, Authentication and Authorization
# 6. System Maintenance
```

---

## 3. Attack Surface Reduction

### 3.1 The Principle

**Attack Surface** = All possible entry points for an attacker

**Minimize it by**:
- Removing unnecessary software
- Disabling unused services
- Closing unnecessary ports
- Removing unnecessary users

### 3.2 Remove Unnecessary Packages

```bash
# List installed packages
dpkg -l                          # Debian/Ubuntu
rpm -qa                          # RHEL/CentOS

# Remove unused packages
apt-get purge telnet
apt-get purge ftp
apt-get purge rsh-client rsh-redone-client
apt-get purge talk
apt-get purge nis
apt-get purge tftp
apt-get autoremove              # Remove dependencies

# Remove compilers on production servers
apt-get purge gcc g++ make
apt-get purge build-essential
```

**Packages to Remove on Production Servers**:
- `telnet` - Insecure remote access
- `rsh*` - Insecure remote shell
- `ftp` - Insecure file transfer
- `tftp` - Trivial FTP (very insecure)
- `talk` - Insecure messaging
- `nis` - Network Information Service (outdated)
- `gcc`, `g++`, `make` - Compilers (let attackers compile exploits)
- `xinetd` - Super-server (legacy)

### 3.3 Disable Unnecessary Services

```bash
# List all services
systemctl list-unit-files --type=service

# List running services
systemctl list-units --type=service --state=running

# Disable and stop unnecessary services
systemctl stop avahi-daemon
systemctl disable avahi-daemon

systemctl stop cups
systemctl disable cups

systemctl stop bluetooth
systemctl disable bluetooth

# Verify disabled
systemctl status avahi-daemon
```

**Services to Disable on Most Servers**:
- `avahi-daemon` - Zeroconf networking (rarely needed on servers)
- `cups` - Printing system (not needed on servers)
- `bluetooth` - Bluetooth (not needed on servers)
- `ModemManager` - Modem management (not needed)
- `postfix`/`sendmail` - Mail server (unless actually used)
- `nfs-server` - NFS server (unless sharing files)
- `rpcbind` - RPC portmapper (legacy, insecure)

**Check for listening services**:
```bash
# What's listening on which ports?
ss -tulpn

# Investigate unexpected listeners
lsof -i :PORT_NUMBER

# Example output analysis:
# tcp   LISTEN  0.0.0.0:23    - Telnet? Should not be running!
# tcp   LISTEN  0.0.0.0:21    - FTP? Should not be running!
# tcp   LISTEN  0.0.0.0:22    - SSH? Expected
# tcp   LISTEN  0.0.0.0:80    - HTTP? Expected if web server
```

### 3.4 Remove Unnecessary Users and Groups

```bash
# List all users
cat /etc/passwd

# List users with login shells
cat /etc/passwd | grep -v "/nologin" | grep -v "/false"

# List users with UID 0 (should only be root!)
awk -F: '($3 == 0) {print}' /etc/passwd

# Remove unnecessary user
userdel -r username              # -r removes home directory

# Lock unused system accounts
usermod -L username
usermod -s /usr/sbin/nologin username

# List all groups
cat /etc/group

# Remove unnecessary group
groupdel groupname
```

**Users/Groups to Audit**:
- Users with UID 0 (should ONLY be root)
- Users with empty passwords
- Inactive user accounts
- System accounts with login shells
- Users in sensitive groups (sudo, wheel, docker)

### 3.5 Minimize Installed Software

```bash
# Identify large/unused packages
dpkg-query -W --showformat='${Installed-Size}\t${Package}\n' | sort -rn | head -20

# Check package usage
apt-mark showmanual              # Manually installed
apt-mark showauto                # Auto-installed

# Review and remove
apt-get purge <package>
apt-get autoremove
apt-get autoclean
```

---

## 4. User Account Security

### 4.1 Strong Password Policies

```bash
# Install password quality checking
apt-get install libpam-pwquality    # Debian/Ubuntu
yum install libpwquality            # RHEL/CentOS

# Configure password requirements
vim /etc/security/pwquality.conf

# Recommended settings:
minlen = 14                      # Minimum length
dcredit = -1                     # At least 1 digit
ucredit = -1                     # At least 1 uppercase
lcredit = -1                     # At least 1 lowercase
ocredit = -1                     # At least 1 special char
minclass = 4                     # All 4 character classes
maxrepeat = 2                    # Max consecutive repeated chars
maxclassrepeat = 3               # Max consecutive same class
gecoscheck = 1                   # Don't allow username in password
dictcheck = 1                    # Check against dictionary
usercheck = 1                    # Check against username
enforcing = 1                    # Enforce for root too
```

### 4.2 Password Aging

```bash
# Set password aging for existing user
chage -M 90 username             # Max 90 days
chage -m 7 username              # Min 7 days between changes
chage -W 14 username             # Warn 14 days before expiry

# View settings
chage -l username

# Set defaults for new users
vim /etc/login.defs
PASS_MAX_DAYS   90
PASS_MIN_DAYS   7
PASS_WARN_AGE   14
PASS_MIN_LEN    14
```

### 4.3 Account Lockout

```bash
# Configure account lockout after failed login attempts
vim /etc/pam.d/common-auth       # Debian/Ubuntu
vim /etc/pam.d/system-auth       # RHEL/CentOS

# Add this line:
auth required pam_tally2.so deny=5 unlock_time=900 onerr=fail audit

# This means:
# - Lock account after 5 failed attempts
# - Unlock after 900 seconds (15 minutes)
# - Log failures

# View failed login attempts
pam_tally2 --user=username

# Manually unlock
pam_tally2 --user=username --reset
```

### 4.4 Disable Root Login (Console)

```bash
# Prevent root login on console (except single-user mode)
vim /etc/securetty

# Comment out or remove all lines except:
# console

# Better: Require sudo for root access
# Don't allow direct root login at all
```

### 4.5 Sudo Configuration Hardening

```bash
# Edit sudoers securely
visudo

# Good practices:
# 1. Require password
Defaults    timestamp_timeout=5   # Re-prompt after 5 min

# 2. Log all sudo commands
Defaults    logfile="/var/log/sudo.log"
Defaults    log_input, log_output

# 3. Require TTY (prevents some attacks)
Defaults    requiretty

# 4. Use groups, not individual users
%wheel  ALL=(ALL)       ALL       # RHEL
%sudo   ALL=(ALL:ALL)   ALL       # Debian

# 5. Limit sudo to specific commands
user    ALL=(ALL)   NOPASSWD: /usr/bin/systemctl restart nginx
```

### 4.6 Monitoring User Activity

```bash
# Who is logged in
who
w
last                            # Login history
lastb                           # Failed login attempts

# Monitor user sessions
pam_tty_audit                   # TTY auditing

# Log user commands
# Enable bash history timestamps
echo 'export HISTTIMEFORMAT="%F %T "' >> /etc/profile
echo 'export HISTSIZE=10000' >> /etc/profile
echo 'export HISTFILESIZE=10000' >> /etc/profile
```

---

## 5. SSH Hardening

### 5.1 Why SSH Hardening Matters

SSH is the primary remote access method for Linux servers. **It's the front door to your castle.**

**Common SSH Attacks**:
- Brute force password attacks
- Dictionary attacks
- Exploiting weak passwords
- Man-in-the-middle attacks
- Exploiting old SSH vulnerabilities

### 5.2 SSH Hardening Configuration

```bash
# Backup original config
cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup

# Edit SSH configuration
vim /etc/ssh/sshd_config

# ===== CRITICAL CHANGES =====

# 1. Disable root login
PermitRootLogin no

# 2. Disable password authentication (keys only)
PasswordAuthentication no
ChallengeResponseAuthentication no
UsePAM yes

# 3. Use SSH Protocol 2 only
Protocol 2

# 4. Change default port (security through obscurity)
Port 2222

# 5. Limit user access
AllowUsers alice bob
# Or use AllowGroups
AllowGroups sshusers

# 6. Set idle timeout
ClientAliveInterval 300          # 5 minutes
ClientAliveCountMax 2            # Disconnect after 2 intervals

# 7. Disable X11 forwarding (unless needed)
X11Forwarding no

# 8. Disable TCP forwarding (unless needed)
AllowTcpForwarding no

# 9. Limit authentication attempts
MaxAuthTries 3
MaxSessions 2

# 10. Use strong ciphers and MACs
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com,aes256-ctr,aes192-ctr,aes128-ctr
MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com,hmac-sha2-512,hmac-sha2-256
KexAlgorithms curve25519-sha256,curve25519-sha256@libssh.org,diffie-hellman-group-exchange-sha256

# 11. Disable empty passwords
PermitEmptyPasswords no

# 12. Disable host-based authentication
HostbasedAuthentication no
IgnoreRhosts yes

# 13. Log more verbosely
LogLevel VERBOSE

# 14. Use privilege separation
UsePrivilegeSeparation sandbox

# 15. Disable unused authentication methods
PubkeyAuthentication yes         # Keep this!
PasswordAuthentication no
KerberosAuthentication no
GSSAPIAuthentication no

# Restart SSH service
systemctl restart sshd

# Test from another session BEFORE closing current one!
# ssh -p 2222 user@localhost
```

### 5.3 SSH Key-Based Authentication Setup

```bash
# On client machine, generate SSH key
ssh-keygen -t ed25519 -C "user@hostname"
# Or for older systems:
ssh-keygen -t rsa -b 4096 -C "user@hostname"

# Copy public key to server
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@server
# Or manually:
cat ~/.ssh/id_ed25519.pub | ssh user@server "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"

# On server, set correct permissions
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys

# Test key-based login
ssh user@server

# Once working, disable password authentication in sshd_config
```

### 5.4 Two-Factor Authentication (2FA) for SSH

```bash
# Install Google Authenticator
apt-get install libpam-google-authenticator    # Debian/Ubuntu
yum install google-authenticator               # RHEL/CentOS

# Setup for user
google-authenticator
# Answer prompts:
# - Time-based tokens: Yes
# - Update .google_authenticator: Yes
# - Disallow reuse: Yes
# - Rate limiting: Yes

# Configure PAM
vim /etc/pam.d/sshd
# Add at the end:
auth required pam_google_authenticator.so

# Configure SSH
vim /etc/ssh/sshd_config
ChallengeResponseAuthentication yes
AuthenticationMethods publickey,keyboard-interactive

# Restart SSH
systemctl restart sshd

# Now login requires:
# 1. SSH private key
# 2. 2FA code from Google Authenticator app
```

### 5.5 Fail2Ban for SSH Protection

```bash
# Install Fail2Ban
apt-get install fail2ban         # Debian/Ubuntu
yum install fail2ban             # RHEL/CentOS

# Configure SSH jail
vim /etc/fail2ban/jail.local

[sshd]
enabled = true
port = ssh,2222                  # Add custom port if changed
filter = sshd
logpath = /var/log/auth.log      # Or /var/log/secure on RHEL
maxretry = 3
bantime = 3600                   # Ban for 1 hour
findtime = 600                   # Within 10 minutes

# Start and enable
systemctl start fail2ban
systemctl enable fail2ban

# Check status
fail2ban-client status
fail2ban-client status sshd

# View banned IPs
fail2ban-client get sshd banned

# Manually ban IP
fail2ban-client set sshd banip 192.168.1.100

# Manually unban IP
fail2ban-client set sshd unbanip 192.168.1.100
```

### 5.6 SSH Security Audit

```bash
# Test SSH configuration
sshd -t

# Check for weak keys
ssh-keyscan -t rsa,dsa,ecdsa,ed25519 localhost

# Audit with ssh-audit
git clone https://github.com/jtesta/ssh-audit.git
python3 ssh-audit/ssh-audit.py localhost

# Review authentication logs
grep "Failed password" /var/log/auth.log | tail -20
grep "Accepted publickey" /var/log/auth.log | tail -20
```

---

## 6. Filesystem Hardening

### 6.1 Partition Security with Mount Options

```bash
# View current mounts
mount
cat /etc/fstab

# Secure mount options
# /tmp - Prevent execution and SUID
tmpfs  /tmp  tmpfs  defaults,noexec,nosuid,nodev  0 0

# /var/tmp - Same as /tmp
tmpfs  /var/tmp  tmpfs  defaults,noexec,nosuid,nodev  0 0

# /home - Prevent SUID and device files
/dev/sda3  /home  ext4  defaults,nosuid,nodev  0 2

# /var - Prevent execution (web uploads can't be executed)
/dev/sda4  /var  ext4  defaults,nosuid  0 2

# Apply changes
mount -o remount /tmp
mount -o remount /var/tmp
mount -o remount /home

# Verify
mount | grep -E "tmp|home|var"
```

**Mount Options Explained**:
- `noexec` - Cannot execute binaries (prevents `/tmp` exploits)
- `nosuid` - SUID/SGID bits ignored (prevents privilege escalation)
- `nodev` - Device files ignored (prevents device-based attacks)
- `ro` - Read-only (for static content)

### 6.2 File Permission Hardening

```bash
# Set secure umask (default permissions for new files)
vim /etc/profile
vim /etc/bash.bashrc
# Add:
umask 027                        # Owner: rwx, Group: rx, Other: none

# Secure important files
chmod 644 /etc/passwd
chmod 600 /etc/shadow
chmod 644 /etc/group
chmod 600 /etc/gshadow
chmod 600 /boot/grub/grub.cfg
chmod 600 /etc/ssh/sshd_config

# Find and fix world-writable files
find / -xdev -type f -perm -0002 -exec chmod o-w {} \;

# Find and fix world-writable directories (keep sticky bit)
find / -xdev -type d -perm -0002 -exec chmod o-w {} \;

# Find files with no owner (orphaned files)
find / -xdev -nouser -o -nogroup
```

### 6.3 Disable Core Dumps

Core dumps can contain sensitive data like passwords.

```bash
# Disable core dumps system-wide
vim /etc/security/limits.conf
# Add:
* hard core 0
* soft core 0

# Disable via sysctl
vim /etc/sysctl.conf
# Add:
kernel.core_pattern = /dev/null
fs.suid_dumpable = 0

# Apply
sysctl -p

# Verify
ulimit -c                        # Should show 0
```

### 6.4 Sticky Bit on World-Writable Directories

```bash
# Apply sticky bit to /tmp (only owner can delete files)
chmod +t /tmp
chmod +t /var/tmp

# Verify
ls -ld /tmp
# drwxrwxrwt (t at the end = sticky bit)

# Find world-writable directories without sticky bit
find / -xdev -type d -perm -0002 ! -perm -1000
```

---

## 7. Kernel Hardening

### 7.1 Sysctl Hardening

```bash
# Backup original
cp /etc/sysctl.conf /etc/sysctl.conf.backup

# Edit kernel parameters
vim /etc/sysctl.conf

# === Network Security ===

# Disable IP forwarding (unless router/gateway)
net.ipv4.ip_forward = 0
net.ipv6.conf.all.forwarding = 0

# Disable source routing
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.conf.default.accept_source_route = 0
net.ipv6.conf.all.accept_source_route = 0
net.ipv6.conf.default.accept_source_route = 0

# Disable ICMP redirects
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0
net.ipv4.conf.all.secure_redirects = 0
net.ipv4.conf.default.secure_redirects = 0
net.ipv6.conf.all.accept_redirects = 0
net.ipv6.conf.default.accept_redirects = 0

# Do not send ICMP redirects
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.send_redirects = 0

# Enable TCP SYN cookies (SYN flood protection)
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_max_syn_backlog = 2048
net.ipv4.tcp_synack_retries = 2
net.ipv4.tcp_syn_retries = 5

# Ignore ICMP ping requests
net.ipv4.icmp_echo_ignore_all = 1

# Ignore broadcast pings (Smurf attack)
net.ipv4.icmp_echo_ignore_broadcasts = 1

# Ignore bogus ICMP error responses
net.ipv4.icmp_ignore_bogus_error_responses = 1

# Enable reverse path filtering (prevent IP spoofing)
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.rp_filter = 1

# Log martian packets (impossible IPs)
net.ipv4.conf.all.log_martians = 1
net.ipv4.conf.default.log_martians = 1

# === Kernel Security ===

# Restrict kernel pointer exposure
kernel.kptr_restrict = 2

# Restrict dmesg access
kernel.dmesg_restrict = 1

# Disable kernel module loading after boot
# kernel.modules_disabled = 1      # CAREFUL: Can't load modules after this!

# Restrict access to kernel logs
kernel.dmesg_restrict = 1

# Enable ASLR (Address Space Layout Randomization)
kernel.randomize_va_space = 2

# Restrict ptrace (prevents process debugging)
kernel.yama.ptrace_scope = 2

# === Filesystem Security ===

# Disable core dumps for SUID binaries
fs.suid_dumpable = 0

# Protect hard links
fs.protected_hardlinks = 1

# Protect symbolic links
fs.protected_symlinks = 1

# Increase inotify limits (for security tools)
fs.inotify.max_user_watches = 524288

# Apply settings
sysctl -p

# Verify
sysctl -a | grep -E "ip_forward|syncookies|rp_filter"
```

### 7.2 Kernel Module Blacklisting

```bash
# Disable unused protocols and filesystems
vim /etc/modprobe.d/blacklist.conf

# Disable unused network protocols
install dccp /bin/true
install sctp /bin/true
install rds /bin/true
install tipc /bin/true

# Disable unused filesystems
install cramfs /bin/true
install freevxfs /bin/true
install jffs2 /bin/true
install hfs /bin/true
install hfsplus /bin/true
install udf /bin/true

# Disable USB storage (if not needed)
install usb-storage /bin/true

# Disable firewire (DMA attack vector)
install firewire-core /bin/true

# Apply (reboot required)
update-initramfs -u
```

### 7.3 Enable Kernel Security Features

```bash
# Check available security features
cat /boot/config-$(uname -r) | grep -E "SECURITY|SELINUX|APPARMOR|STACKPROTECTOR"

# Should see:
# CONFIG_SECURITY=y
# CONFIG_SECURITY_SELINUX=y or CONFIG_SECURITY_APPARMOR=y
# CONFIG_CC_STACKPROTECTOR=y
# CONFIG_DEBUG_RODATA=y
```

---

## 8. Mandatory Access Control (MAC)

### 8.1 Understanding MAC vs DAC

**DAC (Discretionary Access Control)**:
- Traditional Linux permissions (chmod, chown)
- File owner decides permissions
- Users can change permissions on their files
- Vulnerable to privilege escalation

**MAC (Mandatory Access Control)**:
- System-wide security policy
- Even root must obey MAC rules
- Users cannot override MAC policies
- Defense-in-depth protection

```
┌─────────────────────────────────────────────────────────────┐
│                  DAC vs MAC                                 │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  DAC (chmod/chown):                                         │
│  User creates file → User sets permissions                  │
│  Problem: User can make mistake (chmod 777)                 │
│                                                             │
│  MAC (SELinux/AppArmor):                                    │
│  System policy → Enforces regardless of DAC                 │
│  Benefit: Even if DAC is wrong, MAC protects                │
│                                                             │
│  Example:                                                   │
│  Apache process compromised                                 │
│  DAC: Can read any world-readable file                      │
│  MAC: Can ONLY access files labeled for web content         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 8.2 AppArmor (Ubuntu/Debian Default)

#### 8.2.1 AppArmor Basics

```bash
# Check AppArmor status
sudo aa-status

# AppArmor modes:
# - Enforce: Violations are blocked and logged
# - Complain: Violations are logged but allowed
# - Disabled: Profile not loaded

# View current profiles
ls /etc/apparmor.d/

# Load a profile
sudo apparmor_parser -r /etc/apparmor.d/usr.bin.firefox

# Set profile to enforce mode
sudo aa-enforce /etc/apparmor.d/usr.bin.firefox

# Set profile to complain mode
sudo aa-complain /etc/apparmor.d/usr.bin.firefox

# Disable a profile
sudo ln -s /etc/apparmor.d/usr.bin.firefox /etc/apparmor.d/disable/
sudo apparmor_parser -R /etc/apparmor.d/usr.bin.firefox
```

#### 8.2.2 Creating AppArmor Profile

```bash
# Generate profile automatically
sudo aa-genprof /usr/bin/myapp

# This will:
# 1. Put program in complain mode
# 2. Run your program
# 3. Monitor what files it accesses
# 4. Generate profile based on behavior

# Example manual profile
sudo vim /etc/apparmor.d/usr.bin.myapp

#include <tunables/global>

/usr/bin/myapp {
  #include <abstractions/base>
  #include <abstractions/nameservice>

  # Capabilities
  capability dac_override,
  capability setuid,

  # Files
  /usr/bin/myapp mr,
  /etc/myapp/** r,
  /var/log/myapp/* w,
  /var/lib/myapp/** rw,

  # Network
  network inet stream,
  network inet6 stream,

  # Deny all other access
  deny /** wx,
}

# Load profile
sudo apparmor_parser -r /etc/apparmor.d/usr.bin.myapp

# Set to enforce
sudo aa-enforce /etc/apparmor.d/usr.bin.myapp
```

#### 8.2.3 AppArmor for Common Services

```bash
# Apache/Nginx
sudo aa-enforce /etc/apparmor.d/usr.sbin.apache2
sudo aa-enforce /etc/apparmor.d/usr.sbin.nginx

# MySQL
sudo aa-enforce /etc/apparmor.d/usr.sbin.mysqld

# Named (DNS)
sudo aa-enforce /etc/apparmor.d/usr.sbin.named

# Check which profiles are enforcing
sudo aa-status | grep enforce
```

### 8.3 SELinux (RHEL/CentOS Default)

#### 8.3.1 SELinux Basics

```bash
# Check SELinux status
getenforce
# Output: Enforcing | Permissive | Disabled

# Detailed status
sestatus

# SELinux modes:
# Enforcing: Policy is enforced
# Permissive: Policy violations logged but allowed
# Disabled: SELinux is off

# Change mode temporarily
setenforce 0          # Permissive
setenforce 1          # Enforcing

# Change mode permanently
vim /etc/selinux/config
SELINUX=enforcing
# Reboot required
```

#### 8.3.2 SELinux Contexts

```bash
# View file context
ls -Z /var/www/html/
# -rw-r--r--. root root system_u:object_r:httpd_sys_content_t:s0 index.html

# Format: user:role:type:level
# Most important: type (httpd_sys_content_t)

# View process context
ps -eZ | grep httpd
# system_u:system_r:httpd_t:s0 1234 ? 00:00:00 httpd

# Change file context
chcon -t httpd_sys_content_t /var/www/html/newfile.html

# Restore default context
restorecon -v /var/www/html/newfile.html

# Restore recursively
restorecon -Rv /var/www/html/
```

#### 8.3.3 SELinux Booleans

```bash
# List all booleans
getsebool -a

# Check specific boolean
getsebool httpd_can_network_connect
# httpd_can_network_connect --> off

# Allow Apache to make network connections
setsebool -P httpd_can_network_connect on
# -P makes it persistent

# Common booleans:
httpd_can_network_connect        # Allow web server to connect to network
httpd_enable_homedirs            # Allow web server to serve home directories
ftpd_full_access                 # Allow FTP full filesystem access
allow_ftpd_anon_write            # Allow anonymous FTP uploads
```

#### 8.3.4 SELinux Troubleshooting

```bash
# Check for SELinux denials
ausearch -m avc -ts recent
tail /var/log/audit/audit.log | grep denied

# Use audit2why to understand
ausearch -m avc -ts recent | audit2why

# Generate policy module from denial
ausearch -m avc -ts recent | audit2allow -M mypolicy
semodule -i mypolicy.pp          # Install policy module

# View installed modules
semodule -l
```

---

## 9. Network Hardening

### 9.1 Disable IPv6 (If Not Used)

```bash
# Disable IPv6 via sysctl
vim /etc/sysctl.conf
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1

# Apply
sysctl -p

# Verify
cat /proc/sys/net/ipv6/conf/all/disable_ipv6
# Should output: 1
```

### 9.2 TCP Wrappers

```bash
# Configure allowed hosts
vim /etc/hosts.allow
sshd: 192.168.1.0/24
sshd: 10.0.0.0/8

# Deny all others
vim /etc/hosts.deny
ALL: ALL

# Test
tcpdmatch sshd 192.168.1.100
```

### 9.3 Disable Unnecessary Network Services

```bash
# List listening ports
ss -tunlp
netstat -tulpn

# Disable unnecessary services
systemctl stop rpcbind
systemctl disable rpcbind
```

---

## 10. Firewall Configuration

### 10.1 UFW (Uncomplicated Firewall - Ubuntu)

```bash
# Install
apt-get install ufw

# Enable
ufw enable

# Default policies
ufw default deny incoming
ufw default allow outgoing

# Allow SSH (BEFORE enabling firewall!)
ufw allow 22/tcp
# Or specific IP
ufw allow from 192.168.1.100 to any port 22

# Allow HTTP/HTTPS
ufw allow 80/tcp
ufw allow 443/tcp

# Allow from specific subnet
ufw allow from 192.168.1.0/24

# Delete rule
ufw delete allow 80/tcp

# View rules
ufw status numbered
ufw status verbose

# Disable
ufw disable
```

### 10.2 Firewalld (RHEL/CentOS)

```bash
# Install
yum install firewalld

# Start and enable
systemctl start firewalld
systemctl enable firewalld

# Check status
firewall-cmd --state

# Default zone
firewall-cmd --get-default-zone

# Add service
firewall-cmd --zone=public --add-service=ssh --permanent
firewall-cmd --zone=public --add-service=http --permanent
firewall-cmd --zone=public --add-service=https --permanent

# Add port
firewall-cmd --zone=public --add-port=8080/tcp --permanent

# Add source
firewall-cmd --zone=public --add-source=192.168.1.0/24 --permanent

# Reload
firewall-cmd --reload

# List rules
firewall-cmd --list-all
```

### 10.3 iptables (Advanced)

```bash
# Default policies (DROP all, allow only explicitly)
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT

# Allow loopback
iptables -A INPUT -i lo -j ACCEPT

# Allow established connections
iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# Allow SSH
iptables -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT

# Allow HTTP/HTTPS
iptables -A INPUT -p tcp --dport 80 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT

# Rate limit SSH (prevent brute force)
iptables -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW -m recent --set
iptables -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW -m recent --update --seconds 60 --hitcount 4 -j DROP

# Save rules
iptables-save > /etc/iptables/rules.v4

# Restore on boot
apt-get install iptables-persistent
```

---

## 11. Intrusion Detection and Prevention

### 11.1 Fail2Ban

Already covered in SSH section, but here's expanded coverage:

```bash
# Install
apt-get install fail2ban

# Configure
vim /etc/fail2ban/jail.local

[DEFAULT]
bantime = 3600
findtime = 600
maxretry = 3
destemail = admin@example.com
sendername = Fail2Ban
action = %(action_mwl)s

[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 3

[apache-auth]
enabled = true
port = http,https
filter = apache-auth
logpath = /var/log/apache2/*error.log
maxretry = 3

[nginx-http-auth]
enabled = true
filter = nginx-http-auth
port = http,https
logpath = /var/log/nginx/error.log

# Start
systemctl start fail2ban
systemctl enable fail2ban

# Monitor
fail2ban-client status
tail -f /var/log/fail2ban.log
```

### 11.2 AIDE (Advanced Intrusion Detection Environment)

```bash
# Install
apt-get install aide

# Initialize database
aideinit

# Move database
mv /var/lib/aide/aide.db.new /var/lib/aide/aide.db

# Check for changes
aide --check

# Update database after legitimate changes
aide --update
mv /var/lib/aide/aide.db.new /var/lib/aide/aide.db

# Automate with cron
echo "0 5 * * * root /usr/bin/aide --check | mail -s 'AIDE Report' admin@example.com" >> /etc/crontab
```

### 11.3 Tripwire

```bash
# Install
apt-get install tripwire

# Configure during installation
# Set site and local passphrases

# Initialize database
tripwire --init

# Check integrity
tripwire --check

# View report
twprint --print-report -r /var/lib/tripwire/report/$(hostname)-$(date +%Y%m%d).twr

# Update database after legitimate changes
tripwire --update --twrfile /var/lib/tripwire/report/latest.twr
```

### 11.4 OSSEC

```bash
# Download and install
wget https://github.com/ossec/ossec-hids/archive/3.6.0.tar.gz
tar -xzf 3.6.0.tar.gz
cd ossec-hids-3.6.0
./install.sh

# Follow prompts:
# - Server, agent, or local installation
# - Email notifications
# - Active response
# - Rootkit detection

# Start OSSEC
/var/ossec/bin/ossec-control start

# View alerts
tail -f /var/ossec/logs/alerts/alerts.log
```

---

## 12. File Integrity Monitoring

### 12.1 Why File Integrity Monitoring Matters

**FIM** detects unauthorized changes to critical system files - a key indicator of compromise.

**What to Monitor**:
- System binaries (`/bin`, `/sbin`, `/usr/bin`)
- Configuration files (`/etc`)
- Bootloader (`/boot`)
- System libraries (`/lib`, `/usr/lib`)
- Log files (detect tampering)

### 12.2 AIDE Configuration

```bash
# Configure what to monitor
vim /etc/aide/aide.conf

# Default rules
All=p+i+n+u+g+s+b+m+c+md5+sha1

# Directories to monitor
/bin Sha512
/sbin Sha512
/usr/bin Sha512
/usr/sbin Sha512
/etc All
/boot Sha512
/lib Sha512
/lib64 Sha512
/var/log Log

# Exclude changing directories
!/var/log/.*
!/tmp
!/proc
!/sys

# Initialize
aide --init
mv /var/lib/aide/aide.db.new /var/lib/aide/aide.db

# Schedule daily checks
vim /etc/cron.daily/aide
#!/bin/bash
/usr/bin/aide --check | mail -s "AIDE Daily Report" admin@example.com

chmod +x /etc/cron.daily/aide
```

---

## 13. Log Management and Monitoring

### 13.1 Centralized Logging

```bash
# Configure rsyslog to send logs to central server
vim /etc/rsyslog.conf

# Send all logs to remote syslog server
*.* @@logserver.example.com:514    # TCP
*.* @logserver.example.com:514     # UDP

# Restart rsyslog
systemctl restart rsyslog
```

### 13.2 Log Rotation

```bash
# Configure logrotate
vim /etc/logrotate.conf

# Rotate logs weekly
weekly

# Keep 4 weeks of backlogs
rotate 4

# Create new log files after rotation
create

# Use date as a suffix
dateext

# Compress old logs
compress

# Example service-specific config
vim /etc/logrotate.d/apache2

/var/log/apache2/*.log {
    daily
    missingok
    rotate 14
    compress
    delaycompress
    notifempty
    create 640 root adm
    sharedscripts
    postrotate
        systemctl reload apache2 > /dev/null
    endscript
}
```

### 13.3 Log Monitoring with Logwatch

```bash
# Install
apt-get install logwatch

# Configure
vim /etc/logwatch/conf/logwatch.conf

Output = mail
Format = html
MailTo = admin@example.com
MailFrom = logwatch@server.com
Detail = High

# Run manually
logwatch --detail High --mailto admin@example.com --range today

# Schedule daily
echo "0 6 * * * root /usr/sbin/logwatch --output mail --mailto admin@example.com --detail high" >> /etc/crontab
```

### 13.4 Important Logs to Monitor

```bash
# Authentication logs
tail -f /var/log/auth.log          # Debian/Ubuntu
tail -f /var/log/secure            # RHEL/CentOS

# Sudo usage
grep sudo /var/log/auth.log

# Failed login attempts
grep "Failed password" /var/log/auth.log

# System logs
tail -f /var/log/syslog            # Debian/Ubuntu
tail -f /var/log/messages          # RHEL/CentOS

# Kernel logs
dmesg
tail -f /var/log/kern.log

# Web server logs
tail -f /var/log/apache2/access.log
tail -f /var/log/apache2/error.log
tail -f /var/log/nginx/access.log
tail -f /var/log/nginx/error.log
```

---

## 14. Patch Management

### 14.1 Update Strategy

```bash
# Update package lists
apt-get update                     # Debian/Ubuntu
yum check-update                   # RHEL/CentOS

# Upgrade all packages
apt-get upgrade                    # Security and bug fixes
apt-get dist-upgrade               # Full upgrade including kernel
yum update                         # RHEL

# Security-only updates (Ubuntu)
apt-get install unattended-upgrades
dpkg-reconfigure --priority=low unattended-upgrades

# Configure automatic security updates
vim /etc/apt/apt.conf.d/50unattended-upgrades

Unattended-Upgrade::Allowed-Origins {
    "${distro_id}:${distro_codename}-security";
};

Unattended-Upgrade::Automatic-Reboot "false";
Unattended-Upgrade::Mail "admin@example.com";
```

### 14.2 Kernel Updates

```bash
# Check current kernel
uname -r

# List available kernels
apt-cache search linux-image       # Debian/Ubuntu
yum list available kernel          # RHEL/CentOS

# Install new kernel
apt-get install linux-image-VERSION
yum install kernel-VERSION

# Reboot to use new kernel
reboot

# Remove old kernels (keep 2-3 versions)
apt-get autoremove --purge
```

### 14.3 Vulnerability Scanning

```bash
# Install Lynis
apt-get install lynis

# Scan for vulnerabilities
lynis audit system --quick

# Check for known CVEs
apt-get install debian-goodies
checkrestart                       # Check which services need restart after updates
```

---

## 15. Service Hardening

### 15.1 Apache Web Server

```bash
# Edit Apache config
vim /etc/apache2/apache2.conf       # Debian/Ubuntu
vim /etc/httpd/conf/httpd.conf      # RHEL/CentOS

# Hide version information
ServerTokens Prod
ServerSignature Off

# Disable directory listing
<Directory /var/www/html>
    Options -Indexes
</Directory>

# Limit request size (prevent DoS)
LimitRequestBody 10485760          # 10MB

# Disable unnecessary modules
a2dismod status
a2dismod userdir
a2dismod autoindex

# Enable security modules
a2enmod headers
a2enmod ssl

# Add security headers
<IfModule mod_headers.c>
    Header always set X-Content-Type-Options "nosniff"
    Header always set X-Frame-Options "SAMEORIGIN"
    Header always set X-XSS-Protection "1; mode=block"
    Header always set Referrer-Policy "strict-origin-when-cross-origin"
</IfModule>

# Run Apache as non-privileged user
User www-data
Group www-data
```

### 15.2 Nginx Web Server

```bash
# Edit Nginx config
vim /etc/nginx/nginx.conf

# Hide version
server_tokens off;

# Security headers
add_header X-Frame-Options "SAMEORIGIN" always;
add_header X-Content-Type-Options "nosniff" always;
add_header X-XSS-Protection "1; mode=block" always;
add_header Referrer-Policy "no-referrer-when-downgrade" always;

# Limit request size
client_max_body_size 10M;

# Rate limiting
limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;
limit_req zone=one burst=5;

# SSL/TLS hardening
ssl_protocols TLSv1.2 TLSv1.3;
ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512;
ssl_prefer_server_ciphers on;
```

### 15.3 MySQL/MariaDB

```bash
# Run MySQL secure installation
mysql_secure_installation

# Configure MySQL
vim /etc/mysql/my.cnf

[mysqld]
# Bind to localhost only
bind-address = 127.0.0.1

# Disable LOCAL INFILE
local-infile = 0

# Set max connections
max_connections = 100

# Enable slow query log
slow_query_log = 1
slow_query_log_file = /var/log/mysql/slow.log
long_query_time = 2
```

### 15.4 PostgreSQL

```bash
# Configure PostgreSQL
vim /etc/postgresql/*/main/postgresql.conf

# Listen on localhost only
listen_addresses = 'localhost'

# Enable SSL
ssl = on

# Log connections
log_connections = on
log_disconnections = on

# Configure authentication
vim /etc/postgresql/*/main/pg_hba.conf

# TYPE  DATABASE  USER  ADDRESS       METHOD
local   all       all                 peer
host    all       all   127.0.0.1/32  md5
host    all       all   ::1/128       md5
```

---

## 16. Container Security

### 16.1 Docker Security

```bash
# Run containers as non-root user
docker run -u 1000:1000 image

# Use read-only root filesystem
docker run --read-only image

# Drop capabilities
docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE image

# Limit resources
docker run --memory="256m" --cpus="1.0" image

# Scan images for vulnerabilities
docker scan image-name

# Use trusted images only
docker pull ubuntu:20.04
# Verify signature
docker trust inspect ubuntu:20.04

# Enable Docker Content Trust
export DOCKER_CONTENT_TRUST=1
```

### 16.2 Rootless Docker

```bash
# Install rootless Docker
dockerd-rootless-setuptool.sh install

# Run Docker as non-root user
systemctl --user start docker
systemctl --user enable docker

# Export Docker socket
export DOCKER_HOST=unix://$XDG_RUNTIME_DIR/docker.sock
```

### 16.3 Docker Daemon Hardening

```bash
# Configure Docker daemon
vim /etc/docker/daemon.json

{
  "icc": false,
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  },
  "userns-remap": "default",
  "no-new-privileges": true,
  "live-restore": true
}

# Restart Docker
systemctl restart docker
```

---

## 17. Compliance and Benchmarks

### 17.1 CIS Benchmarks

**CIS (Center for Internet Security)** provides security configuration benchmarks.

```bash
# Download CIS Benchmark PDF
# Visit: https://www.cisecurity.org/cis-benchmarks/

# Key areas:
# 1.1 - Initial Setup (partition schemes, filesystem configuration)
# 1.2 - Configure Software Updates
# 1.3 - Filesystem Integrity Checking
# 1.4 - Secure Boot Settings
# 1.5 - Additional Process Hardening
# 1.6 - Mandatory Access Control
# 1.7 - Warning Banners
# 2.x - Services
# 3.x - Network Configuration
# 4.x - Logging and Auditing
# 5.x - Access, Authentication and Authorization
# 6.x - System Maintenance

# Automated CIS checking with Lynis
lynis audit system --quick
```

### 17.2 STIG (Security Technical Implementation Guide)

**STIG** - DoD security standards.

```bash
# Download STIGs
# Visit: https://public.cyber.mil/stigs/

# Apply RHEL STIG
git clone https://github.com/ansible/ansible-lockdown-rhel7-stig
cd ansible-lockdown-rhel7-stig
ansible-playbook site.yml
```

### 17.3 PCI-DSS Compliance

For systems handling credit card data:

```bash
# Key requirements:
# 1. Install and maintain firewall
# 2. Don't use vendor-supplied defaults
# 3. Protect stored cardholder data (encryption)
# 4. Encrypt transmission of cardholder data
# 5. Use and regularly update anti-virus
# 6. Develop and maintain secure systems
# 7. Restrict access by business need-to-know
# 8. Assign unique ID to each person with access
# 9. Restrict physical access to cardholder data
# 10. Track and monitor all network access
# 11. Regularly test security systems
# 12. Maintain information security policy
```

---

## 18. Security Auditing Tools

### 18.1 Lynis - Comprehensive

```bash
# Install
apt-get install lynis

# Run full audit
lynis audit system

# Run and save report
lynis audit system --logfile /var/log/lynis-$(date +%Y%m%d).log

# Check score
cat /var/log/lynis.log | grep "Hardening index"

# Review suggestions
cat /var/log/lynis.log | grep "Suggestion"

# Focus on specific tests
lynis show tests                   # List all tests
lynis --tests "AUTH-9282"          # Run specific test
```

### 18.2 OpenSCAP

```bash
# Install
apt-get install libopenscap8 ssg-debian

# List available profiles
oscap info /usr/share/xml/scap/ssg/content/ssg-debian9-ds.xml

# Run scan
oscap xccdf eval --profile standard \
  --results /tmp/results.xml \
  --report /tmp/report.html \
  /usr/share/xml/scap/ssg/content/ssg-debian9-ds.xml

# View report
firefox /tmp/report.html
```

### 18.3 Nessus (Vulnerability Scanner)

```bash
# Download and install
# Visit: https://www.tenable.com/products/nessus

# Start Nessus
systemctl start nessusd

# Access web interface
https://localhost:8834

# Create scan:
# 1. New Scan → Basic Network Scan
# 2. Target: localhost or 127.0.0.1
# 3. Launch scan
# 4. Review vulnerabilities
# 5. Apply fixes
# 6. Rescan to verify
```

### 18.4 Nikto (Web Server Scanner)

```bash
# Install
apt-get install nikto

# Scan web server
nikto -h http://localhost

# Comprehensive scan
nikto -h http://localhost -Tuning 123456789

# Save results
nikto -h http://localhost -output /tmp/nikto-results.html -Format html
```

### 18.5 ClamAV (Antivirus)

```bash
# Install
apt-get install clamav clamav-daemon

# Update virus database
freshclam

# Scan system
clamscan -r --infected --remove /home

# Schedule daily scans
vim /etc/cron.daily/clamav

#!/bin/bash
/usr/bin/clamscan -r --infected --log=/var/log/clamav-scan.log /home
/usr/bin/clamscan -r --infected --log=/var/log/clamav-scan.log /var/www

chmod +x /etc/cron.daily/clamav
```

---

## 19. Incident Response Preparation

### 19.1 Backup Strategy

```bash
# Full system backup with rsync
rsync -aAXv --exclude={"/dev/*","/proc/*","/sys/*","/tmp/*","/run/*","/mnt/*","/media/*","/lost+found"} / /backup/

# Automated daily backups
vim /etc/cron.daily/backup

#!/bin/bash
BACKUP_DIR="/backup/$(date +%Y%m%d)"
mkdir -p $BACKUP_DIR
rsync -aAXv --exclude={"/dev/*","/proc/*","/sys/*","/tmp/*"} / $BACKUP_DIR/
find /backup/* -mtime +7 -exec rm -rf {} \;

chmod +x /etc/cron.daily/backup
```

### 19.2 Disaster Recovery Plan

```bash
# Create bootable recovery USB
dd if=/path/to/ubuntu.iso of=/dev/sdX bs=4M status=progress

# Document system configuration
mkdir /root/system-docs

# Save package list
dpkg --get-selections > /root/system-docs/packages.txt

# Save network config
ip addr > /root/system-docs/network-config.txt
cat /etc/network/interfaces >> /root/system-docs/network-config.txt

# Save firewall rules
iptables-save > /root/system-docs/iptables-rules

# Save cron jobs
crontab -l > /root/system-docs/cron-root.txt
```

### 19.3 Forensics Preparation

```bash
# Install forensics tools
apt-get install sleuthkit autopsy volatility

# Create forensic image (if system compromised)
dd if=/dev/sda of=/mnt/external/forensic-image.dd bs=4M status=progress
md5sum /mnt/external/forensic-image.dd > /mnt/external/forensic-image.dd.md5

# Preserve logs
tar -czf /mnt/external/logs-$(date +%Y%m%d).tar.gz /var/log
```

---

## 20. Hands-On Labs

### Lab 1: Complete Server Hardening

**Objective**: Harden a fresh Ubuntu server and achieve Lynis score > 80.

**Steps**:

1. **Initial Assessment**:
```bash
sudo lynis audit system > /tmp/initial-audit.txt
grep "Hardening index" /tmp/initial-audit.txt
```

2. **Apply SSH Hardening**:
```bash
sudo vim /etc/ssh/sshd_config
# PermitRootLogin no
# PasswordAuthentication no
sudo systemctl restart sshd
```

3. **Configure Firewall**:
```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22/tcp
sudo ufw enable
```

4. **Set Strong Password Policy**:
```bash
sudo vim /etc/security/pwquality.conf
# minlen = 14
# dcredit = -1
sudo vim /etc/login.defs
# PASS_MAX_DAYS 90
```

5. **Remove Unnecessary Services**:
```bash
systemctl list-units --type=service --state=running
sudo systemctl disable avahi-daemon
sudo systemctl stop avahi-daemon
```

6. **Enable Automatic Updates**:
```bash
sudo apt-get install unattended-upgrades
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

7. **Kernel Hardening**:
```bash
sudo vim /etc/sysctl.conf
# net.ipv4.ip_forward = 0
# kernel.randomize_va_space = 2
sudo sysctl -p
```

8. **Re-audit**:
```bash
sudo lynis audit system > /tmp/final-audit.txt
grep "Hardening index" /tmp/final-audit.txt
```

**Success Criteria**: Hardening index improved by 20+ points.

---

### Lab 2: Configure SELinux/AppArmor

**Objective**: Enable and configure MAC for Apache web server.

**AppArmor (Ubuntu)**:
```bash
# Install Apache
sudo apt-get install apache2

# Check AppArmor status
sudo aa-status

# Put Apache profile in complain mode
sudo aa-complain /etc/apparmor.d/usr.sbin.apache2

# Generate profile
sudo aa-logprof

# Set to enforce mode
sudo aa-enforce /etc/apparmor.d/usr.sbin.apache2

# Test
sudo systemctl restart apache2
curl http://localhost
```

**SELinux (RHEL/CentOS)**:
```bash
# Install Apache
sudo yum install httpd

# Set SELinux to enforcing
sudo setenforce 1

# Set file context
sudo semanage fcontext -a -t httpd_sys_content_t "/var/www/html(/.*)?"
sudo restorecon -Rv /var/www/html

# Test
sudo systemctl start httpd
curl http://localhost

# Check for denials
sudo ausearch -m avc -ts recent
```

---

### Lab 3: Implement Fail2Ban

**Objective**: Configure Fail2Ban to protect SSH.

```bash
# Install
sudo apt-get install fail2ban

# Configure
sudo vim /etc/fail2ban/jail.local

[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 600
findtime = 600

# Start
sudo systemctl start fail2ban
sudo systemctl enable fail2ban

# Test (from another machine)
# Try failed SSH login 4 times

# Check banned IPs
sudo fail2ban-client status sshd

# Verify iptables rule created
sudo iptables -L -n | grep fail2ban
```

---

### Lab 4: File Integrity Monitoring with AIDE

**Objective**: Set up AIDE and detect file changes.

```bash
# Install
sudo apt-get install aide

# Initialize
sudo aideinit
sudo mv /var/lib/aide/aide.db.new /var/lib/aide/aide.db

# Modify a system file
sudo touch /etc/testfile

# Run check
sudo aide --check

# Should show:
# Added entries:
# f = ..........C..... /etc/testfile

# Update database to accept change
sudo aide --update
sudo mv /var/lib/aide/aide.db.new /var/lib/aide/aide.db
```

---

### Lab 5: Security Audit with Lynis

**Objective**: Run Lynis audit and fix top 5 issues.

```bash
# Run audit
sudo lynis audit system

# Review suggestions
cat /var/log/lynis.log | grep "Suggestion" | head -10

# Example fixes:
# 1. Install debsums
sudo apt-get install debsums

# 2. Enable process accounting
sudo apt-get install acct
sudo systemctl enable acct

# 3. Install fail2ban
sudo apt-get install fail2ban

# 4. Harden kernel
sudo vim /etc/sysctl.conf
# Add recommended settings

# 5. Install rootkit checker
sudo apt-get install rkhunter

# Re-run audit
sudo lynis audit system
# Compare scores
```

---

## 21. Interview Questions and Answers

### Q1: What is the difference between hardening and patching?

**Answer**: 

**Patching** is the process of updating software to fix bugs and security vulnerabilities. It addresses **known** vulnerabilities in specific software versions.

**Hardening** is the process of securing a system by reducing its attack surface and implementing security controls. It's a **proactive** approach to security that makes exploitation more difficult, even against unknown vulnerabilities (zero-days).

**Example**:
- Patching: Updating kernel from 5.4.0 to 5.4.1 to fix CVE-2021-12345
- Hardening: Disabling unnecessary services, configuring firewall, enabling SELinux, setting secure permissions

**Best Practice**: Do BOTH. Patching fixes known issues; hardening reduces overall risk.

---

### Q2: Explain the defense-in-depth approach and provide examples.

**Answer**: 

Defense-in-depth is a security strategy that uses multiple layers of security controls. If one layer fails, others still provide protection.

**Layers** (with examples):
1. **Physical**: Locked server room, biometric access
2. **Perimeter**: Firewall, IPS, DMZ
3. **Network**: Network segmentation, VLANs, internal firewall
4. **Host**: OS hardening, antivirus, HIDS, patching (THIS GUIDE)
5. **Application**: Input validation, WAF, secure coding
6. **Data**: Encryption at rest/transit, DLP, backups
7. **Human**: Security awareness training, policies

**Example Scenario**:
An attacker trying to steal database records must:
1. Physically access building (Physical security)
2. Bypass perimeter firewall (Perimeter security)
3. Compromise DMZ web server (Network + Host security)
4. Exploit web application (Application security)
5. Pivot to database server (Network segmentation)
6. Bypass database access controls (Access control)
7. Decrypt stolen data (Data security)

Each layer increases difficulty and provides detection opportunities.

---

### Q3: What is the principle of least privilege and how do you implement it on Linux?

**Answer**:

**Principle of Least Privilege**: Users/processes should have only the minimum permissions necessary to perform their function - no more, no less.

**Implementation on Linux**:

1. **User Accounts**:
```bash
# Create user without sudo access
useradd -m -s /bin/bash limiteduser

# Grant specific sudo commands only
# Instead of: limiteduser ALL=(ALL) ALL
# Use:
limiteduser ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart nginx
```

2. **File Permissions**:
```bash
# Instead of 777 (all permissions to all)
chmod 777 file.txt                 # BAD

# Use minimum needed
chmod 644 file.txt                 # Good: Owner rw, Others read
chmod 600 sensitive.conf           # Better: Owner only
```

3. **Service Accounts**:
```bash
# Run services as dedicated users, not root
User=www-data                      # Apache runs as www-data
User=mysql                         # MySQL runs as mysql
```

4. **Capabilities vs SUID**:
```bash
# Instead of SUID (full root)
chmod u+s /usr/bin/ping            # BAD: Full root access

# Use capabilities (specific privilege)
setcap cap_net_raw+ep /usr/bin/ping  # Good: Only raw sockets
```

5. **SELinux/AppArmor**:
- Confine processes even if they run as root
- Web server can only access web content, even if compromised

---

### Q4: How do you balance security and usability when hardening a system?

**Answer**:

This is the eternal security dilemma. Too much security = unusable system. Too little = insecure system.

**Approach**:

1. **Risk-Based**: Harden based on system criticality
   - Public-facing web server: Maximum hardening
   - Internal development workstation: Moderate hardening

2. **Gradual Implementation**:
   - Phase 1: Low-impact changes (update software, firewall)
   - Phase 2: Medium-impact (SELinux complain mode, log everything)
   - Phase 3: High-impact (SELinux enforce, strict policies)

3. **Testing Environment**:
   - Test hardening measures in staging first
   - Monitor for application breakage
   - Adjust policies before production

4. **Exception Process**:
   - Document why exceptions are needed
   - Implement compensating controls
   - Regular review of exceptions

**Example**: 
- **SELinux Breaking Application**:
  - DON'T: Disable SELinux entirely (`setenforce 0`)
  - DO: Run in complain mode, review denials, create custom policy
  
5. **User Education**:
   - Explain why security measures exist
   - Provide alternatives (e.g., VPN instead of direct SSH)

**Quote**: "Security at the expense of usability comes at the expense of security" (people will find workarounds)

---

### Q5: What are the most critical configurations in SSH hardening?

**Answer**:

Top 10 SSH hardening configurations (in priority order):

1. **Disable Root Login**:
```bash
PermitRootLogin no
```
Forces users to login as themselves (accountability), then sudo to root.

2. **Key-Based Authentication Only**:
```bash
PasswordAuthentication no
PubkeyAuthentication yes
```
Prevents brute-force password attacks.

3. **Change Default Port**:
```bash
Port 2222
```
Reduces automated attack attempts (security through obscurity, but effective).

4. **Limit User Access**:
```bash
AllowUsers alice bob
# or
AllowGroups sshusers
```
Reduces attack surface.

5. **Disable Protocol 1**:
```bash
Protocol 2
```
SSH Protocol 1 is insecure.

6. **Set Idle Timeout**:
```bash
ClientAliveInterval 300
ClientAliveCountMax 2
```
Disconnects inactive sessions (10 minutes in this example).

7. **Limit Authentication Attempts**:
```bash
MaxAuthTries 3
```
Prevents brute-force attacks.

8. **Use Strong Ciphers**:
```bash
Ciphers aes256-gcm@openssh.com,aes128-gcm@openssh.com
```
Removes weak encryption algorithms.

9. **Disable Empty Passwords**:
```bash
PermitEmptyPasswords no
```
Prevents accounts with no password.

10. **Enable 2FA** (bonus):
```bash
AuthenticationMethods publickey,keyboard-interactive
```
Requires both SSH key AND 2FA code.

**Verification**: After changes, test from another session BEFORE closing current session!

---

### Q6: Explain SELinux modes and when to use each.

**Answer**:

SELinux has **3 modes**:

**1. Enforcing**:
- Policy is **enforced**
- Violations are **blocked** and **logged**
- System is **secure**

**When to use**: Production systems, once policies are tested and working.

**2. Permissive**:
- Policy is **NOT enforced**
- Violations are **allowed** but **logged**
- System behaves as if SELinux is disabled, but logs what WOULD be blocked

**When to use**: 
- Testing policies before deploying
- Troubleshooting SELinux issues
- Developing new policies

**3. Disabled**:
- SELinux is completely **off**
- No protection, no logging

**When to use**: **NEVER** in production (except extreme compatibility issues)

**Commands**:
```bash
# Check current mode
getenforce

# Change temporarily
setenforce 0           # Permissive
setenforce 1           # Enforcing

# Change permanently
vim /etc/selinux/config
SELINUX=enforcing      # or permissive or disabled
# Reboot required
```

**Workflow**:
1. Start: **Permissive** (collect denial logs)
2. Review: Check `/var/log/audit/audit.log` for denials
3. Fix: Create policies or fix labels
4. Test: Ensure application works
5. Deploy: Switch to **Enforcing**
6. Monitor: Continue watching logs

**Common Mistake**: Disabling SELinux when it breaks something
**Correct Approach**: Switch to permissive, investigate denials, fix policy

---

### Q7: What is the difference between AIDE and Tripwire?

**Answer**:

Both are **File Integrity Monitoring (FIM)** tools that detect unauthorized file changes, but differ in implementation and features:

**AIDE (Advanced Intrusion Detection Environment)**:

**Pros**:
- Open source and free
- Lightweight
- Easy to configure
- Good for basic FIM needs
- Single-file database

**Cons**:
- No client-server architecture
- Less sophisticated reporting
- Manual database updates

**Configuration**:
```bash
# Simple rules
/bin Sha512
/etc All
```

**Tripwire**:

**Pros**:
- More sophisticated
- Enterprise features
- Client-server architecture
- Better reporting
- Policy signing (prevents attacker from modifying policies)

**Cons**:
- Commercial version ($$$)
- More complex setup
- Heavier resource usage

**When to use**:
- **AIDE**: Small-medium deployments, budget-conscious, simple needs
- **Tripwire**: Enterprise environments, compliance requirements, centralized management

**Similarity**: Both work on same principle:
1. Create baseline database of file hashes
2. Periodically scan files
3. Compare to baseline
4. Alert on changes

**Alternative**: **OSSEC** (combines FIM with HIDS)

---

### Q8: How does Fail2Ban work and what are its limitations?

**Answer**:

**How Fail2Ban Works**:

1. **Monitor Logs**: Watches log files (e.g., `/var/log/auth.log`)
2. **Pattern Matching**: Uses filters (regex) to detect failures
3. **Count Failures**: Tracks failures per IP within a time window
4. **Ban Action**: When threshold exceeded, bans IP via iptables
5. **Unban**: After ban time expires, removes iptables rule

**Example Flow**:
```
1. SSH login failed from 192.168.1.100
2. Fail2Ban sees "Failed password" in auth.log
3. Increment counter for 192.168.1.100
4. After 3 failures in 10 minutes:
5. Execute: iptables -A INPUT -s 192.168.1.100 -j DROP
6. After 1 hour (bantime):
7. Execute: iptables -D INPUT -s 192.168.1.100 -j DROP
```

**Configuration**:
```bash
[sshd]
enabled = true
filter = sshd               # Regex patterns in /etc/fail2ban/filter.d/sshd.conf
logpath = /var/log/auth.log
maxretry = 3                # Ban after 3 failures
findtime = 600              # Within 10 minutes
bantime = 3600              # Ban for 1 hour
```

**Limitations**:

1. **Distributed Attacks**: Can't detect slow attacks from many IPs
2. **Post-Compromise**: Doesn't help if attacker already has valid credentials
3. **Log Dependency**: If attacker deletes logs, Fail2Ban blind
4. **Resource**: Heavy log parsing on busy servers
5. **False Positives**: Legitimate users can be banned (forgotten password)
6. **IP Spoofing**: Can be used for DoS (ban legitimate users)

**Mitigations**:
- Combine with firewall geo-blocking
- Whitelist trusted IPs
- Implement account lockout in addition to Fail2Ban
- Use honeypot ports to catch scanners
- Monitor Fail2Ban itself

**Best Use**: Complement to other security measures, not sole defense.

---

### Q9: What are the key differences between a firewall and an IDS/IPS?

**Answer**:

**Firewall** (e.g., iptables, UFW, firewalld):

**Function**: **Access control** - Allows or denies traffic based on rules

**Works at**: Network layer (Layer 3-4) - IP addresses, ports, protocols

**Rules**: Static rules based on source/destination IP, port, protocol
```bash
# Example: Allow SSH from 192.168.1.0/24
iptables -A INPUT -s 192.168.1.0/24 -p tcp --dport 22 -j ACCEPT
```

**Decision**: Binary - ALLOW or DENY

**Speed**: Fast (simple rule matching)

**Analogy**: Security guard checking IDs at the door

---

**IDS (Intrusion Detection System)** (e.g., Snort, Suricata):

**Function**: **Monitoring** - Detects malicious activity and alerts

**Works at**: Network layer AND application layer (deep packet inspection)

**Detection**: Signature-based and anomaly-based
```
# Example: Detect SQL injection in HTTP traffic
alert tcp any any -> any 80 (content:"UNION SELECT"; msg:"SQL Injection Attempt";)
```

**Decision**: **ALERT** only (passive)

**Speed**: Slower (deep inspection)

**Analogy**: CCTV camera recording suspicious activity

---

**IPS (Intrusion Prevention System)** (e.g., Snort in inline mode, Suricata):

**Function**: **Detection + Prevention** - Detects AND blocks attacks

**Works at**: Same as IDS but inline (traffic passes through it)

**Action**: Can **DROP** malicious traffic

**Decision**: DETECT → BLOCK

**Speed**: Slower than firewall (deep inspection + blocking)

**Analogy**: Security guard who both monitors AND takes action

---

**Comparison**:

| Feature | Firewall | IDS | IPS |
|---------|----------|-----|-----|
| Action | Block | Alert | Block + Alert |
| Position | Gateway | Passive tap | Inline |
| Layer | L3-L4 | L3-L7 | L3-L7 |
| Detection | Rule-based | Signature/Anomaly | Signature/Anomaly |
| Speed | Fast | Medium | Medium-Slow |
| False Positives | Low | Medium-High | Medium-High |

**Use Together** (Defense-in-Depth):
1. Firewall: Block known bad (e.g., close unused ports)
2. IDS: Detect attacks that pass firewall
3. IPS: Block sophisticated attacks firewall missed

**Example Scenario**:
- Firewall allows port 80 (web traffic)
- IPS detects SQL injection in HTTP request
- IPS blocks that specific request
- IDS alerts security team for investigation

---

### Q10: How would you harden a Linux server for PCI-DSS compliance?

**Answer**:

PCI-DSS (Payment Card Industry Data Security Standard) has 12 requirements. Key Linux hardening steps:

**Requirement 1: Install and maintain firewall**:
```bash
# Configure firewall
ufw default deny incoming
ufw allow from 192.168.1.0/24 to any port 22  # SSH from internal only
ufw allow 443/tcp                             # HTTPS only
ufw enable
```

**Requirement 2: Don't use vendor-supplied defaults**:
```bash
# Change default passwords
passwd root

# Change default ports
vim /etc/ssh/sshd_config
Port 2222

# Remove/rename default accounts
userdel -r guest
```

**Requirement 3: Protect stored cardholder data**:
```bash
# Encrypt filesystems
apt-get install cryptsetup
cryptsetup luksFormat /dev/sdb1

# Encrypt databases
# Use TDE (Transparent Data Encryption) for MySQL/PostgreSQL

# Mask PAN in logs
# Configure applications to NOT log full card numbers
```

**Requirement 4: Encrypt transmission**:
```bash
# Enforce TLS 1.2+ only
vim /etc/nginx/nginx.conf
ssl_protocols TLSv1.2 TLSv1.3;

# Strong ciphers
ssl_ciphers 'ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-GCM-SHA256';
```

**Requirement 5: Protect against malware**:
```bash
# Install antivirus
apt-get install clamav clamav-daemon
freshclam
clamscan -r --infected /var/www

# Schedule daily scans
echo "0 2 * * * /usr/bin/clamscan -r /var/www" >> /etc/crontab
```

**Requirement 6: Secure systems and applications**:
```bash
# Patch regularly
apt-get update && apt-get upgrade
unattended-upgrades

# Secure coding practices
# Input validation, parameterized queries
```

**Requirement 7: Restrict access by business need-to-know**:
```bash
# Principle of least privilege
usermod -aG cardholderdata-read alice  # Read-only access
usermod -aG cardholderdata-write bob   # Write access only for specific users

# File permissions
chmod 640 /data/cardholder/*
chown root:cardholderdata-read /data/cardholder/*
```

**Requirement 8: Unique IDs and strong authentication**:
```bash
# Unique user accounts (no shared accounts)
# Strong password policy
vim /etc/security/pwquality.conf
minlen = 12
dcredit = -1
ucredit = -1

# 2FA
apt-get install libpam-google-authenticator

# Account lockout
vim /etc/pam.d/common-auth
auth required pam_tally2.so deny=6 unlock_time=1800
```

**Requirement 9: Restrict physical access**:
- Lock server room
- Badge access
- CCTV
- (Mostly physical controls, not Linux-specific)

**Requirement 10: Track and monitor all access**:
```bash
# Enable auditd
apt-get install auditd
systemctl enable auditd

# Audit cardholder data access
auditctl -w /data/cardholder -p rwxa -k cardholder_access

# Centralized logging
vim /etc/rsyslog.conf
*.* @@logserver.example.com:514

# Log retention (1 year minimum)
vim /etc/logrotate.d/syslog
rotate 365
```

**Requirement 11: Test security systems**:
```bash
# Vulnerability scanning (quarterly)
nessus-cli scan create --targets localhost

# Penetration testing (annually)
# External penetration test firm

# File integrity monitoring
aide --init
aide --check
```

**Requirement 12: Maintain information security policy**:
- Document all procedures
- Security awareness training
- Incident response plan
- (Policy/procedural, not technical)

**Verification**:
```bash
# Use PCI-DSS scanning tools
lynis audit system
openscap

# Self-Assessment Questionnaire (SAQ)
# Or hire QSA (Qualified Security Assessor)
```

---

## 22. Summary and Checklist

### 22.1 Complete Hardening Checklist

```
┌─────────────────────────────────────────────────────────────┐
│       COMPREHENSIVE LINUX HARDENING CHECKLIST              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ PHASE 1: PRE-HARDENING (Assessment)                        │
│ ☐ Document current system state                            │
│ ☐ Run Lynis baseline audit                                 │
│ ☐ Inventory all services and users                         │
│ ☐ Create system backup                                     │
│ ☐ Create rollback plan                                     │
│                                                             │
│ PHASE 2: ACCOUNT SECURITY                                  │
│ ☐ Set strong password policy (14+ chars, complexity)       │
│ ☐ Configure password aging (max 90 days)                   │
│ ☐ Enable account lockout (5 failures)                      │
│ ☐ Remove/lock unnecessary user accounts                    │
│ ☐ Ensure only root has UID 0                               │
│ ☐ Configure sudo properly (no NOPASSWD unless needed)      │
│ ☐ Set umask to 027                                         │
│                                                             │
│ PHASE 3: SSH HARDENING                                     │
│ ☐ Disable root login (PermitRootLogin no)                  │
│ ☐ Disable password authentication (keys only)              │
│ ☐ Change default port                                      │
│ ☐ Limit user access (AllowUsers/AllowGroups)               │
│ ☐ Set idle timeout                                         │
│ ☐ Use strong ciphers                                       │
│ ☐ Configure Fail2Ban for SSH                               │
│ ☐ Consider 2FA                                             │
│                                                             │
│ PHASE 4: FILESYSTEM SECURITY                               │
│ ☐ Set secure mount options (noexec,nosuid on /tmp)         │
│ ☐ Fix world-writable files                                 │
│ ☐ Set correct permissions on critical files                │
│ ☐ Disable core dumps                                       │
│ ☐ Apply sticky bit on /tmp                                 │
│                                                             │
│ PHASE 5: KERNEL HARDENING                                  │
│ ☐ Configure sysctl (IP forwarding, syncookies, etc.)       │
│ ☐ Blacklist unused kernel modules                          │
│ ☐ Enable ASLR                                              │
│ ☐ Restrict dmesg access                                    │
│                                                             │
│ PHASE 6: NETWORK SECURITY                                  │
│ ☐ Configure firewall (UFW/firewalld/iptables)              │
│ ☐ Default deny incoming traffic                            │
│ ☐ Allow only necessary ports                               │
│ ☐ Disable IPv6 if not used                                 │
│ ☐ Configure TCP wrappers                                   │
│                                                             │
│ PHASE 7: SERVICE HARDENING                                 │
│ ☐ Remove unnecessary packages                              │
│ ☐ Disable unnecessary services                             │
│ ☐ Harden Apache/Nginx (hide version, security headers)     │
│ ☐ Harden MySQL/PostgreSQL (bind to localhost)              │
│ ☐ Remove compilers from production servers                 │
│                                                             │
│ PHASE 8: MANDATORY ACCESS CONTROL                          │
│ ☐ Enable SELinux/AppArmor (enforcing mode)                 │
│ ☐ Create/tune profiles for services                        │
│ ☐ Monitor for denials                                      │
│ ☐ Create custom policies as needed                         │
│                                                             │
│ PHASE 9: LOGGING AND MONITORING                            │
│ ☐ Configure centralized logging                            │
│ ☐ Set up log rotation                                      │
│ ☐ Install and configure Logwatch                           │
│ ☐ Enable auditd                                            │
│ ☐ Monitor authentication logs                              │
│                                                             │
│ PHASE 10: INTRUSION DETECTION                              │
│ ☐ Install and configure AIDE/Tripwire (FIM)                │
│ ☐ Configure Fail2Ban for all services                      │
│ ☐ Consider OSSEC/Wazuh for HIDS                            │
│ ☐ Install and update ClamAV                                │
│                                                             │
│ PHASE 11: PATCH MANAGEMENT                                 │
│ ☐ Configure automatic security updates                     │
│ ☐ Establish patch testing procedure                        │
│ ☐ Schedule regular maintenance windows                     │
│ ☐ Subscribe to security mailing lists                      │
│                                                             │
│ PHASE 12: BACKUP AND RECOVERY                              │
│ ☐ Implement automated backups                              │
│ ☐ Test backup restoration                                  │
│ ☐ Store backups off-site                                   │
│ ☐ Document recovery procedures                             │
│                                                             │
│ PHASE 13: COMPLIANCE                                       │
│ ☐ Run CIS benchmark assessment                             │
│ ☐ Address compliance requirements (PCI, HIPAA, etc.)       │
│ ☐ Document all configurations                              │
│ ☐ Schedule regular audits                                  │
│                                                             │
│ PHASE 14: POST-HARDENING                                   │
│ ☐ Run Lynis final audit                                    │
│ ☐ Compare scores (should be 20+ points higher)             │
│ ☐ Test all applications still work                         │
│ ☐ Document all changes made                                │
│ ☐ Train users on new security measures                     │
│ ☐ Establish ongoing monitoring procedures                  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 22.2 Quick Command Reference

```bash
# === ASSESSMENT ===
lynis audit system
nessus scan localhost

# === USER SECURITY ===
vim /etc/security/pwquality.conf   # Password policy
chage -M 90 username               # Password aging
visudo                             # Sudo configuration

# === SSH ===
vim /etc/ssh/sshd_config
systemctl restart sshd
fail2ban-client status sshd

# === FIREWALL ===
ufw enable
ufw default deny incoming
ufw allow 22/tcp

# === KERNEL ===
vim /etc/sysctl.conf
sysctl -p

# === MAC ===
aa-status                          # AppArmor
getenforce                         # SELinux
aa-enforce /path/to/profile        # Enforce AppArmor
setenforce 1                       # Enforce SELinux

# === MONITORING ===
tail -f /var/log/auth.log
aide --check
fail2ban-client status

# === UPDATES ===
apt-get update && apt-get upgrade
unattended-upgrades

# === AUDIT ===
lynis audit system --quick
```

### 22.3 Final Thoughts

**Hardening is a Journey, Not a Destination**

Security is not a one-time project - it's an ongoing process:

1. **Initial Hardening** (this guide)
2. **Continuous Monitoring** (logs, IDS, FIM)
3. **Regular Updates** (patches, security advisories)
4. **Periodic Audits** (quarterly Lynis scans)
5. **Incident Response** (when breaches happen)
6. **Lessons Learned** (improve after incidents)

**The 80/20 Rule**:
- 80% of security comes from 20% of measures
- Focus on high-impact items first (SSH, firewall, updates)

**Balance**:
- Too strict → Users find workarounds → Less secure
- Too lax → Easy to compromise
- Find the right balance for your environment

**Resources for Continued Learning**:
- CIS Benchmarks: https://www.cisecurity.org/cis-benchmarks/
- NIST Cyber Security Framework: https://www.nist.gov/cyberframework
- OWASP: https://owasp.org/
- SANS: https://www.sans.org/
- Linux Security Mailing Lists

---

## Congratulations!

You've completed the **Linux Hardening & Defense** comprehensive guide. You now have:

✅ Complete understanding of Linux system hardening
✅ Systematic hardening methodology
✅ CIS Benchmark knowledge
✅ SELinux/AppArmor expertise
✅ Firewall and network security skills
✅ Intrusion detection capabilities
✅ Compliance framework understanding
✅ 5 hands-on labs with solutions
✅ 10 interview questions with expert answers
✅ Complete hardening checklist

**The Complete Picture**:
- **File 09** (Linux Privilege Escalation) = **Red Team** perspective
- **File 10** (Linux Hardening & Defense) = **Blue Team** perspective

Understanding both makes you a **complete security professional**.

**Next Actions**:
1. Practice hardening on test systems
2. Run Lynis audits regularly
3. Stay updated on security news
4. Contribute to open-source security projects
5. Get certified (OSCP, GIAC, etc.)

---

*End of Linux Hardening & Defense - Comprehensive Guide*
*File 10 of 11 - 03_Linux Module*
*Total: 2,100+ lines | Complete Hardening Framework | 5 Labs | 10 Interview Q&A*
*Project Status: 10/11 FILES COMPLETE (91%)* 🎉