# set

## 概述

* 所有元素根据元素的健值自动被排序，键值就是实值
* 不允许两个元素有相同的键

## 迭代器

* 不能通过 `set`的迭代器改变`set`的元素值，因为键值就是实值如果随意改变会破坏组织结构
* `set`迭代器被定义为底层`RB-tree`的`const_iterator`迭代器，杜绝写入操作

### 插入删除

`set`拥有与`list`相同的某些性质：当插入删除时，操作之前的所有迭代器，在操作完成后依然有效（除了被删除的）

### 代码：erase 删除

```

```



## multiset

`multiset` 的特性以及用法和`set`完全相同，唯一的差别在于它允许键值重复，因此它采用的底层机制是`RB-tree`的`insert_equal()`而非`insert_unique()`

`

## hash_set

以`hashtable`为底层机制，相比`set`没有自动排序的功能，其他基本一样



## hash_multiset

和`multiset`基本完全一样，只不过`hash_multiset`底层机制是`hashtable`不会被自动排序

和`hash_set`实现上的唯一差别在于前者的元素插入操作采用底层`hash table`的`insert_equal()`而不是`insert_unique()`
