# 04 - Symmetric Encryption: AES and Block Ciphers

---
title: "Symmetric Encryption: AES, Modes, and Stream Ciphers"
parent: "[[07_Cryptography]]"
tags:
  - "encryption"
  - "symmetric"
  - "aes"
  - "block-cipher"
  - "stream-cipher"
created: 2026-02-07
updated: 2026-02-07
difficulty: "Intermediate"
---

> **Executive Summary**: Symmetric encryption uses the SAME key for both encryption and decryption. It's the workhorse of modern cryptography - fast enough to encrypt gigabytes of data in seconds. This guide covers everything from basic concepts through AES internals, modes of operation (ECB, CBC, CTR, GCM), stream ciphers (ChaCha20), and practical implementation. Understanding symmetric encryption is essential for HTTPS, VPNs, file encryption, and any security work.

**What You'll Learn**:
- How symmetric encryption works
- Block ciphers vs stream ciphers
- AES (Advanced Encryption Standard) in depth
- Modes of operation (why ECB is dangerous)
- Authenticated encryption (GCM)
- Stream ciphers (ChaCha20)
- Key management and IVs
- Practical encryption with Python and OpenSSL

---

## Table of Contents

1. [What is Symmetric Encryption? (ABSOLUTE BEGINNER)](#1-what-is-symmetric-encryption)
2. [Block Ciphers vs Stream Ciphers](#2-block-vs-stream)
3. [AES: The Standard](#3-aes)
4. [Modes of Operation](#4-modes-of-operation)
5. [ECB Mode (DANGEROUS)](#5-ecb-mode)
6. [CBC Mode](#6-cbc-mode)
7. [CTR Mode](#7-ctr-mode)
8. [GCM Mode (Recommended)](#8-gcm-mode)
9. [Stream Ciphers](#9-stream-ciphers)
10. [Keys and IVs](#10-keys-and-ivs)
11. [Padding](#11-padding)
12. [Practical Implementation](#12-practical-implementation)
13. [Common Attacks](#13-common-attacks)
14. [Real-World Applications](#14-real-world-applications)
15. [Hands-On Labs](#15-hands-on-labs)
16. [Interview Questions](#16-interview-questions)
17. [Quick Reference](#17-quick-reference)

---

## 1. What is Symmetric Encryption? (ABSOLUTE BEGINNER)

### 1.1 The Concept

**Symmetric** means "same on both sides." In symmetric encryption, the SAME KEY is used to lock (encrypt) and unlock (decrypt) data.

```
+------------------------------------------------------------------+
|                    SYMMETRIC ENCRYPTION                           |
+------------------------------------------------------------------+
|                                                                   |
|                        SAME KEY                                   |
|                           │                                       |
|           ┌───────────────┼───────────────┐                      |
|           │               │               │                      |
|           ▼               │               ▼                      |
|   ┌──────────────┐        │       ┌──────────────┐               |
|   │   ENCRYPT    │        │       │   DECRYPT    │               |
|   └──────┬───────┘        │       └──────┬───────┘               |
|          │                │              │                        |
|   "Hello" ────────> "X7$kL@9" ────────> "Hello"                  |
|   (Plaintext)      (Ciphertext)       (Plaintext)                |
|                                                                   |
+------------------------------------------------------------------+
```

### 1.2 Real-World Analogy

Think of a **lockbox with a physical key**:
- You lock valuable documents in the box with your key
- Anyone with the SAME key can unlock it
- Without the key, the box is secure

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│    You (Key A)                      Friend (Key A - copy)       │
│         │                                  │                     │
│         ▼                                  ▼                     │
│    ┌─────────┐                        ┌─────────┐               │
│    │  LOCK   │                        │ UNLOCK  │               │
│    │  BOX    │  ═══════════════════>  │  BOX    │               │
│    └─────────┘                        └─────────┘               │
│                                                                  │
│    Challenge: How do you securely share the key?                │
│    (This is why asymmetric crypto exists - later chapter!)      │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 1.3 Why Symmetric Encryption?

**Speed!** Symmetric encryption is 1000x faster than asymmetric.

```
Speed Comparison (approximate):
──────────────────────────────────────────────
AES-256:        1-10 GB/second
RSA-2048:       1-10 MB/second
Ratio:          ~1000x faster!

Use Cases:
- HTTPS data transfer (after key exchange)
- VPN tunnels
- File/disk encryption
- Database encryption
```

### 1.4 The Key Distribution Problem

```
+------------------------------------------------------------------+
|                THE KEY DISTRIBUTION PROBLEM                       |
+------------------------------------------------------------------+

Scenario: Alice wants to send encrypted message to Bob

Problem:
1. Alice encrypts with Key K
2. Alice needs to give Key K to Bob
3. If she sends K over internet, Eve can intercept it!
4. Now Eve has the key and can decrypt everything!

Solutions:
1. Meet in person (impractical for internet)
2. Use a trusted courier (still risky)
3. Use ASYMMETRIC crypto to exchange the symmetric key!
   (This is how TLS/HTTPS works - covered in PKI chapter)

+------------------------------------------------------------------+
```

---

## 2. Block Ciphers vs Stream Ciphers

### 2.1 Block Ciphers

Process data in fixed-size **blocks** (e.g., 128 bits for AES).

```
+------------------------------------------------------------------+
|                    BLOCK CIPHER                                   |
+------------------------------------------------------------------+

Input: "Hello World!!!!!" (16 bytes = 128 bits)
                │
                ▼
        ┌──────────────┐
        │  AES Block   │ ← Key
        │   Cipher     │
        └──────┬───────┘
               │
               ▼
Output: [encrypted 16 bytes]

If input > 16 bytes: Split into blocks, use MODE OF OPERATION
If input < 16 bytes: Add PADDING to fill the block
```

**Examples**: AES, DES, 3DES, Blowfish

### 2.2 Stream Ciphers

Process data one bit/byte at a time, generating a **keystream**.

```
+------------------------------------------------------------------+
|                    STREAM CIPHER                                  |
+------------------------------------------------------------------+

Key + Nonce ──> ┌──────────────┐
                │  Keystream   │
                │  Generator   │
                └──────┬───────┘
                       │
                       ▼
               Keystream: 01101001 01110010 ...
                              ⊕ (XOR)
              Plaintext: 01001000 01100101 ...
                              │
                              ▼
             Ciphertext: 00100001 00010111 ...

Encryption = Decryption (XOR is reversible!)
```

**Examples**: ChaCha20, RC4 (broken), Salsa20

### 2.3 Comparison

| Property | Block Cipher | Stream Cipher |
|----------|--------------|---------------|
| **Processing** | Fixed blocks | Bit/byte at a time |
| **Padding** | Required | Not needed |
| **Random access** | Mode dependent | Easy (CTR mode) |
| **Speed** | Fast | Very fast |
| **Examples** | AES, 3DES | ChaCha20, RC4 |

---

## 3. AES: The Standard

### 3.1 AES Overview

**AES** (Advanced Encryption Standard) is THE symmetric cipher used everywhere.

```
History:
- 1997: NIST calls for DES replacement
- 2001: Rijndael selected as AES
- Today: Used in HTTPS, VPNs, disk encryption, everywhere!

Specifications:
- Block size: 128 bits (16 bytes) - ALWAYS
- Key sizes: 128, 192, or 256 bits
- Rounds: 10 (AES-128), 12 (AES-192), 14 (AES-256)
- Structure: Substitution-Permutation Network (SPN)
```

### 3.2 AES Internals (Simplified)

```
+------------------------------------------------------------------+
|                    AES ROUND STRUCTURE                            |
+------------------------------------------------------------------+

Input: 16-byte block arranged as 4×4 matrix

For each round (10/12/14 rounds):
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. SubBytes: Substitute each byte using S-box (confusion)      │
│     ┌───┬───┬───┬───┐      ┌───┬───┬───┬───┐                   │
│     │ a │ b │ c │ d │  ──> │ S │ S │ S │ S │ (lookup table)    │
│     │ e │ f │ g │ h │  ──> │ S │ S │ S │ S │                    │
│     │ i │ j │ k │ l │  ──> │ S │ S │ S │ S │                    │
│     │ m │ n │ o │ p │  ──> │ S │ S │ S │ S │                    │
│     └───┴───┴───┴───┘      └───┴───┴───┴───┘                   │
│                                                                  │
│  2. ShiftRows: Rotate rows left (diffusion)                     │
│     Row 0: No shift                                              │
│     Row 1: Shift left 1                                          │
│     Row 2: Shift left 2                                          │
│     Row 3: Shift left 3                                          │
│                                                                  │
│  3. MixColumns: Matrix multiplication (diffusion)               │
│     Each column mixed using fixed polynomial                     │
│                                                                  │
│  4. AddRoundKey: XOR with round key (from key schedule)         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

After all rounds: Output 16-byte ciphertext block
```

### 3.3 AES Key Sizes

```
+------------------------------------------------------------------+
|                    AES KEY SIZE COMPARISON                        |
+------------------------------------------------------------------+

Key Size    Rounds    Security Level         Use Case
────────────────────────────────────────────────────────────────
AES-128     10        128 bits               General purpose
AES-192     12        192 bits               Higher security
AES-256     14        256 bits               Quantum-resistant*

*Grover's algorithm halves effective key length
 AES-256 → 128 bits vs quantum (still secure)
 AES-128 → 64 bits vs quantum (potentially breakable)

Recommendation:
- Most uses: AES-128 is sufficient
- Long-term secrets: AES-256
- Government/military: AES-256
```

### 3.4 AES Security

```
Breaking AES-128 by brute force:
- Keys to try: 2^128 = 340,282,366,920,938,463,463,374,607,431,768,211,456
- At 1 billion keys/second: 10^22 years
- All computers on Earth: Still billions of years
- More atoms in observable universe: Yes, but still impossible

Known Attacks:
- No practical attack exists
- Best known: Reduces AES-256 to 2^254.4 (theoretical, not practical)
- Side-channel attacks target IMPLEMENTATION, not algorithm
```

---

## 4. Modes of Operation

### 4.1 Why Modes Exist

AES encrypts exactly 128 bits. Real data is bigger!

```
Problem: How to encrypt a 1 GB file?

Solution: MODES OF OPERATION
- Define how to chain multiple block encryptions
- Handle data larger than block size
- Add security properties (randomization, authentication)
```

### 4.2 Mode Overview

```
+------------------------------------------------------------------+
|                    MODES OF OPERATION                             |
+------------------------------------------------------------------+

Mode    Type           Authentication    Parallelizable    Status
────────────────────────────────────────────────────────────────────
ECB     Block          No                Yes               BROKEN
CBC     Block          No                Decrypt only      Legacy
CTR     Stream-like    No                Yes               Good
GCM     Stream+Auth    YES               Yes               BEST
CCM     Block+Auth     YES               No                Good

Recommendation: Always use GCM (or CCM) for new applications
```

---

## 5. ECB Mode (DANGEROUS!)

### 5.1 How ECB Works

**ECB** (Electronic Codebook) encrypts each block independently.

```
ECB Mode:
┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐
│ Block 1 │    │ Block 2 │    │ Block 3 │    │ Block 4 │
└────┬────┘    └────┬────┘    └────┬────┘    └────┬────┘
     │              │              │              │
     ▼              ▼              ▼              ▼
  ┌─────┐       ┌─────┐       ┌─────┐       ┌─────┐
  │ AES │ Key   │ AES │ Key   │ AES │ Key   │ AES │ Key
  └──┬──┘       └──┬──┘       └──┬──┘       └──┬──┘
     │              │              │              │
     ▼              ▼              ▼              ▼
┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐
│Cipher 1 │    │Cipher 2 │    │Cipher 3 │    │Cipher 4 │
└─────────┘    └─────────┘    └─────────┘    └─────────┘
```

### 5.2 Why ECB is Broken

**Same plaintext block = Same ciphertext block!**

```
+------------------------------------------------------------------+
|                    THE ECB PENGUIN                                |
+------------------------------------------------------------------+

Original Image:              ECB Encrypted:
┌───────────────┐            ┌───────────────┐
│    🐧🐧🐧     │            │    ???🔒???   │
│   🐧🐧🐧🐧    │    ────>   │   🔒🔒🔒🔒    │ ← Pattern visible!
│    🐧🐧🐧     │            │    🔒🔒🔒     │
│  Background   │            │  Same blocks  │
└───────────────┘            └───────────────┘

The penguin is still visible in the encrypted image!
Each pixel block encrypts to same ciphertext.

This is because identical plaintext blocks produce
identical ciphertext blocks in ECB mode.
```

### 5.3 Never Use ECB

```python
# NEVER DO THIS!
from Crypto.Cipher import AES

key = b'Sixteen byte key'
cipher = AES.new(key, AES.MODE_ECB)  # DANGEROUS!
ciphertext = cipher.encrypt(plaintext)

# Problems:
# 1. Patterns leak through
# 2. Blocks can be reordered
# 3. No IV means identical plaintexts = identical ciphertexts
```

---

## 6. CBC Mode

### 6.1 How CBC Works

**CBC** (Cipher Block Chaining) XORs each block with previous ciphertext.

```
CBC Encryption:
          IV            Ciphertext 1      Ciphertext 2
          │                   │                 │
          ▼                   ▼                 ▼
Block 1 ─>⊕──> AES ──> C1 ──>⊕──> AES ──> C2 ──>⊕──> AES ──> C3
          │    │              │    │              │    │
          └────┘              └────┘              └────┘
              Key                Key                Key

IV = Initialization Vector (random, sent with ciphertext)

Key Properties:
- Each block depends on ALL previous blocks
- Same plaintext = different ciphertext (due to IV)
- Encryption is sequential (not parallelizable)
- Decryption CAN be parallelized
```

### 6.2 CBC Decryption

```
CBC Decryption:
          IV            Ciphertext 1      Ciphertext 2
          │                   │                 │
          │                   │                 │
          │    ┌─────┐        │    ┌─────┐      │    ┌─────┐
          │    │ AES │ Key    │    │ AES │ Key  │    │ AES │ Key
C1 ───────┼───>│ DEC │        C2 ─>│ DEC │      C3 ─>│ DEC │
          │    └──┬──┘        │    └──┬──┘      │    └──┬──┘
          │       │           │       │         │       │
          ▼       ▼           ▼       ▼         ▼       ▼
          ⊕<──────┘           ⊕<──────┘         ⊕<──────┘
          │                   │                 │
          ▼                   ▼                 ▼
        Plain 1            Plain 2           Plain 3
```

### 6.3 CBC in Python

```python
from Crypto.Cipher import AES
from Crypto.Random import get_random_bytes
from Crypto.Util.Padding import pad, unpad

# Key and IV
key = get_random_bytes(32)  # AES-256
iv = get_random_bytes(16)   # Block size

# Encryption
plaintext = b"Secret message that needs padding!"
cipher = AES.new(key, AES.MODE_CBC, iv)
ciphertext = cipher.encrypt(pad(plaintext, AES.block_size))

print(f"IV: {iv.hex()}")
print(f"Ciphertext: {ciphertext.hex()}")

# Decryption
cipher = AES.new(key, AES.MODE_CBC, iv)
decrypted = unpad(cipher.decrypt(ciphertext), AES.block_size)
print(f"Decrypted: {decrypted}")
```

### 6.4 CBC Vulnerabilities

```
1. Padding Oracle Attack
   - If server reveals whether padding is valid
   - Can decrypt entire ciphertext byte by byte!
   - POODLE attack on SSL 3.0

2. Bit Flipping Attack
   - Modify ciphertext byte → Predictable plaintext change
   - Can change "user=alice" to "user=admin"

3. IV Reuse
   - Leaks information about plaintext relationship

Defense: Use GCM mode instead (provides authentication)
```

---

## 7. CTR Mode

### 7.1 How CTR Works

**CTR** (Counter Mode) turns a block cipher into a stream cipher.

```
CTR Mode:
                    Nonce + Counter 0    Nonce + Counter 1    Nonce + Counter 2
                           │                    │                    │
                           ▼                    ▼                    ▼
                       ┌─────┐              ┌─────┐              ┌─────┐
                 Key ─>│ AES │        Key ─>│ AES │        Key ─>│ AES │
                       └──┬──┘              └──┬──┘              └──┬──┘
                          │                    │                    │
                          ▼                    ▼                    ▼
                      Keystream 0         Keystream 1         Keystream 2
                          │                    │                    │
Plain 0 ─────────────────>⊕                   ⊕                    ⊕
Plain 1 ────────────────────────────────────>⊕                    ⊕
Plain 2 ──────────────────────────────────────────────────────────>⊕
                          │                    │                    │
                          ▼                    ▼                    ▼
                      Cipher 0             Cipher 1             Cipher 2

Key Properties:
- Parallelizable (encrypt/decrypt)
- Random access (seek to any position)
- No padding needed
- NONCE MUST NEVER BE REUSED with same key!
```

### 7.2 CTR Nonce Danger

```
+------------------------------------------------------------------+
|                    CTR NONCE REUSE DISASTER                       |
+------------------------------------------------------------------+

If you use the same nonce twice:
C1 = P1 ⊕ Keystream
C2 = P2 ⊕ Keystream

Then:
C1 ⊕ C2 = P1 ⊕ Keystream ⊕ P2 ⊕ Keystream
        = P1 ⊕ P2  (Keystreams cancel!)

Attacker now has P1 ⊕ P2
If they know P1 (e.g., HTTP header), they can derive P2!

NEVER REUSE NONCES in CTR mode!
```

### 7.3 CTR in Python

```python
from Crypto.Cipher import AES
from Crypto.Random import get_random_bytes

key = get_random_bytes(32)
nonce = get_random_bytes(8)  # 64-bit nonce for AES-CTR

# Encrypt
cipher = AES.new(key, AES.MODE_CTR, nonce=nonce)
ciphertext = cipher.encrypt(b"Hello World!")

print(f"Nonce: {nonce.hex()}")
print(f"Ciphertext: {ciphertext.hex()}")

# Decrypt
cipher = AES.new(key, AES.MODE_CTR, nonce=nonce)
plaintext = cipher.decrypt(ciphertext)
print(f"Plaintext: {plaintext}")

# No padding needed! Works with any length
```

---

## 8. GCM Mode (Recommended)

### 8.1 What is GCM?

**GCM** (Galois/Counter Mode) combines CTR encryption with authentication.

```
GCM = CTR Mode + GMAC Authentication

Provides:
- Confidentiality (encryption)
- Integrity (detect tampering)
- Authenticity (verify sender)

This is called AEAD: Authenticated Encryption with Associated Data
```

### 8.2 How GCM Works

```
+------------------------------------------------------------------+
|                    GCM MODE STRUCTURE                             |
+------------------------------------------------------------------+

                    ┌─────────────────────────────────────────┐
                    │              ENCRYPTION                  │
                    │                                          │
 Nonce + Counter ──>│   AES-CTR (same as CTR mode)            │
 Plaintext ────────>│                              ──> Ciphertext
                    │                                          │
                    └──────────────────┬──────────────────────┘
                                       │
                    ┌──────────────────▼──────────────────────┐
                    │           AUTHENTICATION                 │
                    │                                          │
 Associated Data ──>│   GMAC (Galois MAC)                     │
 Ciphertext ───────>│                              ──> Auth Tag
                    │                                          │
                    └─────────────────────────────────────────┘

Output: Ciphertext + Auth Tag (usually 16 bytes)

Associated Data: Authenticated but NOT encrypted (headers, metadata)
```

### 8.3 Why GCM is Recommended

```
GCM Advantages:
─────────────────────────────────────────
✓ Authenticated: Detects ANY tampering
✓ Fast: Hardware acceleration (AES-NI + CLMUL)
✓ Parallelizable: Efficient on modern CPUs
✓ Standard: Used in TLS 1.2/1.3, IPsec

Comparison to CBC:
─────────────────────────────────────────
CBC:
- Padding oracle attacks possible
- Bit flipping attacks possible
- Need separate MAC (HMAC) for authentication

GCM:
- Authentication built-in
- Bit flipping detected by auth tag
- Single pass encryption + authentication
```

### 8.4 GCM in Python

```python
from Crypto.Cipher import AES
from Crypto.Random import get_random_bytes

# Encrypt with authentication
def encrypt_aes_gcm(key, plaintext, associated_data=b""):
    nonce = get_random_bytes(12)  # 96-bit nonce recommended
    cipher = AES.new(key, AES.MODE_GCM, nonce=nonce)
    
    # Authenticate additional data (not encrypted)
    cipher.update(associated_data)
    
    # Encrypt and get auth tag
    ciphertext, tag = cipher.encrypt_and_digest(plaintext)
    
    return nonce, ciphertext, tag

# Decrypt with verification
def decrypt_aes_gcm(key, nonce, ciphertext, tag, associated_data=b""):
    cipher = AES.new(key, AES.MODE_GCM, nonce=nonce)
    cipher.update(associated_data)
    
    try:
        plaintext = cipher.decrypt_and_verify(ciphertext, tag)
        return plaintext
    except ValueError:
        raise Exception("Authentication failed! Data was tampered!")

# Usage
key = get_random_bytes(32)
plaintext = b"Secret message"
associated_data = b"Header info"  # Authenticated but not encrypted

nonce, ciphertext, tag = encrypt_aes_gcm(key, plaintext, associated_data)

print(f"Nonce: {nonce.hex()}")
print(f"Ciphertext: {ciphertext.hex()}")
print(f"Auth Tag: {tag.hex()}")

decrypted = decrypt_aes_gcm(key, nonce, ciphertext, tag, associated_data)
print(f"Decrypted: {decrypted}")

# Try tampering
tampered = bytearray(ciphertext)
tampered[0] ^= 1  # Flip one bit
try:
    decrypt_aes_gcm(key, nonce, bytes(tampered), tag, associated_data)
except Exception as e:
    print(f"Tampering detected: {e}")
```

### 8.5 GCM Nonce Requirements

```
GCM Nonce Rules:
─────────────────────────────────────────
- Size: 96 bits (12 bytes) recommended
- MUST be unique for each encryption with same key
- Can be random OR counter-based
- NEVER REUSE with same key (catastrophic!)

Nonce Reuse Consequence:
- Auth tag forgery becomes possible
- XOR of plaintexts leaked
- Complete security breakdown
```

---

## 9. Stream Ciphers

### 9.1 ChaCha20

**ChaCha20** is the modern stream cipher, alternative to AES.

```
ChaCha20 Overview:
─────────────────────────────────────────
- Key size: 256 bits
- Nonce: 96 bits
- Counter: 32 bits
- Speed: Very fast (especially on mobile)
- No AES-NI needed (pure software)

Used in:
- TLS 1.3 (ChaCha20-Poly1305)
- WireGuard VPN
- SSH
```

### 9.2 ChaCha20-Poly1305

```
ChaCha20-Poly1305 = ChaCha20 + Poly1305 MAC

Similar to AES-GCM:
- ChaCha20 for encryption
- Poly1305 for authentication
- AEAD (Authenticated Encryption)

When to use:
- Mobile devices (no AES hardware acceleration)
- When you need software-only implementation
- TLS cipher suite option
```

### 9.3 ChaCha20 in Python

```python
from Crypto.Cipher import ChaCha20_Poly1305
from Crypto.Random import get_random_bytes

# Encrypt
key = get_random_bytes(32)
nonce = get_random_bytes(12)

cipher = ChaCha20_Poly1305.new(key=key, nonce=nonce)
cipher.update(b"Associated data")
ciphertext, tag = cipher.encrypt_and_digest(b"Secret message")

print(f"Ciphertext: {ciphertext.hex()}")
print(f"Tag: {tag.hex()}")

# Decrypt
cipher = ChaCha20_Poly1305.new(key=key, nonce=nonce)
cipher.update(b"Associated data")
plaintext = cipher.decrypt_and_verify(ciphertext, tag)
print(f"Plaintext: {plaintext}")
```

### 9.4 RC4 (Broken - Historical)

```
RC4: DO NOT USE!
─────────────────────────────────────────
- Created 1987
- Was used in WEP, SSL/TLS, RDP
- Multiple vulnerabilities found
- Biases in keystream
- Key-recovery attacks

Still seen in:
- Legacy systems
- Old Windows protocols
- MUST be disabled in modern systems
```

---

## 10. Keys and IVs

### 10.1 Key Generation

```python
# WRONG - Predictable!
import random
key = bytes([random.randint(0, 255) for _ in range(32)])

# CORRECT - Cryptographically secure
import os
key = os.urandom(32)

# Also correct
from Crypto.Random import get_random_bytes
key = get_random_bytes(32)
```

### 10.2 Key Derivation from Password

```python
from Crypto.Protocol.KDF import PBKDF2
from Crypto.Random import get_random_bytes

password = "user_password"
salt = get_random_bytes(16)

# Derive 256-bit key from password
key = PBKDF2(
    password,
    salt,
    dkLen=32,       # 256 bits
    count=100000    # Iterations (slow down brute force)
)

# Store salt with encrypted data (salt is not secret)
```

### 10.3 IV/Nonce Best Practices

```
+------------------------------------------------------------------+
|                    IV/NONCE GUIDELINES                            |
+------------------------------------------------------------------+

Property          CBC             CTR/GCM
────────────────────────────────────────────────────────────────
Size              Block size      Typically 96 bits
                  (128 bits)

Must be...        Unpredictable   Unique (can be counter)

Can be...         Random each     Random OR sequential
                  time

Reuse impact      Info leak       Catastrophic!

Storage           Send with       Send with
                  ciphertext      ciphertext

Generation:
─────────────
CBC: os.urandom(16)  # Must be random
CTR: Counter OR random (must NEVER repeat)
GCM: os.urandom(12)  # 96-bit random
```

---

## 11. Padding

### 11.1 Why Padding?

Block ciphers need input = multiple of block size.

```
AES block size: 16 bytes

Input: "Hello" (5 bytes)
Need: 16 bytes
Padding: 11 bytes

Input: "Hello World!!!!" (16 bytes)
Need: Still 16 more! (so we know where data ends)
Padding: 16 bytes
```

### 11.2 PKCS#7 Padding

Most common padding scheme:

```
PKCS#7 Rule: Pad with N bytes of value N

Examples:
─────────────────────────────────────────
Data: "Hello" (5 bytes)
Padded: "Hello" + 0x0B 0x0B 0x0B 0x0B 0x0B 0x0B 0x0B 0x0B 0x0B 0x0B 0x0B
        (11 bytes of value 11 = 0x0B)

Data: "Hello World!" (12 bytes)
Padded: "Hello World!" + 0x04 0x04 0x04 0x04
        (4 bytes of value 4)

Data: "16 bytes exactly" (16 bytes)
Padded: "16 bytes exactly" + 0x10 x 16
        (16 bytes of value 16 = full block of padding!)
```

### 11.3 Padding in Python

```python
from Crypto.Util.Padding import pad, unpad

block_size = 16

# Pad
data = b"Hello"
padded = pad(data, block_size)
print(f"Padded: {padded.hex()}")
# 48656c6c6f0b0b0b0b0b0b0b0b0b0b0b

# Unpad
unpadded = unpad(padded, block_size)
print(f"Unpadded: {unpadded}")
# b'Hello'

# Invalid padding raises exception!
try:
    unpad(b"Invalid padding!", block_size)
except ValueError as e:
    print(f"Padding error: {e}")
```

---

## 12. Practical Implementation

### 12.1 Complete AES-GCM Example

```python
from Crypto.Cipher import AES
from Crypto.Random import get_random_bytes
from Crypto.Protocol.KDF import PBKDF2
import json
import base64

class AESCipher:
    def __init__(self, password):
        self.password = password
    
    def encrypt(self, plaintext, associated_data=b""):
        # Generate salt and derive key
        salt = get_random_bytes(16)
        key = PBKDF2(self.password, salt, dkLen=32, count=100000)
        
        # Encrypt
        nonce = get_random_bytes(12)
        cipher = AES.new(key, AES.MODE_GCM, nonce=nonce)
        cipher.update(associated_data)
        ciphertext, tag = cipher.encrypt_and_digest(plaintext.encode())
        
        # Package everything
        result = {
            'salt': base64.b64encode(salt).decode(),
            'nonce': base64.b64encode(nonce).decode(),
            'ciphertext': base64.b64encode(ciphertext).decode(),
            'tag': base64.b64encode(tag).decode()
        }
        return json.dumps(result)
    
    def decrypt(self, encrypted_json, associated_data=b""):
        # Unpackage
        data = json.loads(encrypted_json)
        salt = base64.b64decode(data['salt'])
        nonce = base64.b64decode(data['nonce'])
        ciphertext = base64.b64decode(data['ciphertext'])
        tag = base64.b64decode(data['tag'])
        
        # Derive key
        key = PBKDF2(self.password, salt, dkLen=32, count=100000)
        
        # Decrypt and verify
        cipher = AES.new(key, AES.MODE_GCM, nonce=nonce)
        cipher.update(associated_data)
        plaintext = cipher.decrypt_and_verify(ciphertext, tag)
        
        return plaintext.decode()

# Usage
cipher = AESCipher("my_password")
encrypted = cipher.encrypt("Super secret message!")
print(f"Encrypted: {encrypted}")

decrypted = cipher.decrypt(encrypted)
print(f"Decrypted: {decrypted}")
```

### 12.2 OpenSSL Commands

```bash
# Generate random key
openssl rand -hex 32

# Encrypt file with AES-256-GCM
openssl enc -aes-256-gcm -salt -in plain.txt -out encrypted.bin -pass pass:password

# Decrypt
openssl enc -d -aes-256-gcm -in encrypted.bin -out decrypted.txt -pass pass:password

# Encrypt with explicit key and IV
KEY=$(openssl rand -hex 32)
IV=$(openssl rand -hex 12)
openssl enc -aes-256-ctr -in plain.txt -out encrypted.bin -K $KEY -iv $IV

# Show supported ciphers
openssl enc -list
```

---

## 13. Common Attacks

### 13.1 Key Reuse Attacks

```
Problem: Using same key for different purposes

Attack: Cross-protocol attacks, key recovery

Defense: Derive separate keys for each purpose
key_enc = HKDF(master_key, info=b"encryption")
key_mac = HKDF(master_key, info=b"authentication")
```

### 13.2 Padding Oracle Attack

```
Target: CBC mode with padding validation

Attack Steps:
1. Send modified ciphertext to server
2. Observe if server says "invalid padding" vs "decryption failed"
3. Use this oracle to decrypt byte-by-byte

Defense: Use AEAD (GCM), constant-time comparison
```

### 13.3 Bit Flipping Attack

```
Target: CBC mode without authentication

Attack: Modify ciphertext to change plaintext predictably

Example:
Original: "user=alice"
Attacker flips bits in previous block
Result: "user=admin"

Defense: Use AEAD (GCM) for authentication
```

---

## 14. Real-World Applications

### 14.1 TLS/HTTPS

```
TLS 1.3 Cipher Suites:
─────────────────────────────────────────
TLS_AES_256_GCM_SHA384
TLS_AES_128_GCM_SHA256
TLS_CHACHA20_POLY1305_SHA256

Process:
1. Asymmetric: Exchange symmetric key (ECDHE)
2. Symmetric: Encrypt all data with AES-GCM or ChaCha20
```

### 14.2 Disk Encryption

```
BitLocker (Windows):
- AES-128 or AES-256
- XTS mode (tweakable block cipher)
- TPM key protection

LUKS (Linux):
- AES-256-XTS
- Key slots for multiple passwords
- Header contains encrypted master key
```

### 14.3 VPNs

```
IPsec:
- AES-GCM preferred
- ESP (Encapsulating Security Payload)

WireGuard:
- ChaCha20-Poly1305 only
- Simple, modern, fast
```

---

## 15. Hands-On Labs

### Lab 1: ECB vs CBC Visual

```python
"""
Demonstrate why ECB is dangerous
"""
from Crypto.Cipher import AES
from Crypto.Random import get_random_bytes

key = get_random_bytes(16)

# Create repeating plaintext
plaintext = b"AAAAAAAAAAAAAAAA" * 4  # 64 bytes, same block repeated

# ECB Mode
ecb = AES.new(key, AES.MODE_ECB)
ecb_ciphertext = ecb.encrypt(plaintext)

# CBC Mode
iv = get_random_bytes(16)
cbc = AES.new(key, AES.MODE_CBC, iv)
cbc_ciphertext = cbc.encrypt(plaintext)

print("ECB Ciphertext (notice repeated blocks!):")
for i in range(0, len(ecb_ciphertext), 16):
    print(f"  Block {i//16}: {ecb_ciphertext[i:i+16].hex()}")

print("\nCBC Ciphertext (all blocks different!):")
for i in range(0, len(cbc_ciphertext), 16):
    print(f"  Block {i//16}: {cbc_ciphertext[i:i+16].hex()}")
```

### Lab 2: Implement Simple XOR Cipher

```python
"""
Understand XOR encryption (foundation of stream ciphers)
"""
def xor_encrypt(plaintext, key):
    """XOR plaintext with repeating key"""
    result = []
    for i, byte in enumerate(plaintext):
        key_byte = key[i % len(key)]
        result.append(byte ^ key_byte)
    return bytes(result)

# XOR is symmetric - same operation encrypts and decrypts!
xor_decrypt = xor_encrypt

# Test
key = b"SECRET"
plaintext = b"Hello, World! This is a secret message."

ciphertext = xor_encrypt(plaintext, key)
print(f"Ciphertext: {ciphertext.hex()}")

decrypted = xor_decrypt(ciphertext, key)
print(f"Decrypted: {decrypted}")

# Why this is weak: frequency analysis, known plaintext, etc.
```

---

## 16. Interview Questions

**Q1: What's the difference between block and stream ciphers?**
> Block ciphers encrypt fixed-size blocks (AES: 128 bits), stream ciphers encrypt bit-by-bit using a keystream (ChaCha20). Block ciphers need modes and padding, stream ciphers don't.

**Q2: Why is ECB mode insecure?**
> Identical plaintext blocks produce identical ciphertext blocks, revealing patterns. Also allows block reordering attacks.

**Q3: What is authenticated encryption (AEAD)?**
> Provides both confidentiality AND integrity/authenticity. Detects tampering. Examples: AES-GCM, ChaCha20-Poly1305. Prevents padding oracle and bit flipping attacks.

**Q4: Why must nonces never be reused in CTR/GCM?**
> Nonce reuse causes keystream reuse. C1⊕C2=P1⊕P2, revealing plaintext relationship. In GCM, also enables auth tag forgery.

**Q5: When would you use ChaCha20 instead of AES?**
> On devices without AES hardware acceleration (mobile, embedded), or when software-only implementation is required. ChaCha20 is faster in pure software.

---

## 17. Quick Reference

### Algorithm Recommendations

| Use Case | Algorithm |
|----------|-----------|
| General encryption | AES-256-GCM |
| TLS/HTTPS | AES-GCM or ChaCha20-Poly1305 |
| Disk encryption | AES-256-XTS |
| Mobile/embedded | ChaCha20-Poly1305 |

### Mode Comparison

| Mode | Auth | Parallel | Use? |
|------|------|----------|------|
| ECB | No | Yes | NEVER |
| CBC | No | Decrypt | Legacy |
| CTR | No | Yes | With MAC |
| GCM | Yes | Yes | RECOMMENDED |

### Python Quick Reference

```python
from Crypto.Cipher import AES
from Crypto.Random import get_random_bytes

# AES-GCM (recommended)
key = get_random_bytes(32)
nonce = get_random_bytes(12)
cipher = AES.new(key, AES.MODE_GCM, nonce=nonce)
ciphertext, tag = cipher.encrypt_and_digest(plaintext)

# Decrypt and verify
cipher = AES.new(key, AES.MODE_GCM, nonce=nonce)
plaintext = cipher.decrypt_and_verify(ciphertext, tag)
```

---

## Next Steps

Continue to:
- **[[05_Asymmetric_Encryption]]** - RSA, ECC, key exchange
- **[[08_Cryptographic_Attacks]]** - Exploiting crypto weaknesses

---

## References

- [[01_Cryptography_Fundamentals]]
- [[03_Hashing_and_Integrity]]
- [NIST AES Specification](https://csrc.nist.gov/publications/detail/fips/197/final)
- [PyCryptodome Documentation](https://pycryptodome.readthedocs.io/)
