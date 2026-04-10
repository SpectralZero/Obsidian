
# Group Policy Objects (GPO): The Master Control

## 1. Introduction
Imagine a system that allows you to execute a command on 5,000 computers simultaneously, with SYSTEM privileges, just by editing a text file. That system exists. It's called **Group Policy**.

For System Admins, it's a management tool. For Attackers, it is the holy grail of **Lateral Movement** and **Persistence**. If you control a GPO, you control the OUs it links to.

### Beginner Analogy
Think of GPO as "Company Rules".
*   **The CEO (Domain Controller)** writes a rule: "Everyone must have a red wallpaper."
*   **The Employees (Computers)** check the bulletin board (SYSVOL) every 90 minutes.
*   **Compliance**: They see the rule and change their wallpaper.
*   **The Attack**: If you can sneak a fake rule onto the bulletin board ("Everyone must give Bob the master key"), the computers will blindly obey.

---

## 2. Architecture: GPC vs. GPT

A GPO is not a single object. It is split into two parts kept in sync.

### 2.1 Group Policy Container (GPC)
*   **Location**: LDAP (Active Directory Database).
*   **Content**: Version number, status (Enabled/Disabled), list of Client Side Extensions (CSEs) needed.
*   **Path**: `CN=Policies,CN=System,DC=corp,DC=local`.

### 2.2 Group Policy Template (GPT)
*   **Location**: SMB Share (`\\domain.com\SYSVOL\domain.com\Policies\{GUID}`).
*   **Content**: The actual settings (Files).
    *   `Registry.pol`: Registry keys to set.
    *   `Scripts\`: Startup/Shutdown scripts.
    *   `Preferences\`: XML files for drive maps, printers, scheduled tasks.

---

## 3. The Processing Order (LSDOU)

When a computer boots, it applies policies in this strict order. **Last Writer Wins**.

1.  **L**ocal Policy ( `gpedit.msc` on the machine).
2.  **S**ite Policy (Linked to AD Site).
3.  **D**omain Policy (Default Domain Policy).
4.  **O**U Policy (Policies on specific Organizational Units).

**Implication**: An Admin in the "Marketing OU" can override the "Default Domain Policy" unless the Domain Policy is marked **Enforced**.

---

## 4. Offensive GPO Attacks

### 4.1 GPP Password Decryption (MS14-025)
*   **History**: Before 2014, Admins used GPO "Preferences" to create local admin users or map drives. The password was stored in an XML file in SYSVOL.
*   **The Fail**: Microsoft encrypted it with AES-256 but published the **Private Key** on MSDN.
*   **Attack**:
    1.  Browse `\\domain\SYSVOL`.
    2.  Find `Groups.xml`, `Services.xml`, `ScheduledTasks.xml`.
    3.  Look for `cpassword="Az9..."`.
    4.  Decrypt using `gpp-decrypt` (Kali) or PowerSploit.

### 4.2 Permission Abuse (GPO Edit)
If you compromise a user who has **Write** permissions on a GPO.
*   **Weapon**: **SharpGPOAbuse** or manual RSAT.
*   **Technique**: Create a "Scheduled Task" inside the GPO.
    *   **Trigger**: "Immediate".
    *   **Run As**: `NT AUTHORITY\SYSTEM`.
    *   **Action**: `powershell.exe -enc <Base64Payload>`.
*   **Result**: Every computer in the linked OU executes the shell.

### 4.3 Startup/Logon Scripts
*   **Technique**: Copy a script to the GPO's `Scripts\Startup` folder in SYSVOL and edit `scripts.ini`.
*   **Pros**: Runs as SYSTEM (Startup) or User (Logon).
*   **Cons**: Only runs on Reboot or Login (slower than Scheduled Task).

---

## 5. Persistence via GPO

### 5.1 Restricted Groups
*   **Mechanism**: "Restricted Groups" policy enforces group membership.
*   **Attack**: Edit policy to add your "Shadow Admin" user to the built-in "Administrators" group on all workstations.
*   **Result**: You are now Local Admin everywhere. Even if someone removes you, the GPO re-adds you 90 minutes later.

### 5.2 Registry Persistence
*   **Mechanism**: GPO sets a Registry Key.
*   **Attack**: Set `HKLM\Software\Microsoft\Windows\CurrentVersion\Run` to point to your malware.
*   **Stealth**: The registry key reappears if deleted.

---

## 6. Blue Team Defense

### 6.1 ACL Auditing
*   **Action**: regularly audit GPO permissions. Only "Domain Admins" and specific "GPO Admins" should have Write access.
*   **Danger**: "Authenticated Users" with "Write" access is a critical vulnerability.

### 6.2 Monitoring
*   **Event 5136**: Directory Service Change.
*   **Event 4663**: File access in SYSVOL.
*   **Alert**: Alert on any modification to `Default Domain Policy` or `Default Domain Controllers Policy`.

### 6.3 Hardening
*   **AGPM**: Use Advanced Group Policy Management (Microsoft tool) to implement "Check-in/Check-out" and approval workflows.
*   **Tiering**: Ensure workstation GPOs cannot apply to Servers or DCs.

---

## 7. Practical Lab: Analyzing a Malicious GPO

### Scenario: The compromised SYSVOL
**Target**: You have read access to SYSVOL. Identify the backdoor.

**Step 1: Mount SYSVOL**
```powershell
New-PSDrive -Name "S" -PSProvider FileSystem -Root "\\corp.local\SYSVOL\corp.local\Policies"
cd S:
```

**Step 2: Hunt for Scheduled Tasks**
Scheduled Tasks are stored in `Machine\Preferences\ScheduledTasks\ScheduledTasks.xml`.
```powershell
Get-ChildItem -Recurse -Filter "ScheduledTasks.xml" | Select-String "Command"
```
*   *Output*: `<Command>powershell.exe</Command> <Arguments>-enc JAB...</Arguments>`
*   *Analysis*: Decode the Base64. If it's a reverse shell, you found the smoking gun.

**Step 3: Identify the GPO**
The folder name is the GUID (e.g., `{31B2F340...}`).
```powershell
Get-NetGPO -GUID "{31B2F340...}"
```
*   *Output*: "DisplayName: Workstation Updates". (Attackers choose boring names).

---

## 8. Diagrams (Mermaid)

### GPO Processing & Attack Vector

```mermaid
graph TD
    Attacker[Attacker]
    GPO[GPO: 'Office Config']
    SYSVOL[SYSVOL Share]
    PC1[PC: HR-01]
    PC2[PC: IT-01]

    Attacker -->|1. Edit (Write Access)| GPO
    GPO -->|2. Updates Files| SYSVOL
    
    PC1 -->|3. gpupdate (Pull)| SYSVOL
    PC2 -->|3. gpupdate (Pull)| SYSVOL
    
    SYSVOL -->|4. Execute Payload| PC1
    SYSVOL -->|4. Execute Payload| PC2
    
    style Attacker fill:#f96
    style SYSVOL fill:#ff9
```

---

## 9. Critical Analysis

### The "Enforced" Myth
Many people think "Enforced" means "Security Settings".
*   **Reality**: Enforced simply means "This GPO cannot be blocked by lower-level OUs".
*   **Attack**: If you compromise an Enforced GPO, it is *more* powerful because OU admins cannot block your malware.

### Interview Questions
1.  **Q**: What is the "Central Store"?
    -   **A**: A folder in SYSVOL (`PolicyDefinitions`) where ADMX (Template) files are stored so all Admins see the same settings options.
2.  **Q**: How do you force a GPO update remotely?
    -   **A**: `Invoke-GPUpdate -Computer TARGET`. This creates a scheduled task on the target to run `gpupdate /force`.

---

## 10. References
- [[07_Active_Directory_Enumeration]]
- [[10_Windows_Persistence_Mechanisms]]

# End of Document
