# Chapter 2: System Models

> **Course Reference**: Distributed Systems (I4-GIC-A / I4-GIC-S1)  
> **Institution**: Institute of Technology of Cambodia (ITC / Techno)  
> **Lecturer**: Khem Thay  
> **Textbook Reference**: *Distributed Systems: Concepts and Design (5th Edition)*, Chapter 2

---

## 📌 1. Introduction: Why System Models?

Designing distributed systems is notoriously challenging due to:
- **Widely varying modes of use**: Highly unpredictable workloads, sudden traffic spikes, and shifting user patterns.
- **Wide range of system environments**: Massive heterogeneity across operating systems, network latencies, bandwidths, and hardware platforms.
- **Internal faults**: Lack of synchronized clocks, concurrent data conflicts, and independent crashes of hardware and software components.
- **External threats**: Unauthorized access, data interception, manipulation, and denial-of-service attacks.

To address these complexities, engineers employ three complementary categories of models:
1. **Physical Models**: Represent the underlying hardware components and their interconnectivity without low-level implementation details.
2. **Architectural Models**: Define the structure of the system in terms of computational elements, roles, communication paradigms, and placement strategies.
3. **Fundamental Models**: Formal abstractions that analyze core behavioral properties: communication delays (**Interaction Model**), system faults (**Failure Model**), and vulnerabilities (**Security Model**).

```
                            +-----------------------------------+
                            |       SYSTEM MODEL TAXONOMY       |
                            +-----------------------------------+
                               /              |              \
                              /               |               \
             +-----------------+     +-----------------+     +-----------------+
             | Physical Models |     |  Architectural  |     |   Fundamental   |
             |                 |     |     Models      |     |     Models      |
             +-----------------+     +-----------------+     +-----------------+
             - Baseline Model        - Communicating   - Interaction
             - Early (LAN)             Entities          (Sync/Async)
             - Internet-Scale        - Paradigms       - Failure
             - Contemporary            (Direct/Indirect) (Omission/Byzantine)
               (Mobile/Cloud)        - Roles (C-S/P2P) - Security
             - Systems of Systems    - Placement &       (Threats/Channels)
                                       Patterns
```

---

## 🖥️ 2. Physical Models

A **Physical Model** captures the physical nodes and network links that constitute a system.

### Baseline Physical Model
The minimal abstraction: an extensible set of autonomous computing nodes interconnected by a communication network that enables the passing of messages.

### The Generations of Distributed Systems

| Generation | Era / Scope | Node Count | Typical Infrastructure | Key Characteristics |
| :--- | :--- | :--- | :--- | :--- |
| **Early Distributed Systems** | 1970s – 1980s | 10 – 100 nodes | Local Area Networks (LAN) | Homogeneous or mildly heterogeneous; focused on local file and print sharing. |
| **Internet-Scale Systems** | 1990s – 2000s | Millions of nodes | Global Internet (WAN) | Cross-organizational boundaries; open standards (TCP/IP, HTTP); high heterogeneity. |
| **Contemporary Systems** | Mid-2000s – Present | Billions of nodes | Mobile, Wireless, Edge, Cloud | Dynamic node mobility, IoT, ubiquitous computing, elastic cloud utility data centers. |
| **Ultra-Large-Scale (ULS) Systems of Systems** | Modern Enterprise / Cloud | Heterogeneous complex subsystems | Global multi-cloud backbones | Systems composed of autonomous, collaborating complex subsystems (e.g., modern AWS/Google infrastructure). |

---

## 🏛️ 3. Architectural Models

An **Architectural Model** describes how software tasks are partitioned, mapped onto physical hosts, and coordinated.

### 3.1 Communicating Entities
- **System Perspective**: The communicating endpoints are **processes**. In multi-threaded systems, individual **threads** act as the concurrent communication execution contexts.
- **Programming Perspective**:
  - **Distributed Objects**: Encapsulate data and operations; accessed via remote invocations (e.g., Java RMI).
  - **Components**: Self-contained software units with explicit interfaces and contracts (e.g., Enterprise JavaBeans, CCM).
  - **Web Services**: Loosely coupled services using web standards (HTTP, REST, JSON, SOAP) integrated over the World Wide Web.

### 3.2 Communication Paradigms

```
                  COMMUNICATION PARADIGMS
                 /                       \
       [ Direct Communication ]      [ Indirect Communication ]
       - Sockets (IPC)                - Group Multicast
       - RPC (Remote Procedure Call)  - Publish-Subscribe
       - RMI (Remote Method Invocation) - Message Queues
       - Request-Reply Protocols      - Tuple Spaces
                                      - Distributed Shared Memory
```

#### The Decoupling Dimensions (Space and Time)
- **Direct Communication** (RPC/RMI): Sender and receiver are tightly coupled. The sender must know the identity/address of the receiver (**space coupled**), and both must be online simultaneously (**time coupled**).
- **Indirect Communication**: Mediated by an intermediary service, providing:
  - **Space Uncoupling**: Senders do not know or care who the receivers are (or how many exist).
  - **Time Uncoupling**: Senders and receivers do not need to exist at the same time (messages persist until consumers retrieve them).

| Indirect Technique | Decoupling Properties | Description & Example |
| :--- | :--- | :--- |
| **Group Multicast** | Space uncoupled; Time coupled | Messages sent to a group ID; delivered to all active joined members. |
| **Publish-Subscribe** | Space & Time uncoupled | Publishers emit events by topic; subscribers receive matching events asynchronously (e.g., Apache Kafka, JMS). |
| **Message Queues** | Space & Time uncoupled | Point-to-point queues; producers push messages, consumers pull when ready (e.g., RabbitMQ, IBM MQ). |
| **Tuple Spaces** | Space & Time uncoupled | Processes write and associatively read structured data records (tuples) in shared persistent memory (e.g., JavaSpaces, Linda). |
| **Distributed Shared Memory (DSM)** | Space coupled; Time coupled | Abstraction presenting a virtual shared memory across physically separated nodes. |

---

### 3.3 Roles and Responsibilities
1. **Client-Server Architecture**:
   - Asymmetric roles: Clients initiate requests; servers listen, process, and respond.
   - Servers may themselves become clients of other downstream backend servers (e.g., web server querying a database).
2. **Peer-to-Peer (P2P) Architecture**:
   - Symmetric roles: All participating nodes run identical software interfaces.
   - Shared data is partitioned and replicated across all peers.
   - System capacity scales organically as users join the network.

---

### 3.4 Placement Strategies

Placement determines how services and objects are mapped onto underlying hardware:

1. **Mapping Services to Multiple Servers**:
   - **Partitioning (Sharding)**: Dividing data subsets across multiple servers (e.g., DNS dividing domains).
   - **Replication**: Storing duplicate copies of resources across servers to boost availability and fault tolerance.
2. **Caching**:
   - Local storage of recently retrieved objects located closer to clients (browser cache or proxy cache) to eliminate WAN round-trips.
3. **Mobile Code**:
   - Executable code downloaded to the client and executed locally (e.g., JavaScript in browsers). Provides instant UI interactivity but poses security risks.
4. **Mobile Agents**:
   - Autonomous running programs that travel across nodes carrying code and data, performing tasks locally (e.g., web crawlers, diagnostic network probes).

---

### 3.5 Architectural Patterns

* **Layering**: Vertical organization of services into levels of abstraction (Platform $\to$ Middleware $\to$ Applications).
* **Tiered Architectures**: Horizontal organization of functionality across separate physical machines (e.g., 3-Tier: Presentation Tier $\to$ Application Logic Tier $\to$ Database Tier).
* **Thin Clients**: Minimal local UI display software executing user interfaces locally while running all heavy computations on a powerful remote server (e.g., VNC, Microsoft RDP, Citrix).
* **Proxy Pattern**: A local representative in the client's address space facilitating location transparency and caching.
* **Brokerage Pattern**: Trio of Service Provider, Service Requester, and Service Broker (Registry) enabling dynamic service discovery.
* **Reflection**: Enables systems to adapt dynamically via **Introspection** (discovering internal properties) and **Intercession** (modifying behavior at runtime).

---

## ⚖️ 4. Fundamental Models

Fundamental models strip away non-essential implementation details to analyze foundational limits:

### 4.1 Interaction Model
Analyzes communication delays and timing assumptions:
- **Synchronous Distributed Systems**:
  - Bound on process execution step time exists.
  - Bound on message transmission delay across channels exists.
  - Bound on local clock drift rate exists.
  - *Reality*: Rare in the public Internet; used in safety-critical real-time embedded systems.
- **Asynchronous Distributed Systems**:
  - No bounds on execution speeds, message delivery times, or clock drift.
  - *Reality*: Accurately models the Internet (congestion can delay packets indefinitely).

### 4.2 Failure Model
Defines how components can fail:
- **Omission Failures**: A process or channel fails to perform actions it was supposed to do (e.g., dropped packet, process crash).
  - *Crash-Stop*: Process halts permanently; other processes can detect it (in synchronous systems).
- **Arbitrary (Byzantine) Failures**: The worst-case failure mode. A process or channel omits steps, writes corrupted data, or acts maliciously.

### 4.3 Security Model
Analyzes vulnerabilities and protection mechanisms:
- Identifies threats to processes (impersonation, trojans) and communication channels (eavesdropping, tampering, replay attacks).
- Introduces the **Secure Channel** abstraction: provides mutual authentication, message integrity, and confidentiality.

---

## 🔗 Quick Navigation

* 📖 **[Chapter 2 Study Guide](file:///C:/Desktop/Student%20Online%20%28SO%29/Techno/I4-GIC-A/I4-GIC-S1/Distributed%20Systems/C2-System%20model/StudyGuide.md)** — In-depth breakdown and complete answers for **TP03** & **TP04**.
* 📝 **[Chapter 2 Quiz Preparation](file:///C:/Desktop/Student%20Online%20%28SO%29/Techno/I4-GIC-A/I4-GIC-S1/Distributed%20Systems/C2-System%20model/QuizPrep.md)** — Practice questions, comparison matrices, and mock exam questions.
