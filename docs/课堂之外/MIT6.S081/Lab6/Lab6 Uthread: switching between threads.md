# Lab6 Uthread: switching between threads

## 要求

In this exercise you will design the context switch mechanism for a user-level threading system, and then implement it. To get you started, your xv6 has two files `user/uthread.c` and `user/uthread_switch.S`, and a rule in the Makefile to build a uthread program. uthread.c contains most of a user-level threading package, and code for three simple test threads. The threading package is missing some of the code to create a thread and to switch between threads.

Your job is to come up with a plan to create threads and save/restore registers to switch between threads, and implement that plan. When you're done, `make grade` should say that your solution passes the `uthread` test.

Once you've finished, you should see the following output when you run `uthread` on xv6 (the three threads might start in a different order):

```bash
$ make qemu
...
$ uthread
thread_a started
thread_b started
thread_c started
thread_c 0
thread_a 0
thread_b 0
thread_c 1
thread_a 1
thread_b 1
...
thread_c 99
thread_a 99
thread_b 99
thread_c: exit after 100
thread_a: exit after 100
thread_b: exit after 100
thread_schedule: no runnable threads
$
```

This output comes from the three test threads, each of which has a loop that prints a line and then yields the CPU to the other threads.



At this point, however, with no context switch code, you'll see no output.



You will need to add code to `thread_create()` and `thread_schedule()` in `user/uthread.c`, and `thread_switch` in `user/uthread_switch.S`. One goal is ensure that when `thread_schedule()` runs a given thread for the first time, the thread executes the function passed to `thread_create()`, on its own stack. Another goal is to ensure that `thread_switch` saves the registers of the thread being switched away from, restores the registers of the thread being switched to, and returns to the point in the latter thread's instructions where it last left off. You will have to decide where to save/restore registers; modifying `struct thread` to hold registers is a good plan. You'll need to add a call to `thread_switch` in `thread_schedule`; you can pass whatever arguments you need to `thread_switch`, but the intent is to switch from thread `t` to `next_thread`.

Some hints:

- `thread_switch` needs to save/restore only the callee-save registers. Why?

- You can see the assembly code for uthread in user/uthread.asm, which may be handy for debugging.

- To test your code it might be helpful to single step through your`thread_switch` using`riscv64-linux-gnu-gdb`. You can get started in this way:

	```bash
	(gdb) file user/_uthread
	Reading symbols from user/_uthread...
	(gdb) b uthread.c:60
	```

	This sets a breakpoint at line 60 of `uthread.c`. The breakpoint may (or may not) be triggered before you even run `uthread`. How could that happen?

	Once your xv6 shell runs, type "uthread", and gdb will break at line 60. Now you can type commands like the following to inspect the state of `uthread`:

	```
	(gdb) p/x *next_thread
	```

	With "x", you can examine the content of a memory location:

	```
	(gdb) x/x next_thread->stack
	```

	You can skip to the start of `thread_switch` thus:

	```
	(gdb) b thread_switch
	(gdb) c
	```

	You can single step assembly instructions using:

	```
	(gdb) si
	```

	On-line documentation for gdb is [here](https://sourceware.org/gdb/current/onlinedocs/gdb/).



## 解决方案

先看看 `uthread.c`

线程结构体和全局变量，每个线程有自己的栈和状态

```c
struct thread {
  char       stack[STACK_SIZE]; /* the thread's stack */
  int        state;             /* FREE, RUNNING, RUNNABLE */
};
struct thread all_thread[MAX_THREAD];	//所有线程
struct thread *current_thread;			//当前线程
```



主函数：

1. 首先调用`thread_init`对主函数线程进行初始化
2. 然后调用`thread_create`将线程函数绑定到线程上，并将线程状态设为就绪态`RUNNABLE`
3. 最后调用`thread_schedule`启动线程调度

下面来详细看一下各个函数

```c
int 
main(int argc, char *argv[]) 
{
  a_started = b_started = c_started = 0;
  a_n = b_n = c_n = 0;
  thread_init();
  thread_create(thread_a);
  thread_create(thread_b);
  thread_create(thread_c);
  thread_schedule();
  exit(0);
}
```



`void thread_init(void)`

`main()` 是线程 0，它将进行第一次调用`thread_schedule()`。 它需要一个栈，以便第一个 `thread_switch()` 可以
保存线程 0 的状态。 

`thread_schedule()` 永远不会再次运行`main`线程，因为它的状态设置为 `RUNNING`，但是` thread_schedule()` 要选择一个 `RUNNABLE` 线程。

```c
void 
thread_init(void)
{
  // main() is thread 0, which will make the first invocation to
  // thread_schedule().  it needs a stack so that the first thread_switch() can
  // save thread 0's state.  thread_schedule() won't run the main thread ever
  // again, because its state is set to RUNNING, and thread_schedule() selects
  // a RUNNABLE thread.
  current_thread = &all_thread[0];
  current_thread->state = RUNNING;
}
```



`void thread_create(void (*func)())` 

将传入线程函数绑定到线程上

并将线程状态设为就绪态`RUNNABLE`

```c
void 
thread_create(void (*func)())
{
  struct thread *t;

  for (t = all_thread; t < all_thread + MAX_THREAD; t++) {
    if (t->state == FREE) break;
  }
  t->state = RUNNABLE;
  // YOUR CODE HERE
}
```

可以发现将线程函数绑定到线程上的代码还没有实现，如何绑定呢？

首先要解决的问题，绑定的信息存在哪里？结合所学可以想到应该存在上下文`context`当中

我们直接从把`proc.h`的`context`结构体照搬过来，塞到`thread`结构体里

```c
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

struct thread {
  char       stack[STACK_SIZE]; /* the thread's stack */
  int        state;             /* FREE, RUNNING, RUNNABLE */
  struct context context;
};
```

这样我们就可以把线程函数绑定到线程上了

`ra`寄存器存放函数指针

`sp`寄存器存放栈指针，需要注意的是栈生长方向从高地址向地址生长，所以栈底是高地址开始

```c
void 
thread_create(void (*func)())
{
  struct thread *t;

  for (t = all_thread; t < all_thread + MAX_THREAD; t++) {
    if (t->state == FREE) break;
  }
  t->state = RUNNABLE;
  // YOUR CODE HERE
  t->context.ra = (uint64)func;
  t->context.sp = (uint64)&t->stack[STACK_SIZE - 1];
}
```



`void thread_yield(void)`

当前线程让出 CPU ，状态变为`RUNNABLE`并自动调用`thread_scheduler`

```c
void 
thread_yield(void)
{
  current_thread->state = RUNNABLE;
  thread_schedule();
}
```



`thread_schedule(void)`

线程调度分为两步

1. 找一个`RUNNABLE`状态的线程
2. 如果这个线程不是当前线程，就进行线程切换

```c
void 
thread_schedule(void)
{
  struct thread *t, *next_thread;

  /* Find another runnable thread. */
  next_thread = 0;
  t = current_thread + 1;
  for(int i = 0; i < MAX_THREAD; i++){
    if(t >= all_thread + MAX_THREAD)
      t = all_thread;
    if(t->state == RUNNABLE) {
      next_thread = t;
      break;
    }
    t = t + 1;
  }

  if (next_thread == 0) {
    printf("thread_schedule: no runnable threads\n");
    exit(-1);
  }

  if (current_thread != next_thread) {         /* switch threads?  */
    next_thread->state = RUNNING;
    t = current_thread;
    current_thread = next_thread;
    /* YOUR CODE HERE
     * Invoke thread_switch to switch from t to next_thread:
     * thread_switch(??, ??);
     */
  } else
    next_thread = 0;
}
```

这里需要将当前线程`t`与确定要调度的线程进行上下文的切换

```c
if (current_thread != next_thread) {         /* switch threads?  */
    next_thread->state = RUNNING;
    t = current_thread;
    current_thread = next_thread;
    /* YOUR CODE HERE
     * Invoke thread_switch to switch from t to next_thread:
     * thread_switch(??, ??);
     */
    thread_switch((uint64)&t->context,(uint64)&current_thread->context);
} 
```



`thread_switch`代码在`uthread_switch.S`中实现，同样模仿`switch.S`的即可

```c
	.text

	/*
         * save the old thread's registers,
         * restore the new thread's registers.
         */

	.globl thread_switch
thread_switch:
	/* YOUR CODE HERE */
	    sd ra, 0(a0)
        sd sp, 8(a0)
        sd s0, 16(a0)
        sd s1, 24(a0)
        sd s2, 32(a0)
        sd s3, 40(a0)
        sd s4, 48(a0)
        sd s5, 56(a0)
        sd s6, 64(a0)
        sd s7, 72(a0)
        sd s8, 80(a0)
        sd s9, 88(a0)
        sd s10, 96(a0)
        sd s11, 104(a0)

        ld ra, 0(a1)
        ld sp, 8(a1)
        ld s0, 16(a1)
        ld s1, 24(a1)
        ld s2, 32(a1)
        ld s3, 40(a1)
        ld s4, 48(a1)
        ld s5, 56(a1)
        ld s6, 64(a1)
        ld s7, 72(a1)
        ld s8, 80(a1)
        ld s9, 88(a1)
        ld s10, 96(a1)
        ld s11, 104(a1)

	ret    /* return to ra */

```



## 问题

`thread_switch` needs to save/restore only the callee-save registers. Why?