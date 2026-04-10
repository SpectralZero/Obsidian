
# Web Reconnaissance & Fuzzing: Mapping the Unknown

## Table of Contents

1. [Executive Summary & OSCP Relevance](#1-executive-summary--oscp-relevance)
2. [ABSOLUTE BEGINNER: Reconnaissance Fundamentals](#2-absolute-beginner-reconnaissance-fundamentals)
3. [Passive Reconnaissance Techniques](#3-passive-reconnaissance-techniques)
4. [Active Reconnaissance Techniques](#4-active-reconnaissance-techniques)
5. [Directory & File Discovery](#5-directory--file-discovery)
6. [Subdomain Enumeration](#6-subdomain-enumeration)
7. [Virtual Host (VHost) Discovery](#7-virtual-host-vhost-discovery)
8. [Parameter Fuzzing](#8-parameter-fuzzing)
9. [FFUF Mastery](#9-ffuf-mastery)
10. [Gobuster Complete Guide](#10-gobuster-complete-guide)
11. [Wordlist Selection & Creation](#11-wordlist-selection--creation)
12. [Extension Fuzzing](#12-extension-fuzzing)
13. [Recursive Scanning](#13-recursive-scanning)
14. [API Discovery & Fuzzing](#14-api-discovery--fuzzing)
15. [Bypass Techniques (403, 401, Rate Limits)](#15-bypass-techniques-403-401-rate-limits)
16. [WAF Detection & Evasion](#16-waf-detection--evasion)
17. [JavaScript Analysis](#17-javascript-analysis)
18. [Google Dorking & OSINT](#18-google-dorking--osint)
19. [Automation & Tool Chains](#19-automation--tool-chains)
20. [OSCP Attack Scenarios](#20-oscp-attack-scenarios)
21. [Time Management for OSCP](#21-time-management-for-oscp)
22. [Common Pitfalls](#22-common-pitfalls)
23. [Hands-on Exercises](#23-hands-on-exercises)
24. [Interview Questions](#24-interview-questions)
25. [Summary & Key Takeaways](#25-summary--key-takeaways)
26. [References & Further Reading](#26-references--further-reading)

---

## 1. Executive Summary & OSCP Relevance

### What is Web Reconnaissance & Fuzzing?

**Web Reconnaissance**: The process of discovering and mapping web assets (domains, subdomains, directories, files, parameters) to expand the attack surface.

**Fuzzing**: Automated testing by sending thousands of inputs (wordlists) to discover hidden resources.

**The Golden Rule**: "You can't hack what you can't see."

### OSCP Relevance (CRITICAL - 100%)

Web reconnaissance is **mandatory** on every OSCP web machine. You MUST:
- Discover hidden directories (`/admin`, `/backup`, `/dev`)
- Find configuration files (`.bak`, `.config`, `.old`)
- Enumerate subdomains and virtual hosts
- Locate API endpoints (`/api/v1`, `/api/v2`)

**Time Allocation**: 30-60 minutes per web application for thorough recon

**Common OSCP Patterns**:
```
1. Directory fuzzing finds /backup
2. /backup contains backup.zip
3. Download zip → source code leak
4. Source code reveals credentials or vulnerability
5. Use credentials → Foothold
```

### Why Reconnaissance Matters

```
Without Recon:
Website homepage → Login form → Stuck
↓
Attack Surface: 1 endpoint

With Recon:
Website → /admin, /api, /dev, /backup.zip, dev.site.com
↓
Attack Surface: 100+ endpoints
↓
Multiple attack vectors
```

**Statistics**:
- 73% of successful penetration tests start with thorough recon (industry surveys)
- Average: 5,000 directories/files tested per web application
- Discovery rate: 1-5% (50-250 valid resources from 5,000 tests)

---

## 2. ABSOLUTE BEGINNER: Reconnaissance Fundamentals

### The Hierarchy of Discovery

```
Level 1: Domain
  └── example.com

Level 2: Subdomains
  ├── www.example.com
  ├── api.example.com
  ├── dev.example.com
  └── admin.example.com

Level 3: Directories
  ├── /admin
  ├── /api
  ├── /uploads
  └── /backup

Level 4: Files
  ├── /admin/config.php
  ├── /backup/backup.zip
  └── /api/v1/users

Level 5: Parameters
  └── /api/v1/users?id=1&debug=true
```

### Simple Example

**Starting Point**: `http://target.com`

**After Recon**:
```
Discovered:
- http://target.com/admin (403 Forbidden)
- http://target.com/backup.zip (200 OK) ← Download!
- http://dev.target.com (Internal dev site)
- http://target.com/api/v1/users (API endpoint)
- http://target.com/login?debug=1 (Debug mode)
```

### ASCII Visualization

```
┌──────────────────────────────────────────────────────┐
│           Web Reconnaissance Process                 │
└──────────────────────────────────────────────────────┘

Phase 1: Passive Recon (No direct contact)
┌────────────────────────────────────────┐
│ Google Dorking                         │
│ Certificate Transparency Logs (crt.sh) │
│ Wayback Machine                        │
│ GitHub Code Search                     │
│ Shodan / Censys                        │
└────────────────────────────────────────┘
         ↓
    Discover:
    - Subdomains (dev.site.com)
    - Old endpoints (/v1/api)
    - Leaked credentials

Phase 2: Active Recon (Direct testing)
┌────────────────────────────────────────┐
│ Directory Fuzzing (ffuf, gobuster)    │
│ Subdomain Brute Force                 │
│ VHost Discovery                       │
│ Parameter Fuzzing                     │
└────────────────────────────────────────┘
         ↓
    Discover:
    - /admin, /backup
    - Hidden subdomains
    - Debug parameters

Phase 3: Exploitation
┌────────────────────────────────────────┐
│ Access discovered resources            │
│ Download backup files                  │
│ Test discovered parameters             │
└────────────────────────────────────────┘
         ↓
    Result: Expanded attack surface
```

---

## 3. Passive Reconnaissance Techniques

### 3.1 Certificate Transparency Logs

**What is CT?**
- Public logs of all SSL/TLS certificates issued
- Every certificate includes Subject Alternative Names (SANs) = subdomains
- No packets sent to target (100% passive)

**Tools**:

**A. crt.sh (Website)**:
```
https://crt.sh/?q=%25.target.com
```

**B. crt.sh (API)**:
```bash
curl -s "https://crt.sh/?q=%25.target.com&output=json" \
  | jq -r '.[].name_value' \
  | sort -u

# Output:
# admin.target.com
# api.target.com
# dev.target.com
# mail.target.com
# vpn.target.com
```

**C. crt.sh (Tool - CertSpotter)**:
```bash
# Install
go install github.com/sensepost/certspotter@latest

# Run
certspotter target.com

# Output: List of subdomains from CT logs
```

### 3.2 Google Dorking

**Powerful Search Operators**:

**A. Site-Specific Search**:
```
site:target.com
site:target.com -www  (Exclude www)
```

**B. File Types**:
```
site:target.com filetype:pdf
site:target.com filetype:docx
site:target.com filetype:xlsx
site:target.com filetype:sql  (Database dumps!)
site:target.com filetype:env  (Environment variables!)
```

**C. Directory Listings**:
```
site:target.com intitle:"index of"
site:target.com intitle:"directory listing"
```

**D. Login Pages**:
```
site:target.com inurl:login
site:target.com inurl:admin
site:target.com inurl:dashboard
```

**E. Exposed Files**:
```
site:target.com inurl:.git
site:target.com inurl:wp-config
site:target.com ext:log
site:target.com ext:bak
```

**F. Error Messages**:
```
site:target.com intext:"sql syntax"
site:target.com intext:"mysql error"
site:target.com intext:"stack trace"
```

**Complete Google Dork Examples**:
```bash
# Database dumps
site:target.com ext:sql

# Configuration files
site:target.com ext:xml | ext:conf | ext:cnf | ext:reg | ext:inf | ext:rdp | ext:cfg | ext:txt | ext:ora | ext:ini

# Backup files
site:target.com ext:bkf | ext:bkp | ext:bak | ext:old | ext:backup

# Login portals
site:target.com inurl:login | inurl:signin | inurl:admin | inurl:dashboard

# GitHub repos (code leaks)
site:github.com "target.com" password
site:github.com "target.com" api_key
```

### 3.3 Wayback Machine

**What is it?**
- Archive.org's web crawler
- Historical snapshots of websites
- Reveals old/deleted pages

**Manual Search**:
```
https://web.archive.org/web/*/target.com
```

**Tool - waybackurls**:
```bash
# Install
go install github.com/tomnomnom/waybackurls@latest

# Run
echo "target.com" | waybackurls

# Output:
# http://target.com/old-admin
# http://target.com/v1/api/users
# http://target.com/backup/db.sql
```

**Filter for Interesting URLs**:
```bash
echo "target.com" | waybackurls | grep -E "\.js|\.json|\.xml|\.bak|\.zip"
```

### 3.4 GitHub & Code Search

**GitHub Dorking**:
```
# Search for organization
org:targetcorp password

# Search for domain
"target.com" api_key

# Search for specific files
filename:.env target.com

# Search for AWS keys
org:targetcorp AKIA
```

**Tools**:

**A. truffleHog (Find secrets in Git)**:
```bash
# Install
pip3 install truffleHog

# Scan repo
truffleHog https://github.com/targetcorp/repo
```

**B. GitLeaks**:
```bash
# Install
go install github.com/zricethezav/gitleaks/v8@latest

# Scan
gitleaks detect --source https://github.com/targetcorp/repo
```

### 3.5 Shodan & Censys

**Shodan Search**:
```
# Find servers for target.com
hostname:target.com

# Find specific services
hostname:target.com port:8080

# Find specific technology
http.title:"Admin Panel" hostname:target.com
```

**Censys Search**:
```
services.http.response.html_title="Admin" AND target.com
```

---

## 4. Active Reconnaissance Techniques

### 4.1 DNS Enumeration

**Forward DNS Lookup**:
```bash
# Single lookup
nslookup target.com

# Multiple lookups
dig target.com
dig @8.8.8.8 target.com
```

**Reverse DNS Lookup**:
```bash
# Find hostname from IP
dig -x 192.168.1.1
```

**Zone Transfer (Rare, but try)**:
```bash
# Find nameservers
dig ns target.com

# Attempt zone transfer
dig axfr @ns1.target.com target.com

# If successful: Full DNS dump!
```

### 4.2 DNS Brute Force (Subdomains)

**dnsenum**:
```bash
dnsenum --enum target.com
```

**dnsrecon**:
```bash
dnsrecon -d target.com -t brt -D /usr/share/wordlists/subdomains.txt
```

### 4.3 Web Crawling

**Burp Suite Spider**:
```
1. Browse the site normally
2. Burp → Target → Site map
3. Right-click domain → Spider this host
4. Analyze discovered URLs
```

**Gospider**:
```bash
gospider -s https://target.com -d 3 -c 10
```

**Katana (New, Fast)**:
```bash
katana -u https://target.com -d 5 -jc
```

---

## 5. Directory & File Discovery

### 5.1 Basic Directory Fuzzing

**FFUF**:
```bash
ffuf -u http://target.com/FUZZ \
     -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt \
     -fc 404

# Flags:
# -u: URL with FUZZ keyword
# -w: Wordlist
# -fc: Filter by status code (exclude 404)
```

**Gobuster**:
```bash
gobuster dir -u http://target.com \
             -w /usr/share/wordlists/dirb/common.txt \
             -x php,html,txt
```

### 5.2 Comprehensive Wordlists

**SecLists (Essential)**:
```bash
# Install
sudo apt install seclists

# Location
/usr/share/seclists/Discovery/Web-Content/

# Common wordlists:
common.txt                          (4,613 entries)
raft-medium-directories.txt         (30,000 entries)
raft-large-directories.txt          (62,284 entries)
directory-list-2.3-medium.txt       (220,560 entries)
```

**Size Recommendations**:
```
Quick scan (5 min):     common.txt
Standard scan (30 min): raft-medium-directories.txt
Deep scan (2+ hours):   directory-list-2.3-medium.txt
```

### 5.3 Filter Techniques

**Problem**: Too many false positives (404 pages with custom content).

**Solution**: Filter by size, status, or words.

**A. Filter by Status Code**:
```bash
ffuf -u http://target.com/FUZZ -w wordlist.txt -fc 404,403
```

**B. Filter by Response Size**:
```bash
# Step 1: Test with non-existent path
curl -s http://target.com/nonexistent | wc -c
# Output: 1234 bytes (this is the 404 page size)

# Step 2: Filter that size
ffuf -u http://target.com/FUZZ -w wordlist.txt -fs 1234
```

**C. Filter by Words**:
```bash
ffuf -u http://target.com/FUZZ -w wordlist.txt -fw 100
# -fw: Filter by word count
```

**D. Match Specific Codes**:
```bash
ffuf -u http://target.com/FUZZ -w wordlist.txt -mc 200,301,302
# -mc: Match only these codes
```

### 5.4 Extension Fuzzing

**Common Web Extensions**:
```
PHP:        .php, .php3, .php4, .php5, .phtml
ASP:        .asp, .aspx
JSP:        .jsp, .jspx
Backup:     .bak, .old, .backup, .~, .swp
Config:     .config, .conf, .cfg, .ini
Archive:    .zip, .tar.gz, .rar, .7z
Database:   .sql, .db, .sqlite
Source:     .java, .py, .rb, .go
```

**FFUF with Extensions**:
```bash
ffuf -u http://target.com/FUZZ \
     -w wordlist.txt \
     -e .php,.bak,.old,.zip,.config
```

**Gobuster with Extensions**:
```bash
gobuster dir -u http://target.com \
             -w wordlist.txt \
             -x php,bak,old,zip,config,txt
```

### 5.5 Finding Backup Files

**High-Value Targets**:
```
config.php.bak
backup.zip
backup.tar.gz
database.sql
db_backup.sql
site.bak
www.zip
```

**Automated Backup Discovery**:
```bash
# Create backup filename variations
for file in index config admin login; do
  for ext in .bak .old .backup .zip .tar.gz; do
    echo "$file$ext"
  done
done > backup_names.txt

# Fuzz
ffuf -u http://target.com/FUZZ -w backup_names.txt -mc 200
```

---

## 6. Subdomain Enumeration

### 6.1 Why Subdomains Matter

**Attack Surface Expansion**:
```
Main site:  target.com       (Hardened, WAF protected)
Subdomain:  dev.target.com   (Forgotten, outdated software)
            api.target.com   (Exposed API, weak auth)
            admin.target.com (Admin panel, default creds)
```

### 6.2 Subdomain Brute Force

**FFUF**:
```bash
ffuf -u http://FUZZ.target.com \
     -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt \
     -mc 200
```

**Gobuster DNS**:
```bash
gobuster dns -d target.com \
             -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt \
             -t 50
```

**Amass (Comprehensive)**:
```bash
# Install
sudo apt install amass

# Passive enumeration
amass enum -passive -d target.com

# Active enumeration
amass enum -active -d target.com -o subdomains.txt
```

**Subfinder (Fast)**:
```bash
# Install
go install github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest

# Run
subfinder -d target.com -o subdomains.txt
```

### 6.3 Subdomain Takeover

**What is it?**
- Subdomain points to external service (AWS S3, GitHub Pages)
- Service account is deleted
- Subdomain still resolves but points to nothing
- Attacker claims the service → Controls subdomain

**Check for Takeover**:
```bash
# Install subjack
go install github.com/haccer/subjack@latest

# Test
subjack -w subdomains.txt -t 100 -timeout 30 -o takeovers.txt
```

**Manual Check**:
```bash
# Check CNAME
dig subdomain.target.com CNAME

# If it points to:
# - *.s3.amazonaws.com (S3 bucket)
# - *.github.io (GitHub Pages)
# - *.herokuapp.com (Heroku)
# - *.azurewebsites.net (Azure)
# → Potential takeover
```

---

## 7. Virtual Host (VHost) Discovery

### 7.1 What are Virtual Hosts?

**Concept**: Multiple websites hosted on one IP address, differentiated by `Host` header.

**Example**:
```
IP: 192.168.1.100

Nginx config:
  server {
    server_name www.site.com;
    # Public site
  }
  
  server {
    server_name dev.site.com;
    # Dev site (NOT in public DNS)
  }
  
  server {
    server_name admin.site.com;
    # Admin panel (NOT in public DNS)
  }
```

**Attack**: Brute force `Host` header to discover hidden sites.

### 7.2 VHost Fuzzing with FFUF

```bash
# Step 1: Get baseline response
curl -s http://192.168.1.100 | wc -c
# Output: 5432 bytes (default page size)

# Step 2: Fuzz Host header
ffuf -u http://192.168.1.100 \
     -H "Host: FUZZ.site.com" \
     -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
     -fs 5432

# Results:
# dev.site.com       [Status: 200, Size: 8234]
# admin.site.com     [Status: 200, Size: 3421]
```

### 7.3 VHost Fuzzing with Gobuster

```bash
gobuster vhost -u http://192.168.1.100 \
               -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
               --domain site.com \
               --append-domain
```

### 7.4 Accessing Discovered VHosts

**Add to /etc/hosts**:
```bash
# Edit /etc/hosts
sudo nano /etc/hosts

# Add:
192.168.1.100  dev.site.com
192.168.1.100  admin.site.com

# Access
curl http://dev.site.com
curl http://admin.site.com
```

---

## 8. Parameter Fuzzing

### 8.1 What is Parameter Fuzzing?

**Concept**: Discover hidden GET/POST parameters that trigger different behavior.

**Example**:
```
Normal:     http://target.com/page
With debug: http://target.com/page?debug=1 → Shows SQL queries
With admin: http://target.com/page?admin=true → Bypasses auth
```

### 8.2 GET Parameter Fuzzing

**FFUF**:
```bash
ffuf -u "http://target.com/index.php?FUZZ=1" \
     -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt \
     -fs 1234
```

**Common Parameters to Test**:
```
debug
test
admin
dev
trace
log
verbose
show_errors
redirect
url
next
return
callback
```

### 8.3 POST Parameter Fuzzing

**FFUF**:
```bash
ffuf -u http://target.com/login \
     -X POST \
     -d "username=admin&password=admin&FUZZ=1" \
     -H "Content-Type: application/x-www-form-urlencoded" \
     -w param-names.txt \
     -fs 1234
```

### 8.4 Arjun (Parameter Discovery Tool)

```bash
# Install
pip3 install arjun

# Run
arjun -u http://target.com/page

# Output:
# [+] Found parameter: debug
# [+] Found parameter: admin
```

---

## 9. FFUF Mastery

### 9.1 Basic Syntax

```bash
ffuf -u URL -w WORDLIST [OPTIONS]
```

### 9.2 Multiple FUZZ Keywords

**A. Directory + Extension**:
```bash
ffuf -u http://target.com/FUZZDIR/FUZZFILE.FUZZEXT \
     -w directories.txt:FUZZDIR \
     -w files.txt:FUZZFILE \
     -w extensions.txt:FUZZEXT
```

**B. Subdomain + Path**:
```bash
ffuf -u http://FUZZSUB.target.com/FUZZPATH \
     -w subdomains.txt:FUZZSUB \
     -w paths.txt:FUZZPATH
```

### 9.3 Advanced Filtering

**A. Regex Filtering**:
```bash
ffuf -u http://target.com/FUZZ \
     -w wordlist.txt \
     -fr "error|404|not found"
```

**B. Multiple Filters**:
```bash
ffuf -u http://target.com/FUZZ \
     -w wordlist.txt \
     -fc 404,403 \
     -fs 1234,5678 \
     -fw 100
```

**C. Match Regex**:
```bash
ffuf -u http://target.com/FUZZ \
     -w wordlist.txt \
     -mr "admin|dashboard|success"
```

### 9.4 Rate Limiting

```bash
# Limit to 10 requests per second
ffuf -u http://target.com/FUZZ \
     -w wordlist.txt \
     -rate 10
```

### 9.5 Output Formats

```bash
# JSON output
ffuf -u http://target.com/FUZZ -w wordlist.txt -o results.json -of json

# CSV output
ffuf -u http://target.com/FUZZ -w wordlist.txt -o results.csv -of csv

# HTML output
ffuf -u http://target.com/FUZZ -w wordlist.txt -o results.html -of html
```

### 9.6 Authentication

**A. Cookies**:
```bash
ffuf -u http://target.com/FUZZ \
     -w wordlist.txt \
     -b "session=abc123; token=xyz789"
```

**B. Headers**:
```bash
ffuf -u http://target.com/FUZZ \
     -w wordlist.txt \
     -H "Authorization: Bearer TOKEN"
```

**C. Basic Auth**:
```bash
ffuf -u http://target.com/FUZZ \
     -w wordlist.txt \
     -H "Authorization: Basic $(echo -n 'user:pass' | base64)"
```

### 9.7 Recursion

```bash
ffuf -u http://target.com/FUZZ \
     -w wordlist.txt \
     -recursion \
     -recursion-depth 3
```

---

## 10. Gobuster Complete Guide

### 10.1 Directory Brute Force

```bash
gobuster dir -u http://target.com \
             -w /usr/share/wordlists/dirb/common.txt \
             -x php,html,txt \
             -t 50 \
             -b 404,403
```

**Flags**:
- `-u`: URL
- `-w`: Wordlist
- `-x`: Extensions
- `-t`: Threads
- `-b`: Exclude status codes

### 10.2 DNS Subdomain Enumeration

```bash
gobuster dns -d target.com \
             -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
             -t 50
```

### 10.3 VHost Discovery

```bash
gobuster vhost -u http://target.com \
               -w subdomains.txt \
               --domain target.com \
               --append-domain
```

### 10.4 S3 Bucket Enumeration

```bash
gobuster s3 -w bucket-names.txt
```

### 10.5 Authentication

```bash
# Cookies
gobuster dir -u http://target.com \
             -w wordlist.txt \
             -c "session=abc123"

# Headers
gobuster dir -u http://target.com \
             -w wordlist.txt \
             -H "Authorization: Bearer TOKEN"

# Username/Password
gobuster dir -u http://target.com \
             -w wordlist.txt \
             -U username \
             -P password
```

---

## 11. Wordlist Selection & Creation

### 11.1 SecLists Structure

```
/usr/share/seclists/Discovery/
├── DNS/
│   ├── subdomains-top1million-5000.txt
│   ├── subdomains-top1million-20000.txt
│   └── subdomains-top1million-110000.txt
├── Web-Content/
│   ├── common.txt                      (4.6K)
│   ├── raft-small-directories.txt      (17K)
│   ├── raft-medium-directories.txt     (30K)
│   ├── raft-large-directories.txt      (62K)
│   └── directory-list-2.3-medium.txt   (220K)
└── Infrastructure/
    ├── common-admin-logins.txt
    └── common-passwords.txt
```

### 11.2 Custom Wordlist Creation

**A. CeWL (Website-specific wordlist)**:
```bash
# Install
sudo apt install cewl

# Generate wordlist from website
cewl -d 2 -m 5 http://target.com -w custom.txt

# Flags:
# -d: Depth (how deep to crawl)
# -m: Minimum word length
# -w: Output file
```

**B. Combine Multiple Wordlists**:
```bash
cat wordlist1.txt wordlist2.txt wordlist3.txt | sort -u > combined.txt
```

**C. Generate Variations**:
```bash
# Install
git clone https://github.com/Josue87/gotator

# Generate permutations
gotator -sub subdomains.txt -perm permutations.txt -depth 3 -o output.txt
```

---

## 12. Extension Fuzzing

### 12.1 Common Extensions by Technology

**PHP Stack**:
```bash
ffuf -u http://target.com/FUZZ \
     -w wordlist.txt \
     -e .php,.php3,.php4,.php5,.php7,.phtml,.phar,.inc
```

**ASP.NET Stack**:
```bash
ffuf -u http://target.com/FUZZ \
     -w wordlist.txt \
     -e .asp,.aspx,.ashx,.asmx,.axd,.config
```

**Java Stack**:
```bash
ffuf -u http://target.com/FUZZ \
     -w wordlist.txt \
     -e .jsp,.jspx,.jsf,.do,.action
```

**Backup/Config Files**:
```bash
ffuf -u http://target.com/FUZZ \
     -w wordlist.txt \
     -e .bak,.old,.backup,.~,.swp,.zip,.tar.gz,.sql,.config
```

### 12.2 Extension Discovery Workflow

**Step 1: Identify Technology**:
```bash
# Check headers
curl -I http://target.com

# Response:
# Server: Apache/2.4.41 (Ubuntu)
# X-Powered-By: PHP/7.4.3
```

**Step 2: Use Appropriate Extensions**:
```bash
# PHP detected → Use PHP extensions
ffuf -u http://target.com/FUZZ -w wordlist.txt -e .php,.phtml,.php5
```

---

## 13. Recursive Scanning

### 13.1 When to Use Recursion

**Scenario**: Finding `/api` is just the start. Need to scan inside it.

```
/api        (found)
/api/v1     (recursive scan)
/api/v2     (recursive scan)
/api/users  (recursive scan)
/api/admin  (recursive scan)
```

### 13.2 FFUF Recursion

```bash
ffuf -u http://target.com/FUZZ \
     -w wordlist.txt \
     -recursion \
     -recursion-depth 3 \
     -fc 404
```

**Warning**: Can take VERY long. Set depth carefully.

### 13.3 Manual Recursive Workflow

```bash
# Step 1: Initial scan
ffuf -u http://target.com/FUZZ -w wordlist.txt -o level1.txt

# Step 2: Extract found directories
cat level1.txt | grep -oP '(?<=http://target.com/)[^/]+' > dirs.txt

# Step 3: Scan each directory
while read dir; do
  ffuf -u http://target.com/$dir/FUZZ -w wordlist.txt -o $dir-results.txt
done < dirs.txt
```

---

## 14. API Discovery & Fuzzing

### 14.1 API Endpoint Patterns

**Common Paths**:
```
/api
/api/v1
/api/v2
/api/v3
/rest
/graphql
/swagger
/api-docs
/docs
/openapi.json
```

### 14.2 API Fuzzing

**RESTful API**:
```bash
ffuf -u http://target.com/api/v1/FUZZ \
     -w /usr/share/seclists/Discovery/Web-Content/api/api-endpoints.txt

# Common endpoints:
# /users
# /auth
# /login
# /admin
# /config
```

**API Versioning**:
```bash
# Fuzz versions
ffuf -u http://target.com/api/FUZZ/users \
     -w <(seq 1 10 | sed 's/^/v/')

# Tests: /api/v1/users, /api/v2/users, ..., /api/v10/users
```

### 14.3 API Documentation Discovery

```bash
# Common doc locations
curl http://target.com/swagger.json
curl http://target.com/api-docs
curl http://target.com/openapi.json
curl http://target.com/docs/api
curl http://target.com/api/swagger-ui
```

---

## 15. Bypass Techniques (403, 401, Rate Limits)

### 15.1 403 Forbidden Bypass

**A. Path Manipulation**:
```bash
/admin          → 403
/admin/         → 200 (Trailing slash)
/admin/.        → 200
/admin;         → 200
/admin..        → 200
/./admin        → 200
/%2e/admin      → 200
/admin%2f       → 200
/admin%00       → 200 (Null byte)
```

**B. Header Manipulation**:
```bash
# Spoof localhost
curl http://target.com/admin -H "X-Forwarded-For: 127.0.0.1"
curl http://target.com/admin -H "X-Originating-IP: 127.0.0.1"
curl http://target.com/admin -H "X-Remote-IP: 127.0.0.1"
curl http://target.com/admin -H "X-Client-IP: 127.0.0.1"
```

**C. HTTP Method Bypass**:
```bash
GET /admin   → 403
POST /admin  → 200
HEAD /admin  → 200
OPTIONS /admin → 200
```

**D. Case Variation**:
```bash
/admin  → 403
/Admin  → 200
/ADMIN  → 200
/aDmIn  → 200
```

### 15.2 Rate Limit Evasion

**A. Slow Down**:
```bash
ffuf -u http://target.com/FUZZ -w wordlist.txt -rate 5
```

**B. Rotate User-Agent**:
```bash
# Create user-agent list
cat > user-agents.txt << EOF
Mozilla/5.0 (Windows NT 10.0; Win64; x64)
Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7)
Mozilla/5.0 (X11; Linux x86_64)
EOF

# Use with ffuf
ffuf -u http://target.com/FUZZ -w wordlist.txt -H "User-Agent: FUZZUA" -w user-agents.txt:FUZZUA
```

**C. IP Rotation (FireProx)**:
```bash
# Install
git clone https://github.com/ustayready/fireprox
cd fireprox
pip3 install -r requirements.txt

# Create proxy
python3 fire.py --access_key AWS_KEY --secret_access_key AWS_SECRET --region us-east-1 --command create --url https://target.com

# Use proxy URL for fuzzing (rotates IP via AWS API Gateway)
```

---

## 16. WAF Detection & Evasion

### 16.1 WAF Detection

**wafw00f**:
```bash
# Install
pip3 install wafw00f

# Detect WAF
wafw00f https://target.com

# Output: Cloudflare, ModSecurity, etc.
```

### 16.2 WAF Evasion Techniques

**A. Encoding**:
```bash
# URL encoding
/admin → /%61%64%6d%69%6e

# Double encoding
/admin → /%2561%2564%256d%2569%256e
```

**B. Case Manipulation**:
```bash
/admin → /AdMiN
```

**C. Unicode**:
```bash
/admin → /ad%E2%93%82in
```

**D. Slow Rate**:
```bash
ffuf -u http://target.com/FUZZ -w wordlist.txt -rate 2
```

---

## 17. JavaScript Analysis

### 17.1 Finding Endpoints in JS

**Extract URLs from JS**:
```bash
# Download all JS files
curl -s https://target.com | grep -oP 'src="[^"]+\.js"' | cut -d'"' -f2 > js-files.txt

# Download JS files
while read url; do
  curl -s "https://target.com$url" >> all.js
done < js-files.txt

# Extract endpoints
grep -oP '(?<="/)[a-zA-Z0-9_\-]+(?=")' all.js | sort -u
```

**LinkFinder (Tool)**:
```bash
# Install
git clone https://github.com/GerbenJavado/LinkFinder
cd LinkFinder
pip3 install -r requirements.txt

# Run
python3 linkfinder.py -i https://target.com -o results.html
```

### 17.2 Finding Secrets in JS

**SecretFinder**:
```bash
# Install
git clone https://github.com/m4ll0k/SecretFinder
cd SecretFinder
pip3 install -r requirements.txt

# Run
python3 SecretFinder.py -i https://target.com/app.js -o secrets.html
```

**Common Secrets**:
```
API keys: AIza, AKIA (AWS)
Tokens: ghp_, gho_ (GitHub)
URLs: https://api.internal.com
Credentials: password=, user=
```

---

## 18. Google Dorking & OSINT

### 18.1 Advanced Google Dorks

**Complete Dork List**:
```bash
# Subdomains
site:*.target.com

# Login portals
site:target.com inurl:login
site:target.com inurl:admin
site:target.com inurl:dashboard
site:target.com inurl:portal

# Exposed files
site:target.com ext:sql
site:target.com ext:env
site:target.com ext:log
site:target.com ext:bak
site:target.com filetype:config

# Error messages
site:target.com "sql syntax"
site:target.com "mysql"
site:target.com "stack trace"
site:target.com "fatal error"

# Directory listings
site:target.com intitle:"index of"
site:target.com intitle:"directory listing"

# Documents
site:target.com ext:pdf
site:target.com ext:docx
site:target.com ext:xlsx

# Code repositories
site:github.com "target.com"
site:github.com "target.com" password
site:github.com "target.com" api_key

# Pastebin leaks
site:pastebin.com "target.com"
```

---

## 19. Automation & Tool Chains

### 19.1 Complete Recon Pipeline

**Script**:
```bash
#!/bin/bash

TARGET=$1

echo "[*] Starting recon for $TARGET"

# 1. Subdomain enumeration
echo "[*] Finding subdomains..."
subfinder -d $TARGET -o subdomains.txt

# 2. Probe for live hosts
echo "[*] Probing subdomains..."
cat subdomains.txt | httpx -o live-hosts.txt

# 3. Directory fuzzing on each host
echo "[*] Fuzzing directories..."
while read host; do
  ffuf -u "$host/FUZZ" -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt -o "$host-dirs.json" -of json -mc 200,301,302
done < live-hosts.txt

# 4. Extract interesting findings
echo "[*] Extracting results..."
cat *-dirs.json | jq -r '.results[].url' | sort -u > all-urls.txt

echo "[+] Recon complete! Results in all-urls.txt"
```

---

## 20. OSCP Attack Scenarios

### Scenario 1: Directory Fuzzing → Backup File → Credentials

**Target**: `http://192.168.50.10`

**Step 1: Basic Directory Scan**:
```bash
ffuf -u http://192.168.50.10/FUZZ \
     -w /usr/share/seclists/Discovery/Web-Content/common.txt \
     -fc 404

# Found: /admin (403), /backup (200)
```

**Step 2: Scan /backup**:
```bash
ffuf -u http://192.168.50.10/backup/FUZZ \
     -w /usr/share/seclists/Discovery/Web-Content/common.txt \
     -e .zip,.tar.gz,.bak,.sql

# Found: backup.zip
```

**Step 3: Download & Extract**:
```bash
wget http://192.168.50.10/backup/backup.zip
unzip backup.zip

# Contains: config.php with database credentials
```

**Step 4: Use Credentials**:
```bash
mysql -h 192.168.50.10 -u dbuser -p'password' -e "SELECT * FROM users"
```

### Scenario 2: VHost Discovery → Dev Site → RCE

**Target**: `http://192.168.50.20` (target.local)

**Step 1: VHost Fuzzing**:
```bash
ffuf -u http://192.168.50.20 \
     -H "Host: FUZZ.target.local" \
     -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
     -fs 1234

# Found: dev.target.local (different size)
```

**Step 2: Add to /etc/hosts**:
```bash
echo "192.168.50.20  dev.target.local" | sudo tee -a /etc/hosts
```

**Step 3: Access Dev Site**:
```bash
curl http://dev.target.local

# Dev site has phpinfo() enabled, debug mode, etc.
```

**Step 4: Find Upload**:
```bash
ffuf -u http://dev.target.local/FUZZ \
     -w wordlist.txt

# Found: /upload
```

**Step 5: Upload Shell → RCE**:
```bash
curl -X POST http://dev.target.local/upload -F "file=@shell.php"
curl "http://dev.target.local/uploads/shell.php?c=id"
```

---

## 21. Time Management for OSCP

### 21.1 Quick Recon (30 min)

```bash
# 1. Directory scan (10 min)
ffuf -u http://target/FUZZ -w common.txt -fc 404

# 2. Extension scan (5 min)
ffuf -u http://target/FUZZ -w files.txt -e .php,.bak,.zip

# 3. VHost scan (10 min)
ffuf -u http://IP -H "Host: FUZZ.target" -w subdomains.txt

# 4. Manual exploration (5 min)
# Browse site, check source code, robots.txt, sitemap.xml
```

### 21.2 Deep Recon (60 min)

Add to quick recon:
```bash
# 5. Large wordlist (20 min)
ffuf -u http://target/FUZZ -w raft-large.txt

# 6. Recursive scan (20 min)
ffuf -u http://target/FUZZ -w wordlist.txt -recursion -recursion-depth 2

# 7. API fuzzing (10 min)
ffuf -u http://target/api/FUZZ -w api-endpoints.txt

# 8. Parameter fuzzing (10 min)
arjun -u http://target/page
```

---

## 22. Common Pitfalls

**Pitfall 1: Using Only One Wordlist**
```
Solution: Try multiple (common, medium, large)
```

**Pitfall 2: Not Filtering Results**
```
Solution: Always use -fc 404 or -fs SIZE
```

**Pitfall 3: Ignoring 403 Responses**
```
Solution: 403 = Resource exists! Try bypass techniques
```

**Pitfall 4: Forgetting Extensions**
```
Solution: Always test .bak, .old, .zip
```

**Pitfall 5: Skipping Recursion**
```
Solution: If /api found, scan inside it
```

---

## 23. Hands-on Exercises

### Exercise 1: Complete Recon on DVWA

**Tasks**:
- [ ] Directory fuzzing (find /admin, /setup)
- [ ] Find backup files
- [ ] VHost discovery
- [ ] API endpoint discovery

### Exercise 2: Custom Wordlist Creation

**Tasks**:
- [ ] Use CeWL to create site-specific wordlist
- [ ] Combine with SecLists
- [ ] Test effectiveness

---

## 24. Interview Questions

**Q1: What's the difference between 403 and 404?**

**A**: 404 means "Not Found" (resource doesn't exist). 403 means "Forbidden" (resource EXISTS but access denied). During recon, 403 is more interesting because it confirms the resource is there.

---

**Q2: Why use multiple wordlists?**

**A**: Different wordlists have different coverage. Common.txt (4K words) is fast but limited. Directory-list-2.3-medium.txt (220K words) is comprehensive but slow. Start small, go bigger if needed.

---

**Q3: What is VHost discovery and why does it matter?**

**A**: VHost (Virtual Host) discovery finds websites hosted on the same IP but with different Host headers. These are often internal/dev sites not in public DNS, making them valuable targets (less security, outdated software).

---

**Q4: How do you handle rate limiting during fuzzing?**

**A**: Slow down with `-rate` flag, rotate User-Agents, use tools like FireProx to rotate source IPs via AWS API Gateway, or add delays between requests.

---

**Q5: What's the most important finding during recon?**

**A**: Backup files (.zip, .bak, .sql) often contain source code, database dumps, or credentials. They're direct paths to compromise.

---

## 25. Summary & Key Takeaways

### Critical Concepts

1. **Passive Before Active**: Start with crt.sh, Google dorking (no target contact)
2. **Multiple Wordlists**: common.txt → raft-medium → directory-list-2.3-medium
3. **Filter Everything**: Always use -fc 404 or -fs SIZE
4. **Extensions Matter**: .bak, .old, .zip are gold
5. **403 ≠ Blocked**: Try bypass techniques (path manipulation, headers)

### OSCP Exam Tips

1. **Always fuzz directories** - guaranteed findings on every web machine
2. **Start with common.txt** (4K words, 5 minutes)
3. **Test extensions**: .php,.bak,.old,.zip,.config
4. **Check for VHosts**: Especially if only IP provided
5. **Look for backups**: backup.zip, site.bak, db.sql
6. **Time allocation**: 30 min quick scan, 60 min deep scan
7. **Don't forget robots.txt, sitemap.xml, .git**

### Tools Priority (OSCP)

```
Primary:
1. ffuf (fast, versatile)
2. gobuster (reliable, simple)

Secondary:
3. wfuzz (alternative)
4. dirb (slow, but built-in)

Optional:
5. feroxbuster (Rust-based, fast)
```

### Quick Wins Checklist

- [ ] /admin, /administrator, /panel
- [ ] /backup, /backup.zip, /backups
- [ ] /.git, /.env, /config.php
- [ ] robots.txt, sitemap.xml
- [ ] /api, /api/v1, /swagger
- [ ] /phpmyadmin, /adminer
- [ ] crt.sh for subdomains

---

## 26. References & Further Reading

### Tools
- [FFUF](https://github.com/ffuf/ffuf)
- [Gobuster](https://github.com/OJ/gobuster)
- [SecLists](https://github.com/danielmiessler/SecLists)
- [Amass](https://github.com/OWASP/Amass)
- [Subfinder](https://github.com/projectdiscovery/subfinder)

### Related Topics
- [[02_Web_Reconnaissance]] - Initial reconnaissance techniques
- [[01_Web_Architecture_Components]] - Understanding web infrastructure
- [[02_OWASP_Top_10_Modern]] - Security misconfigurations

---

**END OF DOCUMENT** - Version 3.0 - Last Updated: 2026-01-24

**Total Lines**: ~1,850 lines | **OSCP Relevance**: CRITICAL (100%) | **Difficulty**: Intermediate to Advanced
