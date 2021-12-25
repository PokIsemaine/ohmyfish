<h1 align="center">📔 C++ Prime 0x01 练习题解</h1>

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

```cpp
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

```cpp
int main(){
	return -1;
}
```

执行同样命令，发现还是没反应

## 1.2节 练习

> 练习1.3: 标准输出上打印Hello,World 

```cpp
#include<iostream>

int main(){
	std::cout << "Hello,World";
	return 0;
}
```



> 练习1.4+1.5：写个两个数的加法和乘法，分开输出

```cpp
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

```cpp
std::cout << "The sum of " << v1;
		<< "and " << v2;
		<< " is " << v1 + v2 << std::endl;
```

不合法，编译报错如下

![image-20211214203753306](https://s2.loli.net/2021/12/14/hFcoDUGx2AMq6rI.png)

```cpp
std::cout << "The sum of " << v1
		<< "and " << v2
		<< " is " << v1 + v2 << std::endl；
```

去除分号即可

```cpp
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

```cpp
std::cout << "/*"; 合法
std::cout << "*/"; 合法
std::cout << /* "*/" */; 不合法
std::cout << /* "*/" /* "/*" */; 合法 相当于输出" /*"
```

## 1.4节 练习

### 1.4.1节 练习

> 1.9：编写程序，使用while循环将50到100的整数相加

```cpp
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

```cpp
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

```cpp
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

```cpp
int sum = 0;
for (int i = -100; i<= 100;++i)
	sum += i;
```

从-100加到100,sum最终为0



> 1.13：使用for循环重做1.4.1节中所有练习

练习1.9

```cpp
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

```cpp
#include <iostream>

int main(){
    for(int val=10;val>=0;--val){
        std::cout << val << std::endl;
    }
    return 0;
}
```

练习1.11

```cpp
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

> 1.14 对比for循环和while循环，两种形式的优缺点各是什么

for和while基本可以相互转换

如果知道循环次数，那更适合 for 循环

如果循环次数不容易知道，那更适合 while 循环

```cpp
for(;cond_true;)
while(cond_true)
```

```cpp
//可能会报警告conditional expression is constant
while(1){

}

for(;;){

}
```

[stackoverflow 上关于这个的讨论](https://stackoverflow.com/questions/2950931/for-vs-while-in-c-programming)

> 1.15 编写程序，包含第14页再探编译中讨论的常见错误。熟悉编译器生成的错误信息

编译器可以检查语法错误，没法检查语义错误。

常见的错误有下面这些

* 语法错误 syntax error

* 类型错误 type error

* 声明错误 declaration error

	

### 1.4.3节 练习

> 1.16 编写程序，从cin读取一组数，输出其和
>
> Ctrl + D 结束输入

```cpp
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



### 1.4.4节 练习

> 1.17 如果输入的所有值都是相等的，本节程序会输出什么？如果没有重复值，输出又会是怎么样



> 1.18 编译运行本节程序，给它输入全部相等的值。再次运行，输入没有重复的值



> 1.19 修改1.4.1节练习1.10,使其能处理用户输入的第一个数比第二个数小的情况？

应该是练习1.11吧，下面是处理第一个数比第二个数大的情况

```cpp
#include <iostream>

int main(){
    int start,end;
    std::cout << "请输入两个整数:start end" << std::endl;
    std::cin >> start >> end;
    if(start>end){
        int tmp=start;
        start=end;
        end=tmp;
    }
    while(start<=end){
        std::cout << start << std::endl;
        ++start;
    }
    return 0;
}
```

## 1.5节 练习

### 1.5.1节 练习

`Sales_item.h`

```cpp
/*
 * This file contains code from "C++ Primer, Fifth Edition", by Stanley B.
 * Lippman, Josee Lajoie, and Barbara E. Moo, and is covered under the
 * copyright and warranty notices given in that book:
 * 
 * "Copyright (c) 2013 by Objectwrite, Inc., Josee Lajoie, and Barbara E. Moo."
 * 
 * 
 * "The authors and publisher have taken care in the preparation of this book,
 * but make no expressed or implied warranty of any kind and assume no
 * responsibility for errors or omissions. No liability is assumed for
 * incidental or consequential damages in connection with or arising out of the
 * use of the information or programs contained herein."
 * 
 * Permission is granted for this code to be used for educational purposes in
 * association with the book, given proper citation if and when posted or
 * reproduced.Any commercial use of this code requires the explicit written
 * permission of the publisher, Addison-Wesley Professional, a division of
 * Pearson Education, Inc. Send your request for permission, stating clearly
 * what code you would like to use, and in what specific way, to the following
 * address: 
 * 
 *     Pearson Education, Inc.
 *     Rights and Permissions Department
 *     One Lake Street
 *     Upper Saddle River, NJ  07458
 *     Fax: (201) 236-3290
*/ 

/* This file defines the Sales_item class used in chapter 1.
 * The code used in this file will be explained in
 * Chapter 7 (Classes) and Chapter 14 (Overloaded Operators)
 * Readers shouldn't try to understand the code in this file
 * until they have read those chapters.
*/

#ifndef SALESITEM_H
// we're here only if SALESITEM_H has not yet been defined 
#define SALESITEM_H

// Definition of Sales_item class and related functions goes here
#include <iostream>
#include <string>

class Sales_item {
// these declarations are explained section 7.2.1, p. 270 
// and in chapter 14, pages 557, 558, 561
friend std::istream& operator>>(std::istream&, Sales_item&);
friend std::ostream& operator<<(std::ostream&, const Sales_item&);
friend bool operator<(const Sales_item&, const Sales_item&);
friend bool 
operator==(const Sales_item&, const Sales_item&);
public:
    // constructors are explained in section 7.1.4, pages 262 - 265
    // default constructor needed to initialize members of built-in type
    Sales_item() = default;
    Sales_item(const std::string &book): bookNo(book) { }
    Sales_item(std::istream &is) { is >> *this; }
public:
    // operations on Sales_item objects
    // member binary operator: left-hand operand bound to implicit this pointer
    Sales_item& operator+=(const Sales_item&);
    
    // operations on Sales_item objects
    std::string isbn() const { return bookNo; }
    double avg_price() const;
// private members as before
private:
    std::string bookNo;      // implicitly initialized to the empty string
    unsigned units_sold = 0; // explicitly initialized
    double revenue = 0.0;
};

// used in chapter 10
inline
bool compareIsbn(const Sales_item &lhs, const Sales_item &rhs) 
{ return lhs.isbn() == rhs.isbn(); }

// nonmember binary operator: must declare a parameter for each operand
Sales_item operator+(const Sales_item&, const Sales_item&);

inline bool 
operator==(const Sales_item &lhs, const Sales_item &rhs)
{
    // must be made a friend of Sales_item
    return lhs.units_sold == rhs.units_sold &&
           lhs.revenue == rhs.revenue &&
           lhs.isbn() == rhs.isbn();
}

inline bool 
operator!=(const Sales_item &lhs, const Sales_item &rhs)
{
    return !(lhs == rhs); // != defined in terms of operator==
}

// assumes that both objects refer to the same ISBN
Sales_item& Sales_item::operator+=(const Sales_item& rhs) 
{
    units_sold += rhs.units_sold; 
    revenue += rhs.revenue; 
    return *this;
}

// assumes that both objects refer to the same ISBN
Sales_item 
operator+(const Sales_item& lhs, const Sales_item& rhs) 
{
    Sales_item ret(lhs);  // copy (|lhs|) into a local object that we'll return
    ret += rhs;           // add in the contents of (|rhs|) 
    return ret;           // return (|ret|) by value
}

std::istream& 
operator>>(std::istream& in, Sales_item& s)
{
    double price;
    in >> s.bookNo >> s.units_sold >> price;
    // check that the inputs succeeded
    if (in)
        s.revenue = s.units_sold * price;
    else 
        s = Sales_item();  // input failed: reset object to default state
    return in;
}

std::ostream& 
operator<<(std::ostream& out, const Sales_item& s)
{
    out << s.isbn() << " " << s.units_sold << " "
        << s.revenue << " " << s.avg_price();
    return out;
}

double Sales_item::avg_price() const
{
    if (units_sold) 
        return revenue/units_sold; 
    else 
        return 0;
}
#endif

```

> 1.20 在网站http://www.informit.com/title/032174113 上，第1章的代码目录包含了头文件 Sales_item.h。将它拷贝到你自己的工作目录中。用它编写一个程序，读取一组书籍销售记录，将每条记录打印到标准输出上。

`add_item`

```cpp
0-201-78345-X 3 20.00
0-201-78345-X 2 25.00
```

`1.20.cpp`

```cpp
#include <iostream>
#include "Sales_item.h"

int main(){
    Sales_item item;
    while(std::cin >> item){
        std::cout << item <<std::endl;
    }
    return 0;
}
```

可以直接在终端输入数据，也可以用文件重定向输入

```sh
./1.20 < data/add_item 
```

输出

```
0-201-78345-X 3 60 20
0-201-78345-X 2 50 25
```



> 1.21 编写程序，读取两个 ISBN 相同的 Sales_item 对象，输出它们的和

还是用`add_item`的数据

```cpp
#include <iostream>
#include "Sales_item.h"

int main(){
    Sales_item item_a,item_b;
    std::cin >> item_a >> item_b;
    std::cout << "sum:" << item_a + item_b;
    return 0;
}
```

输出

```
sum:0-201-78345-X 5 110 22
```



> 1.22 编写程序，读取多个具有相同 ISBN的 Sales_item 对象，输出所有记录的和

```cpp
#include <iostream>
#include "Sales_item.h"

int main(){
    Sales_item item,item_sum;
    while(std::cin >> item){
        item_sum += item;
    }
    std::cout << "sum:" << item_sum;
    return 0;
}
```



### 1.5.2节 练习

数据在`book_sales`里

```
0-201-70353-X 4 24.99
0-201-82470-1 4 45.39
0-201-88954-4 2 15.00 
0-201-88954-4 5 12.00 
0-201-88954-4 7 12.00 
0-201-88954-4 2 12.00 
0-399-82477-1 2 45.39
0-399-82477-1 3 45.39
0-201-78345-X 3 20.00
0-201-78345-X 2 25.00
```



> 1.23 编写程序，读取多条销售记录，并统计每个 ISBN （每本书）有几条销售记录
>
> 1.24 输入表示多个 ISBN 的多条销售记录来测试上一个程序，每一个 ISBN 的记录应该聚集在一起

如果真的按照题目意思的话，其实应该看这本书有没有出现过，然后在加在对应的书上，写起来是比较麻烦的。



但是看了几个前辈的仓库题解，还有书本的上下文。我觉得应该是要我们基于 1.5.2初识成员函数 中的代码片段修改，**并且数据特点是相同 ISBN在一块**，所以我们编写下面这样的程序。

这个程序实际上可以参考一下 1.6节的书店程序

```cpp
#include <iostream>
#include "Sales_item.h"

int main(){
    Sales_item total,item;
    if (std::cin >> total){
        while (std::cin >> item){
            if (total.isbn() == item.isbn()) {
                total += item;
            }else {
                std::cout << total << std::endl;
                total = item;
            }
        }
        std::cout << total << std::endl;
    }else {
        std::cerr << "No data?!" << std::endl;
        return -1;
    }
    return 0;
}
```

## 1.6节 练习

> 1.25 借助网站上的 Sales_item.h 头文件，编译并运行本书给出的书店程序

就是上面那个程序
