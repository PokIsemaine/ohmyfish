# Lab4 trap

## RISC-V assembly ([easy](https://pdos.csail.mit.edu/6.S081/2021/labs/guidance.html))

It will be important to understand a bit of RISC-V assembly, which you were exposed to in 6.004. There is a file `user/call.c` in your xv6 repo. make fs.img compiles it and also produces a readable assembly version of the program in `user/call.asm`.

Read the code in call.asm for the functions `g`, `f`, and `main`. The instruction manual for RISC-V is on the [reference page](https://pdos.csail.mit.edu/6.S081/2021/reference.html). Here are some questions that you should answer (store the answers in a file answers-traps.txt):

Which registers contain arguments to functions? For example, which register holds 13 in main's call to `printf`?

Where is the call to function `f` in the assembly code for main? Where is the call to `g`? (Hint: the compiler may inline functions.)

At what address is the function `printf` located?

What value is in the register `ra` just after the `jalr` to `printf` in `main`?

Run the following code.

``` c
unsigned int i = 0x00646c72;
printf("H%x Wo%s", 57616, &i);
```

What is the output? [Here's an ASCII table](http://web.cs.mun.ca/~michael/c/ascii-table.html) that maps bytes to characters.

The output depends on that fact that the RISC-V is little-endian. If the RISC-V were instead big-endian what would you set `i` to in order to yield the same output? Would you need to change `57616` to a different value?

[Here's a description of little- and big-endian](http://www.webopedia.com/TERM/b/big_endian.html) and [a more whimsical description](http://www.networksorcery.com/enp/ien/ien137.txt).

In the following code, what is going to be printed after `'y='`? (note: the answer is not a specific value.) Why does this happen?

```c
printf("x=%d y=%d", 3);  
```



## 题意

本实验主要是为了理解 RISC-V 的汇编代码，回答几个简单的问题



## 过程

$ <kbd>make fs.img</kbd>程序`user/call.c`的汇编版本`user/call.asm`



`user/call.c`

```c
#include "kernel/param.h"
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"

int g(int x) {
  return x+3;
}

int f(int x) {
  return g(x);
}

void main(void) {
  printf("%d %d\n", f(8)+1, 13);
  exit(0);
}
```



`user/call.asm`部分

```assembly
0000000000000000 <g>:
#include "kernel/param.h"
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"

int g(int x) {
   0:	1141                	addi	sp,sp,-16
   2:	e422                	sd	s0,8(sp)
   4:	0800                	addi	s0,sp,16
  return x+3;
}
   6:	250d                	addiw	a0,a0,3
   8:	6422                	ld	s0,8(sp)
   a:	0141                	addi	sp,sp,16
   c:	8082                	ret

000000000000000e <f>:

int f(int x) {
   e:	1141                	addi	sp,sp,-16
  10:	e422                	sd	s0,8(sp)
  12:	0800                	addi	s0,sp,16
  return g(x);
}
  14:	250d                	addiw	a0,a0,3
  16:	6422                	ld	s0,8(sp)
  18:	0141                	addi	sp,sp,16
  1a:	8082                	ret

000000000000001c <main>:

void main(void) {
  1c:	1141                	addi	sp,sp,-16
  1e:	e406                	sd	ra,8(sp)
  20:	e022                	sd	s0,0(sp)
  22:	0800                	addi	s0,sp,16
  printf("%d %d\n", f(8)+1, 13);
  24:	4635                	li	a2,13
  26:	45b1                	li	a1,12
  28:	00000517          	auipc	a0,0x0
  2c:	7a050513          	addi	a0,a0,1952 # 7c8 <malloc+0xe8>
  30:	00000097          	auipc	ra,0x0
  34:	5f8080e7          	jalr	1528(ra) # 628 <printf>
  exit(0);
  38:	4501                	li	a0,0
  3a:	00000097          	auipc	ra,0x0
  3e:	274080e7          	jalr	628(ra) # 2ae <exit>
```



### Question1：

Which registers contain arguments to functions? For example, which register holds 13 in main's call to `printf`?

哪些寄存器包含函数的参数？ 例如，哪个寄存器在main对printf的调用中保留13？

### Answer1：

可以看 Lab 给的参考资料 [RISC-V unprivileged instructions](https://github.com/riscv/riscv-isa-manual/releases/download/draft-20200727-8088ba4/riscv-spec.pdf) 

Chapter25 RISC-V Assembly Programmer’s Handbook

![image-20220509103113738](https://s2.loli.net/2022/05/09/diIPRTrVY3KotWL.png)

通过这张表可以知道 `a0-a7` 均可作为函数参数传递，在生成的汇编代码中使用了`a1`存储`f(8)+1=12`，使用`a2`存储`13`



### Question2：

Where is the call to function `f` in the assembly code for main? Where is the call to `g`? (Hint: the compiler may inline functions.)

在main的汇编代码中对函数f的调用在哪里？对函数g的调用在哪里？ （提示：编译器可以内联函数。）



### Answer2：

没有，直接内联优化算出来`f(8)+1 = 12`



### Question3：

At what address is the function `printf` located?

`printf`函数位于哪个地址？



### Answer3：

```assembly
30:	00000097          	auipc	ra,0x0
34:	5f8080e7          	jalr	1528(ra) # 628 <printf>
```

关于 `AUIPC`

> 资料：[RISC-V unprivileged instructions](https://github.com/riscv/riscv-isa-manual/releases/download/draft-20200727-8088ba4/riscv-spec.pdf) 
>
> 章：Chapter 5 RV64I Base Integer Instruction Set, Version 2.1
>
> 小节：5.2 Integer Computational Instructions
>
> AUIPC (add upper immediate to pc) uses the same opcode as RV32I. AUIPC is used to build pcrelative addresses and uses the U-type format. AUIPC forms a 32-bit offset from the U-immediate, filling in the lowest 12 bits with zeros, sign-extends the result to 64 bits, adds it to the address of the AUIPC instruction, then places the result in register rd.
>
> ![image-20220509111827627](https://s2.loli.net/2022/05/09/AGZ5dgSlWToXFn1.png)

`auipc ra,0x0`：将立即数向左移动 12 位(给下面的 `JARL`指令腾地方），然后加上 `pc` 寄存器的值，赋值给 `ra` 寄存器

`ra = (0x0)<<(12) + pc = pc = 0x30 `



关于 `JARL`

>资料：[RISC-V unprivileged instructions](https://github.com/riscv/riscv-isa-manual/releases/download/draft-20200727-8088ba4/riscv-spec.pdf) 
>
>章：Chapter2 RV32I Base Integer Instruction Set, Version 2.1(64 位的那个章节只列出了和32位不同的，所以直接去32位的看)
>
>小节：2.5 Control Transfer Instructions
>
>The indirect jump instruction JALR (jump and link register) uses the I-type encoding. The target address is obtained by adding the sign-extended 12-bit I-immediate to the register rs1, then setting the least-significant bit of the result to zero. The address of the instruction following the jump (pc+4) is written to register rd. Register x0 can be used as the destination if the result is not required.
>
>![image-20220509112024691](https://s2.loli.net/2022/05/09/y1HrevDgCWJ3F4f.png)



JARL 用于跳转和链接，具体位置如下

`1528(ra) = 1528 + ra = 1528 + 0x30 = 0x628`

代码中搜索 628 可以确认 printf 确实在这个位置

```assembly
0000000000000628 <printf>:

void
printf(const char *fmt, ...)
{
 628:	711d                	addi	sp,sp,-96
 62a:	ec06                	sd	ra,24(sp)
 62c:	e822                	sd	s0,16(sp)
 62e:	1000                	addi	s0,sp,32
 630:	e40c                	sd	a1,8(s0)
 632:	e810                	sd	a2,16(s0)
 634:	ec14                	sd	a3,24(s0)
 636:	f018                	sd	a4,32(s0)
 638:	f41c                	sd	a5,40(s0)
 63a:	03043823          	sd	a6,48(s0)
 63e:	03143c23          	sd	a7,56(s0)
  va_list ap;

  va_start(ap, fmt);
 642:	00840613          	addi	a2,s0,8
 646:	fec43423          	sd	a2,-24(s0)
  vprintf(1, fmt, ap);
 64a:	85aa                	mv	a1,a0
 64c:	4505                	li	a0,1
 64e:	00000097          	auipc	ra,0x0
 652:	dce080e7          	jalr	-562(ra) # 41c <vprintf>
}
 656:	60e2                	ld	ra,24(sp)
 658:	6442                	ld	s0,16(sp)
 65a:	6125                	addi	sp,sp,96
 65c:	8082                	ret
```



### Question4：

What value is in the register `ra` just after the `jalr` to `printf` in `main`?

寄存器`ra`中`jalr`到`main`中的`printf`之后有什么值？



### Answer4：

The address of the instruction following the jump (pc+4) is written to register rd

跳转后的指令地址 (pc+4) 被写入寄存器 rd

`ra = pc + 4 = 0x38`



### Code1

Run the following code.

``` c
unsigned int i = 0x00646c72;
printf("H%x Wo%s", 57616, &i);
```

What is the output? [Here's an ASCII table](http://web.cs.mun.ca/~michael/c/ascii-table.html) that maps bytes to characters.

The output depends on that fact that the RISC-V is little-endian. If the RISC-V were instead big-endian what would you set `i` to in order to yield the same output? Would you need to change `57616` to a different value?

[Here's a description of little- and big-endian](http://www.webopedia.com/TERM/b/big_endian.html) and [a more whimsical description](http://www.networksorcery.com/enp/ien/ien137.txt).

```c
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"

int main() {
	unsigned int i = 0x00646c72;
    /
	printf("H%x Wo%s", 57616, &i);
	//57616 十六进制 E110
    //i 小端序 72 6c 64 00
    //         r l  d  Null char
    //如果大端序 i = 0x726c6400，57616 不用改
	return 0;
}
```

![image-20220509113816983](../../../../../../.config/Typora/typora-user-images/image-20220509113816983.png)

### Code2

In the following code, what is going to be printed after `'y='`? (note: the answer is not a specific value.) Why does this happen?

```c
printf("x=%d y=%d", 3);  
```

```c
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"

int main() {
	printf("x=%d y=%d", 3);  
	
	return 0;
}
```

可以参考[riscv 架构 printf 传参研究](http://www.lujun.org.cn/?p=4518)

`printf()`在这个用例中传3个参数

`a0`：字符串地址

`a1`：第一个参数

`a2`：第二个参数，在本例中缺失，所以 y 的值就是 `a2`寄存器里的值

`

![image-20220509113911854](../../../../../../.config/Typora/typora-user-images/image-20220509113911854.png)



来看看汇编代码验证一下吧



## 关于 mkfs & fs.img

