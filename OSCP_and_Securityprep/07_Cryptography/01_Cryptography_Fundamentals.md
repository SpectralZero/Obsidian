# 01 - Cryptography Fundamentals: The Complete Introduction

---
title: "Cryptography Fundamentals: From Zero to Understanding"
parent: "[[07_Cryptography]]"
tags:
  - "crypto"
  - "fundamentals"
  - "security"
  - "basics"
  - "introduction"
created: 2026-02-07
updated: 2026-02-07
difficulty: "Beginner"
---

> **Executive Summary**: Cryptography is the science of securing information. It's not just about "encryption" - it encompasses encoding, hashing, digital signatures, key exchange, and much more. This foundational chapter explains cryptography from absolute zero, covering its history, the three pillars of security (Confidentiality, Integrity, Authentication), the critical differences between encoding/encryption/hashing, and the fundamental concepts every security professional must master. By the end, you'll understand WHY cryptography exists, WHAT problems it solves, and HOW to identify different cryptographic techniques in the wild.

**What You'll Learn**:
- The complete history and evolution of cryptography
- CIA Triad and security fundamentals
- Encoding vs Hashing vs Encryption (the CRITICAL distinction)
- Symmetric vs Asymmetric cryptography
- Key management fundamentals
- Cryptographic primitives and building blocks
- Real-world applications in security
- How this connects to all other crypto topics

**Prerequisites**: None - this is truly from zero!

---

## Table of Contents

1. [What is Cryptography? (ABSOLUTE BEGINNER)](#1-what-is-cryptography)
2. [A Brief History of Cryptography](#2-history-of-cryptography)
3. [The CIA Triad: Why Crypto Exists](#3-the-cia-triad)
4. [The Three Pillars: Encoding, Hashing, Encryption](#4-the-three-pillars)
5. [Symmetric vs Asymmetric Cryptography](#5-symmetric-vs-asymmetric)
6. [Keys: The Heart of Cryptography](#6-keys)
7. [Cryptographic Primitives](#7-cryptographic-primitives)
8. [Common Algorithms Overview](#8-common-algorithms)
9. [Real-World Applications](#9-real-world-applications)
10. [The Attacker's Perspective](#10-attackers-perspective)
11. [Kerckhoffs's Principle](#11-kerckhoffs-principle)
12. [Entropy and Randomness](#12-entropy-and-randomness)
13. [Common Mistakes in Cryptography](#13-common-mistakes)
14. [Tools for Crypto Analysis](#14-tools)
15. [How This Module Connects](#15-module-connections)
16. [Hands-On Labs](#16-hands-on-labs)
17. [Interview Questions](#17-interview-questions)
18. [Quick Reference](#18-quick-reference)

---

## 1. What is Cryptography? (ABSOLUTE BEGINNER)

### 1.1 The Simple Definition

**Cryptography** comes from Greek:
- **Kryptos** = Hidden
- **Graphein** = Writing

It literally means "hidden writing" - the art and science of keeping secrets.

### 1.2 A Real-World Analogy

Imagine you want to send a secret message to your friend across the classroom:

```
+------------------------------------------------------------------+
|                    SECRET MESSAGE PROBLEM                         |
+------------------------------------------------------------------+

You want to send: "Meet me at 3pm"

Problem: Anyone who intercepts the note can read it!

Solution 1: ENCODING (Simple transformation)
"Nffu nf bu 4qn"  (Each letter shifted by 1)
Problem: Anyone who knows the method can decode it

Solution 2: ENCRYPTION (Requires a secret)
"X7$kL@9mN!" + You and friend share a SECRET KEY
Only someone with the key can decrypt

Solution 3: LOCKED BOX (Asymmetric)
Friend gives you an OPEN PADLOCK (public key)
You lock the message, only friend has the KEY (private key)
```

### 1.3 Why Cryptography Matters in Security

```
+------------------------------------------------------------------+
|                    WITHOUT CRYPTOGRAPHY                           |
+------------------------------------------------------------------+

[You] ------ "Password: hunter2" -------> [Bank Website]
                     ↑
               [Attacker]
          "I can see everything!"

+------------------------------------------------------------------+
|                    WITH CRYPTOGRAPHY                              |
+------------------------------------------------------------------+

[You] ------ "X7#kL$9nM!@pQ" -------> [Bank Website]
                     ↑                        ↓
               [Attacker]              [Decrypts with key]
          "Gibberish to me!"           "Password: hunter2"
```

### 1.4 The Scope of Cryptography

Cryptography is much broader than just "encryption":

```
+------------------------------------------------------------------+
|                    CRYPTOGRAPHY UNIVERSE                          |
+------------------------------------------------------------------+
|                                                                   |
|  ┌─────────────────────────────────────────────────────────────┐  |
|  │                     CRYPTOGRAPHY                             │  |
|  │                                                              │  |
|  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐       │  |
|  │  │  ENCODING    │  │   HASHING    │  │ ENCRYPTION   │       │  |
|  │  │              │  │              │  │              │       │  |
|  │  │ Base64, Hex  │  │ MD5, SHA256  │  │ AES, RSA     │       │  |
|  │  │ URL, ASCII   │  │ bcrypt       │  │ ChaCha20     │       │  |
|  │  └──────────────┘  └──────────────┘  └──────────────┘       │  |
|  │                                                              │  |
|  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐       │  |
|  │  │ SIGNATURES   │  │ KEY EXCHANGE │  │ CERTIFICATES │       │  |
|  │  │              │  │              │  │              │       │  |
|  │  │ RSA, ECDSA   │  │ DH, ECDH     │  │ X.509, PKI   │       │  |
|  │  └──────────────┘  └──────────────┘  └──────────────┘       │  |
|  │                                                              │  |
|  └─────────────────────────────────────────────────────────────┘  |
+------------------------------------------------------------------+
```

---

## 2. A Brief History of Cryptography

Understanding history helps you understand why things work the way they do.

### 2.1 Ancient Cryptography

**Caesar Cipher (50 BC)**
Julius Caesar shifted letters by 3 positions:
```
Plaintext:  ATTACK AT DAWN
Shift by 3: DWWDFN DW GDZQ

A → D, B → E, C → F, ... X → A, Y → B, Z → C
```

**Weakness**: Only 26 possible keys - easy to brute force!

**Scytale (Ancient Greece)**
Wrap a strip of leather around a rod of specific diameter. Message only readable with same size rod.

### 2.2 Classical Cryptography

**Vigenère Cipher (1553)**
Uses a keyword to shift each letter differently:
```
Plaintext:  ATTACKATDAWN
Keyword:    LEMONLEMONLE (repeated)
Ciphertext: LXFOPVEFRNHR

A + L = L, T + E = X, T + M = F, ...
```

**Weakness**: Key length can be found through frequency analysis (Kasiski examination).

**Enigma Machine (WWII)**
Electro-mechanical cipher machine used by Nazi Germany:
- Multiple rotating wheels
- Plugboard for additional substitution
- Changed settings daily

**Breaking it changed the war** - Alan Turing and team at Bletchley Park.

### 2.3 Modern Cryptography

**1970s: DES (Data Encryption Standard)**
- First standardized encryption
- 56-bit key (now broken)

**1976: Diffie-Hellman Key Exchange**
- Revolutionary: Exchange keys over insecure channel!
- Foundation of modern secure communications

**1977: RSA**
- First practical public-key cryptosystem
- Still used today (with larger keys)

**2001: AES (Advanced Encryption Standard)**
- Replaced DES
- Current standard (128/192/256-bit keys)

### 2.4 Timeline Visualization

```
+------------------------------------------------------------------+
|                    CRYPTOGRAPHY TIMELINE                          |
+------------------------------------------------------------------+

Ancient         Classical           Modern              Post-Quantum
(BC-1800)       (1800-1970)        (1970-2020)         (2020+)
    |               |                  |                   |
    |               |                  |                   |
    v               v                  v                   v
┌───────┐       ┌───────┐         ┌───────┐          ┌───────┐
│Caesar │       │Enigma │         │DES    │          │Lattice│
│Scytale│       │       │         │RSA    │          │Based  │
│       │       │       │         │AES    │          │       │
└───────┘       └───────┘         └───────┘          └───────┘
    │               │                  │                   │
    │   Broken by   │   Broken by     │   Quantum        │
    │   frequency   │   Turing &      │   computers      │
    │   analysis    │   Bombes        │   threaten       │
    │               │                  │   RSA/ECC       │
```

---

## 3. The CIA Triad: Why Crypto Exists

Cryptography exists to protect the **CIA Triad** (not the US agency!):

### 3.1 Confidentiality

**Definition**: Only authorized parties can access the information.

```
┌─────────────────────────────────────────────────────────────────┐
│                      CONFIDENTIALITY                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  [Sender] ──encrypted message──> [Receiver]                      │
│                    ↑                                             │
│             [Eavesdropper]                                       │
│           "I can't read this!"                                   │
│                                                                  │
│  Achieved by: ENCRYPTION (AES, RSA, ChaCha20)                   │
│                                                                  │
│  Examples:                                                       │
│  - HTTPS encrypts web traffic                                    │
│  - WhatsApp end-to-end encryption                               │
│  - BitLocker encrypts hard drives                               │
└─────────────────────────────────────────────────────────────────┘
```

### 3.2 Integrity

**Definition**: Data has not been modified in transit or storage.

```
┌─────────────────────────────────────────────────────────────────┐
│                        INTEGRITY                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Original:  "Transfer $100"  → Hash: abc123                      │
│  Modified:  "Transfer $999"  → Hash: xyz789 ← DIFFERENT!         │
│                                                                  │
│  Achieved by: HASHING (SHA-256, HMAC)                           │
│                                                                  │
│  Examples:                                                       │
│  - File checksums (did download corrupt?)                       │
│  - Git commits (was code modified?)                             │
│  - Blockchain (was transaction altered?)                        │
└─────────────────────────────────────────────────────────────────┘
```

### 3.3 Authentication

**Definition**: Verify the identity of parties involved.

```
┌─────────────────────────────────────────────────────────────────┐
│                      AUTHENTICATION                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  "Is this really from Alice?"                                    │
│                                                                  │
│  Alice signs with Private Key ──> Anyone verifies with Public   │
│                                                                  │
│  Achieved by: DIGITAL SIGNATURES (RSA-SHA256, ECDSA)            │
│                                                                  │
│  Examples:                                                       │
│  - Code signing (is this app from Microsoft?)                   │
│  - Email signatures (is this email really from CEO?)            │
│  - SSL certificates (is this really google.com?)                │
└─────────────────────────────────────────────────────────────────┘
```

### 3.4 Additional Security Properties

Beyond CIA, cryptography also provides:

| Property                  | Description                             | Achieved By            |
| ------------------------- | --------------------------------------- | ---------------------- |
| **Non-repudiation**       | Sender cannot deny sending              | Digital signatures     |
| **Forward Secrecy**       | Past sessions stay secure if key leaked | Ephemeral keys (ECDHE) |
| **Anonymity**             | Hide identity of parties                | Onion routing (Tor)    |
| **Plausible Deniability** | Cannot prove what was encrypted         | Deniable encryption    |

---

## 4. The Three Pillars: Encoding, Hashing, Encryption

This is the **MOST IMPORTANT** section. Most security mistakes come from confusing these!

### 4.1 The Critical Comparison

```
+------------------------------------------------------------------+
|              ENCODING vs HASHING vs ENCRYPTION                    |
+------------------------------------------------------------------+

                    ENCODING
                 (Transformation)
                       │
    Input: "Hello" ────┼───> Output: "SGVsbG8="
                       │
    • Reversible: YES (by anyone)
    • Key required: NO
    • Security: NONE
    • Purpose: Format conversion
    • Examples: Base64, Hex, URL encoding

─────────────────────────────────────────────────────────────────

                     HASHING
                 (One-way function)
                       │
    Input: "Hello" ────┼───> Output: "2cf24dba..."
                       │
    • Reversible: NO (mathematically impossible)
    • Key required: NO
    • Security: Integrity only
    • Purpose: Fingerprint/verification
    • Examples: MD5, SHA-256, bcrypt

─────────────────────────────────────────────────────────────────

                    ENCRYPTION
                (Two-way with key)
                       │
    Input: "Hello" ────┼───> Output: "X7$kL@9"
         + Key: "secret"
                       │
    • Reversible: YES (only with key)
    • Key required: YES
    • Security: Confidentiality
    • Purpose: Secrecy
    • Examples: AES, RSA, ChaCha20
```

### 4.2 Encoding Explained

**What it is**: Converting data from one format to another.

**Why it exists**: 
- Make binary data safe for text-based protocols
- Standardize data representation

**Critical Point**: Encoding provides **NO SECURITY**!

```python
import base64

# Anyone can encode
message = "Secret Password: hunter2"
encoded = base64.b64encode(message.encode())
print(f"Encoded: {encoded}")
# Output: U2VjcmV0IFBhc3N3b3JkOiBodW50ZXIy

# Anyone can decode (no key needed!)
decoded = base64.b64decode(encoded)
print(f"Decoded: {decoded.decode()}")
# Output: Secret Password: hunter2
```

**Common Mistake**:
```
BAD: "I Base64 encoded the password, so it's secure!"
REALITY: Base64 is NOT encryption. Anyone can decode it.
```

### 4.3 Hashing Explained

**What it is**: One-way mathematical transformation producing fixed-length output.

**Properties**:
- **Deterministic**: Same input = Same hash (always)
- **One-way**: Cannot reverse hash to get input
- **Fixed size**: Any input produces same size output
- **Avalanche**: Small change = Completely different hash

```python
import hashlib

# Hashing examples
print(hashlib.sha256(b"Hello").hexdigest())
# 185f8db32271fe25f561a6fc938b2e264306ec304eda518007d1764826381969

print(hashlib.sha256(b"Hello!").hexdigest())  # Just added "!"
# 334d016f755cd6dc58c53a86e183882f8ec14f52fb05345887c8a5edd42c87b7
# COMPLETELY different!

# You CANNOT reverse this to get "Hello" back
# You can only try guessing until you find a match
```

**Use Cases**:
- Password storage (hash passwords, not plain text!)
- File integrity verification
- Data deduplication
- Digital signatures (hash then sign)

### 4.4 Encryption Explained

**What it is**: Scrambling data with a key so only key holders can unscramble.

**Two Types**:
1. **Symmetric**: Same key encrypts and decrypts (AES, ChaCha20)
2. **Asymmetric**: Different keys for encrypt/decrypt (RSA, ECC)

```python
# Symmetric encryption example (simplified concept)
from cryptography.fernet import Fernet

# Generate a key (keep this SECRET!)
key = Fernet.generate_key()
cipher = Fernet(key)

# Encrypt
message = b"Top Secret Data"
encrypted = cipher.encrypt(message)
print(f"Encrypted: {encrypted}")
# Output: gAAAAABh... (gibberish without key)

# Decrypt (needs the same key!)
decrypted = cipher.decrypt(encrypted)
print(f"Decrypted: {decrypted}")
# Output: b'Top Secret Data'
```

### 4.5 Visual Summary

```
+------------------------------------------------------------------+
|                    DECISION FLOWCHART                             |
+------------------------------------------------------------------+
|                                                                   |
|  "What should I use?"                                             |
|           │                                                       |
|           v                                                       |
|  ┌────────────────────┐                                          |
|  │ Need to keep data  │──YES──> ENCRYPTION                       |
|  │ secret?            │         (AES, RSA)                       |
|  └────────────────────┘                                          |
|           │ NO                                                    |
|           v                                                       |
|  ┌────────────────────┐                                          |
|  │ Need to verify     │──YES──> HASHING                          |
|  │ integrity? (but    │         (SHA-256, HMAC)                  |
|  │ don't need to      │                                          |
|  │ reverse it)        │                                          |
|  └────────────────────┘                                          |
|           │ NO                                                    |
|           v                                                       |
|  ┌────────────────────┐                                          |
|  │ Need to convert    │──YES──> ENCODING                         |
|  │ format? (no        │         (Base64, Hex)                    |
|  │ security needed)   │                                          |
|  └────────────────────┘                                          |
|                                                                   |
+------------------------------------------------------------------+
```

---

## 5. Symmetric vs Asymmetric Cryptography

### 5.1 Symmetric Cryptography

**Concept**: Same key for encryption and decryption.

```
┌─────────────────────────────────────────────────────────────────┐
│                  SYMMETRIC ENCRYPTION                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│                      SAME KEY                                    │
│                         │                                        │
│    ┌──────────┐        │        ┌──────────┐                    │
│    │ Plaintext│──Encrypt with───>│Ciphertext│                   │
│    └──────────┘      Key        └──────────┘                    │
│                         │                                        │
│    ┌──────────┐        │        ┌──────────┐                    │
│    │ Plaintext│<─Decrypt with───│Ciphertext│                    │
│    └──────────┘      Key        └──────────┘                    │
│                                                                  │
│    Examples: AES-256, ChaCha20, 3DES (legacy)                   │
│                                                                  │
│    Pros: FAST (suitable for large data)                         │
│    Cons: How do you share the key securely?                     │
└─────────────────────────────────────────────────────────────────┘
```

**Speed Comparison**:
- AES can encrypt gigabytes per second
- Used for: File encryption, VPNs, HTTPS data

### 5.2 Asymmetric Cryptography

**Concept**: Two mathematically related keys - Public and Private.

```
┌─────────────────────────────────────────────────────────────────┐
│                  ASYMMETRIC ENCRYPTION                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│    Public Key (share freely)    Private Key (keep SECRET!)      │
│          │                              │                        │
│          v                              v                        │
│    ┌──────────┐                  ┌──────────┐                   │
│    │ Encrypt  │                  │ Decrypt  │                   │
│    │   with   │                  │   with   │                   │
│    │ Public   │                  │ Private  │                   │
│    └──────────┘                  └──────────┘                   │
│                                                                  │
│    Anyone can encrypt ──────> Only owner can decrypt            │
│                                                                  │
│    Examples: RSA, ECC (ECDSA, ECDH)                             │
│                                                                  │
│    Pros: Solves key distribution problem!                       │
│    Cons: SLOW (1000x slower than symmetric)                     │
└─────────────────────────────────────────────────────────────────┘
```

**The Brilliant Solution**:
- Publish your Public Key everywhere
- Anyone can encrypt messages to you
- Only you (with Private Key) can read them
- No need to secretly share keys beforehand!

### 5.3 Hybrid Encryption

**Real-world solution**: Use BOTH!

```
┌─────────────────────────────────────────────────────────────────┐
│                    HYBRID ENCRYPTION                             │
│                  (How HTTPS Actually Works)                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Step 1: Asymmetric exchange                                     │
│  ─────────────────────────────                                   │
│  Client ──[RSA encrypt session key]──> Server                   │
│                    │                                             │
│          Random AES Key (256-bit)                               │
│                    │                                             │
│  Step 2: Symmetric data transfer                                │
│  ────────────────────────────────                                │
│  Client <──[AES encrypted data]──> Server                       │
│                    │                                             │
│            Fast bulk encryption                                  │
│                                                                  │
│  Result: Best of both worlds!                                   │
│  - Asymmetric solves key exchange                               │
│  - Symmetric provides speed                                     │
└─────────────────────────────────────────────────────────────────┘
```

### 5.4 Comparison Table

| Property | Symmetric | Asymmetric |
|----------|-----------|------------|
| **Keys** | 1 shared key | 2 keys (public + private) |
| **Speed** | Very fast | 1000x slower |
| **Key Length** | 128-256 bits | 2048-4096 bits (RSA) |
| **Key Distribution** | Difficult | Easy (publish public) |
| **Use Case** | Bulk data | Key exchange, signatures |
| **Examples** | AES, ChaCha20 | RSA, ECC |

---

## 6. Keys: The Heart of Cryptography

### 6.1 What is a Key?

A cryptographic key is a piece of data (usually a large number) that controls:
- How plaintext is transformed into ciphertext
- How ciphertext is transformed back to plaintext

```
Same algorithm + Different keys = Different ciphertext

AES("Hello", Key1) = "X7$kL"
AES("Hello", Key2) = "9mN@p"  ← Completely different!
```

### 6.2 Key Sizes and Security

```
+------------------------------------------------------------------+
|                    KEY SIZE SECURITY                              |
+------------------------------------------------------------------+

Key Size    Possible Keys    Time to Brute Force
──────────  ──────────────   ────────────────────
40-bit      ~1 trillion      Seconds (broken)
56-bit      ~72 quadrillion  Days (DES is broken)
128-bit     3.4 × 10^38      Billions of years
256-bit     1.1 × 10^77      Until heat death of universe

Rule of Thumb:
- 128-bit: Secure against classical computers
- 256-bit: Secure against quantum computers (symmetric)
- RSA: 2048+ bits (asymmetric needs larger keys)
- ECC: 256+ bits (equivalent to RSA-3072)
```

### 6.3 Key Generation

Keys MUST be generated from **cryptographically secure** random sources:

```python
# WRONG - Predictable!
import random
key = random.randint(0, 2**256)  # Uses time-based seed, predictable!

# CORRECT - Cryptographically secure
import secrets
key = secrets.token_bytes(32)  # 256-bit truly random key

# Or using OS-level randomness
import os
key = os.urandom(32)
```

### 6.4 Key Management

```
+------------------------------------------------------------------+
|                    KEY MANAGEMENT LIFECYCLE                       |
+------------------------------------------------------------------+

Generation ──> Distribution ──> Storage ──> Usage ──> Rotation ──> Destruction
    │               │             │          │           │              │
    │               │             │          │           │              │
    v               v             v          v           v              v
 Secure RNG    Secure channel   HSM/KMS   Minimal     Regular      Secure
 Full entropy  Out-of-band      Encrypted  exposure   schedule     deletion
               TLS/PKI          at rest                            Zero memory
```

**Key Management Best Practices**:
1. Never hardcode keys in source code
2. Use environment variables or secrets managers
3. Rotate keys regularly
4. Use different keys for different purposes
5. Destroy keys securely when expired

---

## 7. Cryptographic Primitives

Primitives are the building blocks of cryptographic systems.

### 7.1 Block Ciphers

Encrypt fixed-size blocks (typically 128 bits):

```
AES Block Cipher:
┌────────────────┐      ┌────────────────┐
│  128-bit block │──────│   AES Core     │──────│  128-bit block │
│   (plaintext)  │  +   │  (algorithm)   │      │  (ciphertext)  │
└────────────────┘ Key  └────────────────┘      └────────────────┘

To encrypt more than 128 bits, use a MODE OF OPERATION:
- ECB (bad - patterns visible)
- CBC (better - chains blocks)
- GCM (best - authenticated encryption)
```

### 7.2 Stream Ciphers

Encrypt one bit/byte at a time:

```
Stream Cipher:
┌────────────────┐
│  Key + Nonce   │
└───────┬────────┘
        │
        v
┌────────────────┐
│   Keystream    │ ──> XOR with plaintext
│   Generator    │
└────────────────┘

Plaintext:  10110010
Keystream:  11001101
            ────────
Ciphertext: 01111111
```

Examples: ChaCha20, RC4 (broken), Salsa20

### 7.3 Hash Functions

One-way compression functions:

```
Hash Function:
┌────────────────────────────────┐
│ Input: Any size                │
│ "Hello World" (11 bytes)       │
│ "Lorem ipsum..." (1 MB)        │
└───────────────┬────────────────┘
                │
                v
        ┌───────────────┐
        │ Hash Function │
        │  (SHA-256)    │
        └───────┬───────┘
                │
                v
┌────────────────────────────────┐
│ Output: Fixed size (256 bits)  │
│ 64 hex characters              │
└────────────────────────────────┘
```

### 7.4 Message Authentication Codes (MAC)

Hash + Secret Key = Verified integrity:

```
HMAC (Hash-based MAC):
┌──────────────────────────────────────────┐
│ Message: "Transfer $100"                 │
│ Secret Key: "shared_secret"              │
└─────────────────┬────────────────────────┘
                  │
                  v
          ┌───────────────┐
          │     HMAC      │
          │  (with key)   │
          └───────┬───────┘
                  │
                  v
┌──────────────────────────────────────────┐
│ Tag: "a1b2c3d4..."                       │
│ Only someone with the key can verify!    │
└──────────────────────────────────────────┘
```

### 7.5 Digital Signatures

Prove authenticity and integrity:

```
Digital Signature:
┌──────────────────────────────────────────────────────────────────┐
│                                                                   │
│  Signing (with Private Key):                                      │
│  ───────────────────────────                                      │
│  Document ──> Hash ──> Encrypt with Private Key ──> Signature    │
│                                                                   │
│  Verification (with Public Key):                                  │
│  ──────────────────────────────                                   │
│  Document ──> Hash ──> Compare with decrypted Signature          │
│                                                                   │
│  If they match: Document is authentic and unmodified             │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

---

## 8. Common Algorithms Overview

### 8.1 Algorithm Categories

```
+------------------------------------------------------------------+
|                    ALGORITHM OVERVIEW                             |
+------------------------------------------------------------------+

SYMMETRIC ENCRYPTION:
┌─────────────────────────────────────────────────────────────────┐
│ Algorithm  │ Key Size    │ Status      │ Use Case              │
├────────────┼─────────────┼─────────────┼───────────────────────┤
│ AES        │ 128/192/256 │ Standard    │ Everything            │
│ ChaCha20   │ 256         │ Standard    │ Mobile, TLS           │
│ 3DES       │ 168         │ Deprecated  │ Legacy only           │
│ DES        │ 56          │ BROKEN      │ Never use             │
│ Blowfish   │ 32-448      │ Deprecated  │ bcrypt (hashing)      │
└─────────────────────────────────────────────────────────────────┘

ASYMMETRIC ENCRYPTION:
┌─────────────────────────────────────────────────────────────────┐
│ Algorithm  │ Key Size    │ Status      │ Use Case              │
├────────────┼─────────────┼─────────────┼───────────────────────┤
│ RSA        │ 2048+       │ Standard    │ Signatures, key ex.   │
│ ECDSA      │ 256+        │ Standard    │ Signatures            │
│ ECDH       │ 256+        │ Standard    │ Key exchange          │
│ Ed25519    │ 256         │ Modern      │ Signatures, SSH       │
│ X25519     │ 256         │ Modern      │ Key exchange          │
└─────────────────────────────────────────────────────────────────┘

HASH FUNCTIONS:
┌─────────────────────────────────────────────────────────────────┐
│ Algorithm  │ Output Size │ Status      │ Use Case              │
├────────────┼─────────────┼─────────────┼───────────────────────┤
│ MD5        │ 128-bit     │ BROKEN      │ Checksums only        │
│ SHA-1      │ 160-bit     │ Deprecated  │ Legacy git            │
│ SHA-256    │ 256-bit     │ Standard    │ General purpose       │
│ SHA-3      │ 256/512-bit │ Standard    │ High security         │
│ bcrypt     │ Variable    │ Standard    │ Passwords             │
│ Argon2     │ Variable    │ Modern      │ Passwords             │
└─────────────────────────────────────────────────────────────────┘
```

### 8.2 When to Use What

```python
# SYMMETRIC - For encrypting data
from cryptography.hazmat.primitives.ciphers.aead import AESGCM
key = AESGCM.generate_key(bit_length=256)
aesgcm = AESGCM(key)
ciphertext = aesgcm.encrypt(nonce, plaintext, associated_data)

# ASYMMETRIC - For key exchange
from cryptography.hazmat.primitives.asymmetric import x25519
private_key = x25519.X25519PrivateKey.generate()
public_key = private_key.public_key()
shared_key = private_key.exchange(peer_public_key)

# HASHING - For integrity
import hashlib
hash_value = hashlib.sha256(data).hexdigest()

# PASSWORD HASHING - For storing passwords
import bcrypt
hashed = bcrypt.hashpw(password.encode(), bcrypt.gensalt())
```

---

## 9. Real-World Applications

### 9.1 HTTPS (Web Security)

```
┌──────────────────────────────────────────────────────────────────┐
│                    HTTPS BREAKDOWN                                │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│  Your Browser                              Web Server             │
│       │                                         │                 │
│       │──────── ClientHello (ciphers) ────────>│                 │
│       │                                         │                 │
│       │<─────── ServerHello + Certificate ─────│                 │
│       │         (contains public key)          │                 │
│       │                                         │                 │
│       │──────── Key Exchange (ECDHE) ─────────>│                 │
│       │         (generate shared secret)        │                 │
│       │                                         │                 │
│       │<════════ Encrypted Data (AES) ════════>│                 │
│       │         (using shared secret)          │                 │
│                                                                   │
│  Crypto Used:                                                     │
│  - RSA/ECDSA: Server authentication (certificate)               │
│  - ECDHE: Key exchange (forward secrecy)                         │
│  - AES-GCM: Bulk data encryption                                 │
│  - SHA-256: Integrity of handshake                               │
└──────────────────────────────────────────────────────────────────┘
```

### 9.2 Password Storage

```
┌──────────────────────────────────────────────────────────────────┐
│                    PASSWORD STORAGE                               │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│  WRONG WAY:                                                       │
│  ──────────                                                       │
│  Database: | username | password    |                            │
│            | alice    | hunter2     |  ← Plaintext! Disaster!    │
│                                                                   │
│  STILL WRONG:                                                     │
│  ────────────                                                     │
│  Database: | username | password_hash                |            │
│            | alice    | ab56b4d...  |  ← Unsalted, rainbow-able │
│                                                                   │
│  CORRECT WAY:                                                     │
│  ────────────                                                     │
│  Database: | username | password_hash                            │
│            | alice    | $2b$12$LQv3... | ← bcrypt with salt       │
│                                                                   │
│  bcrypt includes:                                                 │
│  - Unique salt per password                                       │
│  - Work factor (makes brute force slow)                          │
│  - Algorithm identifier                                           │
└──────────────────────────────────────────────────────────────────┘
```

### 9.3 File Encryption

```
Common Use Cases:
- BitLocker (Windows) - Full disk encryption
- FileVault (Mac) - Full disk encryption
- LUKS (Linux) - Full disk encryption
- Age/GPG - File encryption

Architecture:
┌──────────────────────────────────────────────────────────────────┐
│                                                                   │
│  User Password ─────────────────┐                                │
│                                 v                                 │
│                         ┌──────────────┐                         │
│                         │     PBKDF2   │                         │
│                         │   (or Argon2)│                         │
│                         └──────┬───────┘                         │
│                                │                                  │
│                                v                                  │
│                    ┌─────────────────────┐                       │
│                    │ Key Encryption Key  │                       │
│                    └──────────┬──────────┘                       │
│                               │ Decrypts                         │
│                               v                                  │
│                    ┌─────────────────────┐                       │
│                    │   Master Key (AES)  │                       │
│                    └──────────┬──────────┘                       │
│                               │ Encrypts                         │
│                               v                                  │
│                    ┌─────────────────────┐                       │
│                    │   Disk Sectors      │                       │
│                    └─────────────────────┘                       │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

---

## 10. The Attacker's Perspective

### 10.1 Cryptographic Attacks Overview

```
+------------------------------------------------------------------+
|                    ATTACK CATEGORIES                              |
+------------------------------------------------------------------+

1. BRUTE FORCE
   Try all possible keys until one works
   Defense: Use sufficiently large key sizes

2. CRYPTANALYSIS
   Find mathematical weaknesses in algorithm
   Defense: Use well-vetted, modern algorithms

3. SIDE CHANNEL
   Attack the implementation, not the math
   - Timing attacks
   - Power analysis
   - Electromagnetic emanations
   Defense: Constant-time implementations

4. IMPLEMENTATION FLAWS
   - Weak random number generation
   - IV/Nonce reuse
   - Padding oracles
   Defense: Use high-level crypto libraries

5. KEY THEFT
   Don't break the crypto, steal the key!
   - Memory dumps
   - Config files
   - Insider threats
   Defense: Key management, HSMs
```

### 10.2 Common Attack Scenarios

```python
# Scenario 1: Predictable Random
import random
random.seed(12345)  # Attacker knows the seed
key = random.getrandbits(256)  # Predictable key!

# Scenario 2: IV Reuse
# Encrypting two messages with same IV
# C1 = P1 XOR Keystream
# C2 = P2 XOR Keystream
# C1 XOR C2 = P1 XOR P2 (key eliminated!)

# Scenario 3: ECB Mode Pattern
# Same plaintext block = Same ciphertext block
# Attacker can see patterns in encrypted data!
```

---

## 11. Kerckhoffs's Principle

### 11.1 The Principle

> **"A cryptosystem should be secure even if everything about the system, except the key, is public knowledge."**

In other words:
- The algorithm should be public
- Security comes from the KEY, not secrecy of the algorithm

### 11.2 Why This Matters

```
+------------------------------------------------------------------+
|                    SECURITY BY OBSCURITY                          |
+------------------------------------------------------------------+

BAD APPROACH (Security through obscurity):
"My encryption is secure because nobody knows my algorithm"
Problem: Once algorithm is leaked, everything is compromised

GOOD APPROACH (Kerckhoffs's Principle):
"My encryption uses AES-256 which is public, 
 but my key is secret"
Benefit: Algorithm can be publicly analyzed, only key matters

Real World Example:
- AES algorithm is public, analyzed by thousands of researchers
- If there was a flaw, it would be found
- Security relies ONLY on keeping the key secret
```

### 11.3 Practical Implications

1. **Never roll your own crypto** - Use standard algorithms
2. **Assume the attacker knows your algorithm**
3. **All security must come from the key**
4. **Audit and review are benefits of public algorithms**

---

## 12. Entropy and Randomness

### 12.1 What is Entropy?

Entropy measures the unpredictability of data.

```
Low Entropy (Predictable):
"AAAAAAAAAA"     ← Very predictable
"password123"    ← Common pattern
"1234567890"     ← Sequential

High Entropy (Unpredictable):
"X7$kL@9mN!#pQ"  ← Random looking
0x8f2a4b7c9d1e   ← Truly random bytes
```

### 12.2 Measuring Entropy

```
Shannon Entropy Formula:
H = -Σ P(x) × log2(P(x))

Maximum entropy for n bits = n bits
(when all outcomes equally likely)

Example:
- Fair coin flip: 1 bit of entropy
- Fair 6-sided die: 2.58 bits of entropy
- 256-bit random key: 256 bits of entropy
```

### 12.3 Sources of Randomness

```
+------------------------------------------------------------------+
|                    RANDOMNESS SOURCES                             |
+------------------------------------------------------------------+

HARDWARE (True Random - Best):
- CPU timing jitter
- Thermal noise
- Radioactive decay
- Hardware Random Number Generators (HRNG)

SOFTWARE (Pseudo-Random - Acceptable for crypto):
- /dev/urandom (Linux)
- CryptGenRandom (Windows)
- secrets module (Python)

SOFTWARE (NOT for crypto!):
- rand() in C
- random.random() in Python
- Math.random() in JavaScript

These use deterministic algorithms seeded with time!
```

### 12.4 Python Example

```python
# WRONG - Not cryptographically secure
import random
key = random.randint(0, 2**256)  # Predictable if seed known!

# CORRECT - Cryptographically secure
import secrets
key = secrets.randbits(256)  # Uses OS entropy pool

# Also correct
import os
key = os.urandom(32)  # 256 bits from /dev/urandom
```

---

## 13. Common Mistakes in Cryptography

### 13.1 Top 10 Crypto Mistakes

```
+------------------------------------------------------------------+
|                    TOP 10 CRYPTO MISTAKES                         |
+------------------------------------------------------------------+

1. Using encoding instead of encryption
   BAD: base64.b64encode(password)
   FIX: Use actual encryption (AES)

2. Using broken algorithms
   BAD: MD5, DES, RC4
   FIX: SHA-256, AES, ChaCha20

3. Hardcoding keys in source code
   BAD: key = "supersecretkey"
   FIX: Environment variables, secrets manager

4. Using weak/predictable random
   BAD: random.random()
   FIX: secrets.token_bytes()

5. Not using salt for password hashing
   BAD: SHA256(password)
   FIX: bcrypt(password, salt)

6. Reusing IVs/nonces
   BAD: Always using IV = 0
   FIX: Random IV for each encryption

7. Using ECB mode
   BAD: AES.new(key, AES.MODE_ECB)
   FIX: AES.new(key, AES.MODE_GCM)

8. Not authenticating ciphertext
   BAD: AES-CBC without HMAC
   FIX: AES-GCM (authenticated encryption)

9. Comparing hashes with ==
   BAD: if user_hash == stored_hash:
   FIX: if hmac.compare_digest(user_hash, stored_hash):

10. Rolling your own crypto
    BAD: def my_encrypt(data): ...
    FIX: Use established libraries (cryptography, libsodium)
```

---

## 14. Tools for Crypto Analysis

### 14.1 Essential Tools

| Tool | Purpose | Usage |
|------|---------|-------|
| **CyberChef** | Multi-purpose decoder | Web-based, drag-and-drop |
| **OpenSSL** | General crypto operations | Command line |
| **Hashcat** | Password cracking | GPU-accelerated |
| **John the Ripper** | Password cracking | Format versatile |
| **Wireshark** | Network crypto analysis | TLS inspection |
| **xxd/hexdump** | Binary analysis | Hex viewing |

### 14.2 CyberChef Examples

```
Common Recipes:

Decode Base64:
Input: SGVsbG8gV29ybGQ=
Operation: From Base64
Output: Hello World

Identify Hash:
Input: 5d41402abc4b2a76b9719d911017c592
Operation: Analyze hash
Output: Likely MD5

Decrypt XOR:
Input: 0x1a0b1c0d
Operation: XOR with key
Output: Plaintext
```

### 14.3 OpenSSL Commands

```bash
# Generate AES key
openssl rand -hex 32

# Encrypt file
openssl enc -aes-256-cbc -salt -in plain.txt -out encrypted.bin

# Decrypt file
openssl enc -d -aes-256-cbc -in encrypted.bin -out decrypted.txt

# Generate RSA key pair
openssl genrsa -out private.pem 2048
openssl rsa -in private.pem -pubout -out public.pem

# View certificate
openssl x509 -in cert.pem -text -noout

# Calculate hash
openssl dgst -sha256 file.txt
```

---

## 15. How This Module Connects

### 15.1 Module Learning Path

```
┌─────────────────────────────────────────────────────────────────┐
│                 CRYPTOGRAPHY MODULE FLOW                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  YOU ARE HERE                                                    │
│       │                                                          │
│       v                                                          │
│  01_Fundamentals ──────────────────────────────────────────────┐│
│       │                                                         ││
│       v                                                         ││
│  02_Encoding ──────> Understanding data representation          ││
│       │                                                         ││
│       v                                                         ││
│  03_Hashing ───────> One-way functions, integrity               ││
│       │                                                         ││
│       v                                                         ││
│  04_Symmetric ─────> AES, modes, bulk encryption                ││
│       │                                                         ││
│       v                                                         ││
│  05_Asymmetric ────> RSA, ECC, key exchange                     ││
│       │                                                         ││
│       v                                                         ││
│  06_PKI_TLS ───────> Certificates, trust chains, HTTPS          ││
│       │                                                         ││
│       ├─────────────────────────────────────────────────────────┘│
│       │                                                          │
│       v                                                          │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ APPLIED TOPICS:                                              ││
│  │ 07_Password_Cracking - Breaking hashes                       ││
│  │ 08_Crypto_Attacks - Exploiting implementations               ││
│  │ 09_Encryption_at_Rest - BitLocker, LUKS                      ││
│  │ 10_SSH - Key management, pivoting                            ││
│  │ 11_Obfuscation - Encoding for evasion                        ││
│  │ 12_Steganography - Hidden data                               ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 15.2 Connection to Other Modules

```
07_Cryptography connects to:

┌─────────────────────────────────────────────────────────────────┐
│ 03_Linux          │ Password files, /etc/shadow, OpenSSL       │
│ 04_Windows_AD     │ NTLM hashes, Kerberos, certificates       │
│ 05_Web_Security   │ TLS, JWT, session tokens                  │
│ 06_AD_Attacks     │ Kerberoasting, Pass-the-Hash              │
│ 08_Tooling        │ Hashcat, John, CyberChef                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 16. Hands-On Labs

### Lab 1: Identify the Crypto

```
Challenge: What type of crypto is each of these?

1. "SGVsbG8gV29ybGQ="
2. "5d41402abc4b2a76b9719d911017c592"
3. "U2FsdGVkX1+..."
4. "-----BEGIN RSA PRIVATE KEY-----"
5. "%48%65%6C%6C%6F"

Answers:
1. Base64 encoding (ends with =, A-Za-z0-9+/)
2. MD5 hash (32 hex chars)
3. OpenSSL encrypted data (Salted__ prefix in Base64)
4. RSA private key (PEM format)
5. URL encoding (%XX pattern)
```

### Lab 2: Basic Encryption with Python

```python
"""
Lab: Encrypt and decrypt a message using Fernet (symmetric)
"""
from cryptography.fernet import Fernet

# Step 1: Generate a key
key = Fernet.generate_key()
print(f"Key: {key.decode()}")

# Step 2: Create cipher
cipher = Fernet(key)

# Step 3: Encrypt
message = b"Secret message for OSCP preparation!"
encrypted = cipher.encrypt(message)
print(f"Encrypted: {encrypted}")

# Step 4: Decrypt
decrypted = cipher.decrypt(encrypted)
print(f"Decrypted: {decrypted.decode()}")

# Challenge: What happens if you use wrong key?
wrong_key = Fernet.generate_key()
wrong_cipher = Fernet(wrong_key)
# wrong_cipher.decrypt(encrypted)  # Raises InvalidToken!
```

### Lab 3: Hash Comparison

```python
"""
Lab: Compare different hash algorithms
"""
import hashlib

message = b"password123"

algorithms = ['md5', 'sha1', 'sha256', 'sha512']

for algo in algorithms:
    hasher = hashlib.new(algo)
    hasher.update(message)
    print(f"{algo.upper():8} ({len(hasher.hexdigest()):3} chars): {hasher.hexdigest()}")

# Output:
# MD5      ( 32 chars): 482c811da5d5b4bc6d497ffa98491e38
# SHA1     ( 40 chars): cbfdac6008f9cab4083784cbd1874f76618d2a97
# SHA256   ( 64 chars): ef92b778ba...
# SHA512   (128 chars): b109f3bbbc...
```

---

## 17. Interview Questions

### Beginner Questions

**Q1: What is the difference between encoding and encryption?**
> **A**: Encoding transforms data format (like Base64) and can be reversed by anyone - it provides no security. Encryption uses a key to scramble data so only key holders can read it - it provides confidentiality.

**Q2: Why shouldn't you use MD5 for password hashing?**
> **A**: MD5 is broken - collisions can be generated quickly. It's also too fast, making brute force attacks easy. Use bcrypt or Argon2 instead, which are slow by design and include salts.

**Q3: What is the CIA triad?**
> **A**: Confidentiality (only authorized access), Integrity (data not modified), Authentication (verify identity). These are the three core security properties that cryptography protects.

### Intermediate Questions

**Q4: Explain the difference between symmetric and asymmetric encryption.**
> **A**: Symmetric uses one shared key for both encryption and decryption (fast, used for bulk data). Asymmetric uses a key pair - public key encrypts, private key decrypts (slow, used for key exchange and signatures). Real systems use hybrid approach.

**Q5: What is a digital signature and how does it work?**
> **A**: A digital signature proves authenticity and integrity. The signer hashes the document and encrypts the hash with their private key. Anyone can verify by decrypting with the public key and comparing to their own hash calculation.

**Q6: What is forward secrecy?**
> **A**: Forward secrecy means if a long-term private key is compromised, past encrypted sessions remain secure. Achieved by using ephemeral (temporary) keys for each session (like ECDHE in TLS).

### Advanced Questions

**Q7: Explain a padding oracle attack.**
> **A**: In CBC mode, if a server reveals whether padding is valid (through error messages or timing), an attacker can decrypt ciphertext byte by byte by manipulating the previous block and observing the response. Mitigated by using authenticated encryption (GCM).

**Q8: Why is nonce reuse dangerous in stream ciphers?**
> **A**: If the same nonce and key are used twice: C1 = P1 XOR K, C2 = P2 XOR K. Then C1 XOR C2 = P1 XOR P2, eliminating the key. With known plaintext (like HTTP headers), the attacker can recover everything.

---

## 18. Quick Reference

### Algorithm Recommendations (2024+)

| Use Case | Recommended | Avoid |
|----------|-------------|-------|
| Symmetric encryption | AES-256-GCM, ChaCha20-Poly1305 | DES, 3DES, AES-ECB |
| Asymmetric encryption | RSA-2048+, X25519 | RSA-1024, plain RSA |
| Signatures | Ed25519, ECDSA P-256 | RSA-1024, DSA |
| Password hashing | Argon2id, bcrypt | MD5, SHA-1, plain SHA-256 |
| General hashing | SHA-256, SHA-3 | MD5, SHA-1 |
| Key exchange | X25519, ECDHE | Static RSA, DH-1024 |

### Quick Commands

```bash
# OpenSSL - Generate random key
openssl rand -hex 32

# Hash a file
sha256sum file.txt

# Encrypt with password
openssl enc -aes-256-gcm -pbkdf2 -in plain.txt -out encrypted.bin

# Base64 decode
echo "SGVsbG8=" | base64 -d

# Hex decode
echo "48656c6c6f" | xxd -r -p
```

### Key Sizes

```
Symmetric:     128-bit minimum, 256-bit recommended
RSA:           2048-bit minimum, 4096-bit high security
ECC:           256-bit minimum (equivalent to RSA-3072)
Hash output:   256-bit minimum (SHA-256)
Password hash: bcrypt cost 12+, Argon2 appropriate parameters
```

---

## Next Steps

Now that you understand the fundamentals, proceed to:

1. **[[02_Encoding_Fundamentals]]** - Deep dive into Base64, Hex, URL encoding
2. **[[03_Hashing_and_Integrity]]** - Hashing algorithms and use cases
3. **[[04_Symmetric_Encryption]]** - AES, modes, and practical encryption

---

## References

- [Cryptography Engineering - Ferguson, Schneier, Kohno](https://www.schneier.com/books/cryptography-engineering/)
- [Serious Cryptography - Jean-Philippe Aumasson](https://nostarch.com/seriouscrypto)
- [NIST Cryptographic Standards](https://csrc.nist.gov/publications/fips)
- [CyberChef](https://gchq.github.io/CyberChef/)
