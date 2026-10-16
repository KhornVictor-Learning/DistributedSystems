# Chapter 1: Characterization of Distributed Systems

> **Course Reference**: Distributed Systems (I4-GIC-A / I4-GIC-S1)  
> **Institution**: Institute of Technology of Cambodia (ITC / Techno)  
> **Lecturer**: Khem Thay  
> **Textbook Reference**: *Distributed Systems: Concepts and Design (5th Edition)*, Chapter 1

---

## 📌 1. Introduction & Formal Definition

A **Distributed System** is defined as one in which hardware or software components located at networked computers communicate and coordinate their actions only by **passing messages**.

```
+------------------+          Network           +------------------+
|   Computer A     | <------------------------> |    Computer B    |
| [Process 1 (P1)] |      Message Passing       | [Process 2 (P2)] |
+------------------+      (No Shared RAM)       +------------------+
```

### Key Differences from Centralized Systems
* In a centralized system, processes communicate through shared physical memory and coordinate using hardware interrupts or kernel primitives with a single system clock.
* In a distributed system, components run on autonomous physical machines with distinct physical memory spaces, communicate exclusively via network packets, and cannot rely on shared memory or a single physical clock.

---

## ⚙️ 2. Core Characteristics of Distributed Systems

All distributed systems exhibit three intrinsic characteristics:

| Characteristic | Description | Architectural Implication |
| :--- | :--- | :--- |
| **Concurrency** | Multiple programs/threads execute simultaneously on different computers across the network. | Shared resources must be protected against race conditions using concurrency control (locks, semaphores, transactions). |
| **Lack of a Global Clock** | Each computer has its own local quartz oscillator clock. Clocks drift relative to real time and to each other. | There is no single authoritative global timestamp. Coordination must rely on message exchange and logical clocks (e.g., Lamport timestamps). |
| **Independent Failures** | Individual components (nodes, switches, links) can fail unexpectedly without causing other parts of the system to stop running. | Components must handle partial failures gracefully. A node cannot distinguish whether a remote peer has crashed, is slow, or is partitioned by a network outage. |

---

## 📈 3. Trends in Distributed Systems

Modern distributed systems have evolved far beyond early static local-area networks. Key contemporary trends include:

1. **Pervasive Networking and the Modern Internet**:
   - The Internet is a vast interconnected collection of heterogeneous networks (intranets, ISPs, backbones, satellite links, cellular networks).
   - Devices ranging from smartphones to high-performance server clusters connect continuously.
2. **Mobile and Ubiquitous Computing**:
   - Small, portable, and embedded computing devices (smartphones, wearables, smart home appliances, IoT sensors) operate seamlessly across varying locations.
   - Requires dynamic service discovery, handling intermittent network disconnections, and adapting to low-power constraints.
3. **Distributed Multimedia Systems**:
   - Real-time streaming of high-definition video and audio (e.g., YouTube, Netflix, Zoom).
   - Demands guaranteed Quality of Service (QoS), bounded packet delays, jitter management, and high throughput.
4. **Distributed Systems as a Utility (Cloud Computing)**:
   - Computing resources (processing, storage, infrastructure, software) are hosted on massive remote data centers and provisioned dynamically on-demand over the Internet.
   - Resource elasticity allows clients to pay only for the compute and storage they consume (IaaS, PaaS, SaaS).

---

## 🔄 4. Resource Sharing & The Client-Server Model

Resource sharing is the primary driving motivation behind distributed systems.

### What Resources are Shared?
* **Hardware Resources**: High-throughput compute clusters, network storage arrays (SAN/NAS), high-end printers.
* **Data Resources**: Files, distributed database records, web documents, application states.
* **Specific Functionality / Services**: Search engines, authentication providers (OAuth/SAML), payment processing gateways.

### Services, Clients, and Servers
* **Service**: An autonomous software entity that manages a collection of related resources and presents their functionality to applications via a well-defined communication interface.
* **Server**: A running program (process) on a networked computer that accepts incoming requests from other computers, performs the requested operation on the resources it manages, and replies with results.
* **Client**: A requesting process running on another computer that issues requests to servers.
* **Remote Invocation**: The complete end-to-end communication sequence where a client sends a request message, the server executes the required operation, and the server returns a reply message.

```
+------------+       Request Message (Remote Invocation)       +------------+
|   Client   | ----------------------------------------------> |   Server   |
| (invoker)  | <---------------------------------------------- | (manager)  |
+------------+                   Reply Message                 +------------+
```

---

## ⚡ 5. The Eight Fundamental Challenges

Building robust distributed systems requires addressing eight primary engineering challenges:

### 1. Heterogeneity
Distributed systems operate across heterogeneous environments:
- **Networks**: Varying protocols (Ethernet, Wi-Fi, 4G/5G, fiber) unified by the Internet Protocol (IP).
- **Computer Hardware**: Differing CPU architectures and data representations (e.g., Big-Endian vs. Little-Endian integers, 32-bit vs. 64-bit word sizes).
- **Operating Systems**: Diverse kernel APIs (POSIX Linux, Windows, macOS).
- **Programming Languages**: Differing internal structures, byte layouts, and object representations.
- **Implementations**: Independent codebases adhering to common standard protocols.
* **Solutions**:
  - **Middleware**: A software layer (e.g., CORBA, Java RMI, gRPC, REST/JSON) providing uniform programming abstractions that conceal underlying hardware, OS, and network differences.
  - **Mobile Code & Virtual Machines**: Compiling to portable bytecode (e.g., Java Virtual Machine, WebAssembly) that can run on any host machine.

### 2. Openness
The degree to which new resource-sharing services can be added and made accessible to heterogeneous client programs without disrupting existing services.
- **Hardware Extensibility**: Adding more compute nodes, storage drives, or network interfaces.
- **Software Extensibility**: Publishing standard interfaces (e.g., RFC specifications, OpenAPI specs) so any developer can implement compatible clients or services.

### 3. Security
Protecting resources transferred across insecure open networks against threats. Security rests on three core pillars:
	- **Confidentiality**: Ensuring data is protected against disclosure to unauthorized parties (via encryption).
- **Integrity**: Ensuring data is protected against unauthorized alteration or corruption (via cryptographic hashes and digital signatures).
- **Availability**: Ensuring authorized users can access resources without denial-of-service interference.

### 4. Scalability
A system is scalable if it remains effective and performant when the number of users, requests, and physical resources increases significantly.
- **Key Scalability Challenges**:
  - Controlling the cost of physical resources (costs must scale linearly, $O(n)$, not exponentially).
  - Controlling performance loss (avoiding severe latency increases under high traffic).
  - Preventing software resource exhaustion (e.g., running out of 32-bit IPv4 addresses or 16-bit port numbers).
  - Avoiding performance bottlenecks (e.g., a single centralized DNS server or database).
- **Standard Solutions**:
  - **Caching**: Storing frequently accessed data closer to clients.
  - **Replication**: Running duplicate instances of services or databases to balance traffic.
  - **Data Partitioning (Sharding)**: Splitting datasets across multiple servers.
  - **Concurrent execution**: Utilizing multi-threading and asynchronous I/O.

### 5. Failure Handling
In distributed systems, failures are **partial**; some components fail while others continue operating.
- **Detecting Failures**: Using checksums for data corruption; using heartbeats and timeouts to detect unreachable nodes.
- **Masking Failures**: Concealing faults or mitigating their effects (e.g., TCP retransmitting dropped packets, RAID mirroring disk drives).
- **Tolerating Failures**: Designing clients to handle server timeouts gracefully and retry operations or notify users.
- **Recovery from Failures**: Writing persistent state logs and transaction journals so servers can roll back or replay operations after a crash.
- **Redundancy**: Deploying standby servers and redundant network links to eliminate single points of failure.

### 6. Concurrency
Multiple clients frequently execute requests on shared resources simultaneously.
- If multiple threads or processes modify shared data without coordination, race conditions occur, leading to corrupted state.
- **Solution**: Shared objects and servers must implement synchronization mechanisms (mutual exclusion locks, semaphores, atomic transactions).

### 7. Transparency
The concealment of the separation of components from users and application programmers so that the system is perceived as a single integrated whole. The ISO Open Distributed Processing (ODP) reference model defines 8 forms:

1. **Access Transparency**: Enables local and remote resources to be accessed using identical operations/APIs (e.g., POSIX `read()` on local disk vs. NFS remote file).
2. **Location Transparency**: Enables resources to be accessed without knowledge of their physical or network location (e.g., URLs such as `http://example.com/data` do not expose the physical IP or data center).
3. **Concurrency Transparency**: Enables several processes to operate concurrently using shared resources without mutual interference.
4. **Replication Transparency**: Enables multiple copies of resources to be used to increase reliability and performance without clients knowing replicas exist.
5. **Failure Transparency**: Enables the concealment of faults, allowing users and programs to complete tasks despite hardware or software crashes (e.g., automatic failover).
6. **Mobility (Migration) Transparency**: Allows resources and clients to move to different nodes without interrupting active operations.
7. **Performance Transparency**: Allows the system to be reconfigured (e.g., adding nodes or scaling up) to maintain or improve performance as workloads vary.
8. **Scaling Transparency**: Allows the system and applications to expand in scale without altering system structure or application algorithms.

### 8. Quality of Service (QoS)
The ability of a distributed system to meet non-functional requirements experienced by users and clients:
- **Timeliness & Deadlines**: Crucial for real-time video, audio, and gaming (bounded latency and low jitter).
- **Reliability & Availability**: Ensuring continuous operation and minimum uptime.
- **Adaptability**: Dynamic reconfiguration when network conditions or resource availability fluctuate.

---

## 🔗 Quick Navigation

* 📖 **[Chapter 1 Study Guide](file:///C:/Desktop/Student%20Online%20%28SO%29/Techno/I4-GIC-A/I4-GIC-S1/Distributed%20Systems/C1-Charaterization%20of%20Distributed%20Systems/StudyGuide.md)** — Comprehensive deep dive, key diagrams, and full solutions for **TP01** and **TP02**.
* 📝 **[Chapter 1 Quiz Preparation](file:///C:/Desktop/Student%20Online%20%28SO%29/Techno/I4-GIC-A/I4-GIC-S1/Distributed%20Systems/C1-Charaterization%20of%20Distributed%20Systems/QuizPrep.md)** — Flashcards, Multiple-Choice Questions (MCQs), and mock exam questions with explanations.
