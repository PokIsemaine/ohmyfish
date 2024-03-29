# Facade

## 动机

![image-20220211234944642](https://s2.loli.net/2022/02/11/NqVEoMZ48xW5PtF.png)

* 上述 A 方案的问题在于组件的客户和组件中各种复杂的子系统有了过多的耦合，随着外部客户程序和各子系统有了过多的耦合，随着外部苦护程序和各子系统的演化，这种过多耦合面临很多变化的挑战
* 如何简化外部客户程序程序和系统间的交互接口？如何将外部客户程序的演化和内部子系统的变化之间的依赖相互解耦？

## 模式定义

> 为子系统中的一组接口提供一个一致(稳定)的界面，Façade 模式定义了一个高层接口，这个接口使得这一子系统更加容易使用(复用)。	——《设计模式》GoF



## SHOW ME THE CODE

`Facad`模式没有特定的代码结构

`Faced`设计模式更像是一种设计的素养，我们对内的层次和对外的层次要有个划分，对外稳定，对内又要求快速更新



## 结构设计

![image-20220211235619310](https://s2.loli.net/2022/02/11/pZKJaQ6HSzIU2XP.png)

## 要点总结

- 从客户程序角度来看，Façade 模式简化了整个组件系统的接口，对于组件内部与外部的客户程序来说， 达到了一种”解耦“的效果——内部子系统的任何变化不会影响到 Façade 接口的变化。
- Façade 设计模式**更注重架构的层次去看整个系统，而不是单个类的层次**。Façade 很多时候是一种架构设计模式。
- Façade 设计模式并非一个集装箱，可以任意地放进任何多个对象。Façade 模式组件中的内部应该是”**相互耦合关系比较大的**一系列组件“，而不是一个简单的功能集合。（例如已经确定`Facade`是一个数据访问层，那里面就应该是数据访问相关的、耦合关系比较大 的对象，而不是啥都往里面塞。对外松耦合，对内高内聚。）