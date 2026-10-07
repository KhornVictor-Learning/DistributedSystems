# Chapter 4: Interprocess Communication (IPC)

> **Course Reference**: Distributed Systems (I4-GIC-A / I4-GIC-S1)  
> **Institution**: Institute of Technology of Cambodia (ITC / Techno)  
> **Lecturer**: Khem Thay  
> **Textbook Reference**: *Distributed Systems: Concepts and Design (5th Edition)*, Chapter 4

---

## 📌 1. Introduction to Interprocess Communication

**Interprocess Communication (IPC)** refers to the software mechanisms, operating system primitives, and network protocols that enable distinct processes to exchange data and synchronize their execution across network boundaries.

In distributed computing, processes do not share physical memory; IPC operates fundamentally via **message passing**.

```
+-------------------------------------------------------------+
|                      IPC PROTOCOL STACK                     |
+-------------------------------------------------------------+
| High-Level Abstractions: RPC, RMI, Pub/Sub, Message Queues  |
+-------------------------------------------------------------+
| Transport Layer Sockets: TCP (Byte Stream) | UDP (Datagram) |
+-------------------------------------------------------------+
| Network Layer: Internet Protocol (IP)                       |
+-------------------------------------------------------------+
| OS Kernel Primitives: Sockets, Ports, Buffers, Descriptors   |
+-------------------------------------------------------------+
```

---

## 🔌 2. Low-Level Communication: Sockets & Ports

### Ports
- A **port** is an integer-identified destination endpoint within a host computer (ranging from `0` to `65535`).
- In standard Internet protocols, a port has **exactly one receiving process** (bound socket), but can receive messages from **many sending processes**.
- Ports `0–1023` are **well-known / reserved ports** (e.g., HTTP 80, SSH 22, SMTP 25).
- Ports `1024–49151` are registered ports; `49152–65535` are dynamic/ephemeral client ports.

### Sockets
- A **socket** is an operating system abstraction representing an endpoint of communication.
- A process binds its socket to a local `(IP Address, Port Number)` tuple to receive incoming traffic.

---

## ⚖️ 3. UDP Datagram vs. TCP Byte Stream

| Dimension | UDP (User Datagram Protocol) | TCP (Transmission Control Protocol) |
| :--- | :--- | :--- |
| **Communication Style** | Connectionless message passing (Datagrams). | Connection-oriented continuous two-way Byte Stream. |
| **Message Boundaries** | **Preserved**: Each `send()` corresponds to a discrete datagram packet received by `recv()`. | **No boundaries**: Data flows continuously; receiver must parse boundaries. |
| **Reliability** | **Unreliable**: Packets may be dropped, duplicated, or delivered out of order. No automatic retries. | **Reliable**: Guarantees in-order delivery, error detection, and automatic retransmission. |
| **Overhead** | Minimal (8-byte header; no connection handshake, no state kept in kernel). | Moderate (20-byte header; 3-way handshake SYN/ACK, ACK tracking, flow control). |
| **Buffer Constraints** | Practical message size limit of 8 KB (or IP MTU fragmentation limits). | Dynamic sliding window buffer; handles arbitrarily large data streams. |
| **Ideal Applications** | DNS queries, VoIP, live video streaming, multiplayer games. | HTTP/Web, SSH, File Transfer (FTP), Email (SMTP), Databases. |

---

## 🔄 4. Synchronous vs. Asynchronous Communication

- **Synchronous Communication**:
  - The sending process (or thread) **blocks** until the receiving process issues a corresponding receive operation and accepts the message.
  - Senders and receivers synchronize at every message exchange.
- **Asynchronous Communication**:
  - The `send` operation is **non-blocking**: the calling process hands the message to the OS kernel buffer and immediately resumes execution while the network stack transmits the packet in parallel.
  - The `receive` operation can be configured as blocking (waiting for data) or non-blocking (polling/event-driven).

---

## 🐧 5. Linux Process Management & POSIX System Calls

Interprocess communication in operating systems relies on understanding process lifecycles and kernel management:

### 5.1 Process vs. Thread
- **Process**: An executing instance of a program with an isolated, private virtual address space (Code, Data, Heap, Stack), file descriptor table, and Process Control Block (PCB). Heavyweight context switching.
- **Thread**: A lightweight unit of execution within a process. Multiple threads share the same address space, global variables, and file descriptors, but each possesses its own Program Counter (PC), CPU registers, and private stack.

### 5.2 Process Creation: `fork()` and `exec()`
1. **`fork()`**:
   - Clones the calling process to create an exact duplicate child process.
   - **Return values**:
     - Returns child's `PID` ($> 0$) to the parent process.
     - Returns `0` to the child process.
     - Returns `-1` on error.
2. **`exec()` Family (`execl`, `execvp`, `execve`, etc.)**:
   - Replaces the current process memory image with an entirely new executable program.
   - If successful, it **never returns** to the calling code.

### 5.3 Process Termination, Zombies & Orphans
- **`wait(&status)` / `waitpid(pid, &status, options)`**:
  - Suspends the parent process until a child process terminates, retrieving its exit status and releasing its PCB entry from the kernel process table.
- **Zombie Process (`<defunct>`, state `Z`)**:
  - A child process that has called `exit()`, but whose parent has not yet executed `wait()`. Its memory is freed, but its exit code remains in the kernel process table.
- **Orphan Process**:
  - A running child process whose parent died. The orphan is automatically adopted by the system initialization daemon (`init` / `systemd`, PID 1), which automatically calls `wait()` when the orphan terminates.

---

## 🔔 6. UNIX Signals & POSIX Reliable Signal Handling

A **signal** is an asynchronous kernel notification (software interrupt) dispatched to a process to report an event:
- Standard Signals: `SIGINT` (Ctrl+C, termination), `SIGTERM` (polite termination), `SIGKILL` (unconditional immediate kill, uncatchable), `SIGALRM` (timer expiration from `alarm()`), `SIGSEGV` (segmentation fault).

### Unreliable `signal()` vs. Reliable POSIX `sigaction()`
- **Legacy `signal()`**:
  - Unreliable: Whenever a signal is triggered, its disposition automatically resets back to `SIG_DFL` (default), requiring the programmer to re-install the handler within the handler itself (creating race conditions).
  - Cannot block signals during critical section execution.
- **POSIX `sigaction()`**:
  - Reliable: Retains signal disposition permanently.
  - Uses `sigset_t` masks (`sa_mask`) to atomically block specified signals while the signal handler executes, preventing signal corruption.
  - Supports `SA_RESTART` flag to automatically restart interrupted slow system calls.

---

## 🔗 Quick Navigation

* 📖 **[Chapter 4 Study Guide](file:///C:/Desktop/Student%20Online%20%28SO%29/Techno/I4-GIC-A/I4-GIC-S1/Distributed%20Systems/C4-Interprocess%20Communication/StudyGuide.md)** — Detailed C code walkthroughs, process state diagrams, complete solutions for **TP06**, and full lab guide for **TP07**.
* 📝 **[Chapter 4 Quiz Preparation](file:///C:/Desktop/Student%20Online%20%28SO%29/Techno/I4-GIC-A/I4-GIC-S1/Distributed%20Systems/C4-Interprocess%20Communication/QuizPrep.md)** — Flashcards, `fork()` output trace calculations, and exam questions with explanations.
