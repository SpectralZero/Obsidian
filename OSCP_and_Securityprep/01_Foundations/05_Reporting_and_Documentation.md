# Reporting & Documentation: The Professional Standard

---
title: "Reporting & Documentation"
parent: "[[01_Foundations]]"
tags:
  - foundations
  - reporting
  - documentation
  - professional-skills
  - evidence
created: 2026-01-23
updated: 2026-01-23
---

> **Executive Summary**: In offensive security, **if it isn't documented, it didn't happen.** You are not paid to hack; you are paid to produce a report that describes the hack. The report is the only tangible deliverable of a penetration test. A successful root compromise with a sloppy report is a failure. A failed compromise with a brilliant, insightful report on defense effectiveness is a success. This chapter covers the art of technical note-taking, evidence management, and the construction of high-impact deliverables.

---

## Table of Contents
1. [Learning Objectives](#1-learning-objectives)
2. [The Psychology of Reporting](#2-the-psychology-of-reporting)
3. [Note-Taking Systems & Architectures](#3-note-taking-systems--architectures)
4. [Evidence Management: The Chain of Custody](#4-evidence-management-the-chain-of-custody)
5. [Real-Time Logging Strategies](#5-real-time-logging-strategies)
6. [Structuring the Engagement Repository](#6-structuring-the-engagement-repository)
7. [Writing for the Audience: Executive vs. Technical](#7-writing-for-the-audience-executive-vs-technical)
8. [Vulnerability Scoring (CVSS)](#8-vulnerability-scoring-cvss)
9. [Critical Analysis & Checkpoints](#9-critical-analysis--checkpoints)

---

## 1. Learning Objectives

By the end of this chapter, you will be able to:

- **Architect a Knowledge Base**: Build a resilient, searchable note repository using Obsidian or similar tools.
- **Capture Defensible Evidence**: Take screenshots and logs that survive forensic scrutiny and client skepticism.
- **Log Automatically**: Configure terminal multiplexers to record every keystroke and output for playback.
- **Classify Risk**: Translate technical findings into Common Vulnerability Scoring System (CVSS) scores.
- **Sanitize Data**: Implement procedures to clean and redact sensitive client data from your notes.

---

## 2. The Psychology of Reporting

### 2.1 The Value Proposition
Clients do not pay $20,000 for you to pop a shell. They pay $20,000 to understand **risk**.
- **The Hack**: "I used EternalBlue."
- **The Report**: "An unpatched SMB vulnerability allows attackers to bypass all authentication and seize control of the financial database, risking $5M in regulatory fines."

### 2.2 The "Bus Factor"
Documentation is a safety mechanism.
**Scenario**: You are the lead pentester on a critical infrastructure assessment. On Day 3, you have a medical emergency and cannot continue.
- **Bad Documentation**: The project fails. The client is angry. You lose the contract.
- **Good Documentation**: Your colleague opens your notes, sees exactly where you stopped (IP `10.10.10.5` scanned, `10.10.10.6` pending), has your credentials, and resumes within 1 hour.

### 2.3 Legal Protection
Notes are your alibi.
- **Client Accusation**: "You crashed our production server at 14:00!"
- **Your Notes**: "14:00 - Lunch break. Last command sent at 12:55."
- **Terminal Logs**: Timestamped proof that you were idle.

---

## 3. Note-Taking Systems & Architectures

Do not rely on your brain. It will fail you under pressure.

### 3.1 Tool Selection

#### 1. Obsidian (The Modern Standard)
- **Format**: Markdown (`.md`). Future-proof. Text-based.
- **Pros**:
    - **Linking**: `[[IP_Address]]` links hosts to vulnerabilities.
    - **Search**: Grep-able.
    - **Code Highlighting**: Excellent support for scripts.
    - **Local**: No cloud risk.
- **Cons**: Steep learning curve for plugins.

#### 2. CherryTree (The Classic)
- **Format**: Hierarchical XML (`.ctb`).
- **Pros**: Rigid structure forces organization. Easy screenshot pasting.
- **Cons**: Proprietary format. Corruption risk on large files. Hard to version control (Git).

#### 3. Joplin / Standard Notes
- **Pros**: Encryption support. Sync capabilities.
- **Cons**: Code block formatting can be clunky.

### 3.2 The "Engagement Structure"
For every assessment, create a standardized folder tree.

```text
Project_Name/
├── 00_Admin/
│   ├── Scope_of_Work.pdf
│   ├── RoE.pdf
│   ├── Contact_List.md
│   └── Credentials.md
├── 10_Infrastructure/
│   ├── External_IPs.txt
│   └── Internal_IPs.txt
├── 20_Scans/
│   ├── nmap/
│   ├── nikto/
│   └── nessus/
├── 30_Evidence/
│   ├── screenshots/
│   └── downloads/ (exfiltrated files)
├── 40_Hosts/
│   ├── 192.168.1.10.md
│   └── 192.168.1.50.md
└── 99_Daily_Logs/
    ├── Day1.md
    └── Day2.md
```

### 3.3 The Host Note Template
Do not stare at a blank page. Use a template.

```markdown
# Target: 10.10.10.5 (Hostname)

## 1. Summary
- **OS**: Windows Server 2019
- **Role**: Domain Controller
- **Criticality**: High

## 2. Services (Nmap)
- 53/UDP (DNS)
- 88/TCP (Kerberos)
- 445/TCP (SMB)

## 3. Enumeration Steps
- [x] Nmap Full Scan
- [x] SMB Null Session Check (`enum4linux`)
- [ ] Kerberoasting

## 4. Vulnerabilities
### VULN-01: SMB Signing Disabled
- **Status**: Confirmed
- **Proof**: `nmap --script smb-security-mode ...`

## 5. Loot
- User: `SVC_BACKUP`
- Hash: `$krb5tgs$...`
```

---

## 4. Evidence Management: The Chain of Custody

### 4.1 Screenshot Etiquette
A bad screenshot is worthless.
**The "Uncrop" Rule**: Never crop a screenshot to just the terminal output.
**Capture Context**:
1.  **The Command**: Show *what* you ran.
2.  **The Output**: Show the result.
3.  **The Target**: Show the IP in the URL bar or command arguments.
4.  **The Attacker**: Show your IP (`ip a`) or timestamp (`date`) in the same shot if possible.

**Naming Convention**:
`YYYYMMDD_HHMM_Target_Description.png`
*Example*: `20240315_1430_10.10.10.5_SQLInjection_Proof.png`

### 4.2 Data Handling (Loot)
You will find sensitive data (PII, Credit Cards, Passwords).
1.  **Encryption**: Store loot in an encrypted Veracrypt container.
2.  **Sanitization**:
    - **In Report**: "Found password: `P@ssword1!`" -> "Found password: `P@ssw******`".
    - **In Notes**: Keep raw data (encrypted) for verification, but mark it clearly.
3.  **Destruction**: `shred -u loot.txt` after the project closes.

---

## 5. Real-Time Logging Strategies

"I forgot what command I ran to get that error." -> Amateur.

### 5.1 The `script` Command
Linux has a built-in flight recorder.
```bash
# Start recording
script -a -t 0 logs/day1_session.log

# Do your hacking...

# Stop recording
exit
```
- `-a`: Append (don't overwrite).
- `-t`: Timing data (allows replay).

### 5.2 Terminal Multiplexer Logging (Tmux)
Configure Tmux to log every pane to a file automatically.
**Add to `.tmux.conf`**:
```bash
bind-key H pipe-pane "cat >>$HOME/logs/tmux-#{session_name}-%Y%m%d-%H%M%S.log" \; display-message "Started logging to $HOME/logs/"
```

### 5.3 Shell History
Your `.bash_history` or `.zsh_history` is a backup log.
**Optimization**:
- Increase history size: `HISTSIZE=100000`
- Add timestamps: `HISTTIMEFORMAT="%F %T "`
- Save immediately (not at exit): `PROMPT_COMMAND="history -a"`

---

## 6. Structuring the Engagement Repository

### 6.1 The Master Index
Create a `README.md` at the root of your project folder.
**Contents**:
- **Project Goal**: "Compromise Domain Admin" or "Test Web App X".
- **Key Dates**: Start/End.
- **Current Status**: "3/10 Hosts Compromised".
- **Todo List**: High-level tasks.

### 6.2 Tracking Vulnerabilities
Use a tagging system in Obsidian.
- `#vuln/critical`: Immediate shell/root.
- `#vuln/high`: PrivEsc, SQLi.
- `#vuln/medium`: Info disclosure, weak crypto.
- `#status/verified`: You exploited it.
- `#status/potential`: Nessus says it's there, but you haven't checked.

---

## 7. Writing for the Audience: Executive vs. Technical

You are writing two reports in one document.

### 7.1 The Executive Summary
**Audience**: CEO, CFO, Board Members.
**Goal**: Explain business risk and budget justification.
**Language**: No jargon.
- **Bad**: "We found an unauthenticated RCE in Struts 2 CVE-2017-5638."
- **Good**: "We identified a critical flaw in the payment portal that allows any stranger on the internet to take full control of the server and steal customer credit card data. This issue is being actively exploited in the wild."

### 7.2 The Technical Findings
**Audience**: Sysadmins, Developers, DevOps.
**Goal**: Reproduction and Remediation.
**Structure**:
1.  **Title**: Clear and concise.
2.  **Description**: What is the bug?
3.  **Impact**: What can an attacker do?
4.  **Reproduction Steps**: Copy-pasteable commands.
5.  **Remediation**: Code fixes, patches, or config changes.

---

## 8. Vulnerability Scoring (CVSS)

Don't guess the risk. Calculate it.
**Common Vulnerability Scoring System (CVSS) v3.1**.

### 8.1 Base Metrics (The Vulnerability)
- **Attack Vector (AV)**: Network (High Risk) vs Physical (Low Risk).
- **Attack Complexity (AC)**: Low (Script kiddie) vs High (Needs a race condition).
- **Privileges Required (PR)**: None (High Risk) vs Admin (Low Risk).
- **User Interaction (UI)**: None (Wormable) vs Required (Phishing).
- **Scope (S)**: Changed (Escapes sandbox) vs Unchanged.
- **Confidentiality (C)**: Data loss.
- **Integrity (I)**: Data modification.
- **Availability (A)**: Server crash.

### 8.2 Calculation Example
**Scenario**: Unauthenticated Remote Code Execution on Web Server.
- AV: Network
- AC: Low
- PR: None
- UI: None
- Scope: Changed (Can attack internal network)
- C/I/A: High
- **Score**: **10.0 (Critical)**.

**Scenario**: Stored XSS in Admin Dashboard.
- AV: Network
- AC: Low
- PR: Low (Need login)
- UI: Required (Admin must visit page)
- Scope: Changed (Attacks the user's browser)
- C/I/A: Low/Low/None
- **Score**: **5.4 (Medium)**.

---

## 9. Critical Analysis & Checkpoints

### 9.1 Common Pitfalls
1.  **The "Data Dump" Report**: Pasting 50 pages of Nmap output. **Nobody reads this.** Summarize.
2.  **Missing Remediation**: "Fix it." is not remediation. "Update Apache to v2.4.50 or disable mod_cgi" is remediation.
3.  **Tone Issues**: "Your admins are idiots for leaving this open." **Never insult the client.** "A misconfiguration was identified..."

### 9.2 Checkpoint Questions
1.  **Scenario**: You find a critical vulnerability at 4 PM on Friday. What do you do?
    - *Answer*: Do not wait for the report. Call the Emergency Contact immediately. Write a "Flash Report" (1 page) and email it.
2.  **Evidence**: Why is `whoami` important in a screenshot?
    - *Answer*: It proves the *level* of access. Getting RCE as `www-data` is bad; getting RCE as `root` is catastrophic.
3.  **Ops**: What is the "Clean-Up" phase?
    - *Answer*: Documentation of artifacts created (shells, users, files) and proof of their removal.

### 9.3 Final Thoughts
Your report will exist long after you leave. It might be read by auditors, lawyers, or the FBI. Write it as if your reputation depends on it—because it does.

---
*End of Chapter 05*
