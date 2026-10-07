# Chapter 1: Quiz Preparation & Exam Practice

> **Course Reference**: Distributed Systems (I4-GIC-A / I4-GIC-S1) — ITC (Techno)  
> **Topic**: Characterization of Distributed Systems  
> **Textbook**: Coulouris et al., Chapter 1

---

## ⚡ Section 1: Flashcards & Quick-Recall Concepts

| Flashcard Front | Flashcard Back (Answer & Key Insight) |
| :--- | :--- |
| **What is the definition of a Distributed System?** | A system whose components are located at networked computers, communicating and coordinating their actions solely by **passing messages**. |
| **What are the 3 essential characteristics of any Distributed System?** | 1. **Concurrency** of components.<br>2. **Absence of a global clock**.<br>3. **Independent (partial) failures** of components. |
| **What is Access Transparency?** | Enables local and remote resources to be accessed using identical operations (e.g., standard POSIX file I/O over local disk and remote NFS). |
| **What is Location Transparency?** | Enables resources to be accessed without knowledge of their physical or network location (e.g., DNS hostnames or Web URLs). |
| **What is Replication Transparency?** | Allows multiple copies of a resource to be maintained for reliability/performance without clients knowing that multiple copies exist. |
| **What is Failure Transparency?** | Conceals faults from users and applications, allowing operations to proceed despite partial software or hardware crashes. |
| **What is Big-Endian vs. Little-Endian?** | **Big-Endian**: Stores the Most Significant Byte (MSB) at the lowest memory address (used in Network Byte Order).<br>**Little-Endian**: Stores the Least Significant Byte (LSB) at the lowest memory address (used in x86/AMD64). |
| **What is Middleware?** | A software layer situated between applications and underlying OS/network layers that masks platform heterogeneity and provides high-level communication abstractions (e.g., RPC, RMI). |
| **What are the 3 pillars of Security?** | **Confidentiality** (protection against unauthorized disclosure), **Integrity** (protection against unauthorized corruption), and **Availability** (protection against denial of access). |
| **What is Remote Invocation?** | A complete two-way interaction cycle from when a client dispatches a request message to a server until it receives the corresponding reply message. |

---

## 🎯 Section 2: Multiple Choice Questions (MCQs)

### Question 1
Which of the following is **NOT** an intrinsic characteristic of a distributed system according to Coulouris?
- A) Concurrency of components
- B) Independent failure of components
- C) Existence of a shared global clock
- D) Absence of shared physical memory  
**Answer: C**  
*Explanation*: Distributed systems inherently lack a global clock. Network delays and local oscillator drift prevent nodes from having an identical synchronized physical time.

---

### Question 2
In the ISO/ODP model, enabling a user to access files on a remote server using the exact same system calls (`read()`, `write()`) as a local hard drive is an example of:
- A) Location transparency
- B) Access transparency
- C) Scaling transparency
- D) Mobility transparency  
**Answer: B**  
*Explanation*: Access transparency hides differences in data representation and the mechanisms used to access resources, allowing identical operations for local and remote data.

---

### Question 3
An Intel x86 workstation (Little-Endian) stores the 32-bit integer `0x10203040` in memory. If byte address `0x00` is the start, which byte is stored at `0x00`?
- A) `0x10`
- B) `0x20`
- C) `0x30`
- D) `0x40`  
**Answer: D**  
*Explanation*: Little-Endian architectures store the Least Significant Byte (LSB), which is `0x40`, at the lowest memory address (`0x00`).

---

### Question 4
Why is multicasting all client requests to every server in a cluster an **unsatisfactory** mechanism for achieving mobility transparency?
- A) Multicast cannot be implemented over modern switches.
- B) It causes duplicate executions of non-idempotent operations and floods network bandwidth.
- C) Multicast packets do not support IP addresses.
- D) It violates access transparency.  
**Answer: B**  
*Explanation*: Flooding every server consumes massive bandwidth and CPU processing, and if the operation modifies state (e.g., deducting money), all servers might process it redundantly, corrupting data.

---

### Question 5
Which architectural technique is most effective at reducing WAN bandwidth consumption and improving interactive response time for clients?
- A) Distributed Shared Memory (DSM)
- B) Client-side and Proxy Caching
- C) Big-Endian conversion
- D) Increasing NTP polling frequency  
**Answer: B**  
*Explanation*: Caching retains recently requested objects close to the client (in browser storage or local proxy servers), eliminating redundant round-trips over the WAN.

---

### Question 6
Network Time Protocol (NTP) servers are organized in a hierarchical tree. What are the servers called that synchronize directly with atomic clocks or GPS satellites?
- A) Stratum 0
- B) Stratum 1
- C) Stratum 2
- D) Root Proxies  
**Answer: B**  
*Explanation*: Stratum 0 devices are the physical reference clocks (atomic clocks, GPS). Stratum 1 servers are the primary computers directly attached to Stratum 0 devices.

---

## ✍️ Section 3: Short Answer & Concept Checks

### Q1: Distinguish between Masking Failures and Tolerating Failures.
**Sample Answer**:
- **Masking Failures**: The system detects a fault and conceals it from higher-level software or users so that the failure appears not to have occurred. *Example*: TCP detecting a dropped packet via checksum/timeout and silently retransmitting it; or RAID-1 mirroring data to a secondary disk if one disk fails.
- **Tolerating Failures**: The system acknowledges that certain failures cannot be concealed economically, so it is designed to degrade gracefully or report an explicit error that client applications and users are built to accommodate. *Example*: A web browser reporting a "connection timed out" message and allowing the user to click "Retry".

### Q2: Why does an HTTP URL provide only partial location transparency?
**Sample Answer**:
An HTTP URL provides *logical* location transparency because the client does not need to know the physical server chassis, rack, or internal IP address (DNS resolves the host). However, it lacks *full* location transparency because the domain name (e.g., `www.example.com`) and directory path (e.g., `/dept/doc.html`) are tightly bound to a specific host organization and server filesystem hierarchy. If the resource is moved to another server host, the URL breaks unless redirects are explicitly maintained.

### Q3: What is the primary role of Middleware in addressing heterogeneity?
**Sample Answer**:
Middleware acts as an abstraction layer above diverse operating systems, hardware architectures, and network protocols. It masks differences such as byte ordering, data serialization formats, and communication APIs by providing high-level, language-independent interfaces (e.g., RPC/RMI, CORBA, gRPC) allowing distributed components to interact as if they were local.

---

## 🧩 Section 4: Scenario-Based Exam Questions

### Problem 1: Endianness Conversion Trace
**Scenario**: Host A is a Big-Endian mainframe. Host B is a Little-Endian laptop. Host A wants to send a 16-bit port number `8080` (`0x1F90` in hexadecimal) across the network to Host B.
1. What sequence of bytes is transmitted across the physical network link?
2. If Host B reads the bytes directly into memory without using `ntohs()`, what decimal number will Host B perceive?
3. What standard function must Host B invoke to interpret the integer correctly?

**Solution**:
1. Network Byte Order is Big-Endian. Therefore, Host A transmits the bytes as: `0x1F` followed by `0x90`.
2. Host B is Little-Endian. When Host B reads `[0x1F, 0x90]` into a 16-bit integer without conversion, it treats `0x1F` as the LSB and `0x90` as the MSB:
   $$\text{Value} = 0x901F = (9 \times 16^3) + (0 \times 16^2) + (1 \times 16^1) + 15 = 36864 + 0 + 16 + 15 = 36895$$
3. Host B must invoke `ntohs()` (Network to Host Short) to swap the bytes into proper host order (`0x1F90` = 8080).

---

### Problem 2: Transparency Identification in Practice
Identify which form of transparency is primarily exhibited in each of the following scenarios:
1. When a user queries Google Search, their query is serviced by one of tens of thousands of identical server machines without the user being aware of which specific machine was selected.
   - **Answer**: *Replication Transparency* (and *Location Transparency*).
2. A programmer writes `read(fd, buf, count)` to process input from both a local SSD and an Amazon S3 mounted bucket without altering source code.
   - **Answer**: *Access Transparency*.
3. An active database node in a cluster suffers a sudden power loss, and a standby replica assumes control within 200 ms without throwing an unhandled exception to active client transactions.
   - **Answer**: *Failure Transparency*.
4. A smartphone user on a video call moves from a home Wi-Fi network to a 5G cellular network without dropping the active call.
   - **Answer**: *Mobility (Migration) Transparency*.
