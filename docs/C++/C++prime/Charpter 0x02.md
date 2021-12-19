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

