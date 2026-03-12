
---

# SIEM Integration & Log Ingestion Setup

## Overview

This section describes how logs from multiple machines in the SOC lab are collected and forwarded to the **Splunk Enterprise** running on the Ubuntu server.

Log sources include:

* Windows endpoint
* Ubuntu server logs
* pfSense firewall logs

Logs are forwarded using:

* **Splunk Universal Forwarder**
* Syslog (for firewall)

---

# Step 1 — Install Splunk Enterprise on Ubuntu (SIEM Server)

### Download

Download Splunk Enterprise for Linux from the official site.([⬅ Back to Download sources](../Virtualbox-setup.md))

Example package:

```
splunk-9.x.x-linux-amd64.deb
```

### Installation

```
sudo dpkg -i splunk-9.x.x-linux-amd64.deb
```

Start Splunk service:

```
sudo /opt/splunk/bin/splunk start
```

During first startup:

* Accept license
* Create admin username and password

### Access Splunk Web Interface

```
http://<Ubuntu-IP>:8000
```

Example:

```
http://192.168.56.40:8000
```

Purpose:

The Splunk server acts as:

* Indexer (stores logs)
* Search head (analysis & dashboards)

---

# Step 2 — Configure Splunk to Receive Logs

Splunk must listen for logs from forwarders.

Navigate in Splunk UI:

```
Settings → Forwarding and Receiving
```

Enable receiving on port:

```
9997
```

Purpose:

```
Forwarders → send logs → Splunk Indexer
```

---

# Step 3 — Install Splunk Universal Forwarder on Windows Endpoint

Download:

Splunk Universal Forwarder for Windows.

Example installer:

```
splunkforwarder-x.x.x-x64-release.msi
```

### Installation Steps

1. Run installer
2. Choose **Forwarder mode**
3. Enter Splunk server IP

Example:

```
192.168.56.40
```

Port:

```
9997
```

This registers the endpoint with the Splunk server.

---

# Step 4 — Install Sysmon on Windows Endpoint


## What Sysmon Is

Sysmon

### Definition

Sysmon is a **Windows system monitoring tool from Microsoft Sysinternals** that logs detailed system activity to the Windows Event Log.

### Purpose

It provides **high-fidelity telemetry** used by SOC analysts and detection engineers.

### Why We Need Sysmon Even Though Windows Has Logs

Windows native logs are **limited for threat detection**.

| Windows Native Logs | Sysmon Logs                        |
| ------------------- | ---------------------------------- |
| Login events        | Process creation with command line |
| Service start/stop  | Network connections per process    |
| System errors       | File creation monitoring           |
| Application logs    | Registry persistence detection     |

Example detection use cases:

| Attack Activity   | Sysmon Event                    |
| ----------------- | ------------------------------- |
| Malware execution | Event ID 1 (Process creation)   |
| C2 communication  | Event ID 3 (Network connection) |
| Persistence       | Event ID 13 (Registry change)   |
| File dropper      | Event ID 11 (File creation)     |

Location of Sysmon logs in Windows:

```
Event Viewer
Applications and Services Logs
Microsoft
Windows
Sysmon
Operational
```

---

# Installing Sysmon

Download from Microsoft Sysinternals.

Files extracted:

```
Sysmon.exe
Sysmon64.exe
```

Install Sysmon with configuration:

```
Sysmon64.exe -i sysmonconfig.xml
```

The XML config defines **which events Sysmon logs**.

---

# Where to Configure Sysmon Log Collection in Splunk Forwarder


Path:

```
C:\Program Files\SplunkUniversalForwarder\etc\apps\<app_name>\local\inputs.conf
```

Example:

```
C:\Program Files\SplunkUniversalForwarder\etc\apps\sysmon_inputs\local\inputs.conf
```

Why?

```
Splunk architecture uses apps for modular configuration.
```

Benefits:

* Easier management
* Clear separation of configs
* No modification of default system files
* Easier upgrades

---

# Example Sysmon Inputs Configuration

File:

```
inputs.conf
```

Example configuration:

```
[WinEventLog://Microsoft-Windows-Sysmon/Operational]
disabled = 0
index = windows
renderXml = true
```

Explanation:

| Parameter      | Meaning                            |
| -------------- | ---------------------------------- |
| WinEventLog    | Monitor Windows Event Log          |
| disabled=0     | Enable input                       |
| index          | Splunk index where logs are stored |
| renderXml=true | Preserve full event structure      |

---

# Windows Event Log Inputs (Example)

Your forwarder can monitor multiple logs:

```
[WinEventLog://Security]
disabled = 0
index = windows

[WinEventLog://System]
disabled = 0
index = windows

[WinEventLog://Application]
disabled = 0
index = windows
```

---

# Indexing Strategy in Splunk

This is an important **SIEM architecture decision**.

You have two options.

---

# Option 1 — Single Index (Simple Lab Setup)

Example:

```
index = windows
```

All Windows logs go into **one index**.

Example structure:

```
windows index
 ├─ Security logs
 ├─ System logs
 ├─ Application logs
 └─ Sysmon logs
```

### Advantages

* Simple configuration
* Easier searching

Example search:

```
index=windows
```

### Recommended for

Small SOC labs.

---

# Option 2 — Multiple Indexes (Production SOC)

Example indexes:

```
windows_security
windows_sysmon
windows_system
```

Configuration example:

```
[WinEventLog://Security]
index = windows_security

[WinEventLog://Microsoft-Windows-Sysmon/Operational]
index = windows_sysmon
```

### Advantages

Better data organization.

SOC teams separate logs by **data source**.

Example searches:

```
index=windows_sysmon
```

```
index=windows_security
```

---

# Recommended Index Structure for  Lab

For a beginner SOC lab, keep it simple.

Example indexes:

```
windows
linux
firewall
```

Example mapping:

| Source       | Index    |
| ------------ | -------- |
| Windows logs | windows  |
| Ubuntu logs  | linux    |
| pfSense logs | firewall |

---

---

# Final Configuration Files in Your Lab

### Windows Forwarder

```
inputs.conf
outputs.conf
```

### Ubuntu Forwarder

```
inputs.conf
outputs.conf
```

### Splunk Server

```
Receiving Port: 9997
Syslog Port: 514
```

---


# Step 5 — Configure Splunk Forwarder on Windows

Forwarder must monitor Windows logs.

Edit inputs configuration.

Location:

```
C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf
```

Example configuration:

```
[WinEventLog://Security]
disabled = 0
index = windows

[WinEventLog://System]
disabled = 0
index = windows

[WinEventLog://Application]
disabled = 0
index = windows
```

This tells the forwarder to monitor:

* Windows logs

---

# Step 6 — Configure Forwarding Destination

Edit:

```
outputs.conf
```

Location:

```
SplunkUniversalForwarder/etc/system/local/
```

Example:

```
[tcpout]
defaultGroup = splunk-indexer

[tcpout:splunk-indexer]
server = 192.168.56.40:9997
```

Meaning:

```
Send logs → Splunk server
```

---

# Step 7 — Install Splunk Universal Forwarder on Ubuntu

Download Linux forwarder.

Install:

```
sudo dpkg -i splunkforwarder.deb
```

Start service:

```
sudo /opt/splunkforwarder/bin/splunk start
```

---

# Step 8 — Configure Linux Log Monitoring

Edit:

```
inputs.conf
```

Location:

```
/opt/splunkforwarder/etc/system/local/
```

Example configuration:

```
[monitor:///var/log/syslog]
disabled = 0

[monitor:///var/log/auth.log]
disabled = 0
```

Meaning:

Forward Linux logs to Splunk.

---

# Step 9 — Configure Output for Ubuntu Forwarder

Edit:

```
outputs.conf
```

Example:

```
[tcpout]
defaultGroup = splunk-indexer

[tcpout:splunk-indexer]
server = 192.168.56.40:9997
```

---

# Step 10 — Send Firewall Logs (pfSense)

pfSense sends logs using **Syslog protocol**.

Configure in pfSense:

```
Status → System Logs → Settings
```

Enable:

```
Remote Logging
```

Set remote server:

```
192.168.56.40
```

Port:

```
514
```

Protocol:

```
UDP
```

---

# Step 11 — Configure Syslog Input in Splunk

On Splunk server configure:

```
Settings → Data Inputs → UDP
```

Add port:

```
514
```

This allows Splunk to ingest firewall logs.

---

# Final Log Pipeline

Your SOC lab pipeline becomes:

```
Windows Endpoint
   │
   │ Sysmon + Windows Logs
   ▼
Splunk Universal Forwarder
   │
   ▼
Splunk Indexer (Ubuntu)

--------------------------------

Ubuntu Logs
   │
   │ syslog / auth.log
   ▼
Splunk Universal Forwarder
   │
   ▼
Splunk Indexer

--------------------------------

pfSense Firewall
   │
   │ Syslog
   ▼
Splunk UDP Input
   │
   ▼
Splunk Indexer
```

---

# Final Verification

In Splunk search bar run:

```
index=*
```

Or:

```
sourcetype=syslog
```

You should see logs from:

* Windows
* Ubuntu
* pfSense

---
he **real SOC analyst skills interviews look for**.
