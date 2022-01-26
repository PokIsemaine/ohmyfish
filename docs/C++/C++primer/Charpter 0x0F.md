<h1 align="center">📔 C++ Primer 0x0F 练习题解</h1>

## 15.2 定义基类和派生类

### 15.2.1 定义基类

> 15.1 什么是虚成员

通过关键字`virtural`可以将成员声明为虚成员

虚函数用来区分基类里希望派生类进行覆盖的函数和不希望派生类覆盖的函数



> 15.2 `protected` 访问说明符与 `private` 有何区别？

* `protected`允许派生类、基类、友元访问，禁止其他用户访问
* `private`只允许基类本身和友元访问





> 15.3 定义你自己的 `Quote` 类和 `print_total` 函数。

```cpp
#include <cstddef>
#include <iostream>
#include <string>


class Quote{
public:
	Quote() = default;
	Quote(const std::string &book,double sales_price)
		:bookNo(book),price(sales_price){}
	std::string isbn()const{return bookNo;}

	virtual double net_price(std::size_t n)const{return n * price;}
	virtual ~Quote() = default;	//对析构函数动态绑定
private:
	std::string bookNo;	//ISBN编号
protected:
	double price = 0.0; //不打折的价格
};


double print_total(std::ostream &os, const Quote &item, size_t n);
```

```cpp
#include "Quote.hpp"

double print_total(std::ostream &os, const Quote &item, size_t n){
	double ret = item.net_price(n);
	os << "ISBN: " << item.isbn() << "# sold: " << n << "total due: " << ret;
	return  ret;
}
int main(){

	return 0;
}
```



### 15.2.2 定义派生类

> 15.4 下面哪条声明语句是不正确的？请解释原因。

```cpp
class Base { ... };
(a) class Derived : public Derived { ... };//不能派生自己
(b) class Derived : private Base { ... };//这是定义
(c) class Derived : public Base;//声明不包含派生列表
```



> 15.5 定义你自己的 `Bulk_quote` 类。

```cpp
#include <cstddef>
#include <iostream>
#include <string>


class Quote{
public:
	Quote() = default;
	Quote(const std::string &book,double sales_price)
		:bookNo(book),price(sales_price){}
	std::string isbn()const{return bookNo;}
	virtual ~Quote() = default;	//对析构函数动态绑定

public:
	virtual double net_price(std::size_t n)const{return n * price;}

private:
	std::string bookNo;	//ISBN编号
protected:
	double price = 0.0; //不打折的价格
};

class Bulk_quote:public Quote {
public:
	Bulk_quote() = default;
	Bulk_quote(const std::string& b,double p,std::size_t q,double disc)
		:Quote(b,p),min_qty(q),discount(disc){}
public:
	double net_price(std::size_t)const override;	//覆盖基类函数版本以实现基于大量购买的折扣政策

private:
	std::size_t min_qty = 0;	//适用折扣政策的最低购买量
	double discount = 0.0;	//以小数表示的折扣额
};

double print_total(std::ostream &os, const Quote &item, size_t n);
```



> 15.6 将 `Quote` 和 `Bulk_quote` 的对象传给15.2.1节练习中的 `print_total` 函数，检查该函数是否正确。

```cpp
#include <cstddef>
#include <iostream>
#include <string>


class Quote{
public:
	Quote() = default;
	Quote(const std::string &book,double sales_price)
		:bookNo(book),price(sales_price){}
	std::string isbn()const{return bookNo;}
	virtual ~Quote() = default;	//对析构函数动态绑定

public:
	virtual double net_price(std::size_t n)const{return n * price;}

private:
	std::string bookNo;	//ISBN编号
protected:
	double price = 0.0; //不打折的价格
};

class Bulk_quote:public Quote {
public:
	Bulk_quote() = default;
	Bulk_quote(const std::string& b,double p,std::size_t q,double disc)
		:Quote(b,p),min_qty(q),discount(disc){}
public:
	double net_price(std::size_t cnt)const override{//覆盖基类函数版本以实现基于大量购买的折扣政策
		if (cnt >= min_qty) return cnt * (1-discount) * price;
		else return cnt * price;
	}	

private:
	std::size_t min_qty = 0;	//适用折扣政策的最低购买量
	double discount = 0.0;	//以小数表示的折扣额
};

double print_total(std::ostream &os, const Quote &item, size_t n);
```

```cpp
#include "Quote.hpp"

double print_total(std::ostream &os, const Quote &item, size_t n){
	double ret = item.net_price(n);
	os << "ISBN: " << item.isbn() << " # sold: " << n << " total due: " << ret;
	return  ret;
}

int main(){
	Quote q("book1",15.4);
	Bulk_quote bq("book2",15.4,5,0.3);
	print_total(std::cout,q,10);
	std::cout << std::endl;
	print_total(std::cout,bq,10);
	std::cout << std::endl;
	return 0;
}
```



> 15.7 定义一个类使其实现一种数量受限的折扣策略，具体策略是：当购买书籍的数量不超过一个给定的限量时享受折扣，如果购买量一旦超过了限量，则超出的部分将以原价销售。

```cpp
class Limit_quote:public Quote {
public:
	Limit_quote() = default;
	Limit_quote(const std::string& b,double p,std::size_t q,double disc)
		:Quote(b,p),max_qty(q),discount(disc){}
public:
	double net_price(std::size_t cnt)const override{
		if(cnt <= max_qty)return cnt * (1-discount) * price;
		return max_qty * (1-discount) * price + (cnt-max_qty) * price;
	}
private:
	std::size_t max_qty = 0;
	double discount = 0.0;	//以小数表示的折扣额

};
```



### 15.2.3 类型转换与继承

> 15.8 给出静态类型和动态类型的定义。

* 表达式的静态类型是在编译时已知的，它是变量声明时的类型或表达式生成的类型
* 动态类型是变量或表达式表示的内存中的对象的类型，直到运行时才可知



> 15.9 在什么情况下表达式的静态类型可能与动态类型不同？请给出三个静态类型与动态类型不同的例子。

基类的指针或引用的静态类型与动态类型可能不一致



> 15.10 回忆我们在8.1节进行的讨论，解释第284页中将 `ifstream` 传递给 `Sales_data` 的`read` 函数的程序是如何工作的。

在要求使用基类型对象的地方，我们可以用继承类型的对象来代替

`std::ifstream`是`std::istream`的派生类，所以`read`传入`std::ifstream`也是可以的



## 15.3 虚函数

> 15.11 为你的 `Quote` 类体系添加一个名为 `debug` 的虚函数，令其分别显示每个类的数据成员。

```cpp
#include <cstddef>
#include <iostream>
#include <string>


class Quote{
public:
	Quote() = default;
	Quote(const std::string &book,double sales_price)
		:bookNo(book),price(sales_price){}
	std::string isbn()const{return bookNo;}
	virtual ~Quote() = default;	//对析构函数动态绑定

public:
	virtual double net_price(std::size_t n)const{return n * price;}
	virtual void debug()const{
		std::cout << "data members of this class:\n"
			<< "bookNo= " <<this->bookNo << " "
			<< "price= " <<this->price<< " ";
	}

private:
	std::string bookNo;	//ISBN编号
protected:
	double price = 0.0; //不打折的价格
};

class Bulk_quote:public Quote {
public:
	Bulk_quote() = default;
	Bulk_quote(const std::string& b,double p,std::size_t q,double disc)
		:Quote(b,p),min_qty(q),discount(disc){}
public:
	double net_price(std::size_t cnt)const override{//覆盖基类函数版本以实现基于大量购买的折扣政策
		if (cnt >= min_qty) return cnt * (1-discount) * price;
		return cnt * price;
	}	
	void debug()const override {
		std::cout << "data members of this class:\n"
			<< "bookNo= " <<this->min_qty << " "
			<< "price= " <<this->discount<< " ";
	}
private:
	std::size_t min_qty = 0;	//适用折扣政策的最低购买量
	double discount = 0.0;	//以小数表示的折扣额
};

class Limit_quote:public Quote {
public:
	Limit_quote() = default;
	Limit_quote(const std::string& b,double p,std::size_t q,double disc)
		:Quote(b,p),max_qty(q),discount(disc){}
public:
	double net_price(std::size_t cnt)const override{
		if(cnt <= max_qty)return cnt * (1-discount) * price;
		return max_qty * (1-discount) * price + (cnt-max_qty) * price;
	}

	void debug()const override {
		std::cout << "data members of this class:\n"
			<< "bookNo= " <<this->max_qty << " "
			<< "price= " <<this->discount<< " ";
	}
private:
	std::size_t max_qty = 0;
	double discount = 0.0;	//以小数表示的折扣额

};

double print_total(std::ostream &os, const Quote &item, size_t n);
```





> 15.12 有必要将一个成员函数同时声明成 `override` 和 `final` 吗？为什么？

有必要

* 使用`override`可以让意图更加清晰，如果使用`override`标记了某个函数，但该函数没有覆盖已存在基类的虚函数，编译器会报错

* `final`关键字用来防止继承，禁止了派生类覆盖该函数



> 15.13 给定下面的类，解释每个 `print` 函数的机理：

```cpp
class base {
public:
	string name() { return basename;}
	virtual void print(ostream &os) { os << basename; }
private:
	string basename;
};
class derived : public base {
public:
	void print(ostream &os) { print(os); os << " " << i; }
private:
	int i;
};
```

在上述代码中存在问题吗？如果有，你该如何修改它？



如果一个派生类虚函数需要调用它的基类版本，但没有使用作用于运算符，则运行时该调用将被解析为对派生类版本自身的调用，从而导致无限递归

顺便加上`override`这样更明确意图

改为

```cpp
public:
	void print(ostream &os)override { base::print(os); os << " " << i; }
```



> 15.14 给定上一题中的类以及下面这些对象，说明在运行时调用哪个函数：

```cpp
base bobj; 		base *bp1 = &bobj; 	base &br1 = bobj;
derived dobj; 	base *bp2 = &dobj; 	base &br2 = dobj;

(a) bobj.print();//base::print()
(b)dobj.print();//derived::print()
(c)bp1->name();//base::name()
(d)bp2->name();//base::name()		
(e)br1.print();		
(f)br2.print();
```



## 15.4 抽象基类

> 15.15 定义你自己的 `Disc_quote` 和 `Bulk_quote`。

```cpp
#include <cstddef>
#include <iostream>
#include <string>


class Quote{
public:
	Quote() = default;
	Quote(const std::string &book,double sales_price)
		:bookNo(book),price(sales_price){}
	std::string isbn()const{return bookNo;}
	virtual ~Quote() = default;	//对析构函数动态绑定

public:
	virtual double net_price(std::size_t n)const{return n * price;}
	virtual void debug()const{
		std::cout << "data members of this class:\n"
			<< "bookNo= " <<this->bookNo << " "
			<< "price= " <<this->price<< " ";
	}

private:
	std::string bookNo;	//ISBN编号
protected:
	double price = 0.0; //不打折的价格
};

class Disc_quote:public Quote{//抽象基类
public:
	Disc_quote();
	Disc_quote(const std::string& b,double p,std::size_t q,double d)
		:Quote(b,p),quantity(q),discount(d){}
public:
	double net_price(std::size_t)const override = 0 ;
	void debug()const override {
		std::cout << "data members of this class:\n"
			<< "bookNo= " <<this->quantity << " "
			<< "price= " <<this->discount<< " ";
	}
protected:
	std::size_t quantity = 0;//折扣适用的购买量
    double	discount = 0.0;//表示折扣的小数值
};

class Bulk_quote:public Disc_quote {
public:
	Bulk_quote() = default;
	Bulk_quote(const std::string& b,double p,std::size_t q,double disc)
		:Disc_quote(b,p,q,disc){}
public:
	double net_price(std::size_t cnt)const override{//覆盖基类函数版本以实现基于大量购买的折扣政策
		if (cnt >= quantity) return cnt * (1-discount) * price;
		return cnt * price;
	}	

};

class Limit_quote:public Disc_quote {
public:
	Limit_quote() = default;
	Limit_quote(const std::string& b,double p,std::size_t q,double disc)
		:Disc_quote(b,p,q,disc){}
public:
	double net_price(std::size_t cnt)const override{
		if(cnt <= quantity)return cnt * (1-discount) * price;
		return quantity * (1-discount) * price + (cnt-quantity) * price;
	}
};

double print_total(std::ostream &os, const Quote &item, size_t n);
```



> 15.16 改写你在15.2.2节练习中编写的数量受限的折扣策略，令其继承 `Disc_quote`。

见`15.15`



> 15.17 尝试定义一个 `Disc_quote` 的对象，看看编译器给出的错误信息是什么？

`variable type 'Disc_quote' is an abstract class`



## 15.5 访问控制与继承

> 15.18 假设给定了第543页和第544页的类，同时已知每个对象的类型如注释所示，判断下面的哪些赋值语句是合法的。解释那些不合法的语句为什么不被允许：

```cpp
Base *p = &d1;  //d1 的类型是 Pub_Derv 合法
p = &d2;		//d2 的类型是 Priv_Derv
p = &d3;		//d3 的类型是 Prot_Derv
p = &dd1;		//dd1 的类型是 Derived_from_Public	合法
p = &dd2;		//dd2 的类型是 Derived_from_Private
p = &dd3;		//dd3 的类型是 Derived_from_Protected
```

只有当派生类是`public`继承时，用户代码才可以使用派生类到基类的转换



> 15.19 假设543页和544页的每个类都有如下形式的成员函数：

```cpp
void memfcn(Base &b) { b = *this; }
```

对于每个类，分别判断上面的函数是否合法。

* 不论D以什么方式继承B，D的成员函数和友元都能使用派生类向基类的转换；派生类向其直接基类的类型转换对于派生类的成员和友元永远都是可访问的

* `Pub_Derv`合法
* `Priv_Derv`合法
* `Prot_Derv`合法

* 如果D继承B的方式是公有的或受保护的，则**D的派生类**的成员和友元可以使用D向B的类型转换，反之不行

* `Derived_from_Public`合法
*  `Derived_from_Private`非法
* `Derived_from_Protected`合法

> 15.20 编写代码检验你对前面两题的回答是否正确。



> 15.21从下面这些一般性抽象概念中任选一个（或者选一个你自己的），将其对应的一组类型组织成一个继承体系：

```cpp
(a) 图形文件格式（如gif、tiff、jpeg、bmp）
(b) 图形基元（如方格、圆、球、圆锥）
(c) C++语言中的类型（如类、函数、成员函数）
```

```cpp
#include <iostream>
#include <utility>
#include <cmath>

using Coordinate = std::pair<double,double>;
double Distance(const Coordinate& Point1,const Coordinate& Point2);
class Shape{

public:
	Shape() = default;
	Shape(const std::string& s):id(s){}
	virtual ~Shape() = default;
public:
	virtual double perimeter()const=0;
	virtual double area()const=0;
private:
	std::string id;
};

class Rectangle:public Shape{
public:
	friend double Distance(const Coordinate& Point1,const Coordinate& Point2);
public:
	Rectangle() = default;
	Rectangle(const std::string& s,
			const Coordinate& a,
			const Coordinate& b,
			const Coordinate& c,
			const Coordinate& d)
	:Shape(s),A(a),B(b),C(c),D(d){}
	~Rectangle() = default;
public:
	double perimeter() const override { 
		return 2*Distance(A,B)+Distance(C,D);
	}
	double area()const override{
		return Distance(A,B)*Distance(C,D);
	}

protected:
	Coordinate A;
	Coordinate B;
	Coordinate C;
	Coordinate D;

};

class Square:public Rectangle{
public:
	Square() = default;
	Square(const std::string& s,
			const Coordinate& a,
			const Coordinate& b,
			const Coordinate& c,
			const Coordinate& d)
	:Rectangle(s,a,b,c,d){}
	~Square() = default;
};
double Distance(const Coordinate& Point1,const Coordinate& Point2){
	return std::sqrt((Point1.first-Point2.first)*(Point1.first-Point2.first)
				+(Point1.second-Point2.second)*(Point1.second-Point2.second));
}
int main(){
	Rectangle rt("rt1",{0,0},{10,0},{0,10},{10,10});
	std::cout << rt.area() <<" " << rt.perimeter() << std::endl;
	Square sq("sq1",{0,0},{10,0},{0,10},{10,10});
	std::cout << sq.area() <<" " << sq.perimeter() << std::endl;
	return 0;
}
```



> 15.22 对于你在上一题中选择的类，为其添加函数的虚函数及公有成员和受保护的成员。

见`15.21`

## 15.6 继承中的类作用域

> 15.23 假设第550页的 `D1` 类需要覆盖它继承而来的 `fcn` 函数，你应该如何对其进行修改？如果你修改之后 `fcn` 匹配了 `Base` 中的定义，则该节的那些调用语句将如何解析？

去掉`int`

一个派生类如果覆盖了某个继承来的虚函数，它的形参类型必须要与被覆盖的基类函数完全一致

## 15.7 构造函数与拷贝控制

### 15.7.1 虚析构函数

> 15.24 哪种类需要虚析构函数？虚析构函数必须执行什么样的操作？

基类

* 只要基类的析构函数是虚函数，就能确保我们`delete`基类指针时将运行正确的析构函数版本
* 如果基类的析构函数不是虚函数，则`delete`一个指向派生类对象的基类指针将产生未定义的行为
* 基类的虚析构函数，并不像普通类的一样，我们不能因为析构函数推断出该类是否还需要赋值运算符和拷贝构造函数。因为为了成为虚函数，它的内容是空的



### 15.7.2 合成拷贝控制与继承

> 15.25 我们为什么为 `Disc_quote` 定义一个默认构造函数？如果去掉该构造函数的话会对 `Bulk_quote` 的行为产生什么影响？

因为大部分情况下不同的类定义了不同的构造函数，那么就不会自动合成默认构造函数了，需要自己定义

`Bulk_quote`的默认构造函数运行`Disc_quote`的默认构造函数，后者又运行`Quote`的默认构造函数，如果`Disc_quote`没有默认构造函数，那`Bulk_quote`也没法完成初始化工作



### 15.7.3 派生类的拷贝控制成员

> 15.26 定义 `Quote` 和 `Bulk_quote` 的拷贝控制成员，令其与合成的版本行为一致。为这些成员以及其他构造函数添加打印状态的语句，使得我们能够知道正在运行哪个程序。使用这些类编写程序，预测程序将创建和销毁哪些对象。重复实验，不断比较你的预测和实际输出结果是否相同，直到预测完全准确再结束。

```cpp
#include <cstddef>
#include <iostream>
#include <string>


class Quote{
public:
	Quote(){std::cout << "default constructing Quote\n";};
	Quote(const std::string &book,double sales_price)
		:bookNo(book),price(sales_price){
			std::cout << "Quote : constructor taking 2 parameters\n";
		}
	Quote(const Quote& q):bookNo(q.bookNo),price(q.price){
		std::cout << "Quote: copy constructing\n";
	}
	Quote(Quote&& q)noexcept:bookNo(std::move(q.bookNo)),price(std::move(q.price)){
		std::cout << "Quote: move constructing\n"; 
	}
	Quote& operator=(const Quote& rhs){
		if(this != &rhs){
			bookNo = rhs.bookNo;
			price = rhs.price;
		}
		std::cout << "Quote: copy =() \n";
		return *this;
	}
	Quote& operator=(Quote&& rhs)noexcept{
		if(this != &rhs){
			bookNo = std::move(rhs.bookNo);
			price = std::move(rhs.price);
		}
		std::cout << "Quote: move =() \n";
		return *this;
	}

	virtual ~Quote(){
		std::cout << "destructing Quote\n";
	}

public:
	std::string isbn()const{return bookNo;}
	virtual double net_price(std::size_t n)const{return n * price;}
	virtual void debug()const{
		std::cout << "data members of this class:\n"
			<< "bookNo= " <<this->bookNo << " "
			<< "price= " <<this->price<< " ";
	}

private:
	std::string bookNo;	//ISBN编号
protected:
	double price = 0.0; //不打折的价格
};

class Disc_quote:public Quote{//抽象基类
public:
	Disc_quote() = default;
	Disc_quote(const std::string& b,double p,std::size_t q,double d)
		:Quote(b,p),quantity(q),discount(d){}
public:
	double net_price(std::size_t)const override = 0 ;
	void debug()const override {
		std::cout << "data members of this class:\n"
			<< "bookNo= " <<this->quantity << " "
			<< "price= " <<this->discount<< " ";
	}
protected:
	std::size_t quantity = 0;//折扣适用的购买量
    double	discount = 0.0;//表示折扣的小数值
};

class Bulk_quote:public Disc_quote {
public:
	Bulk_quote(){ std::cout << "default constructing Bulk_quote\n";}
	Bulk_quote(const std::string& b,double p,std::size_t q,double disc)
		:Disc_quote(b,p,q,disc){
			std::cout << "Bulk_quote : constructor taking 4 parameters\n";
		}
	Bulk_quote(const Bulk_quote& bq):Disc_quote(bq){
		std::cout << "Bulk_quote : copy constructor\n";
	}
	Bulk_quote(Bulk_quote&& bq)noexcept:Disc_quote(std::move(bq)){
		std::cout << "Bulk_quote : move constructor\n";
	}
	Bulk_quote& operator=(const Bulk_quote& rhs){
		Disc_quote::operator=(rhs);
		std::cout << "Bulk_quote : copy =()\n";
		return *this;
	}
	Bulk_quote& operator=(Bulk_quote&& rhs){
		Disc_quote::operator=(rhs);
		std::cout << "Bulk_quote : move =()\n";
		return *this;
	}
	~Bulk_quote()
	{
		std::cout << "destructing Bulk_quote\n";
	}
public:
	double net_price(std::size_t cnt)const override{//覆盖基类函数版本以实现基于大量购买的折扣政策
		if (cnt >= quantity) return cnt * (1-discount) * price;
		return cnt * price;
	}	

};

class Limit_quote:public Disc_quote {
public:
	Limit_quote() = default;
	Limit_quote(const std::string& b,double p,std::size_t q,double disc)
		:Disc_quote(b,p,q,disc){}
public:
	double net_price(std::size_t cnt)const override{
		if(cnt <= quantity)return cnt * (1-discount) * price;
		return quantity * (1-discount) * price + (cnt-quantity) * price;
	}
};

double print_total(std::ostream &os, const Quote &item, size_t n);
```

```cpp
#include "Quote.hpp"

double print_total(std::ostream &os, const Quote &item, size_t n){
	double ret = item.net_price(n);
	os << "ISBN: " << item.isbn() << " # sold: " << n << " total due: " << ret;
	return  ret;
}

int main(){
	Quote q("book1",15.4);
	Quote q2;
	Quote q3(q);
	Quote q4 = q;

	Bulk_quote bq("book2",15.4,5,0.3);
	Bulk_quote bq2;
	Bulk_quote bq3(bq);
	Bulk_quote bq4 = bq;

	return 0;
}
```



```cpp
Quote : constructor taking 2 parameters
default constructing Quote
Quote: copy constructing
Quote: copy constructing
Quote : constructor taking 2 parameters
Bulk_quote : constructor taking 4 parameters
default constructing Quote
default constructing Bulk_quote
Quote: copy constructing
Bulk_quote : copy constructor
Quote: copy constructing
Bulk_quote : copy constructor
destructing Bulk_quote
destructing Quote
destructing Bulk_quote
destructing Quote
destructing Bulk_quote
destructing Quote
destructing Bulk_quote
destructing Quote
destructing Quote
destructing Quote
destructing Quote
destructing Quote
```



## 15.7.4 继承的构造函数

> 15.27 重新定义你的 `Bulk_quote` 类，令其继承构造函数。

```cpp
class Bulk_quote:public Disc_quote {
public:
	Bulk_quote(){ std::cout << "default constructing Bulk_quote\n";}
	// Bulk_quote(const std::string& b,double p,std::size_t q,double disc)
	// 	:Disc_quote(b,p,q,disc){
	// 		std::cout << "Bulk_quote : constructor taking 4 parameters\n";
	// 	}
	using Disc_quote::Disc_quote;
	Bulk_quote(const Bulk_quote& bq):Disc_quote(bq){
		std::cout << "Bulk_quote : copy constructor\n";
	}
	Bulk_quote(Bulk_quote&& bq)noexcept:Disc_quote(std::move(bq)){
		std::cout << "Bulk_quote : move constructor\n";
	}
	Bulk_quote& operator=(const Bulk_quote& rhs){
		Disc_quote::operator=(rhs);
		std::cout << "Bulk_quote : copy =()\n";
		return *this;
	}
	Bulk_quote& operator=(Bulk_quote&& rhs){
		Disc_quote::operator=(rhs);
		std::cout << "Bulk_quote : move =()\n";
		return *this;
	}
	~Bulk_quote()
	{
		std::cout << "destructing Bulk_quote\n";
	}
public:
	double net_price(std::size_t cnt)const override{//覆盖基类函数版本以实现基于大量购买的折扣政策
		if (cnt >= quantity) return cnt * (1-discount) * price;
		return cnt * price;
	}	

};
```



* 一个类只"继承"其直接基类的构造函数（非常常规方法，姑且算是继承），类不能继承默认、拷贝和移动构造函数（基本就是自己写的那些传参数的普通构造函数），如果类没有直接定义这些构造函数，则编译器将为派生类合成它们
* 派生类继承基类构造函数的方式是提供一条注明了直接基类名的`using`声明语句(注意是继承构造函数，赋值运算符不算)
	* 通常情况下`using`声明语句只是令某个名字在当前作用域内可见
	* 当`using`作用于构造函数时，会让编译器生产代码，对于基类的每个构造函数，都生成一个对应的派生类构造函数
	* 如果派生类有数据成员，那会被默认初始化

## 15.8 容器与继承

> 15.28 定义一个存放 `Quote` 对象的 `vector`，将 `Bulk_quote` 对象传入其中。计算 `vector` 中所有元素总的 `net_price`。

```cpp
#include "Quote.hpp"
#include <vector>

double print_total(std::ostream &os, const Quote &item, size_t n){
	double ret = item.net_price(n);
	os << "ISBN: " << item.isbn() << " # sold: " << n << " total due: " << ret;
	return  ret;
}

int main(){
	std::vector<Quote>v;
	for(int i = 0; i != 10; ++i){
		v.push_back(Bulk_quote("bq",15.4,5,0.3));
	}
	double total = 0.0;
	for(auto bq:v){
		total += bq.net_price(10);
	}
	std::cout << total << std::endl;

	return 0;
}
```



> 15.29 再运行一次你的程序，这次传入 `Quote` 对象的 `shared_ptr` 。如果这次计算出的总额与之前的不一致，解释为什么;如果一致，也请说明原因。

```cpp
#include "Quote.hpp"
#include <vector>
#include <memory>

double print_total(std::ostream &os, const Quote &item, size_t n){
	double ret = item.net_price(n);
	os << "ISBN: " << item.isbn() << " # sold: " << n << " total due: " << ret;
	return  ret;
}

int main(){
	std::vector<std::shared_ptr<Quote>>pv;
	for(int i = 0; i != 10; ++i){
		pv.push_back(std::make_shared<Bulk_quote>(Bulk_quote("bq",15.4,5,0.3)));
	}
	double total = 0.0;
	for(auto p:pv){
		total += p->net_price(10);
	}
	std::cout << total << std::endl;

	return 0;
}
```

* 当我们使用容器存放继承体系中的对象时，通常必须采取间接存储的方式。因为不允许在容器中保存不同类型的元素，所以我们不能把具有继承关系的多种类型对象直接存放到容器中
* 当派生类对象被赋值给基类对象时，**其中的派生类部分会被切掉**，因此容器和存在继承关系的类型无法兼容



### 15.8.1 编写 Basket 类

> 编写你自己的 `Basket` 类，用它计算上一个练习中交易记录的总价格。

增加虚拷贝函数

```cpp
#ifndef Quote_H
#define Quote_H
#include <cstddef>
#include <iostream>
#include <string>


class Quote{
public:
	Quote()=default;
	Quote(const std::string &book,double sales_price)
		:bookNo(book),price(sales_price){}
	Quote(const Quote& q):bookNo(q.bookNo),price(q.price){}
	Quote(Quote&& q)noexcept:bookNo(std::move(q.bookNo)),price(std::move(q.price)){}
	Quote& operator=(const Quote& rhs){
		if(this != &rhs){
			bookNo = rhs.bookNo;
			price = rhs.price;
		}
		return *this;
	}
	Quote& operator=(Quote&& rhs)noexcept{
		if(this != &rhs){
			bookNo = std::move(rhs.bookNo);
			price = std::move(rhs.price);
		}
		return *this;
	}

	virtual ~Quote() = default;

public:
	//该虚函数返回当前对象的一份动态分配的开被
	virtual Quote* clone()const &{return new Quote(*this);}
	virtual Quote* clone()&&{return new Quote(std::move(*this));}

	std::string isbn()const{return bookNo;}
	virtual double net_price(std::size_t n)const{return n * price;}
	virtual void debug()const{
		std::cout << "data members of this class:\n"
			<< "bookNo= " <<this->bookNo << " "
			<< "price= " <<this->price<< " ";
	}

private:
	std::string bookNo;	//ISBN编号
protected:
	double price = 0.0; //不打折的价格
};

class Disc_quote:public Quote{//抽象基类
public:
	Disc_quote() = default;
	Disc_quote(const std::string& b,double p,std::size_t q,double d)
		:Quote(b,p),quantity(q),discount(d){}
public:
	double net_price(std::size_t)const override = 0 ;
	void debug()const override {
		std::cout << "data members of this class:\n"
			<< "bookNo= " <<this->quantity << " "
			<< "price= " <<this->discount<< " ";
	}
protected:
	std::size_t quantity = 0;//折扣适用的购买量
    double	discount = 0.0;//表示折扣的小数值
};

class Bulk_quote:public Disc_quote {
public:
	Bulk_quote()=default;
	Bulk_quote(const std::string& b,double p,std::size_t q,double disc)
		:Disc_quote(b,p,q,disc){}
	Bulk_quote(const Bulk_quote& bq)=default;
	Bulk_quote(Bulk_quote&& bq)=default;
	Bulk_quote& operator=(const Bulk_quote& rhs)=default;
	Bulk_quote& operator=(Bulk_quote&& rhs)=default;
	~Bulk_quote() = default;
public:
	Bulk_quote* clone()const& override{return new Bulk_quote(*this);}
	Bulk_quote* clone()&& override{return new Bulk_quote(std::move(*this));}
	double net_price(std::size_t cnt)const override{//覆盖基类函数版本以实现基于大量购买的折扣政策
		if (cnt >= quantity) return cnt * (1-discount) * price;
		return cnt * price;
	}	

};

class Limit_quote:public Disc_quote {
public:
	Limit_quote() = default;
	Limit_quote(const std::string& b,double p,std::size_t q,double disc)
		:Disc_quote(b,p,q,disc){}
public:
	double net_price(std::size_t cnt)const override{
		if(cnt <= quantity)return cnt * (1-discount) * price;
		return quantity * (1-discount) * price + (cnt-quantity) * price;
	}
};

inline double print_total(std::ostream &os, const Quote &item, size_t n){
	double ret = item.net_price(n);
	os << "ISBN: " << item.isbn() << " # sold: " << n << " total due: " << ret;
	return  ret;
}
#endif
```



```cpp
#ifndef Basket_H
#define Basket_H

#include <memory>
#include <iostream>
#include <set>
#include "Quote.hpp"

class Basket{
public:
	//使用合成的默认构造函数和拷贝控制成员

	void add_item(const Quote& sale){
		items.insert(std::shared_ptr<Quote>(sale.clone()));
	}
	void add_item(Quote && sale){
		items.insert(std::shared_ptr<Quote>(std::move(sale).clone()));
	}

	//打印每本书的总价和购物篮里所有书的总价
	double total_receipt(std::ostream& os)const{
		double sum = 0.0;
		for(auto iter = items.cbegin();iter != items.cend();iter = items.upper_bound(*iter)){//会跳过关键字相同的
			sum += print_total(os, **iter, items.count(*iter));
			//*iter 得到一个指向准备打印对象的shared_ptr
			//**iter 是个Quote对象
		}
		os << "Total Sale: " << sum << std::endl;
		return sum;
	}
private:
	// 用于比较 shared_ptr,multiset会用到
	static bool compare(const std::shared_ptr<Quote> &lhs,
						const std::shared_ptr<Quote> &rhs){
							return lhs->isbn() < rhs->isbn();
						}
	std::multiset<std::shared_ptr<Quote>,decltype(compare)*>items{compare};
	//multiset 保存多个报价，按compare成员排序
};

#endif
```



```cpp
#include "Quote.hpp"
#include "Basket.hpp"
#include <vector>
#include <memory>


int main(){
	Basket basket;

	for (unsigned i = 0; i != 10; ++i)
		basket.add_item(Bulk_quote("Bible", 20.6, 20, 0.3));

	for (unsigned i = 0; i != 10; ++i)
		basket.add_item(Bulk_quote("C++Primer", 30.9, 5, 0.4));

	for (unsigned i = 0; i != 10; ++i)
		basket.add_item(Quote("CLRS", 40.1));

	basket.total_receipt(std::cout);
	return 0;

	return 0;
}
```



## 15.9 文本查询程序再探

### 15.9.1 面向对象的解决方案

> 15.31 已知 `s1`、`s2`、`s3` 和 `s4` 都是 `string`，判断下面的表达式分别创建了什么样的对象：

```cpp
(a) Query(s1) | Query(s2) & ~Query(s3);//返回一个Query 绑定到 AndQuery
(b) Query(s1) | (Query(s2) & ~Query(s3));//返回一个Query 绑定到 OrQuery
(c) (Query(s1) & (Query(s2)) | (Query(s3) & Query(s4)));//返回一个Query 绑定到 OrQuery
```



### 15.9.2 Query_base 类和 Query 类

> 15.32 当一个 `Query` 类型的对象被拷贝、移动、赋值或销毁时，将分别发生什么？

会使用合成的版本进行拷贝、移动、赋值和销毁

拷贝：智能指针计数器+1

移动：新对象智能指针指向原对象地址，原对象指针为`nullptr`，新对象智能指针计数器为1

赋值：和拷贝差不多

销毁：智能指针计数器-1，如果为0,释放内存

> 15.33 当一个 `Query_base` 类型的对象被拷贝、移动赋值或销毁时，将分别发生什么？

`Query_base`是抽象基类，所以实际对象是派生类对象



### 15.9.3  派生类

> 15.34 针对图15.3构建的表达式：

```cpp
(a) 例举出在处理表达式的过程中执行的所有构造函数。
(b) 例举出 cout << q 所调用的 rep。
(c) 例举出 q.eval() 所调用的 eval。
```



> 15.35 实现 `Query` 类和 `Query_base` 类，其中需要定义`rep` 而无须定义 `eval`。



> 15.36 在构造函数和 `rep` 成员中添加打印语句，运行你的代码以检验你对本节第一个练习中(a)、(b)两小题的回答是否正确。



> 15.37 如果在派生类中含有 `shared_ptr<Query_base>` 类型的成员而非 `Query` 类型的成员，则你的类需要做出怎样的改变？



> 15.38 下面的声明合法吗？如果不合法，请解释原因;如果合法，请指出该声明的含义。

```cpp
BinaryQuery a = Query("fiery") & Query("bird");//不合法，BinaryQuery是抽象类
AndQuery b = Query("fiery") & Query("bird");//不合法&返回Query，不能转为AndQuery
OrQuery c = Query("fiery") & Query("bird");//不合法&返回Query，不能转为OrQuery
```

* 不存在从基类向派生类的隐式转换
* 我们不能直接创建一个抽象基类的对象
