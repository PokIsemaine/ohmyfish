# Lab3 page table

## Detecting which pages have been accessed

Some garbage collectors (a form of automatic memory management) can benefit from information about which pages have been accessed (read or write). In this part of the lab, you will add a new feature to xv6 that detects and reports this information to userspace by inspecting the access bits in the RISC-V page table. The RISC-V hardware page walker marks these bits in the PTE whenever it resolves a TLB miss.

Your job is to implement `pgaccess()`, a system call that reports which pages have been accessed. The system call takes three arguments. First, it takes the starting virtual address of the first user page to check. Second, it takes the number of pages to check. Finally, it takes a user address to a buffer to store the results into a bitmask (a datastructure that uses one bit per page and where the first page corresponds to the least significant bit). You will receive full credit for this part of the lab if the `pgaccess` test case passes when running `pgtbltest`.

Some hints:

- Start by implementing `sys_pgaccess()` in `kernel/sysproc.c`.
- You'll need to parse arguments using `argaddr()` and `argint()`.
- For the output bitmask, it's easier to store a temporary buffer in the kernel and copy it to the user (via `copyout()`) after filling it with the right bits.
- It's okay to set an upper limit on the number of pages that can be scanned.
- `walk()` in `kernel/vm.c` is very useful for finding the right PTEs.
- You'll need to define `PTE_A`, the access bit, in `kernel/riscv.h`. Consult the RISC-V manual to determine its value.
- Be sure to clear `PTE_A` after checking if it is set. Otherwise, it won't be possible to determine if the page was accessed since the last time `pgaccess()` was called (i.e., the bit will be set forever).
- `vmprint()` may come in handy to debug page tables.



## 题意

一些GC（垃圾回收)可以受益于哪些页面已经被访问（读取或写入）的信息。在这个实验中，你需要给 xv6 添加一个新功能，该功能通过检查 RISC-V 页表中的访问位来检测并向用户空间报告此信息。当`TLB`没有命中时，`RISC-V`的 `hardware page walker`会在`PTE`中标记这些位。

你需要实现`pgaccess()`来报告被访问的页。

## 过程

### step1 了解 pgaccess_test

首先看`pgtbltest.c`里的`pgaccess_test()`

```c
void
pgaccess_test()
{
  char *buf;
  unsigned int abits;
  printf("pgaccess_test starting\n");
  testname = "pgaccess_test";
  buf = malloc(32 * PGSIZE);
  if (pgaccess(buf, 32, &abits) < 0)
    err("pgaccess failed");
  buf[PGSIZE * 1] += 1;
  buf[PGSIZE * 2] += 1;
  buf[PGSIZE * 30] += 1;
  if (pgaccess(buf, 32, &abits) < 0)
    err("pgaccess failed");
  if (abits != ((1 << 1) | (1 << 2) | (1 << 30)))
    err("incorrect access bits set");
  free(buf);
  printf("pgaccess_test: OK\n");
}

```



### step2 观察 pgacess进行的系统调用

The system call takes three arguments. 

* First, it takes the starting virtual address of the first user page to check. 
* Second, it takes the number of pages to check. 
* Finally, it takes a user address to a buffer to store the results into a bitmask (a datastructure that uses one bit per page and where the first page corresponds to the least significant bit). 



`user.h`中可以看到`pgaccess`的声明

```c
int pgaccess(void *base, int len, void *mask);
//void *base 第一个用户页面的起始虚拟地址
//int len 检查的页数
//void * mask 将结果存储到一个位掩码，每页使用一个为并且第一页对应最低有效位
```

`pgaccess`通过`entry.S`和`syscall`进行系统调用，调用`sys_pgaccess`



`sysproc.c`中

```c
#ifdef LAB_PGTBL
int
sys_pgaccess(void)
{
  // lab pgtbl: your code here.
  return 0;
}
#endif
```



### step3 获取参数

```c
// #define LAB_PGTBL
#ifdef LAB_PGTBL
pte_t * walk(pagetable_t pagetable, uint64 va, int alloc);	//because walk() in vm.c
int
sys_pgaccess(void)
{
  int len;
  uint64 base,abits;
  if(argaddr(0,&base) < 0)
	return -1;
  if(argint(1,&len) < 0)
	return -1;
  if(argaddr(2, &abits) < 0)
	return -1;
  
  if(len > 32 || len < 0) 
	return -1;

  // lab pgtbl: your code here.
  return 0;
}
#endif
```



### step4 实现 sys_pgaccess

```C
// #define LAB_PGTBL
#ifdef LAB_PGTBL
pte_t * walk(pagetable_t pagetable, uint64 va, int alloc);	//because walk() in vm.c
int
sys_pgaccess(void)
{
  int len;
  uint64 base,abits;
  if(argaddr(0,&base) < 0)
	return -1;
  if(argint(1,&len) < 0)
	return -1;
  if(argaddr(2, &abits) < 0)
	return -1;
  
  if(len > 32 || len < 0) 
	return -1;
  struct proc* p = myproc();

  uint64 mask = 0;
  for(int i = 0 ; i < len ; ++i) {
	  uint64 pg = base + i * PGSIZE;
	  pte_t* pte = walk(p->pagetable, pg ,0);
	  if(*pte & PTE_A) {
	    mask |= (1<<i);
		*pte = (*pte) & (~PTE_A); //clear PTE_A
	  }
  }
  
  copyout(p->pagetable, abits, (char*)&mask,sizeof(mask));
  // lab pgtbl: your code here.
  return 0;
}
#endif
```

### step4 定义 PTE_A

![image-20220428092855643](https://s2.loli.net/2022/04/28/h1qGeWPCU6ZMF5y.png)

由图可知`PTE_A`在第6位，在`risc.h`中定义`PTE_A`

```c
#define PTE_V (1L << 0) // valid
#define PTE_R (1L << 1)
#define PTE_W (1L << 2)
#define PTE_X (1L << 3)
#define PTE_U (1L << 4) // 1 -> user can access
#define PTE_A (1L << 6)	// access, RTFM of RISC-V
```

