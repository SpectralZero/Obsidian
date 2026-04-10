# 04 - Conditional Logic: If/Case Statements & Decision Making

> **Executive Summary**: Conditional logic is the decision-making engine of your scripts. Just like you make decisions in real life ("If it's raining, take an umbrella"), Bash scripts make decisions based on conditions. This module covers if/then/else statements, test operators, file checking, string comparisons, numeric comparisons, and case statements. You'll learn how to build intelligent scripts that adapt their behavior based on different situations - essential for security automation, system checks, and penetration testing workflows. By the end, you'll be writing scripts that can check if you're root, verify files exist, compare values, and execute different code paths based on conditions.

**What You'll Learn**:
- Complete beginner's guide to if/then/else logic
- Test conditions with [ ] and [[ ]] (and their critical differences!)
- File tests (-f, -d, -e, -r, -w, -x, -s and more)
- String comparisons (equality, null checks, pattern matching)
- Numeric comparisons (-eq, -ne, -lt, -le, -gt, -ge)
- Logical operators (AND, OR, NOT)
- Case statements for pattern matching
- 20+ working security/automation examples
- Practical OSCP-style scripts

---

## Table of Contents

1. [What is Conditional Logic? (ABSOLUTE BEGINNER)](#1-what-is-conditional-logic)
2. [The if Statement - Basic Syntax](#2-the-if-statement)
3. [Test Operators: [ ] vs [[ ]]](#3-test-operators)
4. [if/then/else/elif Structure](#4-if-then-else-elif)
5. [File Test Operators](#5-file-test-operators)
6. [String Comparisons](#6-string-comparisons)
7. [Numeric Comparisons](#7-numeric-comparisons)
8. [Logical Operators (AND/OR/NOT)](#8-logical-operators)
9. [Case Statements](#9-case-statements)
10. [Nested Conditionals](#10-nested-conditionals)
11. [Security Examples (Root Check, Port Open, etc.)](#11-security-examples)
12. [Practical OSCP-Style Scripts](#12-practical-oscp-scripts)
13. [Common Mistakes](#13-common-mistakes)
14. [Hands-On Labs](#14-hands-on-labs)
15. [Interview Questions & Checkpoints](#15-interview-questions)
16. [Summary](#16-summary)

---

## 1. What is Conditional Logic? (ABSOLUTE BEGINNER)

### 1.1 Real-Life Decision Making

**ABSOLUTE BEGINNER EXPLANATION**:

Think about how you make decisions every day:
- **If** it's raining, **then** take an umbrella
- **If** the door is locked, **then** use your key, **else** just open it
- **If** you're hungry **and** it's lunchtime, **then** eat

Your brain constantly evaluates conditions and takes action based on whether they're true or false.

**Bash scripts work the same way**:
```
If the file exists → do something
If user is root → allow action
If password is correct → grant access
If port is open → mark as vulnerable
```

**ANALOGY**: Think of conditional logic as a traffic light:
```
┌─────────────┐
│   RED?      │──YES──→ STOP
│   (Check)   │
└─────────────┘
      │
      NO
      ↓
   GO (Green)
```

Your script "checks" a condition (is the light red?) and decides what to do based on the answer.

### 1.2 The Basic Structure

Every conditional has three parts:

```
1. CONDITION: Something to check (Is file_exists?)
2. TRUE PATH: What to do if condition is TRUE
3. FALSE PATH: What to do if condition is FALSE (optional)
```

**Visual Representation**:
```
        ┌──────────────┐
        │  CONDITION   │
        │  (Test)      │
        └───────┬──────┘
                │
        ┌───────┴────────┐
        ↓                ↓
    ┌───────┐        ┌───────┐
    │ TRUE  │        │ FALSE │
    │ Path  │        │ Path  │
    └───────┘        └───────┘
```

---

## 2. The if Statement - Basic Syntax

### 2.1 Your First if Statement

**ABSOLUTE BEGINNER**: The `if` statement is how you tell Bash: "Check this condition, and if it's true, do this action."

**Basic Syntax**:
```bash
if [ condition ]
then
    # Commands to run if condition is TRUE
fi
```

**IMPORTANT NOTES**:
- Spaces around `[ ]` are **REQUIRED**: `[ condition ]` not `[condition]`
- `fi` is "if" spelled backwards - it marks the end of the if block
- `then` can be on same line with semicolon: `if [ condition ]; then`

**Example 1: Check if File Exists**
```bash
#!/bin/bash
# Simple file existence check

FILE="report.txt"

if [ -f "$FILE" ]
then
    echo "File $FILE exists!"
fi
```

**What Happens**:
1. `[ -f "$FILE" ]` tests if "report.txt" is a regular file
2. If TRUE (file exists) → print message
3. If FALSE (file doesn't exist) → do nothing (script ends)

**Running It**:
```bash
# Create the file first
touch report.txt

# Run script
bash check_file.sh
# Output: File report.txt exists!

# Delete file and run again
rm report.txt
bash check_file.sh
# Output: (nothing - condition was false)
```

### 2.2 Compact Syntax (One-Line)

You can write if statements on one line using semicolons:

```bash
#!/bin/bash
if [ -f "report.txt" ]; then echo "File exists"; fi
```

**BEGINNER TIP**: Multi-line format is more readable for complex conditions. Use one-liners for simple checks.

### 2.3 Example: Check if Running as Root

```bash
#!/bin/bash
# Check if user is root (UID 0)

if [ "$UID" -eq 0 ]
then
    echo "You are running as root - full system access!"
fi
```

**What Happens**:
- `$UID` is a special variable containing user ID
- Root user always has UID = 0
- `-eq` means "equal to" for numbers
- If UID equals 0 → print message

**Running It**:
```bash
# As normal user
bash root_check.sh
# Output: (nothing)

# As root
sudo bash root_check.sh
# Output: You are running as root - full system access!
```

---

## 3. Test Operators: [ ] vs [[ ]]

### 3.1 Two Types of Test Operators

**ABSOLUTE BEGINNER**: Bash has two ways to test conditions:
1. **`[ ]`** - Old-style test (POSIX compatible)
2. **`[[ ]]`** - New-style test (Bash-specific, more features)

**Think of them like this**:
- `[ ]` = Basic calculator (works everywhere, limited features)
- `[[ ]]` = Scientific calculator (more powerful, but only works in Bash)

### 3.2 Single Brackets [ ]

**Syntax**:
```bash
if [ condition ]
```

**Characteristics**:
- POSIX compliant (works in sh, bash, dash, etc.)
- Requires quotes around variables
- No pattern matching
- No regex support

**Example**:
```bash
#!/bin/bash
NAME="Alice"

# Must quote $NAME
if [ "$NAME" = "Alice" ]
then
    echo "Hello Alice!"
fi
```

**Why Quote Variables?**
```bash
# Without quotes - BREAKS if variable is empty
NAME=""
if [ $NAME = "Alice" ]  # ERROR: [ = Alice ] - syntax error!
then
    echo "Hello"
fi

# With quotes - WORKS even if empty
if [ "$NAME" = "Alice" ]  # Becomes: [ "" = "Alice" ] - valid!
then
    echo "Hello"
fi
```

### 3.3 Double Brackets [[ ]]

**Syntax**:
```bash
if [[ condition ]]
```

**Characteristics**:
- Bash-specific (doesn't work in sh)
- Quotes are optional (safer variable handling)
- Supports pattern matching with `=~`
- Supports glob patterns
- Better error handling

**Example 1: No Quotes Needed**
```bash
#!/bin/bash
NAME="Alice"

# Quotes optional (but recommended)
if [[ $NAME = Alice ]]
then
    echo "Hello Alice!"
fi
```

**Example 2: Pattern Matching**
```bash
#!/bin/bash
EMAIL="user@example.com"

# Regex matching - check if valid email
if [[ $EMAIL =~ ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$ ]]
then
    echo "Valid email format!"
else
    echo "Invalid email format!"
fi
```

**Example 3: Glob Patterns**
```bash
#!/bin/bash
FILENAME="report.pdf"

# Check if filename ends with .pdf
if [[ $FILENAME == *.pdf ]]
then
    echo "This is a PDF file"
fi
```

### 3.4 Comparison Chart

| Feature | `[ ]` | `[[ ]]` |
|---------|-------|---------|
| POSIX Compliant | Yes | No (Bash only) |
| Variable Quoting | Required | Optional |
| Pattern Matching | No | Yes (`==` with globs) |
| Regex Support | No | Yes (`=~`) |
| Logical Operators | `-a`, `-o` | `&&`, `||` |
| String Comparison | `=`, `!=` | `=`, `==`, `!=` |
| Recommended? | For portability | For Bash scripts |

**BEGINNER TIP**: Use `[[ ]]` for Bash scripts (more features, safer). Use `[ ]` only if you need POSIX compatibility.

---

## 4. if/then/else/elif Structure

### 4.1 if/then/else - Two Paths

**ABSOLUTE BEGINNER**: The `else` clause provides an alternative action when the condition is FALSE.

**Syntax**:
```bash
if [ condition ]
then
    # TRUE path
else
    # FALSE path
fi
```

**Visual Flow**:
```
        ┌──────────────┐
        │  CONDITION   │
        └───────┬──────┘
                │
        ┌───────┴────────┐
        ↓                ↓
    ┌────────┐      ┌────────┐
    │ TRUE   │      │ FALSE  │
    │ (then) │      │ (else) │
    └────────┘      └────────┘
```

**Example: File Backup Check**
```bash
#!/bin/bash
# Check if backup exists

BACKUP_FILE="/backup/system_backup.tar.gz"

if [ -f "$BACKUP_FILE" ]
then
    echo "✓ Backup exists: $BACKUP_FILE"
    echo "Size: $(du -h "$BACKUP_FILE" | cut -f1)"
else
    echo "✗ WARNING: Backup file not found!"
    echo "Creating backup now..."
    # Create backup command here
fi
```

### 4.2 if/elif/else - Multiple Conditions

**ABSOLUTE BEGINNER**: When you have more than two options, use `elif` (else-if).

**Syntax**:
```bash
if [ condition1 ]
then
    # Condition1 is TRUE
elif [ condition2 ]
then
    # Condition1 is FALSE, Condition2 is TRUE
elif [ condition3 ]
then
    # Condition1 and 2 are FALSE, Condition3 is TRUE
else
    # All conditions are FALSE
fi
```

**Visual Flow**:
```
┌─────────────┐
│ Condition 1 │─YES→ Action 1
└──────┬──────┘
       NO
       ↓
┌─────────────┐
│ Condition 2 │─YES→ Action 2
└──────┬──────┘
       NO
       ↓
┌─────────────┐
│ Condition 3 │─YES→ Action 3
└──────┬──────┘
       NO
       ↓
   Default Action
```

**Example: Grade Calculator**
```bash
#!/bin/bash
# Convert numeric score to letter grade

echo "Enter your score (0-100):"
read SCORE

if [ "$SCORE" -ge 90 ]
then
    echo "Grade: A - Excellent!"
elif [ "$SCORE" -ge 80 ]
then
    echo "Grade: B - Good job!"
elif [ "$SCORE" -ge 70 ]
then
    echo "Grade: C - Satisfactory"
elif [ "$SCORE" -ge 60 ]
then
    echo "Grade: D - Needs improvement"
else
    echo "Grade: F - Failed"
fi
```

**What Happens**:
```
Input: 85
Check: 85 >= 90? NO
Check: 85 >= 80? YES → "Grade: B"
(Stops checking, doesn't test 70, 60)

Input: 55
Check: 55 >= 90? NO
Check: 55 >= 80? NO
Check: 55 >= 70? NO
Check: 55 >= 60? NO
Execute else → "Grade: F"
```

### 4.3 Practical Example: System Health Check

```bash
#!/bin/bash
# Check system load and alert based on severity

LOAD=$(uptime | awk -F'load average:' '{print $2}' | awk '{print $1}' | cut -d'.' -f1)

if [ "$LOAD" -gt 10 ]
then
    echo "🚨 CRITICAL: System load is $LOAD - immediate action required!"
    # Send alert email
    echo "High load alert!" | mail -s "CRITICAL Load" admin@example.com
elif [ "$LOAD" -gt 5 ]
then
    echo "⚠️  WARNING: System load is $LOAD - monitor closely"
    # Log warning
    echo "$(date): High load $LOAD" >> /var/log/system_alerts.log
elif [ "$LOAD" -gt 2 ]
then
    echo "ℹ️  NOTICE: System load is $LOAD - slightly elevated"
else
    echo "✓ OK: System load is $LOAD - normal"
fi
```

---

## 5. File Test Operators

### 5.1 Understanding File Tests

**ABSOLUTE BEGINNER**: File test operators let you check the status of files and directories before working with them.

**Why Important?**
- Prevent errors (don't try to read a file that doesn't exist!)
- Security checks (is file writable? executable?)
- Automation (process only if backup exists)

**Syntax**:
```bash
if [ -operator filename ]
then
    # File meets condition
fi
```

### 5.2 Common File Test Operators

| Operator | Meaning | Example |
|----------|---------|---------|
| `-f` | Is a regular file | `[ -f "report.txt" ]` |
| `-d` | Is a directory | `[ -d "/etc" ]` |
| `-e` | Exists (file or directory) | `[ -e "data.csv" ]` |
| `-r` | Is readable | `[ -r "config.txt" ]` |
| `-w` | Is writable | `[ -w "log.txt" ]` |
| `-x` | Is executable | `[ -x "script.sh" ]` |
| `-s` | Exists and has size > 0 | `[ -s "output.log" ]` |
| `-L` | Is a symbolic link | `[ -L "shortcut" ]` |
| `-h` | Is a symbolic link (same as -L) | `[ -h "link" ]` |

### 5.3 Example: Check Regular File

```bash
#!/bin/bash
# Verify target is a regular file

TARGET="passwords.txt"

if [ -f "$TARGET" ]
then
    echo "✓ $TARGET is a regular file"
    echo "Contents:"
    cat "$TARGET"
else
    echo "✗ $TARGET is not a regular file (might be directory, link, or doesn't exist)"
fi
```

### 5.4 Example: Check Directory

```bash
#!/bin/bash
# Ensure backup directory exists

BACKUP_DIR="/var/backups/mysql"

if [ -d "$BACKUP_DIR" ]
then
    echo "✓ Backup directory exists"
else
    echo "✗ Backup directory missing - creating it..."
    mkdir -p "$BACKUP_DIR"
    echo "✓ Directory created: $BACKUP_DIR"
fi
```

### 5.5 Example: Check Permissions

```bash
#!/bin/bash
# Verify file is readable before processing

LOG_FILE="/var/log/apache2/access.log"

if [ -r "$LOG_FILE" ]
then
    echo "✓ Log file is readable - analyzing..."
    grep "404" "$LOG_FILE" | wc -l
else
    echo "✗ ERROR: Cannot read $LOG_FILE"
    echo "Run with: sudo $0"
    exit 1
fi
```

### 5.6 Example: Check File Size

```bash
#!/bin/bash
# Process file only if it contains data

DATA_FILE="scan_results.txt"

if [ -s "$DATA_FILE" ]
then
    echo "✓ File has data (size > 0 bytes)"
    echo "Lines: $(wc -l < "$DATA_FILE")"
    echo "Size: $(du -h "$DATA_FILE" | cut -f1)"
else
    echo "✗ File is empty or doesn't exist"
fi
```

### 5.7 Example: Security - Check Executable

```bash
#!/bin/bash
# Security check: Verify script is executable before running

SCRIPT="/opt/security/scan.sh"

if [ -f "$SCRIPT" ] && [ -x "$SCRIPT" ]
then
    echo "✓ Script exists and is executable"
    echo "Running scan..."
    "$SCRIPT"
else
    if [ ! -f "$SCRIPT" ]; then
        echo "✗ ERROR: Script not found: $SCRIPT"
    elif [ ! -x "$SCRIPT" ]; then
        echo "✗ ERROR: Script is not executable"
        echo "Fix with: chmod +x $SCRIPT"
    fi
fi
```

### 5.8 Complete File Test Reference

```bash
#!/bin/bash
# Demonstrate all file test operators

FILE="test.txt"

echo "File Test Operators Demo"
echo "Testing: $FILE"
echo "========================"

# Create test file
echo "Sample content" > "$FILE"

# Test each operator
echo "-f (regular file):    $([ -f "$FILE" ] && echo YES || echo NO)"
echo "-d (directory):       $([ -d "$FILE" ] && echo YES || echo NO)"
echo "-e (exists):          $([ -e "$FILE" ] && echo YES || echo NO)"
echo "-r (readable):        $([ -r "$FILE" ] && echo YES || echo NO)"
echo "-w (writable):        $([ -w "$FILE" ] && echo YES || echo NO)"
echo "-x (executable):      $([ -x "$FILE" ] && echo YES || echo NO)"
echo "-s (size > 0):        $([ -s "$FILE" ] && echo YES || echo NO)"
echo "-L (symbolic link):   $([ -L "$FILE" ] && echo YES || echo NO)"

# Cleanup
rm "$FILE"
```

**Output**:
```
File Test Operators Demo
Testing: test.txt
========================
-f (regular file):    YES
-d (directory):       NO
-e (exists):          YES
-r (readable):        YES
-w (writable):        YES
-x (executable):      NO
-s (size > 0):        YES
-L (symbolic link):   NO
```

---

## 6. String Comparisons

### 6.1 What are String Comparisons?

**ABSOLUTE BEGINNER**: String comparisons check if text values are equal, different, empty, or match patterns.

**Common Uses**:
- Check if username is correct
- Verify password format
- Compare command output
- Check if variable is set

### 6.2 String Comparison Operators

| Operator | Meaning | Example |
|----------|---------|---------|
| `=` or `==` | Strings are equal | `[ "$A" = "$B" ]` |
| `!=` | Strings are not equal | `[ "$A" != "$B" ]` |
| `<` | String A is less than B (alphabetically) | `[[ "$A" < "$B" ]]` |
| `>` | String A is greater than B | `[[ "$A" > "$B" ]]` |
| `-z` | String is empty (zero length) | `[ -z "$VAR" ]` |
| `-n` | String is not empty | `[ -n "$VAR" ]` |

**IMPORTANT**: 
- Use `=` or `==` in `[[ ]]` (both work)
- Use `=` in `[ ]` for POSIX compatibility
- `<` and `>` require `[[ ]]` (need escaping in `[ ]`)

### 6.3 Example: Equality Check

```bash
#!/bin/bash
# Check if username matches

CURRENT_USER=$(whoami)
EXPECTED_USER="admin"

if [ "$CURRENT_USER" = "$EXPECTED_USER" ]
then
    echo "✓ Logged in as admin - access granted"
else
    echo "✗ Access denied - admin access required"
    echo "Current user: $CURRENT_USER"
    exit 1
fi
```

### 6.4 Example: Empty String Check

```bash
#!/bin/bash
# Validate input is not empty

echo "Enter target IP address:"
read TARGET_IP

if [ -z "$TARGET_IP" ]
then
    echo "✗ ERROR: IP address cannot be empty!"
    exit 1
else
    echo "✓ Target set: $TARGET_IP"
    # Continue with scanning
fi
```

**Alternative - Check NOT Empty**:
```bash
#!/bin/bash
echo "Enter password:"
read -s PASSWORD

if [ -n "$PASSWORD" ]
then
    echo "✓ Password received"
else
    echo "✗ ERROR: Password cannot be empty"
    exit 1
fi
```

### 6.5 Example: Pattern Matching with [[ ]]

```bash
#!/bin/bash
# Check if IP address format looks valid

IP="192.168.1.100"

# Simple pattern check (not full validation)
if [[ $IP == [0-9]*.[0-9]*.[0-9]*.[0-9]* ]]
then
    echo "✓ IP format looks valid: $IP"
else
    echo "✗ Invalid IP format: $IP"
fi
```

### 6.6 Example: Regex Matching

```bash
#!/bin/bash
# Validate email format using regex

EMAIL="user@example.com"

# Email regex pattern
if [[ $EMAIL =~ ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$ ]]
then
    echo "✓ Valid email format: $EMAIL"
else
    echo "✗ Invalid email format: $EMAIL"
fi
```

**What Happens**:
- `=~` is the regex match operator (only works in `[[ ]]`)
- `^` = start of string
- `[a-zA-Z0-9._%+-]+` = one or more valid email characters before @
- `@` = literal @ symbol
- `[a-zA-Z0-9.-]+` = domain name
- `\.` = escaped dot (literal .)
- `[a-zA-Z]{2,}` = TLD (2+ letters)
- `$` = end of string

### 6.7 Example: Case-Insensitive Comparison

```bash
#!/bin/bash
# Accept yes/no input (case insensitive)

echo "Do you want to continue? (yes/no)"
read ANSWER

# Convert to lowercase for comparison
ANSWER_LOWER=$(echo "$ANSWER" | tr '[:upper:]' '[:lower:]')

if [ "$ANSWER_LOWER" = "yes" ] || [ "$ANSWER_LOWER" = "y" ]
then
    echo "✓ Continuing..."
else
    echo "✗ Aborted by user"
    exit 0
fi
```

### 6.8 Practical Example: Password Strength Checker

```bash
#!/bin/bash
# Check password meets minimum requirements

echo "Enter password:"
read -s PASSWORD

# Check if empty
if [ -z "$PASSWORD" ]
then
    echo "✗ Password cannot be empty"
    exit 1
fi

# Check minimum length
if [ ${#PASSWORD} -lt 8 ]
then
    echo "✗ Password must be at least 8 characters"
    exit 1
fi

# Check contains number (using regex)
if [[ ! $PASSWORD =~ [0-9] ]]
then
    echo "✗ Password must contain at least one number"
    exit 1
fi

# Check contains uppercase
if [[ ! $PASSWORD =~ [A-Z] ]]
then
    echo "✗ Password must contain at least one uppercase letter"
    exit 1
fi

# Check contains lowercase
if [[ ! $PASSWORD =~ [a-z] ]]
then
    echo "✗ Password must contain at least one lowercase letter"
    exit 1
fi

echo "✓ Password meets all requirements!"
```

---

## 7. Numeric Comparisons

### 7.1 Understanding Numeric Comparisons

**ABSOLUTE BEGINNER**: Numeric comparisons check if numbers are equal, greater than, less than, etc.

**Important**: Bash treats numbers differently than strings!

**WRONG** (treats as strings):
```bash
if [ "10" > "2" ]; then echo "yes"; fi
# Compares alphabetically: "10" < "2" (because "1" comes before "2")
```

**RIGHT** (treats as numbers):
```bash
if [ 10 -gt 2 ]; then echo "yes"; fi
# Compares numerically: 10 is greater than 2
```

### 7.2 Numeric Comparison Operators

| Operator | Meaning | Example |
|----------|---------|---------|
| `-eq` | Equal to | `[ $A -eq $B ]` |
| `-ne` | Not equal to | `[ $A -ne $B ]` |
| `-lt` | Less than | `[ $A -lt $B ]` |
| `-le` | Less than or equal to | `[ $A -le $B ]` |
| `-gt` | Greater than | `[ $A -gt $B ]` |
| `-ge` | Greater than or equal to | `[ $A -ge $B ]` |

**Memory Aid**:
- `-eq` = EQual
- `-ne` = Not Equal
- `-lt` = Less Than
- `-le` = Less or Equal
- `-gt` = Greater Than
- `-ge` = Greater or Equal

### 7.3 Alternative: Arithmetic Comparison with (( ))

You can also use `(( ))` for arithmetic comparisons with familiar operators:

```bash
if (( A == B )); then
if (( A != B )); then
if (( A < B )); then
if (( A <= B )); then
if (( A > B )); then
if (( A >= B )); then
```

### 7.4 Example: Basic Numeric Comparison

```bash
#!/bin/bash
# Check if port number is valid (1-65535)

echo "Enter port number:"
read PORT

if [ "$PORT" -ge 1 ] && [ "$PORT" -le 65535 ]
then
    echo "✓ Valid port: $PORT"
else
    echo "✗ Invalid port (must be 1-65535)"
    exit 1
fi
```

### 7.5 Example: Age Verification

```bash
#!/bin/bash
# Verify user is old enough

echo "Enter your age:"
read AGE

if [ "$AGE" -lt 18 ]
then
    echo "✗ Access denied - must be 18 or older"
    exit 1
elif [ "$AGE" -ge 18 ] && [ "$AGE" -lt 21 ]
then
    echo "⚠️  Limited access (18-20 years old)"
elif [ "$AGE" -ge 21 ]
then
    echo "✓ Full access granted (21+)"
fi
```

### 7.6 Example: System Resource Check

```bash
#!/bin/bash
# Check if disk space is critically low

DISK_USAGE=$(df / | tail -1 | awk '{print $5}' | sed 's/%//')

echo "Root filesystem usage: ${DISK_USAGE}%"

if [ "$DISK_USAGE" -ge 90 ]
then
    echo "🚨 CRITICAL: Disk usage above 90%!"
    echo "Free space immediately!"
elif [ "$DISK_USAGE" -ge 75 ]
then
    echo "⚠️  WARNING: Disk usage above 75%"
    echo "Consider cleaning up files"
elif [ "$DISK_USAGE" -ge 50 ]
then
    echo "ℹ️  NOTICE: Disk usage above 50%"
else
    echo "✓ OK: Disk usage is healthy"
fi
```

### 7.7 Example: Process Count Alert

```bash
#!/bin/bash
# Alert if too many processes running

PROCESS_COUNT=$(ps aux | wc -l)
MAX_ALLOWED=300

echo "Running processes: $PROCESS_COUNT"

if [ "$PROCESS_COUNT" -gt "$MAX_ALLOWED" ]
then
    echo "⚠️  WARNING: Too many processes!"
    echo "Threshold: $MAX_ALLOWED"
    echo "Current: $PROCESS_COUNT"
    echo "Excess: $(( PROCESS_COUNT - MAX_ALLOWED ))"
else
    echo "✓ Process count within limits"
fi
```

### 7.8 Example: Retry Counter

```bash
#!/bin/bash
# Connection retry with max attempts

TARGET="192.168.1.100"
MAX_RETRIES=3
RETRY_COUNT=0

while [ $RETRY_COUNT -lt $MAX_RETRIES ]
do
    echo "Attempt $(( RETRY_COUNT + 1 )) of $MAX_RETRIES..."
    
    if ping -c 1 -W 2 "$TARGET" &>/dev/null
    then
        echo "✓ Connection successful!"
        exit 0
    else
        echo "✗ Connection failed"
        RETRY_COUNT=$(( RETRY_COUNT + 1 ))
        
        if [ $RETRY_COUNT -lt $MAX_RETRIES ]
        then
            echo "Retrying in 5 seconds..."
            sleep 5
        fi
    fi
done

echo "✗ All $MAX_RETRIES attempts failed - giving up"
exit 1
```

---

## 8. Logical Operators (AND/OR/NOT)

### 8.1 Understanding Logical Operators

**ABSOLUTE BEGINNER**: Logical operators let you combine multiple conditions.

**Real-Life Example**:
- "If it's Saturday **AND** sunny, go to beach"
- "If it's raining **OR** snowing, take umbrella"
- "If it's **NOT** a workday, sleep in"

**In Bash**:
- **AND**: Both conditions must be TRUE
- **OR**: At least one condition must be TRUE
- **NOT**: Reverse the condition

### 8.2 AND Operator

**Syntax Options**:
```bash
# Option 1: Using -a in [ ]
if [ condition1 -a condition2 ]

# Option 2: Separate brackets (recommended)
if [ condition1 ] && [ condition2 ]

# Option 3: Using && in [[ ]]
if [[ condition1 && condition2 ]]
```

**Truth Table**:
```
Condition1 | Condition2 | Result
-----------|------------|-------
TRUE       | TRUE       | TRUE
TRUE       | FALSE      | FALSE
FALSE      | TRUE       | FALSE
FALSE      | FALSE      | FALSE
```

**Example 1: File Must Exist AND Be Readable**
```bash
#!/bin/bash
# Process file only if exists AND is readable

FILE="config.txt"

if [ -f "$FILE" ] && [ -r "$FILE" ]
then
    echo "✓ File exists and is readable - processing..."
    cat "$FILE"
else
    echo "✗ Cannot process file"
    [ ! -f "$FILE" ] && echo "  Reason: File doesn't exist"
    [ ! -r "$FILE" ] && echo "  Reason: File not readable"
fi
```

**Example 2: User Must Be Root AND File Must Exist**
```bash
#!/bin/bash
# Security: Only root can edit sensitive file

SENSITIVE_FILE="/etc/shadow"

if [ "$UID" -eq 0 ] && [ -f "$SENSITIVE_FILE" ]
then
    echo "✓ Root access confirmed - file accessible"
else
    if [ "$UID" -ne 0 ]; then
        echo "✗ ERROR: Must be root to access this file"
        echo "Run with: sudo $0"
    fi
    if [ ! -f "$SENSITIVE_FILE" ]; then
        echo "✗ ERROR: File not found: $SENSITIVE_FILE"
    fi
    exit 1
fi
```

### 8.3 OR Operator

**Syntax Options**:
```bash
# Option 1: Using -o in [ ]
if [ condition1 -o condition2 ]

# Option 2: Separate brackets (recommended)
if [ condition1 ] || [ condition2 ]

# Option 3: Using || in [[ ]]
if [[ condition1 || condition2 ]]
```

**Truth Table**:
```
Condition1 | Condition2 | Result
-----------|------------|-------
TRUE       | TRUE       | TRUE
TRUE       | FALSE      | TRUE
FALSE      | TRUE       | TRUE
FALSE      | FALSE      | FALSE
```

**Example 1: Accept Multiple Valid Inputs**
```bash
#!/bin/bash
# Accept yes/y/YES as confirmation

echo "Delete all files? (yes/no)"
read ANSWER

if [ "$ANSWER" = "yes" ] || [ "$ANSWER" = "y" ] || [ "$ANSWER" = "YES" ]
then
    echo "✓ Deletion confirmed - proceeding..."
else
    echo "✗ Deletion cancelled"
    exit 0
fi
```

**Example 2: System Architecture Check**
```bash
#!/bin/bash
# Script compatible with x86_64 or ARM

ARCH=$(uname -m)

if [ "$ARCH" = "x86_64" ] || [ "$ARCH" = "aarch64" ]
then
    echo "✓ Compatible architecture: $ARCH"
else
    echo "✗ Unsupported architecture: $ARCH"
    echo "Supported: x86_64, aarch64"
    exit 1
fi
```

### 8.4 NOT Operator

**Syntax**:
```bash
# Using ! (NOT)
if [ ! condition ]

# In [[ ]]
if [[ ! condition ]]
```

**Truth Table**:
```
Condition | Result
----------|-------
TRUE      | FALSE
FALSE     | TRUE
```

**Example 1: Check File Does NOT Exist**
```bash
#!/bin/bash
# Create file only if it doesn't exist

CONFIG_FILE="app.conf"

if [ ! -f "$CONFIG_FILE" ]
then
    echo "✓ File doesn't exist - creating default config..."
    cat > "$CONFIG_FILE" <<EOF
# Default Configuration
port=8080
host=localhost
EOF
    echo "✓ Created: $CONFIG_FILE"
else
    echo "ℹ️  Config file already exists"
fi
```

**Example 2: Ensure NOT Running as Root**
```bash
#!/bin/bash
# This script should NOT run as root

if [ ! "$UID" -eq 0 ]
then
    echo "✓ Running as normal user - safe to proceed"
else
    echo "✗ ERROR: Do not run this script as root!"
    echo "Run as normal user instead"
    exit 1
fi
```

### 8.5 Combining Operators

**Example 1: Complex Permission Check**
```bash
#!/bin/bash
# File must exist AND (be readable OR be writable)

FILE="data.txt"

if [ -f "$FILE" ] && ([ -r "$FILE" ] || [ -w "$FILE" ])
then
    echo "✓ File accessible"
    [ -r "$FILE" ] && echo "  - Can read"
    [ -w "$FILE" ] && echo "  - Can write"
else
    echo "✗ File not accessible"
fi
```

**Example 2: Validation with Multiple Conditions**
```bash
#!/bin/bash
# Validate IP and port parameters

IP="$1"
PORT="$2"

# Check both parameters provided AND port is valid
if [ -n "$IP" ] && [ -n "$PORT" ] && [ "$PORT" -ge 1 ] && [ "$PORT" -le 65535 ]
then
    echo "✓ Valid parameters"
    echo "  IP: $IP"
    echo "  Port: $PORT"
    # Proceed with connection
else
    echo "✗ Invalid parameters"
    [ -z "$IP" ] && echo "  Missing IP address"
    [ -z "$PORT" ] && echo "  Missing port number"
    [ -n "$PORT" ] && ([ "$PORT" -lt 1 ] || [ "$PORT" -gt 65535 ]) && echo "  Port out of range (1-65535)"
    echo ""
    echo "Usage: $0 <IP> <PORT>"
    exit 1
fi
```

### 8.6 Practical Security Example

```bash
#!/bin/bash
# Comprehensive pre-flight check for security scan

TARGET="$1"
WORDLIST="/usr/share/wordlists/rockyou.txt"

echo "Pre-flight Security Check"
echo "========================="

# Must be root AND target provided AND wordlist exists
if [ "$UID" -eq 0 ] && [ -n "$TARGET" ] && [ -f "$WORDLIST" ]
then
    echo "✓ Root privileges: OK"
    echo "✓ Target specified: $TARGET"
    echo "✓ Wordlist found: $WORDLIST"
    echo ""
    echo "All checks passed - ready to scan"
else
    echo "✗ Pre-flight check failed:"
    
    [ "$UID" -ne 0 ] && echo "  - Must run as root (sudo $0)"
    [ -z "$TARGET" ] && echo "  - Target not specified (usage: $0 <target>)"
    [ ! -f "$WORDLIST" ] && echo "  - Wordlist not found: $WORDLIST"
    
    exit 1
fi
```

---

## 9. Case Statements

### 9.1 What is a Case Statement?

**ABSOLUTE BEGINNER**: A case statement is like a multi-way switch. Instead of writing many if/elif statements, you match a value against multiple patterns.

**Analogy**: Think of a vending machine:
```
Press Button:
  A1 → Dispense chips
  A2 → Dispense candy
  B1 → Dispense soda
  B2 → Dispense water
  *  → "Invalid selection"
```

**When to Use**:
- Matching against multiple possible values
- Menu systems
- Parsing command-line options
- File type detection

### 9.2 Case Statement Syntax

```bash
case VARIABLE in
    pattern1)
        # Commands for pattern1
        ;;
    pattern2)
        # Commands for pattern2
        ;;
    pattern3|pattern4)
        # Commands for pattern3 OR pattern4
        ;;
    *)
        # Default case (catch-all)
        ;;
esac
```

**Important Parts**:
- `case` ... `esac` (esac = case backwards)
- Each pattern ends with `)`
- Commands end with `;;` (like break in other languages)
- `|` for multiple patterns (OR)
- `*` for default case (matches anything)

### 9.3 Example: Simple Menu

```bash
#!/bin/bash
# Simple interactive menu

echo "Select an option:"
echo "1) List files"
echo "2) Show date"
echo "3) Show users"
echo "4) Exit"
read CHOICE

case $CHOICE in
    1)
        echo "Files in current directory:"
        ls -lh
        ;;
    2)
        echo "Current date and time:"
        date
        ;;
    3)
        echo "Logged in users:"
        who
        ;;
    4)
        echo "Goodbye!"
        exit 0
        ;;
    *)
        echo "Invalid choice: $CHOICE"
        exit 1
        ;;
esac
```

### 9.4 Example: File Type Detection

```bash
#!/bin/bash
# Detect file type by extension

FILENAME="$1"

if [ -z "$FILENAME" ]; then
    echo "Usage: $0 <filename>"
    exit 1
fi

case $FILENAME in
    *.txt)
        echo "Text file - opening in editor"
        nano "$FILENAME"
        ;;
    *.pdf)
        echo "PDF document - opening viewer"
        xdg-open "$FILENAME"
        ;;
    *.jpg|*.png|*.gif)
        echo "Image file - opening image viewer"
        xdg-open "$FILENAME"
        ;;
    *.sh)
        echo "Shell script - making executable"
        chmod +x "$FILENAME"
        echo "Executable permission set"
        ;;
    *)
        echo "Unknown file type: $FILENAME"
        ;;
esac
```

### 9.5 Example: Command-Line Options

```bash
#!/bin/bash
# Parse command-line options

ACTION="$1"

case $ACTION in
    start)
        echo "Starting service..."
        # Start commands here
        ;;
    stop)
        echo "Stopping service..."
        # Stop commands here
        ;;
    restart)
        echo "Restarting service..."
        # Restart commands here
        ;;
    status)
        echo "Checking service status..."
        # Status commands here
        ;;
    -h|--help)
        echo "Usage: $0 {start|stop|restart|status}"
        exit 0
        ;;
    *)
        echo "ERROR: Unknown action: $ACTION"
        echo "Usage: $0 {start|stop|restart|status}"
        exit 1
        ;;
esac
```

### 9.6 Example: OS Detection

```bash
#!/bin/bash
# Detect operating system

OS=$(uname -s)

case $OS in
    Linux)
        echo "Running on Linux"
        PACKAGE_MANAGER="apt"
        ;;
    Darwin)
        echo "Running on macOS"
        PACKAGE_MANAGER="brew"
        ;;
    CYGWIN*|MINGW*|MSYS*)
        echo "Running on Windows (bash emulation)"
        PACKAGE_MANAGER="choco"
        ;;
    FreeBSD)
        echo "Running on FreeBSD"
        PACKAGE_MANAGER="pkg"
        ;;
    *)
        echo "Unknown operating system: $OS"
        PACKAGE_MANAGER="unknown"
        ;;
esac

echo "Package manager: $PACKAGE_MANAGER"
```

### 9.7 Example: Yes/No Confirmation

```bash
#!/bin/bash
# Flexible yes/no confirmation

echo "Delete all log files? (yes/no)"
read ANSWER

# Convert to lowercase
ANSWER=$(echo "$ANSWER" | tr '[:upper:]' '[:lower:]')

case $ANSWER in
    yes|y|yeah|yep)
        echo "✓ Deletion confirmed"
        rm -f *.log
        echo "Log files deleted"
        ;;
    no|n|nope|nah)
        echo "✗ Deletion cancelled"
        exit 0
        ;;
    *)
        echo "Invalid response: $ANSWER"
        echo "Please answer yes or no"
        exit 1
        ;;
esac
```

### 9.8 Practical Example: Network Port Scanner Menu

```bash
#!/bin/bash
# Port scanning menu system

echo "================================"
echo "  Network Port Scanner"
echo "================================"
echo "1) Quick scan (common ports)"
echo "2) Full scan (all 65535 ports)"
echo "3) Web ports (80, 443, 8080)"
echo "4) Custom port range"
echo "5) Exit"
echo ""
echo -n "Select scan type: "
read CHOICE

echo -n "Enter target IP: "
read TARGET

case $CHOICE in
    1)
        echo "Starting quick scan on $TARGET..."
        nmap -F "$TARGET"
        ;;
    2)
        echo "Starting full scan on $TARGET (this will take time)..."
        nmap -p- "$TARGET"
        ;;
    3)
        echo "Scanning web ports on $TARGET..."
        nmap -p 80,443,8080,8443 "$TARGET"
        ;;
    4)
        echo -n "Enter port range (e.g., 1-1000): "
        read PORTS
        echo "Scanning ports $PORTS on $TARGET..."
        nmap -p "$PORTS" "$TARGET"
        ;;
    5)
        echo "Exiting..."
        exit 0
        ;;
    *)
        echo "Invalid choice: $CHOICE"
        exit 1
        ;;
esac
```

### 9.9 Pattern Matching with Wildcards

```bash
#!/bin/bash
# Demonstrate wildcard patterns in case

INPUT="$1"

case $INPUT in
    [0-9])
        echo "Single digit: $INPUT"
        ;;
    [0-9][0-9])
        echo "Two digits: $INPUT"
        ;;
    [a-z]*)
        echo "Starts with lowercase letter: $INPUT"
        ;;
    [A-Z]*)
        echo "Starts with uppercase letter: $INPUT"
        ;;
    *[!0-9]*)
        echo "Contains non-numeric characters: $INPUT"
        ;;
    *)
        echo "Other: $INPUT"
        ;;
esac
```

---

## 10. Nested Conditionals

### 10.1 What are Nested Conditionals?

**ABSOLUTE BEGINNER**: Nested conditionals are if statements inside other if statements. Like Russian nesting dolls - conditions within conditions.

**Visual Example**:
```
if (outer condition)
    if (inner condition 1)
        action
    else
        if (inner condition 2)
            action
```

### 10.2 Example: File Permission Checker

```bash
#!/bin/bash
# Check file permissions in detail

FILE="$1"

if [ -z "$FILE" ]; then
    echo "Usage: $0 <filename>"
    exit 1
fi

if [ -e "$FILE" ]; then
    echo "✓ File exists: $FILE"
    
    if [ -f "$FILE" ]; then
        echo "  Type: Regular file"
        
        if [ -r "$FILE" ]; then
            echo "  ✓ Readable"
        else
            echo "  ✗ Not readable"
        fi
        
        if [ -w "$FILE" ]; then
            echo "  ✓ Writable"
        else
            echo "  ✗ Not writable"
        fi
        
        if [ -x "$FILE" ]; then
            echo "  ✓ Executable"
        else
            echo "  ✗ Not executable"
        fi
        
    elif [ -d "$FILE" ]; then
        echo "  Type: Directory"
    elif [ -L "$FILE" ]; then
        echo "  Type: Symbolic link"
    fi
else
    echo "✗ File does not exist: $FILE"
fi
```

### 10.3 Example: User Permission Levels

```bash
#!/bin/bash
# Multi-level permission check

if [ "$UID" -eq 0 ]; then
    echo "✓ Running as root - full access"
    
    if [ -f "/etc/shadow" ]; then
        echo "  Shadow file accessible"
        
        if [ -r "/etc/shadow" ]; then
            echo "  Can read password hashes"
        fi
    fi
else
    echo "Running as regular user: $(whoami)"
    
    if groups | grep -q "sudo"; then
        echo "  ✓ Member of sudo group"
        echo "  Can elevate privileges with sudo"
    else
        echo "  ✗ Not in sudo group"
        echo "  Limited permissions"
    fi
fi
```

### 10.4 Example: Network Connectivity Checker

```bash
#!/bin/bash
# Nested network diagnostics

TARGET="8.8.8.8"

echo "Checking network connectivity..."

if ping -c 1 -W 2 "$TARGET" &>/dev/null; then
    echo "✓ Internet connection available"
    
    if host google.com &>/dev/null; then
        echo "  ✓ DNS resolution working"
        
        if curl -s --connect-timeout 5 https://google.com &>/dev/null; then
            echo "    ✓ HTTPS working"
            echo "    Network fully functional!"
        else
            echo "    ✗ HTTPS not working"
            echo "    Possible firewall blocking port 443"
        fi
    else
        echo "  ✗ DNS resolution failed"
        echo "  Check /etc/resolv.conf"
    fi
else
    echo "✗ No internet connection"
    
    if ping -c 1 -W 2 192.168.1.1 &>/dev/null; then
        echo "  Local gateway reachable"
        echo "  Problem is upstream (ISP or router)"
    else
        echo "  Local gateway unreachable"
        echo "  Check network cable/WiFi"
    fi
fi
```

---

## 11. Security Examples (Root Check, Port Open, etc.)

### 11.1 Example: Root Privilege Checker

```bash
#!/bin/bash
# Ensure script runs with root privileges

check_root() {
    if [ "$EUID" -ne 0 ]; then
        echo "✗ ERROR: This script must run as root"
        echo ""
        echo "Please run with sudo:"
        echo "  sudo $0"
        exit 1
    else
        echo "✓ Root privileges confirmed"
    fi
}

check_root
# Continue with privileged operations...
```

### 11.2 Example: Port Availability Checker

```bash
#!/bin/bash
# Check if port is open/listening

TARGET="$1"
PORT="$2"

if [ -z "$TARGET" ] || [ -z "$PORT" ]; then
    echo "Usage: $0 <target> <port>"
    exit 1
fi

# Method 1: Using netcat (if available)
if command -v nc &>/dev/null; then
    if nc -zv "$TARGET" "$PORT" 2>&1 | grep -q "succeeded"; then
        echo "✓ Port $PORT is OPEN on $TARGET"
        exit 0
    else
        echo "✗ Port $PORT is CLOSED on $TARGET"
        exit 1
    fi
fi

# Method 2: Using /dev/tcp (fallback)
if timeout 2 bash -c ">/dev/tcp/$TARGET/$PORT" 2>/dev/null; then
    echo "✓ Port $PORT is OPEN on $TARGET"
else
    echo "✗ Port $PORT is CLOSED on $TARGET"
fi
```

### 11.3 Example: SSH Key Checker

```bash
#!/bin/bash
# Verify SSH keys are properly configured

SSH_KEY="$HOME/.ssh/id_rsa"
SSH_PUB="$HOME/.ssh/id_rsa.pub"
AUTHORIZED="$HOME/.ssh/authorized_keys"

echo "SSH Key Configuration Check"
echo "============================"

# Check if private key exists
if [ -f "$SSH_KEY" ]; then
    echo "✓ Private key exists: $SSH_KEY"
    
    # Check permissions (should be 600)
    PERMS=$(stat -c %a "$SSH_KEY" 2>/dev/null || stat -f %Lp "$SSH_KEY" 2>/dev/null)
    if [ "$PERMS" = "600" ]; then
        echo "  ✓ Correct permissions (600)"
    else
        echo "  ⚠️  WARNING: Permissions are $PERMS (should be 600)"
        echo "  Fix with: chmod 600 $SSH_KEY"
    fi
else
    echo "✗ Private key not found: $SSH_KEY"
    echo "  Generate with: ssh-keygen -t rsa -b 4096"
fi

# Check if public key exists
if [ -f "$SSH_PUB" ]; then
    echo "✓ Public key exists: $SSH_PUB"
else
    echo "✗ Public key not found: $SSH_PUB"
fi

# Check authorized_keys
if [ -f "$AUTHORIZED" ]; then
    COUNT=$(wc -l < "$AUTHORIZED")
    echo "✓ Authorized keys file exists ($COUNT keys)"
else
    echo "ℹ️  No authorized_keys file (incoming SSH connections not configured)"
fi
```

### 11.4 Example: Security Audit Script

```bash
#!/bin/bash
# Basic security audit for Linux system

echo "========================================="
echo "  Quick Security Audit"
echo "========================================="
echo ""

# Check 1: Running as root?
echo "[1] Root Access Check"
if [ "$UID" -eq 0 ]; then
    echo "    ✓ Running with root privileges"
else
    echo "    ⚠️  Running as user: $(whoami)"
    echo "    Some checks may be limited"
fi
echo ""

# Check 2: Firewall status
echo "[2] Firewall Status"
if command -v ufw &>/dev/null; then
    if ufw status | grep -q "Status: active"; then
        echo "    ✓ UFW firewall is active"
    else
        echo "    ✗ UFW firewall is inactive"
    fi
elif command -v firewall-cmd &>/dev/null; then
    if firewall-cmd --state &>/dev/null; then
        echo "    ✓ Firewalld is running"
    else
        echo "    ✗ Firewalld is not running"
    fi
else
    echo "    ⚠️  No firewall detected (ufw/firewalld)"
fi
echo ""

# Check 3: SSH configuration
echo "[3] SSH Security"
SSH_CONFIG="/etc/ssh/sshd_config"
if [ -f "$SSH_CONFIG" ]; then
    if grep -q "^PermitRootLogin no" "$SSH_CONFIG"; then
        echo "    ✓ Root SSH login disabled"
    else
        echo "    ⚠️  Root SSH login may be enabled"
    fi
    
    if grep -q "^PasswordAuthentication no" "$SSH_CONFIG"; then
        echo "    ✓ Password authentication disabled"
    else
        echo "    ⚠️  Password authentication may be enabled"
    fi
fi
echo ""

# Check 4: Open ports
echo "[4] Open Ports"
if command -v ss &>/dev/null; then
    LISTENING=$(ss -tuln | grep LISTEN | wc -l)
    echo "    Listening ports: $LISTENING"
    if [ "$LISTENING" -gt 10 ]; then
        echo "    ⚠️  Many ports open - review services"
    fi
elif command -v netstat &>/dev/null; then
    LISTENING=$(netstat -tuln | grep LISTEN | wc -l)
    echo "    Listening ports: $LISTENING"
fi
echo ""

# Check 5: Updates available
echo "[5] System Updates"
if command -v apt &>/dev/null; then
    UPDATES=$(apt list --upgradable 2>/dev/null | grep -c upgradable)
    if [ "$UPDATES" -gt 0 ]; then
        echo "    ⚠️  $UPDATES updates available"
        echo "    Run: sudo apt update && sudo apt upgrade"
    else
        echo "    ✓ System is up to date"
    fi
fi

echo ""
echo "Audit complete!"
```

### 11.5 Example: Password File Integrity Check

```bash
#!/bin/bash
# Check /etc/passwd for suspicious entries

if [ "$UID" -ne 0 ]; then
    echo "Must run as root"
    exit 1
fi

echo "Checking /etc/passwd for security issues..."
echo ""

# Check for UID 0 users (should only be root)
echo "[1] Users with UID 0 (root equivalent):"
awk -F: '$3 == 0 {print "  ⚠️  " $1 " (UID 0)"}' /etc/passwd

# Check for users with no password
echo ""
echo "[2] Users with empty passwords:"
if [ -f /etc/shadow ]; then
    awk -F: '$2 == "" {print "  ✗ " $1 " has no password!"}' /etc/shadow
fi

# Check for users with login shell
echo ""
echo "[3] Users with login shells:"
grep -v "nologin\|false" /etc/passwd | grep -v "^#" | awk -F: '{print "  " $1 " - " $7}'

echo ""
echo "Review complete!"
```

---

## 12. Practical OSCP-Style Scripts

### 12.1 Script: Automated Ping Sweep

```bash
#!/bin/bash
# Simple ping sweep to find live hosts

NETWORK="$1"

if [ -z "$NETWORK" ]; then
    echo "Usage: $0 <network>"
    echo "Example: $0 192.168.1"
    exit 1
fi

echo "Scanning $NETWORK.0/24 network..."
echo "=================================="

for HOST in {1..254}; do
    IP="$NETWORK.$HOST"
    
    # Ping with 1 packet, 1 second timeout
    if ping -c 1 -W 1 "$IP" &>/dev/null; then
        echo "✓ $IP is UP"
    fi
done

echo "Scan complete!"
```

### 12.2 Script: Port Scanner

```bash
#!/bin/bash
# Basic TCP port scanner

TARGET="$1"
START_PORT="${2:-1}"
END_PORT="${3:-1000}"

if [ -z "$TARGET" ]; then
    echo "Usage: $0 <target> [start_port] [end_port]"
    echo "Example: $0 192.168.1.1 1 1000"
    exit 1
fi

echo "Scanning $TARGET ports $START_PORT-$END_PORT..."
echo "================================================"

for PORT in $(seq "$START_PORT" "$END_PORT"); do
    # Try to connect using /dev/tcp
    if timeout 1 bash -c ">/dev/tcp/$TARGET/$PORT" 2>/dev/null; then
        echo "✓ Port $PORT is OPEN"
    fi
done

echo "Scan complete!"
```

### 12.3 Script: Service Enumerator

```bash
#!/bin/bash
# Enumerate common services on target

TARGET="$1"

if [ -z "$TARGET" ]; then
    echo "Usage: $0 <target>"
    exit 1
fi

echo "Enumerating services on $TARGET"
echo "=================================="

# Define common ports and services
declare -A SERVICES
SERVICES[21]="FTP"
SERVICES[22]="SSH"
SERVICES[23]="Telnet"
SERVICES[25]="SMTP"
SERVICES[53]="DNS"
SERVICES[80]="HTTP"
SERVICES[110]="POP3"
SERVICES[143]="IMAP"
SERVICES[443]="HTTPS"
SERVICES[445]="SMB"
SERVICES[3306]="MySQL"
SERVICES[3389]="RDP"
SERVICES[5432]="PostgreSQL"
SERVICES[8080]="HTTP-Alt"

for PORT in "${!SERVICES[@]}"; do
    SERVICE="${SERVICES[$PORT]}"
    
    if timeout 2 bash -c ">/dev/tcp/$TARGET/$PORT" 2>/dev/null; then
        echo "✓ $SERVICE (port $PORT) is OPEN"
        
        # Try to grab banner for some services
        if [ "$PORT" -eq 22 ]; then
            BANNER=$(timeout 2 nc "$TARGET" "$PORT" 2>/dev/null | head -1)
            [ -n "$BANNER" ] && echo "  Banner: $BANNER"
        fi
    fi
done

echo "Enumeration complete!"
```

### 12.4 Script: Web Directory Fuzzer

```bash
#!/bin/bash
# Simple web directory fuzzer

TARGET="$1"
WORDLIST="${2:-/usr/share/wordlists/dirb/common.txt}"

if [ -z "$TARGET" ]; then
    echo "Usage: $0 <target_url> [wordlist]"
    echo "Example: $0 http://192.168.1.100"
    exit 1
fi

if [ ! -f "$WORDLIST" ]; then
    echo "✗ Wordlist not found: $WORDLIST"
    exit 1
fi

echo "Fuzzing directories on $TARGET"
echo "Wordlist: $WORDLIST"
echo "==============================="

while IFS= read -r DIR; do
    URL="$TARGET/$DIR"
    
    # Send HTTP request
    RESPONSE=$(curl -s -o /dev/null -w "%{http_code}" --connect-timeout 2 "$URL")
    
    if [ "$RESPONSE" -eq 200 ]; then
        echo "✓ [200] $URL"
    elif [ "$RESPONSE" -eq 301 ] || [ "$RESPONSE" -eq 302 ]; then
        echo "↪  [$RESPONSE] $URL (redirect)"
    elif [ "$RESPONSE" -eq 403 ]; then
        echo "🔒 [403] $URL (forbidden)"
    fi
done < "$WORDLIST"

echo "Fuzzing complete!"
```

### 12.5 Script: User Enumeration

```bash
#!/bin/bash
# Enumerate valid usernames on Linux system

# Must run as root to read /etc/shadow
if [ "$UID" -ne 0 ]; then
    echo "✗ Must run as root"
    exit 1
fi

echo "Enumerating User Accounts"
echo "=========================="
echo ""

echo "[1] Users with UID >= 1000 (regular users):"
awk -F: '$3 >= 1000 && $1 != "nobody" {print "  " $1 " (UID " $3 ")"}' /etc/passwd

echo ""
echo "[2] Users with /bin/bash shell:"
grep "/bin/bash" /etc/passwd | awk -F: '{print "  " $1}'

echo ""
echo "[3] Users in sudo group:"
if getent group sudo &>/dev/null; then
    getent group sudo | awk -F: '{print $4}' | tr ',' '\n' | sed 's/^/  /'
fi

echo ""
echo "[4] Users with recent logins (last 7 days):"
lastlog -t 7 | tail -n +2 | grep -v "Never" | awk '{print "  " $1 " (Last: " $4 " " $5 " " $6 ")"}'

echo ""
echo "[5] Locked accounts:"
passwd -S -a 2>/dev/null | grep " L " | awk '{print "  " $1}'

echo ""
echo "Enumeration complete!"
```

---

## 13. Common Mistakes

### 13.1 Missing Spaces in [ ]

**WRONG**:
```bash
if ["$VAR" = "value"]  # ERROR!
```

**RIGHT**:
```bash
if [ "$VAR" = "value" ]  # Correct
```

**Why**: `[` is actually a command (test), so it needs spaces.

### 13.2 Forgetting to Quote Variables

**WRONG**:
```bash
VAR=""
if [ $VAR = "value" ]; then  # ERROR when VAR is empty!
```

**RIGHT**:
```bash
if [ "$VAR" = "value" ]; then  # Safe
```

### 13.3 Using = for Numeric Comparison

**WRONG**:
```bash
if [ $NUM = 10 ]; then  # Compares as string!
```

**RIGHT**:
```bash
if [ $NUM -eq 10 ]; then  # Compares as number
```

### 13.4 Forgetting ;;  in case Statements

**WRONG**:
```bash
case $VAR in
    1)
        echo "one"  # Missing ;;
    2)
        echo "two"
        ;;
esac
```

**RIGHT**:
```bash
case $VAR in
    1)
        echo "one"
        ;;
    2)
        echo "two"
        ;;
esac
```

### 13.5 Wrong Operator in [[ ]]

**WRONG**:
```bash
if [[ $A -eq $B ]]; then  # Works but strings compared as numbers
```

**BETTER**:
```bash
if [[ $A == $B ]]; then  # String comparison in [[]]
if (( A == B )); then    # Numeric comparison
```

### 13.6 Not Checking Command Success

**WRONG**:
```bash
cd /some/directory
rm *  # Dangerous if cd failed!
```

**RIGHT**:
```bash
if cd /some/directory; then
    rm *
else
    echo "Failed to change directory"
    exit 1
fi
```

### 13.7 Testing File Before Checking Existence

**WRONG**:
```bash
if [ -r "$FILE" ]; then
    cat "$FILE"  # -r checks readable, but file might not exist
fi
```

**BETTER**:
```bash
if [ -f "$FILE" ] && [ -r "$FILE" ]; then
    cat "$FILE"
fi
```

---

## 14. Hands-On Labs

### Lab 1: System Pre-Check Script

**Objective**: Write a script that validates the system meets requirements before installing software.

**Requirements**:
1. Check if running as root
2. Verify at least 1GB free disk space
3. Ensure network connectivity
4. Check if required commands exist (curl, wget, git)
5. Verify OS is Ubuntu or Debian

**Solution**:
```bash
#!/bin/bash
# System pre-check script

echo "System Requirements Check"
echo "=========================="

# Check 1: Root
if [ "$UID" -ne 0 ]; then
    echo "✗ Must run as root"
    exit 1
else
    echo "✓ Running as root"
fi

# Check 2: Disk space
FREE_SPACE=$(df / | tail -1 | awk '{print $4}')
if [ "$FREE_SPACE" -gt 1048576 ]; then  # 1GB in KB
    echo "✓ Sufficient disk space: $(( FREE_SPACE / 1024 ))MB free"
else
    echo "✗ Insufficient disk space: $(( FREE_SPACE / 1024 ))MB (need 1GB)"
    exit 1
fi

# Check 3: Network
if ping -c 1 -W 2 8.8.8.8 &>/dev/null; then
    echo "✓ Network connectivity OK"
else
    echo "✗ No network connectivity"
    exit 1
fi

# Check 4: Required commands
REQUIRED_CMDS="curl wget git"
for CMD in $REQUIRED_CMDS; do
    if command -v "$CMD" &>/dev/null; then
        echo "✓ $CMD is installed"
    else
        echo "✗ $CMD is not installed"
        exit 1
    fi
done

# Check 5: OS
if [ -f /etc/os-release ]; then
    . /etc/os-release
    if [[ "$ID" == "ubuntu" ]] || [[ "$ID" == "debian" ]]; then
        echo "✓ OS is compatible: $PRETTY_NAME"
    else
        echo "✗ OS not supported: $PRETTY_NAME"
        exit 1
    fi
fi

echo ""
echo "All checks passed! Ready to proceed."
```

### Lab 2: Interactive Service Manager

**Objective**: Create a menu-driven script to manage a service (start/stop/restart/status).

**Solution**:
```bash
#!/bin/bash
# Interactive service manager

SERVICE_NAME="apache2"

show_menu() {
    echo ""
    echo "Service Manager: $SERVICE_NAME"
    echo "=============================="
    echo "1) Start service"
    echo "2) Stop service"
    echo "3) Restart service"
    echo "4) Check status"
    echo "5) Exit"
    echo ""
    echo -n "Select option: "
}

while true; do
    show_menu
    read CHOICE
    
    case $CHOICE in
        1)
            echo "Starting $SERVICE_NAME..."
            if systemctl start "$SERVICE_NAME" 2>/dev/null; then
                echo "✓ Service started"
            else
                echo "✗ Failed to start service"
            fi
            ;;
        2)
            echo "Stopping $SERVICE_NAME..."
            if systemctl stop "$SERVICE_NAME" 2>/dev/null; then
                echo "✓ Service stopped"
            else
                echo "✗ Failed to stop service"
            fi
            ;;
        3)
            echo "Restarting $SERVICE_NAME..."
            if systemctl restart "$SERVICE_NAME" 2>/dev/null; then
                echo "✓ Service restarted"
            else
                echo "✗ Failed to restart service"
            fi
            ;;
        4)
            echo "Status of $SERVICE_NAME:"
            systemctl status "$SERVICE_NAME" --no-pager
            ;;
        5)
            echo "Exiting..."
            exit 0
            ;;
        *)
            echo "✗ Invalid option: $CHOICE"
            ;;
    esac
    
    echo ""
    echo -n "Press Enter to continue..."
    read
done
```

### Lab 3: Multi-Host Connectivity Checker

**Objective**: Check connectivity to multiple hosts and report status.

**Solution**:
```bash
#!/bin/bash
# Multi-host connectivity checker

# Define hosts to check
HOSTS=(
    "8.8.8.8:Google DNS"
    "1.1.1.1:Cloudflare DNS"
    "github.com:GitHub"
    "google.com:Google"
)

echo "Network Connectivity Check"
echo "==========================="
echo ""

TOTAL=0
UP=0
DOWN=0

for ENTRY in "${HOSTS[@]}"; do
    HOST=$(echo "$ENTRY" | cut -d: -f1)
    NAME=$(echo "$ENTRY" | cut -d: -f2)
    
    TOTAL=$(( TOTAL + 1 ))
    
    if ping -c 1 -W 2 "$HOST" &>/dev/null; then
        echo "✓ $NAME ($HOST) is reachable"
        UP=$(( UP + 1 ))
    else
        echo "✗ $NAME ($HOST) is unreachable"
        DOWN=$(( DOWN + 1 ))
    fi
done

echo ""
echo "Summary:"
echo "  Total hosts: $TOTAL"
echo "  Reachable:   $UP"
echo "  Unreachable: $DOWN"

if [ "$DOWN" -eq 0 ]; then
    echo ""
    echo "✓ All hosts reachable - network OK!"
    exit 0
else
    echo ""
    echo "⚠️  Some hosts unreachable - check network"
    exit 1
fi
```

---

## 15. Interview Questions & Checkpoints

### Question 1: What's the difference between [ ] and [[ ]]?

**Answer**:
- `[ ]` is the old POSIX test command (works in sh, bash, etc.)
- `[[ ]]` is Bash-specific with more features
- `[[ ]]` supports pattern matching, regex, and safer variable handling
- Use `[[ ]]` for Bash scripts, `[ ]` for portability

### Question 2: How do you check if a file exists?

**Answer**:
```bash
if [ -e filename ]; then    # Exists (file or directory)
if [ -f filename ]; then    # Is a regular file
if [ -d filename ]; then    # Is a directory
```

### Question 3: What's wrong with this code?

```bash
VAR=""
if [ $VAR = "test" ]; then
    echo "match"
fi
```

**Answer**: Missing quotes around `$VAR`. When VAR is empty, it becomes `[ = "test" ]` which is a syntax error. Should be: `if [ "$VAR" = "test" ]`

### Question 4: How do you compare numbers in Bash?

**Answer**:
```bash
if [ $A -eq $B ]  # Equal
if [ $A -ne $B ]  # Not equal
if [ $A -lt $B ]  # Less than
if [ $A -gt $B ]  # Greater than
```

### Question 5: What does this do?

```bash
if [ ! -f "config.txt" ]; then
    echo "Creating config..."
fi
```

**Answer**: Checks if "config.txt" does NOT exist as a regular file. The `!` negates the condition.

---

## 16. Summary

**What You Learned**:

1. **If/Then/Else Logic**: Basic conditional structures for decision making
2. **Test Operators**: `[ ]` vs `[[ ]]` and when to use each
3. **File Tests**: Checking file existence, type, and permissions
4. **String Comparisons**: Equality, emptiness, pattern matching
5. **Numeric Comparisons**: Proper number comparison with -eq, -lt, -gt, etc.
6. **Logical Operators**: Combining conditions with AND, OR, NOT
7. **Case Statements**: Multi-way branching for menus and pattern matching
8. **Security Applications**: Root checks, port scanning, permission validation

**Key Takeaways**:
- Always quote variables in `[ ]` tests
- Use `[[ ]]` for Bash-specific features
- `-eq` for numbers, `=` for strings
- Check file existence before operations
- Case statements are cleaner than multiple if/elif
- Test scripts with both valid and invalid inputs

**Next Steps**:
- Practice writing validation scripts
- Combine conditionals with loops (next module)
- Build security automation tools
- Create interactive menu systems

**File Count**: 1,100+ lines
**Examples**: 20+ working scripts
**Coverage**: Complete conditional logic for OSCP/security work

---

**Remember**: Conditional logic is the foundation of intelligent scripts. Master these concepts and you can build automation that adapts to any situation!
