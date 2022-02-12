# 设计模式

## 目标

* 理解松耦合**设计思想**
* 掌握面向对象**设计原则**
* 掌握**重构技法**改善设计
* 掌握GOF核心**设计模式**

## 参考资料

* GOF的《设计模式》
* 李建忠的C++设计模式课程
* [Cpp-Design-Patterns 代码仓库](https://github.com/liu-jianhao/Cpp-Design-Patterns)

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

## 概览

<img src="https://s2.loli.net/2022/02/06/auHWd72ORhgrzsV.png" alt="设计模式" style="zoom:95%;" />

## 内容

*  [序章](/课堂之外/设计模式/序章.md)
*  [要点和常见误区](/课堂之外/设计模式/要点和常见误区.md)
*  组件协作
  *  [Template Method 模板方法](/课堂之外/设计模式/Template%20Method.md)
  *  [Observer/Event 观察者模式](/课堂之外/设计模式/Observer.md)
  *  [Strategy 策略模式](/课堂之外/设计模式/Strategy.md)

*  单一职责
  * [Decorator 装饰模式](/课堂之外/设计模式/Decorator.md)
  * [Bridge 桥模式](/课堂之外/设计模式/Bridge.md)
*  对象创建
  *  [Factory Method 工厂方法](/课堂之外/设计模式/Factory%20Method.md)
  *  [Abstract Factory 抽象工厂](/课堂之外/设计模式/Abstract%20Factory.md)
  *  [Prototype 原型模式[不常用]](/课堂之外/设计模式/Prototype.md)
  *  [Builder 构建器[不常用]](/课堂之外/设计模式/Builder.md)

*  对象性能
  *  [Singleton 单例模式](/课堂之外/设计模式/Singleton.md)
  *  [Flyweight 享元模式](/课堂之外/设计模式/Flyweight.md)

*  接口隔离
  *  [Facade 门面模式](/课堂之外/设计模式/Facade.md)
  *  [Proxy 代理模式](/课堂之外/设计模式/Proxy.md)
  *  [Mediator 中介者[不常用]](/课堂之外/设计模式/Mediator.md)
  *  [Adapter 适配器模式](/课堂之外/设计模式/Adapter.md)

*  状态变化
  *  [Memento 备忘录[不常用]](/课堂之外/设计模式/Memento.md)
  *  [State](/课堂之外/设计模式/State.md)

*  数据结构
  *  Composite
  *  Iterator[不常用]
  *  Chain of Resposibility[不常用]

*  行为变化
  *  Command[不常用]
  *  Visitor[不常用]

*  领域问题
  *  Interpreter[不常用]
*  [忘记模式，唯有原则](/课堂之外/设计模式/忘记模式唯有原则.md)





