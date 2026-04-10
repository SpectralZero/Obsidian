# 10 - Hands-On Lab Setup: Build Your Own SOC

---
title: "Hands-On Lab Setup - Build Your Own SOC"
parent: "[[13_SOC_and_SIEM]]"
tags:
  - hands-on
  - lab-setup
  - home-lab
  - practice
  - soc-lab
created: 2026-01-31
updated: 2026-01-31
---

> **Executive Summary**: Theory is worthless without practice. This chapter is your complete blueprint for building a functional SOC lab at home where you can practice everything you've learned. You'll set up virtual machines, install and configure Splunk/ELK, generate realistic log data, simulate attacks, create detection rules, and investigate incidents - all for FREE (or <$100). This hands-on experience is what transforms knowledge into employable skills. By completing this lab, you'll have a working SOC environment to demonstrate in interviews, continue learning, and practice real-world scenarios. This is how you become job-ready.

---

## Table of Contents

1. [Lab Overview & Architecture](#1-lab-overview--architecture)
2. [Hardware & Software Requirements](#2-hardware--software-requirements)
3. [Lab Setup Option 1: Free (Local VMs)](#3-lab-setup-option-1-free-local-vms)
4. [Lab Setup Option 2: Cloud ($5-10/month)](#4-lab-setup-option-2-cloud)
5. [Installing Splunk Free](#5-installing-splunk-free)
6. [Setting Up Log Sources](#6-setting-up-log-sources)
7. [Generating Test Data & Attack Scenarios](#7-generating-test-data--attack-scenarios)
8. [Practice Scenarios](#8-practice-scenarios)
9. [Building Your Portfolio](#9-building-your-portfolio)

---

## 1. Lab Overview & Architecture

### 1.1 What You'll Build

```
┌──────────────────────────────────────────────┐
│       YOUR SOC LAB ARCHITECTURE              │
└──────────────────────────────────────────────┘

         ┌─────────────────┐
         │  YOUR COMPUTER  │
         │  (Host System)  │
         └────────┬────────┘
                  │
         VirtualBox/VMware
                  │
      ┌───────────┼───────────┐
      │           │           │
┌─────▼────┐ ┌────▼─────┐ ┌──▼───────┐
│ Ubuntu   │ │ Windows  │ │  Kali    │
│ (SIEM)   │ │ (Victim) │ │(Attacker)│
│          │ │          │ │          │
│ Splunk   │ │ Logs →   │ │ Attack   │
│   or     │ │ Splunk   │ │ Tools    │
│  ELK     │ │          │ │          │
└──────────┘ └──────────┘ └──────────┘

WHAT EACH VM DOES:
- Ubuntu: Runs your SIEM (Splunk or ELK)
- Windows: Generates logs (simulates corporate workstation)
- Kali: Attack platform (simulates threat actor)
```

### 1.2 Learning Objectives

By completing this lab, you will:
```
✅ Install and configure a production-grade SIEM
✅ Collect logs from multiple sources
✅ Create detection rules
✅ Simulate real attacks
✅ Investigate incidents
✅ Document findings
✅ Have portfolio project for interviews
```

---

## 2. Hardware & Software Requirements

### 2.1 Minimum Requirements (Budget Setup)

```
COMPUTER:
├─ CPU: 4 cores (Intel i5/AMD Ryzen 5)
├─ RAM: 16 GB (8 GB works but slow)
├─ Disk: 100 GB free space
├─ OS: Windows 10/11, macOS, or Linux
└─ Network: Internet connection

SOFTWARE (ALL FREE):
├─ VirtualBox 7.0+ (free)
├─ Ubuntu Server 22.04 ISO (free)
├─ Windows 10 Evaluation (free 90 days)
├─ Kali Linux (free)
├─ Splunk Free (500MB/day, free forever)
└─ Text editor (VSCode, Notepad++, free)

COST: $0 (100% free!)
```

### 2.2 Recommended Requirements (Better Performance)

```
COMPUTER:
├─ CPU: 6+ cores
├─ RAM: 32 GB
├─ Disk: 250 GB SSD
└─ Network: Gigabit ethernet

COST: $0 (if you have decent computer)
```

---

## 3. Lab Setup Option 1: Free (Local VMs)

### 3.1 Install VirtualBox

**Windows/Mac/Linux**:
```
1. Download VirtualBox: https://www.virtualbox.org/
2. Install with default options
3. Install VirtualBox Extension Pack (USB, networking features)
```

### 3.2 Create Virtual Network

```
VirtualBox → Tools → Network Manager
→ Create NAT Network

Name: SOCLab
IPv4 Prefix: 192.168.100.0/24
DHCP: Enabled

This network allows VMs to talk to each other
```

### 3.3 Create VM #1: Ubuntu (SIEM Server)

```
STEP 1: Create VM
├─ Name: SOC-SIEM
├─ Type: Linux
├─ Version: Ubuntu (64-bit)
├─ RAM: 4 GB (4096 MB)
├─ Disk: 50 GB (dynamically allocated)
└─ Network: SOCLab (NAT Network)

STEP 2: Install Ubuntu Server
├─ Download Ubuntu Server 22.04 LTS ISO
├─ Mount ISO to VM
├─ Start VM and install:
│  ├─ Username: socadmin
│  ├─ Password: (your choice)
│  ├─ Install OpenSSH server: YES
│  └─ No additional packages
└─ Reboot when complete

STEP 3: Install Splunk (covered in Section 5)
```

### 3.4 Create VM #2: Windows 10 (Victim/Log Source)

```
STEP 1: Download Windows 10 Evaluation
├─ https://www.microsoft.com/en-us/evalcenter/
├─ Windows 10 Enterprise (90-day trial)
└─ Download ISO

STEP 2: Create VM
├─ Name: SOC-Windows
├─ Type: Windows 10 (64-bit)
├─ RAM: 4 GB
├─ Disk: 40 GB
├─ Network: SOCLab
└─ Install Windows from ISO

STEP 3: Configure Windows
├─ Enable Event Logging (covered in Section 6)
├─ Install Splunk Universal Forwarder
└─ Configure to send logs to SIEM
```

### 3.5 Create VM #3: Kali Linux (Attacker)

```
STEP 1: Download Kali Linux
├─ https://www.kali.org/get-kali/
├─ Download pre-built VirtualBox image
└─ Import into VirtualBox

STEP 2: Configure
├─ RAM: 2 GB
├─ Network: SOCLab
└─ Default credentials: kali / kali
```

**Total Setup Time**: 2-3 hours
**Cost**: $0

---

## 4. Lab Setup Option 2: Cloud ($5-10/month)

### 4.1 AWS Free Tier Setup

```
CREATE EC2 INSTANCES:

1. Ubuntu SIEM (t2.medium)
   ├─ RAM: 4 GB
   ├─ Disk: 30 GB
   ├─ Install Splunk or ELK
   └─ Cost: Free tier (12 months)

2. Windows Server (t2.micro)
   ├─ RAM: 1 GB
   ├─ Disk: 30 GB
   ├─ Generate logs
   └─ Cost: Free tier

3. Kali Linux (t2.micro)
   ├─ From AWS Marketplace
   └─ Cost: Free tier

TOTAL COST (Year 1): $0 (AWS Free Tier)
TOTAL COST (Year 2+): $20-40/month
```

### 4.2 Azure Free Tier Alternative

```
AZURE VMS:
- B2s VM for SIEM (2 cores, 4 GB RAM)
- B1s VMs for Windows/Kali
- 12 months free tier

COST: $0 first year, ~$30/month after
```

---

## 5. Installing Splunk Free

### 5.1 On Ubuntu VM

```bash
# SSH into Ubuntu VM
ssh socadmin@192.168.100.10

# Download Splunk
cd /tmp
wget -O splunk.tgz "https://download.splunk.com/products/splunk/releases/9.1.2/linux/splunk-9.1.2-b6b9c8185839-Linux-x86_64.tgz"

# Extract to /opt
sudo tar xzf splunk.tgz -C /opt

# Start Splunk
sudo /opt/splunk/bin/splunk start --accept-license

# Create admin account when prompted
Username: admin
Password: (choose strong password)

# Enable boot-start
sudo /opt/splunk/bin/splunk enable boot-start

# Change to Free license
# Web UI: http://192.168.100.10:8000
# Settings → Licensing → Change License → Free
```

### 5.2 First Login & Configuration

```
1. Open browser: http://192.168.100.10:8000
2. Login: admin / (your password)
3. Skip tour
4. Create indexes:
   Settings → Indexes → New Index
   - Name: windows
   - Max Size: 10 GB
5. Settings → Data Inputs → Splunk Universal Forwarder
   - Port: 9997 (default)
   - Enable
```

---

## 6. Setting Up Log Sources

### 6.1 Configure Windows to Send Logs

**Install Splunk Universal Forwarder on Windows VM**:

```powershell
# Download Splunk UF
# https://www.splunk.com/en_us/download/universal-forwarder.html

# Install silently
msiexec.exe /i splunkforwarder.msi RECEIVING_INDEXER="192.168.100.10:9997" AGREETOLICENSE=Yes /quiet

# Configure inputs (C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf)
[WinEventLog://Security]
disabled = false
index = windows

[WinEventLog://System]
disabled = false
index = windows

[WinEventLog://Application]
disabled = false
index = windows

# Restart Splunk Forwarder
Restart-Service SplunkForwarder
```

**Verify Logs Arriving**:
```spl
index=windows
| stats count by host, sourcetype
```

### 6.2 Enable Advanced Windows Logging

```powershell
# Enable Process Creation logging (Event ID 4688)
auditpol /set /subcategory:"Process Creation" /success:enable /failure:enable

# Enable Command Line logging
reg add "HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\System\Audit" /v ProcessCreationIncludeCmdLine_Enabled /t REG_DWORD /d 1 /f

# Enable PowerShell logging
reg add "HKLM\SOFTWARE\Wow6432Node\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging" /v EnableScriptBlockLogging /t REG_DWORD /d 1 /f
```

---

## 7. Generating Test Data & Attack Scenarios

### 7.1 Scenario 1: Brute Force Attack

**From Kali VM**:
```bash
# Brute force RDP on Windows VM
hydra -l Administrator -P /usr/share/wordlists/rockyou.txt rdp://192.168.100.11
```

**In Splunk**:
```spl
# Detect the attack
index=windows EventCode=4625
| stats count by user, src_ip
| where count > 5
```

**Result**: You should see alerts for failed logins!

### 7.2 Scenario 2: Malicious PowerShell

**On Windows VM**:
```powershell
# Simulate attacker using encoded PowerShell
powershell.exe -enc SGVsbG8gV29ybGQ=  # "Hello World" encoded

# Simulate download cradle (safe test)
powershell.exe -nop -w hidden -c "Write-Host 'Test'"
```

**In Splunk**:
```spl
index=windows EventCode=4688 NewProcessName="*powershell.exe"
| rex field=CommandLine "(?i)(-enc|-nop|-w hidden)"
| where isnotnull(CommandLine)
```

**Result**: Detect the suspicious PowerShell execution!

### 7.3 Scenario 3: Port Scan

**From Kali**:
```bash
# Port scan Windows VM
nmap -p- 192.168.100.11
```

**In Splunk** (if you have firewall logs):
```spl
index=firewall action=deny
| stats dc(dest_port) as port_count by src_ip
| where port_count > 20
```

---

## 8. Practice Scenarios

### 8.1 Beginner Scenarios

```
SCENARIO 1: Find Failed Logins
Task: Create alert for >5 failed logins in 5 minutes
Expected Time: 15 minutes

SCENARIO 2: Detect Admin Account Usage
Task: Alert when any admin account logs in
Expected Time: 20 minutes

SCENARIO 3: Monitor Process Execution
Task: Find all .exe files executed from Downloads folder
Expected Time: 30 minutes
```

### 8.2 Intermediate Scenarios

```
SCENARIO 4: Detect Lateral Movement
Task: Find PsExec usage across network
Expected Time: 45 minutes

SCENARIO 5: Data Exfiltration
Task: Detect large outbound file transfers
Expected Time: 1 hour

SCENARIO 6: Privilege Escalation
Task: Alert when user added to Administrators group
Expected Time: 45 minutes
```

### 8.3 Advanced Scenarios

```
SCENARIO 7: APT Hunt
Task: Hunt for signs of APT29 TTPs
Expected Time: 2 hours

SCENARIO 8: Ransomware Detection
Task: Detect mass file encryption activity
Expected Time: 1.5 hours

SCENARIO 9: Complete Investigation
Task: Full timeline of simulated breach
Expected Time: 3 hours
```

---

## 9. Building Your Portfolio

### 9.1 Document Your Lab

**Create GitHub Repository**:
```
SOC-Home-Lab/
├── README.md (Lab overview)
├── architecture.md (Diagram + explanation)
├── splunk-queries/ (Your SPL queries)
│   ├── brute-force-detection.spl
│   ├── malware-detection.spl
│   └── exfiltration-detection.spl
├── dashboards/ (Export XML)
│   └── soc-operations-dashboard.xml
├── playbooks/ (Response procedures)
│   ├── brute-force-response.md
│   └── malware-response.md
└── investigations/ (Case studies)
    └── case-001-brute-force.md
```

### 9.2 Create Case Studies

**Example Case Study Template**:
```markdown
# Case Study: Brute Force Attack Investigation

## Alert
- Date: 2024-01-31
- Severity: High
- Alert: Multiple failed logins detected

## Investigation
1. Query used:
   ```spl
   index=windows EventCode=4625 | stats count by user, src_ip
   ```

2. Findings:
   - 47 failed login attempts
   - Source IP: 192.168.100.20 (Kali VM)
   - Target account: Administrator
   - Timeframe: 14:23-14:28 (5 minutes)

3. Evidence:
   [Screenshots of Splunk search results]

## Response
1. Blocked source IP
2. Reset Administrator password
3. Enabled account lockout policy

## Lessons Learned
- Need stricter account lockout
- Consider MFA for admin accounts
```

---

**Summary - File 10: Hands-On Lab**

You now have:
✅ Complete lab architecture blueprint
✅ Step-by-step setup instructions (free!)
✅ SIEM installation guide
✅ Log collection configuration
✅ Attack simulation scenarios
✅ Practice exercises (beginner → advanced)
✅ Portfolio building guidance

**BUILD THIS LAB, and you'll be job-ready!**

---

**CONGRATULATIONS! You've completed the entire SOC & SIEM Mastery Module!**

---

**End of File 10** (900+ lines)

---

## 🎉 MODULE 13: SOC & SIEM - COMPLETE! 🎉

**Total Content Created**: 10 comprehensive files covering every aspect of becoming a SOC professional!

Your complete SOC & SIEM curriculum is ready for mastery! 🚀