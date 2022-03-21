# deque

## deque 概述

* 双向开口的连续线性空间
* deque 允许常数时间内对头端进行插入或移除
* deque 没有容量（capacity）的观念，因为是动态地以分段连续空间组合而成，随时可以增加一段新的空间拼接起来。所以也没必要提供保留空间的功能（reserve）
* deque 和 vector 一样提供随机访问迭代器但不是普通指针
* 排序：可将deque先完整复制到一个vector身上，排序后在复制到deque

## deque 的中控器

deque 由一段段定量连续空间构成，如果要在前端或尾端增加新的空间，便配置一段定量连续空间，串接在deque的头端或尾端

deque 的最大任务就是维护整体连续假象并提供随机存取接口，避免了重新配置、复制、释放，代价是迭代器复杂架构

deque 采用一块所谓的 map（不是 STL 的 map）容器作为主控，这里的 map 是一小块连续空间，其中每个元素都是指针，指向另一段较大的连续线性空间，称为缓冲区，缓冲区才是 deque 存储空间的主体

实际上 map 是一个 T**，也就是说他是一个指针，所指之物又是一个指针，指向型别为 T 的一块空间

![image-20220321124200616](https://s2.loli.net/2022/03/21/zLvcqMQ6oNPufa7.png)

## deque 的迭代器

deque 是分段连续空间，维持其整体假象的任务落在了迭代器`operator++`和`operator--`两个运算子身上

![image-20220321125011458](https://s2.loli.net/2022/03/21/LrI4WaJjfzvKcnm.png)

![image-20220321125048241](https://s2.loli.net/2022/03/21/RkiSl2r89BxX71L.png)

`operator++`：先切换到下一个元素，如果到达所在缓冲区尾端`cur = last`则调用`set_node(node+1)`跳转到下一个结点，并令`cur = first`。后置递增调用前置递增

`operator--`：如果已到达所在缓冲区头端`cur = first`，切换到前一个结点，并令`cur = last`，然后切换到前一个元素。后置递增调用前置递增

`operator+=`：实现随机存取，迭代器可以直接跳跃 n 个距离。如果目标位置在同意缓冲区内，`cur += n`，否则先`set_node(node + node_offset)`切换到正确结点，然后跳转元素

![image-20220321130207566](https://s2.loli.net/2022/03/21/Wrv7pVm3jFOBdwK.png)



## deque 数据结构

 

## deque与vector对比

* 扩容方式
* 头端增删
* 底层结构
