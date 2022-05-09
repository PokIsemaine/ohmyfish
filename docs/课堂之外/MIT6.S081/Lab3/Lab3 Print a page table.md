# Lab3 page table

## Print a page table

To help you visualize RISC-V page tables, and perhaps to aid future debugging, your second task is to write a function that prints the contents of a page table.

Define a function called `vmprint()`. It should take a `pagetable_t` argument, and print that pagetable in the format described below. Insert `if(p->pid==1) vmprint(p->pagetable)` in exec.c just before the `return argc`, to print the first process's page table. You receive full credit for this part of the lab if you pass the `pte printout` test of `make grade`.

Now when you start xv6 it should print output like this, describing the page table of the first process at the point when it has just finished `exec()`ing `init`:

```
page table 0x0000000087f6e000
 ..0: pte 0x0000000021fda801 pa 0x0000000087f6a000
 .. ..0: pte 0x0000000021fda401 pa 0x0000000087f69000
 .. .. ..0: pte 0x0000000021fdac1f pa 0x0000000087f6b000
 .. .. ..1: pte 0x0000000021fda00f pa 0x0000000087f68000
 .. .. ..2: pte 0x0000000021fd9c1f pa 0x0000000087f67000
 ..255: pte 0x0000000021fdb401 pa 0x0000000087f6d000
 .. ..511: pte 0x0000000021fdb001 pa 0x0000000087f6c000
 .. .. ..509: pte 0x0000000021fdd813 pa 0x0000000087f76000
 .. .. ..510: pte 0x0000000021fddc07 pa 0x0000000087f77000
 .. .. ..511: pte 0x0000000020001c0b pa 0x0000000080007000
  
```

The first line displays the argument to `vmprint`. After that there is a line for each PTE, including PTEs that refer to page-table pages deeper in the tree. Each PTE line is indented by a number of `" .."` that indicates its depth in the tree. Each PTE line shows the PTE index in its page-table page, the pte bits, and the physical address extracted from the PTE. Don't print PTEs that are not valid. In the above example, the top-level page-table page has mappings for entries 0 and 255. The next level down for entry 0 has only index 0 mapped, and the bottom-level for that index 0 has entries 0, 1, and 2 mapped.

Your code might emit different physical addresses than those shown above. The number of entries and the virtual addresses should be the same.

Some hints:

- You can put `vmprint()` in `kernel/vm.c`.
- Use the macros at the end of the file kernel/riscv.h.
- The function `freewalk` may be inspirational.
- Define the prototype for `vmprint` in kernel/defs.h so that you can call it from exec.c.
- Use `%p` in your printf calls to print out full 64-bit hex PTEs and addresses as shown in the example.

Explain the output of `vmprint` in terms of Fig 3-4 from the text. What does page 0 contain? What is in page 2? When running in user mode, could the process read/write the memory mapped by page 1? What does the third to last page contain?



## 题意

打印页表，打印信息含义见下图

![image-20220428082000572](https://s2.loli.net/2022/04/28/DLV6fmHGQdZ1y4W.png)

In the above example, the top-level page-table page has mappings for entries 0 and 255. The next level down for entry 0 has only index 0 mapped, and the bottom-level for that index 0 has entries 0, 1, and 2 mapped.

可以得到多级页表大概长这样

![pgtbl](https://s2.loli.net/2022/04/28/ekYTsHq2cn8NIOp.png)



## 过程

### step1 PTE2PA & PTE_FLAGS

模拟一下从PTE获取PA

```C
//pte 0x0000000021fda801 pa 0x0000000087f6a000
#define PTE2PA(pte) (((pte) >> 10) << 12)
```

使用python模拟一下这个转换

```python
>>> hex(int("0x0000000021fda801",16)>>10<<12)
'0x87f6a000'
```

模拟一下从PTE获取PA

```c
//pte 0x0000000021fda801
#define PTE_FLAGS(pte) ((pte) & 0x3FF)
```

使用python模拟一下这个转换

```python
>>> hex(int("0x0000000021fda801",16) & 0x3FF)
'0x1'
```



### step 2 写个小工具吧

经过上次的模拟大概知道了如何从PTE中分离出`PA`和`FALGS`，不过感觉有些麻烦，那么就写个小工具吧

![image-20220428092855643](https://s2.loli.net/2022/04/28/h1qGeWPCU6ZMF5y.png)

```python
#!/usr/zsh/python
# -*- coding:UTF-8 -*-

import  sys

PTA = int(sys.argv[1],16)
PTE_PA = hex(PTA>>10<<12)
PTE_FLAGS = PTA & 0x3ff

print("PTE",hex(PTA))
print("PNN",PTE_PA)
print("Flags",bin(PTE_FLAGS))

FLAGS = ["V-Valid",
        "R-Readable",
        "W-Writeable",
        "X-Executable",
        "U-User",
        "G-Global",
        "A-Accessed",
        "D-Dirty(0 in page directory)",
        "RSW-Reserved for supervisor software"]

for i in range (0,10):
    if(PTE_FLAGS & (1 << i)):
        print(FLAGS[i])

```

使用方法

```shell
python PTE.py 0x0000000021fda801
//output
PTE 0x21fda801
PNN 0x87f6a000
Flags 0b1
V-Valid
```



### step3 vmprint

模仿一下`freewalk`

```c
// Recursively free page-table pages.
// All leaf mappings must already have been removed.
void
freewalk(pagetable_t pagetable)
{
  // there are 2^9 = 512 PTEs in a page table.
  for(int i = 0; i < 512; i++){
    pte_t pte = pagetable[i];
    if((pte & PTE_V) && (pte & (PTE_R|PTE_W|PTE_X)) == 0){
      // this PTE points to a lower-level page table.
      uint64 child = PTE2PA(pte);
      freewalk((pagetable_t)child);
      pagetable[i] = 0;
    } else if(pte & PTE_V){
      panic("freewalk: leaf");
    }
  }
  kfree((void*)pagetable);
}
```



```c
void traversal_pt(pagetable_t pagetable, int level) {
	for(int i = 0 ; i < 512 ; ++i) {
		pte_t PTE = pagetable[i];
		if(PTE_FLAGS(PTE) & PTE_V) {
            // this PTE points to a lower-level page table.
			uint64 PNN = PTE2PA(PTE);
			if(level == 0) {
				printf("..%d: pte %p pa %p\n",i, PTE, PNN);
				traversal_pt((pagetable_t)PNN, level + 1);
			} else if(level == 1) {
				printf(".. ..%d: pte %p pa %p\n", i, PTE, PNN);
				traversal_pt((pagetable_t)PNN, level + 1);
			} else {
				printf(".. .. ..%d: pte %p pa %p\n", i, PTE, PNN);
			}
		}
	}
}
void vmprint(pagetable_t pt) {
	printf("page table %p\n ",pt);
	traversal_pt(pt, 0);
}
```



### step4 插入到 exec.c

Insert `if(p->pid==1) vmprint(p->pagetable)` in `exec.c` just before the `return argc`

```c
 if(p->pid == 1) vmprint(p->pagetable);
  return argc; // this ends up in a0, the first argument to main(argc, argv)
```

