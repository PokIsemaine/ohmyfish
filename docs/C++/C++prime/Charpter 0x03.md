<h1 align="center">📔 C++ Prime 0x03 练习题解</h1>

## 3.1节 练习

> 3.1 使用恰当的 using 声明重做 1.4.1节 和 2.6.2节 的练习

以2.6.2节的1.5.2+1.6的书店程序练习为例

原来的书店程序

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

使用 using 声明后的

```cpp
#include <iostream>
using std::cin;
using std::cout;
using std::string;
using std::cerr;
using std::endl;

struct Sales_data {
	string bookNo;
	unsigned units_sold = 0;
	double revenue = 0.0;
};

int main(){
	Sales_data total,item;
	double price;
    if (cin >> total.bookNo >> total.units_sold >> price){
		total.revenue = total.units_sold * price;
        while (cin >> item.bookNo >> item.units_sold >> price){
			item.revenue = item.units_sold * price;
            if (total.bookNo == item.bookNo) {
                total.units_sold += item.units_sold;
                total.revenue += item.revenue;
            }else {
                cout << total.bookNo << " "
						<< total.units_sold << " "
						<< total.revenue << endl;
                total = item;
            }
        }
		cout << total.bookNo << " "
			<< total.units_sold << " "
			<< total.revenue << endl;
    }else {
        cerr << "No data?!" << endl;
        return -1;
    }

	return 0;
}
```



## 3.2节 练习

### 3.2.2节 练习

> 3.2 编写一段程序，从标准输入中一次读入一整行，然后修改该程序使其一次读入一个词

读一行

```cpp
#include <iostream>
#include <string>

using std::string;
using std::cin;
using std::cout;
using std::endl;

int main(){
	string line;
	while(getline(cin,line)){
		cout << line << endl;
	}
	return 0;
}
```

读一个词

```cpp
#include <iostream>
#include <string>

using std::string;
using std::cin;
using std::cout;
using std::endl;

int main(){
	string word;
	while(cin>>word){
		cout << word << endl;
	}
	return 0;
}
```



> 3.3 请说明 string 类的输入运算符和 getline 函数分别是如何处理空白字符的

cin 读入 string 对象，string 对象会自动忽略开头的空白（空格符、换行符、制表符等）并从第一个真正的字符开始读起，知道遇到下一处空白

getline会把换行符读入，但不存进 string。因为不包含换行符，所以我们有时候要手动加上换行操作符 endl 结束当前行并刷新



> 3.4 编写一段程序读入两个字符串，比较其是否相等并输出结果。如果不相等，输出较大的那个字符串。改写上述程序，比较输入的两个字符串是否等长，如果不等长，输出长度较大的

输出较大的字符串

```cpp
#include <iostream>
#include <string>

using std::string;
using std::cin;
using std::cout;
using std::endl;

int main(){
	string a,b;
	cin >> a >> b;
	if(a > b)cout << a << endl;
	else if(a < b)cout << b <<endl;
	else cout << "a = b" << endl;
	return 0;
}
```

输出长度较大的

```cpp
#include <iostream>
#include <string>

using std::string;
using std::cin;
using std::cout;
using std::endl;

int main(){
	string a,b;
	cin >> a >> b;
	if(a.size() > b.size())cout << a << endl;
	else if(a.size() < b.size())cout << b <<endl;
	else cout << "a.size() = b.size()" << endl;
	return 0;
}
```



> 3.5 编写一段程序从标准输入中读入多个字符串并将它们连接在一起，输出连接成的大字符串。修改上述程序，用空格把输入的多个字符串分隔开来

没有空格分隔

```cpp
#include <iostream>
#include <string>

using std::string;
using std::cin;
using std::cout;
using std::endl;

int main(){
	string input,ans;
	while(cin>>input){
		ans += input;
	}
	cout << ans << endl;
	
	return 0;
}
```

空格分隔

```cpp
#include <iostream>
#include <string>

using std::string;
using std::cin;
using std::cout;
using std::endl;

int main(){
	string input,ans;
	while(cin>>input){
		ans += input + " ";
	}
	cout << ans << endl;
	
	return 0;
}
```

### 3.2.3节 练习

> 3.6 编写一段程序，使用范围 for 语句将字符串内的所有字符用X代替

错误的

```cpp
#include <iostream>
#include <string>

using std::string;
using std::cin;
using std::cout;
using std::endl;

int main(){
	string s;
	cin >> s;
	for(auto &c : s){
		c = "X"; //报错 invalid conversion from ‘const char*’ to ‘char’
	}
	cout << s << endl;
	
	return 0;
}
```

c 是char &，"X"是 const char*，不能直接转过去的

区分 "X"和 'X'

正确的写法

```cpp
#include <iostream>
#include <string>

using std::string;
using std::cin;
using std::cout;
using std::endl;

int main(){
	string s;
	cin >> s;
	for(auto &c : s){
		c = 'X';
	}
	cout << s << endl;
	
	return 0;
}
```



> 3.7 就上一题完成的程序而言，如果将循环控制变量的类型设为 char 将发生什么？先估计一下结果，然后实际编程验证

```cpp
#include <iostream>
#include <string>

using std::string;
using std::cin;
using std::cout;
using std::endl;

int main(){
	string s;
	cin >> s;
	for(char c : s){
		c = 'X';
	}
	cout << s << endl;
	
	return 0;
}
```

没加引用的话不会改变，s 和输入的时候一样

> 3.8 分别用 while 循环 和 传统的 for 循环重写第一题的1程序，你觉得那种形式更好？为什么？

for

```cpp
#include <iostream>
#include <string>

using std::string;
using std::cin;
using std::cout;
using std::endl;

int main(){
	string s;
	cin >> s;
	for(int i=0;s[i]; ++i){
		s[i] = 'X';
	}
	cout << s << endl;
	
	return 0;
}
```

while

```cpp
#include <iostream>
#include <string>

using std::string;
using std::cin;
using std::cout;
using std::endl;

int main(){
	string s;
	cin >> s;
	int i=0;
	while(s[i]){
		s[i++] = 'X';
	}
	cout << s << endl;
	
	return 0;
}
```

范围for更好一些，对于遍历每一个都要遍历的情况很方便



> 3.9 下面的程序有何作用？它合法吗？如果不合法，为什么？

```cpp
string s;
cout << s[0] << endl;
```

能编译运行，但是不合法，下标访问空字符串是非法行为

> 3.10  编写一段程序，读入一个包含标点符号的字符串，将标点符号去除后输出字符串剩余的部分

```cpp
#include <iostream>
#include <string>

using std::string;
using std::cin;
using std::cout;
using std::endl;

int main(){
	string s = "abc,wdjwio.dniap!";
	string ret;
	for(auto c:s){
		if(!ispunct(c)){
			ret += c;
		}
	}
	cout << ret;
	return 0;
}
```



> 3.11 下面的范围 for 语句合法吗？如果合法，c的类型是什么？

```cpp
const string s = "Keep out!";//底层const
for (auto &c : s){/*...*/}
```

合法，c是 `const char &`类型



## 3.3节 练习

### 3.3.1节 练习

> 3.12 下列 vector 对象的定义有不正确的吗？如果有，请指出来。对于不正确的，描述其执行结果；对于不正确的，请说明其错误的原因。

```cpp
(a) vector<vector<int>>vec;//合法，C++ 11
(b) vector<string> svec = ivec;//不合法，类型不同
(c) vector<string> svec(10,"null");//svec有10个元素，每个值为"null"
```



> 3.13 下列的 vector 对象各包含多少个元素？这些元素的值分别是多少

```cpp
(a) vector<int> v2;//空
(b) vector<int> v2(10);//10个元素，每个元素值为0
(c) vector<int> v3(10,42);//10个元素，每个元素值为42
(d) vector<int> v4{10};//1个元素，值为10
(e) vector<int> v5{10,42};//2个元素，分别为10、42
(f) vector<string> v6{10};//10个元素，值为""
(g) vector<string> v7{10,"hi"};//10个元素，值为"hi"
```



### 3.3.2节 练习

> 3.14 编写一段程序，用 cin 读入一组整数并把它存入一个 vector 对象

```cpp
#include <iostream>
#include <string>
#include <vector>

using std::vector;
using std::string;
using std::cin;
using std::cout;
using std::endl;

int main(){
	vector<int> ivec;
	int x;
	while(cin >> x){
		ivec.push_back(x);
	}
	for(auto v:ivec){
		cout << v << " ";
	}

	return 0;
}
```



> 3.15 改写上题的程序，不过这次读入字符串

```cpp
#include <iostream>
#include <string>
#include <vector>

using std::vector;
using std::string;
using std::cin;
using std::cout;
using std::endl;

int main(){
	vector<string> svec;
	string x;
	while(cin >> x){
		svec.push_back(x);
	}
	for(auto v:svec){
		cout << v << " ";
	}

	return 0;
}
```

### 3.3.3节 练习

> 3.16 编写一段程序，把 3.13 中 vector 对象的容量和具体内容输出来。检验之前的回答是否正确，如果部队，重头学3.3.1节 知道弄明白

```cpp
#include <iostream>
#include <string>
#include <vector>

using std::cin;
using std::cout;
using std::endl;
using std::string;
using std::vector;

int main() {
	vector<int> v1;              //空
	vector<int> v2(10);          // 10个元素，每个元素值为0
	vector<int> v3(10, 42);      // 10个元素，每个元素值为42
	vector<int> v4{10};          // 1个元素，值为10
	vector<int> v5{10, 42};      // 2个元素，分别为10、42
	vector<string> v6{10};       // 10个元素，值为""
	vector<string> v7{10, "hi"}; // 10个元素，值为"hi"

	for (auto v : v1) {
		cout << v << " ";
	}
	cout << endl;
	for (auto v : v2) {
		cout << v << " ";
	}
	cout << endl;
	for (auto v : v3) {
		cout << v << " ";
	}
	cout << endl;
	for (auto v : v4) {
		cout << v << " ";
	}
	cout << endl;
	for (auto v : v5) {
		cout << v << " ";
	}
	cout << endl;
	for (auto v : v6) {
		cout << v << " ";
	}
	cout << endl;
		for (auto v : v7) {
		cout << v << " ";
	}
	cout << endl;

  return 0;
}
```



> 3.17 从 cin 读入一组词并把它们存入一个 vector 对象，然后设法把所有次都改写为大写形式。输出改变后的结果，每个词占一行

```cpp
#include <iostream>
#include <string>
#include <vector>

using std::cin;
using std::cout;
using std::endl;
using std::string;
using std::vector;

int main() {
	string word;
	vector<string>svec;
	while(cin >> word){
		svec.push_back(word);
	}
	for(auto &v:svec){
		for(auto &c:v){
			c = toupper(c);
		}
	}
	for(auto v:svec){
		cout << v << endl;
	}

  return 0;
}
```



> 3.18 下面程序合法吗，如果不合法，你准备如何修改

```cpp
vector<int> ivec;
ivec[0] = 42;
```

不合法，下标不能访问不存在的元素

```cpp
vector<int> ivec;
ivec.push_back(42);
```



> 3.19 如果想定义一个含有10个元素的 vector 对象，所有元素都是 42,请列去除三个不同的实现方法，哪个最好，为什么

```cpp
#include <iostream>
#include <string>
#include <vector>

using std::cin;
using std::cout;
using std::endl;
using std::string;
using std::vector;

int main() {
	vector<int>vec1(10,42);
	vector<int>vec2{42,42,42,42,42,42,42,42,42,42};
	vector<int>vec3;
	for(int i = 0; i < 10; ++i){
		vec3.push_back(42);
	}
	for(auto v:vec1){
		cout << v << " ";
	}
	cout << endl;
	for(auto v:vec2){
		cout << v << " ";
	}
	cout << endl;
	for(auto v:vec3){
		cout << v << " ";
	}
	cout << endl;
  return 0;
}
```

第一种最好，简洁明了



> 3.20 读入一组整数并把它们存入一个 vector 对象，将每对相邻整数和输出出来。改写程序，要求输出第一个和最后一个元素的和，第二个元素和倒数第二个元素的和，以此类推

```c++
#include <iostream>
#include <string>
#include <vector>

using std::cin;
using std::cout;
using std::endl;
using std::string;
using std::vector;

int main() {
	vector<int>vec;
	int x;
	while(cin >> x)vec.push_back(x);
	for(vector<int>::size_type i = 0; i < vec.size()-1; ++i){
		cout << vec[i] + vec[i+1] << " ";
	}
	return 0;
}
```



```cpp
#include <iostream>
#include <string>
#include <vector>

using std::cin;
using std::cout;
using std::endl;
using std::string;
using std::vector;

int main() {
	vector<int>vec;
	int x;
	while(cin >> x)vec.push_back(x);
	for(vector<int>::size_type i = 0; i < vec.size(); ++i){
		if(2*(i+1)>vec.size())break;
		cout << vec[i] + vec[vec.size()-1-i] << " ";
	}
	return 0;
}
```



## 3.4节 练习

### 3.4.1节 练习

> 3.21 请使用迭代器重做 3.3.3 节的第一个练习

```cpp
#include <iostream>
#include <string>
#include <vector>

using std::cin;
using std::cout;
using std::endl;
using std::string;
using std::vector;

int main() {
	vector<int> v1;              //空
	vector<int> v2(10);          // 10个元素，每个元素值为0
	vector<int> v3(10, 42);      // 10个元素，每个元素值为42
	vector<int> v4{10};          // 1个元素，值为10
	vector<int> v5{10, 42};      // 2个元素，分别为10、42
	vector<string> v6{10};       // 10个元素，值为""
	vector<string> v7{10, "hi"}; // 10个元素，值为"hi"

	for(auto it = v1.begin(); it != v1.end(); ++it){
		cout << *it << " ";
	}
	cout << endl;
		for(auto it = v2.begin(); it != v2.end(); ++it){
		cout << *it << " ";
	}
	cout << endl;
	for(auto it = v3.begin(); it != v3.end(); ++it){
		cout << *it << " ";
	}
	cout << endl;
	for(auto it = v4.begin(); it != v4.end(); ++it){
		cout << *it << " ";
	}
	cout << endl;
	for(auto it = v5.begin(); it != v5.end(); ++it){
		cout << *it << " ";
	}
	cout << endl;
	for(auto it = v6.begin(); it != v6.end(); ++it){
		cout << *it << " ";
	}
	cout << endl;
	for(auto it = v7.begin(); it != v7.end(); ++it){
		cout << *it << " ";
	}
	cout << endl;
  return 0;
}
```



> 3.22 修改之前那个输出 text 第一段的程序，首先把 text 的 第一段全部改成大写形式，然后再输出它

```cpp
#include<bits/stdc++.h>
using namespace std;

int main(){
	vector<string>text;
	string sentence;
	
	while(getline(cin,sentence)){
		text.push_back(sentence);
	}
	//只改第一段的
	for(auto it = (*text.begin()).begin(); it != (*text.begin()).end(); ++it){
		*it=toupper(*it);
	}
	
	
	for(auto it = text.cbegin(); it != text.cend(); ++ it){
		cout << (*it) << endl;
	}
	
	return 0;
}
```



> 3.23 编写一段程序，创建一个含有 10 个整数的 vector 对象，然后使用迭代器将所有元素的值变成原来的两倍。输出 vector 对象的内容，检验程序是否正确

​                              

```cpp
#include<bits/stdc++.h>
using namespace std;

int main(){
	
	vector<int>vec;
	for(int i = 0; i < 10; ++i)vec.push_back(i); 
	
	for(auto it = vec.begin(); it != vec.end(); ++it){
		*it = 2*(*it);
	} 
	
	for(auto it = vec.cbegin(); it != vec.cend(); ++it){
		cout << *it << " ";
	} 
	
	return 0;
}
```

​        

### 3.4.2节 练习

> 3.24 请使用迭代器重做 3.3.3 节最后一个练习

```cpp
#include <iostream>
#include <string>
#include <vector>

using std::cin;
using std::cout;
using std::endl;
using std::string;
using std::vector;

int main() {
	vector<int>vec;
	int x;
	while(cin >> x)vec.push_back(x);
	
	for(auto it = vec.begin(); it != vec.begin() + ( vec.size()+1)/2; ++it){
		int a = (*it);
		int b = *(vec.end()-(it-vec.begin())-1);
		cout << a+b << " ";
	}
	return 0;
}
```



> 3.25 3.3.3 节 划分分数段的程序是使用下标运算符实现的，请利用迭代器改写而该程序并实现完全相同的功能

```cpp
#include <iostream>
#include <string>
#include <vector>

using std::cin;
using std::cout;
using std::endl;
using std::string;
using std::vector;

int main() {
	
	vector<unsigned> scores(11,0);
	unsigned grade;
	
	while(cin >> grade){
		if(grade <= 100){
			*(scores.begin()+grade/10)+=1;
		}
	}
	
	for(auto v:scores){
		cout << v << endl;
	}
	return 0;
}
```



> 3.26 在 100 页的二分搜索程序中，为什么用的是 mid = beg + (end-beg) / 2，而非 mid = (beg + end)/2;

两个迭代器不能相加，要先做差得到 difference_type 的带符号整数,然后相对于 begin 做偏移



## 3.5节 练习

### 3.5.1节 练习

> 3.27 假设 txt_size 是一个无参数的函数，它的返回值是 int.请回答下列哪个定义是非法的？为什么？

```cpp
unsigned buf_size = 1024;
(a)int ia[buf_size];//非法，维度要为常量表达式
(b)int ia[4*7-14];//合法
(c)int ia[text_size()];//非法，维度运行的时候才知道，除非text_size 是 constexpr
(d)char st[11] = "fundamental";//非法，大小应该为12，空字符
```



> 3.28 下列数组的元素的值是什么

```cpp
string sa[10];
int ia[10];
int main() {
	string sa2[10];//10个空字符串
	int ia2[10];//未定义的值
}
```



> 3.29 相比于 vector 数组有哪些缺点，请列举一些

* 大小固定了
* 增删元素不方便
* 不能拷贝赋值



### 3.5.2节 练习

> 3.30 指出下面代码中的索引错误

```cpp
constexpr size_t array_size = 10;
int ia[array_size];
for(size_t ix = 1; ix <= array_size; ++ix)
	ia[ix] = ix;
```

下标应该 >=0 并且 < 数组大小



> 3.31 编写一段程序，定义一个含有 10 个 int的数组，令每个元素的值就是其下标值

```cpp
#include <iostream>
using std::cout;

int main() {
	constexpr size_t array_size = 10;
	int ia[array_size];
	for(size_t ix = 0; ix < array_size; ++ix)
		ia[ix] = ix;
	for(auto i:ia)cout << i << " ";
	return 0;
}
```



> 3.32 将上一题刚刚创建的数组拷贝给另一个数组。利用vector 重写程序，实现类似功能

```cpp
#include <iostream>
using std::cout;

int main() {
	constexpr size_t array_size = 10;
	int ia[array_size],ib[array_size];
	for(size_t ix = 0; ix < array_size; ++ix)
		ia[ix] = ix;
	for(size_t ix = 0; ix < array_size; ++ix)
		ib[ix]=ia[ix];
	for(auto i:ib)cout << i << " ";
	return 0;
}
```



vector 实现

```cpp
#include <iostream>
#include <vector>
using std::cout;
using std::vector;

int main() {
	vector<int>vec1,vec2;
	for(int i=0;i<10;i++)vec1.push_back(i);
	vec2 = vec1;
	for(auto v:vec2)cout << v << " ";
	return 0;
}
```



> 3.33 对于 104页 的程序来说，如果不初始化 scores 将发生什么

数组元素值未定义



### 3.5.3节 练习

> 3.34 假定p1 和 p2 都指向同一个数组中的元素，则下面程序的功能是什么？什么情况下该程序是非法的？

```cpp
p1 += p2 - p1;
//等价 p1 = p1 + p2 -p1 =p2;
```

就是 p1 指针移动的 p2 的位置，只要 p1 p2 没问题就合法



> 3.35 编写一段程序，利用指针数组的元素置为 0

```cpp
#include <iostream>
using std::cout;

int main() {
	constexpr size_t buf_size=10;
	int arr[buf_size]={1,2,3,4,5,6,7,8,9,10};
	for(auto ptr = arr; ptr != arr + buf_size; ++ptr){
		*ptr=0;
	}
	for(auto i:arr)cout << i << " ";
	return 0;
}
```



> 3.36 编写一段程序，比较两个数组是否相等。再写一段程序，比较两个 vector 对象是否相等

```cpp
#include <iostream>
#include <iterator>
using std::cout;
using std::begin;
using std::end;

int main() {
	int arr1[5]={1,2,3,4,5};
	int arr2[5]={1,2,4,3,5};
	auto beg1=begin(arr1),beg2=begin(arr2),end1=end(arr1),end2=end(arr2);
	if(end1-beg1==end2-beg2){
		for(auto i = beg1,j = beg2; i != end1 && j != end2; ++i,++j){
			if(*i != *j){
				cout << "No";
				return 0;
			}
		}
		cout << "Yes";
	}else{
		cout << "No";
		return 0;
	}
	return 0;
}
```

vector

```cpp
#include <iostream>
#include <vector>
using std::cout;
using std::endl;
using std::vector;

int main() {
	vector<int>vec1={1,2,4,4,5};
	vector<int>vec2={1,2,3,4,5};
	cout << (vec1==vec2?"Yes":"No") <<endl;
	return 0;
}
```

### 3.5.4节 练习

> 3.37 下面的程序是何含义，程序的输出结果是什么

```cpp
const char ca[] = { 'h', 'e', 'l', 'l', 'o' };
const char *cp = ca;
while (*cp) {
    cout << *cp << endl;
    ++cp;
}
```

原本用g++ 编译，正常输出了 hello

但这实际上是未定义行为，因为没有给末尾空字符'\0' 我们用clang++ 编译 并采用一下参数，可以检测

主要是`-fsanitize=undefined,address`可以检测未定义行为和内存

```cpp
clang++ -Wall -std=c++14 -O2  -fsanitize=undefined,address
```

> -fsanitize=address检查是否使用了错误的指针(不指向有效内存)，而-fsanitize=undefined启用了一组轻量级UB检查(整数溢出，错误移位，指针未对齐等)。



> 3.38 在本节我们提到，将连个指针相加不但是非法的，而且也没什么意义。请问为什么两个指针相加没有意义

指针的值是指向对象的地址，相减可以表示距离，+-整数可以表示移动，相加有啥意义？



> 3.39 编写一个程序，比较两个 string 对象。再编写一段程序，比较两个C风格的字符串的内容

```cpp
#include <iostream>
#include <string>

using std::cin;
using std::cout;
using std::endl;
using std::string;

int main(){
	string a,b;
	cin >> a >> b;
	if(a==b)cout << "a=b" << endl;
	else cout << (a>b?"a>b":"a<b") <<endl;
	return 0;
}
```

C风格字符串

```cpp
#include <iostream>
#include <string>
#include <cstring>

using std::cin;
using std::cout;
using std::endl;
using std::string;

int main(){
	char a[]="abcd";
	char b[]="abcd";
	if(strcmp(a,b)==0)cout << "a==b" << endl;
	else cout << (strcmp(a,b)<0?"a<b":"a>b") << endl;
	return 0;
}
```



> 3.40 编写一段程序，定义两个字符数组并用字符串字面值初始化它们；接着再定义一个字符数组存放在前两个数组连接后的结果。使用strcpy 和 strcat 把前两个数组的内容拷贝到第三个数组中

```cpp
#include <iostream>
#include <string>
#include <cstring>

using std::cin;
using std::cout;
using std::endl;
using std::string;

int main(){
	char a[]="abcd";
	char b[]="defg";
	char c[11];//size>=11
	strcpy(c,a);
	strcat(c,b);

	cout << c << endl;
	return 0;
}
```



### 3.5.5节 练习

> 3.41 编写一段程序，用整型数组初始化一个 vector 对象

```cpp
#include <iostream>
#include <vector>

using std::cout;
using std::vector;
using std::begin;
using std::end;

int main(){
	int arr[]={0,1,2,3,4,5};
	vector<int>vec(begin(arr),end(arr));
	for(auto v:vec)cout << v <<" ";
	return 0;
}
```



> 3.42 编写一段程序，将含有整数元素的 vector 对象拷贝给一个整型数组

```cpp
#include <iostream>
#include <vector>

using std::cout;
using std::vector;

int main(){
	vector<int>vec{0,1,2,3,4};
	int arr[5];
	for(int i = 0; i<5; ++i)
		arr[i]=vec[i];
	for(auto i:arr)cout << i << " ";
	return 0;
}
```

​                                                                                                                                                                                                  

### 3.6节 练习

> 3.43 编写3个不同版本的程序，令其均能输出 ia 的元素。版本 1 使用 范围for语句，版本2、3都使用普通for语句。其中版本 2 使用 下标运算符，版本 3 使用 指针。此外三个版本的程序都要求直接写出数据类型，不能使用类型别名、 auto 关键字或 decltype 关键字

```cpp
#include <iostream>
using std::cout;
using std::endl;
int main(){
	int ia[3][4] = {{0,1,2,3},{4,5,6,7},{8,9,10,11}};
	//版本1
	for(int (&row)[4]:ia){
		for(int (&col):row){
			cout << col << " ";
		}
		cout << endl;
	}
	//版本2
	for(int row = 0; row < 3; ++row){
		for(int col = 0; col < 4; ++col){
			cout << ia[row][col] << " ";
		}
		cout << endl;
	}
	//版本3
	for(int (*row)[4] = ia; row != ia + 3; ++row){
		for(int *col = *row; col != *row + 4; ++ col){
			cout << *col << " ";
		}
		cout << endl;
	}
	return 0;
}
```



> 3.44 改写上一个练习中的程序，使用类型别名来代替循环变量的类型

```cpp
#include <iostream>
using std::cout;
using std::endl;
using int_array = int[4];
int main(){
	int ia[3][4] = {{0,1,2,3},{4,5,6,7},{8,9,10,11}};
	//版本1
	for(int (&row)[4]:ia){
		for(int (&col):row){
			cout << col << " ";
		}
		cout << endl;
	}
	//版本2
	for(int row = 0; row < 3; ++row){
		for(int col = 0; col < 4; ++col){
			cout << ia[row][col] << " ";
		}
		cout << endl;
	}
	//版本3 类型代替
	for(int_array *row = ia; row != ia + 3; ++row){
		for(int *col = *row; col != *row + 4; ++ col){
			cout << *col << " ";
		}
		cout << endl;
	}
	return 0;
}
```



> 3.45 再次改写程序，这次使用 auto 关键字

```cpp
#include <iostream>
using std::cout;
using std::endl;
int main(){
	int ia[3][4] = {{0,1,2,3},{4,5,6,7},{8,9,10,11}};
	//版本1
	for(auto &row:ia){
		for(auto &col:row){
			cout << col << " ";
		}
		cout << endl;
	}
	//版本2
	for(auto row = 0; row < 3; ++row){
		for(auto col = 0; col < 4; ++col){
			cout << ia[row][col] << " ";
		}
		cout << endl;
	}
	//版本3
	for(auto *row = ia; row != ia + 3; ++row){
		for(auto *col = *row; col != *row + 4; ++ col){
			cout << *col << " ";
		}
		cout << endl;
	}
	return 0;
}
```

