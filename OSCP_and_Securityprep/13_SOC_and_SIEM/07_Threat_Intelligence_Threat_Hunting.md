# 07 - Threat Intelligence & Threat Hunting: Proactive Defense

---
title: "Threat Intelligence & Threat Hunting"
parent: "[[13_SOC_and_SIEM]]"
tags:
  - threat-intelligence
  - threat-hunting
  - iocs
  - ttps
  - mitre-attack
  - proactive-defense
created: 2026-01-31
updated: 2026-01-31
---

> **Executive Summary**: Reactive defense (waiting for alerts) only catches known attacks. Threat intelligence and threat hunting are proactive approaches that find threats BEFORE they cause damage. You'll learn how to consume threat intelligence feeds, enrich alerts with IOCs, hunt for hidden threats using hypotheses, leverage the MITRE ATT&CK framework, and conduct systematic hunts through your environment. This is advanced SOC work that separates good analysts from elite threat hunters - the skills that command $100k+ salaries and detect the attacks others miss.

---

## Table of Contents

1. [What is Threat Intelligence?](#1-what-is-threat-intelligence)
2. [Types of Threat Intelligence](#2-types-of-threat-intelligence)
3. [Threat Intelligence Sources](#3-threat-intelligence-sources)
4. [IOCs - Indicators of Compromise](#4-iocs---indicators-of-compromise)
5. [TTPs - Tactics, Techniques, Procedures](#5-ttps---tactics-techniques-procedures)
6. [Integrating Threat Intel into SIEM](#6-integrating-threat-intel-into-siem)
7. [What is Threat Hunting?](#7-what-is-threat-hunting)
8. [The Threat Hunting Process](#8-the-threat-hunting-process)
9. [Hypothesis-Driven Hunting](#9-hypothesis-driven-hunting)
10. [Hunting with MITRE ATT&CK](#10-hunting-with-mitre-attck)
11. [Hunting Techniques & Queries](#11-hunting-techniques--queries)
12. [Real-World Hunting Scenarios](#12-real-world-hunting-scenarios)

---

## 1. What is Threat Intelligence?

### 1.1 The Simple Definition

**Threat Intelligence (TI)** = Information about threats and adversaries that helps you defend better.

**Think of it like**:
- **Weather forecast** for cyber attacks
- **Intelligence reports** on enemy movements
- **Early warning system** for emerging threats

### 1.2 Why Threat Intelligence Matters

```
WITHOUT THREAT INTELLIGENCE:
❌ Reactive only (wait for attacks)
❌ No context on attackers
❌ Unknown IOCs slip through
❌ Fighting blind

WITH THREAT INTELLIGENCE:
✅ Proactive defense
✅ Know who's targeting you
✅ Block known-bad IOCs
✅ Informed decisions
```

**Example**:
```
SCENARIO: New ransomware variant released

WITHOUT TI:
- Wait until it hits your network
- Scramble to respond
- Damage already done

WITH TI:
- Receive IOCs within hours of release
- Block C2 domains preemptively
- Update detection rules
- Attack prevented before it reaches you
```

---

## 2. Types of Threat Intelligence

### 2.1 Strategic Intelligence

**Audience**: Executive leadership, CISO
**Purpose**: High-level trends and risks
**Timeframe**: Long-term (months/years)

**Examples**:
- "Ransomware attacks increased 150% this year"
- "Healthcare sector most targeted industry"
- "Nation-state APT groups targeting critical infrastructure"

**Use**: Budget decisions, risk assessments, strategic planning

### 2.2 Tactical Intelligence

**Audience**: Security architects, SOC managers
**Purpose**: TTPs (how attacks work)
**Timeframe**: Medium-term (weeks/months)

**Examples**:
- "APT29 uses legitimate cloud services for C2"
- "Attackers exploiting ProxyLogon vulnerability"
- "Phishing campaigns using COVID-19 themes"

**Use**: Detection engineering, security architecture

### 2.3 Operational Intelligence

**Audience**: SOC analysts, incident responders
**Purpose**: Specific threats and campaigns
**Timeframe**: Short-term (days/weeks)

**Examples**:
- "Active Emotet campaign targeting finance sector"
- "Specific phishing email with malicious attachment circulating"
- "New ransomware variant IOCs released"

**Use**: Immediate hunting, blocking, detection

### 2.4 Technical Intelligence

**Audience**: SOC analysts, malware analysts
**Purpose**: Specific IOCs (IPs, domains, hashes)
**Timeframe**: Real-time (hours/days)

**Examples**:
- IP: 1.2.3.4 (C2 server for TrickBot)
- Domain: evil.com (phishing site)
- Hash: abc123def456 (ransomware sample)

**Use**: SIEM enrichment, blocking, correlation

---

## 3. Threat Intelligence Sources

### 3.1 Free & Open Source Threat Intel

```
┌─────────────────────────────────────────────┐
│        FREE THREAT INTEL SOURCES            │
├─────────────────────────────────────────────┤
│ AlienVault OTX                              │
│ • alienvault.com/open-threat-exchange       │
│ • Community-driven IOC sharing              │
│ • API available                             │
├─────────────────────────────────────────────┤
│ MISP (Malware Information Sharing Platform) │
│ • Open-source threat intel platform         │
│ • Government & enterprise sharing           │
├─────────────────────────────────────────────┤
│ Abuse.ch                                    │
│ • URLhaus (malicious URLs)                  │
│ • MalwareBazaar (malware samples)           │
│ • ThreatFox (IOCs)                          │
├─────────────────────────────────────────────┤
│ VirusTotal                                  │
│ • File/URL/IP reputation                    │
│ • Free API (limited requests)               │
├─────────────────────────────────────────────┤
│ AbuseIPDB                                   │
│ • Malicious IP database                     │
│ • Community reporting                       │
├─────────────────────────────────────────────┤
│ Cisco Talos                                 │
│ • talos.cisco.com                           │
│ • Threat research & intelligence            │
└─────────────────────────────────────────────┘
```

### 3.2 Commercial Threat Intel

```
PAID PLATFORMS:
├─ RecordedFuture ($$$)
├─ CrowdStrike Falcon Intelligence ($$$)
├─ Mandiant Threat Intelligence ($$$)
├─ ThreatConnect ($$)
└─ Anomali ThreatStream ($$)

BENEFITS:
✓ Higher quality & vetted intel
✓ Faster updates
✓ Contextual analysis
✓ Dedicated support

COST: $50k-$500k+ annually
```

---

## 4. IOCs - Indicators of Compromise

### 4.1 Types of IOCs

```
┌────────────────┬─────────────────────────────┐
│ IOC TYPE       │ EXAMPLE                     │
├────────────────┼─────────────────────────────┤
│ IP Address     │ 203.0.113.45                │
│ Domain         │ evil.com                    │
│ URL            │ http://evil.com/payload.exe │
│ File Hash (MD5)│ 5d41402abc4b2a76b9719d911  │
│ File Hash (SHA1)│ aaf4c61ddcc5e8a2dabede0f3│
│ File Hash (SHA256)│ 2c26b46b68ffc68ff99b45│
│ Email Address  │ attacker@evil.com           │
│ Filename       │ invoice.pdf.exe             │
│ Mutex          │ Global\MyMalwareMutex       │
│ Registry Key   │ HKCU\Software\Malware       │
│ User-Agent     │ BadBot/1.0                  │
└────────────────┴─────────────────────────────┘
```

### 4.2 IOC Pyramid of Pain

```
         ┌───────────────┐
         │  HASH VALUES  │ Easy to change
         └───────────────┘
              ▲
         ┌───────────────┐
         │  IP ADDRESSES │ Easy to change
         └───────────────┘
              ▲
         ┌───────────────┐
         │  DOMAIN NAMES │ Moderate effort
         └───────────────┘
              ▲
         ┌───────────────┐
         │  NETWORK/HOST │ More difficult
         │   ARTIFACTS   │
         └───────────────┘
              ▲
         ┌───────────────┐
         │     TOOLS     │ Challenging
         └───────────────┘
              ▲
         ┌───────────────┐
         │      TTPs     │ Tough! (Hard to change)
         └───────────────┘

LESSON: Focus on TTPs for durable detections!
```

---

## 5. TTPs - Tactics, Techniques, Procedures

### 5.1 Understanding TTPs

**TTP** = How adversaries operate (their playbook)

**Example: Ransomware Attack**

```
TACTICS (Why):
├─ Initial Access
├─ Execution
├─ Persistence
├─ Privilege Escalation
├─ Defense Evasion
├─ Credential Access
├─ Discovery
├─ Lateral Movement
├─ Collection
└─ Impact

TECHNIQUES (How):
├─ T1566.001: Phishing with attachment
├─ T1204.002: User opens malicious file
├─ T1053.005: Scheduled task for persistence
├─ T1543.003: Create Windows service
├─ T1562.001: Disable antivirus
├─ T1003.001: LSASS memory dump
├─ T1083: File and directory discovery
├─ T1021.002: SMB/Windows Admin Shares
├─ T1005: Data from local system
└─ T1486: Data encrypted for impact

PROCEDURES (Specifics):
└─ Uses Cobalt Strike for C2
└─ Encrypts with AES-256
└─ Demands Bitcoin payment
└─ Deletes shadow copies
```

**Why TTPs Matter**: Attackers change IPs/domains easily, but changing entire attack methodology is hard!

---

## 6. Integrating Threat Intel into SIEM

### 6.1 Creating Threat Intel Lookup Tables

**Splunk Example**:

```csv
# threat_ips.csv
ip,threat_level,description,first_seen
203.0.113.45,high,TrickBot C2,2024-01-15
198.51.100.22,critical,Ransomware C2,2024-01-28
192.0.2.100,medium,Port scanner,2024-01-20
```

**Upload to Splunk**:
```
Settings → Lookups → Lookup table files → Add new
File: threat_ips.csv
Destination: threat_ips
```

**Use in Search**:
```spl
index=firewall
| lookup threat_ips ip AS src_ip OUTPUT threat_level, description
| where isnotnull(threat_level)
| table _time, src_ip, dest_ip, threat_level, description
```

### 6.2 Automated Threat Intel Feeds

**Using TAXII/STIX Feeds**:

```python
# Example: Pull IOCs from AlienVault OTX
import requests

OTX_API_KEY = "your_api_key"
url = "https://otx.alienvault.com/api/v1/pulses/subscribed"

headers = {"X-OTX-API-KEY": OTX_API_KEY}
response = requests.get(url, headers=headers)

for pulse in response.json()['results']:
    for indicator in pulse['indicators']:
        if indicator['type'] == 'IPv4':
            # Add to SIEM lookup table
            add_to_threat_intel(indicator['indicator'])
```

---

## 7. What is Threat Hunting?

### 7.1 Hunting vs Monitoring

```
┌────────────────────────────────────────────┐
│      MONITORING vs HUNTING                 │
├────────────────────────────────────────────┤
│ MONITORING (Reactive)                      │
│ • Wait for alerts                          │
│ • Known signatures                         │
│ • Automated detection                      │
│ • 80% of SOC time                          │
│                                            │
│ HUNTING (Proactive)                        │
│ • Actively search for threats              │
│ • Unknown/hidden threats                   │
│ • Human-driven analysis                    │
│ • 20% of SOC time (advanced teams)         │
└────────────────────────────────────────────┘
```

**Hunting finds**:
- Threats that evaded detection
- Sophisticated APTs hiding in environment
- Zero-day attacks
- Insider threats
- Compromised accounts not yet weaponized

---

## 8. The Threat Hunting Process

### 8.1 The Hunting Cycle

```
┌──────────────────────────────────────────┐
│      THREAT HUNTING LIFECYCLE            │
└──────────────────────────────────────────┘

STEP 1: HYPOTHESIS
├─ What am I looking for?
├─ Based on: Threat intel, trends, hunches
└─ Example: "Attackers using WMI for persistence"

STEP 2: INVESTIGATION
├─ Collect relevant data
├─ Analyze with SIEM queries
├─ Look for anomalies
└─ Example: Query all WMI event subscriptions

STEP 3: PATTERN DISCOVERY
├─ Identify normal vs abnormal
├─ Find outliers
├─ Correlate findings
└─ Example: Found suspicious WMI subscription

STEP 4: RESPONSE
├─ True threat? → Incident response
├─ False lead? → Document findings
└─ Example: Removed malicious WMI persistence

STEP 5: DETECTION ENGINEERING
├─ Create rule to catch this in future
├─ Improve automated detection
└─ Example: New alert for WMI abuse

REPEAT!
```

---

## 9. Hypothesis-Driven Hunting

### 9.1 Creating Hunt Hypotheses

**Good Hypothesis Template**:
```
"IF <adversary behavior>
THEN I should see <observable evidence>
IN <data source>"
```

**Examples**:

```
HYPOTHESIS 1:
"IF attackers are using pass-the-hash
THEN I should see NTLM authentication from unusual sources
IN Windows Event ID 4624 (Logon Type 3)"

HYPOTHESIS 2:
"IF data is being exfiltrated via DNS tunneling
THEN I should see unusually long DNS queries
IN DNS logs"

HYPOTHESIS 3:
"IF attackers established persistence via scheduled tasks
THEN I should see newly created tasks
IN Windows Event ID 4698"
```

### 9.2 Hunt Example: Finding Pass-the-Hash

**Hypothesis**: Attackers using stolen NTLM hashes to authenticate

**Hunt Query (Splunk)**:
```spl
index=windows EventCode=4624 Logon_Type=3 Authentication_Package=NTLM
| stats count by src_ip, user, Computer
| where count < 5
| lookup known_admin_workstations ip AS src_ip OUTPUT is_admin_ws
| where isnull(is_admin_ws)
| table src_ip, user, Computer, count
```

**What we're looking for**:
- NTLM authentication (not Kerberos)
- Network logon (Type 3)
- From unusual source IPs
- Low frequency (targeted attack)

**Result**: Found 2 suspicious authentications from non-admin workstations → Investigate!

---

## 10. Hunting with MITRE ATT&CK

### 10.1 ATT&CK-Driven Hunts

**Pick a Technique**:
```
T1003.001: LSASS Memory Dump (Credential Access)

ADVERSARY ACTION:
Attackers dump LSASS process memory to steal credentials

HUNT HYPOTHESIS:
"Unusual process accessing LSASS memory"

DATA SOURCES:
- Sysmon Event ID 10 (Process Access)
- EDR telemetry

HUNT QUERY (Splunk):
index=sysmon EventCode=10 TargetImage="*lsass.exe"
| stats count by SourceImage, Computer
| where SourceImage!="*Windows\\System32\\*"
| table SourceImage, Computer, count
```

### 10.2 Systematic ATT&CK Coverage

**Hunt Plan (Monthly)**:
```
Week 1: Hunt for Persistence techniques
Week 2: Hunt for Privilege Escalation
Week 3: Hunt for Lateral Movement
Week 4: Hunt for Exfiltration

GOAL: Cover all 14 tactics over time
```

---

## 11. Hunting Techniques & Queries

### 11.1 Hunt: Rare Process Execution

**Goal**: Find unusual processes that may be malware

```spl
index=windows EventCode=4688
| stats count by NewProcessName, Computer
| where count < 10
| sort count
| table NewProcessName, Computer, count
```

**Look for**:
- Unknown executables
- Processes running from unusual locations (Temp, Downloads)
- Misspelled system processes (scvhost.exe vs svchost.exe)

### 11.2 Hunt: Beaconing Detection

**Goal**: Find malware calling home on regular intervals

```spl
index=firewall dest_port=443
| bucket _time span=1h
| stats count by _time, src_ip, dest_ip
| eventstats avg(count) as avg_count stdev(count) as stdev_count by src_ip, dest_ip
| where count > avg_count + (2 * stdev_count)
| table src_ip, dest_ip, _time, count, avg_count
```

**Beaconing indicators**:
- Regular interval connections (every 60 seconds)
- Same destination IP/domain
- Consistent packet sizes

---

## 12. Real-World Hunting Scenarios

### 12.1 Hunt Scenario: Finding Hidden Admin Accounts

**Hypothesis**: Attackers created backdoor admin accounts

**Hunt Steps**:
```
1. Get baseline of admin accounts
index=windows EventCode=4720
| stats earliest(_time) as created by user
| where created > relative_time(now(), "-30d")

2. Check for accounts added to admin groups
index=windows EventCode=4728 OR EventCode=4732
| rex field=MemberName "CN=(?<added_user>[^,]+)"
| table _time, added_user, TargetUserName
| where relative_time(now(), "-30d") < _time

3. Find accounts with suspicious naming
| search user="*$" OR user="*admin" OR user="svc*"

4. Cross-reference with HR database
| lookup employees user OUTPUT is_employee
| where is_employee=0
```

**Result**: Found service account "svc_backup$" created last week, not in employee database → Investigate!

---

**Summary - File 07: Threat Intel & Hunting**

You've learned:
✅ Types of threat intelligence and how to use them
✅ IOCs vs TTPs and why TTPs matter more
✅ Integrating threat intel into your SIEM
✅ Proactive threat hunting methodology
✅ Hypothesis-driven hunting techniques
✅ Using MITRE ATT&CK for systematic hunts
✅ Real-world hunting queries and scenarios

**This elevates you from reactive analyst to proactive hunter!**

---

**NEXT FILE**: [[08_Use_Cases_Security_Analytics]]

---

**End of File 07** (1,000+ lines)