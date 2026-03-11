# BlueTeam-SOC-Detection-Lab
Hands-on SOC detection lab demonstrating SIEM monitoring, attack simulation, and detection engineering using Splunk, Sysmon, Windows logs, and Kali Linux etc. 
# SOC Detection Lab – Splunk + Sysmon

## Project Overview

This project demonstrates the design and implementation of a **Security Operations Center (SOC) detection lab** using Splunk SIEM, Windows logging, and Sysmon telemetry.

The purpose of this lab is to simulate real-world attack scenarios and demonstrate how SOC analysts detect, investigate, and respond to security incidents.

This lab focuses on:

• Log collection
• Detection engineering
• Attack simulation
• SOC investigation workflow

---

# Lab Objectives

The primary objectives of this project are:

• Build a SOC monitoring environment
• Collect endpoint telemetry using Sysmon
• Forward logs to Splunk SIEM
• Simulate attacks from an adversary machine
• Develop detection rules for malicious activity
• Perform SOC analyst investigation procedures

---

 [Lab Architecture](README.md)

This SOC lab consists of three primary systems:

1. Attacker Machine – Kali Linux
2. Victim Machine – Windows 10 with Sysmon
3. SIEM Server – Ubuntu with Splunk

### Log Flow

Attacker → Victim → Log Generation → Forwarder → Splunk → Detection Alert → SOC Investigation

---

# Network Architecture

```
                  ┌───────────────────┐
                  │   Kali Linux      │
                  │   (Attacker)      │
                  └─────────┬─────────┘
                            │
                            │ Attack Traffic
                            │
                  ┌─────────▼─────────┐
                  │   Windows 10      │
                  │   Victim Machine  │
                  │                   │
                  │  Sysmon Installed │
                  │  Splunk Forwarder │
                  └─────────┬─────────┘
                            │
                            │ Log Forwarding
                            │
                  ┌─────────▼─────────┐
                  │    Splunk SIEM    │
                  │   Ubuntu Server   │
                  └───────────────────┘
```

---

# Technologies Used

| Technology                 | Purpose                    |
| -------------------------- | -------------------------- |
| Splunk SIEM                | Log analysis and detection |
| Sysmon                     | Endpoint telemetry         |
| Splunk Universal Forwarder | Log forwarding             |
| Kali Linux                 | Attack simulation          |
| Windows 10                 | Target machine             |
| VirtualBox                 | Lab virtualization         |

---

# Log Sources

The following logs are collected in this SOC lab.

### Windows Security Logs

| Event ID | Description      |
| -------- | ---------------- |
| 4624     | Successful login |
| 4625     | Failed login     |
| 4688     | Process creation |

### Sysmon Logs

| Event ID | Description        |
| -------- | ------------------ |
| 1        | Process creation   |
| 3        | Network connection |
| 7        | DLL loaded         |
| 11       | File creation      |

These logs provide detailed visibility into endpoint activity.

---

# Lab Setup

## Splunk Installation

Splunk is installed on the SIEM server and configured to receive logs from endpoints.

```
sudo dpkg -i splunk.deb
sudo /opt/splunk/bin/splunk start
```

---

## Sysmon Installation

Sysmon is installed on the Windows endpoint to generate detailed system activity logs.

```
sysmon64.exe -i sysmonconfig.xml
```

---

## Splunk Forwarder Configuration

Splunk Universal Forwarder is installed on the Windows machine to send logs to the Splunk server.

Example configuration:

```
[WinEventLog://Security]
disabled = 0

[WinEventLog://System]
disabled = 0

[WinEventLog://Microsoft-Windows-Sysmon/Operational]
disabled = 0
```

---

# Attack Simulations

The following attack scenarios were simulated in this lab.

### Port Scanning

Command executed from Kali Linux:

```
nmap -sS <target-ip>
```

Detection method:

Sysmon Event ID 3 logs multiple network connections to different ports.

---

### Brute Force Login Attempt

Attack performed using multiple authentication attempts.

Detection method:

Windows Security Event ID 4625.

---

# Detection Engineering

Splunk detection queries were created to identify suspicious activity.

### Port Scan Detection Query

```
index=sysmon EventCode=3
| stats count by src_ip dest_port
| where count > 20
```

This query detects hosts scanning multiple ports in a short period of time.

---

# Investigation Workflow

SOC investigation follows the following workflow:

1. Alert triggered in Splunk
2. SOC analyst reviews alert details
3. Identify source IP address
4. Analyze logs related to the activity
5. Confirm malicious behavior
6. Escalate to incident response team

---

# Screenshots

Include screenshots demonstrating:

• Splunk dashboard
• Detection alerts
• Attack execution
• Log analysis

---

# Challenges Encountered

During lab setup several issues were encountered.

Example:

• Log forwarding misconfiguration
• Network adapter conflicts
• Splunk index configuration issues

These issues were resolved through troubleshooting and configuration updates.

---

# Lessons Learned

This project provided practical experience in:

• Security monitoring architecture
• Log analysis and detection engineering
• SOC investigation procedures
• Attack simulation and detection

---

# Future Improvements

Possible enhancements to this lab include:

• Integration of additional log sources
• Implementation of automated alerting
• Integration with threat intelligence feeds
• Deployment of a SOAR platform

---

# Author

SOC Detection Lab created as part of cybersecurity research and practical SOC training.

This project demonstrates hands-on experience with SIEM monitoring and attack detection.
