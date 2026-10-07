# Chapter 3: Study Guide & Lab Solutions

> **Course**: Distributed Systems (I4-GIC-A / I4-GIC-S1) — ITC (Techno)  
> **Topic**: Networking and Internetworking  
> **Textbook**: Coulouris et al., *Distributed Systems: Concepts and Design (5th Ed.)*, Chapter 3  
> **Included Labs**: Full solutions for **TP05**

---

## 📑 Table of Contents
1. [Core Networking & Cryptographic Theory](#1-core-networking--cryptographic-theory)
   - [The 5 Core Cryptographic Techniques](#the-5-core-cryptographic-techniques)
   - [Ethernet Switching & MAC Learning](#ethernet-switching--mac-learning)
   - [Internet Routing Scalability (CIDR & BGP)](#internet-routing-scalability-cidr--bgp)
   - [Public vs. Private IP Addresses (RFC 1918 & NAT)](#public-vs-private-ip-addresses-rfc-1918--nat)
2. [TP05: Complete Solutions & Explanations](#tp05-complete-solutions--explanations)
3. [Exam Review Summary & Key Formulas](#exam-review-summary--key-formulas)

---

## 1. Core Networking & Cryptographic Theory

### The 5 Core Cryptographic Techniques

Distributed systems depend on five foundational cryptographic primitives to protect message passing:

```
+------------------------------------------------------------------------+
|                      5 CORE CRYPTOGRAPHIC TECHNIQUES                   |
+------------------------------------------------------------------------+
| 1. Symmetric Encryption   | Same shared key for encryption & decryption|
| 2. Asymmetric Encryption  | Key pair: Public key (encrypt), Private key|
| 3. Hash Functions         | One-way fixed-length message digest        |
| 4. Digital Signatures     | Hash signed by Private key (Auth + Non-Rep)|
| 5. Digital Certificates   | CA binds a Public key to an identity (PKI) |
+------------------------------------------------------------------------+
```

1. **Symmetric Key Encryption (Secret-Key)**:
   - Uses the identical secret key ($K$) for both encryption and decryption ($C = E_K(M)$, $M = D_K(C)$).
   - *Algorithms*: **AES** (Advanced Encryption Standard), ChaCha20.
   - *Characteristics*: Extremely fast; ideal for high-volume data stream encryption. Requires a secure channel for initial key exchange.
2. **Asymmetric Key Encryption (Public-Key)**:
   - Uses a mathematically linked key pair: a **Public Key** ($K_{\text{pub}}$) openly distributed to the world, and a **Private Key** ($K_{\text{priv}}$) kept strictly secret.
   - Anyone can encrypt using $K_{\text{pub}}$, but only the owner of $K_{\text{priv}}$ can decrypt.
   - *Algorithms*: **RSA**, **ECC** (Elliptic Curve Cryptography).
   - *Characteristics*: Computationally intensive; primarily used for digital signatures and secure key exchange (e.g., TLS handshake).
3. **Cryptographic Hash Functions (Message Digests)**:
   - Mathematical one-way function converting an arbitrary-length message $M$ into a fixed-length string $h = H(M)$ (e.g., 256 bits).
   - *Properties*: Deterministic, fast, preimage resistant (cannot deduce $M$ from $h$), and collision resistant (cannot find two distinct messages with identical hashes).
   - *Algorithms*: **SHA-256**, SHA-3.
4. **Digital Signatures**:
   - Provides authenticity, message integrity, and non-repudiation.
   - The sender computes hash $h = H(M)$, then encrypts $h$ using their **Private Key**: $\text{Sig} = E_{K_{\text{priv}}}(h)$.
   - Any receiver verifies by computing $H(M)$ independently and checking if it matches $D_{K_{\text{pub}}}(\text{Sig})$.
5. **Digital Certificates & Public Key Infrastructure (PKI)**:
   - Solves the impersonation problem: How can a client know that a public key actually belongs to `bank.com` and not an attacker?
   - A trusted **Certificate Authority (CA)** inspects the organization's legal credentials, generates an **X.509 Certificate** containing the organization's public key and domain name, and signs it with the CA's private key. Browsers verify the signature using built-in root CA certificates.

---

### Ethernet Switching & MAC Learning

An **Ethernet switch** operates at Layer 2 (Data Link Layer) of the OSI model:
- Maintains a **Forwarding Table (CAM Table)** mapping MAC addresses to physical switch ports.
- **Learning Process**: When a frame arrives on Port 1 with Source MAC `AA:BB:CC:...`, the switch records `Port 1 -> AA:BB:CC:...`.
- **Forwarding Process**:
  - If Destination MAC is in the CAM table, the switch forwards the frame directly to that specific output port (micro-segmentation).
  - If Destination MAC is unknown (or is broadcast `FF:FF:FF:FF:FF:FF`), the switch floods the frame across all ports except the arrival port.
- **Benefit**: Replaces legacy shared-collision Ethernet hubs with dedicated collision-free, full-duplex communication paths.

---

### Internet Routing Scalability (CIDR & BGP)

With billions of connected devices, an Internet router cannot possibly maintain a separate routing table entry for every individual IP address:
1. **Hierarchical Routing & Autonomous Systems (AS)**:
   - The Internet is partitioned into thousands of Autonomous Systems (ISPs, universities, tech giants).
   - Internal routing within an AS uses **IGP** (OSPF, IS-IS). Inter-domain routing between ASes uses **BGP** (Border Gateway Protocol).
2. **CIDR (Classless Inter-Domain Routing) & Route Aggregation**:
   - Replaced rigid Class A, B, and C networks with arbitrary prefix lengths (`/24`, `/16`).
   - Multiple smaller network prefixes are aggregated into a single summarized routing entry (prefix supernetting), reducing global BGP routing table size by orders of magnitude.
3. **Default Routes ($0.0.0.0/0$)**:
   - Edge and departmental routers maintain a small routing table and forward all unknown outbound packets to a designated **Default Gateway**.

---

## 📝 TP05: Complete Solutions & Explanations

### Question 1: What is a gateway?
**Answer**:
- **Definition**:
  A **gateway** is a network node or hardware/software device that acts as an entrance and translator between two distinct, often heterogeneous networks operating with different protocols, addressing schemes, or administrative boundaries.
- **Types of Gateways in Distributed Systems**:
  1. *Default Gateway / Router*: Connects a local LAN subnet to the broader Internet, performing IP packet forwarding and NAT translation.
  2. *Application-Level Gateway (Proxy Gateway)*: Operates at Layer 7, translating between different application protocols (e.g., an API Gateway translating external REST/JSON calls to internal gRPC or SOAP services).
  3. *Security Gateway (Firewall)*: A boundary gateway monitoring, filtering, and enforcing security policies between an internal intranet and external networks.

---

### Question 2: Explain the main 5 cryptographic techniques in detail.
**Answer**:
*(Refer to Section 1 for in-depth foundational details)*
1. **Symmetric Encryption**: Shared single secret key (AES-256) used for high-speed encryption and decryption. Both parties must securely agree on the secret key beforehand.
2. **Asymmetric Encryption**: Public/private key pair (RSA, ECC). Public key encrypts; private key decrypts. Eliminates the secret key exchange vulnerability over open networks.
3. **Cryptographic Hash Functions**: One-way irreversible mathematical functions (SHA-256) transforming arbitrary inputs into fixed-size digests to detect data corruption or unauthorized tampering.
4. **Digital Signatures**: Combining hashing with private key signing. The sender encrypts the document hash with their private key, proving identity, ensuring data integrity, and providing non-repudiation.
5. **Digital Certificates**: X.509 certificates signed by trusted Certificate Authorities (CAs) that bind an entity's identity (domain name) to its public key, preventing Man-in-the-Middle (MITM) impersonation.

---

### Question 3: Explain the differences between public and private IP addresses.
**Answer**:

| Attribute | Public IP Address | Private IP Address (RFC 1918) |
| :--- | :--- | :--- |
| **Routability** | Globally unique; routable across the public Internet. | Non-routable on the public Internet; dropped by ISP edge routers. |
| **Address Space** | Assigned by IANA and Regional Internet Registries (RIRs). | Reserved blocks: `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`. |
| **Cost & Scarcity** | Paid resource; extremely scarce in IPv4 ($2^{32} \approx 4.3 \text{ billion}$). | Free; reusable infinitely across isolated private networks. |
| **Internet Access** | Communicates directly with Internet servers. | Requires **Network Address Translation (NAT)** on a gateway router to access the Internet. |
| **Security** | Directly exposed to global Internet port scans and attacks. | Naturally shielded behind NAT/Firewall; not directly addressable from outside. |

---

### Question 4: What are the differences between Telnet and SSH?
**Answer**:

| Feature | Telnet (RFC 854) | SSH — Secure Shell (RFC 4251) |
| :--- | :--- | :--- |
| **Default Port** | Port 23 | Port 22 |
| **Security / Encryption** | Transmits all traffic (passwords, commands) in **plaintext**. | Encrypts all communication using **strong symmetric/asymmetric encryption**. |
| **Vulnerability to Sniffing** | Vulnerable: Anyone running Wireshark can capture passwords immediately. | Immune: Sniffed packets reveal only encrypted ciphertext. |
| **Authentication** | Basic plaintext username/password. | Robust public-key cryptography, key pairs, and certificates. |
| **Integrity** | No cryptographic data integrity verification. | Uses HMACs to detect packet tampering or replay attacks. |
| **Modern Status** | Deprecated and forbidden in modern enterprise production networks. | Standard industry protocol for remote server administration. |

---

### Question 5: Explain VPNs in detail.
**Answer**:
- **Definition**:
  A Virtual Private Network (VPN) creates a cryptographically encrypted communication tunnel over an unsecure public network (such as the Internet), allowing remote users or branch offices to access an internal private intranet as though they were physically connected to the local LAN.
- **Architectural Mechanics (Tunneling & Encapsulation)**:
  1. *Encapsulation*: When a host transmits a packet destined for a private address (`10.0.1.5`), the VPN software wraps the entire original IP packet inside a new public IP packet addressed to the corporate VPN Gateway.
  2. *Encryption*: The inner packet (headers and payload) is encrypted using AES-GCM, rendering it unreadable to ISP routers.
  3. *Transmission & Decapsulation*: The public packet travels across the Internet. Upon arrival at the corporate VPN Gateway, the gateway decrypts the payload, strips the outer headers, and forwards the original packet onto the internal LAN.
- **Major VPN Architectures**:
  - **Remote Access VPN**: Individual employees connect laptops to the corporate network using software clients (OpenVPN, WireGuard).
  - **Site-to-Site VPN**: Two edge routers connect two geographically distant enterprise offices, seamlessly merging their private subnets.
- **Protocols**:
  - **IPsec**: Operates at Layer 3 (Network Layer) using **ESP** (Encapsulating Security Payload) for encryption and integrity, and **AH** (Authentication Header) for authentication.

---

### Question 6: Internet routing scheme for scaling without storing all destinations.
**Answer**:
The Internet manages scalability through a **hierarchical, delegated routing scheme**:
1. **Autonomous Systems (AS)**:
   - The Internet is partitioned into thousands of independently managed administrative domains called Autonomous Systems (e.g., AS15169 for Google, AS701 for Verizon).
2. **Hierarchical Separation of Routing**:
   - Routers *inside* an AS only maintain routes for internal subnets using Interior Gateway Protocols (OSPF / IS-IS).
   - Only specialized **Border Routers** exchange inter-AS routes using the **Border Gateway Protocol (BGP)**.
3. **CIDR (Classless Inter-Domain Routing) Prefix Aggregation**:
   - Border routers aggregate hundreds of contiguous subnets into a single summarized CIDR route announcement (e.g., advertising one `/16` block instead of 256 individual `/24` subnets).
4. **Default Routes ($0.0.0.0/0$)**:
   - The overwhelming majority of enterprise and consumer routers do not store full BGP routing tables; they simply store local subnet routes and forward all other traffic to the upstream ISP's default gateway. Only Tier-1 backbone routers maintain full "default-free" tables ($\approx 950,000$ BGP prefixes).

---

### Question 7: What is the task of an Ethernet switch? Explain in detail.
**Answer**:
An Ethernet switch is a Layer 2 (Data Link Layer) networking device whose primary task is to forward Ethernet frames between connected computers based on their **MAC addresses**, isolating collision domains:
1. **Address Learning**:
   - The switch maintains a **CAM (Content Addressable Memory) table**.
   - When a frame enters a physical port, the switch inspects the **Source MAC address** and records the mapping: `[Source MAC -> Ingress Port]` with a timestamp timer.
2. **Intelligent Frame Forwarding / Filtering**:
   - The switch examines the **Destination MAC address**:
     - *Known Unicast*: If the Destination MAC exists in its CAM table, the frame is directed *only* to the specific corresponding port.
     - *Unknown Unicast*: If the Destination MAC is not in the CAM table, the switch floods the frame across all active ports except the sender port.
     - *Broadcast / Multicast*: Frames destined for `FF:FF:FF:FF:FF:FF` are flooded out all ports except the arrival port.
3. **Collision Domain Segmentation**:
   - Unlike legacy Ethernet hubs (which shared bandwidth and caused collisions), each switch port represents an isolated collision domain operating in **full-duplex mode**, allowing simultaneous sending and receiving at maximum link speed.

---

### Question 8: How can we be sure that no two computers on the Internet have the same IP address?
**Answer**:
Global IP address uniqueness is guaranteed through a strict **hierarchical governance and allocation process**:
1. **Top-Level Governance (IANA)**:
   - The **Internet Assigned Numbers Authority (IANA)** maintains the master authoritative registry of global IP addresses.
2. **Regional Internet Registries (RIRs)**:
   - IANA delegates large address blocks to five Regional Registries:
     - **APNIC** (Asia-Pacific)
     - **ARIN** (North America)
     - **RIPE NCC** (Europe & Middle East)
     - **LACNIC** (Latin America)
     - **AFRINIC** (Africa)
3. **Internet Service Providers (ISPs) & Enterprises**:
   - RIRs allocate smaller CIDR blocks to National Internet Registries (NIRs) and ISPs (Local Internet Registries).
   - ISPs assign specific, unique public IP subnets to organizational customers.
4. **Dynamic & Local Collision Detection**:
   - Within local networks, routers use **DHCP (Dynamic Host Configuration Protocol)** with lease management to ensure each host receives a distinct address.
   - Hosts run **Gratuitous ARP (Address Resolution Protocol)** upon booting: the host broadcasts an ARP query for its newly assigned IP. If another machine responds, an IP conflict is flagged and the address is rejected.

---

### Question 9: Best technique to calculate message transmission time? Explain the procedure.
**Answer**:
1. **The Mathematical Model**:
   $$\text{Transmission Time } (T) = \text{Latency } (L) + \frac{\text{Message Size } (S)}{\text{Data Transfer Rate } (R)}$$
2. **Step-by-Step Practical Measurement Procedure**:
   - **Step 1: Measure Latency ($L$)**:
     - Transmit an empty or minimal-size control probe (e.g., using ICMP `ping` or empty TCP SYN) between the two distributed hosts.
     - Measure the Round-Trip Time ($RTT$) over multiple samples and calculate one-way network latency:
       $$L \approx \frac{RTT}{2} + \text{Kernel Software Overhead}$$
   - **Step 2: Measure Available Transfer Rate ($R$)**:
     - Execute a standardized throughput test between the hosts using network benchmarking tools like `iperf3` to determine effective data transfer rate $R$ in bits/second under current link congestion.
   - **Step 3: Account for Packetization Overhead & MTU**:
     - Determine the payload size $S$. If $S > \text{MTU}$ (Maximum Transmission Unit, typically 1500 bytes for standard Ethernet), calculate the number of packets:
       $$N_{\text{packets}} = \left\lceil \frac{S}{\text{MTU} - \text{Header Size}} \right\rceil$$
     - Each packet adds Layer 2 (14 bytes), IP (20 bytes), and TCP (20 bytes) headers.
   - **Step 4: Compute Total Transmission Time**:
     - Substitute measured latency $L$, total transmitted bits (payload + headers), and measured rate $R$ into the formula.

---

### Question 10: Describe possible occurrences of each of the main types of security threats on the Internet.
**Answer**:
1. **Threats to Processes**:
   - *Masquerading / Spoofing*: An attacker crafts malicious IP packets or spins up a rogue web server claiming to be `paypal.com` to harvest credentials.
   - *Remote Code Execution / Trojan Horses*: An adversary exploits an unpatched buffer overflow in a web server (e.g., Log4j or Apache vulnerability) to gain root shell execution on the host.
2. **Threats to Communication Channels**:
   - *Eavesdropping (Sniffing)*: An attacker connected to an unencrypted public coffee shop Wi-Fi runs packet analyzers (Wireshark) to capture plaintext passwords and session cookies transmitted over HTTP.
   - *Message Tampering (Integrity Violation)*: An in-path malicious router alters packet payloads in flight, such as injecting malicious JavaScript into unencrypted HTTP webpages.
   - *Replay Attacks*: An attacker records an authenticated banking transfer request packet and re-transmits it 100 times, causing the server to process repeated fraudulent transfers.
3. **Denial of Service (DoS / DDoS)**:
   - *TCP SYN Flooding*: Attackers flood a server with spoofed TCP SYN handshake packets without completing the third ACK step, exhausting the server's connection backlog queue (`listen()`) and preventing legitimate users from connecting.
   - *Volumetric Distributed Reflection Attacks (DDoS)*: Attackers utilize a botnet to send spoofed UDP queries to vulnerable DNS/NTP servers, reflecting massive amplified traffic (tens of gigabits/sec) onto a victim's IP, overwhelming its ISP link.

---

## 📊 Exam Review Summary & Key Formulas

* **Message Transmission Time Formula**:
  $$T = L + \frac{S}{R}$$
* **Network Hierarchy Quick Summary**:
  - *PAN*: $\le 30\text{ m}$ (Bluetooth)
  - *LAN*: $\le 2\text{ km}$ (Ethernet, 10 Mbps–10 Gbps)
  - *MAN*: $\le 50\text{ km}$ (DSL/ATM/Fiber, $\approx 10\text{ ms}$)
  - *WAN*: Worldwide (IP routing, 100–500 ms)
* **Firewall Triad**:
  $$\text{Service Control} \quad|\quad \text{Behavior Control} \quad|\quad \text{User Control}$$
