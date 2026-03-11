
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

### Attacker Network (WAN Side)

```
Network: 192.168.56.0/24
Gateway: 192.168.56.1
VirtualBox Type: Host-Only Adapter
```

Devices:

```
Kali Linux
pfSense WAN Interface
```

---

### Internal SOC Network (LAN Side)

```
Network: 192.168.10.0/24
Gateway: 192.168.10.1
DHCP Server: pfSense
VirtualBox Type: Internal Network
```

Devices:

```
Windows 11 Endpoint
Ubuntu SOC Server
pfSense LAN Interface
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

