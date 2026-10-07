# Chapter 1: Study Guide & Lab Solutions

> **Course**: Distributed Systems (I4-GIC-A / I4-GIC-S1) — ITC (Techno)  
> **Topic**: Characterization of Distributed Systems  
> **Textbook**: Coulouris et al., *Distributed Systems: Concepts and Design (5th Ed.)*, Chapter 1  
> **Included Labs**: Full solutions for **TP01** & **TP02**

---

## 📑 Table of Contents
1. [Core Theoretical Notes & Concept Deep Dive](#1-core-theoretical-notes--concept-deep-dive)
   - [The 3 Intrinsic Characteristics](#the-3-intrinsic-characteristics)
   - [The 8 Forms of Transparency with Engineering Examples](#the-8-forms-of-transparency-with-engineering-examples)
   - [Heterogeneity and Byte Ordering (Endianness)](#heterogeneity-and-byte-ordering-endianness)
   - [Client-Server vs. Peer-to-Peer vs. Cloud Computing](#client-server-vs-peer-to-peer-vs-cloud-computing)
2. [TP01: Complete Solutions & Explanations](#tp01-complete-solutions--explanations)
3. [TP02: Complete Solutions & Explanations](#tp02-complete-solutions--explanations)
4. [Exam Review Summary & Key Formulas](#exam-review-summary--key-formulas)

---

## 1. Core Theoretical Notes & Concept Deep Dive

### The 3 Intrinsic Characteristics

1. **Concurrency of Components**:
   - Multiple computers simultaneously execute distinct programs sharing network and software resources.
   - Requires mutual exclusion and locking to prevent race conditions on shared data.
2. **Absence of a Global Clock**:
   - Communication occurs solely through network messages with unpredictable transmission delays.
   - No two computers have identically synchronized physical quartz clocks due to drift ($10^{-6}$ seconds/sec).
   - Time coordination relies on algorithms like NTP (Network Time Protocol) or logical clocks (Lamport, Vector Clocks).
3. **Independent (Partial) Failures**:
   - Hardware or software faults may terminate a node or isolate a network segment while remaining nodes continue running normally.
   - Other nodes cannot immediately distinguish between a slow network link, a crashed process, or a severed physical cable.

---

### The 8 Forms of Transparency with Engineering Examples

*Transparency* is the concealment of separation from users and application programmers.

| Transparency Type | Definition | Concrete Engineering Example |
| :--- | :--- | :--- |
| **Access** | Identical operations are used to access local and remote resources. | In POSIX systems, `read()` and `write()` system calls operate seamlessly on local ext4 disks and remote **NFS** (Network File System) mounts. |
| **Location** | Resources are accessed without knowledge of their physical location or network host. | Accessing `https://www.google.com` or `urn:isbn:0451450523` without knowing which physical data center or IP address hosts the file. |
| **Concurrency** | Multiple processes operate concurrently on shared resources without mutual interference. | Multiple banking clients withdrawing cash simultaneously from accounts hosted in a relational database using **ACID transactions** and row locks. |
| **Replication** | Multiple instances of resources are maintained to improve reliability and performance without clients knowing about replicas. | A client reads a file from a **CDN** (e.g., Cloudflare) or distributed key-value store (e.g., Amazon DynamoDB) unaware of which replica responded. |
| **Failure** | Conceals hardware/software faults, enabling tasks to complete despite partial system failures. | When a primary database node crashes, an automated failover promotes a secondary replica while the client's connection transparently retries. |
| **Mobility (Migration)** | Resources and clients can move within the system without affecting ongoing operations. | A mobile phone moves between Wi-Fi access points using **Mobile IP**, maintaining an active TCP session without re-establishing the socket. |
| **Performance** | The system can be dynamically reconfigured or load-balanced as workload fluctuates without altering application code. | An AWS Auto Scaling group spins up additional virtual server instances behind an **Application Load Balancer (ALB)** as CPU traffic spikes. |
| **Scaling** | The system can expand in scale (users, storage, nodes) without modifying algorithms or system structure. | The **Domain Name System (DNS)** scales from thousands to billions of hostnames by utilizing a hierarchical, delegated distributed tree. |

---

### Heterogeneity and Byte Ordering (Endianness)

When integers are transmitted across a network between computers with different processor architectures:
- **Big-Endian (Network Byte Order)**: The Most Significant Byte (MSB) is stored at the lowest memory address (first in sequence). Common in network protocols (TCP/IP) and ARM/SPARC in big-endian mode.
- **Little-Endian**: The Least Significant Byte (LSB) is stored at the lowest memory address. Common in Intel/AMD x86-64 architectures.

#### Concrete Example: 32-bit Integer `0x12345678`

```
Memory Address:       0x00     0x01     0x02     0x03
                     +--------+--------+--------+--------+
Big-Endian (MSB 1st):|  0x12  |  0x34  |  0x56  |  0x78  |  <- Transmitted as 12 34 56 78
                     +--------+--------+--------+--------+

Little-Endian (LSB): |  0x78  |  0x56  |  0x34  |  0x12  |  <- In x86 memory
                     +--------+--------+--------+--------+
```

To resolve this heterogeneity, network software utilizes standard conversion functions:
- `htonl()` (Host to Network Long) / `htons()` (Host to Network Short)
- `ntohl()` (Network to Host Long) / `ntohs()` (Network to Host Short)

---

### Client-Server vs. Peer-to-Peer vs. Cloud Computing

```
[ Client-Server ]                [ Peer-to-Peer (P2P) ]              [ Cloud Computing ]
  C1    C2    C3                      PeerA <---> PeerB             Client -> [ Elastic Pool ]
    \   |   /                             \       /                          [ VM1 | VM2 | VM3 ]
     v  v  v                               v     v                           [   Storage SAN   ]
    [ SERVER ]                        PeerC <---> PeerD                     Pay-as-you-go Utility
```

| Dimension | Client-Server | Peer-to-Peer (P2P) | Cloud Computing |
| :--- | :--- | :--- | :--- |
| **Role Distribution** | Asymmetric: Clients request, dedicated Servers respond. | Symmetric: Every peer acts as both client (servant) and server. | Multi-tier virtualization: Cloud provider manages elastic server pools. |
| **Scalability** | Server can become a performance and bandwidth bottleneck. | Highly scalable: As user count increases, aggregate capacity increases. | Dynamically elastic: Auto-scales on demand by provisioning virtual instances. |
| **Fault Tolerance** | Single point of failure unless redundant servers/clusters are deployed. | High resilience: Loss of individual peers does not halt the network. | Managed high availability backed by Service Level Agreements (SLAs). |

---

## 📝 TP01: Complete Solutions & Explanations

### Question 1: What happens if a request needs to be forwarded from a server to another server?
**Answer**:
In multi-tier distributed architectures (e.g., 3-tier web applications or microservices), a server frequently acts as an intermediary or client to downstream backend servers:
1. **Role Reversal**: The receiving server pauses or continues its client handling, acts as a **client** to the downstream server, and initiates a secondary remote invocation (e.g., over RPC, gRPC, or HTTP).
2. **Increased Latency**: Total request turnaround time increases by the sum of internal network transfer times and secondary processing latencies ($\text{Latency}_{\text{total}} = \text{Latency}_{C \to S1} + \text{Processing}_{S1} + \text{Latency}_{S1 \to S2} + \text{Processing}_{S2}$).
3. **State & Connection Management**: The intermediary server must maintain context (session state, correlation ID, transaction context) to pair the downstream response back to the original client request.
4. **Failure Propagation**: If the secondary server crashes or experiences a network partition, the first server must detect the timeout and decide whether to retry, fail over, or return an error response (e.g., HTTP `504 Gateway Timeout`) to the client.
5. **Security Propagation**: The user's authentication and authorization credentials must be securely passed down (e.g., via OAuth Bearer Tokens or mutual TLS) so the secondary server can enforce access permissions.

---

### Question 2: What is peer-to-peer computing? Give an example.
**Answer**:
- **Definition**: Peer-to-Peer (P2P) is a distributed architecture where participating nodes (peers) share equal privileges and responsibilities. All nodes run identical software interfaces and interact cooperatively without requiring centralized coordination or dedicated server hardware. Each node simultaneously acts as both a **client** (requesting data) and a **server** (serving data).
- **Key Advantage**: Aggregate system bandwidth and storage capacity scale organically as more peers join the network.
- **Example**: **BitTorrent**. When downloading a file, a user's client downloads file chunks from multiple other participating peers while simultaneously uploading already-downloaded chunks to peers seeking them.

---

### Question 3: Explain the client-server program between a web browser and a web server. Give an example.
**Answer**:
- **Interaction Mechanism**:
  1. **User Action**: The user enters a URL into the browser (e.g., `http://www.itc.edu.kh/index.html`).
  2. **Resolution & Connection**: The browser queries DNS to resolve the hostname `www.itc.edu.kh` into an IP address, then establishes a **TCP connection** to the server on port 80 (or port 443 for HTTPS).
  3. **Client Request**: The browser (client) sends an HTTP formatted text request:
     ```http
     GET /index.html HTTP/1.1
     Host: www.itc.edu.kh
     User-Agent: Mozilla/5.0
     ```
  4. **Server Processing**: The web server process (e.g., Nginx, Apache) parses the request, retrieves `/index.html` from its local storage, and constructs an HTTP response.
  5. **Server Reply**: The server transmits the response back over the established TCP socket:
     ```http
     HTTP/1.1 200 OK
     Content-Type: text/html
     Content-Length: 4210

     <!DOCTYPE html><html>...</html>
     ```
  6. **Rendering**: The browser parses the HTML, issues subsequent requests for embedded stylesheets, images, and scripts, and renders the webpage.

---

### Question 4: Explain the 2 alternative ways for byte ordering of integers.
**Answer**:
When representing multi-byte binary numbers (such as 16-bit, 32-bit, or 64-bit integers) in memory:
1. **Big-Endian**:
   - The most significant byte (MSB) is stored at the lowest memory address (the first byte).
   - *Analogy*: Reading numbers left-to-right in natural order (e.g., thousands digit first).
   - Used by the Internet Protocol suite as standard **Network Byte Order**.
2. **Little-Endian**:
   - The least significant byte (LSB) is stored at the lowest memory address.
   - *Analogy*: Storing units first, then tens, then hundreds.
   - Used natively by Intel x86 and AMD64 processors.
- **Why it matters**: If an x86 machine sends `0x00000001` in native byte order to a big-endian machine without translation, the recipient reads it as `0x01000000` (16,777,216). Systems use functions like `htonl()` and `ntohl()` to guarantee interoperability.

---

### Question 5: Find 1 example of each transparency.
**Answer**:
1. **Access Transparency**: NFS (Network File System) — commands like `cat`, `cp`, and system calls `open()`, `read()` execute identically on local disk files and remote server files.
2. **Location Transparency**: URLs — accessing `http://www.bbc.com/news` does not require knowing which physical server, city, or IP hosts the content.
3. **Concurrency Transparency**: Multi-user Database Systems (e.g., PostgreSQL, Oracle) — two users booking airplane seats concurrently are managed with locking so neither corrupts the other's booking.
4. **Replication Transparency**: Distributed Key-Value Stores (e.g., Cassandra) or Google Search — queries hit replicated data clusters without users knowing how many replicas exist or which one answered.
5. **Failure Transparency**: Redundant Web Clusters behind a reverse proxy (e.g., HAProxy) — if a backend web server node crashes, HAProxy reroutes pending requests to healthy nodes without user interruption.
6. **Mobility Transparency**: Mobile Telephony (GSM / LTE) — a mobile phone in a moving train shifts from one cell tower to another without dropping an ongoing voice call.
7. **Performance Transparency**: Dynamic Load Balancing — an elastic cloud database automatically redistributes partitions across added CPU nodes when traffic doubles, preserving query latency.
8. **Scaling Transparency**: Domain Name System (DNS) — new top-level domains, subdomains, and millions of hosts are continuously added without re-architecting DNS client resolution software.

---

### Question 6: Find functional and non-functional properties of a system with examples (QoS).
**Answer**:
- **Functional Properties**:
  - Define *what* the system does (the specific behaviors, services, functions, and data transformations).
  - *Example*: A banking system must allow authenticated users to check account balances, transfer money between accounts, and download monthly statements in PDF format.
- **Non-Functional Properties (Quality of Service - QoS)**:
  - Define *how well* the system performs its functions (constraints, quality metrics, performance characteristics).
  - *Key Examples*:
    - **Performance / Timeliness**: Video conferencing audio must be delivered with end-to-end latency $< 150 \text{ ms}$ and jitter $< 30 \text{ ms}$.
    - **Reliability & Availability**: The online payment gateway must achieve $99.999\%$ uptime (no more than 5.26 minutes of downtime per year).
    - **Security**: All communication channels must enforce TLS 1.3 encryption and multi-factor authentication.
    - **Scalability**: The system must sustain 50,000 concurrent transactions per second during flash sales without response time exceeding 500 ms.

---

## 📝 TP02: Complete Solutions & Explanations

### Question 1: Give four types of hardware resource and five types of data or software resource that can usefully be shared. Give examples of their sharing in practice.
**Answer**:
- **Hardware Resources**:
  1. *Compute / Processing Power*: GPU compute clusters shared among machine learning researchers using Slurm or Kubernetes.
  2. *Storage Arrays*: Storage Area Networks (SAN) and Network Attached Storage (NAS) providing shared high-speed disk arrays to database nodes.
  3. *Printers / Peripherals*: Network office printers shared across an intranet using the Internet Printing Protocol (IPP) or CUPS.
  4. *Network Communication Links / Gateways*: Shared corporate fiber backbones and satellite uplinks shared among thousands of office workstations.
- **Data or Software Resources**:
  1. *Databases / Tables*: Centralized SQL databases (e.g., customer account records) shared among web apps, billing services, and mobile APIs.
  2. *Files & Documents*: Shared shared spreadsheets and source repositories (e.g., Git repos on GitHub, Google Docs).
  3. *Application Software / Web Services*: Mapping APIs (Google Maps API) embedded into thousands of third-party delivery applications.
  4. *Search Indexes*: Web search indexes (Google / Bing) shared and queried by billions of concurrent users.
  5. *Authentication Tokens / Directories*: Lightweight Directory Access Protocol (LDAP) or Active Directory services shared across all corporate enterprise software.

---

### Question 2: Clock synchronization on LAN without external time source vs. Internet scale.
**Answer**:
1. **Synchronizing Clocks on a LAN without External Reference**:
   - **Internal Synchronization Algorithm (e.g., Berkeley Algorithm)**:
     - A chosen coordinator node polls each node for its current clock time.
     - The coordinator calculates the round-trip transit time ($RTT$) for each node to estimate message delivery delay.
     - The coordinator computes the fault-tolerant average time of all nodes (discarding outliers).
     - Instead of jumping clocks backward (which breaks monotonic time and causes software errors), the coordinator instructs each node to adjust its local oscillator frequency (gradual slew rate adjustment).
2. **Factors Limiting LAN Accuracy**:
   - **Non-deterministic Network Delay**: Variable queuing delays in switches and network interface buffers.
   - **Operating System Latency**: Variable interrupt handling, context switching, and scheduling delays when recording timestamps in user space.
   - **Clock Drift**: Quartz crystal oscillator frequency drifts due to temperature fluctuations.
3. **Synchronizing Large Numbers of Computers across the Internet**:
   - Utilizes the **Network Time Protocol (NTP)** arranged in a hierarchical tree of servers called **Stratums**:
     - *Stratum 0*: Atomic clocks and GPS receivers.
     - *Stratum 1*: Primary servers directly attached to Stratum 0 devices.
     - *Stratum 2*: Secondary servers synchronizing with Stratum 1 servers and each other via peer links.
     - *Stratum 3+*: Enterprise and client workstations synchronizing with Stratum 2 servers.
   - NTP uses statistical filters to eliminate jitter, calculates offset $\theta$ and delay $\delta$:
     $$\theta = \frac{(T_2 - T_1) + (T_3 - T_4)}{2}, \quad \delta = (T_4 - T_1) - (T_3 - T_2)$$
4. **Accuracy Discussion**:
   - LAN synchronization achieves accuracy within **1 to 10 milliseconds**.
   - Internet NTP synchronization achieves accuracy within **10 to 50 milliseconds** due to asymmetric Internet routing paths and variable WAN congestion.

---

### Question 3: Wireless PDA at a restaurant — service discovery without typing names.
**Answer**:
1. **Discovery Mechanism**:
   - **Beacons / Service Discovery Protocols**:
     - The restaurant transmits periodic local wireless discovery beacons (e.g., mDNS/DNS-SD, Bluetooth Low Energy BLE beacons, Wi-Fi Aware / Wi-Fi NAN, or UPnP).
     - When the PDA enters the radio boundary, its background listener captures the beacon advertisement containing a Uniform Resource Identifier (URI) or local service descriptor.
     - The PDA issues a service query over the local link and dynamically receives the restaurant menu, specials, and digital ordering interface.
2. **Technical Challenges**:
   - **Security & Trust**: Verifying that the advertised service is authentic and not a rogue malicious node launching a phishing attack.
   - **Interference & Range Precision**: Limiting signals so adjacent shops do not advertise conflicting services to customers walking past.
   - **Privacy**: Preventing the restaurant from silently tracking the customer's permanent MAC address without consent.
   - **Heterogeneity & Interoperability**: Ensuring cross-platform service descriptors work seamlessly across different mobile OS platforms.
   - **Power Consumption**: Continuous background radio scanning rapidly depletes mobile device batteries.

---

### Question 4: Compare and contrast Cloud Computing with traditional Client-Server computing. What is novel?
**Answer**:
- **Comparison**:
  - *Commonality*: Both rely on clients requesting computational operations or data from remote servers over networks.
  - *Infrastructure*: Traditional client-server relies on static, dedicated physical servers sized for peak load. Cloud computing relies on dynamic pools of virtualized, multi-tenant resources.
  - *Ownership*: In client-server, the organization typically buys and maintains physical server racks. In cloud computing, third-party cloud providers (AWS, Azure, GCP) own and manage the physical hardware.
- **Novelty of Cloud Computing**:
  1. **Elasticity**: Resources can be dynamically scaled up or down instantaneously in response to real-time workload fluctuations.
  2. **Utility Billing Model (Pay-as-you-go)**: Computing power, bandwidth, and storage are metered and charged like electricity or water.
  3. **On-Demand Self-Service**: Developers can provision thousands of virtual servers via an API call in seconds without human administrator intervention.
  4. **Multi-Tenancy & Resource Pooling**: Virtualization isolates workloads, allowing unrelated organizations to securely share the same physical hardware.

---

### Question 5: World Wide Web resource sharing, client, server, and core technologies (HTML, URLs, HTTP).
**Answer**:
1. **Resource Sharing in the Web**:
   - Resources (HTML pages, media, APIs) are published on servers and retrieved by millions of browser clients globally using standard protocols.
2. **Pros & Cons of Core Technologies**:
   - **HTML**:
     - *Pros*: Human-readable, platform-independent, declarative, hypertext links enable effortless cross-referencing.
     - *Cons*: Originally designed for static document presentation; lacks strong typing, requires complex client scripting (JavaScript) to build rich interactive applications.
   - **URLs**:
     - *Pros*: Universal, human-readable naming scheme; standard syntax for locating resources anywhere on the Internet.
     - *Cons*: Fragile; if a file is moved or renamed, the link breaks ("404 Not Found" / link rot). They couple resource names to host domain names.
   - **HTTP**:
     - *Pros*: Simple, extensible text-based protocol; stateless (simplifies server recovery and horizontal scaling); built-in caching support.
     - *Cons*: Statelessness complicates multi-step workflows (requiring cookies/tokens); high protocol overhead for frequent small messages compared to binary protocols.
3. **Suitability for General Client-Server Computing**:
   - HTTP/REST has become the dominant foundation for general distributed computing (REST APIs, Microservices, Webhooks). However, for low-latency, high-frequency distributed communications (e.g., distributed locks, financial trading, real-time gaming), HTTP text overhead and connection latency make binary RPC frameworks (e.g., gRPC, CORBA, Thrift) far superior.

---

### Question 6: Openness vs. Heterogeneity in an Open Distributed System (BLOB Object).
**Answer**:
- **Heterogeneity** addresses the challenge of making components built on **different** underlying platforms (CPUs, OSs, network protocols, programming languages) communicate correctly.
  - *Requirement*: Common communication standards (e.g., TCP/IP) and serialization/marshaling rules (e.g., JSON, Protocol Buffers, CORBA CDR) to translate data types between platforms.
- **Openness** addresses the challenge of making the system **extensible** so that new resources and services (such as a Binary Large Object / BLOB service) can be added and accessed by arbitrary clients without modifying existing infrastructure.
  - *Requirement*: Published, standardized, and uniform interfaces (API definitions) that allow new software components to integrate seamlessly.
- **Distinction**:
  - A system can handle heterogeneity without being open (e.g., a proprietary protocol bridging Windows and Linux that is secret and closed to third-party developers).
  - Openness requires heterogeneity support as a foundation, but specifically focuses on open specifications, standardization, and backward-compatible evolution.

---

### Question 7: BLOB Object security: Public vs. Protected operations.
**Answer**:
1. **Problems in Ensuring Only Named Users Access Protected Operations**:
   - **Authentication**: Reliably verifying the claimed identity of the client (passwords, cryptographic signatures, public keys) across an insecure network.
   - **Authorization (Access Control)**: Maintaining an Access Control List (ACL) associating named identities with allowed operations on each BLOB.
   - **Message Integrity & Non-Repudiation**: Preventing attackers from altering the request in flight to change the target BLOB ID or operation parameters.
   - **Replay Attacks**: An eavesdropper intercepting a valid signed request could replay it to trigger repeated execution. Requires nonces or timestamps.
   - **Spoofing / Impersonation**: Ensuring an unauthorized process cannot forge IP packets or identity tokens.
2. **Further Problems if Protected Operations Return Sensitive Data**:
   - **Confidentiality (Eavesdropping)**: The returned BLOB data traveling across network routers can be sniffed by adversaries unless protected with strong encryption (e.g., AES with TLS).
   - **Covert Channels & Traffic Analysis**: Even if data is encrypted, the size of the response and timing of packets can leak information about the protected BLOB content.
   - **Client-Side Caching Leakage**: If the client OS caches the decrypted BLOB to a local temporary file or swap space, other unprivileged local users might read it.

---

### Question 8: Three software components that may fail during remote method invocation.
**Answer**:
1. **The Client Process**:
   - *Failure Example*: Client program encounters a segmentation fault or runs out of memory and crashes immediately after dispatching the request.
   - *Tolerance Mechanism*: The server should implement transactional timeouts; if the client never collects the result, the server garbage-collects temporary state without deadlock.
2. **The Communication Channel (Network)**:
   - *Failure Example*: Network router buffer overflow drops the request packet or reply packet, or a fiber link is severed.
   - *Tolerance Mechanism*: Transport-level retransmissions (TCP) or application-level timeouts and retries using **idempotent** operation semantics.
3. **The Server Process**:
   - *Failure Example*: Server crashes mid-execution (e.g., unhandled exception or power outage) while modifying database state.
   - *Tolerance Mechanism*: Write-Ahead Logging (WAL) and recovery algorithms so upon restart the server restores state; clients use timeouts and fail over to replicated backup servers.

---

### Question 9: Multi-server resource migration & multicast request evaluation.
**Answer**:
1. **Why Transfer Resources Between Servers?**:
   - **Load Balancing**: Offloading resources from an overloaded server to an underutilized server.
   - **Data Locality**: Moving resources closer to clients actively accessing them (reducing network latency).
   - **Hardware Maintenance**: Migrating resources off physical hardware scheduled for maintenance or decommissioning.
   - **Fault Isolation**: Evacuating resources when hardware health diagnostics indicate impending failure.
2. **Evaluating Client Multicast for Mobility Transparency**:
   - *Concept*: If a client multicasts every request to all servers, the server currently holding the resource will respond.
   - *Why this is UNSATISFACTORY*:
     - **Massive Bandwidth & Processing Waste**: Flooding all servers creates $O(N)$ unnecessary network traffic and CPU load processing dropped requests.
     - **Risk of Duplicate Execution**: If the operation is non-idempotent (e.g., deducting funds) and multiple servers mistakenly process it, severe data corruption occurs.
     - **Poor Scalability**: Multicast does not scale efficiently across wide-area networks (WAN).
   - *Correct Solution*: Use a **Location/Directory Service** (e.g., DNS, ZooKeeper, Consul) or a **Reverse Proxy** that tracks current locations and forwards requests directly.

---

### Question 10: URLs denoting Web Resources.
**Answer**:
- **Acronym**: **URL** stands for **Uniform Resource Locator**.
- **Three Different Sorts of Web Resources Named by URLs**:
  1. *Static Documents*: `https://www.example.org/docs/manual.pdf` (a static file stored on a server's disk).
  2. *Dynamic Application Endpoints / REST Resources*: `https://api.example.org/v1/users/42` (a programmatic endpoint executing database queries).
  3. *Continuous Multimedia Streams*: `rtsp://media.example.org/live/stream.m3u8` (a live audio/video stream).

---

### Question 11: HTTP URL breakdown and Location Transparency analysis.
**Answer**:
1. **Example HTTP URL**:
   ```
   http://www.techno.edu.kh:8080/academic/courses/ds.html?semester=1#syllabus
   ```
2. **Main Components and Delimiters**:
   - `http`: **Scheme/Protocol** — delimited by `://`.
   - `www.techno.edu.kh`: **Host (Domain Name)** — delimited by `:` (or `/` if port omitted).
   - `8080`: **Port Number** (optional) — delimited by preceding `:` and succeeding `/`.
   - `/academic/courses/ds.html`: **Path** — hierarchical path delimited by forward slashes `/`.
   - `semester=1`: **Query String** (optional) — preceded by `?`.
   - `syllabus`: **Fragment Identifier** (optional) — preceded by `#`.
3. **To What Extent is an HTTP URL Location-Transparent?**:
   - **Partial / Logical Location Transparency**:
     - The URL hides the physical geographical location of the server, the physical hardware rack, and the internal IP address (resolved via DNS).
   - **Limitations (Why it lacks complete location transparency)**:
     - The URL explicitly hardcodes the server's **host domain name** (`www.techno.edu.kh`). If the organization migrates the resource to a different domain (e.g., `www.itc.edu.kh`), all existing URLs break unless HTTP 301 redirects are configured.
     - The path often mirrors the server's internal filesystem or routing hierarchy. A pure location-independent name (such as a URN) does not encode the host address.

---

## 📊 Exam Review Summary & Key Formulas

$$\text{Transmission Time} = \text{Latency} + \frac{\text{Message Size (bytes)}}{\text{Data Transfer Rate (bytes/sec)}}$$

* **NTP Offset Equation**:
  $$\theta = \frac{(T_2 - T_1) + (T_3 - T_4)}{2}$$
* **NTP Round-Trip Delay Equation**:
  $$\delta = (T_4 - T_1) - (T_3 - T_2)$$
* **Key Transparencies Rule of Thumb**:
  - *Access*: "How I call it" (same API)
  - *Location*: "Where it lives" (no IP/path needed)
  - *Replication*: "How many copies exist" (looks like one)
  - *Failure*: "If it crashes, I still get my answer"
