# 06 - Incident Detection, Triage & Response: The SOC Analyst Playbook

---
title: "Incident Detection, Triage & Response"
parent: "[[13_SOC_and_SIEM]]"
tags:
  - incident-response
  - triage
  - soc-operations
  - alert-handling
  - playbooks
created: 2026-01-31
updated: 2026-01-31
---

> **Executive Summary**: This is your complete operational playbook for working as a SOC analyst. You'll learn the systematic process for triaging alerts, investigating incidents, determining true vs false positives, escalating appropriately, containing threats, and documenting everything. This chapter covers real-world workflows, decision trees, investigation techniques, common incident types, and response playbooks used in production SOCs. Master this, and you'll confidently handle any alert that comes your way - the core skill every SOC analyst must have.

---

## Table of Contents

1. [The SOC Analyst Workflow](#1-the-soc-analyst-workflow)
2. [Alert Triage Process](#2-alert-triage-process)
3. [True Positive vs False Positive](#3-true-positive-vs-false-positive)
4. [Investigation Methodology](#4-investigation-methodology)
5. [Incident Severity Classification](#5-incident-severity-classification)
6. [Common Incident Types & Responses](#6-common-incident-types--responses)
7. [Incident Response Playbooks](#7-incident-response-playbooks)
8. [Escalation Procedures](#8-escalation-procedures)
9. [Containment Strategies](#9-containment-strategies)
10. [Documentation & Reporting](#10-documentation--reporting)
11. [Communication Best Practices](#11-communication-best-practices)
12. [Tools for Incident Response](#12-tools-for-incident-response)

---

## 1. The SOC Analyst Workflow

### 1.1 A Day in the Life (Realistic View)

```
TIER 1 SOC ANALYST SHIFT (8 hours)

08:00 - Shift Start
├─ Review handover notes from night shift
├─ Check current alert queue (47 alerts pending)
├─ Read morning threat intel brief
└─ Grab coffee ☕

08:30 - Alert Triage Marathon
├─ Alert #1: Failed SSH login (3 min → False Positive)
├─ Alert #2: Port scan detected (5 min → True Positive, escalate)
├─ Alert #3: Malware download blocked (2 min → Document & close)
├─ Alert #4: Brute force attack (15 min → Containment needed!)
├─ ... (repeat 40+ more times)

12:00 - Lunch (if you're lucky!)

13:00 - Continue Alert Processing
├─ High-priority alerts first
├─ Medium alerts as time allows
├─ Low alerts batch processed

15:00 - Incident Investigation
├─ Deep dive on escalated Port Scan incident
├─ Timeline creation
├─ Write detailed report

16:00 - Shift Handover
├─ Document all open incidents
├─ Brief evening shift
└─ Update shift report

REALITY CHECK:
- 80% of time: Triaging false positives
- 15% of time: Real incident investigation
- 5% of time: Training, meetings, breaks
```

### 1.2 The Alert Handling Process (Step-by-Step)

```
┌────────────────────────────────────────────┐
│       ALERT HANDLING WORKFLOW              │
└────────────────────────────────────────────┘

STEP 1: RECEIVE ALERT
├─ Alert appears in SIEM dashboard
├─ Email/SMS notification
├─ Ticket auto-created
└─ TIME: 0 minutes

STEP 2: INITIAL REVIEW (2-3 minutes)
├─ What triggered the alert?
├─ What is the severity?
├─ What asset is affected?
├─ When did it occur?
└─ Quick context check

STEP 3: TRIAGE DECISION (5-10 minutes)
├─ Is this a True Positive or False Positive?
├─ Check:
│  ├─ Alert history (seen before?)
│  ├─ Asset context (what system?)
│  ├─ User context (normal behavior?)
│  ├─ Threat intel (known bad?)
│  └─ Environmental factors
└─ DECISION POINT: TP or FP?

STEP 4A: FALSE POSITIVE
├─ Document reason
├─ Close ticket
├─ Tune rule (if recurring)
└─ TIME: 5-10 minutes total

STEP 4B: TRUE POSITIVE (Incident!)
├─ Escalate severity if needed
├─ Begin investigation
├─ Document findings
├─ Escalate to Tier 2 if complex
└─ TIME: 15 minutes - 2 hours

STEP 5: CONTAINMENT (if needed)
├─ Block malicious IP
├─ Isolate affected system
├─ Reset compromised credentials
├─ Disable malicious accounts
└─ TIME: 5-30 minutes

STEP 6: DOCUMENTATION
├─ Update ticket with all actions
├─ Timeline of events
├─ Evidence collected
├─ Lessons learned
└─ TIME: 10-30 minutes

STEP 7: CLOSURE
├─ Verify threat eliminated
├─ Restore normal operations
├─ Final report
└─ Close ticket
```

---

## 2. Alert Triage Process

### 2.1 The Triage Decision Tree

```
NEW ALERT RECEIVED
     │
     ▼
Is alert in known FP list?
     ├─ YES → Document & Close
     │
     ▼ NO
Does it match known attack pattern?
     ├─ YES → TRUE POSITIVE → Investigate
     │
     ▼ NO
Is behavior normal for this user/asset?
     ├─ YES → Likely FP → Verify & Close
     │
     ▼ NO
Does threat intel confirm malicious?
     ├─ YES → TRUE POSITIVE → Investigate
     │
     ▼ NO
Is there business justification?
     ├─ YES → Authorized activity → Close
     │
     ▼ NO
UNCERTAIN → Escalate to Tier 2
```

### 2.2 Triage Checklist (Use Every Time!)

```
ALERT TRIAGE CHECKLIST:

□ CONTEXT
  □ What is the alert?
  □ What triggered it?
  □ What rule/correlation?
  
□ ASSET
  □ What system is involved?
  □ What's its criticality? (Domain Controller = Critical!)
  □ What's running on it?
  
□ USER
  □ Who is the user?
  □ What's their role?
  □ Normal behavior for them?
  
□ TIME
  □ When did it happen?
  □ During business hours or off-hours?
  □ Is timing suspicious?
  
□ FREQUENCY
  □ First time or recurring?
  □ How often does this occur?
  
□ THREAT INTEL
  □ Is IP/domain/hash known malicious?
  □ Check VirusTotal, AlienVault OTX
  
□ HISTORICAL
  □ Has this happened before?
  □ Previous ticket reference?
  
□ DECISION
  □ True Positive or False Positive?
  □ If TP: What severity?
  □ If FP: Document why
```

---

## 3. True Positive vs False Positive

### 3.1 Determining True vs False Positive

**True Positive (TP)** = Alert correctly identified actual malicious/suspicious activity
**False Positive (FP)** = Alert triggered on benign/authorized activity

**Examples**:

```
SCENARIO 1: Failed Login Alert
Alert: "5 failed logins for user 'admin' from 1.2.3.4"

INVESTIGATION:
- Check: Is 1.2.3.4 internal or external?
  → External IP from Russia
- Check: Is user 'admin' a real account?
  → Yes, it's a privileged account
- Check: Did login eventually succeed?
  → No, all attempts failed

VERDICT: TRUE POSITIVE (brute force attack)
```

```
SCENARIO 2: Failed Login Alert
Alert: "5 failed logins for user 'john' from 192.168.1.105"

INVESTIGATION:
- Check: Is IP internal or external?
  → Internal (corporate network)
- Check: Who is John?
  → Legitimate employee
- Check: Did login eventually succeed?
  → Yes, 6th attempt succeeded
- Check: Time of day?
  → 8:30 AM (start of workday)

VERDICT: FALSE POSITIVE (user forgot password, typical Monday morning)
```

### 3.2 Common False Positive Patterns

```
PATTERN 1: Authorized Scanners
Alert: "Port scan detected from 10.0.0.50"
Reality: Vulnerability scanner (Nessus) doing authorized scan
Action: Whitelist scanner IP

PATTERN 2: Legitimate Admin Activity
Alert: "Admin account used from unusual location"
Reality: IT admin working from home
Action: Add exception for known admin IPs

PATTERN 3: Business Process
Alert: "Large data transfer detected"
Reality: Nightly backup to cloud
Action: Whitelist backup time window

PATTERN 4: Software Updates
Alert: "Suspicious outbound connection"
Reality: Windows Update downloading patches
Action: Whitelist Microsoft Update servers

PATTERN 5: Developer Activity
Alert: "PowerShell with suspicious flags"
Reality: Developer testing automation script
Action: Whitelist dev workstations (with caution!)
```

---

## 4. Investigation Methodology

### 4.1 The 5W1H Investigation Framework

**Every investigation should answer**:

```
WHO?
├─ What user account?
├─ What system/asset?
└─ Source IP address?

WHAT?
├─ What action occurred?
├─ What files/data involved?
└─ What tool/malware used?

WHEN?
├─ Start time?
├─ End time?
└─ Timeline of events?

WHERE?
├─ Which system(s)?
├─ Network location?
└─ Geographic location?

WHY?
├─ What was the intent?
├─ What was compromised?
└─ Business impact?

HOW?
├─ Method of attack?
├─ Entry point?
└─ Lateral movement?
```

### 4.2 Investigation Steps (Detailed)

**STEP 1: Gather Initial Information**
```
FROM ALERT:
- Alert name/ID
- Trigger time
- Affected assets
- Rule/correlation logic

FROM SIEM:
- Raw log events
- Related events (±1 hour)
- Frequency of occurrence
```

**STEP 2: Pivot & Expand**
```
START: User "john" logged in from 1.2.3.4

PIVOT 1: Find all activity from that IP
index=* src_ip="1.2.3.4" | stats count by action

PIVOT 2: Find all activity for that user
index=* user="john" | stats count by src_ip, action

PIVOT 3: Check if others affected
index=* src_ip="1.2.3.4" | stats dc(user) as unique_users
```

**STEP 3: Build Timeline**
```
14:23:45 - Failed login attempt (john from 1.2.3.4)
14:23:46 - Failed login attempt (john from 1.2.3.4)
14:23:47 - Failed login attempt (john from 1.2.3.4)
...
14:25:12 - Successful login! (john from 1.2.3.4)
14:26:30 - File accessed: //server/confidential/passwords.xlsx
14:28:15 - Large file transfer to external IP

PATTERN: Brute force → Access → Exfiltration
```

**STEP 4: Enrich with Threat Intel**
```
Check 1.2.3.4:
□ VirusTotal: 5/90 engines flagged as malicious
□ AbuseIPDB: Reported for brute force (confidence 85%)
□ GeoIP: Located in Russia
□ ASN: Hosting provider (not residential)

VERDICT: Known malicious IP
```

---

## 5. Incident Severity Classification

### 5.1 Severity Matrix

```
┌─────────────┬────────────────────────────────────┐
│ SEVERITY    │ CRITERIA                           │
├─────────────┼────────────────────────────────────┤
│ CRITICAL    │ • Active data breach               │
│ (P0)        │ • Ransomware spreading             │
│             │ • Critical system compromised      │
│             │ • Ongoing data exfiltration        │
│             │ RESPONSE TIME: 15 minutes          │
├─────────────┼────────────────────────────────────┤
│ HIGH        │ • Confirmed malware (contained)    │
│ (P1)        │ • Account compromise               │
│             │ • Successful attack (limited)      │
│             │ RESPONSE TIME: 1 hour              │
├─────────────┼────────────────────────────────────┤
│ MEDIUM      │ • Failed attack attempts           │
│ (P2)        │ • Policy violations                │
│             │ • Suspicious activity              │
│             │ RESPONSE TIME: 4 hours             │
├─────────────┼────────────────────────────────────┤
│ LOW         │ • Informational alerts             │
│ (P3)        │ • Scanning activity (blocked)      │
│             │ • Configuration issues             │
│             │ RESPONSE TIME: 24 hours            │
└─────────────┴────────────────────────────────────┘
```

---

## 6. Common Incident Types & Responses

### 6.1 Incident Type: Phishing Email

**Indicators**:
- Suspicious sender
- Urgent language
- Unexpected attachment
- Suspicious link

**Response Playbook**:
```
1. CONTAIN (5 minutes)
   □ Delete email from all mailboxes
   □ Block sender domain
   □ Block malicious URLs/IPs

2. INVESTIGATE (15 minutes)
   □ How many users received it?
   □ Did anyone click/open?
   □ Was malware downloaded?
   □ Check email logs for delivery

3. REMEDIATE (30 minutes)
   □ Scan affected systems for malware
   □ Reset passwords if credentials entered
   □ Block C2 communication

4. COMMUNICATE
   □ Email affected users
   □ Security awareness reminder
   □ Report to email security team

5. DOCUMENT
   □ IOCs (sender, URLs, hashes)
   □ Timeline
   □ Actions taken
```

---

