# OS 02 几行汇编几行C：实现一个最简单的的内核

## PC机的引导程序

我们先借助GRUB引导程序，来学习一下

Hello OS的引导流程

1. PC机加电
2. PC机BIOS固件（固化在主板上的ROM芯片中）
3. 加载可引导设备中的GRUB（0x7c00 处的程序）
4. 加载磁盘分区中的Hello OS文件
5. Hello OS



PC机上电后第一条指令就是在BIOS固件中，负责检测和初始化CPU、内存、主板平台，然后加载引导设备(一般为硬盘)中的第一个扇区数据，到0x7c00 地址开始的内存空间，再执行0x7c00处执行指令（GRUB引导程序）



## Hello OS 引导汇编代码

使用汇编语言而非C语言：C语言不能直接操作特定的硬件，C语言的函数调用和传参都要使用栈

我们得先用汇编语言处理好C语言的工作环境，主要步骤如下

```c
;彭东 @ 2021.01.09
MBT_HDR_FLAGS EQU 0x00010003
MBT_HDR_MAGIC EQU 0x1BADB002 ;多引导协议头魔数
MBT_HDR2_MAGIC EQU 0xe85250d6 ;第二版多引导协议头魔数
global _start ;导出_start符号
extern main ;导入外部的main函数符号
[section .start.text] ;定义.start.text代码节
[bits 32] ;汇编成32位代码
_start:
jmp _entry
ALIGN 8
mbt_hdr:
dd MBT_HDR_MAGIC
dd MBT_HDR_FLAGS
dd -(MBT_HDR_MAGIC+MBT_HDR_FLAGS)
dd mbt_hdr
dd _start
dd 0
dd 0
dd _entry
;以上是GRUB所需要的头
ALIGN 8
mbt2_hdr:
DD MBT_HDR2_MAGIC
DD 0
DD mbt2_hdr_end - mbt2_hdr
DD -(MBT_HDR2_MAGIC + 0 + (mbt2_hdr_end - mbt2_hdr))
DW 2, 0
DD 24
DD mbt2_hdr
DD _start
DD 0
DD 0
DW 3, 0
DD 12
DD _entry
DD 0
DW 0, 0
DD 8
mbt2_hdr_end:
;以上是GRUB2所需要的头
;包含两个头是为了同时兼容GRUB、GRUB2
ALIGN 8
_entry:
;关中断
cli
;关不可屏蔽中断
in al, 0x70
or al, 0x80
out 0x70,al
;重新加载GDT
lgdt [GDT_PTR]
jmp dword 0x8 :_32bits_mode
_32bits_mode:
;下面初始化C语言可能会用到的寄存器
mov ax, 0x10
mov ds, ax
mov ss, ax
mov es, ax
mov fs, ax
mov gs, ax
xor eax,eax
xor ebx,ebx
xor ecx,ecx
xor edx,edx
xor edi,edi
xor esi,esi
xor ebp,ebp
xor esp,esp
;初始化栈，C语言需要栈才能工作
mov esp,0x9000
;调用C语言函数main
call main
;让CPU停止执行指令
halt_step:
halt
jmp halt_step
GDT_START:
knull_dsc: dq 0
kcode_dsc: dq 0x00cf9e000000ffff
kdata_dsc: dq 0x00cf92000000ffff
k16cd_dsc: dq 0x00009e000000ffff
k16da_dsc: dq 0x000092000000ffff
GDT_END:
GDT_PTR:
GDTLEN dw GDT_END-GDT_START-1
GDTBASE dd GDT_START
```

1. 定义GRUB的多引导协议头，兼容GRUB1和GRUB2

2. 关掉中断，设定CPU的工作模式

3. 初始化CPU的寄存器和C语言的运行环境

4. CPU工作需要的数据

	

上面代码中的汇编代码基本含义

MOV指令是数据传送指令

XOR 指令在两个操作数的对应位之间进行（按位）逻辑异或（XOR）操作，并将结果存放在目标操作数中

JMP 指令无条件跳转到目标地址，该地址用代码标号来标识，并被汇编器转换为偏移 量。

EQU 伪指令把一个符号名称与一个整数表达式或一个任意文本连接起来

## Hello OS的主函数

汇编代码中并没有main函数体的代码，而是从外部引入了一个符号。因为这个函数是在main.c文件中的，分别由nasm和gcc编译成链接的模块，然后由LD链接器链接在一起，形成可执行文件



## 控制计算机的屏幕

显卡是输出设备，有很多种。

集显：集成在主板上

核显：做在CPU芯片内

独显：独立存在通过PCIE接口连接



它把屏幕分成 24 行，每行 80 个字符，把这（24*80）个位置映射到以 **0xb8000 地址**开始的内存中，每两个字节对应一个字符，其中一个字节是字符的 ASCII 码，另一个字节为字符的颜色值。



所以我们可以实现自己的printf函数了

```c
//彭东 @ 2021.01.09

void _strwrite(char* string)
{
    char* p_strdst = (char*)(0xb8000);
    while (*string)
    {

        *p_strdst = *string++;
        p_strdst += 2;
    }
    return;
}

void printf(char* fmt, ...)
{
    _strwrite(fmt);
    return;
}

```



## 编译和安装Hello OS

### 编译Hello OS

对于编译关系比较复杂的项目，可以采用make工具

![img](https://static001.geekbang.org/resource/image/cb/34/cbd634cd5256e372bcbebd4b95f21b34.jpg?wh=4378*4923)

```c++
#彭东 @ 2021.01.09

MAKEFLAGS = -sR
MKDIR = mkdir
RMDIR = rmdir
CP = cp
CD = cd
DD = dd
RM = rm

ASM             = nasm
CC              = gcc
LD              = ld
OBJCOPY = objcopy

ASMBFLAGS       = -f elf -w-orphan-labels
CFLAGS          = -c -Os -std=c99 -m32 -Wall -Wshadow -W -Wconversion -Wno-sign-conversion  -fno-stack-protector -fomit-frame-pointer -fno-builtin -fno-common  -ffreestanding  -Wno-unused-parameter -Wunused-variable
LDFLAGS         = -s -static -T hello.lds -n -Map HelloOS.map 
OJCYFLAGS       = -S -O binary

HELLOOS_OBJS :=
HELLOOS_OBJS += entry.o main.o vgastr.o
HELLOOS_ELF = HelloOS.elf
HELLOOS_BIN = HelloOS.bin

.PHONY : build clean all link bin

all: clean build link bin

clean:
        $(RM) -f *.o *.bin *.elf

build: $(HELLOOS_OBJS)

link: $(HELLOOS_ELF)
$(HELLOOS_ELF): $(HELLOOS_OBJS)
        $(LD) $(LDFLAGS) -o $@ $(HELLOOS_OBJS)
bin: $(HELLOOS_BIN)
$(HELLOOS_BIN): $(HELLOOS_ELF)
        $(OBJCOPY) $(OJCYFLAGS) $< $@

%.o : %.asm
        $(ASM) $(ASMBFLAGS) -o $@ $<
%.o : %.c
        $(CC) $(CFLAGS) -o $@ $<

```



### 安装Hello OS

查看boot目录挂载的分区

```c++
df /boot/
```



```c++
menuentry 'HelloOS' {
     insmod part_msdos #GRUB加载分区模块识别分区
     insmod ext2 #GRUB加载ext文件系统模块识别ext文件系统
     set root='hd0,msdos3' #注意boot目录挂载的分区，这是我机器上的情况
     multiboot2 /boot/HelloOS.bin #GRUB以multiboot2协议加载HelloOS.bin
     boot #GRUB启动HelloOS.bin
}
```





## 自问自答（核心）

### 1.更先进的UEFI BIOS如何引导？

### 2.能否使用其他语言来写操作系统？
