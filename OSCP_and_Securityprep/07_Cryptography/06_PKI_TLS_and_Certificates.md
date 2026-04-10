# 06 - PKI, TLS, and Certificates: Trust on the Internet

---
title: "PKI, TLS, and Certificates: Building Trust Online"
parent: "[[07_Cryptography]]"
tags:
  - "pki"
  - "tls"
  - "certificates"
  - "ssl"
  - "https"
created: 2026-02-07
updated: 2026-02-07
difficulty: "Intermediate"
---

> **Executive Summary**: Public Key Infrastructure (PKI) is how we establish trust on the internet. When you visit a website, how do you know you're really talking to your bank and not an attacker? Certificates and the chain of trust make this possible. This guide covers X.509 certificates, Certificate Authorities, the TLS handshake, and practical certificate management - essential knowledge for web security, penetration testing, and OSCP.

**What You'll Learn**:
- How PKI creates trust hierarchies
- X.509 certificate structure
- Certificate Authorities (CAs) and chain of trust
- TLS handshake step-by-step
- TLS 1.2 vs TLS 1.3
- Certificate verification and revocation
- Practical certificate analysis with OpenSSL
- Common attacks (MitM, certificate forgery)

---

## Table of Contents

1. [What is PKI? (ABSOLUTE BEGINNER)](#1-what-is-pki)
2. [The Trust Problem](#2-the-trust-problem)
3. [Certificates Explained](#3-certificates-explained)
4. [X.509 Certificate Structure](#4-x509-structure)
5. [Certificate Authorities](#5-certificate-authorities)
6. [Chain of Trust](#6-chain-of-trust)
7. [TLS Overview](#7-tls-overview)
8. [TLS 1.2 Handshake](#8-tls-12-handshake)
9. [TLS 1.3 Handshake](#9-tls-13-handshake)
10. [Certificate Verification](#10-certificate-verification)
11. [Certificate Revocation](#11-certificate-revocation)
12. [Practical Certificate Management](#12-practical-management)
13. [Attacks and Defenses](#13-attacks-and-defenses)
14. [Hands-On Labs](#14-hands-on-labs)
15. [Interview Questions](#15-interview-questions)
16. [Quick Reference](#16-quick-reference)

---

## 1. What is PKI? (ABSOLUTE BEGINNER)

### 1.1 The Problem

You type `https://bank.com` in your browser. How do you know you're REALLY talking to your bank?

```
+------------------------------------------------------------------+
|                    THE IDENTITY PROBLEM                           |
+------------------------------------------------------------------+

Scenario: You want to visit your bank website

Without PKI:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   You ───────────────────────────────────────────────> bank.com │
│                      │                                           │
│                      │  Attacker (Man-in-the-Middle)            │
│                      └──> Fake Bank Site                        │
│                           Looks exactly like real bank!         │
│                           Steals your credentials!               │
│                                                                  │
│   How do you know which is the real bank?                       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 1.2 The Solution: PKI

**PKI** (Public Key Infrastructure) is a system for:
- Verifying identity online
- Binding public keys to identities
- Creating chains of trust

```
+------------------------------------------------------------------+
|                    PKI SOLUTION                                   |
+------------------------------------------------------------------+

Components:
1. CERTIFICATES: Digital identity documents
2. CERTIFICATE AUTHORITIES: Trusted issuers (like passport offices)
3. CHAIN OF TRUST: Hierarchy of trust

Result:
Your browser can VERIFY that bank.com is really your bank!
```

### 1.3 Real-World Analogy

```
PKI is like the passport system:
─────────────────────────────────────────

PASSPORT SYSTEM              PKI SYSTEM
────────────────             ──────────────
Passport Office              Certificate Authority
Passport                     Digital Certificate  
Your Photo + Info            Public Key + Identity
Signature/Stamp              Digital Signature
Border Agent                 Browser/Client

Border agent trusts passport    Browser trusts certificate
because they trust the          because they trust the
passport office                 Certificate Authority
```

---

## 2. The Trust Problem

### 2.1 Public Key Distribution

```
+------------------------------------------------------------------+
|                    THE DISTRIBUTION PROBLEM                       |
+------------------------------------------------------------------+

Problem: Alice wants Bob's public key

Option 1: Bob emails his public key
├── Eve intercepts, substitutes her own key!
└── Alice encrypts with Eve's key, not Bob's

Option 2: Meet in person
└── Not practical for internet scale

Option 3: Use PKI!
├── Bob gets certificate from trusted CA
├── Certificate contains Bob's identity + public key
├── CA digitally signs the certificate
├── Alice trusts CA, therefore trusts Bob's key
└── Eve cannot forge (would need CA's private key)
```

### 2.2 Trust Hierarchy

```
+------------------------------------------------------------------+
|                    TRUST HIERARCHY                                |
+------------------------------------------------------------------+

                    Root CA
                   (Trusted)
                       │
                       │ Signs
                       ▼
               Intermediate CA
                       │
                       │ Signs
                       ▼
                  Leaf Certificate
                  (bank.com)

Your browser trusts Root CA (pre-installed)
Root CA vouches for Intermediate CA
Intermediate CA vouches for bank.com
Therefore: Browser trusts bank.com!
```

---

## 3. Certificates Explained

### 3.1 What is a Certificate?

A digital certificate is a digitally signed document that:
1. Identifies an entity (person, website, organization)
2. Contains their public key
3. Is signed by a trusted authority

```
+------------------------------------------------------------------+
|                    CERTIFICATE STRUCTURE                          |
+------------------------------------------------------------------+

┌─────────────────────────────────────────────────────────────────┐
│                     DIGITAL CERTIFICATE                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  SUBJECT: CN=www.example.com, O=Example Inc, C=US               │
│  (Who this certificate is for)                                   │
│                                                                  │
│  ISSUER: CN=DigiCert, O=DigiCert Inc, C=US                     │
│  (Who issued/signed this certificate)                           │
│                                                                  │
│  PUBLIC KEY: RSA 2048-bit                                       │
│  30820122300d06092a864886f70d0101...                           │
│                                                                  │
│  VALIDITY:                                                       │
│    Not Before: Jan 1, 2024                                       │
│    Not After:  Dec 31, 2024                                      │
│                                                                  │
│  SIGNATURE: (Issuer's digital signature)                        │
│  4a3c2f1e8b9d7a6c5e4f3d2a1b0c9e8f...                           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 3.2 Certificate Types

```
Types by Validation Level:
─────────────────────────────────────────

DV (Domain Validation):
├── Proves: You control the domain
├── Verification: Email/DNS/HTTP challenge
├── Time: Minutes
├── Cost: Free (Let's Encrypt) to $
└── Visual: No org name in browser

OV (Organization Validation):
├── Proves: Organization legally exists
├── Verification: Business documents
├── Time: Days
├── Cost: $$
└── Visual: Org name visible in certificate

EV (Extended Validation):
├── Proves: Rigorous identity verification
├── Verification: In-depth vetting
├── Time: Weeks
├── Cost: $$$
└── Visual: Previously green bar (browsers removed this)
```

---

## 4. X.509 Certificate Structure

### 4.1 ASN.1 Structure

```
X.509 v3 Certificate Fields:
─────────────────────────────────────────

tbsCertificate (to be signed):
├── Version: 3 (v3)
├── Serial Number: Unique identifier
├── Signature Algorithm: e.g., sha256WithRSAEncryption
├── Issuer: CA's distinguished name
├── Validity:
│   ├── Not Before: Start date
│   └── Not After: Expiry date
├── Subject: Certificate owner's distinguished name
├── Subject Public Key Info:
│   ├── Algorithm: RSA, ECDSA, etc.
│   └── Public Key: The actual key
└── Extensions (v3):
    ├── Key Usage: digitalSignature, keyEncipherment
    ├── Extended Key Usage: serverAuth, clientAuth
    ├── Subject Alternative Name: DNS names, IPs
    ├── Basic Constraints: CA:TRUE/FALSE
    └── CRL Distribution Points: Where to check revocation

signatureAlgorithm: Algorithm used to sign
signatureValue: The actual signature
```

### 4.2 Distinguished Name (DN)

```
DN Format:
─────────────────────────────────────────

CN = Common Name (domain or person's name)
O  = Organization
OU = Organizational Unit
L  = Locality (city)
ST = State/Province
C  = Country (2-letter code)

Example:
CN=www.google.com, O=Google LLC, L=Mountain View, ST=California, C=US
```

### 4.3 View Certificate with OpenSSL

```bash
# Download and view certificate
echo | openssl s_client -connect google.com:443 2>/dev/null | \
    openssl x509 -text -noout

# Key fields to look for:
# - Issuer
# - Subject
# - Validity dates
# - Subject Alternative Names
# - Public key algorithm and size
```

---

## 5. Certificate Authorities

### 5.1 What is a CA?

A **Certificate Authority** is a trusted entity that:
- Verifies identity of certificate requesters
- Issues signed certificates
- Maintains revocation lists

```
+------------------------------------------------------------------+
|                    CA RESPONSIBILITIES                            |
+------------------------------------------------------------------+

1. IDENTITY VERIFICATION
   ├── DV: Verify domain control
   ├── OV: Verify organization exists
   └── EV: Extensive verification

2. CERTIFICATE ISSUANCE
   ├── Sign certificate with CA private key
   └── Include appropriate extensions

3. REVOCATION MANAGEMENT
   ├── Revoke compromised certificates
   ├── Publish CRLs (Certificate Revocation Lists)
   └── Run OCSP responders

4. SECURITY
   ├── Protect CA private key (HSM)
   └── Audit and compliance
```

### 5.2 Types of CAs

```
Root CA:
├── Self-signed certificate
├── Pre-installed in browsers/OS
├── Signs intermediate CAs
└── Key kept OFFLINE for security

Intermediate CA:
├── Signed by Root CA
├── Issues end-entity certificates
├── Key can be online (less risk)
└── If compromised, only it is revoked (not root)

Why Intermediates?
├── Root CA key is extremely valuable
├── If compromised, EVERYTHING is compromised
├── Intermediate provides isolation
└── Common practice: Root offline, intermediate online
```

### 5.3 Major CAs

```
Public CAs:
─────────────────────────────────────────
• Let's Encrypt (free, automated)
• DigiCert
• Sectigo (formerly Comodo)
• GlobalSign
• GoDaddy

Browser Trust Stores:
─────────────────────────────────────────
• ~150+ Root CAs trusted by browsers
• Each browser/OS maintains their own list
• Removal = CA's certificates no longer trusted
```

---

## 6. Chain of Trust

### 6.1 How the Chain Works

```
+------------------------------------------------------------------+
|                    CERTIFICATE CHAIN                              |
+------------------------------------------------------------------+

            ┌─────────────────────────────────────────┐
            │         ROOT CA CERTIFICATE              │
            │  Issuer: Self (DigiCert Root)           │
            │  Subject: DigiCert Root                 │
            │  Self-signed: Yes                        │
            │  In browser trust store: YES            │
            └──────────────────┬──────────────────────┘
                               │ Signs
                               ▼
            ┌─────────────────────────────────────────┐
            │     INTERMEDIATE CA CERTIFICATE         │
            │  Issuer: DigiCert Root                  │
            │  Subject: DigiCert SHA2 CA              │
            │  Signed by: Root CA                     │
            └──────────────────┬──────────────────────┘
                               │ Signs
                               ▼
            ┌─────────────────────────────────────────┐
            │       LEAF CERTIFICATE                  │
            │  Issuer: DigiCert SHA2 CA              │
            │  Subject: www.example.com               │
            │  Signed by: Intermediate CA             │
            └─────────────────────────────────────────┘

Browser Verification:
1. Receive leaf certificate
2. Find intermediate (sent by server or fetched)
3. Verify intermediate's signature on leaf
4. Find root (in trust store)
5. Verify root's signature on intermediate
6. Root is trusted → Chain is valid!
```

### 6.2 Chain Verification in Python

```python
from cryptography import x509
from cryptography.hazmat.backends import default_backend

def verify_chain_signatures(chain):
    """Verify signatures in certificate chain"""
    for i in range(len(chain) - 1):
        cert = chain[i]
        issuer = chain[i + 1]
        
        # Get issuer's public key
        issuer_public_key = issuer.public_key()
        
        # Verify signature
        try:
            issuer_public_key.verify(
                cert.signature,
                cert.tbs_certificate_bytes,
                cert.signature_algorithm_parameters
            )
            print(f"✓ {cert.subject.rfc4514_string()} signed by {issuer.subject.rfc4514_string()}")
        except Exception as e:
            print(f"✗ Signature verification failed: {e}")
            return False
    
    return True
```

---

## 7. TLS Overview

### 7.1 What is TLS?

**TLS** (Transport Layer Security) provides:
- **Confidentiality**: Encryption
- **Integrity**: Tamper detection
- **Authentication**: Identity verification

```
+------------------------------------------------------------------+
|                    TLS PROTOCOL STACK                             |
+------------------------------------------------------------------+

┌──────────────────────────────────────────────────────────────────┐
│                     APPLICATION (HTTP)                            │
├──────────────────────────────────────────────────────────────────┤
│                          TLS                                      │
│  ┌──────────────┬──────────────┬───────────────────────────────┐ │
│  │  Handshake   │    Alert     │        Record Protocol        │ │
│  │  Protocol    │   Protocol   │   (Encryption/MAC/Compress)   │ │
│  └──────────────┴──────────────┴───────────────────────────────┘ │
├──────────────────────────────────────────────────────────────────┤
│                          TCP                                      │
├──────────────────────────────────────────────────────────────────┤
│                          IP                                       │
└──────────────────────────────────────────────────────────────────┘
```

### 7.2 TLS Versions

```
Version History:
─────────────────────────────────────────

SSL 2.0 (1995): BROKEN - Never use
SSL 3.0 (1996): BROKEN - POODLE attack
TLS 1.0 (1999): DEPRECATED - BEAST attack
TLS 1.1 (2006): DEPRECATED
TLS 1.2 (2008): CURRENT - Still widely used
TLS 1.3 (2018): RECOMMENDED - Modern, faster, more secure

Use: TLS 1.2 or TLS 1.3 only!
```

---

## 8. TLS 1.2 Handshake

### 8.1 Full Handshake (2-RTT)

```
+------------------------------------------------------------------+
|                    TLS 1.2 HANDSHAKE                              |
+------------------------------------------------------------------+

Client                                              Server
   │                                                   │
   │──── ClientHello ─────────────────────────────────>│ RTT 1
   │     (TLS version, cipher suites, random)          │
   │                                                   │
   │<─── ServerHello ─────────────────────────────────│
   │     (Selected cipher, random)                     │
   │                                                   │
   │<─── Certificate ─────────────────────────────────│
   │     (Server's certificate chain)                  │
   │                                                   │
   │<─── ServerKeyExchange (if DHE/ECDHE) ────────────│
   │     (DH parameters, signed)                       │
   │                                                   │
   │<─── ServerHelloDone ─────────────────────────────│
   │                                                   │
   │──── ClientKeyExchange ────────────────────────────>│ RTT 2
   │     (Premaster secret or DH public)               │
   │                                                   │
   │──── [ChangeCipherSpec] ───────────────────────────>│
   │                                                   │
   │──── Finished ─────────────────────────────────────>│
   │     (Encrypted with new keys)                     │
   │                                                   │
   │<─── [ChangeCipherSpec] ───────────────────────────│
   │                                                   │
   │<─── Finished ─────────────────────────────────────│
   │                                                   │
   │<═══════════ Encrypted Application Data ══════════>│
```

### 8.2 Cipher Suite Format

```
Cipher Suite Example:
TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
│    │     │        │   │   │    │
│    │     │        │   │   │    └── PRF Hash (for key derivation)
│    │     │        │   │   └─────── Block size
│    │     │        │   └─────────── Mode (GCM = authenticated)
│    │     │        └─────────────── Bulk cipher (AES)
│    │     └──────────────────────── Authentication (RSA cert)
│    └────────────────────────────── Key Exchange (ECDHE)
└─────────────────────────────────── Protocol

Components:
- Key Exchange: ECDHE, DHE, RSA
- Authentication: RSA, ECDSA
- Encryption: AES-128, AES-256, ChaCha20
- Mode: GCM, CCM, CBC (avoid CBC!)
- Hash: SHA256, SHA384
```

---

## 9. TLS 1.3 Handshake

### 9.1 Improvements Over TLS 1.2

```
TLS 1.3 Changes:
─────────────────────────────────────────

REMOVED (insecure):
├── RSA key exchange (no forward secrecy)
├── CBC mode ciphers (padding oracles)
├── RC4, 3DES, MD5, SHA-1
├── Static DH
└── Compression

ADDED:
├── Mandatory forward secrecy (ECDHE only)
├── 1-RTT handshake (faster!)
├── 0-RTT resumption (even faster, but risky)
├── Encrypted handshake (SNI still visible though)
└── Simpler cipher suites
```

### 9.2 TLS 1.3 Handshake (1-RTT)

```
+------------------------------------------------------------------+
|                    TLS 1.3 HANDSHAKE                              |
+------------------------------------------------------------------+

Client                                              Server
   │                                                   │
   │──── ClientHello ─────────────────────────────────>│
   │     + key_share (ECDHE public key)               │
   │     + supported_versions                          │
   │                                                   │
   │<─── ServerHello ─────────────────────────────────│ RTT 1
   │     + key_share (ECDHE public key)               │
   │                                                   │
   │<─── {EncryptedExtensions} ───────────────────────│
   │     {Certificate}                                 │
   │     {CertificateVerify}                          │
   │     {Finished}                                    │
   │                                                   │
   │──── {Finished} ───────────────────────────────────>│
   │                                                   │
   │<═══════════ Encrypted Application Data ══════════>│

{ } = Encrypted with handshake keys

Note: Only 1 round trip needed!
(TLS 1.2 needed 2 round trips)
```

### 9.3 TLS 1.3 Cipher Suites

```
TLS 1.3 Cipher Suites (simplified format):
─────────────────────────────────────────

TLS_AES_256_GCM_SHA384
TLS_AES_128_GCM_SHA256
TLS_CHACHA20_POLY1305_SHA256
TLS_AES_128_CCM_SHA256

Key exchange is ALWAYS ECDHE (not in cipher suite name)
Authentication comes from certificate type
```

---

## 10. Certificate Verification

### 10.1 What Browsers Check

```
Browser Certificate Verification:
─────────────────────────────────────────

1. CHAIN VALIDATION
   ├── Build chain to trusted root
   ├── Verify each signature
   └── Check each cert's validity period

2. HOSTNAME VERIFICATION
   ├── Certificate subject matches requested host
   ├── Check CN (Common Name) - legacy
   └── Check SAN (Subject Alternative Name) - preferred

3. VALIDITY PERIOD
   ├── Not before: Certificate not yet valid
   └── Not after: Certificate expired

4. KEY USAGE
   ├── Certificate allows TLS server authentication
   └── Extended Key Usage includes serverAuth

5. REVOCATION CHECK
   ├── CRL (Certificate Revocation List)
   ├── OCSP (Online Certificate Status Protocol)
   └── OCSP Stapling (server provides status)

6. POLICY CHECKS
   ├── Minimum key sizes
   └── Signature algorithms
```

### 10.2 Common Verification Failures

```
Certificate Errors:
─────────────────────────────────────────

NET::ERR_CERT_DATE_INVALID
└── Certificate expired or not yet valid

NET::ERR_CERT_COMMON_NAME_INVALID
└── Hostname doesn't match certificate

NET::ERR_CERT_AUTHORITY_INVALID
└── Issued by untrusted CA (self-signed, unknown CA)

NET::ERR_CERT_REVOKED
└── Certificate has been revoked

NET::ERR_CERTIFICATE_TRANSPARENCY_REQUIRED
└── Missing CT logs (for EV certs)
```

---

## 11. Certificate Revocation

### 11.1 Why Revocation?

```
Reasons to Revoke:
─────────────────────────────────────────
• Private key compromised
• CA was compromised
• Certificate information changed
• Certificate issued incorrectly
• Domain control lost
```

### 11.2 CRL vs OCSP

```
CRL (Certificate Revocation List):
─────────────────────────────────────────
• CA publishes list of revoked serial numbers
• Client downloads entire list
• Problems: Large, slow updates, privacy

OCSP (Online Certificate Status Protocol):
─────────────────────────────────────────
• Client queries CA for specific certificate
• Real-time response: Good/Revoked/Unknown
• Problems: Privacy leak, availability

OCSP Stapling:
─────────────────────────────────────────
• Server fetches its own OCSP response
• Sends response during TLS handshake
• Best of both: Fresh, no privacy leak, fast
```

### 11.3 OCSP Query

```bash
# Get OCSP responder URL from certificate
openssl x509 -in cert.pem -noout -ocsp_uri

# Query OCSP
openssl ocsp -issuer issuer.pem -cert cert.pem \
    -url http://ocsp.example.com -resp_text
```

---

## 12. Practical Certificate Management

### 12.1 Generate Self-Signed Certificate

```bash
# Generate private key
openssl genrsa -out server.key 2048

# Generate self-signed certificate
openssl req -new -x509 -days 365 \
    -key server.key \
    -out server.crt \
    -subj "/CN=localhost/O=Test/C=US"

# View certificate
openssl x509 -in server.crt -text -noout
```

### 12.2 Generate CSR (Certificate Signing Request)

```bash
# Generate key and CSR
openssl req -newkey rsa:2048 -nodes \
    -keyout server.key \
    -out server.csr \
    -subj "/CN=example.com/O=Example Inc/C=US"

# With Subject Alternative Names
openssl req -newkey rsa:2048 -nodes \
    -keyout server.key \
    -out server.csr \
    -subj "/CN=example.com" \
    -addext "subjectAltName=DNS:example.com,DNS:www.example.com"

# View CSR
openssl req -in server.csr -text -noout
```

### 12.3 Create Your Own CA

```bash
# 1. Create CA private key
openssl genrsa -aes256 -out ca.key 4096

# 2. Create CA certificate (self-signed)
openssl req -new -x509 -days 3650 \
    -key ca.key \
    -out ca.crt \
    -subj "/CN=My CA/O=My Organization/C=US"

# 3. Sign a certificate with your CA
openssl x509 -req -days 365 \
    -in server.csr \
    -CA ca.crt \
    -CAkey ca.key \
    -CAcreateserial \
    -out server.crt
```

### 12.4 Let's Encrypt (Free Certificates)

```bash
# Using certbot
sudo apt install certbot

# Get certificate
sudo certbot certonly --standalone -d example.com

# Certificates stored in:
# /etc/letsencrypt/live/example.com/fullchain.pem
# /etc/letsencrypt/live/example.com/privkey.pem

# Auto-renew (add to cron)
certbot renew
```

---

## 13. Attacks and Defenses

### 13.1 Man-in-the-Middle (MitM)

```
Attack:
─────────────────────────────────────────
1. Attacker intercepts connection
2. Presents fake certificate to client
3. Establishes separate connection to real server
4. Decrypts, reads, modifies traffic

Defense:
─────────────────────────────────────────
• Certificate verification (browser does this)
• Certificate Pinning (app hardcodes expected cert)
• HSTS (HTTP Strict Transport Security)
```

### 13.2 Certificate Pinning Bypass

```
For Penetration Testing:
─────────────────────────────────────────

On Android:
• Frida + objection
• Xposed Framework
• Modify APK

On iOS:
• Frida + SSL Kill Switch
• Jailbreak + Cydia

Tools:
• Burp Suite with installed CA
• mitmproxy
• Frida scripts
```

### 13.3 SSL Stripping

```
Attack:
─────────────────────────────────────────
1. Attacker intercepts HTTP request
2. Attacker connects to server via HTTPS
3. Attacker sends HTTP (unencrypted) to client
4. Client thinks connection is secure

Defense:
─────────────────────────────────────────
• HSTS (Strict-Transport-Security header)
• Preload HSTS list
```

---

## 14. Hands-On Labs

### Lab 1: Analyze a Certificate Chain

```bash
# Connect and get certificate chain
echo | openssl s_client -connect google.com:443 -showcerts 2>/dev/null

# Save leaf certificate
echo | openssl s_client -connect google.com:443 2>/dev/null | \
    openssl x509 > google.pem

# View certificate details
openssl x509 -in google.pem -text -noout | head -50

# Check specific fields
openssl x509 -in google.pem -noout -subject -issuer -dates
```

### Lab 2: TLS Handshake Analysis

```bash
# Capture TLS handshake
openssl s_client -connect google.com:443 -msg

# Check supported protocols
openssl s_client -connect google.com:443 -tls1_2
openssl s_client -connect google.com:443 -tls1_3

# Check cipher suites
nmap --script ssl-enum-ciphers -p 443 google.com
```

### Lab 3: Create Certificate Chain

```bash
# Create Root CA
openssl genrsa -out root.key 4096
openssl req -new -x509 -days 3650 -key root.key -out root.crt \
    -subj "/CN=Test Root CA"

# Create Intermediate CA
openssl genrsa -out intermediate.key 4096
openssl req -new -key intermediate.key -out intermediate.csr \
    -subj "/CN=Test Intermediate CA"
openssl x509 -req -days 1825 -in intermediate.csr \
    -CA root.crt -CAkey root.key -CAcreateserial \
    -out intermediate.crt \
    -extfile <(echo "basicConstraints=CA:TRUE")

# Create End Entity Certificate
openssl genrsa -out server.key 2048
openssl req -new -key server.key -out server.csr \
    -subj "/CN=test.example.com"
openssl x509 -req -days 365 -in server.csr \
    -CA intermediate.crt -CAkey intermediate.key -CAcreateserial \
    -out server.crt

# Verify chain
openssl verify -CAfile root.crt -untrusted intermediate.crt server.crt
```

---

## 15. Interview Questions

**Q1: What is a digital certificate and what does it contain?**
> A digital certificate is a digitally signed document that binds a public key to an identity. It contains: subject (identity), issuer (CA), public key, validity period, extensions (key usage, SAN), and the CA's digital signature.

**Q2: Explain the chain of trust in PKI.**
> The chain of trust is a hierarchy where each certificate is signed by the one above it. Root CA (trusted by browsers) signs Intermediate CA, which signs end-entity certificates. Verification traces back to a trusted root.

**Q3: What is the difference between TLS 1.2 and TLS 1.3?**
> TLS 1.3 is faster (1-RTT vs 2-RTT), removes insecure options (RSA key exchange, CBC mode), mandates forward secrecy (ECDHE only), and encrypts more of the handshake. It has fewer cipher suites, all secure by design.

**Q4: What is OCSP stapling and why is it better?**
> OCSP stapling has the server fetch its own OCSP response and send it during TLS handshake. It's better because: no privacy leak (client doesn't query CA), faster (no extra connection), more reliable (server can cache).

**Q5: How would you detect a MitM attack using a fake certificate?**
> Check certificate: Is it issued by trusted CA? Does hostname match? Is it valid? Is it revoked? For apps: certificate pinning compares against expected cert. Users: Look for browser warnings, check certificate in browser.

---

## 16. Quick Reference

### OpenSSL Commands

```bash
# View certificate
openssl x509 -in cert.pem -text -noout

# Verify certificate chain
openssl verify -CAfile ca.crt -untrusted intermediate.crt server.crt

# Generate key and CSR
openssl req -newkey rsa:2048 -nodes -keyout key.pem -out csr.pem

# Test TLS connection
openssl s_client -connect host:443

# Check certificate expiry
openssl x509 -enddate -noout -in cert.pem
```

### Certificate Fields

| Field | Purpose |
|-------|---------|
| Subject | Identity (CN, O, C) |
| Issuer | Who signed it |
| Validity | Start and end dates |
| Public Key | The actual key |
| SAN | Alternative names |
| Key Usage | What key can be used for |

### TLS Cipher Suite Components

| Component | Options |
|-----------|---------|
| Key Exchange | ECDHE, DHE |
| Authentication | RSA, ECDSA |
| Encryption | AES-GCM, ChaCha20 |
| Hash | SHA256, SHA384 |

---

## Next Steps

Continue to:
- **[[07_Password_Cracking]]** - Breaking password hashes
- **[[10_SSH_and_Key_Management]]** - SSH certificates and keys

---

## References

- [[05_Asymmetric_Encryption]]
- [RFC 8446 - TLS 1.3](https://tools.ietf.org/html/rfc8446)
- [SSL Labs](https://www.ssllabs.com/ssltest/)
- [Let's Encrypt Documentation](https://letsencrypt.org/docs/)
