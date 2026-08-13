
### Lecture Notes (based on _Operating System Concepts_, 10th Edition — Silber Schatz, Galvin & Gagne)

---

## Learning Objectives

By the end of this chapter you should be able to:

- Describe the critical-section problem and illustrate a race condition.
- Explain hardware solutions to the critical-section problem: memory barriers, compare-and-swap, and atomic variables.
- Demonstrate how mutex locks, semaphores, monitors, and condition variables solve the critical-section problem.
- Evaluate synchronization tools under low-, moderate-, and high-contention scenarios.

---

## 1. Background

**Core idea:** When processes execute concurrently, the OS may interrupt any process at any point, letting it complete only part of its work before switching to another process. If two or more processes access shared data at the same time, the final result can depend on the _order_ in which their instructions happen to interleave — this is unsafe and unpredictable.

- Concurrent access to shared data can cause **data inconsistency**.
- To keep data consistent, we need mechanisms that guarantee an **orderly execution** of cooperating processes.

### Summary of the Previously Referenced Chapter (Chapter 4 — Threads & Concurrency)

The slides refer back to an example "illustrated in chapter 4," where the **Bounded-Buffer (producer–consumer) problem** was used to introduce the idea of a **race condition**. In that earlier chapter, a shared `count` variable was incremented by a producer thread each time it added an item to the buffer, and decremented by a consumer thread each time it removed an item. Because `count++` and `count--` are not single, indivisible machine operations (each compiles into a load, a modify, and a store), two threads executing them at the same time can interleave their instructions and leave `count` with an incorrect final value. That earlier discussion was meant to _motivate_ the need for the synchronization tools that Chapter 6 goes on to formally define — mutual exclusion, hardware instructions, mutex locks, semaphores, and monitors.

### Race Condition Example — `fork()` and Process IDs

- Two processes, **P₀** and **P₁**, each call `fork()` to create a child process.
- Both requests read and update a shared kernel variable `next_available_pid`, which holds the next process ID to hand out.
- **Timeline of the problem:**
    1. Both P₀ and P₁ request a pid at roughly the same time.
    2. Both read `next_available_pid` = 2615 _before_ either one updates it.
    3. Both children are (incorrectly) assigned pid **2615**.
- **Lesson:** Unless access to `next_available_pid` is protected, two different processes can end up with the same PID — a race condition caused by unsynchronized access to shared kernel data.

---

## 2. The Critical-Section Problem

- Consider a system of _n_ processes `{p₀, p₁, …, pₙ₋₁}`.
- Each process contains a **critical section (CS)**: a segment of code where it accesses/modifies data shared with other processes (a common variable, a table, a file, etc.).
- **Key rule:** while one process is inside its critical section, _no other process_ may be inside its own critical section.
- The **critical-section problem** is to design a protocol (entry/exit logic) that processes use to safely coordinate access.

### General structure of a process

```
while (true) {
    entry section        // request permission to enter
    critical section      // access shared data
    exit section           // release permission
    remainder section    // all other code
}
```

### The Three Requirements for a Correct Solution

1. **Mutual Exclusion** — If process Pᵢ is executing in its critical section, no other process may execute in its own critical section at the same time. _(This is the fundamental safety property — without it, shared data can be corrupted.)_
2. **Progress** — If no process is currently in its critical section, and some processes wish to enter theirs, the decision of who goes next cannot be postponed forever. Only processes _not_ in their remainder section may participate in that decision, and it must complete in a finite amount of time. _(This prevents the system from deadlocking or "freezing" the decision indefinitely.)_
3. **Bounded Waiting** — There must be a limit on how many times _other_ processes are allowed to enter their critical sections after a process has requested entry and before that request is granted. _(This prevents starvation — a process waiting forever while others repeatedly cut in line.)_
    - Assumption: every process executes at a nonzero speed.
    - No assumption is made about the _relative_ speeds of the _n_ processes (i.e., no process may be assumed to be arbitrarily faster or slower than another).

---

## 3. Early (Flawed) Approaches

### 3.1 Interrupt-Based Solution

- **Idea:** In the entry section, disable interrupts; in the exit section, re-enable them. While interrupts are off, the running code cannot be preempted.
- **Problems:**
    - If the critical section runs for a long time (e.g., an hour), the entire machine is unresponsive that whole time.
    - Other processes may **starve** — never get a turn.
    - It **does not work on multiprocessor systems**, because disabling interrupts on one CPU has no effect on the other CPUs, which can still access the shared data.

### 3.2 Software Solution 1 (the `turn` variable)

- Designed for exactly **two** processes.
- Assumes `load` and `store` machine instructions are **atomic** (cannot be interrupted mid-execution).
- Shared variable: `int turn;` — indicates whose turn it is to enter the critical section. Initially `turn = i` (some fixed default).

```
while (true) {
    while (turn == j) ;   // busy-wait until it's my turn
    /* critical section */
    turn = j;              // hand control to the other process
    /* remainder section */
}
```

**Correctness analysis:**

- **Mutual exclusion** _is_ preserved: Pᵢ can only enter when `turn == i`, and `turn` cannot simultaneously equal both 0 and 1.
- **Progress** is _violated_: the solution forces strict alternation. If Pᵢ finishes and neither process wants to re-enter immediately, but it is Pⱼ's `turn` and Pⱼ is not ready, Pᵢ cannot re-enter even though the critical section is free — the CPU may sit idle waiting on a process that has no interest in entering.
- **Bounded waiting** question is left open because of the progress failure above.

This limitation motivates a better two-process algorithm: **Peterson's Solution**.

---



## 4. Peterson's Solution

- A classic two-process algorithm assuming atomic `load`/`store`.
- Shares **two** variables instead of one:
    - `int turn;` — whose turn it is to enter.
    - `boolean flag[2];` — `flag[i] == true` means process Pᵢ is _ready_ to enter its critical section.

```
while (true) {
    flag[i] = true;
    turn = j;
    while (flag[j] && turn == j) ;   // wait only if the other wants in AND it's their turn
    /* critical section */
    flag[i] = false;
    /* remainder section */
}
```

### Why It Works

- Pᵢ enters the critical section only when **either** `flag[j] == false` (the other process doesn't want in) **or** `turn == i` (it's genuinely Pᵢ's turn).
- **Mutual exclusion** is provably preserved.
- **Progress** is satisfied — a waiting process is never blocked by a process that isn't trying to enter.
- **Bounded waiting** is satisfied — a waiting process will be let in after, at most, one entry by the other process.

### Peterson's Solution on Modern Architectures — A Caveat

- Although Peterson's Solution is mathematically correct and great for **teaching** the concept, it is **not guaranteed to work on modern hardware**.
- **Reason:** Modern CPUs and compilers may **reorder instructions** that have no data dependency between them, in order to improve performance.
- For **single-threaded** programs this reordering is harmless — the final observable result of that one thread is unaffected.
- For **multithreaded** programs, however, reordering can let another thread observe memory changes out of the order the programmer wrote them, breaking correctness.

**Illustrative example:**

```
Shared:  boolean flag = false;  int x = 0;

Thread 1:                Thread 2:
while (!flag) ;              x = 100;
print x;                      flag = true;
```

- The programmer expects Thread 1 to always print `100`.
- But since `x` and `flag` are logically independent, Thread 2's compiler/CPU might reorder the two statements to execute `flag = true` _before_ `x = 100`.
- If that reordering happens, Thread 1 could see `flag == true` and print `x` before it has been set — printing **0** instead of the expected 100.

**Applied to Peterson's Solution:** if the statements `turn = j;` and `flag[i] = true;` (or their equivalents) are reordered by the hardware, both processes can end up believing it is safe to enter — and **both** end up inside the critical section simultaneously, violating mutual exclusion.

### Memory Barriers — The Fix

- **Memory model:** the guarantees a computer architecture makes about when a memory write becomes visible to other processors.
    - **Strongly ordered:** a write by one processor is immediately visible to every other processor.
    - **Weakly ordered:** a write may _not_ be immediately visible to other processors.
- **Memory barrier (memory fence):** a special instruction that forces all pending loads/stores to complete and become visible to other processors _before_ any subsequent load/store is allowed to execute.

**Fixed example using memory barriers:**

```
Thread 1:                     Thread 2:
while (!flag)                    x = 100;
    memory_barrier();             memory_barrier();
print x;                           flag = true;
```

- Thread 1's barrier guarantees `flag` is read _before_ `x` is read.
- Thread 2's barrier guarantees `x = 100` is completed and visible _before_ `flag = true` is executed.
- Together, these barriers restore the intended ordering and guarantee Thread 1 prints 100.

---

## 5. Hardware Support for Synchronization

Many architectures provide special atomic instructions to make the critical-section problem tractable at a low level.

- **Uniprocessors** could simply disable interrupts, but (as discussed above) this doesn't scale to multiprocessor systems.
- Instead, most modern hardware offers:
    1. **Hardware instructions** (test-and-set, compare-and-swap)
    2. **Atomic variables**

### 5.1 `test_and_set()` Instruction

```
boolean test_and_set(boolean *target) {
    boolean rv = *target;
    *target = true;
    return rv;
}
```

- **Properties:** executes **atomically** (as an indivisible unit); returns the _original_ value; always sets the target to `true`.

**Solution using `test_and_set()`:**

```
// shared boolean lock, initialized to false
do {
    while (test_and_set(&lock)) ; /* do nothing — busy wait */
    /* critical section */
    lock = false;
    /* remainder section */
} while (true);
```

- This guarantees **mutual exclusion** and **progress**, but on its own it does **not** guarantee **bounded waiting** — a process could be repeatedly overtaken by others.

### 5.2 `compare_and_swap()` (CAS) Instruction

```
int compare_and_swap(int *value, int expected, int new_value) {
    int temp = *value;
    if (*value == expected)
        *value = new_value;
    return temp;
}
```

- **Properties:** executes atomically; returns the original value of `*value`; only performs the swap to `new_value` if the current value equals `expected`.

**Basic solution using CAS:**

```
// shared int lock, initialized to 0
while (true) {
    while (compare_and_swap(&lock, 0, 1) != 0) ; /* do nothing */
    /* critical section */
    lock = 0;
    /* remainder section */
}
```

- Like the `test_and_set` version, this satisfies mutual exclusion and progress but **not necessarily bounded waiting**.

### 5.3 Bounded-Waiting Solution Using CAS

To fix the fairness gap, we add a `waiting[]` array so processes take turns fairly:

```
while (true) {
    waiting[i] = true;
    key = 1;
    while (waiting[i] && key == 1)
        key = compare_and_swap(&lock, 0, 1);
    waiting[i] = false;
    /* critical section */
    j = (i + 1) % n;
    while ((j != i) && !waiting[j])
        j = (j + 1) % n;
    if (j == i)
        lock = 0;
    else
        waiting[j] = false;
    /* remainder section */
}
```

- **How it works:** when a process exits its critical section, instead of blindly releasing the lock, it scans the `waiting[]` array (starting right after its own index) to find the _next_ process that is waiting, and passes control directly to that process. This guarantees a process waits **at most n − 1 turns** before it is admitted, satisfying bounded waiting.

### 5.4 Atomic Variables

- Hardware instructions like CAS are often used as low-level **building blocks** for higher-level synchronization tools.
- An **atomic variable** provides uninterruptible updates on basic types (integers, booleans) without the programmer having to hand-write CAS loops each time.

**Example — an atomic `increment()`:**

```
void increment(atomic_int *v) {
    int temp;
    do {
        temp = *v;
    } while (temp != compare_and_swap(v, temp, temp + 1));
}
```

- This repeatedly attempts the CAS until it succeeds (i.e., until no other thread changed `*v` between the read and the swap), guaranteeing the increment is effectively atomic even though it's built from multiple lower-level steps.

---

## 6. Mutex Locks

- The hardware-level solutions above are **too low-level** for application programmers to use directly and error-prone to reimplement everywhere.
- OS/language designers therefore package this logic into a reusable software tool: the **mutex lock** ("mutual exclusion lock").
- A mutex lock is essentially a boolean flag indicating whether the lock is currently **available** or **held**.
- **Usage pattern:**
    - `acquire()` the lock before the critical section.
    - `release()` the lock after the critical section.
- `acquire()` and `release()` must themselves be **atomic operations** — typically implemented using hardware instructions such as compare-and-swap.

```
while (true) {
    acquire lock
    critical section
    release lock
    remainder section
}
```

- **Drawback:** the straightforward implementation requires **busy waiting** — a process repeatedly checks the lock in a tight loop rather than sleeping. A lock implemented this way is called a **spinlock**.
    - Spinlocks _do_ have a legitimate use case: on multiprocessor systems, if a critical section is expected to be held only briefly, it can be cheaper to "spin" on one CPU than to pay the overhead of putting a process to sleep and waking it up again later.

---

## 7. Semaphores

- A more flexible/sophisticated synchronization tool than a simple mutex lock.
- **Semaphore `S`** is an integer variable that can only be accessed through two **atomic (indivisible)** operations: `wait()` and `signal()` (historically named **`P()`** and **`V()`**, from the Dutch words _proberen_ "to test" and _verhogen_ "to increment").

```
wait(S) {
    while (S <= 0) ;   // busy wait
    S--;
}

signal(S) {
    S++;
}
```

### Counting vs. Binary Semaphores

- **Counting semaphore:** the integer value may range over any domain — useful for controlling access to a resource that has multiple identical instances (e.g., a pool of _k_ printers: initialize `S = k`).
- **Binary semaphore:** the value is restricted to 0 or 1 — functionally equivalent to a **mutex lock**.
- A counting semaphore can, in fact, be implemented using binary semaphores.
- Semaphores are versatile enough to solve a wide range of synchronization problems beyond simple mutual exclusion.

### Usage Examples

**(a) Mutual exclusion (like a mutex):**

```
// semaphore "mutex", initialized to 1
wait(mutex);
   critical section
signal(mutex);
```

**(b) Enforcing execution order between two processes** Suppose statement S₁ in process P1 must always execute _before_ statement S₂ in process P2:

```
// semaphore "synch", initialized to 0
P1:                      P2:
S1;                       wait(synch);
signal(synch);       S2;
```

- Since `synch` starts at 0, P2's `wait(synch)` will block until P1 calls `signal(synch)` after finishing S1 — guaranteeing the required ordering.

### Semaphore Implementation

- The internal implementation of `wait()`/`signal()` must itself guarantee that no two processes can execute either operation on the _same_ semaphore at exactly the same time.
- This means the implementation reduces to **another instance of the critical-section problem**, applied to just the (short) bodies of `wait()` and `signal()`.
- Because these bodies are very short, some designs accept limited busy waiting here — acceptable if critical sections are rarely occupied, but a poor choice if applications spend a lot of time inside critical sections.

### Semaphore Implementation Without Busy Waiting

Instead of spinning, a blocked process can be put to sleep and placed on a **waiting queue** associated with the semaphore.

**Data structure:**

```
typedef struct {
    int value;
    struct process *list;   // waiting queue
} semaphore;
```

**Two supporting OS operations:**

- **`block()`** — suspends the calling process and places it on the appropriate waiting queue.
- **`wakeup(P)`** — removes process P from a waiting queue and moves it to the **ready queue** so it can be scheduled again.

**`wait()` and `signal()` without busy waiting:**

```
wait(semaphore *S) {
    S->value--;
    if (S->value < 0) {
        add this process to S->list;
        block();
    }
}

signal(semaphore *S) {
    S->value++;
    if (S->value <= 0) {
        remove a process P from S->list;
        wakeup(P);
    }
}
```

- Here, `value` can go negative — a negative value's magnitude tells you how many processes are currently waiting on the semaphore.

### Problems with Semaphores (Common Misuse)

Because correctness depends entirely on programmer discipline, several classic bugs are easy to introduce:

- Calling `signal(mutex)` _before_ `wait(mutex)` — no protection is actually in place.
- Calling `wait(mutex)` twice in a row without an intervening `signal` — the second call blocks forever (self-deadlock).
- **Omitting** either `wait(mutex)` or `signal(mutex)` entirely — either no mutual exclusion, or a lock that is never released.

These pitfalls are a major motivation for moving to a **higher-level, structured** synchronization construct: the **monitor**.

---

## 8. Monitors

- A **high-level abstraction** that packages both the shared data and the operations on it, offering a convenient and less error-prone alternative to raw semaphores.
- A monitor is essentially an **abstract data type (ADT)**: its internal (shared) variables can only be accessed through the procedures defined inside the monitor — never directly from outside.
- **Key guarantee:** only **one process may be active inside the monitor at a time** — mutual exclusion is enforced automatically by the construct itself, rather than by the programmer remembering to call `wait`/`signal` correctly.

**Pseudocode syntax:**

```
monitor monitor-name {
    // shared variable declarations

    procedure P1 (...) { .... }
    procedure P2 (...) { .... }
    procedure Pn (...) { .... }

    initialization code (...) { ... }
}
```

**Schematic view:** Processes wishing to invoke any monitor procedure must first queue up on a single **entry queue**; only one process is admitted inside the monitor's boundary at a time.

### Condition Variables

- Simply enforcing mutual exclusion within the monitor is not always enough — sometimes a process needs to _wait_ for some specific condition to become true (e.g., "buffer is not full") before it can proceed, without holding up every other process trying to enter the monitor.
- **Condition variables** solve this. Declared as `condition x, y;` and support exactly two operations:
    - **`x.wait()`** — suspends the calling process until some other process calls `x.signal()`.
    - **`x.signal()`** — resumes (wakes up) one process that is suspended on `x.wait()`. If no process is currently waiting on `x`, the signal has **no effect** (it is _not_ remembered/queued for later — this differs from a semaphore's `signal`).
- Each condition variable has its **own queue** of waiting processes, separate from the monitor's main entry queue (see the schematic with queues for `x` and `y`).

### Usage of Condition Variables — Example

To force statement S₁ (in P1) to complete before statement S₂ (in P2), using a monitor with two procedures F1/F2, a condition variable `x`, and a boolean flag `done`:

```
F1:                                F2:
S1;                                 if (done == false)
done = true;                          x.wait();
x.signal();                        S2;
```

- If F2 runs first and `done` is still false, it calls `x.wait()` and sleeps until F1 finishes S1 and signals.
- If F1 has already run by the time F2 checks, `done` is already true and F2 proceeds straight to S2 without waiting.

### Monitor Implementation Using Semaphores

Monitors are commonly implemented "underneath" using semaphores:

**Variables:**

```
semaphore mutex;        // initially = 1  (enforces mutual exclusion on entry)
semaphore next;          // initially = 0  (used to hand control to a waiting signaler)
int next_count = 0;     // number of processes waiting on 'next'
```

**Every monitor procedure P is wrapped as:**

```
wait(mutex);
   ... body of P ...
if (next_count > 0)
    signal(next);
else
    signal(mutex);
```

- This wrapper ensures that a process calling `signal()` inside the monitor immediately hands control to a process it just woke up (via `next`), rather than letting a third, unrelated process barge in — preserving the intended semantics of condition variables.

**Implementing a condition variable `x` on top of semaphores:**

```
semaphore x_sem;       // initially = 0
int x_count = 0;

// x.wait():
x_count++;
if (next_count > 0)
    signal(next);
else
    signal(mutex);
wait(x_sem);
x_count--;

// x.signal():
if (x_count > 0) {
    next_count++;
    signal(x_sem);
    wait(next);
    next_count--;
}
```

### Resuming Processes Within a Monitor — Priority

- If several processes are queued on the same condition variable `x` and `x.signal()` is called, which one should wake up?
- Plain **FCFS (first-come-first-served)** is often not adequate for real applications.
- A more flexible construct is the **conditional-wait**: `x.wait(c)`, where `c` is an integer **priority number**.
    - The waiting process with the **lowest** priority number (i.e., **highest** priority) is the one scheduled next when a signal occurs.

### Worked Example — Single Resource Allocation

**Goal:** allocate one shared resource among competing processes, letting each process declare (via a priority number) the maximum time it plans to use the resource; the process with the _shortest_ planned usage time gets priority.

**Usage pattern:**

```
R.acquire(t);
   ... access the resource ...
R.release;
```

Here `t` is the maximum time the calling process plans to hold the resource, and `R` is an instance of a monitor type called `ResourceAllocator`.

**The monitor itself:**

```
monitor ResourceAllocator {
    boolean busy;
    condition x;

    void acquire(int time) {
        if (busy)
            x.wait(time);     // wait, with priority = requested time
        busy = true;
    }

    void release() {
        busy = false;
        x.signal();            // wake the waiting process with the lowest 'time'
    }

    initialization code() {
        busy = false;
    }
}
```

- Because `x.wait(time)` uses the requested duration as its priority number, when the resource is released, the monitor automatically wakes the waiting process that asked for the **shortest** usage time first — a simple built-in scheduling policy.

**Same class of misuse risk as semaphores applies here too:**

- Calling `release()` before `acquire()`
- Calling `acquire()` twice without an intervening `release()`
- Omitting `acquire()` and/or `release()` entirely

Even monitors, which are much safer than raw semaphores, still rely on the programmer using them correctly and in the right order.

---

## 9. Liveness

- **Liveness** refers to a set of properties a system must satisfy to guarantee that processes actually make forward progress — not just that data stays consistent (safety), but that nothing gets stuck forever.
- Processes waiting on a synchronization tool (mutex, semaphore, etc.) may, if the tool or its usage is flawed, end up **waiting indefinitely** — violating both the _progress_ and _bounded-waiting_ requirements from Section 2. This category of bug is called a **liveness failure**.

### Deadlock

- **Definition:** two or more processes are waiting indefinitely for an event that can only be triggered by one of those same waiting processes — so nobody can ever make the triggering event happen.

**Illustrative example** — two semaphores `S` and `Q`, both initialized to 1:

```
P0                         P1
wait(S);                    wait(Q);
wait(Q);                    wait(S);
...                            ...
signal(S);                  signal(Q);
signal(Q);                  signal(S);
```

- Suppose P0 executes `wait(S)` and P1 executes `wait(Q)` around the same time.
- P0 then tries `wait(Q)` — but Q is held by P1, so P0 blocks, waiting for P1 to call `signal(Q)`.
- Meanwhile, P1 tries `wait(S)` — but S is held by P0, so P1 blocks, waiting for P0 to call `signal(S)`.
- Neither `signal()` call can ever be reached, because each process is stuck waiting on the _other_. This is a **deadlock**.

### Starvation (Indefinite Blocking)

- A process may remain in a semaphore's waiting queue and simply **never** get removed (never chosen for `wakeup()`), even though the semaphore is being signaled repeatedly for _other_ processes.
- Unlike deadlock, the system as a whole keeps making progress — just not for this particular unlucky process.

### Priority Inversion

- A scheduling anomaly that occurs when a **lower-priority** process holds a lock that a **higher-priority** process needs, effectively forcing the higher-priority process to wait on the lower-priority one — inverting the intended priority order.
- **Solution: priority-inheritance protocol** — while the lower-priority process holds the needed resource, it temporarily "inherits" the higher priority of the process waiting on it, so it gets scheduled sooner, finishes with the resource, and releases it faster.

---

## 10. Evaluation

The chapter's stated objective is to compare and evaluate the synchronization tools covered above (mutex locks, semaphores, monitors, hardware instructions) under **low-, moderate-, and high-contention** scenarios — i.e., how well each tool performs depending on how frequently processes actually compete for the same critical section. This slide deck's outline reserves a dedicated "Evaluation" section for that comparison; the general takeaways to keep in mind while studying it are:

- **Low contention:** simple spinlocks/busy-waiting approaches can be efficient, since waits are typically short.
- **High contention:** busy waiting wastes significant CPU time; blocking-based tools (semaphores/monitors with `block()`/`wakeup()`) tend to perform better because waiting processes give up the CPU instead of spinning.
- Higher-level constructs (monitors) trade a small amount of raw performance for much greater programmer safety and reduced risk of subtle synchronization bugs.

---

## Key Takeaways / Study Summary

|Tool|Level|Busy-Waiting?|Key Idea|
|---|---|---|---|
|Disabling interrupts|Hardware/OS|N/A|Works only on uniprocessors; doesn't scale|
|Peterson's Solution|Software (2 processes)|Yes|Correct in theory; broken by modern instruction reordering unless memory barriers are used|
|`test_and_set` / `compare_and_swap`|Hardware instruction|Yes (unless paired with `waiting[]` array)|Atomic read-modify-write primitives; building blocks for higher-level tools|
|Atomic variables|Hardware-backed software|Depends on implementation|Uninterruptible updates to simple types, often built on CAS|
|Mutex Lock|Software (OS/library)|Yes (spinlock)|Simple `acquire()`/`release()` pair|
|Semaphore|Software (OS/library)|Optional (can block instead)|Integer variable + atomic `wait()`/`signal()`; counting or binary|
|Monitor|Language-level construct|No (uses condition variables)|Encapsulates shared data + procedures; automatic mutual exclusion; `wait()`/`signal()` on condition variables for finer coordination|

**Core dangers to watch for:** race conditions (Section 1), failing any of the three critical-section requirements — mutual exclusion, progress, bounded waiting (Section 2) — and liveness failures such as deadlock, starvation, and priority inversion (Section 9).


# The Mutual Exclusion Problem — Lecture Notes

### (Part 1 — Deriving Dekker's Algorithm)

_Based on slides adapted from Professor N.S. Scott (Queen's University, Belfast)_

---

## 1. Introduction: What Is the Mutual Exclusion Problem?

- Eliminating undesirable **interleavings** of concurrent instructions is called the **mutual exclusion problem**.
- We must identify **critical sections** — code that only one thread may execute at a time.
- We need a **pre-protocol** (run before entering the critical section) and a **post-protocol** (run after leaving it) that together prevent two or more threads from being inside their critical sections simultaneously.

**General structure of a process:**

```
while (true) {
    nonCriticalSection;
    preProtocol;
    criticalSection;
    postProtocol;
}
```

_(This maps directly onto the "entry section / critical section / exit section / remainder section" structure from Chapter 6 — `preProtocol` = entry section, `postProtocol` = exit section.)_

---

## 2. Formal Requirements for N Processes

_N_ processes run in an infinite loop, each executing a sequence split into a **critical section** and a **non-critical section**. A correct solution inserts a pre-protocol and post-protocol around the critical section (possibly using extra shared variables) and must guarantee:

1. **Mutual exclusion** — instructions from the critical sections of two or more processes must never be interleaved.
2. **No halting problems** — a process _may_ halt while in its non-critical section (and must not interfere with others if it does), but it must **never** halt while inside its protocols or its critical section.
3. **No deadlock** — if some processes are trying to enter their critical sections, at least one of them must eventually succeed. (Deadlock = no process ever transitions from pre-protocol to critical section.)
4. **No starvation** — if a process begins its pre-protocol (signaling intent to enter), it must **eventually** succeed in entering.
5. **Minimal overhead absent contention** — if only one process wants to enter and nobody else is competing, it should succeed with little overhead.

_(These map closely onto Chapter 6's three requirements — mutual exclusion, progress, and bounded waiting — but are broken out here into more granular test cases: deadlock-freedom and starvation-freedom, tested separately with and without contention.)_

---

## 3. Setting Up the Two-Process Case

- The two-process version of the problem is solved here using only **`Load`** and **`Store`** to shared memory as the sole atomic instructions (the same assumption used for Peterson's Solution in Chapter 6).
- The famous two-process solution developed here is **Dekker's Algorithm** — historically one of the very first correct mutual-exclusion algorithms.
- The slides build up to it through a sequence of **deliberately flawed** attempts, each illustrating a different pathological failure mode common in concurrent algorithms.

---

## 4. First Attempt — The `turn` Variable

```
int turn = 1;

process P1                          process P2
while (true) {                       while (true) {
  nonCriticalSection1;                 nonCriticalSection2;
  while (turn == 2) {}                 while (turn == 1) {}
  criticalSection1;                    criticalSection2;
  turn = 2;                             turn = 1;
}                                       }
end P1;                                end P2;
```

- A single shared variable `turn` dictates whose turn it is to enter the critical section.
- **This is the same idea as "Software Solution 1" from Chapter 6.**

**Properties evaluated:** mutual exclusion, no deadlock, no starvation, no starvation in the absence of contention.

|Property|Result|
|---|---|
|Mutual exclusion|**Satisfied.** _Proof sketch:_ if P1 entered at t1 and P2 at t2 (t1 < t2), `turn` must equal 1 at t1 and 2 at t2 — but P1 never leaves its critical section (and so never sets `turn = 2`) during that interval, so `turn` cannot have changed. Contradiction.|
|No deadlock|**Satisfied.** For a deadlock, both processes would need to be stuck forever failing their `while` tests — but that requires `turn == 2` in P1 _and_ `turn == 1` in P2 at once, which is impossible for a single variable.|
|No starvation|**Satisfied** (given both processes keep trying). If P1 ever executes its critical section, it will set `turn = 2`, freeing P2 to enter.|
|No starvation in the **absence** of contention|**Violated.** If P2 halts in its non-critical section, `turn` is never reset back to 1. P1 can enter its critical section at most one more time — after that, `turn` stays at 2 forever and P1 can never re-enter, even though P2 isn't competing at all.|

**Takeaway:** even though this solution satisfies mutual exclusion, no-deadlock, and no-starvation-under-contention, it fails the "minimal overhead / no starvation absent contention" requirement — forcing strict alternation is too restrictive. This motivates a better approach.

---

## 5. Second Attempt — Per-Process Flags (flag set _inside_ the critical section)

```
int c1 = 1;
int c2 = 1;

process P1                          process P2
while (true) {                       while (true) {
  nonCriticalSection1;                 nonCriticalSection2;
  while (c2 != 1) {}                    while (c1 != 1) {}
  c1 = 0;                                c2 = 0;
  criticalSection1;                     criticalSection2;
  c1 = 1;                                c2 = 1;
}                                       }
end P1;                                end P2;
```

- Each process now has its _own_ shared variable `Ci`. `Ci == 0` signals "process i is about to enter its critical section."

**Properties evaluated:** mutual exclusion.

|Property|Result|
|---|---|
|Mutual exclusion|**Violated.** _Counter-example interleaving:_ (1) P1 checks `c2`, finds it 1. (2) P2 checks `c1`, finds it 1. (3) P1 sets `c1 = 0`. (4) P2 sets `c2 = 0`. (5) P1 enters its critical section. (6) P2 _also_ enters its critical section — both are inside at once.|

**Why it fails:** each process checks the _other's_ flag before setting its _own_ — there's a gap between the check and the flag-set during which the other process can slip through the same check.

---

## 6. Third Attempt — Flag Set _Before_ the Wait Loop

```
int c1 = 1;
int c2 = 1;

process P1                          process P2
while (true) {                       while (true) {
  nonCriticalSection1;                 nonCriticalSection2;
  c1 = 0;                                c2 = 0;
  while (c2 != 1) {}                    while (c1 != 1) {}
  criticalSection1;                     criticalSection2;
  c1 = 1;                                c2 = 1;
}                                       }
end P1;                                end P2;
```

- Fix attempt: move `Ci = 0` to the very **start** of the pre-protocol (instead of after the wait loop). Now `Ci == 0` signals "I _wish_ to enter," not "I am entering."

**Properties evaluated:** mutual exclusion, no deadlock.

|Property|Result|
|---|---|
|Mutual exclusion|**Satisfied.** Similar proof to the First Attempt: if P1 entered at t1 and P2 at t2 (t1<t2), then at t1 `c1==0, c2==1`, and at t2 `c2==0, c1==1` — but P1 never left its critical section in between to reset `c1` back to 1. Contradiction.|
|No deadlock|**Violated.** _Counter-example:_ (1) P1 sets `c1 = 0`. (2) P2 sets `c2 = 0`. (3) P1 tests `c2`, sees 0, loops forever. (4) P2 tests `c1`, sees 0, loops forever. **Both processes are now locked in their pre-protocols permanently** — classic deadlock.|

**Takeaway:** fixing mutual exclusion introduced a new failure mode — deadlock — because both processes can simultaneously announce intent and then wait on each other forever.

---

## 7. Fourth Attempt — Backing Off

```
int c1 = 1;
int c2 = 1;

process P1                              process P2
while (true) {                           while (true) {
  nonCriticalSection1;                     nonCriticalSection2;
  c1 = 0;                                    c2 = 0;
  while (c2 != 1) {                         while (c1 != 1) {
    c1 = 1; c1 = 0;                            c2 = 1; c2 = 0;
  }                                           }
  criticalSection1;                         criticalSection2;
  c1 = 1;                                    c2 = 1;
}                                           }
end P1;                                    end P2;
```

- Fix attempt: if a process detects that both are trying to enter at once, it **backs off** — briefly resets its own flag to 1 (then immediately back to 0) inside the wait loop, giving the other process a chance to proceed.

**Properties evaluated:** mutual exclusion, no livelock, no starvation.

**Key definition given on this slide:**

> **Livelock is described as a form of deadlock.** In a _deadlocked_ computation, there is _no_ possible execution sequence that succeeds. In a _livelocked_ computation, successful executions **do** exist, but there also exist one or more execution sequences in which no process ever enters its critical section (processes stay "active" — toggling flags — without making real progress).
> 
> _(Note: many other textbooks, including Silberschatz's_ Operating System Concepts*, treat livelock as a **distinct** liveness failure from deadlock — active-but-non-progressing vs. blocked-and-non-progressing — rather than a "form of" deadlock. This slide deck's own framing differs from that more common distinction, so be aware both definitions circulate and check which one your instructor/exam expects.)*

|Property|Result|
|---|---|
|Mutual exclusion|**Satisfied.** Same argument as the Third Attempt.|
|No starvation|**Violated.** _Counter-example:_ (1) P1 sets `c1=0`. (2) P2 sets `c2=0`. (3) P2 checks `c1`, backs off, resets `c2` to 1. (4) P1 completes an entire cycle — checks `c2`, enters its critical section, resets `c1`, goes to its non-critical section, and re-sets `c1=0` again. (5) P2 sets `c2=0` again... This pattern can repeat: **P1 enters its critical section infinitely often while P2 is starved indefinitely** in its pre-protocol.|
|No livelock|**Violated.** _Counter-example:_ both processes repeatedly set their flags to 0, check the other's flag, find it 0, back off (reset to 1 then immediately back to 0), and repeat — forever, in lockstep, with **neither ever entering** its critical section. (Note: the slide adds that "the slightest deviation" from this exact interleaving lets one process through — which is the hallmark of livelock rather than deadlock: recovery is _possible_ but not _guaranteed_.)|

**Takeaway:** the Fourth Attempt guarantees mutual exclusion but is still not a fully correct solution — it can starve one process and can livelock both. A genuinely correct algorithm is still needed.

---

## 8. A Correct Solution: Dekker's Algorithm

- Dekker's Algorithm avoids the strict, forced alternation of the naive turn-taking approach (First Attempt) while still guaranteeing correctness.
- It is one of the **first mutual-exclusion algorithms ever devised**.
- **Core idea:** if both processes try to enter the critical section at the same moment, the algorithm defers to whichever process's `turn` it currently is. If the other process is already inside modifying the critical section, the requester waits for it to finish.
- It combines **both** mechanisms seen so far: per-process intent flags (`c1`, `c2`, as in the Third/Fourth Attempts) **and** a shared `turn` variable (as in the First Attempt) that resolves conflicts when both flags are set simultaneously.

```
int c1 = 1;
int c2 = 1;
int turn = 1;

process P1                                    process P2
while (true) {                                 while (true) {
  nonCriticalSection1;                           nonCriticalSection2;
  c1 = 0;                                          c2 = 0;
  while (c2 != 1)                                 while (c1 != 1)
    if (turn == 2) {                                if (turn == 1) {
      c1 = 1;                                          c2 = 1;
      while (turn != 1) {}                             while (turn != 2) {}
      c1 = 0;                                          c2 = 0;
    }                                                }
  criticalSection1;                               criticalSection2;
  c1 = 1; turn = 2;                                c2 = 1; turn = 1;
}                                                 }
end P1;                                          end P2;
```

**How it resolves the earlier failure modes:**

- Each process announces intent by setting its own flag to 0 **before** checking the other's flag (fixes the Second Attempt's mutual-exclusion bug).
- If both processes want in at the same time, `turn` breaks the tie: only the process whose `turn` it is proceeds to wait actively; the other process **temporarily backs off** (sets its own flag back to 1) and waits until `turn` changes in its favor — instead of both looping forever (fixes the Third Attempt's deadlock).
- Because the process that backs off waits specifically for `turn` to flip — rather than repeatedly toggling its flag in lockstep with the other process — the two processes take **turns backing off** in a way that is guaranteed to eventually let each one through, avoiding both the starvation and livelock problems seen in the Fourth Attempt.

---

## Connection Back to Chapter 6 (Synchronization Tools)

This deck develops, with much more rigorous step-by-step correctness/failure proofs, the same territory that Chapter 6 covered more briefly:

|This deck|Chapter 6 equivalent|
|---|---|
|First Attempt (single `turn` variable, strict alternation)|"Software Solution 1" — same idea, same weakness (fails progress/no-starvation-absent-contention)|
|Second/Third/Fourth Attempts (per-process flags `c1`, `c2`)|Peterson's Solution's `flag[]` array plays a similar role, but Peterson's Solution combines the flag array _and_ the turn variable together from the start, avoiding the deadlock/starvation/livelock pitfalls seen in these intermediate attempts|
|Dekker's Algorithm (flags **+** `turn`, with explicit back-off)|Functionally analogous to Peterson's Solution — both are historically important, _provably correct_, two-process mutual-exclusion algorithms built only from atomic Load/Store, though Dekker's is structurally more complex (nested `if`/`while` back-off logic) while Peterson's is more compact|
|Both algorithms|As Chapter 6 notes, algorithms of this style (Peterson's, Dekker's) are **not guaranteed to work on modern hardware** without memory barriers, due to instruction/memory reordering — the same caveat applies here|

**No other prior "chapter" is explicitly referenced in this deck** — its own internal narrative is self-contained (First → Fourth Attempt → Dekker's Algorithm), so the summary above is provided to bridge it with the Chapter 6 notes you already have, rather than summarizing a chapter this deck itself cited by name.

---

## Quick Reference Table — All Four Attempts + Final Solution

|Attempt|Mutual Exclusion|No Deadlock|No Starvation|No Livelock|Verdict|
|---|---|---|---|---|---|
|1st (turn only)|✅|✅|✅ (under contention)|—|❌ Starves in absence of contention|
|2nd (flag set inside CS)|❌|—|—|—|❌ Fails mutual exclusion|
|3rd (flag set before wait)|✅|❌|—|—|❌ Can deadlock|
|4th (flag + back-off)|✅|✅ (no permanent block)|❌|❌|❌ Can starve / livelock|
|**Dekker's Algorithm** (flag + turn + back-off)|✅|✅|✅|✅|✅ **Correct**|