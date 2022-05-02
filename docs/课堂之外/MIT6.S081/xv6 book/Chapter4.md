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

1. If the trap is a device interrupt, and the sstatus SIE bit is clear, don’t do any of the following. 
2. Disable interrupts by clearing the SIE bit in sstatus. 
3.  Copy the pc to sepc. 
4. Save the current mode (user or supervisor) in the SPP bit in sstatus.
5. Set scause to reflect the trap’s cause. 
6. Set the mode to supervisor. 
7. Copy stvec to the pc .
8.  Start executing at the new pc.



## Traps from user space



## Code: Calling system calls



## Code: System call arguments



## Traps from kernel space



## Page-fault exceptions

