# Input, Output & Redirection: Controlling Data Flow

---
title: "Input, Output & Redirection"
parent: "[[03b_Bash_Scripting]]"
tags:
  - bash
  - redirection
  - pipes
  - stdin
  - stdout
  - stderr
  - streams
created: 2026-01-24
updated: 2026-01-24
difficulty: "Beginner to Intermediate"
---

> **Executive Summary**:
> Every program has three fundamental channels for data: input (where data comes from), output (where results go), and errors (where error messages go). Bash gives you incredible control over these streams through redirection. This chapter teaches you how to save command output to files, chain commands together with pipes, handle errors separately, and build powerful one-liners. By the end, you'll understand why `>` overwrites files, `>>` appends, `2>&1` redirects errors, and `|` creates pipelines - essential skills for automation and security scripting.

---

## Table of Contents

1. [Understanding Standard Streams](#1-understanding-standard-streams)
2. [Output Redirection (stdout)](#2-output-redirection-stdout)
3. [Input Redirection (stdin)](#3-input-redirection-stdin)
4. [Error Redirection (stderr)](#4-error-redirection-stderr)
5. [Combining Redirections](#5-combining-redirections)
6. [Pipes: Chaining Commands](#6-pipes-chaining-commands)
7. [Here Documents (<<)](#7-here-documents)
8. [Here Strings (<<<)](#8-here-strings)
9. [The tee Command](#9-the-tee-command)
10. [File Descriptors](#10-file-descriptors)
11. [Process Substitution](#11-process-substitution)
12. [Reading User Input](#12-reading-user-input)
13. [Practical Examples](#13-practical-examples)
14. [Common Mistakes](#14-common-mistakes)
15. [Hands-On Labs](#15-hands-on-labs)
16. [Interview Questions & Checkpoints](#16-interview-questions--checkpoints)

---

## 1. Understanding Standard Streams

**ABSOLUTE BEGINNER EXPLANATION**:
Imagine every program is like a factory. It has:
- An **entrance** where raw materials come in (STDIN - Standard Input)
- A **main exit** where finished products go out (STDOUT - Standard Output)  
- An **emergency exit** where defective products are discarded (STDERR - Standard Error)

### 1.1 The Three Streams

```
┌─────────────────────────────────────────────────┐
│                  PROGRAM                        │
│                                                 │
│  ┌────────────┐                                │
│  │   STDIN    │ ← Input (keyboard, file, pipe) │
│  │    (0)     │                                │
│  └──────┬─────┘                                │
│         │                                       │
│         ▼                                       │
│    ┌─────────┐                                 │
│    │Processing│                                 │
│    └────┬────┘                                 │
│         │                                       │
│    ┌────┴────┐                                 │
│    │         │                                  │
│    ▼         ▼                                  │
│  ┌────────┐ ┌────────┐                        │
│  │ STDOUT │ │ STDERR │                        │
│  │  (1)   │ │  (2)   │                        │
│  └───┬────┘ └───┬────┘                        │
│      │          │                               │
└──────┼──────────┼───────────────────────────────┘
       │          │
       ▼          ▼
   Terminal   Terminal
   (screen)   (screen)
```

**By Default**:
- **STDIN (0)**: Keyboard input
- **STDOUT (1)**: Normal output to screen
- **STDERR (2)**: Error messages to screen

### 1.2 Visual Example

**Normal command** (no redirection):

```bash
ls /home
```

**What happens**:
1. `ls` receives no input (STDIN empty)
2. `ls` produces output → STDOUT → Screen
3. If errors occur → STDERR → Screen

**Result**: You see the listing on your terminal.

### 1.3 Why Redirection Matters

**Without Redirection**:
```bash
nmap 192.168.1.0/24
# Output scrolls on screen and disappears
# Can't save results
# Can't analyze later
```

**With Redirection**:
```bash
nmap 192.168.1.0/24 > scan_results.txt
# Output saved to file
# Can analyze anytime
# Can share with others
```

---

## 2. Output Redirection (stdout)

**ABSOLUTE BEGINNER**: Redirection is like changing where water flows. Instead of flowing to the screen, we redirect it to a file.

### 2.1 Basic Output Redirection (>)

**Redirect output to file (OVERWRITES existing file)**:

```bash
# Send output to file
echo "Hello, World!" > output.txt

# Check the file
cat output.txt
# Output: Hello, World!
```

**What happened**:
1. `echo` produces output
2. `>` redirects STDOUT to `output.txt`
3. File is created (or overwritten if exists)

**WARNING**: `>` DESTROYS existing file content!

```bash
echo "First line" > file.txt
echo "Second line" > file.txt

cat file.txt
# Output: Second line
# (First line was erased!)
```

### 2.2 Append Output (>>)

**Append to file (PRESERVES existing content)**:

```bash
echo "First line" > file.txt
echo "Second line" >> file.txt
echo "Third line" >> file.txt

cat file.txt
# Output:
# First line
# Second line
# Third line
```

### 2.3 Practical Examples

**Logging scan results**:
```bash
#!/bin/bash

TARGET="192.168.1.1"
LOGFILE="scan.log"

# Clear old log
> "$LOGFILE"

# Add timestamp
echo "[$(date)] Starting scan of $TARGET" >> "$LOGFILE"

# Run scan and append results
echo "--- Ping Test ---" >> "$LOGFILE"
ping -c 3 "$TARGET" >> "$LOGFILE" 2>&1

echo "--- Port Scan ---" >> "$LOGFILE"
nmap -p 80,443 "$TARGET" >> "$LOGFILE" 2>&1

echo "[$(date)] Scan complete" >> "$LOGFILE"
```

**Building a report**:
```bash
#!/bin/bash

REPORT="system_report.txt"

# Create fresh report
echo "System Report" > "$REPORT"
echo "Generated: $(date)" >> "$REPORT"
echo "=====================" >> "$REPORT"
echo "" >> "$REPORT"

# Add sections
echo "Hostname: $(hostname)" >> "$REPORT"
echo "Kernel: $(uname -r)" >> "$REPORT"
echo "Uptime: $(uptime -p)" >> "$REPORT"
echo "" >> "$REPORT"

echo "Disk Usage:" >> "$REPORT"
df -h >> "$REPORT"
echo "" >> "$REPORT"

echo "Memory Usage:" >> "$REPORT"
free -h >> "$REPORT"
```

### 2.4 Redirecting to /dev/null

**The Black Hole** - discard output:

```bash
# Run command but hide output
ls /tmp > /dev/null

# Useful for checking if command succeeds without seeing output
if ping -c 1 192.168.1.1 > /dev/null 2>&1; then
    echo "Host is up"
else
    echo "Host is down"
fi
```

**What is /dev/null?**
- Special file that discards everything written to it
- Think of it as a trash can
- Reading from it returns nothing (EOF)

---

## 3. Input Redirection (stdin)

**ABSOLUTE BEGINNER**: Instead of typing input, we can feed it from a file.

### 3.1 Basic Input Redirection (<)

**Read from file instead of keyboard**:

```bash
# Create a file with data
cat > usernames.txt << EOF
admin
root
user1
user2
EOF

# Use file as input
while read username; do
    echo "Processing: $username"
done < usernames.txt
```

### 3.2 Practical Example - Processing a List

**File: targets.txt**
```
192.168.1.1
192.168.1.10
192.168.1.20
```

**Script: scan_targets.sh**
```bash
#!/bin/bash

while read TARGET; do
    echo "Scanning $TARGET..."
    ping -c 1 -W 1 "$TARGET" > /dev/null 2>&1
    
    if [ $? -eq 0 ]; then
        echo "  [+] Host UP"
    else
        echo "  [-] Host DOWN"
    fi
done < targets.txt
```

### 3.3 Using Input Redirection with Commands

```bash
# Count lines in file
wc -l < file.txt

# Sort a file
sort < unsorted.txt > sorted.txt

# Mail content of file
mail -s "Report" user@example.com < report.txt
```

### 3.4 Stdin in Scripts

**Interactive vs Non-Interactive**:

```bash
#!/bin/bash

# This waits for keyboard input
read -p "Enter your name: " NAME
echo "Hello, $NAME"

# This reads from file if provided
# Usage: ./script.sh < input.txt
read NAME
echo "Hello, $NAME"
```

---

## 4. Error Redirection (stderr)

**ABSOLUTE BEGINNER**: Error messages and normal output are SEPARATE streams. We can redirect them independently.

### 4.1 Understanding stderr

**Example without error redirection**:

```bash
ls /existing_dir /nonexistent_dir
```

**Output**:
```
/existing_dir:
file1.txt
file2.txt
ls: cannot access '/nonexistent_dir': No such file or directory
```

- First 3 lines → STDOUT (normal output)
- Last line → STDERR (error message)
- Both appear on screen, but from different streams!

### 4.2 Redirecting stderr (2>)

**Send errors to file**:

```bash
ls /existing_dir /nonexistent_dir 2> errors.txt
```

**Result**:
- Normal output appears on screen
- Errors saved to `errors.txt`

**Check errors file**:
```bash
cat errors.txt
# Output: ls: cannot access '/nonexistent_dir': No such file or directory
```

### 4.3 Redirect Both stdout and stderr

**Method 1: Separate files**:
```bash
command > output.txt 2> errors.txt
# Normal output → output.txt
# Errors → errors.txt
```

**Method 2: Same file**:
```bash
command > combined.txt 2>&1
# Both stdout and stderr → combined.txt
```

**EXPLANATION of `2>&1`**:
- `2>` = redirect stderr
- `&1` = to wherever stdout is going
- Order matters! `> file 2>&1` not `2>&1 > file`

### 4.4 Practical Examples

**Scan with error logging**:
```bash
#!/bin/bash

TARGET="192.168.1.0/24"
OUTPUT="scan_results.txt"
ERRORS="scan_errors.log"

# Redirect output and errors separately
nmap -p 1-1000 "$TARGET" > "$OUTPUT" 2> "$ERRORS"

# Check if errors occurred
if [ -s "$ERRORS" ]; then
    echo "Scan completed with errors. Check $ERRORS"
else
    echo "Scan completed successfully"
    rm "$ERRORS"  # Remove empty error log
fi
```

**Silent execution**:
```bash
# Discard both output and errors
command > /dev/null 2>&1

# Or shorter (Bash 4+):
command &> /dev/null
```

### 4.5 Append stderr

```bash
# Append errors to file
command 2>> errors.log

# Append both stdout and stderr
command >> output.log 2>&1
```

---

## 5. Combining Redirections

**ABSOLUTE BEGINNER**: You can redirect multiple streams in one command.

### 5.1 Common Patterns

**Pattern 1: Save output, show errors**
```bash
command > output.txt
# Errors still shown on screen
```

**Pattern 2: Save errors, show output**
```bash
command 2> errors.txt
# Normal output still shown on screen
```

**Pattern 3: Save both to same file**
```bash
command > combined.txt 2>&1
# Everything in one file
```

**Pattern 4: Save both to different files**
```bash
command > output.txt 2> errors.txt
# Separated for analysis
```

**Pattern 5: Discard everything**
```bash
command > /dev/null 2>&1
# Silent execution
```

### 5.2 Order Matters!

**WRONG**:
```bash
command 2>&1 > file.txt
# stderr copied to current stdout (terminal)
# then stdout redirected to file
# Result: errors on screen, output in file
```

**RIGHT**:
```bash
command > file.txt 2>&1
# stdout redirected to file
# then stderr copied to stdout (which is now the file)
# Result: both in file
```

### 5.3 Practical Script Template

```bash
#!/bin/bash

# Configuration
LOGFILE="/var/log/myscript.log"
ERRLOG="/var/log/myscript.err"

# Redirect all output for entire script
exec 1>> "$LOGFILE"  # stdout to logfile
exec 2>> "$ERRLOG"   # stderr to error log

# Now everything is logged automatically
echo "Script started at $(date)"
echo "This goes to log"

ls /nonexistent_directory  # Error goes to error log

echo "Script finished at $(date)"
```

---

## 6. Pipes: Chaining Commands

**ABSOLUTE BEGINNER**: Pipes (`|`) connect the output of one command to the input of another, like connecting water pipes.

### 6.1 Basic Pipes

```
┌──────────┐       ┌──────────┐       ┌──────────┐
│Command 1 │──────→│Command 2 │──────→│Command 3 │
│  stdout  │ pipe  │stdin│stdout│pipe │  stdin   │
└──────────┘       └──────────┘       └──────────┘
                                           │
                                           ▼
                                       Terminal
```

**Example**:
```bash
ls -l | grep ".txt" | wc -l
#  │      │            │
#  │      │            └─ Count lines
#  │      └─ Filter for .txt files
#  └─ List files
```

### 6.2 Common Pipe Patterns

**Filter and count**:
```bash
ps aux | grep firefox | wc -l
# Show processes → Filter for firefox → Count
```

**Sort and find unique**:
```bash
cat access.log | awk '{print $1}' | sort | uniq
# Read log → Extract IP → Sort → Remove duplicates
```

**Chain multiple filters**:
```bash
cat /var/log/auth.log | grep "Failed password" | grep -v "invalid user" | awk '{print $11}' | sort | uniq -c | sort -rn
# Read log → Failed passwords → Exclude invalid users → Extract IP → Sort → Count → Sort by count
```

### 6.3 Pipes vs Redirection

**Redirection** (to file):
```bash
command > file.txt
```

**Pipe** (to another command):
```bash
command1 | command2
```

### 6.4 Practical Examples

**Network scanning pipeline**:
```bash
#!/bin/bash

# Find live hosts in network
nmap -sn 192.168.1.0/24 | grep "Nmap scan report" | awk '{print $5}'
```

**Log analysis pipeline**:
```bash
#!/bin/bash

# Find most common error in logs
cat /var/log/syslog | grep ERROR | sort | uniq -c | sort -rn | head -10
```

**Process monitoring**:
```bash
#!/bin/bash

# Find top memory consumers
ps aux | sort -k4 -rn | head -10
```

### 6.5 Tee in Pipelines

**See output AND save it**:
```bash
command | tee output.txt | other_command
#           └─ Saves to file and passes to next command
```

**Example**:
```bash
nmap 192.168.1.1 | tee scan.txt | grep "open"
# Save full scan to scan.txt
# AND show only open ports on screen
```

---

## 7. Here Documents (<<)

**ABSOLUTE BEGINNER**: Here documents let you type multi-line input directly in your script, like a mini embedded file.

### 7.1 Basic Syntax

```bash
command << DELIMITER
line 1
line 2
line 3
DELIMITER
```

**Example - Create file**:
```bash
cat > config.txt << EOF
server_ip=192.168.1.1
port=8080
timeout=30
EOF
```

**Result**: Creates `config.txt` with 3 lines.

### 7.2 Variable Expansion in Here Docs

**With expansion** (default):
```bash
NAME="Alice"
cat << EOF
Hello, $NAME!
Your home is $HOME
EOF

# Output:
# Hello, Alice!
# Your home is /home/kali
```

**Without expansion** (quote delimiter):
```bash
NAME="Alice"
cat << 'EOF'
Hello, $NAME!
Your home is $HOME
EOF

# Output:
# Hello, $NAME!
# Your home is $HOME
```

### 7.3 Practical Examples

**Create script from script**:
```bash
#!/bin/bash

# Generate a scan script
cat > scan.sh << 'SCRIPT'
#!/bin/bash
TARGET=$1
echo "Scanning $TARGET..."
nmap -sV "$TARGET"
SCRIPT

chmod +x scan.sh
echo "Scan script created!"
```

**Send email with report**:
```bash
#!/bin/bash

RECIPIENT="admin@example.com"

mail -s "Daily Report" "$RECIPIENT" << EOF
Daily System Report
==================

Date: $(date)
Hostname: $(hostname)

Disk Usage:
$(df -h /)

Memory:
$(free -h | grep Mem)

Active Users:
$(who)

End of Report
EOF
```

**Create configuration file**:
```bash
#!/bin/bash

SERVER_IP="192.168.1.100"
PORT=8080

cat > /etc/myapp/config.conf << EOF
# Auto-generated configuration
# Generated: $(date)

[server]
ip = $SERVER_IP
port = $PORT
timeout = 30

[logging]
level = info
file = /var/log/myapp.log

[security]
enable_ssl = true
cert_path = /etc/ssl/certs/myapp.crt
EOF

echo "Configuration created"
```

### 7.4 Here Docs with Indentation

**Problem**: Indented here docs look messy.

**Solution**: Use `<<-` to strip leading tabs:

```bash
#!/bin/bash

if true; then
    cat <<- EOF
	This is indented with tabs
	But tabs are stripped from output
	EOF
fi
```

**NOTE**: Only works with **TABS**, not spaces!

---

## 8. Here Strings (<<<)

**ABSOLUTE BEGINNER**: Here strings are like here documents but for a single line.

### 8.1 Basic Syntax

```bash
command <<< "string"
```

**Example**:
```bash
# Count words in string
wc -w <<< "Hello world from Bash"
# Output: 4

# Convert to uppercase
tr 'a-z' 'A-Z' <<< "hello"
# Output: HELLO
```

### 8.2 Practical Examples

**Quick calculations**:
```bash
bc <<< "scale=2; 10/3"
# Output: 3.33
```

**Process a single line**:
```bash
#!/bin/bash

IP="192.168.1.100"

# Extract last octet
LAST_OCTET=$(cut -d'.' -f4 <<< "$IP")
echo "Last octet: $LAST_OCTET"
# Output: Last octet: 100
```

**Check if string matches pattern**:
```bash
#!/bin/bash

EMAIL="user@example.com"

if grep -q "@" <<< "$EMAIL"; then
    echo "Valid email format"
else
    echo "Invalid email"
fi
```

---

## 9. The tee Command

**ABSOLUTE BEGINNER**: `tee` is like a T-junction in plumbing - it splits the flow so it goes to TWO places.

### 9.1 Basic Usage

```
┌─────────┐
│ Command │
└────┬────┘
     │
     ▼
  ┌─────┐
  │ tee │
  └──┬──┘
     ├────→ file.txt
     │
     └────→ stdout (screen)
```

**Example**:
```bash
echo "Hello, World!" | tee output.txt
# Output appears on screen
# AND saved to output.txt
```

### 9.2 Append Mode

```bash
# Overwrite file
command | tee file.txt

# Append to file
command | tee -a file.txt
```

### 9.3 Multiple Outputs

```bash
# Save to multiple files
command | tee file1.txt file2.txt file3.txt
```

### 9.4 Practical Examples

**Log and display**:
```bash
#!/bin/bash

LOGFILE="install.log"

echo "Starting installation..." | tee "$LOGFILE"

apt-get update 2>&1 | tee -a "$LOGFILE"
apt-get install -y nmap 2>&1 | tee -a "$LOGFILE"

echo "Installation complete" | tee -a "$LOGFILE"
```

**Real-time monitoring**:
```bash
#!/bin/bash

# Watch log file and save matches
tail -f /var/log/syslog | grep --line-buffered "ERROR" | tee errors.txt
```

**Scan and save**:
```bash
#!/bin/bash

TARGET="192.168.1.0/24"

# Show scan progress AND save to file
nmap -v "$TARGET" | tee scan_results.txt

# Also works with multiple commands
nmap -v "$TARGET" | tee scan_results.txt | grep "open"
```

---

## 10. File Descriptors

**ABSOLUTE BEGINNER**: File descriptors are numbers that refer to open files or streams.

### 10.1 Standard Descriptors

```
0 = STDIN  (keyboard input)
1 = STDOUT (normal output)
2 = STDERR (error output)
```

**Usage**:
```bash
# These are equivalent:
command > file.txt
command 1> file.txt    # Explicit FD 1

# These are equivalent:
command 2> errors.txt
```

### 10.2 Custom File Descriptors

**Open file for reading** (FD 3):
```bash
exec 3< input.txt

# Read from FD 3
while read line <&3; do
    echo "Line: $line"
done

# Close FD 3
exec 3<&-
```

**Open file for writing** (FD 4):
```bash
exec 4> output.txt

# Write to FD 4
echo "Line 1" >&4
echo "Line 2" >&4

# Close FD 4
exec 4>&-
```

### 10.3 Practical Example - Logging

```bash
#!/bin/bash

# Open log file on FD 3
exec 3> script.log

# Function to log
log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" >&3
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1"
}

# Use logging
log "Script started"
log "Processing data..."
sleep 2
log "Script completed"

# Close log
exec 3>&-
```

### 10.4 Swapping stdout and stderr

```bash
# Swap stdout and stderr
command 3>&1 1>&2 2>&3 3>&-

# What happens:
# 3>&1  Save stdout to FD 3
# 1>&2  Redirect stdout to stderr
# 2>&3  Redirect stderr to saved stdout (FD 3)
# 3>&-  Close FD 3
```

---

## 11. Process Substitution

**ABSOLUTE BEGINNER**: Process substitution treats command output as if it were a file.

### 11.1 Basic Syntax

**Output as file** `<( command )`:
```bash
diff <(ls dir1) <(ls dir2)
# Compare contents of two directories
```

**What happens**:
- `<(ls dir1)` runs `ls dir1` and creates temporary "file"
- `diff` reads from these "files"

### 11.2 Practical Examples

**Compare remote and local files**:
```bash
diff <(ssh user@server cat /etc/config) /etc/config
```

**Process multiple sources**:
```bash
#!/bin/bash

# Compare hosts that are up vs down
comm <(nmap -sn 192.168.1.0/24 | grep "up" | sort) \
     <(cat known_hosts.txt | sort)
```

**Parallel processing**:
```bash
#!/bin/bash

# Count lines in multiple files simultaneously
paste <(wc -l file1.txt) <(wc -l file2.txt) <(wc -l file3.txt)
```

---

## 12. Reading User Input

**ABSOLUTE BEGINNER**: The `read` command gets input from the user (or from a file/pipe).

### 12.1 Basic Read

```bash
#!/bin/bash

echo "What is your name?"
read NAME
echo "Hello, $NAME!"
```

### 12.2 Read with Prompt

```bash
#!/bin/bash

read -p "Enter your username: " USERNAME
read -sp "Enter your password: " PASSWORD
echo  # New line after hidden password
echo "Username: $USERNAME"
```

**Options**:
- `-p "prompt"` : Show prompt
- `-s` : Silent (for passwords)
- `-t 10` : Timeout after 10 seconds
- `-n 1` : Read only 1 character

### 12.3 Read Multiple Variables

```bash
#!/bin/bash

read -p "Enter first and last name: " FIRST LAST
echo "First: $FIRST"
echo "Last: $LAST"
```

### 12.4 Read into Array

```bash
#!/bin/bash

echo "Enter three colors:"
read -a COLORS

echo "You entered:"
for COLOR in "${COLORS[@]}"; do
    echo "  - $COLOR"
done
```

### 12.5 Read from File

```bash
#!/bin/bash

while read LINE; do
    echo "Processing: $LINE"
done < data.txt
```

### 12.6 Read with IFS (Field Separator)

```bash
#!/bin/bash

# Parse CSV
while IFS=',' read NAME AGE CITY; do
    echo "Name: $NAME, Age: $AGE, City: $CITY"
done < users.csv
```

### 12.7 Timeout Example

```bash
#!/bin/bash

if read -t 5 -p "Enter password (5 sec timeout): " PASSWORD; then
    echo "Password received: $PASSWORD"
else
    echo "Timeout! Using default."
    PASSWORD="default"
fi
```

---

## 13. Practical Examples

### 13.1 Complete Logging Script

```bash
#!/bin/bash

# Configuration
LOGDIR="/var/log/myscripts"
LOGFILE="$LOGDIR/scan_$(date +%Y%m%d_%H%M%S).log"
ERRFILE="$LOGDIR/scan_errors.log"

# Create log directory
mkdir -p "$LOGDIR"

# Function to log with timestamp
log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOGFILE"
}

log_error() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] ERROR: $1" | tee -a "$LOGFILE" "$ERRFILE"
}

# Main script
log "Script started"

TARGET="192.168.1.0/24"
log "Scanning target: $TARGET"

# Run scan with all output logged
if nmap -sV "$TARGET" >> "$LOGFILE" 2>> "$ERRFILE"; then
    log "Scan completed successfully"
else
    log_error "Scan failed with exit code $?"
fi

log "Script finished"

# Show log location
echo "Log saved to: $LOGFILE"
```

### 13.2 Interactive Menu System

```bash
#!/bin/bash

while true; do
    cat << EOF

============================
   SECURITY TOOLS MENU
============================
1. Scan network
2. Check ports
3. View logs
4. Exit
============================
EOF

    read -p "Choose an option: " CHOICE

    case $CHOICE in
        1)
            read -p "Enter network (e.g., 192.168.1.0/24): " NETWORK
            nmap -sn "$NETWORK" | tee scan_results.txt
            ;;
        2)
            read -p "Enter target IP: " IP
            read -p "Enter ports (e.g., 80,443): " PORTS
            nmap -p "$PORTS" "$IP" | tee port_scan.txt
            ;;
        3)
            if [ -f scan_results.txt ]; then
                less scan_results.txt
            else
                echo "No logs found"
            fi
            ;;
        4)
            echo "Goodbye!"
            exit 0
            ;;
        *)
            echo "Invalid option"
            ;;
    esac
done
```

### 13.3 Bulk File Processor

```bash
#!/bin/bash

# Process all .log files
while read LOGFILE; do
    echo "Processing: $LOGFILE"
    
    # Count errors
    ERROR_COUNT=$(grep -c "ERROR" "$LOGFILE")
    
    # Generate report
    cat >> summary.txt << EOF
File: $LOGFILE
Errors: $ERROR_COUNT
Size: $(du -h "$LOGFILE" | cut -f1)
---
EOF

done < <(find /var/log -name "*.log" -type f)

echo "Summary created in summary.txt"
```

### 13.4 Password Strength Checker

```bash
#!/bin/bash

check_password() {
    local PASSWORD=$1
    local SCORE=0
    
    # Length check
    if [ ${#PASSWORD} -ge 8 ]; then
        ((SCORE++))
    fi
    
    # Uppercase check
    if grep -q "[A-Z]" <<< "$PASSWORD"; then
        ((SCORE++))
    fi
    
    # Lowercase check
    if grep -q "[a-z]" <<< "$PASSWORD"; then
        ((SCORE++))
    fi
    
    # Number check
    if grep -q "[0-9]" <<< "$PASSWORD"; then
        ((SCORE++))
    fi
    
    # Special char check
    if grep -q "[!@#$%^&*]" <<< "$PASSWORD"; then
        ((SCORE++))
    fi
    
    # Report strength
    case $SCORE in
        0|1|2)
            echo "Weak password (Score: $SCORE/5)"
            return 1
            ;;
        3)
            echo "Moderate password (Score: $SCORE/5)"
            return 0
            ;;
        4|5)
            echo "Strong password (Score: $SCORE/5)"
            return 0
            ;;
    esac
}

# Main
while true; do
    read -sp "Enter password to check (or 'quit'): " PASSWORD
    echo
    
    if [ "$PASSWORD" = "quit" ]; then
        break
    fi
    
    check_password "$PASSWORD"
    echo
done
```

---

## 14. Common Mistakes

### 14.1 Wrong Redirection Order

**WRONG**:
```bash
command 2>&1 > file.txt
# Errors go to terminal, output to file
```

**RIGHT**:
```bash
command > file.txt 2>&1
# Both go to file
```

### 14.2 Forgetting Quotes in Read

**WRONG**:
```bash
while read LINE; do
    echo $LINE    # Word splitting breaks spacing
done < file.txt
```

**RIGHT**:
```bash
while read LINE; do
    echo "$LINE"  # Preserves spacing
done < file.txt
```

### 14.3 Overwriting Important Files

**DANGEROUS**:
```bash
important_command > /etc/hosts
# Overwrites system file!
```

**SAFER**:
```bash
important_command > /tmp/output.txt
# Then verify before moving
```

### 14.4 Not Checking File Existence

**WRONG**:
```bash
cat data.txt | grep pattern
# Breaks if file doesn't exist
```

**RIGHT**:
```bash
if [ -f data.txt ]; then
    cat data.txt | grep pattern
else
    echo "Error: data.txt not found"
    exit 1
fi
```

### 14.5 Infinite Loops with Redirections

**WRONG**:
```bash
cat file.txt > file.txt
# File becomes empty!
```

**RIGHT**:
```bash
cat file.txt > temp.txt && mv temp.txt file.txt
```

---

## 15. Hands-On Labs

### Lab 1: Basic Redirection Practice

**Tasks**:
1. Create a file with 10 lines using echo and >>
2. Append 5 more lines
3. Redirect a command's output to file
4. Redirect errors to separate file

**Solution**:
```bash
#!/bin/bash

# Task 1: Create file with 10 lines
for i in {1..10}; do
    echo "Line $i" >> file.txt
done

# Task 2: Append 5 more lines
for i in {11..15}; do
    echo "Line $i" >> file.txt
done

# Task 3: Redirect command output
ls -la > directory_listing.txt

# Task 4: Redirect errors separately
ls /existing /nonexistent > output.txt 2> errors.txt

echo "Files created:"
ls -lh file.txt directory_listing.txt output.txt errors.txt
```

### Lab 2: Build a Log Analyzer

**Objective**: Create a script that reads a log file and generates statistics.

**Solution**:
```bash
#!/bin/bash

LOGFILE="/var/log/syslog"

if [ ! -f "$LOGFILE" ]; then
    echo "Error: $LOGFILE not found"
    exit 1
fi

# Create report
cat > log_report.txt << EOF
Log Analysis Report
===================
Generated: $(date)
Log File: $LOGFILE

Total Lines: $(wc -l < "$LOGFILE")

Top 10 Most Common Words:
$(cat "$LOGFILE" | tr ' ' '\n' | sort | uniq -c | sort -rn | head -10)

Error Count: $(grep -c "error" "$LOGFILE")
Warning Count: $(grep -c "warning" "$LOGFILE")

Recent Errors:
$(grep "error" "$LOGFILE" | tail -5)

End of Report
EOF

# Show report
cat log_report.txt

# Also save to dated file
cp log_report.txt "log_report_$(date +%Y%m%d).txt"
```

### Lab 3: Interactive Data Processor

**Objective**: Create a script that processes user input and saves results.

**Solution**:
```bash
#!/bin/bash

OUTPUT="processed_data.txt"

# Clear old output
> "$OUTPUT"

echo "Data Entry System"
echo "================="
echo "Enter data (type 'done' to finish)"
echo ""

while true; do
    read -p "Enter item: " ITEM
    
    if [ "$ITEM" = "done" ]; then
        break
    fi
    
    # Process and save
    UPPER=$(echo "$ITEM" | tr 'a-z' 'A-Z')
    LENGTH=${#ITEM}
    
    echo "$ITEM|$UPPER|$LENGTH" >> "$OUTPUT"
    echo "  Saved: $ITEM"
done

echo ""
echo "Data saved to $OUTPUT:"
column -t -s'|' "$OUTPUT"
```

---

## 16. Interview Questions & Checkpoints

### 16.1 Checkpoint Questions

**Q1**: What's the difference between `>` and `>>`?  
**A**: `>` overwrites file, `>>` appends to file.

**Q2**: What does `2>&1` mean?  
**A**: Redirect stderr (2) to wherever stdout (1) is currently going.

**Q3**: How do you discard all output?  
**A**: `command > /dev/null 2>&1` or `command &> /dev/null`

**Q4**: What does `|` (pipe) do?  
**A**: Connects stdout of one command to stdin of another.

**Q5**: What is the difference between `<` and `|`?  
**A**: `<` reads from file, `|` reads from command output.

### 16.2 Interview Questions

**Q1**: How would you save both output and errors to the same file?  
**A**: `command > file.txt 2>&1` or `command &> file.txt`

**Q2**: Explain what this does: `cat file.txt | grep pattern | sort | uniq -c | sort -rn`  
**A**: Reads file, filters lines with pattern, sorts them, counts duplicates, sorts by count (highest first).

**Q3**: How do you create a multi-line file from a script?  
**A**: Use here document:
```bash
cat > file.txt << EOF
line 1
line 2
EOF
```

**Q4**: What's wrong with: `cat file.txt > file.txt`?  
**A**: File is opened for writing (truncated) before reading, resulting in empty file.

**Q5**: How can you see command output AND save it to a file?  
**A**: `command | tee output.txt`

### 16.3 Practical Challenges

**Challenge 1**: Write a command that finds all .txt files, counts lines in each, and saves summary to file.

**Challenge 2**: Create a script that reads from a file, processes each line, and outputs to different files based on content.

**Challenge 3**: Build a logging function that timestamps all messages and saves to file while displaying on screen.

**Challenge 4**: Write a script that takes input from user, validates it, and saves only valid entries to file.

---

## Summary

You've learned:
- ✅ The three standard streams (stdin, stdout, stderr)
- ✅ Output redirection (`>`, `>>`)
- ✅ Input redirection (`<`)
- ✅ Error redirection (`2>`, `2>&1`)
- ✅ Pipes for chaining commands (`|`)
- ✅ Here documents (`<<`) and here strings (`<<<`)
- ✅ The `tee` command for splitting output
- ✅ File descriptors and custom streams
- ✅ Process substitution
- ✅ Reading user input with `read`
- ✅ Practical logging and data processing
- ✅ Common mistakes and how to avoid them

**Next**: Learn conditional logic (if/else/case) to make your scripts make decisions!

---

*End of Chapter 03 - Input, Output & Redirection*

**Total Lines: 1,150+**
