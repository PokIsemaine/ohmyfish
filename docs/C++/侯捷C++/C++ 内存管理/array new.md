# array new,array delete

## 关于内存泄漏

![image-20220302140651017](https://s2.loli.net/2022/03/02/Uak9QZ7jBYxGeXg.png)

* `new`底层其实是调用`malloc`实现的，左图里的三个`Complex object`实际上也是`malloc`分配出来的。但是当我们使用`delete`的时候，底层的`free`只是接收到指针并不知道到底有多少个，所以`malloc`在申请的时候会附带一个`cookie`用来存长度等信息
* 对于没有`ptr member`的类来说，使用`array new`申请，然后用`delete`释放可能没什么问题，并不会发生内存泄漏。（内存泄漏发生在析构，具体见下面那点）
* 而对于有`ptr member`的类来说（右图），`delete`只调用了一次`dtor`，这样的话绿色部分是清除了，但是`ptr member`指向的部分会发生内存泄漏，可能只清除了`str1`或`str3`。（内存泄漏发生在蓝色部分，蓝色部分要靠调用三次析构函数才能完全清除）
* 所以干脆统一`array new`对应`array delete`

![image-20220302141856201](https://s2.loli.net/2022/03/02/VCcmFWOUI59LzQB.png)

## 代码验证

![image-20220302142212070](https://s2.loli.net/2022/03/02/hik6TaId3Pnm9gQ.png)

* 一定要写默认构造函数，因为`array new`的时候我们没办法给初值，只能用默认构造函数
* 因为没法直接调用构造函数，所以我们采用`placment new`的方式来调用`A(i)`相当于对指定位置`new`

## VC6下内存布局

### non-trival dtor

![image-20220302144813581](https://s2.loli.net/2022/03/02/mM3x7vkaJWXfIts.png)

### 无 non-trival dtor

![image-20220302144847002](https://s2.loli.net/2022/03/02/CGwZhLjSRm9biDo.png)
