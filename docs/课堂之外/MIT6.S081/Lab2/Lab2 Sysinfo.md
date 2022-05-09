# Lab: system calls

## Sysinfo ([moderate](https://pdos.csail.mit.edu/6.S081/2021/labs/guidance.html))

In this assignment you will add a system call, `sysinfo`, that collects information about the running system. The system call takes one argument: a pointer to a `struct sysinfo` (see `kernel/sysinfo.h`). The kernel should fill out the fields of this struct: the `freemem` field should be set to the number of bytes of free memory, and the `nproc` field should be set to the number of processes whose `state` is not `UNUSED`. We provide a test program `sysinfotest`; you pass this assignment if it prints "sysinfotest: OK".

Some hints:

- Add `$U/_sysinfotest` to UPROGS in Makefile

- Run make qemu; `user/sysinfotest.c` will fail to compile. Add the system call sysinfo, following the same steps as in the previous assignment. To declare the prototype for sysinfo() `in user/user.h` you need predeclare the existence of `struct sysinfo`:

	```
	    struct sysinfo;
	    int sysinfo(struct sysinfo *);
	  
	```

	Once you fix the compilation issues, run sysinfotest; it will fail because you haven't implemented the system call in the kernel yet.

- sysinfo needs to copy a `struct sysinfo` back to user space; see `sys_fstat()` (`kernel/sysfile.c`) and `filestat()` (`kernel/file.c`) for examples of how to do that using `copyout()`.

- To collect the amount of free memory, add a function to `kernel/kalloc.c`

- To collect the number of processes, add a function to `kernel/proc.c`

## 题意

完成一个系统调用`sysinfo`来收集系统信息，参数为一个指向`struct sysinfo`的指针

```c
struct sysinfo {
  uint64 freemem;   // amount of free memory (bytes)
  uint64 nproc;     // number of process
};

int sysinfo(struct sysinfo *);
```

* the freemem field should be set to the number of bytes of free memory
* and the nproc field should be set to the number of processes w**hose state s not UNUSED.**

## 过程

### step1 查看`sysinfotest.c`

```c
#include "../kernel/types.h"
#include "../kernel/riscv.h"
#include "../kernel/sysinfo.h"
#include "../user/user.h"


void
sinfo(struct sysinfo *info) {
  if (sysinfo(info) < 0) {
    printf("FAIL: sysinfo failed");
    exit(1);
  }
}

//
// use sbrk() to count how many free physical memory pages there are.
//
int
countfree()
{
  uint64 sz0 = (uint64)sbrk(0);
  struct sysinfo info;
  int n = 0;

  while(1){
    if((uint64)sbrk(PGSIZE) == 0xffffffffffffffff){
      break;
    }
    n += PGSIZE;
  }
  sinfo(&info);
  if (info.freemem != 0) {
    printf("FAIL: there is no free mem, but sysinfo.freemem=%d\n",
      info.freemem);
    exit(1);
  }
  sbrk(-((uint64)sbrk(0) - sz0));
  return n;
}

void
testmem() {
  struct sysinfo info;
  uint64 n = countfree();
  
  sinfo(&info);

  if (info.freemem!= n) {
    printf("FAIL: free mem %d (bytes) instead of %d\n", info.freemem, n);
    exit(1);
  }
  
  if((uint64)sbrk(PGSIZE) == 0xffffffffffffffff){
    printf("sbrk failed");
    exit(1);
  }

  sinfo(&info);
    
  if (info.freemem != n-PGSIZE) {
    printf("FAIL: free mem %d (bytes) instead of %d\n", n-PGSIZE, info.freemem);
    exit(1);
  }
  
  if((uint64)sbrk(-PGSIZE) == 0xffffffffffffffff){
    printf("sbrk failed");
    exit(1);
  }

  sinfo(&info);
    
  if (info.freemem != n) {
    printf("FAIL: free mem %d (bytes) instead of %d\n", n, info.freemem);
    exit(1);
  }
}

void
testcall() {
  struct sysinfo info;
  
  if (sysinfo(&info) < 0) {
    printf("FAIL: sysinfo failed\n");
    exit(1);
  }

  if (sysinfo((struct sysinfo *) 0xeaeb0b5b00002f5e) !=  0xffffffffffffffff) {
    printf("FAIL: sysinfo succeeded with bad argument\n");
    exit(1);
  }
}

void testproc() {
  struct sysinfo info;
  uint64 nproc;
  int status;
  int pid;
  
  sinfo(&info);
  nproc = info.nproc;

  pid = fork();
  if(pid < 0){
    printf("sysinfotest: fork failed\n");
    exit(1);
  }
  if(pid == 0){
    sinfo(&info);
    if(info.nproc != nproc+1) {
      printf("sysinfotest: FAIL nproc is %d instead of %d\n", info.nproc, nproc+1);
      exit(1);
    }
    exit(0);
  }
  wait(&status);
  sinfo(&info);
  if(info.nproc != nproc) {
      printf("sysinfotest: FAIL nproc is %d instead of %d\n", info.nproc, nproc);
      exit(1);
  }
}

int
main(int argc, char *argv[])
{
  printf("sysinfotest: start\n");
  testcall();
  testmem();
  testproc();
  printf("sysinfotest: OK\n");
  exit(0);
}

```

会进行三个测试

* `testcall()`测试调用
* `testmem()`测试统计内存功能
* `testproc()`测试统计进程功能

step2 尝试编译运行

在`Makefile`中添加`$U/_sysinfotest`

```makefile
UPROGS=\
	$U/_cat\
	$U/_echo\
	$U/_forktest\
	$U/_grep\
	$U/_init\
	$U/_kill\
	$U/_ln\
	$U/_ls\
	$U/_mkdir\
	$U/_rm\
	$U/_sh\
	$U/_stressfs\
	$U/_usertests\
	$U/_grind\
	$U/_wc\
	$U/_zombie\
	$U/_trace\
	$U/_sysinfotest\
```

Run `make qemu`发现报错因为还没有实现`sysinfo`



### step3 添加系统调用



`usys.pl` 添加系统调用，生成入口

```perl
entry("sysinfo");
```

`syscall.h` 添加系统调用号

```
#define SYS_sysinfo 23
```

`syscall.c`

```c
//... some system call ...
extern uint64 sys_sysinfo(void);

static uint64 (*syscalls[])(void) = {
//... some system call ...
[SYS_sysinfo] sys_sysinfo,
};

```

`sysproc.c`

```c
uint64
sys_sysinfo(void){

}

```



### step4 实现 `sys_sysinfo(void)`

使用`argaddr`读取`int sysinfo(struct sysinfo *);`传入的地址参数`struct sysinfo *`

使用`copyout`将内核态`sys_sysinfo`计算得到的结果传回用户态

![image-20220426162313053](https://s2.loli.net/2022/04/26/whRyPjfsxIgmBep.png)

```c
uint64
sys_sysinfo(void){
	struct proc *p = myproc();
	struct sysinfo info;
	uint64 addr;
	
	//... do something ...
	
	if(argaddr(0, &addr) < 0)
		return -1;
	if(copyout(p->pagetable, addr, (char *)&info, sizeof(info)) < 0)
		return -1;

    return 0;
}

```



接下来考虑如何获取`freemem`和`nproc`，实现`getfreemem`和`getnproc`

```c
uint64
sys_sysinfo(void){
	struct proc *p = myproc();
	struct sysinfo info;
	uint64 addr;
	info.freemem = getfreemem();
	info.nproc = getnproc();
	if(argaddr(0, &addr) < 0)
		return -1;
	if(copyout(p->pagetable, addr, (char *)&info, sizeof(info)) < 0)
		return -1;

    return 0;
}

```



先在`defs.h`中添加两个函数的声明

```c
// kalloc.c
uint64 			getfreemem(void);

// proc.c
uint64          getnproc(void);
```



### step5 实现 `getfreemem(void)`

在 `kalloc.c`中实现`getfreemem(void)`



`kalloc.`的核心数据结构

```c
struct run {
  struct run *next;
};

struct {
  struct spinlock lock;
  struct run *freelist;
} kmem;

```

可以通过 `freelist`来统计空闲内存



```c
uint64
getfreemem() {
	uint64 ret = 0;
	
	struct run *p = kmem.freelist;
	while(p) {
		ret += PGSIZE;
		p = p->next;
	}

	return ret;
}

```



### step6 实现`getnproc(void)`

在`proc.c`中实现`proc.h`

核心数据结构 一个存了 NPROC 个进程的数组

```c
struct proc proc[NPROC];
```

根据提示，要统计进程状态不为`UNUSED`的进程数量，遍历即可

```c
uint64
getnproc(void)
{
	uint64 ret = 0;

	for(int i = 0; i < NPROC; ++i) {
		if(proc[i].state != UNUSED) {
			++ ret;
		}
	}

	return ret;
}

```

