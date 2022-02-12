# Proxy

## 动机

- 在面向对象系统中，有些对象由于某种原因(比如对象创建的开销很大，或者某些操作需要安全控制，或者需要进程外的访问等)， 直接访问会给使用者、或者系统结构带来很多麻烦。
- 如何在不失去透明操作对象的同时来管理/控制这些对象特有的复杂性？增加一层间接层是软件开发中常见的解决方式。



## 模式定义

> 为其他对象提供一种代理以控制(隔离，使用接口)对这对象的访问。 ——《设计模式》GoF



## 结构设计

这次先看结构设计

![image-20220212000910421](https://s2.loli.net/2022/02/12/URBb95CM7823Zxi.png)

`Subject`是一个接口

本来我们`Client`创建的时候通过`Subject`接口直接去创建和访问`RealSubject`对象

但是由于种种原因（控制安全、性能优化、或者是一些分布式的完全就做不到）

那么现在我们就要通过`Proxy`去访问

## SHOW ME THE CODE

`client.cpp`

```cpp
class ISubject{
public:
    virtual void process();
};


class RealSubject: public ISubject{
public:
    virtual void process(){
        //....
    }
};

class ClientApp{//直接创建和访问可能不符合要求（安全、性能）或者像分布式系统就不能实现
    
    ISubject* subject;
    
public:
    
    ClientApp(){
        subject=new RealSubject();
    }
    
    void DoTask(){
        //...
        subject->process();
        
        //....
    }
};
```

使用`Proxy`间接访问

`proxy.cpp`

```cpp
class ISubject{
public:
    virtual void process();
};


//Proxy的设计
class SubjectProxy: public ISubject{
    
    
public:
    virtual void process(){
        //对RealSubject的一种间接访问
        //....
    }
};

class ClientApp{
    
    ISubject* subject;
    
public:
    
    ClientApp(){
        subject=new SubjectProxy();
    }
    
    void DoTask(){
        //...
        subject->process();
        
        //....
    }
};
```



## 要点总结

* 增加一层层是软件系统中对许多复杂问题的一种常见解决方法。在面向对象系统中，直接使用某些对象会带来很多问题，作为间接层的`proxy`对象是解决这一问题的常用手段
* 具体`proxy`设计模式的实现方法、实现粒度都相差很大，有些可能对单个对象做细粒度的控制，如`copy-on-write`技术，有些可能对组件模块提供抽象代理层，在架构层次对对象做`proxy`

* Proxy并不一定要求保持接口完整的一致性，只要能够实现间接控制，有时候损及一些透明性是可以接受的。