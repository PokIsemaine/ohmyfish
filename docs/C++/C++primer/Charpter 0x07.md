<h1 align="center">📔 C++ Primer 0x07 练习题解</h1>

## 7.1  定义抽象数据类型

### 7.1.1 定义抽象数据类型

> 7.1 使用2.6.1节定义的`Sales_data`类为 1.6节 的交易处理程序编写一个新版本。

```cpp
#include <iostream>
#include <string>

struct Sales_data {
	std::string bookNo;
	unsigned units_sold = 0;
	double revenue = 0.0;
};

int main(){
	Sales_data total,item;
    double price = 0;
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

### 7.1.2  定义改进的 Salses_data 类

> 7.2 曾在2.6.2节的练习中编写了一个`Sales_data`类，请向这个类添加`combine`函数和`isbn`成员。
>
> 7.3 修改7.1.1节的交易处理程序，令其使用这些成员。

```cpp
#include <iostream>
#include <string>

struct Sales_data {
	std::string bookNo;
	unsigned units_sold = 0;
	double revenue = 0.0;
	
	Sales_data& combine(const Sales_data &rhs);
	std::string isbn(){return bookNo;}
};

Sales_data& Sales_data::combine(const Sales_data &rhs){
	units_sold += rhs.units_sold;
	revenue += rhs.revenue;
	return *this;
}

int main(){
	Sales_data total,item;
	double price = 0;
    if (std::cin >> total.bookNo >> total.units_sold >> price){
    	total.revenue = total.units_sold * price;
        while (std::cin >> item.bookNo >> item.units_sold >> price){
        	item.revenue = item.units_sold * price;
            if (total.isbn() == item.isbn()) {
                total.combine(item);
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



> 7.4 编写一个名为`Person`的类，使其表示人员的姓名和地址。使用`string`对象存放这些元素，接下来的练习将不断充实这个类的其他特征。

```cpp
struct Person{
	std::string name;
	std::string address;
};
```



> 7.5 在你的`Person`类中提供一些操作使其能够返回姓名和地址。 这些函数是否应该是`const`的呢？解释原因。

```cpp
struct Person{
	std::string name;
	std::string address;
	std::string get_name()const{return name;}
	std::string get_address()const{return address;}
};
```

应该加，考虑常量对象可能也要使用这些操作



### 7.1.3 定义类相关的非成员函数

> 7.6 对于函数`add`、`read`和`print`，定义你自己的版本。
>
> 7.7 使用这些新函数重写7.1.2节练习中的程序。

```cpp
#include <iostream>
#include <string>


struct Sales_data {
	std::string bookNo;
	unsigned units_sold = 0;
	double revenue = 0.0;
	
	Sales_data& combine(const Sales_data &rhs);
	std::string isbn()const{return bookNo;}
};

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
	os << item.isbn() << " "<< item.units_sold << " "<< item.revenue;
	return os;
}

Sales_data add(const Sales_data &lhs,const Sales_data &rhs){
	Sales_data sum = lhs;
	sum.combine(rhs);
	return sum;
}

int main(){
	Sales_data total,item;
    if (read(std::cin,total)){
        while (read(std::cin,item)){
            if (total.isbn() == item.isbn()) {
                total.combine(item);
            }else {
                print(std::cout,total);
                std::cout << std::endl;
                total = item;
            }
        }
        print(std::cout,total);
        std::cout << std::endl;
        
    }else {
        std::cerr << "No data?!" << std::endl;
        return -1;
    }
	return 0;
}
```



> 7.8 为什么`read`函数将其`Sales_data`参数定义成普通的引用，而`print`函数将其参数定义成常量引用？

`read` 会改变`Sales_data item`的内容，`print`不会



> 7.9 对于7.1.2节练习中代码，添加读取和打印`Person`对象的操作。

```cpp
#include <iostream>
#include <string>

struct Person{
	std::string name;
	std::string address;
	std::string get_name()const{return name;}
	std::string get_address()const{return address;}
};

std::istream& read(std::istream &is,Person &p){
	is >> p.name >> p.address;
	return is;
}

std::ostream& print(std::ostream &os,const Person &p){
	os << p.get_name() << " " << p.get_address();
	return os;
}
int main(){
	Person p;
	read(std::cin,p);
	print(std::cout,p);
	return 0;
}
```



> 7.10 在下面这条`if`语句中，条件部分的作用是什么？

```cpp
if (read(read(cin, data1), data2)) //连续读入 data1 data2 的数据
```



### 7.1.4 构造函数

> 7.11 在你的`Sales_data`类中添加构造函数， 然后编写一段程序令其用到每个构造函数。

```cpp
#include <iostream>
#include <string>


struct Sales_data {
	Sales_data() = default;
	Sales_data(const std::string &s):bookNo(s){}
	Sales_data(const std::string &s,unsigned u,double p)
		:bookNo(s),units_sold(u),revenue(u*p){}
	Sales_data(std::istream &);
	
	std::string bookNo;
	unsigned units_sold = 0;
	double revenue = 0.0;
	
	Sales_data& combine(const Sales_data &rhs);
	std::string isbn()const{return bookNo;}
	double avg_price()const;
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

Sales_data::Sales_data(std::istream &is){
	read(is,*this);
}

Sales_data add(const Sales_data &lhs,const Sales_data &rhs){
	Sales_data sum = lhs;
	sum.combine(rhs);
	return sum;
}

int main(){
	Sales_data A;
	print(std::cout,A) << std::endl;
	
	Sales_data B("bookB");
	print(std::cout,B) << std::endl;
	
	Sales_data C("bookC",12,3.0);
	print(std::cout,C) << std::endl;
	
	Sales_data D(std::cin);
	print(std::cout,D) << std::endl;
	
	return 0;
}
```



> 7.12 把只接受一个`istream`作为参数的构造函数移到类的内部。

```cpp
#include <iostream>
#include <string>

struct Sales_data;
std::istream &read(std::istream&,Sales_data&);

struct Sales_data {
	Sales_data() = default;
	Sales_data(const std::string &s):bookNo(s){}
	Sales_data(const std::string &s,unsigned u,double p)
		:bookNo(s),units_sold(u),revenue(u*p){}
	Sales_data(std::istream &is){read(is,*this);}
	
	std::string bookNo;
	unsigned units_sold = 0;
	double revenue = 0.0;
	
	Sales_data& combine(const Sales_data &rhs);
	std::string isbn()const{return bookNo;}
	double avg_price()const;
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
	Sales_data A;
	print(std::cout,A) << std::endl;
	
	Sales_data B("bookB");
	print(std::cout,B) << std::endl;
	
	Sales_data C("bookC",12,3.0);
	print(std::cout,C) << std::endl;
	
	Sales_data D(std::cin);
	print(std::cout,D) << std::endl;
	
	return 0;
}
```



> 7.13 使用`istream`构造函数重写第229页的程序。

```cpp
#include <iostream>
#include <string>


struct Sales_data;
std::istream &read(std::istream&,Sales_data&);

struct Sales_data {
	Sales_data() = default;
	Sales_data(const std::string &s):bookNo(s){}
	Sales_data(const std::string &s,unsigned u,double p)
		:bookNo(s),units_sold(u),revenue(u*p){}
	Sales_data(std::istream &is){read(is,*this);}
	
	std::string bookNo;
	unsigned units_sold = 0;
	double revenue = 0.0;
	
	Sales_data& combine(const Sales_data &rhs);
	std::string isbn()const{return bookNo;}
	double avg_price()const;
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
	Sales_data total(std::cin);
    if (!total.isbn().empty()){
    	std::istream &is = std::cin;
        while (is){
        	Sales_data item(is);
            if (total.isbn() == item.isbn()) {
                total.combine(item);
            }else {
                print(std::cout,total) << std::endl;
                total = item;
            }
        }
        print(std::cout,total) << std::endl;
        
    }else {
        std::cerr << "No data?!" << std::endl;
        return -1;
    }
	
	return 0;
}
```



> 7.14 编写一个构造函数，令其用我们提供的类内初始值显式地初始化成员。

```cpp
Sales_data():bookNo(0),revenue(0){}
```



> 7.15 为你的`Person`类添加正确的构造函数。

```cpp
#include <iostream>
#include <string>

struct Person;
std::istream& read(std::istream &,Person &);
struct Person{
	Person() = default;
	Person(const std::string n,const std::string a)
		:name(n),address(a){}
	Person(std::istream& is){read(is,*this);}
	
	std::string name;
	std::string address;
	std::string get_name()const{return name;}
	std::string get_address()const{return address;}
};

std::istream& read(std::istream &is,Person &p){
	is >> p.name >> p.address;
	return is;
}

std::ostream& print(std::ostream &os,const Person &p){
	os << p.get_name() << " " << p.get_address();
	return os;
}
int main(){
	Person p(std::cin);
	print(std::cout,p) << std::endl;
	
	Person p2("fishingrod","zj");
	print(std::cout,p2) << std::endl;
	return 0;
}
```



## 7.2 访问控制与封装

> 7.16 在类的定义中对于访问说明符出现的位置和次数有限定吗？ 如果有，是什么？什么样的成员应该定义在`public`说明符之后？ 什么样的成员应该定义在`private`说明符之后？

没有限制

`public`说明符之后的成员在整个程序内可以被访问，`public`成员定义类的接口

`private`说明符之后的成员可以被类的成员函数访问，但是不能使用该类的代码访问，`private`封装了类的实现细节



> 7.17 使用`class`和`struct`时有区别吗？如果有，是什么？

使用`class`和`struct`定义类的唯一区别就是默认的访问权限，`struct`默认所有成员是`public`，`class`默认所有成员是`private`



> 7.18 封装是何含义？它有什么用处？

封装实现了类的接口和实现的分离，隐藏了实现细节，类的用户只能使用接口而无法访问实现部分

封装的两个重要优点

* 确保用户代码不会无意间破坏封装对象的状态
* 被封装的类的具体实现可以随时改变，无需调整用户级别的代码



> 7.19 在你的`Person`类中，你将把哪些成员声明成`public`的？ 哪些声明成`private`的？ 解释你这样做的原因。

修改了一下

```cpp
struct Person{
private://数据设为私有，用户不可见	
	std::string name;
	std::string address;
public://get_name、get_address、构造函数公有
	std::string get_name()const{return name;}
	std::string get_address()const{return address;}
    Person() = default;
	Person(const std::string n,const std::string a)
		:name(n),address(a){}
	Person(std::istream& is){read(is,*this);}
};

```



### 7.2.1 友元

> 7.20 友元在什么时候有用？请分别举出使用友元的利弊。

友元声明让类允许其他类或函数访问它的非公有成员

实际上破坏了封装性和可维护性



> 7.21 修改你的`Sales_data`类使其隐藏实现的细节。 你之前编写的关于`Sales_data`操作的程序应该继续使用，借助类的新定义重新编译该程序，确保其正常工作。

```cpp
#include <iostream>
#include <string>


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
	
	Sales_data() = default;
	Sales_data(const std::string &s):bookNo(s){}
	Sales_data(const std::string &s,unsigned u,double p)
		:bookNo(s),units_sold(u),revenue(u*p){}
	Sales_data(std::istream &is){read(is,*this);}
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
	Sales_data total(std::cin);
    if (!total.isbn().empty()){
    	std::istream &is = std::cin;
        while (is){
        	Sales_data item(is);
            if (total.isbn() == item.isbn()) {
                total.combine(item);
            }else {
                print(std::cout,total) << std::endl;
                total = item;
            }
        }
        print(std::cout,total) << std::endl;
        
    }else {
        std::cerr << "No data?!" << std::endl;
        return -1;
    }
	
	return 0;
}
```



> 7.22 修改你的`Person`类使其隐藏实现的细节。

```cpp
#include <iostream>
#include <string>

class Person;
std::istream& read(std::istream &,Person &);
class Person{
friend std::istream& read(std::istream &,Person &);
friend std::ostream& print(std::ostream &,const Person &);

private:	
	std::string name;
	std::string address;

public:
	std::string get_name()const{return name;}
	std::string get_address()const{return address;}
	
	Person() = default;
	Person(const std::string n,const std::string a)
		:name(n),address(a){}
	Person(std::istream& is){read(is,*this);}
};

std::istream& read(std::istream &is,Person &p){
	is >> p.name >> p.address;
	return is;
}

std::ostream& print(std::ostream &os,const Person &p){
	os << p.get_name() << " " << p.get_address();
	return os;
}
int main(){
	Person p(std::cin);
	print(std::cout,p) << std::endl;
	
	Person p2("fishingrod","zj");
	print(std::cout,p2) << std::endl;
	return 0;
}
```



## 7.3 类的其他特性

### 7.3.1 类成员再探

> 7.23 编写你自己的`Screen`类型。

```cpp
class Screen{
public:
	typedef std::string::size_type pos;
	Screen() = default;
	Screen(pos ht,pos wd,char c)
		:height(ht),width(wd),contents(ht*wd,c){}
	char get()const{return contents[cursor];}
	char get(pos r,pos c)const{return contents[r*width+c];}
    Screen &move (pos r,pos c);
private:
	pos cursor = 0;
	pos height = 0, width = 0;
	std::string contents;
};

inline Screen& Screen::move(pos r,pos c){
	pos row = r * width;
	cursor = row + c;
	return *this;
}
```



> 7.24 给你的`Screen`类添加三个构造函数：一个默认构造函数；另一个构造函数接受宽和高的值，然后将`contents`初始化成给定数量的空白；第三个构造函数接受宽和高的值以及一个字符，该字符作为初始化后屏幕的内容。

```cpp
class Screen{
public:
	typedef std::string::size_type pos;

public:
	Screen() = default;
	Screen(pos ht,pos wd)
		:height(ht),width(wd),contents(ht*wd,' '){}
	Screen(pos ht,pos wd,char c)
		:height(ht),width(wd),contents(ht*wd,c){}

public:
	char get()const{return contents[cursor];}
	char get(pos r,pos c)const{return contents[r*width+c];}
    Screen &move (pos r,pos c);
    
private:
	pos cursor = 0;
	pos height = 0, width = 0;
	std::string contents;
};

inline Screen& Screen::move(pos r,pos c){
	pos row = r * width;
	cursor = row + c;
	return *this;
}
```



> 7.25 `Screen`能安全地依赖于拷贝和赋值操作的默认版本吗？ 如果能，为什么？如果不能？为什么？

可以，采用了`string`能够管理必要的存储空间



* 某些类不能依赖于合成的版本，当类需要分配类对象之外的资源时，合成的版本常常会失效
* 很多动态内存的类能（而且应该）使用 `vector` 对象或者 `string` 对象管理必要的存储空间，这能避免分配和释放内存带来的复杂性
* 当含有`vector`成员对象执行拷贝或赋值操作时，`vector`会设法拷贝或赋值成员中的元素。销毁时也会一次销毁`vector`中的每一个元素。这点与 `string`是非常类似的



> 7.26 将`Sales_data::avg_price`定义成内联函数。

```cpp
inline
double Sales_data::avg_price()const{
	return units_sold?(revenue/units_sold):0;
}
```



### 7.3.2 返回 *this 的成员函数

> 7.27 给你自己的`Screen`类添加`move`、`set` 和`display`函数，通过执行下面的代码检验你的类是否正确。

```cpp
int main()
{
    Screen myScreen(5, 5, 'X');
    myScreen.move(4, 0).set('#').display(std::cout);
    std::cout << "\n";
    myScreen.display(std::cout);
    std::cout << "\n";

    return 0;
}
```



```cpp
#include <iostream>
#include <string>


class Screen{
public:
	typedef std::string::size_type pos;

public:
	Screen() = default;
	Screen(pos ht,pos wd)
		:height(ht),width(wd),contents(ht*wd,' '){}
	Screen(pos ht,pos wd,char c)
		:height(ht),width(wd),contents(ht*wd,c){}

public:
	char get()const{return contents[cursor];}
	char get(pos r,pos c)const{return contents[r*width+c];}
	
    Screen& move(pos r,pos c);
    
    Screen& set(char c);
    Screen& set(pos r,pos col,char ch);
    
    Screen& display(std::ostream &os){
    	do_display(os);return *this;
    }
    const Screen& display(std::ostream &os)const{
    	do_display(os);return *this;
    }
    
private:
	pos cursor = 0;
	pos height = 0, width = 0;
	std::string contents;

private:
	void do_display(std::ostream& os)const{os << contents;}
};

inline 
Screen& Screen::set(char c){
	contents[cursor] = c;
	return *this;
}

inline 
Screen& Screen::set(pos r,pos col,char ch){
	contents[r * width + col] = ch;
	return *this;
}


inline 
Screen& Screen::move(pos r,pos c){
	pos row = r * width;
	cursor = row + c;
	return *this;
}

int main(){
	Screen myScreen(5, 5, 'X');
	myScreen.move(4, 0).set('#').display(std::cout);
	std::cout << "\n";
	myScreen.display(std::cout);
	std::cout << "\n";
	return 0;
}
```



> 7.28 如果`move`、`set`和`display`函数的返回类型不是`Screen&` 而是`Screen`，则在上一个练习中将会发生什么？
>
> 7.29 修改你的`Screen`类，令`move`、`set`和`display`函数返回`Screen`并检查程序的运行结果，在上一个练习中你的推测正确吗？

```cpp
myScreen.move(4, 0).set('#').display(std::cout);//move返回副本，set在副本上修改，display展示set修改后的另一个副本
//没有在同一对象上操作
```

```cpp
#include <iostream>
#include <string>


class Screen{
public:
	typedef std::string::size_type pos;

public:
	Screen() = default;
	Screen(pos ht,pos wd)
		:height(ht),width(wd),contents(ht*wd,' '){}
	Screen(pos ht,pos wd,char c)
		:height(ht),width(wd),contents(ht*wd,c){}

public:
	char get()const{return contents[cursor];}
	char get(pos r,pos c)const{return contents[r*width+c];}
	
    Screen move(pos r,pos c);
    
    Screen set(char c);
    Screen set(pos r,pos col,char ch);
    
    Screen display(std::ostream &os){
    	do_display(os);return *this;
    }
    const Screen display(std::ostream &os)const{
    	do_display(os);return *this;
    }
    
private:
	pos cursor = 0;
	pos height = 0, width = 0;
	std::string contents;

private:
	void do_display(std::ostream& os)const{os << contents;}
};

inline 
Screen Screen::set(char c){
	contents[cursor] = c;
	return *this;
}

inline 
Screen Screen::set(pos r,pos col,char ch){
	contents[r * width + col] = ch;
	return *this;
}


inline 
Screen Screen::move(pos r,pos c){
	pos row = r * width;
	cursor = row + c;
	return *this;
}

int main(){
	Screen myScreen(5, 5, 'X');
	myScreen.move(4, 0).set('#').display(std::cout);
	std::cout << "\n";
	myScreen.display(std::cout);
	std::cout << "\n";
	return 0;
}
```

输出

```cpp
XXXXXXXXXXXXXXXXXXXX#XXXX
XXXXXXXXXXXXXXXXXXXXXXXXX
```



> 7.30 通过`this`指针使用成员的做法虽然合法，但是有点多余。讨论显示使用指针访问成员的优缺点。

优点：

1、使程序意图明确，更易读；

2、可以使形参名和要赋值的成员名相同。

```cpp
std::string& setName(const string& name)  {  this->name = name; }
```

缺点：有些场景下比较多余

```cpp
std::string const& getName() const { return this->name; }
```



### 7.3.3 类类型

> 7.31 定义一对类X和Y，其中X包含一个指向Y的指针，而Y包含一个类型为X的对象。

```cpp
class Y;

class X{
	Y* point_y;
};

class Y{
	X x_in_y;
};
```



### 7.3.4 友元再探

> 7.32 定义你自己的`Screen`和`Window_mgr`，其中`clear`是`Window_mgr`的成员，是`Screen`的友元。

```cpp
#include <iostream>
#include <string>
#include <vector>

class Screen;
class Window_mgr{
public:
	using ScreenIndex = std::vector<Screen>::size_type;

public:
	void clear(ScreenIndex);

private:
	std::vector<Screen> screens;
};

class Screen{
public:
	typedef std::string::size_type pos;
	friend void Window_mgr::clear(ScreenIndex);

public:
	Screen() = default;
	Screen(pos ht,pos wd)
		:height(ht),width(wd),contents(ht*wd,' '){}
	Screen(pos ht,pos wd,char c)
		:height(ht),width(wd),contents(ht*wd,c){}

public:
	char get()const{return contents[cursor];}
	char get(pos r,pos c)const{return contents[r*width+c];}
	
    Screen move(pos r,pos c);
    
    Screen set(char c);
    Screen set(pos r,pos col,char ch);
    
    Screen display(std::ostream &os){
    	do_display(os);return *this;
    }
    const Screen display(std::ostream &os)const{
    	do_display(os);return *this;
    }
    
private:
	pos cursor = 0;
	pos height = 0, width = 0;
	std::string contents;

private:
	void do_display(std::ostream& os)const{os << contents;}
};

inline 
Screen Screen::set(char c){
	contents[cursor] = c;
	return *this;
}

inline 
Screen Screen::set(pos r,pos col,char ch){
	contents[r * width + col] = ch;
	return *this;
}


inline 
Screen Screen::move(pos r,pos c){
	pos row = r * width;
	cursor = row + c;
	return *this;
}

void Window_mgr::clear(ScreenIndex i){
	Screen& s = screens[i];
	s.contents = std::string(s.height*s.width,' ');
}
```

* 首先定义 `Window_mgr`类，其中声明`clear`函数但不定义，在`clear`使用Screen的成员之前必须先声明`Screen`
* 接下来定义`Screen`，包括对于`clear`的友元声明
* 最后定义`clear`此时它才可以使用`Screen`的成员

## 7.4 类的作用域
