# Summary 



## 容器与迭代器

### 迭代器类型

| 容器                                                       | 迭代器                                                       |
| ---------------------------------------------------------- | ------------------------------------------------------------ |
| vector、deque                                              | 随机访问迭代器                                               |
| stack、queue、priority_queue                               | 无                                                           |
| list、（multi)set、(multi)map                              | 双向迭代器<br />`set`的迭代器为`const_iterator`杜绝写入操作，是一种`constant iterator`<br />`map`的迭代器既不是`constant iterator`也不是`mutable iterators` |
| forward_list、unordered\_(multi)map、unordered\_(multi)set | 前向迭代器                                                   |



### 迭代器失效

| 容器              | 失效情况                                                     |
| ----------------- | ------------------------------------------------------------ |
| vector、deque     | 插入元素<br />1. 尾后插入，size < capacity：首迭代器不失效，尾迭代器失效<br />2. 尾后插入/中间插入，size == capacity：所有迭代器均失效（需要重新分配空间）<br />3. 中间插入，size < capacity：首迭代器不失效，插入元素之后的所有迭代器失效<br />删除元素<br />尾后删除：只有尾迭代器失效<br />中间删除：删除位置之后所有迭代器失效 |
| list              | 插入、接合不会造成迭代器失效<br />删除节点：仅当前迭代器失效，erase 返回下一个有效迭代器 |
| map、set          | 删除节点：仅当前迭代器失效，mp.erase(iter++)获取下一个迭代器 |
| unordered_map/set | 迭代器意义不大，rehash 之后全部失效                           |



## 容器底层实现

| 容器           | 底层实现                                                     |
| -------------- | ------------------------------------------------------------ |
| list           | 双向循环链表                                                 |
| vector         | 单向开口的连续线性空间，因此普通指针都可以作为 vector 的迭代器 |
| deque          | 双向开口的连续线性空间，动态地以分段连续空间组合而成。因此 deque 没有容量概念，也没提供空间保留（reserve） |
| stack/queue    | deque 或 list                                                |
| priority_queue | vector 形式的 `binary heap`                                  |
|                |                                                              |



## 容器扩容

| 容器   | 扩容方式                                        |
| ------ | ----------------------------------------------- |
| array  | 大小固定，不能扩容                              |
| vector | 重新分配空间（2 倍/1.5 倍），移动数据，释放原空间 |
| deque  |                                                 |
|        |                                                 |
|        |                                                 |
|        |                                                 |


## 删除元素

https://zhuanlan.zhihu.com/p/450086692

```c++
bool badValue(int) { return true; } // 返回x是否为"坏值"
 
int test_item_9()
{
	// 第一种情况：删除 c 中所有值指定值 2021 的元素
	std::vector<int> c1;
	c1.erase(std::remove(c1.begin(), c1.end(), 2021), c1.end()); // 当c1是vector, string或deque时，erase-remove习惯用法是删除特定值的元素的最好办法
 
	std::list<int> c2;
	c2.remove(2021); // 当c2是list时，remove成员函数是删除特定值的元素的最好办法
 
	std::set<int> c3;
	c3.erase(2021); // 当c3是标准关联容器时，erase成员函数是删除特定值元素的最好办法
 
	// 第二种情况：删除判别式(predicate)返回 true 的每一个对象
	c1.erase(std::remove_if(c1.begin(), c1.end(), badValue), c1.end()); // 当c1是vector, string或deque时，这是删除使badValue返回true的对象的最好办法
 
	c2.remove_if(badValue); // 当c2是list时，这是删除使badValue返回true的对象的最好办法
 
	for (std::set<int>::iterator i = c3.begin(); i != c3.end();) {
		if (badValue(*i)) c3.erase(i++); // 对坏值，把当前的i传给erase，递增i是副作用
		else ++i;                        // 对好值，则简单的递增i
	}//当对关联式容器来说
 
	// 第三种情况，每次元素被删除时，还需要都向一个日志(log)文件中写一条信息
	std::ofstream logFile;
	for (std::set<int>::iterator i = c3.begin(); i != c3.end();) {
		if (badValue(*i)) {
			logFile << "Erasing " << *i << '\n'; // 写日志文件
			c3.erase(i++); // 对坏值，把当前的i传给erase，递增i是副作用
		}
		else ++i;              // 对好值，则简单第递增i
	}//当对关联式容器来说
 
	for (std::vector<int>::iterator i = c1.begin(); i != c1.end();) {
		if (badValue(*i)) {
			logFile << "Erasing " << *i << '\n';
			i = c1.erase(i); // 把erase的返回值赋给i，使i的值保持有效
		}
		else ++i;
	}//当对序列式容器来说
 
	return 0;
}
```
* 要删除容器中有特定值的所有对象：如果容器是 vector，string 或 deque，则使用 erase-remove 习惯用法；如果容器是 list，则使用 list::remove；如果容器是一个标准关联容器，则使用它的 erase 成员函数。
* 要删除容器中满足特定判别式(条件)的所有对象：如果容器是 vector， string 或 deque，则使用 erase-remove_if 习惯用法；如果容器是 list，则使用 list::remove_if；如果容器是一个标准关联容器，则使用 remove_copy_if 和 swap，或者写一个循环来遍历容器中的元素，记住当把迭代器传给 erase 时，要对它进行后缀递增。
* 要在循环内做某些(除了删除对象之外的)操作：如果容器是一个标准序列容器，则写一个循环来遍历容器中的元素，记住每次调用 erase 时，要用它的返回值更新迭代器；如果容器是一个标准关联容器，则写一个循环来遍历容器中的元素，记住当把迭代器传给 erase 时，要对迭代器做后缀递增。