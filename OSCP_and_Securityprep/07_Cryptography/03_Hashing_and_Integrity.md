# 03 - Hashing and Integrity: One-Way Functions Explained

---
title: "Hashing and Integrity: Complete Guide to One-Way Functions"
parent: "[[07_Cryptography]]"
tags:
  - "hashing"
  - "integrity"
  - "md5"
  - "sha256"
  - "bcrypt"
  - "hmac"
created: 2026-02-07
updated: 2026-02-07
difficulty: "Beginner to Intermediate"
---

> **Executive Summary**: Hashing is a one-way mathematical transformation that creates a fixed-size "fingerprint" of any data. Unlike encryption, hashing cannot be reversed - you cannot get the original data back from a hash. Hashing is fundamental to password storage, file integrity verification, digital signatures, and blockchain technology. This comprehensive guide covers hash functions from absolute basics through advanced topics like HMAC, salting, and password hashing algorithms (bcrypt, Argon2).

**What You'll Learn**:
- What hashing is and why it's one-way
- Properties of cryptographic hash functions
- Common algorithms: MD5, SHA-1, SHA-256, SHA-3
- Password hashing: bcrypt, scrypt, Argon2
- HMAC for message authentication
- Hash attacks and defenses
- Real-world applications
- Hands-on labs with Python and command line

**Critical Understanding**: Hashing is NOT encryption! You cannot "decrypt" a hash!

---

## Table of Contents

1. [What is Hashing? (ABSOLUTE BEGINNER)](#1-what-is-hashing)
2. [Properties of Hash Functions](#2-properties-of-hash-functions)
3. [How Hash Functions Work](#3-how-hash-functions-work)
4. [Common Hash Algorithms](#4-common-hash-algorithms)
5. [MD5: The Broken Standard](#5-md5)
6. [SHA Family: The Workhorses](#6-sha-family)
7. [Password Hashing](#7-password-hashing)
8. [Salting: Why and How](#8-salting)
9. [bcrypt, scrypt, Argon2](#9-modern-password-hashing)
10. [HMAC: Keyed Hashing](#10-hmac)
11. [Hash Attacks](#11-hash-attacks)
12. [Real-World Applications](#12-real-world-applications)
13. [Tools and Commands](#13-tools-and-commands)
14. [Hands-On Labs](#14-hands-on-labs)
15. [Common Mistakes](#15-common-mistakes)
16. [Interview Questions](#16-interview-questions)
17. [Quick Reference](#17-quick-reference)

---

## 1. What is Hashing? (ABSOLUTE BEGINNER)

### 1.1 The Simple Explanation

A **hash function** is like a meat grinder:
- You put in a steak (input data)
- You get ground beef (hash output)
- You CANNOT reverse the process to get the steak back!

```
┌─────────────────────────────────────────────────────────────────┐
│                    HASHING VISUALIZATION                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   INPUT (any size)              OUTPUT (fixed size)             │
│   ─────────────────             ─────────────────               │
│                                                                  │
│   "Hello"          ───┐                                         │
│   (5 bytes)           │                                         │
│                       │     ┌──────────────────┐                │
│   "Hello World"   ───────>  │  HASH FUNCTION   │ ──> 256 bits  │
│   (11 bytes)          │     │    (SHA-256)     │                │
│                       │     └──────────────────┘                │
│   1 GB file       ───┘                                          │
│   (1,073,741,824                                                │
│    bytes)                                                        │
│                                                                  │
│   ALL outputs are exactly 256 bits (64 hex characters)          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 1.2 A Real-World Analogy

Think of hashing like a **fingerprint**:
- Every person has a unique fingerprint
- You can identify someone by their fingerprint
- But you CANNOT recreate the person from their fingerprint!

```
Person (input)        → Fingerprint (hash)
─────────────────────────────────────────
"password123"         → "ef92b778ba..."
"Password123"         → "b2e98ad6fe..."  ← Completely different!
"password124"         → "5e884898da..."  ← Completely different!
```

### 1.3 Key Concept: One-Way Function

```
+------------------------------------------------------------------+
|                    ONE-WAY FUNCTION                               |
+------------------------------------------------------------------+

ENCRYPTION (Two-way):
Plaintext ──encrypt──> Ciphertext ──decrypt──> Plaintext
                  ↑                       ↑
                 KEY                     KEY
Can go both directions with key!

HASHING (One-way):
Input ──────hash──────> Hash Output ──────X──────> ???
                                    
                Cannot reverse! No key involved!

+------------------------------------------------------------------+
|  You can NEVER get "password" back from its hash!                |
|  You can only GUESS passwords and hash them to compare.          |
+------------------------------------------------------------------+
```

### 1.4 Why Hashing Exists

| Use Case | Why Hashing? |
|----------|--------------|
| **Password Storage** | Store hash, not password. If database leaks, passwords aren't exposed |
| **File Integrity** | Compare hashes to verify file wasn't modified |
| **Digital Signatures** | Sign a small hash instead of entire document |
| **Deduplication** | Find duplicate files by comparing hashes |
| **Blockchain** | Chain blocks together with hashes |
| **Caching** | Use hash as cache key |

---

## 2. Properties of Hash Functions

### 2.1 The Five Essential Properties

A **cryptographic** hash function must have these properties:

```
+------------------------------------------------------------------+
|                HASH FUNCTION PROPERTIES                           |
+------------------------------------------------------------------+

1. DETERMINISTIC
   Same input ALWAYS produces same output
   
   hash("Hello") = "185f8db3..." (always!)
   hash("Hello") = "185f8db3..." (every time!)

2. FIXED OUTPUT SIZE
   Any input produces same size output
   
   hash("A")              = 256 bits
   hash("A very long...") = 256 bits

3. FAST TO COMPUTE
   Can hash large files quickly
   (But NOT too fast for passwords!)

4. ONE-WAY (Pre-image Resistance)
   Given hash H, cannot find input M where hash(M) = H
   (Cannot reverse the hash)

5. AVALANCHE EFFECT
   Small input change = drastically different output
   
   hash("Hello")  = "185f8db32271..."
   hash("Hello!") = "334d016f755c..."  ← Completely different!
```

### 2.2 Collision Resistance

**Collision**: When two different inputs produce the same hash.

```
+------------------------------------------------------------------+
|                    COLLISION RESISTANCE                           |
+------------------------------------------------------------------+

Weak Collision Resistance (Second Pre-image):
Given input M1, hard to find different M2 where hash(M1) = hash(M2)

Strong Collision Resistance:
Hard to find ANY two inputs M1 ≠ M2 where hash(M1) = hash(M2)

Why Collisions Exist:
- Input: Infinite possible values
- Output: Fixed size (e.g., 256 bits = 2^256 possible values)
- By Pigeonhole Principle, collisions MUST exist
- But finding them should be computationally infeasible!

What Happens When Collisions Are Found:
- MD5: Collisions can be generated in seconds → BROKEN
- SHA-1: Collisions demonstrated (SHAttered) → DEPRECATED
- SHA-256: No practical collisions → SECURE
```

### 2.3 Visual: Avalanche Effect

```python
import hashlib

# Tiny change = completely different hash
inputs = ["Hello", "Hello!", "hello", "Hello1"]

for text in inputs:
    h = hashlib.sha256(text.encode()).hexdigest()
    print(f"{text:10} → {h[:32]}...")

# Output:
# Hello      → 185f8db32271fe25f561a6fc938b2e26...
# Hello!     → 334d016f755cd6dc58c53a86e183882f...
# hello      → 2cf24dba5fb0a30e26e83b2ac5b9e29e...
# Hello1     → 948edbe7ede5aa7423476ae29dcd7d61...
```

---

## 3. How Hash Functions Work

### 3.1 High-Level Overview

```
+------------------------------------------------------------------+
|                HOW HASHING WORKS (SIMPLIFIED)                     |
+------------------------------------------------------------------+

Step 1: PADDING
Add bits to make input a multiple of block size
┌────────────────────────────────────────────────────┐
│ Original message │ 1 │ 000...000 │ Message length │
└────────────────────────────────────────────────────┘

Step 2: SPLIT INTO BLOCKS
Divide padded message into fixed-size blocks
┌─────────┬─────────┬─────────┬─────────┐
│ Block 1 │ Block 2 │ Block 3 │ Block 4 │
└─────────┴─────────┴─────────┴─────────┘

Step 3: PROCESS EACH BLOCK
Each block goes through compression function
┌──────────────────────────────────────────────────┐
│                                                   │
│  Initial    ┌─────────────┐                      │
│  Value  ───>│ Compression │──> Intermediate ──┐  │
│             │  Function   │      Hash         │  │
│  Block 1 ──>│             │                   │  │
│             └─────────────┘                   │  │
│                                               │  │
│  Intermediate ┌─────────────┐                 │  │
│  Hash     ───>│ Compression │──> Next     ───┘  │
│               │  Function   │    Hash           │
│  Block 2  ───>│             │                   │
│               └─────────────┘                   │
│                                                  │
│  ... repeat for all blocks ...                   │
│                                                  │
│  Final intermediate hash = OUTPUT HASH           │
└──────────────────────────────────────────────────┘
```

### 3.2 Merkle-Damgård Construction

Most hash functions (MD5, SHA-1, SHA-2) use this structure:

```
Message Blocks:  M1        M2        M3
                 │         │         │
                 v         v         v
IV ──> [f] ──> H1 ──> [f] ──> H2 ──> [f] ──> H3 (Final Hash)
        │              │              │
       M1             M2             M3

Where:
- IV = Initial Value (fixed constant)
- f = Compression function
- Each step mixes previous hash with new block
```

### 3.3 The Compression Function

The core of a hash function. For SHA-256:

```
Each round performs:
- Bitwise operations (AND, OR, XOR, NOT)
- Bit rotations and shifts
- Modular addition

64 rounds per block, each using:
- Message schedule (expanded from 16 to 64 words)
- Round constants (derived from cube roots of primes)
- Working variables (8 × 32-bit words)

Result: Thoroughly mixed, unpredictable output
```

---

## 4. Common Hash Algorithms

### 4.1 Algorithm Comparison

```
+------------------------------------------------------------------+
|                    HASH ALGORITHM COMPARISON                      |
+------------------------------------------------------------------+

Algorithm   Output Size   Block Size   Status        Speed
─────────────────────────────────────────────────────────────────
MD5         128 bits      512 bits     BROKEN        Fast
SHA-1       160 bits      512 bits     DEPRECATED    Fast
SHA-224     224 bits      512 bits     Secure        Medium
SHA-256     256 bits      512 bits     STANDARD      Medium
SHA-384     384 bits      1024 bits    Secure        Medium
SHA-512     512 bits      1024 bits    Secure        Fast (64-bit)
SHA-3-256   256 bits      1088 bits    SECURE        Medium
BLAKE2      256/512 bits  Variable     SECURE        Very Fast
BLAKE3      256 bits      64 bytes     SECURE        Fastest
```

### 4.2 Output Examples

```python
import hashlib

message = b"Hello, World!"

algorithms = ['md5', 'sha1', 'sha256', 'sha512']

for algo in algorithms:
    h = hashlib.new(algo, message).hexdigest()
    print(f"{algo.upper():8} ({len(h):3} chars): {h}")

# Output:
# MD5      ( 32 chars): 65a8e27d8879283831b664bd8b7f0ad4
# SHA1     ( 40 chars): 0a0a9f2a6772942557ab5355d76af442f8f65e01
# SHA256   ( 64 chars): dffd6021bb2bd5b0af676290809ec3a5319...
# SHA512   (128 chars): 374d794a95cdcfd8b35993185fef9ba368f1...
```

---

## 5. MD5: The Broken Standard

### 5.1 MD5 Overview

**MD5** (Message Digest 5) was designed in 1991 by Ron Rivest.

```
Specifications:
- Output: 128 bits (32 hex characters)
- Block size: 512 bits
- Rounds: 64

Status: BROKEN - Do NOT use for security!
```

### 5.2 Why MD5 is Broken

```
+------------------------------------------------------------------+
|                    MD5 COLLISION ATTACK                           |
+------------------------------------------------------------------+

2004: Wang et al. find collision in hours
2006: Collision in 1 minute
2012: Collision in seconds
Today: Collision in milliseconds!

What This Means:
- Two different files can have the SAME MD5 hash
- Attacker can create malicious file matching legitimate hash
- Certificate forgery possible (Flame malware used this)

Example Attack:
┌─────────────────┐     ┌─────────────────┐
│ Good Contract   │     │ Evil Contract   │
│ "Pay $1000"     │     │ "Pay $1000000"  │
└────────┬────────┘     └────────┬────────┘
         │                       │
         v                       v
    MD5: abc123...          MD5: abc123... (SAME!)
    
Get signature on good contract, swap with evil one!
```

### 5.3 When MD5 is Still Used

```
Acceptable Uses (Non-Security):
✓ File deduplication (finding identical files)
✓ Cache keys
✓ Non-cryptographic checksums
✓ Legacy system compatibility

NOT Acceptable:
✗ Password hashing
✗ Digital signatures
✗ Integrity verification (security-critical)
✗ Certificate generation
```

### 5.4 MD5 in Python

```python
import hashlib

# Basic MD5 hash
text = "password"
md5_hash = hashlib.md5(text.encode()).hexdigest()
print(f"MD5: {md5_hash}")
# Output: 5f4dcc3b5aa765d61d8327deb882cf99

# File hash
def md5_file(filepath):
    hasher = hashlib.md5()
    with open(filepath, 'rb') as f:
        for chunk in iter(lambda: f.read(4096), b''):
            hasher.update(chunk)
    return hasher.hexdigest()
```

---

## 6. SHA Family: The Workhorses

### 6.1 SHA-1 (Deprecated)

```
Specifications:
- Output: 160 bits (40 hex characters)
- Status: DEPRECATED (collision found in 2017)

SHAttered Attack (2017):
- Google and CWI Amsterdam created two PDFs with same SHA-1
- Cost: ~$110,000 in cloud computing
- Result: SHA-1 should NOT be used for security

Still Used In:
- Git (transitioning to SHA-256)
- Legacy systems
- Non-security checksums
```

### 6.2 SHA-256 (Current Standard)

```
Specifications:
- Output: 256 bits (64 hex characters)
- Block size: 512 bits
- Rounds: 64
- Status: SECURE - Recommended for most uses

Used In:
- TLS certificates
- Bitcoin/blockchain
- File integrity
- Code signing
- Password hashing (as component)
```

### 6.3 SHA-512

```
Specifications:
- Output: 512 bits (128 hex characters)
- Block size: 1024 bits
- Rounds: 80
- Status: SECURE

Interesting Fact:
- Actually FASTER than SHA-256 on 64-bit systems!
- Uses 64-bit operations (SHA-256 uses 32-bit)
```

### 6.4 SHA-3 (Keccak)

```
Specifications:
- Output: 224/256/384/512 bits (configurable)
- Structure: Sponge construction (different from SHA-2)
- Status: SECURE - Alternative to SHA-2

Why SHA-3 Exists:
- SHA-2 is not broken, but...
- SHA-3 uses completely different design
- Insurance against future SHA-2 attacks
- NIST competition winner (2012)
```

### 6.5 SHA in Python

```python
import hashlib

message = b"Secure Hash Algorithm"

# SHA-1 (deprecated)
sha1 = hashlib.sha1(message).hexdigest()
print(f"SHA-1:   {sha1}")

# SHA-256 (recommended)
sha256 = hashlib.sha256(message).hexdigest()
print(f"SHA-256: {sha256}")

# SHA-512
sha512 = hashlib.sha512(message).hexdigest()
print(f"SHA-512: {sha512}")

# SHA-3
sha3_256 = hashlib.sha3_256(message).hexdigest()
print(f"SHA3-256: {sha3_256}")

# Output lengths:
# SHA-1:    40 characters
# SHA-256:  64 characters
# SHA-512:  128 characters
# SHA3-256: 64 characters
```

---

## 7. Password Hashing

### 7.1 Why Passwords Need Special Hashing

Regular hash functions (SHA-256) are TOO FAST for passwords!

```
+------------------------------------------------------------------+
|                    PASSWORD HASHING PROBLEM                       |
+------------------------------------------------------------------+

SHA-256 Speed: ~1 billion hashes/second (GPU)

Attack Scenario:
- Attacker gets database of SHA-256 password hashes
- Common password list: 1 billion passwords
- Time to try all: 1 SECOND!

Solution: Use SLOW hash functions for passwords
- bcrypt: ~4 hashes/second (configurable)
- Time to try 1 billion: 250 MILLION seconds (8 years!)

+------------------------------------------------------------------+
|  Fast hashes are GOOD for file integrity                         |
|  Fast hashes are BAD for password storage                        |
+------------------------------------------------------------------+
```

### 7.2 Password Hashing Requirements

```
Requirements for Password Hash Functions:
─────────────────────────────────────────

1. SLOW (Configurable Work Factor)
   - Can increase difficulty over time
   - Makes brute force impractical

2. MEMORY-HARD (for modern algorithms)
   - Requires significant RAM
   - Defeats GPU/ASIC attacks

3. SALTED
   - Unique random value per password
   - Prevents rainbow tables
   - Same password → different hashes

4. RESISTANT TO PARALLELIZATION
   - Hard to speed up with multiple CPUs/GPUs
```

### 7.3 The Wrong Way

```python
import hashlib

# WRONG: Plain SHA-256
def bad_hash_password(password):
    return hashlib.sha256(password.encode()).hexdigest()

# Problems:
# 1. Too fast (billions per second)
# 2. No salt (rainbow tables work)
# 3. Same password = same hash

password1 = "password123"
password2 = "password123"

hash1 = bad_hash_password(password1)
hash2 = bad_hash_password(password2)

print(hash1 == hash2)  # True! Both have same hash!
```

### 7.4 The Right Way

```python
import bcrypt

# CORRECT: bcrypt with auto-generated salt
def good_hash_password(password):
    salt = bcrypt.gensalt(rounds=12)  # Work factor
    return bcrypt.hashpw(password.encode(), salt)

def verify_password(password, hashed):
    return bcrypt.checkpw(password.encode(), hashed)

# Usage
password = "password123"
hashed = good_hash_password(password)
print(f"Hash: {hashed}")

# Verify
print(verify_password("password123", hashed))  # True
print(verify_password("wrong", hashed))         # False

# Same password, different hashes (due to salt)!
hash1 = good_hash_password("password123")
hash2 = good_hash_password("password123")
print(hash1 == hash2)  # False! Different salts!
```

---

## 8. Salting: Why and How

### 8.1 What is a Salt?

A **salt** is random data added to input before hashing.

```
+------------------------------------------------------------------+
|                    SALTING EXPLAINED                              |
+------------------------------------------------------------------+

Without Salt:
password → hash("password") → 5f4dcc3b5aa765d61d8327deb882cf99

With Salt:
password + "x7Kj9m" → hash("passwordx7Kj9m") → 8b3e7a2f1c...
password + "Qw2sP4" → hash("passwordQw2sP4") → f91c3d8e2a...

Same password + different salts = different hashes!
```

### 8.2 Why Salting is Critical

```
+------------------------------------------------------------------+
|                    RAINBOW TABLE ATTACK                           |
+------------------------------------------------------------------+

Rainbow Table: Precomputed hash → password mappings

Without Salt:
┌─────────────────────────────────────────────────────────────────┐
│ Attacker precomputes:                                            │
│ hash("password")  = 5f4dcc3b...                                 │
│ hash("123456")    = e10adc39...                                 │
│ hash("qwerty")    = d8578edf...                                 │
│ ... millions more ...                                            │
│                                                                  │
│ Stolen hash: 5f4dcc3b...                                        │
│ Lookup in table → Found! Password is "password"                 │
│ Time: Milliseconds                                               │
└─────────────────────────────────────────────────────────────────┘

With Salt:
┌─────────────────────────────────────────────────────────────────┐
│ Each user has unique salt:                                       │
│ User1: hash("password" + "x7Kj9m") = 8b3e7a2f...                │
│ User2: hash("password" + "Qw2sP4") = f91c3d8e...                │
│                                                                  │
│ Attacker would need separate rainbow table for EACH salt!       │
│ With 16-byte salts: 2^128 possible salts                        │
│ Precomputation becomes infeasible                                │
└─────────────────────────────────────────────────────────────────┘
```

### 8.3 Salt Best Practices

```
Salt Requirements:
─────────────────────────────────────────

1. UNIQUE per password
   - Never reuse salts
   - Generate new salt for each password/update

2. RANDOM
   - Use cryptographic random generator
   - NOT: username, email, timestamp

3. SUFFICIENT LENGTH
   - Minimum: 16 bytes (128 bits)
   - Recommended: 32 bytes (256 bits)

4. STORED with hash
   - Salt is NOT secret
   - Store in database alongside hash
   - Format: $algorithm$salt$hash
```

### 8.4 Implementing Salting

```python
import os
import hashlib

def hash_with_salt(password, salt=None):
    """Hash password with salt (educational example - use bcrypt in practice!)"""
    if salt is None:
        salt = os.urandom(32)  # 256-bit random salt
    
    # Combine password and salt
    salted = salt + password.encode()
    
    # Hash the combination
    hashed = hashlib.sha256(salted).digest()
    
    # Return salt + hash (salt is needed for verification)
    return salt + hashed

def verify_with_salt(password, stored_hash):
    """Verify password against stored hash"""
    salt = stored_hash[:32]  # Extract salt
    expected_hash = stored_hash[32:]  # Extract hash
    
    # Hash the attempt
    attempt = hashlib.sha256(salt + password.encode()).digest()
    
    # Compare (use constant-time comparison in production!)
    return attempt == expected_hash

# Usage
stored = hash_with_salt("password123")
print(f"Stored hash length: {len(stored)} bytes")  # 64 bytes (32 salt + 32 hash)

print(verify_with_salt("password123", stored))  # True
print(verify_with_salt("wrong", stored))         # False
```

---

## 9. Modern Password Hashing: bcrypt, scrypt, Argon2

### 9.1 bcrypt

**bcrypt** (1999) - The proven workhorse.

```
Features:
- Built-in salt (128-bit)
- Configurable work factor (cost)
- Based on Blowfish cipher
- Resistant to GPU attacks

Format: $2b$cost$salt+hash
Example: $2b$12$LQv3c1yqBWVHxkd0LHAkCOYz6TtxMQJqhN8/X4.bNLeMxPkzLEK/u

Cost Factor:
- cost=10: ~100ms
- cost=12: ~300ms (recommended)
- cost=14: ~1s

Python:
import bcrypt
hashed = bcrypt.hashpw(b"password", bcrypt.gensalt(rounds=12))
```

### 9.2 scrypt

**scrypt** (2009) - Memory-hard function.

```
Features:
- Configurable CPU cost (N)
- Configurable memory cost (r)
- Configurable parallelization (p)
- Designed to defeat hardware attacks

Parameters:
- N: CPU/memory cost (power of 2)
- r: Block size
- p: Parallelization factor

Python:
import hashlib
hashed = hashlib.scrypt(
    b"password",
    salt=os.urandom(32),
    n=16384,   # CPU cost
    r=8,       # Block size
    p=1        # Parallelization
)
```

### 9.3 Argon2 (Recommended)

**Argon2** (2015) - Winner of Password Hashing Competition.

```
Variants:
- Argon2d: Maximum GPU resistance (side-channel vulnerable)
- Argon2i: Side-channel resistant
- Argon2id: Hybrid (RECOMMENDED)

Parameters:
- time_cost: Number of iterations
- memory_cost: Memory in KB
- parallelism: Number of threads

Format: $argon2id$v=19$m=65536,t=3,p=4$salt$hash

Python:
from argon2 import PasswordHasher
ph = PasswordHasher(
    time_cost=3,
    memory_cost=65536,
    parallelism=4
)
hashed = ph.hash("password")
```

### 9.4 Comparison

```
+------------------------------------------------------------------+
|              PASSWORD HASHING ALGORITHM COMPARISON                |
+------------------------------------------------------------------+

Algorithm   Memory-Hard   GPU Resistant   Recommended
────────────────────────────────────────────────────────────────
bcrypt      No            Yes             Yes (proven)
scrypt      Yes           Yes             Yes
Argon2id    Yes           Yes             Yes (best)
PBKDF2      No            No              Legacy only
SHA-256     No            No              NEVER

Recommendation:
1. New projects: Argon2id
2. Existing projects: bcrypt is fine
3. Compatibility needed: bcrypt (widest support)
```

### 9.5 Python Examples

```python
# bcrypt
import bcrypt

password = b"secure_password"
hashed = bcrypt.hashpw(password, bcrypt.gensalt(rounds=12))
print(f"bcrypt: {hashed.decode()}")

verified = bcrypt.checkpw(password, hashed)
print(f"Verified: {verified}")

# Argon2
from argon2 import PasswordHasher, exceptions

ph = PasswordHasher()
hashed = ph.hash("secure_password")
print(f"Argon2: {hashed}")

try:
    ph.verify(hashed, "secure_password")
    print("Password correct!")
except exceptions.VerifyMismatchError:
    print("Wrong password!")
```

---

## 10. HMAC: Keyed Hashing

### 10.1 What is HMAC?

**HMAC** (Hash-based Message Authentication Code) adds a secret key to hashing.

```
+------------------------------------------------------------------+
|                    HMAC vs REGULAR HASH                           |
+------------------------------------------------------------------+

Regular Hash:
- Anyone can compute hash("message")
- Cannot verify WHO created the hash

HMAC:
- Only key holder can compute HMAC(key, "message")
- Receiver with same key can verify authenticity

Use Case:
API sends: "transfer $100" + HMAC
Server verifies HMAC with shared secret key
If HMAC matches: Request is authentic and unmodified
```

### 10.2 Why Not Just hash(key + message)?

```
+------------------------------------------------------------------+
|                    LENGTH EXTENSION ATTACK                        |
+------------------------------------------------------------------+

Vulnerable: hash(secret + message)

Attack:
1. Attacker sees: message = "amount=100"
2. Attacker sees: hash(secret + "amount=100") = abc123
3. Attacker can compute: hash(secret + "amount=100" + padding + "&admin=true")
   WITHOUT knowing the secret!

This works because of Merkle-Damgård construction.

HMAC prevents this:
HMAC(K, M) = H((K ⊕ opad) || H((K ⊕ ipad) || M))

The nested structure prevents length extension.
```

### 10.3 HMAC Construction

```
HMAC(K, M) = H((K ⊕ opad) || H((K ⊕ ipad) || M))

Where:
- K = Secret key (padded to block size)
- M = Message
- H = Hash function (SHA-256, etc.)
- ipad = 0x36 repeated
- opad = 0x5c repeated
- || = Concatenation
- ⊕ = XOR

Visual:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   Key ─────> XOR with ipad ─────┐                               │
│                                  │                               │
│   Message ──────────────────────┼──> Hash ──> Inner Hash        │
│                                                    │             │
│   Key ─────> XOR with opad ─────┼──────────────────┘            │
│                                  │                               │
│                                  └──> Hash ──> HMAC Output      │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 10.4 HMAC in Python

```python
import hmac
import hashlib

# Create HMAC
key = b"secret_key"
message = b"Important message"

# Method 1: hmac module
mac = hmac.new(key, message, hashlib.sha256).hexdigest()
print(f"HMAC-SHA256: {mac}")

# Method 2: Using digest for binary output
mac_bytes = hmac.new(key, message, hashlib.sha256).digest()

# Verify HMAC (use constant-time comparison!)
def verify_hmac(key, message, received_mac):
    computed = hmac.new(key, message, hashlib.sha256).hexdigest()
    return hmac.compare_digest(computed, received_mac)

print(verify_hmac(key, message, mac))  # True
print(verify_hmac(key, b"tampered", mac))  # False
```

### 10.5 HMAC Use Cases

```
1. API Authentication
   Client: request + HMAC(api_secret, request)
   Server: Verifies HMAC, processes if valid

2. JWT Signatures (HS256)
   header.payload.HMAC(secret, header + payload)

3. Cookie Integrity
   cookie_value + HMAC(secret, cookie_value)
   Server verifies cookie wasn't tampered

4. Webhook Verification
   GitHub/Stripe send: payload + HMAC signature
   Your server verifies authenticity
```

---

## 11. Hash Attacks

### 11.1 Brute Force Attack

```
Attack: Try all possible inputs until hash matches

Defense:
- Long passwords (exponential search space)
- Slow hash functions (bcrypt, Argon2)

Example:
8-character password (lowercase only): 26^8 = 208 billion
With SHA-256 (1 billion/sec): 3.5 minutes
With bcrypt (1000/sec): 6.6 years!
```

### 11.2 Dictionary Attack

```
Attack: Try common passwords from wordlist

rockyou.txt: 14 million passwords
Time with SHA-256: < 1 second
Time with bcrypt: ~4 hours

Defense:
- Password policies (avoid common passwords)
- Slow hash functions
```

### 11.3 Rainbow Table Attack

```
Attack: Precomputed hash → password tables

Defense:
- Salting (unique salt per password)
- Modern algorithms (bcrypt/Argon2 include salt)

With 16-byte salt:
Rainbow table would need 2^128 variants
= Computationally impossible
```

### 11.4 Collision Attack

```
Attack: Find two inputs with same hash

Impact:
- Forge digital signatures
- Bypass integrity checks
- Create malicious files matching legitimate hashes

Defense:
- Use collision-resistant algorithms (SHA-256, SHA-3)
- Avoid MD5 and SHA-1
```

### 11.5 Birthday Attack

```
Theory: Birthday Paradox

For n-bit hash, expected collisions after 2^(n/2) attempts

MD5 (128-bit): Collision expected after 2^64 attempts
SHA-256 (256-bit): Collision expected after 2^128 attempts

2^64 is feasible today
2^128 is not feasible with current technology
```

---

## 12. Real-World Applications

### 12.1 Password Storage

```python
# Secure password storage workflow
from argon2 import PasswordHasher

ph = PasswordHasher()

# Registration
def register_user(username, password):
    hashed = ph.hash(password)
    # Store in database: (username, hashed)
    return save_to_db(username, hashed)

# Login
def login_user(username, password):
    stored_hash = get_from_db(username)
    try:
        ph.verify(stored_hash, password)
        return True
    except:
        return False
```

### 12.2 File Integrity Verification

```python
import hashlib

def get_file_hash(filepath):
    """Calculate SHA-256 hash of a file"""
    sha256 = hashlib.sha256()
    with open(filepath, 'rb') as f:
        for chunk in iter(lambda: f.read(4096), b''):
            sha256.update(chunk)
    return sha256.hexdigest()

# Verify download
expected = "abc123..."  # From trusted source
actual = get_file_hash("downloaded_file.iso")
if expected == actual:
    print("File integrity verified!")
else:
    print("WARNING: File may be corrupted or tampered!")
```

### 12.3 Git Commits

```bash
# Git uses SHA-1 (transitioning to SHA-256)
$ git log --oneline
a1b2c3d First commit
d4e5f6g Second commit

# Commit hash includes:
# - Tree hash (files)
# - Parent commit hash
# - Author/committer info
# - Timestamp
# - Commit message
```

### 12.4 Blockchain

```
Each block contains:
- Previous block's hash
- Transaction data
- Nonce (for proof of work)
- Timestamp

Block Hash = SHA256(previous_hash + transactions + nonce)

If ANY data changes, hash changes, chain breaks!
```

---

## 13. Tools and Commands

### 13.1 Command Line

```bash
# Linux/Mac

# MD5 (use for non-security purposes only)
md5sum file.txt
echo -n "text" | md5sum

# SHA-256
sha256sum file.txt
echo -n "text" | sha256sum

# SHA-512
sha512sum file.txt

# OpenSSL (any algorithm)
openssl dgst -sha256 file.txt
openssl dgst -md5 file.txt

# HMAC with OpenSSL
echo -n "message" | openssl dgst -sha256 -hmac "secret_key"

# Windows PowerShell
Get-FileHash file.txt -Algorithm SHA256
```

### 13.2 Python

```python
import hashlib
import hmac

# Basic hashing
hashlib.md5(b"text").hexdigest()
hashlib.sha256(b"text").hexdigest()
hashlib.sha512(b"text").hexdigest()
hashlib.sha3_256(b"text").hexdigest()

# HMAC
hmac.new(b"key", b"message", hashlib.sha256).hexdigest()

# File hashing
def hash_file(path, algorithm='sha256'):
    h = hashlib.new(algorithm)
    with open(path, 'rb') as f:
        for chunk in iter(lambda: f.read(4096), b''):
            h.update(chunk)
    return h.hexdigest()
```

### 13.3 Online Tools

```
CyberChef: https://gchq.github.io/CyberChef/
- Hash generation
- Hash identification
- Multiple algorithms

Hash Identifier: https://hashes.com/en/tools/hash_identifier
- Identify unknown hash types

Crackstation: https://crackstation.net/
- Look up hashes (unsalted only)
- For research/recovery only!
```

---

## 14. Hands-On Labs

### Lab 1: Hash Comparison

```python
"""
Compare different hash algorithms
"""
import hashlib
import time

message = b"The quick brown fox jumps over the lazy dog"

algorithms = ['md5', 'sha1', 'sha256', 'sha512', 'sha3_256']

print("Algorithm    | Output Length | Hash (first 32 chars)")
print("-" * 70)

for algo in algorithms:
    h = hashlib.new(algo, message).hexdigest()
    print(f"{algo:12} | {len(h):13} | {h[:32]}...")
```

### Lab 2: Password Cracking Simulation

```python
"""
Demonstrate why fast hashes are bad for passwords
"""
import hashlib
import time

# Common passwords to "crack"
wordlist = ["password", "123456", "password123", "admin", "letmein"]

# Target hash (SHA-256 of "password")
target = hashlib.sha256(b"password").hexdigest()

# Simulate brute force
start = time.time()
for word in wordlist * 1000000:  # Repeat for timing
    if hashlib.sha256(word.encode()).hexdigest() == target:
        print(f"Found: {word}")
        break
elapsed = time.time() - start
print(f"Time: {elapsed:.2f} seconds for {len(wordlist) * 1000000} attempts")
print(f"Rate: {len(wordlist) * 1000000 / elapsed:,.0f} hashes/second")
```

### Lab 3: HMAC Verification

```python
"""
Implement API request signing with HMAC
"""
import hmac
import hashlib
import json
import time

def sign_request(api_key, method, path, body, timestamp):
    """Create HMAC signature for API request"""
    message = f"{method}\n{path}\n{timestamp}\n{body}"
    signature = hmac.new(
        api_key.encode(),
        message.encode(),
        hashlib.sha256
    ).hexdigest()
    return signature

def verify_request(api_key, method, path, body, timestamp, signature):
    """Verify HMAC signature"""
    expected = sign_request(api_key, method, path, body, timestamp)
    return hmac.compare_digest(expected, signature)

# Test
API_KEY = "super_secret_key"
timestamp = str(int(time.time()))
body = json.dumps({"amount": 100, "recipient": "alice"})

sig = sign_request(API_KEY, "POST", "/transfer", body, timestamp)
print(f"Signature: {sig}")

# Verify
print(f"Valid: {verify_request(API_KEY, 'POST', '/transfer', body, timestamp, sig)}")
print(f"Tampered: {verify_request(API_KEY, 'POST', '/transfer', 'modified', timestamp, sig)}")
```

---

## 15. Common Mistakes

### Mistake 1: Using MD5/SHA-1 for Security

```python
# WRONG
password_hash = hashlib.md5(password.encode()).hexdigest()

# CORRECT
password_hash = bcrypt.hashpw(password.encode(), bcrypt.gensalt())
```

### Mistake 2: No Salt for Passwords

```python
# WRONG - Same password = same hash
hash1 = hashlib.sha256(b"password").hexdigest()
hash2 = hashlib.sha256(b"password").hexdigest()
# hash1 == hash2 (vulnerable to rainbow tables!)

# CORRECT - Different salts = different hashes
hash1 = bcrypt.hashpw(b"password", bcrypt.gensalt())
hash2 = bcrypt.hashpw(b"password", bcrypt.gensalt())
# hash1 != hash2 (rainbow tables don't work)
```

### Mistake 3: Comparing Hashes with ==

```python
# WRONG - Timing attack vulnerability
if computed_hash == provided_hash:
    return True

# CORRECT - Constant-time comparison
import hmac
if hmac.compare_digest(computed_hash, provided_hash):
    return True
```

### Mistake 4: Thinking Hash = Encryption

```
WRONG: "I hashed the data so it's encrypted"
FACT: Hashing is one-way, you cannot get the data back!

If you need to retrieve original data: use ENCRYPTION
If you only need to verify: use HASHING
```

---

## 16. Interview Questions

**Q1: What is the difference between hashing and encryption?**
> Hashing is one-way (cannot recover original), produces fixed-size output, no key required. Encryption is two-way (can decrypt), preserves data size roughly, requires key.

**Q2: Why is MD5 considered broken?**
> Collisions can be generated quickly - two different inputs producing the same hash. This breaks integrity guarantees and enables signature forgery.

**Q3: What is a salt and why is it important?**
> A salt is random data added to a password before hashing. It ensures same passwords produce different hashes, defeating rainbow table attacks.

**Q4: Why use bcrypt instead of SHA-256 for passwords?**
> bcrypt is intentionally slow (configurable), includes built-in salt, and is resistant to GPU acceleration. SHA-256 is too fast (billions/second), making brute force easy.

**Q5: What is HMAC and when would you use it?**
> HMAC is a keyed hash providing message authentication. Use it when you need to verify both integrity AND authenticity - that the message wasn't modified AND came from someone with the secret key.

**Q6: Explain the avalanche effect.**
> A small change in input (even one bit) produces a drastically different hash output. "Hello" and "hello" produce completely different hashes.

---

## 17. Quick Reference

### Algorithm Recommendations

| Use Case | Algorithm |
|----------|-----------|
| File integrity | SHA-256 |
| Password storage | Argon2id or bcrypt |
| Message authentication | HMAC-SHA256 |
| Digital signatures | SHA-256 (with RSA/ECDSA) |
| Checksums (non-security) | MD5 or CRC32 |

### Hash Output Sizes

| Algorithm | Bits | Hex Chars |
|-----------|------|-----------|
| MD5 | 128 | 32 |
| SHA-1 | 160 | 40 |
| SHA-256 | 256 | 64 |
| SHA-512 | 512 | 128 |

### Python Quick Reference

```python
import hashlib
import hmac
import bcrypt
from argon2 import PasswordHasher

# Simple hash
hashlib.sha256(b"text").hexdigest()

# File hash
hashlib.sha256(open("file", "rb").read()).hexdigest()

# HMAC
hmac.new(b"key", b"message", hashlib.sha256).hexdigest()

# Password (bcrypt)
bcrypt.hashpw(b"password", bcrypt.gensalt(rounds=12))
bcrypt.checkpw(b"password", hashed)

# Password (Argon2)
ph = PasswordHasher()
ph.hash("password")
ph.verify(hash, "password")
```

### Command Line

```bash
# SHA-256
sha256sum file.txt
echo -n "text" | sha256sum

# HMAC
echo -n "message" | openssl dgst -sha256 -hmac "key"
```

---

## Next Steps

Continue to:
- **[[04_Symmetric_Encryption]]** - AES and block ciphers
- **[[07_Password_Cracking]]** - Breaking hashes with Hashcat/John

---

## References

- [[01_Cryptography_Fundamentals]]
- [[02_Encoding_Fundamentals]]
- [OWASP Password Storage Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)
- [RFC 2104 - HMAC](https://tools.ietf.org/html/rfc2104)
