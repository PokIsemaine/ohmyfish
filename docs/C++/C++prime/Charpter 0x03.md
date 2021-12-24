<h1 align="center">📔 C++ Prime 0x03 练习题解</h1>

## 3.1节 练习

> 3.1 使用恰当的 using 声明重做 1.4.1节 和 2.6.2节 的练习

以2.6.2节的1.5.2+1.6的书店程序练习为例

原来的书店程序

```c++
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

```c++
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

```c++
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

```c++
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

```c++
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

```c++
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

```c++
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

```c++
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

```c++
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

```c++
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

```c++
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



> 3.9 下面的程序有何作用？它合法吗？如果不合法，为什么？

```c++
string s;
cout << s[0] << endl;
```



> 3.10  编写一段程序，读入一个包含标点符号的字符串，将标点符号去除后输出字符串剩余的部分



> 3.11 下面的范围 for 语句合法吗？如果合法，c的类型是什么？

```c++
const string = "Keep out!";
for (auto &c : s){/*...*/}
```

