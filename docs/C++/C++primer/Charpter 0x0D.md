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

参考了`applenob`的答案，感觉有几处代码的重构简化写的很不错（比如提炼了`copy_n_move`)

头文件

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
	StrVec():elements(nullptr),first_free(nullptr),cap(nullptr){}
	StrVec(const StrVec&);
	StrVec& operator=(const StrVec&);
	~StrVec();

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
private:
	std::string *elements;
	std::string *first_free;
	std::string *cap;
};

#endif
```

实现文件

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

StrVec::~StrVec(){free();}

StrVec& StrVec::operator=(const StrVec &rhs){
	auto data = alloc_n_copy(rhs.begin(),rhs.end());
	free();
	elements = data.first;
	first_free = cap = data.second;
	return *this;
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


```



> 13.40 为你的 `StrVec` 类添加一个构造函数，它接受一个 `initializer_list<string>` 参数。

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
	StrVec():elements(nullptr),first_free(nullptr),cap(nullptr){}
	StrVec(const StrVec&);
	StrVec& operator=(const StrVec&);
	~StrVec();
	StrVec(std::initializer_list<std::string>);

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

StrVec::StrVec(std::initializer_list<std::string> il)
{
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
```



> 13.41 在 `push_back` 中，我们为什么在 `construct` 调用中使用后置递增运算？如果使用前置递增运算的话，会发生什么？

会跳过一个空位置





> 13.42 在你的 `TextQuery` 和 `QueryResult` 类中用你的 `StrVec` 类代替`vector<string>`，以此来测试你的 `StrVec` 类。



> 13.43 重写 `free` 成员，用 `for_each` 和 `lambda` 来代替 `for` 循环 `destroy` 元素。你更倾向于哪种实现，为什么？

```cpp
void StrVec::free(){
	if(elements){
        for_each(elements,first_free,[this](std::string& s){alloc.destroy(&s);})
		alloc.deallocate(elements,cap-elements);
	}
}

```

`for_each`+`lambda`更简洁,而且可以避免越界



> 13.44 编写标准库 `string` 类的简化版本，命名为 `String`。你的类应该至少有一个默认构造函数和一个接受 C 风格字符串指针参数的构造函数。使用 `allocator` 为你的 `String`类分配所需内存。

参考书上的`StrVec`和`applenon`

`MyString.hpp`

```cpp
#ifndef MyString_H
#define MyString_H

#include <memory>
#include <algorithm>
#include <iterator>
class MyString{
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

`MyString.cpp`

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

```

`Test.cpp`

```cpp
#include <memory>
#include <vector>
#include <iostream>
#include "MyString.hpp"

void foo(MyString x)
{
    std::cout << x.begin() << std::endl;
}

void bar(const MyString& x)
{
    std::cout << x.begin() << std::endl;
}

MyString baz()
{
    MyString ret("world");
    return ret;
}

int main()
{
    char text[] = "world";

    MyString s0;
    MyString s1("hello");
    MyString s2(s0);
    MyString s3 = s1;
    MyString s4(text);
    s2 = s1;

    foo(s1);
    bar(s1);
    foo("temporary");
    bar("temporary");
    MyString s5 = baz();

    std::vector<MyString> svec;
    svec.reserve(8);
    svec.push_back(s0);
    svec.push_back(s1);
    svec.push_back(s2);
    svec.push_back(s3);
    svec.push_back(s4);
    svec.push_back(s5);
    svec.push_back(baz());
    svec.push_back("good job");

    for (const auto &s : svec) {
        std::cout << s.begin() << std::endl;
    }
}
```

`makefile`

```makefile
main:Test.o MyString.o
	clang++ Test.o MyString.o -o main -g
Test.o:Test.cpp
	clang++ -c Test.cpp -o Test.o
MyString.o:MyString.cpp MyString.hpp
	clang++ -c MyString.cpp -o MyString.o
clean:
	rm *.o main
```



## 13.6 对象移动

### 13.6.1 右值引用

> 13.45 解释左值引用和右值引用的区别？

左值引用：一般引用

右值引用：绑定到右值上的引用

* 右值引用必须绑定在右值（要求转换的表达式，字面常量，返回右值的表达式），不能直接绑定到一个左值上，通过`&&`而不是`&`来获得
	* 返回左值引用的函数，连通赋值、下标、解引用和前置递增/递减运算符，都是返回左值表达式的例子
	* 返回非引用类型的函数，连通算术、关系、位以及后值递增/递减运算符，都生成右值。不能将左值引用绑定到这类表达式上，但可以将一个`cosnt`的左值引用或者一个右值引用绑定到这类表达式上
* 右值引用只能绑定到一个将要销毁的对象上（就好比打游戏，对面有武器，对面要死了你才可以去捡武器）
* 左值持久：对象的身份，具有持久的状态
* 右值短暂：对象的值，要么是字面常量，要么是表达式求值过程中创建的临时对象



> 13.46 什么类型的引用可以绑定到下面的初始化器上？

```cpp
int f();
vector<int> vi(100);
int? r1 = f();//右值引用，返回非引用类型的函数
int? r2 = vi[0];//左值引用，下标
int? r3 = r1;//左值引用，变量是左值，即使这个变量是右值引用（右值引用不等于右值）。
int? r4 = vi[0] * f();//右值引用，返回右值的表达式（右值短暂临时）
```



> 13.47 对你在练习13.44中定义的 `String`类，为它的拷贝构造函数和拷贝赋值运算符添加一条语句，在每次函数执行时打印一条信息。
>
> 13.48 定义一个`vector<String>` 并在其上多次调用 `push_back`。运行你的程序，并观察 `String` 被拷贝了多少次。

见练习`13.44`



### 13.6.2 移动构造函数和移动赋值运算符

> 13.49 为你的 `StrVec`、`String` 和 `Message` 类添加一个移动构造函数和一个移动赋值运算符。

以`String`为例

```cpp
#ifndef MyString_H
#define MyString_H

#include <memory>
#include <algorithm>
#include <iterator>
#include <iostream>

class MyString{
public:
	MyString():MyString(""){}
	MyString(const char*);
	MyString(const MyString&);
	MyString& operator=(const MyString&);
	MyString(MyString &&) noexcept;//移动构造函数
	MyString& operator=(MyString &&)noexcept;//移动赋值函数
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

`MyString.cpp`

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
	std::cout << "void MyString::range_initializer(const char *first, const char *last)" << std::endl;
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
MyString::MyString(MyString && ms) noexcept:elements(ms.elements),ends(ms.ends){
	std::cout << "MyString::MyString(MyString && ms)" << std::endl;
	ms.elements = ms.ends = nullptr;
}

MyString& MyString::operator=(MyString && ms)noexcept{
	if(this != &ms){
		free();
		elements = ms.elements;
		ends = ms.ends;
		ms.elements = ms.ends = nullptr;
	}
	std::cout << "MyString& MyString::operator=(MyString && ms)" << std::endl;
	return *this;
}




```



`Test.cpp`

```cpp
#include <memory>
#include <vector>
#include <iostream>
#include "MyString.hpp"

void foo(MyString x)
{
    std::cout << x.begin() << std::endl;
}

void bar(const MyString& x)
{
    std::cout << x.begin() << std::endl;
}

MyString baz()
{
    MyString ret("world");
    return ret;
}

int main(){
//参考了 https://blog.csdn.net/qq_24739717/article/details/104473034#t48
    char text[] = "world";
	std::cout << "---------------定义-----------------" << std::endl;
    MyString s0;
    MyString s1("hello");
    MyString s2(s0);
    MyString s3 = s1;
    MyString s4(text);
    s2 = s1;

	std::cout << "---------------调用-----------------" << std::endl;
    foo(s1);
    bar(s1);
    foo("temporary");
    bar("temporary");
    MyString s5 = baz();
	std::cout << "---------------添加-----------------" << std::endl;
    std::vector<MyString> svec;
    svec.reserve(8);
    svec.push_back(s0);
    svec.push_back(s1);
    svec.push_back(s2);
    svec.push_back(s3);
    svec.push_back(s4);
    svec.push_back(s5);
    svec.push_back(baz());//避免拷贝
    svec.push_back("good job");//避免拷贝
	std::cout << "---------------输出-----------------" << std::endl;
    for (const auto &s : svec) {
        std::cout << s.begin() << std::endl;
    }
}
```



> 13.50 在你的 `String` 类的移动操作中添加打印语句，并重新运行13.6.1节的练习13.48中的程序，它使用了一个`vector<String>`，观察什么时候会避免拷贝。

```cpp
---------------定义-----------------
void MyString::range_initializer(const char *first, const char *last)
void MyString::range_initializer(const char *first, const char *last)
void MyString::range_initializer(const char *first, const char *last)
MyString::MyString(const MyString& ms)
void MyString::range_initializer(const char *first, const char *last)
MyString::MyString(const MyString& ms)
void MyString::range_initializer(const char *first, const char *last)
call MyString& MyString::operator=(const MyString &rhs)
---------------调用-----------------
void MyString::range_initializer(const char *first, const char *last)
MyString::MyString(const MyString& ms)
hello
hello
void MyString::range_initializer(const char *first, const char *last)
temporary
void MyString::range_initializer(const char *first, const char *last)
temporary
void MyString::range_initializer(const char *first, const char *last)
---------------添加-----------------
void MyString::range_initializer(const char *first, const char *last)
MyString::MyString(const MyString& ms)
void MyString::range_initializer(const char *first, const char *last)
MyString::MyString(const MyString& ms)
void MyString::range_initializer(const char *first, const char *last)
MyString::MyString(const MyString& ms)
void MyString::range_initializer(const char *first, const char *last)
MyString::MyString(const MyString& ms)
void MyString::range_initializer(const char *first, const char *last)
MyString::MyString(const MyString& ms)
void MyString::range_initializer(const char *first, const char *last)
MyString::MyString(const MyString& ms)
void MyString::range_initializer(const char *first, const char *last)
MyString::MyString(MyString && ms)
void MyString::range_initializer(const char *first, const char *last)
MyString::MyString(MyString && ms)
---------------输出-----------------

hello
hello
hello
world
world
world
good job
```



> 13.51 虽然 `unique_ptr` 不能拷贝，但我们在12.1.5节中编写了一个 `clone` 函数，它以值的方式返回一个 `unique_ptr`。解释为什么函数是合法的，以及为什么它能正确工作。

我们可以拷贝或赋值一个将要被销毁的`unique_ptr`最常见的例子是从函数返回一个`unique_ptr`

实际上是调用了移动构造函数或移动赋值运算符接管此函数中`unique_ptr`的所有权



> 13.52 详细解释第478页中的 `HasPtr` 对象的赋值发生了什么？特别是，一步一步描述 `hp`、`hp2` 以及 `HasPtr` 的赋值运算符中的参数 `rhs` 的值发生了什么变化。

左值拷贝，右值移动

`hp = hp2;`

* `hp2`是个左值，所以参数传递时通过拷贝构造函数来拷贝
* `rhs`为`hp2`的副本，两者独立，`string`内容相同。赋值结束后`rhs`被销毁



`hp = std::move(hp2);`

* `std::move`将一个右值引用绑定到`hp2`上，使用移动构造函数
* `rhs`接管`hp2`所有权



不管使用的是拷贝构造还是移动构造，赋值运算符的函数体都会`swap`两个运算对象的状态，交换两个对象指针后，`rhs`中的指针指向原来左侧运算对象所用的`string`。当`rhs`离开作用域时，这个`string`将被销毁



> 13.53 从底层效率的角度看，`HasPtr` 的赋值运算符并不理想，解释为什么？为 `HasPtr` 实现一个拷贝赋值运算符和一个移动赋值运算符，并比较你的新的移动赋值运算符中执行的操作和拷贝并交换版本中的执行的操作。

编译相关参数

```cpp
clang++ -Wall -std=c++14 -O2  -fsanitize=undefined,address
```

```cpp
#include <string>
#include <iostream>
#include <ctime>

class HasPtr {
public:
	friend void swap(HasPtr&, HasPtr&);
public:
	HasPtr(const std::string& s = std::string()):ps(new std::string(s)), i(0) { }//默认构造
	HasPtr(const HasPtr& hp):ps(new std::string(*hp.ps)),i(hp.i){}//拷贝构造
	HasPtr& operator=(HasPtr rhs){//拷贝赋值 swap版本
		swap(*this,rhs);
		return *this;
	}
	HasPtr(HasPtr && hp)noexcept:ps(hp.ps),i(hp.i){//移动构造
		hp.ps = nullptr;
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
}

int main(){
	HasPtr hp1("test1"),hp2("test2");
	clock_t start = clock();
	hp1 = hp2;
	clock_t end = clock();
	std::cout << end - start << std::endl;
	return 0;
}
```



```cpp
#include <string>
#include <iostream>
#include <ctime>

class HasPtr {
public:
	friend void swap(HasPtr&, HasPtr&);
public:
	HasPtr(const std::string& s = std::string()):ps(new std::string(s)), i(0) { }//默认构造
	HasPtr(const HasPtr& hp):ps(new std::string(*hp.ps)),i(hp.i){}//拷贝构造
	HasPtr(HasPtr && hp)noexcept:ps(hp.ps),i(hp.i){//移动构造
		hp.ps = nullptr;
	}
	HasPtr& operator=(const HasPtr& rhs){//拷贝赋值
		auto newp = new std::string(*rhs.ps);
		delete ps;
		ps = newp;
		i = rhs.i;
		return *this;
	}
	HasPtr& operator=(HasPtr&& rhs)noexcept{
		if(this != &rhs){
			delete ps;
			ps = rhs.ps;
			rhs.ps = nullptr;
		}
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
}

int main(){
	HasPtr hp1("test1"),hp2("test2");
	clock_t start = clock();
	hp1 = hp2;
	clock_t end = clock();
	std::cout << end - start << std::endl;
	return 0;
}
```

好像差距不大，甚至swap的更快？可能测的/写的有问题

参考其他前辈的题解说是`swap`操作交换时都会创建临时空间用来存临时变量的值，所以赋值运算效率低



> 13.54 如果我们为 `HasPtr` 定义了移动赋值运算符，但未改变拷贝并交换运算符，会发生什么？编写代码验证你的答案。

```cpp
#include <string>
#include <iostream>
#include <ctime>

class HasPtr {
public:
	friend void swap(HasPtr&, HasPtr&);
public:
	HasPtr(const std::string& s = std::string()):ps(new std::string(s)), i(0) { }//默认构造
	HasPtr(const HasPtr& hp):ps(new std::string(*hp.ps)),i(hp.i){}//拷贝构造
	HasPtr(HasPtr && hp)noexcept:ps(hp.ps),i(hp.i){//移动构造
		hp.ps = nullptr;
	}
	HasPtr& operator=(HasPtr rhs){//拷贝赋值 swap版本
		swap(*this,rhs);
		return *this;
	}
	HasPtr& operator=(HasPtr&& rhs)noexcept{
		if(this != &rhs){
			delete ps;
			ps = rhs.ps;
			rhs.ps = nullptr;
		}
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
}

int main(){
    HasPtr hp1("test"),*hp2 = new HasPtr("train");
    hp1 = std::move(*hp2);
	return 0;
}
```

报错

```cpp
error: use of overloaded operator '=' is ambiguous (with operand types 'HasPtr' and 'typename std::remove_reference<HasPtr &>::type' (aka 'HasPtr'))
```



### 13.6.3 右值引用和成员函数

> 13.55 为你的 `StrBlob` 添加一个右值引用版本的 `push_back`。

```cpp
void push_back(string &&t){data->push_back(std::move(t));}
```



> 13.56 如果 `sorted` 定义如下，会发生什么？

```cpp
Foo Foo::sorted() const & {
	Foo ret(*this);
	return ret.sorted();
}
```

`ret`由拷贝构造函数产生，是一个左值，循环调用

> 13.57 如果 `sorted`定义如下，会发生什么：

```cpp
Foo Foo::sorted() const & { return Foo(*this).sorted(); }
```

`Foo(*this)`是个临时量，是右值，会调用右值版本的`sorted()`

> 13.58 编写新版本的 `Foo` 类，其 `sorted` 函数中有打印语句，测试这个类，来验证你对前两题的答案是否正确。

```cpp
#include <algorithm>
#include <string>
#include <vector>
#include <iostream>

class Foo{
public:
	Foo sorted() &&;
	Foo sorted() const &;
private:
	std::vector<int> data;
};

Foo Foo::sorted()&&{
	sort(data.begin(),data.end());
	std::cout << "Foo Foo::sorted()&&" << std::endl;
	return *this;
}

Foo Foo::sorted() const &{
	Foo ret(*this);
	sort(ret.data.begin(),ret.data.end());
	std::cout << "Foo Foo:sorted() const &" << std::endl;
	return ret;
}

int main(){
	Foo().sorted();//调用右值版本，如果没有的写右值版本会调用左值版本的
	Foo f;
	f.sorted();//调用左值版本
	return 0;
}
```

输出

```cpp
Foo Foo::sorted()&&
Foo Foo:sorted() const &
```

