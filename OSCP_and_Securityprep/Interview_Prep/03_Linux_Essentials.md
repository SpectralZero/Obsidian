# Interview Preparation: Linux Essentials

> **Purpose**: Master essential Linux concepts for cybersecurity interviews. Covers commands, distributions, file system, permissions, user management, and administration - everything you need to demonstrate Linux proficiency.

**What You'll Master**:
- Essential Linux commands with practical examples
- Linux distributions - what to use and why
- File system hierarchy and navigation
- Permissions and ownership (chmod, chown, umask)
- User and group management
- Process management and monitoring
- Package management (apt, yum, dnf)
- Shell scripting basics
- Security hardening fundamentals
- 80+ interview questions with detailed answers

---

## Table of Contents

1. [Why Linux for Cybersecurity?](#1-why-linux-for-cybersecurity)
2. [Linux Distributions Comparison](#2-linux-distributions-comparison)
3. [File System Hierarchy](#3-file-system-hierarchy)
4. [Essential Commands - Navigation](#4-essential-commands-navigation)
5. [File Operations](#5-file-operations)
6. [Text Processing Commands](#6-text-processing-commands)
7. [File Permissions & Ownership](#7-file-permissions-ownership)
8. [User & Group Management](#8-user-group-management)
9. [Process Management](#9-process-management)
10. [Network Commands](#10-network-commands)
11. [Package Management](#11-package-management)
12. [System Information](#12-system-information)
13. [Log Files & Monitoring](#13-log-files-monitoring)
14. [Basic Shell Scripting](#14-basic-shell-scripting)
15. [Security Best Practices](#15-security-best-practices)
16. [Interview Questions & Answers](#16-interview-questions-answers)

---

## 1. Why Linux for Cybersecurity?

### 1.1 Why Learn Linux?

**Essential for Cybersecurity**:
- ✅ **90% of servers** run Linux (web servers, databases, cloud)
- ✅ **All penetration testing tools** (Kali Linux, Parrot OS)
- ✅ **Cloud platforms** (AWS, Azure, GCP) are Linux-based
- ✅ **Security tools** (Nmap, Metasploit, Wireshark) built for Linux
- ✅ **Docker/Kubernetes** run on Linux
- ✅ **Open source** - free, customizable, transparent

**Industry Standard**:
- Security Operations Centers (SOC) use Linux
- Red Team/Blue Team tools are Linux-native
- OSCP exam requires Linux knowledge
- Most enterprise servers are Linux

**Power and Control**:
- Command-line efficiency (faster than GUI)
- Automation with shell scripts
- Fine-grained control over system
- Remote administration (SSH)

---

## 2. Linux Distributions Comparison

### 2.1 Popular Distributions

| Distribution       | Based On    | Use Case                   | Package Manager | Why Choose It?                                                        |
| ------------------ | ----------- | -------------------------- | --------------- | --------------------------------------------------------------------- |
| **Kali Linux**     | Debian      | Penetration Testing        | apt             | Pre-installed security tools (600+), industry standard for pentesting |
| **Ubuntu**         | Debian      | Desktop, Servers, Learning | apt             | Beginner-friendly, huge community, extensive documentation            |
| **Debian**         | Independent | Servers, Stability         | apt             | Rock-solid stability, minimal, great for servers                      |
| **CentOS / Rocky** | RHEL        | Enterprise Servers         | yum/dnf         | Enterprise stability, free RHEL clone, corporate environments         |
| **Fedora**         | Independent | Desktop, Development       | dnf             | Cutting-edge features, Red Hat backed, stable but modern              |
| **Arch Linux**     | Independent | Advanced Users             | pacman          | Rolling release, highly customizable, bleeding edge                   |
| **Parrot OS**      | Debian      | Security, Privacy          | apt             | Alternative to Kali, privacy-focused, lighter weight                  |
| **Alpine Linux**   | Independent | Containers, Minimal        | apk             | Tiny (5 MB), secure, Docker containers                                |

### 2.2 Debian-based vs Red Hat-based

**Debian Family** (Ubuntu, Kali, Debian, Mint):
```bash
Package Manager: apt (Advanced Package Tool)

Commands:
sudo apt update                    # Update package lists
sudo apt upgrade                   # Upgrade installed packages
sudo apt install nginx             # Install package
sudo apt remove nginx              # Remove package
sudo apt search apache             # Search for packages
sudo apt autoremove                # Remove unused dependencies

Configuration: /etc/apt/sources.list
Package Format: .deb
```

**Red Hat Family** (CentOS, Fedora, RHEL, Rocky):
```bash
Package Manager: yum (older) / dnf (newer)

Commands:
sudo dnf update                    # Update all packages
sudo dnf install nginx             # Install package
sudo dnf remove nginx              # Remove package
sudo dnf search apache             # Search for packages
sudo dnf autoremove                # Remove unused dependencies

Configuration: /etc/yum.repos.d/
Package Format: .rpm
```

### 2.3 Interview Answer: "What Linux Distribution Do You Use and Why?"

**Example Answer 1** (Security Focus):
> "I primarily use **Kali Linux** for penetration testing and security research because it comes pre-installed with 600+ security tools like Nmap, Metasploit, Burp Suite, and Wireshark. It's based on Debian, so I'm comfortable with apt package management. For production servers, I prefer **Ubuntu LTS** because of its stability, 5-year support cycle, and extensive documentation. For personal learning and staying current with new features, I sometimes use **Fedora** since it's cutting-edge but still stable."

**Example Answer 2** (Enterprise Focus):
> "In my work environment, we use **CentOS/Rocky Linux** for production servers because it's a free RHEL clone, offering enterprise-level stability and 10-year support. For development and testing, I use **Ubuntu** because of its ease of use and vast community support. I also have experience with **Kali Linux** for security assessments and vulnerability testing."

**Example Answer 3** (Learning Focus):
> "I started with **Ubuntu** because it's beginner-friendly with great documentation. As I became more comfortable, I explored **Kali Linux** for cybersecurity tools and **Debian** for understanding minimalist, stable systems. I chose these because Ubuntu teaches good fundamentals, Kali is essential for security work, and Debian helps me understand the foundation that Ubuntu builds on."

**Key Points to Mention**:
- ✅ Name specific distributions
- ✅ Explain WHY (use case, features)
- ✅ Show progression (beginner → advanced)
- ✅ Mention work vs personal use
- ✅ Relate to job requirements

---

## 3. File System Hierarchy

### 3.1 Linux Directory Structure

```
/ (root)
├── bin/          → Essential user binaries (ls, cat, cp, mv)
├── boot/         → Boot loader files (kernel, grub)
├── dev/          → Device files (hard drives, USB, terminals)
├── etc/          → Configuration files (system-wide)
├── home/         → User home directories (/home/username)
├── lib/          → Shared libraries (like .dll in Windows)
├── media/        → Mount points for removable media (USB, CD)
├── mnt/          → Temporary mount points
├── opt/          → Optional software packages
├── proc/         → Virtual filesystem (process info, system stats)
├── root/         → Root user's home directory
├── run/          → Runtime data (PIDs, sockets)
├── sbin/         → System binaries (admin commands: fdisk, iptables)
├── srv/          → Service data (web server files, FTP)
├── sys/          → Virtual filesystem (kernel/hardware info)
├── tmp/          → Temporary files (cleared on reboot)
├── usr/          → User programs and data
│   ├── bin/      → User binaries (non-essential)
│   ├── lib/      → Libraries for /usr/bin
│   ├── local/    → Locally compiled software
│   └── share/    → Shared data (docs, icons)
└── var/          → Variable data (logs, databases, mail)
    ├── log/      → Log files (/var/log/syslog, /var/log/auth.log)
    ├── www/      → Web server files
    └── mail/     → Email storage
```

### 3.2 Important Directories Explained

**/etc** - Configuration Files:
```bash
/etc/passwd          # User account information
/etc/shadow          # Encrypted passwords (root only)
/etc/group           # Group information
/etc/hosts           # Static hostname to IP mappings
/etc/hostname        # System hostname
/etc/ssh/sshd_config # SSH server configuration
/etc/apache2/        # Apache web server config
/etc/network/        # Network configuration
/etc/fstab           # Filesystem mount table
```

**/var/log** - Log Files:
```bash
/var/log/syslog      # System log (Debian/Ubuntu)
/var/log/messages    # System log (Red Hat/CentOS)
/var/log/auth.log    # Authentication logs (login attempts)
/var/log/apache2/    # Web server logs
/var/log/mysql/      # Database logs
/var/log/kern.log    # Kernel logs
```

**/proc** - Process and System Info:
```bash
/proc/cpuinfo        # CPU information
/proc/meminfo        # Memory information
/proc/[PID]/         # Process-specific info
/proc/sys/           # Kernel parameters (tunable)
```

**Home Directory Structure**:
```bash
/home/username/
├── .bashrc          # Bash shell configuration
├── .bash_history    # Command history
├── .ssh/            # SSH keys and config
│   ├── id_rsa       # Private key
│   ├── id_rsa.pub   # Public key
│   └── authorized_keys  # Allowed public keys
├── Documents/
├── Downloads/
└── .config/         # Application configurations
```

---

## 4. Essential Commands - Navigation

### 4.1 Directory Navigation

**pwd** - Print Working Directory
```bash
pwd
# Output: /home/username/Documents
# Shows your current location
```

**ls** - List Directory Contents
```bash
ls                   # List files in current directory
ls -l                # Long format (permissions, owner, size, date)
ls -a                # Show hidden files (files starting with .)
ls -la               # Long format + hidden files
ls -lh               # Human-readable sizes (KB, MB, GB)
ls -lt               # Sort by modification time (newest first)
ls -lS               # Sort by size (largest first)
ls -R                # Recursive (show subdirectories)
ls /etc              # List specific directory

Example output (ls -l):
-rw-r--r-- 1 user group 4096 Feb 14 10:30 file.txt
│││││││││  │ │    │     │    │           └─ filename
│││││││││  │ │    │     │    └─ modification date/time
│││││││││  │ │    │     └─ size in bytes
│││││││││  │ │    └─ group owner
│││││││││  │ └─ user owner
│││││││││  └─ number of hard links
││││││││└─ others permissions (r--)
│││││└─ group permissions (r--)
││└─ owner permissions (rw-)
└─ file type (- = file, d = directory, l = link)
```

**cd** - Change Directory
```bash
cd /etc              # Go to /etc directory
cd ~                 # Go to home directory (/home/username)
cd                   # Same as cd ~ (go home)
cd ..                # Go up one directory
cd ../..             # Go up two directories
cd -                 # Go to previous directory
cd Documents/        # Go to Documents in current directory
cd /var/log/apache2  # Absolute path
```

**tree** - Display Directory Structure
```bash
tree                 # Show directory tree
tree -L 2            # Limit depth to 2 levels
tree -a              # Include hidden files
tree -d              # Directories only

Example output:
.
├── dir1
│   ├── file1.txt
│   └── file2.txt
└── dir2
    └── file3.txt
```

---

## 5. File Operations

### 5.1 Creating Files and Directories

**touch** - Create Empty File / Update Timestamp
```bash
touch file.txt                 # Create empty file
touch file1.txt file2.txt      # Create multiple files
touch -t 202401011200 file.txt # Set specific timestamp
```

**mkdir** - Make Directory
```bash
mkdir mydir                    # Create directory
mkdir -p dir1/dir2/dir3        # Create parent directories (-p)
mkdir dir1 dir2 dir3           # Create multiple directories
mkdir -m 755 mydir             # Create with specific permissions
```

**echo** - Print Text / Create Files
```bash
echo "Hello World"             # Print to screen
echo "Text" > file.txt         # Overwrite file with text
echo "More" >> file.txt        # Append to file
echo $HOME                     # Print variable value
```

### 5.2 Copying, Moving, Deleting

**cp** - Copy Files/Directories
```bash
cp file.txt backup.txt         # Copy file
cp file.txt /tmp/              # Copy to directory
cp -r dir1/ dir2/              # Copy directory recursively
cp -p file.txt backup.txt      # Preserve permissions/timestamps
cp -i file.txt dest.txt        # Interactive (ask before overwrite)
cp -v file.txt /tmp/           # Verbose (show what's being copied)
cp *.txt /backup/              # Copy all .txt files
```

**mv** - Move/Rename Files
```bash
mv file.txt newname.txt        # Rename file
mv file.txt /tmp/              # Move file to directory
mv dir1/ /opt/                 # Move directory
mv -i file.txt dest.txt        # Interactive (ask before overwrite)
mv *.log /var/log/             # Move all .log files
```

**rm** - Remove Files/Directories
```bash
rm file.txt                    # Remove file
rm -i file.txt                 # Interactive (ask confirmation)
rm -f file.txt                 # Force (no confirmation)
rm -r dir1/                    # Remove directory recursively
rm -rf dir1/                   # Force remove directory (DANGEROUS!)
rm *.tmp                       # Remove all .tmp files
rm -v file.txt                 # Verbose (show what's being deleted)

⚠️ WARNING: rm -rf / (deletes everything - NEVER RUN!)
```

**ln** - Create Links
```bash
ln file.txt hardlink.txt       # Hard link (same inode)
ln -s file.txt symlink.txt     # Symbolic link (like shortcut)
ln -s /var/log/syslog ~/log    # Symlink to file in home directory
```

### 5.3 Viewing File Contents

**cat** - Concatenate / Display File
```bash
cat file.txt                   # Display entire file
cat file1.txt file2.txt        # Display multiple files
cat -n file.txt                # Show line numbers
cat > newfile.txt              # Create file (Ctrl+D to save)
cat file1.txt file2.txt > merged.txt  # Merge files
```

**less** - View File with Pagination
```bash
less file.txt                  # View file (scroll with arrows)
# Navigation:
# Space = next page
# b = previous page
# / = search forward
# ? = search backward
# q = quit
```

**more** - View File (Basic Pagination)
```bash
more file.txt                  # View file (less features than 'less')
```

**head** - View First Lines
```bash
head file.txt                  # First 10 lines
head -n 20 file.txt            # First 20 lines
head -5 file.txt               # First 5 lines
```

**tail** - View Last Lines
```bash
tail file.txt                  # Last 10 lines
tail -n 20 file.txt            # Last 20 lines
tail -f /var/log/syslog        # Follow (real-time updates)
tail -f -n 50 /var/log/apache2/access.log  # Follow last 50 lines
```

---

## 6. Text Processing Commands

### 6.1 Searching and Filtering

**grep** - Search Text Patterns
```bash
grep "error" file.txt          # Search for "error" in file
grep -i "error" file.txt       # Case-insensitive
grep -r "password" /etc/       # Recursive search in directory
grep -v "success" file.txt     # Invert (show lines NOT matching)
grep -n "error" file.txt       # Show line numbers
grep -c "error" file.txt       # Count matching lines
grep -l "error" *.log          # List files containing match
grep -E "error|warning" file.txt  # Extended regex (multiple patterns)

# Real-world examples:
grep "Failed password" /var/log/auth.log        # Find failed login attempts
grep "192.168.1.100" /var/log/apache2/access.log  # Find IP in web logs
grep -r "TODO" ~/code/                          # Find TODO comments
```

**find** - Find Files/Directories
```bash
find /home -name "*.txt"       # Find all .txt files in /home
find . -name "file.txt"        # Find file.txt in current directory
find /var -type f -mtime -7    # Files modified in last 7 days
find /tmp -type f -size +100M  # Files larger than 100 MB
find . -perm 777               # Files with 777 permissions
find /home -user john          # Files owned by user john
find . -name "*.log" -delete   # Find and delete .log files

# Powerful combinations:
find . -name "*.txt" -exec grep "password" {} \;  # Find and search
find /var/log -name "*.log" -mtime +30 -delete    # Delete old logs
```

**wc** - Word Count
```bash
wc file.txt                    # Lines, words, bytes
wc -l file.txt                 # Count lines only
wc -w file.txt                 # Count words only
wc -c file.txt                 # Count bytes
wc -m file.txt                 # Count characters

# Examples:
cat /etc/passwd | wc -l        # Count number of users
grep "error" app.log | wc -l   # Count errors in log
```

### 6.2 Text Manipulation

**cut** - Extract Columns
```bash
cut -d: -f1 /etc/passwd        # Extract usernames (1st field, delimiter :)
cut -d: -f1,6 /etc/passwd      # Username and home directory
cut -c1-10 file.txt            # Extract characters 1-10

# Real-world:
cut -d' ' -f1 /var/log/apache2/access.log | sort | uniq -c  # Count IPs
```

**sort** - Sort Lines
```bash
sort file.txt                  # Sort alphabetically
sort -r file.txt               # Reverse sort
sort -n numbers.txt            # Numeric sort
sort -u file.txt               # Sort and remove duplicates
sort -k2 file.txt              # Sort by 2nd column

# Example:
du -h /var/log/* | sort -h     # Sort files by size (human-readable)
```

**uniq** - Remove Duplicates
```bash
uniq file.txt                  # Remove adjacent duplicates (use with sort)
uniq -c file.txt               # Count occurrences
uniq -d file.txt               # Show only duplicates
uniq -u file.txt               # Show only unique lines

# Common pattern:
sort file.txt | uniq -c | sort -nr  # Count and sort by frequency
```

**sed** - Stream Editor
```bash
sed 's/old/new/' file.txt      # Replace first occurrence per line
sed 's/old/new/g' file.txt     # Replace all occurrences (global)
sed -i 's/old/new/g' file.txt  # Edit file in-place
sed -n '10,20p' file.txt       # Print lines 10-20
sed '/pattern/d' file.txt      # Delete lines matching pattern

# Examples:
sed 's/http:/https:/g' urls.txt           # Change http to https
sed -i 's/password123/REDACTED/g' log.txt # Redact passwords
```

**awk** - Pattern Scanning
```bash
awk '{print $1}' file.txt      # Print first column
awk -F: '{print $1}' /etc/passwd  # Custom delimiter (:)
awk '{print $1, $3}' file.txt  # Print columns 1 and 3
awk '$3 > 100' file.txt        # Print lines where column 3 > 100

# Real-world:
awk '{print $1}' /var/log/apache2/access.log | sort | uniq -c  # Count IPs
ps aux | awk '{print $11}' | sort | uniq  # List running programs
```

---

## 7. File Permissions & Ownership

### 7.1 Understanding Permissions

**Permission Types**:
```
r = read    (4)
w = write   (2)
x = execute (1)

Permission Groups:
Owner (u) - Group (g) - Others (o)

Example: -rwxr-xr--
         │││││││││
         │││└┴┴┴┴┴─ Others: r-- (read only) = 4
         ││└─┴┴┴─── Group:  r-x (read, execute) = 5
         │└──┴┴──── Owner:  rwx (read, write, execute) = 7
         └────────── File type: - (regular file)

Numeric: 754
Owner: 7 (rwx) = 4+2+1
Group: 5 (r-x) = 4+0+1
Others: 4 (r--) = 4+0+0
```

**File Type Indicators**:
```
-  = Regular file
d  = Directory
l  = Symbolic link
c  = Character device
b  = Block device
s  = Socket
p  = Named pipe
```

### 7.2 chmod - Change Permissions

**Symbolic Mode**:
```bash
chmod u+x file.sh              # Add execute for owner
chmod g-w file.txt             # Remove write for group
chmod o+r file.txt             # Add read for others
chmod a+x script.sh            # Add execute for all (a=all)
chmod u=rwx,g=rx,o=r file.txt  # Set specific permissions

# Examples:
chmod +x script.sh             # Make script executable
chmod -R 755 /var/www/html     # Recursive (directories and files)
```

**Numeric Mode**:
```bash
chmod 644 file.txt             # rw-r--r-- (common for files)
chmod 755 script.sh            # rwxr-xr-x (common for scripts)
chmod 700 private.txt          # rwx------ (owner only)
chmod 777 public/              # rwxrwxrwx (everyone, DANGEROUS!)
chmod 400 id_rsa               # r-------- (SSH private key)

# Common permission patterns:
# 644 - Files (owner read/write, others read)
# 755 - Directories and executables
# 700 - Private directories
# 600 - Private files (SSH keys, config files)
# 777 - Full access (avoid in production!)
```

### 7.3 chown - Change Ownership

```bash
chown user file.txt            # Change owner
chown user:group file.txt      # Change owner and group
chown :group file.txt          # Change group only
chown -R user:group /var/www/  # Recursive

# Real-world examples:
sudo chown www-data:www-data /var/www/html/*  # Web server files
sudo chown -R mysql:mysql /var/lib/mysql/     # Database files
```

### 7.4 chgrp - Change Group

```bash
chgrp developers file.txt      # Change group to developers
chgrp -R team /project/        # Recursive
```

### 7.5 umask - Default Permissions

```bash
umask                          # Show current umask (e.g., 0022)
umask 0027                     # Set umask

# umask 0022:
# New files:       666 - 022 = 644 (rw-r--r--)
# New directories: 777 - 022 = 755 (rwxr-xr-x)

# umask 0077:
# New files:       666 - 077 = 600 (rw-------)
# New directories: 777 - 077 = 700 (rwx------)
```

### 7.6 Special Permissions

**SUID (Set User ID)** - Run as file owner:
```bash
chmod u+s /usr/bin/passwd      # Numeric: 4755
ls -l /usr/bin/passwd
-rwsr-xr-x  # 's' in owner execute position

Example: passwd command runs as root to change passwords
```

**SGID (Set Group ID)** - Run as group / Inherit group:
```bash
chmod g+s /shared/directory    # Numeric: 2755
# New files inherit directory's group
```

**Sticky Bit** - Only owner can delete:
```bash
chmod +t /tmp                  # Numeric: 1777
ls -ld /tmp
drwxrwxrwt  # 't' in others execute position

Example: /tmp directory - anyone can write, only owner can delete their files
```

---

## 8. User & Group Management

### 8.1 User Information

**whoami** - Current User
```bash
whoami                         # Display current username
```

**id** - User ID Information
```bash
id                             # Show UID, GID, groups
id username                    # Show info for specific user
id -u                          # Show UID only
id -g                          # Show GID only
id -Gn                         # Show group names

Example output:
uid=1000(john) gid=1000(john) groups=1000(john),27(sudo),33(www-data)
```

**who** - Logged In Users
```bash
who                            # Show logged-in users
who -a                         # Detailed information
w                              # Who + what they're doing
```

**last** - Login History
```bash
last                           # Recent logins
last -n 10                     # Last 10 logins
last username                  # Specific user's logins
lastb                          # Failed login attempts (root only)
```

### 8.2 User Management

**useradd** - Create User
```bash
sudo useradd john              # Create user (minimal)
sudo useradd -m -s /bin/bash john  # Create with home directory and shell
sudo useradd -m -G sudo,developers john  # Add to groups
sudo useradd -m -e 2024-12-31 john  # Set expiration date

# Common pattern (Ubuntu/Debian):
sudo adduser john              # Interactive user creation (recommended)
```

**usermod** - Modify User
```bash
sudo usermod -aG sudo john     # Add user to sudo group
sudo usermod -s /bin/zsh john  # Change shell
sudo usermod -L john           # Lock account
sudo usermod -U john           # Unlock account
sudo usermod -d /new/home john # Change home directory
```

**userdel** - Delete User
```bash
sudo userdel john              # Delete user (keep home)
sudo userdel -r john           # Delete user and home directory
```

**passwd** - Change Password
```bash
passwd                         # Change your password
sudo passwd john               # Change john's password (root)
sudo passwd -l john            # Lock account
sudo passwd -u john            # Unlock account
sudo passwd -e john            # Expire password (force change)
```

### 8.3 Group Management

**groups** - Show Group Membership
```bash
groups                         # Show your groups
groups username                # Show user's groups
```

**groupadd** - Create Group
```bash
sudo groupadd developers       # Create group
sudo groupadd -g 5000 team     # Create with specific GID
```

**groupmod** - Modify Group
```bash
sudo groupmod -n newname oldname  # Rename group
```

**groupdel** - Delete Group
```bash
sudo groupdel developers       # Delete group
```

**gpasswd** - Manage Group Membership
```bash
sudo gpasswd -a john developers   # Add user to group
sudo gpasswd -d john developers   # Remove user from group
sudo gpasswd -A john developers   # Make john group admin
```

### 8.4 Important Files

**/etc/passwd** - User Accounts:
```bash
cat /etc/passwd
# Format: username:x:UID:GID:comment:home:shell
john:x:1000:1000:John Doe:/home/john:/bin/bash
root:x:0:0:root:/root:/bin/bash

# x = password in /etc/shadow
```

**/etc/shadow** - Encrypted Passwords (root only):
```bash
sudo cat /etc/shadow
# Format: username:encrypted_password:last_changed:min:max:warn:inactive:expire
john:$6$rounds=656000$...:19000:0:99999:7:::
```

**/etc/group** - Groups:
```bash
cat /etc/group
# Format: groupname:x:GID:members
sudo:x:27:john,alice
developers:x:1001:john,bob
```

---

## 9. Process Management

### 9.1 Viewing Processes

**ps** - Process Status
```bash
ps                             # Processes in current shell
ps aux                         # All processes (BSD style)
ps -ef                         # All processes (Unix style)
ps aux | grep apache           # Find specific process
ps -u username                 # Processes for specific user
ps --forest                    # Process tree

# Common columns:
# PID   = Process ID
# %CPU  = CPU usage
# %MEM  = Memory usage
# VSZ   = Virtual memory size
# RSS   = Resident set size (physical memory)
# TTY   = Terminal
# STAT  = Process state (R=running, S=sleeping, Z=zombie)
# START = Start time
# TIME  = CPU time
# COMMAND = Command name
```

**top** - Real-time Process Monitor
```bash
top                            # Interactive process viewer
# Navigation:
# q = quit
# k = kill process
# r = renice process
# M = sort by memory
# P = sort by CPU
# u = filter by user
```

**htop** - Enhanced top (install first)
```bash
htop                           # Better interface, colors, mouse support
```

**pgrep** - Find Process by Name
```bash
pgrep apache2                  # Show PIDs of apache2 processes
pgrep -u john                  # Processes owned by john
pgrep -l ssh                   # Show PID and process name
```

### 9.2 Managing Processes

**kill** - Terminate Process
```bash
kill 1234                      # Send SIGTERM (graceful shutdown)
kill -9 1234                   # Send SIGKILL (force kill)
kill -15 1234                  # SIGTERM (default, same as kill 1234)
kill -l                        # List all signals

# Common signals:
# 1  (HUP)  = Hangup (reload config)
# 9  (KILL) = Force kill (cannot be caught)
# 15 (TERM) = Terminate gracefully
# 18 (CONT) = Continue if stopped
# 19 (STOP) = Stop process
```

**pkill** - Kill by Name
```bash
pkill apache2                  # Kill all apache2 processes
pkill -u john                  # Kill all processes by user john
pkill -9 firefox               # Force kill firefox
```

**killall** - Kill All Instances
```bash
killall apache2                # Kill all apache2 processes
killall -9 chrome              # Force kill all chrome processes
```

**nice** - Start Process with Priority
```bash
nice -n 10 command             # Start with priority 10 (lower priority)
nice -n -10 command            # Higher priority (root only)
# Priority range: -20 (highest) to 19 (lowest)
# Default: 0
```

**renice** - Change Running Process Priority
```bash
renice -n 10 -p 1234           # Change PID 1234 to priority 10
renice -n 5 -u john            # Change all john's processes
```

### 9.3 Background and Foreground

**&** - Run in Background
```bash
command &                      # Run in background
python script.py &             # Background job
```

**jobs** - List Background Jobs
```bash
jobs                           # Show background jobs
jobs -l                        # Show with PIDs
```

**fg** - Bring to Foreground
```bash
fg                             # Bring most recent job to foreground
fg %1                          # Bring job 1 to foreground
```

**bg** - Resume in Background
```bash
bg                             # Resume stopped job in background
bg %1                          # Resume job 1 in background
```

**Ctrl+Z** - Suspend Current Process
```bash
# Press Ctrl+Z to pause current process
# Then use 'bg' to resume in background
```

**nohup** - No Hangup (Keep Running After Logout)
```bash
nohup command &                # Run in background, ignore hangup
nohup python script.py > output.log 2>&1 &
```

---

## 10. Network Commands

**ifconfig** - Network Interfaces (deprecated, use ip)
```bash
ifconfig                       # Show all interfaces
ifconfig eth0                  # Show specific interface
sudo ifconfig eth0 up          # Enable interface
sudo ifconfig eth0 down        # Disable interface
```

**ip** - Modern Network Configuration
```bash
ip addr show                   # Show IP addresses
ip a                           # Short form
ip link show                   # Show interfaces
ip route show                  # Show routing table
sudo ip addr add 192.168.1.100/24 dev eth0  # Add IP address
```

**ping** - Test Connectivity
```bash
ping google.com                # Ping continuously (Ctrl+C to stop)
ping -c 4 google.com           # Send 4 packets
ping -i 2 google.com           # 2 second interval
ping 8.8.8.8                   # Ping IP address
```

**netstat** - Network Statistics (deprecated, use ss)
```bash
netstat -tuln                  # Show listening ports
# -t = TCP
# -u = UDP
# -l = listening
# -n = numeric (no DNS resolution)
# -p = show program (requires root)

netstat -an | grep :80         # Find what's using port 80
```

**ss** - Socket Statistics (modern netstat)
```bash
ss -tuln                       # Listening ports
ss -tunap                      # All connections with programs
ss -s                          # Summary statistics
ss -o state established        # Established connections
```

**nslookup** - DNS Lookup
```bash
nslookup google.com            # Basic lookup
nslookup google.com 8.8.8.8    # Use specific DNS server
```

**dig** - DNS Query Tool
```bash
dig google.com                 # Detailed DNS query
dig google.com MX              # Mail server records
dig google.com NS              # Nameserver records
dig @8.8.8.8 google.com        # Use specific DNS server
```

**host** - Simple DNS Lookup
```bash
host google.com                # Basic lookup
host 8.8.8.8                   # Reverse lookup
```

**traceroute** - Trace Route to Host
```bash
traceroute google.com          # Show path to host
traceroute -n google.com       # No DNS resolution (faster)
```

**curl** - Transfer Data
```bash
curl https://example.com       # Get webpage
curl -I https://example.com    # Headers only
curl -o file.zip URL           # Download file
curl -L URL                    # Follow redirects
curl -u user:pass URL          # Basic authentication
```

**wget** - Download Files
```bash
wget URL                       # Download file
wget -O filename URL           # Save as filename
wget -c URL                    # Resume download
wget -r URL                    # Recursive download
```

---

## 11. Package Management

### 11.1 APT (Debian/Ubuntu)

```bash
sudo apt update                # Update package lists
sudo apt upgrade               # Upgrade installed packages
sudo apt full-upgrade          # Upgrade with dependency changes
sudo apt install nginx         # Install package
sudo apt remove nginx          # Remove package (keep config)
sudo apt purge nginx           # Remove package and config
sudo apt autoremove            # Remove unused dependencies
sudo apt search apache         # Search for packages
apt show nginx                 # Show package details
apt list --installed           # List installed packages
apt list --upgradable          # List upgradable packages
```

### 11.2 DNF/YUM (Red Hat/CentOS/Fedora)

```bash
sudo dnf update                # Update all packages
sudo dnf install nginx         # Install package
sudo dnf remove nginx          # Remove package
sudo dnf search apache         # Search for packages
sudo dnf info nginx            # Package details
sudo dnf list installed        # List installed packages
sudo dnf autoremove            # Remove unused dependencies
```

### 11.3 Snap (Universal)

```bash
sudo snap install package      # Install snap package
sudo snap remove package       # Remove snap
snap list                      # List installed snaps
snap find package              # Search for snaps
```

---

## 12. System Information

**uname** - System Information
```bash
uname -a                       # All information
uname -r                       # Kernel version
uname -m                       # Machine architecture (x86_64)
```

**hostname** - System Hostname
```bash
hostname                       # Show hostname
sudo hostname newhostname      # Change hostname (temporary)
```

**uptime** - System Uptime
```bash
uptime                         # Uptime and load average
# Output: 10:30:45 up 5 days, 2:15, 3 users, load average: 0.50, 0.45, 0.40
```

**df** - Disk Free Space
```bash
df -h                          # Human-readable sizes
df -h /                        # Specific filesystem
df -i                          # Inode usage
```

**du** - Disk Usage
```bash
du -h file.txt                 # File size
du -sh /var/log                # Total size of directory
du -h --max-depth=1 /var       # Size of subdirectories
du -ah /var/log | sort -h      # Sort by size
```

**free** - Memory Usage
```bash
free -h                        # Human-readable
free -m                        # In megabytes
```

**lscpu** - CPU Information
```bash
lscpu                          # Detailed CPU info
```

**lsblk** - Block Devices
```bash
lsblk                          # List disks and partitions
```

**lsusb** - USB Devices
```bash
lsusb                          # List USB devices
```

**dmesg** - Kernel Messages
```bash
dmesg | less                   # View kernel messages
dmesg | grep -i error          # Find errors
```

---

## 13. Log Files & Monitoring

### 13.1 Important Log Files

```bash
/var/log/syslog               # System log (Debian/Ubuntu)
/var/log/messages             # System log (Red Hat/CentOS)
/var/log/auth.log             # Authentication (login attempts)
/var/log/secure               # Authentication (Red Hat)
/var/log/apache2/access.log   # Web server access
/var/log/apache2/error.log    # Web server errors
/var/log/mysql/error.log      # Database errors
/var/log/kern.log             # Kernel logs
/var/log/dmesg                # Boot messages
```

### 13.2 Viewing Logs

```bash
# Real-time monitoring
sudo tail -f /var/log/syslog          # Follow system log
sudo tail -f /var/log/auth.log        # Follow authentication log

# Find failed login attempts
sudo grep "Failed password" /var/log/auth.log

# Find specific IP in web logs
grep "192.168.1.100" /var/log/apache2/access.log

# Count errors
grep -c "error" /var/log/apache2/error.log
```

### 13.3 journalctl (systemd logs)

```bash
journalctl                            # All logs
journalctl -f                         # Follow (real-time)
journalctl -u nginx                   # Logs for nginx service
journalctl --since "2024-02-14"       # Since date
journalctl --since "1 hour ago"       # Last hour
journalctl -p err                     # Errors only
journalctl -k                         # Kernel messages
```

---

## 14. Basic Shell Scripting

### 14.1 Shell Script Basics

**Creating a Script**:
```bash
#!/bin/bash
# This is a comment

echo "Hello, World!"
echo "Current user: $(whoami)"
echo "Current directory: $(pwd)"
```

**Making Executable**:
```bash
chmod +x script.sh
./script.sh
```

### 14.2 Variables

```bash
#!/bin/bash

# Variables
NAME="John"
AGE=25

echo "Name: $NAME"
echo "Age: $AGE"

# Command substitution
CURRENT_DATE=$(date)
echo "Today is: $CURRENT_DATE"

# User input
read -p "Enter your name: " USERNAME
echo "Hello, $USERNAME"
```

### 14.3 Conditionals

```bash
#!/bin/bash

if [ "$USER" = "root" ]; then
    echo "You are root"
elif [ "$USER" = "admin" ]; then
    echo "You are admin"
else
    echo "Regular user"
fi

# File tests
if [ -f "/etc/passwd" ]; then
    echo "File exists"
fi

if [ -d "/var/log" ]; then
    echo "Directory exists"
fi
```

### 14.4 Loops

```bash
#!/bin/bash

# For loop
for i in 1 2 3 4 5; do
    echo "Number: $i"
done

# For loop with range
for i in {1..10}; do
    echo "Count: $i"
done

# While loop
COUNT=0
while [ $COUNT -lt 5 ]; do
    echo "Count: $COUNT"
    COUNT=$((COUNT + 1))
done

# Loop through files
for file in /var/log/*.log; do
    echo "Processing: $file"
done
```

---

## 15. Security Best Practices

### 15.1 User Security

✅ **Use sudo instead of root login**:
```bash
# Disable root login
sudo passwd -l root

# Add user to sudo group
sudo usermod -aG sudo username
```

✅ **Strong password policy**:
```bash
# Install password quality checker
sudo apt install libpam-pwquality

# Edit /etc/pam.d/common-password
# Add: minlen=12 ucredit=-1 lcredit=-1 dcredit=-1
```

✅ **SSH key authentication**:
```bash
# Generate SSH key
ssh-keygen -t ed25519 -C "your_email@example.com"

# Copy to server
ssh-copy-id user@server

# Disable password authentication in /etc/ssh/sshd_config:
# PasswordAuthentication no
```

### 15.2 File Security

✅ **Proper permissions**:
```bash
# System files
chmod 644 /etc/passwd
chmod 600 /etc/shadow
chmod 600 ~/.ssh/id_rsa
chmod 644 ~/.ssh/id_rsa.pub
chmod 700 ~/.ssh

# Web files
chmod 755 /var/www/html
chmod 644 /var/www/html/*.html
```

✅ **Find suspicious files**:
```bash
# Find SUID files
find / -perm -4000 2>/dev/null

# Find world-writable files
find / -perm -002 -type f 2>/dev/null

# Find files without owner
find / -nouser -o -nogroup 2>/dev/null
```

### 15.3 System Security

✅ **Firewall (UFW)**:
```bash
sudo ufw enable
sudo ufw allow 22/tcp          # SSH
sudo ufw allow 80/tcp          # HTTP
sudo ufw allow 443/tcp         # HTTPS
sudo ufw status
```

✅ **Automatic updates**:
```bash
sudo apt install unattended-upgrades
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

✅ **Monitor logs**:
```bash
# Check failed logins
sudo lastb | head

# Check authentication log
sudo tail -50 /var/log/auth.log
```

---

## 16. Interview Questions & Answers

**Q: What Linux distribution do you use and why?**
**A**: "I primarily use Kali Linux for penetration testing because it has 600+ pre-installed security tools. For production servers, I prefer Ubuntu LTS for its stability and 5-year support. I also have experience with CentOS/Rocky Linux in enterprise environments."

**Q: How do you check disk space in Linux?**
**A**: 
```bash
df -h           # Overall disk usage (human-readable)
du -sh /var/*   # Size of directories in /var
```

**Q: How do you find a file in Linux?**
**A**: 
```bash
find /path -name "filename"          # By name
find /path -type f -mtime -7         # Modified in last 7 days
locate filename                      # Faster (uses database)
```

**Q: What's the difference between chmod 644 and 755?**
**A**: 
- **644** (rw-r--r--): Owner can read/write, others can only read. Common for files.
- **755** (rwxr-xr-x): Owner can read/write/execute, others can read/execute. Common for directories and scripts.

**Q: How do you check running processes?**
**A**:
```bash
ps aux              # All processes
top                 # Real-time monitor
pgrep apache2       # Find specific process
```

**Q: How do you find which process is using port 80?**
**A**:
```bash
sudo netstat -tulnp | grep :80
# or modern:
sudo ss -tulnp | grep :80
# or:
sudo lsof -i :80
```

**Q: How do you check failed login attempts?**
**A**:
```bash
sudo grep "Failed password" /var/log/auth.log
sudo lastb              # Failed login database
```

**Q: What's the difference between apt and apt-get?**
**A**: `apt` is newer, user-friendly command that combines `apt-get`, `apt-cache`, and other tools. It has progress bars, colored output, and simpler syntax. Use `apt` for command-line, `apt-get` for scripts (stable interface).

**Q: How do you make a script executable?**
**A**:
```bash
chmod +x script.sh
./script.sh
```

**Q: What's the difference between hard link and symbolic link?**
**A**: 
- **Hard link**: Points to same inode (file data). If original deleted, hard link still works.
- **Symbolic link**: Points to filename (like shortcut). If original deleted, symlink breaks.

**Q: How do you view the last 100 lines of a log file in real-time?**
**A**:
```bash
tail -f -n 100 /var/log/syslog
```

**Q: How do you search for a string in multiple files?**
**A**:
```bash
grep -r "password" /etc/
grep -i "error" *.log    # Case-insensitive in .log files
```

**Q: What does the sudo command do?**
**A**: Allows authorized users to run commands as root (superuser) or another user. Provides better security than logging in as root - actions are logged, temporary privilege elevation, and granular control via /etc/sudoers.

**Q: How do you check system uptime and load?**
**A**:
```bash
uptime
# Output: 10:30:45 up 5 days, 2:15, load average: 0.50, 0.45, 0.40
```

**Q: What are the main differences between /etc and /var directories?**
**A**: 
- **/etc**: Configuration files (static)
- **/var**: Variable data (logs, databases, mail) - changes frequently

---

## Quick Reference

**Navigation**:
```bash
pwd, ls -la, cd, tree
```

**File Operations**:
```bash
cp, mv, rm, mkdir, touch, ln
```

**Viewing Files**:
```bash
cat, less, head, tail -f
```

**Search**:
```bash
grep, find, locate
```

**Permissions**:
```bash
chmod 644 file    # rw-r--r--
chmod 755 dir     # rwxr-xr-x
chown user:group file
```

**Processes**:
```bash
ps aux, top, kill, pkill
```

**Network**:
```bash
ip addr, ping, netstat, ss
```

**Package Management**:
```bash
sudo apt update && sudo apt upgrade
sudo apt install package
```

**System Info**:
```bash
uname -a, df -h, free -h, uptime
```

**Good luck with your interview!** 🎯

You now have comprehensive Linux knowledge covering commands, distributions, permissions, processes, networking, and security best practices!
