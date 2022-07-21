# File systems

## 问题关联图



## 梳理



## 问题1 文件系统是什么？

### 什么是文件系统？



### 文件系统有什么用？

文件系统的目的是组织和存储数据。

文件系统通常支持用户和应用程序之间的数据共享，以及持久性，以便在重新启动后数据仍然可用。



### 文件系统需要做什么？

- 文件系统需要磁盘上的数据结构来**表示目录和文件名称树，记录保存每个文件内容的块的标识，以及记录磁盘的哪些区域是空闲的。**
- 文件系统必须**支持崩溃恢复（crash recovery）**。也就是说，如果发生崩溃（例如，电源故障），文件系统必须在重新启动后仍能正常工作。风险在于崩溃可能会中断一系列更新，并使磁盘上的数据结构不一致（例如，一个块在某个文件中使用但同时仍被标记为空闲）。
- **不同的进程可能同时在文件系统上运行**，因此文件系统代码必须协调以保持不变量。
- 访问磁盘的速度比访问内存慢几个数量级，因此文件系统必须**保持常用块的内存缓存**。



### xv6 的文件系统长什么样？

![image-20210207194130429](https://fanxiao.tech/assets/img/posts/MIT_6S081/image-20210207194130429.png)

- disk层：对virtio硬盘上的文件块进行读写操作

- buffer cache层：对磁盘文件块进行缓存，并确保只有1个内核进程能在一段时间内修改文件块上存储的数据。

- logging层：让更高的层级能够将对文件块的所有update打包到一个*transaction*中，从而能保证所有文件块能够在将要崩溃时原子地进行update

- inode层：为每个文件提供一个独一无二的*inode number*

- directory层：将每个文件夹作为一个特殊的inode，这个inode的内容是文件夹entry

- pathname层：将文件夹组织为层级，并通过递归查找来解析路径

- file descriptor层：将管道、设备等UNIX资源用文件系统进行抽象

	



![img](http://xyfjason.top/2021/11/30/xv6-mit-6-S081-2020-Lab9-fs/fs.png)



### xv6 文件系统如何工作？

启动XV6的过程中，调用了makefs指令，来创建一个文件系统

makefs创建了一个全新的磁盘镜像，在这个磁盘镜像中包含了我们在指令中传入的一些文件。makefs为你创建了一个包含这些文件的新的文件系统。

XV6总是会打印文件系统的一些信息，所以从指令的下方可以看出有46个meta block，其中包括了：

- boot block
- super block
- 30个log block
- 13个inode block
- 1个bitmap block

之后是954个data block。所以这是一个袖珍级的文件系统，总共就包含了1000个block。



**xv6 创建并写一个文件**

* 第一阶段是创建文件
* 第二阶段将“hi”写入文件
* 第三阶段将“\n”换行符写入到文件



![img](https://906337931-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-MHZoT2b_bcLghjAOPsJ%2F-MRjLOMOKliOOsweWquL%2F-MRnQIqwv24XHAFU0MIp%2Fimage.png?alt=media&token=faaf555e-9035-42a1-99a1-59ca11f2dafa)

创建文件

* write 33 表示在写 inode,第一个是为了标记 inode 将要被使用，第二个就是实际写入 inode 的内容
* write 46 向第一个属于根目录的 data block 写数据，表示正在向根目录创建一个新文件。向根目录增加了一个新的entry，其中包含了文件名x，以及我们刚刚分配的inode编号
* write 32 表示 更新 inode ，因为我们刚刚添加了16个字节的entry来代表文件x的信息导致根目录的大小变了
* 最后的 write 33 再次更新了文件 x 的 inode

向文件写入 "hi"

* write 45，这是更新bitmap。文件系统首先会扫描bitmap来找到一个还没有使用的data block，未被使用的data block对应bit 0。找到之后，文件系统需要将该bit设置为1，表示对应的data block已经被使用了。所以更新block 45是为了更新bitmap。
* 接下来的两次write 595表明，文件系统挑选了data block 595。所以在文件x的inode中，第一个direct block number是595。因为写入了两个字符，所以write 595被调用了两次。
* 最后的write 33是更新文件x对应的inode中的size字段，因为现在文件x中有了两个字符。



> 学生提问：block 595看起来在磁盘中很靠后了，是因为前面的block已经被系统内核占用了吗？
>
> Frans教授：我们可以看前面makefs指令，makefs存了很多文件在磁盘镜像中，这些都发生在创建文件x之前，所以磁盘中很大一部分已经被这些文件填满了。
>
> 学生提问：第二阶段最后的write 33是否会将block 595与文件x的inode关联起来？
>
> Frans教授：会的。这里的write 33会发生几件事情：首先inode的size字段会更新；第一个direct block number会更新。这两个信息都会通过write 33一次更新到磁盘上的inode中。





## 问题2 文件系统如何使用磁盘？



**文件系统的工作就是将所有的数据结构以一种能够在重启之后重新构建文件系统的方式，存放在磁盘上**



### 如何与磁盘连接？

常见的磁盘SSD和HDD。这两类存储虽然有着不同的性能，但是都在合理的成本上提供了大量的存储空间。SSD通常是0.1到1毫秒的访问时间，而HDD通常是在10毫秒量级完成读写一个disk block。

一些术语，它们是sectors和blocks。

- sector通常是磁盘驱动可以读写的最小单元，它过去通常是512字节。
- block通常是操作系统或者文件系统视角的数据。**它由文件系统定义**，在XV6中它是1024字节。所以XV6中一个block对应两个sector。通常来说一个block对应了一个或者多个sector。



这些存储设备连接到了电脑总线之上，总线也连接了CPU和内存。**一个文件系统运行在CPU上，将内部的数据存储在内存**，同时也会以读写block的形式存储在SSD或者HDD。这里的接口还是挺简单的，包括了read/write，然后以block编号作为参数。虽然我们这里描述的过于简单了，但是实际的接口大概就是这样。

![img](https://906337931-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-MHZoT2b_bcLghjAOPsJ%2F-MRhBFGrbi6iZo-luZ0H%2F-MRhy7Cd_NccmqjQhB8S%2Fimage.png?alt=media&token=7cc0655d-df6a-4d7b-9283-a4953d3f105a)

在内部，SSD和HDD工作方式完全不一样，但是对于硬件的抽象屏蔽了这些差异。磁盘驱动通常会使用一些标准的协议，例如PCIE，与磁盘交互。从上向下看磁盘驱动的接口，大部分的磁盘看起来都一样，你可以提供block编号，在驱动中通过写设备的控制寄存器，然后设备就会完成相应的工作。这是从一个文件系统的角度的描述。尽管不同的存储设备有着非常不一样的属性，从驱动的角度来看，你可以以大致相同的方式对它们进行编程。



从文件系统的角度来看磁盘还是很直观的。因为对于磁盘就是读写block或者sector，我们可以将磁盘看作是一个巨大的block的数组，数组从0开始，一直增长到磁盘的最后。



### 采用怎样的布局结构？

![img](http://xv6.dgs.zone/tranlate_books/book-riscv-rev1/images/c8/p1.png)

* block0要么没有用，要么被用作boot sector来启动操作系统。它里面通常包含了足够启动操作系统的代码。之后再从文件系统中加载操作系统的更多内容。
* block1通常被称为super block，它描述了文件系统。它可能包含磁盘上有多少个block共同构成了文件系统这样的信息。我们之后会看到XV6在里面会存更多的信息，你可以通过block1构造出大部分的文件系统信息。
* 在XV6中，log从block2开始，到block32结束。实际上log的大小可能不同，这里在super block中会定义log就是30个block。
* 接下来在block32到block45之间，XV6存储了inode。我之前说过多个inode会打包存在一个block中，一个inode是64字节。
* 之后是bitmap block，这是我们构建文件系统的默认方法，它只占据一个block。它记录了数据block是否空闲。
* 之后就全是数据block了，数据block存储了文件的内容和目录的内容。









## 问题3为什么要 Buffer Cach？

### Buffer cache 做什么？

Buffer cache有两个任务：

1. 同步对磁盘块的访问，以确保磁盘块在内存中只有一个副本，并且一次只有一个内核线程使用该副本
2. 缓存常用块，以便不需要从慢速磁盘重新读取它们。



为读写磁盘是代价较高的操作，可能要消耗数百毫秒，而block cache确保了如果我们最近从磁盘读取了一个block，那么我们将不会再从磁盘读取相同的block。



### Buffer cache 如何实现？

![image-20220715003955506](https://s2.loli.net/2022/07/15/QPhf748eLRm6tDZ.png)

### Buffer cache 为什么要用 spinlock？

```c
struct {
  struct spinlock lock;
  struct buf buf[NBUF];

  // Linked list of all buffers, through prev/next.
  // Sorted by how recently the buffer was used.
  // head.next is most recent, head.prev is least.
  struct buf head;
} bcache;

```



它有两层锁。第一层锁用来保护buffer cache的内部数据；第二层锁也就是sleep lock用来保护单个block的cache。

### block cache 为什么使用 sleep lock 而不是 spinlock？

```c
struct buf {
  int valid;   // has data been read from disk?
  int disk;    // does disk "own" buf?
  uint dev;
  uint blockno;
  struct sleeplock lock;
  uint refcnt;
  struct buf *prev; // LRU cache list
  struct buf *next;
  uchar data[BSIZE];
};


```

因为磁盘的操作需要很长的时间。

对于spinlock有很多限制，其中之一是加锁时中断必须要关闭。所以如果使用spinlock的话，当我们对block cache做操作的时候需要持有锁，那么我们就永远也不能从磁盘收到数据。或许另一个CPU核可以收到中断并读到磁盘数据，但是如果我们只有一个CPU核的话，我们就永远也读不到数据了。出于同样的原因，也不能在持有spinlock的时候进入sleep状态（注，详见13.1）。所以这里我们使用sleep lock。sleep lock的优势就是，我们可以在持有锁的时候不关闭中断。我们可以在磁盘操作的过程中持有锁，我们也可以长时间持有锁。当我们在等待sleep lock的时候，我们并没有让CPU一直空转，我们通过sleep将CPU出让出去了



### buffer cache 替换策略

brelease函数中首先释放了sleep lock；之后获取了bcache的锁；之后减少了block cache的引用计数，表明一个进程不再对block cache感兴趣；最后如果引用计数为0，那么它会修改buffer cache的linked-list，将block cache移到linked-list的头部，这样表示这个block cache是最近使用过的block cache。这一点很重要，当我们在bget函数中不能找到block cache时，我们需要在buffer cache中腾出空间来存放新的block cache，这时会使用LRU（Least Recent Used）算法找出最不常使用的block cache，并撤回它（注，而将刚刚使用过的block cache放在linked-list的头部就可以直接更新linked-list的tail来完成LRU操作）。为什么这是一个好的策略呢？因为通常系统都遵循temporal locality策略，也就是说如果一个block cache最近被使用过，那么很有可能它很快会再被使用，所以最好不要撤回这样的block cache。

以上就是对于block cache代码的介绍。这里有几件事情需要注意：

- 首先在内存中，对于一个block只能有一份缓存。这是block cache必须维护的特性。
- 其次，这里使用了与之前的spinlock略微不同的sleep lock。与spinlock不同的是，可以在I/O操作的过程中持有sleep lock。
- 第三，它采用了LRU作为cache替换策略。
- 第四，它有两层锁。第一层锁用来保护buffer cache的内部数据；第二层锁也就是sleep lock用来保护单个block的cache。



什是 LRU

https://zhuanlan.zhihu.com/p/34133067



必须要吗？ 近似的可以吗？OSTEP 第22章内存页替换

## 问题4 日志



## 问题5 inode 是什么？

术语inode（即索引结点）可以具有两种相关含义之一。

* 它可能是指包含文件大小和数据块编号列表的**磁盘上的数据结构**。
* 或者“inode”可能指**内存中的inode**，它包含磁盘上inode的副本以及内核中所需的额外信息。

### 磁盘上的 inode 是什么？

磁盘上的inode都被打包到一个称为inode块的连续磁盘区域中。每个inode的大小都相同，因此在给定数字n的情况下，很容易在磁盘上找到第n个inode。事实上，这个编号n，称为inode number或i-number，是在具体实现中标识inode的方式。

![img](http://xyfjason.top/2021/11/30/xv6-mit-6-S081-2020-Lab9-fs/inodes.png)

是一个64字节的数据结构。

* 字段`type`区分文件、目录和特殊文件（设备）。`type`为零表示磁盘inode是空闲的。
* 字段`nlink`统计引用此inode的目录条目数，以便识别何时应释放磁盘上的inode及其数据块。
* 字段`size`记录文件中内容的字节数。
* `addrs`数组记录保存文件内容的磁盘块的块号。 不同文件系统中的表达方式可能不一样，不过在XV6中接下来是一些block的编号，例如编号0，编号1，等等。XV6的inode中总共有12个block编号。这些被称为direct block number。这12个block编号指向了构成文件的前12个block。举个例子，如果文件只有2个字节，那么只会有一个block编号0，它包含的数字是磁盘上文件前2个字节的block的位置。
* `addrs`数组中的最后一个元素给出了间接块的地址。indirect block number，它对应了磁盘上一个block，这个block包含了256个block number，这256个block number包含了文件的数据。所以inode中block number 0到block number 11都是direct block number，而block number 12保存的indirect block number指向了另一个block。（因此，可以从inode中列出的块加载文件的前12 kB（`NDIRECT x BSIZE`）字节，而只有在查阅间接块后才能加载下一个256 kB）

![img](http://xv6.dgs.zone/tranlate_books/book-riscv-rev1/images/c8/p2.png)

磁盘中 inode 结构的代码在`kernel/fs.h`文件当中

```c
// On-disk inode structure
struct dinode {
  short type;           // File type
  short major;          // Major device number (T_DEVICE only)
  short minor;          // Minor device number (T_DEVICE only)
  short nlink;          // Number of links to inode in file system
  uint size;            // Size of file (bytes)
  uint addrs[NDIRECT+1];   // Data block addresses
};
```



### 内存里的 inode 是什么？

内核将活动的inode集合保存在内存中；`struct inode`（`kernel/file.h:17`）是磁盘上`struct dinode`的内存副本。只有当有C指针引用某个inode时，内核才会在内存中存储该inode。`ref`字段统计引用内存中inode的C指针的数量，如果引用计数降至零，内核将从内存中丢弃该inode。`iget`和`iput`函数分别获取和释放指向inode的指针，修改引用计数。指向inode的指针可以来自文件描述符、当前工作目录和如`exec`的瞬态内核代码。

```c
// in-memory copy of an inode
struct inode {
  uint dev;           // Device number
  uint inum;          // Inode number
  int ref;            // Reference count
  struct sleeplock lock; // protects everything below here
  int valid;          // inode has been read from disk?

  short type;         // copy of disk inode
  short major;
  short minor;
  short nlink;
  uint size;
  uint addrs[NDIRECT+1];
};

```





### xv6 如何创建 inode？

1. sys_open()
	1. 调用 create()
2. create()
	1. 解析路径名并找到最后一个目录
	2. 查看文件是否存在，如果存在的话会返回错误
	3. 调用 ialloc（inode allocate）分配 inode
3. ialloc
	1. 会遍历所有可能的inode编号，找到inode所在的block
	2. 看位于block中的inode数据的type字段。如果这是一个空闲的inode，那么将其type字段设置为文件，这会将inode标记为已被分配



## 问题6

