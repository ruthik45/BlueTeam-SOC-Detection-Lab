
---

# Allowing Kali (Attacker) to Reach the Internal Network

## Purpose

In this SOC lab architecture, **Kali Linux acts as the attacker machine** and is located on the **WAN network**. The internal machines (Windows, Ubuntu, etc.) are located on the **LAN network behind the pfSense firewall**.

By default, **pfSense blocks all incoming traffic from WAN to LAN** for security reasons. To simulate attack scenarios (port scanning, exploitation, lateral movement), we must configure:

1. A **Firewall Rule** to allow traffic from Kali to the internal network.
2. A **Default Route on Kali** so packets are sent through the pfSense firewall.

---

# Network Architecture

```
Attacker Network (WAN)
192.168.56.0/24

Kali Linux
192.168.56.30
        │
        ▼
pfSense Firewall
WAN: 192.168.56.1
LAN: 192.168.10.1
        │
        ▼
Internal SOC Network
192.168.10.0/24

Victim Machines
Windows / Ubuntu
```

---

# Step 1 — Create Firewall Rule (WAN → LAN)

Open the pfSense web interface.

Navigate to:

```
Firewall → Rules → WAN
```

Click **Add Rule**.

Configure the rule as follows:

```
Action: Pass
Interface: WAN
Address Family: IPv4
Protocol: Any
```

### Source Configuration

```
Source Type: Single Host
Source Address: 192.168.56.30
```

### Destination Configuration

```
Destination Type: Network
Destination: LAN net
```

### Description(Make sure to check the log box it is very important)

```

Allow Kali attacker traffic to internal SOC network
```

Click:

```
Save → Apply Changes
```
<img width="1920" height="1020" alt="image" src="https://github.com/user-attachments/assets/fcf0f88c-62d9-4a19-8d55-351dd3158124" />
<img width="1920" height="1020" alt="image" src="https://github.com/user-attachments/assets/1b7976a0-c3d9-4c4c-a7e1-75a9ab0eac31" />


---

# Step 2 — Verify Firewall Rule

After applying the changes, the rule table should look similar to:

```
PASS  IPv4  192.168.56.30 → LAN net
BLOCK Bogon Networks
```

This rule allows **Kali (attacker) to reach machines inside the LAN network**.
<img width="1920" height="1020" alt="image" src="https://github.com/user-attachments/assets/46bf33e8-274b-4d49-9bd6-96e9e50049d3" />
---

# Step 3 — Configure Default Route on Kali

Kali must send traffic for other networks through **pfSense WAN interface**.

Check current routes:

```bash
ip route
```

If a default route is missing, add it manually:

```bash
sudo ip route add default via 192.168.56.1
```

Where:

```
192.168.56.1 = pfSense WAN interface
```

---

# Step 4 — Verify Connectivity

Test connectivity from Kali to the internal network.

Ping the firewall LAN interface:

```bash
ping 192.168.10.1
```

Ping a victim machine:

```bash
ping 192.168.10.100
```

If successful, Kali can now reach the internal SOC network.


<img width="1920" height="1020" alt="image" src="https://github.com/user-attachments/assets/6bf0a26c-a4cf-4b9f-aecb-87eea7e9eddc" />

---

# Step 5 — Test Attack Simulation

Run a basic network scan:

```bash
nmap 192.168.10.0/24
```

Expected result:

```
Kali can discover internal machines.
```

This allows the SOC lab to generate realistic attack traffic for detection and monitoring.

---

# Monitoring the Attack Traffic

The generated traffic can now be observed in:

```
pfSense Firewall Logs
Windows Event Logs
Sysmon Logs
Splunk SIEM
```

Navigate in pfSense:

```
Status → System Logs → Firewall
```

You should see traffic generated from the Kali machine.

---

# Why This Configuration Is Important

Without this configuration:

```
WAN → LAN traffic is blocked
```

Meaning:

```
Kali cannot reach internal machines
No attack traffic is generated
SOC monitoring becomes impossible
```

This configuration enables **realistic attacker simulation inside the SOC lab environment**.

---
