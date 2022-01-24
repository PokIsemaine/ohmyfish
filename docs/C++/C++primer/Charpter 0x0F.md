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



> 15.20 编写代码检验你对前面两题的回答是否正确。



> 15.21从下面这些一般性抽象概念中任选一个（或者选一个你自己的），将其对应的一组类型组织成一个继承体系：

```cpp
(a) 图形文件格式（如gif、tiff、jpeg、bmp）
(b) 图形基元（如方格、圆、球、圆锥）
(c) C++语言中的类型（如类、函数、成员函数）
```



> 15.22 对于你在上一题中选择的类，为其添加函数的虚函数及公有成员和受保护的成员。



## 15.6 继承中的类作用域
