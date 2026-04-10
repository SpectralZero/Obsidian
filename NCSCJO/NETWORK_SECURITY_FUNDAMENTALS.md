# NETWORK SECURITY FUNDAMENTALS — Complete Zero-to-Hero Guide

> **Module**: NCSCJO — Network Security
> **Scope**: Networking Foundations, Protocols, Firewalls, Wireless Security, Protocol Exploits, Monitoring Tools, Architecture & Design
> **Level**: Beginner → Intermediate → Advanced → Expert
> **Goal**: Master every networking and network security concept from absolute zero to professional-grade expertise

---

# Table of Contents

```
PART I   — NETWORKING FOUNDATIONS
  1.  What Is a Network — The Absolute Basics
  2.  The OSI Model — Understanding the 7 Layers
  3.  The TCP/IP Model — The Internet's Real Architecture
  4.  IPv4 Addressing — The Language of Networks
  5.  Subnetting — Dividing Networks Like a Surgeon
  6.  Routing — How Packets Find Their Way
  7.  NAT / PAT — Translating Addresses at the Border
  8.  VLANs — Logical Network Segmentation
  9.  Encapsulation — How Data Travels Through Layers

PART II  — NETWORK PROTOCOLS DEEP DIVE
  10. LDAP — Lightweight Directory Access Protocol
  11. DHCP — Dynamic Host Configuration Protocol
  12. ARP — Address Resolution Protocol
  13. NTP — Network Time Protocol
  14. DNS — Domain Name System
  15. SMTP / SMTPS — Email Protocols
  16. SNMP — Simple Network Management Protocol
  17. TFTP — Trivial File Transfer Protocol
  18. SIP & VoIP — Voice Over IP Security
  19. SMB — Server Message Block

PART III — FIREWALLS — THE COMPLETE GUIDE
  20. What Is a Firewall — Absolute Fundamentals
  21. How Firewalls Work Internally — Packet Processing Pipeline
  22. Firewall Types — Complete Classification
  23. The Linux Firewall Stack — Netfilter Architecture
  24. iptables — The Classic Workhorse
  25. nftables — The Modern Successor
  26. UFW — Uncomplicated Firewall Deep Dive
  27. firewalld — Zone-Based Dynamic Firewall
  28. Windows Firewall — Defender Firewall with Advanced Security
  29. pfSense & OPNsense — Dedicated Firewall Appliances
  30. Cloud Firewalls — AWS, Azure, and GCP

PART IV  — NETWORK SECURITY TOOLS & MONITORING
  31. tcpdump — Command-Line Packet Analysis
  32. Wireshark — GUI Packet Analysis
  33. Network Monitoring Best Practices

PART V   — WIRELESS SECURITY
  34. Wireless Standards — 802.11 Family
  35. Wireless Encryption — WEP, WPA, WPA2, WPA3
  36. Wireless Attacks and Defenses

PART VI  — PROTOCOL EXPLOITS & ATTACK TECHNIQUES
  37. EternalBlue — SMBv1 Exploitation
  38. ARP Spoofing / Poisoning — Man-in-the-Middle
  39. DHCP Attacks — Starvation, Rogue Servers, TunnelVision
  40. NTP Amplification — DDoS via Time Servers
  41. LDAP Attacks — Bad Password Count & Account Lockout
  42. SIP/VoIP Attacks — Eavesdropping, Toll Fraud, Vishing

PART VII — FIREWALL ARCHITECTURE & DESIGN
  43. DMZ Architecture — Multi-Zone Design
  44. Network Segmentation — Microsegmentation
  45. NAT/PAT and Firewall Integration
  46. VPN Integration with Firewalls
  47. Firewall Policy Management and Best Practices

PART VIII— AUDITING, TROUBLESHOOTING & MASTERY
  48. Firewall Auditing — Finding Weaknesses
  49. Firewall Troubleshooting — Diagnosing Issues
  50. Common Mistakes — What Gets Networks Compromised
  51. Hands-On Labs
  52. Interview Questions — Prove Your Mastery
  53. Comprehensive Network Security Glossary
  54. Final Consolidated Review — The Big Picture
```

---

# PART I — NETWORKING FOUNDATIONS

---

# 1. What Is a Network — The Absolute Basics

## 1.1 Definition

```
WHAT IS A COMPUTER NETWORK?

  A computer network is a collection of interconnected computing devices
  that can exchange data and share resources with each other.

  These networked devices use a system of rules, called COMMUNICATION
  PROTOCOLS, to transmit information over physical or wireless technologies.

THE REAL-WORLD ANALOGY:

  Think of a network like a postal system:
  
  ┌──────────────────────────────────────────────────────────────┐
  │                    THE POSTAL ANALOGY                         │
  ├────────────────────┬─────────────────────────────────────────┤
  │ Postal System      │ Computer Network                        │
  ├────────────────────┼─────────────────────────────────────────┤
  │ Letter             │ Data Packet                              │
  │ Envelope           │ Packet Header (source, destination)     │
  │ Your Address       │ Source IP Address                        │
  │ Recipient Address  │ Destination IP Address                   │
  │ Post Office        │ Router                                   │
  │ Mail Carrier       │ Network Cable / Wireless Signal          │
  │ ZIP Code           │ Subnet / Network Address                 │
  │ Language Written   │ Protocol (HTTP, FTP, SSH, etc.)         │
  └────────────────────┴─────────────────────────────────────────┘
  
  Just as a letter needs an envelope with addresses and goes through
  post offices to reach its destination, a data packet needs headers
  with IP addresses and passes through routers to reach its target.
```

## 1.2 Types of Networks

```
NETWORK TYPES BY SIZE:

  ┌──────────┬──────────────────────────────────────────────────────┐
  │ Type     │ Description                                          │
  ├──────────┼──────────────────────────────────────────────────────┤
  │ PAN      │ Personal Area Network — Bluetooth, NFC               │
  │          │ Range: ~10 meters. Your phone to your earbuds.       │
  ├──────────┼──────────────────────────────────────────────────────┤
  │ LAN      │ Local Area Network — Ethernet, Wi-Fi                 │
  │          │ Range: Building/campus. Your office or home network. │
  ├──────────┼──────────────────────────────────────────────────────┤
  │ MAN      │ Metropolitan Area Network — City-wide                │
  │          │ Range: City. University campuses across a city.      │
  ├──────────┼──────────────────────────────────────────────────────┤
  │ WAN      │ Wide Area Network — Global                           │
  │          │ Range: Countries/World. The Internet is a WAN.       │
  ├──────────┼──────────────────────────────────────────────────────┤
  │ WLAN     │ Wireless LAN — Wi-Fi based LAN                      │
  │          │ Same as LAN but using wireless technology.           │
  └──────────┴──────────────────────────────────────────────────────┘

NETWORK TYPES BY FUNCTION:

  Client-Server: Central server provides resources to clients
    Example: Web server hosting a website, clients = browsers
    
  Peer-to-Peer (P2P): All devices are equal, share directly
    Example: BitTorrent, LAN file sharing
    
  Hybrid: Combination of client-server and P2P
    Example: Skype (signaling through servers, voice P2P)
```

## 1.3 Key Network Devices

```
ESSENTIAL NETWORK HARDWARE:

  ┌─────────────┬──────────────────────────────────────────────────┐
  │ Device      │ Function                                          │
  ├─────────────┼──────────────────────────────────────────────────┤
  │ Hub         │ Layer 1 — Broadcasts to ALL ports. Dumb device.  │
  │             │ No intelligence. Obsolete in modern networks.     │
  ├─────────────┼──────────────────────────────────────────────────┤
  │ Switch      │ Layer 2 — Forwards frames to SPECIFIC port       │
  │             │ based on MAC address table. Smart forwarding.     │
  ├─────────────┼──────────────────────────────────────────────────┤
  │ Router      │ Layer 3 — Forwards packets between NETWORKS      │
  │             │ based on IP address and routing table.            │
  ├─────────────┼──────────────────────────────────────────────────┤
  │ Firewall    │ Layer 3-7 — Filters traffic based on rules.      │
  │             │ Decides what gets in and what gets blocked.       │
  ├─────────────┼──────────────────────────────────────────────────┤
  │ Access Point│ Layer 2 — Extends wired network to wireless.     │
  │ (AP)        │ Bridges Wi-Fi devices to the wired LAN.          │
  ├─────────────┼──────────────────────────────────────────────────┤
  │ Load        │ Layer 4-7 — Distributes traffic across multiple  │
  │ Balancer    │ servers for performance and reliability.          │
  ├─────────────┼──────────────────────────────────────────────────┤
  │ IDS/IPS     │ Layer 3-7 — Intrusion Detection/Prevention.      │
  │             │ Monitors traffic for malicious activity.          │
  └─────────────┴──────────────────────────────────────────────────┘

  Security Note: Every network device is a potential attack surface.
  Default credentials, unpatched firmware, and misconfigured ACLs
  are the most common entry points for attackers.
```

---

# 2. The OSI Model — Understanding the 7 Layers

## 2.1 What Is the OSI Model?

```
THE OSI MODEL (Open Systems Interconnection):

  Developed in the late 1970s by the U.S. Department of Defense
  and standardized by ISO (International Organization for Standardization).
  
  PURPOSE: Provides a universal framework for understanding HOW
  network communication works, regardless of the technology used.
  
  WHY IT MATTERS FOR SECURITY:
    Every attack targets a specific OSI layer.
    Every defense operates at a specific OSI layer.
    Understanding which layer you're working at determines
    which tools, protocols, and defenses are relevant.
    
  MNEMONIC (Top to Bottom): All People Seem To Need Data Processing
  MNEMONIC (Bottom to Top): Please Do Not Throw Sausage Pizza Away
```

## 2.2 The Seven Layers — Complete Breakdown

```
THE SEVEN LAYERS — FROM PHYSICAL WIRE TO USER APPLICATION:

  ┌─────┬───────────────┬──────────────┬────────────────────────────────┐
  │ #   │ Layer         │ Data Unit    │ Function                        │
  ├─────┼───────────────┼──────────────┼────────────────────────────────┤
  │  7  │ Application   │ Data         │ User interface & services       │
  │     │               │              │ HTTP, FTP, SMTP, DNS, SSH      │
  ├─────┼───────────────┼──────────────┼────────────────────────────────┤
  │  6  │ Presentation  │ Data         │ Encoding, Encryption,          │
  │     │               │              │ Compression. SSL/TLS, JPEG     │
  ├─────┼───────────────┼──────────────┼────────────────────────────────┤
  │  5  │ Session       │ Data         │ Session management             │
  │     │               │              │ Establish, maintain, terminate │
  ├─────┼───────────────┼──────────────┼────────────────────────────────┤
  │  4  │ Transport     │ Segments     │ Reliable delivery (TCP)        │
  │     │               │              │ Fast delivery (UDP)            │
  ├─────┼───────────────┼──────────────┼────────────────────────────────┤
  │  3  │ Network       │ Packets      │ Logical addressing (IP)        │
  │     │               │              │ Routing between networks       │
  ├─────┼───────────────┼──────────────┼────────────────────────────────┤
  │  2  │ Data Link     │ Frames       │ Physical addressing (MAC)      │
  │     │               │              │ Error detection, LAN access    │
  ├─────┼───────────────┼──────────────┼────────────────────────────────┤
  │  1  │ Physical      │ Bits         │ Physical medium                │
  │     │               │              │ Cables, signals, voltages      │
  └─────┴───────────────┴──────────────┴────────────────────────────────┘
```

## 2.3 Each Layer — Detailed Explanation

```
LAYER 7 — APPLICATION LAYER:
  
  What it does: Provides network services directly to end-user applications.
  This is the layer that HUMANS interact with.
  
  Protocols:
    HTTP/HTTPS  (80/443)   — Web browsing
    FTP/SFTP    (21/22)    — File transfer
    SMTP/SMTPS  (25/465)   — Sending email
    POP3/IMAP   (110/143)  — Receiving email
    DNS         (53)       — Domain name resolution
    SSH         (22)       — Secure remote access
    Telnet      (23)       — Insecure remote access (NEVER USE)
    SNMP        (161/162)  — Network device management
    LDAP/LDAPS  (389/636)  — Directory services
    SMB         (445)      — Windows file sharing
    
  Security Attacks at Layer 7:
    SQL Injection, XSS, CSRF, directory traversal,
    buffer overflows in application code, phishing,
    credential stuffing, API abuse
    
  Security Controls at Layer 7:
    WAF (Web Application Firewall), input validation,
    authentication, authorization, rate limiting

─────────────────────────────────────────────────────────────────

LAYER 6 — PRESENTATION LAYER:

  What it does: Translates data between the application and the network.
  Handles encoding, encryption, and compression.
  
  Functions:
    Data Translation: ASCII ↔ EBCDIC, Unicode conversions
    Encryption/Decryption: SSL/TLS encryption happens here
    Compression: Reduces data size for efficient transmission
    
  Real-world examples:
    JPEG, GIF, PNG — Image format conversion
    MPEG, AVI — Video encoding
    SSL/TLS — Encryption of data in transit
    
  Security Relevance:
    SSL/TLS vulnerabilities (POODLE, BEAST, Heartbleed)
    Certificate validation failures
    Weak cipher suites
    Data encoding bypasses (e.g., double URL encoding to bypass WAFs)

─────────────────────────────────────────────────────────────────

LAYER 5 — SESSION LAYER:

  What it does: Manages sessions (connections) between applications.
  Establishes, maintains, and terminates communication sessions.
  
  Functions:
    Session establishment — Set up communication channel
    Session maintenance — Keep session alive, handle interruptions
    Session termination — Close connection gracefully
    Synchronization — Checkpoints for long data transfers
    
  Protocols:
    NetBIOS — Windows networking sessions
    RPC — Remote Procedure Call
    PPTP — Point-to-Point Tunneling Protocol
    
  Security Relevance:
    Session hijacking — Attacker takes over an active session
    Session fixation — Attacker pre-sets the session ID
    Cookie theft — Stealing session tokens for unauthorized access

─────────────────────────────────────────────────────────────────

LAYER 4 — TRANSPORT LAYER:

  What it does: Provides end-to-end communication between processes.
  Ensures data arrives completely and in order (TCP) or quickly (UDP).
  
  TCP (Transmission Control Protocol):
    Connection-oriented — Three-way handshake (SYN → SYN-ACK → ACK)
    Reliable — Guarantees delivery via acknowledgments
    Ordered — Data arrives in sequence
    Error-checked — Checksums verify data integrity
    Flow controlled — Prevents overwhelming the receiver
    Used for: HTTP, SSH, FTP, SMTP, database connections
    
  UDP (User Datagram Protocol):
    Connectionless — No handshake, just send
    Unreliable — No guarantee of delivery
    Unordered — Packets may arrive out of sequence
    Fast — Minimal overhead
    Used for: DNS, DHCP, VoIP, streaming, gaming, NTP
    
  THE TCP THREE-WAY HANDSHAKE:
  
    Client                     Server
      │                          │
      │──── SYN (seq=100) ──────▶│  "I want to connect"
      │                          │
      │◀── SYN-ACK (seq=300, ───│  "OK, I acknowledge"
      │     ack=101)             │
      │                          │
      │──── ACK (seq=101, ──────▶│  "Connection established"
      │     ack=301)             │
      │                          │
      │   CONNECTION ESTABLISHED  │
      │   Data transfer begins    │
      
  Security Relevance:
    SYN Flood attack — Sends thousands of SYN packets without completing
    the handshake, exhausting server resources (DoS attack).
    Port scanning — TCP SYN scan (half-open scan) sends SYN,
    checks for SYN-ACK (port open) or RST (port closed).

─────────────────────────────────────────────────────────────────

LAYER 3 — NETWORK LAYER:

  What it does: Handles logical addressing (IP addresses) and routing.
  Determines the best path for data to travel across networks.
  
  Protocols:
    IPv4 — 32-bit addressing (e.g., 192.168.1.1)
    IPv6 — 128-bit addressing (e.g., fe80::1)
    ICMP — Ping, traceroute, error messages
    IGMP — Multicast group management
    
  Devices: Routers, Layer 3 switches
  
  Security Relevance:
    IP spoofing — Forging source IP addresses
    ICMP attacks — Ping of Death, Smurf attack
    Route hijacking — BGP hijacking redirects traffic
    Reconnaissance — Traceroute reveals network topology

─────────────────────────────────────────────────────────────────

LAYER 2 — DATA LINK LAYER:

  What it does: Handles physical addressing (MAC addresses) and
  controls access to the physical medium.
  
  Sub-layers:
    LLC (Logical Link Control) — Error checking, flow control
    MAC (Media Access Control) — Physical addressing, frame formatting
    
  Protocols: Ethernet (802.3), Wi-Fi (802.11), PPP, ARP
  Data Unit: Frames
  Devices: Switches, Bridges, NICs
  
  MAC Address Format:
    48-bit address, written as hex pairs: AA:BB:CC:DD:EE:FF
    First 3 bytes = OUI (Organizationally Unique Identifier) = manufacturer
    Last 3 bytes = Device-specific identifier
    
  Security Relevance:
    ARP spoofing/poisoning — Redirecting traffic via forged ARP replies
    MAC flooding — Overloading switch MAC table (CAM table overflow)
    MAC spoofing — Impersonating another device's MAC address
    VLAN hopping — Escaping VLAN isolation via double tagging

─────────────────────────────────────────────────────────────────

LAYER 1 — PHYSICAL LAYER:

  What it does: Transmits raw bits over the physical medium.
  Defines electrical signals, light pulses, radio frequencies.
  
  Media Types:
    Copper cables — Cat5e, Cat6, Cat6a (Ethernet)
    Fiber optic — Single-mode (long distance), Multi-mode (short)
    Wireless — Radio waves (Wi-Fi, Bluetooth, cellular)
    
  Security Relevance:
    Physical wiretapping — Tapping copper cables
    Fiber tapping — Bending fiber to intercept light
    Jamming — Radio frequency interference on wireless
    Physical access — If attacker has physical access, game over
    
  Best Practice: Physical security is the FOUNDATION of all security.
  Locked server rooms, cable management, security cameras, badge access.
```

## 2.4 Attacks by OSI Layer — Quick Reference

```
ATTACK SURFACE BY OSI LAYER:

  ┌─────┬───────────────┬────────────────────────────────────────────┐
  │ L#  │ Layer         │ Common Attacks                              │
  ├─────┼───────────────┼────────────────────────────────────────────┤
  │  7  │ Application   │ SQLi, XSS, CSRF, RFI/LFI, SSRF, phishing│
  │  6  │ Presentation  │ SSL stripping, cipher downgrade, encoding │
  │  5  │ Session       │ Session hijacking, session fixation        │
  │  4  │ Transport     │ SYN flood, UDP flood, port scanning       │
  │  3  │ Network       │ IP spoofing, ICMP flood, route hijacking  │
  │  2  │ Data Link     │ ARP spoofing, MAC flooding, VLAN hopping  │
  │  1  │ Physical      │ Wiretapping, jamming, physical theft      │
  └─────┴───────────────┴────────────────────────────────────────────┘
  
  Key Insight: Defense in depth means implementing security controls
  at EVERY layer, not just one. A firewall (Layer 3-4) cannot stop
  a SQL injection (Layer 7). A WAF (Layer 7) cannot stop ARP spoofing
  (Layer 2). You need controls at every layer.
```

---

# 3. The TCP/IP Model — The Internet's Real Architecture

## 3.1 TCP/IP vs. OSI

```
THE TCP/IP MODEL (Internet Protocol Suite):

  The TCP/IP model is the ACTUAL protocol stack used on the Internet.
  The OSI model is a THEORETICAL reference model.
  
  TCP/IP was adopted by CISCO and is the basis for all modern
  internet communication. It has only 4 layers.

  OSI MODEL (7 Layers)          TCP/IP MODEL (4 Layers)
  ─────────────────────         ─────────────────────────
  7. Application    ─┐
  6. Presentation    ├────────▶  4. Application Layer
  5. Session        ─┘              (HTTP, FTP, DNS, SSH, SMTP)
  
  4. Transport      ──────────▶  3. Transport Layer
                                    (TCP, UDP)
  
  3. Network        ──────────▶  2. Internet Layer
                                    (IP, ICMP, ARP)
  
  2. Data Link     ─┐
  1. Physical       ┘─────────▶  1. Link Layer / Network Access
                                    (Ethernet, Wi-Fi, PPP)

WHY TCP/IP USES 4 LAYERS INSTEAD OF 7:

  The TCP/IP designers were pragmatists, not theorists.
  They combined layers that didn't need separate implementations:
  
  • Application + Presentation + Session = One "Application" layer
    (Because most applications handle all three functions internally)
    
  • Data Link + Physical = One "Link" layer
    (Because the protocol doesn't care about the physical medium —
     Ethernet, Wi-Fi, or fiber all work the same way at the IP level)
```

## 3.2 TCP/IP Layers — Detailed

```
TCP/IP LAYER 4 — APPLICATION LAYER:

  Combines OSI layers 5, 6, 7.
  This is where applications interact with the network.
  
  Common Protocols and Their Ports:
  ┌──────────────┬──────────┬─────────────────────────────────────┐
  │ Protocol     │ Port     │ Purpose                              │
  ├──────────────┼──────────┼─────────────────────────────────────┤
  │ HTTP         │ 80       │ Web browsing (unencrypted)           │
  │ HTTPS        │ 443      │ Web browsing (encrypted TLS)         │
  │ FTP          │ 21       │ File transfer (control channel)      │
  │ SFTP/SCP     │ 22       │ Secure file transfer over SSH        │
  │ SSH          │ 22       │ Secure remote shell access           │
  │ Telnet       │ 23       │ Insecure remote access (AVOID!)     │
  │ SMTP         │ 25/465   │ Send email                           │
  │ POP3         │ 110/995  │ Receive email (download)             │
  │ IMAP         │ 143/993  │ Receive email (sync)                 │
  │ DNS          │ 53       │ Domain name resolution               │
  │ DHCP         │ 67/68    │ Automatic IP assignment              │
  │ SNMP         │ 161/162  │ Network device management            │
  │ TFTP         │ 69       │ Simple file transfer (no auth!)     │
  │ NTP          │ 123      │ Time synchronization                 │
  │ LDAP/LDAPS   │ 389/636  │ Directory services                   │
  │ SMB          │ 445      │ Windows file/print sharing           │
  │ RDP          │ 3389     │ Windows Remote Desktop               │
  │ MySQL        │ 3306     │ MySQL database                       │
  │ PostgreSQL   │ 5432     │ PostgreSQL database                  │
  │ MongoDB      │ 27017    │ MongoDB database                     │
  │ Redis        │ 6379     │ Redis cache/database                 │
  │ SIP          │ 5060/5061│ VoIP signaling                       │
  │ RTP          │ Dynamic  │ VoIP media (voice/video data)        │
  └──────────────┴──────────┴─────────────────────────────────────┘

  SECURITY NOTE: Memorize these ports! During penetration testing and
  incident response, recognizing ports instantly is critical.
  
  Port Ranges:
    0-1023     = Well-Known Ports (require root/admin to bind)
    1024-49151 = Registered Ports (assigned by IANA for specific apps)
    49152-65535 = Dynamic/Ephemeral Ports (used by client connections)

───────────────────────────────────────────────────────────────

TCP/IP LAYER 3 — TRANSPORT LAYER:

  Provides end-to-end communication between processes on different hosts.
  
  TCP vs UDP — The Complete Comparison:
  ┌───────────────────┬──────────────────┬──────────────────────┐
  │ Feature           │ TCP              │ UDP                   │
  ├───────────────────┼──────────────────┼──────────────────────┤
  │ Connection        │ Connection-      │ Connectionless        │
  │                   │ oriented         │                       │
  ├───────────────────┼──────────────────┼──────────────────────┤
  │ Reliability       │ Guaranteed       │ Best-effort           │
  ├───────────────────┼──────────────────┼──────────────────────┤
  │ Ordering          │ Ordered          │ Unordered             │
  ├───────────────────┼──────────────────┼──────────────────────┤
  │ Speed             │ Slower           │ Faster                │
  ├───────────────────┼──────────────────┼──────────────────────┤
  │ Header Size       │ 20-60 bytes      │ 8 bytes               │
  ├───────────────────┼──────────────────┼──────────────────────┤
  │ Flow Control      │ Yes (windowing)  │ No                    │
  ├───────────────────┼──────────────────┼──────────────────────┤
  │ Error Recovery    │ Yes (retransmit) │ No                    │
  ├───────────────────┼──────────────────┼──────────────────────┤
  │ Use Cases         │ HTTP, SSH, FTP,  │ DNS, DHCP, NTP,      │
  │                   │ SMTP, databases  │ VoIP, streaming      │
  └───────────────────┴──────────────────┴──────────────────────┘

───────────────────────────────────────────────────────────────

TCP/IP LAYER 2 — INTERNET LAYER:

  Handles logical addressing and routing. The IP protocol lives here.
  
  Key Protocols:
    IP (Internet Protocol) — Addressing and routing packets
    ICMP (Internet Control Message Protocol) — Error reporting, ping
    ARP (Address Resolution Protocol) — Maps IP → MAC address
    IGMP — Manages multicast groups
    
  This layer determines HOW a packet gets from source to destination,
  regardless of what's inside the packet.

───────────────────────────────────────────────────────────────

TCP/IP LAYER 1 — LINK LAYER (Network Access):

  Handles the physical transmission of data on the local network.
  
  Includes both the physical medium (cables, radio waves) and
  the data link protocols (Ethernet, Wi-Fi, PPP).
  
  Key Protocols:
    Ethernet (IEEE 802.3) — Wired LAN standard
    Wi-Fi (IEEE 802.11) — Wireless LAN standard
    PPP (Point-to-Point Protocol) — WAN serial links
    ARP (also operates here) — IP-to-MAC resolution
```

---

# 4. IPv4 Addressing — The Language of Networks

## 4.1 IPv4 Address Structure

```
IPv4 ADDRESS FORMAT:

  An IPv4 address is a 32-bit number, divided into 4 octets (groups
  of 8 bits), each separated by dots (dotted-decimal notation).
  
  Example: 192.168.1.128
  
  In binary:
    192       .168       .1         .128
    1100 0000 .1010 1000 .0000 0001 .1000 0000
    
  Each octet can hold values from 0 to 255 (2^8 = 256 values).
  Total IPv4 addresses: 2^32 = 4,294,967,296 (about 4.3 billion)
  
THE TWO PARTS OF AN IP ADDRESS:

  Every IP address has TWO components:
  
  1. NETWORK ADDRESS (left side) — Identifies WHICH NETWORK
  2. HOST ADDRESS (right side) — Identifies WHICH DEVICE on that network
  
  The SUBNET MASK determines where the boundary is:
  
  Example: 192.168.1.128/24
  
    Address:  1100 0000 . 1010 1000 . 0000 0001 . 1000 0000
    Mask /24: 1111 1111 . 1111 1111 . 1111 1111 . 0000 0000
              ─────────── NETWORK ──────────────  ── HOST ──
              
    Network Address: 192.168.1.0   (the "street name")
    Host Address:    128            (the "house number")
    Subnet Mask:     255.255.255.0  (or /24 in CIDR notation)
```

## 4.2 IP Address Classes

```
IPv4 ADDRESS CLASSES (Classful Addressing):

  ┌───────┬────────────────────┬──────────────┬─────────────────────┐
  │ Class │ Range              │ Default Mask │ Purpose              │
  ├───────┼────────────────────┼──────────────┼─────────────────────┤
  │   A   │ 1.0.0.0 -          │ /8           │ Very large networks  │
  │       │ 126.255.255.255    │ 255.0.0.0    │ 16 million hosts     │
  ├───────┼────────────────────┼──────────────┼─────────────────────┤
  │   B   │ 128.0.0.0 -        │ /16          │ Medium networks      │
  │       │ 191.255.255.255    │ 255.255.0.0  │ 65,534 hosts         │
  ├───────┼────────────────────┼──────────────┼─────────────────────┤
  │   C   │ 192.0.0.0 -        │ /24          │ Small networks       │
  │       │ 223.255.255.255    │ 255.255.255.0│ 254 hosts            │
  ├───────┼────────────────────┼──────────────┼─────────────────────┤
  │   D   │ 224.0.0.0 -        │ N/A          │ Multicast            │
  │       │ 239.255.255.255    │              │                      │
  ├───────┼────────────────────┼──────────────┼─────────────────────┤
  │   E   │ 240.0.0.0 -        │ N/A          │ Reserved/Research    │
  │       │ 255.255.255.255    │              │                      │
  └───────┴────────────────────┴──────────────┴─────────────────────┘

  Note: 127.0.0.0/8 is reserved for LOOPBACK (localhost).
  127.0.0.1 is your own machine. Always available, never on the network.
  
PRIVATE IP ADDRESS RANGES (RFC 1918):

  These addresses are NOT routable on the Internet.
  Used inside private networks (your home, your office, corporate LANs).
  
  ┌───────┬─────────────────────────────┬──────────────────────────┐
  │ Class │ Private Range               │ Usage                     │
  ├───────┼─────────────────────────────┼──────────────────────────┤
  │   A   │ 10.0.0.0 – 10.255.255.255  │ Large enterprises         │
  │       │ (10.0.0.0/8)               │ 16 million addresses      │
  ├───────┼─────────────────────────────┼──────────────────────────┤
  │   B   │ 172.16.0.0 – 172.31.255.255│ Medium organizations     │
  │       │ (172.16.0.0/12)            │ 1 million addresses       │
  ├───────┼─────────────────────────────┼──────────────────────────┤
  │   C   │ 192.168.0.0 –              │ Home networks, small biz  │
  │       │ 192.168.255.255            │ 65,536 addresses          │
  │       │ (192.168.0.0/16)           │                           │
  └───────┴─────────────────────────────┴──────────────────────────┘
  
  Security Implication: Private IPs behind NAT are NOT directly
  reachable from the Internet. This provides a basic layer of security
  (but NOT a firewall — NAT is NOT a security control by itself!).
  
SPECIAL ADDRESSES:
  0.0.0.0        — "This network" / Default route (catch-all)
  127.0.0.1      — Loopback (localhost)
  169.254.x.x    — APIPA (link-local, when DHCP fails)
  255.255.255.255 — Broadcast to entire network
```

---

# 5. Subnetting — Dividing Networks Like a Surgeon

## 5.1 Why Subnetting?

```
WHY DO WE SUBNET?

  Imagine a university with 5,000 students and staff, all on ONE
  flat network: 10.0.0.0/8.
  
  Problems with ONE big network:
    ❌ Broadcast storms — Every ARP request hits ALL 5,000 devices
    ❌ No security boundaries — Students can access admin servers
    ❌ Performance — Congestion from too many devices competing
    ❌ Management — Impossible to apply different policies per group
    
  Solution: SUBNETTING — Divide 10.0.0.0/8 into smaller subnets:
    10.1.0.0/24 — Student Wi-Fi            (254 hosts)
    10.2.0.0/24 — Faculty network           (254 hosts)
    10.3.0.0/24 — Server farm               (254 hosts)
    10.4.0.0/24 — Management / Admin        (254 hosts)
    10.5.0.0/24 — IoT devices               (254 hosts)
    
  Benefits:
    ✅ Reduced broadcast domain — Broadcasts stay in their subnet
    ✅ Security segmentation — Firewall rules between subnets
    ✅ Better performance — Less congestion per subnet
    ✅ Easier management — Different policies per subnet
    ✅ Compliance — PCI-DSS requires cardholder data isolation
```

## 5.2 CIDR Notation Quick Reference

```
CIDR (Classless Inter-Domain Routing) CHEAT SHEET:

  ┌────────┬─────────────────┬──────────┬──────────────────────────┐
  │ CIDR   │ Subnet Mask     │ # Hosts  │ Use Case                  │
  ├────────┼─────────────────┼──────────┼──────────────────────────┤
  │ /32    │ 255.255.255.255 │ 1        │ Single host route         │
  │ /31    │ 255.255.255.254 │ 2        │ Point-to-point link       │
  │ /30    │ 255.255.255.252 │ 2        │ WAN links (2 usable)     │
  │ /29    │ 255.255.255.248 │ 6        │ Small office              │
  │ /28    │ 255.255.255.240 │ 14       │ Small department          │
  │ /27    │ 255.255.255.224 │ 30       │ Medium department         │
  │ /26    │ 255.255.255.192 │ 62       │ Large department          │
  │ /25    │ 255.255.255.128 │ 126      │ Half a Class C            │
  │ /24    │ 255.255.255.0   │ 254      │ Standard LAN (Class C)   │
  │ /23    │ 255.255.254.0   │ 510      │ Two Class C combined     │
  │ /22    │ 255.255.252.0   │ 1022     │ Medium organization       │
  │ /16    │ 255.255.0.0     │ 65,534   │ Large organiztion (B)    │
  │ /8     │ 255.0.0.0       │ 16.7M    │ Massive network (A)      │
  └────────┴─────────────────┴──────────┴──────────────────────────┘
  
  Formula: Usable hosts = 2^(32 - CIDR) - 2
  (Subtract 2 for network address and broadcast address)
  
  Example: /24 → 2^(32-24) - 2 = 2^8 - 2 = 256 - 2 = 254 hosts
```

---

# 6. Routing — How Packets Find Their Way

## 6.1 What Is Routing?

```
ROUTING FUNDAMENTALS:

  Network routing is the process of selecting a path across one or
  more networks for data packets to travel from source to destination.
  
  ROUTING DECISIONS are made by ROUTERS — specialized network hardware
  that maintains a ROUTING TABLE telling it where to send packets.
  
  ANALOGY:
    You're driving from City A to City B. At every intersection,
    there are road signs telling you which direction to go.
    
    The intersections = Routers
    The road signs = Routing Tables
    Your car = The Data Packet
    The highway = The Network Link

HOW A ROUTER DECIDES:

  When a router receives a packet:
  
  1. Extract the DESTINATION IP ADDRESS from the packet header
  2. Look up the destination in the ROUTING TABLE
  3. Find the BEST MATCH (longest prefix match)
  4. Forward the packet to the NEXT HOP router or directly to host
  5. If no match found → Send to DEFAULT GATEWAY (0.0.0.0/0)
  6. If no default gateway → DROP the packet (destination unreachable)
```

## 6.2 Routing Protocols

```
ROUTING PROTOCOLS — HOW ROUTERS LEARN ROUTES:

  STATIC ROUTING:
    Routes manually configured by an administrator.
    Simple but doesn't adapt to network changes.
    Used for small networks or specific permanent routes.
    
    Example: ip route 10.0.0.0 255.255.255.0 192.168.1.1

  DYNAMIC ROUTING:
    Routers automatically discover and share route information.
    Adapts to network changes (link failures, new networks).
    
  INTERIOR GATEWAY PROTOCOLS (IGP) — Within an organization:
  
    ┌────────────┬──────────────────────────────────────────────────┐
    │ Protocol   │ Description                                      │
    ├────────────┼──────────────────────────────────────────────────┤
    │ RIP/RIPv2  │ Routing Information Protocol                     │
    │            │ Distance-vector. Max 15 hops. Simple but slow.  │
    │            │ Suitable only for small networks.                │
    ├────────────┼──────────────────────────────────────────────────┤
    │ OSPF       │ Open Shortest Path First                         │
    │            │ Link-state. Fast convergence. Industry standard. │
    │            │ Uses Dijkstra's algorithm for shortest path.     │
    ├────────────┼──────────────────────────────────────────────────┤
    │ EIGRP      │ Enhanced Interior Gateway Routing Protocol       │
    │            │ Cisco proprietary (now partially open).          │
    │            │ Hybrid protocol. Very efficient.                 │
    ├────────────┼──────────────────────────────────────────────────┤
    │ IS-IS      │ Intermediate System to Intermediate System       │
    │            │ Link-state. Used by large ISPs.                  │
    │            │ Very scalable for massive networks.              │
    └────────────┴──────────────────────────────────────────────────┘

  EXTERIOR GATEWAY PROTOCOL (EGP) — Between organizations:
  
    BGP (Border Gateway Protocol):
      The routing protocol of THE INTERNET.
      Manages routing between autonomous systems (AS).
      Every ISP, cloud provider, and large enterprise uses BGP.
      
      Security Risk: BGP HIJACKING
        Attacker advertises false BGP routes, redirecting internet
        traffic through their network. This allows:
        - Traffic interception (Man-in-the-Middle)
        - Denial of Service (black-holing traffic)
        - Cryptocurrency theft (redirecting mining pool traffic)
        
      Real-World Example:
        2018 — Attackers hijacked BGP routes to Amazon's Route 53
        DNS service, redirecting MyEtherWallet.com traffic to steal
        $150,000 in cryptocurrency.
```

---

# 7. NAT / PAT — Translating Addresses at the Border

## 7.1 What Is NAT?

```
NETWORK ADDRESS TRANSLATION (NAT):

  NAT translates PRIVATE IP addresses to PUBLIC IP addresses
  (and vice versa) as traffic crosses the network boundary.
  
  WHY NAT EXISTS:
    Problem: There are only ~4.3 billion IPv4 addresses, but there
    are 30+ billion connected devices. Not enough public IPs!
    
    Solution: Use PRIVATE addresses internally (10.x, 172.16.x, 192.168.x)
    and translate them to a SHARED public IP when accessing the Internet.
    
    Your entire home network (phone, laptop, tablet, smart TV) uses
    ONE public IP address. The router uses NAT to keep track of which
    internal device made which request.

NAT TYPES:

  ┌──────────────────┬──────────────────────────────────────────────┐
  │ Type             │ How It Works                                  │
  ├──────────────────┼──────────────────────────────────────────────┤
  │ Static NAT       │ 1:1 mapping. One private IP = one public IP. │
  │ (SNAT)           │ Used for servers that need a fixed public IP. │
  │                  │ Example: Web server 10.0.0.5 → 203.0.113.5   │
  ├──────────────────┼──────────────────────────────────────────────┤
  │ Dynamic NAT      │ Pool of public IPs. Private IPs are mapped   │
  │                  │ to available public IPs from a pool on-demand.│
  │                  │ Less common today.                            │
  ├──────────────────┼──────────────────────────────────────────────┤
  │ PAT / NAT        │ MANY private IPs share ONE public IP.         │
  │ Overload         │ Differentiated by PORT NUMBERS.               │
  │ (Port Address    │ This is what your home router does.           │
  │  Translation)    │ Also called "masquerading" in Linux.          │
  │                  │                                               │
  │                  │ 10.0.0.5:49321 → 203.0.113.1:49321           │
  │                  │ 10.0.0.6:52100 → 203.0.113.1:52100           │
  │                  │ 10.0.0.7:38000 → 203.0.113.1:38000           │
  │                  │ (All use the SAME public IP, different ports) │
  └──────────────────┴──────────────────────────────────────────────┘

  SECURITY NOTE:
    NAT provides OBSCURITY, not SECURITY.
    NAT hides internal IPs from the Internet (attackers can't directly
    reach 192.168.1.5 from outside). But NAT is NOT a firewall.
    NAT does not inspect packets, does not block malicious traffic,
    and does not replace the need for a proper firewall.
```

---

# 8. VLANs — Logical Network Segmentation

## 8.1 What Are VLANs?

```
VIRTUAL LOCAL AREA NETWORKS (VLANs):

  A VLAN is a method of creating LOGICALLY separate networks
  on the SAME physical switch infrastructure.
  
  Without VLANs: All devices on a switch are in the same broadcast
  domain. A broadcast from one device reaches ALL other devices.
  
  With VLANs: The switch creates virtual boundaries. Devices in
  different VLANs cannot communicate directly — they need a ROUTER
  (or Layer 3 switch) to route traffic between VLANs.

  PHYSICAL SETUP (without VLANs):
  
    ┌──────────────────────────────────────────┐
    │              SINGLE SWITCH               │
    │                                          │
    │  Port 1: HR PC      Port 5: Finance PC  │
    │  Port 2: HR PC      Port 6: Finance PC  │
    │  Port 3: Server     Port 7: Dev PC      │
    │  Port 4: Printer    Port 8: Dev PC      │
    │                                          │
    │  ALL devices can see ALL other devices   │
    │  One broadcast domain. No isolation.     │
    └──────────────────────────────────────────┘
    
  WITH VLANs:
  
    ┌──────────────────────────────────────────┐
    │              SAME SWITCH                  │
    │                                          │
    │  VLAN 10 (HR):        VLAN 20 (Finance): │
    │    Port 1: HR PC        Port 5: Fin PC   │
    │    Port 2: HR PC        Port 6: Fin PC   │
    │                                          │
    │  VLAN 30 (Servers):   VLAN 40 (Dev):     │
    │    Port 3: Server       Port 7: Dev PC   │
    │    Port 4: Printer      Port 8: Dev PC   │
    │                                          │
    │  HR cannot see Finance traffic           │
    │  Dev cannot see Server traffic directly  │
    │  Inter-VLAN traffic goes through router  │
    └──────────────────────────────────────────┘

VLAN BENEFITS FOR SECURITY:

  ✅ Segmentation — Limits blast radius of a breach
  ✅ Access Control — Firewall rules between VLANs
  ✅ Compliance — PCI-DSS: isolate cardholder data network
  ✅ Reduced Broadcast — Better performance per VLAN
  ✅ Cost Effective — No need for separate physical switches

VLAN SECURITY ATTACKS:

  VLAN Hopping (Double Tagging):
    Attacker sends a frame with TWO VLAN tags.
    The first switch strips the outer tag (attacker's native VLAN).
    The inner tag routes the frame to a different, target VLAN.
    
    Defense: Set the native VLAN to an unused VLAN ID.
    Never use VLAN 1 as the native VLAN. Explicitly tag all ports.
    
  Switch Spoofing:
    Attacker configures their NIC to act as a trunk port.
    If the switch is misconfigured with DTP (Dynamic Trunking Protocol)
    enabled, the attacker negotiates a trunk and gains access to all VLANs.
    
    Defense: Disable DTP. Set all access ports to "switchport mode access".
    Only manually configure trunk ports where needed.
```

---

# 9. Encapsulation — How Data Travels Through Layers

## 9.1 The Encapsulation Process

```
ENCAPSULATION — DATA WRAPPING AT EACH LAYER:

  When data is sent across a network, each OSI/TCP-IP layer WRAPS
  the data from the layer above with its own HEADER (and sometimes a
  trailer). This process is called ENCAPSULATION.
  
  The reverse process at the receiving end is called DE-ENCAPSULATION.

  ┌─────────────────────────────────────────────────────────────────┐
  │                    ENCAPSULATION PROCESS                        │
  │                                                                 │
  │  Application Layer:    [ DATA ]                                │
  │                         "Hello, this is a web page"            │
  │                                                                 │
  │  Transport Layer:      [ TCP Header | DATA ]                   │
  │                         Adds: Source port, Dest port,          │
  │                         Sequence number, Flags                  │
  │                         → Now called a SEGMENT                  │
  │                                                                 │
  │  Network Layer:        [ IP Header | TCP Header | DATA ]       │
  │                         Adds: Source IP, Dest IP,              │
  │                         TTL, Protocol                           │
  │                         → Now called a PACKET                   │
  │                                                                 │
  │  Data Link Layer:      [ ETH Header | IP | TCP | DATA | FCS ] │
  │                         Adds: Source MAC, Dest MAC,            │
  │                         EtherType, Frame Check Sequence        │
  │                         → Now called a FRAME                    │
  │                                                                 │
  │  Physical Layer:       01101001 01001000 11010010 ...          │
  │                         Raw bits on the wire or radio waves    │
  │                         → Now called BITS                       │
  └─────────────────────────────────────────────────────────────────┘
  
  DATA UNIT NAMES BY LAYER:
  
    Application  →  Data
    Transport    →  Segment (TCP) / Datagram (UDP)
    Network      →  Packet
    Data Link    →  Frame
    Physical     →  Bits

  Security Insight:
    Each layer's header is a potential attack surface.
    Attackers can craft custom packets with modified headers at any layer:
    - Modified IP header → IP spoofing (Layer 3)
    - Modified TCP flags → SYN flood, TCP RST injection (Layer 4)
    - Modified MAC header → MAC spoofing (Layer 2)
    - Modified application data → SQL injection, XSS (Layer 7)
```

---

# PART II — NETWORK PROTOCOLS DEEP DIVE

---

# 10. LDAP — Lightweight Directory Access Protocol

## 10.1 What Is LDAP?

```
LDAP (Lightweight Directory Access Protocol):

  An open, vendor-neutral APPLICATION PROTOCOL used to access and
  maintain distributed directory information services over an IP network.
  
  Think of LDAP as a specialized database designed for READ-HEAVY,
  HIGHLY AVAILABLE LOOKUPS. It's commonly used for authentication
  and storing information about users, groups, devices, and
  organizational structure.
  
  THE PHONE BOOK ANALOGY:
    LDAP is like a digital phone book for an organization.
    You can look up a person by name and find their:
    - Email address
    - Phone number
    - Department
    - Manager
    - Login credentials
    
    But unlike a phone book, LDAP is:
    - Hierarchical (tree structure)
    - Queryable (search with filters)
    - Writable (add/modify/delete entries)
    - Access-controlled (different permissions per user)

LDAP vs. ACTIVE DIRECTORY — THE CRITICAL DISTINCTION:

  LDAP is the PROTOCOL — the set of rules for communicating
  Active Directory is the SERVICE — Microsoft's complete directory product
  
  ┌──────────────────┬────────────────────────────────────────────────┐
  │ Feature          │ LDAP                  │ Active Directory (AD) │
  ├──────────────────┼───────────────────────┼───────────────────────┤
  │ Core Identity    │ Open protocol         │ Microsoft product     │
  ├──────────────────┼───────────────────────┼───────────────────────┤
  │ Function         │ Directory             │ Complete IAM          │
  │                  │ communication rules   │ solution              │
  ├──────────────────┼───────────────────────┼───────────────────────┤
  │ Platform         │ Cross-platform        │ Windows-centric       │
  ├──────────────────┼───────────────────────┼───────────────────────┤
  │ Features         │ Operations, data      │ LDAP + Kerberos +     │
  │                  │ model, security       │ Group Policy + DNS    │
  ├──────────────────┼───────────────────────┼───────────────────────┤
  │ Cost             │ Free (OpenLDAP)       │ Licensed Microsoft    │
  ├──────────────────┼───────────────────────┼───────────────────────┤
  │ Ports            │ 389 (LDAP)            │ 389 (LDAP)            │
  │                  │ 636 (LDAPS)           │ 636 (LDAPS)           │
  │                  │                       │ 3268 (Global Catalog) │
  └──────────────────┴───────────────────────┴───────────────────────┘
  
  In short: Active Directory SPEAKS the LDAP language, but AD is
  much more than just LDAP. AD also uses Kerberos for authentication,
  Group Policy for configuration management, and DNS for name resolution.
```

## 10.2 LDAP Architecture — The Directory Information Tree

```
THE LDAP DATA MODEL:

  LDAP stores data in a hierarchical tree structure called the
  DIRECTORY INFORMATION TREE (DIT). Think of it as an upside-down
  tree, where the root is at the top.
  
  EXAMPLE DIT STRUCTURE:
  
    dc=example,dc=com                     ← ROOT (Domain Component)
    │
    ├── ou=Users                          ← Organizational Unit
    │   ├── cn=John Doe                   ← Entry (Common Name)
    │   │   ├── mail: john@example.com    ← Attribute
    │   │   ├── uid: jdoe                 ← Attribute
    │   │   └── userPassword: {SSHA}...   ← Attribute
    │   │
    │   └── cn=Jane Smith
    │       ├── mail: jane@example.com
    │       ├── uid: jsmith
    │       └── userPassword: {SSHA}...
    │
    ├── ou=Groups
    │   ├── cn=Admins
    │   └── cn=Developers
    │
    └── ou=Servers
        ├── cn=web-server-01
        └── cn=db-server-01

KEY TERMINOLOGY:

  Entry: The fundamental unit — represents one object
    (a person, a group, a device)
    
  Attribute: A piece of information about an entry
    Type + Value(s): cn=John Doe, mail=john@example.com
    
  Object Class: Blueprint defining required/optional attributes
    person: requires cn, sn; allows description
    
  Schema: Complete set of rules for valid object classes and attributes
  
  Distinguished Name (DN): Full path to an entry
    Example: cn=John Doe,ou=Users,dc=example,dc=com
    
  Relative Distinguished Name (RDN): Just the entry's own identifier
    Example: cn=John Doe
    
  Common Attribute Types:
    dc = Domain Component (e.g., dc=example)
    ou = Organizational Unit (e.g., ou=Users)
    cn = Common Name (e.g., cn=John Doe)
    sn = Surname
    uid = User ID
    mail = Email address
    userPassword = Password hash
```

## 10.3 LDAP Operations

```
LDAP OPERATIONS — THE CORE COMMANDS:

  BINDING AND SESSION:
    Bind     — Authenticate to the LDAP server (provide DN + credentials)
    Unbind   — Terminate the LDAP session
    Abandon  — Cancel a pending operation
    
  QUERYING:
    Search   — The MOST common operation. Query the DIT with filters.
               Example filter: (&(objectClass=person)(mail=*@example.com))
               
    Compare  — Check if a specific attribute value exists in an entry
               Used for password verification without retrieving the entry

  UPDATING:
    Add      — Insert a new entry into the DIT
    Delete   — Remove an entry (must be a leaf — no children)
    Modify   — Change attributes of an existing entry
    ModifyDN — Rename or move an entry

LDAP SEARCH EXAMPLE:

  # Search for all users in the Users OU
  ldapsearch -x -H ldap://ldap.example.com \
    -b "ou=Users,dc=example,dc=com" \
    -D "cn=admin,dc=example,dc=com" \
    -W \
    "(objectClass=person)" \
    cn mail uid
    
  # Breakdown:
  # -x         = Simple authentication
  # -H         = LDAP server URL
  # -b         = Base DN (where to start searching)
  # -D         = Bind DN (who you are)
  # -W         = Prompt for password
  # "(...)     = Search filter
  # cn mail uid = Attributes to return
```

## 10.4 LDAP Security

```
LDAP SECURITY — AUTHENTICATION AND ENCRYPTION:

  AUTHENTICATION METHODS:
  
  1. Anonymous Authentication
     Client binds with NO credentials.
     Used for public, read-only access (company address book).
     RISK: Should be disabled in most environments.
     
  2. Simple Authentication
     Client sends DN + PLAINTEXT password.
     INHERENTLY INSECURE unless wrapped in TLS.
     MUST use in conjunction with LDAPS or StartTLS.
     
  3. SASL (Simple Authentication and Security Layer)
     Framework supporting multiple strong auth mechanisms:
     
     SASL EXTERNAL — Uses TLS client certificate for authentication
     SASL GSSAPI — Integrates with Kerberos (Active Directory SSO)
     SASL DIGEST-MD5 — Challenge-response (deprecated)

  ENCRYPTION — LDAPS vs StartTLS:
  
    LDAPS (LDAP over SSL/TLS):
      Port 636. Secure from the first byte.
      The connection starts encrypted immediately.
      Non-standard but widely supported.
      
    StartTLS (Extended Operation):
      Port 389. Starts as plaintext, then upgrades to TLS.
      Standard method defined in LDAPv3.
      More flexible — uses single port for both modes.
      
    CRITICAL: Always use one of these. NEVER send LDAP
    credentials in plaintext over the network!

  ACCESS CONTROL:
    ACLs (Access Control Lists) or ACIs (Access Control Instructions)
    define who can read, write, search, or compare attributes.
    
    Example: Allow users to change their own password but
    deny them from reading other users' password hashes.
```

## 10.5 LDAP Attacks — Bad Password Count & Account Lockout

```
LDAP ATTACK: BAD PASSWORD COUNT EXPLOITATION

  "Bad Password Count" is a directory service mechanism for tracking
  FAILED authentication attempts against a user account.
  
  It's central to password throttling and account lockout policies —
  PRIMARY defenses against brute-force attacks.

THE VULNERABILITY — WEAPONIZING THE DEFENSE:

  While designed for security, this feature can be WEAPONIZED:
  
  Attack: Lockout Denial of Service
    1. Attacker identifies the LDAP server (port 389/636)
    2. Enumerates valid usernames (via anonymous queries or guessing)
    3. Floods authentication requests with WRONG passwords for each user
    4. Bad password count exceeds threshold (e.g., 5 attempts)
    5. Accounts become LOCKED — legitimate users denied access
    
  Impact:
    ❌ Denial of Service — Hundreds of users locked out
    ❌ Noise Generation — Distracts security team from real attack
    ❌ Business Disruption — Users can't access email, VPN, workstations

THE THREE PHASES OF IDENTITY-BASED ATTACKS:

  1. ENUMERATION (Reconnaissance):
     Finding valid usernames before trying passwords.
     
     Techniques:
       - Different error messages for "Invalid Username" vs "Invalid Password"
       - Response time differences (non-existent user fails instantly,
         real user takes time to check password hash)
       - Anonymous LDAP queries returning user lists
       
     Defense: ALWAYS return generic "Bad username or password" error.
     Never distinguish between invalid username and invalid password.

  2. PATTERN (Password Spraying):
     "Low and slow" approach to avoid lockout.
     
     Traditional Brute Force: Try 100 passwords on ONE account
       → Locks the account after 5 attempts. DETECTED.
       
     Password Spraying: Try ONE password on 100 accounts
       → Only 1 bad attempt per account. Under the radar.
       → Wait 30 minutes, try second password on all 100 accounts.
       → After 4 rounds: 400 attempts, zero lockouts.
       → If 2 users had weak passwords → Attacker is IN.
       
     Commonly tried passwords:
       Season2026!, Password123, Company2026!, Welcome1

  3. HONEYPOT (Deception):
     Defender creates FAKE accounts that look legitimate:
       - svc-backup, ftp-user, admin-test
       - Very long password (64 chars, impossible to guess)
       - No real access to any data
       - Monitoring rule: "ANY login attempt → alert immediately"
       
     When attacker includes honeypot in spray → instant detection.
     Provides: source IP, time, attack pattern intelligence.

DEFENSE IN DEPTH FOR LDAP:

  ✅ Network Segmentation — Don't expose LDAP to internet/untrusted networks
  ✅ Smart Lockout — Distinguish user mistype from botnet (ADFS smart lockout)
  ✅ Application-Level Throttling — CAPTCHA, cool-down before forwarding to LDAP
  ✅ Monitoring — Alert on unusual spikes in bad password events
  ✅ Generic Errors — Never reveal whether username exists
  ✅ Honeypot Accounts — Trap attackers during enumeration/spraying
```

---

# 11. DHCP — Dynamic Host Configuration Protocol

## 11.1 What Is DHCP?

```
DHCP (Dynamic Host Configuration Protocol):

  DHCP automates the assignment of IP addresses and other network
  configuration parameters to devices on a network.
  
  Without DHCP: You'd have to manually configure every device:
    - IP address
    - Subnet mask
    - Default gateway
    - DNS server(s)
    
  With DHCP: Devices get all configuration automatically when
  they connect to the network. Plug in → get an IP → start working.
  
  Ports: 67 (server), 68 (client)
  Protocol: UDP (connectionless)

THE DHCP PROCESS — DORA:

  ┌──────────┐                              ┌──────────┐
  │  CLIENT  │                              │  SERVER  │
  └────┬─────┘                              └────┬─────┘
       │                                         │
       │──── 1. DISCOVER (broadcast) ───────────▶│
       │     "Is there a DHCP server?"           │
       │     Src: 0.0.0.0  Dst: 255.255.255.255 │
       │                                         │
       │◀─── 2. OFFER ──────────────────────────│
       │     "Here's an IP you can use"          │
       │     Offered IP: 192.168.1.100           │
       │                                         │
       │──── 3. REQUEST (broadcast) ────────────▶│
       │     "I want that IP please"             │
       │     Requested IP: 192.168.1.100         │
       │                                         │
       │◀─── 4. ACK ───────────────────────────│
       │     "It's yours! Lease: 24 hours"       │
       │                                         │
       │     CLIENT NOW HAS:                     │
       │     IP: 192.168.1.100                   │
       │     Mask: 255.255.255.0                 │
       │     Gateway: 192.168.1.1                │
       │     DNS: 8.8.8.8, 8.8.4.4              │
       └─────────────────────────────────────────┘

  WHY IS DISCOVER A BROADCAST?
    Because the client has NO IP address yet — it literally doesn't
    know who to ask. It shouts to the entire network "HELP!"
    
  WHY IS REQUEST A BROADCAST (not unicast)?
    To tell ALL DHCP servers on the network "I accepted THIS server's
    offer" — so other servers can retract their offers.
```

## 11.2 DHCP Attacks

```
DHCP ATTACKS — EXPLOITING TRUST:

  DHCP was designed in an era of inherent network trust.
  It has NO built-in authentication. Any device can claim to
  be a DHCP server. This makes it a prime attack target.

ATTACK 1: DHCP STARVATION (Availability Attack)

  How it works:
    Attacker floods the network with DHCPDISCOVER requests,
    each using a DIFFERENT, SPOOFED MAC address.
    
  Goal: Exhaust the DHCP server's pool of available IP addresses.
  
  Result: Once the pool is empty, legitimate new clients requesting
  an IP address are DENIED service → Denial of Service.
  
  Tool: Yersinia, DHCPig
  
  Command example (with Yersinia):
    yersinia dhcp -attack 1   # DHCP starvation attack

ATTACK 2: ROGUE DHCP SERVER (Integrity & Confidentiality Attack)

  How it works:
    Attacker sets up a MALICIOUS DHCP server on the network.
    When clients broadcast DHCPDISCOVER, the rogue server responds
    BEFORE the legitimate one.
    
  Goal: Assign MALICIOUS network configurations:
    - Set attacker as DEFAULT GATEWAY → Man-in-the-Middle
    - Hand out attacker's DNS server → DNS hijacking (redirect to phishing)
    - Assign wrong subnet mask → Network disruption
    
  This is devastatingly effective because clients trust ANY DHCP response.

ATTACK 3: TUNNELVISION / VPN DECLOAKING (CVE-2024-3661)

  How it works:
    Attacker with access to the local network forces a VPN user's
    traffic OUT of the encrypted tunnel by manipulating DHCP options.
    The attacker sets their own machine as the gateway via DHCP.
    
  Goal: Bypass VPN security. Traffic the user believed was encrypted
  now flows through the attacker's machine in cleartext.
  
  Impact: Complete bypass of VPN protection on the local network.

DEFENSES:

  DHCP SNOOPING (Critical Layer-2 Switch Feature):
    ┌─────────────────────────────────────────────────────────┐
    │ Switch ports are classified:                             │
    │                                                         │
    │   TRUSTED PORTS: Connected to legitimate DHCP servers  │
    │   UNTRUSTED PORTS: Connected to clients                │
    │                                                         │
    │   The switch BLOCKS DHCP server responses              │
    │   (DHCPOFFER, DHCPACK) from UNTRUSTED ports.          │
    │                                                         │
    │   This PREVENTS rogue DHCP servers from responding.    │
    │                                                         │
    │   Also builds a "DHCP Snooping Binding Table":        │
    │   Maps valid IP → MAC assignments.                     │
    │   Used by Dynamic ARP Inspection (DAI) for            │
    │   additional protection.                               │
    └─────────────────────────────────────────────────────────┘
  
  PORT SECURITY: Limit MACs per port → mitigates starvation
  NETWORK MONITORING: Alert on high DHCPDISCOVER rates
  VLAN SEGMENTATION: Isolate DHCP servers in secure VLAN
```

---

# 12. ARP — Address Resolution Protocol

## 12.1 What Is ARP?

```
ARP (Address Resolution Protocol):

  ARP maps NETWORK LAYER addresses (IP) to DATA LINK LAYER
  addresses (MAC) within a LOCAL network.
  
  WHY ARP IS NEEDED:
    When your computer wants to send data to 192.168.1.1,
    it needs to know the MAC address of 192.168.1.1 to
    create the Ethernet frame. IP addresses are logical;
    Ethernet switches forward based on MAC addresses.
    
  HOW ARP WORKS:
  
    1. Device A wants to talk to 192.168.1.1
    2. Device A checks its ARP CACHE (local table of IP→MAC mappings)
    3. If not in cache → sends ARP REQUEST (broadcast):
       "Who has IP 192.168.1.1? Tell 192.168.1.5"
    4. ALL devices on the network receive this broadcast
    5. The device WITH 192.168.1.1 sends ARP REPLY (unicast):
       "192.168.1.1 is at MAC AA:BB:CC:DD:EE:FF"
    6. Device A stores this mapping in its ARP cache
    7. Device A can now send Ethernet frames to that MAC address

THE FUNDAMENTAL SECURITY FLAW:

  ARP is STATELESS and has NO AUTHENTICATION.
  
  ❌ ARP accepts responses even if it NEVER sent a request
  ❌ ARP blindly updates its cache with ANY ARP reply it receives
  ❌ There is NO way to verify an ARP reply is legitimate
  
  This is by design — ARP was created in a trusted network era.
  This fundamental lack of trust is what makes ARP spoofing possible.
```

## 12.2 ARP Spoofing / Poisoning Attack

```
ARP SPOOFING (ARP POISONING) — MAN-IN-THE-MIDDLE:

  The attacker sends FORGED ARP REPLIES to trick devices into
  updating their ARP caches with the attacker's MAC address.

  ATTACK MECHANICS:
  
    BEFORE ATTACK:
    
      Victim's ARP Cache:         Gateway's ARP Cache:
      192.168.1.1 → GW-MAC        192.168.1.5 → VICTIM-MAC
      
      Victim ←──── Normal traffic ────→ Gateway ←──→ Internet
      
    STEP 1: Attacker sends forged ARP replies:
    
      To Victim:  "192.168.1.1 (gateway) is at ATTACKER-MAC"
      To Gateway: "192.168.1.5 (victim) is at ATTACKER-MAC"
      
    STEP 2: ARP caches are poisoned:
    
      Victim's ARP Cache:         Gateway's ARP Cache:
      192.168.1.1 → ATTACKER-MAC  192.168.1.5 → ATTACKER-MAC
      
    STEP 3: All traffic now flows through attacker:
    
      Victim ──→ ATTACKER ──→ Gateway ──→ Internet
      Victim ←── ATTACKER ←── Gateway ←── Internet
      
    STEP 4: Attacker enables IP forwarding to stay hidden.
      Traffic flows normally, but attacker can:
      ✓ Eavesdrop (read passwords, emails, cookies)
      ✓ Modify data in transit (inject malicious code)
      ✓ Steal session tokens (session hijacking)
      ✓ Redirect to fake websites

  TOOLS:
    arpspoof (dsniff package): arpspoof -i eth0 -t <victim> <gateway>
    Ettercap: GUI-based MitM tool with ARP spoofing
    Bettercap: Modern, powerful MitM framework
    
IMPACT:
    ❌ Man-in-the-Middle — Eavesdrop on ALL traffic
    ❌ Data Manipulation — Alter data in transit
    ❌ Session Hijacking — Steal authentication tokens
    ❌ Credential Theft — Capture plaintext passwords
    ❌ Denial of Service — Map all IPs to non-existent MAC

DEFENSES:

  ✅ Dynamic ARP Inspection (DAI):
     Switch feature using DHCP Snooping Binding Table.
     Validates ARP packets against known-good IP-MAC bindings.
     Invalid ARP packets are DROPPED. Most effective defense.
     
  ✅ Static ARP Entries:
     Manually define ARP entries for critical devices (routers, servers).
     arp -s 192.168.1.1 AA:BB:CC:DD:EE:FF
     Not scalable but effective for critical infrastructure.
     
  ✅ Port Security + 802.1X:
     Limit MAC addresses per port. Require authentication before
     network access. Prevents unauthorized devices.
     
  ✅ Encryption:
     Use HTTPS, SSH, VPNs. Doesn't prevent ARP spoofing but
     renders intercepted data unreadable.
     
  ✅ Detection Tools:
     ARPWatch, XArp, Snort IDS — Monitor for ARP anomalies.
```

---

# 13. NTP — Network Time Protocol

## 13.1 What Is NTP?

```
NTP (Network Time Protocol):

  Port: 123 (UDP)
  Purpose: Synchronize clocks between computer systems.
  
  WHY ACCURATE TIME MATTERS FOR SECURITY:
    ✓ Log correlation — Correlating events across systems requires
      synchronized timestamps. Without it, incident investigation fails.
    ✓ Kerberos authentication — Kerberos tickets have timestamps.
      If clock skew exceeds 5 minutes, authentication FAILS.
    ✓ Certificate validation — SSL/TLS certificates have validity periods.
    ✓ Scheduled tasks — Backups, patches, cron jobs depend on time.
    ✓ Digital forensics — Accurate timestamps are evidence.
    
  NTP uses a hierarchical system of time sources:
    Stratum 0: Atomic clocks, GPS receivers (reference clocks)
    Stratum 1: Servers directly connected to Stratum 0 devices
    Stratum 2: Servers synchronized to Stratum 1 servers
    ...and so on, up to Stratum 15. Stratum 16 = unsynchronized.
```

## 13.2 NTP Amplification Attack (DDoS)

```
NTP AMPLIFICATION — DDoS VIA TIME SERVERS:

  A type of Distributed Denial of Service (DDoS) attack that
  leverages publicly accessible NTP servers to overwhelm a target.
  It is both a REFLECTION and AMPLIFICATION attack.

THE VULNERABILITY — monlist COMMAND:

  Older NTP implementations have a diagnostic command called
  "monlist" (MON_GETLIST) enabled by default.
  
  monlist returns a list of the last 600 hosts that have connected
  to the NTP server. The request is tiny (~40 bytes), but the
  response is MASSIVE (often 100x larger).
  
  Amplification factor: 1:100 or higher!
  
ATTACK MECHANICS:

  1. SPOOFING: Attacker crafts small NTP monlist requests
     with the SOURCE IP ADDRESS forged to be the VICTIM's IP.
     
  2. REFLECTION: Attacker sends these spoofed requests to
     thousands of vulnerable NTP servers on the internet.
     
  3. AMPLIFICATION: Each NTP server sends its large monlist
     response (100x the request size) to the VICTIM's IP.
     
  4. OVERLOAD: Victim receives a massive flood of amplified
     traffic from thousands of NTP servers simultaneously.
     Network saturated → Denial of Service.

  Attacker                  NTP Servers               Victim
    │                          │                        │
    │─── Tiny request ────────▶│                        │
    │    (src IP = victim)     │                        │
    │                          │── MASSIVE response ──▶│
    │                          │   (100x amplified)    │
    │                          │                        │ OVERLOADED!
    │─── Tiny request ────────▶│                        │
    │    (src IP = victim)     │── MASSIVE response ──▶│
    │                          │                        │ DOWN!

NTP RELAY ATTACK (TIME MANIPULATION):

  A different type of NTP attack where the goal is to CHANGE
  the victim's system clock, not flood them with traffic.
  
  Using ARP spoofing or another MitM technique, the attacker
  intercepts NTP requests and modifies the timestamp in the response.
  
  Why manipulate time?
    ❌ Bypass Security Tokens — TOTP codes expire/become invalid
    ❌ Expire Encryption Keys — Certificates appear expired
    ❌ Log Tampering — Wrong timestamps break forensic analysis
    ❌ Cache Poisoning — Time-dependent cache becomes invalid
    ❌ Kerberos Failure — Authentication breaks (5-min tolerance)

DEFENSES:

  ✅ Patch NTP — Upgrade to ntp-4.2.7p26+ (monlist disabled)
  ✅ Disable monlist — Add "disable monitor" to ntp.conf
  ✅ Ingress Filtering (BCP 38) — Block packets with spoofed IPs
  ✅ Rate Limiting — Limit incoming NTP traffic volume
  ✅ DDoS Mitigation — Cloudflare, Akamai, AWS Shield
  ✅ NTP Authentication — Use symmetric key or Autokey authentication
```

---

# 14. DNS — Domain Name System

## 14.1 What Is DNS?

```
DNS (Domain Name System):

  Port: 53 (TCP and UDP)
  Purpose: Translates human-readable domain names to IP addresses.
  
  Without DNS: You'd type http://142.250.185.78 instead of google.com
  
  DNS is one of the most CRITICAL services on the Internet.
  If DNS fails, NOTHING works — no web, no email, no applications.
  
  THE PHONE BOOK ANALOGY:
    You look up a person's name (domain name) in a phone book
    to find their phone number (IP address).
    
    google.com → 142.250.185.78
    example.com → 93.184.216.34
    
DNS RECORD TYPES:
  ┌──────────┬─────────────────────────────────────────────────────┐
  │ Record   │ Purpose                                              │
  ├──────────┼─────────────────────────────────────────────────────┤
  │ A        │ Maps domain to IPv4 address                          │
  │ AAAA     │ Maps domain to IPv6 address                          │
  │ CNAME    │ Alias — points one domain to another domain         │
  │ MX       │ Mail exchange — where to deliver email               │
  │ NS       │ Nameserver — authoritative DNS servers               │
  │ PTR      │ Reverse DNS — maps IP to domain name                │
  │ TXT      │ Arbitrary text — used for SPF, DKIM, DMARC          │
  │ SRV      │ Service locator — specifies host/port for services  │
  │ SOA      │ Start of Authority — zone administrative info       │
  └──────────┴─────────────────────────────────────────────────────┘

DNS SECURITY THREATS:
  DNS Cache Poisoning — Inject false DNS records into cache
  DNS Tunneling — Exfiltrate data by encoding it in DNS queries
  DNS Amplification — DDoS using open DNS resolvers (like NTP amp)
  DNS Hijacking — Modify DNS settings to redirect traffic
  Domain Squatting — Register look-alike domains for phishing
  
DEFENSES:
  DNSSEC — Cryptographic signatures on DNS records
  DNS over HTTPS (DoH) / DNS over TLS (DoT) — Encrypted DNS queries
  Response Rate Limiting — Mitigate DNS amplification
  DNS Monitoring — Detect anomalous query patterns
```

---

# 15. SMTP / SMTPS — Email Protocols

```
SMTP (Simple Mail Transfer Protocol):

  Ports: 25 (SMTP), 465 (SMTPS), 587 (Submission)
  Purpose: SEND email between mail servers
  
  SMTP is a TEXT-BASED protocol. You can literally telnet to
  a mail server and type email commands manually:
  
  # Manual SMTP session (for educational purposes):
  telnet mail.example.com 25
  HELO attacker.com
  MAIL FROM:<ceo@company.com>     ← Can be ANY address (spoofed!)
  RCPT TO:<employee@company.com>
  DATA
  Subject: Urgent Wire Transfer
  Send $50,000 to this account immediately.
  .
  QUIT
  
  THIS IS WHY EMAIL SPOOFING IS SO EASY.
  SMTP has NO built-in authentication of the sender.

EMAIL SECURITY PROTOCOLS:

  SPF (Sender Policy Framework):
    DNS TXT record listing which servers can send email for your domain.
    Receiver checks: "Is this server authorized to send for @company.com?"
    
  DKIM (DomainKeys Identified Mail):
    Cryptographic signature added to email headers.
    Proves the email was not modified in transit.
    
  DMARC (Domain-based Message Authentication, Reporting & Conformance):
    Policy telling receivers what to do when SPF/DKIM checks fail.
    Options: none (monitor), quarantine, reject
    
  Without SPF + DKIM + DMARC:
    Anyone can send email AS your domain.
    Phishing becomes trivially easy.
    Business Email Compromise (BEC) costs billions annually.
```

---

# 16. SNMP — Simple Network Management Protocol

```
SNMP (Simple Network Management Protocol):

  Ports: 161 (queries), 162 (traps/notifications)
  Purpose: Monitor and manage network devices (routers, switches,
  servers, printers, firewalls)
  
  SNMP allows network administrators to:
    - Monitor device health (CPU, memory, bandwidth)
    - Configure devices remotely
    - Receive alerts when something goes wrong (traps)
    
SNMP VERSIONS — CRITICAL SECURITY DIFFERENCES:

  ┌──────────┬──────────────────────────────────────────────────────┐
  │ Version  │ Security                                              │
  ├──────────┼──────────────────────────────────────────────────────┤
  │ SNMPv1   │ Community strings (passwords) sent in PLAINTEXT.     │
  │          │ Default: "public" (read), "private" (read-write).   │
  │          │ ZERO encryption. ZERO authentication.                │
  │          │ NEVER USE IN PRODUCTION.                              │
  ├──────────┼──────────────────────────────────────────────────────┤
  │ SNMPv2c  │ Still uses community strings in PLAINTEXT.           │
  │          │ Better performance but same security problems.       │
  │          │ AVOID if possible.                                    │
  ├──────────┼──────────────────────────────────────────────────────┤
  │ SNMPv3   │ ENCRYPTED communication (DES, AES).                  │
  │          │ AUTHENTICATED users (username + password + engine).  │
  │          │ Integrity verification (HMAC-MD5/SHA).               │
  │          │ USE THIS VERSION. Always.                              │
  └──────────┴──────────────────────────────────────────────────────┘

  SECURITY RISK:
    If SNMPv1/v2c is running with default community strings:
    - Attacker can read ENTIRE device configuration
    - Attacker can modify device settings (with "private" community)
    - Attacker can extract routing tables, ARP tables, user lists
    - Attacker can disable interfaces, change passwords
    
  ALWAYS: Use SNMPv3. Change default community strings. Restrict
  SNMP access to management network only via ACLs.
```

---

# 17. TFTP — Trivial File Transfer Protocol

```
TFTP (Trivial File Transfer Protocol):

  Port: 69 (UDP)
  Purpose: Simple file transfer with NO authentication
  
  TFTP is designed for simplicity, not security:
    ❌ No authentication — Anyone can read/write files
    ❌ No encryption — Data transferred in plaintext
    ❌ No directory listing — Must know exact filename
    ❌ UDP-based — Unreliable, no connection state
    
  Use Cases:
    - PXE boot (network boot of diskless workstations)
    - Router/switch firmware upgrades
    - Configuration backup/restore on network devices
    
  SECURITY WARNING:
    TFTP should NEVER be exposed to untrusted networks.
    It should be restricted to isolated management VLANs.
    Attackers finding an open TFTP server can:
    - Download device configurations (containing passwords)
    - Upload malicious firmware
    - Overwrite critical files
```

---

# 18. SIP & VoIP — Voice Over IP Security

## 18.1 Understanding SIP and VoIP

```
VoIP (Voice over Internet Protocol):
  The technology that allows voice communication over IP networks,
  replacing traditional circuit-switched telephone networks.

SIP (Session Initiation Protocol):
  The dominant SIGNALING protocol used to set up, manage, and
  tear down VoIP calls. It handles the "control plane."
  
  Ports: 5060 (SIP), 5061 (SIP over TLS)

RTP (Real-time Transport Protocol):
  Carries the actual voice and video media (the "data plane")
  once a call is established by SIP.

THE RESTAURANT ANALOGY:

  You and your friend are having a conversation at a restaurant.
  
  The WAITER is SIP:
    1. You tell the waiter: "I want to talk to my friend at table 5"
       → SIP sends INVITE to the other party
    2. Waiter goes to table 5, gets attention
       → SIP locates the user, their phone rings
    3. Friend says "Sure, I'll talk"
       → SIP returns "200 OK" — connection established
    4. You start talking directly to each other
       → Voice travels via RTP, NOT through SIP
    5. When done, you tell the waiter "We're finished"
       → SIP sends BYE to terminate the session
       
  SIP is the MANAGER — it sets up and tears down the call.
  SIP does NOT carry your voice. RTP carries your voice.
```

## 18.2 SIP/VoIP Attacks

```
VoIP/SIP ATTACKS:

  Because VoIP runs on IP networks, it inherits ALL vulnerabilities
  of traditional data networks PLUS telephony-specific threats.

  1. REGISTRATION HIJACKING:
     Attacker impersonates a valid user and re-registers their
     contact address with the SIP registrar.
     Incoming calls for the victim are routed to the attacker.
     
  2. INVITE FLOODING (DoS):
     Flooding a SIP server with INVITE requests overwhelms it,
     preventing it from processing legitimate calls.
     
  3. SIP MESSAGE TAMPERING:
     Modifying SIP messages in transit to redirect calls,
     drop calls, or change service parameters.
     
  4. EAVESDROPPING / CALL INTERCEPTION:
     Using packet sniffers (Wireshark) on the same network to
     capture RTP voice packets. If unencrypted, the attacker
     can reassemble packets and LISTEN to conversations.
     
  5. MAN-IN-THE-MIDDLE:
     Using ARP spoofing or rogue DHCP to intercept, modify,
     or record conversations silently.
     
  6. VISHING (Voice Phishing):
     Social engineering attacks using VoIP — calling targets
     and impersonating banks, IT support, or executives.
     
  7. TOLL FRAUD:
     Attackers compromise a VoIP system to make premium-rate
     or international calls, generating massive bills.
     
  8. SPIT (Spam over Internet Telephony):
     The VoIP equivalent of email spam — automated bulk calls.

DEFENSES:

  ✅ Session Border Controller (SBC):
     The VoIP firewall. Sits at the border between private VoIP
     and public internet. Provides:
     - Topology hiding (masks internal IPs)
     - Protocol validation (inspects SIP traffic)
     - DoS/DDoS protection (filters malicious SIP)
     - Encryption management (TLS + SRTP)
     
  ✅ Encryption:
     TLS for SIP signaling + SRTP for voice media
     Prevents eavesdropping and tampering
     
  ✅ Separate VLANs:
     Isolate VoIP on its own VLAN. Prevents direct access
     from data network devices.
     
  ✅ Strong Authentication:
     Enforce strong passwords for voicemail and admin access.
     Consider MFA for VoIP administrative interfaces.
     
  ✅ IDS/IPS:
     Deploy with SIP-specific attack signatures.
```

---

# 19. SMB — Server Message Block

## 19.1 What Is SMB?

```
SMB (Server Message Block):

  Port: 445 (modern), 139 (legacy over NetBIOS)
  Purpose: Network file sharing protocol for Windows.
  Allows reading/writing files and requesting services
  from server programs over a network.
  
  Versions:
    SMBv1 — Original. INSECURE. OBSOLETE. Disable immediately.
    SMBv2 — Introduced in Windows Vista. Major improvements.
    SMBv3 — Introduced in Windows 8. Encryption support.
    SMBv3.1.1 — Windows 10+. Pre-auth integrity, AES-128-GCM.
    
  CRITICAL SECURITY RULE:
    DISABLE SMBv1 ON ALL SYSTEMS.
    SMBv1 is the protocol exploited by EternalBlue (WannaCry, NotPetya).
    There is NO reason to run SMBv1 in a modern network.
    
    # Disable SMBv1 on Windows:
    Disable-WindowsOptionalFeature -Online -FeatureName smb1protocol
    
    # Check SMBv1 status:
    Get-WindowsOptionalFeature -Online -FeatureName smb1protocol
```

---

# PART III — FIREWALLS — THE COMPLETE GUIDE

---

# 20. What Is a Firewall — Absolute Fundamentals

## 20.1 The Real-World Analogy

```
WHAT IS A FIREWALL?

  Imagine a MILITARY CHECKPOINT at the entrance to a secure base.
  Every vehicle approaching must stop. A guard checks:
  
  ┌──────────────────────────────────────────────────────────────┐
  │  FIREWALL = THE GUARD AT THE NETWORK GATE                    │
  ├──────────────────────────────────────────────────────────────┤
  │  Who are you?           → Source IP address                  │
  │  Where are you going?   → Destination IP address             │
  │  What's your purpose?   → Port number / service              │
  │  What are you carrying? → Packet payload (advanced firewalls)│
  │  Do you have clearance? → Does a rule permit this traffic?   │
  └──────────────────────────────────────────────────────────────┘
  
  If the vehicle matches an approved entry on the guard's clipboard,
  it passes through. If not, it gets turned away — or silently
  ignored as if the checkpoint doesn't even exist.
  
  The FIREWALL is the guard.
  The CLIPBOARD is the rule set.
  The CHECKPOINT is the network boundary.

TECHNICAL DEFINITION:

  A firewall is a network security device — hardware, software, or
  both — that MONITORS incoming and outgoing network traffic and
  DECIDES whether to ALLOW or BLOCK specific traffic based on a
  defined set of SECURITY RULES.
  
  Key points:
    MONITORS:  Inspects every packet passing through it
    TRAFFIC:   Both INBOUND (entering) AND OUTBOUND (leaving)
    BLOCK/ALLOW: The two fundamental actions — ACCEPT or DROP
    RULES:     The firewall follows instructions humans wrote
```

## 20.2 Where Firewalls Sit in the Network

```
FIREWALL PLACEMENT:

                  ┌─────────────────────────────────────────────┐
                  │              THE INTERNET                    │
                  │         (Untrusted Territory)                │
                  └─────────────────┬───────────────────────────┘
                                    │
                                    │  All traffic enters here
                                    ▼
                            ┌───────────────┐
                            │    GATEWAY     │
                            │   (Router)     │
                            └───────┬───────┘
                                    │
                           ═════════╪═════════  ← Security Boundary
                                    │
                            ┌───────────────┐
                            │   FIREWALL     │  ← Inspects ALL traffic
                            │ Rule 1: HTTP   │     crossing boundary
                            │ Rule 2: HTTPS  │
                            │ Rule 3: SSH    │
                            │ Rule 4: DROP * │
                            └───────┬───────┘
                                    │
                  ┌─────────────────┴───────────────────────────┐
                  │           INTERNAL NETWORK                   │
                  │         (Protected Territory)                │
                  │  ┌────────┐  ┌──────────┐  ┌────────┐      │
                  │  │ Server │  │Workstation│  │ Printer│      │
                  │  └────────┘  └──────────┘  └────────┘      │
                  └─────────────────────────────────────────────┘

  In many cases, the gateway and firewall are the SAME device —
  pfSense, Cisco ASA, or FortiGate often serve as both router
  and firewall. In enterprise networks, they're typically separate.
```

## 20.3 The Three Fundamental Actions

```
EVERY FIREWALL TAKES ONE OF THREE ACTIONS:

  ┌──────────┬──────────────────────────────────┬────────────────────┐
  │ Action   │ What Happens                     │ Analogy             │
  ├──────────┼──────────────────────────────────┼────────────────────┤
  │ ACCEPT   │ Packet is allowed through        │ Guard waves vehicle │
  │          │                                  │ through the gate    │
  ├──────────┼──────────────────────────────────┼────────────────────┤
  │ DROP     │ Packet is silently discarded.    │ Guard ignores the   │
  │          │ NO response sent to sender.      │ vehicle completely  │
  ├──────────┼──────────────────────────────────┼────────────────────┤
  │ REJECT   │ Packet is discarded AND an       │ Guard tells vehicle │
  │          │ error message sent back           │ "access denied"     │
  │          │ (ICMP error or TCP RST)          │                     │
  └──────────┴──────────────────────────────────┴────────────────────┘
  
  OSCP NOTE: DROP is more secure than REJECT for external traffic.
  REJECT tells the attacker "something exists here but you can't access it."
  DROP gives them NOTHING — they don't know if the host even exists.
  However, DROP can slow down Nmap scans (tool waits for timeouts).
```

## 20.4 Stateful vs. Stateless Firewalls

```
THE MOST IMPORTANT DISTINCTION IN FIREWALL TECHNOLOGY:

  STATELESS FIREWALL (First Generation):
    ❌ Examines each packet INDEPENDENTLY — no memory
    ❌ Doesn't understand "connections" — only individual packets
    ❌ Must have explicit rules for BOTH directions of traffic
    ✅ Very FAST but very DUMB
    
  STATEFUL FIREWALL (Modern Standard):
    ✅ Tracks the STATE of network connections (TCP handshake)
    ✅ Maintains a STATE TABLE — record of all active connections
    ✅ If connection was legitimately established, return traffic auto-allowed
    ❌ Slower but INTELLIGENT
    
  COMPARISON:
  
    STATELESS — Must define rules for BOTH directions:
      Rule 1: Allow TCP to port 80    (outbound web request)
      Rule 2: Allow TCP from port 80  (inbound response) ← Manual!
    
    STATEFUL — Only define the INITIATING direction:
      Rule 1: Allow outbound TCP to port 80
      [State Table auto-tracks the connection and allows the response]

  STATE TABLE EXAMPLE:
    ┌───────────────┬──────────────┬────────┬──────────────┐
    │ Source         │ Destination  │ State  │ Timeout      │
    ├───────────────┼──────────────┼────────┼──────────────┤
    │ 192.168.1.5   │ 93.184.216.34│ ESTAB  │ 3600s        │
    │ :49321        │ :80          │        │              │
    ├───────────────┼──────────────┼────────┼──────────────┤
    │ 192.168.1.10  │ 10.0.0.50   │ SYN_   │ 120s         │
    │ :52100        │ :22          │ SENT   │              │
    └───────────────┴──────────────┴────────┴──────────────┘
```

## 20.5 Default Policy — The Safety Net

```
DEFAULT POLICY — WHAT HAPPENS WHEN NO RULE MATCHES:

  DEFAULT ALLOW (Blacklist — INSECURE):
    Everything is PERMITTED unless explicitly DENIED.
    Easy to manage but fundamentally insecure.
    One missed rule = one vulnerability.
    
  DEFAULT DENY (Whitelist — SECURE):
    Everything is BLOCKED unless explicitly ALLOWED.
    Harder to manage but fundamentally secure.
    One missed rule = one service unavailable (but secure).
    REQUIRED by every security standard: PCI-DSS, HIPAA, SOC2.

  DEFAULT ALLOW (Bad):          DEFAULT DENY (Good):
  ┌──────────────────────┐      ┌──────────────────────┐
  │ Rule 1: BLOCK port X │      │ Rule 1: ALLOW port 22│
  │ Rule 2: BLOCK port Y │      │ Rule 2: ALLOW port 80│
  │ Rule 3: BLOCK port Z │      │ Rule 3: ALLOW port 443│
  │ Default: ALLOW ALL  ←│ BAD  │ Default: DROP ALL   ←│ GOOD
  └──────────────────────┘      └──────────────────────┘
  "Block bad, allow rest"       "Allow good, block rest"
  
  ALWAYS use Default DENY. ALWAYS.
```

---

# 21. How Firewalls Work Internally — Packet Processing Pipeline

```
WHEN A PACKET ARRIVES, THE FIREWALL PROCESSES IT THROUGH A PIPELINE:

  Packet Arrives
       │
       ▼
  ┌─────────────┐
  │ 1. CAPTURE  │  Packet received on network interface
  └──────┬──────┘
         │
         ▼
  ┌─────────────────┐
  │ 2. DECODE        │  Headers parsed:
  │    HEADERS       │  - Ethernet → MAC addresses
  │                  │  - IP header → Source/Dest IP, TTL, Protocol
  │                  │  - TCP/UDP → Source/Dest Port, Flags
  └──────┬──────────┘
         │
         ▼
  ┌─────────────────┐
  │ 3. STATE TABLE   │  Part of existing connection?
  │    LOOKUP        │  YES → Fast-path: Allow
  │                  │  NO  → Continue to rule evaluation
  └──────┬──────────┘
         │ (New connection)
         ▼
  ┌─────────────────┐
  │ 4. RULE          │  Compare packet against rules TOP-TO-BOTTOM
  │    EVALUATION    │  FIRST MATCHING RULE WINS
  │                  │  
  │  Rule 1: Match? ─── YES → Execute action (ACCEPT/DROP/REJECT)
  │  Rule 2: Match? ─── YES → Execute action
  │  Rule 3: Match?
  │  ...
  │  Default Policy ─── No rule matched → Apply default action
  └──────┬──────────┘
         │
         ▼
  ┌─────────────────┐
  │ 5. ACTION &      │  ACCEPT: Forward to destination
  │    LOGGING       │  DROP: Silently discard
  │                  │  REJECT: Discard + send error
  │                  │  LOG: Write to log (then continue)
  └─────────────────┘

CRITICAL CONCEPT — FIRST MATCH WINS:

  Rules are evaluated TOP TO BOTTOM.
  The FIRST rule that matches determines the action.
  ALL subsequent rules are IGNORED for that packet.
  
  # WRONG ORDER — SSH rule is useless!
  Rule 1: DROP   all traffic from any to any     ← Matches EVERYTHING
  Rule 2: ACCEPT tcp from any to any port 22     ← NEVER reached!
  
  # CORRECT ORDER — Specific allows first, then deny-all
  Rule 1: ACCEPT tcp from any to any port 22     ← SSH matches here
  Rule 2: ACCEPT tcp from any to any port 80     ← HTTP matches here
  Rule 3: DROP   all traffic from any to any     ← Everything else dropped
  
  OSCP TIP: When auditing a firewall, ALWAYS check the rule order.
  A common misconfiguration: placing a broad ALLOW above specific DENYs.
```

---

# 22. Firewall Types — Complete Classification

## 22.1 Layer 3 Firewalls — Packet Filtering

```
LAYER 3 / PACKET FILTERING FIREWALL:

  The simplest and fastest type. Decisions based on IP header
  and transport header ONLY:
  
  ┌──────────────────────────────────────────────────────────┐
  │  What It CAN See:          What It CANNOT See:           │
  │                                                          │
  │  ✓ Source IP               ✗ HTTP request content        │
  │  ✓ Destination IP          ✗ SQL injection in payload    │
  │  ✓ Source Port             ✗ Malware in file download    │
  │  ✓ Destination Port        ✗ User identity               │
  │  ✓ Protocol (TCP/UDP/ICMP) ✗ Application behavior        │
  │  ✓ TCP Flags               ✗ Encrypted content (HTTPS)   │
  └──────────────────────────────────────────────────────────┘
  
  Use Case: High-speed perimeter filtering, ACLs on routers.
  Limitation: Cannot detect application-layer attacks. SQL injection
  to port 80 looks identical to legitimate web traffic.
```

## 22.2 Layer 7 Firewalls — Application-Level (WAF)

```
LAYER 7 / APPLICATION FIREWALL (WAF):

  Inspects the ACTUAL CONTENT (payload) of packets.
  Understands HTTP, FTP, DNS, SMTP protocols deeply.
  
  WAF INSPECTION EXAMPLES:
  
    Normal Request:
      GET /index.html HTTP/1.1
      Host: example.com
      → WAF: ✅ Normal. ALLOW.
      
    SQL Injection Attempt:
      GET /search?q=1' OR '1'='1' -- HTTP/1.1
      → WAF: 🚫 SQL injection pattern! BLOCK.
      
    XSS Attempt:
      POST /comment HTTP/1.1
      Body: message=<script>document.cookie</script>
      → WAF: 🚫 Cross-site scripting! BLOCK.
      
    Command Injection:
      GET /api/ping?host=8.8.8.8;cat /etc/passwd HTTP/1.1
      → WAF: 🚫 OS command injection! BLOCK.

  WAF DETECTION METHODS:
    Signature-Based — Matches known attack patterns (regex rules)
    Anomaly-Based — Detects deviations from "normal" traffic baseline
    Behavioral — Tracks request rates, session patterns, bot behavior
    Reputation-Based — Blocks known malicious IPs, Tor exits
    
  COMMON WAFs:
    ModSecurity — Open-source, Apache/Nginx (OWASP CRS ruleset)
    AWS WAF — Cloud-native, integrates with CloudFront/ALB
    Cloudflare WAF — CDN-integrated, massive threat intelligence
    Imperva/Incapsula — Enterprise-grade, DDoS + WAF
    F5 BIG-IP ASM — Hardware appliance WAF
```

## 22.3 Next-Generation Firewalls (NGFW)

```
NEXT-GENERATION FIREWALLS (NGFW):

  Combines EVERYTHING into a single device:
  
  ┌──────────────────────────────────────────────────────┐
  │              NEXT-GENERATION FIREWALL                 │
  │                                                      │
  │  ┌────────────────────────────────────────────────┐  │
  │  │ Layer 3/4: Packet Filtering + Stateful         │  │
  │  │ - Source/Dest IP, Ports, Protocols              │  │
  │  │ - Connection state tracking                     │  │
  │  └────────────────────────────────────────────────┘  │
  │                      +                               │
  │  ┌────────────────────────────────────────────────┐  │
  │  │ Layer 7: Deep Packet Inspection (DPI)          │  │
  │  │ - Application identification regardless of port│  │
  │  │ - Content inspection & SSL/TLS decryption      │  │
  │  └────────────────────────────────────────────────┘  │
  │                      +                               │
  │  ┌────────────────────────────────────────────────┐  │
  │  │ Additional Services:                            │  │
  │  │ - Intrusion Prevention System (IPS)             │  │
  │  │ - Antivirus / Anti-malware                      │  │
  │  │ - URL filtering / Web categorization            │  │
  │  │ - User identity awareness (LDAP/AD integration) │  │
  │  │ - Application control (block Tor, BitTorrent)   │  │
  │  │ - Sandboxing (detonate suspicious files)        │  │
  │  │ - Threat intelligence feeds                      │  │
  │  └────────────────────────────────────────────────┘  │
  │                                                      │
  │  Vendors: Palo Alto, Fortinet, Cisco Firepower,      │
  │           Check Point, Sophos XG, SonicWall           │
  └──────────────────────────────────────────────────────┘

  KEY NGFW CAPABILITY — APPLICATION AWARENESS:
  
    Traditional Firewall:         NGFW:
    Port 443 → Allow HTTPS        Port 443 + Application ID:
    Port 443 → Allow HTTPS          → HTTPS (Google) ✅ Allow
    Port 443 → Allow HTTPS          → Tor Browser   🚫 Block
                                     → BitTorrent    🚫 Block
    Can't tell them apart!           → VPN Tunnel    🚫 Block
                                   Identifies the ACTUAL application!
```

## 22.4 AI-Based Firewalls

```
AI-BASED FIREWALLS — THE CUTTING EDGE:

  Use machine learning and behavioral analysis for decisions
  that go BEYOND static rules.
  
  How They Work:
    1. Training Phase — Learn what "normal" traffic looks like
    2. Baseline Creation — Build behavioral profiles
    3. Anomaly Detection — Flag deviations from baseline
    4. Adaptive Response — Auto-adjust rules based on threats
    5. Continuous Learning — Model improves over time
    
  Traditional Rule:                  AI-Based Decision:
  "Block if source IP is             "This IP has never connected to
   in blacklist"                     the database server before, and
                                     it's 3 AM, and request volume is
  Static. Binary. Dumb.              500x normal. Anomalous. BLOCK."
                                     Dynamic. Contextual. Intelligent.

  AI Firewall Vendors:
    Darktrace — "Enterprise Immune System"
    Palo Alto Cortex — ML-powered threat detection
    Fortinet FortiAI — Deep neural network detection
    Vectra AI — Network detection and response (NDR)
    
  Limitations:
    ❌ High false-positive rates during training
    ❌ Can be poisoned by adversarial ML techniques
    ❌ Expensive — requires significant compute resources
    ❌ Not a replacement for good rule-based policies
```

## 22.5 Complete Firewall Type Comparison

```
ALL FIREWALL TYPES — COMPARISON TABLE:

  ┌──────────────┬───────────┬────────────┬───────────┬────────────┐
  │ Feature      │ Layer 3   │ Layer 7    │ NGFW      │ AI-Based   │
  │              │ (Packet)  │ (App/WAF)  │           │            │
  ├──────────────┼───────────┼────────────┼───────────┼────────────┤
  │ IP/Port      │ ✅ Yes    │ ✅ Yes     │ ✅ Yes    │ ✅ Yes     │
  │ filtering    │           │            │           │            │
  ├──────────────┼───────────┼────────────┼───────────┼────────────┤
  │ Stateful     │ Optional  │ ✅ Yes     │ ✅ Yes    │ ✅ Yes     │
  ├──────────────┼───────────┼────────────┼───────────┼────────────┤
  │ Content      │ ❌ No     │ ✅ Yes     │ ✅ Yes    │ ✅ Yes     │
  │ inspection   │           │            │           │            │
  ├──────────────┼───────────┼────────────┼───────────┼────────────┤
  │ App aware    │ ❌ No     │ Partial    │ ✅ Yes    │ ✅ Yes     │
  ├──────────────┼───────────┼────────────┼───────────┼────────────┤
  │ IPS/IDS      │ ❌ No     │ ❌ No      │ ✅ Yes    │ ✅ Yes     │
  ├──────────────┼───────────┼────────────┼───────────┼────────────┤
  │ ML/AI        │ ❌ No     │ ❌ No      │ Limited   │ ✅ Core    │
  ├──────────────┼───────────┼────────────┼───────────┼────────────┤
  │ Speed        │ Fastest   │ Slow       │ Medium    │ Variable   │
  ├──────────────┼───────────┼────────────┼───────────┼────────────┤
  │ Cost         │ Lowest    │ Medium     │ High      │ Highest    │
  ├──────────────┼───────────┼────────────┼───────────┼────────────┤
  │ Example      │ ACL on    │ ModSec WAF │ Palo Alto │ Darktrace  │
  │              │ router    │            │ PA-Series │            │
  └──────────────┴───────────┴────────────┴───────────┴────────────┘

  Host-Based vs. Network-Based:
  ┌─────────────┬────────────────────────┬─────────────────────────┐
  │ Aspect      │ Network Firewall       │ Host-Based Firewall      │
  ├─────────────┼────────────────────────┼─────────────────────────┤
  │ Location    │ Network perimeter      │ On each individual host  │
  │ Scope       │ Protects entire network│ Protects only that host  │
  │ Examples    │ pfSense, Cisco ASA,    │ iptables, Windows FW,   │
  │             │ Palo Alto              │ macOS PF                 │
  │ Visibility  │ Sees ALL network traffic│ Only traffic to/from host│
  │ Management  │ Centralized            │ Distributed (harder)     │
  │ Role        │ First line of defense  │ Last line of defense     │
  └─────────────┴────────────────────────┴─────────────────────────┘
  
  Best Practice: Use BOTH. Network firewall blocks broad threats
  at the perimeter. Host-based provides per-machine protection —
  critical for lateral movement prevention during a breach.
```

---

# 23. The Linux Firewall Stack — Netfilter Architecture

```
THE LINUX NETFILTER ARCHITECTURE:

  Every Linux firewall tool — iptables, nftables, UFW, firewalld —
  is a FRONT-END to the same kernel framework: NETFILTER.
  
  Netfilter is a set of hooks inside the Linux kernel's networking
  stack that allow modules to process packets at specific points.

  ┌─────────────────────────────────────────────────────────────┐
  │                      USER SPACE                              │
  │                                                              │
  │   ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  │
  │   │ iptables │  │ nftables │  │   UFW    │  │ firewalld│  │
  │   │ (legacy) │  │ (modern) │  │ (simple) │  │ (zones)  │  │
  │   └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘  │
  │        │              │              │              │        │
  │        └──────────────┼──────────────┼──────────────┘        │
  │                       │   All are front-ends to:             │
  ├───────────────────────┼──────────────────────────────────────┤
  │                       ▼                                      │
  │                  KERNEL SPACE                                │
  │              ┌─────────────────────┐                        │
  │              │     NETFILTER       │                        │
  │              │  (Kernel Framework) │                        │
  │              │                     │                        │
  │              │  Hook: PREROUTING   │ → DNAT, port forwarding│
  │              │  Hook: INPUT        │ → Filter incoming      │
  │              │  Hook: FORWARD      │ → Filter forwarded     │
  │              │  Hook: OUTPUT       │ → Filter outgoing      │
  │              │  Hook: POSTROUTING  │ → SNAT, masquerading   │
  │              └─────────────────────┘                        │
  └─────────────────────────────────────────────────────────────┘

THE FIVE NETFILTER HOOKS — PACKET FLOW:

  Incoming Packet                              Outgoing Packet
  from Network                                 to Network
       │                                            ▲
       ▼                                            │
  ┌──────────┐                                ┌──────────┐
  │PREROUTING│  NAT (DNAT) happens here       │POSTROUTING│  SNAT/MASQ
  └────┬─────┘                                └─────▲────┘
       │                                            │
       ▼                                            │
  ┌──────────┐     Is packet for     ┌──────────┐  │
  │ Routing  │───── this host? ─────│ Routing  │──┘
  │ Decision │      NO → FORWARD    │ Decision │
  └────┬─────┘         │            └─────▲────┘
       │ YES           │                  │
       ▼               ▼                  │
  ┌──────────┐    ┌──────────┐       ┌──────────┐
  │  INPUT   │    │ FORWARD  │       │  OUTPUT  │
  │ (to this │    │ (through │       │ (from    │
  │  machine)│    │  machine)│       │  machine)│
  └────┬─────┘    └────┬─────┘       └─────▲────┘
       │               │                    │
       ▼               │              ┌──────────┐
  ┌──────────┐         │              │ Local    │
  │ Local    │         └─────────────│ Process  │
  │ Process  │                       │ (Client) │
  └──────────┘                       └──────────┘
```

---

# 24. iptables — The Classic Workhorse

## 24.1 iptables Fundamentals

```
iptables — THE STANDARD LINUX FIREWALL SINCE 2001:

  Even though nftables is its successor, iptables is still the most
  widely deployed firewall on Linux. Every security professional must know it.

  CORE CONCEPTS:
    Tables  → Categories of rules (filter, nat, mangle, raw)
    Chains  → Lists of rules within each table
    Rules   → Individual instructions (match + action)
    Targets → What to do when matched (ACCEPT, DROP, REJECT, LOG)

THE FOUR TABLES:
  ┌───────────┬──────────────────────────────────────────────────┐
  │  filter   │ DEFAULT table. Allow or block packets.           │
  │           │ 99% of "firewall rules" are here.                │
  │           │ Chains: INPUT, FORWARD, OUTPUT                    │
  ├───────────┼──────────────────────────────────────────────────┤
  │  nat      │ Network Address Translation.                     │
  │           │ Port forwarding, masquerading.                    │
  │           │ Chains: PREROUTING, OUTPUT, POSTROUTING           │
  ├───────────┼──────────────────────────────────────────────────┤
  │  mangle   │ Specialized packet alteration (TTL, TOS, MARK). │
  │           │ Used for QoS. Rarely used in basic firewalling.  │
  │           │ Chains: All five hooks                            │
  ├───────────┼──────────────────────────────────────────────────┤
  │  raw      │ Marks packets to bypass connection tracking.     │
  │           │ Performance optimization. Advanced use.           │
  │           │ Chains: PREROUTING, OUTPUT                        │
  └───────────┴──────────────────────────────────────────────────┘
```

## 24.2 iptables Command Syntax

```
GENERAL SYNTAX:
  iptables -t <table> <command> <chain> <match> -j <target>
  # If -t omitted → "filter" table is used by default

COMMAND FLAGS:
  ┌──────┬────────────────────────┬───────────────────────────────────┐
  │ Flag │ Meaning                │ Example                            │
  ├──────┼────────────────────────┼───────────────────────────────────┤
  │ -A   │ Append rule to chain   │ iptables -A INPUT -p tcp --dport  │
  │      │                        │   22 -j ACCEPT                     │
  ├──────┼────────────────────────┼───────────────────────────────────┤
  │ -I   │ Insert rule at top     │ iptables -I INPUT 1 -p tcp --dport│
  │      │                        │   22 -j ACCEPT                     │
  ├──────┼────────────────────────┼───────────────────────────────────┤
  │ -D   │ Delete a rule          │ iptables -D INPUT -p tcp --dport  │
  │      │                        │   22 -j ACCEPT                     │
  ├──────┼────────────────────────┼───────────────────────────────────┤
  │ -L   │ List rules             │ iptables -L INPUT -n -v           │
  ├──────┼────────────────────────┼───────────────────────────────────┤
  │ -F   │ Flush (delete all)     │ iptables -F INPUT                 │
  ├──────┼────────────────────────┼───────────────────────────────────┤
  │ -P   │ Set default policy     │ iptables -P INPUT DROP            │
  ├──────┼────────────────────────┼───────────────────────────────────┤
  │ -N   │ Create new chain       │ iptables -N MY_CHAIN              │
  ├──────┼────────────────────────┼───────────────────────────────────┤
  │ -X   │ Delete custom chain    │ iptables -X MY_CHAIN              │
  └──────┴────────────────────────┴───────────────────────────────────┘
```

## 24.3 Complete iptables Ruleset — Production Web Server

```bash
#!/bin/bash
# ===========================================================
# iptables Firewall Script — Production Web Server
# ===========================================================
# Configures a complete firewall for a Linux server running
# SSH (22), HTTP (80), and HTTPS (443).
# Default policy: DROP everything, then whitelist services.
# ===========================================================

# ---- STEP 1: Flush all existing rules (clean slate) ----
iptables -F           # Flush filter table
iptables -t nat -F    # Flush nat table
iptables -t mangle -F # Flush mangle table
iptables -X           # Delete custom chains

# ---- STEP 2: Set default policies (DENY ALL) ----
iptables -P INPUT DROP      # Block all incoming by default
iptables -P FORWARD DROP    # Block all forwarded by default
iptables -P OUTPUT ACCEPT   # Allow all outgoing

# ---- STEP 3: Allow loopback interface (localhost) ----
iptables -A INPUT -i lo -j ACCEPT
iptables -A OUTPUT -o lo -j ACCEPT

# ---- STEP 4: Allow established and related connections ----
# This is the STATEFUL part — allows responses to our requests
iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# ---- STEP 5: Allow SSH (port 22) from management network ----
iptables -A INPUT -p tcp --dport 22 -s 10.0.0.0/24 -j ACCEPT

# ---- STEP 6: Allow HTTP (port 80) from anywhere ----
iptables -A INPUT -p tcp --dport 80 -j ACCEPT

# ---- STEP 7: Allow HTTPS (port 443) from anywhere ----
iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# ---- STEP 8: Allow ICMP (ping) — rate limited ----
iptables -A INPUT -p icmp --icmp-type echo-request \
  -m limit --limit 1/s --limit-burst 4 -j ACCEPT

# ---- STEP 9: Drop invalid packets ----
iptables -A INPUT -m conntrack --ctstate INVALID -j DROP

# ---- STEP 10: Log dropped packets ----
iptables -A INPUT -j LOG --log-prefix "IPTABLES-DROPPED: " --log-level 4

# ---- STEP 11: Explicitly drop everything else ----
iptables -A INPUT -j DROP

echo "Firewall rules applied successfully!"
iptables -L INPUT -n -v --line-numbers
```

## 24.4 Common iptables Rule Examples

```bash
# ---- Block a specific IP address ----
iptables -A INPUT -s 45.33.32.156 -j DROP

# ---- Block an entire subnet ----
iptables -A INPUT -s 10.10.10.0/24 -j DROP

# ---- Allow DNS (TCP and UDP port 53) ----
iptables -A INPUT -p udp --dport 53 -j ACCEPT
iptables -A INPUT -p tcp --dport 53 -j ACCEPT

# ---- Allow MySQL only from web server ----
iptables -A INPUT -p tcp --dport 3306 -s 10.0.0.5 -j ACCEPT

# ---- Rate limit SSH (brute-force protection) ----
iptables -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW \
  -m recent --set --name SSH
iptables -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW \
  -m recent --update --seconds 60 --hitcount 4 --name SSH -j DROP

# ---- Allow port range ----
iptables -A INPUT -p tcp --dport 8000:8100 -j ACCEPT

# ---- Multiple ports with multiport ----
iptables -A INPUT -p tcp -m multiport --dports 80,443,8080,8443 -j ACCEPT

# ---- Block outbound to known C2 server ----
iptables -A OUTPUT -d 198.51.100.66 -j DROP

# ---- View current rules ----
iptables -L -v -n --line-numbers

# ---- Save rules (survive reboot) ----
iptables-save > /etc/iptables/rules.v4  # Debian/Ubuntu
# OR
service iptables save                    # CentOS/RHEL
```

---

# 25. UFW — Uncomplicated Firewall Deep Dive

```
UFW (Uncomplicated Firewall):

  UFW is a USER-FRIENDLY interface for managing iptables.
  Its goal: simplify firewall configuration for beginners
  while still being powerful enough for production use.
  
  UFW is a FRONT-END to iptables — all UFW commands generate
  iptables rules behind the scenes.

ESSENTIAL UFW COMMANDS:

  # Enable/Disable UFW
  sudo ufw enable          # Turn on the firewall
  sudo ufw disable         # Turn off the firewall
  
  # Check status
  sudo ufw status          # Show active rules
  sudo ufw status verbose  # Show detailed status
  sudo ufw status numbered # Show rules with line numbers
  
  # Default policies
  sudo ufw default deny incoming   # Block all incoming (ALWAYS DO THIS)
  sudo ufw default allow outgoing  # Allow all outgoing
  
  # Allow specific services
  sudo ufw allow ssh           # Allow SSH (port 22)
  sudo ufw allow http          # Allow HTTP (port 80)
  sudo ufw allow https         # Allow HTTPS (port 443)
  sudo ufw allow 3306/tcp      # Allow MySQL over TCP
  
  # Allow from specific source
  sudo ufw allow from 192.168.1.0/24 to any port 22 proto tcp
  # Allow SSH only from the 192.168.1.0/24 subnet
  
  # Deny specific traffic
  sudo ufw deny from 45.33.32.156  # Block attacker IP
  
  # Delete a rule
  sudo ufw delete allow http       # Remove HTTP rule
  sudo ufw delete 3                # Delete rule number 3
  
  # Logging
  sudo ufw logging on high         # Enable high-level logging
  # Log levels: low, medium, high, full
  # Check logs: sudo less /var/log/ufw.log
  
  # Reset to defaults
  sudo ufw reset                   # Delete ALL rules, disable UFW
```

---

# 26. Windows Firewall — Defender Firewall with Advanced Security

```
WINDOWS DEFENDER FIREWALL:

  Built-in host-based firewall in all modern Windows systems.
  Supports inbound/outbound rules, profiles, and IPsec.
  
  THREE NETWORK PROFILES:
    Domain  — Computer connected to Active Directory domain
    Private — Home or office network (trusted)
    Public  — Coffee shop, hotel, airport (untrusted)
    
  Each profile can have different firewall rules and settings.

WINDOWS FIREWALL — COMMAND LINE MANAGEMENT:

  # ===== Using netsh (legacy) =====
  
  # View all firewall rules
  netsh advfirewall firewall show rule name=all
  
  # Enable firewall for all profiles
  netsh advfirewall set allprofiles state on
  
  # Block inbound by default
  netsh advfirewall set allprofiles firewallpolicy blockinbound,allowoutbound
  
  # Allow SSH (port 22) inbound
  netsh advfirewall firewall add rule name="Allow SSH" ^
    dir=in action=allow protocol=TCP localport=22
  
  # Block specific IP
  netsh advfirewall firewall add rule name="Block Attacker" ^
    dir=in action=block remoteip=45.33.32.156
  
  # Delete a rule
  netsh advfirewall firewall delete rule name="Allow SSH"

  # ===== Using PowerShell (modern) =====
  
  # View firewall rules
  Get-NetFirewallRule | Where-Object {$_.Enabled -eq 'True'} |
    Format-Table Name, Direction, Action, Profile
  
  # Allow HTTP inbound
  New-NetFirewallRule -DisplayName "Allow HTTP" `
    -Direction Inbound -Action Allow `
    -Protocol TCP -LocalPort 80
  
  # Allow HTTPS inbound
  New-NetFirewallRule -DisplayName "Allow HTTPS" `
    -Direction Inbound -Action Allow `
    -Protocol TCP -LocalPort 443
  
  # Block outbound to suspicious IP
  New-NetFirewallRule -DisplayName "Block C2 Server" `
    -Direction Outbound -Action Block `
    -RemoteAddress 198.51.100.66
    
  # Remove a rule
  Remove-NetFirewallRule -DisplayName "Allow HTTP"
  
  # Disable SMBv1 (CRITICAL SECURITY ACTION)
  Disable-WindowsOptionalFeature -Online -FeatureName smb1protocol
```

---

# PART IV — NETWORK SECURITY TOOLS & MONITORING

---

# 27. tcpdump — Command-Line Packet Analysis

```
tcpdump — THE ESSENTIAL PACKET ANALYSIS TOOL:

  tcpdump is a powerful command-line packet analyzer for network
  troubleshooting, analysis, and security monitoring.
  
  It captures and displays the content of packets being transmitted
  or received over a network interface.

KEY FEATURES:
  ✓ Packet Capture — Capture live traffic from any interface
  ✓ Filtering — BPF (Berkeley Packet Filter) syntax for precise filtering
  ✓ Real-time Analysis — View packets as they arrive
  ✓ Save to File — Write captures to .pcap files for later analysis
  ✓ Versatility — Works on virtually any Unix/Linux system

ESSENTIAL tcpdump COMMANDS:

  # Capture all traffic on default interface
  sudo tcpdump
  
  # Capture on specific interface
  sudo tcpdump -i eth0
  
  # Capture only TCP traffic
  sudo tcpdump tcp
  
  # Capture only UDP traffic
  sudo tcpdump udp
  
  # Capture traffic on specific port
  sudo tcpdump port 80         # HTTP traffic
  sudo tcpdump port 443        # HTTPS traffic
  sudo tcpdump port 22         # SSH traffic
  sudo tcpdump port 53         # DNS traffic
  
  # Capture traffic from/to specific host
  sudo tcpdump host 192.168.1.1
  sudo tcpdump src host 192.168.1.5    # Only FROM this host
  sudo tcpdump dst host 10.0.0.1       # Only TO this host
  
  # Capture traffic from specific subnet
  sudo tcpdump net 192.168.1.0/24
  
  # Combine filters with AND, OR, NOT
  sudo tcpdump src host 192.168.1.5 and dst port 80
  sudo tcpdump port 80 or port 443
  sudo tcpdump not port 22            # Everything except SSH
  
  # Save capture to file
  sudo tcpdump -w capture.pcap
  
  # Read from saved file
  sudo tcpdump -r capture.pcap
  
  # Limit number of packets captured
  sudo tcpdump -c 100          # Capture only 100 packets
  
  # Verbose output (more detail)
  sudo tcpdump -v              # Verbose
  sudo tcpdump -vv             # More verbose
  sudo tcpdump -vvv            # Maximum verbosity
  
  # Show packet contents in hex and ASCII
  sudo tcpdump -X              # Hex + ASCII
  sudo tcpdump -XX             # Include link-layer header
  
  # Don't resolve hostnames (faster output)
  sudo tcpdump -n              # Numeric IP addresses
  sudo tcpdump -nn             # Numeric IPs AND port numbers

SECURITY USE CASES:

  # Detect ARP spoofing — look for duplicate MAC addresses
  sudo tcpdump -n arp
  
  # Monitor DNS queries — detect DNS tunneling
  sudo tcpdump -n port 53
  
  # Capture credentials on insecure protocols
  sudo tcpdump -A port 21      # FTP passwords (plaintext!)
  sudo tcpdump -A port 23      # Telnet sessions
  sudo tcpdump -A port 80      # HTTP form submissions
  
  # Detect SYN flood attacks
  sudo tcpdump 'tcp[tcpflags] & tcp-syn != 0'
  
  # Monitor ICMP (detect ping sweeps/floods)
  sudo tcpdump icmp
```

---

# 28. Wireshark — GUI Packet Analysis

```
WIRESHARK — THE WORLD'S MOST POPULAR NETWORK ANALYZER:

  "Where every packet tells a tale, and every byte has a
  secret to share. It's the gossip column of the networking world!"
  
  Wireshark provides a graphical interface for deep packet analysis
  with protocol dissectors for hundreds of protocols.

WIRESHARK DISPLAY FILTERS (Most Used):

  # Filter by IP address
  ip.addr == 192.168.1.1        # Traffic to OR from this IP
  ip.src == 10.0.0.2            # Traffic FROM this IP only
  ip.dst == 172.16.0.1          # Traffic TO this IP only
  ip.addr == 192.168.1.0/24     # Entire subnet
  
  # Filter by protocol
  tcp                            # All TCP traffic
  udp                            # All UDP traffic
  http                           # HTTP traffic
  dns                            # DNS traffic
  arp                            # ARP traffic
  icmp                           # ICMP (ping) traffic
  
  # Filter by port
  tcp.port == 443                # HTTPS traffic
  tcp.srcport == 80              # From port 80
  tcp.dstport == 80              # To port 80
  
  # Filter by TCP flags
  tcp.flags.syn == 1             # SYN packets (connection start)
  tcp.flags.rst == 1             # RST packets (connection reset)
  
  # Combine filters
  ip.addr == 192.168.1.1 && ip.addr == 10.0.0.2
  ip.src == 10.0.0.1 && tcp.dstport == 8080
  
  # Filter by MAC address
  eth.addr == 00:11:22:33:44:55
  
  # Filter by frame length
  frame.len == 128
  
  # Filter by VLAN
  vlan.id == 100
  
  # Search for string in payload
  frame contains "OpenAI"
  frame contains "password"

WIRESHARK FOR SECURITY:

  Detecting ARP Spoofing:
    Filter: arp
    Look for: Multiple ARP replies with DIFFERENT MAC addresses
    for the same IP. This indicates ARP poisoning.
    
  Detecting SYN Floods:
    Filter: tcp.flags.syn == 1 && tcp.flags.ack == 0
    Look for: Massive volume of SYN-only packets from many source IPs.
    
  Capturing VoIP Calls:
    Telephony → VoIP Calls → Select call → Play Streams
    If SIP/RTP is unencrypted, you can literally LISTEN to calls.
    
  Extracting Files from HTTP Traffic:
    File → Export Objects → HTTP → Save selected files
    Can recover downloaded files, images, documents.
    
  Following TCP Streams:
    Right-click a packet → Follow → TCP Stream
    Shows the entire conversation in readable format.
    Great for viewing HTTP requests/responses, email content.
```

---

# PART V — WIRELESS SECURITY

---

# 29. Wireless Standards — 802.11 Family

```
IEEE 802.11 — THE WIRELESS LAN STANDARD:

  ┌──────────┬──────────┬──────────┬──────────┬─────────────────┐
  │ Standard │ Year     │ Freq     │ Max Speed│ Notes            │
  ├──────────┼──────────┼──────────┼──────────┼─────────────────┤
  │ 802.11a  │ 1999     │ 5 GHz   │ 54 Mbps  │ Short range      │
  │ 802.11b  │ 1999     │ 2.4 GHz │ 11 Mbps  │ Long range, slow │
  │ 802.11g  │ 2003     │ 2.4 GHz │ 54 Mbps  │ Backward compat  │
  │ 802.11n  │ 2009     │ Both    │ 600 Mbps │ MIMO technology   │
  │ (Wi-Fi 4)│          │         │          │                   │
  │ 802.11ac │ 2013     │ 5 GHz   │ 6.9 Gbps │ Wave 2, MU-MIMO  │
  │ (Wi-Fi 5)│          │         │          │                   │
  │ 802.11ax │ 2019     │ Both    │ 9.6 Gbps │ OFDMA, WPA3      │
  │ (Wi-Fi 6)│          │ + 6 GHz │          │                   │
  └──────────┴──────────┴──────────┴──────────┴─────────────────┘
  
  2.4 GHz vs. 5 GHz:
    2.4 GHz: Longer range, better wall penetration, more interference
    5 GHz:   Shorter range, less interference, higher speeds
    6 GHz:   New spectrum (Wi-Fi 6E), very high speed, minimal interference
```

---

# 30. Wireless Encryption — WEP, WPA, WPA2, WPA3

```
WIRELESS ENCRYPTION EVOLUTION:

  WEP (Wired Equivalent Privacy) — 1997
    ❌ BROKEN. COMPLETELY INSECURE. Do NOT use.
    ❌ Uses RC4 stream cipher with static keys
    ❌ Can be cracked in MINUTES using aircrack-ng
    ❌ IV (Initialization Vector) is only 24 bits → repeats quickly
    ❌ No per-packet key rotation
    
  WPA (Wi-Fi Protected Access) — 2003
    ⚠️ Interim fix for WEP. Still uses RC4 but adds TKIP.
    ⚠️ TKIP provides per-packet keys and message integrity (MIC)
    ⚠️ Better than WEP but still has vulnerabilities
    ⚠️ Deprecated — do not use in production
    
  WPA2 (Wi-Fi Protected Access 2) — 2004
    ✅ CURRENT STANDARD for most networks
    ✅ Uses AES-CCMP encryption (strong)
    ✅ Two modes:
       WPA2-Personal (PSK): Pre-shared key (password)
       WPA2-Enterprise (802.1X): RADIUS authentication
    ⚠️ Vulnerable to KRACK attack (Key Reinstallation Attack)
    ⚠️ PSK mode vulnerable to offline dictionary attacks
       if passphrase is weak
    
  WPA3 (Wi-Fi Protected Access 3) — 2018
    ✅ LATEST STANDARD — strongest wireless security
    ✅ Uses SAE (Simultaneous Authentication of Equals)
       Replaces PSK 4-way handshake with Dragonfly protocol
    ✅ Forward secrecy — even if password is compromised later,
       previously captured traffic CANNOT be decrypted
    ✅ Protected Management Frames (mandatory)
    ✅ 192-bit security suite for WPA3-Enterprise
    ✅ Resistant to offline dictionary attacks
    ✅ Individualized data encryption — each client gets unique key

COMPARISON TABLE:
  ┌───────────┬────────────┬────────────┬───────────┬───────────┐
  │ Feature   │ WEP        │ WPA        │ WPA2      │ WPA3      │
  ├───────────┼────────────┼────────────┼───────────┼───────────┤
  │ Encryption│ RC4        │ RC4+TKIP   │ AES-CCMP  │ AES-GCMP  │
  │ Key Mgmt  │ Static     │ TKIP rekeying│ 4-way HS │ SAE/Dragon│
  │ Security  │ ❌ Broken  │ ⚠️ Weak    │ ✅ Strong │ ✅ Strongest│
  │ Crack Time│ Minutes    │ Hours      │ Days-Never│ Not feasible│
  │ Status    │ Obsolete   │ Deprecated │ Current   │ Future std │
  └───────────┴────────────┴────────────┴───────────┴───────────┘
```

---

# 31. Wireless Attacks and Defenses

```
COMMON WIRELESS ATTACKS:

  1. DEAUTHENTICATION ATTACK:
     Send forged deauth frames to disconnect clients from AP.
     Used to force reconnection for WPA handshake capture.
     Tool: aireplay-ng -0 10 -a <AP_MAC> -c <CLIENT_MAC> wlan0mon
     
  2. EVIL TWIN ATTACK:
     Create a fake AP with the same SSID as a legitimate network.
     Victims connect to the fake AP. Attacker gets MitM position.
     Tool: hostapd, Fluxion, WiFi-Pumpkin
     
  3. WPA/WPA2 HANDSHAKE CAPTURE + DICTIONARY ATTACK:
     Capture the 4-way handshake, then crack offline with wordlist.
     Tools: airodump-ng (capture), aircrack-ng / hashcat (crack)
     
  4. KARMA/MANA ATTACK:
     Exploit the auto-connect behavior of devices by responding to
     ANY probe request with a matching SSID.
     
  5. WARDRIVING:
     Driving around with a laptop scanning for wireless networks.
     Maps SSIDs, encryption types, signal strength, locations.

WIRELESS SECURITY BEST PRACTICES:
  ✅ Use WPA3 (or WPA2-Enterprise at minimum)
  ✅ Strong passphrase: 20+ characters, random
  ✅ Disable WPS (Wi-Fi Protected Setup) — easily brute-forced
  ✅ Use 802.1X (RADIUS) for enterprise environments
  ✅ Segment wireless traffic on separate VLAN
  ✅ Implement WIDS/WIPS (Wireless IDS/IPS) to detect rogues
  ✅ Disable SSID broadcast only as minimal obscurity (not real security)
  ✅ MAC filtering provides minimal security (easily spoofed)
  ✅ Regularly audit connected devices
  ✅ Use VPN over public Wi-Fi
```

---

# PART VI — PROTOCOL EXPLOITS & ATTACK TECHNIQUES

---

# 32. EternalBlue — SMBv1 Exploitation

```
ETERNALBLUE (CVE-2017-0144):

  The most significant exploit of the decade. Developed by the NSA.
  Leaked by the Shadow Brokers hacker group.
  Used in WannaCry and NotPetya ransomware — caused BILLIONS in damage.

THE VULNERABILITY:
  
  Targets Microsoft's implementation of SMBv1 protocol.
  
  Three separate bugs exploited together:
  
  1. MEMORY CORRUPTION (CVE-2017-0144):
     SrvOs2FeaListToNt function miscalculates memory size when
     parsing a TRANS2 request with a crafted FEA list.
     This causes a BUFFER OVERFLOW in kernel memory.
     
  2. TYPE CONFUSION:
     Sends primary request as NT_TRANSACT but sends secondary
     fragments as TRANS2_SECONDARY. Server fails to validate the
     type — allows assembling a massive malicious packet.
     
  3. ARBITRARY CODE EXECUTION:
     By carefully manipulating the overflow, attacker overwrites
     critical kernel memory structures and redirects execution
     to shellcode running with SYSTEM privileges.

ATTACK MECHANICS:
  1. SCAN: Attacker scans for machines with port 445 (SMB) open
  2. EXPLOIT: Send crafted SMBv1 packet to trigger buffer overflow
  3. PAYLOAD: Shellcode executes — installs ransomware/backdoor
  4. PROPAGATION: Infected machine scans and attacks other vulnerable
     machines on the same network (WORMABLE capability)

IMPACT:
  ❌ Remote Code Execution — Unauthenticated
  ❌ Wormable — Self-propagating across networks
  ❌ Full System Compromise — Runs in kernel (SYSTEM privileges)
  ❌ WannaCry: 200,000+ computers in 150 countries in ONE DAY
  ❌ NotPetya: $10+ BILLION in damage worldwide

DEFENSES:
  ✅ PATCH (MS17-010) — Microsoft released the fix in March 2017
     This is the SINGLE most effective defense.
  ✅ DISABLE SMBv1 — Obsolete, insecure protocol.
     PowerShell: Disable-WindowsOptionalFeature -Online -FeatureName smb1protocol
     Group Policy: Configure "SMBv1 Server" and "SMBv1 Client" to disabled
  ✅ BLOCK PORT 445 — At perimeter firewall. Block external SMB.
  ✅ NETWORK SEGMENTATION — Limit lateral movement.
```

---

# PART VII — FIREWALL ARCHITECTURE & DESIGN

---

# 33. DMZ Architecture — Multi-Zone Design

```
DMZ (Demilitarized Zone) — THE SECURITY BUFFER ZONE:

  A DMZ is a network segment that sits BETWEEN the trusted internal
  network and the untrusted external network (Internet).
  
  Purpose: Host PUBLIC-FACING servers (web, email, DNS) in a zone
  that is more exposed than internal but still protected by firewalls.

SINGLE FIREWALL DMZ (Three-Legged):

  The firewall has THREE interfaces:
  
                        THE INTERNET
                             │
                             ▼
                    ┌─────────────────┐
                    │    FIREWALL      │
                    │  (3 interfaces)  │
                    └─┬─────────┬───┘
                      │         │
           ┌──────────┘         └──────────┐
           ▼                               ▼
    ┌──────────────┐              ┌──────────────┐
    │     DMZ       │              │   INTERNAL    │
    │ (Semi-Trusted)│              │  (Trusted)    │
    │               │              │               │
    │ Web Server    │              │ Workstations  │
    │ Email Server  │              │ File Server   │
    │ DNS (public)  │              │ Database      │
    └──────────────┘              └──────────────┘
    
  Firewall Rules:
    Internet → DMZ:     Allow HTTP/HTTPS/SMTP to DMZ servers
    Internet → Internal: DENY ALL (never allow direct access)
    DMZ → Internal:      Allow only SPECIFIC, NECESSARY connections
                          (e.g., web server → database on port 3306)
    Internal → DMZ:      Allow administrative access (SSH, RDP)
    Internal → Internet: Allow outbound (NAT/PAT)
    DMZ → Internet:      Limited (only what's needed for updates)

DUAL FIREWALL DMZ (Maximum Security):

                        THE INTERNET
                             │
                             ▼
                    ┌─────────────────┐
                    │  OUTER FIREWALL  │  ← Vendor A (e.g., Palo Alto)
                    └────────┬────────┘
                             │
                    ┌────────┴────────┐
                    │       DMZ        │
                    │  Web, Email, DNS │
                    └────────┬────────┘
                             │
                    ┌────────┴────────┐
                    │  INNER FIREWALL  │  ← Vendor B (e.g., Fortinet)
                    └────────┬────────┘
                             │
                    ┌────────┴────────┐
                    │    INTERNAL      │
                    │  (Trusted Zone)  │
                    └─────────────────┘
                    
  Using different vendors is intentional:
    If a vulnerability is found in Vendor A's firewall, the attacker
    still must bypass Vendor B's completely different firewall.
    This is Defense in Depth applied to firewall architecture.
```

---

# 34. Firewall Policy Management and Best Practices

```
FIREWALL POLICY MANAGEMENT — THE RULES OF RULES:

  1. DEFAULT DENY ALWAYS
     Start with "deny all" and whitelist only needed traffic.
     
  2. PRINCIPLE OF LEAST PRIVILEGE
     Allow ONLY the minimum traffic needed for business functions.
     
  3. SPECIFIC BEFORE GENERAL
     Place narrow, specific rules before broad rules.
     Remember: FIRST MATCH WINS.
     
  4. LOG EVERYTHING
     Log all denied traffic. Log accepted traffic for sensitive zones.
     Feed logs to SIEM for correlation and alerting.
     
  5. REVIEW REGULARLY
     Quarterly firewall rule reviews. Remove unused/stale rules.
     Every rule should have a documented business justification.
     
  6. DOCUMENT EVERY RULE
     Who requested it? Why? When does it expire?
     
  7. CHANGE MANAGEMENT
     All firewall changes go through formal change control.
     Test in staging before production. Have rollback plans.
     
  8. SERVICE ACCOUNTS, NOT USER ACCOUNTS
     Firewall admin access via named accounts with MFA.
     Audit admin actions.
     
  9. SEPARATE MANAGEMENT PLANE
     Management interface on dedicated, isolated network.
     Never expose firewall management to the Internet.
     
  10. BACKUP CONFIGURATIONS
      Regular automated backups of firewall configs.
      Test restoration periodically.
```

---

# PART VIII — AUDITING, TROUBLESHOOTING & MASTERY

---

# 35. Firewall Auditing — Finding Weaknesses

```
FIREWALL AUDITING — WHAT TO CHECK:

  ┌────────────────────────────────────────────────────────────────┐
  │ AUDITING CHECKLIST:                                           │
  │                                                                │
  │ ✓ DEFAULT POLICY: Is it DROP/DENY? If ALLOW → CRITICAL RISK  │
  │ ✓ ANY-ANY RULES: Are there rules allowing "any" to "any"?    │
  │   Each one is a potential bypass.                              │
  │ ✓ OVERLY BROAD RULES: Rules allowing entire subnets when     │
  │   only specific hosts need access.                             │
  │ ✓ RULE ORDER: Are specific rules before general rules?       │
  │ ✓ STALE RULES: Rules for decommissioned servers/services?    │
  │ ✓ UNDOCUMENTED RULES: Rules with no business justification?  │
  │ ✓ LOGGING: Is logging enabled for denied traffic?            │
  │ ✓ MANAGEMENT ACCESS: Is management on isolated network?      │
  │ ✓ FIRMWARE: Is firmware up to date? Known CVEs?              │
  │ ✓ AUTHENTICATION: MFA for admin access? Strong passwords?    │
  │ ✓ OUTBOUND RULES: Are they restrictive or wide open?         │
  │ ✓ INBOUND SSH/RDP: Is remote access limited to specific IPs? │
  │ ✓ SEGMENTATION: Proper zones? DMZ correctly configured?      │
  │ ✓ VPN: Split vs. full tunnel? MFA for VPN access?            │
  └────────────────────────────────────────────────────────────────┘
```

---

# 36. Firewall Troubleshooting — Diagnosing Issues

```
SYSTEMATIC FIREWALL TROUBLESHOOTING:

  STEP 1: Verify the problem is actually the firewall
    - Can you ping the destination? traceroute?
    - Is the service actually running on the target?
    - Is it a DNS issue? Try using IP directly.
    
  STEP 2: Check firewall rules
    iptables -L -v -n --line-numbers   # Linux
    netsh advfirewall firewall show rule name=all  # Windows
    
  STEP 3: Check the logs
    grep "DROPPED" /var/log/syslog      # Linux
    grep "IPTABLES-DROPPED" /var/log/kern.log
    # Look for the specific packet being blocked
    
  STEP 4: Test with temporary rule
    # Temporarily allow traffic to confirm it's the firewall
    iptables -I INPUT 1 -p tcp --dport <port> -j ACCEPT
    # If traffic works now → firewall was blocking it
    # REMOVE the temporary rule and add proper one
    
  STEP 5: Verify routing
    ip route show                        # Linux
    route print                          # Windows
    # Is traffic going through the expected path?
    
  STEP 6: Verify NAT (if applicable)
    iptables -t nat -L -v -n            # Check NAT rules
    # Is DNAT/SNAT configured correctly?

COMMON TROUBLESHOOTING COMMANDS:

  # Test connectivity
  ping <ip>                # ICMP connectivity
  traceroute <ip>          # Path to destination (Linux)
  tracert <ip>             # Path to destination (Windows)
  
  # Test specific port
  telnet <ip> <port>       # TCP port test
  nc -zv <ip> <port>       # Netcat port test (Linux)
  Test-NetConnection -ComputerName <ip> -Port <port>  # PowerShell
  
  # Capture traffic at firewall
  sudo tcpdump -i eth0 host <ip> and port <port>
```

---

# 37. Common Mistakes — What Gets Networks Compromised

```
THE TOP FIREWALL MISCONFIGURATIONS:

  1. DEFAULT ALLOW POLICY
     ❌ Everything allowed unless explicitly denied
     ✅ Everything denied unless explicitly allowed
     
  2. ANY-ANY-ANY RULES
     ❌ iptables -A INPUT -j ACCEPT
     ✅ iptables -A INPUT -p tcp --dport 443 -j ACCEPT
     
  3. NOT FILTERING OUTBOUND TRAFFIC
     ❌ All outbound traffic allowed (malware can phone home)
     ✅ Whitelist necessary outbound ports (80, 443, 53)
     
  4. FORGETTING TO LOG
     ❌ No logging — blind to attacks
     ✅ Log all denied traffic, alert on anomalies
     
  5. USING SMBv1
     ❌ Port 445 open with SMBv1 enabled → EternalBlue target
     ✅ Disable SMBv1, use SMBv3, block 445 at perimeter
     
  6. MANAGEMENT ON PUBLIC INTERFACE
     ❌ Firewall admin panel accessible from Internet
     ✅ Management on isolated network, MFA required
     
  7. NOT UPDATING FIRMWARE
     ❌ Known CVEs in firewall firmware → attacker bypasses firewall
     ✅ Patch firewall firmware as urgently as any server
     
  8. TRUSTING THE INTERNAL NETWORK
     ❌ "Castle and moat" — trusted inside, untrusted outside
     ✅ Zero Trust — verify everything regardless of location
     
  9. NO RULE REVIEW PROCESS
     ❌ Rules accumulate over years, never cleaned up
     ✅ Quarterly reviews, expiration dates on temporary rules
     
  10. SINGLE POINT OF FAILURE
      ❌ One firewall, no redundancy → failure = no security
      ✅ HA (High Availability) pairs, failover configurations
```

---

# 38. Hands-On Labs

```
LAB 1: Build a Complete iptables Firewall
  Objective: Configure a Linux server with a production-ready firewall
  Steps:
    1. Start with a fresh Linux VM (Ubuntu or CentOS)
    2. Write the iptables script from Section 24.3
    3. Test each rule with netcat and nmap from another machine
    4. Verify logging with: tail -f /var/log/syslog | grep IPTABLES
    5. Practice saving and restoring rules
  
LAB 2: UFW for Quick Deployment
  Objective: Deploy a firewall in 5 minutes using UFW
  Steps:
    1. sudo ufw default deny incoming
    2. sudo ufw default allow outgoing
    3. sudo ufw allow ssh
    4. sudo ufw allow http
    5. sudo ufw allow https
    6. sudo ufw enable
    7. sudo ufw status verbose
    
LAB 3: Windows Firewall Hardening
  Objective: Harden Windows using PowerShell firewall commands
  Steps:
    1. Open PowerShell as Administrator
    2. Create rules to allow only HTTP, HTTPS, and RDP
    3. Block all SMBv1 traffic
    4. Test with Test-NetConnection from another machine

LAB 4: ARP Spoofing Detection
  Objective: Detect ARP spoofing using tcpdump and Wireshark
  Steps:
    1. Run tcpdump -n arp on a Linux machine
    2. Observe normal ARP traffic patterns
    3. Use Wireshark to filter: arp && arp.duplicate-address-detected
    4. Identify suspicious duplicate MAC-IP mappings

LAB 5: Network Capture and Analysis
  Objective: Capture and analyze network traffic
  Steps:
    1. Start tcpdump capture: sudo tcpdump -w lab5.pcap -i eth0
    2. Generate some traffic (browse websites, SSH, ping)
    3. Stop capture and open in Wireshark
    4. Apply filters to identify specific protocols
    5. Follow a TCP stream to see complete conversation
    6. Export any HTTP objects found in the capture

LAB 6: Firewall Rule Auditing
  Objective: Audit an existing firewall configuration
  Steps:
    1. Review iptables -L -v -n --line-numbers
    2. Identify overly permissive rules (any-any-any)
    3. Check for stale rules (zero packet counters)
    4. Verify default policy is DROP
    5. Document findings in an audit report
```

---

# 39. Interview Questions — Prove Your Mastery

```
NETWORK SECURITY INTERVIEW QUESTIONS:

FUNDAMENTALS:
  Q: What is the difference between a hub, switch, and router?
  A: Hub = Layer 1, broadcasts to all ports. Switch = Layer 2,
     forwards to specific port by MAC address. Router = Layer 3,
     routes between networks by IP address.

  Q: Explain the OSI model. Which layer does a firewall operate at?
  A: 7 layers from Physical to Application. Traditional firewalls
     operate at Layer 3-4. NGFWs operate at Layer 3-7. WAFs at Layer 7.

  Q: What is the difference between TCP and UDP?
  A: TCP is connection-oriented, reliable, ordered (3-way handshake).
     UDP is connectionless, best-effort, unordered but fast.

FIREWALL:
  Q: What is the difference between stateful and stateless firewalls?
  A: Stateless examines each packet independently. Stateful tracks
     connections in a state table — return traffic for established
     connections is automatically allowed.

  Q: Explain Default Deny vs Default Allow policy.
  A: Default Deny blocks everything not explicitly allowed (secure).
     Default Allow permits everything not explicitly blocked (insecure).
     Every security standard requires Default Deny.

  Q: What is the difference between DROP and REJECT in iptables?
  A: DROP silently discards the packet — attacker gets no response.
     REJECT discards AND sends an error back — confirms something exists.
     DROP is more secure for external traffic.

PROTOCOL ATTACKS:
  Q: How does ARP spoofing work and how do you defend against it?
  A: Attacker sends forged ARP replies to poison ARP caches, redirecting
     traffic through their machine (MitM). Defense: Dynamic ARP Inspection,
     static ARP entries for critical devices, 802.1X, encryption.

  Q: What is EternalBlue and why is it significant?
  A: EternalBlue exploits a vulnerability in SMBv1 (CVE-2017-0144).
     Allows unauthenticated remote code execution with SYSTEM privileges.
     Used in WannaCry/NotPetya. Defense: patch MS17-010, disable SMBv1,
     block port 445 at perimeter.

  Q: What is NTP amplification?
  A: DDoS attack using NTP's monlist command. Small spoofed request
     generates large response (100x amplification). Traffic floods
     the spoofed victim IP. Defense: disable monlist, BCP38, rate limiting.

  Q: How does DHCP starvation work?
  A: Attacker sends many DHCPDISCOVER with spoofed MACs to exhaust
     the IP address pool. New clients can't get IPs → DoS.
     Defense: DHCP snooping, port security.

ARCHITECTURE:
  Q: What is a DMZ and why is it important?
  A: A DMZ is a network segment between the internet and internal network.
     Hosts public-facing servers. If compromised, attacker still can't
     directly access the internal network.

  Q: Explain the difference between NAT and a firewall.
  A: NAT translates IP addresses (private ↔ public). It provides
     obscurity, NOT security. A firewall inspects and filters traffic
     based on rules. NAT is not a security control by itself.

  Q: What is Zero Trust networking?
  A: "Never trust, always verify." Every user, device, and connection
     is verified regardless of network location. No implicit trust
     for internal traffic. Identity is the new perimeter.
```

---

# 40. Comprehensive Network Security Glossary

```
A
──────────────────────────────────────────────────────────────────────
ACL (Access Control List): Set of rules on a network device defining
  what traffic is permitted or denied. Used on routers, switches, firewalls.

ARP (Address Resolution Protocol): Maps IP addresses to MAC addresses
  on a local network. Port: N/A (Layer 2). Vulnerable to spoofing.

B
──────────────────────────────────────────────────────────────────────
BGP (Border Gateway Protocol): Internet routing protocol between
  autonomous systems. Vulnerable to hijacking attacks.

BPF (Berkeley Packet Filter): Filtering language used by tcpdump
  and Wireshark for packet capture and analysis.

C
──────────────────────────────────────────────────────────────────────
CIDR (Classless Inter-Domain Routing): IP addressing method using
  prefix notation (e.g., /24) instead of classful addressing.

D
──────────────────────────────────────────────────────────────────────
DAI (Dynamic ARP Inspection): Switch feature that validates ARP
  packets against DHCP snooping binding table. Best defense against ARP spoofing.

DHCP (Dynamic Host Configuration Protocol): Automatically assigns
  IP addresses and network configuration. Ports: 67/68 (UDP).

DMZ (Demilitarized Zone): Network segment hosting public-facing
  servers, positioned between internet and internal network.

DPI (Deep Packet Inspection): Examining the data portion of packets
  beyond just headers. Used by NGFWs and IPS systems.

F
──────────────────────────────────────────────────────────────────────
Firewall: Network security device that monitors and controls traffic
  based on predefined rules. Types: packet filter, stateful, NGFW, WAF.

G
──────────────────────────────────────────────────────────────────────
Gateway: Device connecting different networks, typically the router
  providing the path to the internet (default gateway).

I
──────────────────────────────────────────────────────────────────────
iptables: Linux command-line firewall tool. Front-end to Netfilter.
  Standard since 2001. Uses tables, chains, rules, and targets.

L
──────────────────────────────────────────────────────────────────────
LDAP (Lightweight Directory Access Protocol): Protocol for accessing
  directory services. Ports: 389/636. Used by Active Directory.

M
──────────────────────────────────────────────────────────────────────
MAC Address: 48-bit hardware address uniquely identifying network
  interfaces. Format: AA:BB:CC:DD:EE:FF.

N
──────────────────────────────────────────────────────────────────────
NAT (Network Address Translation): Translates private IPs to public
  IPs. Not a security control — provides obscurity only.

Netfilter: Linux kernel framework providing packet filtering,
  NAT, and mangling. Backend for iptables, nftables, UFW, firewalld.

NGFW (Next-Generation Firewall): Firewall combining packet filtering,
  DPI, IPS, application awareness, and threat intelligence.

NTP (Network Time Protocol): Synchronizes clocks. Port: 123 (UDP).
  Vulnerable to amplification DDoS attacks via monlist command.

O
──────────────────────────────────────────────────────────────────────
OSI Model: 7-layer networking reference model. Physical, Data Link,
  Network, Transport, Session, Presentation, Application.

P
──────────────────────────────────────────────────────────────────────
PAT (Port Address Translation): Many-to-one NAT using port numbers
  to differentiate connections. What your home router does.

S
──────────────────────────────────────────────────────────────────────
SBC (Session Border Controller): VoIP firewall managing SIP/RTP
  traffic at the network border. Essential for VoIP security.

SIP (Session Initiation Protocol): VoIP signaling protocol.
  Ports: 5060/5061. Text-based, similar to HTTP.

SMB (Server Message Block): Windows file sharing protocol. Port: 445.
  SMBv1 is exploited by EternalBlue — DISABLE immediately.

SNMP (Simple Network Management Protocol): Network device monitoring.
  Ports: 161/162. v1/v2c are insecure — always use SNMPv3.

T
──────────────────────────────────────────────────────────────────────
tcpdump: Command-line packet analyzer for Unix/Linux systems.
  Essential tool for network troubleshooting and security analysis.

U
──────────────────────────────────────────────────────────────────────
UFW (Uncomplicated Firewall): User-friendly iptables front-end
  for Ubuntu/Debian systems. Simplifies firewall configuration.

V
──────────────────────────────────────────────────────────────────────
VLAN (Virtual LAN): Logical network segmentation on switches.
  Reduces broadcast domains and improves security.

VPN (Virtual Private Network): Encrypted tunnel over public network.
  Types: Site-to-site, Remote access. Protocols: IPsec, WireGuard, OpenVPN.

W
──────────────────────────────────────────────────────────────────────
WAF (Web Application Firewall): Layer 7 firewall protecting web
  applications from OWASP Top 10 attacks (SQLi, XSS, etc.).

WPA3 (Wi-Fi Protected Access 3): Latest wireless encryption standard.
  Uses SAE/Dragonfly protocol. Forward secrecy. Resistant to offline attacks.

Wireshark: GUI-based packet analyzer. Supports hundreds of protocols.
  Essential tool for security analysis and network forensics.
```

---

# 41. Final Consolidated Review — The Big Picture

```
THE COMPLETE NETWORK SECURITY PICTURE:

     ┌──────────────────────────────────────────────────────────┐
     │                   THREAT LANDSCAPE                       │
     │   Nation-States  │ Criminals │ Insiders │ Hacktivists    │
     └──────────────────────────────────────────────────────────┘
                                   │
                                   │ Attack via:
                                   ▼
     ┌──────────────────────────────────────────────────────────┐
     │              NETWORK ATTACK VECTORS                      │
     │  ARP Spoofing │ DHCP Attacks │ SMB Exploits │ NTP Amp   │
     │  DNS Hijack   │ VoIP Attacks │ Wireless Attacks │ MitM  │
     └──────────────────────────────────────────────────────────┘
                                   │
                          Targeting:
                                   ▼
     ┌──────────────────────────────────────────────────────────┐
     │                   YOUR NETWORK                           │
     │  Servers │ Workstations │ IoT │ VoIP │ Wireless │ Cloud │
     └──────────────────────────────────────────────────────────┘
                                   │
                         Protected by:
                                   ▼
     ┌──────────────────────────────────────────────────────────┐
     │              DEFENSE IN DEPTH LAYERS                     │
     │                                                          │
     │  Layer 1: Physical Security (locks, cameras, badge)     │
     │  Layer 2: Network Segmentation (VLANs, DMZ)             │
     │  Layer 3: Perimeter Firewalls (NGFW, WAF)               │
     │  Layer 4: Host-Based Firewalls (iptables, Win FW)       │
     │  Layer 5: Protocol Security (LDAPS, SRTP, SNMPv3)      │
     │  Layer 6: Monitoring & Detection (SIEM, IDS/IPS)        │
     │  Layer 7: Encryption (TLS, VPN, WPA3)                   │
     │  Layer 8: Human Layer (Training, Awareness, Policy)     │
     └──────────────────────────────────────────────────────────┘
                                   │
                           Guided by:
                                   ▼
     ┌──────────────────────────────────────────────────────────┐
     │              10 RULES OF NETWORK SECURITY                │
     │                                                          │
     │  1. DEFAULT DENY — Block everything, whitelist needed   │
     │  2. LEAST PRIVILEGE — Minimum access for every entity   │
     │  3. DEFENSE IN DEPTH — Multiple layers, never just one  │
     │  4. ENCRYPT EVERYTHING — TLS, VPN, WPA3, LDAPS, SRTP   │
     │  5. SEGMENT ALWAYS — VLANs, DMZ, microsegmentation     │
     │  6. PATCH IMMEDIATELY — Unpatched = EternalBlue target  │
     │  7. MONITOR CONTINUOUSLY — Logs are your eyes           │
     │  8. DISABLE UNUSED — SMBv1, Telnet, SNMPv1, TFTP      │
     │  9. AUDIT REGULARLY — Quarterly firewall rule reviews   │
     │ 10. ASSUME BREACH — Zero Trust, plan for incident       │
     └──────────────────────────────────────────────────────────┘
```

---

## End of NETWORK_SECURITY_FUNDAMENTALS.md

> **Document Statistics**:
> - Module: NCSCJO — Network Security
> - Topics covered: 41 major sections, 100+ subsections
> - Coverage: Networking Foundations, OSI/TCP-IP, IPv4/Subnetting, Routing,
>   NAT/VLANs, LDAP, DHCP, ARP, NTP, DNS, SMTP, SNMP, TFTP, SIP/VoIP, SMB,
>   Firewalls (Types, iptables, nftables, UFW, firewalld, Windows, pfSense, Cloud),
>   tcpdump, Wireshark, Wireless Security (WEP/WPA/WPA2/WPA3), EternalBlue,
>   ARP Spoofing, DHCP Attacks, NTP Amplification, DMZ Architecture,
>   Segmentation, Policy Management, Auditing, Troubleshooting, Labs, Glossary
