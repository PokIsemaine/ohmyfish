<h1 align="center">📔 C++ Primer 0x0E 练习题解</h1>

## 14.1 基本概念

> 14.1 在什么情况下重载的运算符与内置运算符有所区别？在什么情况下重载的运算符又与内置运算符一样？

重载的时候就会有不一样的

* 重载的运算符是具有特殊名字的函数：它们的名字由关键字`operator`和其后要定义的运算符号共同组成，和其他函数一样有返回类型、参数列表以及函数体

* 当一个重载的运算符是成员函数时，`this`绑定到左侧运算对象，成员函数的显式参数数量比运算对象的数量少一个
* 对于一个运算符号来说，它或者是类的成员，或者至少包含一个类类型的参数，这意味着当运算符作用于内置类型的运算对象时，我们无法改变该运算符的含义
* 对于一个重载的运算符来说，其优先级和结合律与对应的内置运算符保持一致



> 14.2 为 `Sales_data` 编写重载的输入、输出、加法和复合赋值运算符。

```cpp
#ifndef SalesData_H
#define SalesData_H

#include <fstream>
#include <iostream>
#include <istream>
#include <ostream>
#include <string>

class SalesData{
public:
	friend std::istream& operator>>(std::istream&,SalesData&);
	friend std::ostream& operator<<(std::ostream&,const SalesData&);
	friend SalesData operator+(const SalesData&,const SalesData&);

public:
	SalesData(const std::string &s,unsigned n,double p)
		:bookNo(s),units_sold(n),revenue(n*p){}
	SalesData():SalesData("",0,0.0){}
	SalesData(const std::string &s):SalesData(s,0,0.0){}
	SalesData(std::ifstream& is){is >> *this;}
public:
	SalesData& operator+=(const SalesData&);
	std::string isbn()const{return bookNo;}
private:
	double avg_price()const{
		return units_sold ? revenue/units_sold : 0;
	}
private:
	std::string bookNo;
	unsigned units_sold = 0;
	double revenue = 0.0;
};

#endif
```



```cpp
#include "SalesData.hpp"

std::istream& operator>>(std::istream& is,SalesData& item){
	double price = 0.0;
	is >> item.bookNo >> item.units_sold >> price;
    if (is)item.revenue = price * item.units_sold;
    else item = SalesData();
    return is;
}
std::ostream& operator<<(std::ostream& os,const SalesData& item){
	os << item.isbn() << " " << item.units_sold << " " << item.revenue << " " << item.avg_price();
	return os;
}
SalesData operator+(const SalesData& lhs,const SalesData& rhs){
	auto sum = lhs;
	sum += rhs;
	return sum;
}

SalesData& SalesData::operator+=(const SalesData& rhs){
	units_sold += rhs.units_sold;
	revenue += rhs.revenue;
	return *this;
}
```



> 14.3 `string` 和 `vector` 都定义了重载的`==`以比较各自的对象，假设 `svec1` 和 `svec2` 是存放 `string` 的 `vector`，确定在下面的表达式中分别使用了哪个版本的`==`？

```cpp
(a) "cobble" == "stone"//都不是
(b) svec1[0] == svec2[0]//string
(c) svec1 == svec2//vector
(d) svec1[0] == "stone"//string
```



> 14.4 如何确定下列运算符是否应该是类的成员？

```cpp
(a) % 非成员
(b) %= 成员
(c) ++ 成员
(d) -> 必须成员
(e) << 非成员
(f) && 非成员
(g) == 非成员
(h) () 必须是成员
```



> 14.5 在7.5.1节中的练习7.40中，编写了下列类中某一个的框架，请问在这个类中应该定义重载的运算符吗？如果是，请写出来。

`Book`直接按`SalesData`的写，见练习`14.2`



## 14.2 输入和输出运算符

### 14.2.1 重载输出运算符 <<

> 14.6 为你的 `Sales_data` 类定义输出运算符。

见`14.2`



> 14.7 你在13.5节的练习中曾经编写了一个`String`类，为它定义一个输出运算符。

```cpp
#ifndef MyString_H
#define MyString_H

#include <memory>
#include <algorithm>
#include <iterator>
#include <iostream>
class MyString{
public:
	friend std::ostream& operator<<(std::ostream&,const MyString&);
public:
    MyString():MyString(""){}
    MyString(const char*);
    MyString(const MyString&);
    MyString& operator=(const MyString&);
    ~MyString();
public:
    size_t size()const{return ends-elements;}
    const char* begin()const{return elements;}
    const char* end()const{return ends;}

private:
    std::allocator<char> alloc;
    std::pair<char*,char*>alloc_n_copy(const char*,const char*);
    void free();
    void range_initializer(const char *first, const char *last);

private:
    char* elements;
    char* ends;
};


#endif
```



```cpp
#include "MyString.hpp"

std::pair<char*,char*>
MyString::alloc_n_copy(const char*b,const char*e){
    auto data = alloc.allocate(e-b);
    return {data,std::uninitialized_copy(b, e,data)};
}

void MyString::free(){
    if(elements){
        std::for_each(elements,ends,[this](char &c){alloc.destroy(&c);});
        alloc.deallocate(elements,ends-elements);
    }
}

void MyString::range_initializer(const char *first, const char *last){
    auto newstr = alloc_n_copy(first, last);
    elements = newstr.first;
    ends = newstr.second;
}
MyString::MyString(const char* s){
    char * sl = const_cast<char*>(s);
    while(*sl)
        ++sl;
    range_initializer(s,++sl);
}

MyString::~MyString(){free();}

MyString& MyString::operator=(const MyString &rhs){
    auto data = alloc_n_copy(rhs.begin(),rhs.end());
    free();
    elements = data.first;
    ends =  data.second;
    std::cout << "call MyString& MyString::operator=(const MyString &rhs)" << std::endl;
    return *this;
}

MyString::MyString(const MyString& ms){
    range_initializer(ms.begin(),ms.end());
    std::cout << "MyString::MyString(const MyString& ms)" << std::endl;
}

std::ostream& operator<<(std::ostream& os,const MyString& ms){
	char *c = const_cast<char*>(ms.begin());
	while(*c){
		os << *c++;
	}
	return os;
}
```



```cpp
#include "MyString.hpp"

int main(){
	MyString s("abcd");
	std::cout << s;
	return 0;
}
```



> 14.8 你在7.5.1节中的练习中曾经选择并编写了一个类，为它定义一个输出运算符。

选`Book`实现和`SalesData`一样，见`14.2`



### 14.2.2 重载输入运算符 >>

> 14.9 为你的 `Sales_data` 类定义输入运算符。

见`14.2`



> 14.10 对于 `Sales_data` 的输入运算符来说如果给定了下面的输入将发生什么情况？

```cpp
(a) 0-201-99999-9 10 24.95//正确
(b) 10 24.95 0-210-99999-9//第三个转不成double
```



> 14.11 下面的 `Sales_data` 输入运算符存在错误吗？如果有，请指出来。对于这个输入运算符如果仍然给定上个练习的输入将会发生什么情况？

```cpp
istream& operator>>(istream& in, Sales_data& s)
{
    //没有对输入进行检查
	double price;//没初始化，不过没啥影响
	in >> s.bookNo >> s.units_sold >> price;
	s.revence = s.units_sold * price;
	return in;
}
```



> 14.12 你在7.5.1节的练习中曾经选择并编写了一个类，为它定义一个输入运算符并确保该运算符可以处理输入错误。

选`Book`实现和`SalesData`一样，见`14.2`



## 14.3 算术和关系运算符

### 14.3.1 相等运算符

> 14.13 你认为 `Sales_data` 类还应该支持哪些其他算术运算符？如果有的话，请给出它们的定义。

见`14.2`



> 14.14 你觉得为什么调用 `operator+=` 来定义`operator+` 比其他方法更有效？

`operator+=`不用创建额外的临时变量

而且减少代码量，少写重复代码



> 14.15 你在7.5.1节的练习7.40中曾经选择并编写了一个类，你认为它应该含有其他算术运算符吗？如果是，请实现它们；如果不是，解释原因。

```cpp
#ifndef Book_H
#define Book_H

#include <iostream>
#include <istream>
#include <ostream>
#include <string>

class Book{
public:
	friend std::istream& operator>>(std::istream&,Book&);
	friend std::ostream& operator<<(std::ostream&,const Book&);
	friend Book operator+(const Book&,const Book&);
	friend bool operator==(const Book&,const Book&);
	friend bool operator!=(const Book&,const Book&);
	friend bool operator<(const Book&,const Book&);
	friend bool operator>(const Book&,const Book&);

public:
	Book () = default;
	Book (const std::string&b_no,const std::string& b_name,const std::string& author,double p)
		:BookNo(b_no),BookName(b_name),Author(author),price(p){}
	Book (std::istream& is){is >> *this;}

public:
	Book& operator+=(const Book&);
	std::string isbn()const{return BookNo;}

private:
	std::string BookNo;
	std::string BookName;
	std::string Author;
	double price = 0.0;
};

std::istream& operator>>(std::istream&,Book&);
std::ostream& operator<<(std::ostream&,const Book&);
Book operator+(const Book&,const Book&);
bool operator==(const Book&,const Book&);
bool operator!=(const Book&,const Book&);
bool operator<(const Book&,const Book&);
bool operator>(const Book&,const Book&);


#endif
```



```cpp
#include "Book.hpp"

Book& Book::operator+=(const Book& rhs){
	if(*this == rhs)price += rhs.price;
	return *this;
}
std::istream& operator>>(std::istream& is,Book& bk){
	if(is)is >> bk.BookNo >> bk.BookName >> bk.Author >> bk.price;
	else bk = Book();
	return is;
}
std::ostream& operator<<(std::ostream& os,const Book& bk){
	os << bk.isbn() << " " << bk.BookName << " " << bk.Author << " " << bk.price;
	return os;
}
Book operator+(const Book&lhs,const Book&rhs){
	auto sum = lhs;
	sum += rhs;
	return sum;
}
bool operator==(const Book&lhs,const Book&rhs){
	return lhs.isbn()==rhs.isbn();
}
bool operator!=(const Book&lhs,const Book&rhs){
	return !(lhs == rhs);
}

bool operator<(const Book&lhs ,const Book& rhs){
	return lhs.isbn() < rhs.isbn();
}
bool operator>(const Book&lhs,const Book& rhs){
	return rhs < lhs;
}
```



```cpp
#include "Book.hpp"

int main(){
	Book book1("1000","BookTest","zsl",12.0);
	Book book2("1001","BookRead","zsl",5.0);
	Book book3 = book1;
	Book book4;

	std::cin >> book4;
	std::cout << book4 << std::endl;
	if(book1==book2)std::cout << " book1 == book2" << std::endl;
	else std::cout << " book1 != book2" << std::endl;
	if(book1==book3)std::cout << " book1 == book3" << std::endl;
	else std::cout << " book1 != book3" << std::endl;
	std::cout << book3 << std::endl;
	std::cout << book1+book3 << std::endl;

	return 0;
}
```

### 14.3.1 相等运算符

> 14.16 为你的 `StrBlob` 类、`StrBlobPtr` 类、`StrVec` 类和 `String` 类分别定义相等运算符和不相等运算符。

`StrVec` 见`14.23`

> 14.17 你在7.5.1节中的练习7.40中曾经选择并编写了一个类，你认为它应该含有相等运算符吗？如果是，请实现它；如果不是，解释原因。

见`14.15`





### 14.3.2 关系运算符

> 14.18 为你的 `StrBlob` 类、`StrBlobPtr` 类、`StrVec` 类和 `String` 类分别定义关系运算符。

`StrVec` 见`14.23`

> 14.19 你在7.5.1节的练习7.40中曾经选择并编写了一个类，你认为它应该含有关系运算符吗？如果是，请实现它；如果不是，解释原因。

我觉得那个关系运算符可以不定义，这个有点主观。

`14.15`的代码里是按照`BookNo`进行比较的，实际上你也可以按`BookName`或者其他数据成员去比较。`<`的逻辑算是合理，但不唯一



## 14.4 赋值运算符

> 14.20 为你的 `Sales_data` 类定义加法和复合赋值运算符。

见`14.2`



> 14.21 编写 `Sales_data` 类的`+` 和`+=` 运算符，使得 `+` 执行实际的加法操作而 `+=` 调用`+`。相比14.3节和14.4节对这两个运算符的定义，本题的定义有何缺点？试讨论之。

用`+=`来实现`+`，可以避免一个临时对象，同时减少重复代码



> 14.22 定义赋值运算符的一个新版本，使得我们能把一个表示 `ISBN` 的 `string` 赋给一个 `Sales_data` 对象。

 头文件添加

```cpp
SalesData& operator=(const std::string &);
```

`cpp`文件添加

```cpp
SalesData& SalesData::operator=(const std::string & s){
	*this = SalesData(s);
	return *this;
}
```



> 14.23 为你的`StrVec` 类定义一个 `initializer_list` 赋值运算符。

```cpp
#ifndef StrVec_H
#define StrVec_H

#include <cstdlib>
#include <string>
#include <iostream>
#include <vector>
#include <memory>
#include <utility>
#include <initializer_list>

class StrVec{
public:
	friend bool operator==(const StrVec&, const StrVec&);
	friend bool operator!=(const StrVec&, const StrVec&);
	friend bool operator< (const StrVec&, const StrVec&);
	friend bool operator> (const StrVec&, const StrVec&);
	friend bool operator<=(const StrVec&, const StrVec&);
	friend bool operator>=(const StrVec&, const StrVec&);
public:
    StrVec():elements(nullptr),first_free(nullptr),cap(nullptr){}
    StrVec(const StrVec&);
	StrVec(StrVec&&)noexcept;

    ~StrVec();
    StrVec(std::initializer_list<std::string>);

public:
    StrVec& operator=(const StrVec&);
	StrVec& operator=(StrVec&&)noexcept;
	StrVec& operator=(std::initializer_list<std::string>);
public:
    void push_back(const std::string&);
    size_t size()const{return first_free-elements;}
    size_t capacity()const{return cap-elements;}
    std::string* begin()const{return elements;}
    std::string* end()const{return first_free;}

    void reserve(size_t new_cap);
    void resize(size_t count);
    void resize(size_t count,const std::string& s);
private:
    static std::allocator<std::string>alloc;
    void chk_n_alloc(){
        if (size() == capacity())reallocate();
    }
    std::pair<std::string*,std::string*>alloc_n_copy(const std::string*,const std::string*);
    void free();
    void reallocate();
    void alloc_n_move(size_t new_cap);
    void range_initialize(const std::string *first, const std::string *last);

private:
    std::string *elements;
    std::string *first_free;
    std::string *cap;
};

bool operator==(const StrVec&, const StrVec&);
bool operator!=(const StrVec&, const StrVec&);
bool operator< (const StrVec&, const StrVec&);
bool operator> (const StrVec&, const StrVec&);
bool operator<=(const StrVec&, const StrVec&);
bool operator>=(const StrVec&, const StrVec&);

#endif
```



```cpp
#include "StrVec.hpp"
#include <memory>
#include <string>

void StrVec::push_back(const std::string& s){
    chk_n_alloc();
    alloc.construct(first_free++,s);
}

std::pair<std::string*,std::string*>
StrVec::alloc_n_copy (const std::string*b,const std::string*e){
    auto data = alloc.allocate(e-b);
    return {data,std::uninitialized_copy(b, e,data)};
}

void StrVec::free(){
    if(elements){
        for(auto p = first_free; p != elements;)
            alloc.destroy(--p);
        alloc.deallocate(elements,cap-elements);
    }
}

StrVec::StrVec(const StrVec &s){
    auto newdata = alloc_n_copy(s.begin(), s.end());
    elements = newdata.first;
    first_free = cap = newdata.second;
}

StrVec::StrVec(StrVec&& s)noexcept
	:elements(s.elements),first_free(s.first_free),cap(s.cap){
	s.elements = s.first_free = s.cap = nullptr;
}

StrVec& StrVec::operator = (StrVec &&rhs) noexcept{
	if (this != &rhs){
		free();
		elements = rhs.elements;
		first_free = rhs.first_free;
		cap = rhs.cap;
		rhs.elements = rhs.first_free = rhs.cap = nullptr;
	}
	return *this;
}

StrVec::~StrVec(){free();}

StrVec& StrVec::operator=(const StrVec &rhs){
    auto data = alloc_n_copy(rhs.begin(),rhs.end());
    free();
    elements = data.first;
    first_free = cap = data.second;
    return *this;
}

void StrVec::range_initialize(const std::string *first, const std::string *last){
    auto newdata = alloc_n_copy(first, last);
    elements = newdata.first;
    first_free = cap = newdata.second;
}

StrVec::StrVec(std::initializer_list<std::string> il){
    range_initialize(il.begin(), il.end());
}

void StrVec::alloc_n_move (size_t new_cap){
    auto newdata = alloc.allocate(new_cap);
    auto dest = newdata;
    auto elem = elements;
    for(size_t i = 0 ;i != size();++i)
        alloc.construct(dest++,std::move(*elem++));
    free();
    elements = newdata;
    first_free = dest;
    cap = elements + new_cap;
}
void StrVec::reallocate (){
    auto newcapacity = size()?2*size():1;
    alloc_n_move(newcapacity);
}

void StrVec::reserve(size_t new_cap){
    if(new_cap <= capacity())return;
    alloc_n_move(new_cap);
}

void StrVec::resize(size_t count){
    resize(count,std::string());
}

void StrVec::resize(size_t count,const std::string& s){
    if(count > size()){
        if(count > capacity())reserve(count*2);
        for(size_t i = size(); i != count;++i){
            alloc.construct(first_free++,s);
        }
    }else if(count <size()){
        while(first_free != elements + count)
            alloc.destroy(--first_free);
    }
}

bool operator==(const StrVec &lhs, const StrVec &rhs){
	return (lhs.size() == rhs.size() && std::equal(lhs.begin(), lhs.end(), rhs.begin()));
}

bool operator!=(const StrVec &lhs, const StrVec &rhs){
	return !(lhs == rhs);
}

bool operator<(const StrVec &lhs, const StrVec &rhs){
	return std::lexicographical_compare(lhs.begin(), lhs.end(), rhs.begin(), rhs.end());
}

bool operator>(const StrVec &lhs, const StrVec &rhs){
	return rhs < lhs;
}

bool operator<=(const StrVec &lhs, const StrVec &rhs){
	return !(rhs < lhs);
}

bool operator>=(const StrVec &lhs, const StrVec &rhs){
	return !(lhs < rhs);
}

StrVec& StrVec::operator=(std::initializer_list<std::string> il){
	*this = StrVec(il);
	return *this;
}
```



> 14.24 你在7.5.1节的练习7.40中曾经选择并编写了一个类，你认为它应该含有拷贝赋值和移动赋值运算符吗？如果是，请实现它们。

```cpp
#ifndef Book_H
#define Book_H

#include <iostream>
#include <istream>
#include <ostream>
#include <string>

class Book{
public:
	friend std::istream& operator>>(std::istream&,Book&);
	friend std::ostream& operator<<(std::ostream&,const Book&);
	friend Book operator+(const Book&,const Book&);
	friend bool operator==(const Book&,const Book&);
	friend bool operator!=(const Book&,const Book&);
	friend bool operator<(const Book&,const Book&);
	friend bool operator>(const Book&,const Book&);

public:
	Book () = default;
	Book (const std::string&b_no,const std::string& b_name,const std::string& author,double p)
		:BookNo(b_no),BookName(b_name),Author(author),price(p){}
	Book (std::istream& is){is >> *this;}
	Book (const Book&);
	Book (Book&&)noexcept;

public:
	Book& operator+=(const Book&);
	Book& operator=(const Book&);
	Book& operator=(Book&&)noexcept;
	std::string isbn()const{return BookNo;}

private:
	std::string BookNo;
	std::string BookName;
	std::string Author;
	double price = 0.0;
};

std::istream& operator>>(std::istream&,Book&);
std::ostream& operator<<(std::ostream&,const Book&);
Book operator+(const Book&,const Book&);
bool operator==(const Book&,const Book&);
bool operator!=(const Book&,const Book&);
bool operator<(const Book&,const Book&);
bool operator>(const Book&,const Book&);


#endif
```



```cpp
#include "Book.hpp"

Book& Book::operator+=(const Book& rhs){
	if(*this == rhs)price += rhs.price;
	return *this;
}

std::istream& operator>>(std::istream& is,Book& bk){
	if(is)is >> bk.BookNo >> bk.BookName >> bk.Author >> bk.price;
	else bk = Book();
	return is;
}
std::ostream& operator<<(std::ostream& os,const Book& bk){
	os << bk.isbn() << " " << bk.BookName << " " << bk.Author << " " << bk.price;
	return os;
}
Book operator+(const Book&lhs,const Book&rhs){
	auto sum = lhs;
	sum += rhs;
	return sum;
}
bool operator==(const Book&lhs,const Book&rhs){
	return lhs.isbn()==rhs.isbn();
}
bool operator!=(const Book&lhs,const Book&rhs){
	return !(lhs == rhs);
}

bool operator<(const Book&lhs ,const Book& rhs){
	return lhs.isbn() < rhs.isbn();
}
bool operator>(const Book&lhs,const Book& rhs){
	return rhs < lhs;
}

Book& Book::operator=(const Book& rhs){
	BookNo = rhs.BookNo;
	BookName = rhs.BookName;
	Author = rhs.Author;
	price = rhs.price;
	return *this;
}
Book& Book::operator=(Book&& rhs)noexcept{
	if(this != &rhs){
		BookNo = rhs.BookNo;
		BookName = rhs.BookName;
		Author = rhs.Author;
		price = rhs.price;
	}
	return *this;
}
Book::Book (const Book& rhs)
	:BookNo(rhs.BookNo),BookName(rhs.BookName),Author(rhs.Author),price(rhs.price){}
Book::Book (Book&& rhs)noexcept
	:BookNo(rhs.BookNo),BookName(rhs.BookName),Author(rhs.Author),price(rhs.price){}
```



> 14.25 上题的这个类还需要定义其他赋值运算符吗？如果是，请实现它们；同时说明运算对象应该是什么类型并解释原因。

见`14.24`



## 14.5 下标运算符

> 14.26 为你的 `StrBlob` 类、`StrBlobPtr` 类、`StrVec` 类和 `String` 类定义下标运算符。

```cpp
	std::string& operator[](std::size_t n) { return elements[n]; }
	const std::string& operator[](std::size_t n) const { return elements[n]; }
```



## 14.6 递增和递减运算符

> 14.27  为你的StrBlobPtr类添加递增和递减运算符。
>
> 14.28 为你的StrBlobPtr类添加加法和减法运算符，使其可以实现指针的算术运算。



> 14.29 为什么不定义const版本的递增和递减运算符？

会改变调用对象内容



## 14.7 成员访问运算符

> 14.30 为你的`StrBlob`类和在`12.1.6`节练习`12.22`中定义的`ConstStrBlobPtr`类分别添加解引用运算符和箭头运算符。注意：因为`ConstStrBlobPtr`的数据成员指向`const vector`，所以`ConstStrBlobPtr`中的运算符必须返回常量引用。



