# NETWORK SECURITY FUNDAMENTALS — Complete Zero-to-Hero Guide

> **Module**: NCSCJO — Network Security
> **Scope**: Networking Foundations, Protocols, Firewalls, Wireless Security, Protocol Exploits, Monitoring Tools, Architecture & Design
> **Level**: Beginner to Advanced to Expert
> **Goal**: Master every networking and network security concept from absolute zero to professional-grade expertise
> **Version**: 2.0 — Enhanced & Expanded Edition

---

# Table of Contents

```
PART I   — NETWORKING FOUNDATIONS
  1.  What Is a Network — The Absolute Basics
  2.  The OSI Model — Understanding the 7 Layers
  3.  The TCP/IP Model — The Internet's Real Architecture
  4.  IPv4 Addressing — The Language of Networks
  5.  IPv6 Addressing — The Future of Networking
  6.  Subnetting — Dividing Networks Like a Surgeon
  7.  Routing — How Packets Find Their Way
  8.  NAT / PAT — Translating Addresses at the Border
  9.  VLANs — Logical Network Segmentation
  10. Encapsulation — How Data Travels Through Layers

PART II  — NETWORK PROTOCOLS DEEP DIVE
  11. LDAP — Lightweight Directory Access Protocol
  12. DHCP — Dynamic Host Configuration Protocol
  13. ARP — Address Resolution Protocol
  14. NTP — Network Time Protocol
  15. DNS — Domain Name System
  16. SMTP / SMTPS — Email Protocols
  17. SNMP — Simple Network Management Protocol
  18. TFTP — Trivial File Transfer Protocol
  19. SIP & VoIP — Voice Over IP Security
  20. SMB — Server Message Block
  21. FTP / SFTP / SCP — File Transfer Protocols
  22. HTTP / HTTPS — Web Protocols Deep Dive
  23. SSH — Secure Shell Protocol

PART III — FIREWALLS — THE COMPLETE GUIDE
  24. What Is a Firewall — Absolute Fundamentals
  25. How Firewalls Work Internally — Packet Processing Pipeline
  26. Firewall Types — Complete Classification
  27. The Linux Firewall Stack — Netfilter Architecture
  28. iptables — The Classic Workhorse
  29. nftables — The Modern Successor
  30. UFW — Uncomplicated Firewall Deep Dive
  31. firewalld — Zone-Based Dynamic Firewall
  32. Windows Firewall — Defender Firewall with Advanced Security
  33. pfSense & OPNsense — Dedicated Firewall Appliances
  34. Cloud Firewalls — AWS, Azure, and GCP

PART IV  — NETWORK SECURITY TOOLS & MONITORING
  35. tcpdump — Command-Line Packet Analysis
  36. Wireshark — GUI Packet Analysis
  37. Network Monitoring Best Practices
  38. Port Knocking — Advanced Access Control

PART V   — WIRELESS SECURITY
  39. Wireless Standards — 802.11 Family
  40. Wireless Encryption — WEP, WPA, WPA2, WPA3
  41. Wireless Attacks and Defenses

PART VI  — PROTOCOL EXPLOITS & ATTACK TECHNIQUES
  42. EternalBlue — SMBv1 Exploitation
  43. ARP Spoofing Deep Dive
  44. DHCP Attacks — Starvation, Rogue Servers, TunnelVision
  45. NTP Amplification & NTP Relay Attacks
  46. LDAP Attacks — Bad Password Count, Enumeration, Spraying, Honeypot
  47. SIP/VoIP Attacks — Eavesdropping, Toll Fraud, Vishing
  48. DNS Attacks — Cache Poisoning, Tunneling, Amplification

PART VII — FIREWALL ARCHITECTURE & DESIGN
  49. DMZ Architecture — Multi-Zone Design
  50. Network Segmentation — Microsegmentation & Zero Trust
  51. NAT/PAT and Firewall Integration
  52. VPN Integration with Firewalls
  53. VPN vs Proxy — Understanding the Difference
  54. Firewall Policy Management and Best Practices

PART VIII— AUDITING, TROUBLESHOOTING & MASTERY
  55. Firewall Auditing — Finding Weaknesses
  56. Firewall Troubleshooting — Diagnosing Issues
  57. Common Mistakes — What Gets Networks Compromised
  58. Hands-On Labs
  59. Interview Questions — Prove Your Mastery
  60. Critical Analysis and Checkpoints
  61. Comprehensive Network Security Glossary
  62. Final Consolidated Review — The Big Picture
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

  ┌────────────────────┬─────────────────────────────────────────┐
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
  │ Registered Mail    │ TCP (reliable, tracked delivery)         │
  │ Postcard           │ UDP (fast, no tracking, no guarantee)    │
  │ P.O. Box           │ Port Number (specific service address)  │
  │ Return Address     │ Source Port (reply destination)          │
  └────────────────────┴─────────────────────────────────────────┘

  Just as a letter needs an envelope with addresses and goes through
  post offices to reach its destination, a data packet needs headers
  with IP addresses and passes through routers to reach its target.

  The analogy extends deeper:
  - A FIRE STATION only accepts fire-related calls. A WEB SERVER
    only accepts traffic on port 80/443.
  - A GATED COMMUNITY has a guard checking visitors. A FIREWALL
    checks every packet against its rules.
  - A TRANSLATOR converts between languages. The PRESENTATION LAYER
    converts between data formats (encryption, compression).
```

## 1.2 Why Networks Exist — The Business Case

```
WHY DO WE BUILD NETWORKS?

  1. RESOURCE SHARING:
     Without networks: Every employee needs their own printer,
     their own copy of every file, their own internet connection.
     With networks: 50 employees share 3 printers, a file server,
     and one internet connection. Cost savings: massive.

  2. COMMUNICATION:
     Email, instant messaging, video conferencing, VoIP calls.
     Without networks, coordination requires physical presence.

  3. CENTRALIZED MANAGEMENT:
     One Active Directory server manages authentication for
     10,000 employees. One SIEM collects logs from every device.
     Without networks, managing each device individually is impossible.

  4. DATA BACKUP AND REDUNDANCY:
     Files stored on network drives can be automatically backed up.
     Redundant servers ensure availability during failures.

  5. REMOTE ACCESS:
     VPN, SSH, RDP — work from anywhere.
     Cloud services — access applications globally.

  SECURITY PERSPECTIVE:
     Every benefit of networking is also a potential ATTACK VECTOR.
     Resource sharing leads to lateral movement opportunity.
     Communication allows eavesdropping opportunity.
     Centralized management creates single point of compromise.
     Remote access means remote exploitation.

     Understanding networks is the FOUNDATION of cybersecurity.
     You cannot defend what you do not understand.
```

## 1.3 Types of Networks

```
NETWORK TYPES BY SIZE:

  ┌──────────┬──────────────────────────────────────────────────────┐
  │ Type     │ Description                                          │
  ├──────────┼──────────────────────────────────────────────────────┤
  │ PAN      │ Personal Area Network — Bluetooth, NFC               │
  │          │ Range: ~10 meters. Your phone to your earbuds.       │
  │          │ Example: Bluetooth headset, smartwatch to phone.     │
  │          │ Security: Bluejacking, Bluesnarfing attacks.         │
  ├──────────┼──────────────────────────────────────────────────────┤
  │ LAN      │ Local Area Network — Ethernet, Wi-Fi                 │
  │          │ Range: Building/campus. Your office or home network. │
  │          │ Example: Office with 50 PCs connected to switches.   │
  │          │ Security: ARP spoofing, VLAN hopping, rogue devices. │
  ├──────────┼──────────────────────────────────────────────────────┤
  │ MAN      │ Metropolitan Area Network — City-wide                │
  │          │ Range: City. University campuses across a city.      │
  │          │ Example: City government connecting offices.         │
  │          │ Technology: Metro Ethernet, WiMAX, dark fiber.       │
  ├──────────┼──────────────────────────────────────────────────────┤
  │ WAN      │ Wide Area Network — Global                           │
  │          │ Range: Countries/World. The Internet is a WAN.       │
  │          │ Example: Company connecting offices in 20 countries. │
  │          │ Technology: MPLS, leased lines, SD-WAN, VPN tunnels. │
  ├──────────┼──────────────────────────────────────────────────────┤
  │ WLAN     │ Wireless LAN — Wi-Fi based LAN                      │
  │          │ Same as LAN but using wireless technology.           │
  │          │ Standards: 802.11a/b/g/n/ac/ax (Wi-Fi 4/5/6).       │
  │          │ Security: WPA3, rogue APs, evil twins, deauth.      │
  ├──────────┼──────────────────────────────────────────────────────┤
  │ SAN      │ Storage Area Network — Dedicated storage network     │
  │          │ Connects servers to shared storage arrays.           │
  │          │ Technology: Fibre Channel, iSCSI.                    │
  │          │ Security: Zoning, LUN masking, isolated from data.   │
  ├──────────┼──────────────────────────────────────────────────────┤
  │ CAN      │ Campus Area Network — Multiple buildings             │
  │          │ Range: University/corporate campus.                  │
  │          │ Multiple LANs connected via backbone switches.       │
  └──────────┴──────────────────────────────────────────────────────┘

NETWORK TYPES BY FUNCTION:

  Client-Server:
    Central server provides resources to clients.
    Example: Web server hosting a website, clients are browsers.
    Advantages: Centralized management, backup, security policies.
    Disadvantages: Single point of failure, server cost.
    Security: Harden the server — it is the crown jewel target.

  Peer-to-Peer (P2P):
    All devices are equal, share directly.
    Example: BitTorrent, LAN file sharing, blockchain nodes.
    Advantages: No central server needed, resilient, scalable.
    Disadvantages: Hard to manage, no centralized security.
    Security: Each peer is a potential attack vector.

  Hybrid:
    Combination of client-server and P2P.
    Example: Skype (signaling through servers, voice P2P).
    Most modern applications are hybrid architectures.
```

## 1.4 Key Network Devices

```
ESSENTIAL NETWORK HARDWARE:

  ┌─────────────┬──────────────────────────────────────────────────┐
  │ Device      │ Function                                          │
  ├─────────────┼──────────────────────────────────────────────────┤
  │ Hub         │ Layer 1 — Broadcasts to ALL ports. Dumb device.  │
  │             │ No intelligence. Obsolete in modern networks.     │
  │             │ Security: ALL traffic visible to ALL ports —      │
  │             │ passive sniffing trivial. Never use in production.│
  ├─────────────┼──────────────────────────────────────────────────┤
  │ Switch      │ Layer 2 — Forwards frames to SPECIFIC port       │
  │             │ based on MAC address table (CAM table). Smart.    │
  │             │ Managed switches support VLANs, port security,   │
  │             │ DHCP snooping, DAI, 802.1X authentication.       │
  │             │ Security: CAM table overflow, VLAN hopping,      │
  │             │ MAC spoofing, STP manipulation attacks.           │
  ├─────────────┼──────────────────────────────────────────────────┤
  │ Router      │ Layer 3 — Forwards packets between NETWORKS      │
  │             │ based on IP address and routing table.            │
  │             │ Connects different subnets and networks.          │
  │             │ Often includes ACLs for basic traffic filtering.  │
  │             │ Security: Route hijacking, default credentials,   │
  │             │ unpatched firmware, exposed management interfaces.│
  ├─────────────┼──────────────────────────────────────────────────┤
  │ Firewall    │ Layer 3-7 — Filters traffic based on rules.      │
  │             │ Decides what gets in and what gets blocked.       │
  │             │ Types: Packet filter, stateful, NGFW, WAF.       │
  │             │ THE most critical security device in any network. │
  ├─────────────┼──────────────────────────────────────────────────┤
  │ Access Point│ Layer 2 — Extends wired network to wireless.     │
  │ (AP)        │ Bridges Wi-Fi devices to the wired LAN.          │
  │             │ Enterprise APs: 802.1X, WPA3, WIDS.              │
  │             │ Security: Rogue APs, evil twins, deauth attacks. │
  ├─────────────┼──────────────────────────────────────────────────┤
  │ Load        │ Layer 4-7 — Distributes traffic across servers.  │
  │ Balancer    │ Types: Hardware (F5), Software (HAProxy, Nginx).  │
  │             │ Can perform SSL termination, health checks.       │
  │             │ Security: DDoS mitigation, SSL offloading risks. │
  ├─────────────┼──────────────────────────────────────────────────┤
  │ IDS/IPS     │ Layer 3-7 — Intrusion Detection/Prevention.      │
  │             │ IDS = Detection only (alerts).                    │
  │             │ IPS = Prevention (blocks).                        │
  │             │ Examples: Snort, Suricata, Zeek.                 │
  │             │ Deployment: Inline (IPS) or mirrored port (IDS). │
  ├─────────────┼──────────────────────────────────────────────────┤
  │ Proxy       │ Layer 7 — Intermediary between clients and       │
  │ Server      │ servers. Can cache, filter, and log requests.     │
  │             │ Forward proxy: Client to Proxy to Internet        │
  │             │ Reverse proxy: Internet to Proxy to Internal      │
  │             │ Security: Content filtering, SSL inspection.      │
  ├─────────────┼──────────────────────────────────────────────────┤
  │ VPN         │ Layer 3 — Creates encrypted tunnel across         │
  │ Gateway     │ untrusted networks. Types: Site-to-site,          │
  │             │ Remote access. Protocols: IPsec, WireGuard,       │
  │             │ OpenVPN, SSL VPN.                                  │
  └─────────────┴──────────────────────────────────────────────────┘

  Network Device Hardening Checklist:
  [x] Change ALL default credentials immediately
  [x] Disable unused ports and services
  [x] Keep firmware updated (patch management)
  [x] Use encrypted management protocols (SSH, not Telnet)
  [x] Implement management VLAN (isolated from user traffic)
  [x] Enable logging and send to central SIEM
  [x] Apply principle of least privilege to admin access
  [x] Use AAA (Authentication, Authorization, Accounting)
```

## 1.5 Network Topologies

```
NETWORK TOPOLOGIES — HOW DEVICES ARE CONNECTED:

  ┌──────────────┬────────────────────────────────────────────────────┐
  │ Topology     │ Description                                        │
  ├──────────────┼────────────────────────────────────────────────────┤
  │ Star         │ All devices connect to a CENTRAL switch/hub.      │
  │              │ Most common in modern LANs.                       │
  │              │ Failure of center node = entire network down.     │
  ├──────────────┼────────────────────────────────────────────────────┤
  │ Bus          │ All devices share a SINGLE cable (backbone).      │
  │              │ Legacy technology (10BASE2 coaxial). Obsolete.    │
  ├──────────────┼────────────────────────────────────────────────────┤
  │ Ring         │ Each device connects to exactly TWO others,       │
  │              │ forming a circular path. Token Ring / FDDI.        │
  ├──────────────┼────────────────────────────────────────────────────┤
  │ Mesh         │ Every device connects to every other device.      │
  │              │ Maximum redundancy. Used in WAN/data centers.     │
  ├──────────────┼────────────────────────────────────────────────────┤
  │ Hybrid       │ Combination of topologies. Modern enterprise      │
  │              │ networks are almost always hybrid (star-mesh).    │
  └──────────────┴────────────────────────────────────────────────────┘

  THREE-TIER ENTERPRISE ARCHITECTURE:

    ┌─────────────────────────────────────────────────────┐
    │              CORE LAYER                              │
    │  High-speed backbone. Redundant mesh topology.      │
    │  Purpose: Fast packet forwarding between all areas. │
    └───────────────────┬─────────────────────────────────┘
                        │
    ┌───────────────────┴─────────────────────────────────┐
    │           DISTRIBUTION LAYER                         │
    │  Policy enforcement, VLAN routing, access control.  │
    │  Aggregates traffic from access layer.              │
    └───────────────────┬─────────────────────────────────┘
                        │
    ┌───────────────────┴─────────────────────────────────┐
    │             ACCESS LAYER                             │
    │  Where end devices connect (PCs, phones, printers). │
    │  Port security, 802.1X, DHCP snooping, VLANs.      │
    └─────────────────────────────────────────────────────┘
```

---

# 2. The OSI Model — Understanding the 7 Layers

## 2.1 What Is the OSI Model?

```
THE OSI MODEL (Open Systems Interconnection):

  A 7-layer framework for understanding how network communication
  works, standardized by ISO.

  WHY IT MATTERS FOR SECURITY:
    Every attack targets a specific OSI layer.
    Every defense operates at a specific OSI layer.
    Understanding which layer you are working at determines
    which tools, protocols, and defenses are relevant.

  MNEMONIC (Top to Bottom): All People Seem To Need Data Processing
  MNEMONIC (Bottom to Top): Please Do Not Throw Sausage Pizza Away

  When a network engineer says "this is a Layer 2 problem" vs
  "this is a Layer 7 problem," they are telling you:
  - Layer 2 problem = switch issue, MAC address issue, cabling
  - Layer 7 problem = application issue, web server config, DNS
  This language is UNIVERSAL in IT and security. You MUST know it.
```

## 2.2 The Seven Layers — Complete Breakdown

```
THE SEVEN LAYERS:

  ┌─────┬───────────────┬──────────────┬────────────────────────────────┐
  │ #   │ Layer         │ Data Unit    │ Function                        │
  ├─────┼───────────────┼──────────────┼────────────────────────────────┤
  │  7  │ Application   │ Data         │ User interface and services     │
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

## 2.3 Each Layer — Detailed Explanation with Security Context

```
LAYER 7 — APPLICATION LAYER:

  Provides network services directly to end-user applications.
  This is the layer that HUMANS interact with.

  KEY CONCEPT: The Application Layer does NOT mean "the application
  itself." It means the NETWORK SERVICES that applications use.
  Firefox is an application. HTTP (which Firefox uses) is the
  Application Layer protocol.

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
    RDP         (3389)     — Windows Remote Desktop
    SIP         (5060/5061)— VoIP signaling
    NTP         (123)      — Time synchronization
    DHCP        (67/68)    — IP address assignment
    TFTP        (69)       — Trivial file transfer

  Attacks: SQL Injection, XSS, CSRF, Directory Traversal,
    Buffer Overflows, Phishing, Credential Stuffing, API Abuse,
    DNS Tunneling, Command Injection

  Controls: WAF, Input Validation, Authentication (MFA),
    Authorization, Rate Limiting, CSP, CORS

────────────────────────────────────────────────────────

LAYER 6 — PRESENTATION LAYER:

  Translates data between the application and the network.
  Handles encoding, encryption, and compression.

  Think of it as the TRANSLATOR at an international conference.

  Functions:
    Data Translation: ASCII to EBCDIC, Unicode conversions
    Encryption/Decryption: SSL/TLS encryption happens here
    Compression: Reduces data size for efficient transmission
    Serialization: Converting data structures to byte streams

  Examples: JPEG, PNG, MPEG, SSL/TLS, ASCII, Unicode, gzip, JSON

  Security Relevance:
    SSL/TLS vulnerabilities: POODLE, BEAST, Heartbleed, DROWN
    Certificate validation failures
    Weak cipher suites (RC4, DES)
    SSL stripping (downgrading HTTPS to HTTP)
    Data encoding bypasses (double URL encoding to bypass WAFs)

────────────────────────────────────────────────────────

LAYER 5 — SESSION LAYER:

  Manages sessions between applications.
  Establishes, maintains, and terminates communication sessions.

  Protocols: NetBIOS, RPC, PPTP, L2TP, SIP, SOCKS

  Security Relevance:
    Session hijacking — Attacker takes over an active session
    Session fixation — Attacker pre-sets the session ID
    Cookie theft — Stealing session cookies via XSS or sniffing
    Session replay — Capturing and replaying valid session data

  Defense:
    Use secure, random, long session tokens
    Set session expiration timeouts
    Regenerate session ID after authentication
    Use Secure and HttpOnly flags on cookies

────────────────────────────────────────────────────────

LAYER 4 — TRANSPORT LAYER:

  Provides end-to-end communication between processes.
  Ensures data arrives completely and in order (TCP) or quickly (UDP).

  TCP (Transmission Control Protocol):
    Connection-oriented — Three-way handshake (SYN, SYN-ACK, ACK)
    Reliable — Guarantees delivery via acknowledgments
    Ordered — Data arrives in sequence
    Error-checked — Checksums verify data integrity
    Flow controlled — Prevents overwhelming the receiver
    Used for: HTTP, SSH, FTP, SMTP, database connections

  UDP (User Datagram Protocol):
    Connectionless — No handshake, just send
    Unreliable — No guarantee of delivery
    Unordered — Packets may arrive out of sequence
    Fast — Minimal overhead (8-byte header only)
    Used for: DNS, DHCP, VoIP, streaming, gaming, NTP, SNMP, TFTP

  THE TCP THREE-WAY HANDSHAKE:

    Client                     Server
      |                          |
      |---- SYN (seq=100) ----->|  "I want to connect"
      |                          |
      |<--- SYN-ACK (seq=300, --|  "OK, I acknowledge"
      |     ack=101)             |
      |                          |
      |---- ACK (seq=101, ----->|  "Connection established"
      |     ack=301)             |
      |   CONNECTION ESTABLISHED  |

  TCP FLAGS:
    SYN     — Synchronize. "I want to start a connection"
    ACK     — Acknowledge. "I received your data"
    FIN     — Finish. "I want to close this connection"
    RST     — Reset. "Something is wrong, abort connection"
    PSH     — Push. "Process this data immediately"
    URG     — Urgent. "This data is high priority"

  WHY UDP EXISTS:
    Some applications need SPEED over reliability:
    - Video call: A dropped packet = tiny glitch. Acceptable.
      Retransmitting would cause even worse lag.
    - DNS query: One small question, one small answer.
      TCP handshake would triple the latency.
    - Online gaming: Your position 0.1 seconds ago is USELESS.

  Security Relevance:
    SYN Flood — Sends thousands of SYN packets without completing
    handshake, exhausting server resources (DoS).
    Defense: SYN cookies, rate limiting, firewall rules.

    Port Scanning — TCP SYN scan sends SYN, checks for SYN-ACK
    (port open) or RST (port closed). Tool: Nmap.

    TCP RST Injection — Forged RST packets terminate connections.

    UDP Flood — Overwhelming target with UDP packets.

────────────────────────────────────────────────────────

LAYER 3 — NETWORK LAYER:

  Handles logical addressing (IP addresses) and routing.
  Determines the best path for data to travel across networks.

  Protocols: IPv4, IPv6, ICMP, IGMP, IPsec, GRE
  Devices: Routers, Layer 3 switches, firewalls

  Security Relevance:
    IP spoofing — Forging source IP addresses
    ICMP attacks — Ping of Death, Smurf attack
    Route hijacking — BGP hijacking redirects internet traffic
    IP fragmentation attacks — Overlapping fragments bypass FW/IDS
    Reconnaissance — Traceroute reveals network topology

  Defense:
    Ingress/egress filtering (BCP 38) — Block spoofed IPs
    ACLs on routers — Filter traffic at Layer 3
    RPKI for BGP — Route Origin Validation
    Disable unnecessary ICMP types
    Use IPsec for sensitive inter-network communication

────────────────────────────────────────────────────────

LAYER 2 — DATA LINK LAYER:

  Handles physical addressing (MAC addresses) and
  controls access to the physical medium.

  Sub-layers: LLC (error checking, flow control) and
  MAC (physical addressing, frame formatting)

  MAC Address Format:
    48-bit address: AA:BB:CC:DD:EE:FF
    First 3 bytes = OUI (manufacturer identifier)
    FF:FF:FF:FF:FF:FF = Broadcast (all devices on LAN)

  Security Relevance:
    ARP spoofing/poisoning — Redirecting traffic via forged ARP
    MAC flooding — Overloading switch CAM table (becomes hub)
    MAC spoofing — Impersonating another device MAC address
    VLAN hopping — Double tagging, switch spoofing via DTP
    STP manipulation — Becoming root bridge to redirect traffic

  Defense:
    Dynamic ARP Inspection (DAI) on managed switches
    Port security — Limit MACs per port
    802.1X — Network Access Control
    Disable DTP — Prevent VLAN trunk negotiation
    BPDU Guard — Prevent STP manipulation
    DHCP snooping — Prevent rogue DHCP servers

────────────────────────────────────────────────────────

LAYER 1 — PHYSICAL LAYER:

  Transmits raw bits over the physical medium.
  Defines electrical signals, light pulses, radio frequencies.

  Media Types:
    Copper: Cat5e (1 Gbps), Cat6 (10 Gbps), Cat6a (10 Gbps at 100m)
    Fiber: Single-mode (long distance), Multi-mode (short distance)
    Wireless: Wi-Fi (802.11), Bluetooth, Cellular (4G/5G)

  Security Relevance:
    Physical wiretapping, fiber tapping
    Radio frequency jamming
    Physical theft of hardware
    Shoulder surfing, dumpster diving

  Best Practice:
    Locked server rooms with badge access and cameras
    Cable management to prevent unauthorized taps
    Full disk encryption in case of physical theft
    Secure disposal of equipment (degaussing, shredding)
    USB port lockdown (disable autorun)

  "If an attacker has physical access, you have already lost."
```

## 2.4 Attacks and Defenses by OSI Layer — Quick Reference

```
ATTACK SURFACE BY OSI LAYER:

  ┌─────┬───────────────┬────────────────────────────────────────────┐
  │ L#  │ Layer         │ Common Attacks                              │
  ├─────┼───────────────┼────────────────────────────────────────────┤
  │  7  │ Application   │ SQLi, XSS, CSRF, phishing, API abuse      │
  │  6  │ Presentation  │ SSL stripping, cipher downgrade, Heartbleed│
  │  5  │ Session       │ Session hijacking, fixation, cookie theft  │
  │  4  │ Transport     │ SYN flood, UDP flood, port scanning        │
  │  3  │ Network       │ IP spoofing, ICMP flood, route hijacking   │
  │  2  │ Data Link     │ ARP spoofing, MAC flooding, VLAN hopping   │
  │  1  │ Physical      │ Wiretapping, jamming, physical theft       │
  └─────┴───────────────┴────────────────────────────────────────────┘

  DEFENSE BY OSI LAYER:

  ┌─────┬───────────────┬────────────────────────────────────────────┐
  │ L#  │ Layer         │ Security Controls                           │
  ├─────┼───────────────┼────────────────────────────────────────────┤
  │  7  │ Application   │ WAF, input validation, auth, rate limiting │
  │  6  │ Presentation  │ TLS 1.3, strong ciphers, cert pinning      │
  │  5  │ Session       │ Secure tokens, timeouts, session rotation  │
  │  4  │ Transport     │ SYN cookies, rate limits, firewall rules   │
  │  3  │ Network       │ ACLs, RPKI, IPsec, ingress filtering      │
  │  2  │ Data Link     │ DAI, port security, 802.1X, BPDU guard    │
  │  1  │ Physical      │ Locks, cameras, encryption, secure wipe   │
  └─────┴───────────────┴────────────────────────────────────────────┘

  Key Insight: Defense in depth means implementing security controls
  at EVERY layer, not just one. A firewall (Layer 3-4) cannot stop
  a SQL injection (Layer 7). A WAF (Layer 7) cannot stop ARP spoofing
  (Layer 2). You need controls at every layer.
```

---

# 3. The TCP/IP Model — The Internet's Real Architecture

## 3.1 TCP/IP vs OSI

```
THE TCP/IP MODEL — WHAT THE INTERNET ACTUALLY USES:

  While the OSI model is the THEORETICAL framework (7 layers),
  the TCP/IP model is the PRACTICAL implementation (4 layers)
  that the internet is actually built on.

  OSI is for UNDERSTANDING. TCP/IP is for BUILDING.

  COMPARISON:

    OSI Model              TCP/IP Model         Protocols
    ─────────              ──────────           ─────────
    7. Application  ┐
    6. Presentation ├──→   4. Application       HTTP, FTP, DNS,
    5. Session      ┘                           SSH, SMTP, SNMP

    4. Transport    ──→    3. Transport          TCP, UDP

    3. Network      ──→    2. Internet           IP, ICMP, ARP, IGMP

    2. Data Link    ┐
    1. Physical     ┘──→   1. Network Access     Ethernet, Wi-Fi,
                           (Link Layer)          PPP, Frame Relay

  WHY TCP/IP ONLY HAS 4 LAYERS:
    The OSI model was designed by academics for completeness.
    TCP/IP was designed by engineers for practicality.
    Layers 5-7 of OSI are merged because in practice, most
    protocols handle presentation and session management
    within the application itself.

TCP/IP LAYER DETAILS:

  APPLICATION LAYER (Layer 4):
    Combines OSI Layers 5, 6, and 7.
    Where user-facing protocols operate.
    Each application protocol defines its own data format,
    session management, and (optionally) encryption.

  TRANSPORT LAYER (Layer 3):
    Identical to OSI Layer 4.
    TCP provides reliable, ordered delivery.
    UDP provides fast, best-effort delivery.

  INTERNET LAYER (Layer 2):
    Corresponds to OSI Layer 3.
    IP handles logical addressing and routing.
    ICMP provides error reporting and diagnostics.
    ARP maps IP addresses to MAC addresses.

  NETWORK ACCESS LAYER (Layer 1):
    Combines OSI Layers 1 and 2.
    Handles physical transmission and data link framing.
    Ethernet, Wi-Fi, PPP, and other link-layer technologies.
```

## 3.2 Common Protocols and Their Ports

```
CRITICAL PORTS EVERY SECURITY PROFESSIONAL MUST MEMORIZE:

  ┌──────────┬──────────┬──────────────────────────────────────────┐
  │ Port     │ Protocol │ Service and Security Notes                │
  ├──────────┼──────────┼──────────────────────────────────────────┤
  │ 20/21    │ TCP      │ FTP — File Transfer. PLAINTEXT.          │
  │          │          │ Credentials visible to sniffers.          │
  │          │          │ Use SFTP (port 22) or FTPS instead.      │
  ├──────────┼──────────┼──────────────────────────────────────────┤
  │ 22       │ TCP      │ SSH — Secure Shell. Encrypted remote.    │
  │          │          │ Also used for SFTP and SCP.               │
  │          │          │ Brute force target. Use key-based auth.  │
  ├──────────┼──────────┼──────────────────────────────────────────┤
  │ 23       │ TCP      │ Telnet — INSECURE remote access.         │
  │          │          │ Everything in PLAINTEXT. NEVER USE.       │
  ├──────────┼──────────┼──────────────────────────────────────────┤
  │ 25       │ TCP      │ SMTP — Email sending. PLAINTEXT.         │
  │          │          │ Easy to spoof sender. Use SPF/DKIM/DMARC.│
  ├──────────┼──────────┼──────────────────────────────────────────┤
  │ 53       │ TCP/UDP  │ DNS — Domain Name System.                │
  │          │          │ UDP for queries, TCP for zone transfers.  │
  │          │          │ Cache poisoning, tunneling, amplification.│
  ├──────────┼──────────┼──────────────────────────────────────────┤
  │ 67/68    │ UDP      │ DHCP — IP address assignment.            │
  │          │          │ No authentication. Rogue server attacks.  │
  ├──────────┼──────────┼──────────────────────────────────────────┤
  │ 69       │ UDP      │ TFTP — Trivial file transfer.            │
  │          │          │ NO authentication. NO encryption.         │
  ├──────────┼──────────┼──────────────────────────────────────────┤
  │ 80       │ TCP      │ HTTP — Web traffic. UNENCRYPTED.         │
  │          │          │ Vulnerable to interception and injection. │
  ├──────────┼──────────┼──────────────────────────────────────────┤
  │ 110      │ TCP      │ POP3 — Retrieve email. PLAINTEXT.        │
  ├──────────┼──────────┼──────────────────────────────────────────┤
  │ 123      │ UDP      │ NTP — Time sync. Amplification attacks.  │
  ├──────────┼──────────┼──────────────────────────────────────────┤
  │ 135      │ TCP      │ MS-RPC — Windows RPC. Worm target.       │
  ├──────────┼──────────┼──────────────────────────────────────────┤
  │ 139      │ TCP      │ NetBIOS — Legacy Windows networking.     │
  ├──────────┼──────────┼──────────────────────────────────────────┤
  │ 143      │ TCP      │ IMAP — Retrieve email. PLAINTEXT.        │
  ├──────────┼──────────┼──────────────────────────────────────────┤
  │ 161/162  │ UDP      │ SNMP — Network management.               │
  │          │          │ v1/v2c PLAINTEXT. Always use SNMPv3.     │
  ├──────────┼──────────┼──────────────────────────────────────────┤
  │ 389      │ TCP      │ LDAP — Directory services. PLAINTEXT.    │
  ├──────────┼──────────┼──────────────────────────────────────────┤
  │ 443      │ TCP      │ HTTPS — Encrypted web traffic.           │
  │          │          │ TLS encryption protects data in transit.  │
  ├──────────┼──────────┼──────────────────────────────────────────┤
  │ 445      │ TCP      │ SMB — Windows file sharing.              │
  │          │          │ EternalBlue target. Disable SMBv1.       │
  ├──────────┼──────────┼──────────────────────────────────────────┤
  │ 465      │ TCP      │ SMTPS — Encrypted email sending.         │
  ├──────────┼──────────┼──────────────────────────────────────────┤
  │ 587      │ TCP      │ SMTP Submission — Email with STARTTLS.   │
  ├──────────┼──────────┼──────────────────────────────────────────┤
  │ 636      │ TCP      │ LDAPS — Encrypted LDAP.                  │
  ├──────────┼──────────┼──────────────────────────────────────────┤
  │ 993      │ TCP      │ IMAPS — Encrypted IMAP.                  │
  ├──────────┼──────────┼──────────────────────────────────────────┤
  │ 995      │ TCP      │ POP3S — Encrypted POP3.                  │
  ├──────────┼──────────┼──────────────────────────────────────────┤
  │ 1433     │ TCP      │ MS-SQL — Microsoft SQL Server.           │
  ├──────────┼──────────┼──────────────────────────────────────────┤
  │ 1521     │ TCP      │ Oracle DB — Oracle database.             │
  ├──────────┼──────────┼──────────────────────────────────────────┤
  │ 3306     │ TCP      │ MySQL/MariaDB — Database.                │
  ├──────────┼──────────┼──────────────────────────────────────────┤
  │ 3389     │ TCP      │ RDP — Remote Desktop. Brute force target.│
  │          │          │ Use VPN + MFA. Never expose to internet. │
  ├──────────┼──────────┼──────────────────────────────────────────┤
  │ 5060/5061│ TCP/UDP  │ SIP — VoIP signaling / SIP over TLS.    │
  ├──────────┼──────────┼──────────────────────────────────────────┤
  │ 5432     │ TCP      │ PostgreSQL — Database.                   │
  ├──────────┼──────────┼──────────────────────────────────────────┤
  │ 5900     │ TCP      │ VNC — Remote desktop. Often unencrypted. │
  ├──────────┼──────────┼──────────────────────────────────────────┤
  │ 8080     │ TCP      │ HTTP Proxy / Alternate web server.       │
  ├──────────┼──────────┼──────────────────────────────────────────┤
  │ 8443     │ TCP      │ HTTPS alternate / management interfaces. │
  └──────────┴──────────┴──────────────────────────────────────────┘

  PORT RANGES:
    Well-Known Ports:   0 - 1023    (system services, require root)
    Registered Ports:   1024 - 49151 (application services)
    Dynamic/Ephemeral:  49152 - 65535 (temporary client ports)

  SECURITY RULE: If a port is open, it is an attack surface.
  Close every port you do not need. Monitor every port you keep open.
```

---

# 4. IPv4 Addressing — The Language of Networks

## 4.1 IPv4 Address Structure

```
IPv4 ADDRESSING:

  An IPv4 address is a 32-bit number written as four decimal octets.
  Each octet ranges from 0 to 255.

  Example: 192.168.1.100

  Binary representation:
  192     .168     .1       .100
  11000000.10101000.00000001.01100100

  Total possible IPv4 addresses: 2^32 = 4,294,967,296 (~4.3 billion)
  This is NOT enough for all devices in the modern world (30+ billion).
  This scarcity led to NAT and eventually IPv6.

  AN IPv4 ADDRESS HAS TWO PARTS:

    ┌──────────────────────────┬──────────────────────────┐
    │     NETWORK PORTION      │      HOST PORTION         │
    │  (Which network?)        │  (Which device on that    │
    │                          │   network?)               │
    └──────────────────────────┴──────────────────────────┘

    The SUBNET MASK determines where the boundary falls.

    Example:
      IP:      192.168.1.100
      Mask:    255.255.255.0 (/24)

      Network: 192.168.1.0     (first 24 bits)
      Host:    .100            (last 8 bits)

      This means: "Device 100 on the 192.168.1.0 network"
```

## 4.2 IP Address Classes

```
IPv4 ADDRESS CLASSES (Historical/Legacy):

  ┌───────┬──────────────────────┬────────────┬───────────────────┐
  │ Class │ Range                │ Default    │ Purpose            │
  │       │                      │ Mask       │                    │
  ├───────┼──────────────────────┼────────────┼───────────────────┤
  │ A     │ 1.0.0.0 -            │ /8         │ Huge networks      │
  │       │ 126.255.255.255      │ 255.0.0.0  │ 16.7M hosts each  │
  ├───────┼──────────────────────┼────────────┼───────────────────┤
  │ B     │ 128.0.0.0 -          │ /16        │ Medium networks    │
  │       │ 191.255.255.255      │ 255.255.0.0│ 65,534 hosts each │
  ├───────┼──────────────────────┼────────────┼───────────────────┤
  │ C     │ 192.0.0.0 -          │ /24        │ Small networks     │
  │       │ 223.255.255.255      │ 255.255.   │ 254 hosts each    │
  │       │                      │ 255.0      │                    │
  ├───────┼──────────────────────┼────────────┼───────────────────┤
  │ D     │ 224.0.0.0 -          │ N/A        │ Multicast          │
  │       │ 239.255.255.255      │            │                    │
  ├───────┼──────────────────────┼────────────┼───────────────────┤
  │ E     │ 240.0.0.0 -          │ N/A        │ Reserved/Research  │
  │       │ 255.255.255.255      │            │                    │
  └───────┴──────────────────────┴────────────┴───────────────────┘

PRIVATE IP RANGES (RFC 1918) — MEMORIZE THESE:

  These addresses are NOT routable on the internet.
  Used internally within organizations. Translated via NAT.

  ┌──────────┬──────────────────────────┬──────────────────────┐
  │ Class    │ Private Range            │ CIDR Notation         │
  ├──────────┼──────────────────────────┼──────────────────────┤
  │ A        │ 10.0.0.0 - 10.255.      │ 10.0.0.0/8           │
  │          │ 255.255                  │ 16.7 million hosts   │
  ├──────────┼──────────────────────────┼──────────────────────┤
  │ B        │ 172.16.0.0 - 172.31.    │ 172.16.0.0/12        │
  │          │ 255.255                  │ 1 million hosts      │
  ├──────────┼──────────────────────────┼──────────────────────┤
  │ C        │ 192.168.0.0 - 192.168.  │ 192.168.0.0/16       │
  │          │ 255.255                  │ 65,534 hosts         │
  └──────────┴──────────────────────────┴──────────────────────┘

SPECIAL ADDRESSES:

  127.0.0.0/8       — Loopback (localhost). 127.0.0.1 = this machine.
  169.254.0.0/16    — APIPA (link-local). Auto-assigned when DHCP fails.
  0.0.0.0           — "Any address" or "unknown." Used in routing.
  255.255.255.255   — Limited broadcast (current network only).
  x.x.x.0           — Network address (identifies the network itself).
  x.x.x.255         — Directed broadcast (broadcast to specific subnet).

  SECURITY NOTE: If you see 169.254.x.x on a device, DHCP has failed.
  This is a red flag — could indicate DHCP starvation attack or
  network misconfiguration.
```

## 4.3 Subnet Mask Explained

```
THE SUBNET MASK — THE MOST MISUNDERSTOOD CONCEPT:

  A subnet mask is a 32-bit number that SEPARATES the network
  portion from the host portion of an IP address.

  IMPORTANT: The subnet mask is NOT an address. It is a FILTER
  that tells the device "which bits of my IP address identify
  my network, and which bits identify me on that network."

  HOW IT WORKS (Binary):

    IP Address:  192.168.1.100  = 11000000.10101000.00000001.01100100
    Subnet Mask: 255.255.255.0  = 11111111.11111111.11111111.00000000
                                  ├── Network (1s) ──┤├─ Host (0s) ─┤

    AND operation:
    IP:   11000000.10101000.00000001.01100100
    Mask: 11111111.11111111.11111111.00000000
    ────────────────────────────────────────
    Net:  11000000.10101000.00000001.00000000 = 192.168.1.0

  The subnet mask tells the device:
  "The first 24 bits identify my network. The last 8 bits identify me."

  CIDR NOTATION:
    Instead of writing 255.255.255.0, write /24
    /24 means "24 bits for network, 8 bits for hosts"

    Common subnet masks:
    /8  = 255.0.0.0       — 16,777,214 hosts
    /16 = 255.255.0.0     — 65,534 hosts
    /24 = 255.255.255.0   — 254 hosts
    /25 = 255.255.255.128 — 126 hosts
    /26 = 255.255.255.192 — 62 hosts
    /27 = 255.255.255.224 — 30 hosts
    /28 = 255.255.255.240 — 14 hosts
    /29 = 255.255.255.248 — 6 hosts
    /30 = 255.255.255.252 — 2 hosts (point-to-point links)
    /32 = 255.255.255.255 — 1 host (single host route)

  WHY SUBNET MASKS MATTER FOR SECURITY:
    A device sends traffic DIRECTLY (Layer 2) to devices on
    the SAME subnet. Traffic to DIFFERENT subnets must go
    through a ROUTER (or Layer 3 switch).

    If two devices are on the same subnet, there is NO router
    (and typically no firewall) between them. ARP spoofing
    attacks work because devices on the same subnet communicate
    directly via MAC addresses.

    This is why NETWORK SEGMENTATION (VLANs + subnets) is
    critical — it forces traffic through controlled chokepoints
    where firewalls can inspect and filter.
```

---

# 5. IPv6 Addressing — The Future of Networking

```
IPv6 (Internet Protocol version 6):

  WHY IPv6 EXISTS:
    IPv4 has ~4.3 billion addresses. Not enough.
    IPv6 has 2^128 = 340 undecillion addresses.
    That is 340,282,366,920,938,463,463,374,607,431,768,211,456 addresses.
    Enough for every grain of sand on Earth to have billions of IPs.

  FORMAT:
    128-bit address written as 8 groups of 4 hex digits.
    Example: 2001:0db8:85a3:0000:0000:8a2e:0370:7334

    Shortening rules:
    - Leading zeros in a group can be omitted:
      2001:0db8 becomes 2001:db8
    - Consecutive groups of all zeros can be replaced with ::
      (only once per address):
      2001:0db8:0000:0000:0000:0000:0000:0001 becomes 2001:db8::1

  KEY IPv6 ADDRESS TYPES:
    ::1             — Loopback (equivalent to 127.0.0.1)
    fe80::/10       — Link-local (auto-configured, not routable)
    fc00::/7        — Unique local (equivalent to RFC1918 private)
    2000::/3        — Global unicast (public, routable)
    ff00::/8        — Multicast (replaces broadcast)

  SECURITY IMPLICATIONS OF IPv6:
    - No NAT needed (every device can have a public IP)
      This means internal devices may be directly reachable
    - IPsec was originally mandatory (now recommended)
    - New attack surface: IPv6 neighbor discovery attacks
      (equivalent to ARP spoofing in IPv4)
    - Dual-stack environments (IPv4 + IPv6) create complexity.
      Attackers may bypass IPv4 firewalls via IPv6 if IPv6
      rules are not configured
    - Many organizations deploy IPv6 unknowingly (auto-configured)
      without corresponding security policies

  CRITICAL: If your firewall only has IPv4 rules, IPv6 traffic
  may flow completely unfiltered. Always configure firewall rules
  for BOTH IPv4 AND IPv6, or disable IPv6 if not needed.
```

---

# 6. Subnetting — Dividing Networks Like a Surgeon

## 6.1 Why Subnet?

```
WHY WE SUBNET:

  1. BROADCAST CONTROL:
     Without subnetting, a single broadcast reaches ALL devices.
     In a network of 10,000 devices, broadcasts consume massive
     bandwidth and CPU on every device (broadcast storms).
     Subnetting limits broadcasts to smaller domains.

  2. SECURITY:
     Subnetting enables SEGMENTATION. Different subnets can have
     different security policies. Firewall rules between subnets
     control what traffic flows between departments.
     Example: HR subnet cannot directly reach the server subnet.

  3. PERFORMANCE:
     Smaller broadcast domains = less noise = better performance.
     Devices process fewer irrelevant broadcasts.

  4. MANAGEMENT:
     Easier to organize, troubleshoot, and manage smaller networks.
     Logical grouping: Department A = 10.0.1.0/24, Dept B = 10.0.2.0/24

  5. ADDRESS EFFICIENCY:
     Allocate only the addresses needed for each segment.
     A point-to-point link needs /30 (2 hosts), not /24 (254 hosts).

CIDR NOTATION QUICK REFERENCE:

  /24  = 256 addresses, 254 usable hosts  (most common LAN)
  /25  = 128 addresses, 126 usable hosts
  /26  = 64 addresses, 62 usable hosts
  /27  = 32 addresses, 30 usable hosts
  /28  = 16 addresses, 14 usable hosts    (small department)
  /29  = 8 addresses, 6 usable hosts      (server farm)
  /30  = 4 addresses, 2 usable hosts      (point-to-point)
  /32  = 1 address (single host route)

  Formula: Usable hosts = 2^(32-prefix) - 2
  The -2 accounts for network address and broadcast address.

SUBNETTING EXAMPLE:

  Given: 192.168.1.0/24 — Split into 4 equal subnets

  New prefix: /26 (2 extra bits borrowed = 4 subnets)

  Subnet 1: 192.168.1.0/26    (hosts: .1 to .62,  broadcast: .63)
  Subnet 2: 192.168.1.64/26   (hosts: .65 to .126, broadcast: .127)
  Subnet 3: 192.168.1.128/26  (hosts: .129 to .190, broadcast: .191)
  Subnet 4: 192.168.1.192/26  (hosts: .193 to .254, broadcast: .255)

  Each subnet has 62 usable host addresses.
```

---

# 7. Routing — How Packets Find Their Way

## 7.1 How Routers Make Decisions

```
ROUTING FUNDAMENTALS:

  A router connects multiple networks and determines the BEST PATH
  for each packet to reach its destination.

  HOW A ROUTER PROCESSES A PACKET:

  1. Extract the DESTINATION IP ADDRESS from the packet header
  2. Look up the destination in the ROUTING TABLE
  3. Find the BEST MATCH (longest prefix match)
  4. Forward the packet to the NEXT HOP router or directly to host
  5. If no match found, send to DEFAULT GATEWAY (0.0.0.0/0)
  6. If no default gateway, DROP the packet (destination unreachable)
```

## 7.2 Routing Protocols

```
ROUTING PROTOCOLS — HOW ROUTERS LEARN ROUTES:

  STATIC ROUTING:
    Routes manually configured by an administrator.
    Simple but does not adapt to network changes.
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
    │            │ Uses Dijkstra algorithm for shortest path.       │
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
        2018 — Attackers hijacked BGP routes to Amazon Route 53
        DNS service, redirecting MyEtherWallet.com traffic to steal
        $150,000 in cryptocurrency.

      Defense:
        RPKI (Resource Public Key Infrastructure) for route validation
        BGP route filtering and prefix lists
        IRR (Internet Routing Registry) verification
        BGPsec for cryptographic route attestation
```

---

# 8. NAT / PAT — Translating Addresses at the Border

## 8.1 What Is NAT?

```
NETWORK ADDRESS TRANSLATION (NAT):

  NAT translates PRIVATE IP addresses to PUBLIC IP addresses
  (and vice versa) as traffic crosses the network boundary.

  WHY NAT EXISTS:
    Problem: There are only ~4.3 billion IPv4 addresses, but there
    are 30+ billion connected devices. Not enough public IPs!

    Solution: Use PRIVATE addresses internally (10.x, 172.16.x, 192.168.x)
    and translate them to a SHARED public IP for internet access.

    Your entire home network (phone, laptop, tablet, smart TV) uses
    ONE public IP address. The router uses NAT to keep track of which
    internal device made which request.

NAT TYPES:

  ┌──────────────────┬──────────────────────────────────────────────┐
  │ Type             │ How It Works                                  │
  ├──────────────────┼──────────────────────────────────────────────┤
  │ Static NAT       │ 1:1 mapping. One private IP = one public IP. │
  │ (SNAT)           │ Used for servers that need a fixed public IP. │
  │                  │ Example: Web server 10.0.0.5 maps to          │
  │                  │ 203.0.113.5                                    │
  ├──────────────────┼──────────────────────────────────────────────┤
  │ Dynamic NAT      │ Pool of public IPs. Private IPs are mapped   │
  │                  │ to available public IPs from pool on-demand.  │
  │                  │ Less common today.                            │
  ├──────────────────┼──────────────────────────────────────────────┤
  │ PAT / NAT        │ MANY private IPs share ONE public IP.         │
  │ Overload         │ Differentiated by PORT NUMBERS.               │
  │ (Port Address    │ This is what your home router does.           │
  │  Translation)    │ Also called "masquerading" in Linux.          │
  │                  │                                               │
  │                  │ 10.0.0.5:49321 to 203.0.113.1:49321            │
  │                  │ 10.0.0.6:52100 to 203.0.113.1:52100            │
  │                  │ 10.0.0.7:38000 to 203.0.113.1:38000            │
  │                  │ (All use the SAME public IP, different ports) │
  └──────────────────┴──────────────────────────────────────────────┘

  SECURITY NOTE:
    NAT provides OBSCURITY, not SECURITY.
    NAT hides internal IPs from the Internet (attackers cannot
    directly reach 192.168.1.5 from outside). But NAT is NOT a
    firewall. NAT does not inspect packets, does not block
    malicious traffic, and does not replace a proper firewall.
```

---

# 9. VLANs — Logical Network Segmentation

## 9.1 What Are VLANs?

```
VIRTUAL LOCAL AREA NETWORKS (VLANs):

  A VLAN is a method of creating LOGICALLY separate networks
  on the SAME physical switch infrastructure.

  Without VLANs: All devices on a switch are in the same broadcast
  domain. A broadcast from one device reaches ALL other devices.

  With VLANs: The switch creates virtual boundaries. Devices in
  different VLANs cannot communicate directly — they need a ROUTER
  (or Layer 3 switch) to route traffic between VLANs.

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
  [x] Segmentation — Limits blast radius of a breach
  [x] Access Control — Firewall rules between VLANs
  [x] Compliance — PCI-DSS: isolate cardholder data network
  [x] Reduced Broadcast — Better performance per VLAN
  [x] Cost Effective — No need for separate physical switches

VLAN SECURITY ATTACKS:

  VLAN Hopping (Double Tagging):
    Attacker sends a frame with TWO VLAN tags.
    The first switch strips the outer tag (attacker native VLAN).
    The inner tag routes the frame to a different, target VLAN.

    Defense: Set native VLAN to an unused VLAN ID.
    Never use VLAN 1 as the native VLAN. Explicitly tag all ports.

  Switch Spoofing:
    Attacker configures NIC to act as a trunk port.
    If DTP (Dynamic Trunking Protocol) is enabled, the attacker
    negotiates a trunk and gains access to all VLANs.

    Defense: Disable DTP. Set all access ports to
    "switchport mode access". Only manually configure trunks.
```

---

# 10. Encapsulation — How Data Travels Through Layers

## 10.1 The Encapsulation Process

```
ENCAPSULATION — DATA WRAPPING AT EACH LAYER:

  When data is sent across a network, each OSI/TCP-IP layer WRAPS
  the data from the layer above with its own HEADER (and sometimes
  a trailer). This process is called ENCAPSULATION.

  The reverse process at the receiving end is DE-ENCAPSULATION.

  ┌─────────────────────────────────────────────────────────────────┐
  │                    ENCAPSULATION PROCESS                        │
  │                                                                 │
  │  Application Layer:    [ DATA ]                                │
  │                         "Hello, this is a web page"            │
  │                                                                 │
  │  Transport Layer:      [ TCP Header | DATA ]                   │
  │                         Adds: Source port, Dest port,          │
  │                         Sequence number, Flags                  │
  │                         Now called a SEGMENT                    │
  │                                                                 │
  │  Network Layer:        [ IP Header | TCP Header | DATA ]       │
  │                         Adds: Source IP, Dest IP,              │
  │                         TTL, Protocol                           │
  │                         Now called a PACKET                     │
  │                                                                 │
  │  Data Link Layer:      [ ETH Header | IP | TCP | DATA | FCS ] │
  │                         Adds: Source MAC, Dest MAC,            │
  │                         EtherType, Frame Check Sequence        │
  │                         Now called a FRAME                      │
  │                                                                 │
  │  Physical Layer:       01101001 01001000 11010010 ...          │
  │                         Raw bits on the wire or radio waves    │
  │                         Now called BITS                         │
  └─────────────────────────────────────────────────────────────────┘

  DATA UNIT NAMES BY LAYER:

    Application  =  Data
    Transport    =  Segment (TCP) / Datagram (UDP)
    Network      =  Packet
    Data Link    =  Frame
    Physical     =  Bits

  Security Insight:
    Each layer header is a potential attack surface.
    Attackers can craft custom packets with modified headers:
    - Modified IP header: IP spoofing (Layer 3)
    - Modified TCP flags: SYN flood, TCP RST injection (Layer 4)
    - Modified MAC header: MAC spoofing (Layer 2)
    - Modified application data: SQL injection, XSS (Layer 7)
```

---

# PART II — NETWORK PROTOCOLS DEEP DIVE

---

# 11. LDAP — Lightweight Directory Access Protocol

## 11.1 What Is LDAP?

```
LDAP (Lightweight Directory Access Protocol):

  An open, vendor-neutral APPLICATION PROTOCOL used to access and
  maintain distributed directory information services over IP networks.

  Think of LDAP as a specialized database designed for READ-HEAVY,
  HIGHLY AVAILABLE LOOKUPS. Commonly used for authentication
  and storing information about users, groups, devices, and
  organizational structure.

  THE PHONE BOOK ANALOGY:
    LDAP is like a digital phone book for an organization.
    You can look up a person by name and find their:
    - Email address, phone number, department, manager
    - Login credentials, group memberships

    But unlike a phone book, LDAP is:
    - Hierarchical (tree structure)
    - Queryable (search with filters)
    - Writable (add/modify/delete entries)
    - Access-controlled (different permissions per user)

LDAP vs. ACTIVE DIRECTORY — THE CRITICAL DISTINCTION:

  LDAP is the PROTOCOL — the set of rules for communicating.
  Active Directory is the SERVICE — Microsoft complete directory product.

  ┌──────────────────┬───────────────────────┬───────────────────────┐
  │ Feature          │ LDAP                  │ Active Directory (AD) │
  ├──────────────────┼───────────────────────┼───────────────────────┤
  │ Core Identity    │ Open protocol         │ Microsoft product     │
  │ Function         │ Directory comms rules │ Complete IAM solution │
  │ Platform         │ Cross-platform        │ Windows-centric       │
  │ Features         │ Operations, data      │ LDAP + Kerberos +     │
  │                  │ model, security       │ Group Policy + DNS    │
  │ Cost             │ Free (OpenLDAP)       │ Licensed Microsoft    │
  │ Ports            │ 389 (LDAP)            │ 389 (LDAP)            │
  │                  │ 636 (LDAPS)           │ 636 (LDAPS)           │
  │                  │                       │ 3268 (Global Catalog) │
  └──────────────────┴───────────────────────┴───────────────────────┘

  Active Directory SPEAKS the LDAP language, but AD is much more
  than just LDAP. AD also uses Kerberos for authentication,
  Group Policy for configuration, and DNS for name resolution.
```

## 11.2 LDAP Architecture — The Directory Information Tree

```
THE LDAP DATA MODEL:

  LDAP stores data in a hierarchical tree called the
  DIRECTORY INFORMATION TREE (DIT).

  EXAMPLE DIT STRUCTURE:

    dc=example,dc=com                     <-- ROOT (Domain Component)
    |
    +-- ou=Users                          <-- Organizational Unit
    |   +-- cn=John Doe                   <-- Entry (Common Name)
    |   |   +-- mail: john@example.com    <-- Attribute
    |   |   +-- uid: jdoe                 <-- Attribute
    |   |   +-- userPassword: {SSHA}...   <-- Attribute
    |   |
    |   +-- cn=Jane Smith
    |       +-- mail: jane@example.com
    |       +-- uid: jsmith
    |
    +-- ou=Groups
    |   +-- cn=Admins
    |   +-- cn=Developers
    |
    +-- ou=Servers
        +-- cn=web-server-01
        +-- cn=db-server-01

KEY TERMINOLOGY:

  Entry: The fundamental unit - represents one object (person, group, device)
  Attribute: A piece of information about an entry (Type + Value)
  Object Class: Blueprint defining required/optional attributes
  Schema: Complete set of rules for valid object classes and attributes
  Distinguished Name (DN): Full path to an entry
    Example: cn=John Doe,ou=Users,dc=example,dc=com
  Relative Distinguished Name (RDN): Just the entry own identifier
    Example: cn=John Doe

  Common Attribute Types:
    dc = Domain Component    ou = Organizational Unit
    cn = Common Name         sn = Surname
    uid = User ID            mail = Email address
    userPassword = Password hash
```

## 11.3 LDAP Operations

```
LDAP OPERATIONS — THE CORE COMMANDS:

  BINDING AND SESSION:
    Bind     — Authenticate to the LDAP server (provide DN + credentials)
    Unbind   — Terminate the LDAP session
    Abandon  — Cancel a pending operation

  QUERYING:
    Search   — The MOST common operation. Query DIT with filters.
               Example filter: (&(objectClass=person)(mail=*@example.com))
    Compare  — Check if a specific attribute value exists in an entry

  UPDATING:
    Add      — Insert a new entry into the DIT
    Delete   — Remove an entry (must be a leaf, no children)
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

## 11.4 LDAP Security

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

    StartTLS (Extended Operation):
      Port 389. Starts as plaintext, then upgrades to TLS.
      Standard method defined in LDAPv3.
      More flexible — uses single port for both modes.

    CRITICAL: Always use one of these. NEVER send LDAP
    credentials in plaintext over the network!

  ACCESS CONTROL:
    ACLs or ACIs define who can read, write, search, or compare attributes.
    Example: Allow users to change their own password but
    deny them from reading other users password hashes.
```

## 11.5 LDAP Attacks — Bad Password Count and Account Lockout

```
LDAP ATTACK: BAD PASSWORD COUNT EXPLOITATION

  "Bad Password Count" is a directory service mechanism for tracking
  FAILED authentication attempts against a user account.
  It is central to password throttling and account lockout policies.

THE VULNERABILITY — WEAPONIZING THE DEFENSE:

  Attack: Lockout Denial of Service
    1. Attacker identifies the LDAP server (port 389/636)
    2. Enumerates valid usernames (via anonymous queries or guessing)
    3. Floods authentication requests with WRONG passwords
    4. Bad password count exceeds threshold (e.g., 5 attempts)
    5. Accounts become LOCKED — legitimate users denied access

  Impact:
    - Denial of Service — Hundreds of users locked out
    - Noise Generation — Distracts security team from real attack
    - Business Disruption — Users cannot access email, VPN, workstations

THE THREE PHASES OF IDENTITY-BASED ATTACKS:

  1. ENUMERATION (Reconnaissance):
     Finding valid usernames before trying passwords.

     Techniques:
       - Different error messages for "Invalid Username" vs "Invalid Password"
       - Response time differences (non-existent user fails instantly,
         real user takes time to check password hash)
       - Anonymous LDAP queries returning user lists

     Defense: ALWAYS return generic "Bad username or password" error.

  2. PATTERN (Password Spraying):
     "Low and slow" approach to avoid lockout.

     Traditional Brute Force: Try 100 passwords on ONE account
       Locks the account after 5 attempts. DETECTED.

     Password Spraying: Try ONE password on 100 accounts
       Only 1 bad attempt per account. Under the radar.
       Wait 30 minutes, try second password on all 100 accounts.
       After 4 rounds: 400 attempts, zero lockouts.
       If 2 users had weak passwords, attacker is IN.

     Commonly tried passwords:
       Season2026!, Password123, Company2026!, Welcome1

  3. HONEYPOT (Deception):
     Defender creates FAKE accounts that look legitimate:
       - svc-backup, ftp-user, admin-test
       - Very long password (64 chars, impossible to guess)
       - No real access to any data
       - Monitoring rule: "ANY login attempt = alert immediately"

     When attacker includes honeypot in spray, instant detection.

DEFENSE IN DEPTH FOR LDAP:
  [x] Network Segmentation — Do not expose LDAP to internet
  [x] Smart Lockout — Distinguish mistype from botnet (ADFS smart lockout)
  [x] Application-Level Throttling — CAPTCHA, cool-down before LDAP
  [x] Monitoring — Alert on unusual spikes in bad password events
  [x] Generic Errors — Never reveal whether username exists
  [x] Honeypot Accounts — Trap attackers during enumeration/spraying
```

---

# 12. DHCP — Dynamic Host Configuration Protocol

## 12.1 What Is DHCP?

```
DHCP (Dynamic Host Configuration Protocol):

  Ports: 67 (server), 68 (client). Protocol: UDP (connectionless)

  DHCP automates the assignment of IP addresses and other network
  configuration parameters to devices on a network.

  Without DHCP: Manually configure every device (IP, mask, gateway, DNS).
  With DHCP: Devices get all configuration automatically on connect.

THE DHCP PROCESS — DORA:

  ┌──────────┐                              ┌──────────┐
  │  CLIENT  │                              │  SERVER  │
  └────┬─────┘                              └────┬─────┘
       │                                         │
       │---- 1. DISCOVER (broadcast) ----------->│
       │     "Is there a DHCP server?"           │
       │     Src: 0.0.0.0  Dst: 255.255.255.255 │
       │                                         │
       │<--- 2. OFFER --------------------------│
       │     "Here is an IP you can use"         │
       │     Offered IP: 192.168.1.100           │
       │                                         │
       │---- 3. REQUEST (broadcast) ------------>│
       │     "I want that IP please"             │
       │     Requested IP: 192.168.1.100         │
       │                                         │
       │<--- 4. ACK ----------------------------│
       │     "It is yours! Lease: 24 hours"      │
       │                                         │
       │     CLIENT NOW HAS:                     │
       │     IP: 192.168.1.100                   │
       │     Mask: 255.255.255.0                 │
       │     Gateway: 192.168.1.1                │
       │     DNS: 8.8.8.8, 8.8.4.4              │
       └─────────────────────────────────────────┘

  WHY IS DISCOVER A BROADCAST?
    Client has NO IP address yet — it shouts to the entire network.

  WHY IS REQUEST A BROADCAST (not unicast)?
    To tell ALL DHCP servers "I accepted THIS server offer"
    so other servers can retract their offers.
```

## 12.2 DHCP Attacks

```
DHCP ATTACKS — EXPLOITING TRUST:

  DHCP was designed in an era of inherent network trust.
  It has NO built-in authentication. Any device can claim to
  be a DHCP server. This makes it a prime attack target.

ATTACK 1: DHCP STARVATION (Availability Attack)
  Attacker floods network with DHCPDISCOVER requests,
  each using a DIFFERENT, SPOOFED MAC address.
  Goal: Exhaust the DHCP server pool of available IPs.
  Result: Legitimate new clients are DENIED service.
  Tool: Yersinia, DHCPig

ATTACK 2: ROGUE DHCP SERVER (Integrity and Confidentiality Attack)
  Attacker sets up a MALICIOUS DHCP server on the network.
  When clients broadcast DHCPDISCOVER, the rogue server responds
  BEFORE the legitimate one.
  Goal: Assign MALICIOUS network configurations:
    - Set attacker as DEFAULT GATEWAY = Man-in-the-Middle
    - Hand out attacker DNS server = DNS hijacking (phishing)
    - Assign wrong subnet mask = Network disruption

ATTACK 3: DHCP REPLAY ATTACK
  Attacker captures valid DHCP messages and replays them at a later
  time or on a different network segment. Can be used to assign
  previously-valid but now stale configurations, causing conflicts
  or enabling network access with specific parameters.

ATTACK 4: TUNNELVISION / VPN DECLOAKING (CVE-2024-3661)
  Attacker with local network access forces VPN user traffic
  OUT of the encrypted tunnel by manipulating DHCP options.
  Attacker sets their own machine as the gateway via DHCP.
  Impact: Complete bypass of VPN protection on local network.

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
    │   Maps valid IP to MAC assignments.                     │
    │   Used by Dynamic ARP Inspection (DAI).               │
    └─────────────────────────────────────────────────────────┘

  PORT SECURITY: Limit MACs per port to mitigate starvation
  NETWORK MONITORING: Alert on high DHCPDISCOVER rates
  VLAN SEGMENTATION: Isolate DHCP servers in secure VLAN
```

---

# 13. ARP — Address Resolution Protocol

## 13.1 What Is ARP?

```
ARP (Address Resolution Protocol):

  ARP maps NETWORK LAYER addresses (IP) to DATA LINK LAYER
  addresses (MAC) within a LOCAL network.

  WHY ARP IS NEEDED:
    When your computer wants to send data to 192.168.1.1,
    it needs to know the MAC address of that IP to create
    the Ethernet frame. IP addresses are logical; Ethernet
    switches forward based on MAC addresses.

  HOW ARP WORKS:
    1. Device A wants to talk to 192.168.1.1
    2. Device A checks its ARP CACHE (local table of IP to MAC mappings)
    3. If not in cache, sends ARP REQUEST (broadcast):
       "Who has IP 192.168.1.1? Tell 192.168.1.5"
    4. ALL devices on the network receive this broadcast
    5. The device WITH 192.168.1.1 sends ARP REPLY (unicast):
       "192.168.1.1 is at MAC AA:BB:CC:DD:EE:FF"
    6. Device A stores this mapping in its ARP cache
    7. Device A can now send Ethernet frames to that MAC address

THE FUNDAMENTAL SECURITY FLAW:

  ARP is STATELESS and has NO AUTHENTICATION.

  ARP accepts responses even if it NEVER sent a request.
  ARP blindly updates its cache with ANY ARP reply it receives.
  There is NO way to verify an ARP reply is legitimate.

  This is by design — ARP was created in a trusted network era.
```

## 13.2 ARP Spoofing / Poisoning Attack

```
ARP SPOOFING (ARP POISONING) — MAN-IN-THE-MIDDLE:

  The attacker sends FORGED ARP REPLIES to trick devices into
  updating their ARP caches with the attacker MAC address.

  ATTACK MECHANICS:

    BEFORE ATTACK:
      Victim ARP Cache:          Gateway ARP Cache:
      192.168.1.1 = GW-MAC       192.168.1.5 = VICTIM-MAC

      Victim <---- Normal traffic ----> Gateway <---> Internet

    STEP 1: Attacker sends forged ARP replies:
      To Victim:  "192.168.1.1 (gateway) is at ATTACKER-MAC"
      To Gateway: "192.168.1.5 (victim) is at ATTACKER-MAC"

    STEP 2: ARP caches are poisoned:
      Victim ARP Cache:          Gateway ARP Cache:
      192.168.1.1 = ATTACKER-MAC 192.168.1.5 = ATTACKER-MAC

    STEP 3: All traffic now flows through attacker:
      Victim --> ATTACKER --> Gateway --> Internet
      Victim <-- ATTACKER <-- Gateway <-- Internet

    STEP 4: Attacker enables IP forwarding to stay hidden.
      Traffic flows normally, but attacker can:
      - Eavesdrop (read passwords, emails, cookies)
      - Modify data in transit (inject malicious code)
      - Steal session tokens (session hijacking)
      - Redirect to fake websites

  TOOLS:
    arpspoof (dsniff): arpspoof -i eth0 -t <victim> <gateway>
    Ettercap: GUI-based MitM tool with ARP spoofing
    Bettercap: Modern, powerful MitM framework

DEFENSES:
  [x] Dynamic ARP Inspection (DAI):
      Switch feature using DHCP Snooping Binding Table.
      Validates ARP packets against known-good IP-MAC bindings.
  [x] Static ARP Entries:
      Manually define ARP entries for critical devices.
      arp -s 192.168.1.1 AA:BB:CC:DD:EE:FF
  [x] Port Security + 802.1X:
      Limit MAC addresses per port. Require authentication.
  [x] Encryption:
      Use HTTPS, SSH, VPNs. Does not prevent ARP spoofing but
      renders intercepted data unreadable.
  [x] Detection Tools:
      ARPWatch, XArp, Snort IDS — Monitor for ARP anomalies.
```

---

# 14. NTP — Network Time Protocol

## 14.1 What Is NTP?

```
NTP (Network Time Protocol):
  Port: 123 (UDP)
  Purpose: Synchronize clocks between computer systems.

  WHY ACCURATE TIME MATTERS FOR SECURITY:
    - Log correlation — Synchronized timestamps for incident investigation
    - Kerberos authentication — 5-minute clock skew tolerance
    - Certificate validation — SSL/TLS certificates have validity periods
    - Scheduled tasks — Backups, patches, cron jobs depend on time
    - Digital forensics — Accurate timestamps are evidence

  NTP uses a hierarchical system of time sources:
    Stratum 0: Atomic clocks, GPS receivers (reference clocks)
    Stratum 1: Servers directly connected to Stratum 0 devices
    Stratum 2: Servers synchronized to Stratum 1 servers
    ...up to Stratum 15. Stratum 16 = unsynchronized.
```

## 14.2 NTP Amplification Attack (DDoS)

```
NTP AMPLIFICATION — DDoS VIA TIME SERVERS:

  A Distributed Denial of Service attack that leverages publicly
  accessible NTP servers. Both a REFLECTION and AMPLIFICATION attack.

THE VULNERABILITY — monlist COMMAND:

  Older NTP implementations have a diagnostic command "monlist"
  (MON_GETLIST) enabled by default. monlist returns the last 600
  hosts that connected. Request is tiny (~40 bytes), response is
  MASSIVE (often 100x larger). Amplification factor: 1:100 or higher!

ATTACK MECHANICS:

  1. SPOOFING: Attacker crafts small NTP monlist requests
     with SOURCE IP forged to be the VICTIM IP.
  2. REFLECTION: Sends spoofed requests to thousands of
     vulnerable NTP servers on the internet.
  3. AMPLIFICATION: Each NTP server sends large monlist
     response (100x size) to the VICTIM IP.
  4. OVERLOAD: Victim receives massive flood from thousands
     of NTP servers simultaneously. Network saturated. DoS.

NTP RELAY ATTACK (TIME MANIPULATION):

  A different type where the goal is to CHANGE the victim system clock.
  Using ARP spoofing or MitM, attacker intercepts NTP requests and
  modifies the timestamp in the response.

  Why manipulate time?
    - Bypass Security Tokens — TOTP codes expire/become invalid
    - Expire Encryption Keys — Certificates appear expired
    - Log Tampering — Wrong timestamps break forensic analysis
    - Cache Poisoning — Time-dependent cache becomes invalid
    - Kerberos Failure — Authentication breaks (5-min tolerance)

DEFENSES:
  [x] Patch NTP — Upgrade to ntp-4.2.7p26+ (monlist disabled)
  [x] Disable monlist — Add "disable monitor" to ntp.conf
  [x] Ingress Filtering (BCP 38) — Block packets with spoofed IPs
  [x] Rate Limiting — Limit incoming NTP traffic volume
  [x] DDoS Mitigation — Cloudflare, Akamai, AWS Shield
  [x] NTP Authentication — Use symmetric key or Autokey authentication
```

---

# 15. DNS — Domain Name System

```
DNS (Domain Name System):

  Port: 53 (TCP and UDP)
  Purpose: Translates human-readable domain names to IP addresses.
  Without DNS: You would type http://142.250.185.78 instead of google.com

  DNS is one of the most CRITICAL services on the Internet.
  If DNS fails, NOTHING works — no web, no email, no applications.

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
  DNS Amplification — DDoS using open DNS resolvers
  DNS Hijacking — Modify DNS settings to redirect traffic
  Domain Squatting — Register look-alike domains for phishing

DEFENSES:
  DNSSEC — Cryptographic signatures on DNS records
  DNS over HTTPS (DoH) / DNS over TLS (DoT) — Encrypted DNS queries
  Response Rate Limiting — Mitigate DNS amplification
  DNS Monitoring — Detect anomalous query patterns
```

---

# 16. SMTP / SMTPS — Email Protocols

```
SMTP (Simple Mail Transfer Protocol):

  Ports: 25 (SMTP), 465 (SMTPS), 587 (Submission)
  Purpose: SEND email between mail servers

  SMTP is a TEXT-BASED protocol. You can literally telnet to
  a mail server and type email commands manually:

  # Manual SMTP session (educational purposes):
  telnet mail.example.com 25
  HELO attacker.com
  MAIL FROM:<ceo@company.com>     <-- Can be ANY address (spoofed!)
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
    DNS TXT record listing which servers can send for your domain.
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

# 17. SNMP — Simple Network Management Protocol

```
SNMP (Simple Network Management Protocol):

  Ports: 161 (queries), 162 (traps/notifications)
  Purpose: Monitor and manage network devices

SNMP VERSIONS — CRITICAL SECURITY DIFFERENCES:

  ┌──────────┬──────────────────────────────────────────────────────┐
  │ Version  │ Security                                              │
  ├──────────┼──────────────────────────────────────────────────────┤
  │ SNMPv1   │ Community strings in PLAINTEXT.                      │
  │          │ Default: "public" (read), "private" (read-write).   │
  │          │ ZERO encryption. ZERO authentication.                │
  │          │ NEVER USE IN PRODUCTION.                              │
  ├──────────┼──────────────────────────────────────────────────────┤
  │ SNMPv2c  │ Still uses community strings in PLAINTEXT.           │
  │          │ Better performance but same security problems.       │
  ├──────────┼──────────────────────────────────────────────────────┤
  │ SNMPv3   │ ENCRYPTED communication (DES, AES).                  │
  │          │ AUTHENTICATED users (username + password + engine).  │
  │          │ Integrity verification (HMAC-MD5/SHA).               │
  │          │ USE THIS VERSION. Always.                              │
  └──────────┴──────────────────────────────────────────────────────┘

  SECURITY RISK with SNMPv1/v2c and default community strings:
    - Attacker can read ENTIRE device configuration
    - Attacker can modify device settings
    - Attacker can extract routing tables, ARP tables, user lists
    - Attacker can disable interfaces, change passwords

  ALWAYS: Use SNMPv3. Change default community strings. Restrict
  SNMP access to management network only via ACLs.
```

---

# 18. TFTP — Trivial File Transfer Protocol

```
TFTP (Trivial File Transfer Protocol):
  Port: 69 (UDP)
  Purpose: Simple file transfer with NO authentication

  TFTP is designed for simplicity, not security:
    - No authentication — Anyone can read/write files
    - No encryption — Data transferred in plaintext
    - No directory listing — Must know exact filename
    - UDP-based — Unreliable, no connection state

  Use Cases:
    - PXE boot (network boot of diskless workstations)
    - Router/switch firmware upgrades
    - Configuration backup/restore on network devices

  SECURITY WARNING:
    TFTP should NEVER be exposed to untrusted networks.
    Restrict to isolated management VLANs.
    Attackers finding an open TFTP server can:
    - Download device configurations (containing passwords)
    - Upload malicious firmware
    - Overwrite critical files
```

---

# 19. SIP & VoIP — Voice Over IP Security

## 19.1 Understanding SIP and VoIP

```
VoIP (Voice over Internet Protocol):
  Technology allowing voice communication over IP networks.

SIP (Session Initiation Protocol):
  The dominant SIGNALING protocol for VoIP calls.
  Ports: 5060 (SIP), 5061 (SIP over TLS)

RTP (Real-time Transport Protocol):
  Carries the actual voice and video media once a call is established.

THE RESTAURANT ANALOGY:

  The WAITER is SIP:
    1. You tell the waiter: "I want to talk to my friend at table 5"
       SIP sends INVITE to the other party
    2. Waiter goes to table 5, gets attention
       SIP locates the user, their phone rings
    3. Friend says "Sure, I will talk"
       SIP returns "200 OK" — connection established
    4. You start talking directly to each other
       Voice travels via RTP, NOT through SIP
    5. When done, you tell the waiter "We are finished"
       SIP sends BYE to terminate the session

  SIP is the MANAGER. SIP does NOT carry your voice. RTP does.
```

## 19.2 SIP/VoIP Attacks

```
VoIP/SIP ATTACKS:

  1. REGISTRATION HIJACKING:
     Attacker re-registers victim contact address with SIP registrar.
     Incoming calls for victim are routed to attacker.

  2. INVITE FLOODING (DoS):
     Flooding SIP server with INVITE requests overwhelms it.

  3. SIP MESSAGE TAMPERING:
     Modifying SIP messages in transit to redirect or drop calls.

  4. EAVESDROPPING / CALL INTERCEPTION:
     Capture RTP voice packets with Wireshark on same network.
     If unencrypted, attacker can reassemble and LISTEN to calls.

  5. MAN-IN-THE-MIDDLE:
     ARP spoofing or rogue DHCP to intercept, modify, or record calls.

  6. VISHING (Voice Phishing):
     Social engineering via VoIP — impersonating banks, IT, executives.

  7. TOLL FRAUD:
     Compromise VoIP system to make premium-rate or international calls.

  8. SPIT (Spam over Internet Telephony):
     The VoIP equivalent of email spam — automated bulk calls.

DEFENSES:
  [x] Session Border Controller (SBC) — VoIP firewall
      Topology hiding, protocol validation, DoS protection, encryption
  [x] Encryption — TLS for SIP signaling + SRTP for voice media
  [x] Separate VLANs — Isolate VoIP on its own VLAN
  [x] Strong Authentication — Strong passwords, MFA for admin
  [x] IDS/IPS — Deploy with SIP-specific attack signatures
```

---

# 20. SMB — Server Message Block

```
SMB (Server Message Block):

  Port: 445 (modern), 139 (legacy over NetBIOS)
  Purpose: Network file sharing protocol for Windows.

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

# 21. FTP / SFTP / SCP — File Transfer Protocols

```
FILE TRANSFER PROTOCOLS — SECURITY COMPARISON:

  FTP (File Transfer Protocol):
    Ports: 21 (control), 20 (data)
    Security: NONE. Everything in PLAINTEXT including credentials.
    Passive vs Active mode: Passive uses random high ports (firewall-friendly).
    NEVER use FTP for sensitive data. Use SFTP or SCP instead.

  FTPS (FTP over SSL/TLS):
    Ports: 990 (implicit), 21 (explicit with STARTTLS)
    FTP wrapped in TLS encryption. Better than plain FTP.
    Still has complexity issues with firewall rules (multiple ports).

  SFTP (SSH File Transfer Protocol):
    Port: 22 (same as SSH)
    Runs over SSH tunnel. Strong encryption and authentication.
    Single port makes firewall configuration simple.
    RECOMMENDED for secure file transfers.

  SCP (Secure Copy Protocol):
    Port: 22 (same as SSH)
    Simple file copy over SSH. No directory listing or resume.
    Being deprecated in favor of SFTP in modern systems.

  COMPARISON:
    ┌──────────┬──────────┬──────────┬──────────┬──────────┐
    │ Feature  │ FTP      │ FTPS     │ SFTP     │ SCP      │
    ├──────────┼──────────┼──────────┼──────────┼──────────┤
    │ Encrypted│ No       │ Yes(TLS) │ Yes(SSH) │ Yes(SSH) │
    │ Port     │ 20,21    │ 990/21   │ 22       │ 22       │
    │ Auth     │ Plaintext│ TLS cert │ SSH key  │ SSH key  │
    │ Firewall │ Complex  │ Complex  │ Simple   │ Simple   │
    │ Recommend│ NEVER    │ OK       │ YES      │ Deprecated│
    └──────────┴──────────┴──────────┴──────────┴──────────┘
```

---

# 22. HTTP / HTTPS — Web Protocols Deep Dive

```
HTTP (Hypertext Transfer Protocol):
  Port: 80 (HTTP), 443 (HTTPS)
  Purpose: Foundation of the World Wide Web.

  HTTP is STATELESS — each request is independent.
  Cookies and sessions add state management on top.

  HTTP METHODS:
    GET     — Retrieve resource (should not modify data)
    POST    — Submit data (create resource)
    PUT     — Update/replace resource
    DELETE  — Remove resource
    PATCH   — Partial update
    HEAD    — Like GET but headers only (no body)
    OPTIONS — Discover allowed methods (used in CORS preflight)

  HTTP STATUS CODES:
    1xx — Informational (100 Continue)
    2xx — Success (200 OK, 201 Created, 204 No Content)
    3xx — Redirection (301 Moved, 302 Found, 304 Not Modified)
    4xx — Client Error (400 Bad Request, 401 Unauthorized,
           403 Forbidden, 404 Not Found, 429 Too Many Requests)
    5xx — Server Error (500 Internal Server Error, 502 Bad Gateway,
           503 Service Unavailable)

  HTTPS = HTTP + TLS:
    TLS encrypts the entire HTTP conversation.
    Provides: Confidentiality, Integrity, Authentication.
    TLS 1.2 is minimum acceptable. TLS 1.3 is recommended.
    SSL 2.0, SSL 3.0, TLS 1.0, TLS 1.1 are all DEPRECATED.

  SECURITY HEADERS:
    Strict-Transport-Security (HSTS) — Force HTTPS
    Content-Security-Policy (CSP) — Prevent XSS
    X-Content-Type-Options: nosniff — Prevent MIME sniffing
    X-Frame-Options: DENY — Prevent clickjacking
    X-XSS-Protection — Browser XSS filter
    Referrer-Policy — Control referrer information
```

---

# 23. SSH — Secure Shell Protocol

```
SSH (Secure Shell):
  Port: 22
  Purpose: Encrypted remote access and command execution.

  SSH replaced Telnet and rlogin which sent everything in PLAINTEXT.

  SSH CAPABILITIES:
    Remote shell access (secure command line)
    SFTP (secure file transfer)
    SCP (secure file copy)
    Port forwarding / tunneling
    X11 forwarding (remote GUI applications)
    SOCKS proxy (dynamic port forwarding)

  SSH AUTHENTICATION METHODS:
    Password — Simple but vulnerable to brute force
    Public Key — RECOMMENDED. Key pair (public + private)
    Certificate — SSH certificates signed by a CA
    GSSAPI/Kerberos — Enterprise SSO integration

  SSH KEY-BASED AUTHENTICATION:
    1. Generate key pair: ssh-keygen -t ed25519
    2. Copy public key to server: ssh-copy-id user@server
    3. Connect: ssh user@server (no password needed)

    The private key stays on YOUR machine (never share it).
    The public key goes on the SERVER (authorized_keys file).

  SSH HARDENING:
    # In /etc/ssh/sshd_config:
    PermitRootLogin no           # Never allow root SSH
    PasswordAuthentication no     # Force key-based auth
    Port 2222                     # Change from default 22
    MaxAuthTries 3                # Limit failed attempts
    AllowUsers admin deploy       # Whitelist specific users
    Protocol 2                    # SSH protocol version 2 only
    LoginGraceTime 30             # 30 seconds to authenticate
    X11Forwarding no              # Disable if not needed

  SSH SECURITY THREATS:
    Brute force attacks — Automated password guessing
      Defense: Key-based auth, fail2ban, rate limiting
    Man-in-the-Middle — If host key not verified on first connect
      Defense: Verify host fingerprint, use known_hosts
    SSH tunneling abuse — Attackers use SSH to bypass firewalls
      Defense: Monitor SSH sessions, restrict port forwarding
```

---

# PART III — FIREWALLS — THE COMPLETE GUIDE

---

# 24. What Is a Firewall — Absolute Fundamentals

```
FIREWALL — THE GATEKEEPER OF YOUR NETWORK:

  A firewall is a network security device (hardware, software, or both)
  that monitors incoming and outgoing network traffic and decides
  whether to allow or block specific traffic based on predefined rules.

  REAL-WORLD ANALOGY — THE NIGHTCLUB BOUNCER:

    ┌──────────────────────────────────────────────────────────┐
    │                                                          │
    │   Nightclub (Bouncer)    =   Network (Firewall)         │
    │                                                          │
    │   Guest list              =   Access Control List (ACL)  │
    │   Checking ID             =   Inspecting packet headers │
    │   "No sneakers" policy    =   Block specific ports      │
    │   VIP entrance            =   Trusted network zone      │
    │   Denied entry            =   Packet DROPPED            │
    │   Kicked out              =   Connection RESET           │
    │   Watching behavior       =   Stateful inspection        │
    │   Calling police          =   Logging + SIEM alert       │
    │                                                          │
    └──────────────────────────────────────────────────────────┘

FIREWALL PLACEMENT:

  HOST-BASED FIREWALL:
    Software running ON the device it protects.
    Examples: Windows Defender Firewall, iptables, UFW
    Protects the individual machine regardless of network position.
    Last line of defense if network firewall is bypassed.

  NETWORK-BASED FIREWALL:
    Dedicated device at the network perimeter or between segments.
    Examples: pfSense, Palo Alto, Fortinet, Cisco ASA
    Protects entire network segments or the whole organization.
    First line of defense at the network boundary.

  BEST PRACTICE: Use BOTH. Defense in depth.
    Network firewall filters at the perimeter.
    Host-based firewall protects each individual machine.
```

---

# 25. How Firewalls Work Internally — Packet Processing Pipeline

```
FIREWALL PACKET PROCESSING — HOW DECISIONS ARE MADE:

  When a packet arrives, the firewall processes it through
  a decision pipeline:

  Incoming Packet
       │
       ▼
  ┌─────────────────────────────────────┐
  │  1. RECEIVE PACKET                  │
  │     Read source/dest IP, ports,     │
  │     protocol, flags                 │
  └──────────────┬──────────────────────┘
                 │
                 ▼
  ┌─────────────────────────────────────┐
  │  2. CHECK STATE TABLE              │
  │     Is this packet part of an      │
  │     EXISTING, ESTABLISHED          │
  │     connection?                     │
  │     YES → Allow (already approved) │
  │     NO  → Continue to rule check   │
  └──────────────┬──────────────────────┘
                 │ (New connection)
                 ▼
  ┌─────────────────────────────────────┐
  │  3. MATCH AGAINST RULES            │
  │     Go through rules TOP to BOTTOM │
  │     FIRST MATCH WINS               │
  │                                     │
  │     Rule 1: Allow SSH from 10.0.0.x│
  │     Rule 2: Allow HTTP from any    │
  │     Rule 3: Allow HTTPS from any   │
  │     Rule 4: Block all              │
  │                                     │
  │     If packet matches Rule 2 →     │
  │     STOP checking, apply action    │
  └──────────────┬──────────────────────┘
                 │
                 ▼
  ┌─────────────────────────────────────┐
  │  4. APPLY ACTION                   │
  │     ACCEPT = Allow packet through  │
  │     DROP   = Silently discard      │
  │     REJECT = Discard + send error  │
  │     LOG    = Record then continue  │
  └──────────────┬──────────────────────┘
                 │
                 ▼
  ┌─────────────────────────────────────┐
  │  5. UPDATE STATE TABLE             │
  │     Record this new connection     │
  │     Future packets in same flow    │
  │     will be auto-allowed           │
  └─────────────────────────────────────┘

CRITICAL CONCEPT: FIRST MATCH WINS

  Rules are evaluated TOP to BOTTOM. The FIRST rule that matches
  determines the action. All subsequent rules are IGNORED.

  This means RULE ORDER IS EVERYTHING.

  WRONG ORDER (broken):
    Rule 1: ALLOW ALL traffic         <-- Matches everything first!
    Rule 2: BLOCK port 445 (SMB)      <-- NEVER reaches this rule

  CORRECT ORDER:
    Rule 1: BLOCK port 445 (SMB)      <-- Specific block first
    Rule 2: ALLOW HTTP traffic        <-- Then specific allows
    Rule 3: BLOCK ALL remaining       <-- Default deny last

STATEFUL VS STATELESS:

  STATELESS FIREWALL:
    Examines each packet in ISOLATION. No memory of previous packets.
    Does NOT know if a packet is part of an established connection.
    Must explicitly allow BOTH directions of traffic.
    Simple, fast, but limited. Like a guard with amnesia.

  STATEFUL FIREWALL:
    Maintains a STATE TABLE tracking all active connections.
    Automatically allows RETURN traffic for established connections.
    Knows the difference between a legitimate response and an
    unsolicited inbound connection. Much smarter, more secure.

    State Table Example:
    ┌────────────┬────────────┬────────────┬────────────┬───────┐
    │ Source IP   │ Src Port   │ Dest IP    │ Dst Port   │ State │
    ├────────────┼────────────┼────────────┼────────────┼───────┤
    │ 10.0.0.5   │ 49321      │ 8.8.8.8    │ 443        │ ESTAB │
    │ 10.0.0.6   │ 52100      │ 142.250.x  │ 80         │ ESTAB │
    │ 10.0.0.7   │ 38042      │ 10.0.1.5   │ 22         │ NEW   │
    └────────────┴────────────┴────────────┴────────────┴───────┘
```

---

# 26. Firewall Types — Complete Classification

```
FIREWALL TYPES — FROM SIMPLE TO ADVANCED:

  TYPE 1: PACKET FILTER (Layer 3)
    The oldest and simplest type.
    Decisions based on: source/dest IP, source/dest port, protocol.
    Does NOT inspect packet contents (payload).
    Stateless — each packet examined independently.
    Fast but easily bypassed by application-layer attacks.

  TYPE 2: STATEFUL INSPECTION FIREWALL (Layer 3-4)
    Tracks connection state (NEW, ESTABLISHED, RELATED, INVALID).
    Automatically allows return traffic for approved connections.
    Much more secure than simple packet filtering.
    Standard for most network firewalls today.

  TYPE 3: APPLICATION LAYER GATEWAY / PROXY (Layer 7)
    Inspects the CONTENT of packets, not just headers.
    Understands application protocols (HTTP, FTP, DNS).
    Can detect attacks hidden within legitimate-looking traffic.
    Higher security but higher latency (must process content).

  TYPE 4: WEB APPLICATION FIREWALL / WAF (Layer 7)
    SPECIFICALLY designed to protect web applications.
    Inspects HTTP/HTTPS traffic for application-layer attacks:
      SQL Injection, XSS, CSRF, file inclusion, etc.
    Understands web application logic, not just packets.
    Examples: ModSecurity, AWS WAF, Cloudflare WAF, Imperva.

  TYPE 5: NEXT-GENERATION FIREWALL / NGFW (Layer 3-7)
    Combines EVERYTHING:
      Traditional packet filtering + Stateful inspection
      + Deep Packet Inspection (DPI)
      + Application awareness (identify apps, not just ports)
      + Integrated IPS (Intrusion Prevention System)
      + URL filtering + Threat intelligence feeds
      + User identity awareness (integrates with AD/LDAP)
      + SSL/TLS inspection (decrypt, inspect, re-encrypt)
    Examples: Palo Alto, Fortinet FortiGate, Cisco Firepower.

  TYPE 6: AI-BASED / ML FIREWALLS (Emerging)
    Uses machine learning to identify abnormal traffic PATTERNS.
    Can detect zero-day attacks that signature-based systems miss.
    Learns what "normal" traffic looks like, alerts on anomalies.
    Augments (not replaces) traditional rule-based firewalls.

  COMPARISON TABLE:
  ┌───────────────────┬────────┬─────────┬─────────┬──────────┐
  │ Feature           │Packet  │Stateful │ NGFW    │ WAF      │
  │                   │Filter  │         │         │          │
  ├───────────────────┼────────┼─────────┼─────────┼──────────┤
  │ IP/Port filtering │ Yes    │ Yes     │ Yes     │ Limited  │
  │ Connection state  │ No     │ Yes     │ Yes     │ Yes      │
  │ Deep inspection   │ No     │ No      │ Yes     │ Yes      │
  │ App awareness     │ No     │ No      │ Yes     │ Web only │
  │ IPS integration   │ No     │ No      │ Yes     │ No       │
  │ SSL inspection    │ No     │ No      │ Yes     │ Yes      │
  │ Performance       │ Fast   │ Good    │ Medium  │ Medium   │
  │ Cost              │ Low    │ Medium  │ High    │ Medium   │
  └───────────────────┴────────┴─────────┴─────────┴──────────┘
```

---

# 27. The Linux Firewall Stack — Netfilter Architecture

```
THE LINUX FIREWALL STACK — HOW IT ALL FITS TOGETHER:

  In Linux, ALL firewalling is powered by NETFILTER, a kernel framework.
  Tools like iptables, nftables, UFW, and firewalld are just
  FRONT-ENDS that configure Netfilter rules.

  ┌──────────────────────────────────────────────────────────────┐
  │                    USER SPACE (Front-ends)                    │
  │                                                              │
  │   ┌─────────┐  ┌──────────┐  ┌──────┐  ┌──────────┐       │
  │   │iptables │  │ nftables │  │ UFW  │  │ firewalld│       │
  │   │(classic)│  │(modern)  │  │(easy)│  │(zones)   │       │
  │   └────┬────┘  └────┬─────┘  └──┬───┘  └────┬─────┘       │
  │        │            │           │            │              │
  │        │     All translate to Netfilter rules              │
  │        └────────────┴───────────┴────────────┘              │
  │                         │                                    │
  ├─────────────────────────┼────────────────────────────────────┤
  │                         │                                    │
  │                    KERNEL SPACE                              │
  │                         │                                    │
  │              ┌──────────┴──────────┐                        │
  │              │    NETFILTER        │                        │
  │              │  (Packet processing │                        │
  │              │   framework)        │                        │
  │              └─────────────────────┘                        │
  │                                                              │
  │   NETFILTER HOOKS (where packets are intercepted):          │
  │                                                              │
  │   PREROUTING ──> ROUTING ──> FORWARD ──> POSTROUTING       │
  │                    │                                         │
  │                    └──> INPUT ──> Local Process              │
  │                                        │                     │
  │                    OUTPUT <─────────────┘                    │
  │                      │                                       │
  │                      └──> POSTROUTING                       │
  │                                                              │
  └──────────────────────────────────────────────────────────────┘

  WHICH FRONT-END TO USE?

  ┌────────────────┬───────────────────────────────────────────────┐
  │ Tool           │ When to Use                                    │
  ├────────────────┼───────────────────────────────────────────────┤
  │ iptables       │ Legacy systems. Still widely used. Many        │
  │                │ tutorials and scripts use iptables syntax.     │
  ├────────────────┼───────────────────────────────────────────────┤
  │ nftables       │ Modern systems. Official successor to iptables.│
  │                │ Better performance, cleaner syntax, atomic     │
  │                │ updates. Default on newer distros.             │
  ├────────────────┼───────────────────────────────────────────────┤
  │ UFW            │ Ubuntu/Debian systems where simplicity is      │
  │                │ priority. Perfect for single servers.          │
  ├────────────────┼───────────────────────────────────────────────┤
  │ firewalld      │ RHEL/CentOS/Fedora. Zone-based management.   │
  │                │ Dynamic rule changes without restart.          │
  └────────────────┴───────────────────────────────────────────────┘
```

---

# 28. iptables — The Classic Workhorse

## 28.1 iptables Structure

```
iptables ARCHITECTURE:

  iptables organizes rules into TABLES, CHAINS, and RULES.

  TABLES (Categories of Processing):
    filter — DEFAULT table. Accept, drop, or reject packets.
    nat    — Network Address Translation. Modify source/dest IPs.
    mangle — Modify packet headers (TTL, TOS, etc.).
    raw    — Bypass connection tracking. For high-performance rules.

  CHAINS (Decision Points within Tables):
    INPUT    — Packets destined FOR this machine
    OUTPUT   — Packets originating FROM this machine
    FORWARD  — Packets passing THROUGH this machine (routing)
    PREROUTING  — Before routing decision (DNAT)
    POSTROUTING — After routing decision (SNAT/Masquerade)

  RULES (Individual Conditions):
    Each rule specifies: Match criteria + Target (action)
    Match: source IP, dest IP, protocol, port, interface, state
    Target: ACCEPT, DROP, REJECT, LOG, DNAT, SNAT, MASQUERADE

iptables COMMAND SYNTAX:

  iptables [-t table] COMMAND CHAIN [match criteria] -j TARGET

  Commands:
    -A = Append rule to end of chain
    -I = Insert rule at position (default: top)
    -D = Delete rule
    -R = Replace rule at position
    -L = List rules
    -F = Flush (delete all rules)
    -P = Set default policy for chain
    -N = Create new custom chain
    -X = Delete custom chain

  Common Match Options:
    -p tcp/udp/icmp      = Protocol
    --dport 80           = Destination port
    --sport 1024:65535   = Source port range
    -s 10.0.0.0/24       = Source IP/network
    -d 192.168.1.5       = Destination IP
    -i eth0              = Input interface
    -o eth0              = Output interface
    -m state --state     = Connection state
    -m limit --limit     = Rate limiting
    -m multiport         = Multiple ports
```

## 28.2 Production iptables Firewall Script

```
#!/bin/bash
# ================================================================
# PRODUCTION IPTABLES FIREWALL — WEB SERVER
# ================================================================
# Save as: /etc/iptables/firewall.sh
# Run: sudo bash /etc/iptables/firewall.sh
# ================================================================

# --- 1. FLUSH ALL EXISTING RULES ---
iptables -F                      # Flush filter table
iptables -t nat -F               # Flush NAT table
iptables -t mangle -F            # Flush mangle table
iptables -X                      # Delete custom chains

# --- 2. SET DEFAULT POLICIES (DENY ALL) ---
iptables -P INPUT DROP           # Drop all incoming by default
iptables -P FORWARD DROP         # Drop all forwarded by default
iptables -P OUTPUT ACCEPT        # Allow all outgoing (adjust for paranoid)

# --- 3. ALLOW LOOPBACK ---
iptables -A INPUT -i lo -j ACCEPT
iptables -A OUTPUT -o lo -j ACCEPT

# --- 4. ALLOW ESTABLISHED/RELATED CONNECTIONS ---
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# --- 5. DROP INVALID PACKETS ---
iptables -A INPUT -m state --state INVALID -j DROP

# --- 6. ANTI-SPOOFING ---
iptables -A INPUT -s 127.0.0.0/8 ! -i lo -j DROP    # Spoofed loopback
iptables -A INPUT -s 0.0.0.0/8 -j DROP               # "This" network
iptables -A INPUT -s 169.254.0.0/16 -j DROP           # APIPA
iptables -A INPUT -s 224.0.0.0/4 -j DROP              # Multicast
iptables -A INPUT -s 240.0.0.0/4 -j DROP              # Reserved

# --- 7. ALLOW SPECIFIC SERVICES ---
# SSH from management network only
iptables -A INPUT -p tcp -s 10.0.0.0/24 --dport 22 -j ACCEPT

# HTTP and HTTPS from anywhere
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# DNS (if running DNS server)
# iptables -A INPUT -p udp --dport 53 -j ACCEPT

# ICMP (ping) with rate limiting
iptables -A INPUT -p icmp --icmp-type echo-request \
  -m limit --limit 1/second --limit-burst 4 -j ACCEPT

# --- 8. BRUTE-FORCE PROTECTION FOR SSH ---
iptables -A INPUT -p tcp --dport 22 -m state --state NEW \
  -m recent --set --name SSH
iptables -A INPUT -p tcp --dport 22 -m state --state NEW \
  -m recent --update --seconds 60 --hitcount 4 --name SSH -j DROP

# --- 9. LOG DROPPED PACKETS (DEBUGGING) ---
iptables -A INPUT -j LOG --log-prefix "IPTABLES-DROPPED: " \
  --log-level 4 --log-uid

# --- 10. FINAL DROP (redundant with policy, but explicit) ---
iptables -A INPUT -j DROP

# --- 11. SAVE RULES ---
# Debian/Ubuntu:
# iptables-save > /etc/iptables/rules.v4
# RHEL/CentOS:
# service iptables save

echo "Firewall rules applied successfully!"
iptables -L -n -v --line-numbers
```

## 28.3 Viewing and Debugging iptables

```
ESSENTIAL iptables DEBUGGING COMMANDS:

  # List all rules with line numbers, verbose, numeric
  sudo iptables -L -n -v --line-numbers

  # -L = List rules
  # -n = Numeric output (do not resolve DNS — faster)
  # -v = Verbose (show packet/byte counters)
  # --line-numbers = Show rule position

  # Example output:
  # Chain INPUT (policy DROP 1523 packets, 91380 bytes)
  # num   pkts bytes target   prot opt in  out  source     dest
  # 1     8234  492K ACCEPT   all  --  lo  *    0.0.0.0/0  0.0.0.0/0
  # 2    45231   27M ACCEPT   all  --  *   *    0.0.0.0/0  0.0.0.0/0  ESTABLISHED
  # 3      892  53K ACCEPT   tcp  --  *   *    10.0.0.0/24 0.0.0.0/0  dpt:22

  # List NAT table rules
  sudo iptables -t nat -L -n -v

  # Show rules in save format (most readable)
  sudo iptables-save

  # Watch rule counters in real-time
  watch -n 1 'iptables -L INPUT -n -v'

  # Check if a specific rule exists (returns 0 if yes, 1 if no)
  sudo iptables -C INPUT -p tcp --dport 22 -j ACCEPT

  # Zero all packet counters (reset)
  sudo iptables -Z

SAVING AND RESTORING RULES:

  # iptables rules are IN MEMORY. They are LOST on reboot unless saved!

  # Save (Debian/Ubuntu):
  iptables-save > /etc/iptables/rules.v4

  # Restore (Debian/Ubuntu):
  iptables-restore < /etc/iptables/rules.v4

  # Install persistence package (auto-restore on boot):
  sudo apt install iptables-persistent

  # Save (RHEL/CentOS):
  iptables-save > /etc/sysconfig/iptables
  sudo service iptables save
```

---

# 29. nftables — The Modern Successor

## 29.1 Why nftables Replaces iptables

```
nftables — THE FUTURE OF LINUX FIREWALLING:

  nftables is the official successor to iptables, in the Linux kernel
  since version 3.13 (2014). Default on Debian 10+, Ubuntu 20.04+,
  RHEL 8+, Fedora 18+.

  KEY ADVANTAGES OVER iptables:
  ┌────────────────────────────────┬──────────────────────────────────┐
  │ Problem with iptables          │ nftables Solution                │
  ├────────────────────────────────┼──────────────────────────────────┤
  │ Separate tools for IPv4/IPv6   │ Single tool: nft handles both   │
  │ (iptables vs ip6tables)        │ using "inet" address family      │
  ├────────────────────────────────┼──────────────────────────────────┤
  │ Code duplication in kernel     │ Single unified kernel framework  │
  ├────────────────────────────────┼──────────────────────────────────┤
  │ Performance issues with large  │ Optimized rule evaluation using  │
  │ rulesets (linear evaluation)   │ maps, sets, and concatenations   │
  ├────────────────────────────────┼──────────────────────────────────┤
  │ No atomic rule updates (gap    │ Atomic rule replacement means    │
  │ during reload = packets drop)  │ zero packet loss during updates  │
  ├────────────────────────────────┼──────────────────────────────────┤
  │ Requires external ipset        │ Built-in sets, maps, and meters  │
  │ for IP sets                    │ with timeouts and dynamic updates│
  └────────────────────────────────┴──────────────────────────────────┘
```

## 29.2 Complete nftables Ruleset

```
#!/usr/sbin/nft -f
# ===========================================================
# nftables Firewall Configuration - Production Web Server
# ===========================================================
# Save as: /etc/nftables.conf
# Apply: sudo nft -f /etc/nftables.conf
# Enable at boot: sudo systemctl enable nftables
# ===========================================================

# Flush all existing rules
flush ruleset

# Define variables
define MGMT_NET = 10.0.0.0/24
define WEB_PORTS = { 80, 443 }
define SSH_PORT  = 22

table inet firewall {

    # Dynamic blocklist with auto-expiring entries
    set blocklist {
        type ipv4_addr
        flags timeout
        elements = {
            45.33.32.156 timeout 24h,
            198.51.100.66 timeout 24h
        }
    }

    # Allowed management hosts
    set mgmt_hosts {
        type ipv4_addr
        elements = { 10.0.0.5, 10.0.0.10, 10.0.0.254 }
    }

    chain input {
        type filter hook input priority 0; policy drop;

        iif lo accept                            # Loopback
        ip saddr @blocklist drop                 # Block listed IPs
        ct state established,related accept      # Established
        ct state invalid drop                    # Invalid packets
        ip saddr @mgmt_hosts tcp dport $SSH_PORT accept  # SSH mgmt
        tcp dport $WEB_PORTS accept              # Web traffic
        icmp type echo-request limit rate 1/second burst 4 packets accept
        icmpv6 type { nd-neighbor-solicit, nd-neighbor-advert,
                      nd-router-solicit, nd-router-advert } accept
        log prefix "NFT-DROPPED: " flags all counter drop
    }

    chain output {
        type filter hook output priority 0; policy accept;
    }

    chain forward {
        type filter hook forward priority 0; policy drop;
    }
}
```

## 29.3 nftables Dynamic Sets and Brute-Force Protection

```
nftables DYNAMIC SETS — AUTOMATED THREAT RESPONSE:

  # Create auto-expiring dynamic blocklist
  sudo nft add set inet firewall dynamic_blocklist \
    '{ type ipv4_addr; flags dynamic,timeout; timeout 1h; }'

  # Auto-block IPs with too many SSH connections (brute-force defence)
  # More than 3 new connections in 60 seconds = blocked for 1 hour
  sudo nft add rule inet firewall input \
    tcp dport 22 ct state new \
    meter ssh_meter '{ ip saddr limit rate over 3/minute }' \
    add @dynamic_blocklist '{ ip saddr timeout 1h }' drop

  # View current dynamic blocklist
  sudo nft list set inet firewall dynamic_blocklist

  # Manually manage entries
  sudo nft add element inet firewall blocklist { 192.168.1.100 timeout 2h }
  sudo nft delete element inet firewall blocklist { 192.168.1.100 }

COMMAND COMPARISON — iptables vs nftables:

  ┌────────────────────────────┬──────────────────────────────────┐
  │ iptables                   │ nftables                         │
  ├────────────────────────────┼──────────────────────────────────┤
  │ iptables -A INPUT -p tcp   │ nft add rule inet fw input       │
  │   --dport 22 -j ACCEPT     │   tcp dport 22 accept            │
  ├────────────────────────────┼──────────────────────────────────┤
  │ iptables -P INPUT DROP     │ chain input { ... policy drop; } │
  ├────────────────────────────┼──────────────────────────────────┤
  │ iptables -L -n -v          │ nft list ruleset                 │
  ├────────────────────────────┼──────────────────────────────────┤
  │ iptables-save > file       │ nft list ruleset > file          │
  │ iptables-restore < file    │ nft -f file                      │
  └────────────────────────────┴──────────────────────────────────┘
```

---

# 30. UFW — Uncomplicated Firewall Deep Dive

```
UFW (Uncomplicated Firewall):

  A user-friendly front-end for iptables/nftables, designed for
  people who prefer simplicity over granular control.
  Default on Ubuntu and many Debian-based distributions.

  KEY PRINCIPLE: UFW does not replace iptables — it generates
  iptables rules behind the scenes. When you type a simple UFW
  command, it translates to complex iptables syntax.

ESSENTIAL UFW COMMANDS:

  # Enable the firewall
  sudo ufw enable

  # Set default policies (always do this FIRST)
  sudo ufw default deny incoming
  sudo ufw default allow outgoing

  # Allow specific services
  sudo ufw allow ssh          # Port 22
  sudo ufw allow http         # Port 80
  sudo ufw allow https        # Port 443

  # Allow specific port and protocol
  sudo ufw allow 8080/tcp
  sudo ufw allow 53/udp

  # Allow from specific IP
  sudo ufw allow from 10.0.0.5

  # Allow from subnet to specific port
  sudo ufw allow from 10.0.0.0/24 to any port 22

  # Deny specific traffic
  sudo ufw deny from 192.168.1.100
  sudo ufw deny in on eth0 to any port 25

  # Delete a rule
  sudo ufw delete allow 8080/tcp
  sudo ufw delete 3              # Delete by rule number

  # View status
  sudo ufw status verbose
  sudo ufw status numbered

  # Disable/reset
  sudo ufw disable
  sudo ufw reset                 # Remove ALL rules

UFW LOGGING:
  sudo ufw logging on
  sudo ufw logging medium        # Options: off, low, medium, high, full
  # Logs: /var/log/ufw.log

UFW APPLICATION PROFILES:
  sudo ufw app list              # Show available profiles
  sudo ufw app info 'Nginx Full' # Show profile details
  sudo ufw allow 'Nginx Full'    # Allow entire profile
```

---

# 31. firewalld — Zone-Based Dynamic Firewall

```
firewalld — DYNAMIC FIREWALL FOR RHEL/CENTOS/FEDORA:

  firewalld uses the concept of ZONES to manage firewall rules.
  Each network interface (or connection) is assigned to a zone,
  and zones have different trust levels.

  KEY ADVANTAGE: Dynamic rule changes WITHOUT service restart.
  Rules can be added/removed at runtime without dropping connections.

PREDEFINED ZONES (from least to most trusted):
  drop      — Drop all incoming, no notification
  block     — Reject all incoming with icmp-host-prohibited
  public    — DEFAULT zone. Untrusted networks.
  external  — NAT/masquerading for external networks
  dmz       — DMZ with limited internal access
  work      — Trusted work network
  home      — Home network
  internal  — Internal network
  trusted   — ALL traffic accepted (dangerous!)

ESSENTIAL firewalld COMMANDS:

  # Check status
  sudo firewall-cmd --state
  sudo firewall-cmd --list-all

  # Get/set default zone
  sudo firewall-cmd --get-default-zone
  sudo firewall-cmd --set-default-zone=public

  # Allow services (temporary until reload)
  sudo firewall-cmd --add-service=http
  sudo firewall-cmd --add-service=https

  # Make changes permanent
  sudo firewall-cmd --permanent --add-service=http
  sudo firewall-cmd --reload

  # Allow specific port
  sudo firewall-cmd --permanent --add-port=8080/tcp

  # Rich rules (advanced)
  sudo firewall-cmd --permanent --add-rich-rule='
    rule family="ipv4"
    source address="10.0.0.0/24"
    port protocol="tcp" port="22"
    accept'

  # Block an IP
  sudo firewall-cmd --permanent --add-rich-rule='
    rule family="ipv4"
    source address="192.168.1.100"
    drop'

  # List all zones and services
  sudo firewall-cmd --get-zones
  sudo firewall-cmd --get-services
```

---

# 32. Windows Firewall — Defender Firewall with Advanced Security

```
WINDOWS DEFENDER FIREWALL WITH ADVANCED SECURITY:

  Built into all modern Windows versions.
  THREE PROFILES (applied based on network type):

    Domain  — Applied when connected to Active Directory domain
    Private — Applied on trusted home/work networks
    Public  — Applied on untrusted public networks (most restrictive)

  Default Behavior:
    Inbound:  BLOCK all (except allowed rules)
    Outbound: ALLOW all (can be changed to restrictive)

POWERSHELL FIREWALL COMMANDS:

  # Check all profiles
  Get-NetFirewallProfile

  # Set default inbound to block for all profiles
  Set-NetFirewallProfile -Profile Domain,Public,Private `
    -DefaultInboundAction Block

  # Create allow rule for HTTP
  New-NetFirewallRule -DisplayName "Allow HTTP" `
    -Direction Inbound -Protocol TCP -LocalPort 80 -Action Allow

  # Create allow rule for HTTPS
  New-NetFirewallRule -DisplayName "Allow HTTPS" `
    -Direction Inbound -Protocol TCP -LocalPort 443 -Action Allow

  # Block SMBv1 (CRITICAL — prevents EternalBlue)
  New-NetFirewallRule -DisplayName "Block SMBv1 Inbound" `
    -Direction Inbound -Protocol TCP -LocalPort 445 `
    -Action Block -Profile Any

  # Allow RDP from specific subnet only
  New-NetFirewallRule -DisplayName "Allow RDP from Admin Net" `
    -Direction Inbound -Protocol TCP -LocalPort 3389 `
    -RemoteAddress 10.0.0.0/24 -Action Allow

  # List all rules
  Get-NetFirewallRule | Format-Table Name,Enabled,Direction,Action

  # Export/import rules
  netsh advfirewall export "C:\firewall-backup.wfw"
  netsh advfirewall import "C:\firewall-backup.wfw"

  NETSH COMMANDS (Legacy but still useful):
  netsh advfirewall set allprofiles state on
  netsh advfirewall firewall show rule name=all
  netsh advfirewall set allprofiles firewallpolicy blockinbound,allowoutbound
```

---

# 33. pfSense & OPNsense — Dedicated Firewall Appliances

```
pfSense AND OPNsense — OPEN-SOURCE FIREWALL APPLIANCES:

  pfSense:
    FreeBSD-based open-source firewall/router distribution.
    Web-based management interface (GUI).
    Enterprise-grade features in a free package.
    Used by small businesses to large enterprises.

  OPNsense:
    Fork of pfSense with focus on code quality and security.
    More frequent updates and modern UI.
    HardenedBSD base (security-focused FreeBSD fork).

FEATURES (both platforms):
  [x] Stateful packet filtering with rule management via GUI
  [x] NAT/PAT with port forwarding and 1:1 NAT
  [x] VPN — OpenVPN, IPsec, WireGuard
  [x] Multi-WAN with failover and load balancing
  [x] DHCP server and relay
  [x] DNS resolver (Unbound) and forwarder
  [x] Traffic shaping and QoS
  [x] Intrusion Detection/Prevention (Snort/Suricata)
  [x] Web proxy with content filtering (Squid)
  [x] Captive portal for guest networks
  [x] VLAN support for network segmentation
  [x] High Availability (CARP) for failover
  [x] Extensive logging and reporting

WHEN TO USE:
  - Small-to-medium businesses needing enterprise firewall features
  - Home labs and learning environments
  - Network perimeter protection
  - When budget does not allow commercial NGFW solutions
  - As a dedicated VPN concentrator

DEPLOYMENT OPTIONS:
  - Dedicated hardware (Netgate appliances for pfSense)
  - Virtual machine (VMware, Proxmox, Hyper-V)
  - Cloud deployment (AWS, Azure marketplace)
```

---

# 34. Cloud Firewalls — AWS, Azure, and GCP

```
CLOUD FIREWALLS — SECURITY IN THE CLOUD:

  Cloud providers implement firewall functionality differently
  than traditional on-premise firewalls. Key concept: cloud firewalls
  are software-defined and API-driven.

AWS (Amazon Web Services):

  SECURITY GROUPS (Instance-Level):
    - Applied directly to EC2 instances (virtual servers)
    - STATEFUL — Return traffic automatically allowed
    - Default: Deny all inbound, allow all outbound
    - Rules are ALLOW only (no explicit deny rules)
    - Apply to instance regardless of subnet

    Example:
      Allow SSH (22) from 10.0.0.0/24
      Allow HTTP (80) from 0.0.0.0/0
      Allow HTTPS (443) from 0.0.0.0/0

  NETWORK ACLs (Subnet-Level):
    - Applied to entire subnets
    - STATELESS — Must explicitly allow return traffic
    - Default: Allow all inbound and outbound
    - Rules support both ALLOW and DENY
    - Processed in order (rule number)

    Think of it: Security Groups = host-based firewall
                 NACLs = network firewall at subnet boundary

AZURE:

  NETWORK SECURITY GROUPS (NSGs):
    - Applied to subnets or individual NICs
    - Both allow and deny rules
    - Priority-based rule evaluation (lower number = higher priority)
    - Default rules: Allow intra-VNet, allow Azure LB, deny all inbound

  AZURE FIREWALL:
    - Managed, cloud-native network firewall service
    - Layer 3-7 filtering with FQDN-based rules
    - Threat intelligence integration
    - Centralized network policy across subscriptions

GCP (Google Cloud Platform):

  VPC FIREWALL RULES:
    - Applied at VPC network level
    - Target instances using network tags or service accounts
    - Priority-based (0-65535, lower = higher priority)
    - Default: Deny all ingress, allow all egress

CLOUD FIREWALL BEST PRACTICES:
  [x] Principle of Least Privilege for all security group rules
  [x] Never use 0.0.0.0/0 for SSH or RDP
  [x] Use separate security groups per role (web, app, database)
  [x] Enable VPC Flow Logs for traffic visibility
  [x] Regularly audit security group rules (stale/over-permissive)
  [x] Use infrastructure as code (Terraform) for consistent rules
```

---

# PART IV — NETWORK SECURITY TOOLS & MONITORING

---

# 35. tcpdump — Command-Line Packet Analysis

## 35.1 What Is tcpdump?

```
tcpdump — THE SYSADMIN'S STETHOSCOPE:

  tcpdump is a command-line packet analyzer for Unix/Linux.
  It captures network packets and displays their contents.
  Essential for network troubleshooting, security analysis,
  and incident response.

  tcpdump uses BPF (Berkeley Packet Filter) syntax for
  filtering packets. Master BPF = master packet analysis.
```

## 35.2 Essential tcpdump Commands

```
BASIC CAPTURE:

  # Capture all traffic on default interface
  sudo tcpdump

  # Capture on specific interface
  sudo tcpdump -i eth0

  # Capture with verbose output
  sudo tcpdump -v        # verbose
  sudo tcpdump -vv       # more verbose
  sudo tcpdump -vvv      # maximum verbose

  # Show packet contents in hex and ASCII
  sudo tcpdump -X
  sudo tcpdump -XX       # Include link-layer header

  # Do NOT resolve hostnames (faster, cleaner output)
  sudo tcpdump -n

  # Do NOT resolve hostnames OR port names
  sudo tcpdump -nn

  # Show timestamps in readable format
  sudo tcpdump -tttt

  # Capture only N packets then stop
  sudo tcpdump -c 100

FILTERING (BPF Syntax):

  # Filter by HOST
  sudo tcpdump host 192.168.1.1
  sudo tcpdump src host 192.168.1.5
  sudo tcpdump dst host 10.0.0.1

  # Filter by NETWORK
  sudo tcpdump net 192.168.1.0/24

  # Filter by PORT
  sudo tcpdump port 80
  sudo tcpdump src port 443
  sudo tcpdump dst port 22
  sudo tcpdump portrange 1-1024

  # Filter by PROTOCOL
  sudo tcpdump tcp
  sudo tcpdump udp
  sudo tcpdump icmp
  sudo tcpdump arp

  # COMBINE filters with AND, OR, NOT
  sudo tcpdump 'tcp and port 80'
  sudo tcpdump 'host 10.0.0.5 and port 22'
  sudo tcpdump 'src 192.168.1.5 and (dst port 80 or dst port 443)'
  sudo tcpdump 'not port 22'             # Exclude SSH
  sudo tcpdump 'not arp and not icmp'     # Exclude ARP and ICMP

SAVING AND READING CAPTURES:

  # Save to file (pcap format — open in Wireshark)
  sudo tcpdump -w capture.pcap

  # Save with rotation (new file every 100MB, keep 10 files)
  sudo tcpdump -w capture.pcap -C 100 -W 10

  # Read from saved file
  sudo tcpdump -r capture.pcap

  # Read and filter
  sudo tcpdump -r capture.pcap 'tcp and port 443'

SECURITY-SPECIFIC tcpdump EXAMPLES:

  # Detect ARP spoofing (watch for duplicate IP-MAC mappings)
  sudo tcpdump -n arp

  # Capture DNS queries (port 53)
  sudo tcpdump -n 'udp port 53'

  # Detect SYN flood (many SYN without SYN-ACK completion)
  sudo tcpdump -n 'tcp[tcpflags] & (tcp-syn) != 0'

  # Capture ONLY SYN packets (port scanning detection)
  sudo tcpdump -n 'tcp[tcpflags] == tcp-syn'

  # Capture failed SSH attempts
  sudo tcpdump -n 'tcp dst port 22 and tcp[tcpflags] & (tcp-rst) != 0'

  # Monitor DHCP traffic
  sudo tcpdump -n 'udp and (port 67 or port 68)'

  # Capture SMTP traffic (email sending)
  sudo tcpdump -n 'tcp port 25'

  # Capture SMB traffic (watch for EternalBlue scanning)
  sudo tcpdump -n 'tcp port 445'

  # Detect ICMP flood
  sudo tcpdump -n 'icmp and icmp[icmptype] == 8'  # Echo requests
```

---

# 36. Wireshark — GUI Packet Analysis

## 36.1 What Is Wireshark?

```
WIRESHARK — THE WORLD'S MOST POPULAR PACKET ANALYZER:

  Wireshark is a GUI-based network protocol analyzer.
  Supports hundreds of protocols with deep inspection.
  Essential for network forensics and security analysis.

  KEY CAPABILITIES:
    - Live packet capture and offline analysis
    - Deep inspection of hundreds of protocols
    - Read pcap files from tcpdump and other tools
    - Apply display filters to isolate specific traffic
    - Follow TCP/UDP streams (reconstruct conversations)
    - Export objects from HTTP traffic (files, images)
    - Color-coded display for quick visual identification
    - Statistics and graphing capabilities
```

## 36.2 Essential Wireshark Display Filters

```
WIRESHARK DISPLAY FILTERS (different syntax from tcpdump):

  PROTOCOL FILTERS:
    http              — Show HTTP traffic only
    dns               — Show DNS traffic only
    tcp               — Show TCP only
    udp               — Show UDP only
    arp               — Show ARP only
    icmp              — Show ICMP only
    ssh               — Show SSH traffic
    tls               — Show TLS traffic
    smb               — Show SMB traffic
    sip               — Show SIP/VoIP signaling

  IP ADDRESS FILTERS:
    ip.addr == 192.168.1.1       — Source or destination
    ip.src == 192.168.1.5        — Source only
    ip.dst == 10.0.0.1           — Destination only
    ip.addr == 10.0.0.0/24       — Subnet

  PORT FILTERS:
    tcp.port == 80               — Source or destination port 80
    tcp.dstport == 443           — Destination port 443
    udp.port == 53               — DNS traffic

  COMBINATION FILTERS:
    http and ip.addr == 192.168.1.5    — HTTP from/to specific IP
    tcp.port == 22 and ip.src == 10.0.0.0/24  — SSH from subnet
    dns and !ip.addr == 8.8.8.8       — DNS not involving 8.8.8.8
    tcp.flags.syn == 1 and tcp.flags.ack == 0  — SYN only (no SYN-ACK)

  SECURITY-FOCUSED FILTERS:
    # SMBv1 detection (EternalBlue vulnerability)
    smb.cmd == 0x72             # SMB negotiate protocol
    smb

    # Possible ARP spoofing detection
    arp.duplicate-address-detected

    # Failed TCP connections (RST packets)
    tcp.flags.reset == 1

    # HTTP clear-text passwords
    http.request.method == "POST"

    # DNS zone transfers (reconnaissance)
    dns.qr == 0 and dns.flags.opcode == 0

    # DHCP traffic
    bootp

    # NTP monlist responses (amplification attack indicator)
    ntp.priv.reqcode == 42

WIRESHARK FEATURES FOR SECURITY ANALYSIS:

  Follow TCP Stream:
    Right-click packet > Follow > TCP Stream
    Shows the complete conversation between two hosts.
    Plaintext protocols (HTTP, FTP, Telnet) reveal credentials!

  Export HTTP Objects:
    File > Export Objects > HTTP
    Extracts files transferred via HTTP (images, downloads, etc.)

  IO Graph:
    Statistics > IO Graph
    Visualize traffic patterns over time. Spikes may indicate attacks.

  Protocol Hierarchy:
    Statistics > Protocol Hierarchy
    Shows breakdown of all protocols seen. Unusual protocols are flags.

  Conversations:
    Statistics > Conversations
    Lists all communication pairs. Large data transfers are suspicious.
```

---

# 37. Network Monitoring Best Practices

```
NETWORK MONITORING — SITUATIONAL AWARENESS:

  WHAT TO MONITOR:
    [x] ALL firewall logs (accepted and denied traffic)
    [x] DNS query logs (detect tunneling, exfiltration)
    [x] DHCP logs (detect rogue servers, starvation)
    [x] Authentication logs (detect brute force, spraying)
    [x] VPN connection logs (detect unauthorized access)
    [x] Bandwidth utilization (detect DDoS, data exfiltration)
    [x] ARP tables (detect spoofing)
    [x] Network device configurations (detect unauthorized changes)

  MONITORING TOOLS:
    SIEM (Security Information and Event Management):
      Splunk, Elastic SIEM, IBM QRadar, Microsoft Sentinel
      Centralize logs, correlate events, generate alerts.

    Network Monitoring:
      Nagios, Zabbix, PRTG Network Monitor, LibreNMS
      Monitor uptime, bandwidth, device health.

    IDS/IPS:
      Snort, Suricata, Zeek (formerly Bro)
      Detect and prevent known attack patterns.

    NetFlow/sFlow:
      Analyze traffic flow data at network layer.
      Identify top talkers, unusual traffic patterns.

  ALERTING:
    High Priority: Multiple failed auth attempts
    High Priority: Traffic to known C2 (Command & Control) IPs
    High Priority: Large data transfer to external IPs at unusual hours
    Medium: New device on network (unknown MAC address)
    Medium: DNS queries to newly registered domains
    Low: Normal traffic baseline deviations
```

---

# 38. Port Knocking — Advanced Access Control

```
PORT KNOCKING — HIDING SERVICES BEHIND A SECRET SEQUENCE:

  Port knocking is a method of opening a HIDDEN port on a firewall
  by sending a specific SEQUENCE of connection attempts to closed
  ports in the correct order.

  ANALOGY: A secret knock on a door. Only those who know the
  pattern can get in. Everyone else sees a locked door.

HOW IT WORKS:

  1. Firewall blocks ALL ports by default (including SSH port 22)
  2. From outside, port scanning shows NO open ports
  3. Authorized user sends connection attempts to ports in sequence:
     First: Port 7000, then Port 8000, then Port 9000
  4. Firewall daemon detects correct sequence
  5. Firewall temporarily opens port 22 for that specific IP
  6. User connects via SSH
  7. After timeout, port 22 is closed again

  ┌──────────────────────────────────────────────────────────┐
  │              PORT KNOCKING SEQUENCE                       │
  │                                                          │
  │  Step 1:  Knock 7000  ──>  [Firewall records knock]    │
  │  Step 2:  Knock 8000  ──>  [Firewall records knock]    │
  │  Step 3:  Knock 9000  ──>  [Sequence complete!]        │
  │  Step 4:  SSH port 22 opens for your IP only            │
  │  Step 5:  Connect via SSH                               │
  │  Step 6:  Port 22 auto-closes after timeout             │
  └──────────────────────────────────────────────────────────┘

IMPLEMENTATION (knockd on Linux):

  # Install
  sudo apt install knockd

  # Configuration (/etc/knockd.conf):
  [options]
      logfile = /var/log/knockd.log
  [openSSH]
      sequence    = 7000,8000,9000
      seq_timeout = 30
      command     = /sbin/iptables -I INPUT -s %IP% -p tcp --dport 22 -j ACCEPT
      tcpflags    = syn
  [closeSSH]
      sequence    = 9000,8000,7000
      seq_timeout = 30
      command     = /sbin/iptables -D INPUT -s %IP% -p tcp --dport 22 -j ACCEPT
      tcpflags    = syn

  # Client knocking:
  knock server.example.com 7000 8000 9000
  ssh user@server.example.com
  # When done, close:
  knock server.example.com 9000 8000 7000

SECURITY CONSIDERATIONS:
  Advantage: Dramatically reduces attack surface (port scanners see nothing)
  Advantage: Zero-day exploits cannot target services behind locked ports
  Limitation: Not a replacement for strong authentication
  Limitation: Knock sequence can be observed via packet capture on same LAN
  Enhancement: Use SPA (Single Packet Authorization) for encrypted knocking
```

---

# PART V — WIRELESS SECURITY

---

# 39. Wireless Standards — 802.11 Family

```
IEEE 802.11 — WIRELESS LAN STANDARDS:

  ┌────────────┬──────────┬────────────┬──────────┬──────────────────┐
  │ Standard   │ Name     │ Frequency  │ Max Speed│ Year             │
  ├────────────┼──────────┼────────────┼──────────┼──────────────────┤
  │ 802.11a    │ Wi-Fi 2  │ 5 GHz      │ 54 Mbps  │ 1999             │
  │ 802.11b    │ Wi-Fi 1  │ 2.4 GHz    │ 11 Mbps  │ 1999             │
  │ 802.11g    │ Wi-Fi 3  │ 2.4 GHz    │ 54 Mbps  │ 2003             │
  │ 802.11n    │ Wi-Fi 4  │ 2.4/5 GHz  │ 600 Mbps │ 2009             │
  │ 802.11ac   │ Wi-Fi 5  │ 5 GHz      │ 6.9 Gbps │ 2013             │
  │ 802.11ax   │ Wi-Fi 6  │ 2.4/5/6 GHz│ 9.6 Gbps │ 2019             │
  │ 802.11ax   │ Wi-Fi 6E │ 6 GHz      │ 9.6 Gbps │ 2020 (6GHz ext) │
  │ 802.11be   │ Wi-Fi 7  │ 2.4/5/6 GHz│ 46 Gbps  │ 2024             │
  └────────────┴──────────┴────────────┴──────────┴──────────────────┘

  2.4 GHz vs 5 GHz:
    2.4 GHz: Longer range, more interference, fewer channels, slower
    5 GHz:   Shorter range, less interference, more channels, faster
    6 GHz:   Even more bandwidth, very clean spectrum (Wi-Fi 6E/7)
```

---

# 40. Wireless Encryption — WEP, WPA, WPA2, WPA3

```
WIRELESS ENCRYPTION EVOLUTION:

  WEP (Wired Equivalent Privacy) — 1997:
    COMPLETELY BROKEN. Crackable in minutes.
    Uses RC4 stream cipher with weak 24-bit IV (Initialization Vector).
    IV space is small (16 million) and reuses quickly.
    Tools: aircrack-ng can crack WEP in under 5 minutes.
    STATUS: NEVER USE. If you see WEP: IMMEDIATE security risk.

  WPA (Wi-Fi Protected Access) — 2003:
    Interim fix for WEP. Uses TKIP (Temporal Key Integrity Protocol).
    Still uses RC4 but with key mixing function.
    Better than WEP but TKIP has known weaknesses.
    STATUS: Deprecated. Should not be used.

  WPA2 (Wi-Fi Protected Access 2) — 2004:
    Uses AES-CCMP encryption (much stronger than RC4/TKIP).
    Modes:
      WPA2-Personal (PSK): Pre-shared key (password).
        Vulnerable to offline dictionary attacks if passphrase is weak.
      WPA2-Enterprise (802.1X): RADIUS server authentication.
        Each user has unique credentials. Much more secure.
    KRACK Attack (2017): Key reinstallation attack on handshake.
      Affects all WPA2 implementations. Requires proximity.
      Patched in most systems since 2017. Update your devices.
    STATUS: Acceptable if configured properly. Minimum standard.

  WPA3 (Wi-Fi Protected Access 3) — 2018:
    Uses SAE (Simultaneous Authentication of Equals) handshake.
    Also known as Dragonfly handshake.
    Benefits:
      Forward secrecy — Past sessions cannot be decrypted
        even if key is later compromised
      Protection against offline dictionary attacks —
        SAE prevents capturing handshake for offline cracking
      192-bit security mode for enterprise (WPA3-Enterprise)
      Protected Management Frames (PMF) mandatory
      Easy Connect — QR code based device provisioning
      Enhanced Open — Encrypted public Wi-Fi (OWE)
    STATUS: RECOMMENDED. Use WPA3 wherever supported.

COMPARISON TABLE:
  ┌──────────┬──────────┬────────────┬──────────┬──────────────────┐
  │ Standard │ Cipher   │ Key Mgmt   │ Security │ Use?             │
  ├──────────┼──────────┼────────────┼──────────┼──────────────────┤
  │ WEP      │ RC4      │ Static key │ Broken   │ NEVER            │
  │ WPA      │ TKIP/RC4 │ TKIP       │ Weak     │ No               │
  │ WPA2-PSK │ AES-CCMP │ 4-way HS   │ Good     │ Acceptable       │
  │ WPA2-Ent │ AES-CCMP │ 802.1X     │ Strong   │ Recommended      │
  │ WPA3-PSK │ AES-CCMP │ SAE        │ Very Good│ Recommended      │
  │ WPA3-Ent │ AES-GCM  │ 802.1X+SAE │ Strongest│ Best             │
  └──────────┴──────────┴────────────┴──────────┴──────────────────┘
```

---

# 41. Wireless Attacks and Defenses

```
COMMON WIRELESS ATTACKS:

  1. EVIL TWIN ATTACK:
     Attacker creates a rogue AP with same SSID as legitimate network.
     Victim connects thinking it is the real network.
     Attacker can intercept all traffic (MitM).
     Defense: 802.1X authentication, WIDS, certificate validation.

  2. DEAUTHENTICATION ATTACK:
     Attacker sends deauth frames (802.11 management frames) to
     disconnect clients from legitimate AP.
     Used to force reconnection to evil twin or capture handshakes.
     Defense: WPA3 with Protected Management Frames (PMF/802.11w).

  3. WPA2 HANDSHAKE CAPTURE + OFFLINE CRACK:
     Capture 4-way handshake (deauth then monitor reconnection).
     Attempt offline dictionary/brute force attack against PSK.
     Tools: aircrack-ng, hashcat, hcxdumptool
     Defense: Use strong, long passphrases (20+ chars). Use WPA3.

  4. KARMA/MANA ATTACK:
     Rogue AP responds to ALL probe requests regardless of SSID.
     Device probing for "HomeWiFi" gets response from attacker AP.
     Defense: Disable auto-connect, remove saved networks.

  5. WARDRIVING:
     Driving around with antenna to discover and map wireless networks.
     Tools: Kismet, WiGLE, GPS + Wi-Fi adapter
     Defense: Not directly preventable. Use strong encryption.

  6. MAC FILTERING BYPASS:
     MAC filters are easily bypassed by sniffing a valid MAC
     and spoofing it. MAC filtering is NOT a security control.
     Defense: Use proper authentication (802.1X, WPA3-Enterprise).

WIRELESS SECURITY BEST PRACTICES:
  [x] Use WPA3 or WPA2-Enterprise at minimum
  [x] Enable Protected Management Frames (802.11w/PMF)
  [x] Use strong, unique passphrases (20+ characters)
  [x] Hide SSID (minor obscurity, not real security)
  [x] Segment guest Wi-Fi from corporate network
  [x] Deploy Wireless IDS (WIDS) to detect rogue APs
  [x] Conduct regular wireless site surveys
  [x] Disable WPS (Wi-Fi Protected Setup) — easily brute-forced
  [x] Keep AP firmware updated
  [x] Use certificate-based auth (EAP-TLS) for enterprise
```

---

# PART VI — PROTOCOL EXPLOITS & ATTACK TECHNIQUES

---

# 42. EternalBlue — SMBv1 Exploitation

```
ETERNALBLUE — THE EXPLOIT THAT CHANGED CYBERSECURITY:

  CVE: CVE-2017-0144
  Patch: MS17-010
  Protocol: SMBv1 (Server Message Block version 1)
  Port: 445
  Impact: Unauthenticated Remote Code Execution with SYSTEM privileges.
  Origin: Developed by NSA's Equation Group, leaked by Shadow Brokers
          in April 2017.

WHAT MAKES ETERNALBLUE DEVASTATING:

  1. UNAUTHENTICATED — No credentials needed. No user interaction needed.
  2. REMOTE — Attack over the network. Just need port 445 reachable.
  3. SYSTEM PRIVILEGES — Full control of the target machine.
  4. WORMABLE — Can spread automatically from machine to machine
     with NO human interaction. Self-propagating.

THE TECHNICAL EXPLOIT:

  SMBv1 has a buffer overflow vulnerability in how it handles
  SMB_COM_TRANSACTION2 requests.

  Attack chain:
  1. Attacker sends specially crafted SMBv1 packets to port 445
  2. The SMB server mishandles a Transaction request with a
     large buffer allocation
  3. Buffer overflow corrupts kernel memory structures
  4. Attacker achieves arbitrary code execution in kernel space
  5. Code runs with SYSTEM privileges (highest Windows privilege)
  6. Attacker has COMPLETE control of the target machine
  7. Malware can then spread to OTHER machines on the network
     by scanning for more SMBv1 services on port 445

MAJOR INCIDENTS USING ETERNALBLUE:

  WannaCry (May 2017):
    Ransomware worm that infected 230,000+ computers in 150 countries.
    Hit hospitals (NHS UK), telecom companies, factories.
    Encrypted files, demanded Bitcoin ransom.
    Damage: $4-8 billion estimated.

  NotPetya (June 2017):
    Most destructive cyberattack in history.
    Disguised as ransomware but was actually a WIPER (data destroyer).
    Started via compromised Ukrainian accounting software update.
    Used EternalBlue to spread through networks globally.
    Damage: $10+ billion. Maersk lost $300M. Merck lost $870M.

DEFENSE — ABSOLUTE REQUIREMENTS:
  [x] PATCH MS17-010 on ALL Windows systems immediately
  [x] DISABLE SMBv1 everywhere, no exceptions:
      Disable-WindowsOptionalFeature -Online -FeatureName smb1protocol
  [x] BLOCK port 445 at perimeter firewall (never expose to internet)
  [x] NETWORK SEGMENTATION — Limit lateral movement paths
  [x] IDS/IPS signatures for EternalBlue exploit traffic
  [x] MONITOR for internal port 445 scanning activity
  [x] VULNERABILITY SCANNING — Regular scans for SMBv1
```

---

# 43. ARP Spoofing Deep Dive

```
ARP SPOOFING — THE COMPLETE ATTACK AND DEFENSE:

  Already covered in detail in Section 13.2. Here we expand on
  advanced attack scenarios and detection methods.

ADVANCED ARP ATTACK SCENARIOS:

  SCENARIO 1: DNS HIJACKING VIA ARP SPOOFING
    1. ARP spoof to become MitM between victim and gateway
    2. Intercept DNS queries from victim
    3. Return false DNS answers (e.g., bank.com = attacker IP)
    4. Victim visits fake banking site, enters credentials
    5. Attacker captures credentials and forwards to real site

  SCENARIO 2: SSL STRIPPING VIA ARP SPOOFING
    1. ARP spoof to become MitM
    2. Use sslstrip tool to downgrade HTTPS to HTTP
    3. Victim connects to HTTP version of site
    4. Attacker forwards to HTTPS version (server side)
    5. Credentials visible in plaintext between victim and attacker

  SCENARIO 3: SESSION HIJACKING
    1. ARP spoof to intercept traffic
    2. Capture session cookies from HTTP traffic
    3. Inject stolen cookies into attacker browser
    4. Attacker is now logged in as victim

DETECTION METHODS:

  tcpdump:
    sudo tcpdump -n arp | grep "is-at"
    # Look for multiple "is-at" replies for same IP with different MACs

  arpwatch:
    Daemon that monitors ARP table changes and sends alerts
    sudo apt install arpwatch
    sudo arpwatch -i eth0

  Wireshark:
    Filter: arp.duplicate-address-detected
    Expert Info: Duplicate IP address detected
```

---

# 44. DHCP Attacks — Starvation, Rogue Servers, TunnelVision

```
DHCP ATTACKS — COMPREHENSIVE COVERAGE:

  Covered in detail in Section 12.2. Additional technical details:

TUNNELVISION (CVE-2024-3661) — VPN DECLOAKING:

  This attack manipulates DHCP options to redirect VPN user traffic
  OUTSIDE the encrypted VPN tunnel.

  Pre-conditions: Attacker must have local network access
  (e.g., same coffee shop Wi-Fi, compromised switch).

  Technical Mechanism:
  1. Attacker runs rogue DHCP server or does DHCP option injection
  2. Assigns DHCP Option 121 (classless static routes)
  3. Option 121 adds specific routes that override VPN tunnel
  4. Traffic to targeted destinations goes via attacker gateway
     INSTEAD of through VPN tunnel
  5. VPN client shows "connected" but traffic is NOT encrypted

  This bypasses VPN on Windows, macOS, Linux, iOS.
  Only Android (which ignores Option 121) is unaffected.

  Defense:
  - Use network namespaces (Linux) to isolate VPN traffic
  - Firewall rules to block non-VPN traffic on untrusted networks
  - DHCP snooping to prevent rogue DHCP servers
  - Avoid connecting to untrusted networks for sensitive work
```

---

# 45. NTP Amplification & NTP Relay Attacks

```
NTP ATTACKS — COMPREHENSIVE COVERAGE:

  Covered in detail in Section 14.2. Summary of both attack types:

  NTP AMPLIFICATION:
    DDoS attack using monlist command (100x amplification).
    Small spoofed request generates massive responses to victim.
    Defense: Disable monlist, patch NTP, BCP38, rate limiting.

  NTP RELAY (TIME MANIPULATION):
    ARP spoof or MitM to intercept NTP requests.
    Modify timestamp responses to skew victim system clock.
    Impact: Break TOTP, expire certificates, corrupt logs.
    Defense: NTP authentication, authenticated NTP servers only.
```

---

# 46. LDAP Attacks — Enumeration, Spraying, Honeypot

```
LDAP-BASED ATTACKS — COMPREHENSIVE COVERAGE:

  Covered in detail in Section 11.5. The three phases:

  1. ENUMERATION: Finding valid usernames via timing differences
     or error message variations.
  2. PASSWORD SPRAYING: Low-and-slow brute force (one password
     per account per cycle) to avoid lockout.
  3. HONEYPOT DETECTION: Fake accounts to catch attackers spraying.

  Additional Attacks:
  - LDAP Injection: Similar to SQL injection but targeting LDAP
    queries. Modifying search filters with special characters:
    *)(uid=*))    <-- injected filter modification
  - Anonymous Bind Enumeration: If anonymous access is enabled,
    attacker can enumerate all users, groups, and org structure.

  Defense: Input validation for LDAP queries, disable anonymous
  binds, monitor for LDAP injection patterns, rate limiting.
```

---

# 47. SIP/VoIP Attacks — Eavesdropping, Toll Fraud, Vishing

```
VoIP/SIP ATTACKS — COMPREHENSIVE COVERAGE:

  Covered in detail in Section 19.2. Summary:

  Call Eavesdropping: Capture unencrypted RTP streams via ARP spoofing.
  Toll Fraud: Compromise VoIP system to make expensive calls.
  Vishing: Social engineering via spoofed caller ID.
  SIP INVITE Flooding: DoS against SIP servers.
  Registration Hijacking: Re-register victim contact at SIP registrar.

  Critical Defense: Session Border Controller (SBC), TLS for SIP
  signaling, SRTP for voice media, separate VoIP VLAN, strong auth.
```

---

# 48. DNS Attacks — Cache Poisoning, Tunneling, Amplification

```
DNS ATTACKS — DETAILED COVERAGE:

  DNS CACHE POISONING:
    Inject false DNS records into a resolver cache.
    Result: All users of that resolver are redirected to attacker IP.
    Kaminsky Attack (2008): Exploited predictable transaction IDs.
    Defense: DNSSEC (cryptographic DNS record signing),
    randomized source ports, 0x20 encoding.

  DNS TUNNELING:
    Encode data within DNS queries and responses to exfiltrate data.
    DNS traffic is often allowed through firewalls unfiltered.
    Attacker registers domain (evil.com), runs DNS server.
    Malware on victim encodes stolen data in DNS queries:
      base64data.evil.com  (data hidden in subdomain)
    Attacker DNS server extracts data from queries.
    Defense: DNS query logging and analysis, detect unusually long
    subdomains, high volume of TXT record queries, deep DNS inspection.

  DNS AMPLIFICATION:
    DDoS using open recursive DNS resolvers.
    Small query (50 bytes) generates large response (3000 bytes).
    Amplification factor: 1:60.
    Attacker sends spoofed queries, responses flood victim.
    Defense: Disable open recursion, response rate limiting, BCP38.

  DNS HIJACKING:
    Modify DNS server configuration or registrar settings.
    Redirect domain resolution to attacker-controlled IP.
    Can happen at: host file, local resolver, ISP DNS, registrar.
    Defense: Registrar lock, DNSSEC, monitor DNS record changes.
```

---

# PART VII — FIREWALL ARCHITECTURE & DESIGN

---

# 49. DMZ Architecture — Multi-Zone Design

```
DMZ (Demilitarized Zone) — THE SECURITY BUFFER ZONE:

  A DMZ is a network segment that sits BETWEEN the trusted internal
  network and the untrusted external network (Internet).

  Purpose: Host PUBLIC-FACING servers (web, email, DNS) in a zone
  that is more exposed than internal but still protected by firewalls.

SINGLE FIREWALL DMZ (Three-Legged):

  The firewall has THREE interfaces:

                        THE INTERNET
                             |
                             v
                    ┌─────────────────┐
                    │    FIREWALL      │
                    │  (3 interfaces)  │
                    └─┬─────────┬───┘
                      |         |
           ┌──────────┘         └──────────┐
           v                               v
    ┌──────────────┐              ┌──────────────┐
    │     DMZ       │              │   INTERNAL    │
    │ (Semi-Trusted)│              │  (Trusted)    │
    │               │              │               │
    │ Web Server    │              │ Workstations  │
    │ Email Server  │              │ File Server   │
    │ DNS (public)  │              │ Database      │
    └──────────────┘              └──────────────┘

  Firewall Rules:
    Internet to DMZ:      Allow HTTP/HTTPS/SMTP to DMZ servers
    Internet to Internal: DENY ALL (never allow direct access)
    DMZ to Internal:      Allow only SPECIFIC, NECESSARY connections
                          (e.g., web server to database on port 3306)
    Internal to DMZ:      Allow administrative access (SSH, RDP)
    Internal to Internet: Allow outbound (NAT/PAT)
    DMZ to Internet:      Limited (only what is needed for updates)

DUAL FIREWALL DMZ (Maximum Security):

                        THE INTERNET
                             |
                             v
                    ┌─────────────────┐
                    │  OUTER FIREWALL  │  (Vendor A, e.g., Palo Alto)
                    └────────┬────────┘
                             |
                    ┌────────┴────────┐
                    │       DMZ        │
                    │  Web, Email, DNS │
                    └────────┬────────┘
                             |
                    ┌────────┴────────┐
                    │  INNER FIREWALL  │  (Vendor B, e.g., Fortinet)
                    └────────┬────────┘
                             |
                    ┌────────┴────────┐
                    │    INTERNAL      │
                    │  (Trusted Zone)  │
                    └─────────────────┘

  Using different vendors is intentional:
    If a vulnerability is found in Vendor A firewall, the attacker
    still must bypass Vendor B completely different firewall.
    This is Defense in Depth applied to firewall architecture.
```

---

# 50. Network Segmentation — Microsegmentation & Zero Trust

```
NETWORK SEGMENTATION — LIMITING BLAST RADIUS:

  Without segmentation: One compromised machine can reach everything.
  With segmentation: Attacker is contained within one segment.

  TRADITIONAL SEGMENTATION (VLANs + Firewalls):
    Divide network into zones using VLANs.
    Place firewalls between zones.
    Control traffic between zones with ACLs.
    Example: HR VLAN, Finance VLAN, Server VLAN, Guest VLAN

  MICROSEGMENTATION:
    Take segmentation to the INDIVIDUAL WORKLOAD level.
    Each server, container, or VM has its own security policy.
    Communication between workloads is controlled and monitored.
    Even within the SAME VLAN, traffic is filtered.

    Technologies: VMware NSX, Cisco ACI, Illumio, Guardicore
    Approach: Software-defined security policies, not physical boundaries

  ZERO TRUST ARCHITECTURE:
    "Never trust, always verify."
    Traditional model: Trust internal network, block external.
    Zero Trust: Trust NOTHING regardless of location.

    Principles:
    1. Verify explicitly — Always authenticate and authorize
    2. Use least privilege access — Minimum permissions needed
    3. Assume breach — Design as if attacker is already inside

    Implementation:
    - Identity-based access (user + device + context)
    - MFA for all users (no exceptions)
    - Microsegmentation of all networks
    - Continuous monitoring and validation
    - Encryption everywhere (even internal traffic)
    - No implicit trust based on network location
```

---

# 51. NAT/PAT and Firewall Integration

```
NAT/PAT AND FIREWALL INTEGRATION:

  NAT and firewall functionality are often combined in the same device.
  Understanding how they interact is critical.

  PACKET PROCESSING ORDER (Linux/Netfilter):

    Incoming packet:
    PREROUTING (DNAT first) --> Routing --> FORWARD --> POSTROUTING
                                   |
                                   v
                                 INPUT --> Local Process
                                              |
                                              v
                                           OUTPUT --> POSTROUTING (SNAT)

    DNAT (Destination NAT) happens BEFORE routing decision.
    SNAT (Source NAT) happens AFTER routing decision.
    Firewall filter rules see the TRANSLATED addresses.

  PORT FORWARDING (DNAT + Firewall):

    # Forward incoming port 8080 to internal web server port 80
    iptables -t nat -A PREROUTING -p tcp --dport 8080 \
      -j DNAT --to-destination 10.0.0.5:80

    # Allow the forwarded traffic through the firewall
    iptables -A FORWARD -p tcp -d 10.0.0.5 --dport 80 -j ACCEPT

    Both rules are needed: NAT translates the address,
    but the FORWARD chain firewall rule must PERMIT the traffic.

  MASQUERADING (PAT for dynamic IPs):

    # Enable IP forwarding
    echo 1 > /proc/sys/net/ipv4/ip_forward

    # Masquerade outbound traffic (PAT)
    iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

    # Allow forwarding from internal to external
    iptables -A FORWARD -i eth1 -o eth0 -j ACCEPT
    iptables -A FORWARD -i eth0 -o eth1 -m state \
      --state ESTABLISHED,RELATED -j ACCEPT
```

---

# 52. VPN Integration with Firewalls

```
VPN AND FIREWALL INTEGRATION:

  VPN tunnels must pass through firewalls. Firewall rules must
  accommodate VPN traffic while maintaining security.

  IPsec VPN Requirements:
    Protocol 50 (ESP) — Encapsulated Security Payload
    Protocol 51 (AH)  — Authentication Header
    UDP 500  — IKE (Internet Key Exchange) negotiation
    UDP 4500 — NAT Traversal (IPsec through NAT)

  OpenVPN:
    UDP 1194 (default) or TCP 443 (stealth mode)

  WireGuard:
    UDP 51820 (default)

  FIREWALL RULES FOR VPN:

    # Allow IPsec VPN
    iptables -A INPUT -p udp --dport 500 -j ACCEPT
    iptables -A INPUT -p udp --dport 4500 -j ACCEPT
    iptables -A INPUT -p esp -j ACCEPT

    # Allow OpenVPN
    iptables -A INPUT -p udp --dport 1194 -j ACCEPT

    # Allow WireGuard
    iptables -A INPUT -p udp --dport 51820 -j ACCEPT

  SPLIT TUNNEL VS FULL TUNNEL:

    Split Tunnel:
      Only corporate traffic goes through VPN.
      Internet traffic goes directly (faster).
      Risk: Malware on user device can reach both networks.

    Full Tunnel:
      ALL traffic goes through VPN.
      Internet traffic routed through corporate gateway.
      More secure but slower. Adds load to corporate bandwidth.

    Security Recommendation: Full tunnel for sensitive environments.
    Split tunnel acceptable with strong endpoint protection.
```

---

# 53. VPN vs Proxy — Understanding the Difference

```
VPN vs PROXY — THEY ARE NOT THE SAME:

  ┌────────────────────┬──────────────────────┬───────────────────────┐
  │ Feature            │ VPN                  │ Proxy                 │
  ├────────────────────┼──────────────────────┼───────────────────────┤
  │ Encryption         │ ALL traffic encrypted│ Usually none (unless  │
  │                    │ (full tunnel)        │ HTTPS proxy)          │
  ├────────────────────┼──────────────────────┼───────────────────────┤
  │ Scope              │ ALL applications     │ Usually only web      │
  │                    │ and protocols        │ traffic (browser)     │
  ├────────────────────┼──────────────────────┼───────────────────────┤
  │ Layer              │ Layer 3 (Network)    │ Layer 7 (Application) │
  ├────────────────────┼──────────────────────┼───────────────────────┤
  │ Privacy            │ ISP cannot see any   │ ISP sees you connect  │
  │                    │ traffic content      │ to proxy server       │
  ├────────────────────┼──────────────────────┼───────────────────────┤
  │ Authentication     │ Strong (certs, keys) │ Varies (often weak)   │
  ├────────────────────┼──────────────────────┼───────────────────────┤
  │ Speed Impact       │ Some overhead (crypto)│ Minimal (caching     │
  │                    │                      │ may speed up)         │
  ├────────────────────┼──────────────────────┼───────────────────────┤
  │ Use Case           │ Secure remote access │ Content filtering,    │
  │                    │ to corporate network │ web anonymization     │
  ├────────────────────┼──────────────────────┼───────────────────────┤
  │ IP Masking         │ Yes (exit point IP)  │ Yes (proxy IP)        │
  └────────────────────┴──────────────────────┴───────────────────────┘

  PROXY TYPES:
    Forward Proxy: Client to Proxy to Internet (hide client identity)
    Reverse Proxy: Internet to Proxy to Server (hide server, load balance)
    Transparent Proxy: Intercepts traffic without client configuration
    SOCKS Proxy: Protocol-agnostic, works at Layer 5

  SECURITY NOTE:
    A VPN replaces a proxy for security purposes.
    A proxy does NOT provide the same level of protection as a VPN.
    Free proxy services often log and sell your browsing data.
```

---

# 54. Firewall Policy Management and Best Practices

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

FIREWALL RULE LIFECYCLE:
  Request --> Review --> Approve --> Implement --> Test --> Monitor
  --> Quarterly Review --> Renew/Retire
```

---

# PART VIII — AUDITING, TROUBLESHOOTING & MASTERY

---

# 55. Firewall Auditing — Finding Weaknesses

```
FIREWALL AUDITING — WHAT TO CHECK:

  ┌────────────────────────────────────────────────────────────────┐
  │ AUDITING CHECKLIST:                                           │
  │                                                                │
  │ [x] DEFAULT POLICY: Is it DROP/DENY? If ALLOW = CRITICAL RISK│
  │ [x] ANY-ANY RULES: Are there rules allowing "any" to "any"?  │
  │     Each one is a potential bypass.                            │
  │ [x] OVERLY BROAD RULES: Rules allowing entire subnets when   │
  │     only specific hosts need access.                          │
  │ [x] RULE ORDER: Are specific rules before general rules?     │
  │ [x] STALE RULES: Rules for decommissioned servers/services?  │
  │ [x] UNDOCUMENTED RULES: Rules with no business justification?│
  │ [x] LOGGING: Is logging enabled for denied traffic?          │
  │ [x] MANAGEMENT ACCESS: Is management on isolated network?    │
  │ [x] FIRMWARE: Is firmware up to date? Known CVEs?            │
  │ [x] AUTHENTICATION: MFA for admin access? Strong passwords?  │
  │ [x] OUTBOUND RULES: Are they restrictive or wide open?       │
  │ [x] INBOUND SSH/RDP: Is remote access limited to specific IPs│
  │ [x] SEGMENTATION: Proper zones? DMZ correctly configured?    │
  │ [x] VPN: Split vs. full tunnel? MFA for VPN access?          │
  │ [x] IPv6: Do IPv6 rules mirror IPv4 security policies?       │
  │ [x] CHANGE LOG: Are all rule changes documented and approved?│
  │ [x] BACKUP: Are configurations backed up and tested?         │
  └────────────────────────────────────────────────────────────────┘

AUTOMATED AUDITING TOOLS:
  Nipper — Firewall configuration analysis
  FireMon — Enterprise firewall management and compliance
  AlgoSec — Network security policy management
  Tufin — Security policy orchestration
  Nmap — External port scanning to verify firewall effectiveness
```

---

# 56. Firewall Troubleshooting — Diagnosing Issues

```
SYSTEMATIC FIREWALL TROUBLESHOOTING:

  STEP 1: Verify the problem IS actually the firewall
    - Can you ping the destination? traceroute?
    - Is the service actually running on the target?
    - Is it a DNS issue? Try IP directly.
    - Is it a routing issue? Check route tables.

  STEP 2: Check firewall rules
    iptables -L -v -n --line-numbers   # Linux
    nft list ruleset                    # nftables
    sudo ufw status verbose             # UFW
    sudo firewall-cmd --list-all       # firewalld
    netsh advfirewall firewall show rule name=all  # Windows

  STEP 3: Check the logs
    grep "DROPPED" /var/log/syslog             # Linux
    grep "IPTABLES-DROPPED" /var/log/kern.log
    # Look for the specific packet being blocked

  STEP 4: Test with temporary rule
    # Temporarily allow traffic to confirm firewall is blocking
    iptables -I INPUT 1 -p tcp --dport <port> -j ACCEPT
    # If traffic works now = firewall was blocking
    # REMOVE the temp rule and add proper one

  STEP 5: Verify routing
    ip route show                    # Linux
    route print                      # Windows
    # Is traffic going through the expected path?

  STEP 6: Verify NAT (if applicable)
    iptables -t nat -L -v -n       # Check NAT rules
    # Is DNAT/SNAT configured correctly?

COMMON TROUBLESHOOTING COMMANDS:

  # Test connectivity
  ping <ip>                # ICMP connectivity
  traceroute <ip>          # Path to destination (Linux)
  tracert <ip>             # Path to destination (Windows)

  # Test specific port
  telnet <ip> <port>       # TCP port test
  nc -zv <ip> <port>       # netcat port test (Linux)
  Test-NetConnection -ComputerName <ip> -Port <port>  # PowerShell

  # Capture traffic at firewall
  sudo tcpdump -i eth0 host <ip> and port <port>
```

---

# 57. Common Mistakes — What Gets Networks Compromised

```
THE TOP FIREWALL AND NETWORK MISCONFIGURATIONS:

  1. DEFAULT ALLOW POLICY
     BAD:  Everything allowed unless explicitly denied
     GOOD: Everything denied unless explicitly allowed

  2. ANY-ANY-ANY RULES
     BAD:  iptables -A INPUT -j ACCEPT
     GOOD: iptables -A INPUT -p tcp --dport 443 -j ACCEPT

  3. NOT FILTERING OUTBOUND TRAFFIC
     BAD:  All outbound traffic allowed (malware can phone home)
     GOOD: Whitelist necessary outbound ports (80, 443, 53)

  4. FORGETTING TO LOG
     BAD:  No logging — blind to attacks
     GOOD: Log all denied traffic, alert on anomalies

  5. USING SMBv1
     BAD:  Port 445 open with SMBv1 enabled = EternalBlue target
     GOOD: Disable SMBv1, use SMBv3, block 445 at perimeter

  6. MANAGEMENT ON PUBLIC INTERFACE
     BAD:  Firewall admin panel accessible from Internet
     GOOD: Management on isolated network, MFA required

  7. NOT UPDATING FIRMWARE
     BAD:  Known CVEs in firewall firmware = attacker bypasses FW
     GOOD: Patch firewall firmware as urgently as any server

  8. TRUSTING THE INTERNAL NETWORK
     BAD:  "Castle and moat" — trusted inside, untrusted outside
     GOOD: Zero Trust — verify everything regardless of location

  9. NO RULE REVIEW PROCESS
     BAD:  Rules accumulate over years, never cleaned up
     GOOD: Quarterly reviews, expiration dates on temporary rules

  10. SINGLE POINT OF FAILURE
      BAD:  One firewall, no redundancy = failure means no security
      GOOD: HA (High Availability) pairs, failover configurations

  11. IGNORING IPv6
      BAD:  IPv4 locked down, IPv6 wide open or not configured
      GOOD: Apply equivalent security policies to both IPv4 and IPv6

  12. EXPOSING DATABASE PORTS TO INTERNET
      BAD:  MySQL 3306, PostgreSQL 5432 accessible from internet
      GOOD: Database only accessible from application server subnet

  13. WEAK VPN CONFIGURATION
      BAD:  VPN with password-only auth, no MFA, split tunnel
      GOOD: VPN with MFA, certificate auth, full tunnel, timeouts

  14. NO NETWORK SEGMENTATION
      BAD:  Flat network where all devices can reach all others
      GOOD: VLANs with firewall rules between segments
```

---

# 58. Hands-On Labs

```
LAB 1: Build a Complete iptables Firewall
  Objective: Configure a Linux server with production-ready firewall
  Steps:
    1. Start with a fresh Linux VM (Ubuntu or CentOS)
    2. Write the iptables script from Section 28.2
    3. Test each rule with netcat and nmap from another machine
    4. Verify logging: tail -f /var/log/syslog | grep IPTABLES
    5. Practice saving and restoring rules
    6. Bonus: Add rate limiting for SSH brute-force protection

LAB 2: nftables Migration
  Objective: Convert iptables rules to nftables
  Steps:
    1. Install nftables on a Linux system
    2. Create table, chains, and rules matching the iptables lab
    3. Implement dynamic blocklist with auto-expiring entries
    4. Test brute-force protection with SSH meter
    5. Compare performance and syntax differences

LAB 3: UFW for Quick Deployment
  Objective: Deploy a firewall in 5 minutes using UFW
  Steps:
    1. sudo ufw default deny incoming
    2. sudo ufw default allow outgoing
    3. sudo ufw allow ssh
    4. sudo ufw allow http
    5. sudo ufw allow https
    6. sudo ufw enable
    7. sudo ufw status verbose

LAB 4: Windows Firewall Hardening
  Objective: Harden Windows using PowerShell firewall commands
  Steps:
    1. Open PowerShell as Administrator
    2. Create rules to allow only HTTP, HTTPS, and RDP
    3. Block all SMBv1 traffic
    4. Test with Test-NetConnection from another machine

LAB 5: ARP Spoofing Detection
  Objective: Detect ARP spoofing using tcpdump and Wireshark
  Steps:
    1. Run tcpdump -n arp on a Linux machine
    2. Observe normal ARP traffic patterns
    3. Use Wireshark filter: arp.duplicate-address-detected
    4. Identify suspicious duplicate MAC-IP mappings

LAB 6: Network Capture and Analysis
  Objective: Capture and analyze network traffic
  Steps:
    1. Start capture: sudo tcpdump -w lab6.pcap -i eth0
    2. Generate traffic (browse, SSH, ping)
    3. Open in Wireshark, apply filters
    4. Follow a TCP stream to see complete conversation
    5. Export HTTP objects found in the capture

LAB 7: Firewall Rule Auditing
  Objective: Audit an existing firewall configuration
  Steps:
    1. Review iptables -L -v -n --line-numbers
    2. Identify overly permissive rules (any-any-any)
    3. Check for stale rules (zero packet counters)
    4. Verify default policy is DROP
    5. Document findings in an audit report

LAB 8: DNS Security Testing
  Objective: Test DNS security configurations
  Steps:
    1. Use dig to query DNS records and test DNSSEC
    2. Check for open recursive DNS servers (amplification risk)
    3. Use tcpdump to capture DNS traffic patterns
    4. Identify anomalous DNS query lengths (tunneling indicator)

LAB 9: VPN Configuration
  Objective: Set up and test VPN with firewall integration
  Steps:
    1. Install OpenVPN or WireGuard on a Linux server
    2. Configure firewall rules to allow VPN traffic
    3. Set up NAT/masquerading for VPN clients
    4. Test split tunnel vs full tunnel configurations
    5. Verify encryption with packet capture
```

---

# 59. Interview Questions — Prove Your Mastery

```
NETWORK SECURITY INTERVIEW QUESTIONS:

FUNDAMENTALS:
  Q: What is the difference between a hub, switch, and router?
  A: Hub = Layer 1, broadcasts to all ports, obsolete.
     Switch = Layer 2, forwards to specific port by MAC address.
     Router = Layer 3, routes between networks by IP address.
     Security: Hub allows trivial sniffing, switch is better but
     vulnerable to MAC flooding, router adds ACL capability.

  Q: Explain the OSI model. Which layer does a firewall operate at?
  A: 7 layers from Physical to Application. Traditional firewalls
     at Layer 3-4. NGFWs at Layer 3-7. WAFs at Layer 7 only.

  Q: What is the difference between TCP and UDP?
  A: TCP is connection-oriented, reliable, ordered (3-way handshake).
     UDP is connectionless, best-effort, unordered but fast.
     TCP for HTTP/SSH/FTP. UDP for DNS/DHCP/VoIP/streaming.

FIREWALL:
  Q: Stateful vs stateless firewall?
  A: Stateless examines each packet independently. Stateful tracks
     connections in a state table — return traffic for established
     connections is automatically allowed.

  Q: Default Deny vs Default Allow?
  A: Default Deny blocks everything not explicitly allowed (secure).
     Default Allow permits everything not explicitly blocked (insecure).
     Every security standard requires Default Deny.

  Q: DROP vs REJECT in iptables?
  A: DROP silently discards — attacker gets no response.
     REJECT discards AND sends error — confirms something exists.
     DROP is more secure for external traffic.

PROTOCOL ATTACKS:
  Q: How does ARP spoofing work? How do you defend?
  A: Attacker sends forged ARP replies to poison caches, redirecting
     traffic through their machine (MitM). Defense: DAI on switches,
     static ARP for critical devices, 802.1X, encryption.

  Q: What is EternalBlue and why is it significant?
  A: Exploits SMBv1 (CVE-2017-0144). Unauthenticated RCE with
     SYSTEM privileges. Wormable. Used in WannaCry/NotPetya.
     Defense: Patch MS17-010, disable SMBv1, block port 445.

  Q: How does DHCP starvation work?
  A: Flood DHCPDISCOVER with spoofed MACs to exhaust IP pool.
     New clients cannot get IPs. Defense: DHCP snooping, port security.

  Q: What is NTP amplification?
  A: DDoS using monlist (100x amplification). Spoofed request
     generates massive response flooding victim.
     Defense: Disable monlist, BCP38, rate limiting.

  Q: Explain DNS tunneling. How do you detect it?
  A: Data exfiltration via DNS queries (hide data in subdomain names).
     Detection: Unusually long subdomains, high volume of TXT queries,
     DNS query analysis tools, deep packet inspection.

ARCHITECTURE:
  Q: What is a DMZ and why is it important?
  A: Network segment between internet and internal network.
     Hosts public-facing servers. If compromised, attacker still
     cannot directly access the internal network.

  Q: NAT vs Firewall?
  A: NAT translates IPs (private to public). Provides obscurity,
     NOT security. Firewall inspects and filters traffic based on
     rules. NAT is NOT a security control by itself.

  Q: What is Zero Trust?
  A: "Never trust, always verify." Every user, device, and connection
     is verified regardless of network location. No implicit trust
     for internal traffic. Identity is the new perimeter.

  Q: What tools would you use to investigate a suspected network breach?
  A: Wireshark/tcpdump for packet capture and analysis. SIEM for log
     correlation. Nmap for network mapping. netstat for active
     connections. Firewall logs for blocked/allowed traffic. DNS logs
     for exfiltration indicators.

SCENARIO-BASED:
  Q: You notice a spike in outbound DNS traffic at 3 AM. What do you do?
  A: 1. Check DNS query logs for unusual patterns (long subdomains,
     high TXT query volume). 2. Identify source hosts. 3. Isolate
     suspicious hosts. 4. Capture traffic with tcpdump. 5. Check for
     DNS tunneling indicators. 6. Analyze with threat intelligence.
     7. Incident response if confirmed.

  Q: Users report slow internet. How do you troubleshoot?
  A: 1. Check bandwidth utilization (is something saturating the link?).
     2. Check firewall for unusual traffic spikes. 3. Run speed tests.
     4. Check for broadcast storms. 5. Look for ARP storm or DHCP
     issues. 6. Check DNS resolution times. 7. Review recent changes.
```

---

# 60. Critical Analysis and Checkpoints

```
CHECKPOINT — TEST YOUR UNDERSTANDING:

  NETWORKING FUNDAMENTALS:
    Can you explain the OSI model and identify which layer
    each protocol operates at?
    Can you calculate subnets and usable hosts from CIDR notation?
    Can you explain the difference between TCP and UDP and
    when each is appropriate?

  PROTOCOL SECURITY:
    Can you explain how ARP spoofing enables Man-in-the-Middle?
    Can you describe the DHCP DORA process and three DHCP attacks?
    Can you explain why NTP amplification works and how to prevent it?
    Can you describe the EternalBlue exploit chain step by step?
    Can you explain password spraying vs brute force vs credential stuffing?

  FIREWALLS:
    Can you write an iptables script from scratch?
    Can you explain stateful vs stateless inspection?
    Can you describe the Netfilter packet processing pipeline?
    Can you compare iptables, nftables, UFW, and firewalld?
    Can you design a DMZ architecture with proper firewall rules?

  TOOLS:
    Can you write tcpdump filters for specific attack detection?
    Can you use Wireshark display filters for security analysis?
    Can you capture and analyze a TCP handshake?

  ARCHITECTURE:
    Can you design a segmented network with proper VLAN and
    firewall policies?
    Can you explain Zero Trust and its implementation principles?
    Can you differentiate between VPN and proxy and when to use each?
```

---

# 61. Comprehensive Network Security Glossary

```
A
──────────────────────────────────────────────────────────────────────
ACL (Access Control List): Rules on a network device defining
  what traffic is permitted or denied.

AES (Advanced Encryption Standard): Symmetric block cipher.
  Key sizes: 128, 192, or 256 bits. Industry standard.

ARP (Address Resolution Protocol): Maps IP to MAC addresses
  on a local network. Vulnerable to spoofing.

B
──────────────────────────────────────────────────────────────────────
BCP 38 (Best Current Practice 38): Network ingress filtering
  to prevent IP address spoofing at the network edge.

BGP (Border Gateway Protocol): Internet routing protocol between
  autonomous systems. Vulnerable to hijacking attacks.

BPF (Berkeley Packet Filter): Filtering language used by tcpdump
  and Wireshark for packet capture and analysis.

C
──────────────────────────────────────────────────────────────────────
CIDR (Classless Inter-Domain Routing): IP addressing using
  prefix notation (e.g., /24) instead of classful addressing.

D
──────────────────────────────────────────────────────────────────────
DAI (Dynamic ARP Inspection): Switch feature validating ARP
  packets against DHCP snooping binding table.

DHCP (Dynamic Host Configuration Protocol): Automatically assigns
  IP addresses and config. Ports: 67/68 (UDP).

DKIM (DomainKeys Identified Mail): Email authentication using
  cryptographic signatures to verify message integrity.

DMARC: Policy telling receivers what to do when SPF/DKIM fail.

DMZ (Demilitarized Zone): Network segment for public-facing
  servers between internet and internal network.

DNSSEC (DNS Security Extensions): Cryptographic signatures
  on DNS records to prevent cache poisoning.

DPI (Deep Packet Inspection): Examining packet data beyond
  headers. Used by NGFWs and IPS systems.

E
──────────────────────────────────────────────────────────────────────
EternalBlue: SMBv1 exploit (CVE-2017-0144). Unauthenticated
  RCE with SYSTEM privileges. Used in WannaCry/NotPetya.

F
──────────────────────────────────────────────────────────────────────
Firewall: Security device monitoring/controlling traffic.
  Types: packet filter, stateful, NGFW, WAF.

G
──────────────────────────────────────────────────────────────────────
Gateway: Device connecting different networks, typically the
  router providing the path to the internet.

H
──────────────────────────────────────────────────────────────────────
HSTS (HTTP Strict Transport Security): Header forcing browsers
  to use HTTPS only. Prevents SSL stripping.

I
──────────────────────────────────────────────────────────────────────
IDS (Intrusion Detection System): Monitors for malicious activity.
  Detection only (alerts). Examples: Snort, Suricata, Zeek.

IPS (Intrusion Prevention System): Monitors AND blocks malicious
  activity inline. Deployed between network segments.

iptables: Linux command-line firewall tool. Front-end to Netfilter.

IPsec: Protocol suite for encrypted IP communication (VPN tunnels).

L
──────────────────────────────────────────────────────────────────────
LDAP (Lightweight Directory Access Protocol): Directory services
  protocol. Ports: 389/636. Used by Active Directory.

M
──────────────────────────────────────────────────────────────────────
MAC Address: 48-bit hardware address (AA:BB:CC:DD:EE:FF).

MFA (Multi-Factor Authentication): Two or more verification
  factors (know, have, are). Essential for security.

N
──────────────────────────────────────────────────────────────────────
NAT (Network Address Translation): Translates private to public
  IPs. Provides obscurity, NOT security.

Netfilter: Linux kernel framework for packet filtering, NAT.

nftables: Modern successor to iptables. Unified framework.

NGFW (Next-Generation Firewall): Combines packet filtering,
  DPI, IPS, application awareness, and threat intelligence.

NTP (Network Time Protocol): Clock sync. Port 123 (UDP).
  Vulnerable to amplification attacks.

O
──────────────────────────────────────────────────────────────────────
OSPF (Open Shortest Path First): Link-state routing protocol.
  Industry standard for interior routing.

OSI Model: 7-layer networking reference model.

P
──────────────────────────────────────────────────────────────────────
PAT (Port Address Translation): Many-to-one NAT using ports.

PMF (Protected Management Frames): 802.11w standard protecting
  Wi-Fi management frames from spoofing.

S
──────────────────────────────────────────────────────────────────────
SAE (Simultaneous Authentication of Equals): WPA3 handshake
  protocol providing forward secrecy.

SBC (Session Border Controller): VoIP firewall.

SIEM (Security Information and Event Management): Centralized
  log collection, correlation, and alerting.

SIP (Session Initiation Protocol): VoIP signaling.
  Ports: 5060/5061.

SMB (Server Message Block): Windows file sharing. Port 445.
  SMBv1 exploited by EternalBlue — DISABLE immediately.

SNMP (Simple Network Management Protocol): Network monitoring.
  Ports: 161/162. v1/v2c insecure — always use SNMPv3.

SPF (Sender Policy Framework): DNS TXT record listing
  authorized email sending servers.

SRTP (Secure Real-time Transport Protocol): Encrypted RTP
  for VoIP media. Essential for call security.

T
──────────────────────────────────────────────────────────────────────
tcpdump: Command-line packet analyzer for Unix/Linux.

TLS (Transport Layer Security): Cryptographic protocol for
  secure communication. Currently TLS 1.3 recommended.

TOTP (Time-based One-Time Password): MFA token that depends
  on accurate system time (NTP security is critical).

U
──────────────────────────────────────────────────────────────────────
UFW (Uncomplicated Firewall): User-friendly iptables front-end.

V
──────────────────────────────────────────────────────────────────────
VLAN (Virtual LAN): Logical network segmentation on switches.

VPN (Virtual Private Network): Encrypted tunnel over public
  network. Types: IPsec, WireGuard, OpenVPN.

W
──────────────────────────────────────────────────────────────────────
WAF (Web Application Firewall): Layer 7 firewall protecting
  web applications from OWASP Top 10 attacks.

WPA3: Latest wireless encryption. SAE handshake, forward
  secrecy, resistant to offline dictionary attacks.

Wireshark: GUI packet analyzer. Hundreds of protocol decoders.

Z
──────────────────────────────────────────────────────────────────────
Zero Trust: Security model. "Never trust, always verify."
  Every connection verified regardless of network location.
```

---

# 62. Final Consolidated Review — The Big Picture

```
THE COMPLETE NETWORK SECURITY PICTURE:

     ┌──────────────────────────────────────────────────────────┐
     │                   THREAT LANDSCAPE                       │
     │   Nation-States  | Criminals | Insiders | Hacktivists    │
     └──────────────────────────────────────────────────────────┘
                                   |
                                   | Attack via:
                                   v
     ┌──────────────────────────────────────────────────────────┐
     │              NETWORK ATTACK VECTORS                      │
     │  ARP Spoofing | DHCP Attacks | SMB Exploits | NTP Amp   │
     │  DNS Hijack   | VoIP Attacks | Wireless Attacks | MitM  │
     └──────────────────────────────────────────────────────────┘
                                   |
                          Targeting:
                                   v
     ┌──────────────────────────────────────────────────────────┐
     │                   YOUR NETWORK                           │
     │  Servers | Workstations | IoT | VoIP | Wireless | Cloud │
     └──────────────────────────────────────────────────────────┘
                                   |
                         Protected by:
                                   v
     ┌──────────────────────────────────────────────────────────┐
     │              DEFENSE IN DEPTH LAYERS                     │
     │                                                          │
     │  Layer 1: Physical Security (locks, cameras, badge)     │
     │  Layer 2: Network Segmentation (VLANs, DMZ)             │
     │  Layer 3: Perimeter Firewalls (NGFW, WAF)               │
     │  Layer 4: Host-Based Firewalls (iptables, Win FW)       │
     │  Layer 5: Protocol Security (LDAPS, SRTP, SNMPv3)      │
     │  Layer 6: Monitoring and Detection (SIEM, IDS/IPS)      │
     │  Layer 7: Encryption (TLS, VPN, WPA3)                   │
     │  Layer 8: Human Layer (Training, Awareness, Policy)     │
     └──────────────────────────────────────────────────────────┘
                                   |
                           Guided by:
                                   v
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
     │  8. DISABLE UNUSED — SMBv1, Telnet, SNMPv1, TFTP       │
     │  9. AUDIT REGULARLY — Quarterly firewall rule reviews   │
     │ 10. ASSUME BREACH — Zero Trust, plan for incident       │
     └──────────────────────────────────────────────────────────┘
```

---

## End of NETWORK_SECURITY_FUNDAMENTALS.md

> **Document Statistics**:
> - Module: NCSCJO — Network Security
> - Version: 2.0 — Enhanced & Expanded Edition
> - Topics covered: 62 major sections, 200+ subsections
> - Coverage: Networking Foundations, OSI/TCP-IP, IPv4/IPv6/Subnetting,
>   Routing, NAT/VLANs, Encapsulation, LDAP, DHCP, ARP, NTP, DNS,
>   SMTP, SNMP, TFTP, SIP/VoIP, SMB, FTP/SFTP, HTTP/HTTPS, SSH,
>   Firewalls (Types, Netfilter, iptables, nftables, UFW, firewalld,
>   Windows Defender, pfSense/OPNsense, Cloud AWS/Azure/GCP),
>   tcpdump, Wireshark, Port Knocking, Network Monitoring,
>   Wireless Security (WEP/WPA/WPA2/WPA3), EternalBlue,
>   ARP Spoofing, DHCP Attacks/TunnelVision, NTP Amplification/Relay,
>   LDAP Attacks (Enumeration/Spraying/Honeypot), SIP/VoIP Attacks,
>   DNS Attacks (Poisoning/Tunneling/Amplification),
>   DMZ Architecture, Network Segmentation/Microsegmentation/Zero Trust,
>   NAT+Firewall Integration, VPN Integration, VPN vs Proxy,
>   Policy Management, Auditing, Troubleshooting, Common Mistakes,
>   Labs, Interview Questions, Glossary


---

# APPENDIX A — EXTENDED PROTOCOL DEEP DIVES

---

## A.1 TCP Deep Dive — The Three-Way Handshake and Beyond

```
TCP (Transmission Control Protocol) — RELIABLE DELIVERY:

  TCP is a CONNECTION-ORIENTED, RELIABLE protocol that ensures
  data arrives in order, complete, and error-free.

THE THREE-WAY HANDSHAKE — HOW CONNECTIONS START:

  ┌──────────┐                              ┌──────────┐
  │  CLIENT  │                              │  SERVER  │
  └────┬─────┘                              └────┬─────┘
       │                                         │
       │──── 1. SYN ──────────────────────────>│
       │     Flags: SYN                          │
       │     Seq: x (client ISN)                 │
       │     "I want to connect."                │
       │                                         │
       │<─── 2. SYN-ACK ────────────────────── │
       │     Flags: SYN, ACK                     │
       │     Seq: y (server ISN)                 │
       │     Ack: x+1                            │
       │     "OK, let us connect."               │
       │                                         │
       │──── 3. ACK ──────────────────────────>│
       │     Flags: ACK                          │
       │     Seq: x+1                            │
       │     Ack: y+1                            │
       │     "Great, connection established."    │
       │                                         │
       │     CONNECTION ESTABLISHED              │
       │     Data transfer can begin             │
       └─────────────────────────────────────────┘

THE FOUR-WAY TEARDOWN — HOW CONNECTIONS END:

  ┌──────────┐                              ┌──────────┐
  │  CLIENT  │                              │  SERVER  │
  └────┬─────┘                              └────┬─────┘
       │                                         │
       │──── 1. FIN ──────────────────────────>│
       │     "I am done sending data."           │
       │                                         │
       │<─── 2. ACK ────────────────────────── │
       │     "I received your FIN."              │
       │                                         │
       │<─── 3. FIN ────────────────────────── │
       │     "I am done too."                    │
       │                                         │
       │──── 4. ACK ──────────────────────────>│
       │     "Connection terminated."            │
       └─────────────────────────────────────────┘

TCP FLAGS AND THEIR SECURITY SIGNIFICANCE:

  ┌─────────┬──────────────────────────────────────────────────────┐
  │ Flag    │ Purpose and Security Relevance                       │
  ├─────────┼──────────────────────────────────────────────────────┤
  │ SYN     │ Initiates connection. SYN FLOOD = DDoS attack.      │
  │         │ Thousands of SYN packets without completing handshake│
  │         │ exhaust server connection table resources.            │
  │         │ Defense: SYN cookies, rate limiting, SYN proxy.      │
  ├─────────┼──────────────────────────────────────────────────────┤
  │ ACK     │ Acknowledgment. ACK SCAN used in reconnaissance.    │
  │         │ Reveals firewall rules (stateful FW drops unsolicited│
  │         │ ACKs, stateless FW may pass them).                   │
  ├─────────┼──────────────────────────────────────────────────────┤
  │ FIN     │ Graceful connection close. FIN SCAN used for stealth │
  │         │ port scanning (no SYN = may bypass some firewall     │
  │         │ rules). Open port: no response. Closed: RST.        │
  ├─────────┼──────────────────────────────────────────────────────┤
  │ RST     │ Reset connection immediately. TCP RST INJECTION:     │
  │         │ Attacker injects RST packet to forcefully disconnect │
  │         │ two communicating parties. Used in censorship.       │
  ├─────────┼──────────────────────────────────────────────────────┤
  │ PSH     │ Push data to application immediately. XMAS SCAN:    │
  │         │ FIN+PSH+URG flags set = "lit up like a Christmas     │
  │         │ tree." Used for OS fingerprinting and stealth scan.  │
  ├─────────┼──────────────────────────────────────────────────────┤
  │ URG     │ Urgent data pointer. Rarely used legitimately.      │
  │         │ Common in attack traffic, used in XMAS scans.       │
  └─────────┴──────────────────────────────────────────────────────┘

TCP SECURITY ATTACKS IN DETAIL:

  SYN FLOOD:
    Attack: Send millions of SYN packets with spoofed source IPs.
    Impact: Server allocates resources for each half-open connection.
    Server connection table fills up. Legitimate connections denied.
    This is one of the most common DDoS techniques.

    Defense mechanisms:
    - SYN Cookies: Server encodes connection state in the ISN,
      does NOT allocate resources until handshake completes.
    - SYN Proxy (firewall): Firewall completes handshake on behalf
      of server, only forwards if client completes handshake.
    - Rate Limiting: Limit SYN packets per source IP per second.
    - Connection Timeouts: Reduce half-open connection timeout.
    - Cloud DDoS Services: Cloudflare, AWS Shield, Akamai.

  TCP SESSION HIJACKING:
    Attack: Attacker predicts or captures TCP sequence numbers
    to inject packets into an active TCP session.
    Can inject commands into an existing authenticated session.
    Defense: Randomized ISN, encryption (TLS), detection tools.

  TCP RST INJECTION (Connection Killing):
    Attack: Send forged RST packet with correct sequence number.
    Victim sees connection drop. Used by nation-state censorship.
    Also used by attackers to disrupt VPN or SSH sessions.
    Defense: Encryption makes it harder to guess sequence numbers.
```

## A.2 UDP Deep Dive — Fast but Risky

```
UDP (User Datagram Protocol):

  Connectionless, unreliable, but FAST. No handshake, no retransmission.

  UDP PACKET STRUCTURE:
    ┌─────────────────────────────────────────────────────────┐
    │ Source Port (16 bits) │ Destination Port (16 bits)       │
    ├─────────────────────────────────────────────────────────┤
    │ Length (16 bits)      │ Checksum (16 bits)               │
    ├─────────────────────────────────────────────────────────┤
    │                    DATA (Payload)                       │
    └─────────────────────────────────────────────────────────┘

    Header is only 8 bytes (vs TCP 20+ bytes). Minimal overhead.

  WHY UDP FOR ATTACKS:

    1. NO CONNECTION STATE:
       Firewalls have difficulty tracking UDP "connections" because
       there is no handshake. Stateful firewalls must use timeouts.

    2. SPOOFABLE:
       No handshake = no verification of source IP.
       Perfect for REFLECTION and AMPLIFICATION DDoS attacks.
       Attacker sends UDP packet with spoofed source IP.
       Reflector sends response to victim.

    3. AMPLIFICATION:
       Some UDP protocols return much MORE data than requested.
       DNS: 60x amplification. NTP: 100x. SSDP: 30x. Memcached: 50,000x!

  UDP SERVICES TO PROTECT:
    DNS (53), DHCP (67/68), NTP (123), SNMP (161/162),
    TFTP (69), SIP (5060), RTP (dynamic), Syslog (514)

  UDP DEFENSE:
    Rate limiting on UDP-heavy ports.
    Source IP validation (BCP 38 / ingress filtering).
    Disable unnecessary UDP services.
    Use encrypted alternatives (DNS over HTTPS, SRTP).
```

## A.3 ICMP Deep Dive — Ping and Beyond

```
ICMP (Internet Control Message Protocol):

  ICMP carries ERROR messages and DIAGNOSTIC information.
  Not a transport protocol — it is a control protocol at Layer 3.

  COMMON ICMP TYPES:
    Type 0  — Echo Reply (ping response)
    Type 3  — Destination Unreachable
      Code 0: Network unreachable
      Code 1: Host unreachable
      Code 3: Port unreachable (important for UDP scanning!)
      Code 4: Fragmentation needed (MTU issues / Path MTU Discovery)
      Code 13: Admin prohibited (firewall)
    Type 5  — Redirect (gateway redirect)
    Type 8  — Echo Request (ping)
    Type 11 — Time Exceeded (TTL expired — used by traceroute)

  ICMP SECURITY THREATS:

    PING FLOOD / SMURF ATTACK:
      Send massive volume of ICMP Echo Requests to overwhelm target.
      Smurf: Spoofed ICMP sent to broadcast address; all hosts reply
      to victim. Amplification via broadcast.
      Defense: Block directed broadcast, rate limit ICMP.

    PING OF DEATH:
      Craft oversized ICMP packet (>65,535 bytes) causing buffer
      overflow on older systems. Largely patched in modern OS.

    ICMP REDIRECT ATTACK:
      Send fake ICMP Redirect to change victim routing table.
      Victim routes traffic through attacker.
      Defense: Disable ICMP redirect acceptance:
        sysctl -w net.ipv4.conf.all.accept_redirects=0

    ICMP TUNNELING:
      Embed covert data in ICMP Echo payload.
      Bypass firewalls that allow ICMP but block other protocols.
      Tools: ptunnel, icmpsh, ICMP Shell
      Defense: Deep packet inspection of ICMP payloads,
      detect abnormally large or frequent ICMP packets.

  FIREWALL ICMP POLICY:
    Allow Echo Reply (outbound pings returning)
    Allow Destination Unreachable (needed for PMTUD)
    Allow Time Exceeded (traceroute functionality)
    Rate limit Echo Requests (prevent flood)
    Block Redirects (prevent routing manipulation)
```

---

## A.4 DHCP Options — Advanced Exploitation Vectors

```
DHCP OPTIONS — BEYOND IP ASSIGNMENT:

  DHCP does not just assign IP addresses. It can configure many
  network parameters via OPTIONS (numbered fields in DHCP messages).

  CRITICAL DHCP OPTIONS:
    Option 1:   Subnet Mask
    Option 3:   Default Gateway (Router)
    Option 6:   DNS Server(s)
    Option 12:  Hostname
    Option 15:  Domain Name
    Option 42:  NTP Server(s)
    Option 43:  Vendor-Specific Information
    Option 51:  IP Address Lease Time
    Option 66:  TFTP Server (PXE Boot)
    Option 67:  Bootfile Name (PXE Boot)
    Option 119: DNS Search Domain List
    Option 121: Classless Static Routes (used in TunnelVision!)
    Option 150: TFTP Server IP (Cisco IP Phones)
    Option 249: Classless Static Routes (Microsoft private)

  EXPLOITATION VIA DHCP OPTIONS:

    ROGUE DNS (Option 6):
      Attacker DHCP server assigns ATTACKER DNS server.
      All DNS queries from victim go to attacker.
      Attacker returns false DNS answers.
      Impact: Phishing, credential theft, malware delivery.

    ROGUE GATEWAY (Option 3):
      Attacker DHCP server assigns ATTACKER as default gateway.
      All victim traffic routed through attacker.
      Impact: Full Man-in-the-Middle.

    TUNNELVISION (Option 121):
      Attacker injects specific routes via Option 121.
      Routes override VPN tunnel routes (more specific wins).
      VPN traffic decloaked — plaintext on local network.
      Impact: Complete VPN bypass for targeted destinations.

    PXE BOOT EXPLOIT (Options 66/67):
      Attacker DHCP assigns malicious PXE server.
      Victim machine boots from attacker-controlled image.
      Impact: Full system compromise from boot.
      Defense: Disable PXE when not actively deploying.

  DHCP SECURITY HARDENING:
    [x] DHCP Snooping on ALL switches
    [x] Port Security to limit MACs per port
    [x] DHCP Guard (Windows) / DHCPGuard (RA Guard for IPv6)
    [x] Separate DHCP servers on monitored, dedicated VLAN
    [x] Monitor DHCP logs for unusual activity
    [x] Lock down PXE boot options when not in use
    [x] Implement 802.1X for network access control
```

---

## A.5 ARP — Extended Security Analysis

```
ARP TABLE MANAGEMENT AND SECURITY:

  VIEWING ARP TABLES:
    Windows: arp -a
    Linux:   ip neigh show  (or legacy: arp -n)
    macOS:   arp -a

  INTERPRETING ARP OUTPUT:
    $ ip neigh show
    192.168.1.1     dev eth0  lladdr aa:bb:cc:dd:ee:ff  REACHABLE
    192.168.1.5     dev eth0  lladdr 11:22:33:44:55:66  STALE
    192.168.1.10    dev eth0  lladdr 77:88:99:00:aa:bb  DELAY
    192.168.1.200   dev eth0                            FAILED

    States:
      REACHABLE — Recently confirmed reachable
      STALE — Not recently confirmed, will re-probe if needed
      DELAY — Waiting for confirmation after probe
      FAILED — No response to ARP request (host down or wrong IP)
      PERMANENT — Manually configured static entry (never expires)

  GRATUITOUS ARP:
    An ARP reply sent without being requested.
    Legitimate uses:
      - Announce IP address after NIC change
      - Update switches after failover (VRRP, CARP)
      - Detect IP conflicts
    Attack use:
      - ARP spoofing / poisoning (send unsolicited fake replies)

    Defense: Rate-limit gratuitous ARPs on switch ports.

  ARP SECURITY CONFIGURATION ON SWITCHES:

    CISCO SWITCH EXAMPLE:
      ! Enable DHCP snooping (prerequisite for DAI)
      ip dhcp snooping
      ip dhcp snooping vlan 10-40

      ! Set trusted port (uplink to DHCP server)
      interface GigabitEthernet0/1
        ip dhcp snooping trust

      ! Enable Dynamic ARP Inspection
      ip arp inspection vlan 10-40

      ! Set trusted port for DAI
      interface GigabitEthernet0/1
        ip arp inspection trust

      ! Rate-limit ARP on untrusted ports (prevent ARP flood)
      interface range GigabitEthernet0/2-24
        ip arp inspection limit rate 15

  ARPTABLES (Linux):
    Linux-specific tool for filtering ARP packets:
    arptables -A INPUT --source-mac ! aa:bb:cc:dd:ee:ff \
      --source-ip 192.168.1.1 -j DROP
```

---

## A.6 DNS Security — Extended Coverage

```
DNS SECURITY — ADVANCED TOPICS:

  DNSSEC IN DETAIL:
    DNSSEC adds cryptographic SIGNATURES to DNS records.
    It does NOT encrypt DNS traffic — it provides INTEGRITY.

    How it works:
    1. Zone owner generates key pair (DNSKEY record)
    2. Signs each DNS record with private key (RRSIG record)
    3. Resolvers verify signatures with public key
    4. Chain of trust from root zone to target domain

    New record types:
    DNSKEY — Public key for verification
    RRSIG  — Signature for a DNS record set
    DS     — Delegation signer (connects parent to child zone)
    NSEC/NSEC3 — Proves non-existence of a record

  DNS OVER HTTPS (DoH) AND DNS OVER TLS (DoT):

    Both ENCRYPT DNS queries to prevent eavesdropping.

    DoT (DNS over TLS):
      Port 853. Dedicated port, easy to block/filter.
      Resolvers: Cloudflare (1.1.1.1), Google (8.8.8.8), Quad9 (9.9.9.9)

    DoH (DNS over HTTPS):
      Port 443. Mixed with regular HTTPS traffic.
      Extremely difficult to block without blocking all HTTPS.
      Privacy benefit: ISP cannot see which domains you visit.
      Security concern: Makes DNS-based security controls harder
      (corporate DNS filtering, parental controls).

  DNS REBINDING ATTACK:
    Sophisticated attack that tricks a browser into making requests
    to internal network resources.
    1. Victim visits attacker DNS with very short TTL
    2. First resolution: attacker IP (serves JavaScript)
    3. TTL expires, browser re-resolves
    4. Second resolution: internal IP (e.g., 192.168.1.1)
    5. Same-origin policy treats both as same domain
    6. JavaScript can now access internal network devices!

    Defense: DNS rebinding protection in resolvers,
    validate Host headers on internal services.

  DNS EXFILTRATION DETECTION INDICATORS:
    - Subdomain length > 50 characters
    - High entropy in subdomain strings (base64/hex encoded data)
    - TXT record queries to unusual/new domains
    - High volume of DNS queries to single external domain
    - DNS queries at unusual times (3 AM corporate network)
    - Queries to domains registered within last 30 days

  # Detecting DNS tunneling with tcpdump:
  sudo tcpdump -i eth0 -n 'udp port 53' | \
    awk '{if(length($0)>200) print $0}'  # Flag long queries
```

---

## A.7 Network Scanning and Reconnaissance

```
NETWORK SCANNING — WHAT ATTACKERS DO FIRST:

  Before attacking, adversaries ENUMERATE the network.
  Understanding these techniques helps defenders detect them.

  NMAP — THE ULTIMATE NETWORK SCANNER:

    # Basic ping sweep (discover live hosts)
    nmap -sn 192.168.1.0/24

    # TCP SYN scan (stealth, most common)
    nmap -sS 192.168.1.1

    # TCP Connect scan (full handshake, detectable)
    nmap -sT 192.168.1.1

    # UDP scan (much slower than TCP)
    nmap -sU 192.168.1.1

    # Service version detection
    nmap -sV 192.168.1.1

    # OS detection
    nmap -O 192.168.1.1

    # Aggressive scan (version + OS + scripts + traceroute)
    nmap -A 192.168.1.1

    # Scan specific ports
    nmap -p 22,80,443,445 192.168.1.1

    # Scan all 65535 ports
    nmap -p- 192.168.1.1

    # Run vulnerability scripts
    nmap --script vuln 192.168.1.1

    # Detect EternalBlue vulnerability specifically
    nmap --script smb-vuln-ms17-010 -p 445 192.168.1.1

  DETECTION:
    Nmap scans generate distinctive traffic patterns:
    - SYN scan: Many SYN packets to different ports, no completion
    - Sequential port access (1, 2, 3, 4...) = obvious scanning
    - Multiple connections from single source to many destinations

    Tools for detection:
    - Snort/Suricata IDS with port scan detection rules
    - portsentry (Linux daemon)
    - psad (Port Scan Attack Detector)
    - Firewall logs analysis (many denied connections from single IP)
```

---

## A.8 Network Hardening Checklist

```
COMPREHENSIVE NETWORK HARDENING CHECKLIST:

  ROUTERS AND SWITCHES:
    [ ] Disable unused ports on all switches
    [ ] Set unused ports to a "black hole" VLAN
    [ ] Enable DHCP snooping on all VLANs
    [ ] Enable Dynamic ARP Inspection (DAI)
    [ ] Enable Port Security (limit MACs per port)
    [ ] Disable DTP (Dynamic Trunking Protocol) on access ports
    [ ] Set all access ports to "switchport mode access"
    [ ] Only manually configure trunk ports as needed
    [ ] Set native VLAN to unused VLAN (not VLAN 1)
    [ ] Disable CDP/LLDP on edge ports (information leakage)
    [ ] Enable 802.1X (NAC) for network access control
    [ ] Set strong passwords on all network devices
    [ ] Use SSH, not Telnet, for management
    [ ] Disable HTTP management, use HTTPS only
    [ ] Place management interfaces on dedicated VLAN
    [ ] Enable logging to centralized syslog/SIEM
    [ ] Implement NTP authentication
    [ ] Use SNMPv3 only (disable v1/v2c)
    [ ] Keep firmware updated
    [ ] Backup configurations regularly

  SERVERS:
    [ ] Enable host-based firewall (iptables/nftables/Windows FW)
    [ ] Default deny inbound policy
    [ ] Allow only required ports
    [ ] Disable SMBv1
    [ ] Disable unnecessary services
    [ ] Enable audit logging
    [ ] SSH: Key-based auth only, disable root login
    [ ] Apply patches promptly
    [ ] Use encrypted protocols (HTTPS, LDAPS, SFTP, SNMPv3)

  PERIMETER:
    [ ] Default deny on perimeter firewall
    [ ] No ANY-ANY rules
    [ ] Filter both inbound AND outbound
    [ ] Block bogon/RFC1918 addresses from internet
    [ ] Enable IDS/IPS with current signatures
    [ ] Deploy WAF for web applications
    [ ] Configure DMZ for public services
    [ ] VPN with MFA for all remote access
    [ ] Never expose management interfaces to internet
    [ ] Enable DDoS protection (rate limiting, cloud services)

  WIRELESS:
    [ ] Use WPA3 or WPA2-Enterprise
    [ ] Strong passphrase (20+ characters)
    [ ] Disable WPS
    [ ] Enable PMF (802.11w)
    [ ] Separate guest network on isolated VLAN
    [ ] Deploy WIDS for rogue AP detection
    [ ] Regular wireless security audits
```

---

## A.9 Incident Response for Network Attacks

```
NETWORK INCIDENT RESPONSE — WHAT TO DO WHEN ATTACKED:

  PHASE 1: PREPARATION (Before any incident)
    - Incident Response Plan documented and tested
    - Contact list (security team, management, legal, ISP)
    - Forensic toolkit ready (live USB, packet capture tools)
    - Baseline of normal network traffic established
    - Log retention configured (min 90 days, ideally 1 year)

  PHASE 2: DETECTION AND ANALYSIS
    Indicators of Compromise (IoC):
    - Unusual outbound traffic volume or destinations
    - Connections to known malicious IPs (threat intelligence)
    - Multiple failed authentication attempts
    - New accounts or privilege escalations
    - ARP table changes or duplicate IP-MAC mappings
    - Unexpected DNS query patterns
    - Firewall rule changes not matching change management records

    Tools for Detection:
    - SIEM alerts and correlation rules
    - IDS/IPS alerts (Snort, Suricata)
    - NetFlow analysis for traffic anomalies
    - Wireshark/tcpdump for deep packet analysis
    - Endpoint Detection and Response (EDR) agents

  PHASE 3: CONTAINMENT
    Short-term Containment:
    - Isolate affected systems (disable switch port or VLAN change)
    - Block attacker IPs on perimeter and host firewalls
    - Disable compromised accounts
    - Capture memory dump before shutdown (volatile evidence!)
    - Preserve all logs and packet captures

    Long-term Containment:
    - Rebuild affected systems from known-good images
    - Reset all potentially compromised credentials
    - Implement additional monitoring on affected segments
    - Temporary firewall rules to restrict lateral movement

  PHASE 4: ERADICATION
    - Remove malware and backdoors
    - Patch exploited vulnerabilities
    - Close the attack vector (disable SMBv1, patch NTP, etc.)
    - Verify eradication with scanning and monitoring

  PHASE 5: RECOVERY
    - Restore systems from clean backups
    - Verify normal operations
    - Continue enhanced monitoring for recurrence
    - Gradually remove containment restrictions

  PHASE 6: LESSONS LEARNED
    - Document timeline and actions taken
    - Identify what worked and what failed
    - Update incident response procedures
    - Implement preventive measures
    - Brief management on impact and improvements
```

---

## A.10 Compliance Frameworks and Network Security

```
COMPLIANCE FRAMEWORKS RELEVANT TO NETWORK SECURITY:

  PCI DSS (Payment Card Industry Data Security Standard):
    Requirement 1: Install and maintain network security controls
    Requirement 2: Secure all system components with proper config
    Network requirements:
    - Firewall between internet and cardholder data environment (CDE)
    - DMZ for public-facing systems
    - Prohibit direct public access to CDE
    - No direct connections between internet and CDE
    - Review firewall rules every 6 months

  NIST Cybersecurity Framework (CSF):
    Identify → Protect → Detect → Respond → Recover
    Network-relevant controls:
    - Asset inventory (know every device on your network)
    - Network segmentation and access control
    - Continuous monitoring and anomaly detection
    - Incident response and recovery planning

  CIS Controls (Center for Internet Security):
    Control 1:  Inventory of Enterprise Assets
    Control 4:  Secure Configuration of Enterprise Assets
    Control 6:  Access Control Management
    Control 8:  Audit Log Management
    Control 9:  Email & Web Browser Protections
    Control 12: Network Infrastructure Management
    Control 13: Network Monitoring and Defense

  ISO 27001:
    Annex A.8: Asset Management
    Annex A.9: Access Control
    Annex A.10: Cryptography
    Annex A.13: Communications Security
      - Network controls and segregation
      - Information transfer policies
```

---

## A.11 IPv4 Header Deep Dive

```
IPv4 HEADER — FIELD-BY-FIELD ANALYSIS:

  ┌─────┬──────┬──────────────────┬──────────────────────────────┐
  │ Ver │ IHL  │ DSCP/ECN (ToS)   │ Total Length                  │
  │ 4b  │ 4b   │ 8 bits           │ 16 bits                       │
  ├─────┴──────┼──────────────────┼──────────────────────────────┤
  │ Identification               │ Flags     │ Fragment Offset   │
  │ 16 bits                      │ 3 bits    │ 13 bits            │
  ├──────────────────────────────┼───────────┴───────────────────┤
  │ TTL (8 bits)                 │ Protocol (8 bits)              │
  ├──────────────────────────────┼───────────────────────────────┤
  │ Header Checksum (16 bits)                                     │
  ├───────────────────────────────────────────────────────────────┤
  │ Source IP Address (32 bits)                                    │
  ├───────────────────────────────────────────────────────────────┤
  │ Destination IP Address (32 bits)                               │
  ├───────────────────────────────────────────────────────────────┤
  │ Options (variable, 0-40 bytes, padded to 32-bit boundary)    │
  └───────────────────────────────────────────────────────────────┘

  SECURITY-RELEVANT FIELDS:

  TTL (Time to Live):
    Decremented by 1 at each router. Packet dropped at TTL=0.
    Prevents packets from looping forever.
    Security: OS fingerprinting — different OSes use different
    default TTLs (Windows=128, Linux=64, Cisco=255).
    Traceroute works by sending packets with incrementing TTLs.

  Protocol:
    Identifies the next-level protocol.
    1 = ICMP, 6 = TCP, 17 = UDP, 50 = ESP (IPsec), 51 = AH (IPsec)
    Used by firewalls to identify and filter by protocol.

  Source IP:
    CAN BE SPOOFED in raw packets. No inherent verification.
    Used in: DDoS reflection, scanning evasion, forged attacks.
    Defense: Ingress filtering (BCP38), egress filtering.

  Fragment Offset:
    Used for IP fragmentation — breaking large packets into smaller ones.
    FRAGMENTATION ATTACKS:
    - Tiny fragment attack: Fragment so TCP header spans two fragments,
      bypassing firewall that only checks first fragment.
    - Overlapping fragment: Fragments overlap, second fragment
      overwrites port information in first. Bypasses packet filters.
    - Teardrop: Overlapping fragments crash vulnerable systems.
    Defense: Reassemble fragments before inspection, drop tiny fragments.

  IP Options:
    Record Route, Timestamp, Source Routing (LSRR/SSRR).
    Source Routing: Sender specifies the routing path.
    EXTREMELY dangerous — allows attacker to control packet path.
    ALWAYS disabled: sysctl -w net.ipv4.conf.all.accept_source_route=0
```

---

## A.12 Linux Network Security Hardening (sysctl)

```
LINUX KERNEL NETWORK SECURITY PARAMETERS:

  # File: /etc/sysctl.d/99-network-security.conf

  # === IP FORWARDING ===
  # Disable unless this is a router/firewall
  net.ipv4.ip_forward = 0
  net.ipv6.conf.all.forwarding = 0

  # === ICMP PROTECTION ===
  # Ignore ICMP redirects (prevent route manipulation)
  net.ipv4.conf.all.accept_redirects = 0
  net.ipv4.conf.default.accept_redirects = 0
  net.ipv6.conf.all.accept_redirects = 0

  # Do not send ICMP redirects
  net.ipv4.conf.all.send_redirects = 0
  net.ipv4.conf.default.send_redirects = 0

  # Ignore broadcast ICMP (prevent Smurf attacks)
  net.ipv4.icmp_echo_ignore_broadcasts = 1

  # Reject bogus ICMP error responses
  net.ipv4.icmp_ignore_bogus_error_responses = 1

  # === SOURCE ROUTING ===
  # Disable source routing (CRITICAL)
  net.ipv4.conf.all.accept_source_route = 0
  net.ipv4.conf.default.accept_source_route = 0
  net.ipv6.conf.all.accept_source_route = 0

  # === ANTI-SPOOFING ===
  # Enable reverse path filtering (BCP38)
  net.ipv4.conf.all.rp_filter = 1
  net.ipv4.conf.default.rp_filter = 1

  # === TCP HARDENING ===
  # Enable SYN cookies (protect against SYN flood)
  net.ipv4.tcp_syncookies = 1

  # Increase SYN backlog
  net.ipv4.tcp_max_syn_backlog = 4096

  # Reduce SYN-ACK retries (faster timeout of half-open)
  net.ipv4.tcp_synack_retries = 2

  # Reduce FIN timeout
  net.ipv4.tcp_fin_timeout = 15

  # Limit orphaned connections
  net.ipv4.tcp_max_orphans = 16384

  # Enable TCP timestamps (aids in RTT calculation, minor privacy leak)
  net.ipv4.tcp_timestamps = 1

  # === IPv6 ===
  # Disable IPv6 if not used (reduces attack surface)
  # net.ipv6.conf.all.disable_ipv6 = 1
  # net.ipv6.conf.default.disable_ipv6 = 1

  # Apply: sudo sysctl -p /etc/sysctl.d/99-network-security.conf
```

---

## A.13 Network Forensics — Capturing Evidence

```
NETWORK FORENSICS — CAPTURING AND PRESERVING EVIDENCE:

  PRINCIPLES:
    1. PRESERVATION — Capture before it is lost (network traffic is ephemeral)
    2. CHAIN OF CUSTODY — Document who handled evidence and when
    3. INTEGRITY — Hash captures immediately (sha256sum capture.pcap)
    4. DOCUMENTATION — Log every action with timestamps

  FULL PACKET CAPTURE (FPC):
    Capture ALL network traffic for later analysis.
    Storage intensive but provides complete evidence.
    Tools: tcpdump, Wireshark, Moloch/Arkime, NetworkMiner

    # Continuous capture with rotation (100MB files, keep 50)
    sudo tcpdump -i eth0 -w /evidence/capture_%Y%m%d_%H%M%S.pcap \
      -G 3600 -C 100 -W 50

  NETFLOW/METADATA COLLECTION:
    Capture flow metadata (IPs, ports, bytes, duration) without payload.
    Much less storage. Good for finding patterns.
    Tools: softflowd, nfsen, ntopng, Zeek (Bro)

  EVIDENCE ANALYSIS:
    After capturing traffic during an incident:

    1. Identify attacker IP(s):
       Wireshark: Statistics > Endpoints, sort by bytes
       tcpdump: Process with tshark for statistics

    2. Reconstruct attack timeline:
       Filter by attacker IP, sort by timestamp
       Follow TCP streams to see exact commands/requests

    3. Extract transferred files:
       Wireshark: File > Export Objects > HTTP/SMB/TFTP
       NetworkMiner: Automatic file extraction from pcap

    4. Identify data exfiltration:
       Look for large outbound transfers to external IPs
       Check DNS queries for tunneling indicators
       Inspect HTTPS certificate details for suspicious domains

    5. Generate hashes:
       sha256sum evidence.pcap > evidence.pcap.sha256
       Document hash and timestamp for chain of custody
```


---

## A.14 Advanced Firewall Techniques

```
ADVANCED FIREWALL TECHNIQUES:

  GEO-BLOCKING (IP Geolocation Filtering):
    Block or allow traffic based on geographic origin.
    Useful for: restricting access to country-specific services,
    blocking traffic from countries where you have no business.

    Implementation (iptables with ipset + GeoIP):
      # Install geoip database and xtables-addons
      # Create ipset with country IP ranges
      ipset create china hash:net
      # Populate from GeoIP database
      # Then block:
      iptables -A INPUT -m set --match-set china src -j DROP

    Cloud implementation:
      AWS WAF — Geographic match conditions
      Cloudflare — Firewall rules with country codes
      Azure Front Door — Geo-filtering rules

    LIMITATIONS:
      VPNs and proxies defeat geo-blocking.
      IP databases are not 100% accurate.
      Use as ADDITIONAL layer, never sole defense.

  RATE LIMITING:
    Restrict the number of connections or packets from a single source.
    Prevents brute force, DDoS, and scanning.

    # iptables: Limit SSH to 3 new connections per minute per IP
    iptables -A INPUT -p tcp --dport 22 -m state --state NEW \
      -m recent --set --name SSH_LIMIT
    iptables -A INPUT -p tcp --dport 22 -m state --state NEW \
      -m recent --update --seconds 60 --hitcount 4 --name SSH_LIMIT \
      -j DROP

    # nftables: Rate limiter with meter
    nft add rule inet fw input tcp dport 22 ct state new \
      meter ssh_meter { ip saddr limit rate 3/minute } accept

    # nginx (web server level):
    limit_req_zone $binary_remote_addr zone=login:10m rate=5r/m;
    location /login {
        limit_req zone=login burst=3 nodelay;
    }

  CONNECTION TRACKING TUNING:
    Stateful firewalls track connections in a connection tracking table.
    Under DDoS, table can fill up, causing all new connections to fail.

    # View current connection tracking stats
    cat /proc/sys/net/netfilter/nf_conntrack_count
    cat /proc/sys/net/netfilter/nf_conntrack_max

    # Increase table size (default is often too small)
    echo 262144 > /proc/sys/net/netfilter/nf_conntrack_max

    # Reduce timeout for established connections (default 432000 = 5 days!)
    echo 86400 > /proc/sys/net/netfilter/nf_conntrack_tcp_timeout_established

    # Reduce timeout for connections in TIME_WAIT
    echo 60 > /proc/sys/net/netfilter/nf_conntrack_tcp_timeout_time_wait

  FAIL2BAN INTEGRATION:
    Fail2ban monitors log files and dynamically adds firewall rules
    to block IPs that show malicious behavior.

    # Install fail2ban
    sudo apt install fail2ban

    # Configuration (/etc/fail2ban/jail.local):
    [sshd]
    enabled = true
    port    = ssh
    filter  = sshd
    logpath = /var/log/auth.log
    maxretry = 3
    bantime  = 3600
    findtime = 600
    banaction = iptables-multiport

    # This will automatically ban IPs after 3 failed SSH logins
    # in 10 minutes, blocking them for 1 hour.

    # View banned IPs:
    sudo fail2ban-client status sshd

    # Unban an IP:
    sudo fail2ban-client set sshd unbanip 192.168.1.100
```

---

## A.15 SSL/TLS Security for Network Professionals

```
TLS (Transport Layer Security) — ENCRYPTION IN TRANSIT:

  TLS protects data in transit between two endpoints.
  Used by: HTTPS, LDAPS, SMTPS, IMAPS, POP3S, FTPS, etc.

  TLS HANDSHAKE (Simplified):

  1. Client Hello:
     Client sends supported TLS versions, cipher suites, random data

  2. Server Hello:
     Server selects TLS version and cipher suite, sends certificate

  3. Certificate Verification:
     Client verifies server certificate chain against trusted CAs

  4. Key Exchange:
     Both sides derive shared session key (Diffie-Hellman or RSA)

  5. Encrypted Communication:
     All subsequent data encrypted with session key

  TLS VERSIONS:
    SSL 2.0 — BROKEN, NEVER USE (1995)
    SSL 3.0 — BROKEN (POODLE attack), NEVER USE (1996)
    TLS 1.0 — DEPRECATED, vulnerabilities (BEAST) (1999)
    TLS 1.1 — DEPRECATED (2006)
    TLS 1.2 — ACCEPTABLE, widely supported (2008)
    TLS 1.3 — RECOMMENDED, fastest and most secure (2018)

  TLS 1.3 IMPROVEMENTS:
    - Reduced handshake from 2 round-trips to 1 (faster)
    - Removed insecure cipher suites (RC4, DES, 3DES, MD5, SHA-1)
    - Mandatory forward secrecy (ECDHE only)
    - 0-RTT resumption (even faster for repeated connections)
    - Simplified cipher suite negotiation

  CIPHER SUITE SELECTION (TLS 1.2):
    GOOD:  TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
    GOOD:  TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
    BAD:   TLS_RSA_WITH_RC4_128_SHA (no forward secrecy, weak cipher)
    BAD:   TLS_RSA_WITH_3DES_EDE_CBC_SHA (weak, slow)

    Key terms:
    ECDHE — Key exchange with forward secrecy
    RSA/ECDSA — Authentication algorithm
    AES_256_GCM — Encryption cipher and mode
    SHA384 — Hash function for integrity

  FORWARD SECRECY:
    Even if server private key is compromised LATER, past sessions
    remain encrypted. Ephemeral keys used per session are discarded.
    Always use ECDHE-based cipher suites for forward secrecy.

  CERTIFICATE PINNING:
    Application remembers (pins) the expected server certificate.
    Prevents MitM even if attacker has valid CA-signed certificate.
    Used in mobile apps and sensitive web applications.

  SSL/TLS INSPECTION (CORPORATE FIREWALLS):
    NGFWs can decrypt TLS traffic for inspection, then re-encrypt.
    Process: Firewall presents its own CA cert to client.
    Client trusts the firewall CA (pushed via Group Policy).
    Firewall decrypts, inspects for malware/policy violations, re-encrypts.
    Privacy concern: Employee traffic is visible to firewall.
    Legal consideration: Inform employees of inspection policy.
```

---

## A.16 802.1X — Network Access Control Deep Dive

```
802.1X — PORT-BASED NETWORK ACCESS CONTROL:

  802.1X is THE standard for network authentication.
  Prevents unauthorized devices from connecting to the network.

  THREE ROLES:

    ┌───────────────┐    ┌──────────────────┐    ┌──────────────┐
    │  SUPPLICANT   │───>│  AUTHENTICATOR   │───>│  AUTHENTICATION│
    │  (Client)     │    │  (Switch/AP)     │    │  SERVER (RADIUS)│
    └───────────────┘    └──────────────────┘    └──────────────┘

    Supplicant:     Software on the device requesting access
    Authenticator:  Network device (switch port or wireless AP)
    Auth Server:    RADIUS server (FreeRADIUS, Microsoft NPS)

  PROCESS:
    1. Device connects to switch port
    2. Port starts in UNAUTHORIZED state (only EAP traffic allowed)
    3. Switch sends EAP Identity Request to device
    4. Device responds with identity (username or certificate)
    5. Switch forwards to RADIUS server
    6. RADIUS authenticates (password, certificate, token)
    7. If successful: Port moves to AUTHORIZED state
       RADIUS may assign VLAN, ACLs based on identity
    8. If failed: Port remains UNAUTHORIZED (no access)

  EAP METHODS (HOW AUTHENTICATION IS DONE):
    EAP-TLS:       Certificate-based (most secure, complex to deploy)
    EAP-TTLS:      TLS tunnel + inner authentication (password or cert)
    PEAP:          Microsoft's tunneled EAP (widely used with MSCHAPv2)
    EAP-FAST:      Cisco's flexible authentication

  BENEFITS:
    [x] Unauthorized devices cannot access the network
    [x] Dynamic VLAN assignment based on user identity
    [x] Guest access can be provided on separate VLAN
    [x] Accountability — know who is on which port
    [x] Posture assessment — check device compliance before access

  CHALLENGES:
    - Complex deployment (certificates, RADIUS, switch configuration)
    - IoT devices and printers that do not support 802.1X
      Solution: MAC Authentication Bypass (MAB) for limited devices
    - User experience on first connection
```

---

## A.17 Network Traffic Analysis Patterns

```
IDENTIFYING MALICIOUS TRAFFIC PATTERNS:

  PATTERN: COMMAND AND CONTROL (C2) BEACONING
    Description: Malware periodically contacts attacker server
    Indicators:
      - Regular interval connections to same external IP
      - Short, encrypted sessions at consistent intervals
      - Traffic to newly registered domains
      - Unusual ports or protocols for external communication
    Detection:
      - Zeek/Bro connection logs with interval analysis
      - DNS query logging for DGA (Domain Generation Algorithm) domains
      - NetFlow analysis for periodic small flows

  PATTERN: DATA EXFILTRATION
    Description: Attacker stealing data from compromised network
    Indicators:
      - Large outbound transfers during off-hours
      - DNS queries with unusually long subdomains (DNS tunneling)
      - HTTP/HTTPS POST requests with large payloads
      - Traffic to cloud storage (Dropbox, Google Drive, Mega)
      - Encrypted traffic on non-standard ports
    Detection:
      - NetFlow analysis: sort by bytes out, filter business hours
      - DLP (Data Loss Prevention) tools
      - DNS query length monitoring
      - Proxy logs analysis

  PATTERN: LATERAL MOVEMENT
    Description: Attacker spreading from one machine to others
    Indicators:
      - Internal scanning (SMB, RDP, SSH from unexpected sources)
      - Pass-the-hash / Pass-the-ticket authentication
      - Remote execution tools (PsExec, WMI, PowerShell Remoting)
      - Port 445 (SMB) connections between workstations
    Detection:
      - Internal IDS/IPS sensors
      - Windows Event logs for remote logins
      - Network segmentation violation alerts

  PATTERN: CRYPTOCURRENCY MINING
    Description: Attacker using compromised systems for mining
    Indicators:
      - High CPU usage on servers
      - Connections to known mining pools (port 3333, 4444, etc.)
      - Stratum protocol traffic
    Detection:
      - Block mining pool IPs/domains
      - Monitor CPU utilization anomalies
      - DNS blacklists for mining domains

  PATTERN: PORT SCANNING
    Description: Reconnaissance to map network services
    Indicators:
      - Many SYN packets to sequential ports from single source
      - Connections to many hosts on same port (horizontal scan)
      - TCP flags anomalies (NULL, FIN, XMAS scans)
    Detection:
      - IDS port scan detection rules
      - Firewall log analysis for repeated denied connections
      - psad or portsentry for real-time detection
```

---

## A.18 Network Security Architecture Patterns

```
ENTERPRISE NETWORK SECURITY ARCHITECTURE:

  TIER 1 — SMALL BUSINESS (Under 50 users):
    ┌──────────────────────────────────────────┐
    │  Internet                                │
    │      │                                    │
    │  [UTM Firewall] (pfSense/FortiGate)     │
    │      │                                    │
    │  [Managed Switch with VLANs]            │
    │    VLAN 10: Staff        (192.168.10.0/24)│
    │    VLAN 20: Servers      (192.168.20.0/24)│
    │    VLAN 30: Guest Wi-Fi  (192.168.30.0/24)│
    │    VLAN 99: Management   (192.168.99.0/24)│
    │                                          │
    │  [Wireless AP] with WPA2/3-Enterprise    │
    └──────────────────────────────────────────┘

  TIER 2 — MEDIUM ENTERPRISE (50-500 users):
    ┌──────────────────────────────────────────┐
    │  [ISP A] ──┐      ┌── [ISP B]           │
    │            └──┬──┘  (Redundant WAN)     │
    │               │                          │
    │  [NGFW Cluster] (Palo Alto HA pair)     │
    │     DMZ: Web/Email/DNS                   │
    │               │                          │
    │  [Core L3 Switch] (Inter-VLAN routing)  │
    │     ├── VLAN 10: Workstations           │
    │     ├── VLAN 20: Servers                │
    │     ├── VLAN 30: VoIP                   │
    │     ├── VLAN 40: IoT                    │
    │     ├── VLAN 50: Guest                  │
    │     └── VLAN 99: Management             │
    │                                          │
    │  [IDS/IPS Sensors] on critical segments │
    │  [SIEM] centralized logging              │
    │  [RADIUS] 802.1X on all ports           │
    │  [VPN Concentrator] for remote access   │
    └──────────────────────────────────────────┘

  TIER 3 — LARGE ENTERPRISE (500+ users):
    Internet Edge → DDoS Mitigation → NGFW Cluster → Core
    Core → Distribution → Access Layer with 802.1X
    Separate:
      - Data Center Segment (microsegmented)
      - Cloud connectivity (VPN or Direct Connect)
      - Remote access VPN with MFA
      - OT/SCADA isolated network (air-gapped or heavily firewalled)
      - Security Operations Center (SOC)
      - SIEM with 24/7 monitoring
      - Multiple IDS/IPS sensors
      - Full packet capture capability
      - Threat intelligence integration
```

---

## A.19 Summary Tables and Quick Reference Cards

```
QUICK REFERENCE — PROTOCOL SECURITY STATUS:

  ┌──────────────────┬──────────────┬──────────────────────────────┐
  │ Insecure Protocol│ Secure Alt.  │ Action Required               │
  ├──────────────────┼──────────────┼──────────────────────────────┤
  │ Telnet (23)      │ SSH (22)     │ Disable Telnet, use SSH      │
  │ FTP (21)         │ SFTP (22)    │ Migrate to SFTP              │
  │ HTTP (80)        │ HTTPS (443)  │ Enforce HSTS, redirect HTTP  │
  │ LDAP (389)       │ LDAPS (636)  │ Enable TLS/StartTLS          │
  │ POP3 (110)       │ POP3S (995)  │ Encrypt email retrieval      │
  │ IMAP (143)       │ IMAPS (993)  │ Encrypt email retrieval      │
  │ SMTP (25)        │ SMTPS (465)  │ Encrypt + SPF/DKIM/DMARC    │
  │ SNMPv1/v2c (161) │ SNMPv3 (161) │ Upgrade immediately          │
  │ SMBv1 (445)      │ SMBv3 (445)  │ Disable SMBv1 everywhere     │
  │ TFTP (69)        │ SFTP (22)    │ Isolate to mgmt VLAN only   │
  │ SIP (5060)       │ SIPS (5061)  │ TLS + SRTP for media        │
  │ DNS (53)         │ DoH/DoT      │ Encrypt DNS queries          │
  │ NTP (123)        │ NTS          │ Authenticated NTP             │
  │ RDP (3389)       │ RDP over VPN │ Never expose directly        │
  │ VNC (5900)       │ VNC over SSH │ Tunnel through SSH           │
  └──────────────────┴──────────────┴──────────────────────────────┘

QUICK REFERENCE — ATTACK AND DEFENSE MATRIX:

  ┌────────────────────┬────────────────────┬───────────────────────┐
  │ Attack             │ Protocol/Layer     │ Primary Defense        │
  ├────────────────────┼────────────────────┼───────────────────────┤
  │ ARP Spoofing       │ ARP / Layer 2      │ DAI, Static ARP       │
  │ DHCP Starvation    │ DHCP / Layer 2-3   │ DHCP Snooping         │
  │ Rogue DHCP         │ DHCP / Layer 2-3   │ DHCP Snooping         │
  │ TunnelVision       │ DHCP / Layer 3     │ Network namespaces    │
  │ MAC Flooding       │ Ethernet / Layer 2 │ Port Security          │
  │ VLAN Hopping       │ 802.1Q / Layer 2   │ Disable DTP, tag all  │
  │ SYN Flood          │ TCP / Layer 4      │ SYN Cookies, rate limit│
  │ TCP RST Injection  │ TCP / Layer 4      │ Encryption (TLS)      │
  │ NTP Amplification  │ NTP / Layer 7      │ Disable monlist       │
  │ DNS Amplification  │ DNS / Layer 7      │ Disable open recursion│
  │ DNS Cache Poison   │ DNS / Layer 7      │ DNSSEC                │
  │ DNS Tunneling      │ DNS / Layer 7      │ Deep DNS inspection   │
  │ BGP Hijacking      │ BGP / Layer 3      │ RPKI                  │
  │ EternalBlue        │ SMBv1 / Layer 7    │ Patch + Disable SMBv1 │
  │ LDAP Spraying      │ LDAP / Layer 7     │ Smart lockout, MFA    │
  │ Password Spray     │ Various / Layer 7  │ MFA, honeypot accounts│
  │ SSL Stripping      │ TLS / Layer 5-6    │ HSTS                  │
  │ Evil Twin           │ 802.11 / Layer 2   │ 802.1X, WIDS          │
  │ WPA2 Crack         │ 802.11 / Layer 2   │ WPA3, strong PSK      │
  │ Email Spoofing     │ SMTP / Layer 7     │ SPF + DKIM + DMARC   │
  │ Port Scanning      │ TCP/UDP / Layer 3-4│ IDS, firewall logging │
  └────────────────────┴────────────────────┴───────────────────────┘

QUICK REFERENCE — FIREWALL TOOL COMPARISON:

  ┌───────────┬──────────────────────────────────────────────────────┐
  │ Tool      │ Best For                                              │
  ├───────────┼──────────────────────────────────────────────────────┤
  │ iptables  │ Legacy Linux. Huge knowledge base. Battle-tested.   │
  │ nftables  │ Modern Linux. Unified IPv4/v6. Better performance.  │
  │ UFW       │ Quick, simple firewall on Ubuntu/Debian servers.    │
  │ firewalld │ RHEL/CentOS with zone-based management.            │
  │ Windows FW│ Windows servers and workstations. PowerShell mgmt.  │
  │ pfSense   │ Dedicated open-source firewall/router appliance.   │
  │ OPNsense  │ Like pfSense but with faster updates, HardenedBSD. │
  │ AWS SG    │ AWS EC2 instance-level firewall (stateful).        │
  │ AWS NACL  │ AWS subnet-level firewall (stateless).              │
  │ Azure NSG │ Azure subnet/NIC-level security group.              │
  │ GCP FW    │ Google Cloud VPC firewall rules.                    │
  └───────────┴──────────────────────────────────────────────────────┘
```
