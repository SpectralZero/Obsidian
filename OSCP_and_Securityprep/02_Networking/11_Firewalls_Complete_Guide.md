# Firewalls: The Gatekeeper of Every Network — From Packet Filtering to AI-Driven Defense

---
title: "Firewalls: The Gatekeeper of Every Network"
parent: "[[02_Networking]]"
tags:
  - network
  - firewalls
  - iptables
  - ufw
  - network-security
  - defense-in-depth
created: 2026-01-24
updated: 2026-03-04
---

> **Executive Summary**: A firewall is the single most critical chokepoint in any network — it decides what traffic lives and what traffic dies. Understanding how firewalls work internally, how to configure them from scratch, and how to audit their rules is non-negotiable for any penetration tester. You cannot bypass what you do not understand, and you cannot defend what you cannot build. This guide takes you from absolute zero to building enterprise-grade firewall architectures, writing bulletproof rule sets, and auditing configurations for weaknesses — the foundational knowledge that separates script kiddies from professionals.

---

## Table of Contents
1. [Learning Objectives](#1-learning-objectives)
2. [What Is a Firewall — The Absolute Fundamentals](#2-what-is-a-firewall--the-absolute-fundamentals)
3. [How Firewalls Work Internally — Packet Processing Pipeline](#3-how-firewalls-work-internally--packet-processing-pipeline)
4. [Firewall Types — Complete Classification](#4-firewall-types--complete-classification)
5. [Network Firewalls and Gateway Protection](#5-network-firewalls-and-gateway-protection)
6. [The Linux Firewall Stack — Netfilter Architecture](#6-the-linux-firewall-stack--netfilter-architecture)
7. [iptables — The Classic Workhorse](#7-iptables--the-classic-workhorse)
8. [nftables — The Modern Successor](#8-nftables--the-modern-successor)
9. [UFW — Uncomplicated Firewall Deep Dive](#9-ufw--uncomplicated-firewall-deep-dive)
10. [firewalld — Zone-Based Dynamic Firewall](#10-firewalld--zone-based-dynamic-firewall)
11. [Windows Firewall — Defender Firewall with Advanced Security](#11-windows-firewall--defender-firewall-with-advanced-security)
12. [pfSense and OPNsense — Dedicated Firewall Appliances](#12-pfsense-and-opnsense--dedicated-firewall-appliances)
13. [Cloud Firewalls — AWS, Azure, and GCP](#13-cloud-firewalls--aws-azure-and-gcp)
14. [Firewall Design Patterns — DMZ, Zones, and Segmentation](#14-firewall-design-patterns--dmz-zones-and-segmentation)
15. [NAT/PAT and Firewall Integration](#15-natpat-and-firewall-integration)
16. [VPN Integration with Firewalls](#16-vpn-integration-with-firewalls)
17. [Firewall Policy Management and Best Practices](#17-firewall-policy-management-and-best-practices)
18. [Firewall Auditing — Finding Weaknesses Before Attackers Do](#18-firewall-auditing--finding-weaknesses-before-attackers-do)
19. [Firewall Troubleshooting — Diagnosing Connectivity Issues](#19-firewall-troubleshooting--diagnosing-connectivity-issues)
20. [Common Mistakes — What Gets Networks Compromised](#20-common-mistakes--what-gets-networks-compromised)
21. [Hands-On Labs](#21-hands-on-labs)
22. [Interview Questions — Prove Your Mastery](#22-interview-questions--prove-your-mastery)
23. [Critical Analysis and Checkpoints](#23-critical-analysis-and-checkpoints)

---

## 1. Learning Objectives

After mastering this module, you will be able to:

- **Define** what a firewall is, how it processes packets, and where it sits in network architecture
- **Classify** firewall types (Layer 3, Layer 7, NGFW, AI-based) and explain when each is deployed
- **Build** complete iptables rule sets from scratch with chains, tables, and targets
- **Configure** UFW, firewalld, and nftables for production Linux systems
- **Design** DMZ architectures, network segmentation, and multi-zone firewall topologies
- **Implement** NAT/PAT rules and integrate VPN tunnels through firewall policies
- **Administer** Windows Defender Firewall via GUI and PowerShell/netsh
- **Deploy** pfSense/OPNsense virtual appliances as dedicated perimeter firewalls
- **Configure** cloud-native firewalls (AWS Security Groups, Azure NSGs, GCP Firewall Rules)
- **Audit** firewall configurations to identify misconfigurations, overly permissive rules, and policy violations
- **Troubleshoot** firewall-related connectivity issues using systematic diagnostic methodology
- **Differentiate** between firewall administration (this file) and firewall evasion (see `[[09_Firewall_IDS_Evasion]]`)

---

## 2. What Is a Firewall — The Absolute Fundamentals

### 2.1 The Real-World Analogy

Imagine a military checkpoint at the entrance to a secure base. Every vehicle approaching must stop. A guard checks:

- **Who are you?** (Source IP address)
- **Where are you going?** (Destination IP address)
- **What's your purpose?** (Port number / service)
- **What are you carrying?** (Packet payload — for advanced firewalls)
- **Do you have authorization?** (Does a rule permit this traffic?)

If the vehicle matches an approved entry on the guard's clipboard, it passes through. If not, it gets turned away — or in some cases, silently ignored as if the checkpoint doesn't even exist.

**A firewall is that guard. The clipboard is the rule set. The checkpoint is the network boundary.**

### 2.2 The Technical Definition

> A **firewall** is a network security device — hardware, software, or both — that **monitors incoming and outgoing network traffic** and **decides whether to allow or block** specific traffic based on a defined set of security rules.

Key points:

- **Monitors**: The firewall inspects every packet that passes through it
- **Traffic**: Both inbound (entering the network) AND outbound (leaving the network)
- **Block/Allow**: The two fundamental actions — ACCEPT or DROP (some firewalls also REJECT)
- **Predefined Rules**: The firewall doesn't "think" — it follows instructions humans wrote

### 2.3 Where Firewalls Sit in the Network

```
                    ┌─────────────────────────────────────────────┐
                    │              THE INTERNET                    │
                    │         (Untrusted Territory)                │
                    └─────────────────┬───────────────────────────┘
                                      │
                                      │  All traffic enters here
                                      ▼
                              ┌───────────────┐
                              │    GATEWAY     │
                              │   (Router)     │
                              │  Default GW    │
                              └───────┬───────┘
                                      │
                                      │  Traffic hits the firewall
                                      │  IMMEDIATELY after the gateway
                                      ▼
                         ┌────────────────────────┐
                         │       FIREWALL          │
                         │                        │
                         │  Rule 1: Allow HTTP    │
                         │  Rule 2: Allow HTTPS   │
                         │  Rule 3: Allow SSH     │
                         │  Rule 4: DROP all      │
                         │                        │
                         └───────────┬────────────┘
                                     │
                                     │  Only approved traffic
                                     │  reaches the internal network
                                     ▼
                    ┌─────────────────────────────────────────────┐
                    │           INTERNAL NETWORK                   │
                    │         (Protected Territory)                │
                    │                                              │
                    │  ┌────────┐  ┌────────┐  ┌────────┐        │
                    │  │ Server │  │Workstation│ │ Printer│        │
                    │  └────────┘  └────────┘  └────────┘        │
                    └─────────────────────────────────────────────┘
```

**Critical Concept**: Network firewalls are usually deployed **immediately after the gateway** (the router connecting your network to the internet). This is the chokepoint — ALL traffic must pass through this single point. Think of it as the castle gate: you don't put your guards in the middle of the courtyard, you put them at the gate.

### 2.4 The Three Fundamental Actions

Every firewall can take one of three actions on any packet:

| Action     | What Happens                                     | Analogy                                      |
|------------|--------------------------------------------------|----------------------------------------------|
| **ACCEPT** | Packet is allowed through                        | Guard waves the vehicle through               |
| **DROP**   | Packet is silently discarded — no response sent  | Guard ignores the vehicle completely          |
| **REJECT** | Packet is discarded AND an error is sent back    | Guard tells the vehicle "access denied"       |

**OSCP Note**: From a penetration testing perspective, DROP is more secure than REJECT. Why? Because REJECT tells the attacker "there's something here, but you can't access it." DROP gives them nothing — the attacker doesn't even know if the host exists. However, DROP can slow down scans because tools like Nmap wait for timeouts.

### 2.5 Stateful vs. Stateless Firewalls

This is one of the most important distinctions in firewall technology:

**Stateless Firewall** (First Generation):
- Examines each packet **independently**, with no memory of previous packets
- Doesn't understand "connections" — only individual packets
- Must have explicit rules for BOTH directions of traffic
- Fast but dumb

**Stateful Firewall** (Modern Standard):
- Tracks the **state of network connections** (TCP handshake, UDP "connections")
- Maintains a **state table** — a record of all active connections
- If a connection was legitimately established, return traffic is automatically allowed
- Slower but intelligent

```
STATELESS FIREWALL — Must define rules for BOTH directions:

  Rule 1: Allow TCP to port 80    (outbound web request)
  Rule 2: Allow TCP from port 80  (inbound web response)  ← Must add manually!
  Rule 3: Allow TCP to port 443
  Rule 4: Allow TCP from port 443  ← Must add manually!

STATEFUL FIREWALL — Only define the INITIATING direction:

  Rule 1: Allow outbound TCP to port 80     (outbound web request)
  Rule 2: Allow outbound TCP to port 443
  [State Table]: Connection 192.168.1.5:49321 → 93.184.216.34:80 ESTABLISHED
                  → Return traffic automatically permitted

  ┌──────────────────────────────────────────────────────┐
  │                  STATE TABLE                          │
  ├───────────────┬──────────────┬────────┬──────────────┤
  │   Source       │ Destination  │ State  │   Timeout    │
  ├───────────────┼──────────────┼────────┼──────────────┤
  │ 192.168.1.5   │ 93.184.216.34│ ESTAB  │ 3600s        │
  │ :49321        │ :80          │        │              │
  ├───────────────┼──────────────┼────────┼──────────────┤
  │ 192.168.1.10  │ 10.0.0.50   │ SYN_   │ 120s         │
  │ :52100        │ :22          │ SENT   │              │
  └───────────────┴──────────────┴────────┴──────────────┘
```

---

## 3. How Firewalls Work Internally — Packet Processing Pipeline

### 3.1 The Decision Engine

When a packet arrives at a firewall, it goes through a processing pipeline. Understanding this pipeline is crucial for both configuring firewalls correctly and for penetration testing.

```
  Packet Arrives
       │
       ▼
  ┌─────────────┐
  │ 1. CAPTURE  │  Packet is received on the network interface
  └──────┬──────┘
         │
         ▼
  ┌─────────────────┐
  │ 2. DECODE        │  Headers are parsed:
  │    HEADERS       │  - Ethernet frame → MAC addresses
  │                  │  - IP header → Source/Dest IP, TTL, Protocol
  │                  │  - TCP/UDP header → Source/Dest Port, Flags
  └──────┬──────────┘
         │
         ▼
  ┌─────────────────┐
  │ 3. STATE TABLE   │  Is this packet part of an EXISTING connection?
  │    LOOKUP        │  YES → Fast-path: Allow (if connection is valid)
  │                  │  NO  → Continue to rule evaluation
  └──────┬──────────┘
         │ (New connection or stateless)
         ▼
  ┌─────────────────┐
  │ 4. RULE          │  Compare packet against rules TOP-TO-BOTTOM
  │    EVALUATION    │  First matching rule wins
  │                  │  
  │  Rule 1: Match? ─── YES → Execute action (ACCEPT/DROP/REJECT)
  │  Rule 2: Match? ─── YES → Execute action
  │  Rule 3: Match? ─── YES → Execute action
  │  ...                 
  │  Default Policy ─── No rule matched → Apply default action
  └──────┬──────────┘
         │
         ▼
  ┌─────────────────┐
  │ 5. ACTION        │  ACCEPT: Forward packet to destination
  │    EXECUTION     │  DROP:   Silently discard
  │                  │  REJECT: Discard + send ICMP error/TCP RST
  │                  │  LOG:    Write to log file (then continue)
  └──────┬──────────┘
         │
         ▼
  ┌─────────────────┐
  │ 6. LOGGING       │  Record the decision (if logging enabled)
  │    (Optional)    │  Source, Dest, Port, Action, Timestamp
  └─────────────────┘
```

### 3.2 First Match Wins — Rule Order Matters

This is the single most important concept in firewall administration:

> **Rules are evaluated from top to bottom. The FIRST rule that matches the packet determines the action. All subsequent rules are IGNORED for that packet.**

This means rule ORDER is critical:

```bash
# WRONG ORDER — SSH rule is useless because Rule 1 drops everything first!
Rule 1: DROP   all traffic from any to any          ← Matches EVERYTHING
Rule 2: ACCEPT tcp from any to any port 22          ← NEVER reached!

# CORRECT ORDER — Specific allows first, then deny-all at the bottom
Rule 1: ACCEPT tcp from any to any port 22          ← SSH traffic matches here
Rule 2: ACCEPT tcp from any to any port 80          ← HTTP traffic matches here
Rule 3: DROP   all traffic from any to any          ← Everything else is dropped
```

**OSCP Tip**: When auditing a firewall, always check the rule order. A common misconfiguration is placing a broad ALLOW rule above more specific DENY rules, effectively bypassing security restrictions.

### 3.3 Default Policy — The Safety Net

Every firewall chain has a **default policy** — the action taken when NO rule matches a packet. There are two philosophies:

**Default ALLOW (Blacklist approach)**:
- Everything is permitted unless explicitly denied
- Easy to manage but inherently insecure
- One missed rule = one vulnerability
- Common on consumer routers and poorly configured networks

**Default DENY (Whitelist approach)**:
- Everything is blocked unless explicitly allowed
- Harder to manage but fundamentally secure
- One missed rule = one service unavailable (but secure)
- Required by every security standard (PCI-DSS, HIPAA, SOC2)

```
DEFAULT ALLOW (Insecure):                DEFAULT DENY (Secure):
┌──────────────────────┐                ┌──────────────────────┐
│ Rule 1: BLOCK port X │                │ Rule 1: ALLOW port 22│
│ Rule 2: BLOCK port Y │                │ Rule 2: ALLOW port 80│
│ Rule 3: BLOCK port Z │                │ Rule 3: ALLOW port 443│
│ Default: ALLOW ALL   │ ← Dangerous   │ Default: DROP ALL    │ ← Secure
└──────────────────────┘                └──────────────────────┘
 "Block bad, allow rest"                 "Allow good, block rest"
```

---

## 4. Firewall Types — Complete Classification

### 4.1 Layer 3 Firewalls — Packet Filtering

The simplest and fastest type. Layer 3 firewalls make decisions based on information in the **IP header and transport header** only:

- **Source IP address** — Where the packet came from
- **Destination IP address** — Where the packet is going
- **Source Port** — Originating port number
- **Destination Port** — Target service port
- **Protocol** — TCP, UDP, ICMP, etc.

```
Layer 3 Firewall Decision Matrix:
┌──────────────────────────────────────────────────────────┐
│  What It CAN See:          What It CANNOT See:           │
│                                                          │
│  ✓ Source IP: 10.0.0.5     ✗ HTTP request content        │
│  ✓ Dest IP: 192.168.1.1   ✗ SQL injection in payload    │
│  ✓ Source Port: 49321      ✗ Malware in file download    │
│  ✓ Dest Port: 80           ✗ User identity               │
│  ✓ Protocol: TCP           ✗ Application behavior        │
│  ✓ TCP Flags: SYN          ✗ Encrypted content (HTTPS)   │
└──────────────────────────────────────────────────────────┘
```

**Use Case**: High-speed perimeter filtering, ACLs on routers, basic traffic control.

**Limitation**: Cannot detect application-layer attacks. A SQL injection to port 80 looks exactly like legitimate web traffic to a Layer 3 firewall — both are "TCP to port 80."

### 4.2 Layer 7 Firewalls — Application-Level Inspection

Layer 7 firewalls (also called **Application Layer Gateways** or **Proxy Firewalls**) inspect the actual **content (payload)** of packets. They understand application protocols like HTTP, FTP, DNS, and SMTP.

The most common example is a **Web Application Firewall (WAF)**:

```
Layer 7 / WAF Inspection:

  Normal Request:
  GET /index.html HTTP/1.1
  Host: example.com
  → WAF: ✅ Looks normal. ALLOW.

  SQL Injection Attempt:
  GET /search?q=1' OR '1'='1' -- HTTP/1.1
  Host: example.com
  → WAF: 🚫 SQL injection pattern detected! BLOCK.

  XSS Attempt:
  POST /comment HTTP/1.1
  Body: message=<script>document.cookie</script>
  → WAF: 🚫 Cross-site scripting detected! BLOCK.

  Command Injection:
  GET /api/ping?host=8.8.8.8;cat /etc/passwd HTTP/1.1
  → WAF: 🚫 OS command injection detected! BLOCK.
```

**How WAFs Detect Attacks**:

| Detection Method        | How It Works                                            |
|------------------------|---------------------------------------------------------|
| **Signature-Based**     | Matches known attack patterns (regex rules)             |
| **Anomaly-Based**       | Detects deviations from "normal" traffic baseline       |
| **Behavioral**          | Tracks request rates, session patterns, bot behavior    |
| **Reputation-Based**    | Blocks known malicious IPs, Tor exit nodes, etc.        |

**Common WAFs in the Wild**:
- **ModSecurity** — Open-source, works with Apache/Nginx (OWASP CRS ruleset)
- **AWS WAF** — Cloud-native, integrates with CloudFront/ALB
- **Cloudflare WAF** — CDN-integrated, massive threat intelligence
- **Imperva/Incapsula** — Enterprise-grade, DDoS + WAF
- **F5 BIG-IP ASM** — Hardware appliance WAF

### 4.3 Next-Generation Firewalls (NGFW)

Next-Generation Firewalls combine **everything** — packet filtering, stateful inspection, deep packet inspection, AND additional security services into a single device:

```
┌──────────────────────────────────────────────────────┐
│              NEXT-GENERATION FIREWALL                 │
│                                                      │
│  ┌────────────────────────────────────────────────┐  │
│  │ Layer 3/4: Packet Filtering + Stateful         │  │
│  │ - Source/Dest IP, Ports, Protocols              │  │
│  │ - Connection state tracking                     │  │
│  └────────────────────────────────────────────────┘  │
│                      +                               │
│  ┌────────────────────────────────────────────────┐  │
│  │ Layer 7: Deep Packet Inspection (DPI)          │  │
│  │ - Application identification                    │  │
│  │ - Content inspection                            │  │
│  │ - SSL/TLS decryption and inspection             │  │
│  └────────────────────────────────────────────────┘  │
│                      +                               │
│  ┌────────────────────────────────────────────────┐  │
│  │ Additional Services:                            │  │
│  │ - Intrusion Prevention System (IPS)             │  │
│  │ - Antivirus / Anti-malware                      │  │
│  │ - URL filtering / Web categorization            │  │
│  │ - User identity awareness (LDAP/AD integration) │  │
│  │ - Application control (block Tor, BitTorrent)   │  │
│  │ - Sandboxing (detonate suspicious files)        │  │
│  │ - Threat intelligence feeds                      │  │
│  └────────────────────────────────────────────────┘  │
│                                                      │
│  Vendors: Palo Alto, Fortinet, Cisco Firepower,      │
│           Check Point, Sophos XG, SonicWall           │
└──────────────────────────────────────────────────────┘
```

**Key NGFW Capability — Application Awareness**:

A traditional firewall sees "TCP port 443" and allows HTTPS. But what if someone is tunneling BitTorrent over port 443? Or using a VPN client over port 443? A traditional firewall can't tell the difference.

An NGFW identifies the **actual application**, regardless of port:

```
Traditional Firewall:                  NGFW:
Port 443 → Allow HTTPS                Port 443 + Application ID:
Port 443 → Allow HTTPS                  → HTTPS (Google) ✅ Allow
Port 443 → Allow HTTPS                  → Tor Browser   🚫 Block
                                         → BitTorrent    🚫 Block
Can't tell them apart!                   → VPN Tunnel    🚫 Block
                                       Identifies the ACTUAL app!
```

### 4.4 AI-Based Firewalls — The Cutting Edge

AI-based firewalls use **machine learning** and **behavioral analysis** to make firewall decisions that go beyond static rules:

**How AI Firewalls Work**:

1. **Training Phase**: The AI model learns what "normal" traffic looks like for your network
2. **Baseline Creation**: Builds a behavioral profile — normal users, normal services, normal volumes
3. **Anomaly Detection**: Identifies traffic that deviates from the learned baseline
4. **Adaptive Response**: Automatically adjusts rules based on detected threats
5. **Continuous Learning**: The model improves over time as it sees more traffic

```
Traditional Rule:                      AI-Based Decision:
"Block if source IP is                 "This IP has never connected to
 in blacklist"                          the database server before, and
                                        it's 3 AM, and the request volume
Static. Binary. Dumb.                   is 500x normal. This is anomalous.
                                        Block and alert."

                                       Dynamic. Contextual. Intelligent.
```

**AI Firewall Vendors**:
- **Darktrace** — Self-learning AI, "Enterprise Immune System"
- **Palo Alto Cortex** — ML-powered threat detection
- **Fortinet FortiAI** — Deep neural network-based detection
- **Vectra AI** — Network detection and response (NDR)

**Limitations of AI Firewalls**:
- High false-positive rates during training period
- Can be poisoned by adversarial ML techniques (slowly introducing malicious traffic as "normal")
- Expensive — requires significant compute resources
- Not a replacement for good rule-based policies — they complement each other

### 4.5 Comparison Table — All Firewall Types

```
┌──────────────┬───────────┬────────────┬───────────┬────────────┐
│ Feature      │ Layer 3   │ Layer 7    │ NGFW      │ AI-Based   │
│              │ (Packet)  │ (App/WAF)  │           │            │
├──────────────┼───────────┼────────────┼───────────┼────────────┤
│ IP/Port      │ ✅ Yes    │ ✅ Yes     │ ✅ Yes    │ ✅ Yes     │
│ filtering    │           │            │           │            │
├──────────────┼───────────┼────────────┼───────────┼────────────┤
│ Stateful     │ Optional  │ ✅ Yes     │ ✅ Yes    │ ✅ Yes     │
│ inspection   │           │            │           │            │
├──────────────┼───────────┼────────────┼───────────┼────────────┤
│ Content      │ ❌ No     │ ✅ Yes     │ ✅ Yes    │ ✅ Yes     │
│ inspection   │           │            │           │            │
├──────────────┼───────────┼────────────┼───────────┼────────────┤
│ App          │ ❌ No     │ Partial    │ ✅ Yes    │ ✅ Yes     │
│ awareness    │           │            │           │            │
├──────────────┼───────────┼────────────┼───────────┼────────────┤
│ IPS/IDS      │ ❌ No     │ ❌ No      │ ✅ Yes    │ ✅ Yes     │
│ integrated   │           │            │           │            │
├──────────────┼───────────┼────────────┼───────────┼────────────┤
│ ML/AI        │ ❌ No     │ ❌ No      │ Limited   │ ✅ Core    │
│ detection    │           │            │           │            │
├──────────────┼───────────┼────────────┼───────────┼────────────┤
│ Speed        │ Fastest   │ Slow       │ Medium    │ Variable   │
├──────────────┼───────────┼────────────┼───────────┼────────────┤
│ Cost         │ Lowest    │ Medium     │ High      │ Highest    │
├──────────────┼───────────┼────────────┼───────────┼────────────┤
│ Example      │ ACL on    │ ModSec     │ Palo Alto │ Darktrace  │
│              │ router    │ WAF        │ PA-Series │            │
└──────────────┴───────────┴────────────┴───────────┴────────────┘
```

---

## 5. Network Firewalls and Gateway Protection

### 5.1 Gateway Firewalls — The Front Door

Network firewalls are typically implemented **right after the gateway**. The gateway (router) is the device that connects your internal network to the outside world (the internet). The firewall sits between this gateway and your internal network, inspecting every packet that attempts to enter or leave.

```
                          THE INTERNET
                               │
                               ▼
                    ┌─────────────────────┐
                    │   ISP Connection     │
                    │   (Modem/ONT)        │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │   GATEWAY (Router)   │  ← Default route
                    │   Public IP:         │     for all internal
                    │   203.0.113.1        │     devices
                    └──────────┬──────────┘
                               │
                    ═══════════╪═══════════  ← Security Boundary
                               │
                    ┌─────────────────────┐
                    │   FIREWALL           │  ← Inspects ALL traffic
                    │   (Dedicated or      │     crossing the boundary
                    │    on the router)    │
                    └──────────┬──────────┘
                               │
                    ┌──────────┴──────────┐
                    │   INTERNAL SWITCH    │
                    │                     │
                ┌───┴───┐  ┌───┴───┐  ┌──┴────┐
                │Server │  │ PC 1  │  │ PC 2  │
                │10.0.0.2│ │10.0.0.3│ │10.0.0.4│
                └───────┘  └───────┘  └───────┘
```

**In many cases, the gateway and firewall are the SAME device** — especially in small and medium businesses. A pfSense box, a Cisco ASA, or a Fortinet FortiGate often serves as both the router and the firewall. In enterprise networks, they're typically separate devices for performance and redundancy.

### 5.2 Host-Based vs. Network-Based Firewalls

| Aspect             | Network Firewall                          | Host-Based Firewall                        |
|--------------------|--------------------------------------------|---------------------------------------------|
| **Location**        | Network perimeter (gateway)               | On each individual host/server              |
| **Scope**           | Protects entire network                   | Protects only that one machine              |
| **Examples**        | pfSense, Cisco ASA, Palo Alto             | iptables, Windows Firewall, macOS PF        |
| **Visibility**      | Sees all network traffic                  | Sees only traffic to/from its host          |
| **Management**      | Centralized                               | Distributed (harder to manage at scale)     |
| **Defense-in-Depth**| First line of defense                     | Last line of defense                        |

**Best Practice**: Use BOTH. The network firewall blocks broad threats at the perimeter. Host-based firewalls provide per-machine protection — critical for lateral movement prevention during a breach.

### 5.3 Hardware vs. Software Firewalls

**Hardware Firewalls** (dedicated appliances):
- Purpose-built networking hardware with custom ASIC chips
- Can handle millions of packets per second
- Examples: Cisco ASA 5500, Palo Alto PA-Series, Fortinet FortiGate
- Expensive ($500 for SOHO to $500,000+ for data center grade)

**Software Firewalls** (run on general-purpose OS):
- Software running on Linux, Windows, or BSD
- Performance limited by underlying hardware
- Examples: iptables, pfSense (on commodity hardware), Windows Firewall
- Cost-effective, flexible, easy to deploy in VMs/containers

**Virtual Firewalls** (cloud/virtualized):
- Software firewalls designed for virtual environments
- Examples: AWS Security Groups, NSX Distributed Firewall, Palo Alto VM-Series
- Scale elastically with cloud infrastructure

---

## 6. The Linux Firewall Stack — Netfilter Architecture

### 6.1 Understanding Netfilter — The Kernel Framework

Every Linux firewall tool you've ever heard of — iptables, nftables, UFW, firewalld — is just a **front-end** to the same underlying kernel framework called **Netfilter**.

Netfilter is a set of **hooks** inside the Linux kernel's networking stack. These hooks allow kernel modules to register callback functions that process packets at specific points in the network path.

```
                    THE LINUX NETFILTER ARCHITECTURE
                    
  ┌─────────────────────────────────────────────────────────────┐
  │                      USER SPACE                              │
  │                                                              │
  │   ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  │
  │   │ iptables │  │ nftables │  │   UFW    │  │ firewalld│  │
  │   │ (legacy) │  │ (modern) │  │ (simple) │  │ (zones)  │  │
  │   └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘  │
  │        │              │              │              │        │
  │        │   All are front-ends to the same kernel    │        │
  │        └──────────────┼──────────────┼──────────────┘        │
  │                       │              │                       │
  ├───────────────────────┼──────────────┼───────────────────────┤
  │                       ▼              ▼                       │
  │                  KERNEL SPACE                                │
  │                                                              │
  │              ┌─────────────────────┐                        │
  │              │     NETFILTER       │                        │
  │              │  (Kernel Framework) │                        │
  │              │                     │                        │
  │              │  Hook: PREROUTING   │                        │
  │              │  Hook: INPUT        │                        │
  │              │  Hook: FORWARD      │                        │
  │              │  Hook: OUTPUT       │                        │
  │              │  Hook: POSTROUTING  │                        │
  │              └─────────────────────┘                        │
  │                                                              │
  └─────────────────────────────────────────────────────────────┘
```

### 6.2 The Five Netfilter Hooks

Every packet passing through a Linux system encounters these hooks in a specific order:

```
                    PACKET FLOW THROUGH NETFILTER HOOKS

  Incoming Packet                              Outgoing Packet
  from Network                                 to Network
       │                                            ▲
       ▼                                            │
  ┌──────────┐                                ┌──────────┐
  │PREROUTING│ ← NAT (DNAT) happens here      │POSTROUTING│ ← NAT (SNAT/MASQ)
  └────┬─────┘                                └─────▲────┘
       │                                            │
       ▼                                            │
  ┌──────────┐     Is packet for     ┌──────────┐  │
  │ Routing  │────── this host? ─────│ Routing  │──┘
  │ Decision │      NO → FORWARD     │ Decision │
  └────┬─────┘         │             └─────▲────┘
       │ YES           │                   │
       ▼               ▼                   │
  ┌──────────┐    ┌──────────┐        ┌──────────┐
  │  INPUT   │    │ FORWARD  │        │  OUTPUT  │
  │          │    │          │        │          │
  │ Packet   │    │ Packet   │        │ Packet   │
  │ going TO │    │ passing  │        │ FROM     │
  │ this     │    │ THROUGH  │        │ this     │
  │ machine  │    │ (routing)│        │ machine  │
  └────┬─────┘    └────┬─────┘        └─────▲────┘
       │               │                    │
       ▼               │                    │
  ┌──────────┐         │              ┌──────────┐
  │ Local    │         └──────────────│ Local    │
  │ Process  │  (e.g., SSH server,    │ Process  │
  │ (Server) │   web server, etc.)    │ (Client) │
  └──────────┘                        └──────────┘
```

| Hook           | When It Fires                                              | Common Use                    |
|----------------|------------------------------------------------------------|-------------------------------|
| **PREROUTING** | Immediately when packet arrives, before routing decision   | DNAT, port forwarding         |
| **INPUT**      | After routing, if packet is destined for THIS machine      | Filter incoming traffic       |
| **FORWARD**    | After routing, if packet is destined for ANOTHER machine   | Filter routed/forwarded traffic|
| **OUTPUT**     | When a local process generates a packet                    | Filter outgoing traffic       |
| **POSTROUTING**| Just before packet leaves the machine                      | SNAT, masquerading            |

---

## 7. iptables — The Classic Workhorse

### 7.1 iptables Fundamentals

`iptables` has been the standard Linux firewall tool since 2001. Even though `nftables` is its successor, iptables is still the most widely deployed firewall on Linux systems. Every security professional must know it.

**Core Concepts**:
- **Tables**: Categories of rules (filter, nat, mangle, raw)
- **Chains**: Lists of rules within each table (INPUT, OUTPUT, FORWARD, etc.)
- **Rules**: Individual instructions (match criteria + target/action)
- **Targets**: What to do when a rule matches (ACCEPT, DROP, REJECT, LOG)

### 7.2 The Four Tables

```
┌─────────────────────────────────────────────────────────────┐
│                    iptables TABLES                            │
├───────────┬──────────────────────────────────────────────────┤
│  filter   │ DEFAULT table. Decides whether to ALLOW or       │
│           │ BLOCK packets. This is what 99% of people mean   │
│           │ when they say "firewall rules."                   │
│           │ Chains: INPUT, FORWARD, OUTPUT                    │
├───────────┼──────────────────────────────────────────────────┤
│  nat      │ Network Address Translation. Modifies source or  │
│           │ destination addresses. Used for port forwarding,  │
│           │ masquerading (sharing internet via one IP).        │
│           │ Chains: PREROUTING, OUTPUT, POSTROUTING           │
├───────────┼──────────────────────────────────────────────────┤
│  mangle   │ Specialized packet alteration. Modifies packet    │
│           │ headers (TTL, TOS, MARK). Used for QoS, policy   │
│           │ routing. Rarely used in basic firewalling.         │
│           │ Chains: All five hooks                             │
├───────────┼──────────────────────────────────────────────────┤
│  raw      │ Marks packets to bypass connection tracking       │
│           │ (conntrack). Used for performance optimization.    │
│           │ Chains: PREROUTING, OUTPUT                         │
└───────────┴──────────────────────────────────────────────────┘
```

### 7.3 iptables Command Syntax

```bash
# General syntax:
iptables -t <table> <command> <chain> <match-criteria> -j <target>

# If -t is omitted, the "filter" table is used by default
```

**Commands**:

| Flag | Meaning               | Example                                        |
|------|-----------------------|------------------------------------------------|
| `-A` | Append rule to chain  | `iptables -A INPUT -p tcp --dport 22 -j ACCEPT`|
| `-I` | Insert rule at top    | `iptables -I INPUT 1 -p tcp --dport 22 -j ACCEPT`|
| `-D` | Delete a rule         | `iptables -D INPUT -p tcp --dport 22 -j ACCEPT`|
| `-L` | List rules            | `iptables -L INPUT -n -v`                      |
| `-F` | Flush (delete all)    | `iptables -F INPUT`                            |
| `-P` | Set default policy    | `iptables -P INPUT DROP`                       |
| `-N` | Create new chain      | `iptables -N MY_CHAIN`                         |
| `-X` | Delete custom chain   | `iptables -X MY_CHAIN`                         |

### 7.4 Building a Complete iptables Ruleset — From Scratch

Here is a production-ready iptables firewall script for a Linux web server:

```bash
#!/bin/bash
# ===========================================================
# iptables Firewall Script — Production Web Server
# ===========================================================
# This script configures a complete firewall for a Linux
# server running SSH (22), HTTP (80), and HTTPS (443).
# Default policy: DROP everything, then whitelist services.
# ===========================================================

# ---- STEP 1: Flush all existing rules (clean slate) ----
# Remove all rules from all chains in all tables
iptables -F           # Flush all rules in filter table
iptables -t nat -F    # Flush all rules in nat table
iptables -t mangle -F # Flush all rules in mangle table
iptables -X           # Delete all custom chains

# ---- STEP 2: Set default policies (DENY ALL) ----
# If no rule matches, DROP the packet
iptables -P INPUT DROP      # Block all incoming traffic by default
iptables -P FORWARD DROP    # Block all forwarded traffic by default
iptables -P OUTPUT ACCEPT   # Allow all outgoing traffic
# Note: OUTPUT ACCEPT is common for servers. For paranoid setups,
# set it to DROP and whitelist outbound connections too.

# ---- STEP 3: Allow loopback interface (localhost) ----
# The loopback interface (127.0.0.1) is used for internal
# communication between processes on the same machine.
# Blocking it breaks many applications.
iptables -A INPUT -i lo -j ACCEPT
iptables -A OUTPUT -o lo -j ACCEPT

# ---- STEP 4: Allow established and related connections ----
# This is the STATEFUL part. If WE initiated a connection
# (e.g., apt update, DNS lookup), allow the response back in.
# Without this rule, the server can't receive responses to its
# own outbound requests.
iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# ---- STEP 5: Allow SSH (port 22) ----
# Limit SSH to a specific management subnet for security
# -p tcp        = TCP protocol
# --dport 22    = Destination port 22 (SSH)
# -s 10.0.0.0/24 = Only from the management network
iptables -A INPUT -p tcp --dport 22 -s 10.0.0.0/24 -j ACCEPT

# ---- STEP 6: Allow HTTP (port 80) from anywhere ----
iptables -A INPUT -p tcp --dport 80 -j ACCEPT

# ---- STEP 7: Allow HTTPS (port 443) from anywhere ----
iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# ---- STEP 8: Allow ICMP (ping) — rate limited ----
# Allow ping but limit to 1 per second to prevent ping floods
# -m limit --limit 1/s = Match at most 1 packet per second
# --limit-burst 4      = Allow initial burst of 4 packets
iptables -A INPUT -p icmp --icmp-type echo-request \
  -m limit --limit 1/s --limit-burst 4 -j ACCEPT

# ---- STEP 9: Drop invalid packets ----
# Packets that don't match any known connection or are
# malformed should be dropped immediately
iptables -A INPUT -m conntrack --ctstate INVALID -j DROP

# ---- STEP 10: Log dropped packets (for debugging) ----
# Log any packet that reaches this point (about to be dropped)
# --log-prefix adds a label to syslog entries for easy filtering
# --log-level 4 = warning level
iptables -A INPUT -j LOG --log-prefix "IPTABLES-DROPPED: " --log-level 4

# ---- STEP 11: Explicitly drop everything else ----
# This is redundant with the default policy but makes the
# rule set self-documenting
iptables -A INPUT -j DROP

# ---- VERIFICATION ----
echo "Firewall rules applied successfully!"
echo ""
iptables -L INPUT -n -v --line-numbers
```

### 7.5 iptables — Common Rule Examples

```bash
# ---- Block a specific IP address ----
# A known attacker is scanning your network from 45.33.32.156
iptables -A INPUT -s 45.33.32.156 -j DROP

# ---- Block an entire subnet ----
# Block all traffic from the 10.10.10.0/24 network
iptables -A INPUT -s 10.10.10.0/24 -j DROP

# ---- Allow DNS (TCP and UDP port 53) ----
iptables -A INPUT -p udp --dport 53 -j ACCEPT
iptables -A INPUT -p tcp --dport 53 -j ACCEPT

# ---- Allow MySQL only from web server ----
# Only the web server at 10.0.0.5 should access MySQL
iptables -A INPUT -p tcp --dport 3306 -s 10.0.0.5 -j ACCEPT

# ---- Rate limit SSH connections (brute-force protection) ----
# Allow max 3 new SSH connections per minute per source IP
iptables -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW \
  -m recent --set --name SSH
iptables -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW \
  -m recent --update --seconds 60 --hitcount 4 --name SSH -j DROP

# ---- Allow specific port range ----
# Allow ports 8000-8100 (common for development servers)
iptables -A INPUT -p tcp --dport 8000:8100 -j ACCEPT

# ---- Allow multiple ports with multiport ----
iptables -A INPUT -p tcp -m multiport --dports 80,443,8080,8443 -j ACCEPT

# ---- Block outbound connections to a specific IP ----
# Prevent the server from connecting to a known C2 server
iptables -A OUTPUT -d 198.51.100.66 -j DROP

# ---- Log and drop all traffic from a country (using ipset) ----
# First create an ipset with the country's IP ranges
# ipset create CHINA hash:net
# ipset add CHINA 1.0.0.0/24
# ipset add CHINA 1.0.4.0/22
# ... (add all ranges)
iptables -A INPUT -m set --match-set CHINA src -j LOG \
  --log-prefix "CHINA-BLOCKED: "
iptables -A INPUT -m set --match-set CHINA src -j DROP
```

### 7.6 Saving and Restoring iptables Rules

iptables rules are stored in memory. They are **lost on reboot** unless you save them:

```bash
# ---- Save current rules to a file ----
iptables-save > /etc/iptables/rules.v4       # Debian/Ubuntu
iptables-save > /etc/sysconfig/iptables       # RHEL/CentOS

# ---- Restore rules from a file ----
iptables-restore < /etc/iptables/rules.v4     # Debian/Ubuntu
iptables-restore < /etc/sysconfig/iptables     # RHEL/CentOS

# ---- On Debian/Ubuntu, install persistence package ----
sudo apt install iptables-persistent
# Rules are automatically loaded at boot from:
#   /etc/iptables/rules.v4  (IPv4)
#   /etc/iptables/rules.v6  (IPv6)

# ---- On RHEL/CentOS, use the iptables service ----
sudo systemctl enable iptables
sudo service iptables save
```

### 7.7 Viewing and Debugging iptables

```bash
# ---- List all rules with line numbers, verbose, numeric ----
sudo iptables -L -n -v --line-numbers
# -L = List rules
# -n = Numeric output (don't resolve DNS — much faster)
# -v = Verbose (show packet/byte counters)
# --line-numbers = Show rule position numbers

# Example output:
# Chain INPUT (policy DROP 1523 packets, 91380 bytes)
# num   pkts bytes target   prot opt in  out  source     destination
# 1     8234  492K ACCEPT   all  --  lo  *    0.0.0.0/0  0.0.0.0/0
# 2    45231   27M ACCEPT   all  --  *   *    0.0.0.0/0  0.0.0.0/0  ctstate RELATED,ESTABLISHED
# 3      892  53K ACCEPT   tcp  --  *   *    10.0.0.0/24 0.0.0.0/0  tcp dpt:22
# 4    12045  722K ACCEPT   tcp  --  *   *    0.0.0.0/0  0.0.0.0/0  tcp dpt:80
# 5     8901  534K ACCEPT   tcp  --  *   *    0.0.0.0/0  0.0.0.0/0  tcp dpt:443

# ---- List rules for a specific table ----
sudo iptables -t nat -L -n -v

# ---- Show rules in iptables-save format (more readable) ----
sudo iptables-save

# ---- Watch rule counters in real-time ----
watch -n 1 'iptables -L INPUT -n -v'

# ---- Check if a specific rule exists ----
sudo iptables -C INPUT -p tcp --dport 22 -j ACCEPT
# Returns 0 if rule exists, 1 if not
```

---

## 8. nftables — The Modern Successor

### 8.1 Why nftables Replaces iptables

`nftables` is the **official successor** to iptables, included in the Linux kernel since version 3.13 (2014). Most modern distributions (Debian 10+, Ubuntu 20.04+, RHEL 8+, Fedora 18+) ship with nftables as the default.

**Why the migration?**:

| Problem with iptables               | nftables Solution                         |
|--------------------------------------|-------------------------------------------|
| Separate tools for IPv4/IPv6        | Single tool: `nft` handles both           |
| Code duplication in kernel          | Single kernel framework                    |
| Performance issues with large rulesets| Optimized rule evaluation (maps, sets)    |
| Complex syntax                      | Cleaner, more consistent syntax            |
| No atomic rule updates              | Atomic rule replacement (no packet loss)   |
| Limited data structures             | Built-in sets, maps, concatenations        |

### 8.2 nftables Syntax and Structure

```bash
# ---- List all rules ----
sudo nft list ruleset

# ---- Create a table (container for chains) ----
# "inet" family handles both IPv4 and IPv6
sudo nft add table inet my_firewall

# ---- Create a chain with a base hook ----
# type filter = filtering packets
# hook input  = process packets destined for this machine
# priority 0  = standard priority
# policy drop = default action if no rule matches
sudo nft add chain inet my_firewall input \
  '{ type filter hook input priority 0; policy drop; }'

# ---- Add rules to the chain ----
# Allow loopback
sudo nft add rule inet my_firewall input iif lo accept

# Allow established/related connections
sudo nft add rule inet my_firewall input ct state established,related accept

# Allow SSH from management network
sudo nft add rule inet my_firewall input ip saddr 10.0.0.0/24 tcp dport 22 accept

# Allow HTTP and HTTPS
sudo nft add rule inet my_firewall input tcp dport { 80, 443 } accept

# Allow ICMP (ping) — rate limited
sudo nft add rule inet my_firewall input icmp type echo-request \
  limit rate 1/second accept

# Log and drop everything else
sudo nft add rule inet my_firewall input log prefix \"DROPPED: \" drop
```

### 8.3 Complete nftables Ruleset File

```bash
#!/usr/sbin/nft -f
# ===========================================================
# nftables Firewall Configuration — Production Web Server
# ===========================================================
# Save as: /etc/nftables.conf
# Apply with: sudo nft -f /etc/nftables.conf
# ===========================================================

# Flush all existing rules
flush ruleset

# Define variables for readability
define MGMT_NET = 10.0.0.0/24
define WEB_PORTS = { 80, 443 }
define SSH_PORT  = 22

# ---- Main firewall table (IPv4 + IPv6) ----
table inet firewall {

    # ---- Set: Blocked IPs (can be updated dynamically) ----
    set blocklist {
        type ipv4_addr
        flags timeout
        elements = {
            45.33.32.156 timeout 24h,
            198.51.100.66 timeout 24h
        }
    }

    # ---- Set: Allowed management IPs ----
    set mgmt_hosts {
        type ipv4_addr
        elements = {
            10.0.0.5,
            10.0.0.10,
            10.0.0.254
        }
    }

    # ---- INPUT chain (traffic TO this machine) ----
    chain input {
        type filter hook input priority 0; policy drop;

        # Allow loopback interface
        iif lo accept

        # Drop blocked IPs immediately
        ip saddr @blocklist drop

        # Allow established and related connections
        ct state established,related accept

        # Drop invalid packets
        ct state invalid drop

        # Allow SSH from management hosts only
        ip saddr @mgmt_hosts tcp dport $SSH_PORT accept

        # Allow web traffic from anywhere
        tcp dport $WEB_PORTS accept

        # Allow ICMP (ping) with rate limiting
        icmp type echo-request limit rate 1/second burst 4 packets accept

        # Allow ICMPv6 (required for IPv6 to function)
        icmpv6 type { nd-neighbor-solicit, nd-neighbor-advert,
                      nd-router-solicit, nd-router-advert } accept

        # Log dropped packets
        log prefix "NFT-DROPPED: " flags all counter drop
    }

    # ---- OUTPUT chain (traffic FROM this machine) ----
    chain output {
        type filter hook output priority 0; policy accept;

        # Allow all outgoing traffic (common for servers)
        # For paranoid mode, set policy drop and whitelist
    }

    # ---- FORWARD chain (traffic THROUGH this machine) ----
    chain forward {
        type filter hook forward priority 0; policy drop;
        # Not a router, drop all forwarded traffic
    }
}
```

### 8.4 nftables Dynamic Sets — Real-Time Blocklists

One of nftables' killer features is dynamic sets with timeouts:

```bash
# ---- Create a set that auto-expires entries ----
sudo nft add set inet firewall dynamic_blocklist \
  '{ type ipv4_addr; flags dynamic,timeout; timeout 1h; }'

# ---- Auto-add IPs that connect too fast to SSH ----
# If more than 3 connections in 60 seconds, add to blocklist
sudo nft add rule inet firewall input \
  tcp dport 22 ct state new \
  meter ssh_meter '{ ip saddr limit rate over 3/minute }' \
  add @dynamic_blocklist '{ ip saddr timeout 1h }' drop

# ---- View the dynamic blocklist ----
sudo nft list set inet firewall dynamic_blocklist
```

---

## 9. UFW — Uncomplicated Firewall Deep Dive

### 9.1 What Is UFW and Why Does It Exist?

**UFW (Uncomplicated Firewall)** is a user-friendly front-end for iptables/nftables, designed to make firewall management accessible to people who don't want to memorize iptables syntax. It was created by Canonical for Ubuntu and is now the default firewall management tool on Ubuntu and many Debian-based systems.

**Key Principle**: UFW doesn't replace iptables — it generates iptables rules for you behind the scenes. When you type a simple UFW command, it translates it into the complex iptables syntax automatically.

```
         What YOU type:                     What UFW GENERATES:

  sudo ufw allow 22/tcp          →   iptables -A ufw-user-input 
                                       -p tcp --dport 22 
                                       -j ACCEPT

  sudo ufw deny from             →   iptables -A ufw-user-input
    10.0.0.50                          -s 10.0.0.50 
                                       -j DROP

  One simple command              →   Complex iptables rule
```

### 9.2 UFW Installation and Initial Setup

```bash
# ---- Install UFW (usually pre-installed on Ubuntu) ----
sudo apt update && sudo apt install ufw    # Debian/Ubuntu
sudo yum install ufw                        # RHEL/CentOS (EPEL)
sudo pacman -S ufw                          # Arch Linux

# ---- Check UFW status ----
sudo ufw status
# Output: Status: inactive

# ---- CRITICAL: Before enabling UFW, allow SSH first! ----
# If you enable UFW without allowing SSH, you will LOCK
# YOURSELF OUT of a remote server. This is the #1 UFW mistake.
sudo ufw allow ssh
# OR equivalently:
sudo ufw allow 22/tcp

# ---- Set default policies ----
# Deny all incoming, allow all outgoing (standard secure config)
sudo ufw default deny incoming
sudo ufw default allow outgoing

# ---- Enable UFW ----
sudo ufw enable
# Output: Firewall is active and enabled on system startup

# ---- Verify status ----
sudo ufw status verbose
# Output:
# Status: active
# Logging: on (low)
# Default: deny (incoming), allow (outgoing), disabled (routed)
# New profiles: skip
#
# To                         Action      From
# --                         ------      ----
# 22/tcp                     ALLOW IN    Anywhere
# 22/tcp (v6)                ALLOW IN    Anywhere (v6)
```

### 9.3 UFW Rule Management — Complete Command Reference

```bash
# ============================================================
# BASIC ALLOW/DENY RULES
# ============================================================

# ---- Allow a specific port ----
sudo ufw allow 80/tcp          # Allow HTTP
sudo ufw allow 443/tcp         # Allow HTTPS
sudo ufw allow 53              # Allow DNS (both TCP and UDP)

# ---- Deny a specific port ----
sudo ufw deny 23/tcp           # Block Telnet
sudo ufw deny 3389/tcp         # Block RDP

# ---- Allow by service name ----
# UFW reads service names from /etc/services
sudo ufw allow ssh              # Same as allow 22/tcp
sudo ufw allow http             # Same as allow 80/tcp
sudo ufw allow https            # Same as allow 443/tcp

# ---- Allow a port range ----
sudo ufw allow 8000:8100/tcp    # Allow ports 8000-8100

# ============================================================
# SOURCE-BASED RULES (WHO can connect)
# ============================================================

# ---- Allow from a specific IP ----
sudo ufw allow from 10.0.0.5           # Allow ALL traffic from this IP

# ---- Allow from a specific IP to a specific port ----
sudo ufw allow from 10.0.0.5 to any port 22    # SSH from 10.0.0.5 only

# ---- Allow from a subnet ----
sudo ufw allow from 192.168.1.0/24              # Allow entire subnet
sudo ufw allow from 192.168.1.0/24 to any port 80  # Web from subnet only

# ---- Block a specific IP ----
sudo ufw deny from 45.33.32.156        # Block attacker IP

# ---- Block an entire subnet ----
sudo ufw deny from 10.10.10.0/24       # Block entire network

# ============================================================
# INTERFACE-SPECIFIC RULES
# ============================================================

# ---- Allow on a specific network interface ----
sudo ufw allow in on eth0 to any port 80     # HTTP only on eth0
sudo ufw allow in on eth1 from 10.0.0.0/24   # Management only on eth1

# ============================================================
# RULE MANAGEMENT
# ============================================================

# ---- List rules with numbers ----
sudo ufw status numbered
# Output:
# Status: active
#      To                         Action      From
#      --                         ------      ----
# [ 1] 22/tcp                     ALLOW IN    Anywhere
# [ 2] 80/tcp                     ALLOW IN    Anywhere
# [ 3] 443/tcp                    ALLOW IN    Anywhere

# ---- Delete a rule by number ----
sudo ufw delete 3                # Delete rule #3

# ---- Delete a rule by specification ----
sudo ufw delete allow 80/tcp    # Delete the "allow 80/tcp" rule

# ---- Insert a rule at a specific position ----
# Insert a deny rule BEFORE the allow rules
sudo ufw insert 1 deny from 45.33.32.156

# ---- Reset UFW (remove ALL rules) ----
sudo ufw reset
# WARNING: This deletes everything and disables UFW

# ============================================================
# LOGGING
# ============================================================

# ---- Enable/disable logging ----
sudo ufw logging on             # Enable logging
sudo ufw logging off            # Disable logging
sudo ufw logging low            # Low verbosity
sudo ufw logging medium         # Medium verbosity
sudo ufw logging high           # High verbosity (all packets)
sudo ufw logging full           # Full verbosity

# ---- View firewall logs ----
sudo tail -f /var/log/ufw.log
# Or use journalctl on systemd systems:
sudo journalctl -f | grep UFW
```

### 9.4 UFW Application Profiles

UFW supports application profiles — predefined rule sets for common applications:

```bash
# ---- List available application profiles ----
sudo ufw app list
# Output:
# Available applications:
#   Apache
#   Apache Full
#   Apache Secure
#   Nginx Full
#   Nginx HTTP
#   Nginx HTTPS
#   OpenSSH

# ---- View profile details ----
sudo ufw app info "Nginx Full"
# Output:
# Profile: Nginx Full
# Title: Web Server (Nginx, HTTP + HTTPS)
# Description: Small, but very powerful and efficient web server
# Ports:
#   80,443/tcp

# ---- Allow an application profile ----
sudo ufw allow "Nginx Full"    # Allows both 80 and 443
sudo ufw allow "OpenSSH"       # Allows port 22

# ---- Create a custom application profile ----
# Profiles are stored in /etc/ufw/applications.d/
# Create file: /etc/ufw/applications.d/myapp
cat > /etc/ufw/applications.d/myapp << 'EOF'
[MyWebApp]
title=My Custom Web Application
description=Custom app running on ports 8080 and 8443
ports=8080,8443/tcp
EOF

# Reload and use:
sudo ufw app update MyWebApp
sudo ufw allow MyWebApp
```

### 9.5 UFW Configuration Files — What's Under the Hood

```bash
# ---- Main configuration file ----
/etc/default/ufw
# Contains: default policies, IPv6 enable/disable, kernel modules

# ---- User rules (your ufw allow/deny commands) ----
/etc/ufw/user.rules        # IPv4 rules
/etc/ufw/user6.rules       # IPv6 rules

# ---- System rules (don't edit unless you know what you're doing) ----
/etc/ufw/before.rules      # Rules applied BEFORE user rules
/etc/ufw/after.rules       # Rules applied AFTER user rules
/etc/ufw/before6.rules     # IPv6 before rules
/etc/ufw/after6.rules      # IPv6 after rules

# ---- Sysctl settings (kernel network parameters) ----
/etc/ufw/sysctl.conf       # Controls IP forwarding, ICMP, etc.

# ---- Application profiles ----
/etc/ufw/applications.d/   # Directory containing app profiles
```

### 9.6 UFW for Port Forwarding and NAT

UFW can handle NAT and port forwarding by editing the before.rules file:

```bash
# ---- Enable IP forwarding first ----
# Edit /etc/ufw/sysctl.conf:
# net/ipv4/ip_forward=1

# ---- Add NAT rules to /etc/ufw/before.rules ----
# Add BEFORE the *filter section:

*nat
:PREROUTING ACCEPT [0:0]
:POSTROUTING ACCEPT [0:0]

# Port forward: External port 8080 → Internal 10.0.0.5:80
-A PREROUTING -i eth0 -p tcp --dport 8080 -j DNAT --to-destination 10.0.0.5:80

# Masquerade outbound traffic (NAT for internet sharing)
-A POSTROUTING -s 10.0.0.0/24 -o eth0 -j MASQUERADE

COMMIT

# ---- Allow forwarded traffic in UFW ----
sudo ufw route allow in on eth0 out on eth1 to 10.0.0.5 port 80 proto tcp

# ---- Set default forward policy ----
# Edit /etc/default/ufw:
# DEFAULT_FORWARD_POLICY="ACCEPT"

# ---- Reload UFW ----
sudo ufw reload
```

---

## 10. firewalld — Zone-Based Dynamic Firewall

### 10.1 What Is firewalld?

`firewalld` is the default firewall management tool on **Red Hat-based distributions** (RHEL, CentOS, Fedora, Rocky Linux, AlmaLinux). Unlike UFW's simple allow/deny model, firewalld uses a **zone-based** architecture where network interfaces are assigned to security zones with different trust levels.

### 10.2 Zone Concept

```
         FIREWALLD ZONE ARCHITECTURE
         
  ┌──────────────────────────────────────────────────────┐
  │                    ZONES (Trust Levels)                │
  │                                                      │
  │  ┌──────────┐  ┌──────────┐  ┌──────────┐          │
  │  │  DROP     │  │ PUBLIC   │  │ TRUSTED  │          │
  │  │          │  │          │  │          │          │
  │  │ Trust: 0 │  │ Trust: 5 │  │ Trust: 10│          │
  │  │ Drop ALL │  │ Allow    │  │ Allow    │          │
  │  │ traffic  │  │ selected │  │ ALL      │          │
  │  │          │  │ services │  │ traffic  │          │
  │  └──────────┘  └──────────┘  └──────────┘          │
  │                                                      │
  │  Each NETWORK INTERFACE is assigned to a ZONE:       │
  │    eth0 (internet-facing) → public zone              │
  │    eth1 (internal LAN)    → internal zone            │
  │    eth2 (management VLAN) → trusted zone             │
  └──────────────────────────────────────────────────────┘
```

**Predefined Zones** (ordered from least trusted to most):

| Zone       | Description                                               |
|------------|-----------------------------------------------------------|
| `drop`     | Drop ALL incoming. No reply sent. Only outbound allowed.  |
| `block`    | Reject ALL incoming with icmp-host-prohibited reply.      |
| `public`   | Default zone. Untrusted networks. Only select services.   |
| `external` | For NAT/masquerading. External-facing interfaces.         |
| `dmz`      | DMZ zone. Limited access to internal services.            |
| `work`     | Work network. Mostly trusted, some services allowed.      |
| `home`     | Home network. More trusted than work.                     |
| `internal` | Internal network. High trust level.                       |
| `trusted`  | All connections accepted. FULL trust.                     |

### 10.3 firewalld Command Reference

```bash
# ============================================================
# STATUS AND INFORMATION
# ============================================================

# ---- Check if firewalld is running ----
sudo firewall-cmd --state
# Output: running

# ---- List all zones and their configurations ----
sudo firewall-cmd --list-all-zones

# ---- Show the default zone ----
sudo firewall-cmd --get-default-zone
# Output: public

# ---- Show active zones (zones with interfaces assigned) ----
sudo firewall-cmd --get-active-zones
# Output:
# public
#   interfaces: eth0

# ---- List everything in the current zone ----
sudo firewall-cmd --list-all
# Output:
# public (active)
#   target: default
#   icmp-block-inversion: no
#   interfaces: eth0
#   sources:
#   services: ssh dhcpv6-client
#   ports:
#   protocols:
#   masquerade: no
#   forward-ports:
#   source-ports:
#   icmp-blocks:
#   rich rules:

# ============================================================
# MANAGING SERVICES
# ============================================================

# ---- List available predefined services ----
sudo firewall-cmd --get-services
# Output: (long list including http, https, ssh, dns, ftp, etc.)

# ---- Add a service to the current zone ----
# --permanent = survives reboot (without it, rule is temporary)
sudo firewall-cmd --add-service=http --permanent
sudo firewall-cmd --add-service=https --permanent
sudo firewall-cmd --reload    # Apply permanent changes

# ---- Remove a service ----
sudo firewall-cmd --remove-service=dhcpv6-client --permanent
sudo firewall-cmd --reload

# ---- Add a service to a specific zone ----
sudo firewall-cmd --zone=dmz --add-service=http --permanent

# ============================================================
# MANAGING PORTS
# ============================================================

# ---- Open a specific port ----
sudo firewall-cmd --add-port=8080/tcp --permanent
sudo firewall-cmd --add-port=5000-5100/tcp --permanent   # Range

# ---- Remove a port ----
sudo firewall-cmd --remove-port=8080/tcp --permanent

# ============================================================
# MANAGING INTERFACES AND ZONES
# ============================================================

# ---- Assign an interface to a zone ----
sudo firewall-cmd --zone=internal --change-interface=eth1 --permanent

# ---- Change the default zone ----
sudo firewall-cmd --set-default-zone=internal

# ============================================================
# RICH RULES (Advanced rules — like iptables in firewalld)
# ============================================================

# ---- Allow SSH only from a specific subnet ----
sudo firewall-cmd --permanent --add-rich-rule='
  rule family="ipv4"
  source address="10.0.0.0/24"
  service name="ssh"
  accept'

# ---- Block a specific IP address ----
sudo firewall-cmd --permanent --add-rich-rule='
  rule family="ipv4"
  source address="45.33.32.156"
  drop'

# ---- Rate limit connections ----
sudo firewall-cmd --permanent --add-rich-rule='
  rule family="ipv4"
  service name="ssh"
  accept
  limit value="3/m"'

# ---- Log and drop traffic from a subnet ----
sudo firewall-cmd --permanent --add-rich-rule='
  rule family="ipv4"
  source address="192.168.100.0/24"
  log prefix="SUSPICIOUS: " level="warning"
  drop'

# ---- Port forwarding ----
# Forward external port 8080 to internal 10.0.0.5:80
sudo firewall-cmd --permanent --add-forward-port=\
  port=8080:proto=tcp:toport=80:toaddr=10.0.0.5
sudo firewall-cmd --permanent --add-masquerade    # Enable NAT
sudo firewall-cmd --reload

# ============================================================
# RUNTIME vs PERMANENT
# ============================================================

# Runtime (temporary — lost on reload/reboot):
sudo firewall-cmd --add-service=http
# This takes effect IMMEDIATELY but disappears after reload

# Permanent (survives reboot — but NOT active until reload):
sudo firewall-cmd --add-service=http --permanent
sudo firewall-cmd --reload    # Now it's active AND permanent

# Both at once (immediate + permanent):
sudo firewall-cmd --add-service=http
sudo firewall-cmd --add-service=http --permanent
```

---

## 11. Windows Firewall — Defender Firewall with Advanced Security

### 11.1 Overview

**Windows Defender Firewall with Advanced Security** (formerly Windows Firewall) is the built-in host-based firewall on all modern Windows systems (Vista/Server 2008 and later). It's a stateful firewall that supports:

- Inbound and outbound rule filtering
- Program-based rules (allow/block specific executables)
- Port-based rules
- IPsec integration (authenticated connections)
- Three network profiles (Domain, Private, Public)

### 11.2 Network Profiles

Windows Firewall uses three profiles, similar to firewalld's zones:

```
┌──────────────────────────────────────────────────────┐
│           WINDOWS FIREWALL PROFILES                   │
│                                                      │
│  ┌──────────────┐  ┌──────────────┐  ┌────────────┐ │
│  │   DOMAIN     │  │   PRIVATE    │  │   PUBLIC   │ │
│  │              │  │              │  │            │ │
│  │ Connected to │  │ Home/Office  │  │ Coffee     │ │
│  │ AD domain    │  │ network      │  │ shop, etc. │ │
│  │              │  │              │  │            │ │
│  │ Most relaxed │  │ Moderate     │  │ Most       │ │
│  │ (trusted     │  │ (somewhat    │  │ restrictive│ │
│  │  corporate   │  │  trusted)    │  │ (untrusted)│ │
│  │  network)    │  │              │  │            │ │
│  └──────────────┘  └──────────────┘  └────────────┘ │
│                                                      │
│  Windows automatically selects the profile based     │
│  on the network type detected.                       │
└──────────────────────────────────────────────────────┘
```

### 11.3 Managing Windows Firewall via PowerShell

```powershell
# ============================================================
# VIEWING FIREWALL STATUS
# ============================================================

# ---- Check firewall status for all profiles ----
Get-NetFirewallProfile | Format-Table Name, Enabled
# Output:
# Name    Enabled
# ----    -------
# Domain     True
# Private    True
# Public     True

# ---- Get detailed profile settings ----
Get-NetFirewallProfile -Name Public | Format-List *

# ---- List all firewall rules (there are hundreds!) ----
Get-NetFirewallRule | Format-Table DisplayName, Direction, Action, Enabled

# ---- List only enabled inbound rules ----
Get-NetFirewallRule -Direction Inbound -Enabled True |
  Format-Table DisplayName, Action

# ============================================================
# CREATING FIREWALL RULES
# ============================================================

# ---- Allow inbound SSH (port 22) ----
New-NetFirewallRule -DisplayName "Allow SSH" `
  -Direction Inbound `
  -Protocol TCP `
  -LocalPort 22 `
  -Action Allow `
  -Profile Any

# ---- Allow inbound HTTP/HTTPS ----
New-NetFirewallRule -DisplayName "Allow HTTP" `
  -Direction Inbound `
  -Protocol TCP `
  -LocalPort 80 `
  -Action Allow

New-NetFirewallRule -DisplayName "Allow HTTPS" `
  -Direction Inbound `
  -Protocol TCP `
  -LocalPort 443 `
  -Action Allow

# ---- Block a specific IP address ----
New-NetFirewallRule -DisplayName "Block Attacker" `
  -Direction Inbound `
  -RemoteAddress 45.33.32.156 `
  -Action Block

# ---- Allow a specific program ----
New-NetFirewallRule -DisplayName "Allow Python" `
  -Direction Inbound `
  -Program "C:\Python39\python.exe" `
  -Action Allow

# ---- Allow a port range ----
New-NetFirewallRule -DisplayName "Allow Dev Ports" `
  -Direction Inbound `
  -Protocol TCP `
  -LocalPort 8000-8100 `
  -Action Allow

# ---- Block outbound to a C2 server ----
New-NetFirewallRule -DisplayName "Block C2" `
  -Direction Outbound `
  -RemoteAddress 198.51.100.66 `
  -Action Block

# ============================================================
# MANAGING EXISTING RULES
# ============================================================

# ---- Disable a rule ----
Disable-NetFirewallRule -DisplayName "Allow SSH"

# ---- Enable a rule ----
Enable-NetFirewallRule -DisplayName "Allow SSH"

# ---- Remove a rule ----
Remove-NetFirewallRule -DisplayName "Allow SSH"

# ---- Modify a rule ----
Set-NetFirewallRule -DisplayName "Allow SSH" -RemoteAddress 10.0.0.0/24

# ============================================================
# PROFILE MANAGEMENT
# ============================================================

# ---- Enable/Disable firewall for a profile ----
Set-NetFirewallProfile -Name Public -Enabled True
Set-NetFirewallProfile -Name Private -Enabled True

# ---- Set default inbound action for a profile ----
Set-NetFirewallProfile -Name Public -DefaultInboundAction Block
Set-NetFirewallProfile -Name Public -DefaultOutboundAction Allow

# ---- Enable logging for a profile ----
Set-NetFirewallProfile -Name Public `
  -LogAllowed True `
  -LogBlocked True `
  -LogFileName "C:\Windows\System32\LogFiles\Firewall\pfirewall.log" `
  -LogMaxSizeKilobytes 4096
```

### 11.4 Managing Windows Firewall via netsh (Legacy but Useful)

```cmd
REM ---- Show firewall state ----
netsh advfirewall show allprofiles

REM ---- Turn firewall on/off ----
netsh advfirewall set allprofiles state on
netsh advfirewall set allprofiles state off

REM ---- Add an inbound rule ----
netsh advfirewall firewall add rule name="Allow SSH" ^
  dir=in action=allow protocol=tcp localport=22

REM ---- Block an IP ----
netsh advfirewall firewall add rule name="Block Attacker" ^
  dir=in action=block remoteip=45.33.32.156

REM ---- Delete a rule ----
netsh advfirewall firewall delete rule name="Allow SSH"

REM ---- Show all rules ----
netsh advfirewall firewall show rule name=all

REM ---- Export firewall config (backup) ----
netsh advfirewall export "C:\firewall-backup.wfw"

REM ---- Import firewall config (restore) ----
netsh advfirewall import "C:\firewall-backup.wfw"
```

### 11.5 OSCP Relevance — Windows Firewall for Pentesters

During OSCP, you'll often need to modify Windows Firewall rules after gaining access:

```powershell
# ---- Disable Windows Firewall entirely (post-exploitation) ----
# WARNING: Only do this in authorized penetration tests!
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled False

# ---- Allow reverse shell traffic ----
New-NetFirewallRule -DisplayName "svchost-update" `
  -Direction Outbound `
  -Protocol TCP `
  -RemotePort 4444 `
  -Action Allow

# ---- Allow incoming bind shell ----
New-NetFirewallRule -DisplayName "system-service" `
  -Direction Inbound `
  -Protocol TCP `
  -LocalPort 4444 `
  -Action Allow

# ---- Check if firewall is blocking your shell ----
Get-NetFirewallRule -Direction Inbound -Action Block |
  Where-Object { $_.Enabled -eq 'True' } |
  Get-NetFirewallPortFilter |
  Where-Object { $_.LocalPort -eq 4444 }
```

---

## 12. pfSense and OPNsense — Dedicated Firewall Appliances

### 12.1 What Is pfSense?

**pfSense** is a free, open-source firewall/router distribution based on **FreeBSD**. It's one of the most popular dedicated firewall platforms in the world, used by home labs, small businesses, and enterprises. It turns any commodity x86 hardware (or VM) into a fully featured enterprise-grade firewall.

**OPNsense** is a fork of pfSense with a modern UI, more frequent updates, and some additional features (like built-in Zenarmor/Sensei for NGFW functionality).

### 12.2 pfSense Architecture

```
                    pfSense Deployment Architecture

                         INTERNET
                            │
                            ▼
                    ┌───────────────┐
                    │   ISP Modem   │
                    └───────┬───────┘
                            │
                    ┌───────┴───────┐
                    │   pfSense     │
                    │   Firewall    │
                    │               │
                    │  WAN: em0     │ ← Public IP (DHCP or static)
                    │  LAN: em1     │ ← 192.168.1.1/24
                    │  OPT1: em2    │ ← DMZ: 172.16.0.1/24
                    │  OPT2: em3    │ ← Guest WiFi: 10.10.10.1/24
                    │               │
                    │  Services:    │
                    │  - DHCP       │
                    │  - DNS (Unbound)│
                    │  - OpenVPN    │
                    │  - HAProxy    │
                    │  - Suricata   │
                    │  - pfBlockerNG│
                    └───┬───┬───┬───┘
                        │   │   │
              ┌─────────┘   │   └─────────┐
              ▼             ▼             ▼
        ┌──────────┐  ┌──────────┐  ┌──────────┐
        │ LAN      │  │ DMZ      │  │ Guest    │
        │ Switch   │  │ Switch   │  │ WiFi AP  │
        │          │  │          │  │          │
        │ PCs,     │  │ Web      │  │ Visitor  │
        │ Printers │  │ servers, │  │ devices  │
        │          │  │ Mail     │  │          │
        └──────────┘  └──────────┘  └──────────┘
```

### 12.3 Key pfSense Features for Security Professionals

| Feature              | Description                                                    |
|----------------------|----------------------------------------------------------------|
| **Stateful Firewall** | Full stateful packet inspection with rule sets per interface  |
| **NAT/PAT**          | 1:1 NAT, port forwarding, outbound NAT, NAT reflection       |
| **VPN**              | OpenVPN, IPsec, WireGuard (via package)                       |
| **IDS/IPS**          | Suricata or Snort packages with ET rules                      |
| **DNS Filtering**    | pfBlockerNG for DNS-based ad/malware blocking                 |
| **Traffic Shaping**  | ALTQ-based QoS for bandwidth management                      |
| **VLAN Support**     | 802.1Q VLAN tagging — network segmentation without hardware   |
| **High Availability**| CARP (Common Address Redundancy Protocol) for failover        |
| **Captive Portal**   | Guest WiFi authentication with vouchers                       |
| **Logging/Reporting**| Real-time firewall logs, traffic graphs, Netflow export       |

### 12.4 pfSense Firewall Rule Evaluation

pfSense evaluates rules **per interface**, **top-to-bottom**, **first match wins** — identical to iptables:

```
┌─────────────────────────────────────────────────────────┐
│          pfSense RULE EVALUATION ORDER                   │
│                                                         │
│  1. Floating Rules (global — applied before interface)  │
│     ↓                                                   │
│  2. Interface Group Rules                               │
│     ↓                                                   │
│  3. Interface Rules (WAN, LAN, OPT1, etc.)             │
│     ↓                                                   │
│  4. Default Deny (implicit — block everything else)     │
│                                                         │
│  Within each level: TOP-TO-BOTTOM, FIRST MATCH WINS    │
└─────────────────────────────────────────────────────────┘
```

**Default Behaviors**:
- **WAN interface**: Block ALL inbound by default (most secure)
- **LAN interface**: Allow ALL outbound by default (convenience for users)
- **OPT interfaces**: Block ALL by default (must explicitly create rules)

---

## 13. Cloud Firewalls — AWS, Azure, and GCP

### 13.1 AWS Security Groups

AWS Security Groups are **stateful**, **instance-level** virtual firewalls:

```
┌─────────────────────────────────────────────────────┐
│                   AWS VPC                             │
│                                                     │
│  ┌─────────────────────────────────────────────┐    │
│  │           Security Group: web-sg             │    │
│  │                                             │    │
│  │  Inbound Rules:                             │    │
│  │    TCP 80   from 0.0.0.0/0    (HTTP)       │    │
│  │    TCP 443  from 0.0.0.0/0    (HTTPS)      │    │
│  │    TCP 22   from 10.0.0.0/16  (SSH-mgmt)   │    │
│  │                                             │    │
│  │  Outbound Rules:                            │    │
│  │    All traffic to 0.0.0.0/0   (default)    │    │
│  │                                             │    │
│  │  ┌──────────┐  ┌──────────┐                │    │
│  │  │ EC2: Web │  │ EC2: Web │                │    │
│  │  │ Server 1 │  │ Server 2 │                │    │
│  │  └──────────┘  └──────────┘                │    │
│  └─────────────────────────────────────────────┘    │
│                                                     │
│  ┌─────────────────────────────────────────────┐    │
│  │           Security Group: db-sg              │    │
│  │                                             │    │
│  │  Inbound Rules:                             │    │
│  │    TCP 3306  from web-sg      (MySQL)       │    │
│  │    TCP 22    from 10.0.1.0/24 (SSH-mgmt)   │    │
│  │                                             │    │
│  │  ┌──────────┐                              │    │
│  │  │ EC2: DB  │                              │    │
│  │  │ Server   │                              │    │
│  │  └──────────┘                              │    │
│  └─────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────┘
```

```bash
# ---- AWS CLI: Create a security group ----
aws ec2 create-security-group \
  --group-name web-sg \
  --description "Web server security group" \
  --vpc-id vpc-0123456789abcdef0

# ---- Add inbound rules ----
# Allow HTTP from anywhere
aws ec2 authorize-security-group-ingress \
  --group-id sg-0123456789abcdef0 \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0

# Allow SSH from management subnet only
aws ec2 authorize-security-group-ingress \
  --group-id sg-0123456789abcdef0 \
  --protocol tcp \
  --port 22 \
  --cidr 10.0.0.0/16

# Allow MySQL from another security group (reference by SG ID)
aws ec2 authorize-security-group-ingress \
  --group-id sg-db-group-id \
  --protocol tcp \
  --port 3306 \
  --source-group sg-web-group-id

# ---- List security group rules ----
aws ec2 describe-security-groups \
  --group-ids sg-0123456789abcdef0
```

**AWS NACLs vs Security Groups**:

| Feature            | Security Group                    | Network ACL (NACL)               |
|--------------------|-----------------------------------|----------------------------------|
| **Level**           | Instance (ENI)                    | Subnet                          |
| **Stateful**        | Yes (return traffic auto-allowed) | No (must define both directions)|
| **Rule evaluation** | All rules evaluated (union)       | Rules evaluated in order        |
| **Default action**  | Deny all inbound by default       | Allow all by default            |
| **Use case**        | Per-instance access control       | Subnet-wide guardrails          |

### 13.2 Azure Network Security Groups (NSGs)

```bash
# ---- Azure CLI: Create an NSG ----
az network nsg create \
  --name web-nsg \
  --resource-group myResourceGroup \
  --location eastus

# ---- Add an inbound rule ----
az network nsg rule create \
  --name AllowHTTP \
  --nsg-name web-nsg \
  --resource-group myResourceGroup \
  --priority 100 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --destination-port-ranges 80 443 \
  --source-address-prefixes '*'

# ---- Add SSH from specific subnet ----
az network nsg rule create \
  --name AllowSSH \
  --nsg-name web-nsg \
  --resource-group myResourceGroup \
  --priority 200 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --destination-port-ranges 22 \
  --source-address-prefixes 10.0.0.0/24

# ---- Block all other inbound ----
az network nsg rule create \
  --name DenyAllInbound \
  --nsg-name web-nsg \
  --resource-group myResourceGroup \
  --priority 4096 \
  --direction Inbound \
  --access Deny \
  --protocol '*' \
  --destination-port-ranges '*' \
  --source-address-prefixes '*'
```

### 13.3 GCP Firewall Rules

```bash
# ---- GCP: Create a firewall rule ----
gcloud compute firewall-rules create allow-http \
  --network=default \
  --direction=INGRESS \
  --action=ALLOW \
  --rules=tcp:80,tcp:443 \
  --source-ranges=0.0.0.0/0 \
  --target-tags=web-server

# ---- Allow SSH from IAP (Identity-Aware Proxy) ----
gcloud compute firewall-rules create allow-ssh-iap \
  --network=default \
  --direction=INGRESS \
  --action=ALLOW \
  --rules=tcp:22 \
  --source-ranges=35.235.240.0/20 \
  --target-tags=ssh-enabled

# ---- Deny all other ingress ----
gcloud compute firewall-rules create deny-all-ingress \
  --network=default \
  --direction=INGRESS \
  --action=DENY \
  --rules=all \
  --source-ranges=0.0.0.0/0 \
  --priority=65534
```

---

## 14. Firewall Design Patterns — DMZ, Zones, and Segmentation

### 14.1 The DMZ (Demilitarized Zone) Architecture

A DMZ is a network segment that sits between the untrusted external network (internet) and the trusted internal network. Servers that need to be publicly accessible (web servers, mail servers, DNS servers) are placed in the DMZ.

```
                    CLASSIC DMZ ARCHITECTURE
                    (Two-Firewall Design)
                    
                         INTERNET
                            │
                            ▼
                    ┌───────────────┐
                    │  OUTER        │
                    │  FIREWALL     │  ← Filters internet traffic
                    │  (Perimeter)  │
                    └───────┬───────┘
                            │
                    ┌───────┴───────────────────────┐
                    │            DMZ                  │
                    │                                │
                    │  ┌─────────┐  ┌─────────────┐ │
                    │  │  Web    │  │    Mail      │ │
                    │  │ Server  │  │   Server     │ │
                    │  │ :80/443 │  │   :25/587    │ │
                    │  └─────────┘  └─────────────┘ │
                    │                                │
                    │  ┌─────────┐  ┌─────────────┐ │
                    │  │  DNS    │  │    VPN       │ │
                    │  │ Server  │  │  Endpoint    │ │
                    │  │  :53    │  │              │ │
                    │  └─────────┘  └─────────────┘ │
                    └───────┬───────────────────────┘
                            │
                    ┌───────┴───────┐
                    │  INNER        │
                    │  FIREWALL     │  ← Strict filtering between
                    │  (Internal)   │     DMZ and internal network
                    └───────┬───────┘
                            │
                    ┌───────┴───────────────────────┐
                    │       INTERNAL NETWORK          │
                    │                                │
                    │  ┌──────────┐  ┌────────────┐ │
                    │  │ Database │  │  Domain     │ │
                    │  │ Server   │  │ Controller  │ │
                    │  └──────────┘  └────────────┘ │
                    │                                │
                    │  ┌──────────┐  ┌────────────┐ │
                    │  │ File     │  │  User       │ │
                    │  │ Server   │  │ Workstations│ │
                    │  └──────────┘  └────────────┘ │
                    └───────────────────────────────┘
```

**DMZ Rules of Engagement**:

| From → To        | Allowed?                                                 |
|------------------|---------------------------------------------------------|
| Internet → DMZ   | Yes, but ONLY to specific ports (80, 443, 25, etc.)    |
| Internet → Internal | **NEVER** — blocked by both firewalls                |
| DMZ → Internal   | Very limited (e.g., DB queries on specific port only)   |
| Internal → DMZ   | Usually allowed (admins managing DMZ servers)           |
| Internal → Internet | Allowed (users browsing, updates, etc.)              |
| DMZ → Internet   | Limited (updates, DNS queries only)                     |

### 14.2 Single-Firewall DMZ (Three-Legged Design)

In smaller networks, a single firewall with three interfaces creates the DMZ:

```
                 SINGLE-FIREWALL DMZ (Three-Legged)
                 
                         INTERNET
                            │
                            │ WAN Interface (em0)
                    ┌───────┴───────┐
                    │               │
          DMZ ◄─────┤   pfSense /   ├─────► Internal
     Interface      │   Firewall    │     Interface
      (em2)         │               │      (em1)
                    └───────────────┘
                    
     172.16.0.0/24      Rules:           192.168.1.0/24
     ┌──────────┐       - WAN→DMZ:       ┌──────────┐
     │Web Server│         80,443 only     │   PCs    │
     │Mail Srvr │       - WAN→LAN:       │ Servers  │
     └──────────┘         BLOCKED         │ Printers │
                        - DMZ→LAN:        └──────────┘
                          DB port only
                        - LAN→DMZ: 
                          ALLOWED
                        - LAN→WAN:
                          ALLOWED
```

### 14.3 Network Segmentation with VLANs and Firewall Zones

Modern networks use VLANs combined with firewall zones to create micro-segmentation:

```
    ENTERPRISE NETWORK SEGMENTATION
    
    ┌────────────────────────────────────────────────────┐
    │                   CORE FIREWALL                     │
    │                                                    │
    │  Zone: INTERNET  ──►  WAN interface               │
    │  Zone: DMZ       ──►  VLAN 10  (172.16.10.0/24)  │
    │  Zone: SERVERS   ──►  VLAN 20  (172.16.20.0/24)  │
    │  Zone: USERS     ──►  VLAN 30  (172.16.30.0/24)  │
    │  Zone: MGMT      ──►  VLAN 40  (172.16.40.0/24)  │
    │  Zone: GUEST     ──►  VLAN 50  (172.16.50.0/24)  │
    │  Zone: IOT       ──►  VLAN 60  (172.16.60.0/24)  │
    │                                                    │
    │  Inter-zone policies:                              │
    │  USERS  → INTERNET: Allow (with content filter)    │
    │  USERS  → SERVERS:  Allow (specific ports only)    │
    │  USERS  → MGMT:    DENY                           │
    │  GUEST  → INTERNET: Allow (bandwidth limited)      │
    │  GUEST  → *:        DENY (isolated from everything)│
    │  IOT    → INTERNET: Allow (specific IPs only)      │
    │  IOT    → *:        DENY (quarantined)             │
    │  MGMT   → *:        Allow (admin access)           │
    └────────────────────────────────────────────────────┘
```

---

## 15. NAT/PAT and Firewall Integration

### 15.1 NAT Fundamentals

**Network Address Translation (NAT)** is a firewall function that modifies the source or destination IP addresses in packet headers as they pass through the firewall. It serves two primary purposes:

1. **IPv4 address conservation** — Allow many internal devices to share one public IP
2. **Security** — Hide internal network topology from the internet

### 15.2 Types of NAT

```
SOURCE NAT (SNAT) — Outbound traffic
────────────────────────────────────
Internal host 192.168.1.5 wants to reach google.com

Before NAT:                          After NAT:
┌──────────────────┐                ┌──────────────────┐
│ Src: 192.168.1.5 │  ──Firewall──► │ Src: 203.0.113.1 │
│ Dst: 142.250.x.x │                │ Dst: 142.250.x.x │
└──────────────────┘                └──────────────────┘
  Private IP                          Public IP
  (not routable                       (the firewall's
   on internet)                        WAN address)


DESTINATION NAT (DNAT) — Inbound traffic / Port Forwarding
───────────────────────────────────────────────────────────
External user wants to reach your web server

Before NAT:                          After NAT:
┌──────────────────┐                ┌──────────────────┐
│ Src: 8.8.8.8     │  ──Firewall──► │ Src: 8.8.8.8     │
│ Dst: 203.0.113.1 │                │ Dst: 192.168.1.10│
│ Port: 80         │                │ Port: 80         │
└──────────────────┘                └──────────────────┘
  Public IP                           Private IP
  (firewall WAN)                      (internal web server)


PAT (Port Address Translation) — Many-to-One NAT
─────────────────────────────────────────────────
Multiple internal hosts share ONE public IP by using different source ports

  192.168.1.5:49321  ──►  203.0.113.1:49321  ──►  Internet
  192.168.1.6:50012  ──►  203.0.113.1:50012  ──►  Internet
  192.168.1.7:51444  ──►  203.0.113.1:51444  ──►  Internet
  
  The firewall tracks which internal host corresponds to which port
  in its NAT translation table.
```

### 15.3 NAT Configuration Examples

```bash
# ============================================================
# iptables NAT Configuration
# ============================================================

# ---- Enable IP forwarding (required for NAT) ----
echo 1 > /proc/sys/net/ipv4/ip_forward
# Permanent: add "net.ipv4.ip_forward = 1" to /etc/sysctl.conf

# ---- MASQUERADE: Share internet connection (PAT) ----
# All internal hosts (192.168.1.0/24) share the firewall's
# public IP when accessing the internet through eth0
iptables -t nat -A POSTROUTING -s 192.168.1.0/24 \
  -o eth0 -j MASQUERADE

# ---- SNAT: Same as masquerade but with a static public IP ----
# More efficient than MASQUERADE when the public IP is static
iptables -t nat -A POSTROUTING -s 192.168.1.0/24 \
  -o eth0 -j SNAT --to-source 203.0.113.1

# ---- DNAT: Port forwarding ----
# Forward incoming port 80 on the firewall to internal web server
iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 \
  -j DNAT --to-destination 192.168.1.10:80

# Don't forget to allow the forwarded traffic!
iptables -A FORWARD -i eth0 -o eth1 -p tcp \
  --dport 80 -d 192.168.1.10 -j ACCEPT

# ---- 1:1 NAT: Map a public IP directly to an internal IP ----
iptables -t nat -A PREROUTING -d 203.0.113.5 \
  -j DNAT --to-destination 192.168.1.20
iptables -t nat -A POSTROUTING -s 192.168.1.20 \
  -j SNAT --to-source 203.0.113.5

# ============================================================
# nftables NAT Configuration
# ============================================================

table ip nat {
    chain prerouting {
        type nat hook prerouting priority -100; policy accept;
        
        # Port forwarding: Public port 80 → Internal 192.168.1.10:80
        iif eth0 tcp dport 80 dnat to 192.168.1.10:80
        
        # Port forwarding: Public port 2222 → Internal SSH
        iif eth0 tcp dport 2222 dnat to 192.168.1.10:22
    }

    chain postrouting {
        type nat hook postrouting priority 100; policy accept;
        
        # Masquerade all outbound traffic from LAN
        oif eth0 ip saddr 192.168.1.0/24 masquerade
    }
}
```

---

## 16. VPN Integration with Firewalls

### 16.1 Firewall Rules for VPN Traffic

VPN tunnels must pass through firewalls. You need to allow the VPN protocol AND the traffic that flows through the tunnel:

```bash
# ============================================================
# OpenVPN Firewall Rules (iptables)
# ============================================================

# ---- Allow OpenVPN connections on UDP 1194 ----
iptables -A INPUT -p udp --dport 1194 -j ACCEPT

# ---- Allow traffic on the VPN tunnel interface (tun0) ----
iptables -A INPUT -i tun0 -j ACCEPT
iptables -A FORWARD -i tun0 -j ACCEPT
iptables -A FORWARD -o tun0 -j ACCEPT

# ---- NAT VPN traffic to access the internet through the server ----
iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE

# ---- Allow VPN clients to access internal network ----
iptables -A FORWARD -i tun0 -o eth1 -s 10.8.0.0/24 \
  -d 192.168.1.0/24 -j ACCEPT
iptables -A FORWARD -i eth1 -o tun0 -s 192.168.1.0/24 \
  -d 10.8.0.0/24 -j ACCEPT

# ============================================================
# WireGuard Firewall Rules (iptables)
# ============================================================

# ---- Allow WireGuard UDP port ----
iptables -A INPUT -p udp --dport 51820 -j ACCEPT

# ---- Allow traffic on WireGuard interface ----
iptables -A FORWARD -i wg0 -j ACCEPT
iptables -A FORWARD -o wg0 -j ACCEPT

# ---- NAT for WireGuard clients ----
iptables -t nat -A POSTROUTING -s 10.66.66.0/24 -o eth0 -j MASQUERADE

# ============================================================
# IPsec Firewall Rules (iptables)
# ============================================================

# ---- Allow IKE (key exchange) ----
iptables -A INPUT -p udp --dport 500 -j ACCEPT    # IKEv1/v2
iptables -A INPUT -p udp --dport 4500 -j ACCEPT   # NAT-T

# ---- Allow ESP (Encapsulated Security Payload) ----
iptables -A INPUT -p esp -j ACCEPT

# ---- Allow AH (Authentication Header) ----
iptables -A INPUT -p ah -j ACCEPT

# ============================================================
# UFW VPN Rules
# ============================================================

# ---- OpenVPN ----
sudo ufw allow 1194/udp comment 'OpenVPN'
sudo ufw allow in on tun0
sudo ufw route allow in on tun0 out on eth0

# ---- WireGuard ----
sudo ufw allow 51820/udp comment 'WireGuard'
sudo ufw allow in on wg0
sudo ufw route allow in on wg0 out on eth0
```

### 16.2 VPN + Firewall Architecture

```
            VPN INTEGRATION ARCHITECTURE
            
  Remote Worker                         Corporate Network
  ┌──────────┐                         ┌──────────────────┐
  │ Laptop   │                         │                  │
  │          │     VPN Tunnel          │  ┌────────────┐  │
  │ VPN      │◄═══════════════════════►│  │  VPN       │  │
  │ Client   │   (Encrypted)          │  │  Server    │  │
  │          │                         │  │  (on FW)   │  │
  └──────────┘                         │  └─────┬──────┘  │
                                       │        │         │
  The VPN tunnel passes THROUGH        │  ┌─────┴──────┐  │
  the firewall. The firewall must:     │  │  Firewall  │  │
                                       │  │  Rules:    │  │
  1. Allow the VPN protocol            │  │            │  │
     (UDP 1194, UDP 51820, etc.)       │  │  VPN users │  │
                                       │  │  can access│  │
  2. Allow traffic ON the tunnel       │  │  internal  │  │
     interface (tun0, wg0)             │  │  servers   │  │
                                       │  │  only on   │  │
  3. Apply firewall rules TO the       │  │  specific  │  │
     decrypted VPN traffic (restrict   │  │  ports     │  │
     what VPN users can access)        │  └────────────┘  │
                                       │                  │
                                       │  ┌────────────┐  │
                                       │  │ Internal   │  │
                                       │  │ Servers    │  │
                                       │  └────────────┘  │
                                       └──────────────────┘
```

---

## 17. Firewall Policy Management and Best Practices

### 17.1 The Principle of Least Privilege

Every firewall rule should follow the **Principle of Least Privilege**: allow only the minimum access required for a service to function. Nothing more.

**Bad rule** (overly permissive):
```bash
# WRONG — Allows ALL traffic from ANYWHERE to ANYWHERE
iptables -A INPUT -j ACCEPT
```

**Good rule** (least privilege):
```bash
# RIGHT — Allows ONLY HTTP traffic from a specific subnet
# to a specific server on a specific port
iptables -A INPUT -p tcp -s 10.0.0.0/24 -d 192.168.1.10 \
  --dport 80 -j ACCEPT
```

### 17.2 Firewall Rule Documentation

Every rule should be documented. Use comments in your firewall rules:

```bash
# iptables — Comments via -m comment
iptables -A INPUT -p tcp --dport 22 -s 10.0.0.0/24 -j ACCEPT \
  -m comment --comment "SSH from management VLAN - Ticket #4521"

# UFW — Comments
sudo ufw allow from 10.0.0.0/24 to any port 22 \
  comment 'SSH from management VLAN - Ticket #4521'

# nftables — Comments
nft add rule inet firewall input tcp dport 22 \
  ip saddr 10.0.0.0/24 accept \
  comment "SSH from management VLAN - Ticket #4521"
```

### 17.3 Firewall Change Management Process

```
┌──────────────────────────────────────────────────────┐
│         FIREWALL CHANGE MANAGEMENT WORKFLOW           │
│                                                      │
│  1. REQUEST                                          │
│     └── Business justification submitted             │
│     └── Specific ports, IPs, protocols defined       │
│     └── Duration (permanent or temporary?)           │
│                                                      │
│  2. REVIEW                                           │
│     └── Security team reviews the request            │
│     └── Check for policy violations                  │
│     └── Assess risk impact                           │
│     └── Verify no existing rule covers it            │
│                                                      │
│  3. APPROVE                                          │
│     └── Manager/CISO signs off                       │
│     └── Change ticket created (ITSM)                 │
│                                                      │
│  4. IMPLEMENT                                        │
│     └── Rule added during change window              │
│     └── Tested to verify it works                    │
│     └── Documented in rule set comments              │
│                                                      │
│  5. VERIFY                                           │
│     └── Confirm traffic flows as expected            │
│     └── Verify no unintended side effects            │
│     └── Log monitoring for anomalies                 │
│                                                      │
│  6. REVIEW/EXPIRE                                    │
│     └── Periodic rule review (quarterly/annually)    │
│     └── Remove rules that are no longer needed       │
│     └── Validate rules still have a business need    │
└──────────────────────────────────────────────────────┘
```

### 17.4 Rule Ordering Best Practices

```bash
# ============================================================
# RECOMMENDED RULE ORDER (Top to Bottom)
# ============================================================

# 1. Anti-spoofing rules (block obviously forged traffic)
iptables -A INPUT -s 127.0.0.0/8 ! -i lo -j DROP    # Spoofed localhost
iptables -A INPUT -s 10.0.0.0/8 -i eth0 -j DROP      # Private IP on WAN
iptables -A INPUT -s 172.16.0.0/12 -i eth0 -j DROP   # Private IP on WAN
iptables -A INPUT -s 192.168.0.0/16 -i eth0 -j DROP  # Private IP on WAN

# 2. Explicit DENY rules (block known bad actors)
iptables -A INPUT -s 45.33.32.0/24 -j DROP           # Known attacker range

# 3. Loopback and established connections
iptables -A INPUT -i lo -j ACCEPT
iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
iptables -A INPUT -m conntrack --ctstate INVALID -j DROP

# 4. Specific ALLOW rules (most specific first)
iptables -A INPUT -p tcp --dport 22 -s 10.0.0.0/24 -j ACCEPT
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# 5. ICMP handling
iptables -A INPUT -p icmp --icmp-type echo-request \
  -m limit --limit 1/s -j ACCEPT

# 6. Logging rule (log everything that reaches this point)
iptables -A INPUT -j LOG --log-prefix "DROPPED: "

# 7. Default DENY (explicit — redundant with policy but clear)
iptables -A INPUT -j DROP
```

### 17.5 Firewall Hardening Checklist

```
╔══════════════════════════════════════════════════════════╗
║            FIREWALL HARDENING CHECKLIST                  ║
╠══════════════════════════════════════════════════════════╣
║                                                          ║
║  □ Default policy set to DROP/DENY for INPUT/FORWARD     ║
║  □ Only required ports are open (whitelist approach)     ║
║  □ SSH access restricted to management subnet            ║
║  □ Firewall management interface NOT on public network   ║
║  □ Anti-spoofing rules in place                          ║
║  □ ICMP rate-limited (not fully blocked)                 ║
║  □ Logging enabled for denied traffic                    ║
║  □ Unused interfaces disabled                            ║
║  □ Firewall firmware/software up to date                 ║
║  □ Strong admin password / SSH key authentication        ║
║  □ SNMP disabled or using SNMPv3 with auth               ║
║  □ NTP configured for accurate log timestamps            ║
║  □ Configuration backups stored securely                 ║
║  □ Change management process in place                    ║
║  □ Regular rule review scheduled (quarterly minimum)     ║
║  □ No "allow any any" rules (unless fully justified)     ║
║  □ IPv6 rules configured (or IPv6 disabled if not used)  ║
║  □ Outbound filtering configured (egress filtering)      ║
║  □ DNS traffic restricted (prevent DNS tunneling)        ║
║  □ Geographic IP blocking considered                     ║
║                                                          ║
╚══════════════════════════════════════════════════════════╝
```

---

## 18. Firewall Auditing — Finding Weaknesses Before Attackers Do

### 18.1 What Is a Firewall Audit?

A firewall audit is a systematic review of firewall configurations, rules, and policies to identify:

- **Overly permissive rules** — Rules that allow more access than necessary
- **Unused rules** — Rules with zero hit counts (dead rules)
- **Shadowed rules** — Rules that are never reached because a broader rule above catches the traffic first
- **Conflicting rules** — Rules that contradict each other
- **Missing rules** — Traffic that should be blocked but isn't
- **Compliance violations** — Rules that violate organizational security policy

### 18.2 Manual Audit Techniques

```bash
# ============================================================
# iptables AUDIT COMMANDS
# ============================================================

# ---- Show all rules with packet/byte counters ----
# Rules with 0 packets/bytes may be unused (dead rules)
sudo iptables -L -n -v --line-numbers

# ---- Check for overly permissive rules ----
# Look for rules with source/dest 0.0.0.0/0 (any)
sudo iptables-save | grep "0.0.0.0/0.*0.0.0.0/0"

# ---- Check for "allow any" rules ----
sudo iptables-save | grep "\-j ACCEPT" | grep -v "\-\-dport"

# ---- Check default policies ----
sudo iptables -L | grep "Chain.*policy"
# If you see "policy ACCEPT" on INPUT or FORWARD, that's a problem!

# ---- Look for rules allowing dangerous ports ----
for port in 23 21 3389 445 135 139 1433 3306 5432 6379; do
    echo "=== Port $port ==="
    sudo iptables-save | grep "dport $port"
done

# ---- Check for anti-spoofing rules ----
sudo iptables-save | grep -E "10\.0\.0\.0|172\.16\.0\.0|192\.168\.0\.0"

# ---- Find shadowed rules (rules that can never match) ----
# Export rules and manually analyze order
sudo iptables-save > /tmp/fw-audit.txt
# A rule is shadowed if a broader rule above it matches
# the same traffic first

# ============================================================
# nftables AUDIT COMMANDS
# ============================================================

# ---- List full ruleset with counters ----
sudo nft list ruleset -a    # -a shows handles (for deletion)

# ---- Show specific chain with counters ----
sudo nft list chain inet firewall input

# ---- Monitor firewall in real-time ----
sudo nft monitor

# ============================================================
# UFW AUDIT COMMANDS
# ============================================================

# ---- Show all rules with numbers ----
sudo ufw status numbered verbose

# ---- Check for "Anywhere" rules (overly permissive) ----
sudo ufw status | grep "Anywhere"

# ---- Check logging is enabled ----
sudo ufw status verbose | grep "Logging"

# ============================================================
# firewalld AUDIT COMMANDS
# ============================================================

# ---- List all zones with their rules ----
sudo firewall-cmd --list-all-zones

# ---- Check for open ports in each zone ----
for zone in $(firewall-cmd --get-zones); do
    echo "=== Zone: $zone ==="
    firewall-cmd --zone=$zone --list-ports
    firewall-cmd --zone=$zone --list-services
done

# ---- Check rich rules ----
sudo firewall-cmd --list-rich-rules
```

### 18.3 Automated Audit Tools

| Tool                  | Description                                              |
|-----------------------|----------------------------------------------------------|
| **Nipper**            | Commercial firewall config analysis tool                  |
| **FireMon**           | Enterprise firewall policy management and auditing       |
| **AlgoSec**           | Automated firewall policy analysis and optimization      |
| **Tufin**             | Network security policy orchestration                    |
| **fw-audit (custom)** | Custom scripts that parse iptables-save output           |
| **Nmap**              | Scan from outside to verify what's actually open         |

### 18.4 External Audit — Verify with Nmap

The most reliable way to audit a firewall is to scan it from the outside:

```bash
# ---- Full TCP port scan from external machine ----
# This tells you what ports are ACTUALLY open through the firewall
nmap -sS -p- -T4 -Pn <target-public-ip>

# ---- Service version detection on open ports ----
nmap -sV -sC -p <open-ports> <target-public-ip>

# ---- UDP scan (slower but important) ----
nmap -sU --top-ports 100 <target-public-ip>

# ---- Compare scan results with firewall rules ----
# If a port shows as "open" but should be blocked → MISCONFIGURATION
# If a port shows as "filtered" → Firewall is correctly blocking it
# If a port shows as "closed" → Port is reachable but no service listening

# ---- Check firewall type detection ----
nmap -O --osscan-guess <target-public-ip>

# ---- Test specific ACL bypass techniques ----
# Source port manipulation (some firewalls allow port 53/80 source)
nmap -sS -g 53 -p 22 <target-public-ip>
nmap -sS -g 80 -p 445 <target-public-ip>
```

---

## 19. Firewall Troubleshooting — Diagnosing Connectivity Issues

### 19.1 Systematic Troubleshooting Methodology

When a firewall is blocking legitimate traffic (or allowing illegitimate traffic), follow this diagnostic process:

```
┌─────────────────────────────────────────────────────────┐
│        FIREWALL TROUBLESHOOTING FLOWCHART               │
│                                                         │
│  "Something isn't connecting!"                          │
│                │                                        │
│                ▼                                        │
│  1. Is the service running?                             │
│     └── ss -tlnp | grep <port>                         │
│     └── systemctl status <service>                      │
│         NO → Start the service (not a firewall issue)   │
│         YES → Continue                                  │
│                │                                        │
│                ▼                                        │
│  2. Is the firewall blocking it?                        │
│     └── Check firewall logs for DROPs                   │
│     └── Temporarily disable firewall to test            │
│         (iptables -F or ufw disable)                    │
│         WORKS WITH FW OFF → Firewall issue. Continue    │
│         STILL BROKEN → Not a firewall issue             │
│                │                                        │
│                ▼                                        │
│  3. Which rule is blocking?                             │
│     └── iptables -L -n -v (check hit counts)           │
│     └── Add LOG rule before DROP to see what's dropped  │
│     └── tcpdump on firewall to see packet flow          │
│                │                                        │
│                ▼                                        │
│  4. Fix the rule                                        │
│     └── Add ALLOW rule for the traffic                  │
│     └── Verify rule ORDER is correct                    │
│     └── Test connectivity                               │
│     └── Save rules permanently                          │
│                │                                        │
│                ▼                                        │
│  5. Verify fix doesn't break security                   │
│     └── Rule is as specific as possible                 │
│     └── No overly broad permissions added               │
│     └── Document the change                             │
└─────────────────────────────────────────────────────────┘
```

### 19.2 Diagnostic Commands

```bash
# ============================================================
# NETWORK CONNECTIVITY TESTS
# ============================================================

# ---- Test if a port is reachable ----
nc -zv <target-ip> <port>         # Netcat connection test
# "Connection succeeded" = port is open
# "Connection refused"   = port reachable, service not running
# "Connection timed out" = firewall is probably blocking

# ---- Test with specific source port (firewall bypass test) ----
nc -zv -p 53 <target-ip> <port>   # Source port 53 (DNS)
nc -zv -p 80 <target-ip> <port>   # Source port 80 (HTTP)

# ---- Trace the path and find where blocking occurs ----
traceroute -T -p <port> <target-ip>   # TCP traceroute to specific port
# If traceroute stops at a certain hop, that's likely the firewall

# ---- Check if packet reaches the firewall ----
# Run on the firewall itself:
sudo tcpdump -i eth0 host <source-ip> and port <port>
# If you see SYN packets but no SYN-ACK → firewall or service issue
# If you don't see anything → traffic never reached the firewall

# ============================================================
# FIREWALL-SPECIFIC DIAGNOSTICS
# ============================================================

# ---- iptables: Add temporary logging to diagnose ----
# Insert a LOG rule at the top of INPUT to see what's happening
iptables -I INPUT 1 -p tcp --dport <port> -j LOG \
  --log-prefix "DEBUG-FW: " --log-level 4

# Watch the logs:
sudo tail -f /var/log/syslog | grep "DEBUG-FW"
# or
sudo dmesg -w | grep "DEBUG-FW"

# Don't forget to remove the debug rule when done!
iptables -D INPUT -p tcp --dport <port> -j LOG \
  --log-prefix "DEBUG-FW: " --log-level 4

# ---- iptables: Check connection tracking table ----
sudo conntrack -L
sudo conntrack -L -p tcp --dport <port>
# Shows all tracked connections — useful to see if connections
# are being established or stuck in SYN_SENT state

# ---- iptables: Monitor packet counters in real-time ----
watch -n 1 'iptables -L INPUT -n -v --line-numbers'
# Watch the packet counters increment to see which rule matches

# ---- UFW: Check the actual iptables rules UFW generated ----
sudo iptables -L -n -v | grep <port>

# ---- firewalld: Check if a port/service is allowed ----
sudo firewall-cmd --query-port=<port>/tcp
sudo firewall-cmd --query-service=<service>

# ---- Windows: Test firewall rules ----
# PowerShell:
Test-NetConnection -ComputerName <target> -Port <port>
Get-NetFirewallRule -Direction Inbound -Enabled True |
  Get-NetFirewallPortFilter |
  Where-Object { $_.LocalPort -eq <port> }
```

### 19.3 Common Troubleshooting Scenarios

**Scenario 1: New web server can't be reached externally**
```bash
# 1. Verify Apache/Nginx is running and bound to all interfaces
ss -tlnp | grep ':80'
# If bound to 127.0.0.1 only → change bind address to 0.0.0.0

# 2. Check firewall rules
sudo iptables -L INPUT -n -v | grep 80
# If no rule → add one:
sudo iptables -I INPUT -p tcp --dport 80 -j ACCEPT

# 3. If behind NAT, check port forwarding
sudo iptables -t nat -L PREROUTING -n -v
```

**Scenario 2: Can ping but can't SSH**
```bash
# ICMP (ping) is allowed but TCP port 22 is blocked
# Check specifically for SSH rule:
sudo iptables -L INPUT -n -v | grep "dpt:22"

# Check if SSH is restricted to a specific source:
sudo iptables-save | grep "22"
# Maybe SSH is only allowed from 10.0.0.0/24 and you're connecting
# from a different subnet
```

**Scenario 3: Outbound connections failing**
```bash
# Check OUTPUT chain (often forgotten!)
sudo iptables -L OUTPUT -n -v
# If OUTPUT policy is DROP, you need explicit ALLOW rules for
# outbound traffic (DNS, HTTP, HTTPS, NTP, etc.)

# Check for egress filtering:
sudo iptables -L OUTPUT -n -v | grep DROP
```

---

## 20. Common Mistakes — What Gets Networks Compromised

### 20.1 The Fatal Flaws

```
╔══════════════════════════════════════════════════════════╗
║         TOP 15 FIREWALL MISTAKES                        ║
╠══════════════════════════════════════════════════════════╣
║                                                         ║
║ 1. DEFAULT ALLOW POLICY                                 ║
║    The #1 mistake. If default policy is ACCEPT, any     ║
║    service you forgot to block is exposed.              ║
║    FIX: Always use default DENY/DROP.                   ║
║                                                         ║
║ 2. "ALLOW ANY ANY" RULES                                ║
║    A single "allow everything" rule makes the entire    ║
║    firewall pointless.                                  ║
║    FIX: Every rule must specify source, dest, and port. ║
║                                                         ║
║ 3. NOT ENABLING UFW/FIREWALLD AFTER INSTALL             ║
║    Installing a firewall tool doesn't enable it.        ║
║    UFW starts DISABLED by default on Ubuntu!            ║
║    FIX: sudo ufw enable (after allowing SSH first!)     ║
║                                                         ║
║ 4. FORGETTING TO ALLOW SSH BEFORE ENABLING              ║
║    Enable firewall without SSH rule → locked out         ║
║    of remote server forever.                            ║
║    FIX: ALWAYS "ufw allow ssh" BEFORE "ufw enable"     ║
║                                                         ║
║ 5. WRONG RULE ORDER                                     ║
║    Placing ALLOW rules after a broad DENY rule.         ║
║    First match wins — the ALLOW never triggers.         ║
║    FIX: Specific rules FIRST, default deny LAST.        ║
║                                                         ║
║ 6. NO EGRESS FILTERING                                  ║
║    Only filtering inbound traffic. Malware can freely   ║
║    phone home to C2 servers on any port.                ║
║    FIX: Filter outbound traffic too — block unusual     ║
║    outbound ports and destinations.                     ║
║                                                         ║
║ 7. NOT SAVING RULES (iptables)                          ║
║    Spending hours crafting rules, then rebooting and    ║
║    losing everything because iptables is in-memory.     ║
║    FIX: iptables-save or install iptables-persistent.   ║
║                                                         ║
║ 8. IGNORING IPv6                                        ║
║    Writing perfect IPv4 rules but leaving IPv6 wide     ║
║    open. Attackers use IPv6 to bypass IPv4 firewalls.   ║
║    FIX: Use ip6tables or nftables inet family.          ║
║                                                         ║
║ 9. LEAVING FIREWALL MANAGEMENT ON PUBLIC INTERFACE      ║
║    Web UI or SSH on the WAN-facing interface.           ║
║    FIX: Management ONLY from internal/VPN.              ║
║                                                         ║
║ 10. STALE RULES (NEVER REVIEWING)                       ║
║     Rules from 2019 for servers decommissioned in 2020. ║
║     Every stale rule is potential attack surface.        ║
║     FIX: Quarterly rule review. Delete what's unused.   ║
║                                                         ║
║ 11. NOT LOGGING DROPPED PACKETS                         ║
║     No logs = no visibility = no forensics capability.  ║
║     FIX: Log all dropped/rejected packets.              ║
║                                                         ║
║ 12. ALLOWING ALL ICMP                                   ║
║     ICMP can be used for tunneling, network recon.      ║
║     FIX: Allow echo-request only, rate-limited.         ║
║                                                         ║
║ 13. FLAT NETWORK (NO SEGMENTATION)                      ║
║     One firewall at the edge, nothing internal.         ║
║     Once breached, attacker has full run of network.    ║
║     FIX: Internal firewalls + VLANs + micro-segments.   ║
║                                                         ║
║ 14. TRUSTING SOURCE PORTS                               ║
║     "Allow traffic from source port 53 (DNS)"           ║
║     Attackers can set ANY source port they want.        ║
║     FIX: Never filter on source port alone.             ║
║                                                         ║
║ 15. DISABLING THE FIREWALL "TEMPORARILY"                ║
║     "I'll turn it back on later." You won't.            ║
║     FIX: Never disable. Add the specific rule needed.   ║
║                                                         ║
╚══════════════════════════════════════════════════════════╝
```

---

## 21. Hands-On Labs

### Lab 1: Build a Complete iptables Firewall from Scratch

**Objective**: Configure a Linux machine as a firewall protecting a web server.

**Scenario**: You have a Linux server with two interfaces:
- `eth0`: Connected to the internet (192.168.122.0/24 — simulated WAN)
- `eth1`: Connected to an internal web server (10.0.0.0/24)

**Step-by-Step Solution**:

```bash
# ===========================================================
# LAB 1: Complete iptables Firewall Setup
# ===========================================================

# ---- Step 1: Verify network interfaces ----
ip addr show
# Confirm eth0 has an IP on 192.168.122.0/24
# Confirm eth1 has an IP on 10.0.0.0/24 (e.g., 10.0.0.1)

# ---- Step 2: Enable IP forwarding ----
# Required for the firewall to route packets between interfaces
echo 1 > /proc/sys/net/ipv4/ip_forward

# Make permanent:
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
sysctl -p

# ---- Step 3: Flush existing rules ----
iptables -F
iptables -t nat -F
iptables -X

# ---- Step 4: Set default policies ----
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT

# ---- Step 5: Allow loopback ----
iptables -A INPUT -i lo -j ACCEPT

# ---- Step 6: Allow established connections ----
iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
iptables -A FORWARD -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# ---- Step 7: Allow SSH to the firewall from WAN ----
iptables -A INPUT -i eth0 -p tcp --dport 22 -j ACCEPT

# ---- Step 8: Forward HTTP/HTTPS to internal web server ----
# DNAT: Redirect port 80 to 10.0.0.10:80
iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 \
  -j DNAT --to-destination 10.0.0.10:80

# DNAT: Redirect port 443 to 10.0.0.10:443
iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 443 \
  -j DNAT --to-destination 10.0.0.10:443

# Allow the forwarded traffic
iptables -A FORWARD -i eth0 -o eth1 -p tcp -d 10.0.0.10 \
  --dport 80 -j ACCEPT
iptables -A FORWARD -i eth0 -o eth1 -p tcp -d 10.0.0.10 \
  --dport 443 -j ACCEPT

# ---- Step 9: Allow internal network to reach internet ----
iptables -A FORWARD -i eth1 -o eth0 -s 10.0.0.0/24 -j ACCEPT

# ---- Step 10: NAT outbound traffic ----
iptables -t nat -A POSTROUTING -o eth0 -s 10.0.0.0/24 -j MASQUERADE

# ---- Step 11: ICMP (ping) with rate limiting ----
iptables -A INPUT -p icmp --icmp-type echo-request \
  -m limit --limit 1/s --limit-burst 4 -j ACCEPT

# ---- Step 12: Log dropped packets ----
iptables -A INPUT -j LOG --log-prefix "FW-INPUT-DROP: " --log-level 4
iptables -A FORWARD -j LOG --log-prefix "FW-FORWARD-DROP: " --log-level 4

# ---- Step 13: Save rules ----
iptables-save > /etc/iptables/rules.v4

# ---- Step 14: Verify ----
echo "=== INPUT Chain ==="
iptables -L INPUT -n -v --line-numbers
echo ""
echo "=== FORWARD Chain ==="
iptables -L FORWARD -n -v --line-numbers
echo ""
echo "=== NAT Table ==="
iptables -t nat -L -n -v

# ---- Step 15: Test from another machine ----
# From an external machine, test:
#   curl http://<firewall-wan-ip>        → Should reach web server
#   ssh user@<firewall-wan-ip>           → Should connect to firewall
#   nmap -sS -p 1-1000 <firewall-wan-ip> → Should show 22, 80, 443 open
```

### Lab 2: UFW Web Server Hardening

**Objective**: Use UFW to secure an Ubuntu web server running Apache, SSH, and a PostgreSQL database.

```bash
# ===========================================================
# LAB 2: UFW Web Server Hardening
# ===========================================================

# ---- Step 1: Reset UFW to clean state ----
sudo ufw reset
sudo ufw default deny incoming
sudo ufw default allow outgoing

# ---- Step 2: Allow SSH (BEFORE enabling!) ----
sudo ufw allow from 10.0.0.0/24 to any port 22 proto tcp \
  comment 'SSH from management subnet only'

# ---- Step 3: Allow web traffic ----
sudo ufw allow 80/tcp comment 'HTTP'
sudo ufw allow 443/tcp comment 'HTTPS'

# ---- Step 4: Allow PostgreSQL from app server only ----
sudo ufw allow from 10.0.0.5 to any port 5432 proto tcp \
  comment 'PostgreSQL from app server'

# ---- Step 5: Enable logging ----
sudo ufw logging medium

# ---- Step 6: Enable the firewall ----
sudo ufw enable

# ---- Step 7: Verify configuration ----
sudo ufw status verbose numbered

# ---- Step 8: Test ----
# From management subnet: ssh user@server    → Works
# From external: ssh user@server              → Blocked
# From anywhere: curl http://server           → Works
# From 10.0.0.5: psql -h server              → Works
# From 10.0.0.6: psql -h server              → Blocked

# Expected output of ufw status:
# Status: active
# Logging: on (medium)
# Default: deny (incoming), allow (outgoing), disabled (routed)
#
#      To                         Action      From
#      --                         ------      ----
# [ 1] 22/tcp                     ALLOW IN    10.0.0.0/24  # SSH
# [ 2] 80/tcp                     ALLOW IN    Anywhere     # HTTP
# [ 3] 443/tcp                    ALLOW IN    Anywhere     # HTTPS
# [ 4] 5432/tcp                   ALLOW IN    10.0.0.5     # PostgreSQL
```

### Lab 3: Multi-Zone firewalld Configuration

**Objective**: Configure firewalld on a CentOS/RHEL server with multiple network interfaces in different security zones.

```bash
# ===========================================================
# LAB 3: firewalld Multi-Zone Setup
# ===========================================================

# ---- Step 1: Start and enable firewalld ----
sudo systemctl start firewalld
sudo systemctl enable firewalld

# ---- Step 2: Assign interfaces to zones ----
# eth0 (internet-facing) → public zone
sudo firewall-cmd --zone=public --change-interface=eth0 --permanent

# eth1 (internal LAN) → internal zone
sudo firewall-cmd --zone=internal --change-interface=eth1 --permanent

# eth2 (DMZ servers) → dmz zone
sudo firewall-cmd --zone=dmz --change-interface=eth2 --permanent

# ---- Step 3: Configure the PUBLIC zone (internet-facing) ----
# Only allow HTTP and HTTPS from the internet
sudo firewall-cmd --zone=public --add-service=http --permanent
sudo firewall-cmd --zone=public --add-service=https --permanent
# Remove any default services that shouldn't be public
sudo firewall-cmd --zone=public --remove-service=ssh --permanent
sudo firewall-cmd --zone=public --remove-service=dhcpv6-client --permanent

# ---- Step 4: Configure the INTERNAL zone (LAN) ----
# Allow more services for internal users
sudo firewall-cmd --zone=internal --add-service=ssh --permanent
sudo firewall-cmd --zone=internal --add-service=http --permanent
sudo firewall-cmd --zone=internal --add-service=https --permanent
sudo firewall-cmd --zone=internal --add-service=dns --permanent
sudo firewall-cmd --zone=internal --add-service=ntp --permanent

# ---- Step 5: Configure the DMZ zone ----
# DMZ servers should only accept web traffic
sudo firewall-cmd --zone=dmz --add-service=http --permanent
sudo firewall-cmd --zone=dmz --add-service=https --permanent

# Allow SSH to DMZ only from internal network (rich rule)
sudo firewall-cmd --zone=dmz --permanent --add-rich-rule='
  rule family="ipv4"
  source address="192.168.1.0/24"
  service name="ssh"
  accept'

# ---- Step 6: Add port forwarding ----
# Forward public port 80 to DMZ web server at 172.16.0.10
sudo firewall-cmd --zone=public --permanent \
  --add-forward-port=port=80:proto=tcp:toport=80:toaddr=172.16.0.10
sudo firewall-cmd --zone=public --permanent --add-masquerade

# ---- Step 7: Apply all changes ----
sudo firewall-cmd --reload

# ---- Step 8: Verify configuration ----
echo "=== Active Zones ==="
sudo firewall-cmd --get-active-zones

echo "=== Public Zone ==="
sudo firewall-cmd --zone=public --list-all

echo "=== Internal Zone ==="
sudo firewall-cmd --zone=internal --list-all

echo "=== DMZ Zone ==="
sudo firewall-cmd --zone=dmz --list-all
```

### Lab 4: Windows Firewall Hardening with PowerShell

**Objective**: Use PowerShell to harden a Windows Server hosting IIS (web server).

```powershell
# ===========================================================
# LAB 4: Windows Firewall Hardening
# ===========================================================

# ---- Step 1: Ensure firewall is enabled on all profiles ----
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled True

# ---- Step 2: Set default inbound to Block ----
Set-NetFirewallProfile -Profile Domain,Public,Private `
  -DefaultInboundAction Block `
  -DefaultOutboundAction Allow

# ---- Step 3: Enable logging on all profiles ----
Set-NetFirewallProfile -Profile Domain,Public,Private `
  -LogAllowed True `
  -LogBlocked True `
  -LogFileName "C:\Windows\System32\LogFiles\Firewall\pfirewall.log" `
  -LogMaxSizeKilobytes 16384

# ---- Step 4: Allow inbound HTTP/HTTPS (IIS) ----
New-NetFirewallRule -DisplayName "Allow HTTP (IIS)" `
  -Direction Inbound -Protocol TCP -LocalPort 80 `
  -Action Allow -Profile Any

New-NetFirewallRule -DisplayName "Allow HTTPS (IIS)" `
  -Direction Inbound -Protocol TCP -LocalPort 443 `
  -Action Allow -Profile Any

# ---- Step 5: Allow RDP from management subnet only ----
New-NetFirewallRule -DisplayName "Allow RDP (Mgmt Only)" `
  -Direction Inbound -Protocol TCP -LocalPort 3389 `
  -RemoteAddress 10.0.0.0/24 `
  -Action Allow -Profile Domain,Private

# ---- Step 6: Allow WinRM from management subnet ----
New-NetFirewallRule -DisplayName "Allow WinRM (Mgmt Only)" `
  -Direction Inbound -Protocol TCP -LocalPort 5985,5986 `
  -RemoteAddress 10.0.0.0/24 `
  -Action Allow -Profile Domain

# ---- Step 7: Block known malicious IPs ----
$blocked_ips = @("45.33.32.156", "198.51.100.66", "203.0.113.99")
foreach ($ip in $blocked_ips) {
    New-NetFirewallRule -DisplayName "Block Malicious: $ip" `
      -Direction Inbound -RemoteAddress $ip -Action Block
}

# ---- Step 8: Block outbound to known C2 servers ----
New-NetFirewallRule -DisplayName "Block C2 Outbound" `
  -Direction Outbound `
  -RemoteAddress 198.51.100.0/24 `
  -Action Block

# ---- Step 9: Verify all rules ----
Get-NetFirewallRule -Enabled True |
  Sort-Object Direction, DisplayName |
  Format-Table DisplayName, Direction, Action, Profile -AutoSize

# ---- Step 10: Export config for backup ----
netsh advfirewall export "C:\Backups\fw-config-$(Get-Date -Format 'yyyyMMdd').wfw"
```

### Lab 5: Firewall Audit — Find the Misconfigurations

**Objective**: Given a set of iptables rules, identify all security issues.

**The Misconfigured Ruleset** (find all the problems):

```bash
# This ruleset has MULTIPLE security issues. Can you find them all?
iptables -P INPUT ACCEPT                           # Issue #1
iptables -P FORWARD ACCEPT                         # Issue #2
iptables -A INPUT -p tcp --dport 22 -j ACCEPT      # Issue #3
iptables -A INPUT -p tcp --dport 23 -j ACCEPT      # Issue #4
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 3306 -j ACCEPT    # Issue #5
iptables -A INPUT -s 10.0.0.0/8 -j ACCEPT         # Issue #6
iptables -A INPUT -j ACCEPT                        # Issue #7
iptables -A INPUT -p tcp --dport 445 -j DROP       # Issue #8
# No logging configured                            # Issue #9
# No IPv6 rules (ip6tables not configured)         # Issue #10
```

**Solution — All Issues Identified**:

| #  | Issue                                      | Severity  | Fix                                              |
|----|-------------------------------------------|-----------|--------------------------------------------------|
| 1  | Default INPUT policy is ACCEPT            | CRITICAL  | Change to `iptables -P INPUT DROP`               |
| 2  | Default FORWARD policy is ACCEPT          | CRITICAL  | Change to `iptables -P FORWARD DROP`             |
| 3  | SSH open to entire internet (no source)   | HIGH      | Restrict to management subnet `-s 10.0.0.0/24`  |
| 4  | Telnet (port 23) is NEVER acceptable      | CRITICAL  | Remove this rule entirely — use SSH only          |
| 5  | MySQL (3306) open to internet             | CRITICAL  | Restrict to app server IP only                   |
| 6  | Entire 10.0.0.0/8 allowed (way too broad) | HIGH      | Use specific subnet like 10.0.0.0/24             |
| 7  | "ACCEPT ALL" rule makes firewall useless  | CRITICAL  | Remove this rule entirely                        |
| 8  | SMB DROP rule is SHADOWED by rule #7      | HIGH      | Move DENY rules before ACCEPT rules              |
| 9  | No logging — no forensics capability      | MEDIUM    | Add LOG rules before final DROP                  |
| 10 | No IPv6 rules — IPv6 completely open      | HIGH      | Configure ip6tables or use nftables inet family  |

---

## 22. Interview Questions — Prove Your Mastery

### Q1: What is a firewall and how does it protect a network?

**Expert Answer**: A firewall is a network security device — implemented in hardware, software, or both — that monitors incoming and outgoing network traffic and makes allow/deny decisions based on a predefined set of security rules. It protects a network by creating a controlled chokepoint between trusted and untrusted zones. Network firewalls are typically deployed immediately after the gateway router, ensuring ALL traffic entering or leaving the network must pass through the inspection point. The firewall examines packet headers (source/destination IP, ports, protocol, TCP flags) and, in advanced firewalls, packet content (deep packet inspection). It enforces the organization's security policy by permitting only explicitly authorized traffic and blocking everything else — the principle of default deny. A firewall's effectiveness depends entirely on its rule set: well-crafted rules create a strong security boundary; poorly configured rules create a false sense of security.

### Q2: Explain the difference between Layer 3, Layer 7, NGFW, and AI-based firewalls.

**Expert Answer**: Layer 3 (packet filtering) firewalls make decisions based solely on IP headers and transport headers — source/destination IP, ports, and protocol. They're the fastest but can't inspect content, so a SQL injection to port 80 looks identical to legitimate HTTP traffic. Layer 7 (application layer) firewalls, including WAFs, inspect the actual payload content and understand application protocols. They can detect SQL injection, XSS, and command injection by pattern matching against the request body and URL parameters. Next-Generation Firewalls (NGFWs) combine L3/L4 filtering, stateful inspection, deep packet inspection, plus integrated IPS, application identification, SSL decryption, and user identity awareness in a single device. They can distinguish between different applications running on the same port — for example, identifying Tor browser traffic on port 443 versus legitimate HTTPS. AI-based firewalls add machine learning models that learn "normal" traffic baselines and detect anomalies that signature-based detection would miss, though they require training periods and can produce false positives. Each type adds visibility and intelligence but at the cost of performance, complexity, and price.

### Q3: What is the difference between stateful and stateless firewalls?

**Expert Answer**: A stateless firewall treats every packet as an independent, isolated unit. It has no memory of previous packets and cannot understand the concept of a "connection." You must create explicit rules for both directions of traffic — one rule to allow the outbound request and another to allow the inbound response. This makes rule sets large, complex, and error-prone. A stateful firewall maintains a state table that tracks all active connections. When an internal host initiates a TCP connection (SYN), the firewall records it. When the response (SYN-ACK) arrives, the firewall checks its state table, confirms this is a legitimate response to a known connection, and allows it through without needing a separate inbound rule. This dramatically simplifies rule management and improves security because you can't inject packets that aren't part of an established connection. Modern stateful firewalls track TCP connection states (SYN_SENT, ESTABLISHED, FIN_WAIT, etc.), UDP "connections" (pseudo-state based on source/dest pairs), and ICMP request/reply pairs.

### Q4: Describe the default deny vs. default allow philosophy. Which is correct?

**Expert Answer**: Default deny (whitelist) means the firewall blocks ALL traffic by default, and administrators explicitly create rules to ALLOW specific, authorized traffic. Default allow (blacklist) means the firewall permits ALL traffic by default, and administrators create rules to BLOCK specific, known-bad traffic. Default deny is the universally accepted correct approach for any security-conscious environment. It's mandated by PCI-DSS, HIPAA, SOC2, NIST 800-41, and every major security framework. The reasoning is mathematical: with default allow, you must anticipate and block every possible threat — miss one, and you're compromised. With default deny, you only need to enumerate your legitimate services — miss one, and a service is unavailable (inconvenient but secure). In iptables, this means `iptables -P INPUT DROP`; in UFW, `sudo ufw default deny incoming`; in firewalld, using restrictive zones like `public` or `drop`. The only exception is typically the OUTPUT chain on servers, which is sometimes set to ACCEPT for operational simplicity, though paranoid configurations use default deny for all chains.

### Q5: How does iptables process packets? Explain tables, chains, and the processing order.

**Expert Answer**: iptables organizes rules into tables (filter, nat, mangle, raw) containing chains (INPUT, OUTPUT, FORWARD, PREROUTING, POSTROUTING). When a packet arrives, it first hits the PREROUTING hook where raw table rules apply (conntrack bypass), then nat PREROUTING (DNAT/port forwarding). A routing decision determines if the packet is for this host (→ INPUT chain) or needs forwarding (→ FORWARD chain). Locally generated packets enter at OUTPUT. Before leaving, packets hit POSTROUTING for SNAT/masquerading. Within each chain, rules are evaluated top-to-bottom, first match wins. When a packet matches a rule, the specified target executes: ACCEPT (allow), DROP (discard silently), REJECT (discard with ICMP error), or LOG (log and continue). If no rule matches, the chain's default policy applies. The processing order across tables within the same hook is: raw → mangle → nat → filter. Understanding this pipeline is critical because rule placement in the wrong table or wrong chain position can create subtle security holes or break connectivity.

### Q6: You just deployed a new Linux server. Walk me through securing it with UFW.

**Expert Answer**: First, install UFW if not present: `sudo apt install ufw`. Before enabling anything, set default policies: `sudo ufw default deny incoming` and `sudo ufw default allow outgoing`. This ensures default deny is in place before any rules are added. Next, allow SSH to prevent lockout: `sudo ufw allow from 10.0.0.0/24 to any port 22 proto tcp` — restricted to the management subnet, never from 0.0.0.0/0. Add service-specific rules: `sudo ufw allow 80/tcp` and `sudo ufw allow 443/tcp` for a web server. Enable logging: `sudo ufw logging medium`. Only then enable the firewall: `sudo ufw enable`. Verify with `sudo ufw status verbose numbered`. Important post-deployment steps: verify from an external host that only intended ports are open (`nmap -sS -p- target`), verify SSH still works before closing your current session, and check that the firewall persists across reboots (`sudo systemctl enable ufw`). Advanced hardening: edit `/etc/ufw/before.rules` to add anti-spoofing rules and configure rate limiting for SSH: `sudo ufw limit ssh/tcp`.

### Q7: What is a DMZ and why is it used?

**Expert Answer**: A Demilitarized Zone (DMZ) is a physically or logically isolated network segment that sits between the untrusted external network (internet) and the trusted internal network. It hosts services that must be publicly accessible — web servers, mail servers, DNS servers, VPN endpoints — while preventing direct access from the internet to the internal network. The architecture typically uses either two firewalls (outer firewall between internet and DMZ, inner firewall between DMZ and LAN) or a single firewall with three interfaces (WAN, DMZ, LAN). The critical rule: internet-to-DMZ traffic is allowed on specific ports, DMZ-to-internal traffic is severely restricted (only specific connections like database queries), and internet-to-internal traffic is never permitted. If an attacker compromises a DMZ server, they're contained in the DMZ and still face the inner firewall before reaching critical internal assets like domain controllers, databases, and file servers. This compartmentalization is a core principle of defense-in-depth.

### Q8: How would you troubleshoot a firewall that appears to be blocking legitimate traffic?

**Expert Answer**: Systematic approach: First, verify the service is actually running and listening on the expected interface and port: `ss -tlnp | grep <port>`. A service bound to 127.0.0.1 won't accept external connections regardless of firewall rules. Second, temporarily add a LOG rule to see what's being dropped: `iptables -I INPUT 1 -p tcp --dport <port> -j LOG --log-prefix "DEBUG: "` and monitor `dmesg -w`. Third, check connection tracking: `conntrack -L -p tcp --dport <port>` to see if connections are being tracked or stuck in abnormal states. Fourth, use tcpdump on the firewall itself: `tcpdump -i eth0 -n port <port>` to confirm packets are arriving. Fifth, check rule order: `iptables -L -n -v --line-numbers` — look at packet counters to see which rule is matching. A common cause is a broad DROP rule positioned above the intended ALLOW rule. Sixth, as a last resort (and only temporarily), flush rules to confirm the firewall is the issue: `iptables -F && iptables -P INPUT ACCEPT`. If traffic flows, the firewall was the issue. Immediately restore rules and add the correct ALLOW rule. Never leave a server with flushed rules in production.

### Q9: Explain the difference between DROP and REJECT. When should you use each?

**Expert Answer**: DROP silently discards the packet with no response — the sender receives nothing, as if the packet disappeared into a void. The sender eventually times out. REJECT discards the packet AND sends an error response back — typically an ICMP "port unreachable" or TCP RST. From a security perspective, DROP is preferred on external-facing firewalls because it gives attackers zero information: they can't distinguish between "host exists but port is filtered" and "host doesn't exist." This makes port scanning significantly slower because Nmap must wait for timeouts on every filtered port. REJECT is sometimes preferred on internal networks where legitimate users need immediate feedback that a service is blocked (rather than waiting 30+ seconds for a timeout), and on OUTPUT chains where you want internal applications to fail fast when trying to reach blocked destinations. The tradeoff: DROP improves security posture against external reconnaissance but can cause connection delays for legitimate users; REJECT provides faster failure feedback but confirms the host exists and is actively filtering.

### Q10: What is the difference between AWS Security Groups and Network ACLs?

**Expert Answer**: Security Groups operate at the instance level (ENI — Elastic Network Interface) and are stateful — if inbound traffic is allowed, the return traffic is automatically permitted. All rules are evaluated together as a union (if any rule allows, traffic passes). They only support ALLOW rules — there are no explicit DENY rules. Default behavior: deny all inbound, allow all outbound. Network ACLs operate at the subnet level and are stateless — you must configure rules for both directions independently. Rules are evaluated in order (lowest number first), and the first matching rule applies (first match wins). They support both ALLOW and DENY rules. Default behavior: allow all traffic. Best practice is defense-in-depth: use NACLs as broad subnet-level guardrails (e.g., block known bad IP ranges) and Security Groups for fine-grained instance-level access control (e.g., only web servers accept port 80). A common mistake is relying solely on Security Groups and ignoring NACLs, leaving no subnet-level protection against misconfigured instances.

### Q11: How do you persist iptables rules across reboots?

**Expert Answer**: iptables rules are stored in kernel memory and are lost on reboot by default. There are several persistence methods depending on the distribution. On Debian/Ubuntu: install `iptables-persistent` package (`apt install iptables-persistent`), which saves rules to `/etc/iptables/rules.v4` and `/etc/iptables/rules.v6` and automatically restores them at boot via a systemd service. To update saved rules: `iptables-save > /etc/iptables/rules.v4`. On RHEL/CentOS (with iptables service): `service iptables save` writes to `/etc/sysconfig/iptables`, and `systemctl enable iptables` ensures restoration at boot. On systems using nftables: rules are saved to `/etc/nftables.conf` and loaded by the `nftables.service` unit. A distribution-agnostic approach is creating a custom systemd service or `@reboot` cron job that runs `iptables-restore < /path/to/rules`. Critical warning: if you save rules that lock you out (e.g., no SSH rule), you'll be locked out permanently — always verify SSH access is preserved before saving.

### Q12: A pentest reveals port 445 (SMB) is accessible from the internet through the firewall. How do you fix this?

**Expert Answer**: This is a critical finding — SMB should never be exposed to the internet (WannaCry, EternalBlue, and countless other exploits target port 445). Immediate response: add an explicit block rule at the top of the firewall: `iptables -I INPUT 1 -i eth0 -p tcp --dport 445 -j DROP` and for UDP: `iptables -I INPUT 2 -i eth0 -p udp --dport 445 -j DROP`. Then investigate why it was open. Check for: an explicit ALLOW rule for port 445 (`iptables-save | grep 445`), an overly broad ALLOW rule like `-j ACCEPT` with no port restriction, or a default ACCEPT policy that's letting everything through. Root cause remediation: if the default policy is ACCEPT, change it to DROP. If there's a broad ALLOW rule, remove it and replace with specific per-port rules. Additionally, block other SMB-related ports: 135 (RPC), 137-139 (NetBIOS). Verify the fix from outside: `nmap -sS -p 135,137-139,445 <public-ip>` — all should show "filtered." Finally, audit the entire ruleset for other accidentally exposed internal services, document the incident, and implement quarterly firewall audits to prevent recurrence.

### Q13: What is egress filtering and why is it important?

**Expert Answer**: Egress filtering is the practice of controlling outbound traffic leaving your network — filtering what goes OUT, not just what comes IN. Most organizations focus entirely on inbound filtering and ignore egress, which is a critical mistake. Without egress filtering, if an attacker compromises an internal host, they can freely: establish reverse shells to C2 servers on any port, exfiltrate data over DNS tunneling (port 53), HTTPS (port 443), or custom protocols, download additional tools and malware, and participate in botnets or DDoS attacks. Effective egress filtering includes: restricting outbound traffic to only required ports (80, 443, 53 for most workstations), forcing all DNS through an internal DNS server (blocking direct DNS to prevent DNS tunneling), blocking outbound SMTP except from authorized mail servers (prevents spam relay), blocking connections to known malicious IP ranges and Tor exit nodes, and monitoring for unusual outbound traffic volumes or destinations. Implementation: `iptables -P OUTPUT DROP` followed by explicit ALLOW rules for authorized outbound traffic. This is harder to maintain but dramatically improves post-compromise containment.

### Q14: Compare UFW and firewalld. When would you use each?

**Expert Answer**: UFW (Uncomplicated Firewall) is Ubuntu/Debian's default, designed for simplicity with a flat allow/deny rule model. It's ideal for single-interface servers, development environments, and teams that want quick, readable firewall management. Commands are intuitive: `ufw allow 80/tcp`. firewalld is RHEL/CentOS/Fedora's default, using a zone-based architecture where interfaces are assigned to security zones with different trust levels. It excels in multi-interface environments where different interfaces need different security postures (e.g., public-facing eth0 in "public" zone, internal eth1 in "internal" zone, management eth2 in "trusted" zone). firewalld also supports runtime vs. permanent rules, allowing temporary changes that don't survive reloads — useful for testing. Both are front-ends to iptables/nftables. Use UFW for simple servers with one or two interfaces. Use firewalld for multi-homed systems, gateway/firewall appliances, or environments requiring zone-based segmentation. In enterprise RHEL environments, firewalld is the standard and expected by auditors. Neither is inherently "better" — they solve different complexity scales.

### Q15: What security risks does NAT introduce or mitigate?

**Expert Answer**: NAT provides incidental security by hiding internal IP addresses — external attackers can't directly address internal hosts because private IPs (10.x, 172.16.x, 192.168.x) aren't routable on the internet. This obscures the internal network topology and prevents direct inbound connections unless port forwarding is configured. However, NAT is NOT a security feature — it's an address conservation technique that happens to have security side effects. Risks NAT introduces: port forwarding creates explicit paths into the network that bypass the hiding benefit; NAT state table exhaustion can be exploited for DoS (flood connections to fill the translation table); NAT breaks end-to-end connectivity and IPsec (requiring NAT-T workarounds); ALGs (Application Layer Gateways) for protocols like FTP and SIP add code complexity and potential vulnerabilities; and NAT creates a false sense of security — administrators assume "we have NAT, so we're protected" and skip proper firewall rules. The critical point: NAT should never be considered a substitute for proper firewall rules. Always configure explicit firewall policies regardless of NAT.

---

## 23. Critical Analysis and Checkpoints

### Self-Assessment Checklist

Before moving on, verify you can answer YES to each of these:

```
□ Can I explain what a firewall is to a complete beginner using an analogy?
□ Can I describe the packet processing pipeline (how a firewall decides)?
□ Can I explain stateful vs. stateless firewalling with examples?
□ Can I classify firewall types (L3, L7/WAF, NGFW, AI-based)?
□ Can I explain why network firewalls sit after the gateway?
□ Can I draw the Netfilter hook architecture from memory?
□ Can I write a complete iptables ruleset from scratch?
□ Can I explain iptables tables (filter, nat, mangle, raw)?
□ Can I configure NAT (SNAT, DNAT, masquerade) with iptables?
□ Can I write an equivalent nftables configuration?
□ Can I harden a server using UFW (including the critical SSH-first step)?
□ Can I configure firewalld zones and assign interfaces?
□ Can I manage Windows Firewall via PowerShell (New-NetFirewallRule)?
□ Can I describe pfSense's role as a dedicated firewall appliance?
□ Can I configure AWS Security Groups and explain vs. NACLs?
□ Can I design a DMZ architecture (two-firewall and three-legged)?
□ Can I implement port forwarding through a firewall?
□ Can I configure firewall rules for VPN protocols (OpenVPN, WireGuard, IPsec)?
□ Can I audit a firewall ruleset and identify misconfigurations?
□ Can I troubleshoot firewall connectivity issues systematically?
□ Can I explain why egress filtering matters?
□ Can I list at least 10 common firewall mistakes?
```

### Key Relationships to Other Modules

```
This File (11 - Firewalls)
    │
    ├──► [[09_Firewall_IDS_Evasion]]
    │    HOW TO BYPASS the firewalls you learned to build here.
    │    Prerequisite: Understand firewall architecture (this file)
    │    BEFORE learning to evade them (File 09).
    │
    ├──► [[03_Internet_Protocol_Deep_Dive]]
    │    IP headers are what Layer 3 firewalls inspect.
    │    Source/Dest IP, TTL, Protocol field.
    │
    ├──► [[04_Transport_Layer_Mechanics]]
    │    TCP/UDP headers, port numbers, TCP flags (SYN, ACK)
    │    are the basis of all firewall rules.
    │
    ├──► [[10_Tunneling_and_Pivoting]]
    │    VPN tunnels must pass through firewalls.
    │    Tunneling can bypass firewall restrictions.
    │
    ├──► [[01_Networking_Architecture_Models]]
    │    OSI model layers correspond to firewall types.
    │    Network architecture determines firewall placement.
    │
    └──► [[05_DNS_Architecture_and_Attacks]]
         DNS traffic must be permitted through firewalls.
         DNS tunneling bypasses firewalls — egress filtering.
```

### What You Should Build Next

1. **Set up a lab**: Install pfSense in a VM, create WAN/LAN/DMZ interfaces, and practice building rule sets
2. **iptables muscle memory**: Write 10 different firewall scripts from scratch without looking at examples
3. **Break it to understand it**: Configure a firewall, then try to bypass your own rules from an external VM using Nmap and Netcat
4. **Cloud practice**: Create an AWS free-tier account, launch EC2 instances, and practice Security Group configuration
5. **Audit practice**: Download sample firewall configs from CTF challenges and practice identifying misconfigurations

---

> **Cross-Reference**: This file covers firewall **architecture, configuration, and management**. For firewall **evasion and bypass techniques** from a penetration testing perspective, see `[[09_Firewall_IDS_Evasion]]`.

> **Next File**: Continue to `[[12_Wireless_Networking_and_Attacks]]` for wireless security fundamentals, WPA/WPA2 cracking, evil twin attacks, and wireless penetration testing methodology.
