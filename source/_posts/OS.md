---
title: Operating System
date: 2018-11-16 21:47:21
updated: 2020-07-08 12:00:00
tags: [OS]
typora-root-url: ../
---

Memory Management, Interprocess Communication, Process Management, Threads, I/O

<!-- more -->

## 1. Memory Management

### 1.1 Virtual Memory

> Virtual memory (also virtual storage) is a memory management technique that provides an "idealized abstraction of the storage resources that are actually available on a given machine" which "creates the illusion to users of a very large (main) memory".

CPU无法直接访问其他存储设备。

通过paging技术，可以使主存与其他存储设备逻辑上看起来映射到了同一个虚拟地址空间。

### 1.2 Virtual Address Space

A virtual address space (VAS) or address space is the set of ranges of virtual addresses that an operating system makes available to a process.

转换虚拟地址需要借助硬件，比如CPU的MMU。使用paging技术。

### 1.3 Paging and Segmentation

#### Segmentation

> Memory segmentation is a computer (primary) memory management technique of division of a computer's primary memory into segments or sections. In a computer system using segmentation, a reference to a memory location includes a value that identifies a segment and an offset (memory location) within that segment.

> The x86-64 architecture does not use segmentation in long mode (64-bit mode). Four of the segment registers: CS, SS, DS, and ES are forced to 0, and the limit to 2^64.

> In a x86-64 architecture it is considered legacy and most x86-64-based modern system software don't use memory segmentation. Instead they handle programs and their data by utilizing memory-paging which also serves as a way of memory protection. However most x86-64 implementations still support it for backward compatibility reasons.
>

#### Paging

> Paging is a memory management scheme by which a computer stores and retrieves data from secondary storage for use in main memory. In this scheme, the operating system retrieves data from secondary storage in same-size blocks called pages. Paging is an important part of virtual memory implementations in modern operating systems, using secondary storage to let programs exceed the size of available physical memory.

##### Page Table

- Inverted page tables
- Multilevel page tables
- Virtualized page tables
- Nested page tables

##### Page Fault

> When a process tries to reference a page not currently present in RAM, the processor treats this invalid memory reference as a page fault and transfers control from the program to the operating system. The operating system must:
>
> 1. Determine the location of the data on disk.
> 2. Obtain an empty page frame in RAM to use as a container for the data.
> 3. Load the requested data into the available page frame.
> 4. Update the page table to refer to the new page frame.
> 5. Return control to the program, transparently retrying the instruction that caused the page fault.
>
> When all page frames are in use, the operating system must select a page frame to reuse for the page the program now needs. If the evicted page frame was dynamically allocated by a program to hold data, or if a program modified it since it was read into RAM (in other words, if it has become "dirty"), it must be written out to disk before being freed. If a program later references the evicted page, another page fault occurs and the page must be read back into RAM.
>
> The method the operating system uses to select the page frame to reuse, which is its page replacement algorithm, is important to efficiency. The operating system predicts the page frame least likely to be needed soon, often through the least recently used (LRU) algorithm or an algorithm based on the program's working set. To further increase responsiveness, paging systems may predict which pages will be needed soon, preemptively loading them into RAM before a program references them.
>

##### Page Cache

> A page cache is implemented in kernels with the paging memory management, and is mostly transparent to applications.

### 1.4 Linux Memory Model

- FLATMEM

- DISCONTIGMEM

- SPARSEMEM

> Recent memory-management developments, such as memory hotplug, persistent-memory support, and various performance optimizations, all target the SPARSEMEM model. But the older models still exist, which comes with the cost of numerous #ifdef blocks in the architecture and memory-management code, and a peculiar maze of related configuration options. There is an ongoing work to completely switch the remaining users of DISCONTIGMEM to SPARSEMEM, but making the change for such architectures as ia64 and mips64 won't be an easy task. And the ARC architecture's use of DISCONTIGMEM to represent a "high memory" area that resides below the "normal" memory will definitely be challenging to change.

### 1.5 Memory Management Unit

MMU用于将虚拟地址转换成物理地址。

TLB (Translation-Lookaside Buffer）是MMU的一部分，存储一些最近使用的page table中的mapping，用于加速虚拟地址到物理地址的转换。

### 1.6 Buddy Allocation

管理一个bitmap的数据结构。

Buddy算法用于管理空闲页，快速分配内存，以及回收内存时，也需要重新构建索引。

回收内存页时，通过bitmap可以快速判定是否有内存块可以与之合并。

一个64位Linux内核的buddyinfo，从左到右，数值的单位依次是，2的0次方页->2的10次方页。

![](/images/os1.png)

### 1.7 Slab Allocation

> In Linux, slab allocation provides a kind of front-end to the zoned buddy allocator for those sections of the kernel that require more flexible memory allocation than the standard 4KB page size.

> The technique is used to retain allocated memory that contains a data object of a certain type for reuse upon subsequent allocations of objects of the same type.

该方法用于在内核中分配内核对象时进行优化。

![](/images/os2.png)

### 1.8 malloc, vmalloc, kmalloc

kmalloc使用的slab算法，分配到的内存较小，物理上连续，在内核空间使用。

vmalloc、malloc在buddy系统上使用，分配的内存较大，分配到的内存在逻辑上连续。vmalloc在内核空间使用。

malloc是libc库的一个函数。

## 2. Interprocess Communication

IPC的方法繁多。

Pipe

File

Shared memory

Message queue/passing

mmap

Signal

Socket（应用系统开发最常用）

### 2.1 Applications

#### Remote procedure call

主要组件包括：Transporter、Protocol Design、Dispatcher、多线程设计、序列化组件、配套的客户端等等。

## 3. Process Management

> Linux uses a 1-1 threading model, with (to the kernel) no distinction between processes and threads -- everything is simply a runnable task.

### 3.1 Completely Fair Scheduler

> The data structure used for the scheduling algorithm is a red-black tree in which the nodes are scheduler-specific sched_entity structures. These are derived from the general task_struct process descriptor, with added scheduler elements.
>
> The nodes are indexed by processor "execution time" in nanoseconds.
>
> A "maximum execution time" is also calculated for each process to represent the time the process would have expected to run on an "ideal processor". This is the time the process has been waiting to run, divided by the total number of processes.
>
> When the scheduler is invoked to run a new process:
>
> 1. The leftmost node of the scheduling tree is chosen (as it will have the lowest spent execution time), and sent for execution.
> 2. If the process simply completes execution, it is removed from the system and scheduling tree.
> 3. If the process reaches its maximum execution time or is otherwise stopped (voluntarily or via interrupt) it is reinserted into the scheduling tree based on its new spent execution time.
> 4. The new leftmost node will then be selected from the tree, repeating the iteration.
>
> If the process spends a lot of its time sleeping, then its spent time value is low and it automatically gets the priority boost when it finally needs it. Hence such tasks do not get less processor time than the tasks that are constantly running.
>
> The fair queuing CFS scheduler has a scheduling complexity of O(log N), where N is the number of tasks in the runqueue. Choosing a task can be done in constant time, but reinserting a task after it has run requires O(log N) operations, because the runqueue is implemented as a red-black tree.

## 4. Threads and Concurrency

### 4.1 Light-weight process

> A light-weight process (LWP) is a means of achieving multitasking. In the traditional meaning of the term, a LWP runs in user space on top of a single kernel thread and shares its address space and system resources with other LWPs within the same process.
>

> LWPs are slower and more expensive to create than user threads. Whenever an LWP is created a system call must first be made to create a corresponding kernel thread, causing a switch to kernel mode. These mode switches would typically involve copying parameters between kernel and user space, also the kernel may need to have extra steps to verify the parameters to check for invalid behavior. A context switch between LWPs means that the LWP that is being preempted has to save its registers, then go into kernel mode for the kernel thread to save its registers, and the LWP that is being scheduled must restore the kernel and user registers separately also.
>
> For this reason, some user level thread libraries allow multiple user threads to be implemented on top of LWPs. User threads can be created, destroyed, synchronized and switched between entirely in user space without system calls and switches into kernel mode. This provides a significant performance improvement in thread creation time and context switches. However, there are difficulties in implementing a user level thread scheduler that works well together with the kernel.

### 4.2 Kernel threads

> Kernel threads are handled entirely by the kernel. They need not be associated with a process; a kernel can create them whenever it needs to perform a particular task. Kernel threads cannot execute in user mode. LWPs (in systems where they are a separate layer) bind to kernel threads and provide a user-level context. This includes a link to the shared resources of the process to which the LWP belongs. When a LWP is suspended, it needs to store its user-level registers until it resumes, and the underlying kernel thread must also store its own kernel-level registers.

### 4.3 1:1 (kernel-level threading)

> Threads created by the user in a 1:1 correspondence with schedulable entities in the kernel are the simplest possible threading implementation. OS/2 and Win32 used this approach from the start, while on Linux the usual C library implements this approach (via the NPTL or older LinuxThreads).

### 4.4 Fiber、Coroutine、Green threads

> Fibers describe essentially the same concept as coroutines. The distinction, if there is any, is that coroutines are a language-level construct, a form of control flow, while fibers are a systems-level construct, viewed as threads that happen to not run concurrently. It is contentious which of the two concepts has priority: fibers may be viewed as an implementation of coroutines, or as a substrate on which to implement coroutines.

> Green threads are threads that are scheduled by a runtime library or virtual machine (VM) instead of natively by the underlying operating system (OS). Green threads emulate multi-threaded environments without relying on any native OS abilities, and they are managed in user space instead of kernel space, enabling them to work in environments that do not have native thread support.

## 5. I/0 Management

### 5.1 Peripheral Component Interconnect

![](/images/os3.png)

> 1. The PCI bus connects high-speed high-bandwidth devices to the memory subsystem ( and the CPU. )
> 2. The expansion bus connects slower low-bandwidth devices, which typically deliver data one character at a time ( with buffering. )
> 3. The SCSI bus connects a number of SCSI devices to a common SCSI controller.
> 4. A daisy-chain bus, ( not shown) is when a string of devices is connected to each other like beads on a chain, and only one of the devices is directly connected to the host.

### 5.2 Interrupts

> Interrupts allow devices to notify the CPU when they have data to transfer or when an operation is complete, allowing the CPU to perform other duties when no I/O transfers need its immediate attention.

### 5.3 Application I/O Interface

> Most devices can be characterized as either block I/O, character I/O, memory mapped file access, or network sockets. A few devices are special, such as time-of-day clock and the system timer.

#### Network Devices

> One common and popular interface is the socket interface, which acts like a cable or pipeline connecting two networked entities. Data can be put into the socket at one end, and read out sequentially at the other end. Sockets are normally full-duplex, allowing for bi-directional data transfer.

#### 五种I/O模型

- blocking I/O。以recvfrom为例，等待数据与复制数据过程中全部阻塞。
- nonblocking I/O。以recvfrom为例，等待数据时通过轮询来确认数据是否准备完毕，并在复制数据过程中阻塞。
- I/O multiplexing。以select为例，通过多路复用可以监听多个fd，在数据复制阶段仍然阻塞。
- signal-driven I/O。
- asynchronous I/O。

##### select需要配合非阻塞I/O

> Under Linux, select() may report a socket file descriptor as "ready for reading", while nevertheless a subsequent read blocks. This could for example happen when data has arrived but upon examination has wrong checksum and is discarded. There may be other circumstances in which a file descriptor is spuriously reported as ready. Thus it may be safer to use O_NONBLOCK on sockets that should not block.

##### epoll

> epoll provides both edge-triggered and level-triggered modes. In edge-triggered mode, a call to epoll_wait will return only when a new event is enqueued with the epoll object, while in level-triggered mode, epoll_wait will return as long as the condition holds.
>
> For instance, if a pipe registered with epoll has received data, a call to epoll_wait will return, signaling the presence of data to be read. Suppose, the reader only consumed part of data from the buffer. In level-triggered mode, further calls to epoll_wait will return immediately, as long as the pipe's buffer contains data to be read. In edge-triggered mode, however, epoll_wait will return only once new data is written to the pipe.

在边缘触发时，需要将socket设置成非阻塞，否则如果buffer中还有数据的话无法读到。