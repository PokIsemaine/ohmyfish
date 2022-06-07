# Chapter 3 Page tables

![image-20220427101641527](https://s2.loli.net/2022/04/27/lPmRudUEWX4vApc.png)

![image-20220427101807660](https://s2.loli.net/2022/04/27/M3HaUo5KkJBPQEO.png)

* Page tables provide each process with its own **private address space and memory**
* Page tables determine **what memory addresses** mean, and **what part of physical memory can be accessed**

perform many tricks，eg. xv6：

* mapping **the same memory**(a trampoline page) in several address spaces
* guarding kernel and user stacks **with an unmapped page**

## 3.1 Paging hardware

### hardware

hardware：Sv39 RISC-V

* Sv39? Sv48?
* PTE = 44 + 10 = 54 bits != 64 bits ?

![image-20220421092218990](https://s2.loli.net/2022/04/21/QPlA8dOsuDYc1TR.png)



### VA to PA

a RISC-V CPU translates a virtual address into a physical in three steps

* contain the **physical addresses** for page-table pages in the **next level** of the tree

* A page table is stored in physical memory as **a three-level tree** (why?)
	
	![image-20220427100843427](https://s2.loli.net/2022/04/27/z1NAMj436LiF2ka.png)
	
	* good: allows a **memory-efficient way** of recording PTE (In the **common case** in which large ranges of virtual addresses have no mappings)
	* bad: the CPU must load three PTEs from memory to perform the translation of the virtual address in the load/store instruction to a physical address (TLB)
	
* If any of the three PTEs required to translate an address is not present, the paging hardware raises **a page-fault exception**, leaving it up to the kernel to handle the exception

* satp register
	* To tell the hardware to use a page table, the kernel must **write the physical address of the root page-table page into the satp register**. 
	* Each CPU has its own satp so that different CPUs can run different processes, each with a private address space described by its own page table.

![image-20220421092624549](https://s2.loli.net/2022/04/21/7dnv2DcquxN8RHj.png)



![image-20220427100816405](https://s2.loli.net/2022/04/27/awmB3H4ixKstfAv.png)

### PTE details

![image-20220421094855374](https://s2.loli.net/2022/04/21/D7LBt8XAW3J9rdZ.png)



![image-20220427101237907](https://s2.loli.net/2022/04/27/S2JoXrByZTCQgjV.png)

### TLB

the CPU must load three PTEs from memory to perform the translation of the virtual address in the load/store instruction to a physical address (TLB)

![image-20220427110610320](https://s2.loli.net/2022/04/27/XDH1yRF7cMQE4qr.png)

## 3.2 Kernel address space

![image-20220421095731028](https://s2.loli.net/2022/04/21/zcaOYHXW7x9AwkV.png)



Xv6 maintains one page table per process, describing each process’s user address space, plus a single page table that describes the kernel’s address space.

user address space page != kernel address space page -> why?

The kernel configures the layout of its address space to give itself access to physical memory and various hardware resources **at predictable virtual addresses.** 



### QEMU simulates

RAM [ KERNBASE : PHYSTP]

IO below 0x8000 0000

`memlayout.h`

```c
// Physical memory layout

// qemu -machine virt is set up like this,
// based on qemu's hw/riscv/virt.c:
//
// 00001000 -- boot ROM, provided by qemu
// 02000000 -- CLINT
// 0C000000 -- PLIC
// 10000000 -- uart0 
// 10001000 -- virtio disk 
// 80000000 -- boot ROM jumps here in machine mode
//             -kernel loads the kernel here
// unused RAM after 80000000.

// the kernel uses physical memory thus:
// 80000000 -- entry.S, then kernel text and data
// end -- start of kernel page allocation area
// PHYSTOP -- end RAM used by the kernel

// qemu puts UART registers here in physical memory.
#define UART0 0x10000000L
#define UART0_IRQ 10

// virtio mmio interface
#define VIRTIO0 0x10001000
#define VIRTIO0_IRQ 1

#ifdef LAB_NET
#define E1000_IRQ 33
#endif

// core local interruptor (CLINT), which contains the timer.
#define CLINT 0x2000000L
#define CLINT_MTIMECMP(hartid) (CLINT + 0x4000 + 8*(hartid))
#define CLINT_MTIME (CLINT + 0xBFF8) // cycles since boot.

// qemu puts platform-level interrupt controller (PLIC) here.
#define PLIC 0x0c000000L
#define PLIC_PRIORITY (PLIC + 0x0)
#define PLIC_PENDING (PLIC + 0x1000)
#define PLIC_MENABLE(hart) (PLIC + 0x2000 + (hart)*0x100)
#define PLIC_SENABLE(hart) (PLIC + 0x2080 + (hart)*0x100)
#define PLIC_MPRIORITY(hart) (PLIC + 0x200000 + (hart)*0x2000)
#define PLIC_SPRIORITY(hart) (PLIC + 0x201000 + (hart)*0x2000)
#define PLIC_MCLAIM(hart) (PLIC + 0x200004 + (hart)*0x2000)
#define PLIC_SCLAIM(hart) (PLIC + 0x201004 + (hart)*0x2000)

// the kernel expects there to be RAM
// for use by the kernel and user pages
// from physical address 0x80000000 to PHYSTOP.
#define KERNBASE 0x80000000L
#define PHYSTOP (KERNBASE + 128*1024*1024)

// map the trampoline page to the highest address,
// in both user and kernel space.
#define TRAMPOLINE (MAXVA - PGSIZE)

// map kernel stacks beneath the trampoline,
// each surrounded by invalid guard pages.
#define KSTACK(p) (TRAMPOLINE - (p)*2*PGSIZE - 3*PGSIZE)

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
#define TRAPFRAME (TRAMPOLINE - PGSIZE)

#ifdef LAB_PGTBL
#define USYSCALL (TRAPFRAME - PGSIZE)

struct usyscall {
  int pid;  // Process ID
};
#endif

```



### memory-mapped

![image-20220427101423507](https://s2.loli.net/2022/04/27/SeUR1oLsPahOTk6.png)

**direct mapping**

The kernel **gets at RAM and memory-mapped device registers** using “direct mapping” that is, mapping the resources at virtual addresses that are equal to the physical address

**indirect mapping**

* **The trampoline page**. It is mapped at the top of the virtual address space; user page tables have this same mapping.(R-X)
* **The kernel stack pages**. Each process has its own kernel stack, which is mapped high so that below it xv6 can leave an **unmapped** guard page.(R-W)



## 3.3 Code : create an address space

Most of the xv6 code for manipulating address spaces and page tables resides in` vm.c`

### the central data structre: pagetable_t

 The central data structure is `pagetable_t`, which is really **a pointer to a RISC-V  root page-table page**

```c
//vm.c
//the kernel's page table.
pagetable_t kernel_pagetable;


//riscv.h
typedef uint64 pte_t;
typedef uint64 *pagetable_t; // 512 PTEs
```



### the central function: walk

The central functions are walk , which **installs PTEs for new mappings**

1.  finds the PTE for a virtual address 

2. mappages 

​	

```c
// Return the address of the PTE in page table pagetable
// that corresponds to virtual address va.  If alloc!=0,
// create any required page-table pages.
//
// The risc-v Sv39 scheme has three levels of page-table
// pages. A page-table page contains 512 64-bit PTEs.
// A 64-bit virtual address is split into five fields:
//   39..63 -- must be zero.
//   30..38 -- 9 bits of level-2 index.
//   21..29 -- 9 bits of level-1 index.
//   12..20 -- 9 bits of level-0 index.
//    0..11 -- 12 bits of byte offset within the page.
pte_t *
walk(pagetable_t pagetable, uint64 va, int alloc)
{
  if(va >= MAXVA)
    panic("walk");

  for(int level = 2; level > 0; level--) {
    pte_t *pte = &pagetable[PX(level, va)];
    if(*pte & PTE_V) {
      pagetable = (pagetable_t)PTE2PA(*pte);
    } else {
      if(!alloc || (pagetable = (pde_t*)kalloc()) == 0)
        return 0;
      memset(pagetable, 0, PGSIZE);
      *pte = PA2PTE(pagetable) | PTE_V;
    }
  }
  return &pagetable[PX(0, va)];
}
```

![image-20220421092624549](https://s2.loli.net/2022/04/21/7dnv2DcquxN8RHj.png)

**The hardware does the level 3 page table lookup, so why do we have a walk function in XV6 that does the same job?**

First of all, the walk function in XV6 sets the initial page table, it needs to program the 3-level page table, so it first needs to be able to simulate the 3-level page table.

 Another reason may have been encountered in the syscall experiment. In XV6, the kernel has its own page table, and the user process also has its own page table. The pointer of the user process to the sys_info structure exists in the page table of user space, but The kernel needs to translate this pointer into a physical address that it can read and write. If you look at copy_in, copy_out, you can find that the kernel will translate the user's virtual address to get the physical address through the page table of the user process, so that the kernel can read and write the corresponding physical memory address. This is some of the reasons why the walk function is needed in XV6.



### functions starting with kvm

**Functions starting with kvm manipulate the kernel page table;**

| function                                                     | do                                                           |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `pagetable_t kvmmake(void);`                                 | Make a direct-map page table for the kernel.                 |
| `void kvminit(void);`                                        | Initialize the one kernel_pagetable                          |
| `pagetable_t kvmmake(void);`                                 | Make a direct-map page table for the kernel.                 |
| `void kvminit(void);`                                        | Initialize the one kernel_pagetable                          |
| `void kvminithart();`                                        | Switch h/w page table register to the kernel's page table, and enable paging. |
| `void kvmmap(pagetable_t kpgtbl, uint64 va, uint64 pa, uint64 sz, int perm);` | add a mapping to the kernel page table. only used when booting.does not flush TLB or enable paging. |

 

### functions starting with uvm

**Functions starting with uvm manipulate a user page table;** 

| function                                                     | do                                                           |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `void uvmunmap(pagetable_t pagetable, uint64 va, uint64 npages, int do_free);` | Remove npages of mappings starting from va. va must be page-aligned. The mappings must exist.Optionally free the physical memory. |
| `pagetable_t uvmcreate();`                                   | create an empty user page table. returns 0 if out of memory. |
| `void uvminit(pagetable_t pagetable, uchar *src, uint sz);`  | Load the user initcode into address 0 of pagetable, for the very first process. sz must be less than a page. |
| `uint64 uvmalloc(pagetable_t pagetable, uint64 oldsz, uint64 newsz);` | Allocate PTEs and physical memory to grow process from oldsz to newsz, which need not be page aligned.  Returns new size or 0 on error. |
| `uint64 uvmdealloc(pagetable_t pagetable, uint64 oldsz, uint64 newsz);` | Deallocate user pages to bring the process size from oldsz to newsz.  oldsz and newsz need not be page-aligned, nor does newsz need to be less than oldsz.  oldsz can be larger than the actual process size.  Returns the new process size. |
| `void uvmfree(pagetable_t pagetable, uint64 sz);`            | Free user memory pages, then free page-table pages.          |
| `int uvmcopy(pagetable_t old, pagetable_t new, uint64 sz);`  | Given a parent process's page table, copy its memory into a child's page table. Copies both the page table and the physical memory. returns 0 on success, -1 on failure. frees any allocated pages on failure. |
| `void uvmclear(pagetable_t pagetable, uint64 va);`           | mark a PTE invalid for user access. used by exec for the user stack guard page. |

### other functions

**other functions are used for both**

| function                                                     | do                                                           |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `uint64 walkaddr(pagetable_t pagetable, uint64 va);`         | Look up a virtual address, return the physical address, or 0 if not mapped. Can only be used to look up user pages. |
| `int mappages(pagetable_t pagetable, uint64 va, uint64 size, uint64 pa, int perm);` | Create PTEs for virtual addresses starting at va that refer to physical addresses starting at pa. va and size might not be page-aligned. Returns 0 on success, -1 if walk() couldn't allocate a needed page-table page. |
| `void freewalk(pagetable_t pagetable);`                      | Recursively free page-table pages. All leaf mappings must already have been removed. |



```c
int
mappages(pagetable_t pagetable, uint64 va, uint64 size, uint64 pa, int perm)
{
  uint64 a, last;
  pte_t *pte;

  if(size == 0)
    panic("mappages: size");
  
  a = PGROUNDDOWN(va);
  last = PGROUNDDOWN(va + size - 1);
  for(;;){
    if((pte = walk(pagetable, a, 1)) == 0)
      return -1;
    if(*pte & PTE_V)			//检查有没有重复映射
      panic("mappages: remap");
    *pte = PA2PTE(pa) | perm | PTE_V;
    if(a == last)
      break;
    a += PGSIZE;
    pa += PGSIZE;
  }
  return 0;
}
```







**about copy**

copyout and copyin copy data to and from user virtual addresses provided as system call arguments; they are in vm.c because they need to explicitly translate those addresses in order to find the corresponding physical memory

| function                                                     | do                                                           |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `int copyout(pagetable_t pagetable, uint64 dstva, char *src, uint64 len);` | Copy from kernel to user. Copy len bytes from src to virtual address dstva in a given page table. Return 0 on success, -1 on error. |
| `int copyin(pagetable_t pagetable, char *dst, uint64 srcva, uint64 len);` | Copy from user to kernel. Copy len bytes to dst from virtual address srcva in a given page table. Return 0 on success, -1 on error. |
| `int copyinstr(pagetable_t pagetable, char *ds, uint64 srcva, uint64 max);` | Copy a null-terminated string from user to kernel. Copy bytes to dst from virtual address srcva in a given page table, until a '\0', or max. Return 0 on success, -1 on error. |





### process

**step1**

`main` calls `kvminit`

```c
// start() jumps here in supervisor mode on all CPUs.
void
main()
{
  if(cpuid() == 0){
    //... do something...
    kinit();         // physical page allocator
    kvminit();       // create kernel page table   <-----------------------------------HERE!
    kvminithart();   // turn on paging
    //... do something...
  } else {
    while(started == 0)
      ;
    __sync_synchronize();
    printf("hart %d starting\n", cpuid());
    kvminithart();    // turn on paging
    trapinithart();   // install kernel trap vector
    plicinithart();   // ask PLIC for device interrupts
  }

  scheduler();        
}

// Initialize the one kernel_pagetable
void
kvminit(void)
{
  kernel_pagetable = kvmmake();
}
```



**step2**

`kvmmake` first allocates a page of physical memory to hold the root page-table page

```c
// Make a direct-map page table for the kernel.
pagetable_t
kvmmake(void)
{
  pagetable_t kpgtbl;

  kpgtbl = (pagetable_t) kalloc();
  memset(kpgtbl, 0, PGSIZE);

  //...do something...
  
  return kpgtbl;
}
```



**step3**

`kvmmake` calls `kvmmap` to install the **translations** that the kernel needs

```c
// Make a direct-map page table for the kernel.
pagetable_t
kvmmake(void)
{
  //...do something...

  // uart registers
  kvmmap(kpgtbl, UART0, UART0, PGSIZE, PTE_R | PTE_W);

  // virtio mmio disk interface
  kvmmap(kpgtbl, VIRTIO0, VIRTIO0, PGSIZE, PTE_R | PTE_W);

  // PLIC
  kvmmap(kpgtbl, PLIC, PLIC, 0x400000, PTE_R | PTE_W);

  // map kernel text executable and read-only.
  kvmmap(kpgtbl, KERNBASE, KERNBASE, (uint64)etext-KERNBASE, PTE_R | PTE_X);

  // map kernel data and the physical RAM we'll make use of.
  kvmmap(kpgtbl, (uint64)etext, (uint64)etext, PHYSTOP-(uint64)etext, PTE_R | PTE_W);

  // map the trampoline for trap entry/exit to
  // the highest virtual address in the kernel.
  kvmmap(kpgtbl, TRAMPOLINE, (uint64)trampoline, PGSIZE, PTE_R | PTE_X);

  //...do something...
    
  return kpgtbl;
}
```



some details about `kvmmap`( take  uart registers as an example)

```c
  // uart registers
  kvmmap(kpgtbl, UART0, UART0, PGSIZE, PTE_R | PTE_W);
```



```c
//memlayout.h

//riscv.h
#define PGSIZE 4096 // bytes per page
#define PGSHIFT 12  // bits of offset within a page

#define PGROUNDDOWN(a) (((a)) & ~(PGSIZE-1)) //为往下4k地址对齐，相当于可以把一个地址转换为比此地址低的4k地址。

#define PTE_V (1L << 0) // valid
#define PTE_R (1L << 1)
#define PTE_W (1L << 2)
#define PTE_X (1L << 3)
#define PTE_U (1L << 4) // 1 -> user can access

// shift a physical address to the right place for a PTE.
#define PA2PTE(pa) ((((uint64)pa) >> 12) << 10)
```



```c
kvmmap(kpgtbl, UART0, UART0, PGSIZE, PTE_R | PTE_W);
```



```c
// add a mapping to the kernel page table.
// only used when booting.
// does not flush TLB or enable paging.
void
kvmmap(pagetable_t kpgtbl, uint64 va, uint64 pa, uint64 sz, int perm)
{
  if(mappages(kpgtbl, va, sz, pa, perm) != 0)
    panic("kvmmap");
}

// Create PTEs for virtual addresses starting at va that refer to
// physical addresses starting at pa. va and size might not
// be page-aligned. Returns 0 on success, -1 if walk() couldn't
// allocate a needed page-table page.

//pagetable = kernel_pagetable
//va = UART0 = 0x10000000L
//size = PGSIZE = 4096 
// pa = UART0 = 0x100000L
//perm = PTE_R|PTE_W
int
mappages(pagetable_t pagetable, uint64 va, uint64 size, uint64 pa, int perm)
{
  uint64 a, last;
  pte_t *pte;

  if(size == 0)
    panic("mappages: size");
  a = PGROUNDDOWN(va);
  last = PGROUNDDOWN(va + size - 1);
  for(;;){
    if((pte = walk(pagetable, a, 1)) == 0)
      return -1;
    if(*pte & PTE_V)
      panic("mappages: remap");
    *pte = PA2PTE(pa) | perm | PTE_V;
    if(a == last)
      break;
    a += PGSIZE;
    pa += PGSIZE;
  }
  return 0;
}

// Return the address of the PTE in page table pagetable
// that corresponds to virtual address va.  If alloc!=0,
// create any required page-table pages.
//
// The risc-v Sv39 scheme has three levels of page-table
// pages. A page-table page contains 512 64-bit PTEs.
// A 64-bit virtual address is split into five fields:
//   39..63 -- must be zero.
//   30..38 -- 9 bits of level-2 index.
//   21..29 -- 9 bits of level-1 index.
//   12..20 -- 9 bits of level-0 index.
//    0..11 -- 12 bits of byte offset within the page.
pte_t *
walk(pagetable_t pagetable, uint64 va, int alloc)
{
  if(va >= MAXVA)
    panic("walk");

  for(int level = 2; level > 0; level--) {
    pte_t *pte = &pagetable[PX(level, va)];
    if(*pte & PTE_V) {
      pagetable = (pagetable_t)PTE2PA(*pte);
    } else {
      if(!alloc || (pagetable = (pde_t*)kalloc()) == 0)
        return 0;
      memset(pagetable, 0, PGSIZE);
      *pte = PA2PTE(pagetable) | PTE_V;
    }
  }
  return &pagetable[PX(0, va)];
}
```



**step4**

`proc_mapstacks`  allocates a kernel stack for each process.

```c
// Make a direct-map page table for the kernel.
pagetable_t
kvmmake(void)
{
  //...do something...
 
  // map kernel stacks
  proc_mapstacks(kpgtbl);
    
  return kpgtbl;
}

// Allocate a page for each process's kernel stack.
// Map it high in memory, followed by an invalid
// guard page.
void
proc_mapstacks(pagetable_t kpgtbl) {
  struct proc *p;
  
  for(p = proc; p < &proc[NPROC]; p++) {
    char *pa = kalloc();
    if(pa == 0)
      panic("kalloc");
    uint64 va = KSTACK((int) (p - proc));
    kvmmap(kpgtbl, va, (uint64)pa, PGSIZE, PTE_R | PTE_W);
  }
}
```



**step5**

`main`calls`kvminithart`  to install the kernel page table.

```c
// start() jumps here in supervisor mode on all CPUs.
void
main()
{
  if(cpuid() == 0){
    //... do something...
    kinit();         // physical page allocator
    kvminit();       // create kernel page table
    kvminithart();   // turn on paging   <-----------------------------------HERE!
    //... do something...
  } else {
    while(started == 0)
      ;
    __sync_synchronize();
    printf("hart %d starting\n", cpuid());
    kvminithart();    // turn on paging
    trapinithart();   // install kernel trap vector
    plicinithart();   // ask PLIC for device interrupts
  }

  scheduler();        
}

// Switch h/w page table register to the kernel's page table,
// and enable paging.
void
kvminithart()
{
  w_satp(MAKE_SATP(kernel_pagetable));//It writes the physical address of the root page-table page into the register satp. 
  sfence_vma();
}

// supervisor address translation and protection;
// holds the address of the page table.
static inline void 
w_satp(uint64 x)
{
  asm volatile("csrw satp, %0" : : "r" (x));
}

// flush the TLB.
static inline void
sfence_vma()
{
  // the zero, zero means flush all TLB entries.
  asm volatile("sfence.vma zero, zero");
}

```

After this the CPU will translate addresses using the kernel page table. Since the kernel uses an identity mapping, the now virtual address of the next instruction will map to the right physical memory address.



### Summary

how to create an address place

1. `main` call `kvminit` to initialize the one `kernel_pagetable`
2. `kvmmake` first allocates a page of physical memory to hold the root page-table page
3. `kvmmake` calls `kvmmap` to install the **translations** that the kernel needs
4. `proc_mapstacks`  allocates a kernel stack for each process.
5. `main`calls`kvminithart`  to install the kernel page table.



//TODO：更新成 PAD 图

![image-20220421160132180](https://s2.loli.net/2022/04/21/dbr1WGjtyOnKZ4J.png)

## 3.4 Physical memory allocation

xv6 uses the physical memory between the end of the kernel and PHYSTOP for run-time allocation.

![image-20220421161006932](https://s2.loli.net/2022/04/21/qfSLhKsAtrkV8mD.png)

* It allocates and frees whole 4096-byte pages **at a time**. 

* It keeps track of which pages are free by threading **a linked list** through the pages themselves.

	* allocation consists of **removing** a page from the linked list
	* freeing consists of **adding** the freed page to the list.

	

## 3.5  Code: Physical memory allocator

`kallo.c`

The allocator's data structure is `a free list` of physical memory pages for allocation

 Each free page’s list element is a `struct run `

The `free list` is protected by a `spin lock`

The list and the lock are wrapped in a struct to make clear that the lock protects the fields in the struct

```c
struct run {
  struct run *next;
};

struct {
  struct spinlock lock;
  struct run *freelist;
} kmem;
```



### main call kinit

```c
// start() jumps here in supervisor mode on all CPUs.
void
main()
{
  if(cpuid() == 0){
    //... do something...
    kinit();         // physical page allocator		<-------------------------HERE!
    kvminit();       // create kernel page table
    kvminithart();   // turn on paging   
    //... do something...
  } else {
    while(started == 0)
      ;
    __sync_synchronize();
    printf("hart %d starting\n", cpuid());
    kvminithart();    // turn on paging
    trapinithart();   // install kernel trap vector
    plicinithart();   // ask PLIC for device interrupts
  }

  scheduler();        
}

void
kinit()
{
  initlock(&kmem.lock, "kmem");
  freerange(end, (void*)PHYSTOP);
}

void
kinit()
{
  initlock(&kmem.lock, "kmem");
  freerange(end, (void*)PHYSTOP);//reats addresses as integers
}

void
freerange(void *pa_start, void *pa_end)
{
  char *p;
//A PTE can only refer to
//a physical address that is aligned on a 4096-byte boundary (is a multiple of 4096), so freerange
//uses PGROUNDUP to ensure that it frees only aligned physical addresses.
  p = (char*)PGROUNDUP((uint64)pa_start);
  for(; p + PGSIZE <= (char*)pa_end; p += PGSIZE)
    kfree(p);
}
```



 **Why is the allocator code is full of C type casts ?**

* The main reason: The allocator sometimes treats addresses as integers in order to perform arithmetic on them, and sometimes uses addresses as pointers to read and write memory . 
* The other reason is that freeing and allocation inherently change the type of the memory

`kfree`

```c
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
  memset(pa, 1, PGSIZE);	// hopefully that will cause such code to break faster

  r = (struct run*)pa;//reats addresses as pointer

  acquire(&kmem.lock);
  r->next = kmem.freelist;
  kmem.freelist = r;
  release(&kmem.lock);
}
```





## 3.6 Process address space

![image-20220421165446640](https://s2.loli.net/2022/04/21/mNkJzDSdC19bEG5.png)

When a process asks `xv6` for more user memory, `xv6` first uses `kalloc` to allocate physical pages. It then adds `PTEs` to the process’s page table that point to the new physical pages.`Xv6` sets the `PTE_W`, `PTE_X`, `PTE_R`, `PTE_U`, and `PTE_V` flags in these `PTEs`. Most processes do not use the entire user address space; `xv6` leaves `PTE_V` clear in unused `PTEs`.



Each process has a separate page table, and when xv6 switches between processes, it also changes page tables

 a process’s user memory starts at virtual address zero and can grow up to MAXVA , allowing a process to address in principle 256 Gigabytes of memory

![image-20220427101132964](https://s2.loli.net/2022/04/27/Z416u2eKmqXp8Ig.png)

We see here a few nice examples of use of page tables. 

* First, different processes’ page tables translate user addresses to different pages of physical memory, so that each **process has private user memory.**
* Second, each process sees its memory as having contiguous virtual addresses starting at zero, while the process’s **physical memory can be non-contiguous**.
* Third, **the kernel maps a page with trampoline code at the top of the user address space**, thus a single page of physical memory shows up in all address spaces



**command-line arguments**

```c
int 
main(int argc,char* argv[]) {

    return 0;
}
```

**guard page**

To detect a user stack overflowing the allocated stack memory, xv6 places an inaccessible guard page right below the stack by clearing the PTE_U flag



## 3.7 Code: sbrk

 ![image-20220427101038290](https://s2.loli.net/2022/04/27/KwNX9oFcEsmYde8.png)

```c
uint64
sys_sbrk(void)
{
  int addr;
  int n;

  if(argint(0, &n) < 0)
    return -1;
  
  addr = myproc()->sz;
  if(growproc(n) < 0)
    return -1;
  return addr;
}
```



```c
// Grow or shrink user memory by n bytes.
// Return 0 on success, -1 on failure.
int
growproc(int n)
{
  uint sz;
  struct proc *p = myproc();

  sz = p->sz;
  if(n > 0){
    if((sz = uvmalloc(p->pagetable, sz, sz + n)) == 0) {
      return -1;
    }
  } else if(n < 0){
    sz = uvmdealloc(p->pagetable, sz, sz + n);
  }
  p->sz = sz;
  return 0;
}

// Allocate PTEs and physical memory to grow process from oldsz to
// newsz, which need not be page aligned.  Returns new size or 0 on error.
uint64
uvmalloc(pagetable_t pagetable, uint64 oldsz, uint64 newsz)
{
  char *mem;
  uint64 a;

  if(newsz < oldsz)
    return oldsz;

  oldsz = PGROUNDUP(oldsz);
  for(a = oldsz; a < newsz; a += PGSIZE){
    mem = kalloc();
    if(mem == 0){
      uvmdealloc(pagetable, a, oldsz);
      return 0;
    }
    memset(mem, 0, PGSIZE);
    if(mappages(pagetable, a, PGSIZE, (uint64)mem, PTE_W|PTE_X|PTE_R|PTE_U) != 0){
      kfree(mem);
      uvmdealloc(pagetable, a, oldsz);
      return 0;
    }
  }
  return newsz;
}

// Deallocate user pages to bring the process size from oldsz to
// newsz.  oldsz and newsz need not be page-aligned, nor does newsz
// need to be less than oldsz.  oldsz can be larger than the actual
// process size.  Returns the new process size.
uint64
uvmdealloc(pagetable_t pagetable, uint64 oldsz, uint64 newsz)
{
  if(newsz >= oldsz)
    return oldsz;

  if(PGROUNDUP(newsz) < PGROUNDUP(oldsz)){
    int npages = (PGROUNDUP(oldsz) - PGROUNDUP(newsz)) / PGSIZE;
    uvmunmap(pagetable, PGROUNDUP(newsz), npages, 1);
  }

  return newsz;
}
```



## 3.8 Code: exec

Exec is the system call that **creates the user part of an address space**. It initializes the user part of an address space from a file **stored in the file system.** 



`exec.c`

```c
int
exec(char *path, char **argv)
{
  char *s, *last;
  int i, off;
  uint64 argc, sz = 0, sp, ustack[MAXARG], stackbase;
  struct elfhdr elf;
  struct inode *ip;
  struct proghdr ph;
  pagetable_t pagetable = 0, oldpagetable;
  struct proc *p = myproc();

  begin_op();

  if((ip = namei(path)) == 0){
    end_op();
    return -1;
  }
  ilock(ip);

  // Check ELF header
  if(readi(ip, 0, (uint64)&elf, 0, sizeof(elf)) != sizeof(elf))
    goto bad;
  if(elf.magic != ELF_MAGIC)
    goto bad;

  if((pagetable = proc_pagetable(p)) == 0)
    goto bad;

  // Load program into memory.
  for(i=0, off=elf.phoff; i<elf.phnum; i++, off+=sizeof(ph)){
    if(readi(ip, 0, (uint64)&ph, off, sizeof(ph)) != sizeof(ph))
      goto bad;
    if(ph.type != ELF_PROG_LOAD)
      continue;
    if(ph.memsz < ph.filesz)
      goto bad;
    if(ph.vaddr + ph.memsz < ph.vaddr)
      goto bad;
    uint64 sz1;
    if((sz1 = uvmalloc(pagetable, sz, ph.vaddr + ph.memsz)) == 0)
      goto bad;
    sz = sz1;
    if((ph.vaddr % PGSIZE) != 0)
      goto bad;
    if(loadseg(pagetable, ph.vaddr, ip, ph.off, ph.filesz) < 0)
      goto bad;
  }
  iunlockput(ip);
  end_op();
  ip = 0;

  p = myproc();
  uint64 oldsz = p->sz;

  // Allocate two pages at the next page boundary.
  // Use the second as the user stack.
  sz = PGROUNDUP(sz);
  uint64 sz1;
  if((sz1 = uvmalloc(pagetable, sz, sz + 2*PGSIZE)) == 0)
    goto bad;
  sz = sz1;
  uvmclear(pagetable, sz-2*PGSIZE);
  sp = sz;
  stackbase = sp - PGSIZE;

  // Push argument strings, prepare rest of stack in ustack.
  for(argc = 0; argv[argc]; argc++) {
    if(argc >= MAXARG)
      goto bad;
    sp -= strlen(argv[argc]) + 1;
    sp -= sp % 16; // riscv sp must be 16-byte aligned
    if(sp < stackbase)
      goto bad;
    if(copyout(pagetable, sp, argv[argc], strlen(argv[argc]) + 1) < 0)
      goto bad;
    ustack[argc] = sp;
  }
  ustack[argc] = 0;

  // push the array of argv[] pointers.
  sp -= (argc+1) * sizeof(uint64);
  sp -= sp % 16;
  if(sp < stackbase)
    goto bad;
  if(copyout(pagetable, sp, (char *)ustack, (argc+1)*sizeof(uint64)) < 0)
    goto bad;

  // arguments to user main(argc, argv)
  // argc is returned via the system call return
  // value, which goes in a0.
  p->trapframe->a1 = sp;

  // Save program name for debugging.
  for(last=s=path; *s; s++)
    if(*s == '/')
      last = s+1;
  safestrcpy(p->name, last, sizeof(p->name));
    
  // Commit to the user image.
  oldpagetable = p->pagetable;
  p->pagetable = pagetable;
  p->sz = sz;
  p->trapframe->epc = elf.entry;  // initial program counter = main
  p->trapframe->sp = sp; // initial stack pointer
  proc_freepagetable(oldpagetable, oldsz);
  
  if(p->pid == 1) vmprint(p->pagetable);
  return argc; // this ends up in a0, the first argument to main(argc, argv)

 bad:
  if(pagetable)
    proc_freepagetable(pagetable, sz);
  if(ip){
    iunlockput(ip);
    end_op();
  }
  return -1;
}
```

### What did exec do ?

1. opens the named binary path using `namei`
2. reads the ELF header. 
	1. a quick check that the file probably contains an ELF binary(magic number)
	2.  allocates a new page table with no user mappings with `proc_pagetable`
	3. allocates memory for each ELF segment with `uvmalloc `
3. loads each segment into memory with `loadseg`
	1. loadseg uses walkaddr to find the physical address of the allocated memory at which to write each page of the ELF segment
	2. and readi to read from the file
4. allocates and initializes the user stack.
5. 

### What is ELF

`elf.h`

```c
// Format of an ELF executable file

#define ELF_MAGIC 0x464C457FU  // "\x7FELF" in little endian

// File header
struct elfhdr {
  uint magic;  // must equal ELF_MAGIC
  uchar elf[12];
  ushort type;
  ushort machine;
  uint version;
  uint64 entry;
  uint64 phoff;
  uint64 shoff;
  uint flags;
  ushort ehsize;
  ushort phentsize;
  ushort phnum;
  ushort shentsize;
  ushort shnum;
  ushort shstrndx;
};

// Program section header
struct proghdr {
  uint32 type;
  uint32 flags;
  uint64 off;
  uint64 vaddr;
  uint64 paddr;
  uint64 filesz;
  uint64 memsz;
  uint64 align;
};

// Values for Proghdr type
#define ELF_PROG_LOAD           1

// Flag bits for Proghdr flags
#define ELF_PROG_FLAG_EXEC      1
#define ELF_PROG_FLAG_WRITE     2
#define ELF_PROG_FLAG_READ      4

```

