# Lab4 traps

## Backtrace ([moderate](https://pdos.csail.mit.edu/6.S081/2021/labs/guidance.html))

For debugging it is often useful to have a `backtrace`: a list of the function calls on the stack above the point at which the error occurred.

Implement a `backtrace()` function in `kernel/printf.c`. Insert a call to this function in `sys_sleep`, and then run `bttest`, which calls `sys_sleep`. Your output should be as follows:

```shell
backtrace:
0x0000000080002cda
0x0000000080002bb6
0x0000000080002898
```

After `bttest` exit qemu. In your terminal: the addresses may be slightly different but if you run `addr2line -e kernel/kernel` (or `riscv64-unknown-elf-addr2line -e kernel/kernel`) and cut-and-paste the above addresses as follows:

```shell
$ addr2line -e kernel/kernel
0x0000000080002de2
0x0000000080002f4a
0x0000000080002bfc
Ctrl-D
```

You should see something like this:

```shell
kernel/sysproc.c:74
kernel/syscall.c:224
kernel/trap.c:85
```

The compiler puts in each stack frame a frame pointer that holds the address of the caller's frame pointer. Your `backtrace` should use these frame pointers to walk up the stack and print the saved return address in each stack frame.

Some hints:

- Add the prototype for`backtrace` to `kernel/defs.h` so that you can invoke `backtrace` in `sys_sleep`.

- The `GCC` compiler stores the frame pointer of the currently executing function in the register`s0`. Add the following function to`kernel/riscv.h`:

	```c
	static inline uint64
	r_fp()
	{
	  uint64 x;
	  asm volatile("mv %0, s0" : "=r" (x) );
	  return x;
	}
	```

	and call this function in backtrace to read the current frame pointer. This function uses in-line assembly

	 to read `s0`

- These [lecture notes](https://pdos.csail.mit.edu/6.828/2021/lec/l-riscv-slides.pdf) have a picture of the layout of stack frames. Note that the return address lives at a fixed offset (-8) from the frame pointer of a stackframe, and that the saved frame pointer lives at fixed offset (-16) from the frame pointer.

- `Xv6` allocates one page for each stack in the xv6 kernel at PAGE-aligned address. You can compute the top and bottom address of the stack page by using `PGROUNDDOWN(fp)` and `PGROUNDUP(fp)` (see `kernel/riscv.h`. These number are helpful for `backtrace` to terminate its loop.

Once your `backtrace` is working, call it from `panic` in `kernel/printf.c` so that you see the kernel's `backtrace `when it panics.



## 过程

实现一个 `backtrace()`，展现错误发生时的函数调用栈

`bttest`->`sleep`->`backtrace`



### step1：声明原型

在`kernel/def.h`中声明`backtrace()` 原型

```c
void backtrace();
```



### step2：读取 frame pointer

The `GCC` compiler stores the frame pointer of the currently executing function in the register`s0`. Add the following function to`kernel/riscv.h`:

`GCC` 编译器在`s0`寄存器中存储了当前运行的函数的`frame pointer`，将下面的函数添加到`kernel/riscv.h`中

```c
//使用内联汇编读取 s0寄存器，即当前调用函数的 frame pointer
static inline uint64
r_fp()
{
  uint64 x;
  asm volatile("mv %0, s0" : "=r" (x) );
  return x;
}
```



### step3：了解 stack frame

![image-20220515094814647](https://s2.loli.net/2022/05/15/4xaYqusUhlmztEj.png)



* 返回地址` Return Address` ： 位于距堆栈帧的帧指针固定偏移（-8）的位置
* 已保存的帧指针` Prev. Frame(fp)`：位于距帧指针固定偏移（-16）的位置。



### step4：栈的页

`Xv6`给每个内核里的页对齐`PAGE-aligned`位置给每个栈分配了一个页。你可以通过使用 `PGROUNDUP(fp)`和`PGROUNDDOWN(fp)`来计算这个页的顶部地址和底部地址

参阅`kernel / riscv.h`,这些数字有助于回溯以终止其循环。

```c
#define PGSIZE 4096 // bytes per page
#define PGSHIFT 12  // bits of offset within a page

#define PGROUNDUP(sz)  (((sz)+PGSIZE-1) & ~(PGSIZE-1))
#define PGROUNDDOWN(a) (((a)) & ~(PGSIZE-1))
```



### step5：添加加 backtrace

在`kernel/printf.c`的`panic`中添加`backtrace`调用，这样就能在 panic 的时候调用 `backtrace`

```c
void
panic(char *s)
{
  pr.locking = 0;
  printf("panic: ");
  printf(s);
  printf("\n");
  backtrace();
  panicked = 1; // freeze uart output from other CPUs
  for(;;)
    ;
}
```

根据题目要求还要在`sys_sleep`中添加 `backtrace()`

```c
uint64
sys_sleep(void)
{
  int n;
  uint ticks0;

  backtrace();

  if(argint(0, &n) < 0)
    return -1;
  acquire(&tickslock);
  ticks0 = ticks;
  while(ticks - ticks0 < n){
    if(myproc()->killed){
      release(&tickslock);
      return -1;
    }
    sleep(&ticks, &tickslock);
  }
  release(&tickslock);
  return 0;
}

```



### step6：实现 backtrace

首先通过`PGROUNDUP`和`PGROUNDDOWN`获取页的顶部地址和底部地址

```c
uint64 fp = r_fp();
uint64 high = PGROUNDUP(fp);
uint64 low = PGROUNDDOWN(fp);
```

在调用的时候，栈是从高地址向低地址增长的

`fp-8`可以得到当前调用函数的返回地址

`fp-16`可以得到调用这个函数的函数的`fp`，也就是`To Prev.Frame`，这个会指向高地址处的前一个`stack frame`

如果`fp >= high`那就说明已经到头了

```c
void backtrace()
{
	uint64 fp = r_fp();
	uint64 high = PGROUNDUP(fp);
	uint64 low = PGROUNDDOWN(fp);
	while(fp >= low && fp < high) {
		printf("%p\n",*((uint64*)(fp - 8)));
		fp = *((uint64*)(fp - 16));
	}
}
```

