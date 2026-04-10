# Linux Networking Stack - COMPREHENSIVE GUIDE

---
title: "Linux Networking Stack"
parent: "[[03_Linux]]"
tags:
  - linux
  - networking
  - iptables
  - nftables
  - namespaces
  - routing
  - pivoting
  - tcpdump
  - wireshark
created: 2026-01-23
updated: 2026-01-24
---

> **Executive Summary**: Linux is the backbone of the internet - most routers, firewalls, and network infrastructure run on Linux. Understanding the Linux networking stack is CRITICAL for OSCP and penetration testing because you need to configure network interfaces, manipulate routing tables, set up port forwarding, create SSH tunnels, and pivot through compromised systems. Whether you're redirecting traffic through a compromised host, setting up a SOCKS proxy, analyzing packets with tcpdump, or understanding how iptables firewalls block your reverse shells, networking knowledge separates script kiddies from professional penetration testers. This comprehensive guide covers network fundamentals from absolute basics (what is an IP address?) to advanced OSCP techniques (SSH tunneling, port forwarding, network pivoting). You'll learn TCP/IP fundamentals, network interface configuration, routing, DNS, iptables/nftables firewalls, packet analysis, network reconnaissance, and most importantly - how to pivot through networks during penetration tests. With 20+ working examples, hands-on labs, and real OSCP scenarios, you'll master Linux networking from beginner to expert level.

---

## Table of Contents

1. [Network Fundamentals (Absolute Beginner)](#1-network-fundamentals-absolute-beginner)
2. [TCP/IP Protocol Stack](#2-tcpip-protocol-stack)
3. [IP Addressing and Subnetting](#3-ip-addressing-and-subnetting)
4. [Network Interface Configuration](#4-network-interface-configuration)
5. [Routing and the Routing Table](#5-routing-and-the-routing-table)
6. [DNS Configuration and Resolution](#6-dns-configuration-and-resolution)
7. [Network Troubleshooting Tools](#7-network-troubleshooting-tools)
8. [iptables: The Linux Firewall](#8-iptables-the-linux-firewall)
9. [nftables: The Modern Firewall](#9-nftables-the-modern-firewall)
10. [Network Namespaces and Container Networking](#10-network-namespaces-and-container-networking)
11. [Packet Analysis with tcpdump](#11-packet-analysis-with-tcpdump)
12. [Wireshark for Network Analysis](#12-wireshark-for-network-analysis)
13. [SSH Tunneling and Port Forwarding](#13-ssh-tunneling-and-port-forwarding)
14. [Network Pivoting Techniques](#14-network-pivoting-techniques)
15. [SOCKS Proxy and Proxychains](#15-socks-proxy-and-proxychains)
16. [VPN and Tunneling Tools](#16-vpn-and-tunneling-tools)
17. [Network Reconnaissance](#17-network-reconnaissance)
18. [Firewall Evasion Techniques](#18-firewall-evasion-techniques)
19. [Hands-On Labs](#19-hands-on-labs)
20. [Interview Questions and Answers](#20-interview-questions-and-answers)
21. [Summary and Next Steps](#21-summary-and-next-steps)

---

## 1. Network Fundamentals (Absolute Beginner)

### 1.1 What Is a Network?

A **network** is multiple computers connected together to share information.

**Real-world analogy**: Think of a network like a postal system:
- **Your computer**: A house with an address
- **IP Address**: The street address (123.45.67.89)
- **Router**: The local post office that forwards mail
- **Packets**: Individual letters being sent
- **Protocols**: The rules (write address on envelope, add stamp)

### 1.2 The Internet vs Local Network

```
┌─────────────────────────────────────────────────┐
│         NETWORK TYPES                           │
├─────────────────────────────────────────────────┤
│                                                 │
│  LOCAL NETWORK (LAN)                            │
│  ┌────────────────────────────────┐             │
│  │  Your Computer  192.168.1.10   │             │
│  │  Laptop         192.168.1.20   │             │
│  │  Printer        192.168.1.30   │             │
│  │         │                      │             │
│  │         ▼                      │             │
│  │  Router/Gateway 192.168.1.1    │             │
│  └───────────┬────────────────────┘             │
│              │                                  │
│              ▼                                  │
│  ┌───────────────────────┐                     │
│  │  INTERNET (WAN)       │                     │
│  │  Public IP addresses  │                     │
│  │  8.8.8.8 (Google DNS) │                     │
│  │  1.1.1.1 (Cloudflare) │                     │
│  └───────────────────────┘                     │
│                                                 │
└─────────────────────────────────────────────────┘
```

**Key concepts**:
- **LAN (Local Area Network)**: Your home/office network (192.168.x.x)
- **WAN (Wide Area Network)**: The internet
- **Gateway**: The router that connects LAN to WAN

### 1.3 What Is an IP Address?

An **IP address** is a unique identifier for a device on a network.

**IPv4 Format**: Four numbers (0-255) separated by dots
- Example: `192.168.1.100`
- Total possible: ~4 billion addresses

**IPv6 Format**: Eight groups of hexadecimal numbers
- Example: `2001:0db8:85a3:0000:0000:8a2e:0370:7334`
- Total possible: 340 undecillion addresses (we won't run out)

### 1.4 Private vs Public IP Addresses

**Private IP Ranges** (Not routable on the internet):
- `10.0.0.0` to `10.255.255.255` (10.0.0.0/8)
- `172.16.0.0` to `172.31.255.255` (172.16.0.0/12)
- `192.168.0.0` to `192.168.255.255` (192.168.0.0/16)

**Public IP Addresses**: Everything else (routable on internet)

**Why private addresses?**: Not enough IPv4 addresses for every device, so we use NAT (Network Address Translation) to share one public IP among many private IPs.

### 1.5 Basic Network Commands

```bash
# Check your IP address
ip addr
# or older command
ifconfig

# Check if a host is reachable
ping 8.8.8.8

# Trace route to destination
traceroute google.com

# Check network connections
netstat -tuln
# or modern version
ss -tuln

# DNS lookup
nslookup google.com
dig google.com
```

---

## 2. TCP/IP Protocol Stack

### 2.1 The OSI Model vs TCP/IP Model

```
┌──────────────────────────────────────────────┐
│  OSI MODEL (7 Layers)    TCP/IP MODEL        │
├──────────────────────────────────────────────┤
│  7. Application    ┐                         │
│  6. Presentation   ├──► Application          │
│  5. Session        ┘                         │
│  4. Transport      ───► Transport (TCP/UDP)  │
│  3. Network        ───► Internet (IP)        │
│  2. Data Link      ┐                         │
│  1. Physical       └──► Network Access       │
└──────────────────────────────────────────────┘

FOR OSCP: Focus on TCP/IP model
```

### 2.2 TCP vs UDP Explained

#### TCP (Transmission Control Protocol)

**Characteristics**:
- Connection-oriented (establishes connection first)
- Reliable (guarantees delivery)
- Ordered (packets arrive in order)
- Error-checked (corrupted packets retransmitted)
- Slower (overhead from reliability)

**Use cases**: HTTP, SSH, FTP, Email

**TCP Three-Way Handshake**:
```
CLIENT                    SERVER
  │                         │
  │──── SYN ──────────────► │  (Want to connect?)
  │                         │
  │ ◄─── SYN-ACK ─────────── │  (Yes, let's connect)
  │                         │
  │──── ACK ──────────────► │  (Connected!)
  │                         │
  │ ◄──── DATA ────────────► │  (Communication)
```

#### UDP (User Datagram Protocol)

**Characteristics**:
- Connectionless (no handshake)
- Unreliable (no delivery guarantee)
- Unordered (packets can arrive out of order)
- No error checking
- Faster (less overhead)

**Use cases**: DNS, DHCP, VoIP, Gaming, Streaming

**Example**:
```bash
# TCP connection (3-way handshake)
nc -nv 192.168.1.100 22  # SSH (TCP)

# UDP connection (no handshake)
nc -nvu 192.168.1.100 53  # DNS (UDP)
```

### 2.3 Common Ports Reference

| Port | Service | Protocol | Usage |
|:-----|:--------|:---------|:------|
| 20/21 | FTP | TCP | File transfer |
| 22 | SSH | TCP | Secure remote access |
| 23 | Telnet | TCP | Insecure remote access |
| 25 | SMTP | TCP | Email sending |
| 53 | DNS | UDP/TCP | Domain name resolution |
| 80 | HTTP | TCP | Web traffic |
| 110 | POP3 | TCP | Email retrieval |
| 143 | IMAP | TCP | Email access |
| 443 | HTTPS | TCP | Encrypted web |
| 445 | SMB | TCP | File sharing |
| 3306 | MySQL | TCP | Database |
| 3389 | RDP | TCP | Remote desktop |
| 8080 | HTTP-Alt | TCP | Alternative web |

### 2.4 The IP Packet Structure

```
┌──────────────────────────────────────┐
│         IP PACKET HEADER             │
├──────────────────────────────────────┤
│  Version (4 or 6)                    │
│  Header Length                       │
│  Type of Service (QoS)               │
│  Total Length                        │
│  Identification (for fragmentation)  │
│  Flags                               │
│  Fragment Offset                     │
│  Time To Live (TTL) ◄─ Important!    │
│  Protocol (TCP=6, UDP=17, ICMP=1)    │
│  Header Checksum                     │
│  Source IP Address                   │
│  Destination IP Address              │
│  Options (if any)                    │
│  ─────────────────────               │
│  DATA (Payload)                      │
└──────────────────────────────────────┘
```

**TTL (Time To Live)**: Prevents packets from looping forever. Decremented by each router; packet dropped when TTL=0.

---

## 3. IP Addressing and Subnetting

### 3.1 Understanding Subnet Masks

A **subnet mask** defines which part of an IP address is the network and which part is the host.

**Example**:
- IP: `192.168.1.100`
- Subnet Mask: `255.255.255.0`
- Network: `192.168.1.0`
- Host: `100`

### 3.2 CIDR Notation

**CIDR (Classless Inter-Domain Routing)**: Shorthand for subnet masks

```
255.255.255.0   = /24 (24 bits for network)
255.255.0.0     = /16
255.0.0.0       = /8
255.255.255.128 = /25
```

**Example**: `192.168.1.0/24` means:
- Network: 192.168.1.0
- Usable IPs: 192.168.1.1 to 192.168.1.254
- Broadcast: 192.168.1.255
- Total hosts: 254 (256 - 2 for network and broadcast)

### 3.3 Common Subnet Masks

| CIDR | Subnet Mask | Usable Hosts | Use Case |
|:-----|:------------|:-------------|:---------|
| /8 | 255.0.0.0 | 16,777,214 | Huge networks |
| /16 | 255.255.0.0 | 65,534 | Large organizations |
| /24 | 255.255.255.0 | 254 | Home/small office |
| /25 | 255.255.255.128 | 126 | Small subnets |
| /30 | 255.255.255.252 | 2 | Point-to-point links |
| /32 | 255.255.255.255 | 1 | Single host |

### 3.4 Calculating Subnets

**Question**: What is the network address for `192.168.1.130/25`?

**Solution**:
- /25 = 255.255.255.128
- Binary: 11111111.11111111.11111111.10000000
- Network bits: First 25 bits
- Host bits: Last 7 bits

```
IP:      192.168.1.130    = 10000010 (last octet)
Mask:    255.255.255.128  = 10000000
                            ────────
Network: 192.168.1.128    = 10000000
```

**Answer**: Network = `192.168.1.128/25`, Range = 192.168.1.128-255

### 3.5 Private Networks for OSCP

In OSCP labs, you'll see these private ranges:
- **10.10.10.0/24**: Common HTB/OSCP range
- **192.168.X.0/24**: Your local network
- **172.16.0.0/16**: Enterprise internal networks

---

## 4. Network Interface Configuration

### 4.1 Modern Tool: ip Command (iproute2)

The `ip` command replaces old tools like `ifconfig`, `route`, `arp`.

**Key subcommands**:
- `ip addr` - Manage IP addresses
- `ip link` - Manage network interfaces
- `ip route` - Manage routing table
- `ip neigh` - Manage ARP table
- `ip netns` - Manage network namespaces

### 4.2 Viewing Network Interfaces

```bash
# List all interfaces
ip addr show
# or shorter
ip a

# Example output:
# 1: lo: <LOOPBACK,UP,LOWER_UP>
#     inet 127.0.0.1/8
# 2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP>
#     inet 192.168.1.100/24 brd 192.168.1.255

# Show specific interface
ip addr show eth0

# Show only IPv4
ip -4 addr

# Show only IPv6
ip -6 addr
```

### 4.3 Configuring IP Addresses

```bash
# Add IP address to interface
sudo ip addr add 192.168.1.50/24 dev eth0

# Add multiple IPs to same interface
sudo ip addr add 192.168.1.51/24 dev eth0
sudo ip addr add 192.168.1.52/24 dev eth0

# Remove IP address
sudo ip addr del 192.168.1.50/24 dev eth0

# Enable interface
sudo ip link set eth0 up

# Disable interface
sudo ip link set eth0 down
```

### 4.4 Persistent Configuration

#### Debian/Ubuntu (/etc/network/interfaces)

```bash
# Edit config
sudo nano /etc/network/interfaces

# Static IP configuration
auto eth0
iface eth0 inet static
    address 192.168.1.100
    netmask 255.255.255.0
    gateway 192.168.1.1
    dns-nameservers 8.8.8.8 8.8.4.4

# DHCP configuration
auto eth0
iface eth0 inet dhcp

# Restart networking
sudo systemctl restart networking
```

#### Red Hat/CentOS (/etc/sysconfig/network-scripts/)

```bash
# Edit interface config
sudo nano /etc/sysconfig/network-scripts/ifcfg-eth0

# Content:
DEVICE=eth0
BOOTPROTO=static
IPADDR=192.168.1.100
NETMASK=255.255.255.0
GATEWAY=192.168.1.1
DNS1=8.8.8.8
ONBOOT=yes

# Restart network
sudo systemctl restart network
```

#### Modern: netplan (Ubuntu 18.04+)

```bash
# Edit netplan config
sudo nano /etc/netplan/01-netcfg.yaml

# YAML format:
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: no
      addresses:
        - 192.168.1.100/24
      gateway4: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]

# Apply config
sudo netplan apply
```

### 4.5 Interface Types

```bash
# Physical interface
eth0, ens33, enp0s3

# Loopback (localhost)
lo - 127.0.0.1

# Wireless
wlan0, wlp2s0

# Virtual/Bridge
br0, virbr0

# Tunnel
tun0, tap0 (VPN interfaces)

# Docker
docker0

# Check interface type
ip link show
```

---

## 5. Routing and the Routing Table

### 5.1 What Is Routing?

**Routing** determines the path packets take from source to destination.

```
┌──────────────────────────────────────────┐
│         ROUTING EXAMPLE                  │
├──────────────────────────────────────────┤
│  Your PC (192.168.1.10)                  │
│      │                                   │
│      ▼                                   │
│  Router/Gateway (192.168.1.1)            │
│      │                                   │
│      ▼                                   │
│  ISP Router                              │
│      │                                   │
│      ▼                                   │
│  Internet Routers (multiple hops)        │
│      │                                   │
│      ▼                                   │
│  Destination Server (8.8.8.8)            │
└──────────────────────────────────────────┘
```

### 5.2 Viewing the Routing Table

```bash
# Modern way
ip route show
# or
ip r

# Example output:
# default via 192.168.1.1 dev eth0 
# 192.168.1.0/24 dev eth0 proto kernel scope link src 192.168.1.100

# Older way (deprecated but still works)
route -n
netstat -rn
```

**Understanding the output**:
- `default via 192.168.1.1` - Default gateway (for internet)
- `192.168.1.0/24 dev eth0` - Local network directly connected
- `proto kernel` - Added by kernel automatically
- `src 192.168.1.100` - Source IP for this route

### 5.3 Adding Static Routes

```bash
# Add route to specific network
sudo ip route add 10.10.10.0/24 via 192.168.1.1

# Add route through specific interface
sudo ip route add 172.16.0.0/16 dev eth1

# Add default gateway
sudo ip route add default via 192.168.1.1

# Delete route
sudo ip route del 10.10.10.0/24

# Replace route
sudo ip route replace 10.10.10.0/24 via 192.168.1.2
```

### 5.4 OSCP Routing Scenario

**Scenario**: You compromised a dual-homed server (two network interfaces):
- eth0: 10.10.14.5 (your attacker network)
- eth1: 172.16.50.1 (internal network)

**Goal**: Access internal network (172.16.50.0/24) from your attack machine.

**Solution**:
```bash
# On your attack machine, add route through compromised server
sudo ip route add 172.16.50.0/24 via 10.10.14.5

# Now you can access internal hosts
ping 172.16.50.10
nmap 172.16.50.0/24
```

### 5.5 Enabling IP Forwarding

For a Linux box to act as a router, IP forwarding must be enabled.

```bash
# Check current status
cat /proc/sys/net/ipv4/ip_forward
# 0 = disabled, 1 = enabled

# Enable temporarily (until reboot)
sudo sysctl -w net.ipv4.ip_forward=1
# or
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward

# Enable permanently
sudo nano /etc/sysctl.conf
# Add or uncomment:
net.ipv4.ip_forward=1

# Apply changes
sudo sysctl -p
```

**OSCP Use**: Turn compromised Linux box into a router for pivoting.

---

## 6. DNS Configuration and Resolution

### 6.1 What Is DNS?

**DNS (Domain Name System)** translates human-readable names to IP addresses.

```
You type: www.google.com
DNS resolves to: 142.250.185.46
```

### 6.2 DNS Resolution Process

```
┌────────────────────────────────────────────┐
│       DNS RESOLUTION FLOW                  │
├────────────────────────────────────────────┤
│  1. Your Computer                          │
│     "What is google.com?"                  │
│              ↓                             │
│  2. Check /etc/hosts (local file)          │
│     Not found → Continue                   │
│              ↓                             │
│  3. Check /etc/resolv.conf (DNS servers)   │
│     nameserver 8.8.8.8                     │
│              ↓                             │
│  4. Query DNS Server (8.8.8.8)             │
│              ↓                             │
│  5. Recursive lookup if needed             │
│     Root → TLD → Authoritative             │
│              ↓                             │
│  6. Return IP: 142.250.185.46              │
└────────────────────────────────────────────┘
```

### 6.3 DNS Configuration Files

#### /etc/hosts

Static hostname to IP mappings (checked first):

```bash
# View hosts file
cat /etc/hosts

# Example content:
127.0.0.1   localhost
127.0.1.1   mycomputer
192.168.1.100   webserver.local
10.10.10.10     target.htb

# Add custom entry
echo "10.10.10.50 mybox.htb" | sudo tee -a /etc/hosts

# Test
ping mybox.htb
```

**OSCP Use**: Add HTB/lab machine names to /etc/hosts for easier access.

#### /etc/resolv.conf

DNS server configuration:

```bash
# View DNS servers
cat /etc/resolv.conf

# Example:
nameserver 8.8.8.8
nameserver 8.8.4.4
search local.domain

# Temporarily change DNS
sudo nano /etc/resolv.conf
# Add:
nameserver 1.1.1.1
```

**Note**: Many systems use `systemd-resolved` or `NetworkManager` which auto-generates `/etc/resolv.conf`.

### 6.4 DNS Lookup Tools

```bash
# Simple lookup
nslookup google.com

# Detailed lookup
dig google.com

# Specific record types
dig google.com A      # IPv4 address
dig google.com AAAA   # IPv6 address
dig google.com MX     # Mail servers
dig google.com NS     # Name servers
dig google.com TXT    # Text records

# Query specific DNS server
dig @8.8.8.8 google.com

# Reverse DNS lookup
dig -x 8.8.8.8

# Short answer only
dig +short google.com
```

### 6.5 DNS Cache

```bash
# View systemd-resolved cache
systemd-resolve --statistics

# Flush DNS cache (systemd)
sudo systemd-resolve --flush-caches

# Flush DNS cache (nscd)
sudo /etc/init.d/nscd restart

# Flush DNS cache (dnsmasq)
sudo killall -HUP dnsmasq
```

---

## 7. Network Troubleshooting Tools

### 7.1 ping - ICMP Echo Request

```bash
# Basic ping
ping google.com

# Ping specific count
ping -c 4 google.com

# Flood ping (requires root)
sudo ping -f target

# Change packet size
ping -s 1000 target

# Set TTL
ping -t 64 target

# Ping sweep (check if host is up)
for i in {1..254}; do ping -c 1 192.168.1.$i | grep "64 bytes" & done
```

**OSCP**: Use ping to check if targets are alive.

### 7.2 traceroute - Path Discovery

```bash
# Trace route to destination
traceroute google.com

# Use ICMP instead of UDP
traceroute -I google.com

# Don't resolve hostnames (faster)
traceroute -n 8.8.8.8

# Set max hops
traceroute -m 15 target
```

**Shows each hop (router) between you and destination.**

### 7.3 netstat - Network Statistics

```bash
# Show all listening ports
netstat -tuln

# Show all connections
netstat -tun

# Show processes using ports
sudo netstat -tulpn

# Show routing table
netstat -rn

# Continuous refresh
netstat -c

# TCP connections only
netstat -t

# UDP connections only
netstat -u
```

### 7.4 ss - Socket Statistics (Modern netstat)

```bash
# Show all listening TCP ports
ss -tln

# Show all connections
ss -tun

# Show processes
ss -tulpn

# Show established connections
ss -o state established

# Filter by port
ss -tln sport = :80
ss -tln dport = :443

# Show socket memory usage
ss -tm
```

### 7.5 nc (netcat) - The Swiss Army Knife

```bash
# Connect to port
nc 192.168.1.100 80

# Listen on port
nc -lvnp 4444

# Send file
nc -w3 192.168.1.100 4444 < file.txt

# Receive file
nc -lvnp 4444 > received.txt

# Port scanning
nc -zv 192.168.1.100 1-1000

# Banner grabbing
nc -v 192.168.1.100 22

# UDP connection
nc -u 192.168.1.100 53
```

### 7.6 nmap - Network Mapper

```bash
# Basic scan
nmap 192.168.1.100

# Scan all ports
nmap -p- 192.168.1.100

# Service version detection
nmap -sV 192.168.1.100

# OS detection
sudo nmap -O 192.168.1.100

# Fast scan (top 100 ports)
nmap -F 192.168.1.100

# Scan network range
nmap 192.168.1.0/24

# Output to file
nmap -oN scan.txt 192.168.1.100
```

---

## 8. iptables: The Linux Firewall

### 8.1 Understanding iptables

**iptables** is the Linux firewall that filters network packets.

```
┌──────────────────────────────────────────────┐
│         IPTABLES PACKET FLOW                 │
├──────────────────────────────────────────────┤
│                                              │
│  INCOMING PACKET                             │
│       │                                      │
│       ▼                                      │
│  ┌──────────────┐                            │
│  │ PREROUTING   │  (DNAT here)               │
│  └──────┬───────┘                            │
│         │                                    │
│    ┌────▼────┐                               │
│    │ ROUTING │                               │
│    │ DECISION│                               │
│    └────┬────┘                               │
│         │                                    │
│    ┌────▼──────────┐                         │
│    │ For me?       │                         │
│    └───┬──────┬────┘                         │
│        │      │                              │
│   YES  │      │  NO                          │
│        │      │                              │
│        ▼      ▼                              │
│   ┌─────┐  ┌─────────┐                      │
│   │INPUT│  │ FORWARD │  (Routing through)   │
│   └──┬──┘  └────┬────┘                      │
│      │          │                            │
│      ▼          │                            │
│  Local Process  │                            │
│      │          │                            │
│      ▼          ▼                            │
│  ┌────────┐  ┌──────────────┐               │
│  │ OUTPUT │  │ POSTROUTING  │  (SNAT here)  │
│  └────┬───┘  └──────┬───────┘               │
│       │             │                        │
│       └─────────────┘                        │
│             │                                │
│             ▼                                │
│      OUTGOING PACKET                         │
│                                              │
└──────────────────────────────────────────────┘
```

### 8.2 iptables Syntax

```bash
iptables -t [TABLE] -A [CHAIN] [MATCH] -j [TARGET]

# Tables: filter (default), nat, mangle, raw
# Chains: INPUT, OUTPUT, FORWARD, PREROUTING, POSTROUTING
# Targets: ACCEPT, DROP, REJECT, LOG, MASQUERADE, DNAT, SNAT
```

### 8.3 Basic iptables Rules

```bash
# List all rules
sudo iptables -L -v -n

# List rules with line numbers
sudo iptables -L --line-numbers

# List NAT rules
sudo iptables -t nat -L -v -n

# Allow incoming SSH
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Allow incoming HTTP/HTTPS
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Allow established connections
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Allow loopback
sudo iptables -A INPUT -i lo -j ACCEPT

# Block specific IP
sudo iptables -A INPUT -s 192.168.1.50 -j DROP

# Default policy (drop everything not explicitly allowed)
sudo iptables -P INPUT DROP
sudo iptables -P FORWARD DROP
sudo iptables -P OUTPUT ACCEPT
```

### 8.4 Port Forwarding with iptables

```bash
# Forward port 80 to 8080 (run service as non-root)
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8080

# Forward traffic to another host
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination 192.168.1.100:8080

# SNAT (change source IP)
sudo iptables -t nat -A POSTROUTING -o eth0 -j SNAT --to-source 203.0.113.1

# MASQUERADE (dynamic SNAT for DHCP)
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

### 8.5 iptables for OSCP

**Allow reverse shell connections**:
```bash
# Allow outbound connections on port 4444
sudo iptables -A OUTPUT -p tcp --dport 4444 -j ACCEPT
sudo iptables -A INPUT -p tcp --sport 4444 -m state --state ESTABLISHED -j ACCEPT
```

**Delete rules**:
```bash
# Delete by line number
sudo iptables -D INPUT 3

# Delete specific rule
sudo iptables -D INPUT -p tcp --dport 22 -j ACCEPT

# Flush all rules
sudo iptables -F

# Flush NAT rules
sudo iptables -t nat -F
```

**Save and restore**:
```bash
# Save current rules
sudo iptables-save > /tmp/iptables.rules

# Restore rules
sudo iptables-restore < /tmp/iptables.rules

# Persistent (Debian/Ubuntu)
sudo apt install iptables-persistent
sudo netfilter-persistent save
sudo netfilter-persistent reload
```

---

## 9. nftables: The Modern Firewall

### 9.1 What Is nftables?

**nftables** is the modern replacement for iptables, combining IPv4, IPv6, ARP, and bridge filtering.

**Advantages**:
- Simpler syntax
- Better performance
- One tool for all protocols
- Atomic rule updates

### 9.2 Basic nftables Commands

```bash
# List all rules
sudo nft list ruleset

# Create table
sudo nft add table inet filter

# Create chains
sudo nft add chain inet filter input { type filter hook input priority 0 \; }
sudo nft add chain inet filter output { type filter hook output priority 0 \; }

# Add rules
sudo nft add rule inet filter input tcp dport 22 accept
sudo nft add rule inet filter input tcp dport 80 accept

# Delete rule
sudo nft delete rule inet filter input handle 5

# Flush all rules
sudo nft flush ruleset
```

### 9.3 nftables Example Configuration

```bash
# Create complete firewall
sudo nft -f /etc/nftables.conf

# Example config:
table inet filter {
    chain input {
        type filter hook input priority 0; policy drop;
        
        # Allow established
        ct state established,related accept
        
        # Allow loopback
        iif lo accept
        
        # Allow SSH
        tcp dport 22 accept
        
        # Allow HTTP/HTTPS
        tcp dport { 80, 443 } accept
    }
    
    chain output {
        type filter hook output priority 0; policy accept;
    }
}
```

---

## 10. Network Namespaces and Container Networking

### 10.1 What Are Network Namespaces?

**Network namespaces** isolate network stacks - each namespace has its own interfaces, routing tables, firewall rules.

**Use case**: Docker containers use network namespaces for isolation.

### 10.2 Creating and Using Namespaces

```bash
# Create namespace
sudo ip netns add blue

# List namespaces
ip netns list

# Execute command in namespace
sudo ip netns exec blue ip addr

# Run shell in namespace
sudo ip netns exec blue bash

# Delete namespace
sudo ip netns del blue
```

### 10.3 Connecting Namespaces with veth Pairs

```bash
# Create two namespaces
sudo ip netns add red
sudo ip netns add blue

# Create veth pair (virtual ethernet cable)
sudo ip link add veth-red type veth peer name veth-blue

# Attach each end to a namespace
sudo ip link set veth-red netns red
sudo ip link set veth-blue netns blue

# Configure IPs
sudo ip netns exec red ip addr add 10.0.0.1/24 dev veth-red
sudo ip netns exec blue ip addr add 10.0.0.2/24 dev veth-blue

# Bring interfaces up
sudo ip netns exec red ip link set veth-red up
sudo ip netns exec blue ip link set veth-blue up

# Test connectivity
sudo ip netns exec red ping 10.0.0.2
```

### 10.4 Docker Networking Basics

```bash
# List Docker networks
docker network ls

# Inspect network
docker network inspect bridge

# Create custom network
docker network create --subnet=172.18.0.0/16 mynetwork

# Run container on specific network
docker run --network=mynetwork nginx

# Connect container to network
docker network connect mynetwork container_name
```

---

## 11. Packet Analysis with tcpdump

### 11.1 What Is tcpdump?

**tcpdump** captures and analyzes network packets on the command line.

**OSCP Use**: Capture network traffic, debug exploits, analyze protocols.

### 11.2 Basic tcpdump Usage

```bash
# Capture on interface
sudo tcpdump -i eth0

# Capture specific number of packets
sudo tcpdump -c 100

# Save to file
sudo tcpdump -w capture.pcap

# Read from file
tcpdump -r capture.pcap

# Verbose output
sudo tcpdump -v
sudo tcpdump -vv
sudo tcpdump -vvv
```

### 11.3 tcpdump Filters

```bash
# Capture specific host
sudo tcpdump host 192.168.1.100

# Capture specific port
sudo tcpdump port 80

# Capture TCP only
sudo tcpdump tcp

# Capture UDP only
sudo tcpdump udp

# Capture specific network
sudo tcpdump net 192.168.1.0/24

# Capture source/destination
sudo tcpdump src 192.168.1.100
sudo tcpdump dst 192.168.1.100

# Combine filters (AND)
sudo tcpdump host 192.168.1.100 and port 80

# OR filter
sudo tcpdump port 80 or port 443

# NOT filter
sudo tcpdump not port 22
```

### 11.4 Advanced tcpdump Examples

```bash
# Capture HTTP requests
sudo tcpdump -A -s 0 'tcp port 80 and (((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0)'

# Capture passwords (cleartext protocols)
sudo tcpdump port 21 or port 23 or port 110 -A

# Capture DNS queries
sudo tcpdump -i eth0 -n port 53

# Capture ICMP (ping)
sudo tcpdump icmp

# Capture SYN packets (port scan detection)
sudo tcpdump 'tcp[tcpflags] & tcp-syn != 0'

# Capture packets with specific size
sudo tcpdump greater 1000
sudo tcpdump less 100
```

### 11.5 OSCP tcpdump Scenarios

**Capture reverse shell traffic**:
```bash
# On attack machine
sudo tcpdump -i tun0 -w shell_capture.pcap port 4444

# Start listener
nc -lvnp 4444

# Trigger exploit
# Analyze captured traffic
tcpdump -r shell_capture.pcap -A
```

**Debug why exploit isn't working**:
```bash
# Capture all traffic to/from target
sudo tcpdump -i tun0 host 10.10.10.50 -w debug.pcap

# Run exploit
# Analyze what happened
tcpdump -r debug.pcap
```

---

## 12. Wireshark for Network Analysis

### 12.1 Wireshark Display Filters

```bash
# Filter by IP
ip.addr == 192.168.1.100
ip.src == 192.168.1.100
ip.dst == 192.168.1.100

# Filter by port
tcp.port == 80
tcp.srcport == 4444
tcp.dstport == 443

# Filter by protocol
http
dns
ssh
ftp

# TCP flags
tcp.flags.syn == 1
tcp.flags.rst == 1

# HTTP methods
http.request.method == "GET"
http.request.method == "POST"

# Follow TCP stream
tcp.stream eq 0

# Contains string
frame contains "password"
http contains "admin"
```

### 12.2 Wireshark for OSCP

**Analyze HTTP traffic**:
1. Capture traffic: `sudo tcpdump -i eth0 -w http.pcap port 80`
2. Open in Wireshark: `wireshark http.pcap`
3. Filter: `http`
4. Right-click packet → Follow → HTTP Stream
5. See full HTTP request/response

**Extract files from packet capture**:
1. File → Export Objects → HTTP
2. Save suspicious files

**Find credentials**:
1. Filter: `ftp or telnet or http.authbasic`
2. Look for usernames and passwords

---

## 13. SSH Tunneling and Port Forwarding

### 13.1 SSH Local Port Forwarding

**Forward local port to remote service.**

```bash
# Syntax
ssh -L [local_port]:[destination_host]:[destination_port] user@ssh_server

# Example: Access remote database
ssh -L 3306:localhost:3306 user@remote_server
# Now: mysql -h localhost -P 3306 connects to remote MySQL

# Access internal web server through jump host
ssh -L 8080:internal_server:80 user@jump_host
# Now: http://localhost:8080 → internal_server:80
```

**OSCP Example**:
```bash
# Target has internal web server on 127.0.0.1:8000
# You have SSH access to target
ssh -L 9000:127.0.0.1:8000 user@target

# On your browser: http://localhost:9000
# → accesses target's internal web server
```

### 13.2 SSH Remote Port Forwarding

**Forward remote port to local service.**

```bash
# Syntax
ssh -R [remote_port]:[local_host]:[local_port] user@remote_server

# Example: Expose your local web server
ssh -R 8080:localhost:80 user@remote_server
# Now: remote_server:8080 → your localhost:80
```

**OSCP Reverse Shell Alternative**:
```bash
# On attack machine (start SSH server)
sudo systemctl start sshd

# From target (connect back with reverse tunnel)
ssh -R 4444:localhost:4444 attacker@10.10.14.5

# On target, send reverse shell to localhost:4444
# It gets forwarded to attacker's machine
```

### 13.3 SSH Dynamic Port Forwarding (SOCKS Proxy)

**Create SOCKS proxy through SSH.**

```bash
# Syntax
ssh -D [local_port] user@ssh_server

# Example
ssh -D 9050 user@remote_server

# Configure proxychains
nano /etc/proxychains.conf
# Add: socks5 127.0.0.1 9050

# Use proxy
proxychains nmap -sT 172.16.50.0/24
proxychains curl http://internal-website.local
```

**OSCP Pivoting**:
```bash
# SSH to compromised server
ssh -D 9050 user@10.10.14.5

# Scan internal network through proxy
proxychains nmap -sT -Pn 172.16.50.0/24

# Access internal web apps
proxychains firefox
# Browse to 172.16.50.10
```

### 13.4 SSH ProxyJump (Jump Host)

```bash
# SSH through intermediate host
ssh -J jump_user@jump_host final_user@final_destination

# Multiple jumps
ssh -J jump1,jump2,jump3 user@final_destination

# In SSH config (~/.ssh/config)
Host final_destination
    ProxyJump jump_host
```

---

## 14. Network Pivoting Techniques

### 14.1 What Is Pivoting?

**Pivoting**: Using a compromised machine to attack other machines not directly accessible from your attack machine.

```
┌───────────────────────────────────────────────┐
│          NETWORK PIVOTING SCENARIO            │
├───────────────────────────────────────────────┤
│                                               │
│  [Attacker]  ──────► [DMZ Server] ──────►    │
│  10.10.14.5          10.10.10.50              │
│                      192.168.50.1 (internal)  │
│                             │                 │
│                             │                 │
│                             ▼                 │
│                      [Internal Network]       │
│                      192.168.50.0/24          │
│                      - Database Server        │
│                      - File Server            │
│                      - Domain Controller      │
│                                               │
│  WITHOUT PIVOTING:                            │
│  Cannot reach 192.168.50.0/24 directly        │
│                                               │
│  WITH PIVOTING:                               │
│  Use DMZ Server as jump point                │
│                                               │
└───────────────────────────────────────────────┘
```

### 14.2 Pivoting with SSH

```bash
# Method 1: SSH Dynamic Forwarding (SOCKS)
ssh -D 9050 user@10.10.10.50

# Configure proxychains
echo "socks5 127.0.0.1 9050" >> /etc/proxychains.conf

# Scan internal network
proxychains nmap -sT 192.168.50.0/24

# Method 2: SSH Local Port Forwarding
ssh -L 3389:192.168.50.10:3389 user@10.10.10.50

# Connect to RDP
rdesktop localhost:3389

# Method 3: Add route + IP forwarding
# On compromised server (10.10.10.50)
echo 1 > /proc/sys/net/ipv4/ip_forward
iptables -t nat -A POSTROUTING -o eth1 -j MASQUERADE

# On attacker
sudo ip route add 192.168.50.0/24 via 10.10.10.50
```

### 14.3 Pivoting with Chisel

**Chisel** is a fast TCP/UDP tunnel over HTTP secured via SSH.

```bash
# Download chisel
wget https://github.com/jpillora/chisel/releases/download/v1.7.7/chisel_1.7.7_linux_amd64.gz
gunzip chisel_1.7.7_linux_amd64.gz
chmod +x chisel_1.7.7_linux_amd64

# On attacker (server mode)
./chisel server -p 8000 --reverse

# On compromised host (client mode)
./chisel client 10.10.14.5:8000 R:socks

# Configure proxychains
echo "socks5 127.0.0.1 1080" >> /etc/proxychains.conf

# Use proxy
proxychains nmap -sT 192.168.50.0/24
```

### 14.4 Pivoting with sshuttle

**sshuttle** creates a transparent VPN over SSH.

```bash
# Install
sudo apt install sshuttle

# Create VPN to internal network
sshuttle -r user@10.10.10.50 192.168.50.0/24

# Now access internal network directly (no proxychains needed)
nmap 192.168.50.10
curl http://192.168.50.20
```

---

## 15. SOCKS Proxy and Proxychains

### 15.1 Configuring Proxychains

```bash
# Edit config
sudo nano /etc/proxychains.conf

# Uncomment dynamic_chain (tries proxies in order)
dynamic_chain

# Comment out strict_chain

# Add SOCKS proxy at end
socks5  127.0.0.1 9050
```

### 15.2 Using Proxychains

```bash
# General syntax
proxychains [command]

# Examples
proxychains nmap -sT -Pn 192.168.50.10
proxychains curl http://internal-site.local
proxychains firefox
proxychains msfconsole

# Quiet mode (less output)
proxychains -q nmap -sT 192.168.50.10
```

### 15.3 ProxyChains Limitations

**Won't work with**:
- Ping (ICMP)
- Nmap SYN scan (-sS) - use -sT (TCP connect)
- UDP scans

**Solutions**:
- Use `-sT` for Nmap (TCP connect scan)
- Use `fping` through proxy for host discovery
- Use TCP-based tools only

---

## 16. VPN and Tunneling Tools

### 16.1 OpenVPN

```bash
# Connect to VPN
sudo openvpn config.ovpn

# Connect in background
sudo openvpn --config config.ovpn --daemon

# Check VPN interface
ip addr show tun0

# Check VPN routes
ip route | grep tun0
```

### 16.2 Ligolo-ng (Modern Pivoting)

```bash
# On attacker (proxy)
./proxy -selfcert

# On target (agent)
./agent -connect 10.10.14.5:11601 -ignore-cert

# In proxy console
session
start

# Add route on attacker
sudo ip route add 192.168.50.0/24 dev ligolo
```

---

## 17. Network Reconnaissance

### 17.1 Host Discovery

```bash
# Ping sweep
nmap -sn 192.168.1.0/24

# ARP scan (local network only)
sudo arp-scan -l

# Netdiscover
sudo netdiscover -r 192.168.1.0/24

# Fping
fping -ag 192.168.1.0/24
```

### 17.2 Port Scanning

```bash
# Fast scan
nmap -F 192.168.1.100

# All ports
nmap -p- 192.168.1.100

# Specific ports
nmap -p 22,80,443 192.168.1.100

# Service detection
nmap -sV 192.168.1.100

# OS detection
sudo nmap -O 192.168.1.100
```

---

## 18. Firewall Evasion Techniques

### 18.1 Fragmentation

```bash
# Fragment packets
nmap -f target

# Custom MTU
nmap --mtu 8 target
```

### 18.2 Decoy Scans

```bash
# Use decoys
nmap -D RND:10 target

# Specific decoys
nmap -D 192.168.1.5,192.168.1.6,ME target
```

### 18.3 Source Port Manipulation

```bash
# Use specific source port (53, 80, 443 often allowed)
nmap --source-port 53 target
```

---

## 19. Hands-On Labs

### Lab 1: Configure Static IP

**Objective**: Set up static IP address.

```bash
# Check current config
ip addr show

# Add static IP
sudo ip addr add 192.168.1.50/24 dev eth0

# Add default gateway
sudo ip route add default via 192.168.1.1

# Test
ping 8.8.8.8
```

### Lab 2: SSH Port Forwarding

**Objective**: Access internal service through SSH.

```bash
# Setup: Remote server has MySQL on localhost:3306
# Goal: Access it from your machine

# Create tunnel
ssh -L 3306:localhost:3306 user@remote_server

# Connect
mysql -h localhost -P 3306 -u root -p
```

### Lab 3: Network Pivoting

**Objective**: Access internal network through compromised host.

```bash
# Compromised host: 10.10.10.50 (external), 192.168.50.1 (internal)
# Goal: Scan 192.168.50.0/24

# Method: SSH Dynamic Forwarding
ssh -D 9050 user@10.10.10.50

# Configure proxychains
echo "socks5 127.0.0.1 9050" | sudo tee -a /etc/proxychains.conf

# Scan
proxychains nmap -sT 192.168.50.0/24
```

### Lab 4: iptables Port Forwarding

**Objective**: Forward port 80 to 8080.

```bash
# Enable IP forwarding
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward

# Add rule
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8080

# Test
curl http://localhost:80
# Should connect to service on port 8080
```

### Lab 5: Packet Capture

**Objective**: Capture HTTP traffic.

```bash
# Start capture
sudo tcpdump -i eth0 -w http.pcap port 80

# Generate traffic
curl http://example.com

# Stop capture (Ctrl+C)

# Analyze
tcpdump -r http.pcap -A | grep -i "host:"
```

---

## 20. Interview Questions and Answers

### Q1: Explain the difference between TCP and UDP.

**Answer**:
- **TCP**: Connection-oriented, reliable, ordered, error-checked. Uses 3-way handshake. Slower but guarantees delivery. Used for HTTP, SSH, FTP.
- **UDP**: Connectionless, unreliable, no ordering, no error checking. Faster with less overhead. Used for DNS, DHCP, streaming, gaming.

---

### Q2: What is the purpose of a subnet mask?

**Answer**:
A subnet mask determines which portion of an IP address represents the network and which represents the host. For example, with IP 192.168.1.100 and mask 255.255.255.0 (/24):
- Network: 192.168.1.0
- Host: 100
- Network has 254 usable addresses (192.168.1.1-254)

---

### Q3: How would you pivot through a compromised Linux machine to access an internal network?

**Answer**:
Multiple methods:
1. **SSH Dynamic Forwarding**: `ssh -D 9050 user@compromised_host` then use proxychains
2. **Add Route**: Enable IP forwarding on compromised host, add route on attacker: `ip route add internal_network via compromised_host_ip`
3. **sshuttle**: `sshuttle -r user@compromised_host internal_network`
4. **Chisel**: Set up reverse SOCKS tunnel

---

### Q4: What is the difference between SNAT and DNAT?

**Answer**:
- **SNAT (Source NAT)**: Changes source IP address. Used when internal hosts access the internet (masquerading). Happens in POSTROUTING chain.
- **DNAT (Destination NAT)**: Changes destination IP address. Used for port forwarding to internal servers. Happens in PREROUTING chain.

---

### Q5: How do you enable IP forwarding on Linux and why is it needed for pivoting?

**Answer**:
```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
# or
sysctl -w net.ipv4.ip_forward=1
```

IP forwarding allows a Linux box to route packets between interfaces. Without it, packets arriving on one interface destined for another network are dropped. This is essential for using a compromised host as a router/pivot point.

---

### Q6: What does the iptables FORWARD chain control?

**Answer**:
The FORWARD chain controls packets that are routed THROUGH the system (not destined for the system itself). It's used when the Linux box acts as a router/firewall between networks. Traffic going IN or OUT of the system itself uses INPUT and OUTPUT chains.

---

### Q7: How would you capture only SSH traffic with tcpdump?

**Answer**:
```bash
sudo tcpdump -i eth0 port 22
# or
sudo tcpdump -i eth0 tcp port 22
```

To save to file for later analysis:
```bash
sudo tcpdump -i eth0 -w ssh_capture.pcap port 22
```

---

### Q8: What is the purpose of /etc/hosts and when would you use it in OSCP?

**Answer**:
`/etc/hosts` provides static hostname-to-IP mappings checked before DNS resolution. In OSCP/HTB, you add machine names for easier access:
```bash
echo "10.10.10.50 target.htb" | sudo tee -a /etc/hosts
```
Now you can use `target.htb` instead of remembering the IP.

---

### Q9: Explain SSH local port forwarding with an example.

**Answer**:
SSH local port forwarding creates a tunnel from your local machine to a remote service through an SSH server.

**Example**: Access a database on a remote server:
```bash
ssh -L 3306:localhost:3306 user@remote_server
```

This forwards your local port 3306 to the remote server's localhost:3306. Now `mysql -h localhost` connects to the remote database.

---

### Q10: How do you check what ports are listening on a Linux system?

**Answer**:
Multiple methods:
```bash
# Modern (ss)
ss -tuln

# Traditional (netstat)
netstat -tuln

# With process names
sudo ss -tulpn
sudo netstat -tulpn

# Specific port
sudo lsof -i :80
```

---

## 21. Summary and Next Steps

### 21.1 What We Covered

You've learned:
- ✅ Network fundamentals (IP, TCP/UDP, ports)
- ✅ IP addressing and subnetting
- ✅ Network interface configuration (ip command)
- ✅ Routing table management
- ✅ DNS configuration and resolution
- ✅ Network troubleshooting tools
- ✅ iptables firewall configuration
- ✅ nftables modern firewall
- ✅ Network namespaces
- ✅ Packet analysis (tcpdump, Wireshark)
- ✅ SSH tunneling and port forwarding
- ✅ Network pivoting techniques
- ✅ SOCKS proxy and proxychains
- ✅ 20+ working examples

### 21.2 Critical Skills for OSCP

**Must-master techniques**:
1. **SSH Dynamic Forwarding** - Create SOCKS proxy
2. **Port Forwarding** - Access internal services
3. **Proxychains** - Route tools through pivots
4. **IP Forwarding + Routes** - Manual pivoting
5. **tcpdump** - Debug network issues
6. **iptables basics** - Allow reverse shells

### 21.3 Essential Commands

```bash
# Network config
ip addr show
ip route show

# SSH tunneling
ssh -D 9050 user@host
ssh -L 8080:internal:80 user@host

# Pivoting
echo 1 > /proc/sys/net/ipv4/ip_forward
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

# Packet capture
tcpdump -i eth0 -w capture.pcap
tcpdump -r capture.pcap

# Proxychains
proxychains nmap -sT target
```

### 21.4 Next Steps

**Practice**:
- Set up multi-network VMs
- Practice SSH tunneling
- Build pivoting scenarios
- Capture and analyze traffic

**Further Reading**:
- HackTheBox multi-machine networks
- OSCP lab pivoting exercises
- VulnHub multi-tier networks

**Next Chapter**: File 07 - Process Management & Services

---

**You now have comprehensive Linux networking knowledge from fundamentals to advanced pivoting. Master these techniques and network-based OSCP challenges become manageable.**

---
*End of Chapter 06 - Linux Networking Stack*
