
# SOC Alert Investigation Report

## ICMP Reconnaissance (Ping Sweep)

---

# 0. Advisory Perspective

An attacker initiated **ICMP echo requests** to the target host using a ping command.
This technique is commonly used for **network reconnaissance** to identify active systems.

**Command used by attacker**

```
ping -c 20 <victim_ip> or nmap -sn <victin_ip> or for whole network nmap -sn 192.168.23.0/24
```

**Attacker Objective**

If the host responds, the attacker confirms that the system is **alive and reachable** on the network.
This information can be used to perform further actions such as:

* Port scanning
* Service enumeration
* Vulnerability probing

---

# 1. Alert Information

**Alert Name:** ICMP Reconnaissance Detection
**Rule Name:** ICMP Ping Sweep Detection Rule
**Alert ID:**
**Severity:** Low / Medium / High / Critical
**Detection Source:** SIEM / Firewall / IDS / EDR
**Alert Trigger Time:**

---

# 2. Initial Alert Details

**Source IP:**
**Destination IP:**
**Username:** N/A
**Hostname:**
**Process Name:** ping
**File Path:** /bin/ping (Linux) or system ping utility

**Short Alert Description**

Multiple ICMP echo requests detected from a single source host targeting another system within a short time period, indicating possible **network reconnaissance activity**.

---

# 3. Investigation Queries Used

### Query 1 – ICMP Traffic Detection

```
index=network protocol=icmp
```

---

### Query 2 – Source IP Activity

```
index=network src_ip=<source_ip>
```

---

### Query 3 – ICMP Packet Frequency

```
index=network protocol=icmp
| stats count by src_ip dest_ip
```

---

# 4. Pivot Investigation

**Pivot Chain**

```
Alert → IP → Host → Network Activity
```

---

## IP Investigation

**Source IP Reputation:**
Check whether the IP is known malicious using threat intelligence.

**Internal / External:**

**Previous Activity from IP:**
Search historical logs to determine if the IP has previously generated alerts.

---

## Host Investigation

**Hostname:**

**Critical Asset:** Yes / No

**Previous Alerts on Host:**
Check if this host has been previously targeted or involved in alerts.

---

## Process Investigation

**Process Name:** ping

**Parent Process:**

**File Hash:** N/A

**File Reputation:** Legitimate system utility

---

## Network Investigation

**Outbound Connections:**

**Destination Hosts:**

**Suspicious Traffic Detected:**
Check if scanning or port probing followed the ICMP activity.

---

# 5. Timeline of Events

| Time  | Event                  | Description                          |
| ----- | ---------------------- | ------------------------------------ |
| 10:10 | ICMP Echo Request      | Source host initiated ping request   |
| 10:10 | ICMP Echo Reply        | Target host responded                |
| 10:11 | Multiple ICMP Requests | High frequency ICMP traffic observed |

---

# 6. MITRE ATT&CK Mapping

**Tactic:** Reconnaissance

**Technique:** Active Scanning

**Technique ID:** T1595

**MITRE Explanation**

Adversaries may perform **active scanning of networks** to discover reachable hosts, services, and network topology.

---

# 7. Analysis

### Key Questions Answered

* Is the activity malicious or benign?
* Is this normal behavior for the host?
* Is the targeted system a critical asset?
* Is additional reconnaissance or scanning observed?

---

### Findings

The source host generated **multiple ICMP echo requests** toward the destination system within a short timeframe.
This pattern is consistent with **network discovery activity**, often performed prior to port scanning or exploitation attempts.

---

# 8. Impact Assessment

**Affected Host:**

**Asset Criticality:** Low / Medium / High

**Potential Impact**

* Network reconnaissance
* Identification of active hosts
* Possible preparation for further attacks

---

# 9. Conclusion

**Incident Classification**

Benign Activity / Suspicious Activity / True Positive

**Reason for Conclusion**

ICMP traffic analysis indicates repeated ping requests from the source host.
No immediate exploitation activity observed during the investigation period.

---

# 10. Action Taken

**Alert Status:** Closed / Escalated to L2 / Monitoring

**Response Steps**

* Alert reviewed
* Logs analyzed
* Pivot investigation performed
* MITRE ATT&CK technique identified

**Escalation Notes (if applicable)**

Further monitoring recommended to detect additional reconnaissance or scanning attempts.

---

# 11. Analyst Information

**Analyst Name:**
**Role:** SOC L1 Analyst

**Investigation Start Time:**

**Investigation End Time:**
