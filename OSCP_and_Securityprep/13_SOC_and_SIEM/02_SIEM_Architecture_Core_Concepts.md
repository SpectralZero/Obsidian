# 02 - SIEM Architecture & Core Concepts: The Brain of Your SOC

---
title: "SIEM Architecture & Core Concepts"
parent: "[[13_SOC_and_SIEM]]"
tags:
  - siem
  - architecture
  - log-management
  - correlation
  - security-analytics
created: 2026-01-31
updated: 2026-01-31
---

> **Executive Summary**: A SIEM (Security Information and Event Management) is the "brain" of your Security Operations Center. It collects logs from every device in your organization, correlates them to detect threats, and provides the interface where SOC analysts do their work. Understanding how a SIEM works "under the hood" is essential for becoming an effective SOC analyst. This chapter breaks down SIEM architecture from absolute beginner to advanced concepts - how data flows through the system, how correlation rules detect attacks, how searches work, and how to think like a SIEM. Whether you're preparing for your first SOC job or want to master SIEM platforms, this is your complete guide.

---

## Table of Contents

1. [What is a SIEM? (Absolute Beginner Explanation)](#1-what-is-a-siem-absolute-beginner-explanation)
2. [The Evolution: From Logs to SIEM](#2-the-evolution-from-logs-to-siem)
3. [Core SIEM Components](#3-core-siem-components)
4. [SIEM Architecture Deep Dive](#4-siem-architecture-deep-dive)
5. [Data Flow: From Source to Alert](#5-data-flow-from-source-to-alert)
6. [Log Collection Methods](#6-log-collection-methods)
7. [Log Parsing & Normalization](#7-log-parsing--normalization)
8. [Data Storage & Indexing](#8-data-storage--indexing)
9. [Correlation Engine Explained](#9-correlation-engine-explained)
10. [Search & Query Capabilities](#10-search--query-capabilities)
11. [Dashboards & Visualizations](#11-dashboards--visualizations)
12. [Alerting & Notification Systems](#12-alerting--notification-systems)
13. [SIEM Use Cases](#13-siem-use-cases)
14. [SIEM vs SOAR vs XDR](#14-siem-vs-soar-vs-xdr)
15. [Scaling & Performance](#15-scaling--performance)
16. [Common SIEM Architectures](#16-common-siem-architectures)
17. [Hands-On: Understanding SIEM Components](#17-hands-on-understanding-siem-components)
18. [Interview Questions](#18-interview-questions)

---

## 1. What is a SIEM? (Absolute Beginner Explanation)

### 1.1 The Simple Definition

**SIEM = Security Information and Event Management**

Think of it as a **"Security Camera System for Your Entire IT Infrastructure"**.

Just like a security camera system:
- **Cameras** = Log sources (firewalls, servers, applications)
- **DVR/Recording System** = SIEM storage
- **Monitor Room** = SIEM dashboard
- **Security Guard** = SOC Analyst
- **Motion Detection** = Correlation rules
- **Alarm** = SIEM alerts

### 1.2 Real-World Analogy

```
WITHOUT SIEM:
┌─────────────────────────────────────────────────────┐
│ IT Team has 100+ different systems                  │
│ Each system has its own log file                    │
│ Logs scattered everywhere                           │
│                                                      │
│ Problem: "Was our network attacked yesterday?"      │
│ Solution: Check 100 different log files manually    │
│           (Takes days! Attacker long gone!)         │
└─────────────────────────────────────────────────────┘

WITH SIEM:
┌─────────────────────────────────────────────────────┐
│ SIEM collects ALL logs in ONE place                 │
│ Analyzes them automatically                         │
│ Detects patterns across ALL systems                 │
│                                                      │
│ Problem: "Was our network attacked yesterday?"      │
│ Solution: Query SIEM (Takes 30 seconds!)            │
│           SIEM already alerted you during attack!   │
└─────────────────────────────────────────────────────┘
```

### 1.3 What Problems Does SIEM Solve?

**Problem 1: Log Chaos**
```
BEFORE SIEM:
- Firewall logs in /var/log/firewall.log
- Windows logs in Event Viewer
- Linux logs in /var/log/syslog
- Web server logs in /var/log/apache/access.log
- Database logs in proprietary format
→ IMPOSSIBLE to see the full picture

WITH SIEM:
- All logs sent to SIEM
- Unified view
- Single search interface
→ See everything in one place
```

**Problem 2: Needle in a Haystack**
```
BEFORE SIEM:
- 10 million log events per day
- 1 suspicious event hidden in those millions
- Manual review = impossible

WITH SIEM:
- Correlation rules analyze ALL events
- Automatically flag suspicious patterns
- Analyst sees only the 1 important alert
→ Find threats instantly
```

**Problem 3: Connecting the Dots**
```
ATTACK SCENARIO:
10:00 AM → Failed login on Server A (logged on Server A)
10:15 AM → Port scan from external IP (logged on Firewall)
10:30 AM → Successful login Server A (logged on Server A)
10:45 AM → Data transferred to external IP (logged on Proxy)

BEFORE SIEM:
- Events in different log files
- No connection made
- Attack succeeds undetected

WITH SIEM:
- SIEM correlates all events
- Recognizes attack pattern
- Alerts: "Brute force followed by data exfiltration!"
→ Attack detected and stopped
```

### 1.4 What Does SIEM Actually DO?

```
┌──────────────────────────────────────────────────┐
│         SIEM CORE FUNCTIONS                      │
├──────────────────────────────────────────────────┤
│                                                  │
│  1. COLLECT  → Gather logs from all sources     │
│  2. STORE    → Retain logs for compliance       │
│  3. PARSE    → Understand different log formats │
│  4. NORMALIZE→ Convert to common format         │
│  5. CORRELATE→ Find attack patterns             │
│  6. ALERT    → Notify analysts of threats       │
│  7. SEARCH   → Query historical data            │
│  8. REPORT   → Generate compliance reports      │
│  9. VISUALIZE→ Dashboards for monitoring        │
│ 10. RESPOND  → Integrate with response tools    │
│                                                  │
└──────────────────────────────────────────────────┘
```

---

## 2. The Evolution: From Logs to SIEM

### 2.1 The History (Why SIEM Exists)

**Era 1: 1990s - Basic Logging**
```
Systems: Individual log files
Problem: No centralization
Analysis: Manual grep/search
Threat Detection: Reactive (after breach)
```

**Era 2: Early 2000s - Log Management Systems**
```
Systems: Syslog servers, log aggregators
Problem: Just storage, no intelligence
Analysis: Centralized but still manual
Threat Detection: Slightly better
```

**Era 3: Mid 2000s - SIM (Security Information Management)**
```
Systems: Early SIEM predecessors
Problem: Only log storage & reporting
Analysis: Better search, compliance focus
Threat Detection: Limited correlation
```

**Era 4: Late 2000s - SEM (Security Event Management)**
```
Systems: Real-time event correlation
Problem: Real-time but no historical analysis
Analysis: Alert-focused
Threat Detection: Better, but incomplete
```

**Era 5: 2010s - Modern SIEM (SIM + SEM)**
```
Systems: Splunk, QRadar, ArcSight
Capabilities: Storage + Real-time + Correlation
Analysis: Advanced analytics
Threat Detection: Sophisticated
```

**Era 6: 2020s - Next-Gen SIEM**
```
Systems: Cloud-native (Sentinel, Chronicle)
Capabilities: AI/ML, automation, XDR integration
Analysis: Machine learning anomaly detection
Threat Detection: Predictive + Behavioral
```

### 2.2 SIM + SEM = SIEM

```
┌─────────────────────────────────────────────────┐
│  SIM (Security Information Management)          │
│  • Long-term log storage                        │
│  • Historical analysis                          │
│  • Compliance reporting                         │
│  • Forensic investigation                       │
└─────────────────────────────────────────────────┘
                     +
┌─────────────────────────────────────────────────┐
│  SEM (Security Event Management)                │
│  • Real-time monitoring                         │
│  • Event correlation                            │
│  • Instant alerting                             │
│  • Active threat detection                      │
└─────────────────────────────────────────────────┘
                     =
┌─────────────────────────────────────────────────┐
│         SIEM (Best of Both Worlds)              │
│  ✓ Store + Analyze + Detect + Alert            │
└─────────────────────────────────────────────────┘
```

---

## 3. Core SIEM Components

### 3.1 The 7 Essential Components

```
┌──────────────────────────────────────────────────────┐
│            SIEM COMPONENT ARCHITECTURE               │
└──────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────┐
│ 1. DATA SOURCES (Inputs)                            │
│    • Firewalls, Servers, Endpoints, Apps            │
└────────────────────┬────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────┐
│ 2. LOG COLLECTORS (Forwarders/Agents)               │
│    • Splunk Universal Forwarder                     │
│    • Elastic Beats                                  │
│    • Syslog                                         │
└────────────────────┬────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────┐
│ 3. PARSING ENGINE                                   │
│    • Extracts fields from raw logs                  │
│    • Identifies log formats                         │
│    • Transforms data                                │
└────────────────────┬────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────┐
│ 4. NORMALIZATION ENGINE                             │
│    • Converts to common format (CIM, ECS)           │
│    • Maps fields (src_ip, dest_ip, etc.)           │
│    • Enriches data (GeoIP, DNS, etc.)              │
└────────────────────┬────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────┐
│ 5. STORAGE & INDEXING                               │
│    • High-speed database                            │
│    • Hot/Warm/Cold tiers                           │
│    • Compressed storage                             │
└────────────────────┬────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────┐
│ 6. CORRELATION ENGINE (The Brain)                   │
│    • Analyzes patterns                              │
│    • Runs detection rules                           │
│    • Generates alerts                               │
└────────────────────┬────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────┐
│ 7. PRESENTATION LAYER                               │
│    • Search interface                               │
│    • Dashboards                                     │
│    • Reporting                                      │
│    • Alert console                                  │
└─────────────────────────────────────────────────────┘
```

### 3.2 Component 1: Data Sources (Explained for Beginners)

**What are data sources?**
Any system that generates security-relevant logs.

**Common Data Sources**:

```
NETWORK DEVICES:
├─ Firewalls (Palo Alto, Fortinet, Checkpoint)
│  Log: Allow/deny decisions, threats detected
│  Example: "DENY src=1.2.3.4 dst=10.0.0.5 port=22"
│
├─ IDS/IPS (Snort, Suricata)
│  Log: Attack signatures detected
│  Example: "ALERT: SQL injection attempt detected"
│
├─ Routers/Switches
│  Log: Network flow, port status
│  Example: "Port GigabitEthernet0/1 down"
│
└─ VPN Concentrators
   Log: VPN connections, authentication
   Example: "User john connected from 5.6.7.8"

ENDPOINTS:
├─ Windows Workstations
│  Log: Event Viewer (Security, System, Application)
│  Example: "EventID 4625: Failed login attempt"
│
├─ Linux/Unix Servers
│  Log: /var/log/auth.log, syslog, messages
│  Example: "Failed password for root from 1.2.3.4"
│
└─ Mac Systems
   Log: unified logging system
   Example: "Login attempt failed for user admin"

SECURITY TOOLS:
├─ Antivirus/EDR (CrowdStrike, Defender)
│  Log: Malware detections, quarantines
│  Example: "Malware detected: Trojan.Generic"
│
├─ DLP (Data Loss Prevention)
│  Log: Data transfer attempts
│  Example: "Blocked email with SSNs attached"
│
└─ Proxy/Web Gateway
   Log: Web traffic, URL filtering
   Example: "User visited malicious site blocked.com"

APPLICATIONS:
├─ Web Servers (Apache, IIS, Nginx)
│  Log: HTTP requests
│  Example: "GET /admin SQL injection attempt"
│
├─ Databases (SQL Server, Oracle, MySQL)
│  Log: Queries, authentication
│  Example: "Failed login for sa account"
│
├─ Active Directory
│  Log: User authentication, group changes
│  Example: "User added to Domain Admins group"
│
└─ Cloud Services (AWS, Azure, GCP)
   Log: CloudTrail, Azure logs
   Example: "S3 bucket made public"
```

### 3.3 Component 2: Log Collectors

**Purpose**: Get logs FROM source TO SIEM.

**Methods**:

**1. Agent-Based Collection**
```
HOW IT WORKS:
- Install small software on source system
- Agent reads local logs
- Sends to SIEM over network
- Lightweight, efficient

EXAMPLES:
- Splunk Universal Forwarder (UF)
- Elastic Beats (Filebeat, Winlogbeat)
- Wazuh agent

PROS:
✓ Reliable delivery
✓ Buffering (if SIEM down, stores locally)
✓ Can compress data
✓ Secure (encryption)

CONS:
✗ Must install on every system
✗ Maintenance overhead
```

**2. Agentless Collection**
```
HOW IT WORKS:
- SIEM pulls logs remotely
- Uses APIs or protocols (Syslog, WMI, SSH)
- No software on source

EXAMPLES:
- Syslog (UDP 514, TCP 514)
- Windows Event Forwarding (WEF)
- API integrations

PROS:
✓ No agent to manage
✓ Good for network devices (can't install agents)

CONS:
✗ Less reliable (UDP can drop packets)
✗ No buffering
✗ Limited preprocessing
```

**3. Cloud-Native Collection**
```
HOW IT WORKS:
- Cloud services send logs to SIEM directly
- Uses cloud-native connectors

EXAMPLES:
- AWS CloudTrail → Splunk
- Azure logs → Sentinel
- Google Cloud → Chronicle

PROS:
✓ Designed for cloud
✓ Real-time
✓ Scalable

CONS:
✗ Vendor-specific
```

### 3.4 Component 3: Parsing Engine

**What is Parsing?**
Breaking down raw log text into structured fields.

**Example**:

```
RAW LOG (Unstructured):
Jan 31 14:23:45 firewall kernel: DROP IN=eth0 OUT= SRC=1.2.3.4 DST=10.0.0.5 PROTO=TCP SPT=54321 DPT=22

AFTER PARSING (Structured):
{
  "timestamp": "2024-01-31T14:23:45",
  "hostname": "firewall",
  "action": "DROP",
  "interface_in": "eth0",
  "src_ip": "1.2.3.4",
  "dest_ip": "10.0.0.5",
  "protocol": "TCP",
  "src_port": 54321,
  "dest_port": 22
}
```

**Why Parsing Matters**:
- **Searchable**: Can query "show all DROP actions"
- **Correlation**: Can match src_ip across different logs
- **Visualization**: Can graph by dest_port

**Parsing Methods**:

**1. Regular Expressions (Regex)**
```
RAW LOG:
Failed password for admin from 1.2.3.4 port 52341

REGEX PATTERN:
Failed password for (?<user>\w+) from (?<ip>\d+\.\d+\.\d+\.\d+) port (?<port>\d+)

EXTRACTED FIELDS:
user = "admin"
ip = "1.2.3.4"
port = "52341"
```

**2. Delimiter-Based (CSV, TSV)**
```
RAW LOG:
2024-01-31,14:23:45,firewall,1.2.3.4,10.0.0.5,DENY

PARSING:
Split by comma
Field1 = Date
Field2 = Time
Field3 = Device
Field4 = Source IP
Field5 = Dest IP
Field6 = Action
```

**3. JSON/XML (Structured Formats)**
```
RAW LOG (JSON):
{"timestamp":"2024-01-31T14:23:45","src":"1.2.3.4","action":"deny"}

PARSING:
Already structured! Just read JSON fields directly.
```

---

## 4. SIEM Architecture Deep Dive

### 4.1 Three-Tier SIEM Architecture

```
┌──────────────────────────────────────────────────────────┐
│                  TIER 1: DATA COLLECTION                 │
├──────────────────────────────────────────────────────────┤
│  Forwarders/Agents on source systems                     │
│  • Installed on 1000s of devices                         │
│  • Lightweight (minimal CPU/RAM)                         │
│  • Compress & forward logs                               │
│  • Buffer locally if network down                        │
├──────────────────────────────────────────────────────────┤
│  Examples:                                                │
│  • Splunk UF on each Windows/Linux server               │
│  • Syslog from network devices                           │
│  • API connectors for cloud services                     │
└──────────────────────────────────────────────────────────┘
                         │
                         ▼
┌──────────────────────────────────────────────────────────┐
│            TIER 2: INDEXING & PROCESSING                 │
├──────────────────────────────────────────────────────────┤
│  Indexers (Heavy Forwarders)                             │
│  • Receive logs from forwarders                          │
│  • Parse & normalize                                     │
│  • Store in searchable index                             │
│  • Distribute across cluster (scalability)              │
├──────────────────────────────────────────────────────────┤
│  Components:                                              │
│  • Parsing pipelines                                     │
│  • Field extraction                                      │
│  • Data enrichment (GeoIP, DNS lookups)                 │
│  • Hot/Warm/Cold data tiers                             │
└──────────────────────────────────────────────────────────┘
                         │
                         ▼
┌──────────────────────────────────────────────────────────┐
│          TIER 3: SEARCH & ANALYTICS                      │
├──────────────────────────────────────────────────────────┤
│  Search Heads                                            │
│  • User interface (web UI)                               │
│  • Run searches across indexers                          │
│  • Correlation engine                                    │
│  • Dashboards & reports                                  │
│  • Alert management                                      │
├──────────────────────────────────────────────────────────┤
│  Users:                                                   │
│  • SOC Analysts (search for threats)                    │
│  • Security Engineers (build rules)                     │
│  • Management (view dashboards)                          │
└──────────────────────────────────────────────────────────┘
```

### 4.2 Distributed vs Standalone SIEM

**Standalone (Small Environment)**
```
┌────────────────────────────────────┐
│     ALL-IN-ONE SIEM SERVER         │
│                                    │
│  • Collection                      │
│  • Parsing                         │
│  • Indexing                        │
│  • Search                          │
│  • All on single server            │
└────────────────────────────────────┘

BEST FOR:
- Small businesses (<100 log sources)
- <10 GB/day log volume
- Budget constraints

EXAMPLES:
- Splunk Free on one server
- ELK stack on one VM
```

**Distributed (Enterprise)**
```
┌─────────────────────────────────────────────────┐
│         DISTRIBUTED SIEM CLUSTER                │
└─────────────────────────────────────────────────┘

  [Forwarders]                    [Indexers]
  (1000s of agents)               (10+ servers)
        │                               │
        ▼                               ▼
    ┌───────┐                      ┌────────┐
    │Agent 1│───┐              ┌──▶│Index 1 │
    │Agent 2│───┼──────────────┤   │Index 2 │
    │Agent 3│───┘   Load       │   │Index 3 │
    │  ...  │      Balancer    └──▶│  ...   │
    └───────┘                      └────┬───┘
                                        │
                                        ▼
                                  [Search Heads]
                                  (3+ servers)
                                   ┌──────────┐
                                   │ Search 1 │
                                   │ Search 2 │
                                   │ Search 3 │
                                   └──────────┘

BEST FOR:
- Enterprises (1000+ log sources)
- 100GB+ per day
- High availability requirements
- Geographic distribution
```

---

## 5. Data Flow: From Source to Alert

### 5.1 The Complete Journey (Step-by-Step)

```
STEP 1: EVENT OCCURS
┌─────────────────────────────────────┐
│ User types wrong password           │
│ Windows generates Event ID 4625     │
│ Written to Security.evtx log        │
└─────────────────────────────────────┘
           │
           ▼
STEP 2: LOG COLLECTION
┌─────────────────────────────────────┐
│ Splunk Forwarder detects new event  │
│ Reads: "Failed login for user bob"  │
│ Timestamps and forwards to SIEM     │
└─────────────────────────────────────┘
           │
           ▼
STEP 3: PARSING
┌─────────────────────────────────────┐
│ SIEM receives raw log string        │
│ Applies parsing rule                │
│ Extracts fields:                    │
│   event_id = 4625                   │
│   user = "bob"                      │
│   src_ip = "192.168.1.105"         │
└─────────────────────────────────────┘
           │
           ▼
STEP 4: NORMALIZATION
┌─────────────────────────────────────┐
│ Maps to Common Information Model:   │
│   action = "failure"                │
│   category = "authentication"       │
│   dest = "DC01"                     │
│ Enriches:                           │
│   src_country = "USA" (GeoIP)       │
└─────────────────────────────────────┘
           │
           ▼
STEP 5: INDEXING
┌─────────────────────────────────────┐
│ Stores in searchable index          │
│ Creates inverted index:             │
│   "bob" → [Event123, Event789...]  │
│   "4625" → [Event123, Event456...]  │
│ Now searchable in milliseconds      │
└─────────────────────────────────────┘
           │
           ▼
STEP 6: CORRELATION
┌─────────────────────────────────────┐
│ Correlation rule checks:            │
│ "IF 5+ failed logins for same user  │
│  within 5 minutes → ALERT"          │
│                                     │
│ SIEM detects: 6 failures for "bob"  │
│ within 3 minutes → TRIGGER!         │
└─────────────────────────────────────┘
           │
           ▼
STEP 7: ALERT GENERATION
┌─────────────────────────────────────┐
│ Creates alert:                      │
│   Title: "Brute Force Attack"       │
│   Severity: Medium                  │
│   User: bob                         │
│   Source IP: 192.168.1.105         │
│   Count: 6 attempts                 │
└─────────────────────────────────────┘
           │
           ▼
STEP 8: NOTIFICATION
┌─────────────────────────────────────┐
│ Sends to SOC analyst:               │
│   • Email notification              │
│   • Dashboard popup                 │
│   • Ticket in ServiceNow            │
│   • Slack message                   │
└─────────────────────────────────────┘
           │
           ▼
STEP 9: ANALYST RESPONSE
┌─────────────────────────────────────┐
│ Analyst sees alert                  │
│ Investigates in SIEM                │
│ Takes action (block IP, reset pwd)  │
│ Documents in ticket                 │
└─────────────────────────────────────┘
```

### 5.2 Timeline (How Fast Does This Happen?)

```
EVENT TO ALERT TIMELINE:

00:00.000 - Event occurs (user types wrong password)
00:00.001 - Windows logs to Event Viewer
00:00.050 - Splunk forwarder picks up event
00:00.100 - Sent to SIEM over network
00:00.150 - SIEM parses event
00:00.200 - Event indexed and searchable
00:00.250 - Correlation rule evaluates
00:05.000 - 5 minutes pass, 6 failed logins accumulated
00:05.001 - Correlation rule triggers alert
00:05.002 - Alert appears on dashboard
00:05.010 - Email sent to analyst

TOTAL TIME: ~5 seconds (near real-time!)
```

---

## 6. Log Collection Methods

### 6.1 Method 1: Syslog (Universal Standard)

**What is Syslog?**
A standard protocol for sending log messages over IP networks.

**How it Works**:
```
┌─────────────┐                    ┌─────────────┐
│   SOURCE    │    Syslog          │    SIEM     │
│  (Firewall) │   Message          │   Server    │
│             │─────────────────────▶│             │
│ 10.0.0.10   │  UDP Port 514      │ 10.0.0.100  │
└─────────────┘                    └─────────────┘
```

**Syslog Message Format**:
```
STANDARD SYSLOG MESSAGE:
<34>Jan 31 14:23:45 firewall.company.com kernel: DROP IN=eth0 SRC=1.2.3.4

BREAKDOWN:
<34>                    → Priority (Facility + Severity)
Jan 31 14:23:45         → Timestamp
firewall.company.com    → Hostname
kernel                  → Program/Process
DROP IN=eth0 SRC=1.2.3.4→ Message
```

**Syslog Variants**:
```
SYSLOG (RFC 3164) - Original
- Simple format
- UDP only (unreliable)
- No encryption

SYSLOG-NG (Next Generation)
- Better parsing
- TCP support (reliable)
- Filtering capabilities

RSYSLOG (Reliable Syslog)
- High performance
- Encryption (TLS)
- Database output
- Most popular on Linux
```

**Pros & Cons**:
```
PROS:
✓ Universal (works with almost everything)
✓ Lightweight
✓ Standard protocol
✓ No agent needed

CONS:
✗ UDP unreliable (can lose messages)
✗ No encryption by default
✗ Timestamp issues (timezone sync)
✗ No delivery confirmation
```

### 6.2 Method 2: Agent-Based (Splunk Example)

**Splunk Universal Forwarder (UF)**:

```
INSTALLATION:
1. Download Splunk UF (free)
2. Install on source system
3. Configure inputs.conf (what to monitor)
4. Configure outputs.conf (where to send)
5. Start service

CONFIGURATION EXAMPLE (inputs.conf):
[monitor:///var/log/auth.log]
disabled = false
sourcetype = linux_secure

[WinEventLog://Security]
disabled = false

SENDS TO (outputs.conf):
[tcpout]
defaultGroup = siem_servers

[tcpout:siem_servers]
server = splunk.company.com:9997
```

**How It Works**:
```
┌──────────────────────────────────────┐
│      SOURCE SYSTEM (Web Server)      │
│                                      │
│  ┌────────────────┐                 │
│  │ Log Files:     │                 │
│  │ /var/log/      │                 │
│  │ apache2/       │                 │
│  │ access.log  ◀──┼─────┐          │
│  └────────────────┘     │          │
│                         │          │
│  ┌──────────────────────▼────────┐ │
│  │ Splunk Universal Forwarder    │ │
│  │ • Monitors log files          │ │
│  │ • Detects changes             │ │
│  │ • Compresses data             │ │
│  │ • Forwards to indexer         │ │
│  └───────────────┬───────────────┘ │
└──────────────────┼───────────────────┘
                   │ Encrypted TCP
                   │ Port 9997
                   ▼
          ┌────────────────┐
          │ SIEM Indexer   │
          │ (Splunk)       │
          └────────────────┘
```

### 6.3 Method 3: Windows Event Forwarding (WEF)

**Agentless Windows Log Collection**:

```
ARCHITECTURE:
┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│  Windows    │  │  Windows    │  │  Windows    │
│  Client 1   │  │  Client 2   │  │  Client 3   │
└──────┬──────┘  └──────┬──────┘  └──────┬──────┘
       │                │                │
       └────────────────┼────────────────┘
                        │ WinRM (5985/5986)
                        ▼
              ┌──────────────────┐
              │ WEC (Collector)  │
              │ Windows Server   │
              └────────┬─────────┘
                       │
                       ▼
                  ┌─────────┐
                  │  SIEM   │
                  └─────────┘
```

**Benefits**:
- No agent needed
- Built into Windows
- Centralized collection
- Subscription-based (define what events to collect)

---

## 7. Log Parsing & Normalization

### 7.1 Why Normalization Matters

**The Problem**:
```
SAME EVENT, DIFFERENT FORMATS:

Firewall A (Palo Alto):
type=TRAFFIC,src=1.2.3.4,dst=10.0.0.5,action=deny

Firewall B (Fortinet):
srcip=1.2.3.4 dstip=10.0.0.5 action=blocked

Firewall C (Checkpoint):
src:1.2.3.4;dst:10.0.0.5;action:drop

HOW DO YOU SEARCH ACROSS ALL THREE?
→ Answer: NORMALIZATION
```

**The Solution**:
```
NORMALIZED FORMAT (Common Information Model):

Original Logs:
- Firewall A: src=1.2.3.4 action=deny
- Firewall B: srcip=1.2.3.4 action=blocked
- Firewall C: src:1.2.3.4 action:drop

After Normalization:
- src_ip = 1.2.3.4
- action = "deny"  (standardized value)
- category = "network"

NOW ONE SEARCH WORKS:
"src_ip=1.2.3.4 action=deny"
→ Finds all three!
```

### 7.2 Common Information Model (CIM)

**What is CIM?**
A standard schema for normalizing security data.

**Example: Authentication Events**

```
CIM STANDARD FIELDS:
┌──────────────┬─────────────────────────────────┐
│ Field        │ Description                     │
├──────────────┼─────────────────────────────────┤
│ action       │ success, failure                │
│ app          │ ssh, rdp, web                   │
│ src          │ Source IP or host               │
│ dest         │ Destination host                │
│ user         │ Username                        │
│ signature    │ Event type description          │
└──────────────┴─────────────────────────────────┘

WINDOWS LOGIN (EventID 4624):
Raw: An account was successfully logged on
     Subject: bob
     Logon Type: 3
     Source Network Address: 1.2.3.4

Normalized (CIM):
action = "success"
app = "windows_security"
src = "1.2.3.4"
dest = "DC01"
user = "bob"
signature = "User logged on"

LINUX SSH LOGIN:
Raw: Accepted password for bob from 1.2.3.4 port 52341 ssh2

Normalized (CIM):
action = "success"
app = "ssh"
src = "1.2.3.4"
dest = "web-server-01"
user = "bob"
signature = "User logged on"

NOW SEARCHABLE TOGETHER:
index=* action=success user=bob
→ Returns BOTH Windows AND Linux logins!
```

### 7.3 Elastic Common Schema (ECS)

**Similar to CIM, but for Elastic Stack**:

```
ECS FIELD STRUCTURE:

event.action       → allowed, denied, logged
event.category     → network, authentication, process
event.outcome      → success, failure
source.ip          → Source IP
destination.ip     → Destination IP
user.name          → Username
```

---

## 8. Data Storage & Indexing

### 8.1 How SIEM Stores Data (Indexing Explained)

**What is an Index?**
Think of it like a book's index - helps you find information quickly.

**Without Index (Linear Search)**:
```
SEARCHING 1 BILLION LOGS:
[Log1][Log2][Log3]...[Log 1000000000]
           ↓
Must check EVERY log sequentially
Time: HOURS
```

**With Index (Instant Search)**:
```
INVERTED INDEX:
"bob" → [Log 5, Log 892, Log 5234...]
"failed" → [Log 12, Log 892, Log 9001...]
"1.2.3.4" → [Log 892, Log 1234...]

SEARCH: "failed login bob from 1.2.3.4"
→ Intersection of all three lists
→ Result: Log 892
Time: MILLISECONDS
```

### 8.2 Hot/Warm/Cold Data Tiers

**The Problem**: Storing petabytes of logs forever is expensive!

**The Solution**: Tiered storage based on age & access frequency.

```
┌──────────────────────────────────────────────────────┐
│              TIERED STORAGE MODEL                    │
└──────────────────────────────────────────────────────┘

┌────────────────────────────────────┐
│         HOT DATA (0-7 days)        │
├────────────────────────────────────┤
│ • Most recent logs                 │
│ • Stored on fast SSD               │
│ • Fully searchable                 │
│ • Used for real-time monitoring    │
│ • Highest cost ($$$$)              │
└────────────────────────────────────┘
                ▼
┌────────────────────────────────────┐
│       WARM DATA (7-90 days)        │
├────────────────────────────────────┤
│ • Older logs (investigations)      │
│ • Stored on slower HDD             │
│ • Searchable (slower)              │
│ • Used for forensics               │
│ • Medium cost ($$$)                │
└────────────────────────────────────┘
                ▼
┌────────────────────────────────────┐
│      COLD DATA (90+ days)          │
├────────────────────────────────────┤
│ • Archive logs (compliance)        │
│ • Stored on cheap storage/cloud    │
│ • Searchable (very slow)           │
│ • Used for compliance/legal        │
│ • Low cost ($$)                    │
└────────────────────────────────────┘
                ▼
┌────────────────────────────────────┐
│       FROZEN (Delete/Archive)      │
├────────────────────────────────────┤
│ • Deleted or moved to tape         │
│ • Not searchable                   │
│ • Lowest cost ($)                  │
└────────────────────────────────────┘
```

**Example Retention Policy**:
```
HOT:  Last 7 days   (100 GB, SSD)   - Daily monitoring
WARM: 8-90 days     (1 TB, HDD)     - Investigations
COLD: 91-365 days   (10 TB, Cloud)  - Compliance
FROZEN: 365+ days   (Deleted)       - Meet legal requirements
```

---

## 9. Correlation Engine Explained

### 9.1 What is Correlation?

**Simple Definition**: Connecting multiple events to detect attack patterns.

**Analogy**:
```
CORRELATION = CONNECTING THE DOTS

Event 1: Person enters bank (normal)
Event 2: Person goes to vault (normal)
Event 3: Alarm triggered (abnormal!)
Event 4: Person runs out (abnormal!)

ALONE: Each event might be normal
TOGETHER: Correlation detects bank robbery!
```

### 9.2 Correlation Rule Types

**Type 1: Threshold-Based**
```
RULE: If failed logins > 5 within 5 minutes → Alert

TIME    | EVENT
--------|------------------------
14:00   | Failed login (count: 1)
14:01   | Failed login (count: 2)
14:02   | Failed login (count: 3)
14:03   | Failed login (count: 4)
14:04   | Failed login (count: 5)
14:05   | Failed login (count: 6) → ALERT TRIGGERED!
```

**Type 2: Sequence-Based (Attack Chain)**
```
RULE: If Event A, then Event B, then Event C within 1 hour → Alert

Event A: Port scan detected
   ↓
Event B: Failed SSH login attempts
   ↓
Event C: Successful SSH login
   ↓
Event D: New admin user created
   ↓
ALERT: "Complete attack chain detected!"
```

**Type 3: Frequency-Based (Anomaly Detection)**
```
RULE: If event frequency > 3x normal → Alert

NORMAL BASELINE:
User bob logs in 5 times per day

ANOMALY DETECTED:
User bob logs in 50 times in 1 hour
→ ALERT: "Unusual login frequency for bob"
```

**Type 4: Absence Detection (Missing Events)**
```
RULE: If expected event doesn't occur → Alert

EXPECTED:
Backup server sends "backup completed" log daily at 2 AM

DETECTION:
2:30 AM - No backup log received
→ ALERT: "Backup failed or server down"
```

### 9.3 Real Correlation Rule Example (Splunk SPL)

**Detect Brute Force Attack**:
```
index=windows EventCode=4625
| stats count by user, src_ip
| where count > 5
| eval severity="HIGH"
| table _time, user, src_ip, count, severity
```

**Breakdown**:
- `index=windows EventCode=4625` → Find failed logins
- `stats count by user, src_ip` → Count failures per user/IP
- `where count > 5` → Filter only if > 5 attempts
- `eval severity="HIGH"` → Mark as high severity
- `table` → Display results

---

## 10. Search & Query Capabilities

### 10.1 SIEM Search Languages

**Splunk SPL (Search Processing Language)**:
```
BASIC SEARCH:
index=firewall action=deny

WITH TIME RANGE:
index=firewall action=deny earliest=-1h

WITH STATS:
index=firewall action=deny | stats count by src_ip

WITH CHART:
index=firewall action=deny | timechart count by dest_port
```

**Microsoft Sentinel KQL (Kusto Query Language)**:
```
BASIC SEARCH:
SecurityEvent
| where EventID == 4625

WITH TIME:
SecurityEvent
| where TimeGenerated > ago(1h)
| where EventID == 4625

WITH AGGREGATION:
SecurityEvent
| where EventID == 4625
| summarize count() by Account

WITH JOIN:
SecurityEvent
| where EventID == 4625
| join (SigninLogs) on $left.Account == $right.UserPrincipalName
```

**Elastic EQL (Event Query Language)**:
```
BASIC:
event.category == "process" and process.name == "powershell.exe"

SEQUENCE DETECTION:
sequence
  [process where process.name == "cmd.exe"]
  [network where destination.ip == "1.2.3.4"]
```

### 10.2 Common Search Patterns

**Find All Failed Logins**:
```
SPLUNK:
index=* (EventCode=4625 OR "failed password" OR "authentication failure")

SENTINEL:
SecurityEvent
| where EventID == 4625
union
Syslog
| where SyslogMessage contains "Failed password"
```

**Find Successful Logins After Failed Attempts**:
```
SPLUNK:
index=windows user=bob
| transaction user startswith=eval(EventCode=4625) endswith=eval(EventCode=4624)
| where duration < 300
```

---

## 11. Dashboards & Visualizations

### 11.1 What is a Dashboard?

**Simple Definition**: A visual display showing security data in real-time.

Think of it like a **car dashboard**:
- Speedometer = Events per second
- Fuel gauge = Log ingestion rate
- Warning lights = Active alerts
- Temperature = System health

### 11.2 Dashboard Types

**1. Security Operations Dashboard (SOC Analyst View)**
```
┌────────────────────────────────────────────────────────┐
│           SOC OPERATIONS DASHBOARD                     │
├────────────────────────────────────────────────────────┤
│                                                        │
│  ┌──────────────┐  ┌──────────────┐  ┌────────────┐ │
│  │ Active       │  │ Critical     │  │ Incidents  │ │
│  │ Alerts       │  │ Incidents    │  │ Today      │ │
│  │    47        │  │      3       │  │    12      │ │
│  └──────────────┘  └──────────────┘  └────────────┘ │
│                                                        │
│  ┌─────────────────────────────────────────────────┐ │
│  │    Events Over Time (Last 24 Hours)             │ │
│  │                                                  │ │
│  │    ████                                          │ │
│  │  ██████                                          │ │
│  │ ███████ ██ ███                                   │ │
│  │ ████████████████████                             │ │
│  └─────────────────────────────────────────────────┘ │
│                                                        │
│  ┌──────────────────┐  ┌──────────────────────────┐ │
│  │ Top Source IPs   │  │ Top Triggered Rules      │ │
│  │ 1.2.3.4 (142)   │  │ Brute Force (23)         │ │
│  │ 5.6.7.8 (89)    │  │ Malware Detected (15)    │ │
│  │ 9.10.11.12 (67) │  │ Port Scan (12)           │ │
│  └──────────────────┘  └──────────────────────────┘ │
│                                                        │
│  ┌─────────────────────────────────────────────────┐ │
│  │         Alert Queue (Real-Time)                  │ │
│  ├──────┬──────────┬───────────┬──────────────────┤ │
│  │ Time │ Severity │ Alert     │ Source           │ │
│  ├──────┼──────────┼───────────┼──────────────────┤ │
│  │14:23 │ HIGH     │ Ransomware│ HR-LAPTOP-05     │ │
│  │14:20 │ MEDIUM   │ Port Scan │ 1.2.3.4          │ │
│  │14:18 │ LOW      │ Failed SSH│ web-server-01    │ │
│  └──────┴──────────┴───────────┴──────────────────┘ │
└────────────────────────────────────────────────────────┘
```

**2. Executive Dashboard (Management View)**
```
┌────────────────────────────────────────────────────────┐
│        EXECUTIVE SECURITY DASHBOARD                    │
├────────────────────────────────────────────────────────┤
│                                                        │
│  Security Posture Score: 87/100  ████████████░░        │
│                                                        │
│  ┌──────────────────────────────────────────────────┐ │
│  │  Incidents This Month                             │ │
│  │  ┌────────────┬────────┬──────────┐              │ │
│  │  │ Severity   │ Count  │ Resolved │              │ │
│  │  ├────────────┼────────┼──────────┤              │ │
│  │  │ Critical   │   5    │   4      │              │ │
│  │  │ High       │  23    │  20      │              │ │
│  │  │ Medium     │ 156    │ 142      │              │ │
│  │  │ Low        │ 892    │ 890      │              │ │
│  │  └────────────┴────────┴──────────┘              │ │
│  └──────────────────────────────────────────────────┘ │
│                                                        │
│  ┌────────────────────┐  ┌──────────────────────────┐│
│  │ MTTD (Detect)      │  │ MTTR (Respond)           ││
│  │ 15 minutes         │  │ 2.5 hours                ││
│  │ ↓ 20% from last mo │  │ ↓ 15% from last month    ││
│  └────────────────────┘  └──────────────────────────┘│
│                                                        │
│  ┌─────────────────────────────────────────────────┐ │
│  │  Top Attack Vectors (This Month)                 │ │
│  │  Phishing        ██████████████ 45%              │ │
│  │  Brute Force     █████████ 28%                   │ │
│  │  Malware         ██████ 18%                      │ │
│  │  Other           ███ 9%                          │ │
│  └─────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────┘
```

**3. Threat Hunting Dashboard**
```
┌────────────────────────────────────────────────────────┐
│           THREAT HUNTING DASHBOARD                     │
├────────────────────────────────────────────────────────┤
│                                                        │
│  ┌─────────────────────────────────────────────────┐ │
│  │  Rare Process Executions (Last 7 Days)           │ │
│  │  powershell.exe -enc [base64] (5 occurrences)    │ │
│  │  psexec.exe (2 occurrences)                      │ │
│  │  mimikatz.exe (1 occurrence) ← INVESTIGATE!      │ │
│  └─────────────────────────────────────────────────┘ │
│                                                        │
│  ┌──────────────────────────────────────────────────┐│
│  │  Anomalous Network Connections                    ││
│  │  HR-PC-03 → Russia (Never seen before)           ││
│  │  Marketing-Laptop → TOR Exit Node                ││
│  └──────────────────────────────────────────────────┘│
│                                                        │
│  ┌─────────────────────────────────────────────────┐ │
│  │  MITRE ATT&CK Coverage                           │ │
│  │  Detected Techniques: 47/185 (25%)               │ │
│  │  Most Common: T1078 (Valid Accounts)             │ │
│  └─────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────┘
```

### 11.3 Visualization Types

**1. Time Series (Line Chart)**
```
USE CASE: Show trends over time
EXAMPLE: "Events per hour for last 24 hours"

     Events
     10000 |           ╱╲
      8000 |          ╱  ╲
      6000 |    ╱╲   ╱    ╲
      4000 |   ╱  ╲ ╱      ╲╱╲
      2000 |  ╱    ╲           ╲
         0 |─┴──────┴────────────┴─────
           0  4   8   12  16  20  24 (hours)

INSIGHT: Traffic spike at 8 AM (business hours start)
```

**2. Pie Chart**
```
USE CASE: Show proportions/percentages
EXAMPLE: "Alert distribution by severity"

        ┌────────────┐
        │   Critical │ 5%
        │    High    │ 15%
        │   Medium   │ 30%
        │    Low     │ 50%
        └────────────┘
```

**3. Bar Chart**
```
USE CASE: Compare categories
EXAMPLE: "Top 10 blocked IPs"

IP Address      Count
1.2.3.4        ████████████████ 523
5.6.7.8        ████████████ 412
9.10.11.12     ██████████ 389
13.14.15.16    ████████ 301
...
```

**4. Geographic Map (GeoIP)**
```
USE CASE: Show attack origins
EXAMPLE: "Source countries of attacks"

         WORLD MAP
    ┌─────────────────────┐
    │                     │
    │  USA: 1,234 🔴      │
    │  Russia: 892 🔴     │
    │  China: 745 🔴      │
    │  Brazil: 234 🟡     │
    │                     │
    └─────────────────────┘

🔴 High activity
🟡 Medium activity
⚪ Low activity
```

**5. Heatmap**
```
USE CASE: Show patterns by time/day
EXAMPLE: "Failed logins by hour and day of week"

         Hour →
Day   0  4  8  12 16 20
Mon   ░  ░  █  ██ █  ░
Tue   ░  ░  ██ ██ █  ░
Wed   ░  ░  █  █  █  ░
Thu   ░  ░  ██ ██ ██ ░
Fri   ░  ░  █  █  ░  ░
Sat   ░  ░  ░  ░  ░  ░
Sun   ░  ░  ░  ░  ░  ░

█ = High activity
░ = Low activity

INSIGHT: Most brute force attempts during business hours on weekdays
```

### 11.4 Building Your First Dashboard (Hands-On)

**Splunk Dashboard Example**:

```xml
<dashboard>
  <label>SOC Overview</label>
  <row>
    <panel>
      <title>Total Alerts Today</title>
      <single>
        <search>
          <query>
            index=alerts earliest=@d
            | stats count
          </query>
        </search>
      </single>
    </panel>
  </row>
  <row>
    <panel>
      <title>Alerts Over Time</title>
      <chart>
        <search>
          <query>
            index=alerts earliest=-24h
            | timechart count by severity
          </query>
        </search>
        <option name="charting.chart">line</option>
      </chart>
    </panel>
  </row>
</dashboard>
```

---

## 12. Alerting & Notification Systems

### 12.1 How SIEM Alerting Works

**The Alert Lifecycle**:

```
┌──────────────────────────────────────────────────────┐
│              ALERT GENERATION PROCESS                │
└──────────────────────────────────────────────────────┘

STEP 1: Correlation Rule Evaluates
├─ SIEM continuously runs all enabled rules
├─ Each rule checks conditions against incoming events
└─ Example: "If failed logins > 5 in 5 minutes..."

STEP 2: Condition Met
├─ Rule condition becomes TRUE
├─ Example: 6 failed logins detected in 3 minutes
└─ Alert triggered!

STEP 3: Alert Created
├─ Alert object generated with:
│  ├─ Title: "Brute Force Attack Detected"
│  ├─ Severity: HIGH
│  ├─ Description: "6 failed logins from 1.2.3.4"
│  ├─ Timestamp: 2024-01-31 14:23:45
│  └─ Affected Assets: "DC01"

STEP 4: Enrichment
├─ SIEM adds context:
│  ├─ GeoIP lookup: 1.2.3.4 = Russia
│  ├─ User info: bob = Domain Admin (critical!)
│  └─ Historical: This IP seen before?

STEP 5: Notification Sent
├─ Email to on-call analyst
├─ Slack message to #soc-alerts channel
├─ Ticket created in ServiceNow
├─ Dashboard popup
└─ SMS to manager (if Critical)

STEP 6: Analyst Response
├─ Analyst sees alert
├─ Investigates
└─ Takes action
```

### 12.2 Alert Severity Levels

**Standard Severity Definitions**:

```
┌──────────────────────────────────────────────────────┐
│              ALERT SEVERITY MATRIX                   │
├──────────┬───────────────────────────────────────────┤
│ CRITICAL │ Immediate action required                 │
│  (P0)    │ • Active data breach                      │
│          │ • Ransomware spreading                    │
│          │ • Critical system compromised             │
│          │ → Notify: Everyone (SMS + call)          │
│          │ → SLA: 15 minutes response                │
├──────────┼───────────────────────────────────────────┤
│ HIGH     │ Urgent investigation needed               │
│  (P1)    │ • Confirmed malware detection             │
│          │ • Successful brute force                  │
│          │ • Privilege escalation attempt            │
│          │ → Notify: SOC manager + analyst           │
│          │ → SLA: 1 hour response                    │
├──────────┼───────────────────────────────────────────┤
│ MEDIUM   │ Investigate within shift                  │
│  (P2)    │ • Failed attack attempts                  │
│          │ • Policy violations                       │
│          │ • Suspicious activity                     │
│          │ → Notify: Assigned analyst                │
│          │ → SLA: 4 hours response                   │
├──────────┼───────────────────────────────────────────┤
│ LOW      │ Informational, review as time allows      │
│  (P3)    │ • Successful logins                       │
│          │ • Configuration changes                   │
│          │ • Vulnerability scan results              │
│          │ → Notify: Daily summary email             │
│          │ → SLA: 24 hours response                  │
├──────────┼───────────────────────────────────────────┤
│ INFO     │ Log for compliance, no action needed      │
│  (P4)    │ • Routine system events                   │
│          │ • Health checks                           │
│          │ → Notify: None (logged only)              │
│          │ → SLA: N/A                                │
└──────────┴───────────────────────────────────────────┘
```

### 12.3 Notification Methods

**1. Email Notifications**
```
PROS:
✓ Universal (everyone has email)
✓ Can include detailed info
✓ Audit trail

CONS:
✗ Can be missed/ignored
✗ Email delays
✗ Spam folder risk

BEST FOR: Medium/Low severity, daily summaries

EXAMPLE CONFIG (Splunk):
alert.action = email
alert.to = soc-team@company.com
alert.subject = [SIEM Alert] $name$
alert.message = Alert triggered at $trigger_time$
                Details: $results$
```

**2. Ticketing System Integration**
```
PROS:
✓ Tracks alert lifecycle
✓ Assignment & workflow
✓ SLA tracking
✓ Metrics & reporting

CONS:
✗ Requires integration setup
✗ Can create ticket spam

BEST FOR: All severities (primary method)

EXAMPLE WORKFLOW:
Alert → Creates ServiceNow ticket
      → Assigns to SOC queue
      → Tracks until closure
      → Reports metrics
```

**3. Instant Messaging (Slack/Teams)**
```
PROS:
✓ Instant delivery
✓ Team visibility
✓ Quick collaboration

CONS:
✗ Can be noisy
✗ Might be missed if offline

BEST FOR: High/Critical alerts

EXAMPLE SLACK MESSAGE:
🚨 CRITICAL ALERT 🚨
Type: Ransomware Detected
Host: HR-LAPTOP-05
Time: 2024-01-31 14:23:45
Action Required: Immediate isolation
Assigned to: @oncall-analyst
```

**4. SMS/Phone Calls**
```
PROS:
✓ Reaches analysts anywhere
✓ Hardest to miss

CONS:
✗ Limited information
✗ Costly
✗ Can cause alert fatigue

BEST FOR: Critical only

EXAMPLE:
"CRITICAL: Ransomware detected on HR-LAPTOP-05.
Immediate action required. Call SOC ASAP."
```

**5. Dashboard Popups**
```
PROS:
✓ Immediate visual alert
✓ Analyst already in SIEM

CONS:
✗ Only works if actively monitoring

BEST FOR: Real-time SOC monitoring

EXAMPLE:
[POPUP BANNER]
⚠️ NEW HIGH SEVERITY ALERT
Brute Force Attack - DC01
Click to investigate →
```

### 12.4 Alert Tuning (Reducing False Positives)

**The False Positive Problem**:
```
SCENARIO:
Rule: "Alert on any failed login attempt"

DAY 1:
- 10,000 alerts generated
- 9,950 = False positives (users typing wrong password)
- 50 = Real attacks
- Analysts overwhelmed!

SOLUTION: TUNE THE RULE

IMPROVED RULE:
"Alert on 5+ failed logins from EXTERNAL IP within 5 minutes"

DAY 2:
- 50 alerts generated
- 10 = False positives (legit users on VPN with wrong password)
- 40 = Real attacks
- Much better! (80% accurate)
```

**Tuning Techniques**:

**1. Threshold Adjustment**
```
BEFORE: Alert on ANY port scan
AFTER: Alert on port scan of >20 ports in 1 minute
```

**2. Whitelisting**
```
BEFORE: Alert on ALL file downloads
AFTER: Alert on file downloads EXCEPT from approved-software.company.com
```

**3. Time-Based Filtering**
```
BEFORE: Alert on login from unusual location
AFTER: Alert on login from unusual location OUTSIDE business hours (9-5)
```

**4. Asset-Based Filtering**
```
BEFORE: Alert on ANY admin account usage
AFTER: Alert on admin account usage EXCEPT from jump servers
```

**5. Correlation Improvement**
```
BEFORE: Alert on PowerShell execution
AFTER: Alert on PowerShell with suspicious arguments (-enc, -nop, -w hidden)
       AND from non-IT user
       AND outside business hours
```

---

## 13. SIEM Use Cases

### 13.1 What is a Use Case?

**Definition**: A specific security scenario that SIEM is configured to detect.

Think of use cases as **"detection recipes"** - step-by-step instructions for how SIEM should detect a specific type of attack.

### 13.2 Top 20 Essential SIEM Use Cases

**1. Brute Force Attack Detection**
```
ATTACK: Attacker tries many passwords rapidly

DETECTION LOGIC:
IF failed_login_count > 5
AND timespan < 5 minutes
AND from_same_source_ip
THEN alert "Brute Force Attack"

DATA SOURCES:
- Windows Event ID 4625
- Linux /var/log/auth.log
- VPN logs
- Web application logs

RESPONSE:
- Block source IP at firewall
- Notify user of attack attempt
- Force password reset if successful login follows
```

**2. Ransomware Detection**
```
ATTACK: Malware encrypts files and demands ransom

DETECTION LOGIC:
IF file_extension_change_rate > 100 files/minute
AND extensions include (.encrypted, .locked, .crypto)
OR new files created: (README.txt, HOW_TO_DECRYPT.txt)
THEN alert "Ransomware Activity"

DATA SOURCES:
- EDR/Endpoint logs (file system monitoring)
- Windows Event ID 5145 (file share access)
- Process execution logs

RESPONSE:
- IMMEDIATELY isolate affected system
- Notify incident response team
- Initiate backup restoration plan
```

**3. Data Exfiltration Detection**
```
ATTACK: Attacker steals sensitive data

DETECTION LOGIC:
IF outbound_data_volume > normal_baseline * 3
AND destination = external_IP
AND protocol = (FTP, HTTP POST, DNS tunneling)
AND during off-hours
THEN alert "Possible Data Exfiltration"

DATA SOURCES:
- Firewall logs
- Proxy logs
- DLP (Data Loss Prevention) logs
- NetFlow data

RESPONSE:
- Block connection to external IP
- Investigate source system
- Check what data was accessed
```

**4. Insider Threat - Unusual Access**
```
ATTACK: Insider accessing files they shouldn't

DETECTION LOGIC:
IF user_accesses_file
AND file_classification = "Confidential"
AND user_department != file_owner_department
AND access_time = off_hours
THEN alert "Unusual File Access"

DATA SOURCES:
- File server logs
- SharePoint/OneDrive logs
- Database access logs

RESPONSE:
- Notify user's manager
- Investigate business justification
- Review user's recent activity
```

**5. Privilege Escalation**
```
ATTACK: User gains unauthorized admin rights

DETECTION LOGIC:
IF user_added_to_group
AND group = (Domain Admins, Enterprise Admins)
AND by_user != authorized_admin
THEN alert "Unauthorized Privilege Escalation"

DATA SOURCES:
- Windows Event ID 4728, 4732
- Linux sudo logs
- Active Directory logs

RESPONSE:
- Immediately remove user from group
- Investigate who made the change
- Check for account compromise
```

**6. Credential Theft - Pass-the-Hash**
```
ATTACK: Attacker uses stolen password hashes

DETECTION LOGIC:
IF authentication_method = NTLM
AND from_IP != usual_workstation
AND destination = sensitive_server
THEN alert "Possible Pass-the-Hash Attack"

DATA SOURCES:
- Windows Event ID 4624 (Logon Type 3)
- Authentication logs

RESPONSE:
- Investigate source system
- Force password reset
- Enable Credential Guard
```

**7. Malware Command & Control (C2)**
```
ATTACK: Malware communicating with attacker server

DETECTION LOGIC:
IF outbound_connection
AND destination_IP in threat_intel_feed (known_bad_IPs)
OR DNS_query matches DGA_pattern
OR beaconing_detected (regular intervals)
THEN alert "Malware C2 Communication"

DATA SOURCES:
- Firewall logs
- DNS logs
- Proxy logs
- IDS/IPS alerts

RESPONSE:
- Block C2 domain/IP
- Isolate infected system
- Analyze malware sample
```

**8. Lateral Movement**
```
ATTACK: Attacker spreads across network after initial compromise

DETECTION LOGIC:
IF user_logs_into_multiple_systems
AND systems > 5
AND timespan < 10 minutes
AND logon_type = network (Type 3)
THEN alert "Possible Lateral Movement"

DATA SOURCES:
- Windows Event ID 4624, 4648
- Network authentication logs

RESPONSE:
- Identify compromised account
- Isolate affected systems
- Reset account credentials
```

**9. Web Application Attack**
```
ATTACK: SQL injection, XSS, command injection

DETECTION LOGIC:
IF HTTP_request contains SQL_keywords
AND (UNION, SELECT, DROP, OR 1=1)
OR contains script_tags (<script>, javascript:)
OR contains command_injection (; cat /etc/passwd)
THEN alert "Web Application Attack"

DATA SOURCES:
- Web server logs (Apache, IIS, Nginx)
- WAF (Web Application Firewall) logs

RESPONSE:
- Block source IP
- Patch vulnerable application
- Review for successful exploitation
```

**10. Account Compromise - Impossible Travel**
```
ATTACK: Account used from geographically impossible locations

DETECTION LOGIC:
IF user_login_location_1 = "New York"
AND user_login_location_2 = "China"
AND time_difference < 2 hours
(Impossible to travel that fast!)
THEN alert "Impossible Travel Detected"

DATA SOURCES:
- VPN logs (with GeoIP enrichment)
- Cloud authentication logs (O365, AWS)

RESPONSE:
- Disable account immediately
- Notify user (likely compromised)
- Force MFA reset
```

### 13.3 Building a Use Case (Step-by-Step)

**Let's Build: "Detect Kerberoasting Attack"**

**STEP 1: Understand the Attack**
```
WHAT IS KERBEROASTING?
- Attacker requests Kerberos service tickets
- Extracts ticket hashes
- Cracks offline to get service account passwords

ATTACK INDICATORS:
- Multiple TGS (Ticket Granting Service) requests
- For service accounts
- From single user
- In short timespan
```

**STEP 2: Identify Data Sources**
```
NEEDED LOGS:
- Windows Event ID 4769 (Kerberos Service Ticket Request)

FIELDS TO COLLECT:
- Account Name (who requested)
- Service Name (what service)
- Ticket Encryption Type (weak encryption = suspicious)
- Source IP
- Timestamp
```

**STEP 3: Define Detection Logic**
```
RULE:
IF EventID = 4769
AND Service_Account_Count > 10 (many services requested)
AND Timespan < 1 hour
AND Encryption_Type = 0x17 (RC4 - weak, often used in attack)
THEN Alert "Kerberoasting Detected"
```

**STEP 4: Write SIEM Query**

**Splunk SPL**:
```
index=windows EventCode=4769 Ticket_Encryption_Type=0x17
| bucket _time span=1h
| stats dc(Service_Name) as service_count by _time, Account_Name, src_ip
| where service_count > 10
| eval severity="HIGH"
| table _time, Account_Name, src_ip, service_count, severity
```

**Microsoft Sentinel KQL**:
```
SecurityEvent
| where EventID == 4769
| where TicketEncryptionType == "0x17"
| summarize ServiceCount = dcount(ServiceName) by bin(TimeGenerated, 1h), Account, IpAddress
| where ServiceCount > 10
| extend Severity = "HIGH"
```

**STEP 5: Test the Rule**
```
1. Simulate the attack in lab (using Rubeus or Invoke-Kerberoast)
2. Verify SIEM generates alert
3. Check for false positives
4. Tune threshold if needed
```

**STEP 6: Create Response Playbook**
```
WHEN ALERT TRIGGERS:
1. Identify compromised user account
2. Disable account immediately
3. Check what services were targeted
4. Force password reset for all service accounts
5. Enable MFA
6. Review for successful exploitation
```

---

## 14. SIEM vs SOAR vs XDR

### 14.1 The Evolution of Security Platforms

```
┌──────────────────────────────────────────────────────┐
│         SECURITY TECHNOLOGY EVOLUTION                │
└──────────────────────────────────────────────────────┘

2000s → SIEM
        "Collect and analyze security logs"

2010s → SIEM + SOAR
        "SIEM detects, SOAR automates response"

2020s → XDR
        "Unified detection & response across all layers"
```

### 14.2 SIEM vs SOAR vs XDR Comparison

```
┌──────────────────────────────────────────────────────────────┐
│                  SIEM vs SOAR vs XDR                         │
├────────────┬─────────────────┬──────────────┬───────────────┤
│ Feature    │ SIEM            │ SOAR         │ XDR           │
├────────────┼─────────────────┼──────────────┼───────────────┤
│ Purpose    │ Detect threats  │ Automate     │ Unified       │
│            │                 │ response     │ detection     │
├────────────┼─────────────────┼──────────────┼───────────────┤
│ Data       │ Logs from all   │ Alerts from  │ Telemetry     │
│ Sources    │ sources         │ SIEM/EDR/etc │ from vendor   │
│            │                 │              │ ecosystem     │
├────────────┼─────────────────┼──────────────┼───────────────┤
│ Core       │ Correlation     │ Playbooks    │ AI/ML         │
│ Function   │ Log analysis    │ Orchestration│ Auto response │
├────────────┼─────────────────┼──────────────┼───────────────┤
│ Analyst    │ Manual          │ Semi-        │ Highly        │
│ Work       │ investigation   │ automated    │ automated     │
├────────────┼─────────────────┼──────────────┼───────────────┤
│ Examples   │ Splunk, QRadar  │ Phantom,     │ Cortex XDR,   │
│            │ Sentinel, ELK   │ XSOAR        │ Microsoft 365 │
│            │                 │              │ Defender      │
└────────────┴─────────────────┴──────────────┴───────────────┘
```

### 14.3 SIEM Explained (You Already Know This!)

```
SIEM:
┌────────────────────────────────────────┐
│ 1. Collect logs from everywhere        │
│ 2. Parse and normalize                 │
│ 3. Correlate events                    │
│ 4. Generate alerts                     │
│ 5. Analyst investigates manually       │
│ 6. Analyst responds manually           │
└────────────────────────────────────────┘

STRENGTH: Visibility across entire environment
WEAKNESS: Requires lots of manual work
```

### 14.4 SOAR Explained (Automation Layer)

**What is SOAR?**
Security Orchestration, Automation, and Response

**How It Works**:
```
WITHOUT SOAR:
Alert arrives → Analyst manually:
1. Looks up IP reputation (VirusTotal)
2. Checks user in Active Directory
3. Queries EDR for process details
4. Blocks IP in firewall (if malicious)
5. Creates ticket
6. Sends email to user
TIME: 20-30 minutes per alert

WITH SOAR:
Alert arrives → SOAR automatically:
1. Looks up IP (API call to VirusTotal)
2. Checks user (API to Active Directory)
3. Queries EDR (API call)
4. Blocks IP (API to firewall)
5. Creates ticket (API to ServiceNow)
6. Emails user
TIME: 30 seconds (automated!)
```

**SOAR Workflow Example (Playbook)**:
```
TRIGGER: SIEM alert "Phishing Email Detected"

SOAR PLAYBOOK:
┌──────────────────────────────────────────────┐
│ 1. Enrich Alert                              │
│    ├─ Query VirusTotal for URL reputation   │
│    ├─ Check sender domain age (Whois)       │
│    └─ Scan attachment in sandbox            │
├──────────────────────────────────────────────┤
│ 2. Determine if Malicious                   │
│    IF malicious score > 7/10:               │
│    ├─ Mark as "Confirmed Phishing"          │
│    ELSE:                                     │
│    └─ Escalate to analyst for review        │
├──────────────────────────────────────────────┤
│ 3. Containment (if malicious)               │
│    ├─ Delete email from all mailboxes       │
│    ├─ Block sender domain in email gateway  │
│    └─ Block malicious URL in proxy          │
├──────────────────────────────────────────────┤
│ 4. Communication                             │
│    ├─ Create ticket in ServiceNow           │
│    ├─ Send notification to affected users   │
│    └─ Update SIEM with results              │
├──────────────────────────────────────────────┤
│ 5. Reporting                                 │
│    └─ Log all actions taken                 │
└──────────────────────────────────────────────┘

RESULT: Fully automated response (analyst only involved if needed)
```

**Popular SOAR Platforms**:
- Splunk Phantom
- Palo Alto Cortex XSOAR
- IBM Resilient
- Swimlane

### 14.5 XDR Explained (Next Generation)

**What is XDR?**
Extended Detection and Response

**Key Concept**: Instead of collecting logs from many tools (SIEM approach), XDR is a **unified platform** from one vendor that includes:
- Endpoint protection (EDR)
- Network detection
- Cloud security
- Email security
- Identity security

**How XDR Works**:
```
TRADITIONAL SIEM ARCHITECTURE:
┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐
│Firewall │  │ Endpoint│  │ Email   │  │  Cloud  │
│ (Palo)  │  │(CrowdSt)│  │(Proofpt)│  │  (AWS)  │
└────┬────┘  └────┬────┘  └────┬────┘  └────┬────┘
     │            │             │             │
     └────────────┴─────────────┴─────────────┘
                      ▼
               ┌──────────────┐
               │ SIEM (Splunk)│
               └──────────────┘
Problem: Correlation limited, lots of integration work

XDR ARCHITECTURE:
┌──────────────────────────────────────────────────┐
│           VENDOR XDR PLATFORM                    │
├──────────────────────────────────────────────────┤
│  Endpoints │ Network │ Email │ Cloud │ Identity  │
│     ▲          ▲         ▲       ▲        ▲      │
│     └──────────┴─────────┴───────┴────────┘      │
│              Built-in correlation                │
│              AI/ML-powered detection             │
│              Automated response                  │
└──────────────────────────────────────────────────┘
Benefit: Tight integration, better detection, auto response
```

**XDR Examples**:
- Microsoft 365 Defender (Endpoints, Email, Identity, Cloud Apps)
- Palo Alto Cortex XDR
- CrowdStrike Falcon XDR
- SentinelOne Singularity XDR

**SIEM vs XDR: When to Use What?**
```
USE SIEM WHEN:
✓ You have multi-vendor environment
✓ Need to collect logs from EVERYTHING (legacy systems, OT, etc.)
✓ Compliance requirements (log retention)
✓ Have skilled analysts
✓ Need custom detection logic

USE XDR WHEN:
✓ You want simplified operations
✓ Smaller security team
✓ Willing to use single vendor ecosystem
✓ Want automated response
✓ Cloud-native environment
```

---

## 15. Scaling & Performance

### 15.1 SIEM Performance Challenges

**The Scale Problem**:
```
SMALL COMPANY:
- 100 devices
- 1 GB/day logs
- Simple SIEM setup works fine

ENTERPRISE:
- 10,000+ devices
- 1 TB/day logs (1000x more!)
- Single SIEM server would:
  ├─ Take hours to search
  ├─ Run out of storage in days
  └─ Crash under load

SOLUTION: Distributed architecture
```

### 15.2 Scaling Strategies

**1. Horizontal Scaling (Add More Servers)**
```
SINGLE SERVER:
┌──────────────┐
│  SIEM Server │
│  (1 TB/day)  │
│  Overloaded! │
└──────────────┘

DISTRIBUTED (10 Indexers):
┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐
│Idx1│ │Idx2│ │Idx3│ │Idx4│ │Idx5│
│100G│ │100G│ │100G│ │100G│ │100G│
└────┘ └────┘ └────┘ └────┘ └────┘
┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐
│Idx6│ │Idx7│ │Idx8│ │Idx9│ │Id10│
│100G│ │100G│ │100G│ │100G│ │100G│
└────┘ └────┘ └────┘ └────┘ └────┘

Each handles 100 GB/day
Searches run in parallel → 10x faster!
```

**2. Data Tiering (Hot/Warm/Cold)**
```
RETENTION STRATEGY:
Last 7 days    → Hot (SSD) - Fast searches
8-90 days      → Warm (HDD) - Slower searches
91-365 days    → Cold (Cloud) - Slow searches, cheap storage
365+ days      → Archive/Delete

COST SAVINGS:
All Hot:  $50,000/month storage
Tiered:   $10,000/month storage
Savings:  $40,000/month (80% reduction!)
```

**3. Index Optimization**
```
WITHOUT OPTIMIZATION:
Search for "failed login" in 1 TB of data
→ Scans ALL 1 TB
→ Takes 20 minutes

WITH OPTIMIZATION:
- Index only important fields (user, src_ip, action)
- Bloom filters for fast lookups
- Compressed storage
→ Search takes 5 seconds
```

**4. Data Sampling**
```
FOR VERY HIGH VOLUME (10+ TB/day):

Option A: Sample (collect 10% of data)
- Cheaper
- Faster
- But might miss attacks (not recommended for security!)

Option B: Summarize (aggregate before storing)
- Instead of every event, store:
  "User bob: 1000 successful logins today"
- Good for metrics, bad for forensics

BEST PRACTICE: Collect everything, tier aggressively
```

### 15.3 Performance Tuning

**Search Optimization**:
```
SLOW SEARCH:
index=* user=bob
(Searches ALL indexes - slow!)

FAST SEARCH:
index=windows sourcetype=WinEventLog:Security user=bob earliest=-24h
(Specific index, time range - 100x faster!)

TIPS:
✓ Always specify index
✓ Use time ranges (earliest/latest)
✓ Filter early in search pipeline
✓ Avoid wildcards at start of search (*bob*)
✗ Don't use: index=* *
```

**Forwarder Optimization**:
```
ISSUE: Forwarders consuming too much CPU/Network

SOLUTIONS:
1. Compression: Enable on forwarders (saves network)
2. Batching: Send logs in batches, not one-by-one
3. Filtering: Don't send useless logs (reduces volume by 50%+)
   Example: Filter out "INFO" level logs, only send WARNING+
```

---

## 16. Common SIEM Architectures

### 16.1 Architecture Pattern 1: All-in-One (Small Business)

```
┌────────────────────────────────────────────┐
│         ALL-IN-ONE SIEM                    │
│                                            │
│  ┌──────────────────────────────────────┐ │
│  │  Single Server (16 GB RAM, 1 TB SSD) │ │
│  │                                       │ │
│  │  ✓ Collection                        │ │
│  │  ✓ Parsing                           │ │
│  │  ✓ Indexing                          │ │
│  │  ✓ Search                            │ │
│  │  ✓ Correlation                       │ │
│  │  ✓ Web UI                            │ │
│  └──────────────────────────────────────┘ │
└────────────────────────────────────────────┘

USE CASE:
- <500 employees
- <10 GB/day logs
- Single office
- Budget: $0-$10k

EXAMPLE:
- Splunk Free on VM
- ELK stack on single server
```

### 16.2 Architecture Pattern 2: Distributed (Mid-Size Enterprise)

```
┌──────────────────────────────────────────────────┐
│      DISTRIBUTED SIEM ARCHITECTURE               │
└──────────────────────────────────────────────────┘

[Log Sources]
├─ Firewalls
├─ Endpoints (1000+)
├─ Servers
└─ Cloud
     │
     ▼
[Forwarders/Collectors]
├─ Universal Forwarders on each system
└─ Load balanced across indexers
     │
     ▼
[Indexer Cluster (3-5 servers)]
┌────────┐ ┌────────┐ ┌────────┐
│Index 1 │ │Index 2 │ │Index 3 │
│ 200GB  │ │ 200GB  │ │ 200GB  │
└────────┘ └────────┘ └────────┘
     │
     ▼
[Search Head Cluster (2-3 servers)]
┌──────────┐ ┌──────────┐
│ Search 1 │ │ Search 2 │
│ (Primary)│ │ (Backup) │
└──────────┘ └──────────┘
     │
     ▼
[Analysts access via web UI]

USE CASE:
- 500-5000 employees
- 50-500 GB/day
- Multiple locations
- Budget: $50k-$500k
```

### 16.3 Architecture Pattern 3: Global Enterprise

```
┌──────────────────────────────────────────────────┐
│      GLOBAL DISTRIBUTED ARCHITECTURE             │
└──────────────────────────────────────────────────┘

        GLOBAL SEARCH HEAD
        (Centralized view)
               │
      ┌────────┼────────┐
      │        │        │
   [US DC]  [EU DC] [APAC DC]
      │        │        │
   Indexers Indexers Indexers
   Cluster  Cluster  Cluster
      │        │        │
   Regional Regional Regional
   Sources  Sources  Sources

FEATURES:
- Geographic distribution
- Data sovereignty compliance (EU data stays in EU)
- Disaster recovery
- 24/7 global SOC coverage

USE CASE:
- 10,000+ employees
- Multi-national
- 1+ TB/day
- Budget: $1M+
```

---

## 17. Hands-On: Understanding SIEM Components

### 17.1 Exercise 1: Parse a Raw Log (Manual)

**Raw Firewall Log**:
```
<134>Jan 31 14:23:45 firewall01 kernel: [UFW BLOCK] IN=eth0 OUT= MAC=00:11:22:33:44:55:66:77:88:99:aa:bb:cc:dd SRC=203.0.113.45 DST=10.0.1.50 LEN=60 TOS=0x00 PREC=0x00 TTL=52 ID=12345 DF PROTO=TCP SPT=44821 DPT=22 WINDOW=29200 RES=0x00 SYN URGP=0
```

**Your Task**: Extract these fields manually
- Timestamp: ?
- Source IP: ?
- Destination IP: ?
- Source Port: ?
- Destination Port: ?
- Protocol: ?
- Action: ?

**Answer**:
```
timestamp: Jan 31 14:23:45
src_ip: 203.0.113.45
dest_ip: 10.0.1.50
src_port: 44821
dest_port: 22 (SSH)
protocol: TCP
action: BLOCK
interface: eth0
```

**What Attack Is This?**
→ Attempted SSH brute force (port 22 from external IP, blocked by firewall)

### 17.2 Exercise 2: Build a Simple Correlation Rule

**Scenario**: Detect when a user account is compromised

**Requirements**:
- Alert when user logs in from unusual location
- AND accesses sensitive files
- Within 1 hour

**Your Rule (Pseudo-code)**:
```
RULE: Compromised Account Detection

STEP 1: Detect unusual location login
query = index=auth user=$USER$ src_country != usual_country
save as event_A

STEP 2: Detect sensitive file access
query = index=fileserver user=$USER$ file_classification=confidential
save as event_B

STEP 3: Correlate
IF event_A exists
AND event_B exists
AND time_difference < 1 hour
THEN create_alert("Possible Account Compromise", severity=HIGH)
```

### 17.3 Exercise 3: Calculate SIEM Sizing

**Your Company**:
- 1000 employees
- 500 Windows endpoints
- 100 Linux servers
- 10 firewalls
- 5 web servers

**Estimate Daily Log Volume**:
```
CALCULATION:
Windows endpoints: 500 × 100 MB/day = 50 GB
Linux servers: 100 × 50 MB/day = 5 GB
Firewalls: 10 × 500 MB/day = 5 GB
Web servers: 5 × 1 GB/day = 5 GB

TOTAL: 65 GB/day

Retention (1 year):
65 GB × 365 days = 23.7 TB

With compression (3:1):
23.7 TB / 3 = ~8 TB storage needed

RECOMMENDATION:
- 10 TB storage (with buffer)
- 2-3 indexer servers
- 1 search head
- Budget: ~$100k for 1st year
```

---

## 18. Interview Questions

### 18.1 Beginner Level

**Q1: What does SIEM stand for?**
```
A: Security Information and Event Management
```

**Q2: What is log normalization and why is it important?**
```
A: Converting logs from different formats into a common format.
   Important because it allows correlation across different systems.
   Example: Firewall A calls it "src_ip", Firewall B calls it "source"
   → Normalization makes both "src_ip" so you can search across both.
```

**Q3: What is the difference between correlation and aggregation?**
```
A:
AGGREGATION: Combining similar events
  Example: 1000 failed logins → count = 1000

CORRELATION: Linking different types of events
  Example: Failed login + successful login + file access
           → Detects compromised account
```

**Q4: Name 3 methods of log collection.**
```
A:
1. Agent-based (Splunk Universal Forwarder)
2. Agentless (Syslog)
3. API-based (cloud connector)
```

### 18.2 Intermediate Level

**Q5: Explain the hot/warm/cold data model.**
```
A: Tiered storage based on data age and access frequency:
- HOT: Recent data (0-7 days), fast SSD, frequently searched
- WARM: Older data (8-90 days), slower HDD, occasional searches
- COLD: Archive (90+ days), cheap storage, rare searches

Benefits: Reduces storage costs by 70-80% while maintaining searchability.
```

**Q6: How would you reduce false positives in SIEM?**
```
A: Five main techniques:
1. Threshold tuning (e.g., 5+ failed logins instead of 1)
2. Whitelisting (exclude known-good IPs/users)
3. Time-based filtering (alert only outside business hours)
4. Asset-based filtering (different rules for different systems)
5. Enrichment (add context to improve accuracy)
```

**Q7: What is the Common Information Model (CIM)?**
```
A: A standard schema for normalizing security data.
   Defines common field names (src_ip, dest_ip, user, action)
   so searches work across all data sources regardless of original format.
   
   Example: Both Windows and Linux authentication logs map to same fields,
   enabling a single search: index=* action=failure user=bob
```

### 18.3 Advanced Level

**Q8: Design a SIEM architecture for a company with 5000 employees generating 500 GB/day of logs.**
```
A:
ARCHITECTURE:
- 5 Indexers (100 GB/day each)
- 3 Search Heads (load balanced)
- 1 Deployment server
- 1 License master

STORAGE:
- Hot: 7 days × 500 GB = 3.5 TB (SSD)
- Warm: 83 days × 500 GB = 41.5 TB (HDD)
- Cold: 275 days × 500 GB / 3 (compressed) = 45 TB (Cloud/NAS)

REDUNDANCY:
- Replication factor: 2 (data stored on 2 indexers)
- Search head clustering (HA)

ESTIMATED COST: $200k-$300k first year
```

**Q9: How does a correlation rule detect a multi-stage attack?**
```
A: Using stateful correlation with time windows:

Example: Detecting full attack chain
1. Port scan detected (Stage 1)
2. Save attacker IP + timestamp
3. Watch for SSH brute force from same IP (Stage 2) within 1 hour
4. If successful login follows (Stage 3), correlate all stages
5. Alert: "Multi-stage attack detected - Port scan → Brute force → Access"

Implementation uses:
- Transaction commands (group related events)
- Subsearches (query results feed into another query)
- Join operations (correlate different event types)
```

**Q10: Explain SIEM vs XDR. When would you choose each?**
```
A:
SIEM:
- Best for: Multi-vendor environments, compliance, custom detections
- Flexibility: High (collect anything)
- Complexity: High (requires skilled analysts)
- Cost: Moderate (license by data volume)

XDR:
- Best for: Simplified operations, single vendor ecosystem
- Flexibility: Lower (vendor-specific)
- Complexity: Lower (automated)
- Cost: Higher per endpoint, but less operational cost

CHOOSE SIEM IF:
✓ You have complex, multi-vendor environment
✓ Need custom correlation logic
✓ Compliance requires specific log retention
✓ Have skilled SOC team

CHOOSE XDR IF:
✓ Want to reduce analyst workload
✓ Standardizing on single vendor (e.g., Microsoft)
✓ Limited security staff
✓ Cloud-native architecture
```

---

## Summary: What You've Learned

You now understand:
✅ **How SIEM works** from data collection to alert generation
✅ **SIEM architecture** - components, data flow, scaling
✅ **Log collection methods** - agents, syslog, APIs
✅ **Parsing and normalization** - making sense of different log formats
✅ **Correlation engines** - detecting attacks across multiple events
✅ **Storage strategies** - hot/warm/cold tiers
✅ **Dashboards and visualization** - monitoring security posture
✅ **Use cases** - detecting brute force, ransomware, exfiltration, etc.
✅ **SIEM vs SOAR vs XDR** - understanding the ecosystem
✅ **Performance and scaling** - enterprise SIEM design

**Next Steps**:
1. Read **File 03: Log Management & Data Collection** (deep dive into log sources)
2. Read **File 04: SIEM Platforms Deep Dive** (hands-on Splunk, ELK, Sentinel)
3. Practice in your home lab (set up Splunk Free or ELK)
4. Complete TryHackMe "Splunk 101" room

**You're well on your way to becoming a SIEM expert!** 🚀

---

