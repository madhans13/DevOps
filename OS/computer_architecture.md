what is pageing?

Paging is a memory management technique used by operating systems to implement virtual memory.
It divides both the virtual address space of a process and the physical memory (RAM) into fixed-size blocks —
called pages (virtual) and frames (physical).

The OS maintains a page table that maps each virtual page to a physical frame.
This allows processes to use memory non-contiguously and efficiently share limited RAM.

🧠 What is a Process (Interview Definition)

A process is an independent program in execution.
It has its own memory space, CPU registers, stack, and system resources managed by the operating system.
Each process runs in its own isolated address space, which ensures that one process cannot directly access another’s memory.

✅ Key Points to Mention in Interview:

Created by the OS when a program starts (e.g., running Chrome creates a Chrome process).

Has its own virtual address space and page table.

Processes communicate via Inter-Process Communication (IPC) — e.g., pipes, sockets, shared memory.

Heavyweight: creating/switching between processes is costlier.



⚙️ What is a Thread (Interview Definition)

A thread is the smallest unit of execution within a process.
Threads share the same memory space and resources of their parent process but have their own stack and registers.
They are sometimes called lightweight processes because they enable multitasking within a single process.

✅ Key Points to Mention in Interview:

All threads of a process share code, data, and open files.

Each thread has its own stack and program counter.

Faster to create and switch compared to processes.

Useful for performing multiple tasks simultaneously (parallelism).

💡 Example:

In Chrome:

One thread handles the UI,

Another handles network loading,

Another runs JavaScript.

All share the same process memory, but work independently — if one tab hangs, the others can still respond.




Interview Definition: What is Multithreading

Multithreading is the ability of a process to have multiple threads executing concurrently — each performing a different task while sharing the same memory and resources of that process.

It allows a program to perform multiple operations at the same time, improving responsiveness and resource utilization.

⚙️ In Simple Words:

Normally, a program (single-threaded) executes one task at a time — line by line.
But in multithreading, the same program can perform multiple tasks simultaneously within a single process.

✅ Threads run independently, but they share:

The same code, data, and heap memory of the process.

Each thread has its own stack, registers, and program counter.


what is hyperthreading?

“Hyper-Threading (Intel’s implementation of Simultaneous Multi-Threading) allows a single physical CPU core to execute multiple threads concurrently.
It duplicates certain parts of the processor (like registers and scheduling logic) so that while one thread is waiting, another can use idle execution units.
This improves CPU utilization and overall performance.”


what is cache?
Cache memory is a small, high-speed memory in the CPU that stores frequently accessed data and instructions to reduce access time to RAM.
It is divided into L1, L2, and L3 levels, where L1 is the smallest and fastest, and L3 is the largest and shared across cores.
Caching enhances CPU efficiency by exploiting temporal and spatial locality.

what is cores and clock speed?

Clock speed (measured in GHz) defines how fast each core executes instructions. It indicates the number of cycles a core completes per second — higher clock speeds mean faster instruction execution for single-threaded tasks, such as opening applications or gaming, where only one thread is active at a time.

Cores, on the other hand, represent how many independent processing units exist within a CPU. Each core can handle its own thread of execution, allowing multiple tasks to run in parallel. A CPU with more cores (like 8 or 16) can process multiple operations simultaneously, making it ideal for multitasking, video rendering, program compilation, and server workloads.

In short:

Clock speed = how fast one core works.

Core count = how many tasks can run at the same time.

A well-balanced CPU has both high clock speeds for responsiveness and multiple cores for efficiency and parallelism — which is why modern CPUs combine both features for optimal performance.


memory heirarcchy?

The memory hierarchy organizes storage components from fastest to slowest: registers → cache → RAM → secondary → tertiary storage.

This structure ensures that the CPU always gets data quickly from the nearest possible level while balancing speed, cost, and capacity.

Faster levels (like cache and registers) are small and expensive, while slower levels (like RAM and disk) are larger and cheaper — together they provide an optimal trade-off between performance and storage size.


what is virtual memory?

Virtual memory is a memory management technique that gives each process the illusion of having a large, continuous block of memory — even if the system’s physical RAM is smaller.

It allows the operating system to use both RAM and a portion of the disk (swap space) together to efficiently manage memory, isolate processes, and run programs that are larger than available physical memory.

⚙️ How Virtual Memory Works

Every process gets its own virtual address space, divided into fixed-size blocks called pages (usually 4 KB).

Physical RAM is divided into frames of the same size.

The OS maintains a page table for each process to map virtual pages → physical frames.

When a process accesses a virtual address:

The MMU (Memory Management Unit) translates it to a physical address using the page table.

If the page is not currently in RAM, a page fault occurs, and the OS loads it from disk (swap space) into memory.

🧩 Example

Imagine you have:

8 GB of physical RAM

16 GB of virtual memory configured (8 GB RAM + 8 GB swap)

If a process uses 10 GB of memory:

The most frequently used pages stay in RAM.

The less-used pages are stored in swap space on disk.

When needed again, the OS swaps them back into RAM.

This gives the appearance that you have more memory than physically installed.

💾 Swap Space

Swap space (on disk) acts as an overflow area for inactive memory pages.

It’s much slower than RAM, but allows more processes to run simultaneously without crashing due to “out of memory” errors.



What is a Virtual Machine (VM)

A Virtual Machine (VM) is a software-based emulation of a physical computer that runs an operating system (guest OS) and applications as if it were a separate physical machine.
It is created and managed by a hypervisor, which allocates hardware resources (CPU, memory, storage, network) from the host system and presents them virtually to the VM.

how paging works in vms?

In a virtualized environment, each Virtual Machine (VM) runs its own operating system with its own virtual memory and paging system.

The hypervisor (like KVM, VMware, or Hyper-V) manages an additional layer of paging that maps guest physical memory to actual host physical memory.

So effectively, paging happens at two levels — inside the guest OS and again by the hypervisor — a concept known as nested paging or two-level address translation.


what is load average?

Load average in Linux shows the average number of processes actively running or waiting for CPU or I/O over the past 1, 5, and 15 minutes.

The three values help track short-term and long-term trends in system workload.

A system is considered healthy when the load average is roughly equal to or less than the number of CPU cores.

In short — it’s a real-time indicator of system load, not just CPU utilization.

what is context switching?

3️⃣ Context Switching ⭐⭐⭐⭐
What Interviewer Asks:

"What is context switching?"
"Why is it expensive?"
"How do you detect excessive context switching?"

Your Answer Template:
Definition:
"Context switching is when the CPU stops executing one process and starts executing another. The OS saves the current process state and loads the next process state."
What Gets Saved/Restored:
Context includes:
├── CPU registers (Program Counter, Stack Pointer)
├── Page table pointer
├── Process state
└── Open file descriptors

Time: 1-10 microseconds per switch
Why It's Expensive:
Direct cost: Saving/loading registers (small)
Indirect cost: Cache invalidation (BIG!)

When switching from Process A to Process B:
├── CPU cache contains Process A's data
├── Switch happens
├── CPU cache now "cold" for Process B
└── Must reload from RAM (100× slower)

This cache miss penalty hurts more than the switch itself!


what is OOM killer/

What is OOM Killer:
"When physical RAM and swap are exhausted, Linux's Out-Of-Memory Killer selects and kills a process to free memory. It calculates an OOM score for each process based on memory usage and importance."

New → Ready → Running → Waiting → Terminated

Zombie Process:
Definition:
"A zombie process has finished execution but its parent hasn't read its exit status. It takes up a PID slot and shows as <defunct> in process list."



what is inodes?

What is an Inode:
"An inode is a data structure that stores file metadata - permissions, ownership, size, timestamps, and pointers to data blocks. Each file has one inode. The inode does NOT store the filename or actual data."

what is load average?

Interview Answer:
"Load average shows the average number of processes in runnable or uninterruptible state. On a 4-core system:

Load < 4.0: Healthy, CPU has capacity
Load = 4.0: Fully utilized
Load > 4.0: Processes waiting for CPU

The three numbers show 1-minute, 5-minute, and 15-minute averages. Increasing trend (1.8 → 2.0 → 2.5) indicates growing problem."
