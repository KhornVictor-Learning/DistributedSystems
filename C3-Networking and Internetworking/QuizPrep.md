# Chapter 3: Quiz Preparation & Exam Practice

> **Course Reference**: Distributed Systems (I4-GIC-A / I4-GIC-S1) — ITC (Techno)  
> **Topic**: Networking and Internetworking  
> **Textbook**: Coulouris et al., Chapter 3

---

## ⚡ Section 1: Flashcards & Quick-Recall Concepts

| Flashcard Front | Flashcard Back (Answer & Key Insight) |
| :--- | :--- |
| **What is Network Latency ($L$)?** | The delay after a send operation is executed before the first bit of data begins arriving at the destination computer (time to transfer an empty message). |
| **What is the formula for Message Transmission Time?** | $\text{Transmission Time} = \text{Latency} + \frac{\text{Message Length (bits)}}{\text{Data Transfer Rate (bps)}}$. |
| **What is a Bastion Host?** | A highly secured, hardened server located inside a DMZ between two firewalls that runs public-facing gateway services and application-level proxies. |
| **What are the 3 dimensions of Firewall Security Policies?** | 1. **Service Control** (ports & protocols).<br>2. **Behavior Control** (content & spam inspection).<br>3. **User Control** (identity & access privileges). |
| **What is a CAM Table in an Ethernet Switch?** | Content Addressable Memory table mapping physical switch ports to learned hardware MAC addresses for intelligent frame forwarding. |
| **What are RFC 1918 Private IP address ranges?** | - `10.0.0.0/8`<br>- `172.16.0.0/12`<br>- `192.168.0.0/16` |
| **What is the primary difference between Telnet and SSH?** | Telnet transmits data in unencrypted plaintext (Port 23); SSH provides end-to-end encrypted and authenticated communication (Port 22). |
| **What does IPsec ESP provide?** | Encapsulating Security Payload (ESP) provides both confidentiality (encryption) and data origin authentication/integrity for IP packets. |
| **How does CIDR solve routing table bloat?** | Allows prefix route aggregation (supernetting), summarizing multiple contiguous subnets into a single BGP routing advertisement. |
| **What is Gratuitous ARP used for?** | A broadcast ARP request sent by a host with its own IP address to detect if another host on the local LAN has duplicate IP configuration. |

---

## 🎯 Section 2: Multiple Choice Questions (MCQs)

### Question 1
For small RPC messages in a distributed system, which factor dominates the total transmission time?
- A) Raw channel bandwidth
- B) Network Latency
- C) Cable diameter
- D) RAM bus clock  
**Answer: B**  
*Explanation*: For small payloads, the $\frac{\text{Length}}{\text{Rate}}$ term approaches zero, meaning total transmission time is determined almost entirely by latency (software overheads and routing delays).

---

### Question 2
Which network standard defines Wireless Local Area Networks (Wi-Fi)?
- A) IEEE 802.3
- B) IEEE 802.11
- C) IEEE 802.15.1
- D) IEEE 802.16  
**Answer: B**  
*Explanation*: IEEE 802.11 governs Wi-Fi. IEEE 802.3 is wired Ethernet; IEEE 802.15.1 is Bluetooth; IEEE 802.16 is WiMAX.

---

### Question 3
An Ethernet switch receives a frame destined for a MAC address that does **NOT** exist in its forwarding CAM table. What action does the switch take?
- A) Drops the frame immediately.
- B) Returns an ICMP Destination Unreachable packet.
- C) Floods the frame out of all ports except the arrival port (Unknown Unicast Flooding).
- D) Buffers the frame indefinitely until the device boots up.  
**Answer: C**  
*Explanation*: When a destination MAC is not found in the CAM table, the switch floods the frame across all ports (except the ingress port) so the target machine can receive and respond, allowing the switch to learn its port location.

---

### Question 4
In a screened subnet (DMZ) architecture with two routers in series, why is the internal intranet protected even if the outer router is compromised?
- A) The inner router uses a different electrical voltage.
- B) The inner router is invisible from the public Internet, has strict independent filtering rules, and rejects unacceptable IP packets.
- C) The inner router does not support IP.
- D) The outer router shuts down automatically.  
**Answer: B**  
*Explanation*: The inner router is isolated on the private subnet, possesses its own hardened filtering rules, and hides internal IP addresses, preventing external attackers from penetrating directly to internal databases.

---

### Question 5
Which cryptographic technique enables a receiver to verify both the authentic identity of the sender and that the message was not modified in transit?
- A) Symmetric block cipher without HMAC
- B) Digital Signature
- C) Simple MD5 checksum
- D) Base64 encoding  
**Answer: B**  
*Explanation*: A digital signature encrypts the message hash with the sender's private key. Verifying it with the sender's public key confirms both sender identity (authenticity) and data integrity (hash match).

---

## ✍️ Section 3: Short Answer & Concept Checks

### Q1: Explain why DSL broadband has a maximum operational distance of approximately 5.5 km.
**Sample Answer**:
DSL (Digital Subscriber Line) transmits high-frequency analog signals over existing twisted-pair copper telephone wires. High frequencies attenuate (decay) rapidly over copper due to electrical resistance and capacitance. Beyond approximately 5.5 km (18,000 feet) from the central telephone office switch, high-frequency signal degradation is too severe to sustain reliable data transfer rates.

### Q2: How does Network Address Translation (NAT) enhance security for private network hosts?
**Sample Answer**:
NAT translates private RFC 1918 IP addresses to one or more public IP addresses at the border router. Because external Internet hosts only see the public gateway IP, internal private devices cannot be directly addressed or scanned from the outside world. Inbound packets are only permitted if they correspond to an established, statefully tracked outbound connection initiated by an internal client.

---

## 🧩 Section 4: Numerical Calculation & Scenario Problems

### Problem 1: Message Transmission Time Calculation
**Scenario**: Consider two communication scenarios in a distributed banking system:
- **Scenario A (LAN)**: Latency $L_1 = 2 \text{ ms}$, Data Transfer Rate $R_1 = 1 \text{ Gbps}$ ($10^9 \text{ bits/sec}$).
- **Scenario B (WAN / Trans-oceanic)**: Latency $L_2 = 120 \text{ ms}$, Data Transfer Rate $R_2 = 20 \text{ Mbps}$ ($20 \times 10^6 \text{ bits/sec}$).

Calculate the total transmission time for:
1. A small RPC message of size $S_{\text{small}} = 250 \text{ bytes}$ ($2,000 \text{ bits}$).
2. A large database replication backup of size $S_{\text{large}} = 25 \text{ MB}$ ($25 \times 10^6 \text{ bytes} = 200 \times 10^6 \text{ bits}$).

**Solution**:
1. **For the Small RPC Message (2,000 bits)**:
   - *LAN*:
     $$T = 2\text{ ms} + \frac{2,000}{10^9}\text{ s} = 0.002\text{ s} + 0.000002\text{ s} = 2.002\text{ ms}$$
     *(Latency accounts for $99.9\%$ of the time!)*
   - *WAN*:
     $$T = 120\text{ ms} + \frac{2,000}{20 \times 10^6}\text{ s} = 0.120\text{ s} + 0.0001\text{ s} = 120.1\text{ ms}$$
     *(Latency completely dominates.)*

2. **For the Large Database Backup ($200 \times 10^6$ bits)**:
   - *LAN*:
     $$T = 0.002\text{ s} + \frac{200 \times 10^6}{10^9}\text{ s} = 0.002\text{ s} + 0.200\text{ s} = 0.202\text{ s} \approx 202\text{ ms}$$
   - *WAN*:
     $$T = 0.120\text{ s} + \frac{200 \times 10^6}{20 \times 10^6}\text{ s} = 0.120\text{ s} + 10.0\text{ s} = 10.12\text{ seconds}$$
     *(Transfer rate / Bandwidth dominates.)*

---

### Problem 2: Firewall Rule Evaluation
**Scenario**: An administrator defines the following firewall rules on a corporate border gateway:
1. `ALLOW inbound TCP to 192.168.1.10:443 (HTTPS)`
2. `ALLOW outbound TCP to ANY:80, ANY:443`
3. `DENY ALL OTHER`

Evaluate whether the following operations succeed or fail:
1. An external client opens a web browser to `https://company.com` hosted at `192.168.1.10`.
   - **Result**: **Allowed** (Rule 1 explicitly allows inbound TCP to port 443).
2. An internal employee attempts to establish a Telnet connection (`port 23`) to an external server.
   - **Result**: **Denied** (Rule 2 only permits outbound ports 80 and 443; Rule 3 drops port 23).
3. An external attacker attempts an SSH brute-force login on port 22 of `192.168.1.10`.
   - **Result**: **Denied** (Rule 3 drops all inbound traffic not matching Rule 1).
