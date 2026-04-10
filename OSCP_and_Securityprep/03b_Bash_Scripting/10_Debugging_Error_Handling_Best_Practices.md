# 10 - Debugging, Error Handling & Best Practices

> **Executive Summary**: Writing code is one thing - writing bulletproof, professional, maintainable code is another. This final module covers everything you need to write production-ready Bash scripts: debugging techniques (set -x, set -e, set -u), error handling patterns, logging, input validation, security best practices, code review checklists, and professional script templates. You'll learn how to catch bugs before they happen, handle errors gracefully, write defensive code, and follow industry best practices. These skills separate hobbyist scripts from professional tools used in production environments and OSCP exam scenarios where reliability is critical.

**What You'll Learn**:
- Complete beginner's guide to debugging
- Bash debugging options (set -x, set -e, set -u, set -o pipefail)
- Error handling patterns and strategies
- Input validation and sanitization
- Logging best practices
- Security considerations and pitfalls
- Code review checklist
- Professional script template
- Performance optimization
- 20+ real-world debugging examples
- Production-ready patterns for OSCP

---

## Table of Contents

1. [What is Debugging? (ABSOLUTE BEGINNER)](#1-what-is-debugging)
2. [Bash Debug Mode (set -x)](#2-bash-debug-mode)
3. [Strict Mode (set -e, set -u)](#3-strict-mode)
4. [Error Handling Patterns](#4-error-handling)
5. [Input Validation](#5-input-validation)
6. [Logging Best Practices](#6-logging)
7. [Exit Codes and Status](#7-exit-codes)
8. [Defensive Programming](#8-defensive-programming)
9. [Security Best Practices](#9-security)
10. [Code Review Checklist](#10-code-review)
11. [Professional Script Template](#11-professional-template)
12. [Common Pitfalls](#12-common-pitfalls)
13. [Performance Optimization](#13-performance)
14. [Testing Scripts](#14-testing)
15. [Hands-On Labs](#15-hands-on-labs)
16. [Interview Questions](#16-interview-questions)
17. [Final Project](#17-final-project)
18. [Summary & Next Steps](#18-summary)

---

## 1. What is Debugging? (ABSOLUTE BEGINNER)

### 1.1 Understanding Debugging

**ABSOLUTE BEGINNER EXPLANATION**:

Debugging is finding and fixing problems (bugs) in your code:
- **Bug** = Something that doesn't work as expected
- **Debugging** = Finding WHY it doesn't work
- **Fixing** = Making it work correctly

**ANALOGY - Car Mechanic**:
```
Problem:  Car won't start
Debug:    Check battery → Dead battery found
Fix:      Replace battery
Test:     Car starts successfully
```

**In Scripting**:
```
Problem:  Script crashes with "command not found"
Debug:    Add set -x to see what's running
Find:     Typo in command name (nap instead of nmap)
Fix:      Correct the typo
Test:     Script runs successfully
```

### 1.2 Common Bug Types

| Bug Type | Description | Example |
|----------|-------------|---------|
| **Syntax Error** | Invalid Bash syntax | Missing quote: `echo "hello` |
| **Logic Error** | Wrong logic | Using `-eq` for strings |
| **Runtime Error** | Fails during execution | File not found |
| **Edge Case** | Fails with unusual input | Empty variable |

---

## 2. Bash Debug Mode (set -x)

### 2.1 Understanding set -x

**ABSOLUTE BEGINNER**: `set -x` shows each command before running it - like a slow-motion replay.

**Basic Usage**:
```bash
#!/bin/bash
set -x  # Enable debug mode

echo "Starting script"
MYVAR="test"
echo "Variable is: $MYVAR"

set +x  # Disable debug mode
```

**Output**:
```
+ echo 'Starting script'
Starting script
+ MYVAR=test
+ echo 'Variable is: test'
Variable is: test
```

### 2.2 Example: Debug Specific Section

```bash
#!/bin/bash
# Only debug problematic section

echo "This won't show debug info"

set -x  # Turn on debugging
RESULT=$(cat /nonexistent/file.txt)
echo "Result: $RESULT"
set +x  # Turn off debugging

echo "Back to normal"
```

### 2.3 Example: Conditional Debugging

```bash
#!/bin/bash
# Enable debugging with environment variable

if [ "$DEBUG" = "1" ]; then
    set -x
fi

# Your script here
echo "Running script..."
```

**Usage**:
```bash
# Normal run
./script.sh

# Debug run
DEBUG=1 ./script.sh
```

### 2.4 Example: Debug Functions

```bash
#!/bin/bash

debug_function() {
    set -x
    local VAR1="$1"
    local VAR2="$2"
    echo "Processing: $VAR1 and $VAR2"
    set +x
}

debug_function "hello" "world"
```

---

## 3. Strict Mode (set -e, set -u, set -o pipefail)

### 3.1 Understanding Strict Mode

**ABSOLUTE BEGINNER**: Strict mode makes Bash less forgiving - it stops when things go wrong instead of continuing with errors.

**The "Unofficial Bash Strict Mode"**:
```bash
#!/bin/bash
set -euo pipefail
IFS=$'\n\t'
```

### 3.2 set -e (Exit on Error)

**What it does**: Exit immediately if any command fails.

**WITHOUT set -e (BAD)**:
```bash
#!/bin/bash
# Dangerous - continues even after errors!

cd /nonexistent/directory
rm -rf *  # DANGER: runs in current directory!
```

**WITH set -e (GOOD)**:
```bash
#!/bin/bash
set -e  # Exit on error

cd /nonexistent/directory  # Script stops here
rm -rf *  # Never runs - safe!
```

### 3.3 set -u (Error on Undefined Variables)

**What it does**: Treat undefined variables as errors.

**WITHOUT set -u (BAD)**:
```bash
#!/bin/bash
# Dangerous - typo creates empty variable

rm -rf /$DIRECTORY_NAME/*  # If DIRECTORY_NAME unset, runs: rm -rf /*
```

**WITH set -u (GOOD)**:
```bash
#!/bin/bash
set -u  # Error on undefined variables

rm -rf /$DIRECTORY_NAME/*  # Exits with error if unset
```

### 3.4 set -o pipefail (Catch Pipe Failures)

**What it does**: Make pipelines fail if ANY command fails.

**WITHOUT pipefail (BAD)**:
```bash
#!/bin/bash
# Only checks last command

cat nonexistent.txt | grep "error"
echo "Exit code: $?"  # Shows 1 (grep found nothing) not the real error
```

**WITH pipefail (GOOD)**:
```bash
#!/bin/bash
set -o pipefail

cat nonexistent.txt | grep "error"
echo "Exit code: $?"  # Shows error from cat
```

### 3.5 Complete Strict Mode Template

```bash
#!/bin/bash
# Professional script with strict mode

set -euo pipefail
IFS=$'\n\t'

# Optional: trap for cleanup
trap 'echo "Error on line $LINENO"' ERR

main() {
    echo "Script running with strict mode"
    # Your code here
}

main "$@"
```

---

## 4. Error Handling Patterns

### 4.1 Basic Error Handling

```bash
#!/bin/bash
# Check command success

if ! command_that_might_fail; then
    echo "ERROR: Command failed" >&2
    exit 1
fi
```

### 4.2 Error Handling with Functions

```bash
#!/bin/bash

error_exit() {
    echo "ERROR: $1" >&2
    exit "${2:-1}"
}

# Usage
if [ ! -f "required_file.txt" ]; then
    error_exit "Required file not found" 2
fi
```

### 4.3 Try-Catch Pattern

```bash
#!/bin/bash
# Simulate try-catch

try() {
    "$@" 2>/dev/null
    return $?
}

catch() {
    local exit_code=$?
    echo "Caught error with code: $exit_code" >&2
    return $exit_code
}

# Usage
if ! try risky_command; then
    catch
    echo "Handling the error..."
fi
```

### 4.4 Comprehensive Error Handling Example

```bash
#!/bin/bash
set -euo pipefail

# Error handling function
handle_error() {
    local line_num="$1"
    local error_code="$2"
    echo "ERROR: Command failed on line $line_num with exit code $error_code" >&2
    cleanup
    exit "$error_code"
}

# Cleanup function
cleanup() {
    echo "Cleaning up..."
    rm -f /tmp/tempfile_$$
}

# Set trap
trap 'handle_error ${LINENO} $?' ERR
trap cleanup EXIT

# Main script
echo "Starting risky operations..."

# Safe file operations
if [ -f "important_file.txt" ]; then
    cp important_file.txt /tmp/tempfile_$$
else
    handle_error ${LINENO} 1
fi

echo "Operations complete"
```

---

## 5. Input Validation

### 5.1 Validate Required Arguments

```bash
#!/bin/bash
# Validate script arguments

if [ $# -lt 2 ]; then
    echo "ERROR: Missing required arguments" >&2
    echo "Usage: $0 <input_file> <output_file>" >&2
    exit 1
fi

INPUT_FILE="$1"
OUTPUT_FILE="$2"

# Validate input file exists
if [ ! -f "$INPUT_FILE" ]; then
    echo "ERROR: Input file not found: $INPUT_FILE" >&2
    exit 1
fi

# Validate output file doesn't exist (prevent overwrite)
if [ -f "$OUTPUT_FILE" ]; then
    echo "ERROR: Output file already exists: $OUTPUT_FILE" >&2
    exit 1
fi

echo "Validation passed"
```

### 5.2 Validate Input Format

```bash
#!/bin/bash
# Validate IP address format

validate_ip() {
    local ip="$1"
    
    # Check format
    if [[ ! $ip =~ ^[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}$ ]]; then
        echo "ERROR: Invalid IP format: $ip" >&2
        return 1
    fi
    
    # Check each octet is 0-255
    IFS='.' read -ra OCTETS <<< "$ip"
    for octet in "${OCTETS[@]}"; do
        if [ "$octet" -gt 255 ]; then
            echo "ERROR: IP octet out of range: $octet" >&2
            return 1
        fi
    done
    
    return 0
}

# Usage
if ! validate_ip "192.168.1.1"; then
    exit 1
fi
```

### 5.3 Sanitize User Input

```bash
#!/bin/bash
# Prevent command injection

sanitize_input() {
    local input="$1"
    
    # Remove dangerous characters
    input="${input//[;|&<>]/}"
    
    # Remove leading/trailing whitespace
    input=$(echo "$input" | xargs)
    
    echo "$input"
}

# Usage
read -p "Enter filename: " FILENAME
SAFE_FILENAME=$(sanitize_input "$FILENAME")

echo "Sanitized: $SAFE_FILENAME"
```

### 5.4 Complete Validation Example

```bash
#!/bin/bash
# Comprehensive input validation

validate_all_inputs() {
    local target="$1"
    local port="$2"
    local username="$3"
    
    # Validate target (IP or hostname)
    if [ -z "$target" ]; then
        echo "ERROR: Target cannot be empty" >&2
        return 1
    fi
    
    # Validate port number
    if ! [[ "$port" =~ ^[0-9]+$ ]] || [ "$port" -lt 1 ] || [ "$port" -gt 65535 ]; then
        echo "ERROR: Invalid port: $port (must be 1-65535)" >&2
        return 1
    fi
    
    # Validate username (alphanumeric only)
    if ! [[ "$username" =~ ^[a-zA-Z0-9_-]+$ ]]; then
        echo "ERROR: Invalid username: $username" >&2
        return 1
    fi
    
    return 0
}

# Usage
if validate_all_inputs "$TARGET" "$PORT" "$USER"; then
    echo "All inputs valid - proceeding..."
else
    exit 1
fi
```

---

## 6. Logging Best Practices

### 6.1 Basic Logging Function

```bash
#!/bin/bash

LOG_FILE="/var/log/myscript.log"

log() {
    local level="$1"
    shift
    local message="$@"
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] [$level] $message" | tee -a "$LOG_FILE"
}

# Usage
log "INFO" "Script started"
log "ERROR" "Something went wrong"
log "SUCCESS" "Operation complete"
```

### 6.2 Log Levels

```bash
#!/bin/bash

LOG_FILE="app.log"
LOG_LEVEL="${LOG_LEVEL:-INFO}"  # Default to INFO

log_debug() { [ "$LOG_LEVEL" = "DEBUG" ] && log "DEBUG" "$@"; }
log_info() { log "INFO" "$@"; }
log_warn() { log "WARN" "$@"; }
log_error() { log "ERROR" "$@" >&2; }

log() {
    local level="$1"
    shift
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] [$level] $@" >> "$LOG_FILE"
}

# Usage
log_debug "Detailed debug information"
log_info "General information"
log_warn "Warning message"
log_error "Error occurred"
```

### 6.3 Log Rotation

```bash
#!/bin/bash
# Rotate logs when they get too large

LOG_FILE="script.log"
MAX_SIZE=10485760  # 10MB

rotate_log() {
    if [ -f "$LOG_FILE" ]; then
        SIZE=$(stat -f%z "$LOG_FILE" 2>/dev/null || stat -c%s "$LOG_FILE" 2>/dev/null)
        
        if [ "$SIZE" -gt "$MAX_SIZE" ]; then
            mv "$LOG_FILE" "${LOG_FILE}.old"
            gzip "${LOG_FILE}.old"
            echo "Log rotated at $(date)" > "$LOG_FILE"
        fi
    fi
}

rotate_log
```

### 6.4 Complete Logging Example

```bash
#!/bin/bash
# Professional logging system

readonly SCRIPT_NAME=$(basename "$0")
readonly LOG_DIR="/var/log/${SCRIPT_NAME%.sh}"
readonly LOG_FILE="$LOG_DIR/${SCRIPT_NAME%.sh}_$(date +%Y%m%d).log"

setup_logging() {
    mkdir -p "$LOG_DIR"
    
    # Start log
    log_info "=========================================="
    log_info "Script: $SCRIPT_NAME"
    log_info "Started: $(date)"
    log_info "User: $(whoami)"
    log_info "=========================================="
}

log_message() {
    local level="$1"
    shift
    local message="$@"
    local timestamp="$(date '+%Y-%m-%d %H:%M:%S')"
    
    # Log to file
    echo "[$timestamp] [$level] $message" >> "$LOG_FILE"
    
    # Also print to console for INFO and ERROR
    if [ "$level" = "INFO" ] || [ "$level" = "ERROR" ]; then
        echo "[$level] $message"
    fi
}

log_info() { log_message "INFO" "$@"; }
log_error() { log_message "ERROR" "$@" >&2; }
log_debug() { log_message "DEBUG" "$@"; }

# Usage
setup_logging
log_info "Processing started"
log_debug "Debug information"
log_error "An error occurred"
```

---

## 7. Exit Codes and Status

### 7.1 Standard Exit Codes

```bash
#!/bin/bash
# Use meaningful exit codes

readonly EXIT_SUCCESS=0
readonly EXIT_GENERAL_ERROR=1
readonly EXIT_INVALID_ARGUMENT=2
readonly EXIT_FILE_NOT_FOUND=3
readonly EXIT_PERMISSION_DENIED=4

if [ ! -f "$CONFIG_FILE" ]; then
    echo "ERROR: Config file not found" >&2
    exit $EXIT_FILE_NOT_FOUND
fi

if [ ! -r "$CONFIG_FILE" ]; then
    echo "ERROR: Cannot read config file" >&2
    exit $EXIT_PERMISSION_DENIED
fi

exit $EXIT_SUCCESS
```

### 7.2 Check and Report Exit Status

```bash
#!/bin/bash

run_with_status() {
    local cmd="$@"
    
    echo "Running: $cmd"
    $cmd
    local exit_code=$?
    
    if [ $exit_code -eq 0 ]; then
        echo "✓ Command succeeded"
    else
        echo "✗ Command failed with exit code: $exit_code" >&2
    fi
    
    return $exit_code
}

# Usage
if run_with_status nmap -sn 192.168.1.0/24; then
    echo "Scan completed successfully"
else
    echo "Scan failed"
    exit 1
fi
```

---

## 8. Defensive Programming

### 8.1 Check Before Acting

```bash
#!/bin/bash
# Always check before destructive operations

safe_delete() {
    local file="$1"
    
    # Check file exists
    if [ ! -e "$file" ]; then
        echo "ERROR: File doesn't exist: $file" >&2
        return 1
    fi
    
    # Check we have permission
    if [ ! -w "$file" ]; then
        echo "ERROR: No write permission: $file" >&2
        return 1
    fi
    
    # Check it's a regular file (not directory)
    if [ ! -f "$file" ]; then
        echo "ERROR: Not a regular file: $file" >&2
        return 1
    fi
    
    # Finally, safe to delete
    rm -f "$file"
    echo "Deleted: $file"
}

safe_delete "myfile.txt"
```

### 8.2 Use Temporary Files Safely

```bash
#!/bin/bash
# Safe temporary file handling

# Create unique temp file
TEMP_FILE=$(mktemp) || exit 1

# Cleanup on exit
trap "rm -f '$TEMP_FILE'" EXIT

# Use temp file
echo "data" > "$TEMP_FILE"
```

### 8.3 Prevent Race Conditions

```bash
#!/bin/bash
# Use lock files to prevent concurrent execution

LOCK_FILE="/var/run/myscript.lock"

acquire_lock() {
    if [ -f "$LOCK_FILE" ]; then
        PID=$(cat "$LOCK_FILE")
        if ps -p "$PID" > /dev/null 2>&1; then
            echo "ERROR: Another instance is running (PID: $PID)" >&2
            exit 1
        else
            echo "Removing stale lock file"
            rm -f "$LOCK_FILE"
        fi
    fi
    
    echo $$ > "$LOCK_FILE"
}

release_lock() {
    rm -f "$LOCK_FILE"
}

trap release_lock EXIT

acquire_lock
echo "Running exclusively..."
sleep 5
```

---

## 9. Security Best Practices

### 9.1 Never Store Passwords in Scripts

**BAD**:
```bash
#!/bin/bash
PASSWORD="mysecretpassword123"  # NEVER DO THIS!
```

**GOOD**:
```bash
#!/bin/bash
# Read from secure source

# Option 1: Environment variable
PASSWORD="$MYSQL_PASSWORD"

# Option 2: Read from file with restricted permissions
PASSWORD=$(cat /secure/password.txt)

# Option 3: Prompt user
read -s -p "Enter password: " PASSWORD
echo
```

### 9.2 Validate and Sanitize All Input

```bash
#!/bin/bash
# Prevent command injection

unsafe_command() {
    # DANGER: User input directly in command
    FILENAME="$1"
    cat "$FILENAME"  # User could pass "; rm -rf /"
}

safe_command() {
    # SAFE: Validate input first
    local filename="$1"
    
    # Only allow alphanumeric, dash, underscore
    if [[ ! $filename =~ ^[a-zA-Z0-9_-]+$ ]]; then
        echo "ERROR: Invalid filename" >&2
        return 1
    fi
    
    cat "$filename"
}
```

### 9.3 Use Absolute Paths

```bash
#!/bin/bash
# Use absolute paths for security

# BAD: Relative to PATH
rm file.txt

# GOOD: Absolute path
/bin/rm /path/to/file.txt
```

### 9.4 Set Restrictive Permissions

```bash
#!/bin/bash
# Create files with secure permissions

# Set umask
umask 077  # New files will be 600 (rw-------)

# Create file
touch secure_file.txt

# Explicit permissions
chmod 600 sensitive_data.txt
chmod 700 myscript.sh
```

---

## 10. Code Review Checklist

```markdown
## Bash Script Code Review Checklist

### General
- [ ] Script has shebang (#!/bin/bash)
- [ ] Uses strict mode (set -euo pipefail)
- [ ] Has meaningful comments
- [ ] Functions have docstrings
- [ ] No hardcoded passwords/secrets

### Error Handling
- [ ] All critical commands check exit codes
- [ ] Proper error messages to stderr
- [ ] Cleanup on error (trap)
- [ ] Input validation for all user input
- [ ] File existence checks before operations

### Variables
- [ ] All variables quoted ("$VAR")
- [ ] Local variables in functions
- [ ] Readonly for constants
- [ ] Meaningful variable names (not x, y, z)

### Security
- [ ] No command injection vulnerabilities
- [ ] Input sanitization
- [ ] Temporary files cleaned up
- [ ] Restrictive file permissions
- [ ] No eval unless absolutely necessary

### Best Practices
- [ ] Functions for reusable code
- [ ] One purpose per function
- [ ] Exit codes documented
- [ ] Logging implemented
- [ ] No parsing ls output
```

---

## 11. Professional Script Template

```bash
#!/bin/bash
#
# Script Name: professional_template.sh
# Description: Template for professional Bash scripts
# Author: Your Name
# Date: 2024-01-01
# Version: 1.0
#
# Usage: ./professional_template.sh [options] <arguments>
#

#============================================
# STRICT MODE
#============================================
set -euo pipefail
IFS=$'\n\t'

#============================================
# CONSTANTS
#============================================
readonly SCRIPT_NAME="$(basename "$0")"
readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
readonly VERSION="1.0.0"

#============================================
# CONFIGURATION
#============================================
LOG_FILE="/var/log/${SCRIPT_NAME%.sh}.log"
VERBOSE=false

#============================================
# HELPER FUNCTIONS
#============================================

# Display usage information
usage() {
    cat <<EOF
Usage: $SCRIPT_NAME [OPTIONS] <target>

Description:
    Professional script template

Options:
    -h, --help      Show this help message
    -v, --verbose   Enable verbose output
    -V, --version   Show version

Examples:
    $SCRIPT_NAME target.example.com
    $SCRIPT_NAME -v 192.168.1.1

EOF
}

# Logging function
log() {
    local level="$1"
    shift
    local message="$@"
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] [$level] $message" | tee -a "$LOG_FILE"
}

log_info() { log "INFO" "$@"; }
log_error() { log "ERROR" "$@" >&2; }
log_debug() { $VERBOSE && log "DEBUG" "$@"; }

# Error handler
error_exit() {
    log_error "$1"
    cleanup
    exit "${2:-1}"
}

# Cleanup function
cleanup() {
    log_debug "Cleaning up..."
    # Add cleanup tasks here
}

# Trap errors and cleanup
trap 'error_exit "Error on line $LINENO"' ERR
trap cleanup EXIT INT TERM

#============================================
# INPUT VALIDATION
#============================================

validate_inputs() {
    if [ -z "${TARGET:-}" ]; then
        error_exit "Target not specified" 1
    fi
    
    log_debug "Validation passed"
}

#============================================
# CORE FUNCTIONS
#============================================

main() {
    log_info "=========================================="
    log_info "$SCRIPT_NAME v$VERSION started"
    log_info "=========================================="
    
    validate_inputs
    
    # Main logic here
    log_info "Processing target: $TARGET"
    
    log_info "Completed successfully"
}

#============================================
# ARGUMENT PARSING
#============================================

while [ $# -gt 0 ]; do
    case "$1" in
        -h|--help)
            usage
            exit 0
            ;;
        -v|--verbose)
            VERBOSE=true
            shift
            ;;
        -V|--version)
            echo "$SCRIPT_NAME version $VERSION"
            exit 0
            ;;
        -*)
            error_exit "Unknown option: $1" 2
            ;;
        *)
            TARGET="$1"
            shift
            ;;
    esac
done

#============================================
# MAIN EXECUTION
#============================================

main "$@"
```

---

## 12. Common Pitfalls

### 12.1 Unquoted Variables

**WRONG**:
```bash
FILE="my file.txt"
cat $FILE  # Tries to cat "my" and "file.txt" separately
```

**RIGHT**:
```bash
FILE="my file.txt"
cat "$FILE"  # Correctly cats "my file.txt"
```

### 12.2 Word Splitting in For Loops

**WRONG**:
```bash
for FILE in $(ls *.txt); do  # Breaks on spaces
    echo "$FILE"
done
```

**RIGHT**:
```bash
for FILE in *.txt; do  # Glob expansion
    echo "$FILE"
done
```

### 12.3 Using [ ] Instead of [[ ]]

**WRONG**:
```bash
if [ $VAR = "value" ]; then  # Fails if $VAR is empty
```

**RIGHT**:
```bash
if [[ $VAR = "value" ]]; then  # Safe
```

---

## 13. Performance Optimization

### 13.1 Use Built-ins Instead of External Commands

**SLOW**:
```bash
BASENAME=$(basename "$FILE")
DIRNAME=$(dirname "$FILE")
```

**FAST**:
```bash
BASENAME="${FILE##*/}"
DIRNAME="${FILE%/*}"
```

### 13.2 Avoid Unnecessary Subshells

**SLOW**:
```bash
COUNT=$(cat file.txt | wc -l)
```

**FAST**:
```bash
COUNT=$(wc -l < file.txt)
```

### 13.3 Use Parallel Processing

```bash
#!/bin/bash
# Process files in parallel

for FILE in *.txt; do
    process_file "$FILE" &
    
    # Limit concurrent processes
    if (( $(jobs -r | wc -l) >= 10 )); then
        wait -n
    fi
done

wait  # Wait for all
```

---

## 14. Testing Scripts

### 14.1 Manual Testing Checklist

```bash
#!/bin/bash
# Test script manually

# Test 1: Normal operation
./script.sh valid_input

# Test 2: Missing arguments
./script.sh

# Test 3: Invalid input
./script.sh "'; rm -rf /"

# Test 4: File doesn't exist
./script.sh /nonexistent/file

# Test 5: Run as non-root
sudo -u nobody ./script.sh

# Test 6: Run with different locales
LC_ALL=C ./script.sh
```

### 14.2 Automated Testing

```bash
#!/bin/bash
# Simple test framework

test_count=0
test_passed=0

assert_equals() {
    local expected="$1"
    local actual="$2"
    local message="${3:-Test}"
    
    test_count=$((test_count + 1))
    
    if [ "$expected" = "$actual" ]; then
        echo "✓ PASS: $message"
        test_passed=$((test_passed + 1))
    else
        echo "✗ FAIL: $message"
        echo "  Expected: $expected"
        echo "  Actual: $actual"
    fi
}

# Run tests
assert_equals "5" "$(echo '2 + 3' | bc)" "Addition test"
assert_equals "10" "$(echo '5 * 2' | bc)" "Multiplication test"

# Report
echo ""
echo "Tests: $test_passed/$test_count passed"
```

---

## 15. Hands-On Labs

**Lab 1**: Take a simple script and add comprehensive error handling
**Lab 2**: Add logging to an existing script
**Lab 3**: Create a production-ready script using the professional template

---

## 16. Interview Questions

**Q1**: What does set -euo pipefail do?
**A**: Exit on error, error on undefined variables, catch pipe failures

**Q2**: How do you debug a Bash script?
**A**: Use set -x, echo statements, or shellcheck

**Q3**: What's the best way to handle errors?
**A**: Check exit codes, use trap for cleanup, meaningful error messages

---

## 17. Final Project

**Build a Complete OSCP Reconnaissance Tool**:
- Input validation
- Error handling
- Logging
- Progress indicators
- Professional output
- Graceful cleanup

---

## 18. Summary & Next Steps

**Completed**: All 10 Bash Scripting Modules!

**What You've Learned**:
1. Bash basics and first scripts
2. Variables and data types
3. Input/output and redirection
4. Conditional logic
5. Loops and iteration
6. Functions and organization
7. Text processing (grep/sed/awk)
8. System interaction
9. Network scripting
10. Debugging and best practices

**Total**: 10,000+ lines of comprehensive, OSCP-ready content!

**Next Steps**:
1. Practice with OSCP labs
2. Build your own tools
3. Contribute to open source
4. Keep learning!

**Congratulations - you're now a Bash scripting expert!**
