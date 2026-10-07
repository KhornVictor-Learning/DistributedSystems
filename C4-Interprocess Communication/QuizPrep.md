# Chapter 4: Quiz Preparation & Exam Practice

> **Course Reference**: Distributed Systems (I4-GIC-A / I4-GIC-S1) — ITC (Techno)  
> **Topic**: Interprocess Communication & Linux Systems Programming  
> **Textbook**: Coulouris et al., Chapter 4 & POSIX Lab Guide

---

## ⚡ Section 1: Flashcards & Quick-Recall Concepts

| Flashcard Front | Flashcard Back (Answer & Key Insight) |
| :--- | :--- |
| **What does `fork()` return in parent and child?** | Returns the **child's PID** ($> 0$) to the parent process; returns **0** to the child process; returns **-1** on failure. |
| **What is a Zombie Process?** | A process that has called `exit()` and terminated, but whose parent process has not yet executed `wait()` to collect its exit status. It shows status `Z` or `<defunct>`. |
| **What is an Orphan Process?** | A child process whose parent died while the child was still running. It is automatically adopted by the `init` / `systemd` daemon (PID 1). |
| **What does `execvp()` do upon success?** | Replaces the current process virtual memory, code, and data with a new binary program. Upon success, it **never returns** to the calling code. |
| **What is the difference between UDP and TCP?** | **UDP**: Unreliable, connectionless, preserves datagram packet boundaries, 8 KB typical buffer limit.<br>**TCP**: Reliable, connection-oriented byte stream with no message boundaries, handles flow control and retransmissions. |
| **Why is POSIX `sigaction()` preferred over `signal()`?** | `sigaction()` does not reset the signal disposition back to default upon signal delivery, and it atomically blocks specified signals during handler execution via `sa_mask`. |
| **What signal is sent by `alarm(seconds)`?** | `SIGALRM` is sent to the process when the timer expires. |
| **Which signals cannot be caught, blocked, or ignored?** | `SIGKILL` (kill immediately) and `SIGSTOP` (pause immediately). |
| **What macro extracts normal exit codes in `wait()`?** | `WEXITSTATUS(status)` (valid only if `WIFEXITED(status)` evaluates to true). |
| **What does the `SO_REUSEPORT` socket option allow?** | Allows multiple processes or threads to bind to the exact same IP and port; the kernel load-balances incoming connections across them. |

---

## 🎯 Section 2: Multiple Choice Questions (MCQs)

### Question 1
What is printed by the following C code snippet?
```c
int val = 10;
if (fork() == 0) {
    val += 5;
    exit(0);
} else {
    wait(NULL);
    printf("%d\n", val);
}
```
- A) 15
- B) 10
- C) 5
- D) Undefined / Random  
**Answer: B**  
*Explanation*: Processes have isolated address spaces. When `fork()` is called, the child receives a copy of the parent's memory (Copy-On-Write). The child modifies its own private copy of `val` (`15`), leaving the parent's `val` untouched at `10`.

---

### Question 2
In the output of `ps -e -o pid,stat,cmd`, what status indicator represents a process in an **uninterruptible disk sleep** state?
- A) `R`
- B) `S`
- C) `D`
- D) `T`  
**Answer: C**  
*Explanation*: `D` represents uninterruptible sleep (usually waiting for disk I/O). `R` is running/runnable; `S` is interruptible sleep; `T` is stopped (e.g., via SIGTSTP / Ctrl+Z).

---

### Question 3
Why must an argument list passed to `execvp(program, arg_list)` terminate with a `NULL` pointer?
- A) To clear CPU registers.
- B) So `execvp()` knows where the variable-length list of argument strings ends.
- C) To pass a zero exit status to the child.
- D) It is an optional convention for C99 compilers.  
**Answer: B**  
*Explanation*: In C, arrays do not store their own length. The `exec` functions iterate through `arg_list` until reaching a `NULL` sentinel pointer.

---

### Question 4
A server socket uses UDP. What happens if a sender transmits a 4 KB datagram, but the receiver's allocated buffer is only 1 KB?
- A) The remaining 3 KB is queued for the next `recvfrom()` call.
- B) The remaining 3 KB of data is discarded (truncated), and UDP flags an error or truncation warning.
- C) TCP is automatically spawned to negotiate buffer size.
- D) The packet is retransmitted automatically.  
**Answer: B**  
*Explanation*: UDP datagrams are atomic. If the receiving buffer is smaller than the incoming datagram, excess bytes are truncated and permanently lost; they are not saved for subsequent reads.

---

### Question 5
How does the `SA_RESTART` flag in `struct sigaction` affect system calls like `read()` or `accept()`?
- A) It reboots the operating system upon crash.
- B) It automatically restarts slow system calls if they are interrupted by the arrival of a handled signal, preventing `EINTR` errors.
- C) It resets the signal handler to default `SIG_DFL`.
- D) It restarts the parent process if the child dies.  
**Answer: B**  
*Explanation*: By default, signals cause blocking system calls to fail with error `EINTR` (Interrupted system call). Setting `SA_RESTART` causes the kernel to automatically resume the blocked call after the signal handler completes.

---

## ✍️ Section 3: Short Answer & Concept Checks

### Q1: Explain why a zombie process consumes system resources even though it has finished running.
**Sample Answer**:
When a process exits, the operating system reclaims its virtual memory, closed file descriptors, and CPU allocations. However, the kernel must keep its entry in the **Process Table** containing its exit code, termination status, and PID so that the parent process can retrieve them via `wait()`. If the parent never calls `wait()`, the PID entry persists indefinitely. Because operating systems have a finite maximum number of PIDs, leaking zombies can exhaust available PIDs, preventing the system from launching any new processes.

### Q2: How does `pstree` differ from standard `ps`?
**Sample Answer**:
Standard `ps` displays a tabular list or snapshot of running processes and their attributes. In contrast, `pstree` parses the Parent Process IDs (PPID) and renders all running processes visually as a tree hierarchy rooted at PID 1 (`systemd` or `init`), allowing developers to immediately inspect parent-child relationships and process ancestor chains.

---

## 🧩 Section 4: Code Tracing & Problem Solving

### Problem 1: Tracing Consecutive `fork()` Calls
**Scenario**: Trace the following C program:
```c
#include <stdio.h>
#include <unistd.h>

int main() {
    int i;
    for (i = 0; i < 3; i++) {
        fork();
    }
    printf("X\n");
    return 0;
}
```
1. How many total processes exist when the loop finishes?
2. How many times will `"X"` be printed?
3. Draw the process tree hierarchy.

**Solution**:
1. At each iteration $i$, every currently running process splits into two:
   - Initial: $2^0 = 1$ process.
   - Iteration $i=0$: $2^1 = 2$ processes.
   - Iteration $i=1$: $2^2 = 4$ processes.
   - Iteration $i=2$: $2^3 = 8$ processes.
   - Total processes created = **8 processes**.
2. Because all 8 processes exit the loop and execute `printf("X\n")`, `"X"` is printed **8 times**.
3. **Hierarchy**:
   ```
   P0 (root)
   ├── P1 (fork i=0)
   │   ├── P3 (fork i=1)
   │   │   └── P7 (fork i=2)
   │   └── P4 (fork i=2)
   ├── P2 (fork i=1)
   │   └── P6 (fork i=2)
   └── P5 (fork i=2)
   ```

---

### Problem 2: Exit Code Calculation
**Scenario**: A parent process executes:
```c
pid_t p = fork();
if (p == 0) {
    exit(42);
} else {
    int status;
    wait(&status);
    if (WIFEXITED(status)) {
        printf("Code: %d\n", WEXITSTATUS(status));
    }
}
```
1. What value is printed by `printf`?
2. What happens if the child process is terminated from a terminal with `kill -9 <child_pid>` before it calls `exit()`?

**Solution**:
1. The child exits normally with `42`. `WIFEXITED(status)` evaluates to true, and `WEXITSTATUS(status)` extracts the lower 8 bits, printing: `Code: 42`.
2. If killed by `kill -9` (`SIGKILL`), the child did **not** exit normally:
   - `WIFEXITED(status)` evaluates to **false**.
   - `WIFSIGNALED(status)` evaluates to **true**.
   - `WTERMSIG(status)` evaluates to **9** (the signal number for `SIGKILL`).
