<h1 align="center">📔 C++ Prime 0x0D 练习题解</h1>

## 13.1 拷贝、赋值与销毁

### 13.1.1 拷贝构造函数

> 13.1 拷贝构造函数是什么？什么时候使用它？

如果一个构造函数的第一个参数是自身类类型的**引用**（且一般是一个`const`的引用），且任何额外参数都有默认值，则此构造函数是拷贝构造函数

当拷贝初始化的时候会使用拷贝构造函数



拷贝初始化不仅在我们用`=`定义变量时发生，在下列情况也会发生

* 将一个对象作为实参传递给一个非引用类型的形参
* 从一个返回类型为非引用类型的函数返回一个对象
* 用花括号列表初始化一个数组中的元素或聚合类中的成员
* 某些类型还会对它们所分配的对象使用拷贝初始化（`insert`、`push`会进行拷贝初始化，`emplace`会直接初始化）



> 13.2 解释为什么下面的声明是非法的：

```cpp
Sales_data::Sales_data(Sales_data rhs);
```

参数应该是自身类型的引用



> 13.3 当我们拷贝一个`StrBlob`时，会发生什么？拷贝一个`StrBlobPtr`呢？

`StrBlob`使用了`shared_ptr`引用计数加一

`StrBlobPtr`使用了`weak_ptr`引用计数不增加

其他数据成员直接拷贝



> 13.4 假定 `Point` 是一个类类型，它有一个`public`的拷贝构造函数，指出下面程序片段中哪些地方使用了拷贝构造函数：

```cpp
Point global;
Point foo_bar(Point arg) // 1
{
	Point local = arg, *heap = new Point(global); // 2，3
	*heap = local; //4
	Point pa[4] = { local, *heap }; // 5, 6
	return *heap;  // 7
}
```

* 将一个对象作为实参传递给一个非引用类型的形参(1)
* 使用=定义变量（2,3,4）
* 用花括号列表初始化一个数组中的元素或聚合类中的成员（5，6 用了两次)
* 从一个返回类型为非引用类型的函数返回一个对象（7）



> 13.5  给定下面的类框架，编写一个拷贝构造函数，拷贝所有成员。你的构造函数应该动态分配一个新的`string`，并将对象拷贝到`ps`所指向的位置，而不是拷贝ps本身：

```cpp
class HasPtr {
public:
	HasPtr(const std::string& s = std::string()):
		ps(new std::string(s)), i(0) { }
private:
	std::string *ps;
	int i;
}
```



```cpp
#include <string>

class HasPtr {
public:
	HasPtr(const std::string& s = std::string()):
		ps(new std::string(s)), i(0) { }
	HasPtr(const HasPtr& hp):ps(new std::string(*hp.ps)),i(hp.i){}
private:
	std::string *ps;
	int i;
};
```



### 13.1.2 拷贝赋值运算符

> 13.6 拷贝赋值运算符是什么？什么时候使用它？合成拷贝赋值运算符完成什么工作？什么时候会生成合成拷贝赋值运算符？

* 拷贝运算符是函数`operator=`的一个重载，拷贝赋值运算符接受一个与其所在类相同类型的参数

在使用赋值运算时会使用拷贝赋值运算符

**合成拷贝赋值运算符**

* 如果一个类未定义自己的拷贝赋值运算符，编译器会为它生产一个合成拷贝赋值运算符
* 对于某些类，合成拷贝赋值运算符用来禁止该类型对象的赋值
* 一般的，拷贝赋值运算符会将右侧运算对象的每个非`static`成员赋予左侧运算对象的对应成员，这一工作是通过**成员类型**的拷贝赋值运算符来完成的。对于数组类型的成员，逐个赋值数组元素
* 合成拷贝赋值运算符返回一个指向左侧运算对象的引用



> 13.7 当我们将一个 `StrBlob` 赋值给另一个 `StrBlob` 时，会发生什么？赋值 `StrBlobPtr` 呢？

`StrBlob`：会通过`shared_ptr`类的拷贝赋值运算符将`shared_ptr`拷贝赋值，且其计数器自增。

`StrBlobPtr`：会通过`weak_ptr`类的拷贝赋值运算符将`weak_ptr`拷贝赋值。`curr`调用内置类型的赋值运算符。



> 13.8  为13.1.1节练习13.5中的 `HasPtr` 类编写赋值运算符。类似拷贝构造函数，你的赋值运算符应该将对象拷贝到`ps`指向的位置。

记住处理自赋值和异常安全问题，这个是类值拷贝赋值运算符编写方式

```cpp
#include <string>

class HasPtr {
public:
	HasPtr(const std::string& s = std::string()):
		ps(new std::string(s)), i(0) { }
	HasPtr(const HasPtr& hp):ps(new std::string(*hp.ps)),i(hp.i){}
	HasPtr& operator=(const HasPtr& rhs){
		auto newp = new std::string(*rhs.ps);
		delete ps;
		ps = newp;
		i = rhs.i;
		return *this;
	}
    ~HasPtr() { delete ps;} 
private:
	std::string *ps;
	int i;
};
```



> 13.9 析构函数是什么？合成析构函数完成什么工作？什么时候会生成合成析构函数？

* 析构函数：释放对象使用的资源，并销毁对象的非`static`数据成员
* 析构函数是类的一个成员函数，名字波浪号接类名，没有返回值，也不接受参数，不能被重载，对一个给定类唯一

**析构函数完成什么工作**

* 在析构函数中，首先执行函数体，然后销毁成员，成员按初始化顺序的逆序销毁
* 不像构造函数有初始化列表，析构函数的析构部分是隐式的，成员销毁时发生什么完全依赖成员的类型
	* 销毁类类型的成员执行类类型的析构函数
	* 内置类型没有析构函数，销毁内置类型成员什么也不需要做
	* 隐式销毁一个内置指针类型的成员不会`delete`它所指向的对象
	* 与普通指针不同，智能指针是类类型，所以具有析构函数。智能指针成员在析构阶段会被自动销毁

**合成析构函数**

* 当一个类未定义自己的析构函数时，编译器会为它定义一个合成析构函数
* 对于某些类，合成析构函数被用来阻止该类型的对象被销毁
* 一般，合成析构函数的函数体为空
* **认识到析构函数体自身并不直接销毁成员很重要。成员是在析构函数体之后隐含的析构阶段被销毁的。**在整个对象销毁过程中，析构函数体是作为成员销毁步骤之外的另一部分进行的



> 13.10 当一个 `StrBlob` 对象销毁时会发生什么？一个 `StrBlobPtr` 对象销毁时呢？

`StrBlob`：`shared_ptr`的引用计数减少

`StrBlobPtr`：不影响引用计数



> 13.11 为前面练习中的 `HasPtr` 类添加一个析构函数。

```cpp
#include <string>

class HasPtr {
public:
	HasPtr(const std::string& s = std::string()):
		ps(new std::string(s)), i(0) { }
	HasPtr(const HasPtr& hp):ps(new std::string(*hp.ps)),i(hp.i){}
	HasPtr& operator=(const HasPtr& rhs){
		auto newp = new std::string(*rhs.ps);
		delete ps;
		ps = newp;
		i = rhs.i;
		return *this;
	}
	~HasPtr(){delete ps;}
private:
	std::string *ps;
	int i;
};
```



> 13.12 下面的代码片段中会发生几次析构函数调用？

```cpp
bool fcn(const Sales_data *trans, Sales_data accum)
{
	Sales_data item1(*trans), item2(accum);
	return item1.isbn() != item2.isbn();
}
```

3次

`item1`、`item2`、`accum`离开作用域被销毁

`trans`是指向对象的指针，析构函数不会执行



> 13.13 理解拷贝控制成员和构造函数的一个好方法的定义一个简单的类，为该类定义这些成员，每个成员都打印出自己的名字：

```cpp
struct X {
	X() {std::cout << "X()" << std::endl;}
	X(const X&) {std::cout << "X(const X&)" << std::endl;}
};
```

给 `X` 添加拷贝赋值运算符和析构函数，并编写一个程序以不同的方式使用 `X` 的对象：将它们作为非引用参数传递；动态分配它们；将它们存放于容器中；诸如此类。观察程序的输出，直到你确认理解了什么时候会使用拷贝控制成员，以及为什么会使用它们。**当你观察程序输出时，记住编译器可以略过对拷贝构造函数的调用。**

```
#include <iostream>
#include <string>
#include <vector>

struct X {
	X() {std::cout << "X()" << std::endl;}
	X(const X&) {std::cout << "X(const X&)" << std::endl;}
	X& operator=(const X&rhs){std::cout << "X& operator=(const X&rhs)" << std::endl;}
	~X(){std::cout << "~X()" << std::endl;}
};
void f(X a,const X& b){
	std::vector<X>v;
	std::cout << "push_back a" << std::endl;
	v.push_back(a);
	std::cout << "push_back b" << std::endl;
	v.push_back(b);


}
int main(){
	std::cout << "create pb" << std::endl;
	X a;

	std::cout << "create pb" << std::endl;
	X* pb = new X(a);

	std::cout << "f(a,*pb)" << std::endl;
	f(a,*pb);
	delete pb;


	
	return 0;
}
```

### 13.1.4 三/五法则

> 13.14 假定 `numbered` 是一个类，它有一个默认构造函数，能为每个对象生成一个唯一的序号，保存在名为 `mysn` 的数据成员中。假定 `numbered` 使用合成的拷贝控制成员，并给定如下函数：

```cpp
void f (numbered s) { cout << s.mysn < endl; }
```

则下面代码输出什么内容？

```cpp
numbered a, b = a, c = b;
f(a); f(b); f(c);
```

输出三个一样的数字



> 13.15 假定`numbered` 定义了一个拷贝构造函数，能生成一个新的序列号。这会改变上一题中调用的输出结果吗？如果会改变，为什么？新的输出结果是什么？

输出三个不一样的数字，但是和 a,b,c 也没有关系



> 13.16 如果 `f` 中的参数是 `const numbered&`，将会怎样？这会改变输出结果吗？如果会改变，为什么？新的输出结果是什么？

输出三个不一样的数字，分别是a,b,c的数字



> 13.17 分别编写前三题中所描述的 `numbered` 和 `f`，验证你是否正确预测了输出结果。

`3.14`

```cpp
#include <iostream>
#include <string>
#include <vector>

class numbered{
public:
	friend void f(numbered s);
	numbered() {mysn = num++;}
private:
	static int num;
	int mysn;
};

int numbered::num = 0;

void f (numbered s) { std::cout << s.mysn << std::endl; }

int main(){
	numbered a,b=a,c=b;
	f(a);f(b);f(c);
	return 0;
}
```

输出三个0

`3.15`

```cpp
#include <iostream>
#include <string>
#include <vector>

class numbered{
public:
	friend void f(numbered s);
	numbered() {mysn = num++;}
	numbered(const numbered &s){
		mysn = num++;
	}
private:
	static int num;
	int mysn;
};

int numbered::num = 0;

void f (numbered s) { std::cout << s.mysn << std::endl; }

int main(){
	numbered a,b=a,c=b;
	f(a);f(b);f(c);
	return 0;
}
```

输出3,4,5

`3.16`

```cpp
#include <iostream>
#include <string>
#include <vector>

class numbered{
public:
	friend void f(const numbered &s);
	numbered() {mysn = num++;}
	numbered(const numbered &s){
		mysn = num++;
	}
private:
	static int num;
	int mysn;
};

int numbered::num = 0;

void f (const numbered &s) { std::cout << s.mysn << std::endl; }

int main(){
	numbered a,b=a,c=b;
	f(a);f(b);f(c);
	return 0;
}
```

输出0,1,2



### 13.1.6 阻止拷贝

> 3.18 定义一个 `Employee` 类，它包含雇员的姓名和唯一的雇员证号。为这个类定义默认构造函数，以及接受一个表示雇员姓名的 `string` 的构造函数。每个构造函数应该通过递增一个 `static` 数据成员来生成一个唯一的证号。

```cpp
class Employee{
public:
	Employee(){id = id_num++;}
	Employee(const std::string &s):name(s),id(id_num){
		id_num++;
	}

private:
	std::string name;
	int id;
	static int id_num;
};
static int id_num = 1000;
```



> 3.19 你的 `Employee` 类需要定义它自己的拷贝控制成员吗？如果需要，为什么？如果不需要，为什么？实现你认为 `Employee` 需要的拷贝控制成员。

不需要拷贝，雇员没有必要因为拷贝而递增编号。将拷贝构造和赋值构造运算符定义为删除来阻止拷贝和赋值

```cpp
class Employee{
public:
	Employee(){id = id_num++;}
	Employee(const std::string &s):name(s),id(id_num){
		id_num++;
	}
	Employee(const Employee &)=delete;
	Employee& operator=(const Employee&)=delete;

private:
	std::string name;
	int id;
	static int id_num;
};
static int id_num = 1000;
```



> 13.20 解释当我们拷贝、赋值或销毁 `TextQuery` 和 `QueryResult` 类对象时会发生什么？

因为这两个类中使用的是智能指针，因此在拷贝时，类的所有成员都将被拷贝，在销毁时所有成员也将被销毁。

> 13.21你认为 `TextQuery` 和 `QueryResult` 类需要定义它们自己版本的拷贝控制成员吗？如果需要，为什么？实现你认为这两个类需要的拷贝控制操作。

不需要，因为用的智能指针，合成版本可以自动控制内存释放

**当我们决定一个类是否要定义他自己版本的拷贝控制成员时，一个基本原则就是确定这个类是否需要析构函数**，需要析构函数的类也需要拷贝和赋值操作



### 13.2 拷贝控制和资源管理

> 13.22 假定我们希望 `HasPtr` 的行为像一个值。即，对于对象所指向的 `string` 成员，每个对象都有一份自己的拷贝。我们将在下一节介绍拷贝控制成员的定义。但是，你已经学习了定义这些成员所需的所有知识。在继续学习下一节之前，为 `HasPtr` 编写拷贝构造函数和拷贝赋值运算符。

同`13.8`

### 13.2.1 行为像值的类

> 13.23 比较上一节练习中你编写的拷贝控制成员和这一节中的代码。确定你理解了你的代码和我们的代码之间的差异。

因为先把整章看了一遍才写练习的，所以基本没区别。

朴素一些的写法是用`if`判一下是不是自赋值，高级一点的写法是用`swap`来实现，后面会有



> 13.24 如果本节的 `HasPtr` 版本未定义析构函数，将会发生什么？如果未定义拷贝构造函数，将会发生什么？

三/五法则，拷贝成员控制的几个操作应该看成一个整体，一般要写就都写

未定义析构函数会内存泄漏

未定义拷贝构造函数会只拷贝指针的值，几个指针指向同一个地址



> 13.25 假定希望定义 `StrBlob` 的类值版本，而且希望继续使用 `shared_ptr`，这样我们的 `StrBlobPtr` 类就仍能使用指向`vector`的 `weak_ptr` 了。你修改后的类将需要一个拷贝的构造函数和一个拷贝赋值运算符，但不需要析构函数。解释拷贝构造函数和拷贝赋值运算符必须要做什么。解释为什么不需要析构函数。

拷贝构造函数和拷贝赋值运算符要重新动态分配内存

因为用的智能指针，可以自动控制内存释放，引用计数为0自动销毁对象，所以不需要析构函数



> 13.26 对上一题中描述的 `strBlob` 类，编写你自己的版本。

`StrBlob`里添加

```cpp
//拷贝构造
StrBlob(const StrBlob& sb):data(std::make_shared<std::vector<std::string>>(*sb.data)){}
//拷贝赋值
StrBlob& operator=(const StrBlob& sb){
    data = std::make_shared<std::vector<std::string>>(*sb.data);
    return *this;
}
```



### 13.2.2 定义行为像指针的类

> 13.27 定义你自己的使用引用计数版本的 `HasPtr`。

```cpp
class HasPtr {
public:
	HasPtr(const std::string &s = std::string())
		:ps(new std::string(s)),i(0),use(new std::size_t(1)){}
	HasPtr(const HasPtr &p)
		:ps(p.ps),i(p.i),use(p.use){++*use;}
	HasPtr& operator=(const HasPtr& rhs){
		++*rhs.use;
		if(--*use == 0){
			delete ps;
			delete use;
		}
		ps = rhs.ps;
		i = rhs.i;
		use = rhs.use;
		return *this;
	}
	~HasPtr(){
		if(--*use == 0){
			delete ps;
			delete use;
		}
	}
private:
	std::string *ps;
	int i;
	std::size_t *use;//引用计数
};
```



> 13.28 给定下面的类，为其实现一个默认构造函数和必要的拷贝控制成员。
>
> TreeNode 做了修改，count应该是引用计数，照道理这个应该放内存而不属于任何一个对象，所以改成了int* 类型

```cpp
(a) 
class TreeNode {
private:
	std::string value;
	int *count;
	TreeNode *left;
	TreeNode *right;	
};
(b)
class BinStrTree{
private:
	TreeNode *root;	
};
```



```cpp
#include <cstddef>
#include <string>


class TreeNode {
public:
	TreeNode()
		:value(std::string()),count(new int(1)),left(nullptr),right(nullptr){}
	TreeNode(const TreeNode& t)
		:value(t.value),count(t.count),left(t.left),right(t.right){
			++*count;
		}
	TreeNode& operator=(const TreeNode& rhs){
		++*rhs.count;
		if(--*count == 0){
			delete left;
			delete right;
			delete count;
		}
		value = rhs.value;
		left = rhs.left;
		right = rhs.right;
		return *this;
	}

	~TreeNode(){
		if(--*count == 0){
			delete left;
			delete right;
			delete count;
		}
	}

private:
	std::string value;
	int* count;
	TreeNode *left;
	TreeNode *right;	
};

class BinStrTree{
public:
	BinStrTree():root(nullptr){}
	BinStrTree(const BinStrTree &bst):root(new TreeNode(*bst.root)){}
	BinStrTree& operator=(const BinStrTree& rhs){
		auto newroot = new TreeNode(*rhs.root);
		delete root;
		root = newroot;
		return *this;
	}
	~BinStrTree(){delete root;}
private:
	TreeNode *root;	
};
```



## 13.3 交换操作

> 13.29 解释 `swap(HasPtr&, HasPtr&)`中对 `swap` 的调用不会导致递归循环。

因为没有给内置类型编写自定义的`swap`所以优先匹配到的是标准库的`std::swap`



> 13.30 为你的类值版本的 `HasPtr` 编写 `swap` 函数，并测试它。为你的 `swap` 函数添加一个打印语句，指出函数什么时候执行。

```cpp
#include <string>
#include <iostream>

class HasPtr {
public:
	friend void swap(HasPtr&, HasPtr&);
public:
	HasPtr(const std::string& s = std::string()):
		ps(new std::string(s)), i(0) { }
	HasPtr(const HasPtr& hp):ps(new std::string(*hp.ps)),i(hp.i){}
	HasPtr& operator=(const HasPtr& rhs){
		auto newp = new std::string(*rhs.ps);
		delete ps;
		ps = newp;
		i = rhs.i;
		return *this;
	}

	~HasPtr() {delete ps;} 
private:
	std::string *ps;
	int i;
};

inline 
void swap(HasPtr& lhs, HasPtr& rhs){
	using std::swap;
	swap(lhs.ps, rhs.ps);
	swap(lhs.i,rhs.i);
	std::cout << "using HasPtr's swap" << std::endl;
}
```



> 13.31 为你的 `HasPtr` 类定义一个 `<` 运算符，并定义一个 `HasPtr` 的 `vector`。为这个 `vector` 添加一些元素，并对它执行 `sort`。注意何时会调用 `swap`。

```cpp
#include <string>
#include <iostream>
#include <vector>
#include <algorithm>

class HasPtr {
public:
	friend void swap(HasPtr&, HasPtr&);
	friend bool operator<(const HasPtr& lhs,const HasPtr& rhs);
public:
	HasPtr(const std::string& s = std::string()):
		ps(new std::string(s)), i(0) { }
	HasPtr(const HasPtr& hp):ps(new std::string(*hp.ps)),i(hp.i){}
	HasPtr& operator=(HasPtr rhs){//赋值拷贝要写成swap形式会被用到
		swap(*this,rhs);
		return *this;
	}
	~HasPtr(){delete ps;}

	std::string show(){return *ps;}
	
private:
	std::string *ps;
	int i;
};

inline 
void swap(HasPtr& lhs, HasPtr& rhs){
	using std::swap;
	swap(lhs.ps, rhs.ps);
	swap(lhs.i,rhs.i);
	std::cout << "using HasPtr's swap" << std::endl;
}

inline
bool operator<(const HasPtr& lhs,const HasPtr& rhs){
	return *lhs.ps < *rhs.ps;
}

int main(){
	std::vector<HasPtr>vec;
	HasPtr a("fbc"),b("cde"),c("avpq");
	vec.push_back(a);
	vec.push_back(b);	
	vec.push_back(c);	

	sort(vec.begin(),vec.end());
	for(auto i:vec){
		std::cout <<i.show() << std::endl;
	}

}
```

输出结果

```
using HasPtr's swap
using HasPtr's swap
using HasPtr's swap
using HasPtr's swap
using HasPtr's swap
avpq
cde
fbc
```



> 13.32 类指针的 `HasPtr` 版本会从 `swap` 函数收益吗？如果会，得到了什么益处？如果不是，为什么？

不会，类值版本采用自己写的`swap`通过交换指针避免了重新内存分配，类指针版本本身就是指针，所以没啥收益



## 13.4 拷贝控制实例

> 13.33 为什么`Message`的成员`save`和`remove`的参数是一个 `Folder&`？为什么我们不能将参数定义为 `Folder` 或是 `const Folder`？

因为`save`和`remove`是要操作到某个具体的对象，而不是某一类



> 13.34 编写本节所描述的 `Message`。
>
> 13.36 设计并实现对应的 `Folder` 类。此类应该保存一个指向 `Folder` 中包含  `Message` 的 `set`。
>
> 13.37 为 `Message` 类添加成员，实现向 `folders` 添加和删除一个给定的 `Folder*`。这两个成员类似`Folder` 类的 `addMsg` 和 `remMsg` 操作。 （这里用引用的参数代替了）

```cpp
#include <string>
#include <iostream>
#include <set>
#include <vector>

class Folder;
class Message{
public:
	friend void swap(Message &lhs,Message &rhs);
	friend class Folder;
public:
	explicit Message(const std::string &str = ""):contents(str){}
	Message(const Message&);
	Message& operator=(const Message&);
	~Message();

	void save(Folder &);//向指定folder添加这条消息
	void remove(Folder &);//从指定folder删除这条消息
private:
	std::string contents;//保存内容
	std::set<Folder*>folders;//保存这条message在哪些folder里
	void add_to_Folders(const Message&);
	void remove_from_Folders();
};


class Folder{
public:
	friend void swap(Message &lhs,Message &rhs);
	friend class Message;
public:
	Folder()= default;
	Folder(const Folder &f);
	Folder& operator=(const Folder &f);
	~Folder();

private:
	std::set<Message*>messages;

	void add_to_Message(const Folder&);
    void remove_from_Message();

	void addMsg(const Message *);
	void remMsg(const Message *);
};
void Message::save(Folder &f){
	folders.insert(&f);
	f.addMsg(this);
}

void Message::remove(Folder &f){
	folders.erase(&f);
	f.remMsg(this);
}

void Message::add_to_Folders(const Message &m){
	for(auto f:m.folders)
		f->addMsg(this);
}
Message::Message(const Message &m):contents(m.contents),folders(m.folders){
	add_to_Folders(m);
}

Message::~Message(){
	remove_from_Folders();
}

Message& Message::operator=(const Message &rhs){
	remove_from_Folders();
	contents = rhs.contents;
	folders = rhs.folders;
	add_to_Folders(rhs);
	return *this;
}

void swap(Message &lhs,Message &rhs){
	using std::swap;
	for(auto f:lhs.folders)
		f->remMsg(&lhs);
	for(auto f:rhs.folders)
		f->remMsg(&rhs);
	swap(lhs.folders,rhs.folders);
	swap(lhs.contents,rhs.contents);
	for(auto f:lhs.folders)
		f->addMsg(&lhs);
	for(auto f:rhs.folders)
		f->addMsg(&rhs);

}

void Folder::add_to_Message(const Folder &f) {
    for (auto m : f.messages)
        m->save(*this);
}

Folder::Folder(const Folder &f) : messages(f.messages) { 
    add_to_Message(f); 
}

void Folder::remove_from_Message() {
    for (auto m : messages)
        m->remove(*this);
}

Folder::~Folder() 
{ 
    remove_from_Message(); 
}

Folder &Folder::operator=(const Folder &rhs) {
    remove_from_Message();
    messages = rhs.messages; 
    add_to_Message(rhs);
    return *this;
}

int main(){

	return 0;
}
```



> 13.35 如果`Message` 使用合成的拷贝控制成员，将会发生什么？

合成的拷贝控制成员可能导致赋值后已存在的`Folders`和`Message`不同步



> 13.38 我们并未使用拷贝交换方式来设计 `Message` 的赋值运算符。你认为其原因是什么？

对于动态分配内存的例子来说，拷贝交换方式是一种简洁的设计。而这里的 `Message` 类并不需要动态分配内存，用拷贝交换方式只会增加实现的复杂度。



## 13.5 动态内存管理类

> 13.39 编写你自己版本的 `StrVec`，包括自己版本的 `reserve`、`capacity` 和`resize`。

