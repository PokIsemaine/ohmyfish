# 设计模式

## 目标

* 理解松耦合**设计思想**
* 掌握面向对象**设计原则**
* 掌握**重构技法**改善设计
* 掌握GOF核心**设计模式**

## 参考资料

* GOF的《设计模式》
* 李建忠的C++设计模式课程

## 前言

在学习设计模式之前我觉得，有必要了解一下设计模式是干什么用的。

[为什么我们需要学习（设计）模式](https://zhuanlan.zhihu.com/p/19835717)

[设计模式有何不妥，所谓的荼毒体现在哪？](https://www.zhihu.com/question/23757237)

我个人认为有以下几点需要牢记的

* 不要滥用设计模式，不要为了使用设计模式而使用设计模式，设计模式是用来解决问题的
* 设计模式可以认为是一些经验总结或者说是一些建议，不是什么非常牛逼的东西，不要认为学了设计模式就高人一等
* 学习设计模式可以减少一些沟通成本，方便我们理解代码，当然有可能让代码逻辑变得复杂一些，你需要自己权衡
* 设计模式应该重点学习解决问题的方法策略，而不是代码
* 设计模式可以算是对编程语言本身的一种弥补，有的新语言可能在被设计时就已经通过语法糖帮你实现了某些设计模式

## 内容

*  [序章](/课堂之外/设计模式/序章.md)
* 组件协作
	* Template Method
	* Observer/Event
	* Strategy
* 单一职责
	* Decorator
	* Bridge
* 对象创建
	* Factory Method
	* Abstract Factory
	* Prototype
	* Builder
* 对象性能
	* Singleton
	* Flyweight
* 接口隔离
	* Facade
	* Proxy
	* Mediator
	* Adapter
* 状态变化
	* Memento
	* State
* 数据结构
	* Composite
	* Iterator
	* Chain of Resposibility
* 行为变化
	* Command
	* Visitor
* 领域问题
	* Interpreter
