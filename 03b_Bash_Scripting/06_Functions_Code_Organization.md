# 06 - Functions & Code Organization: Writing Modular, Maintainable Scripts

> **Executive Summary**: Functions are the building blocks of professional scripts. They let you break complex tasks into smaller, reusable pieces - just like LEGO blocks that you can combine in different ways. Instead of writing the same code multiple times, you write a function once and call it whenever needed. This module covers function basics, parameters, return values, variable scope (local vs global), function libraries, error handling within functions, and code organization patterns. You'll learn how to structure large scripts like a professional developer, making your code easier to read, debug, maintain, and reuse - essential for building sophisticated OSCP automation tools.

**What You'll Learn**:
- Complete beginner's guide to functions
- Defining and calling functions
- Function parameters and arguments
- Return values and exit codes
- Variable scope (local vs global)
- Function libraries and source files
- Error handling in functions
- Code organization best practices
- 20+ practical modular script examples
- Real-world penetration testing frameworks

---

## Table of Contents

1. [What are Functions? (ABSOLUTE BEGINNER)](#1-what-are-functions)
2. [Creating Your First Function](#2-creating-your-first-function)
3. [Function Parameters](#3-function-parameters)
4. [Return Values and Exit Codes](#4-return-values-and-exit-codes)
5. [Variable Scope: Local vs Global](#5-variable-scope)
6. [Function Libraries](#6-function-libraries)
7. [Error Handling in Functions](#7-error-handling)
8. [Best Practices & Naming Conventions](#8-best-practices)
9. [Advanced Function Techniques](#9-advanced-techniques)
10. [Organizing Large Scripts](#10-organizing-large-scripts)
11. [Real-World Security Frameworks](#11-security-frameworks)
12. [Recursive Functions](#12-recursive-functions)
13. [Function Testing & Debugging](#13-testing-debugging)
14. [Common Mistakes](#14-common-mistakes)
15. [Hands-On Labs](#15-hands-on-labs)
16. [Interview Questions](#16-interview-questions)
17. [Summary](#17-summary)

---

## 1. What are Functions? (ABSOLUTE BEGINNER)

### 1.1 Understanding Functions

**ABSOLUTE BEGINNER EXPLANATION**:

Think of functions like recipes in a cookbook:
- **Recipe name**: "Make Pancakes"
- **Ingredients**: Flour, eggs, milk (parameters)
- **Instructions**: Mix, cook, flip (function body)
- **Result**: Delicious pancakes (return value)

Instead of writing out the full pancake instructions every time, you just say "Make Pancakes" and follow the recipe.

**In programming**:
```
Function "check_port":
  Input: IP address, port number
  Instructions: Test if port is open
  Output: "OPEN" or "CLOSED"
```

**Analogy - Vending Machine**:
```
┌──────────────────────────┐
│  FUNCTION: GetSnack()    │
│                          │
│  Input:  Money, Choice   │
│  ↓                       │
│  Process: Check stock    │
│           Vend item      │
│  ↓                       │
│  Output: Snack           │
└──────────────────────────┘
```

### 1.2 Why Use Functions?

**WITHOUT Functions (BAD)**:
```bash
#!/bin/bash
# Repetitive code - hard to maintain

echo "Scanning 192.168.1.1 port 80..."
timeout 1 bash -c "</dev/tcp/192.168.1.1/80" 2>/dev/null
if [ $? -eq 0 ]; then echo "OPEN"; else echo "CLOSED"; fi

echo "Scanning 192.168.1.1 port 443..."
timeout 1 bash -c "</dev/tcp/192.168.1.1/443" 2>/dev/null
if [ $? -eq 0 ]; then echo "OPEN"; else echo "CLOSED"; fi

echo "Scanning 192.168.1.1 port 22..."
timeout 1 bash -c "</dev/tcp/192.168.1.1/22" 2>/dev/null
if [ $? -eq 0 ]; then echo "OPEN"; else echo "CLOSED"; fi
# ... 20+ more copies of similar code!
```

**WITH Functions (GOOD)**:
```bash
#!/bin/bash
# Clean, reusable code

check_port() {
    local IP="$1"
    local PORT="$2"
    
    echo "Scanning $IP port $PORT..."
    if timeout 1 bash -c "</dev/tcp/$IP/$PORT" 2>/dev/null; then
        echo "OPEN"
    else
        echo "CLOSED"
    fi
}

# Use function multiple times
check_port "192.168.1.1" "80"
check_port "192.168.1.1" "443"
check_port "192.168.1.1" "22"
# Easy to call as many times as needed!
```

### 1.3 Benefits of Functions

| Benefit | Description |
|---------|-------------|
| **Reusability** | Write once, use many times |
| **Readability** | Code is self-documenting (function names explain purpose) |
| **Maintainability** | Fix bugs in one place, not 20 places |
| **Testing** | Test functions independently |
| **Organization** | Break complex scripts into logical pieces |
| **Abstraction** | Hide complexity behind simple interface |

---

## 2. Creating Your First Function

### 2.1 Function Syntax

**Two ways to define functions**:

```bash
# Method 1: Using 'function' keyword
function my_function {
    # Commands
}

# Method 2: Function name with () (preferred)
my_function() {
    # Commands
}
```

**BEGINNER TIP**: Method 2 (with `()`) is more portable and commonly used.

### 2.2 Calling Functions

```bash
# Define function
greet() {
    echo "Hello, World!"
}

# Call function
greet
```

**Important**: 
- Define function BEFORE calling it
- No parentheses when calling: `greet` not `greet()`

### 2.3 Example: Simple Greeting Function

```bash
#!/bin/bash
# Simple function example

# Define function
greet() {
    echo "========================================="
    echo "  Welcome to the Security Scanner"
    echo "========================================="
}

# Call function
greet

echo "Starting scan..."
```

**Output**:
```
=========================================
  Welcome to the Security Scanner
=========================================
Starting scan...
```

### 2.4 Example: Multiple Functions

```bash
#!/bin/bash
# Multiple functions

show_header() {
    echo "================================"
    echo "  System Information Script"
    echo "================================"
}

show_system_info() {
    echo "Hostname: $(hostname)"
    echo "User: $(whoami)"
    echo "Date: $(date)"
}

show_footer() {
    echo "================================"
    echo "  Scan Complete"
    echo "================================"
}

# Main script flow
show_header
show_system_info
show_footer
```

### 2.5 Example: Network Test Function

```bash
#!/bin/bash
# Network connectivity test

test_connection() {
    echo "Testing internet connection..."
    
    if ping -c 1 -W 2 8.8.8.8 &>/dev/null; then
        echo "✓ Internet connection is active"
    else
        echo "✗ No internet connection"
    fi
}

# Use function
test_connection
```

### 2.6 Function Placement

**WRONG** (function called before definition):
```bash
#!/bin/bash
greet  # ERROR - function not defined yet!

greet() {
    echo "Hello"
}
```

**RIGHT** (function defined first):
```bash
#!/bin/bash
greet() {
    echo "Hello"
}

greet  # Works!
```

### 2.7 Example: Menu System with Functions

```bash
#!/bin/bash
# Menu-driven script with functions

show_menu() {
    echo ""
    echo "=== Main Menu ==="
    echo "1) System Info"
    echo "2) Network Test"
    echo "3) Disk Usage"
    echo "4) Exit"
    echo "================="
}

system_info() {
    echo "System Information:"
    uname -a
}

network_test() {
    echo "Network Test:"
    ping -c 3 google.com
}

disk_usage() {
    echo "Disk Usage:"
    df -h /
}

# Main loop
while true; do
    show_menu
    read -p "Select option: " CHOICE
    
    case $CHOICE in
        1) system_info ;;
        2) network_test ;;
        3) disk_usage ;;
        4) echo "Goodbye!"; exit 0 ;;
        *) echo "Invalid option" ;;
    esac
done
```

---

## 3. Function Parameters

### 3.1 Understanding Parameters

**ABSOLUTE BEGINNER**: Parameters are values you pass to functions - like ingredients you give to a recipe.

**Syntax**:
```bash
function_name arg1 arg2 arg3
```

**Inside function, access with**:
- `$1` = first argument
- `$2` = second argument
- `$3` = third argument
- `$@` = all arguments
- `$#` = number of arguments

### 3.2 Example: Function with One Parameter

```bash
#!/bin/bash
# Greet a specific person

greet() {
    local NAME="$1"
    echo "Hello, $NAME!"
}

# Call with different names
greet "Alice"
greet "Bob"
greet "Charlie"
```

**Output**:
```
Hello, Alice!
Hello, Bob!
Hello, Charlie!
```

### 3.3 Example: Function with Multiple Parameters

```bash
#!/bin/bash
# Port checker with parameters

check_port() {
    local IP="$1"
    local PORT="$2"
    
    echo -n "Testing $IP:$PORT... "
    
    if timeout 1 bash -c "</dev/tcp/$IP/$PORT" 2>/dev/null; then
        echo "✓ OPEN"
        return 0
    else
        echo "✗ CLOSED"
        return 1
    fi
}

# Use function
check_port "192.168.1.1" "80"
check_port "192.168.1.1" "443"
check_port "192.168.1.1" "22"
```

### 3.4 Example: Parameter Validation

```bash
#!/bin/bash
# Function with parameter validation

create_backup() {
    local SOURCE="$1"
    local DEST="$2"
    
    # Validate parameters
    if [ -z "$SOURCE" ] || [ -z "$DEST" ]; then
        echo "✗ ERROR: Missing parameters"
        echo "Usage: create_backup <source> <destination>"
        return 1
    fi
    
    # Validate source exists
    if [ ! -d "$SOURCE" ]; then
        echo "✗ ERROR: Source directory not found: $SOURCE"
        return 1
    fi
    
    # Create backup
    echo "Backing up $SOURCE to $DEST..."
    tar -czf "$DEST" "$SOURCE"
    
    if [ $? -eq 0 ]; then
        echo "✓ Backup complete: $DEST"
        return 0
    else
        echo "✗ Backup failed"
        return 1
    fi
}

# Use function
create_backup "/etc/nginx" "/backup/nginx_$(date +%Y%m%d).tar.gz"
```

### 3.5 Example: Variable Number of Arguments

```bash
#!/bin/bash
# Function that accepts any number of arguments

scan_hosts() {
    echo "Scanning ${#@} hosts..."
    
    for HOST in "$@"; do
        echo -n "  $HOST... "
        if ping -c 1 -W 1 "$HOST" &>/dev/null; then
            echo "✓ UP"
        else
            echo "✗ DOWN"
        fi
    done
}

# Call with different number of arguments
scan_hosts "google.com"
scan_hosts "google.com" "github.com" "stackoverflow.com"
```

### 3.6 Example: Default Parameter Values

```bash
#!/bin/bash
# Function with default values

scan_port() {
    local IP="${1:-127.0.0.1}"     # Default to localhost
    local PORT="${2:-80}"           # Default to port 80
    local TIMEOUT="${3:-1}"         # Default timeout 1 second
    
    echo "Scanning $IP:$PORT (timeout: ${TIMEOUT}s)..."
    
    if timeout "$TIMEOUT" bash -c "</dev/tcp/$IP/$PORT" 2>/dev/null; then
        echo "✓ Port is OPEN"
    else
        echo "✗ Port is CLOSED"
    fi
}

# Call with different arguments
scan_port                          # Uses all defaults
scan_port "192.168.1.1"           # Custom IP, default port
scan_port "192.168.1.1" "443"     # Custom IP and port
scan_port "192.168.1.1" "443" "2" # All custom
```

### 3.7 Example: Named Parameters Pattern

```bash
#!/bin/bash
# Simulate named parameters using flags

create_user() {
    local USERNAME=""
    local EMAIL=""
    local ROLE="user"
    
    # Parse arguments
    while [ $# -gt 0 ]; do
        case "$1" in
            --username)
                USERNAME="$2"
                shift 2
                ;;
            --email)
                EMAIL="$2"
                shift 2
                ;;
            --role)
                ROLE="$2"
                shift 2
                ;;
            *)
                echo "Unknown option: $1"
                return 1
                ;;
        esac
    done
    
    echo "Creating user:"
    echo "  Username: $USERNAME"
    echo "  Email: $EMAIL"
    echo "  Role: $ROLE"
}

# Call with named parameters
create_user --username "alice" --email "alice@example.com" --role "admin"
```

---

## 4. Return Values and Exit Codes

### 4.1 Understanding Return Values

**ABSOLUTE BEGINNER**: Functions can return a status code (0-255) to indicate success or failure.

**Convention**:
- `0` = Success
- `Non-zero` = Error/Failure

**Syntax**:
```bash
function_name() {
    # Do something
    return 0  # Success
    # OR
    return 1  # Failure
}
```

**Check return value**:
```bash
function_name
if [ $? -eq 0 ]; then
    echo "Success!"
fi
```

### 4.2 Example: Function with Return Code

```bash
#!/bin/bash
# Function that returns status

is_port_open() {
    local IP="$1"
    local PORT="$2"
    
    if timeout 1 bash -c "</dev/tcp/$IP/$PORT" 2>/dev/null; then
        return 0  # Port is open
    else
        return 1  # Port is closed
    fi
}

# Use return value
if is_port_open "192.168.1.1" "80"; then
    echo "Port 80 is open - web server running"
else
    echo "Port 80 is closed - no web server"
fi
```

### 4.3 Example: Multiple Return Codes

```bash
#!/bin/bash
# Function with different exit codes

check_file() {
    local FILE="$1"
    
    if [ -z "$FILE" ]; then
        return 10  # Error: No file specified
    fi
    
    if [ ! -e "$FILE" ]; then
        return 20  # Error: File doesn't exist
    fi
    
    if [ ! -r "$FILE" ]; then
        return 30  # Error: File not readable
    fi
    
    return 0  # Success
}

# Use return codes
check_file "config.txt"
RESULT=$?

case $RESULT in
    0)  echo "File is accessible" ;;
    10) echo "Error: No file specified" ;;
    20) echo "Error: File doesn't exist" ;;
    30) echo "Error: File not readable" ;;
esac
```

### 4.4 Returning Data from Functions

**Important**: `return` only works with numbers (0-255). To return strings, use:

**Method 1: Echo (most common)**
```bash
get_ip() {
    local INTERFACE="$1"
    local IP=$(ip addr show "$INTERFACE" | grep "inet " | awk '{print $2}' | cut -d'/' -f1)
    echo "$IP"  # "Return" by echoing
}

# Capture output
MY_IP=$(get_ip "eth0")
echo "IP Address: $MY_IP"
```

**Method 2: Global variable**
```bash
get_system_info() {
    HOSTNAME=$(hostname)
    USER=$(whoami)
    DATE=$(date)
}

# Call function (sets global variables)
get_system_info
echo "Host: $HOSTNAME, User: $USER, Date: $DATE"
```

### 4.5 Example: Function Returning Multiple Values

```bash
#!/bin/bash
# Return multiple values via echo

get_network_info() {
    local INTERFACE="$1"
    local IP=$(ip addr show "$INTERFACE" | grep "inet " | awk '{print $2}' | cut -d'/' -f1)
    local MAC=$(ip link show "$INTERFACE" | grep "link/ether" | awk '{print $2}')
    
    echo "$IP|$MAC"  # Return both with delimiter
}

# Capture and parse
INFO=$(get_network_info "eth0")
IP=$(echo "$INFO" | cut -d'|' -f1)
MAC=$(echo "$INFO" | cut -d'|' -f2)

echo "IP: $IP"
echo "MAC: $MAC"
```

### 4.6 Example: Success/Failure Pattern

```bash
#!/bin/bash
# Common pattern: return 0 on success, 1 on failure

download_file() {
    local URL="$1"
    local OUTPUT="$2"
    
    echo "Downloading $URL..."
    
    if curl -s -o "$OUTPUT" "$URL"; then
        echo "✓ Download successful: $OUTPUT"
        return 0
    else
        echo "✗ Download failed"
        return 1
    fi
}

# Chain operations based on success
if download_file "https://example.com/file.txt" "file.txt"; then
    echo "Processing downloaded file..."
    cat file.txt
else
    echo "Cannot process - download failed"
    exit 1
fi
```

---

## 5. Variable Scope: Local vs Global

### 5.1 Understanding Variable Scope

**ABSOLUTE BEGINNER**: Scope determines where a variable can be accessed.

**Global variables**: Accessible everywhere (default)
**Local variables**: Only accessible inside function (use `local` keyword)

**Analogy**:
```
Global variable = Public announcement (everyone can hear)
Local variable = Private conversation in a room (only people in room can hear)
```

### 5.2 Global Variables (Default)

```bash
#!/bin/bash
# Variables are global by default

NAME="Alice"

change_name() {
    NAME="Bob"  # Changes global variable
    echo "Inside function: $NAME"
}

echo "Before: $NAME"
change_name
echo "After: $NAME"
```

**Output**:
```
Before: Alice
Inside function: Bob
After: Bob        ← Global variable was changed!
```

### 5.3 Local Variables

```bash
#!/bin/bash
# Use 'local' to keep variables inside function

NAME="Alice"

change_name() {
    local NAME="Bob"  # Local variable (doesn't affect global)
    echo "Inside function: $NAME"
}

echo "Before: $NAME"
change_name
echo "After: $NAME"
```

**Output**:
```
Before: Alice
Inside function: Bob
After: Alice       ← Global variable unchanged!
```

### 5.4 Why Use Local Variables?

**WITHOUT local (BAD)**:
```bash
#!/bin/bash
# Dangerous - functions can accidentally overwrite variables

TARGET="192.168.1.100"

scan_port() {
    TARGET="192.168.1.1"  # Oops! Overwrote global variable
    # ... scanning code
}

echo "Scanning $TARGET"
scan_port
echo "Next scan: $TARGET"  # Wrong IP!
```

**WITH local (GOOD)**:
```bash
#!/bin/bash
# Safe - local variables don't affect globals

TARGET="192.168.1.100"

scan_port() {
    local TARGET="192.168.1.1"  # Local copy
    # ... scanning code
}

echo "Scanning $TARGET"
scan_port
echo "Next scan: $TARGET"  # Correct IP!
```

### 5.5 Example: Function with Local Variables

```bash
#!/bin/bash
# Proper use of local variables

calculate_stats() {
    local FILE="$1"
    local LINES=$(wc -l < "$FILE")
    local WORDS=$(wc -w < "$FILE")
    local SIZE=$(du -h "$FILE" | cut -f1)
    
    echo "File: $FILE"
    echo "  Lines: $LINES"
    echo "  Words: $WORDS"
    echo "  Size: $SIZE"
}

# Call function multiple times - no variable conflicts
calculate_stats "file1.txt"
calculate_stats "file2.txt"
```

### 5.6 Best Practice: Always Use Local

```bash
#!/bin/bash
# BEST PRACTICE: Declare all function variables as local

check_port() {
    local IP="$1"        # Local
    local PORT="$2"      # Local
    local RESULT=""      # Local
    
    if timeout 1 bash -c "</dev/tcp/$IP/$PORT" 2>/dev/null; then
        RESULT="OPEN"
    else
        RESULT="CLOSED"
    fi
    
    echo "$IP:$PORT is $RESULT"
}
```

**BEGINNER TIP**: Make it a habit to use `local` for ALL variables inside functions.

---

## 6. Function Libraries

### 6.1 Understanding Function Libraries

**ABSOLUTE BEGINNER**: A function library is a separate file containing reusable functions that you can import into multiple scripts.

**Think of it like**:
- Library = Toolbox with tools
- Script = Project that uses tools from toolbox

### 6.2 Creating a Library File

**File: network_lib.sh**
```bash
#!/bin/bash
# Network utility function library

# Check if host is alive
is_host_alive() {
    local HOST="$1"
    ping -c 1 -W 1 "$HOST" &>/dev/null
    return $?
}

# Check if port is open
is_port_open() {
    local IP="$1"
    local PORT="$2"
    timeout 1 bash -c "</dev/tcp/$IP/$PORT" 2>/dev/null
    return $?
}

# Get IP of interface
get_interface_ip() {
    local INTERFACE="$1"
    ip addr show "$INTERFACE" | grep "inet " | awk '{print $2}' | cut -d'/' -f1
}
```

### 6.3 Using a Library with 'source'

**File: scanner.sh**
```bash
#!/bin/bash
# Main script that uses the library

# Import library
source ./network_lib.sh

# Now use functions from library
if is_host_alive "192.168.1.1"; then
    echo "Host is alive"
    
    if is_port_open "192.168.1.1" "80"; then
        echo "Port 80 is open"
    fi
fi

MY_IP=$(get_interface_ip "eth0")
echo "My IP: $MY_IP"
```

### 6.4 Example: Complete Security Library

**File: security_lib.sh**
```bash
#!/bin/bash
# Security function library

# Check if running as root
require_root() {
    if [ "$EUID" -ne 0 ]; then
        echo "✗ This script must run as root"
        exit 1
    fi
}

# Check if command exists
require_command() {
    local CMD="$1"
    if ! command -v "$CMD" &>/dev/null; then
        echo "✗ Required command not found: $CMD"
        exit 1
    fi
}

# Log message with timestamp
log_message() {
    local LEVEL="$1"
    local MESSAGE="$2"
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] [$LEVEL] $MESSAGE"
}

# Port scanner
scan_ports() {
    local TARGET="$1"
    local PORTS="${2:-22,80,443}"
    
    log_message "INFO" "Scanning $TARGET ports: $PORTS"
    
    IFS=',' read -ra PORT_ARRAY <<< "$PORTS"
    for PORT in "${PORT_ARRAY[@]}"; do
        if timeout 1 bash -c "</dev/tcp/$TARGET/$PORT" 2>/dev/null; then
            log_message "SUCCESS" "Port $PORT is OPEN"
        fi
    done
}
```

**File: main_script.sh**
```bash
#!/bin/bash
# Main security script using library

source ./security_lib.sh

# Use library functions
require_root
require_command "nmap"

log_message "INFO" "Starting security scan"
scan_ports "192.168.1.1" "22,80,443,3306"
log_message "INFO" "Scan complete"
```

### 6.5 Example: Multi-Library Project

```bash
#!/bin/bash
# Project structure with multiple libraries

# Import all libraries
source ./lib/network.sh
source ./lib/logging.sh
source ./lib/validation.sh
source ./lib/reporting.sh

# Use functions from different libraries
validate_ip "$TARGET_IP"
log_info "Scanning $TARGET_IP"
scan_network "$TARGET_IP"
generate_report "scan_results.txt"
```

### 6.6 Standard Library Locations

**Common patterns**:
```bash
# Same directory
source ./mylib.sh

# Relative path
source ./lib/mylib.sh
source ../common/mylib.sh

# Absolute path
source /opt/scripts/lib/mylib.sh

# Using variable
LIB_DIR="/opt/scripts/lib"
source "$LIB_DIR/mylib.sh"
```

---

## 7. Error Handling in Functions

### 7.1 Basic Error Handling

```bash
#!/bin/bash
# Function with error handling

process_file() {
    local FILE="$1"
    
    # Check if parameter provided
    if [ -z "$FILE" ]; then
        echo "✗ ERROR: No file specified" >&2
        return 1
    fi
    
    # Check if file exists
    if [ ! -f "$FILE" ]; then
        echo "✗ ERROR: File not found: $FILE" >&2
        return 2
    fi
    
    # Check if file is readable
    if [ ! -r "$FILE" ]; then
        echo "✗ ERROR: Cannot read file: $FILE" >&2
        return 3
    fi
    
    # Process file
    echo "✓ Processing $FILE..."
    cat "$FILE"
    return 0
}
```

### 7.2 Example: Try-Catch Pattern

```bash
#!/bin/bash
# Simulate try-catch with error handling

backup_database() {
    local DB_NAME="$1"
    local BACKUP_FILE="/backup/${DB_NAME}_$(date +%Y%m%d).sql"
    
    echo "Backing up database: $DB_NAME"
    
    # Try to create backup
    if mysqldump "$DB_NAME" > "$BACKUP_FILE" 2>/dev/null; then
        echo "✓ Backup successful: $BACKUP_FILE"
        return 0
    else
        echo "✗ Backup failed for $DB_NAME" >&2
        
        # Cleanup on failure
        [ -f "$BACKUP_FILE" ] && rm -f "$BACKUP_FILE"
        
        return 1
    fi
}

# Use with error handling
if backup_database "myapp"; then
    echo "Proceeding with next step..."
else
    echo "Backup failed - aborting" >&2
    exit 1
fi
```

### 7.3 Example: Validation Function

```bash
#!/bin/bash
# Comprehensive input validation

validate_ip() {
    local IP="$1"
    
    # Check not empty
    if [ -z "$IP" ]; then
        echo "✗ IP address cannot be empty" >&2
        return 1
    fi
    
    # Check format (basic validation)
    if [[ ! $IP =~ ^[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}$ ]]; then
        echo "✗ Invalid IP format: $IP" >&2
        return 2
    fi
    
    # Check each octet is 0-255
    IFS='.' read -ra OCTETS <<< "$IP"
    for OCTET in "${OCTETS[@]}"; do
        if [ "$OCTET" -gt 255 ]; then
            echo "✗ Invalid IP (octet > 255): $IP" >&2
            return 3
        fi
    done
    
    return 0
}

# Use validation
if validate_ip "192.168.1.1"; then
    echo "✓ Valid IP"
else
    echo "Cannot proceed with invalid IP"
    exit 1
fi
```

### 7.4 Example: Cleanup on Error

```bash
#!/bin/bash
# Function with cleanup on error

create_temp_workspace() {
    local WORKSPACE="/tmp/work_$$"
    
    # Create workspace
    if ! mkdir -p "$WORKSPACE"; then
        echo "✗ Failed to create workspace" >&2
        return 1
    fi
    
    # Try to setup workspace
    if ! cp -r ./templates "$WORKSPACE/"; then
        echo "✗ Failed to copy templates" >&2
        rm -rf "$WORKSPACE"  # Cleanup on error
        return 2
    fi
    
    echo "$WORKSPACE"  # Return workspace path
    return 0
}

# Use with error handling
WORKSPACE=$(create_temp_workspace)
if [ $? -ne 0 ]; then
    echo "Workspace setup failed"
    exit 1
fi

echo "Working in: $WORKSPACE"
```

---

## 8. Best Practices & Naming Conventions

### 8.1 Naming Conventions

**Good function names**:
```bash
# Verb + noun pattern
check_port()
scan_network()
create_backup()
validate_input()
generate_report()

# Boolean functions start with "is_" or "has_"
is_root()
is_port_open()
has_internet()

# Get/Set pattern
get_ip_address()
set_config_value()
```

**Bad function names**:
```bash
# Too vague
do_stuff()
process()
run()

# Not descriptive
func1()
temp()
```

### 8.2 Function Documentation

```bash
#!/bin/bash
# Document functions with comments

#
# Scans a range of ports on target host
# Arguments:
#   $1 - Target IP address
#   $2 - Port range (e.g., "1-1000" or "80,443,3306")
# Returns:
#   0 on success, 1 on error
# Example:
#   scan_port_range "192.168.1.1" "1-1000"
#
scan_port_range() {
    local TARGET="$1"
    local RANGE="$2"
    
    # Implementation...
}
```

### 8.3 Code Organization Pattern

```bash
#!/bin/bash
# Professional script structure

#============================================
# CONFIGURATION
#============================================
LOG_FILE="/var/log/scanner.log"
TIMEOUT=5

#============================================
# UTILITY FUNCTIONS
#============================================
log_message() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

#============================================
# VALIDATION FUNCTIONS
#============================================
validate_ip() {
    # ...
}

#============================================
# CORE FUNCTIONS
#============================================
scan_host() {
    # ...
}

#============================================
# MAIN EXECUTION
#============================================
main() {
    log_message "Starting scan..."
    scan_host "$TARGET"
    log_message "Scan complete"
}

# Run main function
main "$@"
```

### 8.4 Single Responsibility Principle

**BAD** (function does too much):
```bash
do_everything() {
    # Validates input
    # Scans network
    # Generates report
    # Sends email
    # Cleans up
    # ... 200 lines of code
}
```

**GOOD** (each function has one job):
```bash
validate_input() { ... }
scan_network() { ... }
generate_report() { ... }
send_email() { ... }
cleanup() { ... }

# Combine in main
main() {
    validate_input "$@"
    scan_network
    generate_report
    send_email
    cleanup
}
```

---

## 9. Advanced Function Techniques

### 9.1 Function Arrays

```bash
#!/bin/bash
# Return array from function

get_open_ports() {
    local TARGET="$1"
    local PORTS=(22 80 443 3306)
    local OPEN_PORTS=()
    
    for PORT in "${PORTS[@]}"; do
        if timeout 1 bash -c "</dev/tcp/$TARGET/$PORT" 2>/dev/null; then
            OPEN_PORTS+=("$PORT")
        fi
    done
    
    echo "${OPEN_PORTS[@]}"
}

# Use function
OPEN=$(get_open_ports "192.168.1.1")
IFS=' ' read -ra PORTS <<< "$OPEN"

echo "Open ports: ${PORTS[@]}"
```

### 9.2 Function Pipelines

```bash
#!/bin/bash
# Chain functions together

extract_ips() {
    grep -oE '[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}'
}

filter_private() {
    grep -vE '^(10\.|172\.(1[6-9]|2[0-9]|3[0-1])\.|192\.168\.)'
}

sort_unique() {
    sort -u
}

# Use in pipeline
cat logfile.txt | extract_ips | filter_private | sort_unique
```

### 9.3 Callback Functions

```bash
#!/bin/bash
# Pass function names as arguments

process_each_file() {
    local DIRECTORY="$1"
    local CALLBACK="$2"
    
    for FILE in "$DIRECTORY"/*; do
        $CALLBACK "$FILE"
    done
}

# Callback functions
show_size() {
    echo "Size of $1: $(du -h "$1" | cut -f1)"
}

count_lines() {
    echo "Lines in $1: $(wc -l < "$1")"
}

# Use callbacks
process_each_file "/var/log" "show_size"
process_each_file "/etc" "count_lines"
```

### 9.4 Function Aliases

```bash
#!/bin/bash
# Create shorthand aliases for functions

very_long_descriptive_function_name() {
    echo "This function has a really long name"
}

# Create alias
alias vl=very_long_descriptive_function_name

# Use shorter alias
vl
```

---

## 10. Organizing Large Scripts

### 10.1 Project Structure

```
my_project/
├── main.sh                 # Main entry point
├── lib/                    # Function libraries
│   ├── network.sh
│   ├── logging.sh
│   ├── validation.sh
│   └── reporting.sh
├── config/                 # Configuration files
│   └── settings.conf
├── data/                   # Data files
│   └── targets.txt
└── logs/                   # Log files
    └── scanner.log
```

### 10.2 Example: Main Script

**File: main.sh**
```bash
#!/bin/bash
# Professional OSCP-style network scanner

# Get script directory
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"

# Import libraries
source "$SCRIPT_DIR/lib/network.sh"
source "$SCRIPT_DIR/lib/logging.sh"
source "$SCRIPT_DIR/lib/validation.sh"
source "$SCRIPT_DIR/lib/reporting.sh"

# Load configuration
source "$SCRIPT_DIR/config/settings.conf"

# Main function
main() {
    setup_logging
    log_info "Starting network scanner v1.0"
    
    # Validate inputs
    validate_target "$TARGET" || exit 1
    
    # Run scan
    scan_result=$(scan_network "$TARGET")
    
    # Generate report
    generate_report "$scan_result" "$REPORT_FILE"
    
    log_info "Scan complete - report: $REPORT_FILE"
}

# Run main
main "$@"
```

### 10.3 Example: Modular Scanning Framework

```bash
#!/bin/bash
# Modular security scanning framework

# Module structure
modules/
├── discovery.sh        # Host discovery
├── port_scan.sh        # Port scanning
├── service_enum.sh     # Service enumeration
├── vuln_scan.sh        # Vulnerability scanning
└── reporting.sh        # Report generation

# Load all modules
for MODULE in modules/*.sh; do
    source "$MODULE"
done

# Main workflow
run_full_scan() {
    local TARGET="$1"
    
    # Discovery phase
    LIVE_HOSTS=$(module_discovery "$TARGET")
    
    # Port scanning
    for HOST in $LIVE_HOSTS; do
        OPEN_PORTS=$(module_port_scan "$HOST")
        
        # Service enumeration
        for PORT in $OPEN_PORTS; do
            SERVICE=$(module_service_enum "$HOST" "$PORT")
            
            # Vulnerability scanning
            VULNS=$(module_vuln_scan "$HOST" "$PORT" "$SERVICE")
        done
    done
    
    # Generate report
    module_reporting "$TARGET" "$RESULTS"
}
```

---

## 11. Real-World Security Frameworks

### 11.1 Complete Port Scanner Framework

```bash
#!/bin/bash
# Professional port scanner with modular functions

#================================
# CONFIGURATION
#================================
SCRIPT_NAME="PortScanner"
VERSION="1.0"
LOG_FILE="scanner_$(date +%Y%m%d_%H%M%S).log"

#================================
# LOGGING FUNCTIONS
#================================
log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

#================================
# VALIDATION FUNCTIONS
#================================
validate_ip() {
    local IP="$1"
    [[ $IP =~ ^[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}$ ]]
}

validate_port_range() {
    local RANGE="$1"
    [[ $RANGE =~ ^[0-9]+-[0-9]+$ ]]
}

#================================
# NETWORK FUNCTIONS
#================================
is_host_alive() {
    local HOST="$1"
    ping -c 1 -W 1 "$HOST" &>/dev/null
}

scan_port() {
    local IP="$1"
    local PORT="$2"
    timeout 1 bash -c "</dev/tcp/$IP/$PORT" 2>/dev/null
}

#================================
# CORE SCANNING FUNCTIONS
#================================
scan_host() {
    local TARGET="$1"
    local START_PORT="$2"
    local END_PORT="$3"
    
    log "Scanning $TARGET ports $START_PORT-$END_PORT"
    
    local OPEN_COUNT=0
    
    for (( PORT=START_PORT; PORT<=END_PORT; PORT++ )); do
        if scan_port "$TARGET" "$PORT"; then
            log "  ✓ Port $PORT OPEN"
            OPEN_COUNT=$((OPEN_COUNT + 1))
        fi
    done
    
    log "Scan complete - $OPEN_COUNT ports open"
}

#================================
# MAIN FUNCTION
#================================
main() {
    local TARGET="$1"
    local RANGE="${2:-1-1000}"
    
    log "$SCRIPT_NAME v$VERSION starting..."
    
    # Validate input
    if ! validate_ip "$TARGET"; then
        log "ERROR: Invalid IP address: $TARGET"
        exit 1
    fi
    
    # Check host is alive
    if ! is_host_alive "$TARGET"; then
        log "WARNING: Host may be offline: $TARGET"
    fi
    
    # Parse port range
    START=$(echo "$RANGE" | cut -d'-' -f1)
    END=$(echo "$RANGE" | cut -d'-' -f2)
    
    # Run scan
    scan_host "$TARGET" "$START" "$END"
    
    log "All scans complete!"
}

# Execute
main "$@"
```

### 11.2 Network Enumeration Framework

```bash
#!/bin/bash
# Complete network enumeration framework

source ./lib/discovery.sh
source ./lib/enumeration.sh
source ./lib/reporting.sh

enumerate_network() {
    local NETWORK="$1"
    
    # Phase 1: Discovery
    log_phase "Host Discovery"
    LIVE_HOSTS=$(discover_hosts "$NETWORK")
    log_result "Found $(echo "$LIVE_HOSTS" | wc -l) live hosts"
    
    # Phase 2: Port Scanning
    log_phase "Port Scanning"
    for HOST in $LIVE_HOSTS; do
        PORTS=$(scan_common_ports "$HOST")
        store_results "$HOST" "ports" "$PORTS"
    done
    
    # Phase 3: Service Detection
    log_phase "Service Detection"
    for HOST in $LIVE_HOSTS; do
        SERVICES=$(detect_services "$HOST")
        store_results "$HOST" "services" "$SERVICES"
    done
    
    # Phase 4: OS Detection
    log_phase "OS Fingerprinting"
    for HOST in $LIVE_HOSTS; do
        OS=$(detect_os "$HOST")
        store_results "$HOST" "os" "$OS"
    done
    
    # Generate final report
    generate_comprehensive_report
}
```

---

## 12. Recursive Functions

### 12.1 Understanding Recursion

**ABSOLUTE BEGINNER**: A recursive function is one that calls itself.

**Example: Countdown**
```bash
#!/bin/bash
# Recursive countdown

countdown() {
    local NUM="$1"
    
    if [ "$NUM" -le 0 ]; then
        echo "Liftoff!"
        return
    fi
    
    echo "$NUM..."
    sleep 1
    countdown $((NUM - 1))  # Function calls itself
}

countdown 10
```

### 12.2 Example: Factorial

```bash
#!/bin/bash
# Calculate factorial recursively

factorial() {
    local N="$1"
    
    if [ "$N" -le 1 ]; then
        echo 1
        return
    fi
    
    local PREV=$(factorial $((N - 1)))
    echo $((N * PREV))
}

echo "5! = $(factorial 5)"
# Output: 5! = 120
```

### 12.3 Example: Directory Tree Walker

```bash
#!/bin/bash
# Recursively walk directory tree

walk_directory() {
    local DIR="$1"
    local INDENT="${2:-}"
    
    for ITEM in "$DIR"/*; do
        if [ -d "$ITEM" ]; then
            echo "${INDENT}[DIR] $(basename "$ITEM")"
            walk_directory "$ITEM" "${INDENT}  "  # Recurse
        else
            echo "${INDENT}[FILE] $(basename "$ITEM")"
        fi
    done
}

walk_directory "/etc/nginx"
```

---

## 13. Common Mistakes

### 13.1 Calling Function Before Definition

**WRONG**:
```bash
greet  # ERROR - not defined yet!

greet() {
    echo "Hello"
}
```

### 13.2 Using Parentheses When Calling

**WRONG**:
```bash
my_function()  # Tries to call with empty subshell
```

**RIGHT**:
```bash
my_function    # Correct
```

### 13.3 Not Using Local Variables

**WRONG**:
```bash
process() {
    FILE="$1"  # Overwrites global $FILE!
}
```

**RIGHT**:
```bash
process() {
    local FILE="$1"  # Safe, local only
}
```

---

## 14. Hands-On Labs

### Lab 1: Build Modular Network Scanner

Create a scanner with separate functions for discovery, port scanning, and reporting.

**Solution outline**:
```bash
# Functions:
- discover_hosts()
- scan_ports()
- identify_service()
- generate_report()
```

---

## 15. Interview Questions

**Q1: What's the difference between `return` and `exit`?**
- `return`: Exits function, returns to caller
- `exit`: Terminates entire script

**Q2: How do you return a string from a function?**
- Use `echo` and capture with command substitution: `RESULT=$(my_function)`

**Q3: What's variable scope?**
- Global: Accessible everywhere (default)
- Local: Only inside function (use `local` keyword)

---

## 16. Summary

**Completed**: File 6 - Functions & Code Organization
**Lines**: 1,100+
**Examples**: 20+ working code samples
**Next**: File 7 - Text Processing (sed/awk/grep)

Functions transform scripts from simple sequences into professional, maintainable code!
