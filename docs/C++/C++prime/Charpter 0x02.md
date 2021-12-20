<h1 align="center">📔 C++ Prime 0x02 练习题解</h1>

## 2.1节 练习

### 2.1.1节 练习

> 2.1 类型 int、long、long long 和 short 的区别是什么？无符号类型和带符号类型的区别是什么？ float 和 double 的区别是什么？

![查看源图像](https://tse1-mm.cn.bing.net/th/id/R-C.ed92105fcbcb8f224a4ee2000a36cd87?rik=NwIj%2fjEPdcuKsw&riu=http%3a%2f%2fconglang.github.io%2fimg%2fcpp_primer5_arithmetic_types.png&ehk=2Yezw3g8w2w2H6cDgP%2b7L5T3bMTOrrvCz6MbV8wgaWs%3d&risl=&pid=ImgRaw&r=0)

> 2.2 计算按揭贷款时，对于利率、本金和付款分别应该选择何种数据类型？说明你的理由

double类型，涉及浮点运算，精度比float高，运算代价比long double少

### 2.1.2节 练习

> 2.3 读程序写结果
>
> 2.4 编写程序检查你的估计是否正确

```c++
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

```c++
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

```c++
int month = 9,day = 7;//十进制整型
int month = 09,day = 07;//字面值是八进制整型，month字面值超范围了
```

![image-20211219113222191](https://s2.loli.net/2021/12/19/opy8akxCBjrOLbN.png)

> 2.7 下述字面值表示何种含义？它们各自的数据类型是什么

```c++
"Who goes with F\145rgus?\012"// \145是e \012是换行 
3.14eLL //浮点型没有LL
3.14e1L //很好我直接看错了，这个才是书上的例子，long doublel
1024f //整型没有f
3.14L//long double
```

![image-20211219113435868](../../../../../.config/Typora/typora-user-images/image-20211219113435868.png)

> 2.8 请利用转义序列编写一段程序，要求输出2M,然后转到新一行。修改程序，使其先输出2,然后输出制表符，在输出M，最后转到新一行

```c++
#include <iostream>

int main(){
    std::cout << "2M\n";
    return 0;
}
```

```c++
#include <iostream>

int main(){
    std::cout << "2\tM\n";
    return 0;
}
```

## 2.2节 练习

### 2.2.1节 练习

> 2.9 解释下列定义的含义，对于非法的定义，请说明错在何处并将其修正

```c++
(a) std::cin >> int input_value;//先定义再使用
(b) int i = {3.14};//初始化列表存在数据丢失风险报错
(c) double salary = wage = 9999.99;//wage没定义
(d) int i = 3.14;//不报错，小数部分丢失
```

```c++
int input_value;
std::cin >> input_value;
```

```c++
int i = 3.14; 
```

```c++
double wage;
double salary = wage = 9999.99;
```



> 2.10 下列变量的初值是分别是什么

```c++
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

```c++
(a) extern int ix = 1024;//定义
(b) int iy;//定义
(c) extern int iz;//声明
```



### 2.2.3节 练习

> 2.12 请指出下面名字中哪些是非法的

```c++
(a) int double = 3.14;//非法，double关键字
(b) int _;
(c) int catch-22;//非法，运算符
(d) int 1_or_2 = 1;//非法，数字开头
(e) double Double = 3.14;
```

### 2.2.4节 练习

> 2.13 下面程序中 j 的值是多少

```c++
int i =42;
int main(){
	int i=100;
	int j=i;
}
```

j=100



> 2.14 下面程序合法吗？如果合法，它将输出什么？

```c++
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

```c++
(a) int ival = 1.01;
(b) int &rval1 = 1.01;//不合法，不能绑定字面值
(c) int &rval2 = ival;
(d) int &ravl3;//不合法，没有初始化
```



> 2.16 考查下面的所有赋值然后回答：哪些赋值是不合法的？为什么？哪些赋值是合法的？它们执行了哪些操作？

```c++
int i=0,&r1=i;
double d = 0, &r2 =d;
(a) r2 = 3.14159;//相当于d=3.14159, d变为3.14159
(b) r2 = r1;//相当于d=i, 类型自动转换(int->double)
(c) i = r2;//相当于i=d,小数截断
(d) r1 = d;//相当于i=d,小数截断
```

**弄清楚定义引用和使用引用的区别！！！** 2.15是定义引用，有很多规则。2.16是使用引用，当成别名来用



> 2.17 执行下面代码段将输出什么结果

```c++
int i,&ri=i;
i=5;ri=10;
std::cout<<i<<" "<<ri<<std::endl;
```

10 10



### 2.3.2节 练习

> 2.18 编写代码分别更改指针的值以及指针所指对象的值

```c++
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

```c++
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





> 2.20 请叙述下面这段带暧昧的作用

```c++
int i = 42;
int *p1 = &i;
*p1 = *p1 * *p1;//等价i=42*42;
```



> 2.21 请解释下述定义，在这些定义中有非法部分的吗？如果有，为什么？

```c++
int i=0;
(a) double *dp = &i;//非法，类型不匹配
(b) int *ip=i;//非法，区分字面值0和等于0的变量
(c) int *p=&i;//合法
```



> 2.22 假设p是一个int型指针，请说明下述代码的含义

```c++
if(p)//判断是不是空指针
if(*p)//判断指向的值是的真假
```



> 2.23 给定指针p,你能知道它是否指向了一个合法对象对象吗？如果可以，叙述思路；如果不可以，说明原因





> 2.24 下面这段代码中为什么p合法而lp不合法

```c++
int i=42;
void *p = &i;
long *lp = &i;
```

指针类型要和指向的对象严格匹配，不过有特例

void* 可以指向任意类型的对象地址
