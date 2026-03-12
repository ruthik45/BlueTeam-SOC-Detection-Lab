
[⬅ Back to Main Project Documentation](../README.md)



## 1. Lab Overview

* Virtualized SOC lab built using **VirtualBox**
* Simulated enterprise network
* Logs forwarded to **Splunk SIEM**
* Multiple operating systems generating telemetry

##Lab architecture:
<img width="2816" height="1536" alt="image" src="https://github.com/user-attachments/assets/411271fa-c484-499e-87f6-053c6612cfe5" />


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
https://www.virtualbox.org/wiki/Downloads

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

Download the **Linux .deb package** for Ubuntu.(This should download in Ubuntu desktop vm later)

Example:

```
splunk-9.x.x-linux-amd64.deb
```

### Account Required?

✅ **Yes — free account required**(https://www.fakenamegenerator.com/)(https://emailfake.com/)

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

**Sysmon**

[https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)

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





# 5. Virtual Machine Creation (VirtualBox)

 lab contains **4 virtual machines**.

| VM             | Role                     | Installation Method |
| -------------- | ------------------------ | ------------------- |
| pfSense        | Firewall                 | ISO                 |
| Windows 11     | Endpoint                 | ISO                 |
| Ubuntu Desktop | Splunk SIEM + Linux logs | ISO                 |
| Kali Linux     | Attacker machine         | OVA (prebuilt VM)   |

---

# 1. Windows 11 Installation (ISO Required)

## Why ISO is Required

Microsoft distributes Windows installation media as **ISO images**, so the OS must be installed manually.

## File Type

```
.iso
```

Example

```
Win11_23H2_English_x64.iso
```

## VirtualBox Setup

```
Name: Windows11-SOC
Type: Microsoft Windows
Version: Windows 11 (64-bit)

RAM: 4096 MB
CPU: 2
Disk: 50 GB
```

Attach ISO in:

```
Settings → Storage → Optical Drive → Choose ISO
```

Boot the VM and install Windows.

After installation:

```
Remove ISO from Optical Drive
```

Reason:

```
To prevent the VM from booting back into the installer
```

---

# 2. Ubuntu Desktop Installation (ISO Required)

Ubuntu is installed manually because you will **install Splunk on it later**.

## File Type

```
.iso
```

Example

```
ubuntu-24.04-desktop-amd64.iso
```

## VM Configuration

```
Name: SOC-Splunk-Server

RAM: 4096 MB
CPU: 2
Disk: 50 GB
```

Attach ISO and install Ubuntu.

After installation:

```
Remove ISO
```

Reason

```
System should boot from installed disk
```

---

# 3. pfSense Firewall (ISO Required)

pfSense is distributed as a **bootable ISO installer**.

## File Type

```
.iso
```

Example

```
pfSense-CE-2.7.2-RELEASE-amd64.iso
```

## VM Configuration

```
Name: SOC-Firewall

RAM: 2048 MB
CPU: 2
Disk: 20 GB
```

After installation:

```
Remove ISO
```

Reason

```
Firewall must boot from installed disk
```

---

# 4. Kali Linux Attacker Machine (OVA – No ISO Needed)

Kali provides **prebuilt VirtualBox images**.

These contain:

* OS installed
* Virtual disk
* VM configuration

So installation is **not required**.

---

## File Type

```
.ova
```

Example

```
kali-linux-2025.1-virtualbox-amd64.ova
```

---

## Importing in VirtualBox

```
File
 → Import Appliance
 → Select kali-linux.ova
```

VirtualBox automatically creates the VM.

---

## Why Kali Uses OVA

Advantages:

```
No installation required
Preconfigured tools
Faster setup
Less configuration errors
```

---

# Summary of Installation Types

| VM             | File Type | Installation        |
| -------------- | --------- | ------------------- |
| Windows 11     | .iso      | Manual installation |
| Ubuntu Desktop | .iso      | Manual installation |
| pfSense        | .iso      | Manual installation |
| Kali Linux     | .ova      | Prebuilt VM import  |


