# Packet Analysis & Forensics: The Digital Microscope

---
title: "Packet Analysis & Forensics"
parent: "[[02_Networking]]"
tags:
  - network
  - forensics
  - wireshark
  - tshark
  - blue-team
  - analysis
created: 2026-01-23
updated: 2026-01-24
---

> **Executive Summary**: While basic Wireshark usage is about "seeing" traffic, Forensics is about "reconstructing" reality. This chapter moves beyond simple filters to carving files, decrypting malware traffic, statistical anomaly detection, and using command-line tools like TShark for massive datasets. Mastering these skills allows you to prove *exactly* what an exploit did, what data was stolen, and where the malware is beaconing.

---

## Table of Contents
1. [Learning Objectives](#1-learning-objectives)
2. [The Analyst's Toolkit](#2-the-analysts-toolkit)
3. [Packet Capture Fundamentals](#3-packet-capture-fundamentals)
4. [Wireshark Mastery](#4-wireshark-mastery)
5. [Display Filters Deep Dive](#5-display-filters-deep-dive)
6. [TShark: Automation & Large Scale](#6-tshark-automation--large-scale)
7. [Berkeley Packet Filter (BPF) Syntax](#7-berkeley-packet-filter-bpf-syntax)
8. [Protocol Dissection](#8-protocol-dissection)
9. [File Carving & Extraction](#9-file-carving--extraction)
10. [Malware Traffic Analysis](#10-malware-traffic-analysis)
11. [Decrypting TLS Traffic](#11-decrypting-tls-traffic)
12. [Statistical Analysis & Anomaly Detection](#12-statistical-analysis--anomaly-detection)
13. [Network Forensics Workflow](#13-network-forensics-workflow)
14. [Advanced Tools & Techniques](#14-advanced-tools--techniques)
15. [Practical Lab: Carving & Decrypting](#15-practical-lab-carving--decrypting)
16. [Critical Analysis & Checkpoints](#16-critical-analysis--checkpoints)

---

## 1. Learning Objectives

By the end of this chapter, you will be able to:

- **Carve Files**: Recover binaries, images, and zip files from raw PCAPs using Wireshark and `foremost`.
- **Decrypt Traffic**: Configure Wireshark to decrypt TLS streams using a Session Key log file (`sslkeylog`).
- **Automate with TShark**: Filter and extract fields from GBs of PCAP data via CLI.
- **Identify Malware**: Recognize C2 beacons, data exfiltration, and shellcode on the wire.
- **Filter Noise**: Use Display Filters to isolate the "needle" in the "haystack."
- **Reconstruct Events**: Build a timeline of network activity from packet captures.
- **Analyze Protocols**: Deep dive into TCP, HTTP, DNS, SMB traffic patterns.
- **Detect Attacks**: Identify port scans, DDoS, ARP poisoning, and exploitation attempts.
- **Master BPF**: Write complex Berkeley Packet Filters for precise capture.
- **Use Statistics**: Generate meaningful insights from traffic patterns.

---

## 2. The Analyst's Toolkit

**BEGINNER EXPLANATION**: Think of packet analysis like being a detective at a crime scene. You have tools to photograph evidence (tcpdump), magnifying glasses to examine details (Wireshark), and automated systems to catalog everything (Zeek).

### 2.1 Full Packet Capture (FPC) vs Flow Data

**Full Packet Capture**:
- Captures complete packets (headers + payload)
- Definitive proof of what happened
- Storage intensive (1 Gbps = ~450 GB/hour)
- Privacy and compliance concerns

**Flow Data (NetFlow/IPFIX)**:
- Metadata only (who talked to whom, when, how much)
- Much smaller storage (1% of FPC)
- Cannot reconstruct payloads
- Good for long-term visibility

**Comparison**:
```
┌─────────────────────┬──────────────┬─────────────┐
│     Feature         │     FPC      │    Flow     │
├─────────────────────┼──────────────┼─────────────┤
│ Storage Required    │  Very High   │    Low      │
│ Payload Visibility  │     Yes      │     No      │
│ Can Carve Files     │     Yes      │     No      │
│ Privacy Concerns    │    High      │  Moderate   │
│ Retention Period    │  Days/Weeks  │ Months/Years│
│ Analysis Speed      │    Slow      │    Fast     │
└─────────────────────┴──────────────┴─────────────┘
```

### 2.2 Essential Tools

#### Capture Tools
```
┌──────────────┬──────────────────────────────────────┐
│   Tool       │           Purpose                    │
├──────────────┼──────────────────────────────────────┤
│ tcpdump      │ CLI packet capture (Linux/Unix)      │
│ WinDump      │ Windows version of tcpdump           │
│ dumpcap      │ Wireshark's capture engine (fast)    │
│ tshark       │ CLI version of Wireshark             │
│ Moloch/Arkime│ Large-scale packet indexing          │
└──────────────┴──────────────────────────────────────┘
```

#### Analysis Tools
```
┌──────────────┬──────────────────────────────────────┐
│   Tool       │           Purpose                    │
├──────────────┼──────────────────────────────────────┤
│ Wireshark    │ GUI packet analyzer (best for deep   │
│              │ inspection of small-medium captures) │
│ tshark       │ CLI analysis, automation, scripting  │
│ NetworkMiner │ Automated artifact extraction        │
│ Zeek (Bro)   │ Network security monitor, log gen    │
│ tcpflow      │ TCP stream reassembly                │
│ ngrep        │ Network grep, regex on packets       │
│ Scapy        │ Packet manipulation and analysis     │
└──────────────┴──────────────────────────────────────┘
```

#### Forensics Tools
```
┌──────────────┬──────────────────────────────────────┐
│   Tool       │           Purpose                    │
├──────────────┼──────────────────────────────────────┤
│ foremost     │ File carving from PCAP               │
│ binwalk      │ Firmware and binary analysis         │
│ bulk_extractor│ Extract data (emails, URLs, IPs)    │
│ xplico       │ Internet traffic decoder             │
└──────────────┴──────────────────────────────────────┘
```

---

## 3. Packet Capture Fundamentals

**BEGINNER EXPLANATION**: Capturing packets is like recording a phone conversation. You need to be on the line (have network access) and have a recorder (capture tool) running.

### 3.1 Capture Points

**Where to Capture**:
```
Internet
    │
    ▼
 [Firewall] ◄── Capture Point 1 (Perimeter)
    │
    ▼
 [Core Switch] ◄── Capture Point 2 (Internal backbone)
    │
    ├── [VLAN 10] ◄── Capture Point 3 (Specific network segment)
    ├── [VLAN 20]
    └── [VLAN 30]
```

**Capture Methods**:
1. **Port Mirroring (SPAN)**: Switch copies traffic from monitored ports to analyzer port
2. **TAP (Test Access Point)**: Physical device that splits network cable
3. **Hub**: Broadcasts all traffic (old school, rare today)
4. **Host-based**: Capture on the endpoint itself

### 3.2 tcpdump Basics

**Installation**:
```bash
# Linux
apt-get install tcpdump

# macOS (pre-installed)
# Windows: Use WinDump
```

**Basic Capture**:
```bash
# Capture on interface eth0
tcpdump -i eth0

# Save to file
tcpdump -i eth0 -w capture.pcap

# Capture with verbose output
tcpdump -i eth0 -vvv

# Capture specific number of packets
tcpdump -i eth0 -c 100 -w first100.pcap

# Rotate capture files by size
tcpdump -i eth0 -w capture.pcap -C 100
# Creates capture.pcap, capture.pcap1, capture.pcap2... (100 MB each)

# Rotate capture files by time
tcpdump -i eth0 -w capture.pcap -G 3600 -W 24
# Creates 24 hourly files, then overwrites
```

**Common Options**:
```bash
-i <interface>   : Interface to capture on (eth0, any, lo)
-w <file>        : Write to file
-r <file>        : Read from file
-c <count>       : Capture N packets then stop
-s <snaplen>     : Bytes per packet (0 = unlimited, default 262144)
-n               : Don't resolve hostnames
-nn              : Don't resolve hostnames or ports
-v/-vv/-vvv      : Verbosity levels
-X               : Print packet data in hex and ASCII
-A               : Print packet data in ASCII only
-q               : Quick/quiet output
```

### 3.3 Capture Filters vs Display Filters

**Capture Filters** (BPF):
- Applied during capture
- Reduces file size
- Uses Berkeley Packet Filter syntax
- Cannot be changed after capture
- Example: `tcpdump -i eth0 'tcp port 80'`

**Display Filters**:
- Applied during analysis
- Does not reduce file size
- Uses Wireshark display filter syntax
- Can be changed anytime
- Example: `http.request.method == "POST"`

**When to Use Each**:
- **Capture Filter**: When you know exactly what you want (e.g., only HTTP traffic)
- **Display Filter**: When you need flexibility to explore the data

---

## 4. Wireshark Mastery

**BEGINNER EXPLANATION**: Wireshark is like a microscope for network traffic. You can zoom in on individual packets, follow conversations, and see exactly what data is being transmitted.

### 4.1 Interface Overview

```
┌─────────────────────────────────────────────────────────┐
│  Menu Bar: File, Edit, View, Go, Capture, Analyze...   │
├─────────────────────────────────────────────────────────┤
│  Toolbar: Start, Stop, Restart, Filters, Options       │
├─────────────────────────────────────────────────────────┤
│  Filter Bar: [Display Filter Input]         [Apply]    │
├─────────────────────────────────────────────────────────┤
│  Packet List Pane:                                      │
│  No. Time     Source      Destination   Protocol  Info  │
│  1   0.000    10.0.0.1    10.0.0.2      TCP       ...   │
│  2   0.001    10.0.0.2    10.0.0.1      TCP       ...   │
├─────────────────────────────────────────────────────────┤
│  Packet Details Pane:                                   │
│  ▼ Frame 1: 74 bytes on wire                           │
│  ▼ Ethernet II, Src: aa:bb:cc:dd:ee:ff                 │
│  ▼ Internet Protocol Version 4, Src: 10.0.0.1          │
│  ▼ Transmission Control Protocol, Src Port: 12345      │
├─────────────────────────────────────────────────────────┤
│  Packet Bytes Pane:                                     │
│  0000  00 11 22 33 44 55 66 77 88 99 aa bb 08 00 45 00 │
│  0010  00 3c 12 34 40 00 40 06 a1 b2 0a 00 00 01 0a 00 │
└─────────────────────────────────────────────────────────┘
```

### 4.2 Following Streams

**TCP Stream**:
```
Right-click packet → Follow → TCP Stream
```

**Benefits**:
- Reassembles fragmented data
- Shows entire conversation
- Color-coded (red = client, blue = server)
- Can save stream to file

**Stream Index**:
- Bottom right shows stream number (tcp.stream == 0)
- Use arrows to navigate between streams
- Filter automatically applied when following

**HTTP Stream**:
```
Right-click packet → Follow → HTTP Stream
```
- Automatically decodes chunked encoding
- Shows HTTP headers and body clearly

**TLS Stream** (if decrypted):
```
Right-click packet → Follow → TLS Stream
```
- Requires session keys (see Section 11)

### 4.3 Coloring Rules

**View → Coloring Rules**

**Default Colors**:
- **Light Blue**: UDP traffic
- **Light Purple**: TCP traffic
- **Black**: TCP errors (resets, retransmissions)
- **Green**: HTTP traffic
- **Light Green**: DNS traffic
- **Yellow**: Routing protocols
- **Gray**: TCP SYN/FIN
- **Red**: ICMP errors, TCP problems

**Custom Coloring**:
```
Name: Suspicious Ports
Filter: tcp.port in {4444 6666 31337}
Foreground: White
Background: Red
```

### 4.4 Expert Information

**Analyze → Expert Information**

**Categories**:
- **Errors** (Red): Malformed packets, checksum errors
- **Warnings** (Yellow): Retransmissions, fast retransmissions
- **Notes** (Cyan): Duplicate ACKs, zero window
- **Chats** (Blue): Normal protocol operations

**Common Issues**:
- **TCP Retransmission**: Original packet lost, resent
- **TCP Dup ACK**: Receiver telling sender a packet is missing
- **TCP Window Full**: Receiver's buffer is full
- **TCP Previous segment not captured**: Packet missing from capture

### 4.5 Protocol Hierarchy

**Statistics → Protocol Hierarchy**

Shows distribution of protocols in capture:
```
Frame: 10000 packets (100%)
├─ Ethernet: 10000 packets (100%)
│  ├─ IPv4: 9500 packets (95%)
│  │  ├─ TCP: 7000 packets (70%)
│  │  │  ├─ HTTP: 3000 packets (30%)
│  │  │  ├─ TLS: 2000 packets (20%)
│  │  │  └─ SMB: 1000 packets (10%)
│  │  └─ UDP: 2500 packets (25%)
│  │     ├─ DNS: 1500 packets (15%)
│  │     └─ QUIC: 500 packets (5%)
│  └─ IPv6: 500 packets (5%)
└─ ARP: 100 packets (1%)
```

---

## 5. Display Filters Deep Dive

**BEGINNER EXPLANATION**: Display filters are like search queries for packets. Instead of scrolling through thousands of packets, you tell Wireshark "show me only packets where the source IP is 192.168.1.100 and the destination port is 443."

### 5.1 Basic Operators

```
==   Equal
!=   Not equal
>    Greater than
<    Less than
>=   Greater than or equal
<=   Less than or equal
contains     Contains substring
matches      Regex match
&&   Logical AND
||   Logical OR
!    Logical NOT
```

### 5.2 Common Filters by Layer

**Ethernet/MAC**:
```
eth.addr == aa:bb:cc:dd:ee:ff
eth.src == aa:bb:cc:dd:ee:ff
eth.dst == aa:bb:cc:dd:ee:ff
```

**IP**:
```
ip.addr == 192.168.1.100
ip.src == 192.168.1.100
ip.dst == 192.168.1.100
ip.addr == 192.168.1.0/24
ip.ttl < 64
ip.flags.df == 1     # Don't Fragment
```

**TCP**:
```
tcp.port == 80
tcp.srcport == 12345
tcp.dstport == 443
tcp.flags.syn == 1 && tcp.flags.ack == 0   # SYN packet
tcp.flags.reset == 1                        # RST packet
tcp.analysis.retransmission                 # Retransmissions
tcp.stream == 5                             # Specific stream
tcp.len > 1000                              # Payload > 1000 bytes
```

**UDP**:
```
udp.port == 53
udp.length > 512
```

**HTTP**:
```
http.request.method == "POST"
http.request.uri contains "login"
http.host == "example.com"
http.user_agent contains "curl"
http.response.code == 200
http.response.code >= 400
http.content_type contains "application/json"
http.cookie contains "sessionid"
http.authorization
```

**DNS**:
```
dns.qry.name == "google.com"
dns.qry.type == 1    # A record
dns.qry.type == 16   # TXT record
dns.flags.response == 0    # Query
dns.flags.response == 1    # Response
dns.flags.rcode != 0       # Error responses
```

**TLS/SSL**:
```
tls.handshake.type == 1    # ClientHello
tls.handshake.type == 2    # ServerHello
tls.handshake.extensions_server_name contains "example.com"
ssl.record.content_type == 23    # Application Data
```

**SMB**:
```
smb.cmd == 0x72    # Negotiate Protocol
smb.cmd == 0xa2    # NT Create AndX
smb2.cmd == 5      # SMB2 Create
smb.path contains "\\admin"
```

**ARP**:
```
arp.opcode == 1    # Request
arp.opcode == 2    # Reply
arp.duplicate-address-detected
```

**ICMP**:
```
icmp.type == 8     # Echo Request (ping)
icmp.type == 0     # Echo Reply
icmp.type == 3     # Destination Unreachable
```

### 5.3 Advanced Filter Techniques

**Multiple Conditions**:
```
# Traffic between two IPs
(ip.addr == 192.168.1.100 && ip.addr == 192.168.1.200)

# Exclude local traffic
!(ip.src == 192.168.1.0/24 && ip.dst == 192.168.1.0/24)

# HTTP POST or PUT
http.request.method == "POST" || http.request.method == "PUT"

# Large TCP packets
tcp.len > 1400

# Suspicious ports
tcp.port in {4444 5555 6666 31337}
```

**Content Matching**:
```
# Contains string (case-sensitive)
frame contains "password"

# Contains hex bytes
frame contains 4d:5a:90:00    # MZ header (Windows executable)

# Regex (case-insensitive with ?i)
http.host matches "(?i)evil"

# Credit card pattern
frame matches "\d{4}[\s-]?\d{4}[\s-]?\d{4}[\s-]?\d{4}"
```

**Time-based**:
```
# Packets in specific time range
frame.time >= "2026-01-24 10:00:00" && frame.time <= "2026-01-24 11:00:00"

# Time delta (time since previous packet in stream)
tcp.time_delta > 1.0    # More than 1 second delay
```

**Statistical**:
```
# Packets with specific lengths
frame.len == 1514    # Maximum Ethernet frame
frame.len < 64       # Runt frames

# High entropy (potential encryption/compression)
data.len > 1000
```

### 5.4 Filter Macros

**Creating Macros**:
```
# Create macro for suspicious traffic
Analyze → Display Filter Macros → New

Name: suspicious_ports
Text: tcp.port in {4444 5555 6666 31337 8080}

# Use in filter
${suspicious_ports} && ip.src == 10.0.0.0/8
```

---

## 6. TShark: Automation & Large Scale

**BEGINNER EXPLANATION**: tshark is Wireshark's command-line cousin. It's perfect for analyzing huge capture files, automating analysis, and integrating with scripts.

### 6.1 Basic Usage

**Read and Display**:
```bash
# Read PCAP and display summary
tshark -r capture.pcap

# Apply display filter
tshark -r capture.pcap -Y "http"

# Read only first 100 packets
tshark -r capture.pcap -c 100
```

**Live Capture**:
```bash
# Capture on eth0
tshark -i eth0

# Capture with filter
tshark -i eth0 -f "tcp port 80"

# Capture and save
tshark -i eth0 -w output.pcap
```

### 6.2 Field Extraction

**Extract Specific Fields**:
```bash
# Extract DNS queries
tshark -r capture.pcap -T fields -e dns.qry.name -Y "dns.flags.response == 0"

# Extract HTTP hosts
tshark -r capture.pcap -T fields -e http.host -Y "http.request"

# Extract IP addresses and ports
tshark -r capture.pcap -T fields -e ip.src -e tcp.srcport -e ip.dst -e tcp.dstport -Y "tcp"

# Multiple fields with custom delimiter
tshark -r capture.pcap -T fields -E separator=, -e frame.time -e ip.src -e ip.dst -e http.host -Y "http"
```

**Output Formats**:
```bash
# JSON output
tshark -r capture.pcap -T json -Y "http" > output.json

# JSON with specific fields
tshark -r capture.pcap -T json -e http.host -e http.request.uri -Y "http"

# PDML (Packet Details Markup Language)
tshark -r capture.pcap -T pdml > output.xml

# Tab-separated
tshark -r capture.pcap -T tabs -e ip.src -e ip.dst
```

### 6.3 Statistics and Analysis

**Conversations**:
```bash
# IP conversations (who talked to whom)
tshark -r capture.pcap -q -z conv,ip

# TCP conversations
tshark -r capture.pcap -q -z conv,tcp

# Limit to top 10
tshark -r capture.pcap -q -z conv,ip | head -n 12
```

**Protocol Hierarchy**:
```bash
tshark -r capture.pcap -q -z io,phs
```

**Endpoints**:
```bash
# Top IP addresses
tshark -r capture.pcap -q -z endpoints,ip

# Top TCP ports
tshark -r capture.pcap -q -z endpoints,tcp
```

**HTTP Statistics**:
```bash
# HTTP requests
tshark -r capture.pcap -q -z http,tree

# HTTP request methods
tshark -r capture.pcap -q -z http_req,tree

# HTTP response codes
tshark -r capture.pcap -q -z http_srv,tree
```

**DNS Statistics**:
```bash
tshark -r capture.pcap -q -z dns,tree
```

### 6.4 Advanced Analysis Scripts

**Extract All DNS Queries**:
```bash
#!/bin/bash
tshark -r capture.pcap -T fields -e dns.qry.name -Y "dns.flags.response == 0" | \
sort | uniq -c | sort -rn > dns_queries.txt
```

**Find Unique User-Agents**:
```bash
tshark -r capture.pcap -T fields -e http.user_agent -Y "http.user_agent" | \
sort | uniq -c | sort -rn
```

**Extract HTTP Cookies**:
```bash
tshark -r capture.pcap -T fields -e http.cookie -Y "http.cookie" | \
grep -v "^$" | sort | uniq
```

**Timeline of HTTP Requests**:
```bash
tshark -r capture.pcap -T fields -e frame.time -e ip.src -e http.host -e http.request.uri -Y "http.request" -E separator=,
```

**Find Long-Duration Connections**:
```bash
#!/bin/bash
# Extract TCP streams with duration > 300 seconds
tshark -r capture.pcap -q -z conv,tcp | \
awk '$6 > 300.0 {print $1, $3, $6}' | column -t
```

---

## 7. Berkeley Packet Filter (BPF) Syntax

**BEGINNER EXPLANATION**: BPF is the language used for capture filters. It's more limited than display filters but very efficient because it works at the kernel level.

### 7.1 BPF Primitives

**Host Filters**:
```bash
host 192.168.1.100           # Traffic to/from this IP
src host 192.168.1.100       # Traffic from this IP
dst host 192.168.1.100       # Traffic to this IP
```

**Network Filters**:
```bash
net 192.168.1.0/24           # Traffic in this subnet
net 192.168.1.0 mask 255.255.255.0
src net 10.0.0.0/8
```

**Port Filters**:
```bash
port 80                      # Traffic on port 80 (TCP or UDP)
src port 80
dst port 443
portrange 1-1024             # Well-known ports
```

**Protocol Filters**:
```bash
tcp
udp
icmp
arp
ip
ip6
```

### 7.2 Combining Filters

**Logical Operators**:
```bash
and (or &&)
or  (or ||)
not (or !)
```

**Examples**:
```bash
# HTTP traffic
tcp port 80

# HTTPS traffic
tcp port 443

# DNS traffic
udp port 53 or tcp port 53

# Traffic to/from specific host on specific port
host 192.168.1.100 and port 22

# Exclude broadcast
not broadcast and not multicast

# Only SYN packets
tcp[tcpflags] & tcp-syn != 0

# Only SYN packets, no ACK
tcp[tcpflags] == tcp-syn

# All TCP except port 22
tcp and not port 22

# Multiple hosts
host 192.168.1.100 or host 192.168.1.200

# Specific subnet to specific host
src net 192.168.1.0/24 and dst host 8.8.8.8
```

### 7.3 Advanced BPF

**Byte Offset Matching**:
```bash
# Match bytes at specific offsets
# Syntax: proto[offset:size] operator value

# TCP flags (byte 13 of TCP header)
tcp[13] == 2         # SYN
tcp[13] == 18        # SYN-ACK
tcp[13] == 4         # RST

# IP TTL < 64
ip[8] < 64

# ICMP type 8 (echo request)
icmp[0] == 8

# HTTP GET (looking for "GET " in payload)
tcp[((tcp[12:1] & 0xf0) >> 2):4] = 0x47455420

# Match magic bytes (MZ header for Windows executables)
tcp[((tcp[12:1] & 0xf0) >> 2):2] = 0x4d5a
```

**VLAN Filtering**:
```bash
vlan 100                     # VLAN ID 100
vlan and tcp port 80         # VLAN traffic on port 80
```

**Fragmentation**:
```bash
ip[6:2] & 0x1fff != 0        # Fragmented packets
ip[6:2] & 0x2000 != 0        # Don't Fragment flag set
ip[6:2] & 0x4000 != 0        # More Fragments flag set
```

### 7.4 Practical BPF Examples

**Capture Only New TCP Connections**:
```bash
tcpdump -i eth0 'tcp[tcpflags] == tcp-syn'
```

**Capture SSH Brute Force Attempts**:
```bash
tcpdump -i eth0 'tcp dst port 22 and tcp[tcpflags] == tcp-syn' -c 100
```

**Capture Large Packets** (potential exfiltration):
```bash
tcpdump -i eth0 'greater 1000'
```

**Capture Specific IP Protocol** (GRE tunnels):
```bash
tcpdump -i eth0 'ip proto 47'
```

**Capture Non-HTTP on Port 80** (tunneling detection):
```bash
tcpdump -i eth0 'tcp port 80 and tcp[((tcp[12:1] & 0xf0) >> 2):4] != 0x47455420 and tcp[((tcp[12:1] & 0xf0) >> 2):4] != 0x504f5354'
```

---

## 8. Protocol Dissection

**BEGINNER EXPLANATION**: Protocol dissection is like taking apart an onion layer by layer. Each protocol wraps around the next, and you need to understand each layer to see the full picture.

### 8.1 TCP Stream Analysis

**TCP Handshake Verification**:
```
1. SYN → [Flags: S]
2. SYN-ACK ← [Flags: SA]
3. ACK → [Flags: A]

Look for:
- Missing SYN-ACK (filtered port)
- RST instead of SYN-ACK (closed port)
- Multiple SYN retransmissions (packet loss)
```

**TCP Teardown**:
```
Normal Close:
1. FIN → [Flags: FA]
2. ACK ← [Flags: A]
3. FIN ← [Flags: FA]
4. ACK → [Flags: A]

Abnormal Close:
- RST (abrupt termination)
```

**Window Size Analysis**:
```
Filter: tcp.analysis.zero_window

Indicates:
- Receiver buffer full
- Application not reading fast enough
- Potential DoS or performance issue
```

**Retransmission Analysis**:
```
Filter: tcp.analysis.retransmission

Causes:
- Network congestion
- Packet loss
- Firewalls dropping packets
```

### 8.2 HTTP Request/Response Pairs

**Matching Requests to Responses**:
```
1. Find HTTP request
2. Note tcp.stream number
3. Filter: tcp.stream == X && http.response
4. Look at http.time (response time)
```

**Slow Response Detection**:
```
Filter: http.time > 5.0

Shows HTTP requests that took > 5 seconds to respond
```

### 8.3 DNS Traffic Analysis

**DNS Query Types**:
```
Type 1   = A (IPv4 address)
Type 5   = CNAME (Canonical name)
Type 15  = MX (Mail exchange)
Type 16  = TXT (Text record)
Type 28  = AAAA (IPv6 address)
Type 255 = ANY (All records)
```

**DNS Response Codes**:
```
0 = No Error
1 = Format Error
2 = Server Failure
3 = Non-Existent Domain (NXDOMAIN)
5 = Refused
```

**DNS Tunneling Detection**:
```
Filter: dns.qry.name.len > 50

Long subdomain names may indicate tunneling:
AAAAABBBBCCCCDDDD.EEEEFFFF.evil.com
```

### 8.4 SMB Session Analysis

**SMB Session Flow**:
```
1. TCP Handshake (445)
2. SMB Negotiate Protocol
3. SMB Session Setup (authentication)
4. SMB Tree Connect (\\server\share)
5. SMB Create/Open file
6. SMB Read/Write
7. SMB Close
8. SMB Tree Disconnect
9. SMB Logoff
10. TCP Close
```

**File Transfer Reconstruction**:
```
1. Filter: smb2.cmd == 5 (Create/Open)
2. Find filename in smb2.filename
3. Follow TCP stream
4. Look for SMB2 Read/Write responses
5. Extract data from smb2.file_data
```

---

## 9. File Carving & Extraction

**BEGINNER EXPLANATION**: File carving is like finding torn pieces of a document in a trash bag and taping them back together. You look for file signatures (magic bytes) and reassemble the data.

### 9.1 File Signatures (Magic Bytes)

**Common File Types**:
```
┌─────────────┬──────────────────┬─────────────────┐
│  File Type  │  Magic Bytes     │  Extension      │
├─────────────┼──────────────────┼─────────────────┤
│ JPEG        │ FF D8 FF         │ .jpg            │
│ PNG         │ 89 50 4E 47      │ .png            │
│ GIF         │ 47 49 46 38      │ .gif            │
│ PDF         │ 25 50 44 46      │ .pdf            │
│ ZIP         │ 50 4B 03 04      │ .zip            │
│ Windows EXE │ 4D 5A            │ .exe            │
│ ELF Binary  │ 7F 45 4C 46      │ (none)          │
│ MS Office   │ 50 4B 03 04      │ .docx/.xlsx     │
│ RAR         │ 52 61 72 21      │ .rar            │
│ 7-Zip       │ 37 7A BC AF      │ .7z             │
│ MP3         │ FF FB or ID3     │ .mp3            │
│ MP4         │ 66 74 79 70      │ .mp4            │
└─────────────┴──────────────────┴─────────────────┘
```

### 9.2 Wireshark Export Objects

**HTTP Objects**:
```
File → Export Objects → HTTP

Shows all HTTP transfers:
- Filename
- Content Type
- Size
- Hostname

Select and save individual files or save all.
```

**SMB Objects**:
```
File → Export Objects → SMB

Extracts files transferred over SMB/CIFS:
- Useful for lateral movement analysis
- Can reveal malware dropped on shares
```

**DICOM (Medical Imaging)**:
```
File → Export Objects → DICOM
```

**IMF (Email)**:
```
File → Export Objects → IMF
Extracts email messages from SMTP/POP3/IMAP
```

### 9.3 Manual File Carving

**Using tcpflow**:
```bash
# Extract all TCP streams
tcpflow -r capture.pcap -o output_dir/

# This creates files like:
# 192.168.001.100.12345-10.000.000.050.00080

# Find executables
find output_dir/ -type f -exec file {} \; | grep -i "executable"

# Find images
find output_dir/ -type f -exec file {} \; | grep -i "image"
```

**Using foremost**:
```bash
# Convert PCAP to raw data
tcpdump -r capture.pcap -w raw.bin -s 0

# Carve files
foremost -v -i raw.bin -o carved_files/

# Results in carved_files/ organized by type:
# jpg/, png/, pdf/, zip/, exe/, etc.
```

**Using binwalk**:
```bash
# Extract embedded files
binwalk -e --dd='.*' capture.pcap
```

**Using Scapy**:
```python
#!/usr/bin/env python3
from scapy.all import *

def extract_http_files(pcap_file):
    packets = rdpcap(pcap_file)
    
    for pkt in packets:
        if pkt.haslayer(TCP) and pkt.haslayer(Raw):
            payload = bytes(pkt[Raw].load)
            
            # Look for HTTP response with content
            if b'HTTP/1.' in payload and b'\r\n\r\n' in payload:
                header, body = payload.split(b'\r\n\r\n', 1)
                
                # Check for image
                if b'Content-Type: image/jpeg' in header:
                    if body.startswith(b'\xff\xd8\xff'):
                        filename = f"carved_image_{pkt.time}.jpg"
                        with open(filename, 'wb') as f:
                            f.write(body)
                        print(f"[+] Saved {filename}")

# extract_http_files("capture.pcap")
```

### 9.4 Reconstructing Files from TCP Streams

**Manual Method**:
```
1. Follow TCP Stream
2. Show data as "Raw"
3. Save as...
4. For HTTP: Remove HTTP headers manually
5. Verify file with `file` command
```

**Automated with tshark**:
```bash
# Export HTTP objects
tshark -r capture.pcap --export-objects http,http_objects/

# Export SMB objects
tshark -r capture.pcap --export-objects smb,smb_objects/
```

---

## 10. Malware Traffic Analysis

**BEGINNER EXPLANATION**: Malware creates specific patterns in network traffic. Learning to recognize these patterns is like learning to spot counterfeit money - once you know what to look for, it becomes obvious.

### 10.1 Command & Control (C2) Beaconing

**Characteristics**:
- Regular intervals (every 60 seconds, 5 minutes, etc.)
- Small request, small response
- Same destination IP/domain
- Often on non-standard ports (8080, 8443, 443 with non-HTTPS)

**Detection with Statistics**:
```
Statistics → Conversations → TCP
Sort by Duration
Look for:
- Long-lived connections
- Regular packet patterns
- Small packet sizes
```

**Beaconing Script**:
```python
#!/usr/bin/env python3
from scapy.all import *
from collections import defaultdict
import statistics

def detect_beaconing(pcap_file, threshold=0.1):
    """
    Detect beaconing based on regular time intervals
    threshold: acceptable variance in interval (0.1 = 10%)
    """
    packets = rdpcap(pcap_file)
    connections = defaultdict(list)
    
    # Group packets by connection
    for pkt in packets:
        if pkt.haslayer(TCP) and pkt.haslayer(IP):
            key = (pkt[IP].src, pkt[TCP].sport, pkt[IP].dst, pkt[TCP].dport)
            connections[key].append(float(pkt.time))
    
    # Analyze time intervals
    for conn, times in connections.items():
        if len(times) < 10:  # Need at least 10 packets
            continue
        
        times.sort()
        intervals = [times[i+1] - times[i] for i in range(len(times)-1)]
        
        if intervals:
            mean_interval = statistics.mean(intervals)
            if mean_interval > 0:
                stdev = statistics.stdev(intervals) if len(intervals) > 1 else 0
                variance = stdev / mean_interval
                
                if variance < threshold and mean_interval > 10:  # Regular + not too frequent
                    print(f"[!] Possible beaconing detected:")
                    print(f"    {conn[0]}:{conn[1]} → {conn[2]}:{conn[3]}")
                    print(f"    Interval: {mean_interval:.2f}s (±{variance*100:.1f}%)")
                    print(f"    Packet count: {len(times)}\n")

# detect_beaconing("capture.pcap")
```

### 10.2 Data Exfiltration Patterns

**DNS Tunneling**:
```
Filter: dns.qry.name.len > 50

Example:
4142434445464748494a4b4c4d4e4f50.exfil.evil.com

Detection:
- Long subdomain names
- High entropy in subdomain
- Large number of unique subdomains
- NXDOMAIN responses (using DNS as transport, not resolution)
```

**ICMP Tunneling**:
```
Filter: icmp.data.len > 64

Normal ping: 32-56 bytes
Tunnel: 100+ bytes with high entropy
```

**HTTP POST Exfiltration**:
```
Filter: http.request.method == "POST" and frame.len > 1400

Large POST requests to suspicious domains
Sort by frame.len to find largest uploads
```

**TLS Exfiltration**:
```
Filter: tls.record.content_type == 23 and frame.len > 1400

Large encrypted uploads
Look at SNI to identify destination
```

### 10.3 Shellcode Detection

**NOP Sleds**:
```
Filter: frame contains 90:90:90:90:90:90:90:90

0x90 = NOP instruction in x86
Long sequences indicate shellcode
```

**Metasploit Patterns**:
```
# Meterpreter HTTP beaconing
http.request.uri contains "INITM"
http.request.uri contains "CONN_"

# Meterpreter user-agent
http.user_agent contains "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1)"
```

**Executable in Network Traffic**:
```
Filter: frame contains 4d:5a:90:00

MZ header (DOS/Windows executable)
Indicates binary download or upload
```

### 10.4 Common Malware Families

**Emotet**:
- HTTP C2 on port 80/443
- POST requests to /random_path
- Cookie header with bot ID
- User-Agent spoofing

**Cobalt Strike**:
- HTTPS beaconing (default 60s jitter)
- Malleable C2 profiles (customizable)
- Can mimic legitimate traffic
- Check TLS certificate (often self-signed or suspicious)

**TrickBot**:
- HTTPS C2
- Large encrypted POST requests
- Targets banking websites
- Web injects visible if TLS decrypted

**Ransomware**:
- SMB lateral movement (445)
- Large file transfers
- Connections to TOR .onion sites (via proxy)
- Bitcoin payment infrastructure

---

## 11. Decrypting TLS Traffic

**BEGINNER EXPLANATION**: TLS encryption is like a locked box. To see inside, you need either the key (private key method) or a copy of the temporary lock combination (session key method).

### 11.1 Session Key Logging (SSLKEYLOGFILE)

**How It Works**:
- Browser/client writes session keys to a log file
- Format defined by NSS (Network Security Services)
- Contains pre-master secrets for each TLS session
- Wireshark uses these to decrypt traffic

**Setup for Firefox**:
```bash
# Linux/macOS
export SSLKEYLOGFILE=~/sslkeys.log
firefox &

# Windows (PowerShell)
$env:SSLKEYLOGFILE="C:\Users\User\sslkeys.log"
& "C:\Program Files\Mozilla Firefox\firefox.exe"
```

**Setup for Chrome**:
```bash
# Linux/macOS
export SSLKEYLOGFILE=~/sslkeys.log
google-chrome &

# Windows
Set SSLKEYLOGFILE=C:\Users\User\sslkeys.log
"C:\Program Files\Google\Chrome\Application\chrome.exe"
```

**Wireshark Configuration**:
```
Edit → Preferences → Protocols → TLS
(Pre)-Master-Secret log filename: /home/user/sslkeys.log
```

**Verify**:
1. Browse to https://example.com
2. Capture traffic in Wireshark
3. Look for "Decrypted TLS" tab in packet details
4. HTTP requests/responses should be visible

### 11.2 Private Key Method (Legacy)

**When It Works**:
- RSA key exchange (no forward secrecy)
- You have server's private key
- Mostly obsolete (modern TLS uses ECDHE)

**Configuration**:
```
Edit → Preferences → Protocols → TLS
RSA keys list → Edit → Add new keyfile
IP address: 192.168.1.100
Port: 443
Protocol: http
Key File: /path/to/server-private-key.pem
```

### 11.3 Decryption in tshark

**Using Session Keys**:
```bash
tshark -r capture.pcap -o tls.keylog_file:sslkeys.log -Y "http" -T fields -e http.host -e http.request.uri
```

**Extract HTTP from Decrypted TLS**:
```bash
tshark -r capture.pcap -o tls.keylog_file:sslkeys.log --export-objects http,http_objects/
```

### 11.4 Limitations and Challenges

**Cannot Decrypt**:
- Forward secrecy (ECDHE) without session log
- TLS 1.3 (without session log)
- Certificate pinned connections (need to intercept)
- Client certificates without client private key

**Solution for Mobile Apps**:
```
1. Root device / jailbreak
2. Install custom CA certificate
3. Use proxy (Burp, mitmproxy) with SSL interception
4. For certificate pinning:
   - Use Frida to hook SSL validation
   - Recompile app with pinning removed
   - Use SSLKillSwitch (iOS) or similar
```

---

## 12. Statistical Analysis & Anomaly Detection

**BEGINNER EXPLANATION**: Statistics help you see the forest, not just the trees. Instead of looking at individual packets, you look at patterns and trends to spot anomalies.

### 12.1 Baseline vs Anomaly

**Baseline Metrics**:
- Normal traffic volume per hour
- Typical protocol distribution
- Expected internal/external ratio
- Average packet sizes
- Common ports and destinations

**Anomalies**:
- Sudden spike in traffic
- New protocols
- Unusual packet sizes
- Connections to new countries/ASNs
- Traffic during off-hours

### 12.2 I/O Graphs

**Wireshark I/O Graphs**:
```
Statistics → I/O Graphs

Configurations:
- Y Axis: Packets/sec, Bytes/sec, Bits/sec
- X Axis: Time
- Multiple graphs for comparison
```

**Detecting Spikes**:
```
Graph 1 (All traffic): (no filter)
Graph 2 (DNS): dns
Graph 3 (HTTP): http
Graph 4 (TLS): tls

Look for anomalies in specific protocols
```

**DDoS Detection**:
```
Graph 1: tcp.flags.syn == 1 && tcp.flags.ack == 0
Graph 2: tcp.flags.reset == 1

SYN flood = Graph 1 spike
RST responses = Graph 2 spike
```

### 12.3 Conversation Analysis

**Top Talkers**:
```bash
# Using tshark
tshark -r capture.pcap -q -z conv,ip | sort -k6 -rn | head -n 20
# Column 6 = total bytes
```

**Geographic Analysis**:
```bash
# Extract all external IPs
tshark -r capture.pcap -T fields -e ip.dst -Y "ip.dst != 192.168.0.0/16 and ip.dst != 10.0.0.0/8 and ip.dst != 172.16.0.0/12" | sort | uniq > external_ips.txt

# Use GeoIP database
while read ip; do
    geoiplookup $ip
done < external_ips.txt
```

### 12.4 Entropy Analysis

**High Entropy = Encryption/Compression**:
```python
#!/usr/bin/env python3
from scapy.all import *
import math
from collections import Counter

def calculate_entropy(data):
    """Calculate Shannon entropy of byte data"""
    if not data:
        return 0
    
    counter = Counter(data)
    length = len(data)
    entropy = 0
    
    for count in counter.values():
        freq = count / length
        entropy -= freq * math.log2(freq)
    
    return entropy

def analyze_payload_entropy(pcap_file, threshold=7.0):
    """Find packets with high entropy payloads"""
    packets = rdpcap(pcap_file)
    
    for pkt in packets:
        if pkt.haslayer(Raw):
            payload = bytes(pkt[Raw].load)
            if len(payload) > 100:  # Only analyze significant payloads
                entropy = calculate_entropy(payload)
                if entropy > threshold:
                    print(f"[!] High entropy detected: {entropy:.2f}")
                    if pkt.haslayer(IP):
                        print(f"    {pkt[IP].src} → {pkt[IP].dst}")
                    print(f"    Length: {len(payload)} bytes\n")

# analyze_payload_entropy("capture.pcap")
```

---

## 13. Network Forensics Workflow

**BEGINNER EXPLANATION**: Network forensics is like being a detective. You need a systematic approach to analyze evidence, preserve chain of custody, and build a timeline of events.

### 13.1 Incident Response Steps

```
1. Preparation
   ├─ Have capture tools ready
   ├─ Know network topology
   └─ Document baseline behavior

2. Identification
   ├─ Alert triggers (IDS, SIEM)
   ├─ Initial triage
   └─ Scope determination

3. Containment
   ├─ Isolate affected systems
   ├─ Start packet capture
   └─ Preserve evidence

4. Analysis
   ├─ Timeline reconstruction
   ├─ Artifact extraction
   ├─ Malware identification
   └─ Impact assessment

5. Eradication
   ├─ Remove malware
   ├─ Patch vulnerabilities
   └─ Reset credentials

6. Recovery
   ├─ Restore systems
   ├─ Monitor for reinfection
   └─ Resume normal operations

7. Lessons Learned
   ├─ Document findings
   ├─ Update defenses
   └─ Training/awareness
```

### 13.2 Evidence Collection

**Capture Considerations**:
```bash
# Full packet capture with rotation
tcpdump -i eth0 -w /evidence/capture-%Y%m%d-%H%M%S.pcap -G 3600 -W 24 -s 0 -Z root

# -G 3600: Rotate every hour
# -W 24: Keep 24 files (24 hours)
# -s 0: Capture full packets
# -Z root: Drop privileges after opening capture
```

**Chain of Custody**:
```
Document:
- Date/time of capture
- Who performed capture
- Source network/interface
- Capture method and tools
- Hash of capture file (md5sum, sha256sum)
- Storage location
```

**Hashing Evidence**:
```bash
sha256sum capture.pcap > capture.pcap.sha256
md5sum capture.pcap > capture.pcap.md5
```

### 13.3 Timeline Reconstruction

**Extract All Timestamps**:
```bash
# Create CSV timeline
tshark -r capture.pcap -T fields -E separator=, -e frame.time -e ip.src -e ip.dst -e tcp.dstport -e http.host -e dns.qry.name > timeline.csv
```

**Filter by Time Range**:
```bash
# Packets between specific times
tshark -r capture.pcap -Y "frame.time >= \"2026-01-24 14:30:00\" and frame.time <= \"2026-01-24 15:00:00\"" -w filtered.pcap
```

**Build Event Timeline**:
```
14:30:15 - Initial compromise (exploit payload to 192.168.1.50)
14:30:18 - Reverse shell established (192.168.1.50:4444 → attacker)
14:31:00 - Credential dumping (LSASS access via Process Hacker)
14:32:30 - Lateral movement (SMB to 192.168.1.51)
14:35:00 - Data exfiltration (large HTTP POST to evil.com)
14:40:00 - Malware installation (download trojan.exe)
```

### 13.4 Correlation with Other Logs

**Cross-Reference**:
```
PCAP Timeline:
14:30:15 - Connection to 192.168.1.50:80

Windows Event Log:
14:30:15 - EventID 4624 (Logon) from 192.168.1.100

Firewall Log:
14:30:15 - ALLOW 192.168.1.100 → 192.168.1.50:80

IDS Alert:
14:30:16 - Exploit attempt detected (CVE-2024-XXXX)
```

---

## 14. Advanced Tools & Techniques

### 14.1 Zeek (Formerly Bro)

**Installation**:
```bash
apt-get install zeek
```

**Basic Usage**:
```bash
# Process PCAP and generate logs
zeek -r capture.pcap

# Output files:
# conn.log - Connection summaries
# dns.log - DNS queries/responses
# http.log - HTTP requests/responses
# ssl.log - TLS/SSL connections
# files.log - File transfers
# weird.log - Protocol anomalies
```

**Analyzing Zeek Logs**:
```bash
# Find long connections
cat conn.log | zeek-cut id.orig_h id.resp_h duration | awk '$3 > 3600'

# Extract HTTP hosts
cat http.log | zeek-cut host uri | sort | uniq

# Find TLS certificates
cat ssl.log | zeek-cut subject issuer

# Detect scanning
cat conn.log | zeek-cut id.orig_h | sort | uniq -c | sort -rn | head
```

### 14.2 NetworkMiner

**Features**:
- Automated OS fingerprinting
- File extraction (HTTP, FTP, SMB, TFTP)
- Credential extraction
- DNS hostname resolution
- Anomaly detection

**Usage**:
```
1. Open PCAP in NetworkMiner
2. Review Hosts tab (discovered systems)
3. Check Files tab (extracted files)
4. Inspect Credentials tab (cleartext passwords)
5. Review Images tab (carvedimages)
```

### 14.3 Moloch/Arkime

**Purpose**: Large-scale packet indexing and search

**Features**:
- Web interface for search
- Full packet storage with metadata indexing
- Tags and annotations
- Session export to PCAP
- Multi-user with RBAC

**Deployment**:
```
Capture Node → Elasticsearch → Viewer (Web UI)
```

### 14.4 Custom Analysis with Scapy

**Packet Replay**:
```python
from scapy.all import *

# Read and replay packets
packets = rdpcap("capture.pcap")
sendp(packets, iface="eth0")
```

**Packet Modification**:
```python
from scapy.all import *

packets = rdpcap("capture.pcap")
modified = []

for pkt in packets:
    if pkt.haslayer(IP):
        # Change source IP
        pkt[IP].src = "192.168.1.100"
        # Recalculate checksums
        del pkt[IP].chksum
        del pkt[TCP].chksum if pkt.haslayer(TCP) else None
    modified.append(pkt)

wrpcap("modified.pcap", modified)
```

**Protocol Fuzzing**:
```python
from scapy.all import *

# Fuzz HTTP request
pkt = IP(dst="192.168.1.100")/TCP(dport=80)/Raw(load="GET /"+fuzz("A"*100)+" HTTP/1.1\r\n\r\n")
send(pkt)
```

---

## 15. Practical Lab: Carving & Decrypting

### Lab 1: HTTP File Extraction

**Objective**: Extract all files transferred over HTTP.

**Step 1: Open PCAP**:
```
File → Open → capture.pcap
```

**Step 2: Filter HTTP**:
```
http
```

**Step 3: Export Objects**:
```
File → Export Objects → HTTP
```

**Step 4: Analyze**:
- Sort by Size (find largest files)
- Check Content Type
- Look for suspicious filenames (shell.php, payload.exe)
- Save selected files

**Step 5: Verify with file command**:
```bash
file extracted_file
md5sum extracted_file
```

### Lab 2: TLS Decryption

**Objective**: Decrypt HTTPS traffic and extract passwords.

**Step 1: Setup Browser**:
```bash
export SSLKEYLOGFILE=~/sslkeys.log
firefox &
```

**Step 2: Configure Wireshark**:
```
Edit → Preferences → Protocols → TLS
(Pre)-Master-Secret log filename: /home/user/sslkeys.log
```

**Step 3: Capture Traffic**:
```
Start capture on appropriate interface
Browse to https://testsite.com/login
Submit login form with credentials
```

**Step 4: Analyze**:
```
Filter: http.request.method == "POST"
Look for decrypted POST data
Extract username/password from form data
```

### Lab 3: DNS Tunneling Detection

**Objective**: Identify DNS tunneling activity.

**Step 1: Extract DNS Queries**:
```bash
tshark -r capture.pcap -T fields -e dns.qry.name -Y "dns.flags.response == 0" > dns_queries.txt
```

**Step 2: Analyze Lengths**:
```bash
awk '{print length, $0}' dns_queries.txt | sort -rn | head -n 50
```

**Step 3: Look for Patterns**:
```
- Long subdomain names (> 50 chars)
- High entropy strings
- Many unique subdomains to same domain
- Base64-looking strings
```

**Step 4: Wireshark Filter**:
```
dns.qry.name.len > 50
```

### Lab 4: Malware C2 Beaconing

**Objective**: Identify command and control beaconing.

**Step 1: Conversations**:
```
Statistics → Conversations → TCP
Sort by "Packets" or "Duration"
```

**Step 2: I/O Graph**:
```
Statistics → I/O Graphs
Graph specific connection:
ip.addr == 192.168.1.100 && ip.addr == 203.0.113.50
```

**Step 3: Look for Regularity**:
- Packets at regular intervals
- Same packet sizes
- Long-duration connection

**Step 4: Follow Stream**:
```
Right-click → Follow → TCP Stream
Analyze payload for commands/responses
```

### Lab 5: SMB Lateral Movement

**Objective**: Reconstruct attacker's lateral movement via SMB.

**Step 1: Find SMB Authentication**:
```
smb2.cmd == 1 && smb2.session_setup
```

**Step 2: Identify Accounts Used**:
```
Look at NTLMSSP authentication
Extract usernames from smb2.user_name
```

**Step 3: Find File Access**:
```
smb2.cmd == 5 (Create)
smb2.filename
```

**Step 4: Map Timeline**:
```
Sort by time
Document:
- Which systems accessed
- Which accounts used
- Which files accessed/created
- Timestamps
```

---

## 16. Critical Analysis & Checkpoints

### 16.1 Common Pitfalls

**1. Trusting Capture Completeness**:
- Missing packets (check for gaps in seq numbers)
- Interface dropped packets (check `capinfos`)
- Asymmetric routing (only see one direction)

**2. Time Synchronization**:
- Different systems with different clocks
- Correlation becomes difficult
- Use NTP, document timezone

**3. Privacy and Legal**:
- Capturing traffic may violate laws
- PII (Personally Identifiable Information)
- Requires authorization and proper handling

**4. Encryption Everywhere**:
- TLS 1.3 adoption
- DNS over HTTPS (DoH)
- VPNs and tunnels
- Solution: Endpoint visibility (EDR)

### 16.2 Checkpoint Questions

**Q1**: What is the "Magic Number" for a JPEG file?
- **Answer**: `FF D8 FF`. These bytes appear at the start of every JPEG file. Looking for them in packet payloads helps identify JPEG files even if headers are modified.

**Q2**: Why can't I see the "Host" header in an HTTPS packet in Wireshark?
- **Answer**: HTTP headers are inside the encrypted TLS tunnel. You can only see the SNI (Server Name Indication) in the TLS ClientHello, which is sent in cleartext (unless ESNI/ECH is used).

**Q3**: How do you detect a "Slowloris" attack in Wireshark?
- **Answer**: Look for many incomplete HTTP connections from a single source IP. Filter: `tcp.stream && http` and look for connections with HTTP request headers but no final blank line (`\r\n\r\n`), with periodic `Keep-Alive` headers to maintain the connection.

**Q4**: What's the difference between a capture filter and a display filter?
- **Answer**: Capture filters (BPF syntax) are applied during packet capture and reduce file size. Display filters (Wireshark syntax) are applied during analysis and don't affect file size. Capture filters cannot be changed after capture.

**Q5**: How can you detect port scanning in a PCAP?
- **Answer**: Look for many SYN packets from one source to many destinations/ports. Filter: `tcp.flags.syn == 1 && tcp.flags.ack == 0`. Use Statistics → Conversations to find source IPs with many unique destination IPs/ports.

**Q6**: What indicates a TCP connection was successfully established?
- **Answer**: Three packets: SYN (from client), SYN-ACK (from server), ACK (from client). Filter for a specific stream: `tcp.stream == 0` and look for this sequence.

### 16.3 Real-World Scenarios

**Scenario 1: Ransomware Investigation**:
```
Evidence: Encrypted files on fileserver
Timeline Reconstruction:
1. Initial access (phishing email with macro)
2. Macro downloads stage 2 (HTTP GET to malicious domain)
3. Stage 2 establishes C2 (HTTPS beaconing)
4. Lateral movement (SMB connections with admin creds)
5. File encryption (large SMB writes to fileserver)
6. Ransom note delivery (HTTP POST)

PCAP Analysis:
- Extract stage 2 payload (HTTP object export)
- Identify C2 domain (TLS SNI)
- Map lateral movement (SMB authentication logs)
- Calculate encryption start time (spike in SMB write operations)
```

**Scenario 2: Data Breach**:
```
Evidence: 10GB database dump found on dark web
Investigation:
1. Identify exfiltration method (large HTTP POST, DNS tunneling, etc.)
2. Find initial compromise (SQL injection, RCE)
3. Determine attacker IP/infrastructure
4. Calculate data volume exfiltrated
5. Timeline of access

PCAP Indicators:
- Large outbound transfers
- Connections to suspicious domains
- Unusual time-of-day activity
- Database server connecting to internet (abnormal)
```

### 16.4 Future Challenges

**"Going Dark"**:
- TLS 1.3 everywhere
- ESNI/ECH (encrypted SNI)
- DNS over HTTPS/TLS
- VPNs and tunnels
- End-to-end encryption

**Solutions**:
- Endpoint detection and response (EDR)
- TLS inspection at endpoints
- Behavioral analysis (NetFlow/metadata)
- Memory forensics
- Application-level logging

### 16.5 Final Thoughts

Packet analysis is both an art and a science. It requires:
- Technical knowledge (protocols, encoding, encryption)
- Pattern recognition (anomalies, IOCs)
- Attention to detail (one packet can break the case)
- Persistence (digging through gigabytes of data)
- Creativity (thinking like the attacker)

**Key Takeaways**:
- Packets don't lie (ground truth)
- Always verify findings with multiple sources
- Document everything (chain of custody)
- Practice with malware traffic analysis exercises
- Stay current with new protocols and encryption

**Resources for Practice**:
- Malware-Traffic-Analysis.net (free PCAPs)
- SANS NetWars
- HackTheBox / TryHackMe
- Wireshark sample captures
- CTF challenges (PCAP forensics)

---

*End of Chapter 08 - Packet Analysis & Forensics*


