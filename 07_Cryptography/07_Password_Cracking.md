
# 07 - Password Cracking Methodology

> **"The best password is one you can't remember."** - A security professional

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [Absolute Beginner's Guide](#absolute-beginners-guide)
3. [Understanding Password Storage](#understanding-password-storage)
4. [Hash Identification](#hash-identification)
5. [Attack Types Overview](#attack-types-overview)
6. [Hashcat Deep Dive](#hashcat-deep-dive)
7. [John the Ripper Deep Dive](#john-the-ripper-deep-dive)
8. [Wordlist Strategies](#wordlist-strategies)
9. [Rule-Based Attacks](#rule-based-attacks)
10. [Mask and Brute Force Attacks](#mask-and-brute-force-attacks)
11. [Rainbow Tables](#rainbow-tables)
12. [Online vs Offline Cracking](#online-vs-offline-cracking)
13. [Platform-Specific Hashes](#platform-specific-hashes)
14. [GPU Acceleration](#gpu-acceleration)
15. [Distributed Cracking](#distributed-cracking)
16. [OSCP Relevance](#oscp-relevance)
17. [Hands-On Labs](#hands-on-labs)
18. [Interview Questions](#interview-questions)
19. [Quick Reference](#quick-reference)

---

## Executive Summary

Password cracking is the art and science of recovering plaintext passwords from their hashed representations. In penetration testing, this skill is **essential** for:

- **Post-exploitation**: Escalating privileges using cracked credentials
- **Lateral movement**: Accessing other systems with recovered passwords
- **Credential stuffing**: Testing password reuse across services
- **Security auditing**: Identifying weak password policies

This guide covers everything from basic concepts to advanced GPU-accelerated attacks using industry-standard tools like Hashcat and John the Ripper.

```
+------------------+     +-----------------+     +------------------+
|   Hash Obtained  | --> | Cracking Tools  | --> | Password Found   |
|  (from SAM/NTDS) |     | (hashcat/john)  |     | (plaintext)      |
+------------------+     +-----------------+     +------------------+
         |                       |
         v                       v
  +-------------+         +---------------+
  | Identify    |         | Attack Types: |
  | Hash Type   |         | - Dictionary  |
  |             |         | - Rules       |
  +-------------+         | - Mask/Brute  |
                          | - Hybrid      |
                          +---------------+
```

---

## Absolute Beginner's Guide

### What is Password Cracking?

Think of password cracking like this:

**Analogy: The Locked Diary**
```
Imagine you found a diary with a 4-digit combination lock.
The lock shows: ####

You could:
1. Try every combo from 0000-9999 (brute force)
2. Try common combos like 1234, 0000, 1111 (dictionary)
3. If you know they use their birth year, try 19XX (mask attack)
4. Use a list of common combinations (wordlist)
```

Password cracking works the same way, but with **hash functions**:

```
Password → Hash Function → Hash Value
"password123" → MD5() → "482c811da5d5b4bc6d497ffa98491e38"
```

### Why Can't We Just Reverse the Hash?

Hash functions are **one-way**. You can't mathematically reverse them:

```
Forward (Easy):
"hello" → SHA256 → "2cf24dba5fb0a30e..."

Backward (Impossible):
"2cf24dba5fb0a30e..." → ??? → "hello"
```

**So how do we crack them?**

We **guess** passwords, hash our guesses, and compare:

```
+-----------------+     +-------------+     +------------------+
| Guess: "test"   | --> | Hash it     | --> | Compare to       |
|                 |     | (MD5/SHA/.) |     | target hash      |
+-----------------+     +-------------+     +------------------+
                                                    |
                              +---------------------+
                              |
                    Match? ---+--- Yes --> PASSWORD FOUND!
                              |
                              +--- No  --> Try next guess
```

### The Cracking Process Simplified

```
Step 1: Obtain the hash
        $ cat /etc/shadow
        user:$6$rounds=5000$salt$hashvalue...

Step 2: Identify the hash type
        $ hashid '$6$rounds=5000$...'
        [+] SHA-512 Crypt

Step 3: Choose attack method
        - Dictionary (wordlist)
        - Brute force (try everything)
        - Rules (modify words: password → P@ssw0rd!)

Step 4: Run the cracker
        $ hashcat -m 1800 hash.txt wordlist.txt

Step 5: Retrieve plaintext
        $ hashcat -m 1800 hash.txt --show
        $6$...:password123
```

---

## Understanding Password Storage

### How Systems Store Passwords

No secure system stores passwords in plaintext. Instead, they store **hashes**:

```
+-------------------------------------------------------------------+
|                    PASSWORD STORAGE EVOLUTION                      |
+-------------------------------------------------------------------+
|                                                                   |
|  1970s: Plaintext (terrible!)                                     |
|         password.txt: admin:admin123                              |
|                                                                   |
|  1980s: Simple hashing                                            |
|         shadow: admin:5f4dcc3b5aa765d61d8327deb882cf99            |
|                                                                   |
|  1990s: Salted hashing                                            |
|         shadow: admin:$1$salt$hashedvalue                         |
|                                                                   |
|  2000s+: Adaptive functions (bcrypt, scrypt)                      |
|         db: admin:$2a$12$salt...longhashedvalue                   |
|                                                                   |
|  Modern: Argon2 (memory-hard)                                     |
|         db: admin:$argon2id$v=19$m=65536...                       |
|                                                                   |
+-------------------------------------------------------------------+
```

### Common Hash Storage Formats

#### Linux /etc/shadow Format

```
username:$id$salt$hash:lastchange:min:max:warn:inactive:expire:reserved

Example:
root:$6$xyz123$KJH8d7f...long_hash...:19000:0:99999:7:::

Breakdown:
- $6$     = SHA-512 (algorithm identifier)
- xyz123  = Salt (random string)
- KJH8d7f... = Actual password hash
```

**Algorithm Identifiers:**
| ID | Algorithm | Strength |
|----|-----------|----------|
| $1$ | MD5 | Weak |
| $2a$, $2b$, $2y$ | bcrypt | Strong |
| $5$ | SHA-256 | Medium |
| $6$ | SHA-512 | Medium |
| $y$ | yescrypt | Strong |

#### Windows Hash Storage

```
+-------------------------------------------------------------------+
|                    WINDOWS PASSWORD HASHES                         |
+-------------------------------------------------------------------+

SAM Database (Local):
  Location: C:\Windows\System32\config\SAM
  Contains: LM hash : NT hash
  
  Format: Username:RID:LM_Hash:NT_Hash:::
  Example: Administrator:500:aad3b435...:31d6cfe0d16ae931b73c59d7e0c089c0:::

NTDS.dit (Domain Controller):
  Location: C:\Windows\NTDS\ntds.dit
  Contains: All domain user hashes
  
  Extraction requires: SYSTEM hive + NTDS.dit file

+-------------------------------------------------------------------+
```

**LM vs NT Hashes:**
```
LM Hash (Legacy, Weak):
- Converts password to uppercase
- Splits into two 7-character halves
- Uses DES encryption
- Empty LM: aad3b435b51404eeaad3b435b51404ee

NT Hash (NTLM):
- MD4 hash of UTF-16LE password
- Case-sensitive
- No salt (vulnerability!)
- Empty NT: 31d6cfe0d16ae931b73c59d7e0c089c0
```

### Web Application Password Storage

```python
# BAD: Plaintext
password = "secret123"
store_in_database(password)

# BAD: Simple hash
import hashlib
password_hash = hashlib.md5(password.encode()).hexdigest()

# BAD: Hash with weak salt
salt = "123"
password_hash = hashlib.sha256((salt + password).encode()).hexdigest()

# GOOD: bcrypt with proper cost factor
import bcrypt
password_hash = bcrypt.hashpw(password.encode(), bcrypt.gensalt(rounds=12))

# BEST: Argon2id
from argon2 import PasswordHasher
ph = PasswordHasher()
password_hash = ph.hash(password)
```

---

## Hash Identification

### Why Identification Matters

Before cracking, you MUST know the hash type. Different types require different hashcat modes:

```
Wrong mode = 0% success rate
Right mode = Your wordlist determines success
```

### Manual Identification by Length and Format

```
+-------------------------------------------------------------------+
|                     HASH LENGTH REFERENCE                          |
+-------------------------------------------------------------------+

32 characters:
  - MD5: 5d41402abc4b2a76b9719d911017c592
  - NTLM: 32ed87bdb5fdc5e9cba88547376818d4
  - MD4: 31d6cfe0d16ae931b73c59d7e0c089c0

40 characters:
  - SHA-1: aaf4c61ddcc5e8a2dabede0f3b482cd9aea9434d

64 characters:
  - SHA-256: 2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824
  - SHA3-256: similar length, different algorithm

128 characters:
  - SHA-512: cf83e1357eefb8bdf1542850d66d8007d...

Variable length (with format markers):
  - $1$salt$hash = MD5-crypt
  - $2a$12$salt...hash = bcrypt
  - $5$salt$hash = SHA-256 crypt
  - $6$salt$hash = SHA-512 crypt
  - $argon2id$... = Argon2

+-------------------------------------------------------------------+
```

### Automated Hash Identification Tools

#### hashid (Python)

```bash
# Installation
pip install hashid

# Usage
hashid 'e10adc3949ba59abbe56e057f20f883e'
# Analyzing 'e10adc3949ba59abbe56e057f20f883e'
# [+] MD2
# [+] MD5
# [+] MD4
# [+] Double MD5
# [+] LM
# [+] NTLM
# ...

# Show hashcat modes
hashid -m 'e10adc3949ba59abbe56e057f20f883e'
# [+] MD5 [Hashcat Mode: 0]
# [+] NTLM [Hashcat Mode: 1000]
```

#### hash-identifier (Kali Built-in)

```bash
hash-identifier
# Enter hash: $6$rounds=5000$saltsalt$...
# Possible Hashs:
# [+] SHA-512 Crypt
```

#### haiti (Ruby)

```bash
# Installation
gem install haiti-hash

# Usage
haiti 'e10adc3949ba59abbe56e057f20f883e'
# MD5 [HC: 0] [JtR: raw-md5]
# MD4 [HC: 900] [JtR: raw-md4]
# ...
```

#### Name-That-Hash (Python)

```bash
# Installation
pip install name-that-hash

# Usage
nth --text 'e10adc3949ba59abbe56e057f20f883e'
# Most Likely
# MD5, HC: 0, JtR: raw-md5
```

### Common Hash Examples with Modes

```
+-------------------------------------------------------------------+
|  HASH EXAMPLE                              | HASHCAT | JOHN       |
+-------------------------------------------------------------------+
| 5d41402abc4b2a76b9719d911017c592           | 0       | raw-md5    |
| aaf4c61ddcc5e8a2dabede0f3b482cd9aea9434d   | 100     | raw-sha1   |
| 2cf24dba5fb0a30e26e83b2ac5b9e29e...        | 1400    | raw-sha256 |
| $1$salt$hash                               | 500     | md5crypt   |
| $5$salt$hash                               | 7400    | sha256crypt|
| $6$salt$hash                               | 1800    | sha512crypt|
| $2a$12$salt$hash                           | 3200    | bcrypt     |
| Administrator:500:LM:NT:::                  | 1000    | nt         |
| user:$apr1$salt$hash                       | 1600    | md5apr1    |
| sha512:1000:salt:hash (PBKDF2)             | 10900   | pbkdf2-hmac|
+-------------------------------------------------------------------+
```

---

## Attack Types Overview

### Visual Attack Type Comparison

```
+-------------------------------------------------------------------+
|                      ATTACK TYPE COMPARISON                        |
+-------------------------------------------------------------------+

DICTIONARY ATTACK
+--------+     +-----------+     +--------+
|wordlist| --> | hash each | --> | compare|
| file   |     |   word    |     | hashes |
+--------+     +-----------+     +--------+
Speed: Fast | Coverage: Limited to wordlist

BRUTE FORCE
+--------+     +-----------+     +--------+
| all    | --> | hash each | --> | compare|
| combos |     |  combo    |     | hashes |
+--------+     +-----------+     +--------+
Speed: Slow | Coverage: 100% (eventually)

RULE-BASED ATTACK
+--------+     +-------+     +-----------+     +--------+
|wordlist| --> | apply | --> | hash each | --> | compare|
| file   |     | rules |     | mutation  |     | hashes |
+--------+     +-------+     +-----------+     +--------+
Speed: Medium | Coverage: Wordlist + common mutations

MASK ATTACK (Smart Brute Force)
+--------+     +-----------+     +--------+
| pattern| --> | generate  | --> | compare|
| mask   |     | & hash    |     | hashes |
+--------+     +-----------+     +--------+
Speed: Fast | Coverage: Pattern-limited

HYBRID ATTACK
+--------+     +--------+     +-----------+     +--------+
|wordlist| --> | + mask | --> | hash each | --> | compare|
| file   |     | append |     | hybrid    |     | hashes |
+--------+     +--------+     +-----------+     +--------+
Speed: Medium | Coverage: Wordlist + patterns

+-------------------------------------------------------------------+
```

### Attack Type Decision Matrix

```
+-------------------------------------------------------------------+
|             WHEN TO USE EACH ATTACK TYPE                          |
+-------------------------------------------------------------------+

Dictionary Attack:
  USE WHEN:
    - Starting fresh (always try first)
    - Testing for common/weak passwords
    - Time is limited
  WORDLISTS:
    - rockyou.txt (14 million)
    - SecLists (various)
    - Custom/targeted

Rule-Based Attack:
  USE WHEN:
    - Dictionary fails
    - Policy requires complexity (P@ssw0rd!)
    - Users modify common words
  RULES:
    - best64.rule (quick)
    - rockyou-30000.rule (thorough)
    - dive.rule (deep)

Mask Attack:
  USE WHEN:
    - You know password structure
    - Password policy reveals pattern
    - Brute force portion of password
  EXAMPLES:
    - Company + 4 digits: Company?d?d?d?d
    - Name + year: ?u?l?l?l?l2023

Hybrid Attack:
  USE WHEN:
    - Dictionary + suffix/prefix pattern
    - Users add numbers to words
    - "password123" style passwords

Full Brute Force:
  USE WHEN:
    - All else fails
    - Short passwords (< 8 chars)
    - You have massive GPU power
    - Fast hash algorithm (MD5, NTLM)

+-------------------------------------------------------------------+
```

---

## Hashcat Deep Dive

### What is Hashcat?

Hashcat is the world's fastest password recovery tool, leveraging GPU acceleration to achieve billions of hashes per second.

```
+-------------------------------------------------------------------+
|                        HASHCAT OVERVIEW                            |
+-------------------------------------------------------------------+

Key Features:
  - GPU-accelerated (CUDA/OpenCL)
  - 350+ hash types supported
  - Multiple attack modes
  - Session management
  - Distributed cracking support

Performance Example (RTX 4090):
  - NTLM: ~200 billion/second
  - MD5: ~150 billion/second  
  - bcrypt: ~180 thousand/second (intentionally slow!)

+-------------------------------------------------------------------+
```

### Hashcat Basic Syntax

```bash
hashcat [options] hashfile [dictionary|mask|directory]

Essential Options:
  -m, --hash-type        Hash type (0=MD5, 1000=NTLM, etc.)
  -a, --attack-mode      Attack mode (0=dict, 1=combo, 3=mask, 6=hybrid)
  -o, --outfile          Output file for cracked passwords
  -r, --rules-file       Rules file for mutations
  --show                 Show already cracked hashes
  --username             Hash file contains usernames
  -w, --workload-profile Workload profile (1=low, 2=default, 3=high, 4=nightmare)
```

### Attack Mode Reference

```
+-------------------------------------------------------------------+
|                    HASHCAT ATTACK MODES                            |
+-------------------------------------------------------------------+

Mode 0: Straight (Dictionary)
  hashcat -m 0 -a 0 hashes.txt wordlist.txt
  
  Tries each word in wordlist directly.

Mode 1: Combination
  hashcat -m 0 -a 1 hashes.txt wordlist1.txt wordlist2.txt
  
  Combines words from two lists: pass + word = password

Mode 3: Brute Force / Mask
  hashcat -m 0 -a 3 hashes.txt ?a?a?a?a?a?a
  
  Generates passwords matching the mask pattern.

Mode 6: Hybrid Wordlist + Mask
  hashcat -m 0 -a 6 hashes.txt wordlist.txt ?d?d?d
  
  Appends mask to each word: password + 123 = password123

Mode 7: Hybrid Mask + Wordlist
  hashcat -m 0 -a 7 hashes.txt ?d?d?d wordlist.txt
  
  Prepends mask to each word: 123 + password = 123password

+-------------------------------------------------------------------+
```

### Hashcat Charsets for Masks

```
Built-in Charsets:
  ?l = abcdefghijklmnopqrstuvwxyz
  ?u = ABCDEFGHIJKLMNOPQRSTUVWXYZ
  ?d = 0123456789
  ?h = 0123456789abcdef
  ?H = 0123456789ABCDEF
  ?s = !"#$%&'()*+,-./:;<=>?@[\]^_`{|}~
  ?a = ?l?u?d?s (all printable ASCII)
  ?b = 0x00 - 0xff (full byte range)

Custom Charsets:
  -1 ?l?d           # lowercase + digits
  -2 ?u?l           # upper + lower
  -3 aeiou          # vowels only
  -4 !@#$           # specific specials

Usage:
  hashcat -m 0 -a 3 -1 ?l?d hash.txt ?1?1?1?1?1?1
  # 6-character lowercase letters and digits
```

### Practical Hashcat Examples

```bash
# Example 1: Crack NTLM with dictionary
hashcat -m 1000 ntlm_hashes.txt /usr/share/wordlists/rockyou.txt

# Example 2: Crack NTLM with rules
hashcat -m 1000 -r /usr/share/hashcat/rules/best64.rule ntlm_hashes.txt rockyou.txt

# Example 3: Crack MD5 with mask (8 lowercase chars)
hashcat -m 0 -a 3 md5_hash.txt ?l?l?l?l?l?l?l?l

# Example 4: Crack SHA512crypt from Linux shadow
hashcat -m 1800 shadow_hashes.txt rockyou.txt

# Example 5: Hybrid attack - word + 4 digits
hashcat -m 0 -a 6 hashes.txt wordlist.txt ?d?d?d?d

# Example 6: Crack with username included in hash file
hashcat -m 1000 --username domain_hashes.txt rockyou.txt

# Example 7: Show cracked passwords
hashcat -m 1000 --show hashes.txt

# Example 8: Restore interrupted session
hashcat --restore

# Example 9: Benchmark your system
hashcat -b

# Example 10: Crack bcrypt (slow, be patient)
hashcat -m 3200 bcrypt_hash.txt rockyou.txt -w 3
```

### Session Management

```bash
# Save session with custom name
hashcat -m 1000 hashes.txt rockyou.txt --session=mycrack

# Check session status
hashcat --status --session=mycrack

# Restore specific session
hashcat --restore --session=mycrack

# Session files location
ls ~/.local/share/hashcat/sessions/
# mycrack.restore
# mycrack.log
```

### Potfile Management

```bash
# Hashcat stores cracked passwords in potfile
# Default: ~/.local/share/hashcat/hashcat.potfile

# Disable potfile (re-crack everything)
hashcat -m 1000 hashes.txt wordlist.txt --potfile-disable

# Use custom potfile
hashcat -m 1000 hashes.txt wordlist.txt --potfile-path=./custom.pot

# Show all cracked from potfile
hashcat -m 1000 hashes.txt --show

# Show only usernames with cracked passwords
hashcat -m 1000 hashes.txt --show --username --outfile-format=2
```

### Common Hashcat Modes Reference

```
+-------------------------------------------------------------------+
|                 FREQUENTLY USED HASHCAT MODES                      |
+-------------------------------------------------------------------+

GENERAL PURPOSE:
  0     MD5
  100   SHA1
  1400  SHA2-256
  1700  SHA2-512
  900   MD4
  
WINDOWS:
  1000  NTLM
  3000  LM
  5500  NetNTLMv1
  5600  NetNTLMv2
  1100  Domain Cached Credentials (DCC), MS Cache
  2100  Domain Cached Credentials 2 (DCC2), MS Cache 2
  
LINUX/UNIX:
  500   MD5crypt, MD5 (Unix)
  1800  sha512crypt, SHA512 (Unix)
  7400  sha256crypt, SHA256 (Unix)
  
WEB APPLICATIONS:
  3200  bcrypt $2*$
  1600  Apache $apr1$ MD5, md5apr1
  400   phpass (WordPress, phpBB3)
  2611  vBulletin < v3.8.5
  2711  vBulletin >= v3.8.5
  121   SMF > v1.1

DATABASE:
  12    PostgreSQL
  112   Oracle S
  1731  MSSQL (2012, 2014)
  300   MySQL4.1/MySQL5

NETWORK/PROTOCOLS:
  2500  WPA-EAPOL-PBKDF2
  22000 WPA-PBKDF2-PMKID+EAPOL (new)
  16800 WPA-PMKID-PBKDF2
  7500  Kerberos 5 AS-REQ Pre-Auth etype 23 (AS-REP roasting)
  18200 Kerberos 5 TGS-REP etype 23 (Kerberoasting)
  13100 Kerberos 5 TGS-REP etype 23 (Kerberoasting, legacy)

CRYPTOCURRENCY:
  11300 Bitcoin/Litecoin wallet.dat
  15200 Blockchain, My Wallet
  16600 Electrum Wallet

ARCHIVES:
  13600 WinZip
  17200 PKZIP (Compressed)
  11600 7-Zip
  13000 RAR5

DOCUMENTS:
  9400  MS Office 2007
  9500  MS Office 2010
  9600  MS Office 2013
  10400 PDF 1.1 - 1.3
  10500 PDF 1.4 - 1.6
  10600 PDF 1.7 Level 3
  10700 PDF 1.7 Level 8

+-------------------------------------------------------------------+
```

---

## John the Ripper Deep Dive

### What is John the Ripper?

John the Ripper (JtR) is a versatile password cracking tool that excels at:
- Auto-detecting hash types
- Cracking many formats out of the box
- Strong community rules and wordlists

```
+-------------------------------------------------------------------+
|                    JOHN THE RIPPER OVERVIEW                        |
+-------------------------------------------------------------------+

Versions:
  - John (core): Basic functionality
  - Jumbo John: Extended version with 300+ formats
  
Strengths:
  - Auto-detect hash format
  - Built-in incremental mode
  - Excellent for shadow/passwd files
  - Strong rule engine

Installation:
  apt install john                    # Basic
  apt install john-jumbo              # Recommended (Jumbo)

+-------------------------------------------------------------------+
```

### John Basic Syntax

```bash
john [options] [password files]

Key Options:
  --wordlist=FILE       Wordlist attack
  --rules=RULESET       Apply rules (with --wordlist)
  --incremental=MODE    Brute force mode
  --format=FORMAT       Force hash format
  --show                Display cracked passwords
  --users=USER          Only crack specific user
  --pot=FILE            Custom pot file
```

### John Attack Modes

```bash
# 1. Single Crack Mode (default, uses login info)
john --single shadow.txt

# 2. Wordlist Mode
john --wordlist=/usr/share/wordlists/rockyou.txt shadow.txt

# 3. Wordlist + Rules
john --wordlist=rockyou.txt --rules shadow.txt

# 4. Incremental (Brute Force)
john --incremental shadow.txt
john --incremental=alpha shadow.txt   # Only letters
john --incremental=digits shadow.txt  # Only numbers

# 5. External Mode (custom C functions)
john --external=filter shadow.txt

# 6. Markov Mode (statistical)
john --markov shadow.txt
```

### Format-Specific Extractors

John includes tools to extract hashes from various file types:

```bash
# SSH Private Keys
ssh2john id_rsa > ssh_hash.txt
john ssh_hash.txt --wordlist=rockyou.txt

# ZIP Files
zip2john protected.zip > zip_hash.txt
john zip_hash.txt --wordlist=rockyou.txt

# RAR Files
rar2john archive.rar > rar_hash.txt
john rar_hash.txt

# PDF Files
pdf2john document.pdf > pdf_hash.txt
john pdf_hash.txt

# Office Documents
office2john document.docx > office_hash.txt
john office_hash.txt

# Keepass Database
keepass2john database.kdbx > keepass_hash.txt
john keepass_hash.txt

# PGP/GPG Keys
gpg2john key.asc > gpg_hash.txt
john gpg_hash.txt

# 7z Archives
7z2john archive.7z > 7z_hash.txt
john 7z_hash.txt

# Ethereum Wallets
ethereum2john wallet.json > eth_hash.txt
john eth_hash.txt

# Kerberos Tickets
kirbi2john ticket.kirbi > kirbi_hash.txt
john kirbi_hash.txt
```

### Practical John Examples

```bash
# Crack Linux shadow file
sudo unshadow /etc/passwd /etc/shadow > unshadowed.txt
john unshadowed.txt --wordlist=rockyou.txt

# Crack NTLM hashes (from secretsdump output)
john ntlm_hashes.txt --format=NT --wordlist=rockyou.txt

# Crack with specific format
john --format=Raw-MD5 md5_hashes.txt --wordlist=rockyou.txt

# Show cracked passwords
john --show shadow.txt
john --show --format=NT ntlm_hashes.txt

# Resume interrupted session
john --restore

# List available formats
john --list=formats

# List available rules
john --list=rules

# Crack with verbose output
john --wordlist=rockyou.txt hash.txt --verbosity=5
```

### John Pot File Management

```bash
# Default pot file location
~/.john/john.pot

# View pot file
cat ~/.john/john.pot

# Use custom pot file
john --pot=./custom.pot hash.txt

# Clear pot file (re-crack all)
rm ~/.john/john.pot
```

### John vs Hashcat Comparison

```
+-------------------------------------------------------------------+
|                    JOHN VS HASHCAT COMPARISON                      |
+-------------------------------------------------------------------+

Feature              | John the Ripper    | Hashcat
---------------------|--------------------|-----------------------
Speed (GPU)          | Good               | Excellent (faster)
Hash auto-detect     | Yes                | No (must specify -m)
Format extractors    | Built-in (2john)   | Separate tools needed
Rule flexibility     | Excellent          | Excellent
Learning curve       | Easier             | Steeper
CPU cracking         | Excellent          | Good
GPU cracking         | Good               | Excellent
Windows support      | Good               | Excellent
Session management   | Basic              | Advanced

RECOMMENDATION:
- Quick tests, unknown formats: John
- Maximum speed, known format: Hashcat
- File-based hashes (ZIP, PDF): John (has extractors)
- Active Directory/NTLM: Hashcat
- Linux shadow files: Both work well

+-------------------------------------------------------------------+
```

---

## Wordlist Strategies

### Essential Wordlists

```
+-------------------------------------------------------------------+
|                      ESSENTIAL WORDLISTS                           |
+-------------------------------------------------------------------+

ROCKYOU.TXT (~14 million passwords)
  Location: /usr/share/wordlists/rockyou.txt
  Source: 2009 RockYou breach
  Note: Compressed by default on Kali (gunzip first)
  
SECLISTS (Collection)
  Location: /usr/share/seclists/Passwords/
  Categories:
    - Common-Credentials/
    - Leaked-Databases/
    - Default-Credentials/
    - Keyboard-Patterns/
  
CRACKSTATION
  Human passwords only: 64 million
  Full list: 1.5 billion
  Download: https://crackstation.net/crackstation-wordlist-password-cracking-dictionary.htm

WEAKPASS
  Consolidated wordlists
  Download: https://weakpass.com/wordlist

KAONASHI (Japanese passwords)
  Download: https://github.com/kaonashi-passwords/Kaonashi

+-------------------------------------------------------------------+
```

### Custom Wordlist Generation

#### CeWL - Web Scraping Wordlist

```bash
# Generate wordlist from target website
cewl https://target.com -w wordlist.txt

# Options
cewl https://target.com \
  -w wordlist.txt \
  -d 2 \              # Depth to spider
  -m 5 \              # Minimum word length
  --with-numbers \    # Include words with numbers
  -e \                # Include email addresses
  -a                  # Include meta data author
  
# Combine with rules later
hashcat -m 1000 hashes.txt wordlist.txt -r best64.rule
```

#### Crunch - Pattern-Based Generation

```bash
# Generate all 4-character lowercase passwords
crunch 4 4 -o wordlist.txt

# Generate 8-char passwords: 4 lowercase + 4 digits
crunch 8 8 -t @@@@%%%% -o wordlist.txt

# Character substitution
# @ = lowercase, , = uppercase, % = digit, ^ = special
crunch 8 8 -t @@@@,,%% -o wordlist.txt

# Custom charset
crunch 6 6 abc123!@ -o wordlist.txt

# Generate and pipe directly to hashcat
crunch 8 8 -t Company%% | hashcat -m 1000 hashes.txt
```

#### CUPP - Common User Passwords Profiler

```bash
# Interactive mode - creates targeted wordlist
cupp -i

# Questions:
# - First/Last name
# - Nickname
# - Birthdate
# - Partner's name
# - Pet's name
# - Company name
# - Keywords

# Creates wordlist based on user info
# Example outputs: John1990, John123!, john_2020, etc.
```

#### Mentalist - GUI Wordlist Generator

```bash
# Install
pip install mentalist

# GUI-based wordlist generation
# Supports base words + rules + substitutions
```

### Wordlist Manipulation

```bash
# Remove duplicates
sort wordlist.txt | uniq > clean.txt

# Sort by length
awk '{ print length, $0 }' wordlist.txt | sort -n | cut -d" " -f2-

# Filter by length (8-12 chars)
awk 'length >= 8 && length <= 12' wordlist.txt > filtered.txt

# Combine wordlists
cat list1.txt list2.txt | sort -u > combined.txt

# Add prefix to all words
sed 's/^/Company/' wordlist.txt > prefixed.txt

# Add suffix to all words
sed 's/$/2023/' wordlist.txt > suffixed.txt

# Lowercase everything
tr 'A-Z' 'a-z' < wordlist.txt > lowercase.txt

# Generate variations with hashcat-utils
# combinator: word1 + word2
/usr/lib/hashcat-utils/combinator.bin list1.txt list2.txt

# princeprocessor: advanced word combination
pp64.bin wordlist.txt | hashcat -m 1000 hash.txt
```

### Strategic Wordlist Building

```
+-------------------------------------------------------------------+
|              STRATEGIC WORDLIST BUILDING APPROACH                  |
+-------------------------------------------------------------------+

1. START WITH RECONNAISSANCE
   - Scrape target website with CeWL
   - OSINT on employees (names, birthdates)
   - Company products, services, locations
   - Industry-specific terminology

2. BUILD BASE WORDLIST
   base_words.txt:
     - Company name + variations
     - Product names
     - Employee names
     - Location names
     - Industry terms
     
3. EXPAND WITH PATTERNS
   Apply common patterns:
     - Word + year (2020-2024)
     - Word + season (Summer2023)
     - Word + number (password123)
     - Word + special (Password!)

4. APPLY SUBSTITUTIONS
   - a → @, 4
   - e → 3
   - i → 1, !
   - o → 0
   - s → $, 5

5. COMBINE STRATEGICALLY
   Priority order:
     1. Custom targeted wordlist
     2. rockyou.txt
     3. SecLists common passwords
     4. Leaked database dumps (if ethical/legal)

+-------------------------------------------------------------------+
```

---

## Rule-Based Attacks

### What are Rules?

Rules modify wordlist entries to generate password variations:

```
Word: "password"
Rule: Capitalize first, add "123"
Result: "Password123"

This turns a simple wordlist into millions of candidates.
```

### Hashcat Rule Syntax

```
+-------------------------------------------------------------------+
|                    HASHCAT RULE REFERENCE                          |
+-------------------------------------------------------------------+

CASE OPERATIONS:
  l          Lowercase all: PASSWORD → password
  u          Uppercase all: password → PASSWORD
  c          Capitalize first: password → Password
  C          lowercase first: Password → password
  t          Toggle case all: PaSsWoRd → pAsSwOrD
  TN         Toggle @ position N: passWord → Password (T0)

CHARACTER OPERATIONS:
  ^X         Prepend character X: ^1password → 1password
  $X         Append character X: password$1 → password1
  [          Delete first char: password → assword
  ]          Delete last char: password → passwor
  DN         Delete char @ position N: D3password → pasword
  iNX        Insert X @ position N: i4!password → pass!word

DUPLICATE OPERATIONS:
  d          Duplicate word: password → passwordpassword
  p          Duplicate word 3x: pass → passpasspass
  q          Duplicate all chars: pass → ppaassss
  r          Reverse: password → drowssap
  f          Reflect: password → passworddrowssap

ROTATION:
  {          Rotate left: password → asswordp
  }          Rotate right: password → dpasswor

MEMORY:
  X          Extract memory
  M          Memorize word

SUBSTITUTION:
  sXY        Replace X with Y: sa@password → p@ssword
  @X         Purge X: @apassword → pssword

LENGTH:
  'N         Truncate @ N: '6password → passwo
  <N         Reject if > N chars
  >N         Reject if < N chars

+-------------------------------------------------------------------+
```

### Popular Rule Files

```bash
# Hashcat rules location
/usr/share/hashcat/rules/

# Key rule files (by effectiveness/time):
best64.rule         # 64 rules, fast, high hit rate
rockyou-30000.rule  # 30K rules, thorough
d3ad0ne.rule        # 34K rules, comprehensive
dive.rule           # 100K rules, deep
Incisive-leetspeak.rule  # Leet speak substitutions
toggles1.rule       # Simple case toggles
toggles2.rule       # More case toggles
toggles3.rule       # Even more toggles
combinator.rule     # Word combinations

# John rules location
/etc/john/john.conf  # In [List.Rules:*] sections
/usr/share/john/rules/
```

### Rule Attack Examples

```bash
# Single rule file
hashcat -m 1000 hash.txt wordlist.txt -r best64.rule

# Multiple rule files (rules combine!)
hashcat -m 1000 hash.txt wordlist.txt -r best64.rule -r toggles1.rule

# Custom inline rule
hashcat -m 1000 hash.txt wordlist.txt -j 'c$1$2$3'
# Capitalize + append "123": password → Password123

# Generate candidates to stdout (debugging)
hashcat --stdout wordlist.txt -r best64.rule | head

# Rule stacking warning:
# 2 files with 100 rules each = 10,000 combined rules!
```

### Custom Rule Creation

```bash
# Create custom rule file
cat > my_rules.rule << 'EOF'
:              
c              
c$1           
c$!           
c$1$2$3       
c$2023        
c$2024        
sa@           
sa@$1         
c sa@         
c sa@ se3     
c$!$1         
$!$@$#        
^1            
^1^2^3        
u             
u$1           
l$123         
EOF

# Use custom rules
hashcat -m 1000 hash.txt wordlist.txt -r my_rules.rule
```

### John Rule Syntax

```bash
# John rules in john.conf:
[List.Rules:Custom]
:                       # Original word
l                       # Lowercase
u                       # Uppercase  
c                       # Capitalize
$[0-9]                  # Append digit
$[0-9]$[0-9]           # Append 2 digits
^[0-9]                  # Prepend digit
Az"[0-9][0-9]"         # Append 2-digit number
A0"[!@#$]"             # Prepend special char
sa@                     # Replace a with @
so0                     # Replace o with 0
r                       # Reverse

# Use custom ruleset
john --wordlist=rockyou.txt --rules=Custom hash.txt
```

### Effective Rule Strategies

```
+-------------------------------------------------------------------+
|                   RULE STRATEGY GUIDE                              |
+-------------------------------------------------------------------+

FOR QUICK TESTS (< 5 minutes):
  hashcat -r best64.rule
  
FOR THOROUGH TESTS (< 1 hour):
  hashcat -r rockyou-30000.rule
  
FOR COMPREHENSIVE (overnight):
  hashcat -r dive.rule
  
STACKING STRATEGY:
  1st pass: No rules (raw wordlist)
  2nd pass: -r best64.rule
  3rd pass: -r best64.rule -r toggles1.rule
  4th pass: -r rockyou-30000.rule

COMMON PASSWORD PATTERNS TO TARGET:
  Word + number:      password123
  Word + year:        password2023
  Caps + word + num:  Password1
  Caps + word + spec: Password!
  L33t speak:         p@ssw0rd
  Word + word:        passwordsecret
  Caps + l33t + num:  P@ssw0rd1

CREATE RULES FOR:
  - Company name + year
  - Season + year (Summer2023)
  - City + number
  - Name + birthdate

+-------------------------------------------------------------------+
```

---

## Mask and Brute Force Attacks

### Understanding Masks

Masks define password patterns using character placeholders:

```
+-------------------------------------------------------------------+
|                      MASK ATTACK CONCEPT                           |
+-------------------------------------------------------------------+

Mask: ?u?l?l?l?l?d?d?d?d
Means: Uppercase + 4 lowercase + 4 digits
Examples: Admin1234, Hello2023, Guest9999

Character Sets:
  ?l = lowercase [a-z]              26 chars
  ?u = uppercase [A-Z]              26 chars
  ?d = digits [0-9]                 10 chars
  ?s = special [!@#$%^&*()...]      33 chars
  ?a = all printable                95 chars

Keyspace Calculation:
  ?u?l?l?l?l?d?d?d?d = 26 × 26 × 26 × 26 × 26 × 10 × 10 × 10 × 10
                     = 26^5 × 10^4
                     = 11,881,376 × 10,000
                     = 118,813,760,000 combinations

+-------------------------------------------------------------------+
```

### Mask Attack Timing Estimation

```
+-------------------------------------------------------------------+
|              KEYSPACE VS TIME (NTLM on RTX 4090)                   |
+-------------------------------------------------------------------+

Length | Charset   | Keyspace        | Time @ 100 GH/s
-------|-----------|-----------------|------------------
4      | ?l        | 456,976         | Instant
6      | ?l        | 308 million     | < 1 second
8      | ?l        | 208 billion     | ~35 minutes
10     | ?l        | 141 trillion    | ~16 days
8      | ?a        | 6.6 quadrillion | ~2 years
6      | ?d        | 1 million       | Instant
8      | ?d        | 100 million     | < 1 second
10     | ?d        | 10 billion      | ~2 seconds

TAKEAWAY:
  - Short passwords (< 8): Always crackable
  - 8-10 lowercase: Hours to days
  - 8+ with full charset: Impractical without mask optimization

+-------------------------------------------------------------------+
```

### Mask Attack Examples

```bash
# 8-character lowercase only
hashcat -m 1000 -a 3 hash.txt ?l?l?l?l?l?l?l?l

# 8-character with numbers at end
hashcat -m 1000 -a 3 hash.txt ?l?l?l?l?l?l?d?d

# Capital + 6 lowercase + digit
hashcat -m 1000 -a 3 hash.txt ?u?l?l?l?l?l?l?d

# Capital + lowercase + numbers + special
hashcat -m 1000 -a 3 hash.txt ?u?l?l?l?l?l?d?d?s

# PIN codes (4-8 digits)
hashcat -m 0 -a 3 hash.txt ?d?d?d?d
hashcat -m 0 -a 3 hash.txt ?d?d?d?d?d?d
hashcat -m 0 -a 3 hash.txt ?d?d?d?d?d?d?d?d

# Phone numbers
hashcat -m 0 -a 3 hash.txt ?d?d?d?d?d?d?d?d?d?d

# Incrementing length (try 1-8 chars)
hashcat -m 1000 -a 3 --increment --increment-min=1 --increment-max=8 hash.txt ?a?a?a?a?a?a?a?a
```

### Custom Charset Masks

```bash
# Define custom charset with -1, -2, -3, -4
hashcat -m 1000 -a 3 -1 ?l?d hash.txt ?1?1?1?1?1?1?1?1
# 8 characters of lowercase + digits

# Two custom charsets
hashcat -m 1000 -a 3 -1 ?l?u -2 ?d?s hash.txt ?1?1?1?1?2?2?2?2
# 4 letters + 4 digits/specials

# Specific characters only
hashcat -m 1000 -a 3 -1 aeiou -2 0123 hash.txt ?1?1?1?1?2?2?2?2
# 4 vowels + 4 specific digits

# Company name + year pattern
hashcat -m 1000 -a 3 hash.txt Company?d?d?d?d
# Company0000 through Company9999

# Summer/Winter + year
hashcat -m 1000 -a 3 -1 SW hash.txt ?1ummer2023
hashcat -m 1000 -a 3 -1 SW hash.txt ?1inter2023
```

### Hybrid Attacks

```bash
# Mode 6: Wordlist + Mask (append)
hashcat -m 1000 -a 6 hash.txt wordlist.txt ?d?d?d?d
# password0000 through password9999

# Mode 7: Mask + Wordlist (prepend)
hashcat -m 1000 -a 7 hash.txt ?d?d?d wordlist.txt
# 000password through 999password

# Hybrid with custom charset
hashcat -m 1000 -a 6 -1 ?d?s hash.txt wordlist.txt ?1?1?1
# password followed by 3 chars of digits or specials

# Hybrid with rules (not directly supported, but:)
# First expand wordlist with rules, then hybrid
hashcat --stdout wordlist.txt -r best64.rule > expanded.txt
hashcat -m 1000 -a 6 hash.txt expanded.txt ?d?d?d?d
```

### Intelligent Mask Usage

```
+-------------------------------------------------------------------+
|                 COMMON PASSWORD PATTERNS                           |
+-------------------------------------------------------------------+

Pattern                  | Mask                    | Example
-------------------------|-------------------------|----------------
Word + 4 digits          | ?l?l?l?l?l?l?d?d?d?d   | spring2023
Cap + word + 2 digits    | ?u?l?l?l?l?l?d?d       | Summer23
Cap + word + special     | ?u?l?l?l?l?l?l?s       | Password!
Cap + word + 3 digits    | ?u?l?l?l?l?l?d?d?d     | Company123
Word + year              | ?l?l?l?l?l?l20?d?d     | summer2023
Month + year             | ?u?l?l?l?l?l?l?d?d?d?d | January2023
Name + birth year        | ?u?l?l?l?l19?d?d       | Steve1985
PIN                      | ?d?d?d?d               | 1234
Extended PIN             | ?d?d?d?d?d?d           | 123456
Social Security          | ?d?d?d?d?d?d?d?d?d     | 123456789

STRATEGIC APPROACH:
  1. Start with patterns matching password policy
  2. Try common formats (Capital + word + digits)
  3. Use incrementing mode for unknown length
  4. Fall back to pure brute force as last resort

+-------------------------------------------------------------------+
```

### Mask Files for Automation

```bash
# Create mask file (.hcmask)
cat > patterns.hcmask << 'EOF'
?u?l?l?l?l?l?d?d
?u?l?l?l?l?l?l?d
?u?l?l?l?l?l?l?d?d
?u?l?l?l?l?l?d?d?d
?u?l?l?l?l?l?l?d?d?d
?l?l?l?l?l?l?d?d?d?d
Password?d?d?d
Summer20?d?d
Winter20?d?d
?u?l?l?l?l?l?l?s
EOF

# Use mask file
hashcat -m 1000 -a 3 hash.txt patterns.hcmask

# With custom charset in mask file
# Format: charset1,charset2,charset3,charset4,mask
cat > custom.hcmask << 'EOF'
?l?u,?d?s,Admin?1?1?1?2?2?2
?l?u,?d,User?1?1?1?2?2?2?2
EOF

hashcat -m 1000 -a 3 hash.txt custom.hcmask
```

---

## Rainbow Tables

### What are Rainbow Tables?

Rainbow tables are precomputed lookup tables for reversing hash functions:

```
+-------------------------------------------------------------------+
|                    RAINBOW TABLE CONCEPT                           |
+-------------------------------------------------------------------+

TRADITIONAL LOOKUP:
  Precompute ALL hashes:
    password → 5f4dcc3b5aa765d61d8327deb882cf99
    123456   → e10adc3949ba59abbe56e057f20f883e
    ...millions more...
  
  Storage: ENORMOUS (terabytes for full coverage)

RAINBOW TABLE (Space-Time Tradeoff):
  Uses "reduction functions" to create chains:
  
  password → [hash] → reduce → newword → [hash] → reduce → ...
  
  Store only START and END of each chain.
  To crack: Apply reduction, check if matches any endpoint.
  
  Storage: Much smaller (gigabytes instead of terabytes)
  Speed: Slower than pure lookup, faster than brute force

+-------------------------------------------------------------------+
```

### Rainbow Table Visualization

```
Chain Creation:

Start: "apple"
  |
  v
hash("apple") = "1f3870be274f6c49b3e31a0c6728957f"
  |
  v
reduce("1f387...") = "74fg8"  (reduction function)
  |
  v
hash("74fg8") = "a2c5d..."
  |
  v
reduce("a2c5d...") = "kL9m2"
  |
  v
...continue for N iterations...
  |
  v
End: "xY7pQ"

STORED: apple → xY7pQ (just start and end!)

To Crack Hash "a2c5d...":
  1. Apply reduction: "a2c5d..." → "kL9m2"
  2. Hash "kL9m2" → check against endpoints
  3. If match found, replay chain from start
  4. Password is somewhere in that chain!
```

### Rainbow Table Limitations

```
+-------------------------------------------------------------------+
|              RAINBOW TABLE LIMITATIONS                             |
+-------------------------------------------------------------------+

DEFEATED BY SALTS:
  Without salt: password → hash
  With salt:    password + "xyz123" → different_hash
  
  Each salt requires a SEPARATE rainbow table!
  If salt is random per user, rainbow tables are useless.

DEFEATED BY SLOW HASHES:
  bcrypt, scrypt, Argon2 are too slow to precompute.
  The computational cost makes table generation impractical.

SPACE REQUIREMENTS:
  MD5, alphanumeric, 8 chars: ~460 GB
  NTLM, alphanumeric, 8 chars: ~460 GB
  Full charset tables: Multiple terabytes

WHERE RAINBOW TABLES WORK:
  - Unsalted hashes (legacy systems)
  - LM hashes (old Windows)
  - Fast algorithms (MD5, SHA1, NTLM)
  - Limited keyspace (known patterns)

WHERE THEY DON'T WORK:
  - Modern Linux (/etc/shadow with salt)
  - bcrypt/scrypt/Argon2 hashes
  - Long, random passwords
  - Unknown password patterns

+-------------------------------------------------------------------+
```

### Using Rainbow Tables with RainbowCrack

```bash
# Download tables from:
# https://project-rainbowcrack.com/table.htm
# http://www.freerainbowtables.com/

# Generate your own (time-consuming)
rtgen md5 loweralpha 1 7 0 2400 8000000 all

# Sort tables (required before use)
rtsort *.rt

# Crack with rainbow tables
rcrack /path/to/tables -h 5d41402abc4b2a76b9719d911017c592

# Crack multiple hashes from file
rcrack /path/to/tables -l hashlist.txt

# LM hash cracking (Windows)
rcrack /path/to/lm_tables -l lm_hashes.txt
```

### Online Rainbow Table Services

```
+-------------------------------------------------------------------+
|                ONLINE RAINBOW TABLE SERVICES                       |
+-------------------------------------------------------------------+

CrackStation (Free):
  https://crackstation.net/
  - Huge lookup database
  - Supports many hash types
  
Hashes.com (Free tier):
  https://hashes.com/
  - Community-driven
  - Hash identification + lookup

OnlineHashCrack (Paid):
  https://www.onlinehashcrack.com/
  - Professional service
  - Captures, WPA, documents

HashKiller (Free):
  https://hashkiller.io/
  - Hash lookup
  - Community submissions

ETHICAL CONSIDERATIONS:
  - Don't submit sensitive production hashes
  - These services log submissions
  - Use for CTF/lab only, not real engagements
  - Consider generating your own tables for sensitive work

+-------------------------------------------------------------------+
```

### When to Use Rainbow Tables vs Active Cracking

```
+-------------------------------------------------------------------+
|           RAINBOW TABLES VS ACTIVE CRACKING DECISION               |
+-------------------------------------------------------------------+

USE RAINBOW TABLES WHEN:
  ✓ Hash is unsalted
  ✓ Hash type is fast (MD5, SHA1, NTLM, LM)
  ✓ You have pre-generated tables
  ✓ Multiple hashes to crack (amortize table cost)
  ✓ GPU resources are limited

USE ACTIVE CRACKING WHEN:
  ✓ Hashes are salted
  ✓ You have powerful GPUs
  ✓ Hash type is slow (bcrypt, scrypt)
  ✓ Targeted wordlists are more effective
  ✓ Password pattern is known (mask attack)

HYBRID APPROACH:
  1. Quick online lookup (free services)
  2. Rainbow table check (if applicable)
  3. Dictionary attack with rules
  4. Mask attack based on policy
  5. Brute force (last resort)

+-------------------------------------------------------------------+
```

---

## Online vs Offline Cracking

### Understanding the Difference

```
+-------------------------------------------------------------------+
|               ONLINE VS OFFLINE PASSWORD ATTACKS                   |
+-------------------------------------------------------------------+

ONLINE ATTACKS:
  Target: Live authentication system
  Method: Send login attempts over network
  Speed: SLOW (network latency, lockouts)
  Detection: HIGH (logs, failed attempts)
  Examples: SSH brute force, web login spray
  
OFFLINE ATTACKS:
  Target: Captured password hashes
  Method: Compute hashes locally
  Speed: FAST (billions/second with GPU)
  Detection: NONE (attacker's own hardware)
  Examples: Cracking NTDS.dit, /etc/shadow

+-------------------------------------------------------------------+
                         
ONLINE ATTACK FLOW:
+--------+     +----------+     +--------+
| Hydra  | --> | Network  | --> | Target |
|Medusa  |     | Request  |     | Server |
+--------+     +----------+     +--------+
     ^                               |
     |          Response             |
     +-------------------------------+
     
Speed: ~100-1000 attempts/second
Risk: Account lockout, IP blocking, detection

OFFLINE ATTACK FLOW:
+--------+     +------------+     +----------+
| Hash   | --> | Hashcat/   | --> | Compare  |
| File   |     | John       |     | Locally  |
+--------+     +------------+     +----------+

Speed: 1,000,000,000+ attempts/second (GPU)
Risk: None (all local computation)
```

### Online Attack Tools

#### Hydra - Network Login Cracker

```bash
# SSH brute force
hydra -l admin -P wordlist.txt ssh://192.168.1.100

# HTTP POST form
hydra -l admin -P wordlist.txt 192.168.1.100 http-post-form \
  "/login.php:user=^USER^&pass=^PASS^:Invalid"

# FTP brute force
hydra -L users.txt -P passwords.txt ftp://192.168.1.100

# SMB brute force
hydra -L users.txt -P passwords.txt smb://192.168.1.100

# RDP brute force
hydra -l administrator -P passwords.txt rdp://192.168.1.100

# MySQL brute force
hydra -l root -P passwords.txt mysql://192.168.1.100

# Options
-l username         Single username
-L users.txt        Username list
-p password         Single password
-P passwords.txt    Password list
-t 4                Threads (parallel connections)
-vV                 Verbose output
-f                  Stop on first success
-s port             Custom port
```

#### Medusa - Parallel Network Cracker

```bash
# SSH attack
medusa -h 192.168.1.100 -u admin -P wordlist.txt -M ssh

# Multiple hosts
medusa -H hosts.txt -U users.txt -P passwords.txt -M ssh

# FTP attack
medusa -h 192.168.1.100 -u admin -P wordlist.txt -M ftp

# HTTP Basic Auth
medusa -h 192.168.1.100 -u admin -P wordlist.txt -M http -m DIR:/admin

# Options
-h host             Target host
-H hosts.txt        Host list
-u username         Single username
-U users.txt        Username list
-P passwords.txt    Password list
-M module           Protocol module (ssh, ftp, http, etc.)
-n port             Custom port
-t threads          Parallel connections
-f                  Stop after first success
```

#### CrackMapExec - Network Password Attacks

```bash
# SMB password spray
crackmapexec smb 192.168.1.0/24 -u userlist.txt -p 'Summer2023!'

# SMB brute force
crackmapexec smb 192.168.1.100 -u admin -p passwords.txt

# WinRM attack
crackmapexec winrm 192.168.1.100 -u users.txt -p passwords.txt

# LDAP attack
crackmapexec ldap dc.domain.local -u users.txt -p passwords.txt

# Pass-the-hash
crackmapexec smb 192.168.1.100 -u admin -H <NTLM_HASH>

# Useful options
--continue-on-success   Don't stop on valid creds
--no-bruteforce        User-pass pairs only (1:1)
--local-auth           Use local authentication
```

### Password Spraying

```
+-------------------------------------------------------------------+
|                      PASSWORD SPRAYING                             |
+-------------------------------------------------------------------+

Traditional Brute Force:
  user1 + password1  →  FAIL
  user1 + password2  →  FAIL
  user1 + password3  →  LOCKOUT!  ← Account locked after N attempts

Password Spray:
  user1 + Summer2023!  →  FAIL
  user2 + Summer2023!  →  FAIL
  user3 + Summer2023!  →  SUCCESS!
  ...all users with same password...
  
  Wait 30+ minutes (lockout reset)
  
  user1 + Winter2023!  →  FAIL
  user2 + Winter2023!  →  FAIL
  ...continue...

ADVANTAGE: Avoids lockouts by trying one password across all users

COMMON SPRAY PASSWORDS:
  - Season + Year: Summer2023, Winter2023
  - Company + Numbers: Acme123!, Acme2023
  - Password + Number: Password1!, Welcome1!
  - Month + Year: January2023, February2023

+-------------------------------------------------------------------+
```

```bash
# Password spray with CrackMapExec
crackmapexec smb dc.domain.local -u users.txt -p 'Summer2023!' --continue-on-success

# Password spray with Ruler (Exchange)
ruler --domain domain.local -k brute --userpass userpass.txt

# Password spray with SprayingToolkit
python3 atomizer.py owa 192.168.1.100 users.txt passwords.txt -i 0:30:00
```

### Obtaining Hashes for Offline Cracking

```bash
# Linux - /etc/shadow
sudo cat /etc/shadow
# Requires root access

# Windows - SAM database (local)
# Method 1: Mimikatz
mimikatz# lsadump::sam

# Method 2: secretsdump (from Linux)
secretsdump.py ./Administrator:Password123@192.168.1.100

# Method 3: SAM file copy (offline)
# Boot from USB, copy C:\Windows\System32\config\SAM and SYSTEM

# Windows - NTDS.dit (Domain Controller)
# Method 1: secretsdump
secretsdump.py domain/admin:Password@dc.domain.local -just-dc-ntlm

# Method 2: ntdsutil + manual extraction
ntdsutil "ac i ntds" "ifm" "create full c:\temp" q q

# Method 3: DCSync attack (requires replication rights)
secretsdump.py domain/admin:Password@dc.domain.local -just-dc

# Kerberoasting - Get service account hashes
GetUserSPNs.py -dc-ip 192.168.1.10 domain/user:password -outputfile kerberoast.txt

# AS-REP Roasting - Get hashes for accounts with pre-auth disabled
GetNPUsers.py domain/ -usersfile users.txt -dc-ip 192.168.1.10 -outputfile asrep.txt
```

---

## Platform-Specific Hashes

### Windows Hashes

```
+-------------------------------------------------------------------+
|                    WINDOWS HASH TYPES                              |
+-------------------------------------------------------------------+

LM HASH (LAN Manager) - Legacy:
  - Format: aad3b435b51404eeaad3b435b51404ee
  - Algorithm: DES-based
  - Weakness: Case-insensitive, splits password into 7-char halves
  - Disabled by default since Vista
  - Hashcat mode: 3000

NTLM HASH (NT LAN Manager):
  - Format: 31d6cfe0d16ae931b73c59d7e0c089c0
  - Algorithm: MD4(UTF-16LE(password))
  - Weakness: No salt, fast to crack
  - Used for local and domain auth
  - Hashcat mode: 1000

NetNTLMv1:
  - Challenge-response authentication
  - Can be cracked or relayed
  - Hashcat mode: 5500

NetNTLMv2:
  - Improved version, includes timestamp
  - Still crackable with good wordlist
  - Hashcat mode: 5600

DCC (Domain Cached Credentials):
  - Cached domain logon hashes
  - Location: HKLM\SECURITY\Cache
  - Hashcat mode: 1100 (DCC1), 2100 (DCC2/mscash2)

Kerberos:
  - AS-REP (Pre-auth disabled): mode 18200
  - TGS-REP (Kerberoasting): mode 13100

+-------------------------------------------------------------------+
```

### Windows Hash Cracking Examples

```bash
# NTLM hash cracking
hashcat -m 1000 ntlm_hashes.txt rockyou.txt

# NTLM with rules
hashcat -m 1000 -r best64.rule ntlm_hashes.txt rockyou.txt

# LM hash (if present)
hashcat -m 3000 lm_hashes.txt rockyou.txt

# NetNTLMv2 (captured with Responder)
hashcat -m 5600 netntlmv2.txt rockyou.txt

# Domain Cached Credentials v2
hashcat -m 2100 mscash2.txt rockyou.txt

# Kerberoast (TGS tickets)
hashcat -m 13100 kerberoast.txt rockyou.txt

# AS-REP roast
hashcat -m 18200 asrep.txt rockyou.txt

# With John
john --format=NT ntlm_hashes.txt --wordlist=rockyou.txt
john --format=netntlmv2 netntlmv2.txt --wordlist=rockyou.txt
john --format=krb5tgs kerberoast.txt --wordlist=rockyou.txt
```

### Linux Hashes

```
+-------------------------------------------------------------------+
|                     LINUX HASH TYPES                               |
+-------------------------------------------------------------------+

/etc/shadow format:
  username:$id$salt$hash:lastchg:min:max:warn:inactive:expire

ALGORITHM IDENTIFIERS:
  $1$  = MD5crypt         (legacy, weak)    Hashcat: 500
  $2a$ = bcrypt           (strong)          Hashcat: 3200
  $2b$ = bcrypt           (strong)          Hashcat: 3200
  $5$  = SHA-256 crypt    (medium)          Hashcat: 7400
  $6$  = SHA-512 crypt    (common default)  Hashcat: 1800
  $y$  = yescrypt         (modern, strong)  Hashcat: N/A (slow)

EXAMPLE:
  root:$6$rounds=5000$saltsalt$hashvalue...:18500:0:99999:7:::
        |   |           |      |
        |   |           |      +-- Actual hash
        |   |           +-- Salt
        |   +-- Optional rounds parameter
        +-- Algorithm identifier

+-------------------------------------------------------------------+
```

### Linux Hash Cracking Examples

```bash
# Combine passwd and shadow
unshadow /etc/passwd /etc/shadow > combined.txt

# SHA-512 crypt ($6$)
hashcat -m 1800 combined.txt rockyou.txt

# MD5 crypt ($1$)
hashcat -m 500 combined.txt rockyou.txt

# SHA-256 crypt ($5$)
hashcat -m 7400 combined.txt rockyou.txt

# bcrypt ($2a$, $2b$)
hashcat -m 3200 combined.txt rockyou.txt
# Note: bcrypt is SLOW (~180 kH/s on RTX 4090)

# With John
john --wordlist=rockyou.txt combined.txt
# John auto-detects format
```

### Web Application Hashes

```bash
# WordPress (phpass)
# Format: $P$B...
hashcat -m 400 wordpress_hashes.txt rockyou.txt

# phpBB3 (phpass)
hashcat -m 400 phpbb_hashes.txt rockyou.txt

# Drupal 7
# Format: $S$...
hashcat -m 7900 drupal_hashes.txt rockyou.txt

# Joomla
# Format: hash:salt or $2y$ for newer
hashcat -m 11 joomla_old.txt rockyou.txt
hashcat -m 3200 joomla_bcrypt.txt rockyou.txt

# Django (PBKDF2-SHA256)
# Format: pbkdf2_sha256$iterations$salt$hash
hashcat -m 10000 django_hashes.txt rockyou.txt

# vBulletin
# Format: hash:salt
hashcat -m 2711 vbulletin_hashes.txt rockyou.txt

# MediaWiki
# Format: :B:salt:hash
hashcat -m 3711 mediawiki_hashes.txt rockyou.txt
```

---

## GPU Acceleration

### Why GPUs Dominate Password Cracking

```
+-------------------------------------------------------------------+
|                    GPU VS CPU CRACKING                             |
+-------------------------------------------------------------------+

CPU Architecture:
  - Few powerful cores (4-16 typical)
  - Optimized for complex sequential tasks
  - Good at branching, conditionals

GPU Architecture:
  - Thousands of simple cores (5000+ on high-end)
  - Optimized for parallel identical operations
  - Perfect for: hash(candidate), compare, repeat

NTLM Benchmark Comparison:
  CPU (Intel i9-13900K):    ~500 million/second
  GPU (RTX 4090):           ~200 billion/second
  
  GPU is 400x FASTER for this hash type!

+-------------------------------------------------------------------+
```

### Hashcat OpenCL/CUDA Setup

```bash
# Check GPU detection
hashcat -I

# Sample output:
# OpenCL API (OpenCL 3.0 CUDA 12.2.138) - Platform #1 [NVIDIA Corporation]
# * Device #1: NVIDIA GeForce RTX 4090, 24320/24564 MB, 128MCU

# If no GPU detected:
# 1. Install NVIDIA drivers
# 2. Install CUDA toolkit
# 3. Reboot

# Force GPU device
hashcat -m 1000 hashes.txt wordlist.txt -d 1

# Use multiple GPUs
hashcat -m 1000 hashes.txt wordlist.txt -d 1,2
```

### Workload Optimization

```bash
# Workload profiles (-w)
# 1 = Low (don't impact desktop usage)
# 2 = Default
# 3 = High (may cause lag)
# 4 = Nightmare (system unusable during crack)

# For dedicated cracking rig
hashcat -m 1000 -w 4 hashes.txt wordlist.txt

# For shared workstation
hashcat -m 1000 -w 1 hashes.txt wordlist.txt

# Temperature limits
hashcat -m 1000 hashes.txt wordlist.txt --hwmon-temp-abort=90
# Abort if GPU reaches 90°C

# Kernel optimization
hashcat -m 1000 hashes.txt wordlist.txt -O
# Optimized kernels, may limit password length
```

### GPU Performance by Hash Type

```
+-------------------------------------------------------------------+
|            RTX 4090 PERFORMANCE BY HASH TYPE                       |
+-------------------------------------------------------------------+

Hash Type              | Speed           | Time for 8-char ?l
-----------------------|-----------------|--------------------
NTLM                   | 200 GH/s        | ~1 second
MD5                    | 150 GH/s        | ~1.4 seconds
SHA1                   | 40 GH/s         | ~5 seconds
SHA256                 | 12 GH/s         | ~17 seconds
SHA512                 | 4 GH/s          | ~52 seconds
bcrypt ($2a$, cost 5)  | 180 kH/s        | ~13 days
bcrypt (cost 12)       | 2 kH/s          | ~3 years
scrypt                 | 1 MH/s          | ~2 days
Argon2                 | 100 H/s         | Impractical

TAKEAWAY:
  - Fast hashes (MD5, NTLM, SHA1): GPUs dominate
  - Slow hashes (bcrypt, scrypt, Argon2): Design working as intended!

+-------------------------------------------------------------------+
```

### Cloud GPU Options

```
+-------------------------------------------------------------------+
|                   CLOUD GPU CRACKING                               |
+-------------------------------------------------------------------+

AWS EC2:
  - p4d.24xlarge: 8x A100 GPUs (~$32/hour)
  - p3.16xlarge: 8x V100 GPUs (~$24/hour)
  - Good for burst cracking needs

Google Cloud:
  - a2-highgpu-8g: 8x A100 GPUs (~$30/hour)
  - Preemptible instances for cost savings

Vast.ai (GPU Rental):
  - Rent from individuals
  - Often cheaper than cloud providers
  - RTX 4090: ~$0.50-1.00/hour

Lambda Labs:
  - ML-focused, good GPU availability
  - 8x A100: ~$10/hour (cheaper)

Penguin Computing:
  - HPC-focused
  - Good for large-scale cracking

LEGAL CONSIDERATIONS:
  - Check cloud provider TOS
  - Some prohibit "cracking" activities
  - Use for legitimate security testing only

+-------------------------------------------------------------------+
```

---

## Distributed Cracking

### Hashtopolis - Web-Based Distributed Cracking

```
+-------------------------------------------------------------------+
|                      HASHTOPOLIS                                   |
+-------------------------------------------------------------------+

Architecture:
+-------------+        +-------------+        +-------------+
|   Agent 1   |        |   Server    |        |   Agent 2   |
| (GPU Node)  | <----> | (Web UI)    | <----> | (GPU Node)  |
+-------------+        +-------------+        +-------------+
                             ^
                             |
                       +-------------+
                       |   Agent N   |
                       | (GPU Node)  |
                       +-------------+

Features:
  - Web-based management
  - Automatic work distribution
  - Progress tracking
  - Multiple agent support
  - Hashcat backend

Installation:
  # Server (Ubuntu)
  apt install apache2 php mysql-server
  git clone https://github.com/hashtopolis/server.git
  # Configure via web installer
  
  # Agent (Any OS with Python)
  python3 hashtopolis.zip --url https://server/api/agent.php

+-------------------------------------------------------------------+
```

### Manual Distribution with Hashcat

```bash
# Split wordlist manually
split -n l/4 rockyou.txt part_

# Each node runs a portion
# Node 1: hashcat -m 1000 hashes.txt part_aa
# Node 2: hashcat -m 1000 hashes.txt part_ab
# Node 3: hashcat -m 1000 hashes.txt part_ac
# Node 4: hashcat -m 1000 hashes.txt part_ad

# Or use --skip and --limit
# Total keyspace: 1000000000
# Node 1:
hashcat -m 0 -a 3 hash.txt ?l?l?l?l?l?l --skip 0 --limit 250000000
# Node 2:
hashcat -m 0 -a 3 hash.txt ?l?l?l?l?l?l --skip 250000000 --limit 250000000
# Node 3:
hashcat -m 0 -a 3 hash.txt ?l?l?l?l?l?l --skip 500000000 --limit 250000000
# Node 4:
hashcat -m 0 -a 3 hash.txt ?l?l?l?l?l?l --skip 750000000 --limit 250000000
```

---

## OSCP Relevance

### Common OSCP Password Scenarios

```
+-------------------------------------------------------------------+
|               OSCP PASSWORD CRACKING SCENARIOS                     |
+-------------------------------------------------------------------+

SCENARIO 1: Linux Privilege Escalation
  - Find readable /etc/shadow or backup
  - Extract hashes
  - Crack with hashcat -m 1800 (SHA512crypt)
  - Use cracked password for sudo/root access

SCENARIO 2: Windows Credential Theft
  - Dump SAM with mimikatz or secretsdump
  - Extract NTLM hashes
  - Crack with hashcat -m 1000
  - Use for Pass-the-Hash or RDP access

SCENARIO 3: Web Application
  - SQL injection reveals password hashes
  - Identify hash type (MD5, bcrypt, etc.)
  - Crack to access admin panels
  - Pivot from web shell to system access

SCENARIO 4: Network Services
  - Capture NetNTLMv2 with Responder
  - Crack with hashcat -m 5600
  - Use credentials for lateral movement

SCENARIO 5: Kerberoasting
  - Request TGS tickets for service accounts
  - Crack offline with hashcat -m 13100
  - Service accounts often have weak passwords

+-------------------------------------------------------------------+
```

### OSCP Password Attack Methodology

```
+-------------------------------------------------------------------+
|              OSCP PASSWORD ATTACK FLOW                             |
+-------------------------------------------------------------------+

1. ENUMERATION
   +-- Identify authentication services
   +-- Check for default credentials
   +-- Look for credential files
   
2. ONLINE ATTACKS (Limited attempts)
   +-- Try top 10-20 passwords manually
   +-- Credential stuffing (if you have creds)
   +-- Password spray (one password, many users)
   
3. HASH EXTRACTION (Post-exploitation)
   +-- Linux: /etc/shadow, config files
   +-- Windows: SAM, NTDS.dit, lsass
   +-- Web: Database dumps
   
4. OFFLINE CRACKING
   +-- Identify hash type
   +-- Dictionary attack (rockyou)
   +-- Rules attack (best64.rule)
   +-- Targeted wordlist (CeWL, CUPP)
   
5. CREDENTIAL REUSE
   +-- Try cracked passwords on other services
   +-- SSH, RDP, SMB, web apps
   +-- Admin panels, databases

+-------------------------------------------------------------------+
```

### Quick OSCP Cracking Commands

```bash
# === LINUX SHADOW FILE ===
# Copy shadow file
sudo cat /etc/shadow > shadow.txt

# Create combined file
unshadow /etc/passwd /etc/shadow > unshadowed.txt

# Crack with john (auto-detect)
john unshadowed.txt --wordlist=/usr/share/wordlists/rockyou.txt

# Crack with hashcat (SHA512crypt)
hashcat -m 1800 shadow_hashes.txt /usr/share/wordlists/rockyou.txt

# === WINDOWS NTLM ===
# After secretsdump.py or mimikatz
hashcat -m 1000 ntlm_hashes.txt rockyou.txt
hashcat -m 1000 -r best64.rule ntlm_hashes.txt rockyou.txt

# === KERBEROASTING ===
# Get tickets
GetUserSPNs.py -dc-ip 10.10.10.10 domain/user:pass -outputfile tgs.txt

# Crack
hashcat -m 13100 tgs.txt rockyou.txt

# === WEB APP HASHES ===
# MD5 (common in old apps)
hashcat -m 0 md5_hashes.txt rockyou.txt

# WordPress/phpBB
hashcat -m 400 wordpress_hashes.txt rockyou.txt

# === SSH KEYS ===
ssh2john id_rsa > ssh_hash.txt
john ssh_hash.txt --wordlist=rockyou.txt

# === ZIP/RAR FILES ===
zip2john protected.zip > zip_hash.txt
john zip_hash.txt --wordlist=rockyou.txt
```

---

## Hands-On Labs

### Lab 1: Linux Shadow Cracking

```markdown
OBJECTIVE: Crack passwords from a Linux shadow file

SETUP:
1. Create test users on a Linux VM:
   sudo useradd -m testuser1
   echo "testuser1:password123" | sudo chpasswd
   
   sudo useradd -m testuser2
   echo "testuser2:Summer2023!" | sudo chpasswd
   
   sudo useradd -m testuser3
   echo "testuser3:Tr0ub4dor&3" | sudo chpasswd

2. Extract hashes:
   sudo cat /etc/shadow | grep testuser > hashes.txt

TASKS:
1. Identify the hash type using hashid
2. Crack with rockyou.txt
3. Crack remaining with rules
4. Document which passwords were cracked and why

SOLUTION:
# Identify
hashid '$(cat hashes.txt | head -1 | cut -d: -f2)'
# SHA512crypt

# Crack
hashcat -m 1800 hashes.txt /usr/share/wordlists/rockyou.txt
# Should crack testuser1 quickly

# With rules for harder ones
hashcat -m 1800 -r best64.rule hashes.txt rockyou.txt
```

### Lab 2: Windows NTLM Cracking

```markdown
OBJECTIVE: Crack NTLM hashes extracted from Windows

SETUP:
Create file ntlm_hashes.txt with:
```
user1:1001:aad3b435b51404eeaad3b435b51404ee:8846f7eaee8fb117ad06bdd830b7586c:::
user2:1002:aad3b435b51404eeaad3b435b51404ee:e52cac67419a9a224a3b108f3fa6cb6d:::
user3:1003:aad3b435b51404eeaad3b435b51404ee:7ce21f17c0aee7fb9ceba532d0546ad6:::
```

TASKS:
1. Extract just the NT hashes
2. Identify that LM hashes are empty (aad3b...)
3. Crack using dictionary attack
4. Use mask attack for remaining

COMMANDS:
# Extract NT hash portion
cat ntlm_hashes.txt | cut -d: -f4 > nt_only.txt

# Crack
hashcat -m 1000 nt_only.txt rockyou.txt

# Expected results:
# 8846f7eaee8fb117ad06bdd830b7586c:password
# e52cac67419a9a224a3b108f3fa6cb6d:123456
# (third may need rules or mask)
```

### Lab 3: Kerberoasting Simulation

```markdown
OBJECTIVE: Practice cracking Kerberos TGS tickets

SETUP:
Create file kerberoast.txt with sample hash:
$krb5tgs$23$*sqlsvc$DOMAIN.LOCAL$MSSQLSvc/sql.domain.local:1433*$hash...

(Use a real hash from a lab like HackTheBox or generate one)

TASKS:
1. Identify the hash as Kerberos 5 TGS-REP
2. Determine correct hashcat mode (13100)
3. Crack using targeted wordlist

APPROACH:
# Kerberoast hashes are usually service accounts
# Service accounts often have weak passwords

# First try common service account passwords
echo -e "Password1\nPassword123\nSummer2023\nWinter2023\nAdmin123" > service_passwords.txt
hashcat -m 13100 kerberoast.txt service_passwords.txt

# Then rockyou with rules
hashcat -m 13100 -r best64.rule kerberoast.txt rockyou.txt
```

### Lab 4: Hash Identification Challenge

```markdown
OBJECTIVE: Identify various hash types

HASHES TO IDENTIFY:
1. 5d41402abc4b2a76b9719d911017c592
2. aaf4c61ddcc5e8a2dabede0f3b482cd9aea9434d
3. $2a$12$LQv3c1yqBWVHxkd0LHAkCOYz6TtxMQJqhN8/LewKyDAXKJN.WeQIq
4. $6$rounds=5000$saltsalt$IxDD.oi...
5. 31d6cfe0d16ae931b73c59d7e0c089c0
6. $apr1$71850310$gh9m4xcAn3MGxogwX/ztb.

TASKS:
1. Use hashid on each hash
2. Determine hashcat mode for each
3. Match to: MD5, SHA1, bcrypt, SHA512crypt, NTLM, Apache MD5

ANSWERS:
1. MD5 (mode 0) - 32 chars, hex
2. SHA1 (mode 100) - 40 chars, hex
3. bcrypt (mode 3200) - $2a$ prefix
4. SHA512crypt (mode 1800) - $6$ prefix
5. NTLM (mode 1000) - 32 chars, looks like MD5 but from Windows
6. Apache MD5 (mode 1600) - $apr1$ prefix
```

### Lab 5: Rule Creation Exercise

```markdown
OBJECTIVE: Create custom rules for company password policy

SCENARIO:
Company policy requires:
- Minimum 8 characters
- At least one uppercase
- At least one number
- At least one special character

Users likely follow pattern: Word + Number + Special

TASKS:
1. Create a rule file targeting this pattern
2. Test against sample hashes
3. Measure effectiveness

SOLUTION:
# Create rules file
cat > company_rules.rule << 'EOF'
# Capitalize first, add digits and specials
c$1
c$1$!
c$12$!
c$123
c$123$!
c$2023
c$2023$!
c$2024
c$2024$!
# Add common endings
c$!
c$@
c$#
c$1$2$3
# Leetspeak
c sa@ so0 se3
c sa@ so0 $1
# Common patterns
c $! $@ $#
l $2023
l $2024
u $1
EOF

# Test
hashcat -m 1000 test_hashes.txt wordlist.txt -r company_rules.rule
```

---

## Interview Questions

### Basic Level

1. **Q: What's the difference between online and offline password attacks?**
   
   A: Online attacks send authentication attempts to a live system (slow, detectable, may trigger lockouts). Offline attacks work against captured hashes locally (fast, undetectable, no lockouts). Offline is vastly more powerful when hashes are available.

2. **Q: Why can't you reverse a hash to get the original password?**
   
   A: Hash functions are one-way mathematical operations designed to be computationally infeasible to reverse. Instead, we guess passwords, hash them, and compare to the target hash.

3. **Q: What is a salt, and why does it matter for cracking?**
   
   A: A salt is random data added to a password before hashing. It prevents rainbow table attacks and ensures identical passwords produce different hashes. Each unique salt effectively requires a separate cracking effort.

### Intermediate Level

4. **Q: Explain the difference between dictionary, rule-based, and mask attacks.**
   
   A: Dictionary tries words from a list directly. Rule-based modifies dictionary words (password→P@ssw0rd!). Mask attacks generate candidates matching a pattern (?u?l?l?l?d?d). Each has different speed/coverage tradeoffs.

5. **Q: What makes bcrypt harder to crack than MD5?**
   
   A: bcrypt has a configurable "cost factor" that controls iterations. A cost of 12 means 2^12 (4096) iterations. This intentionally slows hashing to ~100-200 kH/s vs MD5's billions/second. bcrypt also includes a built-in salt.

6. **Q: How would you approach cracking an unknown hash type?**
   
   A: First identify using hashid or haiti (length, format, prefixes). Then verify with hashcat's example hashes. Start with dictionary attack, add rules, then try targeted masks based on likely password patterns.

### Advanced Level

7. **Q: Describe a Kerberoasting attack and how to crack the resulting hashes.**
   
   A: Request TGS tickets for service accounts (any domain user can do this). Tickets are encrypted with service account's password hash. Extract tickets with GetUserSPNs.py, crack offline with hashcat -m 13100. Service accounts often have weak passwords.

8. **Q: How does GPU password cracking achieve such high speeds?**
   
   A: GPUs have thousands of simple cores optimized for parallel identical operations. Hashing is embarrassingly parallel - each candidate can be hashed independently. Modern GPUs can achieve 200+ billion hashes/second for NTLM.

9. **Q: What strategies would you use for a targeted attack against a specific organization?**
   
   A: Build custom wordlists using CeWL (website scraping), OSINT (employee names, company info), and CUPP (personal info). Create targeted masks based on password policy. Use company name, locations, products as base words with rules.

---

## Quick Reference

### Hash Mode Cheat Sheet

```
+-------------------------------------------------------------------+
|                    HASHCAT MODE QUICK REFERENCE                    |
+-------------------------------------------------------------------+

FAST HASHES (GPU-friendly):
  0      MD5
  100    SHA1  
  1000   NTLM
  1400   SHA256
  1700   SHA512
  3000   LM

LINUX:
  500    MD5crypt ($1$)
  1800   SHA512crypt ($6$)
  7400   SHA256crypt ($5$)

WINDOWS:
  1000   NTLM
  1100   DCC1
  2100   DCC2 (mscash2)
  5500   NetNTLMv1
  5600   NetNTLMv2

KERBEROS:
  13100  Kerberos 5 TGS (Kerberoasting)
  18200  Kerberos 5 AS-REP (AS-REP roasting)

SLOW HASHES:
  3200   bcrypt
  10000  Django (PBKDF2)
  10900  PBKDF2-HMAC-SHA256

WEB:
  400    phpass (WordPress/phpBB)
  1600   Apache $apr1$
  7900   Drupal7

FILES:
  13600  WinZip
  11600  7-Zip
  9600   MS Office 2013
  10700  PDF 1.7 Level 8

+-------------------------------------------------------------------+
```

### Command Cheat Sheet

```bash
# === HASH IDENTIFICATION ===
hashid 'HASHVALUE'
hashid -m 'HASHVALUE'                    # Show hashcat modes
nth --text 'HASHVALUE'                   # Name-that-hash

# === HASHCAT BASIC ===
hashcat -m MODE hash.txt wordlist.txt    # Dictionary
hashcat -m MODE -r rules.rule hash.txt wordlist.txt  # With rules
hashcat -m MODE -a 3 hash.txt ?a?a?a?a?a # Mask attack
hashcat -m MODE -a 6 hash.txt words.txt ?d?d?d  # Hybrid

# === HASHCAT SESSION ===
hashcat --session=NAME ...               # Named session
hashcat --restore                        # Resume last
hashcat --restore --session=NAME         # Resume specific
hashcat -m MODE hash.txt --show          # Show cracked

# === JOHN BASIC ===
john hash.txt --wordlist=rockyou.txt     # Dictionary
john hash.txt --rules --wordlist=rockyou.txt  # With rules
john hash.txt --incremental               # Brute force
john --show hash.txt                       # Show cracked
john --format=FORMAT hash.txt             # Force format

# === JOHN EXTRACTORS ===
ssh2john id_rsa > hash.txt
zip2john file.zip > hash.txt
rar2john file.rar > hash.txt
pdf2john file.pdf > hash.txt
office2john file.docx > hash.txt

# === WORDLISTS ===
cewl https://target.com -w custom.txt    # Scrape website
crunch 8 8 -t @@@@%%%% -o wordlist.txt   # Generate patterns
cupp -i                                   # Interactive profiler

# === ONLINE ATTACKS ===
hydra -l user -P pass.txt ssh://IP
hydra -l user -P pass.txt IP http-post-form "/login:user=^USER^&pass=^PASS^:F=fail"
crackmapexec smb IP -u users.txt -p 'Password1'
```

### Attack Strategy Decision Tree

```
+-------------------------------------------------------------------+
|                  PASSWORD CRACKING DECISION TREE                   |
+-------------------------------------------------------------------+

START
  |
  v
Hash obtained? --No--> Online attack (Hydra/CrackMapExec)
  |                           |
  Yes                        Consider spray vs brute
  |
  v
Identify hash type (hashid)
  |
  v
Is hash salted?
  |
  +--Yes--> Active cracking only (no rainbow tables)
  |
  +--No---> Try rainbow tables first, then active
  |
  v
Fast hash (MD5, NTLM)? --Yes--> GPU attack feasible
  |                              Larger keyspace possible
  No
  |
  v
Slow hash (bcrypt, scrypt)?
  |
  Yes --> Limited attempts, prioritize:
          1. Common passwords
          2. Targeted wordlist
          3. Smart rules only
  |
  v
ATTACK ORDER:
  1. rockyou.txt (no rules)
  2. rockyou.txt + best64.rule
  3. Custom wordlist (CeWL, company-specific)
  4. Mask attack based on policy
  5. rockyou.txt + dive.rule (thorough)
  6. Full brute force (last resort)

+-------------------------------------------------------------------+
```

---

## Related Files

- [[03_Hashing_and_Integrity]] - Understanding hash functions
- [[04_Symmetric_Encryption]] - Understanding encryption vs hashing
- [[08_Cryptographic_Attacks]] - Advanced cryptographic vulnerabilities
- [[10_SSH_and_Key_Management]] - SSH key cracking scenarios
- [[11_Obfuscation]] - Deobfuscation for password extraction

---

## Resources

### Official Documentation
- [Hashcat Wiki](https://hashcat.net/wiki/)
- [John the Ripper Documentation](https://www.openwall.com/john/doc/)
- [Hashcat Example Hashes](https://hashcat.net/wiki/doku.php?id=example_hashes)

### Wordlists
- [SecLists](https://github.com/danielmiessler/SecLists)
- [CrackStation](https://crackstation.net/crackstation-wordlist-password-cracking-dictionary.htm)
- [Weakpass](https://weakpass.com/)

### Practice
- [HackTheBox](https://www.hackthebox.com/) - Many boxes require password cracking
- [TryHackMe](https://tryhackme.com/) - Password cracking rooms
- [CrackMe Challenges](https://crackmes.one/)

---

*Last updated: 2025-02-07*
