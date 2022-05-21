# Lab4 traps

## Alarm ([hard](https://pdos.csail.mit.edu/6.S081/2021/labs/guidance.html))

In this exercise you'll add a feature to xv6 that periodically alerts a process as it uses CPU time. This might be useful for compute-bound processes that want to limit how much CPU time they chew up, or for processes that want to compute but also want to take some periodic action. More generally, you'll be implementing a primitive form of user-level interrupt/fault handlers; you could use something similar to handle page faults in the application, for example. Your solution is correct if it passes alarmtest and usertests.

You should add a new `sigalarm(interval, handler)` system call. If an application calls `sigalarm(n, fn)`, then after every `n` "ticks" of CPU time that the program consumes, the kernel should cause application function `fn` to be called. When `fn` returns, the application should resume where it left off. A tick is a fairly arbitrary unit of time in xv6, determined by how often a hardware timer generates interrupts. If an application calls `sigalarm(0, 0)`, the kernel should stop generating periodic alarm calls.

You'll find a file `user/alarmtest.c` in your xv6 repository. Add it to the Makefile. It won't compile correctly until you've added `sigalarm` and `sigreturn` system calls (see below).

`alarmtest` calls `sigalarm(2, periodic)` in `test0` to ask the kernel to force a call to `periodic()` every 2 ticks, and then spins for a while. You can see the assembly code for alarmtest in user/alarmtest.asm, which may be handy for debugging. Your solution is correct when `alarmtest` produces output like this and usertests also runs correctly:

```shell
$ alarmtest
test0 start
........alarm!
test0 passed
test1 start
...alarm!
..alarm!
...alarm!
..alarm!
...alarm!
..alarm!
...alarm!
..alarm!
...alarm!
..alarm!
test1 passed
test2 start
................alarm!
test2 passed
$ usertests
...
ALL TESTS PASSED
$
```

When you're done, your solution will be only a few lines of code, but it may be tricky to get it right. We'll test your code with the version of `alarmtest.c` in the original repository. You can modify `alarmtest.c` to help you debug, but make sure the original alarmtest says that all the tests pass.

### test0: invoke handler

Get started by modifying the kernel to jump to the alarm handler in user space, which will cause test0 to print "alarm!". Don't worry yet what happens after the "alarm!" output; it's OK for now if your program crashes after printing "alarm!". Here are some hints:

- You'll need to modify the Makefile to cause `alarmtest.c` to be compiled as an xv6 user program.

- The right declarations to put in`user/user.h`are:

	```c
	int sigalarm(int ticks, void (*handler)());
	int sigreturn(void);
	```
	
- Update user/usys.pl (which generates user/usys.S), kernel/syscall.h, and kernel/syscall.c to allow `alarmtest` to invoke the sigalarm and sigreturn system calls.

- For now, your `sys_sigreturn` should just return zero.

- Your `sys_sigalarm()` should store the alarm interval and the pointer to the handler function in new fields in the `proc` structure (in `kernel/proc.h`).

- You'll need to keep track of how many ticks have passed since the last call (or are left until the next call) to a process's alarm handler; you'll need a new field in `struct proc` for this too. You can initialize `proc` fields in `allocproc()` in `proc.c`.

- Every tick, the hardware clock forces an interrupt, which is handled in `usertrap()` in `kernel/trap.c`.

- You only want to manipulate a process's alarm ticks if there's a timer interrupt; you want something like

	```c
	if(which_dev == 2) ...
	```

- Only invoke the alarm function if the process has a timer outstanding. Note that the address of the user's alarm function might be 0 (e.g., in `user/alarmtest.asm`, `periodic` is at address 0).

- You'll need to modify `usertrap()` so that when a process's alarm interval expires, the user process executes the handler function. When a trap on the RISC-V returns to user space, what determines the instruction address at which user-space code resumes execution?

- It will be easier to look at traps with gdb if you tell qemu to use only one CPU, which you can do by running

	```shell
	make CPUS=1 qemu-gdb
	```

- You've succeeded if alarmtest prints "alarm!".

### test1/test2(): resume interrupted code

Chances are that alarmtest crashes in test0 or test1 after it prints "alarm!", or that alarmtest (eventually) prints "test1 failed", or that alarmtest exits without printing "test1 passed". To fix this, you must ensure that, when the alarm handler is done, control returns to the instruction at which the user program was originally interrupted by the timer interrupt. You must ensure that the register contents are restored to the values they held at the time of the interrupt, so that the user program can continue undisturbed after the alarm. Finally, you should "re-arm" the alarm counter after each time it goes off, so that the handler is called periodically.

As a starting point, we've made a design decision for you: user alarm handlers are required to call the `sigreturn` system call when they have finished. Have a look at `periodic` in `alarmtest.c` for an example. This means that you can add code to `usertrap` and `sys_sigreturn` that cooperate to cause the user process to resume properly after it has handled the alarm.

Some hints:

- Your solution will require you to save and restore registers---what registers do you need to save and restore to resume the interrupted code correctly? (Hint: it will be many).
- Have `usertrap` save enough state in `struct proc` when the timer goes off that `sigreturn` can correctly return to the interrupted user code.
- Prevent re-entrant calls to the handler----if a handler hasn't returned yet, the kernel shouldn't call it again. `test2` tests this.

Once you pass `test0`, `test1`, and `test2` run `usertests` to make sure you didn't break any other parts of the kernel.



## 过程

在本练习中，您将向 xv6 添加一个功能，该功能会在进程使用 CPU 时间时定期提醒它。这对于想要限制它们占用多少 CPU 时间的计算绑定进程，或者对于想要计算但又想要采取一些定期操作的进程可能很有用。**更一般地说，您将实现一种原始形式的用户级中断/故障处理程序**；例如，您可以使用类似的东西来处理应用程序中的页面错误。如果它通过了警报测试和用户测试，则您的解决方案是正确的。



需要通过两个测试`alarmtest.c`和``usertest.c`，`usertest.c`太长了就不直接放出来了，下面是`alarmtest.c`的代码

* `test0()`：测试内核是否调用了一次警报处理程序。
* `test1()`：测试内核多次调用处理程序。 测试当处理程序返回时，它返回到程序中发生定时器中断的点，所有寄存器都保持与中断发生时相同的值。
* `test2()`：测试内核不允许重入警报调用。

```c
//
// test program for the alarm lab.
// you can modify this file for testing,
// but please make sure your kernel
// modifications pass the original
// versions of these tests.
//

#include "kernel/param.h"
#include "kernel/types.h"
#include "kernel/stat.h"
#include "kernel/riscv.h"
#include "user/user.h"

void test0();
void test1();
void test2();
void periodic();
void slow_handler();

int
main(int argc, char *argv[])
{
  test0();
  test1();
  test2();
  exit(0);
}

volatile static int count;

void
periodic()
{
  count = count + 1;
  printf("alarm!\n");
  sigreturn();
}

// tests whether the kernel calls
// the alarm handler even a single time.
void
test0()
{
  int i;
  printf("test0 start\n");
  count = 0;
  sigalarm(2, periodic);
  for(i = 0; i < 1000*500000; i++){
    if((i % 1000000) == 0)
      write(2, ".", 1);
    if(count > 0)
      break;
  }
  sigalarm(0, 0);
  if(count > 0){
    printf("test0 passed\n");
  } else {
    printf("\ntest0 failed: the kernel never called the alarm handler\n");
  }
}

void __attribute__ ((noinline)) foo(int i, int *j) {
  if((i % 2500000) == 0) {
    write(2, ".", 1);
  }
  *j += 1;
}

//
// tests that the kernel calls the handler multiple times.
//
// tests that, when the handler returns, it returns to
// the point in the program where the timer interrupt
// occurred, with all registers holding the same values they
// held when the interrupt occurred.
//
void
test1()
{
  int i;
  int j;

  printf("test1 start\n");
  count = 0;
  j = 0;
  sigalarm(2, periodic);
  for(i = 0; i < 500000000; i++){
    if(count >= 10)
      break;
    foo(i, &j);
  }
  if(count < 10){
    printf("\ntest1 failed: too few calls to the handler\n");
  } else if(i != j){
    // the loop should have called foo() i times, and foo() should
    // have incremented j once per call, so j should equal i.
    // once possible source of errors is that the handler may
    // return somewhere other than where the timer interrupt
    // occurred; another is that that registers may not be
    // restored correctly, causing i or j or the address ofj
    // to get an incorrect value.
    printf("\ntest1 failed: foo() executed fewer times than it was called\n");
  } else {
    printf("test1 passed\n");
  }
}

//
// tests that kernel does not allow reentrant alarm calls.
void
test2()
{
  int i;
  int pid;
  int status;

  printf("test2 start\n");
  if ((pid = fork()) < 0) {
    printf("test2: fork failed\n");
  }
  if (pid == 0) {
    count = 0;
    sigalarm(2, slow_handler);
    for(i = 0; i < 1000*500000; i++){
      if((i % 1000000) == 0)
        write(2, ".", 1);
      if(count > 0)
        break;
    }
    if (count == 0) {
      printf("\ntest2 failed: alarm not called\n");
      exit(1);
    }
    exit(0);
  }
  wait(&status);
  if (status == 0) {
    printf("test2 passed\n");
  }
}

void
slow_handler()
{
  count++;
  printf("alarm!\n");
  if (count > 1) {
    printf("test2 failed: alarm handler called more than once\n");
    exit(1);
  }
  for (int i = 0; i < 1000*500000; i++) {
    asm volatile("nop"); // avoid compiler optimizing away loop
  }
  sigalarm(0, 0);
  sigreturn();
}

```



### test0：调用处理程序

#### step1：修改 makefile，添加声明

您需要修改 Makefile 以使`alarmtest.c` 编译为 xv6 用户程序

并在`user/user.h`添加如下声明

```c
int sigalarm(int ticks, void (*handler)());
int sigreturn(void);
```

#### step2：添加 sigalarm 和 sigreturn 系统调用

`user/uys.pl`

```perl
entry("sigalarm");
entry("sigreturn");
```

`kernel/syscall.h`

```c
#define SYS_sigalarm 22
#define SYS_sigreturn 23
```

`kernel/syscall.c`

```c
extern uint64 sys_sigalarm(void);
extern uint64 sys_sigreturn(void);

static uint64 (*syscalls[])(void) = {
[SYS_fork]    sys_fork,
[SYS_exit]    sys_exit,
[SYS_wait]    sys_wait,
[SYS_pipe]    sys_pipe,
[SYS_read]    sys_read,
[SYS_kill]    sys_kill,
[SYS_exec]    sys_exec,
[SYS_fstat]   sys_fstat,
[SYS_chdir]   sys_chdir,
[SYS_dup]     sys_dup,
[SYS_getpid]  sys_getpid,
[SYS_sbrk]    sys_sbrk,
[SYS_sleep]   sys_sleep,
[SYS_uptime]  sys_uptime,
[SYS_open]    sys_open,
[SYS_write]   sys_write,
[SYS_mknod]   sys_mknod,
[SYS_unlink]  sys_unlink,
[SYS_link]    sys_link,
[SYS_mkdir]   sys_mkdir,
[SYS_close]   sys_close,
[SYS_sigalarm]	sys_sigalarm,	//new
[SYS_sigreturn]	sys_sigreturn	//new
};

```

在`sysproc.c`准备实现这两个系统调用，目前`sys_sigreturn`直接返回0即可

```c
uint64 sys_sigalarm(void)
{

}

uint64 sys_sigreturn(void) 
{
    return 0;
}
```



#### step3：在 proc 中添加相关字段

在`kernel/proc.h`的`proc`结构体内添加以下字段

* 警报间隔
* 指向处理函数的指针
* 已经经过的 tick 数（如何获得呢）

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

  int ticks;			//经过的tcik数
  void(*handler)();		//处理函数
  int alarm_interval;	//时间间隔
};

```

我们可以通过`proc.c`的`allocproc()`来初始化`proc`里的字段，在`freeproc()`恢复初始值

```c
static struct proc*
allocproc(void)
{
// do something
found:
  p->pid = allocpid();
  p->state = USED;
  p->alarm_ticks = 0;
  p->alarm_interval = 0;
  p->handler = 0;
// do something
}


static void
freeproc(struct proc *p)
{
// do something
  p->alarm_interval = 0;
  p->handler = 0;
  p->alarm_ticks = 0;
}

```



#### step4：获取 ticks

每个 tick，硬件时钟都会强制中断，该中断在`kernel/trap.c`的`usertrap()`中处理。

找到`usertrap()`中时钟中断相关的代码

```c
 // give up the CPU if this is a timer interrupt.
  if(which_dev == 2) 
    yield();
```

添加计数

```c
  // give up the CPU if this is a timer interrupt.
  if(which_dev == 2) {
	++(p->ticks);
    yield();
  }
```



### step5: 调用警报函数

仅当进程有未完成的计时器时才调用警报函数。

```c
  // give up the CPU if this is a timer interrupt.
  if(which_dev == 2) {
	  if(p->alarm_interval != 0) {	//如果设定了时钟事件
			++(p->alarm_ticks);
			if(p->alarm_ticks % p->alarm_interval == 0) {// 到达计时周期
				p->trapframe->epc = p->handler;	//调用处理函数
			}
		}
    yield();
  }
```



### test1/test2()：恢复中断的代码

有可能 alarmtest 在打印“alarm!”后在 test0 或 test1 中崩溃，或者 alarmtest（最终）打印“test1 failed”，或者 alarmtest 退出而不打印“test1 pass”。要解决此问题，您必须确保在警报处理程序完成后，控制权返回到用户程序最初被定时器中断中断的指令。您必须确保寄存器内容恢复到它们在中断时保持的值，以便用户程序可以在警报后不受干扰地继续运行。最后，您应该在每次关闭后“重新武装”警报计数器，以便定期调用处理程序。

作为起点，我们已经为您做出了设计决策：用户警报处理程序需要在完成后调用`sigreturn` 系统调用。以`alarmtest.c` 中的`周期性``为例`。这意味着您可以将代码添加到`usertrap`和 `sys_sigreturn`中，它们会协同工作以使用户进程在处理完警报后正确恢复。

一些提示：

- 您的解决方案将要求您保存和恢复寄存器——您需要保存和恢复哪些寄存器才能正确恢复中断的代码？（提示：会很多）。
- 当计时器关闭时，让 usertrap 在 struct proc 中保存足够的状态，以便sigreturn`可以``正确`返回 `到`被中断的用户代码。````
- 防止对处理程序的重入调用——如果处理程序尚未返回，内核不应再次调用它。`test2` 对此进行测试。

一旦你通过`了 test0`、`test1`和`test2` 运行`usertests`以确保你没有破坏内核的任何其他部分。

### step1 添加新的 trapframe

为了正确恢复被中断的代码，需要有个地方来保存原来的寄存器，这些寄存器存在 `struct trapframe`中

```c
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
```

我们进行下面修改

* 在`struct proc`中新增一个字段`alarm_trapframe`
* 在`allocproc`中使用`kalloc`初始化
* 在`freeproc`中使用`kfree`释放



```cpp
// Per-process state
struct proc {
  // do something
  int alarm_ticks;							//经过的tcik数
  uint64 handler;							//处理函数
  int alarm_interval;						//时间间隔
  struct trapframe* alarm_trapframe;		//保存寄存器
};

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
  //do something
  if((p->alarm_trapframe = (struct trapframe*)kalloc()) == 0) {
	  release(&p->lock);
	  return 0;
  }
  return p;
}

static void
freeproc(struct proc *p)
{
  //do something
  if(p->alarm_trapframe)
	kfree((void*)p->alarm_trapframe);
}

```



### step2：保存寄存器

在`usertrap(void)`的时钟中断处理部分添加保存寄存器的代码

```c
void
usertrap(void)
{
  //do something

  // give up the CPU if this is a timer interrupt.
  if(which_dev == 2) {
	  if(p->alarm_interval != 0) {	//如果设定了时钟事件
			++(p->alarm_ticks);
			if(p->alarm_ticks % p->alarm_interval == 0) {	// 到达计时周期
                //保存寄存器
				memmove(p->alarm_trapframe, p->trapframe, sizeof(struct trapframe)); 
                //调用处理函数
				p->trapframe->epc = p->handler;
			}
		}
    yield();
  }

  usertrapret();
}
```



### step3：返回到被中断的代码

```c
uint64 sys_sigreturn(void)
{
	struct proc* p = myproc();
	memmove(p->trapframe, p->alarm_trapframe, sizeof(struct trapframe));
	p->alarm_ticks = 0;
	p->handler = 0;
	return 0;
}
```



### step4：处理重复调用

防止对处理程序的重入调用——如果处理程序尚未返回，内核不应再次调用它

通过添加一个标记变量`handlerlock`来模拟一个锁，判断是否有处理程序尚未返回

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

  int alarm_ticks;							//经过的tcik数
  uint64 handler;							//处理函数
  int alarm_interval;						//时间间隔
  struct trapframe* alarm_trapframe;		//保存寄存器
  int handlerlock;							//标记是否有处理程序没返回
};

```

对应的在`allocproc`和`freeproc`中修改

```c
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
  //do something

  p->alarm_ticks = 0;
  p->alarm_interval = 0;
  p->handler = 0;
  p->handlerlock = 1;
  if((p->alarm_trapframe = (struct trapframe*)kalloc()) == 0) {
	  release(&p->lock);
	  return 0;
  }
  return p;
}

static void
freeproc(struct proc *p)
{
 //do something
  p->alarm_interval = 0;
  p->handler = 0;
  p->alarm_ticks = 0;
  p->handlerlock = 1;
  if(p->alarm_trapframe)
	kfree((void*)p->alarm_trapframe);
}
```

在`usertrap`的时钟处理部分添加`handlerlock`判断条件

```c
// give up the CPU if this is a timer interrupt.
if(which_dev == 2) {
    if(p->alarm_interval != 0) {	//如果设定了时钟事件
        ++(p->alarm_ticks);
        if(p->alarm_ticks % p->alarm_interval == 0
           && p->handlerlock) {
            p->handlerlock = 0;	
            memmove(p->alarm_trapframe, p->trapframe, sizeof(struct trapframe));
            p->trapframe->epc = p->handler;
        }
    }
    yield();
}
```

在`sys_sigreturn`中释放锁

```c
uint64 sys_sigreturn(void)
{
	struct proc* p = myproc();
	memmove(p->trapframe, p->alarm_trapframe, sizeof(struct trapframe));
	p->alarm_ticks = 0;
	p->handler = 0;
	p->handlerlock = 1;
	return 0;
}
```



至此应该已经可以通过`alarmtest`和`usertest`了

## 遗留问题

在最后一步写了下面代码，为何有问题

```c
uint64 sys_sigreturn(void)
{
	struct proc* p = myproc();
	memmove(p->trapframe, p->alarm_trapframe, sizeof(struct trapframe));
	p->alarm_ticks = 0;
	// p->alarm_interval = 0; 为什么不能写这个？
	p->handler = 0;
	p->handlerlock = 1;
	return 0;
}

```

研究一下测试程序的`test2`

```c
// tests that kernel does not allow reentrant alarm calls.
void
test2()
{
  int i;
  int pid;
  int status;

  printf("test2 start\n");
  if ((pid = fork()) < 0) {
    printf("test2: fork failed\n");
  }
  if (pid == 0) {
    count = 0;
    sigalarm(2, slow_handler);
    for(i = 0; i < 1000*500000; i++){
      if((i % 1000000) == 0)
        write(2, ".", 1);
      if(count > 0)
        break;
    }
    if (count == 0) {
      printf("\ntest2 failed: alarm not called\n");
      exit(1);
    }
    exit(0);
  }
  wait(&status);
  if (status == 0) {
    printf("test2 passed\n");
  }
}

void
slow_handler()
{
  count++;
  printf("alarm!\n");
  if (count > 1) {
    printf("test2 failed: alarm handler called more than once\n");
    exit(1);
  }
  for (int i = 0; i < 1000*500000; i++) {
    asm volatile("nop"); // avoid compiler optimizing away loop
  }
  sigalarm(0, 0);
  sigreturn();
}

```

