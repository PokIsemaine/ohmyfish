# new和delete

## 关于 new,delete

![image-20220224155705472](https://s2.loli.net/2022/02/24/SmYPOXJx9KZoF25.png)

`new`和`delete`是运算符，不过平常我们以表达式的形式去使用，但是`operator new`和`operator delete`是库函数

表达式的行为是不能变的，但是分解出来的几个函数是可以重载的

`operator new`和`operator delete`底层实际上调用`malloc`和`free`

## 重载 operator new 和 operator delete

### 重载全局的 operator new/delete/new[]/delete[]

![image-20220224162436389](https://s2.loli.net/2022/02/24/O1P4I9TwceBqYRa.png)

注意重载全局的影响是无远无界的，很多东西底层都会用到这四个的重载



### 重载 member operator new/delete/new[]/delete[]

![image-20220224163012976](https://s2.loli.net/2022/02/24/61xVTItO4qWa5Ey.png)



![image-20220224162750993](https://s2.loli.net/2022/02/24/qZBTXE5y6blN4kx.png)



### 示例

![image-20220224164300403](https://s2.loli.net/2022/02/24/28M4qcdfUGYTprQ.png)



```cpp
//如果有members就用members,否则调用全局的
Foo* pf = new Foo;
delete pf;

//强制用全局的new和delete
Foo* pf = ::new Foo;
::delete pf;
```



![image-20220224165135960](https://s2.loli.net/2022/02/24/1F3YTZKPeRdyqSU.png)

思考一下12,16,64,84这几个大小是怎么来的？



![image-20220224165630688](https://s2.loli.net/2022/02/24/e3LumMQqUh8Rrfw.png)

## 重载 new()，delete()

![image-20220224165855696](https://s2.loli.net/2022/02/24/d57PH1vQsLyReo4.png)

我们还可以重载`placment new`和`placement delete`
```cpp
Foo* pf = new Foo;//一般的operator new()重载，实际上隐藏这第一个参数size_t

//placment new,带着括号,除了第一个参数size_t,还有以placement arguments作为初值的参数
//注意300不是第一个参数size_t啊，所以实际上有三个参数
Foo* pf = new(300,'c')Foo;
```

我们也可以重载`class member operator delete()`但要注意他们不会被`delete`调用，只有`new`的`ctor`抛出`exception`的时候才会调用重载版本的`operator delete()`其主要作用使用来处理未能完全创建成功的`object`所占有的`memory`

### 示例

![image-20220224170450131](https://s2.loli.net/2022/02/24/iqXkEreAY9wpDf5.png)



![image-20220224170522588](https://s2.loli.net/2022/02/24/tWENoPvpaRfUMJG.png)

上面图片第五条，我期待是`ctor`失败后先调用对应的`placement delete`，然后抛出异常退出。

但实际上不同版本好像不一样，这个就属于比较具体的、细节的东西了，要具体看情况而定

但要注意的是`operator delete(...)`未能一对一`operator new(...)`也不会出现任何报错。表示告诉编译器放弃处理`ctor`抛出的异常

### placment new 的标准库例子

![image-20220224193938078](https://s2.loli.net/2022/02/24/XNDJF1mtjacQzl7.png)
