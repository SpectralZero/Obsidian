# Final Project: The Recon Suite

---
title: "Capstone: The Recon Suite"
parent: "[[03b_Bash_Scripting]]"
tags:
  - bash
  - project
  - automation
  - recon
  - scanning
  - reporting
created: 2026-01-23
updated: 2026-01-23
difficulty: Expert
length: Very Long
---

> **Executive Summary**: 
> You have learned Variables, Loops, Functions, Traps, Network IO, and Text Processing. Now, we will combine them all into a single, professional-grade tool: **`recon_suite.sh`**.
>
> This script will:
> 1.  Take a target domain/IP.
> 2.  Validate input.
> 3.  Create a structured workspace.
> 4.  Perform Ping Sweeps (Parallel).
> 5.  Perform Port Scans (Native Bash).
> 6.  Grab Banners (`curl`).
> 7.  Generate a Markdown Report.
> 8.  Clean up gracefully on exit.

---

## Table of Contents

1. [Architecture & Design](#1-architecture--design)
2. [The Code: Section by Section](#2-the-code-section-by-section)
    - [2.1 Setup & Config](#21-setup--config)
    - [2.2 Helper Functions](#22-helper-functions)
    - [2.3 Core Logic: Scanning](#23-core-logic-scanning)
    - [2.4 Reporting](#24-reporting)
    - [2.5 Main Execution Flow](#25-main-execution-flow)
3. [Full Source Code](#3-full-source-code)
4. [Critical Analysis](#4-critical-analysis)

---

## 1. Architecture & Design

### The Goal
Automate the first 30 minutes of a pentest.

### The Features
- **Parallelism**: Use background jobs (`&`) for speed.
- **Resilience**: Handle Ctrl+C with traps.
- **Portability**: No `nmap` dependency (uses `/dev/tcp`).
- **Modularity**: Logic separated into functions.

### The Flow
`Start` -> `Check Args` -> `Setup Dir` -> `Ping Sweep` -> `Port Scan` -> `Web Recon` -> `Report` -> `Exit`.

---

## 2. The Code: Section by Section

### 2.1 Setup & Config
We use "Strict Mode" and global constants.

```bash
#!/usr/bin/env bash
set -euo pipefail
IFS=$'\n\t'

# Constants
readonly VERSION="1.0"
readonly WORKSPACE_ROOT="./recon_workspace"
readonly PORTS=(21 22 80 443 445 3306 8080) # Top ports
declare -A HOSTS_ALIVE
```

### 2.2 Helper Functions
Logging and Cleanup.

```bash
log() {
    local level=$1
    shift
    local msg="$@"
    local color=""
    
    case $level in
        INFO) color="\e[32m" ;; # Green
        WARN) color="\e[33m" ;; # Yellow
        ERROR) color="\e[31m" ;; # Red
    esac
    
    echo -e "${color}[${level}]\e[0m ${msg}"
}

cleanup() {
    log WARN "Interrupted! Cleaning up jobs..."
    jobs -p | xargs -r kill
    exit 1
}
trap cleanup SIGINT SIGTERM
```

### 2.3 Core Logic: Scanning

**Ping Sweep**:
```bash
ping_sweep() {
    local subnet=$1
    log INFO "Starting Ping Sweep on ${subnet}.0/24..."
    
    for i in {1..254}; do
        (
            local ip="${subnet}.${i}"
            if timeout 1 bash -c "echo >/dev/tcp/$ip/80" 2>/dev/null; then
                echo "$ip" >> "$SCOPE_DIR/alive.txt"
            elif ping -c 1 -W 1 "$ip" &>/dev/null; then
                echo "$ip" >> "$SCOPE_DIR/alive.txt"
            fi
        ) &
        
        # Limit parallelism to 50 jobs
        if [[ $(jobs -r -p | wc -l) -ge 50 ]]; then
            wait -n
        fi
    done
    wait
}
```

**Port Scan**:
```bash
port_scan() {
    local target=$1
    log INFO "Scanning ports on $target..."
    
    for port in "${PORTS[@]}"; do
        timeout 1 bash -c "echo >/dev/tcp/$target/$port" 2>/dev/null && \
            echo "$port" >> "$SCOPE_DIR/${target}_ports.txt" &
    done
    wait
}
```

### 2.4 Reporting
Generating Markdown.

```bash
generate_report() {
    local report_file="$SCOPE_DIR/report.md"
    echo "# Recon Report: $TARGET" > "$report_file"
    echo "## Live Hosts" >> "$report_file"
    
    while read -r ip; do
        echo "- **$ip**" >> "$report_file"
        if [[ -f "$SCOPE_DIR/${ip}_ports.txt" ]]; then
            echo "  - Open Ports: $(tr '\n' ', ' < "$SCOPE_DIR/${ip}_ports.txt")" >> "$report_file"
        fi
    done < "$SCOPE_DIR/alive.txt"
    
    log INFO "Report generated at $report_file"
}
```

---

## 3. Full Source Code

Copy this into `recon_suite.sh`.

```bash
#!/usr/bin/env bash
# Recon Suite v1.0
# Author: OpenCode
# Description: Automated native Bash reconnaissance tool.

set -euo pipefail
IFS=$'\n\t'

# --- Configuration ---
readonly VERSION="1.0"
readonly PORTS=(21 22 23 25 53 80 110 139 443 445 1433 3306 3389 5900 8080 8443)
readonly DATE=$(date +%Y%m%d_%H%M)

# --- Helpers ---
usage() {
    echo "Usage: $0 <target_subnet_prefix>"
    echo "Example: $0 192.168.1"
    exit 1
}

log() {
    local level=$1
    shift
    echo -e "[$(date +%T)] [$level] $@"
}

cleanup() {
    echo ""
    log WARN "Trapped Signal. Killing child processes..."
    jobs -p | xargs -r kill
}
trap cleanup SIGINT SIGTERM

# --- Core Functions ---

setup_workspace() {
    local target=$1
    SCOPE_DIR="./recon_${target}_${DATE}"
    mkdir -p "$SCOPE_DIR"
    log INFO "Workspace created: $SCOPE_DIR"
}

check_dependencies() {
    # We are using native bash, so dependencies are minimal.
    # But let's check for 'curl' just in case.
    if ! command -v curl &>/dev/null; then
        log ERROR "Curl is required for web recon."
        exit 1
    fi
}

ping_sweep() {
    local subnet=$1
    log INFO "Sweeping ${subnet}.0/24 for live hosts..."
    
    for i in {1..254}; do
        local ip="${subnet}.${i}"
        (
            # Try Ping first
            if ping -c 1 -W 1 "$ip" &>/dev/null; then
                echo "$ip" >> "$SCOPE_DIR/hosts_alive.txt"
                log INFO "Found: $ip (ICMP)"
            # Fallback to TCP/80 check if ping fails
            elif timeout 1 bash -c "echo >/dev/tcp/$ip/80" 2>/dev/null; then
                echo "$ip" >> "$SCOPE_DIR/hosts_alive.txt"
                log INFO "Found: $ip (TCP/80)"
            fi
        ) &
        
        # Batch processing to prevent FD exhaustion
        if [[ $(jobs -r -p | wc -l) -ge 50 ]]; then
            wait -n
        fi
    done
    wait
}

port_scan_target() {
    local ip=$1
    log INFO "Scanning TCP Ports on $ip..."
    
    for port in "${PORTS[@]}"; do
        (
            if timeout 1 bash -c "echo >/dev/tcp/$ip/$port" 2>/dev/null; then
                echo "$port" >> "$SCOPE_DIR/${ip}_ports.txt"
            fi
        ) &
    done
    wait
}

web_recon() {
    local ip=$1
    local port=$2
    
    # Simple banner grab
    local banner=$(curl -s -I --connect-timeout 2 "http://${ip}:${port}" | head -n 1 | tr -d '\r')
    if [[ -n "$banner" ]]; then
        echo "$port: $banner" >> "$SCOPE_DIR/${ip}_web.txt"
    fi
}

generate_report() {
    local outfile="$SCOPE_DIR/README.md"
    
    echo "# Recon Report" > "$outfile"
    echo "Date: $DATE" >> "$outfile"
    echo "Target: ${TARGET}.0/24" >> "$outfile"
    echo "---" >> "$outfile"
    
    if [[ ! -f "$SCOPE_DIR/hosts_alive.txt" ]]; then
        log WARN "No hosts found."
        echo "No hosts found." >> "$outfile"
        return
    fi
    
    while read -r ip; do
        echo "## Host: $ip" >> "$outfile"
        
        if [[ -f "$SCOPE_DIR/${ip}_ports.txt" ]]; then
            echo "### Open Ports" >> "$outfile"
            echo "\`\`\`" >> "$outfile"
            cat "$SCOPE_DIR/${ip}_ports.txt" >> "$outfile"
            echo "\`\`\`" >> "$outfile"
        fi
        
        if [[ -f "$SCOPE_DIR/${ip}_web.txt" ]]; then
            echo "### Web Headers" >> "$outfile"
            echo "\`\`\`" >> "$outfile"
            cat "$SCOPE_DIR/${ip}_web.txt" >> "$outfile"
            echo "\`\`\`" >> "$outfile"
        fi
        
    done < "$SCOPE_DIR/hosts_alive.txt"
    
    log INFO "Final Report saved to $outfile"
}

# --- Main ---

main() {
    if [[ $# -ne 1 ]]; then
        usage
    fi
    
    TARGET=$1
    
    check_dependencies
    setup_workspace "$TARGET"
    
    ping_sweep "$TARGET"
    
    if [[ -f "$SCOPE_DIR/hosts_alive.txt" ]]; then
        log INFO "Starting detailed scans on live hosts..."
        
        while read -r ip; do
            port_scan_target "$ip"
            
            # If ports found, check for Web
            if [[ -f "$SCOPE_DIR/${ip}_ports.txt" ]]; then
                while read -r port; do
                    if [[ "$port" == "80" || "$port" == "8080" || "$port" == "443" ]]; then
                        web_recon "$ip" "$port"
                    fi
                done < "$SCOPE_DIR/${ip}_ports.txt"
            fi
            
        done < "$SCOPE_DIR/hosts_alive.txt"
    else
        log WARN "No live hosts found. Exiting."
        exit 0
    fi
    
    generate_report
    log INFO "All tasks completed successfully."
}

main "$@"
```

---

## 4. Critical Analysis

### Strengths
1.  **Dependency-Free**: Uses native Bash sockets for scanning. Works on compromised boxes without `nmap`.
2.  **Modular**: Easy to add a "Vulnerability Scan" function later.
3.  **Parallel**: Scans a /24 in seconds, not minutes.

### Weaknesses
1.  **Accuracy**: `/dev/tcp` connect scanning is not as reliable as Nmap SYN scanning.
2.  **Noise**: This is a noisy scanner. It will light up logs.
3.  **Error Handling**: If `curl` hangs, the script might wait (fixed with `timeout` command).

### Final Thoughts
This script represents the culmination of your Bash knowledge. It uses every concept: Shebang, Loops, Logic, Redirection, Functions, Arrays, and Networking.

---
*End of Module 03b*
