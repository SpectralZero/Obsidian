
# Positional Parameters: Handling Arguments

> **Executive Summary**: Interactive scripts (using `read`) are fine for humans, but bad for automation. Professional tools accept arguments (`./scan.sh 10.10.10.10`) so they can be chained in pipes or scheduled via cron. This chapter explains `$1`, `$2`, `$@`, and how to build robust Command Line Interfaces (CLI).

## 1. Learning Objectives
By the end of this chapter, you will be able to:
- **Use Parameters**: Access `$1` through `$9`.
- **Count Args**: Use `$#` to validate input.
- **Use All Args**: Use `$@` to iterate over lists of arguments.
- **Shift**: Process arguments sequentially.

## 2. Core Concepts: The Variables

| Variable | Meaning | Example (`./script.sh A B`) |
| :--- | :--- | :--- |
| `$0` | The name of the script | `./script.sh` |
| `$1` | First argument | `A` |
| `$2` | Second argument | `B` |
| `$#` | Total number of arguments | `2` |
| `$@` | All arguments list | `"A" "B"` |
| `$*` | All arguments string | `"A B"` (Rarely used) |

## 3. Deep Dive: Validating Input

Always check if the user provided arguments before trying to use them.

```bash
if [[ $# -eq 0 ]]; then
    echo "Usage: $0 <target_ip>"
    exit 1
fi

TARGET=$1
echo "Scanning $TARGET..."
```

## 4. Deep Dive: Iterating Arguments (`$@`)

If you want a script that takes *any* number of IPs and scans them:
`./scan.sh 10.1 10.2 10.3 ...`

```bash
echo "Scanning $# targets..."

for IP in "$@"; do
    echo "Target: $IP"
    # scan logic here
done
```

## 5. Advanced: The `shift` Command
`shift` removes `$1` from the list and moves everything down. `$2` becomes `$1`, `$3` becomes `$2`.
Useful for processing flags (`-p 80`).

```bash
# ./script.sh -p 80 10.10.10.10

while [[ $# -gt 0 ]]; do
    key="$1"
    case $key in
        -p|--port)
            PORT="$2"
            shift # Remove key
            shift # Remove value
            ;;
        *)
            TARGET="$1" # Assume it's the target
            shift
            ;;
    esac
done

echo "Target: $TARGET, Port: $PORT"
```

## 6. Hacker Context: Wrappers

You often write "Wrappers" around complex tools.
**Example**: A script that runs `gobuster` but with your favorite wordlist and extensions pre-configured.
```bash
# go.sh
# Usage: ./go.sh http://url
gobuster dir -u $1 -w /usr/share/seclists/Discovery/Web-Content/common.txt -x php,txt,html,sh
```

## 7. Practical Lab: The Multi-Pinger

**Script: `pinger.sh`**
Accepts any number of IPs as arguments. Pings them.

```bash
#!/bin/bash

# Check args
if [[ $# -eq 0 ]]; then
    echo "Usage: $0 <ip1> [ip2] [ip3]..."
    exit 1
fi

echo "Starting Ping Sweep on $# hosts..."

for HOST in "$@"; do
    if ping -c 1 -W 1 "$HOST" &> /dev/null; then
        echo "[+] $HOST is UP"
    else
        echo "[-] $HOST is Down"
    fi
done
```

**Run**:
`./pinger.sh 192.168.1.1 192.168.1.50 google.com`

## 8. Review Questions
1.  **Q**: What is the difference between `$@` and `$*`?
    *   **A**: `"$@"` treats arguments as separate strings (`"A" "B"`). `"$*"` treats them as one single string (`"A B"`). Always use `"$@"` for loops.
2.  **Q**: If I run `./script.sh a b c`, what is `$2`?
    *   **A**: `b`.

## 9. Next Steps
We can handle lists of arguments. Now let's learn about Arrays to store data inside the script. Go to [[03b_Bash_Scripting/13_Indexed_Arrays]].
