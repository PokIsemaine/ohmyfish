<h1 align="center">📔 C++ Prime 0x06 练习题解</h1>

## 6.1 函数基础

> 6.1 实参和形参的区别的什么？

实参是形参的初始值,用来定义和初始化形参

实参是函数调用时的实际值



> 6.2 请指出下列函数哪个有错误，为什么？应该如何修改这些错误呢？

```cpp
(a) int f() {
         string s;
         // ...
         return s;//返回值为int
   }
(b) f2(int i) { /* ... */ }//没定义返回值
(c) int calc(int v1, int v1) { /* ... */ }//形参名字重复
(d) double square (double x)  return x * x; //函数体没用花括号括起来
```

```cpp
(a) string f() {
         string s;
         // ...
         return s;//返回值为int
   }
(b) void f2(int i) { /* ... */ }//没定义返回值
(c) int calc(int v1, int v2) { /* ... */ }//形参名字重复
(d) double square (double x) { return x * x; } //函数体没用花括号括起来
```



> 6.3 编写你自己的`fact`函数，上机检查是否正确。注：阶乘。
>
> 6.4 编写一个与用户交互的函数，要求用户输入一个数字，计算生成该数字的阶乘。在`main`函数中调用该函数。

```cpp
#include<bits/stdc++.h>
using namespace std;

long long fact(int n){
	long long ret=1;
	for(int i=2;i<=n;++i){
		ret*=i;
	}
	return ret;
}

int main(){
	int n;cin>>n;
	cout << fact(n) << endl;	
	return 0;
}
```



> 6.5 编写一个函数输出其实参的绝对值。

```cpp
#include<bits/stdc++.h>
using namespace std;

int get_abs(int n){
	return n>0?n:-n;
}

int main(){
	int n;cin>>n;
	cout << get_abs(n) << endl;	
	return 0;
}
```



### 6.1.1 局部对象

> 6. 6 说明形参、局部变量以及局部静态变量的区别。编写一个函数，同时达到这三种形式。

形参定义在函数形参列表里

普通局部变量属于自动对象，函数控制路径经过定义语句时创建，到达定义所在块尾销毁

局部静态变量是将普通变量定义成`static`类型，函数控制路径**第一次经过**定义语句时创建，**程序结束时销毁**

```cpp
#include<bits/stdc++.h>
using namespace std;

size_t count_calls(int i){
	static size_t ctr = 0;
	int add = i;
	ctr += add;
	return ctr;
}

int main(){
	for(size_t i = 0 ;i != 10 ;++i){
		cout << count_calls(i) << endl;
	}	
	return 0;
}
```



> 6.7 编写一个函数，当它第一次被调用时返回0，以后每次被调用返回值加1。

```cpp
#include<bits/stdc++.h>
using namespace std;

size_t count_calls(){
	static size_t ctr = 0;
	return ctr++;
}

int main(){
	for(size_t i = 0 ;i != 10 ;++i){
		cout << count_calls() << endl;
	}	
	return 0;
}
```

### 6.1.2 函数声明

> 6.8 编写一个名为`Chapter6.h` 的头文件，令其包含6.1节练习中的函数声明。

```cpp
long long fact(int n);
int get_abs(int n);
```

### 6.1.3 分离式编译

> 6.9 编写你自己的`fact.cc `和`factMain.cc` ，这两个文件都应该包含上一小节的练习中编写的` Chapter6.h` 头文件。通过这些文件，理解你的编译器是如何支持分离式编译的。

`Chapter6.h`

```cpp
#ifndef Chapter6_H
#define Chapter6_H

long long fact(int n);
int get_abs(int n);

#endif

```

`fact.cc`

```cpp
#include "Chapter6.h"

long long fact(int n){
        long long ret = 1;
        for(int i = 2; i <= n; ++i){
                ret *= i;
        }
        return ret;
}

int get_abs(int n){
        return n>0?n:-n;
}
```

`factMain.cc`

```cpp
#include "Chapter6.h"
#include <iostream>
using namespace std;

int main(){
        int n;cin>>n;
        int k = get_abs(n);
        cout << fact(k) << endl;
}
```

编译命令:`clang++ factMain.cc fact.cpp` 没装`clang++`也可以用`g++`

运行：`./a.out`



## 6.2 参数传递

### 6.2.1 传值参数

> 6.10 编写一个函数，使用指针形参交换两个整数的值。 在代码中调用该函数并输出交换后的结果，以此验证函数的正确性。

```cpp
#include<bits/stdc++.h>
using namespace std;

void swap(int* a,int *b){
	int tmp = *a;
	*a = *b;
	*b = tmp;
}

int main(){
	int a,b;cin>>a>>b;
	swap(&a,&b);
	cout << a << " " << b << endl;
	return 0;
}
```



### 6.2.2 传引用参数

> 6.11 编写并验证你自己的reset函数，使其作用于引用类型的参数。注：reset即置0。

```cpp
void reset(int &i){
	i = 0;
}
```



> 6.12 改写6.2.1节练习中的程序，使其引用而非指针交换两个整数的值。你觉得哪种方法更易于使用呢？为什么？

```cpp
#include<bits/stdc++.h>
using namespace std;

void swap(int &a,int &b){
	int tmp = a;
	a = b;
	b = tmp;
}

int main(){
	int a,b;cin>>a>>b;
	swap(a,b);
	cout << a << " " << b << endl;
	return 0;
}
```

引用写更方便，不用传地址，也不用解引用



> 6.13 假设`T`是某种类型的名字，说明以下两个函数声明的区别： 一个是`void f(T)`, 另一个是`void f(&T)`。

`void f(T)`传值，改变T不会影响实参

`void f(&T)`传引用，改变T会影响实参



> 6.14 举一个形参应该是引用类型的例子，再举一个形参不能是引用类型的例子。

形参应该是引用类型：见练习 6.12 交换两个整数

形参不能是引用：实参是字面值、表达式、需要转换的不同类型的对象时（没法引用的）

```cpp
#include<bits/stdc++.h>
using namespace std;

int add(int &a,int &b){
	return a+b;
}

int main(){
	cout << add(1,3) << endl;//会报错
}
```



> 6.15 说明`find_char`函数中的三个形参为什么是现在的类型，特别说明为什么`s`是常量引用而`occurs`是普通引用？ 为什么`s`和`occurs`是引用类型而`c`不是？ 如果令`s`是普通引用会发生什么情况？ 如果令`occurs`是常量引用会发生什么情况？

为什么`s`是常量引用而`occurs`是普通引用？

我们只是遍历`s`不希望改变`s`的内容所以使用常量引用，`occurs`作为额外返回的信息，需要被改变不能使用常量引用

 为什么`s`和`occurs`是引用类型而`c`不是？

`c`的实参可能是字面值，不能引用

 如果令`s`是普通引用会发生什么情况？

本程序正常情况不会发生什么，但是常量引用可以保护`s`不被修改。而且如果s为字面值的话，普通常量引用会编译失败

如果令`occurs`是常量引用会发生什么情况？

不能修改`occurs`的值，发生错误



### 6.2.3 const 形参和实参

> 6.16 下面的这个函数虽然合法，但是不算特别有用。指出它的局限性并设法改善。

```cpp
bool is_empty(string& s) { return s.empty(); }
```

字符串字面值，常量字符串等不能作为实参

```cpp
bool is_empty(const string& s) { return s.empty(); }
```



> 6.17 编写一个函数，判断`string`对象中是否含有大写字母。 编写另一个函数，把`string`对象全部改写成小写形式。 在这两个函数中你使用的形参类型相同吗？为什么？

不一样，第一个不需要修改传常量引用，第二个要修改传普通引用



> 6.18 为下面的函数编写函数声明，从给定的名字中推测函数具备的功能。
>
> (a) 名为`compare`的函数，返回布尔值，两个参数都是`matrix`类的引用。
>
> (b) 名为`change_val`的函数，返回`vector`的迭代器，有两个参数：一个是`int`，另一个是`vector<int>`的迭代器。

```cpp
bool compare(const matrix &,const matrix &);
vector<int> change_val(cint,vector<int>::iterator);
```



> 6.19 假定有如下声明，判断哪个调用合法、哪个调用不合法。对于不合法的函数调用，说明原因。

```cpp
double calc(double);
int count(const string &, char);
int sum(vector<int>::iterator, vector<int>::iterator, int);
vector<int> vec(10);
(a) calc(23.4, 55.1);//不合法，实参过多
(b) count("abcda",'a');
(c) calc(66);
(d) sum(vec.begin(), vec.end(), 3.8);
```

### 6.2.4 数组传参

> 6.21 编写一个函数，令其接受两个参数：一个是`int`型的数，另一个是`int`指针。 函数比较`int`的值和指针所指的值，返回较大的那个。 在该函数中指针的类型应该是什么？

```cpp
#include<bits/stdc++.h>
using namespace std;

int cmp(const int a,const int *b){
	return a>(*b)?a:(*b);
}

int main(){
	int a,b;cin>>a>>b;
	cout << cmp(a,&b) << endl;
	cout << cmp(7,&b) << endl;
	
	return 0;
}
```



> 6.22 编写一个函数，令其交换两个`int`指针。

```cpp
#include<bits/stdc++.h>
using namespace std;

void swap(int* &l,int* &r){
	int* tmp = l;
	l = r;
	r = tmp;
}

int main(){
	int a,b;cin>>a>>b;
	int *pa = &a;
	int *pb = &b;
	cout << pa << " " << pb << endl;
	swap(pa,pb);
	cout << pa << " " << pb << endl;
	return 0;
}
```



> 6.23 参考本节介绍的几个`print`函数，根据理解编写你自己的版本。 依次调用每个函数使其输入下面定义的`i`和`j`:

```cpp
int i = 0, j[2] = { 0, 1 };
```

```cpp
#include<bits/stdc++.h>
using namespace std;

void print(const int *i){
	if(i)cout << *i << endl;
}

void print(const int arr[],size_t n){
	for(size_t i = 0; i != n; ++i){
		cout << arr[i] << " ";
	}
	cout<<endl;
}

void print(const int* begin,const int* end){
	while(begin!=end){
		cout << *begin++ << " ";
	}
	cout << endl;
}

void print(const int (&arr)[2],size_t n){
	for(size_t i = 0; i != n; ++i){
		cout << arr[i] << " ";
	}
	cout << endl;
}
int main(){
	int i = 0, j[2] = { 0, 1 };
	print(&i);	
	print(j,2);
	print(begin(j),end(j));
	print(j,2);
	return 0;
}
```



> 6.24 描述下面这个函数的行为。如果代码中存在问题，请指出并改正。

```cpp
void print(const int ia[10])
{
	for (size_t i = 0; i != 10; ++i)
		cout << ia[i] << endl;
}
```

函数等价下面两种形式

```cpp
void print(const int ia[]){}
void print(const int *){}
```

实参自动转换为指向数组首元素的指针，数组的大小对函数调用没影响

也就是说，如果传了大小不为10的数组，会出问题

我们可以传引用，把数组维度限定

```cpp
void print(const int (&ia)[10])
{
	for (size_t i = 0; i != 10; ++i)
		cout << ia[i] << endl;
}
```

### 6.2.5 main: 处理命令行选项

> 6.25 编写一个`main`函数，令其接受两个实参。把实参的内容连接成一个`string`对象并输出出来。

```cpp
#include<bits/stdc++.h>
using namespace std;

int main(int argc,char *argv[]){
	string s;
	s+=*argv[1];
	s+=*argv[2];
	cout<<s<<endl;
	return 0;
}
```

运行时传入参数`./a.out abc def`



> 6.26 编写一个程序，使其接受本节所示的选项；输出传递给main函数的实参内容。

```cpp
#include<bits/stdc++.h>
using namespace std;

int main(int argc,char *argv[]){
	string s;
	for(int i = 1; i < argc;++i){
		cout << argv[i] << endl;
	}
	return 0;
}
```

### 6.2.6 含有可变形参的函数

> 6.27 编写一个函数，它的参数是`initializer_list<int>`类型的对象，函数的功能是计算列表中所有元素的和。

```cpp
#include<bits/stdc++.h>
using namespace std;

int sum(initializer_list<int>l){
	int ret = 0;
	for(const auto &e:l)ret += e;
	return ret;
}

int main(){
	cout << sum({1,2,3,4,5,6}) << endl;
	cout << sum({1,2,3,4}) << endl;
	return 0;
}
```



> 6.28 在`error_msg`函数的第二个版本中包含`ErrCode`类型的参数，其中循环内的elem是什么类型？

`const string &`类型



> 6.29 在范围`for`循环中使用`initializer_list`对象时，应该将循环控制变量声明成引用类型吗？为什么？

常量引用类型。

`initializer_list`对象中的元素都是常量，我们无法修改`initializer_list`对象中的元素的值。

## 6.3 返回类型和 return 语句

### 6.3.2 有返回值函数

> 6.30 编译第200页的`str_subrange`函数，看看你的编译器是如何处理函数中的错误的。

```cpp
#include<bits/stdc++.h>
using namespace std;

bool str_subrange(const string &str1, const string &str2){
	if(str1.size() == str2.size())
		return str1 == str2;
	auto size = (str1.size() < str2.size())
				? str1.size():str2.size();
	for(decltype(size)i = 0; i != size; ++i){
		if(str1[i] != str2[i])
			return;
	}
}
int main(){
	return 0;
}
```

编译器

`clang version 13.0.0`

报错

```cpp
[
/tmp/CP Editor-UZZoIa/sol.cpp:11:4: error: non-void function 'str_subrange' should return a value [-Wreturn-type]
                        return;
                        ^
1 error generated.
]
```



> 6.31 什么情况下返回的引用无效？什么情况下返回常量的引用无效？

返回引用无效：引用的对象是局部变量，函数结束对象销毁了

返回常量引用无效：返回对象要被修改



> 6.32 下面的函数合法吗？如果合法，说明其功能；如果不合法，修改其中的错误并解释原因。

```cpp
int &get(int *array, int index) { return array[index]; }
int main()
{
    int ia[10];
    for (int i = 0; i != 10; ++i)
        get(ia, i) = i;
}
```

合法，函数返回引用为右值，可以被赋值



> 6.33 编写一个递归函数，输出vector对象的内容。

```cpp
#include<bits/stdc++.h>
using namespace std;

void print(vector<int>::iterator begin,vector<int>::const_iterator end){
	if(begin!=end){
		cout << *begin++ << endl;
		print(begin,end);
	}
}
int main(){
	vector<int>vec{1,2,3,4,5};
	print(vec.begin(),vec.end());
	return 0;
}
```



> 6.34 如果`factorial`函数的停止条件如下所示，将发生什么？

```cpp
if(val != 0)
```

如果`val < 0`，递归不会终止



> 6.35 在调用`factorial`函数时，为什么我们传入的值是`val-1`而非`val--`？

`val--`传入的时候实参没有变，递归不终止



### 6.3.3 返回数组的指针

> 6.36 编写一个函数声明，使其返回数组的引用并且该数组包含10个`string`对象。 不用使用尾置返回类型、`decltype`或者类型别名。

```cpp
string (&fun())[10];
```



> 6.37 为上一题的函数再写三个声明，一个使用类型别名，另一个使用尾置返回类型，最后一个使用`decltype`关键字。 你觉得哪种形式最好？为什么？

```cpp
typedef string arrT[10];
arrT (&fun());

auto fun() -> string(&)[10];

string str[10];
decltype(str) &fun();
```

尾置返回类型，简单明了



> 6.38 修改`arrPtr`函数，使其返回数组的引用。

```cpp
decltype(odd) &arrPtr(int i){
	return (i%2)?odd:even;
}
```



## 6.4 函数重载