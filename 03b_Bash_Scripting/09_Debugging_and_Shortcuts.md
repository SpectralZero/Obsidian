
# Debugging and Shortcuts: Efficient Hacking

> **Executive Summary**: You spend 90% of your time in the terminal. Efficiency matters. Knowing how to debug a broken script quickly (`set -x`) and how to navigate the command line (`Ctrl+R`) separates the pros from the struggle.

## 1. Learning Objectives
By the end of this chapter, you will be able to:
- **Debug Scripts**: Trace execution and find syntax errors.
- **ShellCheck**: Use static analysis to find bugs before running.
- **Keyboard Shortcuts**: Edit commands faster than you can type.
- **History**: Search past commands instantly.

## 2. Core Concepts: Debugging

### 2.1 Verbose Mode
Add `set -x` to the top of your script.
It prints every command *after* variable expansion.
```bash
#!/bin/bash
set -x
VAR="World"
echo "Hello $VAR"
```
Output:
`+ VAR=World`
`+ echo 'Hello World'`
`Hello World`

### 2.2 ShellCheck
The linter for Bash.
- **Online**: shellcheck.net
- **CLI**: `apt install shellcheck`.
- **Usage**: `shellcheck script.sh`.
- **Value**: Finds unquoted variables, logic errors, and portability issues.

## 3. Deep Dive: Terminal Shortcuts

| Shortcut | Action |
| :--- | :--- |
| `Ctrl + A` | Go to Start of line |
| `Ctrl + E` | Go to End of line |
| `Ctrl + U` | Clear from cursor to Start |
| `Ctrl + K` | Clear from cursor to End |
| `Ctrl + W` | Delete word backwards |
| `Ctrl + L` | Clear screen |
| `Ctrl + R` | Search History (Reverse) |
| `Alt + .` | Paste last argument of previous command |

## 4. Red Team Perspective

### 4.1 History Hiding
Sometimes you don't want your commands saved to `~/.bash_history`.
- **Space Prefix**: Start command with a space ` ls -la`. (If `HISTCONTROL=ignorespace`).
- **Clear**: `history -c`.
- **Disable**: `unset HISTFILE`.

### 4.2 Background Jobs
- `Ctrl + Z`: Pause current job.
- `bg`: Resume in background.
- `fg`: Bring to foreground.
- `jobs`: List jobs.
- **Use Case**: Start a reverse shell listener, pause it, check IP, resume listener.

## 5. Practical Lab: Fixing a Broken Script

### Scenario: The Buggy Script
**Broken Script**:
```bash
#!/bin/bash
TARGET = $1
if [ $TARGET == "127.0.0.1" ]
echo "Scanning localhost"
```

**Step 1: Run it**
`./buggy.sh` -> Error: `TARGET: command not found`.

**Step 2: ShellCheck**
`shellcheck buggy.sh`
- Error 1: Spaces around `=`.
- Error 2: Missing `then`.
- Error 3: Missing `fi`.
- Suggestion: Quote variables.

**Step 3: Fix**
```bash
#!/bin/bash
TARGET="$1"
if [ "$TARGET" == "127.0.0.1" ]; then
    echo "Scanning localhost"
fi
```

## 6. References
- [[01_Foundations/03_Kali_Linux_Deep_Dive]]
