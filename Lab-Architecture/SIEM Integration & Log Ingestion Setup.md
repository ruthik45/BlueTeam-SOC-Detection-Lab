
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

Windows already has logs, but they are **limited for security monitoring**.

Therefore we install:

Sysmon

Sysmon provides **deep telemetry** for detection engineering.

### Download Sysmon

From Microsoft Sysinternals.

Extract files:

```
Sysmon.exe
Sysmon64.exe
```

### Install Sysmon

```
Sysmon64.exe -i sysmonconfig.xml
```

Purpose:

Sysmon captures events such as:

| Event               | Detection Use     |
| ------------------- | ----------------- |
| Process creation    | Malware execution |
| Network connections | C2 communication  |
| File creation       | Malware dropper   |
| Registry changes    | Persistence       |

These events are stored in:

```
Event Viewer
Applications and Services Logs
Microsoft
Windows
Sysmon
Operational
```

---

# Why Sysmon is Needed

Windows Event Logs alone are **not detailed enough for threat detection**.

### Windows Native Logs

Examples:

| Log             | Information  |
| --------------- | ------------ |
| Security log    | Login events |
| System log      | OS events    |
| Application log | App crashes  |

### Sysmon Logs

Provide **high-fidelity security telemetry**.

Example events:

| Event ID | Description        |
| -------- | ------------------ |
| 1        | Process creation   |
| 3        | Network connection |
| 7        | DLL loaded         |
| 11       | File creation      |

SOC analysts use these events to detect attacks.

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

[WinEventLog://System]
disabled = 0

[WinEventLog://Application]
disabled = 0

[WinEventLog://Microsoft-Windows-Sysmon/Operational]
disabled = 0
```

This tells the forwarder to monitor:

* Windows logs
* Sysmon logs

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
