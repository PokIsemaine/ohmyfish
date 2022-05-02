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



### 代码：erase 删除

```cpp
for(map<int,int>::iterator it = mapInt.begin(); it != mapInt.end();) {
    if(it->second == 0) {
        mapInt.erase(it++);
    } else {
        it++;
    }
}
```



## multimap

`multimap` 的特性以及用法和`map`完全相同，唯一的差别在于它允许键值重复，因此它采用的底层机制是`RB-tree`的`insert_equal()`而非`insert_unique()`



## hash_map

以`hashtable`为底层机制，相比`map`没有自动排序的功能，其他基本一样



## hash_multimap

和`multimap`基本完全一样，只不过`hash_multimap`底层机制是`hashtable`不会被自动排序

和`hash_map`实现上的唯一差别在于前者的元素插入操作采用底层`hash table`的`insert_equal()`而不是`insert_unique()`