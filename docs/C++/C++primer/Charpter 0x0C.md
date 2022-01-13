<h1 align="center">📔 C++ Prime 0x0C 练习题解</h1>

## 12.1 动态内存与智能指针

### 12.1.1 shared_ptr 类

> 12.1 在此代码的结尾，`b1` 和 `b2` 各包含多少个元素？
>
> ```cpp
> StrBlob b1;
> {
> 	StrBlob b2 = {"a", "an", "the"};
> 	b1 = b2;//b1,b2共享相同的元素
> 	b2.push_back("about");//b1,b2都有4个元素
> }//b2被析构了，但是不影响b1
> ```

如果两个对象共享底层数据，当某个对象被销毁是，我们不能单方面地销毁底层数据



> 12.2 编写你自己的`StrBlob` 类，包含`const` 版本的 `front` 和 `back`。

```cpp
#include <stdexcept>
#include <string>
#include <vector>
#include <iostream>
#include <initializer_list>
#include <memory>
#include <exception>

class StrBlob{
public:
	typedef std::vector<std::string>::size_type size_type;

	StrBlob();
	StrBlob(std::initializer_list<std::string> il);

	size_type size()const{return data->size();}
	bool empty()const{return data->empty();}

	void push_back(const std::string &t){data->push_back(t);}
	void pop_back();

	std::string& front();
	std::string& front()const;

	std::string& back();
	std::string& back()const;

private:
	std::shared_ptr<std::vector<std::string>>data;
	void check(size_type i,const std::string &msg)const;
};

StrBlob::StrBlob():data(std::make_shared<std::vector<std::string>>()){}
StrBlob::StrBlob(std::initializer_list<std::string> il)
	:data(std::make_shared<std::vector<std::string>>(il)){}

void StrBlob::check(size_type i,const std::string &msg)const{
	if(i >= data->size())
		throw std::out_of_range(msg);
}

std::string& StrBlob::front(){
	check(0,"front on empty StrBlod");
	return data->front();
}
std::string& StrBlob::front()const{
	check(0,"front on empty StrBlod");
	return data->front();
}
std::string& StrBlob::back(){
	check(0,"back on empty StrBlod");
	return data->back();
}
std::string& StrBlob::back()const{
	check(0,"back on empty StrBlod");
	return data->back();
}

void StrBlob::pop_back(){
	check(0,"pop_back on empty StrBlod");
	data->pop_back();
}


int main(){
	const StrBlob a{"this","is","a","const","StrBlob"};
	StrBlob b{"this","is","not","a","const","StrBlob"};

	std::cout << a.front() << " " << a.back() << std::endl;
	std::cout << b.front() << " " << b.back() << std::endl;

	return 0;
}
```



> 12.3     StrBlob 需要const 版本的push_back 和 pop_back吗？如需要，添加进去。否则，解释为什么不需要。

不需要，`push_back`和`pop_back`会改变对象内容



> 12.4 在我们的 `check` 函数中，没有检查 `i` 是否大于0。为什么可以忽略这个检查？

`size_type`是无符号整型，传负数会变成正数



> 12.5 我们未编写接受一个 `initializer_list explicit` 参数的构造函数。讨论这个设计策略的优点和缺点。

优点：不使用`explicit`则`initializer_list`可以隐式转换为`StrBlob`，这个逻辑上是可行的

缺点：如果我们确实需要用`initializer_list`，那么编译器会自动转为`StrBlod`



### 12.1.2 直接管理内存

> 12.6 编写函数，返回一个动态分配的 `int` 的`vector`。将此`vector` 传递给另一个函数，这个函数读取标准输入，将读入的值保存在 `vector` 元素中。再将`vector`传递给另一个函数，打印读入的值。记得在恰当的时刻`delete vector`。

```cpp
#include <vector>
#include <iostream>
#include <memory>

std::vector<int>* alloc_vector(){
	return new std::vector<int>();
}

void read_vector(std::vector<int>* p){
	for(int x;std::cin >> x;){
		p->push_back(x);
	}
}

void print_vector(const std::vector<int>* p){
	for(auto i:*p)std::cout << i << std::endl;
}

int main(){
	auto p = alloc_vector();

	read_vector(p);
	print_vector(p);
	
	delete p;

	return 0;
}
```



> 12.7 重做上一题，这次使用 `shared_ptr` 而不是内置指针。

```cpp
#include <vector>
#include <iostream>
#include <memory>

using T = std::vector<int>;
std::shared_ptr<T> alloc_vector(){
	return std::make_shared<T>();
}

void read_vector(std::shared_ptr<T> p){
	for(int x;std::cin >> x;){
		p->push_back(x);
	}
}

void print_vector(const std::shared_ptr<T> p){
	for(auto i:*p)std::cout << i << std::endl;
}

int main(){
	auto p = alloc_vector();
	read_vector(p);
	print_vector(p);
	
	return 0;
}
```



> 12.8 下面的函数是否有错误？如果有，解释错误原因。

```cpp
bool b() {
	int* p = new int;
	// ...
	return p;//被强制转为 bool 没有释放 p 指向的对象
}
```



> 12.9 解释下面代码执行的结果。

```cpp
int *q = new int(42), *r = new int(100);//q 指向 42，r 指向100
r = q;//r q 都指向 42，r 指向 100的内存空间没有释放，内存泄漏
auto q2 = make_shared<int>(42), r2 = make_shared<int>(100);
r2 = q2;//r2,q2都指向42, 100的内存空间不被r2引用的时候自动释放
```



### 12.1.3 shared_ptr 和 new 结合使用

> 12.10 下面的代码调用了第413页中定义的`process` 函数，解释此调用是否正确。如果不正确，应如何修改？

```cpp
shared_ptr<int> p(new int(42));
process(shared_ptr<int>(p));
```

正确，传入的`shared_ptr<int>(p)`创建一个临时对象，和`p`引用同一个对象，引用计数为2

函数结束，临时的智能指针被销毁，此时引用计数为1



> 12.11 如果我们像下面这样调用 `process`，会发生什么？

```cpp
process(shared_ptr<int>(p.get()));
```

`p.get()`返回一个内置指针

`shared_ptr<int>(p.get())`创建一个新的智能指针对象，传入的时候引用计数为1,传出的时候引用计数为0,指向的内存被释放，此时`p`变成了空悬指针



> 12.12 `p` 和 `sp` 的定义如下，对于接下来的对 `process` 的每个调用，如果合法，解释它做了什么，如果不合法，解释错误原因：

```cpp
auto p = new int();
auto sp = make_shared<int>();
(a) process(sp);//合法，传入的时候引用计数为2,结束的时候引用计数为1
(b) process(new int());//不合法，内置指针不能隐式转为智能指针
(c) process(p);//不合法，内置指针不能隐式转为智能指针
(d) process(shared_ptr<int>(p));//对于这个函数而言可以，但是会造成p成为空悬指针
```



> 12.13  如果执行下面的代码，会发生什么？

```cpp
auto sp = make_shared<int>();
auto p = sp.get();
delete p;
```

`sp`指向的内存空间已经被释放，再用`sp`操作会出错



### 12.1.4 智能指针和异常

> 12.14 编写你自己版本的用 `shared_ptr` 管理 `connection` 的函数。

```cpp
#include <vector>
#include <iostream>
#include <memory>

struct connection{//连接需要的信息
	std::string ip;
	int port;
	connection(std::string i,int p):ip(i),port(p){}
};

struct destination{//正在连接什么
	std::string ip;
	int port;
	destination(std::string i,int p):ip(i),port(p){}
};

connection connect(destination* pDest){//打开连接
	std::shared_ptr<connection> pConn(new connection(pDest->ip,pDest->port));
	std::cout << "creating connection(" << pConn.use_count() << ")" << std::endl;
	return *pConn;
};

void disconnect(connection pConn){//关闭给定连接
	std::cout << "connection close(" << pConn.ip << ":" << pConn.port << ")" << std::endl;	
};

void end_connection(connection* pConn){//自定义删除器
	disconnect(*pConn);
}

void f(destination &d){
	connection c = connect(&d);
	std::shared_ptr<connection>p(&c,end_connection);
	std::cout << "connecting now(" << p.use_count() << ")" << std::endl;
}
int main(){
	destination dest("220.181.111.111",10086);
	f(dest);
	
	return 0;
}
```



> 12.15 重写上一题的程序，用 `lambda` 代替`end_connection` 函数。

```cpp
#include <vector>
#include <iostream>
#include <memory>

struct connection{//连接需要的信息
	std::string ip;
	int port;
	connection(std::string i,int p):ip(i),port(p){}
};

struct destination{//正在连接什么
	std::string ip;
	int port;
	destination(std::string i,int p):ip(i),port(p){}
};

connection connect(destination* pDest){//打开连接
	std::shared_ptr<connection> pConn(new connection(pDest->ip,pDest->port));
	std::cout << "creating connection(" << pConn.use_count() << ")" << std::endl;
	return *pConn;
};

void disconnect(connection pConn){//关闭给定连接
	std::cout << "connection close(" << pConn.ip << ":" << pConn.port << ")" << std::endl;	
};

void f(destination &d){
	connection c = connect(&d);
	std::shared_ptr<connection>p(&c,[](connection* pConn){disconnect(*pConn);});
	std::cout << "connecting now(" << p.use_count() << ")" << std::endl;
}
int main(){
	destination dest("220.181.111.111",10086);
	f(dest);

	return 0;
}
```



### 12.1.5 unique_ptr

> 12.16 如果你试图拷贝或赋值 `unique_ptr`，编译器并不总是能给出易于理解的错误信息。编写包含这种错误的程序，观察编译器如何诊断这种错误。

```cpp
#include <string>
#include <vector>
#include <iostream>
#include <memory>

int main(){
	std::unique_ptr<std::string>p(new std::string("abc"));
	// std::unique_ptr<std::string>q(p);//拷贝
    // unique_ptr' has been explicitly marked deleted here unique_ptr(const unique_ptr&) = delete;
	std::unique_ptr<std::string>r;
	//r = p;//赋值
	//candidate function not viable: no known conversion from 'std::unique_ptr<std::string>' (aka 'unique_ptr<basic_string<char>>') to 'std::nullptr_t' (aka 'nullptr_t') for 1st argumentoperator=(nullptr_t) noexcept
	return 0;
}
```



> 12.17 下面的 `unique_ptr` 声明中，哪些是合法的，哪些可能导致后续的程序错误？解释每个错误的问题在哪里。

```cpp
int ix = 1024, *pi = &ix, *pi2 = new int(2048);
typedef unique_ptr<int> IntP;
(a) IntP p0(ix);//不合法，当我们在定义一个 unique_ptr 时，需要将其绑定到一个new 返回的指针上。
(b) IntP p1(pi);//不合法，当我们在定义一个 unique_ptr 时，需要将其绑定到一个new 返回的指针上。
(c) IntP p2(pi2);//合法，但pi2可能会变成空悬指针
(d) IntP p3(&ix);//不合法，当 p3 被销毁时，它试图释放一个栈空间的对象。
(e) IntP p4(new int(2048));//合法
(f) IntP p5(p2.get());//不合法，p5 和 p2 指向同一个对象，但是他们互相独立，p5被销毁的时候也释放了p2指向的对象，p2变成空悬
```



> 12.18 `shared_ptr` 为什么没有 `release` 成员？

`release`放弃对指针的控制权，返回指针

`unique_ptr`一个对象只能被一个指针控制，需要释放控制权来给其他指针

`shared_ptr`允许多个指针指向同一个对象，不需要转移控制权



### 12.1.6 weak_ptr

> 12.19 定义你自己版本的 `StrBlobPtr`，更新 `StrBlob` 类，加入恰当的 `friend` 声明以及 `begin` 和 `end` 成员。
>
> 12.20 编写程序，逐行读入一个输入文件，将内容存入一个 `StrBlob` 中，用一个 `StrBlobPtr` 打印出 `StrBlob` 中的每个元素。

```cpp
#include <cstddef>
#include <stdexcept>
#include <string>
#include <vector>
#include <iostream>
#include <initializer_list>
#include <memory>
#include <exception>
#include <fstream>


class StrBlobPtr;

class StrBlob{
public:
	friend class StrBlobPtr;
	typedef std::vector<std::string>::size_type size_type;

	StrBlob():data(std::make_shared<std::vector<std::string>>()){};
	StrBlob(std::initializer_list<std::string> il)
		:data(std::make_shared<std::vector<std::string>>(il)){}

	size_type size()const{return data->size();}
	bool empty()const{return data->empty();}

	void push_back(const std::string &t){data->push_back(t);}
	void pop_back(){check(0,"pop_back on empty StrBlod");data->pop_back();};

	std::string& front(){check(0,"front on empty StrBlod");return data->front();};
	std::string& front()const{check(0,"front on empty StrBlod");return data->front();};

	std::string& back(){check(0,"back on empty StrBlod");return data->back();};
	std::string& back()const{check(0,"back on empty StrBlod");return data->back();};

	StrBlobPtr begin();
	StrBlobPtr end();

private:
	std::shared_ptr<std::vector<std::string>>data;
	void check(size_type i,const std::string &msg)const{
		if(i >= data->size())throw std::out_of_range(msg);
	};
};


class StrBlobPtr{
public:
	StrBlobPtr():curr(0){}
	StrBlobPtr(StrBlob &a,std::size_t sz = 0):wptr(a.data),curr(sz){}

	bool operator!=(const StrBlobPtr& p) { return p.curr != curr; }

	std::string& deref() const{
		auto p = check(curr,"dereference past end");
		return (*p)[curr];
	}
	StrBlobPtr& incr(){
		check(curr,"increment past end of StrBlobPtr");
		++curr;
		return *this;
	}

private:
	std::shared_ptr<std::vector<std::string>> check(std::size_t i,const std::string &msg)const{
		auto ret = wptr.lock();
		if(!ret)throw std::runtime_error("unbound StrBlodPtr");
		if(i >= ret->size())throw std::out_of_range(msg);
		return ret;
	}
	std::weak_ptr<std::vector<std::string>> wptr;
	std::size_t curr;
};

StrBlobPtr StrBlob::begin(){return StrBlobPtr(*this);}
StrBlobPtr StrBlob::end(){return StrBlobPtr(*this,data->size());}

int main(int argc,char** argv){
	std::ifstream ifs(argv[1]);
	StrBlob s_blod;
	for(std::string s;std::getline(ifs,s);){
		s_blod.push_back(s);
	}
	for(StrBlobPtr iter = s_blod.begin(); iter != s_blod.end(); iter.incr()){
		std::cout << iter.deref() << std::endl;
	}

	return 0;
}
```



> 12.21 也可以这样编写 `StrBlobPtr` 的 `deref` 成员：

```cpp
std::string& deref() const {
	return (*check(curr, "dereference past end"))[curr];
}
```

你认为哪个版本更好？为什么？

原来的好，容易看懂





> 12.22 为了能让 `StrBlobPtr` 使用 `const StrBlob`，你觉得应该如何修改？定义一个名为`ConstStrBlobPtr` 的类，使其能够指向 `const StrBlob`。

```cpp
#include <cstddef>
#include <stdexcept>
#include <string>
#include <vector>
#include <iostream>
#include <initializer_list>
#include <memory>
#include <exception>
#include <fstream>


class StrBlobPtr;
class ConstStrBlobPtr;

class StrBlob{
public:
	friend class StrBlobPtr;
	friend class ConstStrBlobPtr;
	typedef std::vector<std::string>::size_type size_type;

	StrBlob():data(std::make_shared<std::vector<std::string>>()){};
	StrBlob(std::initializer_list<std::string> il)
		:data(std::make_shared<std::vector<std::string>>(il)){}

	size_type size()const{return data->size();}
	bool empty()const{return data->empty();}

	void push_back(const std::string &t){data->push_back(t);}
	void pop_back(){check(0,"pop_back on empty StrBlod");data->pop_back();};

	std::string& front(){check(0,"front on empty StrBlod");return data->front();};
	std::string& front()const{check(0,"front on empty StrBlod");return data->front();};

	std::string& back(){check(0,"back on empty StrBlod");return data->back();};
	std::string& back()const{check(0,"back on empty StrBlod");return data->back();};

	StrBlobPtr begin();
	StrBlobPtr end();

	ConstStrBlobPtr cbegin();
	ConstStrBlobPtr cend();

private:
	std::shared_ptr<std::vector<std::string>>data;
	void check(size_type i,const std::string &msg)const{
		if(i >= data->size())throw std::out_of_range(msg);
	};
};


class StrBlobPtr{
public:
	StrBlobPtr():curr(0){}
	StrBlobPtr(StrBlob &a,std::size_t sz = 0):wptr(a.data),curr(sz){}

	bool operator!=(const StrBlobPtr& p) { return p.curr != curr; }

	std::string& deref() const{
		auto p = check(curr,"dereference past end");
		return (*p)[curr];
	}
	StrBlobPtr& incr(){
		check(curr,"increment past end of StrBlobPtr");
		++curr;
		return *this;
	}

private:
	std::shared_ptr<std::vector<std::string>> check(std::size_t i,const std::string &msg)const{
		auto ret = wptr.lock();
		if(!ret)throw std::runtime_error("unbound StrBlodPtr");
		if(i >= ret->size())throw std::out_of_range(msg);
		return ret;
	}
	std::weak_ptr<std::vector<std::string>> wptr;
	std::size_t curr;
};

class ConstStrBlobPtr{
public:
	ConstStrBlobPtr():curr(0){}
	ConstStrBlobPtr(const StrBlob &a,std::size_t sz = 0):wptr(a.data),curr(sz){}

	bool operator!=(const ConstStrBlobPtr& p) { return p.curr != curr; }

	const std::string& deref() const{
		auto p = check(curr,"dereference past end");
		return (*p)[curr];
	}
	ConstStrBlobPtr& incr(){
		check(curr,"increment past end of StrBlobPtr");
		++curr;
		return *this;
	}

private:
	std::shared_ptr<std::vector<std::string>> check(std::size_t i,const std::string &msg)const{
		auto ret = wptr.lock();
		if(!ret)throw std::runtime_error("unbound StrBlodPtr");
		if(i >= ret->size())throw std::out_of_range(msg);
		return ret;
	}
	std::weak_ptr<std::vector<std::string>> wptr;
	std::size_t curr;
};

StrBlobPtr StrBlob::begin(){return StrBlobPtr(*this);}
StrBlobPtr StrBlob::end(){return StrBlobPtr(*this,data->size());}

ConstStrBlobPtr StrBlob::cbegin(){return ConstStrBlobPtr(*this);}
ConstStrBlobPtr StrBlob::cend(){return ConstStrBlobPtr(*this,data->size());}

int main(int argc,char** argv){
	std::ifstream ifs(argv[1]);
	StrBlob s_blod;
	for(std::string s;std::getline(ifs,s);){
		s_blod.push_back(s);
	}
	for(ConstStrBlobPtr iter = s_blod.cbegin(); iter != s_blod.cend(); iter.incr()){
		std::cout << iter.deref() << std::endl;
	}

	return 0;
}
```



## 12.2 动态数组

### 12.2.1 new 和数组

> 12.23 编写一个程序，连接两个字符串字面常量，将结果保存在一个动态分配的`char`数组中。重写这个程序，连接两个标准库`string`对象。

```cpp
#include <cstddef>
#include <stdexcept>
#include <string>
#include <vector>
#include <iostream>
#include <initializer_list>
#include <memory>
#include <exception>
#include <fstream>
#include <cstring>

int main(){
	const char* s1="abc";
	const char* s2="def";
	unsigned len = strlen(s1)+strlen(s2)+1;
	char* s3 = new char[len]();
	
	std::strcat(s3,s1);
	std::strcat(s3,s2);
	std::cout << s3 << std::endl;

	delete[] s3;
	return 0;
}
```



```cpp
#include <cstddef>
#include <stdexcept>
#include <string>
#include <vector>
#include <iostream>
#include <initializer_list>
#include <memory>
#include <exception>
#include <fstream>
#include <cstring>

int main(){
	std::string s1="abc",s2="def";
	unsigned len = s1.size()+s2.size()+1;
	char* s3 = new char[len]();
	
	std::strcat(s3,s1.c_str());
	std::strcat(s3,s2.c_str());
	std::cout << s3 << std::endl;

	delete[] s3;
	return 0;
}
```



> 12.24 编写一个程序，从标准输入读取一个字符串，存入一个动态分配的字符数组中。描述你的程序如何处理变长输入。测试你的程序，输入一个超出你分配的数组长度的字符串。

```cpp
#include <cstddef>
#include <stdexcept>
#include <string>
#include <vector>
#include <iostream>
#include <initializer_list>
#include <memory>
#include <exception>
#include <fstream>
#include <cstring>

int main(){
	unsigned size = 0;
	std::cin >> size;
	char* s = new char[size+1];

	std::cin.ignore();
	std::cin.get(s,size+1);
	
	std::cout << s << std::endl;
	delete[] s;
	return 0;
}
```



> 12.25 给定下面的`new`表达式，你应该如何释放`pa`？

```cpp
int *pa = new int[10];
```



```cpp
delete [] pa;
```



### 12.2.2 allocator 类

> 12.26 用 allocator 重写第427页中的程序。

```cpp
#include <cstddef>
#include <stdexcept>
#include <string>
#include <vector>
#include <iostream>
#include <initializer_list>
#include <memory>
#include <exception>
#include <fstream>
#include <cstring>

int main(){
	unsigned int n = 0;
	std::cin >> n;

	std::allocator<std::string> alloc;
	auto const p = alloc.allocate(n);

	std::string *q = p;
	std::string s;
	while(std::cin >> s && q != p+n){
		alloc.construct(q++,s);
	}
	
	while(q!=p){
		std::cout << *--q << " ";
		alloc.destroy(q);
	}
	alloc.deallocate(p,n);

	return 0;
}
```



## 12.3 使用标准库：文本查询程序

### 12.3.1 文本查询程序设计

> 12.27 `TextQuery` 和 `QueryResult` 类只使用了我们已经介绍过的语言和标准库特性。不要提前看后续章节内容，只用已经学到的知识对这两个类编写你自己的版本。

头文件 `TQ.hpp`

```cpp
#ifndef TQ_H
#define TQ_H

#include <string>
#include <vector>
#include <memory>
#include <map>
#include <set>
#include <iostream>
#include <fstream>
#include <sstream>


class QueryResult;
class TextQuery{
public:
	using line_no = std::vector<std::string>::size_type;
public:
	TextQuery() = default;
	TextQuery(std::ifstream &ifs);

	QueryResult query(const std::string &sought)const;

private:
	std::shared_ptr<std::vector<std::string>> file;
	std::map<std::string, std::shared_ptr<std::set<line_no>>> wm;
};

class QueryResult{
public:
	friend std::ostream& print(std::ostream &os,const QueryResult &result);//打印结果
public:
	QueryResult()= default;
	QueryResult(std::string s, std::shared_ptr<std::set<TextQuery::line_no>>p,
		std::shared_ptr<std::vector<std::string>>f):sought(s),lines(p),file(f){}

private:
	std::string sought;	//查询单词
	std::shared_ptr<std::set<TextQuery::line_no>>lines;//出现的行号
	std::shared_ptr<std::vector<std::string>>file;//输入文件
};
std::string make_plural(const int count,const std::string word,const std::string &suf);
std::ostream& print(std::ostream &os,const QueryResult &result);

#endif
```



头文件实现`TQ.cpp`

```cpp
#include "TQ.hpp"

TextQuery::TextQuery(std::ifstream &ifs):file(new std::vector<std::string>){
	std::string text;
	while (getline(ifs, text)){//text保存每一行
		file->push_back(text);
		int n = file->size() - 1;
		std::istringstream line(text);
		std::string word;
		while (line >> word){//word保存一行的每个单词
			auto &lines = wm[word];//看word映射过去的指针是否存在
			if (!lines)lines.reset(new std::set<line_no>);//指针不存在分配一个新的set
			lines->insert(n);//将此行号插入set
		}
	}
}

QueryResult TextQuery::query(const std::string &sought)const{
	static std::shared_ptr<std::set<line_no>>nodata(new std::set<line_no>);
	auto loc = wm.find(sought);
	if(loc == wm.end()){
		return QueryResult(sought,nodata,file);
	}else{
		return QueryResult(sought,loc->second,file);
	}
}
std::string make_plural(const int count,const std::string word,const std::string &suf){
	return count>1?(word+suf):word;
}
std::ostream& print(std::ostream &os,const QueryResult &result){
	os << result.sought <<" occurs " << result.lines->size() << " "
		<< make_plural(result.lines->size(),"time","s") << std::endl;
	for(auto num:*result.lines){
		os << "\t(line " << num+1 <<") "
			<< *(result.file->begin()+num) << std::endl;
	}
	return os;
}
```



主函数`Test.cpp`

```cpp
#include "TQ.hpp"

void runQueries(std::ifstream &infile){
	TextQuery tq(infile);
	while(true){
		std::cout << "enter word to look for , or q to quit: ";
		std::string s;
		if(!(std::cin >> s)|| s == "q")break;
		print(std::cout,tq.query(s)) << std::endl;
	}
}

int main(int argc,char **argv){
	std::ifstream ifs(argv[1]);
	runQueries(ifs);
	return 0;
}
```

数据文件`wordsfile.txt`

```
Twenty years from now you will be more disappointed by the things that you didn't do than by the ones you did do. So throw off the bowlines. Sail away from the safe harbor. 
Catch the trade winds in your sails. 
Explore. Dream. Discover. 
```

编译链接

```shell
clang++ -c TQ.cpp
clang++ -c Test.cpp
clang++ -o main Test.o TQ.o
clang++ -c TQ.cpp
./main wordsfile.txt
```

 



> 12.28 编写程序实现文本查询，不要定义类来管理数据。你的程序应该接受一个文件，并与用户交互来查询单词。使用`vector`、`map` 和 `set` 容器来保存来自文件的数据并生成查询结果。

```cpp
#include <string>
#include <vector>
#include <map>
#include <set>
#include <iostream>
#include <fstream>
#include <memory>
#include <sstream>
#include <functional>
#include <algorithm>

using std::vector;using std::set;using std::map;
using std::shared_ptr;using std::ifstream;using std::string;
using std::istringstream;

int main(int argc,char** argv){
	ifstream ifs(argv[1]);
	vector<string>input;
	map<string,set<vector<string>::size_type>>dict;
	vector<string>::size_type lineNo = 0;

	for(string line;std::getline(ifs,line);++lineNo){
		input.push_back(line);
		istringstream line_stream(line);
		for (string text, word; line_stream >> text; word.clear()){
			std::remove_copy_if(text.begin(), text.end(), std::back_inserter(word), ispunct);//去除标点符号
			dict[word].insert(lineNo);
		}
	}


	do{
		std::cout << "enter word to look for , or q to quit: ";
		std::string s;
		if(!(std::cin >> s)|| s == "q")break;
		auto found = dict.find(s);
		if(found != dict.end()){
			std::cout << s << " occurs " << found->second.size() << (found->second.size() > 1 ? " times" : " time") << std::endl;
			for (auto i : found->second)
			std::cout << "\t(line " << i + 1 << ") " << input.at(i) << std::endl;
		}else{
			std::cout << s << " occurs 0 time" << std::endl;
		}
		
	}while(true);
	
	return 0;
}
```



> 12.29 我们曾经用`do while` 循环来编写管理用户交互的循环。用`do while` 重写本节程序，解释你倾向于哪个版本，为什么？

```cpp
void runQueries(std::ifstream &infile){
	TextQuery tq(infile);
	do{
		std::cout << "enter word to look for , or q to quit: ";
		std::string s;
		if(!(std::cin >> s)|| s == "q")break;
		print(std::cout,tq.query(s)) << std::endl;
	}while(true);
}
```

`do while`看起来顺眼哈哈，不过平常都是用`while`

### 12.3.2 文本查询程序类的定义

> 12.30 定义你自己版本的 `TextQuery` 和 `QueryResult` 类，并执行12.3.1节中的`runQueries` 函数。

见练习 12.27



> 12.31 如果用`vector` 代替 `set` 保存行号，会有什么差别？哪个方法更好？为什么？

`set`更好，可以保证行号有序不重复

`vector`可能重复



> 12.32 重写 `TextQuery` 和 `QueryResult`类，用`StrBlob` 代替 `vector<string>` 保存输入文件。

`StrBlob.hpp`

```cpp
#ifndef StrBlob_H
#define StrBlob_H

#include <cstddef>
#include <stdexcept>
#include <string>
#include <vector>
#include <iostream>
#include <initializer_list>
#include <memory>
#include <exception>
#include <fstream>


class StrBlobPtr;

class StrBlob{
public:
	friend class StrBlobPtr;
	typedef std::vector<std::string>::size_type size_type;

	StrBlob():data(std::make_shared<std::vector<std::string>>()){};
	StrBlob(std::initializer_list<std::string> il)
		:data(std::make_shared<std::vector<std::string>>(il)){}

	size_type size()const{return data->size();}
	bool empty()const{return data->empty();}

	void push_back(const std::string &t){data->push_back(t);}
	void pop_back(){check(0,"pop_back on empty StrBlod");data->pop_back();};

	std::string& front(){check(0,"front on empty StrBlod");return data->front();};
	std::string& front()const{check(0,"front on empty StrBlod");return data->front();};

	std::string& back(){check(0,"back on empty StrBlod");return data->back();};
	std::string& back()const{check(0,"back on empty StrBlod");return data->back();};

	StrBlobPtr begin();
	StrBlobPtr end();

private:
	std::shared_ptr<std::vector<std::string>>data;
	void check(size_type i,const std::string &msg)const{
		if(i >= data->size())throw std::out_of_range(msg);
	};
};


class StrBlobPtr{
public:
	StrBlobPtr():curr(0){}
	StrBlobPtr(StrBlob &a,std::size_t sz = 0):wptr(a.data),curr(sz){}

	bool operator!=(const StrBlobPtr& p) { return p.curr != curr; }

	std::string& deref() const{
		auto p = check(curr,"dereference past end");
		return (*p)[curr];
	}
	StrBlobPtr& incr(){
		check(curr,"increment past end of StrBlobPtr");
		++curr;
		return *this;
	}

private:
	std::shared_ptr<std::vector<std::string>> check(std::size_t i,const std::string &msg)const{
		auto ret = wptr.lock();
		if(!ret)throw std::runtime_error("unbound StrBlodPtr");
		if(i >= ret->size())throw std::out_of_range(msg);
		return ret;
	}
	std::weak_ptr<std::vector<std::string>> wptr;
	std::size_t curr;
};

inline StrBlobPtr StrBlob::begin(){return StrBlobPtr(*this);}
inline StrBlobPtr StrBlob::end(){return StrBlobPtr(*this,data->size());}

#endif
```

`TQ.hpp`

```cpp
#ifndef TQ_H
#define TQ_H

#include <string>
#include <vector>
#include <memory>
#include <map>
#include <set>
#include <iostream>
#include <fstream>
#include <sstream>
#include "StrBlob.hpp"


class QueryResult;
class TextQuery{
public:
	using line_no = std::vector<std::string>::size_type;
public:
	TextQuery() = default;
	TextQuery(std::ifstream &ifs);

	QueryResult query(const std::string &sought)const;

private:
	std::shared_ptr<StrBlob> file;
	std::map<std::string, std::shared_ptr<std::set<line_no>>> wm;
};

class QueryResult{
public:
	friend std::ostream& print(std::ostream &os,const QueryResult &result);//打印结果
public:
	QueryResult()= default;
	QueryResult(std::string s, std::shared_ptr<std::set<TextQuery::line_no>>p,
		std::shared_ptr<StrBlob>f):sought(s),lines(p),file(f){}

private:
	std::string sought;	//查询单词
	std::shared_ptr<std::set<TextQuery::line_no>>lines;//出现的行号
	std::shared_ptr<StrBlob>file;//输入文件
};
std::string make_plural(const int count,const std::string word,const std::string &suf);
std::ostream& print(std::ostream &os,const QueryResult &result);

#endif
```

`TQ.cpp`

```cpp
#include "TQ.hpp"

TextQuery::TextQuery(std::ifstream &ifs):file(new StrBlob){
	std::string text;
	while (getline(ifs, text)){//text保存每一行
		file->push_back(text);
		int n = file->size() - 1;
		std::istringstream line(text);
		std::string word;
		while (line >> word){//word保存一行的每个单词
			auto &lines = wm[word];//看word映射过去的指针是否存在
			if (!lines)lines.reset(new std::set<line_no>);//指针不存在分配一个新的set
			lines->insert(n);//将此行号插入set
		}
	}
}

QueryResult TextQuery::query(const std::string &sought)const{
	static std::shared_ptr<std::set<line_no>>nodata(new std::set<line_no>);
	auto loc = wm.find(sought);
	if(loc == wm.end()){
		return QueryResult(sought,nodata,file);
	}else{
		return QueryResult(sought,loc->second,file);
	}
}
std::string make_plural(const int count,const std::string word,const std::string &suf){
	return count>1?(word+suf):word;
}
std::ostream& print(std::ostream &os,const QueryResult &result){
	os << result.sought <<" occurs " << result.lines->size() << " "
		<< make_plural(result.lines->size(),"time","s") << std::endl;
	for(auto num:*result.lines){
		StrBlobPtr p(*result.file,num);
		os << "\t(line " << num+1 <<") "
			<< p.deref() << std::endl;
	}
	return os;
}

```

`Test.cpp`

```cpp
#include "TQ.hpp"

void runQueries(std::ifstream &infile){
	TextQuery tq(infile);
	do{
		std::cout << "enter word to look for , or q to quit: ";
		std::string s;
		if(!(std::cin >> s)|| s == "q")break;
		print(std::cout,tq.query(s)) << std::endl;
	}while(true);
}

int main(int argc,char **argv){
	std::ifstream ifs(argv[1]);
	runQueries(ifs);
	return 0;
}
```



> 12.33 在第15章中我们将扩展查询系统，在 `QueryResult` 类中将会需要一些额外的成员。添加名为 `begin` 和 `end` 的成员，返回一个迭代器，指向一个给定查询返回的行号的 `set` 中的位置。再添加一个名为 `get_file` 的成员，返回一个 `shared_ptr`，指向 `QueryResult` 对象中的文件。

```cpp
class QueryResult{
public:
	friend std::ostream& print(std::ostream &os,const QueryResult &result);//打印结果
	using Iter = std::set<TextQuery::line_no>::iterator;	
public:
	QueryResult()= default;
	QueryResult(std::string s, std::shared_ptr<std::set<TextQuery::line_no>>p,
		std::shared_ptr<StrBlob>f):sought(s),lines(p),file(f){}
		
public:
	Iter begin() const { return lines->begin(); }
	Iter end() const { return lines->end(); }
	std::shared_ptr<std::vector<std::string>> get_file() const 
	{ 
		return std::make_shared<std::vector<std::string>>(file); 
	}

private:
	std::string sought;	//查询单词
	std::shared_ptr<std::set<TextQuery::line_no>>lines;//出现的行号
	std::shared_ptr<StrBlob>file;//输入文件
};
```

