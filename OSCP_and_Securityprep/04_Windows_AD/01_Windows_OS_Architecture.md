
# Windows OS Architecture — Complete Master Reference

> **Purpose:** From absolute beginner to expert — this file covers every layer of Windows architecture: CPU privilege rings, the kernel/user boundary, process internals, PE format, kernel data structures, the boot chain, and how attackers abuse every single layer. Beginner analogies are provided alongside expert-level technical detail.

---

## Table of Contents

1. [Why Windows Architecture Matters for Security](#1-why-it-matters)
2. [CPU Privilege Rings — The Foundation](#2-cpu-rings)
3. [Windows Architecture Layers](#3-architecture-layers)
4. [User Mode vs Kernel Mode — Deep Dive](#4-user-vs-kernel)
5. [The Windows API Call Chain](#5-api-call-chain)
6. [Process Internals — What is a Process Really?](#6-process-internals)
7. [The PE Format — Complete Anatomy](#7-pe-format)
8. [Kernel Data Structures — EPROCESS, KTHREAD, Tokens](#8-kernel-structures)
9. [Memory Management — Virtual vs Physical](#9-memory)
10. [The Windows Boot Process — Attack Surface](#10-boot-process)
11. [Reverse Engineering Walkthrough](#11-reverse-engineering)
12. [Code Examples](#12-code)
13. [ASCII Architecture Diagrams](#13-diagrams)
14. [Attack & Defense Mapping — Per Layer](#14-attack-defense)
15. [Interview Questions & Checkpoints](#15-interview)

---

# 1. Why Windows Architecture Matters for Security

**Beginner:**
If you want to hack Windows systems or defend them, you must understand what Windows is made of internally. It is not enough to know commands — you must understand WHY those commands work, and WHY attacks succeed or fail.

**Expert:**
Every offensive technique — privilege escalation, persistence, defense evasion, credential theft — abuses a specific architectural component. Understanding the architecture lets you:
- Find new attack primitives (not just use known ones)
- Understand EDR/AV detection logic and bypass it
- Write custom tools (shellcode, injectors, rootkits)
- Perform malware analysis at a binary level

```
SECURITY RELEVANCE OF EACH LAYER:

CPU Rings        → Understanding privilege boundaries (ring 0 vs ring 3)
Kernel/User      → Understanding what requires admin vs SYSTEM vs driver
API Layers       → Understanding where EDR hooks to detect calls
PE Format        → Malware analysis, packers, shellcode injection
Kernel Structs   → DKOM rootkits, token impersonation, process hiding
Memory Layout    → Buffer overflows, shellcode, ROP chains, ASLR bypass
Boot Process     → Bootkits, UEFI persistence, boot-level rootkits
```

---

# 2. CPU Privilege Rings — The Foundation

## 2.1 What are CPU Rings?

**Beginner analogy:**
Think of a medieval castle with concentric walls. The innermost ring (Ring 0) is the royal chamber — only the king (kernel) lives there and can do anything. The outermost ring (Ring 3) is the public square — ordinary citizens (user programs) live here with restricted access. Guards at each gate check permissions before letting anyone move inward.

**Technical explanation:**
Modern x86/x64 CPUs implement 4 privilege levels (rings 0–3), controlled by the `CPL` (Current Privilege Level) bits in the `CS` (Code Segment) register. The hardware enforces these boundaries — software cannot bypass them without using specific CPU instructions.

```
CPU PRIVILEGE RINGS — COMPLETE DIAGRAM:

        ┌───────────────────────────────────────────────────────────┐
        │                      RING -2 / -1                        │
        │            SMM (System Management Mode)                   │
        │            Hypervisor (VMX Root Mode)                     │
        │     • Lowest level — controls everything                  │
        │     • Invisible to OS                                     │
        │     • Firmware/BIOS/UEFI runs here                       │
        │     • Where UEFI bootkits and VM escapes live             │
        └───────────────────┬───────────────────────────────────────┘
                            │
        ┌───────────────────▼───────────────────────────────────────┐
        │                      RING 0                              │
        │              KERNEL MODE (Most Privileged)                │
        │                                                           │
        │  • Windows Kernel (ntoskrnl.exe)                         │
        │  • Hardware Abstraction Layer (hal.dll)                   │
        │  • Device Drivers (.sys files)                            │
        │  • NDIS (Network Driver Interface Specification)          │
        │                                                           │
        │  CAPABILITIES:                                            │
        │  ✅ Direct hardware access (memory, ports, interrupts)    │
        │  ✅ Access all physical memory                            │
        │  ✅ Execute privileged CPU instructions (CLI, HLT, RDMSR) │
        │  ✅ No memory access restrictions                         │
        │  ✅ Modify page tables (control virtual memory)           │
        └───────────────────┬───────────────────────────────────────┘
                            │   (Rings 1 and 2 unused by Windows)
        ┌───────────────────▼───────────────────────────────────────┐
        │                      RING 3                              │
        │             USER MODE (Least Privileged)                  │
        │                                                           │
        │  • All user applications (notepad.exe, chrome.exe)        │
        │  • System processes (lsass.exe, svchost.exe)              │
        │  • Security tools (AV user-mode components)               │
        │                                                           │
        │  RESTRICTIONS:                                            │
        │  ❌ Cannot directly access hardware                       │
        │  ❌ Cannot access other processes' memory (by default)    │
        │  ❌ Cannot execute privileged CPU instructions             │
        │  ❌ Cannot write to kernel memory                         │
        │  Must call kernel via syscalls (controlled crossing)      │
        └───────────────────────────────────────────────────────────┘
```

## 2.2 Why Windows Only Uses Ring 0 and Ring 3

Windows uses only Ring 0 (kernel) and Ring 3 (user), skipping 1 and 2:
- **Portability**: Rings 1/2 varied between CPU architectures (Alpha, MIPS, PowerPC). Skipping them kept Windows portable.
- **Simplicity**: Two boundaries are easier to enforce than four.
- **Security model**: The binary Kernel/User split maps cleanly to the Windows security model.

## 2.3 The Privilege Level Transition — How Crossing Works

```
USER MODE → KERNEL MODE TRANSITION (Syscall Path):

User Mode (Ring 3):
  Process calls:  WriteFile("C:\file.txt", ...)
                      │
                      ▼
  kernel32.dll:   CreateFileW()   ← Win32 API (documented)
                      │
                      ▼
  ntdll.dll:      NtWriteFile()   ← Native API (undocumented)
                      │
                      ▼
  CPU Instruction: SYSCALL         ← Triggers mode switch
                   (or SYSENTER on 32-bit)
                      │
                      │  CPU switches CPL from 3 to 0
                      │  Loads kernel stack
                      │  Jumps to syscall handler in ntoskrnl
                      ▼
Kernel Mode (Ring 0):
  ntoskrnl.exe:   NtWriteFile() implementation
                      │  Validates parameters
                      │  Accesses hardware/file system
                      │  Returns result
                      ▼
  CPU Instruction: SYSRET / IRET  ← Return to Ring 3
                      │
                      ▼
User Mode resumes. Return value in RAX register.
```

**Security implication:** EDR/AV products hook this transition at multiple points to intercept calls. Attackers use "direct syscalls" (bypassing ntdll) or "indirect syscalls" (using ntdll stubs but swapping syscall numbers) to evade this monitoring.

---

# 3. Windows Architecture Layers

## 3.1 Full Architecture Stack — Top to Bottom

```
WINDOWS ARCHITECTURE — COMPLETE STACK:

┌─────────────────────────────────────────────────────────────────────┐
│                        USER APPLICATIONS                            │
│   Web browsers, Office, malware.exe, cmd.exe, powershell.exe        │
│                   (Ring 3 — restricted)                             │
└─────────────────────────────┬───────────────────────────────────────┘
                              │ calls
┌─────────────────────────────▼───────────────────────────────────────┐
│                      WIN32 SUBSYSTEM (USER MODE)                    │
│                                                                     │
│  ┌───────────────────┐  ┌──────────────────┐  ┌────────────────┐   │
│  │   kernel32.dll    │  │   user32.dll      │  │   gdi32.dll   │   │
│  │  CreateFile()     │  │  MessageBox()     │  │  DrawText()   │   │
│  │  CreateProcess()  │  │  CreateWindow()   │  │  BitBlt()     │   │
│  └─────────┬─────────┘  └──────────────────┘  └───────────────┘   │
│            │                                                        │
│  ┌─────────▼─────────────────────────────────────────────────────┐ │
│  │               ntdll.dll (Native API Layer)                    │ │
│  │  NtCreateFile(), NtWriteVirtualMemory(), NtCreateThread()     │ │
│  │  Contains the SYSCALL stubs — doorway to kernel               │ │
│  └─────────┬─────────────────────────────────────────────────────┘ │
└────────────┼────────────────────────────────────────────────────────┘
             │ SYSCALL instruction — mode switch here
┌────────────▼────────────────────────────────────────────────────────┐
│                      KERNEL MODE (Ring 0)                           │
│                                                                     │
│  ┌────────────────────────────────────────────────────────────────┐ │
│  │              Executive (ntoskrnl.exe)                         │ │
│  │                                                               │ │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌───────────────┐   │ │
│  │  │  Process │ │  Memory  │ │   I/O    │ │   Security    │   │ │
│  │  │  Manager │ │  Manager │ │  Manager │ │   Reference   │   │ │
│  │  │          │ │  (VMM)   │ │          │ │   Monitor     │   │ │
│  │  └──────────┘ └──────────┘ └──────────┘ └───────────────┘   │ │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐                     │ │
│  │  │  Object  │ │  Cache   │ │  Config  │                     │ │
│  │  │  Manager │ │  Manager │ │  Manager │                     │ │
│  │  └──────────┘ └──────────┘ └──────────┘                     │ │
│  └────────────────────────────────────────────────────────────────┘ │
│                                                                     │
│  ┌────────────────────────────────────────────────────────────────┐ │
│  │              Kernel (Core Scheduling Layer)                   │ │
│  │   Thread scheduling, Interrupt handling, Synchronization      │ │
│  └────────────────────────────────────────────────────────────────┘ │
│                                                                     │
│  ┌────────────────────────────────────────────────────────────────┐ │
│  │              HAL (Hardware Abstraction Layer)                 │ │
│  │   Abstracts CPU, interrupts, timers from the rest of OS      │ │
│  └────────────────────────────────────────────────────────────────┘ │
│                                                                     │
│  ┌────────────────────────────────────────────────────────────────┐ │
│  │              Device Drivers (.sys files)                      │ │
│  │   Filesystem (NTFS), Network (NDIS), Storage, Display        │ │
│  │   ← EDR kernel drivers also live here                        │ │
│  └────────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────┘
             │
┌────────────▼────────────────────────────────────────────────────────┐
│                          HARDWARE                                   │
│        CPU, RAM, Disk, NIC, GPU, USB controllers                    │
└─────────────────────────────────────────────────────────────────────┘
```

## 3.2 Key DLL Files and Their Security Role

| DLL | Full Name | Role | Security Relevance |
|-----|-----------|------|--------------------|
| `ntdll.dll` | NT Layer DLL | Native API + syscall stubs | EDR hooks here; direct syscall bypass target |
| `kernel32.dll` | Kernel32 | Core Win32 API (files, processes, memory) | Most malware uses these functions |
| `kernelbase.dll` | KernelBase | Refactored kernel32 (Win7+) | Many Win32 functions re-routed here |
| `user32.dll` | User32 | GUI, windows, messages | Keyloggers hook `SetWindowsHookEx` here |
| `advapi32.dll` | Advanced API | Registry, security, services, crypto | Malware uses `CreateService`, `RegSetValue` |
| `wininet.dll` | WinInet | HTTP/FTP internet comms | C2 malware uses `InternetOpenUrl` |
| `ws2_32.dll` | Winsock | Raw socket operations | RATs and backdoors use `connect()` here |
| `lsasrv.dll` | LSA Server | Authentication logic in lsass.exe | Credential dumping target (Mimikatz) |
| `wdigest.dll` | WDigest | Legacy plaintext credential storage | Dumped by Mimikatz; disable via reg key |

---

# 4. User Mode vs Kernel Mode — Deep Dive

## 4.1 What "User Mode" Really Means

**Beginner:** User mode is the sandboxed environment where normal programs run. Even if your program crashes, Windows kills just that process — the OS keeps running. Programs cannot directly touch hardware or other programs' memory.

**Technical detail:**
- Each process gets its OWN virtual address space (4GB on 32-bit, 128TB on 64-bit)
- The CPU's Memory Management Unit (MMU) + page tables enforce isolation
- Trying to access another process's memory → `Access Violation` exception
- Trying to execute a privileged instruction → `General Protection Fault`

```
USER MODE MEMORY SPACE (per process, 64-bit Windows):

  0x0000000000000000   ← Lowest user address
  
  User-mode range:
    0x0000000000001000  Code (.text sections)
    0x00007FFFFFFF0000  Stack (grows downward)
    0x00007FFFFFFFFFFF  ← Highest user address
  
  ─────────────────── KERNEL/USER BOUNDARY ───────────────────
  
  Kernel range:
    0xFFFF800000000000  ← Kernel begins (inaccessible from Ring 3)
    0xFFFFFFFFFFFFFFFF  ← Kernel ends (max 64-bit address)

IMPORTANT:
  • User-mode processes CANNOT access kernel range addresses
  • Attempting access → Access Violation exception
  • Exception: Kernel-mapped pages (KUSER_SHARED_DATA at 0x7FFE0000)
    This is a READ-ONLY shared page that gives user mode access to
    system time, CPU features, and other fast-read data
```

## 4.2 What "Kernel Mode" Really Means

**Technical detail:**
- Kernel mode code shares ONE address space — the kernel virtual address space
- A bug in kernel code (driver bug) → immediate BSOD (Blue Screen of Death)
- No memory protection in kernel — all kernel code can access all kernel memory
- Kernel code runs with full hardware privileges — can access all RAM, all I/O ports

**Security implication:**
- Rootkits load kernel drivers to get Ring 0 access — then they can hide anything
- `DRIVER_SIGNATURE_ENFORCEMENT` (DSE) requires drivers to be signed by Microsoft
- Attackers bypass DSE via:
  - Vulnerable signed drivers (BYOVD — Bring Your Own Vulnerable Driver)
  - Kernel exploits (CVEs)
  - Test signing mode (for dev/research)

## 4.3 The Kernel/User Boundary as an Attack Surface

```
ATTACK TECHNIQUES AT THE KERNEL/USER BOUNDARY:

OFFENSIVE (abusing the boundary):
  1. Direct Syscall:
     • Skip ntdll.dll entirely
     • Call SYSCALL instruction directly with syscall number in EAX
     • Bypasses ntdll hooks (EDR/AV see nothing via ntdll hooks)
     
     Example (Havoc C2 / Cobalt Strike):
     ; Instead of calling NtAllocateVirtualMemory via ntdll:
     mov r10, rcx
     mov eax, 0x18    ; Syscall number for NtAllocateVirtualMemory
     syscall
     ret
  
  2. Heaven's Gate (32-bit process on 64-bit Windows):
     • 32-bit process can switch to 64-bit mode via far jump
     • Call 64-bit syscalls directly from a 32-bit process
     • Most security tools miss 64-bit calls from 32-bit processes
  
  3. Kernel Exploit:
     • Use a kernel vulnerability (race condition, UAF, overflow)
     • Gain Ring 0 execution
     • Disable AMSI, modify tokens, patch DSE

DEFENSIVE (monitoring the boundary):
  1. ntdll.dll Hooking (most common):
     • EDR patches ntdll.dll functions in memory at process startup
     • Hooks NtAllocateVirtualMemory, NtWriteVirtualMemory, etc.
     • Before call goes to kernel, EDR inspects parameters
     • Limitation: Bypassed by direct syscalls
  
  2. Kernel Callbacks (best visibility):
     • EDR kernel driver registers OS callbacks:
       PsSetCreateProcessNotifyRoutine → called for every process
       PsSetCreateThreadNotifyRoutine → called for every thread
       ObRegisterCallbacks → called for every object access
     • Cannot be bypassed by direct syscalls (runs IN kernel)
     • Modern EDR uses BOTH methods for layered coverage
  
  3. ETW (Event Tracing for Windows) monitoring:
     • Kernel ETW providers emit events for syscalls, network, etc.
     • ETW-TI (Threat Intelligence) provider: high-fidelity kernel events
     • Attackers patch ETW providers in memory to suppress events
```

---

# 5. The Windows API Call Chain

## 5.1 Full Example: CreateFile() — What Really Happens

```
COMPLETE CALL CHAIN — CreateFile("evil.txt", ...)

1. APPLICATION LAYER:
   Your code calls:  CreateFileA("C:\\evil.txt", ...)
   
2. KERNEL32.DLL:
   CreateFileA() → converts to Unicode → calls CreateFileW()
   CreateFileW() → validates parameters → calls CreateFileInternal()
   CreateFileInternal() → calls → NtCreateFile() in ntdll.dll
   
   ⚠️  EDR HOOK POINT #1: kernel32!CreateFileW hooked
   
3. NTDLL.DLL (Native API — user mode syscall stubs):
   NtCreateFile():
     mov r10, rcx         ; Save first arg (rcx → r10 per syscall ABI)
     mov eax, 0x55        ; Syscall number for NtCreateFile (varies by OS version!)
     syscall              ; TRANSITION TO KERNEL MODE
     ret
   
   ⚠️  EDR HOOK POINT #2: ntdll!NtCreateFile hooked (most common)
   
4. KERNEL SYSCALL HANDLER (ntoskrnl.exe):
   nt!NtCreateFile():
     • Validates all parameters (from user space — could be malicious)
     • Calls Security Reference Monitor to check DACL permissions
     • Calls I/O Manager to create file object
     • Passes to NTFS driver (ntfs.sys) for actual disk operation
   
   ⚠️  EDR HOOK POINT #3: Kernel minifilter driver sees ALL file ops
   
5. NTFS DRIVER (ntfs.sys — Ring 0):
   • Reads Master File Table (MFT) — NTFS's database of all files
   • Writes new file record if creating
   • Returns file handle
   
6. RETURN PATH:
   File handle → NtCreateFile (kernel) → SYSRET → ntdll stub → 
   kernel32!CreateFileW → Your application
   Handle is now valid: e.g., handle = 0x00000080
```

## 5.2 Syscall Numbers — Why They Matter

```
SYSCALL NUMBERS CHANGE BETWEEN WINDOWS VERSIONS:

Function         Win10 1507  Win10 1809  Win10 21H2  Win11 22H2
NtAllocateVMem      0x18       0x18        0x18        0x18
NtCreateFile        0x55       0x55        0x55        0x55
NtWriteVMem         0x3A       0x3A        0x3A        0x3A
NtCreateThread      0xBD       0xBD        0xBD        0xBD

ISSUE FOR DIRECT SYSCALLS:
  If you hardcode syscall number 0x18 for NtAllocateVirtualMemory
  and run on a different OS version where the number differs,
  your call will invoke the WRONG syscall → crash or wrong behavior.

SOLUTION (used by offensive tools):
  1. Hell's Gate: Parse ntdll.dll's EAT at runtime to get syscall numbers
  2. Syswhispers3: Tool that generates dynamic syscall number resolution
  3. Tartarus' Gate: Scans for clean syscall stubs in memory
```

---

# 6. Process Internals — What is a Process Really?

## 6.1 What a Process Contains

**Beginner:** A process is a running program. But internally, it is much more than just code executing — it is a container with its own memory, handles, threads, and security context.

```
PROCESS = VIRTUAL CONTAINER:

┌─────────────────────────────────────────────────────────────────┐
│                    PROCESS (e.g., notepad.exe PID 4412)         │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │              VIRTUAL ADDRESS SPACE (128TB)               │   │
│  │                                                          │   │
│  │  [0x00000000 - 0x7FFFFFFFFFFF]  User mode range         │   │
│  │                                                          │   │
│  │  Sections:                                               │   │
│  │    Code  (.text): notepad.exe code                       │   │
│  │    Data  (.data): global variables                       │   │
│  │    Stack: 1 stack per thread (default 1MB)               │   │
│  │    Heap:  dynamic allocations (malloc/HeapAlloc)         │   │
│  │    Shared: DLLs mapped (kernel32.dll, ntdll.dll, ...)    │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ┌──────────────────┐   ┌──────────────────────────────────┐   │
│  │  HANDLE TABLE    │   │  SECURITY CONTEXT (TOKEN)        │   │
│  │                  │   │                                  │   │
│  │  0x04 → File     │   │  User: CORP\jsmith               │   │
│  │  0x08 → Process  │   │  Groups: Users, Remote Desktop   │   │
│  │  0x0C → Thread   │   │  Privileges: SeChangeNotify,    │   │
│  │  0x10 → Event    │   │    SeShutdown                    │   │
│  │  ...             │   │  Integrity: Medium               │   │
│  └──────────────────┘   └──────────────────────────────────┘   │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │              THREADS (execution units)                   │   │
│  │  Thread 1: Main thread (executing notepad's UI loop)     │   │
│  │  Thread 2: Background thread (autosave)                  │   │
│  │  Each thread has: own stack, own register set            │   │
│  │  All threads SHARE: virtual memory, handles, token       │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

## 6.2 The PEB (Process Environment Block)

The PEB is a crucial user-mode structure that describes a process. Attackers read and manipulate the PEB constantly.

```
PEB (Process Environment Block) — Located at: gs:[0x60] (64-bit)

KEY PEB FIELDS (for security):

  PEB.ImageBaseAddress     → Where the main EXE is loaded in memory
  PEB.Ldr                  → Pointer to PEB_LDR_DATA
                              Contains list of all loaded DLLs
                              Attackers walk this to find ntdll base address
                              Attackers modify this to hide loaded DLLs
  PEB.ProcessParameters    → Command line, working directory, environment
  PEB.BeingDebugged        → Anti-debug: set to 1 if process is being debugged
  PEB.NtGlobalFlag         → Anti-debug: certain bits set when debugger present
  PEB.HeapFlags            → Anti-debug: heap behavior changes under debugger

COMMON ANTI-DEBUG CHECKS (malware reads these):
  mov rax, gs:[0x60]           ; Load PEB address
  movzx eax, byte [rax + 0x02] ; PEB.BeingDebugged
  test al, al
  jnz malware_exits_if_debugged
  
ATTACKER USE:
  • Shellcode walks PEB.Ldr linked list to find ntdll.dll base address
    (without calling LoadLibrary, which would be suspicious)
  • Then parses ntdll.dll's EAT to find function addresses
  • Used in "position-independent" shellcode (no hardcoded addresses)
```

## 6.3 Access Tokens — The Security Identity of a Process

**Beginner:** An access token is like an ID badge worn by a process. Every time the process tries to open a file, access a registry key, or create a network connection — Windows checks the badge. The badge lists who you are, what groups you belong to, and what special privileges you have.

```
ACCESS TOKEN STRUCTURE:

┌──────────────────────────────────────────────────────────────────┐
│                      ACCESS TOKEN                                │
│  (Attached to every process and thread)                          │
│                                                                  │
│  User SID:        S-1-5-21-12345-67890-1234-1001 (jsmith)       │
│  Group SIDs:      S-1-5-32-544 (Administrators)   [ENABLED]     │
│                   S-1-5-21-...-513 (Domain Users)  [ENABLED]    │
│                   S-1-16-12288 (High Integrity)     [ENABLED]   │
│                                                                  │
│  Privileges:                                                     │
│    SeDebugPrivilege          → Disabled (UAC removed it)        │
│    SeImpersonatePrivilege    → Enabled                          │
│    SeAssignPrimaryToken      → Disabled                         │
│    SeShutdownPrivilege       → Disabled                         │
│    SeChangeNotifyPrivilege   → Enabled (always on for users)    │
│                                                                  │
│  Integrity Level: High (Administrator token after UAC elevation) │
│  Token Type: Primary (processes use primary tokens)              │
│              Impersonation (threads can use impersonation tokens)│
└──────────────────────────────────────────────────────────────────┘

ATTACKER TOKEN ABUSE TECHNIQUES:
  1. Token Impersonation:
     • Steal another process's token (e.g., SYSTEM token from lsass.exe)
     • Use ImpersonateLoggedOnUser() or OpenProcessToken() + DuplicateToken()
     • Your process runs with a different user's identity

  2. Token Manipulation (Privilege Escalation):
     • Process running as SYSTEM
     • Enable disabled privileges: AdjustTokenPrivileges()
     • Grant SeDebugPrivilege, SeImpersonatePrivilege

  3. Pass-the-Token:
     • Steal Kerberos TGT or NTLM hash from LSASS
     • Inject into new process token
     • Lateral movement with stolen identity

WINDOWS INTEGRITY LEVELS (Mandatory Integrity Control):
  System:    0x4000  Kernel and system processes (SYSTEM account)
  High:      0x3000  Admin processes after UAC elevation
  Medium:    0x2000  Normal user processes (notepad, word, chrome)
  Low:       0x1000  Internet Explorer tabs, sandbox
  Untrusted: 0x0000  Fully sandboxed (AppContainer)

  RULE: A process can only write to objects AT or BELOW its integrity level.
  LOW cannot write to MEDIUM objects.
  MEDIUM cannot write to HIGH objects.
  Privilege escalation = elevating integrity level.
```

---

# 7. The PE Format — Complete Anatomy

## 7.1 What is the PE Format?

**Beginner:** PE stands for Portable Executable. It is the file format that Windows uses for all programs (`.exe`), libraries (`.dll`), and drivers (`.sys`). Just like a book has a cover, table of contents, and chapters, a PE file has a header, section table, and sections. Windows reads the header to know how to load and run the program.

**Expert:** The PE format derives from the COFF (Common Object File Format) specification. Understanding PE structure is mandatory for malware analysis, shellcode writing, packer analysis, and AV bypass.

## 7.2 Complete PE Layout

```
PE FILE STRUCTURE — COMPLETE LAYOUT:

    ON DISK (File)                       IN MEMORY (After loading)
    ─────────────                        ──────────────────────────
    ┌──────────────────────┐             ┌──────────────────────────┐
    │  DOS Header          │             │  DOS Header              │
    │  "MZ" magic (4D 5A) │             │  (same)                  │
    │  e_lfanew → offset   │             │                          │
    │  of NT headers       │             │                          │
    ├──────────────────────┤             ├──────────────────────────┤
    │  DOS Stub            │             │  DOS Stub                │
    │  "This program       │             │                          │
    │  cannot be run       │             │                          │
    │  in DOS mode"        │             │                          │
    ├──────────────────────┤             ├──────────────────────────┤
    │  NT Headers          │             │  NT Headers              │
    │  Signature: "PE\0\0" │             │                          │
    │  ┌────────────────┐  │             │                          │
    │  │  File Header   │  │             │                          │
    │  │  Machine type  │  │             │                          │
    │  │  NumSections   │  │             │                          │
    │  │  TimeDateStamp │  │             │                          │
    │  └────────────────┘  │             │                          │
    │  ┌────────────────┐  │             │                          │
    │  │Optional Header │  │             │                          │
    │  │  EntryPoint RVA│  │             │                          │
    │  │  ImageBase     │  │             │                          │
    │  │  SectionAlign  │  │             │                          │
    │  │  FileAlignment │  │             │                          │
    │  │  DataDirectory │  │             │                          │
    │  └────────────────┘  │             │                          │
    ├──────────────────────┤             ├──────────────────────────┤
    │  Section Headers     │             │  Section Headers         │
    │  (Table of Contents) │             │                          │
    │  .text header        │             │                          │
    │  .data header        │             │                          │
    │  .rdata header       │             │                          │
    ├──────────────────────┤             ├──────────────────────────┤ ← ImageBase
    │                      │             │  .text  [page aligned]   │
    │  .text (raw)         │  ═══════►   │  Code — RX permissions   │
    │  512-byte aligned    │             │  4096-byte page aligned  │
    ├──────────────────────┤             ├──────────────────────────┤
    │  .data (raw)         │  ═══════►   │  .data  [page aligned]   │
    │  512-byte aligned    │             │  Global vars — RW perms  │
    ├──────────────────────┤             ├──────────────────────────┤
    │  .rdata (raw)        │  ═══════►   │  .rdata  [page aligned]  │
    │                      │             │  Strings, IAT — R only   │
    ├──────────────────────┤             ├──────────────────────────┤
    │  .rsrc (raw)         │  ═══════►   │  .rsrc                   │
    │  Resources           │             │  Icons, manifests        │
    └──────────────────────┘             └──────────────────────────┘
    
    NOTE: File alignment (disk) ≠ Section alignment (memory)
          Disk: typically 512 bytes
          Memory: typically 4096 bytes (one page)
```

## 7.3 The Import Address Table (IAT) — How Malware is Identified

```
IAT — DETAILED BREAKDOWN:

CONCEPT:
  A PE file lists all external functions it needs.
  These are listed in the Import Directory → Import Descriptor Tables → IAT.
  When the PE loader loads the binary, it resolves each function's actual
  memory address and writes it into the IAT.

STRUCTURE:
  Import Descriptor (one per DLL imported):
    Name:             "KERNEL32.DLL\0"     ← Which DLL
    FirstThunk:       → Pointer to IAT     ← Where loader writes addresses
    OriginalFirstThunk: → Pointer to INT   ← Import Name Table (function names)

  After loading:
    IAT[0] = 0x7FF805AF1234  ← Actual address of CreateFileW in memory
    IAT[1] = 0x7FF805AF5678  ← Actual address of WriteFile in memory

MALWARE SIGNATURE IN IAT:
  These import combinations are highly suspicious:
  
  PROCESS INJECTION:
    VirtualAllocEx    → Allocate memory in target process
    WriteProcessMemory → Write shellcode to target process
    CreateRemoteThread → Execute shellcode in target process
  
  DOWNLOAD/DROPPER:
    InternetOpenUrl   → Connect to URL
    InternetReadFile  → Download content
    VirtualAlloc      → Allocate memory for downloaded payload
    CreateThread      → Execute downloaded payload
  
  KEYLOGGER:
    SetWindowsHookEx  → Install keyboard hook
    GetAsyncKeyState  → Read key state
    
  CREDENTIAL THEFT:
    OpenProcess       → Open lsass.exe
    ReadProcessMemory → Read lsass.exe memory
```

## 7.4 Sections — Malware Red Flags

```
SECTION ANOMALIES = MALWARE INDICATORS:

NORMAL PE:
  Name     VirtSize  RawSize  Characteristics
  .text    0x4A200   0x4A400  0x60000020 (CODE | EXECUTE | READ)
  .data    0x3000    0x2000   0xC0000040 (INITIALIZED | READ | WRITE)
  .rdata   0x8000    0x8000   0x40000040 (INITIALIZED | READ)
  .rsrc    0x5000    0x5000   0x40000040 (INITIALIZED | READ)

PACKED MALWARE:
  Name     VirtSize  RawSize  Characteristics
  .text    0x000A    0x0200   0xE0000020 ← RWX! (READ | WRITE | EXECUTE)
  UPX0     0xA0000   0x0000   0xE0000020 ← VirtSize >> RawSize (decompresses in memory)
  UPX1     0x5000    0x5000   0xE0000020 ← Stub code that decompresses
  
  RED FLAGS:
  ✅ Section named "UPX0/UPX1" → UPX packer
  ✅ VirtualSize >> SizeOfRawData → Content is compressed/packed
  ✅ RWX section permissions → Shellcode will be written and executed
  ✅ High entropy section → Encrypted/compressed content
  ✅ Unknown/random section names → Custom packer

ENTROPY ANALYSIS:
  Normal code entropy: 5.0–6.5 bits/byte
  Encrypted/packed:    7.0–8.0 bits/byte (max is 8.0)
  
  Tools:
    die (Detect It Easy) → shows entropy per section
    PE-Bear → entropy visualization
    FLOSS → extracts obfuscated strings
```

---

# 8. Kernel Data Structures

## 8.1 EPROCESS — The Kernel's View of a Process

```
EPROCESS STRUCTURE (Simplified — Windows 10/11):

Offset  Field                    Description
──────────────────────────────────────────────────────────────────
0x000   Pcb (KPROCESS)          Kernel scheduling info
0x2D8   UniqueProcessId          PID (what you see in Task Manager)
0x2E0   ActiveProcessLinks       DOUBLY-LINKED LIST of all processes
                                 Flink → next EPROCESS
                                 Blink → prev EPROCESS
                                 ← ROOTKIT TARGET: unlink to hide process
0x300   QuotaBlock               Memory quota info
0x358   ObjectTable              Handle table pointer
0x360   Token                    Security token (what privileges this process has)
                                 ← TOKEN STEALING TARGET (for privilege escalation)
0x3B0   ImageFileName            Process name (15 chars, e.g., "lsass.exe")
0x3C8   PriorityClass            Priority for scheduling
0x550   Peb                      Pointer to User-mode PEB
0x7D0   Protection               PP (Protected Process) flags

IMPORTANT KERNEL STRUCTURES ATTACHED:
  • KPROCESS (scheduling)
  • VAD (Virtual Address Descriptor) tree → maps virtual memory
  • Object table → all handles this process owns
  • Token → security context
```

## 8.2 Rootkit Technique — DKOM Process Hiding

```
DKOM (Direct Kernel Object Manipulation) — PROCESS HIDING:

NORMAL PROCESS LIST (doubly-linked via ActiveProcessLinks):
  
  [System] ↔ [smss.exe] ↔ [csrss.exe] ↔ [winlogon.exe] ↔
  [svchost.exe] ↔ [malware.exe] ↔ [explorer.exe]

ATTACK — UNLINK malware.exe FROM THE LIST:
  1. Load kernel driver (requires Ring 0)
  2. Find EPROCESS of malware.exe
  3. Modify Flink/Blink pointers of NEIGHBORS:
  
  Before:
    svchost.exe.Flink = &malware.exe.ActiveProcessLinks
    explorer.exe.Blink = &malware.exe.ActiveProcessLinks

  After DKOM:
    svchost.exe.Flink = &explorer.exe.ActiveProcessLinks  ← skip malware.exe
    explorer.exe.Blink = &svchost.exe.ActiveProcessLinks  ← skip malware.exe

  malware.exe.Flink and .Blink are now dangling pointers.
  malware.exe is STILL RUNNING (scheduler uses KPROCESS, not this list).
  
RESULT:
  Task Manager → does NOT show malware.exe (walks ActiveProcessLinks)
  Process Explorer → does NOT show malware.exe
  NtQuerySystemInformation → does NOT return malware.exe
  
  BUT malware.exe:
  ✅ Still executes
  ✅ Still has network connections
  ✅ Still has file handles

DETECTION (how forensics finds it):
  • Scan ALL physical memory for EPROCESS magic bytes
  • Compare memory-scan list vs API list → difference = hidden process
  • Tools: Volatility (process list via multiple methods), Rekall
```

## 8.3 Access Token Manipulation in the Kernel

```
TOKEN STEALING — HOW ATTACKERS ELEVATE PRIVILEGES:

GOAL: Get SYSTEM-level access from a Medium-integrity process

METHOD 1: OpenProcess + Token APIs (User Mode):
  1. Find a process running as SYSTEM (e.g., lsass.exe, winlogon.exe)
  2. OpenProcess(PROCESS_QUERY_INFORMATION, FALSE, pid_of_lsass)
     → Requires SeDebugPrivilege (only admins have this)
  3. OpenProcessToken(h_lsass, TOKEN_DUPLICATE, &h_token)
  4. DuplicateTokenEx(h_token, MAXIMUM_ALLOWED, NULL, 
       SecurityImpersonation, TokenPrimary, &h_dup_token)
  5. CreateProcessWithTokenW(h_dup_token, ..., "cmd.exe", ...)
     → cmd.exe runs as SYSTEM

METHOD 2: SYSTEM Shell via Token Impersonation (Incognito / Meterpreter):
  meterpreter> getsystem
  → Tries multiple token impersonation techniques
  → One works: spawns SYSTEM shell

METHOD 3: Kernel Driver — Direct Token Replacement (most reliable):
  1. Load kernel driver (Ring 0 access)
  2. Find EPROCESS of SYSTEM process:
     PsLookupProcessByProcessId(4, &SystemEPROCESS)
  3. Read SYSTEM token from SYSTEM EPROCESS:
     SystemToken = *(SYSTEM_EPROCESS + TOKEN_OFFSET)
  4. Find our process's EPROCESS
  5. Write SYSTEM token into our process:
     *(OurEPROCESS + TOKEN_OFFSET) = SystemToken
  6. We now have SYSTEM token → full administrative access
```

---

# 9. Memory Management — Virtual vs Physical

## 9.1 Virtual Memory — How Processes "See" Memory

```
VIRTUAL MEMORY CONCEPT:

PHYSICAL RAM: 16GB actual (one resource, shared by all)

VIRTUAL ADDRESS SPACE: Each process gets its OWN 128TB virtual space (64-bit)
  notepad.exe thinks it owns memory from 0x0 to 0x7FFFFFFFFFFF (128TB)
  chrome.exe   thinks it owns memory from 0x0 to 0x7FFFFFFFFFFF (128TB)
  
  In reality, there is only 16GB of physical RAM.
  The CPU's MMU + Page Tables do the translation.

PAGE TABLE TRANSLATION:
  Virtual Address 0x401000 (notepad.exe code)
       │
       │ MMU looks up page table entry
       ▼
  Physical Address 0x14F4000 (actual RAM)
  
  For chrome.exe:
  Virtual Address 0x401000 (same virtual address!)
       │
       │ Different page table entry
       ▼
  Physical Address 0x9A20000 (different actual RAM)
  
  → Same virtual address, different physical address. Complete isolation.

PAGE STATES (important for shellcode/injection):
  COMMITTED:  Memory has physical backing (RAM or page file)
  RESERVED:   Address range claimed but no physical backing yet
  FREE:       Not in use at all
  
  Attacker use of VirtualAlloc:
    VirtualAlloc(NULL, 0x1000, MEM_COMMIT|MEM_RESERVE, PAGE_EXECUTE_READWRITE)
    → Allocates 4096 bytes with RWX permissions
    → Write shellcode → Execute shellcode
    
  Detection: VirtualAlloc with PAGE_EXECUTE_READWRITE = strong indicator
```

## 9.2 Key Memory Security Features

| Feature | Description | Bypass Technique |
|---------|-------------|------------------|
| **ASLR** (Address Space Layout Randomization) | Randomizes load addresses of modules | Info leak → calculate base address |
| **DEP/NX** (Data Execution Prevention) | Memory pages are W^X (write XOR execute) | ROP (Return-Oriented Programming) |
| **Stack Canaries** | Random value before return address on stack | Info leak or format string to read canary |
| **CFG** (Control Flow Guard) | Validates indirect call targets | CFG bypass via unchecked targets |
| **VBS/HVCI** | Hypervisor-enforced code integrity | Requires firmware-level attack |
| **Heap Isolation** | Separate heaps per type to prevent spray | Heap grooming techniques |

---

# 10. The Windows Boot Process — Attack Surface

## 10.1 Complete Boot Chain

```
FULL BOOT SEQUENCE — FROM POWER ON TO LOGON:

PHASE 0: POWER ON
  CPU reset → Begins executing from RESET vector (0xFFFFFFF0)
  Jumps to UEFI firmware (in NVRAM/SPI flash on motherboard)
  
PHASE 1: UEFI FIRMWARE (Ring -2 / SMM)
  ┌─────────────────────────────────────────────────────┐
  │ UEFI Initialization                                 │
  │  • Hardware discovery (PCI, USB, SATA, NVMe)        │
  │  • Memory initialization (DRAM training)            │
  │  • Secure Boot verification (if enabled):           │
  │    → Checks digital signature of bootloader         │
  │    → Only Microsoft/OEM-signed bootloaders allowed  │
  │                                                     │
  │ ATTACK: UEFI Bootkit (e.g., BlackLotus)            │
  │  • Modifies UEFI module to persist before OS loads  │
  │  • Survives OS reinstall (lives in UEFI flash)      │
  │  • Can disable Secure Boot, inject drivers          │
  └───────────────────────┬─────────────────────────────┘
                          │
PHASE 2: BOOT MANAGER (bootmgfw.efi / bootmgr)
  ┌─────────────────────────────────────────────────────┐
  │ Windows Boot Manager                                │
  │  • Reads BCD (Boot Configuration Data) from ESP     │
  │  • BCD = Modern boot.ini (stored in registry hive) │
  │  • Presents boot menu (if multiple OS)              │
  │  • Loads OS loader: winload.efi                     │
  │                                                     │
  │ ATTACK: BCD Modification                           │
  │  bcdedit /set {bootmgr} path \Windows\bootmgr.bak  │
  │  → Replace legitimate loader with malicious one     │
  └───────────────────────┬─────────────────────────────┘
                          │
PHASE 3: OS LOADER (winload.efi)
  ┌─────────────────────────────────────────────────────┐
  │ Windows OS Loader                                   │
  │  • Reads SYSTEM registry hive (from disk)           │
  │  • Loads ntoskrnl.exe (kernel) into memory         │
  │  • Loads hal.dll (hardware abstraction layer)       │
  │  • Loads ALL drivers marked START_TYPE = BOOT (0)  │
  │    Registry: HKLM\SYSTEM\CurrentControlSet\Services │
  │    Start=0 → boot driver (loads before kernel init) │
  │  • Verifies driver signatures (DSE — Driver Sign.)  │
  │                                                     │
  │ ATTACK: Boot Driver Persistence                    │
  │  • Register malicious driver as Start=0 (Boot)     │
  │  • Loaded before any AV/EDR                        │
  │  • Can disable EDR before it loads                  │
  │  • Requires BYOVD or kernel exploit to sign driver  │
  └───────────────────────┬─────────────────────────────┘
                          │
PHASE 4: KERNEL INITIALIZATION (ntoskrnl.exe)
  ┌─────────────────────────────────────────────────────┐
  │ Kernel Init                                         │
  │  • Initialize all Executive subsystems             │
  │  • Set up HAL, ACPI                                │
  │  • Initialize Memory Manager (page tables)         │
  │  • Initialize I/O Manager, Object Manager          │
  │  • Load System drivers (Start=1 SERVICE_SYSTEM_START│
  │  • Start SMSS.EXE (PID 4 = System, PID 6 = smss)  │
  └───────────────────────┬─────────────────────────────┘
                          │
PHASE 5: SESSION MANAGER (smss.exe)
  ┌─────────────────────────────────────────────────────┐
  │ SMSS.EXE — Session Manager Subsystem               │
  │  • First user-mode process (PID 6)                  │
  │  • Uses ONLY ntdll.dll (native API only)            │
  │  • Creates paging file (pagefile.sys)               │
  │  • Loads system environment variables               │
  │  • Runs BootExecute programs:                       │
  │      Registry: HKLM\SYSTEM\CurrentControlSet\       │
  │                Control\Session Manager\BootExecute │
  │      Default: "autocheck autochk *" (chkdsk)       │
  │  • Starts csrss.exe and winlogon.exe               │
  │                                                     │
  │ ATTACK: BootExecute Persistence                    │
  │  Add malware to BootExecute key                    │
  │  → Runs before any user-mode security tools        │
  └───────────────────────┬─────────────────────────────┘
                          │
PHASE 6: SUBSYSTEM + LOGON
  ┌─────────────────────────────────────────────────────┐
  │ CSRSS.EXE (Client/Server Runtime Subsystem)        │
  │  • Handles Win32 console windows                    │
  │  • Manages process/thread lifecycle for Win32       │
  │                                                     │
  │ WINLOGON.EXE                                        │
  │  • Shows logon screen (GINA / Credential Provider) │
  │  • Loads LSASS.EXE for credential verification     │
  │  • On success: runs userinit.exe                   │
  │                                                     │
  │ LSASS.EXE (Local Security Authority Subsystem)     │
  │  • Verifies credentials (NTLM/Kerberos)            │
  │  • Caches credentials (target of Mimikatz)         │
  │  • Creates logon sessions + access tokens          │
  │                                                     │
  │ USERINIT.EXE → runs logon scripts → launches SHELL │
  │                                                     │
  │ EXPLORER.EXE (Shell)                               │
  │  • Desktop, taskbar, Start menu                    │
  │  • All user-mode persistence runs from here down   │
  └─────────────────────────────────────────────────────┘
```

## 10.2 Persistence Points — Sorted by Boot Phase

```
PERSISTENCE TIMELINE — EARLIEST TO LATEST LOADING:

PHASE          LOCATION                              ACCESS REQUIRED
──────────────────────────────────────────────────────────────────────────
UEFI/SPI      UEFI firmware flash                   Firmware write access
  └─► DEEPEST persistence — survives OS reinstall

MBR           Master Boot Record (sector 0)          Raw disk write (SYSTEM)
VBR           Volume Boot Record                     Raw disk write (SYSTEM)

Boot Driver   HKLM\SYSTEM\CCS\Services\[mal]\        SYSTEM + driver signing
              Start = 0 (SERVICE_BOOT_START)

System Driver Start = 1 (SERVICE_SYSTEM_START)       SYSTEM + driver signing

BootExecute   HKLM\SYSTEM\CCS\Control\              SYSTEM
              Session Manager\BootExecute

Init Reg Key  HKLM\SYSTEM\CCS\Control\              SYSTEM
              Session Manager\KnownDLLs  (DLL hijack)

Service       HKLM\SYSTEM\CCS\Services\[mal]\        Administrator
              Start = 2 (SERVICE_AUTO_START)
              ImagePath = C:\Windows\mal.exe

AppInit_DLLs  HKLM\SOFTWARE\Microsoft\              Administrator
              Windows NT\CurrentVersion\             (loaded into EVERY process
              Windows\AppInit_DLLs                   that loads user32.dll)

Winlogon Notify HKLM\SOFTWARE\Microsoft\            Administrator
              Windows NT\CurrentVersion\
              Winlogon\Notify

LSA Provider  HKLM\SYSTEM\CCS\Control\LSA\          Administrator (SYSTEM)
              SecurityPackages

Scheduled Task HKLM\SOFTWARE\Microsoft\Windows\     Administrator
               NT\CurrentVersion\Schedule\
               TaskCache\Tasks

Run Keys      HKLM\SOFTWARE\Microsoft\Windows\      Administrator (HKLM)
              CurrentVersion\Run                     or User (HKCU)

Startup Folder C:\ProgramData\Microsoft\Windows\     Any user
               Start Menu\Programs\Startup\
  └─► SHALLOWEST persistence — last to load
```

---

# 11. Reverse Engineering Walkthrough

## 11.1 Scenario: Analyzing a Packed Malware Sample

**Tools needed:** PE-Bear or CFF Explorer, strings / FLOSS, x64dbg (dynamic), Ghidra/IDA (static)

```
STEP 1: FILE IDENTIFICATION
  file malware.exe
  → PE32+ executable (console) x86-64, for MS Windows

  diec malware.exe  (Detect-It-Easy)
  → Packer: UPX 3.95
  → Language: C
  → Entropy: 7.82 (HIGH — confirms compression)

STEP 2: STATIC HEADER ANALYSIS (PE-Bear)
  
  Section Table:
    .text  VirtSize: 0x10    RawSize: 0x200  → Suspiciously small!
    UPX0   VirtSize: 0xB0000 RawSize: 0x0    → Raw=0, will expand in memory
    UPX1   VirtSize: 0x25000 RawSize: 0x25000 → Contains stub code
    .rsrc  VirtSize: 0x4000  RawSize: 0x4000  → Resources
  
  Imports: Very few (just LoadLibrary and GetProcAddress)
  → Confirms packing: loader resolves all imports at runtime
  
  EntryPoint: Points into UPX1 section (the decompression stub)

STEP 3: STRING ANALYSIS
  strings malware.exe | grep -E "http|cmd|powershell|\\\\\.\\\\pipe"
  → http://update.evil-domain.com/payload
  → cmd.exe /c whoami
  → \\.\pipe\malware_pipe
  (These strings are in plaintext within UPX stub — often visible)
  
  FLOSS malware.exe
  → Attempts to deobfuscate strings dynamically

STEP 4: DYNAMIC UNPACKING (x64dbg)
  1. Load in x64dbg
  2. Set breakpoint at OEP (Original Entry Point)
     Technique: Break on VirtualProtect calls (packer changes section perms)
  3. Run until packer finishes decompression
  4. At OEP break — now real code is in memory
  5. Dump process memory → analyze unpacked binary

STEP 5: BEHAVIORAL ANALYSIS
  Process Monitor (ProcMon) while running:
  → Creates: C:\Users\User\AppData\Roaming\svchost32.exe (dropped payload)
  → Sets: HKCU\Software\Microsoft\Windows\CurrentVersion\Run\svchost32
  → Connects: TCP 45.33.32.156:443 (C2 server)
  → Injects into: explorer.exe (ProcessInject API calls)

INDICATORS OF COMPROMISE (IOCs) identified:
  • File: svchost32.exe (SHA256: 4a3b9f...)
  • Registry: HKCU\...\Run\svchost32
  • Network: 45.33.32.156:443
  • Pipe: \\.\pipe\malware_pipe
  • Parent-child: explorer.exe spawning powershell.exe
```

---

# 12. Code Examples

## 12.1 PE Header Parser (Python)

```python
import pefile
import sys
import math
import hashlib

def entropy(data):
    """Calculate Shannon entropy of bytes"""
    if not data:
        return 0.0
    freq = [0] * 256
    for byte in data:
        freq[byte] += 1
    length = len(data)
    return -sum((c/length) * math.log2(c/length) for c in freq if c)

def analyze_pe(file_path):
    try:
        pe = pefile.PE(file_path)
    except pefile.PEFormatError as e:
        print(f"[-] Not a valid PE: {e}")
        return

    # --- Hash ---
    with open(file_path, "rb") as f:
        data = f.read()
    print(f"[*] MD5:    {hashlib.md5(data).hexdigest()}")
    print(f"[*] SHA256: {hashlib.sha256(data).hexdigest()}")

    # --- Headers ---
    print(f"\n[*] Machine:      {hex(pe.FILE_HEADER.Machine)}")
    print(f"[*] Compiled:     {pe.FILE_HEADER.TimeDateStamp}")
    print(f"[*] Image Base:   {hex(pe.OPTIONAL_HEADER.ImageBase)}")
    print(f"[*] Entry Point:  {hex(pe.OPTIONAL_HEADER.AddressOfEntryPoint)}")
    print(f"[*] Num Sections: {pe.FILE_HEADER.NumberOfSections}")

    # --- Sections ---
    print("\n[*] SECTIONS:")
    print(f"  {'Name':<10} {'VirtSize':<12} {'RawSize':<12} {'Entropy':<10} Flags")
    for s in pe.sections:
        name = s.Name.decode(errors='replace').strip('\x00')
        vsize = hex(s.Misc_VirtualSize)
        rsize = hex(s.SizeOfRawData)
        ent   = entropy(s.get_data())
        flags = hex(s.Characteristics)

        warning = ""
        if s.Characteristics & 0x20000000 and s.Characteristics & 0x80000000:
            warning = " ⚠️  RWX!"
        if ent > 7.0:
            warning += " ⚠️  HIGH ENTROPY (packed?)"
        if s.Misc_VirtualSize > 0 and s.SizeOfRawData == 0:
            warning += " ⚠️  VirtSize>0 RawSize=0 (unpacks in memory)"

        print(f"  {name:<10} {vsize:<12} {rsize:<12} {ent:<10.4f} {flags}{warning}")

    # --- Suspicious Imports ---
    SUSPICIOUS = {
        "VirtualAllocEx", "WriteProcessMemory", "CreateRemoteThread",
        "OpenProcess", "ReadProcessMemory", "SetWindowsHookEx",
        "InternetOpenUrl", "InternetReadFile", "URLDownloadToFile",
        "CreateRemoteThreadEx", "NtUnmapViewOfSection", "QueueUserAPC"
    }
    print("\n[*] IMPORTS (suspicious highlighted):")
    if hasattr(pe, 'DIRECTORY_ENTRY_IMPORT'):
        for entry in pe.DIRECTORY_ENTRY_IMPORT:
            dll = entry.dll.decode(errors='replace')
            for imp in entry.imports:
                if imp.name:
                    fn = imp.name.decode(errors='replace')
                    if fn in SUSPICIOUS:
                        print(f"  ⚠️  {dll}!{fn}")

if __name__ == "__main__":
    if len(sys.argv) < 2:
        print("Usage: python pe_analyzer.py <binary.exe>")
    else:
        analyze_pe(sys.argv[1])
```

## 12.2 Process Token Inspector (C — Windows API)

```c
#include <windows.h>
#include <stdio.h>
#include <sddl.h>

/*
 * Inspect the access token of any process by PID.
 * Shows user, groups, privileges, and integrity level.
 * Usage: token_inspector.exe <PID>
 */

void PrintTokenInfo(HANDLE hToken) {
    DWORD dwSize = 0;
    TOKEN_USER *tokenUser = NULL;
    TOKEN_GROUPS *tokenGroups = NULL;
    TOKEN_PRIVILEGES *tokenPriv = NULL;
    
    // --- Get Token User ---
    GetTokenInformation(hToken, TokenUser, NULL, 0, &dwSize);
    tokenUser = (TOKEN_USER*)malloc(dwSize);
    if (GetTokenInformation(hToken, TokenUser, tokenUser, dwSize, &dwSize)) {
        LPSTR sidString;
        ConvertSidToStringSidA(tokenUser->User.Sid, &sidString);
        printf("[+] User SID: %s\n", sidString);
        LocalFree(sidString);
    }
    free(tokenUser);
    
    // --- Get Privileges ---
    GetTokenInformation(hToken, TokenPrivileges, NULL, 0, &dwSize);
    tokenPriv = (TOKEN_PRIVILEGES*)malloc(dwSize);
    if (GetTokenInformation(hToken, TokenPrivileges, tokenPriv, dwSize, &dwSize)) {
        printf("[+] Privileges (%d):\n", tokenPriv->PrivilegeCount);
        for (DWORD i = 0; i < tokenPriv->PrivilegeCount; i++) {
            char privName[256]; DWORD nameLen = 256;
            LookupPrivilegeNameA(NULL, &tokenPriv->Privileges[i].Luid, privName, &nameLen);
            BOOL enabled = (tokenPriv->Privileges[i].Attributes & SE_PRIVILEGE_ENABLED);
            printf("  [%s] %s\n", enabled ? "ON " : "OFF", privName);
        }
    }
    free(tokenPriv);
    
    // --- Integrity Level ---
    TOKEN_MANDATORY_LABEL *pTIL = NULL;
    GetTokenInformation(hToken, TokenIntegrityLevel, NULL, 0, &dwSize);
    pTIL = (TOKEN_MANDATORY_LABEL*)malloc(dwSize);
    if (GetTokenInformation(hToken, TokenIntegrityLevel, pTIL, dwSize, &dwSize)) {
        DWORD intLevel = *GetSidSubAuthority(pTIL->Label.Sid, 
                         *GetSidSubAuthorityCount(pTIL->Label.Sid) - 1);
        const char* level = "Unknown";
        if      (intLevel >= 0x4000) level = "SYSTEM";
        else if (intLevel >= 0x3000) level = "HIGH";
        else if (intLevel >= 0x2000) level = "MEDIUM";
        else if (intLevel >= 0x1000) level = "LOW";
        else                         level = "UNTRUSTED";
        printf("[+] Integrity Level: %s (0x%04X)\n", level, intLevel);
    }
    free(pTIL);
}

int main(int argc, char* argv[]) {
    if (argc < 2) { printf("Usage: %s <PID>\n", argv[0]); return 1; }
    
    DWORD pid = atoi(argv[1]);
    HANDLE hProcess = OpenProcess(PROCESS_QUERY_INFORMATION, FALSE, pid);
    if (!hProcess) {
        printf("[-] OpenProcess failed: %d\n", GetLastError());
        return 1;
    }
    
    HANDLE hToken;
    if (!OpenProcessToken(hProcess, TOKEN_QUERY, &hToken)) {
        printf("[-] OpenProcessToken failed: %d\n", GetLastError());
        CloseHandle(hProcess);
        return 1;
    }
    
    printf("[*] Token info for PID %d:\n", pid);
    PrintTokenInfo(hToken);
    
    CloseHandle(hToken);
    CloseHandle(hProcess);
    return 0;
}
```

---

# 13. ASCII Architecture Diagrams

## 13.1 PE File Memory Mapping Diagram

```
PE FILE — FROM DISK TO MEMORY:

    ON DISK (File)                       IN MEMORY (Process VAS)
    ─────────────────────────────────    ────────────────────────
    ┌──────────────────────────────┐
    │ DOS Header  [0x00 - 0x3F]   │ →  ┌──────────────────────────┐ ImageBase
    │ "MZ" (4D 5A)                │    │ DOS Header               │ 0x400000
    ├──────────────────────────────┤    ├──────────────────────────┤
    │ DOS Stub                     │ →  │ DOS Stub                 │
    ├──────────────────────────────┤    ├──────────────────────────┤
    │ NT Headers                   │ →  │ NT Headers               │
    │  "PE\0\0"                    │    │  EntryPoint              │
    │  Machine: 8664 (x64)         │    │  ImageBase               │
    │  EntryPoint RVA: 0x1000      │    │  DataDirectory[16]       │
    │  ImageBase: 0x140000000      │    │                          │
    │  DataDirectory[Import]=0x..  │    │                          │
    ├──────────────────────────────┤    ├──────────────────────────┤
    │ Section Headers              │ →  │ Section Headers          │
    │  .text  VA:0x1000 VS:0x4200  │    │  (same in memory)        │
    │  .data  VA:0x6000 VS:0x800   │    │                          │
    │  .rdata VA:0x7000 VS:0x3000  │    │                          │
    ├──────────────────────────────┤    ├──────────────────────────┤ 0x401000
    │ .text (CODE)                 │ →  │ .text                    │ RX
    │ FileAlign: 512 bytes         │    │ SectionAlign: 4096 bytes │
    │ Raw: 512 bytes               │    │ Padded to: 4096 bytes    │
    ├──────────────────────────────┤    ├──────────────────────────┤ 0x402000
    │ .data                        │ →  │ .data                    │ RW
    │                              │    │                          │
    ├──────────────────────────────┤    ├──────────────────────────┤ 0x403000
    │ .rdata (READ-ONLY DATA)      │ →  │ .rdata                   │ R
    │ Contains: IAT, strings       │    │ IAT filled by loader     │
    └──────────────────────────────┘    └──────────────────────────┘
```

## 13.2 Windows Security Architecture — Attacker View

```
WINDOWS SECURITY LAYERS — ATTACKER PERSPECTIVE:

 DEFENSE                                   BYPASS
 ──────────────────────────────────────────────────────────────────
 UEFI Secure Boot     ←───────────── UEFI Bootkit (BlackLotus)
 Driver Signature     ←───────────── BYOVD (vulnerable signed driver)
 Enforcement (DSE)
 Kernel Callbacks     ←───────────── Patch callback table / remove pointer
 (EDR kernel)
 ntdll.dll hooks      ←───────────── Direct syscall / indirect syscall
 (EDR user mode)
 AMSI (Antimalware    ←───────────── Patch AmsiScanBuffer() to return 0
 Scan Interface)
 PowerShell           ←───────────── Constrained Language Mode bypass
 Constrained Mode                    Reflection, COM objects
 UAC (User Account    ←───────────── Token impersonation, auto-elevated COM
 Control)
 AppLocker / SRP      ←───────────── LOLBins (regsvr32, rundll32, mshta)
 Defender             ←───────────── Obfuscation, encryption, LOLBins
 Windows Firewall     ←───────────── C2 over HTTPS (port 443, allowed)
 Credential Guard     ←───────────── SSP injection (older), DPAPI, DCSync
```

---

# 14. Attack & Defense Mapping — Per Layer

## 14.1 Complete Attack Surface Table

| Layer | What Attackers Target | Attack Technique | Defense |
|-------|----------------------|-----------------|---------|
| **UEFI** | Firmware persistence | UEFI bootkits (BlackLotus) | Secure Boot, UEFI integrity monitoring |
| **Boot** | Boot chain hijack | BCD modification, MBR overwrite | BitLocker + Secure Boot, measured boot |
| **Drivers (Ring 0)** | Kernel-level persistence/evasion | BYOVD, kernel exploits, DKOM | HVCI, DSE, kernel telemetry |
| **ntdll.dll** | EDR bypass | Direct/indirect syscalls, unhooking | Kernel callbacks (cannot be bypassed via syscalls) |
| **Process Creation** | Spawn malicious processes | cmd.exe, powershell.exe from Office | Process creation monitoring (Event 4688), EDR behavioral |
| **Memory (VAS)** | Code injection | VirtualAllocEx + WriteProcessMemory + CreateRemoteThread | Memory scanning, no-execute (DEP), CFG |
| **Token** | Privilege escalation | Token impersonation, token theft | Privilege Use auditing (Event 4673), PAM |
| **PE Format** | Packing/obfuscation | UPX, custom packers, RWX sections | Static AV, entropy analysis, sandboxing |
| **Registry** | Persistence | Run keys, Services, AppInit_DLLs | Registry auditing (Event 4657), baseline diff |
| **File System** | Payload dropping | Drop to %TEMP%, %AppData%, $MFT manipulation | File integrity monitoring (FIM), application control |

---

# 15. Interview Questions & Checkpoints

## Beginner Level

**Q1: What is the difference between user mode and kernel mode?**
> User mode (Ring 3) is a restricted environment where user applications run. Programs cannot directly access hardware or other processes' memory. Kernel mode (Ring 0) is the privileged environment where the Windows kernel and drivers run — code here has full hardware access and no memory restrictions. The transition between them is controlled via the SYSCALL CPU instruction.

**Q2: What does ASLR do?**
> Address Space Layout Randomization randomizes the base address where modules (EXE, DLLs) are loaded into a process's virtual memory each time. This prevents attackers from using hardcoded addresses in exploits, since the address of functions like `system()` or `VirtualAlloc()` changes every run. Bypassed via information leaks (infoleak vulnerabilities).

**Q3: What is LSASS and why is it targeted?**
> LSASS (Local Security Authority Subsystem Service) is the Windows process that handles authentication — it verifies credentials, creates logon sessions, and issues access tokens. It caches credential material in memory (NTLM hashes, Kerberos tickets, sometimes plaintext passwords). Attackers target it with tools like Mimikatz (`sekurlsa::logonpasswords`) to extract credentials for lateral movement.

## Intermediate Level

**Q4: Explain the Windows API call chain from user code to hardware.**
> A user program calls a Win32 function in kernel32.dll → which calls the Native API equivalent in ntdll.dll → which executes the SYSCALL instruction to transition to Ring 0 → the kernel's syscall handler dispatches to the appropriate kernel function in ntoskrnl.exe → which interacts with hardware via the HAL and device drivers → result returned via SYSRET back to user mode.

**Q5: What is DKOM and how do rootkits use it?**
> Direct Kernel Object Manipulation — attackers load a kernel driver and directly modify kernel data structures. Most commonly: unlinking a process's EPROCESS from the `ActiveProcessLinks` doubly-linked list hides it from API-based process enumeration (Task Manager, Process Explorer). The process continues running — only its list entry is removed. Detected by scanning physical memory for EPROCESS structures and comparing against the API list.

**Q6: What is a PE section anomaly that indicates malware?**
> Key anomalies: (1) RWX sections (Read-Write-Execute) — shellcode is written and executed in the same region. (2) VirtualSize >> SizeOfRawData — content is compressed/packed. (3) High entropy (>7.0) — content is encrypted or packed. (4) Unknown section names — custom packer. (5) Very few imports (only LoadLibrary + GetProcAddress) — all imports resolved dynamically (packer).

## Expert Level

**Q7: How do modern EDRs use kernel callbacks, and how do attackers bypass them?**
> EDRs register kernel callbacks via APIs like `PsSetCreateProcessNotifyRoutine` (called on every process creation), `PsSetCreateThreadNotifyRoutine`, and `ObRegisterCallbacks` (called on every object handle operation). These cannot be bypassed via direct syscalls because they run IN the kernel, not in user-mode hooks. Attackers bypass kernel callbacks by: (1) removing their callback registration from the kernel's callback array (requires Ring 0 access via BYOVD), (2) using process ghosting/hollowing to spawn processes in ways that may avoid certain callbacks, (3) calling kernel functions that predate certain callback mechanisms.

**Q8: Explain BYOVD and why it is significant.**
> Bring Your Own Vulnerable Driver — attackers load a legitimate, Microsoft-signed driver that contains a known vulnerability (e.g., a kernel memory read/write primitive). Since the driver is legitimately signed, DSE (Driver Signature Enforcement) allows it to load. The attacker then exploits the vulnerability IN the driver to gain arbitrary kernel read/write — effectively giving them Ring 0 access. Used by threat groups including Lazarus (BYOVD to disable EDR), ransomware groups (disable endpoint protection before deploying ransomware). Defense: HVCI (Hypervisor Protected Code Integrity) + vulnerable driver blocklist.

**Q9: What is Heaven's Gate and why do analysts need to know it?**
> On 64-bit Windows, 32-bit processes run in a compatibility layer (WoW64). Heaven's Gate is a technique where a 32-bit process performs a far jump from 32-bit code segment (CS=0x23) to 64-bit code segment (CS=0x33), switching to 64-bit execution mode within the same process. From there, it can make 64-bit syscalls directly. This bypasses: (1) 32-bit ntdll hooks (EDR often hooks only the 32-bit ntdll in WoW64 processes), (2) Tools that only inspect 32-bit code. Analysts must enable x64 inspection of WoW64 processes and watch for far jumps (0x33 segment selector).

**Q10: What is the difference between PRIMARY and IMPERSONATION tokens?**
> A **primary token** defines the security context of a process — it is assigned when a process starts and represents the process's "identity". An **impersonation token** is used by threads to temporarily assume a different user's identity for specific operations. A thread in a service process might impersonate a client to check if the client has access to a file — then revert to the service's primary token. Attackers abuse impersonation tokens: a compromised service (e.g., a network service with `SeImpersonatePrivilege`) can impersonate any user that connects to it — including SYSTEM, leading to privilege escalation via techniques like `PrintSpoofer` or `RoguePotato`.

---

## Summary Table — Architecture at a Glance

```
LAYER           WHAT LIVES THERE                    SECURITY RELEVANCE
──────────────────────────────────────────────────────────────────────────
Ring -2/-1      UEFI, SMM, Hypervisor              Bootkits, VM escapes
Ring 0          ntoskrnl, hal.dll, drivers          Rootkits, EDR kernel
──── BOUNDARY ─────────────────────────────────────────────────────────
Ring 3          ntdll (syscall stubs)              Direct syscall bypass
                kernel32, user32, advapi32         API hooking by EDR
                LSASS, CSRSS, services             Credential theft, pivot
                User applications                  Initial access, execution
──────────────────────────────────────────────────────────────────────────
PE Format       Headers, sections, IAT             Malware identification
Tokens          Primary + impersonation            Privesc, lateral movement
Virtual Memory  VAS, page tables, ASLR             Code injection, exploit
Boot Chain      UEFI → bootmgr → winload → kernel  Bootkit persistence layers
```

---

*Next: `02_PowerShell_Offensive_Defensive.md` — PowerShell as both attack tool and defensive platform, including AMSI, logging, constrained language mode, and offensive tradecraft.*
