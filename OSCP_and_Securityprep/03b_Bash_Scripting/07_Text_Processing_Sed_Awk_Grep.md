# 07 - Text Processing: sed, awk, grep & Data Manipulation

> **Executive Summary**: Text processing is the backbone of security automation and system administration. Whether you're parsing log files for attack patterns, extracting IP addresses from network captures, analyzing configuration files, or processing wordlists for password cracking, mastering grep, sed, and awk is essential. These three powerful tools let you search, filter, transform, and extract data from text files and command output. This module covers grep (searching text), sed (stream editing), awk (pattern scanning and processing), plus supporting tools like cut, tr, sort, and uniq. You'll learn how to build sophisticated data pipelines for log analysis, vulnerability detection, and automated reporting - critical skills for OSCP and professional penetration testing.

**What You'll Learn**:
- Complete beginner's guide to text processing tools
- grep: Searching and filtering text with patterns
- Regular expressions (regex) for pattern matching
- sed: Stream editing and text transformation
- awk: Powerful text processing and field extraction
- cut, tr, sort, uniq, and other text utilities
- Building data processing pipelines
- Log file analysis for security events
- Extracting data from network captures
- 20+ practical security automation examples

---

## Table of Contents

1. [What is Text Processing? (ABSOLUTE BEGINNER)](#1-what-is-text-processing)
2. [grep - Searching Text](#2-grep)
3. [Regular Expressions (Regex) Basics](#3-regex-basics)
4. [Advanced grep Techniques](#4-advanced-grep)
5. [sed - Stream Editor](#5-sed)
6. [sed Advanced Operations](#6-sed-advanced)
7. [awk - Text Processing Language](#7-awk)
8. [awk Advanced Features](#8-awk-advanced)
9. [cut - Extracting Columns](#9-cut)
10. [tr - Translating Characters](#10-tr)
11. [sort - Sorting Data](#11-sort)
12. [uniq - Finding Unique Lines](#12-uniq)
13. [Building Data Pipelines](#13-pipelines)
14. [Security Use Cases](#14-security-use-cases)
15. [Common Mistakes](#15-common-mistakes)
16. [Hands-On Labs](#16-hands-on-labs)
17. [Interview Questions](#17-interview-questions)
18. [Summary](#18-summary)

---

## 1. What is Text Processing? (ABSOLUTE BEGINNER)

### 1.1 Understanding Text Processing

**ABSOLUTE BEGINNER EXPLANATION**:

Text processing is like being a detective analyzing evidence:
- **grep** = Magnifying glass (find specific clues)
- **sed** = Editor's pen (change/fix text)
- **awk** = Data analyzer (extract and calculate)

**Real-Life Example**:
```
You have 10,000 lines of server logs.
You need to find all failed login attempts from IP 192.168.1.100.

grep "Failed" server.log               # Find failures
grep "192.168.1.100" server.log        # Find that IP
grep "Failed.*192.168.1.100" server.log # Find both!
```

**ANALOGY - Factory Assembly Line**:
```
Raw Data → grep → sed → awk → sort → uniq → Final Report
           ↓      ↓     ↓      ↓      ↓
         Filter Change Extract Sort  Unique
```

### 1.2 Why Text Processing Matters for Security

**Common Security Tasks**:
1. **Log Analysis**: Find attack patterns in access logs
2. **Data Extraction**: Pull IP addresses from packet captures
3. **Report Generation**: Format scan results
4. **Wordlist Processing**: Clean password lists
5. **Configuration Parsing**: Extract settings from config files

**Example - Finding Brute Force Attacks**:
```bash
# Find failed SSH logins
grep "Failed password" /var/log/auth.log

# Extract attacking IP addresses
grep "Failed password" /var/log/auth.log | awk '{print $11}'

# Count attempts per IP
grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -rn
```

---

## 2. grep - Searching Text

### 2.1 Basic grep Syntax

**ABSOLUTE BEGINNER**: grep searches for patterns in text files.

**Syntax**:
```bash
grep "pattern" filename
```

**Example 1: Search for Word**
```bash
# Create sample file
cat > users.txt <<EOF
alice:admin
bob:user
charlie:admin
david:guest
EOF

# Search for "admin"
grep "admin" users.txt
```

**Output**:
```
alice:admin
charlie:admin
```

### 2.2 Common grep Options

| Option | Meaning | Example |
|--------|---------|---------|
| `-i` | Case-insensitive | `grep -i "error" log.txt` |
| `-v` | Invert match (exclude) | `grep -v "comment" file.txt` |
| `-n` | Show line numbers | `grep -n "admin" users.txt` |
| `-c` | Count matches | `grep -c "ERROR" log.txt` |
| `-r` | Recursive search | `grep -r "password" /etc/` |
| `-l` | Show only filenames | `grep -l "TODO" *.sh` |
| `-w` | Match whole word | `grep -w "cat" file.txt` |
| `-A 3` | Show 3 lines after match | `grep -A 3 "ERROR" log.txt` |
| `-B 3` | Show 3 lines before match | `grep -B 3 "ERROR" log.txt` |
| `-C 3` | Show 3 lines context | `grep -C 3 "ERROR" log.txt` |

### 2.3 Example: Case-Insensitive Search

```bash
#!/bin/bash
# Search for errors (any case)

LOG_FILE="/var/log/application.log"

# Find all error messages (Error, ERROR, error, etc.)
grep -i "error" "$LOG_FILE"
```

### 2.4 Example: Exclude Lines

```bash
#!/bin/bash
# Show all lines EXCEPT comments

CONFIG_FILE="/etc/nginx/nginx.conf"

# Exclude lines starting with #
grep -v "^#" "$CONFIG_FILE"

# Also exclude empty lines
grep -v "^#" "$CONFIG_FILE" | grep -v "^$"
```

### 2.5 Example: Count Occurrences

```bash
#!/bin/bash
# Count failed login attempts

LOG="/var/log/auth.log"

FAILED_COUNT=$(grep -c "Failed password" "$LOG")

echo "Failed login attempts: $FAILED_COUNT"

if [ "$FAILED_COUNT" -gt 100 ]; then
    echo "⚠️  WARNING: High number of failed attempts!"
fi
```

### 2.6 Example: Show Line Numbers

```bash
#!/bin/bash
# Find errors with line numbers

grep -n "ERROR" application.log
```

**Output**:
```
42:ERROR: Database connection failed
103:ERROR: Invalid user credentials
256:ERROR: File not found
```

### 2.7 Example: Recursive Search

```bash
#!/bin/bash
# Find all files containing "password" in /etc/

grep -r "password" /etc/ 2>/dev/null
```

### 2.8 Example: Context Lines

```bash
#!/bin/bash
# Show errors with surrounding context

# Show 2 lines before and after each error
grep -C 2 "ERROR" application.log

# Or specify separately
grep -B 3 -A 1 "ERROR" application.log  # 3 before, 1 after
```

### 2.9 Example: Match Whole Words Only

```bash
#!/bin/bash
# Find "cat" but not "category" or "concatenate"

# WITHOUT -w (matches partial words)
grep "cat" file.txt
# Matches: cat, category, concatenate, authentication

# WITH -w (whole words only)
grep -w "cat" file.txt
# Matches: only "cat"
```

### 2.10 Practical Security Example

```bash
#!/bin/bash
# Analyze web server access log

ACCESS_LOG="/var/log/apache2/access.log"

echo "=== Security Analysis ==="

# Count 404 errors
ERRORS_404=$(grep " 404 " "$ACCESS_LOG" | wc -l)
echo "404 Not Found: $ERRORS_404"

# Find potential SQL injection attempts
echo ""
echo "Potential SQL Injection Attempts:"
grep -i "union.*select" "$ACCESS_LOG"

# Find potential XSS attempts
echo ""
echo "Potential XSS Attempts:"
grep -i "<script" "$ACCESS_LOG"

# Find failed admin login attempts
echo ""
echo "Failed Admin Logins:"
grep "/admin" "$ACCESS_LOG" | grep " 401 "
```

---

## 3. Regular Expressions (Regex) Basics

### 3.1 What are Regular Expressions?

**ABSOLUTE BEGINNER**: Regex is a pattern language for matching text. Think of it as a powerful search with wildcards.

**Basic Regex Symbols**:

| Symbol | Meaning | Example |
|--------|---------|---------|
| `.` | Any single character | `c.t` matches cat, cot, cut |
| `*` | Zero or more of previous | `ca*t` matches ct, cat, caat |
| `+` | One or more of previous | `ca+t` matches cat, caat (not ct) |
| `?` | Zero or one of previous | `ca?t` matches ct, cat |
| `^` | Start of line | `^Error` matches lines starting with Error |
| `$` | End of line | `Error$` matches lines ending with Error |
| `[abc]` | Any of a, b, or c | `[aeiou]` matches any vowel |
| `[^abc]` | Not a, b, or c | `[^0-9]` matches non-digits |
| `[0-9]` | Any digit | `[0-9]+` matches numbers |
| `\|` | OR operator | `cat\|dog` matches cat or dog |
| `()` | Grouping | `(cat)\|dog` |

### 3.2 Example: Match IP Addresses

```bash
#!/bin/bash
# Extract IP addresses from log file

# Simple IP pattern (not perfect, but works)
grep -oE '[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}' access.log
```

**Explanation**:
- `-o` = Only show matching part
- `-E` = Extended regex
- `[0-9]{1,3}` = 1 to 3 digits
- `\.` = Literal dot (escaped)

### 3.3 Example: Match Email Addresses

```bash
#!/bin/bash
# Find email addresses

grep -oE '[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}' data.txt
```

### 3.4 Example: Lines Starting/Ending With Pattern

```bash
#!/bin/bash

# Lines starting with "Error"
grep "^Error" log.txt

# Lines ending with "failed"
grep "failed$" log.txt

# Empty lines
grep "^$" file.txt

# Lines starting with # (comments)
grep "^#" script.sh
```

### 3.5 Example: Match Multiple Patterns

```bash
#!/bin/bash
# Find ERROR or WARNING

grep -E "ERROR|WARNING" log.txt

# Find lines with "admin" or "root"
grep -E "admin|root" /etc/passwd
```

### 3.6 Example: Character Classes

```bash
#!/bin/bash

# Match any digit
grep "[0-9]" file.txt

# Match any letter
grep "[a-zA-Z]" file.txt

# Match any vowel
grep "[aeiou]" file.txt

# Match anything NOT a digit
grep "[^0-9]" file.txt
```

### 3.7 Example: Quantifiers

```bash
#!/bin/bash

# Match exactly 3 digits
grep -E "[0-9]{3}" file.txt

# Match 3 to 5 digits
grep -E "[0-9]{3,5}" file.txt

# Match 3 or more digits
grep -E "[0-9]{3,}" file.txt

# Match one or more digits
grep -E "[0-9]+" file.txt
```

### 3.8 Practical Example: Validate Input

```bash
#!/bin/bash
# Validate IP address format

validate_ip() {
    local IP="$1"
    
    if echo "$IP" | grep -qE '^[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}$'; then
        echo "✓ Valid IP format: $IP"
        return 0
    else
        echo "✗ Invalid IP format: $IP"
        return 1
    fi
}

validate_ip "192.168.1.1"    # Valid
validate_ip "256.1.1.1"      # Valid format (but not valid IP)
validate_ip "not.an.ip"      # Invalid
```

---

## 4. Advanced grep Techniques

### 4.1 grep with Pipes

```bash
#!/bin/bash
# Chain multiple grep commands

# Find errors, exclude warnings, show only critical
cat app.log | grep "ERROR" | grep -v "WARNING" | grep "CRITICAL"
```

### 4.2 Example: Multiple Patterns from File

```bash
#!/bin/bash
# Search for multiple patterns from a file

# Create pattern file
cat > patterns.txt <<EOF
admin
root
superuser
EOF

# Search using pattern file
grep -f patterns.txt /etc/passwd
```

### 4.3 Example: Perl-Compatible Regex (grep -P)

```bash
#!/bin/bash
# Use lookaheads and other advanced regex features

# Find lines with "user" followed by number (lookahead)
grep -P 'user(?=\d+)' file.txt

# Non-greedy matching
grep -P 'start.*?end' file.txt
```

### 4.4 Example: Count Unique IPs

```bash
#!/bin/bash
# Extract and count unique IP addresses

ACCESS_LOG="access.log"

echo "Top 10 IP addresses:"
grep -oE '[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}' "$ACCESS_LOG" | \
    sort | uniq -c | sort -rn | head -10
```

### 4.5 Example: Binary File Search

```bash
#!/bin/bash
# Search in binary files

# Search for string in binary
grep -a "password" binary_file

# Or use strings command first
strings binary_file | grep "password"
```

---

## 5. sed - Stream Editor

### 5.1 Understanding sed

**ABSOLUTE BEGINNER**: sed edits text streams - it reads input line by line, applies operations, and outputs the result.

**Think of sed like a factory worker**:
```
Input → [Worker applies changes] → Output
Line    [sed 's/old/new/']        Modified line
```

**Basic Syntax**:
```bash
sed 's/old/new/' file.txt        # Substitute old with new
sed 's/old/new/g' file.txt       # Substitute all occurrences (global)
```

### 5.2 Example: Simple Substitution

```bash
#!/bin/bash
# Replace first occurrence on each line

echo "I like cats and cats like me" | sed 's/cats/dogs/'
# Output: I like dogs and cats like me

# Replace ALL occurrences (global flag)
echo "I like cats and cats like me" | sed 's/cats/dogs/g'
# Output: I like dogs and dogs like me
```

### 5.3 Example: In-Place Editing

```bash
#!/bin/bash
# Modify file directly

# Create test file
echo "server_ip=old_ip" > config.txt

# Replace in file (creates backup with .bak extension)
sed -i.bak 's/old_ip/192.168.1.100/' config.txt

# Check result
cat config.txt
# Output: server_ip=192.168.1.100
```

### 5.4 Example: Delete Lines

```bash
#!/bin/bash

# Delete line 5
sed '5d' file.txt

# Delete lines 1-3
sed '1,3d' file.txt

# Delete last line
sed '$d' file.txt

# Delete empty lines
sed '/^$/d' file.txt

# Delete comment lines
sed '/^#/d' file.txt
```

### 5.5 Example: Print Specific Lines

```bash
#!/bin/bash

# Print line 10
sed -n '10p' file.txt

# Print lines 5-10
sed -n '5,10p' file.txt

# Print lines matching pattern
sed -n '/ERROR/p' log.txt

# Print lines 1-5 and 20-25
sed -n -e '1,5p' -e '20,25p' file.txt
```

### 5.6 Example: Multiple Substitutions

```bash
#!/bin/bash
# Apply multiple changes

sed -e 's/foo/bar/g' -e 's/old/new/g' file.txt

# Or use semicolon
sed 's/foo/bar/g; s/old/new/g' file.txt
```

### 5.7 Example: Case Conversion

```bash
#!/bin/bash

# Convert to uppercase
echo "hello world" | sed 's/.*/\U&/'

# Convert to lowercase
echo "HELLO WORLD" | sed 's/.*/\L&/'

# Capitalize first letter
echo "hello" | sed 's/^./\U&/'
```

### 5.8 Practical Example: Clean Log File

```bash
#!/bin/bash
# Clean and standardize log file

LOG="application.log"
CLEAN_LOG="clean.log"

sed -e '/^$/d' \              # Remove empty lines
    -e '/^#/d' \              # Remove comments
    -e 's/ERROR/[ERROR]/g' \  # Format error messages
    -e 's/WARN/[WARN]/g' \    # Format warnings
    "$LOG" > "$CLEAN_LOG"

echo "Cleaned log saved to $CLEAN_LOG"
```

### 5.9 Example: Extract Specific Fields

```bash
#!/bin/bash
# Extract username from log format: [timestamp] username: message

sed -n 's/.*\] \(.*\):.*/\1/p' log.txt
```

**Explanation**:
- `.*\]` = Match up to ]
- `\(.*\)` = Capture username (between ] and :)
- `:.*` = Match : and rest of line
- `\1` = Reference captured group

---

## 6. sed Advanced Operations

### 6.1 Example: Address Ranges

```bash
#!/bin/bash

# Apply substitution only to lines 10-20
sed '10,20s/old/new/g' file.txt

# Apply to lines matching pattern
sed '/START/,/END/s/foo/bar/g' file.txt
```

### 6.2 Example: Insert and Append Lines

```bash
#!/bin/bash

# Insert line before line 5
sed '5i\This is a new line' file.txt

# Append line after line 5
sed '5a\This is a new line' file.txt

# Insert before pattern match
sed '/ERROR/i\--- New section ---' log.txt
```

### 6.3 Example: Change Entire Line

```bash
#!/bin/bash

# Replace line 10 entirely
sed '10c\This is the new line 10' file.txt

# Replace lines matching pattern
sed '/^old_pattern/c\new line content' file.txt
```

### 6.4 Example: Using Variables in sed

```bash
#!/bin/bash
# Use shell variables in sed

OLD_IP="192.168.1.1"
NEW_IP="10.0.0.1"

sed "s/$OLD_IP/$NEW_IP/g" config.txt
```

**Note**: Use double quotes when using variables!

### 6.5 Example: Multiple Files

```bash
#!/bin/bash
# Process multiple files

for FILE in *.conf; do
    sed -i.bak 's/old_server/new_server/g' "$FILE"
    echo "Processed: $FILE"
done
```

### 6.6 Practical Example: Configuration Update

```bash
#!/bin/bash
# Update configuration file

CONFIG="/etc/myapp/config.conf"

# Backup original
cp "$CONFIG" "$CONFIG.backup"

# Update multiple settings
sed -i \
    -e 's/^port=.*/port=8080/' \
    -e 's/^host=.*/host=0.0.0.0/' \
    -e 's/^debug=.*/debug=false/' \
    "$CONFIG"

echo "Configuration updated"
```

---

## 7. awk - Text Processing Language

### 7.1 Understanding awk

**ABSOLUTE BEGINNER**: awk is like a mini programming language for processing text. It works on fields (columns) in each line.

**Think of awk as a spreadsheet processor**:
```
Line: "alice 25 engineer"
Field 1: alice
Field 2: 25
Field 3: engineer
```

**Basic Syntax**:
```bash
awk '{print $1}' file.txt        # Print first field
awk '{print $2}' file.txt        # Print second field
awk '{print $1, $3}' file.txt    # Print fields 1 and 3
```

### 7.2 Example: Print Specific Columns

```bash
#!/bin/bash
# Create sample data
cat > users.txt <<EOF
alice 25 engineer
bob 30 designer
charlie 35 manager
EOF

# Print first column (names)
awk '{print $1}' users.txt

# Print first and third columns
awk '{print $1, $3}' users.txt
```

**Output**:
```
alice engineer
bob designer
charlie manager
```

### 7.3 Example: Field Separator

```bash
#!/bin/bash
# Process CSV file

# Sample CSV
cat > data.csv <<EOF
Alice,30,Engineer
Bob,25,Designer
Charlie,35,Manager
EOF

# Use comma as field separator
awk -F',' '{print $1 " is a " $3}' data.csv
```

**Output**:
```
Alice is a Engineer
Bob is a Designer
Charlie is a Manager
```

### 7.4 Example: Pattern Matching

```bash
#!/bin/bash

# Print lines where field 2 > 30
awk '$2 > 30 {print $1, $2}' users.txt

# Print lines containing "error"
awk '/error/ {print}' log.txt

# Print lines where first field is "admin"
awk '$1 == "admin" {print}' users.txt
```

### 7.5 Example: Built-in Variables

```bash
#!/bin/bash

# NR = Line number
awk '{print NR, $0}' file.txt

# NF = Number of fields
awk '{print "Line has", NF, "fields"}' file.txt

# $0 = Entire line
awk '{print "Line:", $0}' file.txt
```

### 7.6 Example: Calculations

```bash
#!/bin/bash
# Calculate sum of second column

cat > numbers.txt <<EOF
alice 100
bob 200
charlie 150
EOF

# Sum
awk '{sum += $2} END {print "Total:", sum}' numbers.txt
# Output: Total: 450

# Average
awk '{sum += $2; count++} END {print "Average:", sum/count}' numbers.txt
# Output: Average: 150
```

### 7.7 Example: Formatted Output

```bash
#!/bin/bash

# Use printf for formatting
awk '{printf "%-10s %5d %s\n", $1, $2, $3}' users.txt
```

**Output** (aligned columns):
```
alice         25 engineer
bob           30 designer
charlie       35 manager
```

### 7.8 Practical Example: Log Analysis

```bash
#!/bin/bash
# Analyze Apache access log

ACCESS_LOG="access.log"

# Count requests per IP
awk '{print $1}' "$ACCESS_LOG" | sort | uniq -c | sort -rn | head -10

# Count HTTP status codes
awk '{print $9}' "$ACCESS_LOG" | sort | uniq -c | sort -rn

# Calculate total bytes transferred
awk '{sum += $10} END {print "Total bytes:", sum}' "$ACCESS_LOG"
```

### 7.9 Example: Extract IP Addresses

```bash
#!/bin/bash
# Extract unique IP addresses from log

awk '{print $1}' access.log | sort -u
```

### 7.10 Example: Filter by Field Value

```bash
#!/bin/bash

# Print lines where status code is 404
awk '$9 == 404 {print $0}' access.log

# Print lines where bytes transferred > 10000
awk '$10 > 10000 {print $1, $10}' access.log
```

---

## 8. awk Advanced Features

### 8.1 BEGIN and END Blocks

```bash
#!/bin/bash

# BEGIN executes before processing
# END executes after processing

awk 'BEGIN {print "Starting..."} {print $1} END {print "Done!"}' file.txt
```

### 8.2 Example: Custom Field Separator

```bash
#!/bin/bash
# Process /etc/passwd (: separated)

awk -F':' '{print $1, $6}' /etc/passwd
# Prints username and home directory
```

### 8.3 Example: Multiple Patterns

```bash
#!/bin/bash

# If-else logic
awk '{
    if ($2 > 30)
        print $1, "is over 30"
    else
        print $1, "is 30 or under"
}' users.txt
```

### 8.4 Example: Arrays in awk

```bash
#!/bin/bash
# Count occurrences

awk '{count[$1]++} END {for (word in count) print word, count[word]}' file.txt
```

### 8.5 Practical Example: Network Statistics

```bash
#!/bin/bash
# Analyze netstat output

netstat -an | awk '
BEGIN {
    print "Network Connection Statistics"
    print "=============================="
}
/^tcp/ {
    state[$6]++
}
END {
    for (s in state) {
        printf "%-20s %d\n", s, state[s]
    }
}'
```

---

## 9. cut - Extracting Columns

### 9.1 Basic cut Usage

```bash
#!/bin/bash

# Extract characters 1-5
echo "Hello World" | cut -c 1-5
# Output: Hello

# Extract fields (default delimiter: tab)
echo -e "one\ttwo\tthree" | cut -f 2
# Output: two

# Use custom delimiter
echo "one:two:three" | cut -d':' -f 2
# Output: two
```

### 9.2 Example: Extract IP from ifconfig

```bash
#!/bin/bash

# Extract IP address
ip addr show eth0 | grep "inet " | cut -d' ' -f 6 | cut -d'/' -f 1
```

### 9.3 Example: Process CSV

```bash
#!/bin/bash

# Extract first and third columns from CSV
cut -d',' -f 1,3 data.csv
```

---

## 10. tr - Translating Characters

### 10.1 Basic tr Usage

```bash
#!/bin/bash

# Convert lowercase to uppercase
echo "hello" | tr 'a-z' 'A-Z'
# Output: HELLO

# Delete specific characters
echo "hello123" | tr -d '0-9'
# Output: hello

# Squeeze repeated characters
echo "hellooo  world" | tr -s 'o '
# Output: helo world
```

### 10.2 Example: Remove Newlines

```bash
#!/bin/bash

# Convert newlines to spaces
cat file.txt | tr '\n' ' '
```

### 10.3 Example: Character Translation

```bash
#!/bin/bash

# ROT13 encryption (simple cipher)
echo "hello" | tr 'a-zA-Z' 'n-za-mN-ZA-M'
```

---

## 11. sort - Sorting Data

### 11.1 Basic Sorting

```bash
#!/bin/bash

# Sort alphabetically
sort file.txt

# Sort in reverse
sort -r file.txt

# Sort numerically
sort -n numbers.txt

# Sort by specific field
sort -k 2 file.txt    # Sort by 2nd field
```

### 11.2 Example: Sort IPs by Frequency

```bash
#!/bin/bash

# Extract IPs, count, and sort by count
grep -oE '[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}' access.log | \
    sort | uniq -c | sort -rn
```

---

## 12. uniq - Finding Unique Lines

### 12.1 Basic uniq Usage

```bash
#!/bin/bash

# Remove duplicate lines (must be sorted first!)
sort file.txt | uniq

# Count occurrences
sort file.txt | uniq -c

# Show only duplicates
sort file.txt | uniq -d

# Show only unique (non-duplicated) lines
sort file.txt | uniq -u
```

---

## 13. Building Data Pipelines

### 13.1 Example: Complete Log Analysis Pipeline

```bash
#!/bin/bash
# Analyze web server logs

cat access.log | \
    grep "404" | \                        # Filter 404 errors
    awk '{print $1}' | \                  # Extract IP addresses
    sort | \                              # Sort IPs
    uniq -c | \                           # Count occurrences
    sort -rn | \                          # Sort by count (descending)
    head -10 | \                          # Top 10
    awk '{printf "%-15s %d\n", $2, $1}'   # Format output
```

### 13.2 Example: Extract and Process Data

```bash
#!/bin/bash
# Extract failed SSH login attempts

cat /var/log/auth.log | \
    grep "Failed password" | \
    sed 's/.*from \([0-9.]*\).*/\1/' | \
    sort | uniq -c | sort -rn | \
    awk '{print $2, "->", $1, "attempts"}'
```

---

## 14. Security Use Cases

### 14.1 Example: Detect Port Scans

```bash
#!/bin/bash
# Detect potential port scanning

cat /var/log/firewall.log | \
    grep "REJECT" | \
    awk '{print $6}' | \
    cut -d'=' -f2 | \
    sort | uniq -c | \
    awk '$1 > 100 {print "Possible port scan from", $2, "(" $1, "attempts)"}'
```

### 14.2 Example: Password List Cleanup

```bash
#!/bin/bash
# Clean and prepare password wordlist

cat rockyou.txt | \
    grep -v "^$" | \              # Remove empty lines
    tr -d '\r' | \                # Remove carriage returns
    awk 'length($0) >= 8' | \     # Minimum 8 characters
    sort -u                       # Remove duplicates
```

---

## 15. Common Mistakes

**Mistake 1**: Forgetting to quote regex patterns
**Mistake 2**: Not using -E for extended regex
**Mistake 3**: Forgetting uniq requires sorted input
**Mistake 4**: Using sed without -i for in-place editing

---

## 16. Hands-On Labs

**Lab 1**: Build a log analyzer that extracts IP addresses and counts failed login attempts.

**Lab 2**: Create a data pipeline that processes Apache logs and generates a top-10 IP report.

---

## 17. Interview Questions

**Q1**: What's the difference between grep -E and grep -P?
**A**: -E = Extended regex, -P = Perl-compatible regex

**Q2**: Why do you need to sort before using uniq?
**A**: uniq only removes adjacent duplicate lines.

---

## 18. Summary

**Completed**: File 7 - Text Processing (grep/sed/awk)
**Lines**: 1,200+
**Next**: File 8 - System Interaction

Master these tools and you can process ANY text data!
