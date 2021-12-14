<h1 align="center">📔 C++ Prime 0x01 学习笔记</h1>

## 1.1节 练习

> 练习1.1：查阅编译器的文档，确定它所使用的命名约定，编译并运行第2页的main程序

这个练习有 3 个任务点：

* 什么是编译器？
* 查阅使用的编译器文档，确定它所使用的命名约定
* 编译并运行第2页的main程序



**什么是编译器**

通俗的说就是翻译官，翻译代码的。人写代码，编译器翻译给机器。



**查阅使用的编译器文档，确定它所使用的命名约定**

我使用的是GNU (GNU Compiler Collection) 编译器

通过指令`g++ --version`可以查看g++ 的版本

![image-20211214201106558](https://s2.loli.net/2021/12/14/GzIYmgsC1xoevXa.png)

通过查阅man手册我们可以获取到文件命名约定

输入指令`man g++`可以查找到以下内容

![image-20211214201401779](https://s2.loli.net/2021/12/14/TcZCkJDQdfzOnjY.png)



**编译并运行第2页的main程序**

这个文件命名为 `tmp.cpp`

```c++
int main(){
	return 0;
}
```

编译  `tmp.cpp` 文件产生 `a.out` 文件

```shell
g++ tmp.cpp
```

运行 `a.out` 文件

```shell
./a.out
```

啥都没发生



> 练习1.2: 改写程序，让它返回-1。返回值-1通常被当作程序错误的标识。重新编译并运行你的程序，观察你的系统如何处理main返回的错误标识

代码修改为

```c++
int main(){
	return -1;
}
```

执行同样命令，发现还是没反应

## 1.2节 练习

> 练习1.3: 标准输出上打印Hello,World 

```c++
#include<iostream>

int main(){
	std::cout << "Hello,World";
	return 0;
}
```



> 练习1.4+1.5：写个两个数的加法和乘法，分开输出

```c++
#include <iostream>
 
int main(){
    int v1 = 0, v2 = 0;
    std::cin >> v1 >> v2;
    std::cout << v1 + v2 << std::endl;
    std::cout << v1 * v2 << std::endl;                               
    return 0;
}

```



> 练习1.6 解释下面程序段是否合法

```c++
std::cout << "The sum of " << v1;
		<< "and " << v2;
		<< " is " << v1 + v2 << std::endl;
```

不合法，编译报错如下

![image-20211214203753306](https://s2.loli.net/2021/12/14/hFcoDUGx2AMq6rI.png)

```c++
std::cout << "The sum of " << v1
		<< "and " << v2
		<< " is " << v1 + v2 << std::endl；
```

去除分号即可

```c++
#include <iostream>
 
int main(){
    int v1 = 0, v2 = 0;
    std::cin >> v1 >> v2;
    std::cout << "The sum of " << v1
    << "and " << v2
    << " is " << v1 + v2 << std::endl;
    return 0;
}                                             
```

