# Chapter 2 Operating system organization

An operating system must fulfill three requirements: multiplexing, isolation, and interaction



## Abstracting physical resoures

The first question one might ask when encountering an operating system is **why** have it at all?



### Strawman solution

![image-20220427103446998](https://s2.loli.net/2022/04/27/mawnxy4ZMLQY9eS.png)



 **Problem: Can’t multiplex**

* Each app must periodically give up hardware 

* But weak isolation 
	*  App forgets to give up -> nothing else runs 
	*  Apps has end-less loop -> nothing else runs 
	*  Can’t even kill the misbehaving app from another app 
*  This scheme is sometimes used in practice 
	* Called cooperative scheduling

**Bigger problem: Memory isolation**

* All apps share physical memory 
* One app can overwrite another app’s memory 
*  One app can overwrite the OS library 
* No security!



**This means that the application must be well-behaved !!! **( it's difficult )



### UNIX interface

**Processes (instead of cores): fork()** 

*  OS transparently allocates cores 
*  Save + Restore registers 
*  Enforces that processes give them up 
*  Periodically re-allocates cores

**Memory (instead of physical memory): exec(), brk()**

*  Each process has its own memory 
*  OS decides where to place app in memory 
*  OS enforces isolation between apps 
*  OS stores image in file system (loaded with exec)

**Files instead of disk blocks: open(), read(), write()** 

* OS provides convenient names 
* OS allows files to be shared by apps and users 

**Pipes instead of shared physical memory: pipe()** 

* OS buffers data 
* OS stops sender if it transmits too fast

## User mode, supervisor mode,and system calls



Strong isolation **requires a hard boundary** between applications and the operating system

![image-20220427101641527](https://s2.loli.net/2022/04/27/lPmRudUEWX4vApc.png)

| mode            | privilege                                                |
| --------------- | -------------------------------------------------------- |
| User mode       | only user-mode instructions (e.g., adding numbers, etc.) |
| supervisor mode | the CPU is allowed to execute privileged instructions    |
| machine mode    | have full privilege                                      |



![image-20220427101614510](https://s2.loli.net/2022/04/27/rYtIKL15aSZMyXV.png)

## Kernel organization

### Monolithic kernels

![image-20220427102059611](https://s2.loli.net/2022/04/27/iRkq8FpD5chPNB1.png)

### Microkernels

![image-20220427102155295](https://s2.loli.net/2022/04/27/g5HNOZUclXeY8fb.png)

![image-20220427102252926](https://s2.loli.net/2022/04/27/sV41ChkvWRKNzUn.png)

## Code: xv6 organization

![image-20220426150355419](https://s2.loli.net/2022/04/26/3YcFxZAEOas5LBw.png)

![image-20220426150319467](https://s2.loli.net/2022/04/26/9CkXwsiM8354deL.png)

## Process overview

![image-20220426150935703](https://s2.loli.net/2022/04/26/Tf7Oe6SHZvKwRL1.png)



The unit of isolation in xv6 (as in other Unix operating systems) is a process



* **Own memory (address)** : A process provides a program with what appears to be a private memory system, or address space, which other processes cannot read or write.
	* Xv6 uses **page tables (which are implemented by hardware)** to give each process its own address space.
*  **Own CPU (thread)** : A process also provides the program with what appears to be its own CPU to execute the program’s instructions.





`proc.h`

```c
// Saved registers for kernel context switches.
struct context {
  uint64 ra;
  uint64 sp;

  // callee-saved
  uint64 s0;
  uint64 s1;
  uint64 s2;
  uint64 s3;
  uint64 s4;
  uint64 s5;
  uint64 s6;
  uint64 s7;
  uint64 s8;
  uint64 s9;
  uint64 s10;
  uint64 s11;
};

// Per-CPU state.
struct cpu {
  struct proc *proc;          // The process running on this cpu, or null.
  struct context context;     // swtch() here to enter scheduler().
  int noff;                   // Depth of push_off() nesting.
  int intena;                 // Were interrupts enabled before push_off()?
};

extern struct cpu cpus[NCPU];

// per-process data for the trap handling code in trampoline.S.
// sits in a page by itself just under the trampoline page in the
// user page table. not specially mapped in the kernel page table.
// the sscratch register points here.
// uservec in trampoline.S saves user registers in the trapframe,
// then initializes registers from the trapframe's
// kernel_sp, kernel_hartid, kernel_satp, and jumps to kernel_trap.
// usertrapret() and userret in trampoline.S set up
// the trapframe's kernel_*, restore user registers from the
// trapframe, switch to the user page table, and enter user space.
// the trapframe includes callee-saved user registers like s0-s11 because the
// return-to-user path via usertrapret() doesn't return through
// the entire kernel call stack.
struct trapframe {
  /*   0 */ uint64 kernel_satp;   // kernel page table
  /*   8 */ uint64 kernel_sp;     // top of process's kernel stack
  /*  16 */ uint64 kernel_trap;   // usertrap()
  /*  24 */ uint64 epc;           // saved user program counter
  /*  32 */ uint64 kernel_hartid; // saved kernel tp
  /*  40 */ uint64 ra;
  /*  48 */ uint64 sp;
  /*  56 */ uint64 gp;
  /*  64 */ uint64 tp;
  /*  72 */ uint64 t0;
  /*  80 */ uint64 t1;
  /*  88 */ uint64 t2;
  /*  96 */ uint64 s0;
  /* 104 */ uint64 s1;
  /* 112 */ uint64 a0;
  /* 120 */ uint64 a1;
  /* 128 */ uint64 a2;
  /* 136 */ uint64 a3;
  /* 144 */ uint64 a4;
  /* 152 */ uint64 a5;
  /* 160 */ uint64 a6;
  /* 168 */ uint64 a7;
  /* 176 */ uint64 s2;
  /* 184 */ uint64 s3;
  /* 192 */ uint64 s4;
  /* 200 */ uint64 s5;
  /* 208 */ uint64 s6;
  /* 216 */ uint64 s7;
  /* 224 */ uint64 s8;
  /* 232 */ uint64 s9;
  /* 240 */ uint64 s10;
  /* 248 */ uint64 s11;
  /* 256 */ uint64 t3;
  /* 264 */ uint64 t4;
  /* 272 */ uint64 t5;
  /* 280 */ uint64 t6;
};

enum procstate { UNUSED, USED, SLEEPING, RUNNABLE, RUNNING, ZOMBIE };

// Per-process state
struct proc {
  struct spinlock lock;

  // p->lock must be held when using these:
  enum procstate state;        // Process state
  void *chan;                  // If non-zero, sleeping on chan
  int killed;                  // If non-zero, have been killed
  int xstate;                  // Exit status to be returned to parent's wait
  int pid;                     // Process ID

  // wait_lock must be held when using this:
  struct proc *parent;         // Parent process

  // these are private to the process, so p->lock need not be held.
  uint64 kstack;               // Virtual address of kernel stack
  uint64 sz;                   // Size of process memory (bytes)
  pagetable_t pagetable;       // User page table
  struct trapframe *trapframe; // data page for trampoline.S
  struct context context;      // swtch() here to run process
  struct file *ofile[NOFILE];  // Open files
  struct inode *cwd;           // Current directory
  char name[16];               // Process name (debugging)
};
```



## Code:starting xv6,the first process and system call



## Security Model

* The operating system must assume that a process’s user-level code will do its best to wreck the kernel or other processes.
*  Kernel code is assumed to be written by well-meaning and careful programmers

![image-20220427101901599](https://s2.loli.net/2022/04/27/wGVBaD2OITS5Jiz.png)