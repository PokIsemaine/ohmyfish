<h1>Lab 3 page table</h1>

 <h2>Speed up system calls</h2>

Some operating systems (e.g., Linux) speed up certain system calls by sharing data in a read-only region between userspace and the kernel. This eliminates the need for kernel crossings when performing these system calls. To help you learn how to insert mappings into a page table, your first task is to implement this optimization for the `getpid()` system call in xv6.

When each process is created, map one read-only page at USYSCALL (a VA defined in `memlayout.h`). At the start of this page, store a `struct usyscall` (also defined in `memlayout.h`), and initialize it to store the PID of the current process. For this lab, `ugetpid()` has been provided on the userspace side and will automatically use the USYSCALL mapping. You will receive full credit for this part of the lab if the `ugetpid` test case passes when running `pgtbltest`.

Some hints:

- You can perform the mapping in `proc_pagetable()` in `kernel/proc.c`.
- Choose permission bits that allow userspace to only read the page.
- You may find that `mappages()` is a useful utility.
- Don't forget to allocate and initialize the page in `allocproc()`.
- Make sure to free the page in `freeproc()`.

Which other xv6 system call(s) could be made faster using this shared page? Explain how.

## 题意

一些操作系统会在一个用户空间和内核之间的只读区域中共享数据来加速某些系统调用，这样就可以避免陷入内核。为了理解这个，先实现一个`getpid()`的加速。



## 过程

`pgtbltest.c`

```c
void
ugetpid_test()
{
  int i;

  printf("ugetpid_test starting\n");
  testname = "ugetpid_test";

  for (i = 0; i < 64; i++) {
    int ret = fork();
    if (ret != 0) {
      wait(&ret);
      if (ret != 0)
        exit(1);
      continue;
    }
    if (getpid() != ugetpid())
      err("missmatched PID");
    exit(0);
  }
  printf("ugetpid_test: OK\n");
}
```



### step1 页面映射

我们从这句话开始“When each process is created, map one read-only page at USYSCALL (a VA defined in `memlayout.h`)“

每个进程被创建的时候，映射一个`read-only`页到`USYSCALL`（一个虚拟地址）

`USYSCALL`在哪里？

看到`memlayout.h`里的一段注释

```c
// User memory layout.
// Address zero first:
//   text
//   original data and bss
//   fixed-size stack
//   expandable heap
//   ...
//   USYSCALL (shared with kernel)
//   TRAPFRAME (p->trapframe, used by the trampoline)
//   TRAMPOLINE (the same page as in the kernel)

#ifdef LAB_PGTBL
#define USYSCALL (TRAPFRAME - PGSIZE)

struct usyscall {
  int pid;  // Process ID
};
#endif
```

可以发现`USYSCALL`在`TRAPFRAME`的下方

![image-20220425142244817](../../../../../../.config/Typora/typora-user-images/image-20220425142244817.png)

![image-20220428081606937](https://s2.loli.net/2022/04/28/G4yEQ7WbvxriqNA.png)

谁来完成这项映射的工作？根据提示可以知道要去看`proc_pagetable`函数

```c
// Create a user page table for a given process,
// with no user memory, but with trampoline pages.
pagetable_t
proc_pagetable(struct proc *p)
{
  pagetable_t pagetable;

  // An empty page table.
  pagetable = uvmcreate();
  if(pagetable == 0)
    return 0;

  // map the trampoline code (for system call return)
  // at the highest user virtual address.
  // only the supervisor uses it, on the way
  // to/from user space, so not PTE_U.
  if(mappages(pagetable, TRAMPOLINE, PGSIZE,
              (uint64)trampoline, PTE_R | PTE_X) < 0){
    uvmfree(pagetable, 0);
    return 0;
  }

  // map the trapframe just below TRAMPOLINE, for trampoline.S.
  if(mappages(pagetable, TRAPFRAME, PGSIZE,
              (uint64)(p->trapframe), PTE_R | PTE_W) < 0){
    uvmunmap(pagetable, TRAMPOLINE, 1, 0);
    uvmfree(pagetable, 0);
    return 0;
  }

  return pagetable;
}
```

可以看到`proc_pagetable`已经做了对`TRAMPOLINE`和`TRAPFRAME`的映射失败的处理

以`TRAPFRAME`的部分代码来理解这一过程

```C
  // map the trapframe just below TRAMPOLINE, for trampoline.S.
if(mappages(pagetable, TRAPFRAME, PGSIZE,
              (uint64)(p->trapframe), PTE_R | PTE_W) < 0){
    uvmunmap(pagetable, TRAMPOLINE, 1, 0);
    uvmfree(pagetable, 0);
    return 0;
}
```

涉及3个函数

`int mappages(pagetable_t pagetable, uint64 va, uint64 size, uint64 pa, int perm)`

创建一个`PTE`使得虚拟地址`va`映射到`pa`，`va`和`size`可能没有对齐

`perm`表示属性标志

返回值：0 表示成功，-1表示`walk()`没有分配到需要的`page-table page`



`void uvmunmap(pagetable_t pagetable, uint64 va, uint64 npages, int do_free)`

删除 从`va`开始删除`npages` 个映射，`va`必须页对齐，映射必须存在，可选择是否释放物理内存



`void uvmfree(pagetable_t pagetable, uint64 sz)`

释放`user memory pages`和`page-table pages`



所以这段代码的就是创建一个`PTE`使得`TRAPFRAME`映射到对应的物理地址`(uint64)(p->trapframe)`

如果失败了就要删除`TRAMPOLINE`的映射并释放内存页和页表页



照葫芦画瓢，我们来完成`USYSCALL`的映射，并且设置标志位为`PTE_R|PTE_U`

```c
// Create a user page table for a given process,
// with no user memory, but with trampoline pages.
pagetable_t
proc_pagetable(struct proc *p)
{
  pagetable_t pagetable;

  // An empty page table.
  pagetable = uvmcreate();
  if(pagetable == 0)
    return 0;

  // map the trampoline code (for system call return)
  // at the highest user virtual address.
  // only the supervisor uses it, on the way
  // to/from user space, so not PTE_U.
  if(mappages(pagetable, TRAMPOLINE, PGSIZE,
              (uint64)trampoline, PTE_R | PTE_X) < 0){
    uvmfree(pagetable, 0);
    return 0;
  }

  // map the trapframe just below TRAMPOLINE, for trampoline.S.
  if(mappages(pagetable, TRAPFRAME, PGSIZE,
              (uint64)(p->trapframe), PTE_R | PTE_W) < 0){
    uvmunmap(pagetable, TRAMPOLINE, 1, 0);
    uvmfree(pagetable, 0);
    return 0;
  }

  //map the usyscall just below TRAMPOLINE and TRAPFRAME
  if(mappages(pagetable, USYSCALL, PGSIZE,
              (uint64)(p->usyscall), PTE_R | PTE_U) < 0){
    uvmunmap(pagetable, TRAMPOLINE, 1, 0);
    uvmunmap(pagetable, TRAPFRAME, 1, 0);
    uvmfree(pagetable, 0);
    return 0;
  }

  return pagetable;
}
```



回顾一下题目，现在我们已经使用了以下提示（标粗）

**When each process is created, map one read-only page at USYSCALL (a VA defined in `memlayout.h`)**. At the start of this page, store a `struct usyscall` (also defined in `memlayout.h`), and initialize it to store the PID of the current process. For this lab, `ugetpid()` has been provided on the userspace side and will automatically use the USYSCALL mapping. You will receive full credit for this part of the lab if the `ugetpid` test case passes when running `pgtbltest`.

Some hints:

- **You can perform the mapping in `proc_pagetable()` in `kernel/proc.c`.**
- **Choose permission bits that allow userspace to only read the page.**
- **You may find that `mappages()` is a useful utility.**
- Don't forget to allocate and initialize the page in `allocproc()`.
- Make sure to free the page in `freeproc()`.

Which other xv6 system call(s) could be made faster using this shared page? Explain how.



### step2 ugetpid()

接下来我们来关注这句话

”At the start of this page, store a `struct usyscall` (also defined in `memlayout.h`), and initialize it to store the PID of the current process“

在`USYSCALL`页的起始处存储一个`struct usyscall`，并且初始化这个`struct usyscall`来存储当前进程的`PID`



来看看`struct uyscall`长什么样吧，回顾一下先前给出的`memlayout.h`中的部分代码

```c
// User memory layout.
// Address zero first:
//   text
//   original data and bss
//   fixed-size stack
//   expandable heap
//   ...
//   USYSCALL (shared with kernel)
//   TRAPFRAME (p->trapframe, used by the trampoline)
//   TRAMPOLINE (the same page as in the kernel)

#ifdef LAB_PGTBL
#define USYSCALL (TRAPFRAME - PGSIZE)

struct usyscall {
  int pid;  // Process ID
};
#endif
```



“For this lab, `ugetpid()` has been provided on the userspace side and will automatically use the USYSCALL mapping.”

好耶，不用自己写`ugetpid()`了，去`ulib.c`文件中查看`ugetpid（）`长啥样

```c
#ifdef LAB_PGTBL
int
ugetpid(void)
{
  struct usyscall *u = (struct usyscall *)USYSCALL;	//因为struct usyscall 在 USYSCALL 的起始位置所以直接取UYSYCALL即可
  return u->pid;
}
#endif

```



### step3 内存管理

还剩最后两个提示没用到，来看看他们是做什么的

- Don't forget to allocate and initialize the page in `allocproc()`.
- Make sure to free the page in `freeproc()`.



首先来看看`allocproc`，其作用是给`UNUSED`的进程做初始化

```c
// Look in the process table for an UNUSED proc.
// If found, initialize state required to run in the kernel,
// and return with p->lock held.
// If there are no free procs, or a memory allocation fails, return 0.
static struct proc*
allocproc(void)
{
  struct proc *p;

  for(p = proc; p < &proc[NPROC]; p++) {
    acquire(&p->lock);
    if(p->state == UNUSED) {
      goto found;
    } else {
      release(&p->lock);
    }
  }

  return 0;

found:
  p->pid = allocpid();
  p->state = USED;

  // Allocate a trapframe page.
  if((p->trapframe = (struct trapframe *)kalloc()) == 0){
    freeproc(p);
    release(&p->lock);
    return 0;
  }

  // An empty user page table.
  p->pagetable = proc_pagetable(p);
  if(p->pagetable == 0){
    freeproc(p);
    release(&p->lock);
    return 0;
  }

 

  // Set up new context to start executing at forkret,
  // which returns to user space.
  memset(&p->context, 0, sizeof(p->context));
  p->context.ra = (uint64)forkret;
  p->context.sp = p->kstack + PGSIZE;

  return p;
```

分配的过程从`found`标志后开始

发现一个问题：为什么没有分配内存给`TRAMPOLINE`？

回顾`proc_pagetable`的注释，

```c
// Create a user page table for a given process,
// with no user memory, but with trampoline pages.
pagetable_t
proc_pagetable(struct proc *p)
{
  pagetable_t pagetable;

  // An empty page table.
  // ... do something ...

  // map the trampoline code (for system call return)
  // at the highest user virtual address.
  // only the supervisor uses it, on the way
  // to/from user space, so not PTE_U.
  if(mappages(pagetable, TRAMPOLINE, PGSIZE,
              (uint64)trampoline, PTE_R | PTE_X) < 0){
    uvmfree(pagetable, 0);
    return 0;
  }

  // map the trapframe just below TRAMPOLINE, for trampoline.S.
  // ... do something ...

  //map the usyscall just below TRAMPOLINE and TRAPFRAME
  // ... do something ...
    
  return pagetable;
}
```



在解决这个疑问之后，我们照葫芦画瓢对`USYSCALL`分配内存

* 使用`kalloc()`分配内存
	* Allocate one 4096-byte page of physical memory.
	* Returns a pointer that the kernel can use.
	* Returns 0 if the memory cannot be allocated.
* 如果分配失败则调用`freeproc(p)`

```c
// Look in the process table for an UNUSED proc.
// If found, initialize state required to run in the kernel,
// and return with p->lock held.
// If there are no free procs, or a memory allocation fails, return 0.
static struct proc*
allocproc(void)
{
  struct proc *p;

  for(p = proc; p < &proc[NPROC]; p++) {
    acquire(&p->lock);
    if(p->state == UNUSED) {
      goto found;
    } else {
      release(&p->lock);
    }
  }

  return 0;

found:
  p->pid = allocpid();
  p->state = USED;

  // Allocate a trapframe page.
  if((p->trapframe = (struct trapframe *)kalloc()) == 0){
    freeproc(p);
    release(&p->lock);
    return 0;
  }

  //Allocate a USYSCALL page.
  if((p->usyscall = (struct usyscall*)kalloc()) == 0) {
	  freeproc(p);
	  release(&p->lock);
	  return 0;
  }

  // An empty user page table.
  p->pagetable = proc_pagetable(p);
  if(p->pagetable == 0){
    freeproc(p);
    release(&p->lock);
    return 0;
  }

 

  // Set up new context to start executing at forkret,
  // which returns to user space.
  memset(&p->context, 0, sizeof(p->context));
  p->context.ra = (uint64)forkret;
  p->context.sp = p->kstack + PGSIZE;

  p->usyscall->pid = p->pid;

  return p;
}
```



既然要对USYSCALL的内存管理，那必然要有这个玩意才行，在`struct proc`添加`struct usyscall*`

```c
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

  struct usyscall* usyscall;   // to spped up user's syscall to avoid
                               // switch to kernel, likes ugetpid;
};

```

同样的在`freeproc`中添加对`struct usyscall*`的处理

模仿`trapframe`的处理即可

```c
// free a proc structure and the data hanging from it,
// including user pages.
// p->lock must be held.
static void
freeproc(struct proc *p)
{
  if(p->trapframe)
    kfree((void*)p->trapframe);
  p->trapframe = 0;
    
  //free usyscall
  if(p->usyscall)	
    kfree((void*)p->usyscall);
  p->usyscall = 0;

  if(p->pagetable)
    proc_freepagetable(p->pagetable, p->sz);
  
  p->pagetable = 0;
  p->sz = 0;
  p->pid = 0;
  p->parent = 0;
  p->name[0] = 0;
  p->chan = 0;
  p->killed = 0;
  p->xstate = 0;
  p->state = UNUSED;
}
```

## 调用关系

main->userinit->allocproc->kalloc

​				->proc_pagetable

## 更多加速

Which other xv6 system call(s) could be made faster using this shared page? Explain how.

* Must have no side-effects 
* Returns constant value while process runs 
* But value can change after entering kernel (e.g., ticks) 

Q: Which system calls in xv6?

* Getpid() – constant value, doesn’t change 
* Uptime() – constant until next tick 
* Each tick triggers a kernel interrupt, can update value 
* Fstat() – maybe possible, not likely worth it, too much state
* Fork()?

怎么确定有没有加速？

## How does Linux use USYSCALL?(VDSO)

![image-20220428101214998](https://s2.loli.net/2022/04/28/p7O8JotxeWVG1Aa.png)



Linux VDSO methods

* clock_gettime()
* getcpu()
* getpid()
* getppide()
* gettimeofday()
* set_tid_address()

