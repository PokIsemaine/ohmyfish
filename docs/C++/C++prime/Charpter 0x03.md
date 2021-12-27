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

### 3.4.
