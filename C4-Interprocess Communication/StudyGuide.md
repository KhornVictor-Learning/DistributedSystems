# Chapter 4: Study Guide & Lab Solutions

> **Course**: Distributed Systems (I4-GIC-A / I4-GIC-S1) — ITC (Techno)  
> **Topic**: Interprocess Communication & Linux POSIX System Programming  
> **Textbook**: Coulouris et al., *Distributed Systems: Concepts and Design (5th Ed.)*, Chapter 4  
> **Included Labs**: Full solutions for **TP06** & comprehensive lab implementation guide for **TP07**

---

## 📑 Table of Contents
1. [Core Systems Programming & IPC Theory](#1-core-systems-programming--ipc-theory)
   - [Process Memory Layout & Attributes](#process-memory-layout--attributes)
   - [Process Lifecycle State Transitions](#process-lifecycle-state-transitions)
   - [Processes vs. Threads](#processes-vs-threads)
   - [The `fork()`, `exec()`, `wait()` Paradigm](#the-fork-exec-wait-paradigm)
   - [Zombies and Orphan Processes](#zombies-and-orphan-processes)
   - [Signals: Unreliable `signal()` vs. POSIX Reliable `sigaction()`](#signals-unreliable-signal-vs-posix-reliable-sigaction)
2. [TP06: Complete Solutions & Explanations](#tp06-complete-solutions--explanations)
   - [The `print.c` Process Tree Analysis](#the-printc-process-tree-analysis)
3. [TP07: Complete Lab Walkthrough & C Code Implementations](#tp07-complete-lab-walkthrough--c-code-implementations)
   - [Listing 1: `print-pid.c`](#listing-1-print-pidc)
   - [Listing 2: `system.c`](#listing-2-systemc)
   - [Listing 3: `fork.c`](#listing-3-forkc)
   - [Listing 4: `parent-child.c`](#listing-4-parent-childc)
   - [Listing 5: `fork-exec.c`](#listing-5-fork-execc)
   - [Listing 6: `clock.c`](#listing-6-clockc)
   - [Listing 7: `old-signal.c`](#listing-7-old-signalc)
   - [Listing 8: `new-signal.c`](#listing-8-new-signalc)
   - [Listing 9: `zombie.c`](#listing-9-zombiec)
   - [GDB & GDB-TUI Command Reference](#gdb--gdb-tui-command-reference)
4. [Exam Review Summary](#exam-review-summary)

---

## 1. Core Systems Programming & IPC Theory

### Process Memory Layout & Attributes

When a compiled C program is executed in Linux, the kernel creates an isolated virtual address space:

```
+------------------------------------+  High Memory (0xFFFFFFFF)
|           Kernel Space             |  (Protected OS Memory)
+------------------------------------+
|               Stack                |  (Grows downward: local variables, call frames)
|                 |                  |
|                 v                  |
|                                    |
|                 ^                  |
|                 |                  |
|               Heap                 |  (Grows upward: malloc, calloc, dynamic memory)
+------------------------------------+
|        BSS (Uninitialized Data)    |  (Zero-initialized globals/statics)
+------------------------------------+
|     Data (Initialized Data)        |  (Initialized globals/statics)
+------------------------------------+
|         Text (Code Segment)        |  (Read-only binary machine instructions)
+------------------------------------+  Low Memory (0x00000000)
```

Each process is tracked in the Linux kernel by a **Process Control Block (PCB)** (`task_struct`):
- **PID**: Unique Process Identifier (16-bit integer).
- **PPID**: Parent Process Identifier.
- **Process Credentials**: User ID (UID), Group ID (GID).
- **State**: Execution status (Running, Sleeping, Stopped, Zombie).
- **File Descriptor Table**: Pointers to open files and sockets (0 = `stdin`, 1 = `stdout`, 2 = `stderr`).
- **Scheduling Priority**: Priority (`PRI`) and `nice` value ($-20$ highest priority to $+19$ lowest).

---

### Process Lifecycle State Transitions

```
              +------------+
              |    NEW     |
              +------------+
                    | (Admitted / Forked)
                    v
  +---------> +------------+ (Scheduler Dispatch) +------------+
  |           |   READY    | --------------------> |  RUNNING   |
  |           +------------+ <-------------------- +------------+
  |                 ^           (Preemption / Interrupt) |
  |                 |                                    |
  | (Event Occurs)  |                                    | (I/O or Event Wait)
  |           +------------+                             v
  +---------- |  WAITING   | <---------------------------+
              |  (BLOCKED) |
              +------------+
                                (Process Calls exit() or Killed)
                                                         |
                                                         v
                                                   +------------+
                                                   | TERMINATED |
                                                   |  (ZOMBIE)  |
                                                   +------------+
                                                         | (Parent calls wait())
                                                         v
                                                   +------------+
                                                   | REAPED /   |
                                                   | DESTROYED  |
                                                   +------------+
```

1. **New**: The process is being created via `fork()`.
2. **Ready**: The process is loaded in memory and waiting for the CPU scheduler to allocate a time slice.
3. **Running**: The process instructions are actively executing on a CPU core.
4. **Waiting (Blocked / Sleeping)**: The process cannot proceed until an external event completes (e.g., waiting for disk I/O, network packet, or sleep timer).
5. **Terminated / Zombie**: The process has finished execution (`exit()`), but its PCB entry remains until its parent reaps it.

---

### Processes vs. Threads

| Metric | Process | Thread (POSIX `pthread`) |
| :--- | :--- | :--- |
| **Address Space** | Completely isolated virtual memory space. | Shared virtual address space within the parent process. |
| **Creation Cost** | High (memory pages cloned via Copy-On-Write). | Low (only stack and registers allocated). |
| **Context Switching** | Slow (requires invalidating CPU TLB and cache reload). | Fast (shares memory mappings, no TLB flush needed). |
| **Communication** | IPC required (Sockets, Pipes, Shared Memory). | Direct read/write to shared heap/global variables. |
| **Crash Isolation** | High: Crash in one process does not kill other processes. | Low: Segmentation fault in one thread crashes the entire process. |

---

### The `fork()`, `exec()`, `wait()` Paradigm

```c
pid_t pid = fork();
if (pid < 0) {
    /* Fork error */
} else if (pid == 0) {
    /* Child Process executes here */
    execvp(args[0], args);
} else {
    /* Parent Process executes here */
    int status;
    waitpid(pid, &status, 0); /* Wait for child to finish */
}
```

- **`fork()` clones memory**: After `fork()`, child and parent execute concurrently.
- **File Descriptor Inheritance**: Open file descriptors are duplicated into the child's file descriptor table, referencing the identical open file offset!

---

### Zombies and Orphan Processes

- **Zombie (`<defunct>`, STAT `Z`)**:
  - Occurs when a child process terminates, but the parent does not call `wait()`.
  - The process holds no RAM or CPU, but retains its slot in the kernel PID table. A leak of thousands of zombies exhausts the system's PID limit, preventing new processes from spawning.
- **Orphan**:
  - Occurs when the parent process exits *before* the child process.
  - The orphan is automatically adopted by the system root process (`init` or `systemd`, PID 1), which reaps the orphan when it eventually finishes.

---

### Signals: Unreliable `signal()` vs. POSIX Reliable `sigaction()`

1. **Legacy `signal()` Flaws**:
   - Resets signal disposition back to `SIG_DFL` immediately upon signal delivery. If a second signal arrives before the handler calls `signal(SIG, handler)` again, the default action (crash) is triggered.
   - Does not block other signals during handler execution, causing race conditions.
2. **POSIX `sigaction()` Reliability**:
   - The disposition remains permanently attached.
   - The `sa_mask` parameter specifies signals to atomically block while the handler is executing.
   - The `SA_RESTART` flag automatically restarts interrupted blocking system calls (e.g., `read()` or `accept()`).

---

## 📝 TP06: Complete Solutions & Explanations

### Question 1: What is a process?
**Answer**:
A **process** is an active, executing instance of a computer program. While a program is a passive collection of instructions stored on disk (an executable binary file), a process is an active entity loaded into RAM consisting of:
- Machine code instructions (Text segment).
- Program Counter (PC) tracking current execution.
- CPU register values.
- Memory segments (Data, BSS, Heap, Stack).
- An assigned set of OS resources (PID, file descriptors, network sockets).

---

### Question 2: What are the attributes and characteristics of a process? Explain.
**Answer**:
A process possesses the following key attributes managed in its Process Control Block (PCB):
1. **PID & PPID**: Unique 16-bit process identifier and the parent process identifier that spawned it.
2. **Process State**: Current scheduling status (`R`, `S`, `D`, `T`, `Z`).
3. **Address Space**: Isolated memory partitions (Text, Data, Heap, Stack).
4. **File Descriptor Table**: Pointers to open input/output channels (`stdin`, `stdout`, `stderr`, files, sockets).
5. **CPU Context / State**: Current registers, stack pointer, program counter, and floating-point registers saved during context switches.
6. **Security Credentials**: Real and Effective User IDs (UID) and Group IDs (GID) governing file access permissions.
7. **Scheduling Priority & Nice Value**: Defines the proportion of CPU time allocated by the OS scheduler.
8. **Resource Usage Statistics**: Memory footprint (VSZ, RSS), CPU execution time, and page fault counters.

---

### Question 3: What is the Process Life Cycle in Operating Systems? Explain each state.
**Answer**:
*(Refer to state transition diagram in Section 1)*
1. **New**: The process is being instantiated via system calls like `fork()`.
2. **Ready**: The process is fully initialized, resident in memory, and waiting in the CPU run queue for an available core.
3. **Running**: The CPU scheduler has dispatched the process; its instructions are actively executing on hardware.
4. **Waiting / Blocked (Sleeping)**: The process has relinquished the CPU because it is waiting for an external event (I/O operation completion, network packet, lock acquisition, or sleep timer).
5. **Terminated / Zombie**: Execution has finished (via `exit()` or fatal signal); waiting for the parent process to harvest its exit code via `wait()`.

---

### Question 4: What is a thread?
**Answer**:
A **thread** (often called a lightweight process or LWP) is the smallest independent unit of CPU scheduling and execution within an operating system. A thread belongs to a process and shares the process's code segment, global data, heap memory, and open file descriptors, but maintains its own:
- Private Program Counter (PC).
- Private CPU register set.
- Private Execution Stack (local function call frames).

---

### Question 5: Why Multithreading?
**Answer**:
1. **Enhanced Responsiveness**: In interactive applications, a background thread can perform long disk I/O or network requests while the UI thread remains smooth and responsive to user input.
2. **Resource Sharing & Memory Efficiency**: Threads within the same process share memory natively without requiring complex, slow IPC mechanisms (pipes, sockets, or shared memory segments).
3. **Low Creation & Context Switching Overhead**: Creating a thread is orders of magnitude cheaper than creating a process because no new virtual memory page tables or address spaces are allocated. Context switching between threads does not require flushing the CPU Translation Lookaside Buffer (TLB).
4. **Multicore Parallelism**: Threads allow a single application to execute across multiple physical CPU cores simultaneously, drastically speeding up computational tasks.

---

### Question 6: Explain the difference between thread and process.
**Answer**:

| Characteristic | Process | Thread |
| :--- | :--- | :--- |
| **Address Space** | Isolated; private memory space. | Shared within the parent process. |
| **Creation Overhead** | Heavyweight (clones memory tables). | Lightweight (allocates stack only). |
| **Context Switch Speed** | Slower (TLB flush, cache invalidation). | Extremely fast (shares address space). |
| **Communication** | Requires IPC (Sockets, Pipes, Signals). | Direct access to shared memory variables. |
| **Fault Isolation** | High (one process crash does not affect others). | Low (one thread crash can kill all threads). |

---

### Question 7: Is it conceivably useful for a port to have several receivers?
**Answer**:
**Yes, highly useful in specific modern networking architectures**:
1. **Standard TCP/UDP (Single Receiver Rule)**:
   - Traditionally, a local port is bound to exactly one receiving socket to prevent non-deterministic message delivery (ambiguity over which process gets the packet).
2. **When Multiple Receivers are Invaluable**:
   - **Kernel-Level Load Balancing (`SO_REUSEPORT`)**:
     - Modern Linux kernels allow multiple independent worker processes to bind to the **exact same port** using the `SO_REUSEPORT` socket option.
     - The Linux kernel automatically distributes incoming TCP connection requests across the listening processes using an internal round-robin / 4-tuple hash. This powers high-performance web servers like **Nginx** without requiring a userspace master proxy process.
   - **IP Multicast**:
     - For multicast UDP groups, multiple independent processes across the same host (or different hosts) bind to the same multicast port to simultaneously receive copies of the identical media/data stream.

---

### Question 8: The `print.c` Process Tree Analysis
**Question**: Run the program `print.c` to see how many "A", "B" and "C" will be printed?

Consider the standard POSIX process tree program:

```c
#include <stdio.h>
#include <unistd.h>

int main() {
    fork();
    printf("A\n");
    fork();
    printf("B\n");
    fork();
    printf("C\n");
    return 0;
}
```

#### Step-by-Step Mathematical Derivation:

```
Initially: 1 process (P0)

1. fork() #1:
   - P0 creates P1.
   - Total processes = 2.
   - Both execute printf("A\n"):
     ==> "A" is printed 2 times (2^1 = 2).

2. fork() #2:
   - Both P0 and P1 call fork(), creating P2 and P3.
   - Total processes = 4.
   - All 4 execute printf("B\n"):
     ==> "B" is printed 4 times (2^2 = 4).

3. fork() #3:
   - All 4 processes (P0, P1, P2, P3) call fork(), creating 4 new children.
   - Total processes = 8.
   - All 8 execute printf("C\n"):
     ==> "C" is printed 8 times (2^3 = 8).
```

#### Final Summary of Outputs:
- **"A" will be printed**: **2 times** ($2^1$)
- **"B" will be printed**: **4 times** ($2^2$)
- **"C" will be printed**: **8 times** ($2^3$)
- **Total output lines**: $2 + 4 + 8 = \mathbf{14}$ lines.
- **Total processes generated**: $2^3 = \mathbf{8}$ processes.

> [!IMPORTANT]
> **The `\n` Buffering Trap**: If `printf("A")` is executed **without** a newline `\n` and stdout is buffered, the string `"A"` remains in the userspace buffer and is **copied** to the child process during subsequent `fork()` calls, causing additional duplicate prints! With `\n` or `fflush(stdout)`, the buffer is flushed before each `fork()`, producing exactly 2 'A', 4 'B', and 8 'C'.

---

## 💻 TP07: Complete Lab Walkthrough & C Code Implementations

All programs below are standardized, bug-fixed, and fully commented for Linux/POSIX compilation.

### Listing 1: `print-pid.c`
*Prints current process ID and parent process ID.*

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>

int main() {
    printf("The process id is %d\n", (int) getpid());
    printf("The parent process id is %d\n", (int) getppid());
    return 0;
}
```
- **Compilation**: `gcc -Wall -g print-pid.c -o print-pid`
- **Observation**: `getpid()` changes on each run, while `getppid()` matches the shell PID (`echo $$`).

---

### Listing 2: `system.c`
*Executes a command via `/bin/sh`.*

```c
#include <stdlib.h>
#include <stdio.h>

int main() {
    int return_value;
    return_value = system("ls -l /");
    printf("Command exited with code: %d\n", return_value);
    return return_value;
}
```
- **Flaws of `system()`**: High overhead (spawns `/bin/sh`) and severe vulnerability to command injection if arguments come from user input.

---

### Listing 3: `fork.c`
*Demonstrates branching parent and child execution paths.*

```c
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>

int main() {
    pid_t child_pid;
    printf("The main program process id is %d\n", (int) getpid());

    child_pid = fork();
    if (child_pid != 0) {
        /* Parent process branch */
        printf("This is the parent process, with id %d\n", (int) getpid());
        printf("The child's process id is %d\n", (int) child_pid);
    } else {
        /* Child process branch */
        printf("This is the child process, with id %d\n", (int) getpid());
    }

    return 0;
}
```

---

### Listing 4: `parent-child.c`
*Demonstrates open file descriptor sharing and CPU scheduling competition.*

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>

int main() {
    int pid, i, fd;
    char c;

    fd = creat("TEST", 0644);
    if (fd < 0) {
        perror("creat failed");
        exit(1);
    }

    pid = fork();
    if (pid == 0) {
        /* Child process writes lowercase 'a' through 'e' */
        c = 'a';
        for (i = 0; i < 5; i++) {
            write(fd, &c, 1);
            c++;
            sleep(1);
        }
        printf("Child %d exit\n", getpid());
        close(fd);
        exit(0);
    } else {
        /* Parent process writes uppercase 'A' through 'E' */
        c = 'A';
        for (i = 0; i < 5; i++) {
            write(fd, &c, 1);
            c++;
            sleep(1);
        }
        printf("Parent %d process of %d exit\n", getpid(), pid);
        close(fd);
    }
    return 0;
}
```
- **Key Insight**: Both parent and child write to the shared file offset. The resulting `TEST` file contains interleaved characters like `AaBbCcDdEe`, proving concurrent non-deterministic scheduling.

---

### Listing 5: `fork-exec.c`
*Robust process creation using `fork()` and `execvp()` with parent `wait()` reaping.*

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <unistd.h>

int spawn(char* program, char** arg_list) {
    pid_t child_pid = fork();
    if (child_pid != 0) {
        /* Parent returns child PID */
        return child_pid;
    } else {
        /* Child executes new program */
        execvp(program, arg_list);
        /* execvp returns ONLY if an error occurred */
        fprintf(stderr, "An error occurred in execvp\n");
        abort();
    }
}

int main() {
    int child_status;
    char* arg_list[] = {
        "ls",
        "-l",
        "/",
        NULL /* Must be NULL-terminated */
    };

    pid_t child_pid = spawn("ls", arg_list);

    /* Parent waits for child to complete */
    waitpid(child_pid, &child_status, 0);

    if (WIFEXITED(child_status)) {
        printf("The child process exited normally, with exit code %d\n",
               WEXITSTATUS(child_status));
    } else if (WIFSIGNALED(child_status)) {
        printf("The child process exited abnormally by signal %d\n",
               WTERMSIG(child_status));
    }

    printf("Done with main program\n");
    return 0;
}
```

---

### Listing 6: `clock.c`
*Electronic digital clock driven by `alarm()` and `SIGALRM`.*

```c
#include <signal.h>
#include <time.h>
#include <stdio.h>
#include <string.h>
#include <unistd.h>

void tick(int nsig) {
    time_t tim;
    char *s;

    /* Re-arm alarm and reinstall handler */
    signal(SIGALRM, tick);
    alarm(1); /* Tick every 1 second */

    time(&tim);
    s = ctime(&tim);
    s[strlen(s) - 1] = '\0'; /* Strip newline */
    fprintf(stderr, "\rCurrent Time: %s", s);
}

int main() {
    tick(0);
    for (;;) {
        pause(); /* Suspend process until signal arrives */
    }
    return 0;
}
```

---

### Listing 7: `old-signal.c`
*Demonstrating legacy unreliable signal handling.*

```c
#include <signal.h>
#include <stdio.h>
#include <unistd.h>

static void sig_hndlr(int signo) {
    /* Must restore disposition because legacy signal() resets to SIG_DFL */
    signal(SIGINT, sig_hndlr);
    printf("\nReceived signal SIGINT (%d)\n", signo);
}

int main() {
    signal(SIGINT, sig_hndlr);
    signal(SIGUSR1, SIG_DFL); /* Default: terminates process */
    signal(SIGUSR2, SIG_IGN); /* Ignore signal */

    printf("Running old-signal (PID %d). Press Ctrl+C or send SIGUSR1/2\n", getpid());
    while (1) {
        pause();
    }
    return 0;
}
```

---

### Listing 8: `new-signal.c`
*POSIX reliable signal handling using `sigaction()`.*

```c
#include <signal.h>
#include <stdio.h>
#include <unistd.h>

void (*mysignal(int signo, void (*hndlr)(int)))(int) {
    struct sigaction act, oact;

    act.sa_handler = hndlr;
    sigemptyset(&act.sa_mask); /* Clear signal mask */
    act.sa_flags = 0;

    if (signo != SIGALRM) {
        act.sa_flags |= SA_RESTART; /* Automatically restart slow system calls */
    }

    if (sigaction(signo, &act, &oact) < 0) {
        return SIG_ERR;
    }
    return oact.sa_handler;
}

static void sig_hndlr(int signo) {
    /* No need to restore disposition! POSIX sigaction persists */
    printf("\nReceived reliable signal SIGINT (%d)\n", signo);
}

int main() {
    mysignal(SIGINT, sig_hndlr);
    mysignal(SIGUSR1, SIG_DFL);
    mysignal(SIGUSR2, SIG_IGN);

    printf("Running new-signal (PID %d). Send signals to test...\n", getpid());
    while (1) {
        pause();
    }
    return 0;
}
```

---

### Listing 9: `zombie.c`
*Creating and observing a zombie process.*

```c
#include <stdlib.h>
#include <sys/types.h>
#include <unistd.h>
#include <stdio.h>

int main() {
    pid_t child_pid = fork();

    if (child_pid > 0) {
        /* Parent process sleeps for 60 seconds without calling wait() */
        printf("Parent (PID %d) sleeping for 60s. Child is PID %d\n", getpid(), child_pid);
        sleep(60);
    } else if (child_pid == 0) {
        /* Child process terminates immediately */
        printf("Child (PID %d) exiting now...\n", getpid());
        exit(0);
    }

    return 0;
}
```
- **How to Inspect the Zombie**:
  Run in a separate terminal:
  ```bash
  ps -e -o pid,ppid,stat,cmd | grep zombie
  ```
  Output shows:
  ```
  1234  1000 S+   ./zombie
  1235  1234 Z+   [zombie] <defunct>
  ```
  Status **`Z+`** confirms the process is a zombie!

---

### GDB & GDB-TUI Command Reference

```bash
# Compilation with debugging symbols and zero optimization
gcc -g -O0 -Wall clock.c -o clock

# Start GDB in TUI mode
gdb -tui ./clock
```

| Command | Short | Action Description |
| :--- | :--- | :--- |
| `break main` | `b main` | Set a breakpoint at function `main`. |
| `break 25` | `b 25` | Set a breakpoint at line number 25. |
| `run` | `r` | Start executing the program. |
| `list` | `l` | Display source code surrounding current line. |
| `next` | `n` | Step over next line of code without entering functions. |
| `step` | `s` | Step into next line of code (enters function calls). |
| `continue` | `c` | Resume program execution until next breakpoint. |
| `print var` | `p var` | Print current value of variable `var`. |
| `info locals` | `i lo` | Display all local variables in current stack frame. |
| `watch var` | `watch var` | Stop execution whenever variable `var` changes value. |
| `info breakpoints`| `i b` | List all active breakpoints and hit counts. |
| `delete 1` | `d 1` | Delete breakpoint number 1. |
| `finish` | `fin` | Execute until current function returns. |
| `Ctrl + L` | — | Refresh/redraw corrupted TUI terminal window. |
| `quit` | `q` | Exit the GDB debugger. |

---

## 📊 Exam Review Summary

* **Exit Status Macros**:
  - `WIFEXITED(status)`: True if child terminated normally via `exit()`.
  - `WEXITSTATUS(status)`: Returns the 8-bit exit code ($0–255$).
  - `WIFSIGNALED(status)`: True if child was killed by an unhandled signal.
  - `WTERMSIG(status)`: Returns the fatal signal number.
* **Fork Tree Formula**:
  $$N_{\text{processes}} = 2^k \quad \text{after } k \text{ sequential fork() calls}$$
