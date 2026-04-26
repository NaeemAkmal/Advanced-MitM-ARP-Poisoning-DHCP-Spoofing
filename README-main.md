<div align="center">

# 🔐 Advanced MitM — ARP Poisoning & DHCP Spoofing

[![Kali Linux](https://img.shields.io/badge/Kali_Linux-557C94?style=for-the-badge&logo=kalilinux&logoColor=white)](https://www.kali.org/)
[![Parrot OS](https://img.shields.io/badge/Parrot_OS-15E0ED?style=for-the-badge&logo=linux&logoColor=black)](https://www.parrotsec.org/)
[![Ettercap](https://img.shields.io/badge/Ettercap-0.8.4-red?style=for-the-badge&logo=gnubash&logoColor=white)](https://www.ettercap-project.org/)
[![Wireshark](https://img.shields.io/badge/Wireshark-1679A7?style=for-the-badge&logo=wireshark&logoColor=white)](https://www.wireshark.org/)
[![GNS3](https://img.shields.io/badge/GNS3-Network_Sim-orange?style=for-the-badge&logo=cisco&logoColor=white)](https://www.gns3.com/)
[![VMware](https://img.shields.io/badge/VMware-607078?style=for-the-badge&logo=vmware&logoColor=white)](https://www.vmware.com/)

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=for-the-badge)](LICENSE)
[![Platform](https://img.shields.io/badge/Platform-Linux%20%7C%20Windows-blue?style=for-the-badge)]()
[![Status](https://img.shields.io/badge/Status-Completed-brightgreen?style=for-the-badge)]()
[![Educational](https://img.shields.io/badge/Purpose-Educational-purple?style=for-the-badge)]()

<br/>

> **Hands-on demonstration of Man-in-the-Middle attacks using ARP Poisoning and DHCP Spoofing techniques in a controlled virtual lab environment — with full mitigation strategies.**

</div>

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Lab Environment](#-lab-environment)
- [Attack 1 — ARP Poisoning (MitM)](#-attack-1--arp-poisoning-mitm)
- [Attack 2 — DHCP Spoofing](#-attack-2--dhcp-spoofing)
- [Mitigation Strategies](#-mitigation-strategies)
- [Tools & Technologies](#-tools--technologies)
- [Disclaimer](#-disclaimer)

---

## 🧠 Overview

This lab demonstrates two critical Layer 2 network attacks that are commonly exploited in real-world penetration testing scenarios:

| Attack | Protocol Abused | Impact |
|--------|----------------|--------|
| **ARP Poisoning** | ARP (Layer 2) | Full traffic interception between two hosts |
| **DHCP Spoofing** | DHCP (Layer 3) | Rogue gateway/DNS assignment to network clients |

Both attacks were executed in an **isolated virtual lab** using VMware and GNS3, with full packet-level verification via Wireshark.

---

## 🖥️ Lab Environment

```
┌─────────────────────────────────────────────────────────────┐
│                    VIRTUAL LAB SETUP                        │
├──────────────────┬──────────────────────────────────────────┤
│ Attacker         │ Kali Linux / Parrot OS                   │
│ Victim 1         │ Windows 7  — 192.168.23.135              │
│ Victim 2         │ Windows 10 — 192.168.23.136              │
│ Attacker IP      │ eth0: 192.168.23.129 │ eth1: 150.1.7.101 │
│ DHCP Network     │ 150.1.7.0/24 (GNS3)                     │
│ Hypervisor       │ VMware Workstation                       │
│ Network Sim      │ GNS3                                     │
└──────────────────┴──────────────────────────────────────────┘
```

---

## ⚡ Attack 1 — ARP Poisoning (MitM)

> Intercept traffic between two VMs by poisoning their ARP caches using Ettercap.

📁 Full walkthrough with screenshots → **[arp-poisoning-mitm/README.md](arp-poisoning-mitm/README.md)**

### Quick Summary

```
1. Verify IPs on all machines
2. Confirm connectivity (ping tests)
3. Enable IP forwarding on Kali
4. Record baseline ARP tables
5. Launch Ettercap → Scan → Set Targets
6. Start ARP Poisoning (MITM)
7. Verify poisoned ARP caches on victims
8. Capture intercepted traffic in Wireshark
```

### Key Results

| Observation | Before Attack | After Attack |
|-------------|--------------|--------------|
| Win7 ARP entry for Win10 | Correct MAC | **Attacker's MAC** |
| Win10 ARP entry for Win7 | Correct MAC | **Attacker's MAC** |
| Wireshark ICMP | Not visible on Kali | **Fully visible on Kali** |

### 📸 Preview

| IP Configuration | ARP Poisoning Active | Traffic Captured |
|:---:|:---:|:---:|
| ![](arp-poisoning-mitm/victim2-win10-ipconfig.png) | ![](arp-poisoning-mitm/arp-table-win10-after-poisoning.png) | ![](arp-poisoning-mitm/wireshark-icmp-interception.png) |

---

## ⚡ Attack 2 — DHCP Spoofing

> Impersonate a DHCP server to assign rogue gateway and DNS settings to network clients.

📁 Full walkthrough with screenshots → **[dhcp-spoofing/README.md](dhcp-spoofing/README.md)**

### Quick Summary

```
1. Configure R1 as legitimate DHCP server (GNS3)
2. R2 receives legitimate IP (150.1.7.111) from R1
3. Launch Ettercap on Parrot OS → Scan hosts
4. Start MITM → DHCP Spoofing plugin
5. Set rogue pool: 150.1.7.200-250, DNS: 150.1.7.101
6. Trigger DHCP renewal on R2 (shutdown/no shutdown)
7. R2 receives rogue IP (150.1.7.200) from attacker
```

### Key Results

| | Legitimate | After Spoofing |
|--|-----------|---------------|
| R2 IP Address | 150.1.7.111 (from R1) | **150.1.7.200 (from attacker)** |
| Gateway | 150.1.7.103 | **150.1.7.101 (attacker)** |
| DNS Server | 8.8.8.8 | **150.1.7.101 (attacker)** |

### 📸 Preview

| GNS3 Topology | DHCP Spoof Config | Rogue IP Assigned |
|:---:|:---:|:---:|
| ![](dhcp-spoofing/network-topology-gns3.png) | ![](dhcp-spoofing/ettercap-dhcp-spoofing-config.png) | ![](dhcp-spoofing/r2-rogue-ip-assigned.png) |

---

## 🛡️ Mitigation Strategies

### Against ARP Poisoning
- ✅ **Dynamic ARP Inspection (DAI)** — validates ARP packets against DHCP snooping table
- ✅ **Static ARP entries** — for critical devices (gateways, servers)
- ✅ **Traffic encryption** — TLS/HTTPS/VPN so intercepted data is unreadable
- ✅ **VLAN segmentation** — limits broadcast domain scope
- ✅ **ARP monitoring tools** — XArp, Wireshark alerts for duplicate IP-MAC mappings

### Against DHCP Spoofing
- ✅ **DHCP Snooping** — only trusted ports can send DHCP replies
- ✅ **802.1X port authentication** — blocks unauthorized devices
- ✅ **Static IP assignments** — for critical infrastructure
- ✅ **IDS/IPS monitoring** — detect rogue DHCP OFFER packets
- ✅ **VLAN isolation** — contain DHCP traffic scope

---

## 🛠️ Tools & Technologies

| Tool | Version | Purpose |
|------|---------|---------|
| **Ettercap** | 0.8.4 / 0.8.3.1 | ARP poisoning & DHCP spoofing |
| **Wireshark** | Latest | Packet capture & analysis |
| **Kali Linux** | 2024 | Primary attack platform |
| **Parrot OS** | Latest | Secondary attack platform |
| **VMware Workstation** | — | Virtual machine hypervisor |
| **GNS3** | Latest | Network simulation (routers) |
| **Windows 7 / 10** | — | Victim machines |

---

## ⚠️ Disclaimer

> This project is strictly for **educational and research purposes** in a controlled, isolated lab environment. All techniques demonstrated here are performed on systems owned and operated by the researcher. Unauthorized use of these techniques on any network or system without explicit permission is **illegal and unethical**. The author takes no responsibility for misuse of this information.

---

<div align="center">

**Naeem Akmal** · Cyber Security Researcher

[![GitHub](https://img.shields.io/badge/GitHub-NaeemAkmal-black?style=flat-square&logo=github)](https://github.com/NaeemAkmal)

*Built with 🔐 for learning — not exploitation*

</div>
