<h1 align="center">📔 C++ Prime 0x0A 练习题解</h1>

## 10.1 概述

> 10.1 头文件`algorithm`中定义了一个名为`count`的函数，它类似`find`， 接受一对迭代器和一个值作为参数。`count`返回给定值在序列中出现的次数。编写程序，读取`int`序列存入`vector`中，打印有多少个元素的值等于给定值。

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

int main(){
	std::vector<int>vec;

	int x;while(std::cin>>x)vec.push_back(x);

	std::cout << std::count(vec.cbegin(),vec.cend(),3);

	return 0;
}
```



> 10.2 重做上一题，但读取 `string` 序列存入 `list` 中。

```cpp
#include <iostream>
#include <algorithm>
#include <string>
#include <list>

int main(){
	std::list<std::string>ls;

	std::string x;
	while(std::cin>>x)ls.push_back(x);

	std::cout << std::count(ls.cbegin(),ls.cend(),std::string("abc"));

	return 0;
}
```



## 10.2 初识泛型算法

### 10.2.1 只读算法

> 10.3 用 `accumulate`求一个 `vector<int>` 中元素之和。

```cpp
#include <iostream>
#include <algorithm>
#include <string>
#include <list>
#include <vector>
#include <numeric>//accumulate

int main(){
	std::vector<int>vec={1,2,3,4,5};
	int sum = std::accumulate(vec.cbegin(),vec.cend(),0);	
	std:: cout << sum;
	return 0;
}
```



> 10.4 假定 `v` 是一个`vector<double>`，那么调用 `accumulate(v.cbegin(),v.cend(),0)` 有何错误（如果存在的话）？

```cpp
#include <iostream>
#include <algorithm>
#include <string>
#include <list>
#include <vector>
#include <numeric>//accumulate

int main(){
	std::vector<double>vec={1.5,2.0,3.0,4.0,5.0};
	//直接输出的话结果int类型
	std:: cout << std::accumulate(vec.cbegin(),vec.cend(),0)<<std::endl;
	//第三个参数决定类型	
	std:: cout << std::accumulate(vec.cbegin(),vec.cend(),0.0);	
	
	return 0;
}
```



> 10.5 在本节对名册（`roster`）调用`equal`的例子中，如果两个名册中保存的都是C风格字符串而不是`string`，会发生什么？

C风格字符串使用指向字符数组的指针表示，所以比较两个指针存的地址值，而不是内容



### 10.2.2 写容器元素的算法

> 10.6 编写程序，使用 `fill_n` 将一个序列中的 `int` 值都设置为0。

```cpp
#include <iostream>
#include <algorithm>
#include <string>
#include <list>
#include <vector>
#include <numeric>//accumulate

int main(){
	std::vector<int>vec={1,2,3,4,5};
	std::fill_n(vec.begin(),vec.size(),0);
	for(auto v:vec)std::cout << v << " ";
	
	return 0;
}
```



> 10.7 下面程序是否有错误？如果有，请改正：

```cpp
(a) vector<int> vec; list<int> lst; int i;
	vec.resize(list.size());//加这句
	while (cin >> i)
		lst.push_back(i);
	copy(lst.cbegin(), lst.cend(), vec.begin());
(b) vector<int> vec;
	vec.reserve(10);//改成vec.resize(10)
	fill_n(vec.begin(), 10, 0);
```

* 拷贝算法将输入范围中的元素拷贝到目的序列中，传递给`copy`的目的序列至少包含和输入序列一样多的元素
* 算法不检查写操作，向目的位置迭代器写入数据的算法假定目的位置足够大，能容纳要写入的元素



> 10.8 本节提到过，标准库算法不会改变它们所操作的容器的大小。为什么使用 `back_inserter`不会使这一断言失效？

`back_inserter` 是插入迭代器，在 `iterator.h` 头文件中，不是标准库的算法 ？？？

我觉得`back_inserter`本身确实没有改变所操作容器的大小，它是将赋值运算符变成调用`push_back()`，改变容器的是`push_back()`而不是`back_inserter`,这样解释比较合理



### 10.2.3 重排容器元素的算法

> 10.9  实现你自己的`elimDups`。分别在读取输入后、调用`unique`后以及调用`erase`后打印`vector`的内容。

```cpp
#include <iostream>
#include <algorithm>
#include <string>
#include <list>
#include <vector>
#include <numeric>//accumulate

void print(const std::vector<std::string> &v){
	for(auto i:v)std::cout << i << " ";
	std::cout << std::endl;
}

void elimDups(std::vector<std::string> &words){
	std::sort(words.begin(),words.end());
	auto iter = std::unique(words.begin(),words.end());
	print(words);
	words.erase(iter,words.end());
}


int main(){
	std::vector<std::string>vs;
	std::string s;
	while(std::cin>>s)vs.push_back(s);
	print(vs);
	elimDups(vs);
	print(vs);
	return 0;
}
```



> 10.10 你认为算法不改变容器大小的原因是什么？

* 迭代器令算法不依赖于容器，但算法依赖于元素类型的操作
* 泛型算法本身不会执行容器的操作，它们只会运行于迭代器之上，执行迭代器的操作
* 将算法和容器的成员函数区分开来



## 10.3 定制操作

### 10.3.1 向算法传递函数

> 10.11编写程序，使用 `stable_sort` 和 `isShorter` 将传递给你的 `elimDups` 版本的 `vector` 排序。打印 `vector`的内容，验证你的程序的正确性。

```cpp
#include <iostream>
#include <algorithm>
#include <string>
#include <list>
#include <vector>
#include <numeric>//accumulate

void print(const std::vector<std::string> &v){
	for(auto i:v)std::cout << i << " ";
	std::cout << std::endl;
}

bool isShorter(const std::string &s1,const std::string &s2){
	return s1.size() < s2.size();
}
void elimDups(std::vector<std::string> &words){
	std::stable_sort(words.begin(),words.end(),isShorter);
	words.erase(std::unique(words.begin(),words.end()),words.end());
}


int main(){
	std::vector<std::string>vs;
	std::string s;
	while(std::cin>>s)vs.push_back(s);
	print(vs);
	elimDups(vs);
	print(vs);
	return 0;
}
```



> 10.12 编写名为 `compareIsbn` 的函数，比较两个 `Sales_data` 对象的`isbn()` 成员。使用这个函数排序一个保存 `Sales_data` 对象的 `vector`。

```cpp
#include <iostream>
#include <algorithm>
#include <string>
#include <list>
#include <vector>
#include <numeric>//accumulate

class Sales_data;
std::istream &read(std::istream&,Sales_data&);

class Sales_data {

friend std::istream & read(std::istream &,Sales_data &);
friend std::ostream & print(std::ostream &,const Sales_data &);
friend Sales_data add(const Sales_data &,const Sales_data &);

private:
	std::string bookNo;
	unsigned units_sold = 0;
	double revenue = 0.0;
public:	
	Sales_data& combine(const Sales_data &rhs);
	std::string isbn()const{return bookNo;}
	double avg_price()const;
	
	Sales_data(const std::string &s,unsigned u,double p)
		:bookNo(s),units_sold(u),revenue(u*p){}
	Sales_data() : Sales_data("",0,0){}
	Sales_data(const std::string &s):Sales_data(s,0,0){}
	Sales_data(std::istream &is):Sales_data(){read(is,*this);}
};


double Sales_data::avg_price()const{
	return units_sold?(revenue/units_sold):0;
}

Sales_data& Sales_data::combine(const Sales_data &rhs){
	units_sold += rhs.units_sold;
	revenue += rhs.revenue;
	return *this;
}

std::istream & read(std::istream &is,Sales_data &item){
	double price = 0;
	is >> item.bookNo >> item.units_sold >> price;
	item.revenue = item.units_sold * price;
	return is;
}

std::ostream & print(std::ostream &os,const Sales_data &item){
	os << item.isbn() << " "<< item.units_sold << " "
		<< item.revenue << " " << item.avg_price();
	return os;
}

Sales_data add(const Sales_data &lhs,const Sales_data &rhs){
	Sales_data sum = lhs;
	sum.combine(rhs);
	return sum;
}

bool compareIsbn(const Sales_data &A,const Sales_data&B){
	return A.isbn() < B.isbn(); 
}

int main(){
	Sales_data a("aa"),b("bb"),c("cc"),d("aaa");
	std::vector<Sales_data>v{b,a,c,d};

	for(auto i:v){
		print(std::cout,i)<<std::endl;
	}
	std::sort(v.begin(),v.end(),compareIsbn);
	std::cout << std::endl;
	for(auto i:v){
		print(std::cout,i)<<std::endl;
	}
	return 0;
}
```



> 10.13 标准库定义了名为 `partition` 的算法，它接受一个谓词，对容器内容进行划分，使得谓词为`true` 的值会排在容器的前半部分，而使得谓词为 `false` 的值会排在后半部分。算法返回一个迭代器，指向最后一个使谓词为 `true` 的元素之后的位置。编写函数，接受一个 `string`，返回一个 `bool` 值，指出 `string` 是否有5个或更多字符。使用此函数划分 `words`。打印出长度大于等于5的元素。

```cpp
#include <iostream>
#include <algorithm>
#include <string>
#include <list>
#include <vector>
#include <numeric>

bool check(const std::string &s){
	return s.size()>5;
}
int main(){
	std::vector<std::string>v={"a","aa","bbbbda","dafwgw","cccccc"};
	auto stop = std::partition(v.begin(), v.end(), check);
	for(auto iter = v.begin(); iter != stop; ++iter){
		std::cout << *iter << std::endl;
	}
	return 0;
}

```



### 10.3.2 lambda 表达式

> 10.14 编写一个`lambda`，接受两个`int`，返回它们的和。

```cpp
auto f = [](int i,int j){return i+j;}
```



> 10.15 编写一个 `lambda` ，捕获它所在函数的 `int`，并接受一个 `int`参数。`lambda` 应该返回捕获的 `int` 和 `int` 参数的和。

```cpp
int x = 10;
auto f = [x](int i) { i + x; };
```



> 10.16 使用 `lambda` 编写你自己版本的 `biggies`。

```cpp
#include <iostream>
#include <algorithm>
#include <string>
#include <list>
#include <vector>
#include <numeric>

void print(const std::vector<std::string> &v){
	for(auto i:v)std::cout << i << " ";
	std::cout << std::endl;
}

void elimDups(std::vector<std::string> &words){
	std::sort(words.begin(),words.end());
	words.erase(std::unique(words.begin(),words.end()),words.end());
}

std::string make_plural(const int count,const std::string word,const std::string &suf){
	return count>1?(word+suf):word;
}
void biggies(std::vector<std::string> &words,std::vector<std::string>::size_type sz){
	elimDups(words);
	std::stable_sort(words.begin(),words.end(),
					[](const std::string &a,const std::string &b){return a.size()<b.size();});
	auto wc = find_if(words.begin(),words.end(),
					[sz](const std::string &a){return a.size()>=sz;});
	auto count = words.end() -wc;
	std::cout << count << " " << make_plural(count,"word","s") 
			<<" of length " << sz << " or longer" << std::endl;
}
int main(){
	std::vector<std::string>v={"123","123","abc","gnoawdnoi","ddddd","ddddd"};
	biggies(v,3);
	return 0;
}
```



> 10.17 重写10.3.1节练习10.12的程序，在对`sort`的调用中使用 `lambda` 来代替函数 `compareIsbn`。

```cpp
#include <iostream>
#include <algorithm>
#include <string>
#include <list>
#include <vector>
#include <numeric>//accumulate

class Sales_data;
std::istream &read(std::istream&,Sales_data&);

class Sales_data {

friend std::istream & read(std::istream &,Sales_data &);
friend std::ostream & print(std::ostream &,const Sales_data &);
friend Sales_data add(const Sales_data &,const Sales_data &);

private:
	std::string bookNo;
	unsigned units_sold = 0;
	double revenue = 0.0;
public:	
	Sales_data& combine(const Sales_data &rhs);
	std::string isbn()const{return bookNo;}
	double avg_price()const;
	
	Sales_data(const std::string &s,unsigned u,double p)
		:bookNo(s),units_sold(u),revenue(u*p){}
	Sales_data() : Sales_data("",0,0){}
	Sales_data(const std::string &s):Sales_data(s,0,0){}
	Sales_data(std::istream &is):Sales_data(){read(is,*this);}
};


double Sales_data::avg_price()const{
	return units_sold?(revenue/units_sold):0;
}

Sales_data& Sales_data::combine(const Sales_data &rhs){
	units_sold += rhs.units_sold;
	revenue += rhs.revenue;
	return *this;
}

std::istream & read(std::istream &is,Sales_data &item){
	double price = 0;
	is >> item.bookNo >> item.units_sold >> price;
	item.revenue = item.units_sold * price;
	return is;
}

std::ostream & print(std::ostream &os,const Sales_data &item){
	os << item.isbn() << " "<< item.units_sold << " "
		<< item.revenue << " " << item.avg_price();
	return os;
}

Sales_data add(const Sales_data &lhs,const Sales_data &rhs){
	Sales_data sum = lhs;
	sum.combine(rhs);
	return sum;
}

int main(){
	Sales_data a("aa"),b("bb"),c("cc"),d("aaa");
	std::vector<Sales_data>v{b,a,c,d};

	for(auto i:v){
		print(std::cout,i)<<std::endl;
	}
	std::sort(v.begin(),v.end(),
			[](const Sales_data &A,const Sales_data&B){return A.isbn() < B.isbn(); });
	std::cout << std::endl;
	for(auto i:v){
		print(std::cout,i)<<std::endl;
	}
	return 0;
}
```



> 10.18 重写 `biggies`，用 `partition` 代替 `find_if`。我们在10.3.1节练习10.13中介绍了 `partition` 算法。

```cpp
#include <iostream>
#include <algorithm>
#include <string>
#include <list>
#include <vector>
#include <numeric>

void elimDups(std::vector<std::string> &words){
	std::sort(words.begin(),words.end());
	words.erase(std::unique(words.begin(),words.end()),words.end());
}

std::string make_plural(const int count,const std::string word,const std::string &suf){
	return count>1?(word+suf):word;
}
void biggies(std::vector<std::string> &words,std::vector<std::string>::size_type sz){
	elimDups(words);

	auto wc = std::partition(words.begin(),words.end(),
							[sz](const std::string &a){return a.size() >= sz;});

	for(auto iter = words.begin(); iter != wc; ++iter){
		std::cout << *iter << std::endl;
	}
	auto count = wc - words.begin();
	std::cout << count << " " << make_plural(count,"word","s") 
			<<" of length " << sz << " or longer" << std::endl;
}
int main(){
	std::vector<std::string>v={"123","123","abc","gnoawdnoi","ddddd","ddddd"};
	biggies(v,5);
	return 0;
}
```



> 10.19 用 `stable_partition` 重写前一题的程序，与 `stable_sort` 类似，在划分后的序列中维持原有元素的顺序。

```cpp
#include <iostream>
#include <algorithm>
#include <string>
#include <list>
#include <vector>
#include <numeric>

void elimDups(std::vector<std::string> &words){
	std::sort(words.begin(),words.end());
	words.erase(std::unique(words.begin(),words.end()),words.end());
}

std::string make_plural(const int count,const std::string word,const std::string &suf){
	return count>1?(word+suf):word;
}
void biggies(std::vector<std::string> &words,std::vector<std::string>::size_type sz){
	elimDups(words);

	auto wc = std::stable_partition(words.begin(),words.end(),
							[sz](const std::string &a){return a.size() >= sz;});

	for(auto iter = words.begin(); iter != wc; ++iter){
		std::cout << *iter << std::endl;
	}
	auto count = wc - words.begin();
	std::cout << count << " " << make_plural(count,"word","s") 
			<<" of length " << sz << " or longer" << std::endl;
}
int main(){
	std::vector<std::string>v={"123","123","abc","gnoawdnoi","ddddd","ddddd"};
	biggies(v,5);
	return 0;
}
```



### 10.3.3 lambda 捕获和返回

> 10.20 标准库定义了一个名为 `count_if` 的算法。类似 `find_if`，此函数接受一对迭代器，表示一个输入范围，还接受一个谓词，会对输入范围中每个元素执行。`count_if`返回一个计数值，表示谓词有多少次为真。使用`count_if`重写我们程序中统计有多少单词长度超过6的部分。

```cpp
#include <iostream>
#include <algorithm>
#include <string>
#include <list>
#include <vector>
#include <numeric>

int main(){
	std::vector<std::string>v={"abc","defghi","jklabcd","helloword"};
	std::cout << count_if(v.begin(), v.end(),
		[](const std::string &a){return a.size()>6;});

	return 0;
}
```



> 10.21 编写一个 `lambda`，捕获一个局部 `int` 变量，并递减变量值，直至它变为0。一旦变量变为0，再调用`lambda`应该不再递减变量。`lambda`应该返回一个`bool`值，指出捕获的变量是否为0。

```cpp
#include <iostream>
#include <algorithm>
#include <string>
#include <list>
#include <vector>
#include <numeric>

int main(){
	int i = 10;
	auto f = [&i]()->bool {
		return (i==0?true:!(i--));
	};
	while(!f())std::cout << i << std::endl;
}
```



### 10.3.4 参数绑定

> 10.22 重写统计长度小于等于6的单词数量的程序，使用函数代替`lambda`。

```cpp
#include <iostream>
#include <algorithm>
#include <string>
#include <list>
#include <vector>
#include <numeric>

bool check(const std::string &a){
	return a.size() <= 6;
}
int main(){
	std::vector<std::string>v={"abc","defghi","jklabcd","helloword"};
	std::cout << count_if(v.begin(), v.end(),check);

	return 0;
}
```



> 10.23 `bind` 接受几个参数？

假设被绑定的函数接受 `n` 个参数，那么`bind` 接受`n + 1`个参数。



> 10.24 给定一个`string`，使用 `bind` 和 `check_size` 在一个 `int` 的`vector` 中查找第一个大于`string`长度的值。

```cpp
#include <cstddef>
#include <iostream>
#include <algorithm>
#include <string>
#include <list>
#include <vector>
#include <numeric>
#include <functional>

bool check_size(const std::string &a,std::size_t sz){
	return a.size() < sz;
}
int main(){
	std::vector<int> vec {0,1,2,3,4,5,6,7,8};
	std::string s("abcd");
	auto result = std::find_if(vec.cbegin(),vec.cend(),std::bind(check_size,s,std::placeholders::_1));
	if(result != vec.end())std::cout << *result << std::endl;
	else std::cout << "Not found" << std::endl;
	return 0;
}
```



> 10.25 在10.3.2节的练习中，编写了一个使用`partition` 的`biggies`版本。使用 `check_size` 和 `bind` 重写此函数。

```cpp
#include <iostream>
#include <algorithm>
#include <string>
#include <list>
#include <vector>
#include <numeric>
#include <functional>

void elimDups(std::vector<std::string> &words){
	std::sort(words.begin(),words.end());
	words.erase(std::unique(words.begin(),words.end()),words.end());
}

std::string make_plural(const int count,const std::string word,const std::string &suf){
	return count>1?(word+suf):word;
}

bool check_size(const std::string &a,std::vector<std::string>::size_type sz){
	return a.size() >= sz;
}

void biggies(std::vector<std::string> &words,std::vector<std::string>::size_type sz){
	elimDups(words);

	auto wc = std::stable_partition(words.begin(),words.end(),
									bind(check_size,std::placeholders::_1,sz));

	for(auto iter = words.begin(); iter != wc; ++iter){
		std::cout << *iter << std::endl;
	}
	auto count = wc - words.begin();
	std::cout << count << " " << make_plural(count,"word","s") 
			<<" of length " << sz << " or longer" << std::endl;
}
int main(){
	std::vector<std::string>v={"123","123","abc","gnoawdnoi","ddddd","ddddd"};
	biggies(v,5);
	return 0;
}
```



## 10.4  再探迭代器

### 10.4.1 插入迭代器

> 10.26解释三种插入迭代器的不同之处。

* `back_inserter`创建一个使用`push_back`的迭代器
* `front_inserter`创建一个使用`push_front`的迭代器器
* `inserter`创建一个使用`insert`的迭代器



> 10.27 除了 `unique` 之外，标准库还定义了名为 `unique_copy` 的函数，它接受第三个迭代器，表示拷贝不重复元素的目的位置。编写一个程序，使用 `unique_copy`将一个`vector`中不重复的元素拷贝到一个初始化为空的`list`中。

```cpp
#include <iostream>
#include <algorithm>
#include <iterator>
#include <string>
#include <list>
#include <vector>
#include <numeric>
#include <functional>

int main(){
	std::vector<int>vec={0,0,1,1,2,3,4,4};
	std::sort(vec.begin(),vec.end());
	
	std::list<int>ls;
	std::unique_copy(vec.begin(),vec.end(),std::back_inserter(ls));

	for(auto i:ls)std::cout << i << " ";
	return 0;
}
```



> 10.28 一个`vector` 中保存 1 到 9，将其拷贝到三个其他容器中。分别使用`inserter`、`back_inserter` 和 `front_inserter` 将元素添加到三个容器中。对每种 `inserter`，估计输出序列是怎样的，运行程序验证你的估计是否正确。

```cpp
#include <iostream>
#include <algorithm>
#include <iterator>
#include <string>
#include <list>
#include <vector>
#include <numeric>
#include <functional>

void print(const std::list<int> &ls){
	for(auto i:ls)std::cout << i << " ";
	std::cout << std::endl;
}
int main(){
	std::vector<int>vec={1,2,3,4,5,6,7,8,9};
	std::sort(vec.begin(),vec.end());
	
	std::list<int>ls1,ls2,ls3;
	std::copy(vec.cbegin(),vec.cend(),std::inserter(ls1,ls1.begin()));
	std::copy(vec.cbegin(),vec.cend(),std::front_inserter(ls2));
	std::copy(vec.cbegin(),vec.cend(),std::back_inserter(ls3));

	print(ls1);
	print(ls2);
	print(ls3);
	
	return 0;
}
```



### 10.4.2 iostream 迭代器

> 10.29 编写程序，使用流迭代器读取一个文本文件，存入一个`vector`中的`string`里。

```cpp
#include <iostream>
#include <algorithm>
#include <iterator>
#include <string>
#include <list>
#include <vector>
#include <numeric>
#include <functional>
#include <fstream>

int main(int argc,char **argv){
	std::vector<std::string>vs;
	std::ifstream ifs(argv[1]);
	std::istream_iterator<std::string>ifs_iter(ifs),eof;
	std::copy(ifs_iter,eof,std::back_inserter(vs));
	std::copy(vs.cbegin(),vs.cend(),std::ostream_iterator<std::string>(std::cout,"\n"));
	return 0;
}
```



> 10.30 使用流迭代器、`sort` 和 `copy` 从标准输入读取一个整数序列，将其排序，并将结果写到标准输出。

```cpp
#include <iostream>
#include <algorithm>
#include <iterator>
#include <string>
#include <list>
#include <vector>
#include <numeric>
#include <functional>
#include <fstream>

int main(int argc,char **argv){
	std::vector<std::string>v;
	std::istream_iterator<std::string>ifs_iter(std::cin),eof;
	
	std::copy(ifs_iter,eof,std::back_inserter(v));
	std::sort(v.begin(),v.end());

	std::copy(v.cbegin(),v.cend(),std::ostream_iterator<std::string>(std::cout," "));
	return 0;
}
```



> 10.31 修改前一题的程序，使其只打印不重复的元素。你的程序应该使用 `unique_copy`。

```cpp
#include <iostream>
#include <algorithm>
#include <iterator>
#include <string>
#include <list>
#include <vector>
#include <numeric>
#include <functional>
#include <fstream>

int main(int argc,char **argv){
	std::istream_iterator<int> in_iter(std::cin);
	std::istream_iterator<int> eof;
	std::ostream_iterator<int> out_iter(std::cout," ");

	std::vector<int>ivec;

	while(in_iter!=eof){
		ivec.push_back(*in_iter++);
	}
	std::sort(ivec.begin(),ivec.end());
	std::unique_copy(ivec.cbegin(),ivec.cend(),out_iter);


	return 0;
}
```



> 10.32 重写1.6节中的书店程序，使用一个`vector`保存交易记录，使用不同算法完成处理。使用 `sort` 和10.3.1节中的 `compareIsbn` 函数来排序交易记录，然后使用 `find` 和 `accumulate` 求和。

```cpp
#include <iostream>
#include <algorithm>
#include <string>
#include <list>
#include <vector>
#include <numeric>//accumulate
#include <functional>
#include <iterator>
#include "Sales_item.h"//看第一章1.5.1练习题解，有头文件的代码

int main(){
	std::istream_iterator<Sales_item>in_iter(std::cin),eof;
	std::vector<Sales_item>vec;

	while(in_iter != eof){
		vec.push_back(*in_iter++);
	}

	sort(vec.begin(),vec.end(),compareIsbn);
	for(auto beg = vec.cbegin(),end = beg; beg != vec.cend();beg = end){
		end = find_if(beg,vec.cend(),
					[beg](const Sales_item &item){return item.isbn() != beg->isbn();});
		std::cout << std::accumulate(beg,end,Sales_item(beg->isbn())) << std::endl;
	}
	return 0;
}
```



> 10.33 编写程序，接受三个参数：一个输入文件和两个输出文件的文件名。输入文件保存的应该是整数。使用 `istream_iterator` 读取输入文件。使用 `ostream_iterator` 将奇数写入第一个输入文件，每个值后面都跟一个空格。将偶数写入第二个输出文件，每个值都独占一行。

```cpp
#include <iostream>
#include <algorithm>
#include <istream>
#include <string>
#include <list>
#include <vector>
#include <numeric>//accumulate
#include <functional>
#include <iterator>
#include <fstream>

int main(int argc,char** argv){
	if(argc != 4)return -1;
	
	std::ifstream ifs(argv[1]);
	std::ofstream  ofs_odd(argv[2]),ofs_even(argv[3]);

	std::istream_iterator<int> in_iter(ifs),eof;
	std::ostream_iterator<int> out_odd(ofs_odd," "),out_even(ofs_even,"\n");
	
	std::vector<int>odd,even;

	while(in_iter != eof){
		if((*in_iter)%2==0){
			even.push_back(*in_iter);
		}else{
			odd.push_back(*in_iter);
		}
		++in_iter;
	}

	std::copy(odd.cbegin(),odd.cend(),out_odd);
	std::copy(even.cbegin(),even.cend(),out_even);

	return 0;
}
```



### 10.4.3 反向迭代器

> 10.34 使用 `reverse_iterator` 逆序打印一个`vector`。

```cpp
#include <iostream>
#include <algorithm>
#include <istream>
#include <string>
#include <list>
#include <vector>
#include <numeric>//accumulate
#include <functional>
#include <iterator>
#include <fstream>

int main(){
	std::vector<int>v={1,2,3,4,5,6};
	for(auto it = v.crbegin(); it != v.crend(); ++it){
		std::cout << *it << " ";
	}
	return 0;
}
```



> 10.35 使用普通迭代器逆序打印一个`vector`。

```cpp
#include <iostream>
#include <algorithm>
#include <istream>
#include <string>
#include <list>
#include <vector>
#include <numeric>//accumulate
#include <functional>
#include <iterator>
#include <fstream>

int main(){
	std::vector<int>v={1,2,3,4,5,6};
	for(auto it = v.cend()-1; it != v.cbegin()-1; --it){
		std::cout << *it << " ";
	}
	return 0;
}
```



> 10.36 使用 `find` 在一个 `int` 的`list` 中查找最后一个值为0的元素。

```cpp
#include <iostream>
#include <algorithm>
#include <istream>
#include <string>
#include <list>
#include <vector>
#include <numeric>//accumulate
#include <functional>
#include <iterator>
#include <fstream>

int main(){
	std::list<int>ls={0,1,2,3,4,5,0,7};
	auto it = std::find(ls.crbegin(),ls.crend(),0);
	std::cout << distance(it,ls.crend());
	return 0;
}
```



> 10.37 给定一个包含10 个元素的`vector`，将位置3到7之间的元素按逆序拷贝到一个`list`中。

```cpp
#include <iostream>
#include <algorithm>
#include <istream>
#include <string>
#include <list>
#include <vector>
#include <numeric>//accumulate
#include <functional>
#include <iterator>
#include <fstream>

int main(){
	std::list<int>ls;
	std::vector<int>vec={0,1,2,3,4,5,6,7,8,9};
	std::copy(vec.crbegin()+2,vec.crbegin()+7,std::back_inserter(ls));

	for(auto i:ls)std::cout << i << " ";
	return 0;
}
```



### 10.5.1 5类迭代器

> 10.38 列出5个迭代器类别，以及每类迭代器所支持的操作。

* 输入迭代器必须支持：`==`,`!=`,`++`,`*`,`->`
* 输出迭代器必须支持(`ostream_iterator`类型也是输出迭代器)：`++`,`*`
* 前向迭代器
	* `==`,`!=`,`++`,`*`,`->`
	* `replace`算法要求前向迭代器，`foward_list`上的迭代器也是前向迭代器
* 双向迭代器
	*   `==`,`!=`,`++`,`--`,`*`,`->`
	* 算法`reverser`要求双向迭代器，除了`foward_list`其他标准库都提供双向迭代器
* 随机访问迭代器
	*  `==`,`!=`,`<`,`<=`,`>`,`>=`,`++`,`--`,`+`,`+=`,`-`,`-=`,`*`,`->`,`iter[n]`==`*(iter[n])`
	* `sort`要求随机访问运算符，`array`、`deque`、`string`和`vector`迭代器都是随机访问迭代器，用于访问内置数组的指针也是



> 10.39 `list` 上的迭代器属于哪类？`vector`呢？

`list`上的迭代器是双向迭代器

`vector`是随机访问迭代器



> 10.40 你认为 `copy` 要求哪类迭代器？`reverse` 和 `unique` 呢？

`copy`要两个输入迭代器和一个输出迭代器

`reverser`要求双向迭代器

`unique`要求随机访问迭代器

 

### 10.5.3 算法命名规范

> 10.41 仅根据算法和参数的名字，描述下面每个标准库算法执行什么操作：

```cpp
replace(beg, end, old_val, new_val);//迭代器返回内的值用new_val替换
replace_if(beg, end, pred, new_val);//满足 pred 谓词条件才替换
replace_copy(beg, end, dest, old_val, new_val);//复制迭代器范围内的元素到目标迭代器位置，如果元素等于某个旧值，则用新值替换
replace_copy_if(beg, end, dest, pred, new_val);//复制迭代器范围内的元素到目标迭代器位置，满足谓词条件的元素用新值替换
```



## 10.6 特定容器的算法

> 10.42 使用 `list` 代替 `vector` 重新实现10.2.3节中的去除重复单词的程序。

```cpp
#include <iostream>
#include <algorithm>
#include <istream>
#include <string>
#include <list>
#include <vector>
#include <numeric>//accumulate
#include <functional>
#include <iterator>
#include <fstream>

void elimDups(std::list<std::string> &words){
	words.sort();
	words.unique();
}

int main(){
	std::list<std::string> l = { "aa", "aa", "aa", "aa", "aasss", "aa" };
	
	elimDups(l);

	for(const auto& e : l)std::cout << e << " ";

	return 0;
}
```

