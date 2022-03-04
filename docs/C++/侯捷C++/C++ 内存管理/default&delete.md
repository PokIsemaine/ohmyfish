# =default,=delete

* `=default`采用函数的默认形式
* `=delete`删除某个函数

## =default

![image-20220304094040803](https://s2.loli.net/2022/03/04/QyBoDuqlRvn4532.png)

C++中只有拷贝构造函数，拷贝赋值函数，析构函数有默认版本（编译器合成版本）



## =delete

除了拷贝构造函数，拷贝赋值函数，析构函数可以使用`=delete`外，`operator new/new[]`，`operator delete/delete[]`以及它们的重载也可以使用`=delete`

![image-20220304094132570](https://s2.loli.net/2022/03/04/DtGKOFd1saZETvb.png)