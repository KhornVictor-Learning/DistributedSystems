# Chapter 2: Study Guide & Lab Solutions

> **Course**: Distributed Systems (I4-GIC-A / I4-GIC-S1) — ITC (Techno)  
> **Topic**: System Models  
> **Textbook**: Coulouris et al., *Distributed Systems: Concepts and Design (5th Ed.)*, Chapter 2  
> **Included Labs**: Full solutions for **TP03** & **TP04**

---

## 📑 Table of Contents
1. [Core Architectural & Fundamental Theory](#1-core-architectural--fundamental-theory)
   - [Centralized vs. Decentralized vs. Distributed](#centralized-vs-decentralized-vs-distributed)
   - [Space and Time Decoupling](#space-and-time-decoupling)
   - [Synchronous vs. Asynchronous Models](#synchronous-vs-asynchronous-models)
   - [Thin-Client Computing & Remote Desktop Architectures](#thin-client-computing--remote-desktop-architectures)
2. [TP03: Complete Solutions & Explanations](#tp03-complete-solutions--explanations)
3. [TP04: Complete Solutions & Explanations](#tp04-complete-solutions--explanations)
4. [Exam Review Summary](#exam-review-summary)

---

## 1. Core Architectural & Fundamental Theory

### Centralized vs. Decentralized vs. Distributed

```
   (A) Centralized             (B) Decentralized              (C) Distributed
         Node                         Node                          Node
          |                          /    \                        /  |  \
     [ Central ]               [ Hub 1 ]  [ Hub 2 ]             Node--Node--Node
       /  |  \                   /   \      /   \                \  |  /
     N1   N2  N3               N1    N2   N3    N4                  Node
```

| Dimension | Centralized System | Decentralized System | Distributed System |
| :--- | :--- | :--- | :--- |
| **Control** | Single central authority coordinates all actions. | Divided among several regional or functional authorities. | No single authority; coordination is achieved by consensus among peers. |
| **Single Point of Failure (SPOF)** | Yes: If the central server crashes, the entire system stops. | Partially: Failure of a local hub affects only its branch. | No: High fault tolerance; individual nodes can crash without stopping the system. |
| **Scalability** | Severely limited by central server hardware capacity. | Moderate: Sclaed by provisioning additional regional hubs. | Highly scalable: Components can be added incrementally. |
| **Example** | Mainframe database, early banking systems. | DNS hierarchy, cellular base stations. | BitTorrent, Ethereum blockchain, Apache Cassandra. |

---

### Space and Time Decoupling

Indirect communication paradigms provide decoupling along two primary dimensions:
- **Space Uncoupling (Referential Decoupling)**:
  - Senders do not know, specify, or depend on the identities or network addresses of receivers.
  - Senders simply direct messages to an intermediary topic, channel, or queue.
- **Time Uncoupling (Temporal Decoupling)**:
  - The sender and receiver do not need to exist or be connected at the same time.
  - The sender can publish a message and terminate; the receiver can come online hours later and retrieve the message.

```
+-----------+        Space & Time Decoupled         +-----------+
| Producer  |  --->  [ Persistent Intermediary ]  --->  | Consumer  |
| (Sender)  |        (Kafka / RabbitMQ / Queue)     | (Receiver)|
+-----------+                                       +-----------+
```

---

### Synchronous vs. Asynchronous Models

| Property | Synchronous Distributed System | Asynchronous Distributed System |
| :--- | :--- | :--- |
| **Process Execution Time** | Has known lower and upper bounds. | No bounds on step execution speeds. |
| **Message Transmission Delay** | Guaranteed bounded latency ($D$). | Unbounded delay (can take arbitrary time). |
| **Local Clock Drift Rate** | Bounded drift rate ($\rho$) from real time. | Drift rate is unbounded or arbitrary. |
| **Crash Detection** | Feasible via timeouts ($T_{\text{timeout}} > 2D + \text{process bound}$). | Impossible to distinguish between a dead node and a slow network. |
| **Real-World Reality** | Aircraft fly-by-wire, automotive CAN bus. | The public Internet, Web applications. |

---

### Thin-Client Computing & Remote Desktop Architectures

A **thin client** is a lightweight software client that handles only user input (keyboard, mouse) and displays visual graphics, while all application logic, computational processing, and file storage execute on a remote server.
- **Protocols**:
  - **RFB (Remote Framebuffer)**: Used by VNC. Transmits pixel rectangles and framebuffer updates directly. Simple but bandwidth-heavy.
  - **RDP (Remote Desktop Protocol)**: Used by Microsoft. Transmits high-level drawing primitives and UI commands, significantly reducing bandwidth.
- **Main Challenge**: Highly sensitive to network latency and packet loss. Graphical video playback or 3D gaming suffers from noticeable input lag.

---

## 📝 TP03: Complete Solutions & Explanations

### Question 1: Explain the concept of remote desktop, and give examples of TeamViewer, VNC, or MS Remote Desktop.
**Answer**:
- **Concept**:
  Remote Desktop is a thin-client distributed architecture allowing a local client device to remotely control and interact with a desktop environment running on another computer over a network. The client captures user input events (keystrokes, mouse clicks) and transmits them to the remote server, while the remote server renders the OS interface and streams visual screen updates back to the client.
- **Examples**:
  1. **Virtual Network Computing (VNC)**:
     - Operates using the open **RFB (Remote Framebuffer)** protocol at the pixel level.
     - Platform-independent (Linux, macOS, Windows).
     - Because it transmits updated pixel rectangles, it is resilient to OS differences but requires substantial network bandwidth for complex graphical changes.
  2. **Microsoft Remote Desktop (RDP)**:
     - Proprietary protocol deeply integrated into the Windows OS graphics subsystem (GDI / DirectX).
     - Transmits high-level graphical commands and drawing primitives (e.g., "draw a button here", "render this font glyph") rather than raw pixels, achieving high responsiveness even over lower-bandwidth links.
  3. **TeamViewer**:
     - Modern commercial remote access platform featuring **automatic NAT and firewall traversal**.
     - Instead of requiring users to configure port forwarding, both the client and remote host establish outbound connections to a centralized TeamViewer mediation/relay server, enabling effortless cross-network access.

---

### Question 2: Explain semaphores in OS and DB.
**Answer**:
- **In Operating Systems**:
  - A semaphore is a synchronization variable invented by Edsger Dijkstra to manage concurrent access to shared resources by multiple processes or threads.
  - It maintains an integer counter initialized to the number of available resource units and supports two atomic operations:
    1. `wait()` (or `P()`): Decrements the counter. If the counter is $\le 0$, the calling thread blocks until another thread signals.
    2. `signal()` (or `V()`): Increments the counter and wakes up one waiting thread.
  - *Binary Semaphore (Mutex)*: Initialized to 1, guaranteeing mutual exclusion in critical sections.
  - *Counting Semaphore*: Initialized to $N > 1$, managing a pool of identical resources (e.g., database connection pool).
- **In Databases**:
  - In Database Management Systems (DBMS), synchronization manages concurrent transactions to maintain **ACID properties** (Atomicity, Consistency, Isolation, Durability) and prevent anomalies (dirty reads, lost updates).
  - Databases implement locks and latches:
    - *Latches (OS-level semaphores)*: Protect internal DBMS memory structures (e.g., buffer pool hash tables) for microsecond durations.
    - *Logical Database Locks*: Protect tables, rows, or indexes for the duration of a transaction using **Two-Phase Locking (2PL)**:
      - **Shared Locks (S-locks)**: Allow multiple concurrent readers.
      - **Exclusive Locks (X-locks)**: Grant exclusive write access, blocking all other readers and writers.

---

### Question 3: Explain the emergence of Cloud Computing.
**Answer**:
- **Definition & Emergence**:
  Cloud computing emerged from the convergence of server virtualization (hypervisors), high-speed fiber-optic broadband, and massive economy of scale in automated hyperscale data centers. It transforms computing infrastructure into a publicly accessible **utility** (comparable to electricity or water).
- **Service Models**:
  1. **IaaS (Infrastructure as a Service)**: Raw compute VMs, virtual networking, and block storage (e.g., AWS EC2, Google Compute Engine).
  2. **PaaS (Platform as a Service)**: Managed execution environments allowing developers to deploy code without managing OS or runtime patching (e.g., Heroku, AWS Elastic Beanstalk).
  3. **SaaS (Software as a Service)**: Complete end-user applications delivered via web browsers (e.g., Google Workspace, Microsoft 365).
- **Key Advantages**:
  - *Rapid Elasticity*: Instantaneous auto-scaling in response to load spikes.
  - *Cost Efficiency*: Eliminates upfront capital expenditure (CapEx) for physical server racks; converts costs to operational expenditure (OpEx).
  - *High Availability*: Automated multi-region replication and fault failover.

---

### Question 4: Explain the process of how torrent (BitTorrent) works.
**Answer**:
BitTorrent is a decentralized peer-to-peer file-sharing protocol that distributes large files without overloading a single central server:
1. **File Partitioning & Metadata**:
   - The original file is divided into small, uniform pieces (typically 256 KB to 2 MB), and a SHA-1/SHA-256 cryptographic hash is calculated for each piece to guarantee integrity.
   - A `.torrent` metadata file (or Magnet URI) is created containing these hashes and the address of a **Tracker** (or DHT info).
2. **Joining the Swarm**:
   - A client opening the torrent contacts the tracker (or queries the Distributed Hash Table / DHT) to obtain a list of active participating peers sharing that file (the **Swarm**).
3. **Downloading and Uploading**:
   - **Rarest-First Strategy**: The client queries connected peers to determine which pieces are least common across the swarm and downloads those first. This prevents rare pieces from disappearing if seeds disconnect.
   - **Verification**: As each piece arrives, the client verifies its hash against the metadata file. Corrupted pieces are discarded and re-downloaded.
   - **Choking & Tit-for-Tat Algorithm**: Peers upload data to peers that reciprocate by uploading data back to them at high rates ("Tit-for-Tat"), preventing free-riding.
4. **Seeders vs. Leechers**:
   - A **Leecher** is a peer downloading missing pieces while uploading acquired ones.
   - Once a peer possesses 100% of the file, it becomes a **Seeder**, serving pieces to remaining leechers.

---

### Question 5: Find the advantages of peer-to-peer over client-server architecture.
**Answer**:
1. **Organic Scalability**: In client-server, each new client consumes server bandwidth, causing performance degradation unless servers are upgraded. In P2P, every new peer contributes upload bandwidth and storage capacity to the swarm.
2. **Absence of a Single Point of Failure (High Fault Tolerance)**: The failure or departure of individual peer nodes does not disrupt the network; remaining peers continue serving data.
3. **Massive Cost Reduction**: The service creator does not need to purchase, power, or maintain expensive server infrastructure and WAN bandwidth links.
4. **Load Distribution**: Traffic is dispersed across thousands of ISP networks rather than bottlenecked at a single data center uplink.

---

### Question 6: Between client-server and peer-to-peer, which one is more complex? Explain.
**Answer**:
**Peer-to-Peer is drastically more complex** to design, secure, and operate:
1. **Dynamic Churn**: Peer nodes join, disconnect, and crash unpredictably without notice. The protocol must constantly maintain routing tables and replicate data to prevent permanent loss.
2. **Decentralized Resource Discovery**: In client-server, locating data is trivial (query the known server). In P2P, locating which node currently holds a specific data item requires complex distributed routing algorithms (e.g., Distributed Hash Tables like Kademlia/Chord).
3. **Trust and Security Vulnerabilities**: Peer nodes are untrusted third-party consumer machines. P2P systems must protect against malicious peers poisoning files, injecting bogus routing entries (Sybil attacks, Eclipse attacks), and free-riding.
4. **Inconsistent Data State**: Maintaining consistent state and consensus across millions of autonomous, asynchronous peers without a central coordinator requires sophisticated protocols (e.g., Paxos, Raft, or Byzantine Fault Tolerance).

---

### Question 7: Are multiple servers and a cluster the same? Explain.
**Answer**:
**No, they are fundamentally different concepts**:
- **Multiple Servers**:
  - A general, broad term describing any system that deploys more than one server machine.
  - The servers may be geographically separated across different continents, run completely different operating systems, perform unrelated tasks, or connect over the public Internet (e.g., an email server in London and a database server in Tokyo).
- **A Cluster (Computer Cluster)**:
  - A specialized, tightly coupled subset of multiple servers.
  - Characteristics of a cluster:
    1. Servers (nodes) are situated in the **same physical location** (e.g., same rack or data center).
    2. Connected via a **dedicated high-speed, low-latency private interconnect** (e.g., 100 Gbps InfiniBand or dedicated fiber).
    3. Run identical or coordinated software and share storage arrays.
    4. Present the illusion of a **single unified virtual computer** to clients (single-system image) for high-performance computing (HPC) or transparent high-availability failover.

---

### Question 8: Explain proxy server.
**Answer**:
- **Definition**:
  A proxy server is an intermediary server application that acts as an agent or gateway between client programs and destination target servers.
- **Key Functions & Types**:
  1. **Forward Proxy (Client-side)**:
     - Sits in front of client devices within an enterprise or private network.
     - Intercepts outbound client requests to enforce web filtering, security policies, and user anonymity.
  2. **Caching**:
     - Retains copies of frequently requested web resources locally. When subsequent clients request the same URL, the proxy responds immediately from cache, reducing WAN traffic.
  3. **Firewall / NAT Traversal**:
     - Enables internal intranet computers lacking public IP addresses to securely access the external Internet.
  4. **Reverse Proxy (Server-side)**:
     - Sits in front of backend servers (e.g., Nginx, HAProxy, Cloudflare).
     - Terminates TLS/SSL connections, distributes incoming traffic across backend server pools (load balancing), and shields backend servers from direct public attacks (DDoS mitigation).

---

## 📝 TP04: Complete Solutions & Explanations

### Question 1: Comparison – Centralized, Decentralized, and Distributed systems.
**Answer**:
1. **Centralized System**:
   - *Definition*: A single central node manages all data, computing, and administrative decisions.
   - *Architecture*: Star topology; all client nodes connect directly to the central hub.
   - *Pros*: Simple data consistency, straightforward security enforcement, easy administration.
   - *Cons & Limitations*: Single point of failure; hard hardware scalability bottleneck; vulnerable to network link failures to the hub.
   - *Example*: Legacy IBM Mainframe system.
2. **Decentralized System**:
   - *Definition*: Control is partitioned among several central hubs/coordinators, each managing its local cluster of nodes.
   - *Architecture*: Hierarchical or federated tree topology.
   - *Pros*: Better local fault tolerance; reduced bottleneck at top level; handles regional traffic efficiently.
   - *Cons & Limitations*: Coordination between regional hubs is complex; failure of a hub isolates its entire sub-network.
   - *Example*: The Domain Name System (DNS) or cellular telecom infrastructure.
3. **Distributed System**:
   - *Definition*: Autonomous nodes communicate solely by message passing without any centralized coordinator.
   - *Architecture*: Mesh network where any node can communicate with any reachable peer.
   - *Pros*: Extreme fault tolerance, limitless horizontal scalability, no single point of failure.
   - *Cons & Limitations*: Extreme algorithmic complexity (consensus, clock synchronization, concurrency control); high network communication overhead.
   - *Example*: Apache Cassandra distributed database, BitTorrent network.

---

### Question 2: Problems with direct coupling in remote invocation & advantages of space/time uncoupling.
**Answer**:
1. **Problems in Direct Coupling (Remote Invocation / RPC / RMI)**:
   - **Space Coupling**: The client must explicitly know the network IP address, port number, and interface signature of the target server. If the server migrates or changes port, clients break unless reconfiguration occurs.
   - **Time Coupling**: Both sender and receiver must be active, running, and reachable at the exact moment of communication. If the server is offline or experiencing network downtime, the client's invocation fails or blocks indefinitely.
   - **Rigid Synchronization**: Direct RPCs typically block the client thread waiting for a response, wasting compute resources and cascading delays across microservices.
2. **Advantages of Space and Time Uncoupling (Indirect Communication)**:
   - **Dynamic Scalability**: Senders publish messages without caring how many receivers exist (1, 100, or 0). Receivers can be dynamically added or removed without reconfiguring senders.
   - **Tolerance of Intermittent Connectivity**: Messages persist in intermediate queues or topic logs (e.g., Kafka). Nodes can go offline, reboot, or undergo maintenance without losing messages.
   - **Asynchronous Execution**: Senders proceed immediately after handing messages to local buffers, eliminating synchronous latency bottlenecks.

---

### Question 3: Client-server architecture of major Internet applications (Web, Email, Netnews).
**Answer**:
1. **The World Wide Web (HTTP)**:
   - *Client*: Web browsers (Chrome, Firefox) parsing HTML/CSS/JS and issuing HTTP/HTTPS requests.
   - *Server*: Web servers (Nginx, Apache) listening on port 80/443, retrieving files or routing requests to application logic.
   - *Protocol*: Stateless request-reply over TCP.
2. **Electronic Mail (SMTP, IMAP, POP3)**:
   - *Client*: Mail User Agent (MUA, e.g., Thunderbird, Outlook, Apple Mail).
   - *Server*: Mail Transfer Agent (MTA, e.g., Postfix) and Mail Delivery Agent (MDA, e.g., Dovecot).
   - *Architecture*: A client uses **SMTP** (port 25/587) to send mail to its outgoing server. The outgoing server acts as a client to forward mail via SMTP to the recipient's domain MTA. The recipient uses **IMAP** (port 993) or **POP3** (port 995) to retrieve stored messages from their mailbox server.
3. **Netnews (Usenet / NNTP)**:
   - *Client*: Newsreaders connecting via Network News Transfer Protocol (**NNTP**, port 119/563).
   - *Architecture*: Federated client-server with peer-to-peer flooding. Users post articles to their local news server. News servers flood newly received articles to neighboring news servers using a flood-fill algorithm, ensuring all servers synchronize newsgroup feeds.

---

### Question 4: Concurrency in Search Engines: Index Queries vs. Web Crawlers.
**Answer**:
- **Architecture**:
  A search engine concurrently executes two major tasks:
  1. *User Query Serving*: Millions of user queries reading search indexes to retrieve matching web results (read-heavy, low-latency requirement $< 100 \text{ ms}$).
  2. *Web Crawling & Index Building*: Background crawler spiders downloading web pages and updating inverted indexes (write-heavy, batch processing).
- **Synchronization Requirements**:
  1. **Readers-Writers Concurrency Control**:
     - Direct in-place modification of active index structures will corrupt search results for concurrent user queries.
     - Standard locks (Mutex) are unusable because blocking user queries during massive multi-gigabyte index updates creates unacceptable latency spikes.
  2. **Double Buffering / Shadow Indexing**:
     - Crawlers build an entirely separate **new version** of the index (shadow index) on separate storage without touching the active index.
     - Once index generation finishes, an atomic pointer swap instantaneously switches incoming user queries to the new index.
  3. **Eventual Consistency & Partitioning**:
     - Indexes are partitioned (sharded) across thousands of server nodes by document ID or term. Updates are applied incrementally partition by partition.

---

### Question 5: P2P Desktop Computers: Availability, Security, and Replication.
**Answer**:
1. **Implications for Availability**:
   - Consumer desktop computers in homes and offices suffer from high churn: users shut them down at night, reboot for OS updates, or lose Internet connectivity.
   - As a result, the availability of any single node is low (often $< 50\%$).
2. **Implications for Security**:
   - Host computers lack enterprise physical and network security. They may be infected with malware, controlled by adversaries, or packet-sniffed.
   - Data stored in plaintext on peer hard drives is vulnerable to unauthorized inspection and tampering.
3. **Overcoming Weaknesses via Replication**:
   - **Availability Solution**: Store each data object across $k$ independent peers (e.g., $k=8$). Even if individual node availability is $p = 0.5$, the probability that all $k$ replicas are simultaneously offline is $(1 - p)^k = 0.5^8 \approx 0.0039$ ($99.6\%$ availability).
   - **Security Solution**:
     - *Confidentiality*: Encrypt data with strong client-side encryption (AES-256) before uploading; peers store only ciphertext.
     - *Integrity*: Store cryptographic hash digests (SHA-256) in a signed root manifest; when retrieving chunks, peers verify hashes to reject tampered data.

---

### Question 6: Local resources vulnerable to untrusted mobile code.
**Answer**:
When an untrusted program (e.g., JavaScript, Java Applet, or downloaded executable) runs on a local machine, the following resources are vulnerable:
1. **Local Filesystem & Storage**: Sensitive private files (passwords, SSH private keys, personal photos) can be read, exfiltrated, or encrypted by ransomware.
2. **Network Interfaces & Sockets**: The program can open arbitrary sockets to participate in DDoS botnets, scan internal private intranet hosts, or transmit stolen credentials to command-and-control servers.
3. **System Memory & Process Memory Space**: Attempting buffer overflows to execute arbitrary shellcode or reading sensitive tokens from other processes.
4. **Physical Peripherals**: Accessing webcams, microphones, or keyloggers to record user activity without authorization.
5. **CPU & GPU Compute**: Hijacking hardware for cryptocurrency mining or exhaustive cryptographic cracking, causing thermal damage and denial of service.

---

### Question 7: Why is setting a guaranteed response time bound generally impossible for a simple server?
**Answer**:
1. **Why Impossible on General-Purpose Systems**:
   - **OS Scheduling Jitter**: General-purpose operating systems (Linux, Windows) use preemptive time-sharing schedulers. The server process can be preempted by interrupts, background daemons, or page faults at any unpredictable moment.
   - **Non-Deterministic Network Delays**: Packets travel across switches and routers subject to queuing delays, packet drops, retransmissions, and bufferbloat.
   - **Hardware Unpredictability**: CPU cache misses, garbage collection pauses, and virtual memory paging introduce arbitrary latency variations.
2. **What Would Be Required for Bounded Execution?**:
   - Real-Time Operating System (**RTOS**) with strictly prioritized, non-preemptible scheduling.
   - Dedicated physical network links with resource reservation protocols (e.g., **RSVP**, ATM, Time-Sensitive Networking TSN) guaranteeing zero packet collision and fixed transmission times.
   - Hard memory pinning to prevent paging.
3. **Is This Practical?**:
   - **Not practical for general Internet applications**: The open Internet is an uncoordinated, best-effort packet-switched network with no end-to-end bandwidth or delay reservations.
   - **Practical only in specialized niches**: Avionics, automotive brake-by-wire, industrial robotic control systems.

---

### Question 8: Why NTP gives no guaranteed bound for clock differences.
**Answer**:
Even though the Network Time Protocol (NTP) provides statistical synchronization, it cannot guarantee a strict theoretical bound on clock skew because:
1. **Asymmetric Network Transmission Delays**:
   - NTP assumes the outbound transit time ($T_{\text{client} \to \text{server}}$) equals inbound transit time ($T_{\text{server} \to \text{client}}$).
   - On the Internet, asymmetric routing paths are common: packets from A to B travel via a completely different path than packets from B to A.
   - If transit delays are asymmetric, the calculated clock offset $\theta$ has an inherent error that cannot be eliminated without knowing exact one-way delay.
2. **Unpredictable Packet Jitter**:
   - Intermediate Internet router queue lengths change millisecond-by-millisecond depending on unrelated cross-traffic, making message delay variable and unbounded.

---

### Question 9: Security threats on the Internet (Processes, Channels, DoS).
**Answer**:
1. **Threats to Processes**:
   - *Masquerading / Impersonation*: An attacker pretends to be a legitimate server or client (e.g., spoofing a banking website or forging authorization headers).
   - *Trojan Horses / Malicious Code Injection*: Tricking a server process into executing malicious payloads (e.g., SQL Injection, Remote Code Execution).
2. **Threats to Communication Channels**:
   - *Eavesdropping (Sniffing)*: Intercepting plaintext packets traversing public routers to steal confidential passwords, cookies, or credit card numbers.
   - *Message Tampering*: Modifying packet contents in flight (e.g., altering a bank wire transfer destination account number).
   - *Replay Attacks*: Recording valid encrypted request packets and replaying them across the channel to duplicate an action.
3. **Denial of Service (DoS / DDoS)**:
   - Flooding a server's network bandwidth or CPU with overwhelming bogus traffic (e.g., TCP SYN floods, DNS amplification attacks), exhausting server buffers and preventing legitimate users from accessing services.

---

## 📊 Exam Review Summary

* **Coupling Rules**:
  - *RPC/RMI*: Space coupled & Time coupled.
  - *Message Queues & Pub/Sub*: Space uncoupled & Time uncoupled.
* **Failure Model Hierarchy**:
  $$\text{Fail-Stop} \subset \text{Crash-Recovery} \subset \text{Omission} \subset \text{Byzantine (Arbitrary)}$$
* **Cluster vs. Multiple Servers**:
  - *Cluster*: Co-located, high-speed interconnect, single virtual image.
  - *Multiple Servers*: Any set of independent servers, possibly distant.
