# Chapter 2: Quiz Preparation & Exam Practice

> **Course Reference**: Distributed Systems (I4-GIC-A / I4-GIC-S1) — ITC (Techno)  
> **Topic**: System Models  
> **Textbook**: Coulouris et al., Chapter 2

---

## ⚡ Section 1: Flashcards & Quick-Recall Concepts

| Flashcard Front | Flashcard Back (Answer & Key Insight) |
| :--- | :--- |
| **What are the 3 major categories of System Models?** | 1. **Physical Models** (hardware & network connectivity).<br>2. **Architectural Models** (structure, roles, communication styles).<br>3. **Fundamental Models** (Interaction, Failure, and Security models). |
| **What is Space Uncoupling?** | Senders do not know and do not specify the identities or physical addresses of receivers (e.g., Publish-Subscribe topics). |
| **What is Time Uncoupling?** | Senders and receivers do not need to exist or be connected at the same time (e.g., persistent Message Queues). |
| **What is the difference between Synchronous and Asynchronous systems?** | **Synchronous**: Guaranteed bounds exist for message delays, clock drift, and execution times.<br>**Asynchronous**: No bounds exist on communication delays or execution speeds. |
| **What is a Thin Client?** | A software client that only handles local display rendering and user input, while all business logic and computational operations execute on a remote server (e.g., VNC, RDP). |
| **What is Byzantine Failure?** | An arbitrary failure mode where a component may omit steps, produce maliciously corrupted data, or send conflicting messages to different peers. |
| **What is Churn in P2P networks?** | The continuous, rapid process of peer nodes joining, leaving, and crashing unpredictably. |
| **What is the Tit-for-Tat algorithm in BitTorrent?** | An incentive mechanism where peers upload data primarily to those peers that actively upload data back to them at high rates, mitigating free-riding. |
| **What is Reflection in middleware?** | The capability of software to inspect its internal state and properties (**Introspection**) and dynamically adapt or reconfigure its behavior (**Intercession**). |
| **What is the Brokerage pattern?** | An architectural pattern composed of three entities: **Service Provider**, **Service Requester**, and **Service Broker (Registry)** (e.g., Java RMI Registry, CORBA Naming Service). |

---

## 🎯 Section 2: Multiple Choice Questions (MCQs)

### Question 1
Which of the following communication paradigms provides **both** space uncoupling and time uncoupling?
- A) Remote Procedure Call (RPC)
- B) Remote Method Invocation (RMI)
- C) Publish-Subscribe Systems
- D) Direct TCP Socket Streaming  
**Answer: C**  
*Explanation*: In Publish-Subscribe systems (e.g., Kafka, MQTT), publishers emit events without knowing the identity or count of subscribers (space uncoupled), and messages persist in queues/logs so subscribers can consume them hours later (time uncoupled).

---

### Question 2
What is the primary protocol difference between VNC and Microsoft Remote Desktop Protocol (RDP)?
- A) VNC operates over UDP, whereas RDP operates exclusively over ICMP.
- B) VNC transmits raw/compressed pixel rectangles (RFB), while RDP transmits high-level graphics drawing primitives.
- C) RDP requires hardware acceleration, while VNC cannot run in GUI mode.
- D) VNC provides space and time uncoupling.  
**Answer: B**  
*Explanation*: VNC's RFB protocol transmits display framebuffer pixels, making it cross-platform but bandwidth-intensive. RDP transmits GDI/DirectX drawing primitives, making it more bandwidth-efficient over high-latency networks.

---

### Question 3
Why can an asynchronous distributed system NOT guarantee that a crashed process will be detected with certainty?
- A) Asynchronous systems do not support TCP checksums.
- B) Network transmission delays are unbounded, making it impossible to distinguish between an infinitely slow message and a crashed node.
- C) Processes in asynchronous systems lack unique PIDs.
- D) Schedulers run only non-preemptive algorithms.  
**Answer: B**  
*Explanation*: Under the asynchronous model, message delays have no upper bound. If no response arrives within a timeout, the node cannot tell if the peer crashed or if the network is experiencing a transient, unbounded delay (FLM Impossibility Theorem).

---

### Question 4
In BitTorrent, why is the **rarest-first** piece selection strategy crucial for swarm health?
- A) It guarantees that seeders upload only to high-bandwidth leechers.
- B) It prevents rare file pieces from disappearing if the few peers holding them disconnect.
- C) It eliminates the need for SHA cryptographic checksums.
- D) It reduces the number of TCP sockets required.  
**Answer: B**  
*Explanation*: If peers prioritized downloading the most common pieces first, the rarest pieces would only exist on a few nodes. If those nodes leave, the swarm can never reconstruct the complete file.

---

### Question 5
What distinguishes a **computer cluster** from a loose collection of **multiple servers**?
- A) A cluster only runs Linux; multiple servers only run Windows.
- B) A cluster is co-located with dedicated high-speed low-latency interconnects, presenting a single virtual system image.
- C) A cluster cannot share storage arrays.
- D) Multiple servers must share a single quartz clock.  
**Answer: B**  
*Explanation*: Clusters are tightly coupled machines in the same data center interconnected with high-speed links (InfiniBand/100GbE) managed cooperatively to appear as a single unified computational resource.

---

## ✍️ Section 3: Short Answer & Concept Checks

### Q1: Describe the role of a Proxy Server in caching and firewall traversal.
**Sample Answer**:
- **Caching**: A proxy server intercepts client HTTP requests and stores local copies of retrieved objects. Subsequent requests for the same URL are fulfilled directly from cache, slashing WAN transit latency and external bandwidth usage.
- **Firewall Traversal**: Enterprise internal computers frequently reside on private IP subnets blocked from direct Internet routing. The forward proxy sits on the network boundary with a public interface, accepting internal requests, issuing outbound requests on behalf of the internal clients, and returning responses across the firewall.

### Q2: Contrast Introspection and Intercession in Reflective Middleware.
**Sample Answer**:
- **Introspection**: The ability of a system to inspect and query its own internal structure, configuration, and state at runtime (e.g., dynamically querying available methods and parameter types in Java RMI via reflection).
- **Intercession**: The ability of a system to modify and dynamically reconfigure its internal structure, execution strategy, or behavior at runtime (e.g., swapping a serialization protocol from JSON to binary Protobuf on-the-fly when bandwidth drops).

---

## 🧩 Section 4: Scenario-Based Problem Solving

### Problem 1: Concurrency Control in Search Indexing
**Scenario**: An online search engine processes 50,000 search queries per second on its inverted index. Simultaneously, a web crawler discovers 10,000 new web pages per minute that must be indexed.
1. What catastrophic issue occurs if a naive reader-writer mutual exclusion lock (`pthread_rwlock_wrlock`) is acquired by the index updater?
2. Propose an architectural pattern to achieve zero query disruption during updates.

**Solution**:
1. Acquiring an exclusive write lock blocks all 50,000 incoming query reads for the duration of the index write (which may take several seconds). This leads to massive request queueing, buffer overflow, and timeout failures for users.
2. **Double Buffering (Shadow Indexing)**:
   - The system maintains the **Active Index** (serving queries in read-only mode).
   - The crawler builds a separate **Shadow Index** in memory/disk without locking the active index.
   - Once updating completes, an atomic pointer swap instantaneously switches the query router to the new index, after which the old index is recycled.

---

### Problem 2: P2P Swarm Resilience Calculation
**Scenario**: A file is distributed across a P2P network. An individual peer workstation has an availability of $p = 0.6$ (meaning it is online $60\%$ of the day).
1. If a file chunk is stored on only 1 peer, what is the probability that the chunk is unavailable when requested?
2. If the chunk is replicated across $k = 6$ independent peers, what is the probability of chunk loss?

**Solution**:
1. Unavailability for $k = 1$:
   $$P(\text{unavailable}) = 1 - p = 1 - 0.6 = 0.4 \quad (40\% \text{ chance of failure})$$
2. Unavailability for $k = 6$ independent replicas:
   $$P(\text{all replicas offline}) = (1 - p)^k = (0.4)^6 = 0.004096 \approx 0.41\%$$
   *Availability jumps from $60\%$ to $99.59\%$.*
