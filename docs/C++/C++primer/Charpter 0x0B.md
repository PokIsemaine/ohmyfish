<h1 align="center">📔 C++ Prime 0x0B 练习题解</h1>

## 11.1 使用关联容器

> 11.1 描述`map`和`vector`的不同

* `map`是关联容器，`vector`是顺序容器
* 关联容器不支持顺序容器的位置相关的操作如`push_front`或`push_back`
* 关联容器的迭代器都是双向的，`vector`是随机迭代器



> 11.2 分别给出最适合使用`list`、`vector`、`deque`、`map`以及`set`的例子。

- `list`：双向链表，适合频繁插入删除元素的场景。
- `vector`：适合频繁访问元素的场景。
- `deque`：双端队列，适合频繁在头尾插入删除元素的场景。
- `map`：字典。
- `set`：适合有序不重复的元素的场景。



> 11.3 编写你自己的单词计数程序。

```cpp
#include <map>
#include <string>
#include <iostream>

int main(){
	std::map<std::string,int>mp;
	std::string s;
	
	while(std::cin >> s)mp[s]++;
	
	for(const auto& e:mp){
		std::cout << e.first << " " << e.second << std::endl;
	}
	return 0;
}
```



> 11.4 扩展你的程序，忽略大小写和标点。例如，"example."、"example,"和"Example"应该递增相同的计数器。

```cpp
#include <cctype>
#include <map>
#include <string>
#include <iostream>
#include <algorithm>

int main(){
	std::map<std::string,int>mp;
	std::string s;
	
	while(std::cin >> s){
		for(auto &c:s)c = tolower(c);
		s.erase(std::remove_if(s.begin(), s.end(),ispunct),s.end());
		++mp[s];
	}
	
	for(const auto& e:mp){
		std::cout << e.first << " " << e.second << std::endl;
	}
	return 0;
}
```



## 11.2 关联容器概述

### 11.2.1 定义关联容器

> 11.5 解释`map`和`set`的区别。你如何选择使用哪个？

* 定义一个`map`，必须既指明关键字类型又指明值类型
* 定义一个`set`，只需指明关键字类型，因为`set`中没有值

选择

- `map`：字典。
- `set`：适合有序不重复的元素的场景。



> 11.6 解释`set`和`list`的区别。你如何选择使用哪个？

`set` 关联容器，是**有序**不重复集合，底层实现是红黑树

 `list` 顺序容器，是**无序**可重复集合，底层实现是链表



> 11.7 定义一个`map`，关键字是家庭的姓，值是一个`vector`，保存家中孩子（们）的名。编写代码，实现添加新的家庭以及向已有家庭中添加新的孩子。

```cpp
#include <cctype>
#include <map>
#include <string>
#include <iostream>
#include <algorithm>
#include <vector>

int main(){
	std::map<std::string,std::vector<std::string>>mp;
	std::string family_name,children_name;
	while(std::cin >> family_name >> children_name){
		mp[family_name].push_back(children_name);
	}
	return 0;
}
```



> 11.8 编写一个程序，在一个`vector`而不是一个`set`中保存不重复的单词。使用`set`的优点是什么？

```cpp
#include <cctype>
#include <map>
#include <string>
#include <iostream>
#include <algorithm>
#include <vector>

int main(){
	std::vector<std::string>words;
	std::string s;

	while(std::cin >> s)words.push_back(s);

	std::sort(words.begin(),words.end());
	words.erase(std::unique(words.begin(),words.end()),words.end());
	
	for(const auto &s:words)std::cout << s << std::endl;
	
	return 0;
}
```

`set` 本身就是**有序**不重复集合



### 11.2.2 关键字类型的要求

> 11.9 定义一个`map`，将单词与一个行号的`list`关联，`list`中保存的是单词所出现的行号。

```cpp
std::map<std::string, std::list<std::size_t>> mp;
```



> 11.10 可以定义一个`vector<int>::iterator` 到 `int` 的`map`吗？`list<int>::iterator` 到 `int` 的`map`呢？对于两种情况，如果不能，解释为什么。

 `vector<int>::iterator` 到 `int` 的`map`，可以定义

 `list<int>::iterator` 到 `int` 的`map`，不可以定义



有序容器要求所提供的操作必须在关键字类型上定义一个严格弱序（可以自己定义）

因为`map`的键必须实现 `<` 操作，`list` 的迭代器不支持比较运算。



> 11.11 不使用`decltype` 重新定义 `bookstore`。

```cpp
multiset<Sales_data,bool (*)(const Sale_data&,const Sales_data&)>bookstore(compareIsbn);
```



### 11.2.3 pair 类型

> 11.12 编写程序，读入`string`和`int`的序列，将每个`string`和`int`存入一个`pair` 中，`pair`保存在一个`vector`中。

```cpp
#include <cctype>
#include <map>
#include <string>
#include <iostream>
#include <algorithm>
#include <vector>
#include <utility>

int main(){
	std::string s;int i;
	std::vector<std::pair<std::string,int>>v;

	while(std::cin >> s >> i){
		v.push_back(make_pair(s,i));
	}
	for(auto p:v){
		std::cout << p.first << " " << p.second << std::endl;
	}
	return 0;
}
```



> 11.13 在上一题的程序中，至少有三种创建`pair`的方法。编写此程序的三个版本，分别采用不同的方法创建`pair`。解释你认为哪种形式最易于编写和理解，为什么？

```cpp
v.push_back(make_pair(s,i));
v.push_back({s,i});//最容易理解
v.push_back(std::pair<std::string,int>(s,i));
```



> 11.14 扩展你在11.2.1节练习中编写的孩子姓达到名的`map`，添加一个`pair`的`vector`，保存孩子的名和生日。

```cpp
#include <cctype>
#include <map>
#include <string>
#include <iostream>
#include <algorithm>
#include <vector>
#include <utility>

int main(){
	std::map<std::string,std::vector<std::pair<std::string,std::string>>>mp;
	std::string family_name,children_name,birthday;
	while(std::cin >> family_name >> children_name >> birthday){
		mp[family_name].push_back({children_name,birthday});
	}
	return 0;
}
```



## 11.3 关联容器操作

### 11.3.1 关联容器迭代器

> 11.15 对一个`int`到`vector<int>的map`，其`mapped_type`、`key_type`和 `value_type`分别是什么？

* `mapped_type`：`vector<int>的map`
* `key_type`：`int`
* `value_type`：`pair<const int,vector<int>`



> 11.16 使用一个`map`迭代器编写一个表达式，将一个值赋予一个元素。

```cpp
#include <cctype>
#include <map>
#include <string>
#include <iostream>
#include <algorithm>
#include <vector>
#include <utility>

int main(){
	std::map<int,int>mp={{1,10},{2,9},{3,8}};
	std::map<int,int>::iterator iter = mp.begin();
	std::cout << mp[1] << std::endl;
	(*iter).second = 1;
	std::cout << mp[1] << std::endl;
	iter->second = 2;
	std::cout << mp[1] << std::endl;

	return 0;
}
```



> 11.17 假定`c`是一个`string`的`multiset`，`v` 是一个`string` 的`vector`，解释下面的调用。指出每个调用是否合法：

```cpp
copy(v.begin(), v.end(), inserter(c, c.end()));
copy(v.begin(), v.end(), back_inserter(c));//不合法，multiset关联容器不支持push_back
copy(c.begin(), c.end(), inserter(v, v.end()));
copy(c.begin(), c.end(), back_inserter(v));
```



> 11.18 写出第382页循环中`map_it` 的类型，不要使用`auto` 或 `decltype`。

`map<std::string,std::size_t>::const_iterator`

注意迭代器是不能加`const`的，不要写成下面这样

`cosnt map<std::string,std::size_t>::iterator`



> 11.19 定义一个变量，通过对11.2.2节中的名为 `bookstore` 的`multiset` 调用`begin()`来初始化这个变量。写出变量的类型，不要使用`auto` 或 `decltype`。

```cpp
using compareType = bool (*)(const Sales_data &, const Sales_data &);
std::multiset<Sales_data, compareType> bookstore(compareIsbn);
std::multiset<Sales_data, compareType>::iterator c_it = bookstore.begin();
```

### 11.3.2 添加元素

> 11.20 重写11.1节练习的单词计数程序，使用`insert`代替下标操作。你认为哪个程序更容易编写和阅读？解释原因。

```cpp
#include <map>
#include <string>
#include <iostream>

int main(){
	std::map<std::string,int>mp;
	std::string s;
	
	while(std::cin >> s){
		auto result = mp.insert({s,1});
		if(!result.second){
			++result.first->second;
		}
	}
	
	for(const auto& e:mp){
		std::cout << e.first << " " << e.second << std::endl;
	}
	return 0;
}
```

下标操作更容易

* 对一个`map`使用下标操作，其行为与数组或`vecotr`上的下标操作很不相同，使用一个不在容器中的关键字作为下标，会添加一个具有此关键字的元素到`map`中
* `c[k]`返回关键字为`k`的元素，如果`k`不在`c`中则添加一个关键字为`k`的元素，对其值进行值初始化



> 11.21 假定`word_count`是一个`string`到`size_t`的`map`，`word`是一个`string`，解释下面循环的作用：

```cpp
while (cin >> word)
	++word_count.insert({word, 0}).first->second;
```

等价

```cpp
while(cin >> word){
	++word_count[word];
}
```



> 11.22 给定一个`map<string, vector<int>>`，对此容器的插入一个元素的`insert`版本，写出其参数类型和返回类型。

* 参数类型：`std::pair<std::string,std::vector<int>`

* 返回类型：`std::pair<std::map<std::string,std::vector<int>>::iterator,bool`



> 11.23 11.2.1节练习中的`map` 以孩子的姓为关键字，保存他们的名的`vector`，用`multimap` 重写此`map`。

```cpp
#include <cctype>
#include <map>
#include <string>
#include <iostream>
#include <algorithm>
#include <vector>

int main(){
	std::multimap<std::string,std::string>mp;
	std::string family_name,children_name;
	while(std::cin >> family_name >> children_name){
		mp.emplace(family_name,children_name);
	}
	return 0;
}
```



### 11.3.4 map的下标操作

> 11.24 下面的程序完成什么功能？

```cpp
map<int, int> m;
m[0] = 1;//添加key-value:0-1
```

`c[k]`返回关键字为`k`的元素，如果`k`不在`c`中则添加一个关键字为`k`的元素，对其值进行值初始化



> 11.25 对比下面的程序与上一题程序

```cpp
vector<int> v;
v[0] = 1;//下标越界访问
```

对一个`map`使用下标操作，其行为与数组或`vecotr`上的下标操作很不相同，使用一个不在容器中的关键字作为下标，会添加一个具有此关键字的元素到`map`中



> 11.26 可以用什么类型来对一个`map`进行下标操作？下标运算符返回的类型是什么？请给出一个具体例子——即，定义一个`map`，然后写出一个可以用来对`map`进行下标操作的类型以及下标运算符将会返会的类型。

```cpp
std::map<int, std::string> m = { { 1,"ss" },{ 2,"sz" } };
using KeyType = std::map<int, std::string>::key_type;	
using ReturnType = std::map<int, std::string>::mapped_type;
```

* 对`map`进行下标操作会得到一个`mapped_type`对象，解引用`map`迭代器时，会得到一个`value_type`对象，这与`vector`、`string`是不同的
* `map`的下标运算符返回的是一个左值，我们既可以读也可以写元素



### 11.3.5 访问元素

> 11.27 对于什么问题你会使用`count`来解决？什么时候你又会选择`find`呢？

允许重复用`count`不允许重复用`find`



> 11.28 对一个`string`到`int`的`vector`的`map`，定义并初始化一个变量来保存在其上调用`find`所返回的结果。

```cpp
#include <cctype>
#include <map>
#include <string>
#include <iostream>
#include <algorithm>
#include <vector>

int main(){
	std::map<std::string,std::vector<int>>mp={{"a",{1,2,3}},{"ab",{1,2}},{"abc",{3,4}}};
	auto result = mp.find("abc");
	if(result != mp.end()){
		std::cout << result->first << " " << std::endl;
		for(auto i:result->second)std::cout << i << " ";
	}
	return 0;
}
```



> 11.29 如果给定的关键字不在容器中，`upper_bound`、`lower_bound` 和 `equal_range` 分别会返回什么？

* `c.lower_bound(k)`返回一个迭代器，指向第一个关键字不小于`k`的元素，**如果关键字不在容器中，则`lower_bound`会返回关键字的第一个安全插入点（不影响容器中元素顺序的插入位置）**

* 可以通过`lower_bound`和`upper_bound`，获得一对迭代器对形成的范围，表示具有该关键字的元素的范围，如果没有与给定关键字匹配的元素，`lower_bound`和`upper_bound`返回相同的迭代器

* `c.equal_range(k)`返回一个迭代器`pair`表示关键字等于`k`的元素范围，若`k`不存在，`pair`的两成员均等于`c.end()`（两个关键字都指向可以插入的位置）



> 11.30 对于本节最后一个程序中的输出表达式，解释运算对象`pos.first->second`的含义。

* `c.equal_range(k)`返回一个迭代器`pair`表示关键字等于`k`的元素范围，若`k`不存在，`pair`的两成员均等于`c.end()`（两个关键字都指向可以插入的位置）

`pos`是一个`pair`

`pos.first`是一个指向匹配关键字元素的迭代器

`pos.first->second`匹配关键字元素的关联也是个`pair`，访问这个`pair`的第二个成员



> 11.31 编写程序，定义一个作者及其作品的`multimap`。使用`find`在`multimap`中查找一个元素并用`erase`删除它。确保你的程序在元素不在`map` 中时也能正常运行。

```cpp
#include <cctype>
#include <map>
#include <string>
#include <iostream>
#include <algorithm>
#include <vector>

int main(){
	std::multimap<std::string,std::string>authors{
		{"A","123"},{"B","456"},{"C","789"},
		{"A","abc"},{"B","def"},{"C","ghi"}
	};
	auto found = authors.find("B");
	auto count = authors.count("B");
	while(count){
		if(found -> second == "def"){
			authors.erase(found);
			break;
		}
		++found;
		--count;
	}

	for(const auto &author:authors){
		std::cout << author.first << " " << author.second << std::endl;
	}
	return 0;
}
```



> 11.32 使用上一题定义的`multimap`编写一个程序，按字典序打印作者列表和他们的作品。

```cpp
#include <cctype>
#include <map>
#include <string>
#include <iostream>
#include <algorithm>
#include <vector>
#include <set>

int main(){
	std::multimap<std::string,std::string>authors{
		{"B","456"},{"C","789"},{"A","abc"},{"A","123"},{"B","def"},{"C","ghi"}
	};
	std::map<std::string,std::multiset<std::string>>sorted_authors;
	
	for(const auto &author:authors){
		sorted_authors[author.first].emplace(author.second);
	}
	
	for(const auto &author:sorted_authors){
		for(const auto &work:author.second){
			std::cout << author.first << " " << work << std::endl;
		}
	}
	return 0;
}
```



### 11.3.6 一个单词转换的 map

> 11.33 实现你自己版本的单词转换程序。

```cpp
#include <cctype>
#include <fstream>
#include <map>
#include <string>
#include <iostream>
#include <algorithm>
#include <vector>
#include <sstream>

std::map<std::string,std::string> buildMap(std::ifstream &map_file){
	std::map<std::string,std::string>trans_map;
	std::string key,value;
	while(map_file >> key && getline(map_file,value)){
		if(value.size()>1)
			trans_map[key] = value.substr(1);
		else
			throw std::runtime_error("no rule for " + key);
	}
	return trans_map;
}

const std::string& transform(const std::string &s,const std::map<std::string,std::string>& m){
	auto map_it = m.find(s);
	if(map_it != m.cend())return map_it ->second;
	return s;
}

void word_transform(std::ifstream &map_file,std::ifstream &input){
	auto trans_map = buildMap(map_file);
	for(std::string text;std::getline(input,text);){
		std::istringstream stream(text);
		bool firstword = true;
		for(std::string word;stream >> word;){
			if(firstword)
				firstword = false;
			else 
				std::cout << " ";
			std::cout << transform(word,trans_map);
		}
		std::cout << std::endl;
	}
}

int main(int argc,char **argv){
	std::ifstream ifs_map(argv[1]),ifs_content(argv[2]);
	if(ifs_map && ifs_content)word_transform(ifs_map, ifs_content);
	else std::cerr << "cant't find the documents." << std::endl;

	return 0;
}
```



> 11.34 如果你将`transform` 函数中的`find`替换为下标运算符，会发生什么情况？

如果关键字不在容器中，会往容器中添加一个元素



> 11.35 在`buildMap`中，如果进行如下改写，会有什么效果？

```cpp
trans_map[key] = value.substr(1);//关键字多次出现，下标保留最后一次
//改为
trans_map.insert({key, value.substr(1)});//关键字多次出现，insert保留第一次
```



> 11.36 我们的程序并没检查输入文件的合法性。特别是，它假定转换规则文件中的规则都是有意义的。如果文件中的某一行包含一个关键字、一个空格，然后就结束了，会发生什么？预测程序的行为并进行验证，再与你的程序进行比较。

```cpp
std::map<std::string,std::string> buildMap(std::ifstream &map_file){
	std::map<std::string,std::string>trans_map;
	std::string key,value;
	while(map_file >> key && getline(map_file,value)){
		if(value.size()>1)
			trans_map[key] = value.substr(1);
		else
			throw std::runtime_error("no rule for " + key);//抛出runtime_error
	}
	return trans_map;
}
```



## 11.4 无序容器

> 11.37 一个无序容器与其有序版本相比有何优势？有序版本有何优势？

无序容器通过`hash`函数创建，性能更好

有序容器可以保持元素有序



> 11.38 用 `unordered_map` 重写单词计数程序和单词转换程序。

```cpp
#include <cctype>
#include <fstream>
#include <map>
#include <string>
#include <iostream>
#include <algorithm>
#include <vector>
#include <sstream>
#include <unordered_map>

std::unordered_map<std::string,std::string> buildMap(std::ifstream &map_file){
	std::unordered_map<std::string,std::string>trans_map;
	for(std::string key,value;map_file >> key && getline(map_file,value);){
		if(value.size()>1) trans_map[key] = value.substr(1);
		else throw std::runtime_error("no rule for " + key);
	}
	return trans_map;
}

const std::string& transform(const std::string &s,const std::unordered_map<std::string,std::string>& m){
	auto map_it = m.find(s);
	if(map_it != m.cend())return map_it ->second;
	return s;
}

void word_transform(std::ifstream &map_file,std::ifstream &input){
	auto trans_map = buildMap(map_file);
	for(std::string text;std::getline(input,text);){
		std::istringstream stream(text);
		bool firstword = true;
		for(std::string word;stream >> word;){
			if(firstword) firstword = false;
			else std::cout << " ";
			std::cout << transform(word,trans_map);
		}
		std::cout << std::endl;
	}
}

int main(int argc,char **argv){
	std::ifstream ifs_map(argv[1]),ifs_content(argv[2]);
	if(ifs_map && ifs_content)word_transform(ifs_map, ifs_content);
	else std::cerr << "cant't find the documents." << std::endl;

	return 0;
}
```

