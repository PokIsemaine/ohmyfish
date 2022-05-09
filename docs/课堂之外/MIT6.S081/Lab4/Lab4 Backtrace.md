# Lab4 trap

## Backtrace ([moderate](https://pdos.csail.mit.edu/6.S081/2021/labs/guidance.html))

For debugging it is often useful to have a `backtrace`: a list of the function calls on the stack above the point at which the error occurred.

Implement a `backtrace()` function in `kernel/printf.c`. Insert a call to this function in `sys_sleep`, and then run bttest, which calls `sys_sleep`. Your output should be as follows:

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

