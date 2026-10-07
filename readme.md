# Distributed Systems (Systèmes Distribués)

> **Academic Course Reference**: I4-GIC-A / I4-GIC-S1  
> **Institution**: Institute of Technology of Cambodia (ITC / Techno) — Department of Information and Communication Engineering  
> **Lecturer**: Khem Thay  
> **Core Textbook**: *Distributed Systems: Concepts and Design (5th Edition)* — George Coulouris, Jean Dollimore, Tim Kindberg, Gordon Blair (Addison-Wesley / Pearson, 2012)

---

## 📌 Course Overview

A **distributed system** is a computing environment in which hardware and software components located at networked computers communicate and coordinate their actions solely by **passing messages**. 

Modern digital infrastructure—ranging from cloud platforms (AWS, GCP, Azure), search engines, microservices architectures, distributed databases (Cassandra, Spanner), to peer-to-peer protocols (BitTorrent, blockchain)—operates fundamentally as distributed systems.

Unlike traditional single-machine computing, distributed systems introduce fundamental challenges:
1. **Concurrency**: Multiple autonomous processes execute simultaneously on geographically separated hardware.
2. **Absence of a Global Clock**: Because of network delays and clock drift, there is no shared physical clock across nodes; coordination must rely on message passing and logical ordering.
3. **Independent (Partial) Failures**: Individual nodes or network links can fail without warning while the rest of the system continues to operate. A component may not immediately know if another node has crashed or is merely experiencing network delay.

---

## 🎯 Learning Outcomes

By completing this course and its associated practical laboratories (TPs), students will master:
- **Foundational Concepts & Trends**: Ubiquitous computing, pervasive networking, distributed multimedia, and cloud utility models.
- **Architectural & Physical Modeling**: Client-server vs. Peer-to-Peer, direct vs. indirect communication paradigms (space and time uncoupling), layering, tiering, thin-client models, and reflective middleware.
- **Communication Infrastructure**: Network types (PAN, LAN, MAN, WAN, wireless), performance quantification ($\text{Transmission Time} = \text{Latency} + \frac{\text{Length}}{\text{Transfer Rate}}$), firewalls, bastion hosts, and VPNs.
- **Interprocess Communication (IPC)**: Socket programming with UDP datagrams and TCP byte streams, message marshaling, producer-consumer paradigms, and group multicast.
- **Low-Level POSIX System Programming**: Linux process lifecycles, process creation (`fork`), program execution (`execvp`/`execl`), parent-child synchronization (`wait`/`waitpid`), process zombies and orphans, POSIX reliable signal handling (`sigaction`), and debugging with **GDB / GDB-TUI**.

---

## 🗂️ Course Structure & Chapters

The course is organized into four core theoretical chapters and seven practical lab sessions (TPs):

```
Distributed Systems/
├── .gitignore                                      # Ignores heavy slides, binaries, core dumps
├── readme.md                                       # Course master syllabus & roadmap (this file)
│
├── C1-Charaterization of Distributed Systems/       # Chapter 1
│   ├── readme.md                                   # Comprehensive Lecture Overview
│   ├── StudyGuide.md                               # Study Notes & TP01 / TP02 Solutions
│   ├── QuizPrep.md                                 # Flashcards, MCQs & Practice Exam Questions
│   ├── C01.pdf / C01.pptx                          # Lecture slides
│   ├── TP01.pdf                                    # TP01 Lab questions
│   └── TP02.pdf                                    # TP02 Lab questions
│
├── C2-System model/                                # Chapter 2
│   ├── readme.md                                   # Lecture Overview & Architectural Models
│   ├── StudyGuide.md                               # Study Notes & TP03 / TP04 Solutions
│   ├── QuizPrep.md                                 # Flashcards, MCQs & Practice Exam Questions
│   ├── C02.pdf / C02.pptx                          # Lecture slides
│   ├── TP03.pdf                                    # TP03 Lab questions
│   └── TP04.pdf                                    # TP04 Lab questions
│
├── C3-Networking and Internetworking/              # Chapter 3
│   ├── readme.md                                   # Lecture Overview & Network Infrastructure
│   ├── StudyGuide.md                               # Study Notes & TP05 Solutions
│   ├── QuizPrep.md                                 # Flashcards, MCQs & Practice Exam Questions
│   ├── C03.pdf / C03.pptx                          # Lecture slides
│   └── TP05.pdf                                    # TP05 Lab questions
│
└── C4-Interprocess Communication/                  # Chapter 4
    ├── readme.md                                   # Lecture Overview, Sockets & Process Control
    ├── StudyGuide.md                               # Study Notes & TP06 / TP07 Lab Solutions
    ├── QuizPrep.md                                 # Flashcards, MCQs & Practice Exam Questions
    ├── C04.pdf / C04.pptx                          # Lecture slides
    ├── TP06.pdf                                    # TP06 Lab questions
    └── TP07.pdf                                    # TP07 Lab guide (POSIX System Programming)
```

---

## 📚 Curriculum Breakdown

### [Chapter 1: Characterization of Distributed Systems](file:///C:/Desktop/Student%20Online%20%28SO%29/Techno/I4-GIC-A/I4-GIC-S1/Distributed%20Systems/C1-Charaterization%20of%20Distributed%20Systems/readme.md)
* **Topics Covered**:
  - Formal definition and primary characteristics (concurrency, lack of global clock, independent failures).
  - Modern industry trends: Pervasive networking, mobile & ubiquitous computing, distributed multimedia, cloud computing as a utility.
  - Resource sharing models, services, clients, and remote invocation.
  - The 8 Fundamental Challenges:
    1. *Heterogeneity* (hardware, OS, languages, networks; resolved via middleware and virtual machines).
    2. *Openness* (hardware and software extensibility through published interfaces).
    3. *Security* (Confidentiality, Integrity, Availability).
    4. *Scalability* (cost control, performance degradation, software resource limits, avoiding bottlenecks; solved via caching, replication, load distribution).
    5. *Failure Handling* (detecting, masking, tolerating, recovery, redundancy).
    6. *Concurrency* (race conditions, consistency, mutual exclusion via semaphores).
    7. *Transparency* (the 8 forms: Access, Location, Concurrency, Replication, Failure, Mobility, Performance, Scaling).
    8. *Quality of Service (QoS)* (timeliness guarantees, throughput, adaptability).
* **Practical Labs**:
  - [TP01](file:///C:/Desktop/Student%20Online%20%28SO%29/Techno/I4-GIC-A/I4-GIC-S1/Distributed%20Systems/C1-Charaterization%20of%20Distributed%20Systems/StudyGuide.md#tp01-complete-solutions--explanations): Server forwarding, P2P vs Client-Server, Web browsing architectures, Endianness, Transparencies, and QoS metrics.
  - [TP02](file:///C:/Desktop/Student%20Online%20%28SO%29/Techno/I4-GIC-A/I4-GIC-S1/Distributed%20Systems/C1-Charaterization%20of%20Distributed%20Systems/StudyGuide.md#tp02-complete-solutions--explanations): Hardware vs software resource sharing, clock synchronization limitations, ubiquitous service discovery, Cloud vs Client-Server, BLOB object access security, multicast mobility trade-offs, and URL parsing.
* **Study Resources**:
  - 📖 [Chapter 1 Study Guide](file:///C:/Desktop/Student%20Online%20%28SO%29/Techno/I4-GIC-A/I4-GIC-S1/Distributed%20Systems/C1-Charaterization%20of%20Distributed%20Systems/StudyGuide.md)
  - 📝 [Chapter 1 Quiz Preparation](file:///C:/Desktop/Student%20Online%20%28SO%29/Techno/I4-GIC-A/I4-GIC-S1/Distributed%20Systems/C1-Charaterization%20of%20Distributed%20Systems/QuizPrep.md)

---

### [Chapter 2: System Models](file:///C:/Desktop/Student%20Online%20%28SO%29/Techno/I4-GIC-A/I4-GIC-S1/Distributed%20Systems/C2-System%20model/readme.md)
* **Topics Covered**:
  - The Three Pillars of System Modeling: **Physical Models**, **Architectural Models**, and **Fundamental Models**.
  - Physical Model Evolution: Baseline model, early networks (10–100 nodes), Internet-scale, contemporary mobile/cloud, and Ultra-Large-Scale (ULS) Systems of Systems.
  - Architectural Styles & Paradigms:
    - Communicating entities: processes vs. threads, distributed objects, components, web services.
    - Direct communication (IPC, RPC, RMI) vs. Indirect communication (Group communication, Publish-Subscribe, Message Queues, Tuple Spaces, Distributed Shared Memory).
    - Decoupling dimensions: *Space uncoupling* and *Time uncoupling*.
    - Roles: Client-Server vs. Peer-to-Peer architectures.
    - Placement Strategies: Multiple server partitioning, proxy caching, mobile code (applets/push model), and mobile agents.
    - Architectural Patterns: Layering, Tiering, Thin Clients (Remote Desktop/VNC), Proxies, Service Brokerage, and Reflection (Introspection & Intercession).
  - Fundamental Models:
    - *Interaction Model*: Synchronous vs. Asynchronous distributed systems, latency, clock drift.
    - *Failure Model*: Omission failures, crash-stop, crash-recovery, Byzantine (arbitrary) failures.
    - *Security Model*: Securing channels, protecting processes, authentication, cryptography.
* **Practical Labs**:
  - [TP03](file:///C:/Desktop/Student%20Online%20%28SO%29/Techno/I4-GIC-A/I4-GIC-S1/Distributed%20Systems/C2-System%20model/StudyGuide.md#tp03-complete-solutions--explanations): Remote desktop (VNC/RDP/TeamViewer), OS and DB semaphores, BitTorrent P2P mechanisms, Clusters vs. Multiple Servers, and Proxy server roles.
  - [TP04](file:///C:/Desktop/Student%20Online%20%28SO%29/Techno/I4-GIC-A/I4-GIC-S1/Distributed%20Systems/C2-System%20model/StudyGuide.md#tp04-complete-solutions--explanations): Centralized vs. Decentralized vs. Distributed systems, direct coupling vs. space/time decoupling, Web search engine concurrency & crawler synchronization, P2P desktop availability & replication, bounded execution limitations, NTP clock drift bounds, and Internet security threat categories.
* **Study Resources**:
  - 📖 [Chapter 2 Study Guide](file:///C:/Desktop/Student%20Online%20%28SO%29/Techno/I4-GIC-A/I4-GIC-S1/Distributed%20Systems/C2-System%20model/StudyGuide.md)
  - 📝 [Chapter 2 Quiz Preparation](file:///C:/Desktop/Student%20Online%20%28SO%29/Techno/I4-GIC-A/I4-GIC-S1/Distributed%20Systems/C2-System%20model/QuizPrep.md)

---

### [Chapter 3: Networking and Internetworking](file:///C:/Desktop/Student%20Online%20%28SO%29/Techno/I4-GIC-A/I4-GIC-S1/Distributed%20Systems/C3-Networking%20and%20Internetworking/readme.md)
* **Topics Covered**:
  - The Communication Subsystem: Hosts, nodes, subnets, protocol stacks.
  - Performance Quantification:
    $$\text{Message Transmission Time} = \text{Latency} + \frac{\text{Message Length}}{\text{Data Transfer Rate}}$$
    Comparing LAN (1–10 ms, 10–10000 Mbps) vs. WAN (100–500 ms, 0.01–600 Mbps).
  - Network Classification: Wired PAN, LAN (Ethernet), MAN (DSL/ATM), WAN (IP routing), and Wireless counterparts (Bluetooth WPAN, Wi-Fi WLAN 802.11, WiMAX WMAN 802.16, Cellular 3G/4G/5G WWAN).
  - Internetworking Architecture: Heterogeneous subnets, routers, gateways, routing tables, and BGP/CIDR hierarchical routing.
  - Network Security & Perimeter Defense:
    - Firewalls: Service control, behavior control, and user control.
    - Bastion hosts and Dual-router DMZ architectures.
    - Virtual Private Networks (VPNs): Extending intranet security across public IP links using cryptographically protected tunnels (IPsec/TLS).
* **Practical Labs**:
  - [TP05](file:///C:/Desktop/Student%20Online%20%28SO%29/Techno/I4-GIC-A/I4-GIC-S1/Distributed%20Systems/C3-Networking%20and%20Internetworking/StudyGuide.md#tp05-complete-solutions--explanations): Gateways, the 5 core cryptographic techniques (Symmetric, Asymmetric, Hash functions, Digital Signatures, PKI Certificates), Public vs. Private IP (RFC 1918 & NAT), Telnet vs. SSH, VPN tunnel mechanics, Internet routing scalability, Ethernet switch frame forwarding, IP uniqueness, transmission time calculations, and network threat modeling.
* **Study Resources**:
  - 📖 [Chapter 3 Study Guide](file:///C:/Desktop/Student%20Online%20%28SO%29/Techno/I4-GIC-A/I4-GIC-S1/Distributed%20Systems/C3-Networking%20and%20Internetworking/StudyGuide.md)
  - 📝 [Chapter 3 Quiz Preparation](file:///C:/Desktop/Student%20Online%20%28SO%29/Techno/I4-GIC-A/I4-GIC-S1/Distributed%20Systems/C3-Networking%20and%20Internetworking/QuizPrep.md)

---

### [Chapter 4: Interprocess Communication](file:///C:/Desktop/Student%20Online%20%28SO%29/Techno/I4-GIC-A/I4-GIC-S1/Distributed%20Systems/C4-Interprocess%20Communication/readme.md)
* **Topics Covered**:
  - Low-Level IPC Mechanisms: Sockets, ports, `(IP, Port)` tuples, non-blocking vs. blocking operations.
  - UDP Datagrams: Unreliable, connectionless, 8 KB packet constraints, omission failures, absence of state.
  - TCP Byte Streams: Reliable, connection-oriented, flow control, sequence numbering, acknowledgment, producer-consumer stream abstraction.
  - Advanced IPC Paradigms: Group multicast, publish-subscribe, message queues, tuple spaces, distributed shared memory.
  - Operating System Process Management:
    - Process vs. Thread (PCB vs. TCB, address space sharing, context switching overhead).
    - Process Lifecycle: New, Ready, Running, Blocked/Waiting, Terminated.
  - POSIX System Calls & C System Programming:
    - Process inspection: `ps`, `pstree`, `top`, `kpm`.
    - Process creation: `fork()`, parent vs. child execution paths.
    - Program execution: `exec()` family (`execvp`, `execlp`, `execve`).
    - Process termination & reaping: `wait()`, `waitpid()`, macros (`WIFEXITED`, `WEXITSTATUS`, `WTERMSIG`), Zombie processes (`<defunct>`, state `Z`), and Orphan adoption by `init` (PID 1).
    - Signal management: Unreliable `signal()` vs. POSIX reliable `sigaction()` (`sigemptyset`, `sa_mask`, `SA_RESTART`), `alarm()` & `SIGALRM`.
    - Debugging: GNU Debugger (`gdb`, `gdb -tui`, breakpoints, stepping, `info locals`, variable watching).
* **Practical Labs**:
  - [TP06](file:///C:/Desktop/Student%20Online%20%28SO%29/Techno/I4-GIC-A/I4-GIC-S1/Distributed%20Systems/C4-Interprocess%20Communication/StudyGuide.md#tp06-complete-solutions--explanations): Process definitions, attributes, lifecycle states, thread vs. process comparison, port receiver cardinality, and full trace of the `print.c` fork tree.
  - [TP07](file:///C:/Desktop/Student%20Online%20%28SO%29/Techno/I4-GIC-A/I4-GIC-S1/Distributed%20Systems/C4-Interprocess%20Communication/StudyGuide.md#tp07-complete-lab-walkthrough--c-code-implementations): Complete lab walkthrough with production-ready C programs: `print-pid.c`, `system.c`, `fork.c`, `parent-child.c`, `fork-exec.c`, `clock.c`, `old-signal.c`, `new-signal.c`, `zombie.c`, and GDB debugging workflows.
* **Study Resources**:
  - 📖 [Chapter 4 Study Guide](file:///C:/Desktop/Student%20Online%20%28SO%29/Techno/I4-GIC-A/I4-GIC-S1/Distributed%20Systems/C4-Interprocess%20Communication/StudyGuide.md)
  - 📝 [Chapter 4 Quiz Preparation](file:///C:/Desktop/Student%20Online%20%28SO%29/Techno/I4-GIC-A/I4-GIC-S1/Distributed%20Systems/C4-Interprocess%20Communication/QuizPrep.md)

---

## 🛠️ How to Prepare for Exams & Practical Labs

1. **Read the Chapter Readme first**: Understand the architectural models, high-level motivations, and theoretical concepts before writing code.
2. **Review the Study Guide**: Work through the complete solutions to the TPs. The exam questions at ITC (Techno) are drawn directly from the exercises in TP01 through TP07 and Coulouris textbook problems.
3. **Self-Test with QuizPrep**: Use the flashcards and multiple-choice questions to test your conceptual recall, and practice writing the short-answer/essay questions under timed conditions.
4. **Compile and Trace C Code**: For Chapter 4 (TP06 & TP07), compile the programs on Linux or WSL (`gcc -Wall -g file.c -o file`) and run them in GDB (`gdb ./file`) to observe process forks, zombie states, and signal handler executions firsthand.
