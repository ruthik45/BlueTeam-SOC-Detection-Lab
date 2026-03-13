
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
## Log Validation Playbook
---

# 1 — Network Connectivity Test (Endpoint → Splunk)

First verify **the endpoint can reach the Splunk receiving port (9997)**.(Make sure of two things one you need to Root in Ubuntu terminal, Open Powershell as administrator and Give a correct IP of your Splunk server That you configured)

## Windows Endpoint

Test TCP connection.

```
Test-NetConnection 192.168.56.40 -Port 9997
```

### Expected Output

```
ComputerName     : 192.168.56.40
RemotePort       : 9997
TcpTestSucceeded : True
```

If it shows:

```
TcpTestSucceeded : False
```

Problem could be:

* Firewall blocking
* Splunk not listening
* Wrong IP


<img width="1920" height="1020" alt="image" src="https://github.com/user-attachments/assets/fc28a057-e641-4cda-a95f-afbb8de8f101" />
---

## Ubuntu Forwarder

```
nc -zv 192.168.56.40 9997
```

Expected:

```
Connection to 192.168.56.40 9997 port [tcp/*] succeeded!
```

<img width="761" height="477" alt="image" src="https://github.com/user-attachments/assets/d8a40d2e-40d4-41c3-9505-ca8fa88cc6d6" />


---

# 2 — Verify Splunk Receiving Port

Run on the **Splunk server (Ubuntu)**.

```
sudo netstat -tulnp | grep 9997
```

Expected:

```
tcp   0   0 0.0.0.0:9997   0.0.0.0:*   LISTEN
```

<img width="781" height="280" alt="image" src="https://github.com/user-attachments/assets/15a495f1-d7bf-4528-b944-086095fa368b" />


Meaning:

```
Splunk indexer listening for forwarders
```

You can also verify from Splunk UI:

```
Settings → Forwarding and Receiving → Receiving
```
<img width="1325" height="571" alt="image" src="https://github.com/user-attachments/assets/1b8e5608-2a69-485b-afed-a41234bc6de3" />

---

# 3 — Verify Forwarder Status

## Windows Endpoint

Check forwarder service.

```
Get-Service SplunkForwarder
```

Expected:

```
Status : Running
```
<img width="675" height="397" alt="image" src="https://github.com/user-attachments/assets/ca57f4f2-c654-40bb-a0ec-afaba8f9f0d1" />

---

### CLI Check

```
cd "C:\Program Files\SplunkUniversalForwarder\bin"

splunk list forward-server
```

Expected output:

```
Active forwards:
192.168.56.40:9997
```

Meaning:

```
Forwarder successfully connected to indexer
```

---

## Ubuntu Forwarder

Check service:

```
sudo /opt/splunkforwarder/bin/splunk status
```

Expected:

```
splunkd is running
```
<img width="726" height="363" alt="image" src="https://github.com/user-attachments/assets/bb083d0d-c713-40d3-830a-fd9a4fe13860" />

---

Check connection:

```
sudo /opt/splunkforwarder/bin/splunk list forward-server
```

Expected:

```
Active forwards:
192.168.56.40:9997
```
<img width="762" height="543" alt="image" src="https://github.com/user-attachments/assets/6e4c91a9-5e27-42f7-a934-55bb85a83724" />

---

# 4 — Verify pfSense Syslog Connectivity

Since pfSense uses **UDP 514**, verify Splunk is listening.

On Splunk server:

```
sudo netstat -anu | grep 514
```

Expected:

```
udp   0   0 0.0.0.0:514   0.0.0.0:*
```
<img width="792" height="231" alt="image" src="https://github.com/user-attachments/assets/247a6afe-3a08-40a6-9cc3-8051a2a08384" />

---

# 5 — Verify Logs in Splunk (First Basic Check)

Run in Splunk search:

```
index=*
```

Expected output:

Logs from multiple sources.

Fields should show:

```
host
source
sourcetype
index
```

Example:

| host             | source          | index    |
| ---------------- | --------------- | -------- |
| windows-endpoint | WinEventLog     | windows  |
| ubuntu-server    | /var/log/syslog | linux    |
| pfsense-firewall | syslog          | firewall |

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/139e8dee-1fab-4b9e-8519-7037de50fc61" />


<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/a93de57f-abd0-4283-8324-93d25cf48cf9" />


---

# 6 — Check Windows Logs

Search:

```
index=windows
```

Expected logs:

```
Security
System
Application
Sysmon
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/6d0edb76-5c0f-43f4-b078-9e2a77c1a501" />

---

### Specific Query

Security log check

```
index=windows sourcetype="WinEventLog:Security"
```

Expected:

```
EventCode=4624
EventCode=4625
```

Meaning:

```
Login events arriving
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/1855b0fc-f73a-49bb-a311-80d5c8ce4449" />

---

# 7 — Check Sysmon Logs

Search:

```
index=windows sourcetype="WinEventLog:Microsoft-Windows-Sysmon/Operational"
```

Expected events:

| Event ID | Meaning            |
| -------- | ------------------ |
| 1        | Process creation   |
| 3        | Network connection |
| 11       | File creation      |


<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/59caa346-a876-4c5b-ba55-8cabc51b67a1" />

---

# 8 — Check Linux Logs

Search:

```
index=linux
```

Or:

```
index=linux source="/var/log/syslog"
```

Expected events:

```
systemd
ssh
cron
kernel
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/fc827b3c-10c4-4f93-8430-ae04c14a9c7c" />

---

# 9 — Check Firewall Logs

Search:

```
index=firewall
```

Or:

```
sourcetype=syslog
```

Expected events:

```
pf
filterlog
firewall allow/deny
```
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/33108b84-a588-482b-b006-878d328ace8e" />

---

# 10 — Log Generation Tests (Important)

SOC engineers **force activity to test log ingestion**.

---

## Windows Test

Generate login event.

```
Run → logoff
```
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/c9907c70-a000-43d5-9d6b-3d30ca12ef89" />


Then log in again.

Check:

```
index=windows EventCode=4624
```
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/14ffa67f-617e-45a8-8f9d-419c3de6620c" />

---

## Generate Failed Login

Attempt wrong password.

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/56426beb-11bb-462f-bc46-92b38372f106" />


Query:

```
index=windows EventCode=4625
```
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/dd395059-07ed-4dda-9fe1-92333602ca0a" />

---

## Generate Sysmon Process Event

Run command:

```
notepad.exe
```

Query:

```
index=windows EventCode=1
```

Expected:

```
Image="notepad.exe"
```
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/22e1de98-1aca-4807-a9bb-5088ccee01b5" />

---

# 11 — Linux Log Generation Test

SSH login test.

```
ssh localhost
```

Check:

```
index=linux source="/var/log/auth.log"
```

Expected event:

```
Accepted password for user
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/f1982d6d-617b-4130-9c4d-2369f5d26bcd" />

check:
  su -

  <img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/b33ddfe8-073d-4f91-bf24-60e54501ed43" />

---

# 12 — Firewall Log Test

Generate traffic.

From Kali:

```
ping 8.8.8.8
```

Query:

```
index=firewall
```

Expected event:

```
filterlog
action=pass
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/8ecc53e1-c27e-48a4-8abf-23d9c097a523" />

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/98394d58-58c9-4b75-a951-3f84812a5bf2" />

---

# 13 — SOC Analyst Health Check Queries

SOC teams run these periodically.

### Source health

```
| metadata type=hosts
```

Shows active hosts sending logs.

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/a1c73ca1-4644-4014-b6b5-7329dafc6bf5" />


---

### Index health

```
| tstats count where index=* by index
```

Shows log count per index.


<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/cab1e814-e67d-496d-8b85-34649fb3acfc" />


---

### Source type check

```
| tstats count where index=* by sourcetype
```

Shows log type distribution.

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/2bc351cc-9abf-40d6-ad12-e2866a9f80f7" />


---

### Check recent logs

```
index=* earliest=-5m
```

Confirms **logs arriving in real-time**.

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/6b147ab3-c6cc-4e9b-a60f-05a8c8927a28" />


---

# 14 — Detect Forwarder Issues

Search for missing logs:

```
index=windows earliest=-5m
```

If **no logs appear**:

Possible causes:

* Forwarder stopped
* outputs.conf wrong
* receiving port closed
* firewall blocking

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/fee682b2-af4c-40c4-b3c5-b08541b448d3" />


---

# 15 — Final SOC Log Flow Validation

Your pipeline validation checklist becomes:

```
Endpoint
   │
   │ Connectivity test
   ▼
Forwarder service running
   │
   │ Forward-server active
   ▼
Indexer receiving port open
   │
   │ Log ingestion test
   ▼
Search validation
   │
   ▼
SOC detection queries
```

---
# Important Note

All the commands and queries shown above were executed in my SOC lab environment.

The commands visible in the screenshots may slightly differ from the ones written in the documentation because they were adapted based on my lab setup.

Every lab environment will have different configurations such as:

* Different **IP addresses**
* Different **index names**
* Different **hostnames**
* Different **log paths**
* Different **network architectures**

Therefore, do **not copy the commands blindly** from this guide.

Before running any command, ensure the following in your environment:

* Replace the **IP addresses** with the correct IP of your Splunk server.
* Use the **correct index names** configured in your Splunk instance.
* Ensure the **Splunk receiving port (9997)** is reachable.
* Verify that **forwarders are properly connected to the indexer**.
* Confirm **network connectivity between endpoints and the SIEM server**.

Always adapt the commands based on **your own SOC lab configuration** to avoid connection issues or ingestion failures.
---
