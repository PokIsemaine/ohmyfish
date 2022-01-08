<h1 align="center">📔 C++ Primer 0x09 练习题解</h1>

## 9.1 顺序容器概述

> 9.1 对于下面的程序任务，`vector`、`deque`和`list`哪种容器最为适合？解释你的选择的理由。如果没有哪一种容器优于其他容器，也请解释理由。
>
> (a) 读取固定数量的单词，将它们按字典序插入到容器中。我们将在下一章中看到，关联容器更适合这个问题。
>
> (b) 读取未知数量的单词，总是将单词插入到末尾。删除操作在头部进行。
>
> (c) 从一个文件读取未知数量的整数。将这些数排序，然后将它们打印到标准输出。

* (a)`list`插入位置任意，不一定在首尾
* (b)`deque`插入删除操作在头尾
* (c)`vector`不需要插入删除，而且要排序



## 9.2 容器库概览

> 9.2 定义一个`list`对象，其元素类型是`int`的`deque`。

```cpp
std::list<std::deque<int>> ls;
```



### 9.2.1 迭代器

> 9.3 构成迭代器范围的迭代器有何限制？

* 指向同一个容器中的元素或者是容器元素最后一个元素的位置

* end 不在 begin之前



> 9.4 编写函数，接受一对指向`vector<int>`的迭代器和一个`int`值。在两个迭代器指定的范围中查找给定的值，返回一个布尔值来指出是否找到。

```cpp
#include <vector>
#include <iostream>

using  Iter = std::vector<int>::iterator;

bool find(Iter beg, Iter end, int x){
	while(beg!=end){
		if((*beg)==x)return true;
		++beg;
	}
	return false;
}

int main(){
	std::vector<int>vec={1,2,3,4,5,6,7};
	if(find(vec.begin(),vec.end(),3)){
		std::cout << "yes" << std::endl;
	}else{
		std::cout << "no" << std::endl;
	}
	return 0;
}
```



> 9.5 重写上一题的函数，返回一个迭代器指向找到的元素。注意，程序必须处理未找到给定值的情况。

```cpp
#include <vector>
#include <iostream>

using  Iter = std::vector<int>::iterator;

Iter find(Iter beg, Iter end, int x){
	while(beg!=end){
		if((*beg)==x)break;
		++beg;
	}
	return beg;
}

int main(){
	std::vector<int>vec={1,2,3,4,5,6,7};
	Iter ret = find(vec.begin(),vec.end(),3);
	if(ret == vec.end()){
		std::cout << "no" << std::endl;
	}else{
		std::cout << *ret << " " << ret-vec.begin() << std::endl;
	}
	return 0;
}
```



> 9.6 下面的程序有何错误？你应该如何修改它？

```cpp
list<int> lst1;
list<int>::iterator iter1 = lst1.begin(),
					iter2 = lst1.end();
while (iter1 < iter2) /* ... */
```

与 `vector` 和 `deque` 不同，`list` 的迭代器不支持 `<` 运算，只支持递增、递减、`=` 以及 `!=` 运算。

```cpp
while(iter1 != iter2)
```



### 9.2.2 容器类成员

> 9.7 为了索引`int`的`vector`中的元素，应该使用什么类型？

`std::vector<int>::size_type`



> 9.8 为了读取`string`的`list`中的元素，应该使用什么类型？如果写入`list`，又应该使用什么类型？

读`std::list<string>::const_iterator`

写`std::list<string>::iterator`



### 9.2.3 begin 和 end 成员

> 9.9 `begin`和`cbegin`两个函数有什么不同？

`begin`返回普通迭代器

`cbegin`返回常量迭代器



> 9.10 下面4个对象分别是什么类型？

```cpp
vector<int> v1;
const vector<int> v2;
auto it1 = v1.begin(), it2 = v2.begin();
auto it3 = v1.cbegin(), it4 = v2.cbegin();
```

* it1和it2`vector<int>::iterator`
* it3和it4`vector<int>::const_iterator`



### 9.2.4 容器定义和初始化

> 9.11 对6种创建和初始化`vector`对象的方法，每一种都给出一个实例。解释每个`vector`包含什么值。

```cpp
vector<int> vec1;//空
vector<int> vec2(10);//10个0
vector<int> vec3(10,1);//10个1
vector<int> vec4{1,2,3,4,5,6,7,8,9,10};//1～10
vector<int> vec5(vec4);//1～10
vector<int> vec6(vec5.begin(),vec5.end());//1～10
```



> 9.12 对于接受一个容器创建其拷贝的构造函数，和接受两个迭代器创建拷贝的构造函数，解释它们的不同。

* 将一个容器创建为另一个容器的拷贝方法有两种

	* 直接拷贝整个容器，两个容器的类型和元素类型必须匹配
	* 拷贝由一个迭代器对指定的元素范围（`array`除外），不要求容器类型相同，元素类型也可以不同，只要能将拷贝的元素转换



> 9.13 如何从一个`list<int>`初始化一个`vector<double>`？从一个`vector<int>`又该如何创建？编写代码验证你的答案。

```cpp
#include <vector>
#include <iostream>
#include <list>
using namespace std;

int main(){
	list<int>ls{1,2,3,4};
	vector<int>vec{5,6,7,8};
	
	vector<double>dvec1(ls.begin(),ls.end());
	vector<double>dvec2(vec.begin(),vec.end());
	
	for(auto v:dvec1)cout << v << " ";
	cout << endl;
	
	for(auto v:dvec2)cout << v << " ";
	cout << endl;
	return 0;
}
```



### 9.2.5 赋值和 swap

> 9.14 编写程序，将一个`list`中的`char *`指针元素赋值给一个`vector`中的`string`。

```cpp
list<char*>ls{"abc","def","ghi"};
vector<string>vec;
vec.assign(ls.cbegin(),ls.cend());
```



### 9.2.7 关系运算符

> 9.15 编写程序，判定两个`vector<int>`是否相等。

```cpp
#include <vector>
#include <iostream>
#include <list>
using namespace std;

int main(){
	vector<int>vec1{1,2,3,4,5};
	vector<int>vec2{1,2,3,4,5};
	vector<int>vec3{1,2,3,5,4};
	
	cout << (vec1==vec2?"yes":"no") << endl;
	cout << (vec1==vec3?"yes":"no") << endl;
	return 0;
}
```



> 9.16 重写上一题的程序，比较一个list中的元素和一个vector中的元素。

```cpp
#include <vector>
#include <iostream>
#include <list>
using namespace std;

int main(){
	vector<int>vec1{1,2,3,4,5};
	list<int>ls1{1,2,3,4,5};
	list<int>ls2{1,2,3,5,4};
	
	cout << (vec1==vector<int>(ls1.begin(),ls1.end())?"yes":"no") << endl;
	cout << (vec1==vector<int>(ls2.begin(),ls2.end())?"yes":"no") << endl;
	return 0;
}
```



> 9.17 假定`c1`和`c2`是两个容器，下面的比较操作有何限制？(**如果有的话**)

```cpp
if (c1 < c2)
```

* 只有当其中元素类型也定义了相应的比较运算符时，我们才可以使用关系运算符来比较两个容器
* 关系运算符两侧运算对象必须是类型相同的容器且必须保存相同类型的元素



## 9.3 顺序容器操作

### 9.3.1 向顺序容器添加元素

> 9.18 编写程序，从标准输入读取`string`序列，存入一个`deque`中。编写一个循环，用迭代器打印`deque`中的元素。

```cpp
#include <iostream>
#include <string>
#include <deque>
using namespace std;

int main(){
	deque<string>dq;
	string s;
	while(cin>>s)dq.push_back(s);
	for(auto iter=dq.cbegin();iter!=dq.cend();++iter){
		cout << *iter << " ";
	}
	return 0;
}
```



> 9.19 重写上一题的程序，用`list`替代`deque`。列出程序要做出哪些改变。

```cpp
#include <iostream>
#include <list>
#include <string>
using namespace std;

int main(){
	list<string>ls;
	string s;
	while(cin>>s)ls.push_back(s);
	for(auto iter=ls.cbegin();iter!=ls.cend();++iter){
		cout << *iter << " ";
	}
	return 0;
}
```



> 9.20 编写程序，从一个`list<int>`拷贝元素到两个`deque`中。值为偶数的所有元素都拷贝到一个`deque`中，而奇数值元素都拷贝到另一个`deque`中。

```cpp
#include <iostream>
#include <list>
#include <deque>
using namespace std;

int main(){
	list<int>ls={1,2,3,4,5,6,7,8};
	deque<int>odd,even;
	for(auto e:ls){
		if(e&1)odd.push_back(e);
		else even.push_back(e);
	}
	
	for(auto e:odd)cout << e << " ";
	cout << endl;
	for(auto e:even)cout << e << " ";
	
	return 0;
}
```



> 9.21 如果我们将第308页中使用`insert`返回值将元素添加到`list`中的循环程序改写为将元素插入到`vector`中，分析循环将如何工作。

没啥区别

`insert`返回值指向插入的新元素，可以借助`insert`实现`push_front`相同的功能



> 9.22 假定`iv`是一个`int`的`vector`，下面的程序存在什么错误？你将如何修改？

```cpp
vector<int>::iterator iter = iv.begin(),
					  mid = iv.begin() + iv.size() / 2;
while (iter != mid)
	if (*iter == some_val)
		iv.insert(iter, 2 * some_val);
```

* 循环里迭代器没变，死循环
* 循环里插入操作，迭代器没有更新，迭代器会失效

几个要点

插入后`mid`会失效，要更新，保证还是初始中点终止

插入元素后要多移动一步

迭代器也会失效，要更新

```cpp
#include <iostream>
#include <list>
#include <vector>
using namespace std;

int main(){
	vector<int>iv={5,4,4,4,3,1};
	int some_value = 4;
	int move = 0;
	vector<int>::iterator iter = iv.begin();
	int siz=iv.size();
	while(iter != (iv.begin() + siz/2+move)){
		if(*iter == some_value){
			iter = iv.insert(iter,some_value*2);
			move++;
			iter++;iter++;
		}
		else ++iter;
	}
	for(auto v:iv)cout << v << " ";
		
	return 0;
}
```

### 9.3.2 访问元素

> 9.23 在本节第一个程序中，若`c.size()` 为1，则`val`、`val2`、`val3`和`val4`的值会是什么？

都是容器里唯一的那个元素的值



> 9.24 编写程序，分别使用`at`、下标运算符、`front` 和 `begin` 提取一个`vector`中的第一个元素。在一个空`vector`上测试你的程序。

```cpp
#include <iostream>
#include <vector>

int main(){
    std::vector<int> vec;
    
    std::cout << vec.at(0);
    //terminate called after throwing an instance of 'std::out_of_range       
    std::cout << vec[0];//段错误        
    std::cout << vec.front();//段错误    
    std::cout << *vec.begin();//段错误  
      
    return 0;
}
```



### 9.3.3 删除元素

> 9.25 对于第312页中删除一个范围内的元素的程序，如果 `elem1` 与 `elem2` 相等会发生什么？如果 `elem2` 是尾后迭代器，或者 `elem1` 和 `elem2` 皆为尾后迭代器，又会发生什么？

* 如果 `elem1` 与 `elem2` 相等，迭代器范围为空，什么都不做
* 如果 `elem2` 是尾后迭代器，从`elem1`开始删到末尾
* `elem1` 和 `elem2` 皆为尾后迭代器，什么都不做



> 9.26 使用下面代码定义的`ia`，将`ia`拷贝到一个`vector`和一个`list`中。是用单迭代器版本的`erase`从`list`中删除奇数元素，从`vector`中删除偶数元素。

```cpp
int ia[] = { 0, 1, 1, 2, 3, 5, 8, 13, 21, 55, 89 };
```



```cpp
#include <iostream>
#include <vector>
#include <list>

int main(){
	int ia[] = { 0, 1, 1, 2, 3, 5, 8, 13, 21, 55, 89 };
    
    std::vector<int>vec;
    std::list<int>ls;
    
    for(auto i:ia)vec.push_back(i);
    for(auto i:ia)ls.push_back(i);

	auto it1=vec.begin();
	auto it2=ls.begin();
	
	while(it1!=vec.end()){
		if((1&(*it1))==0){
			it1=vec.erase(it1);
		}else{
			++it1;
		}
	}

	
	while(it2!=ls.end()){
		if(1&(*it2)){
			it2=ls.erase(it2);
		}else{
			++it2;
		}
	}
	

	for(auto i:vec)std::cout << i << " ";
	std::cout << std::endl;
	for(auto i:ls)std::cout << i << " ";
	
    return 0;
}
```

###  9.3.4 特殊的 forward_list  操作

> 9.27 编写程序，查找并删除`forward_list<int>`中的奇数元素。

```cpp
#include <iostream>
#include <forward_list>

int main(){
	std::forward_list<int>flst{1,2,3,4,5};
	auto prev = flst.before_begin();
	auto curr = flst.begin();
	while(curr != flst.end()){
		if(*curr%2)
			curr = flst.erase_after(prev);
		else{
			prev = curr;
			++curr;
		}
	}
	for(auto i:flst)std::cout << i << " ";
    return 0;
}
```



> 9.28 编写函数，接受一个`forward_list<string>`和两个`string`共三个参数。函数应在链表中查找第一个`string`，并将第二个`string`插入到紧接着第一个`string`之后的位置。若第一个`string`未在链表中，则将第二个`string`插入到链表末尾。

```cpp
#include <iostream>
#include <forward_list>

using std::string;
using FL = std::forward_list<string>;

void fun(FL& flst,string a,string b){
	auto prev = flst.before_begin();
	auto curr = flst.begin();	
	while(curr != flst.end()){
		if(*curr == a){
			flst.insert_after(curr,b);
			return;
		}
		prev = curr;
		++curr;
	}
	return;
}

int main(){
	FL flst={"abc","def","ghi"};
	fun(flst,"abc","ABC");
	for(auto i:flst)std::cout << i << " ";
    return 0;
}
```

### 9.3.5 改变容器大小

> 9.29 假定`vec`包含25个元素，那么`vec.resize(100)`会做什么？如果接下来调用`vec.resize(10)`会做什么？

`vec.resize(100)`末尾加75个0(如果是int)

`vec.resize(10)`删除末尾90个元素



> 9.30 接受单个参数的`resize`版本对元素类型有什么限制（如果有的话）？

元素类型必须有默认构造函数



### 9.3.6 容器操作可能使迭代器失效

> 9.31 第316页中删除偶数值元素并复制奇数值元素的程序不能用于`list`或`forward_list`。为什么？修改程序，使之也能用于这些类型。

```cpp
iter+=2;//复合赋值语句不能用于这两个
++iter;++iter;//改为这个
```

`forward_list`还需要借助增加一个`prev`来完成增删



> 9.32 在第316页的程序中，向下面语句这样调用`insert`是否合法？如果不合法，为什么？

```cpp
iter = vi.insert(iter, *iter++);
```

不合法，参数求职顺序不确定



> 9.33 在本节最后一个例子中，如果不将`insert`的结果赋予`begin`，将会发生什么？编写程序，去掉此赋值语句，验证你的答案。

迭代器失效



> 9.34 假定`vi`是一个保存`int`的容器，其中有偶数值也有奇数值，分析下面循环的行为，然后编写程序验证你的分析是否正确。

```cpp
iter = vi.begin();
while (iter != vi.end())
	if (*iter % 2)
		iter = vi.insert(iter, *iter);
	++iter;
```

不正确，死循环了

```cpp
iter = vi.begin();
while (iter != vi.end())
	if (*iter % 2){
			iter = vi.insert(iter, *iter);
			++iter;++iter:
	}else{
		++iter;
	}
```



## 9.4 vector 对象是如何快速增长的

> 9.35 解释一个`vector`的`capacity`和`size`有何区别。

`capacity`是在不分配新的内存空间的前提下最多可以保存多少元素，`size`是指已经保存的元素的数目



> 9.36 一个容器的`capacity`可能小于它的`size`吗？

不可能



> 9.37 为什么`list`或`array`没有`capacity`成员函数？

`list`链表没有数量限制

`array`大小固定了





> 9.38 编写程序，探究在你的标准实现中，`vector`是如何增长的。

```cpp
#include <iostream>
#include <string>
#include <vector>

using namespace std;

int main()
{
	vector<int> v;

	for (int i = 0; i < 100; i++)
	{
		cout << "capacity: " << v.capacity() << "  size: " << v.size() << endl;
		v.push_back(i);
	}
	return 0;
}
```

`clang version 13.0.0`

```
capacity: 0  size: 0
capacity: 1  size: 1
capacity: 2  size: 2
capacity: 4  size: 3
capacity: 4  size: 4
capacity: 8  size: 5
capacity: 8  size: 6
capacity: 8  size: 7
capacity: 8  size: 8
capacity: 16  size: 9
capacity: 16  size: 10
capacity: 16  size: 11
capacity: 16  size: 12
capacity: 16  size: 13
capacity: 16  size: 14
capacity: 16  size: 15
capacity: 16  size: 16
capacity: 32  size: 17
capacity: 32  size: 18
capacity: 32  size: 19
capacity: 32  size: 20
capacity: 32  size: 21
capacity: 32  size: 22
capacity: 32  size: 23
capacity: 32  size: 24
capacity: 32  size: 25
capacity: 32  size: 26
capacity: 32  size: 27
capacity: 32  size: 28
capacity: 32  size: 29
capacity: 32  size: 30
capacity: 32  size: 31
capacity: 32  size: 32
capacity: 64  size: 33
capacity: 64  size: 34
capacity: 64  size: 35
capacity: 64  size: 36
capacity: 64  size: 37
capacity: 64  size: 38
capacity: 64  size: 39
capacity: 64  size: 40
capacity: 64  size: 41
capacity: 64  size: 42
capacity: 64  size: 43
capacity: 64  size: 44
capacity: 64  size: 45
capacity: 64  size: 46
capacity: 64  size: 47
capacity: 64  size: 48
capacity: 64  size: 49
capacity: 64  size: 50
capacity: 64  size: 51
capacity: 64  size: 52
capacity: 64  size: 53
capacity: 64  size: 54
capacity: 64  size: 55
capacity: 64  size: 56
capacity: 64  size: 57
capacity: 64  size: 58
capacity: 64  size: 59
capacity: 64  size: 60
capacity: 64  size: 61
capacity: 64  size: 62
capacity: 64  size: 63
capacity: 64  size: 64
capacity: 128  size: 65
capacity: 128  size: 66
capacity: 128  size: 67
capacity: 128  size: 68
capacity: 128  size: 69
capacity: 128  size: 70
capacity: 128  size: 71
capacity: 128  size: 72
capacity: 128  size: 73
capacity: 128  size: 74
capacity: 128  size: 75
capacity: 128  size: 76
capacity: 128  size: 77
capacity: 128  size: 78
capacity: 128  size: 79
capacity: 128  size: 80
capacity: 128  size: 81
capacity: 128  size: 82
capacity: 128  size: 83
capacity: 128  size: 84
capacity: 128  size: 85
capacity: 128  size: 86
capacity: 128  size: 87
capacity: 128  size: 88
capacity: 128  size: 89
capacity: 128  size: 90
capacity: 128  size: 91
capacity: 128  size: 92
capacity: 128  size: 93
capacity: 128  size: 94
capacity: 128  size: 95
capacity: 128  size: 96
capacity: 128  size: 97
capacity: 128  size: 98
capacity: 128  size: 99

```



> 9.39 解释下面程序片段做了什么：

```cpp
vector<string> svec;
svec.reserve(1024);//预先分配1024个元素空间
string word;
while (cin >> word)
	svec.push_back(word);
svec.resize(svec.size() + svec.size() / 2);//元素数量变为原来1.5倍
//如果元素数量变 1.5 倍后超过1024则重新分配空间
```



> 9.40 如果上一题的程序读入了256个词，在`resize`之后容器的`capacity`可能是多少？如果读入了512个、1000个、或1048个呢？

* 256：`capacity=1024`
* 512：`capacity=1024`
* 1000,1048：如果两倍增长，那么`capacity=2048`



## 9.5 额外的 string 操作

### 9.5.1 构造 string 的其他方法

> 9.41 编写程序，从一个`vector<char>`初始化一个`string`。

```cpp
vector<char>vec={'a','b','c'};
string s(vec.begin(),vec.end());
```



> 9.42 假定你希望每次读取一个字符存入一个`string`中，而且知道最少需要读取100个字符，应该如何提高程序的性能？

使用 `reserve(100)` 函数预先分配100个元素的空间。



### 9.5.2 改变 string 的其他方法

> 9.43 编写一个函数，接受三个`string`参数是`s`、`oldVal` 和`newVal`。使用迭代器及`insert`和`erase`函数将`s`中所有`oldVal`替换为`newVal`。测试你的程序，用它替换通用的简写形式，如，将"tho"替换为"though",将"thru"替换为"through"。

```cpp
#include <iostream>
#include <string>
#include <vector>

using namespace std;

void fun(string &s,const string &oldVal,const string &newVal){
	for(auto it = s.begin(); it != s.end();){
		if(oldVal == s.substr(it-s.begin(),oldVal.size())){
			it = s.erase(it,it+oldVal.size());
			it = s.insert(it,newVal.begin(),newVal.end());
			it += newVal.size();
		}else{
			++it;
		}
	}	
}

int main(){
	string s{"123456789"};
	fun(s,"123","567");
	fun(s,"abc","def");
	cout << s << endl;
	return 0;
}
```



> 9.44 重写上一题的函数，这次使用一个下标和`replace`。

```cpp
#include <iostream>
#include <string>
#include <vector>

using namespace std;

void fun(string &s,const string &oldVal,const string &newVal){
	for(size_t pos = 0; pos+oldVal.size() <= s.size(); ++pos){
		if(s.substr(pos,oldVal.size())==oldVal){
			s.replace(pos,oldVal.size(),newVal);
			pos += newVal.size();
		}else{
			++pos;
		}
	}
}

int main(){
	string s{"123456789"};
	fun(s,"123","567");
	fun(s,"abc","def");
	cout << s << endl;
	return 0;
}
```



> 9.45 编写一个函数，接受一个表示名字的`string`参数和两个分别表示前缀（如"Mr."或"Mrs."）和后缀（如"Jr."或"III"）的字符串。使用迭代器及`insert`和`append`函数将前缀和后缀添加到给定的名字中，将生成的新`string`返回。

```cpp
#include <iostream>
#include <string>
#include <vector>

using namespace std;

string fun(string& name,const string& pre,const string& suf){
	name.insert(name.begin(),pre.begin(),pre.end());
	name.append(suf);
	return name;
}

int main(){
	string name("Fishingrod");
	cout << fun(name,"Mr.",".zsl");
	return 0;
}
```



> 9.46 重写上一题的函数，这次使用位置和长度来管理`string`，并只使用`insert`。

```cpp
#include <iostream>
#include <string>
#include <vector>

using namespace std;

string fun(string& name,const string& pre,const string& suf){
	name.insert(0,pre);
	name.insert(name.size(),suf);
	return name;
}

int main(){
	string name("Fishingrod");
	cout << fun(name,"Mr.",".zsl");
	return 0;
}
```



### 9.5.3 string 搜索操作

> 9.47 编写程序，首先查找`string`"ab2c3d7R4E6"中每个数字字符，然后查找其中每个字母字符。编写两个版本的程序，第一个要使用`find_first_of`，第二个要使用`find_first_not_of`。

```cpp
#include <iostream>
#include <string>
#include <vector>

using namespace std;

int main(){
	string s("ab2c3d7R4E6");
	string num("0123456789");
	for(int i=0;s[i];){
		auto pos = s.substr(i).find_first_not_of(num);
		if(pos!=string::npos){
			cout << s[pos+i];
			i = pos+1+i;
		}else{
			++i;
		}
	}
	cout << endl;
	
	for(int i=0;s[i];){
		auto pos = s.substr(i).find_first_of(num);
		if(pos==string::npos){
			cout << s[i++];
		}else{
			cout <<s.substr(i,pos);
			i = pos+i+1; 
		}
	}
	return 0;
}
```



> 9.48 假定`name`和`numbers`的定义如325页所示，`numbers.find(name)`返回什么？

`string::npos`



> 9.49 如果一个字母延伸到中线之上，如d或f，则称其有上出头部分（`ascender`）。如果一个字母延伸到中线之下，如p或g，则称其有下出头部分（`descender`）。编写程序，读入一个单词文件，输出最长的既不包含上出头部分，也不包含下出头部分的单词。

```cpp
#include <fstream>
#include <iostream>
#include <string>

std::string find(const std::string& s,std::string& longest){
	std::string str("aceimnorsuvwxz");
	if(s.find_first_not_of(str)==std::string::npos){
		return (s.size()>longest.size())?s:longest;
	}
	return longest;
}
int main(int argc,char **argv){
	std::ifstream ifs(argv[1]);
	if(ifs){
		std::string longest,s;
		while(ifs>>s){
			longest = find(s,longest);
		}
		std::cout << longest << std::endl;
	}
	return 0;
}
```

### 9.5.5 数值转换

> 9.50 编写程序处理一个`vector<string>`，其元素都表示整型值。计算`vector`中所有元素之和。修改程序，使之计算表示浮点值的`string`之和。

```cpp
#include <fstream>
#include <iostream>
#include <string>
#include <vector>

using std::vector;
using std::string;
using std::cout;
using std::cin;

int main(){
	vector<string>v;
	for(string s;cin>>s;v.push_back(s));
	int sum=0;
	for(auto i:v)sum+=std::stoi(i);
	cout << sum;
	return 0;
}
```



```cpp
#include <fstream>
#include <iostream>
#include <string>
#include <vector>

using std::vector;
using std::string;
using std::cout;
using std::cin;

int main(){
	vector<string>v;
	for(string s;cin>>s;v.push_back(s));
	double sum=0;
	for(auto i:v)sum+=std::stod(i);
	cout << sum;
	return 0;
}
```



> 9.51 设计一个类，它有三个`unsigned`成员，分别表示年、月和日。为其编写构造函数，接受一个表示日期的`string`参数。你的构造函数应该能处理不同的数据格式，如January 1,1900、1/1/1990、Jan 1 1900 等。

```cpp
#include <iostream>
#include <string>

class Time{
private:
	unsigned year;
	unsigned month;
	unsigned day;

public:
	Time() = default;
	Time(std::string s){
		if(s.find_first_of("/")!=std::string::npos){//1/1/1990
			day = stoi(s.substr(0,s.find_first_of("/")));
			month = stoi(s.substr(s.find_first_of("/")+1,
				(s.find_last_of('/')-s.find_first_of('/'))));
			year = stoi(s.substr(s.find_last_of("/")+1,4));
		}else{
			if( s.find("Jan") < s.size() )  month = 1;
			if( s.find("Feb") < s.size() )  month = 2;
			if( s.find("Mar") < s.size() )  month = 3;
			if( s.find("Apr") < s.size() )  month = 4;
			if( s.find("May") < s.size() )  month = 5;
			if( s.find("Jun") < s.size() )  month = 6;
			if( s.find("Jul") < s.size() )  month = 7;
			if( s.find("Aug") < s.size() )  month = 8;
			if( s.find("Sep") < s.size() )  month = 9;
			if( s.find("Oct") < s.size() )  month =10;
			if( s.find("Nov") < s.size() )  month =11;
			if( s.find("Dec") < s.size() )  month =12;
			
			if(s.find_first_of(" ")!=std::string::npos
			&&s.find_first_of(" ")>3){//January 1,1900
				s[s.find(",")]=' ';
			}

			day = stoi(s.substr(s.find_first_of(" ")+1,
				(s.find_last_of(' ')-s.find_first_of(' '))));
			year = stoi(s.substr(s.find_last_of(" ")+1,4));
		}
	} 

public:
	void print(){
		std::cout << year << " " 
				<< month << " " 
				<< day << std::endl;
	}
};

int main(){
	Time t1("January  1,1900");
	Time t2("Jan 1 1900");
	Time t3("1/1/1900");
	
	t1.print();
	t2.print();
	t3.print();

	return 0;
}
```



## 9.6 容器适配器

> 9.52 使用`stack`处理括号化的表达式。当你看到一个左括号，将其记录下来。当你在一个左括号之后看到一个右括号，从`stack`中`pop`对象，直至遇到左括号，将左括号也一起弹出栈。然后将一个值（括号内的运算结果）`push`到栈中，表示一个括号化的（子）表达式已经处理完毕，被其运算结果所替代。

emm不完整版的栈算表达式？下面这个直接是表达式求值的

```cpp
#include<bits/stdc++.h>
using namespace std;

stack<int> num;
stack<char> op;

void eval(){
    auto b=num.top();num.pop();
    auto a=num.top();num.pop();
    auto c=op.top();op.pop();
    int x=0;
    if(c=='+')x=a+b;
    else if(c=='-')x=a-b;
    else if(c=='*')x=a*b;
    else x=a/b;
    num.push(x);
}
int main()
{
    unordered_map<char,int>pr{{'+',1},{'-',1},{'*',2},{'/',2},{'(',0}};//运算符优先级
    string s;
    cin>>s;
    for(int i=0;i<s.size();i++)
    {
        auto c=s[i];
        if(isdigit(c))//转数字
        {
            int x=0,j=i;
            while(j<s.size()&&isdigit(s[j]))
                x=x*10+s[j++]-'0';
            i=j-1;
            num.push(x);
        }
        else if(c=='(')op.push(c);
        else if(c==')')//括号直接算
        {
            while(op.top()!='(')eval();
            op.pop();
        }
        else
        {
            while(op.size()&&pr[op.top()]>=pr[c])eval();//如果当前优先级比前面的低就算
            op.push(c);//操作符入栈
        }
    }
    while(op.size())eval();//剩余计算
    printf("%d",num.top());
    return 0;
}

```

