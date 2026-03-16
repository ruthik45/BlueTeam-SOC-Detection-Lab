
---

# SOC Alert Investigation Report

## 1. Alert Information

```
Alert ID: LAB-SPLUNK-T1046-20260316-002
Alert Name: Network Ping Sweep Detected
Severity: Medium
Detection Source: Splunk SIEM

Detection Rule: Multiple ICMP Echo Requests from Single Source
Timestamp: 16 Mar 2026 09:22 UTC

SOC Analyst: L1 SOC Analyst
Ticket ID: SOC-INC-2026-104
```

---

# 2. Alert Description

```
The alert was triggered when the SIEM detection rule identified multiple
ICMP echo requests originating from a single source IP targeting multiple
hosts within a short time window.

Such behavior is commonly associated with network reconnaissance activity
where an attacker attempts to identify active hosts on the network.
```

---

# 3. Observables / Indicators

```
Source IP: 192.168.1.25
Destination Range: 192.168.1.1 – 192.168.1.40
Protocol: ICMP
Destination Port: N/A
Command Pattern: ICMP Echo Request (Ping)

Hostname: Unknown
User Account: N/A
```

---

# 4. Investigation

```
• Reviewed alert details in Splunk SIEM.

• Firewall logs show ICMP echo requests sent from source IP
  192.168.1.25 to multiple hosts in the internal network.

• Approximately 35 ICMP requests observed within 20 seconds.

• Checked endpoint logs for the source host to determine if any
  scanning tools (Nmap / ping scripts) were executed.

• Verified asset inventory to identify the source host.

• Checked threat intelligence sources for the source IP reputation.

• No external communication or exploitation attempts detected.
```

---

# 5. MITRE ATT&CK Mapping

```
Tactic: Discovery

Technique: Network Service Discovery

Technique ID: T1046
```

---

# 6. Analysis & Verdict

```
The investigation indicates that the source host generated multiple
ICMP echo requests targeting several internal systems.

This behavior is consistent with network reconnaissance techniques
used to identify active hosts in the network.

Further analysis revealed that the source IP belongs to an internal
IT scanning tool used for network diagnostics.

No malicious payloads or exploitation attempts were observed.
```

---

# 7. Response & Closure

```
Actions Taken:
• Verified source host ownership with IT team
• Confirmed activity originated from internal network diagnostic tool

Final Classification:
Benign Activity

Case Status:
Closed
```
# 8. Evidence
  
   - Screenshot 1: Log evidence
   - 
              
   - Screenshot 2: Query results
---

