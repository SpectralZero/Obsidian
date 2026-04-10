
# Debugging and Error Handling: Broken Scripts

> **Executive Summary**: Scripts rarely work the first time. They fail silently, variables are empty, or they keep running after a Critical Error. This chapter covers the Trace Mode (`-x`), strict error checking (`set -e`), and catching Ctrl+C signals with `trap` to cleanup temp files.

## 1. Learning Objectives
By the end of this chapter, you will be able to:
- **Trace Execution**: Use `set -x` to see what Bash is thinking.
- **Fail Fast**: Use `set -e` and `set -u` to stop on errors/unbound vars.
- **Cleanup**: Use `trap` to delete temp files on exit.
- **Logic Debugging**: Use `echo` checkpoints.

## 2. Core Concepts: The `set` command

### 2.1 Trace (`set -x`)
Prints every command to Stderr before running it. Shows variable expansion.
*   **Enable**: `set -x`
*   **Disable**: `set +x`

```bash
#!/bin/bash
set -x
NAME="World"
echo "Hello $NAME"
# Output:
# + NAME=World
# + echo 'Hello World'
# Hello World
```

### 2.2 Strict Mode (`set -euo pipefail`)
The "Safety First" header.
*   `-e`: Exit immediately if a command exits with non-zero.
*   `-u`: Exit if we try to use an undefined variable (catches typos).
*   `-o pipefail`: In a pipe `cmd1 | cmd2`, if `cmd1` fails, the whole pipe fails. (Default is to return exit code of `cmd2`).

## 3. Deep Dive: Traps

### 3.1 Handling Signals
When you press Ctrl+C, you send `SIGINT`. Scripts usually die instantly, leaving junk in `/tmp`.
`trap` allows you to define a function to run **before** exiting.

```bash
cleanup() {
    echo "[!] Caught Exit Signal! Cleaning up..."
    rm -f /tmp/temp_scan_results
}

trap cleanup EXIT
```
Now, whether the script finishes normally, errors out (`set -e`), or is killed (Ctrl+C), `cleanup` runs.

## 4. Hacker Context: OpSec

### 4.1 Trace Warning
**Never** leave `set -x` on in a script that handles passwords.
`+ echo "Logging in with Password123"` -> Logs the password to Stderr (which might be logged to a file).

### 4.2 Cleanup
Red Team scripts must clean up. Use `trap` to ensure your dropped payloads (`/tmp/shell.elf`) are deleted even if your connection drops.

## 5. Practical Lab: The Robust Scanner

**Script: `robust.sh`**
```bash
#!/bin/bash

# Safety
set -u
set -e

# Cleanup Hook
cleanup() {
    echo "[*] Cleaning up temporary files..."
    rm -f output.tmp
}
trap cleanup EXIT

# Main
echo "Start..."
touch output.tmp

# Simulate Work
echo "Working..."
# echo $UNDEFINED_VAR # Uncomment to test set -u crash

# Simulate Error
# ls /nonexistent # Uncomment to test set -e crash

echo "Success."
```

## 6. Review Questions
1.  **Q**: Why use `set -u`?
    *   **A**: It prevents catastrophic errors like `rm -rf /$DIR`. If `$DIR` is empty (undefined), Bash sees `rm -rf /`. With `-u`, it stops and says "DIR: unbound variable".
2.  **Q**: Does `trap` catch `SIGKILL` (kill -9)?
    *   **A**: No. SIGKILL terminates the process immediately at the kernel level. It cannot be caught or ignored.

## 7. Next Steps
Our scripts are robust. Now let's expand our toolkit beyond basic Bash commands to powerful text processors. Go to [[03b_Bash_Scripting/18_Text_Processing_Essentials]].
