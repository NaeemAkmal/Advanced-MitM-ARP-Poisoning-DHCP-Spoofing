<div align="center">

# 🔐 Advanced MitM — ARP Poisoning & DHCP Spoofing

[![Kali Linux](https://img.shields.io/badge/Kali_Linux-557C94?style=for-the-badge&logo=kalilinux&logoColor=white)](https://www.kali.org/)
[![Parrot OS](https://img.shields.io/badge/Parrot_OS-15E0ED?style=for-the-badge&logo=linux&logoColor=black)](https://www.parrotsec.org/)
[![Ettercap](https://img.shields.io/badge/Ettercap-0.8.4-red?style=for-the-badge&logo=gnubash&logoColor=white)](https://www.ettercap-project.org/)
[![Wireshark](https://img.shields.io/badge/Wireshark-1679A7?style=for-the-badge&logo=wireshark&logoColor=white)](https://www.wireshark.org/)
[![EVE-NG](https://img.shields.io/badge/EVE--NG-Network_Sim-orange?style=for-the-badge&logo=cisco&logoColor=white)](https://www.eve-ng.net/)
[![VMware](https://img.shields.io/badge/VMware-607078?style=for-the-badge&logo=vmware&logoColor=white)](https://www.vmware.com/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=for-the-badge)](LICENSE)
[![Status](https://img.shields.io/badge/Status-Completed-brightgreen?style=for-the-badge)]()
[![Educational](https://img.shields.io/badge/Purpose-Educational-purple?style=for-the-badge)]()

<br/>
  **Hands-on demonstration of Man-in-the-Middle attacks using ARP Poisoning and DHCP Spoofing techniques in a controlled virtual lab environment — with full packet-level verification and mitigation strategies.**

<br/>
</div>
---

## 📑 Table of Contents

- [🧠 Overview](#-overview)
- [🖥️ Lab Environment](#️-lab-environment)
- [⚡ Attack 1 — ARP Poisoning MitM](#-attack-1--arp-poisoning-mitm)
- [⚡ Attack 2 — DHCP Spoofing](#-attack-2--dhcp-spoofing)
- [🛡️ Mitigation Strategies](#️-mitigation-strategies)
- [🛠️ Tools and Technologies](#️-tools-and-technologies)
- [⚠️ Disclaimer](#️-disclaimer)

---

## 🧠 Overview

This lab demonstrates two critical **Layer 2 network attacks** commonly exploited in penetration testing:

| 🎯 Attack |   Protocol Abused |   Impact |
|--------|----------------|--------|
| ARP Poisoning | ARP (Layer 2) | Full traffic interception between two hosts |
| DHCP Spoofing | DHCP (Layer 3) | Rogue gateway and DNS assignment to network clients |

Both attacks were executed in an **isolated virtual lab** using VMware and EVE-NG, with full packet-level verification via Wireshark.

---

## 🖥️ Lab Environment

### 🔴 ARP Poisoning Lab

```
┌─────────────────────────────────────────────────┐
│  💀 Attacker    Kali Linux   192.168.23.129       │
│  🖥️  Victim 1    Windows 7    192.168.23.135       │
│  🖥️  Victim 2    Windows 10   192.168.23.136       │
│  ⚙️  Hypervisor  VMware Workstation               │
│  🐛 Tool        Ettercap 0.8.4                   │
└─────────────────────────────────────────────────┘
```

**Network Topology — ARP Poisoning**

![ARP Topology](https://raw.githubusercontent.com/NaeemAkmal/Advanced-MitM-ARP-Poisoning-DHCP-Spoofing/main/Advanced-MitM-ARP-Poisoning-DHCP-Spoofing/arp-poisoning-mitm/network-topology-arp.png)

---

### 🔵 DHCP Spoofing Lab

```
┌─────────────────────────────────────────────────┐
│  🖧  DHCP Server  R1 (EVE-NG)   150.1.7.103/24   │
│  💻 DHCP Client  R2 (EVE-NG)   Dynamic           │
│  🌐 VMNet GW     VMNet1        150.1.7.100/24    │
│  💀 Attacker     Parrot OS     ens34             │
│  ☠️  Rogue Pool   150.1.7.200-250                 │
│  🐛 Tool         Ettercap 0.8.3.1                │
└─────────────────────────────────────────────────┘
```

**Network Topology — DHCP Spoofing**

![DHCP Topology](https://raw.githubusercontent.com/NaeemAkmal/Advanced-MitM-ARP-Poisoning-DHCP-Spoofing/main/Advanced-MitM-ARP-Poisoning-DHCP-Spoofing/dhcp-spoofing/network-topology-gns3.png)

---

## ⚡ Attack 1 — ARP Poisoning MitM

### 🔍 How It Works

```
 NORMAL TRAFFIC FLOW:
   Win7 (135) ─────────────────────────────► Win10 (136)

 AFTER ARP POISONING:
   Win7 (135) ──► 💀 Kali Attacker (129) ──► Win10 (136)
                          |
                  👁️ All traffic intercepted
                     before forwarding
```

The attacker sends **forged ARP Reply** packets to both victims:
-  To Win7 — *"192.168.23.136 is at [Attacker MAC]"*
-  To Win10 — *"192.168.23.135 is at [Attacker MAC]"*

Both victims update their ARP cache with the **wrong MAC**, routing all traffic through the attacker.

---

###  Step 1 — Verify IP Addresses on All Machines

> 📌 Confirm all three machines are on the same subnet before launching.

**🖥️ Victim 2 — Windows 10**

![Win10 ipconfig](https://raw.githubusercontent.com/NaeemAkmal/Advanced-MitM-ARP-Poisoning-DHCP-Spoofing/main/Advanced-MitM-ARP-Poisoning-DHCP-Spoofing/arp-poisoning-mitm/victim2-win10-ipconfig.png)

> Windows 10 — IP: `192.168.23.136` | Gateway: `192.168.23.2`

**🖥️ Victim 1 — Windows 7**

![Win7 ipconfig](https://raw.githubusercontent.com/NaeemAkmal/Advanced-MitM-ARP-Poisoning-DHCP-Spoofing/main/Advanced-MitM-ARP-Poisoning-DHCP-Spoofing/arp-poisoning-mitm/victim1-win7-ipconfig.png)

> Windows 7 — IP: `192.168.23.135`

**💀 Attacker — Kali Linux**

![Kali ip a](https://raw.githubusercontent.com/NaeemAkmal/Advanced-MitM-ARP-Poisoning-DHCP-Spoofing/main/Advanced-MitM-ARP-Poisoning-DHCP-Spoofing/arp-poisoning-mitm/attacker-kali-ip-address.png)

> Kali — `eth0: 192.168.23.129` | `eth1: 150.1.7.101`

---

###  Step 2 — Verify Connectivity

> 📌 All machines must ping each other successfully before the attack.

** Kali pinging both victims**

![Kali ping victims](https://raw.githubusercontent.com/NaeemAkmal/Advanced-MitM-ARP-Poisoning-DHCP-Spoofing/main/Advanced-MitM-ARP-Poisoning-DHCP-Spoofing/arp-poisoning-mitm/connectivity-kali-ping-victims.png)

> Kali reaches Win7 (`.135`) and Win10 (`.136`) — all reachable

** Win7 → Win10**

![Win7 ping Win10](https://raw.githubusercontent.com/NaeemAkmal/Advanced-MitM-ARP-Poisoning-DHCP-Spoofing/main/Advanced-MitM-ARP-Poisoning-DHCP-Spoofing/arp-poisoning-mitm/connectivity-win7-ping-win10.png)

** Win10 → Win7**

![Win10 ping Win7](https://raw.githubusercontent.com/NaeemAkmal/Advanced-MitM-ARP-Poisoning-DHCP-Spoofing/main/Advanced-MitM-ARP-Poisoning-DHCP-Spoofing/arp-poisoning-mitm/connectivity-win10-ping-win7.png)

---

###  Step 3 — Enable IP Forwarding on Kali

> 📌 Without this, victims lose connectivity and the attack is instantly detected.

```bash
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
cat /proc/sys/net/ipv4/ip_forward
# output must be: 1
```

![IP Forwarding](https://raw.githubusercontent.com/NaeemAkmal/Advanced-MitM-ARP-Poisoning-DHCP-Spoofing/main/Advanced-MitM-ARP-Poisoning-DHCP-Spoofing/arp-poisoning-mitm/kali-ip-forwarding-enabled.png)

> Value confirmed as `1` — forwarding active ✔️

---

###  Step 4 — Record Baseline ARP Tables

> 📌 Document the correct ARP state before poisoning for comparison.

** Win7 ARP Table — Before Attack**

![Win7 ARP before](https://raw.githubusercontent.com/NaeemAkmal/Advanced-MitM-ARP-Poisoning-DHCP-Spoofing/main/Advanced-MitM-ARP-Poisoning-DHCP-Spoofing/arp-poisoning-mitm/arp-table-win7-before-attack.png)

> Legitimate MAC addresses mapped correctly

** Kali ARP Table — Before Attack**

![Kali ARP before](https://raw.githubusercontent.com/NaeemAkmal/Advanced-MitM-ARP-Poisoning-DHCP-Spoofing/main/Advanced-MitM-ARP-Poisoning-DHCP-Spoofing/arp-poisoning-mitm/arp-table-kali-before-attack.png)

---

###  Step 5 — Launch Ettercap and Set Targets

```bash
ettercap -G
```

> 📌 Launch Ettercap GUI → interface `eth0` → Unified Sniffing → Host Scan

** Ettercap launched**

![Ettercap launch](https://raw.githubusercontent.com/NaeemAkmal/Advanced-MitM-ARP-Poisoning-DHCP-Spoofing/main/Advanced-MitM-ARP-Poisoning-DHCP-Spoofing/arp-poisoning-mitm/ettercap-gui-launch.png.png)

> Ettercap 0.8.4 started — Unified sniffing on `eth0`

**🔍 Host Scan Results**

![Ettercap host scan](https://raw.githubusercontent.com/NaeemAkmal/Advanced-MitM-ARP-Poisoning-DHCP-Spoofing/main/Advanced-MitM-ARP-Poisoning-DHCP-Spoofing/arp-poisoning-mitm/ettercap-host-scan-results.png)

> 5 hosts discovered on the network

**🎯 Targets Assigned**

![Ettercap targets](https://raw.githubusercontent.com/NaeemAkmal/Advanced-MitM-ARP-Poisoning-DHCP-Spoofing/main/Advanced-MitM-ARP-Poisoning-DHCP-Spoofing/arp-poisoning-mitm/ettercap-targets-assigned.png)

> `192.168.23.135` → Target 1 | `192.168.23.136` → Target 2
>
> Then: **MITM → ARP Poisoning → OK → Start Sniffing** 🚀

---

###  Step 6 — ARP Tables After Poisoning

> 🚨 Both victims now have the attacker's MAC instead of each other's.

** Win7 ARP Table — AFTER (Poisoned)**

![Win7 ARP after](https://raw.githubusercontent.com/NaeemAkmal/Advanced-MitM-ARP-Poisoning-DHCP-Spoofing/main/Advanced-MitM-ARP-Poisoning-DHCP-Spoofing/arp-poisoning-mitm/arp-table-win7-after-poisoning.png)

> Win10's IP (`192.168.23.136`) now maps to **Attacker's MAC** `00:0C:29:D6:40:F4`

**☠️ Win10 ARP Table — AFTER (Poisoned)**

![Win10 ARP after](https://raw.githubusercontent.com/NaeemAkmal/Advanced-MitM-ARP-Poisoning-DHCP-Spoofing/main/Advanced-MitM-ARP-Poisoning-DHCP-Spoofing/arp-poisoning-mitm/arp-table-win10-after-poisoning.png)

> Win7's IP (`192.168.23.135`) now maps to **Attacker's MAC** — poison confirmed 🔴

---

###  Step 7 — Traffic Captured in Wireshark

** ICMP traffic intercepted on Kali**

![Wireshark ICMP](https://raw.githubusercontent.com/NaeemAkmal/Advanced-MitM-ARP-Poisoning-DHCP-Spoofing/main/Advanced-MitM-ARP-Poisoning-DHCP-Spoofing/arp-poisoning-mitm/wireshark-icmp-interception.png)

> ICMP packets between `.135` and `.136` fully visible on Kali — **full interception confirmed** 🔴

** Forged ARP Replies**

![Wireshark ARP forged](https://raw.githubusercontent.com/NaeemAkmal/Advanced-MitM-ARP-Poisoning-DHCP-Spoofing/main/Advanced-MitM-ARP-Poisoning-DHCP-Spoofing/arp-poisoning-mitm/wireshark-arp-forged-replies.png)

> Filter `arp.opcode == 2` — continuous forged ARP replies from attacker

**⚠️ Duplicate IP Detected**

![Wireshark duplicate](https://raw.githubusercontent.com/NaeemAkmal/Advanced-MitM-ARP-Poisoning-DHCP-Spoofing/main/Advanced-MitM-ARP-Poisoning-DHCP-Spoofing/arp-poisoning-mitm/wireshark-duplicate-arp-detected.png)

> `arp.duplicate-address-detected` — confirms ARP conflict caused by poisoning

---

###  ARP Poisoning — Result Summary

| 🔍 Observation |   Before Attack | 😈 After Attack |
|----------------|-----------------|----------------|
| Win7 ARP entry for Win10 | Correct MAC | **Attacker's MAC** |
| Win10 ARP entry for Win7 | Correct MAC | **Attacker's MAC** |
| Wireshark ICMP on Kali | Not visible | **Fully visible** |

---

##  Attack 2 — DHCP Spoofing

### 🔍 How It Works

```
 NORMAL DHCP FLOW:
   Client (R2) ──DISCOVER──► Legitimate Server R1
   Client (R2) ◄──OFFER────  R1 assigns: 150.1.7.111

 DHCP SPOOFING FLOW:
   Client (R2) ──DISCOVER──► R1 [too slow ❌]
                          └──► 💀 Ettercap [wins race ✔️]
   Client (R2) ◄──fake OFFER── Attacker assigns: 150.1.7.200
                                Gateway: Attacker IP ☠️
                                DNS:     Attacker IP ☠️
```

Ettercap **responds faster** than the real server. The client accepts the first OFFER — attacker wins the race.

---

###  Step 1 — Configure R1 as Legitimate DHCP Server

```cisco
Router(config)# interface e0/0
Router(config-if)# ip address 150.1.7.103 255.255.255.0
Router(config-if)# no shutdown
Router(config)# ip dhcp excluded-address 150.1.7.1 150.1.7.110
Router(config)# ip dhcp pool LAN
Router(dhcp-config)# network 150.1.7.0 255.255.255.0
Router(dhcp-config)# default-router 150.1.7.103
Router(dhcp-config)# dns-server 8.8.8.8
```

![R1 DHCP config](https://raw.githubusercontent.com/NaeemAkmal/Advanced-MitM-ARP-Poisoning-DHCP-Spoofing/main/Advanced-MitM-ARP-Poisoning-DHCP-Spoofing/dhcp-spoofing/r1-dhcp-server-configuration.png)

> R1 pool: `150.1.7.0/24` | Gateway: `.103` | DNS: `8.8.8.8`

---

###  Step 2 — Configure R2 as DHCP Client

```cisco
Router(config)# interface e0/0
Router(config-if)# ip address dhcp
Router(config-if)# no shutdown
```

![R2 client config](https://raw.githubusercontent.com/NaeemAkmal/Advanced-MitM-ARP-Poisoning-DHCP-Spoofing/main/Advanced-MitM-ARP-Poisoning-DHCP-Spoofing/dhcp-spoofing/r2-dhcp-client-interface-setup.png)

** R2 receives legitimate IP from R1**

![R2 legitimate IP](https://raw.githubusercontent.com/NaeemAkmal/Advanced-MitM-ARP-Poisoning-DHCP-Spoofing/main/Advanced-MitM-ARP-Poisoning-DHCP-Spoofing/dhcp-spoofing/r2-legitimate-ip-assigned.png)

> `show ip interface brief` — R2 gets `150.1.7.111` from R1 ✔️

---

###  Step 3 — Launch Ettercap on Parrot OS

```bash
sudo ettercap -G
```

> 📌 Set interface to `ens34` → Enable Unified Sniffing

![Ettercap launch](https://raw.githubusercontent.com/NaeemAkmal/Advanced-MitM-ARP-Poisoning-DHCP-Spoofing/main/Advanced-MitM-ARP-Poisoning-DHCP-Spoofing/dhcp-spoofing/ettercap-launch-parrot-os.png)

> Ettercap 0.8.3.1 launched on Parrot OS — `ens34`

---

###  Step 4 — Scan Hosts

![Ettercap hosts](https://raw.githubusercontent.com/NaeemAkmal/Advanced-MitM-ARP-Poisoning-DHCP-Spoofing/main/Advanced-MitM-ARP-Poisoning-DHCP-Spoofing/dhcp-spoofing/ettercap-host-discovery.png)

> 3 hosts found: `150.1.7.100` (VMNet1) | `150.1.7.103` (R1 — DHCP Server) | `150.1.7.111` (R2 — Client)

---

###  Step 5 — Configure DHCP Spoofing Plugin

> 📌 MITM → DHCP Spoofing → Configure rogue pool

```
🎯 IP Pool:    150.1.7.200-250
🌐 Netmask:    255.255.255.0
☠️  DNS Server: 150.1.7.101   ← attacker's IP
```

![DHCP spoof config](https://raw.githubusercontent.com/NaeemAkmal/Advanced-MitM-ARP-Poisoning-DHCP-Spoofing/main/Advanced-MitM-ARP-Poisoning-DHCP-Spoofing/dhcp-spoofing/ettercap-dhcp-spoofing-config.png)

> Rogue DHCP pool ready — Ettercap now listening for DISCOVER packets 

---

###  Step 6 — Trigger DHCP Renewal on R2

```cisco
Router(config)# interface e0/0
Router(config-if)# shutdown
Router(config-if)# no shutdown
```

![R2 DHCP trigger](https://raw.githubusercontent.com/NaeemAkmal/Advanced-MitM-ARP-Poisoning-DHCP-Spoofing/main/Advanced-MitM-ARP-Poisoning-DHCP-Spoofing/dhcp-spoofing/r2-dhcp-request-triggered.png)

> Interface cycled — R2 broadcasts a fresh DHCP DISCOVER 📢

---

### Step 7 — R2 Receives Rogue IP from Attacker

**☠️ Rogue IP Assigned**

![R2 rogue IP](https://raw.githubusercontent.com/NaeemAkmal/Advanced-MitM-ARP-Poisoning-DHCP-Spoofing/main/Advanced-MitM-ARP-Poisoning-DHCP-Spoofing/dhcp-spoofing/r2-rogue-ip-assigned.png)

> `show ip interface brief` — R2 now has `150.1.7.200` via DHCP from **attacker** — **attack successful** 🔴

** Ettercap DHCP Exchange Log**

![Ettercap DHCP log](https://raw.githubusercontent.com/NaeemAkmal/Advanced-MitM-ARP-Poisoning-DHCP-Spoofing/main/Advanced-MitM-ARP-Poisoning-DHCP-Spoofing/dhcp-spoofing/ettercap-dhcp-spoofing-log.png)

```
 DHCP: DISCOVER  from R2
 DHCP spoofing: fake OFFER → offering 150.1.7.200
 DHCP: REQUEST   150.1.7.200
 DHCP spoofing: fake ACK   → assigned 150.1.7.200 ✔️
```

> Full DORA sequence intercepted — DISCOVER → OFFER → REQUEST → ACK ☠️

---

### 📊 DHCP Spoofing — Result Summary

| |   Legitimate (Before) |  After Spoofing |
|--|----------------------|-----------------|
| 🌐 R2 IP Address | `150.1.7.111` from R1 | **`150.1.7.200` from attacker** |
| 🚪 Gateway | `150.1.7.103` | **`150.1.7.101` (attacker)** |
| 🔍 DNS Server | `8.8.8.8` | **`150.1.7.101` (attacker)** |

---

## 🛡️ Mitigation Strategies

### 🔴 Against ARP Poisoning

|  Technique | 💡 How It Helps |
|-------------|---------------|
| Dynamic ARP Inspection (DAI) | Switch validates ARP packets against DHCP snooping table — drops spoofed entries |
| Static ARP entries | Manually pin MAC-to-IP for critical hosts — cannot be overwritten |
| Traffic encryption (TLS/VPN) | Even if intercepted, all data remains unreadable |
| VLAN segmentation | Limits which hosts receive each other's ARP broadcasts |
| ARP monitoring (XArp) | Detects duplicate IP-MAC mappings in real time |

### 🔵 Against DHCP Spoofing

|  Technique | 💡 How It Helps |
|-------------|---------------|
| DHCP Snooping | Only trusted uplink ports can send DHCP replies — blocks rogue servers |
| 802.1X Port Authentication | Unauthorized devices cannot connect to the network |
| Static IP for critical devices | Routers and servers never use DHCP — immune to spoofing |
| IDS/IPS monitoring | Detects multiple DHCP OFFER packets from unknown MACs |
| VLAN isolation | Confines DHCP traffic to specific segments |

---

## 🛠️ Tools and Technologies

| 🔧 Tool |  Version | 🎯 Purpose |
|--------|-----------|-----------|
| Ettercap | 0.8.4 / 0.8.3.1 | ARP poisoning and DHCP spoofing |
| Wireshark | Latest | Packet capture and analysis |
| Kali Linux | 2024 | Primary attack platform |
| Parrot OS | Latest | Secondary attack platform |
| VMware Workstation | — | Virtual machine hypervisor |
| EVE-NG | Latest | Network simulation (routers) |
| Windows 7 / 10 | — | Victim machines |

---

## ⚠️ Disclaimer

This project is strictly for **educational and research purposes** in a controlled, isolated lab environment. All techniques demonstrated here are performed on systems owned and operated by the researcher. Unauthorized use of these techniques on any network or system without explicit written permission is **illegal and unethical**. The author takes no responsibility for misuse of this information.

---
## **Connect with me**
[**Naeem Akmal on LinkedIn**](https://www.linkedin.com/in/naeemakmal15)

<div align="center">

![](https://capsule-render.vercel.app/api?type=waving&color=0:0f3460,50:16213e,100:1a1a2e&height=100&section=footer&animation=fadeIn)

**Naeem Akmal** · Cyber Security Researcher

[![GitHub](https://img.shields.io/badge/GitHub-NaeemAkmal-black?style=flat-square&logo=github)](https://github.com/NaeemAkmal)

*🔒 Built for learning — not exploitation*

</div>
