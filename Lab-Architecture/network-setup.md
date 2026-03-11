KernelGhost (newkaoboy), I kept **everything exactly the same** as you wrote and only converted it into **clean GitHub `README.md` markdown format** (headings, code blocks, spacing, and fixed the broken backtick in the traffic flow block).
No architectural or wording changes.

You can **paste this directly into your `README.md`**.

---

# SOC Lab Network Architecture

```
                     ATTACKER NETWORK
                     192.168.56.0/24
                (VirtualBox Host-Only Network)

                         Kali Linux
                       192.168.56.30
                              │
                              │
                              ▼
                       pfSense Firewall
                   WAN: 192.168.56.1
                   LAN: 192.168.10.1
                              │
                              │
                              ▼
                INTERNAL SOC NETWORK
                  192.168.10.0/24
                 (VirtualBox Internal Network)

                ┌─────────────────────────┐
                │                         │
                ▼                         ▼

        Windows 11 Endpoint        Ubuntu SOC Server
          DHCP Client               Static IP
        192.168.10.102              192.168.10.103
```

---

# Network Summary

The lab contains **two separate networks**.

## External Network (Attack Network)

Used for the attacker machine and the firewall WAN interface.

```
Network: 192.168.56.0/24
```

Machines connected:

* Kali Linux
* pfSense WAN interface

IP configuration:

```
Static IP addressing
No DHCP used
```

---

## Internal SOC Network

This network simulates the **internal enterprise network** monitored by the SOC.

```
Network: 192.168.10.0/24
```

Machines connected:

* Windows endpoint
* Ubuntu SOC server
* pfSense LAN interface

IP configuration:

```
DHCP provided by pfSense
```

---

# Traffic Flow

```
Kali Linux (Attacker)
        │
        │ Attack Traffic
        ▼
pfSense Firewall (WAN)
        │
        │ Routed Traffic
        ▼
Internal SOC Network
        │
        ├── Windows Endpoint
        └── Ubuntu SOC Server
```

---

# Lab Components

| System        | Role                            |
| ------------- | ------------------------------- |
| Kali Linux    | Attacker machine                |
| pfSense       | Firewall, router, gateway, DHCP |
| Windows 11    | Monitored endpoint              |
| Ubuntu Server | SOC server running Splunk       |

---

# Create VirtualBox Networks

## External Network (Host-Only Adapter)

Open:

```
VirtualBox → File → Tools → Host Network Manager
```

Create a host-only network.

Click on **Add** symbol. VirtualBox automatically creates it.

Configuration:

```
IPv4 Address: 192.168.56.1
Subnet Mask: 255.255.255.0
```

Disable DHCP. *(Optional)*

Reason:

```
The external network uses static IP addresses.
```

---

## Internal SOC Network

VirtualBox automatically creates internal networks.
We need to name the network while attaching the VM adapter.

Network name used in this lab:

```
SOC-NET
```

---

# Virtual Machine Network Configuration

---

# Kali Linux (Attacker Machine)

Open:

```
VM Settings → Network
```

Adapter configuration:

```
Adapter 1
Attached To: Host-Only Adapter
Network: Select the one you created.
```

---

# pfSense Firewall

pfSense requires **two network adapters**.

---

## Adapter 1 — WAN

```
Attached To: Host-Only Adapter
Network: Select the one you created and ensure both Kali and pfsense On same adapter.
```

Purpose:

```
Connects firewall to attacker network
```

---

## Adapter 2 — LAN

```
Attached To: Internal Network
Network Name: SOC-NET
```

Purpose:

```
Internal SOC network
```

---

# Configure DHCP Server from pfSense Console (Step-by-Step)

After pfSense boots, you will see the **console menu**.

Example:

```
Welcome to pfSense

1) Assign Interfaces
2) Set interface IP address
3) Reset webConfigurator password
4) Reset to factory defaults
5) Reboot system
6) Halt system
7) Ping host
8) Shell
```

To configure DHCP from the console we use:

```
Option 2 → Set interface IP address
```

---

# Step 1 — Select Interface

You will see:

```
Available interfaces:

1 - WAN (em0)
2 - LAN (em1)

Enter the number of the interface you wish to configure:
```

Choose:

```
2
```

Because DHCP will run on the **LAN interface**.

---

# Step 2 — Configure IPv4 Address

The system will ask:

```
Configure IPv4 address LAN interface via DHCP? (y/n)
```

Choose:

```
n
```

Reason:

```
The firewall must use a static gateway IP.
```

---

# Step 3 — Enter LAN IPv4 Address

Prompt:

```
Enter the new LAN IPv4 address:
```

Enter:

```
192.168.10.1
```

This becomes the **gateway of the internal network**.

---

# Step 4 — Enter Subnet Mask

Prompt:

```
Enter the new LAN IPv4 subnet bit count (1 to 32):
```

Enter:

```
24
```

Which corresponds to:

```
255.255.255.0
```

---

# Step 5 — Configure Upstream Gateway

Prompt:

```
For a LAN, press Enter for none:
```

Press:

```
ENTER
```

LAN interfaces normally **do not have upstream gateways**.

---

# Step 6 — Configure IPv6

Prompt:

```
Configure IPv6 address LAN interface via DHCP6? (y/n)
```

Choose:

```
n
```

Because IPv6 is not needed in this lab.

---

# Step 7 — Enable DHCP Server

Now pfSense will ask:

```
Do you want to enable the DHCP server on LAN? (y/n)
```

Choose:

```
y
```

---

# Step 8 — Enter DHCP Range Start

Prompt:

```
Enter the start address of the DHCP pool:
```

Example:

```
192.168.10.100
```

---

# Step 9 — Enter DHCP Range End

Prompt:

```
Enter the end address of the DHCP pool:
```

Example:

```
192.168.10.200
```

This creates the DHCP range:

```
192.168.10.100 — 192.168.10.200
```

---

# Step 10 — Confirm Configuration

pfSense will display the configuration summary and apply the settings.

Example output:

```
LAN IP address: 192.168.10.1
Subnet: /24
DHCP Server: Enabled
Range: 192.168.10.100 – 192.168.10.200
```

---

# Step 11 — Final Output

pfSense will display:

```
You can now access the webConfigurator by opening:

http://192.168.10.1
```

At this point:

* pfSense gateway = `192.168.10.1`
* DHCP server = enabled
* internal machines can now obtain IP addresses

---

# Step 12 — Start Internal Machines

Now start:

```
Windows VM
Ubuntu VM
```

Both should be connected to:

```
Internal Network → SOC-NET
```

Then run:

```
ipconfig /renew
```

Expected DHCP assignment:

```
IP: 192.168.10.101
Gateway: 192.168.10.1
DNS: 192.168.10.1
```

---

# Correct DHCP Flow in the Lab

```
pfSense LAN
192.168.10.1
     │
     │ DHCP Server
     ▼
Windows Endpoint
192.168.10.101

Ubuntu SOC Server
192.168.10.102
```

---

If you want, KernelGhost (newkaoboy), I can also show you a **small GitHub trick SOC engineers use**:

How to add **clickable architecture diagrams and section navigation inside README** so your repo looks like a **professional security research project instead of just notes.**
