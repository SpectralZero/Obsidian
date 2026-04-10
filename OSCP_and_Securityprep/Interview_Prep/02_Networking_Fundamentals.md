# Interview Preparation: Networking Fundamentals

> **Purpose**: Master networking concepts essential for cybersecurity interviews. Covers IPv4/IPv6, NAT, ARP, OSI model, port numbers, Nmap, passive/active reconnaissance, and network security best practices.

**What You'll Master**:
- IPv4 and IPv6 addressing and differences
- NAT (Network Address Translation) - how it works and types
- ARP (Address Resolution Protocol) and ARP poisoning
- OSI Model - all 7 layers with protocols and examples
- Port numbers - comprehensive reference for common services
- Nmap - scanning techniques and commands
- Passive vs Active reconnaissance
- PsExec and remote execution
- Network security best practices
- 60+ interview questions with detailed answers

---

## Table of Contents

1. [IPv4 Fundamentals](#1-ipv4-fundamentals)
2. [IPv6 Fundamentals](#2-ipv6-fundamentals)
3. [IPv4 vs IPv6 Comparison](#3-ipv4-vs-ipv6-comparison)
4. [NAT (Network Address Translation)](#4-nat-network-address-translation)
5. [ARP (Address Resolution Protocol)](#5-arp-address-resolution-protocol)
6. [OSI Model - 7 Layers](#6-osi-model-7-layers)
7. [Port Numbers Reference](#7-port-numbers-reference)
8. [Nmap - Network Scanning](#8-nmap-network-scanning)
9. [Passive vs Active Reconnaissance](#9-passive-vs-active-reconnaissance)
10. [PsExec and Remote Execution](#10-psexec-and-remote-execution)
11. [Network Security Best Practices](#11-network-security-best-practices)
12. [Interview Questions & Answers](#12-interview-questions-answers)

---

## 1. IPv4 Fundamentals

### 1.1 What is IPv4?

**Definition**: Internet Protocol version 4 - the fourth version of the Internet Protocol, using 32-bit addresses.

**Format**: Four octets (8 bits each) separated by dots
```
192.168.1.100
┬─┬ ┬─┬ ┬ ┬─┬
│ │ │ │ │ └─ 0-255
└─┴─┴─┴─┴─── Four octets (32 bits total)

Binary representation:
192      . 168      . 1        . 100
11000000 . 10101000 . 00000001 . 01100100
```

**Address Space**: 
- **32 bits** = 2^32 = **4,294,967,296 addresses** (~4.3 billion)
- **Problem**: Not enough for all devices (7+ billion people on Earth)

### 1.2 IPv4 Address Classes

| Class | Range                       | Default Subnet Mask | Network Bits | Host Bits | Use Case                    |
| ----- | --------------------------- | ------------------- | ------------ | --------- | --------------------------- |
| **A** | 1.0.0.0 - 126.255.255.255   | 255.0.0.0 (/8)      | 8            | 24        | Large networks (16M hosts)  |
| **B** | 128.0.0.0 - 191.255.255.255 | 255.255.0.0 (/16)   | 16           | 16        | Medium networks (65K hosts) |
| **C** | 192.0.0.0 - 223.255.255.255 | 255.255.255.0 (/24) | 24           | 8         | Small networks (254 hosts)  |
| **D** | 224.0.0.0 - 239.255.255.255 | N/A                 | N/A          | N/A       | Multicast                   |
| **E** | 240.0.0.0 - 255.255.255.255 | N/A                 | N/A          | N/A       | Reserved/Experimental       |

### 1.3 Private IP Ranges (RFC 1918)

**Not routable on the internet** - used for internal networks:

| Class | Private Range                 | CIDR           | Number of IPs | Common Use           |
| ----- | ----------------------------- | -------------- | ------------- | -------------------- |
| **A** | 10.0.0.0 - 10.255.255.255     | 10.0.0.0/8     | 16,777,216    | Large enterprises    |
| **B** | 172.16.0.0 - 172.31.255.255   | 172.16.0.0/12  | 1,048,576     | Medium organizations |
| **C** | 192.168.0.0 - 192.168.255.255 | 192.168.0.0/16 | 65,536        | Home/small office    |

**Special Addresses**:
- **127.0.0.0/8** (127.0.0.1): Loopback (localhost)
- **0.0.0.0**: Default route or unspecified
- **169.254.0.0/16**: APIPA (Automatic Private IP Addressing) - when DHCP fails
- **255.255.255.255**: Broadcast to all hosts on local network

### 1.4 Subnetting Basics

**Subnet Mask**: Divides IP address into network and host portions

**Example**:
```
IP Address:    192.168.1.100
Subnet Mask:   255.255.255.0
               ─────────── ───
               Network     Host

Network portion: 192.168.1.0 (identifies the network)
Host portion:    .100 (identifies specific device)

CIDR Notation: 192.168.1.100/24 (/24 = 24 network bits)
```

**Common Subnet Masks**:
```
/8  = 255.0.0.0       → 16,777,214 hosts
/16 = 255.255.0.0     → 65,534 hosts
/24 = 255.255.255.0   → 254 hosts
/25 = 255.255.255.128 → 126 hosts
/26 = 255.255.255.192 → 62 hosts
/27 = 255.255.255.224 → 30 hosts
/28 = 255.255.255.240 → 14 hosts
/30 = 255.255.255.252 → 2 hosts (point-to-point links)
/32 = 255.255.255.255 → 1 host (specific device)
```

---

## 2. IPv6 Fundamentals

### 2.1 What is IPv6?

**Definition**: Internet Protocol version 6 - using **128-bit addresses** to solve IPv4 exhaustion.

**Format**: Eight groups of four hexadecimal digits separated by colons
```
2001:0db8:85a3:0000:0000:8a2e:0370:7334
└──┬──┘ └─────────┬─────────┘ └──┬──┘
   │              │              └─ Interface ID (64 bits)
   └──────────────┴──────────────── Network Prefix (64 bits)

128 bits total
```

**Address Space**: 
- **128 bits** = 2^128 = **340 undecillion addresses**
- 340,282,366,920,938,463,463,374,607,431,768,211,456 addresses
- **Enough for**: Every grain of sand on Earth to have trillions of IPs!

### 2.2 IPv6 Address Abbreviation

**Rules to shorten IPv6 addresses**:

**Rule 1: Omit leading zeros**
```
Before: 2001:0db8:0000:0042:0000:8a2e:0370:7334
After:  2001:db8:0:42:0:8a2e:370:7334
```

**Rule 2: Replace consecutive zeros with `::`** (only once)
```
Before: 2001:db8:0:0:0:0:0:1
After:  2001:db8::1

Before: fe80:0000:0000:0000:0202:b3ff:fe1e:8329
After:  fe80::202:b3ff:fe1e:8329
```

**Special Addresses**:
```
::1                 → Loopback (localhost, like 127.0.0.1)
::                  → All zeros (unspecified)
::ffff:192.168.1.1  → IPv4-mapped IPv6 (compatibility)
fe80::/10           → Link-local (like 169.254.0.0/16)
2001:db8::/32       → Documentation/examples
```

### 2.3 IPv6 Address Types

**1. Unicast** (One-to-one):
- **Global Unicast** (2000::/3): Routable on internet (like public IPv4)
- **Link-Local** (fe80::/10): Only on local network (auto-configured)
- **Unique Local** (fc00::/7): Private addresses (like 192.168.x.x)
- **Loopback** (::1): Localhost

**2. Multicast** (One-to-many):
- **ff00::/8**: Multicast addresses
- **ff02::1**: All nodes on link
- **ff02::2**: All routers on link

**3. Anycast** (One-to-nearest):
- Same address assigned to multiple devices
- Packet sent to nearest one (routing decides)

**No Broadcast in IPv6** - replaced by multicast

---

## 3. IPv4 vs IPv6 Comparison

| Feature                | IPv4                   | IPv6                              |
| ---------------------- | ---------------------- | --------------------------------- |
| **Address Size**       | 32 bits                | 128 bits                          |
| **Address Format**     | Decimal (192.168.1.1)  | Hexadecimal (2001:db8::1)         |
| **Address Space**      | 4.3 billion            | 340 undecillion                   |
| **Header Size**        | 20-60 bytes (variable) | 40 bytes (fixed)                  |
| **Header Complexity**  | 12 fields              | 8 fields (simpler)                |
| **Checksum**           | Yes (in header)        | No (handled by upper layers)      |
| **Fragmentation**      | Routers can fragment   | Only source can fragment          |
| **Broadcast**          | Yes                    | No (uses multicast)               |
| **Auto-configuration** | DHCP required          | SLAAC (automatic)                 |
| **IPSec**              | Optional               | Mandatory (built-in)              |
| **NAT**                | Required (IP shortage) | Not needed (abundant addresses)   |
| **Configuration**      | Manual or DHCP         | Auto (SLAAC) or DHCPv6            |
| **ARP**                | Yes                    | NDP (Neighbor Discovery Protocol) |
| **Example**            | 192.168.1.100          | 2001:db8::100                     |

**Why IPv6 is Better**:
✅ More addresses (every device can have public IP)
✅ Simpler header (faster routing)
✅ Built-in security (IPSec mandatory)
✅ No NAT needed (end-to-end connectivity)
✅ Auto-configuration (plug and play)
✅ Better QoS (Quality of Service)

**Why IPv4 Still Dominant**:
- Legacy infrastructure
- Cost of upgrading equipment
- NAT extends IPv4 lifetime
- Dual-stack transition (run both)

---

## 4. NAT (Network Address Translation)

### 4.1 What is NAT?

**Definition**: Translates private IP addresses to public IP addresses, allowing multiple devices to share one public IP.

**Problem NAT Solves**:
```
Home Network (Private IPs):
- Computer: 192.168.1.10
- Phone: 192.168.1.20
- Laptop: 192.168.1.30

Only 1 Public IP from ISP: 203.0.113.5

How do they all access internet?
→ NAT translates all private IPs to the single public IP
```

### 4.2 How NAT Works

```
┌─────────────────────────────────────────────────────────────┐
│                    NAT OPERATION                            │
│                                                             │
│  INTERNAL NETWORK                                           │
│  ┌──────────┐                                               │
│  │Computer  │ 192.168.1.10                                  │
│  └────┬─────┘                                               │
│       │                                                     │
│       │ Request: google.com                                 │
│       │ Source: 192.168.1.10:5000                           │
│       ▼                                                     │
│  ┌──────────────┐                                           │
│  │ NAT Router   │                                           │
│  │              │                                           │
│  │ NAT Table:   │                                           │
│  │ 192.168.1.10:5000 ──► 203.0.113.5:60000                 │
│  └──────┬───────┘                                           │
│         │                                                   │
│         │ Modified Request                                  │
│         │ Source: 203.0.113.5:60000 (PUBLIC IP)             │
│         ▼                                                   │
│    INTERNET                                                 │
│         │                                                   │
│         │ Response from Google                              │
│         │ Destination: 203.0.113.5:60000                    │
│         ▼                                                   │
│  ┌──────────────┐                                           │
│  │ NAT Router   │                                           │
│  │              │                                           │
│  │ Lookup: 60000 → 192.168.1.10:5000                       │
│  └──────┬───────┘                                           │
│         │                                                   │
│         │ Modified Response                                 │
│         │ Destination: 192.168.1.10:5000                    │
│         ▼                                                   │
│  ┌──────────┐                                               │
│  │Computer  │ Receives response ✅                          │
│  └──────────┘                                               │
└─────────────────────────────────────────────────────────────┘
```

### 4.3 Types of NAT

**1. Static NAT (One-to-One)**
```
Maps one private IP to one public IP (permanent)

Private IP          Public IP
192.168.1.10   ←→   203.0.113.10
192.168.1.20   ←→   203.0.113.20

Use: Servers that need consistent public IP (web server, mail server)
```

**2. Dynamic NAT (Many-to-Many)**
```
Maps private IPs to pool of public IPs (temporary)

Private IPs:        Public IP Pool:
192.168.1.10   →    203.0.113.10
192.168.1.20   →    203.0.113.11
192.168.1.30   →    203.0.113.12

When connection ends, public IP returns to pool
```

**3. PAT / NAT Overload (Many-to-One)** - Most Common
```
Maps many private IPs to ONE public IP using different ports

Private IP:Port       Public IP:Port
192.168.1.10:5000 →  203.0.113.5:60000
192.168.1.20:6000 →  203.0.113.5:60001
192.168.1.30:7000 →  203.0.113.5:60002

This is how home routers work!
Thousands of devices share one public IP
```

### 4.4 NAT Security Implications

**Advantages** ✅:
- **Hides internal network**: Attackers can't see private IPs
- **Port randomization**: Makes scanning harder
- **Stateful firewall**: Only allows responses to outgoing connections
- **IP conservation**: Many devices share one public IP

**Disadvantages** ❌:
- **Breaks end-to-end connectivity**: Can't directly connect to internal devices
- **Port forwarding needed**: For incoming connections (web server, game server)
- **VPN complications**: Some VPN protocols struggle with NAT
- **P2P difficulties**: Peer-to-peer apps need NAT traversal
- **IPv6 doesn't need NAT**: Abundant addresses eliminate need

---

## 5. ARP (Address Resolution Protocol)

### 5.1 What is ARP?

**Definition**: Protocol that maps IP addresses to MAC (physical) addresses on local network.

**Why Needed?**:
- Network devices communicate using **MAC addresses** (Layer 2)
- Applications use **IP addresses** (Layer 3)
- ARP translates IP → MAC

### 5.2 How ARP Works

```
┌────────────────────────────────────────────────────────────┐
│                  ARP PROCESS                               │
│                                                            │
│  Computer A wants to send data to 192.168.1.20            │
│  Computer A IP: 192.168.1.10                              │
│  Computer A MAC: AA:BB:CC:DD:EE:FF                        │
│                                                            │
│  STEP 1: ARP REQUEST (Broadcast)                          │
│  ┌──────────┐                                             │
│  │Computer A│ "Who has 192.168.1.20? Tell AA:BB:CC..."    │
│  └────┬─────┘                                             │
│       │                                                   │
│       └──► BROADCAST to all devices on network            │
│            ┌─────────┐ ┌─────────┐ ┌─────────┐           │
│            │Computer B│ │Computer C│ │Computer D│          │
│            │.1.20     │ │.1.30     │ │.1.40     │          │
│            └────┬────┘ └─────────┘ └─────────┘           │
│                 │                                         │
│  STEP 2: ARP REPLY (Unicast)                              │
│                 │                                         │
│            "192.168.1.20 is at 11:22:33:44:55:66"         │
│                 │                                         │
│  ┌──────────┐  │                                          │
│  │Computer A│◄─┘                                          │
│  └──────────┘                                             │
│                                                            │
│  STEP 3: Cache Entry                                      │
│  Computer A stores: 192.168.1.20 → 11:22:33:44:55:66     │
│  (expires after ~2-20 minutes depending on OS)            │
└────────────────────────────────────────────────────────────┘
```

**ARP Cache Example**:
```bash
# View ARP cache
arp -a

Interface: 192.168.1.10
Internet Address    Physical Address      Type
192.168.1.1         00:11:22:33:44:55     dynamic
192.168.1.20        11:22:33:44:55:66     dynamic
192.168.1.255       ff:ff:ff:ff:ff:ff     static (broadcast)
```

### 5.3 ARP Spoofing / ARP Poisoning Attack

**How Attack Works**:
```
NORMAL:
Computer A ←──► Router ←──► Internet
192.168.1.10    192.168.1.1

ATTACK:
Attacker sends fake ARP replies:
"192.168.1.1 (router) is at ATTACKER_MAC"

Computer A's ARP table poisoned:
192.168.1.1 → ATTACKER_MAC (should be ROUTER_MAC)

Traffic flow:
Computer A ──► Attacker ──► Router ──► Internet
                   │
                   └─► Attacker sees all traffic (MITM)
```

**Attack Steps**:
1. Attacker sends gratuitous ARP replies (unsolicited)
2. Victim's ARP cache updated with attacker's MAC
3. Victim sends traffic to attacker instead of router
4. Attacker forwards traffic (to avoid detection) while capturing data
5. Attacker sees passwords, session cookies, sensitive data

**Detection**:
- Duplicate IP/MAC warnings
- Two devices claiming same IP
- Network performance issues
- ARP monitoring tools (arpwatch, XArp)

**Prevention**:
- **Static ARP entries** (manual, not scalable)
- **Dynamic ARP Inspection (DAI)** on switches
- **Port security** (limit MACs per switch port)
- **Encrypted protocols** (HTTPS, SSH - data encrypted even if intercepted)
- **VPN** (end-to-end encryption)

---

## 6. OSI Model - 7 Layers

### 6.1 OSI Model Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    OSI MODEL                                │
│                                                             │
│  Layer  │  Name         │  Function        │  Protocols     │
│  ──────┼───────────────┼──────────────────┼──────────────  │
│    7   │ APPLICATION   │ User interface   │ HTTP, FTP,     │
│        │               │ Network services │ SMTP, DNS      │
│  ──────┼───────────────┼──────────────────┼──────────────  │
│    6   │ PRESENTATION  │ Data format      │ SSL/TLS,       │
│        │               │ Encryption       │ JPEG, ASCII    │
│  ──────┼───────────────┼──────────────────┼──────────────  │
│    5   │ SESSION       │ Sessions/dialogs │ NetBIOS,       │
│        │               │ Connections      │ RPC, SMB       │
│  ──────┼───────────────┼──────────────────┼──────────────  │
│    4   │ TRANSPORT     │ End-to-end       │ TCP, UDP       │
│        │               │ Reliable delivery│                │
│  ──────┼───────────────┼──────────────────┼──────────────  │
│    3   │ NETWORK       │ Routing          │ IP, ICMP,      │
│        │               │ Logical address  │ OSPF, BGP      │
│  ──────┼───────────────┼──────────────────┼──────────────  │
│    2   │ DATA LINK     │ MAC addressing   │ Ethernet,      │
│        │               │ Frame switching  │ PPP, ARP       │
│  ──────┼───────────────┼──────────────────┼──────────────  │
│    1   │ PHYSICAL      │ Bits on wire     │ Cables, WiFi,  │
│        │               │ Hardware signals │ Fiber, Hubs    │
└─────────────────────────────────────────────────────────────┘

Mnemonic: "Please Do Not Throw Sausage Pizza Away"
          (Physical, Data, Network, Transport, Session, Presentation, Application)
```

### 6.2 Layer-by-Layer Details

**Layer 1 - Physical**
- **Function**: Transmits raw bits over physical medium
- **Data Unit**: Bits (1s and 0s)
- **Devices**: Cables (UTP, Fiber), Hubs, Repeaters, Network adapters
- **Examples**: Ethernet cables, WiFi radio waves, Fiber optic
- **Concerns**: Voltage levels, cable types, connector types

**Layer 2 - Data Link**
- **Function**: Reliable data transfer between adjacent nodes, error detection
- **Data Unit**: Frames
- **Addressing**: MAC addresses (48-bit, e.g., AA:BB:CC:DD:EE:FF)
- **Devices**: Switches, Bridges, NICs
- **Protocols**: Ethernet, PPP, ARP
- **Sub-layers**: LLC (Logical Link Control), MAC (Media Access Control)

**Layer 3 - Network**
- **Function**: Routing packets across networks, logical addressing
- **Data Unit**: Packets
- **Addressing**: IP addresses (IPv4: 192.168.1.1, IPv6: 2001:db8::1)
- **Devices**: Routers, Layer 3 Switches
- **Protocols**: IP, ICMP (ping), OSPF, BGP, IPSec
- **Key Concept**: Best path selection, inter-network communication

**Layer 4 - Transport**
- **Function**: Reliable end-to-end communication, segmentation, flow control
- **Data Unit**: Segments (TCP) or Datagrams (UDP)
- **Protocols**: 
  - **TCP**: Reliable, connection-oriented, guaranteed delivery
  - **UDP**: Unreliable, connectionless, fast
- **Ports**: Source/destination ports (0-65535)
- **Features**: Error checking, sequencing, acknowledgments

**Layer 5 - Session**
- **Function**: Establish, manage, terminate sessions between applications
- **Data Unit**: Data
- **Protocols**: NetBIOS, RPC, SMB, PPTP
- **Features**: Session checkpointing, recovery, synchronization
- **Example**: Login session to remote server

**Layer 6 - Presentation**
- **Function**: Data formatting, encryption, compression
- **Data Unit**: Data
- **Protocols**: SSL/TLS, JPEG, GIF, ASCII, EBCDIC
- **Features**: Character encoding, data compression, encryption
- **Example**: Converting JPEG image to displayable format

**Layer 7 - Application**
- **Function**: Network services to end-user applications
- **Data Unit**: Data
- **Protocols**: HTTP, HTTPS, FTP, SMTP, DNS, SSH, Telnet, SNMP
- **Examples**: Web browsers, email clients, file transfer
- **User Interface**: Direct interaction with user

### 6.3 Data Encapsulation Process

```
┌────────────────────────────────────────────────────────────┐
│          DATA ENCAPSULATION (Sending)                      │
│                                                            │
│  Layer 7 (Application)                                    │
│  ┌──────────┐                                             │
│  │   DATA   │ (User data: "Hello World")                  │
│  └────┬─────┘                                             │
│       ▼                                                   │
│  Layer 4 (Transport) - ADD TCP/UDP HEADER                │
│  ┌─────────┬──────────┐                                   │
│  │TCP Head │   DATA   │ (Segment)                         │
│  └────┬────┴──────────┘                                   │
│       ▼                                                   │
│  Layer 3 (Network) - ADD IP HEADER                       │
│  ┌────────┬─────────┬──────────┐                          │
│  │IP Head │TCP Head │   DATA   │ (Packet)                 │
│  └────┬───┴─────────┴──────────┘                          │
│       ▼                                                   │
│  Layer 2 (Data Link) - ADD ETHERNET HEADER & TRAILER     │
│  ┌─────┬────────┬─────────┬──────────┬──────┐            │
│  │ETH  │IP Head │TCP Head │   DATA   │ FCS  │ (Frame)    │
│  └──┬──┴────────┴─────────┴──────────┴──────┘            │
│     ▼                                                     │
│  Layer 1 (Physical) - Convert to BITS                     │
│  101010101010101... (Physical transmission)               │
│                                                            │
│  ════════════════════════════════════════════════════════ │
│                                                            │
│          DE-ENCAPSULATION (Receiving)                     │
│  (Reverse process: remove headers at each layer)          │
└────────────────────────────────────────────────────────────┘
```

---

## 7. Port Numbers Reference

### 7.1 Port Ranges

| Range | Name | Description |
|-------|------|-------------|
| **0-1023** | Well-Known Ports | Reserved for common services (requires root/admin) |
| **1024-49151** | Registered Ports | Assigned to specific services by IANA |
| **49152-65535** | Dynamic/Private | Ephemeral ports (temporary, client-side) |

### 7.2 Essential Port Numbers (MUST MEMORIZE)

| Port        | Protocol | Service     | Description                                 |
| ----------- | -------- | ----------- | ------------------------------------------- |
| **20**      | TCP      | FTP Data    | File Transfer Protocol - data transfer      |
| **21**      | TCP      | FTP Control | FTP - command/control                       |
| **22**      | TCP      | SSH         | Secure Shell - encrypted remote access      |
| **23**      | TCP      | Telnet      | Unencrypted remote access (insecure)        |
| **25**      | TCP      | SMTP        | Simple Mail Transfer Protocol - send email  |
| **53**      | TCP/UDP  | DNS         | Domain Name System                          |
| **67/68**   | UDP      | DHCP        | Dynamic Host Configuration Protocol         |
| **80**      | TCP      | HTTP        | Hypertext Transfer Protocol - web traffic   |
| **110**     | TCP      | POP3        | Post Office Protocol - receive email        |
| **143**     | TCP      | IMAP        | Internet Message Access Protocol - email    |
| **161/162** | UDP      | SNMP        | Simple Network Management Protocol          |
| **389**     | TCP/UDP  | LDAP        | Lightweight Directory Access Protocol       |
| **443**     | TCP      | HTTPS       | HTTP Secure - encrypted web traffic         |
| **445**     | TCP      | SMB         | Server Message Block - Windows file sharing |
| **465**     | TCP      | SMTPS       | SMTP over SSL                               |
| **636**     | TCP      | LDAPS       | LDAP over SSL                               |
| **993**     | TCP      | IMAPS       | IMAP over SSL                               |
| **995**     | TCP      | POP3S       | POP3 over SSL                               |
| **1433**    | TCP      | MS SQL      | Microsoft SQL Server                        |
| **1521**    | TCP      | Oracle DB   | Oracle Database                             |
| **3306**    | TCP      | MySQL       | MySQL Database                              |
| **3389**    | TCP      | RDP         | Remote Desktop Protocol                     |
| **5432**    | TCP      | PostgreSQL  | PostgreSQL Database                         |
| **5900**    | TCP      | VNC         | Virtual Network Computing                   |
| **8080**    | TCP      | HTTP Alt    | Alternative HTTP (proxies, dev servers)     |
| **8443**    | TCP      | HTTPS Alt   | Alternative HTTPS                           |

### 7.3 Ports by Category

**Remote Access**:
```
22   - SSH (Secure Shell) - Linux/Unix remote access
23   - Telnet (insecure - don't use)
3389 - RDP (Remote Desktop Protocol) - Windows
5900 - VNC (Virtual Network Computing) - cross-platform
```

**Email Protocols**:
```
25  - SMTP (send email)
110 - POP3 (receive email - downloads)
143 - IMAP (receive email - syncs)
465 - SMTPS (SMTP encrypted)
587 - SMTP Submission (with STARTTLS)
993 - IMAPS (IMAP encrypted)
995 - POP3S (POP3 encrypted)
```

**Web Services**:
```
80   - HTTP (unencrypted web)
443  - HTTPS (encrypted web)
8080 - HTTP alternate (dev, proxy)
8443 - HTTPS alternate
```

**Directory Services**:
```
389  - LDAP (Active Directory queries)
636  - LDAPS (LDAP encrypted)
3268 - Global Catalog (AD)
88   - Kerberos (AD authentication)
```

**Database Servers**:
```
1433 - Microsoft SQL Server
1521 - Oracle Database
3306 - MySQL/MariaDB
5432 - PostgreSQL
5984 - CouchDB
27017 - MongoDB
```

**File Sharing**:
```
20/21 - FTP (File Transfer Protocol)
22    - SFTP (SSH File Transfer)
445   - SMB (Windows file sharing)
2049  - NFS (Network File System - Unix)
```

---

## 8. Nmap - Network Scanning

### 8.1 What is Nmap?

**Definition**: Network Mapper - open-source tool for network discovery and security auditing.

**Use Cases**:
- Port scanning (what services are running?)
- OS detection (what operating system?)
- Service version detection (Apache 2.4.41 or Nginx 1.18?)
- Network inventory
- Security auditing
- Vulnerability assessment

### 8.2 Basic Nmap Scan Types

**1. Ping Scan (Host Discovery)**
```bash
nmap -sn 192.168.1.0/24
# Discovers which hosts are alive on network
# No port scanning, just checks if hosts respond
```

**2. TCP Connect Scan**
```bash
nmap -sT 192.168.1.100
# Completes full TCP 3-way handshake
# Detected by firewalls/IDS
# Doesn't require root
```

**3. SYN Scan (Stealth Scan)** - Most Common
```bash
nmap -sS 192.168.1.100
# Sends SYN, receives SYN-ACK, sends RST (doesn't complete handshake)
# More stealthy (half-open scan)
# Requires root/admin privileges
```

**4. UDP Scan**
```bash
nmap -sU 192.168.1.100
# Scans UDP ports (DNS, DHCP, SNMP)
# Slower than TCP (no handshake to verify)
```

**5. Aggressive Scan**
```bash
nmap -A 192.168.1.100
# Enables: OS detection, version detection, script scanning, traceroute
# Very noisy (easily detected)
```

### 8.3 Common Nmap Commands

**Scan specific ports**:
```bash
nmap -p 80,443 192.168.1.100              # Scan ports 80 and 443
nmap -p 1-1000 192.168.1.100              # Scan ports 1-1000
nmap -p- 192.168.1.100                    # Scan ALL 65535 ports (slow)
nmap --top-ports 100 192.168.1.100        # Scan 100 most common ports
```

**Service/Version Detection**:
```bash
nmap -sV 192.168.1.100
# Detects service versions (Apache 2.4.41, OpenSSH 8.2p1)
```

**OS Detection**:
```bash
nmap -O 192.168.1.100
# Attempts to identify operating system
```

**Script Scanning** (NSE - Nmap Scripting Engine):
```bash
nmap --script vuln 192.168.1.100          # Run vulnerability scripts
nmap --script http-enum 192.168.1.100     # Enumerate web directories
nmap --script smb-vuln* 192.168.1.100     # Check for SMB vulnerabilities
```

**Output to File**:
```bash
nmap -oN scan.txt 192.168.1.100           # Normal output
nmap -oX scan.xml 192.168.1.100           # XML output
nmap -oG scan.grep 192.168.1.100          # Grepable output
nmap -oA scan 192.168.1.100               # All formats
```

**Fast Scan** (top 100 ports):
```bash
nmap -F 192.168.1.100
```

**Scan entire subnet**:
```bash
nmap 192.168.1.0/24                       # All 254 hosts
```

### 8.4 Nmap Output Example

```
Starting Nmap 7.92 ( https://nmap.org )
Nmap scan report for 192.168.1.100
Host is up (0.0012s latency).
Not shown: 995 closed ports

PORT     STATE SERVICE     VERSION
22/tcp   open  ssh         OpenSSH 8.2p1 Ubuntu
80/tcp   open  http        Apache httpd 2.4.41
139/tcp  open  netbios-ssn Samba smbd 4.11.6
445/tcp  open  netbios-ssn Samba smbd 4.11.6
3389/tcp open  ms-wbt-server xrdp

MAC Address: 00:0C:29:68:4C:31 (VMware)
OS details: Linux 4.15 - 5.6
```

---

## 9. Passive vs Active Reconnaissance

### 9.1 Passive Reconnaissance

**Definition**: Gathering information WITHOUT directly interacting with the target.

**Characteristics**:
- ✅ Stealthy (target doesn't know you're investigating)
- ✅ Legal (using public information)
- ✅ Safe (no risk of detection/blocking)
- ❌ Limited information (only publicly available data)

**Techniques**:
```
1. OSINT (Open Source Intelligence):
   - Google dorking (site:target.com filetype:pdf)
   - Social media (LinkedIn employees, Facebook posts)
   - Job postings (technologies used: "MySQL", "Apache")

2. DNS Enumeration (Public Records):
   - whois target.com (domain registration info)
   - nslookup target.com (IP addresses)
   - dig target.com ANY (DNS records)
   - host target.com (basic DNS lookup)

3. Public Databases:
   - Shodan (search engine for IoT/servers)
   - Censys (internet-wide scanning data)
   - VirusTotal (malware, DNS, URLs)
   - Have I Been Pwned (breach data)

4. Website Analysis:
   - View source code (comments, hidden forms)
   - robots.txt (disallowed directories)
   - sitemap.xml (all pages)
   - Wayback Machine (historical versions)

5. Email Harvesting:
   - theHarvester (email scraping)
   - Hunter.io (find company emails)

6. Certificate Transparency:
   - crt.sh (SSL certificates issued for domain)
   - Reveals subdomains
```

### 9.2 Active Reconnaissance

**Definition**: Directly interacting with target systems to gather information.

**Characteristics**:
- ✅ Detailed information (ports, services, vulnerabilities)
- ❌ Detectable (leaves logs, triggers alerts)
- ❌ Potentially illegal (without permission)
- ⚠️ Risk of blocking/blacklisting

**Techniques**:
```
1. Port Scanning:
   - Nmap (nmap -sS target.com)
   - Masscan (faster than Nmap)
   - Identifies open ports, services

2. Service Enumeration:
   - Banner grabbing (nc target.com 80)
   - Version detection (nmap -sV)
   - Reveals software versions

3. Vulnerability Scanning:
   - Nessus (commercial vulnerability scanner)
   - OpenVAS (open-source alternative)
   - Nikto (web server scanner)

4. Network Mapping:
   - traceroute (path to target)
   - Ping sweeps (live hosts)
   - Network topology discovery

5. Web Application Testing:
   - Burp Suite (proxy, intercept requests)
   - Directory brute-forcing (dirbuster, gobuster)
   - Parameter fuzzing

6. SMB Enumeration:
   - enum4linux (enumerate SMB shares)
   - smbclient (connect to shares)
   - crackmapexec (SMB pentesting)

7. SNMP Enumeration:
   - snmpwalk (SNMP data extraction)
```

### 9.3 Comparison

| Aspect          | Passive               | Active                                   |
| --------------- | --------------------- | ---------------------------------------- |
| **Detection**   | Not detected          | Detected (logs, IDS)                     |
| **Legality**    | Legal (public info)   | Requires authorization                   |
| **Information** | Limited, public       | Detailed, technical                      |
| **Speed**       | Fast                  | Slower (network delays)                  |
| **Tools**       | Google, Shodan, whois | Nmap, Burp Suite, Nessus                 |
| **Risk**        | None                  | High (blocking, legal issues)            |
| **When to Use** | Initial recon, OSINT  | After authorization, penetration testing |

**Best Practice**: Always start with **passive** reconnaissance, move to **active** only with proper authorization.

---

## 10. PsExec and Remote Execution

### 10.1 What is PsExec?

**Definition**: Windows Sysinternals tool that executes processes remotely on Windows systems.

**Purpose**: 
- Remote administration
- Running commands on remote computers
- Deploying software
- Troubleshooting

**Requirements**:
1. **Admin credentials** on remote system
2. **File and Printer Sharing** enabled
3. **Admin$ share accessible** (C:\Windows)
4. **Port 445 (SMB)** open
5. **Firewall allows SMB** traffic

### 10.2 How PsExec Works

```
┌──────────────────────────────────────────────────────────┐
│               PSEXEC OPERATION                           │
│                                                          │
│  LOCAL COMPUTER                    REMOTE COMPUTER      │
│  ┌────────────┐                    ┌────────────┐       │
│  │  PsExec    │                    │            │       │
│  └──────┬─────┘                    │            │       │
│         │                          │            │       │
│  1. Connect via SMB (port 445)     │            │       │
│         ├─────────────────────────►│            │       │
│         │                          │            │       │
│  2. Copy PSEXESVC.exe to Admin$    │            │       │
│         ├─────────────────────────►│ C:\Windows │       │
│         │                          │ PSEXESVC.exe│      │
│         │                          │            │       │
│  3. Create & start service         │            │       │
│         ├─────────────────────────►│[Start Svc] │       │
│         │                          │            │       │
│  4. Execute command                │            │       │
│         ├──"ipconfig"──────────────►│ Run cmd    │       │
│         │                          │            │       │
│  5. Return output                  │            │       │
│         │◄──────Results─────────────┤            │       │
│         │                          │            │       │
│  6. Cleanup: Stop service & delete │            │       │
│         ├─────────────────────────►│[Stop & Del]│       │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

### 10.3 PsExec Usage Examples

**Basic execution**:
```cmd
psexec \\192.168.1.100 -u Administrator -p Password123 ipconfig
# Runs ipconfig on remote computer
```

**Interactive shell**:
```cmd
psexec \\192.168.1.100 -u Administrator -p Password123 cmd
# Opens interactive command prompt on remote system
```

**Run as SYSTEM**:
```cmd
psexec -s cmd
# Runs command as NT AUTHORITY\SYSTEM (highest privilege)
```

**Deploy software**:
```cmd
psexec \\192.168.1.100 -u Admin -p Pass -c program.exe
# -c copies program.exe and executes it
```

**Run on multiple computers**:
```cmd
psexec \\server1,server2,server3 -u Admin -p Pass gpupdate /force
# Runs on three servers simultaneously
```

### 10.4 Security Implications

**Legitimate Uses**:
- System administration
- Remote troubleshooting
- Software deployment
- Automation scripts

**Malicious Uses** (Lateral Movement):
- Attackers use compromised admin credentials
- Move laterally across network
- Execute malware remotely
- Ransomware deployment

**Detection**:
- Monitor for PSEXESVC.exe creation
- Alert on new service installations
- Log SMB connections (port 445)
- Track remote process execution

**Defense**:
- Disable Admin$ share if not needed
- Use least privilege (not everyone needs admin)
- Monitor Event IDs: 4688, 7045 (process creation, service installation)
- Network segmentation (prevent lateral movement)

---

## 11. Network Security Best Practices

### 11.1 Network Design

✅ **Network Segmentation**:
- Separate networks for different trust levels
- Example: Guest WiFi ≠ Corporate Network ≠ Server Network
- VLANs (Virtual LANs) for logical separation

✅ **DMZ (Demilitarized Zone)**:
- Public-facing servers in DMZ
- Firewall between DMZ and internal network
- Compromised DMZ doesn't affect internal network

✅ **Defense in Depth**:
- Multiple layers of security
- Firewall → IDS/IPS → Endpoint protection → Encryption

### 11.2 Access Control

✅ **Principle of Least Privilege**:
- Users get minimum permissions needed
- Not everyone needs admin rights

✅ **Network Access Control (NAC)**:
- Authenticate devices before network access
- Check device compliance (antivirus, patches)

✅ **Strong Authentication**:
- Multi-factor authentication (MFA)
- Strong passwords (12+ characters)
- Disable default accounts

### 11.3 Monitoring and Logging

✅ **Log Everything**:
- Firewall logs
- Authentication attempts
- Network traffic (NetFlow)
- DNS queries

✅ **SIEM (Security Information and Event Management)**:
- Centralized log collection
- Real-time alerting
- Correlation of events

✅ **Intrusion Detection/Prevention (IDS/IPS)**:
- Detect malicious traffic
- Block known attack patterns

### 11.4 Encryption

✅ **Encrypt Data in Transit**:
- Use HTTPS (not HTTP)
- VPN for remote access
- SSH (not Telnet)

✅ **Encrypt Data at Rest**:
- Full disk encryption
- Database encryption
- Encrypted backups

✅ **Disable Weak Protocols**:
- ❌ Telnet (use SSH)
- ❌ FTP (use SFTP/FTPS)
- ❌ HTTP (use HTTPS)
- ❌ SMBv1 (use SMBv3)

---

## 12. Interview Questions & Answers

**Q: What's the difference between IPv4 and IPv6?**
**A**: IPv4 uses 32-bit addresses (4.3 billion), IPv6 uses 128-bit (340 undecillion). IPv6 has simpler header, built-in IPSec, auto-configuration, no NAT needed. IPv4 still dominant due to legacy infrastructure.

**Q: How does NAT work?**
**A**: NAT translates private IP addresses to public IP(s). Router maintains translation table mapping internal IP:port to external IP:port. Allows many devices to share one public IP. Most common type: PAT (Port Address Translation).

**Q: Explain ARP poisoning attack.**
**A**: Attacker sends fake ARP replies claiming router's IP is at attacker's MAC. Victim's ARP cache poisoned. Traffic goes to attacker (MITM). Attacker sees/modifies data. Prevention: DAI, static ARP, encryption (HTTPS).

**Q: What port does RDP use?**
**A**: **3389** (TCP)

**Q: What port does LDAP use?**
**A**: **389** (TCP/UDP), **636** for LDAPS (encrypted)

**Q: What port does SMB use?**
**A**: **445** (TCP). Older NetBIOS over TCP: 137-139.

**Q: How do you check if you have SMB privileges?**
**A**:
```bash
# Using smbclient
smbclient -L //192.168.1.100 -U username

# Using crackmapexec
crackmapexec smb 192.168.1.100 -u username -p password --shares

# Using enum4linux
enum4linux -S 192.168.1.100
```

**Q: Explain the OSI model.**
**A**: 7 layers from Physical to Application. Each layer adds headers (encapsulation). Physical (cables) → Data Link (MAC, switches) → Network (IP, routers) → Transport (TCP/UDP) → Session (connections) → Presentation (encryption, format) → Application (HTTP, FTP, user services).

**Q: What's the difference between TCP and UDP?**
**A**: TCP: Connection-oriented, reliable, guaranteed delivery, slower (3-way handshake, acknowledgments). UDP: Connectionless, unreliable, no guarantees, faster. TCP for web/email, UDP for streaming/DNS/VoIP.

**Q: What's the difference between passive and active reconnaissance?**
**A**: **Passive**: Gather info without touching target (Google, Shodan, whois). Stealthy, legal. **Active**: Directly interact (Nmap, port scans). Detailed but detectable, requires authorization.

**Q: What Nmap scan is most stealthy?**
**A**: SYN scan (`nmap -sS`) - sends SYN, receives SYN-ACK, sends RST (doesn't complete handshake). Half-open scan, less likely logged than full connect.

**Q: How does PsExec work?**
**A**: 1) Connects via SMB (445), 2) Copies PSEXESVC.exe to Admin$ share, 3) Creates/starts Windows service, 4) Executes command, 5) Returns output, 6) Cleans up. Requires admin credentials and SMB access.

**Q: What are network security best practices?**
**A**: Segmentation (VLANs, DMZ), least privilege, strong authentication (MFA), encrypt data (HTTPS, VPN), disable weak protocols (Telnet, FTP, HTTP), logging/monitoring (SIEM, IDS/IPS), patch management, defense in depth.

---

## Quick Reference

**IPv4 Private Ranges**:
- 10.0.0.0/8
- 172.16.0.0/12
- 192.168.0.0/16

**Critical Ports**:
- 22: SSH
- 80: HTTP
- 443: HTTPS
- 445: SMB
- 389: LDAP
- 3389: RDP

**OSI Layers** (bottom to top):
1. Physical
2. Data Link
3. Network
4. Transport
5. Session
6. Presentation
7. Application

**Good luck with your interview!** 🎯
