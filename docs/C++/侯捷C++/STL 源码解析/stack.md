# stack

## stack 概述

* stack是一种先进后出数据结构
* stack允许新增、移除、取得最顶端元素，没有其他方法存取其他元素
* 不允许遍历行为

## stack 完整定义列表

* deque 作为缺省情况下的 stack 底层结构
* stack 以底层容器完成工作，修改某物接口形成另一种风貌的一般称为 adpater。因此 `stack `往往不被归为容器，而被归为`container adapter`

## stack 没有迭代器

只有 stack 顶端的元素，才会被外界取用，`stack`不提供走访功能，也不提供迭代器

## 以 list 作为 stack 的底层容器

除了 `deque`，`list`也是双向开口，所以可以以`list`作为底层容器并封闭器头端开口

