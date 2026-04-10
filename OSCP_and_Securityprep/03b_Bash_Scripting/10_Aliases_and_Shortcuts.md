
# Aliases and Shortcuts: Permanent Hacks

> **Executive Summary**: Why type `ls -la` when you can type `ll`? Aliases are custom shortcuts defined in your shell profile (`.bashrc` or `.zshrc`). For a pentester, aliases can automate complex Nmap flags, setup listeners, or manage VPN connections instantly.

## 1. Learning Objectives
By the end of this chapter, you will be able to:
- **Create Aliases**: Syntax `alias name='command'`.
- **Persist Them**: Edit `~/.bashrc`.
- **Useful Aliases**: Deploy a standard set of security aliases.
- **Functions vs Aliases**: Know when to use which.

## 2. Core Concepts: The Alias

### 2.1 Temporary Alias
Run in terminal:
```bash
alias c='clear'
```
Type `c`. Screen clears.
Disappears on reboot.

### 2.2 Permanent Alias
Add to `~/.bashrc` (Bash) or `~/.zshrc` (Kali Default).
```bash
echo "alias c='clear'" >> ~/.zshrc
source ~/.zshrc
```

## 3. Deep Dive: Pentester Aliases

### 3.1 Networking
```bash
alias myip="ip -br -c a"
alias ports="netstat -tulanp"
```

### 3.2 Tools
```bash
# HTTP Server (Current Dir)
alias www="python3 -m http.server 80"

# Quick Nmap (Fast scan)
alias scan="nmap -F -T4 -sC -sV"

# Update System
alias update="sudo apt update && sudo apt full-upgrade -y && sudo apt autoremove -y"
```

### 3.3 Safety
```bash
# Ask before delete
alias rm="rm -i"
alias mv="mv -i"
alias cp="cp -i"
```

## 4. Red Team Perspective

### 4.1 Functions as Aliases
Aliases don't handle arguments well. Use functions in `.zshrc` for complex shortcuts.
**Example**: `mkcd` (Make dir and cd into it).
```bash
mkcd() {
    mkdir -p "$1"
    cd "$1"
}
```

### 4.2 The "Backdoor" Alias
If you compromise a user's `.bashrc`:
`alias sudo='my_script_that_steals_password_then_runs_sudo'`
User types `sudo apt update`. They get a password prompt (your script). You log it. You run real sudo. Transparent.

## 5. Practical Lab: Configuring Your Environment

### Scenario: Setup Kali
**Step 1: Open Config**
`nano ~/.zshrc`

**Step 2: Add Aliases**
```bash
# My Ops
alias ll='ls -la'
alias serve='python3 -m http.server'
alias grab='wget -r -l1 -np' # Grab all files in dir
alias tun0='ip addr show tun0 | grep "inet " | awk "{print \$2}" | cut -d/ -f1'
```

**Step 3: Apply**
`source ~/.zshrc`

**Step 4: Verify**
Run `tun0` (If VPN connected).

## 6. References
- [[01_Foundations/03_Kali_Linux_Deep_Dive]]
