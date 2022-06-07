# Lab: Copy-on-Write Fork for xv6

## The problem

The fork() system call in xv6 copies all of the parent process's user-space memory into the child. If the parent is large, copying can take a long time. Worse, the work is often largely wasted; for example, a fork() followed by exec() in the child will cause the child to discard the copied memory, probably without ever using most of it. On the other hand, if both parent and child use a page, and one or both writes it, a copy is truly needed.

## The solution

The goal of copy-on-write (COW) fork() is to defer allocating and copying physical memory pages for the child until the copies are actually needed, if ever.

COW fork() creates just a pagetable for the child, with PTEs for user memory pointing to the parent's physical pages. COW fork() marks all the user PTEs in both parent and child as not writable. When either process tries to write one of these COW pages, the CPU will force a page fault. The kernel page-fault handler detects this case, allocates a page of physical memory for the faulting process, copies the original page into the new page, and modifies the relevant PTE in the faulting process to refer to the new page, this time with the PTE marked writeable. When the page fault handler returns, the user process will be able to write its copy of the page.

COW fork() makes freeing of the physical pages that implement user memory a little trickier. A given physical page may be referred to by multiple processes' page tables, and should be freed only when the last reference disappears.

## Implement copy-on write([hard](https://pdos.csail.mit.edu/6.S081/2021/labs/guidance.html))



Your task is to implement copy-on-write fork in the xv6 kernel. You are done if your modified kernel executes both the cowtest and usertests programs successfully.

To help you test your implementation, we've provided an xv6 program called cowtest (source in user/cowtest.c). cowtest runs various tests, but even the first will fail on unmodified xv6. Thus, initially, you will see:

```
$ cowtest
simple: fork() failed
$ 
```

The "simple" test allocates more than half of available physical memory, and then fork()s. The fork fails because there is not enough free physical memory to give the child a complete copy of the parent's memory.

When you are done, your kernel should pass all the tests in both cowtest and usertests. That is:

```
$ cowtest
simple: ok
simple: ok
three: zombie!
ok
three: zombie!
ok
three: zombie!
ok
file: ok
ALL COW TESTS PASSED
$ usertests
...
ALL TESTS PASSED
$
```

Here's a reasonable plan of attack.

1. Modify uvmcopy() to map the parent's physical pages into the child, instead of allocating new pages. Clear `PTE_W` in the PTEs of both child and parent.
2. Modify usertrap() to recognize page faults. When a page-fault occurs on a COW page, allocate a new page with kalloc(), copy the old page to the new page, and install the new page in the PTE with `PTE_W` set.
3. Ensure that each physical page is freed when the last PTE reference to it goes away -- but not before. A good way to do this is to keep, for each physical page, a "reference count" of the number of user page tables that refer to that page. Set a page's reference count to one when `kalloc()` allocates it. Increment a page's reference count when fork causes a child to share the page, and decrement a page's count each time any process drops the page from its page table. `kfree()` should only place a page back on the free list if its reference count is zero. It's OK to to keep these counts in a fixed-size array of integers. You'll have to work out a scheme for how to index the array and how to choose its size. For example, you could index the array with the page's physical address divided by 4096, and give the array a number of elements equal to highest physical address of any page placed on the free list by `kinit()` in kalloc.c.
4. Modify copyout() to use the same scheme as page faults when it encounters a COW page.

Some hints:

- The lazy page allocation lab has likely made you familiar with much of the xv6 kernel code that's relevant for copy-on-write. However, you should not base this lab on your lazy allocation solution; instead, please start with a fresh copy of xv6 as directed above.
- It may be useful to have a way to record, for each PTE, whether it is a COW mapping. You can use the RSW (reserved for software) bits in the RISC-V PTE for this.
- `usertests` explores scenarios that `cowtest` does not test, so don't forget to check that all tests pass for both.
- Some helpful macros and definitions for page table flags are at the end of `kernel/riscv.h`.
- If a COW page fault occurs and there's no free memory, the process should be killed.





## 过程

实现写时复制的`fork()`：推迟为子进程分配和复制物理内存页面，直到实际需要副本

`COW fork()` 只为子级创建一个页表，用户内存的 `PTE` 指向父级的物理页面。`COW fork()` 将 `parent` 和 `child` 中的所有用户 `PTE` 标记为不可写。当任一进程尝试写入这些 `COW` 页之一时，CPU 将强制发生页错误。内核页面错误处理程序检测到这种情况，为出错进程分配物理内存页面，将原始页面复制到新页面中，并修改出错进程中的相关 `PTE` 以引用新页面，这次使用`PTE` 标记为可写。当页面错误处理程序返回时，用户进程将能够写入它的页面副本。

`COW fork()` 使实现用户内存的物理页面的释放变得有点棘手。一个给定的物理页可以被多个进程的页表引用，并且只有在最后一个引用消失时才应该被释放。



### 测试文件

`cowtest`以及`usertests`，下面展示`cowtest`

```c
//
// tests for copy-on-write fork() assignment.
//

#include "kernel/types.h"
#include "kernel/memlayout.h"
#include "user/user.h"

// allocate more than half of physical memory,
// then fork. this will fail in the default
// kernel, which does not support copy-on-write.
void
simpletest()
{
  uint64 phys_size = PHYSTOP - KERNBASE;
  int sz = (phys_size / 3) * 2;

  printf("simple: ");
  
  char *p = sbrk(sz);
  if(p == (char*)0xffffffffffffffffL){
    printf("sbrk(%d) failed\n", sz);
    exit(-1);
  }

  for(char *q = p; q < p + sz; q += 4096){
    *(int*)q = getpid();
  }

  int pid = fork();
  if(pid < 0){
    printf("fork() failed\n");
    exit(-1);
  }

  if(pid == 0)
    exit(0);

  wait(0);

  if(sbrk(-sz) == (char*)0xffffffffffffffffL){
    printf("sbrk(-%d) failed\n", sz);
    exit(-1);
  }

  printf("ok\n");
}

// three processes all write COW memory.
// this causes more than half of physical memory
// to be allocated, so it also checks whether
// copied pages are freed.
void
threetest()
{
  uint64 phys_size = PHYSTOP - KERNBASE;
  int sz = phys_size / 4;
  int pid1, pid2;

  printf("three: ");
  
  char *p = sbrk(sz);
  if(p == (char*)0xffffffffffffffffL){
    printf("sbrk(%d) failed\n", sz);
    exit(-1);
  }

  pid1 = fork();
  if(pid1 < 0){
    printf("fork failed\n");
    exit(-1);
  }
  if(pid1 == 0){
    pid2 = fork();
    if(pid2 < 0){
      printf("fork failed");
      exit(-1);
    }
    if(pid2 == 0){
      for(char *q = p; q < p + (sz/5)*4; q += 4096){
        *(int*)q = getpid();
      }
      for(char *q = p; q < p + (sz/5)*4; q += 4096){
        if(*(int*)q != getpid()){
          printf("wrong content\n");
          exit(-1);
        }
      }
      exit(-1);
    }
    for(char *q = p; q < p + (sz/2); q += 4096){
      *(int*)q = 9999;
    }
    exit(0);
  }

  for(char *q = p; q < p + sz; q += 4096){
    *(int*)q = getpid();
  }

  wait(0);

  sleep(1);

  for(char *q = p; q < p + sz; q += 4096){
    if(*(int*)q != getpid()){
      printf("wrong content\n");
      exit(-1);
    }
  }

  if(sbrk(-sz) == (char*)0xffffffffffffffffL){
    printf("sbrk(-%d) failed\n", sz);
    exit(-1);
  }

  printf("ok\n");
}

char junk1[4096];
int fds[2];
char junk2[4096];
char buf[4096];
char junk3[4096];

// test whether copyout() simulates COW faults.
void
filetest()
{
  printf("file: ");
  
  buf[0] = 99;

  for(int i = 0; i < 4; i++){
    if(pipe(fds) != 0){
      printf("pipe() failed\n");
      exit(-1);
    }
    int pid = fork();
    if(pid < 0){
      printf("fork failed\n");
      exit(-1);
    }
    if(pid == 0){
      sleep(1);
      if(read(fds[0], buf, sizeof(i)) != sizeof(i)){
        printf("error: read failed\n");
        exit(1);
      }
      sleep(1);
      int j = *(int*)buf;
      if(j != i){
        printf("error: read the wrong value\n");
        exit(1);
      }
      exit(0);
    }
    if(write(fds[1], &i, sizeof(i)) != sizeof(i)){
      printf("error: write failed\n");
      exit(-1);
    }
  }

  int xstatus = 0;
  for(int i = 0; i < 4; i++) {
    wait(&xstatus);
    if(xstatus != 0) {
      exit(1);
    }
  }

  if(buf[0] != 99){
    printf("error: child overwrote parent\n");
    exit(1);
  }

  printf("ok\n");
}

int
main(int argc, char *argv[])
{
  simpletest();

  // check that the first simpletest() freed the physical memory.
  simpletest();

  threetest();
  threetest();
  threetest();

  filetest();

  printf("ALL COW TESTS PASSED\n");

  exit(0);
}

```

`simpletest`：分配超过一半大小的物理内存，然后 `fork`。如果没有`COW`就会失败

`threetest`：三个进程都写 `COW memory`，这会造成超过一半的物理内存被分配，所以会检查是否有拷贝页面被释放

`filetest`：测试 `copyout()` 是否模拟 COW 故障。

### step1 查看现在的 fork

`proc.c/fork`

```c
// Create a new process, copying the parent.
// Sets up child kernel stack to return as if from fork() system call.
int
fork(void)
{
  int i, pid;
  struct proc *np;
  struct proc *p = myproc();

  // Allocate process.
  if((np = allocproc()) == 0){
    return -1;
  }

  // Copy user memory from parent to child.    <----------------Here
  if(uvmcopy(p->pagetable, np->pagetable, p->sz) < 0){
    freeproc(np);
    release(&np->lock);
    return -1;
  }
  np->sz = p->sz;

  // copy saved user registers.
  *(np->trapframe) = *(p->trapframe);

  // Cause fork to return 0 in the child.
  np->trapframe->a0 = 0;

  // increment reference counts on open file descriptors.
  for(i = 0; i < NOFILE; i++)
    if(p->ofile[i])
      np->ofile[i] = filedup(p->ofile[i]);
  np->cwd = idup(p->cwd);

  safestrcpy(np->name, p->name, sizeof(p->name));

  pid = np->pid;

  release(&np->lock);

  acquire(&wait_lock);
  np->parent = p;
  release(&wait_lock);

  acquire(&np->lock);
  np->state = RUNNABLE;
  release(&np->lock);

  return pid;
}
```



### step2 修改uvmcopy

修改 `uvmcopy()` 以将父级的物理页面映射到子级，而不是分配新页面。在子级和父级的 `PTE` 中清除`PTE_W`。

原来的`uvmcopy()`

```c
// Given a parent process's page table, copy
// its memory into a child's page table.
// Copies both the page table and the
// physical memory.
// returns 0 on success, -1 on failure.
// frees any allocated pages on failure.
int
uvmcopy(pagetable_t old, pagetable_t new, uint64 sz)
{
  pte_t *pte;
  uint64 pa, i;
  uint flags;
  char *mem;

  for(i = 0; i < sz; i += PGSIZE){
    if((pte = walk(old, i, 0)) == 0)	//找到 parent 进程的第 i 个页表项的地址
      panic("uvmcopy: pte should exist");
    if((*pte & PTE_V) == 0)				// 检查找到的页表项是否有效
      panic("uvmcopy: page not present");
    pa = PTE2PA(*pte);					// 得到页表项的物理地址
    flags = PTE_FLAGS(*pte);			// 得到页表项的 FLAGS
    if((mem = kalloc()) == 0)			// 使用 kalloc() 分配一个物理页
      goto err;
    memmove(mem, (char*)pa, PGSIZE);	// 从 parent 进程的页表项的物理地址拷贝到新分配的 mem
 //child pagetable 虚拟地址 i 映射到物理页 mem
    if(mappages(new, i, PGSIZE, (uint64)mem, flags) != 0){	
      kfree(mem);
      goto err;
    }
  }
  return 0;

 err:
  uvmunmap(new, 0, i / PGSIZE, 1);
  return -1;
}
```



不分配新页面，清除`PTE_W`

```c
int
uvmcopy(pagetable_t old, pagetable_t new, uint64 sz)
{
  pte_t *pte;
  uint64 pa, i;
  uint flags;
  char *mem;

  for(i = 0; i < sz; i += PGSIZE){
    if((pte = walk(old, i, 0)) == 0)
      panic("uvmcopy: pte should exist");
    if((*pte & PTE_V) == 0)
      panic("uvmcopy: page not present");
    pa = PTE2PA(*pte);
    flags = PTE_FLAGS(*pte);
	flags &= ~PTE_W;
    // if((mem = kalloc()) == 0)
    //   goto err;
    // memmove(mem, (char*)pa, PGSIZE);
    if(mappages(new, i, PGSIZE, (uint64)pa, flags) != 0){
    //   kfree(mem);
      goto err;
    }
  }
  return 0;

 err:
  uvmunmap(new, 0, i / PGSIZE, 1);
  return -1;
}
```



### step3 修改 usertrap

修改` usertrap() `以识别页面错误。当 `COW` 页面发生缺页时，使用` kalloc()` 分配新页面，将旧页面复制到新页面，并将新页面安装到 `PTE` 中并设置`PTE_W` 。
