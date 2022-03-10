# 标准分配器实现

[TOC]



## VC6 标准分配器之实现

### VC6 malloc()

![image-20220306195539143](https://s2.loli.net/2022/03/06/vuEyYo3F8zhApL5.png)

我们现在希望能够去除上下的`cookie`以节省内存

去除`cookie`的先决条件是区块要一样大小。容器里所有元素是同样的大小

### VC6 allocator

> VC6：哎，我就是不做内存管理设计，我就是玩

VC6 的 `allocator`没做任何的内存管理，只是把`malloc`换一个样貌。就是用`::operator new`和`::operator delete`完成任务罢了

VC里所有的容器的第二个模板参数都是这个偷懒的`allocator`，我们使用容器最终的元素分配都是靠`malloc`去做的，具体是以指定的元素类型为单位。

![image-20220306195633970](https://s2.loli.net/2022/03/06/A5sV9zxLhfOQmcv.png)



## BC5 标准库分配器之实现

BC5和VC6的标准库分配器实现基本完全一样

![image-20220306200148378](https://s2.loli.net/2022/03/06/JErDeYu3USQ5aOk.png)



## G2.9 分配器的实现

![image-20220306202426758](https://s2.loli.net/2022/03/06/ZoEUuK9nswLaYig.png)

G2.9的标准分配器和前面两个并没有太大区别，但是要注意到。`defalloc.h`里的标准库分配器并没有被任何容器包含，这很奇怪。

我们惊讶地发现，容器用的并不是`std::allocator`而是`std::alloc`

![image-20220306202904312](https://s2.loli.net/2022/03/06/e1VmFxY698KzSlD.png)

`alloc`是一个`class`，我们直接以`alloc::allocate()`的方式调用，说明`allocate()`和`deallocate()`是`allocate`里的静态函数。

另外值得注意的是，`alloc::allocate(512)`接受的是字节而非元素。



### 而今安在哉

到了`G4.9`我貌似找不到`G2.9`里的`std::alloc`了，这个好东西难道被去掉了吗？并没有，只是换了一个名字而已。`G4.9`标准库里有许多`extented allocators`其中的`_pool_alloc`就是`G2.9`的`alloc的化身

![image-20220306203337654](https://s2.loli.net/2022/03/06/lAyPk3FK4a9iC5H.png)



```cpp
//如果你想在G4.9使用__pool_alloc这种好东西，可以看下面这个例子
vector<string,__gun_cxx::__pool_alloc<string>>vec;
```



![image-20220306203720971](https://s2.loli.net/2022/03/06/VjrKnEkqFSM2X3I.png)



## G4.9 标准库分配器之实现

先前我们已经看到，`G2.9`的`std::alloc`到了`G4.9`变成体制外的`__pool_alloc`了。那么，`G4.9`体制内的标准分配器有什么变化呢？

![image-20220306204208002](https://s2.loli.net/2022/03/06/BiRTI2yJjokUFEq.png)

`G4.9`的`std::allocator`虽然多了个继承，但是换汤不换药啊（脑内开始范大将军名场面），还是没有做啥特别设计

### G4.9 pool allocator 用例

`pool allocator`相比`std::allocator`的好处就是可以消除`cookie`。

试想一下，一个元素`cookie`为`8 bytes`那么一百万个元素呢？省下的内存可不小啊！

![image-20220306205846984](https://s2.loli.net/2022/03/06/fIXh6RzSw5LsmyT.png)

```cpp
cout<<sizeof(__gun_cxx::__pool_alloc<int>)<<endl;//1,表明是个空类
```

## G2.9 std::alloc 运行模式

![image-20220307154648444](https://s2.loli.net/2022/03/07/SG91rsH5ZtmDk2F.png)

  16条链表分别负责8、16、24、32……字节，如果容器发出来的需求不是8的倍数，那么会调整到8的倍数。负责范围[8-128]，如果超过了这个范围，那么就会调用`malloc`来负责（malloc会有`cookie`，`std::alloc`没有`cookie`。



如果选了32字节的，那就挖一大块（20个，不要问为什么，或许是经验值）来作分割。实际上看到左边那个32和64的`free lit`我们可以看到他们是连着的，这意味着挖的时候其实挖了2*20个，一半用来分割，另一半的先不作处理做准备

![image-20220307161055680](https://s2.loli.net/2022/03/07/SQZpCgGjedNx1of.png)



## embedded pointers

![image-20220307161500782](https://s2.loli.net/2022/03/07/yaShqrbld2RmLKn.png)

如果，客户（容器）要这块，那么链表的第一块就给出去。给出去的时候，容器就开始把这一块当作元素空间往里面填值，覆盖掉里面的指针（前四个字节）。并且给的时候`free list`的头已经给到下一块了。

等到容器归还内存的时候，又把前面的四个字节当作指针

但是如果对象小于四个字节，就不能被借用了，不过这样的情况比较少



## G2.9 std::alloc 运行一瞥

![image-20220307164151912](https://s2.loli.net/2022/03/07/imlBc8vrzyQN7Df.png)



分配器的客户：容器

具体动作：看`pool`，先把`pool`里的挖到链表里。如果还不够就再去索求申请，注入`pool`

### 一帆风顺

STEP1：容器申请 32bytes，我们先看用于准备的`pool`。由于`pool`为空，所以我们用`malloc`索取（注意图里的`cookie`）并向`pool`注入$32*20*2+RoundUp(0>>4)=1280$，其中$RoundUp()$是用来调整到$16$ 的边界的追加量，而且这个追加量会越来越大。

然后前$19$个分到`free list`里，剩下1个用于备用。

![image-20220307164947434](https://s2.loli.net/2022/03/07/7mRrclH2XnjxyTw.png)

STEP2：接着上一步，我们接下来申请 64 bytes。先看 `pool`，因为STEP1之后`pool`里有 `640 bytes`，所以将其分为$640/64=10$个内存块，第一个给客户，剩下九个挂到`list#7`。注意`list#7`的不是`malloc`的所以没有`cookie`

![image-20220307165645739](https://s2.loli.net/2022/03/07/tCwSP5AxlcjReaW.png)

STEP3：接着上一步，我们接下来申请 96 bytes。先看 `pool`，因为STEP2之后`pool`里有 `0 bytes`，所以我们用`malloc`索取并向`pool`注入$96*20*2+RoundUp(1280>>4)=3920$。其中追加量，RoundUp($目前的累计申请量<<4$)会越来越多，这会导致一个安全阀门。其中19个区块给`list#11`，1个给用户，剩下2000备用。

![image-20220307170332486](https://s2.loli.net/2022/03/07/MjoOEY5aGk1Nslc.png)

STEP4：申请88，应该找第$88/8-1=10$号链表，由于`pool`剩下`2000`。所以取`pool`划分20个区块（尽量多但不能超20），第1个个给客户，剩下19个挂到`list#10`.`pool`剩下$2000-88*20=240$

![image-20220307171130077](https://s2.loli.net/2022/03/07/G7VewTpIDygd8K3.png)

STEP5：连续 3 次 申请 88。直接从 #10 链表给客户

![image-20220309182406128](https://s2.loli.net/2022/03/09/jWqsFh4Kvzytdob.png)

STEP6：申请8，由于 `pool` 有余量，所以取 `pool`划分 20 个区块，1个给客户，剩下19个挂于 `list #0`

![image-20220309182746346](https://s2.loli.net/2022/03/09/oc51zW7UZAiCqfp.png)

看了上面几张图片可能会迷惑我们的`pool`怎么跑来跑去的很麻烦啊，实际上我们写代码实现的时候只需要两个指针一指就可以了



STEP7：申请 $104$，`list #12`没有区块，`pool`余量又不足提供一个（成碎片了），于是先将`pool`剩下的碎片($80$)分给`list #9`。然后索取并成功获得$104*20*2+RoundUp(5200>>4)$，切出 $19$ 个给`list #12`最前面的那个给客户，剩下 `2048`备用

![image-20220309183419312](https://s2.loli.net/2022/03/09/DiFKL6nIwlekz7v.png)

STEP8：申请$112$，由于`pool`有余量，从`pool`中取 $20$ 个区块，$1$个给客户，留$19$个挂到`list #13`

![image-20220309183810325](https://s2.loli.net/2022/03/09/EqUBex6YdtXlG8K.png)

STEP9：申请 $48$，由于`pool`有余量，从`pool`中取$3$个区块，$1$个给客户，$2$个挂到`list #5`

![image-20220309184006345](https://s2.loli.net/2022/03/09/ROiAa1gmWQopYKc.png)

### 遭遇苦难

STEP10：申请 $72$，`pool`余量不足以供应$1$个，于是先处理碎片把`pool`给`list #2`。然后索取$72*20*2+RoundUp(9688>>4)$，但是现在已经没法从`sysytem heap`索取那么多了，于是`alloc`已经申请的资源里找最接近 的$80$ （`list #9`）回填给`pool` ，然后切出 $72$给客户，剩下$8$

![(image-20220309185109321)](https://s2.loli.net/2022/03/09/PhxeZKrmoaSbg5y.png)

STEP11：申请 $72$，`pool`余量不足以供应$1$个，于是先处理碎片把`pool`给`list #0`。然后索取$72*20*2+RoundUp(9688>>4)$，但是现在已经没法从`sysytem heap`索取那么多了，于是`alloc`已经申请的资源里找最接近 的$88$ （`list #9`已经空啦，所以是`list #10`）回填给`pool` ，然后切出 $72$给客户，剩下$16$

![image-20220309190757066](https://s2.loli.net/2022/03/09/dUNzb2grZRPXq6J.png)

### 山穷水尽

STEP13：申请 $120$，`pool`余量不足以供应$1$个，于是先处理碎片把`pool`给`list #1`。然后索取$72*20*2+RoundUp(9688>>4)$，但是现在已经没法从`sysytem heap`索取那么多了，于是`alloc`已经申请的资源里找最接近的，发现没有（`list #14`和`list #15`都是空的）结束了

![image-20220309191251340](https://s2.loli.net/2022/03/09/P38DpdxO9WnwH5g.png)

### 柳暗花明？

![image-20220309191621330](https://s2.loli.net/2022/03/09/5fXET2YiLIlDgc7.png)

`std::alloc`在 STEP12 就投降举白旗了。这另我们感到诧异，明明还有出路啊。

* 我们已经拿的资源里还有那么多没有的小区块，为什么不把他们合并了？技术上好像挺难的
* `system heap`还剩 $312$为什么不拿来用？将索取量不断折半直到可以申请

### 蓦然回首

来回顾一下`std::alloc`流程图吧

![](https://s2.loli.net/2022/03/09/BHrEi98KSD3FZPY.png)

## G2.9 std::alloc 源码剖析

暂时放一放实现细节

## G2.9 std::alloc 细节讨论

![image-20220309200546086](https://s2.loli.net/2022/03/09/9FrnSlkhOVmIqae.png)

### 小细节

部分代码可读性很差

* `#136`和`#210`，多级指针生命就不要连着写了

* `#197`和`207`准备变量和使用变量隔太远了，中间发生一些复杂的过程，这回让`debug`变得困难
* `#034`一言难尽，括号套娃了



`#208`、`#218`、`#255`的写法很奇怪，但实际上是值得学习的。这样写可以避免把`==`写成`=`，把字面量放左边可以让编译器帮你检查





### 大灾难

上面都是一些细节问题，现在看到`#212-#214`这块描述的是`malloc`失败的时候（柳暗花明的那节）的处理

注释的意思是，如果`malloc`失败了，会从已有的资源里找到最接近需求的拿来用。

但是我们不尝试更小的请求，因为这会在多进程机器里导致大灾难。

更小的请求也就是我们先前的疑惑：system heap`还剩 $312$为什么不拿来用？将索取量不断折半直到可以申请。

那么这个大灾难是什么？为什么说是多进程机器？

`std::alloc`的策略实际上是一种竭泽而渔的方法，这在单进程的情况下确实没啥大问题。但是多进程机器中并不是只有这一个进程，竭泽而渔会导致其他进程没法正常运行，现在想来 $RoudUp$ 附加量的不断增长大概也是为了这个吧。



### 物归原主

由于`dealloc`的先天缺陷，没有调用`free`或`delete`将内存还给操作系统。

先天缺陷：`std::alloc`成功的去掉了`cookie`节省了内存，但同时也因为`cookie`的缺失，我们没法知道应该释放多少内存了，也就没法把内存还给操作系统

后面我们会在`loki`的分配器里看到更好的设计
