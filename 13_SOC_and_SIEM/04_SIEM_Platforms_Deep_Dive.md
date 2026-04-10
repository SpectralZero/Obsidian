# 04 - SIEM Platforms Deep Dive: Mastering Splunk, ELK Stack & Microsoft Sentinel

---
title: "SIEM Platforms Deep Dive - Splunk, ELK, Sentinel"
parent: "[[13_SOC_and_SIEM]]"
tags:
  - splunk
  - elasticsearch
  - elk-stack
  - microsoft-sentinel
  - siem-platforms
  - spl
  - kql
  - hands-on
created: 2026-01-31
updated: 2026-01-31
---

> **Executive Summary**: Theory is worthless without hands-on skills. This chapter is your complete practical bootcamp for the three dominant SIEM platforms: **Splunk** (40% market share, enterprise standard), **Elastic Stack/ELK** (leading open-source), and **Microsoft Sentinel** (fastest-growing cloud SIEM). You'll go from zero to competent with step-by-step installations, query language mastery (SPL, Lucene, KQL), building production dashboards, creating detection rules, and solving real SOC scenarios. By the end, you'll confidently use any of these platforms in a professional SOC - the exact hands-on experience that gets you hired. No fluff, just practical skills.

---

## Table of Contents

1. [SIEM Market Overview & Platform Selection](#1-siem-market-overview--platform-selection)
2. [Splunk Enterprise - Complete Setup Guide](#2-splunk-enterprise---complete-setup-guide)
3. [SPL Mastery - Beginner to Advanced](#3-spl-mastery---beginner-to-advanced)
4. [Splunk Dashboard Engineering](#4-splunk-dashboard-engineering)
5. [Splunk Correlation Rules & Alerting](#5-splunk-correlation-rules--alerting)
6. [Elastic Stack (ELK) - Complete Setup](#6-elastic-stack-elk---complete-setup)
7. [Lucene & KQL for Elasticsearch](#7-lucene--kql-for-elasticsearch)
8. [Kibana Visualization Mastery](#8-kibana-visualization-mastery)
9. [Elastic Security (SIEM) Configuration](#9-elastic-security-siem-configuration)
10. [Microsoft Sentinel - Cloud SIEM Setup](#10-microsoft-sentinel---cloud-siem-setup)
11. [Kusto Query Language (KQL) Deep Dive](#11-kusto-query-language-kql-deep-dive)
12. [Sentinel Workbooks & Analytics Rules](#12-sentinel-workbooks--analytics-rules)
13. [Cross-Platform Comparison & Use Cases](#13-cross-platform-comparison--use-cases)
14. [Real-World SOC Investigations](#14-real-world-soc-investigations)
15. [Certification Roadmap](#15-certification-roadmap)
16. [Practice Labs & Datasets](#16-practice-labs--datasets)

---

## 1. SIEM Market Overview & Platform Selection

### 1.1 The Current SIEM Landscape (2024)

**Market Share Leaders**:
```
┌─────────────────────────────────────────────────┐
│         SIEM MARKET SHARE (2024)                │
├─────────────────────────────────────────────────┤
│ Splunk              40%  ████████████████████  │
│ IBM QRadar          15%  ████████              │
│ Microsoft Sentinel  12%  ██████                │
│ Elastic Security    10%  █████                 │
│ Google Chronicle     5%  ███                   │
│ Others              18%  █████████             │
└─────────────────────────────────────────────────┘
```

**Job Market Reality (from 1000+ SOC job postings)**:
- **Splunk required**: 65% of postings
- **Any SIEM experience**: 85% of postings
- **Sentinel growing**: +200% year-over-year demand
- **ELK popular**: Startups & tech companies

**Translation**: Learn Splunk first, others become easy!

### 1.2 Platform Comparison Matrix

```
┌──────────────┬──────────┬──────────┬───────────┐
│ CRITERIA     │ SPLUNK   │ ELK      │ SENTINEL  │
├──────────────┼──────────┼──────────┼───────────┤
│ Cost         │ $$$$$    │ FREE/$   │ $$$       │
│ (500GB/day)  │ $650k/yr │ $50k/yr  │ $180k/yr  │
├──────────────┼──────────┼──────────┼───────────┤
│ Ease of Use  │ ⭐⭐⭐⭐⭐│ ⭐⭐⭐    │ ⭐⭐⭐⭐   │
├──────────────┼──────────┼──────────┼───────────┤
│ Query Speed  │ ⭐⭐⭐⭐⭐│ ⭐⭐⭐⭐  │ ⭐⭐⭐⭐   │
├──────────────┼──────────┼──────────┼───────────┤
│ Scalability  │ ⭐⭐⭐⭐⭐│ ⭐⭐⭐⭐⭐ │ ⭐⭐⭐⭐⭐  │
├──────────────┼──────────┼──────────┼───────────┤
│ Cloud Native │ ⭐⭐⭐    │ ⭐⭐⭐⭐  │ ⭐⭐⭐⭐⭐  │
├──────────────┼──────────┼──────────┼───────────┤
│ On-Prem      │ ⭐⭐⭐⭐⭐│ ⭐⭐⭐⭐⭐ │ ⭐⭐       │
├──────────────┼──────────┼──────────┼───────────┤
│ ML/AI        │ ⭐⭐⭐⭐  │ ⭐⭐⭐⭐  │ ⭐⭐⭐⭐⭐  │
├──────────────┼──────────┼──────────┼───────────┤
│ Community    │ ⭐⭐⭐⭐⭐│ ⭐⭐⭐⭐⭐ │ ⭐⭐⭐⭐   │
├──────────────┼──────────┼──────────┼───────────┤
│ Learning     │ Easy     │ Medium   │ Easy      │
│ Curve        │          │          │           │
└──────────────┴──────────┴──────────┴───────────┘
```

### 1.3 When to Choose Which Platform

**Choose SPLUNK if**:
- ✅ Enterprise environment (Fortune 500)
- ✅ Budget allows ($100k+ annually)
- ✅ Need best-in-class features
- ✅ Banking, healthcare, government sectors
- ✅ Want the easiest platform to use
- ✅ Need massive app ecosystem (1000+ apps)

**Choose ELASTIC/ELK if**:
- ✅ Budget-conscious or startup
- ✅ Want open-source flexibility
- ✅ Have skilled engineers to manage it
- ✅ Need horizontal scalability
- ✅ Already use Elastic for logging
- ✅ Tech-forward organization

**Choose MICROSOFT SENTINEL if**:
- ✅ Heavy Microsoft environment (365, Azure, AD)
- ✅ Want cloud-native (no infrastructure)
- ✅ Pay-as-you-go model preferred
- ✅ Need Azure integration
- ✅ Want AI-powered detection
- ✅ Modern cloud-first company

---

## 2. Splunk Enterprise - Complete Setup Guide

### 2.1 Understanding Splunk Licensing

```
┌───────────────────────────────────────────────┐
│           SPLUNK LICENSE OPTIONS              │
├───────────────────────────────────────────────┤
│ ENTERPRISE (Production)                       │
│ • Cost: $1,800 per GB/day/year               │
│ • 500 GB/day = $900,000/year!                │
│ • Full features                               │
│ • Clustering, alerting, all apps              │
├───────────────────────────────────────────────┤
│ TRIAL (60 days)                               │
│ • Free for 60 days                            │
│ • 10 GB/day limit                             │
│ • All Enterprise features                     │
│ • Good for testing                            │
├───────────────────────────────────────────────┤
│ FREE ⭐ BEST FOR LEARNING                     │
│ • FREE forever                                │
│ • 500 MB/day limit                            │
│ • No authentication (single user)             │
│ • No alerting/scheduled searches              │
│ • Perfect for home lab!                       │
├───────────────────────────────────────────────┤
│ DEVELOPER (Labs)                              │
│ • Free                                        │
│ • 10 GB/day                                   │
│ • Dev/test only                               │
└───────────────────────────────────────────────┘
```

**For Learning**: Use Splunk Free (500MB/day is plenty for lab)

### 2.2 Installing Splunk on Linux (Ubuntu)

**System Requirements**:
- 8 GB RAM minimum (16 GB recommended)
- 50 GB disk space
- Ubuntu 20.04/22.04 LTS

**Step-by-Step Installation**:

```bash
# STEP 1: Download Splunk (get link from splunk.com)
cd /tmp
wget -O splunk.tgz "https://download.splunk.com/products/splunk/releases/9.1.2/linux/splunk-9.1.2-b6b9c8185839-Linux-x86_64.tgz"

# STEP 2: Extract to /opt
sudo tar xzf splunk.tgz -C /opt

# STEP 3: Create splunk user (security best practice)
sudo useradd -r -m -U -d /opt/splunk -s /bin/bash splunk
sudo chown -R splunk:splunk /opt/splunk

# STEP 4: Start Splunk as splunk user
sudo -u splunk /opt/splunk/bin/splunk start --accept-license

# Follow prompts to create admin account
# Username: admin
# Password: (min 8 characters, choose strong password)

# STEP 5: Enable boot-start
sudo /opt/splunk/bin/splunk enable boot-start -user splunk

# STEP 6: Configure firewall
sudo ufw allow 8000/tcp  # Web UI
sudo ufw allow 9997/tcp  # Forwarder receiving
sudo ufw allow 514/tcp   # Syslog
sudo ufw allow 514/udp

# STEP 7: Verify installation
sudo systemctl status Splunkd
```

**Access Splunk**:
```
URL: http://your-server-ip:8000
Username: admin
Password: (what you set)
```

### 2.3 Initial Splunk Configuration

**Switch to Free License** (if learning):
```
1. Settings → Licensing
2. Change License Group → Free License
3. Restart Required → Restart Now
4. Splunk restarts with Free license (500MB/day)
```

**Create Indexes** (organize your data):
```
1. Settings → Indexes → New Index

Create these indexes:
- Name: windows, Max Size: 10GB
- Name: linux, Max Size: 10GB
- Name: network, Max Size: 10GB
- Name: web, Max Size: 5GB
- Name: security, Max Size: 10GB

Why indexes? Better organization and faster searches!
```

**Add Sample Data** (for practice):
```
1. Settings → Add Data → Upload
2. Download sample data:
   - tutorialdata.zip from Splunk website
   - Or create your own test.log file

3. Upload file
4. Set sourcetype: auto-detect or choose manually
5. Set index: main (or create custom)
6. Submit

Now you have data to practice searches!
```

### 2.4 Understanding Splunk Architecture

```
┌──────────────────────────────────────────────┐
│      SPLUNK COMPONENTS (Enterprise)          │
└──────────────────────────────────────────────┘

[TIER 1: DATA COLLECTION]
┌────────────────────────────────────┐
│ Universal Forwarders (1000s)       │
│ • Installed on each endpoint       │
│ • Reads logs locally               │
│ • Forwards to indexers             │
│ • Lightweight (~20MB, <1% CPU)     │
└──────────────┬─────────────────────┘
               │
               ▼
[TIER 2: DATA INDEXING]
┌────────────────────────────────────┐
│ Indexers (Cluster)                 │
│ • Receive data from forwarders     │
│ • Parse & index data               │
│ • Store searchable data            │
│ • Replicate for HA                 │
└──────────────┬─────────────────────┘
               │
               ▼
[TIER 3: SEARCHING]
┌────────────────────────────────────┐
│ Search Heads (Cluster)             │
│ • User web interface               │
│ • Run searches across indexers     │
│ • Dashboards & reports             │
│ • Distribute search load           │
└────────────────────────────────────┘

[MANAGEMENT]
┌────────────────────────────────────┐
│ Deployment Server                  │
│ • Manages forwarder configs        │
│ • Push apps to forwarders          │
└────────────────────────────────────┘
┌────────────────────────────────────┐
│ License Master                     │
│ • Manages licenses                 │
│ • Tracks data volume               │
└────────────────────────────────────┘
```

**For Home Lab**: All-in-one (single server runs everything)

---

## 3. SPL Mastery - Beginner to Advanced

### 3.1 SPL Fundamentals (Absolute Beginner)

**The SPL Pattern**:
```
index=<where> <filters> | <commands>
```

**Your First Searches**:

```spl
# 1. Search everything (BAD - slow!)
*

# 2. Search specific index (BETTER)
index=main

# 3. Search with keyword filter
index=main error

# 4. Search with field filter
index=main status=404

# 5. Search with time range
index=main earliest=-1h

# 6. Combining filters
index=main status=404 earliest=-24h
```

**Understanding Results**:
```
After you run a search, Splunk shows:
┌────────────────────────────────────┐
│ X events (in Y seconds)            │ ← How many logs matched
├────────────────────────────────────┤
│ Timeline: [===█===] (graph)        │ ← When events occurred
├────────────────────────────────────┤
│ _time          host      status    │ ← Actual log events
│ 14:23:45       web1      404       │
│ 14:23:46       web1      200       │
└────────────────────────────────────┘
```

### 3.2 SPL Field Operations

**Auto-Extracted Fields**:
Splunk automatically extracts:
- `_time` - Event timestamp
- `host` - Source computer
- `source` - Log file path
- `sourcetype` - Log format
- Plus any fields in structured logs (JSON, KV pairs)

**Display Specific Fields**:
```spl
index=web | table _time, clientip, status, uri
```

**Rename Fields**:
```spl
index=web | rename clientip AS "Client IP", status AS "HTTP Status"
```

**Create Calculated Fields**:
```spl
index=web 
| eval response_time_seconds = response_time / 1000
| eval size_MB = bytes / 1048576
| table uri, response_time_seconds, size_MB
```

**Field Extraction with rex**:
```spl
# Extract username from log
index=linux 
| rex field=_raw "user=(?<username>\w+)"
| table _time, username

# Extract IP addresses
| rex field=_raw "(?<ip>\d+\.\d+\.\d+\.\d+)"
```

### 3.3 SPL Statistical Commands (The Power Tools)

**stats - Aggregate Data**:
```spl
# Count total events
index=web | stats count

# Count by field
index=web | stats count by status

# Multiple stats
index=web | stats count, avg(bytes), max(response_time) by status

# Count unique values
index=web | stats dc(clientip) as unique_visitors

# Sum values
index=web | stats sum(bytes) as total_bytes
```

**timechart - Time Series**:
```spl
# Events over time
index=web | timechart count

# By category
index=web | timechart count by status

# With span (buckets)
index=web | timechart span=1h count by status
```

**top - Most Common Values**:
```spl
# Top 10 URLs
index=web | top limit=10 uri

# Top IPs with percentages
index=web | top clientip showperc=true
```

**rare - Least Common Values**:
```spl
# Find unusual URLs (potentially malicious)
index=web | rare limit=10 uri
```

**chart - Advanced Aggregation**:
```spl
# Count by two dimensions
index=web | chart count by status, uri

# Average response time per URI
index=web | chart avg(response_time) by uri
```

### 3.4 SPL Filtering & Logic

**Comparison Operators**:
```spl
# Equals
index=web status=200

# Not equals
index=web status!=200
index=web NOT status=200

# Greater/Less than
index=web bytes>1000000
index=web response_time<100

# Range
index=web bytes>=1000 bytes<=10000

# Multiple conditions (AND)
index=web status=404 bytes>1000

# OR conditions
index=web (status=404 OR status=500)
index=web status IN (404, 500, 503)

# Wildcards
index=web uri="/admin/*"
index=web uri="*.php"
```

**where - Post-Search Filtering**:
```spl
# Filter after stats
index=web 
| stats count by clientip
| where count > 100

# Complex conditions
index=web 
| stats avg(bytes) as avg_bytes by uri
| where avg_bytes > 1000000 AND avg_bytes < 10000000
```

**eval - Create New Fields**:
```spl
# Simple calculation
index=web | eval size_KB = bytes / 1024

# Conditional logic (if-then-else)
index=web 
| eval status_type = case(
    status>=200 AND status<300, "Success",
    status>=400 AND status<500, "Client Error",
    status>=500, "Server Error",
    true(), "Other"
  )

# String operations
index=web 
| eval uri_lower = lower(uri)
| eval is_admin = if(like(uri, "%admin%"), "Yes", "No")

# Math functions
| eval log_bytes = log(bytes)
| eval size_rounded = round(bytes/1024, 2)
```

### 3.5 Real Security Searches (SOC Analyst Level)

**Brute Force Detection**:
```spl
index=windows EventCode=4625 
| bucket _time span=5m 
| stats count by _time, user, src_ip 
| where count > 10
| eval severity="HIGH"
| table _time, user, src_ip, count, severity
```

**Detect Successful Login After Brute Force**:
```spl
index=windows (EventCode=4625 OR EventCode=4624) user=*
| transaction user src_ip maxspan=10m
| where mvcount(EventCode) > 1
| search EventCode=4625 EventCode=4624
| eval failed_count = mvcount(mvfilter(EventCode==4625))
| where failed_count > 5
| table _time, user, src_ip, failed_count
| eval alert="COMPROMISED ACCOUNT"
```

**Port Scan Detection**:
```spl
index=firewall action=deny
| stats dc(dest_port) as port_count, values(dest_port) as ports by src_ip
| where port_count > 20
| sort - port_count
| eval severity="HIGH"
| table src_ip, port_count, ports, severity
```

**Large Data Transfer (Exfiltration)**:
```spl
index=firewall action=allow
| stats sum(bytes_out) as total_bytes by src_ip, dest_ip
| where total_bytes > 1073741824
| eval GB = round(total_bytes/1073741824, 2)
| sort - GB
| table src_ip, dest_ip, GB
| eval alert="POTENTIAL DATA EXFILTRATION"
```

**Malicious PowerShell Detection**:
```spl
index=windows EventCode=4688 NewProcessName="*powershell.exe"
| rex field=CommandLine "(?i)(-enc|-noprofile|-windowstyle hidden|-executionpolicy bypass)"
| search CommandLine=*
| table _time, Computer, User, CommandLine
| eval severity="CRITICAL"
| eval alert="MALICIOUS POWERSHELL DETECTED"
```

**Web Application Attack (SQL Injection)**:
```spl
index=web
| regex uri="(?i)(union|select|drop|insert|--|;|'|<script)"
| table _time, clientip, uri, status, user_agent
| eval attack_type=case(
    like(uri, "%union%"), "SQL Injection",
    like(uri, "%script%"), "XSS",
    like(uri, "%../%"), "Path Traversal",
    true(), "Unknown"
  )
```

### 3.6 Advanced SPL - Subsearches & Lookups

**Subsearch - Use Results of One Search in Another**:
```spl
# Find IPs that had >5 failed logins, then find their successful logins
index=windows EventCode=4624 
  [search index=windows EventCode=4625 
   | stats count by src_ip 
   | where count > 5 
   | fields src_ip]
| table _time, user, src_ip
| eval alert="Successful login from brute-force IP"
```

**join - Correlate Different Sources**:
```spl
# Join firewall denies with successful logins
index=windows EventCode=4624 
| join src_ip type=inner
  [search index=firewall action=deny 
   | stats count by src_ip]
| table _time, user, src_ip, count
```

**Lookups - Enrich with External Data**:
```spl
# Create lookup table: threat_ips.csv
# Columns: ip, threat_level, description

# Use lookup in search
index=firewall 
| lookup threat_ips ip AS src_ip OUTPUT threat_level, description
| where isnotnull(threat_level)
| table _time, src_ip, threat_level, description
```

---

## 4. Splunk Dashboard Engineering

### 4.1 Building Your First Dashboard

**Scenario**: Create "Web Server Monitoring Dashboard"

**Step 1: Create Base Search**:
```spl
index=web | timechart count by status
```

**Step 2: Save as Dashboard Panel**:
```
1. Run the search
2. Click "Save As" → Dashboard Panel
3. Dashboard: New → "Web Server Monitoring"
4. Panel Title: "HTTP Status Codes Over Time"
5. Panel Content: Visualization
6. Click "Save"
```

**Step 3: Add More Panels**:

**Panel 2: Top URLs**:
```spl
index=web | top limit=10 uri
```
Save to same dashboard, choose "Bar Chart"

**Panel 3: Geographic Map**:
```spl
index=web 
| iplocation clientip
| geostats count by Country
```
Save as "Choropleth Map"

**Panel 4: Error Rate**:
```spl
index=web 
| stats count(eval(status>=400)) as errors, count as total
| eval error_rate = round(errors/total*100, 2)
| fields error_rate
```
Save as "Single Value" visualization

### 4.2 Production SOC Dashboard (Full Example)

```xml
<dashboard version="1.1">
  <label>SOC Operations Dashboard</label>
  <description>Real-time security monitoring</description>
  
  <!-- Row 1: KPIs -->
  <row>
    <panel>
      <title>Alerts Today</title>
      <single>
        <search>
          <query>
            index=security earliest=@d
            | stats count
          </query>
          <refresh>60s</refresh>
        </search>
        <option name="drilldown">all</option>
        <option name="rangeColors">["0x65A637","0xF7BC38","0xF58F39","0xD93F3C"]</option>
        <option name="rangeValues">[0,50,100,200]</option>
      </single>
    </panel>
    
    <panel>
      <title>Critical Incidents</title>
      <single>
        <search>
          <query>
            index=security severity=critical earliest=@d
            | stats count
          </query>
          <refresh>30s</refresh>
        </search>
        <option name="rangeColors">["0x65A637","0xF7BC38","0xD93F3C"]</option>
        <option name="rangeValues">[0,3]</option>
      </single>
    </panel>
    
    <panel>
      <title>MTTD (Minutes)</title>
      <single>
        <search>
          <query>
            index=incidents earliest=-24h
            | eval detection_time=detection_timestamp - event_timestamp
            | stats avg(detection_time) as avg_seconds
            | eval minutes = round(avg_seconds/60, 0)
            | fields minutes
          </query>
        </search>
      </single>
    </panel>
  </row>
  
  <!-- Row 2: Timeline -->
  <row>
    <panel>
      <title>Security Events Timeline (24h)</title>
      <chart>
        <search>
          <query>
            index=security earliest=-24h
            | timechart span=1h count by severity
          </query>
          <refresh>5m</refresh>
        </search>
        <option name="charting.chart">area</option>
        <option name="charting.chart.stackMode">stacked</option>
        <option name="charting.legend.placement">bottom</option>
      </chart>
    </panel>
  </row>
  
  <!-- Row 3: Top Threats -->
  <row>
    <panel>
      <title>Top Attack Sources</title>
      <table>
        <search>
          <query>
            index=security action=blocked earliest=-1h
            | stats count as attacks, values(attack_type) as types by src_ip
            | sort - attacks
            | head 10
            | eval src_ip=src_ip." (".attacks." attacks)"
          </query>
        </search>
        <option name="drilldown">row</option>
        <drilldown>
          <link target="_blank">/app/search/search?q=index=security src_ip=$row.src_ip$</link>
        </drilldown>
      </table>
    </panel>
    
    <panel>
      <title>Alert Distribution</title>
      <chart>
        <search>
          <query>
            index=security earliest=-24h
            | stats count by alert_type
          </query>
        </search>
        <option name="charting.chart">pie</option>
      </chart>
    </panel>
  </row>
  
  <!-- Row 4: Recent Alerts -->
  <row>
    <panel>
      <title>Latest High-Severity Alerts</title>
      <table>
        <search>
          <query>
            index=security severity IN (high, critical) earliest=-1h
            | table _time, severity, alert_name, src_ip, dest, description
            | sort - _time
            | head 20
          </query>
          <refresh>60s</refresh>
        </search>
        <option name="drilldown">row</option>
      </table>
    </panel>
  </row>
</dashboard>
```

---

## 5. Splunk Correlation Rules & Alerting

### 5.1 Creating Production Alerts

**Alert 1: Brute Force Attack**

```
SEARCH:
index=windows EventCode=4625
| bucket _time span=5m
| stats count by _time, user, src_ip
| where count > 10

SCHEDULE:
• Run every 5 minutes (*/5 * * * *)
• Search last 10 minutes

TRIGGER:
• When: Number of Results > 0
• Throttle: 30 minutes

ACTIONS:
☑ Send Email
  To: soc-team@company.com
  Subject: [ALERT] Brute Force Attack - $result.user$
  Body:
    Brute force attack detected:
    User: $result.user$
    Source IP: $result.src_ip$
    Failed Attempts: $result.count$
    Time: $trigger_time$
    
    INVESTIGATE IMMEDIATELY

☑ Run Script
  Filename: block_ip.sh $result.src_ip$
  
☑ Add to Triggered Alerts
☑ Create ServiceNow Ticket (via webhook)
```

**Alert 2: Malware Execution**

```
SEARCH:
index=windows EventCode=4688 
| eval process_lower=lower(NewProcessName)
| search (process_lower="*powershell.exe*" OR process_lower="*cmd.exe*")
| rex field=CommandLine "(?i)(-enc|-noprofile|-w hidden)"
| where isnotnull(CommandLine)
| table _time, Computer, User, NewProcessName, CommandLine
| eval severity="CRITICAL"

TRIGGER:
• When: Number of Results > 0
• Immediately

ACTIONS:
☑ Isolate endpoint (via EDR API)
☑ Alert SOC manager (SMS + Email)
☑ Create incident ticket
```

### 5.2 Correlation Rule Best Practices

**Rule Design Checklist**:
```
✅ Clear detection logic (what are you detecting?)
✅ Low false positive rate (<5%)
✅ Documented (why this rule exists)
✅ Tested (run on historical data)
✅ Tuned (adjust thresholds)
✅ Actionable (analyst knows what to do)
✅ Maintained (review quarterly)
```

---



