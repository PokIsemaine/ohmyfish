# map

## 概述

* 所有元素都会根据元素的键值自动排序
* 所有元素都是`pair`同时拥有`value`和`key`，`pair`第一个元素被视为`key`
* 不允许两个元素拥有相同的键值

## 迭代器

* 不能通过迭代器修改键值，但是可以修改实值
* `map iterators`既不是一种`constant iterators`也不是一种`mutable iterators`

### 插入删除

`map`拥有与`list`相同的某些性质：当插入删除时，操作之前的所有迭代器，在操作完成后依然有效（除了被删除的）



## multimap

`multimap` 的特性以及用法和`map`完全相同，唯一的差别在于它允许键值重复，因此它采用的底层机制是`RB-tree`的`insert_equal()`而非`insert_unique()`