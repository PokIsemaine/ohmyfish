# Chapter7 Scheduling

## 7.1 Multiplexing

Xv6 multiplexes by switching each CPU from one process to another in two situations.

* First, xv6’s sleep and wakeup mechanism switches when a process waits for device or pipe **I/O** to complete, or waits for a child to exit, or waits in the sleep system call.
* Second, xv6 **periodically forces** a switch to cope with processes that compute for long periods without sleeping.



Implementing multiplexing poses a few challenges.

*  First, **how to switch from one process to another?** 
* Second, **how to force switches in a way that is transparent to user processes?** (hardware timer’s interrupts drive context switches. )
* Third, all of the CPUs switch among the same shared set of processes, and **a locking plan is necessary to avoid races**. 
* Fourth, a process’s memory and other **resources must be freed when the process exits, but it cannot do all of this itself** because (for example) it can’t free its own kernel stack while still using it.
* Fifth, each core of **a multi-core machine must remember which process it is executing** so that system calls affect the correct process’s kernel state. 
* Finally, sleep and wakeup allow a process to give up the CPU and wait to be woken up by another process or interrupt. **Care is needed to avoid races that result in the loss of wakeup notifications.**





## 7.2 Code: Context switching

![image-20220630201955691](https://s2.loli.net/2022/06/30/bTgjsYLfCpcy5QX.png)

The xv6 scheduler has **a dedicated thread (saved registers and stack) per CPU** because it is not safe for the scheduler execute on the old process’s kernel stack: some other core might wake the process up and run it, and it would be a disaster to use the same stack on two different cores.



Switching from one thread to another involves saving the old thread’s CPU registers, and restoring the previously-saved registers of the new thread; the fact that the stack pointer and program counter are saved and restored means that the CPU will switch stacks and switch what code it is executing



context

```c
// Saved registers for kernel context switches.
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
```





`kernel/swtch.S`

```assembly
# Context switch
#
#   void swtch(struct context *old, struct context *new);
# 
# Save current registers in old. Load from new.	


.globl swtch
swtch:
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
        
        ret
```

