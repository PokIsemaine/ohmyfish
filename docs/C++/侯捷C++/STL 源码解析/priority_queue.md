# priority_queue

## priority_queue 概述

拥有权值观念的 `queue`，只允许底端加入元素并从顶端取出元素

缺省情况下`priority_queue`利用一个`max-heap`完成，后者是一个以`vector`表现的`complete binary tree`

## priority_queue 定义完整列表

缺省情况下 `priority_queue`以`vector`为底部容器，加上`heap`处理规则（`binary heap` 见下面）

`priority_queue` 以底层容器完成工作，修改某物接口形成另一种风貌的一般称为 adpater。因此 `priority_queue` 往往不被归为容器，而被归为`container  adapter`

## priority_queue 没有迭代器

只有`priority_queue`顶端的元素才有机会被外界取用，`priority_queue`不提供遍历功能，也不提供迭代器



## heap （隐式表述）

### heap 概述

`heap` 并不属于 STL 容器组件，扮演 `priority queue`的幕后助手

如果`list`实现那么找极值需要$O(N)$遍历

如果`binary search tree`（例如`RB-tree`）实现，虽然插入和找极值都是$O(logN)$但有点杀鸡用牛刀了。一方面`binary search tree`的输入需要足够随机性，另一方面实现太麻烦了

因此`binary heap`就成了最佳候选人，`binary heap`是一颗完全二叉树，并且我们可以用一个`array`（`vector`）来隐式表述堆

`heap`没有迭代器



### heap 算法

`push_heap`



`pop_heap`



`sort_heap`





