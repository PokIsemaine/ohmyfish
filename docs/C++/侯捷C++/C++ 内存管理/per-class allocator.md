# per-class allocator

## 目标

目标：针对一个`class`写出它的内存管理

* 虽然`malloc`的速度很快，但是我们还是希望能够尽可能减少调用`malloc`的次数

所以如果`class`的使用者经常`new`那么我们可不可以先挖一大块来做准备，然后切成小块。这个我们可以通过重载`class::operator new`来接管

![image-20220303085050817](https://s2.loli.net/2022/03/03/iuXLohtW4zNCJF1.png)



* 除了降低`malloc`次数，我们还想减少`cookie`的量，减少浪费。如果1000块内存，我们希望1000块内存都不带`cookie`，只有这个整体上下带`cookie`

## per-class allocator 1

![image-20220303090652645](https://s2.loli.net/2022/03/03/nJQf4dmwiHoDBLZ.png)

基于先前的思想，我们利用链表来进行内存管理

重载`operator new`：如果链表是空的，那么就先申请一大块内存，然后进行分割，用链表串起来。否则就从链表里拿

重载`operator delete`：`delete`先调用析构函数，然后调用重载的`operator delete`来释放内存。`operator delete`收到一个`void*`指针，首先进行类型转换。众所周知单向链表有个头指针，我们把传入指针指向的资源会受到头部即可。注意这里是将回收的内存指针放在链表头部，因为这样操作比较快。只需要动头指针就行了。**注意这里的内存并没有还给操作系统**，而是将使用过的内存串成一个链表。

`operator new`

```cpp
void* Screen::operator new(size_t size)
{
  Screen *p;
  if (!freeStore) {
      //linked list 是空的，所以攫取一大塊 memory
      //以下呼叫的是 global operator new
      size_t chunk = screenChunk * size;
      freeStore = p =
         reinterpret_cast<Screen*>(new char[chunk]);
      //將分配得來的一大塊 memory 當做 linked list 般小塊小塊串接起來
      for (; p != &freeStore[screenChunk-1]; ++p)
          p->next = p+1;
      p->next = 0;
  }
  p = freeStore;
  freeStore = freeStore->next;
  return p;
}

```



```cpp
if (!freeStore) {
    //linked list 是空的，所以攫取一大塊 memory
    //以下呼叫的是 global operator new
    size_t chunk = screenChunk * size;
    freeStore = p =
    reinterpret_cast<Screen*>(new char[chunk]);
    //將分配得來的一大塊 memory 當做 linked list 般小塊小塊串接起來
    for (; p != &freeStore[screenChunk-1]; ++p)
    p->next = p+1;
    p->next = 0;
}
```

![image-20220303093909962](https://s2.loli.net/2022/03/04/hUTyYqIFcKoeXtB.png)

```cpp
p = freeStore;
freeStore = freeStore->next;
return p;
```

![image-20220303094104353](https://s2.loli.net/2022/03/03/LyjbWuCS12ZNBnA.png)

`operator delete`

```cpp
void Screen::operator delete(void *p, size_t)
{
  //將 deleted object 收回插入 free list 前端
  (static_cast<Screen*>(p))->next = freeStore;
  freeStore = static_cast<Screen*>(p);
}
```

![image-20220303094313380](https://s2.loli.net/2022/03/03/d6529uPWtaliAce.png)



概览

![image-20220303094520377](https://s2.loli.net/2022/03/03/jJEyW6Fw7ZVeUOx.png)



成本：多耗用一个`next`指针

效果：消除了`cookie`，减少了`malloc`的使用次数

### 验证消除了cookie

![image-20220303091042032](https://s2.loli.net/2022/03/03/nBhruxWYPKCgq7I.png)

对比`member operator/delete`和`global operator new/delete`的结果，比较其间隔。可以发现`member operator new/delete`确实消除了`cookie`

### 问题

但是为了减少`malloc`使用次数，减少`cookie`，我们消耗了一个`next`指针。这样岂不是拆东墙补西墙？考虑如何优化这个`next`指针带来的成本问题。



## per-class allocator 2

### 嵌入式指针

![image-20220303094703021](https://s2.loli.net/2022/03/03/RAJBHGf6YVMhZFu.png)

`per-class allocator 2`的`operator new`和`per-class allocator 1`基本一样。这里主要采用了匿名`union`实现的嵌入式指针的技巧（embeded pointer）。(如果不是匿名那就是type定义式了)

> 嵌入式指针工作原理：借用A对象所占用的内存空间中的前4个字节，这4个字节用来 链住这些空闲的内存块；
>  但是，一旦某一块被分配出去，那么这个块的 前4个字节 就不再需要，此时这4个字节可以被正常使用；
>  从工作原理中可以看出嵌入式指针使用前提：类A对象的[sizeof](https://so.csdn.net/so/search?q=sizeof&spm=1001.2101.3001.7020)必须不小于4字节。

关于嵌入式指针，可以看一下这篇[博客](https://blog.csdn.net/qq_42604176/article/details/113871565)

![image-20220303101130081](https://s2.loli.net/2022/03/03/GhZxWTekzBjb6fV.png)

### 问题

现在还有一个问题就是没有将内存真正还给操作系统。这样好吗？如果申请了很大的一片内存（峰值），一直不还想必是不太好的。但是这也算不上内存泄漏，因为都掌握在内存池里

另外一个问题是，每次都要给不同类写几乎相同的`member operator new/delete`好麻烦啊



## static allocator 3

### 目的

为了解决每次都要给不同类写几乎相同的`member operator new/delete`的麻烦问题，我们提出一个`allocator`类（分配器)，这样不同类都可以用同一个分配器了。

![image-20220303102304253](https://s2.loli.net/2022/03/03/b3dLxsNSHZDqizM.png)

可以看到`allocator`里的`allocate`和`deallocate`和我们先前写的`operator new/delete`实现基本一样（毕竟就是从其中提出来的嘛）

然后注意到`const int CHUNK=5`这里的话设置`5`主要是为了方便观察，具体设多少要看具体情况和经验，标准库里设置的是20

同样的，我们使用`embeded pointer`的技巧来减少成本，借用一个对象的前4个字节作为指针用。

### 简化后的效果

![image-20220303103149532](https://s2.loli.net/2022/03/03/FRijmSIOu1YdBgw.png)



![image-20220303103519681](https://s2.loli.net/2022/03/03/y9V6rSnc24lofzd.png)



## macro for static allocator 4

> 懒是人类进步的源动力！！！

我们可以使用宏进一步简化，不同类的黄色部分基本是一致的，我们可以用宏来简化。

![image-20220303104143343](https://s2.loli.net/2022/03/03/zERit5xTrGkOLPY.png)

结果和版本3是基本相同的

![image-20220303104332911](https://s2.loli.net/2022/03/03/HQXAlVtjvyMTwd4.png)



## 标准库里的 allocator

标准库的`allocator`具有16条`free-lists`，这意味着它可以应付16种不一样的大小，因此它不再是针对某个类而是全局的

![image-20220303104742987](https://s2.loli.net/2022/03/03/u1s9TtQd73mwygZ.png)