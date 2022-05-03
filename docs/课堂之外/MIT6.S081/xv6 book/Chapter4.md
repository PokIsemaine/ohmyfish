# Chapter4 Traps and system calls

## traps

* system call
* illegal instructions
* device interrupt



Xv6 handles all traps in the kernel; traps are not delivered to user code. 



trap handing

1. hardware action
2. assembly instructions
3. C functions
4. systemcall/device-driver service runtime



3 cases

* from user space
* from kernel space
* from interrupts

## RISC-V trap machinery

Privileged CPU features Two main parts: 

1. Registers that can only be accessed by kernel 
2. Instructions that can only be executed by kernel



| registers  | Description                                                  |
| ---------- | ------------------------------------------------------------ |
| `stvec`    | The kernel writes **the address of its trap handler** here; the RISC-V jumps to the address in `stvec` to handle a trap. |
| `sepc`     | When a trap occurs, RISC-V **saves the program counter here** (since the `pc` is then overwritten with the value in `stvec`). <br />The `sret` (return from trap) instruction copies `sepc` to the `pc`. The kernel can write `sepc` to control where `sret `goes. |
| `scause`   | RISC-V puts a number here that describes **the reason for the trap.** |
| `sscratch` | The kernel places a value here that comes in handy **at the very start of a trap handler.** |
| `sstatus`  | The `SIE` bit in `sstatus` **controls whether device interrupts are enabled.** If the kernel clears `SIE` the RISC-V will defer device interrupts until the kernel sets `SIE`. The `SPP` bit indicates whether a trap came from user mode or supervisor mode, and controls to what mode `sret `returns. |

TIP：`SIE` = Supervisor Interrupt Enable

![image-20220429225017408](https://s2.loli.net/2022/04/29/3nJ7WSZjkqiBXQy.png)



When it needs to force a trap, the RISC-V hardware does the following for all trap types (other than timer interrupts): 

1. If the trap is a device interrupt, and the sstatus `SIE` bit is clear, don’t do any of the following. 
2. Disable interrupts by clearing the `SIE` bit in sstatus. 
3.  Copy the pc to sepc. 
4. Save the current mode (user or supervisor) in the `SPP` bit in sstatus.
5. Set scause to reflect the trap’s cause. 
6. Set the mode to supervisor. 
7. Copy stvec to the pc .
8.  Start executing at the new pc.



## Traps from user space

A trap may occur while executing in user space if the user program makes a system call (`ecall` instruction), or does something illegal, or if a device interrupts. The high-level path of a trap from user space is `uservec`, then `usertrap`; and when returning, `usertrapret` and then `userret` .

![image-20220503102004912](https://s2.loli.net/2022/05/03/vY7dGyXV59wMiQf.png)



A major constraint on the design of xv6’s trap handling is the fact that **the RISC-V hardware does not switch page tables when it forces a trap.** This means that

* **the trap handler address in stvec must have a valid mapping in the user page table**, since that’s the page table in force when the trap handling code starts executing.
* Furthermore, xv6’s **trap handling code needs to switch to the kernel page table;** 
* in order to be able to continue executing after that switch, **the kernel page table must also have a mapping for the handler pointed to by stvec.**



**Xv6 satisfies these requirements using a trampoline page.**

*  The trampoline page contains **uservec, the xv6 trap handling code** that stvec points to. 

```assembly
	#
        # code to switch between user and kernel space.
        #
        # this code is mapped at the same virtual address
        # (TRAMPOLINE) in user and kernel space so that
        # it continues to work when it switches page tables.
	#
	# kernel.ld causes this to be aligned
        # to a page boundary.
        #
	.section trampsec
.globl trampoline
trampoline:
.align 4
.globl uservec
uservec:    
	#
        # trap.c sets stvec to point here, so
        # traps from user space start here,
        # in supervisor mode, but with a
        # user page table.
        #
        # sscratch points to where the process's p->trapframe is
        # mapped into user space, at TRAPFRAME.
        #
        
	# swap a0 and sscratch
        # so that a0 is TRAPFRAME
        csrrw a0, sscratch, a0

        # save the user registers in TRAPFRAME
        sd ra, 40(a0)
        sd sp, 48(a0)
        sd gp, 56(a0)
        sd tp, 64(a0)
        sd t0, 72(a0)
        sd t1, 80(a0)
        sd t2, 88(a0)
        sd s0, 96(a0)
        sd s1, 104(a0)
        sd a1, 120(a0)
        sd a2, 128(a0)
        sd a3, 136(a0)
        sd a4, 144(a0)
        sd a5, 152(a0)
        sd a6, 160(a0)
        sd a7, 168(a0)
        sd s2, 176(a0)
        sd s3, 184(a0)
        sd s4, 192(a0)
        sd s5, 200(a0)
        sd s6, 208(a0)
        sd s7, 216(a0)
        sd s8, 224(a0)
        sd s9, 232(a0)
        sd s10, 240(a0)
        sd s11, 248(a0)
        sd t3, 256(a0)
        sd t4, 264(a0)
        sd t5, 272(a0)
        sd t6, 280(a0)

	# save the user a0 in p->trapframe->a0
        csrr t0, sscratch
        sd t0, 112(a0)

        # restore kernel stack pointer from p->trapframe->kernel_sp
        ld sp, 8(a0)

        # make tp hold the current hartid, from p->trapframe->kernel_hartid
        ld tp, 32(a0)

        # load the address of usertrap(), p->trapframe->kernel_trap
        ld t0, 16(a0)

        # restore kernel page table from p->trapframe->kernel_satp
        ld t1, 0(a0)
        csrw satp, t1
        sfence.vma zero, zero

        # a0 is no longer valid, since the kernel page
        # table does not specially map p->tf.

        # jump to usertrap(), which does not return
        jr t0

.globl userret
userret:
        # userret(TRAPFRAME, pagetable)
        # switch from kernel to user.
        # usertrapret() calls here.
        # a0: TRAPFRAME, in user page table.
        # a1: user page table, for satp.

        # switch to the user page table.
        csrw satp, a1
        sfence.vma zero, zero

        # put the saved user a0 in sscratch, so we
        # can swap it with our a0 (TRAPFRAME) in the last step.
        ld t0, 112(a0)
        csrw sscratch, t0

        # restore all but a0 from TRAPFRAME
        ld ra, 40(a0)
        ld sp, 48(a0)
        ld gp, 56(a0)
        ld tp, 64(a0)
        ld t0, 72(a0)
        ld t1, 80(a0)
        ld t2, 88(a0)
        ld s0, 96(a0)
        ld s1, 104(a0)
        ld a1, 120(a0)
        ld a2, 128(a0)
        ld a3, 136(a0)
        ld a4, 144(a0)
        ld a5, 152(a0)
        ld a6, 160(a0)
        ld a7, 168(a0)
        ld s2, 176(a0)
        ld s3, 184(a0)
        ld s4, 192(a0)
        ld s5, 200(a0)
        ld s6, 208(a0)
        ld s7, 216(a0)
        ld s8, 224(a0)
        ld s9, 232(a0)
        ld s10, 240(a0)
        ld s11, 248(a0)
        ld t3, 256(a0)
        ld t4, 264(a0)
        ld t5, 272(a0)
        ld t6, 280(a0)

	# restore user a0, and save TRAPFRAME in sscratch
        csrrw a0, sscratch, a0
        
        # return to user mode and user pc.
        # usertrapret() set up sstatus and sepc.
        sret

```



* The trampoline page is mapped in every process’s page table **at address TRAMPOLINE, which is at the end of the virtual address space** so that it will be above memory that programs use for themselves.
*  The trampoline page is **also** mapped at address TRAMPOLINE **in the kernel page table**. 
* See Figure 2.3 and Figure 3.3. Because the trampoline page is mapped in the user page table, with the PTE_U flag, traps can start executing there in supervisor mode. Because **the trampoline page is mapped at the same address in the kernel address space, the trap handler can continue to execute after it switches to the kernel page table.**

![image-20220503100248783](https://s2.loli.net/2022/05/03/9ajYyoVOMqQUbEI.png)





## Code: Calling system calls

Chapter 2 ended with initcode.S invoking the exec system call (user/initcode.S:11). Let’s look at how the user call makes its way to the exec system call’s implementation in the kernel.

initcode.S places the arguments for exec in registers a0 and a1, and puts the system call number in a7. 

System call numbers match the entries in the syscalls array, a table of function pointers (kernel/syscall.c:108). 

The ecall instruction traps into the kernel and causes uservec, usertrap, and then syscall to execute, as we saw above. 

```assembly
# exec(init, argv)
.globl start
start:
        la a0, init
        la a1, argv
        li a7, SYS_exec
        ecall
```

syscall (kernel/syscall.c:133) retrieves the system call number from the saved a7 in the trapframe and uses it to index into syscalls.

For the first system call, a7 contains SYS_exec (kernel/syscall.h:8), resulting in a call to the system call implementation function sys_exec.

```c
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
};

void
syscall(void)
{
  int num;
  struct proc *p = myproc();

  num = p->trapframe->a7;
  if(num > 0 && num < NELEM(syscalls) && syscalls[num]) {
    p->trapframe->a0 = syscalls[num]();
  } else {
    printf("%d %s: unknown sys call %d\n",
            p->pid, p->name, num);
    p->trapframe->a0 = -1;
  }
}
```

 When sys_exec returns, syscall records its return value in p->trapframe->a0. This will cause the original user-space call to exec() to return that value, since the C calling convention on RISC-V places return values in a0. System calls conventionally return negative numbers to indicate errors, and zero or positive numbers for success. If the system call number is invalid, syscall prints an error and returns −1

![trapexec](https://s2.loli.net/2022/05/03/rpdNwvxy148lkUV.png)

## Code: System call arguments

![image-20220503111437715](https://s2.loli.net/2022/05/03/fqQH91VTubtJgPU.png)

System call implementations in the kernel need to find the arguments passed by user code. Because user code calls system call wrapper functions, the arguments are initially where the RISC-V C calling convention places them: **in registers**. The kernel trap code saves user registers to the current process’s trap frame, where kernel code can find them.



```c
static uint64
argraw(int n)
{
  struct proc *p = myproc();
  switch (n) {
  case 0:
    return p->trapframe->a0;
  case 1:
    return p->trapframe->a1;
  case 2:
    return p->trapframe->a2;
  case 3:
    return p->trapframe->a3;
  case 4:
    return p->trapframe->a4;
  case 5:
    return p->trapframe->a5;
  }
  panic("argraw");
  return -1;
}

// Fetch the nth 32-bit system call argument.
int
argint(int n, int *ip)
{
  *ip = argraw(n);
  return 0;
}

// Retrieve an argument as a pointer.
// Doesn't check for legality, since
// copyin/copyout will do that.
int
argaddr(int n, uint64 *ip)
{
  *ip = argraw(n);
  return 0;
}

// Fetch the nth word-sized system call argument as a null-terminated string.
// Copies into buf, at most max.
// Returns string length if OK (including nul), -1 if error.
int
argstr(int n, char *buf, int max)
{
  uint64 addr;
  if(argaddr(n, &addr) < 0)
    return -1;
  return fetchstr(addr, buf, max);
}



//sysfile.c
// Fetch the nth word-sized system call argument as a file descriptor
// and return both the descriptor and the corresponding struct file.
static int
argfd(int n, int *pfd, struct file **pf)
{
  int fd;
  struct file *f;

  if(argint(n, &fd) < 0)
    return -1;
  if(fd < 0 || fd >= NOFILE || (f=myproc()->ofile[fd]) == 0)
    return -1;
  if(pfd)
    *pfd = fd;
  if(pf)
    *pf = f;
  return 0;
}
```

![syscall_arg_1](https://s2.loli.net/2022/05/03/7drUeopm9GYiF5K.png)







Some system calls **pass pointers as arguments**, and the kernel must use those pointers to read or write user memory. The exec system call, for example, passes the kernel an array of pointers referring to string arguments in user space. 

These pointers pose two challenges. 

* First, **the user program may be buggy or malicious,** and may pass the kernel an invalid pointer or a pointer intended to trick the kernel into accessing kernel memory instead of user memory. 
* **Second, the xv6 kernel page table mappings are not the same as the user page table mappings**, so the kernel cannot use ordinary instructions to load or store from user-supplied addresses.

The kernel implements functions that safely transfer data to and from user-supplied addresses. 

fetchstr is an example (kernel/syscall.c:25). File system calls such as exec use fetchstr to retrieve string file-name arguments from user space. fetchstr calls copyinstr to do the hard work. 

copyinstr (kernel/vm.c:398) copies up to max bytes to dst from virtual address srcva in the user page table pagetable.

Since pagetable is not the current page table, copyinstr uses walkaddr (which calls walk) to look up srcva in pagetable, yielding physical address pa0.

 The kernel maps each physical RAM address to the corresponding kernel virtual address, so copyinstr can directly copy string bytes from pa0 to dst. walkaddr (kernel/vm.c:104) checks that the user-supplied virtual address is part of the process’s user address space, so programs cannot trick the kernel into reading other memory. A similar function, copyout, copies data from the kernel to a user-supplied address.

```c
// Fetch the uint64 at addr from the current process.
int
fetchaddr(uint64 addr, uint64 *ip)
{
  struct proc *p = myproc();
  if(addr >= p->sz || addr+sizeof(uint64) > p->sz)
    return -1;
  if(copyin(p->pagetable, (char *)ip, addr, sizeof(*ip)) != 0)
    return -1;
  return 0;
}

// Fetch the nul-terminated string at addr from the current process.
// Returns length of string, not including nul, or -1 for error.
int
fetchstr(uint64 addr, char *buf, int max)
{
  struct proc *p = myproc();
  int err = copyinstr(p->pagetable, buf, addr, max);
  if(err < 0)
    return err;
  return strlen(buf);
}
```

![fetchstr](https://s2.loli.net/2022/05/03/XnVxwUmvNQblrqY.png)

## Traps from kernel space



## Page-fault exceptions

