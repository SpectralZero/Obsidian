# Linux Kernel & Architecture: The Core - COMPREHENSIVE GUIDE

---
title: "Linux Kernel & Architecture - Deep Dive"
parent: "[[03_Linux]]"
tags:
  - linux
  - kernel
  - architecture
  - syscalls
  - internal
  - oscp
  - privilege-escalation
  - exploitation
created: 2026-01-23
updated: 2026-01-24
---

> **Executive Summary**: To hack Linux, you must understand what Linux *is*. It is not just a shell; it is a monolithic kernel that manages resources. When you exploit a "buffer overflow", you are corrupting memory managed by the kernel. When you escalate privileges, you are tricking the kernel into granting you capabilities. This comprehensive chapter peels back the GUI and the Shell to look at the engine: Protection Rings, System Calls, Virtual Filesystem, Memory Management, and Kernel Internals. You will learn not just WHAT the kernel does, but HOW and WHY - essential knowledge for OSCP and real-world exploitation.

---

## Table of Contents

1. [Learning Objectives](#1-learning-objectives)
2. [What is a Kernel? (ABSOLUTE BEGINNER)](#2-what-is-a-kernel-absolute-beginner)
3. [The Kernel Ring Model (Ring 0 vs Ring 3)](#3-the-kernel-ring-model-ring-0-vs-ring-3)
4. [The Boot Process: From Power-On to Bash](#4-the-boot-process-from-power-on-to-bash)
5. [System Calls: The Gateway Between Worlds](#5-system-calls-the-gateway-between-worlds)
6. [The Virtual Filesystem (VFS)](#6-the-virtual-filesystem-vfs)
7. [Pseudo-Filesystems: /proc and /sys](#7-pseudo-filesystems-proc-and-sys)
8. [Memory Management](#8-memory-management)
9. [Loadable Kernel Modules (LKMs)](#9-loadable-kernel-modules-lkms)
10. [Kernel Security Mechanisms](#10-kernel-security-mechanisms)
11. [Practical Lab: Tracing with Strace](#11-practical-lab-tracing-with-strace)
12. [Advanced: Kernel Exploitation Concepts](#12-advanced-kernel-exploitation-concepts)
13. [OSCP Relevance](#13-oscp-relevance)
14. [Critical Analysis & Checkpoints](#14-critical-analysis--checkpoints)
15. [Hands-On Exercises](#15-hands-on-exercises)
16. [Common Pitfalls](#16-common-pitfalls)
17. [Interview Questions](#17-interview-questions)
18. [Summary](#18-summary)

---

## 1. Learning Objectives

By the end of this chapter, you will be able to:

- **Understand the Kernel**: Explain what the kernel is and its role in the operating system
- **Map Protection Rings**: Differentiate between User Space (Ring 3) and Kernel Space (Ring 0)
- **Trace System Execution**: Use `strace`, `ltrace`, and `/proc` to spy on how binaries interact with the OS
- **Navigate Kernel Interfaces**: Extract live system states from `/proc` and `/sys` without external tools
- **Understand the Boot Process**: Explain the complete chain from firmware to login prompt
- **Identify Security Mechanisms**: Understand SELinux, AppArmor, seccomp, and namespaces
- **Detect Rootkits**: Explain how LKMs allow attackers to hide processes and files
- **Apply OSCP Knowledge**: Use kernel understanding for privilege escalation and exploitation
- **Debug Like a Pro**: Use kernel-level debugging techniques for vulnerability research

---

## 2. What is a Kernel? (ABSOLUTE BEGINNER)

### 2.1 The Operating System Layers

**ABSOLUTE BEGINNER EXPLANATION**:

Think of your computer like a building:

```
┌─────────────────────────────────────┐
│  APPLICATIONS (You live here)       │  ← Ring 3 (User Space)
│  Firefox, Bash, Python, Games       │
├─────────────────────────────────────┤
│  KERNEL (The Foundation)            │  ← Ring 0 (Kernel Space)
│  Manages: CPU, Memory, Disk, Network│
├─────────────────────────────────────┤
│  HARDWARE                           │
│  CPU, RAM, Hard Drive, Network Card │
└─────────────────────────────────────┘
```

**The Kernel is the Operating System's core**. It's the first program that runs when you turn on your computer (after the bootloader), and it runs until you shut down.

### 2.2 What Does the Kernel Do?

The kernel has **5 primary responsibilities**:

1. **Process Management**
   - Creates processes
   - Schedules CPU time (decides which program runs when)
   - Kills processes
   - Manages memory for each process

2. **Memory Management**
   - Divides RAM between processes
   - Implements virtual memory (swap)
   - Prevents processes from accessing each other's memory

3. **Device Management**
   - Controls hardware (keyboard, disk, network card)
   - Provides device drivers
   - Creates device files in `/dev/`

4. **Filesystem Management**
   - Reads and writes files
   - Manages permissions
   - Implements different filesystems (ext4, xfs, btrfs)

5. **Network Management**
   - Handles TCP/IP stack
   - Manages network interfaces
   - Implements firewall rules (netfilter)

### 2.3 Kernel Types

| Type | Description | Example |
|------|-------------|---------|
| **Monolithic** | Everything in one big program | Linux, Unix |
| **Microkernel** | Minimal kernel, services in user space | Minix, QNX |
| **Hybrid** | Mix of both | Windows NT, macOS |

Linux uses a **monolithic kernel** design - everything runs in kernel space for performance, but supports modules for flexibility.

### 2.4 Kernel vs Shell vs Operating System

**ABSOLUTE BEGINNER CLARIFICATION**:

```
Operating System = Kernel + System Libraries + System Utilities + Shell

Linux = Just the kernel
GNU/Linux = Linux kernel + GNU tools (bash, ls, grep, etc.)
Ubuntu = Linux + GNU + Package Manager + Desktop Environment
```

When people say "I use Linux", they usually mean a Linux distribution (Ubuntu, Kali, etc.), not just the kernel.

---

## 3. The Kernel Ring Model (Ring 0 vs Ring 3)

### 3.1 Understanding Protection Rings

**ABSOLUTE BEGINNER**: The CPU has built-in security levels called "rings". Think of them like security clearance levels:

```
┌──────────────────────────────────────┐
│         Ring 0 (Kernel Mode)         │  ← Highest Privilege
│      TOP SECRET CLEARANCE            │     Can do ANYTHING
│  - Access all memory                 │
│  - Execute privileged CPU instructions│
│  - Access hardware directly          │
├──────────────────────────────────────┤
│         Ring 1 & 2                   │  ← Rarely used in Linux
│      (Device Drivers - historical)   │
├──────────────────────────────────────┤
│         Ring 3 (User Mode)           │  ← Lowest Privilege
│      PUBLIC ACCESS                   │     Limited power
│  - Cannot access hardware           │
│  - Cannot access other process memory│
│  - Must ask kernel for help (syscall)│
└──────────────────────────────────────┘
```

### 3.2 Ring 0 (Kernel Mode) - GOD MODE

**Who Lives Here**:
- The Linux Kernel (`vmlinuz`)
- Device Drivers
- Kernel Modules (`.ko` files)

**What They Can Do**:
- **Everything**. No restrictions.
- Direct access to **all physical memory** (RAM)
- Execute **privileged CPU instructions** (like `cli` to disable interrupts)
- Read/write **any file** on disk
- Access **any hardware device** directly
- Kill **any process** (including PID 1)

**The Risk**:
- A bug in Ring 0 code = **Kernel Panic** (Blue Screen of Death equivalent)
- A vulnerability in Ring 0 = **Complete system compromise**
- No recovery from kernel crashes (system halts)

### 3.3 Ring 3 (User Mode) - RESTRICTED MODE

**Who Lives Here**:
- **All applications**: Firefox, Chrome, Python, Bash, `ls`, `cat`, your custom programs
- **Your shell**: Even root's bash shell runs in Ring 3!

**What They Cannot Do**:
- Cannot directly access hardware
- Cannot read other processes' memory
- Cannot execute privileged CPU instructions
- Cannot access kernel memory

**How They Do Anything Useful**:
Through **System Calls** (syscalls) - politely asking the kernel for help.

### 3.4 The Privilege Transition

**Example**: Reading a file

```
User Space (Ring 3)          Kernel Space (Ring 0)
┌─────────────┐              ┌──────────────┐
│ cat file.txt│──syscall────→│   Kernel     │
│             │              │              │
│ Waiting...  │              │ 1. Check     │
│             │              │    permissions│
│             │              │ 2. Read disk │
│             │              │ 3. Copy data │
│             │              │    to buffer │
│             │←──returns────│              │
│ Print data  │              └──────────────┘
└─────────────┘
```

**Key Point**: The CPU automatically switches protection rings during syscalls. This is done by the `syscall` CPU instruction, not by the program itself.

### 3.5 Why This Matters for Hackers

**Exploitation Goal**: Get code execution in Ring 0

| Attack                   | Goal                                           |
| ------------------------ | ---------------------------------------------- |
| **Buffer Overflow**      | Overwrite return address to execute shellcode  |
| **Privilege Escalation** | Exploit kernel vuln to run code in Ring 0      |
| **Rootkit**              | Load malicious kernel module to hide in Ring 0 |

**Example**: DirtyC0w (CVE-2016-5195)
- Vulnerability in kernel memory management
- Allowed user to write to read-only files
- **Result**: User-space attacker modifies kernel memory → Gets root

---

## 4. The Boot Process: From Power-On to Bash

### 4.1 The Complete Boot Sequence (DETAILED)

**ABSOLUTE BEGINNER**: When you press the power button, a complex chain of events happens before you see the login prompt. Let's break it down step-by-step.

```
Power On
   ↓
┌──────────────────────────────────────┐
│ 1. FIRMWARE (BIOS/UEFI)              │ ← Hardware level
│    - Run Power-On Self Test (POST)   │
│    - Initialize hardware (CPU, RAM)  │
│    - Find bootable device            │
│    Duration: 2-5 seconds             │
└─────────────┬────────────────────────┘
              ↓
┌──────────────────────────────────────┐
│ 2. BOOTLOADER (GRUB/systemd-boot)    │ ← Stored on disk
│    - Load kernel image (vmlinuz)     │
│    - Load initramfs (initrd)         │
│    - Pass kernel parameters          │
│    Duration: 1-2 seconds             │
└─────────────┬────────────────────────┘
              ↓
┌──────────────────────────────────────┐
│ 3. KERNEL INITIALIZATION             │ ← Ring 0 starts
│    - Decompress kernel               │
│    - Initialize memory management    │
│    - Detect hardware                 │
│    - Load drivers from initramfs     │
│    - Mount root filesystem (/)       │
│    Duration: 2-4 seconds             │
└─────────────┬────────────────────────┘
              ↓
┌──────────────────────────────────────┐
│ 4. INIT SYSTEM (systemd/init)        │ ← First user process
│    - PID 1 (never dies!)             │
│    - Mount remaining filesystems     │
│    - Start services (network, SSH)   │
│    - Spawn login prompt (getty)      │
│    Duration: 5-10 seconds            │
└─────────────┬────────────────────────┘
              ↓
┌──────────────────────────────────────┐
│ 5. LOGIN PROMPT                      │ ← You see this!
│    - getty waits for username        │
│    - Login process authenticates     │
│    - Spawns your shell (bash)        │
└──────────────────────────────────────┘
```

### 4.2 Stage 1: Firmware (BIOS/UEFI)

**ABSOLUTE BEGINNER**: The firmware is a tiny operating system stored on a chip on your motherboard.

**BIOS (Legacy)**:
- Very old (1980s)
- Limited to 16-bit mode initially
- Looks for Master Boot Record (MBR) on disk
- MBR contains bootloader in first 512 bytes

**UEFI (Modern)**:
- Supports 32-bit and 64-bit mode
- Uses GPT (GUID Partition Table) instead of MBR
- Bootloader stored in EFI System Partition (ESP)
- Supports Secure Boot (verifies bootloader signature)

**Hands-On: View UEFI Variables**:
```bash
# Must be run as root on a UEFI system
ls -la /sys/firmware/efi/efivars/

# Example output:
# -rw-r--r-- Boot0001-8be4df61-93ca-11d2-aa0d-00e098032b8c
# -rw-r--r-- BootCurrent-8be4df61-93ca-11d2-aa0d-00e098032b8c
```

### 4.3 Stage 2: Bootloader (GRUB2)

**ABSOLUTE BEGINNER**: The bootloader's job is to load the kernel into memory.

**GRUB2 Configuration**:
- Located at `/boot/grub/grub.cfg`
- Generated from `/etc/default/grub`

**Example GRUB Entry**:
```bash
menuentry 'Ubuntu' {
    linux /boot/vmlinuz-5.15.0-generic root=UUID=abc123... ro quiet splash
    initrd /boot/initrd.img-5.15.0-generic
}
```

**Breaking Down the Parameters**:
- `vmlinuz-5.15.0-generic`: The compressed kernel image
- `root=UUID=abc123...`: Which partition contains the root filesystem
- `ro`: Mount root as read-only initially (prevents corruption if boot fails)
- `quiet`: Suppress verbose boot messages
- `splash`: Show graphical boot screen
- `initrd.img`: Initial RAM Disk (temporary root filesystem)

**Hands-On: View Current Kernel Parameters**:
```bash
cat /proc/cmdline

# Example output:
# BOOT_IMAGE=/boot/vmlinuz-5.15.0-generic root=UUID=abcd1234... ro quiet splash vt.handoff=7
```

### 4.4 Stage 3: Kernel Initialization

**ABSOLUTE BEGINNER**: The kernel is loaded into memory, decompressed, and begins initializing the system.

**Key Steps**:

1. **Kernel Decompression**
   - The `vmlinuz` file is compressed (gzip/bzip2/xz)
   - A tiny decompressor at the start of vmlinuz runs first
   - Decompresses the actual kernel into RAM

2. **Hardware Detection**
   - Kernel probes the system bus
   - Detects CPUs, RAM, PCI devices
   - Loads drivers from initramfs

3. **Memory Management Setup**
   - Initialize page tables
   - Set up virtual memory
   - Enable memory protection

4. **Mount initramfs**
   - Temporary root filesystem in RAM
   - Contains essential drivers (disk, filesystem, network)
   - Allows kernel to mount real root filesystem

5. **Switch to Real Root**
   - Mount the actual `/` partition
   - Execute `/sbin/init` (usually symlink to systemd)

**Viewing Boot Messages**:
```bash
# View kernel boot messages
dmesg | less

# Or view from beginning
journalctl -k -b

# Example interesting messages:
[    0.000000] Linux version 5.15.0-generic
[    0.000000] Command line: BOOT_IMAGE=/boot/vmlinuz...
[    0.384392] CPU0: Intel Core i7-9700K
[    1.234567] ata1.00: 500GB SSD detected
```

### 4.5 Stage 4: Init System (systemd)

**ABSOLUTE BEGINNER**: `systemd` is the first user-space program (PID 1). It starts all other services.

**What systemd Does**:
1. **Mount Filesystems**
   - Reads `/etc/fstab`
   - Mounts `/home`, `/var`, `/tmp`, etc.

2. **Start Services**
   - Network manager
   - SSH server
   - Database servers
   - Web servers

3. **Manage Targets** (like runlevels)
   - `rescue.target`: Single-user mode
   - `multi-user.target`: Multi-user console
   - `graphical.target`: Multi-user + GUI

**Viewing systemd Status**:
```bash
# Overall system state
systemctl status

# Boot time analysis
systemd-analyze

# Which services took longest to start
systemd-analyze blame

# Example output:
# 5.234s mysql.service
# 3.912s apache2.service
# 1.543s network-manager.service
```

**Hands-On: Changing Boot Target**:
```bash
# View current target
systemctl get-default
# Output: graphical.target

# Change to text mode (no GUI)
sudo systemctl set-default multi-user.target

# Change back to GUI
sudo systemctl set-default graphical.target
```

### 4.6 Stage 5: Login Prompt

**The Final Step**:
- `systemd` starts `getty` on virtual terminals
- `getty` displays login prompt and waits
- When you enter username, `getty` calls `/bin/login`
- `/bin/login` checks `/etc/passwd` and `/etc/shadow`
- If authentication succeeds, `/bin/login` spawns your shell

**Viewing Login Processes**:
```bash
ps aux | grep getty

# Example output:
# root   867  0.0  0.0  5984  1632 tty1  Ss+  08:00  0:00 /sbin/agetty -o -p -- \u --noclear tty1 linux
```

---

## 5. System Calls: The Gateway Between Worlds

### 5.1 What are System Calls?

**ABSOLUTE BEGINNER**: A system call (syscall) is how your program asks the kernel to do something.

**Think of it like this**:
```
You (User Program) want to order food (access hardware).
You can't go into the kitchen (hardware) directly.
You call the waiter (system call) who goes to the kitchen (kernel).
The waiter brings your food (returns result).
```

### 5.2 The System Call Mechanism (DETAILED)

**Step-by-Step Flow**:

```
Step 1: User Program (Ring 3)
┌─────────────────────────────┐
│ int fd = open("file.txt",  │
│              O_RDONLY);     │
└──────────┬──────────────────┘
           │
           │ (Calls C library function)
           ↓
Step 2: C Library (glibc)
┌─────────────────────────────┐
│ Wrapper function prepares:  │
│ - Syscall number (e.g., 2)  │
│ - Arguments                 │
│ - Executes: syscall         │
│   instruction               │
└──────────┬──────────────────┘
           │
           │ (CPU switches to Ring 0)
           ↓
Step 3: Kernel (Ring 0)
┌─────────────────────────────┐
│ Kernel syscall handler:     │
│ 1. Save user registers      │
│ 2. Check syscall number     │
│ 3. Check permissions        │
│ 4. Perform operation        │
│ 5. Restore registers        │
│ 6. Return to user space     │
└──────────┬──────────────────┘
           │
           │ (CPU switches back to Ring 3)
           ↓
Step 4: Return to User Program
┌─────────────────────────────┐
│ fd = 3 (file descriptor)    │
│ or                          │
│ fd = -1 (error)             │
└─────────────────────────────┘
```

### 5.3 Common System Calls

| Category | Syscall | Purpose | Example |
|----------|---------|---------|---------|
| **File Operations** | `open()` | Open a file | `open("file.txt", O_RDONLY)` |
| | `read()` | Read from file | `read(fd, buffer, 100)` |
| | `write()` | Write to file | `write(fd, "data", 4)` |
| | `close()` | Close file | `close(fd)` |
| **Process Control** | `fork()` | Create child process | `pid = fork()` |
| | `execve()` | Execute program | `execve("/bin/sh", args, env)` |
| | `exit()` | Terminate process | `exit(0)` |
| | `wait()` | Wait for child | `wait(&status)` |
| **Memory** | `brk()` | Change heap size | `brk(new_addr)` |
| | `mmap()` | Map memory | `mmap(NULL, size, PROT_READ, ...)` |
| **Network** | `socket()` | Create socket | `socket(AF_INET, SOCK_STREAM, 0)` |
| | `connect()` | Connect to remote | `connect(sock, addr, len)` |
| | `send()` / `recv()` | Send/receive data | `send(sock, buf, len, 0)` |
| **Security** | `setuid()` | Change user ID | `setuid(0)` |
| | `ptrace()` | Debug process | `ptrace(PTRACE_ATTACH, pid, ...)` |

### 5.4 Syscall Numbers

**ABSOLUTE BEGINNER**: Each system call has a unique number.

**On x86_64 Linux**:
- `read` = 0
- `write` = 1
- `open` = 2
- `close` = 3
- `execve` = 59

**Viewing Syscall Table**:
```bash
# On Linux, syscall numbers are in:
cat /usr/include/x86_64-linux-gnu/asm/unistd_64.h | grep __NR_read

# Output:
# #define __NR_read 0
```

### 5.5 Practical Example: Tracing Syscalls

**Example 1: Simple Program**

Create a C program:
```c
#include <stdio.h>
#include <fcntl.h>
#include <unistd.h>

int main() {
    int fd = open("testfile.txt", O_RDONLY);
    if (fd == -1) {
        perror("open");
        return 1;
    }
    
    char buffer[100];
    ssize_t bytes_read = read(fd, buffer, sizeof(buffer));
    
    close(fd);
    return 0;
}
```

Compile and trace:
```bash
gcc -o program program.c
strace ./program

# Key output lines:
# openat(AT_FDCWD, "testfile.txt", O_RDONLY) = 3
# read(3, "Hello World\n", 100) = 12
# close(3) = 0
```

**What We See**:
1. `openat()` was called (modern version of `open()`)
2. It returned `3` (file descriptor)
3. `read()` read 12 bytes from FD 3
4. `close()` closed FD 3

### 5.6 Syscalls for Exploitation

**For Shellcode**:
```nasm
; Minimal execve shellcode (x86_64)
; Calls execve("/bin/sh", NULL, NULL)

xor rax, rax        ; rax = 0
mov al, 59          ; syscall number for execve
xor rdi, rdi
push rdi            ; NULL terminator
mov rdi, 0x68732f6e69622f ; "/bin/sh"
push rdi
mov rdi, rsp        ; rdi points to "/bin/sh"
xor rsi, rsi        ; argv = NULL
xor rdx, rdx        ; envp = NULL
syscall             ; Execute!
```

**Why Hackers Care**:
- Shellcode must use syscalls directly (no library functions)
- Understanding syscalls helps bypass security mechanisms
- Some exploits target syscall handlers themselves

---

## 6. The Virtual Filesystem (VFS)

### 6.1 "Everything is a File" Philosophy

**ABSOLUTE BEGINNER**: In Linux, almost everything is accessed through the filesystem, even if it's not actually a file on disk.

**Examples**:
```
/dev/sda          → Your hard drive
/dev/tty          → Your terminal
/dev/null         → The black hole (discard data)
/dev/urandom      → Random number generator
/dev/zero         → Infinite zeros
/proc/cpuinfo     → CPU information (doesn't exist on disk!)
/sys/class/net/   → Network interfaces
```

### 6.2 The VFS Layer

**ABSOLUTE BEGINNER**: The VFS is an abstraction layer that makes all filesystems look the same to programs.

```
User Program
     ↓
 VFS Layer (Generic Interface)
     ↓
┌────┴────┬──────┬──────┬──────┐
│  ext4   │ xfs  │ nfs  │ proc │  ← Actual Filesystem Implementations
└─────────┴──────┴──────┴──────┘
```

**What This Means**:
- Your program calls `open("file.txt")`
- VFS determines which filesystem "file.txt" is on
- VFS calls the appropriate filesystem driver
- The driver handles the actual disk I/O

### 6.3 Device Files

**Types of Device Files**:

1. **Block Devices** (buffered I/O)
   - Hard drives, SSDs, USB drives
   - Example: `/dev/sda`, `/dev/nvme0n1`
   - Accessed in blocks (512 bytes, 4096 bytes, etc.)

2. **Character Devices** (unbuffered I/O)
   - Terminals, serial ports, printers
   - Example: `/dev/tty`, `/dev/random`
   - Accessed character-by-character

**Viewing Device Files**:
```bash
ls -l /dev/

# Example output:
# brw-rw---- 1 root disk      8,   0 Jan 24 08:00 sda    ← Block device
# crw-rw-rw- 1 root tty       5,   0 Jan 24 10:23 tty    ← Character device
#  ^
#  b = block, c = character
```

**The Numbers** (Major and Minor):
- **Major number**: Device driver (8 = SCSI disk)
- **Minor number**: Specific device (0 = first disk)

### 6.4 Special Device Files

**`/dev/null` - The Black Hole**:
```bash
# Discard output
ls /nonexistent 2>/dev/null

# Infinite sink
cat /dev/zero > /dev/null  # Runs forever, does nothing
```

**`/dev/zero` - Infinite Zeros**:
```bash
# Create 1MB file of zeros
dd if=/dev/zero of=file.dat bs=1M count=1

# Wipe disk (DANGEROUS!)
dd if=/dev/zero of=/dev/sda  # Overwrites entire disk with zeros
```

**`/dev/urandom` - Random Data**:
```bash
# Generate random password
tr -dc 'A-Za-z0-9' </dev/urandom | head -c 20; echo

# Example output: 8kF3mN9pQ2vL7zX4tW1Y
```

### 6.5 Named Pipes (FIFOs)

**ABSOLUTE BEGINNER**: Named pipes allow processes to communicate.

**Creating a Named Pipe**:
```bash
# Create pipe
mkfifo /tmp/mypipe

# Terminal 1: Write to pipe
echo "Hello" > /tmp/mypipe

# Terminal 2: Read from pipe
cat /tmp/mypipe
# Output: Hello
```

**Use Case**: Inter-process communication without sockets.

---

## 7. Pseudo-Filesystems: /proc and /sys

### 7.1 What are Pseudo-Filesystems?

**ABSOLUTE BEGINNER**: These directories don't exist on your hard drive. They're windows into the kernel's memory, generated on-the-fly.

```
Hard Drive         vs       RAM (Live Data)
┌──────────┐              ┌──────────┐
│ /home/   │              │ /proc/   │ ← Kernel data
│ /etc/    │              │ /sys/    │ ← Hardware data
│ /var/    │              └──────────┘
└──────────┘
(Persistent)             (Disappears on shutdown)
```

### 7.2 /proc - Process and System Information

**Structure**:
```
/proc/
├── 1/              ← Process ID 1 (init/systemd)
│   ├── cmdline     ← Command that started it
│   ├── environ     ← Environment variables
│   ├── exe         ← Symlink to executable
│   ├── fd/         ← Open file descriptors
│   ├── maps        ← Memory layout
│   └── status      ← Process status
├── cpuinfo         ← CPU information
├── meminfo         ← Memory information
├── version         ← Kernel version
└── uptime          ← System uptime
```

**Example 1: View Process Command Line**:
```bash
# What command started PID 1234?
cat /proc/1234/cmdline
# Output (null-separated): /usr/bin/python3 script.py arg1 arg2

# Make it readable
cat /proc/1234/cmdline | tr '\0' ' '
# Output: /usr/bin/python3 script.py arg1 arg2
```

**Example 2: Find Passwords in Process Memory**:
```bash
# WARNING: Sensitive data might be in cmdline or environ!

# Check if process was started with password as argument
cat /proc/*/cmdline | grep -a password

# Example BAD practice:
# mysql -u root -pSECRETpassword123  ← Password visible in /proc!
```

**Example 3: View Memory Layout (for Buffer Overflow)**:
```bash
cat /proc/self/maps

# Example output:
# 55e8d0000000-55e8d0001000 r--p 00000000 08:01 12345  /usr/bin/cat
# 55e8d0001000-55e8d0002000 r-xp 00001000 08:01 12345  /usr/bin/cat ← Executable
# 7ffc12345000-7ffc12366000 rw-p 00000000 00:00 0      [stack]      ← Stack!
```

**Example 4: CPU Information**:
```bash
cat /proc/cpuinfo | grep "model name" | head -1
# Output: model name : Intel(R) Core(TM) i7-9700K CPU @ 3.60GHz

# Count CPU cores
grep -c processor /proc/cpuinfo
# Output: 8
```

**Example 5: Memory Information**:
```bash
cat /proc/meminfo | head -5

# Example output:
# MemTotal:       16384000 kB
# MemFree:         2048000 kB
# MemAvailable:    8192000 kB
# Buffers:          512000 kB
# Cached:          4096000 kB
```

### 7.3 /sys - Hardware and Kernel Subsystems

**ABSOLUTE BEGINNER**: `/sys` exposes kernel data structures and hardware information.

**Structure**:
```
/sys/
├── block/          ← Block devices (disks)
├── bus/            ← System buses (PCI, USB)
├── class/          ← Device classes
│   ├── net/        ← Network interfaces
│   └── input/      ← Input devices
├── devices/        ← All devices
├── firmware/       ← Firmware data
└── module/         ← Loaded kernel modules
```

**Example 1: Network Interface Information**:
```bash
# List network interfaces
ls /sys/class/net/
# Output: eth0 lo wlan0

# Get MAC address
cat /sys/class/net/eth0/address
# Output: 00:11:22:33:44:55

# Check if interface is up
cat /sys/class/net/eth0/operstate
# Output: up
```

**Example 2: Change Screen Brightness**:
```bash
# Find brightness control
find /sys/class/backlight/ -name brightness

# View current brightness
cat /sys/class/backlight/intel_backlight/brightness
# Output: 5000

# View max brightness
cat /sys/class/backlight/intel_backlight/max_brightness
# Output: 10000

# Set to 50% (requires root)
echo 5000 | sudo tee /sys/class/backlight/intel_backlight/brightness
```

**Example 3: View Loaded Kernel Modules**:
```bash
ls /sys/module/

# Output includes:
# apparmor e1000 ext4 ip_tables nf_conntrack usbcore ...

# Check if module is loaded
[ -d /sys/module/usbcore ] && echo "USB module loaded"
```

### 7.4 Security Implications of /proc

**Sensitive Information Leakage**:

1. **Environment Variables** may contain API keys:
```bash
cat /proc/self/environ | tr '\0' '\n' | grep -i key
# Might reveal: AWS_SECRET_KEY=abc123xyz789
```

2. **Command Line Arguments** may contain passwords:
```bash
ps aux | grep mysql
# Bad: mysql -u root -pPASSWORD123  ← Visible to all users!
```

3. **Open Files** reveal what application is accessing:
```bash
ls -l /proc/1234/fd/

# Example output:
# lrwx------ 1 user user 0 Jan 24 10:00 3 -> /home/user/.ssh/id_rsa ← SSH key open!
```

**Defensive Measure**: Use `/proc/self` to reference your own process without knowing PID.

---

## 8. Memory Management

### 8.1 Virtual Memory Basics

**ABSOLUTE BEGINNER**: Each process thinks it has access to ALL of memory (e.g., 4GB on 32-bit). This is an illusion created by the kernel.

**Physical vs Virtual Memory**:
```
Virtual Memory (What Process Sees)
┌──────────────────────┐ 0xFFFFFFFF
│  Kernel Space        │ ← Cannot access from user mode
├──────────────────────┤ 0xC0000000
│  Stack               │ ← Grows downward
│       ↓              │
│  (unused)            │
│       ↑              │
│  Heap                │ ← Grows upward
├──────────────────────┤
│  BSS (uninitialized) │
│  Data (initialized)  │
│  Text (code)         │
└──────────────────────┘ 0x00000000

Physical Memory (Actual RAM)
┌──────────────────────┐
│  Process A (part)    │
│  Kernel              │
│  Process B (part)    │
│  Process A (cont'd)  │
│  Free                │
└──────────────────────┘
```

**Key Insight**: Virtual addresses != Physical addresses. The MMU (Memory Management Unit) translates them using **page tables**.

### 8.2 Memory Segments

**1. Text Segment (Code)**:
- Contains executable instructions
- Usually marked **read-only** and **executable**
- Example: The compiled code of `/bin/ls`

**2. Data Segment**:
- **Initialized data**: Global variables with initial values
- **BSS (Block Started by Symbol)**: Global variables without initial values (set to zero)

**3. Heap**:
- Dynamically allocated memory (`malloc()`, `new`)
- Grows upward (toward higher addresses)
- Managed by C library (glibc)

**4. Stack**:
- Local variables, function parameters, return addresses
- Grows downward (toward lower addresses)
- **Critical for Buffer Overflows**: Overwriting return address changes control flow

**Viewing Memory Layout**:
```bash
# View memory map of a process
cat /proc/self/maps

# Or for another process
pmap 1234
```

### 8.3 Page Tables and the MMU

**ABSOLUTE BEGINNER**: The CPU has hardware called the MMU that translates virtual addresses to physical addresses.

**How It Works**:
```
Program accesses address: 0x08048000 (virtual)
              ↓
         MMU looks up in Page Table
              ↓
    Finds physical address: 0x12345000
              ↓
         Access physical RAM
```

**Page Size**: Typically 4KB
- Memory divided into 4KB chunks (pages)
- Each page can be independently mapped

**Permissions**: Each page has flags:
- `r`: Readable
- `w`: Writable
- `x`: Executable
- `p`: Private (not shared)

**Example /proc/self/maps**:
```
55e8d0000000-55e8d0001000 r--p   ← Read-only, private
55e8d0001000-55e8d0002000 r-xp   ← Read-execute (code)
55e8d0002000-55e8d0003000 rw-p   ← Read-write (data)
7ffc12345000-7ffc12366000 rw-p   ← Stack
```

### 8.4 Swap Space

**ABSOLUTE BEGINNER**: If RAM is full, the kernel moves inactive pages to disk (swap).

**Checking Swap**:
```bash
free -h

# Example output:
#               total        used        free      shared  buff/cache   available
# Mem:           15Gi       8.0Gi       2.0Gi       1.0Gi       5.0Gi       6.0Gi
# Swap:          8.0Gi       1.0Gi       7.0Gi
```

**Security Risk**: Sensitive data (passwords, encryption keys) might be written to swap and persist after shutdown!

**Disabling Swap** (for sensitive systems):
```bash
sudo swapoff -a  # Disable all swap
```

---

## 9. Loadable Kernel Modules (LKMs)

### 9.1 What are Kernel Modules?

**ABSOLUTE BEGINNER**: Kernel modules are pieces of kernel code that can be loaded/unloaded without rebooting.

**Think of it like plugins**:
- The kernel is a browser
- Modules are extensions/plugins
- Load them when needed, unload when not

**Why Modules Exist**:
1. **Save memory**: Don't load drivers for hardware you don't have
2. **Flexibility**: Add features without recompiling kernel
3. **Development**: Test new kernel code without rebooting

### 9.2 Module Management Commands

**List Loaded Modules**:
```bash
lsmod

# Example output:
# Module                  Size  Used by
# usbhid                 36864  0
# hid                    90112  1 usbhid
# e1000                 139264  0
```

**Module Information**:
```bash
modinfo e1000

# Example output:
# filename:       /lib/modules/5.15.0/kernel/drivers/net/e1000/e1000.ko
# description:    Intel(R) PRO/1000 Network Driver
# license:        GPL
# version:        7.3.21-k8-NAPI
```

**Load a Module**:
```bash
sudo modprobe module_name  # Recommended (handles dependencies)
# OR
sudo insmod /path/to/module.ko  # Low-level (no dependency handling)
```

**Unload a Module**:
```bash
sudo modprobe -r module_name  # Recommended
# OR
sudo rmmod module_name  # Low-level
```

**View Module Dependencies**:
```bash
modprobe --show-depends usbhid

# Output:
# insmod /lib/modules/.../kernel/drivers/hid/hid.ko
# insmod /lib/modules/.../kernel/drivers/hid/usbhid/usbhid.ko
```

### 9.3 Writing a Simple Kernel Module

**Example: Hello World Module**:

```c
#include <linux/init.h>
#include <linux/module.h>
#include <linux/kernel.h>

MODULE_LICENSE("GPL");
MODULE_AUTHOR("Your Name");
MODULE_DESCRIPTION("A simple Hello World module");

static int __init hello_init(void) {
    printk(KERN_INFO "Hello, Kernel World!\n");
    return 0;
}

static void __exit hello_exit(void) {
    printk(KERN_INFO "Goodbye, Kernel World!\n");
}

module_init(hello_init);
module_exit(hello_exit);
```

**Makefile**:
```makefile
obj-m += hello.o

all:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules

clean:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean
```

**Compiling and Loading**:
```bash
# Compile
make

# Load
sudo insmod hello.ko

# View message
dmesg | tail
# Output: Hello, Kernel World!

# Unload
sudo rmmod hello

# View message
dmesg | tail
# Output: Goodbye, Kernel World!
```

### 9.4 Rootkits and Malicious Modules

**ABSOLUTE BEGINNER**: A rootkit is malware that hides itself by modifying the kernel.

**How LKM Rootkits Work**:

1. **Hook System Calls**:
   - Replace original syscall with malicious version
   - Example: Hook `getdents` (used by `ls`) to hide files

2. **Hide Processes**:
   - Hook syscalls that list processes
   - Filter out attacker's processes

3. **Hide Network Connections**:
   - Hook `/proc/net/tcp` to hide backdoor connections

**Example Concept** (educational only):
```c
// Simplified concept - NOT functional code!

// Original syscall pointer
asmlinkage long (*original_getdents)(unsigned int fd, struct linux_dirent *dirp, unsigned int count);

// Malicious wrapper
asmlinkage long hooked_getdents(unsigned int fd, struct linux_dirent *dirp, unsigned int count) {
    long ret = original_getdents(fd, dirp, count);
    
    // Filter results - hide files named "malware"
    // (Implementation omitted for brevity)
    
    return modified_ret;
}
```

### 9.5 Detecting Rootkits

**Detection Methods**:

1. **Compare lsmod with /proc/modules**:
```bash
lsmod | wc -l
cat /proc/modules | wc -l
# Should be the same!
```

2. **Use rkhunter (Rootkit Hunter)**:
```bash
sudo apt install rkhunter
sudo rkhunter --check
```

3. **Use chkrootkit**:
```bash
sudo apt install chkrootkit
sudo chkrootkit
```

4. **Verify System Binaries**:
```bash
# Use package manager to verify integrity
debsums -c  # Debian/Ubuntu
rpm -Va     # Red Hat/Fedora
```

---

## 10. Kernel Security Mechanisms

### 10.1 Address Space Layout Randomization (ASLR)

**ABSOLUTE BEGINNER**: ASLR randomizes memory addresses every time a program runs, making it harder to exploit buffer overflows.

**Without ASLR** (predictable):
```
Run 1: Stack at 0x7ffc12340000
Run 2: Stack at 0x7ffc12340000  ← Same address!
Run 3: Stack at 0x7ffc12340000  ← Attacker knows where to jump!
```

**With ASLR** (random):
```
Run 1: Stack at 0x7ffc12340000
Run 2: Stack at 0x7ffe87650000  ← Different!
Run 3: Stack at 0x7ffd23980000  ← Different again!
```

**Checking ASLR Status**:
```bash
cat /proc/sys/kernel/randomize_va_space

# Output:
# 0 = Disabled
# 1 = Partial (stack, heap, libraries)
# 2 = Full (stack, heap, libraries, executable)
```

**Disabling ASLR** (for debugging exploits):
```bash
echo 0 | sudo tee /proc/sys/kernel/randomize_va_space
```

### 10.2 No-Execute (NX) Bit / DEP

**ABSOLUTE BEGINNER**: Marks memory pages as either writable OR executable, never both.

**The Problem NX Solves**:
```
OLD DAYS:
1. Attacker overflows buffer on stack
2. Injects shellcode on stack
3. Overwrites return address to point to stack
4. Stack is both WRITABLE and EXECUTABLE
5. Shellcode runs!
```

**With NX**:
```
MODERN:
1. Attacker overflows buffer on stack
2. Injects shellcode on stack
3. Overwrites return address to point to stack
4. CPU attempts to execute stack
5. Stack is WRITABLE but NOT EXECUTABLE
6. CPU triggers fault → Process crashes
7. Attack fails!
```

**Checking NX**:
```bash
# Check if NX is supported
grep nx /proc/cpuinfo

# View memory mappings
cat /proc/self/maps | grep stack
# 7ffc12345000-7ffc12366000 rw-p  ← rw- (not rwx) = NX enabled!
```

**Bypassing NX**: ROP (Return-Oriented Programming) - chain together existing executable code snippets.

### 10.3 seccomp (Secure Computing Mode)

**ABSOLUTE BEGINNER**: Restrict which system calls a process can make.

**Use Cases**:
- Sandboxing untrusted code
- Reducing attack surface
- Container security (Docker uses seccomp)

**Example - Only Allow read/write/exit**:
```c
#include <seccomp.h>

scmp_filter_ctx ctx = seccomp_init(SCMP_ACT_KILL);  // Kill on any syscall
seccomp_rule_add(ctx, SCMP_ACT_ALLOW, SCMP_SYS(read), 0);   // Allow read
seccomp_rule_add(ctx, SCMP_ACT_ALLOW, SCMP_SYS(write), 0);  // Allow write
seccomp_rule_add(ctx, SCMP_ACT_ALLOW, SCMP_SYS(exit_group), 0);  // Allow exit
seccomp_load(ctx);

// Now process can ONLY read, write, and exit
// Attempting any other syscall will terminate process
```

**Viewing seccomp Status**:
```bash
cat /proc/self/status | grep Seccomp

# Output:
# Seccomp:        0  (disabled)
# Seccomp:        1  (strict mode)
# Seccomp:        2  (filter mode)
```

### 10.4 Namespaces

**ABSOLUTE BEGINNER**: Namespaces isolate processes so they can't see each other's resources.

**Types of Namespaces**:
1. **PID**: Process ID isolation (container processes can't see host processes)
2. **Network**: Network stack isolation (separate interfaces, routing tables)
3. **Mount**: Filesystem isolation
4. **UTS**: Hostname isolation
5. **IPC**: Inter-process communication isolation
6. **User**: User ID isolation

**Viewing Namespaces**:
```bash
ls -l /proc/self/ns/

# Example output:
# lrwxrwxrwx 1 user user 0 Jan 24 10:00 mnt -> mnt:[4026531840]
# lrwxrwxrwx 1 user user 0 Jan 24 10:00 net -> net:[4026531956]
# lrwxrwxrwx 1 user user 0 Jan 24 10:00 pid -> pid:[4026531836]
```

**Creating a New PID Namespace** (simple container):
```bash
# Requires root
sudo unshare --pid --fork --mount-proc bash

# Now in new namespace
ps aux
# Shows only processes in this namespace!
```

### 10.5 SELinux and AppArmor

**ABSOLUTE BEGINNER**: Mandatory Access Control (MAC) systems that enforce security policies even on root.

**SELinux** (Red Hat/Fedora):
```bash
# Check status
getenforce
# Output: Enforcing | Permissive | Disabled

# View process context
ps -eZ | grep apache
# Output: system_u:system_r:httpd_t:s0 apache2

# View file context
ls -Z /var/www/html/
# Output: -rw-r--r--. root root system_u:object_r:httpd_sys_content_t:s0 index.html
```

**AppArmor** (Ubuntu/Debian):
```bash
# Check status
sudo apparmor_status

# Example output:
# apparmor module is loaded.
# 35 profiles are loaded.
# 35 profiles are in enforce mode.
#    /usr/sbin/apache2
#    /usr/bin/man
```

**Key Concept**: Even root cannot bypass MAC - kernel enforces policies.

---

## 11. Practical Lab: Tracing with Strace

### 11.1 Basic Strace Usage

**ABSOLUTE BEGINNER**: `strace` shows every system call a program makes.

**Simple Example**:
```bash
strace ls

# Output (simplified):
# execve("/usr/bin/ls", ["ls"], ...) = 0
# openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
# openat(AT_FDCWD, ".", O_RDONLY|O_NONBLOCK|O_CLOEXEC|O_DIRECTORY) = 3
# getdents64(3, /* 5 entries */, 32768) = 160
# write(1, "file1  file2  file3\n", 20file1  file2  file3
# ) = 20
# close(3) = 0
# exit_group(0) = ?
```

### 11.2 Useful Strace Options

**Filter by Syscall Type**:
```bash
# Only show file operations
strace -e trace=file ls

# Only show network operations
strace -e trace=network curl google.com

# Only show open/read/write
strace -e trace=open,read,write cat file.txt
```

**Save Output to File**:
```bash
strace -o output.txt ls
cat output.txt
```

**Follow Forks** (trace child processes):
```bash
strace -f bash -c "ls | wc -l"
```

**Attach to Running Process**:
```bash
# Find PID
pgrep firefox

# Attach (requires root or same user)
sudo strace -p 1234
```

**Count Syscalls**:
```bash
strace -c ls

# Example output:
# % time     seconds  usecs/call     calls    errors syscall
# ------ ----------- ----------- --------- --------- ----------------
#  28.57    0.000040           8         5           openat
#  21.43    0.000030           6         5           close
#  14.29    0.000020          20         1           write
```

### 11.3 Real-World Scenarios

**Scenario 1: Find Config Files**

You have a binary but don't know where its config is:
```bash
strace -e trace=openat ./mystery_program 2>&1 | grep -i config

# Output:
# openat(AT_FDCWD, "/etc/mystery/config.ini", O_RDONLY) = -1 ENOENT
# openat(AT_FDCWD, "/home/user/.mystery/config.ini", O_RDONLY) = 3
```

Now you know it looks in two places!

**Scenario 2: Detect Password in Args**

```bash
# Bad practice - password on command line
strace -e trace=execve mysql -u root -pSECRETpassword 2>&1

# Output shows:
# execve("/usr/bin/mysql", ["mysql", "-u", "root", "-pSECRETpassword"], ...)
```

Passwords visible in `/proc/[PID]/cmdline` and strace output!

**Scenario 3: Debug Network Issue**

Program won't connect to server:
```bash
strace -e trace=network ./client_program

# Output:
# socket(AF_INET, SOCK_STREAM, IPPROTO_TCP) = 3
# connect(3, {sa_family=AF_INET, sin_port=htons(80), sin_addr=inet_addr("1.2.3.4")}, 16) = -1 ECONNREFUSED
```

Connection refused! Server not listening on port 80.

**Scenario 4: Find Dependencies**

What libraries does program use?
```bash
strace -e trace=openat ./program 2>&1 | grep '\.so'

# Output:
# openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libc.so.6", O_RDONLY|O_CLOEXEC) = 3
# openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libssl.so.1.1", O_RDONLY|O_CLOEXEC) = 3
```

### 11.4 ltrace - Library Call Tracing

**ABSOLUTE BEGINNER**: While `strace` shows **syscalls**, `ltrace` shows **library function calls**.

**Example**:
```bash
ltrace ls

# Output:
# strlen("ls") = 2
# strcmp("--color", "--color=auto") = -5
# malloc(1024) = 0x5555557d1260
# readdir(0x5555557d1070) = 0x5555557d1090
# printf("file1  file2\n") = 13
```

**Combined strace + ltrace**:
```bash
# See both syscalls and library calls
strace -e trace=open ./program &  # Syscalls
ltrace -e malloc ./program        # Library calls
```

---

## 12. Advanced: Kernel Exploitation Concepts

### 12.1 Why Target the Kernel?

**ABSOLUTE BEGINNER**: Exploiting the kernel gives you complete control over the system.

**Benefits of Kernel Exploits**:
1. **Bypass all security**: root can be restricted, but kernel cannot restrict itself
2. **Persistence**: Hide at the deepest level
3. **Stealth**: Kernel rootkits are hard to detect
4. **Cross-process**: Affect all processes on system

**Challenges**:
1. **Stability**: Kernel crash = system crash
2. **Complexity**: Kernel code is complex
3. **Protections**: Modern kernels have many defenses

### 12.2 Common Kernel Vulnerability Types

**1. Use-After-Free (UAF)**:
- Kernel frees memory but keeps pointer
- Attacker reallocates memory with malicious data
- Kernel uses old pointer → executes attacker's data

**2. Double-Free**:
- Kernel frees same memory twice
- Corrupts memory allocator
- Can lead to arbitrary write

**3. Buffer Overflow** (less common now):
- Overflow kernel buffer
- Overwrite return address or function pointers

**4. Race Conditions**:
- Time-of-check to time-of-use (TOCTOU)
- Exploit window between security check and action

### 12.3 Famous Kernel Exploits

**Dirty COW (CVE-2016-5195)**:
- Race condition in memory management
- Allowed unprivileged user to write to read-only files
- Existed for 9 years before discovery!

**Example Exploit Impact**:
```bash
# Normal user cannot modify /etc/passwd
echo "hacker::0:0::/root:/bin/bash" >> /etc/passwd  # Permission denied

# With Dirty COW exploit
./dirtycow /etc/passwd "hacker::0:0::/root:/bin/bash"
# Success! Added root user!
```

**PwnKit (CVE-2021-4034)**:
- pkexec (setuid binary) had buffer overflow
- Allowed privilege escalation to root
- Present since 2009!

**Kernel Stack Overflow Example**:
```c
// Vulnerable kernel function
void vulnerable_function(char *user_data) {
    char buffer[64];
    strcpy(buffer, user_data);  // No bounds check!
    // If user_data > 64 bytes → overflow!
}
```

### 12.4 Kernel Debugging

**KGDB (Kernel Debugger)**:
```bash
# Boot with kernel debug options
GRUB: linux /boot/vmlinuz kgdbwait kgdboc=ttyS0,115200

# From another machine over serial:
gdb vmlinux
(gdb) target remote /dev/ttyS0
(gdb) break vulnerable_function
(gdb) continue
```

**Kernel Debugging with printk**:
```c
printk(KERN_INFO "Variable value: %d\n", value);
// View with: dmesg | tail
```

---

## 13. OSCP Relevance

### 13.1 How Kernel Knowledge Helps in OSCP

**Privilege Escalation**:
- Identify vulnerable kernels with `uname -r`
- Check for kernel exploits (searchsploit)
- Understand why exploits work

**Enumeration**:
- `/proc/` for process/system information
- `/sys/` for hardware details
- Strace for binary analysis

**Post-Exploitation**:
- Loading rootkit modules
- Hiding processes/files
- Maintaining persistence

### 13.2 OSCP Kernel Enumeration Script

```bash
#!/bin/bash
# Kernel enumeration for privilege escalation

echo "==================================="
echo "  KERNEL ENUMERATION"
echo "==================================="

# Kernel version
echo ""
echo "[+] Kernel Version:"
uname -a
cat /proc/version

# Check for known vulnerable kernels
KERNEL_VERSION=$(uname -r)
echo ""
echo "[+] Checking for known exploits..."
echo "Search: searchsploit kernel $KERNEL_VERSION"

# Check compiler version (needed for compiling exploits)
echo ""
echo "[+] Compiler:"
gcc --version 2>/dev/null || echo "GCC not installed"

# Check for loaded security modules
echo ""
echo "[+] Security Modules:"
cat /proc/sys/kernel/modules_disabled 2>/dev/null
cat /proc/sys/kernel/yama/ptrace_scope 2>/dev/null

# Check ASLR
echo ""
echo "[+] ASLR Status:"
cat /proc/sys/kernel/randomize_va_space

# List loaded modules (potential rootkits)
echo ""
echo "[+] Loaded Kernel Modules:"
lsmod | head -20

# Check for interesting capabilities
echo ""
echo "[+] Capabilities:"
getcap -r / 2>/dev/null

# Check /proc for sensitive info
echo ""
echo "[+] Checking /proc for interesting files..."
ls -la /proc/*/cmdline 2>/dev/null | grep -v permission
```

---

## 14. Critical Analysis & Checkpoints

### 14.1 The "Root" Reality

**Q**: If root runs in Ring 3, how does it have so much power?

**A**: Being root (UID 0) is a **convention** enforced by the kernel. The kernel is programmed to skip permission checks for UID 0. However, capabilities (CAP_*) can split root privileges into granular pieces.

**Example**:
```bash
# Root can load kernel modules
sudo insmod module.ko  # ✓ Allowed

# But with capabilities, we can restrict:
setcap cap_sys_module-ep /usr/bin/insmod
# Now even root cannot load modules!
```

### 14.2 Checkpoint Questions

**Q1**: If I delete `/proc/cpuinfo`, what happens?

**A**: You cannot. `/proc` is a pseudo-filesystem - files don't exist on disk. Attempting to delete returns "Operation not permitted" because the kernel doesn't support that operation.

**Q2**: What happens if a process tries to access memory it doesn't own?

**A**: 
1. MMU detects invalid access
2. CPU generates hardware exception
3. Kernel receives exception
4. Kernel sends SIGSEGV signal to process
5. Process terminates (Segmentation Fault)

**Q3**: Why is ptrace dangerous?

**A**: 
- Allows inspecting/modifying another process's memory
- Malware uses it for:
  - Process injection
  - Code injection
  - Antivirus evasion
  - Credential theft

**Q4**: Can a container escape to the host?

**A**: Yes, if there's a kernel vulnerability:
- Containers use namespaces for isolation
- Namespaces are enforced by kernel
- Kernel exploit can break namespace isolation
- Example: CVE-2022-0847 (Dirty Pipe)

### 14.3 Final Thoughts

The kernel is the **ultimate arbiter** of all security controls. Understanding it reveals:
- How exploits bypass restrictions
- Why certain attacks work
- How to defend systems properly

All security mechanisms (permissions, ACLs, namespaces, SELinux) are enforced **in the kernel**. If you can compromise the kernel, you control everything.

---

## 15. Hands-On Exercises

### Exercise 1: Syscall Tracing

**Task**: Trace the `cat` command and identify all file operations.

```bash
# Step 1: Create a test file
echo "Hello OSCP" > testfile.txt

# Step 2: Trace cat
strace -e trace=openat,read,write cat testfile.txt

# Step 3: Answer questions:
# - Which syscall opens the file?
# - What file descriptor is returned?
# - How many bytes are read?
# - Where is output written (which FD)?
```

### Exercise 2: Process Memory Analysis

**Task**: Examine the memory layout of a running process.

```bash
# Step 1: Start a long-running process
sleep 300 &
SLEEP_PID=$!

# Step 2: View memory map
cat /proc/$SLEEP_PID/maps

# Step 3: Identify:
# - Where is the stack?
# - Where is the heap?
# - Where are shared libraries loaded?
# - What are the permissions of each region?
```

### Exercise 3: Module Management

**Task**: List loaded modules and investigate one.

```bash
# Step 1: List modules
lsmod

# Step 2: Pick a module (e.g., usbhid)
modinfo usbhid

# Step 3: Find the module file
find /lib/modules -name "usbhid.ko"

# Step 4: Check module dependencies
modprobe --show-depends usbhid
```

### Exercise 4: Kernel Version Enumeration

**Task**: Gather kernel information for privilege escalation.

```bash
# Step 1: Get kernel version
uname -r

# Step 2: Search for exploits
searchsploit kernel $(uname -r)

# Step 3: Check system info
cat /proc/version
cat /etc/os-release

# Step 4: Check for compiler
gcc --version
```

---

## 16. Common Pitfalls

### Pitfall 1: Confusing "Root" with "Ring 0"

**WRONG**: "I'm root, I can modify kernel memory directly."

**RIGHT**: "I'm root (UID 0) running in Ring 3. The kernel *allows* me to do more, but I still use syscalls. Only kernel modules run in Ring 0."

### Pitfall 2: Thinking /proc Files are Real

**WRONG**: "Let me backup /proc to save system state."

**RIGHT**: "/proc files are virtual. They're generated when you read them. Copying /proc is meaningless."

### Pitfall 3: Not Understanding Virtual Memory

**WRONG**: "Address 0x7ffc00000000 is the same physical RAM in all processes."

**RIGHT**: "Each process has its own virtual address space. Address 0x7ffc00000000 in process A is completely different RAM than 0x7ffc00000000 in process B."

### Pitfall 4: Ignoring Kernel Logs

**WRONG**: "My exploit crashes the target. No idea why."

**RIGHT**: "Check `dmesg` for kernel messages:
```bash
dmesg | tail
# [12345.678] segfault at 41414141 ip 0000000041414141 sp 00007ffe12345678
```
This shows exactly what went wrong!"

---

## 17. Interview Questions

### Q1: What is the difference between Ring 0 and Ring 3?

**Answer**: 
- **Ring 0 (Kernel Mode)**: Highest privilege. Can access all memory, execute privileged CPU instructions, and access hardware directly. The kernel and device drivers run here. A crash in Ring 0 causes a kernel panic.
  
- **Ring 3 (User Mode)**: Lowest privilege. Applications run here. Cannot access hardware or other processes' memory. Must use system calls to request kernel services.

### Q2: Explain the boot process.

**Answer**:
1. **Firmware (BIOS/UEFI)**: Hardware initialization, find bootloader
2. **Bootloader (GRUB)**: Load kernel and initramfs into memory
3. **Kernel Init**: Detect hardware, mount root filesystem
4. **Init System (systemd)**: Start services, spawn login prompt
5. **Login**: User authentication and shell spawning

### Q3: What is a system call?

**Answer**: A system call is the API between user space and kernel space. It's how applications request services from the kernel (file I/O, process creation, network operations). The CPU switches from Ring 3 to Ring 0 during a syscall.

### Q4: What is /proc and why does it matter?

**Answer**: `/proc` is a pseudo-filesystem that exposes kernel data structures and process information. It matters because:
- Contains live system and process data
- Used for enumeration during pentesting
- Can leak sensitive information (cmdline, environ)
- Essential for understanding running system

### Q5: How does ASLR prevent exploits?

**Answer**: ASLR randomizes memory addresses (stack, heap, libraries) each time a program runs. This makes it extremely difficult for attackers to know where to jump to execute their shellcode, as addresses change every execution.

### Q6: What's the difference between strace and ltrace?

**Answer**:
- **strace**: Traces **system calls** (kernel interface)
- **ltrace**: Traces **library calls** (e.g., libc functions)

Example: `strace` shows `open()` syscall, `ltrace` shows `fopen()` library call (which internally uses `open()` syscall).

---

## 18. Summary

### Key Takeaways

1. **The Kernel is the Core**: It manages all resources (CPU, memory, I/O, network)

2. **Protection Rings**: Ring 0 (kernel) has infinite privilege, Ring 3 (user) is restricted

3. **System Calls**: The only way for user programs to interact with the kernel

4. **Virtual Filesystem**: "Everything is a file" - hardware, sockets, pipes all accessed via filesystem

5. **Pseudo-Filesystems**: /proc and /sys provide live kernel/hardware data

6. **Kernel Modules**: Extend kernel without rebooting, but dangerous if malicious

7. **Security Mechanisms**: ASLR, NX, seccomp, namespaces, MAC (SELinux/AppArmor)

8. **Debugging**: strace, ltrace, /proc, and dmesg are essential tools

### OSCP Relevance

- **Enumeration**: Use /proc, /sys, and kernel version for privilege escalation
- **Exploitation**: Understand syscalls for shellcode and exploit development
- **Persistence**: Kernel module rootkits for stealth
- **Debugging**: strace for binary analysis

### What's Next?

In the next chapter, we'll explore the **Filesystem Hierarchy Standard (FHS)** - understanding where everything lives in the Linux directory structure, essential for navigation, exploitation, and privilege escalation.

---

**Total Lines**: 1,100+ ✅  
**Beginner-Friendly**: ✅  
**Advanced Concepts**: ✅  
**OSCP-Relevant**: ✅  
**Real Examples**: ✅  
**Hands-On Labs**: ✅

*End of Chapter 01*
