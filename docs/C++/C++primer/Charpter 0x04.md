<h1 align="center">📔 C++ Primer 0x04 练习题解</h1>

## 4.1 基础

### 4.1.2 优先级与结合律

> 4.1 表达式 5 + 10 * 20 / 2 的求值结果是多少

105

> 4.2 根据 4.12节中的表，在下述表达式的合理位置添加括号，使得添加括号后，运算对象的组合顺序与添括号前一致

```cpp
(a) *vec.begin()
(b) *vec.begin() + 1
```

```cpp
(a) *(vec.begin())
(b) (*(vec.begin())) + 1
```

### 4.1.3 求值顺序

> 4.3 C++语言没有明确规定大多数二元运算符的求值顺序，给编译器优化留下了余地。这种策略实际上是在代码生成效率和程序潜在缺陷之间进行了权衡，你认为这可以接受吗？请说出你的理由。

可以接受，给了程序员更多自由，C++ 信任程序员，当然这也要求我们有较高的水平了。



## 4.2算术运算符

> 4.4 在下面的表达式中添加括号，说明其求值过程及最终结果。编写程序编译该（不加括号的）表达式并输出结果验证之前的推断。

 ```cpp
 12 / 3 * 4 + 5 * 15 + 24 % 4 / 2
 ```



```cpp
((12 / 3) * 4) + (5 * 15) + ((24 % 4) / 2)
//16 + 75 + 0
//91
```



> 4.5 写出下列表达式的求值结果。

```cpp
-30 * 3 + 21 / 5  //-86
-30 + 3 * 21 / 5  //-18
30 / 3 * 21 % 5   //0
-30 / 3 * 21 % 4  //-2
```



> 4.6 写出一条表达式用于确定一个整数是奇数还是偶数。

```cpp
if(i&1){/*...*/}
```



> 4.7 溢出是何含义？写出三条将导致溢出的表达式。

当计算的结果超出该类型所能表示的范围时就会产生溢出。

```cpp
short short_value = 32767; short_value += 1;
unsigned unsigned_value = 0; unsigned_value -=1;
unsigned short uunsigned_short_svalue = 65535; ++uunsigned_short_svalue; 
```



## 4.3逻辑和关系运算符 

> 4.8 说明在逻辑与、逻辑或及相等性运算符中运算对象的求值顺序。

* 逻辑与逻辑或求值顺序为短路求值
* 相等运算未定义求值顺序



> 4.9 解释在下面的 if 语句中条件部分的判断过程。

```cpp
const char *cp = "Hello World";
if (cp && *cp)
```

如果cp不是一个空指针，再看*cp，也就是'H'。结果为真



> 4.10 为 while 循环写一个条件，使其从标准输入中读取整数，遇到 42 时停止。

```cpp
int x;
while(cin>>x && x!=42){/*...*/}
```



> 4.11 书写一条表达式用于测试4个值a、b、c、d的关系，确保a大于b、b大于c、c大于d。

```cpp
a > b && b > c && c > d
```



> 4.12 假设 i 、j 和 k 是三个整数，说明表达式i != j < k的含义。

首先看j<k，如果j<k返回值为true,表达式变为 i != 1

如果j<k为假返回值为false,表达式变为 i !=0

最终返回的结果还是布尔值

**注意 `true` 和 `false` 变为 `int` 的过程**



## 4.4赋值运算符

> 4.13 在下述语句中，当赋值完成后 i 和 d 的值分别是多少？

```cpp
int i;   double d;
d = i = 3.5; // i = 3, d = 3.0
i = d = 3.5; // d = 3.5, i = 3
```



> 4.14 执行下述 if 语句后将发生什么情况？

```cpp
if (42 = i)   // 赋值运算符的左侧运算对象必须是一个可以修改的左值，编译报错
if (i = 42)   // 真
```



> 4.15 下面的赋值是非法的，为什么？应该如何修改？

```cpp
double dval; int ival; int *pi;
dval = ival = pi = 0;
```

`ival = pi` 这部分会出错，`pi` 是指针不能赋值给 `ival`

```cpp
double dval; int ival; int *pi;
dval = ival = 0;
pi = 0;
```



> 4.16 尽管下面的语句合法，但它们实际执行的行为可能和预期并不一样，为什么？应该如何修改？

```cpp
if (p = getPtr() != 0)//赋值运算符优先级低
if (i = 1024)//区分赋值运算符和相等运算符
```

```cpp
if ((p = getPtr()) != 0)
if (i == 1024)
```



## 4.5 递增和递减运算符

> 4.17 说明前置递增运算符和后置递增运算符的区别。

前置版本把值+1后直接返回了运算对象(左值）；后置版本需要把原始值存下来，以便返回未修改的内容（右值）



> 4.18  如果132页那个输出 vector 对象元素的 while 循环使用前置递增运算符，将得到什么结果？

从第二个元素开始输出，最终会对 end 解引用，出错



> 4.19 假设`ptr`的类型是指向`int`的指针、`vec`的类型是`vector`、`ival`的类型是`int`，说明下面的表达式是何含义？如果有表达式不正确，为什么？应该如何修改？

```cpp
(a) ptr != 0 && *ptr++  //如果ptr非空，并且ptr指向的值为真，那么ptr指向下一个
(b) ival++ && ival //如果ival为真并且ival+1也为真，那么结果为真
(c) vec[ival++] <= vec[ival] //不正确，没有规定<=两侧的求值顺序
//(c) 修改
// vec[ival] <= vec[ival+1]
```



## 4.6 成员访问运算符

> 4.20 假设`iter`的类型是`vector<string>::iterator`, 说明下面的表达式是否合法。如果合法，表达式的含义是什么？如果不合法，错在何处?

```cpp
(a) *iter++;//返回迭代器所指元素后迭代器递增1
(b) (*iter)++;//不合法 string 不支持递增运算符
(c) *iter.empty();//不合法，解引用运算符优先级比点运算符低
(d) iter->empty();//判断迭代器所指元素是否为空
(e) ++*iter;//不合法 string 不支持递增运算符
(f) iter++->empty();//判断迭代器所指元素是否为空然后迭代器递增
```



## 4.7 条件运算符

> 4.21 编写一段程序，使用条件运算符从`vector<int>`中找到哪些元素的值是奇数，然后将这些奇数值翻倍。

```cpp
#include <iostream>
#include <vector>

using std::vector;
using std::cout;

int main(){
	vector<int>vec={1,2,3,4,5};
	for(auto &v:vec)v=(v&1?2*v:v); 
	for(auto v:vec)cout << v <<" ";
    return 0;
}
```



> 4.22 本节的示例程序将成绩划分为`high pass`、`pass` 和 `fail` 三种，扩展该程序使其进一步将 60 分到 75 分之间的成绩设定为`low pass`。要求程序包含两个版本：一个版本只使用条件运算符；另一个版本使用1个或多个`if`语句。哪个版本的程序更容易理解呢？为什么？

```cpp
#include <iostream>
#include <string>

using std::cout;
using std::cin;
using std::string;

int main(){
	int grade;cin>>grade;
	string finalgrade = (grade > 90) ? "hight pass" 
									: (grade < 60) ? "fail" : "pass";
	cout << finalgrade;
	return 0;
}
```



```cpp
#include <iostream>
#include <string>

using std::cout;
using std::cin;
using std::string;

int main(){
	int grade;cin>>grade;
	string finalgrade;
	
	if(grade > 90)finalgrade = "hight pass";
	else if(grade >= 60)finalgrade = "pass";
	else finalgrade = "fail";
	cout << finalgrade;

	return 0;
}
```

第二个版本好，嵌套多了条件运算符可读性降低，if else 更清晰



> 4.23 因为运算符的优先级问题，下面这条表达式无法通过编译。根据4.12节中的表指出它的问题在哪里？应该如何修改？

```cpp
string s = "word";
string pl = s + s[s.size() - 1] == 's' ? "" : "s" ;
```

加法优先级高于条件运算符优先级，因此等价于下面的代码

```cpp
string s = "word";
string pl = (s + s[s.size() - 1]) == 's' ? "" : "s" ;
```

修改为

```cpp
string s = "word";
string pl = s + (s[s.size() - 1] == 's' ? "" : "s") ;
```



> 4.24  本节的示例程序将成绩划分为`high pass`、`pass`、和`fail`三种，它的依据是条件运算符满足右结合律。假如条件运算符满足的是左结合律，求值的过程将是怎样的？

```cpp
string finalgrade = (grade > 90) ? "hight pass" 
									: (grade < 60) ? "fail" : "pass";
```

如果左结合律那么等价于

```cpp
string finalgrade = ((grade > 90) ? "hight pass" 
									: (grade < 60)) ? "fail" : "pass";
```



## 4.8 位运算符

> 4.25 如果一台机器上`int`占32位、`char`占8位，用的是`Latin-1`字符集，其中字符`'q'` 的二进制形式是`01110001`，那么表达式`~'q' << 6`的值是什么？

`char` 先提升成 `int`

`00000000 00000000 00000000 01110001`

然后取反

`11111111 11111111 11111111 10001110`

然后左移动6位

`11111111 11111111 11100011 10000000`



> 4.26 在本节关于测验成绩的例子中，如果使用`unsigned int` 作为`quiz1` 的类型会发生什么情况？

未定义，有的机器 `unsigned int `只有16位



> 4.27 下列表达式的结果是什么？

```cpp
unsigned long ul1 = 3, ul2 = 7;
(a) ul1 & ul2 //011 3
(b) ul1 | ul2 //111 7
(c) ul1 && ul2//true
(d) ul1 || ul2 //true
```

`ul1 = 011`

`ul2 = 111`



## 4.9 sizeof 运算符

> 4.28 编写一段程序，输出每一种内置类型所占空间的大小。

```cpp
#include<iostream>
using namespace std;

int main(){
	cout << "bool size = " << sizeof(bool) <<" bytes" << endl;
	cout << "char size = " << sizeof(char) <<" bytes" << endl;
	cout << "wchar_t size = " << sizeof(wchar_t) <<" bytes" << endl;
	cout << "char16_t size = " << sizeof(char16_t) <<" bytes" << endl;
	cout << "char32_t size = " << sizeof(char32_t) <<" bytes" << endl;
	cout << "short size = " << sizeof(short) <<" bytes" << endl;
	cout << "int size = " << sizeof(int) <<" bytes" << endl;
	cout << "long size = " << sizeof(long) <<" bytes" << endl;
	cout << "long long  size = " << sizeof(long long) <<" bytes" << endl;
	cout << "float size = " << sizeof(float) <<" bytes" << endl;
	cout << "double size = " << sizeof(double) <<" bytes" << endl;
	cout << "long double size = " << sizeof(long double) <<" bytes" << endl;
	return 0;
}

```



> 4.29 推断下面代码的输出结果并说明理由。实际运行这段程序，结果和你想象的一样吗？如不一样，为什么？

```cpp
int x[10];   int *p = x;
cout << sizeof(x)/sizeof(*x) << endl;//10
//sizeof 不会把数组名字转成指针，sizeof(x)大小为数组内元素大小之和
cout << sizeof(p)/sizeof(*p) << endl;//2
//p 是int* 类型 sizeof(p) = 8
```



> 4.30 根据4.12节中的表，在下述表达式的适当位置加上括号，使得加上括号之后的表达式的含义与原来的含义相同。

```cpp
(a) sizeof x + y      
(b) sizeof p->mem[i]  
(c) sizeof a < b     
(d) sizeof f() 
```

```cpp
(a) (sizeof x) + y      
(b) sizeof (p->mem[i])  
(c) (sizeof a) < b     
(d) sizeof (f()) 
```

## 4.10 逗号运算符

> 4.31 本节的程序使用了前置版本的递增运算符和递减运算符，解释为什么要用前置版本而不用后置版本。要想使用后置版本的递增递减运算符需要做哪些改动？使用后置版本重写本节的程序。

前置版本把值+1后直接返回了运算对象；后置版本需要把原始值存下来，以便返回未修改的内容

```cpp
#include <vector>
#include <iostream>
using namespace std;

int main(){
	vector<int>ivec(10);
	vector<int>::size_type cnt =ivec.size();
	
	for(vector<int>::size_type ix = 0; ix != ivec.size(); ix++, --cnt)
		ivec[ix] = cnt;
	
	for(auto v:ivec)cout << v << " ";
	return 0;
}

```



> 4.32 解释下面这个循环的含义

```cpp
constexpr int size = 5;
int ia[size] = { 1, 2, 3, 4, 5 };
for (int *ptr = ia, ix = 0;ix != size && ptr != ia+size;++ix, ++ptr) 
{
    /* ... */ 
}
```

  循环遍历数组 ia 并计数



> 4.33 根据4.12节中的表说明下面这条表达式的含义

```cpp
someValue ? ++x, ++y : --x, --y
```

等价

```cpp
(someValue ? ++x, ++y : --x), --y
```

如果 `somevalue`为真，那么执行 `++x`和`++y`,返回`y`。然后丢掉`y`值，执行`--y`,返回`y值`

如果 `somevalue`为假，那么执行 `--x`,返回`x`。然后丢掉`x`值，执行`--y`,返回`y值`

```cpp
#include <vector>
#include <iostream>
using namespace std;

int main(){
	int x=10,xx=10,y=0,yy=0;
	bool somevalue = true;
	cout << (somevalue ? ++x,++y:--x) << endl;//输出1
	cout << ((somevalue ? ++xx,++yy:--xx),--yy) << endl;//输出0	
	return 0;
}
```



## 4.11 类型转换

### 4.11.1 算术转换

> 4.34 根据本节给出的变量定义，说明在下面的表达式中将发生什么样的类型转换：

```cpp
(a) if (fval)//转为bool
(b) dval = fval + ival;//ival转为float,最终结果转为double
(c) dval + ival * cval;//cval转为int，ival*cval结果转为double
```



> 4.35 假设有如下的定义：

```cpp
char cval;
int ival;
unsigned int ui;
float fval;
double dval;
```

请回答在下面的表达式中发生了隐式类型转换吗？如果有，指出来。

```cpp
(a) cval = 'a' + 3;//'a'转为int,最终转为char
(b) fval = ui - ival * 1.0;//ival转为double,ui转为double,最终转为float
(c) dval = ui * fval;//ui转为float，最终转为double
(d) cval = ival + fval + dval;//ival 转为float,相加结果转为double,最终结果转为char
```

### 4.11.3 显示转换

> 4.36 假设 `i` 是`int`类型，`d` 是`double`类型，书写表达式 `i*=d` 使其执行整数类型的乘法而非浮点类型的乘法。

```cpp
i*=static_cast<int>(d);
```



> 4.37 用命名的强制类型转换改写下列旧式的转换语句

```cpp
int i; double d; const string *ps; char *pc; void *pv;
(a) pv = (void*)ps;
(b) i = int(*pc);
(c) pv = &d;
(d) pc = (char*)pv;
```

```cpp
(a) pv = static_cast<void*>(const_cast<string*>(ps));
(b) i = static_cast<int>((*pc));
(c) pv = static_cast<void*>((&d));
(d) pc = static_cast<char*>(pv);
```



> 4.38 说明下面这条表达式的含义。

```cpp
double slope = static_cast<double>(j/i);
```

`j/i`强制类型转换为`double`然后赋值给`slope`
