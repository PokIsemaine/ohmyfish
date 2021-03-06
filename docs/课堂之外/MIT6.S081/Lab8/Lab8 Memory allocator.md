# Lab8 Memory allocator

## 要求

The program user/kalloctest stresses xv6's memory allocator: three processes grow and shrink their address spaces, resulting in many calls to `kalloc` and `kfree`. `kalloc` and `kfree` obtain `kmem.lock`. kalloctest prints (as "#fetch-and-add") the number of loop iterations in `acquire` due to attempts to acquire a lock that another core already holds, for the `kmem` lock and a few other locks. The number of loop iterations in `acquire` is a rough measure of lock contention. The output of `kalloctest` looks similar to this before you complete the lab:

```bash
$ kalloctest
start test1
test1 results:
--- lock kmem/bcache stats
lock: kmem: #fetch-and-add 83375 #acquire() 433015
lock: bcache: #fetch-and-add 0 #acquire() 1260
--- top 5 contended locks:
lock: kmem: #fetch-and-add 83375 #acquire() 433015
lock: proc: #fetch-and-add 23737 #acquire() 130718
lock: virtio_disk: #fetch-and-add 11159 #acquire() 114
lock: proc: #fetch-and-add 5937 #acquire() 130786
lock: proc: #fetch-and-add 4080 #acquire() 130786
tot= 83375
test1 FAIL
```

`acquire` maintains, for each lock, the count of calls to `acquire` for that lock, and the number of times the loop in `acquire` tried but failed to set the lock. kalloctest calls a system call that causes the kernel to print those counts for the kmem and bcache locks (which are the focus of this lab) and for the 5 most contended locks. If there is lock contention the number of `acquire` loop iterations will be large. The system call returns the sum of the number of loop iterations for the kmem and bcache locks.

For this lab, you must use a dedicated unloaded machine with multiple cores. If you use a machine that is doing other things, the counts that kalloctest prints will be nonsense. You can use a dedicated Athena workstation, or your own laptop, but don't use a dialup machine.

The root cause of lock contention in kalloctest is that `kalloc()` has a single free list, protected by a single lock. To remove lock contention, you will have to redesign the memory allocator to avoid a single lock and list. The basic idea is to maintain a free list per CPU, each list with its own lock. Allocations and frees on different CPUs can run in parallel, because each CPU will operate on a different list. The main challenge will be to deal with the case in which one CPU's free list is empty, but another CPU's list has free memory; in that case, the one CPU must "steal" part of the other CPU's free list. Stealing may introduce lock contention, but that will hopefully be infrequent.

Your job is to implement per-CPU freelists, and stealing when a CPU's free list is empty. You must give all of your locks names that start with "kmem". That is, you should call `initlock` for each of your locks, and pass a name that starts with "kmem". Run kalloctest to see if your implementation has reduced lock contention. To check that it can still allocate all of memory, run `usertests sbrkmuch`. Your output will look similar to that shown below, with much-reduced contention in total on kmem locks, although the specific numbers will differ. Make sure all tests in `usertests` pass. `make grade` should say that the kalloctests pass.

```bash
$ kalloctest
start test1
test1 results:
--- lock kmem/bcache stats
lock: kmem: #fetch-and-add 0 #acquire() 42843
lock: kmem: #fetch-and-add 0 #acquire() 198674
lock: kmem: #fetch-and-add 0 #acquire() 191534
lock: bcache: #fetch-and-add 0 #acquire() 1242
--- top 5 contended locks:
lock: proc: #fetch-and-add 43861 #acquire() 117281
lock: virtio_disk: #fetch-and-add 5347 #acquire() 114
lock: proc: #fetch-and-add 4856 #acquire() 117312
lock: proc: #fetch-and-add 4168 #acquire() 117316
lock: proc: #fetch-and-add 2797 #acquire() 117266
tot= 0
test1 OK
start test2
total free number of pages: 32499 (out of 32768)
.....
test2 OK
$ usertests sbrkmuch
usertests starting
test sbrkmuch: OK
ALL TESTS PASSED
$ usertests
...
ALL TESTS PASSED
$
```

Some hints:

- You can use the constant `NCPU` from kernel/param.h
- Let `freerange` give all free memory to the CPU running `freerange`.
- The function `cpuid` returns the current core number, but it's only safe to call it and use its result when interrupts are turned off. You should use `push_off()` and `pop_off()` to turn interrupts off and on.
- Have a look at the `snprintf` function in kernel/sprintf.c for string formatting ideas. It is OK to just name all locks "kmem" though.



## 解决方案

### 原版

![img](http://xv6.dgs.zone/tranlate_books/book-riscv-rev1/images/c6/p1.png)

```c
// Physical memory allocator, for user processes,
// kernel stacks, page-table pages,
// and pipe buffers. Allocates whole 4096-byte pages.

#include "types.h"
#include "param.h"
#include "memlayout.h"
#include "spinlock.h"
#include "riscv.h"
#include "defs.h"

void freerange(void *pa_start, void *pa_end);

extern char end[]; // first address after kernel.
                   // defined by kernel.ld.

struct run {
  struct run *next;
};

struct {
  struct spinlock lock;
  struct run *freelist;
} kmem;

void
kinit()
{
  initlock(&kmem.lock, "kmem");
  freerange(end, (void*)PHYSTOP);
}

void
freerange(void *pa_start, void *pa_end)
{
  char *p;
  p = (char*)PGROUNDUP((uint64)pa_start);
  for(; p + PGSIZE <= (char*)pa_end; p += PGSIZE)
    kfree(p);
}

// Free the page of physical memory pointed at by v,
// which normally should have been returned by a
// call to kalloc().  (The exception is when
// initializing the allocator; see kinit above.)
void
kfree(void *pa)
{
  struct run *r;

  if(((uint64)pa % PGSIZE) != 0 || (char*)pa < end || (uint64)pa >= PHYSTOP)
    panic("kfree");

  // Fill with junk to catch dangling refs.
  memset(pa, 1, PGSIZE);

  r = (struct run*)pa;

  acquire(&kmem.lock);
  r->next = kmem.freelist;
  kmem.freelist = r;
  release(&kmem.lock);
}

// Allocate one 4096-byte page of physical memory.
// Returns a pointer that the kernel can use.
// Returns 0 if the memory cannot be allocated.
void *
kalloc(void)
{
  struct run *r;

  acquire(&kmem.lock);
  r = kmem.freelist;
  if(r)
    kmem.freelist = r->next;
  release(&kmem.lock);

  if(r)
    memset((char*)r, 5, PGSIZE); // fill with junk
  return (void*)r;
}

```



### 性能分析

测试程序原理

```bash
xv6 kernel is booting

hart 2 starting
hart 1 starting
init: starting sh
$ ./kalloctest
start test1
test1 results:
--- lock kmem/bcache stats
lock: kmem: #test-and-set 110310 #acquire() 433016
lock: bcache: #test-and-set 0 #acquire() 342
--- top 5 contended locks:
lock: kmem: #test-and-set 110310 #acquire() 433016
lock: virtio_disk: #test-and-set 24554 #acquire() 57
lock: proc: #test-and-set 23316 #acquire() 469982
lock: proc: #test-and-set 19115 #acquire() 469986
lock: proc: #test-and-set 8556 #acquire() 469860
tot= 110310
test1 FAIL
start test2
total free number of pages: 32499 (out of 32768)
.....
test2 OK
```



### 优化策略

为每个 CPU 分配独立的 freelist



### 代码实现

```c
// Physical memory allocator, for user processes,
// kernel stacks, page-table pages,
// and pipe buffers. Allocates whole 4096-byte pages.

#include "types.h"
#include "param.h"
#include "memlayout.h"
#include "spinlock.h"
#include "riscv.h"
#include "defs.h"

void freerange(void *pa_start, void *pa_end);

extern char end[]; // first address after kernel.
                   // defined by kernel.ld.

struct run {
  struct run *next;
};

struct {
  struct spinlock lock;
  struct run *freelist;
} kmem [NCPU];

// NCPU = 8 , maximum number of CPUs

void
kinit()
{
  char lock_name[11];
  for(int i = 0; i < NCPU; ++i) {
    snprintf(lock_name, sizeof(lock_name), "kmem_cpu_%d", i);
    initlock(&kmem[i].lock, lock_name);
  }
  freerange(end, (void*)PHYSTOP);
}

void
freerange(void *pa_start, void *pa_end)
{
  char *p;
  p = (char*)PGROUNDUP((uint64)pa_start);
  for(; p + PGSIZE <= (char*)pa_end; p += PGSIZE)
    kfree(p);
}

// Free the page of physical memory pointed at by v,
// which normally should have been returned by a
// call to kalloc().  (The exception is when
// initializing the allocator; see kinit above.)
void
kfree(void *pa)
{
  struct run *r;

  if(((uint64)pa % PGSIZE) != 0 || (char*)pa < end || (uint64)pa >= PHYSTOP)
    panic("kfree");

  // Fill with junk to catch dangling refs.
  memset(pa, 1, PGSIZE);

  r = (struct run*)pa;
    
  push_off(); // 关中断 for cpuid() xv6 手册 7.4
  int cpu = cpuid();
  pop_off();  // 开中断

  acquire(&kmem[cpu].lock);
  r->next = kmem[cpu].freelist;
  kmem[cpu].freelist = r;
  release(&kmem[cpu].lock);
}

// Allocate one 4096-byte page of physical memory.
// Returns a pointer that the kernel can use.
// Returns 0 if the memory cannot be allocated.
void *
kalloc(void)
{
  struct run *r;
  
  push_off(); // 关中断 for cpuid() xv6 手册 7.4
  int cpu = cpuid();
  pop_off();  // 开中断

  acquire(&kmem[cpu].lock);

  r = kmem[cpu].freelist;
  if(r)
    kmem[cpu].freelist = r->next;
  else {
    // 当前 CPU 空闲内存耗尽，从其他 CPU 窃取
    for(int i = 0; i < NCPU; ++i) {
      // if(i != cpu && kmem[i].freelist) { kmem[i].freelist 这个可能有竞态条件，获取锁后再判断
      if(i != cpu) {
        acquire(&kmem[i].lock);
        
        r = kmem[i].freelist;
        if(r) {
          kmem[i].freelist = r->next;
          release(&kmem[i].lock);
          break;
        }
        
        release(&kmem[i].lock);
      }
    }
  }
  release(&kmem[cpu].lock);


  if(r)
    memset((char*)r, 5, PGSIZE); // fill with junk
  return (void*)r;
}

```

上面的代码已经可以通过测试了，但是 `kalloc`中存在竞态条件，隐含了死锁问题

当 CPU1 和 CPU2 想要互相偷的时候就会发生死锁

![image-20220713152547709](https://s2.loli.net/2022/07/13/sDLzUCHIkO5lZQA.png)

解决方案是在决定要偷其他 CPU 空闲之前把自己的锁先释放了（虽然这可能在短时间内造成重复偷页）

```c
// Allocate one 4096-byte page of physical memory.
// Returns a pointer that the kernel can use.
// Returns 0 if the memory cannot be allocated.
void *
kalloc(void)
{
  struct run *r;
  
   
  push_off(); // 关中断 for cpuid() xv6 手册 7.4
  int cpu = cpuid();
  pop_off();  // 开中断


  acquire(&kmem[cpu].lock);

  r = kmem[cpu].freelist;
  if(r) {
    kmem[cpu].freelist = r->next;
  }
  release(&kmem[cpu].lock);	// 窃取内存前释放当前锁，防止环路死锁

  if(!r) {
    // 当前 CPU 空闲内存耗尽，从其他 CPU 窃取
    for(int i = 0; i < NCPU; ++i) {
      // if(i != cpu && kmem[i].freelist) { kmem[i].freelist 这个可能有竞态条件，获取锁后再判断
      if(i != cpu) {
        acquire(&kmem[i].lock);
        r = kmem[i].freelist;
        if(r) {
          kmem[i].freelist = r->next;
          release(&kmem[i].lock);
          break;
        }
        release(&kmem[i].lock);
      }
    }
  }


  if(r)
    memset((char*)r, 5, PGSIZE); // fill with junk
  return (void*)r;
}

```



### 优化效果

```bash
$ ./kalloctest
start test1
test1 results:
--- lock kmem/bcache stats
lock: bcache: #test-and-set 0 #acquire() 1250
--- top 5 contended locks:
lock: virtio_disk: #test-and-set 67746 #acquire() 114
lock: proc: #test-and-set 48083 #acquire() 708057
lock: proc: #test-and-set 27594 #acquire() 708125
lock: proc: #test-and-set 22654 #acquire() 708128
lock: proc: #test-and-set 20577 #acquire() 708056
tot= 0
test1 OK
start test2
total free number of pages: 32499 (out of 32768)
.....
test2 OK
```



## 进一步的探讨

* 一次偷多个？（测！）
* 释放自己的锁可能会导致短时间内多个kalloc为同一个cpu重复偷页，影响大吗？（测！）
* 候补链？
* 鸵鸟算法，重启大法好！
* 为 xv6 提供死锁检测工具？
* OSTEP 第29章 并发链表 扩展链表、锁耦合、手锁
