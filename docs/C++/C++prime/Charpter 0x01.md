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

## 1.3节 练习

> 1.7 ：编写一个包含不正确的嵌套注释的程序，观察编译器返回的错误信息 

看1.8的例子即可

> 1.8：指出下列哪些输出语句是合法的

```c++
std::cout << "/*"; 合法
std::cout << "*/"; 合法
std::cout << /* "*/" */; 不合法
std::cout << /* "*/" /* "/*" */; 不合法
```

## 1.4节 练习

### 1.4.1节 练习

> 1.9：编写程序，使用while循环将50到100的整数相加

```c++
#include <iostream>

int main(){
    int sum=0,val=50;
    while(val<=100){
        sum+=val;
        ++val;
    }
    std::cout << sum;
    return 0;
}
```



> 1.10：除了++运算符将对象的值+1之外，还有递减运算符--实现将其值减少1。编写程序打印10到0之间的整数

```c++
#include <iostream>

int main(){
    int val=10;
    while(val>=0){
        std::cout << val << std::endl;
        --val;
    }
    return 0;
}
```



> 1.11：编写程序，提示用户输入两个整数，打印出这两个整数所制定的范围内的所有整数

```c++
#include <iostream>

int main(){
    int start,end;
    std::cout << "请输入两个整数:start end" << std::endl;
    std::cin >> start >> end;
    while(start<=end){
        std::cout << start << std::endl;
        ++start;
    }
    return 0;
}
```

### 1.4.2节 练习

> 1.12：下面的for循环完成了什么功能？sum的终值是多少？

```c++
int sum = 0;
for (int i = -100; i<= 100;++i)
	sum += i;
```

从-100加到100,sum最终为0



> 1.13：使用for循环重做1.4.1节中所有练习

练习1.9

```c++
#include <iostream>

int main(){
    int sum=0;
    for(int val = 50;val <= 100;++val)
        sum+=val;
    std::cout << sum;
    return 0;
}
```

练习1.10

```c++
#include <iostream>

int main(){
    for(int val=10;val>=0;--val){
        std::cout << val << std::endl;
    }
    return 0;
}
```

练习1.11

```c++
#include <iostream>

int main(){
    int start,end;
    std::cout << "请输入两个整数:start end" << std::endl;
    std::cin >> start >> end;
    for(int i = start ;i<=end ;++i){
        std::cout << i << std::endl;
    }
    return 0;
}
```

### 1.4.3节 练习

> 1.16 编写程序，从cin读取一组数，输出其和
>
> Ctrl + D 结束输入

```c++
#include <iostream>

int main(){
    int sum=0,val=0;
    while(std::cin>>val){
        sum+=val;
    }
    std::cout<<sum;
    return 0;
}
```

