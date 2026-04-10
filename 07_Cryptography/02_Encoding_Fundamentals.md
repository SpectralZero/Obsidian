# 02 - Encoding Fundamentals: The Complete Guide to Data Representation

---
title: "Encoding Fundamentals: From Binary to Base64"
parent: "[[07_Cryptography]]"
tags:
  - "encoding"
  - "base64"
  - "hex"
  - "ascii"
  - "fundamentals"
created: 2026-02-07
updated: 2026-02-07
difficulty: "Beginner"
---

> **Executive Summary**: Encoding is the foundation of all data representation in computing. Before you can understand encryption, hashing, or any cryptographic concept, you must understand how data is represented. This chapter takes you from the absolute basics of binary and ASCII through hex, Base64, URL encoding, Unicode, and advanced encoding schemes. You'll learn to recognize, decode, and encode data in any format - an essential skill for CTF challenges, malware analysis, web security, and OSCP success.

**What You'll Learn**:
- Binary, decimal, and hexadecimal number systems
- ASCII and Unicode character encoding
- Base64 encoding (how it works, why it exists)
- URL/Percent encoding
- Other encodings (Base32, Base16, ROT13)
- How to identify unknown encodings
- Encoding in security contexts (WAF bypass, obfuscation)
- Practical decoding techniques and tools

**Critical Note**: Encoding is **NOT** encryption! Encoding provides **ZERO** security!

---

## Table of Contents

1. [What is Encoding? (ABSOLUTE BEGINNER)](#1-what-is-encoding)
2. [Number Systems: Binary, Decimal, Hex](#2-number-systems)
3. [ASCII: The Foundation](#3-ascii)
4. [Unicode: Beyond ASCII](#4-unicode)
5. [Hexadecimal Encoding](#5-hexadecimal-encoding)
6. [Base64 Encoding](#6-base64-encoding)
7. [URL/Percent Encoding](#7-url-encoding)
8. [Other Encoding Schemes](#8-other-encodings)
9. [Identifying Unknown Encodings](#9-identifying-encodings)
10. [Multi-Layer Encoding](#10-multi-layer-encoding)
11. [Encoding in Security](#11-encoding-in-security)
12. [Practical Tools](#12-practical-tools)
13. [Hands-On Labs](#13-hands-on-labs)
14. [Common Mistakes](#14-common-mistakes)
15. [Interview Questions](#15-interview-questions)
16. [Quick Reference](#16-quick-reference)

---

## 1. What is Encoding? (ABSOLUTE BEGINNER)

### 1.1 The Simple Explanation

**Encoding** is converting data from one format to another.

Think of it like language translation:
- English: "Hello"
- Spanish: "Hola"
- Binary: "01001000 01100101 01101100 01101100 01101111"

Same meaning, different representation!

### 1.2 Why Encoding Exists

```
+------------------------------------------------------------------+
|                    WHY ENCODING EXISTS                            |
+------------------------------------------------------------------+

Problem 1: COMPUTERS ONLY UNDERSTAND BINARY
┌────────────────────────────────────────────────────────────────┐
│ You type: "A"                                                   │
│ Computer sees: 01000001                                         │
│ Need: A way to convert between human text and binary           │
│ Solution: ASCII/Unicode encoding                                │
└────────────────────────────────────────────────────────────────┘

Problem 2: SOME SYSTEMS ONLY HANDLE TEXT
┌────────────────────────────────────────────────────────────────┐
│ You have: Binary file (image, executable)                      │
│ Email says: "I only accept text characters!"                   │
│ Need: Convert binary to safe text                              │
│ Solution: Base64 encoding                                       │
└────────────────────────────────────────────────────────────────┘

Problem 3: SPECIAL CHARACTERS BREAK THINGS
┌────────────────────────────────────────────────────────────────┐
│ URL: https://example.com/search?q=hello world                  │
│ Problem: Space breaks the URL!                                 │
│ Need: Escape special characters                                │
│ Solution: URL encoding (%20 for space)                         │
└────────────────────────────────────────────────────────────────┘
```

### 1.3 Encoding vs Encryption

**THIS IS CRITICAL - MOST COMMON MISTAKE!**

```
+------------------------------------------------------------------+
|                 ENCODING IS NOT ENCRYPTION!                       |
+------------------------------------------------------------------+

ENCODING:
- Purpose: Format conversion
- Security: NONE
- Reversible: By anyone
- Key required: NO
- Example: Base64("password") = "cGFzc3dvcmQ="
           Anyone can decode this!

ENCRYPTION:
- Purpose: Confidentiality
- Security: Strong (with good algorithm/key)
- Reversible: Only with key
- Key required: YES
- Example: AES(key, "password") = random-looking bytes
           Only key holder can decrypt!

+------------------------------------------------------------------+
|  If someone says "I Base64 encoded the password for security"   |
|  THEY ARE WRONG! Base64 provides ZERO security!                  |
+------------------------------------------------------------------+
```

### 1.4 Common Encoding Types

```
┌─────────────────────────────────────────────────────────────────┐
│                    ENCODING FAMILY TREE                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Character Encoding (Text ↔ Numbers)                            │
│  ├── ASCII (7-bit, English only)                                │
│  ├── Latin-1/ISO-8859-1 (8-bit, Western European)               │
│  └── Unicode                                                     │
│      ├── UTF-8 (variable length, web standard)                  │
│      ├── UTF-16 (2+ bytes, Windows internal)                    │
│      └── UTF-32 (4 bytes, fixed width)                          │
│                                                                  │
│  Binary-to-Text Encoding (Binary ↔ Safe Text)                   │
│  ├── Base64 (6-bit groups, A-Za-z0-9+/)                        │
│  ├── Base32 (5-bit groups, A-Z2-7)                             │
│  ├── Base16/Hex (4-bit groups, 0-9A-F)                         │
│  └── Base58 (Bitcoin, no confusing chars)                      │
│                                                                  │
│  Escape Encoding (Safe for specific contexts)                   │
│  ├── URL/Percent (%XX)                                          │
│  ├── HTML entities (&lt; &gt; &amp;)                            │
│  ├── JavaScript (\xXX, \uXXXX)                                  │
│  └── JSON (\n, \t, \uXXXX)                                      │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Number Systems: Binary, Decimal, Hex

### 2.1 Why Different Number Systems?

Computers use **binary** (base-2) because transistors have two states: on/off.
Humans use **decimal** (base-10) because we have 10 fingers.
Programmers use **hexadecimal** (base-16) as a compact way to represent binary.

### 2.2 Decimal (Base-10)

What you're used to. Digits: 0-9

```
Position values (powers of 10):
  1000   100    10     1
  10³    10²    10¹    10⁰
   │      │      │      │
   │      │      │      └── Ones place
   │      │      └───────── Tens place
   │      └──────────────── Hundreds place
   └─────────────────────── Thousands place

Example: 1234 = 1×1000 + 2×100 + 3×10 + 4×1
```

### 2.3 Binary (Base-2)

Computer's native language. Digits: 0 and 1

```
Position values (powers of 2):
  128    64     32     16     8      4      2      1
  2⁷     2⁶     2⁵     2⁴     2³     2²     2¹     2⁰
   │      │      │      │      │      │      │      │
   1      1      0      0      1      0      0      0

Binary: 11001000
Decimal: 128 + 64 + 8 = 200

Terminology:
- 1 bit = single 0 or 1
- 4 bits = nibble (one hex digit)
- 8 bits = byte (0-255)
```

**Binary to Decimal Conversion**:
```python
# Binary to Decimal
binary = "11001000"
decimal = int(binary, 2)  # 200

# Decimal to Binary
decimal = 200
binary = bin(decimal)  # '0b11001000'

# Manual conversion:
# 11001000
# 1×128 + 1×64 + 0×32 + 0×16 + 1×8 + 0×4 + 0×2 + 0×1
# = 128 + 64 + 8 = 200
```

### 2.4 Hexadecimal (Base-16)

Programmer's shorthand for binary. Digits: 0-9 and A-F

```
Hex digit:  0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F
Decimal:    0  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15
Binary:  0000 0001 0010 0011 0100 0101 0110 0111 1000 1001 1010 1011 1100 1101 1110 1111
```

**Why Hex is Useful**:
Each hex digit = exactly 4 bits (one nibble)
Two hex digits = exactly 1 byte (8 bits)

```
Binary:    1100 1000
Hex:        C    8    = C8 (or 0xC8)
Decimal:   200

Much easier to read "C8" than "11001000"!
```

**Hex to Decimal Conversion**:
```python
# Hex to Decimal
hex_val = "C8"
decimal = int(hex_val, 16)  # 200

# Decimal to Hex
decimal = 200
hex_val = hex(decimal)  # '0xc8'

# Manual conversion:
# C8 = C×16 + 8×1 = 12×16 + 8 = 192 + 8 = 200
```

### 2.5 Conversion Cheat Sheet

```
+------------------------------------------------------------------+
|                    NUMBER CONVERSION TABLE                        |
+------------------------------------------------------------------+

Decimal   Hex     Binary      Character
──────────────────────────────────────────
   0       0     0000 0000       NUL
  32      20     0010 0000      (space)
  48      30     0011 0000        0
  57      39     0011 1001        9
  65      41     0100 0001        A
  90      5A     0101 1010        Z
  97      61     0110 0001        a
 122      7A     0111 1010        z
 127      7F     0111 1111       DEL
 255      FF     1111 1111       ÿ
```

---

## 3. ASCII: The Foundation

### 3.1 What is ASCII?

**ASCII** = American Standard Code for Information Interchange

Maps 128 characters to numbers 0-127 (7 bits):
- 0-31: Control characters (non-printable)
- 32-126: Printable characters
- 127: DEL (delete)

### 3.2 The ASCII Table

```
+------------------------------------------------------------------+
|                    ASCII TABLE (PRINTABLE)                        |
+------------------------------------------------------------------+

Dec Hex Char │ Dec Hex Char │ Dec Hex Char │ Dec Hex Char
─────────────┼──────────────┼──────────────┼─────────────
 32  20  SP  │  56  38   8  │  80  50   P  │ 104  68   h
 33  21   !  │  57  39   9  │  81  51   Q  │ 105  69   i
 34  22   "  │  58  3A   :  │  82  52   R  │ 106  6A   j
 35  23   #  │  59  3B   ;  │  83  53   S  │ 107  6B   k
 36  24   $  │  60  3C   <  │  84  54   T  │ 108  6C   l
 37  25   %  │  61  3D   =  │  85  55   U  │ 109  6D   m
 38  26   &  │  62  3E   >  │  86  56   V  │ 110  6E   n
 39  27   '  │  63  3F   ?  │  87  57   W  │ 111  6F   o
 40  28   (  │  64  40   @  │  88  58   X  │ 112  70   p
 41  29   )  │  65  41   A  │  89  59   Y  │ 113  71   q
 42  2A   *  │  66  42   B  │  90  5A   Z  │ 114  72   r
 43  2B   +  │  67  43   C  │  91  5B   [  │ 115  73   s
 44  2C   ,  │  68  44   D  │  92  5C   \  │ 116  74   t
 45  2D   -  │  69  45   E  │  93  5D   ]  │ 117  75   u
 46  2E   .  │  70  46   F  │  94  5E   ^  │ 118  76   v
 47  2F   /  │  71  47   G  │  95  5F   _  │ 119  77   w
 48  30   0  │  72  48   H  │  96  60   `  │ 120  78   x
 49  31   1  │  73  49   I  │  97  61   a  │ 121  79   y
 50  32   2  │  74  4A   J  │  98  62   b  │ 122  7A   z
 51  33   3  │  75  4B   K  │  99  63   c  │ 123  7B   {
 52  34   4  │  76  4C   L  │ 100  64   d  │ 124  7C   |
 53  35   5  │  77  4D   M  │ 101  65   e  │ 125  7D   }
 54  36   6  │  78  4E   N  │ 102  66   f  │ 126  7E   ~
 55  37   7  │  79  4F   O  │ 103  67   g  │
```

### 3.3 ASCII Patterns to Memorize

```
Important ASCII Values:
─────────────────────────────────────────
'0'-'9' = 48-57 (0x30-0x39)
'A'-'Z' = 65-90 (0x41-0x5A)
'a'-'z' = 97-122 (0x61-0x7A)
Space   = 32 (0x20)
Newline = 10 (0x0A)
Tab     = 9 (0x09)

Useful Relationships:
─────────────────────────────────────────
'a' - 'A' = 32  (lowercase = uppercase + 32)
'1' - 1 = '0'   (digit char - int = '0')
ord('A') = 65
chr(65) = 'A'
```

### 3.4 Python ASCII Operations

```python
# Character to ASCII code
print(ord('A'))  # 65
print(ord('a'))  # 97

# ASCII code to character
print(chr(65))   # 'A'
print(chr(97))   # 'a'

# String to ASCII list
text = "Hello"
codes = [ord(c) for c in text]
print(codes)  # [72, 101, 108, 108, 111]

# ASCII list to string
codes = [72, 101, 108, 108, 111]
text = ''.join([chr(c) for c in codes])
print(text)  # "Hello"

# Case conversion via ASCII
upper = chr(ord('a') - 32)  # 'A'
lower = chr(ord('A') + 32)  # 'a'
```

### 3.5 Control Characters

```
+------------------------------------------------------------------+
|                    ASCII CONTROL CHARACTERS                       |
+------------------------------------------------------------------+

Dec  Hex  Abbr  Name              Common Use
────────────────────────────────────────────────────────────
 0   00   NUL   Null              String terminator (C)
 7   07   BEL   Bell              Terminal beep
 8   08   BS    Backspace         Delete previous char
 9   09   TAB   Horizontal Tab    Indentation
10   0A   LF    Line Feed         Unix newline (\n)
13   0D   CR    Carriage Return   Windows newline part (\r)
27   1B   ESC   Escape            Terminal escape sequences

Windows line ending: CR+LF (0D 0A) = \r\n
Unix line ending:    LF (0A) = \n
Mac (old):          CR (0D) = \r
```

---

## 4. Unicode: Beyond ASCII

### 4.1 Why Unicode?

ASCII only handles English (128 characters). The world has thousands of scripts!

```
ASCII can't represent:
- Chinese: 你好
- Arabic: مرحبا
- Emoji: 😀
- Greek: Ωmega
- Math: ∑∫∂√

Unicode Goal: One encoding for ALL human languages
- Over 150,000 characters defined
- Covers all modern languages + historical scripts + symbols + emoji
```

### 4.2 Unicode Basics

```
┌─────────────────────────────────────────────────────────────────┐
│                    UNICODE CONCEPTS                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Code Point: Unique number for each character                   │
│  - Written as U+XXXX (hex)                                      │
│  - Example: U+0041 = 'A', U+4E2D = '中'                         │
│                                                                  │
│  Range: U+0000 to U+10FFFF (over 1 million possible)           │
│                                                                  │
│  First 128 code points = ASCII (backwards compatible!)          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 4.3 Unicode Encoding Formats

How do you store a code point (up to 21 bits) in bytes?

**UTF-8** (Variable length: 1-4 bytes):
```
Code Point Range        Byte 1     Byte 2     Byte 3     Byte 4
─────────────────────────────────────────────────────────────────
U+0000 to U+007F       0xxxxxxx   (1 byte, same as ASCII!)
U+0080 to U+07FF       110xxxxx   10xxxxxx  (2 bytes)
U+0800 to U+FFFF       1110xxxx   10xxxxxx  10xxxxxx  (3 bytes)
U+10000 to U+10FFFF    11110xxx   10xxxxxx  10xxxxxx  10xxxxxx (4 bytes)
```

**Example: Encoding '中' (U+4E2D) in UTF-8**:
```
Code point: U+4E2D = 0100 1110 0010 1101 (binary)

Falls in range U+0800-U+FFFF, so 3 bytes:
Pattern: 1110xxxx 10xxxxxx 10xxxxxx

Fill in the bits:
4E2D = 0100 1110 0010 1101
Split: 0100 | 111000 | 101101

Result: 1110 0100  10 111000  10 101101
      = E4         B8         AD
      = 0xE4 0xB8 0xAD
```

**UTF-16** (2 or 4 bytes):
- Most common characters: 2 bytes
- Rare characters: 4 bytes (surrogate pairs)
- Used internally by Windows and Java

**UTF-32** (Always 4 bytes):
- Simple but wasteful
- Rarely used

### 4.4 Python Unicode

```python
# Unicode in Python 3 (strings are Unicode by default)
text = "Hello 中文 😀"
print(text)  # Works!

# Get code point
print(ord('中'))  # 20013 (decimal)
print(hex(ord('中')))  # 0x4e2d

# Create from code point
print(chr(20013))  # 中
print(chr(0x4e2d))  # 中

# Unicode escape in string
text = "\u4e2d\u6587"  # 中文
print(text)

# Encode to bytes
text = "中文"
utf8_bytes = text.encode('utf-8')
print(utf8_bytes)  # b'\xe4\xb8\xad\xe6\x96\x87'
print(utf8_bytes.hex())  # e4b8ade69687

# Decode from bytes
decoded = utf8_bytes.decode('utf-8')
print(decoded)  # 中文
```

---

## 5. Hexadecimal Encoding

### 5.1 What is Hex Encoding?

Converting bytes to their hexadecimal representation.

```
Byte: 0xFF (255 in decimal)
Hex Encoded: "FF" (two characters)

String: "Hello"
Bytes: 48 65 6C 6C 6F
Hex Encoded: "48656C6C6F"
```

### 5.2 Recognizing Hex

```
Hex Encoding Characteristics:
─────────────────────────────────────────
✓ Only characters: 0-9, A-F (or a-f)
✓ Usually even length (2 chars per byte)
✓ Often prefixed with "0x" or "\x"
✓ Looks like: "48656c6c6f" or "0x48 0x65 0x6c"

Examples:
"48656c6c6f"           ← Hex (decodes to "Hello")
"48 65 6C 6C 6F"       ← Hex with spaces
"0x48,0x65,0x6c"       ← Hex with 0x prefix
"\x48\x65\x6c"         ← Hex escape sequence
```

### 5.3 Hex Encoding in Python

```python
# String to Hex
text = "Hello"
hex_str = text.encode().hex()
print(hex_str)  # "48656c6c6f"

# Alternative
import binascii
hex_str = binascii.hexlify(text.encode()).decode()
print(hex_str)  # "48656c6c6f"

# Hex to String
hex_str = "48656c6c6f"
text = bytes.fromhex(hex_str).decode()
print(text)  # "Hello"

# Alternative
text = binascii.unhexlify(hex_str).decode()
print(text)  # "Hello"

# Binary data (non-text)
data = b'\x00\x01\x02\xff'
hex_str = data.hex()
print(hex_str)  # "000102ff"
```

### 5.4 Hex in Security Contexts

```python
# Shellcode is often in hex
shellcode_hex = "31c050682f2f7368682f62696e89e3505389e1b00bcd80"
shellcode = bytes.fromhex(shellcode_hex)

# Memory addresses
address = "0x7ffff7a3c000"
addr_int = int(address, 16)

# Hash values are hex
md5_hash = "5d41402abc4b2a76b9719d911017c592"
sha256_hash = "2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824"
```

---

## 6. Base64 Encoding

### 6.1 What is Base64?

Base64 encodes binary data using 64 printable ASCII characters.

**Why Base64?**
- Email (SMTP) is text-only - can't send binary attachments directly
- JSON/XML are text formats - can't embed binary
- URLs have restricted characters
- Base64 converts ANY data to safe text

### 6.2 The Base64 Alphabet

```
+------------------------------------------------------------------+
|                    BASE64 ALPHABET                                |
+------------------------------------------------------------------+

Value  Char    Value  Char    Value  Char    Value  Char
──────────────────────────────────────────────────────────
  0     A       16     Q       32     g       48     w
  1     B       17     R       33     h       49     x
  2     C       18     S       34     i       50     y
  3     D       19     T       35     j       51     z
  4     E       20     U       36     k       52     0
  5     F       21     V       37     l       53     1
  6     G       22     W       38     m       54     2
  7     H       23     X       39     n       55     3
  8     I       24     Y       40     o       56     4
  9     J       25     Z       41     p       57     5
 10     K       26     a       42     q       58     6
 11     L       27     b       43     r       59     7
 12     M       28     c       44     s       60     8
 13     N       29     d       45     t       61     9
 14     O       30     e       46     u       62     +
 15     P       31     f       47     v       63     /

Padding character: =
```

### 6.3 How Base64 Works (Step by Step)

```
Input: "Man" (3 bytes)

Step 1: Convert to binary (8 bits each)
M = 77  = 01001101
a = 97  = 01100001
n = 110 = 01101110

Combined: 01001101 01100001 01101110 (24 bits)

Step 2: Split into 6-bit groups
010011 | 010110 | 000101 | 101110

Step 3: Convert each 6-bit group to decimal
010011 = 19
010110 = 22
000101 = 5
101110 = 46

Step 4: Look up in Base64 alphabet
19 = T
22 = W
5  = F
46 = u

Result: "TWFu"
```

### 6.4 Base64 Padding

When input isn't divisible by 3 bytes, padding is needed:

```
Input: "Ma" (2 bytes = 16 bits)

Step 1: Binary
M = 01001101
a = 01100001

Combined: 01001101 01100001 (16 bits)

Step 2: Pad to multiple of 6
01001101 01100001 0000 (added 4 zeros for padding)

Step 3: Split into 6-bit groups
010011 | 010110 | 000100 | (missing)

Step 4: Convert
19 = T
22 = W
4  = E
(padding) = =

Result: "TWE="

Input: "M" (1 byte = 8 bits)
Result: "TQ==" (needs 2 padding chars)
```

### 6.5 Recognizing Base64

```
Base64 Characteristics:
─────────────────────────────────────────
✓ Characters: A-Z, a-z, 0-9, +, /
✓ Padding: Often ends with = or ==
✓ Length: Multiple of 4 (after padding)
✓ No spaces (usually, unless formatted)
✓ 33% larger than original data

Examples:
"SGVsbG8gV29ybGQ="     ← "Hello World"
"dGVzdA=="             ← "test"
"YWJj"                 ← "abc" (no padding needed)

NOT Base64 (contains invalid chars):
"SGVsbG8-V29ybGQ="     ← Has hyphen
"SGVsbG8 V29ybGQ="     ← Has space
```

### 6.6 Base64 in Python

```python
import base64

# Encode
text = "Hello World"
encoded = base64.b64encode(text.encode()).decode()
print(encoded)  # "SGVsbG8gV29ybGQ="

# Decode
encoded = "SGVsbG8gV29ybGQ="
decoded = base64.b64decode(encoded).decode()
print(decoded)  # "Hello World"

# Encode binary data
binary_data = b'\x00\x01\x02\xff'
encoded = base64.b64encode(binary_data).decode()
print(encoded)  # "AAEC/w=="

# URL-safe Base64 (replaces + with -, / with _)
encoded = base64.urlsafe_b64encode(text.encode()).decode()
print(encoded)  # "SGVsbG8gV29ybGQ=" (no difference for this input)
```

### 6.7 Base64 Variants

```
Standard Base64 (RFC 4648):
Alphabet: A-Za-z0-9+/
Padding: =
Used in: Email (MIME), XML

URL-Safe Base64:
Alphabet: A-Za-z0-9-_ (+ becomes -, / becomes _)
Padding: Optional (often omitted)
Used in: URLs, JWT tokens

Base64url (RFC 4648 Section 5):
Same as URL-Safe
Used in: JSON Web Tokens (JWT)
```

---

## 7. URL/Percent Encoding

### 7.1 What is URL Encoding?

URLs can only contain certain characters. Special characters must be "escaped" using percent encoding.

```
URL: https://example.com/search?q=hello world&lang=c++

Problem: 
- Space is not allowed
- + has special meaning
- & separates parameters

Solution (URL Encoded):
https://example.com/search?q=hello%20world&lang=c%2B%2B
```

### 7.2 How It Works

```
Percent Encoding Format: %XX (where XX is hex value)

Character  ASCII  Encoded
─────────────────────────────
Space       32     %20
!           33     %21
#           35     %23
$           36     %24
&           38     %26
+           43     %2B
/           47     %2F
:           58     %3A
=           61     %3D
?           63     %3F
@           64     %40

Unreserved (don't need encoding):
A-Z, a-z, 0-9, - _ . ~
```

### 7.3 Python URL Encoding

```python
from urllib.parse import quote, unquote, urlencode

# Encode a string
text = "Hello World?"
encoded = quote(text)
print(encoded)  # "Hello%20World%3F"

# Decode
decoded = unquote(encoded)
print(decoded)  # "Hello World?"

# Encode with safe characters (not encoded)
encoded = quote("path/to/file", safe="/")
print(encoded)  # "path/to/file" (/ not encoded)

# Encode dictionary for query string
params = {"name": "John Doe", "city": "New York"}
query = urlencode(params)
print(query)  # "name=John+Doe&city=New+York"
```

### 7.4 URL Encoding in Security

**WAF Bypass via Double Encoding**:
```
Original: <script>
Single encoded: %3Cscript%3E
Double encoded: %253Cscript%253E

If WAF only decodes once, double encoding may bypass!
```

**SQL Injection via URL Encoding**:
```
Normal: ' OR '1'='1
URL encoded: %27%20OR%20%271%27%3D%271
```

---

## 8. Other Encoding Schemes

### 8.1 Base32

Uses 32 characters: A-Z and 2-7

```
Advantages over Base64:
- Case insensitive
- No confusing characters (0/O, 1/I/l)
- Used in: TOTP codes, Onion addresses

Python:
import base64
encoded = base64.b32encode(b"Hello").decode()
print(encoded)  # "JBSWY3DP"
```

### 8.2 Base16 (Hex)

Already covered - just hexadecimal encoding.

### 8.3 Base58 (Bitcoin)

Removes confusing characters from Base64:
- No 0, O, I, l (look similar)
- No + and / (URL problems)

```
Alphabet: 123456789ABCDEFGHJKLMNPQRSTUVWXYZabcdefghijkmnopqrstuvwxyz

Used in: Bitcoin addresses, IPFS
```

### 8.4 ROT13

Simple letter substitution (A↔N, B↔O, etc.):

```
Alphabet:  ABCDEFGHIJKLMNOPQRSTUVWXYZ
ROT13:     NOPQRSTUVWXYZABCDEFGHIJKLM

"Hello" → "Uryyb"
Apply twice → back to original!

Python:
import codecs
encoded = codecs.encode("Hello", "rot_13")
print(encoded)  # "Uryyb"
```

### 8.5 HTML Entities

Escape characters for HTML:

```
Character  Entity       Numeric
─────────────────────────────────
<          &lt;         &#60;
>          &gt;         &#62;
&          &amp;        &#38;
"          &quot;       &#34;
'          &apos;       &#39;
(space)    &nbsp;       &#32;

Any character can use: &#DECIMAL; or &#xHEX;
Example: &#x3C;script&#x3E; = <script>
```

### 8.6 Unicode Escapes

Different formats for representing Unicode:

```
Python:  \uXXXX or \UXXXXXXXX
Example: \u0048\u0065\u006c\u006c\u006f = "Hello"

JavaScript: \uXXXX
Example: "\u0048\u0065\u006c\u006c\u006f"

CSS/HTML: \XXXX (in CSS) or &#xXXXX; (in HTML)
Example: \0048\0065\006c\006c\006f

URL: %uXXXX (non-standard, legacy)
```

---

## 9. Identifying Unknown Encodings

### 9.1 Visual Recognition Guide

```
+------------------------------------------------------------------+
|                    ENCODING IDENTIFICATION                        |
+------------------------------------------------------------------+

Base64:
├─ Characters: A-Za-z0-9+/
├─ Often ends: = or ==
├─ Length: Multiple of 4
└─ Example: "SGVsbG8gV29ybGQ="

Hex:
├─ Characters: 0-9A-Fa-f only
├─ Usually even length
├─ Prefixes: 0x, \x, or none
└─ Example: "48656c6c6f"

URL Encoded:
├─ Contains: %XX patterns
├─ Normal text + encoded special chars
└─ Example: "Hello%20World%21"

Unicode Escaped:
├─ Pattern: \uXXXX or \xXX
└─ Example: "\u0048\u0065\u006c\u006c\u006f"

HTML Entities:
├─ Pattern: &name; or &#number;
└─ Example: "&lt;script&gt;"

Base32:
├─ Characters: A-Z2-7 only
├─ Padding: = (up to 6)
├─ All uppercase
└─ Example: "JBSWY3DP"

Binary:
├─ Characters: 0 and 1 only
├─ Often grouped by 8
└─ Example: "01001000 01100101"
```

### 9.2 Python Detection Script

```python
import re
import base64
from urllib.parse import unquote

def identify_encoding(text):
    """Attempt to identify the encoding of a string"""
    
    # Check Base64
    if re.match(r'^[A-Za-z0-9+/]+={0,2}$', text) and len(text) % 4 == 0:
        try:
            decoded = base64.b64decode(text)
            if decoded.decode('utf-8', errors='ignore').isprintable():
                return "Base64", decoded.decode()
        except:
            pass
    
    # Check Hex
    if re.match(r'^[0-9A-Fa-f]+$', text) and len(text) % 2 == 0:
        try:
            decoded = bytes.fromhex(text).decode()
            return "Hex", decoded
        except:
            pass
    
    # Check URL encoding
    if '%' in text:
        decoded = unquote(text)
        if decoded != text:
            return "URL", decoded
    
    # Check Unicode escapes
    if '\\u' in text or '\\x' in text:
        try:
            decoded = text.encode().decode('unicode_escape')
            return "Unicode Escape", decoded
        except:
            pass
    
    # Check Binary
    if re.match(r'^[01\s]+$', text):
        binary = text.replace(' ', '')
        if len(binary) % 8 == 0:
            chars = [chr(int(binary[i:i+8], 2)) for i in range(0, len(binary), 8)]
            return "Binary", ''.join(chars)
    
    return "Unknown", None

# Test
print(identify_encoding("SGVsbG8="))           # Base64
print(identify_encoding("48656c6c6f"))         # Hex
print(identify_encoding("Hello%20World"))       # URL
```

---

## 10. Multi-Layer Encoding

### 10.1 What is Multi-Layer Encoding?

Attackers often stack multiple encodings to evade detection:

```
Original: <script>alert(1)</script>

Layer 1 (URL): %3Cscript%3Ealert(1)%3C/script%3E
Layer 2 (Base64): JTNDc2NyaXB0JTNFYWxlcnQoMSklM0Mvc2NyaXB0JTNF
Layer 3 (Hex): 4a5433...

Security tool sees gibberish, application decodes it!
```

### 10.2 Decoding Multiple Layers

```python
import base64
from urllib.parse import unquote

def decode_layers(text, max_layers=10):
    """Recursively decode multiple encoding layers"""
    results = [("Original", text)]
    
    for i in range(max_layers):
        decoded = None
        encoding = None
        
        # Try Base64
        try:
            if len(text) % 4 == 0:
                decoded_bytes = base64.b64decode(text)
                decoded = decoded_bytes.decode('utf-8')
                encoding = "Base64"
        except:
            pass
        
        # Try URL
        if not decoded:
            url_decoded = unquote(text)
            if url_decoded != text:
                decoded = url_decoded
                encoding = "URL"
        
        # Try Hex
        if not decoded:
            try:
                if all(c in '0123456789abcdefABCDEF' for c in text):
                    decoded = bytes.fromhex(text).decode()
                    encoding = "Hex"
            except:
                pass
        
        if decoded:
            results.append((encoding, decoded))
            text = decoded
        else:
            break
    
    return results

# Example: Multi-layer encoded XSS
encoded = "SlRETmMyTnlhWEIwSlRORllXeGxjblFvTVNrbE0wTXZjMk55YVhCMEpUTkY="
layers = decode_layers(encoded)
for layer, value in layers:
    print(f"{layer}: {value}")
```

---

## 11. Encoding in Security

### 11.1 WAF Bypass Techniques

```python
# SQL Injection variations
payloads = {
    "Normal": "' OR '1'='1",
    "URL": "%27%20OR%20%271%27%3D%271",
    "Double URL": "%2527%2520OR%2520%25271%2527%253D%25271",
    "Unicode": "%u0027%u0020OR%u0020%u00271%u0027%u003D%u00271",
    "Hex": "0x27204f5220273127273d2731",
}

# XSS variations
xss_payloads = {
    "Normal": "<script>alert(1)</script>",
    "HTML Entity": "&#60;script&#62;alert(1)&#60;/script&#62;",
    "URL": "%3Cscript%3Ealert(1)%3C/script%3E",
    "Unicode": "\u003cscript\u003ealert(1)\u003c/script\u003e",
    "Base64 eval": "eval(atob('YWxlcnQoMSk='))",
}
```

### 11.2 Malware Encoding

```python
# Malware often encodes C2 URLs
c2_url = "http://evil.com/beacon"

# XOR encode (covered in Obfuscation chapter)
key = 0x42
xor_encoded = bytes([b ^ key for b in c2_url.encode()])

# Base64 encode
b64_encoded = base64.b64encode(c2_url.encode())

# Hex encode
hex_encoded = c2_url.encode().hex()

# At runtime, malware decodes to get real URL
```

### 11.3 CTF Challenge Pattern

```
Common CTF encoding puzzles:

1. Identify the encoding
2. Decode layer by layer
3. Often: Base64 → Hex → XOR → Flag

Example:
"NTk0ZDdhNjM3YTM5Nzc2NDY3MzU3YTc5Njg0NzQ1Nzg2NDQ3NTY2OTY3NTU3YQ=="

Step 1: Looks like Base64 (ends with ==)
Decode: "594d7a637a39776467357a7968474578644756696755577a"

Step 2: Looks like Hex (only 0-9a-f)
Decode: "YMzcz9wdg5zyhGExdGVigUWz"

Step 3: Looks like Base64 again
Decode: (may be XOR'd or rotated)

Keep going until you find the flag!
```

---

## 12. Practical Tools

### 12.1 CyberChef

Web-based tool for encoding/decoding: https://gchq.github.io/CyberChef/

```
Common Recipes:
─────────────────────────────────────────
From Base64 → To Hex → XOR (with key)
URL Decode → From HTML Entity
Magic (auto-detect and decode)
```

### 12.2 Command Line Tools

```bash
# Base64
echo -n "Hello" | base64                    # Encode
echo "SGVsbG8=" | base64 -d                 # Decode

# Hex
echo -n "Hello" | xxd -p                    # Encode
echo "48656c6c6f" | xxd -r -p               # Decode

# URL Encoding (Python)
python3 -c "from urllib.parse import quote; print(quote('Hello World'))"
python3 -c "from urllib.parse import unquote; print(unquote('Hello%20World'))"

# ASCII values
printf '%d\n' "'A"                          # 65
printf \\$(printf '%03o' 65)                # A
```

### 12.3 Python Quick Reference

```python
import base64
from urllib.parse import quote, unquote
import binascii
import codecs

# Base64
base64.b64encode(b"text").decode()
base64.b64decode("dGV4dA==").decode()

# Hex
"text".encode().hex()
bytes.fromhex("74657874").decode()

# URL
quote("hello world")
unquote("hello%20world")

# ROT13
codecs.encode("text", "rot_13")

# ASCII
ord('A')  # 65
chr(65)   # 'A'

# Binary string to int
int("1000001", 2)  # 65

# Int to binary string
bin(65)  # '0b1000001'
format(65, '08b')  # '01000001'
```

---

## 13. Hands-On Labs

### Lab 1: Encoding Detection Challenge

```python
"""
Identify and decode each of these:
"""
challenges = [
    "48656c6c6f20576f726c6421",
    "SGVsbG8gV29ybGQh",
    "Hello%20World%21",
    "01001000 01100101 01101100 01101100 01101111",
    "JBSWY3DPEBLW64TMMQQQ====",
    "&lt;script&gt;",
    "\\x48\\x65\\x6c\\x6c\\x6f",
]

# Solve each one!
```

### Lab 2: Multi-Layer Decode

```python
"""
Decode this multi-layer encoded string to find the flag:
"""
encoded = "NTI1NzY0Njk2ZTY3NWYzMTczNWY2NjMxNzI3Mzc0NWY3Mzc0MzM3MDIx"

# Hint: Base64 → Hex → ASCII = flag
```

### Lab 3: Build an Auto-Decoder

```python
"""
Write a function that automatically detects and decodes
any of: Base64, Hex, URL, Binary

Should handle multiple layers!
"""
def auto_decode(text):
    # Your code here
    pass
```

---

## 14. Common Mistakes

### Mistake 1: Confusing Encoding with Encryption

```
BAD:  "I Base64 encoded the API key so it's secure"
FACT: Base64 provides ZERO security

GOOD: "I encrypted the API key with AES, then Base64
       encoded it for transmission"
```

### Mistake 2: Wrong Encoding for Binary

```python
# BAD: Trying to decode non-text binary as string
binary_data = b'\x00\x01\xff\xfe'
text = binary_data.decode('utf-8')  # Error!

# GOOD: Use hex or base64 for binary
hex_str = binary_data.hex()  # "0001fffe"
b64_str = base64.b64encode(binary_data).decode()  # "AAH//g=="
```

### Mistake 3: Ignoring Encoding in Web Input

```python
# User input may be encoded
user_input = request.args.get('name')  # "John%20Doe"

# Always decode user input properly
from urllib.parse import unquote
clean_input = unquote(user_input)  # "John Doe"
```

---

## 15. Interview Questions

**Q1: What's the difference between Base64 and encryption?**
> Base64 is encoding - it transforms data format without a key and provides no security. Anyone can decode it. Encryption uses a key to scramble data so only key holders can decrypt it.

**Q2: Why does Base64 output become 33% larger?**
> Base64 converts 3 bytes (24 bits) into 4 characters (6 bits each). 4/3 = 1.33, so output is 33% larger plus potential padding.

**Q3: When would you use URL encoding?**
> When including special characters in URLs (spaces, &, ?, =, etc.) that have special meaning in URL syntax and would break parsing.

**Q4: How do you identify Base64 encoded text?**
> Characters are A-Za-z0-9+/, often ends with = or ==, length is multiple of 4, no other characters.

**Q5: What's UTF-8 and why is it important?**
> UTF-8 is a variable-length Unicode encoding that's backwards compatible with ASCII. First 128 characters are identical to ASCII. It's the standard encoding for the web and most modern systems.

---

## 16. Quick Reference

### Encoding Cheat Sheet

| Encoding | Alphabet | Padding | Example |
|----------|----------|---------|---------|
| Base64 | A-Za-z0-9+/ | = | SGVsbG8= |
| Base32 | A-Z2-7 | = | JBSWY3DP |
| Base16/Hex | 0-9A-F | None | 48656C6C6F |
| URL | %XX | None | Hello%20World |

### Python One-Liners

```python
# Base64
base64.b64encode(b"text").decode()
base64.b64decode("dGV4dA==")

# Hex
"text".encode().hex()
bytes.fromhex("74657874")

# URL
quote("text")
unquote("text")

# ASCII
ord('A')  # 65
chr(65)   # 'A'
```

### Bash One-Liners

```bash
echo -n "text" | base64           # Encode Base64
echo "dGV4dA==" | base64 -d       # Decode Base64
echo -n "text" | xxd -p           # Encode Hex
echo "74657874" | xxd -r -p       # Decode Hex
```

---

## Next Steps

Now that you understand encoding, proceed to:
- **[[03_Hashing_and_Integrity]]** - One-way functions
- **[[11_Obfuscation]]** - Advanced encoding for evasion

---

## References

- [RFC 4648 - Base Encodings](https://tools.ietf.org/html/rfc4648)
- [CyberChef](https://gchq.github.io/CyberChef/)
- [ASCII Table](https://www.asciitable.com/)
- [Unicode Standard](https://unicode.org/standard/standard.html)
