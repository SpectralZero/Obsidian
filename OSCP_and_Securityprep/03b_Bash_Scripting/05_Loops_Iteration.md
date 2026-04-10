# 05 - Loops & Iteration: Automating Repetitive Tasks

> **Executive Summary**: Loops are the automation workhorses of Bash scripting. They let you repeat actions multiple times without writing the same code over and over. Whether you're scanning 254 IP addresses, processing 1000 log files, or trying passwords from a wordlist, loops make it possible. This module covers for loops (both traditional and C-style), while loops, until loops, loop control (break/continue), nested loops, and iterating over files, arrays, and command output. You'll learn how to build efficient automation for security tasks like port scanning, file processing, brute-forcing, and network enumeration - essential skills for OSCP and real-world penetration testing.

**What You'll Learn**:
- Complete beginner's guide to loops and iteration
- For loops (list-based and C-style)
- While loops (condition-based repetition)
- Until loops (opposite of while)
- Loop control with break and continue
- Nested loops for multi-dimensional tasks
- Reading files line-by-line
- Iterating over arrays and command output
- 20+ practical security/automation examples
- Real OSCP-style enumeration scripts

---

## Table of Contents

1. [What are Loops? (ABSOLUTE BEGINNER)](#1-what-are-loops)
2. [For Loops - Basic Syntax](#2-for-loops)
3. [For Loops with Ranges](#3-for-loops-with-ranges)
4. [C-Style For Loops](#4-c-style-for-loops)
5. [While Loops](#5-while-loops)
6. [Until Loops](#6-until-loops)
7. [Loop Control: break and continue](#7-loop-control)
8. [Nested Loops](#8-nested-loops)
9. [Reading Files Line by Line](#9-reading-files-line-by-line)
10. [Iterating Over Arrays](#10-iterating-over-arrays)
11. [Iterating Over Command Output](#11-iterating-over-command-output)
12. [Infinite Loops](#12-infinite-loops)
13. [Practical Security Examples](#13-practical-security-examples)
14. [Common Mistakes](#14-common-mistakes)
15. [Hands-On Labs](#15-hands-on-labs)
16. [Interview Questions](#16-interview-questions)
17. [Summary](#17-summary)

---

## 1. What are Loops? (ABSOLUTE BEGINNER)

### 1.1 Real-Life Repetition

**ABSOLUTE BEGINNER EXPLANATION**:

Think about repetitive tasks you do in real life:
- Checking each door in a building to see if it's locked
- Washing dishes one by one until the sink is empty
- Reading each page of a book until you finish
- Trying different keys until one opens the lock

**In programming, loops do the same thing**:
```
For each IP address in the network:
    Check if host is alive
    
For each port from 1 to 1000:
    Test if port is open
    
While passwords remain in wordlist:
    Try next password
```

**ANALOGY**: Think of a loop like a factory assembly line:

```
┌─────────────────────────────────────┐
│  ASSEMBLY LINE (Loop)               │
│                                     │
│  Item 1 → Process → Done            │
│  Item 2 → Process → Done            │
│  Item 3 → Process → Done            │
│  ...                                │
│  Item N → Process → Done            │
└─────────────────────────────────────┘
```

Each item goes through the same process, one after another.

### 1.2 Why Use Loops?

**Without loops (BAD)**:
```bash
#!/bin/bash
ping -c 1 192.168.1.1
ping -c 1 192.168.1.2
ping -c 1 192.168.1.3
ping -c 1 192.168.1.4
# ... need 254 lines total!
```

**With loops (GOOD)**:
```bash
#!/bin/bash
for HOST in {1..254}; do
    ping -c 1 192.168.1.$HOST
done
# Just 3 lines for 254 hosts!
```

### 1.3 Types of Loops in Bash

| Loop Type | When to Use | Example |
|-----------|-------------|---------|
| **for** | Known number of iterations | "Scan ports 1-1000" |
| **while** | Continue while condition true | "While file has lines" |
| **until** | Continue until condition true | "Until connection succeeds" |

**Visual Representation**:
```
FOR LOOP:
┌──────┐
│Start │ → Item 1 → Item 2 → Item 3 → Done
└──────┘   (Known list)

WHILE LOOP:
┌──────────┐
│Condition?│─YES→ Execute → Check again
└────┬─────┘           ↑         │
     NO                └─────────┘
     ↓
   Done

UNTIL LOOP:
┌──────────┐
│Condition?│─NO→ Execute → Check again
└────┬─────┘          ↑         │
     YES              └─────────┘
     ↓
   Done
```

---

## 2. For Loops - Basic Syntax

### 2.1 Understanding For Loops

**ABSOLUTE BEGINNER**: A for loop takes a list of items and processes each one, one at a time.

**Basic Syntax**:
```bash
for VARIABLE in LIST
do
    # Commands using $VARIABLE
done
```

**How it works**:
1. Takes first item from LIST
2. Assigns it to VARIABLE
3. Executes commands in the loop
4. Repeats with next item
5. Stops when list is empty

### 2.2 Example: Simple List

```bash
#!/bin/bash
# Print each fruit name

for FRUIT in apple banana orange grape
do
    echo "I like $FRUIT"
done
```

**Output**:
```
I like apple
I like banana
I like orange
I like grape
```

**What Happens**:
```
Iteration 1: FRUIT="apple"   → echo "I like apple"
Iteration 2: FRUIT="banana"  → echo "I like banana"
Iteration 3: FRUIT="orange"  → echo "I like orange"
Iteration 4: FRUIT="grape"   → echo "I like grape"
```

### 2.3 Example: File Operations

```bash
#!/bin/bash
# Create multiple backup files

for NUM in 1 2 3 4 5
do
    echo "Creating backup_$NUM.txt"
    touch "backup_$NUM.txt"
done
```

**Creates**:
```
backup_1.txt
backup_2.txt
backup_3.txt
backup_4.txt
backup_5.txt
```

### 2.4 Example: Processing Multiple Files

```bash
#!/bin/bash
# Count lines in each log file

for LOGFILE in /var/log/*.log
do
    echo "File: $LOGFILE"
    echo "Lines: $(wc -l < "$LOGFILE")"
    echo "---"
done
```

### 2.5 Example: Network Ping Multiple Hosts

```bash
#!/bin/bash
# Ping multiple servers

SERVERS="google.com github.com stackoverflow.com reddit.com"

for SERVER in $SERVERS
do
    echo -n "Pinging $SERVER... "
    if ping -c 1 -W 2 "$SERVER" &>/dev/null
    then
        echo "✓ UP"
    else
        echo "✗ DOWN"
    fi
done
```

**Output**:
```
Pinging google.com... ✓ UP
Pinging github.com... ✓ UP
Pinging stackoverflow.com... ✓ UP
Pinging reddit.com... ✓ UP
```

### 2.6 One-Line For Loop

```bash
# Compact syntax (all on one line)
for i in 1 2 3; do echo "Number $i"; done
```

**BEGINNER TIP**: Use multi-line format for readability. Save one-liners for quick terminal commands.

### 2.7 Example: User Creation

```bash
#!/bin/bash
# Create multiple user accounts (demo)

USERS="alice bob charlie david"

for USERNAME in $USERS
do
    echo "Creating user: $USERNAME"
    # useradd "$USERNAME"  # Uncomment to actually create
    mkdir -p "/home/$USERNAME"
    echo "✓ Home directory created for $USERNAME"
done
```

---

## 3. For Loops with Ranges

### 3.1 Brace Expansion for Number Ranges

**Syntax**:
```bash
{START..END}      # Range from START to END
{START..END..STEP} # Range with custom step
```

**Example 1: Numbers 1-10**
```bash
#!/bin/bash
for NUM in {1..10}
do
    echo "Number: $NUM"
done
```

**Output**:
```
Number: 1
Number: 2
...
Number: 10
```

**Example 2: Even Numbers (Step = 2)**
```bash
#!/bin/bash
# Print even numbers from 0 to 20

for NUM in {0..20..2}
do
    echo "Even number: $NUM"
done
```

**Output**:
```
Even number: 0
Even number: 2
Even number: 4
...
Even number: 20
```

### 3.2 Example: Port Scanner (1-1000)

```bash
#!/bin/bash
# Scan first 1000 ports on localhost

TARGET="127.0.0.1"

echo "Scanning $TARGET ports 1-1000..."

for PORT in {1..1000}
do
    if timeout 1 bash -c "</dev/tcp/$TARGET/$PORT" 2>/dev/null
    then
        echo "✓ Port $PORT is OPEN"
    fi
done

echo "Scan complete!"
```

### 3.3 Example: Countdown Timer

```bash
#!/bin/bash
# Countdown from 10 to 1

echo "Launch sequence initiated..."

for COUNT in {10..1}
do
    echo "$COUNT..."
    sleep 1
done

echo "🚀 Liftoff!"
```

### 3.4 Example: Alphabet Iteration

```bash
#!/bin/bash
# Iterate through alphabet

for LETTER in {a..z}
do
    echo "Letter: $LETTER"
done

# Also works with uppercase
for LETTER in {A..Z}
do
    echo "Uppercase: $LETTER"
done
```

### 3.5 Example: IP Range Scanner

```bash
#!/bin/bash
# Scan IP range 192.168.1.1 to 192.168.1.254

NETWORK="192.168.1"

echo "Scanning network $NETWORK.0/24"

for HOST in {1..254}
do
    IP="$NETWORK.$HOST"
    
    if ping -c 1 -W 1 "$IP" &>/dev/null
    then
        echo "✓ $IP is alive"
    fi
done
```

### 3.6 Seq Command Alternative

**For older Bash versions** that don't support brace expansion:

```bash
#!/bin/bash
# Using seq command

for NUM in $(seq 1 10)
do
    echo "Number: $NUM"
done

# With step
for NUM in $(seq 0 2 20)  # Start, step, end
do
    echo "Even: $NUM"
done
```

---

## 4. C-Style For Loops

### 4.1 Understanding C-Style Loops

**ABSOLUTE BEGINNER**: C-style loops use a counter variable with initialization, condition, and increment - similar to C, Java, JavaScript.

**Syntax**:
```bash
for (( initialization; condition; increment ))
do
    # Commands
done
```

**Example**:
```bash
for (( i=0; i<10; i++ ))
do
    echo "i = $i"
done
```

**Parts Explained**:
```
(( i=0;    i<10;       i++ ))
   ↑       ↑           ↑
   Init    Condition   Increment

1. i=0        → Start with i equals 0
2. i<10       → Continue while i less than 10
3. i++        → Add 1 to i after each iteration
```

### 4.2 Example: Counting 0-9

```bash
#!/bin/bash
# Count from 0 to 9

for (( i=0; i<10; i++ ))
do
    echo "Count: $i"
done
```

**Output**:
```
Count: 0
Count: 1
...
Count: 9
```

### 4.3 Example: Countdown

```bash
#!/bin/bash
# Countdown from 10 to 1

for (( i=10; i>0; i-- ))
do
    echo "$i..."
    sleep 1
done
echo "Done!"
```

### 4.4 Example: Custom Increment

```bash
#!/bin/bash
# Count by 5s

for (( i=0; i<=100; i+=5 ))
do
    echo "i = $i"
done
```

**Output**:
```
i = 0
i = 5
i = 10
...
i = 100
```

### 4.5 Example: Array Indexing

```bash
#!/bin/bash
# Iterate over array using indices

COLORS=("red" "green" "blue" "yellow")

for (( i=0; i<${#COLORS[@]}; i++ ))
do
    echo "Color $i: ${COLORS[$i]}"
done
```

**Output**:
```
Color 0: red
Color 1: green
Color 2: blue
Color 3: yellow
```

### 4.6 Example: Port Scanner with Progress

```bash
#!/bin/bash
# Port scanner with progress indicator

TARGET="192.168.1.100"
START_PORT=1
END_PORT=1000

echo "Scanning $TARGET ports $START_PORT-$END_PORT"

for (( PORT=START_PORT; PORT<=END_PORT; PORT++ ))
do
    # Progress indicator every 100 ports
    if (( PORT % 100 == 0 )); then
        echo "Progress: $PORT/$END_PORT ports scanned..."
    fi
    
    if timeout 0.1 bash -c "</dev/tcp/$TARGET/$PORT" 2>/dev/null
    then
        echo "✓ Port $PORT OPEN"
    fi
done

echo "Scan complete!"
```

### 4.7 Example: Multiple Counters

```bash
#!/bin/bash
# Use multiple variables in C-style loop

for (( i=0, j=10; i<=10; i++, j-- ))
do
    echo "i=$i, j=$j, sum=$((i+j))"
done
```

**Output**:
```
i=0, j=10, sum=10
i=1, j=9, sum=10
i=2, j=8, sum=10
...
```

---

## 5. While Loops

### 5.1 Understanding While Loops

**ABSOLUTE BEGINNER**: A while loop continues executing as long as a condition is TRUE.

**Syntax**:
```bash
while [ condition ]
do
    # Commands
done
```

**Visual Flow**:
```
     ┌──────────────┐
     │  Condition?  │
     └───────┬──────┘
             │
       ┌─────┴─────┐
       ↓           ↓
     TRUE        FALSE
       │           │
       ↓           ↓
   Execute       Exit
   Commands      Loop
       │
       └──→ Check condition again
```

### 5.2 Example: Counter-Based While Loop

```bash
#!/bin/bash
# Count from 1 to 5

COUNT=1

while [ $COUNT -le 5 ]
do
    echo "Count: $COUNT"
    COUNT=$((COUNT + 1))
done

echo "Finished!"
```

**Output**:
```
Count: 1
Count: 2
Count: 3
Count: 4
Count: 5
Finished!
```

### 5.3 Example: User Input Validation

```bash
#!/bin/bash
# Keep asking until valid input

VALID=false

while [ "$VALID" = false ]
do
    echo "Enter 'yes' to continue:"
    read ANSWER
    
    if [ "$ANSWER" = "yes" ]
    then
        VALID=true
        echo "✓ Thank you!"
    else
        echo "✗ Invalid input, try again"
    fi
done
```

### 5.4 Example: File Monitoring

```bash
#!/bin/bash
# Wait for file to appear

TARGET_FILE="completed.txt"

echo "Waiting for $TARGET_FILE to appear..."

while [ ! -f "$TARGET_FILE" ]
do
    echo "Still waiting... ($(date +%T))"
    sleep 5
done

echo "✓ File detected!"
```

### 5.5 Example: Reading File Line by Line

```bash
#!/bin/bash
# Read file line by line

FILE="usernames.txt"

while IFS= read -r LINE
do
    echo "Processing: $LINE"
done < "$FILE"
```

**Explanation**:
- `IFS=` prevents trimming whitespace
- `-r` prevents backslash interpretation
- `< "$FILE"` redirects file to while loop

### 5.6 Example: Retry with Max Attempts

```bash
#!/bin/bash
# Retry connection with maximum attempts

TARGET="192.168.1.100"
MAX_ATTEMPTS=5
ATTEMPT=1

while [ $ATTEMPT -le $MAX_ATTEMPTS ]
do
    echo "Attempt $ATTEMPT of $MAX_ATTEMPTS..."
    
    if ping -c 1 -W 2 "$TARGET" &>/dev/null
    then
        echo "✓ Connection successful!"
        exit 0
    else
        echo "✗ Connection failed"
        ATTEMPT=$((ATTEMPT + 1))
        
        if [ $ATTEMPT -le $MAX_ATTEMPTS ]; then
            echo "Retrying in 3 seconds..."
            sleep 3
        fi
    fi
done

echo "✗ All attempts failed - giving up"
exit 1
```

### 5.7 Example: Menu Loop

```bash
#!/bin/bash
# Menu that loops until user exits

CHOICE=""

while [ "$CHOICE" != "4" ]
do
    echo ""
    echo "Main Menu:"
    echo "1) Option 1"
    echo "2) Option 2"
    echo "3) Option 3"
    echo "4) Exit"
    echo -n "Select: "
    read CHOICE
    
    case $CHOICE in
        1) echo "You selected option 1" ;;
        2) echo "You selected option 2" ;;
        3) echo "You selected option 3" ;;
        4) echo "Goodbye!" ;;
        *) echo "Invalid choice" ;;
    esac
done
```

### 5.8 Example: Process Monitoring

```bash
#!/bin/bash
# Monitor CPU usage

THRESHOLD=80

while true
do
    CPU_USAGE=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}' | cut -d'%' -f1)
    
    echo "[$(date +%T)] CPU: ${CPU_USAGE}%"
    
    if (( $(echo "$CPU_USAGE > $THRESHOLD" | bc -l) ))
    then
        echo "⚠️  WARNING: CPU usage above ${THRESHOLD}%!"
    fi
    
    sleep 5
done
```

---

## 6. Until Loops

### 6.1 Understanding Until Loops

**ABSOLUTE BEGINNER**: An until loop is the opposite of a while loop - it continues UNTIL the condition becomes TRUE.

**Syntax**:
```bash
until [ condition ]
do
    # Commands
done
```

**Comparison**:
```
while [ condition ]:  Continue while TRUE (stop when FALSE)
until [ condition ]:  Continue while FALSE (stop when TRUE)
```

### 6.2 Example: Wait Until File Exists

```bash
#!/bin/bash
# Wait until file appears

FILE="ready.txt"

echo "Waiting for $FILE..."

until [ -f "$FILE" ]
do
    echo "Still waiting..."
    sleep 2
done

echo "✓ File found!"
```

### 6.3 Example: Wait for Network Connection

```bash
#!/bin/bash
# Wait until network is available

TARGET="8.8.8.8"

echo "Waiting for network connection..."

until ping -c 1 -W 2 "$TARGET" &>/dev/null
do
    echo "Network unavailable, retrying..."
    sleep 5
done

echo "✓ Network is up!"
```

### 6.4 Example: Countdown Timer

```bash
#!/bin/bash
# Countdown using until

SECONDS=10

until [ $SECONDS -eq 0 ]
do
    echo "$SECONDS seconds remaining..."
    sleep 1
    SECONDS=$((SECONDS - 1))
done

echo "Time's up!"
```

### 6.5 Example: Wait for Service

```bash
#!/bin/bash
# Wait until service is running

SERVICE="apache2"

echo "Waiting for $SERVICE to start..."

until systemctl is-active --quiet "$SERVICE"
do
    echo "Service not ready yet..."
    sleep 3
done

echo "✓ $SERVICE is running!"
```

### 6.6 While vs Until Comparison

**Same functionality, different syntax**:

```bash
# Using WHILE
while [ ! -f "file.txt" ]
do
    echo "Waiting..."
    sleep 1
done

# Using UNTIL (cleaner for this case)
until [ -f "file.txt" ]
do
    echo "Waiting..."
    sleep 1
done
```

**BEGINNER TIP**: Use `until` when waiting for something to become true. Use `while` for ongoing conditions.

---

## 7. Loop Control: break and continue

### 7.1 Understanding break

**ABSOLUTE BEGINNER**: `break` exits the loop immediately, even if there are more iterations.

**Example: Exit When Found**
```bash
#!/bin/bash
# Search for specific user

USERS="alice bob charlie admin david"

for USER in $USERS
do
    echo "Checking $USER..."
    
    if [ "$USER" = "admin" ]
    then
        echo "✓ Found admin!"
        break  # Stop searching
    fi
done

echo "Search complete"
```

**Output**:
```
Checking alice...
Checking bob...
Checking charlie...
Checking admin...
✓ Found admin!
Search complete
```

**Without break**, it would check david too.

### 7.2 Example: Password Cracking Simulation

```bash
#!/bin/bash
# Try passwords until match found

CORRECT_PASSWORD="secret123"
ATTEMPTS=0
MAX_ATTEMPTS=10

WORDLIST=("password" "12345" "admin" "secret123" "letmein")

for PASSWORD in "${WORDLIST[@]}"
do
    ATTEMPTS=$((ATTEMPTS + 1))
    echo "Attempt $ATTEMPTS: Trying '$PASSWORD'..."
    
    if [ "$PASSWORD" = "$CORRECT_PASSWORD" ]
    then
        echo "✓ Password found: $PASSWORD"
        break  # Stop trying
    fi
    
    if [ $ATTEMPTS -ge $MAX_ATTEMPTS ]
    then
        echo "Max attempts reached"
        break
    fi
done
```

### 7.3 Understanding continue

**ABSOLUTE BEGINNER**: `continue` skips the rest of the current iteration and moves to the next one.

**Example: Skip Certain Items**
```bash
#!/bin/bash
# Process files but skip .log files

for FILE in *
do
    # Skip log files
    if [[ $FILE == *.log ]]
    then
        echo "Skipping log file: $FILE"
        continue  # Skip to next file
    fi
    
    echo "Processing: $FILE"
done
```

### 7.4 Example: Skip Offline Hosts

```bash
#!/bin/bash
# Process only online hosts

for HOST in {1..10}
do
    IP="192.168.1.$HOST"
    
    # Check if host is up
    if ! ping -c 1 -W 1 "$IP" &>/dev/null
    then
        echo "✗ $IP is offline - skipping"
        continue  # Skip to next host
    fi
    
    # Host is online - do more work
    echo "✓ $IP is online"
    echo "  Running nmap scan..."
    # nmap commands here
done
```

### 7.5 Example: Filtering Numbers

```bash
#!/bin/bash
# Print only even numbers

for NUM in {1..20}
do
    # Skip odd numbers
    if [ $((NUM % 2)) -ne 0 ]
    then
        continue
    fi
    
    echo "Even number: $NUM"
done
```

**Output**:
```
Even number: 2
Even number: 4
Even number: 6
...
Even number: 20
```

### 7.6 Break vs Continue Visual

```
WITH BREAK:
Item 1 → Process
Item 2 → Process
Item 3 → BREAK → Exit loop
Item 4 → (never processed)
Item 5 → (never processed)

WITH CONTINUE:
Item 1 → Process
Item 2 → Process
Item 3 → CONTINUE → Skip processing
Item 4 → Process
Item 5 → Process
```

---

## 8. Nested Loops

### 8.1 Understanding Nested Loops

**ABSOLUTE BEGINNER**: A nested loop is a loop inside another loop. The inner loop completes all its iterations for each iteration of the outer loop.

**Visual Example**:
```
Outer loop iteration 1:
    Inner loop: 1, 2, 3, 4, 5
Outer loop iteration 2:
    Inner loop: 1, 2, 3, 4, 5
Outer loop iteration 3:
    Inner loop: 1, 2, 3, 4, 5
```

**Syntax**:
```bash
for OUTER in list1
do
    for INNER in list2
    do
        # Commands using $OUTER and $INNER
    done
done
```

### 8.2 Example: Multiplication Table

```bash
#!/bin/bash
# Print multiplication table

for i in {1..5}
do
    for j in {1..5}
    do
        RESULT=$((i * j))
        printf "%2d " $RESULT
    done
    echo ""  # New line after each row
done
```

**Output**:
```
 1  2  3  4  5
 2  4  6  8 10
 3  6  9 12 15
 4  8 12 16 20
 5 10 15 20 25
```

### 8.3 Example: Network Scan (Multiple Networks)

```bash
#!/bin/bash
# Scan multiple network ranges

NETWORKS="192.168.1 192.168.2 10.0.0"

for NETWORK in $NETWORKS
do
    echo "Scanning network $NETWORK.0/24"
    
    for HOST in {1..254}
    do
        IP="$NETWORK.$HOST"
        
        if ping -c 1 -W 1 "$IP" &>/dev/null
        then
            echo "  ✓ $IP is alive"
        fi
    done
    
    echo "Network $NETWORK complete"
    echo "---"
done
```

### 8.4 Example: Port Scan Multiple Hosts

```bash
#!/bin/bash
# Scan multiple ports on multiple hosts

HOSTS="192.168.1.1 192.168.1.100"
PORTS="22 80 443 3306"

for HOST in $HOSTS
do
    echo "Scanning $HOST..."
    
    for PORT in $PORTS
    do
        if timeout 1 bash -c "</dev/tcp/$HOST/$PORT" 2>/dev/null
        then
            echo "  ✓ Port $PORT OPEN"
        else
            echo "  ✗ Port $PORT closed"
        fi
    done
    
    echo "---"
done
```

### 8.5 Example: File Matrix Processing

```bash
#!/bin/bash
# Process multiple file types in multiple directories

DIRECTORIES="dir1 dir2 dir3"
EXTENSIONS="txt log conf"

for DIR in $DIRECTORIES
do
    echo "Processing directory: $DIR"
    
    for EXT in $EXTENSIONS
    do
        echo "  Looking for *.$EXT files..."
        
        for FILE in "$DIR"/*.$EXT
        do
            if [ -f "$FILE" ]
            then
                echo "    Found: $FILE"
            fi
        done
    done
done
```

### 8.6 Example: Subnet Scanner

```bash
#!/bin/bash
# Scan Class C subnets (192.168.x.x)

for SUBNET in {0..2}  # 192.168.0 to 192.168.2
do
    echo "Scanning 192.168.$SUBNET.0/24"
    
    for HOST in {1..254}
    do
        IP="192.168.$SUBNET.$HOST"
        
        if ping -c 1 -W 1 "$IP" &>/dev/null
        then
            echo "  ✓ $IP is alive"
        fi
    done
done
```

### 8.7 Breaking Out of Nested Loops

```bash
#!/bin/bash
# Break exits only the innermost loop

FOUND=false

for i in {1..5}
do
    echo "Outer loop: $i"
    
    for j in {1..5}
    do
        echo "  Inner loop: $j"
        
        if [ $j -eq 3 ]
        then
            echo "  Breaking inner loop"
            break  # Only exits inner loop
        fi
    done
    
    if [ $i -eq 2 ]
    then
        echo "Breaking outer loop"
        break  # Exits outer loop
    fi
done
```

---

## 9. Reading Files Line by Line

### 9.1 Basic File Reading

**ABSOLUTE BEGINNER**: Reading files line-by-line is one of the most common uses of while loops.

**Syntax**:
```bash
while IFS= read -r LINE
do
    # Process $LINE
done < filename
```

**Parts Explained**:
- `IFS=` - Preserve whitespace
- `read -r` - Don't interpret backslashes
- `LINE` - Variable to store each line
- `< filename` - Redirect file to loop

### 9.2 Example: Process Wordlist

```bash
#!/bin/bash
# Process each word in a wordlist

WORDLIST="passwords.txt"

while IFS= read -r PASSWORD
do
    echo "Trying password: $PASSWORD"
    # Test password here
done < "$WORDLIST"
```

### 9.3 Example: Log File Analysis

```bash
#!/bin/bash
# Count error lines in log file

LOGFILE="/var/log/application.log"
ERROR_COUNT=0

while IFS= read -r LINE
do
    if [[ $LINE == *ERROR* ]]
    then
        ERROR_COUNT=$((ERROR_COUNT + 1))
        echo "Error found: $LINE"
    fi
done < "$LOGFILE"

echo "Total errors: $ERROR_COUNT"
```

### 9.4 Example: Extract IP Addresses

```bash
#!/bin/bash
# Extract unique IP addresses from log

LOGFILE="access.log"

declare -A IP_SEEN

while IFS= read -r LINE
do
    # Extract IP (assumes format: IP - - [date] ...)
    IP=$(echo "$LINE" | awk '{print $1}')
    
    # Store in associative array (ensures uniqueness)
    IP_SEEN["$IP"]=1
done < "$LOGFILE"

echo "Unique IP addresses:"
for IP in "${!IP_SEEN[@]}"
do
    echo "  $IP"
done
```

### 9.5 Example: CSV Processing

```bash
#!/bin/bash
# Process CSV file

CSV_FILE="users.csv"

# Skip header line
FIRST_LINE=true

while IFS=',' read -r USERNAME EMAIL ROLE
do
    # Skip header
    if [ "$FIRST_LINE" = true ]
    then
        FIRST_LINE=false
        continue
    fi
    
    echo "User: $USERNAME"
    echo "  Email: $EMAIL"
    echo "  Role: $ROLE"
    echo "---"
done < "$CSV_FILE"
```

### 9.6 Example: Host List Scanner

```bash
#!/bin/bash
# Scan hosts from file

HOSTFILE="targets.txt"

while IFS= read -r HOST
do
    # Skip empty lines and comments
    if [ -z "$HOST" ] || [[ $HOST == \#* ]]
    then
        continue
    fi
    
    echo "Scanning $HOST..."
    
    if ping -c 1 -W 2 "$HOST" &>/dev/null
    then
        echo "  ✓ $HOST is alive"
        # Run nmap or other scans
    else
        echo "  ✗ $HOST is unreachable"
    fi
done < "$HOSTFILE"
```

### 9.7 Example: Configuration File Parser

```bash
#!/bin/bash
# Parse simple config file (KEY=VALUE format)

CONFIG_FILE="app.conf"

declare -A CONFIG

while IFS='=' read -r KEY VALUE
do
    # Skip comments and empty lines
    [[ $KEY =~ ^[[:space:]]*# ]] && continue
    [[ -z $KEY ]] && continue
    
    # Trim whitespace
    KEY=$(echo "$KEY" | xargs)
    VALUE=$(echo "$VALUE" | xargs)
    
    CONFIG["$KEY"]="$VALUE"
done < "$CONFIG_FILE"

# Use configuration
echo "Host: ${CONFIG[host]}"
echo "Port: ${CONFIG[port]}"
```

---

## 10. Iterating Over Arrays

### 10.1 Basic Array Iteration

**Syntax**:
```bash
for ITEM in "${ARRAY[@]}"
do
    # Process $ITEM
done
```

**Example**:
```bash
#!/bin/bash
# Iterate over array elements

SERVERS=("web1.example.com" "web2.example.com" "db.example.com")

for SERVER in "${SERVERS[@]}"
do
    echo "Connecting to $SERVER..."
    # ssh commands here
done
```

### 10.2 Iterate with Indices

```bash
#!/bin/bash
# Access array with indices

COLORS=("red" "green" "blue" "yellow")

for i in "${!COLORS[@]}"
do
    echo "Index $i: ${COLORS[$i]}"
done
```

**Output**:
```
Index 0: red
Index 1: green
Index 2: blue
Index 3: yellow
```

### 10.3 Example: Port Array Scanner

```bash
#!/bin/bash
# Scan array of ports

TARGET="192.168.1.100"
PORTS=(22 80 443 3306 5432 8080 8443)

echo "Scanning ${#PORTS[@]} ports on $TARGET..."

for PORT in "${PORTS[@]}"
do
    if timeout 1 bash -c "</dev/tcp/$TARGET/$PORT" 2>/dev/null
    then
        echo "✓ Port $PORT OPEN"
    fi
done
```

### 10.4 Example: Associative Array Iteration

```bash
#!/bin/bash
# Iterate over associative array (key-value pairs)

declare -A SERVICES
SERVICES[22]="SSH"
SERVICES[80]="HTTP"
SERVICES[443]="HTTPS"
SERVICES[3306]="MySQL"

TARGET="192.168.1.100"

for PORT in "${!SERVICES[@]}"
do
    SERVICE="${SERVICES[$PORT]}"
    
    if timeout 1 bash -c "</dev/tcp/$TARGET/$PORT" 2>/dev/null
    then
        echo "✓ $SERVICE (port $PORT) is OPEN"
    else
        echo "✗ $SERVICE (port $PORT) is closed"
    fi
done
```

### 10.5 Example: Multi-Dimensional Data

```bash
#!/bin/bash
# Simulate multi-dimensional array with delimiter

USERS=(
    "alice:admin:alice@example.com"
    "bob:user:bob@example.com"
    "charlie:moderator:charlie@example.com"
)

for ENTRY in "${USERS[@]}"
do
    IFS=':' read -r USERNAME ROLE EMAIL <<< "$ENTRY"
    
    echo "Username: $USERNAME"
    echo "  Role: $ROLE"
    echo "  Email: $EMAIL"
    echo "---"
done
```

---

## 11. Iterating Over Command Output

### 11.1 Basic Command Iteration

**Syntax**:
```bash
for ITEM in $(command)
do
    # Process $ITEM
done
```

**Example: Process Running Processes**
```bash
#!/bin/bash
# List all running bash processes

for PID in $(pgrep bash)
do
    echo "Bash process PID: $PID"
    ps -p $PID -o comm=,cmd=
done
```

### 11.2 Example: Process Files from Find

```bash
#!/bin/bash
# Find and process all .log files

for LOGFILE in $(find /var/log -name "*.log" -type f 2>/dev/null)
do
    echo "Processing: $LOGFILE"
    echo "  Size: $(du -h "$LOGFILE" | cut -f1)"
    echo "  Lines: $(wc -l < "$LOGFILE")"
done
```

### 11.3 Example: Network Interfaces

```bash
#!/bin/bash
# List IP addresses of all interfaces

for INTERFACE in $(ip -o link show | awk -F': ' '{print $2}')
do
    echo "Interface: $INTERFACE"
    
    IP=$(ip addr show "$INTERFACE" | grep "inet " | awk '{print $2}')
    
    if [ -n "$IP" ]
    then
        echo "  IP: $IP"
    else
        echo "  No IP assigned"
    fi
done
```

### 11.4 Example: Git Branches

```bash
#!/bin/bash
# Process all git branches

for BRANCH in $(git branch -r | sed 's|origin/||' | grep -v HEAD)
do
    echo "Branch: $BRANCH"
    git log -1 --oneline "origin/$BRANCH"
    echo "---"
done
```

### 11.5 Warning: Word Splitting Issues

**PROBLEM with filenames containing spaces**:
```bash
# BAD - breaks on spaces!
for FILE in $(ls)
do
    echo "$FILE"
done
```

**SOLUTION - use glob patterns**:
```bash
# GOOD - handles spaces correctly
for FILE in *
do
    echo "$FILE"
done
```

---

## 12. Infinite Loops

### 12.1 Creating Infinite Loops

**Syntax Options**:
```bash
# Option 1: while true
while true
do
    # Commands
done

# Option 2: while :
while :
do
    # Commands
done

# Option 3: for (( ; ; ))
for (( ; ; ))
do
    # Commands
done
```

### 12.2 Example: Continuous Monitoring

```bash
#!/bin/bash
# Monitor system load continuously

while true
do
    LOAD=$(uptime | awk -F'load average:' '{print $2}' | awk '{print $1}')
    echo "[$(date +%T)] Load: $LOAD"
    
    sleep 5
done
```

**Exit with**: Ctrl+C

### 12.3 Example: Service Watchdog

```bash
#!/bin/bash
# Restart service if it stops

SERVICE="apache2"

while true
do
    if ! systemctl is-active --quiet "$SERVICE"
    then
        echo "[$(date)] $SERVICE is down - restarting..."
        systemctl restart "$SERVICE"
    else
        echo "[$(date)] $SERVICE is running"
    fi
    
    sleep 60
done
```

### 12.4 Example: Log Tail Simulator

```bash
#!/bin/bash
# Simulate continuous log monitoring

LOGFILE="/var/log/syslog"

while :
do
    clear
    echo "=== Last 20 log entries ==="
    tail -20 "$LOGFILE"
    echo "=== Refreshing in 5 seconds ==="
    sleep 5
done
```

### 12.5 Controlled Infinite Loop

```bash
#!/bin/bash
# Infinite loop with exit condition

COUNTER=0

while true
do
    echo "Iteration: $COUNTER"
    COUNTER=$((COUNTER + 1))
    
    # Exit after 100 iterations
    if [ $COUNTER -ge 100 ]
    then
        echo "Reached limit - exiting"
        break
    fi
    
    sleep 1
done
```

---

## 13. Practical Security Examples

### 13.1 Complete Ping Sweep

```bash
#!/bin/bash
# Professional ping sweep with logging

NETWORK="${1:-192.168.1}"
OUTPUT_FILE="live_hosts_$(date +%Y%m%d_%H%M%S).txt"

echo "Ping Sweep on $NETWORK.0/24"
echo "Output: $OUTPUT_FILE"
echo "====================================="

> "$OUTPUT_FILE"  # Clear file

for HOST in {1..254}
do
    IP="$NETWORK.$HOST"
    
    if ping -c 1 -W 1 "$IP" &>/dev/null
    then
        echo "✓ $IP is alive"
        echo "$IP" >> "$OUTPUT_FILE"
    fi
done

TOTAL=$(wc -l < "$OUTPUT_FILE")
echo ""
echo "Scan complete! Found $TOTAL live hosts"
echo "Results saved to: $OUTPUT_FILE"
```

### 13.2 Port Scanner with Service Detection

```bash
#!/bin/bash
# Advanced port scanner

TARGET="$1"
START="${2:-1}"
END="${3:-1000}"

if [ -z "$TARGET" ]; then
    echo "Usage: $0 <target> [start_port] [end_port]"
    exit 1
fi

declare -A COMMON_PORTS
COMMON_PORTS[22]="SSH"
COMMON_PORTS[80]="HTTP"
COMMON_PORTS[443]="HTTPS"
COMMON_PORTS[3306]="MySQL"
COMMON_PORTS[3389]="RDP"

echo "Scanning $TARGET ports $START-$END"
echo "===================================="

OPEN_PORTS=0

for (( PORT=START; PORT<=END; PORT++ ))
do
    if (( PORT % 100 == 0 )); then
        echo "Progress: $PORT/$END"
    fi
    
    if timeout 0.5 bash -c ">/dev/tcp/$TARGET/$PORT" 2>/dev/null
    then
        SERVICE="${COMMON_PORTS[$PORT]:-Unknown}"
        echo "✓ Port $PORT OPEN - $SERVICE"
        OPEN_PORTS=$((OPEN_PORTS + 1))
    fi
done

echo ""
echo "Scan complete! Found $OPEN_PORTS open ports"
```

### 13.3 Brute Force Simulator

```bash
#!/bin/bash
# Simulated brute force attack (educational only!)

SERVICE="ssh"
TARGET="192.168.1.100"
USERNAME="admin"
WORDLIST="passwords.txt"

if [ ! -f "$WORDLIST" ]; then
    echo "Wordlist not found: $WORDLIST"
    exit 1
fi

ATTEMPT=0

while IFS= read -r PASSWORD
do
    ATTEMPT=$((ATTEMPT + 1))
    echo "Attempt $ATTEMPT: Testing password '$PASSWORD'..."
    
    # Simulated authentication (replace with actual test)
    # Example: sshpass -p "$PASSWORD" ssh "$USERNAME@$TARGET" "exit" 2>/dev/null
    
    sleep 0.5  # Rate limiting
    
done < "$WORDLIST"

echo "Exhausted wordlist - $ATTEMPT attempts made"
```

### 13.4 Directory Enumeration

```bash
#!/bin/bash
# Web directory fuzzer

TARGET_URL="$1"
WORDLIST="${2:-/usr/share/wordlists/dirb/common.txt}"

if [ -z "$TARGET_URL" ]; then
    echo "Usage: $0 <url> [wordlist]"
    exit 1
fi

if [ ! -f "$WORDLIST" ]; then
    echo "Wordlist not found: $WORDLIST"
    exit 1
fi

echo "Fuzzing $TARGET_URL"
echo "==================="

while IFS= read -r DIRECTORY
do
    URL="$TARGET_URL/$DIRECTORY"
    
    RESPONSE=$(curl -s -o /dev/null -w "%{http_code}" --connect-timeout 2 "$URL")
    
    case $RESPONSE in
        200)
            echo "✓ [200] $URL"
            ;;
        301|302)
            echo "↪  [$RESPONSE] $URL (redirect)"
            ;;
        403)
            echo "🔒 [403] $URL (forbidden)"
            ;;
    esac
    
done < "$WORDLIST"
```

### 13.5 Log File Attack Detection

```bash
#!/bin/bash
# Detect potential attacks in log files

LOGFILE="/var/log/auth.log"
SUSPICIOUS_PATTERNS=(
    "Failed password"
    "Invalid user"
    "authentication failure"
    "POSSIBLE BREAK-IN ATTEMPT"
)

echo "Analyzing $LOGFILE for suspicious activity..."
echo "=============================================="

for PATTERN in "${SUSPICIOUS_PATTERNS[@]}"
do
    echo ""
    echo "Searching for: $PATTERN"
    
    COUNT=0
    
    while IFS= read -r LINE
    do
        if [[ $LINE == *"$PATTERN"* ]]
        then
            COUNT=$((COUNT + 1))
            echo "  $LINE"
        fi
    done < "$LOGFILE"
    
    echo "  Total: $COUNT occurrences"
done
```

---

## 14. Common Mistakes

### 14.1 Word Splitting in For Loops

**WRONG**:
```bash
FILES=$(ls *.txt)  # Breaks on filenames with spaces
for FILE in $FILES
do
    echo "$FILE"
done
```

**RIGHT**:
```bash
for FILE in *.txt
do
    echo "$FILE"
done
```

### 14.2 Infinite Loop Without Exit

**WRONG**:
```bash
while true
do
    echo "Running forever with no way out!"
done
# Ctrl+C is the only way to stop
```

**RIGHT**:
```bash
while true
do
    echo "Running..."
    
    # Always provide exit condition
    read -p "Continue? (y/n): " ANSWER
    [ "$ANSWER" != "y" ] && break
done
```

### 14.3 Forgetting to Increment Counter

**WRONG**:
```bash
COUNT=0
while [ $COUNT -lt 10 ]
do
    echo "$COUNT"
    # Forgot to increment - INFINITE LOOP!
done
```

**RIGHT**:
```bash
COUNT=0
while [ $COUNT -lt 10 ]
do
    echo "$COUNT"
    COUNT=$((COUNT + 1))  # Increment!
done
```

### 14.4 Not Quoting Variables in Loops

**WRONG**:
```bash
for FILE in *.txt
do
    cat $FILE  # Breaks if filename has spaces
done
```

**RIGHT**:
```bash
for FILE in *.txt
do
    cat "$FILE"  # Quoted - safe
done
```

### 14.5 Using seq with Brace Expansion

**WRONG**:
```bash
for i in {1..$END}  # Variable expansion doesn't work in braces!
```

**RIGHT**:
```bash
for i in $(seq 1 $END)  # Use seq for variables
# OR
for (( i=1; i<=$END; i++ ))  # C-style loop
```

---

## 15. Hands-On Labs

### Lab 1: Network Scanner

**Objective**: Create a full network scanner that finds live hosts and checks common ports.

**Solution**:
```bash
#!/bin/bash
# Complete network scanner

NETWORK="${1:-192.168.1}"
PORTS=(22 80 443 3306 3389)

echo "Network Scanner - $NETWORK.0/24"
echo "================================"

# Step 1: Find live hosts
echo "Finding live hosts..."
LIVE_HOSTS=()

for HOST in {1..254}
do
    IP="$NETWORK.$HOST"
    
    if ping -c 1 -W 1 "$IP" &>/dev/null
    then
        echo "  ✓ $IP is alive"
        LIVE_HOSTS+=("$IP")
    fi
done

echo ""
echo "Found ${#LIVE_HOSTS[@]} live hosts"
echo ""

# Step 2: Scan ports on live hosts
echo "Scanning ports on live hosts..."

for IP in "${LIVE_HOSTS[@]}"
do
    echo "Scanning $IP..."
    
    for PORT in "${PORTS[@]}"
    do
        if timeout 1 bash -c ">/dev/tcp/$IP/$PORT" 2>/dev/null
        then
            echo "  ✓ Port $PORT OPEN"
        fi
    done
done

echo ""
echo "Scan complete!"
```

### Lab 2: Log Analyzer

**Objective**: Parse Apache access logs and generate statistics.

**Solution**:
```bash
#!/bin/bash
# Apache log analyzer

LOGFILE="${1:-access.log}"

if [ ! -f "$LOGFILE" ]; then
    echo "Log file not found: $LOGFILE"
    exit 1
fi

echo "Analyzing $LOGFILE..."
echo "===================="

# Count by status code
declare -A STATUS_CODES

while IFS= read -r LINE
do
    STATUS=$(echo "$LINE" | awk '{print $9}')
    STATUS_CODES[$STATUS]=$((STATUS_CODES[$STATUS] + 1))
done < "$LOGFILE"

echo ""
echo "Requests by Status Code:"
for CODE in "${!STATUS_CODES[@]}"
do
    echo "  $CODE: ${STATUS_CODES[$CODE]}"
done

# Find most common IPs
echo ""
echo "Top 10 IP Addresses:"
awk '{print $1}' "$LOGFILE" | sort | uniq -c | sort -rn | head -10

# Find most requested URLs
echo ""
echo "Top 10 Requested URLs:"
awk '{print $7}' "$LOGFILE" | sort | uniq -c | sort -rn | head -10
```

### Lab 3: User Management Script

**Objective**: Batch create users from a CSV file.

**Solution**:
```bash
#!/bin/bash
# Batch user creation from CSV

CSV_FILE="${1:-users.csv}"

if [ ! -f "$CSV_FILE" ]; then
    echo "CSV file not found: $CSV_FILE"
    exit 1
fi

if [ "$UID" -ne 0 ]; then
    echo "Must run as root"
    exit 1
fi

FIRST_LINE=true

while IFS=',' read -r USERNAME FULLNAME EMAIL
do
    # Skip header
    if [ "$FIRST_LINE" = true ]; then
        FIRST_LINE=false
        continue
    fi
    
    echo "Creating user: $USERNAME"
    
    # Check if user exists
    if id "$USERNAME" &>/dev/null; then
        echo "  ⚠️  User already exists - skipping"
        continue
    fi
    
    # Create user
    useradd -m -c "$FULLNAME" "$USERNAME"
    
    if [ $? -eq 0 ]; then
        echo "  ✓ User created"
        echo "  Setting temporary password..."
        echo "$USERNAME:ChangeMe123" | chpasswd
        echo "  ✓ Password set (ChangeMe123)"
    else
        echo "  ✗ Failed to create user"
    fi
    
done < "$CSV_FILE"

echo ""
echo "User creation complete!"
```

---

## 16. Interview Questions

**Q1: What's the difference between for, while, and until loops?**

Answer:
- `for`: Iterate over a list of items
- `while`: Continue while condition is TRUE
- `until`: Continue until condition becomes TRUE (opposite of while)

**Q2: How do you read a file line-by-line correctly?**

Answer:
```bash
while IFS= read -r LINE
do
    echo "$LINE"
done < filename
```

**Q3: What does `break` vs `continue` do?**

Answer:
- `break`: Exit loop immediately
- `continue`: Skip to next iteration

**Q4: What's wrong with this?**
```bash
for i in {1..$MAX}
do
    echo $i
done
```

Answer: Brace expansion doesn't work with variables. Use:
```bash
for i in $(seq 1 $MAX)
# OR
for (( i=1; i<=$MAX; i++ ))
```

**Q5: How do you create an infinite loop?**

Answer:
```bash
while true; do ...; done
# OR
while :; do ...; done
# OR
for (( ; ; )); do ...; done
```

---

## 17. Summary

**What You Learned**:
1. **For loops**: Iterate over lists, ranges, and arrays
2. **C-style loops**: Traditional counter-based iteration
3. **While loops**: Repeat while condition is true
4. **Until loops**: Repeat until condition becomes true
5. **Loop control**: break and continue for flow control
6. **Nested loops**: Loops within loops for complex tasks
7. **File processing**: Read files line-by-line safely
8. **Security applications**: Network scanning, brute forcing, log analysis

**Key Takeaways**:
- Loops eliminate code repetition
- Choose the right loop type for your task
- Always quote variables in loops
- Use `break` to exit early, `continue` to skip
- Handle file reading with `while IFS= read -r`

**Next Module**: Functions & Code Organization

**File Stats**: 1,200+ lines, 20+ examples, complete OSCP-ready loop mastery!
