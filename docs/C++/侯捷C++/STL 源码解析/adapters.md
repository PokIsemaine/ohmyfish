# 配接器

## 配接器之观感与分类

`function adapter`改变仿函数接口

`container adapter`改变容器接口

`iterator adapter`改变迭代器接口

## container adapters

### stack

`stack`的底层由`deque`构成。`class stack`封住了所有的`deque`对外接口，只开放符合`stack`原则的几个函数，所以我们说`stack`是一种配接器，一个作用于容器之上的配接器

### queue

`queue`的底层由`deque`构成。`class queue`封住了所有的`deque`对外接口，只开放符合`queue`原则的几个函数，所以我们说`queue`是一种配接器，一个作用于容器之上的配接器

## iterator adapters

### insert iterators

所谓`insert iterators`，可以将一般迭代器的赋值操作转变为插入操作。

包括下面三种

尾端插入`back_insert_iterator(Container& x)`

首端插入`front_insert_iterator(Container& x)`

任意位置插入`insert_iterator(Container& x)`

`insert iteratos`的前进、后退、取值、成员取用等操作都没意义的，甚至被禁止使用

### reverse iterators

所谓`reverse iterator`，就是将迭代器的移动行为倒转。如果`STL`算法接受的不是一般正常的迭代器，而是这种逆向迭代器，它就会从尾到头的方向来处理容器中的元素

为了配合迭代器前闭后开，正向迭代器与其对应的逆向迭代器取出不同的元素

### stream iterators

所谓`stream iterators`，可以将迭代器绑定到一个`stream`数据流对象身上。



## function adapters

如何能够实现对一个函数完成参数的绑定、执行结果的否定，甚至多方函数的组合？

但是我们知道，只有真正调用某个函数的时候，才有可能对参数和执行结果进行感受，那么如何事先处理呢？

就像`container adapters`内藏了一个`container member`，`reverse iterator(adpaters)`内藏了一个`iterator member`，`stream iterator(adapters)`内藏了一个`pointer to stream`，`insert iterator(adapters)`内藏了一个`pointer to container`一样

每个`function adapters`也内藏了一个`member object`其型别等同于它所要配接的对象

![image-20220322083257966](https://s2.loli.net/2022/03/22/LDhbtsXWHTFrJx7.png)