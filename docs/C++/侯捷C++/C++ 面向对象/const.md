# 谈谈 const

![image-20220223235823510](https://s2.loli.net/2022/02/23/eO9WJa8bjRk45UP.png)

我们设计类的时候要考虑用户可能用`const object`和`non-const object`两种情况，以及成员函数是否应该加`const`

![image-20220223235833657](https://s2.loli.net/2022/02/23/EQZrUo5JjLfneHC.png)

看到左边那个`basic_string`的两个成员函数

```cpp
charT operator[](size_type pos)const
{.../*不必考虑COW*/}

reference operator[](size_type pos)
{.../*必须考虑COW*/}
```

首先考虑这两个成员函数是不是重载？换句话说`const`是否属于前面的一部分

函数签名不用看`return type`，我们关注后面部分。`const`实际上是对隐藏的`this`参数所指向的对象加`const`，`const`是函数前面的一部分，所以这两个成员函数是函数重载

那么，为什么要写两个不同`const`版本的`operator[]`呢？原因是 `C++`标准库里的`string`采用了引用计数的共享技术（设计模式），一个字符串和它拷贝出来的字符串是共享的

既然字符串是共享的，那么就要考虑数据变化的情况。如果直接改内容，那么几个共享的字符串会一起改。我们要改A就要让A不再和B、C共享，然后再去修改A，即`Copy On Write`。

如果是常量字符串`operator[]`不会发生修改，就不必考虑`Copy On Write`，这样提高了效率。

如果是非常量字符串`operator[]`有可能读也有可能写，就必须要考虑`Copy On Write`。

为了区分常量字符串和非常量字符串我们使用`const`来重载