# Bash Basics & Your First Script: From Zero to Automation

---
title: "Bash Basics & Your First Script"
parent: "[[03b_Bash_Scripting]]"
tags:
  - bash
  - scripting
  - linux
  - fundamentals
  - shebang
  - permissions
  - beginner
created: 2026-01-24
updated: 2026-01-24
difficulty: "Absolute Beginner to Intermediate"
---

> **Executive Summary**: 
> Bash (Bourne Again SHell) is the command interpreter that lets you talk to your Linux system. While you may have used it to type commands like `ls` or `cd`, Bash is actually a full programming language. This chapter starts from absolute zero - assuming you know nothing - and takes you to writing your first automation scripts. By the end, you'll understand what happens when you type `./script.sh`, why permissions matter, and how to create tools that save hours of manual work.

---

## Table of Contents

1. [What is Bash? Understanding the Shell](#1-what-is-bash-understanding-the-shell)
2. [Your First Command: Hello World](#2-your-first-command-hello-world)
3. [Creating Your First Script](#3-creating-your-first-script)
4. [The Shebang Line Explained](#4-the-shebang-line-explained)
5. [File Permissions & Making Scripts Executable](#5-file-permissions--making-scripts-executable)
6. [Running Scripts: Different Methods](#6-running-scripts-different-methods)
7. [Comments & Documentation](#7-comments--documentation)
8. [Basic Commands Every Script Uses](#8-basic-commands-every-script-uses)
9. [The PATH Variable](#9-the-path-variable)
10. [Script Structure & Best Practices](#10-script-structure--best-practices)
11. [Practical Examples](#11-practical-examples)
12. [Common Beginner Mistakes](#12-common-beginner-mistakes)
13. [Hands-On Labs](#13-hands-on-labs)
14. [Interview Questions & Checkpoints](#14-interview-questions--checkpoints)

---

## 1. What is Bash? Understanding the Shell

**ABSOLUTE BEGINNER EXPLANATION**: 
Imagine your computer is a restaurant kitchen. The operating system (Linux) is like the kitchen staff who actually cook the food. But you can't just walk into the kitchen and tell them what to do directly. You need a waiter to take your order and translate it to the kitchen. **Bash is that waiter** - it takes your commands (in human-readable text) and tells the operating system what to do.

### 1.1 The Shell vs Terminal vs Console

People often confuse these terms. Let's clarify:

```
┌─────────────────────────────────────────────────────┐
│  TERMINAL (The Window)                              │
│  ┌───────────────────────────────────────────────┐  │
│  │ SHELL (The Interpreter - Bash)                │  │
│  │  ┌─────────────────────────────────────────┐  │  │
│  │  │ KERNEL (The Operating System Core)      │  │  │
│  │  │  - Manages files                        │  │  │
│  │  │  - Manages processes                    │  │  │
│  │  │  - Manages hardware                     │  │  │
│  │  └─────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────┘
```

**Terminal**: The window application (like GNOME Terminal, Konsole, Windows Terminal)  
**Shell**: The command interpreter (Bash, Zsh, Fish, Sh)  
**Kernel**: The core of the operating system (Linux)

### 1.2 Why Bash?

**Reasons Bash is Essential**:

1. **Pre-installed Everywhere**: Every Linux system has Bash. You can't say that about Python or other languages.

2. **Living Off The Land**: In penetration testing, you often can't upload custom tools. Bash is already there.

3. **Automation**: Repetitive tasks become one-click operations.

4. **System Integration**: Bash talks directly to the operating system better than most languages.

5. **Quick Prototyping**: For system tasks, Bash is faster to write than Python or C.

### 1.3 Bash vs Other Shells

```
┌──────────────┬─────────────────────────────────────────────┐
│ Shell        │ Description                                 │
├──────────────┼─────────────────────────────────────────────┤
│ sh           │ Original Bourne Shell (very basic)          │
│ bash         │ Bourne Again Shell (most common)            │
│ zsh          │ Z Shell (modern, powerful, MacOS default)   │
│ fish         │ Friendly Interactive Shell (beginner-friendly)│
│ dash         │ Debian Almquist Shell (lightweight, fast)   │
└──────────────┴─────────────────────────────────────────────┘
```

**For OSCP/Security Work**: Stick with Bash. It's everywhere.

---

## 2. Your First Command: Hello World

**ABSOLUTE BEGINNER**: Let's start by typing your first command.

### 2.1 Opening the Terminal

**On Kali Linux / Ubuntu**:
- Press `Ctrl + Alt + T`
- Or search for "Terminal" in applications

**You'll see something like**:
```bash
┌──(kali㉿kali)-[~]
└─$
```

This is called the **prompt**. It shows:
- Your username (`kali`)
- Your hostname (`kali`)
- Current directory (`~` means home)
- `$` means regular user (vs `#` for root)

### 2.2 The Echo Command

Type this and press Enter:

```bash
echo "Hello, World!"
```

**What Happened?**
- `echo` is a command that prints text
- `"Hello, World!"` is the text to print
- The quotes are optional for simple text, but good practice

**Output**:
```
Hello, World!
```

### 2.3 Your Second Command

```bash
echo "My name is $USER and I am in the directory $PWD"
```

**Output** (yours will be different):
```
My name is kali and I am in the directory /home/kali
```

**What are $USER and $PWD?**
- These are **variables** - containers that store information
- `$USER` stores your username
- `$PWD` stores your current directory (Print Working Directory)
- The `$` tells Bash "this is a variable, show me its value"

### 2.4 Trying Basic Commands

```bash
# Show current date and time
date

# Show who is logged in
whoami

# Show current directory
pwd

# List files in current directory
ls

# Show detailed file listing
ls -la
```

**BEGINNER TIP**: Don't memorize these yet. Just try them to see Bash responding to your commands.

---

## 3. Creating Your First Script

**ABSOLUTE BEGINNER EXPLANATION**: 
Typing commands one by one is fine, but what if you need to run the same 20 commands every day? That's where scripts come in. A script is just a text file containing a list of commands. When you run the script, Bash executes all the commands automatically.

### 3.1 Creating the File

**Method 1: Using nano (Beginner-Friendly Editor)**

```bash
nano first_script.sh
```

This opens a text editor. Type the following:

```bash
#!/bin/bash
echo "This is my first script!"
echo "Today is: $(date)"
echo "I am user: $USER"
```

**To Save and Exit**:
1. Press `Ctrl + O` (Write Out)
2. Press `Enter` (Confirm filename)
3. Press `Ctrl + X` (Exit)

**Method 2: Using echo (One-Liner)**

```bash
echo '#!/bin/bash' > first_script.sh
echo 'echo "This is my first script!"' >> first_script.sh
echo 'echo "Today is: $(date)"' >> first_script.sh
```

**Explanation of `>` vs `>>`**:
- `>` creates new file (overwrites if exists)
- `>>` appends to existing file

### 3.2 Viewing Your Script

```bash
cat first_script.sh
```

**Output**:
```bash
#!/bin/bash
echo "This is my first script!"
echo "Today is: $(date)"
echo "I am user: $USER"
```

### 3.3 Understanding the Script

Let's break down each line:

**Line 1**: `#!/bin/bash`
- This is called the **shebang** (we'll explain in detail next section)
- It tells the system "use the Bash interpreter to run this script"

**Line 2**: `echo "This is my first script!"`
- Print a message

**Line 3**: `echo "Today is: $(date)"`
- `$(date)` runs the `date` command and puts its output into the echo
- This is called **command substitution**

**Line 4**: `echo "I am user: $USER"`
- `$USER` is replaced with your username

---

## 4. The Shebang Line Explained

**ABSOLUTE BEGINNER EXPLANATION**:
The shebang (`#!`) is like writing the address on an envelope. It tells the system where to find the program that knows how to read your script.

### 4.1 Why We Need It

Without a shebang, the system doesn't know what language your script is written in. Is it Bash? Python? Perl? The shebang line says "Hey, this is a Bash script, use `/bin/bash` to run it."

### 4.2 Anatomy of the Shebang

```bash
#!/bin/bash
│││      │
│││      └─ Path to the Bash interpreter
││└─ Exclamation mark
│└─ Hash symbol
└─ Together they mean "use this program to interpret this file"
```

### 4.3 Common Shebang Lines

```bash
#!/bin/bash          # Standard Bash script
#!/bin/sh            # Bourne shell (basic, compatible)
#!/usr/bin/env bash  # Find bash in PATH (portable)
#!/usr/bin/python3   # Python script
#!/usr/bin/perl      # Perl script
```

### 4.4 The `/usr/bin/env` Method

**Two Options**:

**Option 1: Absolute Path**
```bash
#!/bin/bash
```
- Direct path to Bash
- Faster (no PATH lookup)
- Problem: Bash might be in different location on different systems

**Option 2: Using env**
```bash
#!/usr/bin/env bash
```
- `env` finds bash in your PATH
- More portable across systems
- Slightly slower

**For Beginners**: Use `#!/bin/bash` for now. It works on all Linux systems.

### 4.5 What Happens Without a Shebang?

**Test it**:

Create a script without shebang:
```bash
echo 'echo "No shebang here"' > no_shebang.sh
chmod +x no_shebang.sh
./no_shebang.sh
```

**Result**: It still works! Why?

When you run `./no_shebang.sh`, your current shell (Bash) notices there's no shebang and says "I'll just assume it's a Bash script and run it myself."

**BUT** - if you try to run it from a different shell, or someone else runs it, it might fail. **Always use a shebang**.

---

## 5. File Permissions & Making Scripts Executable

**ABSOLUTE BEGINNER EXPLANATION**:
In Linux, every file has permissions that control who can read it, write to it, or execute (run) it. Think of it like locks on doors - some people have keys, some don't.

### 5.1 Understanding Linux Permissions

When you type `ls -l`, you see something like this:

```bash
ls -l first_script.sh
```

**Output**:
```
-rw-r--r-- 1 kali kali 89 Jan 24 10:30 first_script.sh
│││││││││
│││││││││
│││││││└└─ Others (everyone else) can read
││││││└──── Group can read
│││││└───── Group cannot write
││││└────── Group cannot execute
│││└─────── Owner can read
││└──────── Owner can write
│└───────── Owner cannot execute
└────────── File type (- = regular file)
```

### 5.2 Permission Types

```
r = Read    (4)
w = Write   (2)
x = Execute (1)
```

**Three Groups**:
1. **Owner** (u) - The person who created the file
2. **Group** (g) - Users in the same group
3. **Others** (o) - Everyone else

### 5.3 Making a Script Executable

**Current State** (from ls -l above):
```
-rw-r--r--
```
- Owner can read/write
- Nobody can execute (no `x`)

**Add Execute Permission**:
```bash
chmod +x first_script.sh
```

**Check Again**:
```bash
ls -l first_script.sh
```

**Output**:
```
-rwxr-xr-x 1 kali kali 89 Jan 24 10:30 first_script.sh
   │  │  │
   │  │  └─ Others can execute
   │  └──── Group can execute
   └─────── Owner can execute
```

### 5.4 Running the Script

**Now you can run it**:
```bash
./first_script.sh
```

**Output**:
```
This is my first script!
Today is: Sat Jan 24 10:30:00 AM EST 2026
I am user: kali
```

**Why `./` ?**

- `.` means "current directory"
- `/` is the path separator
- Together: "run the script in the current directory"

Without `./`, Bash would search for `first_script.sh` in your PATH (system directories), not the current folder.

### 5.5 Numeric Permissions (chmod 755)

**Alternative Method**:
```bash
chmod 755 first_script.sh
```

**What does 755 mean?**
```
7 = rwx (owner: read+write+execute)
5 = r-x (group: read+execute)
5 = r-x (others: read+execute)
```

**Common Permission Sets**:
```
755 - Owner: full, Others: read/execute (common for scripts)
700 - Owner: full, Others: nothing (private script)
644 - Owner: read/write, Others: read only (text files)
600 - Owner: read/write, Others: nothing (private files)
```

### 5.6 Checking Who You Are

```bash
whoami        # Your username
id            # Your user ID and groups
ls -l script.sh | awk '{print $3, $4}'  # Owner and group of file
```

---

## 6. Running Scripts: Different Methods

**ABSOLUTE BEGINNER**: There are actually several ways to run a Bash script. Each method has different effects.

### 6.1 Method 1: Direct Execution (./script.sh)

```bash
./first_script.sh
```

**What Happens**:
1. Your shell creates a new process (sub-shell)
2. The sub-shell reads the shebang
3. Launches `/bin/bash` to interpret the script
4. Script runs in that new Bash process
5. When done, the sub-shell exits
6. Control returns to your original shell

**Key Point**: Changes made in the script (like changing directory with `cd`) don't affect your main shell.

**Test**:
```bash
# Create a script that changes directory
echo '#!/bin/bash' > change_dir.sh
echo 'cd /tmp' >> change_dir.sh
echo 'pwd' >> change_dir.sh
chmod +x change_dir.sh

# Run it
./change_dir.sh
# Output: /tmp

# Check where you are now
pwd
# Output: /home/kali (not /tmp!)
```

### 6.2 Method 2: Bash Interpreter (bash script.sh)

```bash
bash first_script.sh
```

**What Happens**:
- You explicitly tell Bash to run the script
- Shebang is ignored (Bash already knows it's a Bash script)
- Still runs in sub-shell

**When to Use**: When script doesn't have execute permission, or you want to force a specific shell.

### 6.3 Method 3: Sourcing (source script.sh or . script.sh)

```bash
source first_script.sh
# OR
. first_script.sh
```

**What Happens**:
- Script runs in your **current shell** (no sub-shell)
- Changes affect your environment

**Test**:
```bash
# Run the change_dir.sh script with source
source change_dir.sh
# Output: /tmp

# Check where you are now
pwd
# Output: /tmp (you actually changed directory!)
```

**When to Use**: 
- When you want to modify your current environment
- Loading configuration files (`.bashrc`, `.profile`)
- Setting environment variables

### 6.4 Method 4: sh Command (POSIX Shell)

```bash
sh first_script.sh
```

**Warning**: `sh` is not the same as `bash`!
- `sh` is the basic POSIX shell
- Missing many Bash features (arrays, `[[` tests, etc.)
- Your script might not work correctly

### 6.5 Comparison Table

```
┌──────────────────┬────────────┬─────────────┬────────────────┐
│ Method           │ Sub-Shell? │ Needs +x?   │ Uses Shebang?  │
├──────────────────┼────────────┼─────────────┼────────────────┤
│ ./script.sh      │ Yes        │ Yes         │ Yes            │
│ bash script.sh   │ Yes        │ No          │ No             │
│ source script.sh │ No         │ No          │ No             │
│ sh script.sh     │ Yes        │ No          │ No             │
└──────────────────┴────────────┴─────────────┴────────────────┘
```

---

## 7. Comments & Documentation

**ABSOLUTE BEGINNER**: Comments are notes for humans. Bash ignores them completely. They help you remember what your code does when you come back to it months later.

### 7.1 Single-Line Comments

```bash
#!/bin/bash

# This is a comment - Bash ignores this line
echo "This will run"  # Comment after a command

# Comments help you remember what code does
# Always explain WHY, not just WHAT
```

### 7.2 Multi-Line Comments (Trick)

Bash doesn't have multi-line comments, but we can use a trick:

```bash
: '
This is a multi-line comment.
Everything between the quotes is ignored.
Useful for long explanations or temporarily disabling code.
'
```

**How it works**: 
- `:` is a no-op command (does nothing)
- Everything in single quotes is just a string argument to `:`, which does nothing

### 7.3 Script Header Template

**Professional Script Template**:

```bash
#!/bin/bash
#
# Script Name: network_scanner.sh
# Description: Scans a network for alive hosts
# Author: Your Name
# Created: 2026-01-24
# Modified: 2026-01-24
# Version: 1.0
#
# Usage: ./network_scanner.sh 192.168.1.0/24
#
# Notes:
#   - Requires nmap to be installed
#   - Must be run as root for SYN scan
#

# Script starts here
```

### 7.4 Section Dividers

```bash
#!/bin/bash

################################################################################
# CONFIGURATION SECTION
################################################################################
TARGET="192.168.1.1"
PORT=80

################################################################################
# FUNCTIONS
################################################################################
check_host() {
    ping -c 1 "$1" > /dev/null 2>&1
}

################################################################################
# MAIN SCRIPT
################################################################################
echo "Scanning $TARGET"
```

---

## 8. Basic Commands Every Script Uses

**ABSOLUTE BEGINNER**: These are the commands you'll use in almost every script.

### 8.1 Echo - Printing Output

```bash
#!/bin/bash

# Basic echo
echo "Hello"

# Echo without newline (use -n)
echo -n "Loading..."
sleep 2
echo "Done!"

# Echo with colors (ANSI codes)
echo -e "\e[31mThis is red\e[0m"
echo -e "\e[32mThis is green\e[0m"
echo -e "\e[33mThis is yellow\e[0m"

# Echo variables
NAME="John"
echo "Hello, $NAME"
```

**Color Codes**:
```bash
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

echo -e "${RED}Error${NC}"
echo -e "${GREEN}Success${NC}"
```

### 8.2 Read - Getting User Input

```bash
#!/bin/bash

# Basic read
echo "What is your name?"
read name
echo "Hello, $name!"

# Read with prompt
read -p "Enter your age: " age
echo "You are $age years old"

# Read password (hidden input)
read -sp "Enter password: " password
echo
echo "Password received"

# Read with timeout (10 seconds)
read -t 10 -p "Quick! Enter something: " input
echo "You entered: $input"
```

### 8.3 Sleep - Pausing Execution

```bash
#!/bin/bash

echo "Starting process..."
sleep 2          # Pause for 2 seconds
echo "Still working..."
sleep 0.5        # Pause for 0.5 seconds
echo "Done!"
```

### 8.4 Date - Working with Time

```bash
#!/bin/bash

# Current date and time
date

# Custom format
date "+%Y-%m-%d"              # 2026-01-24
date "+%H:%M:%S"              # 14:30:45
date "+%Y-%m-%d_%H-%M-%S"     # 2026-01-24_14-30-45

# Use in filenames
LOGFILE="scan_$(date +%Y%m%d_%H%M%S).log"
echo "Creating log: $LOGFILE"
```

### 8.5 Command Substitution

**Two Methods**:

**Method 1: Backticks** (old style)
```bash
CURRENT_DATE=`date`
echo $CURRENT_DATE
```

**Method 2: $() ** (modern, recommended)
```bash
CURRENT_DATE=$(date)
echo $CURRENT_DATE

# Can be nested
YEAR=$(date +%Y)
MONTH=$(date +%m)
echo "Year: $YEAR, Month: $MONTH"
```

**BEGINNER TIP**: Always use `$()` instead of backticks. It's clearer and easier to nest.

### 8.6 Exit Codes

```bash
#!/bin/bash

# Every command returns an exit code
# 0 = success
# Non-zero = error

ls /existing_directory
echo "Exit code: $?"  # Prints: 0

ls /nonexistent_directory
echo "Exit code: $?"  # Prints: 2 (error)

# You can set your own exit code
exit 0    # Success
exit 1    # General error
exit 127  # Command not found
```

---

## 9. The PATH Variable

**ABSOLUTE BEGINNER**: When you type a command like `ls`, how does Bash know where to find it?

### 9.1 Understanding PATH

```bash
# View your PATH
echo $PATH
```

**Output** (example):
```
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

This is a list of directories separated by `:`. When you type `ls`, Bash searches these directories **in order** until it finds the `ls` command.

### 9.2 Which Command

```bash
# Find where a command is located
which ls
# Output: /usr/bin/ls

which python3
# Output: /usr/bin/python3

which mycommand
# Output: (nothing - not found)
```

### 9.3 Adding to PATH

**Temporary** (only for current session):
```bash
export PATH=$PATH:/home/kali/scripts
```

**Permanent** (add to `~/.bashrc`):
```bash
echo 'export PATH=$PATH:/home/kali/scripts' >> ~/.bashrc
source ~/.bashrc
```

### 9.4 Running Scripts from Anywhere

**Option 1**: Put script in PATH directory
```bash
sudo cp myscript.sh /usr/local/bin/myscript
sudo chmod +x /usr/local/bin/myscript

# Now you can run it from anywhere
myscript
```

**Option 2**: Add your script directory to PATH
```bash
mkdir ~/bin
cp myscript.sh ~/bin/
chmod +x ~/bin/myscript.sh
export PATH=$PATH:~/bin

# Now you can run
myscript.sh
```

---

## 10. Script Structure & Best Practices

**ABSOLUTE BEGINNER**: A well-structured script is easier to read, debug, and maintain.

### 10.1 Professional Script Template

```bash
#!/bin/bash
#
# Script: port_scanner.sh
# Description: Simple port scanner using nc
# Author: Security Student
# Version: 1.0
# Usage: ./port_scanner.sh <IP> <start_port> <end_port>
#

################################################################################
# STRICT MODE (Recommended for all scripts)
################################################################################
set -euo pipefail
# -e: Exit on error
# -u: Error on undefined variable
# -o pipefail: Error if any command in pipe fails

################################################################################
# COLOR CODES
################################################################################
readonly RED='\033[0;31m'
readonly GREEN='\033[0;32m'
readonly YELLOW='\033[1;33m'
readonly NC='\033[0m' # No Color

################################################################################
# GLOBAL VARIABLES
################################################################################
readonly SCRIPT_NAME=$(basename "$0")
readonly SCRIPT_DIR=$(dirname "$0")

################################################################################
# FUNCTIONS
################################################################################

# Print error message and exit
error_exit() {
    echo -e "${RED}[ERROR]${NC} $1" >&2
    exit 1
}

# Print success message
success() {
    echo -e "${GREEN}[SUCCESS]${NC} $1"
}

# Print info message
info() {
    echo -e "${YELLOW}[INFO]${NC} $1"
}

# Show usage information
usage() {
    cat << EOF
Usage: $SCRIPT_NAME <IP> <start_port> <end_port>

Scans ports on target IP address.

Arguments:
    IP          Target IP address
    start_port  First port to scan
    end_port    Last port to scan

Example:
    $SCRIPT_NAME 192.168.1.1 1 1000

EOF
    exit 1
}

################################################################################
# MAIN SCRIPT
################################################################################

# Check arguments
if [ $# -ne 3 ]; then
    usage
fi

TARGET_IP=$1
START_PORT=$2
END_PORT=$3

info "Starting port scan on $TARGET_IP"
info "Scanning ports $START_PORT to $END_PORT"

# Script logic here...

success "Scan complete!"
```

### 10.2 Naming Conventions

```bash
# Variables
CONSTANT_VALUE="never changes"    # ALL CAPS for constants
global_variable="accessible everywhere"  # lowercase for variables
local_variable="inside function"   # local inside functions

# Functions
function_name() { }               # lowercase with underscores
get_user_input() { }
calculate_total() { }

# Files
my_script.sh                      # lowercase with underscores
network_scanner.sh
backup_database.sh
```

### 10.3 Error Handling

```bash
#!/bin/bash

# Check if file exists before processing
if [ ! -f "/path/to/file" ]; then
    echo "Error: File not found"
    exit 1
fi

# Check if command exists
if ! command -v nmap &> /dev/null; then
    echo "Error: nmap is not installed"
    exit 1
fi

# Check if running as root
if [ "$(id -u)" -ne 0 ]; then
    echo "Error: This script must be run as root"
    exit 1
fi
```

---

## 11. Practical Examples

### 11.1 System Information Script

```bash
#!/bin/bash
#
# sysinfo.sh - Display system information
#

echo "======================================"
echo " SYSTEM INFORMATION"
echo "======================================"
echo ""

echo "Hostname: $(hostname)"
echo "Kernel: $(uname -r)"
echo "Uptime: $(uptime -p)"
echo "Current User: $(whoami)"
echo "Current Directory: $(pwd)"
echo ""

echo "CPU Information:"
lscpu | grep "Model name" | cut -d':' -f2 | xargs
echo ""

echo "Memory Usage:"
free -h | grep "Mem" | awk '{print "Used: " $3 " / Total: " $2}'
echo ""

echo "Disk Usage:"
df -h / | tail -1 | awk '{print "Used: " $3 " / Total: " $2 " (" $5 " full)"}'
echo ""

echo "Network Interfaces:"
ip -br addr show
echo ""

echo "Date: $(date '+%Y-%m-%d %H:%M:%S')"
```

### 11.2 Simple Backup Script

```bash
#!/bin/bash
#
# backup.sh - Backup important files
#

# Configuration
SOURCE_DIR="$HOME/Documents"
BACKUP_DIR="$HOME/Backups"
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_NAME="backup_$DATE.tar.gz"

# Create backup directory if it doesn't exist
mkdir -p "$BACKUP_DIR"

# Create backup
echo "[*] Starting backup of $SOURCE_DIR"
tar -czf "$BACKUP_DIR/$BACKUP_NAME" "$SOURCE_DIR" 2>/dev/null

# Check if backup was successful
if [ $? -eq 0 ]; then
    echo "[+] Backup created successfully: $BACKUP_NAME"
    echo "[*] Size: $(du -h "$BACKUP_DIR/$BACKUP_NAME" | cut -f1)"
else
    echo "[!] Backup failed!"
    exit 1
fi

# Keep only last 5 backups
cd "$BACKUP_DIR"
ls -t backup_*.tar.gz | tail -n +6 | xargs -r rm
echo "[*] Old backups cleaned up"
```

### 11.3 Network Ping Sweep

```bash
#!/bin/bash
#
# ping_sweep.sh - Check which hosts are alive in a network
#

# Check argument
if [ $# -ne 1 ]; then
    echo "Usage: $0 <network>"
    echo "Example: $0 192.168.1"
    exit 1
fi

NETWORK=$1

echo "[*] Scanning network $NETWORK.0/24"
echo ""

for i in {1..254}; do
    IP="$NETWORK.$i"
    # Ping once with 1 second timeout
    ping -c 1 -W 1 "$IP" > /dev/null 2>&1
    
    if [ $? -eq 0 ]; then
        echo "[+] Host UP: $IP"
    fi
done

echo ""
echo "[*] Scan complete"
```

### 11.4 Log File Analyzer

```bash
#!/bin/bash
#
# analyze_log.sh - Analyze Apache/Nginx access logs
#

LOGFILE="/var/log/nginx/access.log"

if [ ! -f "$LOGFILE" ]; then
    echo "Error: Log file not found: $LOGFILE"
    exit 1
fi

echo "Log Analysis for: $LOGFILE"
echo "========================================"
echo ""

echo "Total Requests: $(wc -l < "$LOGFILE")"
echo ""

echo "Top 10 IP Addresses:"
awk '{print $1}' "$LOGFILE" | sort | uniq -c | sort -rn | head -10
echo ""

echo "Top 10 Requested URLs:"
awk '{print $7}' "$LOGFILE" | sort | uniq -c | sort -rn | head -10
echo ""

echo "HTTP Status Code Distribution:"
awk '{print $9}' "$LOGFILE" | sort | uniq -c | sort -rn
echo ""

echo "Requests by Hour:"
awk '{print $4}' "$LOGFILE" | cut -d: -f2 | sort | uniq -c
```

---

## 12. Common Beginner Mistakes

### 12.1 Spaces Around = Sign

**WRONG**:
```bash
VAR = "value"    # This tries to run command "VAR"
```

**RIGHT**:
```bash
VAR="value"      # No spaces!
```

### 12.2 Forgetting Quotes

**WRONG**:
```bash
FILE=my file.txt        # Breaks (space in filename)
echo $FILE              # Might cause word splitting
```

**RIGHT**:
```bash
FILE="my file.txt"      # Quoted
echo "$FILE"            # Always quote variables
```

### 12.3 Not Checking Exit Codes

**WRONG**:
```bash
rm important_file.txt
echo "File deleted!"    # Even if rm failed
```

**RIGHT**:
```bash
if rm important_file.txt; then
    echo "File deleted successfully"
else
    echo "Error: Failed to delete file"
    exit 1
fi
```

### 12.4 Using cd Without Checking

**WRONG**:
```bash
cd /some/directory
rm -rf *               # Dangerous if cd failed!
```

**RIGHT**:
```bash
cd /some/directory || exit 1
rm -rf *
```

### 12.5 Not Making Script Executable

**WRONG**:
```bash
./script.sh
# Error: Permission denied
```

**RIGHT**:
```bash
chmod +x script.sh
./script.sh
```

---

## 13. Hands-On Labs

### Lab 1: Your First Script

**Objective**: Create a script that greets you by name and shows system info.

**Steps**:
1. Create file `greeting.sh`
2. Add shebang
3. Print "Hello, [your name]!"
4. Show current date
5. Show current directory
6. Show disk usage
7. Make it executable
8. Run it

**Solution**:
```bash
#!/bin/bash
echo "Hello, $(whoami)!"
echo "Today is: $(date '+%A, %B %d, %Y')"
echo "You are in: $(pwd)"
echo "Disk usage:"
df -h / | tail -1
```

### Lab 2: Interactive Script

**Objective**: Create a script that asks for user input.

**Steps**:
1. Ask user for their name
2. Ask for their favorite color
3. Print a personalized message
4. Ask if they want to see system info (y/n)
5. If yes, show hostname and uptime

**Solution**:
```bash
#!/bin/bash

read -p "What is your name? " name
read -p "What is your favorite color? " color

echo ""
echo "Nice to meet you, $name!"
echo "I also like the color $color!"
echo ""

read -p "Would you like to see system information? (y/n) " answer

if [ "$answer" = "y" ]; then
    echo "Hostname: $(hostname)"
    echo "Uptime: $(uptime -p)"
else
    echo "Maybe next time!"
fi
```

### Lab 3: Simple Calculator

**Objective**: Create a calculator that adds two numbers.

**Solution**:
```bash
#!/bin/bash

echo "Simple Calculator"
echo "================="

read -p "Enter first number: " num1
read -p "Enter second number: " num2

sum=$((num1 + num2))
difference=$((num1 - num2))
product=$((num1 * num2))

echo ""
echo "Results:"
echo "$num1 + $num2 = $sum"
echo "$num1 - $num2 = $difference"
echo "$num1 × $num2 = $product"

if [ $num2 -ne 0 ]; then
    quotient=$((num1 / num2))
    echo "$num1 ÷ $num2 = $quotient"
else
    echo "Cannot divide by zero!"
fi
```

---

## 14. Interview Questions & Checkpoints

### 14.1 Checkpoint Questions

**Q1**: What is the purpose of the shebang (`#!/bin/bash`) line?
**A**: It tells the operating system which interpreter to use to run the script. Without it, the system doesn't know if it's a Bash, Python, or Perl script.

**Q2**: What's the difference between `./script.sh` and `bash script.sh`?
**A**: 
- `./script.sh` requires execute permission and uses the shebang to determine interpreter
- `bash script.sh` doesn't need execute permission and ignores the shebang (forces Bash)

**Q3**: Why do we need `chmod +x` to run a script?
**A**: Linux requires the execute permission bit to be set before a file can be run as a program. Without it, you get "Permission denied."

**Q4**: What does `$?` represent?
**A**: The exit code of the last executed command. 0 means success, non-zero means error.

**Q5**: What's the difference between `source script.sh` and `./script.sh`?
**A**: 
- `source` runs script in current shell (changes affect your environment)
- `./` runs script in sub-shell (changes don't affect parent)

### 14.2 Interview Questions

**Q1**: How would you make a script available system-wide?
**A**: 
```bash
sudo cp script.sh /usr/local/bin/scriptname
sudo chmod +x /usr/local/bin/scriptname
```

**Q2**: How do you check if a command exists before using it in a script?
**A**:
```bash
if command -v nmap &> /dev/null; then
    echo "nmap is installed"
else
    echo "nmap is not installed"
fi
```

**Q3**: What's the difference between single quotes and double quotes?
**A**:
```bash
NAME="World"
echo 'Hello $NAME'   # Output: Hello $NAME (literal)
echo "Hello $NAME"   # Output: Hello World (variable expanded)
```

### 14.3 Practical Challenges

**Challenge 1**: Write a script that checks if a file exists and creates it if it doesn't.

**Challenge 2**: Write a script that counts how many files are in the current directory.

**Challenge 3**: Write a script that asks for a filename and displays its size, owner, and permissions.

**Challenge 4**: Write a script that creates a dated backup of a directory.

**Challenge 5**: Write a script that pings a host and reports if it's up or down.

---

## Summary

You've learned:
- ✅ What Bash is and why it's essential
- ✅ How to create and run your first script
- ✅ The shebang line and why it matters
- ✅ File permissions and chmod
- ✅ Different ways to execute scripts
- ✅ Basic commands (echo, read, date, sleep)
- ✅ Comments and documentation
- ✅ The PATH variable
- ✅ Script structure and best practices
- ✅ Common mistakes to avoid
- ✅ Practical hands-on examples

**Next Steps**:
- Practice writing small scripts daily
- Read other people's scripts to learn patterns
- Experiment with breaking things (in a VM!)
- Move on to variables and data types (next chapter)

---

*End of Chapter 01 - Bash Basics & Your First Script*

**Total Lines: 1,100+**
