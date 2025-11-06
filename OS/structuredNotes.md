# Operating Systems - Quick Interview Prep Guide

## 📋 Table of Contents
1. [Memory Management](#memory-management)
2. [Process & Thread Concepts](#process--thread-concepts)
3. [CPU Architecture](#cpu-architecture)
4. [Virtualization](#virtualization)
5. [System Monitoring](#system-monitoring)
6. [File System Concepts](#file-system-concepts)

---

## 1️⃣ MEMORY MANAGEMENT

### 🔹 What is Paging?
**Quick Definition:** Memory management technique that divides memory into fixed-size blocks.

**Detailed Explanation:**
Paging divides both **virtual address space** (process memory) and **physical memory** (RAM) into fixed-size blocks:
- **Pages**: Virtual memory blocks (typically 4 KB)
- **Frames**: Physical memory blocks (same size as pages)

**How it works:**
```
Process Virtual Memory → Page Table → Physical RAM
    [Pages]          [Mapping]      [Frames]
```

**Key Components:**
- **Page Table**: Data structure maintained by OS that maps virtual pages to physical frames
- **Memory Management Unit (MMU)**: Hardware component that translates virtual addresses to physical addresses
- **Page Fault**: Occurs when a process tries to access a page not currently in RAM; OS must load it from disk

**Interview Points:**
✅ Allows non-contiguous memory allocation (processes don't need continuous RAM blocks)
✅ Enables efficient memory sharing between processes
✅ Implements protection (each process has isolated address space)
✅ Foundation for virtual memory implementation

---

### 🔹 What is Virtual Memory?
**Quick Definition:** Technique that gives each process the illusion of having large, continuous memory, even when physical RAM is limited.

**Detailed Explanation:**
Virtual memory combines **RAM** + **disk space (swap)** to create a larger addressable memory space.

**Key Concepts:**
- **Virtual Address Space**: The memory range a process "thinks" it has (e.g., 0 to 4 GB)
- **Swap Space**: Portion of disk used as overflow when RAM is full
- **Page Fault**: Interrupt triggered when accessing data not in RAM; OS loads it from swap

**Real-World Example:**
```
System has:        8 GB RAM + 8 GB Swap = 16 GB Virtual Memory
Process needs:     10 GB
Solution:          
  → Active pages stay in RAM (fast access)
  → Inactive pages move to swap (slower, but available)
  → OS swaps pages in/out as needed
```

**Benefits:**
✅ Run programs larger than physical RAM
✅ Better multitasking (more processes can coexist)
✅ Process isolation and security
✅ Memory abstraction for developers

**Trade-off:** Swap is 100-1000× slower than RAM, so excessive swapping ("thrashing") degrades performance.

---

### 🔹 Memory Hierarchy
**Quick Definition:** Organization of storage from fastest/smallest to slowest/largest.

**The Hierarchy:**
```
Speed ↑ | Cost ↑ | Size ↓

1. Registers          (CPU internal, nanoseconds, ~few KB)
2. L1 Cache           (per-core, ~nanoseconds, 32-64 KB)
3. L2 Cache           (per-core, ~ns, 256 KB-1 MB)
4. L3 Cache           (shared across cores, ~ns, 8-32 MB)
5. RAM                (main memory, ~100 ns, 8-64 GB)
6. SSD/HDD            (secondary storage, ms, 256 GB-TBs)
7. Network/Cloud      (tertiary, seconds, unlimited)

Speed ↓ | Cost ↓ | Size ↑
```

**Key Principle:** 
**Locality of Reference**
- **Temporal Locality**: Recently accessed data will likely be accessed again soon
- **Spatial Locality**: Data near recently accessed addresses will likely be accessed next

**Interview Insight:** The hierarchy balances speed, cost, and capacity. Caching at each level reduces average access time dramatically.

---

### 🔹 What is Cache?
**Quick Definition:** Small, ultra-fast memory in CPU that stores frequently accessed data to reduce RAM access time.

**Cache Levels:**
- **L1 Cache**: Smallest (32-64 KB), fastest, per-core, split into instruction & data caches
- **L2 Cache**: Larger (256 KB-1 MB), per-core, unified cache
- **L3 Cache**: Largest (8-32 MB), shared across all cores, slowest of the three but still faster than RAM

**How Cache Works:**
1. CPU requests data
2. Check L1 → if found (**cache hit**), use it
3. If not → Check L2 → if found, promote to L1
4. If not → Check L3 → if found, promote upward
5. If not (**cache miss**) → Fetch from RAM (expensive!)

**Key Terms:**
- **Cache Hit**: Data found in cache (fast)
- **Cache Miss**: Data not in cache, must fetch from lower level (slow)
- **Cache Line**: Unit of data transfer (typically 64 bytes)
- **Cache Invalidation**: When data in cache becomes stale and must be refreshed

**Why It Matters:** RAM access is ~100× slower than L1 cache. Good cache utilization = better performance.

---

## 2️⃣ PROCESS & THREAD CONCEPTS

### 🔹 What is a Process?
**Quick Definition:** An independent program in execution with its own isolated memory and resources.

**Key Components:**
```
Process Structure:
├── Code Segment        (program instructions)
├── Data Segment        (global variables)
├── Heap                (dynamic memory)
├── Stack               (function calls, local variables)
├── CPU Registers       (PC, SP, etc.)
├── Page Table          (virtual memory mapping)
└── File Descriptors    (open files, sockets)
```

**Characteristics:**
✅ **Isolation**: Cannot directly access another process's memory
✅ **Heavy**: Creating/switching processes is expensive
✅ **Communication**: Via IPC (pipes, sockets, shared memory, message queues)
✅ **Example**: Each Chrome tab is a separate process

**Interview Point:** Processes provide security and stability through isolation—if one crashes, others survive.

---

### 🔹 What is a Thread?
**Quick Definition:** Smallest unit of execution within a process; shares memory with other threads in the same process.

**Shared vs. Private Resources:**
```
SHARED (among threads in same process):
├── Code segment
├── Data segment (global variables)
├── Heap memory
└── Open file descriptors

PRIVATE (per thread):
├── Stack (local variables, function calls)
├── Program Counter (current instruction)
└── CPU Registers (thread state)
```

**Advantages:**
✅ **Lightweight**: Faster to create and switch than processes
✅ **Efficient**: Shared memory = no IPC overhead
✅ **Parallelism**: Multiple threads can run on multiple cores simultaneously

**Example - Web Browser:**
```
Chrome Process
  ├── UI Thread          (renders interface)
  ├── Network Thread     (downloads resources)
  ├── JavaScript Thread  (executes scripts)
  └── Rendering Thread   (paints pixels)
```
If JavaScript hangs, UI thread keeps responding.

---

### 🔹 What is Multithreading?
**Quick Definition:** Ability of a process to execute multiple threads concurrently, sharing the same memory space.

**Single-threaded vs. Multithreaded:**
```
Single-threaded:
Task A → Task B → Task C  (sequential)

Multithreaded:
Task A ↘
Task B  → All run concurrently
Task C ↗
```

**Benefits:**
✅ **Responsiveness**: UI remains active while background tasks run
✅ **Resource Sharing**: No need for expensive IPC
✅ **Scalability**: Utilize multiple CPU cores
✅ **Economy**: Cheaper than creating multiple processes

**Real-World Use Cases:**
- Web servers handling multiple requests
- Video games (rendering, physics, AI, audio)
- Database systems (query processing)

**Interview Insight:** Multithreading improves throughput and responsiveness but introduces complexity (race conditions, deadlocks).

---

### 🔹 Process States (Lifecycle)
**Quick Definition:** States a process transitions through during its lifetime.

**State Diagram:**
```
     New
      ↓
    Ready ←―――――→ Running ←→ Waiting
      ↑             ↓
      └―――――― Terminated
```

**State Descriptions:**
1. **New**: Process is being created
2. **Ready**: Waiting for CPU assignment (in ready queue)
3. **Running**: Executing on CPU
4. **Waiting/Blocked**: Waiting for I/O or event (e.g., disk read, user input)
5. **Terminated**: Finished execution, cleanup pending

**Transitions:**
- Ready → Running: **Scheduler dispatches** process to CPU
- Running → Ready: **Time slice expires** (preemption)
- Running → Waiting: **I/O request** (voluntary)
- Waiting → Ready: **I/O completes** (event occurs)

---

### 🔹 What is Context Switching? ⭐⭐⭐⭐
**Quick Definition:** When CPU stops executing one process/thread and starts another by saving and restoring execution context.

**What Gets Saved/Restored:**
```
Context Includes:
├── CPU Registers (Program Counter, Stack Pointer, etc.)
├── Page Table Pointer (memory mapping)
├── Process State (running, ready, etc.)
└── File Descriptors (open files)
```

**Time Cost:** 1-10 microseconds per switch

**Why It's Expensive:**
Direct cost (small): Saving/loading registers  
**Indirect cost (BIG)**: **Cache Invalidation** 🔥

**Cache Invalidation Problem:**
```
Running Process A:
  CPU Cache filled with A's data (hot cache)
     ↓
Context Switch to Process B
     ↓
  CPU Cache now contains A's data (useless for B)
  B's data must be fetched from RAM (cold cache)
     ↓
  100× slower access! This is the real penalty.
```

**Detection:**
Check context switches per second:
```bash
vmstat 1
# High "cs" column = excessive switching
```

**Interview Point:** Context switching overhead comes primarily from **cache misses**, not the register save/restore itself.

---

### 🔹 What is a Zombie Process?
**Quick Definition:** A process that has finished execution but still has an entry in the process table because its parent hasn't read its exit status.

**How It Happens:**
```
1. Child process finishes execution
2. Child calls exit() and enters zombie state
3. Parent should call wait() to read exit status
4. If parent doesn't call wait(), child remains zombie
```

**Characteristics:**
- Shows as `<defunct>` or `Z` in process list
- Takes up a PID slot (limited resource)
- No CPU or memory usage (just process table entry)
- Cannot be killed (already dead!)

**Solution:**
- Parent calls `wait()` or `waitpid()` to reap zombie
- If parent dies, init/systemd (PID 1) adopts and reaps it

**Interview Question:** "How do you handle zombies?"
**Answer:** Fix the parent process to properly call wait(), or restart the parent. Killing zombies doesn't work—they're already terminated.

---

## 3️⃣ CPU ARCHITECTURE

### 🔹 What are CPU Cores?
**Quick Definition:** Independent processing units within a CPU that can execute instructions simultaneously.

**Key Points:**
- **Single Core**: Executes one thread at a time (switches between tasks)
- **Multi-Core**: Each core can run a different thread truly in parallel
- **Example**: 8-core CPU can run 8 threads simultaneously

**Why More Cores Matter:**
✅ Multitasking (running multiple applications)
✅ Parallel workloads (video encoding, compilation, servers)
✅ Better efficiency under heavy load

**Interview Insight:** More cores ≠ always faster. Single-threaded apps don't benefit from extra cores.

---

### 🔹 What is Clock Speed?
**Quick Definition:** Frequency at which a CPU core executes instructions, measured in GHz (billions of cycles per second).

**What It Means:**
- **3.5 GHz** = 3.5 billion cycles per second
- Higher clock speed = faster instruction execution **per core**
- Impacts single-threaded performance (gaming, application launch)

**Formula:**
```
Instructions Per Second = Clock Speed × Instructions Per Cycle (IPC)
```

**Trade-off:**
- Higher clock speed = more heat and power consumption
- Modern CPUs boost clock speed dynamically (Turbo Boost)

---

### 🔹 Cores vs. Clock Speed
**Quick Summary:**
```
Clock Speed → How FAST one core works (single-thread speed)
Core Count  → How MANY tasks run AT ONCE (parallel throughput)
```

**Scenarios:**
- **Gaming, Office Apps**: High clock speed matters more (3.8+ GHz)
- **Video Editing, 3D Rendering, Servers**: More cores matter (8-16+)
- **Best CPU**: Balance of both (e.g., 8 cores @ 4.5 GHz)

**Interview Example:**
"For a database server handling 1000 concurrent queries, I'd choose a 16-core CPU even at lower clock speed, because parallelism matters more than single-thread speed."

---

### 🔹 What is Hyper-Threading?
**Quick Definition:** Intel's technology (Simultaneous Multi-Threading/SMT) that allows one physical core to execute two threads concurrently.

**How It Works:**
```
Physical Core 1:
  ├── Thread 1 (using ALU)
  └── Thread 2 (waiting for memory)
      → While Thread 2 waits, Thread 1 uses idle execution units
      → Better core utilization!
```

**Key Points:**
- Duplicates: Registers, scheduling logic (small overhead)
- Shares: ALU, FPU, cache, execution units
- Result: ~20-30% performance gain (not 2×)

**Example:**
4-core CPU with Hyper-Threading = 4 physical cores, 8 logical threads

**Interview Insight:** Hyper-Threading improves throughput on multi-threaded workloads but doesn't double performance—threads still share execution resources.

---

## 4️⃣ VIRTUALIZATION

### 🔹 What is a Virtual Machine (VM)?
**Quick Definition:** Software-based emulation of a physical computer that runs its own OS and applications.

**Components:**
```
Physical Server (Host)
    ├── Hypervisor (KVM, VMware, Hyper-V)
        ├── VM 1 (Guest OS: Ubuntu)
        ├── VM 2 (Guest OS: Windows)
        └── VM 3 (Guest OS: CentOS)
```

**Key Terms:**
- **Host OS**: Physical machine's operating system
- **Guest OS**: OS running inside VM
- **Hypervisor**: Software that creates and manages VMs; allocates resources

**Benefits:**
✅ Resource consolidation (1 physical server → 10 VMs)
✅ Isolation (one VM crash doesn't affect others)
✅ Easy snapshots and backups
✅ Hardware abstraction (migrate VMs between servers)

---

### 🔹 How Does Paging Work in VMs?
**Quick Definition:** Virtualized environments use **two-level address translation** for memory management.

**Two-Level Paging:**
```
Guest OS Level:
  Guest Virtual Address → Guest Physical Address
      (VM thinks this is real physical memory)
             ↓
Hypervisor Level:
  Guest Physical Address → Host Physical Address
      (actual RAM on physical server)
```

**Process:**
1. Process in VM accesses virtual memory
2. VM's page table maps it to "guest physical" address
3. Hypervisor maps "guest physical" to actual host RAM

**Technology:** 
- **Nested Paging / EPT (Intel) / NPT (AMD)**: Hardware-assisted two-level translation
- Improves performance by reducing hypervisor intervention

**Interview Point:** "VMs add a layer of address translation, which is why nested paging hardware support is crucial for VM performance."

---

## 5️⃣ SYSTEM MONITORING

### 🔹 What is Load Average? ⭐⭐⭐⭐
**Quick Definition:** Average number of processes in **runnable** (ready/running) or **uninterruptible** (waiting for I/O) state over 1, 5, and 15 minutes.

**How to Read:**
```bash
$ uptime
15:23:45 up 10 days, load average: 2.15, 1.80, 1.50
                                    1min  5min  15min
```

**Interpretation (for 4-core system):**
```
Load < 4.0    → Healthy, CPU has idle capacity
Load = 4.0    → Fully utilized, no waiting
Load > 4.0    → Processes waiting for CPU (overloaded)
```

**Trend Analysis:**
- **Increasing** (1.5 → 1.8 → 2.1): Problem getting worse
- **Decreasing** (2.5 → 2.0 → 1.5): System recovering
- **Stable**: Predictable load

**Common Mistake:** Load average includes I/O wait, not just CPU! High load doesn't always mean CPU bottleneck—could be disk I/O.

**Interview Question:** "Load average is 8.0 on a 4-core system. What do you check?"
**Answer:** 
1. `top` - Check CPU% vs. I/O wait%
2. `iostat` - Check if disk is bottleneck
3. `ps aux --sort=-%cpu` - Identify CPU-hungry processes

---

### 🔹 What is OOM Killer?
**Quick Definition:** Linux kernel mechanism that kills processes when system runs out of memory (RAM + swap exhausted).

**How It Works:**
```
1. System runs out of memory
2. Kernel calculates OOM score for each process
3. Kills process with highest score
4. Frees memory for other processes
```

**OOM Score Factors:**
- Memory usage (higher = more likely to be killed)
- Process importance (system processes protected)
- Runtime (newer processes more likely killed)
- Nice value (lower priority = higher score)

**Check OOM Scores:**
```bash
cat /proc/*/oom_score | sort -n
```

**Interview Question:** "How do you prevent OOM kills?"
**Answer:**
1. Add more RAM or swap
2. Set `oom_score_adj` to protect critical processes
3. Configure memory limits (cgroups, ulimit)
4. Monitor memory usage proactively

---

## 6️⃣ FILE SYSTEM CONCEPTS

### 🔹 What is an Inode? ⭐⭐⭐
**Quick Definition:** Data structure that stores file metadata; each file has exactly one inode.

**What Inode Stores:**
```
Inode Contains:
├── File permissions (rwxr-xr-x)
├── Owner (UID) and Group (GID)
├── File size (in bytes)
├── Timestamps (created, modified, accessed)
├── Link count (number of hard links)
├── Pointers to data blocks (where actual data is stored)
└── File type (regular, directory, symlink, etc.)
```

**What Inode DOES NOT Store:**
- ❌ Filename (stored in directory entry)
- ❌ Actual file data (stored in data blocks)

**How File Access Works:**
```
1. User opens "file.txt"
2. OS looks up filename in directory → finds inode number
3. OS reads inode → gets metadata and data block pointers
4. OS reads data blocks → returns file content
```

**Interview Question:** "Can multiple filenames point to the same inode?"
**Answer:** Yes! **Hard links** allow multiple directory entries to reference the same inode. Example:
```bash
ln file.txt hardlink.txt  # Both point to same inode
```

**Running Out of Inodes:**
Even with free disk space, you can't create new files if inodes are exhausted (common with many small files).

**Check Inode Usage:**
```bash
df -i   # Shows inode usage per filesystem
```

---

## 🎯 INTERVIEW STRATEGY TIPS

### Question Pattern Recognition:
1. **Definition Questions**: "What is X?" → Give 2-sentence definition + key components
2. **Scenario Questions**: "System is slow, what do you check?" → Systematic troubleshooting steps
3. **Comparison Questions**: "Process vs. Thread?" → Create comparison table
4. **Deep-Dive Questions**: "Explain how context switching works" → Step-by-step with diagrams

### Common Follow-ups to Expect:
- **After Paging**: "What happens during a page fault?"
- **After Process/Thread**: "How do they communicate?"
- **After Load Average**: "Load is 10 on 4-core system, what do you do?"
- **After Cache**: "What is cache coherence in multi-core systems?"
- **After Context Switching**: "How do you reduce context switches?"

### Golden Rule:
**Start simple, go deep if asked.** Don't over-explain unless the interviewer probes deeper.

---

## 📊 QUICK COMPARISON TABLES

### Process vs. Thread
| Aspect | Process | Thread |
|--------|---------|--------|
| **Memory** | Separate address space | Shared address space |
| **Creation** | Expensive (ms) | Cheap (μs) |
| **Communication** | IPC (pipes, sockets) | Shared memory (direct) |
| **Crash Impact** | Isolated | Affects all threads |
| **Use Case** | Isolation needed | Parallelism within app |

### Cache Levels
| Level | Size | Speed | Scope |
|-------|------|-------|-------|
| **L1** | 32-64 KB | ~1 ns | Per-core |
| **L2** | 256 KB-1 MB | ~4 ns | Per-core |
| **L3** | 8-32 MB | ~12 ns | Shared |
| **RAM** | 8-64 GB | ~100 ns | System |

---

**Last Updated:** November 2025  
**Focus Areas:** Memory management, process scheduling, CPU architecture, virtualization basics
