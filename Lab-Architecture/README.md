
[⬅ Back to Main Project Documentation](../README.md)

# SOC Lab – Lab Setup Documentation Structure

## 1. Lab Overview

* Virtualized SOC lab built using **VirtualBox**
* Simulated enterprise network
* Logs forwarded to **Splunk SIEM**
* Multiple operating systems generating telemetry

**Machines in the Lab**

| Machine        | Role                             |
| -------------- | -------------------------------- |
| Windows 11     | Endpoint generating Windows logs |
| Kali Linux     | Attacker machine                 |
| Ubuntu Desktop | Linux endpoint generating logs   |
| pfSense        | Firewall + network gateway       |

---

# 2. System Requirements

what hardware is needed to run the lab.

| Component      | Minimum         |
| -------------- | --------------- |
| RAM            | 16 GB           |
| CPU            | 4 cores         |
| Storage        | 100 GB          |
| Virtualization | Enabled in BIOS |

Because SOC labs are **log heavy**, memory matters.

---

# 3. Software Requirements

List of the tools used.

| Software                   | Purpose             |
| -------------------------- | ------------------- |
| VirtualBox                 | Hypervisor          |
| Splunk Enterprise          | SIEM                |
| Splunk Universal Forwarder | Log forwarding      |
| Sysmon                     | Windows telemetry   |
| pfSense                    | Firewall            |
| Kali Linux                 | Attacker simulation |
| windows 11                 | viticm              |
| ubuntu desktop             | Management(splunk)  |

---

# 4. Download Sources 

I’m only listing **trusted official sources**, which is important for security labs.

---

# 1. Oracle VirtualBox (Hypervisor)

**Purpose**

Runs and manages all virtual machines in your SOC lab.

Download:

Oracle VM VirtualBox

Official page:
- <a href="https://www.virtualbox.org/wiki/Downloads" target="_blank">Download VirtualBox</a>

Download these two files:

1️⃣ **VirtualBox Platform Package**

Example file:

```
VirtualBox-7.x.x-Win.exe
```

2️⃣ **VirtualBox Extension Pack**

Example:

```
Oracle_VM_VirtualBox_Extension_Pack-7.x.x.vbox-extpack
```

### What the Extension Pack Does

Adds support for:

* USB 2.0 / USB 3.0
* VirtualBox RDP
* Disk encryption
* NVMe support

### Account Required?

❌ **No account required**

---

# 2. Windows 11 ISO

Download from Microsoft:

Windows 11

Official page:

[https://www.microsoft.com/software-download/windows11](https://www.microsoft.com/software-download/windows11)

Scroll to:

```
Download Windows 11 Disk Image (ISO)
```

Example file:

```
Win11_23H2_English_x64.iso
```

### Account Required?

❌ **No Microsoft account required for download**

### During Installation

Choose:

```
I don't have a product key
```

Windows installs **unactivated**, which is perfectly fine for a **SOC lab**.

---

# 3. Ubuntu Desktop ISO (Splunk Server)

Download from:

Ubuntu Desktop

Official page:

[https://ubuntu.com/download/desktop](https://ubuntu.com/download/desktop)

Example file:

```
ubuntu-24.04-desktop-amd64.iso
```

### Account Required?

❌ **No account required**

---

# 4. Kali Linux (Attacker Machine)

Download from:

Kali Linux

Official page:

[https://www.kali.org/get-kali/](https://www.kali.org/get-kali/)

Scroll to:

```
Virtual Machines
```

Download:

```
Kali Linux VirtualBox Image
```

Example file:

```
kali-linux-2025.1-virtualbox-amd64.ova
```

### Account Required?

❌ **No account required**

### Installation
<img width="1920" height="1020" alt="image" src="https://github.com/user-attachments/assets/1626f7cc-2fde-4cd0-93f7-137deb6b2dd2" />

Just import the OVA:

```
VirtualBox
 → File
 → Import Appliance
```

No installation needed.

---

# 5. pfSense Firewall

Download from:

pfSense CE

Official page:

[https://www.pfsense.org/download/](https://www.pfsense.org/download/)

Choose:

```
Architecture: AMD64
Installer: ISO Installer
Console: VGA
Mirror: Auto
```

Example file:

```
pfSense-CE-2.7.2-RELEASE-amd64.iso
```

### Account Required?

❌ **No account required**

---

# 6. Splunk Enterprise (SIEM)

Download from:

Splunk Enterprise

Official page:

[https://www.splunk.com/en_us/download/splunk-enterprise.html](https://www.splunk.com/en_us/download/splunk-enterprise.html)

Download the **Linux .deb package** for Ubuntu.

Example:

```
splunk-9.x.x-linux-amd64.deb
```

### Account Required?

✅ **Yes — free account required**

Reason:

Splunk tracks downloads for licensing.

Free license includes:

```
500 MB/day log ingestion
```

Which is **more than enough for your SOC lab**.

---

# 7. Splunk Universal Forwarder

Download from:

Splunk Universal Forwarder

Official page:

[https://www.splunk.com/en_us/download/universal-forwarder.html](https://www.splunk.com/en_us/download/universal-forwarder.html)

Example file:

```
splunkforwarder-9.x.x-linux-amd64.deb
```

### Account Required?

✅ **Yes — same Splunk account**

---

# Complete Download Summary

| Component                 | File Type     | Account Required |
| ------------------------- | ------------- | ---------------- |
| VirtualBox                | .exe          | ❌ No             |
| VirtualBox Extension Pack | .vbox-extpack | ❌ No             |
| Windows 11                | .iso          | ❌ No             |
| Ubuntu Desktop            | .iso          | ❌ No             |
| Kali Linux                | .ova          | ❌ No             |
| pfSense                   | .iso          | ❌ No             |
| Splunk Enterprise         | .deb          | ✅ Yes            |
| Splunk Forwarder          | .deb          | ✅ Yes            |

---

# Important Note for Your GitHub Documentation

You should include a section like:

```
Lab Requirements → Software Downloads
```

Example:

```
All software used in this SOC lab must be downloaded from
official sources to ensure integrity and avoid tampered images.
```

This makes your repository look **professional and security-focused**.

---

If you want, KernelGhost, I can also give you **one more powerful section for your GitHub SOC lab**:

**“File Integrity Verification (SHA256)”**

This is something **real security engineers do before installing ISO images**, and almost no beginner lab guides include it.

**Sysmon**

[https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)

---

# 5. Virtual Machine Creation (VirtualBox)

Document **how each VM is created**.

Example:

### Creating Windows 11 VM

Steps:

1. Open **VirtualBox**
2. Click **New**
3. Configure

```
Name: Windows11-SOC
Type: Microsoft Windows
Version: Windows 11 (64-bit)
RAM: 4096 MB
CPU: 2
Disk: 50 GB (VDI)
```

Then attach the ISO.

---

### Creating Kali Linux VM

```
Name: Kali-Attacker
RAM: 4096 MB
CPU: 2
Disk: 40 GB
```

Attach Kali ISO.

---

### Creating Ubuntu VM

```
Name: Ubuntu-Endpoint
RAM: 4096 MB
CPU: 2
Disk: 40 GB
```

Attach Ubuntu ISO.

---

### Creating pfSense Firewall

```
Name: pfSense-Firewall
RAM: 2048 MB
CPU: 2
Disk: 20 GB
```

Attach pfSense ISO.

---

# 6. Network Architecture

Very important for SOC labs.

Explain **how machines communicate**.

Example architecture:

```
                Internet (Optional)
                       |
                   pfSense
                (Firewall)
                       |
            ---------------------
            |        |         |
        Windows    Ubuntu     Kali
        Endpoint   Endpoint   Attacker
```

In **VirtualBox**

```
Adapter 1 → Internal Network (SOC-NET)
Adapter 2 → NAT (Internet access)
```

pfSense acts as **gateway and firewall**.

---

# 7. Boot Process

Explain the order machines are started.

Correct order:

1️⃣ pfSense (network gateway)
2️⃣ Splunk Server
3️⃣ Windows Endpoint
4️⃣ Ubuntu Endpoint
5️⃣ Kali Attacker

Why?

Because endpoints need the **network infrastructure first**.

---

# 8. Initial Network Configuration

Example:

```
pfSense LAN: 192.168.56.1
Windows:     192.168.56.10
Ubuntu:      192.168.56.20
Kali:        192.168.56.30
Splunk:      192.168.56.40
```

---

# 9. Log Generation Setup

Explain what logs each machine generates.

| Machine | Logs              |
| ------- | ----------------- |
| Windows | Sysmon logs       |
| Ubuntu  | syslog / auth.log |
| pfSense | firewall logs     |
| Kali    | attack traffic    |

---

# 10. Log Forwarding

Explain:

```
Endpoints → Splunk Universal Forwarder → Splunk Indexer → Splunk Dashboard
```

Example:

Windows

```
Sysmon → Windows Event Logs → Splunk UF → Splunk
```

Ubuntu

```
/var/log/syslog → Splunk UF → Splunk
```

pfSense

```
Firewall logs → Syslog → Splunk
```

---

# 11. Verification

Example tests:

**Ping test**

```
ping 192.168.56.1
```

**Log verification**

Splunk query:

```
index=main
```

---

# 12. Troubleshooting Section

Add problems you faced.

Example:

Problem 1
VMs cannot communicate

Solution

```
Ensure all machines use the same internal network
SOC-NET
```

Problem 2
Logs not reaching Splunk

Check:

```
splunk status
```

---

# Professional Tip (Important)

In your GitHub repo create this structure:

```
SOC-LAB
│
├── Architecture
│   └── lab-architecture.md
│
├── Lab-Setup
│   ├── requirements.md
│   ├── downloads.md
│   ├── vm-creation.md
│   ├── network-setup.md
│   └── boot-process.md
│
├── Log-Sources
│
├── Detection-Rules
│
└── Attack-Simulations
```

This makes your lab look like a **real SOC engineering project**.

---

If you want, KernelGhost, I can also show you something **very powerful for your GitHub SOC lab**:

**A professional SOC lab architecture diagram (like real security teams use)** including:

* Splunk
* pfSense
* attackers
* endpoints
* log pipelines

It will make your repository look **10× more professional.**
