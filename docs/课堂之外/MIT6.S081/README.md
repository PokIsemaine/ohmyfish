# MIT 6.s081

## 参考资料

xv6 翻译（2020） http://xv6.dgs.zone/

mit6.s081 课程翻译（2020） https://mit-public-courses-cn-translatio.gitbook.io/mit6-s081/

官方给的参考资料（2021）：https://pdos.csail.mit.edu/6.S081/2021/reference.html



## 笔记

> 2022/7/11 把原来的笔记删了重写了一边，因为原来基本是摘抄和翻译，最近感觉这样没意思，不过是用看起来很多的文字进行自我安慰罢了。
>
> 尝试采用问题驱动的模式来整理笔记



* Introduction
* OS organization and system calls
* Page tables
* GDB, calling conventions and stack frames RISC-V
* Isolation & system call entry/exit
* Page faults
* Interrupts
* Multiprocessors and locking
* File systems



## Lab 参考

### Lab util: Unix utilities

[sleep](课堂之外/MIT6.S081/Lab1/Lab1%20sleep.md)：尝试系统调用

[pingpong](/课堂之外/MIT6.S081/Lab1/Lab1%20pingpong.md)：利用 pipe 和 fork 实现进程通信

[primes](/课堂之外/MIT6.S081/Lab1/Lab1%20primes.md)：利用管道实现并发素数筛

[find](/课堂之外/MIT6.S081/Lab1/Lab1%20find.md)：实现简易版的 find

[xargs](/课堂之外/MIT6.S081/Lab1/Lab1%20xargs.md)：实现简易版的 xargs

------



### Lab syscall: System calls

[System call tracing](/课堂之外/MIT6.S081/Lab2/Lab2%20System%20call%20tracing.md)：实现简易版 strace 追踪系统调用

[Sysinfo](/课堂之外/MIT6.S081/Lab2/Lab2%20Sysinfo.md)：打印系统信息

---



### Lab pgtbl: Page tables

[Speed up System calls](/课堂之外/MIT6.S081/Lab3/Lab3%20Detecting%20which%20pages%20have%20been%20accessed.md)：加速系统调用

[Print a page table](/课堂之外/MIT6.S081/Lab3/Lab3%20Print%20a%20page%20table.md)：打印页表

[Detecting which pages have been accessed](课堂之外/MIT6.S081/Lab3/Lab3%20Speed%20up%20System%20calls.md)：检查一个页是否被访问过

---



### Lab traps: Traps

[RISC-V assembly](/课堂之外/MIT6.S081/Lab4/Lab4%20RISC-V%20assembly.md)：了解 RISC-V 汇编

---



### Lab: Copy-on-Write Fork for xv6

[Implement copy-on write]() 实现 Copy-on-Write Fork





## 杂项

[RISC-V tutorial for MIT6.S081]()

[GDB：状态机视角]()

[Makefile]()
