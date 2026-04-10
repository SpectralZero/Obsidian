# Interview Preparation: Security Tools & Solutions

> **Purpose**: Master security tools and solutions for cybersecurity interviews. Covers SIEM, SOAR, WAF, Firewalls, Burp Suite, and comprehensive tool categorization with practical usage examples.

**What You'll Master**:
- SIEM - where logs come from, how it works, major platforms
- SOAR - automation and orchestration explained
- WAF - web application firewall protection
- Firewall types - stateful, stateless, next-gen
- Burp Suite - complete web testing workflow
- Comprehensive security tools by category
- Tool selection for different scenarios
- 100+ interview questions with detailed answers

---

## Table of Contents

1. [SIEM - Security Information and Event Management](#1-siem)
2. [SOAR - Security Orchestration, Automation and Response](#2-soar)
3. [WAF - Web Application Firewall](#3-waf)
4. [Firewalls - Types and Configuration](#4-firewalls)
5. [Burp Suite - Web Application Testing](#5-burp-suite)
6. [IDS/IPS - Intrusion Detection and Prevention](#6-idsips)
7. [Reconnaissance Tools](#7-reconnaissance-tools)
8. [Vulnerability Scanning Tools](#8-vulnerability-scanning-tools)
9. [Exploitation Tools](#9-exploitation-tools)
10. [Web Application Testing Tools](#10-web-application-testing-tools)
11. [Password Cracking Tools](#11-password-cracking-tools)
12. [Wireless Security Tools](#12-wireless-security-tools)
13. [Forensics Tools](#13-forensics-tools)
14. [Blue Team Defensive Tools](#14-blue-team-defensive-tools)
15. [Interview Questions & Answers](#15-interview-questions-answers)

---

## 1. SIEM - Security Information and Event Management

### 1.1 What is SIEM?

**Definition**: Centralized platform that collects, aggregates, analyzes, and correlates security logs from across the entire IT infrastructure in real-time.

**Purpose**:
- **Centralized logging** - Single pane of glass for all security events
- **Real-time monitoring** - Detect threats as they happen
- **Correlation** - Connect related events across different systems
- **Alerting** - Notify security team of suspicious activity
- **Compliance** - Meet regulatory requirements (PCI-DSS, HIPAA, SOX)
- **Forensics** - Historical analysis for incident investigation

### 1.2 Where Does SIEM Get Logs From?

```
┌─────────────────────────────────────────────────────────────┐
│              SIEM LOG SOURCES                               │
│                                                             │
│  NETWORK DEVICES:                                           │
│  ├─ Firewalls (allowed/blocked connections)                │
│  ├─ Routers (traffic flows, routing changes)               │
│  ├─ Switches (port activity, VLANs)                        │
│  ├─ VPN concentrators (remote access)                      │
│  ├─ Load balancers (traffic distribution)                  │
│  └─ Wireless access points (connections, auth)             │
│                                                             │
│  SECURITY DEVICES:                                          │
│  ├─ IDS/IPS (intrusion attempts, attacks)                  │
│  ├─ WAF (web attacks, SQL injection, XSS)                  │
│  ├─ Anti-malware (virus detections, quarantines)           │
│  ├─ DLP (data exfiltration attempts)                       │
│  └─ Sandboxes (malware analysis results)                   │
│                                                             │
│  SERVERS:                                                   │
│  ├─ Windows Event Logs (login, process creation)           │
│  ├─ Linux Syslog (auth, system events)                     │
│  ├─ Web servers (access, errors - Apache, IIS, Nginx)      │
│  ├─ Database servers (queries, connections - MySQL, MSSQL) │
│  ├─ Email servers (sent/received, spam - Exchange, Postfix)│
│  ├─ DNS servers (queries, zone transfers)                  │
│  └─ File servers (access, modifications)                   │
│                                                             │
│  ENDPOINTS:                                                 │
│  ├─ Workstations (Windows Event Logs)                      │
│  ├─ EDR agents (CrowdStrike, Carbon Black)                 │
│  ├─ Antivirus (detections, scans)                          │
│  └─ Mobile devices (MDM logs)                              │
│                                                             │
│  APPLICATIONS:                                              │
│  ├─ Active Directory (authentication, group changes)       │
│  ├─ VPN (connections, disconnections)                      │
│  ├─ Proxy servers (web traffic, blocked sites)             │
│  ├─ Cloud services (AWS CloudTrail, Azure logs, O365)      │
│  └─ Custom applications (app-specific logs)                │
│                                                             │
│  CLOUD:                                                     │
│  ├─ AWS CloudTrail (API calls, resource changes)           │
│  ├─ Azure Activity Logs (subscriptions, resources)         │
│  ├─ Google Cloud Audit Logs                                │
│  ├─ Office 365 (logins, email, SharePoint)                 │
│  └─ SaaS applications (Salesforce, Workday)                │
└─────────────────────────────────────────────────────────────┘
```

### 1.3 How SIEM Works

```
┌────────────────────────────────────────────────────────────┐
│                 SIEM WORKFLOW                              │
│                                                            │
│  1. LOG COLLECTION                                         │
│     ├─ Agents installed on systems (Splunk Forwarder)     │
│     ├─ Syslog (UDP/TCP 514)                               │
│     ├─ API integration (cloud services)                   │
│     ├─ SNMP traps                                         │
│     └─ File monitoring (watch log files)                  │
│           ↓                                               │
│  2. LOG AGGREGATION                                        │
│     ├─ Centralized storage (SIEM database)                │
│     ├─ Indexing for fast searching                       │
│     └─ Retention policies (90 days, 1 year, etc.)        │
│           ↓                                               │
│  3. NORMALIZATION                                          │
│     ├─ Convert to common format                          │
│     ├─ Extract fields (timestamp, source IP, user, etc.) │
│     └─ Categorize events (authentication, network, etc.) │
│           ↓                                               │
│  4. CORRELATION                                            │
│     ├─ Apply rules (5 failed logins = alert)             │
│     ├─ Pattern matching (detect attack chains)           │
│     ├─ Anomaly detection (unusual behavior)              │
│     └─ Threat intelligence integration                   │
│           ↓                                               │
│  5. ALERTING                                               │
│     ├─ Generate alerts (critical, high, medium, low)     │
│     ├─ Notify SOC team (email, SMS, ticket)              │
│     ├─ Dashboard visualization                           │
│     └─ Incident creation                                 │
│           ↓                                               │
│  6. RESPONSE                                               │
│     ├─ Investigation (drill down into logs)              │
│     ├─ Containment (block IP, disable account)           │
│     └─ Remediation (patch, remove malware)               │
└────────────────────────────────────────────────────────────┘
```

### 1.4 Log Collection Methods

**Syslog** (Most Common for Linux/Network Devices):
```
Protocol: UDP/TCP port 514 (or TLS 6514 for encrypted)
Format: <Priority>Timestamp Hostname Process[PID]: Message

Example:
<34>Feb 14 10:30:45 webserver sshd[1234]: Failed password for admin from 192.168.1.100

How it works:
1. Device sends syslog to SIEM IP:514
2. SIEM receives and parses
3. Extracts: timestamp, hostname, service, message
4. Stores in database
```

**Windows Event Forwarding (WEF)**:
```
Windows-native log collection
Endpoints forward to collector
Collector sends to SIEM

Event ID examples:
4624 - Successful login
4625 - Failed login
4672 - Admin privileges used
4688 - Process creation
7045 - Service installed
```

**Agent-Based Collection** (Most Reliable):
```
Software agent installed on device
Splunk Universal Forwarder
Elastic Beats
LogRhythm System Monitor

Advantages:
✅ Encrypted communication
✅ Buffering (offline resilience)
✅ File monitoring
✅ Registry monitoring
✅ Process monitoring
```

**API-Based Collection** (Cloud Services):
```
SIEM queries API periodically
AWS CloudTrail via API
Office 365 via Graph API
Azure via REST API

Example: SIEM calls AWS CloudTrail API every 5 minutes
```

**SNMP Traps**:
```
Network devices send alerts
Router CPU > 90% → SNMP trap to SIEM
Switch port down → SNMP trap
```

### 1.5 SIEM Use Cases and Correlation Rules

**Use Case 1: Brute Force Detection**
```
Rule: 5+ failed logins from same IP within 5 minutes

Correlation:
Event 1: Failed login (192.168.1.100) at 10:00:00
Event 2: Failed login (192.168.1.100) at 10:00:15
Event 3: Failed login (192.168.1.100) at 10:00:30
Event 4: Failed login (192.168.1.100) at 10:00:45
Event 5: Failed login (192.168.1.100) at 10:01:00

Alert: Brute force attack detected from 192.168.1.100
Action: Block IP, notify SOC
```

**Use Case 2: Lateral Movement**
```
Rule: Admin login from unusual workstation

Baseline: Admin user "john" normally logs in from WS-001
Alert: Admin user "john" logged in from WS-100

Correlation:
1. Check if john owns WS-100 (no)
2. Check time (3 AM - unusual)
3. Check previous activity (john's account also logged in WS-001 simultaneously)

Alert: Possible credential theft / lateral movement
Investigation: john's credentials compromised?
```

**Use Case 3: Data Exfiltration**
```
Rule: Large data transfer to external IP

Event 1: File server accessed by user "alice"
Event 2: 10 GB uploaded to unknown IP 203.0.113.50
Event 3: Time: 2 AM (outside business hours)

Alert: Potential data exfiltration
Investigation: What files? Authorized transfer? Compromised account?
```

**Use Case 4: Malware Infection Chain**
```
Timeline correlation:

10:30:00 - Email received with attachment (mail server log)
10:30:15 - Attachment opened (endpoint log)
10:30:20 - PowerShell executed with suspicious parameters (EDR)
10:30:25 - Outbound connection to known C&C IP (firewall)
10:30:30 - New scheduled task created (Windows Event 4698)
10:30:35 - Files encrypted (ransomware indicators)

SIEM correlates all 6 events → Complete attack chain visualized
```

### 1.6 Major SIEM Platforms

**Splunk** (Market Leader):
```
Type: Commercial (expensive)
Strengths:
✅ Powerful search (SPL - Search Processing Language)
✅ Excellent visualization
✅ Huge app ecosystem
✅ Scales massively
✅ Great for big data

Search example:
index=windows EventCode=4625 | stats count by src_ip | where count > 5

Pricing: Based on data ingestion (GB/day)
Popular for: Large enterprises, SOC operations
```

**IBM QRadar**:
```
Type: Commercial
Strengths:
✅ Strong correlation engine
✅ Network flow analysis
✅ Pre-built rules and reports
✅ Good for compliance

Features:
- Flow analytics (NetFlow, sFlow)
- Offense management (incident tracking)
- Asset profiling
- Vulnerability integration

Popular for: Financial services, healthcare
```

**ArcSight** (Micro Focus):
```
Type: Commercial
Strengths:
✅ Complex event correlation
✅ Enterprise-scale
✅ Strong compliance support
✅ Active channels (correlation)

Features:
- Real-time correlation
- Case management
- Dashboards and reporting
- Integration with 400+ products

Popular for: Government, large enterprises
```

**ELK Stack** (Elastic, Logstash, Kibana):
```
Type: Open source (paid features available)

Components:
Elasticsearch: Search and analytics engine
Logstash: Log collection and parsing
Kibana: Visualization and dashboards
Beats: Lightweight data shippers

Strengths:
✅ Free and open source
✅ Highly scalable
✅ Flexible (can use for non-security too)
✅ Large community

Example pipeline:
Beats → Logstash (parse) → Elasticsearch (store) → Kibana (visualize)

Popular for: Startups, cost-conscious organizations
```

**Microsoft Sentinel** (Azure Sentinel):
```
Type: Cloud-native SIEM
Strengths:
✅ Cloud-native (Azure)
✅ AI/ML for threat detection
✅ Integrated with Microsoft ecosystem
✅ Pay-as-you-go pricing
✅ No infrastructure to manage

Features:
- KQL (Kusto Query Language)
- Workbooks (dashboards)
- Playbooks (automation via Logic Apps)
- Threat intelligence integration

Popular for: Microsoft shops, cloud-first organizations
```

**AlienVault OSSIM** (Open Source):
```
Type: Open source
Strengths:
✅ Free
✅ All-in-one (SIEM + IDS + vulnerability scanning)
✅ Asset discovery
✅ Good for small/medium businesses

Features:
- Built-in correlation directives
- HIDS (Host IDS)
- Network IDS (Suricata)
- Vulnerability assessment (OpenVAS)

Popular for: SMBs, learning environments
```

### 1.7 SIEM Interview Scenarios

**Q: How would you detect a compromised admin account?**
```
SIEM correlation rules:
1. Login from unusual location (GeoIP check)
2. Login outside business hours
3. Multiple failed logins followed by success (password spraying)
4. Simultaneous logins from different IPs (impossible travel)
5. Privilege escalation attempts
6. Access to sensitive systems not normally accessed
7. Large data downloads

Example alert:
"Admin account 'john' logged in from Russia at 3 AM, 
accessed payroll database (never accessed before), 
downloaded 5 GB of data"

Investigation:
- Contact john (verify legitimate?)
- Check last known good location
- Review all actions taken
- Disable account if unauthorized
- Change password, check other accounts
```

---

## 2. SOAR - Security Orchestration, Automation and Response

### 2.1 What is SOAR?

**Definition**: Platform that automates repetitive security tasks, orchestrates workflows across security tools, and enables faster incident response.

**Three Pillars**:
1. **Orchestration** - Connecting disparate security tools
2. **Automation** - Automated response actions
3. **Response** - Incident management and case tracking

**SIEM vs SOAR**:
```
┌─────────────────────────────────────────────────┐
│          SIEM vs SOAR                           │
│                                                 │
│  SIEM:                                          │
│  ├─ Collects and analyzes logs                 │
│  ├─ Detects threats                            │
│  ├─ Generates alerts                           │
│  └─ Human responds to alerts                   │
│                                                 │
│  SOAR:                                          │
│  ├─ Receives alerts (from SIEM, EDR, etc.)     │
│  ├─ Automates initial investigation            │
│  ├─ Orchestrates response across tools         │
│  └─ Tracks cases and metrics                   │
│                                                 │
│  Together:                                      │
│  SIEM detects → SOAR responds automatically    │
└─────────────────────────────────────────────────┘
```

### 2.2 How SOAR Works

**Example: Phishing Email Response Automation**
```
Traditional Manual Process (30-60 minutes):
1. Analyst receives phishing alert from email gateway
2. Manually checks email headers
3. Extracts URLs and attachments
4. Queries VirusTotal for URL reputation
5. Checks if other users received same email
6. Manually deletes email from all mailboxes
7. Blocks sender domain on email gateway
8. Creates incident ticket
9. Documents findings
10. Notifies affected users

With SOAR Automation (2-3 minutes):
1. Alert received from email gateway
2. SOAR playbook triggers automatically
3. Playbook:
   ├─ Extract indicators (URLs, IPs, domains, attachments)
   ├─ Check VirusTotal API (malicious?)
   ├─ Query SIEM (how many users received it?)
   ├─ API call to email server (delete from all mailboxes)
   ├─ API call to firewall (block sender domain)
   ├─ Create ticket in ServiceNow
   ├─ Send notification to affected users
   └─ Generate incident report
4. Analyst reviews automated actions (human in the loop)
5. Approves or adds additional steps
```

### 2.3 SOAR Use Cases

**Use Case 1: Automated Malware Response**
```
Trigger: EDR detects malware on workstation

Automated Response:
1. Isolate endpoint from network (EDR API)
2. Collect forensic data (memory dump, disk image)
3. Query threat intelligence (malware hash)
4. Check SIEM (other infected systems?)
5. Block malware hash on all endpoints
6. Block C&C IP on firewall
7. Create incident ticket
8. Notify SOC analyst
9. Generate timeline of infection

Human decision: Reimage or remediate?
```

**Use Case 2: Account Compromise**
```
Trigger: SIEM alert - impossible travel
(User logged in USA, then Russia 10 minutes later)

Automated Response:
1. Disable user account (Active Directory API)
2. Revoke all active sessions
3. Force password reset
4. Check recent actions (SIEM query)
5. Check email rules (forwarding to external?)
6. Check file access (data exfiltration?)
7. Notify user via SMS
8. Notify manager
9. Create incident ticket

Human decision: False positive or real compromise?
```

**Use Case 3: Vulnerability Management**
```
Trigger: Vulnerability scanner finds critical CVE

Automated Response:
1. Query asset inventory (affected systems?)
2. Check if exploits exist in the wild
3. Check if systems are internet-facing
4. Prioritize patching (internet-facing critical first)
5. Create patching tickets per system owner
6. Set SLA based on criticality
7. Track patch deployment
8. Re-scan after patch window
9. Generate compliance report

Human decision: Approve patch deployment schedule
```

### 2.4 Major SOAR Platforms

**Palo Alto Cortex XSOAR** (formerly Demisto):
```
Features:
- 500+ integrations
- Playbook marketplace
- War room (collaboration)
- Machine learning for alert classification

Example integration:
SIEM (Splunk) → SOAR → EDR (CrowdStrike) → Firewall (Palo Alto)
```

**Splunk SOAR** (formerly Phantom):
```
Features:
- Visual playbook builder (drag-and-drop)
- Tight Splunk integration
- Community playbooks
- Case management

Example:
Splunk detects threat → Phantom automates response
```

**IBM Resilient**:
```
Features:
- Incident response platform
- Workflow automation
- Integrations with IBM products
- Tabletop exercise simulation
```

**Google Chronicle SOAR** (formerly Siemplify):
```
Features:
- Cloud-native
- Case management
- Timeline visualization
- Integration hub
```

---

## 3. WAF - Web Application Firewall

### 3.1 What is a WAF?

**Definition**: Security solution that monitors, filters, and blocks HTTP/HTTPS traffic to and from web applications.

**Purpose**:
- Protect against web attacks (OWASP Top 10)
- SQL injection prevention
- XSS (Cross-Site Scripting) blocking
- CSRF protection
- Bot detection and mitigation
- Virtual patching (protect unpatched apps)

**Traditional Firewall vs WAF**:
```
┌──────────────────────────────────────────────────┐
│     TRADITIONAL FIREWALL vs WAF                  │
│                                                  │
│  TRADITIONAL FIREWALL (Network Layer 3/4):      │
│  ├─ IP addresses, ports, protocols              │
│  ├─ "Allow TCP port 443"                        │
│  ├─ No understanding of HTTP                    │
│  └─ Can't detect SQL injection                  │
│                                                  │
│  WAF (Application Layer 7):                     │
│  ├─ HTTP/HTTPS traffic only                     │
│  ├─ Inspects HTTP request content               │
│  ├─ Detects: SQL injection, XSS, etc.           │
│  └─ "Block requests containing 'UNION SELECT'"  │
│                                                  │
│  Analogy:                                        │
│  Firewall = Building security (who enters?)     │
│  WAF = Room security (what do they do inside?)  │
└──────────────────────────────────────────────────┘
```

### 3.2 How WAF Works

```
┌────────────────────────────────────────────────┐
│            WAF OPERATION                       │
│                                                │
│  CLIENT                                        │
│    │                                           │
│    │ HTTP Request:                             │
│    │ GET /products?id=1' OR '1'='1             │
│    ▼                                           │
│  ┌─────────┐                                   │
│  │   WAF   │                                   │
│  │         │ 1. Inspect request                │
│  │         │ 2. Pattern matching:               │
│  │         │    - SQL injection? YES (' OR ')  │
│  │         │    - XSS? NO                      │
│  │         │    - Rate limit? OK               │
│  │         │ 3. Decision: BLOCK                │
│  └────┬────┘                                   │
│       │                                        │
│       ├─► Blocked (403 Forbidden)              │
│       │   "Suspicious request detected"        │
│       │                                        │
│  OR  │                                        │
│       ▼                                        │
│  ┌─────────┐                                   │
│  │  WEB    │ Legitimate request forwarded     │
│  │ SERVER  │                                   │
│  └─────────┘                                   │
└────────────────────────────────────────────────┘
```

### 3.3 WAF Detection Methods

**Signature-Based Detection**:
```
Predefined attack patterns (rules)

Example rules:
- Block if request contains: ' OR 1=1 --
- Block if request contains: <script>alert
- Block if request contains: UNION SELECT
- Block if User-Agent: sqlmap

Advantages:
✅ Low false positives
✅ Fast

Disadvantages:
❌ New attacks bypass
❌ Requires constant updates
```

**Anomaly-Based Detection**:
```
Learn normal traffic patterns
Alert on deviations

Example:
- Normal: 50 requests/minute from user
- Anomaly: 5000 requests/minute → Block (DDoS/scraping)

- Normal: User accesses /products, /cart, /checkout
- Anomaly: User accesses /admin → Alert

Advantages:
✅ Detects zero-days
✅ Adapts to application

Disadvantages:
❌ Higher false positives
❌ Requires training period
```

**Hybrid Detection**:
```
Combines signatures + anomalies
Best of both worlds
Most modern WAFs use this
```

### 3.4 Common WAF Rules (OWASP Top 10)

**SQL Injection**:
```
Block patterns:
' OR 1=1 --
'; DROP TABLE users--
UNION SELECT password FROM users
exec(@query)
```

**Cross-Site Scripting (XSS)**:
```
Block patterns:
<script>alert(document.cookie)</script>
<img src=x onerror=alert(1)>
javascript:alert(1)
<iframe src="malicious.com">
```

**Command Injection**:
```
Block patterns:
; cat /etc/passwd
| ls -la
`whoami`
$(rm -rf /)
```

**Path Traversal**:
```
Block patterns:
../../etc/passwd
..\..\windows\system32\config\sam
%2e%2e%2f (URL encoded ../)
```

**File Upload**:
```
Block:
- Executable uploads (.exe, .sh, .php)
- Large files (> 10 MB)
- Suspicious file types (.jsp in upload folder)
```

### 3.5 WAF Deployment Modes

**Inline (Blocking Mode)**:
```
Traffic flows through WAF
WAF can block malicious requests
Most secure but adds latency

Client → WAF → Web Server

If WAF fails: Traffic stops (single point of failure)
```

**Out-of-Band (Monitoring Mode)**:
```
Copy of traffic sent to WAF
WAF alerts but doesn't block
No impact on performance
Good for tuning phase

Client → Web Server
       ↓ (copy)
      WAF (monitors only)
```

**Reverse Proxy**:
```
WAF terminates client connection
Makes new connection to backend
Full SSL inspection
Most control

Client → WAF (SSL terminate) → Web Server
```

### 3.6 Major WAF Solutions

**ModSecurity** (Open Source):
```
Type: Open source WAF engine
Deployment: Apache, Nginx, IIS
Rules: OWASP Core Rule Set (CRS)

Advantages:
✅ Free
✅ Flexible
✅ Well-documented

Use case: Small businesses, learning
```

**Cloudflare WAF**:
```
Type: Cloud-based WAF
Deployment: DNS change (proxy traffic through Cloudflare)
Features:
- DDoS protection
- Bot management
- Rate limiting
- Automatic rule updates

Advantages:
✅ No infrastructure
✅ Global CDN
✅ Easy setup

Use case: Websites, SaaS apps
```

**AWS WAF**:
```
Type: Cloud-native WAF (AWS)
Deployment: ALB, CloudFront, API Gateway
Rules: Managed rule groups + custom

Features:
- IP reputation lists
- Rate-based rules
- Geo-blocking
- Integration with AWS ecosystem

Use case: AWS-hosted applications
```

**Imperva (Incapsula)**:
```
Type: Commercial cloud WAF
Features:
- Advanced bot protection
- API security
- DDoS mitigation
- Attack analytics

Use case: Enterprise applications
```

**F5 BIG-IP ASM**:
```
Type: Hardware/software WAF
Deployment: On-premises or virtual
Features:
- Behavioral analysis
- Automatic policy generation
- DataSafe (client-side encryption)

Use case: Large enterprises, financial services
```

---

## 4. Firewalls - Types and Configuration

### 4.1 What is a Firewall?

**Definition**: Network security device that monitors and controls incoming/outgoing network traffic based on security rules.

**Purpose**:
- Block unauthorized access
- Allow legitimate traffic
- Network segmentation
- Logging and monitoring

### 4.2 Firewall Types

**Packet Filtering Firewall** (Stateless):
```
Inspects: IP addresses, ports, protocols
Decision: Per-packet (no session tracking)

Rule example:
Allow TCP from 192.168.1.0/24 to any port 80

Advantages:
✅ Fast
✅ Simple

Disadvantages:
❌ No context (doesn't track connections)
❌ Vulnerable to spoofing
❌ Basic protection

Use case: Simple networks, routers
```

**Stateful Firewall**:
```
Inspects: Packet + connection state
Tracks: Session table (remembers connections)

Example:
1. Client → Server: SYN (new connection)
   Firewall: Creates state entry
2. Server → Client: SYN-ACK
   Firewall: Updates state (connection established)
3. Client → Server: Data
   Firewall: Checks state table, allows (part of established connection)

Advantages:
✅ Context-aware
✅ Better security than stateless
✅ Prevents many attacks

Use case: Enterprise perimeter, most common today
```

**Next-Generation Firewall (NGFW)**:
```
Stateful firewall + additional features:
- Application awareness (detect specific apps, not just ports)
- Intrusion Prevention System (IPS)
- SSL/TLS inspection (decrypt, inspect, re-encrypt)
- User identity awareness
- Threat intelligence integration

Example:
Block Facebook (application) even if using non-standard ports
Allow Salesforce (application) only for sales team (user-based)

Vendors: Palo Alto, Fortinet, Check Point, Cisco Firepower

Use case: Modern enterprises
```

**Web Application Firewall (WAF)**: See Section 3

**Host-Based Firewall**:
```
Runs on individual computer/server
Windows Firewall, iptables (Linux), pfSense

Protects: Single host
Advantages:
✅ Defense in depth (network + host)
✅ Protects against lateral movement

Example (Windows Firewall):
Block inbound TCP port 445 (SMB) on workstations
```

### 4.3 Firewall Rules

**Basic Rule Structure**:
```
[Action] [Protocol] [Source] [Destination] [Port]

Examples:
ALLOW TCP 192.168.1.0/24 ANY 443
  └─ Allow HTTPS from internal network to anywhere

DENY TCP ANY ANY 23
  └─ Block Telnet from everywhere to everywhere

ALLOW TCP 10.0.0.50 203.0.113.100 3389
  └─ Allow RDP from specific IP to specific server
```

**Default Policies**:
```
Implicit Deny (Recommended):
- Default: Block all
- Explicitly allow needed traffic
- More secure (fail-safe)

Implicit Allow:
- Default: Allow all
- Explicitly block malicious
- Less secure (misconfiguration = breach)
```

**Rule Order**:
```
Rules processed top-to-bottom
First match wins

Example (WRONG):
1. ALLOW TCP ANY ANY ANY ← Allows everything!
2. DENY TCP ANY ANY 445   ← Never reached

Example (CORRECT):
1. DENY TCP ANY ANY 445
2. ALLOW TCP 192.168.1.0/24 ANY 443
3. DENY TCP ANY ANY ANY (implicit deny)
```

### 4.4 Firewall Configuration Example

**Scenario: Small Business Network**
```
Requirements:
- Employees can browse internet (HTTP/HTTPS)
- Block social media (Facebook, Twitter)
- Allow email (SMTP, IMAP)
- RDP access only for IT from specific IPs
- Block all inbound except web server
- Web server on DMZ

Rules (Palo Alto style):
┌────────────────────────────────────────────────────┐
│ OUTBOUND (LAN → Internet)                          │
├────────────────────────────────────────────────────┤
│ 1. ALLOW HTTP/HTTPS from 192.168.1.0/24 to ANY    │
│ 2. DENY Application=Facebook from ANY to ANY      │
│ 3. DENY Application=Twitter from ANY to ANY       │
│ 4. ALLOW SMTP/IMAP from 192.168.1.0/24 to ANY     │
│ 5. DENY ALL (implicit)                            │
├────────────────────────────────────────────────────┤
│ INBOUND (Internet → DMZ/LAN)                       │
├────────────────────────────────────────────────────┤
│ 1. ALLOW TCP ANY to 203.0.113.50:80 (web server)  │
│ 2. ALLOW TCP ANY to 203.0.113.50:443 (web server) │
│ 3. DENY ALL OTHER (implicit)                      │
├────────────────────────────────────────────────────┤
│ MANAGEMENT                                         │
├────────────────────────────────────────────────────┤
│ 1. ALLOW TCP 10.10.10.5 to 192.168.1.0/24:3389    │
│    (IT admin RDP)                                  │
│ 2. DENY ALL OTHER                                 │
└────────────────────────────────────────────────────┘
```

---

## 5. Burp Suite - Web Application Testing

### 5.1 What is Burp Suite?

**Definition**: Integrated platform for web application security testing. Swiss Army knife for penetration testers.

**Editions**:
- **Community** (Free): Basic features
- **Professional** ($399/year): Full features, scanner
- **Enterprise**: Automated scanning, CI/CD integration

### 5.2 Burp Suite Components

**Proxy** (Most Used):
```
Man-in-the-middle between browser and server
Intercepts HTTP/HTTPS requests and responses
Modify requests before sending
Inspect responses before browser sees them

Workflow:
1. Configure browser to use Burp proxy (127.0.0.1:8080)
2. Browse target application
3. Burp intercepts all traffic
4. View/modify requests in real-time
5. Forward or drop requests
```

**Scanner** (Professional only):
```
Automated vulnerability scanner
Crawls application
Tests for vulnerabilities:
- SQL injection
- XSS
- SSRF
- XXE
- Path traversal
- Command injection
```

**Intruder** (Fuzzing/Brute Force):
```
Automated attack tool
Use cases:
- Password brute forcing
- Parameter fuzzing
- Payload testing

Attack types:
1. Sniper: One payload position, iterate through payloads
2. Battering Ram: Multiple positions, same payload
3. Pitchfork: Multiple positions, parallel iteration
4. Cluster Bomb: All permutations

Example: Brute force login
POST /login
username=admin&password=§password§

Payload: password list (rockyou.txt)
Intruder tries: password1, password123, admin, etc.
```

**Repeater**:
```
Manual request modification and re-sending
Workflow:
1. Send request to Repeater
2. Modify request (change parameters, headers)
3. Send request
4. Inspect response
5. Repeat with different payloads

Use case: Test SQL injection manually
GET /product?id=1'
GET /product?id=1' OR '1'='1
GET /product?id=1' UNION SELECT username,password FROM users--
```

**Decoder**:
```
Encode/decode data
Formats:
- Base64
- URL encoding
- HTML entities
- Hex
- ASCII
- Hash (MD5, SHA-1, SHA-256)

Example:
URL encoded: %3Cscript%3Ealert%281%29%3C%2Fscript%3E
Decoded: <script>alert(1)</script>
```

**Comparer**:
```
Compare two requests/responses
Identify differences
Useful for:
- Blind SQL injection (compare response lengths)
- Session token analysis
- Finding hidden parameters
```

**Sequencer**:
```
Analyze randomness of session tokens
Statistical tests
Detect weak token generation

Example: Test if session tokens are predictable
Capture 100 tokens
Sequencer analyzes entropy
Report: "POOR" entropy = predictable tokens
```

**Extender**:
```
Plugin framework
BApp Store: 200+ extensions
Popular extensions:
- Turbo Intruder (faster attacks)
- Autorize (authorization testing)
- Active Scan++ (additional checks)
- Retire.js (JavaScript library vulnerabilities)
```

### 5.3 Common Burp Suite Workflows

**Workflow 1: Finding SQL Injection**
```
1. Browse application (Proxy intercepts traffic)
2. Identify parameter: GET /search?query=test
3. Send to Repeater
4. Test payloads:
   - test'              (error?)
   - test' OR '1'='1    (more results?)
   - test' ORDER BY 5-- (error? find column count)
   - test' UNION SELECT null,null,null,null,null-- (success!)
   - test' UNION SELECT 1,version(),database(),user(),5--
5. Extract data
```

**Workflow 2: Finding XSS**
```
1. Find input: GET /comment?text=hello
2. Send to Repeater
3. Test payloads:
   - <script>alert(1)</script>
   - <img src=x onerror=alert(1)>
   - "><script>alert(1)</script>
4. Check response (is script in HTML?)
5. If filtered, try encoding:
   - %3Cscript%3Ealert%281%29%3C%2Fscript%3E
   - <scr<script>ipt>alert(1)</script>
```

**Workflow 3: Session Token Analysis**
```
1. Capture session cookie
2. Send to Sequencer
3. Collect 100-1000 tokens
4. Sequencer analyzes:
   - Entropy
   - Patterns
   - Predictability
5. Report: "EXCELLENT" or "POOR"
```

---

## 6. IDS/IPS - Intrusion Detection and Prevention

### 6.1 What is IDS/IPS?

**IDS (Intrusion Detection System)**:
```
Monitors network traffic
Detects suspicious activity
Generates alerts
DOES NOT block (passive)

Analogy: Security camera (watches, records, doesn't stop)
```

**IPS (Intrusion Prevention System)**:
```
Monitors network traffic
Detects suspicious activity
BLOCKS malicious traffic (active)

Analogy: Security guard (watches AND stops intruders)
```

### 6.2 Detection Methods

**Signature-Based**:
```
Known attack patterns (like antivirus)
Fast, low false positives
Misses new attacks

Example rule:
alert tcp any any -> any 80 (content: "' OR 1=1"; msg: "SQL Injection Attempt";)
```

**Anomaly-Based**:
```
Baseline normal traffic
Alert on deviations
Catches zero-days
Higher false positives

Example:
Normal: 100 packets/sec
Alert: 10,000 packets/sec (DDoS?)
```

**Behavioral**:
```
Monitors host/network behavior
Detects malicious actions

Example:
Alert: Process "word.exe" creating network connection to 203.0.113.50:4444
(Suspicious: Word doesn't normally make network connections)
```

### 6.3 Major IDS/IPS Solutions

**Snort** (Open Source):
```
Most popular open-source IDS/IPS
Rule-based detection
Community rules available

Example rule:
alert tcp $EXTERNAL_NET any -> $HOME_NET 22 (msg:"SSH Brute Force"; 
threshold: type both, track by_src, count 5, seconds 60;)
```

**Suricata** (Open Source):
```
Modern Snort alternative
Multi-threaded (faster)
HTTP/TLS parsing
Compatible with Snort rules
```

**Palo Alto Networks**:
```
NGFW with IPS
Application-aware
Threat prevention subscriptions
```

**Cisco Firepower**:
```
NGFW with IPS
Snort-based engine
Integration with Cisco ecosystem
```

---

## 7. Reconnaissance Tools

**Nmap** - Network Scanner:
```bash
nmap -sS 192.168.1.0/24        # SYN scan (stealth)
nmap -sV 192.168.1.100         # Service version detection
nmap -O 192.168.1.100          # OS detection
nmap --script vuln target.com  # Vulnerability scripts
```

**Masscan** - Fast Port Scanner:
```bash
masscan -p1-65535 10.0.0.0/8   # Scan entire Class A (fast!)
# Can scan internet in 6 minutes (3 million packets/sec)
```

**theHarvester** - Email/Subdomain Harvesting:
```bash
theHarvester -d example.com -b google  # Find emails, subdomains
```

**Shodan** - Internet Search Engine:
```
Search for exposed devices:
- "default password" country:US
- port:3389 country:CN (RDP)
- port:22 "OpenSSH" product:Ubuntu
```

**Recon-ng** - Web Reconnaissance Framework:
```
Modules for:
- Subdomain enumeration
- Email harvesting
- Port scanning
- Vulnerability searches
```

---

## 8. Vulnerability Scanning Tools

**Nessus** (Commercial):
```
Most popular vulnerability scanner
50,000+ plugins
Compliance scanning (PCI-DSS, HIPAA)
Web application scanning

Example scan:
- OS vulnerabilities
- Missing patches
- Weak passwords
- SSL/TLS issues
- Default credentials
```

**OpenVAS** (Open Source):
```
Free alternative to Nessus
50,000+ tests
Good for SMBs

Scans for:
- CVEs
- Misconfigurations
- Weak encryption
```

**Nikto** - Web Server Scanner:
```bash
nikto -h http://target.com
# Scans for:
# - Outdated server versions
# - Dangerous files (/admin, /backup)
# - Misconfigurations
# - Known vulnerabilities
```

**Qualys** (Cloud-based):
```
SaaS vulnerability management
Continuous monitoring
Asset inventory
Cloud security
```

---

## 9. Exploitation Tools

**Metasploit Framework**:
```
Most popular exploitation framework
2000+ exploits
Payloads, encoders, post-exploitation

Example workflow:
1. search ms17-010 (EternalBlue)
2. use exploit/windows/smb/ms17_010_eternalblue
3. set RHOST 192.168.1.100
4. set PAYLOAD windows/x64/meterpreter/reverse_tcp
5. exploit
6. Meterpreter shell obtained!
```

**SQLmap** - SQL Injection:
```bash
sqlmap -u "http://site.com/page?id=1" --dbs  # Enumerate databases
sqlmap -u "http://site.com/page?id=1" -D users -T accounts --dump
# Automated SQL injection exploitation
```

**BeEF** (Browser Exploitation Framework):
```
Exploits web browsers
Client-side attacks
Hooks browsers
Post-exploitation (keylogging, screenshotting)
```

---

## 10. Web Application Testing Tools

**Burp Suite** (See Section 5)

**OWASP ZAP** (Open Source):
```
Free alternative to Burp
Automated scanner
Proxy
Fuzzer
Spider/crawler

Good for: Beginners, budget-conscious
```

**DirBuster / Gobuster** - Directory Brute Forcing:
```bash
gobuster dir -u http://target.com -w /usr/share/wordlists/dirb/common.txt
# Find hidden directories:
# /admin
# /backup
# /uploads
# /.git
```

**Wfuzz** - Web Fuzzer:
```bash
wfuzz -c -z file,wordlist.txt --hc 404 http://site.com/FUZZ
# Fuzz parameters, directories, files
```

---

## 11. Password Cracking Tools

**John the Ripper**:
```bash
john --wordlist=rockyou.txt hashes.txt  # Dictionary attack
john --incremental hashes.txt           # Brute force
# Cracks: MD5, SHA, NTLM, bcrypt, etc.
```

**Hashcat** (GPU-accelerated):
```bash
hashcat -m 1000 -a 0 ntlm.txt rockyou.txt  # NTLM dictionary
hashcat -m 0 -a 3 md5.txt ?a?a?a?a?a?a     # MD5 brute force (6 chars)
# Fastest password cracker (billions of hashes/sec on GPU)
```

**Hydra** - Network Login Brute Force:
```bash
hydra -l admin -P passwords.txt ssh://192.168.1.100
# Brute force: SSH, FTP, HTTP, RDP, SMB, etc.
```

**CrackMapExec** - Active Directory:
```bash
crackmapexec smb 192.168.1.0/24 -u admin -p passwords.txt
# Test credentials across network
# Enumerate shares, users, groups
```

---

## 12. Wireless Security Tools

**Aircrack-ng Suite**:
```bash
# Wireless packet capture and cracking
airmon-ng start wlan0              # Monitor mode
airodump-ng wlan0mon               # Capture packets
aircrack-ng -w rockyou.txt capture.cap  # Crack WPA2
```

**Reaver** - WPS Cracking:
```bash
reaver -i wlan0mon -b [BSSID]
# Brute force WPS PIN (8 digits = crackable in hours)
```

**Kismet** - Wireless Detector:
```
Passive wireless sniffer
Detects hidden SSIDs
Identifies clients
Wardriving
```

---

## 13. Forensics Tools

**Autopsy / Sleuth Kit**:
```
Digital forensics platform
File system analysis
Deleted file recovery
Timeline analysis
Keyword searching
```

**Volatility** - Memory Forensics:
```bash
volatility -f memory.dmp --profile=Win7SP1x64 pslist  # Running processes
volatility -f memory.dmp --profile=Win7SP1x64 netscan # Network connections
volatility -f memory.dmp --profile=Win7SP1x64 malfind # Find injected code
# Analyze RAM dumps for malware, credentials, artifacts
```

**Wireshark** - Network Forensics:
```
Packet capture and analysis
Protocol decoding
Filter traffic
Follow TCP streams
Extract files from PCAP
```

**FTK Imager** - Disk Imaging:
```
Create forensic images
Bit-for-bit copy
Maintain chain of custody
Hash verification (MD5, SHA-1)
```

---

## 14. Blue Team Defensive Tools

**CrowdStrike Falcon** (EDR):
```
Endpoint Detection and Response
Real-time monitoring
Behavioral analysis
Threat hunting
Cloud-native
```

**Carbon Black** (EDR):
```
Endpoint security
Continuous recording (all endpoint activity)
Threat intelligence
Incident response
```

**Splunk** - SIEM (See Section 1)

**Osquery** - Endpoint Visibility:
```
SQL queries for operating system
Cross-platform (Windows, Linux, macOS)

Example:
SELECT * FROM processes WHERE name LIKE '%malware%';
SELECT * FROM listening_ports WHERE port = 4444;
```

**YARA** - Malware Identification:
```
Pattern matching for malware
Create rules based on strings, hex, PE characteristics

Example rule:
rule EmotetMalware {
    strings:
        $s1 = "C:\\Windows\\SysWOW64\\rundll32.exe"
        $s2 = "POST" wide
    condition:
        all of them
}
```

---

## 15. Interview Questions & Answers

**Q: What is SIEM and where does it get logs from?**
**A**: "SIEM (Security Information and Event Management) is a centralized platform that collects, aggregates, analyzes, and correlates security logs from across the IT infrastructure. 

It gets logs from multiple sources:
- **Network devices**: Firewalls, routers, switches (via Syslog)
- **Security devices**: IDS/IPS, WAF, antivirus
- **Servers**: Windows Event Logs, Linux Syslog, web/database/email servers
- **Endpoints**: Workstations, EDR agents
- **Applications**: Active Directory, VPN, proxy servers
- **Cloud**: AWS CloudTrail, Azure logs, Office 365

Common collection methods include Syslog (UDP/TCP 514), agents (Splunk Forwarder), API integration for cloud services, and Windows Event Forwarding."

---

**Q: What's the difference between SIEM and SOAR?**
**A**: "SIEM focuses on **detection** - collecting logs, correlating events, and generating alerts. SOAR focuses on **response** - automating incident response workflows.

**SIEM**:
- Collects and analyzes logs
- Detects threats through correlation
- Generates alerts
- Human analyst investigates

**SOAR**:
- Receives alerts from SIEM (and other tools)
- Automates initial investigation
- Orchestrates response across multiple tools
- Tracks cases and metrics

Example: SIEM detects phishing email → generates alert → SOAR automatically deletes email from all mailboxes, blocks sender domain, creates ticket, notifies users."

---

**Q: How does a WAF protect web applications?**
**A**: "WAF (Web Application Firewall) operates at Layer 7 (application layer) to inspect HTTP/HTTPS traffic and protect against web attacks.

**Protection methods**:
1. **Signature-based**: Blocks known attack patterns (SQL injection: ' OR 1=1, XSS: <script>)
2. **Anomaly-based**: Learns normal traffic, blocks deviations
3. **Rate limiting**: Prevents DDoS and brute force
4. **Virtual patching**: Protects unpatched vulnerabilities

**Difference from traditional firewall**:
- Traditional firewall: Layer 3/4 (IP, ports) - 'Allow TCP port 443'
- WAF: Layer 7 (HTTP content) - 'Block requests containing SQL injection'

Popular WAFs: ModSecurity (open source), Cloudflare, AWS WAF, Imperva."

---

**Q: What tools do you use for penetration testing?**
**A**: "I categorize tools by testing phase:

**Reconnaissance**:
- Nmap (network scanning)
- theHarvester (email/subdomain enumeration)
- Shodan (internet-wide device search)

**Vulnerability Scanning**:
- Nessus/OpenVAS (infrastructure)
- Nikto (web servers)

**Exploitation**:
- Metasploit Framework (general exploitation)
- SQLmap (SQL injection)
- Burp Suite (web application testing)

**Password Cracking**:
- Hashcat (hash cracking)
- Hydra (network login brute force)
- John the Ripper (password cracking)

**Post-Exploitation**:
- CrackMapExec (Active Directory enumeration)
- Mimikatz (credential extraction)

I always ensure proper authorization before using these tools."

---

**Q: Explain how Burp Suite Intruder works.**
**A**: "Burp Intruder is an automated attack tool used for fuzzing and brute forcing.

**Workflow**:
1. **Capture request** in Proxy
2. **Send to Intruder**
3. **Mark payload positions** with § symbols
   Example: `username=admin&password=§password§`
4. **Select attack type**:
   - **Sniper**: One position, iterate payloads sequentially
   - **Battering Ram**: Multiple positions, same payload
   - **Pitchfork**: Multiple positions, parallel iteration
   - **Cluster Bomb**: All permutations (position1 × position2)
5. **Load payload list** (rockyou.txt, custom wordlist)
6. **Start attack** - Burp sends requests with each payload
7. **Analyze results** - Look for different response lengths, status codes

**Example use case**: Brute force login
- Mark password field as payload position
- Load password list
- Intruder tries each password
- Response with different length = successful login"

---

**Q: What's the difference between IDS and IPS?**
**A**: "**IDS (Intrusion Detection System)**:
- **Passive** - monitors traffic, generates alerts
- **Doesn't block** - only detects and notifies
- **Deployment**: Out-of-band (receives copy of traffic)
- **Analogy**: Security camera (watches, doesn't stop)

**IPS (Intrusion Prevention System)**:
- **Active** - monitors and blocks malicious traffic
- **Inline deployment** - traffic flows through IPS
- **Can drop packets** - prevents attacks in real-time
- **Analogy**: Security guard (watches and stops)

**Trade-off**:
- IDS: No false positive impact (alerts only)
- IPS: More risk (false positive = blocked legitimate traffic)

Most organizations use IPS in detection mode initially, then switch to prevention after tuning."

---

**Q: How would you detect SQL injection using Burp Suite?**
**A**: "Step-by-step SQL injection detection:

1. **Browse application** (Burp Proxy intercepts)
2. **Identify input parameter**: 
   `GET /products?id=5`
3. **Send request to Repeater**
4. **Test for SQL injection**:
   - `id=5'` (error? SQL syntax error?)
   - `id=5' OR '1'='1` (more results? Boolean injection)
   - `id=5' AND 1=1--` (same results?)
   - `id=5' AND 1=2--` (different results? Blind SQL)
5. **Determine number of columns**:
   - `id=5' ORDER BY 1--` (success)
   - `id=5' ORDER BY 5--` (error? 4 columns)
6. **Find injectable columns**:
   - `id=5' UNION SELECT null,null,null,null--`
7. **Extract data**:
   - `id=5' UNION SELECT 1,database(),user(),version()--`
8. **Enumerate tables**:
   - `id=5' UNION SELECT 1,table_name,3,4 FROM information_schema.tables--`
9. **Extract credentials**:
   - `id=5' UNION SELECT 1,username,password,4 FROM users--`

**Automated**: Use Burp Scanner (Professional) for automated detection."

---

**Q: What's the difference between Nessus and Nmap?**
**A**: 
| Aspect | Nmap | Nessus |
|--------|------|--------|
| **Purpose** | Port scanning, service detection | Vulnerability scanning |
| **What it finds** | Open ports, running services, OS | CVEs, misconfigurations, missing patches |
| **Depth** | Surface level | Deep analysis |
| **Output** | Port 22 open (SSH) | CVE-2021-44228 (Log4Shell) CRITICAL |
| **Speed** | Fast | Slower (comprehensive checks) |
| **Cost** | Free, open source | Commercial ($2,390/year) |

**Example**:
- Nmap: "Port 22 open, running OpenSSH 7.4"
- Nessus: "OpenSSH 7.4 has CVE-2018-15473 (user enumeration), CVSS 5.3, patch to 7.9+"

**Use together**: Nmap for discovery → Nessus for vulnerability assessment"

---

**Q: How does a SOAR playbook work?**
**A**: "SOAR playbook is an automated workflow that responds to security incidents.

**Example: Phishing Email Playbook**

**Trigger**: Email gateway flags suspicious email

**Automated Actions**:
1. **Extract IOCs** (Indicators of Compromise):
   - Sender email, subject, URLs, attachments
2. **Enrich with threat intel**:
   - Check VirusTotal (URLs malicious?)
   - Query AlienVault OTX (known campaign?)
3. **Search for other victims**:
   - Query SIEM: How many users received this email?
4. **Containment**:
   - Delete email from all mailboxes (Office 365 API)
   - Block sender domain (email gateway API)
   - Block malicious URLs (proxy API)
5. **Create ticket**:
   - ServiceNow incident with all findings
6. **Notify**:
   - Alert affected users via email
   - Notify SOC team

**Human decision point**: Approve automated actions or modify

**Result**: 30-minute manual process reduced to 2 minutes, consistent response, documented actions."

---

**Q: What security tools would you deploy for a small business?**
**A**: "For a small business (~50 employees, limited budget):

**Network Security**:
- **Firewall**: pfSense (free, NGFW capabilities) or Fortinet (affordable SMB option)
- **IDS**: Suricata (free, open source)

**Endpoint Protection**:
- **Antivirus/EDR**: Windows Defender (built-in) + Microsoft Defender for Endpoint
- **Patch management**: WSUS (Windows) + apt (Linux)

**Email Security**:
- **Email filtering**: SpamAssassin or cloud service (Proofpoint Essentials)
- **DMARC/SPF/DKIM**: Prevent email spoofing (free)

**Monitoring**:
- **SIEM**: ELK Stack (free) or AlienVault OSSIM
- **Network monitoring**: PRTG (free up to 100 sensors)

**Web Security**:
- **WAF**: Cloudflare (free tier for basic DDoS protection)

**Backup**:
- **3-2-1 backups**: Veeam Community Edition (free)

**User Security**:
- **Password manager**: Bitwarden (free/affordable)
- **MFA**: Duo or Microsoft Authenticator

**Total cost**: ~$5,000-$10,000/year (mostly Defender for Endpoint licensing)"

---

## Quick Reference

**SIEM Platforms**:
```
Splunk       → Commercial, powerful, expensive
QRadar       → IBM, correlation engine
ELK Stack    → Open source, flexible
Sentinel     → Cloud-native (Azure)
```

**Firewall Types**:
```
Stateless    → Per-packet, no session tracking
Stateful     → Tracks connections
NGFW         → Application-aware, IPS, SSL inspection
WAF          → Web application layer (HTTP/HTTPS)
```

**Burp Suite Components**:
```
Proxy        → Intercept/modify HTTP
Scanner      → Automated vulnerability detection
Intruder     → Brute force/fuzzing
Repeater     → Manual request modification
```

**Tool Categories**:
```
Recon        → Nmap, Shodan, theHarvester
Vuln Scan    → Nessus, OpenVAS, Nikto
Exploit      → Metasploit, SQLmap
Web Test     → Burp Suite, OWASP ZAP
Password     → Hashcat, John, Hydra
Forensics    → Autopsy, Volatility, Wireshark
```

**Good luck with your interview!** 🎯

You now have comprehensive knowledge of security tools, SIEM, SOAR, WAF, firewalls, Burp Suite, and complete tool categorization!
