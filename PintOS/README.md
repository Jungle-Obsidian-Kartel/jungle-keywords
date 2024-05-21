# PintOS

# Project 0: PintOS

Virtual Machine ~= Hypervisor

- qemu

Common bugs

- Memory leak
- Race condition
- Deadlock
- Use after free

# Project 1: Threads

Time-sharing system

- Context Switching
- Scheduler
    - Round Robin (RR)
    - Priority
        - Priority donation
    - Multi-Level Feedback Queue Scheduler (MLFQS)
        - 4BSD
        - nice
- Thread
    - Thread control block (TCB)

Timer Interrupt

Timer sleep

Synchronization

- Semaphore
- lock
- condvar

# Project 2: User Programs

User mode vs. Kernel mode

Process

- Process Environment block (PEB)
- Process identifier (PID)

User Stack

x86_64 calling convention

Register vs. Memory

argument vector

Executable Linkable Format (ELF) & loader

system call (syscall)

**!!CAUTION!! some system call’s semantic may differ from POSIX standards**

- filesys related
    - open, close, create, read, write, seek, tell, ...
- process related
    - halt, exit, exec, fork, wait ...

file descriptor

- file descriptor table
- dup2 syscall

# Project 3: Virtual Memory

Virtual memory management

- paging
    - virtual page
    - physical frame
    - page table & supplementary page table
- Memory Management Unit (MMU)
- Translation Lookaside Buffer (TLB)

Page type

- Uninitialized page
    - Lazy initialization
- Anonymous page
    - stack growth
- file-backed page
    - mmap syscall

Swap in/out

- page replacement policy
- swap disk

Copy on Write (COW)

# Project 4: File System

Type of file system

- FAT(File Allocation Table)
- Berkley FFS
- EXT

File system component

- file
    - extensible file
- directory
    - working directory
- link
    - hard link vs soft link

File system implementation

- sector & cluster
- super block
- disk inode
- in-memory inode
- open inode table

Buffer cache

filesystem mount

Journaling
