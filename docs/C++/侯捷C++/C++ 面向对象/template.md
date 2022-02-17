# 关于模板

## 类模板

![image-20220216212058789](https://s2.loli.net/2022/02/16/6GDosFplxM81L9K.png)

## 函数模板

![image-20220216212117401](https://s2.loli.net/2022/02/16/mZoPQV9OzF6TIxk.png)

编译器会对函数模板进行实参推导，所以可以不用指明类型

## 成员模板

![image-20220216213018567](https://s2.loli.net/2022/02/16/hgjZn1SE6vIqoaR.png)

黄色那块就是成员模板啦，既可以在模板类里，也可以在普通类里

使用成员函数模板可以让代码更具弹性

### 成员模板与构造函数

在下面的代码中，我们令构造函数为成员模板。我们可以让鱼类和鸟类组成一个`pair`也可以让鲫鱼和麻雀组成一个`pair`

![image-20220217222728353](https://s2.loli.net/2022/02/17/LwqpgKWSb6ZjePD.png)

注意到

```cpp
template<class U1,class U2>
pair(const pair<U1,U2>&p):
	first(p.first),second(p.second){}
```

这段代码实际上要求了 鲫鱼能转为鱼类，麻雀能转为鸟类（实际上是继承）

另外一个例子就是，我们智能指针会有个传入普通指针的构造函数，因为普通指针能做的智能指针也要能做。

![image-20220217222744588](https://s2.loli.net/2022/02/17/Bx4alqQfJLkI9tj.png)

## 模板特化

![image-20220217224029110](https://s2.loli.net/2022/02/17/jpg4J3zoUD9ibMS.png)

第一个代码块为泛化（全泛化），第二个代码块为特化

编译器会优先使用特化的版本，不要一股脑认为用的是泛化版本的实现

特化可以专门针对一些类型做出不同的处理

### 偏特化

对应全泛化的就是偏特化（局部特化）

一般有两种情况偏特化：个数的偏、范围的偏

要从左到右逐个特化，不能跳着

![image-20220217224400724](https://s2.loli.net/2022/02/17/gGFphL4EOsrHKmv.png)



![image-20220217224816451](https://s2.loli.net/2022/02/17/nh5Eb8sRNW6avdz.png)

## 模板模板参数

![image-20220217225108227](https://s2.loli.net/2022/02/17/ZcpNqoesSKnMyxk.png)

当我们希望类具有这样的弹性：可以为任意的容器，并且可以指定任意一种类型放入容器

我们可以考虑使用模板模板参数

但要注意的是，容器可能有第二、第三模板参数，平常我们`list<string>`的时候其他模板参数使用了默认值

```cpp
template<typename T>
using Lst = list<T,allocator<T>>;
XCls<string,list> mylst1;//不可行，没考虑第二模板参数
XCls<string,Lst> mylst1;
```

![image-20220217225947327](https://s2.loli.net/2022/02/17/EIAjrMu5gaKSLep.png)

下面为 `cppreference` 上查到的四种只能指针的相关参数

`shared_ptr`

```cpp
template< class T > class shared_ptr; //(since C++11)
```

`unique_ptr`

```cpp
template<
    class T,
    class Deleter = std::default_delete<T>
> class unique_ptr;//(1) 	(since C++11)
	
template <
    class T,
    class Deleter
> class unique_ptr<T[], Deleter>;// 	(since C++11)
```

`weak_ptr`

```cpp
template< class T > class weak_ptr; //(since C++11)
```

`auto_ptr`

```cpp
template< class T > class auto_ptr; //(1) 	(deprecated in C++11)(removed in C++17)
template<> class auto_ptr<void>;	//(2) 	(deprecated in C++11)(removed in C++17)
```



![image-20220217230101066](https://s2.loli.net/2022/02/17/3EqyX2NSGh4I8oj.png)

第二种用法中`list<int>`实际上已经被绑定好不是模板了