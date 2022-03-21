# queue

## queue概述

* 先进先出的数据结构，两个出口
* 不允许遍历

## queue 定义完整列表

* deque 作为缺省情况下的 stack 底层结构，封闭前端入口和后端出口
* `queue` 以底层容器完成工作，修改某物接口形成另一种风貌的一般称为 adpater。因此 `queue` 往往不被归为容器，而被归为`container adapter`

## queue 没有迭代器

只有 `queue`顶端的元素，才会被外界取用，`queue`不提供走访功能，也不提供迭代器

## 以 list 作为 queue 的底层容器

除了 `deque`，`list`也是双向开口，所以可以以`list`作为底层容器并封闭一些开口实现
