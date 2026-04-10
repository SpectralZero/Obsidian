
# 12 - Steganography

> **"The art of hiding a message is sometimes more valuable than encrypting it."**

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [Absolute Beginner's Guide](#absolute-beginners-guide)
3. [Steganography vs Cryptography](#steganography-vs-cryptography)
4. [Image Steganography](#image-steganography)
5. [Audio Steganography](#audio-steganography)
6. [Video Steganography](#video-steganography)
7. [Text and Document Steganography](#text-and-document-steganography)
8. [Network Steganography](#network-steganography)
9. [File Structure Exploitation](#file-structure-exploitation)
10. [Metadata Hiding](#metadata-hiding)
11. [Steganography Tools](#steganography-tools)
12. [Steganalysis and Detection](#steganalysis-and-detection)
13. [CTF Steganography Techniques](#ctf-steganography-techniques)
14. [Real-World Applications](#real-world-applications)
15. [OSCP Relevance](#oscp-relevance)
16. [Hands-On Labs](#hands-on-labs)
17. [Interview Questions](#interview-questions)
18. [Quick Reference](#quick-reference)

---

## Executive Summary

**Steganography** is the practice of concealing messages or data within other non-secret data or physical objects. Unlike cryptography, which makes messages unreadable, steganography hides the very existence of the message.

```
+-------------------------------------------------------------------+
|                   STEGANOGRAPHY OVERVIEW                           |
+-------------------------------------------------------------------+

GOAL: Hide the existence of a message (not just its content)

METHODS:
  +-- Image Steganography (most common)
  |     - LSB (Least Significant Bit)
  |     - DCT coefficient manipulation
  |     - Palette manipulation
  |
  +-- Audio Steganography
  |     - LSB in audio samples
  |     - Echo hiding
  |     - Phase coding
  |
  +-- Video Steganography
  |     - Frame manipulation
  |     - Audio track hiding
  |
  +-- Text Steganography
  |     - Whitespace encoding
  |     - Zero-width characters
  |     - Semantic changes
  |
  +-- Network Steganography
  |     - Protocol field manipulation
  |     - Timing channels
  |
  +-- File Structure Exploitation
        - Appended data
        - Slack space
        - Alternate data streams

+-------------------------------------------------------------------+
```

---

## Absolute Beginner's Guide

### What is Steganography?

**Etymology**: Greek "steganos" (covered) + "graphein" (writing) = "covered writing"

**Analogy: The Invisible Ink Letter**
```
CRYPTOGRAPHY:
  You write: "KHOOR ZRUOG" (Caesar cipher of "HELLO WORLD")
  Anyone can see there's a secret message, but can't read it.
  
STEGANOGRAPHY:
  You write: "Having a great time. Everyone loves the weather. 
              Larry left on Monday. Only thing missing is you."
  
  First letters: H.a.g.t.E.l.t.w.L.l.o.M.O.t.m.i.y.
  Message: HELLO LMOY → "HELLO"
  
  Nobody even knows there's a hidden message!
```

### Visual Example: LSB Steganography

```
+-------------------------------------------------------------------+
|              LEAST SIGNIFICANT BIT (LSB) HIDING                    |
+-------------------------------------------------------------------+

CONCEPT:
  Each pixel in an image has color values (0-255 for each channel).
  The last bit (LSB) has minimal visual impact.
  We can replace it with our secret data!

EXAMPLE: Hiding a bit in a red pixel value

Original Red Value: 156
Binary:            10011100
                         ^-- LSB (0)

We want to hide: 1

Modified Value:    10011101
                         ^-- LSB changed to 1
New Decimal:       157

Visual difference: INVISIBLE to human eye!
  156 red vs 157 red = undetectable

CAPACITY:
  1920×1080 RGB image = 1920 × 1080 × 3 channels × 1 bit
                      = 6,220,800 bits
                      = 777,600 bytes
                      = ~760 KB of hidden data!

+-------------------------------------------------------------------+
```

### Why Use Steganography?

```
+-------------------------------------------------------------------+
|              STEGANOGRAPHY USE CASES                               |
+-------------------------------------------------------------------+

LEGITIMATE USES:
  - Digital watermarking (copyright protection)
  - Covert communication (journalism in oppressive regimes)
  - Data integrity verification
  - Confidential business communication

MALICIOUS USES:
  - Malware command and control (C2)
  - Data exfiltration
  - Hiding attack tools
  - Bypassing DLP systems

CTF/SECURITY USES:
  - Capture The Flag challenges
  - Forensic investigations
  - Security research
  - Red team operations

+-------------------------------------------------------------------+
```

---

## Steganography vs Cryptography

### Conceptual Comparison

```
+-------------------------------------------------------------------+
|           STEGANOGRAPHY VS CRYPTOGRAPHY                            |
+-------------------------------------------------------------------+

                CRYPTOGRAPHY              STEGANOGRAPHY
                
GOAL:           Hide message content     Hide message existence

VISIBLE:        Message is visible       Message is invisible
                (but unreadable)         (looks like normal data)

DETECTION:      Known that secret exists Cannot tell secret exists
                                         (until analyzed)

EXAMPLE:        "Khoor Zruog"           Normal-looking image
                (encrypted)              (with hidden data)

STRENGTH:       Mathematical security    Obscurity/undetectability

WEAKNESS:       Presence obvious         Fragile (compression may
                                         destroy hidden data)

COMBINATION:    Best practice is BOTH!
                Encrypt message, THEN hide it
                (Even if found, can't read it)

+-------------------------------------------------------------------+
```

### When to Use Each

```
CRYPTOGRAPHY ALONE:
  ✓ When you don't care if encryption is detected
  ✓ Standard secure communication
  ✓ Password protection
  ✓ Data at rest protection

STEGANOGRAPHY ALONE:
  ✓ When detection means danger (authoritarian regimes)
  ✓ Digital watermarking
  ✓ Bypassing content filters
  ✗ Weak if discovered (no encryption)

BOTH TOGETHER (BEST):
  ✓ Maximum security
  ✓ Message encrypted (protection even if found)
  ✓ Hidden in cover file (not detected)
  Example: Encrypt secret with AES, then hide in image with steghide
```

---

## Image Steganography

### LSB (Least Significant Bit) Encoding

```
+-------------------------------------------------------------------+
|                    LSB ENCODING DEEP DIVE                          |
+-------------------------------------------------------------------+

RGB IMAGE STRUCTURE:
  Each pixel: (Red, Green, Blue) = 3 bytes = 24 bits
  Each value: 0-255 (8 bits)
  
  Example Pixel: (156, 230, 42)
  Binary:        (10011100, 11100110, 00101010)

HIDING DATA:
  Message: "Hi" = 01001000 01101001 (ASCII binary)
  
  Original Pixels (first 3):
    Pixel 1: (10011100, 11100110, 00101010)  RGB
    Pixel 2: (10101010, 11001100, 10101001)  RGB
    Pixel 3: (11110000, 00001111, 10101010)  RGB
  
  Hide "H" (01001000) in first 8 color values:
    New Pixel 1: (1001110[0], 1110011[1], 0010101[0])
    New Pixel 2: (1010101[0], 1100110[1], 1010100[0])
    New Pixel 3: (1111000[0], 0000111[0], ...)
  
  Modified Pixels:
    Pixel 1: (156, 231, 42)   ← Changed by 0 or 1
    Pixel 2: (170, 204, 168)
    Pixel 3: (240, 14, ...)

EXTRACTION:
  Read LSB of each color value in sequence
  Reconstruct bytes → Original message

+-------------------------------------------------------------------+
```

### Image Formats and Steganography

```
+-------------------------------------------------------------------+
|            IMAGE FORMATS FOR STEGANOGRAPHY                         |
+-------------------------------------------------------------------+

PNG (Portable Network Graphics):
  + Lossless compression
  + Perfect for LSB steganography
  + Hidden data survives compression
  + Supports alpha channel (4th channel to hide data)
  - Larger file size

BMP (Bitmap):
  + No compression (raw pixels)
  + Easiest for steganography
  + Data survives perfectly
  - Very large files
  - Suspicious in web context

JPEG (Joint Photographic Experts Group):
  + Common format (less suspicious)
  - Lossy compression DESTROYS LSB data!
  - Requires DCT-based steganography
  - More complex algorithms needed
  Tools: jsteg, outguess, steghide

GIF (Graphics Interchange Format):
  + Supports multiple frames (more hiding space)
  - Limited to 256 colors (palette-based)
  - Requires palette manipulation
  
TIFF:
  + Lossless
  + Large capacity
  - Less common, may raise suspicion

RECOMMENDATION:
  For LSB: Use PNG
  For JPEG: Use specialized tools (steghide, outguess)

+-------------------------------------------------------------------+
```

### Python LSB Implementation

```python
#!/usr/bin/env python3
"""
Simple LSB Steganography Implementation
Educational purposes
"""

from PIL import Image
import numpy as np

def text_to_binary(text):
    """Convert text to binary string"""
    return ''.join(format(ord(c), '08b') for c in text)

def binary_to_text(binary):
    """Convert binary string to text"""
    chars = [binary[i:i+8] for i in range(0, len(binary), 8)]
    return ''.join(chr(int(c, 2)) for c in chars if c)

def encode_lsb(image_path, message, output_path):
    """Hide message in image using LSB"""
    img = Image.open(image_path)
    pixels = np.array(img)
    
    # Add delimiter to know where message ends
    message += '\x00'  # Null terminator
    binary_message = text_to_binary(message)
    
    # Flatten pixel array
    flat_pixels = pixels.flatten()
    
    if len(binary_message) > len(flat_pixels):
        raise ValueError("Message too long for this image")
    
    # Embed each bit
    for i, bit in enumerate(binary_message):
        # Clear LSB and set to message bit
        flat_pixels[i] = (flat_pixels[i] & 0xFE) | int(bit)
    
    # Reshape and save
    stego_pixels = flat_pixels.reshape(pixels.shape)
    stego_img = Image.fromarray(stego_pixels.astype('uint8'))
    stego_img.save(output_path)
    print(f"Message hidden in {output_path}")

def decode_lsb(image_path):
    """Extract hidden message from image"""
    img = Image.open(image_path)
    pixels = np.array(img).flatten()
    
    # Extract LSBs
    binary_message = ''
    for pixel in pixels:
        binary_message += str(pixel & 1)
        
        # Check for null terminator every 8 bits
        if len(binary_message) % 8 == 0:
            if binary_message[-8:] == '00000000':
                binary_message = binary_message[:-8]
                break
    
    return binary_to_text(binary_message)

# Example usage
if __name__ == "__main__":
    # Hide message
    encode_lsb("original.png", "Secret message here!", "stego.png")
    
    # Extract message
    hidden = decode_lsb("stego.png")
    print(f"Hidden message: {hidden}")
```

### DCT-Based Steganography (JPEG)

```
+-------------------------------------------------------------------+
|              DCT-BASED STEGANOGRAPHY                               |
+-------------------------------------------------------------------+

WHY DCT?
  JPEG uses Discrete Cosine Transform for compression.
  LSB in pixel space destroyed by JPEG compression.
  Must hide in DCT coefficients instead.

HOW JPEG WORKS:
  1. Image split into 8×8 blocks
  2. Each block transformed with DCT
  3. DCT coefficients quantized (lossy)
  4. Coefficients compressed with Huffman/arithmetic coding

HIDING DATA:
  Modify quantized DCT coefficients
  Change least significant bits of AC coefficients
  Avoid DC coefficients (too visible)

JSTEG ALGORITHM:
  For each non-zero, non-one AC coefficient:
    Replace LSB with message bit
  
OUTGUESS:
  More sophisticated
  Preserves statistical properties
  Harder to detect

+-------------------------------------------------------------------+
```

---

## Audio Steganography

### Audio LSB Encoding

```
+-------------------------------------------------------------------+
|                 AUDIO STEGANOGRAPHY                                |
+-------------------------------------------------------------------+

AUDIO DATA STRUCTURE:
  WAV file: Sequence of samples (16-bit typically)
  44.1kHz × 16-bit × 2 channels = ~1.4 Mbps
  
LSB IN AUDIO:
  Same concept as images
  Modify LSB of audio samples
  Inaudible to humans (below noise floor)

CAPACITY EXAMPLE:
  5-minute WAV (44.1kHz, 16-bit, stereo):
  5 × 60 × 44100 × 2 × 1 bit = 26,460,000 bits
  = 3.3 MB of hidden data!

PYTHON EXAMPLE:
```

```python
import wave
import struct

def encode_audio_lsb(audio_path, message, output_path):
    """Hide message in WAV file"""
    # Read WAV
    audio = wave.open(audio_path, 'rb')
    params = audio.getparams()
    frames = bytearray(audio.readframes(audio.getnframes()))
    audio.close()
    
    # Convert message to binary
    message += '\x00'
    binary = ''.join(format(ord(c), '08b') for c in message)
    
    # Embed in LSB of each sample
    for i, bit in enumerate(binary):
        frames[i] = (frames[i] & 0xFE) | int(bit)
    
    # Write output
    output = wave.open(output_path, 'wb')
    output.setparams(params)
    output.writeframes(bytes(frames))
    output.close()
```

### Other Audio Techniques

```
+-------------------------------------------------------------------+
|              ADVANCED AUDIO STEGANOGRAPHY                          |
+-------------------------------------------------------------------+

ECHO HIDING:
  Add inaudible echo to audio signal
  Echo delay encodes binary: Short = 0, Long = 1
  Very robust against compression

PHASE CODING:
  Modify phase of audio segments
  Humans don't perceive phase well
  Encodes data in phase shifts

SPREAD SPECTRUM:
  Spread signal across frequency spectrum
  Below noise floor at any single frequency
  Very hard to detect

TONE INSERTION:
  Add inaudible tones above 18kHz
  Only detectable with analysis
  Used in some watermarking

MP3/AAC STEGANOGRAPHY:
  Similar to JPEG - work in compressed domain
  Modify quantized frequency coefficients
  Survives re-compression

+-------------------------------------------------------------------+
```

---

## Video Steganography

### Video Hiding Techniques

```
+-------------------------------------------------------------------+
|                VIDEO STEGANOGRAPHY                                 |
+-------------------------------------------------------------------+

MASSIVE CAPACITY:
  1080p video at 30fps:
  1920 × 1080 × 3 × 30 = 186 MB/second (uncompressed)
  LSB capacity: ~23 MB/second of hidden data!

TECHNIQUES:

1. FRAME-BY-FRAME LSB
   Apply image steganography to each frame
   Works with uncompressed video
   Destroyed by video compression

2. DCT COEFFICIENT MANIPULATION
   Similar to JPEG
   Modify DCT coefficients in video codec
   Works with H.264, H.265

3. MOTION VECTOR HIDING
   Video codecs track motion between frames
   Hide data in motion vector values
   Subtle changes undetectable

4. TEMPORAL STEGANOGRAPHY
   Hide data in frame timing
   Dropped frames encode bits
   Very low capacity

5. AUDIO TRACK HIDING
   Use audio steganography on video's audio
   Often overlooked in analysis

DETECTION CHALLENGES:
  - Massive data volume to analyze
  - Multiple hiding locations
  - Compression artifacts mask changes

+-------------------------------------------------------------------+
```

---

## Text and Document Steganography

### Text-Based Hiding

```
+-------------------------------------------------------------------+
|              TEXT STEGANOGRAPHY                                    |
+-------------------------------------------------------------------+

1. WHITESPACE ENCODING
   Spaces and tabs at end of lines
   Space = 0, Tab = 1
   
   "Hello World   \t \t  "
                  ^^^ Hidden data in trailing whitespace

2. ZERO-WIDTH CHARACTERS
   Unicode has invisible characters:
   - U+200B: Zero Width Space
   - U+200C: Zero Width Non-Joiner
   - U+200D: Zero Width Joiner
   - U+FEFF: Zero Width No-Break Space
   
   "Hello" could contain invisible chars between letters

3. HOMOGLYPH SUBSTITUTION
   Replace characters with look-alikes:
   - 'a' (Latin) vs 'а' (Cyrillic) - look identical!
   - 'o' vs 'о' vs '0' vs 'ο' (Greek)
   Latin = 0, Cyrillic = 1

4. SEMANTIC STEGANOGRAPHY
   Word choice encodes data:
   "big" = 0, "large" = 1
   "happy" = 0, "glad" = 1
   
   Requires natural language generation

5. FIRST LETTER / ACROSTIC
   First letter of each word/line spells message
   (Like the HELLO example earlier)

+-------------------------------------------------------------------+
```

### Document Steganography

```
+-------------------------------------------------------------------+
|              DOCUMENT STEGANOGRAPHY                                |
+-------------------------------------------------------------------+

PDF STEGANOGRAPHY:
  - Modify object numbers
  - Add hidden annotations
  - Embed in image objects
  - JavaScript hidden in document
  
  Tools: wbStego, pdf-parser

WORD DOCUMENT:
  - Modify docx (ZIP file) structure
  - Hidden XML elements
  - Embedded OLE objects
  - Track changes hidden
  
  Examine: unzip document.docx → inspect XML

OPEN XML MANIPULATION:
  Office documents are ZIP archives
  Add extra files to archive
  Modify XML without visible change

FONT MANIPULATION:
  Custom fonts with modified glyphs
  Embed data in font file
  Document looks normal but font contains data

+-------------------------------------------------------------------+
```

### Zero-Width Character Encoding Example

```python
#!/usr/bin/env python3
"""
Zero-Width Character Steganography
"""

def encode_zwc(cover_text, secret):
    """Hide message using zero-width characters"""
    ZWC = {
        '0': '\u200b',  # Zero Width Space
        '1': '\u200c',  # Zero Width Non-Joiner
    }
    
    # Convert secret to binary
    binary = ''.join(format(ord(c), '08b') for c in secret)
    
    # Convert binary to zero-width characters
    hidden = ''.join(ZWC[b] for b in binary)
    
    # Insert hidden data in middle of cover text
    mid = len(cover_text) // 2
    return cover_text[:mid] + hidden + cover_text[mid:]

def decode_zwc(stego_text):
    """Extract hidden message from zero-width characters"""
    ZWC_DECODE = {
        '\u200b': '0',
        '\u200c': '1',
    }
    
    # Extract zero-width characters
    binary = ''
    for char in stego_text:
        if char in ZWC_DECODE:
            binary += ZWC_DECODE[char]
    
    # Convert binary to text
    chars = [binary[i:i+8] for i in range(0, len(binary), 8)]
    return ''.join(chr(int(c, 2)) for c in chars if len(c) == 8)

# Example
cover = "This is a normal looking sentence."
secret = "SECRET"

stego = encode_zwc(cover, secret)
print(f"Stego text: {stego}")  # Looks identical!
print(f"Length: {len(cover)} → {len(stego)}")  # Longer!

extracted = decode_zwc(stego)
print(f"Extracted: {extracted}")
```

---

## Network Steganography

### Protocol Field Manipulation

```
+-------------------------------------------------------------------+
|              NETWORK STEGANOGRAPHY                                 |
+-------------------------------------------------------------------+

TCP/IP HEADER FIELDS:
  Many fields can carry hidden data:
  
  IP Header:
    - Identification field (16 bits)
    - Fragment offset (13 bits)
    - TTL field (8 bits)
    - Options field (variable)
  
  TCP Header:
    - Sequence number (32 bits)
    - Acknowledgment number (32 bits)
    - Urgent pointer (16 bits)
    - Options field (variable)
  
  ICMP:
    - Identifier (16 bits)
    - Sequence number (16 bits)
    - Payload (any size for echo)

EXAMPLE: ICMP COVERT CHANNEL
  Ping packets with hidden data in payload
  Looks like normal ping traffic
  
  ping -p "48454c4c4f" target  # Pattern = "HELLO" in hex

+-------------------------------------------------------------------+
```

### DNS Covert Channels

```
+-------------------------------------------------------------------+
|                 DNS TUNNELING                                      |
+-------------------------------------------------------------------+

CONCEPT:
  Encode data in DNS queries/responses
  DNS traffic often allowed through firewalls
  
QUERY-BASED:
  Encode data in subdomain:
  SEVJRE1FU1NBR0U.tunnel.attacker.com
  ^^^^^^^^^^^^^^^^^
  Base32 encoded data
  
  Attacker's DNS server receives query
  Decodes data from subdomain

RESPONSE-BASED:
  TXT records can contain arbitrary data
  CNAME records for encoding
  NULL records for binary data

TOOLS:
  - dnscat2: DNS tunneling tool
  - iodine: IP over DNS
  - dns2tcp: TCP over DNS
  
DETECTION:
  - Unusual query volume
  - Long subdomain names
  - Queries to unknown domains
  - High entropy in labels

+-------------------------------------------------------------------+
```

### Timing Channels

```
+-------------------------------------------------------------------+
|               TIMING COVERT CHANNELS                               |
+-------------------------------------------------------------------+

CONCEPT:
  Encode data in timing of packets
  No modification to packet content
  Very hard to detect

EXAMPLE:
  Bit 0: Send packet, wait 100ms
  Bit 1: Send packet, wait 200ms
  
  Receiver measures inter-packet timing
  Decodes binary message

ADVANCED:
  - Keystroke timing (typing cadence)
  - Process scheduling
  - Cache timing (Spectre-like)
  
CHALLENGES:
  - Network jitter affects accuracy
  - Low bandwidth
  - Synchronization required

+-------------------------------------------------------------------+
```

---

## File Structure Exploitation

### Appended Data

```
+-------------------------------------------------------------------+
|                APPENDED DATA HIDING                                |
+-------------------------------------------------------------------+

CONCEPT:
  Many file formats ignore data after their end marker
  Append hidden data after the legitimate file ends
  
EXAMPLE: JPEG
  JPEG ends with FFD9 (End of Image marker)
  Append any data after FFD9
  Image opens normally, hidden data ignored
  
  +------------------+----------+
  | Valid JPEG Image | Hidden   |
  | ....FFD9         | Data     |
  +------------------+----------+

EXAMPLE: PNG
  PNG ends with IEND chunk
  Append data after IEND
  
EXAMPLE: ZIP
  ZIP has end-of-central-directory
  Append data after it

DETECTION:
  Compare file size to expected
  Check for data after end markers
  Use binwalk to analyze

EXTRACTION:
  binwalk --dd='.*' suspicious_file
  Or manually: dd if=file.jpg bs=1 skip=END_OFFSET

+-------------------------------------------------------------------+
```

### Embedded Files

```
+-------------------------------------------------------------------+
|                  EMBEDDED FILES                                    |
+-------------------------------------------------------------------+

ZIP IN IMAGE:
  Many formats can contain embedded archives
  
  # Create stego file
  cat image.jpg secret.zip > stego.jpg
  
  # Image still opens normally!
  # But also openable as ZIP
  unzip stego.jpg
  
  Works because:
  - JPEG reads from beginning
  - ZIP reads from end (locates central directory)

FILE CARVING:
  Extract embedded files with tools:
  
  binwalk image.jpg
  # Shows all embedded files/signatures
  
  foremost -i image.jpg -o output/
  # Extracts all recognized file types

POLYGLOT FILES:
  Single file valid as multiple formats
  image.jpg that's also valid as:
  - HTML file
  - PDF file
  - ZIP archive
  
  Example: JS polyglot in image header

+-------------------------------------------------------------------+
```

### Slack Space and ADS

```
+-------------------------------------------------------------------+
|         SLACK SPACE AND ALTERNATE DATA STREAMS                     |
+-------------------------------------------------------------------+

DISK SLACK SPACE:
  Difference between file size and cluster size
  
  File: 100 bytes
  Cluster: 4096 bytes
  Slack: 3996 bytes of unused space
  
  Can hide data in slack space
  Not visible in directory listing
  Requires low-level disk access
  
  Tools: bmap, slacker

NTFS ALTERNATE DATA STREAMS (ADS):
  NTFS files can have multiple "streams"
  Default stream: file content
  Additional streams: hidden from normal view
  
  # Create ADS
  echo "Hidden data" > file.txt:hidden
  
  # Read ADS
  more < file.txt:hidden
  
  # List ADS
  dir /r
  
  # Or PowerShell
  Get-Item file.txt -Stream *
  
DETECTION:
  # Windows
  Get-ChildItem -Recurse | ForEach-Object { Get-Item $_.FullName -Stream * }
  
  # Tools: LADS, Streams (Sysinternals)

+-------------------------------------------------------------------+
```

---

## Metadata Hiding

### EXIF Metadata

```
+-------------------------------------------------------------------+
|                  EXIF METADATA HIDING                              |
+-------------------------------------------------------------------+

WHAT IS EXIF?
  Exchangeable Image File Format
  Metadata embedded in JPEG, TIFF, etc.
  Camera model, date, GPS, settings

HIDING IN EXIF:
  - Use comment fields
  - Create custom tags
  - Modify existing fields
  - Embed in thumbnail

EXIF FIELDS FOR HIDING:
  - ImageDescription
  - UserComment
  - Copyright
  - Artist
  - MakerNote (camera-specific, often ignored)
  - XMP data (extensible)

EXAMPLE WITH EXIFTOOL:
```

```bash
# View all metadata
exiftool image.jpg

# Hide message in comment
exiftool -comment="SECRET MESSAGE HERE" image.jpg

# Hide in UserComment
exiftool -usercomment="Hidden data" image.jpg

# Read hidden comment
exiftool -comment image.jpg

# Remove all metadata
exiftool -all= image.jpg

# Copy metadata from one file to another
exiftool -tagsFromFile source.jpg target.jpg
```

### Other Metadata Locations

```
+-------------------------------------------------------------------+
|              METADATA HIDING LOCATIONS                             |
+-------------------------------------------------------------------+

PDF METADATA:
  - Document properties (Title, Author, Subject)
  - XMP metadata
  - Custom metadata fields
  
  pdfinfo document.pdf
  exiftool document.pdf

OFFICE DOCUMENTS:
  - Document properties
  - docProps/core.xml (in ZIP structure)
  - Custom XML parts
  
  unzip -l document.docx  # See structure

AUDIO FILES (ID3 TAGS):
  - Title, Artist, Album
  - Comment fields
  - Custom frames
  - Album art (image steganography!)
  
  id3v2 --list file.mp3
  exiftool file.mp3

VIDEO METADATA:
  - Quicktime atoms
  - Matroska tags
  - MPEG headers
  
  ffprobe video.mp4

+-------------------------------------------------------------------+
```

---

## Steganography Tools

### Image Tools

```bash
# === STEGHIDE (JPEG, BMP, WAV, AU) ===
# Popular password-based steganography

# Embed data
steghide embed -cf cover.jpg -ef secret.txt -p password

# Extract data
steghide extract -sf stego.jpg -p password

# Get info (capacity, etc.)
steghide info cover.jpg

# === ZSTEG (PNG, BMP) ===
# Detection and extraction tool

# Analyze image
zsteg image.png

# Check for LSB data
zsteg -a image.png

# Extract specific channel
zsteg -E "b1,rgb,lsb" image.png

# === STEGSOLVE ===
# Java GUI tool for visual analysis

java -jar stegsolve.jar

# Features:
# - Color channel viewing
# - XOR, AND, OR between images
# - LSB extraction
# - Frame analysis (GIF)

# === OPENSTEGO ===
# Java-based steganography

# Embed
openstego embed -mf secret.txt -cf cover.png -sf output.png

# Extract
openstego extract -sf stego.png

# === OUTGUESS ===
# JPEG steganography

# Embed
outguess -k "password" -d secret.txt cover.jpg stego.jpg

# Extract
outguess -k "password" -r stego.jpg extracted.txt
```

### Analysis Tools

```bash
# === BINWALK ===
# Analyze and extract embedded files

# Scan for signatures
binwalk image.png

# Extract all recognized files
binwalk -e image.png

# Extract everything including unknown
binwalk --dd='.*' image.png

# === FOREMOST ===
# File carving tool

foremost -i image.jpg -o output_folder/

# === EXIFTOOL ===
# Metadata analysis

# View all metadata
exiftool image.jpg

# View in detail
exiftool -v3 image.jpg

# === STRINGS ===
# Find text in binary files

strings image.png
strings -n 8 image.png  # Minimum 8 chars

# === HEXDUMP / XXD ===
# Raw hex analysis

hexdump -C image.png | head -50
xxd image.png | head -50

# === FILE ===
# Identify file type

file mysterious_file
file -i mysterious_file  # MIME type
```

### Specialized Tools

```bash
# === STEGCRACKER ===
# Brute force steghide passwords

pip install stegcracker
stegcracker stego.jpg wordlist.txt

# === STEGSEEK ===
# Fast steghide cracker (much faster than stegcracker)

stegseek stego.jpg wordlist.txt

# === JSTEG ===
# JPEG steganography

# Hide
jsteg hide cover.jpg secret.txt stego.jpg

# Reveal
jsteg reveal stego.jpg output.txt

# === SNOW ===
# Whitespace steganography

# Hide
snow -C -m "secret" -p password cover.txt stego.txt

# Extract
snow -C -p password stego.txt

# === STEGDETECT ===
# Detect JPEG steganography

stegdetect -s 10.0 *.jpg
```

---

## Steganalysis and Detection

### Detection Techniques

```
+-------------------------------------------------------------------+
|                 STEGANALYSIS TECHNIQUES                            |
+-------------------------------------------------------------------+

1. VISUAL ANALYSIS
   - Compare to original (if available)
   - Look for unusual artifacts
   - Examine color channels separately
   - Use stegsolve color filters

2. STATISTICAL ANALYSIS
   - Chi-square test for LSB
   - RS steganalysis
   - Sample pair analysis
   - Histogram analysis
   
   Detects: Unnatural LSB distributions

3. SIGNATURE DETECTION
   - Look for known tool signatures
   - steghide, outguess leave traces
   - Specific patterns in headers

4. MACHINE LEARNING
   - Train classifiers on stego vs clean
   - CNNs for image steganalysis
   - High accuracy on known methods

5. FILE STRUCTURE ANALYSIS
   - Check for appended data
   - Verify file size vs content
   - Look for embedded files

+-------------------------------------------------------------------+
```

### Chi-Square Analysis

```
+-------------------------------------------------------------------+
|                CHI-SQUARE TEST FOR LSB                             |
+-------------------------------------------------------------------+

CONCEPT:
  In natural images, adjacent values have specific distribution.
  LSB steganography disrupts this distribution.
  
  Natural: Values 2k and 2k+1 should have similar frequency
  After LSB stego: Frequencies become more equal

CHI-SQUARE TEST:
  Measures deviation from expected distribution
  High chi-square value → Likely steganography

EXAMPLE:
  Color values: 100, 101, 102, 103, 104, 105
  Natural distribution: Some variation
  After LSB stego: (100,101), (102,103), (104,105) pairs equalize
  
TOOLS:
  - StegExpose: Automated detection
  - Aletheia: ML-based steganalysis
  - Commercial forensic tools

+-------------------------------------------------------------------+
```

### Detection Checklist

```bash
# === QUICK DETECTION CHECKLIST ===

# 1. Check file type
file suspicious_file

# 2. Examine metadata
exiftool suspicious_file

# 3. Look for appended data
binwalk suspicious_file

# 4. Check file size (compare to similar images)
ls -la suspicious_file

# 5. Search for strings
strings suspicious_file | less

# 6. Analyze hex
xxd suspicious_file | head -100

# 7. Try common passwords with steghide
stegseek suspicious_file rockyou.txt

# 8. Check for LSB data
zsteg suspicious_file  # For PNG

# 9. Visual analysis
# Open in stegsolve, check color channels

# 10. Compare to original (if available)
compare original.png suspicious.png diff.png
```

---

## CTF Steganography Techniques

### Common CTF Patterns

```
+-------------------------------------------------------------------+
|              CTF STEGANOGRAPHY PATTERNS                            |
+-------------------------------------------------------------------+

DIFFICULTY LEVELS:

EASY:
  - Strings hidden in file
  - Base64/hex in metadata
  - File appended after image
  - Obvious LSB encoding
  - Password is "password" or filename

MEDIUM:
  - Password-protected steghide
  - Multiple layers of encoding
  - Data in specific color channel
  - Polyglot files
  - Custom encoding schemes

HARD:
  - Custom steganography algorithms
  - Encrypted + hidden
  - Network steganography
  - Timing-based channels
  - Machine learning to decode

COMMON FLAGS LOCATIONS:
  - EXIF comments
  - LSB of red/green/blue channel
  - Alpha channel (PNG)
  - Appended data (after EOF)
  - File name/metadata
  - Embedded ZIP/RAR

+-------------------------------------------------------------------+
```

### CTF Methodology

```bash
# === CTF STEGO METHODOLOGY ===

# Step 1: Initial analysis
file challenge.png
exiftool challenge.png
strings challenge.png | grep -i flag
binwalk challenge.png

# Step 2: Check for appended data
binwalk -e challenge.png
foremost -i challenge.png -o extracted/

# Step 3: Try common tools (no password)
steghide extract -sf challenge.jpg -p ""
zsteg challenge.png

# Step 4: Try common passwords
stegseek challenge.jpg rockyou.txt
stegcracker challenge.jpg common_passwords.txt

# Step 5: Visual analysis
# Open stegsolve, check each color plane
# XOR with reference image if available

# Step 6: LSB extraction
zsteg -a challenge.png
# Or Python script for custom extraction

# Step 7: Check specific channels
python3 -c "from PIL import Image; img=Image.open('challenge.png'); print(list(img.split()[0].tobytes()[:100]))"

# Step 8: Audio analysis (if applicable)
sox challenge.wav -n spectrogram
audacity  # Check spectrogram visually

# Step 9: Network analysis (if PCAP)
tshark -r capture.pcap
# Look for unusual patterns
```

### CTF Tools Quick Reference

```bash
# === IMAGE ===
stegsolve           # Visual analysis (Java GUI)
zsteg               # PNG LSB detection
steghide            # JPEG/BMP extraction
stegseek            # Fast password cracker
pngcheck            # PNG validation
identify -verbose   # ImageMagick analysis

# === AUDIO ===
sonic-visualiser    # Spectrogram
audacity            # Audio editor
sox                 # Audio conversion
spectrology         # Hide in spectrogram

# === GENERAL ===
binwalk             # Embedded file extraction
foremost            # File carving
exiftool            # Metadata
strings             # Text extraction
xxd / hexdump       # Hex analysis
file                # Type detection

# === PASSWORD CRACKING ===
stegseek            # Fast steghide cracker
stegcracker         # Steghide brute force
john                # PDF passwords, etc.

# === ONLINE TOOLS ===
# https://www.aperisolve.com/ - Auto stego analysis
# https://stylesuxx.github.io/steganography/ - Online LSB
# https://futureboy.us/stegano/ - Various tools
```

---

## Real-World Applications

### Digital Watermarking

```
+-------------------------------------------------------------------+
|                DIGITAL WATERMARKING                                |
+-------------------------------------------------------------------+

PURPOSE:
  Prove ownership, track distribution, detect tampering

TYPES:
  Visible: Logo overlay (not true steganography)
  Invisible: Hidden watermark (steganography)

PROPERTIES NEEDED:
  - Robustness: Survives compression, scaling
  - Imperceptibility: Invisible to humans
  - Capacity: Enough to encode identifier
  - Security: Can't be removed without key

APPLICATIONS:
  - Stock photo tracking
  - Movie/music piracy tracking
  - Document authenticity
  - Broadcast monitoring

ATTACKS ON WATERMARKS:
  - Compression (JPEG attack)
  - Geometric transforms (rotation, crop)
  - Noise addition
  - Collusion (compare multiple copies)

+-------------------------------------------------------------------+
```

### Malware Communication

```
+-------------------------------------------------------------------+
|              MALWARE STEGANOGRAPHY                                 |
+-------------------------------------------------------------------+

REAL-WORLD EXAMPLES:

TURLA/SNAKE (2015):
  - Commands hidden in PNG images on websites
  - Malware downloads images, extracts commands
  - Appears as normal web traffic

STEGOLOADER:
  - Malware hidden in image files
  - Extracts and executes payload
  - Bypasses AV scanning images

DUQU 2.0:
  - Commands in JPEG XMP metadata
  - Highly sophisticated nation-state malware

ZBOT/ZEUS:
  - Config hidden in images
  - Updated via image downloads

DETECTION CHALLENGES:
  - Huge volume of image traffic
  - Hard to analyze at scale
  - False positives expensive
  - Encrypted payloads even if detected

+-------------------------------------------------------------------+
```

---

## OSCP Relevance

### OSCP Steganography Scenarios

```
+-------------------------------------------------------------------+
|              OSCP STEGANOGRAPHY SCENARIOS                          |
+-------------------------------------------------------------------+

1. PASSWORD/CREDENTIAL DISCOVERY
   - Credentials hidden in images on web servers
   - SSH keys hidden in file metadata
   - Database passwords in image comments

2. DATA EXFILTRATION DETECTION
   - Identify exfil via images
   - Check for hidden data in uploads
   - Analyze captured traffic

3. CTF-STYLE CHALLENGES
   - Some OSCP boxes have stego elements
   - Flags hidden in images
   - Clues in metadata

4. MALWARE ANALYSIS
   - Detect C2 in images
   - Extract payloads from stego
   - Analyze malware communication

5. EVIDENCE GATHERING
   - Find hidden data during forensics
   - Extract concealed information
   - Document findings

+-------------------------------------------------------------------+
```

### OSCP Commands Cheat Sheet

```bash
# === QUICK CHECKS ===
file image.png                    # Identify type
exiftool image.png               # Check metadata
strings image.png                # Find text
binwalk image.png                # Find embedded files
xxd image.png | head             # Hex analysis

# === EXTRACTION ===
binwalk -e image.png             # Extract embedded files
foremost -i image.png -o out/    # File carving
steghide extract -sf image.jpg   # Steghide extract
zsteg -a image.png               # LSB extract (PNG)

# === PASSWORD CRACKING ===
stegseek image.jpg rockyou.txt   # Fast steghide crack

# === METADATA SECRETS ===
exiftool -comment image.jpg      # Read comments
exiftool -usercomment image.jpg  # User comments
exiftool -description image.jpg  # Description field

# === CHECK FOR APPENDED DATA ===
# Compare file size to expected
# JPEG should end with FF D9
xxd image.jpg | tail
# Look for data after FF D9
```

---

## Hands-On Labs

### Lab 1: Basic LSB Steganography

```markdown
OBJECTIVE: Hide and extract message using LSB

TASKS:
1. Create a simple PNG image
2. Hide a message using Python LSB script
3. Compare original and stego image
4. Extract the message

STEPS:
# Create test image (or use any PNG)
convert -size 100x100 xc:red test.png

# Use the Python LSB script from earlier
python3 lsb_stego.py encode test.png "FLAG{lsb_is_easy}" stego.png

# Compare files
ls -la test.png stego.png
compare test.png stego.png diff.png

# Extract
python3 lsb_stego.py decode stego.png
```

### Lab 2: Steghide Password Cracking

```markdown
OBJECTIVE: Crack steghide password

SETUP:
# Create stego file with weak password
echo "SECRET FLAG" > secret.txt
steghide embed -cf image.jpg -ef secret.txt -p "password123"

TASKS:
1. Verify data is hidden
2. Crack the password
3. Extract the secret

STEPS:
# Check if steghide data exists
steghide info stego.jpg

# Crack password
stegseek stego.jpg /usr/share/wordlists/rockyou.txt

# Extract with recovered password
steghide extract -sf stego.jpg -p password123
cat secret.txt
```

### Lab 3: File Carving

```markdown
OBJECTIVE: Extract hidden files from image

SETUP:
# Create compound file
cat image.jpg secret.zip > compound.jpg

TASKS:
1. Analyze the compound file
2. Identify embedded content
3. Extract the hidden file

STEPS:
# Analyze
binwalk compound.jpg
# Should show: JPEG image + ZIP archive

# Extract
binwalk -e compound.jpg
# Or
foremost -i compound.jpg -o extracted/

# Check extracted files
ls extracted/
unzip extracted/zip/*.zip
```

### Lab 4: Metadata Analysis

```markdown
OBJECTIVE: Find hidden data in EXIF

SETUP:
exiftool -comment="BASE64:RkxBR3toaWRkZW59" image.jpg

TASKS:
1. Examine all metadata
2. Find the hidden message
3. Decode it

STEPS:
# View all metadata
exiftool image.jpg

# Find comment
exiftool -comment image.jpg
# Output: BASE64:RkxBR3toaWRkZW59

# Decode
echo "RkxBR3toaWRkZW59" | base64 -d
# Output: FLAG{hidden}
```

### Lab 5: Audio Steganography

```markdown
OBJECTIVE: Detect and extract hidden data from audio

TASKS:
1. Analyze audio file
2. Check spectrogram
3. Extract hidden message

STEPS:
# Generate spectrogram
sox audio.wav -n spectrogram -o spectrogram.png

# Visual inspection for hidden text
# Some CTFs hide text visible in spectrogram

# Check for LSB data
# Use audio LSB extraction script or tool

# Check for steghide data
steghide info audio.wav
steghide extract -sf audio.wav
```

---

## Interview Questions

### Basic Level

1. **Q: What is steganography and how does it differ from cryptography?**
   
   A: Steganography hides the existence of a message within other data (like images), while cryptography makes a message unreadable. An encrypted message is obvious but unreadable; a steganographic message is invisible but readable if found. Best security combines both.

2. **Q: What is LSB steganography?**
   
   A: Least Significant Bit steganography replaces the last bit of each pixel/sample value with message bits. Since the LSB has minimal impact on visual/audio perception, changes are undetectable to humans. A 1920×1080 RGB image can hide about 750KB of data.

3. **Q: Why is PNG preferred over JPEG for steganography?**
   
   A: PNG uses lossless compression, preserving exact pixel values including hidden LSB data. JPEG uses lossy compression that destroys LSB data during quantization. For JPEG, specialized techniques working in the DCT domain (like steghide) are required.

### Intermediate Level

4. **Q: How can you detect if an image contains hidden data?**
   
   A: Multiple approaches: 1) Statistical analysis (chi-square test) for LSB anomalies, 2) File structure analysis (binwalk) for embedded files, 3) Metadata examination (exiftool), 4) Visual analysis (stegsolve) of color channels, 5) Comparison with original if available, 6) Tool-specific signatures.

5. **Q: Explain how DNS tunneling works as a form of network steganography.**
   
   A: Data is encoded in DNS queries, typically in subdomain labels (Base32/64 encoded). The attacker's DNS server receives queries and decodes the data. Responses can carry return data in TXT or other records. This works because DNS is usually allowed through firewalls and rarely inspected deeply.

6. **Q: What are the security implications of NTFS Alternate Data Streams?**
   
   A: ADS allows hiding data in secondary file streams invisible to normal directory listings. Attackers can hide malware, exfiltrate data, or store tools. Detection requires specific commands (dir /r) or tools (Streams). Many security tools historically missed ADS content.

### Advanced Level

7. **Q: How would you detect and analyze potential steganography in a malware investigation?**
   
   A: Analysis approach: 1) Identify image downloads in network traffic, 2) Extract and analyze each image with binwalk/foremost, 3) Check metadata for anomalies, 4) Run statistical analysis for LSB detection, 5) Compare file sizes to standard compression ratios, 6) Look for known stego tool signatures, 7) Attempt extraction with common tools/passwords, 8) Analyze timing patterns if network stego suspected.

8. **Q: Describe how you would implement a robust steganography detection system.**
   
   A: Multi-layer approach: 1) Machine learning classifiers trained on known stego methods, 2) Statistical tests (chi-square, RS analysis) for LSB detection, 3) File structure analysis for embedded content, 4) Metadata anomaly detection, 5) Entropy analysis for encrypted payloads, 6) Network behavior analysis for covert channels, 7) Baseline comparison when originals available. Scale challenges require sampling strategies.

9. **Q: How can steganography be used for data exfiltration and how would you prevent it?**
   
   A: Exfil methods: Hide data in images/files uploaded to cloud services, social media, or sent via email. Prevention: 1) Content inspection with steganalysis at egress, 2) Strip metadata from outgoing files, 3) Re-encode images (breaks stego), 4) Block unnecessary file uploads, 5) Monitor for unusual image traffic patterns, 6) DLP with stego detection, 7) Limit file upload capabilities.

---

## Quick Reference

### Tool Command Reference

```bash
# === STEGHIDE ===
steghide embed -cf cover.jpg -ef secret.txt -p pass
steghide extract -sf stego.jpg -p pass
steghide info file.jpg

# === ZSTEG ===
zsteg image.png
zsteg -a image.png
zsteg -E "b1,rgb,lsb" image.png

# === BINWALK ===
binwalk file
binwalk -e file
binwalk --dd='.*' file

# === EXIFTOOL ===
exiftool file.jpg
exiftool -comment file.jpg
exiftool -all= file.jpg

# === STEGSEEK ===
stegseek stego.jpg wordlist.txt

# === FOREMOST ===
foremost -i file -o output/

# === STRINGS ===
strings file
strings -n 10 file

# === FILE ===
file mystery
file -i mystery
```

### File Signatures Reference

```
+-------------------------------------------------------------------+
|                 FILE SIGNATURES (Magic Bytes)                      |
+-------------------------------------------------------------------+

FORMAT     | SIGNATURE (HEX)      | ASCII
-----------|----------------------|-------------
JPEG       | FF D8 FF             | ÿØÿ
PNG        | 89 50 4E 47          | ‰PNG
GIF        | 47 49 46 38          | GIF8
BMP        | 42 4D                | BM
PDF        | 25 50 44 46          | %PDF
ZIP        | 50 4B 03 04          | PK..
RAR        | 52 61 72 21          | Rar!
7Z         | 37 7A BC AF          | 7z¼¯
GZIP       | 1F 8B                | ..
ELF        | 7F 45 4C 46          | .ELF
EXE (MZ)   | 4D 5A                | MZ
MP3        | 49 44 33 or FF FB    | ID3 or ÿû
WAV        | 52 49 46 46          | RIFF
AVI        | 52 49 46 46          | RIFF
MP4        | 00 00 00 (varies)    | ftyp at +4

END MARKERS:
JPEG       | FF D9                | ÿÙ
PNG        | 49 45 4E 44 AE 42 60 82 | IEND®B`‚

+-------------------------------------------------------------------+
```

### Steganalysis Checklist

```
+-------------------------------------------------------------------+
|              STEGANALYSIS CHECKLIST                                |
+-------------------------------------------------------------------+

INITIAL ANALYSIS:
[ ] file command - identify type
[ ] exiftool - check metadata
[ ] strings - find text
[ ] binwalk - find embedded files
[ ] xxd | head - examine header

FILE STRUCTURE:
[ ] Check file signature matches extension
[ ] Compare size to expected
[ ] Look for data after EOF marker
[ ] Check for appended archives

METADATA:
[ ] EXIF comments
[ ] XMP data
[ ] Document properties
[ ] Audio tags

CONTENT ANALYSIS:
[ ] LSB extraction (zsteg for PNG)
[ ] Visual inspection (stegsolve)
[ ] Spectrogram (audio)
[ ] Color channel analysis

PASSWORD CRACKING:
[ ] Try empty password
[ ] Try filename
[ ] Common passwords
[ ] Wordlist attack (stegseek)

EXTRACTION:
[ ] steghide extract
[ ] binwalk -e
[ ] foremost
[ ] Custom scripts

+-------------------------------------------------------------------+
```

---

## Related Files

- [[02_Encoding_Fundamentals]] - Base64, hex encoding used in stego
- [[03_Hashing_and_Integrity]] - File integrity verification
- [[11_Obfuscation]] - Related data hiding techniques
- [[08_Cryptographic_Attacks]] - Combined encryption + steganography

---

## Resources

### Tools
- [Steghide](http://steghide.sourceforge.net/)
- [zsteg](https://github.com/zed-0xff/zsteg)
- [stegseek](https://github.com/RickdeJager/stegseek)
- [StegSolve](https://github.com/eugenekolo/sec-tools/tree/master/stego/stegsolve)
- [binwalk](https://github.com/ReFirmLabs/binwalk)

### Learning Resources
- [Steganography - A list of useful tools and resources](https://0xrick.github.io/lists/stego/)
- [CTF Steganography Checklist](https://fareedfauzi.gitbook.io/ctf-checklist/steganography)
- [Aperisolve](https://www.aperisolve.com/) - Online stego analyzer

### Academic
- [Steganography in Digital Media](https://www.cambridge.org/core/books/steganography-in-digital-media/)
- [Digital Watermarking and Steganography](https://www.elsevier.com/books/digital-watermarking-and-steganography/katzenbeisser/978-0-12-372585-1)

---

*Last updated: 2025-02-07*
