# new expression

## new expression

![image-20220301091416312](https://s2.loli.net/2022/03/01/jHcv9t4Pauxgzod.png)

* 观察`operator new`的源码，可以看到其底层就是循环调用了`malloc`。在vc98如果失败了会调用`new handler`即`newh`，`newh`是自己弄的一个函数，用于释放掉一些可以释放内存。另外注意到第二个参数`std::nothrow`表明`operator new`保证不抛异常，在`C++11`中可以用`noexcept`表明不抛异常
* 由于`operator new`返回的是`void*`，我们要用的就要通过`static_cast`进行类型转换，得到具体类型指针
* 最后调用对象构造函数（只有编译器能够直接像图里一样调用，想要直接调用可以用`placement new`）
* 编译优化有可能会改变具体顺序

## delete expression

![image-20220301092831090](https://s2.loli.net/2022/03/01/R8vW63zbw95OdGC.png)

* 不同于构造函数，可以直接调用析构函数
* 可以发现`operator delete`底层还是调用`free()`

## Ctor & Dtor 直接调用

![image-20220301093324349](https://s2.loli.net/2022/03/01/8FEsmOpbv7oaqly.png)

* 通过指针不能直接调用构造函数（VC6不太严谨）
* 通过指针直接调用析构函数可以通过，但是如果`99`行那个直接调用构造函数不能用（注释掉了），那调用析构函数也会crash