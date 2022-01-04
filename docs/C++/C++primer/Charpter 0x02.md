<h1 align="center">📔 C++ Primer 0x02 练习题解</h1>

## 2.1节 练习

### 2.1.1节 练习

> 2.1 类型 int、long、long long 和 short 的区别是什么？无符号类型和带符号类型的区别是什么？ float 和 double 的区别是什么？

![查看源图像](https://s2.loli.net/2022/01/04/tFTplwBnfmxehbS.png)

> 2.2 计算按揭贷款时，对于利率、本金和付款分别应该选择何种数据类型？说明你的理由

double类型，涉及浮点运算，精度比float高，运算代价比long double少

### 2.1.2节 练习

> 2.3 读程序写结果
>
> 2.4 编写程序检查你的估计是否正确

```cpp
#include <iostream>

int main(){
    unsigned u=10,u2=42;
    std::cout << u2-u <<std::endl;
    std::cout << u-u2 <<std::endl;//输出取模后的结果
    int i=10,i2=42;
    std::cout << i2-i <<std::endl;
    std::cout << i-i2 <<std::endl;
    std::cout << i-u <<std::endl;
    std::cout << u-i <<std::endl;
    return 0;
}
```

输出

```
32
4294967264
32
-32
0
0
```

### 2.1.3节 练习

> 2.5 指出下述字面值的数据类型并说明每一组内几种字面值的区别

```cpp
(a) 'a',L'a',"a",L"a"
(b) 10,10u,10L,10uL,012,0xC
(c) 3.14,3.14f,3.14L
(d) 10,10u,10.,10e-2
```

* 字符字面值，宽字符字面值，字符串字面值，宽字符串字面值
* 十进制整型，十进制无符号整型，十进制长整型，十进制无符号长整型， 八进制整型，十六进制整型
* double，float，long double
* 十进制整型，十进制无符号整型，double，double

> 2.6 下面两组定义是否有区别，如果有请叙述之

```cpp
int month = 9,day = 7;//十进制整型
int month = 09,day = 07;//字面值是八进制整型，month字面值超范围了
```

![image-20211219113222191](https://s2.loli.net/2021/12/19/opy8akxCBjrOLbN.png)

> 2.7 下述字面值表示何种含义？它们各自的数据类型是什么

```cpp
"Who goes with F\145rgus?\012"// \145是e \012是换行 
3.14eLL //浮点型没有LL
3.14e1L //很好我直接看错了，这个才是书上的例子，long doublel
1024f //整型没有f
3.14L//long double
```

![image-20211219113435868](../../../../../.config/Typora/typora-user-images/image-20211219113435868.png)

> 2.8 请利用转义序列编写一段程序，要求输出2M,然后转到新一行。修改程序，使其先输出2,然后输出制表符，在输出M，最后转到新一行

```cpp
#include <iostream>

int main(){
    std::cout << "2M\n";
    return 0;
}
```

```cpp
#include <iostream>

int main(){
    std::cout << "2\tM\n";
    return 0;
}
```

## 2.2节 练习

### 2.2.1节 练习

> 2.9 解释下列定义的含义，对于非法的定义，请说明错在何处并将其修正

```cpp
(a) std::cin >> int input_value;//先定义再使用
(b) int i = {3.14};//初始化列表存在数据丢失风险报错
(c) double salary = wage = 9999.99;//wage没定义
(d) int i = 3.14;//不报错，小数部分丢失
```

```cpp
int input_value;
std::cin >> input_value;
```

```cpp
int i = 3.14; 
```

```cpp
double wage;
double salary = wage = 9999.99;
```



> 2.10 下列变量的初值是分别是什么

```cpp
std::string global_str;
int global_int;
int main()
{
    int local_int;//0
    std::string local_str;//""
}
```



### 2.2.2节 练习

> 2.11 指出下列语句是声明还是定义

```cpp
(a) extern int ix = 1024;//定义
(b) int iy;//定义
(c) extern int iz;//声明
```



### 2.2.3节 练习

> 2.12 请指出下面名字中哪些是非法的

```cpp
(a) int double = 3.14;//非法，double关键字
(b) int _;
(c) int catch-22;//非法，运算符
(d) int 1_or_2 = 1;//非法，数字开头
(e) double Double = 3.14;
```

### 2.2.4节 练习

> 2.13 下面程序中 j 的值是多少

```cpp
int i =42;
int main(){
	int i=100;
	int j=i;
}
```

j=100



> 2.14 下面程序合法吗？如果合法，它将输出什么？

```cpp
int i = 100, sum = 0;
for(int i = 0; i != 10; ++i){
	sum += i;
}
std::cout << i << " " << sum << std::endl;
```

100 45

## 2.3节 练习

### 2.3.1节 练习

> 2.15 下面的哪个定义是不合法的？为什么？

```cpp
(a) int ival = 1.01;
(b) int &rval1 = 1.01;//不合法，不能绑定字面值
(c) int &rval2 = ival;
(d) int &ravl3;//不合法，没有初始化
```



> 2.16 考查下面的所有赋值然后回答：哪些赋值是不合法的？为什么？哪些赋值是合法的？它们执行了哪些操作？

```cpp
int i=0,&r1=i;
double d = 0, &r2 =d;
(a) r2 = 3.14159;//相当于d=3.14159, d变为3.14159
(b) r2 = r1;//相当于d=i, 类型自动转换(int->double)
(c) i = r2;//相当于i=d,小数截断
(d) r1 = d;//相当于i=d,小数截断
```

**弄清楚定义引用和使用引用的区别！！！** 2.15是定义引用，有很多规则。2.16是使用引用，当成别名来用



> 2.17 执行下面代码段将输出什么结果

```cpp
int i,&ri=i;
i=5;ri=10;
std::cout<<i<<" "<<ri<<std::endl;
```

10 10



### 2.3.2节 练习

> 2.18 编写代码分别更改指针的值以及指针所指对象的值

```cpp
#include <iostream>

int main(){
	int a=5,b=10;
    int *p=&a;
    std::cout << p << std::endl;
    std::cout << *p << std::endl;
    std::cout << a << std::endl;
    std::cout << b << std::endl;
    
    *p=20;//修改指向对象的值
    p=&b;//修改指针的值
    
    std::cout << p << std::endl;
    std::cout << *p << std::endl;
    std::cout << a << std::endl;
    std::cout << b << std::endl;
    
    return 0;
}
```

输出

```cpp
0x7ffdfb823028
5
5
10
0x7ffdfb82302c
10
20
10
```



> 2.19 说明指针和引用的主要区别

* 引用只是别名不是对象，指针是对象
* 引用必须初始化，指针可以不用初始化但是如果不初始化那么值是不确定的
* 无法令引用重新绑定新的对象，指针可以重新指向新的对象



> 2.20 请叙述下面这段代码的作用

```cpp
int i = 42;
int *p1 = &i;
*p1 = *p1 * *p1;//等价i=42*42;
```



> 2.21 请解释下述定义，在这些定义中有非法部分的吗？如果有，为什么？

```cpp
int i=0;
(a) double *dp = &i;//非法，类型不匹配
(b) int *ip=i;//非法，区分字面值0和等于0的变量
(c) int *p=&i;//合法
```



> 2.22 假设p是一个int型指针，请说明下述代码的含义

```cpp
if(p)//判断是不是空指针
if(*p)//判断指向的值是的真假
```



> 2.23 给定指针p,你能知道它是否指向了一个合法对象吗？如果可以，叙述思路；如果不可以，说明原因

网上回答能和不能都有，反正是讨论。

* 不能，因为没法判断指针指向的地址是否是正确的，C++也没检查指针是否初始化、是否指向正确对象
* 可以，直接看解引用后的内容，然后根据结果看。（但是这个结果又要怎么判断是否合法呢？）



> 2.24 下面这段代码中为什么p合法而lp不合法

```cpp
int i=42;
void *p = &i;
long *lp = &i;
```

指针类型要和指向的对象严格匹配，不过有特例

void* 可以指向任意类型的对象地址



### 2.3.3节 练习

> 2.25 说明下列变量的类型和值

```cpp
(a) int* ip,i,&r=i;
(b) int i,*ip=0;
(c) int* ip,ip2;
```

(a)

ip指向int的指针，值不确定

i是int类型，值不确定

r是指向int的引用，值为i

(b)

i是int类型，值不确定

ip是指向int的类型的空指针

(c)

ip指向int的指针，值不确定

ip2是int类型，值不确定



## 2.4节 练习

> 2.26 下面哪些句子是合法的？如果有不合法的句子，请说明为什么

```cpp
(a) const int buf;//不合法，没有初始化
(b) int cnt = 0;//合法
(c) const int sz = cnt;//合法
(d) ++cnt; ++sz;//++不合法，const对象不能修改
```



### 2.4.2节 练习

> 2.27 下面的哪些初始化是合法的？请说明原因

```cpp
(a) int i = -1,&r=0;
//r是指向int的引用，但他不是常量引用，不可以和字面值绑定

(b) int *const p2 = &i2;
//p2 是 指向 int 的 const 指针
//如果 i2 是 const int,不可以
//如果 i2 是 int,可以

(c) const int i = -1,&r=0;
//r 是指向const int的常量引用，可以和字面值绑定

(d) const int *const p3 = &i2;
//p3是指向 const int的常量指针，i2 是不是 const int 都可以

(e) const int *p1 = &i2;
//p1 是指向 const int的指针，i2 是不是 const int 都可以

(f) const int &const r2;
//不可以，const不能用于引用上，因为引用不是对象

(g) const int i2 = i,&r =i;
//i2 可以被其他对象初始化，是不是 i 是不是 const 没关系
//r 是指向const int的常量引用，用来初始化的是不是常量没关系
```



> 2.28 说明下面这些定义是什么意思，跳出其中不合法的部分

```cpp
(a) int i,*const cp;//不合法，const 指针必须初始化

(b) int *p1, *const p2;	//p2 是 const 指针，必须初始化

(c) const int ic, &r = ic;//ic 是个const int 必须初始化

(d) const int *const p3;//不合法，const 指针必须初始化

(e) const int *p;
```



> 2.29 假设已有上个练习中定义的那些变量，则下面那些语句是合法的？

```cpp
(a) i = ic;

(b) p1 = p3;//不合法1,p3是个 const 指针，p1不是指向常量的指针

(c) p1 = &ic;//不合法，ic是个常量，存放常量地址的值只能用指向常量的指针

(d) p3 = &ic;//不合法，p3是 const指针，初始化后不能重新赋值

(e) p2 = p1;//不合法，p2是 const 指针，初始化后不能重新赋值

(f) ic = *p3;//不合法，ic 是 const int,初始化后不能重新赋值
```

### 2.4.3节 练习

> 2.30 对于下面这些语句，请说明对象被声明成了顶层 const 还是底层 const？

``` c++
const int v2 = 0;//顶层const
int v1=v2;

int *p1 =&v1,&r1=v1;
//没有const

const int *p2 = &v2, *const p3 = &i, &r2 =v2;
//p2 底层const,p3底层和顶层const，r2底层const（没法顶层）
```



> 2.31 假设上已有一个练习中所作的那写声明，则下面的那些语句是合法的，请说明顶层const和底层const在每个例子中有和体现

```cpp
r1 = v2;//v2是顶层const 拷贝时不受影响
p1 = p2;//不合法 p2 是底层const,但 p1是不是底层const 
p2 = p1;//合法 p1是int* 可以转化为 const int*
p1 = p3;//不合法 p3 是顶层const也是底层const,但 p1是不是底层const 
p2 = p3;//合法，p2和p3 都是底层const,拷贝忽略顶层const
```



### 2.4.4节 练习

> 2.32 下面代码是否合法？如果非法，请设法将其修改正确

```cpp
int null = 0,*p = null;
```

非法，不能把 int 赋给 int*

```
int null = 0,*p = &null;
```



## 2.5节 练习

### 2.5.2节 练习

> 2.33 利用本节定义的变量，判断下列语句的运行结果
>
> 2.34 编写程序检查推断

```cpp
a = 42;
b = 42;
c = 42;
d = 42;
e = 42;
g = 42;
```

本节定义的变量

```cpp
#include<bits/stdc++.h>
using namespace std;

int main(){
	int i=0, &r = i;
	auto a = r;					//a是一个整数
	const int ci = i, &cr = ci;//ci是顶层const,cr是底层const
	auto b = ci;				//b是一个整数，顶层const被忽略
	auto c = cr;				//c是一个整数，auto引用推断的是引用指向对象的类型
								//也就是ci，它的顶层const还是被忽略了
	
	auto d = &i;				//d是 int*
	auto e = &ci;				//c是 const int*,对常量对象去地址是一种底层const
	const auto f = ci;			//ci的推演类型是int,f类型是 const int
	auto &g = ci;				//ci的推演类型是int,g类型是 int&
	
    a = 42;
    b = 42;
    c = 42;
    d = 42;						//d int* 非法
    e = 42;						//e const int* 非法
    g = 42;

	return 0;
}
```



> 2.35 判断下列定义推断出的类型是什么，然后编写程序进行验证

```cpp
#include<bits/stdc++.h>
using namespace std;

int main(){
	const int i = 42;
	auto j = i;					//j为int
	const auto &k = i;			//k为 const int & 指向整型常量的引用
	auto *p = &i;				//p为 const int*，对常量对象的取地址是底层const
	const auto j2 = i,&k2=i;	//j2为 const int,k2 为 const int &
	return 0;
}
```

### 2.5.3节 练习

> 2.36 关于下面的代码，请指出每一个变量的类型以及程序结束时它们各自的值

```cpp
#include<bits/stdc++.h>
using namespace std;

int main(){
	int a = 3, b = 4;
	decltype(a) c = a;//int c = a;
	decltype((b)) d =a;// int &d =a;
	++c;//4
	++d;//4
    cout<<a<<endl;//4
    cout<<b<<endl;//4
    cout<<c<<endl;//4
    cout<<d<<endl;//4
    
	return 0;
}
```



> 2.37 赋值是会产生引用的一类典型表达式，引用类型就是左值的类型。也就是说，如果i是int,则表达式i=x的类型是int&。根据这一特点，请指出下面代码中每个变量的类型和值

```cpp
int a =3,b =4;
decltype(a) c =a;//c int
decltype(a=b) d = a;//d int&
```



> 2.38 说明由 decltype 指定类型和 有 auto指定类型有何区别。请举出一个例子，decltype 指定的类型与 atuo类型一样，再举一个例子与auto指定的类型不同

* auto 会忽略顶层const,decltype会包括顶层const
* decltype的结果类型和表达式形式密切相关

```cpp
#include<bits/stdc++.h>
using namespace std;

int main(){
	int i=0,&r=i;
	//相同
	auto a = i;//int
	decltype (i) b = i;//int
	//不同
	auto c=r;//int
	decltype(r) d = r;//int &

	return 0;
}
```

## 2.6节 练习

### 2.6.1节 练习

> 2.39 观察运行一下程序，注意如果忘记写类定义体后面的分号会发生什么？记录相关信息，以后可能用到

```cpp
#include<bits/stdc++.h>
using namespace std;

struct Foo{}

int main(){
	

	return 0;
}
```

报错：结构体定义后需要';'



> 2.40 根据自己的理解写出 Sales_data 类，最好和书上的有些区别

```cpp
struct Sales_data {
	std::string bookNo;
	unsigned units_sold = 0;
	double revenue = 0.0;
};
```

### 

### 2.6.2节 练习

> 2.41 使用你自己的类重写1.5.1节(第20页)、1.5.2节(第21页)和1.6节(第22页)的练习。眼下先把 Sales_data类的定义和main函数放同一个文件

**1.5.1** 

> 1.20 在网站http://www.informit.com/title/032174113 上，第1章的代码目录包含了头文件 Sales_item.h。将它拷贝到你自己的工作目录中。用它编写一个程序，读取一组书籍销售记录，将每条记录打印到标准输出上。



 ```cpp
#include <iostream>

struct Sales_data {
	std::string bookNo;
	unsigned units_sold = 0;
	double revenue = 0.0;
};

int main(){
	Sales_data item;
	double price;
	while(std::cin >> item.bookNo >> item.units_sold>>price){
		item.revenue=item.units_sold * price;
		std::cout << item.bookNo << " " << item.units_sold << " " << item.revenue << std::endl;
    }
	return 0;
}
 ```



> 1.21 编写程序，读取两个 ISBN 相同的 Sales_item 对象，输出它们的和

```cpp
#include <iostream>

struct Sales_data {
	std::string bookNo;
	unsigned units_sold = 0;
	double revenue = 0.0;
};

int main(){
	Sales_data item_a,item_b;
	double price;
	std::cin >> item_a.bookNo >> item_a.units_sold>>price;
	item_a.revenue=item_a.units_sold * price;
	std::cin >> item_b.bookNo >> item_b.units_sold>>price;
	item_b.revenue=item_b.units_sold * price;
	std::cout<< item_a.bookNo << " "
			<< item_a.units_sold + item_b.units_sold << " "//units_sold sum
			<< item_a.revenue + item_b.revenue << " "//revenue sum
			<< (item_a.revenue + item_b.revenue)/(item_a.units_sold + item_b.units_sold)//ave
			<< std::endl;

	return 0;
}
```

> 1.22 编写程序，读取多个具有相同 ISBN的 Sales_item 对象，输出所有记录的和

```cpp
#include <iostream>

struct Sales_data {
	std::string bookNo;
	unsigned units_sold = 0;
	double revenue = 0.0;
};

int main(){
	Sales_data item,item_sum;
	double price;
    while(std::cin >> item.bookNo >> item.units_sold >> price){
        item.revenue = item.units_sold * price;
		item_sum.bookNo = item.bookNo;
		item_sum.units_sold += item.units_sold;
		item_sum.revenue += item.revenue;

    }
    std::cout<< item_sum.bookNo << " "
			<< item_sum.units_sold << " "//units_sold sum
			<< item_sum.revenue << " "//revenue sum
			<< item_sum.revenue / item_sum.units_sold //ave
			<< std::endl;

	return 0;
}
```

**1.5.2+1.6**

书店程序

```cpp
#include <iostream>

struct Sales_data {
	std::string bookNo;
	unsigned units_sold = 0;
	double revenue = 0.0;
};

int main(){
	Sales_data total,item;
	double price;
    if (std::cin >> total.bookNo >> total.units_sold >> price){
		total.revenue = total.units_sold * price;
        while (std::cin >> item.bookNo >> item.units_sold >> price){
			item.revenue = item.units_sold * price;
            if (total.bookNo == item.bookNo) {
                total.units_sold += item.units_sold;
                total.revenue += item.revenue;
            }else {
                std::cout << total.bookNo << " "
						<< total.units_sold << " "
						<< total.revenue << std::endl;
                total = item;
            }
        }
		std::cout << total.bookNo << " "
						<< total.units_sold << " "
						<< total.revenue << std::endl;
    }else {
        std::cerr << "No data?!" << std::endl;
        return -1;
    }

	return 0;
}
```



### 2.6.3节 练习

> 2.42 根据你的理解重写一个 Sales_data.h 头文件，并一次为基础重做 2.6.2节练习

include进去就行了

```cpp
include "Sales_data.h"
```



```cpp
#ifndef SALES_DATA_H
#define SALES_DATA_H

#include <string>

struct Sales_data {
	std::string bookNo;
	unsigned units_sold = 0;
	double revenue = 0.0;
};
#endif

```

