# Page tables

## 问题关联图



## 梳理





### 问题1 为什么要创造虚拟内存？

创造虚拟内存的一个出发点是你可以通过它实现隔离性。如果你正确的设置了page table，并且通过代码对它进行正确的管理，那么原则上你可以实现强隔离。



### 什么是虚拟内存？

物理内存是指DRAM中的存储单元。物理内存以一个字节为单位划为地址，称为物理地址。指令只使用虚拟地址，页式硬件将其转换为物理地址，然后将其发送到DRAM硬件来进行读写。与物理内存和虚拟地址不同，虚拟内存不是物理对象，而**是指内核提供的管理物理内存和虚拟地址的抽象和机制的集合。**



### 为什么要强隔离？

不隔离会怎么样



隔离的作用？

Each process has its own memory • Can read and write its own memory • But cannot read or write the kernel’s memory or another process’ memory



## 问题2 如何实现地址空间？

换句话说如何在一个物理内存上，创建不同的地址空间？最常见的方法，同时也是非常灵活的一种方法就是使用页表（Page Tables）

### 什么是页表？



### 如何实现页表？

页表是在硬件中通过处理器和内存管理单元（Memory Management Unit）实现

![image-20220712215703498](https://s2.loli.net/2022/07/12/S9wfbpcDxKqCGlU.png)

对于任何一条带有地址的指令，其中的地址应该认为是虚拟内存地址而不是物理地址。假设寄存器a0中是地址0x1000，那么这是一个虚拟内存地址。虚拟内存地址会被转到内存管理单元（MMU，Memory Management Unit）

从CPU的角度来说，一旦MMU打开了，它执行的每条指令中的地址都是虚拟内存地址。

为了能够完成虚拟内存地址到物理内存地址的翻译，MMU会有一个表单，表单中，一边是虚拟内存地址，另一边是物理内存地址。举个例子，虚拟内存地址0x1000对应了一个我随口说的物理内存地址0xFFF0。这样的表单可以非常灵活。



### 页表存在哪里？

通常来说，内存地址对应关系的表单也保存在内存中。所以CPU中需要有一些寄存器用来存放表单在物理内存中的地址。现在，在内存的某个位置保存了地址关系表单，我们假设这个位置的物理内存地址是0x10。那么在RISC-V上一个叫做SATP的寄存器会保存地址0x10。

这样，CPU就可以告诉MMU，可以从哪找到将虚拟内存地址翻译成物理内存地址的表单。

**所以MMU并不会保存page table，它只会从内存中读取page table，然后完成翻译**



每个应用程序都有自己独立的表单，并且这个表单定义了应用程序的地址空间。所以当操作系统将CPU从一个应用程序切换到另一个应用程序时，同时也需要切换SATP寄存器中的内容，从而指向新的进程保存在物理内存中的地址对应表单。这样的话，cat程序和Shell程序中相同的虚拟内存地址，就可以翻译到不同的物理内存地址，因为每个应用程序都有属于自己的不同的地址对应表单。







### 表单的大小问题如何解决？

问题：为每个地址创建一条表单条目会导致表单变得非常巨大，导致所有内存都被表单消耗殆尽



第一步：不要为每个地址创建一条表单条目，而是为每个page创建一条表单条目

所以每一次地址翻译都是针对一个page。而RISC-V中，一个page是4KB，也就是4096Bytes。并且这 4096 Bytes 在物理内存中也是连续的



![img](http://xv6.dgs.zone/tranlate_books/book-riscv-rev1/images/c3/p1.png)

首先对于虚拟内存地址，我们将它划分为两个部分，index和offset，index用来查找page，offset对应的是一个page中的哪个字节。



现在一个进程需要多达的表项呢？



![img](http://xv6.dgs.zone/tranlate_books/book-riscv-rev1/images/c3/p2.png)

### What about a recursive mapping?



### How do we program the MMU?



## What about segmentation?



## Why use virtual memory while in kernel?



## What permissions for trampoline?

![image-20220712224314631](https://s2.loli.net/2022/07/12/K9nsDhTjbZlzLQo.png)

## 问题 进程如何分配内存