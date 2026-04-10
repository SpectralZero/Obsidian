# 05 - Asymmetric Encryption: RSA, ECC, and Key Exchange

---
title: "Asymmetric Encryption: Public Key Cryptography Explained"
parent: "[[07_Cryptography]]"
tags:
  - "encryption"
  - "asymmetric"
  - "rsa"
  - "ecc"
  - "diffie-hellman"
  - "public-key"
created: 2026-02-07
updated: 2026-02-07
difficulty: "Intermediate to Advanced"
---

> **Executive Summary**: Asymmetric (public-key) cryptography revolutionized secure communication by solving the key distribution problem. Instead of sharing a secret key, each party has a KEY PAIR: a public key (shared freely) and a private key (kept secret). This enables encryption, digital signatures, and key exchange without prior secret sharing. This guide covers RSA, Elliptic Curve Cryptography (ECC), Diffie-Hellman key exchange, and practical applications from HTTPS to SSH.

**What You'll Learn**:
- How public/private key pairs work
- RSA: The original public-key system
- Elliptic Curve Cryptography (ECC)
- Diffie-Hellman key exchange
- Digital signatures
- Hybrid encryption (combining symmetric + asymmetric)
- Key management and security considerations

---

## Table of Contents

1. [What is Asymmetric Encryption? (ABSOLUTE BEGINNER)](#1-what-is-asymmetric-encryption)
2. [The Key Pair Concept](#2-the-key-pair)
3. [RSA: The Foundation](#3-rsa)
4. [RSA Mathematics](#4-rsa-mathematics)
5. [Digital Signatures](#5-digital-signatures)
6. [Elliptic Curve Cryptography](#6-ecc)
7. [Diffie-Hellman Key Exchange](#7-diffie-hellman)
8. [ECDH and Modern Key Exchange](#8-ecdh)
9. [Hybrid Encryption](#9-hybrid-encryption)
10. [Key Sizes and Security](#10-key-sizes)
11. [Practical Implementation](#11-practical-implementation)
12. [Attacks on Asymmetric Crypto](#12-attacks)
13. [Real-World Applications](#13-real-world)
14. [Hands-On Labs](#14-hands-on-labs)
15. [Interview Questions](#15-interview-questions)
16. [Quick Reference](#16-quick-reference)

---

## 1. What is Asymmetric Encryption? (ABSOLUTE BEGINNER)

### 1.1 The Problem with Symmetric Encryption

Remember from symmetric encryption: You need to share a secret key. But how?

```
+------------------------------------------------------------------+
|                    THE KEY EXCHANGE PROBLEM                       |
+------------------------------------------------------------------+

Alice (New York)                           Bob (Tokyo)
     │                                          │
     │   "I want to send you encrypted mail"   │
     │────────────────────────────────────────>│
     │                                          │
     │        "Ok, what's the password?"        │
     │<────────────────────────────────────────│
     │                                          │
     │   "The password is... wait, how do I    │
     │    tell you without Eve hearing?"        │

Eve (listening to everything!)

The internet is INSECURE by default!
```

### 1.2 The Brilliant Solution

**Asymmetric cryptography** uses TWO mathematically related keys:

```
+------------------------------------------------------------------+
|                    THE TWO-KEY SOLUTION                           |
+------------------------------------------------------------------+

Each person has a KEY PAIR:

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   PUBLIC KEY                    PRIVATE KEY                     │
│   ──────────                    ───────────                     │
│   • Share with everyone         • Keep SECRET                   │
│   • Published online            • Never leaves your device      │
│   • Used to ENCRYPT             • Used to DECRYPT               │
│   • Verifies signatures         • Creates signatures            │
│                                                                  │
│   Like a MAILBOX:               Like the MAILBOX KEY:           │
│   Anyone can drop mail in       Only you can open it            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 1.3 How It Works

```
+------------------------------------------------------------------+
|                    ASYMMETRIC ENCRYPTION FLOW                     |
+------------------------------------------------------------------+

1. Bob generates a key pair
   ┌─────────────────────────────────────────────────────────────┐
   │  Bob's Public Key: [can be shared]                          │
   │  Bob's Private Key: [SECRET - Bob keeps this]               │
   └─────────────────────────────────────────────────────────────┘

2. Bob publishes his PUBLIC key (website, email, keyserver)

3. Alice wants to send Bob a secret message
   ┌─────────────────────────────────────────────────────────────┐
   │  Alice gets Bob's public key (from website)                 │
   │  Alice encrypts message with Bob's PUBLIC key               │
   │  Alice sends encrypted message                               │
   └─────────────────────────────────────────────────────────────┘

4. Bob decrypts with his PRIVATE key
   ┌─────────────────────────────────────────────────────────────┐
   │  Only Bob has the private key                               │
   │  Only Bob can decrypt                                        │
   │  Eve has public key but CANNOT decrypt!                     │
   └─────────────────────────────────────────────────────────────┘
```

### 1.4 The Mailbox Analogy

```
Public Key = Mailbox slot (anyone can drop mail in)
Private Key = Mailbox key (only owner can retrieve mail)

          ┌─────────────────┐
          │   📬 MAILBOX    │
          │                 │
   ───────│   [SLOT]  ◄──── Anyone can insert mail (encrypt)
          │                 │
          │   [LOCK]  ◄──── Only owner can open (decrypt)
          │                 │
          └─────────────────┘
```

---

## 2. The Key Pair Concept

### 2.1 Mathematical Relationship

The public and private keys are mathematically related:

```
+------------------------------------------------------------------+
|                    KEY PAIR RELATIONSHIP                          |
+------------------------------------------------------------------+

                    MATHEMATICALLY RELATED
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
        ▼                  │                  ▼
  ┌──────────┐             │            ┌──────────┐
  │  PUBLIC  │             │            │ PRIVATE  │
  │   KEY    │ ◄───────────┼──────────► │   KEY    │
  └──────────┘             │            └──────────┘
        │                  │                  │
        ▼                  │                  ▼
   Can encrypt             │             Can decrypt
   Can verify              │             Can sign
                           │
                 ONE-WAY RELATIONSHIP:
                 Easy: Private → Public
                 Hard: Public → Private
```

### 2.2 The One-Way Function

```
+------------------------------------------------------------------+
|                    ONE-WAY FUNCTION (TRAPDOOR)                    |
+------------------------------------------------------------------+

Easy Direction (computing):
Private Key → Derive → Public Key
   seconds        ↓

Hard Direction (breaking):
Public Key → Derive → Private Key
   Would take billions of years with current computers!

This is called a TRAPDOOR function:
- Easy one way
- Hard the other way
- UNLESS you have the trapdoor (private key)
```

### 2.3 Two Operations

Asymmetric crypto provides two main operations:

```
1. ENCRYPTION/DECRYPTION
   ────────────────────────────────────────
   Encrypt with PUBLIC key → Only PRIVATE key can decrypt
   Purpose: Confidentiality

2. SIGNING/VERIFICATION
   ────────────────────────────────────────
   Sign with PRIVATE key → Anyone with PUBLIC key can verify
   Purpose: Authenticity, Non-repudiation

   ┌──────────────────────────────────────────────────────────────┐
   │                                                               │
   │  ENCRYPTION:  Public encrypts  ──────►  Private decrypts    │
   │  SIGNING:     Private signs    ◄──────  Public verifies     │
   │                                                               │
   │  They are INVERSE operations!                                │
   │                                                               │
   └──────────────────────────────────────────────────────────────┘
```

---

## 3. RSA: The Foundation

### 3.1 RSA Overview

**RSA** (Rivest-Shamir-Adleman, 1977) was the first practical public-key cryptosystem.

```
RSA Security Based On:
─────────────────────────────────────────
Integer Factorization Problem

Easy: Multiply two primes
  p = 61, q = 53
  n = p × q = 3233

Hard: Factor the product
  Given n = 3233, find p and q
  (For small numbers, easy. For 2048-bit numbers, infeasible!)
```

### 3.2 RSA Key Generation

```
Step-by-Step RSA Key Generation:
─────────────────────────────────────────

1. Choose two large primes: p and q
   p = 61, q = 53 (real RSA uses 1024+ bit primes!)

2. Compute n = p × q
   n = 61 × 53 = 3233 (this is the MODULUS)

3. Compute φ(n) = (p-1)(q-1)
   φ(n) = 60 × 52 = 3120 (Euler's totient)

4. Choose public exponent e
   Common choice: e = 65537 (0x10001)
   Requirement: gcd(e, φ(n)) = 1

5. Compute private exponent d
   d × e ≡ 1 (mod φ(n))
   d = e^(-1) mod φ(n)

PUBLIC KEY:  (n, e) = (3233, 17)
PRIVATE KEY: (n, d) = (3233, 2753)
```

### 3.3 RSA Encryption/Decryption

```
RSA Operations:
─────────────────────────────────────────

ENCRYPT (with public key):
ciphertext = plaintext^e mod n

DECRYPT (with private key):
plaintext = ciphertext^d mod n

Example with our keys:
─────────────────────────────────────────
Message m = 65 (ASCII 'A')

Encrypt: c = 65^17 mod 3233 = 2790
Decrypt: m = 2790^2753 mod 3233 = 65 ✓
```

### 3.4 RSA in Python

```python
from Crypto.PublicKey import RSA
from Crypto.Cipher import PKCS1_OAEP
import base64

# Generate key pair
key = RSA.generate(2048)
private_key = key
public_key = key.publickey()

print("Public key (n, e):")
print(f"  n = {public_key.n}")
print(f"  e = {public_key.e}")

# Export keys (PEM format)
private_pem = private_key.export_key()
public_pem = public_key.export_key()

print(f"\nPublic Key:\n{public_pem.decode()[:100]}...")

# Encrypt with public key
cipher = PKCS1_OAEP.new(public_key)
plaintext = b"Hello, RSA!"
ciphertext = cipher.encrypt(plaintext)
print(f"\nCiphertext: {base64.b64encode(ciphertext).decode()[:50]}...")

# Decrypt with private key
cipher = PKCS1_OAEP.new(private_key)
decrypted = cipher.decrypt(ciphertext)
print(f"Decrypted: {decrypted.decode()}")
```

---

## 4. RSA Mathematics

### 4.1 Modular Arithmetic Basics

```
Modular arithmetic is "clock arithmetic":
─────────────────────────────────────────

12 mod 5 = 2 (12 ÷ 5 = 2 remainder 2)
27 mod 7 = 6 (27 ÷ 7 = 3 remainder 6)

Properties:
(a + b) mod n = ((a mod n) + (b mod n)) mod n
(a × b) mod n = ((a mod n) × (b mod n)) mod n
(a^b) mod n = ((a mod n)^b) mod n

These properties make RSA efficient!
```

### 4.2 Why RSA Works (Euler's Theorem)

```
Euler's Theorem:
─────────────────────────────────────────
For any a and n where gcd(a, n) = 1:
a^φ(n) ≡ 1 (mod n)

RSA Uses This:
─────────────────────────────────────────
We chose e and d such that:
e × d ≡ 1 (mod φ(n))
e × d = 1 + k × φ(n) for some integer k

Decryption:
c^d = (m^e)^d = m^(e×d) = m^(1 + k×φ(n))
    = m × (m^φ(n))^k ≡ m × 1^k ≡ m (mod n)

We get back the original message!
```

### 4.3 RSA Padding (OAEP)

Raw RSA is vulnerable. Always use padding!

```
+------------------------------------------------------------------+
|                    RSA PADDING SCHEMES                            |
+------------------------------------------------------------------+

PKCS#1 v1.5 (Legacy):
- Simple padding scheme
- Vulnerable to attacks (Bleichenbacher)
- Still widely used (TLS 1.2)

OAEP (Optimal Asymmetric Encryption Padding):
- More secure
- Uses hash functions and random data
- Recommended for new implementations

Structure:
┌────────────────────────────────────────────────────────────────┐
│ 0x00 │ 0x02 │ Random Padding │ 0x00 │ Message │              │
└────────────────────────────────────────────────────────────────┘
                        PKCS#1 v1.5

┌────────────────────────────────────────────────────────────────┐
│    Encoded Message = (Message ⊕ MGF(Seed)) || MGF(...)        │
└────────────────────────────────────────────────────────────────┘
                           OAEP
```

---

## 5. Digital Signatures

### 5.1 What is a Digital Signature?

A digital signature proves:
1. **Authenticity**: Message came from claimed sender
2. **Integrity**: Message wasn't modified
3. **Non-repudiation**: Sender cannot deny sending

```
+------------------------------------------------------------------+
|                    DIGITAL SIGNATURE PROCESS                      |
+------------------------------------------------------------------+

SIGNING (by sender):
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  Document ───► Hash Function ───► Hash Digest                   │
│                                        │                        │
│                                        ▼                        │
│                              ┌──────────────────┐               │
│  Private Key ───────────────►│ Sign (Encrypt)   │               │
│                              └────────┬─────────┘               │
│                                       │                         │
│                                       ▼                         │
│                                   SIGNATURE                     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

Send: Document + Signature

VERIFICATION (by receiver):
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  Document ───► Hash Function ───► Calculated Hash               │
│                                                                  │
│  Signature ───► Decrypt with ───► Received Hash                 │
│                 Public Key                                       │
│                                                                  │
│  Compare: Calculated Hash == Received Hash ?                    │
│           If yes: Signature is VALID                            │
│           If no:  Signature is INVALID                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 5.2 Why Sign the Hash?

```
Why not sign the whole document?

1. RSA is SLOW
   - Signing 1GB document would take forever
   - Signing 256-bit hash is instant

2. RSA has size limits
   - Can only encrypt data smaller than key size
   - 2048-bit RSA can encrypt ~245 bytes max

3. Security
   - Hash provides fixed-size input
   - Consistent security properties
```

### 5.3 Signature Algorithms

```
Common Signature Algorithms:
─────────────────────────────────────────

RSA-SHA256:
- RSA signature with SHA-256 hash
- Widely used

ECDSA (Elliptic Curve DSA):
- Smaller signatures than RSA
- Same security with smaller keys
- Used in Bitcoin, TLS

Ed25519:
- Modern, fast, secure
- Fixed curve (Curve25519)
- Used in SSH, Signal
```

### 5.4 Signatures in Python

```python
from Crypto.PublicKey import RSA
from Crypto.Signature import pkcs1_15
from Crypto.Hash import SHA256

# Generate keys
key = RSA.generate(2048)
private_key = key
public_key = key.publickey()

# Sign a message
message = b"This is an important document"
hash_obj = SHA256.new(message)
signature = pkcs1_15.new(private_key).sign(hash_obj)

print(f"Signature: {signature.hex()[:64]}...")

# Verify signature
hash_obj = SHA256.new(message)
try:
    pkcs1_15.new(public_key).verify(hash_obj, signature)
    print("Signature is VALID")
except (ValueError, TypeError):
    print("Signature is INVALID")

# Try with tampered message
tampered = b"This is a MODIFIED document"
hash_obj = SHA256.new(tampered)
try:
    pkcs1_15.new(public_key).verify(hash_obj, signature)
    print("Signature is VALID")
except (ValueError, TypeError):
    print("Signature is INVALID - tampering detected!")
```

---

## 6. Elliptic Curve Cryptography (ECC)

### 6.1 Why ECC?

RSA keys are getting bigger (2048+ bits). ECC provides same security with smaller keys!

```
Key Size Comparison:
─────────────────────────────────────────

Security Level   RSA Key Size   ECC Key Size   Ratio
──────────────────────────────────────────────────────
80 bits          1024 bits      160 bits       6:1
112 bits         2048 bits      224 bits       9:1
128 bits         3072 bits      256 bits       12:1
192 bits         7680 bits      384 bits       20:1
256 bits         15360 bits     512 bits       30:1

ECC-256 ≈ RSA-3072 in security!
```

### 6.2 Elliptic Curves Explained

```
+------------------------------------------------------------------+
|                    ELLIPTIC CURVE BASICS                          |
+------------------------------------------------------------------+

An elliptic curve: y² = x³ + ax + b

Looks like:
           *
          / \
         /   \
   *----*     *----*
         \   /
          \ /
           *

Key Operations:
1. Point Addition: P + Q = R
2. Point Doubling: P + P = 2P
3. Scalar Multiplication: n × P = P + P + ... (n times)

The "Hard Problem" (ECDLP):
Given: G (generator point) and Q = n × G
Find: n (the scalar/private key)

This is HARD when n is large!
```

### 6.3 ECC Key Generation

```
ECC Key Generation:
─────────────────────────────────────────

1. Choose a curve (e.g., P-256, secp256k1, Curve25519)
   - Curve defines: a, b, p, G (generator), n (order)

2. Generate private key: d (random number, 1 < d < n)

3. Compute public key: Q = d × G (scalar multiplication)

PRIVATE KEY: d (256-bit number)
PUBLIC KEY:  Q (point on curve, x and y coordinates)

Example (simplified):
Curve: secp256k1 (Bitcoin's curve)
d = 0x1234...abcd (256-bit random)
Q = d × G = (x, y) coordinates on curve
```

### 6.4 Common Curves

```
Standard Curves:
─────────────────────────────────────────

NIST Curves (US Government Standard):
- P-256 (secp256r1): 256-bit, widely used
- P-384 (secp384r1): 384-bit, higher security
- P-521 (secp521r1): 521-bit, highest security

Bitcoin/Ethereum:
- secp256k1: Used for cryptocurrency

Modern Curves (Recommended):
- Curve25519 (X25519): Key exchange, very fast
- Ed25519: Signatures, deterministic

Why Modern Curves?
- Designed to be resistant to side-channel attacks
- No suspicious constants (unlike NIST curves)
- Faster and simpler implementation
```

### 6.5 ECC in Python

```python
from cryptography.hazmat.primitives.asymmetric import ec
from cryptography.hazmat.primitives import hashes, serialization
from cryptography.hazmat.primitives.asymmetric.utils import decode_dss_signature

# Generate ECC key pair (P-256)
private_key = ec.generate_private_key(ec.SECP256R1())
public_key = private_key.public_key()

# Export public key
public_bytes = public_key.public_bytes(
    encoding=serialization.Encoding.PEM,
    format=serialization.PublicFormat.SubjectPublicKeyInfo
)
print(f"Public Key:\n{public_bytes.decode()}")

# Sign a message with ECDSA
message = b"Important document"
signature = private_key.sign(message, ec.ECDSA(hashes.SHA256()))
print(f"Signature: {signature.hex()[:64]}...")

# Verify signature
try:
    public_key.verify(signature, message, ec.ECDSA(hashes.SHA256()))
    print("Signature VALID")
except:
    print("Signature INVALID")
```

---

## 7. Diffie-Hellman Key Exchange

### 7.1 The Genius Idea

Diffie-Hellman (1976) allows two parties to establish a shared secret over an insecure channel!

```
+------------------------------------------------------------------+
|                    DIFFIE-HELLMAN MAGIC                           |
+------------------------------------------------------------------+

The Paint Mixing Analogy:
─────────────────────────────────────────

Alice                 Eve (watching)               Bob
  │                        │                        │
  │  1. Agree on public color (yellow)              │
  │<─────────────────────────────────────────────>│
  │                        │                        │
  │  2. Alice picks SECRET color (red)              │
  │     Bob picks SECRET color (blue)               │
  │                        │                        │
  │  3. Mix with yellow                             │
  │     Alice: yellow + red = ORANGE                │
  │     Bob: yellow + blue = GREEN                  │
  │                        │                        │
  │  4. Exchange mixtures                           │
  │───────── ORANGE ─────────────>│<── GREEN ──────│
  │                        │                        │
  │  5. Add own secret color                        │
  │     Alice: GREEN + red = BROWN                  │
  │     Bob: ORANGE + blue = BROWN                  │
  │                        │                        │
  │  BOTH HAVE BROWN! (shared secret)               │
  │                        │                        │
  │  Eve has: yellow, orange, green                 │
  │  Eve CANNOT make brown without red or blue!     │
```

### 7.2 DH Mathematics

```
Diffie-Hellman Algorithm:
─────────────────────────────────────────

Public Parameters (shared):
- p: Large prime number
- g: Generator (primitive root of p)

Alice:
1. Choose secret: a (random)
2. Compute: A = g^a mod p
3. Send A to Bob

Bob:
1. Choose secret: b (random)
2. Compute: B = g^b mod p
3. Send B to Alice

Shared Secret:
Alice computes: s = B^a mod p = (g^b)^a mod p = g^(ab) mod p
Bob computes:   s = A^b mod p = (g^a)^b mod p = g^(ab) mod p

Both get the same s!

Eve sees: p, g, A, B
Eve needs: a or b
Finding a from A = g^a mod p is the Discrete Logarithm Problem (HARD!)
```

### 7.3 DH Example

```python
# Simplified Diffie-Hellman example
import random

# Public parameters (in reality, use much larger primes!)
p = 23  # Prime
g = 5   # Generator

# Alice's side
a = random.randint(1, p-1)  # Secret
A = pow(g, a, p)            # Public value
print(f"Alice secret: {a}, sends: {A}")

# Bob's side
b = random.randint(1, p-1)  # Secret
B = pow(g, b, p)            # Public value
print(f"Bob secret: {b}, sends: {B}")

# Shared secret computation
alice_shared = pow(B, a, p)
bob_shared = pow(A, b, p)

print(f"Alice's shared secret: {alice_shared}")
print(f"Bob's shared secret: {bob_shared}")
print(f"Same? {alice_shared == bob_shared}")
```

### 7.4 DH Security Considerations

```
Vulnerabilities:
─────────────────────────────────────────

1. Man-in-the-Middle Attack
   - Eve intercepts and substitutes her own values
   - Solution: Authenticate the key exchange (signatures, certificates)

2. Small Subgroup Attack
   - Attacker sends malicious values
   - Solution: Validate received values

3. Logjam Attack (2015)
   - Many servers used same 512-bit or 1024-bit primes
   - Solution: Use 2048+ bit groups or ECDH
```

---

## 8. ECDH and Modern Key Exchange

### 8.1 ECDH (Elliptic Curve Diffie-Hellman)

Same concept as DH, but using elliptic curves for smaller keys.

```
ECDH Process:
─────────────────────────────────────────

Public Parameters: Curve (e.g., X25519), Generator G

Alice:
1. Private key: a (random scalar)
2. Public key: A = a × G (point on curve)
3. Sends A to Bob

Bob:
1. Private key: b (random scalar)
2. Public key: B = b × G (point on curve)
3. Sends B to Alice

Shared Secret:
Alice: S = a × B = a × (b × G) = (a × b) × G
Bob:   S = b × A = b × (a × G) = (a × b) × G

Same point S!
```

### 8.2 X25519 (Curve25519 for Key Exchange)

```
X25519 Advantages:
─────────────────────────────────────────
✓ Designed by Daniel J. Bernstein
✓ Fast (faster than P-256)
✓ Resistant to timing attacks
✓ Simple implementation
✓ Used in TLS 1.3, Signal, WireGuard
```

### 8.3 ECDH in Python

```python
from cryptography.hazmat.primitives.asymmetric import x25519
from cryptography.hazmat.primitives import hashes
from cryptography.hazmat.primitives.kdf.hkdf import HKDF

# Alice generates her key pair
alice_private = x25519.X25519PrivateKey.generate()
alice_public = alice_private.public_key()

# Bob generates his key pair
bob_private = x25519.X25519PrivateKey.generate()
bob_public = bob_private.public_key()

# Key exchange
alice_shared = alice_private.exchange(bob_public)
bob_shared = bob_private.exchange(alice_public)

print(f"Alice's shared secret: {alice_shared.hex()[:32]}...")
print(f"Bob's shared secret:   {bob_shared.hex()[:32]}...")
print(f"Same? {alice_shared == bob_shared}")

# Derive actual encryption key using HKDF
derived_key = HKDF(
    algorithm=hashes.SHA256(),
    length=32,
    salt=None,
    info=b'handshake data',
).derive(alice_shared)

print(f"Derived AES key: {derived_key.hex()}")
```

### 8.4 Ephemeral Key Exchange (ECDHE)

```
+------------------------------------------------------------------+
|                    EPHEMERAL VS STATIC                            |
+------------------------------------------------------------------+

STATIC DH/ECDH:
- Same key pair used for multiple sessions
- If private key is compromised, ALL past sessions decrypted!

EPHEMERAL DH/ECDH (DHE/ECDHE):
- New key pair generated for EACH session
- Private key discarded after session
- Provides FORWARD SECRECY

Forward Secrecy:
Even if long-term key is compromised later,
past sessions remain secure!

TLS Cipher Suites:
─────────────────────────────────────────
TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
    │    │
    │    └── RSA for authentication
    └────── ECDHE for key exchange (ephemeral!)
```

---

## 9. Hybrid Encryption

### 9.1 Why Hybrid?

```
+------------------------------------------------------------------+
|                    THE HYBRID APPROACH                            |
+------------------------------------------------------------------+

Problem:
- Asymmetric: Secure key exchange, but SLOW
- Symmetric: Fast encryption, but key distribution problem

Solution: Use BOTH!

1. Use asymmetric to exchange a symmetric key
2. Use symmetric to encrypt the actual data

TLS does this:
1. ECDHE to establish shared secret
2. Derive AES key from shared secret
3. AES-GCM encrypts all data
```

### 9.2 Hybrid Encryption Flow

```
Hybrid Encryption:
─────────────────────────────────────────

SENDER:
1. Generate random symmetric key (AES-256 key)
2. Encrypt DATA with symmetric key (fast!)
3. Encrypt symmetric KEY with recipient's public key (RSA/ECC)
4. Send: Encrypted Key + Encrypted Data

RECEIVER:
1. Decrypt symmetric key with private key (RSA/ECC)
2. Decrypt data with symmetric key (fast!)
3. Read the message!

         ┌──────────────────────────────────────────────┐
         │                                               │
         │    Random AES Key ──► RSA Encrypt ──► [Encrypted Key]
         │          │                                    │
         │          ▼                                    │
         │    Large Data ──────► AES-GCM ────► [Encrypted Data]
         │                                               │
         │    Send: [Encrypted Key] + [Encrypted Data]  │
         │                                               │
         └──────────────────────────────────────────────┘
```

### 9.3 Hybrid Encryption in Python

```python
from Crypto.PublicKey import RSA
from Crypto.Cipher import PKCS1_OAEP, AES
from Crypto.Random import get_random_bytes
import base64

def hybrid_encrypt(public_key, plaintext):
    """Encrypt data using hybrid encryption"""
    # Generate random AES key
    aes_key = get_random_bytes(32)  # AES-256
    
    # Encrypt data with AES-GCM
    nonce = get_random_bytes(12)
    aes_cipher = AES.new(aes_key, AES.MODE_GCM, nonce=nonce)
    ciphertext, tag = aes_cipher.encrypt_and_digest(plaintext)
    
    # Encrypt AES key with RSA
    rsa_cipher = PKCS1_OAEP.new(public_key)
    encrypted_key = rsa_cipher.encrypt(aes_key)
    
    return encrypted_key, nonce, ciphertext, tag

def hybrid_decrypt(private_key, encrypted_key, nonce, ciphertext, tag):
    """Decrypt hybrid-encrypted data"""
    # Decrypt AES key with RSA
    rsa_cipher = PKCS1_OAEP.new(private_key)
    aes_key = rsa_cipher.decrypt(encrypted_key)
    
    # Decrypt data with AES-GCM
    aes_cipher = AES.new(aes_key, AES.MODE_GCM, nonce=nonce)
    plaintext = aes_cipher.decrypt_and_verify(ciphertext, tag)
    
    return plaintext

# Usage
key = RSA.generate(2048)
public_key = key.publickey()
private_key = key

message = b"Large amount of secret data " * 100

# Encrypt
encrypted_key, nonce, ciphertext, tag = hybrid_encrypt(public_key, message)
print(f"Encrypted key: {len(encrypted_key)} bytes")
print(f"Ciphertext: {len(ciphertext)} bytes")

# Decrypt
decrypted = hybrid_decrypt(private_key, encrypted_key, nonce, ciphertext, tag)
print(f"Decrypted: {decrypted[:50]}...")
```

---

## 10. Key Sizes and Security

### 10.1 Current Recommendations

```
+------------------------------------------------------------------+
|                    KEY SIZE RECOMMENDATIONS (2024)                |
+------------------------------------------------------------------+

Algorithm        Minimum    Recommended    High Security
────────────────────────────────────────────────────────────────────
RSA              2048       3072           4096
ECDSA/ECDH       256        384            521
Ed25519          256        256            256 (fixed)
X25519           256        256            256 (fixed)

Security Equivalence:
RSA-2048 ≈ ECC-224 ≈ 112-bit security
RSA-3072 ≈ ECC-256 ≈ 128-bit security
RSA-7680 ≈ ECC-384 ≈ 192-bit security
```

### 10.2 Quantum Threat

```
+------------------------------------------------------------------+
|                    QUANTUM COMPUTING THREAT                       |
+------------------------------------------------------------------+

Shor's Algorithm (on quantum computer):
- Factors large integers in polynomial time
- Solves discrete log in polynomial time
- BREAKS: RSA, DSA, ECDSA, ECDH, DH

When?
- Full-scale quantum computers: 10-20+ years (maybe)
- But: Harvest now, decrypt later attack!

Defense:
- Post-Quantum Cryptography (PQC)
- NIST standardizing: CRYSTALS-Kyber, CRYSTALS-Dilithium
- Hybrid approaches: Classic + PQC
```

---

## 11. Practical Implementation

### 11.1 Generate RSA Keys with OpenSSL

```bash
# Generate 4096-bit RSA private key
openssl genrsa -out private.pem 4096

# Extract public key
openssl rsa -in private.pem -pubout -out public.pem

# View key details
openssl rsa -in private.pem -text -noout

# Generate with passphrase
openssl genrsa -aes256 -out private_encrypted.pem 4096
```

### 11.2 Generate ECC Keys with OpenSSL

```bash
# Generate P-256 private key
openssl ecparam -genkey -name prime256v1 -out ec_private.pem

# Extract public key
openssl ec -in ec_private.pem -pubout -out ec_public.pem

# List available curves
openssl ecparam -list_curves
```

### 11.3 Sign and Verify with OpenSSL

```bash
# Sign a file
openssl dgst -sha256 -sign private.pem -out signature.bin document.txt

# Verify signature
openssl dgst -sha256 -verify public.pem -signature signature.bin document.txt
# Output: "Verified OK" or "Verification Failure"
```

---

## 12. Attacks on Asymmetric Crypto

### 12.1 Weak Key Generation

```
Attack: Predictable random number generation

Example (Debian OpenSSL Bug 2008):
- RNG had only 32,768 possible seeds
- All keys generated on affected systems were predictable
- Could generate all possible keys in hours!

Defense: Use proper cryptographic RNG (os.urandom, /dev/urandom)
```

### 12.2 Timing Attacks

```
Attack: Measure how long operations take

Example:
- RSA decryption time varies based on private key bits
- Attacker measures many operations
- Statistical analysis reveals private key

Defense: Constant-time implementations
```

### 12.3 Bleichenbacher Attack (RSA PKCS#1 v1.5)

```
Attack: Padding oracle on RSA encryption

Process:
1. Send modified ciphertexts to server
2. Server reveals if padding was valid (error messages, timing)
3. Decrypt ciphertext without key!

Defense: Use OAEP padding, not PKCS#1 v1.5
```

---

## 13. Real-World Applications

### 13.1 TLS/HTTPS

```
TLS 1.3 Key Exchange:
1. Client sends supported key shares (X25519, P-256)
2. Server selects one, sends its key share
3. Both compute shared secret
4. Derive encryption keys

Authentication:
- Server signs handshake with its certificate's private key
- Client verifies with server's public key (from certificate)
```

### 13.2 SSH

```
SSH Key Authentication:
1. User generates key pair (ssh-keygen)
2. Public key copied to server (~/.ssh/authorized_keys)
3. During login, server sends challenge
4. Client signs challenge with private key
5. Server verifies with public key
```

### 13.3 Code Signing

```
Code Signing Process:
1. Developer signs executable with private key
2. User's OS verifies signature with developer's public key
3. If valid, code is trusted to run

Examples:
- Windows Authenticode
- macOS code signing
- Android APK signing
```

---

## 14. Hands-On Labs

### Lab 1: RSA Key Generation

```python
"""
Manually implement simplified RSA key generation
"""
import random

def is_prime(n, k=10):
    """Miller-Rabin primality test"""
    if n < 2: return False
    if n == 2 or n == 3: return True
    if n % 2 == 0: return False
    
    r, d = 0, n - 1
    while d % 2 == 0:
        r += 1
        d //= 2
    
    for _ in range(k):
        a = random.randrange(2, n - 1)
        x = pow(a, d, n)
        if x == 1 or x == n - 1:
            continue
        for _ in range(r - 1):
            x = pow(x, 2, n)
            if x == n - 1:
                break
        else:
            return False
    return True

def generate_prime(bits):
    """Generate a prime number with specified bit length"""
    while True:
        n = random.getrandbits(bits)
        n |= (1 << bits - 1) | 1  # Ensure proper bit length and odd
        if is_prime(n):
            return n

def extended_gcd(a, b):
    """Extended Euclidean algorithm"""
    if a == 0:
        return b, 0, 1
    gcd, x1, y1 = extended_gcd(b % a, a)
    x = y1 - (b // a) * x1
    y = x1
    return gcd, x, y

def mod_inverse(e, phi):
    """Calculate modular multiplicative inverse"""
    gcd, x, _ = extended_gcd(e % phi, phi)
    if gcd != 1:
        raise Exception("Modular inverse doesn't exist")
    return (x % phi + phi) % phi

# Generate RSA keys
print("Generating RSA keys...")
bits = 512  # Small for demo - use 2048+ in practice!

p = generate_prime(bits // 2)
q = generate_prime(bits // 2)
n = p * q
phi = (p - 1) * (q - 1)
e = 65537
d = mod_inverse(e, phi)

print(f"p = {p}")
print(f"q = {q}")
print(f"n = {n}")
print(f"e = {e}")
print(f"d = {d}")

# Encrypt/Decrypt
message = 42
encrypted = pow(message, e, n)
decrypted = pow(encrypted, d, n)
print(f"\nOriginal: {message}")
print(f"Encrypted: {encrypted}")
print(f"Decrypted: {decrypted}")
```

---

## 15. Interview Questions

**Q1: What is the difference between symmetric and asymmetric encryption?**
> Symmetric uses one shared key for both encryption and decryption. Asymmetric uses a key pair: public key encrypts, private key decrypts. Symmetric is fast, asymmetric solves key distribution.

**Q2: Explain how RSA encryption works at a high level.**
> RSA relies on the difficulty of factoring large numbers. Public key (n, e) encrypts by computing c = m^e mod n. Private key d decrypts by computing m = c^d mod n. Security depends on the hardness of deriving d from e without knowing p and q.

**Q3: Why is ECC preferred over RSA in modern systems?**
> ECC provides equivalent security with much smaller keys (256-bit ECC ≈ 3072-bit RSA). Smaller keys mean faster operations, less bandwidth, and better performance on constrained devices.

**Q4: What is forward secrecy and how is it achieved?**
> Forward secrecy ensures that if a long-term private key is compromised, past session keys remain secure. Achieved by using ephemeral Diffie-Hellman (DHE/ECDHE) where session keys are generated fresh and discarded after use.

**Q5: What is a digital signature and what does it provide?**
> A digital signature is created by signing a message hash with a private key. It provides: Authentication (proves who sent it), Integrity (detects tampering), Non-repudiation (sender cannot deny sending).

---

## 16. Quick Reference

### Algorithm Summary

| Algorithm | Type | Use Case |
|-----------|------|----------|
| RSA | Encryption, Signatures | Legacy, wide compatibility |
| ECDSA | Signatures | TLS, Bitcoin |
| Ed25519 | Signatures | SSH, modern systems |
| ECDH | Key Exchange | TLS |
| X25519 | Key Exchange | TLS 1.3, WireGuard |

### Key Generation Commands

```bash
# RSA
openssl genrsa -out rsa_private.pem 4096
openssl rsa -in rsa_private.pem -pubout -out rsa_public.pem

# ECC (P-256)
openssl ecparam -genkey -name prime256v1 -out ec_private.pem
openssl ec -in ec_private.pem -pubout -out ec_public.pem

# Ed25519 (OpenSSL 1.1.1+)
openssl genpkey -algorithm Ed25519 -out ed25519_private.pem
openssl pkey -in ed25519_private.pem -pubout -out ed25519_public.pem
```

### Python Quick Reference

```python
from Crypto.PublicKey import RSA
from cryptography.hazmat.primitives.asymmetric import x25519, ec

# RSA
key = RSA.generate(2048)
public_key = key.publickey()

# X25519 (key exchange)
private = x25519.X25519PrivateKey.generate()
public = private.public_key()

# ECDSA (signatures)
private = ec.generate_private_key(ec.SECP256R1())
```

---

## Next Steps

Continue to:
- **[[06_PKI_TLS_and_Certificates]]** - How trust is established
- **[[10_SSH_and_Key_Management]]** - Practical key management

---

## References

- [[01_Cryptography_Fundamentals]]
- [[04_Symmetric_Encryption]]
- [A Graduate Course in Applied Cryptography](https://toc.cryptobook.us/)
- [RFC 8017 - PKCS #1](https://tools.ietf.org/html/rfc8017)
