
# 09 - Encryption at Rest

> **"Data at rest is a sitting target. Encryption makes it a useless target."**

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [Absolute Beginner's Guide](#absolute-beginners-guide)
3. [Full Disk Encryption Concepts](#full-disk-encryption-concepts)
4. [Windows BitLocker](#windows-bitlocker)
5. [Linux LUKS/dm-crypt](#linux-lukspm-crypt)
6. [macOS FileVault](#macos-filevault)
7. [Hardware Encryption](#hardware-encryption)
8. [TPM and Secure Boot](#tpm-and-secure-boot)
9. [Attacking Encryption at Rest](#attacking-encryption-at-rest)
10. [Cold Boot Attacks](#cold-boot-attacks)
11. [Evil Maid Attacks](#evil-maid-attacks)
12. [Key Recovery Methods](#key-recovery-methods)
13. [Cloud Encryption at Rest](#cloud-encryption-at-rest)
14. [Database Encryption](#database-encryption)
15. [File-Level Encryption](#file-level-encryption)
16. [OSCP Relevance](#oscp-relevance)
17. [Hands-On Labs](#hands-on-labs)
18. [Interview Questions](#interview-questions)
19. [Quick Reference](#quick-reference)

---

## Executive Summary

**Encryption at rest** protects data stored on disks, databases, and files from unauthorized access when systems are powered off or storage media is stolen. Unlike encryption in transit (TLS), encryption at rest secures data in its stored state.

```
+-------------------------------------------------------------------+
|                  ENCRYPTION AT REST OVERVIEW                       |
+-------------------------------------------------------------------+

THREAT MODEL:
  - Laptop theft
  - Discarded/sold hard drives
  - Physical access to servers
  - Unauthorized datacenter access
  - Forensic analysis

PROTECTION LAYERS:
  
  +-----------------------+
  | Application Layer     |  Database TDE, app-level encryption
  +-----------------------+
  | File System Layer     |  EFS, eCryptfs, encrypted folders
  +-----------------------+
  | Volume/Partition      |  LUKS, BitLocker volume encryption
  +-----------------------+
  | Full Disk (FDE)       |  BitLocker, FileVault, LUKS full disk
  +-----------------------+
  | Hardware (SED)        |  Self-Encrypting Drives
  +-----------------------+

+-------------------------------------------------------------------+
```

---

## Absolute Beginner's Guide

### What is "Encryption at Rest"?

**Analogy: The Safety Deposit Box**
```
WITHOUT Encryption:
  Your hard drive is like a filing cabinet.
  Anyone who gets physical access can read everything.
  
  Thief steals laptop → Removes hard drive → Reads all files

WITH Encryption:
  Your hard drive is like a safety deposit box.
  Even if stolen, contents are scrambled without the key.
  
  Thief steals laptop → Removes hard drive → Sees random garbage
```

### How Does It Work?

```
+-------------------------------------------------------------------+
|                    ENCRYPTION AT REST FLOW                         |
+-------------------------------------------------------------------+

WRITING DATA:
                                    
  Plain File  →  Encryption  →  Encrypted File
  "secret.txt"   Algorithm      (stored on disk)
       |              |
       v              v
  "My password"   AES-256 + Key  →  "x9#kL@..."

READING DATA:

  Encrypted File  →  Decryption  →  Plain File
  "x9#kL@..."        Algorithm      "My password"
                          |
                          v
                    Requires same key!

KEY STORAGE:
  - Password → derives key
  - TPM chip → stores/seals key
  - USB token → contains key
  - Recovery key → backup access

+-------------------------------------------------------------------+
```

### Types of Encryption at Rest

```
+-------------------------------------------------------------------+
|              TYPES OF ENCRYPTION AT REST                           |
+-------------------------------------------------------------------+

1. FULL DISK ENCRYPTION (FDE)
   Everything encrypted, including OS
   Examples: BitLocker, FileVault, LUKS
   
   [Boot Partition] [===================== Encrypted =====================]
   (small, unencrypted)       (entire OS + all data)

2. PARTITION/VOLUME ENCRYPTION
   Specific partitions encrypted
   Examples: BitLocker (volume), LUKS container
   
   [Windows C:] [Data D: Encrypted] [Backup E: Encrypted]

3. FILE/FOLDER ENCRYPTION
   Individual files or folders encrypted
   Examples: EFS, VeraCrypt containers, 7-Zip encryption
   
   [OS] [Normal Files] [📁 Encrypted Folder] [Normal Files]

4. DATABASE ENCRYPTION (TDE)
   Database files encrypted transparently
   Examples: SQL Server TDE, Oracle TDE
   
   [OS] [App Files] [Database Files: Encrypted]

5. SELF-ENCRYPTING DRIVE (SED)
   Hardware encrypts everything automatically
   Examples: OPAL 2.0 compliant SSDs
   
   [================ Hardware Encrypted ================]
   (CPU never sees unencrypted data on bus)

+-------------------------------------------------------------------+
```

### Why Encryption at Rest Matters

```
SCENARIO 1: Laptop Theft
  Employee laptop stolen from car.
  
  WITHOUT FDE: Thief boots from USB, reads all corporate data
  WITH FDE: Thief sees encrypted garbage, data safe
  
SCENARIO 2: Decommissioned Server
  Old server sold on eBay.
  
  WITHOUT encryption: New owner recovers customer data
  WITH encryption: Data is unreadable without key
  
SCENARIO 3: Cloud Provider Breach
  AWS datacenter physically breached.
  
  WITHOUT encryption: Attacker reads EBS volumes
  WITH client-managed keys: Data encrypted, keys not on AWS

SCENARIO 4: Insider Threat
  Malicious admin copies database files.
  
  WITHOUT TDE: Admin reads all data
  WITH TDE: Files encrypted, admin needs key access
```

---

## Full Disk Encryption Concepts

### FDE Architecture

```
+-------------------------------------------------------------------+
|                    FDE ARCHITECTURE                                |
+-------------------------------------------------------------------+

                      +-----------------+
                      |  User Password  |
                      +-----------------+
                              |
                              v
                      +-----------------+
                      |  Key Derivation |  (PBKDF2, Argon2)
                      |  Function       |
                      +-----------------+
                              |
                              v
                      +-----------------+
                      |  Key Encryption |  (KEK)
                      |  Key            |
                      +-----------------+
                              |
           Decrypt          |
              +-------------+
              |
              v
      +-----------------+
      | Volume Master   |  (Stored encrypted on disk)
      | Key (VMK)       |
      +-----------------+
              |
              v
      +-----------------+
      |  Full Volume    |  (Actual encryption key)
      |  Encryption Key |
      |  (FVEK)         |
      +-----------------+
              |
              v
      +===================+
      | Encrypted Data    |
      +===================+

WHY MULTIPLE KEYS?
  - Change password without re-encrypting entire disk
  - Multiple unlock methods (password, TPM, recovery)
  - Key escrow and recovery options

+-------------------------------------------------------------------+
```

### Pre-Boot Authentication

```
+-------------------------------------------------------------------+
|                 PRE-BOOT AUTHENTICATION                            |
+-------------------------------------------------------------------+

BOOT SEQUENCE WITH FDE:

1. Power On
   |
   v
2. BIOS/UEFI initializes
   |
   v
3. Bootloader loads (from small unencrypted partition)
   |
   v
4. PRE-BOOT AUTHENTICATION ← User enters password/PIN
   |                         or TPM unlocks automatically
   v
5. Encryption key derived/unsealed
   |
   v
6. Encrypted volume unlocked
   |
   v
7. Operating system boots normally
   |
   v
8. User sees login screen

ATTACK SURFACE:
  - Bootloader is unencrypted (can be modified)
  - Pre-boot environment has limited security
  - Cold boot: keys in RAM after unlock

+-------------------------------------------------------------------+
```

### Encryption Algorithms Used

```
+-------------------------------------------------------------------+
|              COMMON FDE ENCRYPTION ALGORITHMS                      |
+-------------------------------------------------------------------+

ALGORITHM    | KEY SIZE  | MODE      | USED BY
-------------|-----------|-----------|---------------------------
AES-128      | 128-bit   | XTS       | BitLocker (compatible mode)
AES-256      | 256-bit   | XTS       | BitLocker, FileVault, LUKS
AES-256      | 256-bit   | CBC-ESSIV | Older LUKS, Android
Serpent      | 256-bit   | XTS       | VeraCrypt option
Twofish      | 256-bit   | XTS       | VeraCrypt option

XTS MODE (XEX-based Tweaked-codebook with Stealing):
  - Designed specifically for disk encryption
  - Each sector encrypted with unique tweak
  - Prevents ECB-style pattern attacks
  - Industry standard for FDE

Why XTS, not CBC or GCM?
  - No IV storage needed (sector number = tweak)
  - No expansion (ciphertext same size as plaintext)
  - Random access friendly
  - No authentication (too expensive per-sector)

+-------------------------------------------------------------------+
```

---

## Windows BitLocker

### BitLocker Overview

```
+-------------------------------------------------------------------+
|                    BITLOCKER OVERVIEW                              |
+-------------------------------------------------------------------+

WHAT IS BITLOCKER?
  Microsoft's full disk encryption solution
  Built into Windows Pro, Enterprise, Education
  Supports TPM, password, USB key, smart card

PROTECTION MODES:
  1. TPM only - Automatic unlock (transparent)
  2. TPM + PIN - Requires PIN at boot
  3. TPM + USB - Requires USB key at boot
  4. Password only - No TPM required
  5. USB key only - No TPM required

ENCRYPTION:
  - AES-128 or AES-256
  - XTS mode (default since Windows 10)
  - CBC mode (legacy compatibility)

RECOVERY:
  - 48-digit recovery key
  - Can be stored in AD, Azure AD, Microsoft Account
  - Recovery password or recovery key file

+-------------------------------------------------------------------+
```

### BitLocker Architecture

```
+-------------------------------------------------------------------+
|                 BITLOCKER KEY HIERARCHY                            |
+-------------------------------------------------------------------+

                    +------------------+
                    |  Recovery Key    |  48 digits
                    |  (Backup)        |  Stored in AD/Azure/File
                    +------------------+
                            |
                            v (can decrypt VMK)
+------------+      +------------------+      +------------+
| TPM Sealed |  or  | Password/PIN     |  or  |  USB Key   |
| Key        |      | Derived Key      |      |            |
+------------+      +------------------+      +------------+
      \                    |                       /
       \                   |                      /
        v                  v                     v
              +------------------------+
              |  Volume Master Key     |
              |  (VMK)                 |
              +------------------------+
                          |
                          v
              +------------------------+
              |  Full Volume           |
              |  Encryption Key (FVEK) |
              +------------------------+
                          |
                          v
              +------------------------+
              |  ENCRYPTED VOLUME      |
              +------------------------+

KEY PROTECTORS:
  Each method (TPM, PIN, recovery key) can unlock VMK
  Multiple protectors can coexist

+-------------------------------------------------------------------+
```

### BitLocker Commands

```powershell
# === BITLOCKER STATUS ===
# Check BitLocker status
manage-bde -status

# Detailed status of C: drive
manage-bde -status C:

# === ENABLE BITLOCKER ===
# Enable with TPM only
manage-bde -on C: -RecoveryPassword

# Enable with TPM + PIN
manage-bde -on C: -TPMAndPIN -RecoveryPassword

# Enable with password only (no TPM)
manage-bde -on C: -Password -RecoveryPassword

# === KEY PROTECTORS ===
# List key protectors
manage-bde -protectors -get C:

# Add recovery password
manage-bde -protectors -add C: -RecoveryPassword

# Add TPM protector
manage-bde -protectors -add C: -TPM

# Add PIN protector (requires TPM)
manage-bde -protectors -add C: -TPMAndPIN

# Backup recovery key to AD
manage-bde -protectors -adbackup C: -id {KEY-PROTECTOR-ID}

# === RECOVERY ===
# Unlock with recovery password
manage-bde -unlock C: -RecoveryPassword 123456-789012-...

# Unlock with recovery key file
manage-bde -unlock C: -RecoveryKey E:\RecoveryKey.bek

# === DISABLE/SUSPEND ===
# Suspend BitLocker (for updates)
manage-bde -protectors -disable C:

# Resume protection
manage-bde -protectors -enable C:

# Decrypt volume (disable BitLocker)
manage-bde -off C:
```

### BitLocker Metadata Locations

```
+-------------------------------------------------------------------+
|                BITLOCKER METADATA LOCATIONS                        |
+-------------------------------------------------------------------+

ON DISK:
  - BitLocker metadata stored in volume header
  - Three copies for redundancy
  - Contains encrypted VMK, salt, algorithm info
  
  Offset 0x00: FVE metadata block 1
  Offset varies: FVE metadata block 2
  Offset varies: FVE metadata block 3

IN REGISTRY (when unlocked):
  HKLM\SYSTEM\CurrentControlSet\Control\FVE

IN MEMORY (when unlocked):
  FVEK loaded into kernel memory
  Can be extracted with memory forensics

ACTIVE DIRECTORY:
  ms-FVE-RecoveryInformation attribute
  Stored under computer object
  Contains recovery key

AZURE AD:
  BitLocker recovery keys in device properties
  Accessible via portal or Graph API

+-------------------------------------------------------------------+
```

### BitLocker Attack Surface

```
+-------------------------------------------------------------------+
|                BITLOCKER ATTACK VECTORS                            |
+-------------------------------------------------------------------+

1. TPM-ONLY MODE VULNERABILITY
   If only TPM protects BitLocker:
     - Boot process completes automatically
     - Attacker with physical access can boot to login
     - DMA attacks possible (Thunderbolt, FireWire)
     - PCILeech attacks
   
   MITIGATION: Use TPM + PIN or TPM + USB

2. COLD BOOT ATTACK
   RAM retains data briefly after power off
   Boot into attacker OS, dump RAM, extract FVEK
   
   MITIGATION: Disable sleep, enable secure boot

3. EVIL MAID ATTACK
   Replace bootloader with malicious version
   Capture password/PIN entered by user
   
   MITIGATION: Secure Boot, TPM PCR validation

4. RECOVERY KEY THEFT
   Recovery keys stored in AD/Azure AD
   Attacker compromises AD → gets all recovery keys
   
   MITIGATION: Restrict access to recovery keys

5. DMA ATTACKS
   Direct Memory Access via Thunderbolt/FireWire
   Read encryption keys from RAM
   
   MITIGATION: Disable DMA ports, kernel DMA protection

+-------------------------------------------------------------------+
```

---

## Linux LUKS/dm-crypt

### LUKS Overview

```
+-------------------------------------------------------------------+
|                      LUKS OVERVIEW                                 |
+-------------------------------------------------------------------+

WHAT IS LUKS?
  Linux Unified Key Setup
  Standard for Linux disk encryption
  Uses dm-crypt kernel module

KEY FEATURES:
  - Multiple key slots (up to 8)
  - Standardized header format
  - Supports passphrase, keyfile, or both
  - LUKS2 adds Argon2 KDF support

VERSIONS:
  LUKS1: PBKDF2 key derivation
  LUKS2: Argon2i/Argon2id support, better metadata

ENCRYPTION OPTIONS:
  - AES-256-XTS (default, recommended)
  - Serpent, Twofish available
  - Various modes: XTS, CBC-ESSIV

+-------------------------------------------------------------------+
```

### LUKS Architecture

```
+-------------------------------------------------------------------+
|                    LUKS DISK LAYOUT                                |
+-------------------------------------------------------------------+

+-------------+-------------+----------------------------------+
|   LUKS      |   Key       |      Encrypted Data              |
|   Header    |   Slots     |      (dm-crypt volume)           |
+-------------+-------------+----------------------------------+
|  592 bytes  | 8 × 256KB   |       Rest of partition          |
| (LUKS1)     | (LUKS1)     |                                  |
+-------------+-------------+----------------------------------+

LUKS HEADER CONTAINS:
  - Magic bytes ("LUKS\xba\xbe")
  - Version number
  - Cipher name and mode
  - Hash specification
  - Master key digest
  - Salt for each key slot
  - Key slot data

KEY DERIVATION:

  Passphrase  →  PBKDF2/Argon2  →  Key Encryption Key
                      |
                      v
              Decrypt Key Slot
                      |
                      v
              Master Key (for dm-crypt)

+-------------------------------------------------------------------+
```

### LUKS Commands (cryptsetup)

```bash
# === CREATE ENCRYPTED VOLUME ===
# Create LUKS container
cryptsetup luksFormat /dev/sdb1

# Create with specific options
cryptsetup luksFormat --type luks2 \
  --cipher aes-xts-plain64 \
  --key-size 512 \
  --hash sha512 \
  --pbkdf argon2id \
  /dev/sdb1

# === OPEN/CLOSE VOLUME ===
# Open (decrypt) LUKS volume
cryptsetup luksOpen /dev/sdb1 secure_volume
# Creates /dev/mapper/secure_volume

# Close (lock) volume
cryptsetup luksClose secure_volume

# === KEY MANAGEMENT ===
# Add new passphrase (uses empty slot)
cryptsetup luksAddKey /dev/sdb1

# Add keyfile
cryptsetup luksAddKey /dev/sdb1 /path/to/keyfile

# Remove passphrase
cryptsetup luksRemoveKey /dev/sdb1

# Kill specific key slot
cryptsetup luksKillSlot /dev/sdb1 2

# === HEADER OPERATIONS ===
# Dump header information
cryptsetup luksDump /dev/sdb1

# Backup header
cryptsetup luksHeaderBackup /dev/sdb1 --header-backup-file header.img

# Restore header
cryptsetup luksHeaderRestore /dev/sdb1 --header-backup-file header.img

# === STATUS ===
# Check if LUKS
cryptsetup isLuks /dev/sdb1 && echo "Is LUKS"

# Show status of open volume
cryptsetup status secure_volume

# === BENCHMARK ===
# Benchmark encryption performance
cryptsetup benchmark
```

### LUKS Header Analysis

```bash
# Detailed header dump
$ cryptsetup luksDump /dev/sdb1

LUKS header information
Version:        2
Epoch:          3
Metadata area:  16384 [bytes]
Keyslots area:  16744448 [bytes]
UUID:           a1b2c3d4-e5f6-7890-abcd-ef1234567890

Keyslots:
  0: luks2
        Key:        512 bits
        Priority:   normal
        Cipher:     aes-xts-plain64
        PBKDF:      argon2id
        Hash:       sha256
        
  1: luks2
        Key:        512 bits
        Priority:   normal
        Cipher:     aes-xts-plain64
        PBKDF:      argon2id

Tokens:
Digests:
  0: pbkdf2
        Hash:       sha256
```

### LUKS Attack Considerations

```
+-------------------------------------------------------------------+
|                  LUKS ATTACK VECTORS                               |
+-------------------------------------------------------------------+

1. WEAK PASSPHRASE
   Short or common passwords can be brute-forced
   
   Extract header: dd if=/dev/sdb1 of=luks_header.img bs=1M count=10
   Crack: hashcat -m 14600 luks_header.img wordlist.txt

2. HEADER DESTRUCTION
   LUKS header destruction = permanent data loss
   
   dd if=/dev/urandom of=/dev/sdb1 bs=1M count=10
   
   MITIGATION: Always backup headers!

3. KEYFILE EXPOSURE
   If keyfile is readable, volume is compromised
   
   MITIGATION: Proper permissions (600), separate storage

4. COLD BOOT ATTACK
   Master key in RAM when volume mounted
   
   MITIGATION: Use kernel key retention protection

5. HEADER TAMPERING
   Modify LUKS header to capture passphrase
   (Like evil maid, but for LUKS)
   
   MITIGATION: LUKS2 has header integrity check

+-------------------------------------------------------------------+
```

### LUKS Password Cracking

```bash
# Extract LUKS header for cracking
dd if=/dev/sdb1 of=luks_header.img bs=512 count=4097

# Or for LUKS2 (larger header)
dd if=/dev/sdb1 of=luks_header.img bs=1M count=16

# Crack with hashcat
# Mode 14600 = LUKS (legacy)
hashcat -m 14600 luks_header.img wordlist.txt

# Mode 29511 = LUKS2 (Argon2)
hashcat -m 29511 luks_header.img wordlist.txt

# Crack with John the Ripper
luks2john luks_header.img > luks_hash.txt
john luks_hash.txt --wordlist=rockyou.txt
```

---

## macOS FileVault

### FileVault Overview

```
+-------------------------------------------------------------------+
|                    FILEVAULT OVERVIEW                              |
+-------------------------------------------------------------------+

WHAT IS FILEVAULT?
  Apple's full disk encryption for macOS
  FileVault 2 (current) introduced in macOS Lion

FEATURES:
  - XTS-AES-128 encryption
  - Instant wipe via key destruction
  - Recovery key (24 characters)
  - iCloud recovery option
  - Institutional recovery keys

ARCHITECTURE:
  - CoreStorage (older Macs)
  - APFS (Apple File System) native encryption (modern)

KEY MANAGEMENT:
  - Keys derived from user password
  - Stored in Secure Enclave (T2/Apple Silicon)
  - Recovery key as backup

+-------------------------------------------------------------------+
```

### FileVault Commands

```bash
# Check FileVault status
sudo fdesetup status

# Enable FileVault
sudo fdesetup enable

# Enable with recovery key to file
sudo fdesetup enable -outputplist > recovery.plist

# Add user to FileVault
sudo fdesetup add -usertoadd username

# Remove user from FileVault
sudo fdesetup remove -user username

# List enabled users
sudo fdesetup list

# Show recovery key (if personal)
sudo fdesetup showrecoverykey

# Validate recovery key
sudo fdesetup validaterecovery

# Disable FileVault
sudo fdesetup disable
```

### FileVault Attack Considerations

```
+-------------------------------------------------------------------+
|                FILEVAULT SECURITY                                  |
+-------------------------------------------------------------------+

T2/APPLE SILICON MACS:
  - Encryption keys stored in Secure Enclave
  - Hardware brute-force protection
  - Cannot extract keys via DMA
  - Extremely difficult to attack

OLDER MACS (No T2):
  - Keys protected by software
  - DMA attacks possible (Thunderbolt)
  - Cold boot attacks feasible
  - Filevault password bruteforce possible

RECOVERY KEY:
  - 24-character alphanumeric
  - Can be escrowed to iCloud or MDM
  - If attacker gets recovery key = game over

FIRMWARE PASSWORD:
  - Prevents booting from external media
  - Can be reset by Apple (T2 Macs)
  - Important layer of defense

+-------------------------------------------------------------------+
```

---

## Hardware Encryption

### Self-Encrypting Drives (SED)

```
+-------------------------------------------------------------------+
|              SELF-ENCRYPTING DRIVES (SED)                          |
+-------------------------------------------------------------------+

WHAT IS SED?
  Drive hardware performs encryption
  CPU never sees unencrypted data
  Keys stored in drive controller

STANDARDS:
  - TCG Opal 2.0 (most common)
  - IEEE 1667
  - ATA Security

BENEFITS:
  + No CPU overhead
  + Instant secure erase (destroy key)
  + Works with any OS
  + Encrypts everything including swap

RISKS:
  - Implementation bugs (several found)
  - Manufacturer backdoors possible
  - Some SEDs use weak crypto
  - Hardware attacks on controller

+-------------------------------------------------------------------+
```

### SED Architecture

```
+-------------------------------------------------------------------+
|                    SED ARCHITECTURE                                |
+-------------------------------------------------------------------+

                    +------------------+
                    |   Host System    |
                    |   (Computer)     |
                    +--------+---------+
                             |
                    Commands (READ/WRITE)
                             |
                             v
           +----------------------------------+
           |     Drive Controller             |
           |  +---------------------------+   |
           |  | Authentication Handler    |   |
           |  +---------------------------+   |
           |              |                   |
           |  +---------------------------+   |
           |  | AES Encryption Engine     |   |
           |  | (Hardware)                |   |
           |  +---------------------------+   |
           |              |                   |
           |  +---------------------------+   |
           |  | Key Storage (locked)      |   |
           |  +---------------------------+   |
           +----------------------------------+
                             |
                             v
                    +------------------+
                    | Encrypted Data   |
                    | (Platters/NAND)  |
                    +------------------+

AUTHENTICATION:
  - Password unlocks drive at boot
  - Without password, data is encrypted garbage
  - Secure erase = delete key = instant wipe

+-------------------------------------------------------------------+
```

### SED Vulnerabilities

```
+-------------------------------------------------------------------+
|                 SED VULNERABILITIES                                |
+-------------------------------------------------------------------+

RESEARCH: "Self-Encrypting Deception" (2018)
  Researchers found critical flaws in popular SEDs

VULNERABILITY 1: DEK Not Derived from Password
  Some SEDs: Password unlocks drive, but DEK is static
  Attack: Bypass password check in firmware → full access

VULNERABILITY 2: Wear Leveling Leaks
  SSD wear leveling may leave unencrypted data remnants

VULNERABILITY 3: Debug Interfaces
  JTAG/debug ports on controller chip
  Direct access to keys possible

VULNERABILITY 4: Weak Key Generation
  Some drives use weak/predictable keys

AFFECTED DRIVES (2018 research):
  - Crucial MX100, MX200, MX300
  - Samsung 840 EVO, 850 EVO, T3, T5
  
RECOMMENDATIONS:
  - Don't rely on SED alone
  - Layer with software encryption (BitLocker + SED)
  - Use drives with FIPS 140-2 certification

+-------------------------------------------------------------------+
```

---

## TPM and Secure Boot

### TPM Overview

```
+-------------------------------------------------------------------+
|              TRUSTED PLATFORM MODULE (TPM)                         |
+-------------------------------------------------------------------+

WHAT IS TPM?
  Hardware security chip on motherboard
  Stores cryptographic keys securely
  Performs crypto operations

KEY FEATURES:
  - Secure key storage
  - Platform Configuration Registers (PCRs)
  - Sealing/unsealing operations
  - Random number generation
  - Remote attestation

VERSIONS:
  - TPM 1.2: Legacy, SHA-1 based
  - TPM 2.0: Current, supports SHA-256, more algorithms

USE CASES:
  - BitLocker key storage
  - Secure Boot validation
  - Windows Hello/FIDO2
  - Disk encryption sealing

+-------------------------------------------------------------------+
```

### TPM and BitLocker Integration

```
+-------------------------------------------------------------------+
|                TPM + BITLOCKER FLOW                                |
+-------------------------------------------------------------------+

SEALING (Initial Setup):
  
  1. BitLocker enabled
  2. TPM measures boot components into PCRs:
     - BIOS/UEFI
     - Boot loader
     - Boot configuration
  3. VMK encrypted to current PCR state
  4. Sealed VMK stored on disk

UNSEALING (Every Boot):

  1. System powers on
  2. TPM measures boot components
  3. TPM compares current PCRs to sealed state
     |
     +-- Match? → VMK released → BitLocker unlocks
     |
     +-- Different? → VMK denied → Prompt for recovery key

PCR VALUES:
  PCR 0: BIOS/UEFI firmware
  PCR 2: Option ROMs
  PCR 4: MBR (Master Boot Record)
  PCR 5: MBR partition table
  PCR 7: Secure Boot state
  PCR 8: Boot loader (NTFS boot sector)
  PCR 9: Boot loader (NTFS boot block)
  PCR 10: Boot manager
  PCR 11: BitLocker access control

+-------------------------------------------------------------------+
```

### TPM Attacks

```
+-------------------------------------------------------------------+
|                     TPM ATTACKS                                    |
+-------------------------------------------------------------------+

1. TPM SNIFFING (TPM Genie)
   TPM communicates via LPC/SPI bus
   Physical interception of bus = key extraction
   
   Attack: Probe TPM pins, capture VMK during boot
   
   Cost: $10-50 hardware
   Skill: Moderate electronics knowledge

2. COLD BOOT BYPASS
   TPM unseals key → key loaded in RAM
   RAM still readable after power loss
   
   Attack: Boot, let TPM unseal, cold boot attack

3. PCR MANIPULATION
   If boot chain can be modified without PCR change
   Attacker boots malicious OS with valid PCRs
   
   Attack: Exploit bootloader vulnerabilities

4. TPM RESET ATTACK
   Reset TPM to factory state while system running
   May expose secrets in some implementations

MITIGATIONS:
  - Use TPM + PIN (not TPM only)
  - Enable Secure Boot
  - Disable sleep/hibernation
  - Physical tamper evidence

+-------------------------------------------------------------------+
```

### Secure Boot

```
+-------------------------------------------------------------------+
|                     SECURE BOOT                                    |
+-------------------------------------------------------------------+

PURPOSE:
  Verify boot components are signed
  Prevent unauthorized code execution at boot

TRUST CHAIN:
  
  UEFI Firmware (contains Microsoft key)
         |
         v
  Verifies: Boot Loader signature
         |
         v
  Boot Loader (shim, GRUB, Windows Boot Manager)
         |
         v
  Verifies: Kernel signature
         |
         v
  OS Kernel loads

KEYS:
  - Platform Key (PK): Platform owner
  - Key Exchange Key (KEK): Can modify db
  - db: Allowed signatures
  - dbx: Forbidden signatures (revoked)

BYPASS METHODS:
  - Disable Secure Boot in BIOS
  - Add attacker key to db
  - Exploit signed bootloader (BootHole CVE-2020-10713)
  - Exploit signed driver for kernel access

+-------------------------------------------------------------------+
```

---

## Attacking Encryption at Rest

### Attack Decision Tree

```
+-------------------------------------------------------------------+
|           ENCRYPTION AT REST ATTACK DECISION TREE                  |
+-------------------------------------------------------------------+

                    Encrypted Volume
                          |
              +-----------+-----------+
              |                       |
         System OFF              System ON
              |                       |
    +--------------------+    +--------------------+
    | Recovery Key       |    | Memory Extraction  |
    | - AD/Azure lookup  |    | - Cold boot        |
    | - Backup files     |    | - DMA attack       |
    | - User's storage   |    | - Hibernation file |
    | - Social engineer  |    | - Live forensics   |
    +--------------------+    +--------------------+
              |                       |
              v                       v
    +--------------------+    +--------------------+
    | Password Attack    |    | Key Extraction     |
    | - Extract header   |    | - Volatility       |
    | - hashcat/john     |    | - PCILeech         |
    | - Rule-based       |    | - Inception        |
    +--------------------+    +--------------------+
              |                       |
              v                       v
    +--------------------+    +--------------------+
    | Evil Maid Attack   |    | Privilege Escalation|
    | - Bootloader mod   |    | - Gain admin/root  |
    | - Hardware keylog  |    | - Access mounted   |
    +--------------------+    | - volumes          |
                              +--------------------+

+-------------------------------------------------------------------+
```

### Password/Passphrase Attacks

```bash
# === BITLOCKER PASSWORD CRACKING ===
# Extract metadata
bitlocker2john -i encrypted_volume.dd > bitlocker_hash.txt

# Using Hashcat
# Mode 22100 = BitLocker
hashcat -m 22100 bitlocker_hash.txt wordlist.txt

# With rules for better coverage
hashcat -m 22100 -r best64.rule bitlocker_hash.txt wordlist.txt

# === LUKS PASSWORD CRACKING ===
# Extract header
dd if=/dev/sdb1 of=luks_header.img bs=512 count=4097

# hashcat LUKS
hashcat -m 14600 luks_header.img wordlist.txt

# John the Ripper
luks2john luks_header.img > luks_hash.txt
john luks_hash.txt --wordlist=rockyou.txt

# === FILEVAULT CRACKING ===
# Extract hash (from disk image)
fvde2john disk_image.dmg > filevault_hash.txt
john filevault_hash.txt --wordlist=rockyou.txt

# === VERACRYPT ===
# hashcat mode 13711-13713, 13721-13723, 13731-13733, 13751-13753
hashcat -m 13721 veracrypt_header wordlist.txt
```

---

## Cold Boot Attacks

### Cold Boot Attack Theory

```
+-------------------------------------------------------------------+
|                   COLD BOOT ATTACK                                 |
+-------------------------------------------------------------------+

PRINCIPLE:
  DRAM retains data for seconds to minutes after power loss
  Cooling with compressed air extends retention significantly

ATTACK STEPS:

  1. Target system running with decrypted volume
     [RAM contains encryption keys]
     
  2. Power off system (or force reset)
     
  3. IMMEDIATELY boot from attacker USB
     (Before RAM decays)
     
  4. OR: Remove RAM, place in attacker system
     (Cooling keeps data longer)
     
  5. Dump RAM contents
     
  6. Search for encryption keys in dump

KEY FINDING:
  - AES key schedules have recognizable patterns
  - Tools can locate keys in memory dumps

+-------------------------------------------------------------------+
```

### Cold Boot Attack Tools

```bash
# === MEMORY ACQUISITION ===
# Linux: Using dd from live forensic USB
dd if=/dev/mem of=/media/usb/memdump.img

# Using LiME (Linux Memory Extractor)
insmod lime-4.x.x.ko "path=/media/usb/memdump.lime format=lime"

# === WINDOWS MEMORY ANALYSIS ===
# Using Volatility
volatility -f memdump.img --profile=Win10x64 modules
volatility -f memdump.img --profile=Win10x64 dumpfiles

# === KEY EXTRACTION ===
# AES key finder
aeskeyfind memdump.img
# Output: Found AES-256 key: 0x...

# RSA key finder
rsakeyfind memdump.img

# BitLocker key extraction (Volatility plugin)
volatility -f memdump.img --profile=Win10x64 bitlocker
```

### Cold Boot Mitigations

```
+-------------------------------------------------------------------+
|               COLD BOOT MITIGATIONS                                |
+-------------------------------------------------------------------+

1. DISABLE SLEEP/HIBERNATION
   Hibernation writes RAM to disk (hiberfil.sys)
   Sleep keeps RAM powered
   
   Windows: powercfg /h off
   Linux: systemctl mask sleep.target suspend.target hibernate.target

2. BIOS PASSWORD + SECURE BOOT
   Prevents booting from USB/external media
   Makes RAM dumping harder

3. FULL MEMORY ENCRYPTION (AMD SME, Intel TME)
   RAM encrypted by CPU
   Cold boot gets encrypted data

4. RAM CLEARING ON SHUTDOWN
   Some BIOS/OS clear RAM on power off
   Enable if available

5. PHYSICAL SECURITY
   Prevent physical access
   Tamper-evident cases
   Alarmed server rooms

6. USE TPM + PIN
   Even if RAM dumped, TPM won't unseal without PIN
   (Attacker gets encrypted VMK, not FVEK)

7. KERNEL MEMORY ENCRYPTION
   Linux: AMD SME support
   Windows: Memory encryption with HVCI

+-------------------------------------------------------------------+
```

---

## Evil Maid Attacks

### Evil Maid Attack Concept

```
+-------------------------------------------------------------------+
|                    EVIL MAID ATTACK                                |
+-------------------------------------------------------------------+

SCENARIO:
  Attacker has brief physical access (hotel, office)
  Target leaves encrypted laptop unattended
  
ATTACK PHASES:

  Phase 1: Initial Access (Evil maid visit #1)
  
    +-------------------+
    | Original Boot     |
    | Partition         |
    +-------------------+
              |
              v REPLACED WITH
    +-------------------+
    | Malicious Boot    |
    | (Looks identical) |
    +-------------------+
    
  Phase 2: User Returns, Boots Laptop
  
    User enters password → Captured by malicious bootloader
    Password sent to attacker (USB, network, stored on disk)
    Real OS boots normally (user suspects nothing)
    
  Phase 3: Second Access (Evil maid visit #2)
  
    Attacker retrieves captured password
    Decrypts volume at leisure

VARIATIONS:
  - Hardware keylogger installation
  - DMA implant (Thunderbolt device)
  - BIOS/UEFI firmware modification
  - Boot partition modification

+-------------------------------------------------------------------+
```

### Evil Maid Tools

```
+-------------------------------------------------------------------+
|                   EVIL MAID TOOLS                                  |
+-------------------------------------------------------------------+

1. EVIL MAID BOOTLOADER
   Custom bootloader that:
   - Displays fake password prompt
   - Captures entered password
   - Chainloads real bootloader
   
   Example: YOURGOAL project (proof of concept)

2. HARDWARE KEYLOGGERS
   USB keylogger between keyboard and computer
   Or internal PS/2/USB keylogger
   
   Products: KeyGrabber, Keyllama

3. DMA ATTACK TOOLS
   Thunderbolt implant with PCILeech
   Captures memory during boot
   
   Products: Custom builds, Thunderclap research

4. FIRMWARE ROOTKITS
   Modify UEFI firmware to persist malware
   Survives OS reinstall
   
   Examples: Lojax, MosaicRegressor (state actors)

+-------------------------------------------------------------------+
```

### Evil Maid Mitigations

```
+-------------------------------------------------------------------+
|               EVIL MAID MITIGATIONS                                |
+-------------------------------------------------------------------+

1. SECURE BOOT
   Verifies bootloader signature
   Attacker can't replace bootloader without breaking boot

2. MEASURED BOOT (TPM)
   TPM validates boot chain
   Modified bootloader = different PCR values
   BitLocker won't unseal = boot fails

3. BIOS PASSWORD
   Prevents BIOS access
   Harder to disable Secure Boot

4. FULL DISK ENCRYPTION WITH TPM + PIN
   Even with bootloader modification:
   - TPM won't unseal (PCR mismatch)
   - PIN required anyway

5. PHYSICAL TAMPER EVIDENCE
   Glitter nail polish on screws
   Tamper-evident seals
   Detect if case opened

6. BOOT VERIFICATION
   Check boot partition hash before entering password
   External verification device

7. NEVER LEAVE UNATTENDED
   Take laptop with you
   Use hotel safe (still not perfect)
   Work from home

+-------------------------------------------------------------------+
```

---

## Key Recovery Methods

### BitLocker Recovery Methods

```powershell
# === ACTIVE DIRECTORY RECOVERY ===
# Recovery keys stored in AD under computer object
# Query AD for recovery key
Get-ADObject -Filter {objectClass -eq 'msFVE-RecoveryInformation'} -SearchBase "CN=ComputerName,OU=Computers,DC=domain,DC=com" -Properties *

# Using RSAT (GUI)
# Active Directory Users and Computers → Computer → BitLocker Recovery tab

# === AZURE AD RECOVERY ===
# Azure Portal → Devices → BitLocker keys
# Or using Graph API

# === MICROSOFT ACCOUNT ===
# https://account.microsoft.com/devices/recoverykey

# === LOCAL RECOVERY KEY FILE ===
# Search for .BEK files
dir /s *.BEK

# Recovery key format: XXXXXX-XXXXXX-XXXXXX-XXXXXX-XXXXXX-XXXXXX-XXXXXX-XXXXXX

# === REGISTRY (for suspended volumes) ===
reg query "HKLM\SYSTEM\CurrentControlSet\Control\FVE" /s
```

### LUKS Recovery

```bash
# === HEADER BACKUP RECOVERY ===
# If header was backed up
cryptsetup luksHeaderRestore /dev/sdb1 --header-backup-file header_backup.img

# === KEYFILE RECOVERY ===
# If keyfile was used and still accessible
cryptsetup luksOpen /dev/sdb1 volume --key-file /path/to/keyfile

# === BRUTEFORCE PASSPHRASE ===
# Last resort if weak passphrase
hashcat -m 14600 luks_header.img rockyou.txt

# === MASTER KEY RECOVERY (if known) ===
# If you have the master key (e.g., from memory dump)
cryptsetup luksOpen /dev/sdb1 volume --master-key-file masterkey.bin

# === CHECK KEY SLOTS ===
# See which slots are active
cryptsetup luksDump /dev/sdb1 | grep -A5 "Keyslots"
```

### Memory-Based Key Recovery

```bash
# === VOLATILITY PLUGINS ===
# Windows BitLocker
volatility -f memdump.img --profile=Win10x64 bitlocker

# Disk encryption keys
volatility -f memdump.img --profile=Win10x64 cryptscan

# === AES KEY FINDER ===
# Find AES keys in memory dump
./aeskeyfind memdump.img

# Output:
# 2b7e151628aed2a6abf7158809cf4f3c (AES-128)
# 603deb1015ca71be2b73aef0857d77811f352c073b6108d72d9810a30914dff4 (AES-256)

# === RSA KEY FINDER ===
./rsakeyfind memdump.img

# === USING BULK_EXTRACTOR ===
bulk_extractor -o output/ memdump.img
# Check output/aes_keys.txt
```

---

## Cloud Encryption at Rest

### AWS Encryption at Rest

```
+-------------------------------------------------------------------+
|                  AWS ENCRYPTION AT REST                            |
+-------------------------------------------------------------------+

EBS (Elastic Block Store):
  - AES-256 encryption
  - AWS managed keys OR customer managed (CMK)
  - Encrypted at rest and in transit to EC2

S3 (Simple Storage Service):
  - SSE-S3: AWS managed keys
  - SSE-KMS: Customer managed via KMS
  - SSE-C: Customer provided keys

RDS (Relational Database Service):
  - TDE for SQL Server and Oracle
  - AES-256 for others
  - Encrypted snapshots

EFS (Elastic File System):
  - AES-256 encryption
  - KMS integration

KEY MANAGEMENT:
  - AWS KMS (Key Management Service)
  - Customer Master Keys (CMKs)
  - Key rotation support

+-------------------------------------------------------------------+
```

### Azure Encryption at Rest

```
+-------------------------------------------------------------------+
|                 AZURE ENCRYPTION AT REST                           |
+-------------------------------------------------------------------+

AZURE DISK ENCRYPTION:
  - BitLocker for Windows VMs
  - dm-crypt for Linux VMs
  - Keys in Azure Key Vault

STORAGE SERVICE ENCRYPTION (SSE):
  - All Azure Storage encrypted
  - Microsoft managed or customer managed keys

AZURE SQL:
  - TDE (Transparent Data Encryption)
  - Always Encrypted for column-level

AZURE KEY VAULT:
  - Centralized key management
  - Hardware Security Module (HSM) option
  - BYOK (Bring Your Own Key) support

+-------------------------------------------------------------------+
```

### GCP Encryption at Rest

```
+-------------------------------------------------------------------+
|                  GCP ENCRYPTION AT REST                            |
+-------------------------------------------------------------------+

DEFAULT ENCRYPTION:
  - All data encrypted by default
  - AES-256 or AES-128
  - Google managed keys (DEKs and KEKs)

CUSTOMER-MANAGED KEYS (CMEK):
  - Keys in Cloud KMS
  - Customer controls key lifecycle

CUSTOMER-SUPPLIED KEYS (CSEK):
  - Customer provides keys for Compute Engine disks
  - Keys never stored by Google

HIERARCHY:
  Data → DEK (Data Encryption Key) → KEK (Key Encryption Key) → KMS
  
Each layer encrypts the key below it

+-------------------------------------------------------------------+
```

---

## Database Encryption

### Transparent Data Encryption (TDE)

```
+-------------------------------------------------------------------+
|            TRANSPARENT DATA ENCRYPTION (TDE)                       |
+-------------------------------------------------------------------+

WHAT IS TDE?
  Database encrypts data files transparently
  Application sees unencrypted data
  Files on disk are encrypted

SQL SERVER TDE:
  
  +------------------+
  | Service Master   |  Protected by Windows DPAPI
  | Key (SMK)        |
  +------------------+
           |
           v
  +------------------+
  | Database Master  |  Encrypted by SMK
  | Key (DMK)        |
  +------------------+
           |
           v
  +------------------+
  | Certificate      |  Encrypted by DMK
  |                  |
  +------------------+
           |
           v
  +------------------+
  | Database         |  Encrypted by Certificate
  | Encryption Key   |
  | (DEK)            |
  +------------------+
           |
           v
  +------------------+
  | Database Files   |  AES-256 encrypted
  +------------------+

+-------------------------------------------------------------------+
```

### SQL Server TDE Commands

```sql
-- Create master key
CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'Strong!Password123';

-- Create certificate
CREATE CERTIFICATE TDE_Cert WITH SUBJECT = 'TDE Certificate';

-- Create database encryption key
USE YourDatabase;
CREATE DATABASE ENCRYPTION KEY
WITH ALGORITHM = AES_256
ENCRYPTION BY SERVER CERTIFICATE TDE_Cert;

-- Enable TDE
ALTER DATABASE YourDatabase SET ENCRYPTION ON;

-- Check encryption status
SELECT 
    db.name,
    db.is_encrypted,
    dm.encryption_state,
    dm.percent_complete
FROM sys.databases db
LEFT JOIN sys.dm_database_encryption_keys dm
ON db.database_id = dm.database_id;

-- Backup certificate (CRITICAL!)
BACKUP CERTIFICATE TDE_Cert
TO FILE = 'C:\Backup\TDE_Cert.cer'
WITH PRIVATE KEY (
    FILE = 'C:\Backup\TDE_Cert_Key.pvk',
    ENCRYPTION BY PASSWORD = 'BackupPassword!'
);
```

### Column-Level Encryption

```
+-------------------------------------------------------------------+
|              COLUMN-LEVEL ENCRYPTION                               |
+-------------------------------------------------------------------+

USE CASE:
  Encrypt specific sensitive columns
  Credit cards, SSN, medical data
  
TYPES:
  1. Application-level: App encrypts before INSERT
  2. Database functions: SQL encrypt/decrypt functions
  3. Always Encrypted: Client-side encryption (SQL Server)

ALWAYS ENCRYPTED (SQL Server):
  
  Client App                    SQL Server
  +--------+                    +---------+
  | Data   | --Encrypted--->   | Stored  |
  |        | <--Encrypted----  | Data    |
  +--------+                    +---------+
       ^
       |
  Column Master Key
  (Client-side)
  
  Server never sees plaintext!
  Even DBAs can't read data

+-------------------------------------------------------------------+
```

---

## File-Level Encryption

### Windows EFS (Encrypting File System)

```powershell
# === EFS OPERATIONS ===
# Encrypt a file
cipher /e secret.txt

# Encrypt a folder (and contents)
cipher /e /s:C:\SecretFolder

# Decrypt
cipher /d secret.txt

# Check encryption status
cipher /c filename

# === CERTIFICATE MANAGEMENT ===
# Export EFS certificate
certmgr.msc
# → Personal → Certificates → Your EFS cert → Export

# Backup EFS key
cipher /r:EFSBackup
# Creates EFSBackup.cer and EFSBackup.pfx

# === RECOVERY AGENT ===
# Add Data Recovery Agent (domain policy)
# Group Policy → Computer Config → Windows Settings → Security → Public Key

# === FORENSIC NOTES ===
# EFS keys stored in user profile
# DPAPI master key protects EFS private key
# %APPDATA%\Microsoft\Protect\[SID]\
```

### VeraCrypt Containers

```bash
# === CREATE CONTAINER ===
# Interactive creation
veracrypt -c

# Command line creation
veracrypt -c mycontainer.hc \
  --size=100M \
  --password=YourPassword \
  --encryption=AES-256 \
  --hash=SHA-512 \
  --filesystem=NTFS

# === MOUNT/UNMOUNT ===
# Mount container
veracrypt mycontainer.hc /v 1
# Mounts as volume 1 (Windows) or to mount point

# Linux mount
veracrypt mycontainer.hc /mnt/secure

# Unmount
veracrypt -d mycontainer.hc

# Unmount all
veracrypt -d

# === HIDDEN VOLUMES ===
# VeraCrypt supports hidden volumes
# Outer volume: decoy data
# Hidden volume: real sensitive data
# Different password reveals different volume
# Plausible deniability
```

### 7-Zip Encryption

```bash
# Encrypt with AES-256
7z a -p -mhe=on archive.7z files/
# -p prompts for password
# -mhe=on encrypts filenames too

# Extract with password
7z x archive.7z -pYourPassword

# Notes on 7-Zip encryption:
# - AES-256 in CBC mode
# - PBKDF2 with SHA-256 for key derivation
# - Filenames can be encrypted (-mhe=on)
# - Considered secure with strong password
```

---

## OSCP Relevance

### OSCP Encryption Scenarios

```
+-------------------------------------------------------------------+
|              OSCP ENCRYPTION SCENARIOS                             |
+-------------------------------------------------------------------+

1. FINDING ENCRYPTED VOLUMES
   - Identify BitLocker/LUKS during enumeration
   - Check for mounted encrypted volumes
   - Look for recovery keys in AD/files

2. POST-EXPLOITATION KEY RECOVERY
   - Search for recovery keys in file shares
   - Query AD for BitLocker recovery info
   - Check user documents for key backups

3. CREDENTIAL ACCESS
   - Encrypted volumes may contain credentials
   - Password managers, SSH keys, etc.
   - Motivates finding decryption method

4. LATERAL MOVEMENT
   - Recovered credentials enable lateral movement
   - Same password may unlock other volumes
   - AD recovery keys = access to all domain machines

5. FORENSIC ANALYSIS
   - Identify encryption in disk images
   - Memory analysis for keys
   - Offline password attacks

+-------------------------------------------------------------------+
```

### OSCP Commands Cheat Sheet

```bash
# === IDENTIFY ENCRYPTION ===
# Check for BitLocker (Windows)
manage-bde -status

# Check for LUKS (Linux)
cryptsetup isLuks /dev/sda1 && echo "LUKS detected"
file -s /dev/sda1 | grep -i luks

# Check mounted encrypted filesystems
mount | grep -E "(crypt|luks|bitlock)"
lsblk -f

# === SEARCH FOR RECOVERY KEYS ===
# Windows - local files
dir /s *.BEK
dir /s *recovery* 2>nul | findstr /i key

# Linux - common locations
find / -name "*.key" 2>/dev/null
find / -name "*luks*" 2>/dev/null
grep -r "recovery" /home/ 2>/dev/null

# === AD BITLOCKER RECOVERY ===
# PowerShell
Get-ADObject -Filter {objectClass -eq 'msFVE-RecoveryInformation'} -SearchBase "DC=domain,DC=com" -Properties msFVE-RecoveryPassword

# LDAP search
ldapsearch -x -H ldap://dc.domain.com -b "DC=domain,DC=com" "(objectClass=msFVE-RecoveryInformation)" msFVE-RecoveryPassword

# === MEMORY ANALYSIS ===
# Dump memory (if possible)
winpmem_mini_x64.exe memdump.raw

# Analyze with Volatility
volatility -f memdump.raw --profile=Win10x64 bitlocker
```

---

## Hands-On Labs

### Lab 1: LUKS Encryption Setup and Attack

```markdown
OBJECTIVE: Create LUKS volume and practice password cracking

SETUP:
1. Create a test file to use as container
   dd if=/dev/zero of=luks_test.img bs=1M count=100

2. Setup as loop device
   losetup /dev/loop0 luks_test.img

3. Create LUKS container with WEAK password
   cryptsetup luksFormat /dev/loop0
   # Use password: "password123"

TASKS:
1. Open and mount the LUKS volume
2. Create some test files
3. Close the volume
4. Extract the LUKS header
5. Crack the password using hashcat

SOLUTION:
# Mount
cryptsetup luksOpen /dev/loop0 test_volume
mkfs.ext4 /dev/mapper/test_volume
mount /dev/mapper/test_volume /mnt/test

# Unmount
umount /mnt/test
cryptsetup luksClose test_volume

# Extract header for cracking
dd if=/dev/loop0 of=luks_header.img bs=512 count=4097

# Crack
hashcat -m 14600 luks_header.img /usr/share/wordlists/rockyou.txt
```

### Lab 2: BitLocker Recovery Key Discovery

```markdown
OBJECTIVE: Find BitLocker recovery keys in Active Directory

SETUP:
- Windows domain with BitLocker-encrypted machines
- Recovery keys stored in AD

TASKS:
1. Enumerate computers with BitLocker
2. Query AD for recovery keys
3. Use recovery key to unlock volume

COMMANDS:
# PowerShell - Get all BitLocker recovery info
$computers = Get-ADComputer -Filter * -Properties Name
foreach ($computer in $computers) {
    $recoveryInfo = Get-ADObject -Filter {objectClass -eq 'msFVE-RecoveryInformation'} -SearchBase $computer.DistinguishedName -Properties 'msFVE-RecoveryPassword'
    if ($recoveryInfo) {
        Write-Host "Computer: $($computer.Name)"
        Write-Host "Recovery Password: $($recoveryInfo.'msFVE-RecoveryPassword')"
    }
}

# Unlock with recovery key
manage-bde -unlock C: -RecoveryPassword XXXXXX-XXXXXX-...
```

### Lab 3: Cold Boot Attack Simulation

```markdown
OBJECTIVE: Understand cold boot attack process

SETUP:
- VM with encrypted volume
- Memory dump capability

TASKS:
1. Boot VM with encrypted volume
2. Unlock volume (mount it)
3. Take memory snapshot
4. Analyze snapshot for keys

STEPS:
# Create memory dump (VMware)
# Use snapshot or vmss2core tool

# Analyze with Volatility
volatility -f memdump.img imageinfo
volatility -f memdump.img --profile=Win10x64 pslist

# Search for encryption keys
aeskeyfind memdump.img

# For BitLocker specifically
volatility -f memdump.img --profile=Win10x64 bitlocker
```

### Lab 4: VeraCrypt Hidden Volume

```markdown
OBJECTIVE: Create and use hidden volumes for deniability

TASKS:
1. Create VeraCrypt container with hidden volume
2. Mount outer volume, add decoy files
3. Mount hidden volume, add sensitive files
4. Demonstrate different passwords show different content

STEPS:
# Create container with hidden volume (GUI easier for this)
veracrypt -c

# Select "Create an encrypted file container"
# Select "Hidden VeraCrypt volume"
# Follow prompts for outer and hidden volumes

# Mount outer (decoy) volume
veracrypt container.hc /v 1 -p OuterPassword

# Mount hidden volume (same file, different password)
veracrypt container.hc /v 1 -p HiddenPassword

# Note: Only one can be mounted at a time
# Under duress, give outer password - attacker sees decoy
```

---

## Interview Questions

### Basic Level

1. **Q: What is encryption at rest and why is it important?**
   
   A: Encryption at rest protects stored data by encrypting it on disk. It's important because it protects against physical theft, unauthorized access to storage media, and data breaches when hardware is decommissioned or stolen.

2. **Q: What's the difference between full disk encryption and file-level encryption?**
   
   A: Full disk encryption encrypts the entire drive including OS, providing comprehensive protection but requiring pre-boot authentication. File-level encryption only encrypts specific files/folders, allowing selective protection but leaving OS and unencrypted files exposed.

3. **Q: What is a TPM and how does it help with encryption?**
   
   A: A Trusted Platform Module is a hardware security chip that securely stores encryption keys and measures boot integrity. It helps encryption by storing keys in hardware (harder to extract), validating boot chain integrity, and enabling transparent decryption only when system hasn't been tampered with.

### Intermediate Level

4. **Q: Explain how BitLocker uses TPM for key protection.**
   
   A: BitLocker seals the Volume Master Key to TPM PCR values that represent the boot configuration. During boot, TPM measures components and compares to sealed state. If measurements match (system unmodified), TPM releases the VMK. If different (tampering detected), TPM refuses and recovery key is required.

5. **Q: What is a cold boot attack and how can it be mitigated?**
   
   A: A cold boot attack exploits DRAM data remanence - RAM retains data briefly after power loss. Attackers power off a running system, boot to forensic OS, and dump RAM containing encryption keys. Mitigations include: using TPM+PIN (not TPM-only), enabling memory encryption (AMD SME/Intel TME), disabling sleep, and ensuring BIOS prevents external boot media.

6. **Q: What are the security implications of SED vs software encryption?**
   
   A: SEDs offer performance benefits (no CPU overhead) but research has shown implementation flaws in many drives. Software encryption (BitLocker, LUKS) is more auditable but adds CPU load. Best practice is layering both: SED for baseline protection, software encryption for verified security. Never rely on SED alone for sensitive data.

### Advanced Level

7. **Q: Describe an evil maid attack and its countermeasures.**
   
   A: An evil maid attack occurs when an attacker with brief physical access modifies the unencrypted boot partition to capture the encryption password. When the user returns and enters their password, it's captured. Countermeasures include: Secure Boot (validates bootloader signature), TPM measured boot (detects modifications), BIOS password, and physical tamper evidence.

8. **Q: How would you approach recovering data from a BitLocker-encrypted volume during an engagement?**
   
   A: Multiple approaches: 1) Search for recovery keys in AD (msFVE-RecoveryInformation attribute) or Azure AD, 2) Check file shares for .BEK files or documented recovery keys, 3) If system is running, perform memory forensics for FVEK, 4) If TPM-only mode, attempt DMA attack, 5) Social engineering for recovery key/password, 6) Extract metadata and attempt password cracking as last resort.

9. **Q: Explain the key hierarchy in LUKS and why multiple key slots exist.**
   
   A: LUKS uses a master key for actual encryption, but wraps it under multiple key slots (up to 8). Each slot can have a different passphrase or keyfile. This enables: multiple users with different passwords, password changes without re-encrypting data, recovery key separate from daily-use password, and organizational key escrow while users keep their passwords.

---

## Quick Reference

### Encryption Detection Commands

```bash
# === WINDOWS ===
manage-bde -status
Get-BitLockerVolume

# === LINUX ===
cryptsetup isLuks /dev/sda1
lsblk -f
blkid | grep crypto

# === macOS ===
diskutil cs list
fdesetup status

# === GENERIC ===
file -s /dev/sda1
xxd /dev/sda1 | head -5
# Look for magic bytes
```

### Common Hash Modes (Hashcat)

```
BitLocker:     22100
LUKS:          14600
VeraCrypt:     13711-13753 (varies by cipher)
FileVault 2:   16800
eCryptfs:      12200
7-Zip:         11600
```

### Key Locations Summary

```
+-------------------------------------------------------------------+
|                    KEY LOCATION SUMMARY                            |
+-------------------------------------------------------------------+

BITLOCKER:
  - AD: msFVE-RecoveryInformation on computer object
  - Azure AD: Device properties → BitLocker keys
  - Microsoft Account: account.microsoft.com
  - Local: *.BEK files, printed recovery sheets
  - Memory: FVEK when volume mounted

LUKS:
  - Header: First sectors of partition (backup!)
  - Keyfiles: User-defined locations
  - Memory: Master key when volume open

FILEVAULT:
  - Secure Enclave: T2/Apple Silicon Macs
  - iCloud: If escrowed
  - MDM: If enterprise managed
  - Local: Recovery key on paper

EFS:
  - User profile: %APPDATA%\Microsoft\Protect\
  - Certificate store: Personal certificates
  - Recovery agent: Domain GPO

+-------------------------------------------------------------------+
```

### Mitigation Quick Reference

```
+-------------------------------------------------------------------+
|              ENCRYPTION AT REST HARDENING                          |
+-------------------------------------------------------------------+

BITLOCKER:
  ✓ Use TPM + PIN (not TPM only)
  ✓ Enable Secure Boot
  ✓ Disable sleep/hibernation
  ✓ Use AES-256-XTS
  ✓ Store recovery keys securely
  ✓ Disable DMA ports or use kernel DMA protection

LUKS:
  ✓ Use LUKS2 with Argon2
  ✓ Strong passphrase (15+ chars)
  ✓ Backup header securely
  ✓ Consider detached header
  ✓ Use /dev/random for key generation

GENERAL:
  ✓ Physical security
  ✓ BIOS password
  ✓ Tamper evidence
  ✓ Regular security updates
  ✓ Audit recovery key access

+-------------------------------------------------------------------+
```

---

## Related Files

- [[04_Symmetric_Encryption]] - AES and encryption modes
- [[06_PKI_TLS_and_Certificates]] - Key management concepts
- [[07_Password_Cracking]] - Passphrase attacks
- [[08_Cryptographic_Attacks]] - Cryptographic vulnerabilities
- [[10_SSH_and_Key_Management]] - SSH key protection

---

## Resources

### Official Documentation
- [BitLocker Technical Overview](https://docs.microsoft.com/en-us/windows/security/information-protection/bitlocker/bitlocker-overview)
- [LUKS Specification](https://gitlab.com/cryptsetup/cryptsetup/-/wikis/LUKS-standard/on-disk-format.pdf)
- [VeraCrypt Documentation](https://www.veracrypt.fr/en/Documentation.html)

### Research Papers
- [Cold Boot Attacks on Encryption Keys](https://citp.princeton.edu/research/memory/)
- [Self-Encrypting Deception](https://www.ru.nl/publish/pages/909282/draft-paper.pdf)
- [Evil Maid Attacks](https://theinvisiblethings.blogspot.com/2009/10/evil-maid-goes-after-truecrypt.html)

### Tools
- [Volatility Memory Forensics](https://www.volatilityfoundation.org/)
- [PCILeech DMA Attacks](https://github.com/ufrisk/pcileech)
- [AESKeyFind](https://citp.princeton.edu/research/memory/)

---

*Last updated: 2025-02-07*
