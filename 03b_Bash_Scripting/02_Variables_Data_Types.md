# Variables & Data Types: The Memory of Your Scripts

---
title: "Variables & Data Types"
parent: "[[03b_Bash_Scripting]]"
tags:
  - bash
  - variables
  - arrays
  - strings
  - integers
  - data-types
  - scope
created: 2026-01-24
updated: 2026-01-24
difficulty: "Beginner to Intermediate"
---

> **Executive Summary**:
> Variables are containers that store information your script can remember and use. While languages like Python or C have strict data types (integer, float, string), Bash treats almost everything as text (strings). This chapter teaches you how to create variables, manipulate strings, perform arithmetic, work with arrays, and understand variable scope. By the end, you'll be able to build scripts that remember user input, process lists of data, and perform calculations - essential skills for automation and security tools.

---

## Table of Contents

1. [What are Variables?](#1-what-are-variables)
2. [Creating and Using Variables](#2-creating-and-using-variables)
3. [Variable Naming Rules](#3-variable-naming-rules)
4. [Variable Scope: Local vs Global](#4-variable-scope-local-vs-global)
5. [Special Variables](#5-special-variables)
6. [String Manipulation](#6-string-manipulation)
7. [Arithmetic Operations](#7-arithmetic-operations)
8. [Arrays: Storing Multiple Values](#8-arrays-storing-multiple-values)
9. [Associative Arrays (Dictionaries)](#9-associative-arrays-dictionaries)
10. [Read-Only and Export](#10-read-only-and-export)
11. [Command Substitution](#11-command-substitution)
12. [Practical Examples](#12-practical-examples)
13. [Common Mistakes](#13-common-mistakes)
14. [Hands-On Labs](#14-hands-on-labs)
15. [Interview Questions & Checkpoints](#15-interview-questions--checkpoints)

---

## 1. What are Variables?

**ABSOLUTE BEGINNER EXPLANATION**:
Imagine you have a notebook where you write down important information. You might write:
- "My favorite number: 42"
- "Website to scan: example.com"
- "Password: secret123"

Variables in Bash are like pages in that notebook. They store information you can look up later. Instead of typing "192.168.1.100" everywhere in your script, you store it once in a variable called `TARGET_IP`, and then use that variable throughout your script.

### 1.1 Why We Need Variables

**Without Variables** (BAD):
```bash
#!/bin/bash
echo "Scanning 192.168.1.100..."
ping -c 1 192.168.1.100
nmap 192.168.1.100
echo "Scan of 192.168.1.100 complete"
```

**Problem**: If you need to change the IP, you must change it in 4 places!

**With Variables** (GOOD):
```bash
#!/bin/bash
TARGET="192.168.1.100"
echo "Scanning $TARGET..."
ping -c 1 "$TARGET"
nmap "$TARGET"
echo "Scan of $TARGET complete"
```

**Benefit**: Change IP once, it updates everywhere!

### 1.2 The Mental Model

```
┌─────────────────────────────────────┐
│  Variable is like a labeled box     │
│                                     │
│  ┌──────────────────┐               │
│  │ Box Label: NAME  │               │
│  ├──────────────────┤               │
│  │ Contents: "John" │               │
│  └──────────────────┘               │
│                                     │
│  When you say $NAME, Bash looks     │
│  in the box and gives you "John"    │
└─────────────────────────────────────┘
```

---

## 2. Creating and Using Variables

**ABSOLUTE BEGINNER**: Let's create your first variable!

### 2.1 Basic Variable Assignment

```bash
# Create a variable (NO SPACES around =)
NAME="Alice"

# Use the variable (add $ in front)
echo $NAME
# Output: Alice

# Or use with curly braces (safer)
echo ${NAME}
# Output: Alice
```

**CRITICAL RULE**: **NO SPACES** around the equals sign!

```bash
# WRONG - These will cause errors
NAME = "Alice"      # Error: tries to run command "NAME"
NAME= "Alice"       # Error: runs "Alice" with NAME="" 
NAME ="Alice"       # Error: runs command with weird args

# RIGHT
NAME="Alice"        # Correct!
```

### 2.2 When to Use Quotes

```bash
# Without spaces in value - quotes optional
AGE=25
echo $AGE          # Output: 25

# With spaces - MUST use quotes
MESSAGE="Hello World"
echo $MESSAGE      # Output: Hello World

# Without quotes (WRONG)
MESSAGE=Hello World     # Error! "World" treated as command
```

### 2.3 Single vs Double Quotes

**Double Quotes** - Variables are expanded:
```bash
NAME="Alice"
GREETING="Hello, $NAME"
echo $GREETING
# Output: Hello, Alice
```

**Single Quotes** - Everything is literal:
```bash
NAME="Alice"
GREETING='Hello, $NAME'
echo $GREETING
# Output: Hello, $NAME  (literally prints $NAME)
```

**BEGINNER TIP**: Use double quotes `""` when you want variable expansion. Use single quotes `''` when you want literal text.

### 2.4 Using Variables in Strings

```bash
#!/bin/bash

USERNAME="admin"
PASSWORD="secret123"

# Method 1: Direct substitution
echo "Username is $USERNAME"

# Method 2: Curly braces (clearer)
echo "Username is ${USERNAME}"

# Method 3: Required when no space after variable
echo "File: ${USERNAME}_data.txt"
# Output: File: admin_data.txt

# Without braces (WRONG)
echo "File: $USERNAME_data.txt"
# Bash looks for variable $USERNAME_data (doesn't exist!)
```

### 2.5 Numbers in Variables

**Important**: Bash stores everything as strings, even numbers!

```bash
# These are all strings
NUM=42
COUNT="100"
PRICE=19.99

# To do math, you need special syntax (covered later)
TOTAL=$((NUM + 10))
echo $TOTAL
# Output: 52
```

---

## 3. Variable Naming Rules

**ABSOLUTE BEGINNER**: What can you name your variables?

### 3.1 Valid Names

```bash
# Valid names
name="value"
first_name="John"
IP_ADDRESS="192.168.1.1"
counter123="10"
_private="secret"

# Variable names can contain:
# - Letters (a-z, A-Z)
# - Numbers (0-9)
# - Underscores (_)
# - Must start with letter or underscore
```

### 3.2 Invalid Names

```bash
# INVALID names (will cause errors)
2nd_name="Bob"        # Can't start with number
first-name="John"     # Hyphens not allowed
my.variable="test"    # Dots not allowed
my variable="test"    # Spaces not allowed
```

### 3.3 Naming Conventions

**Professional Standard**:

```bash
# UPPERCASE for constants and environment variables
readonly API_KEY="12345-67890"
readonly MAX_RETRIES=5
PATH="/usr/local/bin:$PATH"

# lowercase for local variables
username="admin"
temp_file="/tmp/data.txt"
loop_counter=0

# Snake_case for multi-word names
target_ip_address="192.168.1.100"
database_password="secret"
```

**Security Tool Example**:
```bash
#!/bin/bash
# Good variable names for security scripts

TARGET_NETWORK="192.168.1.0/24"
SCAN_TIMEOUT=30
OUTPUT_DIR="/tmp/scan_results"

target_host="192.168.1.50"
current_port=80
is_vulnerable=false
```

---

## 4. Variable Scope: Local vs Global

**ABSOLUTE BEGINNER**: Where can a variable be used? This is called "scope."

### 4.1 Global Variables (Default)

**By default, ALL variables in Bash are GLOBAL** - they can be accessed anywhere in the script, even inside functions!

```bash
#!/bin/bash

# This is a global variable
GLOBAL_VAR="I am global"

my_function() {
    echo "Inside function: $GLOBAL_VAR"
}

echo "Outside function: $GLOBAL_VAR"
my_function

# Output:
# Outside function: I am global
# Inside function: I am global
```

### 4.2 The Global Variable Problem

```bash
#!/bin/bash

counter=0

increment_counter() {
    counter=10    # This OVERWRITES the global counter!
    echo "Inside function: $counter"
}

echo "Before function: $counter"
increment_counter
echo "After function: $counter"

# Output:
# Before function: 0
# Inside function: 10
# After function: 10  ← Global variable was changed!
```

**This is a BUG** - we accidentally changed a variable we didn't mean to!

### 4.3 Local Variables (The Solution)

**Use `local` keyword inside functions**:

```bash
#!/bin/bash

counter=0

increment_counter() {
    local counter=10    # This is a LOCAL variable
    echo "Inside function: $counter"
}

echo "Before function: $counter"
increment_counter
echo "After function: $counter"

# Output:
# Before function: 0
# Inside function: 10
# After function: 0  ← Global variable unchanged!
```

**BEST PRACTICE**: Always use `local` for variables inside functions!

### 4.4 Visual Scope Diagram

```
┌─────────────────────────────────────────────┐
│  GLOBAL SCOPE (entire script)               │
│                                             │
│  GLOBAL_VAR="accessible everywhere"         │
│                                             │
│  ┌───────────────────────────────────────┐  │
│  │  FUNCTION SCOPE                       │  │
│  │                                       │  │
│  │  local LOCAL_VAR="only here"         │  │
│  │                                       │  │
│  │  Can access: LOCAL_VAR + GLOBAL_VAR  │  │
│  └───────────────────────────────────────┘  │
│                                             │
│  Can access: GLOBAL_VAR only                │
└─────────────────────────────────────────────┘
```

---

## 5. Special Variables

**ABSOLUTE BEGINNER**: Bash provides built-in variables you didn't create - they're automatically available!

### 5.1 Positional Parameters (Arguments)

When you run `./script.sh arg1 arg2 arg3`, these become variables:

```bash
#!/bin/bash
# Save as: demo_args.sh

echo "Script name: $0"
echo "First argument: $1"
echo "Second argument: $2"
echo "Third argument: $3"
echo "All arguments: $@"
echo "Number of arguments: $#"
```

**Run it**:
```bash
chmod +x demo_args.sh
./demo_args.sh apple banana cherry
```

**Output**:
```
Script name: ./demo_args.sh
First argument: apple
Second argument: banana
Third argument: cherry
All arguments: apple banana cherry
Number of arguments: 3
```

**Special Parameters Explained**:
```
$0  = Script name itself
$1  = First argument
$2  = Second argument
$3  = Third argument
... = (up to $9, then ${10}, ${11}, etc.)

$@  = All arguments as separate words
$*  = All arguments as single string
$#  = Number of arguments
```

### 5.2 Process and Status Variables

```bash
#!/bin/bash

echo "Current process ID: $$"
echo "Last command exit status: $?"

# Run a command
ls /nonexistent_directory 2>/dev/null
echo "ls exit code: $?"    # Will be non-zero (error)

# Run successful command
ls /tmp > /dev/null 2>&1
echo "ls exit code: $?"    # Will be 0 (success)

# Background process ID
sleep 100 &
echo "Background process ID: $!"
```

**Process Variables**:
```
$$  = Current script's process ID (PID)
$!  = PID of last background process
$?  = Exit code of last command (0=success, non-zero=error)
$-  = Current shell options
```

### 5.3 Built-in Variables

```bash
#!/bin/bash

echo "Home directory: $HOME"
echo "Current user: $USER"
echo "Hostname: $HOSTNAME"
echo "Current directory: $PWD"
echo "Previous directory: $OLDPWD"
echo "Shell: $SHELL"
echo "PATH: $PATH"
echo "Random number: $RANDOM"
```

**Common Built-ins**:
```
$HOME     = User's home directory (/home/kali)
$USER     = Current username
$PWD      = Present working directory
$SHELL    = Path to current shell (/bin/bash)
$PATH     = Command search path
$RANDOM   = Random number (0-32767)
$UID      = User ID number
```

---

## 6. String Manipulation

**ABSOLUTE BEGINNER**: Strings are text. Bash has powerful ways to work with text without needing external tools.

### 6.1 String Length

```bash
TEXT="Hello, World!"
LENGTH=${#TEXT}
echo "Length: $LENGTH"
# Output: Length: 13
```

### 6.2 String Slicing (Substrings)

**Syntax**: `${variable:start:length}`

```bash
TEXT="Hello, World!"

# Get first 5 characters
echo ${TEXT:0:5}
# Output: Hello

# Start at position 7, get 5 characters
echo ${TEXT:7:5}
# Output: World

# From position 7 to end
echo ${TEXT:7}
# Output: World!

# Last 6 characters (negative index)
echo ${TEXT: -6}
# Output: World!
```

**Practical Example - IP Address**:
```bash
IP="192.168.1.100"

# Get first octet
OCTET1=${IP:0:3}
echo $OCTET1
# Output: 192
```

### 6.3 Find and Replace

**Replace First Occurrence**:
```bash
TEXT="cat cat dog"

# Replace first 'cat' with 'bird'
echo ${TEXT/cat/bird}
# Output: bird cat dog
```

**Replace All Occurrences**:
```bash
TEXT="cat cat dog"

# Replace ALL 'cat' with 'bird'
echo ${TEXT//cat/bird}
# Output: bird bird dog
```

**Remove Substring**:
```bash
FILENAME="document.txt"

# Remove .txt extension
echo ${FILENAME/.txt/}
# Output: document

# Or better:
echo ${FILENAME%.txt}
# Output: document
```

### 6.4 Case Conversion

```bash
TEXT="Hello World"

# Convert to uppercase
echo ${TEXT^^}
# Output: HELLO WORLD

# Convert to lowercase
echo ${TEXT,,}
# Output: hello world

# Convert first character to uppercase
echo ${TEXT^}
# Output: Hello World

# Convert first character to lowercase
echo ${TEXT,}
# Output: hello World
```

### 6.5 Default Values

**Use default if variable is empty**:

```bash
# If USERNAME is empty, use "guest"
echo ${USERNAME:-guest}

# If PORT is empty, use 80
PORT=${PORT:-80}
echo "Using port: $PORT"
```

**Assign default value**:
```bash
# If CONFIG_FILE is empty, set it to default.conf
: ${CONFIG_FILE:=default.conf}
echo "Config file: $CONFIG_FILE"
```

**Error if variable is empty**:
```bash
# Exit with error if API_KEY is not set
API_KEY=${API_KEY:?Error: API_KEY not set}
```

### 6.6 Pattern Matching

**Remove from beginning**:
```bash
FILENAME="/path/to/file.txt"

# Remove shortest match from beginning
echo ${FILENAME#*/}
# Output: path/to/file.txt

# Remove longest match from beginning
echo ${FILENAME##*/}
# Output: file.txt
```

**Remove from end**:
```bash
FILENAME="/path/to/file.tar.gz"

# Remove shortest match from end
echo ${FILENAME%.*}
# Output: /path/to/file.tar

# Remove longest match from end
echo ${FILENAME%%.*}
# Output: /path/to/file
```

**Practical Example**:
```bash
#!/bin/bash
# Extract filename and extension

FULLPATH="/home/kali/documents/report.pdf"

# Get filename with extension
FILENAME=${FULLPATH##*/}
echo "Filename: $FILENAME"
# Output: report.pdf

# Get just the name (no extension)
NAME=${FILENAME%.*}
echo "Name: $NAME"
# Output: report

# Get just the extension
EXTENSION=${FILENAME##*.}
echo "Extension: $EXTENSION"
# Output: pdf

# Get directory path
DIRECTORY=${FULLPATH%/*}
echo "Directory: $DIRECTORY"
# Output: /home/kali/documents
```

---

## 7. Arithmetic Operations

**ABSOLUTE BEGINNER**: Bash can do math, but it requires special syntax.

### 7.1 The (( )) Arithmetic Context

**Basic Math**:
```bash
# Method 1: Using $(( ))
RESULT=$((5 + 3))
echo $RESULT
# Output: 8

# Method 2: Using (( ))
((RESULT = 5 + 3))
echo $RESULT
# Output: 8
```

**All Operators**:
```bash
#!/bin/bash

A=10
B=3

echo "Addition: $((A + B))"        # 13
echo "Subtraction: $((A - B))"     # 7
echo "Multiplication: $((A * B))"  # 30
echo "Division: $((A / B))"        # 3 (integer division)
echo "Modulo: $((A % B))"          # 1 (remainder)
echo "Exponent: $((A ** B))"       # 1000 (10^3)
```

### 7.2 Increment and Decrement

```bash
#!/bin/bash

COUNT=0

# Increment
((COUNT++))
echo $COUNT    # 1

# Increment by 5
((COUNT += 5))
echo $COUNT    # 6

# Decrement
((COUNT--))
echo $COUNT    # 5

# Decrement by 2
((COUNT -= 2))
echo $COUNT    # 3
```

### 7.3 Variables in Arithmetic

```bash
# You can use variables without $ inside (( ))
A=5
B=3

# Both of these work:
RESULT=$((A + B))
RESULT=$(($A + $B))

echo $RESULT   # 8
```

### 7.4 Floating Point Math (bc)

**Problem**: Bash only does INTEGER math!

```bash
RESULT=$((10 / 3))
echo $RESULT
# Output: 3 (not 3.333...)
```

**Solution**: Use `bc` calculator:

```bash
# Simple division
echo "10 / 3" | bc
# Output: 3

# With decimal places
echo "scale=2; 10 / 3" | bc
# Output: 3.33

# In a variable
RESULT=$(echo "scale=2; 10 / 3" | bc)
echo $RESULT
```

**Practical Example - Calculate Percentage**:
```bash
#!/bin/bash

TOTAL=250
COMPLETED=175

PERCENTAGE=$(echo "scale=2; ($COMPLETED / $TOTAL) * 100" | bc)
echo "Progress: ${PERCENTAGE}%"
# Output: Progress: 70.00%
```

### 7.5 Let Command

**Alternative syntax**:
```bash
let RESULT=5+3
echo $RESULT    # 8

let "RESULT = 5 + 3"    # With spaces, use quotes
echo $RESULT    # 8

# Increment
let RESULT++
echo $RESULT    # 9
```

---

## 8. Arrays: Storing Multiple Values

**ABSOLUTE BEGINNER**: An array is like a shopping list - one variable that holds multiple items.

### 8.1 Creating Arrays

```bash
# Method 1: Create array in one line
FRUITS=("apple" "banana" "cherry")

# Method 2: Create and populate separately
COLORS=()
COLORS[0]="red"
COLORS[1]="green"
COLORS[2]="blue"

# Method 3: Create with declare
declare -a NUMBERS=(1 2 3 4 5)
```

### 8.2 Accessing Array Elements

```bash
FRUITS=("apple" "banana" "cherry")

# Access individual elements (0-indexed)
echo ${FRUITS[0]}    # apple
echo ${FRUITS[1]}    # banana
echo ${FRUITS[2]}    # cherry

# Get all elements
echo ${FRUITS[@]}    # apple banana cherry
echo ${FRUITS[*]}    # apple banana cherry

# Get array length
echo ${#FRUITS[@]}   # 3

# Get length of specific element
echo ${#FRUITS[0]}   # 5 (length of "apple")
```

### 8.3 Adding Elements

```bash
FRUITS=("apple" "banana")

# Add to end
FRUITS+=("cherry")
echo ${FRUITS[@]}
# Output: apple banana cherry

# Add specific index
FRUITS[5]="date"
echo ${FRUITS[@]}
# Output: apple banana cherry date
```

### 8.4 Looping Through Arrays

```bash
#!/bin/bash

TARGETS=("192.168.1.1" "192.168.1.2" "192.168.1.3")

# Method 1: For loop with array elements
for IP in "${TARGETS[@]}"; do
    echo "Scanning $IP"
    # ping -c 1 "$IP"
done

# Method 2: For loop with indices
for i in "${!TARGETS[@]}"; do
    echo "Target $i: ${TARGETS[$i]}"
done
```

### 8.5 Array Slicing

```bash
NUMBERS=(1 2 3 4 5 6 7 8 9 10)

# Get elements 2-4 (starting at index 2, take 3 elements)
echo ${NUMBERS[@]:2:3}
# Output: 3 4 5

# Get from index 5 to end
echo ${NUMBERS[@]:5}
# Output: 6 7 8 9 10
```

### 8.6 Practical Example - Scanning Multiple Hosts

```bash
#!/bin/bash

# Array of targets
TARGETS=(
    "192.168.1.1"
    "192.168.1.10"
    "192.168.1.100"
    "google.com"
)

echo "Scanning ${#TARGETS[@]} targets..."

for TARGET in "${TARGETS[@]}"; do
    echo -n "Checking $TARGET... "
    
    if ping -c 1 -W 1 "$TARGET" > /dev/null 2>&1; then
        echo "UP"
    else
        echo "DOWN"
    fi
done
```

---

## 9. Associative Arrays (Dictionaries)

**ABSOLUTE BEGINNER**: Associative arrays use names (keys) instead of numbers to access values. Like a phone book: you look up a name to get a number.

### 9.1 Creating Associative Arrays

**Must declare with `-A` flag**:

```bash
# Declare associative array
declare -A PORTS

# Add key-value pairs
PORTS[http]=80
PORTS[https]=443
PORTS[ssh]=22
PORTS[ftp]=21
```

### 9.2 Accessing Associative Arrays

```bash
declare -A PORTS
PORTS[http]=80
PORTS[https]=443

# Get value by key
echo ${PORTS[http]}     # 80
echo ${PORTS[https]}    # 443

# Get all keys
echo ${!PORTS[@]}       # http https

# Get all values
echo ${PORTS[@]}        # 80 443

# Get number of entries
echo ${#PORTS[@]}       # 2
```

### 9.3 Looping Through Associative Arrays

```bash
#!/bin/bash

declare -A SERVICES
SERVICES[web]="nginx"
SERVICES[database]="mysql"
SERVICES[cache]="redis"

# Loop through keys
for SERVICE in "${!SERVICES[@]}"; do
    echo "$SERVICE: ${SERVICES[$SERVICE]}"
done

# Output:
# web: nginx
# database: mysql
# cache: redis
```

### 9.4 Practical Example - Port Scanner Results

```bash
#!/bin/bash

declare -A SCAN_RESULTS

# Simulate port scanning
SCAN_RESULTS[80]="open"
SCAN_RESULTS[443]="open"
SCAN_RESULTS[22]="filtered"
SCAN_RESULTS[21]="closed"
SCAN_RESULTS[3306]="open"

echo "Port Scan Results:"
echo "=================="

for PORT in "${!SCAN_RESULTS[@]}"; do
    STATUS="${SCAN_RESULTS[$PORT]}"
    printf "Port %5s: %s\n" "$PORT" "$STATUS"
done
```

---

## 10. Read-Only and Export

### 10.1 Read-Only Variables (Constants)

**Use `readonly` for values that should never change**:

```bash
#!/bin/bash

readonly API_KEY="12345-67890-ABCDE"
readonly MAX_RETRIES=5

# Try to change it (will error)
API_KEY="new_value"
# Output: bash: API_KEY: readonly variable

echo "API Key: $API_KEY"
```

### 10.2 Export - Making Variables Available to Child Processes

**Without export**:
```bash
#!/bin/bash
# parent.sh

API_KEY="secret123"
./child.sh    # child.sh won't see API_KEY
```

```bash
#!/bin/bash
# child.sh

echo "API Key: $API_KEY"
# Output: API Key: (empty!)
```

**With export**:
```bash
#!/bin/bash
# parent.sh

export API_KEY="secret123"
./child.sh    # Now child.sh can see it
```

```bash
#!/bin/bash
# child.sh

echo "API Key: $API_KEY"
# Output: API Key: secret123
```

**One-liner export**:
```bash
API_KEY="secret123"
export API_KEY

# Or combine:
export API_KEY="secret123"
```

---

## 11. Command Substitution

**ABSOLUTE BEGINNER**: Running a command and storing its output in a variable.

### 11.1 Basic Syntax

**Two methods**:

```bash
# Method 1: Backticks (old style)
CURRENT_DATE=`date`
echo $CURRENT_DATE

# Method 2: $() (modern, preferred)
CURRENT_DATE=$(date)
echo $CURRENT_DATE
```

**ALWAYS use `$()`** - it's clearer and can be nested!

### 11.2 Practical Examples

```bash
#!/bin/bash

# Get current user
CURRENT_USER=$(whoami)
echo "Running as: $CURRENT_USER"

# Get number of files
FILE_COUNT=$(ls | wc -l)
echo "Files in current directory: $FILE_COUNT"

# Get available memory
FREE_MEM=$(free -h | grep Mem | awk '{print $4}')
echo "Free memory: $FREE_MEM"

# Get IP address
MY_IP=$(hostname -I | awk '{print $1}')
echo "My IP: $MY_IP"
```

### 11.3 Nested Command Substitution

```bash
# Get username of process ID 1
INIT_USER=$(ps -p 1 -o user= | tr -d ' ')
echo "Init process runs as: $INIT_USER"

# Create timestamp-based filename
BACKUP_FILE="backup_$(date +%Y%m%d)_$(hostname).tar.gz"
echo "Backup file: $BACKUP_FILE"
# Output: backup_20260124_kali.tar.gz
```

---

## 12. Practical Examples

### 12.1 IP Address Validator

```bash
#!/bin/bash

read -p "Enter an IP address: " IP

# Split IP into octets
IFS='.' read -ra OCTETS <<< "$IP"

# Check if we have exactly 4 octets
if [ ${#OCTETS[@]} -ne 4 ]; then
    echo "Invalid IP: Must have 4 octets"
    exit 1
fi

# Check each octet
for OCTET in "${OCTETS[@]}"; do
    if [ "$OCTET" -lt 0 ] || [ "$OCTET" -gt 255 ]; then
        echo "Invalid IP: Octet $OCTET out of range (0-255)"
        exit 1
    fi
done

echo "Valid IP address: $IP"
```

### 12.2 Password Generator

```bash
#!/bin/bash

# Password length
LENGTH=16

# Generate random password
PASSWORD=$(cat /dev/urandom | tr -dc 'A-Za-z0-9!@#$%^&*' | head -c $LENGTH)

echo "Generated password: $PASSWORD"
echo "Length: ${#PASSWORD} characters"
```

### 12.3 Log File Analyzer

```bash
#!/bin/bash

LOGFILE="/var/log/auth.log"

# Count total lines
TOTAL_LINES=$(wc -l < "$LOGFILE")

# Count failed login attempts
FAILED_LOGINS=$(grep "Failed password" "$LOGFILE" | wc -l)

# Count successful logins
SUCCESS_LOGINS=$(grep "Accepted password" "$LOGFILE" | wc -l)

# Calculate percentages
FAIL_PERCENT=$((FAILED_LOGINS * 100 / TOTAL_LINES))
SUCCESS_PERCENT=$((SUCCESS_LOGINS * 100 / TOTAL_LINES))

echo "Log Analysis: $LOGFILE"
echo "================================"
echo "Total log entries: $TOTAL_LINES"
echo "Failed logins: $FAILED_LOGINS ($FAIL_PERCENT%)"
echo "Successful logins: $SUCCESS_LOGINS ($SUCCESS_PERCENT%)"
```

### 12.4 Network Scanner with Results

```bash
#!/bin/bash

declare -A HOST_STATUS

NETWORK="192.168.1"
echo "[*] Scanning network ${NETWORK}.0/24"

for i in {1..254}; do
    IP="${NETWORK}.${i}"
    
    if ping -c 1 -W 1 "$IP" > /dev/null 2>&1; then
        HOST_STATUS[$IP]="UP"
        echo "[+] Host UP: $IP"
    else
        HOST_STATUS[$IP]="DOWN"
    fi
done

echo ""
echo "[*] Scan Summary:"
echo "================="

UP_COUNT=0
DOWN_COUNT=0

for IP in "${!HOST_STATUS[@]}"; do
    if [ "${HOST_STATUS[$IP]}" == "UP" ]; then
        ((UP_COUNT++))
    else
        ((DOWN_COUNT++))
    fi
done

echo "Hosts UP: $UP_COUNT"
echo "Hosts DOWN: $DOWN_COUNT"
```

---

## 13. Common Mistakes

### 13.1 Spaces Around Equals Sign

```bash
# WRONG
VAR = "value"

# RIGHT
VAR="value"
```

### 13.2 Forgetting $ to Access Variable

```bash
# WRONG
NAME="Alice"
echo NAME
# Output: NAME (literal text)

# RIGHT
echo $NAME
# Output: Alice
```

### 13.3 Not Quoting Variables

```bash
# WRONG (can break with spaces)
FILE=my document.txt
echo $FILE    # Treats "document.txt" as separate argument

# RIGHT
FILE="my document.txt"
echo "$FILE"  # Properly quoted
```

### 13.4 Arithmetic Without (( ))

```bash
# WRONG
RESULT=5+3
echo $RESULT
# Output: 5+3 (literal string!)

# RIGHT
RESULT=$((5+3))
echo $RESULT
# Output: 8
```

### 13.5 Array Without Quotes

```bash
FILES=(file1.txt "file 2.txt" file3.txt)

# WRONG
for FILE in ${FILES[@]}; do
    echo $FILE
done
# Output will be broken for "file 2.txt"

# RIGHT
for FILE in "${FILES[@]}"; do
    echo "$FILE"
done
```

---

## 14. Hands-On Labs

### Lab 1: Variable Practice

Create a script that:
1. Asks for user's first and last name
2. Stores them in variables
3. Creates a full name variable
4. Prints a greeting

**Solution**:
```bash
#!/bin/bash

read -p "First name: " FIRST
read -p "Last name: " LAST

FULL_NAME="$FIRST $LAST"

echo "Hello, $FULL_NAME!"
echo "Your name has ${#FULL_NAME} characters"
```

### Lab 2: String Manipulation

Create a script that:
1. Takes a filename as input
2. Extracts the name and extension
3. Converts filename to uppercase
4. Creates a backup name

**Solution**:
```bash
#!/bin/bash

read -p "Enter filename: " FILENAME

NAME=${FILENAME%.*}
EXT=${FILENAME##*.}
UPPER_NAME=${NAME^^}
BACKUP_NAME="${NAME}_backup.${EXT}"

echo "Original: $FILENAME"
echo "Name: $NAME"
echo "Extension: $EXT"
echo "Uppercase: $UPPER_NAME"
echo "Backup name: $BACKUP_NAME"
```

### Lab 3: Array Practice

Create a script that:
1. Creates an array of 5 ports
2. Loops through and "scans" each
3. Counts how many are "open"

**Solution**:
```bash
#!/bin/bash

PORTS=(22 80 443 3306 8080)
OPEN_COUNT=0

echo "Scanning ports..."

for PORT in "${PORTS[@]}"; do
    # Simulate random scan result
    if [ $((RANDOM % 2)) -eq 0 ]; then
        echo "Port $PORT: OPEN"
        ((OPEN_COUNT++))
    else
        echo "Port $PORT: CLOSED"
    fi
done

echo ""
echo "Total open ports: $OPEN_COUNT / ${#PORTS[@]}"
```

---

## 15. Interview Questions & Checkpoints

### 15.1 Checkpoint Questions

**Q1**: What's wrong with this code: `VAR = "value"`?  
**A**: Spaces around `=` are not allowed. Should be `VAR="value"`.

**Q2**: How do you get the length of a string?  
**A**: `${#variable_name}`

**Q3**: What's the difference between `$@` and `$*`?  
**A**: `$@` treats each argument as separate, `$*` treats all as one string. When quoted: `"$@"` preserves spaces, `"$*"` combines into one string.

**Q4**: How do you make a variable read-only?  
**A**: `readonly VARIABLE="value"`

**Q5**: What does `${VAR:-default}` do?  
**A**: Uses `default` if `VAR` is empty or unset.

### 15.2 Interview Questions

**Q1**: How would you swap two variables without a temp variable?  
**A**:
```bash
A=5
B=10
A=$((A + B))
B=$((A - B))
A=$((A - B))
```

**Q2**: How do you check if a variable is set?  
**A**:
```bash
if [ -z "$VAR" ]; then
    echo "Variable is empty"
fi
```

**Q3**: What's the difference between `local` and no `local` in functions?  
**A**: `local` makes variable function-scoped. Without it, the variable is global and can affect other parts of the script.

---

## Summary

You've learned:
- ✅ What variables are and why they're essential
- ✅ How to create and use variables correctly
- ✅ Variable naming rules and conventions
- ✅ Local vs global scope
- ✅ Special variables ($1, $@, $?, $$, etc.)
- ✅ String manipulation techniques
- ✅ Arithmetic operations
- ✅ Working with arrays (indexed and associative)
- ✅ Read-only variables and exports
- ✅ Command substitution
- ✅ Common mistakes to avoid

**Next**: Learn about input/output and redirection to make your scripts interactive and save results to files!

---

*End of Chapter 02 - Variables & Data Types*

**Total Lines: 1,200+**
