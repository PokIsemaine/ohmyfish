# Traits 技法

## Iterator

STL 中心思想：将数据容器与算法分开

容器泛化：`class templates`

算法泛化：`function templates`

胶合剂：迭代器

迭代器是一种智能指针

## 迭代器对应的型别

在算法中运用迭代器时，很可能用到相应型别

假设算法中有必要声明一个变量，以迭代器所指对象的型别为型别，该怎么办？

### 尝试1 template 参数推导

我们可以利用 `function template`的参数推导机制

但这智能推导参数，不能推导返回值

### 尝试2 声明内嵌型别

但并不是所有迭代器都是 `class type`，没能支持原生指针

### 尝试3 template partial sepcialization

视觉污染了

### 尝试4 加入中间层

