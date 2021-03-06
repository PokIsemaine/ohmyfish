<h1 align="center">ð C++ Primer 0x0F ç»ä¹ é¢è§£</h1>

## 15.2 å®ä¹åºç±»åæ´¾çç±»

### 15.2.1 å®ä¹åºç±»

> 15.1 ä»ä¹æ¯èæå

éè¿å³é®å­`virtural`å¯ä»¥å°æåå£°æä¸ºèæå

èå½æ°ç¨æ¥åºååºç±»éå¸ææ´¾çç±»è¿è¡è¦ççå½æ°åä¸å¸ææ´¾çç±»è¦ççå½æ°



> 15.2 `protected` è®¿é®è¯´æç¬¦ä¸ `private` æä½åºå«ï¼

* `protected`åè®¸æ´¾çç±»ãåºç±»ãååè®¿é®ï¼ç¦æ­¢å¶ä»ç¨æ·è®¿é®
* `private`åªåè®¸åºç±»æ¬èº«åååè®¿é®





> 15.3 å®ä¹ä½ èªå·±ç `Quote` ç±»å `print_total` å½æ°ã

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
	virtual ~Quote() = default;	//å¯¹ææå½æ°å¨æç»å®
private:
	std::string bookNo;	//ISBNç¼å·
protected:
	double price = 0.0; //ä¸ææçä»·æ ¼
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



### 15.2.2 å®ä¹æ´¾çç±»

> 15.4 ä¸é¢åªæ¡å£°æè¯­å¥æ¯ä¸æ­£ç¡®çï¼è¯·è§£éåå ã

```cpp
class Base { ... };
(a) class Derived : public Derived { ... };//ä¸è½æ´¾çèªå·±
(b) class Derived : private Base { ... };//è¿æ¯å®ä¹
(c) class Derived : public Base;//å£°æä¸åå«æ´¾çåè¡¨
```



> 15.5 å®ä¹ä½ èªå·±ç `Bulk_quote` ç±»ã

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
	virtual ~Quote() = default;	//å¯¹ææå½æ°å¨æç»å®

public:
	virtual double net_price(std::size_t n)const{return n * price;}

private:
	std::string bookNo;	//ISBNç¼å·
protected:
	double price = 0.0; //ä¸ææçä»·æ ¼
};

class Bulk_quote:public Quote {
public:
	Bulk_quote() = default;
	Bulk_quote(const std::string& b,double p,std::size_t q,double disc)
		:Quote(b,p),min_qty(q),discount(disc){}
public:
	double net_price(std::size_t)const override;	//è¦çåºç±»å½æ°çæ¬ä»¥å®ç°åºäºå¤§éè´­ä¹°çææ£æ¿ç­

private:
	std::size_t min_qty = 0;	//éç¨ææ£æ¿ç­çæä½è´­ä¹°é
	double discount = 0.0;	//ä»¥å°æ°è¡¨ç¤ºçææ£é¢
};

double print_total(std::ostream &os, const Quote &item, size_t n);
```



> 15.6 å° `Quote` å `Bulk_quote` çå¯¹è±¡ä¼ ç»15.2.1èç»ä¹ ä¸­ç `print_total` å½æ°ï¼æ£æ¥è¯¥å½æ°æ¯å¦æ­£ç¡®ã

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
	virtual ~Quote() = default;	//å¯¹ææå½æ°å¨æç»å®

public:
	virtual double net_price(std::size_t n)const{return n * price;}

private:
	std::string bookNo;	//ISBNç¼å·
protected:
	double price = 0.0; //ä¸ææçä»·æ ¼
};

class Bulk_quote:public Quote {
public:
	Bulk_quote() = default;
	Bulk_quote(const std::string& b,double p,std::size_t q,double disc)
		:Quote(b,p),min_qty(q),discount(disc){}
public:
	double net_price(std::size_t cnt)const override{//è¦çåºç±»å½æ°çæ¬ä»¥å®ç°åºäºå¤§éè´­ä¹°çææ£æ¿ç­
		if (cnt >= min_qty) return cnt * (1-discount) * price;
		else return cnt * price;
	}	

private:
	std::size_t min_qty = 0;	//éç¨ææ£æ¿ç­çæä½è´­ä¹°é
	double discount = 0.0;	//ä»¥å°æ°è¡¨ç¤ºçææ£é¢
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



> 15.7 å®ä¹ä¸ä¸ªç±»ä½¿å¶å®ç°ä¸ç§æ°éåéçææ£ç­ç¥ï¼å·ä½ç­ç¥æ¯ï¼å½è´­ä¹°ä¹¦ç±çæ°éä¸è¶è¿ä¸ä¸ªç»å®çééæ¶äº«åææ£ï¼å¦æè´­ä¹°éä¸æ¦è¶è¿äºééï¼åè¶åºçé¨åå°ä»¥åä»·éå®ã

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
	double discount = 0.0;	//ä»¥å°æ°è¡¨ç¤ºçææ£é¢

};
```



### 15.2.3 ç±»åè½¬æ¢ä¸ç»§æ¿

> 15.8 ç»åºéæç±»ååå¨æç±»åçå®ä¹ã

* è¡¨è¾¾å¼çéæç±»åæ¯å¨ç¼è¯æ¶å·²ç¥çï¼å®æ¯åéå£°ææ¶çç±»åæè¡¨è¾¾å¼çæçç±»å
* å¨æç±»åæ¯åéæè¡¨è¾¾å¼è¡¨ç¤ºçåå­ä¸­çå¯¹è±¡çç±»åï¼ç´å°è¿è¡æ¶æå¯ç¥



> 15.9 å¨ä»ä¹æåµä¸è¡¨è¾¾å¼çéæç±»åå¯è½ä¸å¨æç±»åä¸åï¼è¯·ç»åºä¸ä¸ªéæç±»åä¸å¨æç±»åä¸åçä¾å­ã

åºç±»çæéæå¼ç¨çéæç±»åä¸å¨æç±»åå¯è½ä¸ä¸è´



> 15.10 åå¿æä»¬å¨8.1èè¿è¡çè®¨è®ºï¼è§£éç¬¬284é¡µä¸­å° `ifstream` ä¼ éç» `Sales_data` ç`read` å½æ°çç¨åºæ¯å¦ä½å·¥ä½çã

å¨è¦æ±ä½¿ç¨åºç±»åå¯¹è±¡çå°æ¹ï¼æä»¬å¯ä»¥ç¨ç»§æ¿ç±»åçå¯¹è±¡æ¥ä»£æ¿

`std::ifstream`æ¯`std::istream`çæ´¾çç±»ï¼æä»¥`read`ä¼ å¥`std::ifstream`ä¹æ¯å¯ä»¥ç



## 15.3 èå½æ°

> 15.11 ä¸ºä½ ç `Quote` ç±»ä½ç³»æ·»å ä¸ä¸ªåä¸º `debug` çèå½æ°ï¼ä»¤å¶åå«æ¾ç¤ºæ¯ä¸ªç±»çæ°æ®æåã

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
	virtual ~Quote() = default;	//å¯¹ææå½æ°å¨æç»å®

public:
	virtual double net_price(std::size_t n)const{return n * price;}
	virtual void debug()const{
		std::cout << "data members of this class:\n"
			<< "bookNo= " <<this->bookNo << " "
			<< "price= " <<this->price<< " ";
	}

private:
	std::string bookNo;	//ISBNç¼å·
protected:
	double price = 0.0; //ä¸ææçä»·æ ¼
};

class Bulk_quote:public Quote {
public:
	Bulk_quote() = default;
	Bulk_quote(const std::string& b,double p,std::size_t q,double disc)
		:Quote(b,p),min_qty(q),discount(disc){}
public:
	double net_price(std::size_t cnt)const override{//è¦çåºç±»å½æ°çæ¬ä»¥å®ç°åºäºå¤§éè´­ä¹°çææ£æ¿ç­
		if (cnt >= min_qty) return cnt * (1-discount) * price;
		return cnt * price;
	}	
	void debug()const override {
		std::cout << "data members of this class:\n"
			<< "bookNo= " <<this->min_qty << " "
			<< "price= " <<this->discount<< " ";
	}
private:
	std::size_t min_qty = 0;	//éç¨ææ£æ¿ç­çæä½è´­ä¹°é
	double discount = 0.0;	//ä»¥å°æ°è¡¨ç¤ºçææ£é¢
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
	double discount = 0.0;	//ä»¥å°æ°è¡¨ç¤ºçææ£é¢

};

double print_total(std::ostream &os, const Quote &item, size_t n);
```





> 15.12 æå¿è¦å°ä¸ä¸ªæåå½æ°åæ¶å£°ææ `override` å `final` åï¼ä¸ºä»ä¹ï¼

æå¿è¦

* ä½¿ç¨`override`å¯ä»¥è®©æå¾æ´å æ¸æ°ï¼å¦æä½¿ç¨`override`æ è®°äºæä¸ªå½æ°ï¼ä½è¯¥å½æ°æ²¡æè¦çå·²å­å¨åºç±»çèå½æ°ï¼ç¼è¯å¨ä¼æ¥é

* `final`å³é®å­ç¨æ¥é²æ­¢ç»§æ¿ï¼ç¦æ­¢äºæ´¾çç±»è¦çè¯¥å½æ°



> 15.13 ç»å®ä¸é¢çç±»ï¼è§£éæ¯ä¸ª `print` å½æ°çæºçï¼

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

å¨ä¸è¿°ä»£ç ä¸­å­å¨é®é¢åï¼å¦ææï¼ä½ è¯¥å¦ä½ä¿®æ¹å®ï¼



å¦æä¸ä¸ªæ´¾çç±»èå½æ°éè¦è°ç¨å®çåºç±»çæ¬ï¼ä½æ²¡æä½¿ç¨ä½ç¨äºè¿ç®ç¬¦ï¼åè¿è¡æ¶è¯¥è°ç¨å°è¢«è§£æä¸ºå¯¹æ´¾çç±»çæ¬èªèº«çè°ç¨ï¼ä»èå¯¼è´æ ééå½

é¡ºä¾¿å ä¸`override`è¿æ ·æ´æç¡®æå¾

æ¹ä¸º

```cpp
public:
	void print(ostream &os)override { base::print(os); os << " " << i; }
```



> 15.14 ç»å®ä¸ä¸é¢ä¸­çç±»ä»¥åä¸é¢è¿äºå¯¹è±¡ï¼è¯´æå¨è¿è¡æ¶è°ç¨åªä¸ªå½æ°ï¼

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



## 15.4 æ½è±¡åºç±»

> 15.15 å®ä¹ä½ èªå·±ç `Disc_quote` å `Bulk_quote`ã

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
	virtual ~Quote() = default;	//å¯¹ææå½æ°å¨æç»å®

public:
	virtual double net_price(std::size_t n)const{return n * price;}
	virtual void debug()const{
		std::cout << "data members of this class:\n"
			<< "bookNo= " <<this->bookNo << " "
			<< "price= " <<this->price<< " ";
	}

private:
	std::string bookNo;	//ISBNç¼å·
protected:
	double price = 0.0; //ä¸ææçä»·æ ¼
};

class Disc_quote:public Quote{//æ½è±¡åºç±»
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
	std::size_t quantity = 0;//ææ£éç¨çè´­ä¹°é
    double	discount = 0.0;//è¡¨ç¤ºææ£çå°æ°å¼
};

class Bulk_quote:public Disc_quote {
public:
	Bulk_quote() = default;
	Bulk_quote(const std::string& b,double p,std::size_t q,double disc)
		:Disc_quote(b,p,q,disc){}
public:
	double net_price(std::size_t cnt)const override{//è¦çåºç±»å½æ°çæ¬ä»¥å®ç°åºäºå¤§éè´­ä¹°çææ£æ¿ç­
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



> 15.16 æ¹åä½ å¨15.2.2èç»ä¹ ä¸­ç¼åçæ°éåéçææ£ç­ç¥ï¼ä»¤å¶ç»§æ¿ `Disc_quote`ã

è§`15.15`



> 15.17 å°è¯å®ä¹ä¸ä¸ª `Disc_quote` çå¯¹è±¡ï¼ççç¼è¯å¨ç»åºçéè¯¯ä¿¡æ¯æ¯ä»ä¹ï¼

`variable type 'Disc_quote' is an abstract class`



## 15.5 è®¿é®æ§å¶ä¸ç»§æ¿

> 15.18 åè®¾ç»å®äºç¬¬543é¡µåç¬¬544é¡µçç±»ï¼åæ¶å·²ç¥æ¯ä¸ªå¯¹è±¡çç±»åå¦æ³¨éæç¤ºï¼å¤æ­ä¸é¢çåªäºèµå¼è¯­å¥æ¯åæ³çãè§£éé£äºä¸åæ³çè¯­å¥ä¸ºä»ä¹ä¸è¢«åè®¸ï¼

```cpp
Base *p = &d1;  //d1 çç±»åæ¯ Pub_Derv åæ³
p = &d2;		//d2 çç±»åæ¯ Priv_Derv
p = &d3;		//d3 çç±»åæ¯ Prot_Derv
p = &dd1;		//dd1 çç±»åæ¯ Derived_from_Public	åæ³
p = &dd2;		//dd2 çç±»åæ¯ Derived_from_Private
p = &dd3;		//dd3 çç±»åæ¯ Derived_from_Protected
```

åªæå½æ´¾çç±»æ¯`public`ç»§æ¿æ¶ï¼ç¨æ·ä»£ç æå¯ä»¥ä½¿ç¨æ´¾çç±»å°åºç±»çè½¬æ¢



> 15.19 åè®¾543é¡µå544é¡µçæ¯ä¸ªç±»é½æå¦ä¸å½¢å¼çæåå½æ°ï¼

```cpp
void memfcn(Base &b) { b = *this; }
```

å¯¹äºæ¯ä¸ªç±»ï¼åå«å¤æ­ä¸é¢çå½æ°æ¯å¦åæ³ã

* ä¸è®ºDä»¥ä»ä¹æ¹å¼ç»§æ¿Bï¼Dçæåå½æ°åååé½è½ä½¿ç¨æ´¾çç±»ååºç±»çè½¬æ¢ï¼æ´¾çç±»åå¶ç´æ¥åºç±»çç±»åè½¬æ¢å¯¹äºæ´¾çç±»çæååååæ°¸è¿é½æ¯å¯è®¿é®ç

* `Pub_Derv`åæ³
* `Priv_Derv`åæ³
* `Prot_Derv`åæ³

* å¦æDç»§æ¿Bçæ¹å¼æ¯å¬æçæåä¿æ¤çï¼å**Dçæ´¾çç±»**çæååååå¯ä»¥ä½¿ç¨DåBçç±»åè½¬æ¢ï¼åä¹ä¸è¡

* `Derived_from_Public`åæ³
*  `Derived_from_Private`éæ³
* `Derived_from_Protected`åæ³

> 15.20 ç¼åä»£ç æ£éªä½ å¯¹åé¢ä¸¤é¢çåç­æ¯å¦æ­£ç¡®ã



> 15.21ä»ä¸é¢è¿äºä¸è¬æ§æ½è±¡æ¦å¿µä¸­ä»»éä¸ä¸ªï¼æèéä¸ä¸ªä½ èªå·±çï¼ï¼å°å¶å¯¹åºçä¸ç»ç±»åç»ç»æä¸ä¸ªç»§æ¿ä½ç³»ï¼

```cpp
(a) å¾å½¢æä»¶æ ¼å¼ï¼å¦gifãtiffãjpegãbmpï¼
(b) å¾å½¢åºåï¼å¦æ¹æ ¼ãåãçãåé¥ï¼
(c) C++è¯­è¨ä¸­çç±»åï¼å¦ç±»ãå½æ°ãæåå½æ°ï¼
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



> 15.22 å¯¹äºä½ å¨ä¸ä¸é¢ä¸­éæ©çç±»ï¼ä¸ºå¶æ·»å å½æ°çèå½æ°åå¬ææåååä¿æ¤çæåã

è§`15.21`

## 15.6 ç»§æ¿ä¸­çç±»ä½ç¨å

> 15.23 åè®¾ç¬¬550é¡µç `D1` ç±»éè¦è¦çå®ç»§æ¿èæ¥ç `fcn` å½æ°ï¼ä½ åºè¯¥å¦ä½å¯¹å¶è¿è¡ä¿®æ¹ï¼å¦æä½ ä¿®æ¹ä¹å `fcn` å¹éäº `Base` ä¸­çå®ä¹ï¼åè¯¥èçé£äºè°ç¨è¯­å¥å°å¦ä½è§£æï¼

å»æ`int`

ä¸ä¸ªæ´¾çç±»å¦æè¦çäºæä¸ªç»§æ¿æ¥çèå½æ°ï¼å®çå½¢åç±»åå¿é¡»è¦ä¸è¢«è¦ççåºç±»å½æ°å®å¨ä¸è´

## 15.7 æé å½æ°ä¸æ·è´æ§å¶

### 15.7.1 èææå½æ°

> 15.24 åªç§ç±»éè¦èææå½æ°ï¼èææå½æ°å¿é¡»æ§è¡ä»ä¹æ ·çæä½ï¼

åºç±»

* åªè¦åºç±»çææå½æ°æ¯èå½æ°ï¼å°±è½ç¡®ä¿æä»¬`delete`åºç±»æéæ¶å°è¿è¡æ­£ç¡®çææå½æ°çæ¬
* å¦æåºç±»çææå½æ°ä¸æ¯èå½æ°ï¼å`delete`ä¸ä¸ªæåæ´¾çç±»å¯¹è±¡çåºç±»æéå°äº§çæªå®ä¹çè¡ä¸º
* åºç±»çèææå½æ°ï¼å¹¶ä¸åæ®éç±»çä¸æ ·ï¼æä»¬ä¸è½å ä¸ºææå½æ°æ¨æ­åºè¯¥ç±»æ¯å¦è¿éè¦èµå¼è¿ç®ç¬¦åæ·è´æé å½æ°ãå ä¸ºä¸ºäºæä¸ºèå½æ°ï¼å®çåå®¹æ¯ç©ºç



### 15.7.2 åææ·è´æ§å¶ä¸ç»§æ¿

> 15.25 æä»¬ä¸ºä»ä¹ä¸º `Disc_quote` å®ä¹ä¸ä¸ªé»è®¤æé å½æ°ï¼å¦æå»æè¯¥æé å½æ°çè¯ä¼å¯¹ `Bulk_quote` çè¡ä¸ºäº§çä»ä¹å½±åï¼

å ä¸ºå¤§é¨åæåµä¸ä¸åçç±»å®ä¹äºä¸åçæé å½æ°ï¼é£ä¹å°±ä¸ä¼èªå¨åæé»è®¤æé å½æ°äºï¼éè¦èªå·±å®ä¹

`Bulk_quote`çé»è®¤æé å½æ°è¿è¡`Disc_quote`çé»è®¤æé å½æ°ï¼åèåè¿è¡`Quote`çé»è®¤æé å½æ°ï¼å¦æ`Disc_quote`æ²¡æé»è®¤æé å½æ°ï¼é£`Bulk_quote`ä¹æ²¡æ³å®æåå§åå·¥ä½



### 15.7.3 æ´¾çç±»çæ·è´æ§å¶æå

> 15.26 å®ä¹ `Quote` å `Bulk_quote` çæ·è´æ§å¶æåï¼ä»¤å¶ä¸åæççæ¬è¡ä¸ºä¸è´ãä¸ºè¿äºæåä»¥åå¶ä»æé å½æ°æ·»å æå°ç¶æçè¯­å¥ï¼ä½¿å¾æä»¬è½å¤ç¥éæ­£å¨è¿è¡åªä¸ªç¨åºãä½¿ç¨è¿äºç±»ç¼åç¨åºï¼é¢æµç¨åºå°åå»ºåéæ¯åªäºå¯¹è±¡ãéå¤å®éªï¼ä¸æ­æ¯è¾ä½ çé¢æµåå®éè¾åºç»ææ¯å¦ç¸åï¼ç´å°é¢æµå®å¨åç¡®åç»æã

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
	std::string bookNo;	//ISBNç¼å·
protected:
	double price = 0.0; //ä¸ææçä»·æ ¼
};

class Disc_quote:public Quote{//æ½è±¡åºç±»
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
	std::size_t quantity = 0;//ææ£éç¨çè´­ä¹°é
    double	discount = 0.0;//è¡¨ç¤ºææ£çå°æ°å¼
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
	double net_price(std::size_t cnt)const override{//è¦çåºç±»å½æ°çæ¬ä»¥å®ç°åºäºå¤§éè´­ä¹°çææ£æ¿ç­
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



## 15.7.4 ç»§æ¿çæé å½æ°

> 15.27 éæ°å®ä¹ä½ ç `Bulk_quote` ç±»ï¼ä»¤å¶ç»§æ¿æé å½æ°ã

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
	double net_price(std::size_t cnt)const override{//è¦çåºç±»å½æ°çæ¬ä»¥å®ç°åºäºå¤§éè´­ä¹°çææ£æ¿ç­
		if (cnt >= quantity) return cnt * (1-discount) * price;
		return cnt * price;
	}	

};
```



* ä¸ä¸ªç±»åª"ç»§æ¿"å¶ç´æ¥åºç±»çæé å½æ°ï¼éå¸¸å¸¸è§æ¹æ³ï¼å§ä¸ç®æ¯ç»§æ¿ï¼ï¼ç±»ä¸è½ç»§æ¿é»è®¤ãæ·è´åç§»å¨æé å½æ°ï¼åºæ¬å°±æ¯èªå·±åçé£äºä¼ åæ°çæ®éæé å½æ°ï¼ï¼å¦æç±»æ²¡æç´æ¥å®ä¹è¿äºæé å½æ°ï¼åç¼è¯å¨å°ä¸ºæ´¾çç±»åæå®ä»¬
* æ´¾çç±»ç»§æ¿åºç±»æé å½æ°çæ¹å¼æ¯æä¾ä¸æ¡æ³¨æäºç´æ¥åºç±»åç`using`å£°æè¯­å¥(æ³¨ææ¯ç»§æ¿æé å½æ°ï¼èµå¼è¿ç®ç¬¦ä¸ç®)
	* éå¸¸æåµä¸`using`å£°æè¯­å¥åªæ¯ä»¤æä¸ªåå­å¨å½åä½ç¨ååå¯è§
	* å½`using`ä½ç¨äºæé å½æ°æ¶ï¼ä¼è®©ç¼è¯å¨çäº§ä»£ç ï¼å¯¹äºåºç±»çæ¯ä¸ªæé å½æ°ï¼é½çæä¸ä¸ªå¯¹åºçæ´¾çç±»æé å½æ°
	* å¦ææ´¾çç±»ææ°æ®æåï¼é£ä¼è¢«é»è®¤åå§å

## 15.8 å®¹å¨ä¸ç»§æ¿

> 15.28 å®ä¹ä¸ä¸ªå­æ¾ `Quote` å¯¹è±¡ç `vector`ï¼å° `Bulk_quote` å¯¹è±¡ä¼ å¥å¶ä¸­ãè®¡ç® `vector` ä¸­ææåç´ æ»ç `net_price`ã

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



> 15.29 åè¿è¡ä¸æ¬¡ä½ çç¨åºï¼è¿æ¬¡ä¼ å¥ `Quote` å¯¹è±¡ç `shared_ptr` ãå¦æè¿æ¬¡è®¡ç®åºçæ»é¢ä¸ä¹åçä¸ä¸è´ï¼è§£éä¸ºä»ä¹;å¦æä¸è´ï¼ä¹è¯·è¯´æåå ã

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

* å½æä»¬ä½¿ç¨å®¹å¨å­æ¾ç»§æ¿ä½ç³»ä¸­çå¯¹è±¡æ¶ï¼éå¸¸å¿é¡»éåé´æ¥å­å¨çæ¹å¼ãå ä¸ºä¸åè®¸å¨å®¹å¨ä¸­ä¿å­ä¸åç±»åçåç´ ï¼æä»¥æä»¬ä¸è½æå·æç»§æ¿å³ç³»çå¤ç§ç±»åå¯¹è±¡ç´æ¥å­æ¾å°å®¹å¨ä¸­
* å½æ´¾çç±»å¯¹è±¡è¢«èµå¼ç»åºç±»å¯¹è±¡æ¶ï¼**å¶ä¸­çæ´¾çç±»é¨åä¼è¢«åæ**ï¼å æ­¤å®¹å¨åå­å¨ç»§æ¿å³ç³»çç±»åæ æ³å¼å®¹



### 15.8.1 ç¼å Basket ç±»

> ç¼åä½ èªå·±ç `Basket` ç±»ï¼ç¨å®è®¡ç®ä¸ä¸ä¸ªç»ä¹ ä¸­äº¤æè®°å½çæ»ä»·æ ¼ã

å¢å èæ·è´å½æ°

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
	//è¯¥èå½æ°è¿åå½åå¯¹è±¡çä¸ä»½å¨æåéçå¼è¢«
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
	std::string bookNo;	//ISBNç¼å·
protected:
	double price = 0.0; //ä¸ææçä»·æ ¼
};

class Disc_quote:public Quote{//æ½è±¡åºç±»
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
	std::size_t quantity = 0;//ææ£éç¨çè´­ä¹°é
    double	discount = 0.0;//è¡¨ç¤ºææ£çå°æ°å¼
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
	double net_price(std::size_t cnt)const override{//è¦çåºç±»å½æ°çæ¬ä»¥å®ç°åºäºå¤§éè´­ä¹°çææ£æ¿ç­
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
	//ä½¿ç¨åæçé»è®¤æé å½æ°åæ·è´æ§å¶æå

	void add_item(const Quote& sale){
		items.insert(std::shared_ptr<Quote>(sale.clone()));
	}
	void add_item(Quote && sale){
		items.insert(std::shared_ptr<Quote>(std::move(sale).clone()));
	}

	//æå°æ¯æ¬ä¹¦çæ»ä»·åè´­ç©ç¯®éææä¹¦çæ»ä»·
	double total_receipt(std::ostream& os)const{
		double sum = 0.0;
		for(auto iter = items.cbegin();iter != items.cend();iter = items.upper_bound(*iter)){//ä¼è·³è¿å³é®å­ç¸åç
			sum += print_total(os, **iter, items.count(*iter));
			//*iter å¾å°ä¸ä¸ªæååå¤æå°å¯¹è±¡çshared_ptr
			//**iter æ¯ä¸ªQuoteå¯¹è±¡
		}
		os << "Total Sale: " << sum << std::endl;
		return sum;
	}
private:
	// ç¨äºæ¯è¾ shared_ptr,multisetä¼ç¨å°
	static bool compare(const std::shared_ptr<Quote> &lhs,
						const std::shared_ptr<Quote> &rhs){
							return lhs->isbn() < rhs->isbn();
						}
	std::multiset<std::shared_ptr<Quote>,decltype(compare)*>items{compare};
	//multiset ä¿å­å¤ä¸ªæ¥ä»·ï¼æcompareæåæåº
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



## 15.9 ææ¬æ¥è¯¢ç¨åºåæ¢

### 15.9.1 é¢åå¯¹è±¡çè§£å³æ¹æ¡

> 15.31 å·²ç¥ `s1`ã`s2`ã`s3` å `s4` é½æ¯ `string`ï¼å¤æ­ä¸é¢çè¡¨è¾¾å¼åå«åå»ºäºä»ä¹æ ·çå¯¹è±¡ï¼

```cpp
(a) Query(s1) | Query(s2) & ~Query(s3);//è¿åä¸ä¸ªQuery ç»å®å° AndQuery
(b) Query(s1) | (Query(s2) & ~Query(s3));//è¿åä¸ä¸ªQuery ç»å®å° OrQuery
(c) (Query(s1) & (Query(s2)) | (Query(s3) & Query(s4)));//è¿åä¸ä¸ªQuery ç»å®å° OrQuery
```



### 15.9.2 Query_base ç±»å Query ç±»

> 15.32 å½ä¸ä¸ª `Query` ç±»åçå¯¹è±¡è¢«æ·è´ãç§»å¨ãèµå¼æéæ¯æ¶ï¼å°åå«åçä»ä¹ï¼

ä¼ä½¿ç¨åæççæ¬è¿è¡æ·è´ãç§»å¨ãèµå¼åéæ¯

æ·è´ï¼æºè½æéè®¡æ°å¨+1

ç§»å¨ï¼æ°å¯¹è±¡æºè½æéæååå¯¹è±¡å°åï¼åå¯¹è±¡æéä¸º`nullptr`ï¼æ°å¯¹è±¡æºè½æéè®¡æ°å¨ä¸º1

èµå¼ï¼åæ·è´å·®ä¸å¤

éæ¯ï¼æºè½æéè®¡æ°å¨-1ï¼å¦æä¸º0,éæ¾åå­

> 15.33 å½ä¸ä¸ª `Query_base` ç±»åçå¯¹è±¡è¢«æ·è´ãç§»å¨èµå¼æéæ¯æ¶ï¼å°åå«åçä»ä¹ï¼

`Query_base`æ¯æ½è±¡åºç±»ï¼æä»¥å®éå¯¹è±¡æ¯æ´¾çç±»å¯¹è±¡



### 15.9.3  æ´¾çç±»

> 15.34 éå¯¹å¾15.3æå»ºçè¡¨è¾¾å¼ï¼

```cpp
(a) ä¾ä¸¾åºå¨å¤çè¡¨è¾¾å¼çè¿ç¨ä¸­æ§è¡çæææé å½æ°ã
(b) ä¾ä¸¾åº cout << q æè°ç¨ç repã
(c) ä¾ä¸¾åº q.eval() æè°ç¨ç evalã
```



> 15.35 å®ç° `Query` ç±»å `Query_base` ç±»ï¼å¶ä¸­éè¦å®ä¹`rep` èæ é¡»å®ä¹ `eval`ã



> 15.36 å¨æé å½æ°å `rep` æåä¸­æ·»å æå°è¯­å¥ï¼è¿è¡ä½ çä»£ç ä»¥æ£éªä½ å¯¹æ¬èç¬¬ä¸ä¸ªç»ä¹ ä¸­(a)ã(b)ä¸¤å°é¢çåç­æ¯å¦æ­£ç¡®ã



> 15.37 å¦æå¨æ´¾çç±»ä¸­å«æ `shared_ptr<Query_base>` ç±»åçæåèé `Query` ç±»åçæåï¼åä½ çç±»éè¦ååºææ ·çæ¹åï¼



> 15.38 ä¸é¢çå£°æåæ³åï¼å¦æä¸åæ³ï¼è¯·è§£éåå ;å¦æåæ³ï¼è¯·æåºè¯¥å£°æçå«ä¹ã

```cpp
BinaryQuery a = Query("fiery") & Query("bird");//ä¸åæ³ï¼BinaryQueryæ¯æ½è±¡ç±»
AndQuery b = Query("fiery") & Query("bird");//ä¸åæ³&è¿åQueryï¼ä¸è½è½¬ä¸ºAndQuery
OrQuery c = Query("fiery") & Query("bird");//ä¸åæ³&è¿åQueryï¼ä¸è½è½¬ä¸ºOrQuery
```

* ä¸å­å¨ä»åºç±»åæ´¾çç±»çéå¼è½¬æ¢
* æä»¬ä¸è½ç´æ¥åå»ºä¸ä¸ªæ½è±¡åºç±»çå¯¹è±¡
