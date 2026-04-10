
# Note Taking Strategies: Brain Backup

> **Executive Summary**: You cannot remember everything. In a 5-day assessment, you will run 500 commands and see 5000 lines of output. Effective note-taking is the only way to correlate findings, generate quality reports, and maintain your sanity. This chapter covers structure, tools, and the Zettelkasten method for hackers.

## 1. Learning Objectives
By the end of this chapter, you will be able to:
- **Structure Data**: Organize notes by IP, Domain, and Vulnerability.
- **Link Concepts**: Use backlinking (Obsidian) to connect "Apache 2.4" to "Exploit X".
- **Log Everything**: Automatically capture terminal output.
- **Template**: Create reusable templates for Host Recon and Findings.

## 2. Core Concepts: The Knowledge Graph

### 2.1 Hierarchical vs Networked
- **Folders**: Rigid. Good for project artifacts (Scans, Screenshots).
- **Links**: Flexible. Good for knowledge. [[SQL_Injection]] links to [[Blind_SQLi]].

### 2.2 The Daily Log
Keep a running journal (`daily.md`).
- `09:00`: Started scan on 10.10.10.10.
- `09:15`: Found Port 80 open.
- `09:30`: Tried default creds (admin:admin). Failed.
**Why**: When the client asks "Did you try X?", you can check the timestamp.

## 3. Deep Dive: Obsidian for Hackers

### 3.1 Setup
- **Vault**: Your root folder.
- **Plugins**:
    - *Templater*: Fast note creation.
    - *Paste Image Rename*: keeps attachments organized.
    - *Dataview*: Query your notes (e.g., "List all hosts where OS=Windows").

### 3.2 Templates
**Host Template**:
```markdown
# {{title}}
IP: 
OS: 
Tags: #host

## Ports
- 80 (HTTP)
- 445 (SMB)

## Todo
- [ ] Gobuster port 80
- [ ] CrackMapExec SMB
```

## 4. Red Team Perspective

### 4.1 Collaborative Notes
Use tools like **SysReptor** or **Ghostwriter** for multi-user engagements.
- Conflict resolution is key (git).
- Real-time sync ensures Team Member A knows Member B already scanned the DC.

### 4.2 Evidence Management
- **Naming**: `evidence_ip_vuln.png`.
- **Storage**: Encrypted volume.
- **Scrubbing**: Remove client data from screenshots (redact PII).

## 5. Practical Lab: The Perfect Screenshot

### Scenario: Capturing Evidence
You found a shell.

**Bad Screenshot**: Full desktop, Spotify open, tiny terminal font.
**Good Screenshot**:
1.  Zoom in terminal (`Ctrl + +`).
2.  Run `ip a` (Show target IP).
3.  Run `whoami` (Show privilege).
4.  Run `date` (Show timestamp).
5.  Use `Flameshot` to crop strictly to the terminal and draw a red box around `uid=0(root)`.

## 6. References
- [[09_Reporting_Professional_Skills/01_Executive_Reporting]]
- [[01_Foundations/05_Reporting_and_Documentation]]
