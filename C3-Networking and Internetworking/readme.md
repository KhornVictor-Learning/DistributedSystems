# Chapter 3: Networking and Internetworking

> **Course Reference**: Distributed Systems (I4-GIC-A / I4-GIC-S1)  
> **Institution**: Institute of Technology of Cambodia (ITC / Techno)  
> **Lecturer**: Khem Thay  
> **Textbook Reference**: *Distributed Systems: Concepts and Design (5th Edition)*, Chapter 3

---

## 📌 1. Introduction: The Communication Subsystem

In distributed systems, the **communication subsystem** is the collection of hardware transmission media, physical switching devices, and software protocol stacks that interconnect autonomous computers.

### Core Terminology
* **Host**: An end-user computing device (workstation, server, smartphone, IoT device) that executes distributed application programs.
* **Node**: Any computer or switching/routing device connected to a network (including routers, switches, and bridges).
* **Subnet**: A unit of network infrastructure that provides data-link level connectivity directly between attached hosts without intermediate IP routing.
* **Internetwork (The Internet)**: A communication subsystem formed by interconnecting diverse, heterogeneous subnets using routers and gateways, unified by the Internet Protocol (IP).

---

## ⚡ 2. Network Performance Metrics

Network performance fundamentally shapes distributed system latency, responsiveness, and algorithmic efficiency.

```
|<------------------- Total Transmission Time ------------------->|
+----------------------+------------------------------------------+
|  Latency (Overhead)  |      Data Transfer Time = Length / Rate  |
+----------------------+------------------------------------------+
```

### The Fundamental Formula

$$\text{Message Transmission Time} = \text{Latency} + \frac{\text{Message Length (bits)}}{\text{Data Transfer Rate (bps)}}$$

1. **Latency ($L$)**:
   - The time elapsed from when a `send` operation is initiated until the first bit arrives at the destination.
   - Includes:
     - Software protocol stack overhead in the OS kernel.
     - Media propagation delay (speed of light in copper/glass: $\approx 2 \times 10^8 \text{ m/s}$).
     - Queuing and store-and-forward switching delays at intermediate routers.
2. **Data Transfer Rate ($R$)**:
   - The raw transmission speed of the physical link in bits per second (e.g., 1 Gbps = $10^9 \text{ bps}$).
3. **Bandwidth vs. Throughput**:
   - **Bandwidth**: The maximum theoretical capacity of a link.
   - **Throughput**: The actual volume of payload data successfully transferred per unit time under real network conditions (accounting for protocol headers, retransmissions, and congestion).

### Small Messages vs. Large Transfers
- For **small RPC/control messages** (e.g., 64 bytes), transmission time is almost entirely dominated by **latency**.
- For **large bulk file transfers** (e.g., 100 MB), transmission time is dominated by **data transfer rate** (bandwidth).

---

## 🌐 3. Network Classifications

Distributed systems leverage diverse network technologies categorized by geographical scope and transmission media:

| Network Category | Technology Standard | Typical Range | Bandwidth | Latency | Key Use Case |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **WPAN** (Wireless PAN) | Bluetooth (IEEE 802.15.1) | 10 – 30 m | 0.5 – 2 Mbps | 5 – 20 ms | Wearables, wireless peripherals, personal sensors. |
| **LAN** (Local Area Network) | Ethernet (IEEE 802.3) | 1 – 2 km | 10 – 10,000 Mbps | 1 – 10 ms | University campus, office buildings, compute clusters. |
| **WLAN** (Wireless LAN) | Wi-Fi (IEEE 802.11 a/b/g/n/ax) | 150 m – 1.5 km | 11 – 1,000+ Mbps | 5 – 20 ms | Home/office wireless access for mobile laptops/phones. |
| **MAN** (Metropolitan Area) | DSL / Metro Ethernet / ATM | 2 – 50 km | 1 – 600 Mbps | $\approx 10 \text{ ms}$ | City-wide interconnects; telco subscriber loops ($\le 5.5\text{ km}$). |
| **WMAN** (Wireless MAN) | WiMAX (IEEE 802.16) | 5 – 50 km | 1.5 – 20 Mbps | 5 – 20 ms | Broadband wireless metropolitan access. |
| **WAN** (Wide Area Network) | Global IP Routing over Fiber | Worldwide | 0.01 – 600+ Mbps | 100 – 500 ms | Trans-oceanic Internet backbones, multi-cloud links. |
| **WWAN** (Wireless WAN) | Cellular (3G / 4G LTE / 5G) | Cell: 1 – 5 km | 384 kbps – 1 Gbps | 20 – 150 ms | Global mobile smartphone data coverage. |

---

## 🛡️ 4. Network Security & Perimeter Defense

Because distributed systems exchange sensitive information over public packet-switched networks, organizations implement multi-layered perimeter defenses.

### 4.1 Firewalls
A **firewall** is a security gateway running on a network boundary router that monitors, filters, and controls incoming and outgoing IP packets according to organizational security policies.

#### Three Dimensions of Firewall Policy
1. **Service Control**: Controls which protocols/ports are allowed (e.g., allow inbound HTTP port 80 and HTTPS port 443; drop all inbound Telnet port 23 and RPC port 111).
2. **Behavior Control**: Inspects payload content to prevent policy violations (e.g., deep packet inspection filtering out spam, malicious scripts, or data exfiltration).
3. **User Control**: Differentiates privileges based on user roles and authentication (e.g., granting internal administrators access to SSH while blocking guest Wi-Fi users).

---

### 4.2 Bastion Hosts & DMZ Architectures

To prevent an attacker who penetrates an edge router from gaining immediate access to sensitive corporate databases, organizations deploy **Bastion Hosts** within a **Demilitarized Zone (DMZ)**:

```
                  +-----------------------------------+
                  |             INTERNET              |
                  +-----------------------------------+
                                    |
                            [ Router 1 (Filter) ]
                                    |
                    +---------------+---------------+
                    |         DMZ SUBNET            |
                    |   [ Bastion / Web Server ]    |
                    +---------------+---------------+
                                    |
                            [ Router 2 (Filter) ]
                                    |
                  +-----------------------------------+
                  |        INTERNAL INTRANET          |
                  |     [ Database & Internal App ]   |
                  +-----------------------------------+
```

- **Bastion Host**: A heavily secured, hardened computer situated in the DMZ that runs application-level proxies and public-facing services (HTTP, SMTP). It runs minimal operating system services, with all unnecessary ports disabled.
- **Two Routers in Series (Screened Subnet / DMZ)**:
  - If Router 1 is breached or misconfigured, Router 2 (invisible to the outside Internet) continues filtering packets.
  - Internal intranet private IP addresses remain completely unexposed to external hosts.

---

### 4.3 Virtual Private Networks (VPNs)

A **Virtual Private Network (VPN)** extends the protection of a private intranet across untrusted public Internet links by establishing a **cryptographically protected secure tunnel** at the IP layer (e.g., **IPsec**) or Transport layer (e.g., **OpenVPN / WireGuard / TLS**):
- **Encapsulation (Tunneling)**: The original IP packet (with private headers and payload) is wrapped inside a new public IP packet.
- **Confidentiality**: The entire inner packet is encrypted using symmetric ciphers (e.g., AES-GCM).
- **Integrity & Authentication**: Cryptographic HMACs and digital certificates authenticate endpoints and detect any in-transit packet modification.

---

## 🔗 Quick Navigation

* 📖 **[Chapter 3 Study Guide](file:///C:/Desktop/Student%20Online%20%28SO%29/Techno/I4-GIC-A/I4-GIC-S1/Distributed%20Systems/C3-Networking%20and%20Internetworking/StudyGuide.md)** — Deep-dive cryptographic notes, routing hierarchies, switch forwarding mechanics, and complete solutions for **TP05**.
* 📝 **[Chapter 3 Quiz Preparation](file:///C:/Desktop/Student%20Online%20%28SO%29/Techno/I4-GIC-A/I4-GIC-S1/Distributed%20Systems/C3-Networking%20and%20Internetworking/QuizPrep.md)** — Flashcards, transmission time calculations, and exam-style questions.
